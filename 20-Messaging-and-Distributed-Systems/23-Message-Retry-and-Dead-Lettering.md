# Message-Retry-and-Dead-Lettering

> Production-oriented notes on retry design, backoff, poison messages, dead-letter queues/topics, replay, failure isolation, Kafka and RabbitMQ implementation patterns, Kubernetes operations, observability, security, and incident troubleshooting.

---

# 1. Retry Fundamentals

A retry means attempting a failed operation again after the initial attempt fails.

In messaging systems, retries are required because many failures are temporary:

- network timeout
- temporary database outage
- broker connection interruption
- downstream service overload
- transient DNS failure
- temporary dependency throttling
- Kubernetes Pod restart
- leader movement
- short resource exhaustion

However, **not every failure should be retried**.

A production retry policy must answer:

```text
What failed?
Why did it fail?
Is the failure transient?
How many times should we retry?
How long should we wait?
Where should the message go after retries?
Can processing safely happen again?
```

---

# 2. Why Retry Design Matters

Poor retry behavior can turn a small dependency problem into a platform-wide outage.

Example:

```text
Payment API
    |
    X timeout
    |
Consumer retries immediately
    |
    X
    |
retry again
    |
    X
    |
retry again
    |
    v
Payment API overloaded
    |
    v
more timeouts
    |
    v
more retries
```

This is called a **retry storm**.

A good retry design protects both the messaging system and downstream dependencies.

---

# 3. Retry Goals

A production retry mechanism should:

1. recover transient failures
2. avoid retrying permanent failures
3. limit retry attempts
4. introduce delay
5. prevent synchronized retries
6. preserve message integrity
7. avoid duplicate business effects
8. isolate poison messages
9. provide operational visibility
10. support safe recovery and replay

---

# 4. Transient vs Permanent Failure

The first retry decision is failure classification.

## Transient failures

Examples:

```text
HTTP 503
HTTP timeout
database connection refused
temporary DNS failure
temporary broker unavailable
connection reset
rate limit
temporary dependency overload
```

These may succeed later.

## Permanent failures

Examples:

```text
invalid JSON
missing mandatory business field
invalid schema
unsupported event version
invalid account identifier
business rule violation
malformed message
```

Retrying these indefinitely usually makes the problem worse.

---

# 5. Failure Classification

A useful production classification is:

```text
Failure
 |
 +--> transient
 |      |
 |      +--> retry
 |
 +--> rate-limited
 |      |
 |      +--> delayed retry
 |
 +--> permanent
 |      |
 |      +--> DLQ
 |
 +--> unknown
        |
        +--> bounded retry
              |
              +--> DLQ if unresolved
```

Never let an `unknown` category become an infinite retry path.

---

# 6. Retryable HTTP Errors

Typical retry candidates can include:

```text
408 Request Timeout
429 Too Many Requests
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

The exact policy depends on the downstream API.

For example, `429` should respect server-provided retry information such as `Retry-After` when available.

---

# 7. Non-Retryable HTTP Errors

Common examples:

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
422 Unprocessable Entity
```

But the exact meaning depends on the API.

A `404` for a temporarily unavailable replicated resource might be retryable in one architecture and permanent in another.

---

# 8. Database Retry

Database failures should be classified carefully.

Potential transient conditions:

```text
connection timeout
connection reset
temporary failover
deadlock
temporary unavailable
```

Potential permanent conditions:

```text
constraint violation
invalid SQL
invalid data
missing required field
business rule violation
```

Retrying a deterministic constraint violation usually wastes resources.

---

# 9. Deadlocks

Database deadlocks can be transient.

Example:

```text
Transaction A locks row 1
Transaction B locks row 2

A requests row 2
B requests row 1

        DEADLOCK
```

The database may abort one transaction.

A bounded retry can be appropriate if the transaction is idempotent and safe to repeat.

---

# 10. Retry and Idempotency

Retries create duplicate execution risk.

Example:

```text
Consumer
   |
   | charge payment
   v
Payment provider
   |
   | success
   v
Consumer crashes before recording success
```

The consumer restarts and retries.

Without idempotency:

```text
payment charged twice
```

With a stable idempotency key:

```text
payment request
key = event_id
```

the provider can return the existing result instead of creating another charge.

---

# 11. Retry Does Not Mean Reprocess Blindly

A retry should repeat a well-defined operation.

Before retrying, consider:

```text
Was the operation partially completed?
Did the external system accept the request?
Did the database commit?
Was the response lost?
Could the side effect have happened?
```

This is especially important for:

- payments
- orders
- inventory
- notifications
- provisioning
- external APIs

---

# 12. Retry Attempt Number

Every retry system should track attempt count.

Example:

```text
attempt = 1
attempt = 2
attempt = 3
...
attempt = N
```

The attempt count can be stored in:

- message headers
- retry topic metadata
- queue metadata
- application state
- a dedicated retry record

Do not rely on an unbounded counter hidden only in application logs.

---

# 13. Maximum Attempts

Example policy:

```text
Initial attempt
   |
retry 1
   |
retry 2
   |
retry 3
   |
DLQ
```

Maximum attempts should be based on business requirements and expected recovery time.

A payment workflow may need different retry behavior from a non-critical analytics event.

---

# 14. Fixed Delay

A fixed-delay policy might be:

```text
retry 1 -> 10 seconds
retry 2 -> 10 seconds
retry 3 -> 10 seconds
```

Simple, but potentially inefficient during long outages.

Thousands of messages can wake up simultaneously.

---

# 15. Exponential Backoff

Exponential backoff increases delay after each failure.

Example:

```text
retry 1 -> 1s
retry 2 -> 2s
retry 3 -> 4s
retry 4 -> 8s
retry 5 -> 16s
```

Formula:

```text
delay = base * 2^(attempt-1)
```

Usually a maximum delay is applied.

---

# 16. Capped Exponential Backoff

Without a cap, delays can become extremely large.

Example:

```text
1s
2s
4s
8s
16s
32s
64s
128s
...
```

Use:

```text
delay = min(max_delay, base * 2^attempt)
```

Example:

```text
base = 2s
max = 5m
```

---

# 17. Jitter

If many messages fail at the same time, exponential backoff alone can still synchronize retries.

Jitter introduces randomness.

Example:

```text
Message A -> retry around 8.2s
Message B -> retry around 7.6s
Message C -> retry around 9.1s
Message D -> retry around 8.7s
```

This spreads load.

---

# 18. Full Jitter

A common approach:

```text
delay = random(0, exponential_delay)
```

Example:

```text
maximum delay = 16s

actual delay:
0-16 seconds
```

This is effective for avoiding synchronized retry waves.

