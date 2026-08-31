# 20-Messaging-and-Distributed-Systems

# 08-RabbitMQ-Consumers-and-Producers

## Purpose

RabbitMQ producers and consumers are the application-facing side of the
messaging system.

A production architecture must treat them as distributed systems components,
not simply as client libraries.

The complete path is:

```text
Producer Application
       |
       | connection
       v
    Channel
       |
       | publish
       v
    Exchange
       |
       | route
       v
      Queue
       |
       | deliver
       v
    Consumer
       |
       | process
       v
 Business System
```

Reliability depends on every stage.

A successful production design must answer:

```text
How does the producer connect?
How are channels managed?
How are messages published?
How is publication confirmed?
What happens when the network fails?
What happens when the broker restarts?
How does the consumer receive messages?
How is concurrency controlled?
How is prefetch selected?
When is a message acknowledged?
What happens if processing fails?
How is graceful shutdown implemented?
How is duplicate processing prevented?
How is backpressure handled?
How are producers and consumers monitored?
How are they deployed on Kubernetes?
How are capacity and failure domains designed?
```

---

# 1. Producer and Consumer Roles

A producer:

```text
creates and publishes messages
```

A consumer:

```text
receives and processes messages
```

They have different reliability responsibilities.

---

# 2. Producer Responsibility

A production producer should consider:

```text
connection
channel
exchange
routing key
message properties
publisher confirms
mandatory publishing
retry
backpressure
idempotency
observability
```

---

# 3. Consumer Responsibility

A production consumer should consider:

```text
connection
channel
queue
consumer registration
prefetch
processing concurrency
acknowledgement
retry
idempotency
graceful shutdown
observability
```

---

# 4. Producer-Consumer Decoupling

RabbitMQ allows:

```text
Producer rate != Consumer rate
```

Example:

```text
Producer: 10,000 msg/s
Consumer: 7,000 msg/s
```

The difference becomes queue backlog.

---

# 5. Queue as Buffer

```text
Producer
   |
   v
Exchange
   |
 Queue
   |
Consumer
```

The queue absorbs temporary differences between producer and consumer speed.

It is not an unlimited buffer.

---

# 6. Producer Connection

A producer establishes a connection to RabbitMQ.

Conceptually:

```text
Application
   |
TCP/TLS connection
   |
RabbitMQ
```

Connections are relatively expensive compared with channels.

---

# 7. Connection vs Channel

A useful model:

```text
Connection
 |
+-- Channel 1
+-- Channel 2
+-- Channel 3
```

Applications commonly reuse connections and create channels according to
library and workload requirements.

---

# 8. Why Channels Exist

Channels provide logical communication sessions over a connection.

They avoid requiring a separate network connection for every operation.

---

# 9. Too Many Connections

Bad architecture:

```text
1000 application threads
 |
1000 RabbitMQ connections
```

This can create:

```text
memory pressure
connection management overhead
network overhead
broker resource pressure
```

---

# 10. Too Few Connections

One connection may become a bottleneck or single client-side failure domain
for a high-throughput application.

Choose connection architecture based on:

```text
throughput
library behavior
concurrency
failure isolation
```

---

# 11. Producer Channel Strategy

Possible:

```text
one publishing channel per worker
```

or a controlled pool, depending on the client library.

Do not blindly share channels across incompatible concurrent operations.

---

# 12. Consumer Channel Strategy

Consumers generally have their own channel context.

A common pattern:

```text
Consumer process
 |
connection
 |
consumer channel
 |
queue
```

---

# 13. Connection Pooling

For high-throughput applications, connection pooling may be useful.

However:

```text
pool size != thread count
```

Tune based on actual workload.

---

# 14. Connection Lifecycle

Production lifecycle:

```text
create
 |
authenticate
 |
open
 |
use
 |
detect failure
 |
reconnect
 |
recreate channels
 |
redeclare required topology
 |
resume
```

---

# 15. Connection Failure

Example:

```text
Producer
 |
publish
 |
network failure
```

The producer may not know whether RabbitMQ received the message.

This is a fundamental distributed-systems ambiguity.

---

# 16. The Ambiguous Publish

Sequence:

```text
Producer -> RabbitMQ: publish
RabbitMQ: accepts
RabbitMQ -> Producer: confirm
X network failure
```

The producer never receives the confirmation.

It may retry.

Now:

```text
original message
+
retry
```

can create duplicates.

---

# 17. Duplicate-Safe Producer Design

Use:

```text
event_id
message_id
publisher confirms
idempotent downstream processing
```

where business correctness requires it.

---

# 18. Publisher Confirms

Publisher confirms allow producers to know whether RabbitMQ accepted a
publication according to its confirm semantics.

They are critical for important asynchronous workflows.

---

# 19. Confirm Mode

Conceptually:

```text
Producer
 |
publish
 |
RabbitMQ
 |
confirm
 |
Producer
```

---

# 20. Synchronous Confirm

A simple implementation can wait for confirmation after publishing.

Advantage:

```text
simple reasoning
```

Disadvantage:

```text
lower throughput
```

if every message waits independently.

---

# 21. Asynchronous Confirms

A high-throughput publisher can track many outstanding publications and process
confirmations asynchronously.

Conceptually:

```text
Publish 1 ----\
Publish 2 -----+--> RabbitMQ
Publish 3 ----/
                  |
             confirmations
                  |
             producer tracker
```

---

# 22. Confirm Tracking

Track:

```text
sequence number
message ID
publication timestamp
confirmation status
```

This allows failures to be reconciled.

---

# 23. Confirm Failure

When a publication is not successfully confirmed:

```text
identify message
 |
retry/persist/alert
```

The correct action depends on business criticality.

---

# 24. Confirm Batch

Applications can publish multiple messages before waiting for confirmation.

This improves throughput but increases:

```text
in-flight messages
recovery complexity
memory usage
```

---

# 25. Confirm Window

Define a maximum outstanding publication window.

Example:

```text
10,000 in-flight messages
```

When the window is full:

```text
pause or slow producer
```

This creates client-side backpressure.

---

# 26. Mandatory Publishing

Mandatory publishing allows the producer to detect publications that have no
matching queue route.

```text
Producer
 |
mandatory publish
 |
Exchange
 |
no matching queue
 |
return
 |
Producer
```

---

# 27. Confirm vs Mandatory

Important distinction:

```text
publisher confirm
=
broker publication confirmation

mandatory return
=
no matching route detected
```

Use both when the workload requires strong routing visibility.

---

# 28. Producer Reliability Stack

For critical messages:

```text
Durable exchange
+
Durable queue
+
Persistent message
+
Publisher confirms
+
Mandatory publishing where required
+
Idempotent consumers
```

---

# 29. Persistent Messages

Message persistence affects how messages survive broker restart when stored in
durable queues.

It should be considered together with:

```text
durable topology
queue type
replication
```

---

# 30. Producer Routing

Producer must select:

```text
exchange
routing key
```

Example:

```text
orders.events
order.created
```

---

# 31. Producer Routing Contract

A producer should not randomly construct routing keys.

Define:

```text
domain
event
version if needed
```

---

# 32. Producer Configuration

Typical configuration:

```text
RABBITMQ_HOST
RABBITMQ_PORT
RABBITMQ_VHOST
RABBITMQ_USERNAME
RABBITMQ_PASSWORD
EXCHANGE
```

Secrets must not be embedded in source code.

---

# 33. Secret Management

Production options include:

```text
Kubernetes Secrets
external secret manager
secret injection mechanism
```

Use the organization's approved mechanism.

---

# 34. TLS

Production RabbitMQ connections may use TLS.

```text
Producer
 |
TLS
 |
RabbitMQ
```

Validate:

```text
certificate
hostname
trust chain
protocol configuration
```

---

# 35. TLS Failure

A producer may fail to connect because:

```text
expired certificate
wrong CA
hostname mismatch
TLS configuration mismatch
```

Monitor certificate expiry.

---

# 36. Authentication

Producer authentication should use a dedicated identity.

Avoid:

```text
admin credentials
```

for application runtime.

---

# 37. Least Privilege

A producer typically needs:

```text
write access to required exchange(s)
```

It should not automatically receive:

```text
administrator
management
unrelated queue access
```

---

# 38. Consumer Permissions

Consumers need access appropriate to their queue and topology operations.

Avoid granting broad permissions just to "make it work."

---

# 39. Producer Application Startup

Startup should:

```text
load configuration
 |
connect
 |
authenticate
 |
open channel
 |
validate/deploy required topology
 |
start publishing
```

Topology ownership should be explicit.

---

# 40. Consumer Startup

Consumer startup:

```text
load config
 |
connect
 |
open channel
 |
declare/validate queue
 |
configure QoS/prefetch
 |
register consumer
 |
begin processing
```

---

# 41. Consumer Registration

The consumer registers interest in a queue.

Conceptually:

```text
Queue
 |
Consumer registration
 |
Consumer
```

---

# 42. Push-Based Delivery

RabbitMQ commonly pushes available messages to consumers.

The consumer does not need to continuously poll the queue in the traditional
sense.

---

# 43. Consumer Callback

Conceptually:

```text
message arrives
 |
callback
 |
business logic
 |
ACK/NACK
```

---

# 44. Consumer Processing Boundary

The safest acknowledgement boundary is generally after the business operation
has completed successfully.

```text
receive
 |
validate
 |
process
 |
persist
 |
ACK
```

---

# 45. ACK Before Processing

Dangerous:

```text
receive
 |
ACK
 |
process
 |
crash
```

The message may be lost from the queue even though the business operation did
not complete.

---

# 46. ACK After Processing

Safer:

```text
receive
 |
process
 |
business success
 |
ACK
```

