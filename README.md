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
 ![image](https://github.com/Nick-zhen/MIT6.824/blob/main/pic/sequential.png)
 
1. Partition the data: the result of split data is <key, value> type. In this case, the key does not matter, it represents the file name. The value represents the word.
2. Run Map function: each machine will run the map function and output <key, value> type. The key represents the word. The value represents the occurrences of that specific word.
3. Run reduce function: the output is still <key, value> type. The key is the word. The value represents the total count of the word. This step is kind of merging.

### Case 2: distributed word count

 ![image](https://github.com/Nick-zhen/MIT6.824/blob/main/pic/distributed.png)

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

## How to Handle Distributed Problems 
### Fault-tolerance Problem
1. If a worker (mapper or reducer) encounters an error at a certain step, how should the worker handle it? Should it retry or abandon the current task, and then request a new task for immediate execution? Is it necessary to inform the master proactively? How should intermediate files generated during the process be handled?
   
   *The Workers will keep running loops and request tasks from the master as long as it has not received a message from the master that indicates the completion of all tasks. If a worker encounters an error during the execution stage, it does not need to report to the master. It can abandon this task and request a new task from the master. Also, the worker could report the error it had to the master. But it also takes some time to send a message. In addition, we are planning to design the master itself to monitor the status of the tasks, which makes the error reporting unnecessary.*

2. If the master detects that a particular task has exceeded its time interval, how should it handle the situation?

   *The master keeps monitoring the status of the tasks. If it detects that a task has exceeded its time interval. It will assign the task to an idle worker for execution. There should be a task controller, responsible for managing task status, execution time, task ID, task type and so on.*

3. If the master observes that a specific task has been redundantly executed multiple times, how should it address this issue?

   *To prevent a task from being executed multiple times. A task ID is necessary for checking each task. The task ID is an incrementing counter that increases by 1 each time a task is assigned to a worker. The master’s task controller should keep those ID records. After a worker notifies the master the completion of the task, the master should verify the task ID from work’s task. If the ID is inconsistent in the master’s task controller, it means the task has been reassigned due to timeout, as the reassigned task ID has been increased. In this case, the master considers the task as invalid due to the mismatch of task ID. After receiving the message of task failure, the worker proceeds to the next iteration, requesting a new task from the master.*

### Worker Parallelism Problem
We are not planning to strictly distinguish the map workers and reduce workers. We treat both as workers. In other words, if the master gives a map task to a worker, then the worker will be responsible for both the map and reduce task. Only after the completion of all map tasks, the master will produce the reduce tasks.

The number of mappers and reducers is decided by the user. If you have 5 machines, then you can create 5 workers and they can work simultaneously. If you have only 1 machine but have 8 cores, then you can create 8 workers.

### Data Race Problem
In this model, we have one master and multiple workers. When multiple workers have requests for master at the same time, it might cause read/write conflicts. How can we handle those conflicts and data race?

We are planning to use mutex locks, making sure to release the lock after each completion of the operation. Try our best to minimize the range of using locks. Because it will decrease the performance of the program.

