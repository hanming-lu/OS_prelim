# OS Review 

## Intro

- Definition of OS: a special layer of software that provides application software access to hardware resources

## Basic Concepts

<u>Four Fundamental OS Concepts</u>

1. Thread - concurrency
   - execution context
   - fully describes program state
   - Program counter, registers, execution flags, stack, memory state
2. Address space
   - memory addresses accessible to program
3. Process - protection
   - An instance of executing program
   - Protected address space + 1 or more threads
   - owns address space
   - owns file descriptors, file system context
   - encapsulate 1 or more threads using shared process resources
   - Easy communications within a process; hard between processes
     - multiple threads in a process? 1) hardware parallelism; 2) ease of handling I/O and other simultaneous events
   - Reliability - crashes process only; security and privacy - protected memory; fairness - shares of disk/CPU
4. Dual mode operation / protection
   - Only the OS has access to certain resources
   - Combined with translation, isolates program from each other and OS from programs
   - Hardware provides two modes: 1) Kernel Mode; 2) User Mode
   - User mode cannot: changing page table pointer, disable disruptions, write to kernel memory, interact directly with hardware
   - Change between modes: system calls, interrupts, exceptions

- User -> Kernel mode methods
  1. syscall
  2. interrupt - hardware
  3. exceptions - software

## Abstractions

<u>Motivation for threads</u>

1. Performance
2. User responsiveness
3. Hide latency

<u>Thread state</u>

- Shared state:
  1. Global variables, heap
  2. I/O state: file descriptors, network connections
- Private state:
  1. Kept in TCB (Thread control block)
  2. CPU registers: Program counter, etc
  3. execution stack: parameters, temporary variables
- Thus, in an address space:
  - 1 heap, global data, code
  - Multiple stacks, sets of CPU registers

- Socket: An abstraction of an endpoint in a network connection

## Synchronization

- Scheduling policies provides:
  - fairness or
  - realtime guarantees or
  - latency optimization or

| (no parallelism)          | Processes | Threads |
| ------------------------- | --------- | ------- |
| Switch overhead           | high      | low     |
| Protection                | high      | low     |
| Shared resources overhead | high      | low     |
|                           |           |         |

Lock using interrupts:

- Disable interrupts in lock acquire and release
- Interrupts enabled after asleep (responsibility of next running thread)

Lock using test&set:

- 

## Scheduling

<u>Scheduling Goals</u>

- Fairness
  - Share CPU among users in an equitable way
- Minimize response time
  - time spent to respond
- Maximize throughput
  - operations or jobs per second

<u>Scheduling Policies</u>

- FIFO
  - Bad for small tasks, long response time
- Round robin
  - Context switch overhead adds up for long tasks
- Strict Priority scheduling
  - always execute highest-priority runnable jobs to completion
  - Priority inversion: a low-level job holds a lock that a high-level job acquires

- Tradeoff: fairness and avg response time
- Shortest Remaining Job first
  - run the shortest remaining job available
  - Optimal avg response time
  - unfair
  - unachievable since needs to predict future
- Lottery scheduling
  - Each job gets several tickets, lottery for who goes next
  - small jobs get more tickets, large jobs get fewer
- Multi-level Feedback scheduling
  - multiple priority queues, each with a scheduling policy
  - Jobs start at the highest priority queue
  - if times out, move down a priority; if not, move up a priority (or to the top)
  - Policies between queues: priority or time slice

<u>Real-time Scheduling</u>

- Earliest deadline first (EDF)
  - schedules the task with earliest deadline

<u>Linux Completely Fair Scheduler</u>

- Fair fraction of CPU
- Track CPU time per thread and schedule threads to match up average rate of execution
- Use weight to achieve different runtime

<u>Requirements for deadlock</u>

1. Mutual exclusion
2. hold and wait
3. no preemption
4. circular wait

<u>Deadlock Solutions</u>

1. Deadlock prevention
   - prevent it from happening
   - allocate all at once
   - allocate in a certain order
2. Deadlock recovery
   - let it happen, recover from it
   - terminate thread
   - preempt resource from thread
   - roll back what deadlock threads have done
3. Deadlock avoidance
   - dynamically delay resource request so deadlock won't happen
   - OS checks if it can result in unsafe state (i.e. deadlock may happen)
4. Deadlock denial
   1. ignore deadlock

## Memory

- Virtual memory
  - isolation and protection
- Memory Management Unit (MMU):
  - translate from virtual memory to physical memory
- Multi-level Paging
  - Pros:
    - only allocate as many page table entries as we need
    - easy translation
    - easy sharing
  - cons:
    - one pointer per page
    - page tables need to be contiguous
    - two lookups per reference
- A process cannot modify its own page table translations
- TLB: translation look-aside buffer: store virtual page to physical frame translation
- Sources of cache misses:
  - compulsory: first time access
  - capacity: full so evict
  - conflict: multiple memory addresses to the same cache
  - coherence: write to other cache 

## I/O

- Programmed I/O
  - Each byte transferred via processor in/out or load/store
  - simple
  - consumes CPU cycles
- Direct Memory Access
  - Give controller access to memory bus
  - no CPU cycles needed
- Notify OS using
  - I/O interrupt
  - OS polling

## Filesystem

- Four components of a file system
  1. Directory
     - A special file
     - A list of \<file_name:file_number> mappings
  2. Index structure
  3. Storage blocks
  4. Free space map
- Buffer cache
  - OS allocated cache for faster file system operations
- Important characteristics
  - Availability: up and running
  - durability: recover from crashes
  - Reliability: up and running under stated conditions

<u>Reliability Approaches</u>

1. careful ordering and recovery
2. versioning and copy-on-write

<u>CAP Theorem</u>

- Consistency
- Availability
- Partition-tolerance