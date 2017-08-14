# Week 11 (10 August to 16 August)

## Summary

## Progress

### Tests fixing

#### fork.spec.js

This test was giving an error while matching task result uid with the 
changing uid from fork.

```javascript
assert.equal(results[0].tasks[1], hash(lineCount.info.uid + 0))
```

and

```javascript
assert.equal(results[1].tasks[1], hash(lineCount.info.uid + 1))
```

However with the new uid creation for tasks after fork this became broken and
 there is no good way for knowing the changed uid by fork before actually 
 running watermill, because uid gets changed from the original uid to allow 
 for the same task to be run more than once within a pipeline (documented 
 [here](https://github.com/bionode/GSoC17/blob/master/Journal/Week_10.md#merkle-tree-like-task-uid)).

Nevertheless, I replaced this check to match the uid within trajectory to 
assure that it is properly being passed to the pipeline workflow rather than 
matching pattern from the previously implemented change promoted by fork for 
the tasks outside fork (which added a number to task uid each time it 
encountered a fork branch).
Therefore, now it checks if the final task `uid` matches the last item in 
`context.trajectory` for that task. This allows to check if workflow is being
 properly created.
 
 ```javascript
 assert.equal(results[0].tasks[1], results[0].context.trajectory[2])
 ```
 
 and
 
 ```javascript
 assert.equal(results[1].tasks[1], results[1].context.trajectory[2])
 ```

#### junction.spec.js

Since `graphson` related changes this became broken and in fact it was quite 
easy to solve because now the path to the outputs is stored in `resolvedOutput`:

```javascript
const finalOutput = store.getState().collection.vertexValue(lastTaskUid).resolvedOutput
```

rather than:

```javascript
const finalOutput = store.getState().collection.vertexValue(lastTaskUid)
```

For more information on these changes check PR #70.

### junction final vertex check

Junction works without final vertex (task).

```javascript
join(task0, junction(task1, task2))
```


![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/junction_graph/junction_without_endvertex.png)

Also 

```javascript
junction(task1, task2)
```

works but renders two non connected nodes.

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/junction_graph/junction_without_startendvertex.png)

The end `junction` orange vertex is useful when in fact a task is executed 
following `junction`, however if there is nothing after the `junction` it works 
somehow similar `fork`.
 
### forkception

After struggling for a while with trying to make fork work within fork, I 
just realized that we can make fork work with fork in two distinct ways:

First way:

```javascript
const pipeline = join(
  task0,
  fork(
    join(task2,task5),
    join(task4,
      fork(
        join(task1,task5),
        join(task3,task5)
      )
    )
  )
)

```

Second way:

```javascript
const pipeline2 = join(
  task0,
  fork(
    join(
      task4,
        fork(
          task1,
          task3
        )
    ),
    task2
  ),
  task5
)
```

In fact, the second way is the one that is intended for fork to work. 
However, there is nothing against making the first approach. In fact, there 
is a reason for doing the first one (for now...). It in fact works properly!

Expected pipeline shape:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_scheme_1.png)

Result from first way with current api:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/forking_the_noob_way.png)

Result with second way:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/forking_fork_style_error.png)

So, in current api `taks5` isn't being correctly duplicated for each branch, 
that is way manual handling it worked for the first way. Ideally `fork` should 
handle this, otherwise it would become a `junction` without a junction vertex.

**What is in fact missing?**

With this I think if we create a new instance every time there is a task 
within forkee (in `join.js`) we can dispatch a new tasks for the outermost 
task (in this case `task5`), but just once because we just need one `task5` 
in each tip of the graph.

**Update**

First I started by handling a fork inside another fork (wrapped in a join):

```javascript
const pipeline2 = join(
  task0,
  fork(
    join(
      task4,
        fork(
          task1,
          task3
        ),
      task6
    ),
    task2
  ),
  task5
)
```

Expected result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_pipeline2_expected.png)

Actual result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_pipeline2.png)

