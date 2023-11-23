# MIT6.824
## What is MapReduce
MapReduce is a programming model and associated implementation designed for processing and generating large datasets in a parallel and distributed fashion. It was popularized by Google through a seminal paper titled “MapReduce: Simplified Data Processing on Large Clusters” written by Jeffrey Dean and Sanjay Ghemawat, published in 2004.

#### Map Function
- The user specifies a map function that processes a key/value pair to generate a set of intermediate key/value pairs.
- The map function is applied to each logical record in the input data, producing intermediate key/value pairs.

#### Reduce Function
- The user also specifies a reduce function that merges all intermediate values related to the same intermediate key.
- The reduce function is applied to the output of the map function, combining values with the same key to produce a possibly smaller set of values.

<img width="420" alt="image" src="https://github.com/Nick-zhen/MIT6.824/assets/62523802/2e319b2e-5b4b-4aca-92a7-13324680bdbe"> <br>
The figure is from MapReduce: Simplified Data Processing on Large Clusters


## What problem are we going to solve
We are using this model to solve the word counting problems.
- The map function produces each word for key and the associated count of occurrences of that key word. (k1, v1) => list (k2, v2)
- The reduce function gets the list of key/value pairs from input and output by summing all counts for a particular word. List (k2, list (v2)) => list (k2, v3)

### Case 1: sequential word count <br>
 ![image](https://github.com/Nick-zhen/MIT6.824/assets/62523802/1119b356-e8f3-424e-ba31-5e836d9c99f9)
 
1. Partition the data: the result of split data is <key, value> type. In this case, the key does not matter, it represents the file name. The value represents the word.
2. Run Map function: each machine will run the map function and output <key, value> type. The key represents the word. The value represents the occurrences of that specific word.
3. Run reduce function: the output is still <key, value> type. The key is the word. The value represents the total count of the word. This step is kind of merging.

### Case 2: distributed word count

 ![image](https://github.com/Nick-zhen/MIT6.824/assets/62523802/a2b15401-f57e-4683-a478-f55056507a04)

1. **Initialization**: The master node initializes and reads the input file.
2. **Conversion to Map Tasks**: The master converts the input file into a series of map tasks.
3. **Assignment of Map Tasks**: These map tasks are then assigned to mapper workers, contingent upon receiving mapper requests for the tasks.
4. **Mapper Operation**: Each mapper worker processes the input file, breaking down the text content into individual key-value pairs. In this case, each key represents a word, and the corresponding value is set to 1. This collection of pairs forms the 'intermediate data'.
5. **Data Partitioning**: Mappers then divide the intermediate data into 'NReduce' segments (equivalent to the number of reducers). This division is based on the hash of each key, followed by taking its modulus with NReduce.
6. **Intermediate Data Handling**: Mappers write the partitioned intermediate data to a temporary storage path. Upon completion of this step, they notify the master about the path, signaling the end of the map task.
7. **Master Verification**: The master verifies the validity of requests from mapper workers and, if confirmed, transfers the intermediate data from the temporary to the final storage path.
8. **Generation of Reduce Tasks**: Following the successful completion of all map tasks, the master generates reduce tasks from the intermediate data located in the final path.
9. **Assignment of Reduce Tasks**: These reduce tasks are then allocated to reducer workers, who are awaiting reducer task requests.
10. **Reducer Collection Step**: Upon receipt of a reduce task, a reducer worker's first action is to gather the intermediate data produced by the mapper workers.
11. **Sorting of Data**: The reducer sorts this collected intermediate data based on the key.
12. **Reduce Operation**: The reducer executes the reduce operation on the sorted data, tallying the frequency of each key (i.e., the number of key-value pairs with identical keys) and employing this count as the new value.
13. **Reducer Output Handling**: The reducer writes the results of the reduce operation to a temporary path. Once this is done, the reducer informs the master of the temporary path, indicating the task's completion.
14. **Final Verification by Master**: The master then verifies the reducer requests. Once validated, it transfers the results of the reduce task from the temporary to the final storage path.
15. **Completion of Process**: With the confirmation of all reduce tasks' completion by the master, the MapReduce operation is successfully concluded.
