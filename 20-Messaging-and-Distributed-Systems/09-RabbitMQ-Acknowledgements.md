# RabbitMQ-Acknowledgements

## Purpose

RabbitMQ acknowledgement semantics determine how a consumer tells the broker
what happened to a delivered message.

Acknowledgements are one of the most important reliability boundaries in a
RabbitMQ production system.

The core production model is:

```text
RabbitMQ
   |
   | delivery
   v
Consumer
   |
   | validate/process
   v
Business Transaction
   |
   | success
   v
ACK
```

The critical principle is:

```text
ACK is a transport-level acknowledgement.
ACK is not a business transaction.
ACK is not a database commit.
ACK is not proof that every downstream side effect is durable.
```

A production design must answer:

```text
When is a message acknowledged?
Who acknowledges it?
What happens before acknowledgement?
What happens after acknowledgement?
What happens if the consumer crashes?
What happens if ACK is lost?
What happens if processing fails?
Should the message be requeued?
When should it be dead-lettered?
How does prefetch interact with acknowledgements?
How are duplicate deliveries handled?
How are multiple messages acknowledged?
How are graceful shutdowns implemented?
How are acknowledgement failures observed?
```

---

# 1. What Is an Acknowledgement?

An acknowledgement is a protocol-level signal from the consumer to RabbitMQ
indicating how a delivered message should be handled.

For successful processing:

```text
Consumer -> ACK -> RabbitMQ
```

---

# 2. Why ACK Exists

Without acknowledgement semantics, RabbitMQ would not know whether a consumer
successfully processed a delivered message.

ACK allows the broker to distinguish:

```text
delivered and successfully handled
```

from:

```text
delivered but not successfully handled
```

---

# 3. Delivery Lifecycle

A simplified lifecycle:

```text
Publish
   |
Exchange
   |
Queue
   |
Delivery
   |
Unacknowledged
   |
   +----> ACK ----> Completed
   |
   +----> NACK ----> Requeue/DLQ path
   |
   +----> Reject --> Requeue/DLQ path
```

---

# 4. Ready vs Unacknowledged

A queue has an important distinction:

```text
READY
```

messages are waiting for delivery.

```text
UNACKNOWLEDGED
```

messages have been delivered to consumers but have not yet been acknowledged.

---

# 5. Why Unacknowledged Matters

A high unacknowledged count can indicate:

```text
slow processing
large prefetch
blocked consumer
dependency latency
consumer crash recovery
application deadlock
```

---

# 6. Manual Acknowledgement

Manual acknowledgement gives the application explicit control.

Conceptually:

```text
receive
 |
process
 |
ACK
```

This is commonly preferred for important business workflows.

---

# 7. Automatic Acknowledgement

Automatic acknowledgement means the broker considers the message acknowledged
according to the configured consumer acknowledgement mode without waiting for
application-level processing completion.

This can simplify consumers but increases loss risk if processing fails after
delivery.

---

# 8. Production Default

For critical business processing:

```text
manual ACK
```

is generally easier to reason about because the application controls the
acknowledgement boundary.

---

# 9. ACK After Processing

Preferred conceptual sequence:

```text
receive
 |
validate
 |
process
 |
commit
 |
ACK
```

---

# 10. ACK Before Processing

Risky:

```text
receive
 |
ACK
 |
process
```

If the process crashes:

```text
message already acknowledged
+
business operation incomplete
=
possible message loss
```

---

# 11. ACK After Database Commit

For a database-backed consumer:

```text
Message
 |
DB transaction
 |
COMMIT
 |
ACK
```

This is a common reliability pattern.

---

# 12. ACK Does Not Commit Database

These are separate operations:

```text
Database COMMIT
```

and:

```text
RabbitMQ ACK
```

There is no automatic atomic transaction between them simply because they occur
in sequence.

---

# 13. Commit Then ACK

Typical:

```text
DB COMMIT
 |
ACK
```

If ACK fails or the connection disappears:

```text
message may be redelivered
```

Therefore:

```text
idempotency
```

is still required.

---

# 14. ACK Then Commit

Dangerous:

```text
ACK
 |
DB COMMIT
 |
DB failure
```

The message may not be redelivered.

---

# 15. The ACK Failure Window

Important sequence:

```text
DB COMMIT
 |
ACK sent
 |
network failure
```

The consumer may not know whether RabbitMQ received the ACK.

A redelivery can occur.

---

# 16. Duplicate Processing

A common sequence:

```text
Consumer processes
 |
DB COMMIT
 |
ACK
 |
connection fails
 |
RabbitMQ redelivers
 |
Consumer processes again
```

The business operation must tolerate the duplicate.

---

# 17. Idempotency Is Mandatory for Strong Reliability

Use:

```text
event_id
message_id
idempotency key
unique constraint
inbox record
```

where required.

---

# 18. Delivery Tag

RabbitMQ deliveries have delivery tags associated with a channel.

The consumer uses the delivery tag when acknowledging or negatively
acknowledging a message.

Conceptually:

```text
message -> delivery_tag = N
```

Then:

```text
ACK(N)
```

---

# 19. Delivery Tags Are Channel-Scoped

Do not treat a delivery tag as a globally unique message ID.

It is associated with the channel on which the delivery occurred.

---

# 20. Delivery Tag vs Message ID

Different concepts:

```text
delivery_tag
=
broker/channel delivery bookkeeping

message_id
=
application/message identity
```

---

# 21. Never Use Delivery Tag as Business Idempotency Key

Bad:

```text
idempotency_key = delivery_tag
```

because delivery tags are not durable business identities.

Use:

```text
event_id
```

or another stable application identifier.

---

# 22. ACK Command

A successful acknowledgement tells RabbitMQ that the consumer has accepted
responsibility for the delivered message.

Conceptually:

```text
ACK(delivery_tag)
```

---

# 23. Multiple ACK

RabbitMQ acknowledgement APIs can support acknowledging multiple outstanding
deliveries through the `multiple` flag.

Conceptually:

```text
ACK(tag=N, multiple=true)
```

can acknowledge deliveries up to the specified delivery tag on that channel.

---

# 24. Why Multiple ACK Exists

It can reduce protocol overhead when processing messages sequentially.

Instead of:

```text
ACK 1
ACK 2
ACK 3
ACK 4
```

a consumer can acknowledge a range where appropriate.

---

# 25. Multiple ACK Risk

Do not use multiple ACK casually when processing messages concurrently.

Example:

```text
Message 1 -> slow
Message 2 -> success
Message 3 -> success
```

If the consumer acknowledges through a higher delivery tag incorrectly, it may
acknowledge messages whose business processing has not actually completed.

---

# 26. Safe Multiple ACK

Multiple ACK is easiest to reason about when:

```text
processing order
=
acknowledgement order
```

---

# 27. Concurrent Processing

With parallel workers:

```text
tag 1 -> worker A
tag 2 -> worker B
tag 3 -> worker C
```

completion may be:

```text
2
3
1
```

Bulk acknowledgement requires careful tracking.

---

# 28. ACK Tracking

A concurrent consumer may need to track:

```text
delivery tag
processing state
success/failure
```

before issuing range acknowledgements.

---

# 29. Per-Message ACK

The simplest safe pattern:

```text
process one
 |
ACK one
```

This has more protocol operations but simpler correctness.

---

# 30. Batch ACK

A batch can be acknowledged after all messages in the batch have successfully
completed.

```text
1
2
3
4
 |
process
 |
all success
 |
ACK range
```

---

# 31. Batch ACK Failure

If:

```text
1 success
2 success
3 failure
4 success
```

the consumer must not accidentally acknowledge message 3.

Design the batch state explicitly.

---

# 32. ACK and Concurrency

There are two independent dimensions:

```text
delivery concurrency
```

and:

```text
acknowledgement ordering
```

Do not assume one automatically provides the other.

---

# 33. NACK

Negative acknowledgement tells RabbitMQ that the consumer did not successfully
process the message and allows a decision about requeue behavior.

Conceptually:

```text
NACK(tag, requeue=true/false)
```

---

# 34. NACK With Requeue

```text
NACK
 |
requeue=true
 |
message returns for delivery
```

This can be appropriate for some transient failures.

---

# 35. NACK Without Requeue

```text
NACK
 |
requeue=false
 |
dead-lettering may occur if configured
```

