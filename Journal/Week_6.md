# Week 6 (6 July to 12 July)

- [Summary](#summary)
- [Progress](#progress)
    - [resolvedInput](#resolvedinput)
    - [operationString](#operationstring)
- [Pending](#pending)

## Summary

This week we set the goals to finish the pending issues with graph 
visualization and output file (`graphson.json`), namely the obtaining of 
`resolvedInput` and `operationString`. Also, we started studying the 
implementation of the graph for many orchestrators of watermill, i.e., how 
they behave under different conditions. For that the development of new 
pipelines (commonly used in bioinformatics) might help us clarify if the 
behavior of orchestrators is what we expect.

## Progress

First I implemented a way to log redux actions and states to the 
console (or chrome inspector, with `--inspect`) with `redux-logger`. For that
 I added:
 
 ```javascript
const createLogger = require('redux-logger')
const logger = createLogger()
```
changed the middleware currently being passed to createStore() to accept 
logger if a given environmental variable is called:

```javascript
// default middlewares
let middlewares = [thunk, sagaMW]

// if Environmental variable REDUX_LOGGER is 1 then...
const loggerEnv = process.env.REDUX_LOGGER
// for now call REDUX_LOGGER=1 before executing a pipeline
// REDUX_LOGGER=1 node pipeline.js
if (loggerEnv === '1') {
  middlewares.push(logger)
}
  
const store = createStore(rootReducer, applyMiddleware(...middlewares))
```

This way we can call `REDUX_LOGGER=1 node pipeline.js` and this will output 
all redux related actions and states to the console. Also, one can use chrome
 inspector (`REDUX_LOGGER=1 node --inspect pipeline.js`) and open chrome 
 browser to have a better visualization of the output. Check the pull request
  [here](https://github.com/bionode/bionode-watermill/pull/61/files).
 
After this implementation, found that three errors were being outputted:

1)
```
(node:16356) DeprecationWarning: Calling an asynchronous function without callback is deprecated.
```
2)
```
(node:16356) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 connection listeners added. Use emitter.setMaxListeners() to increase limit
```
3)
```
Unhandled rejection (<{"threads":1,"container":null,"resume"...>, no stack trace)
```

For the first one, I removed previous manifest file implementation on 
`lib/sagas/lifecycle.js` and replaced writeFile() in `collection.js` with 
writeFileSync().
The second one, was caused by the default setMaxListeners() behavior of 
sockets.io and then I just had to set 
```javascript
io.sockets.setMaxListeners(0)
```

The third issue, wasn't an issue with watermill itself but rather with `bwa` 
which doesn't handled the fasta.gz files for creating the index. Thus I had 
to re-work the `pipeline.js` and `pipeline_lazy.js`
(`examples/pipelines/two-mappers`) 
script to fix 
this.

### resolvedInput
 
In order to display in graphson object and visualization the input(s) provided 
to each task, the task itself must be ran for the first time, otherwise it 
will not have `resolvedInput` (because watermill checks if `resolvedOutput` 
already exists for a given task). So, `resolvedInput` was added to graphson by 
using its definition in `taskState` (check it [here](https://github.com/bionode/bionode-watermill/blob/master/lib/reducers/collection.js#L200)
and [here](https://github.com/bionode/bionode-watermill/blob/master/lib/reducers/collection.js#L125)).


### operationString

Operation or command being passed to shell (at least this is the most 
informative instance for this...) is a quite similar to `resovledInput` in 
the sense that this will only be available in the first run of a given 
pipeline. Otherwise `resolvedOutput` will be read by watermill and render 
`operationString`  as `undefined`. For that, first add to fix a bug that was 
pushing to the `operationString` the function `operationCreator` rather than 
the string itself (check [this](https://github.com/bionode/bionode-watermill/commit/2be0185a1726d314892550a517a0853f90c20abc)).
Until now, there were no operationString instance in default task state, thus
 it was added to default task state and then a new reducer case was added in 
 order to pass this action to the next state and make it available in 
 `taskState` used by ` graphson`. This [commit](https://github.com/bionode/bionode-watermill/commit/f56a1abaf636ca6e4c80e4d35898e92cedef4096) 
 might help to better comprehend the progress done here.

Final implementation of the `graphson` visualization can be checked in [#62](https://github.com/bionode/bionode-watermill/pull/62)
and in [#64](https://github.com/bionode/bionode-watermill/pull/64/files) for the setting of `operationString`.

## Pending

* [ ] Future implementation of logger must be passed through a CLI as 
described in
[Issue #31](https://github.com/bionode/bionode-watermill/issues/31). For now, 
this
is a quick fix for debugging purposes. Later, when we implement a CLI we have
 to have this into account.