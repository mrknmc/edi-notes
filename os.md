# OS notes


## History of computers

TL;DR

## Computer Architecture


### The Von Neumann Architecture

<!-- image -->

### Levels of programming languages

 1. Executable File ("Machine code")
 2. Object File linked with other Object Files ("Libraries") into 1.
 3. ASM Source which is assembled into 2.
 4. C/C++ Source code which is compiled into 3.
 5. ML/Java Byte-code which is interpreted

This is analogous to operation of the computer.

### Layered Virtual Machines

Think of a virtual machine in each layer built on the lower VM; machine in one level understands language of that level

 0. Digital Logic Level
 1. __Conventional Machine Level__
 2. __Operating System Level__
 3. Assembly Language Level
 4. Compiled Language Level
 5. Meta-Language Level

### Registers

 - Very fast on-chip memory
 - Typically 32 or 64 bits
 - 8 to 128 registers is usual
 - Data is loaded onto registers before being operated on
 - Registers may not be visible to the programmer
 - Most processors have _data_ and _control_ (special meaning to CPU) registers

### Memory Hierarchy

 1. CPU
 2. Cache
     i. fast, expensive
     ii. several levels of cache
 3. Main Memory
 4. DISK I/O
     i. I/O devices usually connected via a bus
     ii. very slow and cheap

### Fetch-Execute Cycle

PC initialised to fixed value on CPU reset. Then repeat until halt:

 1. instruction _fetched_ from memory address in PC into instruction buffer
 2. Control Unit _decodes_ the instruction
 3. Execution Unit _executes_ the instruction
 4. PC is updated either explicitly by a jump or implicitly

### Buses

A bus is a group of 'wires' shared by several devices. Buses are cheap and versatile but can become a bottleneck on performance. A bus typically has:

 - address lines
 - data lines
 - control lines

A bus is operated in a master-slave protocol: e.g. to read data from memory, CPU puts address on a bus and asserts 'read'; memory retrieves data, puts data on bus; PC reads from bus. In some cases an initialisation protocol is needed to decide which device is the bus master.

### Bus Hierarchy

<!-- image -->

### Interrupts

There are devices much slower than the CPU. We can't have CPU wait for these devices. Also, external events may occur.

Interrupts provide a suitable mechanism. Interrupt is a signal line into CPU. When asserted, CPU jumps to a particular location (e.g. on x86 on interrupt the CPU jumps to address stored in relevant entry of table pointed to by IDTR control register). The jump saves state; when the interrupt handler finishes, it uses a special return instruction to restore control to original program.

### Direct Memory Access

DMA means allowing devices to write directly (via a bus) into main memory, e.g. CPU tells device 'write next block of data into address x' and gets an interrupt when done.

PCs have a basic DMA; IBM main-frames have I/O channels which are a sophisticated extension to DMA.

## Operating System function

### What is an Operating System for?

 - handles relations between CPU, memory and devices
 - handles allocation of memory
 - handles sharing of memory and CPU between different logical tasks
 - handles file management
 - (in Windows) handles the UI graphics

__Kernel__ - single (logical) program that is loaded at boot time and has primary control of the computer

### Early batch systems

In the beginning, OS simply transferred programs from punch cards into memory. Operator had to set up entire job, programmatically.

_Monitor_ is a simple resident OS that reads jobs, transfers control to programs, receives control. Monitor is permanently resident, programs must be loaded into a different area of the memory.

Batches of jobs can be put onto one tape and read in turn by the monitor - reduces human intervention.

Protecting the monitor from the users, which should not be able to:

 - __memory protection__: write to monitor memory
 - __timer control__: run forever
 - __privileged instructions__: directly access I/O or certain other machine functions
 - __interrupts__: delay the monitor's response to external events

### Multiprogramming

Jobs would waste 75% CPU cycles waiting on I/O. _Multiprogramming_ was introduced to tackle this. Monitor loaded several user programs when one is waiting for I/O, run another.

Multiprogramming, means the monitor must:

 - manage memory among the various tasks
 - schedule execution of the tasks

### Time-sharing

Allow interactive access to computer with many users sharing. Early system gave each user 0.2s of CPU time; then save state and load state of next scheduled user.

### Virtual Memory

