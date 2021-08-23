## 01 - OS Structure

#### Practice Questions:

1. Compare segmentation and paging. How about multi-level paging?
2. (Done) What are RAID 0, 1, 4, 5, 6? What's their read/write performances? Why are they performing like this?
3. (Done) Compare Monolithic kernel, SPIN, and Exokernel in performance, extensibility, safety, portability.
5. Why there is no locking in UNIX? Is it too simple?
6. (Done) Compare thread-based concurrency and event-based concurrency.
7. (Done) In a multicore system, compare shared memory and message passing.

#### Fundamental Tradeoffs

1. **Simplicity vs. functionality**: UNIX
2. **Performance, extensibility (, safety)**: monolithic vs. microkernel vs. SPIN
3. **Programmability vs. scalability**: thread-based vs. event-based concurrency control
4. **Scalability vs. transparency**: shared memory vs. message passing

#### 101 - UNIX

- UNIX is an interactive, multi-user, file-based, simplified file/network/device interface file system that enables a user-shell to interact with the system
- Everything is a file (i.e. file, directory, network, device)
- **Pros**: simplicity is the key
- **Cons**: functionality may be missing? (e.g. locking)

#### 152 - Mach

- Microkernel, in contrast to monolithic kernel, separates OS into two parts:
  1. A **microkernel part** that implements the most essential low-level functionality such as low-level process management, virtual memory, message passing, interrupt handling
  2.  An **OS app part** that is built on top of the microkernel and implements environment for applications, such as file system, memory management
- Syscall redirection is used to allow OS app to handle syscalls, exceptions, interrupt
- All components in the system are modular, and port (message passing) is used to communicate between them, including memory object, process, thread, etc.
- **Pros**: extensibility, performance (since no general abstraction), security, reliability, flexibility (port via network)
- **Cons**: too many modules will incur high communication costs via port

#### 154 - SPIN

- SPIN thinks OS is generally good for common case, but OS should provide the capability for minor modifications for performance or drivers
  - tradeoff: performance, safety, extensibility
- SPIN provides the ability to **dynamically add external extensions** to kernel in an event-driven model
- safety is ensured using type safe language, static checking and dynamic linking checks to ensure extensions are performing safely
  - no protected instructions or memory regions are used
  - no pointer forgery or illegal dereferencing
- **Pros**: extension at kernel level, high extensibility
- **Cons**: safety may be in danger even after checks

#### 155 - Exokernel

- Exokernel is a minimalist design that separates **resource management** and **protection**
  - kernel for protection: only safely expose hardware resource to LibOS, protect LibOS from each other
  - LibOS for management: manage resources in whatever way it likes, has full knowledge of physical resources, no another layer of indirection
- **Pros**: performance (no general abstraction, app choose LibOS), high extensibility (LibOS), portability, functionality
- **Cons**: cost to implement LibOS

#### 171 - SEDA (Staged Event-Driven Architecture)

- **SEDA decomposes a service into stages**, each handle one functionality, separated by queues
  - stage: each stage handles one functionality, allows **local thread-based concurrency** to allow programmability
  - queue: each stage has a queue, which allows varied processing speed between stages, and allow batched processing
- **Pros**: good programmability, good scalability (in # of stages), combine thread-based and event-based concurrency
- **Cons**: may incur a lot of communication and queuing delay between stages

#### 173 - Linux Scalability Problems to Many Cores

- Problems of having many cores with shared resources:
  1. **Lock waiting** time: more total time waiting for locks as # of cores increases
  2. **Cache coherence waiting** time: cores write to a shared memory location, time spent to wait for cache coherence protocol
  3. **Cache miss rate**: cores use shared cache for their own use
  4. Resource contention in general

#### 174 - Multikernel

- In a **multicore**/multiprogramming system, use **message passing instead of shared memory**
  - all OS-level inter-core communications should be done in message passing, no shared memory should exist anywhere (except for implementing message passing)
  - each core should have its own replica of state, and achieve consistency through message passing
- CPU is faster than memory, so message passing is more efficient than shared memory at scale
- **Pros**: higher scalability
- **Cons**: with fewer cores, shared memory is more efficient

## 02 - File Systems & Distributed FS

#### Practice Questions:

1. (Done) Compare having small and large block sizes in a file system.
2. (Done) In Journaling file system, the paper claims "ordered journaling can achieve data and metadata consistency after crash." Why is it possible to claim that data will be consistent given that the system can crash when data is written to fixed location?
3. (Done) Compare Log-Structured FS and FFS. 
4. (Done) What is the key tradeoff in Coda's disconnected operation? Which side does Coda prioritize?
5. (Done) What is the key tradeoff in AutoRAID? Does it successfully achieve both?

#### Fundamental Tradeoffs:

1. Disk throughput vs. space overhead: internal fragmentation vs. large block size
2. Performance vs. durability vs. fault tolerance: journaling
3. Performance vs. storage cost: AutoRAID
4. Availability vs. Consistency: Coda

#### 106 - FFS

- FFS improves **data transfer size (throughput)** and **locality (latency)** by: 1) increasing **block size** to 4MB (transfer throughput); 2) using **cylinder groups** to store files in consecutive blocks (locality); 3) Keep files in the same directory together (locality).
- To mitigate internal fragmentation due to larger block size, it divides a block into segments to store small files
- **Pros**: exploits locality to improve performance and latency
- **Cons**: internal fragmentation gets worse even with segmentation

