# OS Review 

## Transactions and Concurrency

- Problems:
  - Inconsistent reads: read partial updates
  - Lost updates: update overwritten
  - Dirty reads: read dirty value before it is rolled back
- Transaction: a sequence of actions that are executed in a single, logical, atomic unit.
- Transactions guarantees ACID properties:
  - Atomicity: either commit or abort
  - Consistency: if consistent before txn, then still consistent after txn
  - Isolation: transactions are executed as if isolated from each other
  - Durability: if a txn commits, it persists
- 2 phase locking
  - Shared and exclusive locks
  - cannot acquire new locks after releasing any lock
  - strict 2PL: release all locks at once

## Recovery

- Force/No Force
  - Force: flush memory to disk after every commit, ensures durability
  - want no force
- Steal/No Steal
  - No steal: no memory written to disk before commits, ensures atomicity
  - want steal
- Write-Ahead Log (WAL) requirements
  - log written to disk before any data page written to disk
  - All log records need to be written to disk when a txn commits

## MapReduce and Spark

- MapReduce
  - Two steps: map and reduce
  - store intermediate values to disk for fault tolerance
  - For stragglers, use idle workers to start same executions
- Spark
  - Multiple steps instead of just 1 map and 1 reduce
  - Store values to memory instead of disk -> faster
    - Keep track of a lineage for fault tolerance
    - If crashes before an execution finishes, the entire execution needs to be restarted
  - Two operations:
    - transformations: lazy for better operation tree
    - actions: eager
