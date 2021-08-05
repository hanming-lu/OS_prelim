# 2021 Prelim Exam 

## 101 - UNIX

#### Overview

- UNIX is a general-purpose, multi-user, interactive OS
- A hierarchical file system with demountable volumes
- compatible file, device, and inter-process I/O
- support async process
- command language per user
- characteristics are simple, elegant, easy-to-use

#### The File System

- Three kinds of files: ordinary disk file, directories, and special files

<u>Ordinary file</u>

- a file contains whatever information the user places on it, no particular pattern is expected

<u>Directory</u>

- A directory is a mapping between names of files to the files themselves, thus induce a structure as a whole
- A directory behaves just like an ordinary file, except that unprivileged programs cannot write
- The system uses a root directory, and a command directory
- Linking: different names for the same file under different directories
- A file does not exist in a particular directory, but only a pointer to the file is included in a directory
- Directory structure is restrained to be a rooted tree, where a directory must only be in one another directory, which is its parent.

<u>Special Files</u>

- Each I/O device is associated with at least one special file
- Write and read ops to special files are exactly the same as an ordinary file, but they result in activation of the associated device
- Three advantages:
  - Files and device I/O are as similar as possible
  - File and device names have the same syntax and meaning, can be used as parameters
  - Special files have the same protection as ordinary files

<u>Removable File Systems</u>

- A removable volume which has the structure of an independent file system containing its own directory hierarchy
- mount can be used to replace a leaf of the hierarchy tree (an ordinary file) by a whole new subtree
- The treatment of a mounted file system is the same as the ordinary file system
- One exception: no link may exist between two file system hierarchies

<u>Protection</u>

- 7 bits: 
  - 6 independently specify read, write, execute permission for the owner of the file and all other users
  - 7th bit specifies if the user id is temporarily changed to the owner of the file during the file's execution

<u>I/O Calls</u>

- No distinction between random and sequential access
- No logical record size imposed by the system
- open(name, flag) -> returns a file descriptor (an int)
- create -> create a new one or truncate an old one to zeros
- No user-visible locks, no restrictions on the # of users having a file opened
- ==View locks as neither necessary nor sufficient to prevent interference between users of the same file==
  - Unnecessary: not a large, single-file database where multiple processes operate on
  - Insufficient: ordinary locks cannot prevent confusion when multiple users operate on the same file, where each makes a copy of the file being edited
- Read and write read/write a sequential of data from/to file
- seek -> moves pointer to a random spot, then read/write

#### Implementation

- A directory entry contains the name of the file and a pointer (an int)
- The pointer is an integer called i-number of the file
- The i-number is an index of the file into a system table (the i-list)
- The entry found is called i-node of the file, containing metadata of the file: owner, protection bits, address for the file contents, size, time of last modification, # of links to the file, dir bit, special file bit, large or small bit
- When a new file is created, an i-node is allocated for it and a directory entry is made from the path to it
- Removing a path is done by decrementing the i-node's link counter, and removing directory entry
- disk is divided into 512-bytes blocks
- Read and write seem to be synchronous and unbuffered, but they are actually buffered, where content are checked first if they are in a buffer, if not, then fetch from disk. Still, writes are async as the write call returns right after value is written in the buffer.

#### Processes and Images

- **Image**: a computer execution environment, which specifies the current state of a pseudo computer. It includes a core image, general register values, status of open files, current directory, etc.
- **Process**: a process is the execution of an image. While the processor is executing on behalf of a process, its image is in core
- User-core part of the image is divided into three logical segments:
  1. text segment: read-only, begins at location 0
  2. heap: write/read, above the program text segment, non-shared, data segment
  3. stack: starts at the highest address, automatically grows downward as the hardware's stack pointer fluctuates

<u>Processes</u>

- a new process is created using fork
- parent/child have exactly the same original core image
- use return id to identify themselves

<u>Pipes</u>

- pipe -> returns a file descriptor, creates an interprocess channel called pipe
- pipe is passed in fork call
- read/write are used to pass data between the images of the two processes

<u>Execute</u>

- execute -> requests the system to read in and execute the program
- all the code and data in the process using execute is replaced from the file, but open files, current directories, and interprocess relationships are unaltered

<u>Process synchronization</u>

- wait -> caller suspends execution until one of its children has completed execution

<u>Termination</u>

- exit -> terminates a process, destroys its image, closes open files

#### The Shell

- communication with UNIX is done through the Shell
- commands are found and brought into core and executed by the Shell

<u>Standard I/O</u>

- Programs executed by the Shell start off with file 1/0, which are the standard output/input files

<u>Filters</u>

- vertical bar | is used to pipeline outputs to next command's input
- filters can be applied

<u>Command Separators: Multitasking</u>

- & is used to run tasks in the background, so the command returns immediately

<u>Implementation of the Shell</u>

- The Shell waits for new-line char
- when received, arrange arguments and fork
- child runs execute
- parent waits the child to die, and then return its value

<u>Initialization</u>

- init -> creates the very first process, called as the last step in UNIX's initialization
- init creates one process for each typewriter channel which may be dialled up up by a user
- After authenticating a user, the process performs execute and wait for command input
- The mainstream path waits on the Shell process

#### Traps

- Hardware detects program faults, terminates the process, and writes the user's image on a file for debugging purpose

#### Questions

1. What problem does this paper address?
   - The paper addresses the problem of a general-purpose interactive OS that is simple yet powerful, allowing users to easily involve
2. What is the author's main insight?
   - Minimalist design, unified abstractions (everything is a file)
   - Take the view that everything can be seen as a file: string of data, indexing structure, device I/O
3. What are the paper's key strengths?
   - Powerful, time-sharing system
   - Interactive so addictive
   - Easily portable as it is written in high-level language (C)
   - Open-source, people help it evolve
   - Instead of inventing something new, UNIX does a careful selection of mechanisms used in the system, which produces a simple, elegant, and easy-to-use OS
4. What are the paper's key weaknesses?
   - Fixed block size not optimal for all apps but minimizes system overhead

## 102 - System R

#### Overview

- Data independence: a data transparency where user applications should not be affected by the database's in-storage structure and access strategy
- Modern databases provides data independence by offering a high-level user interface through which user applications manage stored data, instead of various bits, pointers, arrays, etc.
- Relational Database properties:
  1. all information is represented by data values, no "connection"
  2. The system supports a very high-level language, user can specify query without specifying algorithms for processing
- System R is an experimental relational database that demonstrates it can support both high performance and complete function required for daily production use

#### Goals of System R

1. Provide a high-level non-navigational interface for maximum user productivity and data independence
2. Support different database usages: programmed transactions, ad hoc queries, report generation
3. Support rapidly changing database environment, where tables, indexes, views, transactions, and objects can be added and removed easily
4. Support concurrency, protect integrity in a concurrent-update environment
5. Database recovery to a consistent state
6. Provide a flexible mechanism whereby different views of data can be managed by various users
7. Support all functions above with a level of performance comparable to existing lower-function database systems

#### Phase Zero lessons

- Store relations in the form of "tuples", where a tuple store data values
  - key advantage: all the data values of a record could be fetched by a single I/O
- inversion: associates domain values with the TIDs of tuples
- Optimizer algorithm: 
  1. cost of fetching tuples
  2. cost of creating and manipulating TID lists
  3. cost of fetching the data pointed to by the tuples
- cost measured as the number of I/Os + CPU time
  - cluster tuples on physical pages
- Emphasize simple interactions instead of complex queries

#### Phase One lessons

- System R consists of an access method (RSS) and an optimizing SQL processor (RDS), modularity design
  - All locking and logging functions are in RSS
  - All authorization and access path selection functions in RDS
- View subsystem: users to define alternative views of the database
- Authorization subsystem: only access the ones granted
- Recovery subsystem: restored to a consistent state after failure

<u>Query Optimization:</u>

- SQL statements of arbitrary complexity could be decomposed into a relatively small collection of machine-language "fragments"
- In R, SQL statements are examined, optimized, and compiled into small, efficient machine-language routings

<u>RSS Access Paths</u>

- index: associate indexed values to records, similar to "inversion"
- link: a pointer stored with record which connect it to other related records
- Access paths available:
  1. index scans: access a table associatively and scan it in value order using an index
  2. relation scans: scan over a table as it is laid out in physical storage
  3. link scans: traverse from one record to another using links

<u>Optimizer</u>

- Optimizer minimizes the weighted sum of cost of I/O and CPU time

<u>Recovery Subsystem</u>

- Disk failure
  - information on disk is lost
  - recover by an alternative disk
- System failure
  - information in main memory is lost
  - the system reverts to the old pages, use the log to undo all updates by incomplete transactions
- Transaction failure
  - need to undo changes by the transaction
  - revert log and undo
  - takes place on-line

<u>Locking Subsystem</u>

- A hierarchy of locks, with several different sizes of lockable units
- Lock sizes range from individual records to several tables
- locking is transparent to end users
- When having a lock on small object, an "intentional" lock is required on its larger object
- When locks on smaller objects accumulate, they can trade for a lock on larger object
- Three levels of isolation
  1. level 1: may read uncommitted data. No lock acquired
  2. level 2: cannot read uncommitted data, but successive read can give inconsistent value. Lock when read, release immediately after read.
  3. level 3: guaranteed consistent successive reads return the same value. Lock held until the end of the transaction.

<u>Compilation Approach</u>

- The approach of compiling SQL statements into machine code is successful
- Can generate machine-language routine to execute any SQL statements
- Beneficial for short, repetitive transactions
- Small cost for database compared to writing transactions on the RSS interface

#### Questions

1. What problem does this paper address?
   - Can a relational database provide the same level of performance as a lower-level database, while providing easy-to-use, high-level language interface, data independence?
2. What is the author's main insight?
   1. Everything is a table
   2. Store values only, no connections/pointers
   3. separate into RDS and RSS, modularity design
   4. Locking system where locks on individual records
   5. shadow page + WAL for recovery
   6. Compile SQL statements into machine code
   7. Use CPU time + # of I/O for optimization
   8. Indexing used for access path
3. What are the paper's key strengths?
   1. Similar performance as lower-level database
   2. Allow evolvability (data independence)
   3. Top-down approach where easy-to-use for programmers (SQL statements)
4. What are the paper's key weaknesses?
   1. Could be slower for simple imperative queries
   2. SQL is declarative programming, it is hard to express certain things in declarative form

## 103 - Architecture of a Database System (119 pages, [blog summary][https://blog.acolyer.org/2015/01/20/architecture-of-a-database-system/])

#### Overview

1. Process models
2. Parallel architecture
3. Storage system design
4. transaction system implementation
5. Query processor and optimizer architectures
6. Typical shared components and utilities

#### Process Models

- Three  process model options:
  1. Process per DBMS worker
  2. Thread per DBMS worker
  3. Process pool (a variant of process-per-worker, bounded pool of workers)
- Move data from DB to client through:
  - disk I/O buffers: some shared memory
  - client communication buffers: use client communications socket as a queue for the tuples it produces (for prefetching)
  - lock table: shared by all DBMS workers and is used by the Lock Manager
- Log:
  - generated during transaction processing
  - log tail: log staged to an in-memory queue, periodically flushed to disk in FIFO order
- Admission control:
  - admission control policy for multi-user systems
  - Does not accept new work unless sufficient DBMS resources are available
  - graceful degradation: under overload, latency increases but throughput remain at peak
  - Two tiers:
    1. front door: limit the number of client connections
    2. within the core DBMS relational query processor: after the query is parsed and optimized, then determines if a query is postponed or with fewer resources

#### Parallel Architecture

- <u>Shared-memory</u> parallel system: 
  - all processors can access the same RAM and disk with roughly the same performance
  - advantage:
    - simple, just like a uniprocessor DBMS
  - challenge:
    - how to modify the query execution layers to take advantage of the ability to parallelize a single query across multiple CPUs
    - scalability
- <u>Shared-nothing</u> parallel system:
  - a cluster of independent machines that communicate over a high-speed network interconnect or over commodity networking components
  - each table is sliced horizontally, spread across the machines
  - common today
  - advantage:
    - scalability
  - challenge:
    - cross-processor coordination
    - partial failure
    - cost
- <u>Shared-disk</u> parallel system:
  - all processors can access the disks with the same performance, unable to access each other's RAM
  - advantage:
    - lower cost of administration compared to shared-nothing
    - no partial failure on nodes
  - challenge:
    - single point of failure
    - corrupt page even if RAID
    - cross-processor coordination 

#### Relational Query Processor

- relational query processor: takes a declarative SQL statement, validates it, optimizes it into a procedural dataflow execution plan, and executes that dataflow program on behalf of a client program

<u>Main Tasks for an SQL Parser:</u>

1. Check the query is correctly specified
2. resolve names and references
3. convert to internal format which optimizer understands
4. verify the user is authorized to execute the query

<u>Query rewrite module</u>

- responsible for simplifying and normalizing the query without changing its semantics
- can only rely on the query and metadata, no access to tables
- e.g. view expansion, semantic optimization

<u>Query optimizer</u>

- transform the internal query representation into an efficient query plan for execution
- A query plan is a dataflow diagram that pipes table data through a graph of query operators

<u>Query executor</u>

- executes a query plan, mostly use iterator model

<u>Versioning</u>

- if versioning is used with timestamp, then queries of the same historical time will provide compatible answers
- no read locks needed

#### Storage Management

- DBMS has more information on its workload access patterns (compared to OS), DBMS architects should have full control over the spatial positioning of database blocks on disk
  - An alternative is to create very large files in the OS file system
  - DBMS should have control over when to physically written to the disk
- Throughput in a well-tuned DBSM is typically not I/O bound, but bounded by memory

#### Transactions, Concurrency Control, and Recovery

<u>ACID</u>

- Atomicity: transaction is atomic, either commit or abort, no partial changes
- Consistency: the state is consistent before and after changes
- Isolation: transactions are not interfering each other
- Durability: data stored can survive crash failures

<u>Implementation of ACID</u>

- Isolation: locking protocol
- Durability: logging and recovery
- Isolation and atomicity: combination of locking and logging
- consistency: runtime checks in the query optimizer

<u>Concurrency control enforcement</u>

- Strict two-phase locking: acquire lock before read/write, release at the end of transaction
- Multi-version concurrency control: no locks, guaranteed a consistent view of the database state at some time in the past
- optimistic concurrency control: no locks, just read/write. keep track of values read/write, if some modified before commit, roll back one of the conflicting transactions

<u>Isolation Levels</u>

- Read uncommitted: dirty read allowed, no lock
- Read committed: dirty read not allowed, need a lock to read, but lock released immediately after read
- Repeatable read: need a lock to read, release lock at the end of the transaction
- serializable: as if all transactions are done by one process

<u>Recovery</u>

- Write-Ahead Logging (WAL)
  1. Each modification to a database page needs to generate a log record, and the log record needs to be flushed before the actual database page is flushed
  2. Log records need to be flushed in order
  3. when a commit request occurs, the commit log record needs to be flushed before the commit request returns successfully

#### Shared Components

<u>Metadata format</u>

- Use the same format for metadata as for data

<u>Replication</u>

1. Physical replication: duplicate the entire database periodically
   - Bad scaling
   - Expensive
2. Trigger-based replication: use triggers to store "difference" records in a special replication table
   - High performance penalty for some workloads
3. Log-based replication (the solution if possible): a log sniffer process intercepts log writes and delivers them to the remote system
   - low performance overhead for the running system
   - provides incremental updates
   - use built-in log logic

#### Questions

1. What problem does this paper address?
   - A summary of DBMS design and implementation
2. What is the author's main insight?
   - Key components are process model, SQL parser, storage management, parallel architecture, ACID (Transactions, concurrency, recovery)
3. What are the paper's key strengths?
   - A detailed, comprehensive summary of academic and industry design and implementation
4. What are the paper's key weaknesses?
   - A survey, so none?



## 104 - INGRES

#### Overview

- Built on UNIX, written in C, 

<u>Topics covered</u>

1. The system process structure
2. The embedding of all INGRES commands in C
3. The access methods implemented
4. The catalog structure and the role of the database administrator
5. Support for views, protection, and integrity constraints
6. The decomposition procedure implemented
7. Implementation of updates and consistency of secondary indices
8. Recovery and concurrency control



## 105 - End to End

#### The Argument

- If the function in question can completely and correctly be implemented only with the knowledge and help of end points, then providing the function as a feature of communication system itself is not possible.
- From performance perspective, lower level can put efforts to improve performance, but it doesn't have to be "perfect".
  - If implemented at a lower level, it should not incur costs to services that do not need it

#### Questions

1. What problem does this paper address?
   - When there are various modules, where should we implement different functions?
2. What is the author's main insight?
   - Implement at end points if the function requires the knowledge of end points
   - Implement at a lower level only if performance enhancement for services that require or not require such function
3. What are the paper's key strengths?
   - Provides theoretical basis for implementing functions at end points
4. What are the paper's key weaknesses?
   - It is a principle, not a ground truth. It may vary in different systems

## 106 - FFS

#### Overview

1. More flexible allocation policies: better locality, adaptive to peripheral and processor characteristics
2. Cluster data
3. Two block sizes
4. Programmer interface enhancement: advisory locks on files, name space extension, long file names, admin control of resource

#### Old System

- Low throughput: 
  - small block size, limited read-ahead in the system, many seeks

#### New System

- Disk drive: 
  - each disk drive contains one or more file systems
  - each file system is described by its super-block, that is at the beginning of the file system's disk partition, replicated
- Block size:
  - Minimum block size of 4096 bytes
  - recorded in super-block
- Cylinder group: 
  - disk partition into one or more areas called cylinder group, each consists of one or more consecutive cylinders on a disk
  - contains super-block, inodes, bit map, usage of data blocks
  - bit map: describes available blocks
  - bookkeeping info begins at a offset, better reliability

#### Storage Utilization

- data is laid out so that larger blocks can be transferred in a single disk transaction, more data transferred per disk transaction
- Divide blocks into segments to efficiently store small files
- A file can occupy several blocks and/or several segments
- allocate new blocks/segments when new data is written and already allocated space is insufficient. Copy old data in fragments over if a new block is allocated.
- wasted space for 4096/1024 system is similar to 1024 system
- a file system has a minimum free space reserve to function optimally

#### File System Parameterization

- File system is parameterized to adapt
  - processor capability
  - storage characteristics

- The file system allocates new blocks optimally
  - On the same cylinder as the file's previous block
  - rotationally well placed
  - disk: number of blocks per track, rate at which the disk spins
  - processor: expected time to interrupt and schedule

#### File System Layout Policies

- Two levels:
  - Global policy: use file system-wide summary information to make decisions regarding the placement of new inodes and data blocks
  - Local policy: local allocation routines that use a locally optimal scheme to lay out data blocks
  - Conflicting goals of localizing data for concurrent access vs. spreading out unrelated data
- Two methods to improve performance:
  - Improve locality to reduce seek latency
  - Improve data transfer size to reduce the number of transfers
- inode:
  - inodes of files in the same dir are usually accessed together, place them in the same cylinder group
  - a new dir is allocated in a cylinder group with high free inodes and small number of dir in it -> allow inode allocation policy to succeed most of the time
  - allocate inodes randomly in a cylinder group: small constant disk transfers cap
- data block:
  - place data blocks of a file in the same cylinder group, with rotationally optimal positions
  - to prevent filling up, try to evenly place data blocks of large files
- Global policy uses partial local information to make decisions
- When local policy received a block allocation request from global:
  - allocate if block is free
  - if not, next available block rotationally closest
  - if no block available in the cylinder, use a block in cylinder group
  - if cylinder group full, hash choose another cylinder group
  - if fail, exhaustive search

#### Performance

- inode layout policy effective -> inodes within a dir are clustered
- High utilization of disk bandwidth
- Reads and writes are faster
- blocks of a file are more optimally ordered

#### File System Functional Enhancements

- Long file names
- File locking
  - granularity of a file
  - advisory shared or exclusive locks on files

#### Questions:

1. What problem does this paper address?
   - The original UNIX file system has low throughput, what are the improvements possible?
2. What is the author's main insight?
   - Increase block size while considering small files
   - Parameterize processor and storage for optimization
   - Global and local layout policies to optimally allocate inodes and data blocks
3. What are the paper's key strengths?
   - High throughput
   - Layout policies that consider processor and disk characteristics: locality, large data transfer size
4. What are the paper's key weaknesses?
   - Specialized to disks, hard to port to other medium (e.g. SSD)





## 107 - Analysis and Evaluation of Journaling File Systems

#### Overview

<u>Two methods for analyzing file system behaviour and evaluating file system changes:</u>

1. Semantic block-level analysis (SBA)
   - combines knowledge of on-disk data structures + a trace of disk traffic to infer system behavior
   - enables users to understand why the file system behaves as it does
2. Semantic trace playback (STP)
   - enable modifying traces of disk traffic to represent changes in the file system implementation
   - enable users to measure the benefits of new policies without actually implementing it

<u>Journaling File System</u>

- modern file systems are journaling file system
- Write information about pending updates to a Write-Ahead Log (WAL) before committing changes to disk
- Enables fast file system recovery after a crash

<u>Analysis of Linux ext3, ReiserFS, Linux JFS, Windows NTFS</u>

- focus on journaling aspect
- determine the events that cause data and metadata to be written to the journal or their fixed locations
- examine how characteristics of workload and configurations affect it

#### Semantic Block-Level Analysis (SBA)

<u>Overview</u>

- Controlled workload patterns
- Analysis of both time taken for operations + resulting stream of read and write requests below the file system
- it leverages info on block type (e.g. inode or journal)
- interposes on the block interface to storage

<u>Metrics</u>

- Quantity of read and write requests to disk: caching and write buffering
- Block numbers: sequential or random, where
- when read/written: burstness of traffic

<u>Implementation</u>

- Use SBA driver to interpose between the filesystem and disk
- track request and response by storing in a circular buffer
- SBA requires in interpret contents of disk block traffic: interpret block-level traces on-line

<u>Workload</u>

- Use synthetic workload that uncover decisions made by the file system
- Parameters to vary:
  - Amount of data written
  - Sequential vs. random access
  - Interval between calling fsync
  - Amount of concurrency
- only write-based workloads

<u>Overhead of SBA</u>

- processing and memory overheads are low
- For each I/O request, overhead is:
  - get_time at start/end
  - determine block number is journal or fixed-location block
  - determine journal metadata or journal data

#### Semantic Trace Playback

<u>Overview</u>