If the consumer crashes before ACK:

```text
redelivery
```

may occur.

---

# 47. At-Least-Once Processing

A typical reliable RabbitMQ consumer provides:

```text
at-least-once processing
```

This means duplicates are possible.

---

# 48. Idempotent Consumer

Consumer must tolerate:

```text
same event
received twice
```

Example:

```text
event_id = evt-123
```

Store processing state.

---

# 49. Inbox Pattern

Conceptual:

```text
Message
 |
check inbox
 |
already processed?
 |             |
yes            no
 |             |
skip        process
               |
            record ID
```

This helps prevent duplicate business effects.

---

# 50. Database Unique Constraint

Example:

```text
UNIQUE(event_id)
```

can protect against duplicate processing when the business transaction and
deduplication state are designed correctly.

---

# 51. Idempotency Key

For external APIs:

```text
idempotency-key = event_id
```

when the external provider supports idempotency.

---

# 52. Consumer Processing Transaction

Ideal conceptual flow:

```text
Message
 |
Database transaction
 |
business update
 |
processed-event record
 |
COMMIT
 |
ACK
```

This reduces the window between business completion and message acknowledgement.

---

# 53. ACK Is Not Database Commit

Do not assume:

```text
ACK
=
database committed
```

They are separate systems.

---

# 54. Commit Then ACK

Common pattern:

```text
DB COMMIT
 |
ACK
```

If ACK fails after commit:

```text
message may be redelivered
```

Therefore idempotency remains important.

---

# 55. ACK Then Commit

Bad for critical processing:

```text
ACK
 |
DB commit fails
```

The message may not return.

---

# 56. Prefetch

Prefetch controls how many unacknowledged messages can be outstanding for a
consumer/channel according to the configured QoS semantics.

---

# 57. Why Prefetch Matters

Too low:

```text
consumer waits frequently
lower throughput
```

Too high:

```text
large in-flight set
memory pressure
uneven work distribution
slower recovery
```

---

# 58. Prefetch Example

```text
prefetch = 1
```

means very limited outstanding unacknowledged work.

Useful for:

```text
strict work fairness
slow/expensive tasks
```

but potentially lower throughput.

---

# 59. High Prefetch

Example:

```text
prefetch = 1000
```

can improve throughput for some workloads.

But if each message is large or processing is slow:

```text
memory usage
```

can become significant.

---

# 60. Prefetch Is a Capacity Control

Think:

```text
in-flight work
```

rather than just a performance knob.

---

# 61. Prefetch and Processing Time

If average processing time is:

```text
100 ms
```

and concurrency is sufficient, throughput can increase with enough in-flight
work to keep consumers busy.

Do not choose values without measuring.

---

# 62. Prefetch and Fairness

If one consumer prefetches a large number of messages:

```text
Consumer A: 100 messages
Consumer B: 5 messages
```

A may hold much more work.

When processing times vary, this can reduce fairness.

---

# 63. Long-Running Tasks

For:

```text
10-minute jobs
```

avoid huge prefetch values.

Otherwise a consumer can reserve a large amount of work it cannot process
quickly.

---

# 64. Short Tasks

For:

```text
5 ms jobs
```

a somewhat higher prefetch may improve throughput.

Measure CPU, memory and latency.

---

# 65. Prefetch and Ordering

High concurrency can make completion order different from delivery order.

If ordering is important:

```text
limit concurrency
or
partition by key
```

as appropriate.

---

# 66. Consumer Concurrency

Possible architecture:

```text
One process
 |
+-- worker 1
+-- worker 2
+-- worker 3
```

Each worker processes messages.

---

# 67. Concurrency vs Replicas

Two different scaling dimensions:

```text
replicas
```

and:

```text
workers per replica
```

Example:

```text
5 pods
x
4 workers/pod
=
20 processing workers
```

---

# 68. Horizontal Scaling

Increase:

```text
consumer replicas
```

when workload grows.

---

# 69. Vertical Scaling

Increase:

```text
CPU
memory
```

per consumer.

Use when individual message processing is resource-heavy.

---

# 70. Consumer Scaling Formula

Approximate capacity:

```text
throughput =
workers × messages/worker/second
```

Example:

```text
10 workers
x
100 msg/s
=
1000 msg/s
```

Real performance depends on downstream dependencies.

---

# 71. Downstream Bottleneck

RabbitMQ consumer scaling cannot overcome:

```text
database max connections
external API rate limit
CPU saturation
```

Scale the whole workflow.

---

# 72. Consumer Backpressure

When downstream capacity decreases:

```text
consumer processing slows
 |
queue backlog increases
```

The backlog is a useful buffer but should be bounded operationally.

---

# 73. Producer Backpressure

If queue growth becomes dangerous:

```text
consumer capacity
       |
       v
queue depth
       |
       v
producer throttling
```

Application-level rate control may be required.

---

# 74. Queue Depth

Monitor:

```text
ready messages
```

and:

```text
unacknowledged messages
```

They mean different things.

---

# 75. Ready Messages

Ready:

```text
waiting for consumers
```

High ready count usually indicates insufficient processing capacity or
excessive producer rate.

---

# 76. Unacknowledged Messages

Unacknowledged:

```text
delivered to consumers
but not yet acknowledged
```

High unacked count can indicate:

```text
slow processing
large prefetch
stuck consumer
```

---

# 77. Oldest Message Age

Queue depth alone is insufficient.

Measure:

```text
age of oldest message
```

because:

```text
10,000 messages
```

may be healthy at high throughput but dangerous if they are hours old.

---

# 78. Consumer Lag

A practical application-level measure:

```text
current time - event creation time
```

This captures business latency better than queue depth alone.

---

# 79. Consumer Throughput

Monitor:

```text
messages processed/s
```

Compare with:

```text
messages published/s
```

---

# 80. Backlog Growth

If:

```text
publish rate > consume rate
```

backlog grows.

If:

```text
consume rate > publish rate
```

backlog shrinks.

---

# 81. Queue Drain Time

Approximate:

```text
drain time =
backlog / net drain rate
```

Example:

```text
1,000,000 backlog
net drain = 5,000/s

drain ≈ 200 seconds
```

---

# 82. Consumer Scaling Decision

Before scaling:

```text
measure CPU
measure memory
measure downstream
measure queue age
measure processing latency
```

Do not automatically add pods without understanding the bottleneck.

---

# 83. KEDA-Style Scaling

Kubernetes event-driven autoscaling can use RabbitMQ queue metrics as a scaling
signal when the chosen integration supports it.

Conceptually:

```text
Queue depth
 |
autoscaler
 |
consumer replicas
```

---

# 84. Autoscaling Risk

If scaling only on queue depth:

```text
queue grows
 |
scale consumers
 |
database overload
 |
consumer slows
 |
queue grows more
 |
scale again
```

Use safe scaling limits and downstream-aware controls.

---

# 85. Scale-to-Zero

For batch workloads, scale-to-zero can reduce cost.

For latency-sensitive workloads:

```text
cold-start latency
```

may be unacceptable.

---

# 86. Consumer Graceful Shutdown

On termination:

```text
stop accepting new work
 |
finish in-flight work
 |
ACK successful messages
 |
NACK/recover unfinished work where appropriate
 |
close consumer
 |
close channel
 |
close connection
```

---

# 87. Kubernetes SIGTERM

Kubernetes sends:

```text
SIGTERM
```

before termination.

Consumer must handle it.

---

# 88. Termination Grace Period

Set enough time for:

```text
longest expected processing
```

plus cleanup.

---

# 89. Forced Termination

If the pod receives:

```text
SIGKILL
```

before completing processing:

```text
unacknowledged messages
```

may be redelivered.

Again:

```text
idempotency
```

is essential.

---

# 90. Readiness Probe

A consumer should not become Ready before:

```text
RabbitMQ connection
channel
queue
consumer registration
```

are healthy when readiness is meant to represent processing capability.

---

# 91. Liveness Probe

Liveness should detect genuine process failure.

Avoid:

```text
RabbitMQ temporarily unavailable
=
restart immediately
```

unless intentionally designed.

Otherwise restart storms can occur.

---

# 92. Startup Probe

Slow-starting applications may need startup protection so Kubernetes does not
kill them before initialization completes.

---

# 93. Consumer Pod Lifecycle

```text
Pending
 |
Starting
 |
Connect
 |
Register
 |
Ready
 |
Processing
 |
SIGTERM
 |
Drain
 |
Exit
```

---

# 94. Rolling Deployment

If consumers share a queue:

```text
old pods
+
new pods
```

can process concurrently.

This supports rolling updates.

---

# 95. Consumer Version Compatibility

During rolling deployment:

```text
Consumer v1
Consumer v2
```

may receive the same event.

The message schema must be compatible.

---

# 96. Producer Version Compatibility

Similarly:

```text
Producer v1
Producer v2
```

may publish different versions during rollout.

Consumers should tolerate the transition.

---

# 97. Schema Evolution

Prefer:

```text
additive fields
optional fields
backward-compatible changes
```

over breaking changes.

---

# 98. Poison Message

A poison message always fails processing.

Bad:

```text
receive
 |
NACK requeue
 |
receive
 |
NACK requeue
 |
forever
```

---

# 99. Retry Boundary

Use bounded retries:

```text
attempt 1
attempt 2
attempt 3
 |
DLQ
```

---

# 100. Consumer Retry

Retry should distinguish:

```text
transient failure
permanent failure
```

Examples:

Transient:

```text
database timeout
HTTP 503
```

Permanent:

```text
invalid schema
invalid business state
```

---

# 101. Retry Backoff

Use:

```text
exponential backoff
```

Example:

```text
1s
5s
30s
5m
```

