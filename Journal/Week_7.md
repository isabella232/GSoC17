# Week 7 (13 July to 20 July)

- [Summary](#summary)
- [Progress](#progress)
    - [Optional tasks](#optional-tasks)
        - [The comment workaround](#the-comment-workaround)
        - [Using nodejs process.argv](using-nodejs-processargv)
    - [Execute pipelines inside another pipeline as a task](#execute-pipelines-inside-another-pipeline-as-a-task)
- [Removal of redundant code](#removal-of-redundant-code)

## Summary

This week was dedicated to the analysis of DAG and to explore solutions for 
problems that arise from previous conceptual discussions, such as: optional 
tasks, multiple input handling and the execution of a pipeline inside another
 pipeline.
 
## Progress

### Optional tasks

#### The comment workaround

Currently optional tasks have to be declared each time the pipeline has to be
 run, i.e. one may comment a given task in the script that exectutes the 
 pipeline. 
 
 ```javascript
const pipeline = join(
  task1,
  task2,
  task3
)
```

This will render the following workflow:

`task1 --> task2 --> task3`

but if we comment task2:

 ```javascript
const pipeline = join(
  task1,
  //task2,    //notice that I commented the comma
  task3
)
```

Thiw will render the following workflow:

`task1 --> task3`
 
 This will actually work but is somehow sketchy.
 
 
 #### Using nodejs process.argv
 
 Node.js already has the option to pass optional args to the command in shell
 . For instance if we write something in shell: `node pipeline.js arg1 arg2`,
we end up with the following array available in `process.argv`:

```javascript
[
  node,
  pipeline.js,
  arg1,
  arg2
]
```
And this can be used to construct alternative pipelines without adding more 
functions to `bionode-watermill` api.

For instance, the user can define two pipelines in its script:

```javascript
const pipeline1 = join(task1, task2, task3)

const pipeline2 = join(task1, task3)

/* then an if statement that checks the presence of args could be like the 
following*/

if (process.argv[2] === 'optional') {
  // executes the optional task
  pipeline1()
} else {
  // executes the standard pipeline
  pipeline2()
}
```

### Execute pipelines inside another pipeline as a task

Another solution for this would be to execute a pipeline inside another 
pipeline. For instance imagine you want to execute `alterativePipeline.js` 
after `mainPipeline.js`.

`mainPipeline.js`:
```javascript
const pipeline = join(task1, task2)
```

`alternativePipeline.js`:
```javascript
const pipeline = join(mainPipelineTask, task3, task4)
```

Here `mainPipelineTask` is in fact a task that executes `node mainPipeline.js`

**Note**: For this `mainPipeline.js` must be in the same folder as 
`alternativePipeline.js`, otherwise bionode-watermill will search for folders 
that do not exist because they are in the folder of ` mainPipeline.js`.

** However this didn't work as expected since I could not invoke a task from 
other pipeline into another task in main pipeline simply by executing `node 
pipeline.js` within a task.**

However, this should work because it is just the execution of a js script 
using node in shell to run it. And so, this issue requires further discussion
 to check what is happening...
 
 Nonetheless, pipelines should ideally be imported into another pipelines 
 instead of executing them in a main pipeline task, because the latter won't 
 allow for future implementation of full DAG visualization and instead will 
 render a node for the task that executes another watermill pipeline.

## Removal of redundant code

Also there existed two instances of `defaultTask` definition. One in 
`lib/reducers/task.js` that was being used only in this instance and another 
one in `lib/constants/default-task-state.js` that is used by orchestrators to
 handle `task` state. So, in order to avoid this duplication of code the 
 instances of `defaultTask` within `lib/reducers/task.js` was replaced with 
 an import from `lib/constants/default-task-state.js`.
 Check pull request [#67](https://github.com/bionode/bionode-watermill/pull/67/files)