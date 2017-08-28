# Week 8 (21 July to 26 July)

- [Summary](#summary)
- [Progress](#progress)
    - [Execute pipelines inside another pipeline as a task (cont...)](#execute-pipelines-inside-another-pipeline-as-a-task-cont)
    - [Graph arrows](#graph-arrows)
    - [Simple DAG examples](#simple-dag-examples)
        - [join](#join)
        - [junction](#junction)
        - [fork](#fork)
- [Pending](#pending)

## Summary

This week was fully dedicated to the characterization of DAG and test of 
orchestrators in diverse cases.

## Progress

### Execute pipelines inside another pipeline as a task (cont...)

Attempted to import the promise of a pipeline from another `pipeline.js`:

```javascript
const testPipe = require('./test.js')
```

`testPipe` is the exported pipeline (promise) from `test.js`.

then created a task:

```javascript
const importedTask = task({}, () => testPipe())
```

and added it to the end of another pipeline as a normal task.

Pipeline executes properly, a new vertex and edge is created but then it 
generates no output folder as well as information from the tasks being 
executed within `test.js` (this latter is an expected behavior).

Export/import is definitely the way to go but there is still work to do 
before this being more user-friendly and intuitive. For instance, by converting 
the pipeline into an object rather than a set of promises.

### Graph arrows

Currently, graph arrows properly show the direction of workflow between 
nodes, however sometimes might be interesting to know where the inputs are 
coming from. Right now edges are being outputted to `graphson.json` according
 to the workflow only, but maybe it would be nice to add some kind of edge 
 type on each edge in `edges` array, which can then be parsed to generate a 
 workflow arrow or a I/O arrow, adding at the same time new edges to the 
 graph where I/O is controlled. Check this out:
 
 ![this](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/join_only/join_only.png)
 
 In the above image you can see a join of 5 tasks, however final 5th task is 
 in fact the collection of all the outputs from tasks 1, 2, 3 and 4 as the 
 inputs of task 5, however looking at the graph you can only see that task 5 
 executes after task 4. This is, in fact, correct but not very handsome! The 
 user will be interested in knowing where the inputs of each tasks are coming
  from. Maybe we could add a green arrow showing I/O connections. If you are 
  interested, you can check the pipeline
  [here](https://github.com/bionode/GSoC17/blob/master/Experimental_code/Experimental_Pipelines/join_only/join_only.js).
  
  ### Simple DAG examples
  
  #### join
  
  Join is in fact a pipe that waits for previous task to finish before 
  executing the next one. Check [this](https://github.com/bionode/GSoC17/tree/master/Experimental_code/Experimental_Pipelines/join_graph)!
  
  #### junction
  
  junction on the other hand can be seen as a parallelization of tasks that 
  waits for all tasks inside junction to finish before proceeding with the 
  pipeline. Check [this](https://github.com/bionode/GSoC17/tree/master/Experimental_code/Experimental_Pipelines/junction_graph)!
  
  #### fork
  
  fork is somewhat similar to junction in the sense that both split the 
  pipeline into two or more branches, but the end result of them is very 
  different. While junction results in a final tree with one leaf, fork 
  splits the pipeline into two or more branches that end in as many leaves as
   the number of branches. Check [this](https://github.com/bionode/GSoC17/tree/master/Experimental_code/Experimental_Pipelines/fork_graph)!
  
  Also, with fork tasks proceed regardless of any branch being finished or 
  not, thus some branches may finish before others since they don't depend on
   each other.
  
  This, in fact, has profound effects in pipeline design. That should be well 
  documented, because after fork we cannot merge the results coming from 
  different branches.
  
  ## Pending
  
  * [x] Test junction inside a junction - other solution (works but check 
  [this](https://github.com/bionode/GSoC17/blob/master/Journal/Week_9.md#junction-inside-junction))
  * [x] Test fork inside a fork (check [here](https://github.com/bionode/GSoC17/blob/master/Journal/Week_9.md#fork-within-fork))
  * [x] Test Junction inside a fork
  