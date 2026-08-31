# 20-Messaging-and-Distributed-Systems

# 15-Kafka-Architecture

> Deep production architecture reference for AWS/EKS, Kubernetes, enterprise messaging and senior-level system design.

# 1. 1. Architecture Objectives

A production Kafka architecture must balance durability, availability, throughput,
latency, scalability, operability, security and cost. Start from business
requirements rather than choosing broker counts or partition counts first.

# 2. 2. Reference Enterprise Architecture

```text
                         Applications
                    /        |        \
               Producers   Services   CDC
                    \        |        /
                         Kafka Cluster
                              |
          +-------------------+-------------------+
          |                   |                   |
        Broker-1            Broker-2            Broker-3
          |                   |                   |
        AZ-1                AZ-2                AZ-3
          |                   |                   |
       Storage             Storage             Storage
          +-------------------+-------------------+
                              |
                 Consumer Groups / Streams
                   /          |          \
                OLTP       Analytics     Search
```

# 3. 3. Kafka Cluster Mental Model

Think about Kafka in four planes:

```text
Client plane      -> producers and consumers
Data plane        -> brokers and partition logs
Metadata plane    -> controllers / KRaft metadata quorum
Operations plane  -> security, monitoring, automation and recovery
```

A production incident can originate in any plane.

# 4. 4. Broker

A broker stores partition replicas and serves produce/fetch traffic.
Broker capacity is constrained by CPU, memory, storage and network.

# 5. 5. Controller

The controller/metadata layer manages cluster metadata and leadership-related
coordination. Modern Kafka architecture can use KRaft rather than ZooKeeper.

# 6. 6. KRaft

KRaft uses a Kafka metadata quorum. Production designs should explicitly
separate the concepts of:

```text
controller quorum
broker capacity
partition replicas
```

They solve different problems.

# 7. 7. Controller Quorum

For production, controller quorum design must survive the failure scenarios
defined by the availability requirement. Avoid treating a single controller as
a production HA architecture.

# 8. 8. Broker Roles

Kafka deployments can combine or separate controller and broker roles depending
on topology, scale and operational requirements. The choice should be
intentional.

# 9. 9. Topic

A topic is a logical event stream. Architecture decisions around a topic
include ownership, schema, partitions, replication, retention, security and
consumer contracts.

# 10. 10. Partition

A partition is an ordered append log and the fundamental unit of Kafka
parallelism. Partition design therefore affects both performance and
correctness.

# 11. 11. Partition Placement

Replicas should be distributed across independent failure domains where
possible. In AWS, availability zones are a primary failure-domain consideration.

# 12. 12. Leader and Followers

Each replicated partition has a leader for normal client traffic and follower
replicas that replicate the leader log. Leadership can change after failure.

# 13. 13. Replication Factor

Replication factor defines how many replicas exist for each partition.

```text
RF=3
P0 -> Broker A
   -> Broker B
   -> Broker C
```

RF is redundancy, not parallelism.

# 14. 14. ISR

The in-sync replica set represents replicas sufficiently caught up to the
leader according to Kafka's replication rules. ISR health is a key durability
signal.

# 15. 15. Min ISR

Min ISR can prevent successful writes under certain acknowledgement
configurations when too few replicas are in sync. It is a safety control, not a
replacement for capacity planning.

# 16. 16. Unclean Leader Election

Allowing an out-of-sync replica to become leader can trade potential data loss
for availability. This is a deliberate reliability trade-off and should never
be enabled casually for critical data.

# 17. 17. Failure Matrix

```text
Failure              Primary concern
------------------------------------------------
consumer             rebalance / lag
broker                leader / replica recovery
disk                  partition replica loss
AZ                   replica availability
controller quorum     metadata availability
region                DR
```

# 18. 18. Multi-AZ Architecture

```text
AWS Region
|
+-- AZ-1 -> Broker-1
|
+-- AZ-2 -> Broker-2
|
+-- AZ-3 -> Broker-3
```

Replica placement should prevent a single AZ failure from removing the required
redundancy for critical partitions.

# 19. 19. Rack Awareness

Kafka can use rack/failure-domain information to influence replica placement.
In AWS this concept is commonly mapped to availability zones through the
deployment configuration.

