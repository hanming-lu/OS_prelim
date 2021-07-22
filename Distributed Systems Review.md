# Distributed Systems Notes

## Intro

<u>Transparency</u>

- Access: Hide differences in data representation and resource access
- Location: hide where the resource is located
- Migration: hide when resource is moved
- Replication: hide multiple copies may exist for reliability and availability
- Concurrency: hide resources may be shared concurrently
- Failure: hide failure and recovery of resource
- Relocation: Hide resources being moved during use

<u>Openness</u>

- Def: the interfaces are clearly specified and freely available
- Interoperability: two different applications or systems working together
- Portability: run on different systems
- Extensibility: easy to add new features

## RPC

- Client app -> client stub -> client os -> server os -> server stub -> server app
- RPC Implementation types:
  1. Shallow integration: must use lots of libcalls to set things up
  2. Deep integration: data formatting done based on type declarations
- RPC Challenges - solution is expose remoteness to client:
  1. Partial failures 
  2. Latency
  3. Memory Access 
- RPC major overheads: copies and marshaling/unmarshaling

## Distributed File System

- Why DFS:
  - Data sharing among multiple users
  - location transparency
  - User mobility
  - Scalability
  - Backups and centralized management
- Challenges:
  - Heterogeneity
  - Performance
  - Scalability
  - Security
  - Failures
  - Concurrency
  - Consider geographic distance and high latency
- Design Choices and implications:
  - Client-side caching
  - Consistency
    - timeout
    - callbacks
    - tokens (similar to lock)
  - Naming
  - Authentication and access control
- Semantics of file sharing
  - Unix: every operation is instantly visible
  - Session: visible to other processes after file is closed
  - Immutable: no changes allowed
  - Transactions: all changes occur atomically

## Time, Synchronization

- Lamport clock
- Vector clock

## Mutual Exclusion

- Requirements:
  - Correctness/safety
  - fairness
- Totally-Ordered Multicast
  - each msg is timestamped
  - msg multicast to everyone, including the sender
  - receiver puts msg into a local queue, then send acks to everyone
  - only deliver msg when it is at the head of queue, and acked by everyone

## Concurrency

- Single server case: Two phase locking
- Distributed transactions: two phase commit

## Recovery

- Recovery strategies
  - backward recovery
    - con: checkpointing is expensive
  - forward recovery
    - con: all errors need to be considered upfront. When one occurs, knows how to forward the system.
- Shadow pages
  - atomicity and durability
  - Essentially copy-on-write where a new page is pointed to (atomically) when commit
  - When abort, just discard the new page
- Write-Ahead-Logging (WAL):
  - Write to a log (disk) before applying changes (in memory)
  - Considered durable once logged to disk
  - After crash, replay changes in log to recover

## Consistency

<u>Strong Levels</u>

1. Strict serializability
2. Linearizability
3. Sequential
4. Causal
5. Eventual

<u>Properties</u>

- **Total Order**: There exists a legal total ordering of operations
  - legal: a read operation sees the latest write operation
- **Preserves real-time ordering**: Any operation A that completes before operation B begins, occurs before B in the total order

<u>Strict Serializability</u>

- **Total Order**: There exists a legal total ordering of *transactions*
  - legal: a read operation sees the latest write operation
- **Preserves real-time ordering**: Any *transaction* A that completes before *transaction* B begins, occurs before B in the total order
- writes in a completed transaction appear to all future reads
- Once a read sees a value, all future reads must see the same value until new write
- pro: easily reason about correctness of transactions
- cons: high read and write latencies

<u>Linearizability</u>

- total order & preserves real-time ordering on *operations*
- difference from strict serializability:
  - in linearizability, clients only have consistency guarantees for operations (instead of transactions)
  - strict serializability allows clients to use transactions
- A completed write appears to all future reads
- Once a read sees a value, all future reads must also return the same value
- Pros: easy to reason correctness
- cons: high read and write overheads

<u>Sequential Consistency</u>

- total order: there exists a legal total ordering of operations
- preserves process ordering: all of a process' operations appear in that order in the total order
- difference from linearizability:
  - sequence of ops across processes not determined by real-time, so more orderings than linearizability
- pros: can have more orderings than linearizability
- cons: many possible sequential executions

<u>Causal+ consistency</u>