- enable developers to rapidly suggest and evaluate file system modifications without actually implementing it
- a user-level process
- takes an input as a trace, parses it, and issues I/O requests to the disk using raw disk interface. Allow threads and concurrency

<u>Requirements</u>

- Observe two high-level activities:
  1. observe any file-system level operations that create dirty buffers in memory
  2. observe application-level calls to fsync: understand why write traffic happens

- STP input: A file system level trace + SBA-generated block-level trace

<u>Advantages</u>

- not time-consuming as actual implementation
- no need for a detailed and accurate simulator

<u>Limitations</u>

- If workload has complex virtual memory behavior whose interactions are not modelled, results may not be meaningful
- Can only evaluate changes that are not too radical
- Does not provide how to implement

#### The Ext3 File System

<u>Overview</u>

- Disk:
  - split into block groups
  - each block group: bitmaps, inode blocks, data blocks
  - ext3 journal stored as a file
- Journal overview:
  - info about pending updates written to the journal
  - force journal updates to disk before updating file system structure
  - recover by scanning through the journal and redo incomplete committed operations
  - Journal treated as a circular buffer; space reclaimed once written to its fixed location in the ext2 structures

<u>Journaling Modes</u>

- Three modes: writeback mode, ordered mode, data journaling mode
- Writeback mode:
  - only file system metadata is journaled
  - data block directly written to disk
  - does not enforce any ordering between journal and disk data writes
  - guarantee consistent file system metadata
  - no guarantee to data block consistency
  - weakest semantics of the three modes
- Ordered journaling mode:
  - only metadata is journaled
  - data writes to disk before the journal writes of the metadata
  - both data and metadata are guaranteed to be consistent after recovery
- Data Journaling mode:
  - journal both data and metadata (written to disk twice)
  - data and metadata written to disk after journal commit
  - both data and metadata are guaranteed to be consistent after recovery

<u>Transactions</u>

- ext3 groups many updates into a single compound transaction that is periodically committed to disk

<u>Journal Structure</u>

- Use additional metadata to track the list of journaled blocks
- journal superblock: tracks summary info of the journal (e.g. block size, head and tail pointers)
- journal descriptor block: marks the beginning of a transaction and its subsequent journaling blocks, including their final fixed on-disk location
  - the descriptor block is followed by medata only / data + metadata in writeback and ordered journaling / data journaling mode
  - journal logs full blocks (instead of diff): if a single bit in the bitmap changes, its entire block is logged
- journal commit block: end of the transaction; once written, the journaled data can be recovered without loss

<u>Checkpointing</u>

- in ext3, a basic form of redo logging is used
- scans the log for committed complete transactions, discard incomplete transactions

<u>Analysis of ext3 with SBA</u>

- Modes and Workload
  - random writes have better performance with logging vs. without
  - sequential higher bandwidth than random
  - fewer fsync call -> higher bandwidth
  - small cost of journaling
  - writeback & ordered -> similar performance to no logging
  - performance of data journaling is irregular -> sawtooth pattern
  - Concurrent synchronous unrelated transactions can result in very low bandwidth
- Journal Commit Policy
  - impacted by settings of the commit timers
    - flush metadata frequently, flush data slowly
  - enforce disk write order by modes

<u>Improve ext3 with STP</u>

- place journal in the middle -> less seek time
- adaptive journaling mode: sequential writes -> ordered journaling; random writes -> data journaling
- separately flush unrelated transactions to disk
- differential journaling

#### Questions

1. What problem does this paper address?
   - How to accurately measure a file system's usage of a disk?
   - How to gauge the performance differences if there are modifications to the system?
2. What is the author's main insight?
   - Use a block-level analysis tool that interposes the file system and disk
   - Understand what is truly happening from the disk's point of view
   - Use synthetic workload to gauge what will happen if certain system changes are made
3. What are the paper's key strengths?
   - Help understand why the file system is performing as it is
   - Gauge performance changes of system modifications
4. What are the paper's key weaknesses?
   - STP not applicable to major system changes
   - SBA/STP cannot give insight on how to change it

## 108 - Log-Structured File System

#### Overview

- All modifications are stored to disk sequentially in a log-like structure
  - speeds up both small file writing and crash recovery
  - almost no seek time for writes
  - log contains indexing info for efficient reads
  - Faster small-file writes, others similar
- Segment:
  - divide log into segments
  - use a segment cleaner to compress the live information from heavily fragmented segments
- Cleaning policy:
  - segregates older, more slowly changing data from young rapidly-changing data; tread them differently during cleaning
- Challenges:
  - How to retrieve information from log: indexing
  - Ensure there are always large extents of free space available for writing new data: segment cleaning

#### Problems with existing file systems

1. Too many small accesses: cuz spread information around the disk
2. Synchronous writes: for small files traffics, synchronous metadata writes dominate disk traffic, unable to benefit from fast CPU

#### Log-Structure File System

<u>Fundamental Idea</u>

- To reduce # of disk I/O: buffer small writes in main memory, then write all of them to disk sequentially in a single disk I/O
- Information to write includes: file data block, attributes (inode), index blocks, directories, all other info to manage file system

<u>Inode</u>

- Inode is written to log (instead of at a fixed-location in UNIX FFS)
- inode map: maintain current location of each inode
  - given a file descriptor, inode map must be indexed to determine the disk address of the inode
  - divided into blocks, written to log
  - a fixed checkpoint region stores location of all inode map blocks
  - inode map usually cached in main memory and does not require disk access

#### Free Space Management: Segment

- A disk is divided into segments for management

<u>Goal</u>

- Maintain large free space for writing new data
- Manage small segments at random locations

<u>Threading</u>

- leave the live data in place, thread the log through the free extents
- cause free space to become severely fragmented
- afterwards, large continuous writes are not possible, no faster than traditional file systems

<u>Copying</u>

- Copy live data out of the log
- live data written back in a compacted form at the head of the log
- costly, especially for long-lived files. Need to copy long-lived files in every pass of the log.

<u>Sprite LFS</u>

- A combination of  threading and copying
- The disk is divided into segments
- Always write to segment from its beginning to its end
- All live data needs to be copied out of a segment before rewriting to the segment
- Can thus compact long-lived files to certain segments and skip them during copying
- Segment is large enough so can use full bandwidth

<u>Segment Cleaning</u>

- segment cleaning: the process of copying live data out of a segment
- three step: 1) read a number of segments into memory; 2) identify live data; 3) copy live data compactly into a smaller number of cleaner segments
- Use segment summary block to identify which file this block belongs to and its block number -> inode points to new block
- Clean cold segments more frequently than hot segments

#### Crash Recovery

<u>Checkpoint</u>

- a position in the log at which all of the file system structures are consistent and complete
- Two-phase process to create a checkpoint
  1. Writes out all modified info to the log (i.e. file data blocks, indirect blocks, inodes, inode map, segment usage table)
  2. writes a checkpoint region to a fixed position on disk (i.e. address of inode map blocks, segment usage table, current time, last segment written)
- checkpoint periodically

<u>Roll-forward</u>

- After reading checkpoint, scan through the log segments
- new inode -> update inode map
- adjust utilization in the segment usage table
- Directory operation log: used to persist directory changes

#### Evaluation

- Temporal locality: data written at similar time are closer
- LFS handles random small writes more efficiently
- slower sequential reads
- similar random reads
- Cleaning cost is reasonable

#### Questions

1. What problem does this paper address?
   - Disk performs poorly with random small write workloads, how to optimize them with reasonable overheads?
2. What is the author's main insight?
   - Store everything in a log
   - random writes become sequential writes
   - buffer random/sequential small writes to a large sequential write
   - Use segment cleaning to ensure a large contiguous space to write log
   - Use segments, both threading and copying to achieve efficient segment cleaning
3. What are the paper's key strengths?
   - Very efficient random small writes
   - Reasonable overheads
4. What are the paper's key weaknesses?
   - Overheads may not be worthwhile for workloads of large sequential writes

## 109 - AutoRAID

#### Overview

- A two-level storage hierarchy
- upper level: two copies of active data, provide full redundancy, excellent performance
- lower level: RAID 5 parity protection, inactive data, excellent storage cost, somewhat lower performance
- Automatically and transparently manages migration of data blocks between the two levels as access patterns change
- Easy-to-use, suitable for a wide variety of workloads, largely insensitive to dynamic workload changes

#### Background

- Different RAID levels have different performance characteristics and perform well only for a narrow range of workloads

#### A Managed Storage Hierarchy

<u>Assumptions</u>

1. Data needs to have active and inactive parts (o/w cost performance will reduce to that of mirrored data)
2. Active subset must change relatively slowly over time (o/w resources wasted on switching between levels instead of doing work)

- Studies show these conditions are usually met in practice

<u>Implementation Approaches</u>

1. Manually, the system admin:
   - pro: human intelligence, extra knowledge. 
   - con: error prone, cannot adapt changing access patterns, require highly skilled people
2. File system, perhaps a per-file basis:
   - pro: balance between knowledge and implementation freedom
   - con: too many file systems, hard for deployment
3. Smart array controller, behind a block-level device interface:
   - pro: very easily deployable
   - con: knowledge about files has been lost

#### Feature Summary

<u>Mapping</u>

- Host block addresses are internally mapped to their physical locations. Allows transparent migration of individual blocks

<u>Mirroring</u>

- Active data are mirrored for best performance and provide single-disk failure redundancy

<u>RAID 5</u>

- Write-inactive data are stored in RAID 5 for best cost capacity, somewhat lower read performance, single-disk redundancy
- Large sequential writes go directly to RAID 5 -> benefit from high bandwidth for this access pattern

<u>Adaptation to Changes in the Amount of Data Stored</u>

- Initially the array is empty
- As data are added, internal space is allocated to upper level until no more data can be stored this way
- Some of the storage space is automatically reallocated to the RAID 5 storage class, data are migrated down into it from the mirrored storage class

<u>Adaptation to Workload Changes</u>

- When active set of data changes, newly active data are promoted to mirrored storage
- less active data are demoted to RAID 5
- Keep the amount of mirrored data roughly constant

<u>On-Line Operations</u>

- online storage capacity expansion
- easy disk upgrades
- controller fail-over
- active hot spare

<u>Log-Structured</u>

- Writing to RAID 5 in a log-structured fashion to avoid reading old-date or old-parity for better performance

#### Data Layout

<u>Physical EXtents (PEXes)</u>

- disk is broken up into PEXes
- 1MB in size
- Several PEXes combined to make a Physical Extent Group (PEG)

<u>Physical Extent Groups (PEGs)</u>

- 1 PEG consists of several PEXes
- PEXes are allocated to PEGs to 1) balance the amount of data on the disks; 2) retain the redundancy guarantees

<u>Segment</u>

- Unit of contiguous space on a disk
- PEX divided into a set of 128KB segments

<u>Relocation Blocks (RB) - Logical Data Layout</u>

- logical space provided by the array is relocation block
- basic unit of migration
- Size of RB
  - smaller -> more mapping info to record, increase seek and rotation time
  - larger -> increase migration costs if only small amount of data updated in each RB

#### Operations

<u>Read Requests</u>

- Similar to the original RAID

<u>Write Requests</u>

1. Data is first written to non-volatile memory
2. When flushed to disk, promotion of RB if it is not in RAID 1
3. If not enough space, demote least-recent-written RB from RAID 1 to RAID 5
4. If thrashing happens (working set too large), do not promote working RB

<u>Background Migration</u>

- Goal: maintain enough empty RB slots in RAID 1 storage
- Demote least-recent-written RBs

<u>Log-Structured RAID 5</u>

- No need to read old data to update data
- Eliminate RAID 5's small write problems

#### Issues

- Write goes to 2 disks for upper level write -> only need to wait for NVRAM
- Compaction: clean RAID5 and plug holes in the mirrored disks

#### Questions

1. What problem does this paper address?
   - How to design an easily deployable automatically tuned RAID storage system that achieve both performance and capacity?
2. What is the author's main insight?
   - Two levels: RAID 1 and RAID 5
   - Implement at smart array controller level to achieve the highest deployability
   - Automatic compaction, migration, and balancing between two levels
3. What are the paper's key strengths?
   - Able to achieve both performance and capacity
   - Automatically adjusted to workloads
4. What are the paper's key weaknesses?
   - Lacking information on files/applications, may not be the optimal solution

## 110 - Lightweight Recoverable Virtual Memory

#### Overview

- Recoverable Virtual Memory: regions of a virtual address space that provides transactional guarantees.
- RVM: an efficient, portable, and easy-to-use implementation of recoverable virtual memory
- Enable independent control over the transactional properties of atomicity, durability, and consistency
- RVM is a balance between system-level concerns of performance and functionality, and software engineering concerns of usability and maintenance
- Minimalist design

#### Design

<u>Functionality</u>

- Layered approach
- RVM only handles atomicity, process failure
- Removes nesting, distribution, concurrency, resiliency to media failure

<u>OS Dependence</u>

- Only depend on small, widely supported OS interface
- No tight coupling with the OS
- RVM is the building block for meta-data in systems
- External data segment: 
  - RVM's backing store for a recoverable region, is completely independent of the region's VM swap space
  - A system file or a raw disk partition

<u>Structure</u>

- Make RVM as a library that is linked in with an application
  - Trust application and RVM will not damage each other
  - Applications cannot share a single WAL on a dedicated disk
- Each process using RVM has a separate log
  - Can be in a Unix file or on a raw disk partition
  - If a file, RVM uses fsync to flush

#### Architecture

<u>Segments and Regions</u>

- Recoverable memory is managed in segments
- Backing store for a segment is the external data segment (file or raw disk partition)
- region: a segment is divided into regions
  - corresponds to a related collection of objects
  - can be as large as the entire segment
- Applications explicitly map regions of segments into their virtual memory
  - RVM ensures the newly mapped data represents the committed image of the region
  - Copy when a region is mapped
- Copying of data from external data segment to virtual memory happens when a region is mapped (instead of on page demand)
  - Copy the process' recoverable memory as a whole
  - startup is slower
- Segment mapping:
  - no region of a segment may be mapped more than once by the same process
  - mappings cannot overlap in virtual memory
  - Regions can be unmapped at any time

<u>RVM Primitives</u>

- Initialization: specify by a process which log to use by RVM
- Map: once per region to be mapped. Specify which external data segment and the range of virtual memory addresses for the mapping.
- Unmap: unmap a region. The region can be mapped to the process' other virtual address space
- begin_transaction: let RVM a transaction begins, with a transaction id
- set_range: let RVM know that a certain area of a region is about to be modified. RVM then record the current value of the area, in case of abort.
- Read operations on mapped regions require no RVM intervention
- end_transaction/abort: commit/abort a transaction
  - a successful commit guarantees permanence of changes made in a transaction
- no-flush: commits don't flush instantly, trade performance with weaker permenence guarantee
- flush: block until all committed no-flush transactions have been forced to disk
- no-restore: application won't explicitly abort transaction, no undo
- truncate: blocks until all committed changes in the WAL have been reflected to external data segments

#### Implementation

<u> Log Management</u>

- Use no-undo/redo logging: no uncommitted changes reflected to external data segment; only the new-value records of committed transactions have to be written to the log
- Upon commit, old-value records are replaced by new-value records, reflect the current contents of the corresponding ranges of memory, is forced to the log and write out to external data segment.
- No-restore and no-flush are efficient:
  - no-restore needs no copying old-value to prepare for explicit abort, time and space efficient
  - no-flush needs not to flush to external data segment, lower latency

<u>Crash Recovery</u>

1. RVM scans the log from tail to head
2. construct an in-memory tree of the latest committed changes for each external data segment
3. trees are traversed, applying modifications
4. the head and tail location information in the log status block is updated to reflect a new empty log

<u>Log Truncation</u>

1. reclaim space allocated to log entries by applying the changes contained in log to the recoverable data segment

#### Optimizations

- Intra-transaction optimization: redundant set-range calls are ignored
- Inter-transaction optimization: check for modifications to the same address, only log the last one

#### Questions

1. What problem does this paper address?
   - How to enable applications to manage recoverable virtual memory, with clear failure semantics
2. What is the author's main insight?
   - Do one thing well
   - Focus on providing a persistent metadata management only
   - Ignore other features that can be layered on top
   - Use no-undo/redo log
   - use external data segment for persistent storage
3. What are the paper's key strengths?
   - Did one thing well
4. What are the paper's key weaknesses?
   - No demonstration of how ignored features can be layered on top

## 111 - Haystack

#### Overview

- Haystack is an object storage system
- Reduce per photo metadata
- Haystack storage machines can perform metadata lookup in memory, leaving disk bandwidth to actual data transfers
- special workload: written once, often read, never modified, rarely deleted
- Limitation of NFS-based storage:
  - long tail:
    - cache doesn't work for old/unpopular photos
  - Linear directory works badly for many photos/directory
    - many disk I/Os for an image
    - directory's block map too big to cache in memory

#### Goals

<u>High throughput and low latency</u>

- Keep all metadata in main memory
  - Reduce metadata per photo
- Achieves high throughput and low latency
  - at most 1 disk operation per photo

<u>Fault-Tolerance</u>

- Replicate each photo in geographically distinct locations

<u>Cost-effective</u>

- Cost 28% less per usable terabyte
- can process 4x more reads per seconds

<u>Simple</u>

- simple, so easier build and deploy

#### Design

- Store multiple photos in a single file

<u>Haystack Store</u>

- persistent storage system for photos
- only component that manages filesystem metadata (used to retrieve photo)
- Store's capacity is divided into physical volumes
- Physical volumes on different machines are grouped to be logical volumes
- Each write to logical volumes is written to all its physical volumes for redundancy

<u>Haystack Directory</u>

- Logical to physical volumes mapping
- logical volume where each photo resides
- logical volumes with free space

<u>Haystack Cache</u>

- internal CDN
- a cache for popular photos
- reduce dependence on external CDNs
- distributed hash table
- Only cache 
  - not from CDN, but directly from client
  - from writable machine (photos are accessed the most right after they are uploaded)

<u>Read Workflow</u>

- Browser requests URL from Web Server, which generates a path using information from Directory
- Browser sends request to Cache, which sends to Store if not cached
- Store looks up the relevant metadata in its in-memory mapping, seeks the appropriate offset in the volume file, reads the entire needle, do some sanity checking, and return

![image-20210625200546365](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210625200546365.png)



<u>Write Workflow</u>

- Web server provides the logical volume id, key, alternate key, cookie, and data to Store
- Each Store machine synchronously appends needle images to its physical volume files, updates in-memory mappings
- Updates to the same needle are appended to the same physical volume, determine the newest update with the highest offset

![image-20210625200703034](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210625200703034.png)

<u>Delete workflow</u>

- Set delete flag in in-memory mapping and volume file

<u>Index File</u>

- Index file: a checkpoint of the in-memory data structures, used to locate needles efficiently on disk
- Used to reboot in-memory mapping quickly
- Since index file is updated asynchronously, there are orphans which are not in the index file during restart -> create a matching index record and append that record to the index file will do the trick

<u>Filesystem</u>

- Use XFS reasons:
  - blockmaps for several contiguous large files can be small enough to be stored in main memory
  - efficient file preallocation

<u>Failure Recovery</u>

- Detection: background task periodically test connection, check availability, attempts to read data from each Store machine. If fails, mark read-only
- Repair: bulk sync from another replica machine

<u>Compaction</u>

- Compact fragmentation due to deletion

<u>Batching</u>

- Batch uploads whenever possible

#### Questions

1. What problem does this paper address?
   - How to efficiently manage photos, need high throughput and low latency
2. What is the author's main insight?
   - Multiple photos in one file (photo offset becomes a metadata) to reduce metadata overhead
   - Reduce photo metadata size and keep all photo mapping in-memory, at most one disk operation per photo
3. What are the paper's key strengths?
   - Considers the specific application of Facebook photo storage, CDN environment, and usage patterns
4. What are the paper's key weaknesses?
   - If the number of photos increases faster than main memory size, then there will be a point where memory is not capable of storing all photo metadata. What to do then?

## 112 - F2FS

#### Flash Storage

- No seek or rotational time
- Latency = queuing time + controller time + transfer time
- Highest throughput -> sequential OR random read

- Data read and written in page-sized chunk (4KB)
- Before writing, entire block needs to be erased (256KB)
  - Cannot overwrite exiting block
- Flash Translation Layer (FTL): translates logical block address to physical block address in flash storage

<u>Limitation</u>

- Frequent random writes to an SSD would incur internal fragmentation, increase SSD's I/O latency, reduce device lifetime

#### F2FS Overview

<u>Flash-friendly on-disk layout</u>

- Three configurable units: segment, section, and zone
- allocates storage blocks in unit of segment from a number of individual zones
- clean in unit of section
- These units align with the underlying FTL's operational units to avoid unnecessary and costly data copying

<u>Cost-effective index structure</u>

- Wandering tree problem: recursive updates of index blocks (data -> direct -> indirect)
- Node address table: index table to attack this problem

<u>Multi-head logging</u>

- Hot/cold data separation scheme during logging time (i.e. block allocation time)
- Multiple active log segments concurrently
- append data and metadata to separate log segments based on their anticipated update frequency
- flash storage devices exploit media parallelism -> multiple active segments can run simultaneously without frequent management operations
- performance degradation is insignificant

<u>Adaptive logging</u>

- At high storage utilization, F2FS use threaded logging
- writes new data to free space in dirty segment without cleaning it in the foreground

<u>fsync acceleration with roll-forward recovery</u>

- F2FS optimizes small synchronous writes to reduce the latency of fsync requests
- minimizes required metadata writes and recovers synchronized data with an efficient roll-forward mechanism

#### <u>Design and Implementation of F2FS</u>

#### On-Disk Layout

<u>Entire volume into 6 areas</u>

1. **Superblock** (SB): basic partition information, default params of F2FS; non-changeable
2. **Checkpoint** (CP): keeps file system status, bitmaps for valid NAT/SIT sets, orphan inode lists, summary entries of currently active segments. Stores two recovery points: last stable version, intermediate version.
3. **Segment Information Table** (SIT): per-segment information, number of valid blocks in the segment, bitmap for validity of all blocks in the Main area. For cleaning, used to select victim segments and identify valid blocks.
4. **Node Address Table** (NAT): block address table, locate all the node blocks in the Main area
5. **Segment Summary Area** (SSA): stores parent information of all blocks in the Main area. During cleaning, SSA is used to identify parent node blocks before migrating valid blocks.
6. **Main Area**: filled with 4KB blocks. Either node block or data block:
   - node block: inode or indices of data block
   - data block: directory or user file data

<u>Look-up operation for "/dir/file"</u>