# 20. 20. Client Traffic

```text
Producer
   |
bootstrap
   |
metadata
   |
specific broker
```

Kafka clients need reachable broker addresses after metadata discovery.

# 21. 21. Advertised Listeners

Advertised listener configuration is critical. A client can successfully reach
a bootstrap endpoint and still fail when connecting to the broker addresses
returned in metadata.

# 22. 22. Internal Listeners

Internal applications should normally use private, low-latency networking and
a dedicated internal listener design.

# 23. 23. External Listeners

External clients require explicit design for DNS, TLS, authentication, broker
advertisement and network reachability. Kafka is not simply an HTTP service
that can be hidden behind one generic load balancer.

# 24. 24. Load Balancer Design

A load balancer may assist bootstrap or external connectivity, but every broker
must ultimately be reachable according to Kafka's advertised metadata model.

# 25. 25. DNS

Stable DNS is important for broker access. Validate DNS resolution from every
client network before production.

# 26. 26. Security Boundary

Production Kafka should be treated as a critical data platform. Security must
cover network access, authentication, authorization, encryption and auditing.

# 27. 27. TLS

Use TLS for client and broker/controller communication where required by the
security model. Test certificate rotation before production.

# 28. 28. Authentication

Common authentication mechanisms include SASL and mTLS depending on the
deployment. Select one standard and automate credential/certificate lifecycle.

# 29. 29. Authorization

Use least privilege for producers, consumers and administrators. A producer
normally needs write access to selected topics, not cluster-wide administration.

# 30. 30. ACL Architecture

Design ACLs around:

```text
principal
topic
operation
consumer group
cluster resources
```

Review them as code where possible.

# 31. 31. Quotas

Quotas can limit noisy tenants and prevent one producer or consumer from
consuming disproportionate broker resources.

# 32. 32. Multi-Tenancy

Tenant isolation can use topic naming, ACLs, quotas and consumer-group
controls. Stronger isolation may require separate clusters.

# 33. 33. Storage Architecture

Kafka's storage design must account for:

```text
retained bytes
replication
recovery headroom
rebalancing
filesystem overhead
```

Do not size disks only from average daily traffic.

# 34. 34. Storage Formula

Approximate logical retention:

```text
bytes/sec × retention_seconds
```

Then multiply for replication and add operational headroom.

# 35. 35. Example Capacity

For 20 MB/s of logical ingress and 24-hour retention:

```text
20 × 86,400 = 1,728,000 MB
≈ 1.73 TB logical
```

With RF=3:

```text
≈ 5.18 TB
```

before additional overhead and headroom.

# 36. 36. Disk Headroom

Keep enough free capacity for replica recovery, retention variance and
rebalancing. Running Kafka disks near full capacity creates operational risk.

# 37. 37. Page Cache

Kafka benefits heavily from the operating system page cache. Do not allocate
the entire machine's memory to JVM heap.

# 38. 38. JVM

Monitor heap and garbage collection while remembering that broker performance
also depends heavily on off-heap memory, page cache and network buffers.

# 39. 39. CPU

CPU is consumed by request processing, compression, TLS, replication and
other broker activities. Measure the actual workload before right-sizing.

# 40. 40. Network

Kafka can be network intensive because client traffic and replica traffic both
consume bandwidth.

# 41. 41. Cross-AZ Cost

Multi-AZ resilience can introduce cross-AZ data transfer. In AWS, include this
in the architecture cost model rather than treating it as an afterthought.

# 42. 42. Partition Count

Choose partition count from:

```text
throughput
consumer parallelism
ordering
future growth
recovery time
```

Too few partitions limit scale; too many increase operational overhead.

# 43. 43. Partition Count Is Not Free

Partitions increase metadata, file descriptors, recovery work and controller
management overhead. Over-partitioning is a production architecture problem.

# 44. 44. Replication Factor vs Partitions

```text
partitions -> parallelism
replication -> redundancy
```

Changing one does not automatically solve the problem represented by the other.

# 45. 45. Key Design

Use stable keys when per-entity ordering is required.

```text
key = order_id
```

All events for an order can then be routed consistently to a partition under
the producer's partitioning behavior.

