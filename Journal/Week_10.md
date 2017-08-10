# Week 10 (3 August to 9 August)

- [Summary](#summary)
- [Progress](#progress)
    - [Fork refactoring](#fork-refactoring)
        - [Fork inside fork](#fork-inside-fork)
            - [Going 'wild' with fork](#going-wild-with-fork)
        - [Junction inside fork](#junction-inside-fork)
        - [Join inside fork](#join-inside-fork)
    - [Orchestrators uid](#orchestrator-uid)
    - [Merkle tree like task uid](#merkle-tree-like-task-uid)
- [What is missing?](#what-is-missing)
        

## Summary

In [last week issue with fork](https://github.com/bionode/GSoC17/blob/master/Journal/Week_9.md#fork-within-fork)
we have characterized all the limitations of each orchestrator and what 
should be done to solve it. Some orchestrators properly worker with each 
other but fork had several limitations, since it didn't permit to have a fork 
inside fork.


## Progress

### Fork refactoring

For instance consider the following pipeline:

```javascript
const pipeline = join(
  task0, 
  fork(   // outer fork
    join(
      task4, 
      fork(   // inner fork
        task1, 
        task3
      )
    ), 
    task2
  ), 
  task5
)
```

Here you can see that this pipeline has two forks, one inside the other. This
 pipeline should render a pipeline equal to the one described in last week:
 
 ![test](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/index.png)

Here I will use `outer fork` to refer to the first fork and `inner fork` for 
the second fork that is inside the first fork.

The issue with fork is that current api doesn't allow for elements inside 
fork to be an array of tasks rather than a task. So in order to deal with 
this I had to add a statement to check (when fork is found) if it has just 
tasks inside:

```javascript
fork(task1, task2)
```

or if it has some more complex structure like:

```javascript
fork(task1, fork(task2, task3))
```

or 

```javascript
fork(task1, junction(task2, task3))
```

or even

```javascript
fork(task1, join(task2, task3))
```

Off course these examples are pseudocode and will not work on real pipelines 
because one 
cannot fork without first having something to fork. But the important thing 
here is that orchestrators struggle to work inside fork, although junction 
and join seemed to work without adding code to the current fork implementation.

However fork requires the pipeline to be able to duplicate tasks with different 
`uids`, otherwise they will stop and not run the pipeline. 

So, I started working on a hack for this in order to make every orchestrator 
work inside a fork.

First of all, fork has to distinguish whether it is dealing with simple 
tasks or with orchestrators and the difference between both is that each 
element of fork

```javascript
task.info.tasks.forEach((forkee, j) //{ ... }
```

can be an array and if so it is an orchestrator (`join`, `junction`, `fork`) 
or a `taskInvocator` function (in which case it has only a `task`).

When it is an array of tasks (in the case of an orchestrator is found):

```javascript
if (forkee.info.tasks) //{ ... }
  // note that this will not exist if it is not an array of `taskInvocator`s
```

This will render an array of `taskInvocator` functions but it may also 
contain other orchestrators like the pipeline described above, where the 
`join` after the `outer fork` has an array of length 2 with `task4` and `fork(task1, task3)`

Or the element inside fork can be a task, in which case it just has to 
duplicate the downstreams tasks twice. For the outer fork downstream tasks 
are defined as `restTasks` which basically `slice` the array of tasks inside 
`outer fork` to construct a new array with all tasks downstream from the 
current one (`i + 1`) ([code](https://github.com/bionode/bionode-watermill/pull/70/files#diff-60c3fb460a0980c2b661e199742c79ceR38)). Then, this downstream task or tasks are 
duplicated and 
a new `lineage` is added to the workflow, which in fact duplicates 
the pipeline from now on [code](https://github.com/bionode/bionode-watermill/pull/70/files#diff-60c3fb460a0980c2b661e199742c79ceR42).

However until now we were just handling first for instance, so how exactly 
will it work inside for other orchestrators inside fork.

#### Fork inside fork

The first orchestrator that we will approach is in fact the most difficult 
but since it was the main cause for this refactoring.

When a `fork` is found inside another an _outer_ `fork` the _outer_ downstream 
tasks (`newRestTasks`) should be duplicated also (`outerRestTasks`) as well as 
the upstream 
and 
downstream tasks of the _inner_ fork (`newUpTasks` and `newDownTasks`, 
respectively). Then a `lineage` (a branch of the workflow) would be:

`inner downstream tasks + current task + inner upstream tasks + outer 
upstream tasks duplicated` 

which translated to code is:

```javascript
const lineage = newDownTasks.concat([subForkee]).concat(newUpTasks).concat(outerRestTasks)
```

Then a lineage is `dispatch`ed using `lineageGetter` function (check [code](https://github.com/bionode/bionode-watermill/pull/70/files#diff-60c3fb460a0980c2b661e199742c79ceR173)).

Results:

```javascript
const pipeline = join(
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

 ![test](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_solution.png)


##### Going 'wild' with fork

going even further:

```javascript
const pipeline = join(
  task0,
  fork(
    join(
      task4,
      fork(
        task1,
        fork(task3,task6)
      )
    ),
    task2
  ),
  task5
)
```

 ![test](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_fork_fork_example.png)


#### Junction inside fork

The second orchestrator that I dealt with was junction because it implies 
some level of duplication inside fork but upstream tasks for _outer_ `fork` 
should not be duplicated inside `junction` (as occurs for `fork`). In turn, 
pipeline should in fact be merged after junction rather than duplicating 
tasks but keep in mind that _outer_ `fork` should still end in two or more 
branches on this pipeline.

Consider following example pipeline:

```javascript
const pipeline = join(
  task0,
  fork(
    join(
      task1,
      junction(
        task4,
        task5
      ),
      task6
    ),
    task2
  ),
  task3
)
```

Notice how junction is inside fork: wrapped in a `join`. In fact, we can 
dismiss in this case `task1` but `task6` after `junction` and wrapped in 
`join` is essential for join to finish otherwise it won't have a task to 
gather the results from both branches. This is utterly important because 
`task3` would expect the same `resolvedOutput` as `task2`, so there should be
 some kind of uniformization in `task6` before reaching `task3`. In fact the 
 same is true for a simple fork like this:
 
 ```javascript
const pipeline = join(task0, fork(task1, task2), task3, task4)
```

These considerations are important to define what should be the final product
 of a `junction` inside a `fork`. Therefore, when a `junction` is one of the 
 elements found inside the `fork` instance fork should save a branch like the
  following:
  
  `inner downstream tasks + current task + inner upstream tasks + outer 
   upstream tasks but without duplication`
   
   which translated to code would be:
   
   ```javascript
const lineage = downStreamTasks.concat([subTask]).concat(upStreamTasks).concat(newRestTasks)
```

Notice the difference with fork in which the last `concat` is `newRestTasks` 
and not `outerRestTasks`. This way, _outer_ upstream tasks are only 
duplicated once (for the _outer_ `fork` but not for `junction`). Besides that
 it works quite similar to fork, i.e., has to make a new `lineage` for the 
 junction branch in which the downstream and upstream tasks within the 
 `fork` are added to current task (in this case the `junction` itself). 
 Taking the above pipeline as an example:
 
 We add to store two branches for the fork
 
 1) `task0` + `task2` + `task3`
 2) `task0` + `task1` + (`task4`, `task5`) (in parallel) + junction vertex + 
 `task6` + `task3`
 
 In branch 2 (the one with the `junction`): 
 
 * `task1` is an _inner_ upstream task (`upStreamTasks`).
 * `task4` and `task5` are `subTask` (the `junction` itself).
 * `task6` is an _inner_ downstream task (`downStreamTasks`).
 * `task3` is an _outer_ downstream task (`newRestTasks`)

result:

 ![test](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_fork/fork_junction.png)

#### Join inside fork

Though this case seems simple it still needed to be properly ruled otherwise 
it won't render any result for the branch with `fork(join(join()))`. Although
 this is an odd behavior because `join(join())` === `join()` this should be 
 handled.
 
 So I used same procedure as for `junction` where only the tasks after the 
 `fork` needed to be duplicated.
  So, if we have:
  
  ```javascript
join(task0, fork(task1, join(task2, task3)), task5)
```

Just needs to duplicate `task5` once, because there are only two elements inside
 the fork (`task1` and `join(task2, task3)`).
 
 Example pipeline:
 
 ```javascript
const pipeline = join(
  task0,
  fork(
    join(
      task4,
      join(
        task1,
        task3,task6
      )
    ),
    task2
  ),
  task5
)
```
 
 Resulting pipeline

 ![test](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/fork_join/fork_join.png)

**IMPORTANT:** This will only work when junction and fork are wrapped 
inside a `join()` (that is inside a `fork`). Also it will not work if we had 
multiple more than 3 level of forks.

### Orchestrators uid

Previously, only tasks had uids which was making it difficult for refactor 
fork,  
since many constraints had to be accounted depending on what is inside a fork.

For example inside a fork we may have:

```javascript
fork(task1, task2)
```

or something more complex:

```javascript
fork( join(task1, task2), join(task3, task4) )
```

Until now branches `uid` was being generated by a number that increased in a 
`forEach` loop for each branch of the fork. Despite this approach seem to 
work it is sketchy and makes it difficult to handle differently tasks and 
orchestrators. 

Therefore I started by adding an `uid` for each orchestrators 
that is a combined uid of: 

```javascript
uid: hash(orchestratorUidGenerator(tasks).join(''))
```

where `orchestratorUidGenerator` is a [module](https://github.com/bionode/bionode-watermill/blob/orchestrators_tests/lib/orchestrators/orchestrator-uid-generator.js) that is imported by `join.js`, 
`junction.js` and `fork.js`, and which basically gathers all uids for all 
tasks inside these orchestrators and returns a new unique id for each 
orchestrator:

```javascript
const orchestratorUidGenerator = (tasks) => {
  return tasks.map(task => task.info.uid)
}
```

Then, we can use this new structure to generate unique ids for fork branches 
depending on: tasks after fork `uids` + fork `uid` + task or 
orchestrator inside fork `uid`:

```javascript
hash(t.info.uid + forkUid + forkeeUid)
```

This will render a unique id for all task after fork, when we use 
`taskCreationDispatcher` in `./lib/join.js`:

```javascript
const newUpStreamTasks = taskCreationDispatcher(dispatch, upStreamTasks, task.info.uid, forkee.info.uid)
```

### Merkle tree like task uid

Following one brainstorming that we had during our weekly meetings tasks 
could have a merkle tree like uid generation, where child vertices inherit 
parents parameters (in this case `uid`)

So, here a task `uid` should be generated from is parents `uids`. Parents 
`uids` are stored in `task.context.trajectory` and is basically an array of 
`uids`, so the idea was to gather them and create a new `uid` (`newUid`) and 
sum to this `uids` the static task `uid` that is created before the execution
 of the pipeline and that consequently does not have `trajectory` before 
 running the pipeline.
 
 ```javascript
const childHashesToUid = (context, uid) => {
  if (context.trajectory !== undefined) {
    const childHashes = Object.keys(context).reduce(
      (sum, key) => Object.assign({}, sum, {[key]: hash(stringify(context[key]))}),
      {}
    )
    uid = hash(childHashes.trajectory + uid)
    return uid
  }
```

Then was just a matter of executing this function before task dispatching in 
`task.js`.

```javascript
const newUid = childHashesToUid(context, uid)
```

And then task creation is dispatched (check it [here](https://github.com/bionode/bionode-watermill/blob/orchestrators_tests/lib/task.js#L63)).

## What is missing?

* [ ] fork work within fork - maybe `newUpStreamTasks` isn't creating 
unique ids for forks within forks which makes tasks after the first fork 
rendering just one instance of that task and not properly duplicating or for 
each task within a fork (if another fork is found inside fork - enough of 
forkception for now!).