with jitter where appropriate.

---

# 102. Retry Storm

If 10,000 messages fail simultaneously and all retry immediately:

```text
dependency
 |
10,000 retries
 |
dependency overload
 |
more failures
```

Use delayed/bounded retry.

---

# 103. Requeue

Immediate requeue can be useful for temporary conditions, but uncontrolled
requeue can create hot loops.

---

# 104. Dead-Letter

After retry limit:

```text
queue
 |
DLX
 |
DLQ
```

---

# 105. DLQ Consumer

A DLQ should have:

```text
owner
monitoring
retention
replay procedure
```

A DLQ is not a permanent garbage bin.

---

# 106. Consumer Error Classification

Classify errors:

```text
validation
business
dependency
network
authentication
timeout
resource
unknown
```

This enables correct retry decisions.

---

# 107. Consumer Timeouts

Every external dependency should have controlled timeouts.

Avoid:

```text
consumer waits forever
```

because the message remains unacknowledged.

---

# 108. Database Timeout

If database latency spikes:

```text
consumer processing slows
 |
unacked grows
 |
queue backlog
```

Alert before the system collapses.

---

# 109. External API Timeout

A consumer calling an external API must handle:

```text
timeout
5xx
429
authentication failure
```

Use retry only where appropriate.

---

# 110. Consumer Circuit Breaker

A circuit breaker can stop repeatedly calling an unhealthy dependency.

```text
Consumer
 |
Circuit Breaker
 |
External API
```

Messages can then follow controlled retry/DLQ behavior.

---

# 111. Consumer Rate Limiting

If external API allows:

```text
100 req/s
```

but consumer fleet can generate:

```text
1000 req/s
```

the consumer must limit itself.

---

# 112. Consumer Concurrency Budget

Define:

```text
max workers
max outstanding tasks
max DB connections
max external requests
```

Do not scale consumers beyond downstream capacity.

---

# 113. Database Connection Pool

Consumer concurrency:

```text
50 workers
```

with database pool:

```text
10 connections
```

may cause contention.

Align:

```text
worker concurrency
DB pool
DB capacity
```

---

# 114. Consumer CPU Bound Work

For CPU-heavy tasks:

```text
CPU
 |
consumer
```

Use enough CPU requests/limits and avoid excessive concurrency.

---

# 115. Consumer Memory Bound Work

Large messages or in-memory processing can increase:

```text
heap
RSS
GC pressure
```

Prefetch must account for message size.

---

# 116. Memory Estimation

Approximate in-flight memory:

```text
prefetch
×
average message size
×
workers
```

Example:

```text
100 prefetch
x 1 MB
x 20 workers
=
~2 GB raw payload
```

Actual memory can be significantly higher due to object and runtime overhead.

---

# 117. Large Message Risk

Large messages increase:

```text
network
memory
broker storage
serialization cost
consumer latency
```

Prefer object storage for very large payloads when appropriate.

---

# 118. Payload Reference Pattern

Instead of:

```text
RabbitMQ -> 20 MB payload
```

use:

```text
RabbitMQ
 |
object reference
 |
Object Storage
```

Consumer retrieves payload securely.

---

# 119. Producer Serialization

Serialization choices affect:

```text
CPU
size
compatibility
schema evolution
```

JSON is easy to inspect but can be larger than binary formats.

---

# 120. Compression

Compression can reduce:

```text
network bandwidth
storage
```

but increases:

```text
CPU
latency
```

Benchmark.

---

# 121. Batching

Producer batching can improve throughput.

Trade-offs:

```text
latency
memory
failure recovery
```

---

# 122. Consumer Batching

Consumers can process batches where business logic supports it.

Example:

```text
10 messages
 |
one database batch
```

This may reduce database overhead.

---

# 123. Batch Failure

If a batch contains:

```text
1 invalid
9 valid
```

define whether:

```text
all fail
```

or:

```text
partial success
```

is supported.

---

# 124. Batch Acknowledgement

Batch processing requires careful acknowledgement semantics.

Do not ACK messages whose business effects did not complete.

---

# 125. Producer Throughput

Factors:

```text
serialization
network
channel strategy
confirm mode
batching
message size
exchange routing
broker capacity
```

---

# 126. Producer Latency

Measure:

```text
publish call latency
confirm latency
```

Separate them.

---

# 127. Confirm Latency

High confirm latency can indicate:

```text
broker load
disk pressure
network latency
resource alarms
```

depending on workload.

---

# 128. Consumer Latency

Break down:

```text
delivery wait
processing
database
external API
ACK
```

---

# 129. End-to-End Latency

Example:

```text
event created
 |
published
 |
routed
 |
queued
 |
delivered
 |
processed
 |
business effect
```

Track total latency.

---

# 130. Producer Metrics

Recommended application metrics:

```text
publish_total
publish_failed_total
publish_confirmed_total
publish_confirm_latency
publish_returned_total
connection_reconnect_total
```

---

# 131. Consumer Metrics

Recommended:

```text
messages_received_total
messages_processed_total
messages_failed_total
ack_total
nack_total
processing_latency
redelivery_total
```

---

# 132. Queue Metrics

Monitor:

```text
ready
unacked
depth
oldest age
publish rate
deliver rate
ack rate
```

---

# 133. Business Metrics

Technical metrics are insufficient.

Track:

```text
orders processed
payments completed
notifications sent
```

---

# 134. Correlation ID

Producer should propagate:

```text
correlation_id
```

through the event workflow.

---

# 135. Message ID

Assign stable:

```text
message_id
```

for tracking.

---

# 136. Event ID

Business event should have:

```text
event_id
```

This is useful for deduplication.

---

# 137. Trace Context

Propagate distributed tracing context through message headers where supported.

---

# 138. Structured Logging

Producer log:

```text
exchange
routing_key
message_id
correlation_id
publish_result
```

Consumer log:

```text
queue
message_id
event_id
processing_result
duration
```

---

# 139. Do Not Log Secrets

Never log:

```text
password
access token
TLS private key
sensitive payload
```

unless explicitly sanitized and approved.

---

# 140. Consumer Health

A consumer is healthy only if:

```text
connected
+
receiving
+
processing
+
acknowledging
```

Connection alone is not enough.

---

# 141. Stuck Consumer

Symptoms:

```text
consumer connected
unacked grows
processing rate approaches zero
```

Possible causes:

```text
deadlock
dependency hang
CPU saturation
memory pressure
thread exhaustion
```

---

# 142. Zombie Consumer

A process may appear alive but stop processing.

Use:

```text
processing heartbeat
last successful processing timestamp
```

for detection.

---

# 143. Consumer Lag Alert

Alert when:

```text
oldest message age > SLO
```

rather than only when:

```text
queue depth > threshold
```

---

# 144. Producer Stall

Producer may remain connected but stop publishing.

Monitor:

```text
expected publication rate
last successful publication
```

---

# 145. Producer Zombie

Application process is alive but its publishing loop is stuck.

Use application-level health signals.

---

# 146. Connection Recovery

After reconnect:

```text
recreate channel
restore confirms
restore consumer registration
restore QoS
restore required topology
```

Do not assume the old channel state automatically remains.

---

# 147. Consumer Recovery

A robust consumer should recover:

```text
connection
channel
queue consumer
```

after broker/network failure.

Test the actual client library behavior.

---

# 148. Producer Recovery

Producer should recover:

```text
connection
channel
publisher state
```

and avoid uncontrolled duplicate publication.

---

# 149. Reconnect Storm

If 1000 pods reconnect simultaneously:

```text
RabbitMQ
 |
1000 connection attempts
```

can create load spikes.

Use:

```text
exponential backoff
jitter
```

where supported.

---

# 150. Broker Restart Scenario

```text
Broker restart
 |
connections drop
 |
producers reconnect
 |
consumers reconnect
 |
topology restored
 |
traffic resumes
```

Measure recovery time.

---

# 151. Consumer Deployment Storm

Rolling out 500 consumers at once can create:

```text
connection spike
consumer registration spike
CPU spike
```

Use controlled rollout.

---

# 152. Producer Deployment Storm

Similarly:

```text
500 producers
 |
reconnect
 |
publish burst
```

can overload RabbitMQ.

Use deployment controls and startup jitter where useful.

---

# 153. Kubernetes PodDisruptionBudget

For important consumer workloads, use PDBs to prevent excessive simultaneous
voluntary disruption.

---

# 154. Topology Spread

Spread consumer pods across:

```text
nodes
AZs
failure domains
```

when required.

---

# 155. Anti-Affinity

Avoid placing all critical consumers on one Kubernetes node.

---

# 156. Resource Requests

Set realistic:

```text
CPU requests
memory requests
```

so consumers receive predictable scheduling.

---

# 157. Resource Limits

Limits can protect cluster capacity but may cause:

```text
OOMKilled
CPU throttling
```

if set incorrectly.

Profile before selecting values.

---

# 158. HPA vs Queue Autoscaling

CPU-based HPA:

```text
CPU -> replicas
```

Queue-based scaling:

```text
backlog -> replicas
```

For message consumers, queue-aware scaling can be more directly aligned with
workload.

---

# 159. Multi-Metric Scaling

A mature system may consider:

```text
queue depth
oldest age
CPU
memory
downstream capacity
```

rather than one signal.

---

# 160. Consumer Maximum Replicas

Always consider:

```text
max replicas
```

because unlimited scaling can overload downstream systems.

---

# 161. Producer Rate Limit

Producer can use:

```text
token bucket
leaky bucket
fixed rate
adaptive rate
```

where appropriate.

---

# 162. Adaptive Backpressure

Producer can reduce rate when:

```text
queue age rises
broker resource pressure rises
downstream capacity drops
```

