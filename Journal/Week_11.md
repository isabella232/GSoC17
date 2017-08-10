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

### Tests for 'orchestration'