This is commonly used for permanent failures when a DLX/DLQ is configured.

---

# 36. Reject

RabbitMQ also supports rejecting an individual message.

Conceptually:

```text
REJECT(tag, requeue=true/false)
```

---

# 37. NACK vs Reject

A practical distinction:

```text
reject
=
single-message negative acknowledgement

nack
=
negative acknowledgement with optional multiple-message support
```

---

# 38. Requeue Semantics

Requeue does not mean:

```text
"try again later with delay"
```

necessarily.

Immediate requeue can create a hot loop.

---

# 39. Requeue Loop

Bad:

```text
receive
 |
fail
 |
NACK requeue
 |
receive
 |
fail
 |
NACK requeue
 |
...
```

This can consume CPU and network while preventing other useful work.

---

# 40. Retry vs Requeue

These are different concepts.

```text
requeue
=
return message to queue for another delivery

retry architecture
=
controlled future attempt with policy
```

---

# 41. Controlled Retry

A robust design:

```text
Main Queue
 |
failure
 |
Retry Queue
 |
delay
 |
Main Exchange
```

---

# 42. Retry Count

Track attempts explicitly when the retry architecture requires it.

Possible metadata:

```text
attempt=1
attempt=2
attempt=3
```

---

# 43. Retry Limit

Example:

```text
attempt 1
attempt 2
attempt 3
 |
DLQ
```

---

# 44. Permanent Failure

Examples:

```text
invalid schema
invalid required field
unsupported event version
permanent business rule violation
```

These normally should not be retried indefinitely.

---

# 45. Transient Failure

Examples:

```text
database timeout
temporary network failure
HTTP 503
temporary dependency overload
```

These may be retried with bounded backoff.

---

# 46. Retry Classification

Consumer logic should classify errors:

```text
retryable
non-retryable
unknown
```

---

# 47. Retry Storm

If thousands of consumers immediately requeue failed messages:

```text
consumer
 |
NACK
 |
queue
 |
consumer
 |
NACK
```

RabbitMQ can become part of a retry storm.

---

# 48. Backoff

Use:

```text
exponential backoff
```

for transient failures.

Example:

```text
1 second
5 seconds
30 seconds
5 minutes
```

---

# 49. Jitter

Add randomness to retry timing:

```text
base delay + random jitter
```

This prevents synchronized retry bursts.

---

# 50. DLQ

A dead-letter queue provides a destination for messages that should no longer
remain on the normal processing path.

Typical:

```text
Main Queue
 |
failure
 |
DLX
 |
DLQ
```

---

# 51. ACK and DLQ

A message is not ACKed as successfully processed when the consumer decides to
negative-acknowledge it without requeue.

The broker's dead-letter configuration then determines the next route.

---

# 52. DLQ Is Not an ACK

Do not describe:

```text
NACK + DLQ
```

as:

```text
successful processing
```

It means:

```text
normal processing failed and message was moved to a failure path
```

---

# 53. ACK and Business Success

Define:

```text
business success
```

before:

```text
ACK
```

---

# 54. ACK and Validation

If validation fails:

```text
validate
 |
invalid
 |
NACK/reject
 |
DLQ
```

Do not ACK invalid critical events as if processing succeeded unless your
business policy explicitly considers them handled.

---

# 55. ACK and Schema Failure

Schema failure is often permanent:

```text
invalid schema
 |
DLQ
```

Investigate before replay.

---

# 56. ACK and Authorization Failure

If the message requests an unauthorized action:

```text
do not blindly retry forever
```

Classify the failure.

---

# 57. ACK and Business Rule Failure

Example:

```text
order already cancelled
```

This may be:

```text
expected idempotent outcome
```

rather than a technical failure.

---

# 58. Idempotent Success

Suppose:

```text
event_id=123
```

was already processed.

The consumer may safely consider the duplicate handled and ACK it after
confirming durable idempotency state.

---

# 59. Redelivered Flag

RabbitMQ can indicate that a delivery is a redelivery.

Conceptually:

```text
redelivered = true
```

This is useful for diagnostics and retry decisions.

---

# 60. Redelivered Does Not Mean Duplicate Business Event

A message can be redelivered because:

```text
consumer crashed
ACK was lost
channel closed before acknowledgement
```

The application should use stable event identity for business deduplication.

---

# 61. Redelivery vs Retry Queue

Redelivery:

```text
broker-level repeated delivery
```

Retry queue:

```text
application/topology-level controlled retry path
```

---

# 62. Requeue and Ordering

Requeued messages can affect processing order.

Do not assume a failed message will always return in exactly the same position
relative to all other messages.

---

# 63. Requeue and Fairness

Repeatedly requeued messages can compete with new messages and create unfair
processing patterns.

---

# 64. Requeue and Poison Messages

A poison message should not remain in an immediate requeue loop.

Use:

```text
bounded retry
+
DLQ
```

---

# 65. ACK and Prefetch

Prefetch determines how many messages can be outstanding without
acknowledgement.

Example:

```text
prefetch = 10
```

limits the number of unacknowledged deliveries under the applicable QoS
semantics.

---

# 66. Prefetch 1

With:

```text
prefetch=1
```

a consumer has at most a small number of outstanding unacknowledged work items.

Useful for:

```text
slow processing
fairness
strictly controlled concurrency
```

---

# 67. High Prefetch

Example:

```text
prefetch=500
```

can improve throughput in some workloads.

But it can increase:

```text
memory
work reservation
recovery time
latency for other consumers
```

---

# 68. ACK and Memory

Unacknowledged messages remain part of the consumer's outstanding work.

Approximate payload memory:

```text
prefetch
×
average message size
×
consumer concurrency
```

plus runtime overhead.

---

# 69. Example Memory Calculation

Suppose:

```text
prefetch = 100
average message = 2 MB
workers = 10
```

Approximate payload exposure:

```text
100 × 2 MB × 10
=
2,000 MB
```

This is a rough planning estimate; actual memory can be higher or lower
depending on client implementation and object overhead.

---

# 70. ACK Latency

Measure:

```text
message delivery
 |
processing
 |
ACK
```

Long processing time creates longer unacknowledged periods.

---

# 71. Long Processing

For a message taking:

```text
10 minutes
```

the message may remain unacknowledged for approximately the processing
duration.

Design:

```text
prefetch
timeout
shutdown
visibility
```

accordingly.

---

# 72. Consumer Crash Before ACK

Sequence:

```text
delivery
 |
processing
 |
CRASH
 |
no ACK
```

RabbitMQ can make the message available for redelivery depending on the
connection/channel and queue state.

---

# 73. Consumer Crash After ACK

Sequence:

```text
delivery
 |
ACK
 |
CRASH
```

The broker considers the message acknowledged.

If business processing actually occurred before ACK, this is fine.

If ACK happened too early, the business operation can be lost.

---

# 74. Consumer Crash During DB Transaction

Example:

```text
delivery
 |
DB transaction
 |
CRASH
```

If the DB transaction rolls back:

```text
message can be redelivered
```

The consumer retries safely.

---

# 75. Consumer Crash After DB Commit

```text
delivery
 |
DB COMMIT
 |
CRASH
 |
no ACK
```

Message may be redelivered.

Idempotency prevents duplicate business effect.

---

# 76. Consumer Crash After External API

```text
delivery
 |
API succeeds
 |
CRASH
 |
no ACK
```

The API may be called again on redelivery.

Use:

```text
external idempotency key
```

where available.

---

# 77. ACK Lost

```text
Consumer
 |
ACK
 |
network failure
 |
RabbitMQ uncertain to consumer
```

The consumer may later see redelivery.

---

# 78. ACK Lost Does Not Mean Business Failure

The business operation may already be complete.

Therefore:

```text
redelivery
+
idempotency
```

is normal.

---

# 79. Exactly-Once Myth

Do not equate:

```text
ACK
```

with:

```text
exactly once
```

ACK only addresses broker acknowledgement state.

---

# 80. At-Least-Once Pattern

A common robust model:

```text
RabbitMQ delivery
+
manual ACK
+
idempotent business transaction
```

This provides practical at-least-once processing with duplicate-safe effects.

---

# 81. Consumer Transaction Boundary

Recommended:

```text
receive
 |
validate
 |
deduplicate
 |
business transaction
 |
commit
 |
ACK
```

---

# 82. Inbox Transaction

A stronger pattern:

```text
BEGIN
 |
check event_id
 |
if new:
   business operation
   record event_id
 |
COMMIT
 |
ACK
```

---

# 83. Duplicate Race

Two deliveries of the same event can arrive concurrently.

Bad:

```text
Worker A -> check -> not found
Worker B -> check -> not found
Worker A -> process
Worker B -> process
```

Use a transactional uniqueness mechanism.

---

# 84. Unique Constraint

Example:

```text
processed_events(event_id UNIQUE)
```

Then concurrent duplicates can be safely resolved.

---

# 85. ACK After Duplicate Detection

If the durable store says:

```text
event already processed
```

the consumer can safely ACK the duplicate after confirming the idempotency
record is trustworthy.

---

# 86. ACK and Transaction Isolation

Database transaction isolation affects how concurrent consumers interact.

Choose based on:

```text
business invariant
```

rather than RabbitMQ alone.

---

# 87. ACK and Outbox

Producer side:

```text
DB + Outbox
```

Consumer side:

```text
Inbox + DB + ACK
```

Together:

```text
Service A
 |
Outbox
 |
RabbitMQ
 |
Inbox
 |
Service B
```

---

# 88. ACK and Saga

A consumer can:

```text
process command
 |
commit state
 |
publish next event
 |
ACK
```

If the publish and DB commit are not atomic, use an outbox for the next event.

---

# 89. ACK and Dual Writes

Bad:

```text
DB commit
+
RabbitMQ publish
```

as independent operations without a recovery strategy.

Use:

```text
transactional outbox
```

when required.

---

# 90. Consumer ACK Strategy

Choose one:

```text
ACK per message
ACK in controlled batches
```

based on throughput and correctness.

---

# 91. ACK Per Message Trade-Off

Advantages:

```text
simple correctness
small failure window
easy troubleshooting
```

Disadvantages:

```text
more protocol operations
potentially lower throughput
```

---

# 92. Batch ACK Trade-Off

Advantages:

```text
lower protocol overhead
higher throughput
```

Disadvantages:

```text
more complex failure handling
larger redelivery set
harder concurrency management
```

---

# 93. Batch Size

If using batch processing:

```text
small batch
```

reduces failure scope.

```text
large batch
```

may improve throughput but increases:

```text
latency
memory
replay work
```

---

# 94. ACK and Latency SLO

If business SLO requires:

```text
process within 5 seconds
```

a batch that waits:

```text
30 seconds
```

before ACK can violate the SLO.

---

# 95. ACK and Throughput

If a consumer processes:

```text
10,000 messages/s
```

per-message acknowledgement overhead can matter.

Benchmark before optimizing.

---

# 96. ACK and CPU

Protocol overhead is only one component.

The actual bottleneck may be:

```text
database
serialization
business logic
network
```

---

# 97. ACK and Network Latency

High network latency between consumer and RabbitMQ can make acknowledgement
round trips more expensive.

Keep workloads geographically appropriate.

---

# 98. ACK and Cross-Region

Do not casually place consumers far away from the RabbitMQ cluster.

Cross-region latency affects:

```text
delivery
ACK
recovery
throughput
```

---

# 99. ACK and Cross-AZ

Cross-AZ consumer placement can affect latency and cost.

Use topology intentionally.

---

# 100. ACK and Kubernetes

Consumers running in Kubernetes must handle:

```text
SIGTERM
```

without accidentally acknowledging incomplete work.

---

# 101. Graceful Shutdown

Correct conceptual flow:

```text
SIGTERM
 |
stop new delivery/processing
 |
finish in-flight work
 |
commit successful work
 |
ACK successful work
 |
recover unfinished work
 |
close consumer
```

---

# 102. Shutdown Too Fast

Bad:

```text
SIGTERM
 |
process exits immediately
```

Unacknowledged messages can be redelivered, and in-flight external work may
continue or partially complete.

---

# 103. Shutdown Too Slow

If Kubernetes sends:

```text
SIGKILL
```

before processing finishes:

```text
unacknowledged messages
```

may be redelivered.

---

# 104. Termination Grace Period

Set enough time for the longest expected in-flight processing where practical.

Example:

```text
processing max = 60s
termination grace = 90s
```

is a planning example, not a universal value.

---

# 105. Readiness During Shutdown

When shutdown begins:

```text
Ready = false
```

so traffic and lifecycle systems can stop treating the pod as healthy.

---

# 106. Stop New Work

A consumer should stop starting new business operations before exiting.

---

# 107. Finish In-Flight Work

Allow active operations to reach a safe boundary.

---

# 108. ACK Successful Work

Only ACK work that actually completed according to the business contract.

---

# 109. Leave Failed Work Unacknowledged or Negatively Acknowledge

Use the intended retry/DLQ policy.

Do not ACK simply because shutdown is happening.

---

# 110. Connection Closure

Closing a consumer channel with outstanding unacknowledged messages can cause
those messages to become eligible for redelivery.

This is useful for safe recovery if the business operation was not committed.

---

# 111. ACK and Consumer Cancellation

If the consumer is cancelled:

```text
stop new processing
```

and safely handle existing in-flight work.

---

# 112. Consumer Cancellation Reasons

Possible causes include:

```text
queue deletion
exclusive consumer conflict
administrative action
connection failure
```

Investigate before automatically restarting.

---

# 113. Consumer Recovery

After reconnect:

```text
connection
 |
channel
 |
QoS
 |
consumer registration
 |
processing
```

must be restored.

---

# 114. ACK State Is Not Persistent Application State

Do not rely on:

```text
"in-memory acknowledged set"
```

for business correctness.

---

# 115. ACK and Persistence

The business record should be persisted in a durable system before ACK when
durability is required.

---

# 116. ACK and Cache

Do not ACK critical work merely because it is stored in an ephemeral cache.

---

# 117. ACK and Redis

Redis can be useful for idempotency depending on durability and business
requirements, but an expiring cache should not be the sole protection for a
critical irreversible operation.

---

# 118. ACK and Database

A durable relational database with an appropriate transaction/constraint can
provide strong business protection.

---

# 119. ACK and Object Storage

If processing creates an object:

```text
upload
 |
confirm object durability
 |
ACK
```

when object creation is the required business effect.

---

# 120. ACK and Multiple Side Effects

Suppose processing does:

```text
DB
+
API
+
object storage
```

before ACK.

A failure between side effects can leave partial state.

ACK semantics do not solve distributed atomicity.

---

# 121. Saga for Multiple Side Effects

Use:

```text
state machine
+
compensation
+
idempotency
```

for complex workflows.

---

# 122. ACK and Compensation

If an operation is partially successful:

```text
process
 |
partial failure
 |
compensation
 |
ACK or DLQ
```

depending on the final business state.

---

# 123. ACK and Business Completion

Define a clear completion point:

```text
Business operation completed
```

Then:

```text
ACK
```

---

# 124. ACK and Asynchronous Downstream Work

If the consumer starts another asynchronous process:

```text
consume
 |
enqueue downstream job
 |
ACK
```

the ACK means:

```text
downstream job accepted
```

not necessarily:

```text
final business result completed
```

Document this distinction.

---

# 125. ACK and Child Jobs

If a consumer creates child jobs and then ACKs, ensure the child-job publication
is durable and recoverable.

Outbox can help when child-job creation must be atomic with database state.

---

# 126. ACK and Event Chaining

```text
Event A
 |
Consumer
 |
Event B
 |
ACK A
```

If publishing B fails before ACK A:

```text
A can be redelivered
```

which may cause another attempt to publish B.

Make B publication idempotent or use an outbox.

---

# 127. ACK and Exactly-Once Event Chaining

Do not claim:

```text
ACK A + publish B = exactly once
```

without a transactional strategy.

---

# 128. ACK and Outbox Event Chaining

Better:

```text
Consumer
 |
DB transaction
 |
business state
 |
outbox B
 |
COMMIT
 |
ACK A
```

Outbox publisher later publishes B.

---

# 129. ACK and Inbox + Outbox

A robust service can use:

```text
RabbitMQ
 |
Inbox
 |
DB transaction
 |
Business state
 |
Outbox
 |
RabbitMQ
 |
ACK
```

This is a powerful microservice reliability pattern.

---

# 130. ACK and Retry Metadata

Retry metadata can include:

```text
attempt
first_seen
last_error
failure_type
```

Use structured metadata.

