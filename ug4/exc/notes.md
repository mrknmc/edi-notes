
## Intro

### Everything as a service

Infrastructure as a Service

:   Rent cycles on machines (Amazon EC2)

Platform as a Service

:   Nice API, they take care of maintenance and upgrades (Google App Engine)

Software as a Service

:   Run the software for me (GMail, Salesforce, etc.)


## Big Picture

 - Architecture has tiers:
     + Tier one handles requests
     + Tier two is caching etc.
 - Inner services (DB and index) are shielded from online load
 - Replicate data within our cache to spread loads and provide fault-tolerance
 - Not everything needs to be replicated

## Second Tier Examples

 - Memcached (in-memory key-value store)
 - Distributed hash tables
 - DynamoDB (Amazon service)
 - BigTable (Google service)

## Read vs Write

 - Reading many values takes as long as the slowest read
 - Writing many values is the same...

## CAP Theorem

 - "You can have just two from Consistency, Availability, and Partition Tolerance"
 - Usually consistency is lowered so the other two are "true"
 - Lower consistency can be okay if data non-essential

## Programming Models

 - Shared memory (pthreads)
 - Message passing (MPI)

## Design Patterns

 - Master-Slave (Farm)
 - Producer-Consumer
 - Shared Work Queues

## MapReduce

 - Beyond von Neumann architecture
 - Hiding system level details from developers
 - Separating the wat from how

### Big Ideas

 - Scaling out not up
 - Processing near data (not wasting bandwidth)
 - Process data sequentially
 - Add more machines => more scalable (data centre == computer)

### Runtime

 - Handles scheduling, data distribution, synchronisation, and errors
 - Specify two functions `map -> <k, v>` and `reduce -> <k, [v1, v2, ...]>`
 - All keys with same key go to same reducer
 - Programmers can also specify `partition` and `combine` (mini-reducers in memory)

### Implementations

 - MapReduce (proprietary by Google)
 - Hadoop (open-source by Apache)

## Distributed Filesystems

 - Move data to workers (not enough RAM, minimise traffic)
 - High component failure rates
 - Commodity hardware (cheap)
 - "Small" number of big files
 - Files are mostly appended to
 - Sequential reads > Random reads

### Implementations

 - GFS (Google MapReduce) and HDFS (Hadoop)
 - Files stored as chunks (64 MB)
 - Reliability via replication
 - One master coordinates access
 - No data caching, simple API

### Architecture

 - There is a master (Google: GFS master, Hadoop: namenode)
 - And there are "workers" (Google: GFS chunkservers, Hadoop: datanodes)

#### Namenode

 - Hold file structure, metadata, permissions and file-to-chunk mapping
 - Directs clients to specific datanodes for reads and writes
 - Maintains health (heartbeats and such)

## Hadoop

 - Mapper
    + void setup(Mapper.Context context) - Called once at the beginning of the task
    + void map(K key, V value, Mapper.Context context) - Called once for each key/value pair in the input split
    + void cleanup(Mapper.Context context) Called once at the end of the task

 - Reducer/Combiner
    + void setup(Reducer.Context context) - Called once at the start of the task
    +  void reduce(K key, Iterable<V> values, Reducer.Context context) Called once for each key
    + void cleanup(Reducer.Context context) Called once at the end of the task

 - Partitioner
    + int getPartition(K key, V value, int numPartitions) -Get the partition number given total number of partitions
 - Job
    +  Represents a packaged Hadoop job for submission to cluster
    + Need to specify input and output paths
    + Need to specify input and output formats
    + Need to specify mapper, reducer, combiner, partitioner classes
    + Need to specify intermediate/final key/value classes
    + Need to specify number of reducers (but not mappers, why?)

### Some Useful Interfaces

 - `Writable` - serialisation protocol (keys and values)
 - `WritableComparable` - Defines sort order (all keys)
 - `IntWritable`, `LongWritable`, ... - Specific types

### Complex Data Types

 - Store them in text => regex them out (hacky) or JSON
 - Implement said interfaces

### Hadoop Architecture

 - Master
     + Namenode: master node for HDFS
     + Jobtracker: gets job submissions and distributes them
 - Worker
     + Tasktracker: contains task slots (assignments)
     + Datanode: contains HDFS file blocks
 - Client creates a job, configures it and sends to jobtracker
 - Job is divided into tasks and executed on tasktrackers which poll for them in a shared location

### Shuffle and Sort

 - Mapper 
     + Outputs are buffered in a circular buffer
     + When buffer hits threshold, spill the contents on to disk
     + Contents are merged into a single file (within each partition), combiners run
 - Reducer
     + Map outputs are copied over to reducer worker
     + Multi-pass sort (in memory and on disk), combiners run
     + Final merge pass goes into the reducer



