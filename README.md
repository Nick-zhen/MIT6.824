# MIT6.824
## What is MapReduce
MapReduce is a programming model and associated implementation designed for processing and generating large datasets in a parallel and distributed fashion. It was popularized by Google through a seminal paper titled “MapReduce: Simplified Data Processing on Large Clusters” written by Jeffrey Dean and Sanjay Ghemawat, published in 2004.

#### Map Function
- The user specifies a map function that processes a key/value pair to generate a set of intermediate key/value pairs.
- The map function is applied to each logical record in the input data, producing intermediate key/value pairs.

#### Reduce Function
- The user also specifies a reduce function that merges all intermediate values related to the same intermediate key.
- The reduce function is applied to the output of the map function, combining values with the same key to produce a possibly smaller set of values.