# 46. 46. Hot Keys

A high-volume key can create a hot partition. Mitigation requires a trade-off
between distribution and ordering guarantees.

# 47. 47. Topic Lifecycle

Every production topic should have:

```text
owner
purpose
schema
partition strategy
replication
retention
security
SLO
deletion policy
```


# 48. 48. Retention

Retention determines how long records remain available. Longer retention
improves replay capability but increases storage and recovery requirements.

# 49. 49. Compaction

Compacted topics are useful for latest-state patterns. Compaction is different
from ordinary time/size deletion and must be designed around key semantics.

# 50. 50. Producer Architecture

```text
Application
 |
Producer client
 |
serialization
 |
partition selection
 |
broker
```

The producer owns correctness decisions around keys, serialization,
acknowledgement, retry and idempotency.

# 51. 51. Producer Batching

Batching reduces request overhead and can improve throughput. Excessive batching
can increase latency.

# 52. 52. Compression

Compression reduces network and storage use at the cost of CPU. Benchmark
compression choices against the real message distribution.

# 53. 53. Producer Acknowledgements

Acknowledgement configuration determines how strongly the producer waits for
broker-side confirmation. Select it according to durability and latency
requirements.

# 54. 54. Producer Idempotency

Idempotent producer behavior can reduce duplicate records caused by retries
within its supported semantics. It does not solve every end-to-end duplicate
problem.

# 55. 55. Producer Retry

Retries must be bounded and paired with timeouts. A producer that retries
forever can hide an outage and increase resource pressure.

# 56. 56. Producer Failure

When an acknowledgement is lost, the producer may not know whether the record
was accepted. Stable event IDs and idempotent behavior reduce ambiguity.

# 57. 57. Consumer Architecture

```text
Kafka partition
 |
consumer
 |
business processing
 |
side effect
 |
offset commit
```

The commit boundary must match the required delivery semantics.

# 58. 58. Consumer Group

A group shares partition ownership. Scaling a group is therefore constrained
by the number of partitions.

# 59. 59. Consumer Rebalance

Membership changes can trigger partition reassignment. Frequent rebalances
usually indicate instability, deployment churn or timeout problems.

# 60. 60. Consumer Lag

Lag should be monitored per group, topic and partition. Pair lag with message
age and processing rate to understand business impact.

# 61. 61. Consumer Commit

Committing after successful processing supports an at-least-once pattern.
Crashes before commit can cause redelivery, so side effects must be duplicate-safe.

# 62. 62. Idempotent Consumer

A durable event ID or business key can be used to prevent repeated business
effects.

# 63. 63. Inbox Pattern

```text
Kafka event
 |
DB transaction
 +-- processed_event
 +-- business update
 |
commit
 |
offset commit
```

This separates message redelivery from duplicate business effects.

# 64. 64. Outbox Pattern

```text
DB transaction
 +-- business update
 +-- outbox event
 |
outbox publisher
 |
Kafka
```

This solves a common database/Kafka dual-write problem.

# 65. 65. Exactly-Once Boundary

Kafka transactions can provide stronger guarantees for Kafka-to-Kafka
processing, but an external database or API side effect is outside the Kafka
transaction unless an explicit integration pattern is used.

# 66. 66. Retry Architecture

Kafka does not provide RabbitMQ-style queue retry semantics by simply turning
on a retry flag. Retry is commonly modeled using application logic and
additional topics.

# 67. 67. Retry Topics

```text
orders
orders.retry.1m
orders.retry.10m
orders.dlq
```

The exact strategy should be based on error classification and ordering
requirements.

# 68. 68. Poison Records

A poison record can repeatedly fail and block later records in its partition
if strict ordering is preserved. Error handling must explicitly address this.

# 69. 69. DLQ/Error Topic

An error topic should have an owner, retention policy, alerting and a safe
replay process.

# 70. 70. Replay

Replay is powerful but dangerous. Validate idempotency, ordering and downstream
capacity before replaying large ranges.

# 71. 71. Downstream Protection

Consumer scaling must be bounded by database, API and cache capacity. Kafka
can absorb backlog, but it cannot make an unhealthy dependency infinitely
scalable.