1. use NAT to find and read root inode, then use root inode block to find its data block
2. use root inode's data block to find dir's inode number
3. use NAT to translate dir's inode number to a physical location
4. find dir's inode, use dir's inode to find dir's data block, which finds file's inode number
5. repeat translation and location for file
6. Actual file data is somewhere in Main area

#### File Structure

- NAT: stores the physical locations of all node blocks, through node ID
- node block:
  - inode: file's metadata
  - direct: block address of data
  - indirect: node IDs of other node blocks
- Wandering tree problem:
  - problem: in traditional LFS, when leaf data is updated, its direct and indirect pointer blocks need to be updated
  - solution: only update one direct node block and its NAT entry
- inline data and inline extended attributes in the inode block: reduces space requirements and improve I/O performance

#### Directory Structure

- 4KB directory entry (dentry) block: a bitmap and two arrays of slots and names in pairs
- bitmap tells whether each slot is valid or not
- a slot has a hash value, inode number, length of a file name, file type
- directory file: constructs multi-level hash tables to manage a large number of dentries efficiently
- When a file name is given in a directory, F2FS calculates its hash value, and traverses the multi-level hash tables in O(logn) complexity

#### Multi-head Logging

- Six major log areas for hot/warm/cold data
- Optimize cleaning:
  - Optimally, the number of valid blocks is either 0 or all

#### Cleaning

- Foreground and background cleaning

<u>Victim selection</u>

- Foreground: greedy (i.e. smallest # of valid blocks first) to minimize latency to application
- background: cost-benefit (i.e. utilization + age) to separate hot and cold data

<u>Valid block identification and migration</u>

Foreground

1. validity bitmap per segment in SIT
2. For each valid block, F2FS uses SSA to retrieve parent node blocks containing its index. 
3. if valid, migrate

Background

1. loads the block into page cache and marks them dirty
   1. lazy migration to alleviate performance impact on foreground I/O activities
   2. allow combining small writes

<u>Post-cleaning process</u>

- victim section becomes pre-free
- after a checkpoint, the section becomes free

#### Adaptive Logging

- normal logging: blocks are written to clean segments, strictly sequential writes. When storage becomes almost full, suffer cleaning overheads
- threaded logging: writes blocks to holes in existing dirty segments. No cleaning required, trigger random writes -> degrade performance
- F2FS switch between them based on the number of clean segments available

#### Checkpointing

- Checkpointing: provide a consistent recovery point from system crashes

<u>Checkpointing Procedure:</u>

1. All dirty node and dentry blocks in the page cache are flushed
2. suspends ordinary writing activities such as create, mkdir...
3. system metadata, NAT, SIT, SSA are written to disk
4. F2FS writes a checkpoint pack

<u>Checkpoint pack</u>

- 2 checkpoint packs in total
- Header and footer: start & end, versioning
- NAT and SIT bitmaps: validity of current NAT and SIT blocks
- NAT and SIT journals: contain a small number of recently modified entries of NAT and SIT to avoid frequent NAT and SIT updates
- Summary blocks of active segments: in-memory SSA blocks that will be flushed to the SSA area
- Orphan blocks: keep orphan inode info

#### Recovery

- roll back to the latest consistent checkpoint

<u>Strategy to fast fsync</u>

- write data blocks and their direct node blocks only, ignoring all other blocks

1. collects the direct node blocks with special flag N+n, construct a list of their node info
2. Use the node info in the list, loads the most recently written node blocks, named N-n, into the page cache
3. compares the data indices in between N-n and N+n
4. if different data indices, refreshes the cached node blocks with new indices from N+n, mark dirty
5. checkpoint

#### Questions

1. What problem does this paper address?
   - How to design a FS with a different storage system (i.e. SSD)? How to utilize its benefits and alleviate its drawbacks.
2. What is the author's main insight?
   - Hot/cold data logging
   - align segment/section/zone with SSD layout
   - optimize fsync performance
   - adaptive logging based on storage utilization
   - Handle wandering tree problem with NAT
   - Cleaning: background and foreground
3. What are the paper's key strengths?
   - Adjust to SSD's characteristics
4. What are the paper's key weaknesses?
   - Designed specifically for SSD?

## 113 - A Fast and Slippery Slope for File Systems

#### Overview

1. Use of virtual block device
   - Use RAM disk
   - manually delay I/O requests
   - variable latency is not emulated
2. Evaluate file systems across different device speed

#### Observations

1. Almost all file systems improve performance when underlying storage latency decreases
2. Performance flattens out under 1ms
3. EXT4 good across many environments
4. F2FS better at latencies below 1ms

## 114 - Granularity of Locks and Degrees of Consistency in a Shared Database

#### Background

- Transaction: a collection of one or more operations on one or more databases, which reflects a single real-world transaction
- ACID: Atomicity, Consistency, Isolation, Durability
- Consistency:
  - Transaction starts with the assumption that integrity constraints hold
  - After transaction, all integrity constraints still hold
- Threats to data integrity
  - Need for application rollback
  - system crashes
  - Concurrency
- Consistency levels: strict serializability, linearizability, sequential consistency, causal consistency, eventual consistency
- Lock modes:
  - exclusive/write (X)
  - shared/read (S)
  - X conflicts with X and S; S and S do not conflict
- Strict two-phase locking
  - locks are incrementally acquired in a specific order
  - all locks are released atomically when txn ends, as part of the commit or abort
- If each transaction does strict two-phase locking, then executions are serializable

#### Granularity

<u>Data item granularity</u>

- one tuple in a table
- a page, several tuples
- a table

<u>Granularity tradeoff</u>

- larger: fewer locks -> less overhead; less concurrency possible
- smaller/finer: more locks -> more overhead; more concurrency possible
- System usually gets fine grain locks until there are too many of them, then they are replaced with larger granulariy locks

<u>Multigranular locking</u>

- A hierarchy of locks, needs to manage conflicts among items of different granularity
- System gets "intention" mode locks on larger granules before getting actual S/X locks on smaller granules

#### Hierarchical Locking

- A hierarchy of locks, each child is at a finer granularity of its parent
- Intention mode: used to tag (lock) ancestor of a locked node

<u>Modes</u>

- Exclusive (**X**): gives exclusive access to the requested node and all its descendants
- Share and Intention Exclusive (**SIX**): gives shared access to the requested node and all its descendants. gives intentional exclusive access to the requested node. Requester is able to acquire X, SIX, or IX lock on descendants.
- Share (**S**): gives shared access to the requested node and all its descendants.
- Intention Exclusive (**IX**): gives intention exclusive access to the requested node. The requestor can acquire X, SIX, S, IX, IS lock on descendants.
- Intention Shared (**IS**): gives intention shared access to the requested node. The requester can acquire S, IS lock on descendants.

<u>Compatibility</u>

|      | X    | SIX  | S    | IX   | IS   |
| ---- | ---- | ---- | ---- | ---- | ---- |
| X    | -    |      |      |      |      |
| SIX  | -    | -    |      |      |      |
| S    | -    | -    | Y    |      |      |
| IX   | -    | -    | -    | Y    |      |
| IS   | -    | Y    | Y    | Y    | Y    |

<u>Policies</u>

1. Before requesting S or IS on a node, all ancestors of it needs to be in IX or IS lock
2. Before requesting X, SIX, IX on a node, all ancestors needs to be in SIX or IX lock
3. Locks are either: a) released atomically; b) from lower to higher level

#### Questions - Granularity of locks

1. What problem does this paper address?
   - How to manage locks under the tradeoff of concurrency and overhead?
2. What is the author's main insight?
   - Hierarchical locking
   - Intentional modes
   - extensible to DAG
3. What are the paper's key strengths?
   - Good balance on the tradeoff for small database
4. What are the paper's key weaknesses?
   - For large database, still too much overhead

#### Isolation Levels

1. Read uncommitted (lv 1): can read uncommitted writes
   - No S-locks at all
2. Read committed (lv 2): only read committed writes, but distinct reads within the same transaction on the same data item could be different
   - S-locks acquired during read, released immediately
3. Repeatable read (lv 2.99): only read committed writes, guarantees serializability on individual data items, but phantom reads could happen (e.g. read different # of entries in a table)
   - S-locks acquired til transaction ends
4. Serializable (lv 3): completely isolated

#### Snapshot Isolation (SI)

<u>Snapshot Isolation</u>

- Every transaction sees a "snapshot" of the database, at an earlier time
- Read may not give current value
- Use old versions to find value

<u>First Committer wins (FCW)</u>

- A transaction T is not allowed to commit if any other transaction has modified the same item after T starts, and committed earlier than T
- T must hold write locks on modified items at time of commit

<u>SI benefits</u>

- reads never blocked
- reads don't block writes
- avoid common anomalies:
  - no dirty data
  - no lost update
  - no inconsistent read
  - no phantoms

#### Questions

1. What problem does this paper address?
   - A clear definition of isolation levels
2. What is the author's main insight?
   - 4 levels
3. What are the paper's key strengths?
4. What are the paper's key weaknesses?

## 115 - Generalized Isolation Level Definitions

#### Overview

- in isolation levels, there is a tradeoff between consistency vs. performance

#### Generalized Isolation Levels

- PL-1 (Read uncommitted): T's writes are completely isolated from the writes of other transactions
- PL-2 (Read committed): T has only read updates of transactions that have committed by the time T commits
- PL-2.99  (Repeatable read): T is completely isolated from other transactions with respect to data items; has PL-2 guarantees for range reads
- PL-3 (Serializable): T is completely isolated from other transactions

#### Questions

1. What problem does this paper address?
   - Make isolation levels generalized and applicable for other isolation protocols
2. What is the author's main insight?
   - Identify behaviors of each level, apply to generalized protocols
3. What are the paper's key strengths?
   - not only pessimistic, but also optimistic and multi-version concurrency control
4. What are the paper's key weaknesses?

## 116 - Mesa

#### Inter-Process Communication (IPC) Methods

- IPC is used to allow communication and synchronization between processes

- Two methods: shared memory and message passing

#### Shared Memory

- Allows two or more processes to have access to the same memory region, can exchange data without copying it
- usually implemented via memory mapped files; shared memory is mapped to address space of each process that requires access
- Preferred when processes need to exchange large amounts of data

#### Message Passing

- Two processes establish a communication link, exchange messages using send/receive
- Fixed or variable size: fixed easy for OS designer, variable easy for programmer
- a standard message has a header and a body

<u>Direct vs. Indirect</u>

- Direct communication link: receiver identity is known and message is sent directly to the receiving process. There is at most one link between two processes. 
  - Drawback: limited modularity: changing the identity of a process requires notifying all sender and receiver having a link
- Indirect communication link: via a shared mailbox (or port), which consists of a queue of messages. The sender keeps the message in mailbox and the receiver picks them up.
  - advantage: the mailbox can be later bound to another process; allow multiple senders to one mailbox
  - drawback: sender doesn't know which process will actually receive its message

<u>Blocking vs. non-blocking</u>

- decide if send/receive ops are blocking (synchronized) or not

<u>Buffering</u>

- Size of receiver's queue
  - no queue: sender has to wait until receiver is ready
  - bounded queue: sender needs to wait if queue is full
  - unbounded: sender never needs to wait. Designer needs to be careful with physical resource limits

| Shared Memory                                                | Message Passing                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Shared memory region                                         | Message passing facility                                     |
| Used for communication between processes on a multiprocessor system where communicating processes reside on the same machine (needs to share a common address space) | Used in a distributed environment  where communicating processes on remote machines, connected via network |
| Application programmer explicitly writes code for write/read ops to/from shared memory | Message passing facility programmer does low-level coding    |
| Maximum speed of computation, no syscall during communication | Time consuming, needs kernel intervention (syscalls)         |
| Processes need to avoid racing conditions                    | Useful for sharing small amounts of data, no racing conditions |
| Faster                                                       | Slower                                                       |

#### Monitor

- monitor unifies:
  - the synchronization
  - the shared data
  - the body of code which performs the accesses
- data is protected by monitor, can only be accessed within the body of a monitor procedure
- monitor ensures that at most one process is executing a monitor procedure at a time

## 117 - Consistency without Concurrency Control

#### Overview

- Scaling shared mutable data is a difficult problem
- Two approaches:
  - Guarantee consistency
    - serializing all updates, hard to scale
  - Guarantee scalability
    - gives up consistency guarantees
    - e.g. use last-writer-wins (LWW)
  - Optimistic replication:
    - replicas diverge
    - resolve with LWW-like methods or serialization

<u>Commutative Replicated Data Type (CRDT)</u>

- ensures Strong Eventual Consistency (SEC): eventual consistency (i.e. may have conflict) with the guarantee that all replicas that have received the same updates (maybe different orders) will eventually reach the correct state
- replicas converge if:
  - concurrent updates are **commutative**
  - all replicas execute all updates in **causal order**
- No need for consensus-based concurrency control because there is no conflict
- Ensures consistency at large scale at a low cost

- e.g. a set with a single add-element operation

#### Ordered-set CRDT

- Abstraction of an ordered sequence of atoms

<u>Model</u>

- A collection of sites (nodes), each carrying a replica of a shared ordered-set object, connected by a reliable broadcast protocol
- Support a peer-to-peer, multi-master execution model
  - Some replica initiates an update and executes it against its local replica
  - each other site eventually receives the operation and replays it locally
  - causally-related operations execute in order, but concurrent operations may execute in different orders in different replicas
- operations
  - *insert(ID, newatom)*: adds newatom with ID to the ordered-set
  - delete(ID): deletes the atom with ID from the ordered-set
- assumptions:
  - two inserts or deletes on different IDs commute
  - operations are idempotent
  - To ensure commutativity across different sites, we ensure that no two IDs are equal across sites
- ID properties:
  - Two replicas of the same atom have  the same ID
  - no two atoms have the same ID
  - ID remains constant
  - There is a total order for IDs
  - ID space is dense (i.e. can allocate atom between any two IDs)

#### Treedoc CRDT

<u>Treedoc Structure</u>

- A tree
- An atom identifier is TID, a path in a tree
- if the tree is balanced, average TID size is O(log n)
- mini-node: internal nodes within a node, to solve concurrent TID allocations
  - disambiguator: identifies which site inserted each mini-node; unique, ordered, gives total order between entries
- major node: contains mini-nodes

<u>Treedoc insert and delete</u>

- delete: discards the atom with TID, mark the tree node as a *tombstone*

- insert: chooses a fresh TID, insert it in a tree as you normally do (append 0 or 1 for left or right child)

<u>Treedoc Restructuring</u>

- After a period of time, the tree is either badly balanced or filled with tombstone
- flatten: transforms the tree into a flat array, eliminate all storage overhead
- flattening operation changes TIDs, so allow non-ambiguous renaming
- flattening is not commutative with update ops:
  - update-win approach: when update happens concurrently with a flatten, update wins and flatten aborts
  - use two-phase commit for coordination
- problematic with large-scale and dynamic systems

- flatten results:
  - decrease file size
  - decrease latency

<u>Nebula</u>

- core: small group of well-known and well-connected sites, only group that participates in flattening; membership protocol needed to join/leave
- nebula: not in the core, does not participate in flattening
- epoch: interval between flattening
  - sites may exchange updates only if in the same epoch
  - core sites have the same epoch
  - nebula may be behind
- Nebula catch-up protocol
  - If core is in epoch n, nebula in epoch n-1
  - core sends all n-1 epoch updates to nebula, which then catch up to n by flattening its local tree
  - nebula transforms its old message to epoch n format, then send to core

<u>Approximate causal ordering</u>

- vector clocks are used to ensure causal ordering and to suppress duplicate messages
- causal ordering is already in Treedoc:
  - insert a node before inserting its descendant
  - insert a node before deleting it
  - unrelated nodes can be updated concurrently

#### Questions

1. What problem does this paper address?
   - How to achieve both consistency and scalability with a special yet generalizable data type?
2. What is the author's main insight?
   - Commutative Replicated Data Type
   - Treedoc
3. What are the paper's key strengths?
   - Achieve both consistency and scalability with CRDT
   - Demonstrates that practical CRDT exists
4. What are the paper's key weaknesses?
   - Coordination is still required at flattening

## 118 - Coordination Avoidance in Database Systems

#### Overview

- Minimizing coordination is the key to high-performance, scalable, high-availability database design
- Coordination: multiple concurrent processes need to synchronize state/operations, o/w stall, in order to achieve correctness

<u>Invariant Confluence</u>

- if an application's operations are I-confluent, a database can correctly execute them without coordination
- if operations are not I-confluent, then coordination is required to ensure correctness
- I-confluence exposes a trade-off between the operations a user wishes to perform and the properties she wishes to guarantee
- Try to define operations that ensure I-confluence

<u>Cost of Coordination</u>

- High latency
- Low throughput
- partial failure -> unavailability

#### Questions

1. What problem does this paper address?
   - When is coordination strictly necessary to maintain application-level consistency?
2. What is the author's main insight?
   - Identify invariants in the database, and conduct I-confluence analysis on invariants to see if certain coordination is necessary.
   - I-confluence presents a trade-off between operations a user wishes to perform and the properties she wish to guarantee
3. What are the paper's key strengths?
   - Identify scenarios where coordination is unnecessary given certain database operations
4. What are the paper's key weaknesses?
   - It is non-trivial to identify invariants, which need to be comprehensive

## 119 - Principles of Transaction-Oriented Database Recovery

#### Failures

<u>Transaction Failure</u>

- e.g.
  - Code aborts due to input/database inconsistency
  - Mechanical aborts due to concurrency control solutions
- Frequent events, need instant recovery

<u>System Failure (Fail stop)</u>

- DBMS bug, OS fault, HW failure: volatile memory dies, durable memory survives
- Infrequent events, minute to recover

<u>Media Failure (fail-stop)</u>

- IO code bugs, disk HW failures: lose disk info
- Rare events, hours to recover from checkpoint & log

#### Recoverable System Model

- Log is written directly to non-volatile storage
- Undo: rollback aborted transaction
  - transaction failure or system failure
- Redo: repeat transaction on old DB data
  - system failure or media failure

#### Views

- Current DB: on disk + memory buffers
  - Some changes in memory, lost during failures
- Materialized DB: the state of DB right after crash restart, and before applying log
  - Some completed changes may not be visible because the buffer was lost
- Physical DB: on disk
  - All blocks including out-of-date blocks & possibly incomplete data structures

<u>Connecting views</u>

- Changing non-volatile memory
  - Modifications of current DB may cause writes to physical DB, which is not part of materialized DB yet
  - After pointer structs are updated (i.e. propagated), then those modifications are part of materialized DB now
  - Some DBs overwrite prior copies, so write is equivalent to propagate. But changes to materialized DB becomes non-atomic

#### Temporary Log Files

- Log contains redundant information to cope with failure
- On-disk temporary WAL contains all info needed to transform materialized DB to current DB
- Memory pressure can push uncommitted dirty data to database
  - in non-overwrite DB, such writes are forgotten when memory is lost
  - in overwrite DB, this requires UNDO log records (STEAL) written before propagation
  - Commit logically forces propagation (FORCE), but efficiency concerns cause DBs to avoid synchronous IO, instead writing REDO log records before transaction commit
- Log types:
  - Physical logging: capture data values directly
  - Logical logging: capture operations that give values
  - State logging: capture full values
  - Transition logging: capture value difference
  - Page logging: capture full page values
  - Record logging: capture only the records changing
  - Tradeoff: simplicity/speed of recovery vs. density of log

#### Checkpoint

- Checkpoint limits the amount of REDO processing required
- REDO starts at beginning of log, can be really slow
- FORCE propagation means no REDO (just UNDOs)
- Transaction consistent:
  - Quiesce all transactions, propagate all dirty data, write log entry
  - allows partial REDO to start here
  - allows global UNDO stops here (o/w UNDO needs to find BEGIN for oldest incomplete transaction)
- Action consistent:
  - Quiesce all transaction-caused actions, propagate all dirty data, write log entry
  - allow partial REDO recovery processing to start here
- Fuzzy checkpoint:
  - propagate only to log, to reduce REDO processing on restart

#### Questions

1. What problem does this paper address?
   - A summary of database recovery techniques and their pros and cons
2. What is the author's main insight?
   - A summary of database recovery techniques and their pros and cons
3. What are the paper's key strengths?
   - Comprehensive
4. What are the paper's key weaknesses?
   - A survey, so no?

## 120 - Time, Clocks, and the Ordering of Events in a Distributed System - Leslie Lamport

- satisfies *clock consistency condition*: if event A's clock comes before event B's clock, then A happens *either before or at the same time* as B.
- vector clock satisfies *strong clock consistency condition*: if event A's clock comes before B's clock, then A happens *strictly before* B.
- A distributed system consists of a collection of distinct processes which are spatially separated, and communicate by exchanging messages.
- A system is distributed if the message transmission delay is not negligible compared to the time between events in a single process.
- In a distributed system, the relation "happen before" is only a partial ordering of the events in the system. It is impossible to say that one of two events occurred first.
- **A single process**: a set of events with an a priori total ordering
- **A distributed system**: a collection of processes
- **Happened before "->"**: 1) same process, a before b; 2) a send, b receive; 3) a->b, b->c, a->c.
- **Logical clocks**: define a clock $C_i$ for each process $P_i$ which assigns a number $C_i<a>$ to any event a in that process. The entire system of clocks is represented by the function C which assigns to any event b the number $C<b>$, where  $C<b>=C_j<b>$ if b is an event in process $P_j$. No assumption about the relationship between C and physical time (e.g. could be counters).
- **Clock condition**: For any events a, b: if a->b, then $C<a>$ < $C<b>$
- **Implementation Rule**:
  1. Each process $P_i$ increments $C_i$ between any two successive events
  2. a) If event a is the sending of a message m by process $P_i$, then the message m contains a timestamp $T_m=C_i<a>$. b) Upon receiving a message m, process $P_j$ sets $C_j$ greater than or equal to its present value and greater than $T_m$
- **Total Order**: Order the events by the times at which they occur. To break ties, we use any arbitrary total ordering < of the processes.
- **Distributed algorithm**: each process independently follows these rules, and there is no central process.

#### Questions

1. What problem does this paper address?
   - A clock protocol to demonstrate partial ordering in a distributed computing environment
2. What is the author's main insight?
   - Separate logical clock from physical clock
   - Each machine has its own logical clock, which is adjusted when messages are received
   - Ensures causal relationships by updating receiver's logical clock
3. What are the paper's key strengths?
   - A solution to achieve partial ordering in distributed systems
