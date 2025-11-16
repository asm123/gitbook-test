# Chapter 11: Stream Processing

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
