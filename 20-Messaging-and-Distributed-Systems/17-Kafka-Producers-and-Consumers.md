# 20-Messaging-and-Distributed-Systems

# 17-Kafka-Producers-and-Consumers

## Deep Production Guide

Kafka reliability is not achieved by configuring only the broker. The producer
and consumer clients are part of the distributed system.

```text
                 APPLICATION
                     |
          +----------+----------+
          |                     |
       PRODUCER              CONSUMER
          |                     |
          v                     ^
       Kafka Broker <------------+
          |
       Partition Log
```

A production design must answer:

```text
How is a record published?
How is publication confirmed?
What happens when the broker is slow?
What happens when the network fails?
How are duplicates handled?
When is a record considered processed?
When is its offset committed?
How does the consumer recover?
How is throughput controlled?
How are downstream dependencies protected?
```

---

# 1. Producer Mental Model

```text
Application
    |
serialize
    |
partition selection
    |
record accumulator
    |
batch
    |
compression
    |
network request
    |
Kafka leader
    |
acknowledgement
```

---

# 2. Consumer Mental Model

```text
Kafka partition
      |
     fetch
      |
consumer buffer
      |
poll
      |
application processing
      |
business side effect
      |
offset commit
```

---

# 3. Producer Responsibilities

A producer owns:

```text
serialization
key selection
partition selection
batching
compression
timeouts
acknowledgements
retry
idempotence
error handling
```

---

# 4. Consumer Responsibilities

A consumer owns:

```text
subscription
partition assignment
fetch
poll
processing
offset management
rebalancing
retry/error handling
idempotency
backpressure
```

---

# 5. Producer Record

Conceptually:

```text
key
value
headers
timestamp
```

---

# 6. Stable Event ID

Use a stable event ID for important business events.

```json
{
  "event_id": "evt-001",
  "event_type": "order.created",
  "payload": {}
}
```

---

# 7. Correlation ID

Use correlation IDs to connect related operations.

---

# 8. Trace Context

Propagate tracing metadata through Kafka headers where supported by the
application tracing architecture.

---

# 9. Serialization

Common formats:

```text
JSON
Avro
Protobuf
JSON Schema
```

Choose based on compatibility, performance and organizational tooling.

---

# 10. Serialization Failure

If serialization fails:

```text
application
   |
serialize
   X
error
```

The record never reaches Kafka.

Monitor application-side serialization failures separately from broker failures.

---

# 11. Producer Key

The key can determine partition placement.

For per-order ordering:

```text
key = order_id
```

---

# 12. Producer Partitioner

The producer determines the target partition according to its configured
partitioning behavior.

Understand the exact client/version behavior before relying on it.

---

# 13. Null Key

When no key is supplied, partition selection follows the producer/client's
configured behavior.

Do not assume a particular distribution algorithm without validating the
client version.

---

# 14. Producer Metadata

The producer obtains metadata about:

```text
topics
partitions
leaders
```

and uses it to route requests.

---

# 15. Metadata Refresh

The producer refreshes metadata when leadership or partition information
changes.

---

# 16. Unknown Topic

A producer may encounter an unknown topic if the topic does not exist or
creation/metadata propagation is not available.

Production systems should define topic provisioning rather than relying on
accidental auto-creation.

---

# 17. Topic Governance

Prefer:

```text
Git
 |
topic definition
 |
review
 |
automation
 |
Kafka
```

---

# 18. Producer Buffering

The producer can buffer records before sending them.

Benefits:

```text
batching
throughput
network efficiency
```

---

# 19. Record Accumulator

Conceptually:

```text
Application
 |
record
 |
accumulator
 |
batch
 |
request
```

---

# 20. Batching

Batching reduces request overhead.

---

# 21. Batch Size

Larger batches can improve throughput but can increase latency and memory use.

---

# 22. Batch Wait

A producer may wait briefly for more records to fill a batch.

---

# 23. Latency vs Throughput

```text
small batching
 -> lower latency
 -> potentially lower throughput

larger batching
 -> higher throughput
 -> potentially higher latency
```

Benchmark instead of guessing.

---

# 24. Compression

Common options include:

```text
gzip
snappy
lz4
zstd
```

depending on client and broker support.

---

# 25. Compression Trade-Off

```text
CPU
  |
  +---- compression
  |
  +---- lower network/storage
```

---

# 26. Compression Benchmark

Test with real:

```text
message sizes
payload distributions
traffic rates
CPU limits
```

---

# 27. Producer Acknowledgement

The producer acknowledgement configuration determines when Kafka considers
the request sufficiently acknowledged for the configured durability model.

---

# 28. Acks=0

The producer does not wait for a broker acknowledgement.

This can reduce latency but weakens delivery assurance.

---

# 29. Acks=1

The leader acknowledges the write according to the configured semantics.

Failure before replication can create durability risk.

---

# 30. Acks=all

The leader waits for the required in-sync replica conditions according to
Kafka's configuration.

This is commonly considered for critical workloads.

---

# 31. Acks and Min ISR

A strong acknowledgement configuration can be paired with a minimum ISR
policy to avoid accepting writes when insufficient replicas are in sync.

---

# 32. Producer Retry

Transient failures can trigger retries.

Examples:

```text
temporary network error
leader change
temporary broker unavailability
```

---

# 33. Retry Is Not Free

Retries consume:

```text
network
CPU
broker capacity
application time
```

---

# 34. Retry Storm

If a cluster is unhealthy:

```text
failure
 |
retry
 |
failure
 |
retry
```

can amplify the incident.

---

# 35. Retry Backoff

Use bounded retry/backoff behavior appropriate to the client and workload.

---

# 36. Retry Jitter

