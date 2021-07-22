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

==TODO==



## 140 - Tapastry

#### Background

- 





## Questions to Answer

#### Questions

1. What problem does this paper address?
2. What is the author's main insight?
3. What are the paper's key strengths?
4. What are the paper's key weaknesses?