---

# 131. ACK and Original Message

When retrying, preserve enough original identity to support:

```text
deduplication
tracing
audit
```

---

# 132. ACK and Correlation ID

Maintain:

```text
correlation_id
```

through retries.

---

# 133. ACK and Trace ID

Preserve distributed trace context where appropriate.

---

# 134. ACK and Logging

Log:

```text
queue
delivery_tag
message_id
event_id
redelivered
processing_result
ack_result
```

Do not log secrets.

---

# 135. Delivery Tag Logging

Delivery tags are useful for operational debugging but should not be used as
business identifiers.

---

# 136. ACK Metrics

Track:

```text
ack_total
nack_total
reject_total
ack_latency
redelivery_total
```

where practical.

---

# 137. ACK Failure Metric

Track acknowledgement-related errors separately from business processing
errors.

---

# 138. Redelivery Rate

A rising redelivery rate may indicate:

```text
consumer crashes
processing failures
ACK failures
dependency outages
```

---

# 139. Unacked Alert

Alert when:

```text
unacked count
```

is unexpectedly high for the workload.

---

# 140. Unacked Age

Track the age of the oldest unacknowledged message when telemetry supports it.

---

# 141. ACK Latency Distribution

Use percentiles:

```text
p50
p95
p99
```

rather than only averages.

---

# 142. Processing vs ACK Latency

Separate:

```text
processing latency
```

from:

```text
ACK transport latency
```

to identify bottlenecks.

---

# 143. ACK Storm

A large backlog can produce:

```text
mass delivery
 |
mass processing
 |
mass ACK
```

Monitor broker and network capacity.

---

# 144. Reconnect ACK Behavior

After a connection failure, outstanding ACK state on the old channel cannot
simply be assumed to have been persisted from the consumer's perspective.

Messages may be redelivered.

---

# 145. Channel Closure

Channel closure can affect:

```text
outstanding unacknowledged deliveries
```

Treat them as potentially redeliverable.

---

# 146. Consumer Channel Isolation

A dedicated consumer channel can simplify:

```text
delivery state
ACK tracking
QoS
recovery
```

---

# 147. Sharing Channels

Sharing channels across unrelated concurrent consumer operations can make
acknowledgement reasoning harder and may violate client-library concurrency
rules.

Follow the library's thread-safety contract.

---

# 148. ACK and Client Library

Different languages expose:

```text
ACK
NACK
reject
QoS
consumer cancellation
```

through different APIs.

Always understand the library semantics rather than assuming syntax implies
identical runtime behavior.

---

# 149. ACK Testing

Every production consumer should test:

```text
success
failure
redelivery
duplicate
ACK loss
consumer crash
connection loss
```

---

# 150. Test: ACK After Commit

Test:

```text
DB commit
 |
ACK
```

and verify correct recovery if ACK fails.

---

# 151. Test: Crash Before ACK

Inject a crash:

```text
after business commit
before ACK
```

Verify:

```text
redelivery
+
idempotent outcome
```

---

# 152. Test: Crash Before Commit

Inject:

```text
before DB commit
```

Verify:

```text
transaction rollback
+
message redelivery
```

---

# 153. Test: Crash After ACK

Inject:

```text
ACK
 |
process termination
```

Verify no required business work remains unfinished.

---

# 154. Test: NACK Requeue

Verify:

```text
NACK requeue=true
```

results in expected redelivery behavior.

---

# 155. Test: NACK No Requeue

Verify:

```text
NACK requeue=false
```

results in expected dead-letter behavior when configured.

---

# 156. Test: Reject

Test:

```text
reject requeue=false
```

for permanent failures.

---

# 157. Test: Retry Limit

Inject repeated failures:

```text
1
2
3
```

Verify:

```text
DLQ
```

after the intended limit.

---

# 158. Test: Poison Message

Verify a poison message does not cause an infinite requeue loop.

---

# 159. Test: Concurrent ACK

If using parallel processing and multiple ACK:

```text
complete messages out of order
```

Verify acknowledgement logic does not acknowledge incomplete messages.

---

# 160. Test: Prefetch

Change:

```text
prefetch=1
prefetch=10
prefetch=100
```

and measure:

```text
throughput
latency
memory
fairness
```

---

# 161. Test: Graceful Shutdown

Send:

```text
SIGTERM
```

during active processing.

Verify:

```text
successful work ACKed
unfinished work redelivered
no silent loss
```

---

# 162. Test: Node Failure

Kill a consumer node/pod.

Verify:

```text
unacknowledged work
```

returns safely.

---

# 163. Test: Broker Failure

Restart RabbitMQ during active processing.

Verify:

```text
consumer reconnect
message redelivery
idempotent business state
```

---

# 164. Test: Network Partition

Simulate network interruption between consumer and RabbitMQ.

Observe:

```text
ACK ambiguity
redelivery
reconnect
```

---

# 165. Test: Database Failure

Cause:

```text
DB timeout
```

and verify:

```text
no premature ACK
bounded retry
```

---

# 166. Test: External API Failure

Cause:

```text
API 503
```

and verify retry behavior.

---

# 167. Test: External API Duplicate

Force:

```text
API success
consumer crash
```

Verify provider idempotency protects the operation.

---

# 168. Test: DLQ Replay

Replay a failed event and verify:

```text
idempotent processing
correct ACK
```

---

# 169. Test: Duplicate Delivery

Deliver the same event twice concurrently.

Verify:

```text
one business effect
```

---

# 170. Test: Schema Error

Publish malformed data.

Verify:

```text
bounded handling
DLQ
```

rather than infinite retry.

---

# 171. Test: Authorization Error

Publish unauthorized operation.

Verify:

```text
correct rejection
```

and no infinite retry.

---

# 172. Test: Shutdown During Batch

If batch ACK is used:

```text
shutdown during batch
```

Verify only completed work is acknowledged.

---

# 173. Production ACK Architecture

```text
                  RabbitMQ
                      |
                   Delivery
                      |
                      v
                +-----------+
                | Consumer  |
                +-----+-----+
                      |
                  Validate
                      |
                 Deduplicate
                      |
                      v
                +-----------+
                | DB Txn    |
                +-----+-----+
                      |
                   Commit
                      |
                      v
                     ACK
```

---

# 174. Failure Path

```text
Consumer
   |
Processing
   |
Failure
   |
Classify
   |
+--+----------------+
|                   |
Transient         Permanent
|                   |
Retry             DLQ
|                   |
Backoff            Inspect
```

---

# 175. ACK + Retry Architecture

```text
Main Queue
    |
Consumer
    |
Failure
    |
Retry Route
    |
Delay
    |
Main Queue
```

The retry path should be bounded.

---

# 176. ACK + Idempotency Architecture

```text
Message
 |
event_id
 |
Idempotency Store
 |
+---- already processed ----> ACK
 |
new
 |
Business Transaction
 |
record event_id
 |
COMMIT
 |
ACK
```

---

# 177. ACK + Outbox Architecture

```text
Message A
   |
Consumer
   |
DB Transaction
   |
+--+----------------+
|                   |
Business State    Outbox B
|                   |
+---------+---------+
          |
        COMMIT
          |
         ACK
          |
       Publisher
          |
      RabbitMQ B
```

---

# 178. ACK + Kubernetes

```text
Pod
 |
RabbitMQ Consumer
 |
SIGTERM
 |
Readiness=false
 |
Stop new work
 |
Finish in-flight
 |
Commit
 |
ACK
 |
Close
 |
Exit
```

---

# 179. ACK + Autoscaling

Autoscaling should consider:

```text
queue depth
queue age
processing latency
unacked count
downstream capacity
```

Do not scale without regard to ACK behavior.

---

# 180. ACK + Prefetch + Scaling

Example:

```text
20 pods
x
10 workers
x
prefetch 50
```

can expose a large number of in-flight messages.

Calculate this before deployment.

---

# 181. In-Flight Calculation

Conceptually:

```text
in-flight
≈
consumer replicas
×
workers
×
prefetch
```

Actual broker/client semantics may differ depending on QoS configuration, so
validate against the selected RabbitMQ client and topology.

---

# 182. Why In-Flight Matters

High in-flight work can increase:

```text
memory
redelivery volume
shutdown duration
duplicate exposure
```

---

# 183. ACK and Queue Drain

A consumer may receive many messages before acknowledging them.