#### 107 - Journaling File System

- **Journaling**: write to log, then move the data to fixed location (checkpoint).
  - Improve crash fault tolerance
  - Improve random write performance to sequential writes
- Journaling modes:
  - **Writeback mode**: journal metadata, async write data to fixed. Committed transactions have consistent metadata, but may have inconsistent data after crash. Fast.
  - **Ordered mode**: journal metadata, sync write data to fixed before journal. Committed transactions must have consistent data and metadata. Faster than data mode if data write is large and sequential.
  - **Data mode**: journal both data and metadata. Committed transactions must have consistent data and metadata. Faster than ordered mode if writes are small and random. Double write.

- **Pros**: data mode may improve random small write performance to large sequential write. Better crash fault tolerance
- **Cons**: data mode may be costly with double write. Writeback mode can still incur inconsistent data

#### 108 - Log-Structured File System

- Log-Structured FS stores the **entire FS in a log**
  - write: fast since requires almost no seek time, small random writes are buffered together so can fully utilize disk throughput
  - **read**: every time an inode is updated, it is stored again in the log. An inode map (in RAM) is used to find inodes
  - **cleaning**: cleaning is required to have large spaces for contiguous log. use copying to combine partially full inactive segments into smaller number of segments, then skip (threading) them during logging.
- **Pros**: fast random writes, similar random read, exploits temporal locality, reasonable cleaning cost
- **Cons**: worse sequential reads, extra overhead for sequential writes

#### 109 - AutoRAID

- AutoRAID is a storage system that uses an **automatically migrating two-level hierarchy** of two RAID levels to optimize performance/cost for active data and storage/cost for inactive data
  - Use **RAID 1 for active data**: high read and write performance, redundancy of 1
  - Use **Log-Structured RAID 5 for inactive data**: slightly lower read performance, lower write performance, redundancy of 1/N
- **Pros**: Achieve performance and storage cost at the same time, easier management, easier deployment, easier disk add/remove 
- **Cons**: assumption that active data is relatively constant may not be true

#### 112 - F2FS

- F2FS is designed specifically for **SSD storage**, where it tries to fully utilize SSD's advantages through **hot/cold data logging**, alignment between segment/section/zone with SSD layout, adaptive logging based on storage utilization
- **Pros**: separates hot and cold data can make cleaning more efficient, adaptive logging makes writes more efficient
- **Cons**: designed for SSD specifically

#### 179 - Coda

- Coda enables **disconnected operation** using 1) **client-side hoarding**; 2) file-level caching; 3) **callback-based optimistic consistency control**
- When in connected operation, client hoard specified files and recently used files
- When in disconnected operation, client can continue operate on cached files, which will be updated to servers after connection is resumed
  - may violate transparency and require users to resolve conflict
- **Pros**: High availability (operate in disconnected mode), 
- **Cons**: consistency is sacrificed during disconnected operation, violate transparency and require user conflict resolution

#### 180 - Sun

- Sun is a simple **stateless distributed FS** that prioritizes:
  1. portability
  2. crash recovery
  3. transparent access

- **Pros**: easier crash recovery
- **Cons**: stateless is less performant

#### 181 - Andrew FS

- Andrew FS implements more functionality on client, in particular, **clients operate on local cache** and only incur RPC during open and close
- Use callback interface for cache coherence protocol
- Pros: only RPCs are open and close, maximize cache hit ratio
- Cons: at large scale, cache invalidation may happen too frequently, which degrades performance

#### 182 - Sprite NFS

- Sprite NFS implements different modes for sequential write sharing and concurrent write sharing:
  - **sequential write sharing**: the client cache and operate on its local cache, lazily copy over dirtied page afterwards
  - **concurrent write sharing**: no caching, operate on server
- **Pros**: if one writer, low latency as client operates locally. If multiple writers, operate on the server to ensure consistency
- **Cons**: no caching hurts performance in the concurrent case

## 03 - Distributed Systems

#### Practice Questions:

1. (Done) Compare lamport clock and vector clock
2. (Done) What are the requirements for CRDT?
3. (Done) Compare optimistic vs. pessimistic concurrency control
4. (Done) Describe PBFT in safety vs. liveness tradeoff.
5. (Done) Compare Chord and Tapestry

#### Fundamental Tradeoffs:

1. CAP - Consistency vs. Availability vs. Partition-tolerance: CRDT
2. performance vs. scalability: optimistic concurrency control
3. availability vs. performance: Chubby
4. safety vs. liveness: PBFT

#### 117 - Commutative Replicated Data Type (CRDT)