---

# 19. Equal Jitter

Another approach:

```text
delay = half exponential delay + random half
```

Example:

```text
exponential = 16s

delay:
8s + random(0,8s)
```

Different retry algorithms can be appropriate for different workloads.

---

# 20. Decorrelated Jitter

Decorrelated jitter varies delay based on the previous delay.

The objective is the same:

```text
avoid synchronized retry traffic
```

The exact algorithm should be standardized by the platform team rather than independently reinvented by every service.

---

# 21. Retry Budget

A retry budget limits how much additional traffic retries can generate.

Example:

```text
Normal traffic = 10,000 requests/min

Maximum retry traffic = 2,000/min
```

This protects downstream systems.

Retries should never be allowed to consume unlimited capacity.

---

# 22. Retry Storm

A retry storm occurs when failed operations generate enough retries to further overload the failing dependency.

```text
Dependency failure
      |
      v
Requests fail
      |
      v
Retries increase
      |
      v
Dependency load increases
      |
      v
More failures
      |
      +----------+
                 |
                 v
             retry storm
```

Mitigation:

```text
backoff
jitter
retry limits
circuit breaker
load shedding
rate limits
DLQ
```

---

# 23. Thundering Herd

A thundering herd occurs when many waiting messages become eligible at the same time.

Example:

```text
Database unavailable for 60 seconds

100,000 messages waiting

Database recovers
        |
        v
100,000 retries immediately
```

The database may fail again.

Use delayed retries and jitter.

---

# 24. Retry Queue

RabbitMQ can implement retry patterns using dedicated retry queues.

Conceptually:

```text
Main Queue
    |
    X
    |
Retry Queue
    |
    | delay
    v
Main Queue
```

Each retry queue can have a configured delay mechanism.

---

# 25. RabbitMQ TTL-Based Retry

RabbitMQ retry architectures can use message TTL and dead-letter routing.

Concept:

```text
main.queue
   |
failure
   v
retry.queue
   |
TTL expires
   |
DLX
   |
   v
main.queue
```

Careful queue and dead-letter configuration is required.

---

# 26. RabbitMQ Dead-Letter Exchange

RabbitMQ supports dead-lettering through a Dead Letter Exchange (DLX).

Concept:

```text
Queue A
   |
   | rejected/expired
   v
DLX
   |
   v
DLQ
```

The DLX determines where dead-lettered messages are routed.

---

# 27. RabbitMQ Reject vs Requeue

Consumers can reject a message with different behavior.

Conceptually:

```text
reject + requeue=true
    |
    v
same queue

reject + requeue=false
    |
    v
dead-letter route if configured
```

Requeueing is not a retry strategy by itself.

Repeated immediate requeue can create a hot failure loop.

---

# 28. RabbitMQ Immediate Requeue Anti-Pattern

Bad:

```text
consume
  |
failure
  |
requeue immediately
  |
consume
  |
failure
  |
requeue
  |
...
```

This can create:

```text
CPU spike
network spike
consumer churn
broker pressure
downstream overload
```

Use delayed retry or bounded retry architecture instead.

---

# 29. RabbitMQ Retry Routing

A production routing design may be:

```text
orders.main
     |
     X
     |
orders.retry.10s
     |
     X
orders.retry.1m
     |
     X
orders.retry.5m
     |
     v
orders.dlq
```

The retry path is explicit and observable.

---

# 30. Kafka Retry Fundamentals

Kafka does not provide a traditional queue-level delayed-message feature in the same way a queue system can.

Common application patterns include:

```text
main topic
    |
    v
retry topic
    |
    v
delayed retry processing
    |
    v
main topic
```

or:

```text
main topic
    |
    v
retry topic 1
    |
    v
retry topic 2
    |
    v
DLQ topic
```

The exact architecture depends on latency and throughput requirements.

---

# 31. Kafka Retry Topics

Example:

```text
orders
orders.retry.1m
orders.retry.5m
orders.retry.30m
orders.dlq
```

A retry consumer reads the retry topic and republishes eligible messages.

Each retry stage can represent a different delay.

---

# 32. Kafka Retry Topic Headers

Useful metadata:

```text
original-topic
original-partition
original-offset
event-id
retry-count
first-failure-time
last-failure-time
failure-reason
exception-type
```

This makes troubleshooting and replay easier.

---

# 33. Preserve Original Metadata

When moving a message to a retry topic, preserve:

```text
event ID
correlation ID
trace ID
original topic
original partition
original offset
event type
event version
```

Do not lose the identity of the original event.

---

# 34. Kafka Retry and Ordering

Retry topics can affect ordering.

Example:

```text
Event A fails
Event B succeeds

A -> retry topic
B -> processed
```

Now B may complete before A.

If strict order matters, retry architecture must explicitly preserve the required ordering semantics.

---

# 35. Blocking Retry

A consumer may pause processing after a failure.

Example:

```text
partition
A -> failure
B -> waiting
C -> waiting
```

This preserves partition order but can reduce throughput dramatically.

Blocking retries should only be used when ordering is more important than throughput.

---

# 36. Non-Blocking Retry

A failed event moves to a retry path while other messages continue.

```text
A -> retry
B -> process
C -> process
D -> process
```

This improves throughput but may break strict event ordering.

Choose deliberately.

---

# 37. Retry Topic Partitioning

Retry topics should consider partition keys.

If ordering for an entity matters:

```text
order_id = 123
```

the retry event should be routed consistently so the relevant processing order can be maintained as far as the architecture requires.

---

# 38. DLQ Fundamentals

A Dead Letter Queue or Dead Letter Topic is a destination for messages that cannot be successfully processed under the normal retry policy.

```text
Message
   |
   v
Consumer
   |
 failure
   |
 retries
   |
 failure
   |
   v
DLQ
```

The DLQ is a **failure isolation mechanism**, not a garbage dump.

---

# 39. DLQ vs Retry

Retry means:

```text
"Try this message again later."
```

DLQ means:

```text
"Normal processing has failed enough that this message requires separate handling."
```

The two mechanisms solve different problems.

---

# 40. DLQ Entry Metadata

A useful DLQ record should include:

```text
event_id
event_type
event_version
original_topic
original_partition
original_offset
consumer_group
retry_count
first_failure_at
last_failure_at
failure_reason
exception_class
service_name
service_version
trace_id
correlation_id
```

---

# 41. Failure Reason

Do not store only:

```text
failed=true
```

Store actionable information:

```text
failure_reason = "database connection timeout"
```

or:

```text
failure_reason = "schema validation failed"
```

This makes incident response much faster.

---

# 42. Original Payload

DLQ systems often need the original message.