Jitter can reduce synchronized retries from many producers.

---

# 37. Producer Timeout

Different timeout settings govern different parts of the producer lifecycle.

Understand:

```text
request timeout
delivery timeout
connection timeout
```

rather than treating all timeouts as one setting.

---

# 38. Delivery Timeout

The overall delivery timeout bounds how long the producer attempts to deliver a
record under the configured retry behavior.

---

# 39. Timeout Relationship

Production tuning requires compatible timeout values.

Do not configure random timeout values independently.

---

# 40. Idempotent Producer

Idempotent producer support reduces duplicate records caused by producer
retries within its supported semantics.

---

# 41. Idempotence Is Not Global Exactly-Once

It does not automatically make:

```text
Kafka
+
database
+
external API
```

exactly once.

---

# 42. Producer Sequence

Kafka uses producer identity and sequencing mechanisms for idempotent
production.

Applications should use supported client configuration rather than manually
recreating those mechanisms.

---

# 43. Producer Session

Producer identity and recovery behavior matter when a producer restarts.

---

# 44. Producer Failure

Possible states:

```text
record never sent
record accepted
ack lost
record retried
```

The application should be prepared for ambiguous outcomes.

---

# 45. Ambiguous Outcome

Example:

```text
producer -> broker
           accepts
broker -> producer
           ACK
           X network failure
```

The producer may not know whether the broker accepted the record.

---

# 46. Stable Event IDs

Stable event IDs make reconciliation and downstream deduplication possible.

---

# 47. Transactional Producer

Kafka supports producer transactions for supported Kafka processing patterns.

---

# 48. Transactional ID

Transactional producers use a stable transactional identity to support
transaction semantics.

---

# 49. Transaction Scope

A Kafka transaction can atomically coordinate supported Kafka writes and
offset commits.

---

# 50. External Database

A Kafka transaction does not automatically include a relational database
transaction.

---

# 51. Database + Kafka

For:

```text
DB update
Kafka event
```

consider an outbox architecture.

---

# 52. Outbox

```text
Application
 |
DB transaction
 +-- business data
 +-- outbox event
 |
outbox publisher
 |
Kafka
```

---

# 53. Producer Error Classification

Classify:

```text
serialization
authentication
authorization
metadata
timeout
network
broker
```

---

# 54. Authentication Failure

Repeated retries do not fix invalid credentials.

---

# 55. Authorization Failure

A producer can connect successfully but still lack permission to write a topic.

---

# 56. Metadata Failure

Potential causes:

```text
DNS
listener
broker unavailable
controller issue
```

---

# 57. Network Failure

A producer must recover from temporary connectivity loss.

---

# 58. Broker Failure

Producer metadata should refresh and route traffic to the new leader where
possible.

---

# 59. Producer Monitoring

Track:

```text
record send rate
request rate
request latency
error rate
retry rate
batch size
compression ratio
buffer utilization
```

---

# 60. Producer SLO

Define:

```text
publish success rate
publish latency
```

for business-critical events.

---

# 61. Producer Backpressure

When Kafka is slow:

```text
producer buffer grows
```

If the application continues producing indefinitely, memory pressure can grow.

---

# 62. Buffer Exhaustion

When producer buffers cannot accept more records, the application may block
or fail according to client behavior.

---

# 63. Application Backpressure

The application should have a policy for:

```text
slow Kafka
full producer buffer
publish timeout
```

---

# 64. Consumer Mental Model

A consumer does not simply receive messages.

It:

```text
joins group
gets assignment
fetches records
polls records
processes records
commits offsets
```

---

# 65. Consumer Group

```text
group-id = payments
```

defines a logical consumption identity.

---

# 66. Group Membership

Members can:

```text
join
leave
crash
restart
```

---

# 67. Partition Assignment

The group coordinator and assignment strategy determine which consumer owns
which partitions.

---

# 68. One Partition, One Active Consumer

Within one consumer group, a partition has one active owner at a time.

---

# 69. Consumer Scaling

```text
partitions = 12
consumers = 12
```

can provide up to 12 active partition owners.

---

# 70. Consumers > Partitions

```text
partitions = 12
consumers = 20
```

means some consumers are idle.

---

# 71. Consumers < Partitions

```text
partitions = 12
consumers = 4
```

means each consumer can process multiple partitions.

---

# 72. Subscribe

Consumers can subscribe to topic patterns or explicit topics depending on
client design.

---

# 73. Assignment

After joining the group, the consumer receives partition assignments.

---

# 74. Poll

The application polls the consumer for records.

---

# 75. Poll Loop

Conceptually:

```text
while running:
    poll
    process
    commit
```

The real production implementation must account for processing time,
exceptions, rebalances and graceful shutdown.

---

# 76. Fetch

The consumer fetches batches of records from brokers.

---

# 77. Fetch Size

Fetch settings influence:

```text
network efficiency
memory
latency
throughput
```

---

# 78. Fetch Min Bytes

Waiting for a useful amount of data can improve efficiency but may increase
latency.

---

# 79. Fetch Max Wait

Controls how long the broker may wait for enough data under relevant fetch
behavior.

---

# 80. Consumer Record Batch

A poll can return multiple records from multiple partitions.

---

# 81. Processing Batch

Applications can process records:

```text
sequentially
parallel
transactionally
```

depending on correctness requirements.

---

# 82. Parallel Processing Risk

Parallel processing within one partition can break ordering.

---

# 83. Preserve Ordering

If strict partition ordering is required:

```text
P0
 |
record 1
 |
record 2
 |
record 3
```

process them in order.

---

# 84. Per-Key Ordering

If the key maps to one partition, partition-sequential processing can preserve
that key's order.

---