4. What are the paper's key weaknesses?
   - Does not exhibit strong clock consistency condition: if A's clock comes before B, Lamport's clock can only ensure that A comes before or at the same time as B (instead of strictly before B).

## 121 - Efficient Optimistic Concurrency Control Using Loosely Synchronous Clocks

#### Overview

- Distributed object-oriented database system: 
  - persistent storage and transaction support at server; 
  - applications (and caching) at clients
- This scheme provides serializability & external consistency
  - Serializability: commit transactions can be totally ordered, called serialization order. The actual effect of running the transactions is the same as running them one at a time in that order.
  - External consistency: the serialization order such that if S commits before T in real time, then S is ordered before T
- Low time and space overhead

#### Protocol Overview

- Application runs on client machines, caches happen on clients
  - methods execute on client machines, using locally cached objects
  - objects are fetched from servers when needed, then cached on clients
- Server stores persistent storage
  - Server tracks the object caches at clients
  - Server has cache of objects in their main memory, used to satisfy fetch requests
  - Each object has an owner server, where the object resides
- Front end: the code that manages the cache on client
  - Keep track of objects read and written by a transaction T at its client

<u>Protocol Overview</u>

1. When the client requests a commit, front end gathers info on T:

   - validation information - what objects are used, and how (read/write)
   - installation information - modified new objects

   - *collected only for mutable objects (concurrency control is not required for read-only objects)

2. front end sends these two info to a server that owns some of the used objects

3. The server becomes a coordinator for a 2-phase commit protocol

   1. (Observed delay by clients) Coordinator sends both info to participants. Participants validate. If succeeds, log the installation changes, send ok back. If fails, send reject back. If all ok, the coordinator logs the commit, sends commit to all participants.
   2. Coordinator sends commit to all participants. Each participant then installs new versions of its objects, log a commit, send ack back to the coordinator.

   - *Read-only transactions don't need phase 2

4. After a read-write transaction that has installations, an invalidation message is sent to all other clients. The front end then evicts the outdated value. If the outdated value is already read, abort the transaction immediately.

5. Clients send ack back to server after installation. Then the server removes the info about the invalidated objects.

#### Validation Scheme

- Want serializability and external consistency
- Backward validation:
  - A validating transaction T is checked against all transactions that have already validated successfully
- For validation, only require cached sets, invalid sets, validation queue on each server, small enough to put in mem

<u>Global Serialization</u>

- Use timestamp from real clocks
- Loosely synchronized clock:
  - clocks at different nodes in the network may differ by at most a small skew (tens of milliseconds)
- Each transaction is assigned a timestamp = {local time, server ID}
- The coordinator sends these info for validation:
  - timestamp
  - objects read
  - objects written
  - client id
- Transaction S and T conflicts when one has modified an object that the other read or modified
- All successfully validated transactions are recorded in a validation queue,
  - when a new validating transaction comes, the participant checks to detect conflicts and ensure serializability and external consistency
  - if the new transaction fails the check (even against a prepared transaction), the participant always aborts the validating transaction

<u>Check against later transaction</u>

- Due to difference in clock, some later transaction T in real time may receive a timestamp earlier than S.
- If T read any object that S modified, T must be aborted.
- O/w, this violates external consistency
- Later-conflict check: later transaction T with an earlier timestamp than S
  - T must not read S's modifications
  - T must not modify any object that S read
- This kind of conflict is rare, only happens in a small clock skew window

<u>Check against earlier transaction</u>

- For each validated transaction S with earlier timestamp than T,
  1. If S reads object x and T modifies x: no check
  2. If S modifies object x and T reads x: 
     1. if S is not committed, check fails
     2. if S is committed, check if T is read the most current version of x
- version check using an invalid set of objects for each client

<u>Truncation</u>

- Threshold timestamp:
  - guaranteed to be greater than or equal to the timestamps of all transactions whose information has been removed form the VQ
- Validation record is retained for
  - all uncommitted read-write transactions
  - all transactions with timestamps above the threshold
- A transaction T timestamped below the threshold fails validation because info necessary for the later-conflict check has been discarded
- Threshold Interval
  - local time + (msg_delay + clock skew)

#### Crash Recovery

- Log validation information (VQ and threshold) on stable storage
  - for read-write transactions, the validation record can be logged along with the installation information, so little latency
  - for read-only transaction, since no information is recorded initially, we maintain a stable threshold
    - which is always later than the timestamp of any transaction
    - only used to restore threshold on restart

#### Adaptive Callback Locking

- page-based scheme that does locking and callbacks at the page level
  - object level locking and callback for pages that exhibit read-write sharing

#### Questions

1. What problem does this paper address?
   - How to do optimistic concurrency control efficiently?
2. What is the author's main insight?
   - Clients use cached objects for transaction read, use a front end to track which objects are read/written
   - After receiving a commit request, the server conducts a 2 phase commit to validate the transaction
   - Use an efficient validation algorithm that is both time and space efficient
   - Do not incur extra logging latency for read-only operations, add a stable threshold for crash recovery
3. What are the paper's key strengths?
   - An optimistic concurrency control protocol that is efficient for low and moderate contention levels
   - Low number of message sent compared to pessimistic protocols
4. What are the paper's key weaknesses?
   - Hard to scale for high contention levels and higher number of clients

## 122 - Paxos

#### Paxos Protocol

<u>Phase 0: Leader Selection</u>

1. When a potential leader detects no leader is present, it chooses a unique ballot ID, which is higher than any it has seen so far, and send to all processes
2. When a process receives a ballot ID, it elects the highest it has seen so far
3. If a candidate receives a majority of OK, it becomes the new leader

- *It is possible that more than 1 leader exists, but Paxos still ensures safety

<u>Phase 1: Prepare</u>

1. The leader sends a new proposal number n, sends the request to all processes
2. Acceptor responds with:
   - a promise of not accepting a proposal numbered less than n
   - the proposal with the highest number less than n that is accepted, if any

<u>Phase 2: Accept</u>

1. After receiving a majority of responds, the leader sends a proposal of <n, v>, where v is either the value of the highest number accepted proposal, or a new value chosen by the leader

#### Some facts

<u>Consensus achieved when?</u>

- Achieved when a majority of acceptors hear the proposed value and accept it, before responding any message

#### Questions

1. What problem does this paper address?
   - How to achieve provable async, failure-stop consensus
2. What is the author's main insight?
   - Ask acceptors to promise ignoring lower proposal numbers
3. What are the paper's key strengths?
   - first provable async, failure-stop consensus
4. What are the paper's key weaknesses?
   - complicated to learn or implement

## 123 - Raft

#### Overview

- Paxos does its job, but it is hard to implement or learn
- Raft provides better understandability, easier for both implementation and education

#### Replicated State Machine

- Typically implemented using a replicated log
- Each server stores a log containing the same commands in the same order
- Keeping the log consistent is the job of a consensus algorithm
- Consensus algo ensures every log eventually contains the same log, even if some servers fail

<u>Consensus Algorithm Requirements</u>

1. Ensures safety (never returns an incorrect result) under any non-Byzantine environment
2. Ensures availability as long as a majority of machines are operational and can communicate with each other and with clients
3. Timing does not affect consistency: faulty clocks and extreme message delays can at most cause availability issues
4. Proceed when a majority of servers have responded, minor slow servers should not affect performance

#### Raft Protocol

<u>Phase 1: Leader Election</u>

1. Current term timeout, a new term starts.
2. A follower increments its current term and transitions to candidate state
3. A candidate sends out requests to vote. Followers vote for at most one leader, first-come-first-serve basis.
4. After receiving a majority of votes, becomes the leader
5. If receive msgs from another leader with at least as large term number, recognize the leader

<u>Phase 2: Log Replication</u>

- Client sends requests to only the leader
- The leader first append the log (without applying it yet), then send appendEntry requests to all followers
- After receiving majority of acks, apply the log and send applyEntry requests to all followers
- A command is logged with its term number and a unique incremental index 
- For each AppendEntry request, the leader piggyback previous log command's term and index to ensure log consistency (induction-like mechanism)

<u>Log Inconsistency b/w Leader and Follower</u>

1. Leader will remove all inconsistent log entries in the followers
2. Add leader's log entries to all followers

- Ensures that the new leader has the most up-to-date log entires by election process:
  - A candidate needs to provide its most current log in its requestToVote msg, which will be denied by a follower if the follower's log is more up-to-date

#### Questions

1. What problem does this paper address?
   - How to improve understandability of consensus algorithm in an async, non-Byzantine environment?
2. What is the author's main insight?
   - Separate consensus into smaller subproblems: leader election, log replication, safety, configuration changes
3. What are the paper's key strengths?
   - Arguably better understandability and implementation
4. What are the paper's key weaknesses?
   - does the same job as paxos

## 124 - Chubby Lock

#### Overview

- Chubby is a lock service used in a loosely-coupled distributed system consisting of moderately large numbers of small machines connected by a high-speed network
- Primary goals are availability and reliability for a moderately large set of clients
- Secondary goals are throughput and storage capacity
- Chubby's interface is similar to a simple file system that performs whole-file reads and writes, with advisory locks, with notification of events

#### Design Decisions

1. Choose a lock service (instead of a library or service for consensus)
2. Serve small-files to permit elected primaries to advertise themselves and their parameters, rather than build and maintain another service
3. Clients and replicas of a replicated service may want to know hen the service's primary changes, so some notification mechanism is needed
4. consistent client-side caching to reduce server load
5. file system interface

<u>Coarse-grained lock service</u>

- Creating a lock service rather than a client library made it easier to integrate with existing applications
- Chubby runs five replicas in a cell; client systems that depend on Chubby can work with fewer replicas (not possible with a client library)
- Chubby is explicitly designed for coarse-grained locking use cases (e.g. electing a primary)
  - coarse-grained locks impose far less load on the lock server
  - allow many clients to be served by a modest number of lock servers with somewhat lower availability

#### System Structure

- Two main components:
  - server
  - client: a library that client apps link against
- All communications are done through the client library

- consists of a small set of servers (e.g. 5), replicas
- A master, elected using Paxos
- Master lease: replica's promise to not elect other masters within a short time period (e.g. 5s)
- Replicas maintain a copy of a simple database
- Only master initiates reads and writes to the database, other replicas copy updates from the master, through Paxos
- Client finds the master using DNS
- All communicates are from clients to the master, master then propagates writes to other replicas through paxos; reads are served by the master as master lease is not expired
- if master fails, other replicas run leader election when master lease expires
- server replacement is done if a server does not respond in hours, updates DNS, master's cell's member table

<u>Files, directories, and handles</u>

- Chubby exports a file system interface, similar to UNIX
- The name space contains files and directories, called nodes
- Any node can act as an advisory reader/writer lock
- Each nodes has meta-data, including ACLs included to control reading, writing, and changing ACL permissions
- Clients open nodes to obtain handles, which is similar to UNIX file descriptors

<u>Caching</u>

- Chubby clients cache file data and node meta-data to reduce read traffic
- cache is consistent and write-through
- Master keeps a list of what clients may be caching
- Cache is either consistent or an error

<u>Events</u>

- When a client creates a handle, clients may subscribe to a range of events on a file
- events are delivered asynchronously

#### Questions

1. What problem does this paper address?
   - A coarse-grained locking service (e.g primary election) that is based on distributed consensus algorithm
2. What is the author's main insight?
   - Emphasize on availability and reliability
   - use distributed consensus protocol to achieve fault tolerance
   - consistent client side caching
   - notification of updates
   - familiar file system interface
   - small storage for meta-data
3. What are the paper's key strengths?
   - High availability
   - Used to elect primary
   - A good name service
4. What are the paper's key weaknesses?



## 125 - Paxos Made Live

#### Questions

1. What problem does this paper address?
   - The gap between Paxos in literature and in implementation
2. What is the author's main insight?
   - Handle membership Management, disk corruption, master lease, snapshot
3. What are the paper's key strengths?
4. What are the paper's key weaknesses?



## 126/127 - Practical Byzantine Fault Tolerance

#### Byzantine Fault Tolerance

<u>Assumptions:</u>

- no assumptions about faulty behavior
- asynchronous (cannot assume synchrony): no bounds on delays
  - solution: 
    - ensures safety without synchrony: guarantee no bad replies
    - eventual liveness: will reply when DOS attack ends

<u>Faulty Clients</u>

- all operations by faulty clients are observed in a consistent way by non-faulty clients
- enforce access control

- Faulty clients cannot break invariants

#### PBFT Algorithm

<u>Algorithm Properties</u>

- Arbitrary replicated service
  - can have complex operations
  - mutable shared state
- Safety and liveness:
  - system behaves as correct centralized service
  - clients eventually receive replies to requests
- Assumptions:
  - 3f+1 replicas with f potentially Byzantine faults
  - strong cryptography
  - for liveness: eventual time bounds

<u>Algorithm Setup</u>

- State machine replication:
  - deterministic replicas, start in the same state
  - replicas execute the same requests in the same order
  - correct replicas produce the same result
- View
  - succession of configurations
  - Each view has a designated primary
- Primary:
  - responsible for receiving requests from clients
  - primary is determined by view number
  - picks ordering
- Backup:
  - responsible for responding messages from primary
  - ensures primary behaves correctly
  - certify correct ordering
  - trigger view changes to replace faulty primary
- Replicas remember decisions in log
- Messages are authenticated

<u>Algorithm Overview</u>

1. A client sends a request to the primary
2. primary multicasts the request to the backups
3. Replicas execute the request and send a reply to the client
4. The client waits for f+1 replies from different replicas with the same result, which is the actual result

<u>Quorums and Certificates</u>

- A quorum has at least 2f+1 replicas so that any two quorums will have at least one intersect
- certificate: set with messages from a quorum
- algorithm steps are justified by certificates

#### Normal Operation

<u>Phase 1: Pre-prepare</u>

1. After receiving a request from a client, the primary assigns a sequence number n to request m in view v
2. Primary multicasts pre-prepare message to all other replicas

<u>Phase 2: Prepare</u>

1. After receiving pre-prepare message from the primary, a backup sends a multicast prepare message to all other replicas
2. In total, each replica receives 1 pre-prepare msg from the primary and 2f prepare message from replicas, so 2f+1 in total -> a quorum so a certificates
3. *Note: no two different messages have the same sequence and view number, because it requires a quorum

<u>Phase 3: Commit</u>

1. After receiving a quorum in prepare stage, both primary and backups multicast commit message to all other replicas
2. Each replica will have 2f+1 commit msgs -> a quorum so a certificate
3. Send result back to the client
4. *Note: request n is executed when having certificate, and requests less than n have been executed

#### View Change

1. When primary fails, timeout is triggered at every replica
2. Each replica will send a view-change message to all other replicas
3. When the new primary receives 2f+1 view-change msgs (a quorum), it sends a new-view message with signatures of all 2f+1 view-change msgs
4. The new primary will start receiving client requests

<u>Liveness</u>

- When the primary fails, timeout triggers view changes

#### Garbage Collection

<u>Checkpoint</u>

- A checkpoint is the state immediately after every (e.g. 100) requests
- A stable checkpoint is a checkpoint with proof
  - proof is achieved when a replica receives 2f+1 checkpoint message from different replicas, each with a signature
  - when a new stable checkpoint is achieved, all messages of previous sequence numbers can be deleted

#### Questions

1. What problem does this paper address?
   - A consensus protocol in an async, Byzantine environment
2. What is the author's main insight?
   - 3 phase commit
   - 3f+1 replicas: 1 primary, 3f backups
   - Safety no matter what
   - Liveness achieved if there is a synchrony bound
3. What are the paper's key strengths?
   - Practical BFT protocol in an async environment
4. What are the paper's key weaknesses?
   - not scalable, more replicas mean worse performance

## 128 - Disco

#### Why Virtualize?

- Hardware development is faster than OS development frequency
- Consolidate machines
- Isolate performance, security, and configuration
- Stay flexible
- Cloud computing

#### Virtualization Approaches

- Disco/VMware/IBM: complete virtualization - runs unmodified OSs and applications
  - Use software emulation to shadow system data structures
  - Need hardware support
- Paravirtualization - change interface to improve VMM performance/simplicity
  - Must change OS and some apps
  - Xen: change OS but not applications - support full Application Binary Interface (ABI)
    - Faster than a full VM

#### Challenges of VMM

<u>Overheads</u>

- privileged executions need to be emulated by the monitor
- Device I/O needs to me intercepted and remapped by the monitor
- Additional memory costs to run multiple OSes, including code and data, file system buffer cache

<u>Resource Management</u>

- Monitor does not have OS-level information on resource management

<u>Communication and Sharing</u>

- Sharing and communication becomes difficult, such as disk, file, etc

#### DISCO Overview

- Disco runs multiple independent VMs on the same hardware by virtualizing all resources of the machine
- Each VM can run a standard OS that manages its virtualized resources independently of the rest of the system

<u>Processors</u>

- The virtual CPU of Disco provide the abstraction of a processor
- Disco emulate all instructions, the memory management unit, and the trap architecture of the processor
- Allow unmodified applications and existing OSes to run on the VM
- Disco support efficient access to some processor functions
  - e.g. kernel operations such as enabling and disabling CPU interrupts can be performed using load and store instructions on special addresses, reducing overheads by trap emulation

<u>Physical Memory</u>

- Disco provides an abstraction of main memory with contiguous physical address space starting at address zero
- Disco uses dynamic page migration and replication to export a nearly uniform memory access time memory architecture to the software -> allows non-NUMA-aware OS to run

<u>I/O Device</u>

- Each VM is created with a specified set of I/O devices
  - e.g. disk, network interface, periodic interrupt timer, clock, console
- Disco virtualize each I/O device
- Disco intercept all communications to and from I/O devices to translate or emulate the operation
- Virtual disks can be configured to support different sharing and persistency models
- Monitor virtualizes access to the networking device, each VM is assigned a different link-level address. Disco acts as a gateway that send and receive packets using the machine's network interface.

#### Disco Implementation

- Multi-threaded shared memory program
- Extra attention given to NUMA memory placement, cache-aware data structures, and interprocess communication pattern
- Disco code copied to each flash processor
- Communicate using shared memory

<u>Virtual CPUs</u>

- Disco emulates the execution of the virtual CPU by using direct execution on the real CPU
- Pros: most operations run at the same speed as they would on the raw hardware
- Challenge: how to detect and fast emulate privileged instructions (not safely exported) by the OS: TLB modification, direct access to the physical memory, I/O device
- For each vCPU, Disco keeps a data structure that contains all state of the vCPU when it is not scheduled on a real CPU: saved registers, privileged registers, TLB contents
- Disco runs in kernel mode, OS runs in supervisor mode, o/w user mode
- Disco emulates the operations that OS cannot run in supervisor mode:
  - when a trap such as page fault, system call, or bus error occurs, the processor traps to the monitor, which emulates the effect of the trap by updating the privileged registers of the virtual processor and jumping to the virtual machine's trap vector

<u>Virtual Physical Memory</u>

- Disco adds a level of address translation, maintains physical-to-machine address mapping
- VMs use physical addresses that start at address zero, contiguous to the size of VM's memory
- Disco maps these physical addresses to machine addresses
- Disco performs this translation using software-reloaded TLB, where when an OS attempts to insert a virtual-to-physical TLB entry, Disco uses physical-to-machine mapping to translate it into virtual-to-machine TLB entry.
- For each TLB entry later on, there is no extra overhead
- To avoid flushing TLB on MMU context switches within VM, each entry in TLB tagged with address space identifier
- However, must flush the real TLB on VM switch
-  Somewhat slower:
  - More TLB misses: TLB used for all VM OSs, TLB must be flushed on VM switch
  - Each TLB miss is more expensive be cause of trap emulation, privileged execution emulation, and remapping of physical-to-machine
- Optimization:
  - add a second-level TLB to store more virtual-to-machine translations

<u>NUMA Memory Management</u>

- Non-uniform Memory Access: each processor has access its local memory (fast, e.g. cache) and other processor's memory (slow) or shared memory (slow). So the access time to different memory are non-uniform
- Manage allocation of real memory to VMs
- Disco needs to allocate memory and schedule vCPUs such that cache misses can be satisfied by local memory, instead of suffering additional latency of a remote cache miss
- Disco targets cache coherence done by hardware, so NUMA memory management is only an optimization that enhances data locality

- **Dynamic page migration and page replication**:
  - Pages heavily accessed by only one node is migrated to that node
  - Read-shared are replicated to nodes heavily accessing them
  - Write-shared are not moved, b/c remote access is still required anyway
  - limit the number of times a page can be moved (limit overhead)
- To migrate, first invalidate all TLB entries, then copy to local memory
- To replicate, first downgrade all TLB entries to read-only, then copy and update TLB entries

![image-20210712121248502](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210712121248502.png)

<u>Virtual I/O Devices</u>

- Main task: translate all I/O instructions from using physical memory addresses to machine memory addresses
- Emulated all programmed I/O instructions

- copy-on-write disk blocks - share OS pages:
  - Track which blocks are already in memory
  - If a block is already in memory, reuse it by marking all versions read-only, and use copy-on-write if they are modified

![image-20210712134035438](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210712134035438.png)

<u>Virtual Network Interface</u>

- Within a physical machine, zero-copy networking allows fake subnet
- sender and receiver can use the same buffer
- the monitor's networking device remaps data page from source's machine address to destination's

![image-20210712134631752](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210712134631752.png)

#### Summary

- Disco VMM hides NUMA-ness from non-NUMA aware OS
- Disco VMM is low effort (only 13K LoC)
- Moderate overhead due to virtualization
  - 16% overhead for uniprocessor workloads
  - System with 8 virtual machines can run some workloads 40% faster

#### Questions

1. What problem does this paper address?
   - Hardware iterates faster than software (OS), how to make OS compatible with existing and future hardware?
2. What is the author's main insight?
   - Add a layer between hardware and OS
   - Virtualize all resources of machine
3. What are the paper's key strengths?
   - Consolidate machines
   - isolate performance, security, and configuration
   - Moderate overhead of 16%
4. What are the paper's key weaknesses?
   - Number of VMs is small
   - Still moderate overhead

## 129 - Xen

#### Overview

- Xen, a VMM that allows multiple OSs to share conventional hardware in a safe and resource managed fashion, but without sacrificing either performance or functionality
- Provides an idealized virtual machine abstraction with minimal porting effort
- Hosting ~100 VM machines with 2-5% performance overhead
- Aim to run industry standard applications and services

#### Challenges

1. Isolate performance from each other, given mutually untrusted users
2. Support a variety of OSs
3. Small performance overhead

