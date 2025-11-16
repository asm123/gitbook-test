# Chapter 7: Transactions

* A system has to deal with faults in order to be reliable. Implementing fault-tolerance mechanisms is a lot of work.
* Transactions have been the mechanisms of choice for simplifying the fault-tolerance issues.
  * Conceptually, all the reads and writes in a transaction are executed as one operation: either the transaction succeeds (commit) or it fails (abort, rollback). Application can safely retry if it fails.
  * Safety guarantees: Application is free to ignore certain potential error scenarios and concurrency issues because the database takes care of them.

## Safety guarantees - ACID

* **Atomicity**
  * It is not about concurrency.
  * Atomicity = abortability = the ability to abort a transaction on error (a fault occurs after some of the writes have been processed) and have all writes from that transaction discarded.
  * Can be implemented using a log for crash recovery.
* **Consistency**
  * Application-specific notion of a database being in "good state". Application has invariants = certain statements about your data, that must always be true.
  * It is a property of the application and not the database. Database cannot guarantee it. It is the application's responsibility to define the transactions correctly so that they preserve consistency.
  * Not the same as consistency in replica consistency, CAP theorem or consistent hashing.
* **Isolation**
  * Concurrently executing transactions are isolated from each other. Each transaction can pretend it is the only transaction running on the database.
  * Formalized as serializability - database ensures that when the concurrent transactions have committed, the result is the same as if they had run one after the another.
  * Serializable isolation is rarely used in practice due to resulting performance penalty.
  * Can be implemented using a lock on each object - allowing only one thread to access an object at any time.
* **Durability**
  * Once a transaction is committed successfully, the data is persisted even in the cases of faults and crashes.
  * Single-node: Data has been written to a nonvolatile storage. Involves a a write-ahead-log for recovery.
  * Replicated: Data has been successfully copied to some number of nodes.

No one technique can provide absolute guarantees. There are only various risk-reduction techniques.

* Multi-object operations are needed when multiple pieces of data need to be kept in sync at once. For example, maintaining referential integrity with foreign keys, updating denormalized information in document data model, updating secondary indexes
* Difficult to implement in distributed datastores across partitions, can get in the way of high availability and performance, error handling without atomicity and lack of isolation can cause concurrency problems.

**Handling errors and aborts**

* ACID databases: If the database is in danger of violating its guarantee of atomicity, isolation or durability, it would rather abandon the transaction entirely than allow it to remain half-finished.
* Datastores with leaderless replication: best-effort basis - db will do as much as it can, and if it runs into an error, it won't undo something it has already done. It is the application's responsibility to recover from errors.
* Retrying an aborted transaction is a simple and effective error handling mechanism, but may not always work to the desired effect. For example, n/w failure after committing but before acknowledging the success of the commit, errors due to overload, permanent errors, transactions with side-effects outside of the database, failure of client process while retrying.

## Weak isolation levels

* Concurrency issues only come into play when one transaction reads data that is concurrently modified by another transaction, or when two transactions try to simultaneously modify the same data.
* Concurrency bugs are hard to find by testing, difficult to reproduce, difficult to reason about in large applications.
* DBs try to hid concurrency issues from application developers by providing transaction isolation.
  * Serializable isolation: db guarantees that concurrent transactions have the same effect as if they ran serially. Has performance cost in practice.
  * Weaker isolation levels: commonly used in practice. Harder to understand and can lead to subtle bugs.
    1. Read committed
    2. Snapshot isolation

**Nonrepeatable read:** When a transaction reads the same row twice and gets different data each time. Example: read skew.

**Phantom read:** A phantom read occurs when, in the course of a transaction, two identical queries are executed, and the collection of rows returned by the second query is different from the first.

Example:

* User A runs the same query twice.
* In between, User B runs a transaction and commits.
* Non-repeatable read: The A row that user A has queried has a different value the second time.
* Phantom read: All the rows in the query have the same value before and after, but different rows are being selected (because B has deleted or inserted some).