However, payload retention must follow security and data-governance requirements.

Avoid copying sensitive data into multiple uncontrolled systems.

---

# 43. DLQ Retention

DLQ retention should be deliberate.

Consider:

```text
How quickly are failures investigated?
How long must business data be recoverable?
What compliance requirements exist?
How much storage will accumulate?
Can messages be safely replayed?
```

A DLQ retained forever can become an operational liability.

---

# 44. DLQ Ownership

Every production DLQ needs an owner.

Define:

```text
team
service
topic
business purpose
runbook
retention
replay procedure
alert threshold
```

An ownerless DLQ eventually becomes invisible technical debt.

---

# 45. DLQ Alerting

Alert on meaningful DLQ conditions:

```text
DLQ message count > threshold
DLQ growth rate increases
critical event enters DLQ
DLQ oldest message age > threshold
```

Do not page on every isolated bad message unless the business requires it.

---

# 46. DLQ Growth

Example:

```text
10:00 -> 2
10:10 -> 15
10:20 -> 200
10:30 -> 5,000
```

This indicates a systemic problem.

Investigate:

```text
schema change
deployment
dependency outage
credential issue
database issue
business validation change
```

---

# 47. Poison Message

A poison message is a message that repeatedly fails because of deterministic conditions.

Examples:

```text
invalid JSON
unsupported version
invalid enum
missing field
corrupt payload
permanent business violation
```

Do not allow it to consume infinite retry capacity.

---

# 48. Poison Message Isolation

Typical flow:

```text
main
 |
retry 1
 |
retry 2
 |
retry 3
 |
DLQ
```

This allows healthy messages to continue.

---

# 49. DLQ Replay

Replay means sending a failed message back through normal processing after the underlying problem has been fixed.

Example:

```text
DLQ
 |
inspect
 |
fix problem
 |
replay
 |
main topic/queue
 |
consumer
```

Replay should be controlled and observable.

---

# 50. Replay Preconditions

Before replay:

```text
[ ] root cause understood
[ ] consumer version compatible
[ ] schema compatible
[ ] duplicate behavior understood
[ ] downstream dependency healthy
[ ] replay volume estimated
[ ] idempotency verified
[ ] monitoring enabled
```

---

# 51. Replay Is Not Automatically Safe

Suppose a payment event entered DLQ because the consumer timed out.

The payment provider may have actually processed the payment.

Replaying blindly could charge the customer again.

Always determine whether the external side effect already happened.

---

# 52. Replay Categories

Useful replay categories:

```text
single-message replay
small batch replay
time-window replay
partition-range replay
topic-range replay
full historical replay
```

Start with the smallest safe scope.

---

# 53. Replay Rate Limiting

Do not dump millions of DLQ records into the main system at once.

Use:

```text
rate limit
batch size
concurrency limit
pause/resume
monitoring
```

Replay should behave like controlled traffic, not an emergency flood.

---

# 54. Replay Audit

Every replay should be auditable.

Record:

```text
who initiated replay
when
source DLQ
message/event IDs
reason
destination
number replayed
number failed
result
```

This is important for production and regulated workloads.

---

# 55. Manual vs Automated Replay

Manual replay is safer for high-impact events.

Automated replay can be appropriate for well-understood technical failures.

Example:

```text
temporary DNS failure
   |
automatic retry
```

versus:

```text
payment duplicate risk
   |
manual verification
```

---

# 56. Retry and Consumer Offset

Kafka consumers must carefully coordinate processing and offset commits.

Bad pattern:

```text
commit offset
   |
process message
   |
failure
```

The message may be skipped after restart.

Another risky pattern:

```text
process side effect
   |
crash
   |
offset not committed
```

The message may be processed again.

Therefore, idempotency is essential.

---

# 57. Kafka Offset Commit Strategy

Conceptually:

```text
poll
 |
process
 |
successful business transaction
 |
commit offset
```

The exact implementation depends on the processing model.

Do not blindly enable auto-commit for critical workflows without understanding the consequences.

---

# 58. RabbitMQ Acknowledgement Strategy

A consumer should acknowledge according to the required delivery semantics.

Concept:

```text
receive
 |
process
 |
business operation successful
 |
ACK
```

If processing fails:

```text
receive
 |
failure
 |
retry/reject/DLQ
```

Acknowledging too early can cause message loss.

---

# 59. ACK vs Retry

ACK means the broker can consider the delivery handled.

Retry means the application needs another processing attempt.

Do not ACK merely because the application received the message.

---

# 60. Failure Before ACK

Example:

```text
RabbitMQ
 |
message
 v
consumer
 |
database update succeeds
 |
consumer crashes
 |
ACK never sent
 |
message delivered again
```

This demonstrates why business processing must be idempotent.

---

# 61. Failure After ACK

Example:

```text
consumer
 |
ACK
 |
process crashes before business operation
```

If ACK happened before processing, the broker may not redeliver the message.

This can cause message loss.

---

# 62. Retry and Transaction Boundary

The most important question is:

```text
When exactly is the message considered successfully processed?
```

For database-backed consumers, ideally:

```text
message
 |
DB transaction
 |-- business update
 |-- inbox/event ID
 |
commit
 |
ACK / offset commit
```

This aligns durable business state with message acknowledgement.

---

# 63. Retry and External Transactions

External APIs usually cannot participate in the consumer's local transaction.

Therefore:

```text
Kafka transaction
        X
external payment transaction
```

are separate.

Use:

```text
idempotency
durable state
reconciliation
provider transaction IDs
```

---

# 64. Retry State Machine

A useful conceptual state machine:

```text
RECEIVED
   |
PROCESSING
   |
   +---- SUCCESS
   |
   +---- RETRY_PENDING
             |
             +---- RETRYING
             |       |
             |       +---- SUCCESS
             |       |
             |       +---- RETRY_PENDING
             |
             +---- DLQ
```

This makes operational behavior explicit.

---

# 65. Retry Headers

Useful headers:

```text
x-event-id
x-retry-count
x-original-topic
x-original-partition
x-original-offset
x-first-failure-at
x-last-failure-at
x-failure-reason
x-correlation-id
x-trace-id
```

Header names should follow organizational standards.

---

# 66. Retry Topic Naming

Example:

```text
orders
orders.retry.30s
orders.retry.5m
orders.retry.30m
orders.dlq
```

Naming should make the purpose obvious.

Avoid:

```text
topic2
topic-final
topic-new
topic-temp
```

---

# 67. Retry Queue Naming

RabbitMQ example:

```text
orders.main
orders.retry.10s
orders.retry.1m
orders.retry.5m
orders.dlq
```

The naming convention should be standardized across teams.

---