- Partial order: order causally related ops the same way across all processes
- +: replicas eventually converge
- difference from sequential consistency:
  - only causally related ops are ordered
  - concurrent ops may be ordered differently across different processes
- pros: preserves causality while improving efficiency
- cons: need to reason about concurrency

<u>Eventual Consistency</u>

- Eventual convergence: if no more writes, all replicas eventually agree
- difference from causal consistency:
  - does not preserve causal relationships
- frequently used with application conflict resolution
- pros: highly available
- cons: no safety guarantees, need conflict resolution

#### Also...

- Sequential Consistency:
  - Operations seem to appear to take place in some total order
  - The total order is consistent with the order of operations on each individual process
- Causal Consistency:
  - Writes that are potentially causally related must be seen by all processes in the same order
  - Concurrent writes may be seen in different order
- Replicate:
  - Propagate only a notification of update, so invalidate other processes' existing state
  - Propagate update operation and data
  - Propagate only update operation, no data transfer
- Primary-Backup
  - Primary handles operations
  - Backup are passive
  - Clients see failures and handle it
  - Problem: if something fails, response time will be large
- Quorum consensus
  - Paxos:
    - lets all nodes to agree on the same value despite f node failures, network failures, and delays
      - Correctness: all nodes agree on the same value; the value is proposed by some node
      - Fault-tolerance: reach consensus eventually if <=f nodes fail
      - Liveness is not guaranteed
- FLP Impossibility Result
  - It is impossible for a set of processors in an asynchronous system to agree on a binary value, even if one processor is subject to an unannounced failure
    - async: timeout is not perfect
- Paxos general approach
  - A leader is elected
  - The leader asks other replicas to exclusively listen to him
  - Replicas ack
  - The leader proposes a value, send to replicas
  - Replicas ack
  - The leader sends commit after receiving a majority of acks
- 

## Errors

- Error detection
  - Parity checking: Single Bit Parity
  - Block Error Detection bit: redundancy
  -  Checksum
  - cyclic redundancy check (CRC): can detect error, as well as correct error
- Error recovery
  1. Redundancy: 1) Error correction code; 2) replication/voting
  2. retry
- Multiple disks
  - Capacity
  - Performance
  - Availability

## DNS

- DNS is a large distributed database
  - scalability
  - robustness
  - independent state
- Hierarchical structure
- DB contains tuples called resource records (RR)
- Various kinds of mappings:
  - 1-1 mapping between domain name and IP addr
  - multiple domain names mapping to the same IP addr
  - Single domain name mapping to multiple IP addr
  - Some valid domain name doesn't map to any IP addr
- Caching
  - DNS responses are cached
  - DNS negative queries are cached
  - cached data periodically time out
- Reliability
  - DNS servers are replicated
  - UDP used for queries
- Recursive vs. iterative DNS name resolution

## Naming

- Name lookup styles	
  - table lookup: name
  - recursive: context + name
  - multiple lookup: try multiple contexts to resolve name
- Naming model: 3 elements
  - Name space: alphabet+symbol that specifies name
  - Name-mapping: name -> value
  - Universe of values: objects
- Content-based naming
  - Name items by their hash
  - self-certifying names
  - Pre-image resistance: given an input h1, it is hard to find any m such that h1 = h(m)
  - Collision resistance: it is hard to find values m1, m2 such that h(m1)=h(m2)
- Consistent hashing
  - when a hash table is resized, only n/m keys need to be remapped on average (n=key #, m=node #)
  - Desired features
    - balanced: load is equal across nodes
    - smoothness: little impacts when buckets added/removed
    - spread: small set of nodes may have an object
    - load: across all views, # of objects assigned to hash bucket is small
- Message authentication code (MAC)
  - H(data, key): can only create or verify hash with the key

## Content Distribution Network

- Server selection
  - Application based
    - HTTP to the primary server, then use HTTP redirection - have drawbacks
  - Naming based 
    - client does name lookup for service
    - name server chooses appropriate server address
    - name server make decision based on:
      - server load/location
      - info in the name lookup request
- How akamai works?
  - Akamai only replicates static content
  - an Akamai server receives a request -> checks local cache -> ask primary server if not present
- Caching proxies
  - capacity: mem & disk
  - compulsory: first access/non-cacheable contents
  - consistency: doc has been updated/expired before reuse

## P2P

- Leverage the resources of client machines (peers)
- P2P routing
  - centralized
  - flooding
  - swarming: 1) contact centralized tracker for peers; 2) be a tracker; 3) get tracker info by google; 4) fetch and upload to peers
  - unstructured
  - structured: chord
