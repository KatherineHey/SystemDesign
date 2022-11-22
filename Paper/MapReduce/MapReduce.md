# Abstract
MapReduce is a programming model and an associ-ated implementation for processing and generating large data sets. 

Users specify a map function that processes akey/value pair to generate a set of intermediate key/value pairs, 
and a reduce function that merges all intermediate values associated with the same intermediate key.

## Intuitive: 
Google have implemented hundreds of special-purposecomputations  that  process  large  amounts  of  raw  data,
such as crawled documents,  web request logs,  etc., to compute various kinds of derived data, such as invertedindices, 
summaries of the number of pagescrawled per host, the set of most frequent queries in a given day, etc.

Most such computations are conceptu-ally straightforward.  
However, the input data is usually large and the computations have to be distributed acrosshundreds or thousands of machines in order to finish ina reasonable amount of time.

## Solution:
A new abstraction (mapreduce) that allows us to express the simple computa-tions we were trying to perform 
but hides the messy de-tails of parallelization, fault-tolerance, data distributionand load balancing in a library.  

# Model

### Map
- (Written by the user) takes an input pair and pro-duces a set ofintermediatekey/value pairs. 
- Groups together all intermediate values associated with the same intermediate key I and passes them to the Reduce function.


### Reduce
- (Also written by the user) accepts an intermediate key I and a set of values for that key. 
- Merges together these values to form a possibly smaller set of values. 

# Execution

- Partitioning the input data into a set of M splits.  
- The input splits can be processed in parallel by different machines

   One of the copies of the program is special – the master. The rest are workers that are assigned workby the master. 
   There are M map tasks and R reduce tasks to assign. The master picks idle workers and assigns each one a map task or a reduce task.

- Reduce invocations are distributed by partitioning the intermediate key space into R pieces using a partitioning function (e.g.,hash(key)modR).

   Note: if output keys are URLs, and we want all entries for asingle host to end up in the same output file. 
   To supportsituations like this, the user of the MapReduce librarycan provide a special partitioning function. 
   For example,using "hash(Hostname(urlkey))modR" as the partitioning function causes all URLs from the same host toend up in the same output file.

- Reduce worker uses remote procedure calls to read the buffered data from the local disks of the map workers. 

   When a reduce worker has read all intermediate data, it sorts it by the intermediate keys so that all occurrences of the same key are grouped together. 
   The sorting is needed because typically many different keys map to the same reduce task.

![image](https://user-images.githubusercontent.com/62370578/123471555-d20a9e80-d5aa-11eb-8e96-51359b48d5e0.png)


## master
- For each map task and reduce task, it stores the state (idle,in-progress,orcompleted), and the identity of the worker machine(for non-idle tasks).
- The master is the conduit through which the location of intermediate file regions is propagated from map tasks to reduce tasks. 
- Master stores the locations and sizes of the R inter-mediate file regions produced by the map task. 
- Updates to this location and size information are received as map tasks are completed. The information is pushed incrementally to workers that have in-progress reduce tasks.

## Fault Tolerance
- Master fails, redo whole Map Reduce.
- Map task fail, redo one map task since its result is in a specific region.
- Reduce task fail, if backup executes, other machine picks up reduce from where it's left since a global result is maintained.

## Locality
When running large MapReduce operations on a significant fraction of theworkers in a cluster, most input data is read locally and consumes no network bandwidth.

## Task Granularity
Master must make O(M+R) scheduling decisions and keeps O(M∗R) state in memory 

In practice, we tend to choose M so that eachindividual task is roughly 16 MB to 64 MB of input data(so that the locality optimization described above is most effective), 
and we make R a small multiple of the number of worker machines we expect to use. 


## Backup Tasks
When a MapReduce operation is close to completion, the master schedules backup executions of the remaining in-progress tasks.

