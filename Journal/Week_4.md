# Week 4 (22 June to 28 June)

- [Summary](#summary)
- [Progress](#progress)
    - [Using graph object from graph.js](#using-graph-object-from-graphjs)
    - [Needing improvements](#needing-improvements)
- [Notes on manifest file](#notes-on-manifest-file)


## Summary

This week was dedicated to the improvement of workflow representation. Output
 representation was devided in two stages: a summary file output as text and 
 a graph representation of the workflow being performed in each pipeline.
 
## Progress

Since pull request [#57](https://github.com/bionode/bionode-watermill/pull/57/files)
wasn't ready to merge yet, we decided to add the fix on task name logging 
separately in a new pull request ([#58](https://github.com/bionode/bionode-watermill/pull/58)).

At first, I struggled to understand the dynamics implemented in 
bionode-watermill with [redux](https://www.npmjs.com/package/redux). A good 
guide to start understanding how redux work is available [here](https://egghead.io/lessons/javascript-redux-the-single-immutable-state-tree).

After, understanding a little more on how redux is working I started to 
transform the current output for the DAG into something more similar to 
[graphson](https://www.npmjs.com/package/graphson). One nice example on how 
graphson json objects should look like are available [here](https://github.com/tinkerpop/blueprints/wiki/GraphSON-Reader-and-Writer-Library).

Currently bionode-watermill use `graph.js` and is rendering 
something like this on each vertex:

```javascript
'someName': {
  stuff: '...'
  someotherstuff: '...'
}
```

However, graphson expects something like:

```javascript
{ 
  stuff: '...'
  someotherstuff: '...'
}
```

Right now, I am not sure if graphson will be able to handle this different 
in vertex object! **UPDATE**: This isn't in fact a limitation... so nevermind.

So, I managed to hack current `jsonifyGraph` function (see [code](https://github.com/bionode/bionode-watermill/blob/master/lib/reducers/collection.js#L60-L78)),
basically by creating a new object `graphsonObj` in which the previous `obj` 
 elements could be inserted. However, it seems not to be working properly 
 since in some cases with fork it seems to be rendering duplicated vertices.
 Ideas to solve this issue:
 
 * [ ] :no_entry: add a statement to check if id object already exist in all 
 `graphsonObj.graph.vertices.uid` under `graphsonObj`.
 * [ ] :no_entry: start creating the `obj` in a different manner, maybe 
 without `graph.js`?
 * [x] use [graph.js](https://www.npmjs.com/package/graph.js) API.
 
Also current DAG implementation renders something odd with task `uid` often 
being replaced by task `params` (**this happens on junction/fork**). Then, this
 is currently rendering wrong format of graphson object.
 
### Using `graph` object from [graph.js](https://www.npmjs.com/package/graph.js)

Right now, the easier and most friendly way to generate graphson is to take 
advantage of `graph`  generated in `addOutputHandler` and 
`addJunctionVertexHandler` functions in [collections.js](https://github.com/bionode/bionode-watermill/blob/master/lib/reducers/collection.js):

First, I created the backbone of the `graphsonObj` (graphson object) as follows:
```javascript
const graphsonObj = {
  graph: {
  mode: "NORMAL",
  vertices: [],
  edges: []
  }
}
```

And then used graph.js API to render `vertices` and `edges` list to graphsonObj:

```javascript
// iterate through verties
// in ECMAScript 6, you can use a for..of loop
for (let [key, value] of graph.vertices()) {
  // iterates over all vertices of the graph
  graphsonObj.graph.vertices.push({
    key: key,
    outputs: value
  })
}
// iterate through edges
// in ECMAScript 6, you can use a for..of loop
for (let [from, to, value] of graph.edges()) {
  // iterates over all vertices of the graph
  graphsonObj.graph.edges.push({
    from: from,
    to: to,
    value: value    // currently value is not being created on .addNewEdge()
  })
}
```

This will render an object like this:


```javascript
{
  "graph": {
    "mode": "NORMAL",
    "vertices": [
      {
        "key": "ba2ed8fd525dccc420afe49017a16ebd0ab9c9c80cc5e59b4bb97cb0290934f7",
        "outputs": "/home/tiago/bin/bionode-watermill/examples/pipelines/pids/data/ba2ed8f/1495738159087.pids"
      },
      {
        "key": "0b4e534d9c457df3357c12ecee06480f9f2077058cd13066eb4711aa59cf636a",
        "outputs": "/home/tiago/bin/bionode-watermill/examples/pipelines/pids/data/0b4e534/1495738159087.txt"
      }
    ],
    "edges": [
      {
        "from": "ba2ed8fd525dccc420afe49017a16ebd0ab9c9c80cc5e59b4bb97cb0290934f7",
        "to": "0b4e534d9c457df3357c12ecee06480f9f2077058cd13066eb4711aa59cf636a"
      }
    ]
  }
}
```

### Needing improvements

* [x] Pass other variables to vertices and edges information like tasks 
`input`, 
`output`, `params`.
* [x] There is an issue in which only one output in `params.output` is being 
passed to `graphsonObj.graph.vertices.outputs`. This is problematic when 
`params.output` is a list. Same may be valid for `input` and `output` (solved).
* [x] As it is now, graph object is generated within `jsonifyGraph` and this 
render several instances of `graphsonObj` and this in fact ok for real-time 
visualization of the graph.

## Notes on manifest file

With the graphson object generated, it might be easy to pass more information
 into the graph object and them use it to output a text file something like 
 exemplified above. For instance, we can pass task params, input, and so on 
 to each vertex and then use it to output not only the graph but also a 
 better manifest file.