Source: [StackOverflow](https://stackoverflow.com/a/11044968).

**Read Committed**

Makes two guarantees:

* **No dirty writes**: while writing to the database, you will only overwrite data that has been committed.
  * Dirty write: Two concurrent transactions try to update the same object in a db, if an earlier write is part of a transaction that has not yet committed, the later write overwrites an uncommitted value.
* **No dirty reads**: while reading from the database, you will only see data that has been committed.
  * Dirty read: A transaction sees uncommitted data from another transaction. May result in a transaction seeing some updates but not others, seeing data that is later rolled back.
  * Any writes by a transaction should only become visible to others when that transaction commits.

Implementation:

* Dirty writes are prevented using row-level locks for writing.
* Dirty reads may be prevented using row-level locks for reading -> transactions will be slow. Commonly implemented as: for every object that is written, database remembers both the old committed value and the new value set by the transaction that currently holds the write lock.

Problems:

* Does not prevent race conditions.
* Read skew is considered acceptable under read committed isolation. Some situations cannot tolerate temporary inconsistencies arising from rad skew - backups, analytic queries and integrity checks.

**Snapshot isolation and repeatable read**

* Each transaction reads from a consistent snapshot of the database.
  * Transaction sees all the data that was committed in the database at the start of the transaction.
  * Even if the data is subsequently changed by another transaction, each transaction sees only the old data from that particular point in time.
  * Readers never block writers, writers never block readers.
* Implemented as _multi-version concurrency control (MVCC)_: Database keeps several different committed versions of an object, because various in-progress transactions may need to see the state of the database at different points in time.
* Read committed uses a separate snapshot for each query, snapshot isolation uses the same snapshot for an entire transaction.
* Visibility rules for observing a consistent snapshot:
  * DB makes a list of all the in-progress transactions at the start of each transaction. Any writes that those transactions have made are ignored even if the transactions subsequently commit.
  * Any writes made by aborted transactions are ignored.
  * Any writes made by transactions with a later transaction id are ignored regardless of whether those transactions have committed.
  * All other writes are visible to the application's queries.
* Indexing:
  * Point to all the versions of an object and require an index query to filter out any object versions that are not visible to the current transaction. Garbage collection removes index entries corresponding to the old object versions.
  * Use an append-only / copy-on-write variant that creates a new copy of modified B-tree page instead of overwriting the page on update. Also requires compaction and garbage collection.

#### Preventing lost updates

Lost update problem: An application reads some value from the database, modifies it and writes back the modified value (read-modify-write cycle). If two transactions do this concurrently, one of the modifications can be lost because the second write does not include the first write.

Possible solutions:

* Atomic write operations: `UPDATE counters SET value = value + 1 WHERE key = 'foo';`
  * Remove the need to implement read-modify-write cycle.
  * Usually implemented as _cursor stability_ - take an exclusive lock on the object when it is read so that no other transaction can read it until the update as been applied.
  * Alternatively, force all the atomic operation to be executed on a single thread.
* Explicit locking: Application explicitly locks objects that are going to be updated and then performs read-modify-write cycle.
* Automatically detecting lost updates: All read-modify-write cycles to execute in parallel. If the transaction manager detects a lost update, abort the transaction and force it to retry its read-modify-write cycle.
  * DBs can perform the check efficiently in conjunction in snapshot isolation.
* Compare-and-set: Allow an update to happen only if the value has not changed since you last read it. If the current values does not match the previously read value, the update has no effect and the read-modify-write cycle must be retried.
  * May not always prevent lost updates because the read condition may be true even though another concurrent write is occurring.
* Conflict resolution and replication: Replicated databases may have data that is potentially modified concurrently on different nodes.
  * Common approach is to allow concurrent writes to create several conflicting versions of a value and use application code or special data structures to resolve and merge these versions after the fact. // TODO: examples of such data structures?
  * Commutative atomic operations - apply in different order on different replicas and still get the same result - can work well.
  * Last write wins (LWW) is default in many replicated databases. The conflict resolution is prone to lost updates.

#### Write skews and phantoms

* **Write skew**: Two transactions read the same objects and then update some of those objects - different transactions may update different objects. Neither a dirty write nor a lost update if the two transactions are concurrently updating two different objects, but causes a race condition.
  * Ineffective solutions: Atomic single-object operations don't help because multiple objects are involved. Automatic detection and prevention requires true serializable isolation.
  * Possible solutions: Configure constraints for multiple objects with triggers or materialized views. Explicitly lock the rows that the transaction depends on.
* **Phantom**: A write in one transaction changes the result of a search query in another transaction.
  * If there is no object to which we can attach the locks, artificially introduce a lock object into the database.
  * Materializing conflicts: Take a phantom and turn it into a lock conflict on a concrete set of rows that exist in the database.
