# Chapter 1: Reliable, Scalable and Maintainable Applications

Many applications today are data-intensive as opposed to compute-intensive.

Different data systems such as databases, caches, search indexes, stream processing, batch processing are optimized for different use cases. No single tool can meet all of the data processing and storage needs of an application.&#x20;

Three concerns that are important in most software systems:

1. Reliability - system should continue to work correctly
2. Scalability - system should deal with growth reasonably
3. Maintainability - people should be able to work on the system productively

## Reliability

> We can understand reliability as meaning, roughly, "continuing to work correctly, even when things go wrong".

A fault occurs when a component of the system deviates from its spec. A failure occurs when the system as a whole stops working.

A fault-tolerant or resilient system anticipates and copes with faults. But not all faults, rather certain types of faults such as hardware faults, software errors and human errors.

* Hardware faults: hard disk crashes, faulty RAM, etc.&#x20;
  * First response is to add redundancy to individual hardware components to reduce the _failure rate_ of the system.&#x20;
  * Combining redundancy with software fault-tolerance techniques can tolerate complete loss of entire machines.
* Software errors: Systematic errors within the system.
  * Harder to anticipate, correlated across nodes, may be dormant for a long time until triggered by an unusual set of circumstances.
  * No quick solution. Can be eased with careful consideration of assumptions and interactions, thorough testing, monitoring.
* Human errors:&#x20;
  * Minimize opportunities for error with well-designed interfaces.
  * Provide sandbox environments.
  * Thorough testing at all levels.
  * Allow quick and easy recovery from errors, e.g., easy rollbacks through config changes, gradual rollouts.
  * Set up telemetry.

## Scalability

Scalability is a system's ability to cope with increased load, defined by it's **load parameters** (e.g., requests per second, ratio of reads to writes in a database) and **performance** when the load increases (e.g., response time, throughput).

> Investigating performance when the load increases:
>
> * When you increase a load parameter and keep the system resources unchanged, how is the performance of your system affected?
> * When you increases a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?

Batch processing systems care about throughput, online systems care about response time.

**Response time**:

* Response time is what the client sees, latency is the duration that a request is waiting to be handled.
* Response time is not a single number, but a distribution of value you can measure.
* Describing this distribution as average response time, i.e., arithmetic mean of all response times is not a very good metric because it doesn't tell you how many users actually experienced that delay. Use **percentiles** instead.
* Commonly reported percentiles: p50 (median), p95, p99, p999 (99.9th percentile) for SLOs and SLAs
* Tail latencies = high percentiles of response times are important because they directly affect users' experience of the service. Reducing response times at very high percentiles is difficult because they are easily affected by random events outside of your control.

**Achieving scalability**:

* Vertical scaling = scaling up = moving to a more powerful machine
* Horizontal scaling = scaling out = distributing the load across multiple smaller machines
* In reality, a good architecture involves a pragmatic mixture of horizontal and vertical scaling.
  * Distributing stateless services across multiple machines is fairly straightforward. Taking stateful data systems from a single node to a distributed setup can introduce a lot of additional complexity.&#x20;

> An architecture that scales well for a particular application is built around assumptions of which operations will be common and which will be rare—the load parameters. If those assumptions turn out to be wrong, the engineering effort for scaling is at best wasted, and at worst counterproductive.

## Maintainability

Principles to design software so that the pain of its maintenance is minimized:

* **Operability** - making life easy for operations teams
  * Visibility into the runtime behaviour through monitoring, good support for automation and integration with standard tools, good documentations, playbooks, self-healing when appropriate.
* **Simplicity** - managing complexity so that new engineers can understand the system
  * Making a system simpler does not necessarily mean reducing its functionality, it can also mean removing accidental complexity, i.e., the complexity not inherent in the problem that the software solves, but arising only from the implementation.
  * Use of good abstractions can remove accidental complexity by hiding a great deal of implementation detail behind a clean, simple-to-understand facade.
* **Evolvability** - make it easy for engineers to make changes to the system, adptable to changing requirements
  * Agility on the level of a larger data system, TDD, refactoring.