---

# 163. Queue Thresholds

Define:

```text
warning
critical
emergency
```

based on:

```text
queue age
queue depth
business SLO
drain capacity
```

---

# 164. Capacity Calculation

Suppose:

```text
incoming = 5,000/s
consumer capacity = 4,000/s
```

backlog grows:

```text
1,000/s
```

At this rate:

```text
600,000 messages
```

accumulate in ten minutes.

---

# 165. Recovery Calculation

If traffic returns to:

```text
2,000/s
```

and consumers can process:

```text
5,000/s
```

net drain:

```text
3,000/s
```

A backlog of:

```text
600,000
```

takes approximately:

```text
200 seconds
```

to drain.

---

# 166. Consumer Capacity Planning

Plan for:

```text
normal
peak
burst
dependency degradation
node/AZ loss
deployment
```

---

# 167. N+1 Consumer Capacity

If normal capacity is:

```text
10 workers
```

do not necessarily run exactly 10.

Plan redundancy:

```text
12 or more
```

based on failure requirements.

---

# 168. AZ Failure Capacity

If one AZ fails:

```text
capacity remaining >= required workload
```

where business requirements demand it.

---

# 169. Consumer Connection Placement

Spread consumers across RabbitMQ endpoints according to the deployment and
client architecture.

Avoid accidental single-endpoint dependency.

---

# 170. Load Balancer and Long Connections

Long-lived RabbitMQ connections mean a load balancer does not necessarily
rebalance an already-established connection.

This matters during scaling and node failure.

---

# 171. DNS

Use stable DNS/service discovery.

Avoid hard-coded broker pod IPs.

---

# 172. Kubernetes Service

Conceptually:

```text
Consumer Pod
 |
RabbitMQ Service
 |
RabbitMQ Cluster
```

The service provides stable discovery.

---

# 173. NetworkPolicy

Restrict RabbitMQ access to authorized application namespaces/workloads.

---

# 174. Security Groups

In AWS, restrict network access so only required workloads can reach RabbitMQ.

---

# 175. Secret Rotation

When credentials rotate:

```text
new secret
 |
consumer reconnect
 |
old credential removed
```

Plan for connection refresh.

---

# 176. Certificate Rotation

TLS certificates must be rotated without unnecessary service interruption.

Test the actual client behavior.

---

# 177. Producer/Consumer Testing

Test:

```text
unit
integration
contract
load
failure
recovery
security
```

---

# 178. Producer Unit Tests

Test:

```text
routing key
exchange
headers
message serialization
error handling
```

---

# 179. Consumer Unit Tests

Test:

```text
valid message
invalid message
dependency failure
ACK
NACK
retry
idempotency
```

---

# 180. Integration Tests

Verify:

```text
producer -> exchange -> queue -> consumer
```

---

# 181. Routing Contract Test

Publish:

```text
test.event
```

and verify expected queue receives it.

---

# 182. Failure Test

Simulate:

```text
RabbitMQ unavailable
```

and verify producer/consumer recovery.

---

# 183. Duplicate Test

Deliver the same:

```text
event_id
```

twice.

Verify:

```text
one business effect
```

---

# 184. Redelivery Test

Force consumer failure before ACK.

Verify:

```text
message redelivered
```

and business operation remains safe.

---

# 185. Graceful Shutdown Test

Terminate consumer during processing.

Verify:

```text
no message loss
message eventually processed
```

---

# 186. Load Test

Test:

```text
normal
2x peak
burst
long-running jobs
large messages
```

---

# 187. Dependency Failure Test

Simulate:

```text
database slow
database unavailable
API 503
API 429
```

Verify retry/backpressure.

---

# 188. Consumer Autoscaling Test

Increase queue backlog.

Verify:

```text
autoscaler reacts
pods scale
database remains healthy
backlog drains
pods scale down
```

---

# 189. Producer Failure Test

Kill producer after publication but before confirmation.

Verify:

```text
recovery
duplicate handling
```

---

# 190. Broker Restart Test

Restart broker during active workload.

Measure:

```text
connection recovery
publication recovery
consumer recovery
message integrity
```

---

# 191. Chaos Engineering

Test realistic failure domains:

```text
pod
node
AZ
network
broker
dependency
```

---

# 192. Consumer Error Budget

Define:

```text
processing failure rate
processing latency
queue age
```

against business SLOs.

---

# 193. Producer SLO

Example:

```text
99.99% of valid publications successfully confirmed.
```

---

# 194. Consumer SLO

Example:

```text
99% of critical messages processed within 30 seconds.
```

---

# 195. End-to-End SLO

Better:

```text
event generated
 |
published
 |
routed
 |
processed
 |
business effect
```

within the required latency.

---

# 196. Producer Incident: Confirm Failures

Symptoms:

```text
confirm failure spike
```

Check:

```text
broker health
connection
channel
resource alarms
network
permissions
```

---

# 197. Producer Incident: Unroutable Spike

Check:

```text
exchange
routing key
binding
vhost
topology deployment
```

---

# 198. Producer Incident: Connection Storm

Symptoms:

```text
connection count spike
CPU spike
```

Check:

```text
deployment
reconnect loop
backoff
health probes
```

---

# 199. Consumer Incident: Queue Backlog

Check:

```text
producer rate
consumer rate
consumer count
processing latency
downstream dependencies
```

---

# 200. Consumer Incident: Unacked Spike

Check:

```text
prefetch
processing latency
dependency timeouts
deadlocks
consumer health
```

---

# 201. Consumer Incident: Redelivery Spike

Check:

```text
consumer crashes
ACK failures
NACK/requeue behavior
network
application exceptions
```

---

# 202. Consumer Incident: CPU Saturation

Check:

```text
message complexity
serialization
business logic
worker count
CPU limits
```

---

# 203. Consumer Incident: Memory Growth

Check:

```text
prefetch
message size
worker concurrency
memory leaks
batch size
```

---

# 204. Consumer Incident: Database Overload

Symptoms:

```text
DB latency
consumer latency
queue backlog
```

Do not blindly scale consumers.

---

# 205. Consumer Incident: External API Rate Limit

Symptoms:

```text
429
retry growth
queue backlog
```

Mitigate with:

```text
rate limiting
bounded retry
backoff
```

---

# 206. Consumer Incident: Poison Message

Symptoms:

```text
same message repeatedly fails
```

Move to controlled DLQ path.

---

# 207. Consumer Incident: Zombie

Symptoms:

```text
pod Running
connection open
no processing
```

Use:

```text
processing heartbeat
last-success timestamp
```

and restart only when appropriate.

---

# 208. Consumer Incident: Shutdown Storm

If many consumers terminate together:

```text
messages requeued/redelivered
connection churn
load spike
```

Use:

```text
PDB
controlled rollout
graceful shutdown
```

---

# 209. Producer Architecture Pattern

```text
Application
    |
Producer Module
    |
Connection Manager
    |
Publishing Channel
    |
Publisher Confirm Tracker
    |
RabbitMQ Exchange
```

---

# 210. Consumer Architecture Pattern

```text
RabbitMQ Queue
      |
Consumer Connection
      |
Consumer Channel
      |
Prefetch
      |
Worker Pool
      |
Business Handler
      |
Database/API
      |
ACK
```

---

# 211. Reliable Consumer Pattern

```text
Message
 |
Validate
 |
Deduplicate
 |
Business Transaction
 |
Commit
 |
ACK
```

---

# 212. Reliable Producer Pattern

```text
Business Event
 |
Assign event_id
 |
Serialize
 |
Publish
 |
Confirm
 |
Record success
```

For stronger consistency between database state and publication, use an
outbox pattern.

---

# 213. Transactional Outbox

```text
Application
 |
DB Transaction
 |
+--> Business State
+--> Outbox Event
          |
       Publisher
          |
       RabbitMQ
```

The business transaction and event creation become atomic in the same database
transaction.

---

# 214. Outbox Publisher

```text
Outbox Table
 |
poll/stream
 |
RabbitMQ
 |
confirm
 |
mark published
```

Failures can cause repeated publication, so consumers remain idempotent.

---

# 215. Consumer Inbox

```text
RabbitMQ
 |
Inbox
 |
Business DB
```

Can help make duplicate processing safe.

---

# 216. Outbox + Inbox

Strong distributed pattern:

```text
Service A
 |
DB + Outbox
 |
RabbitMQ
 |
Service B
 |
Inbox + DB
```

This avoids requiring a distributed transaction across the two services.

---

# 217. Exactly-Once Claims

Be careful with:

```text
exactly once
```

Across a distributed system, exactly-once business effect usually requires
idempotency and transactional design rather than relying on a single broker
feature.

---

# 218. At-Least-Once Design

A practical architecture:

```text
at-least-once delivery
+
idempotent processing
=
effectively-once business result
```

when correctly implemented.

---

# 219. Consumer Business Transaction

Example:

```text
OrderCreated
 |
DB transaction
 |
create inventory reservation
 |
record event_id
 |
commit
 |
ACK
```

---

# 220. External Side Effect

If processing calls an external system:

```text
consume
 |
external API
 |
ACK
```

a crash after the API succeeds but before ACK can cause a duplicate API call.

Use provider idempotency if available.

---

# 221. Idempotent External API

```text
event_id
 |
idempotency key
 |
external provider
```

Repeated calls produce one logical effect.

---

# 222. Consumer State Machine

Useful states:

```text
RECEIVED
VALIDATING
PROCESSING
COMMITTED
ACKED
FAILED
RETRYING
DLQ
```

This improves observability.

---

# 223. Producer State Machine

Useful:

```text
CREATED
SERIALIZED
PUBLISHED
CONFIRMED
RETURNED
FAILED
RETRYING
```

