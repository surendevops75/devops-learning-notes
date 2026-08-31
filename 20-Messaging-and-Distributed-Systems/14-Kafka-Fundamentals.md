# 20-Messaging-and-Distributed-Systems
# 14-Kafka-Fundamentals

## Production-Oriented Kafka Fundamentals

Kafka is a distributed event-streaming platform built around durable, partitioned logs. The most important mental model is:

```text
Producer -> Topic -> Partitions -> Consumer Groups -> Applications
                    |
                    +-> Replicas -> Brokers -> Failure Domains
```

This chapter establishes the foundation required for production Kafka architecture. It deliberately connects every fundamental concept to DevOps, Kubernetes, AWS/EKS, reliability, security, observability and system-design decisions.

---

# 1. What Is Kafka?

Apache Kafka is a distributed event-streaming platform. It is commonly used for event streaming, application integration, data pipelines, CDC, analytics and event-driven architectures.

Kafka is fundamentally based on an append-oriented distributed log.

---

# 2. Why Kafka?

Synchronous integration creates direct runtime coupling:

```text
A -> B -> C
```

Kafka can decouple producers and consumers:

```text
A -> Kafka -> B
          -> C
          -> D
```

A producer can publish an event without synchronously waiting for every downstream consumer.

---

# 3. Kafka Mental Model

Think of Kafka as:

```text
Distributed storage
+
Ordered partitioned logs
+
Consumer position tracking
+
Replication
+
Event streaming
```

This model is more useful than thinking of Kafka as simply a queue.

---

# 4. Event

An event represents something that happened.

Example:

```json
{
  "event_type": "order.created",
  "order_id": "ORD-1001",
  "amount": 2500
}
```

---

# 5. Record

A Kafka record can contain:

```text
key
value
headers
timestamp
```

The key is important for partition selection and ordering.

---

# 6. Producer

A producer application publishes records.

```text
Application
   |
Producer Client
   |
Kafka Broker
```

Producer design affects throughput, latency, durability and duplicate behavior.

---

# 7. Consumer

A consumer reads records.

```text
Kafka Broker
   |
Consumer Client
   |
Application
```

Consumer design affects throughput, lag, retries and business correctness.

---

# 8. Topic

A topic is a logical stream of records.

Examples:

```text
orders
payments
inventory
notifications
user-events
```

A topic is not equivalent to a single physical queue.

---

# 9. Partition

A topic is divided into partitions.

```text
orders
 |
 +-- P0
 +-- P1
 +-- P2
```

Partitions are the fundamental unit of Kafka parallelism and ordering.

---

# 10. Why Partitions Matter

Kafka scales by distributing partitions across brokers and allowing consumers in a group to process different partitions concurrently.

```text
P0 -> Consumer A
P1 -> Consumer B
P2 -> Consumer C
```

---

# 11. Partition Ordering

Kafka guarantees ordering within a partition.

```text
P0:
100 -> A
101 -> B
102 -> C
```

The ordering guarantee does not automatically extend across partitions.

---

# 12. Global Ordering

Global ordering across a topic generally requires a single partition or an application-level sequencing design.

One partition provides simple ordering but limits parallelism.

---

# 13. Per-Key Ordering

For business entities such as orders, use an appropriate key:

```text
key = order_id
```

Events for the same key can be routed consistently to the same partition, preserving per-key ordering under the same partitioning behavior.

---

# 14. Hot Keys

If one key generates disproportionate traffic:

```text
customer-A -> P0
```

P0 may become a hot partition.

Hot keys require a trade-off between ordering and distribution.

---

# 15. Topic Naming

Use an organization-wide naming convention.

Example:

```text
prod.orders.v1
prod.payments.v1
prod.inventory.v1
```

Names should communicate ownership and purpose without becoming excessively complicated.

---

# 16. Topic Ownership

Every production topic should have:

```text
owner
purpose
schema
retention
partition strategy
replication policy
security policy
SLO
```

---

# 17. Topic as API

Treat a topic as a contract.

Changing its schema, key semantics, retention or partition strategy can affect downstream applications.

---

# 18. Event Envelope

A useful production event envelope is:

```json
{
  "event_id": "evt-123",
  "event_type": "order.created",
  "schema_version": 1,
  "occurred_at": "2026-08-28T12:00:00Z",
  "producer": "order-service",
  "correlation_id": "corr-123",
  "payload": {}
}
```

---

# 19. Event ID

Use a stable event ID for tracing, deduplication and operational investigation.

---

# 20. Correlation ID

Correlation IDs connect multiple operations belonging to one business flow.

---

# 21. Trace ID

Trace context can connect:

```text
HTTP request
 -> producer
 -> Kafka
 -> consumer
 -> database/API
```

---

# 22. Headers

Kafka headers can carry metadata such as:

```text
trace context
correlation ID
content type
schema information
```

Do not use headers as a substitute for a well-designed payload.

---

# 23. Serialization

Common formats include:

```text
JSON
Avro
Protobuf
JSON Schema
```

Choose based on interoperability, governance and operational maturity.

---

# 24. JSON

Advantages:

```text
simple
human-readable
widely supported
```

Trade-offs:

```text
larger payloads
weaker contract enforcement unless schema tooling is added
```

---

# 25. Binary Schemas

Avro and Protobuf can provide compact representation and stronger schema discipline, at the cost of additional tooling and governance.

---

# 26. Schema Evolution

Producers and consumers may be deployed independently.

Therefore schemas must support controlled evolution.

---

# 27. Backward Compatibility

A newer consumer should be able to understand supported older events when the compatibility policy requires it.

---

# 28. Forward Compatibility

An older consumer may need to tolerate fields added by a newer producer.

---

# 29. Breaking Schema Changes

Avoid silently changing:

```text
field type
field meaning
required fields
key semantics
```

Use versioning or migration strategies for breaking changes.

---

# 30. Producer Throughput

Producer throughput depends on:

