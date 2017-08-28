# GSoC17 Final Report

- [Summary](#summary)
- [Notebook week reports](#notebook-week-reports)
- [Merged pull requests](#merged-pull-requests)
- [What was not done and why](#what-was-not-done-and-why)
- [What needs to be done yet](#what-needs-to-be-done-yet)

 ---
 
## Summary

This is my final report for the Google Summer of Code aka GSoC. I started 
this GSoC by first getting a feel from the user perspective of 
bionode-watermill and as the time passed (and I was starting to get more 
comfortable with the code),
 I start learning how the API works and how it can be improved. The following
  [reports]((#notebook-week-reports)) clarify the progress that was made and have a detailed explanation 
  on why and how modifications were being added to the previous API.
  
  Below, I also made a list of all pull requests made, and a list of issues 
  still needs work to improve current bionode-watermill API.
 
 
## Notebook week reports

During my GSoC progress was tracked in weekly reports. Here you can find the 
12 reports:

* [Week 1](https://github.com/bionode/GSoC17/blob/master/Journal/Week_1.md)
* [Week 2](https://github.com/bionode/GSoC17/blob/master/Journal/Week_2.md)
* [Week 3](https://github.com/bionode/GSoC17/blob/master/Journal/Week_3.md)
* [Week 4](https://github.com/bionode/GSoC17/blob/master/Journal/Week_4.md)
* [Week 5](https://github.com/bionode/GSoC17/blob/master/Journal/Week_5.md)
* [Week 6](https://github.com/bionode/GSoC17/blob/master/Journal/Week_6.md)
* [Week 7](https://github.com/bionode/GSoC17/blob/master/Journal/Week_7.md)
* [Week 8](https://github.com/bionode/GSoC17/blob/master/Journal/Week_8.md)
* [Week 9](https://github.com/bionode/GSoC17/blob/master/Journal/Week_9.md)
* [Week 10](https://github.com/bionode/GSoC17/blob/master/Journal/Week_10.md)
* [Week 11](https://github.com/bionode/GSoC17/blob/master/Journal/Week_11.md)
* [Week 12](https://github.com/bionode/GSoC17/blob/master/Journal/Week_12.md)

## Merged pull requests

* [PR #48](https://github.com/bionode/bionode-watermill/pull/48) - Correct 
path to home folder

PR #48 corrects pipelines path to properly get bionode-watermill lib, which 
was previously broken.

* [PR #53](https://github.com/bionode/bionode-watermill/pull/53) - Adds a new example pipeline and edits existing example pipeline

PR #53 added a new pipeline used in the [bionode-watermill tutorial](https://github.com/bionode/bionode-watermill-tutorial).
This is a pipeline that fetches some sequence data, as well as reference, and 
maps using two different mappers (bwa and bowtie2) and them performs some 
operations using samtools. The tutorial was something necessary in order to 
attract users and to get familiar with `bionode-watermill`. Also a [docker 
image](https://github.com/bionode/bionode-watermill-tutorial/tree/master/docker-watermill-tutorial) was added in order to experiment this pipeline.

Also, in the scope of this PR a simpler bionode-watermill tutorial was added 
to a new repo, called [bionode-watermill-tutorial](https://github.com/bionode/bionode-watermill-tutorial), which targets new users.
Then, this pipeline is, in fact, part of a more advanced tutorial, in which 
users understanding the basics of `bionode-watermill` can see in action what 
it can make.

* [PR #55](https://github.com/bionode/bionode-watermill/pull/55) -  Added a tutorial on how to run "two-mappers" example

This PR adds the more advanced tutorial referenced above, in which users are 
challenged to continue assembling the pipeline given their previous knowledge
 on the work of `bionode-watermill`. Also, a solution is provided for the 
 challenge in `pipeline_lazy.js`, which is described in the `REAMDE.md` 
 available in `bionode-watermill` tutorials or it can be checked [here](https://github.com/bionode/bionode-watermill/pull/55/files#diff-e8b3724490ab0997f4cee3789ecb5681).

* [PR #56](https://github.com/bionode/bionode-watermill/pull/56) - Updated docker related links and added link to tutorial in README

This PR updates main `bionode-watermill` README.md to include tutorial 
information.

* [PR #58](https://github.com/bionode/bionode-watermill/pull/58) - Fixed issue with logging of task name to stdout

This is a small PR that fixed a logging of the `task` name.

* [PR #59](https://github.com/bionode/bionode-watermill/pull/59) - Transferred experimental branch for development of graph

This PR adds a manifest file implementation that logs graph structure to a 
file and started implementing [graphson](https://github.com/tinkerpop/blueprints/wiki/GraphSON-Reader-and-Writer-Library) like structure to graph.

* [PR #61](https://github.com/bionode/bionode-watermill/pull/61) - Implemented redux-logger given an environmental variable

This PR adds `redux-logger` to be easier to debug issues regarding actions and 
states. Now we can use `REDUX_LOGGER=1 node pipeline.js` to log `redux` 
related actions and states to console. This is particularly cool with chrome 
nodejs V8 [inspector](https://chrome.google.com/webstore/detail/nodejs-v8-inspector-manag/gnhhdgbaldcilmgcpfddgdbkhjohddkj?hl=en). 

* [PR #62](https://github.com/bionode/bionode-watermill/pull/62) - Implemented a graph visualization

This PR added a `d3` visualization for pipeline shape in which `graphson` 
`vertices` are represented as d3 nodes and `graphson` `edges` are represented
 as links. This visualization is available when pipeline is executed and 
 final graphson visualization can be obtained at the end of the run. `d3` 
 visualization nodes also have some metadata regarding tasks definition and 
 execution, such as `uid`, `resolvedInput`, `resolvedOutput`, `name` and 
 `params`. Visualization used `socket.io` to emit data to `localhost:8084`.
 
 * [PR #64](https://github.com/bionode/bionode-watermill/pull/64) - Added operation command to graphson
 
 This PR added `operationString` to `d3` visualization.
 
 * [PR #67](https://github.com/bionode/bionode-watermill/pull/67) - Edits to 
 two-mappers pipeline and removed of redundant defaultTask
 
 This PR edited two-mappers pipeline to follow bionode-watermill philosophy 
 (removed pipes from commands passed to shell and removed hardcoded file 
 names from `input`). Also, removed duplicated definition of `defaultTask` 
 that was not being executed under `lib/reducers/task.js`.
 
 * [PR #70](https://github.com/bionode/bionode-watermill/pull/70) - Orchestrators refactoring and fork handling other orchestrators
 
 This PR handles orchestrators working inside other orchestrators, implements
  merkle tree like task `uid` generation which allows for tasks to be 
  duplicated within the scope of a pipeline. Orchestrators get a similar 
  structure with `uid`. Also a couple of new tests were added in order to 
  test pipeline shape given different combinations of orchestrators.
  
* [PR #72](https://github.com/bionode/bionode-watermill/pull/72) - Going deeper on fork orchestration

This PR increases the cases where `fork` work, because until this PR `fork` just 
worked with other level of `fork` (e.g. `fork(fork())`). Also some other 
examples were added on how to handle multiple inputs and concurrency between 
multiple inputs, which allow to have some control over CPU and memory resources.

* [PR #73](https://github.com/bionode/bionode-watermill/pull/73) - Add another test for fork within fork wrapped in join

This PR fixed a case where fork inside other forks nested in joins and adds 
tests for `travis` and `codecov`.

* [PR #76](https://github.com/bionode/bionode-watermill/pull/76) - Docs update

This PR updates current documentation in [bionode-watermill gitbook](https://bionode.gitbooks.io/bionode-watermill/content/)

## What was not done and why

Unfortunately time grew short and definition of directed acyclic graph (DAG) 
took longer than previously expected. For instance, complex combination of 
orchestrators were never previously tested and they needed to be properly 
ruled for the tool to be usable in the maximum number of possible use cases.

Moreover, the tool has to become usable by anyone that shows interest in 
joining our community and thus documentation was required, both updating the 
existing docs as well as adding new pipelines that are well documented (e.g. 
[two-mappers pipeline](https://github.com/bionode/bionode-watermill/tree/dev/examples/pipelines/two-mappers)) and 
[bionode-watermill tutorial](https://github.com/bionode/bionode-watermill-tutorial).
Before adding more complexity to the tool we now aimed to make it more 
accessible to users.

Therefore, there was no time to implement:

* Streams between tasks - this required that all orchestrators behaved more 
similarl to each other. Therefore a similar structure was made for the three 
orchestrators (as reported in [Week 9](https://github.com/bionode/GSoC17/blob/master/Journal/Week_9.md#consistency-of-junction-and-fork)).
Also fork needed to be more consistent before adding more complexity to the 
API, since in many example pipelines that I experimented in 
`examples/pipelines/tests` it did not worked has expected and thus some 
handling of fork rules had to be performed (check
[Week 9](https://github.com/bionode/GSoC17/blob/master/Journal/Week_9.md) 
and
[Week 10](https://github.com/bionode/GSoC17/blob/master/Journal/Week_10.md#what-is-missing) 
and
[Week 11](https://github.com/bionode/GSoC17/blob/master/Journal/Week_11.md) 
and
[Week 12](https://github.com/bionode/GSoC17/blob/master/Journal/Week_12.md) 
).

* Metrics to control the workflow - although this may have workarounds using 
bluebird module as 
exemplified in [Week 12 report](https://github.com/bionode/GSoC17/blob/master/Journal/Week_12.md#scheduling-inputs-into-pipeline).

* Improved validation - Although some validators already exist in current 
API, we could pass new custom validators that allow the user to contror, for 
example, file size and extension (Issue [#80](https://github.com/bionode/bionode-watermill/issues/80)). 

## What needs to be done yet

* [Issue #74](https://github.com/bionode/bionode-watermill/issues/74) - 
Refactor pipeline parsing

This issue is important in order to get pipeline shape before running the 
pipeline itself within bionode-watermill. We can then check pipeline proper 
execution using visualization, improve the visualization itself to show what 
have ran (green node), is running (yellow node), not run yet (blue node) and 
failed running (red node).

* [Issue #75](https://github.com/bionode/bionode-watermill/issues/75) - 
Refactor 
task definition

This issue suggests that task definition becomes more javascript independent 
and more user friendly. The ultimate goal would be to abstract the users from 
javascript at all when assembling their pipelines.

* [Issue #77](https://github.com/bionode/bionode-watermill/issues/77) - Improvements to visualization tool

This issue was added since current visualization tool is still a first 
implementation and may be further developed. Despite being already useful for
 debugging pipelines execution, it can be improved if we had success/failure 
 checks on tasks and input/output flow to the visualization.
 
* [Issue #78](https://github.com/bionode/bionode-watermill/issues/78) - Tasks as npm modules

This is a proposal for tasks being transformed into npm modules that require 
`bionode-watermill` module and that could be imported into new 
bionode-watermill pipelines.

* [Issue #79](https://github.com/bionode/bionode-watermill/issues/79) - Streaming Between Tasks

As referenced above this goal was delayed because of orchestrators control 
but this is a _must have feature_ for the project and thus we opened a new 
issue that addresses this topic.

* [Issue #80](https://github.com/bionode/bionode-watermill/issues/80) - Improve validators over input/output files

This issue opens a new discussion on how to pass custom file validators.

* [Issue #81](https://github.com/bionode/bionode-watermill/issues/81) - 
Through tasks not properly logging

This issue raises aweareness to a very experimental feature in 
bionode-watermill and that despite working well for a single task renders 
some odd behavior in logging to both `graphson.json` and for the d3 
visualization tool available at `localhost:8084`.

* [Issue #82](https://github.com/bionode/bionode-watermill/issues/82) - 
Execution of a pipeline following other pipeline

This was something that I was trying to accomplish at the very end of 
my GSoC but could not do. For now, I just know that files used in other 
pipelines 
that are `.then` run in the next pipeline seem to be unable to properly be 
executed in the downstream pipeline.