# 72. 72. Backpressure

Backpressure can be created by controlling consumer concurrency, polling,
processing and downstream calls. Queueing is not a substitute for dependency
capacity.

# 73. 73. Autoscaling

Scale consumers using workload signals such as lag, message age and processing
rate, while applying upper bounds and dependency protection.

# 74. 74. KEDA

KEDA can be used for event-driven consumer scaling. Production designs must
still protect downstream systems and avoid oscillating scale decisions.

# 75. 75. Kubernetes Architecture

Kafka is stateful. Use persistent identities, persistent storage, topology
spreading and a Kafka-aware operator or lifecycle system where appropriate.

# 76. 76. StatefulSet

StatefulSet provides stable identities and storage association but does not
itself understand Kafka leadership, replication or safe upgrades.

# 77. 77. Kafka Operator

A Kafka-aware operator can automate cluster lifecycle, topics, users,
configuration and upgrades. Operator automation does not replace architecture
or capacity decisions.

# 78. 78. Strimzi

Strimzi is a common Kubernetes Kafka operator ecosystem. Before production,
validate supported Kafka, Kubernetes and storage versions and operational
features.

# 79. 79. EKS Architecture

A typical EKS deployment:

```text
EKS
 |
+-- AZ-1 -> Kafka broker
+-- AZ-2 -> Kafka broker
+-- AZ-3 -> Kafka broker
```

Use topology-aware scheduling and persistent storage.

# 80. 80. Dedicated Node Groups

Dedicated Kafka nodes can reduce noisy-neighbor effects. Use taints and
tolerations when isolation is required.

# 81. 81. Pod Anti-Affinity

Prevent critical brokers from concentrating on one Kubernetes node when the
failure model requires host-level separation.

# 82. 82. Topology Spread

Use topology spread constraints to distribute brokers across zones and
hosts.

# 83. 83. PDB

A PodDisruptionBudget can limit voluntary disruption, but it cannot prevent
unexpected node or AZ failures.

# 84. 84. Storage Topology

EBS-like zonal storage requires scheduling awareness. A Pod cannot simply be
moved to another AZ while assuming the same zonal volume is immediately
available there.

# 85. 85. Graceful Shutdown

Kafka brokers need sufficient time for controlled shutdown and partition
leadership/replica recovery. Kubernetes termination settings should reflect
this.

# 86. 86. Rolling Restart

A safe rolling operation is:

```text
verify health
 |
restart one broker
 |
wait for recovery
 |
verify replication
 |
continue
```

Do not restart multiple critical brokers simultaneously without a tested
procedure.

# 87. 87. Upgrade Architecture

Upgrade planning must include:

```text
Kafka
JVM
operator
Kubernetes
client libraries
storage
```

Validate compatibility before production.

# 88. 88. Capacity Headroom

Design for:

```text
normal load
peak load
one-broker failure
recovery load
```

A cluster that only works when every broker is healthy is not resilient.

# 89. 89. Failure Domain

The architecture should explicitly state what happens during:

```text
Pod failure
node failure
disk failure
AZ failure
controller failure
region failure
```


# 90. 90. HA vs DR

Multi-AZ replication is in-region HA. Cross-region recovery is DR. They have
different objectives and testing requirements.

# 91. 91. Multi-Region

Multi-region Kafka requires decisions around:

```text
replication
producer routing
consumer routing
offsets
duplicate events
ordering
failback
```

Active-active is substantially more complex than active-passive.

# 92. 92. Backup

Back up the configuration and platform state that must be recreated.
Determine separately how retained event data can be recovered after major
failure or corruption.

# 93. 93. Restore

A restore test should validate:

```text
topics
configuration
security
schemas
connectors
consumer strategy
business event recovery
```

A backup that has never been restored is an unverified recovery assumption.

# 94. 94. Observability Architecture

Monitor:

```text
broker
controller
topic
partition
producer
consumer group
storage
network
Kubernetes
```


# 95. 95. Golden Signals

Use:

```text
traffic
latency
errors
saturation
```

and Kafka-specific signals such as lag, ISR health and offline partitions.

# 96. 96. Alerting

High-value alerts include:

```text
offline partitions
under-replicated partitions
controller quorum risk
disk pressure
consumer lag
message age
producer errors
consumer rebalances
```


# 97. 97. Logging

Centralize broker, operator and application logs. Include event IDs and
correlation IDs where safe.

# 98. 98. Tracing

Trace context can connect:

```text
HTTP
 |
producer
 |
Kafka
 |
consumer
 |
database/API
```

This is essential for end-to-end latency analysis.

# 99. 99. Security Operations

Audit administrative changes, credential use, ACL changes and certificate
rotation. Treat Kafka access as production data access.

# 100. 100. Cost Architecture

Kafka cost comes from:

```text
brokers
storage
replication
network
cross-AZ transfer
retention
monitoring
```

Optimization should never silently violate RPO, RTO or SLO requirements.

# 101. 101. Performance Testing

Load tests should reproduce:

```text
message size distribution
producer rate
partition distribution
consumer concurrency
compression
TLS
replication
```

Measure both steady-state and recovery performance.

# 102. 102. Failure Testing

Test broker, consumer, node, disk, network, AZ and dependency failures.
Measure recovery rather than assuming it.

# 103. 103. Incident: High Consumer Lag

```text
1. Compare producer and consumer rates.
2. Inspect partition-level lag.
3. Check consumer errors/rebalances.
4. Check downstream latency.
5. Check broker CPU/disk/network.
6. Identify hot partitions.
7. Scale only within dependency capacity.
8. Measure recovery.
```

# 104. 104. Incident: Under-Replicated Partitions

```text
1. Identify partitions.
2. Identify affected brokers.
3. Check disk and network.
4. Check broker health.
5. Check follower lag.
6. Restore capacity.
7. Verify ISR recovery.
```


# 105. 105. Incident: Advertised Listener Failure

Typical symptom:

```text
bootstrap connection works
metadata works
broker connection fails
```

Validate advertised addresses, DNS, security groups, routes, TLS and listener
configuration.

# 106. 106. Incident: Disk Full

Protect the broker first, then identify retention/backlog causes and restore
capacity. Do not delete production data blindly as an emergency action.

# 107. 107. Incident: Rebalance Storm

Investigate:

```text
consumer crashes
deployment churn
heartbeat/session settings
long processing
network instability
```

Stabilize group membership before scaling.

# 108. 108. Incident: Producer Timeout

Check:

```text
broker request latency
network
partition leader
ISR
disk
CPU
authentication
```

Determine whether the timeout is a symptom or the root cause.

# 109. 109. Incident: Duplicate Business Effects

Trace:

```text
consumer processing
offset commit
crash timing
retry
event ID
database transaction
```

Fix the application boundary rather than assuming Kafka lost correctness.

# 110. 110. Architecture Review

A senior review should ask:

```text
What failure can this survive?
What can it lose?
How fast can it recover?
How does it scale?
What is the bottleneck?
How is it secured?
How is it observed?
How is it restored?
What happens during an upgrade?
```


# 111. 111. Production Readiness

```text
[ ] requirements
[ ] RPO/RTO
[ ] partitions
[ ] replication
[ ] failure domains
[ ] storage
[ ] listeners
[ ] TLS
[ ] authentication
[ ] authorization
[ ] quotas
[ ] retention
[ ] schema
[ ] producer reliability
[ ] consumer reliability
[ ] retry/error topics
[ ] idempotency
[ ] observability
[ ] backup
[ ] DR
[ ] upgrades
[ ] failure tests
```


# 112. 112. Senior Interview: Design Kafka for EKS

Answer:

```text
I start with throughput, message size, retention, ordering, RPO and RTO.
I then choose partition and replication strategy and distribute brokers across
AZs. On EKS I use persistent storage, topology-aware scheduling, disruption
controls and a Kafka-aware lifecycle system. I design listeners and security
before application onboarding. Finally I test broker, node, AZ, network,
consumer and recovery failures and size the cluster for one-broker failure
rather than only normal traffic.
```


# 113. 113. Senior Interview: Why Three AZs?

Answer:

```text
Three AZs provide independent failure domains and make it practical to retain
a replicated partition set when one AZ is unavailable. The exact broker and
replica count still depends on throughput and failure requirements.
```