```text
batching
compression
network
acks
message size
broker capacity
```

---

# 31. Producer Batching

Batching groups records into requests.

Benefits:

```text
fewer requests
better network utilization
higher throughput
```

---

# 32. Batch Trade-Off

Very large batching can increase latency because the producer may wait longer before sending.

Tune from measurements rather than copying a generic configuration.

---

# 33. Compression

Kafka producers commonly support compression algorithms such as:

```text
gzip
snappy
lz4
zstd
```

Exact availability depends on client/version.

---

# 34. Compression Trade-Off

Compression can reduce network and storage usage but consumes CPU.

---

# 35. Producer Acknowledgements

Producer acknowledgement configuration determines how much broker-side confirmation is required.

Stronger acknowledgement generally improves durability expectations at some latency/capacity cost.

---

# 36. Producer Retry

Transient failures may trigger producer retries.

Retries introduce an important question:

```text
Was the original request actually accepted?
```

---

# 37. Duplicate Publication

If a producer loses an acknowledgement, it may be uncertain whether the record was accepted.

Use idempotent producer capabilities and application-level event IDs where appropriate.

---

# 38. Idempotent Producer

Kafka supports producer idempotence for supported producer workflows. It reduces duplicates caused by producer retries within its defined semantics.

It does not solve every business-level duplicate problem.

---

# 39. Kafka Transactions

Kafka transactions can atomically publish multiple Kafka records and coordinate consumed-offset commits in supported Kafka processing patterns.

Do not equate a Kafka transaction with a database transaction.

---

# 40. Consumer Group

A consumer group is a logical set of consumers sharing topic partitions.

```text
Topic
 |
 +-- P0 -> Consumer A
 +-- P1 -> Consumer B
 +-- P2 -> Consumer C
```

---

# 41. Consumer Group Scaling

If a topic has three partitions and a group has five consumers:

```text
3 active consumers
2 idle consumers
```

Adding consumers beyond useful partition parallelism does not automatically increase throughput.

---

# 42. Multiple Consumer Groups

Different applications can consume the same topic independently:

```text
orders
 |
 +-- payment-group
 +-- inventory-group
 +-- analytics-group
```

Each group maintains its own position.

---

# 43. Queue-Like Behavior

Within one consumer group, a partition has one active consumer owner at a time.

This makes consumer groups useful for work distribution.

---

# 44. Pub/Sub-Like Behavior

Across multiple consumer groups, the same event can be processed independently by multiple applications.

---

# 45. Group ID

The group ID identifies the logical consuming application.

Changing the group ID creates a different consumer position/history.

---

# 46. Consumer Rebalance

Membership changes can cause partition reassignment:

```text
consumer joins
consumer leaves
consumer crashes
partition topology changes
```

---

# 47. Rebalance Impact

Rebalances can temporarily reduce processing throughput.

Frequent rebalances should be investigated rather than accepted as normal.

---

# 48. Consumer Stability

Avoid unnecessary consumer restarts and unstable deployments.

Stable membership reduces rebalance pressure.

---

# 49. Consumer Offset

Consumers track their position through offsets.

Example:

```text
P0:
100
101
102
103
```

A consumer may have committed its progress through a particular offset.

---

# 50. Offset Is Partition-Local

Offset 100 in P0 is unrelated to offset 100 in P1.

There is no single global offset for a multi-partition topic.

---

# 51. Consumer Commit

Consumers commit offsets so they can resume after restart.

---

# 52. Auto Commit

Automatic commits simplify clients but can produce undesirable processing/commit relationships if the workload requires precise control.

---

# 53. Manual Commit

Manual commits provide explicit control over when processing progress becomes durable from the consumer group's perspective.

---

# 54. At-Least-Once Pattern

A common approach is:

```text
poll
 |
process
 |
commit offset
```

If the consumer crashes after processing but before committing, the record may be processed again.

---

# 55. At-Most-Once Pattern

If the offset is committed before processing:

```text
poll
 |
commit
 |
process
```

A crash can cause the record to be skipped.

---

# 56. Exactly-Once Scope

Exactly-once must always be defined by boundary.

For Kafka-to-Kafka processing, transactions can provide stronger semantics. External side effects require additional design.

---

# 57. Kafka to Database

Consider:

```text
Kafka -> Consumer -> Database
```

A Kafka transaction does not automatically make the database update exactly once.

---

# 58. Inbox Pattern

A consumer can use a durable processed-event record:

```text
Kafka event
 |
DB transaction
 +-- processed_event
 +-- business update
 |
COMMIT
 |
commit Kafka offset
```

This provides application-level deduplication.

---

# 59. Outbox Pattern

For database-to-Kafka publication:

```text
DB transaction
 +-- business update
 +-- outbox event
 |
COMMIT
 |
Outbox publisher -> Kafka
```

This reduces the dual-write problem.

---

# 60. Dual-Write Problem

Bad pattern:

```text
DB commit
 |
Kafka publish
```

If Kafka publication fails after DB commit, the database and event stream diverge.

---

# 61. Retention

Kafka retains records independently of whether a particular consumer has read them.

Retention is a fundamental difference from traditional destructive queues.

---

# 62. Time Retention

A topic can retain records for a configured time period.

---

# 63. Size Retention

Retention can also be constrained by log size.

---

# 64. Replay

A consumer group can be moved back to an earlier position while the required records are still retained.

Replay is one of Kafka's major operational capabilities.

---

# 65. Replay Risk

Replay can cause:

```text
duplicate business effects
downstream overload
ordering complications
```

Always validate idempotency before replaying production data.

---

# 66. Log Compaction

Kafka supports a compaction cleanup policy that keeps the latest value for keys subject to Kafka's compaction behavior.

---

# 67. Compaction Use Cases

Compaction can be useful for:

```text
latest entity state
configuration
reference data
```

---

# 68. Tombstones

A null value can be used as a tombstone in compacted topics to represent deletion semantics.

---

# 69. Compaction Is Not a Database