# 85. Consumer Concurrency

Increasing concurrency can improve throughput but may violate ordering or
overload dependencies.

---

# 86. Downstream Capacity

Example:

```text
Kafka can deliver 20,000 records/sec
Database can safely process 2,000/sec
```

The consumer should not blindly process at 20,000/sec.

---

# 87. Backpressure

Use:

```text
bounded concurrency
batching
rate limits
pause/resume where appropriate
```

---

# 88. Consumer Timeout Concepts

Consumer configuration includes several time-related controls.

Understand:

```text
poll interval
session timeout
heartbeat
request timeout
```

as different mechanisms.

---

# 89. Max Poll Interval

If processing takes too long between polls, the consumer may be considered
unhealthy and removed from the group.

---

# 90. Long Processing

For long processing:

```text
poll
 |
long task
 |
poll too late
```

can cause rebalances.

---

# 91. Long Task Solution

Possible approaches:

```text
reduce processing batch
parallelize safely
move long work to bounded worker pool
pause partitions carefully
```

while maintaining correct offset management.

---

# 92. Heartbeat

Heartbeats help maintain group membership.

---

# 93. Session Timeout

If the coordinator does not receive expected heartbeats within the session
window, the consumer can be considered failed.

---

# 94. Poll Interval vs Session Timeout

These solve different problems.

Do not tune one as if it were the other.

---

# 95. Consumer Rebalance

A rebalance redistributes partitions among group members.

---

# 96. Rebalance Triggers

Examples:

```text
consumer join
consumer leave
consumer crash
subscription change
partition change
```

---

# 97. Rebalance Impact

During reassignment, processing can pause or change ownership.

---

# 98. Rebalance Storm

Frequent rebalances can destroy throughput.

---

# 99. Rebalance Causes

Investigate:

```text
consumer crashes
long processing
network instability
deployment churn
incorrect timeout tuning
```

---

# 100. Cooperative Rebalancing

Cooperative assignment can reduce disruptive partition movement in supported
client configurations.

---

# 101. Static Membership

Static membership can reduce avoidable rebalances during transient consumer
restarts when correctly implemented.

---

# 102. Graceful Shutdown

A consumer should:

```text
stop intake
finish safe work
commit appropriate offsets
leave group
close
```

---

# 103. Kubernetes Shutdown

Use enough termination grace period for the consumer to finish its controlled
shutdown path.

---

# 104. Consumer Offset

The offset identifies the consumer's position in each partition.

---

# 105. Committed Offset

The committed offset is the durable group position used for recovery.

---

# 106. Auto Commit

Automatic commits are convenient but can produce incorrect processing semantics
if commit timing does not match business processing.

---

# 107. Manual Commit

Manual commit gives the application more control over the processing boundary.

---

# 108. At-Least-Once

Common pattern:

```text
poll
 |
process
 |
commit
```

Crash before commit:

```text
redelivery possible
```

---

# 109. At-Most-Once

Pattern:

```text
poll
 |
commit
 |
process
```

Crash after commit:

```text
record may not be processed
```

---

# 110. Exactly-Once

Exactly-once must be defined at a specific boundary.

---

# 111. Kafka-to-Kafka

Kafka transactions can provide strong exactly-once semantics for supported
Kafka-to-Kafka processing when correctly implemented.

---

# 112. Kafka-to-Database

For external database side effects:

```text
Kafka transaction != database transaction
```

Use application-level transactional/idempotency patterns.

---

# 113. Inbox

```text
event
 |
database transaction
 +-- processed_event
 +-- business update
 |
commit
 |
offset commit
```

---

# 114. Duplicate Processing

Duplicate processing can occur when:

```text
business update succeeds
offset commit fails
consumer crashes
```

---

# 115. Idempotency Key

Use:

```text
event_id
```

or a stable business identifier.

---

# 116. Idempotency Table

Example:

```text
processed_events
----------------
event_id
processed_at
result
```

---

# 117. Atomicity

Where possible:

```text
business update
+
event deduplication
```

should be inside one database transaction.

---

# 118. Consumer Exception

When processing fails, classify:

```text
transient
permanent
poison
```

---

# 119. Transient Error

Examples:

```text
temporary database outage
temporary API timeout
```

May be retried.

---

# 120. Permanent Error

Examples:

```text
invalid schema
invalid business data
unsupported event version
```

May need an error topic or DLQ strategy.

---

# 121. Poison Record

Repeatedly failing records require bounded handling.

---

# 122. Kafka Error Topic

A common pattern:

```text
main-topic
 |
consumer
 |
failure
 |
error-topic
```

---

# 123. Retry Topic

Example:

```text
orders
orders.retry.1m
orders.retry.10m
orders.dlq
```

---

# 124. Retry Ordering

Moving a record to a retry topic can allow later records from the original
partition to proceed.

This can violate strict ordering.

---

# 125. Retry Trade-Off

Choose explicitly between:

```text
strict ordering
```

and:

```text
forward progress
```

when the two conflict.

---

# 126. Consumer Pause

Consumers can pause partitions for controlled backpressure or error handling.

Use carefully because pausing too long can affect group stability or lag.

---

# 127. Consumer Resume

Resume only when the application and dependency can safely continue.

---

# 128. Poll Thread

Keep the consumer's polling/group-management behavior healthy.

---

# 129. Worker Pool

A bounded worker pool can separate polling from processing.

Architecture:

```text
Kafka consumer
 |
bounded queue
 |
worker pool
 |
database/API
```

---

# 130. Worker Pool Risk

If workers process out of order, offset commits must not move past unfinished
records.

---

# 131. Offset Commit with Parallel Workers

Commit only offsets for records whose processing state is safely known.

---