# 114. 114. Senior Interview: How Do You Choose Partitions?

Answer:

```text
I calculate required throughput and consumer parallelism, then constrain the
choice using ordering and future growth. I also estimate recovery cost because
partition count affects operational work.
```


# 115. 115. Senior Interview: How Do You Choose RF?

Answer:

```text
I choose replication from the required failure tolerance and durability,
then evaluate storage and network cost. RF=3 is a common baseline, not a
universal answer.
```


# 116. 116. Senior Interview: What Happens During Broker Failure?

Answer:

```text
Partitions led by the failed broker may elect eligible replicas. Clients
refresh metadata and reconnect. I monitor leader changes, ISR, producer errors,
consumer lag and recovery time.
```


# 117. 117. Senior Interview: How Do You Handle Ordering?

Answer:

```text
I define the ordering scope first. For per-entity ordering I use a stable
entity key. I do not promise global ordering across multiple partitions unless
the architecture actually enforces it.
```


# 118. 118. Senior Interview: How Do You Handle Large Backlog?

Answer:

```text
I calculate incoming rate versus processing rate and identify the bottleneck.
Then I increase capacity only where downstream systems can handle it. I use
message age and drain rate to estimate recovery.
```


# 119. 119. Senior Interview: Why Kafka on Kubernetes Is Difficult?

Answer:

```text
Kafka is stateful and distributed. Kubernetes manages Pods and storage but does
not automatically solve Kafka replication, partition leadership, listeners,
quorum safety, capacity or upgrade sequencing. A Kafka-aware operator and
careful topology design reduce operational risk.
```


# 120. 120. Senior Interview: Kafka vs RabbitMQ

Answer:

```text
Kafka is optimized around durable partitioned logs, retention, replay and
large-scale streaming. RabbitMQ is strong for routing, queues and work
distribution. I select based on the required message lifecycle rather than
tool popularity.
```


# 121. 121. Final Architecture Principle

```text
requirements
   |
partition + replication
   |
failure-domain design
   |
storage + network
   |
producer + consumer semantics
   |
security
   |
observability
   |
recovery + DR
   |
failure testing
   |
production
```

A Kafka architecture is production-grade only when its behavior under failure
is understood, measured and repeatable.

# 122. Production Architecture Exercise 1: design a 3-AZ production cluster

## Objective

Evaluate the architecture for **design a 3-AZ production cluster**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 123. Production Architecture Exercise 2: calculate partitions from throughput

## Objective

Evaluate the architecture for **calculate partitions from throughput**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 124. Production Architecture Exercise 3: calculate storage from retention

## Objective

Evaluate the architecture for **calculate storage from retention**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 125. Production Architecture Exercise 4: review replication-factor choices

## Objective

Evaluate the architecture for **review replication-factor choices**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 126. Production Architecture Exercise 5: design per-order ordering

## Objective

Evaluate the architecture for **design per-order ordering**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 127. Production Architecture Exercise 6: diagnose a hot partition

## Objective

Evaluate the architecture for **diagnose a hot partition**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 128. Production Architecture Exercise 7: diagnose high consumer lag

## Objective

Evaluate the architecture for **diagnose high consumer lag**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 129. Production Architecture Exercise 8: diagnose under-replicated partitions

## Objective

Evaluate the architecture for **diagnose under-replicated partitions**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 130. Production Architecture Exercise 9: diagnose offline partitions

## Objective

Evaluate the architecture for **diagnose offline partitions**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 131. Production Architecture Exercise 10: diagnose producer timeouts

## Objective

Evaluate the architecture for **diagnose producer timeouts**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 132. Production Architecture Exercise 11: diagnose consumer rebalance storms

## Objective

Evaluate the architecture for **diagnose consumer rebalance storms**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 133. Production Architecture Exercise 12: diagnose advertised-listener failures

## Objective

Evaluate the architecture for **diagnose advertised-listener failures**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 134. Production Architecture Exercise 13: design internal Kafka listeners

## Objective

Evaluate the architecture for **design internal Kafka listeners**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 135. Production Architecture Exercise 14: design external Kafka listeners

## Objective

Evaluate the architecture for **design external Kafka listeners**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 136. Production Architecture Exercise 15: design TLS rotation