- Chord
  - ring-based
  - Finger table

## MapReduce

<u>MapReduce</u>

- Map and then reduce

<u>Hadoop</u>

- Overview
  - a file system with files distributed across nodes
  - Store multiple (3 copies)
  - Any node has access to any file
  - map -> shuffle -> reduce
- Characteristics
  - Computation broken into many small tasks
  - input and output are files on disk
- Strength
  - Great flexibility in placement, scheduling, and load balancing
  - Can access large files
  - Scalability adv.
    - distributed system principles -> scalable
    - dynamically scheduled tasks
  - Provision adv.
    - Can have consumer grade hardware
    - Can have heterogeneous hardware
  - Operational adv.
    - Minimal staffing
    - no downtime
- Weakness
  - Higher overhead
  - lower raw performance
  - Coarse-grained parallelism: low communication
  - no locality of data or activity
  - map/reduce needs to complete sequentially
- Implementation
  - Built on top of parallel file system (GFS, HDFS)
  - breaks works into tasks
  - input/output are files

## Virtualization

Definition:

- Provide APIs that provide a determined set of features made available to language and programs. 

<u>Properties</u>

- Isolation
  - Fault isolation
  - performance isolation
- Encapsulation
  - capture all VM states
  - enable vm clones/snapshots
- Portability
  - Migrate VMs
- Interposition
  - transformations on instructions, memory, I/O

<u>Types of System Virtualization</u>

1. Native/Bare metal: hardware -> supervisor -> VMs
2. Hosted: host OS (supervisor (VMs))

<u>Types of Virtualization</u>

1. Full virtualization
   - No OS modification, virtualization is transparent to OS
   - poorer performance because of binary translation
2. Para virtualization
   1. OS modified to use on virtualization layer
   2. Performance at the cost of transparency

<u>Virtual Machine Monitor</u>

- definition: an efficient, isolated duplicate of the real machine
- Properties:
  - **Equivalent execution**: programs running in the virtualized environment run identically as natively
  - **Performance**: minor decrease in performance
  - **Safety and isolation**: VMM has complete control over access to system resources
- Virtualize CPUs
  - timeslice the VMs, each VM runs OS/Apps
- Virtualize Memory
  - VMM partitions machine memory to physical memory for VM OS to use (3 abstractions of mem: machine, physical, virtual)
  - Shadow Page Table: 
    - VMM page tables, that map from virtual to machine
    - Needs to be consistent with OS's page tables
- Virtualize I/O
  - Many devices, not realistic to writer drivers to all
  - solution: Present virtual I/O devices to guest VMs and channel I/O requests to a trusted host VM
  - Networking: bridge (direct access) or NAT
- VM strorage
  - Store as files in the host file system
  - network attached storage

## RAID

- Disk Striping: interleave data across multiple disks

- Parity: XOR all disks

  - parity disk is the bottleneck, all writes go to it, degraded reads go through it -> solution is to strip parity disk

- RAID Levels

  - |        | Definition                    | Capacity | Reliability | Performance                       |
    | ------ | ----------------------------- | -------- | ----------- | --------------------------------- |
    | RAID 0 | Striping only                 | N        | 0           | Nx                                |
    | RAID 1 | Mirror disk                   | N/2      | 1           | 1/2 Nx for write, 2Nx for read    |
    | RAID 4 | Block-based Parity            | N-1      | 1           | N read, bad for write             |
    | RAID 5 | Block-based Parity + striping | N-1      | 1           | Good for both read and write      |
    | RAID 6 | P+Q Redundancy (2x parity)    | N-2      | 2           | worse for write compared to RAID5 |

<u>Three Modes of Operation</u>

1. Normal mode: max efficiency
2. Degraded mode: some failure, use degraded mode operations
3. Rebuild mode: degraded mode + compete with rebuild

## Byzantine Fault Tolerance

<u>Definition</u>

- A failure is when a system cannot meet its promises
- An error of a part of the system can lead to failures
- The cause of an error is called a fault

<u>Failures</u>

- Crash failure
- Omission failure: fails to respond to incoming requests
- Timing failure: timeouts
- Response failure: wrong response values
- Arbitrary failure: arbitrary response at any time

