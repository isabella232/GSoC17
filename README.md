# GSoC Bionode 2017

[![Join the chat at https://gitter.im/bionode/GSoC17](https://badges.gitter.im/bionode/GSoC17.svg)](https://gitter.im/bionode/GSoC17?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

This repository is the detailed description of the of the work performed 
during my GSoC17 in bionode-watermill.
 
Here you may find all kind of code tests, reports and useful 
documentation that I found useful for my GSoC project 2017.

## Progress tracking

- [x] Bionode-watermill testing & mozzila sprint ([30 May to 7 June](https://github.com/bionode/GSoC17/blob/master/Journal/Week_1.md))
- [x] Bionode-watermill tutorial ([7 June to 14 June](https://github.com/bionode/GSoC17/blob/master/Journal/Week_2.md))
- [x] Basic Manifest files ([15 June to 22 June](https://github.com/bionode/GSoC17/blob/master/Journal/Week_3.md))
- [x] DAG visualization ([23 June to 28 June](https://github.com/bionode/GSoC17/blob/master/Journal/Week_4.md))
- [x] Improve DAG visualization as described 
[here (29 June to 05 July)](https://github.com/bionode/GSoC17/blob/master/Journal/Week_5.md#todo).
    - [x] Real-time graph visualization.
    - [x] Manifest file improvement (this might be solved with DAG 
    visualization)
- [x] Start studying and documenting the graph generation of orchestrators. ([21 July to 26 July](https://github.com/bionode/GSoC17/blob/master/Journal/Week_8.md#week-8-21-july-to-26-july)
and [27 July yo 2 August](https://github.com/bionode/GSoC17/blob/master/Journal/Week_9.md))
- [ ] Folder naming according to the definition of tasks? (needs discussion)
- [ ] Incremental mode (useful for debugging).
- [ ] Benchmark bionode-watermill comparison ([check this issue](https://github.com/bionode/GSoC17/issues/3)).
- [ ] Implement streams.

## Challenges

* [x] Optional tasks (Comment lines, pass an option to task itself and still be 
able to import the task and use it, multiple pipelines). Check [13 July to 20 
July](https://github.com/bionode/GSoC17/blob/master/Journal/Week_7.md#optional-tasks).
* [ ] Object pipeline definition (try to change the pipeline from a promise 
to an
 object.. and ability to import the pipeline into another pipeline, know dag 
 shape ahead of running). Check [13 July to 20 
July](https://github.com/bionode/GSoC17/blob/master/Journal/Week_7.md#execute-pipelines-inside-another-pipeline-as-a-task) and 
[21 July to 26 July](https://github.com/bionode/GSoC17/blob/master/Journal/Week_8.md#execute-pipelines-inside-another-pipeline-as-a-task-cont).
* [ ] CWL
* [ ] use cases for streaming tasks (a | b | c), saving stdout to files, etc
* [ ] Human input in certain tasks
* [ ] Improve graph visualization to see in real-time which files already run
 and
 where it is (marking it as green/red for instance whether or not task is done)
* [x] Ad hoc multi (input) dataset implementation --> each one runs in its own 
dag (first implementation).

## TODO

 - [x] Arrange a way for a task to take several inputs (lets say several _*
 .fas_ 
 - [ ] We need to check bionode-ncbi behavior within bionode-watermill since its 
 outputs are being generated outside `./data`, the expected main output 
 directory.
 **Update**.
 This can be now handled using `object.dir` on each task function (check [this
  topic](https://github.com/bionode/GSoC17/blob/master/Journal/Week_11.md#quick-fix-on-example-pipeline)).
 - [ ] replace d3 graph visualization with some more appropriate model.
 - [ ] control graph using a CLI, rather than being executed by default.
 - [ ] Future implementation of logger must be passed through a CLI as 
 described 
 [here](https://github.com/bionode/bionode-watermill/issues/31). For now, this
 is a quick fix for debugging purposes. Later, when we implement a CLI we have
  to have this into account.