A compacted Kafka topic is still a Kafka log with Kafka-specific semantics. It is not a replacement for transactional database storage.

---

# 70. Topic Cleanup Policies

Common cleanup modes include:

```text
delete
compact
```

and supported combinations.

---

# 71. Partition Count

Partition count should be chosen from:

```text
throughput
consumer parallelism
ordering
future growth
```

---

# 72. Too Few Partitions

Too few partitions can limit:

```text
consumer parallelism
producer distribution
throughput
```

---

# 73. Too Many Partitions

Too many partitions increase:

```text
metadata
file handles
recovery work
rebalance complexity
operational overhead
```

---

# 74. Partition Count Is a Design Decision

Do not select a partition count simply because another company uses that number.

Benchmark the workload.

---

# 75. Increasing Partitions

Increasing partition count can change key-to-partition mapping because the partition count participates in partition selection.

Strict ordering architectures must account for this.

---

# 76. Broker

A Kafka broker is a server participating in the cluster.

A broker can store partition replicas and serve producer and consumer requests.

---

# 77. Kafka Cluster

A Kafka cluster contains multiple brokers and the metadata/coordination architecture required by the selected Kafka deployment.

---

# 78. Partition Leader

For a replicated partition, one replica acts as leader for normal client traffic.

---

# 79. Follower Replica

Followers replicate the leader's log.

---

# 80. Replication Factor

Replication factor specifies how many replicas exist for each partition.

Example:

```text
RF=3
```

means three replicas for that partition.

---

# 81. Replication vs Partitions

These solve different problems:

```text
partitions -> parallelism
replication -> redundancy
```

---

# 82. RF=1

RF=1 provides no replica redundancy.

A broker failure can make a partition unavailable and may create data-loss risk depending on storage and recovery conditions.

---

# 83. RF=3

RF=3 is a common production baseline for critical workloads, but it is not universally correct.

It costs more storage and replication bandwidth.

---

# 84. Replica Placement

Replicas should be distributed across independent failure domains where possible.

```text
P0
 |
 +-- AZ-1
 +-- AZ-2
 +-- AZ-3
```

---

# 85. ISR

ISR means in-sync replicas.

These replicas are sufficiently caught up according to Kafka's replication rules.

---

# 86. ISR Shrink

If followers fall behind, ISR can shrink.

Sustained ISR shrink is an important production health signal.

---

# 87. Under-Replicated Partition

A partition is under-replicated when fewer replicas are healthy/in sync than the configured replication factor.

This should normally trigger investigation.

---

# 88. Offline Partition

An offline partition cannot serve the expected workload because no suitable leader is available.

Treat this as a high-severity availability condition.

---

# 89. Leader Election

If a leader fails, Kafka can elect another eligible replica according to its configured replication and leadership rules.

---

# 90. Broker Failure

```text
Broker fails
 |
partition leaders unavailable
 |
eligible replicas
 |
new leaders
 |
clients recover
```

Actual recovery depends on cluster health and client configuration.

---

# 91. Min ISR

Kafka supports minimum in-sync replica settings that can constrain successful writes under certain acknowledgement configurations.

The purpose is to avoid accepting writes when the desired replication safety is no longer available.

---

# 92. Acknowledgement and Min ISR

Durability is a combined design problem involving:

```text
acks
replication factor
ISR
min ISR
producer idempotence
failure scenario
```

---

# 93. Leader Imbalance

If too many partition leaders reside on one broker:

```text
CPU imbalance
network imbalance
request imbalance
```

can occur.

---

# 94. Broker Balance

Monitor:

```text
leaders
partitions
bytes in
bytes out
CPU
disk
```

---

# 95. Controller Architecture

Kafka requires metadata/coordination functionality.

Modern Kafka can use KRaft instead of ZooKeeper.

---

# 96. KRaft

KRaft is Kafka's metadata quorum architecture.

Conceptually:

```text
Controller quorum
      |
   Metadata
      |
   Brokers
      |
 Partitions
```

---

# 97. ZooKeeper

Older Kafka deployments used ZooKeeper for coordination and metadata.

Modern designs should evaluate KRaft for supported Kafka versions and operational requirements.

---

# 98. KRaft Controller Quorum

Controllers maintain metadata using a quorum.

Controller availability must be designed separately from data partition availability.

---

# 99. Broker vs Controller

Conceptually:

```text
Broker -> data serving/storage
Controller -> metadata/cluster coordination
```

Modern Kafka can combine or separate roles depending on the deployment architecture.

---

# 100. Kafka Storage

Kafka stores partition logs on disk.

The log is divided into segments.

---

# 101. Append-Oriented Storage

Kafka's append-oriented design supports efficient sequential I/O patterns.

---

# 102. Log Segment

A partition consists of log segment files.

Segments roll based on configured limits and operational conditions.

---

# 103. Retention Deletion

Old segments become eligible for deletion when retention rules permit.

---

# 104. Disk Capacity

Plan storage from:

```text
incoming bytes/sec
retention duration
replication factor
headroom
```

---

# 105. Storage Formula

Approximate logical retained data:

```text
bytes_per_second * retention_seconds
```

Then account for replication and operational overhead.

---

# 106. Capacity Example

If logical traffic is 10 MB/s and retention is 24 hours:

```text
10 * 86,400
= 864,000 MB
≈ 864 GB
```

before replication and overhead.

---

# 107. RF Storage Example

At RF=3, the approximate replicated footprint for the previous logical data is:

```text
864 GB * 3
```

before filesystem, segment and operational overhead.

---

# 108. Disk Headroom

Do not operate disks at their absolute capacity limit.

Headroom is needed for:

```text
retention growth
replica recovery
rebalancing
operational spikes
```

---

# 109. Page Cache

Kafka relies significantly on operating-system page cache.

Do not assume all useful performance comes from JVM heap.

---

# 110. JVM Heap

Heap is used for Kafka process structures, requests and other runtime needs.