<u>Requirement</u>

- Any two quorums must have one honest node: (N-f) + (N-f) - N = f+1 -> N = 3f+1

<u>Assumptions:</u>

1. Messages are delivered correctly
2. The receiver knows who sent the message
3. Message delivery time is bounded

<u>Byzantine Agreement Algorithm</u>

1. Each process sends it value to everyone
2. Each process uses the received messages to construct a vector of messages, and send to everyone
3. Each process uses the received vector of messages to compute

<u>Async BFT:</u>

- Async because:
  - faulty network can violate timing assumptions
  - can prevent liveness

<u>FLP Impossibility</u>

- Async consensus may not terminate

<u>BFT Limitations</u>:

1. Expensive
2. Protection only when <= f nodes fail
3. Other attacks possible:
   - Steal SSNs from servers

## Security

<u>Needed for a secure communication channel</u>

- Authentication: you are who you are
- Confidentiality: My data is not exposed
- Integrity: My data is not modified

<u>Attacks:</u>

- Authenticity attack: impersonate a node
- Integrity attack: delay msg, modify msg, release msg
- Availability attack: destroy hw/sw; corrupt pkts

<u>Tools</u>

1. Hashing
   - Consistent
   - one-way
   - collision resistant
2. Secret-key cryptography: symmetric key
   - Use the same key for both encryption and decryption
   - fast crypto
   - need to share secrete beforehand
   - Confidentiality:
     - Steam ciphers: key-generated stream of L bits XOR with msg of length L => encrypted ciphertext
     - Block ciphers: break into blocks, feed into encryption/decryption engine using key
   - Integrity:
     - HMAC
       1. hash(msg, key) => MAC
       2. Append MAC to msg, and send
       3. Receiver calculates hash again, verify MAC is correct => authentication + integrity
   - Authentication:
     - HMAC and nonce (A random bitstring used only once). Alice sends nonce to Bob as a challenge, Bob replies with MAC result
3. Public-key cryptography: asymmetric key
   - Key pair, one public, one private, inverse of each other
   - slower crypto
   - no need to share secret beforehand
   - Confidentiality:
     - encrypt with the public key of receiver
   - Integrity:
     - Sign message with private key of the sender
   - Authentication:
     - entity being authenticated signs a nonce with private key, signature is then verified with the public key

<u>Key Distribution Center (KDC)</u>

- Alice, Bob need shared symmetric key
- KDC: server shares different secret key with each registered user
- Alice, Bob know own symmetric keys for communicating with KDC

- When Alice talks to Bob:
  - Send a request to KDC, sign using their own KDC's symmetric key
  - KDC gives Alice a session key R1 with Bob, and sign K(A, R1)
  - Alice sends K(A, R1) to Bob, Bob can verify it's from KDC

- KDC is the centralized trust and point of failure

<u>Public Key Infrastructure (PKI)</u>

- A system in which "root of trust" authoritatively assigns public keys to real-world identities
- A significant stumbling block in deploying many "next generation" secure Internet protocol or applications

<u>Certification Authority (CA)</u>

- binds public key to a specific entity, E
- Registration:
  1. E provides proof of identity to CA
  2. CA binds E to E's public key, CA signs E's public key with CA's private key. A certificate is created
  3. The certificate contains E's public key, and CA's signature on E's public key
- Verification of Bob's public key:
  - Alice gets Bob's certificate somewhere
  - Alice verifies CA's signature, then accepts

<u>Transport Layer Security (TLS) aka. Secure Socket Layer (SSL)</u>

- Between application and transport layer
- Used for protocols like HTTPS
- Handles confidentiality, integrity, authentication

<u>SSL setup with server</u>

1. Client and server agrees on exact cryptographic protocols
2. Server sends client a certificate with CA public key
3. Client sends server a challenge
4. Server proves it has the secret key
5. A symmetric session key is used afterwards

<u>How SSL works</u>

1. receive a stream of data
2. separate into segments
3. a session key is used to encrypt and MAC each chunk
4. Fed segments to TCP as a stream

<u>Diffie-Hellman Key Exchange</u>

- Situation: Generate keys between two people, securely, no trusted, even if someone is listening in
- Overview:
  1. Decide on a public color
  2. each has a secret color
  3. share publicly the mix of public + secret color
  4. mix the received result with their secret color
  5. The result color is the symmetric key
