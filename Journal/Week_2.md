# Week 2 (8 June to 14 June)

- [Summary](#summary)
- [Simple tutorial](#simple-tutorial)
- [Documenting two-mappers example](#documenting-two-mappers-example)
    - [tutorial](#tutorial)
    - [docker](#docker)
- [What can be improved?](#what-can-be-improved)

## Summary

In this week I took advantage of the acquired 'know-how' from last week, while
 playing with bionode-watermill (resulting in the assemblage of an example 
 pipeline), and starting making a tutorial for beginners. Current [README](https://github.com/bionode/bionode-watermill/blob/master/README.md) of 
 bionode-watermill can be too technical and hard for bioinformaticians that 
 have interest in learning how to use it (as it was my case...). Thus, I 
 decided to make a simple tutorial explaining everything that I first found 
 hard to understand when I tried to assemble my first pipeline. Then, I 
 documented the previous pipeline that I made and extended it to exemplify a 
 fully functional bioinformatics pipeline. Although, bionode-watermill was 
 developed with the intend of helping bioinformaticians to control the 
 workflow within their pipelines, it can have several other usages as it is 
 clear by [@thejmazz](https://github.com/thejmazz) [blog](https://jmazz.me/blog/NGS-Workflows) explanation.
 
 ## Simple tutorial
 
 In this tutorial I intended to demonstrate to anyone that knows a little of 
 _shell_ and no javascript or little javascript prior knowledge how they can 
 make simple **tasks** and use bionode-watermill orchestrators (**join**, 
 **junction** and **fork**). Therefore, I made two tutorials: one using 
 [javascript stanrdard style](https://github.com/bionode/bionode-watermill-tutorial/tree/master/js_standard_tutorial); 
 and another one using [ES6 syntax](https://github.com/bionode/bionode-watermill-tutorial) for more 
 advanced users or users whiling to know a little more on javascript (JS). The 
 default tutorial is ES6, but in the first task it explains how it can be 
 done with both JS standard and ES6 syntax and has a link at the under 
 [useful link](https://github.com/bionode/bionode-watermill-tutorial#useful-links) 
 to js standard tutorial.
 
 ## Documenting two-mappers example
 
 Pull requests: [#55](https://github.com/bionode/bionode-watermill/pull/55/files) and
 [#56](https://github.com/bionode/bionode-watermill/pull/56/files).
 
 ### Tutorial
 
But then I thought: "Ok users can now create files and write to them (with 
`touch` and `echo`), but how will this help someone to understand how to 
assemble their bioinformatics pipeline?". But then I thought that an easy way 
to get someone to understand how to use their own pipeline would be by 
documenting how we have developed one of the example pipelines. 

So, I documented how I assembled the [two-mappers](https://github.com/bionode/bionode-watermill/tree/master/examples/pipelines/two-mappers) 
pipeline, by explaining each step that I made to create the tasks and the 
workflow between them (with all orchestratos included). Then at the end of 
this tutorial I proposed a challenge for the user to experiment continuing 
this pipeline. 

The idea with this tutorial was not to provide the user with all the answers 
right ahead but rather constitute a first challenge to check if someone can 
in fact use tasks to continue this pipeline. Of course a solution was 
provided at the end of the tutorial for those that may struggle to understand
 how to continue the pipeline.
 
 ### Docker
 
 Since pipelines tend to have a lot of dependencies that can be hard to 
 install or configure, I made a docker image with all the necessary tools to 
 run the **two-mappers** pipeline. This docker image is available in 
 [bionode-watermill-tutorial/docker-watermill-tutorial](https://github.com/bionode/bionode-watermill-tutorial/tree/master/docker-watermill-tutorial)
 and also available in [docker hub](https://hub.docker.com/r/tiagofilipe12/bionode-watermill-tutorial/).
 
 Users that wish to try this pipeline may download this image and try it 
 locally, if they have docker installed in their systems, or they can try on 
 [play-with-docker](http://labs.play-with-docker.com/). All instructions are 
 documented in both git hub and docker hub.
 
 ## What can be improved?
 
 For now, three main things must be improved before attracting users to 
 bionode-watermill:
 
 - [x] Make it easier to understand which folders are being created from which 
 tasks.
 - [x] Better explain the input/output in the tutorial (see this [issue]
 (https://github.com/bionode/bionode-watermill-tutorial/issues/2)).
 I.e. explain how glob patterns are fetching files and how can we declare the
  output.
 - [x] Arrange a way for a task to take several inputs (lets say several _*
 .fas_ 
 files) and create a single output, since this is the case of many 
 bioinformatics pipelines. Of course, this should also be documented and 
 further explained (check [Week 11](https://github.com/bionode/GSoC17/blob/master/Journal/Week_11.md#multiple-input-handling)). 
 