# 68. Retry Policy Per Failure Type

Example:

```text
database timeout
    -> retry

HTTP 503
    -> retry

HTTP 429
    -> delayed retry

invalid schema
    -> DLQ

authentication failure
    -> alert + controlled retry

business validation failure
    -> DLQ/business workflow
```

Do not use one retry policy for every exception.

---

# 69. Retry Policy Configuration

Example conceptual configuration:

```yaml
retry:
  maxAttempts: 5
  initialDelay: 2s
  maxDelay: 5m
  backoff: exponential
  jitter: full
  dlq: orders.dlq
```

Production configuration should be version-controlled and reviewed.

---

# 70. Kubernetes Configuration

Retry settings should be configurable through Kubernetes configuration mechanisms.

Example:

```yaml
env:
  - name: RETRY_MAX_ATTEMPTS
    value: "5"
  - name: RETRY_INITIAL_DELAY
    value: "2s"
  - name: RETRY_MAX_DELAY
    value: "5m"
```

Sensitive values should use Secrets, while normal retry settings can use ConfigMaps or deployment configuration.

---

# 71. Kubernetes Restart Is Not Retry Strategy

A common mistake is assuming:

```text
Pod crashes
   |
Kubernetes restarts Pod
   |
message automatically solved
```

Restarting a Pod does not solve:

- invalid payload
- schema incompatibility
- deterministic business error
- downstream permanent rejection

It can actually cause repeated processing.

---

# 72. CrashLoopBackOff and Messaging

If a consumer crashes on one poison message:

```text
message
 |
consumer crash
 |
Pod restart
 |
same message
 |
consumer crash
 |
Pod restart
```

This can create a crash loop.

The message should be isolated through controlled retry/DLQ handling.

---

# 73. Kubernetes Probes

Be careful with liveness probes.

A consumer should not be restarted merely because:

```text
Kafka temporarily unavailable
```

unless the application is genuinely unhealthy.

Poor probe design can create:

```text
Kafka outage
 -> Pod restarts
 -> reconnect storm
 -> more load
```

---

# 74. Readiness Probe

Readiness should represent whether the consumer should receive traffic or participate in the workload.

For messaging consumers, the exact readiness semantics depend on the framework.

Avoid declaring a consumer ready while it cannot safely process messages.

---

# 75. Graceful Shutdown

On termination:

```text
SIGTERM
 |
stop accepting new work
 |
finish safe in-flight processing
 |
commit correct acknowledgement/offset
 |
close connections
 |
exit
```

Forced termination during processing increases duplicate risk.

---

# 76. Kubernetes Rolling Deployment

During a rollout:

```text
old Pods
   |
new Pods start
   |
consumer group rebalances
   |
partitions move
   |
old Pods terminate
```

Monitor:

```text
consumer lag
rebalance count
processing errors
duplicate rate
DLQ
```

---

# 77. Consumer Rebalance

Frequent Kafka consumer rebalances can affect retry behavior.

Symptoms:

```text
processing pauses
latency increases
lag grows
duplicate processing may increase
```

Investigate:

```text
slow processing
session timeout
heartbeat issues
deployment churn
resource starvation
unstable networking
```

---

# 78. Retry and Partition Blocking

If a consumer blocks a Kafka partition while waiting for a retry:

```text
A fails
 |
wait 5 minutes
 |
B cannot process
 |
C cannot process
```

This may be correct for strict ordering but harmful for throughput.

Use non-blocking retry if ordering constraints allow it.

---

# 79. Retry and Consumer Concurrency

Increasing concurrency can improve throughput but does not automatically solve:

```text
hot partition
database bottleneck
external API rate limit
ordered workflow
```

Retry architecture must account for the actual bottleneck.

---

# 80. Retry and Backpressure

When retry volume increases:

```text
normal traffic
+
retry traffic
=
total workload
```

Consumers and downstream services must have capacity for both.

If retry traffic consumes all capacity, healthy messages may starve.

---

# 81. Priority Between New and Retry Traffic

A production design may need to choose:

```text
new messages first
```

or:

```text
retry messages first
```

or:

```text
weighted scheduling
```

For many systems, retry traffic should be bounded so new healthy traffic remains available.

---

# 82. Retry Starvation

If a retry queue is always full:

```text
new messages
   |
   v
retry workload dominates
   |
   v
new work delayed
```

This can become a business-impacting issue.

Use quotas or separate processing capacity where required.

---

# 83. DLQ as Pressure Relief

DLQ provides a stopping point:

```text
main
 |
retry
 |
retry
 |
DLQ
```

Without it:

```text
main
 |
retry
 |
retry
 |
retry forever
```

Infinite retry is almost always an operational smell.

---

# 84. DLQ Is Not a Permanent Storage Strategy

DLQ should not replace:

- proper data storage
- business state
- audit systems
- backup
- event retention

It exists for failed message isolation and controlled recovery.

---

# 85. DLQ Security

DLQs can contain full production payloads.

Apply:

```text
authentication
authorization
encryption
least privilege
retention policy
audit logging
data classification
```

Do not allow every developer account to read sensitive DLQs.

---

# 86. DLQ Data Privacy

If an event contains sensitive customer data, copying it into:

```text
main topic
retry topic
DLQ
logs
metrics
debug database
```

can multiply exposure.

Design failure handling with data minimization.

---

# 87. Observability for Retry

Metrics should include:

```text
retry_attempt_total
retry_success_total
retry_failure_total
retry_latency
retry_queue_depth
dlq_messages_total
dlq_oldest_message_age
```

Exact metric names can vary.

---

# 88. Consumer Error Rate

Monitor:

```text
errors / messages processed
```

A rising error rate is often more useful than raw error count.

Example:

```text
100 errors / 1,000,000 messages = 0.01%

100 errors / 200 messages = 50%
```

---

# 89. Retry Rate

A useful signal:

```text
retry rate =
retry attempts / total processing attempts
```

Sudden increases can indicate a downstream outage or bad deployment.

---

# 90. DLQ Rate

Monitor:

```text
DLQ rate =
DLQ messages / processed messages
```

Separate critical business events from low-value telemetry.

---

# 91. Oldest DLQ Message Age

A DLQ containing:

```text
10 messages
```

may be fine.

But:

```text
10 messages
oldest = 45 days
```

may indicate that the organization has no operational recovery process.

---

# 92. Retry Latency

Track:

```text
failure time
   |
retry time
   |
success time
```

This helps determine whether retry policy is too aggressive or too slow.

---

# 93. Distributed Tracing

Retry attempts should preserve trace/correlation information where appropriate.

Useful fields:

```text
trace_id
correlation_id
event_id
retry_count
```