During a crash:

```text
large in-flight set
```

can return to the queue.

This can create a sudden redelivery burst.

---

# 184. Redelivery Burst

Mitigate through:

```text
reasonable prefetch
controlled concurrency
graceful shutdown
idempotency
```

---

# 185. ACK and Work Reservation

High prefetch reserves work with a consumer before business processing completes.

This can reduce fairness.

---

# 186. Fairness

If:

```text
Consumer A
```

has high prefetch and slow work:

```text
Consumer B
```

may receive less work even if it is faster.

Tune prefetch.

---

# 187. ACK and Slow Consumer

Slow processing:

```text
unacked grows
```

Monitor:

```text
processing latency
```

before changing ACK behavior.

---

# 188. ACK and Fast Consumer

For fast workloads, higher prefetch and efficient acknowledgement can increase
throughput.

Benchmark.

---

# 189. ACK and Long-Lived Consumer

Long-lived consumers must handle:

```text
connection recovery
channel recovery
consumer cancellation
```

and restore acknowledgement-related state.

---

# 190. ACK and Consumer Restart

After restart:

```text
old unacked messages
```

may be redelivered.

The new process must be duplicate-safe.

---

# 191. ACK and Rolling Deployment

During rolling updates:

```text
old consumer
+
new consumer
```

can process concurrently.

Ensure schema compatibility and idempotency.

---

# 192. ACK and Blue-Green

If blue and green consumers use the same queue, both can process messages.

If only one environment should process traffic, use controlled routing/queue
handoff rather than assuming deployment labels change RabbitMQ behavior.

---

# 193. ACK and Canary

A canary consumer should receive only intended traffic.

Use:

```text
separate queue
```

or controlled routing when appropriate.

---

# 194. ACK and Multi-Cluster

If consumers span RabbitMQ clusters:

```text
acknowledgement semantics
+
failure behavior
```

must be designed per cluster.

---

# 195. ACK and Multi-Region

Cross-region consumers increase:

```text
latency
failure ambiguity
reconnect time
```

Keep message processing close to the appropriate broker where possible.

---

# 196. ACK and Security

ACK operations require the consumer to have appropriate access to the queue
and channel.

Least privilege applies to consumers.

---

# 197. ACK and Audit

For critical systems, record enough information to reconstruct:

```text
received
processed
ACKed
failed
retried
DLQed
```

without storing unnecessary sensitive payloads.

---

# 198. ACK and Compliance

Retention/audit requirements may require:

```text
event ID
processing timestamp
consumer version
result
```

rather than raw message contents.

---

# 199. ACK and Message Ordering

Acknowledgement order does not automatically establish business completion
order.

Example:

```text
Message 1 -> slow
Message 2 -> fast
```

Message 2 may be ACKed first if processing is concurrent.

---

# 200. Ordered Processing

If ordering is required:

```text
single worker
```

or:

```text
per-key partition
```

may be needed.

---

# 201. ACK and Per-Key Ordering

For:

```text
order_id
```

route related events to a stable processing partition and ensure only the
required level of concurrency.

---

# 202. ACK and Retry Ordering

Retry can cause:

```text
event 1 -> retry
event 2 -> success
```

Therefore ACK semantics alone cannot guarantee business ordering.

---

# 203. ACK and Poison Message

A poison message should eventually reach:

```text
DLQ
```

rather than block an entire workload indefinitely.

---

# 204. ACK and Priority

If high-priority messages share a queue with slow low-priority work, ACK and
prefetch behavior can affect perceived priority.

Separate queues can provide clearer isolation.

---

# 205. ACK and Workload Isolation

Use separate queues for:

```text
critical
normal
batch
```

when their processing guarantees differ.

---

# 206. ACK and Multi-Tenancy

A shared queue requires:

```text
tenant-aware processing
```

and durable authorization/idempotency where necessary.

---

# 207. ACK and Tenant Failure

One tenant producing poison messages should not prevent other tenants from
making progress.

Use:

```text
bounded retries
isolation
rate limits
```

where appropriate.

---

# 208. ACK and Noisy Neighbor

High prefetch or high concurrency for one workload can consume excessive
processing capacity.

Use separate deployments/queues if required.

---

# 209. ACK and Capacity

Capacity planning must include:

```text
processing time
prefetch
concurrency
ACK rate
retry rate
redelivery rate
```

---

# 210. ACK Rate

A useful metric:

```text
ACKs per second
```

Compare against:

```text
deliveries per second
```

---

# 211. ACK/Delivery Ratio

In a stable system:

```text
ACK rate
```

should roughly track successful delivery/processing rate.

Significant divergence can indicate:

```text
failures
backlog
slow processing
```

---

# 212. NACK Rate

Monitor:

```text
NACK / delivery
```

A sudden increase indicates a processing problem.

---

# 213. Reject Rate

Monitor:

```text
reject / delivery
```

especially for validation and schema failures.

---

# 214. Redelivery Rate

Monitor:

```text
redeliveries / deliveries
```

A sustained increase is a reliability signal.

---

# 215. DLQ Rate

Monitor:

```text
DLQ messages / total messages
```

according to business expectations.

---

# 216. ACK Latency SLO

Example:

```text
95% of successfully processed messages are acknowledged within 5 seconds.
```

The exact SLO must match workload requirements.

---

# 217. Processing SLO

More meaningful:

```text
95% of critical events produce the required business effect within 30 seconds.
```

ACK is one part of that workflow.

---

# 218. ACK Troubleshooting

If queue shows:

```text
unacked rising
```

check:

```text
consumer processing
prefetch
downstream dependency
worker concurrency
memory
CPU
```

---

# 219. ACK Troubleshooting: Zero ACKs

If deliveries occur but ACKs do not:

```text
manual ACK code
exceptions
channel state
consumer callback
```

---

# 220. ACK Troubleshooting: Redelivery Spike

Check:

```text
consumer crashes
ACK timing
network
channel closures
NACK/requeue
```

---

# 221. ACK Troubleshooting: DLQ Spike

Check:

```text
validation
schema
business errors
dependency failure
retry limit
```

---

# 222. ACK Troubleshooting: Unacked Stuck

Possible:

```text
worker deadlock
external API hang
DB lock
high prefetch
CPU saturation
```

---

# 223. ACK Troubleshooting: Consumer Connected

A connection alone does not prove:

```text
messages are being processed
```

Check:

```text
deliver rate
processing rate
ACK rate
```

---

# 224. ACK Troubleshooting: Message Loss Suspicion

If messages appear missing:

```text
check auto-ack
check premature ACK
check queue deletion
check DLQ
check TTL/expiry policies
check producer routing
```

---

# 225. Auto-ACK Incident

If a critical consumer uses automatic acknowledgement:

```text
delivery
 |
auto-ack
 |
application crash
```

messages can be lost from the queue.

Use manual acknowledgement for critical workflows where appropriate.

---

# 226. Premature ACK Incident

Application:

```text
ACK
 |
DB operation
```

and DB fails.

Result:

```text
message lost
```

Move ACK after the required business completion boundary.

---

# 227. Incorrect Multiple ACK Incident

Concurrent workers:

```text
tag 10 completes
tag 9 still processing
```

A multiple ACK through tag 10 can accidentally acknowledge tag 9 if the
application uses the mechanism incorrectly.

---

# 228. NACK Requeue Incident

Consumer:

```text
NACK requeue=true
```

for every exception.

Result:

```text
hot retry loop
```

Classify failures.

---

# 229. DLQ Incident

DLQ is growing.

Do not immediately replay everything.

First:

```text
identify root cause
sample messages
classify failures
fix consumer
replay controlled subset
```

---

# 230. Redelivery Incident

If redelivery is high:

```text
consumer crash
```

or:

```text
ACK ambiguity
```

may be involved.

Correlate with deployment and network events.

---

# 231. Deployment Correlation

A sudden redelivery spike immediately after deployment suggests checking:

```text
new consumer version
ACK code
exception handling
startup/shutdown
```

---

# 232. Broker Correlation

A redelivery spike after broker/network failure suggests:

```text
connection recovery
channel recovery
```

and ACK ambiguity.

---

# 233. Database Correlation

A redelivery spike with DB latency indicates:

```text
processing exceeded normal duration
```

and consumers may have crashed or timed out.

---

# 234. External API Correlation

A redelivery spike with API failures may indicate:

```text
retry policy
timeouts
consumer restarts
```

---

# 235. ACK Runbook

```text
1. Check queue ready count.
2. Check unacknowledged count.
3. Check delivery rate.
4. Check ACK rate.
5. Check NACK/reject rate.
6. Check redelivery rate.
7. Check consumer count.
8. Check processing latency.
9. Check downstream dependencies.
10. Check recent deployments.
11. Check connection/channel errors.
12. Check DLQ.
```

---

# 236. Production ACK Checklist

```text
[ ] manual ACK for critical workloads
[ ] ACK after business completion
[ ] DB commit before ACK where applicable
[ ] idempotency implemented
[ ] delivery tag understood
[ ] message ID/event ID separate from delivery tag
[ ] multiple ACK reviewed carefully
[ ] concurrency model documented
[ ] prefetch tuned
[ ] retry policy defined
[ ] requeue policy defined
[ ] DLQ defined
[ ] redelivery monitored
[ ] ACK latency monitored
[ ] graceful shutdown
[ ] connection recovery
[ ] duplicate processing tested
[ ] crash-before-ACK tested
[ ] crash-after-commit tested
[ ] crash-after-ACK tested
[ ] dependency failures tested
```

---

# 237. Senior Interview: What Is ACK?

Answer:

```text
ACK is the consumer's protocol-level acknowledgement that a delivered message
has been successfully handled. It does not mean the entire distributed
workflow is globally committed.
```

---

# 238. Senior Interview: When Do You ACK?

Answer:

```text
For critical business processing I ACK after the required business operation
has completed successfully, typically after the database transaction commits.
This minimizes loss, while idempotency handles redelivery.
```

---

# 239. Senior Interview: Why Not ACK First?

Answer:

```text
If I ACK before processing and the consumer crashes afterward, RabbitMQ can
consider the message handled even though the business operation never completed.
That creates a message-loss scenario.
```

---

# 240. Senior Interview: ACK vs Commit

Answer:

```text
A database commit and RabbitMQ ACK are separate operations. If the DB commits
and ACK is lost, the message may be redelivered, so the business operation must
be idempotent.
```

---

# 241. Senior Interview: What Happens if Consumer Crashes Before ACK?

Answer:

```text
The unacknowledged message can become available for redelivery after the
consumer channel/connection closes. The consumer must therefore be idempotent.
```

---

# 242. Senior Interview: What Happens if Consumer Crashes After DB Commit but Before ACK?

Answer:

```text
The message may be redelivered. I use a stable event ID and durable
deduplication so the second delivery does not create another business effect.
```

---

# 243. Senior Interview: What Happens if ACK Is Lost?

Answer:

```text
The business operation may already be complete, but the broker may redeliver
the message. I design for at-least-once processing and idempotency rather than
assuming the ACK outcome is always known to the consumer.
```

---

# 244. Senior Interview: NACK

Answer:

```text
NACK tells RabbitMQ that processing was unsuccessful and can optionally request
requeue. I use requeue carefully because immediate requeue can create retry
loops.
```

---

# 245. Senior Interview: Reject

Answer:

```text
Reject is used for negative acknowledgement of an individual delivery. I can
choose whether the message should be requeued, with DLX/DLQ handling commonly
used for permanent failures.
```

---

# 246. Senior Interview: NACK vs Reject

Answer:

```text
Both communicate negative processing outcome. Reject is generally for one
message, while NACK also supports the multiple-message acknowledgement option.
```

---

# 247. Senior Interview: Requeue

Answer:

```text
Requeue puts the message back into the queue for another delivery. It is not a
complete retry strategy because immediate requeue can create a hot loop.
```

---

# 248. Senior Interview: Retry

Answer:

```text
I classify failures, apply bounded retries with backoff and jitter for
transient failures, and route permanent failures to a DLQ.
```

---

# 249. Senior Interview: DLQ

Answer:

```text
A DLQ is a failure-handling destination for messages that should leave the
normal processing path. It should have an owner, monitoring, retention and a
controlled replay process.
```

---

# 250. Senior Interview: Redelivery

Answer:

```text
Redelivery can happen after consumer failure, channel closure or acknowledgement
ambiguity. The redelivered flag is useful operationally, but I use stable event
identity rather than the flag for business deduplication.
```

---

# 251. Senior Interview: Delivery Tag

Answer:

```text
A delivery tag identifies a delivery within a channel for acknowledgement
purposes. It is not a globally unique business message identifier.
```

---

# 252. Senior Interview: Multiple ACK

Answer:

```text
Multiple acknowledgement can acknowledge a range of deliveries through a
delivery tag. I use it carefully with concurrent processing because an
out-of-order completion can cause an incomplete message to be acknowledged
accidentally.
```

---

# 253. Senior Interview: Prefetch

Answer:

```text
Prefetch controls outstanding unacknowledged work. I tune it based on message
size, processing latency, concurrency, fairness, memory and downstream
capacity.
```

---

# 254. Senior Interview: Prefetch 1000

Answer:

```text
I would not automatically choose 1000. I would estimate memory and in-flight
work, measure throughput and latency, and verify that a consumer crash would
not create an excessive redelivery burst.
```

---

# 255. Senior Interview: Duplicate

Answer:

```text
Duplicates are expected in reliable at-least-once systems. I use stable event
IDs and durable idempotency state, often enforced transactionally with the
business operation.
```

---

# 256. Senior Interview: Database

Answer:

```text
I process the message inside the required database transaction, commit the
business change, then ACK. If the ACK fails, the message may be redelivered, so
the transaction must be duplicate-safe.
```

---

# 257. Senior Interview: External API

Answer:

```text
If an external API succeeds and the consumer crashes before ACK, the API can be
called again. I use the provider's idempotency key where available or maintain
durable application-level deduplication.
```

---

# 258. Senior Interview: Outbox

Answer:

```text
For events generated by a database transaction, I use a transactional outbox.
The consumer can similarly use an inbox when duplicate delivery must be
deduplicated transactionally.
```

---

# 259. Senior Interview: Graceful Shutdown

Answer:

```text
I mark the consumer unready, stop starting new work, finish or safely abandon
in-flight work, ACK only completed operations, close the consumer/channel and
exit within the Kubernetes termination window.
```

---

# 260. Senior Interview: Redelivery Spike

Answer:

```text
I check consumer crashes, channel/connection closures, ACK timing, recent
deployments, network failures, database latency and requeue behavior before
changing topology.
```

---

# 261. Senior Interview: Unacked Spike

Answer:

```text
I inspect processing latency, prefetch, worker state, downstream dependencies,
CPU and memory. A high unacked count is not automatically a broker problem.
```

---

# 262. Senior Interview: Retry Storm

Answer:

```text
I stop uncontrolled requeue, classify failures, use bounded retry with
backoff/jitter and move poison messages to a DLQ.
```

---

# 263. Senior Interview: Exactly Once

Answer:

```text
I would avoid promising exactly-once semantics from ACK alone. I use
at-least-once delivery plus idempotent business processing and transactional
patterns where necessary.
```

---

# 264. Senior Interview: Message Loss

Question:

```text
How can a RabbitMQ consumer lose messages?
```

Answer:

```text
Premature acknowledgement, automatic acknowledgement followed by consumer
failure, incorrect topology/TTL policies, or an application that considers
processing complete before the required business operation is durable can cause
loss. I verify the entire lifecycle rather than blaming ACK alone.
```

---

# 265. Senior Interview: Ordering

Answer:

```text
ACK order does not guarantee business completion order. With concurrent
processing, messages can complete and be acknowledged out of order. If
ordering matters, I use sequential processing or stable per-key partitioning.
```

---

# 266. Senior Interview: Batch ACK

Answer:

```text
Batch ACK can improve throughput, but I only use it when I can prove that every
delivery covered by the acknowledgement has completed successfully. Concurrent
out-of-order work makes this more complex.
```

---

# 267. Senior Interview: Kubernetes

Answer:

```text
I design consumers for SIGTERM, readiness changes, termination grace periods,
connection recovery and idempotent redelivery. A pod restart must not cause
business duplication or loss.
```

---

# 268. Senior Interview: Capacity

Answer:

```text
I calculate replicas, worker concurrency and prefetch together because their
product represents potential in-flight work. Then I validate against message
size, memory, processing latency and downstream capacity.
```