# 132. Out-of-Order Completion

Example:

```text
offset 100 -> slow
offset 101 -> fast
```

Committing 101 while 100 is still incomplete can skip 100 after a crash.

---

# 133. Safe Parallelism

Use per-partition sequencing or an offset-tracking mechanism that only commits
the highest contiguous successfully processed offset.

---

# 134. Per-Partition Worker

A simple ordering-preserving pattern:

```text
P0 -> worker 0
P1 -> worker 1
P2 -> worker 2
```

---

# 135. Key-Aware Worker

For more concurrency:

```text
partition
 |
key-aware dispatcher
 |
ordered workers
```

requires careful design.

---

# 136. Batch Processing

Batching can improve:

```text
database efficiency
network efficiency
CPU efficiency
```

---

# 137. Batch Failure

If a batch contains one invalid record, define whether to:

```text
retry whole batch
split batch
isolate bad record
```

---

# 138. Partial Batch Success

Do not commit the whole batch if only part of it has safely completed.

---

# 139. Database Batch

Kafka records can be grouped into database operations where transaction
semantics permit it.

---

# 140. API Batching

Batching external API requests can improve throughput but may increase
latency and failure scope.

---

# 141. Consumer Memory

Large fetch batches plus application buffers can increase memory usage.

---

# 142. Consumer OOM

Potential causes:

```text
large messages
large fetch sizes
unbounded worker queues
large application batches
```

---

# 143. Bounded Queues

Prefer bounded internal queues.

---

# 144. Unbounded Queue

```text
Kafka
 |
unbounded buffer
 |
memory
 |
OOM
```

---

# 145. Backpressure Chain

```text
Kafka
 |
consumer
 |
bounded workers
 |
database
```

If database slows, worker throughput slows, then consumer intake must be
controlled.

---

# 146. Consumer Lag as Buffer

Kafka retention can absorb temporary backlog.

---

# 147. Lag Recovery

If:

```text
incoming = 5,000/sec
processing = 7,000/sec
```

net drain:

```text
2,000/sec
```

---

# 148. Backlog Recovery

For:

```text
400,000 records
```

idealized recovery:

```text
400,000 / 2,000 = 200 seconds
```

assuming stable rates.

---

# 149. Real Recovery

Real recovery is affected by:

```text
record size
downstream latency
broker capacity
rebalance
CPU
network
```

---

# 150. Consumer Scaling

Scale consumers when:

```text
partitions available
downstream capacity available
broker capacity available
```

---

# 151. Scaling Beyond Partitions

If:

```text
partitions = 10
consumers = 30
```

20 consumers cannot create additional partition-level parallelism.

---

# 152. Increasing Partitions

Partition expansion may increase parallelism but can change key-to-partition
mapping.

---

# 153. Consumer Group Migration

Changing group ID creates a different consumption position.

---

# 154. Offset Reset

Offset reset is a production operation.

Possible goals:

```text
replay
skip corrupted range
recover after migration
```

---

# 155. Replay

Replay can create duplicate external effects.

---

# 156. Replay Rate

Throttle replay according to downstream capacity.

---

# 157. Replay Audit

Record:

```text
who
when
topic
partition
offset range
reason
```

---

# 158. Consumer Monitoring

Track:

```text
records consumed
records processed
processing latency
commit latency
poll latency
rebalance count
lag
```

---

# 159. Consumer Error Metrics

Track separately:

```text
deserialization errors
business errors
dependency errors
authorization errors
```

---

# 160. Lag Metrics

Measure:

```text
group
topic
partition
```

---

# 161. Message Age

Message age can be more meaningful than raw record count.

---

# 162. Consumer Health

A process can be alive while making no business progress.

Therefore:

```text
liveness != processing health
```

---

# 163. Business Progress

Monitor:

```text
orders processed
payments completed
notifications sent
```

---

# 164. Consumer Security

Consumers require:

```text
authentication
authorization
TLS
```

---

# 165. Consumer ACL

Grant read permission only to required topics and group IDs.

---

# 166. Producer ACL

Grant write permission only to required topics.

---

# 167. Consumer Credentials

Use secure secret management and rotation.

---

# 168. Kubernetes Consumer

```text
Deployment
 |
consumer Pods
 |
Kafka
```

---

# 169. Consumer Pod Scaling

Use HPA/KEDA or another controlled mechanism based on workload signals.

---

# 170. Pod Disruption

Use graceful shutdown and appropriate disruption controls for critical
consumer workloads.

---

# 171. Consumer Deployment

A rolling deployment can temporarily reduce group capacity.

Plan for enough replicas and partitions.

---

# 172. Readiness

A consumer should not be considered ready merely because its process started.
Validate the required initialization state.

---

# 173. Liveness

Avoid liveness checks that restart a consumer during normal long processing.

---

# 174. Startup

Use startup protection when initialization is slow.

---

# 175. Node Failure

Consumer Pods should reschedule onto healthy nodes.

---

# 176. AZ Failure

A consumer deployment should have replicas distributed across AZs for critical
workloads.

---

# 177. Consumer Affinity

Avoid concentrating all replicas of one critical group on one node.

---

# 178. Dependency Protection

Consumer scaling should be bounded by:

```text
database connections
API rate limits
cache capacity
```

---

# 179. Circuit Breaker

Use circuit breakers for unhealthy downstream services where appropriate.

---

# 180. Rate Limiting

Rate limiting prevents Kafka backlog recovery from overwhelming dependencies.

---

# 181. Retry Budget

Define a maximum retry workload rather than retrying indefinitely.

---

# 182. Retry Amplification

If each failed record is retried five times:

```text
1 original
+
5 retry attempts
=
up to 6 processing attempts
```