Multitasking and time sharing are much easier if all tasks are resident rather than being swapped in and out of memory.

__Virtual memory__ - decouples memory as seen by the user task from physical memory. Task sees virtual memory which may be anywhere in real memory or paged out to a disk.

### The Process Concept

With virtual memory, it becomes natural to give different tasks their own independent _address space_ or view of memory. Monitor then schedules _processes_ appropriately and does all the _context-switching_ transparently to user process.

### Modes of CPU operation

To protect OS from users, all modern CPUs operate in more than one _privilege level_:

 - IBM has __supervisor__ and __problem__ states
 - x86 has rings 0, 1, 2, 3

Transitions to higher privilege levels are only possible through tightly controlled mechanisms. IBM __SVC__ or Intel __INT__ are like software interrupts that change to supervisor mode and jump to pre-determined address.

### Memory Protection

Virtual memory allows user's memory to be isolated from kernel memory and other users' memory:

 - A frame or page may be read or write accessible only to a processor in high privilege level
 - In S/370 each frame of memory has a 4-bit storage key and each task runs with a particular key
 - The virtual memory mechanism can be extended with permission bits; frames can then be shared
 - Combination of all of the above may be used

### OS Structure 

- __Traditional (Monolithic)__ 
    + All OS functions sit in the kernel a single function can crash the whole system
- __Micro-kernel__
    + Small core which talks to (maybe privileged) components in separate servers
    + Increase modularity
    + Increase extensibility
    + More overhead
    + Difficult to implement
    + Keep multiple copies of OS data structures

Modern OSes are hybrid:
 - Linux is monolithic but has (un)loadable modules
 - Windows started micro-kernel but for performance changed

## Processes

__Process is a program in execution.__ It may have own view of memory, sees one processor although it's sharing it with other processes - __virtual processor__. To switch between processes we need to track:

 - its memory, including stack and heap
 - contents of registers
 - PC
 - state

### Process States

 - __New__: process being created
 - __Running__: process being executed on CPU
 - __Ready__: not on CPU but ready to be
 - __Blocked__: waiting for an event (I/O)
 - __Exit__: process finished, awaiting clean-up

<!--  -->

 - __admit__: process control set up, move to run queue
 - __dispatch__: scheduler gives CPU to runnable process
 - __time-out/yield__: running process forced to/volunteers to give up CPU
 - __event-wait__: waiting for an event (I/O)
 - __event__: event occurs - wake up process and tell it
 - __release__: process terminates, release resources

<!-- transition image -->

### Process Control Block (PCB)

 - unique process ID
 - process state
 - PC and other registers
 - memory management
 - scheduling and memory management info
 - list of open files, name of executable, owner, CPU time used so far, devices

### Kernel Context

Kernel executes:

 - Older OSes: single program in real memory
 - Modern OSes: may execute in context of a user process, parts of OSes may be processes (e.g. I/O in Unix and IBM)

### Creating Processes

 - By the OS when a job is submitted or a user logs on
 - By the OS to perform background service for a user (e.g. printing)
 - By explicit request from user program

When a process is created, OS must:

 - assign unique identifier
 - allocate memory space, kernel and user memory
 - initialise PCB and memory management tables
 - link PCB into OS data structures
 - initialise remaining control structures
 - WinNT, IBM: load program
 - Unix: make child process copy of parent (on write)

### Ending Processes

 - Terminate voluntarily (e.g. exit())
 - Perform illegal operation
 - Be killed by user (e.g. kill()) or OS
     + allocated resources exceeded
     + task functionality no longer needed
     + parent terminating

On termination, OS must:

 - deal with pending output, etc.
 - release all system resources held by the process
 - unlink PCB from OS data structures
 - reclaim all user and kernel memory

### Threads

Processes

 - own resources such as address space, I/O devices and files
 - are units of scheduling and execution

Threads, however, are allowed to be executed concurrently in one process. Everything said about scheduling applies to threads as well but process-level context is shared by thread contexts.

 - creating threads is quick (cca. 10 times faster than process)
 - ending threads is quick
 - switching threads within process is quick
 - inter-thread communication is quick and easy (shared memory)