Excessive heap does not replace the need for page cache and sufficient system memory.

---

# 111. Memory Planning

Consider:

```text
JVM heap
page cache
network buffers
request buffers
compression
connections
```

---

# 112. CPU Planning

CPU is consumed by:

```text
request processing
compression/decompression
TLS
replication
network processing
```

---

# 113. Network Planning

Kafka can be network intensive.

Monitor:

```text
bytes in
bytes out
replication traffic
client traffic
```

---

# 114. Replication Network Cost

Replication creates additional network traffic.

Higher replication improves redundancy but increases resource consumption.

---

# 115. Cross-AZ Traffic

In AWS, multi-AZ Kafka can generate cross-AZ network traffic depending on replica and client placement.

This affects both cost and latency.

---

# 116. Rack Awareness

Kafka supports failure-domain-aware replica placement through rack information and related deployment configuration.

In AWS, availability zones are a natural failure-domain concept.

---

# 117. AWS Production Model

A common production layout is:

```text
Region
 |
 +-- AZ-1 -> brokers
 +-- AZ-2 -> brokers
 +-- AZ-3 -> brokers
```

---

# 118. Kafka on Kubernetes

Kafka is stateful and should be deployed using state-aware Kubernetes primitives and/or a Kafka-aware operator.

---

# 119. StatefulSet

StatefulSet provides stable identity and persistent volume association.

It does not itself implement Kafka replication or metadata semantics.

---

# 120. Kafka Operator

A Kafka operator can automate supported lifecycle tasks such as:

```text
cluster configuration
listeners
storage
users
topics
rolling changes
```

The exact capabilities depend on the operator.

---

# 121. Strimzi

Strimzi is a widely used Kubernetes operator ecosystem for Kafka.

Always verify supported Kafka, Kubernetes and operator versions before production use.

---

# 122. EKS Storage

On EKS, evaluate:

```text
CSI driver
volume topology
IOPS
throughput
AZ behavior
```

---

# 123. Dedicated Kafka Nodes

Dedicated node groups can reduce noisy-neighbor risk for critical Kafka brokers.

---

# 124. Topology Spread

Spread Kafka brokers across:

```text
availability zones
worker hosts
```

according to the failure model.

---

# 125. Anti-Affinity

Pod anti-affinity can prevent multiple brokers from sharing the same failure domain.

---

# 126. PodDisruptionBudget

A PDB can reduce voluntary disruption during node maintenance.

It does not protect against unexpected node or AZ failures.

---

# 127. Rolling Restart

During maintenance:

```text
check replication
 |
restart one broker
 |
wait for recovery
 |
verify
 |
continue
```

---

# 128. Kubernetes Node Drain

Node drains can move Kafka brokers.

Validate:

```text
PDB
storage
replica health
spare capacity
```

before maintenance.

---

# 129. Kubernetes Autoscaling

Do not treat Kafka brokers like stateless application Pods.

Autoscaling policies must understand storage, identity and replication constraints.

---

# 130. EKS Worker Capacity

Keep enough worker capacity to reschedule a broker after node failure.

---

# 131. Taints and Tolerations

Dedicated Kafka nodes can be protected with taints and corresponding broker tolerations.

---

# 132. Resource Requests

Set CPU and memory requests based on measured Kafka workloads.

---

# 133. Resource Limits

Overly restrictive CPU/memory limits can cause throttling or OOM conditions.

---

# 134. Storage Performance

Benchmark:

```text
write latency
read latency
throughput
recovery
replication
```

with realistic traffic.

---

# 135. Kafka Listener

Kafka clients connect through configured listeners.

A production architecture may have different listeners for:

```text
internal
external
TLS
authentication
```

---

# 136. Advertised Listener

Kafka returns broker metadata to clients.

Clients must be able to reach the advertised broker addresses.

This is one of the most important Kafka networking concepts.

---

# 137. Advertised Listener Failure

A client may successfully connect to the bootstrap endpoint but fail afterward when the broker metadata contains unreachable addresses.

Symptoms:

```text
bootstrap succeeds
metadata succeeds
produce/fetch fails
```

---

# 138. Kafka Is Not HTTP

A generic load balancer pattern used for HTTP cannot automatically solve Kafka broker discovery.

Kafka clients need correct broker metadata and reachable per-broker endpoints.

---

# 139. Kubernetes DNS

Kafka listener architecture must account for Kubernetes DNS and external DNS depending on client location.

---

# 140. External Clients

For clients outside Kubernetes, design:

```text
DNS
listeners
advertised addresses
TLS
network routing
```

---

# 141. Internal Clients

Internal applications should generally use private networking and stable internal listener addresses.

---

# 142. Public Kafka

Avoid exposing Kafka directly to the public internet unless there is a strong business requirement and comprehensive security controls.

---

# 143. TLS

Kafka security commonly includes TLS for client/broker and appropriate internal communication.

---

# 144. Authentication

Depending on the deployment, authentication can use mechanisms such as SASL or mutual TLS.

---

# 145. Authorization

Control access to:

```text
topics
consumer groups
cluster operations
```

according to least privilege.

---

# 146. ACLs

Kafka ACLs can restrict producers, consumers and administrative operations when configured.

---

# 147. Secrets

Store credentials and certificates through secure secret-management mechanisms.

Never hardcode production credentials into images or Git repositories.

---

# 148. Secret Rotation

Test rotation procedures and client reconnect behavior.

---

# 149. Quotas

Kafka supports client resource quotas that can help prevent noisy-neighbor behavior.

---

# 150. Multi-Tenancy

Use combinations of:

```text
topic naming
ACLs
quotas
consumer groups
```

for logical tenant governance.

---

# 151. Strong Tenant Isolation

If security or blast-radius requirements are high, separate Kafka clusters may be preferable.

---

# 152. Observability

Kafka observability should cover:

```text
broker
controller
partition
topic
consumer group
storage
network
```

---

# 153. Broker Metrics

Monitor:

```text
request rate
request latency
bytes in
bytes out
CPU
disk
JVM
network
```

---

# 154. Producer Metrics

Monitor:

```text
send rate
batch size
latency
errors
retries
record failures
```

---

# 155. Consumer Metrics

Monitor:

```text
records consumed
poll behavior
commit latency
rebalances
consumer lag
```

---

# 156. Partition Metrics

Monitor:

```text
leader distribution
under-replication
offline partitions
```

---

# 157. Consumer Lag

Lag represents how far a consumer group is behind the available records.

Use lag together with message age and processing rate.

---

# 158. Lag Is Not Always Bad

Some workloads intentionally process in batches.

The correct question is:

```text
Does lag violate the workload SLO?
```

---

# 159. Message Age

Message age can be more meaningful than record-count lag when message arrival rates vary.

---

# 160. Under-Replication Alert

Sustained under-replication should trigger investigation.

---

# 161. Offline Partition Alert

An offline partition is a severe availability signal.

---

# 162. Disk Alert

Alert before disk exhaustion.

Disk exhaustion can cause widespread broker problems.

---

# 163. JVM Alert

Monitor heap, garbage collection and pauses.

---

# 164. Network Alert

Monitor network saturation and abnormal replication traffic.

---

# 165. Rebalance Alert

Frequent consumer-group rebalances can indicate unstable consumers, deployment churn, timeout problems or network instability.

---

# 166. Logging

Centralize:

```text
broker logs
operator logs
Kubernetes events
application logs
```

---

# 167. Tracing

Propagate trace context through Kafka headers where supported by the application tracing design.

---

# 168. Correlation

Use:

```text
event_id
correlation_id
trace_id
```

for incident investigation.

---

# 169. Event-Driven Architecture

Kafka can decouple services:

```text
Order Service
     |
     v
   Kafka
  /  |  \
 v   v   v
Pay Inv Notify
```

---

# 170. Loose Coupling

Consumers can evolve independently from producers when the event contract is governed correctly.

---

# 171. Backpressure

Kafka allows a consumer to fall behind while records remain retained.

The limit is defined by retention and storage rather than immediate destructive consumption.

---

# 172. Consumer Recovery

A consumer can resume from its committed position after restart or rebalance.

---

# 173. Replay Recovery

Retained records can be replayed for:

```text
bug recovery
new consumer deployment
backfill
reprocessing
```

---

# 174. Poison Event

A record that repeatedly fails processing can block progress in a partition if the consumer insists on strict in-order processing.

---

# 175. Error Topic

A common design is:

```text
main topic
 |
consumer
 |
processing error
 |
error topic
```

---

# 176. Retry Topic

Example:

```text
orders
orders.retry.1m
orders.retry.10m
orders.dlq
```

---

# 177. Retry Ordering

Retry topics can complicate ordering.

If order-level ordering is critical, retry design must preserve or explicitly redefine the guarantee.

---

# 178. Kafka DLQ Concept

Kafka does not provide RabbitMQ-style DLQ semantics as the same queue primitive. Error/DLQ behavior is normally implemented using separate topics and consumer/application logic.

---

# 179. Consumer Pause

A consumer can pause partition processing in supported client patterns when handling certain conditions.

---

# 180. Retry Blocking

Strictly retrying the first failed record can prevent later records in that partition from progressing.

This is a trade-off between ordering and availability.

---

# 181. Ordering vs Availability

```text
strong ordering
     |
     v
more blocking risk
```

Relaxing ordering can increase recovery parallelism.

---

# 182. Large Messages

Large messages increase:

```text
memory
network
storage
replication
recovery time
```

---

# 183. Payload Reference Pattern

For large payloads:

```text
Object Storage
     ^
     |
Kafka event -> object key/reference
```

This can keep Kafka focused on event metadata rather than huge payload transport.

---

# 184. Event Contract

A reference event might contain:

```json
{
  "event_id": "evt-1",
  "object_uri": "...",
  "content_type": "application/json"
}
```

Secure the referenced object independently.

---

# 185. Kafka Connect

Kafka Connect provides source and sink connector architecture for moving data between Kafka and external systems.

---

# 186. CDC

CDC can publish database changes into Kafka:

```text
Database -> CDC -> Kafka -> Consumers
```

---

# 187. Stream Processing

Kafka can feed stream-processing applications:

```text
Kafka -> Stream Processor -> Kafka/DB/Warehouse
```

---

# 188. Data Platform

Kafka can act as an event backbone for:

```text
analytics
search
warehouse
data lake
ML pipelines
```

---

# 189. Kafka and RabbitMQ Together

Enterprises may use both when workloads differ.

Example:

```text
RabbitMQ -> operational task routing
Kafka     -> durable event stream
```

Do not introduce both without a clear reason.

---

# 190. Kafka vs RabbitMQ

Kafka emphasizes:

```text
durable log
partitions
retention
replay
consumer groups
streaming
```

RabbitMQ emphasizes:

```text
queues
exchanges
routing
acknowledgements
work distribution
```

The decision is workload-driven.

---

# 191. Kafka Capacity Planning

Plan for:

```text
normal load
peak load
broker failure
consumer failure
recovery load
```

---

# 192. Failure Headroom

If one broker fails, remaining brokers may absorb additional traffic and replication work.

They need sufficient headroom.

---

# 193. Storage Headroom

Keep enough free storage for recovery and re-replication.

---

# 194. CPU Headroom

Do not run all brokers at maximum normal CPU utilization.

---

# 195. Network Headroom

Account for:

```text
producer traffic
consumer traffic
replication
TLS
monitoring
```

---

# 196. Capacity Formula

A simple logical retained-data estimate is:

```text
traffic_bytes_per_second * retention_seconds
```

Then multiply for replication and add operational headroom.

---

# 197. Consumer Capacity

If:

```text
incoming = 20,000 records/s
processing = 25,000 records/s
```

then the consumer group has approximately:

```text
5,000 records/s
```

of net drain capacity before other constraints.

---

# 198. Lag Recovery

If backlog is 1,000,000 records and net drain is 5,000 records/s:

```text
1,000,000 / 5,000
= 200 seconds
```

This is only an idealized estimate.

---

# 199. Real Recovery

Actual recovery depends on:

```text
broker capacity
consumer CPU
downstream dependencies
network
rebalances
message size
```

---

# 200. Autoscaling Consumers

Queue-depth-like metrics such as lag can drive consumer scaling.

But scaling must respect downstream capacity.

---

# 201. Autoscaling Failure

```text
Dependency outage
 |
lag increases
 |
consumer autoscaling
 |
dependency receives more traffic
 |
errors increase
```

This can become a feedback loop.

---

# 202. Dependency-Aware Scaling

Consider:

```text
lag
processing rate
downstream health
downstream capacity
retry rate
```

---

# 203. Kubernetes Autoscaling

Kubernetes HPA/KEDA can scale consumer applications, but Kafka broker scaling is a different stateful capacity problem.

---

# 204. KEDA

KEDA can be useful for event-driven consumer scaling.

Set bounds so it cannot overwhelm downstream services.

---

# 205. Security Architecture

Production Kafka security should include:

```text
TLS
authentication
authorization
network controls
secret management
audit
```

---

# 206. Least Privilege

A payment consumer should normally receive only the topic/group permissions it requires.

---

# 207. Management Access

Administrative Kafka access should be restricted and audited.

---

# 208. Data Classification

Classify event data:

```text
public
internal
confidential
restricted
```

Then apply corresponding controls.

---

# 209. Sensitive Data

Avoid putting unnecessary passwords, tokens or secrets into Kafka records.

Kafka retention can make accidental exposure persist for a long time.

---

# 210. Retention Governance

Long retention increases:

```text
storage
cost
backup/recovery scope
security exposure
```

---

# 211. Multi-Region

In-region Kafka HA does not automatically provide regional disaster recovery.

---

# 212. Regional DR

A DR design may include:

```text
Region A -> primary Kafka
Region B -> recovery Kafka
```

with an explicit replication and failover strategy.

---

# 213. Active-Passive

```text
Region A ACTIVE
Region B STANDBY
```

Often operationally simpler than active-active.

---

# 214. Active-Active

Requires explicit handling of:

```text
routing
ownership
duplicates
ordering
conflicts
```

---

# 215. DR RPO

Define how much event loss is acceptable for each failure scenario.

---

# 216. DR RTO

Define how quickly producers and consumers must resume.

---

# 217. DR Testing

Test:

```text
producer failover
consumer failover
offset strategy
duplicate handling
DNS/routing
```

---

# 218. Backup

Back up appropriate configuration and metadata according to the platform architecture.

Replication alone does not replace every recovery requirement.

---

# 219. Configuration Recovery

Recover:

```text
topics
ACLs
users
connect configurations
schema metadata where applicable
```

---

# 220. Data Recovery

Define whether data can be reconstructed from:

```text
source databases
upstream systems
replicated clusters
archives
```

---

# 221. Production Incident: High Lag

First compare:

```text
producer rate
consumer rate
partition lag
message age
```

Then inspect downstream dependencies and broker resources.

---

# 222. High Lag: Hot Partition

If only one partition has extreme lag, investigate key distribution and hot keys.

---

# 223. High Lag: Consumer Failure

If consumers are unstable, stabilize group membership before increasing scale.

---

# 224. High Lag: Dependency Failure

A healthy consumer process may still be blocked on a database or API.

---

# 225. High Lag: Broker Saturation

Inspect:

```text
CPU
disk
network
request latency
```

---

# 226. Under-Replication Runbook

```text
identify topic/partition
 |
identify broker
 |
check disk
 |
check network
 |
check broker health
 |
check follower lag
 |
restore capacity
 |
verify ISR
```

---

# 227. Offline Partition Runbook

```text
identify partition
 |
identify replicas
 |
check broker/controller health
 |
restore eligible replica
 |
verify leader
 |
verify consumers
```

---

# 228. Advertised Listener Runbook

```text
bootstrap test
 |
metadata test
 |
inspect advertised broker addresses
 |
DNS test
 |
route/security test
 |
per-broker connection test
```

---

# 229. ACL Failure Runbook

Check:

```text
principal
listener
authentication
resource
operation
ACL rule
```

---

# 230. TLS Failure Runbook

Check:

```text
certificate
CA trust
hostname/SAN
protocol
listener
client configuration
```

---

# 231. Consumer Rebalance Runbook

Check:

```text
consumer restarts
heartbeat/session configuration
processing duration
network
deployment churn
```

---

# 232. Poison Event Runbook

```text
identify event
 |
identify partition
 |
classify failure
 |
protect downstream
 |
route/error-handle
 |
fix consumer
 |
reprocess safely
```

---

# 233. Replay Runbook

```text
stop uncontrolled replay
 |
identify offset range
 |
validate idempotency
 |
validate downstream capacity
 |
replay gradually
 |
monitor
```

---

# 234. Broker Failure Test

In staging:

```text
stop one broker
 |
observe leader movement
 |
observe producer behavior
 |
observe consumer lag
 |
observe replication recovery
```

---

# 235. Consumer Failure Test

```text
kill consumer
 |
observe rebalance
 |
observe new owner
 |
observe duplicate/reprocessing behavior
```

---

# 236. AZ Failure Test

Test the designed AZ failure scenario in a controlled environment.

Verify replica distribution and recovery.

---

# 237. Disk Failure Test

Validate that replica loss and recovery behave as expected.

---

# 238. Network Failure Test

Test producer, consumer and broker behavior under controlled network disruption.

---

# 239. Kubernetes Failure Test

Test:

```text
Pod eviction
node drain
node failure
```

---

# 240. Upgrade Test

Test Kafka, operator, Kubernetes and client compatibility before production changes.