depending on the exact strategy.

---

# 183. Retry Storm Prevention

Use:

```text
backoff
jitter
bounded attempts
circuit breaker
```

---

# 184. Dead-Letter Strategy

A dead-letter topic should have:

```text
owner
retention
alert
inspection
replay
```

---

# 185. Schema Failure

A consumer should handle incompatible events without silently losing data.

---

# 186. Deserialization Failure

If a record cannot be deserialized, decide how the consumer avoids permanently
blocking the partition.

---

# 187. Poison Handling

Possible architecture:

```text
main
 |
consumer
 |
invalid
 |
error topic
 |
repair
 |
replay
```

---

# 188. Ordering During Error Handling

Moving one record to another topic changes the original partition ordering
relationship.

---

# 189. Consumer Shutdown

A robust shutdown:

```text
receive termination
 |
stop accepting new work
 |
finish bounded work
 |
commit safe offsets
 |
leave group
 |
close
```

---

# 190. Kubernetes Termination Grace

Set enough termination grace for the application to execute its shutdown
procedure.

---

# 191. Consumer Reconnect

Clients should recover from:

```text
broker restart
network interruption
leader change
```

---

# 192. Reconnect Storm

Many consumers reconnecting simultaneously can overload brokers.

Use sensible reconnect backoff and jitter.

---

# 193. Producer Reconnect

Apply similar protection to producers.

---

# 194. Connection Count

Monitor:

```text
connections
channels/sessions
```

according to the client architecture.

---

# 195. One Connection per Record

Avoid designs that create a new network connection for every record.

---

# 196. Connection Reuse

Long-lived producer/consumer clients are generally more efficient.

---

# 197. Producer Thread Safety

Use the client according to its documented thread-safety model.

Do not assume arbitrary shared-client behavior.

---

# 198. Consumer Thread Safety

Consumer APIs commonly require careful single-threaded or controlled access
patterns. Follow the exact client contract.

---

# 199. Producer Performance

Optimize in this order:

```text
measure
 |
identify bottleneck
 |
tune batching
 |
compression
 |
network
 |
broker
```

---

# 200. Consumer Performance

Optimize:

```text
fetch
batch
processing
database
downstream
```

as a complete pipeline.

---

# 201. Producer Throughput Test

Measure:

```text
records/sec
MB/sec
p95 latency
p99 latency
error rate
CPU
```

---

# 202. Consumer Throughput Test

Measure:

```text
records/sec
processing latency
lag
CPU
memory
downstream latency
```

---

# 203. End-to-End Test

Measure:

```text
publish timestamp
-
processing completion timestamp
```

---

# 204. Load Profile

Use realistic:

```text
average
peak
burst
large-message
skewed-key
```

traffic.

---

# 205. Partition Distribution Test

Verify that producer keys produce the intended distribution.

---

# 206. Hot-Key Test

Generate one dominant key and observe:

```text
partition load
consumer lag
broker load
```

---

# 207. Failure Test: Producer

Stop the producer network path and observe:

```text
buffer
timeouts
retries
recovery
```

---

# 208. Failure Test: Consumer

Kill a consumer and observe:

```text
rebalance
partition reassignment
lag
recovery
```

---

# 209. Failure Test: Broker

Stop a broker and observe:

```text
leader changes
producer behavior
consumer behavior
```

---

# 210. Failure Test: Downstream

Make the database/API unavailable.

Observe:

```text
consumer lag
retry rate
memory
dependency recovery
```

---

# 211. Failure Test: Slow Dependency

Introduce latency rather than complete failure.

This tests whether the consumer creates unsafe backlog amplification.

---

# 212. Failure Test: Poison Record

Introduce a known invalid record and verify controlled error handling.

---

# 213. Failure Test: Replay

Replay a small known range and verify:

```text
duplicates
ordering
downstream capacity
audit
```

---

# 214. Production Incident: Producer Errors

```text
1. Check authentication.
2. Check authorization.
3. Check metadata.
4. Check listener/DNS.
5. Check network.
6. Check broker health.
7. Check request latency.
8. Check retries.
```

---

# 215. Production Incident: Consumer Lag

```text
1. Compare producer and consumer rate.
2. Inspect partition-level lag.
3. Check consumer errors.
4. Check rebalances.
5. Check processing latency.
6. Check downstream dependencies.
7. Check broker resources.
8. Scale safely.
```

---

# 216. Production Incident: Duplicate Processing

```text
1. Find event ID.
2. Check consumer processing.
3. Check offset commit timing.
4. Check crash/rebalance.
5. Check database idempotency.
6. Check retry.
7. Prevent repeated business effect.
```

---

# 217. Production Incident: Consumer Restart Loop

Check:

```text
OOM
liveness probe
configuration
authentication
deserialization
dependency
```

---

# 218. Production Incident: OOM

Check:

```text
fetch size
record size
worker queue
batch size
heap
off-heap memory
```

---

# 219. Production Incident: Rebalance Storm

Check:

```text
Pod restarts
deployment
poll interval
processing time
network
session timeout
```

---

# 220. Production Incident: Database Overload

Reduce:

```text
consumer concurrency
batch rate
retries
```

and protect the database.

---

# 221. Producer/Consumer Contract

Define:

```text
schema
key
ordering
delivery
retry
error
retention
```

---

# 222. API-Like Thinking

A topic is a distributed API.

Producer and consumer teams should agree on the contract.

---

# 223. Versioning

Use explicit event/schema versions for breaking evolution.

---

# 224. Compatibility

Test new producer versions against old consumers and vice versa where required.

---

# 225. Contract Testing

Automate schema compatibility checks before deployment.

---

# 226. Observability Correlation

Use:

```text
event_id
correlation_id
trace_id
```

to connect producer and consumer logs.

---

# 227. Consumer Log

Example:

```text
event_id=evt-001
topic=orders
partition=3
offset=18291
processing_started=...
processing_completed=...
```

Do not log sensitive payloads unnecessarily.

---

# 228. Producer Log

Record safe metadata:

```text
event_id
topic
partition
timestamp
```

---

# 229. Offset Visibility

Expose current processed/committed position where useful for operations.

---

# 230. Consumer Lag Alert

Alert based on:

```text
business SLO
message age
```

rather than arbitrary record counts.

---

# 231. Producer Latency Alert

Alert on sustained publish latency degradation.

---

# 232. Error Rate Alert

Separate:

```text
transient errors
permanent errors
authentication errors
application errors
```

---

# 233. Consumer Processing SLO

Example:

```text
99% of payment events processed within X seconds
```

---

# 234. Producer Publish SLO

Example:

```text
99.9% of accepted business events publish successfully within X ms
```

---

# 235. Security

Use:

```text
TLS
authentication
authorization
secret management
network restrictions
```

---

# 236. Least Privilege

Producer:

```text
write selected topics
```

Consumer:

```text
read selected topics
```

Admin:

```text
management
```

---

# 237. Credential Rotation

Test rotation with real client behavior before enabling automatic production
rotation.

---

# 238. Kubernetes Secrets

Do not hardcode Kafka credentials into container images or Git repositories.

---

# 239. External Secrets

An external secret-management integration can provide controlled credential
delivery.

---

# 240. Consumer Autoscaling

Potential signal:

```text
consumer lag
```

But also consider:

```text
message age
processing latency
downstream health
```

---

# 241. Scaling Oscillation

If lag rises and falls rapidly, aggressive autoscaling can create:

```text
Pod churn
rebalances
unstable throughput
```

---

# 242. Cooldown

Use stabilization/cooldown behavior appropriate to the workload.

---

# 243. Maximum Consumer Count

Set an upper bound based on:

```text
partitions
database
API
broker
```

---

# 244. Minimum Consumer Count

Maintain enough consumers for required availability and baseline throughput.

---

# 245. Consumer Availability

Distribute replicas across:

```text
nodes
AZs
```

for critical workloads.

---

# 246. Consumer Pod Resources

Set CPU and memory requests/limits based on measured processing behavior.

---

# 247. CPU Throttling

Excessive CPU limits can increase processing latency and consumer instability.

---

# 248. Memory Limits

Too-small memory limits can cause OOMKilled events.

---

# 249. JVM Consumer

For JVM clients, account for:

```text
heap
off-heap
buffers
application memory
```

---

# 250. Go/Node/Python Consumers

Account for runtime memory and worker concurrency, not only Kafka client
buffers.

---

# 251. Graceful Deployment

A safe deployment should:

```text
start new consumer
 |
verify readiness
 |
drain old consumer
 |
rebalance
 |
complete
```

---

# 252. Blue-Green Consumer Deployment

Changing group IDs can accidentally cause duplicate full-topic consumption.

Use controlled offset/cutover strategy.

---

# 253. Canary Consumer

A canary can consume a controlled workload before full rollout.

---

# 254. Consumer Version Compatibility

Deploying a new consumer version should account for:

```text
schema
business semantics
offset behavior
retry
```

---

# 255. Consumer Rollback

Rollback is not simply redeploying an old binary if offsets or schemas changed.

---

# 256. Producer Rollback

If the new producer emitted incompatible events, rollback alone may not repair
already-published data.

---

# 257. Event Contract Migration

Use:

```text
new version
compatibility window
consumer migration
deprecation
```

---

# 258. Production Producer Checklist

```text
[ ] key strategy
[ ] serialization
[ ] batching
[ ] compression
[ ] acknowledgements
[ ] retries
[ ] idempotence
[ ] timeouts
[ ] authentication
[ ] authorization
[ ] metrics
[ ] tracing
[ ] backpressure
[ ] shutdown
```

---

# 259. Production Consumer Checklist

```text
[ ] group ID
[ ] partition capacity
[ ] fetch tuning
[ ] poll loop
[ ] processing concurrency
[ ] commit strategy
[ ] idempotency
[ ] retry
[ ] error topic
[ ] lag monitoring
[ ] rebalancing
[ ] shutdown
[ ] resource limits
[ ] dependency protection
```

---

# 260. Senior Interview: Producer Architecture

Answer:

```text
I design the producer around key selection, serialization, batching,
compression, acknowledgement, retries and idempotence. For critical events I
use stable event IDs and monitor publish success and latency. I also define
behavior when Kafka becomes slow so the application does not accumulate
unbounded memory or retry traffic.
```

---

# 261. Senior Interview: Consumer Architecture

Answer:

```text
I design the consumer around group membership, partition parallelism, fetch
behavior, bounded processing concurrency, offset commits, idempotency and
failure handling. I commit only after the required business processing is
safe, then monitor lag and message age.
```

---

# 262. Senior Interview: Why Not Auto Commit?

Answer:

```text
Auto commit can move the consumer position independently of the actual
business processing boundary. For critical side effects I prefer explicit
offset control so processing and commit semantics are intentional.
```

---

# 263. Senior Interview: Duplicate Processing

Answer:

```text
Kafka consumers can process a record again when processing succeeds but the
offset commit does not complete. I use stable event IDs and an idempotent
business transaction so duplicate delivery does not create duplicate effects.
```

---

# 264. Senior Interview: Long Processing

Answer:

```text
I first measure processing time and verify it against the consumer's group
management settings. If work is long, I use smaller poll batches or a bounded
worker design while ensuring offsets are committed only for contiguous safe
processing.
```