### Thread operations

 - __create__: thread spawns a new thread, specifying instruction pointer or routine to call, OS sets up everything
 - __block__: thread waits for event - other threads may execute
 - __unblock__: event occurs, thread becomes ready
 - __finish__: thread completes, context reclaimed

### Thread libraries

- thread library implements mini-process scheduler (in user space)
- context of thread is PC, registers, stacks, etc. (in user space)
- thread control block (in user process's memory)
- switching between threads voluntary or on time-out

__Advantages__

 - context-switching is fast (no OS)
 - scheduling can be tailored to the application
 - library can be OS-independent

__Disadvantages__

 - if thread makes blocking system call, entire process is blocked (there are ways around this)
 - user-space threads don't execute concurrently on multi-processor systems
 
## Multi-processing

Several processors used together

 - __Single Instruction Single Data Stream (SISD)__: normal set-up, one processor, one instruction stream, one memory
 - __Single Instruction Multiple Data Stream (SIMD)__: a single program executes in lock-step on several processors (large scientific apps)
 - __Multiple Instruction Single Data Stream (MISD)__: not used
 - __Multiple Instruction Multiple Data Stream (MIMD)__: many processors each execution different programs on different data

Within MIMD, processors could be _loosely coupled_ (e.g. network of PCs with communication links) or _tightly coupled_ (e.g. processors connected via a single bus).

### Symmetric Multi-processing (SMP)

Where does the OS run when multiple processors?

 - __master-slave__: kernel runs on one CPU, and dispatches processes to others. All I/O is done by request on kernel CPU. Easy but inefficient and failure prone.
 - __symmetric__: the kernel executes on any CPU. Kernel may be multi-process or multi-threaded. Each processor may have its own scheduler. More flexible and efficient - more complex.

__SMP OS design considerations__

 - __cache coherence__: several CPUs, one shared memory. Each CPU has its own cache. Usually solved by hardware designers.
 - __re-entrancy__: several CPUs may call kernel simultaneously. Kernel code must be written to handle this.
 - __scheduling__: genuine concurrency between threads and kernel threads.
 - __memory__: must maintain virtual memory consistency between processors.
 - __fault tolerance__: single CPU failure should not influence others.

### Scheduling

Happens over several time-scales and at several levels:

 - __batch scheduling (long-term)__: which jobs should be started
 - __medium-term__: some OSes _suspend_ or _swap out_ processes to ameliorate resource contention
 - __process scheduling (short-term)__: which process gets CPU next, how long

__Criteria for scheduling__

 -  good utilisation: minimise amount of CPU idle time and job throughput
 -  fairness: all jobs should get a 'fair' share of the CPU
 -  priority: high-priority jobs get larger share
 -  response time: fast response to interactive input
 -  real-time: hard deadlines, e.g. chemical plant control
 -  predictability: avoid wild variations in user-visible performance

Balance is dependent on the system. On a PC, response time is important. On a main-frame throughput is important.

### Non-pre-emptive Policies

Once a job gets CPU it keeps it until it yields it or needs e.g. I/O. Suitable for long-term policies, not used for short-term.

 - __first-come-first-served (FCFS)__: Favours long and CPU-bound processes over short or I/O bound processes. Used as sub-component of priority systems.
 - __shortest-process-next (SPN)__: Dispatch process with shortest expected processing time. Improves overall performance time. Favours short jobs and has poor predictability. To estimate expected time - user can estimate (long-term), build up CPU residency over time (short-term)

### Pre-emptive Policies

Processes could be interrupted after some time - __quantum__.

 - __round-robin__: When quantum expires, running process sent to the back of the queue. Favours CPU-bound processes - can be refined to avoid. Quantum should be slightly greater than average interaction time (Unix - 50 ms).
 - __shortest-remaining-time (SRT)__: Pre-emptive version of SPN. On quantum expiry, dispatch process with shortest expected running time. Tends to starve long CPU-bound processes.
 - __feedback__: use dynamically assigned priorities.
     + New process starts in queue with priority 0 (highest).
     + Each time it is pre-empted, goes back to next lower priority queue.
     + Dispatch first process in highest occupied queue.
     + Tends to starve long jobs, possible solutions:
         * Increase quantum for lower priority processes
         * Raise priority for processes that are starved

### Multi-processor Scheduling

 - Assigning processes to processors
     + static assignment - may have idle CPUs
     + dynamic assignment - complexity increased
 - Deciding on multi-programming on each CPU
     + If many CPUs and app parallel at thread-level then maybe don't.
 - Dispatching processes

### SMP Scheduling

For _process scheduling_, performance analysis and simulation indicate that the differences between scheduling algorithms are reduced in SMP - no need to use complex systems, FCFS or a variant may suffice. However, FCFS has disadvantages:

 - single pool of TCBs (like PCB for threads) must be accessed with mutual exclusion - may be a bottleneck
 - pre-empted threads are unlikely to be re-scheduled to the same CPU - loses benefit of a CPU cache
 - unlikely for program to get its threads running at the same time, opposite could impact performance

For _thread scheduling_, situation more complex - unlike processes, threads often interact. Main approaches are:

 - __load sharing__: idle processors selects ready thread from whole pool
     + simplest and most like uni-processing environment
 - __gang scheduling__: a gang of related threads are simultaneously dispatched to a set of CPUs
 - __dedicated CPUs__: static assignments of threads to CPUs
 - __dynamic scheduling__: involve the application in changing number of thread; OS shares CPUs among apps 'fairly'

Most systems use load sharing (with tweaks), some scientific systems use gang scheduling.

### Real-Time Scheduling

Real-time systems have deadlines. Theses may be _hard_ (necessary for success of a task) or _soft_ (if not met, still worth running the task).

Requirements for RT systems:

 - __determinism__: need to acknowledge events within pre-determined time
 - __responsiveness__: take appropriate action quickly enough
 - __user control__: hardness of deadlines and priorities is a matter for user
 - __reliability__: systems must fail softly, no panicking, ideally no fails

## Concurrency

Control access to a shared variable: protect each read-write sequence by a _lock_ which ensures _mutual exclusion_.

### Mutual Exclusion

Allow process to identify _critical sections_ where they have exclusive access to a resource.

Requirements:

 - mutual exclusion must be enforced
 - processes blocking in non-critical section must not interfere with others
 - processes wishing to enter critical section must eventually be allowed to do so
 - entry to critical section should not be delayed without a cause
 - there can be no assumptions about speed or number of processors

Implementation: 

 - via hardware: special machine instructions
 - via OS support: OS provides primitives to call
 - via software: entirely by user code

We assume that mutual exclusion exists in hardware, memory access is atomic - only one write/read at a time.

__Dead lock__ - both processes loop forever waiting for the other to finish.

__Live lock__ - both processes run in exact synchrony and keep deferring to each other.

### Mutex - Dekker's algorithm

Ensure that one process has priority, so will not defer and give other process priority after performing own critical section.

~~~c
/* using j instead of i hat */
flag[i] = true;
while (flag[j]) {
    if (turn == j) {
        flag[i] = false;
        while (turn == j) {}
        flag[i] = true;
    }
}
/* critical section */
turn = j;
flag[i] = false;
~~~

### Mutex - Peterson's algorithm

Peterson found a more simple and elegant algorithm.

~~~c
/* using j instead of i hat */
flag[i] = true;
turn = j;
while (flag[j] && turn == j) {}
/* critical section */
flag[i] = false;
~~~

### Mutex - Using hardware support

 - Uniprocessor: mutex achieved by disabling processes from being interrupted. Used extensively in many OSes. Forbidden to user programs.
 - SMP systems: special instruction, e.g. IBM has `TEST AND SET` which reads a bit of memory and then sets it to 1, atomically. Easy mutex, have a variable `token` and process grabs `token` using test-and-set:

    ~~~c
    while (test-and-set(token) == 1) {}
    /* critical section */
    token = 0;
    ~~~
   
    This is still busy-waiting, deadlock is possible and if low priority grabs the token, high priority pre-empts and can wait forever.

### Semaphores

A semaphore is a special (integer) variable which can be accessed only by the following operations:

 - `init(s, n)`: create semaphore and initialise it to non-negative value `n`
 - `wait(s)`: semaphore value decremented; if value negative calling process is blocked
 - `signal(s)`: semaphore is incremented; if value non-positive one process blocked on `wait` is unblocked.

Traditionally, `P` and `V` are used for `wait` and `signal`.

Semaphore could be:

 - strong: waiting process are released FIFO; more useful, generally provided
 - weak: no guarantee about the order; not used here

__Binary semaphore__ - takes on values 0 and 1. `wait` decrements 1 to 0 or blocks if 0 already. `signal` unblocks, or increments from 0 to 1 if no blocked processes.

Advantage of semaphores - The mutex problem are confined inside just two system calls. User programs do not need to busy-wait; only the OS busy-waits (for a short time).

Using semaphores

~~~c
wait(s);
/* critical section */
signal(s);
~~~

When semaphore is initialised to `m` instead of `1` then `m` processes run at the same time.

### The Producer-Consumer problem

A _producer_ repeatedly puts items into a buffer and a _consumer_ takes them out. The problem is to make this work without delaying either party. Solution using two semaphores `init(n, 0)` (tracks number of items in buffer) and `init(s, 1)` (used to lock the buffer):

__Producer loop__

~~~c
datum = produce();
wait(s); // wait until can add to buffer
/* critical section */
append(buffer, datum);
signal(s); // done with buffer
signal(n); // added another item to be consumed
~~~

__Consumer loop__

~~~c
wait(n); // wait for items in buffer
wait(s); // wait until can extract from buffer
/* critical section */
datum = extract(buffer);
signal(s); // done with buffer
consume(datum);
~~~

### Monitor

Semaphores have `wait` and `signal` separated in code - hard to understand. A _monitor_ is an object which provides some methods (protected by mutex) so only one process can be 'in the monitor' at a time. Monitor variables are only accessible from monitor methods.

 - `cwait(c)`: where `c` is a _condition variable_ confined to monitor; process is suspended and the monitor released for another process.
 - `csignal(c)`: some process suspended on `c` is released and takes the monitor.

Unlike semaphores, `csignal` does nothing if no process is waiting.

__Advantage of monitors__ - monitors enforce mutex and all the synchronisation is inside the monitor methods where it's easier to find and check.

### The Readers/Writers Problem

Resource which can be read by many processes at one but any read must block a write. It can be written by only one process at once, blocking everything else. Can be solved using semaphores. Who has priority?

 - Unix file locks
 - OS/390 ENQ syscall provides general purpose read/write locks
 - Linux kernel uses read/write semaphores internally

### Message Passing

Many systems provide message passing services. Processes may `send` and `receive` messages from each other. `send` and `receive` may be blocking or non-blocking when there is no receiver waiting or no message to receive. Most usual is non-blocking `send` and blocking `receive`.

Can be used for mutex and synchronisation:

 - simple mutex by using a single message as a _token_
 - producer/consumer: producer sends data as messages to consumer; consumer sends null messages to acknowledge them

### Deadlock

Permanent blocking of two or more processes in a situation where each holds a resource the other needs but will not release it until after obtaining the other's resource.

Process P

~~~c
acquire(A);
acquire(B);
release(A);
release(B);
~~~

Process Q

~~~c
acquire(B);
acquire(A);
release(B);
release(A);
~~~

Another instance is when two processes are each waiting for the other to send a message.

__Preventing a deadlock__

3 facts need to be true for deadlock to happen

 - resources are held by only one process at a time
 - a resource can be held while waiting for another
 - processes do not unwillingly lose resources

If any of these does not hold, deadlock does not happen. If they are true, deadlock may happen if

 - a circular dependency arises between resource requests

The first three can be prevented from holding but not practically. The fourth can be prevented by ordering resources and requiring processes to acquire resources in increasing order.

__Avoiding a deadlock__

A more refined approach is to deny resource requests that might lead to a deadlock. This requires processes to declare in advance the maximum resource they might need. Then, when a process requests a resource, _analyse_ whether granting the request might result in a deadlock.

The analysis is done as follows: if we grant the request, is there sufficient resource to allow one process to run to completion? And when it finishes can we run another? And so on.. If not, we should deny the request.

__Deadlock detection__

There are techniques to detect whether a deadlock exists. Then we can:

 - kill all deadlocked processes
 - selectively kill deadlocked processes
 - forcibly remove resources from some processes
 - if checkpoint-restart is available, roll back to pre-deadlock point and hope it doesn't happen again

## Memory Management

 