---

# 224. Correlation

Track:

```text
request_id
correlation_id
event_id
message_id
```

across producer and consumer.

---

# 225. Message Metadata

Useful properties:

```text
message_id
correlation_id
content_type
timestamp
headers
delivery mode
```

Use standardized conventions.

---

# 226. Consumer Header Inspection

Consumers can inspect:

```text
retry count
source
version
trace context
tenant metadata
```

where these are part of the contract.

---

# 227. Do Not Trust Headers Blindly

Treat externally supplied metadata as untrusted input.

Validate:

```text
tenant
version
source
```

when security or business logic depends on it.

---

# 228. Input Validation

Consumers must validate:

```text
required fields
types
ranges
event version
business invariants
```

before executing side effects.

---

# 229. Invalid Message Handling

If schema is invalid:

```text
do not endlessly retry
```

Usually route to:

```text
DLQ
```

with enough metadata for diagnosis.

---

# 230. Consumer Authorization

Even internal messages may require authorization decisions.

Do not assume:

```text
message arrived
=
authorized operation
```

for sensitive workflows.

---

# 231. Multi-Tenant Consumer

Validate tenant scope before processing:

```text
message tenant
 |
consumer authorization
 |
business operation
```

---

# 232. Consumer Isolation

Separate workloads when:

```text
criticality differs
processing time differs
security differs
scaling differs
```

---

# 233. Dedicated Queue

A dedicated queue gives:

```text
independent backlog
independent scaling
independent failure boundary
```

---

# 234. Shared Queue

Shared queues are useful when consumers provide interchangeable worker capacity.

```text
Queue
 |
+--> Worker A
+--> Worker B
+--> Worker C
```

---

# 235. Shared Queue Scaling

More workers can increase throughput until:

```text
broker
CPU
DB
API
```

becomes the bottleneck.

---

# 236. Consumer Competition

Consumers on the same queue compete for messages.

This is different from multiple queues bound to one exchange.

---

# 237. Queue vs Exchange Scaling

Exchange fanout:

```text
one publication
 ->
multiple queues
```

Shared queue:

```text
one queue
 ->
multiple workers
```

These solve different scaling problems.

---

# 238. Consumer Workload Isolation

Avoid placing:

```text
10ms tasks
+
10min tasks
```

on the same queue unless intentionally designed.

Long jobs can create unfairness and capacity issues.

---

# 239. Separate Work Queues

Use:

```text
fast.queue
slow.queue
batch.queue
critical.queue
```

when workload characteristics differ.

---

# 240. Consumer Priority

RabbitMQ and application-level priority mechanisms exist, but separate queues
often provide clearer operational isolation.

---

# 241. Queue Affinity

For stateful processing:

```text
customer
 |
same logical worker/shard
```

may be needed.

Design routing and consumer concurrency together.

---

# 242. Producer Ordering

A single producer can publish messages sequentially, but end-to-end ordering
still depends on routing and consumer architecture.

---

# 243. Consumer Ordering

If strict ordering is required:

```text
single active processing path
```

or:

```text
partition by key
```

may be necessary.

---

# 244. Parallel Processing

Parallelism:

```text
worker A -> event 1
worker B -> event 2
```

improves throughput but may violate global ordering.

---

# 245. Per-Key Parallelism

A scalable design:

```text
key A -> shard 1
key B -> shard 2
key C -> shard 3
```

Each shard can process sequentially while shards operate concurrently.

---

# 246. Producer Batching and Ordering

Batching should preserve required ordering semantics.

Do not assume transport batching automatically guarantees business order.

---

# 247. Consumer Batch Processing

Batching may improve:

```text
database writes
network calls
CPU efficiency
```

but complicates partial failure.

---

# 248. Partial Batch Failure

Design explicitly:

```text
success subset
failure subset
retry subset
```

or:

```text
all-or-nothing
```

depending on business transaction support.

---

# 249. Consumer Retry Metadata

Track:

```text
attempt count
first failure
last failure
reason
```

This makes DLQ investigation easier.

---

# 250. Retry Reason

Example:

```text
database_timeout
api_429
validation_error
```

Use structured values rather than arbitrary text.

---

# 251. Consumer DLQ Metadata

Include enough information to identify:

```text
original exchange
routing key
queue
message ID
event ID
failure reason
attempt count
```

without exposing secrets.

---

# 252. Replay Workflow

```text
DLQ
 |
inspect
 |
root cause fixed
 |
sample replay
 |
verify
 |
controlled replay
```

---

# 253. Replay Rate

Do not replay:

```text
1 million messages
```

instantly.

Use controlled rate to protect downstream dependencies.

---

# 254. Consumer Retry Budget

Define maximum:

```text
attempts
time
```

before DLQ.

---

# 255. Retry vs DLQ Decision

Transient:

```text
retry
```

Permanent:

```text
DLQ
```

Unknown:

```text
bounded retry + investigation
```

---

# 256. Consumer Error Observability

Metric:

```text
consumer_failures_total{reason="database_timeout"}
```

helps identify systemic dependency problems.

---

# 257. Producer Error Observability

Metric:

```text
publisher_failures_total{reason="connection"}
```

helps separate application and broker issues.

---

# 258. Alert on Symptoms

Useful alerts:

```text
queue age
confirm failures
unroutable messages
consumer processing rate
redelivery
DLQ growth
```

---

# 259. Avoid Alert Noise

Do not alert solely on:

```text
queue depth = 1
```

unless business semantics require it.

Use sustained thresholds and SLOs.

---

# 260. Runbook

Every critical consumer should have a runbook covering:

```text
restart
scale
pause
resume
DLQ
replay
dependency failure
RabbitMQ outage
```

---

# 261. Pause Consumer

Sometimes pausing processing is safer than continuing during a dependency
outage.

Example:

```text
database corruption incident
```

Stop consumers to prevent further writes.

---

# 262. Resume Consumer

After dependency recovery:

```text
verify health
 |
resume
 |
monitor backlog
```

---

# 263. Consumer Drain

Before maintenance:

```text
stop new processing
 |
wait for in-flight
 |
ACK completed
 |
exit
```

---

# 264. Queue Drain

If migrating a queue:

```text
new consumers
 |
old queue drains
 |
verify zero traffic
 |
retire
```

---

# 265. Producer Drain

Before shutting down a producer:

```text
stop new events
 |
flush outstanding publications
 |
wait for confirms
 |
close
```

---

# 266. Producer Shutdown

Graceful sequence:

```text
stop accepting new work
 |
flush publisher buffer
 |
wait for confirms
 |
close channel
 |
close connection
```

---

# 267. Producer Crash

Outstanding unconfirmed messages require recovery from:

```text
outbox
local durable buffer
upstream retry
```

if loss cannot be accepted.

---

# 268. Local Buffer Trade-Off

A local persistent buffer can protect publication during short broker outages.

Trade-offs:

```text
complexity
disk management
duplicate handling
```

---

# 269. Outbox Preferred

For database-backed business events, transactional outbox is often safer than
an ad-hoc local file buffer because business state and event creation can be
committed together.

---

# 270. Producer Idempotency

Producer itself may generate duplicate events if:

```text
retry
restart
outbox replay
```

Use stable event IDs.

---

# 271. Consumer Deduplication Store

Possible:

```text
Redis
database
durable inbox
```

Choose based on consistency requirements.

Do not use an ephemeral cache as the only deduplication mechanism for critical
business effects.

---

# 272. Redis Deduplication Risk

If Redis state expires before business replay:

```text
duplicate event
```

may be processed again.

Durable deduplication may be required.

---

# 273. Database Deduplication

A database unique constraint can provide stronger durability.

---

# 274. Consumer Transaction Isolation

Transaction isolation must match business correctness.

Examples:

```text
READ COMMITTED
REPEATABLE READ
SERIALIZABLE
```

Choose intentionally.

---

# 275. RabbitMQ Does Not Provide Business Transactions

RabbitMQ acknowledgement is not a replacement for:

```text
database transaction
```

---

# 276. Distributed Transaction

Avoid attempting to make:

```text
RabbitMQ + database + external API
```

one atomic transaction without a well-justified architecture.

Use patterns such as:

```text
outbox
inbox
idempotency
saga
```

where appropriate.

---

# 277. Saga

A long-running business workflow can be:

```text
Order
 |
Payment
 |
Inventory
 |
Shipment
```

Each step publishes events/commands.

Compensation handles failures.

---

# 278. Producer in Saga

Producer publishes:

```text
payment.requested
```

Consumer processes and publishes:

```text
payment.completed
```

or:

```text
payment.failed
```

---

# 279. Correlation in Saga

All related messages should carry:

```text
correlation_id
```

so the workflow can be reconstructed.

---

# 280. Consumer Timeout in Saga

If no response arrives:

```text
timeout
 |
compensation
```

Messaging reliability and workflow reliability are separate concerns.

---

# 281. Security Architecture

```text
Application
 |
TLS
 |
Network control
 |
RabbitMQ
 |
Vhost
 |
Permissions
 |
Exchange/Queue
```

---

# 282. Consumer Credentials

Use separate credentials per service where practical.

Benefits:

```text
audit
least privilege
credential rotation
blast-radius reduction
```

---

# 283. Producer Credential

Producer should only publish where required.

---

# 284. Management Access

Do not expose RabbitMQ management interfaces publicly without strong security
controls.

---

# 285. Audit

Track:

```text
topology changes
authentication
authorization
```

where supported.

---

# 286. Secret Rotation Test

Test:

```text
credential rotation
 |
connection recovery
 |
continued publication/consumption
```

before emergency rotation is required.

---

# 287. TLS Certificate Test