---

# 265. Senior Interview: Consumer Scaling

Answer:

```text
I scale up to useful partition parallelism and only as far as downstream
dependencies can tolerate. If more parallelism is required, I evaluate
partition expansion and its ordering implications.
```

---

# 266. Senior Interview: High Lag

Answer:

```text
I compare producer rate with processing rate, inspect partition-level lag,
identify hot partitions, check consumer errors/rebalances, and inspect
database/API latency before scaling.
```

---

# 267. Senior Interview: Producer Retry

Answer:

```text
Retries are useful for transient failures but can amplify an outage. I use
bounded retry behavior, backoff and idempotence where appropriate, and monitor
retry rate separately.
```

---

# 268. Senior Interview: Acks

Answer:

```text
I select acknowledgement behavior from the durability requirement. Critical
workloads commonly use strong acknowledgement semantics combined with
appropriate replication and ISR controls.
```

---

# 269. Senior Interview: Exactly Once

Answer:

```text
I define the boundary. Kafka transactions can provide strong Kafka-to-Kafka
semantics, but external database/API effects need their own idempotency or
transactional integration.
```

---

# 270. Senior Interview: Outbox

Answer:

```text
The outbox pattern stores the business change and event record in one database
transaction, then publishes the event asynchronously. It prevents the common
case where the database commits but Kafka publication fails.
```

---

# 271. Senior Interview: Hot Key

Answer:

```text
I inspect partition-level traffic and key distribution. If one key dominates,
I either redesign the key with an explicit ordering trade-off or introduce a
controlled sharding/resequencing mechanism.
```

---

# 272. Senior Interview: Error Topic

Answer:

```text
I use an error topic for records that cannot safely progress, with ownership,
retention, alerting and controlled replay. I explicitly document whether moving
a failed record changes ordering semantics.
```

---

# 273. Senior Interview: Kafka on Kubernetes

Answer:

```text
For consumers I use Deployments with topology distribution, resource requests,
graceful shutdown and controlled autoscaling. For Kafka itself I use a
Kafka-aware stateful platform. The consumer lifecycle must be coordinated with
partition ownership and downstream capacity.
```

---

# 274. Senior Interview: Production Readiness

Answer:

```text
A producer/consumer system is production ready when delivery semantics,
failure behavior, scaling limits, security, observability, recovery and
business correctness have been tested under realistic load.
```

---

# 275. End-to-End Producer Flow

```text
Application
    |
create event
    |
assign event_id
    |
serialize
    |
select key
    |
select partition
    |
batch
    |
compress
    |
send
    |
broker leader
    |
replication
    |
ack
    |
application confirmation
```

---

# 276. End-to-End Consumer Flow

```text
Kafka
 |
partition
 |
fetch
 |
poll
 |
bounded processing
 |
database/API
 |
successful business effect
 |
offset commit
 |
next record
```

---

# 277. Failure-Safe Consumer Flow

```text
poll
 |
process
 |
 +---- success ----> commit
 |
 +---- transient --> bounded retry
 |
 +---- permanent -> error topic
 |
 +---- poison ----> controlled isolation
```

---

# 278. Final Mental Model

```text
                  PRODUCER
                     |
          +----------+----------+
          |          |          |
        Key       Batch      Retry
          |          |          |
          +----------+----------+
                     |
                   Kafka
                     |
                Partitions
                     |
          +----------+----------+
          |                     |
       Consumer Group       Consumer Group
          |                     |
       Processing            Processing
          |                     |
       Idempotency           Idempotency
          |                     |
       Offset                Offset
       Commit                Commit
          |                     |
       Business             Business
       Systems              Systems
```

---

# 279. Golden Rules

```text
1. Producer and consumer clients are part of the distributed system.
2. Choose keys from ordering requirements.
3. Understand the exact partitioner behavior.
4. Use stable event IDs.
5. Use correlation IDs and trace context.
6. Use explicit schema governance.
7. Batch for throughput.
8. Do not over-batch latency-sensitive workloads.
9. Benchmark compression.
10. Choose acknowledgements from durability requirements.
11. Pair strong acknowledgement with replication/ISR design.
12. Bound retries.
13. Use backoff and jitter.
14. Monitor retry amplification.
15. Evaluate idempotent producers.
16. Do not confuse idempotence with end-to-end exactly once.
17. Understand ambiguous publication outcomes.
18. Use outbox for database-to-Kafka dual writes.
19. Treat the consumer poll loop as critical.
20. Understand poll interval and session timeout separately.
21. Avoid long blocking processing in the poll loop.
22. Use bounded worker pools.
23. Never commit past unfinished records.
24. Preserve per-partition ordering when required.
25. Scale consumers only up to useful partition parallelism.
26. Protect downstream systems.
27. Use bounded queues.
28. Avoid unbounded application buffering.
29. Monitor consumer lag.
30. Monitor message age.
31. Monitor partition-level lag.
32. Monitor rebalances.
33. Monitor processing latency.
34. Classify transient and permanent errors.
35. Bound retries.
36. Design error topics.
37. Design replay.
38. Audit replay operations.
39. Expect duplicate processing.
40. Make critical business effects idempotent.
41. Gracefully shut down consumers.
42. Give Kubernetes consumers enough termination time.
43. Distribute critical consumer replicas across AZs.
44. Use resource requests based on measurements.
45. Avoid aggressive liveness probes.
46. Use controlled autoscaling.
47. Bound maximum consumer count.
48. Account for broker capacity.
49. Account for dependency capacity.
50. Test producer failure.
51. Test consumer failure.
52. Test broker failure.
53. Test downstream failure.
54. Test poison records.
55. Test replay.
56. Test duplicate handling.
57. Test deployments.
58. Test scaling.
59. Test recovery.
60. Design for predictable behavior under failure.
```

