
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

## MapReduce Algorithms

### Scaling up vs out

 - small cluster of SMP machines vs large cluster of commodity hardware
 - intra-node latencies ~ 100ns
 - inter-node latencies ~ 100micros

### Optimising Computation

 - Sort order of intermediate keys
 - Control which reducer processes which keys
 - Preserve state in mappers and reducers (local aggregation) => lower communication

### Combiner Design

 - Combiners and reducers have same method signature
 - Combiners and mappers should write same value types

### Large Counting Problems

 - We want to emit bigrams
 - Let mappers create partial counts and reducers aggregate them
 - Two designs:
     + Pairs: Emit ((a, b), 1) for every pair of co-occuring words
         * Easy to implement but lot of shuffling and sorting
     + Stripes: Emit (a, [(b, 1), (c, 1), ...])
         * Far less sorting and shuffling
         * Better use of combiners
         * Limited in memory size

## Replication

 - To continue working even if a fault occurs
 - To improve performance:
     + Load sharing
     + Nearer location for data access
 - Remote sites working when local fail
 - Protection against data correction
 
### Requirements

  - Transparency: clients see logical objects not physical, each access return single object
  - Consistency: All replicas are consistent for some condition

### Synchronisation Models

 - Non-explicit models:
     + Strict: All processes must see shared accesses in absolute time order
     + Linearisability: All processes must see shared accesses in the same order; accesses are ordered according to global timestamp
     + Sequential: All processes must see shared accesses in the same order; timestamps don't matter
     + Causal: All processes must see causally-related shared accesses in the same order
     + Fifo: All processes see writes from each other in order they were used; different processes may not always be seen in that order

 - Explicit models:
     + Weak: Shared data is consistent only after synchronisation
     + Release: Shared data is made consistent when a critical region is exited
     + Entry: Shared data pertaining to a critical region is made consistent when a critical region is entered

### Eventual Consistency

 - Sacrifice global consistency, keep local consistency
 - Read access => no problem
 - Infrequent writes => ok as long as same client same replica

### Fault Tolerance

 - Availability: System is ready to be used immediately
 - Reliability: System is always up
 - Safety: Faillures are never catastrophic
 - Maintainability: All failures can be fixed without noticing

### Failure Models

 - Crash: A node halts, but is working correctly before
 - Omission: A node fails to respond to requests
 - Timing: A node's response lies outside specified time interval
 - Response: A node's response is incorrect
 - Arbitrary: A server produces arbitrary responses at arbitrary times

### Solutions

 - Information redundancy: Error detection and recovery (hardware level)
 - Temporal redundancy: start operation and if it does not complete start it again (transactions required)
 - Physical redundancy: add extra software and hardware, have multiple instances

### Issues associated with fault tolerance

 - Process resilience: replicate processes into groups; agreement within a group?
 - Reliable client/server communication: masking crashes and omissions
 - Reliable group communication: processes coming/leaving the group
 - Distributed commit: performed by all members or none at all
 - Recovery strategies: recovering from an error

### Byzantine fault tolerance

 - Solution only if number of messages is more than 3 times the number of messages that were lost.

### Recovery

 - Backward recovery (more common): return system to some previous correct state
     + Continually take snapshots of the system
     + When to delete snapshots?
 - Forward recovery: bring system to correct state and continue
     + Account for all errors upfront => have strategy

## Virtualisation

 - Technique to separate hardware, OS, and applications

### CPU and Architecture Virtualisation

 - User ISA and system ISA
 - ISA virtualisation, instruction interpretation, trap and emulate, binary translation, and hybrids
 - Virtualisation needs to translate guest state into host state as well as transformations that advance state

### User ISA

 - Application state: Virtual memory, registers

### System ISA

 - The inner rings: 0 (and maybe 1)
 - Control registers of the CPU 
 - System clock
 - Memory management unit: page table, TLB
 - Device I/O
 - Virtualisation monitor (hypervisor)
     + Monitor supervises the guest and virtualises calls to the System ISA
     + Whenever the guest wants to access System API, the monitor takes over
     + Shares address space with address space it virtualises
     + It handles page faults

### Virtualisation Types

 - Trap and emulate: execute normal instructions, trap privileged instructions and emulate running them
     + Could be ran in host kernel/extension level
 - Binary translation:
     + Compile programs to intermediate representation (Java, llvm)
     + Transform instructions on the fly
     + Separate model for host and guest accesses
 - Hybrid models: kernel is binary translated, user code is trapped and emulated

### Further Virtualisation

 - We can emulate CPU & memory but I/O devices as well
     + Uniformity: Remote hard drive or RAID
     + Isolation: Devices operate as if they were alone
     + Performance: Lower level entities optimise I/O path
     + Multiplexing: Parallelise processes (e.g. replication)
     + System Evolution: Connecting new drive while system is alive
 - Three main techniques:
     + Direct Access
         * No changes to guest but specialised hardware for host
         * Hardware interface needs to be visible to guest
     + Device Emulation
         * Drivers are in monitor or host, no special hardware
     + Paravirtualisation
         * Expose monitor and allow guest to make monitor calls
         * Implement guest-specific drivers (one each)

## NoSQL