---

# 269. Senior Interview: Production Design

Question:

```text
Design a payment consumer that must never charge a customer twice.
```

Answer:

```text
The message contains a stable payment operation ID. The consumer uses manual
ACK, controlled prefetch and bounded retry. Before charging, it checks durable
idempotency state and records the payment operation transactionally with the
business state where possible. The payment provider's idempotency key is used
for the external charge. Only after the required business state is durable does
the consumer ACK. If the process crashes after the charge but before ACK,
redelivery finds the existing operation and does not create another charge.
Permanent failures go to a DLQ and replay is controlled.
```

---

# 270. Senior Interview: Production Design

Question:

```text
Database is down for 20 minutes. What happens?
```

Answer:

```text
Consumers should not immediately ACK messages because the business operation
cannot complete. They should apply bounded retry/backoff or controlled pause,
prevent a retry storm, allow RabbitMQ to buffer within planned limits, monitor
queue age and resume gradually after database recovery.
```

---

# 271. Senior Interview: Production Design

Question:

```text
You have 100 consumers and redeliveries suddenly increase.
```

Answer:

```text
I correlate the event with deployments, broker/network failures, consumer
crashes, ACK errors, database latency and requeue behavior. I inspect unacked
messages and processing latency, identify whether the issue is application,
dependency or infrastructure, then recover without blindly purging or replaying
messages.
```

---

# 272. Senior Interview: Production Design

Question:

```text
How would you tune prefetch?
```

Answer:

```text
I start from message size and processing latency, calculate safe in-flight
memory, then benchmark different values. I also consider fairness, ordering,
consumer crash redelivery and downstream limits. I would rather use a measured
value than a universal number.
```

---

# 273. Senior Interview: Production Design

Question:

```text
How do you prevent a poison message from blocking the queue?
```

Answer:

```text
Classify the error, avoid infinite immediate requeue, use bounded retries with
backoff and route the message to a DLQ after the retry limit. The DLQ is then
inspected and replayed only after the root cause is fixed.
```

---

# 274. Senior Interview: Production Design

Question:

```text
How do you guarantee a message is not lost during Kubernetes deployment?
```

Answer:

```text
Use manual ACK after successful processing, graceful SIGTERM handling, a
sufficient termination grace period, readiness shutdown, and idempotent
redelivery. Messages that remain unacknowledged can be redelivered after the
consumer exits.
```

---

# 275. Senior Interview: Production Design

Question:

```text
Why did increasing prefetch from 10 to 1000 make the system worse?
```

Answer:

```text
It may have increased memory pressure, reserved too much work per consumer,
reduced fairness, increased recovery/redelivery scope and hidden downstream
bottlenecks. I would inspect unacked count, memory, latency and processing
throughput.
```

---

# 276. Senior Interview: Production Design

Question:

```text
Why are duplicate payments happening even though consumers ACK?
```

Answer:

```text
ACK does not provide business-level exactly-once semantics. A message can be
redelivered if ACK is lost or the consumer crashes after the business operation
but before ACK. Payment processing needs durable idempotency and provider
idempotency, not just RabbitMQ ACK.
```

---

# 277. Senior Interview: Production Design

Question:

```text
Why did DLQ replay create duplicate orders?
```

Answer:

```text
Replay is another delivery attempt. If the consumer has no durable idempotency
mechanism, the original successful business effect can be executed again. Replay
must therefore be controlled and consumers must be duplicate-safe.
```

---

# 278. Senior Interview: Production Design

Question:

```text
Why does NACK requeue cause CPU spikes?
```

Answer:

```text
The message can immediately return to the queue and be delivered again,
creating a tight failure loop. Bounded retry with delay and a DLQ prevents the
consumer and broker from repeatedly processing the same poison message.
```

---

# 279. Senior Interview: Production Design

Question:

```text
What is the safest ACK boundary?
```

Answer:

```text
The ACK should follow the durable completion point of the business operation.
For a database-backed workflow, that is commonly after the required transaction
commits. For multi-system workflows, I use outbox, inbox, idempotency or saga
patterns as appropriate.
```

---

# 280. Senior Interview: Production Design

Question:

```text
Can I ACK when a downstream asynchronous job is queued?
```

Answer:

```text
Yes if the business contract defines successful handling as durable acceptance
of that downstream job. But the downstream publication itself must be durable
and recoverable. ACK does not mean the final business workflow is complete.
```

---

# 281. Senior Interview: Production Design

Question:

```text
Can I use the redelivered flag for idempotency?
```

Answer:

```text
No. It is useful for operational awareness, but it does not uniquely identify a
business event. I use a stable event ID or idempotency key.
```

---

# 282. Senior Interview: Production Design

Question:

```text
Can I use delivery_tag as idempotency key?
```

Answer:

```text
No. Delivery tags are channel-scoped delivery bookkeeping. They are not durable
global message identities.
```

---

# 283. Senior Interview: Production Design

Question:

```text
When is auto-ack acceptable?
```

Answer:

```text
For genuinely best-effort workloads where message loss after delivery is
acceptable, auto-ack may be reasonable. I would not use it for critical
business transactions that require reliable processing.
```

---

# 284. Senior Interview: Production Design

Question:

```text
How do you design acknowledgement for 100,000 messages per second?
```

Answer:

```text
I benchmark the client and broker, use efficient channels and controlled
prefetch, consider batching and multiple acknowledgements only where correctness
is provable, and keep business processing idempotent. I measure ACK rate,
processing latency, broker CPU, network and memory rather than optimizing ACKs in
isolation.
```

---

# 285. Senior Interview: Production Design

Question:

```text
How do you handle concurrent processing with multiple ACK?
```

Answer:

```text
I track delivery completion explicitly and only acknowledge a contiguous range
where every message in that range is known to have completed successfully.
Otherwise I prefer individual acknowledgements or a safer batching design.
```

---

# 286. Senior Interview: Production Design

Question:

```text
How do you handle database commit followed by ACK failure?
```

Answer:

```text
Assume the message can be redelivered. The business transaction must be
idempotent, usually through a unique event ID or inbox record. The duplicate
delivery then becomes a safe no-op and can be ACKed.
```

---

# 287. Senior Interview: Production Design

Question:

```text
How do you handle external API success followed by consumer crash?
```

Answer:

```text
Use an external idempotency key if supported. Otherwise maintain durable state
that records the operation and its external result so redelivery can determine
whether the side effect already occurred.
```

---

# 288. Senior Interview: Production Design

Question:

```text
How do you protect a database during a RabbitMQ backlog recovery?
```

Answer:

```text
Do not immediately maximize consumer replicas. Calculate backlog drain time,
scale gradually, cap concurrency and monitor database CPU, connections and
latency. The target is to drain the backlog without causing another outage.
```

---

# 289. Senior Interview: Production Design

Question:

```text
What metrics prove ACK health?
```

Answer:

```text
ACK rate, ACK latency, unacked count, unacked age, redelivery rate, NACK/reject
rate, processing latency and DLQ growth. I correlate them with business
completion metrics.
```

---

# 290. Senior Interview: Production Design

Question:

```text
What is your production ACK checklist?
```

Answer:

```text
Manual ACK for critical work, explicit completion boundary, durable
idempotency, correct delivery-tag handling, controlled prefetch, bounded retry,
DLQ, graceful shutdown, connection recovery, redelivery monitoring and failure
testing.
```

---

# 291. Advanced Failure Matrix

| Failure point | Possible result | Required design |
|---|---|---|
| Before delivery | message remains queued | normal queue durability |
| After delivery, before processing | redelivery possible | manual ACK |
| During processing | redelivery possible | idempotency |
| After DB commit, before ACK | duplicate delivery | inbox/unique constraint |
| After ACK | broker considers handled | ACK must not be premature |
| ACK network loss | redelivery ambiguity | idempotency |
| External API success, before ACK | duplicate API call | provider idempotency |
| NACK requeue | immediate retry | bounded retry |
| NACK no requeue | failure path | DLQ/DLX |
| Consumer crash | unacked recovery | graceful shutdown/recovery |
| Channel failure | unacked recovery | reconnect + idempotency |
| Deployment | redelivery | graceful rollout |
| Broker restart | reconnect/redelivery | recovery testing |

---

# 292. Advanced ACK State Machine