Test:

```text
certificate renewal
 |
client trust
 |
reconnection
```

---

# 288. Kubernetes Deployment Pattern

```text
Deployment
 |
+-- Consumer Pod
+-- Consumer Pod
+-- Consumer Pod
        |
        v
 RabbitMQ Service
        |
    RabbitMQ Cluster
```

---

# 289. Producer Deployment Pattern

```text
Application Deployment
 |
+-- Producer Pod
+-- Producer Pod
+-- Producer Pod
        |
        v
 RabbitMQ Service
```

---

# 290. Stateful RabbitMQ vs Stateless Clients

Producer/consumer pods can be stateless.

RabbitMQ itself is stateful.

This difference affects operational design.

---

# 291. Pod Restart

Producer/consumer state should be recoverable from:

```text
database
outbox
queue
```

rather than only local pod memory.

---

# 292. Local State Risk

Do not rely on:

```text
in-memory unconfirmed message
```

for critical events.

Pod restart destroys it.

---

# 293. Deployment Strategy

Use:

```text
RollingUpdate
```

with controlled surge/unavailable settings.

---

# 294. Consumer Rolling Update

Ensure:

```text
old capacity + new capacity
```

does not overload downstream systems.

---

# 295. Producer Rolling Update

Ensure multiple producer versions remain compatible with exchanges and consumers.

---

# 296. Consumer Pod Anti-Affinity

For critical workloads:

```text
Pod A -> Node 1
Pod B -> Node 2
Pod C -> Node 3
```

rather than all on one node.

---

# 297. Multi-AZ Consumers

Distribute consumers across AZs.

If one AZ fails:

```text
remaining consumers
```

must have enough capacity.

---

# 298. RabbitMQ Multi-AZ

Consumer resilience depends on RabbitMQ resilience too.

A perfect consumer deployment cannot compensate for a single-node broker failure
if the broker architecture is not HA.

---

# 299. EKS Failure Domain

Consider:

```text
AZ
node
pod
RabbitMQ node
queue replica
consumer
database
```

together.

---

# 300. Producer/Consumer Dependency Graph

```text
Producer
 |
RabbitMQ
 |
Consumer
 |
Database
 |
External API
```

The slowest critical dependency often controls end-to-end throughput.

---

# 301. Bottleneck Identification

Measure:

```text
CPU
memory
RabbitMQ
queue
database
API
network
```

before scaling.

---

# 302. Little's Law

For stable systems:

```text
L = λW
```

where:

```text
L = average number of items in system
λ = throughput
W = average time in system
```

Useful for estimating in-flight work.

---

# 303. Consumer Concurrency from Latency

Approximate:

```text
required concurrency
≈
arrival rate × processing latency
```

Example:

```text
1000 msg/s
x
0.2 seconds
=
~200 concurrent messages
```

This is a planning estimate, not a guarantee.

---

# 304. Queue Capacity

Queue capacity must account for:

```text
normal backlog
peak backlog
outage backlog
recovery backlog
disk limits
retention
```

---

# 305. Consumer Memory Capacity

Approximate:

```text
memory
≈
prefetch × message size × workers
+
runtime overhead
```

Always benchmark actual memory usage.

---

# 306. Producer Memory Capacity

Outstanding confirms can consume memory.

If:

```text
100,000 messages
```

are in flight, memory can become substantial.

Bound the confirmation window.

---

# 307. Backpressure Architecture

```text
External Traffic
      |
Producer
      |
Rate Limit
      |
RabbitMQ
      |
Queue
      |
Consumer
      |
Concurrency Limit
      |
Database/API
```

Each layer should have safe capacity.

---

# 308. Cascading Failure

Bad:

```text
DB slows
 |
consumer slows
 |
queue grows
 |
producer continues
 |
broker storage grows
 |
broker pressure
 |
system-wide failure
```

Use backpressure before the final resource is exhausted.

---

# 309. Load Shedding

For noncritical workloads:

```text
drop
sample
defer
```

may be acceptable.

Never apply blindly to critical business events.

---

# 310. Criticality Classes

Example:

```text
P0 critical
P1 important
P2 normal
P3 best effort
```

Routing and processing guarantees can differ.

---

# 311. Producer Priority

Classify events before publication:

```text
critical
normal
bulk
```

and route to appropriate queues.

---

# 312. Consumer Resource Isolation

Use separate deployments for:

```text
critical workers
bulk workers
analytics workers
```

when necessary.

---

# 313. Queue Resource Isolation

Separate queues prevent:

```text
bulk backlog
```

from directly blocking:

```text
critical workload
```

---

# 314. Consumer Fairness

For heterogeneous task durations:

```text
short job
long job
short job
```

a shared queue may create uneven latency.

Separate workloads where justified.

---

# 315. Producer Traffic Isolation

A noisy producer can overwhelm shared infrastructure.

Use:

```text
rate limits
quotas
separate exchanges/queues
```

where required.

---

# 316. Multi-Tenant Producer

Tenant-specific rate limits can prevent one tenant from exhausting shared
capacity.

---

# 317. Multi-Tenant Consumer

Track:

```text
tenant
queue age
processing rate
errors
```

to detect noisy-neighbor behavior.

---

# 318. Tenant Isolation

For high-security environments consider:

```text
separate vhost
separate broker
separate credentials
```

according to requirements.

---

# 319. Producer/Consumer Cost

Costs include:

```text
compute
network
RabbitMQ infrastructure
storage
cross-AZ traffic
observability
```

---

# 320. Consumer Cost Optimization

Optimize:

```text
worker count
CPU
memory
batching
scale-to-zero
```

without violating latency SLOs.

---

# 321. Producer Cost Optimization

Optimize:

```text
serialization
batching
compression
connection count
message size
```

---

# 322. Network Cost

High fanout plus large messages can create significant network traffic.

Calculate:

```text
message rate
×
message size
×
fanout
```

---

# 323. Cross-AZ Cost

If traffic crosses AZs:

```text
cost
+
latency
```

may increase.

Design placement intentionally.

---

# 324. Observability Cost

High-cardinality labels can become expensive.

Avoid labeling metrics with unbounded:

```text
message_id
tenant_id
order_id
```

unless the telemetry system can safely support it.

---

# 325. Producer Metric Cardinality

Good:

```text
exchange
routing category
result
```

Potentially bad:

```text
individual message ID
```

as a metric label.

---

# 326. Consumer Metric Cardinality

Good:

```text
queue
result
error category
```

Be careful with:

```text
event_id
order_id
```

labels.

---

# 327. Tracing Sampling

Use appropriate sampling.

Critical errors can be retained at higher rates.

---

# 328. Logging Sampling

Do not log every successful message at high volume if it creates excessive
cost.

Use:

```text
metrics
traces
sampled logs
```

together.

---

# 329. Production Dashboard

Producer panel:

```text
publish rate
confirm latency
confirm failures
returns
connections
```

Consumer panel:

```text
processing rate
processing latency
errors
redelivery
connections
```

Queue panel:

```text
ready
unacked
oldest age
depth
```

---

# 330. Operational Runbook

Producer failure:

```text
check connection
check confirms
check routing
check broker
check network
```

Consumer failure:

```text
check queue
check consumer
check dependency
check unacked
check redelivery
```

---

# 331. Production Checklist: Producer

```text
[ ] TLS where required
[ ] dedicated credentials
[ ] least privilege
[ ] durable topology
[ ] persistent messages where required
[ ] publisher confirms
[ ] mandatory publishing where required
[ ] retry strategy
[ ] duplicate-safe event IDs
[ ] connection recovery
[ ] channel recovery
[ ] bounded confirm window
[ ] backpressure
[ ] metrics
[ ] logs
[ ] tracing
[ ] graceful shutdown
[ ] outbox for critical DB events
```

---

# 332. Production Checklist: Consumer

```text
[ ] dedicated credentials
[ ] TLS
[ ] correct queue
[ ] prefetch tuned
[ ] concurrency tuned
[ ] ACK after successful processing
[ ] idempotency
[ ] retry classification
[ ] bounded retry
[ ] DLQ
[ ] timeout controls
[ ] downstream rate limits
[ ] graceful shutdown
[ ] readiness
[ ] liveness
[ ] resource requests
[ ] autoscaling limits
[ ] metrics
[ ] logs
[ ] tracing
[ ] replay runbook
```

---

# 333. Production Checklist: Kubernetes

```text
[ ] PDB
[ ] topology spread
[ ] anti-affinity where needed
[ ] resource requests
[ ] resource limits
[ ] startup probe
[ ] readiness probe
[ ] liveness probe
[ ] graceful SIGTERM handling
[ ] termination grace period
[ ] controlled rollout
[ ] secrets
[ ] NetworkPolicy
[ ] TLS
[ ] autoscaling max
```

---

# 334. Production Checklist: Failure

```text
[ ] broker restart tested
[ ] connection failure tested
[ ] consumer crash tested
[ ] producer crash tested
[ ] database outage tested
[ ] API outage tested
[ ] retry storm tested
[ ] DLQ replay tested
[ ] duplicate event tested
[ ] graceful shutdown tested
[ ] AZ failure tested
```

---

# 335. Senior Interview: Producer Reliability

### How do you make RabbitMQ publishing reliable?

Answer:

```text
For important events I use durable topology, persistent messages where
appropriate, publisher confirms, mandatory publishing when unroutable detection
is required, controlled retries, stable event IDs and idempotent consumers. For
database-originated events I prefer a transactional outbox so business state
and event creation are committed together.
```

---

# 336. Senior Interview: Ambiguous Publish

### What happens if the producer loses connection immediately after publishing?

Answer:

