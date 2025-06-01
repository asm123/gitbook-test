# Designing Data Intensive Applications

### Chapter 3: Storage and Retrieval

#### Terminology

* Write amplification: One write to the database resulting in multiple writes to the disk over the course of the database's lifetime.

#### B-Trees vs LSM-Trees

**B-Tree**

1. Faster for reads
2. Higher write amplification than LSM-Tree
3. Has to overwrite several pages in the tree
4. Can lead to disk space fragmentatuon
5. Query response time is generally predictable
6. Compaction may not keep up with high write throughput -> unmerged segments on disk increase -> slowed down reads
7. Used by: [MySQL](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html), [PostgreSQL](https://www.postgresql.org/docs/current/btree-implementation.html), [Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/19/admin/managing-indexes.html#GUID-2CE1BB91-3EFA-450D-BD31-0C961549F0C2).

**LSM-Tree**

1. Faster for writes
2. Able to sustain higher write throughput due to:
   * lower write amplification
   * sequentially write compact SSTable files
3. Produces smaller files on disk
4. Lower storage overhead with leveled compaction
5. Compaction process can interfere with the performance of ongoing reads and writes -> response time for queries can sometimes be high
6. Each key exists in only one place in index -> can offer strong transaction isolation semantics
7. Used by: [Cassandra](https://docs.datastax.com/en/cassandra-oss/3.x/cassandra/dml/dmlManageOndisk.html), [CockroachDB](https://www.cockroachlabs.com/docs/v23.1/architecture/storage-layer#pebble), [Ressi for Google Spanner](https://static.googleusercontent.com/media/research.google.com/en/pubs/archive/46103.pdf).

#### Other indexing structures

* Secondary index
* Storing values within the index
  * Storing actual row - Clustered index - store indexed row within the index. MySQL InnoDB.
  * Reference to the row stored in a file called _heap file_ - unordered, append-only, tracking delete rows
  * Covering index / index with included columns - store some of a table's columns within the index.
* Multi-column index
  * Concatenated index - combine several fields into one key by appending one column to another
* Fuzzy index
  * Full-text search - Lucene
* In-memory database
  * Write a log of changes to disk by writing periodic snapshots to disk or by replicating the in-memory state to other machines.
  * Anti-caching - evicting the least recently used data from memory to disk when there is not enough memory, load back into memory when accessed again in future.
  * NVM - non-volatile memory
  *

#### OLTP and OLAP

* OLTP = online transaction processing = small number of records per query, fetched by key
* OLAP = online analytic processing = aggregate over large number of records
* Data warehouse contains a read-only copy of the data in all the OLTP systems.
  * ETL = Extract-Transform-Load - extract data from OLTP Databases, transform into analysis-friendly schema, clean up, load into data warehouse
* Schemas for analytics
  * Star schema / dimensional modelling
    * Fact table - each row represents an event that occurred at a particular time
    * Dimension table - table for foreign key references to other tables, typically columns in fact tables
  * Snowflake schema - similar to star schema, dimensions are further broken down into subdimensions.
    * More normalized than star schemas.

#### Column-oriented storage

* Store all the values from each column together instead of storing all the values form one row together.
* Each column is stored in a separate file.
* Examples: Cassandra, HBase
* Column compression
  * Bitmap encoding - column with n distinct values is condided as n separate bitmaps - one bitmap for each distinct value, one bit for each row.
  * Vectorized processing - bitwise operators on chunks of compressed column data
* Aggregation
  * Materialized views - actual copy of query results with materialized aggregates
  * Data cube / OLAP cube - grid of aggregates grouped by different dimensions

#### Further reading

* [Bitcask: A Log-Structured Hash Table for Fast Key/Value Data](https://riak.com/assets/bitcask-intro.pdf)
* [Dremel: Interactive Analysis of Web-Scale Datasets](https://research.google/pubs/pub36632/)
* [LevelDB Implementation Notes](https://github.com/google/leveldb/blob/main/doc/impl.md)
* [Bigtable: A Distributed Storage System for Structured Data](https://research.google/pubs/pub27898/)
* [The Log-Structured Merge-Tree (LSM-Tree)](https://www.cs.umb.edu/~poneil/lsmtree.pdf)
* [Hacking Lucene - The Index Format](https://massiveprogramming.blogspot.com/2014/10/hacking-lucene-index-format-hacker-labs.html)
* [Space/Time Trade-offs in Hash Coding with Allowable Errors](https://people.cs.umass.edu/~emery/classes/cmpsci691st/readings/Misc/p422-bloom.pdf)
* [The Advantages of an LSM vs a B-Tree](http://smalldatum.blogspot.co.uk/2016/01/summary-of-advantages-of-lsm-vs-b-tree.html)
* [Burst Tries: A Fast, Efficient Data Structure for String Keys](http://citeseer.ist.psu.edu/viewdoc/summary?doi=10.1.1.18.3499)
* [Fast String Correction with Levenshtein Automata](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.16.652)
* [Anti-Caching: A New Approach to Database Management System Architecture](http://www.vldb.org/pvldb/vol6/p1942-debrabant.pdf)
* [The Traditional RDBMS Wisdom Is (Almost Certainly) All Wrong](http://slideshot.epfl.ch/talks/166)
* [Dremel made simple with Parquet](https://blog.twitter.com/engineering/en_us/a/2013/dremel-made-simple-with-parquet)
* [The Design and Implementation of Modern Column-Oriented Database Systems](http://cs-www.cs.yale.edu/homes/dna/papers/abadi-column-stores.pdf)

### Chapter 7. Transactions

* A system has to deal with faults in order to be reliable. Implementing fault-tolerance mechanisms is a lot of work.
* Transactions have been the mechanisms of choice for simplifying the fault-tolerance issues.
  * Conceptually, all the reads and writes in a transaction are executed as one operation: either the transaction succeeds (commit) or it fails (abort, rollback). Application can safely retry if it fails.
  * Safety guarantees: Application is free to ignore certain potential error scenarios and concurrency issues because the database takes care of them.

#### Safety guarantees - ACID

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

#### Weak isolation levels

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

### Chapter 11: Stream Processing

#### Basics

* Batch processes assume bounded input. In reality, a lot of data is unbounded.
* Stream = data that is incrementally made available over time. Stream processing = processing every event as it happens.
* An event is a small, self-contained, immutable object containing the details of something that happened at some point in time.
* Examples: fraud detection systems consuming usage patterns of a credit card for detecting anomalies, trading systems consume price changes in a financial market, manufacturing systems monitoring the status of machines in a factory, military and intelligence systems tracking the activities of aggressors.
* Publish/Subscribe model:
  * Producer sends a message containing the event.
  * Related events are grouped into topics.
  * Consumers subscribe to the topics.
  * Messaging system notifies consumers about new events.
* Multiple approaches within the publish/subscribe model. Differentiate the approaches based on the following:
  * Handling slow consumers:
    * Drop messages
    * Buffer messages in a queue
      * What if the queue no longer fits in memory - crash? write messages to disk instead? impact on performance?
    * Apply backpressure / flow control = block the producer from sending more messages until the consumers empty the buffer.
  * Handling node outages:
    * Is minor message loss acceptable? - high throughput, low latency&#x20;
      * maybe ok for sensor readings, but not for applications needing exact counts.
    * Is durability important? - write to disk, replicate.
* Messaging modes: direct, message brokers.
* Direct from producer to consumer:
  * UDP / TCP / IP multicast, webhooks / callback URLs.
  * Application code has to be aware of the possibility of message loss - producers remember packets sent to retransmit on demand, assume producers and consumers to be online all the time. Remember problems in Chapter 8.

#### Message Brokers

* Google Cloud Pub/Sub, RabbitMQ, AMQP, etc.
* Database optimised for handling message streams, runs as a server, producers and consumers connect to it as clients.
* Producers write messages to the broker, consumers receive them by reading from the broker.
* Asynchronous, fault tolerant, broker handles the durability, slow consumers can be handled with configurable strategies.
* Unlike DBs,
  * transient messaging mindset - delete messages after successful delivery,
  * have shorter working queues,
  * indexing and selection comes in the form of subscribing to topics matching some pattern,
  * do not support arbitrary queries over the events.
* Multiple consumers reading messages in the same topic:
  * Load balancing: deliver each message to one of the consumers arbitrarily, all the consumers process the message.
    * 👍 Easy to parallelize processing of expensive messages.&#x20;
    * 👎 Consumers cannot consume independently.
  * Fan-out: deliver each message to all the consumers.&#x20;
    * 👍 Consumers consume independently.
  * Hybrid: groups of consumers subscribe to a topic, each group collectively receives all messages, only one consumer in a group receives the message.

**Traditional message brokers**

* AMQP, JMS.
* Clients ACK each message on finishing processing it. Handling lost acknowledgements requires atomic commit protocol (Chapter 9). Brokers can redeliver the message to another consumer if no ACK -> can lead to message reordering if using load balancing strategy.
  * Use a separate queue per consumer = no load balancing.
* Processing and ACKing messages is a destructive operation - processed messages are deleted on the broker - non-repeatable.

**Log-based message brokers**

* Apache Kafka, Amazon Kinesis Streams, Twitter's DistributedLog, Google Cloud Pub/Sub.
* Producer sends a message by appending it to the end of the log. Consumer receives by reading the log sequentially. If a consumer reaches the end of the log, it waits for a notification that a new message has been appended.
  * `tail -f` in Unix
* Scaling to higher throughput: partition the log.
  * Group partitions into topics.
  * Every message in each partition is assigned an offset for total order within the partition, no ordering guarantee across partitions.
  * Assign entire partitions to nodes in the consumer group.
    * Read the messages in a partition sequentially.
    * Number of simultaneous consumers = at most number of log partitions.
    * Head-of-line blocking.
* Sequential reading removes the need of tracking ACKs for each message by the broker, broker only needs to periodically record the offsets -> increased throughput.
* Analogous to single-leader DB replication - message broker is the leader, consumers are followers.
* Consumer failover is easier due to recorded consumer offsets.![](https://lh7-us.googleusercontent.com/YvCQ7NAAG1qmnhbZs0CyU6u8f3eWo3uyus-OLF4473r4NnAp_E2nd_iFx0GNfAUq_vnz1Y2rV_KWGYf7Jjy7MTquDORjZ0meIKS1mBHL9c8xFJBB7898qeEOpVd2RpDWBqyUwThK0pVE39LRNYBiNkw)
* Fault tolerance: replicate the log.
* Log is divided into segments, old segments are deleted or archived. A consumer too far behind will lose messages from deleted / archived segments.
  * Implemented as a circular / ring bounded-size buffer on disk.
  * Can typically keep a buffer of several days' or even weeks' worth of messages.
* Handling slow consumers:
  * Monitor how far a consumer is behind the head of the log and raise an alert accordingly. Can catch the lagging consumer up with human intervention.
  * Only the lagging consumer is affected, other consumers work normally.
* Repeatable transformation:
  * Consumer offset moves forward, but is under consumer's control -> makes replaying old messages as easy - start a copy of a consumer with a chosen offset and write the output to a different location to reprocess.
  * Ease of experimentation and recovery.

\|---|---|\
|Traditional messaging is suitable when|Log-based messaging is suitable when|\
\|- message processing is expensive\
\
\- needs to be parallelized\
\
\- message ordering is not so important|- message throughput is high\
\
\- message processing is fast\
\
\- message ordering is important|

#### Databases and Streams

**Similarities**

* Replication log is a stream of DB write events.
* State machine replication principle in total order broadcast is an event stream - if every event represents a write to the DB and every replica processes the same events in the same order, then the replicas will all end up in the same final state.

**Applying ideas from messaging and streams to databases**

* **Change Data Capture (CDC)**:&#x20;
  * Changes written to a database are made available as a stream as they are written, so that all the changes are also reflected asynchronously in the derived data systems (consumers of the stream).
*

    ![](https://lh7-us.googleusercontent.com/WdDruvsOcWSMbEuiEGa1Onu4gn2CT2r6shgAmZdMxliyPjlhVD6glKq6p3SAvE4Ak72GoB-2iTpb6K4LuxUDPfTYFcwYYHkpcW9h61JXR06XIpTRMlg7IWIWZ-TyA4rZxwp8r94ZrOZclvvZHhuedr4)

    * One DB is the leader, others are followers. Log-based message broker transports the change events from leader (source db) to followers (derived data systems), preserving the ordering of messages.
    * Implementation options:
      * Database triggers: registering triggers to observe all changes to data tables and add entries to a changelog table; fragile, incur performance overhead, problems with replication lag (Chapter 5). Databus (LinkedIn), Wormhole (FB), Sherpa (Yahoo).
      * Parse the replication log: handling schema changes can be challenging. MySQL + binlog, MongoDB oplog, Kafka Connect.
      * Change streams as a first-class interface:
        * Queries subscribe to notifications when the results of a query change, make change feed available to applications, log of tuples that committed transactions as a stream.
      * Maintaining the source DB: consistent snapshots, tombstones, log compaction.
* **Event sourcing**:
  * Originally developed in the domain-driven design (DDD) community.
  * Take the log of events representing the data written to the system and transform it into an application state that is suitable for showing to a user.
    * For example, "student cancelled their course enrollment" instead of "one entry was deleted from the enrollments table, one cancellation reason was added to the student feedback table".
  * The transformation has to be deterministic so that replaying the event log allows reconstructing the current state of the system.
  * System has to store all raw events forever and reprocess the full event log whenever required.
* Command -> event -> fact:
  * A command can fail. Application must first validate that it can execute the command. A user trying to book a seat on an airplane.
  * If the validation is successful and the command is accepted, it becomes an event. An event is durable and immutable. A seat has been reserved for a particular user.
  * When the event is generated, it becomes a fact. A user held a reservation for a particular seat, even if it may have been cancelled later (a separate event).
* A consumer cannot reject an event - it is immutable and already happened.
* Synchronous validation - a serializable transaction to atomically validate the command and publish the event.
* Asynchronous validation - linearizable storage using total order broadcast - an event for tentative intent to claim the seat, another event for validation of reservation.

\|---|---|\
|Change Data Capture|Event Sourcing|\
|application uses the db in a mutable way - updating and deleting record at will|application logic is explicitly built on the basis of immutable events from an event log, event store is append-only|\
|entries in the change log reflect low level state change at db level|entries in the event log reflect things that happened at the application level|\
|event represents the entire new version of a record, reflecting the most recent event for a key|event represents the intent of a user action, not the mechanics behind it|\
|log compaction can discard old entries|need the full history of events to reconstruct the final, log compaction works to always know the recent state|

**States, streams and immutability**

* Mutable state and append-only log of immutable events do not contradict each other, they are two sides of the same coin.
*

    ![](https://lh7-us.googleusercontent.com/1mDyZiYi_MTkDdB_iiFqNa8Vn9aspU_3JmSyBtqFLihP9Vv4XOVJBKLAfcT8fW2bdX2IFoOicObPjudORSmgnjD9JMLEmZhKJVt7RTdl4Zo3oANZVyWjXpA7vG8l23Jz2Ie8oMHZqsvLCWP4cIudkro)
*

    ![](https://lh7-us.googleusercontent.com/DsPznQIH4PNfYAoTCNBhn_sbynCQ03gRNEDjrL5sok-zB4lJtphYsJOcRKVdQp-ELFpuSLxViwyYznP44td2_hQEcSH4Er2PZWWL8hbAirlfqCngwnOoKVENz37ncpXYy8c5SAChqmt5pTzkVllYGwA)
* Immutability:
  * is good:
    * Diagnosis of and recovery from the consequences of a deployed buggy code are much easier than destructive overwriting for correction.
    * Captures more information than just the current state, e.g., use the act of emptying the shopping cart for analytics and future recommendations.
  * but also…
    * GDPR requires actually deleting data and not pretending to be deleting with tombstones.
    * Immutability history can grow large, fragmented -> performance overhead of compaction and garbage collection.
* 👍 Separating mutable state from immutable event log:
  * Helps derive different read-oriented views.
  * An explicit translation step from an event log to a db eases evolving an application over time - can use the event log to build a separate read-optimized view, run old and new systems side-by-side instead of complicated schema migration until the old system can be safely shut down.
  * Command query responsibility segregation offers a lot of flexibility away from the complexities of schema design, indexing and storage.
  * Translate data from write-optimised event log to read-optimized application state to cater to both write-heavy and read-heavy use cases.
* 👎 Concurrency control:
  * A user writing to source log may read from a log-derived view and find that their write has not yet been reflected in the read view.
  * Update read view synchronously - needs a transaction to combine the writes into an atomic unit -> need to keep the event log and read view in the same storage system, or a distributed transaction.
  * Much of the need for multi-object transactions comes from a single user action requiring data change in several places. Design an event such that it is self-contained - e.g., both mutable state and immutable logs reside in the same partition -> serial write to a partition.

#### Processing streams

* What to do with a stream once you have it?
  * write to a queryable storage system - db, cache, index
  * push the events to users - emails, push notifications, real-time dashboards
  * produce streams - derived streams
* For the rest of the chapter, we will discuss derived streams.

**Uses of stream processing**

\|---|---|---|\
|Complex event processing (CEP)|Allows specifying rules using SQL, GUI to search for certain event patterns in a stream. When a match is found, the processing engine emits a "complex event" with details of the detected event pattern.\
\
\
\
Queries are stored long-term, events from the streams continuously flow past them.|TIBCO StreamBase, Samza.|\
|Analytics on streams|Aggregations and statistical metrics over a large number of events, e.g., rolling average, rate of an event, detect trends.\
\
\
\
Windowing - aggregating over fixed time intervals.\
\
\
\
Use probabilistic algorithms, e.g., Bloom filters (not an approximation, merely an optimization), HyperLogLog, percentile algorithms.|Frameworks: Apache Storm, Spark Streaming, Flink, Kafka Streams. Hosted services: Google Cloud Dataflow, Azure Stream Analytics.|\
|Materialized views|Derived data systems, maintaining application state in event sourcing.\
\
\
\
Requires all events over an arbitrary time period minus the ones discarded by log compaction, unlike stream analytics.|Samza, Kafka.|\
|Search on streams|Documents run past stored full-text search queries, similar to CEP.\
\
\
\
Can be optimized by indexing queries and documents to narrow down the set of matching queries.\
\
\
\
Media monitoring services, real estate property tracking.|Elasticsearch percolator.|\
|Message passing and RPC|Crossover between RPC-like systems and stream processing.\
\
\
\
Distributed RPC - queries are farmed out to nodes that process event streams, queries are interleaved with events from input streams, results are aggregated and sent back to the user.|Apache Storm|

* Comparing with batch jobs:
  * Similar: patterns for partitioning and parallelization.&#x20;
  * Different: no sort-merge joins, restarting the job from the beginning is not viable.
* Comparing with actor frameworks:
  * Similar: mechanism for services to communicate.
  * Different:
    * Actor frameworks manage concurrency and distributed execution of communication modules, stream processing manages data.
    * Communication between actors is ephemeral and one-to-one, event logs are durable and multi-subscriber.
    * Actors can have cyclic request/response patterns, stream processing pipelines are acyclic.

**Reasoning about time**

* Which time to use?
  * Event time: Timestamp when the event actually occurred, embedded in the event.&#x20;
  * Allows the processing to be deterministic.
  * Processing time: Local system clock timestamp on the processing machine when the event is received and processed.&#x20;
  * Simpler, reasonable if the delay between event creation and processing is negligibly short.
  * Breaks down if there is a significant processing lag.
  * Confusing event time and processing time leads to bad data.
* Knowing when you are ready: When defining windows in terms of event time, one can never be sure if all the events for a particular window are received or if there are more to come.
* Straggler events - events that arrive after the window has already been declared complete.
  * Ignore,&#x20;
  * Publish a correction and retract the previous input,
  * Send a special message declaring that there will be no more messages with event timestamp < t.
* Whose clock are you using anyway?
  * Events may be buffered at several points in the system - locally on the device, on the server - making the events appear as extremely delayed stragglers.
  * Device clocks can be unreliable. To adjust, log three timestamps:
    * time at which the event occurred according to the device clock,
    * time at which the event was sent to the server according to the device clock
    * time at which the event was received by the server according to the server clock
      * subtract event sent time from this to estimate the offset between device and serve clocks, and apply the offset to event occurred timestamp
* Types of time windows:
  * Tumbling window: fixed length, fixed boundary, every event belongs to exactly one window.
  * Hopping window: fixed length, fixed boundary, windows can overlap.
  * Sliding window: fixed length, no fixed boundary.
  * Session window: no fixed length, no fixed boundary, depends on user active / inactive time.

**Stream joins**

* Types of joins:
  * Stream-stream join - window join:
    * Maintain state to join events from two different streams within a suitable window.
  * e.g., searches and clicks.
  * Stream-table join - stream enrichment:
    * Enrich the events in steam with information from the database.
    * DB lookup can be by query, or by loading a copy into the stream processor to avoid network round-trip. Local copy to be kept up to date with CDC -> join between two streams.
    * Effectively a stream-stream join with the window for CDC reaching all the way back to the beginning of time.
* Table-table join - materialized view maintenance:
  * Streams update the materialized view for a query that joins two tables. e.g., Twitter timeline cache for stream of tweets and follow/unfollow events.
* Time-dependence of joins:&#x20;
  * Join becomes non-deterministic if the order of events across streams is undetermined.
  * Slowly changing dimension
  * Determinism can be achieved by assigning a unique identifier to a particular version of the joined record, at the cost of losing log compaction.

**Fault tolerance**

* Batch approach to fault tolerance is exactly-once / effectively-once:&#x20;
  * Failed jobs can be started again on another machine and outputs maid visible only successful completion.&#x20;
  * It looks as if nothing had gone wrong even if some tasks failed because every input record appears to have been processed only once.
* Exactly-once for stream processing:
  * Microbatching:
    * Break the stream into small blocks and treat each block like a miniature batch process.
    * Batch size defines a tumbling window. Trade-offs between scheduling and coordination, and output delay depending on batch size.
    * Exactly-once semantics within a batch. But as soon as the output leaves the stream processor, output of a failed batch cannot be discarded.
    * Spark Streaming.
* Checkpointing:
  * Periodically generate rolling checkpoints of state and write to durable storage.
  * Crashed jobs can restart from the most recent checkpoint, and discard outputs failed between last checkpoint and crash.
  * Exactly-once semantics with side-effects on re-run similar to microbatching.
  * Apache Flink.
* Atomic commit:
  * All-or-none.
  * Stream processing framework handles everything including managing state changes, messaging and side-effect transactions.
  * Google Cloud Dataflow, VoltDB, Kafka.
* Idempotence:
  * An operation should have the same effect as if it was performed only once, even if performed multiple times.
  * Can be made idempotent with extra metadata, e.g., include a monotonically increasing offset in each message.
  * Assumes that a failed task must replay messages in the same order on restart, deterministic processing, no concurrent updates.
* Rebuilding state after a failure:&#x20;
  * Periodically persist the state to a storage and replicate. Storage can be local, remote, another topic.
  * Or don't! Replay the input stream and recompute aggregations.
  * Choice depends on performance characteristics and underlying infra, e.g., disk access latency vs n/w delay.

### Chapter 12: The Future of Data Systems

* Reliability, scalability and maintainability have been recurring themes throughout the book. This chapter brings all those ideas together and builds on them.
* Author's personal opinions and speculations about the future.

#### Data Integration

* Vendors of software products would tell you what their product is best suited for, but reluctant to admit where it is poorly suited.
* No one piece of software is likely to be suitable for all the different circumstances for which data is used.

**Combining specialized tools by deriving data**

* As the number of different representations of data increases, the integration problem becomes harder.
  * db, search index, analytics, cache, change notifications.
* Something not useful for one person may be a central requirement for someone else. Need for data integration becomes apparent only when dataflows across an entire organization are considered.
* Reasoning about dataflows - be clear about inputs and outputs, where is it written first, which format.
* Distributed transactions or derived data or - reading your own writes guarantee vs eventual consistency.
* Constructing a totally ordered event log has limitations due to log partitioning, geographically distributed datacenters, etc. Deciding on a total order of events is equivalent to consensus, still an open research problem to design consensus algorithms that scale beyond a single node.
* Capturing causal dependencies in the absence of total order: sequence number ordering, log an event to record the state of the system, conflict resolution.

**Batch and stream processing**

* Goal of data integration is to make sure that data ends up in the right form in all the right places. Both batch and stream processing frameworks are useful.
* Reprocessing existing data provides a good mechanism for maintaining a system, evolving it to support new features and changed requirements. Derived views allow gradual evolution without needing a sudden switch.
* Lambda architecture:
  * Incoming data is recorded by appending immutable events to an always growing dataset.
  * Derive read-optimized views by running two different systems in parallel.
    * stream processing system to process events as they arrive and produce an approximate update to the view
    * batch processing system to reprocess the same set of events and produce a corrected version of the view
  * Drawbacks:
    * Have to maintain the same logic to run in two different frameworks.
    * Need to merge outputs from both the systems to respond to user requests.
    * Reprocessing can be expensive on large datasets.
* Unifying batch and stream processing:
  * Same processing engine to handle both stream and batch use cases.
  * Exactly-once semantics for stream processors.
  * Windowing by event time instead of processing time for accurate reprocessing.

#### Unbundling databases

Unix and relational DBs have approached the information management problem with very different philosophies.

* Unix: logical, low-level hardware abstraction; pipes and files.
* Relational DBs: high-level abstraction to hide the complexities of data structures on disk, concurrency, crash recovery, etc; SQL and transactions.

Which approach is better?

**Composing data storage technologies**

There are parallels between the features that are built into databases and the derived data systems being built with batch and stream processors.

* Creating an index: DB scans over a consistent snapshot -> pick out field values to be indexed -> sort -> write out the index -> keep the index up to date.
  * Similar to setting up a new follower replica, bootstrapping CDC.
* Meta-DB of everything:
  * Batch and stream processors are like elaborate implementations of triggered, stored procedures and materialized view maintenance routines.
  * Derived data systems are like different index types.
* Future avenues to build cohesive storage and processing systems:
  1. Federated databases: unifying reads - unified query interface to a wide variety of systems.
  2. Unbundled databases: unifying writes - synchronizing writes across a wide variety of systems.
     * Much harder engineering problem than 1.
     * Asynchronous event log with idempotent writes may be more robust and practicable than distributed transactions to make it work; loose coupling between various components.
     * Goal is not to compete with individual DBs, but to allow combining different DBs to achieve good performance for a wider range of workloads than is possible with a single piece of software.
     * Advantages of unbundling and composition make sense only when no single piece of software satisfies all the requirements.
     * What is missing:
       * Still no unbundled-database equivalent of Unix shell to compose storage and processing systems in a simple and declarative way, e.g., `mysql | elasticsearch`
       * Ease of precomputing and updating caches with declaratively specifying materialized views.

**Designing applications around dataflow**

* Database inside-out approach - unbundling databases by composing specialized storage and processing systems with application code.
  * This concept has a lot of overlap with dataflow languages, functional reactive programming language and logic programming languages.
  * Spreadsheets already have dataflow programming capabilities. Today's data systems additionally need to be fault tolerant, scalable, store data durably, integrate a wide variety of technologies.

| **Present**                                                                                                                                                                                                                                             | **Future**                                                                                                                                                                                    | **Challenges**                                                                                                                                                                      |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>Most web applications today are deployed as state services, DB acts as a passive mutable shared variable.<br><br><br>It makes sense to have some parts of a system to specialize in durable data storage and others in running application code.</p> | Application code could respond to state changes in one place by triggering state changes in another place. Same for DB: changes to primary DB trigger creation / changes to derived datasets. | Maintaining derived data is not the same as asynchronous job execution - need stable ordering of state changes, fault tolerance, which are not guaranteed by the messaging systems. |
| Microservices architecture with loose coupling.                                                                                                                                                                                                         | Replace synchronous network request to another service with a query to a local database that responds to updates in dependencies (similar to changes to formulae in spreadsheets).            | Need to handle time dependence in joins.                                                                                                                                            |

#### Observing derived state

* Caches, indexes, materialized views shift the boundary between read path and write path: a trade-off between the amount of work that needs to be done at write time and the amount that needs to be done at read time.
* Servers can push state changes to stateful and offline-capable clients. e.g., EventSource API, websockets.
* Extending the write path all the way to the end user with end-to-end event streams, e.g., publish subscribe model.
* Logging read events for causal dependencies and data provenance across a system.
* Distributed execution of complex queries that need to combine data from several partitions - treat queries as streams and leverage existing stream processing engines.

#### Aiming for correctness

* Correctness - programs have well-defined and understood semantics even in the face of various faults.
* Traditional transactions approach is here to stay, but it may not be the only way to make applications correct and resilient to faults:
* The end-to-end argument of databases:
  * Immutability, exactly-once, duplicate suppression, idempotency and low-level features are useful, but not sufficient by themselves. We need to consider end-to-end flow of the request.
  * It would be nice to wrap up the remaining high-level fault-tolerance machinery in an abstraction to provide application-specific e2e correctness properties, while maintaining good performance and operational characteristics.
  * Enforcing constraints: two people cannot book the same seat on a flight, account balance should not go negative.
    * Need consensus, may be possible with log-based messaging that outputs success or rejection on the stream by reading a log.
  * Timeliness and integrity:
    * Consistency conflates timeliness and integrity, worth considering them separately.
    * Integrity >>>> timeliness.
    * ACID transactions provide both timeliness and integrity. Event-based dataflow systems have integrity at the core (e.g., exactly-once semantics), timeliness is not guaranteed.
    * Possible to preserve integrity in stream processing  without requiring distributed transactions and atomic commit protocol through a combination of mechanisms: write operation as a single immutable message (event sourcing), derive all other state updates from the single message, unique id passed through the entire processing stack.
    * Loosely defined constraints may be acceptable in business workflows keeping integrity intact, e.g., overbooking an airplane, also ensuring apology and compensation when demand > supply.
    * Coordination-avoiding data systems are fine if they offer better trade-offs.
* Trust, but verify:
  * Maintain integrity in the face of software bugs.
  * Don't just blindly trust that everything is working, e.g., are backups actually restorable?
  * Design for auditability. Maybe even develop self-auditing systems that continually check their own integrity. Blockchain?

#### Doing the right thing

* Beware of these too while building systems:
  * Bias and discrimination in predictive analytics, accountability of automated decision making, feedback loops, user privacy violation, consent, freedom of choice.
