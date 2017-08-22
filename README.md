# GSoC17 Final Report

* [x] links to each nodebook entry
* [ ] links to merged PRs
* [ ] links to merged PRs in bionode-watermill docs
* [ ] what wasn't done and why
* [ ] what needs to be done yet
* [ ] link to issues

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

## Merged Pull Requests (PRs) and brief explanation

* [PR #48](https://github.com/bionode/bionode-watermill/pull/48) - correct path to home folder

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

* [PR #59](https://github.com/bionode/bionode-watermill/pull/59) - Transfered experimental branch for development of graph

To be continued...

## What wasn't done and why

## What needs to be done yet

## Issues