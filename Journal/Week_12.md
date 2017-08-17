# Week 12 (17 August to 28 August)

## Summary

## Progress

### Scheduling inputs into pipeline

Last week I have managed to establish an example pipeline that handled 
several inputs and use one at a time (see [here](https://github.com/bionode/GSoC17/blob/master/Journal/Week_11.md#between-tasks)). 
However, this approach can exhaust all available resources in a given 
computer/server if too many files are given as input. Therefore it urged to 
characterize a simple way in which anyone can limit this I/O process.

One way to do this is by using `bluebird` npm module.

```javascript
// calls bluebird
const Promise = require('bluebird')
// makes concurrency the third argument of code in shell
const concurrency = parseFloat(process.argv[2] || "Infinity")

// [...] pipeline stuff
// Then after pipeline definition...

// checks for files in current working directory (where the script is executed)
fs.readdir(process.cwd(), (err, files) => {
  const desiredFiles = []
  files.forEach(infile => {
    // checks file extension, in this case '.fas' and creates an array with
    // just these files
    if (path.extname(infile) === '.fas') {
      desiredFiles.push(infile)
    }
  })
  // bluebird Promise.map is used here for concurrency where we can
  // limit the flow of files into the pipeline. So, if concurrency is set
  // to 1 it will first execute the first file, then the second and so on.
  // But if 2 it will execute the first 2 files and then the other 2.
  return Promise.map(desiredFiles, (infile) => {
    const pipelineMaster = fileFlow(infile)
    return pipelineMaster()
  }, {concurrency}) // in ES6 there is no need to repeat {concurrency: concurrency}
})
```

With this approach users still have to take in account programs cpu usage 
within each task and then see how many files can be processed at a time. 
Users should avoid having more processes fired by bionode-watermill than the 
available cpu usage.

Some examples (imagining a hypothetical server with 20 cpus available):

* 10 files and one of the tasks (the one that uses more cpus) in the pipeline 
use 5 threads.
    * only 4 files should be processed at the same time, therefore 
    `concurrency` should be **4** (4*5 = 20).

* 10 files and one of the tasks (the one that uses more cpus) in the pipeline
 use 2 threads.
    * The 10 files can processed all at once without limiting concurrency 
    (10*2 = 20).
    
* 1000 files and all tasks use one cpu.
    * batches of 20 files should be passed because the limit would be 20 cpus
     and you can have 20 files running same tasks at the same time.
     
Also, this concurrency parameter can be passed through `process.argv` instead
 of hardcoding it inside the script.js.s