```text
RECEIVED
   |
VALIDATING
   |
   +---- invalid ----> FAILED
   |
PROCESSING
   |
   +---- transient ---> RETRYING
   |
   +---- permanent --> DLQ
   |
COMMITTING
   |
COMMITTED
   |
ACKING
   |
ACKED
```

---

# 293. Duplicate State Machine

```text
RECEIVED
   |
event_id lookup
   |
+--+-------------------+
|                      |
NEW                 EXISTING
|                      |
process              verify
|                      |
commit                no-op
|                      |
record ID              |
|                      |
+----------+-----------+
           |
          ACK
```

---

# 294. Retry State Machine

```text
FAILED
  |
classify
  |
+--+------------------+
|                     |
retryable          permanent
|                     |
attempt < limit       DLQ
|
backoff
|
retry
|
success -> ACK
|
failure -> classify again
```

---

# 295. Graceful Shutdown State Machine

```text
RUNNING
  |
SIGTERM
  |
DRAINING
  |
stop new work
  |
finish in-flight
  |
commit successful
  |
ACK successful
  |
close consumer
  |
close channel
  |
EXIT
```

---

# 296. ACK Architecture Decision Tree

```text
Is message business-critical?
 |
 +-- No --> auto/manual based on workload
 |
 Yes
 |
Manual ACK?
 |
 Yes
 |
Is DB transaction involved?
 |
 +-- Yes --> commit then ACK + idempotency
 |
 No
 |
Is external side effect involved?
 |
 +-- Yes --> provider/application idempotency + ACK
 |
 No
 |
ACK after durable business completion
```

---

# 297. Retry Decision Tree

```text
Processing failure
       |
       v
Is it transient?
   /          \
 yes          no
 |             |
retry         DLQ
 |
attempt limit?
 /       \
no       yes
|         |
backoff   DLQ
```

---

# 298. ACK Production Design Example

## Scenario

```text
10,000 orders/s
20 consumer pods
5 workers/pod
average processing = 20 ms
```

Approximate processing capacity requirement:

```text
20 pods
x
5 workers
=
100 workers
```

The theoretical concurrency needed at 10,000/s and 20 ms average processing is:

```text
10,000 × 0.020
=
200 concurrent operations
```

Therefore the initial worker count of 100 may not be sufficient for the target
throughput if each worker handles only one operation at a time.

This is a planning model; benchmark actual throughput.

---

# 299. ACK Capacity Example

Suppose:

```text
successful processing = 8,000/s
```

then expected ACK activity is approximately:

```text
8,000/s
```

before accounting for retries and other traffic.

If ACK rate is:

```text
2,000/s
```

while delivery is:

```text
8,000/s
```

investigate:

```text
processing latency
consumer failures
unacked growth
```

---

# 300. ACK Backlog Example

Suppose:

```text
deliver = 10,000/s
ACK = 8,000/s
```

Then unacknowledged work can grow by roughly:

```text
2,000/s
```

if the rates remain sustained.

This indicates consumer capacity or processing issues.

---

# 301. ACK Memory Example

Suppose:

```text
50 pods
5 workers/pod
prefetch=20
average message=256 KB
```

Potential in-flight message count:

```text
50 × 5 × 20
=
5,000
```

Approximate raw payload:

```text
5,000 × 256 KB
≈ 1.28 GB
```

Actual memory can be substantially higher due to runtime/object overhead.

---

# 302. ACK Recovery Example

Suppose:

```text
100 consumers
prefetch=100
```

and all consumers fail.

Potentially large amounts of work may become available for redelivery.

If the replacement fleet is immediately scaled to maximum:

```text
redelivery burst
+
normal traffic
```

can overload dependencies.

Recover gradually.

---

# 303. ACK and Backlog Drain

After recovery:

```text
incoming = 5,000/s
processing = 8,000/s
```

net drain:

```text
3,000/s
```

For:

```text
900,000 backlog
```

approximate drain:

```text
900,000 / 3,000
=
300 seconds
```

---

# 304. ACK and Failure Budget

If business processing SLO is:

```text
99.9% within 30 seconds
```

then ACK latency should be investigated as one contributor, but the complete
business path is what determines the SLO.

---

# 305. ACK and Incident Severity

Potential severity:

```text
Premature ACK
=
possible permanent message loss
```

while:

```text
Delayed ACK
=
possible backlog/reprocessing
```

Both matter, but they fail differently.

---

# 306. ACK Operational Principle

Prefer:

```text
safe duplicate
```

over:

```text
silent loss
```

for critical business events.

That is why:

```text
process -> commit -> ACK
```

plus:

```text
idempotency
```

is such a common production pattern.

---

# 307. ACK Golden Rules

```text
1. Treat ACK as a transport acknowledgement.
2. Do not treat ACK as a database commit.
3. Do not treat ACK as exactly-once processing.
4. ACK after required business completion.
5. Commit required DB state before ACK.
6. Expect redelivery.
7. Make business processing idempotent.
8. Use stable event IDs.
9. Do not use delivery tags as business IDs.
10. Understand channel-scoped delivery tags.
11. Use multiple ACK only with correct ordering logic.
12. Be careful with concurrent processing.
13. Tune prefetch.
14. Calculate in-flight work.
15. Avoid premature ACK.
16. Avoid uncontrolled requeue.
17. Classify retryable failures.
18. Bound retries.
19. Use backoff and jitter.
20. Use DLQ for persistent failures.
21. Test poison messages.
22. Monitor unacked messages.
23. Monitor redeliveries.
24. Monitor ACK latency.
25. Monitor NACK/reject rates.
26. Monitor DLQ growth.
27. Implement graceful shutdown.
28. Test crash-before-ACK.
29. Test crash-after-commit.
30. Test crash-after-ACK.
31. Test network failure.
32. Test broker failure.
33. Test duplicate delivery.
34. Test DLQ replay.
35. Protect external APIs with idempotency.
36. Use outbox for critical event publication.
37. Use inbox for durable deduplication where appropriate.
38. Do not let retry storms overload dependencies.
39. Scale consumers within downstream limits.
40. Keep failure recovery gradual.
41. Treat redelivery as normal.
42. Separate transport success from business success.
43. Document the ACK boundary.
44. Document retry behavior.
45. Document shutdown behavior.
```

---

# 308. Production Readiness Checklist

```text
Reliability
[ ] manual ACK
[ ] ACK after durable completion
[ ] idempotency
[ ] duplicate handling
[ ] connection recovery
[ ] channel recovery

Acknowledgement
[ ] delivery tags understood
[ ] multiple ACK reviewed
[ ] concurrent ACK logic tested
[ ] ACK metrics

Failure handling
[ ] NACK policy
[ ] reject policy
[ ] requeue policy
[ ] retry policy
[ ] DLQ
[ ] poison-message handling

Performance
[ ] prefetch benchmarked
[ ] concurrency benchmarked
[ ] memory calculated
[ ] ACK throughput measured
[ ] backlog drain calculated

Kubernetes
[ ] SIGTERM
[ ] readiness
[ ] termination grace
[ ] rolling deployment
[ ] PDB
[ ] failure-domain spread

Observability
[ ] ACK rate
[ ] ACK latency
[ ] unacked
[ ] redelivery
[ ] NACK
[ ] reject
[ ] DLQ
[ ] processing latency
[ ] business SLO

Testing
[ ] consumer crash
[ ] network failure
[ ] broker restart
[ ] DB failure
[ ] API failure
[ ] duplicate delivery
[ ] replay
[ ] concurrent ACK
```

---

# 309. Final Mental Model

```text
                  RabbitMQ
                      |
                   Delivery
                      |
                      v
                +-----------+
                | Consumer  |
                +-----+-----+
                      |
                  Validate
                      |
                 Idempotency
                      |
                      v
                +-----------+
                | Business  |
                | Transaction|
                +-----+-----+
                      |
                    Commit
                      |
                      v
                     ACK
```

Failure:

```text
Processing Failure
      |
      v
 Classify
   /    \
retry   permanent
 |         |
backoff    DLQ
 |
limit?
 |
yes -> DLQ
```

Duplicate:

```text
Redelivery
    |
event_id
    |
already processed?
 /              \
yes              no
 |                |
ACK            process
                  |
                commit
                  |
                 ACK
```

The most important production principle is:

```text
RabbitMQ gives you delivery and acknowledgement mechanisms.
Your application must provide business correctness.
```

---