---

# 241. Production Topic Example

```text
Topic: prod.orders.v1
Partitions: 12
Replication: 3
Key: order_id
Retention: 7 days
Groups: payment, inventory, notification, analytics
```

Values are illustrative and must be derived from requirements.

---

# 242. Why 12 Partitions?

Potential reasons include:

```text
parallelism
throughput
future growth
```

There is no universal correct partition count.

---

# 243. Why RF=3?

Potential rationale:

```text
failure tolerance
availability
storage/network trade-off
```

---

# 244. Why order_id Key?

It provides a basis for per-order ordering while allowing unrelated orders to be distributed.

---

# 245. Why Multiple Groups?

Payment, inventory and analytics require independent processing positions.

---

# 246. Production Readiness

Before go-live:

```text
[ ] requirements defined
[ ] RPO/RTO defined
[ ] partitions justified
[ ] key strategy defined
[ ] replication defined
[ ] failure domains defined
[ ] storage benchmarked
[ ] listeners tested
[ ] advertised listeners tested
[ ] TLS enabled where required
[ ] ACLs reviewed
[ ] monitoring configured
[ ] lag SLO defined
[ ] backup/recovery tested
[ ] DR documented
[ ] upgrades tested
[ ] failure tests completed
```

---

# 247. Go-Live Sequence

```text
requirements
 |
architecture
 |
load test
 |
security review
 |
failure test
 |
recovery test
 |
staging
 |
production rollout
 |
monitor
```

---

# 248. Senior Interview: What Is Kafka?

Answer:

```text
Kafka is a distributed event-streaming platform based on durable, partitioned
logs. Producers append records to topic partitions, consumers read by offset,
consumer groups distribute work, retention enables replay and replication
provides fault tolerance.
```

---

# 249. Senior Interview: Topic vs Partition

Answer:

```text
A topic is a logical stream. Partitions are the units inside that stream that
provide ordering scope, parallelism and distribution across brokers.
```

---

# 250. Senior Interview: Why Partitions?

Answer:

```text
Partitions allow Kafka to distribute storage and workload and allow consumers
within a group to process different partitions concurrently.
```

---

# 251. Senior Interview: Ordering

Answer:

```text
Kafka guarantees ordering within a partition. For business-level ordering I
use a stable key such as order_id and avoid claiming global ordering across
multiple partitions.
```

---

# 252. Senior Interview: Consumer Groups

Answer:

```text
A consumer group is a logical application identity whose members share topic
partitions. Different groups can consume the same topic independently.
```

---

# 253. Senior Interview: Offset

Answer:

```text
An offset is a position within a partition. It is partition-local, not a global
message identifier.
```

---

# 254. Senior Interview: Consumer Lag

Answer:

```text
Lag shows how far a consumer group is behind. I evaluate it with message age,
producer rate, consumer rate, partition distribution and downstream health.
```

---

# 255. Senior Interview: RF=3

Answer:

```text
RF=3 creates three replicas per partition. It is a common production baseline
because it balances failure tolerance with storage and replication cost, but
I choose it from requirements rather than habit.
```

---

# 256. Senior Interview: ISR

Answer:

```text
ISR is the set of replicas considered in sync. ISR shrink reduces redundancy
and is an important production health signal.
```

---

# 257. Senior Interview: Under-Replication

Answer:

```text
It means fewer healthy/in-sync replicas are available than the configured
replication factor. I investigate disk, network, broker health and follower
lag immediately.
```

---

# 258. Senior Interview: Kafka on EKS

Answer:

```text
I use a Kafka-aware stateful deployment, persistent storage, multi-AZ broker
placement, topology constraints, disruption controls, secure listeners,
monitoring and tested recovery. I explicitly validate EKS storage topology and
spare worker capacity.
```

---

# 259. Senior Interview: Advertised Listeners

Answer:

```text
Bootstrap only establishes the initial connection. Kafka then returns broker
metadata. If advertised broker addresses are unreachable, clients can fail
after bootstrap. Therefore listener and advertised-address design is a
critical networking concern.
```

---

# 260. Senior Interview: Consumer Crash

Answer:

```text
If processing happens before offset commit, the record can be processed again.
I therefore make critical business consumers idempotent.
```

---

# 261. Senior Interview: Exactly Once

Answer:

```text
I first define the boundary. Kafka transactions can provide stronger semantics
for supported Kafka-to-Kafka workflows, but external database effects still
need idempotency or transactional integration.
```

---

# 262. Senior Interview: Outbox

Answer:

```text
The outbox pattern stores the business update and event record in the same DB
transaction. A publisher then sends the outbox event to Kafka, avoiding the
common database/Kafka dual-write inconsistency.
```

---

# 263. Senior Interview: Hot Partition

Answer:

```text
I inspect key distribution and per-partition traffic. If one key is hot, I
redesign the key only after evaluating the ordering requirement, because
changing the key can change ordering behavior.
```

---

# 264. Senior Interview: Partition Count

Answer:

```text
I derive partitions from throughput, consumer parallelism, ordering and future
growth. I avoid both under-partitioning and excessive partition counts.
```

---

# 265. Senior Interview: Retention

Answer:

```text
Retention is selected from replay requirements, storage capacity, compliance,
cost and recovery needs. Longer retention is not automatically better.
```

---

# 266. Senior Interview: High Lag

Answer:

```text
I compare producer and consumer rates, identify hot partitions, inspect
consumer errors and rebalances, check downstream dependencies and inspect
broker CPU, disk and network before scaling.
```

---

# 267. Senior Interview: Kafka vs RabbitMQ

Answer:

```text
Kafka is particularly strong for durable event streams, partitioned
parallelism, retention and replay. RabbitMQ is strong for queue-based work
distribution and flexible routing. I select based on workload semantics.
```

---

# 268. Senior Interview: Security

Answer:

```text
I use TLS, authentication, least-privilege authorization, private networking,
restricted listeners, secret management and administrative auditing.
```