#### Why Paravirtualization?

<u>Drawbacks of Full Virtualization</u>

1. Supervisor instructions must be handled by VMM for correct virtualization, increased complexity and cost
2. It may be desirable that guest OS knows both real and virtual resources (e.g. time)
3. Moderate overhead

<u>x86 Virtualization</u>

- MMU uses hardware page tables
- Some privileged instructions fail silently instead of fault
- VMWare fixed this using binary rewrite
- Xen by modifying OS to avoid them

<u>Paravirtualization</u>

- Require modification to the OS (but not apps)
- Improve performance, resource isolation

#### VM Interface

![image-20210712162043203](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210712162043203.png)

<u>Memory Management</u>

- x86 has hardware-managed TLB
- No tagging allowed in x86, address space switches require a complete TLB flush
- Each TLB miss is served by the processor by walking the page table structure in hardware
  - All valid page translations for the current address space should be present in hardware-accessible page table
- Solution:
  - guest OSs responsible for allocating and managing hardware page tables. Need Xen to validate entries before write; guest OS has read-only access
  - Xen exists in the top of every address space, so avoiding TLB flushes when entering and leaving the hypervisor

<u>CPU</u>

- OS needs to run at a lower privilege level
- VMM in ring 0, OS in ring 1, Apps in ring 3
- Privileged instructions by guest OS are required to be validated and executed within Xen
- Exceptions (e.g. memory faults, software traps) are virtualized by a table describing the handler for each type of execution. The table is registered with Xen for validation.
- Two most frequent exceptions: system calls, page faults
  - page fault must be delivered to Xen because it requires ring 0 to read the faulting address from register CR2
  - improve performance of syscalls by allowing each guest OS to register a "fast" exception handler that is accessed directly by processor without requiring ring 0 indirect
- Safety is ensured by validating exception handlers when they are presented to Xen
- Each guest OS has a timer interface and is aware of both real and virtual time

<u>Device I/O</u>

- Xen exposes a set of device abstractions
- I/O data is transferred to and from each domain via Xen, using shared-memory, asynchronous buffer-descriptor rings
  - high-performance
  - passing buffer information vertically through the system
  - validation check from Xen possible
- Hardware interrupts are replaced with a lightweight event system

#### Control and Management

- Hypervisor itself only provides basic control operations
- Run the VMM management at user level
- Domain0:
  - manage the entire server
  - given special access to control interface for platform management
  - Ability to create and terminate other domains, control their associated scheduling parameters, physical memory allocations, access to physical disks and network devices

#### Design Details

<u>Control Transfer: Hypercalls and Events</u>

- VM to Xen: synchronous hypercall
  - software trap to Xen to perform privileged operations
  - e.g. page-table updates
- Xen to VM: async event mechanism
  - lightweight notification of important events
  - e.g. domain-termination requests

<u>Data Transfter: I/O Rings</u>

- necessary to have a data transfer mechanism that allows data to be moved vertically through the system with as little overhead as possible
- Asynchronous I/O rings are used to transfer data between Xen and guest OSs
  - zero-copy semantics
- A pair of consumer-producer pointers for each direction
- The structure is generic

<u>CPU Scheduling</u>

- Fair sharing scheduler: effective isolation between domains
- However, allows temporary violations of fair sharing to favor recently-woken domains
  - Reduce wake-up latency to improve interactivity
  - Fast event handling by domains

<u>Virtual Address Translation</u>

- Shadow page table is too costly: Xen needs to intercept and add physical -> machine translation for every change 
- Xen only involve in page table updates (to prevent guest OSes from making unacceptable changes)
  - register guest OS page tables directly with the MMU
  - restrict guest OS to read-only access
  - complete page table updates are passed to Xen, which will only need to validate before applying
- Validation:
  - associate a type and reference count with each machine page frame
  - Xen can ensure that only validated pages are used for the HW page tables
- To minimize the number of hypercalls, guest OS can batch updates before applying an entire batch with a single hypercall

<u>Physical Memory</u>

- physical memory is reserved at domain creation time
  - statically partitioned among domains

- guest OS has a physical -> machine mapping, allowing contiguous physical address space
- Xen has a table to map hardware to physical

<u>Network</u>

- Xen offers a set of Virtual Firewall/Router (VFR)
  - Each guest OS has one or more virtual interface (VIF) that is assigned to a VFR
  - the VFR both limit the guest OS and ensures correct incoming packet dispatch
- Exchange packet's page to a destination VM's free page on packet receipt
  - no copying required
- Bandwidth is round robin

<u>Disk</u>

- Xen provide abstraction of Virtual Block Device (VBD)
- A translation table is maintained within Xen for each VBD
  - entries are installed and managed by Domain0
- On receiving a disk request:
  1. Xen inspects the VBD identifier and offset
  2. produces the corresponding sector address and physical device
  3. Zero-copy data transfer done using DMA between the disk and pinned memory pages in the requesting domain

<u>Time and timers</u>

- Times:
  - Real time: time since machine boost
  - Virtual time: time that only advances in the context of VM
  - wall-clock time

- Each guest OS can program timers for both real and virtual time

#### Questions

1. What problem does this paper address?
   - A VMM that has minimal performance overhead, generic architecture requirement, isolate performance, security, configuration?
2. What is the author's main insight?
   - Minimally alter guest OSes to make VMs simpler and higher performance
   - Run admin at user level: Domain0 
   - I/O rings
3. What are the paper's key strengths?
   - Isolation of performance, security, and configuration
   - Support hardware-managed TLB; no TLB entry tagging
   - I/O rings support zero-copy communication
   - negligible overhead
4. What are the paper's key weaknesses?
   - Communication between domains?

## 130 - Live Migration of Virtual Machines

#### Why Migration is Useful?

1. Load balancing for long-lived jobs
2. System maintenance: controlled maintenance window
3. Fault tolerance: move job away from flaky (yet still alive) servers
4. Cost efficiency: rearrange load

#### Why VM migration instead of Process migration?

1. VM migration does not involve residual dependency:
   - where original machine needs to stay alive and network-accessible for the migrated process (e.g. open file descriptors, shared memory segments, local resources, syscalls, memory accesses)
   - VM migration allows the original machine to shutdown once migration is complete, allowing maintenance
2. VM migration allows migrating in-memory state:
   - in-memory state can be migrated in a consistent and efficient fashion
   - applies to kernel-internal state (e.g. TCL control block for an active connection) and application-level state, even if this is shared between multiple cooperating processes
   - Migrate without requiring clients to reconnect (impossible for application-level restart and layer 7 redirection)
3. Separate concerns between the user and the operator:
   - The user does not need to give operator any OS-level access permissions
   - The operator does not need to know what is happening in the VM, just migrate the entire OS and its attendant processes as a single unit

#### Design Goals

1. Low downtime: during which services are not available
2. Low total migration time: during which state on both machines are synchronized, may affect reliability
3. Disruption to active service: does not unnecessarily disrupt active services through resource contention (e.g. CPU, bandwidth) with the migrating OS

#### Design Overview

- Focus on physical resources: memory, network, and disk

<u>Memory Migration Options</u>

1. Push phase: source VM continues running while copying certain pages across the network to the destination VM. To ensure consistency, any modified pages need to be resent.
2. Stop-and-copy phase: stop the source VM, copy pages to the destination VM, start destination VM
3. Pull phase: the new VM executes, when a non-copied page is accessed, page fault happens across the network and copied in from source VM

<u>Pre-Copy Migration</u>

1. A bounded iterative push phase
   - all pages are copied in the first round
   - for other rounds, only copy modified pages
   - Bound the number of rounds based on writable working set (WWS) analysis
2. A typically very short stop-and-copy phase

- Carefully control network and CPU resources used

<u>Local Resources</u>

- Network resource: 
  - maintain all open network connections
  - a migrating VM will include all protocol state, and will carry IP address
  - Usually connected in a single switched LAN, new VM just broadcast its new physical location
- Disk
  - Network-attached storage (NAS), no storage migration considered

#### Migration Stages

0. **Pre-Migration**: begin with an active VM on physical host A. Preselect a target host B where resources required is guaranteed

1. **Reservation**: A sends a request for B, reserving a VM container at B
2. **Iterative Pre-Copy**: first round copy all pages; consecutive rounds copy dirtied pages
   - Can increase bandwidth used for later iterations to reduce time during which pages are dirtied
3. **Stop-and-Copy**: suspends A, redirect network traffic to B, copy all CPU state and inconsistent memory pages over. At the end of this stage, consistent copies at both A and B
4. **Commitment**: B notifies A that a complete OS image is received. A sends ack back. A can now abort, B becomes the primary host.
5. **Activation**: B is activated. Post-migration code attach device drivers to the new machine and broadcast moved IP address (update IP address -> MAC address translation).

#### Writable Working Sets

- The largest overhead is coherently transferring virtual machine's memory image
- Two kinds of pages:
  1. Seldom or never be modified - good to pre-copy
  2. Often updated (Writable Working Set) - good to stop-and-copy
- Different programs have different WWS behavior, but most are generally amenable to live migration

<u>Tracking WWS</u>

- Xen inserts shadow pages under the guest OS, populated using guest OS's page tables
- If a guest OS tries to modify a page of memory, the resulting page fault is trapped by Xen, so tracked by the shadow pages

#### Migration Types

1. Managed migration:
   - move the OS without its participation
   - can add some paravirtualization to stun processes that dirty pages too frequently or move unused pages out to reduce pages moved
2. Self migration:
   - OS participates in migration
   - hard to get a consistent OS because OS is participating

#### Questions

1. What problem does this paper address?
   - How to migrate VM with minimized downtime, low total migration time, and no unnecessary disruption to active services?
2. What is the author's main insight?
   - Bounded iterative pre-copy, and then stop-and-copy
   - Track Writable Working Set to reduce unnecessary copies
3. What are the paper's key strengths?
   - very low downtime (in tens of ms)
   - low total migration time (in s)
   - no unnecessary disruption
   - once migration is complete, the original VM can shutdown
4. What are the paper's key weaknesses?
   - Does not address storage
   - If some workload has very frequent dirty page percentage, turns out to be pure stop-and-copy

## 131 - An Updated Performance Comparison of Virtual Machines and Linux Containers

#### Goal

- Isolate and understand the overhead introduced by VM and containers, compared to non-virtualized Linux
- Identify the primary performance impact of each virtualization option

#### Background

<u>Motivation and Requirements for Cloud Virtualization</u>

- Unix's shared global filesystem lacks configuration isolation
  - Multiple applications can have conflicting system-wide configuration settings (e.g. shared library dependencies)
- Administrators and developers simplify deployment by installing each application in a separate OS copy, either a new machine or a new VM
- Customers want to get the performance they paid for, so provision fixed units of capacity with no oversubscription



<u>Kernel Virtual Machine (KVM)</u>

- KVM is a feature of Linux that allows Linux to act as a type 1 hypervisor
  - run unmodified guest OS in a Linux process
  - add hardware acceleration and paravirtual I/O to minimize virtualization overhead

<u>Linux Container</u>

- container-based virtualization modified an existing OS to provide extra isolation
  - built on the kernel namespaces feature: allows creating separate instances of previously-global namespaces
  - by adding a container ID to every process, add new access control to every syscall
- usually create a container that has no visibility or access to objects outside the container
  - processes running in a container appear to run on a normal Linux system, yet they share underlying kernel with processes from other namespaces
- Can contain as large as a system (system container) or as little as a process (application container)
- Application container consumes less RAM than system container or VM because it does not contain redundant management processes
- Application containers do not have separate IP addresses



<u>VM vs. Container</u>

|                         | Kernel Virtual Machine (KVM)                                 | Container (Docker)                                           |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Resource Isolation      | - VM has a static number of vCPUs and fixed amount of RAM<br />- cannot be resized | - use cgroups to group processes and manage aggregated resource consumption<br />- can be resized<br />- apps don't know their resource limit |
| Management              | - easy to use via management tools (e.g. libvirt)            | - Several management tools are available, e.g. Docker (i.e. has layered filesystem image) |
| Scheduling              | - Two levels: VM and VMM                                     | - Only one level (Linux OS) of resource allocation and scheduling |
| Performance Consistency | - vCPU can be descheduled, RAM can be swapped out, without any notice | - Applications may not know its container's limit            |
| Security Isolation      | - Restrict VM's communication with outside world: limited number of hypercalls or emulated devices (via hypervisor) | - Cannot see/access what is in other namespaces (container)<br />- root within one namespace is not root in another namespace <br />- Share IP address with other containers<br />- non-namespace-aware syscalls can introduce accidental leakage |
| Communication           | - All communications through VMM, hard to communicate        | - Easy to communicate among containers and between container and the host |
| Overhead                | - add overhead when sharing data between VMs or between guest and VMM<br />- in the cloud, VM access storage through emulated block devices<br />- hardware acceleration<br />- paravirtual I/O devices | - Startup time for container is way less than VM             |

#### Evaluation

- VM: CPU performance loss for workloads that depend on hardware-specific optimizations
- VM: memory performance loss due to nested paging, which makes TLB misses more costly
- VM: disk/network I/O performance loss due to emulated I/O instruction

#### Questions

1. What problem does this paper address?
   - Compare and contrast between VM and Container.
2. What is the author's main insight?
   - VM has more performance overhead due to its lower-level of isolation
   - Container is more performant, easier to deploy, higher elasticity
   - VM is more secure
3. What are the paper's key strengths?
   - Demonstrates the workloads where VMs perform badly compared to containers
4. What are the paper's key weaknesses?
   - Compare and contrast, so N/A?

## 132/133 - Cloudlets

#### Overview

- Currently, cloud providers support mobile application's backend through datacenters, which is centralized.
- End-to-end communications require many hops, so high latencies and low bandwidth
- Cloudlet reduces the number of hops by locating close to a cellular base station
- Workload aimed by cloudlets:
  - resource-intensive
  - interactive

#### Cloud vs. Cloudlet

<u>Similarity</u>

- Need strong isolation between user-level applications
- Mechanism for authentication, access control
- Dynamic resource allocation for user-level applications
- Support a variety of OS, applications
- VM used for both cloud and cloudlet

<u>Differences</u>

- Rapid provisioning: cloudlet needs to be more agile, highly dynamic connection and setup
- VM Migration: when move from cloudlet to cloudlet, VM needs to be migrated, in a WAN fashion

<img src="/Users/hanminglu/Library/Application Support/typora-user-images/image-20210714172357333.png" alt="image-20210714172357333" style="zoom:50%;" />

#### VM Fork

- Stateful swift cloning of VMs
- State inherited up to the point of cloning
- Local modifications are not shared
- VM cloning makes an impromptu cluster

#### Cloudlet

<u>Cloudlet Customization</u>

![image-20210714165926819](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210714165926819.png)

- Cloudlet preload base VM
- mobile device provides VM overlay to run its customized VM
- reduce setup time

#### Limitations?

- How to achieve persistent storage?
  - Send local storage volume to cloudlet at the beginning and receive changes at end of computation?
- How to find and negotiate for computational resources?
- Security of physical cloudlet machines?

#### Questions

1. What problem does this paper address?
   - How to reduce latency and improve bandwidth for resource-intensive, interactive mobile workloads?
2. What is the author's main insight?
   - Use cloudlet that is close to cellular tower
   - 1-hop latency
   - use VM overlay to achieve fast VM launch
3. What are the paper's key strengths?
   - low latency, high bandwidth
   - fast VM startup
4. What are the paper's key weaknesses?
   - security, persistent storage, cloudlet negotiation

## 134 - Mesos

#### Overview

- Cluster computing frameworks are used to deploy different applications
- Different frameworks are suitable for different applications, there is no framework that suits all
- Within each framework, there is fine-grained resource sharing that allows data locality, scalability, high utilization.
- However, there is no way to perform fine-grained resource sharing across frameworks as they are deployed independently
- Mesos is a platform to share a commodity cluster with multiple cluster computing frameworks, where each framework is the most appropriate one for each application
  - Improve cluster utilization
  - Applications can share access to large datasets that are costly to replicate at a per-application basis

<u>Challenge</u>

- How to build a thin layer that is scalable and efficient
  1. Each framework has different scheduling needs (due to programming model, communication pattern, task dependencies, data placement)
  2. Must scale to clusters of tens of thousands of nodes, hundreds of jobs, millions of tasks
  3. Need to be fault tolerant and highly available as all frameworks depend on it

<u>Limitation of a centralized scheduler</u>

- A centralized scheduler: takes input on framework requirements, resource availability, organizational policies, then computes a global schedule for all tasks

1. Complexity: need sufficiently expressive API to satisfy all frameworks' requirements
2. Comprehensiveness: if new requirement/policy comes out, need to include all
3. Framework's scheduling: frameworks have sophisticated scheduling already, not feasible to port them

#### Mesos' Approach

- Mesos provides a new abstraction called resource offer, which encapsulates a set of resources a framework can allocate on a cluster node
  - Mesos decides how many resources to offer
  - Frameworks decides which resources to accept and which tasks to perform
- Properties:
  - Not globally optimal, but works very well; 
  - simple and efficient to implement
  - Scalable and fault tolerant

#### Architecture

- Design philosophy is to design a minimal interface that:
  - enables efficient resource sharing across frameworks
  - push task scheduling and execution to the framework
  - Pros: allow frameworks to implement their own approaches, keeps Mesos simple and minimize the rate of change required
- Mesos consists of a master node that manages slave daemons on each worker node, and frameworks run tasks on worker nodes

<u>Resource Allocation</u>

- The master implements fine-grained sharing across frameworks using resource offers
  - each resource offer is a list of free resources on multiple slaves
  - the master determines how many resources to offer each framework, according to an organizational policy (e.g. fair sharing)
  - Mesos let organizations define their own inter-framework allocation policies
  - Can kill tasks if they are too long
- Each framework on Mesos consists of:
  - a scheduler, registers with master to be offered resources, 
    - selects which of the offered resources to use, pass Mesos a description of the tasks it wants to launch. 
    - Reject if resource not wanted.
    - allow filters to optimize
    - delay scheduling to acquire nodes storing their data
    - Can indicate interests to have more resources
  - an executor process on each slave node, used to run the framework's tasks

<img src="/Users/hanminglu/Library/Application Support/typora-user-images/image-20210715163114092.png" alt="image-20210715163114092" style="zoom:50%;" />

<u>Isolation</u>

- Use OS Container to isolate performance, resources

<u>Fault Tolerance</u>

- Master hold soft state only: active slaves, active frameworks, running tasks
  - sufficient to compute how many resources each framework is using and run the allocation policy
- A new master can completely reconstruct master's internal state from information held by the slaves and framework schedulers
- Multiple masters using a hot-standby configuration using ZooKeeper

#### Mesos Behavior

- Performs better with elastic framework, homogeneous task duration, and prefer all nodes equally

#### Questions

1. What problem does this paper address?
   - How to enable fine-grained resource sharing across cluster computing frameworks?
2. What is the author's main insight?
   - Provide a thin layer between frameworks and cluster nodes
   - Mesos provides resource offer 
   - Frameworks are responsible for accept/reject offers, schedule tasks, and execute tasks
3. What are the paper's key strengths?
   - Increase resource utilization
   - Applications can share large datasets on the cluster instead of replicating them
   - Can satisfy a diverse set of frameworks
   - high scalability, fault tolerance, and high availability
4. What are the paper's key weaknesses?
   - Not globally optimal scheduling

## 135 - Kubernetes

#### Predecessors of Kubernetes

<u>Borg</u>

- Manage resource sharing between long-running latency-sensitive services and batch jobs

<u>Omega</u>

- Improve the software engineering of Borg
- Omege stores the state of the cluster in a centralized Paxos-based transaction oriented store
- Use optimistic concurrency control to handle the occasional conflicts
- Decouple Borg's functionality into separate components that act as peers instead of a monolithic centralized master

#### Containers

- resource isolation
- cannot provide isolation for resources that OS kernel doesn't manage (e.g. level 3 cache, memory bandwidth)
- Modern container provides image (files that make up the application that runs inside the container)
  - a hermetic container image that encapsulates almost all of an application's dependencies into a package that can be deployed into a container
  - Only local external dependencies will be Linux kernel system-call interface

#### Application-Oriented Infrastructure

- Container transforms data center from machine-oriented to application-oriented
- Container abstracts away machine and OS from the application developer
  - improves application development
  - allow faster development of hardware

#### Containers as the unit of management

- Monitor/Log application instead of machine
- Easier to build, manage, and debug applications

#### Kubernetes

<u>Design</u>

- Outermost container is called a pod, always run an application container inside of a top-level pod
  - Each pod can hold an instance of a complex application, where each part of the application sits in one of the pod's child containers
- Choreography: achieving a desired emergent behavior by combining the effects of separate, autonomous entities that collaborate
  - in contrast to a centralized orchestration system, which may be easier to construct at first, by become brittle and rigid over time

#### Questions

1. What problem does this paper address?
   - How to provide virtualization with efficient resource allocation, application-oriented approach, and auto-scaling for long-running jobs and batch jobs altogether?
2. What is the author's main insight?
   - 
3. What are the paper's key strengths?
4. What are the paper's key weaknesses?

## 136 - C-Store

#### Row Store

- row store: most databases are record-oriented storage systems, where the attributes of a record are place contiguously in storage
- a single disk write suffices to push all of the fields of a single record out to disk
  - write-optimized, high write performance
  - insert or delete record in one physical write
- Effective for on-line transaction processing (OLTP), where
  - transactions involve small numbers of records
  - Frequent updates
  - Many users
  - Fast response time
- Easy to add new record, but might read unnecessary data (wasted memory and I/O bandwidth)

#### Column Store

- column store: the values for each single column are stored contiguously
- Read-optimized, oriented toward ad-hoc querying of large amounts of data
- Effective for On-Line Analytical Processing (OLAP), where
  - transactions involve large numbers of records
  - Frequent ad-hoc queries and infrequent updates
  - A few decision-making users
  - Fast response times
  - Read-mostly
- Intuition: **Only read relevant columns**
- A DBMS only reads the values of columns required for processing a given query, avoiding bring into memory irrelevant attributes
- In data warehouse environment where typical queries involve aggregates performed over large numbers of data items, a column store has sizeable performance advantage
- Cons:
  - tuple write needs multiple seeks
- Projection: 
  - C-store physically stores a collection of columns, each sorted on some attribute
  - A group of columns stored on the same attribute is a projection; the same column may exist in multiple projections
- Tuple mover:
  - A Writable Store (WS) component that provides efficient write operations
  - A Read-optimized Store (RS) component that provides efficient bulk read
  - A tuple mover that periodically moves new data from WS to RS