- CRDT is a kind of data that can achieve both **strong eventual consistency** and scalability/**availability**:
  - all concurrent updates are **commutative**
  - all replicas execute updates in **causal order** (can use vector clock)
- Treedoc is a CRDT structure:
  - insert and delete are commutative and idempotent
  - use mini-node for concurrent TID from different machines
  - causal order is inherit
- **Pros**: CRDT can achieve strong eventual consistency under AP
- **Cons**: CRDT is a restricted kind of data type

#### 120 - Lamport Clock & Vector Clock

<u>Lamport Clock</u>

- Lamport clock exploits the "happens-before" relationship, where
  1. one the same processor, a->b implies C(a) < C(b)
  2. between two processors, C(send) < C(receive)
- Lamport clock ensures **clock consistency condition**, that is if C(x) < C(y), then either:
  1. **a -> b**, or
  2. a and b are **concurrent**

<u>Vector Clock</u>

- Vector clock uses a list of timestamps, one per processor
- Vector clock ensures **strong clock consistency condition**, that is if C(x) < C(y), then:
  - **x -> y**

- **Pros**: both clocks use the concept of logical clock
- **Cons**: lamport clock doesn't have strong clock consistency condition

#### 121 - Efficient Optimistic Concurrency Control with Loosely Synchronized Clocks

- Use 1) **callback-based concurrency control**; and 2) **timestamp-based conflict validation** to achieve efficient optimistic concurrency control under low to moderate contention
- **Pros**: good performance for low to moderate contention level, fewer messages exchanged compared to pessimistic concurrency control
- **Cons**: if contention or the number of clients is high, too many aborts, hurt performance

#### 122/125 - Paxos

- Paxos is a consensus protocol that has **two stages**: 0) leader election; 1) **prepare**; 2) **commit**
  1. the leader sends prepare message with sequence number N to acceptors; acceptors promise they won't accept any message < N; acceptor includes its accepted value if there is any
  2. after receiving a majority of promises, the leader sends commit to all acceptors; acceptors send ack back
- Consensus is achieved when a majority of acceptors have flushed the commit message to disk
- Implementation challenges: 1) code complexity; 2) hard to prove large codebase; 3) real-world errors not covered in paper

- **Pros**: provable consensus protocol
- **Cons**: hard to understand or implement

#### 123 - Raft

- Raft is a consensus protocol that is easier to understand and educate (compared to Paxos)
- Raft has **two stages**: 0) leader election; 1) **append**; 2) **apply**
  1. Leader receives log entry from client, append (without apply) it locally, then send appendEntry to replicas
  2. Leader receives a majority of acks, apply locally, send applyEntry to replicas, wait for ack
- **Pros**: compared to Paxos, Raft is easier to educate and implement
- **Cons**: just another consensus algorithm

#### 124 - Chubby Lock Service

- Chubby is a file system interface-based coarse-grained Paxos-based **locking service** that prioritize **availability**, provides client side consistent hashing, support event notification, and store small metadata files.
- Use cases are long-held locks, such as leader election
- **Pros**: high availability due to replicated servers with Paxos
- **Cons**: restricted use case? Paxos results in longer latency? (performance vs. availability)

#### 126 - BFT

- For a cluster of machines to tolerate f possible traitors, it requires **3f+1 machines** to make sure loyal machines can agree on a value
- **Pros**: provable lower-bound for f possible malicious machines
- **Cons**: more communication overhead, higher latency

#### 127 - PBFT

- PBFT is a practical BFT protocol that allows loyal machines to achieve consensus with f malicious machines in asynchronous setup
- PBFT algorithm:
  1. **Pre-prepare**: Primary receives msg from client, assign a view and a sequence number, multicast pre-prepare to all replicas
  2. **Prepare**: Each replica receives pre-prepare, then multicast prepare msg. Each replica will receive 2f+1 (pre-)prepare msgs, so a quorum
  3. **Commit**: Each replica receives a quorum of (pre-)prepare messages, then multicast commit msg. Each replica will receive 2f+1 commit messages, so a quorum

- Assumptions: 1) f malicious replicas; 2) asynchronous; 3) eventual liveness
- **Pros**: always safe, achieve BFT consensus among f malicious replicas in an asynchronous setting
- **Cons**: only eventual liveness when DOS attack ends

#### 138 - Chord

- Chord is a simple **consistent hashing-based DHT** that:
  1. require **O(logN)** for both storage and query message overhead
  2. uses **finger table** for performance
  3. not topology-aware
  4. add/remove node with bounded message exchanges
- **Pros**: simple, load balancing, performance, availability, scalability, flexible mapping
- **Cons**: not topology-aware

#### 139 - Tapestry

- Tapestry is a DHT that uses **topology-aware** routing to minimize network delay in the routing
  - Tapestry uses **prefix matching** to route (e.g. ```4*** => 42** => 42A* => 42AD```)

- Optimizations:
  - assign ID using a random process (which considers topology)
  - Share a DHT among multiple apps to achieve economy of scale

- **Pros**: topology-aware
- **Cons**: algorithm is more complicated

#### 146 - MPI vs. RPC vs. DSM

- MPI: **pass message from process to process**, requiring info on destination process, 
  - easiest to implement, hardest to use
- RPC: higher abstraction than MP, user uses it as a local procedure call, a **client stub and a server stub** are required to marshal/unmarshal and bind published remote procedure calls
  - medium to implement, medium to use
- DSM: even higher abstraction, implement shared memory across machines such that users **see it as one large memory**
  - strong consistency among copies is hard to achieve at a reasonable cost; if only one copy, thrashing may happen if multiple cores simultaneously use it 
  - hard to implement, easy to use

## 04 - Scheduling

#### Practice Questions:

1. (Done) Describe Mesos. What are its pros and cons?
2. (Done) compare lottery vs. stride scheduling

#### Fundamental Tradeoffs:

1. centralized vs. distributed allocation: Mesos
2. fairness vs. locality: delay scheduling

#### 134 - Mesos

- Mesos provides a **thin layer of abstraction** between resources and multiple cluster computation frameworks
  - **Mesos** decides what resources to **offer**
  - **frameworks** decide if they want to **accept** the offer
- **Pros**: frameworks can choose their scheduling algorithm, adaptivity and scalability (compared to centralized scheduler), works well for elastic framework, homogeneous task duration, prefers all nodes equally 
- **Cons**: not globally optimal, works not as well for rigid framework, heterogeneous task duration, node preference

#### 149 - Lottery Scheduling

- Lottery scheduling is a **probabilistic scheduling** algorithm that **handles starvation, inverse priority, and compensation problems**.
  - Every task is given the number of tickets proportional to its priority
  - Every time slice, a ticket is chosen at random
- **Pros**: probabilistically fair in the long term, no starvation, inverse priority solved by giving tickets, provides compensation ticket, add/remove tasks affect all tasks fairly
- **Cons**: **short-term fairness** is not guaranteed

#### 150 - Dominant Resource Fairness (DRF)

- DRF applies **min-max fairness** on the **dominant resource** of tasks
- **Pros**: strategy-proof, share guarantee, etc.
- **Cons**: Long decision making time? O(logN) where N is the number of users

#### 151 - Stride Scheduling

- **Deterministic** version of **lottery scheduling**
  - every task is given a "stride" proportional to its priority
  - For every time slice, pick the task with the lowest value
- **Pros**: ensures **short-term fairness**
- **Cons**: needs global state

#### 189 - Delay Scheduling

- Delay scheduling **delays scheduling tasks** to **wait** for the **node with required data** to execute the task
- **Pros**: better data locality, works well where tasks are short (so node with data becomes available sooner)
- **Cons**: fairness compromised, wait too long or not effective if tasks are long

## 05 - Big Data

#### Practice Questions:

1. (Done) What is the single point of failure in MapReduce? How can you resolve it?
2. (Done) What is the key insight of Spark and why is it useful?
3. Compare parallel DB and MapReduce.
4. Compare container and VM
5. (Done) What makes Dynamo highly write available? What does it compromise instead?
6. (Done) Do you think GFS is scalable given a single master node? Why?

#### Fundamental Tradeoffs:

1. simplicity vs. performance: Parallel DB vs. MapReduce
2. availability vs. consistency: Dynamo
3. Safety vs. performance: Pond
4. performance vs. crash recovery: Spark

#### 111 - Haystack

- Haystack is an object/photo storage system that **store object indexing information in memory**, **log-structured storage** to increase throughput and reduce latency
  - **performance**: an internal CDN, small metadata in memory + flat namespace -> one disk seek per photo, log-structured file -> sequential write performance
  - **fault tolerance**: store a photo in multiple physical machines for redundancy
- **Pros**: one disk access per photo, fault tolerance from redundancy, an internal CDN for caching
- **Cons**: limited application to read-mostly workload?

#### 139 - Dynamo

- Dynamo is a **highly-available** key-value store that is designed **for tail latency**
  - **Partitioning**: use consistent hashing with virtual nodes to partition storage and workload with heterogeneity
  - **Write availability**: user sets R and W values, write is always available if W amount of nodes are available, R+W > N ensures a quorum
  - **Short-term fault tolerance**: hinted handoff put() to another node and wait for its coordinator to restart
  - **Long-term fault tolerance**: replication among nodes and use Merkle tree for synchronization
  - **Versioning**: each modification is new and immutable, multiple versions co-exist
  - **Conflict resolution**: done at read by users instead of the system
  - **Low latency**: every node stores the list of all coordinators 
- **Pros**: highly available, decentralized, fault tolerant, application-level conflict resolution
- **Cons**: application-level conflict resolution can be limited by data types

#### 141 - Pond

- Pond is a large-scale storage system that is built on **untrusted unless in group infrastructure** where **nodes change without notices**
  - **Routing**: tapestry is used for naming, routing messages, mapping from logical to physical location
  - **Data model**: hash pointers to read-only, copy-on-write metadata, data, previous update
  - **BFT**: inner ring determines update serialization with BFT protocol 
  - **Storage**: erasure coded and stored on multiple machines to increase durability
  - **Caching**: every read is cached (read-only) and published to Tapestry to accelerate reading popular blocks
- **Pros**: built on untrusted infrastructure, highly durable, fast read on popular blocks
- **Cons**: slow write due to BFT and erasure coding

#### 142 - Google File System

- GFS is a distributed storage system that is designed for **high append and read workload**
  - master:
    - **Workload**: metadata-related operations only, stores file and chunk namespaces, file to chunk mapping, and chunk to location mapping
    - **Concurrency**: uses lock & operation log order 
    - **Fault-tolerance**: namespaces, mapping, and operation logs are persisted and replicated
    - **Garbage collection**: deletion only mark file, periodic scanning deletes metadata, mappings, and chunks
    - **Re-balancing & chunk migration**: periodic scanning to rebalance chunk load
  - chunkserver:
    - **Workload**: data-related operations, stores chunks, large chunk size
    - **Consistency**: primary takes a lease, secondary listen to primary on ordering
    - **Atomic append**: client provides data, primary determines location. Append to file's last chunk if possible, if not, allocate a new chunk and append
    - **Fault-tolerance**: replication among primary and secondaries
    - **Load balancing**: master allocates chunks on below utilization chunkservers
    - **Stale servers**: use chunkserver versioning to detect stale servers, abandon if any
- **Pros**: scalable as master only does metadata-related operations
- **Cons**: chunkserver that stores small files may become hotspot, master takes too many roles?

#### 143 - Bigtable

- Bigtable is a distributed multi-version storage system for general applications
  - **Data model**: a sparse table where byte data is indexed by row, column, and timestamp
    - **Partitioning**: partition across rows, users can use designed names to achieve data locality
  - **Master**: the master is responsible for assigning tablet (i.e. row range) to tablet servers
  - **Tablet server**: handle its tablet-related operations. load SSTs from GFS, combine with memtable, then query on it
  - **Caching**: use memtable for recently written data
  - **Persistence**: use operation logging
  - **GFS**: used to physically store SSTable
  - **Chubby**: master election, tablet server locations
- **Pros**: can support general applications ok, serve web pages well
- **Cons**: no query optimization possible due to byte column/row, general abstraction may not be optimal for individual use cases

#### 144 - MapReduce

- MapReduce is a large-scale data-intensive system that provides two **simple interfaces** (map and reduce), **fault tolerance at worker**, and **data locality**:
  - Use local disk to store intermediate results
  - master and workers
- **Pros**: simple (map and reduce), fault tolerance at worker, parallelize at scale, handle straggler, data locality when scheduling reduce jobs
- **Cons**: no fault tolerance at master, storing intermediate results on disk is slow

#### 145 - Spark

- Spark is also a large-scale data-intensive application that **caches working set in memory** using Resilient Distributed Dataset (RDD) in order to accelerate repeated computation on the working set
  - RDD is a **read-only dataset** that is only stored in memory, can be reconstructed from its parent (**data lineage**) using handle information if lost

- **Pros**: faster with caching in memory, fault tolerance
- **Cons**: applied to batch workloads only

#### 147 - Ray

- Ray is a distributed computation framework that support **heterogeneous computations (interactive and batch)**, **flexible computation model (stateful and stateless)**, **dynamic execution**
  - GCS: centralized store with sharding
  - two level scheduling: local and global
  - in-memory object store: store input and output
  - data lineage for fault tolerance
- **Pros**: high scalable, support heterogeneous tasks
- **Cons**: cannot use application-optimized scheduler

#### 148 - Parallel DB vs. MapReduce

- Parallel DB and MapReduce can both be used to process large dataset at scale, compare and contrast:
  - **Parallel DB** is better at: **performance** (indexing, optimization, compression), standardized schema support
  - **MapReduce** is better at: **scalability** (>10,000 nodes), **setup & tuning**, low-level code flexibility, job **fault tolerance**

#### 149 - Container vs. VM

- Both container and VM are types of virtualization, where:
  - **Container** offers process-level isolation, **application-oriented** design, better **portability**
  - VM offers machine-level isolation, machine-oriented design, **better security** isolation, higher overhead (control transfer b/w VM OS and hypervisor is costly)

## 06 - Databases

#### Practice Questions:

1. (Done) What is hierarchical locking? What is the tradeoff exposed?
2. (Done) What are the four isolation levels?
3. (Done) What is column store? pros and cons?

#### Fundamental Tradeoffs:

1. locking overhead vs. concurrency: granularity of locks
2. overhead vs. isolation: degrees of isolation

#### 102 - System R

- System R is an implementation of relational database that aims to demonstrate its performance is comparable to low-level database even with data independence and high-level language interface
  - **Modular design**: SQL processor (RDS) + access method (RSS)
  - **Data model**: store values instead of pointers
  - **Optimization**: optimize CPU + I/O time
  - **Locking**: lock on individual records
  - **Recovery**: shadow page + WAL
- **Pros**: data independence, high-level language interface, modular design of SQL processing and access method
- **Cons**: no low-level relationship

#### 114 - Granularity of Locks and Degrees of Consistency in a Shared Database

<u>Granularity of Locks</u>

- A hierarchical locking scheme that provides different modes (**X, S, IX, IS, SIX**) for different usages
  - Acquire higher level locks before lower ones
  - Co-existing: **IS and all but X, S and S, IX and IX**
- **Pros**: **optimize concurrency and overhead** without safety problem
- **Cons**: still too much overhead for large databases

<u>Degrees of Isolation</u>

- Four degrees of isolation are read uncommitted, read committed, repeatable read, serializable
- Snapshot isolation: 
  - **Pros**: read always available, read does not block write, no dirty read, no inconsistent read, no phantom read
  - **Cons**: locking still required for writes, stale data may be read

#### 115 - Generalized Isolation Level Defitions

1. PL1 (**Read Uncommitted**): for one transaction, it may read uncommitted value
2. PL2 (**Read Committed**): for one transaction, two reads on the same value may be different (but must be committed)
3. PL2.99 (**Repeatable Read**): for one transaction, any number of reads on the same value will return the same result
4. PL3 (**Serializable**): for one transaction, range read will return the same result

#### 118 - Coordination Avoidance in Database System

- Using **invariants in DB**, the system can determine if it can **structure updates in a coordination-free manner**
- e.g. incrementing a value is coordination-free if invariant is a greater-than relation
- **Pros**: invariant confluence is necessary and sufficient to show coordination-free is possible or not
- **Cons**: stating invariants is not trivial for programmers

#### 136 - C-Store

- **Row Store**:
  - physically store records in a row together
  - **Pros**: write-optimized, one physical write per record write
  - **Cons**: less compression allowed, reads will physically read unnecessary columns
- **Column store**:
  - Physically store a column together
  - **Pros**: more compression allowed since one type per column, reads only physically read related columns
  - **Cons**: one record write requires one-per-column disk writes 

#### 137 - Database Cracking

- **Each query physically re-arranges the location of data** in a way to accelerate future queries that are similar
- **Pros**: does not need DBA to physically arrange data, allow dynamic workload
- **Cons**: DBA may know better, physical arrangement may not converge


## 07 - Fault Tolerance

#### Practice Questions:

1. (Done) What are force and steal? Why are they efficient or not efficient?

#### Fundamental Tradeoffs:

1. simplicity vs. overhead: LRVM
2. performance vs. durability/atomicity: survey

#### 110 - Lightweight Recoverable Virtual Memory

- LRVM is a simple layer that enables applications to have transactional properties on mapped segments
  - segments (e.g. metadata) are stored on disk, mirrored in virtual memory by applications
  - applications explicitly set which regions to copy, when it is modified
  - use undo/no-redo logging because all changes are committed to log first
  - implements a copy-on-write
- **Pros**: a simple yet very useful layer
- **Cons**: extra overhead if additional functionality needs to be implemented on top

#### 119/166 - Transaction-Oriented Recovery Survey

<u>Steal and No Force</u>

- Steal: uncommitted transaction's new value written back to disk, related to atomicity and UNDO
- No Force: committed transaction's new value not written to disk at commit, related to durability and REDO

<u>Write-Ahead Logging (WAL)</u>

- Can log either old/new value or operation(args)
- Atomicity: UNDO log record needs to be written to disk, before uncommitted page is flushed to disk
- Durability: Commit & all txn's records needs to be written to disk, before return to caller

<u>Crash Recovery</u>

1. Phase 1: analyze the log to find all uncommitted and committed transactions, load checkpoint
2. Phase 2: REDO the records from the first dirty page
3. Phase 3: UNDO record of failed transactions from the end of log

#### 167 - Segment-Based Recovery

- Segment-Based Recovery relaxes the unit of recovery from a page to a segment, which can include multiple pages, to facilitate large record recovery.

## 08 - Virtualization

#### Practice Questions:

1. (Done) How does live migration of VM work? what workload does it work the best? which the worst?

#### Fundamental Tradeoffs:

1. performance vs. portability: Disco, Xen
2. performance vs. security/isolation: vm vs. container

#### 128 - Disco

- Disco is an implementation of **full virtualization** where VMM is a layer between hardware and OSes
  - **Processor**: VMM runs in ring 0 (kernel), guest OS in ring 3 (supervisor), o/w user mode. Traps are handled by the VMM
  - **Memory**: OS knows virtual -> physical, VMM knows physical -> machine, VMM uses a software TLB. TLB needs to be flushed when switching between VMs
  - **Device I/O**: VMM intercepts, maps, translates, and redirect device I/O. Copy-on-write if needed
  - **NUMA Memory Management**: dynamic page migration and replication among machines to support non-NUMA-aware OS
- **Pros**: no modification to OS or app required
- **Cons**: TLB miss expensive (flushed when VM switch, each miss require VMM overhead), processor overhead, device I/O overhead, significant overhead compared to paravirtualization

#### 129 - Xen

- Xen is an implementation of paravirtualization where guest OSes (not app) need to be modified
  - **CPU**: allow guest OS to pre-register exception handler to bypass Xen for certain syscalls
  - **Memory**: 1) allow guest OS to see machine memory and set virtual -> machine mapping in hardware TLB (need validation from Xen, can be batched); 2) VMM exists in every address space, no context switch needed to access
  - **Device I/O**: asynchronous I/O ring is used to achieve zero-copy vertical passing
  - **Network**: VMM offers Network interface for VM to bind to, the network interface accepts packets on behalf of VM and pass to them through zero-copy free-page exchange
  - **Disk I/O**: VMM intercepts, translates, and maps to a physical location, then use DMA to allow zero-copy exchange
  - **Management**: done with domain0, a guest-OS level VM instance; VMM only offers basic operations

- **Pros**: very low performance overhead (2~5%), separate protection and management, zero-copy whenever possible
- **Cons**: need OS modifications

#### 130 - Live Migration of VM

- Live migration of VM uses a three-step process to allow VMM to migrate guest OS between machines with a careful usage of CPU, memory and network
  - Three steps: 1) First round copy all pages over; 2) iteratively copy modified pages (analyze with writable working set with VMM shadow page); 3) stop and copy all remaining modified pages over, redirect and notify network, and remount disk
- vs. process migration: 1) no residual dependency problem; 2) allow migrating all state; 3) separate of concern between user and os (user doesn't need to give any permission, os doesn't need any app info)
- **Pros**: minimize downtime, minimize total migration time, minimize resource utilized, effective for small WSS, no residual dependency
- **Cons**: not efficient if WSS is significant (longer downtime)

#### 131 - Performance Comparison between VM and container

- Kernel Virtual Machine: Linux allows it to run as a hypervisor and allow guest OS to run in a Linux process
- Linux Container: Linux runs natively, uses Linux namespace feature to add container ID to processes. Each container has no visibility or access to outside containers

- Comparison:
  - **Isolation**: VM is safer because it provides strict isolation, where one VM crashes will not affect VMM and other VMs. Container crash may crash the underlying OS.
  - **Performance**: VM has extra overhead in 1) CPU because double scheduling and VMM-level emulated traps; 2) memory because paging requires VMM validation or intercept; 3) disk/network I/O because they are emulated by VMM. Container runs close to native.
  - **Portability**: container is highly portable

#### 132/133 - Cloudlet

- Mobile devices can use **cloudlet on the edge** to **apply its VM overlay on base VM** for computation jobs
- **Pros**: lower latency, higher throughput, fast setup time
- **Cons**: how to achieve persistent storage? How to find and negotiate cloudlet? Security of cloudlet?

#### 165 - Global Data Plane

- GDP shows a **data-centric view** of the world where **data is stored in logs** 
  - log servers are distributed across the cloud and the edge
  - Security and privacy: each log stores its owner's public key so only owner's private key can have access, stored data is encrypted
  - Addressing: DHT
  - Durability & locality: client can specify which log servers to store data
- **Pros**: good for streaming data with relaxed latency overhead
- **Cons**: directly manipulating a materialized view of DB can be more efficient for many other systems

## 09 - Security

#### Practice Questions:

1. (Done) How does CryptDB provide data security?
2. (Done) How does HiStar provide information confidentiality compared to UNIX?
3. (Done) How does spectre attack and meltdown work?

#### Fundamental Tradeoffs:

1. security vs. performance: all
2. integrity vs. performance: Crash Hoare Logic

#### 153 - seL4

- seL4 shows that a **formal verification of a microkernel** is possible, although the effort required is significant (years of labour)
- **Pros**: proving microkernel is formally verifiable
- **Cons**: significant amount of effort, not practical for real systems, performance hit

#### 156 - HiStar

- HiStar is a microkernel that **controls information flow** where sensitive data is marked **tainted** and **cannot be sent outside** of the computer by unprivileged malicious process
- **Pros**: small kernel so safer, processes cannot accidentally/intentionally leak information
- **Cons**: overhead of having label for all kernel components

#### 157 - Crash Hoare Logic

- Crash Hoare logic extends **hoare logic with crash condition** to prove FSCQ file system's **integrity even under crash**
- The key is to **prove core logging's integrity** under crash, upon which transactions can be assumed
- **Pros**: provable integrity under crash condition
- **Cons**: significant performance overhead

#### 158 - CryptDB

- CryptDB uses an **onion layer of encryption schemes** to, from inside out, **gradually expose more operations possible** on the data, while reducing the level of security on the data
  - operations such as equality, ordering, keyword matching, addition
- Enable users to have decrypt key permuted with password to ensure **security when user log out**
- Threat model: 1) curious DBA; 2) snapshot attacker controlling both app and DB servers 
- **Pros**: allow different levels of performance for different levels of encryption, protect user data when logged out (snapshot attacker)
- **Cons**: hard to protect from pessimistic attacker, overhead of extra levels of decryption

#### 159 - Opaque

- Opaque aims to create **data-independent access pattern** to prevent leaking data information via data reconstruction attack
- **Oblivious** mode: 1) encryption; 2) authentication; 3) computation integrity; 4) data-independent computation
- Threat model: attack gains control of the entire cloud software stack
  1. network traffic modification
  2. OS kernel
- **Pros**: prevent leaking access pattern information leakage
- **Cons**: performance overhead

#### 160 - Meltdown

- Meltdown is a side-channel attack by **raising trap exception** and then **immediately lookup kernel memory**, so some kernel data is stored in cache
- The attacker then checks all locations in kernel memory to check which address is fresh in cache

#### 161 - Spectre Attack

- Spectre attack is a side-channel attack by **mis-training speculative execution** to **execute wrong branch** to lookup kernel memory, so some kernel data is stored in cache
- The attacker then checks all locations in kernel memory to check which address is fresh in cache

## 10 - Multiprocessors

#### Practice Questions:

1. Does RAMCloud achieve fault tolerance? Why?
2. Compare kernel-level, user-level, and hybrid threading?

#### Fundamental Tradeoffs:

1. performance and durability: RAMCloud

#### User Thread and Kernel Thread

- **Kernel-level threading** (1:1): 
  - threads created by the user has 1:1 correspondence to kernel thread (Lightweight process)
  - **Pros**: simple
  - **Cons**: context switch requires kernel-level switch, expensive
- **User-level threading** (N:1):
  - all user-level threads map to one kernel thread
  - **Pros**: context switch is cheap, user has more information for efficient scheduling
  - **Cons**: only one thread can run, cannot utilize multi-processor, all threads stale if one user thread sync blocks and not swapped out by library
- **Hybrid threading** (M:N):
  - M user threads mapped onto N kernel threads
  - **Pros**: allow user-level scheduling, user-level context switch, utilize multi-processor
  - **Cons**: complex, suboptimal scheduling unless substantial coordination between user and kernel scheduler

#### 185 - RAMCloud

- RAMCloud stores **everything in RAM** (instead of disk) to optimize latency
- **Fault tolerance**: **replication** is used
  - replicate to 3 machines' RAM before returning write
  - RAM periodically stored to disk for durability
- **Pros**: very low latency because data is in memory
- **Cons**: still no clear solution for instant fault tolerance (e.g. power failure) without synchronously storing to disk

#### 186 - Memory Coherence in Shared Virtual Memory Systems 

- Sharing address space among all processors, each has its local memory
- The two key design decisions for memory coherence protocol:
  1. page size: larger -> higher throughput; smaller -> less conflict
  2. memory coherence strategy: a manager sends invalidation messages to secondary copies when modified
- **cannot use cache coherence protocol** because **communication overhead is higher** than cache's bus
- **Pros**: high level of abstraction
- **Cons**: **poor scalability**

#### 190 - Scheduler Activations

- Scheduler activation allows **kernel** to **provide "virtual processor" to user** thread library
  - **user library manages what threads to schedule** and what resources to request
  - **kernel notifies** user library (via upcall) when **process allocation and deallocation** happen
- **Pros**: coordination between user and kernel threading to achieve optimal scheduling
- **Cons**: complex to implement as modification to both user and kernel code

## 11 - Virtual Memory

#### Practice Questions:

1. (Done) What is working set? What could happen if a process is in RAM but its working set is not?

#### Fundamental Tradeoffs:

1. space vs. performance: working set

#### 183 - Working Set

- Working set is the **resident set** of a process to allow it operate with **minimal page faults**
  - **Temporal locality**: the pages accessed within ω seconds
  - **All or nothing**: if a process is paged out, its entire resident set needs to be; vice versa
- **Pros**: prevents thrashing, optimize the number of parallel processes
- **Cons**: if tolerate more page faults, can save more space?

#### 184 - Virtual Memory Management in VAX/VMS

- Improves process memory isolation, increase disk I/O throughput, whole-unit swapper
  - **Process memory isolation**: each process has a free pool, isolated from others
  - **Increase disk I/O throughput**: read extra faulted pages, batch write modified pages
  - **Whole-unit swapper**: swap processes and all its pages in/out of memory from/to disk

## 12 - Communication

#### Practice Questions:

1. (Done) What does lightweight RPC do?
2. Compare active message and RPC

#### Fundamental Tradeoffs:

1. abstraction level vs. performance: RPC vs. MP vs. DSM

#### 176 - Implementing RPC

- RPC uses **client and server stubs** to **marshal/unmarshal message** and send over network, client and server see it as normal "local" procedure call
  - **Naming and location**: server publishes procedure to a distributed database that client stub can bind their procedure call to it, and track its server location
  - **Low latency**: no setup or tear down, wait infinitely if server stub responds
- **Pros**: user does not need to know process address, can use it as a local procedure call, no restriction on message format (can be marshalled), moderate to implement
- **Cons**: user still need to explicitly manage messages

#### 177 - Lightweight RPC

- Lightweight RPC **optimizes local small RPC** through simpler control and data transfer:
  - **Control transfer**: client thread runs directly in server domain
  - **Data transfer**: use shared buffer among processes, no extra copy
- **Pros**: lower latency due to faster control and data transfer
- **Cons**: client thread running in server domain may compromise security

#### 178 - Active Messages

- Active messages use a **pre-registered server-side handler** that **busy polls from network** and put to computation
  - **Handler**: compared to RPC, handler does not execute, but only send to another computation thread
  - **Queuing and buffering**: minimizes queuing, only buffer at network stack, polled from network asap by handler
  - **Async client**: no blocking at client
- vs. RPC: 
  - **OS involvement**: required for RPC to move data between kernel and user space; active message eliminate OS intervention
  - **Security**: less of concern for active message because from a single program
- **Pros**: low latency
- **Cons**: congestion control and fault tolerance?