## Objective

Evaluate the architecture for **design TLS rotation**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 137. Production Architecture Exercise 16: design least-privilege ACLs

## Objective

Evaluate the architecture for **design least-privilege ACLs**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 138. Production Architecture Exercise 17: design multi-tenant quotas

## Objective

Evaluate the architecture for **design multi-tenant quotas**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 139. Production Architecture Exercise 18: design Kafka on EKS

## Objective

Evaluate the architecture for **design Kafka on EKS**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 140. Production Architecture Exercise 19: design dedicated Kafka node groups

## Objective

Evaluate the architecture for **design dedicated Kafka node groups**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 141. Production Architecture Exercise 20: design topology spread

## Objective

Evaluate the architecture for **design topology spread**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 142. Production Architecture Exercise 21: design persistent storage

## Objective

Evaluate the architecture for **design persistent storage**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 143. Production Architecture Exercise 22: plan a rolling restart

## Objective

Evaluate the architecture for **plan a rolling restart**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 144. Production Architecture Exercise 23: plan a Kafka upgrade

## Objective

Evaluate the architecture for **plan a Kafka upgrade**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 145. Production Architecture Exercise 24: plan an operator upgrade

## Objective

Evaluate the architecture for **plan an operator upgrade**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 146. Production Architecture Exercise 25: test broker failure

## Objective

Evaluate the architecture for **test broker failure**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 147. Production Architecture Exercise 26: test node failure

## Objective

Evaluate the architecture for **test node failure**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 148. Production Architecture Exercise 27: test AZ failure

## Objective

Evaluate the architecture for **test AZ failure**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 149. Production Architecture Exercise 28: test disk pressure

## Objective

Evaluate the architecture for **test disk pressure**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 150. Production Architecture Exercise 29: test network failure

## Objective

Evaluate the architecture for **test network failure**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 151. Production Architecture Exercise 30: test controller failure

## Objective

Evaluate the architecture for **test controller failure**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 152. Production Architecture Exercise 31: design retry topics

## Objective

Evaluate the architecture for **design retry topics**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 153. Production Architecture Exercise 32: design error topics

## Objective

Evaluate the architecture for **design error topics**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 154. Production Architecture Exercise 33: design safe replay

## Objective

Evaluate the architecture for **design safe replay**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 155. Production Architecture Exercise 34: design idempotent consumers

## Objective

Evaluate the architecture for **design idempotent consumers**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 156. Production Architecture Exercise 35: design an inbox pattern

## Objective

Evaluate the architecture for **design an inbox pattern**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 157. Production Architecture Exercise 36: design an outbox pattern

## Objective

Evaluate the architecture for **design an outbox pattern**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 158. Production Architecture Exercise 37: define exactly-once boundaries

## Objective

Evaluate the architecture for **define exactly-once boundaries**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 159. Production Architecture Exercise 38: protect a database dependency

## Objective

Evaluate the architecture for **protect a database dependency**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 160. Production Architecture Exercise 39: protect an API dependency

## Objective

Evaluate the architecture for **protect an API dependency**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 161. Production Architecture Exercise 40: design lag-based autoscaling

## Objective

Evaluate the architecture for **design lag-based autoscaling**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 162. Production Architecture Exercise 41: design KEDA limits

## Objective

Evaluate the architecture for **design KEDA limits**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 163. Production Architecture Exercise 42: design topic governance

## Objective

Evaluate the architecture for **design topic governance**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 164. Production Architecture Exercise 43: design schema governance

## Objective

Evaluate the architecture for **design schema governance**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 165. Production Architecture Exercise 44: design retention policy

## Objective

Evaluate the architecture for **design retention policy**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 166. Production Architecture Exercise 45: design compaction

## Objective

Evaluate the architecture for **design compaction**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 167. Production Architecture Exercise 46: design cross-region DR

## Objective

Evaluate the architecture for **design cross-region DR**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 168. Production Architecture Exercise 47: define RPO and RTO

## Objective

Evaluate the architecture for **define RPO and RTO**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 169. Production Architecture Exercise 48: design backup and restore

## Objective