Create clear spans for:

```text
consume
process
database
external API
retry
```

---

# 94. Logging Retry Failures

Good structured log:

```json
{
  "event_id": "abc",
  "retry_count": 3,
  "topic": "orders",
  "partition": 4,
  "failure": "database timeout",
  "next_action": "retry"
}
```

Avoid logging complete sensitive payloads.

---

# 95. Alert Examples

Examples:

```text
Consumer retry rate > 20% for 10 minutes
DLQ growth > 100 messages/minute
Oldest DLQ message > 30 minutes
Consumer lag age > threshold
Retry topic backlog > threshold
Critical payment events entering DLQ
```

Thresholds should be based on business requirements.

---

# 96. Retry Runbook

A production runbook should answer:

```text
1. Which component is failing?
2. Is the failure transient?
3. Are retries increasing?
4. Is downstream capacity affected?
5. Is DLQ growing?
6. Should retries be paused?
7. Should traffic be reduced?
8. Is replay safe?
9. Has the root cause been fixed?
10. How will recovery be validated?
```

---

# 97. Incident: Database Outage

Scenario:

```text
Consumer
   |
DB timeout
   |
retry rate rises
```

Response:

```text
1. Confirm database health.
2. Stop uncontrolled retries.
3. Apply backoff.
4. Check consumer lag.
5. Check downstream saturation.
6. Verify database recovery.
7. Allow controlled retry.
8. Monitor DLQ and duplicates.
```

---

# 98. Incident: Invalid Schema

Scenario:

```text
new producer deployment
       |
       v
old consumer rejects event
       |
       v
retry
       |
       v
retry
       |
       v
DLQ
```

Do not repeatedly retry a deterministic schema mismatch.

Fix compatibility first, then replay affected events.

---

# 99. Incident: Authentication Failure

Scenario:

```text
consumer
   |
401/403
   |
retry
```

Repeated retries may be pointless if credentials are invalid.

Response:

```text
check secret
check token
check ACL
check certificate
fix authentication
then replay/retry
```

---

# 100. Incident: Rate Limiting

Scenario:

```text
API
 |
429
 |
consumer retries immediately
 |
more 429
```

Use:

```text
Retry-After
backoff
jitter
rate limiting
concurrency reduction
```

Do not fight the provider's rate limit.

---

# 101. Incident: Payment Timeout

Scenario:

```text
payment request
 |
timeout
```

Do not assume:

```text
timeout = payment failed
```

The provider may have processed it.

First query the provider using the idempotency key or transaction identifier where supported.

---

# 102. Incident: Poison Message

Scenario:

```text
message
 |
failure
 |
retry
 |
failure
 |
retry
```

Response:

```text
identify deterministic failure
isolate message
send to DLQ
alert owner
fix producer/consumer
replay after validation
```

---

# 103. Incident: DLQ Explosion

Symptoms:

```text
DLQ = 0
DLQ = 100
DLQ = 10,000
DLQ = 1,000,000
```

Investigate:

```text
recent deployments
schema changes
dependency outage
credential changes
database failures
topic configuration
consumer code
```

Do not start replay until the root cause is fixed.

---

# 104. Incident: Retry Storm

Immediate actions:

```text
identify failing dependency
reduce retry concurrency
increase backoff
enable jitter
pause non-critical retry flow
protect downstream service
```

Then investigate the root cause.

---

# 105. Incident: Consumer Crash Loop

If a consumer crashes on one message:

```text
consumer
 |
poison message
 |
crash
 |
restart
 |
same message
 |
crash
```

Move the message to a controlled failure path rather than repeatedly restarting the consumer.

---

# 106. Incident: Kafka Lag During Retry

If retry processing consumes all consumer capacity:

```text
retry backlog increases
new message lag increases
```

Separate retry capacity from normal traffic if necessary.

---

# 107. Incident: RabbitMQ Queue Growth

If:

```text
ready messages ↑
unacked messages ↑
```

investigate:

```text
consumer throughput
consumer failures
prefetch
downstream latency
retry loop
broker resources
```

Do not simply add consumers without understanding the bottleneck.

---

# 108. RabbitMQ Prefetch and Retry

Prefetch controls how many messages can be delivered without acknowledgement.

Very high prefetch can cause:

```text
many unacked messages
large failure scope
uneven load
memory pressure
```

Very low prefetch can reduce throughput.

Tune based on workload.

---

# 109. Kafka Consumer Poll Behavior

Long processing can interfere with consumer group stability if processing exceeds configured consumer timing expectations.

Design processing so that:

```text
poll
process
poll
```

remains healthy, or use the framework's supported pattern for long-running work.

---

# 110. Retry and Message Ordering

There is no universal answer.

If business rule says:

```text
OrderCreated
must happen before
OrderCancelled
```

then retrying OrderCreated after OrderCancelled may be problematic.

Possible solutions:

```text
partition by order ID
state validation
sequence numbers
blocking retry
workflow orchestration
```

---

# 111. Sequence Numbers

Events can carry:

```text
entity_id
sequence_number
```

Example:

```text
order-123 sequence 1
order-123 sequence 2
order-123 sequence 3
```

Consumers can detect gaps or out-of-order processing.

---

# 112. Retry and Event Time

A retry may happen much later than the original event.

Always distinguish:

```text
occurred_at
first_failed_at
last_attempt_at
successful_at
```

This is useful for both operations and business analytics.

---

# 113. Retry Expiration

Some messages should not be retried forever.

Example:

```text
event age > 24 hours
```

At that point:

```text
DLQ
manual review
compensation
```

The appropriate policy depends on business value.

---

# 114. Time-Based Retry Policy

Example:

```text
0-5 min:
aggressive transient retries

5-30 min:
moderate retries

30 min-2 hours:
slow retries

>2 hours:
DLQ/manual workflow
```

This can prevent stale business actions from executing much later.

---

# 115. Business Deadline

A message may have a business deadline.

Example:

```text
flash sale inventory event
deadline = 15 minutes
```

Retrying it after the deadline may no longer make sense.

Technical retry policy should respect business validity windows.

---

# 116. Retry and SLA

Suppose:

```text
business SLA = 5 minutes
```

and retry policy is:

```text
1m + 2m + 4m + 8m
```

The policy already violates the SLA.

Retry configuration must be derived from service objectives, not arbitrary numbers.

---

# 117. Retry and SLO

Monitor:

```text
successful processing latency
retry processing latency
DLQ rate
```

A service may have high success rate but poor latency because most events succeed only after several retries.

---

# 118. Retry Cost

Retries consume:

```text
CPU
memory
network
broker storage
database connections
external API quota
engineering attention
```