First, I started by storing outermost task (`task5`) at the first fork 
instance, and in order for not duplicating the effort of storing 
`outermostTasks` more than once it is only stored at the end of first fork 
tasks parsing, i.e., when `task.info.tasks.length - 1 === j` where `j` is the
 number of loop cycles for `task.info.tasks`.
 
 Also, everything needs to be duplicated (in these case where there are only 
 two branches for fork) for every `forkee` (branch of `fork`).
 
 ```javascript
 if (firstFork === true) {
   // this if statement is used to store outermostTasks for inner
   // orchestrators inside fork
   console.log('test:', forkee.info, outermostTasks)
   const newUpStreamTasks = taskCreationDispatcher(dispatch, upStreamTasks, task.info.uid, forkee.info.uid)
   // if a task is found for outer fork it will dispatch the outer
   // task as newUpStreamTasks, otherwise it will concat the other
   // orchestrators
   const lineage = [forkee].concat(newUpStreamTasks)
   lineageGetter(dispatch, lineage, joinages)
   // this is used to control when firstFork instance ends
   if (j == task.info.tasks.length - 1) {
     firstFork = false
     outermostTasks = newUpStreamTasks
   }
 }
```

Besides that we still needed to add this stored `outermostTasks` to the to 
the _inner_ `fork` within the _outer_ `fork`. But we have also to add the 
`upstreamTasks` that may exist after this _outer_ `fork`:

```javascript
if (forkee.info.props) {
  const newUpStreamTasks = taskCreationDispatcher(dispatch, upStreamTasks, task.info.uid, forkee.info.uid)
  const newOutermostTasks = taskCreationDispatcher(dispatch, outermostTasks, task.info.uid, forkee.info.uid)
  const lineage = [forkee].concat(newUpStreamTasks).concat(newOutermostTasks)
  lineageGetter(dispatch, lineage, joinages)  
}
```

However this did not handle the case where a `join` is not wrapping the inner
 fork. One may wish to just fork after the fork, something like the following
  pipeline:
  
  ```javascript
const pipeline3 = join(
  task0,
  fork(
      fork(
        task1,
        task3
      ),
      task6
  ),
  task5
)
``` 

Expected result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_pipeline3_expected.png)

Actual result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_pipeline3.png)

It makes not much sense, because the above pipeline 
could be simply:

```javascript
const pipeline3 = join(
  task0,
  fork(task1, task3, task6),
  task5
)
```

However it should be possible to fork just after the fork. Therefore I added 
to the above code the following rational.

```javascript
if (forkee.info.props) {
  // this handles if fork is within fork without join wrapping
  // the inner fork
  if (outermostTasks.info !== upStreamTasks.info) {
    const newUpStreamTasks = taskCreationDispatcher(dispatch, upStreamTasks, task.info.uid, forkee.info.uid)
    const newOutermostTasks = taskCreationDispatcher(dispatch, outermostTasks, task.info.uid, forkee.info.uid)
    // notice that if not within a fork it creates a lineage
    // with the joint tasks plus the outermostTask with a new uid
    const lineage = [forkee].concat(newUpStreamTasks).concat(newOutermostTasks)
    lineageGetter(dispatch, lineage, joinages)
  } else { // if a fork is found within another fork
    // this avoids the duplication of the last task
    const newOutermostTasks = taskCreationDispatcher(dispatch, outermostTasks, task.info.uid, forkee.info.uid)
    // in the case of fork it simply adds to the forkee task the
    // outermost task
    // other orchestrators inside this structure will not render
    // this lineage
    const lineage = [forkee].concat(newOutermostTasks)
    lineageGetter(dispatch, lineage, joinages)
  }
}
```

Now, if I add a 3rd fork inside the _inner_ fork:

```javascript
const pipeline4 = join(
  task0,
  fork(
    join(
      task4,
      fork(
        join(
          task7,
          fork(
            task1,
            task3
          )
        ),
        task6
      )
    ),
    task2
  ),
  task5
)
```

Expected result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_pipeline4_expected.png)

Actual result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_pipeline4.png)

In this `pipeline4` the double circled tasks are represented in the actual 
pipeline that was ran but the other with no no double circle on them are not 
being properly executed.

