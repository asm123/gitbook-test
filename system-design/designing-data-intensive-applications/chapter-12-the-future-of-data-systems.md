# Chapter 12: The Future of Data Systems

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