Therefore, retries have real operational cost.

---

# 119. Retry Capacity Planning

Estimate:

```text
normal traffic
+
expected retry traffic
+
failure burst
```

Example:

```text
normal = 20k/min
expected retry = 5k/min
failure burst = 50k/min
```

The architecture should have a strategy for the burst scenario.

---

# 120. Failure Isolation

A good messaging architecture isolates failures:

```text
Payment failure
     |
     v
Payment retry/DLQ

Inventory
     |
continues independently
```

Avoid allowing one failing consumer to block unrelated business flows.

---

# 121. Separate DLQs

Separate DLQs can improve ownership:

```text
payment.dlq
inventory.dlq
notification.dlq
```

instead of:

```text
everything.dlq
```

unless centralized DLQ management is deliberately designed.

---

# 122. Central DLQ

A central DLQ platform can provide:

```text
standard metadata
search
dashboards
replay controls
access control
audit
```

But ownership must remain clear.

---

# 123. DLQ Topic Per Consumer

A Kafka consumer-specific DLQ can isolate failures:

```text
orders
 |
 +--> payment-group
 |       |
 |       +--> payment.dlq
 |
 +--> inventory-group
         |
         +--> inventory.dlq
```

This is useful because the same event may be valid for one consumer and invalid for another.

---

# 124. Do Not Put Every Failure in DLQ

Some failures should be retried automatically.

Examples:

```text
temporary network timeout
temporary database unavailable
HTTP 503
```

DLQ is generally for messages that cannot currently be processed under normal bounded retry policy.

---

# 125. Retry vs Circuit Breaker

Retry handles an individual operation failure.

Circuit breaker protects a dependency when failures become widespread.

```text
Retry:
message-level behavior

Circuit breaker:
dependency-level protection
```

They work together.

---

# 126. Circuit Breaker States

Typical:

```text
CLOSED
  |
  | failures
  v
OPEN
  |
  | wait
  v
HALF-OPEN
  |
  +--> success -> CLOSED
  |
  +--> failure -> OPEN
```

This prevents continuous calls to an unhealthy dependency.

---

# 127. Retry With Circuit Breaker

Example:

```text
consumer
 |
circuit breaker
 |
dependency
```

When the circuit is open:

```text
do not perform immediate downstream calls
```

Messages can enter a controlled delayed retry path.

---

# 128. Retry and Rate Limiter

Rate limiting controls outbound processing rate.

Useful when:

```text
external API = 1,000 requests/min
consumer capacity = 10,000/min
```

Without rate limiting, retry can quickly violate provider limits.

---

# 129. Retry and Bulkhead

Bulkhead isolation allocates separate capacity.

Example:

```text
Payment processing pool
Inventory processing pool
Notification processing pool
```

Payment retries should not consume all resources needed for inventory.

---

# 130. Retry and Load Shedding

During severe dependency failure, low-priority messages may be delayed or dropped according to business policy.

Critical events should receive appropriate protection.

Load shedding should be explicit and observable.

---

# 131. Retry Priority

Possible priorities:

```text
P0 critical business event
P1 important
P2 normal
P3 analytics
```

The platform can apply different retry and DLQ policies.

---

# 132. Retry for Analytics

Analytics events may tolerate:

```text
longer delay
some loss
batch replay
```

Payment events usually require:

```text
strong durability
idempotency
reconciliation
auditing
```

One policy should not blindly serve both.

---

# 133. Retry for Notifications

Notification retries should consider:

```text
duplicate user-visible message
provider rate limits
message expiration
user preference changes
```

A notification that becomes stale may be better dropped or compensated than delivered hours later.

---

# 134. Retry for Inventory

Inventory operations can be sensitive to order and duplicates.

Use:

```text
idempotency key
atomic state transition
entity ordering
reconciliation
```

---

# 135. Retry for Order Processing

Order workflows may have multiple steps:

```text
order
 |
payment
 |
inventory
 |
shipment
```

Failure handling should be modeled as a workflow rather than independent blind retries.

---

# 136. Retry for File Processing

Large file events may trigger expensive processing.

Retries can cause:

```text
duplicate downloads
duplicate parsing
duplicate database writes
```

Use a stable file/event identifier and checkpointing where appropriate.

---

# 137. Retry for Kubernetes Jobs

A Kubernetes Job has its own retry semantics.

Do not confuse:

```text
Kubernetes Job retry
```

with:

```text
message retry
```

Both layers may retry, producing multiplicative attempts.

---

# 138. Multiplicative Retry

Example:

```text
message retry = 5
Job retry = 6
HTTP client retry = 3
```

Worst-case attempts can multiply dramatically.

Centralize retry ownership where possible.

---

# 139. Layered Retry Anti-Pattern

Bad:

```text
Kafka consumer
   |
framework retry x5
   |
HTTP client retry x5
   |
service retry x5
   |
database retry x5
```

This can produce hundreds or thousands of attempts.

Each layer must understand the retry behavior below it.

---

# 140. Single Retry Owner

A cleaner approach:

```text
consumer
 |
business operation
 |
bounded internal retry where required
 |
message-level retry
 |
DLQ
```

Define which layer owns each retry responsibility.

---

# 141. Retry Documentation

Each service should document:

```text
retryable exceptions
non-retryable exceptions
max attempts
backoff
jitter
DLQ
replay
idempotency strategy
ordering impact
business deadline
owner
```

---

# 142. Configuration Drift

If every service independently configures:

```text
maxAttempts
backoff
DLQ
headers
```

standards can diverge.

Platform teams should provide reusable libraries/templates where practical.

---

# 143. Retry Library

A shared library can standardize:

```text
classification
backoff
jitter
metrics
trace propagation
headers
logging
```

But services must still provide business-specific idempotency and failure classification.

---

# 144. Testing Retry Logic

Unit tests should cover:

```text
transient failure
permanent failure
max attempts
backoff calculation
jitter
success after retry
failure after max retries
DLQ routing
metadata preservation
```

---

# 145. Integration Testing

Test:

```text
broker
consumer
database
downstream API
retry topic/queue
DLQ
```

Verify actual behavior rather than only mocked exceptions.

---

# 146. Failure Injection

Production readiness requires controlled failure testing.

Inject:

```text
database timeout
HTTP 503
HTTP 429
network delay
consumer crash
broker restart
schema mismatch
credential failure
```

Verify retry and DLQ behavior.

---

# 147. Chaos Testing

Chaos experiments can validate:

```text
dependency outage
consumer restart
node failure
network partition
broker failover
```

The goal is not merely to prove that Pods restart.

The goal is to prove that **business processing remains correct**.

---