- Consistency:
  - snapshot isolation
- Column-oriented optimizer and executor

<u>Trade CPU cycles for Disk Bandwidth</u>

1. Code data elements into a more compact form
2. Densepack values in storage (compression)

#### Questions

1. What problem does this paper address?
   - How to support read-optimized workload?
2. What is the author's main insight?
   - Use column store instead of row store
   - Do not read unused data
   - Trade CPU cycles for disk bandwidth, operate on compressed data
3. What are the paper's key strengths?
   - Faster for read-only workloads
   - Both WS and RS to support efficient read and write workloads
   - Use snapshot isolation to provide lock-free fast queries
4. What are the paper's key weaknesses?
   - Tuple mover may impact performance when the database is constantly at high workload

## 137 - Database Cracking

#### Background

- Many DBMS perform well and efficiently only after tuned by a DBA
  - DBA decides:
    - Which indices to build?
    - On which data parts?
    - When to build?
  - Timeline:
    1. Sample workload
    2. Analyze performance
    3. Prepare estimated physical design
    4. Queries
- Tuning by DBA is a complex and time-consuming process
- Hard to do for dynamic workload or a very large database

#### Database Cracking

- Design new auto-tuning kernels
  - continuous on-the-fly physical reorganization
  - Partial, incremental, adaptive indexing
  - For modern column stores
- Each query is used as advice on how data should be physically stored
  - triggers physical re-organization of the database

#### Cracker Design

- The first time a range query is posed on an attribute A, a cracking DBMS makes a copy of column A, and call it cracker column A
- A cracker column is continuously physically reorganized based on queries that need to touch attribute A, such that the result is in a contiguous space
- For each cracker column, there is a cracker index

![image-20210721162220068](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210721162220068.png)

<u>Cracker Select Operator</u>

1. Searches the cracker index
2. Physically re-organize the pieces found
3. update the cracker index
4. Return a slice of the cracker column as a result

- Faster because analyzes less data (even though more steps)

#### Questions

1. What problem does this paper address?
   - How to physically store tuples such that no DBA is required to support fast queries?
2. What is the author's main insight?
   - Continuously physically re-organize database based on queries
   - Use queries as evidence on where to store tuples
3. What are the paper's key strengths?
   - Does not require DBA tuning for efficient DBMS performance
   - Can adapt to dynamic workload or large database
4. What are the paper's key weaknesses?
   - DBA may have a better knowledge
   - May not converge to the efficient layout

## 138 - Chord

#### Background

- A fundamental problem of peer-to-peer applications is to efficiently locate the node that stores a particular item, given that nodes are dynamically changing

#### Chord

<u>Overview</u>

- Chord is a distributed lookup protocol that does one thing: given a key, it maps the key onto a node
  - Chord adapts efficient as nodes join and leave the system
  - Can answer queries even if nodes are continuously changing
  - scalable: communication cost and the state maintained by each node scales logarithmically with the number of Chord nodes

- Chord is a variant of consistent hashing:
  - Load balancing as each node receives roughly the same number of keys
  - Little key movement when nodes join and leave the system
- In steady state:
  - Chord node only needs information on O(logN) nodes
  - a node resolves all lookups by communicating with O(logN) nodes
- When node join and leave:
  - Chord maintain its routing info as nodes join and leave
  - high probability that each event results in no more than O(log^2(N)) messages
- Chord is simple, provable correct, provable performant
- A node only needs one piece of information to ensure correctness, although slow

#### System Model

- **Load Balancing**: natural load balancing provided by consistent hashing, keys are evenly spread over nodes
- **Decentralization**: all nodes run the same software, no node is more important than others. Better robustness. Appropriate for loosely-organized peer-to-peer network
- **Scalability**: Lookup cost (space, communication) is logarithmically to the number of nodes
- **Availability**: highly available, allow continuous changes to internal tables to reflect node joins and leaves. Except for major network failure, the responsible node can always be found
- **Flexible Naming**: no naming structure is required. A flat namespace.

#### The Base Chord Protocol

<u>Consistent Hashing</u>

- Key and Node ID are generated by hashing the key and the node IP
- ID are ordered in an identifier circle module 2^m
- Each key is assigned to the node whose ID is the same or follows the key ID
- Assume the key load on a node may exceed by at most O(logN) 

<u>Finger Table</u>

- Each node keeps a routing table with (at most) m entries, where m is the length of ID
- The ith entry at node n contains the ID of the first node that succeeds by at least 2^(i-1) on the identifier circle
- A finger table entry contains the node ID and its IP+port
- Two characteristics:
  1. One node knows the nodes following it better than further away ones
  2. Information on each node is not enough to resolve most keys
- To search for a key, find in finger table for the node that is greater than its own ID and smaller than the key ID, ask it
  - If none, then return successor

<u>Node Join</u>

- Two invariants for correctness:
  1. Each node's successor is correctly maintained
  2. For each k, successor(k) holds k
- Correct finger tables is for fast lookup
- Each node maintains a predecessor pointer, which points to the node immediately before it
- Three tasks when a node n joins the network:
  1. Initialize the predecessor and finger table of node n
  2. Update the predecessor and finger table of existing nodes to reflect addition of n
  3. Notify the higher layer software so that it can transfer values associated with keys that node n is now responsible for
- Assume new node n knows an existing node n' through external mechanism

1. Initialize fingers and predecessor:
   - Node n knows its predecessor and fingers by asking n' to look them up
   - For each of fingers, n' performs a lookup. An optimization is to check if ith finger is the same as (i+1)th finger
   - Overall time becomes O(log^2(N))
2. Update fingers of existing nodes:
   - Node n will be the ith finger of node p if and only if
     1. p precedes n by at least 2^(i-1)
     2. the ith finger of node p succeeds n
   - Algo starts with the ith finger of node n, and then continues to walk in the counter-clock-wise direction on the id circle
   - O(log^2(N))
3. Transfer keys:
   - Contact the node immediately follows n only
   - move the data associated with keys to n

<u>Stabilization</u>

- To ensure correctness, stabilization protocol keeps nodes' successor pointers up to date
- Those successor pointers are then used to verify and correct finger table entries
- Every node runs stabilize periodically, which asks n's successor for its predecessor p, and decide if p should be n's successor instead

<u>Failures</u>

- Each node keeps a successor-list that tracks r nearest successors on the Chord ring
- If the node fails, contact the next successor node on the list
- For node far away, use finger table to contact a closer node

<u>Replication</u>

- Use successor-list to store replicas on nodes succeeding n

#### Questions

1. What problem does this paper address?
   - A simple, provable correct, provable performant distributed lookup protocol for decentralized applications?
2. What is the author's main insight?
   - Consistent hashing
   - Finger table
   - Separate performance and correctness
3. What are the paper's key strengths?
   - simple, provable correctness, provable performance
   - Separate correctness and performance
4. What are the paper's key weaknesses?
   - Latency vs. memory: finger table -> more memory and less latency
   - fixed ID size: could be problematic if variance in network size

## 139 - Dynamo

#### Background

- Dynamo is a highly available data storage
- A simple key/value interface
- highly available with a clearly defined consistency window, efficient in its resource usage, a simple scale out scheme
- Data is partitioned and replicated using consistent hashing
- Consistency is facilitated by object versioning
- Consistency among replicas is maintained by a quorum-like technique and a decentralized replica synchronization protocol

#### System Assumptions and Requirements

<u>Query Model</u>

- Simple read and write operations to a data item that is uniquely defined by a key
  - state is stored as binary objects identified by unique keys
- No range operations
- No relational schema
- Target applications that store small objects

<u>ACID Properties:</u>

- ACID (Atomicity, Consistency, Isolation, Durability)
- Target applications that operate with weaker consistency, but high availability
- Dynamo does not provide isolation guarantees
- Only single key updates

<u>Efficiency</u>

- System needs to run on commodity hardware
- Storage system must meet stringent SLAs
- Tradeoffs are performance, availability, cost efficiency, and durability

<u>Other Assumptions:</u>

- No security related requirements
- Needs to be highly scalable - hundreds of storage hosts

#### Design Considerations

<u>Consistency vs. Availability</u>

- There is a tradeoff between consistency and availability when network failure happens
- Traditional systems ensures consistency, where data is made unavailable until it is absolutely certain it is correct
- Optimistic Replication Technique:
  - Increase availability by propagating changes to replicas in the background, and concurrent, disconnected work is tolerated
  - challenge: leads to conflicting changes, which need to be detected and resolved
  - Question: when and who to resolve conflicts?
- Dynamo is designed to be an eventually consistent data store: all updates will reach all replicas eventually
- When:
  - Should conflicts be resolved during reads or writes?
  - If during write: writes could be rejected when it cannot reach to all replicas, hurts availability; simpler reads
  - If during read: highly available for writes; complex reads
- Who:
  - Should conflicts be resolved by application or data store?
  - if Data Store: does not have information from application level, can only use simple techniques such as "last write wins"
  - if application: can have higher-level resolve techniques based on application need

<u>Incremental Scalability</u>

- Dynamo should be able to increment by one host at a time with minimal impact on the system operator and the system itself

<u>Symmetry:</u>

- All hosts should have the same responsibilities as their peers; no distinguished node
- This reduces the cost of provisioning and maintenance

<u>Decentralization</u>

- In addition to symmetry, the design should favor decentralized peer-to-peer techniques instead of centralized control
- System is simpler, more scalable, and more available
- Centralized control may result in outages

<u>Heterogeneity</u>

- The system should exploit the heterogeneity of individual hosts
- Work distribution should leverage the capability of hosts

#### System Architecture

- Core distributed system techniques: partitioning, replication, versioning, membership, failure handling, scaling

<u>System Interface</u>

- get(key): returns the object associated with the key or a list of objects, each with conflicting versions and a context
- put(key, object, context): uses key to determine where to place replicas of the object. Context includes system metadata on the object, version of the object, etc.
- Context is stored along with object, so the system can verify that the put caller has validity
- Dynamo treats both key and object as an opaque array of bytes
- Hash of key is used to determine which nodes are responsible for storing/serving the key

<u>Partitioning</u>

- Challenges of basic consistent hashing:
  - Random positioning of nodes may result in non-uniform data and load distribution
  - Does not account for heterogeneity in nodes (good nodes should take more load)
- Consistent hashing with virtual node:
  - each physical node can take multiple virtual nodes
  - advantages:
    - when a node leaves, its workload is evenly distributed to others
    - when a node joins, it takes roughly equivalent workload as others
    - Good nodes can take more virtual nodes (heterogeneity)

<u>Replication</u>

- The coordinator stores the key it is responsible for, and replicate it to N successor nodes
- Preference list: the list of distinct physical nodes responsible for storing a particular key

<u>Data Versioning</u>

- Dynamo provides eventual consistency, where updates can be propagated to all replicas asynchronously
  - a subsequent get() may not return the most recent put()
- Each modification is treated as a new and immutable version of the data
  - Allows multiple versions of the same data to be present
- Version branching:
  - failures + concurrent updates: conflicting versions may happen
  - need client to merge and collapse multiple branches
  - Applications need to explicitly acknowledge multiple versions
- Vector clock:
  - Use vector clock to capture relationships between versions (parallel vs. causal)
  - when a client updates, it must specify which version it is updating using the context parameter
- If multiple versions of an object exist, and cannot be syntactically reconciled, get() will return all existing objects with their context
  - client is responsible for semantically reconcile them
- To limit the amount of (node, counter) pairs, remove the oldest pair when a threshold is reached

<u>Execution of put() and get() requests</u>

- Client either sends requests to 1) a load balancer that routes to a random node; 2) to the coordinator (lower latency)
- The coordinator is the first of top N nodes in the preference list
- Consistency protocol R and W:
  - R: the minimum number of nodes that must participate in a get() request
  - W: the minimum number of nodes that must participate in a put() request
  - If R + W > N, a quorum-like system
  - total latency is the latency of the slowest node
- put():
  1. coordinator stores locally
  2. coord sends to N highest-ranked reachable nodes
  3. Wait for W-1 responses
  4. return
- get():
  1. coordinator sends to all N highest-ranked reachable nodes
  2. wait for R responses 
  3. return (send all conflicting versions)

<u>Handling Failures: Hinted Handoff</u>

- Hinted handoff:
  - When a node (e.g. A) is unavailable during a write, a temporary node (e.g. D) will receive the replica. This ensures availability and durability guarantees.
  - The hinted replica is stored on a separate local storage, and knows it is supposed to be on A
  - When A comes back, D sends the replica to A, and delete its own copy
- To ensure highest availability, set W = 1
- To have better durability, set greater W value
- Tolerate datacenter outage by setting preference list to include multiple datacenter nodes

<u>Handling Permanent Failures: Replica synchronization</u>

- To handle threats to durability, Dynamo implements a replica synchronization protocol
- Use one Merkle tree per virtual node's key range, each leaf is the hash of a key's value
- Two hosting nodes can check if they have the same replicas by exchanging and comparing the root hash
  - if different, recursively does it to its children, eventually find the key differences
- Merkle tree advantages:
  1. each branch can be checked individually without requesting other branches
  2. reduce the data exchanged

<u>Membership and Failure detection</u>

- Assume outages are mostly temporary, use explicit membership addition or removal
- The node that receives the request to add/remove stores it to persistent storage
- Nodes periodically exchange, wants eventual consistency on view:
  1. membership history information
  2. mappings between virtual nodes and key ranges (allows direct forwarding of requests)
- Purely local notion of failure detection:
  - a node sees another node as temporary down if not responding messages
  - repeatedly check unavailable nodes to see when it becomes alive

#### Questions

1. What problem does this paper address?
   - How to design a highly available decentralized data storage system?
2. What is the author's main insight? What are the paper's key strengths?

| Problem                            | Solution                                    | Advantages                                                 |
| ---------------------------------- | ------------------------------------------- | ---------------------------------------------------------- |
| Partitioning                       | Consistent hashing                          | Incremental scalability                                    |
| Highly available for writes        | data versioning with vector clock           | always writable, client resolves semantic conflicts        |
| Handling temporary failures        | sloppy quorum and hinted handoff            | high availability and durability during node failures      |
| Recovering from permanent failures | replica synchronization through merkle tree | synchronize in the background with minimal data exchanges  |
| Membership and failure detection   | gossip-based protocol                       | avoid a central identity to manage membership and failures |

4. What are the paper's key weaknesses?
   - No security aspects?

## 140 - Tapestry

#### Overview

- Tapestry is a P2P overlay network that provides high-performance, scalable, and location-independent routing of messages to close-by endpoints, using only localized resources.
  - Core is a Decentralized Object Location and Routing (DOLR)
- Aim for efficiency:
  - minimize message latency
  - maximizing message throughput
- Tapestry exploits locality in routing messages to mobile endpoints such as object replicas
- Use adaptive algorithms with soft state to maintain fault tolerance under changing node membership and network faults
- Tapestry virtualizes resources, objects are known by name instead of locations

#### Routing and Object Location

<u>Routing Mesh</u>

- Each node stores a neighbor map
  - Each level stores neighbors that match a prefix up to a certain position in the ID
  - Construct locally optimal routing tables from initialization and maintains them in order to reduce routing stretch
- Each ID is mapped to a live node called the root
- Each message is aiming to reach the ID
- There are multiple hops for a message to find the root node:
  - At each hop, a message is progressively routed closer to G by incremental prefix routing
  - Each neighbour map has multiple levels, each level contains links to nodes matching up to a certain digit position in the ID
  - e.g. level 1 has links to nodes that have nothing in common, level 2 has first digit in common, etc.
- Routing takes O(log_B(N)), N is the namespace size, B is the ID base (e.g. hex=16)

<u>Object Publication and Location</u>

- Participants in the network can publish objects by periodically routing a publish message toward the root node
  - Each node on the path store the pointer mapping the object
- Objects are located by routing a message towards the root of the object
  - Each node along the path checks the mapping and redirects the request appropriately

#### Questions

1. What problem does this paper address?
   - How to provide efficient, location-aware distributed routing?
2. What is the author's main insight?
   - Routing table with neighbours that have x digits of prefix.
   - Every hop is progressively closer to the root
   - locality-aware routing
3. What are the paper's key strengths?
   - locality-aware
   - efficient
   - adapt to lower-level network failures
4. What are the paper's key weaknesses?
   - Algo is more complicated than Chord etc.

## 141 - Pond

#### Assumptions

1. infrastructure is untrusted except in aggregate
2. infrastructure is constantly changing without notices

#### Data Model

- A data object is similar to a file in a traditional file system
- Data objects are ordered sequences of read-only versions
- Each version:
  - contains metadata, data, reference to previous versions
  - stored in a data structure similar to a B-tree, where a block reference each child by a cryptographically-secure hash of child's content (i.e. Block GUID)
  - Version GUID is the root block's BGUID
  - copy on write

<u>Application-specific Consistency</u>

- Updates are applied atomically and are represented as an array of potential actions, each guarded by a predicate
- Actions could be replacing bytes in a file, appending new data, etc
- Predicates could be checking the latest version number of the object
- Allow application to specify predicates required
- Not support explicit locks or leases, instead relying on update model

#### System Architecture

<u>Virtualization with Tapestry</u>

- Resources are virtual, where they are not tied to a specific physical instance, can be moved anytime
- A virtual resource is named by a GUID, contains the state required to provide some service
- Use Tapestry to locate resources, manage physical nodes, route messages

<u>Replication and Consistency</u>

- Data blocks are read only, so they can be replicated without worrying about consistency issues
- Mapping from a data object (AGUID) to its latest version (VGUID) needs to be updated in a consistent manner:
  - each data object has a primary replica, which is responsible for serializing and atomically applying updates with a BFT certificate within its inner ring

<u>Archival Storage</u>

- Erasure code: any m of n fragments can reconstruct the message
  - achieve higher fault tolerance with the same amount of storage overhead
- After an update, all data blocks are erasure coded, all encoded segments are distributed to nodes in the system using a deterministic function based on its fragment number and BGUID
- To read a block, a host collects m fragments from different nodes, and decode

<u>Caching</u>

- For each read, first query Tapestry to look for a cached copy of the data block
- After constructing a data block from fragments, publish cached copy to Tapestry

![image-20210726223931040](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210726223931040.png)

#### Questions

1. What problem does this paper address?
   - How to build a decentralized storage system where infrastructure is untrusted unless in groups?
2. What is the author's main insight?
   - Use Tapestry to do the routing and resource management
   - Use primary replica inner ring and BFT signature threshold to ensure inner ring is BFT
   - Use erasure encoding to achieve high durability
   - Use caching to amortize frequent block reads
   - Data blocks are read-only, simplifying replication and consistency
3. What are the paper's key strengths?
   - BFT decentralized storage system
   - Efficient read by utilizing caching
   - High durability using erasure coding
4. What are the paper's key weaknesses?
   - Write is expensive due to BFT protocol, erasure coding, dissemination tree, etc.

## 142 - The Google File System (GFS)

#### Key Observations in Workload

1. Component failures are norm instead of exception. The system needs constant monitor, error detection, failure tolerance, automatic recovery
2. Modest amount of large files (multi-GB)
3. The majority of write operations is appending. Rare overwriting existing data, no random write. 
4. two kinds of reads: large sequential reads and small random reads
5. Support concurrent appends to the same file, need atomicity with minimal synchronization overhead
6. High sustained bandwidth is more important than low latency

#### File System Interface

- Traditional operations: create, open, close, read, write
- Special operations: snapshot and record append
  - snapshot: creates a copy of a file or a directory tree at low cost
  - record append: allows multiple clients to concurrently append to a file while guaranteeing atomicity 

#### Architecture

- A single master, multiple chunkservers, multiple clients
  - Master: maintains all metadata in the system, including namespace, access control information, mapping from files to chunks, locations of chunks. Controls system-wide activities such as chunk lease management, garbage collection, chunk migration
  - Chunkserver: store data chunks
  - Client: asks master for specific chunks' locations, contact chunkserver directly for data chunks and byte range within the data truncks
- No caching on client or chunkserver
  - client: no caching since mostly streaming read or working set too large to cache
  - chunkserver: since local files, so Linux's buffer cache already caches

#### Single Master & Multiple Chunkservers

- Simplifies system design and allows sophisticated chunk placement and replication decisions
- Minimize master's involvement in read and writes to prevent bottleneck
- Master only handles metadata-related requests, no data transfers
- After receiving locations of requested chunk index, the client sends a request (chunk index + byte offset) to the closest replica for actual data

#### Chunck Size

- Large chunks: size of 64MB
- Advantages:
  1. Reduce the number of communications required from client to master for chunk locations, especially for sequential read and write large files
  2. Reduce metadata size stored on master, can be stored in memory
- Disadvantages:
  1. internal fragmentation: can be mitigated with lazy allocation
  2. For a small file that consists of only a small number of chunks, chunkservers storing it can be hot spot. Mitigated with a higher replication factor

#### Metadata

- Master stores three types of metadata:
  1. File and chunk namespaces (persisted by logging)
  2. File-to-chunk mapping (persisted by logging)
  3. chunk-to-chunkserver mapping
- Log mutations to operation log on the master's local disk
- Operation log is replicated to remote machines for fault tolerance
- Does not store chunk-to-chunkserver mapping, instead the master asks chunkservers for chunk information at startup

<u>In-Memory Data Structure</u>

- Metadata is stored in memory -> fast master operations
- In-memory data structure is compact:
  - Less than 64Bytes per file (using filename prefix compression)
  - Less than 64Bytes per 64MB chunk (chunk location + chunk size)
- Master periodically scans its entire state to:
  1. garbage chunk collection
  2. re-replication after chunkserver failure
  3. chunk migration to load balance

<u>Chunk Location</u>

- Master gains chunk location information by asking all chunkservers at startup
- Then the master is up-to-date since it is managing chunk placement and monitor chunkserver status with HeartBeat msgs
- The reason why not persisting chunk locations is that the chunkserver has the final word on what it has (e.g. errors may corrupt disks)

<u>Operation Log</u>

- Operation log contains the historical records of critical metadata changes
- Changes to metadata should not be seen by the clients before they are persisted to both master and remote disks
- Checkpointing is used to enable faster recovery
  - internal state is structured such that checkpointing doesn't delay new changes
  - In particular, master starts a new log file and creates the new checkpoint on a new thread

#### Consistency Model

<u>Metadata Mutations</u>

- File namespace changes are solely handled by the master, so atomic
  - use locking to guarantee atomicity and correctness
  - use operation log to determine an order

