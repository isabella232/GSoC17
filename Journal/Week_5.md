# Week 5 (29 June to 05 July)

- [Summary](#summary)
- [Progress](#progress)
- [TODO or solved...](#todo-or-solved)
    - [Drawbacks](#drawbacks)
- [Needing improvements](#needing-improvements)

## Summary

This week's main goal was to construct a graph object suitable for 
visualization, render the visualization in a simple manner and get output 
manifest text file from this instance of the graph. It would be great to 
establish visualization in real-time meanwhile, where the user could see the 
current status of the graph during the run of any custom pipeline within the 
framework of bionode-watermill.

## Progress

Last week, I figured a very simple way to generate a `graphson` object from the 
established `graph.js` instance already implemented to generate the DAG. 
However, when I started testing with some of the pipelines available in 
[example pipelines](https://github.com/bionode/bionode-watermill/tree/master/examples/pipelines), 
noticed that some extra vertices were being added to the graph object 
(`graphsonObj`). Basically what is adding these extra vertices are tasks that
 have `params` passed to the task object. You can check this code in 
 [collections.js](https://github.com/bionode/bionode-watermill/blob/master/lib/reducers/collection.js)
 
 ```javascript
  // If params is not {}, add a node for it
  const paramsString = JSON.stringify(params)
  if (paramsString !== '{}') {
    // TODO check if params already exists
    graph.addNewVertex(paramsString)

    // Add an edge from params to current task as well
    graph.addNewEdge(paramsString, uid)
  }

  // Add edge from last trajectory node to current output if no params,
  // otherwise to params (which points to current output)
  if (trajectory.length > 0) {
    if (paramsString === '{}') {
      graph.addNewEdge(trajectory[trajectory.length - 1], uid)
    } else {
      graph.addNewEdge(trajectory[trajectory.length - 1], paramsString)
    }
}
```

So, with this piece of code the graphson object rendered something like 
[this](https://github.com/bionode/GSoC17/tree/master/imgs#previous-instance-of-the-graph)
(for the `two-mappers` pipeline.js).
In the previous graph object, you can see in orange the vertices that were 
added as an extra per task with `params`. For graph visualization I think 
these orange vertices aren't interesting, contrary to the green one, which 
represent the junction of the two branches of this join:

```javascript
junction(
  getReference,
  join(getSamples,fastqDump)
)
```

This junction gives the notion that both branches will converge in the next 
tasks, and particularly important if next to a junction, you don't have a 
simple join but rather another junction or a fork (as in the example pipeline
.js):

```javascript
join(
  junction(
      getReference,
      join(getSamples,fastqDump)
  ),
  fork(
    join(IndexReferenceBwa, bwaMapper),
    join(indexReferenceBowtie2, bowtieMapper)
  )
)
```

If we didn't have this junction vertex the pipeline would never converge 
because of the nature of fork which splits the pipeline in two or more 
independent branches.

So what I think is best for now is to keep this junction vertex as 
illustrated [here](https://github.com/bionode/GSoC17/tree/master/imgs#new-instance-of-graph).
Note that `getReference` is the bottom left branch and `join(getSamples,
fastqDump)` is the upper left branch. Then, in orange it is the junction 
vertex and then the fork where `join(IndexReferenceBwa, bwaMapper)` and `join(indexReferenceBowtie2, bowtieMapper)`
fork into two distinct branches on the upper and bottom right.

I think this way it is clearer the flow between tasks, however graph 
visualization still needs improving to better understand it. For now, it has 
labels that show the uid of each task/junction which can be easily used to 
get other parameters from tasks, like input, output, params...

Also an example usage as well as a html page to load the graph in d3 is 
available [here](https://github.com/bionode/GSoC17/blob/master/Experimental_code/graph/index.html).

Also, previous graphson object can be checked [here](https://github.com/bionode/GSoC17/blob/master/Experimental_code/graph/prev_graphson.json), 
with the odd vertices being created like [this one](https://github.com/bionode/GSoC17/blob/master/Experimental_code/graph/prev_graphson.json#L14).
New graphson object can be checked [here](https://github.com/bionode/GSoC17/blob/master/Experimental_code/graph/graphson.json).

## TODO or solved...

* [x] Pass more information to graphson object based on the uid of the tasks

**Solution:**
This was achieved by using the taskState object within 
[collections.js](https://github.com/bionode/bionode-watermill/blob/master/lib/reducers/collection.js).
I had to call `input`  and `name` associated to the reducer of taskState and 
pass it to `graphsonObj`. However, `input` is not really what would be ideal to
 pass to this object (either for output file or for the graph visualization),
  because `input` returns the pattern passed to task rather than the 
  `resolvedInput` which stores the actual input provided in each task 
  (similarly to `resolvedOutput`).

* [x] Add real-time visualization to graphson object rather than outputting it
 to file or stdout.
 
 **Solution:**
 Real-time visualization was implemented using the d3 graph visualization. 
 Basically, the graph is updated on every instance of `jsonifyGraph` in
 [collections.js](https://github.com/bionode/bionode-watermill/blob/master/lib/reducers/collection.js),
 through socket.io, which outputs each time `jsonifyGraph` object is updated.
 
 Note: this now requires the user to refresh `localhost:8084` to visualize 
 the updated pipeline workflow.
 
  In order to do that, I had to give an [index.html](https://github.com/bionode/bionode-watermill/blob/viz/viz/index.html) 
  to render the graph visualization and start the server in [collection.js](https://github.com/bionode/bionode-watermill/blob/viz/lib/reducers/collection.js#L23-L28).
 After initiating the server is just a matter of passing the `graphsonObj` to
  the `index.html`. In the `index.html` (client-side) I had to check if 
  vertices are from a task or from a junction since this should render 
  different labels and vertex colors.
  
  ```javascript
  for (var obj in graph.vertices) {
    // checks if vertex is coming from a junction or a task
    // if object comes from a task it will have an access for output
    // otherwise it will have more objects inside each one with its own output
    graph.vertices[obj]["kind"] = graph.vertices[obj].values.type !==
    undefined ? 'task' : 'junction'
    // this is only undefined because graph.vertices.values.type returns an
    // array rather than an object where type is available.
  }
```

Then, for color I used d3 function:

```javascript
var node = svg.append("g")
  // ... some more code
  .attr("fill", function (d) { return color(d.kind) })
```

A similar check was implemented for label display:

```javascript
    node.append("title")
      .text(function (d) {
        miniUid = d.values.type !== undefined ?
          `./data/${d._id.substring(0, 7)}` : 'n/a'
        return `- uid: ${d._id}
- Kind: ${d.kind}
- Task name: ${d.values.taskName}
- output(s): ${d.values.output}
- Input pattern: ${JSON.stringify(d.values.input)}
- params: ${JSON.stringify(d.values.params)}
- Output folder: ${miniUid}` })
//this will not handle properly bionode-ncbi output folders
```

An important note though is that this type of parsing of output folder will 
not solve the problem with bionode-ncbi rendering output folders outside main
 output folder of bionode-watermill (`./data`). This code can be checked 
 [here](https://github.com/bionode/bionode-watermill/blob/viz/viz/index.html#L49-L89).
 
#### Drawbacks
 
 With this type of implementation each time bionode-watermill executes a 
 script the ending of the script would get stucked because the server-client 
 connection is still open in the end. 
 
 **Bug**: During the first run session is closed at the end of the script. 
 However, if we rerun the script (while everything is already resolved, it 
 will retrieve keep the connection between the server and client open).
 
 Thus, at this stage I think the best way to solve this issue would be to 
 implement a CLI 
 that specifically handles this.

* [x] Work on manifest file based on this object.

 **Solution:**
 Manifest file is no more than the `graphson.json` being saved on every 
 instance of bionode-watermill. It needs to have the **`resolvedInput`** as 
 well 
 as the **command** (`operationString`) executed by shell (when something is 
 passed to shell).
 Sample graphson.json can be seen [here](https://github.com/bionode/GSoC17/blob/master/Experimental_code/graph/graphson.json).
 
## Needing improvements
 
* [x] bionode-ncbi output folder.
* [ ] replace d3 graph visualization with some more appropriate model ([Issue #77](https://github.com/bionode/bionode-watermill/issues/77)).
* [ ] control graph using a CLI, rather than being executed by default.
([Issue #31](https://github.com/bionode/bionode-watermill/issues/31))
 
 