# 148. Retry Metrics Dashboard

A useful dashboard:

```text
+------------------------------+
| Message Processing           |
+------------------------------+
| throughput                   |
| success rate                 |
| error rate                   |
| retry rate                   |
| retry latency                |
+------------------------------+

+------------------------------+
| DLQ                          |
+------------------------------+
| message count                |
| growth rate                  |
| oldest message age           |
+------------------------------+
```

---

# 149. Retry Alert Severity

Example:

```text
INFO:
small transient retry spike

WARNING:
retry rate sustained

CRITICAL:
DLQ growth + business-critical events failing
```

Alert severity should reflect impact.

---

# 150. Production Architecture

A robust Kafka pattern:

```text
                 Producer
                    |
                    v
                Main Topic
                    |
                    v
                 Consumer
                    |
          +---------+---------+
          |                   |
       success              failure
          |                   |
          v                   v
       offset            retry policy
                              |
                   +----------+----------+
                   |                     |
                retryable            permanent
                   |                     |
                   v                     v
              Retry Topic              DLQ
                   |
              delayed retry
                   |
                   v
              Main Topic
```

---

# 151. Production RabbitMQ Architecture

```text
Producer
   |
Exchange
   |
Main Queue
   |
Consumer
   |
   +---- success ---> ACK
   |
   +---- transient -> Retry Queue
   |                    |
   |                    | delay
   |                    v
   |                 Main Queue
   |
   +---- permanent ---> DLX ---> DLQ
```

---

# 152. Retry Architecture Principles

A production design should provide:

```text
bounded retries
backoff
jitter
failure classification
idempotency
DLQ
observability
controlled replay
security
ownership
```

Missing one of these can create operational gaps.

---

# 153. Golden Rule: Never Infinite Retry

Infinite retry is attractive because it sounds reliable.

In reality:

```text
infinite retry
=
infinite resource consumption
+
hidden failure
+
possible outage amplification
```

Use bounded retry and explicit recovery.

---

# 154. Golden Rule: Retry Only What May Recover

If a message is invalid:

```text
retrying invalid data
```

does not make it valid.

Fix the underlying cause and replay later.

---

# 155. Golden Rule: Assume Duplicate Side Effects

Every retry-capable consumer should answer:

```text
What if this message is processed twice?
```

If the answer is:

```text
customer charged twice
```

the design is incomplete.

---

# 156. Golden Rule: Preserve Event Identity

Do not create a new logical event identity every time the same event enters a retry path.

Preserve:

```text
event_id
```

and add:

```text
retry_count
```

---

# 157. Golden Rule: DLQ Must Be Actionable

A DLQ without:

```text
owner
alert
reason
metadata
replay procedure
```

is only delayed failure.

---

# 158. Golden Rule: Replay Only After Root Cause Fix

Bad:

```text
DLQ huge
 |
replay everything
```

Better:

```text
identify cause
 |
fix cause
 |
test fix
 |
replay small sample
 |
validate
 |
replay controlled batches
```

---

# 159. Golden Rule: Protect Downstream Systems

Messaging infrastructure can absorb huge backlogs.

That does not mean downstream systems can process them instantly.

Always consider:

```text
database capacity
API rate limits
external quotas
connection pools
CPU
memory
```

---

# 160. Golden Rule: Monitor Retry Age

Count alone is insufficient.

Track:

```text
oldest retry age
oldest DLQ age
```

A small but old backlog can be operationally worse than a large recent backlog.

---

# 161. Golden Rule: Retry Policy Is Part of Architecture

Do not treat retry configuration as a minor code detail.

It determines:

```text
latency
load
availability
duplicate risk
data recovery
incident behavior
```

---

# 162. Golden Rule: Test Failure Paths

Most teams test:

```text
happy path
```

Production failures happen in:

```text
retry
timeout
crash
duplicate
DLQ
replay
```

Test these deliberately.

---

# 163. Golden Rule: Make Recovery Boring

A good production system makes recovery predictable:

```text
failure
 |
bounded retry
 |
DLQ
 |
alert
 |
root cause fix
 |
controlled replay
 |
validation
```

If recovery requires improvisation, the system is not operationally mature.

---

# 164. Senior Interview: Explain Retry Strategy

A strong answer:

> I classify failures into transient and permanent categories. Transient failures use bounded retries with exponential backoff and jitter. Permanent failures go to a DLQ. I make consumers idempotent because retries can produce duplicate processing. I also monitor retry rate, lag, DLQ growth and oldest-message age, and I provide controlled replay after the root cause is fixed.

---

# 165. Senior Interview: Why Not Retry Everything?

Because permanent failures do not become successful through repetition, and excessive retries can overload dependencies and create retry storms.

---

# 166. Senior Interview: What Is a Poison Message?

A deterministic message that repeatedly fails processing because of malformed data, incompatible schema or a permanent business condition.

It should be isolated rather than retried indefinitely.

---

# 167. Senior Interview: What Is a DLQ?

A destination for messages that could not be successfully processed after the defined retry policy. It enables failure isolation, investigation and controlled replay.

---

# 168. Senior Interview: Is DLQ a Backup?

No.

A DLQ is a failure-handling mechanism. It should not replace backups, event retention or durable business data storage.

---

# 169. Senior Interview: How Do You Replay a DLQ?

First identify and fix the root cause, verify consumer compatibility and idempotency, replay a small sample, validate business results, then replay remaining messages at a controlled rate while monitoring the system.

---

# 170. Senior Interview: Kafka Retry Design

A strong answer:

> Kafka does not behave like a traditional delayed queue, so I normally use application-level retry topics or a supported retry mechanism. I preserve the original event metadata, track retry attempts, apply delay and backoff, route permanently failed records to a DLQ topic, and ensure replay is idempotent.

---

# 171. Senior Interview: RabbitMQ Retry Design

A strong answer:

> I avoid immediate requeue loops. For transient failures I use bounded delayed retries, commonly through retry queues and dead-letter routing. Permanent failures are routed to a DLQ. The consumer acknowledges only after the required business processing succeeds.

---

# 172. Senior Interview: Retry and Ordering

A strong answer:

> Retry can change processing order. If ordering matters, I define the ordering scope first and choose a retry mechanism that preserves that requirement, potentially by blocking processing or using entity-based partitioning and state validation.

---

# 173. Senior Interview: Retry and Exactly Once

A strong answer:

> Retry semantics alone do not guarantee exactly-once business effects. External side effects can be repeated, so I use idempotency keys, durable state and reconciliation where required.

---

# 174. Senior Interview: Retry Storm

A retry storm occurs when failures cause aggressive repeated attempts that further overload the failing dependency. I prevent it using backoff, jitter, retry budgets, circuit breakers, rate limiting and bounded attempts.