```text
The producer may not know whether RabbitMQ accepted the message. Retrying can
therefore create a duplicate. I use publisher confirms plus stable message
identity and idempotent processing, and for critical database events I use an
outbox.
```

---

# 337. Senior Interview: ACK

### When do you ACK?

Answer:

```text
After the business operation has completed successfully, normally after the
required transaction has committed. ACK is not a replacement for the database
commit, so duplicate processing must still be handled.
```

---

# 338. Senior Interview: Prefetch

### How do you choose prefetch?

Answer:

```text
I treat prefetch as an in-flight work and memory control. I consider processing
time, message size, worker concurrency, fairness and downstream capacity. I
benchmark rather than selecting an arbitrary large number.
```

---

# 339. Senior Interview: Consumer Scaling

### Queue depth is increasing. Do you add more consumers?

Answer:

```text
First I identify whether the bottleneck is consumer CPU, memory, database,
external API, broker capacity or producer rate. If consumer capacity is the
bottleneck and downstream systems can handle it, I scale consumers with a safe
maximum.
```

---

# 340. Senior Interview: Retry

### How do you implement retries?

Answer:

```text
I classify transient versus permanent failures, use bounded retries with
exponential backoff and jitter, avoid uncontrolled requeue loops, and send
persistent failures to a DLQ for investigation or controlled replay.
```

---

# 341. Senior Interview: Idempotency

### How do you make a consumer idempotent?

Answer:

```text
I assign a stable event or message ID and persist processing state atomically
with the business transaction, often using an inbox or unique database
constraint. For external APIs I use provider-supported idempotency keys where
available.
```

---

# 342. Senior Interview: Exactly Once

### Does RabbitMQ provide exactly-once processing?

Answer:

```text
I would not design around a simplistic exactly-once claim. A practical system
uses at-least-once delivery combined with idempotent business processing and
transactional patterns such as outbox and inbox.
```

---

# 343. Senior Interview: Outbox

### Why use the outbox pattern?

Answer:

```text
It solves the dual-write problem between a business database and RabbitMQ.
The application commits business state and the event record in one database
transaction, then a publisher reliably sends the outbox event to RabbitMQ.
```

---

# 344. Senior Interview: Inbox

### Why use an inbox?

Answer:

```text
It records consumed event IDs in a durable store so duplicate deliveries do not
create duplicate business effects.
```

---

# 345. Senior Interview: DB Failure

### Database is down. What should consumers do?

Answer:

```text
Do not continuously requeue at full speed. Apply bounded retry and backoff,
protect the database from a retry storm, and use the DLQ for messages that
cannot be processed after the retry policy.
```

---

# 346. Senior Interview: API Rate Limit

### External API returns 429.

Answer:

```text
Respect the provider's rate limit, apply controlled backoff, reduce consumer
concurrency if necessary, and avoid turning the 429 response into an immediate
retry storm.
```

---

# 347. Senior Interview: Zombie

### Consumer pod is healthy but no messages are processed.

Answer:

```text
I would check queue delivery, consumer registration, unacked count, processing
heartbeat, dependency latency, thread/worker state and application logs. A
TCP connection being open is not sufficient proof that the consumer is healthy.
```

---

# 348. Senior Interview: Graceful Shutdown

### How do you safely terminate a consumer?

Answer:

```text
Stop accepting new work, allow in-flight processing to finish, ACK successful
work, recover unfinished messages through normal unacknowledged behavior, close
the consumer/channel and exit within the Kubernetes termination window.
```

---

# 349. Senior Interview: Rolling Deployment

### How do you deploy consumers without losing messages?

Answer:

```text
Use graceful shutdown, compatible message schemas, sufficient termination grace
period, controlled rolling updates and idempotent processing. Messages that are
unacknowledged when a consumer exits can be redelivered.
```

---

# 350. Senior Interview: Kubernetes Scaling

### How would you autoscale RabbitMQ consumers?

Answer:

```text
I would use queue backlog or message age as workload signals, combined with CPU
and downstream constraints where appropriate. I would enforce maximum replicas
so autoscaling cannot overload the database or external dependencies.
```

---

# 351. Senior Interview: Memory

### Consumer memory keeps increasing.

Answer:

```text
I would inspect prefetch, message size, worker concurrency, batch size,
serialization, runtime heap and leaks. A useful first estimate is prefetch
times message size times worker count, plus runtime overhead.
```

---

# 352. Senior Interview: Ordering

### How do you preserve ordering?

Answer:

```text
First determine whether global ordering is actually required. If ordering is
per business key, route the same key to the same processing partition or queue
and avoid concurrent processing that can reorder completion.
```

---

# 353. Senior Interview: Fanout

### One producer message reaches ten queues. Is that a problem?

Answer:

```text
Not inherently, but it creates a 10x destination amplification factor. I would
capacity-plan broker CPU, network, storage and consumer workload before scaling
the topology.
```

---

# 354. Senior Interview: Multi-AZ

### How do you make consumers highly available?

Answer:

```text
Run multiple replicas across nodes and AZs, use PDBs and topology spread,
ensure RabbitMQ itself has appropriate HA, and verify remaining consumer and
broker capacity after an AZ failure.
```

---

# 355. Senior Interview: Broker Restart

### What happens to consumers when RabbitMQ restarts?

Answer:

```text
Connections can be lost, so consumers need reliable connection and channel
recovery. They must re-establish consumer registration and QoS state, and the
application should verify that processing resumes without creating uncontrolled
duplicate effects.
```

---

# 356. Senior Interview: Reconnect Storm

### 500 pods reconnect at once.

Answer:

```text
That can create a connection storm. I would use exponential backoff and jitter,
controlled deployment behavior and sufficient RabbitMQ capacity, and monitor
connection rates during recovery.
```

---

# 357. Senior Interview: Producer Scaling

### How do you scale producers?

Answer:

```text
Scale stateless producer instances while keeping connection/channel counts
controlled. Use publisher confirms, bounded in-flight windows and backpressure.
For database events, use an outbox so each instance can safely publish events
without losing business changes.
```

---

# 358. Senior Interview: Consumer Bottleneck

### Consumers are at 100% CPU.

Answer:

```text
Determine whether CPU is genuinely the bottleneck. If it is, scale CPU or
workers while checking message processing characteristics and downstream
effects. If CPU is caused by serialization, compression or inefficient business
logic, optimize before simply adding replicas.
```

---

# 359. Senior Interview: Queue Backlog

### Queue has ten million messages.

Answer:

```text
I would calculate arrival rate, processing rate, message age and drain time,
then identify the bottleneck. I would not blindly purge or massively scale
consumers because that could overload downstream systems.
```

---

# 360. Senior Interview: DLQ

### How should DLQ replay work?

Answer:

```text
Fix the root cause, inspect a sample, replay through the intended routing path
at a controlled rate, monitor downstream capacity, and rely on idempotency to
protect against duplicate business effects.
```

---

# 361. Senior Interview: Security

### How do you secure producers and consumers?

Answer:

```text
Use TLS, dedicated credentials, least-privilege permissions, private network
access, protected management interfaces and secure secret rotation. Producers
should only publish to authorized exchanges and consumers should only access
required queues.
```

---

# 362. Senior Interview: Production Architecture

### Design a reliable order-event consumer.

Answer:

```text
Order Service writes business state and an outbox record in one DB transaction.
An outbox publisher sends order.created to a durable topic exchange using
publisher confirms. The consumer uses controlled prefetch and concurrency,
checks event-idempotency state, performs the inventory transaction, commits,
then ACKs. Transient failures use bounded retry with backoff and permanent
failures go to a DLQ. Metrics track queue age, processing latency, redelivery
and business completion. Consumers run across AZs with graceful shutdown and
bounded autoscaling.
```

---

# 363. Senior Interview: Production Failure

### Database goes down while traffic is high.

Answer:

```text
I would stop uncontrolled retries, apply consumer backoff or pause processing,
protect RabbitMQ and the database from cascading failure, monitor queue age,
restore database capacity, then resume consumers gradually and calculate the
backlog drain time.
```

---

# 364. Senior Interview: Capacity

### How do you calculate consumer capacity?

Answer:

```text
I estimate processing throughput per worker, required concurrency from arrival
rate and processing latency, downstream limits, and failure-domain redundancy.
Then I validate the model with load tests.
```

---

# 365. Senior Interview: Memory

### Why can high prefetch be dangerous?

Answer:

```text
Because unacknowledged messages remain associated with consumers and their
processing state consumes memory. Large prefetch combined with large messages
and many workers can create substantial memory pressure and uneven work
distribution.
```

---

# 366. Senior Interview: Duplicate

### Why can duplicate messages occur?

Answer:

```text
Producer retries after ambiguous network failures, consumer crashes before ACK,
ACK failures and controlled replay can all produce duplicate processing.
Therefore idempotency is a fundamental application requirement.
```

---

# 367. Senior Interview: Business Transaction

### Where should ACK occur relative to DB commit?

Answer:

```text
For critical processing, commit the required business transaction first and
ACK afterward. If the ACK fails after commit, the message may be redelivered, so
the business transaction must be idempotent.
```

---

# 368. Senior Interview: External Side Effect

### Consumer calls an API successfully then crashes before ACK.

Answer:

```text
The message can be redelivered and the API may be called again. I would use an
idempotency key with the external provider if available, or maintain durable
deduplication state around the business operation.
```

---

# 369. Senior Interview: Monitoring

### What are your top consumer alerts?

Answer:

```text
oldest message age, sustained queue growth, unacked growth, processing failure
rate, redelivery rate, DLQ growth, consumer disappearance and downstream
dependency latency.
```

---

# 370. Senior Interview: Monitoring Producer

### Top producer alerts?