Evaluate the architecture for **design backup and restore**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 170. Production Architecture Exercise 49: build observability

## Objective

Evaluate the architecture for **build observability**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 171. Production Architecture Exercise 50: build SLOs

## Objective

Evaluate the architecture for **build SLOs**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 172. Production Architecture Exercise 51: build production alerts

## Objective

Evaluate the architecture for **build production alerts**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 173. Production Architecture Exercise 52: perform capacity planning

## Objective

Evaluate the architecture for **perform capacity planning**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 174. Production Architecture Exercise 53: calculate recovery drain time

## Objective

Evaluate the architecture for **calculate recovery drain time**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 175. Production Architecture Exercise 54: review cross-AZ cost

## Objective

Evaluate the architecture for **review cross-AZ cost**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 176. Production Architecture Exercise 55: review partition explosion

## Objective

Evaluate the architecture for **review partition explosion**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 177. Production Architecture Exercise 56: review noisy-neighbor isolation

## Objective

Evaluate the architecture for **review noisy-neighbor isolation**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 178. Production Architecture Exercise 57: conduct a production readiness review

## Objective

Evaluate the architecture for **conduct a production readiness review**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 179. Production Architecture Exercise 58: conduct a senior architecture interview

## Objective

Evaluate the architecture for **conduct a senior architecture interview**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 180. Production Architecture Exercise 59: perform an incident postmortem

## Objective

Evaluate the architecture for **perform an incident postmortem**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# 181. Production Architecture Exercise 60: create a failure-domain matrix

## Objective

Evaluate the architecture for **create a failure-domain matrix**.

## Method

```text
1. State assumptions.
2. Define business impact.
3. Identify the failure domain.
4. Identify affected topics and partitions.
5. Identify producer impact.
6. Identify consumer impact.
7. Check broker/controller health.
8. Check storage and network.
9. Check security controls.
10. Check observability.
11. Execute the smallest safe mitigation.
12. Measure recovery.
13. Validate data correctness.
14. Document the root cause.
15. Convert the lesson into automation, policy or testing.
```

## Senior-level questions

```text
What is the bottleneck?
What can fail independently?
What data can be lost?
What happens during recovery?
What is the blast radius?
How does the design scale?
What trade-off was accepted?
How is the decision tested?
```

# Final Golden Rules

```text
1. Start with requirements.
2. Define RPO and RTO.
3. Treat Kafka as a distributed stateful platform.
4. Separate broker, controller and client concerns.
5. Use KRaft for modern supported architectures where appropriate.
6. Design partitions deliberately.
7. Treat partition count as a capacity and correctness decision.
8. Use keys for explicit ordering requirements.
9. Watch for hot keys and hot partitions.
10. Use replication for redundancy.
11. Spread replicas across failure domains.
12. Monitor ISR continuously.
13. Alert on under-replicated partitions.
14. Alert on offline partitions.
15. Keep disk headroom.
16. Account for replication traffic.
17. Account for AWS cross-AZ costs.
18. Use persistent storage.
19. Use topology-aware Kubernetes scheduling.
20. Use Kafka-aware lifecycle automation.
21. Design listeners before onboarding clients.
22. Validate advertised listeners from real client networks.
23. Secure Kafka with TLS and least privilege.
24. Use quotas for noisy-neighbor protection.
25. Treat topics as APIs.
26. Govern schemas.
27. Use producer acknowledgements intentionally.
28. Evaluate idempotent producers for critical workloads.
29. Expect consumer redelivery.
30. Make external side effects idempotent.
31. Use inbox/outbox patterns where appropriate.
32. Define exactly-once scope precisely.
33. Design retry topics deliberately.
34. Design error/DLQ topics deliberately.
35. Protect downstream dependencies.
36. Scale consumers within dependency capacity.
37. Monitor lag and message age.
38. Monitor broker saturation.
39. Monitor controller health.
40. Test broker failure.
41. Test consumer failure.
42. Test node failure.
43. Test disk failure.
44. Test AZ failure.
45. Test network failure.
46. Test upgrades.
47. Test restore.
48. Test DR.
49. Measure recovery time.
50. Design for the failure state, not just the healthy state.
```

# END OF 15-Kafka-Architecture.md