---

# 175. Senior Interview: What Metrics Do You Monitor?

```text
processing throughput
success rate
error rate
retry rate
retry latency
consumer lag
lag age
retry backlog
DLQ count
DLQ growth
DLQ oldest age
downstream latency
```

---

# 176. Production Checklist

```text
RETRY
[ ] failure classification defined
[ ] retryable errors documented
[ ] permanent errors documented
[ ] max attempts configured
[ ] exponential backoff evaluated
[ ] jitter enabled
[ ] maximum delay configured
[ ] retry budget considered

IDEMPOTENCY
[ ] event ID available
[ ] duplicate processing tested
[ ] database idempotency strategy
[ ] external API idempotency strategy
[ ] reconciliation strategy

DLQ
[ ] DLQ defined
[ ] owner defined
[ ] alert configured
[ ] failure reason preserved
[ ] original metadata preserved
[ ] retention defined
[ ] access controlled
[ ] replay procedure documented

KAFKA
[ ] retry topics defined
[ ] partition strategy defined
[ ] ordering impact documented
[ ] offset behavior tested
[ ] consumer rebalance tested

RABBITMQ
[ ] ACK behavior tested
[ ] requeue behavior defined
[ ] retry queues defined
[ ] DLX configured
[ ] prefetch tuned

KUBERNETES
[ ] graceful shutdown
[ ] readiness behavior
[ ] liveness behavior
[ ] resource limits
[ ] deployment behavior tested
[ ] crash-loop scenario tested

OBSERVABILITY
[ ] retry metrics
[ ] DLQ metrics
[ ] lag metrics
[ ] structured logs
[ ] trace propagation
[ ] alerts
[ ] runbook

RECOVERY
[ ] root-cause workflow
[ ] sample replay tested
[ ] controlled replay
[ ] replay audit
[ ] downstream protection
[ ] DR implications tested
```

---

# 177. Final Production Mental Model

Think about message failure as a lifecycle:

```text
             +----------------+
             |    MESSAGE     |
             +--------+-------+
                      |
                      v
                 PROCESSING
                      |
          +-----------+-----------+
          |                       |
       SUCCESS                 FAILURE
          |                       |
          v                       v
      ACK/OFFSET             CLASSIFY ERROR
                                  |
                   +--------------+--------------+
                   |                             |
               TRANSIENT                      PERMANENT
                   |                             |
                   v                             v
              BACKOFF/JITTER                    DLQ
                   |
                   v
                RETRY
                   |
             +-----+-----+
             |           |
          SUCCESS      FAILURE
             |           |
             v           v
           DONE       NEXT RETRY
                         |
                    max attempts
                         |
                         v
                        DLQ
```

The objective is not to retry as many times as possible.

The objective is to **recover transient failures without creating duplicates, overload or hidden data loss, while making permanent failures visible and recoverable**.

---

# 178. Final Golden Rules

1. Never implement infinite retries.
2. Classify failures before retrying.
3. Retry transient failures.
4. Isolate permanent failures.
5. Always consider duplicate processing.
6. Use idempotency for important side effects.
7. Use exponential backoff where appropriate.
8. Add jitter to avoid synchronized retries.
9. Cap maximum retry delay.
10. Set maximum retry attempts.
11. Respect downstream rate limits.
12. Use Retry-After when provided.
13. Protect unhealthy dependencies.
14. Use circuit breakers when appropriate.
15. Avoid immediate RabbitMQ requeue loops.
16. Do not confuse Kubernetes restart with message retry.
17. Preserve event identity across retries.
18. Preserve original topic and partition metadata.
19. Preserve correlation and trace identifiers.
20. Record retry count.
21. Record failure reason.
22. Record first failure time.
23. Record last failure time.
24. Build an explicit DLQ path.
25. Assign DLQ ownership.
26. Alert on DLQ growth.
27. Monitor oldest DLQ message age.
28. Define DLQ retention.
29. Secure DLQ access.
30. Minimize sensitive data in failure records.
31. Never replay blindly.
32. Fix the root cause before replay.
33. Replay a small sample first.
34. Rate-limit large replays.
35. Audit replay operations.
36. Validate business state after replay.
37. Treat external APIs as separate transaction boundaries.
38. Use provider idempotency keys where supported.
39. Reconcile critical external operations.
40. Do not assume timeout means failure.
41. Do not assume ACK means business success unless designed that way.
42. Do not commit Kafka offsets before required processing.
43. Do not ACK RabbitMQ messages before required processing.
44. Understand consumer crash behavior.
45. Test duplicate delivery.
46. Test poison messages.
47. Test dependency outages.
48. Test schema failures.
49. Test credential failures.
50. Test rate limiting.
51. Test broker failures.
52. Test consumer restarts.
53. Test Kubernetes rolling deployments.
54. Test graceful shutdown.
55. Monitor consumer lag.
56. Monitor lag age.
57. Monitor retry rate.
58. Monitor processing latency.
59. Monitor downstream latency.
60. Monitor DLQ rate.
61. Monitor retry backlog.
62. Monitor retry age.
63. Do not let retry traffic starve healthy traffic.
64. Consider separate retry capacity.
65. Consider priority by business criticality.
66. Respect business deadlines.
67. Align retry policy with SLA.
68. Align retry policy with SLO.
69. Avoid layered uncontrolled retries.
70. Define retry ownership between application layers.
71. Standardize retry metadata.
72. Standardize retry configuration.
73. Provide reusable retry libraries where practical.
74. Document service-specific failure classification.
75. Make retry behavior observable.
76. Trace asynchronous retries.
77. Use structured logs.
78. Avoid sensitive payload logging.
79. Preserve ordering requirements explicitly.
80. Understand that retry can break ordering.
81. Choose blocking vs non-blocking retry deliberately.
82. Consider partition impact in Kafka.
83. Consider prefetch impact in RabbitMQ.
84. Do not assume more consumers solve every backlog.
85. Investigate the real bottleneck.
86. Protect databases from retry storms.
87. Protect APIs from retry storms.
88. Protect broker resources from retry storms.
89. Use load shedding when business policy permits.
90. Use bulkhead isolation when workloads compete.
91. Treat DLQ as failure isolation, not backup.
92. Treat retry as controlled recovery, not endless persistence.
93. Make replay an engineered capability.
94. Make incident recovery reproducible.
95. Test the failure path as seriously as the happy path.
96. Measure business impact, not only infrastructure health.
97. Design for duplicates.
98. Design for outages.
99. Design for recovery.
100. The goal of retry is safe recovery, not repeated execution.

---