Answer:

```text
publisher confirm failures, unroutable/returned messages, publication latency,
connection churn and unexpected publication-rate changes.
```

---

# 371. Senior Interview: Architecture Trade-Off

### Prefetch 1 or 1000?

Answer:

```text
Neither is universally correct. Prefetch 1 favors fairness and limits
in-flight work but can reduce throughput. A larger value can improve throughput
but increases memory, work reservation and recovery complexity. I select it from
measured processing characteristics and downstream limits.
```

---

# 372. Senior Interview: Shared Queue

### When would you use multiple consumers on one queue?

Answer:

```text
When consumers are interchangeable workers for the same workload and horizontal
parallelism is desired. I would ensure the processing operation is idempotent
and that ordering requirements allow concurrency.
```

---

# 373. Senior Interview: Separate Queues

### When would you use separate queues?

Answer:

```text
When workloads need independent scaling, failure isolation, security,
retention, retry or processing characteristics.
```

---

# 374. Senior Interview: Outbox vs Local Retry

### Why not just retry in memory?

Answer:

```text
An in-memory retry disappears if the process crashes. A transactional outbox
provides durable recovery for events that originate from a database business
transaction.
```

---

# 375. Senior Interview: Exactly Once

### Customer must never be charged twice. What do you do?

Answer:

```text
I would not rely on broker delivery semantics alone. I would assign a stable
payment operation ID, use a transactional idempotency record in the payment
service, use the provider's idempotency mechanism where available, and design
retries so repeated delivery produces one business effect.
```

---

# 376. Senior Interview: Large Messages

### RabbitMQ messages are 10 MB each. What do you do?

Answer:

```text
I would question whether RabbitMQ should carry the payload. Large messages
increase memory, network and storage pressure. I would consider storing the
large object externally and publishing a secure reference, while preserving
event metadata and access control.
```

---

# 377. Senior Interview: Backpressure

### Queue is growing rapidly. What is your first action?

Answer:

```text
Measure producer rate, consumer throughput, queue age and downstream health.
Then identify the bottleneck and apply the least risky control: scale consumers
if safe, throttle producers, reduce unnecessary workload, or pause processing
if a dependency is failing.
```

---

# 378. Senior Interview: Cascading Failure

### How do you prevent RabbitMQ from causing a cascading failure?

Answer:

```text
Bound queue growth, apply producer backpressure, cap consumer concurrency,
protect downstream dependencies, use bounded retries, isolate critical workloads,
monitor resource pressure and define operational limits before broker storage or
consumer resources are exhausted.
```

---

# 379. Senior Interview: Multi-Region

### How would you run producers and consumers across regions?

Answer:

```text
I would prefer region-local processing where possible, define explicit
cross-region replication/federation requirements, design for duplicates and
failover, and avoid assuming that a single logical exchange automatically
provides global consistency.
```

---

# 380. Senior Interview: Failure Domain

### What is your consumer failure-domain strategy?

Answer:

```text
Spread replicas across nodes and AZs, maintain enough capacity after losing
one failure domain, use PDBs and topology spread, and make RabbitMQ queue
availability consistent with the same failure assumptions.
```

---

# 381. Senior Interview: Production Readiness

A producer/consumer workload is production-ready when:

```text
publication is reliable
routing is verified
processing is idempotent
retries are bounded
DLQ exists
backpressure exists
observability exists
security exists
failure recovery is tested
capacity is understood
deployment is graceful
```

---

# 382. Final Producer Mental Model

```text
                 APPLICATION
                      |
                 Event Creation
                      |
                  Event ID
                      |
                  Serialize
                      |
               Publisher Module
                      |
             Connection/Channel
                      |
              Publisher Confirms
                      |
                 RabbitMQ
                      |
                  Exchange
```

---

# 383. Final Consumer Mental Model

```text
RabbitMQ Queue
      |
   Delivery
      |
   Prefetch
      |
   Worker Pool
      |
   Validate
      |
   Deduplicate
      |
Business Transaction
      |
     Commit
      |
      ACK
```

---

# 384. Final Reliability Model

```text
Producer
   |
Publisher Confirm
   |
Routing Validation
   |
Queue
   |
Controlled Prefetch
   |
Idempotent Consumer
   |
Business Commit
   |
ACK
```

---

# 385. Final Failure Model

```text
Network Failure
      |
Reconnect
      |
Ambiguous Publication
      |
Stable Event ID
      |
Idempotent Consumer
      |
Safe Retry
```

---

# 386. Final Backpressure Model

```text
Producer Rate
      |
      v
   Exchange
      |
    Queue
      |
Consumer Capacity
      |
Downstream Capacity
```

If downstream capacity falls:

```text
downstream
    |
consumer slows
    |
queue grows
    |
producer throttles
```

---

# 387. Final Production Model

```text
                     +----------------+
                     |    Producer    |
                     +-------+--------+
                             |
                     Confirmed Publish
                             |
                             v
                     +---------------+
                     |    Exchange   |
                     +-------+-------+
                             |
                           Route
                             |
                             v
                     +---------------+
                     |     Queue     |
                     +-------+-------+
                             |
                        Prefetch/QoS
                             |
                             v
                     +---------------+
                     |    Consumer   |
                     +-------+-------+
                             |
                    Idempotent Handler
                             |
                             v
                     +---------------+
                     | DB / API / Svc|
                     +-------+-------+
                             |
                           Commit
                             |
                             v
                            ACK
```

---

# 388. Golden Rules

```text
1. Treat producers and consumers as distributed-system components.
2. Reuse connections appropriately.
3. Do not create excessive connections.
4. Manage channels deliberately.
5. Use publisher confirms for important publications.
6. Understand the ambiguous publish problem.
7. Use stable event/message IDs.
8. Use mandatory publishing when unroutable detection matters.
9. Do not confuse confirms with consumer processing.
10. ACK only after required business processing succeeds.
11. Commit the business transaction before ACK.
12. Expect redelivery.
13. Make consumers idempotent.
14. Use inbox/unique constraints for durable deduplication where needed.
15. Use outbox for reliable database-originated events.
16. Bound prefetch.
17. Tune concurrency from measured workload.
18. Protect downstream dependencies.
19. Bound retries.
20. Use backoff and jitter.
21. Prevent retry storms.
22. Use DLQ for persistent failures.
23. Make DLQ replay controlled and idempotent.
24. Monitor queue age, not only queue depth.
25. Monitor ready and unacked messages separately.
26. Monitor producer confirms.
27. Monitor unroutable publications.
28. Monitor redeliveries.
29. Propagate correlation and event IDs.
30. Use distributed tracing.
31. Handle graceful shutdown.
32. Test broker restart.
33. Test network failure.
34. Test duplicate processing.
35. Test dependency failure.
36. Spread consumers across failure domains.
37. Use PDBs and topology spread for critical workloads.
38. Bound autoscaling.
39. Do not scale consumers beyond downstream capacity.
40. Keep credentials least-privileged.
41. Use TLS where required.
42. Protect management interfaces.
43. Manage secrets securely.
44. Test credential/certificate rotation.
45. Avoid giant messages.
46. Calculate memory impact of prefetch.
47. Calculate backlog drain time.
48. Capacity-plan for AZ failure.
49. Keep producer/consumer versions compatible.
50. Treat business correctness as separate from transport semantics.
51. Do not promise exactly-once without proving the complete workflow.
52. Design for duplicates.
53. Design for recovery.
54. Design for observability.
55. Design for operational simplicity.
```

---

# 389. Production Readiness Questions

Before deploying a producer:

```text
What happens if RabbitMQ is unavailable?
What happens if publish confirmation is lost?
How are duplicate publications handled?
How are unroutable messages detected?
Where is the event persisted if publication fails?
What is the maximum in-flight publication count?
```

Before deploying a consumer:

```text
When is ACK sent?
What happens before ACK if the process crashes?
How are duplicates handled?
What failures are retried?
What failures go to DLQ?
What is the prefetch?
What is maximum concurrency?
What is downstream capacity?
How does graceful shutdown work?
```

---

# 390. Senior-Level System Design Framework

For any producer/consumer architecture:

```text
1. Define business event.
2. Define delivery semantics.
3. Define exchange and routing.
4. Define queue topology.
5. Define producer reliability.
6. Define consumer reliability.
7. Define ACK boundary.
8. Define idempotency.
9. Define retry/DLQ.
10. Define ordering.
11. Define backpressure.
12. Define scaling.
13. Define observability.
14. Define security.
15. Define Kubernetes deployment.
16. Define HA/failure domains.
17. Define DR.
18. Load test.
19. Chaos test.
20. Document runbook.
```

---

# 391. Section Progression

```text
04 RabbitMQ Architecture
        |
05 RabbitMQ Queues
        |
06 RabbitMQ Exchanges
        |
07 RabbitMQ Routing
        |
08 RabbitMQ Consumers and Producers
        |
09 RabbitMQ Acknowledgements
        |
10 RabbitMQ Retry and DLQ
        |
11 RabbitMQ High Availability
        |
12 RabbitMQ Kubernetes
        |
13 RabbitMQ Production Architecture
```

Next:

```text
09-RabbitMQ-Acknowledgements.md
```

The next chapter will go deeply into acknowledgement semantics, manual versus
automatic acknowledgement, delivery tags, ACK/NACK/reject, multiple ACKs,
requeue behavior, redelivery, acknowledgement timing, transaction boundaries,
duplicate processing, failure windows, consumer crashes, prefetch interaction,
retry/DLQ behavior, graceful shutdown, idempotency, observability, production
failure scenarios and senior-level interview/system-design questions.

# END OF 08-RabbitMQ-Consumers-and-Producers.md