Related commits: [d1a1d3d](https://github.com/bionode/bionode-watermill/commit/d1a1d3d3310eae60c2ccd1b9d6e7c91f753669fd)
and [904748a](https://github.com/bionode/bionode-watermill/commit/904748a6da194026ffb6e23dd62dd0ec304d89a5)

**Update 2**

Third forkception is now possible!

The `pipeline4` described above that should render the above represented expected 
result for this pipeline.

Now properly returns everything that is not an orchestrator inside the first 
_inner_ fork.

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_pipeline4_result_correct.png)

However it is still unknown at this stage if adding more complexity within 
fork will properly work. More tests will come soon...

* [x] Junction doesn't work inside _inner_ fork

   - The problem was that after the first fork, when `outermostTasks` are 
   stored it should be appended to `lineage`:
   
```javascript
// first create a new task for this instance of junction
const newOutermostTasks = taskCreationDispatcher(dispatch, outermostTasks, task.info.uid, forkee.info.uid)
// then add it to lineage
const lineage = [forkee].concat(newUpStreamTasks).concat(newOutermostTasks)
```

Check this [commit](https://github.com/bionode/bionode-watermill/commit/7089821df8ac4d6a76436797f872965cd3ae9b4c).

Test pipeline:

```javascript
const pipeline5 = join(
  task0,
  fork(
    join(
      task4,
      fork(
        join(
          task7,
          junction(
            task1,
            task3
          ),
          task6
        ),
        task6
      )
    ),
    task2
  ),
  task5
)
```

Expected result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_junction_expected.png)

Previous result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_junction_incomplete.png)

Actual result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_junction.png)

* [x] Add a fourth level of fork doesn't end the pipeline properly?

Yes, it works!

Test pipeline: 

```javascript
const pipeline6 = join(
  task0,
  fork(
    join(
      task4,
      fork(
        join(
          task7,
          fork(
            join(
              task1,
              fork(task3, task3_2)
            ),
            task8
          )
        ),
        task6
      )
    ),
    task2
  ),
  task5
)
```

Expected result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/4thlevel_fork_expected.png)

Actual result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/4thlevel_fork.png)

#### Orchestrators after fork

* [ ] fork followed by other orchestrators

This is currently not working because `taskCreationDispatcher` expects the 
array to have just tasks and not orchestrators.

For instance take the following pipeline:

```javascript
const pipeline7 = join(
  task0,
  fork(task4, task3),
  task5,
  fork(task1, task2),
  task6
)
```

Expected result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_after_fork.png)

**or**

```javascript
const pipeline8 = join(
  task0,
  fork(task4, task3),
  task5,
  junction(task1, task2),
  task6
)
```

Expected result:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/junction_after_fork.png)

Both these examples result in a single node for the initial `task0` (before 
`fork`). Although I have noticed that if within _inner_ forks `join`, 
`junction` and even other `fork`s work. So there must be away to parse it 
similarly in order to allow to duplicate this orchestrators when ran after 
fork.

#####The hack

A way to work around this issue is by duplicating the contents inside a fork,
 however this might be not very interesting for longer pipelines.

```javascript
// not working
const pipeline7 = join(
  task0,
  fork(task4, task3),
  task5,
  fork(task1, task2),
  task6
)

// previously not working... but now it is ok
const pipeline7_2 = join(
  task0,
  fork(join(task4,task5,fork(task1,task2)), join(task3,task5,fork(task1,task2))),
  task6
)

// not working
const pipeline8 = join(
  task0,
  fork(task4, task3),
  task5,
  junction(task1, task2),
  task6
)

// working
const pipeline8_2 = join(
  task0,
  fork(join(task4, task5, junction(task1, task2)), join(task3,task5,junction(task1, task2))),
  //task5,
  //junction(task1, task2),
  task6
)
```

However, this approach did not work for `fork` because it needed to 
properly generate `uid` for `task6` (in the example above). Previously, when 
tasks are the same and 
fork instance the same it could not render new `uid`s. Hence, `fork` needs 
to know `downStreamTasks` (which should be upstream in fact...) and generate 
a unique `uid` that can be passed to `taskCreationDispatcher` function as an 
argument for duplicating tasks, resulting in a different `uid`. 

```javascript
// gets downStreamTasks
const downStreamTasks = tasks.slice(0, i)
// puts the array of uids into a unique uid for each downStreamTasks
const downStreamUids = downStreamTasks.map(
  t => t.info.uid
).reduce((a, b) => hash(a + b), 0)
```

Now, let's see the results with the workaround (which in fact was cool for 
debugging an additional issue):

Result with junction:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_junction_duplicated.png)

Result with fork:

![](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_duplicated_tasks.png)

### Tests for 'orchestration'