<u>File Region Mutation Result Types</u>

- Defined: consistent + clients will see mutation writes in its entirety
  - defined when a mutation succeeds without interference from concurrent writers
- Consistent (not defined): all clients will see the same data, regardless of which replicas; it may not reflect what any one mutation has written; consists of mingled fragments from multiple mutations
  - consistent but not defined when concurrent successful mutations
- inconsistent (not defined): different clients may see different data at different times

<u>Data Mutation</u>

1. Write:
   - data to be written at an application-specified file offset
2. Record Append:
   - data (i.e. a record) is appended atomically at least once even in the presence of concurrent mutations, but at an offset of GFS's choosing
   - The offset is returned to the client, is the beginning of a defined region that contains the record
   - GFS may insert padding or record duplicates in between

- After a sequence of successful mutations, the mutated file region is guaranteed to be defined, and contain the data written by the last mutation. Achieved by:
  1. Applying mutations to a chunk in the same order on all its replicas
  2. using chunk version numbers to detect replicas that are stale due to chunkserver downtime
- Stale replicas will never involve in mutations or given to clients, they are garbage collected
- Stale client caches may allow clients to read from stale chunks, but not serious damage as it mostly returns a premature end of chunk rather than outdated data. The reader gets fresh data when it retries and contacts the master
- Data corruption is detected using checksumming
- Once a failure occurs, data is re-replicated from valid replicas as soon as possible

<u>Implications for Applications</u>

- Checkpoints may include application-level checksums to check how much has been successfully written
  - Readers verify and process only the file region up to the last checkpoint (defined state)
  - Appending is far more efficient and resilient to application failures than random writes
  - Checkpointing allows writers to restart incrementally
- Concurrent record appends:
  - record append-at-least-once
  - may have padding and duplicates between records, rare record duplicates
  - Reader can identify and discard extra padding and record fragments using checksums
  - To remove duplicate records, filter using unique ID in the records

#### Leases and Mutation Order

- Each mutation is performed on all the chunk's replicas
- Lease:
  - the master grants a chunk lease to a replica (the primary)
  - The primary picks a serial order for all mutations to the chunk
  - All replicas follow this order when applying mutations
- Lease has a timeout, but can be extended indefinitely
- Lease grant and request are piggybacked on HeartBeat msgs
- Lease minimizes management overhead of the master

<u>Control Flow Steps:</u>

1. Client asks master for primary and secondary replica locations
2. Client pushes data to all replicas, wait for all acks
3. Client sends primary a request to write
4. Primary serializes mutations, apply to local
5. Primary sends the same order to all secondaries, wait for all acks
6. Primary return to client as success

#### Data Flow

- Control and data flows are separated
- Data is pushed linearly along a chain (instead of a tree) of chunkservers in a pipelined fashion
  - avoid network bottlenecks
  - avoid high-latency links
  - fully utilize network bandwidth
  - minimize latency
- Push to the machine closest to you, starts immediately after receiving some data

#### Atomic Record Appends

- Client only specifies the data, GFS appends it to the file at least once atomically at an offset determined by the GFS
- Add additional logic to the write control flow:
  - after client send record append to the primary, the primary checks if appending the record will exceed the maximum size
    - if no, append at the current end of file. Done.
    - if yes, pad the current chunk, create a new chunk, and ask the client to try again
- After successful record append,
  - does not guarantee all replicas are bytewise identical
  - guarantee that data is written at least once as an atomic unit
    - the data has to be written to the same offset on all replicas of some chunk
    - all replicas are at least as long as the end of record
    - any future record will be assigned a higher offset
- Consistency guarantee:
  - The region in which a successful record append operations is defined
- Advantages:
  - No synchronization (e.g. distributed lock manager) is needed

#### Snapshot

- Use copy on write
- When snapshot, 
  - Need to revoke all leases or wait for them to expire. Ensure subsequent writes need to contact master for new lease holder
  - Log the snapshot operation to disk
  - Apply to in-memory state by duplicating the metadata for the source file
  - The newly created snapshot files reference to the original chunks
- When client first writes,
  - master picks a new chunk and asks all replicas to replicate locally, change pointer to the new chunk, then write

#### <u>Master Operations</u>

#### Namespace Management and Locking

- Allow multiple operations to be active
- Use locks over regions of the namespace to ensure serialization
- GFS represents its namespace as a lookup table mapping full pathnames to metadata
- With prefix compression, can fit in memory
- Each node in the namespace tree has a lock
  - Acquire read lock on directory if modifying files in it
  - Acquire write/read lock on files as needed
- Locks are allocated lazily (create lock when acquired) to reduce storage overhead

#### Replica Placement

- Multi-level distribution: machine level, rack level
  - maximize data reliability and availability
  - maximize network bandwidth utilization

#### Creation, Re-replication, Rebalancing

<u>How to choose a chunkserver to add replica</u>

1. A chunkserver that has below-average disk space utilization
2. A chunkserver that has a limited number of "recent" creations
3. Replicate across racks

#### Garbage Collection

- Lazy garbage collection:
  - when a file is deleted, the master logs the deletion immediately, but the file is just renamed to a hidden name without actually deleting it
  - During master's regular scan of  the file system namespace, it removes any hidden files, and its in-memory metadata is erased, so severs the links to its chunks
  - During a similar scan of the chunk namespace, the master identifies orphaned chunks and erase metadata for those chunks
  - During HeartBeat messages, master and chunkserver exchange chunk information and chunkserver can now delete such chunks
- Advantages of lazy over eager:
  1. simple and reliable in a large-scale distributed system where component failures are normal: provides a uniform and dependable way to clean up any replicas now known to be useful
  2. Merges storage reclamation into regular background activities: done in batches, cost is amortized, done when master is free
  3. delay in deleting provides the option to undo accidental, irreversible deletion
- Disadvantage and mitigation:
  1. Hinder users from actively manage storage space when it is limited
     - provide option to expedite deleting such files

#### Stale Replica Detection

- The master tracks chunk version number to distinguish up-to-date and stale replicas
- Whenever the master grants a new lease, it increments version number and notify all replicas
  - unavailable replicas become stale now
- Master detects stale replicas when chunkservers report its chunks and version numbers
- Master removes stale replicas during garbage collection, and before that, takes it as non-existing
- Master sends client/chunkserver chunk version number to verify up-to-date chunks

#### Fault Tolerance and Diagnosis

<u>Fast Recovery</u>

- Master and chunkservers restart in seconds
- Does not distinguish normal or abnormal termination

<u>Chunk Replication</u>

- Chunk is replicated on multiple chunkservers across racks
- May use parity or erasure codes for cross-server redundancy
  - good for append and read workload (instead of small random writes)

<u>Master Replication</u>

- Master state is replicated for reliability
  - Including operation log and checkpoints

#### Data Integrity

- Use checksumming to detect corruption and use replicated chunks for correction
- Each chunkserver needs to independently verify its own copy because it is not practical to compare with other replicas
- Checksum is stored like a log (persistent)
- Checksumming has little effect on read performance, only a small amount of extra data for verification
  - no I/O required
- Checksum is optimized for append: incrementally update the checksum for the last partial checksum block
  - detect corruption at reads
- Chunkserver verifies inactive chunks during idle periods

#### Diagnostic Tools

- Diagnostic logging: log major events and all RPC requests and replies
- Minimal cost: sequential and asynchronous

#### Questions

1. What problem does this paper address?
   - How to design a storage system with mostly append workload, large files, and failures are normal?
2. What is the author's main insight?
   - A single master and multiple chunkservers
   - Add atomic record append operation
   - Large chunk size
   - Use lease for consistency
3. What are the paper's key strengths?
   - Optimized for given Google workload
   - simple structure: highly scalable, fault tolerant
4. What are the paper's key weaknesses?
   - All metadata in memory still possible nowadays?
   - Master single point of failure?

## 143 - Bigtable

#### Background

- Bigtable is a distributed storage system that is used to manage structured data
  - reliably scale to petabytes of data
  - thousands of machine
- Goals:
  - wide applicability, scalability, high performance, high availability
- Support a variety of workloads: throughput-oriented batch-processing jobs or latency-sensitive serving of data
- Provides clients with a simple data model that supports dynamic control over data layout and format
  - clients can control locality of their data

#### Data Model

- Bigtable is a sparse, distributed, persistent multi-dimensional sorted map
- The map is indexed by a row key, a column key, and a timestamp
- Each value is an uninterpreted array of bytes

<img src="/Users/hanminglu/Library/Application Support/typora-user-images/image-20210728172419027.png" alt="image-20210728172419027" style="zoom:50%;" />

<u>Row</u>

- Row keys are arbitrary strings
- Every read or write of data under a single row key is atomic (regardless of columns)
- Maintain data in lexicographic order by row key
- Tablet: row range
  - row range is dynamically partitioned, each is called a tablet
  - Tablet is the unit of load balancing and distribution
  - Reads of short row ranges are efficient and require less communications
- Clients can select their row keys to store data with locality together

<u>Column</u>

- Column family: column keys are grouped into sets called column families, which is the unit of access control
- All data stored in the same column family is usually the same type
- Intention is to keep the number of column families small; the number of columns can be large

<u>Timestamp</u>

- Each cell in the Bigtable can contain multiple versions of the same data
- Versions are indexed by timestamp
- Old cell versions can be garbage collected as desired

#### API

- API provides functions for creating, modifying, and deleting tables and column families
- Can change cluster, table, and column family metadata
- Operations are on individual row keys

#### Building Blocks

- Use GFS to store log and data files (SSTable files)
  - SSTable provides a consistent, immutable, ordered map from keys to values
- Use scheduler to schedule jobs involved in BigTable serving
- Use Lock service for master election, location bootstrapping
- MapReduce: used to read/write BigTable data

#### Master, Tablet Server, Client

- One master server, many tablet servers. Tablet servers can be dynamically added or removed

<u>Master</u>

- Master is responsible for:
  - assigning tablets to tablet servers
  - detecting the addition and expiration of tablet servers
  - balancing tablet server load
  - garbage collection of files in GFS
  - handle schema changes (e.g. table and column family operations)

<u>Tablet Server</u>

- Tablet server is responsible for:
  - manage a set of tablets (row ranges)
  - Handle read and write requests to the tablet it is responsible for
  - Split tablet if it is too large

<u>Client</u>

- Client talks directly to tablet server for data
- Use chubby to find tablet location

#### Architecture Design

<u>How to get Tablet Location</u>

- Three-level hierarchy to store tablet location information
  - Chubby file
  - Root tablet
  - Metadata tablet
- Steps:
  1. The client contacts Chubby for the root tablet location
  2. Contacts root tablet server too get location of all METADATA tablets
  3. Contact METADATA tablet server to get location of its user tablet
- The client caches METADATA tablet locations to amortize communication cost

<u>Tablet Assignment</u>

- Persistent state of a tablet is stored in GFS (instead of on tablet server)
  - Tablet server is responsible for serving requests
  - Modify persistent state in GFS if needed
  
- Each tablet is assigned to one tablet server at a time
  - master keeps track of the set of live tablet servers and their assigned tablets
  
- Master uses Chubby to keep track of tablet servers:

  - each tablet server acquires a uniquely-named file in a Chubby directory in order to serve
  - When a tablet server stops to server, it releases the lock so the master can reassign its tablets
  - When a tablet server loses the lock due to failure, it attempts to reacquire the lock and restart; if the file is deleted, the tablet server can no longer serve and will terminate

- The existing sets of tablet changes if:

  1. table created or deleted
  2. multiple tablets merged into one
  3. one tablet split into two

  - Master initiates 1 & 2; for 3, tablet server notifies master

<u>Master Assignment</u>

- When master starts, it does:
  1. grab a unique master lock in Chubby
  2. Scans server directory in Chubby to find active tablet servers
  3. Ask tablet servers for their assigned tablets
  4. Master scans the METADATA table to learn the set of tablets. Add to unassigned tablets if found
- Master kills itself if its Chubby session expires

#### Tablet Serving

- Persistent state of tablets is stored in GFS
- Updates are committed to a commit log that stores redo records, among updates:
  1. Recently updates are in memory in a sorted buffer called memtable
  2. older updates are stored in a sequence of SSTables
- Client write operation:
  1. server checks write request is well-formed and authorized
  2. write to commit log
  3. insert to memtable
- Client read operation:
  1. server checks well-formed and authorized
  2. merge SSTables and the memtable (efficient since both are sorted)
  3. read on the merged view

<u>Tablet Recovery</u>

- Get the list of SSTables that comprise a tablet from METADATA table
- Get a set of redo pointers into any commit logs that may contain data for the tablet
- Reconstruct tablet by reading all SSTables into memtable and then apply all commits

#### Compactions

<u>Minor compaction:</u>

- when memtable size reaches a threshold, convert it into an SSTable, write to GFS, initiate a new memtable
- Two goals:
  1. Shrink memorage usage of the tablet server
  2. during tablet recovery, reduce the amount of data read from commit log

<u>Merging Compaction</u>

- Reads the contents of a few SSTables and the memtable, merge into a new SSTable
- Bound the number of SSTables 

<u>Major Compaction</u>

- Rewrites all SSTables into exactly one SSTable
- Some deleted data may still be there by overlapping it with a delete operation
- use major compaction to reclaim resources used by deleted data
- happen periodically in the background

#### Refinements/Optimizations

<u>Locality Group</u>

- Group multiple family columns together into a locality group
- In each tablet, a separate SSTable is used to store values in a locality group
- More efficient reads, don't read unnecessary info

<u>Compression</u>

- Client has the option to compress their SSTables (per-block basis) for a locality group
- Client can choose format
- Very good compression ratio since:
  - All pages from one host are stored close to each other
  - Locality group has similar data
  - multiple versions of the same value

<u>Caching for read performance</u>

1. Scan cache that caches key-value pairs: temporal locality
2. Block cache that cache SSTable blocks: spatial locality

<u>Bloom Filters</u>

- For read:
  - ask whether an SSTable might contain any data for a specific row/column pair

<u>Commit Log</u>

- Use one commit log per tablet server (Instead of one commit log per tablet)
- More complicated recovery:
  - Sort log by tablet, then each new tablet server needs to do one contiguous read

#### Questions

1. What problem does this paper address?
   - How to design a general-purpose data-center distributed storage system for structured data that is highly scalable, high-performance, and flexible to many applications?
2. What is the author's main insight?
   - distributed multi-level mapping
   - Use building blocks wisely
   - Fully utilize client-specifiable locality
3. What are the paper's key strengths?
   - General-purpose, highly scalable, high-performance, highly available
4. What are the paper's key weaknesses?
   - Need clients to be educated to utilize locality features

## 144 - MapReduce

#### Programming Model

<u>Map</u>

- Input: a set of input key/value pairs
- Output: a set of intermediate key/value pairs
- MapReduce groups together all intermediate values with same intermediate key, pass them to Reduce
- Map invocations are distributed across multiple machines by partitioning input data into a set of splits; Each split can be processed in parallel on different machines

<u>Reduce</u>

- Input: an intermediate key and a set of values for the key
- Output: merge these values to produce a smaller set of values
- Intermediate values are supplied to Reduce with an iterator
- Distributed by partitioning the intermediate key space into multiple partitions

#### Execution Overview

1. MapReduce library splits input file into M splits. Start programs on each machine in the cluster
2. One program is the master, all others are workers. Master assigns map and reduce jobs to each worker.
3. Map worker reads assigned input split, parse key/value pairs and send to Map function. Output intermediate kv pairs to memory.
4. Buffered pairs are partitioned by intermediate keys. Stored in local disk. Send disk location to master. Master sends location to reduce workers.
5. Reduce worker learns the locations, use RPC to transfer intermediate data from map worker's local disk.
6. Reduce worker sorts intermediate keys, group values with the same key together
7. Reduce worker pass each unique intermediate key and its values to Reduce function. Append output to final output file.
8. After all map and reduce jobs are completed, the master wakes up the program.

#### Master Data Structure

- For each map and reduce job, the master keeps its state and its assigned worker
- The master stores the location of map worker's local disk, for each map job

#### Fault Tolerance

<u>Worker Failure</u>

- Master periodically ping worker to ensure they are alive
- If some worker is dead, 
  - reschedule both completed and in-progress map jobs (since result lost)
  - reschedule only in-progress reduce jobs
- Reduce workers are notified if any map job is re-executed

<u>Master Failure</u>

- Periodically store master's state in checkpoints
- Restart from checkpoint

<u>Deterministic vs. Non-Deterministic</u>

- If deterministic map and reduce functions, then the output is the same as a non-faulting sequential execution
- If non-deterministic, the second reduce function may produce a different result because its intermediate input may be different (due to failed map tasks)

#### Locality

- GFS stores files on different machines
- MapReduce schedule map jobs considering its input file's location
  - exploit locality of input files and map workers

#### Map and Reduce Task Granularity

- Choose larger M and R than the number of machines in the cluster
- Run many different tasks improve load balancing and recovery speed

#### Straggler Handling

- There are cases where a specific machine is unusually slow
- The master schedules backup executions for in-progress tasks

#### Questions

1. What problem does this paper address?
   - A distributed system for large-scale, data-intensive, parallel data processing on community clusters, simple abstraction, handle parallel computing, job scheduling, fault tolerance.
2. What is the author's main insight?
   - A simple map and reduce abstraction
   - Can utilize machines on a cluster to do parallel jobs
   - A centralized master design
   - fault tolerance on workers
3. What are the paper's key strengths?
   - Easy to use
   - Can distribute parallel jobs to machines
   - fault tolerance on workers
4. What are the paper's key weaknesses?
   - Failure handling for master is not discussed

## 145 - Spark

#### Limitation of MapReduce

- MapReduce does not support one workload well:
  - Reuse a working set of data across multiple parallel operations
  - e.g. iterative ML algorithms, interactive data analysis tools

#### Key Idea behind Spark

- Cache repeatedly used working dataset to reduce the overhead of fetching from disk everytime

#### Programming Model

- Developers 
  1. write a driver program that implements the high-level control flow of their application
  2. launch various operations in parallel

<u>Resilient Distributed Dataset (RDD)</u>

- A resilient distributed dataset is a read-only collection of objects partitioned across a set of machines, that can be rebuilt (from its parent) if a partition is lost 
- The elements of an RDD does not need to exist in persistent storage
  - a handle to an RDD contains enough information to compute it starting from data in reliable storage
- Datasets will be stored as a chain of objects capturing the lineage of each RDD, each object contains a pointer to its parent and information about how the parent was transformed
- RDD can always be reconstructed if nodes fail:
  - its partitions are re-read from their parent datasets and eventually cached on other nodes

- To construct a RDD:
  - from a file in a shared file system
  - Divide an array of files
  - Transform an existing RDD with functions
  - Change the persistence of an existing RDD between cache and save modes

<u>Parallel Operations</u>

- Reduce: combine dataset elements
- Collect: send all elements to the master
- foreach: iterate through each element with a function

#### Questions

1. What problem does this paper address?
   - How to develop a large-scale data-intensive parallel distributed system that supports a fast reusing working set?
2. What is the author's main insight?
   - Use caching to store dataset in memory instead of disk
   - Use lineage & parent information to reconstruct lost in-memory data from persistent storage
3. What are the paper's key strengths?
   - Utilize memory to reduce the cost of reading from disk again and again when reusing a working set
4. What are the paper's key weaknesses?
   - More practical for batch workloads instead of interactive workloads

## 146 - MPI, RPC, and DSM as Communication Paradigms for Distributed Systems

#### Background

- Ease to use: DSM < RPC < MPI
- Complexity added to OS: MPI < RPC < DSM

#### Message Passing

- Message passing is the basis of most interprocess communication in distributed systems
- MP is at the lowest level of abstraction
- Requires application programmer to be able to identify expected destination process, source process, message, and data types

<u>Syntax</u>

- Simplest primitives:
  - send(receiver, message, sender(optional))
  - receive(sender, message)
- send requires to know the receiver process and message data
- receive requires to know the expected sender and provide a storage buffer for the message

<u>Blocking vs. Non-blocking</u>

- Blocking is easier for programming and debugging, but CPU is idle when blocked. More often chosen.
- Non-blocking allows concurrent execution when sending, but need interrupt to be informed when message is sent and message buffer is cleared

<u>Buffered vs. Unbuffered Messages</u>

- Unbuffered: send directly to the receiving process. Problem when send() is called before receive() because the address in send does not refer to an existing process.
- Buffered: saved in a buffer until a process is ready to receive them. Messages are queued in a buffer waiting until requested by the receiver.

<u>Reliable vs. Unreliable send</u>

- Unreliable: after sending the message, does not expect acknowledge nor retransmission if message is lost
- Reliable: wait for acknowledgement and will retransmit if message lost.
- Lost message handled either:
  1. OS retransmission
  2. OS notifies sender
  3. Sender detects it itself

<u>Direct vs. Indirect</u>

- Indirect: messages are sent to a port, then receiver receives it from the port
- Direct: message sent direct to the process itself, named explicitly in the send

<u>Fixed vs. Variable Message Size</u>

- A tradeoff between implementation difficulty and programming complexity

<u>Passing data by value/reference</u>

- Mostly pass by value because the processes execute in separate address space

<u>MP Difficulty</u>

- Application programmer needs to control data movement at the granularity of processes, control the synchronization

#### Remote Procedure Call

- MP leaves the application programmer with the burden of explicit control of data movement
- RPC increases the level of abstraction and provides semantics similar to a local procedure call

<u>Syntax</u>

- Syntax of RPC
  - call procedure_name(value_arguments; result_arguments)
  - receive procedure_name(in value_parameters; out result_parameters)
  - reply(caller, result_parameters)
- The client process blocks at the call() until the reply is received
- The remote procedure is the server processes which has already begun execution on a remote machine. Receive() blocks until it receives a message from the sender. The server reply() after finishing the task

<u>Semantics</u>

- Semantics of RPC is the same as a local procedure call: the calling process calls and passes arguments to the procedure, and it blocks while the procedure executes. When the procedure completes, it return results to the calling process.
- For instance, 
  - The execution of the call() generates a client stub which marshals the arguments into a message and sends the message to the server machine
  - On the server machine, the server is blocked waiting for message. After receiving the message, a server stub is generated and extracts the parameters from the message, and pass to the procedure

<u>Binding</u>

- binding is needed to provide a connection between the name used by the caller and the location of the remote procedure
- Implemented by:
  - using OS, storing a static or dynamic linker between the procedure name and its location on another machine
  - suing procedure variables, which links to the procedure location