---

# 269. Senior Interview: DR

Answer:

```text
I separate in-region HA from regional DR. I define RPO/RTO, cross-region data
strategy, producer/consumer failover, offset handling and duplicate safety,
then rehearse the procedure.
```

---

# 270. Senior Interview: Production Ready

Answer:

```text
Production ready means the platform has demonstrated required performance,
availability, durability and recovery under realistic load and controlled
failures. A running broker alone is not production readiness.
```

---

# 271. Final Mental Model

```text
Requirements
 |
 +-- Throughput
 +-- Latency
 +-- Ordering
 +-- Retention
 +-- RPO/RTO
 |
Topic
 |
Partitions
 |
Replicas
 |
Brokers
 |
Failure Domains
 |
Consumer Groups
 |
Offsets
 |
Applications
 |
Business Side Effects
```

The production mindset is:

```text
Kafka reliability = broker design + partition design + replication + client
behavior + storage + networking + application correctness + observability.
```

---

# 272. Golden Rules

```text
1. Kafka is a distributed event log.
2. Topics are logical streams.
3. Partitions provide parallelism and ordering scope.
4. Ordering is per partition.
5. Keys influence partition placement.
6. Consumer groups share partition work.
7. Different consumer groups consume independently.
8. Offsets are partition-local positions.
9. Retention enables replay.
10. Replication provides redundancy.
11. Replication factor is not partition count.
12. Monitor ISR.
13. Monitor under-replicated partitions.
14. Monitor offline partitions.
15. Monitor consumer lag.
16. Monitor message age.
17. Avoid arbitrary partition counts.
18. Watch for hot keys.
19. Use stable event IDs.
20. Govern event schemas.
21. Use producer acknowledgements intentionally.
22. Evaluate producer idempotence.
23. Expect duplicate risk around ambiguous outcomes.
24. Make critical consumers idempotent.
25. Understand offset commit timing.
26. Define exactly-once boundaries.
27. Use outbox for database-to-Kafka dual writes.
28. Use inbox/idempotency for Kafka-to-database effects.
29. Bound retry behavior.
30. Design error topics explicitly.
31. Protect downstream systems.
32. HA is not automatically DR.
33. Spread replicas across failure domains.
34. Plan disk from retention and traffic.
35. Keep storage headroom.
36. Account for replication traffic.
37. Account for cross-AZ traffic.
38. Treat advertised listeners as critical networking configuration.
39. Secure listeners.
40. Apply least privilege.
41. Protect secrets.
42. Monitor broker and consumer health.
43. Test broker failure.
44. Test consumer failure.
45. Test AZ failure.
46. Test network failure.
47. Test disk pressure.
48. Test upgrades.
49. Test replay.
50. Test duplicate handling.
51. Test DR.
52. Measure recovery.
53. Assign topic ownership.
54. Treat topics as APIs.
55. Use Kafka-aware Kubernetes automation where appropriate.
56. Keep spare capacity for failure.
57. Design scaling with downstream capacity.
58. Do not confuse Kafka transactions with arbitrary database transactions.
59. Do not confuse replication with backup.
60. Design for failure, not only normal traffic.
```

# END OF 14-Kafka-Fundamentals.md


# 273. Production Drill 1: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 1.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 2: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 2.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 3: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 3.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 4: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 4.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 5: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 5.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 6: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 6.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 7: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 7.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 8: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 8.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 9: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 9.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 10: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 10.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 11: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 11.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 12: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 12.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 13: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 13.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 14: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 14.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 15: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 15.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 16: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 16.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 17: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 17.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 18: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 18.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 19: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 19.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 20: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 20.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 21: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 21.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 22: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 22.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 23: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 23.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 24: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 24.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 25: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 25.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 26: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 26.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 27: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 27.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 28: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 28.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 29: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 29.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 30: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 30.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 31: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 31.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 32: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 32.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 33: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 33.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 34: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 34.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 35: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 35.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 36: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 36.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 37: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 37.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 38: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 38.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 39: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 39.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 40: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 40.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 41: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 41.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 42: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 42.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 43: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 43.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 44: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 44.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 45: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 45.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 46: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 46.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 47: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 47.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 48: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 48.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 49: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 49.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 50: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 50.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 51: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 51.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 52: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 52.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 53: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 53.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 54: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 54.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 55: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 55.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 56: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 56.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 57: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 57.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 58: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 58.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 59: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 59.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 60: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 60.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 61: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 61.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 62: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 62.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 63: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 63.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 64: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 64.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 65: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 65.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 66: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 66.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 67: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 67.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 68: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 68.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 69: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 69.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 70: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 70.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 71: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 71.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 72: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 72.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 73: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 73.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 74: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 74.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 75: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 75.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 76: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 76.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 77: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 77.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 78: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 78.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 79: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 79.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```


# 273. Production Drill 80: Kafka Fundamentals Scenario

## Objective
Validate a production Kafka design for scenario 80.

## Investigation
```text
1. Establish producer rate.
2. Establish consumer rate.
3. Record lag and message age.
4. Record partition distribution.
5. Record broker CPU, memory, disk and network.
6. Record ISR and offline-partition status.
7. Record consumer-group membership.
8. Identify the failure domain.
9. Check recent changes.
10. Execute the controlled scenario.
11. Observe broker behavior.
12. Observe producer behavior.
13. Observe consumer behavior.
14. Observe downstream dependencies.
15. Measure recovery.
16. Validate duplicate handling.
17. Validate alerts.
18. Document root cause.
19. Correct the design gap.
20. Repeat the test.
```

## Decision Points
```text
partition distribution
replication health
consumer stability
storage capacity
network reachability
security configuration
downstream capacity
recovery time
```

## Success Criteria
```text
required availability is maintained
data loss remains within RPO
consumer processing recovers
no uncontrolled retry/replay storm occurs
duplicates do not create unsafe business effects
lag returns toward the defined SLO
operators can execute the runbook
```