# END OF 17-Kafka-Producers-and-Consumers.md
# 280. Production Exercise 1: producer batching optimization

## Objective

Validate **producer batching optimization** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 2: producer compression benchmark

## Objective

Validate **producer compression benchmark** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 3: producer acknowledgement comparison

## Objective

Validate **producer acknowledgement comparison** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 4: producer retry storm simulation

## Objective

Validate **producer retry storm simulation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 5: producer idempotence validation

## Objective

Validate **producer idempotence validation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 6: ambiguous publish reconciliation

## Objective

Validate **ambiguous publish reconciliation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 7: producer metadata failure

## Objective

Validate **producer metadata failure** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 8: producer authentication failure

## Objective

Validate **producer authentication failure** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 9: producer authorization failure

## Objective

Validate **producer authorization failure** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 10: producer network interruption

## Objective

Validate **producer network interruption** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 11: producer buffer exhaustion

## Objective

Validate **producer buffer exhaustion** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 12: consumer group scaling

## Objective

Validate **consumer group scaling** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 13: consumer rebalance analysis

## Objective

Validate **consumer rebalance analysis** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 14: consumer long-processing test

## Objective

Validate **consumer long-processing test** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 15: poll interval failure

## Objective

Validate **poll interval failure** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 16: consumer session timeout analysis

## Objective

Validate **consumer session timeout analysis** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 17: consumer graceful shutdown

## Objective

Validate **consumer graceful shutdown** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 18: consumer duplicate processing

## Objective

Validate **consumer duplicate processing** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 19: manual offset commit test

## Objective

Validate **manual offset commit test** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 20: at-least-once processing

## Objective

Validate **at-least-once processing** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 21: at-most-once processing

## Objective

Validate **at-most-once processing** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 22: Kafka transaction exercise

## Objective

Validate **Kafka transaction exercise** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 23: Kafka-to-database idempotency

## Objective

Validate **Kafka-to-database idempotency** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 24: inbox pattern implementation

## Objective

Validate **inbox pattern implementation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 25: outbox pattern implementation

## Objective

Validate **outbox pattern implementation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 26: parallel worker ordering

## Objective

Validate **parallel worker ordering** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 27: contiguous offset commit

## Objective

Validate **contiguous offset commit** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 28: consumer backpressure

## Objective

Validate **consumer backpressure** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 29: bounded worker queue

## Objective

Validate **bounded worker queue** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 30: consumer memory pressure

## Objective

Validate **consumer memory pressure** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 31: consumer OOM investigation

## Objective

Validate **consumer OOM investigation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 32: database throttling

## Objective

Validate **database throttling** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 33: API rate limiting

## Objective

Validate **API rate limiting** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 34: circuit breaker behavior

## Objective

Validate **circuit breaker behavior** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 35: retry topic design

## Objective

Validate **retry topic design** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 36: error topic design

## Objective

Validate **error topic design** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 37: poison record isolation

## Objective

Validate **poison record isolation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 38: controlled replay

## Objective

Validate **controlled replay** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 39: partition-level lag investigation

## Objective

Validate **partition-level lag investigation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 40: hot partition investigation

## Objective

Validate **hot partition investigation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 41: hot key investigation

## Objective

Validate **hot key investigation** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 42: consumer autoscaling

## Objective

Validate **consumer autoscaling** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 43: KEDA scaling limits

## Objective

Validate **KEDA scaling limits** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 44: consumer Pod disruption

## Objective

Validate **consumer Pod disruption** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 45: consumer AZ distribution

## Objective

Validate **consumer AZ distribution** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 46: consumer rolling deployment

## Objective

Validate **consumer rolling deployment** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 47: consumer rollback

## Objective

Validate **consumer rollback** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 48: schema compatibility

## Objective

Validate **schema compatibility** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 49: event version migration

## Objective

Validate **event version migration** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 50: producer/consumer contract test

## Objective

Validate **producer/consumer contract test** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 51: end-to-end latency measurement

## Objective

Validate **end-to-end latency measurement** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 52: producer throughput benchmark

## Objective

Validate **producer throughput benchmark** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 53: consumer throughput benchmark

## Objective

Validate **consumer throughput benchmark** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 54: large-message benchmark

## Objective

Validate **large-message benchmark** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 55: burst traffic benchmark

## Objective

Validate **burst traffic benchmark** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 56: broker failure during publishing

## Objective

Validate **broker failure during publishing** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 57: broker failure during consumption

## Objective

Validate **broker failure during consumption** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 58: dependency outage recovery

## Objective

Validate **dependency outage recovery** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 59: reconnect storm analysis

## Objective

Validate **reconnect storm analysis** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

# 280. Production Exercise 60: production readiness review

## Objective

Validate **production readiness review** under a controlled, production-like workload.

## Procedure

```text
1. Establish a healthy baseline.
2. Record producer rate.
3. Record consumer rate.
4. Record partition distribution.
5. Record lag and message age.
6. Record broker resource usage.
7. Record downstream resource usage.
8. Execute the scenario.
9. Observe producer behavior.
10. Observe consumer behavior.
11. Observe offset behavior.
12. Observe duplicate behavior.
13. Measure recovery.
14. Validate alerts.
15. Validate the operational runbook.
16. Record the architectural lesson.
17. Convert the lesson into automation or policy.
```

## Success Criteria

```text
business correctness preserved
delivery semantics remain within the defined contract
no uncontrolled retry amplification
no unsafe duplicate effect
lag returns toward the expected SLO
resources remain within limits
operators can explain the behavior
```