<u>Communication Transparency</u>

- The user should be unaware of using a remote procedure call (instead of local)
- Three difficulties:
  1. Failure detection and correction due to communication and site failures: can result in inconsistent data because of partially completed processes, leave to programmer to deal with
  2. Parameter passing: only pass value parameters
  3. Exception handling: use available exceptions

<u>Concurrency</u>

- Since call and receive are blocking, single-threaded machines can cause significant delays
- Use mechanisms to execute calls concurrently

<u>Heterogeneity</u>

- Use a static interface declaration of remote procedures to allow communication on different OS or language

<u>Summary</u>

- RPC abstracts away communication and transmission (compared to MP)
- True transparency is hard and unsolved

#### Distributed Shared Memory

- Distributed Shared Memory is memory which, although distributed across machines over a network, gives the appearance of being centralized
- The memory is accessed through virtual addresses, so processes are able to communicate by directly writing and reading data which are directly addressable
- DSM relieves application programmers from concerns of message passing
- Higher complexity for OS as it still needs to send messages between machines to read locally unable memory, and maintain replicated memory consistency 

<u>Syntax</u>

- Same as a normal centralized memory multiprocessor:
  - read(shared_variable)
  - write(data, shared_variable)
- The OS locates the variable through its virtual address

<u>Structure and Granularity of the Shared Memory</u>

- The memory can be in the form of:
  1. an unstructured linear array of bytes
  2. structured forms of objects
- Fine or Coarse grained:
  - data should be shared at bit, word, or page level
  - coarse-grained: page-based distributed memory, where paging takes place over the network instead of to disk. Offer sequential consistency at the cost of performance.
  - fine-grained: higher network traffic

<u>Consistency</u>

- If one copy, then a request for a non-local piece of data results in a trap, causes OS to fetch it remotely. If a piece of data is requested by multiple machines, thrashing happens.
- If multiple copies, consistency among replicated data becomes the concern:
  - Consistency model determines the condition under which memory updates are propagated
  - Cannot just use cache coherence protocol because its strict consistency models cause too much network traffic
  - Stricter models result in more network traffic, thus worse performance
  - Weaker models result in less network traffic, thus better performance, but makes the programming model more complicated. Weaker consistency is a concern of OS designers.

<u>Synchronization</u>

- Shared data must be protected by synchronization primitives
- Three methods:
  - Managed by a synchronization manager
  - Responsibility of the programmer
  - Responsibility of the system developer (implicit at application level)

<u>Scalability</u>

- Scales better than tightly-coupled shared memory multiprocessors
- Limited by physical bottlenecks (e.g. buses)

<u>Heterogeneity</u>

- Hard to accommodate different machines, languages, or OS at the page level

![image-20210802125128246](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210802125128246.png)

## 147 - Ray

#### Requirements

1. Fine-grained, heterogeneous computations
   - milliseconds to hours
   - heterogeneous hardware
2. Flexible computation model
   - Both stateless and stateful computations
3. Dynamic execution
   - the order of computation finish is not known in advance
   - next executions are determined by the execution

#### Programming Model

<u>Task</u>

- Stateless and side-effect free
- Operate on immutable objects
- outputs are determined solely by their inputs

<u>Actor</u>

- Stateful execution
- Operate on mutable objects
- Given a handle to access the actor in the future

![image-20210802135703369](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210802135703369.png)

#### Computation Model

- Dynamic task graph computation model
  - execution of both remote functions and actor methods are triggered by the system when their inputs are available
  - computation graph is constrcuted from a user program
- Two types of nodes:
  - data objects
  - remote function invocations (tasks)
- Three types of edges
  - data edges: data dependencies
  - control edges: computation dependencies
  - stateful edges: connect stateful execution on the same actor

#### Architecture

<u>Application Layer</u>

- A driver
- Stateless workers: execute stateless tasks by the driver or other workers
- Stateful actor: execute on the method it exposes

<u>Global Control Store</u>

- Maintains the entire control state of the system
  - lineage information
- A key-value store with pub-sub functionality
- Sharding to achieve scale and per-shard chain replication
- Store object metadata in the GCS rather than in the scheduler, decoupling task dispatch and task scheduling
  - because involving the scheduler in each object transfer is prohibitively expensive

<u>Bottom-Up Distributed Scheduler</u>

- Two-level hierarchical scheduler
  - global scheduler
  - local scheduler
- Each node's local tasks are tried to schedule locally first
- Only send to global scheduler if:
  - local is overloaded
  - local resource is insufficient to schedule the task

- Global scheduler determines which node using:
  - estimated queue time
  - estimated input transfer time

<u>In-Memory Distributed Object Store</u>

- Store the inputs and outputs of every task
- Implement object store with shared memory
  - allow zero-copy data sharing between tasks running on the same node
  - Immutable data only
- non-local inputs are replicated to the local object store before execution
- Outputs are written to the local object store
- High throughput due to in-memory operations

#### Questions

1. What problem does this paper address?
   - A highly-scalable computation framework that handles heterogeneous requirements in a) stateful vs. stateless; b) short latency-sensitive vs. batch computation; c) 
2. What is the author's main insight?
   - A GCS storing lineage information, data object location
   - A bottom-up scheduler
   - An in-memory distributed object store
   - lineage-based fault tolerance with replication-based GCS
3. What are the paper's key strengths?
   - A general-purpose RL framework that supports training, simulation, etc.
   - Satisfy both task parallel and actor stateful execution
   - Highly scalable with sharded GCS, distributed in-memory object store, hierarchical scheduling
4. What are the paper's key weaknesses?
   - Unable to use application-optimized scheduler

## 148 - A Comparison of Approaches to Large-Scale Data Analysis

#### Background

- MapReduce and parallel database systems can serve the same set of computation requirement through different approaches
- What are the differences between MR and parallel database approaches?

#### Overview

<u>MapReduce</u>

- Use a central scheduler to schedule Map and Reduce tasks to nodes
- Run M Map tasks and R Reduce tasks
- A Map task takes a partition (a set of key/value pairs) of input file and execute a user-defined Map function, produces a set of intermediate key/value pairs, and split them into R disjoint buckets, stored in local disk storage
- Each Reduce task takes its part of bucket from all M Map tasks, apply a user-defined Reduce function that combines intermediate outputs based on key, output a part of final output, stored in global storage

<u>Parallel DBMSs</u>

- Standard relational tables (transparent physical location)
- Data are partitioned over cluster nodes
- SQL
- Join processing: T1 joins T2
  1. if T2 is large, then hash partition T1 and T2
  2. send partitions to different machines
  3. (similar to split-copy in MapReduce)
- Query Optimization
- Intermediate tables not materialized by default

#### Architecture Differences

![image-20210803142937626](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210803142937626.png)

<u>Schema Support</u>

- MapReduce: 
  - More flexible, good for one application, bad for multiple applications sharing one data set
- Parallel DBMS
  - Relational schema required, good for multiple applications sharing

<u>Programming Model & Flexibility</u>

- MapReduce:
  - low level
  - very flexible
- Parallel DBMS:
  - SQL
  - user-defined functions, stored procedures, user-defined aggregates

<u>Indexing</u>

- MapReduce:
  - No native index support, can implement their own but hard to share
- Parallel DBMS:
  - Hash/b-tree indexing well supported

<u>Execution Strategy & Fault Tolerance</u>

- MapReduce:
  - Intermediate results are saved to local files
  - If a node fails, run the node-task again on another node
  - Large number of disk seeks at mapper machine during intermediate result transfer
- Parallel DBMS:
  - Intermediate results are pushed across network
  - If a node fails, the entire query needs to run again

<u>Avoiding Data Transfer</u>

- MapReduce:
  - locate Map close to data
- Parallel DBMS:
  - lots of optimization
  - e.g. where to perform filtering

<u>Node Scalability</u>

- MapReduce:
  - 10000's of commodify nodes
  - 10's of Petabytes of data
- Parallel DBMS:
  - <100 expensive nodes
  - Petabytes of data

<u>Setup and Configuration</u>

- MapReduce:
  - easy
- DBMS:
  - hard to configure

<u>Ease of Use</u>

- More familiar with language used by MapReduce compared to SQL

<u>Performance</u>

- DBMS is faster than MR because:
  1. Indexing: B-tree indices to speed the execution of selection operations
  2. Column store: column oriented storage
  3. Compression: aggressive compression and operate on compressed data
  4. Parallel algorithm for querying large amount of data

#### Questions

1. What problem does this paper address?
   - What's the differences between approaches of MapReduce and Parallel DBMS for large-scale data-intensive computation?
2. What is the author's main insight?
   - For the size of 100 nodes, Parallel DBMS has a better performance due to indexing, storage schema, compression, parallel algorithm.
   - MapReduce is easier to use, may perform better in a larger cluster with commodity hardware
3. What are the paper's key strengths?
4. What are the paper's key weaknesses?

## 149 - Lottery Scheduling

#### Some Scheduling Schemes

- First come first serve
- Round robin
- Priority system
  - cons: highest priority always wins, starvation risk. Priority inversion: high-priority jobs can be blocked behind low-priority jobs
- Multi-level feedback scheduling
  - cons: fairness only over very long term

#### Lottery Scheduling

- Proportional-share scheduling algorithm
- Give each job its proportional number of tickets
- On each time slice, pick a ticket and give owner the resource
- On average, resource fraction (CPU time) is proportional to number of tickets given to each job

<u>To Assign Tickets</u>

- Priority determined by the number of tickets each process has
- To avoid starvation, each job gets at least one ticket

#### Fairness of Lottery Scheduling

- Probabilistically fair
- Advantage over strict priority scheduling: 
  - behaves gracefully as load changes: adding or deleting jobs affect all jobs proportionally
- Mostly fair, but short-term unfairness is possible (see stride scheduling)

#### Ticket Transfer

- If a job depends on another job, it gives all its tickets to the other job
  - solves priority inversion problem

#### Compensation Tickets

- Compensate for resources assigned but not used (yield)
- Ticket inflate proportional to the amount of time you did not use, until next time you win

#### Questions

1. What problem does this paper address?
   - A scheduling scheme that provides fair share, avoids priority inversion, simple and easy-to-use?
2. What is the author's main insight?
   - Lottery-based scheduling, each job gets a proportional number of tickets
   - Hand over tickets to another job if you depend on it to finish
3. What are the paper's key strengths?
   - Simple
   - Solves priority inversion
   - Fairness in the big picture
4. What are the paper's key weaknesses?
   - ticket inflation
   - short-term unfairness

## 150 - Dominant Resource Fairness

#### Max-Min Fairness

1. Sort resource requirement in an increasing order
2. Give the lowest demand first, with min(Capacity/requesters, its demand)
3. Give the extra resource to other resources if any left

- Any person is worse if they lie about their demand

- Weighted Max-Min fairness: takes weights into consideration

#### Why (Weighted) Fair Sharing?

- Higher priority gives higher resource
- Share guarantee of at least C/n
- Strategy-proof
- Isolation: Users cannot affect each other

#### Dominant Resource Fairness (DRF)

- Dominant resource: the resource that a user wants the highest proportion (e.g. 1/2 is greater than 4/100)
- Different users may have different dominant resources
- DRF applies max-min fairness across users' dominant resources
- DRF maximizes the smallest dominant share in the system, then the second-smallest, and so on

<u>Steps</u>

1. DRF picks the user with the lowest dominant share(max{utilization rate}), run one of its tasks
2. Update the dominant share, consumed resources
3. Repeat

- e.g.

![image-20210803174509311](/Users/hanminglu/Library/Application Support/typora-user-images/image-20210803174509311.png)

<u>Characteristics</u>

- O(log n) time per decision, n is the number of users
- Need to determine demand vector for each task

<u>Weighted DRF</u>

- Modify dominant share calculation to max{utilization rate / weight}

#### Properties of DRF

1. Strategy-proof: no better to lie about its demand
2. Share guarantee: each user is guaranteed to have at least C/n share
3. Envy-free: no task wants to swap with another task's resources
4. Pareto Efficiency: it is not possible to give extra resource to any user without hurting at least one other user
5. Single-resource fairness: if only one resource, then max-min
6. Bottleneck fairness: if all tasks have one dominant resource, than it is shared according to max/min fairness

#### Questions

1. What problem does this paper address?
   - How to schedule heterogeneous resources to users with heterogeneous demand in a fair way?
2. What is the author's main insight?
   - Determine a user's dominant resource and schedule tasks based on it
3. What are the paper's key strengths?
   - Strategy-proof, share guarantee
4. What are the paper's key weaknesses?
   - If the number of users is too many, long scheduling time

## 151 - Stride Scheduling/Completely Fair Scheduling

#### Summary

- A deterministic version of lottery scheduling
- Every thread has a "stride" that is inversely proportional to its priority

<u>Steps:</u>

1. Initially, everyone's "pass" is equal to its "stride"
2. Every time slice, choose the thread with the lowest pass
3. After a thread is run, increment its "pass" by "stride"
4. repeat step 2

<u>Advantages over Lottery scheduling:</u>

- deterministic variance from the ideal ratio
- short-term stability

## 152 - Mach

#### Microkernel

<u>Monotholic Kernel</u>

- Running entirely in the kernel mode

<u>Microkernel</u>

- Modular design that separates OS into two parts:
  1. Control basic hardware resources (i.e. microkernel), responsible for low-level process management, handle message passing, interrupt handling
  2. Control unique characteristics of the environment for applications, such as file system, memory management.

- Advantages:
  1. Portability: microkernel as a narrow waist that provides hardware independence for other parts of OS and application
  2. Extensibility & customization: environments can evolve independent of microkernel
  3. Functionality & performance for kernel: simpler functionality so easier to parallelize, distribute, and secure
  4. Flexibility: system environment can run remotely
  5. Real-time: kernel does not need to hold long locks & OS environment is preemptable, can provide real-time support
  6. Fault Isolation
  7. Security
  8. More reliable (less code in kernel)

#### Mach Kernel Features

<u>Process and thread management</u>

- Resources are allocated to processes, which consists of an address space and communication access to system
- Thread performs computation and consumes its process' resources
- Threads are scheduled by the Mach kernel
- User space programs can control scheduling policy

<u>Interprocess Communication</u>

- Interprocess communication is implemented exclusively through ports (similar to a handle)
- Processes, threads, memory objects, and Mach process services are all manipulated by sending messages to ports that represent those objects
- The port abstraction can be accessed through network

<u>Memory Object Management</u>

- Memory objects can represent anything that makes sense to access through memory references (e.g. I/O devices, disk I/O)
- Processes use ports to interact with memory objects
- Process address space is a collection of mappings from linear addresses to offsets within memory objects

<u>System Call Redirection</u>

- Mach kernel allow redirecting certain system calls or traps back to the calling process to handle
  - Require one kernel entry and one kernel exit
- Emulation library is inherited by child processes
- Exceptions can also be redirected

<u>Device Support</u>

- Each device is represented as a port, can talk to it from threads

<u>User Multiprocessing Support</u>

- Allow user-level OS/applications to use multithreading without involving kernel

<u>Multicomputer Support</u>

- All communication happens through IPC (via port), which can be done through networks

#### Implementing OSes on Mach

- Key idea: treat OSes like application programs
- Three approaches:
  1. Native OS systems
  2. Large granularity server systems
  3. Small granularity server systems

<u>Native OS systems</u>

- key idea: Put everything that a kernel would handle into an emulation library, where a system call will be redirected to
- Allows most of original system calls to execute without change
- Emulation library responsible for system calls and virtualizing access to hardware devices
- Similar performance to other UNIX implementations

<u>Large Granularity Server Systems</u>

- Key idea: concentrate most non-native OS's functionality in a single server
- Emulation library redirects again to the server, which handles system calls and memory management
- Requires porting but most code can be reused

<u>Small Granularity Server Systems</u>

- Key idea: One server (module) per non-native OS's functionality
- Each service is implemented in a separate module

- Key advantage:
  - decomposition means isolation
- One server can service multiple client OSes

<u>Performance</u>

- Native OS, large granularity, and UNIX have similar performance
- The amount of IPC is too high for small granularity system, so lower performance

#### Questions

1. What problem does this paper address?
   - How to implement OS using a modular design to improve portability, evolvability, security, etc.?
2. What is the author's main insight?
   - Separate OS into two parts: only the necessary hardware management part runs in the kernel, other OS functionality in the user-level
   - Kernel manages CPU, IPC, virtual memory, device I/O
   - Use system call redirection to allow user-level processes to execute system calls
3. What are the paper's key strengths?
   - Microkernel allows better portability, evolvability, security, flexibility
4. What are the paper's key weaknesses?
   - Performance metrics are missing

## 153 - seL4

#### Key Idea

- microkernel significantly reduces the amount of kernel code, making a formal verification of correctness possible.
- The assumptions are hardware, compiler, assembly code, cache management, boot sequence.
- It shows that formally verifying a system is possible
- It is very time consuming (years of time) to formally verify even microkernel
- No clear mentions of what correctness guarantees are achieved, only mentioned ~80 invariants

## 154 - SPIN

#### SPIN Key Idea

- Extend the kernel (called spindle) at runtime through statically checked extensions
- Safety ensured using programming language
- Event/handler abstraction
  - spindle listen to and handle events (e.g. spindle can listen and react to page faults and error)

<u>Spindle Capabilities</u>

- Run code in the kernel in general
- Listen to and handle events
- Fine-grain hardware resource manipulation
- Define new syscalls

#### Questions

1. What problem does this paper address?
   - How to modify OS to provide extended functionality in a minor way?
2. What is the author's main insight?
   - Dynamically load extensions to the kernel, which is statically checked
   - Partition code across the user/kernel boundary to avoid extra IPC and switching
   - Use special programming language to ensure safety
   - Use an event/handler abstraction for the spindle
3. What are the paper's key strengths?
   - Minor modification to OSes (except PL requirement)
   - extend functionality of existing OSes, while maintaining safety?
4. What are the paper's key weaknesses?
   - require special PL
   - Safety cannot be secured by extensions
   - Extensions should run in user space instead

## 155 - Exokernel

#### Abstractions

- Abstraction is generalization that is often an API in CS, hides implementation details
- Advantages of abstractions:
  1. Simpler. Easy to understand and use
  2. Standardization. Many implementations all satisfy the abstraction
- Disadvantages of abstractions:
  1. It is a compromise, the least common denominator. not perfect for each use case
  2. Low performance (compared to tweaking the software for each use case)
  3. Possible bloated software

#### Problems in existing OSes (back then)

- Low extensibility:
  - abstractions are overly general
  - apps cannot control resource management
  - Implementations are fixed
- Low performance:
  - context switching is expensive
  - Generalization and hiding information affect performance

#### Exokernel Main Ideas

- Separate resource management and protection
  - kernel: resource sharing, protect hardware, protect LibOSes from each other
  - untrusted LibOS: implement traditional OS abstractions, manage resources
- Advantages:
  - Efficient because LibOS is in user space
  - Apps can link to their choice of LibOS for better performance
  - Minimalist kernel design

#### Design Principles

1. Securely expose hardware
   - Exokernel should only manage resources to the extent required by protection (i.e. allocation management, revocation, and ownership)
2. Expose allocation
   - LibOS should be able to request specific physical resources
   - LibOS should participate in every allocation decision
3. Expose names
   - export physical names such as physical page number
   - export bookkeeping data structures such as TLB entries
   - avoids layer of indirection, better performance
4. Expose revocation
   - Apps should be notified when their resource is being taken away
   - Apps can determine which resource to relinquish

5. Exokernel still arbitrate resource contention with its own policy

#### Mechanisms

<u>Secure binding</u>

- Goal: separate authorization and actual use of the resource
  - Only perform expensive authorization at bind time
  - Future accesses only need simple operations for protection checks
- e.g. TLB fault -> LibOS maps virtual address to physical address and load into kernel (expensive authorization here) -> application accesses only need simple access checks

<u>Visible Resource Revocation</u>

- Revocation is like a dialogue where Exokernel revokes some page and LibOS saves the page to disk and frees it
- Higher latency

<u>Abort Protocol</u>

- Forcefully break LibOS's all secure bindings if it fails to respond
  - Inform the LibOS afterwards

#### Questions

1. What problem does this paper address?
   - How to design an OS that provides minimal kernel design and allow maximum freedom to applications for a better performance?
2. What is the author's main insight?
   - Separate protection and resource management
   - Kernel does protection only, and provides maximum freedom to untrusted LibOS which does resource management instead
3. What are the paper's key strengths?
   - Modularity
   - applications can choose LibOS that satisfies its choice the most
4. What are the paper's key weaknesses?
   - Cost of development and portability is high

## 156 - HiStar

#### OS Security Problems today

- OS today have protection
  - file system with RBAC
  - process protection
  - memory protection
- Problem now?
  - Ignoring information flow
  - Process P can read a secret file and write it to public space

#### Information Flow Control (IFC)

<u>Main Idea</u>

- Files and processes colored
  - Label private stuff RED
  - Label public stuff GREEN

- Enforce communications allowed/disallowed

<u>Six Kernel Objects</u>

1. Segment (data), array of bytes
2. Threads
3. Address space
4. Device (Network)
5. Gate (IPC)
6. Container (directory)





## 160 - Meltdown

#### Summary

- Exploit side effects of out of order attacks in order to get private data

## 161 - Spectre Attack

#### High-level Attack

1. Attacker uses micro-architecture
   - e.g. branch predictor or branch target buffer for saving secret
   - e.g. cache for recalling secrete
2. Victim loads secret under mis-speculation
   - Load should NOT trap
   - Still inappropriate if managed language or sandbox
3. Victim saves secret in micro-arch state, e.g. cache
4. Attacker recalls secret from micro-arch state

#### Applicability

- Many existing designs are vulnerable
- Can let Javascript steal from Chrome

#### Spectre Mitigation

- Software: add hardware support to disable branch prediction when important
  - Performance cost

## 189 - Delay Scheduling

#### Key Idea

- When the scheduler is evaluating which node to execute the head-of-line job, it evaluates if current available nodes have the input data required to execute the job
  1. If yes, schedule on that node
  2. If no, delay scheduling this node and evaluate other jobs
     1. If the job is skipped enough times, scheduler will relax the requirement to rack-level input data, eventually becomes global-level input data
- The tradeoff here is fairness and data locality
- This improves overall performance when the workload contains a lot of small jobs (so data locality nodes are available soon)



## Questions to Answer

#### Questions

1. What problem does this paper address?
2. What is the author's main insight?
3. What are the paper's key strengths?
4. What are the paper's key weaknesses?

