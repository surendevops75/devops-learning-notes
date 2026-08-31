# Synchronous-vs-Asynchronous-Communication

## Purpose

Communication style is one of the most important architectural decisions in a distributed system.

A production engineer must know when to use:

```text
synchronous request/response
asynchronous messaging
events
queues
RPC
REST
gRPC
webhooks
streaming
```

The choice affects:

```text
latency
availability
coupling
scalability
failure handling
retries
ordering
observability
security
cost
operational complexity
```

The correct answer is rarely "always use synchronous" or "always use Kafka."

The communication model must match the business requirement.

---

# 1. Communication in a Distributed System

A distributed application normally contains multiple independently running
components.

```text
Client
 |
API
 |
+----------+----------+
|          |          |
User     Order     Catalog
Service  Service    Service
             |
          Database
```

The components need a communication mechanism.

The first architectural question is:

```text
Does the caller need the result immediately?
```

If yes, synchronous communication is often appropriate.

If no, asynchronous communication may provide better isolation and scalability.

---

# 2. Synchronous Communication

Synchronous communication means the caller sends a request and waits for a
response.

```text
Caller
 |
 | request
 v
Service
 |
 | response
 v
Caller
```

The caller's progress depends on the remote operation.

Typical examples:

```text
HTTP REST
gRPC
RPC
database calls
```

---

# 3. Asynchronous Communication

Asynchronous communication allows the producer to hand work to another system
without waiting for final processing.

```text
Producer
 |
 | message
 v
Broker
 |
 | later
 v
Consumer
```

Examples:

```text
RabbitMQ
Kafka
SQS
SNS
event buses
```

The producer and consumer are separated in time.

---

# 4. Temporal Coupling

Synchronous communication creates temporal coupling.

```text
A -> B
```

For A to complete the operation, B generally needs to be available now.

Asynchronous communication can reduce this:

```text
A -> Broker

B processes later
```

The producer can continue even when the consumer is temporarily unavailable,
assuming the broker remains available and capacity is sufficient.

---

# 5. Spatial Coupling

Spatial coupling means one component needs knowledge about where another
component lives.

Bad:

```text
Application -> hardcoded pod IP
```

Better:

```text
Application -> DNS/service name -> current instance
```

Service discovery reduces spatial coupling.

---

# 6. Synchronous Request Lifecycle

A typical HTTP request:

```text
Client
 |
DNS
 |
TCP
 |
TLS
 |
Load Balancer
 |
Service
 |
Dependency
 |
Database
 |
response
```

Every step can add:

```text
latency
failure probability
resource consumption
```

This is why a simple API call may actually involve a long distributed path.

---

# 7. Asynchronous Message Lifecycle

A typical message:

```text
Producer
 |
publish
 |
Broker
 |
store
 |
deliver
 |
Consumer
 |
process
 |
ack/commit
```

The lifecycle contains different failure points:

```text
publish failure
broker failure
delivery failure
consumer crash
processing failure
ack failure
```

Each requires a deliberate strategy.

---

# 8. When Synchronous Is Better

Synchronous communication is usually appropriate when:

```text
caller needs immediate answer
operation is short
result determines next action
strong request/response semantics matter
```

Examples:

```text
login
fetch account
check inventory
authorize payment
read configuration
```

---

# 9. When Asynchronous Is Better

Asynchronous communication is useful when:

```text
work can happen later
processing is expensive
bursts need buffering
consumer availability is variable
producer and consumer should scale independently
multiple consumers need an event
```

Examples:

```text
send email
generate report
process image
resize video
send notification
analytics processing
order fulfillment
```

---

# 10. Immediate vs Deferred Work

A useful decomposition:

```text
User request
 |
+--> immediate critical work
 |
+--> deferred noncritical work
```

Example:

```text
Checkout
 |
+--> authorize payment       SYNC
 |
+--> create fulfillment task ASYNC
 |
+--> send email              ASYNC
 |
+--> analytics event         ASYNC
```

This often provides a good balance.

---

# 11. Synchronous Does Not Mean Simple

A synchronous architecture can still contain many distributed calls.

```text
Client
 |
API
 |
Order
 |
Payment
 |
Inventory
 |
Database
```

The user waits for the entire chain.

A failure deep in the chain can affect the original request.

---

# 12. Asynchronous Does Not Mean Reliable Automatically

Adding a queue does not automatically solve reliability.

Potential problems:

```text
broker unavailable
message lost
duplicate delivery
consumer failure
poison message
queue growth
retention expiration
```

Messaging requires its own reliability architecture.

---

# 13. Latency Comparison

Synchronous:

```text
Client
 |
A -> B -> C
 |
response
```

Latency accumulates across the critical path.

Asynchronous:

```text
Client
 |
A -> Queue
 |
response

Queue -> B -> C later
```

The initial request can finish sooner.

But the business operation now completes over time.

---

# 14. User Experience

Synchronous is suitable when the user needs:

```text
immediate validation
immediate result
immediate authorization
```

Asynchronous is suitable when the user can see:

```text
accepted
processing
completed
```

For example:

```text
Report requested
 |
HTTP 202 Accepted
 |
job_id = 123
 |
poll/status/event
 |
completed
```

---

# 15. HTTP 202 Pattern

An API can accept work without completing it:

```text
POST /reports
 |
202 Accepted
 |
job_id
```

Worker:

```text
job_id
 |
process
 |
store result
```

Client:

```text
GET /reports/{job_id}
```

This is a common asynchronous API pattern.

---

# 16. Request/Response Coupling

Synchronous communication couples:

```text
caller availability
server availability
network availability
```

The caller cannot normally complete the operation without a response.

This can be acceptable when the dependency is critical and fast.

---

# 17. Asynchronous Coupling

Asynchronous systems still have coupling, but it moves into:

```text
message schema
delivery semantics
event meaning
consumer expectations
```

You reduce temporal coupling but may increase operational and data-model
complexity.

---

# 18. RPC

RPC means Remote Procedure Call.

Conceptually:

```text
Client
 |
call()
 |
Remote Service
 |
result
```

The programming model resembles a local function call.

But engineers must remember:

> A remote procedure is not a local procedure.

It can fail because of:

```text
network
server
timeout
serialization
authentication
```

---

# 19. REST

REST commonly uses HTTP resources:

```text
GET /orders/123
POST /orders
PUT /orders/123
DELETE /orders/123
```

Benefits:

```text
widely supported
simple debugging
HTTP ecosystem
browser/proxy compatibility
```

Costs can include:

```text
payload overhead
schema ambiguity without strong contracts
multiple round trips
```

---

# 20. gRPC

gRPC commonly uses:

```text
HTTP/2
Protocol Buffers
strong service contracts
```

Useful for:

```text
service-to-service communication
low-latency APIs
strongly typed contracts
streaming
```

Operational considerations include:

```text
load balancing
timeouts
TLS
proxies
observability
version compatibility
```

---

# 21. REST vs gRPC

Use REST when:

```text
public API
broad client compatibility
simple resource semantics
```

Use gRPC when:

```text
internal service-to-service
strong contracts
high-performance communication
streaming
```

Either can be production-grade.

---

# 22. Synchronous Database Calls

Database calls are synchronous from the application's perspective:

```text
Application
 |
query
 |
Database
 |
result
```

Protect with:

```text
timeouts
connection pools
query limits
circuit protection
```

A database should never be treated as infinitely available or infinitely fast.

---

# 23. Connection Pool Exhaustion

Suppose:

```text
application replicas = 20
connections per replica = 50
```

Potential database connections:

```text
20 × 50 = 1,000
```

If the database can safely support only 300 application connections, scaling
the application creates a database incident.

Connection capacity must be designed end-to-end.

---

# 24. Synchronous Fan-Out

```text
API
 |
+--> Service A
+--> Service B
+--> Service C
+--> Service D
```

If all are required, API completion depends on all required services.

Failure probability and tail latency increase as dependencies increase.

---

# 25. Partial Fan-Out

Not every dependency needs to block the request.

```text
API
 |
+--> Payment       REQUIRED
+--> Inventory     REQUIRED
+--> Recommendation OPTIONAL
```

If recommendation fails:

```text
checkout continues
```

This is graceful degradation.

---

# 26. Asynchronous Fan-Out

```text
OrderCreated
 |
Broker
 |
+--> Notification
+--> Analytics
+--> Search Index
+--> Recommendation
```

The producer does not need to call every consumer directly.

This is useful for loosely coupled side effects.

---

# 27. One-to-One Queue

```text
Producer -> Queue -> Worker
```

Typical use:

```text
background job
```

A message is generally intended to be processed by one worker.

---

# 28. One-to-Many Event

```text
Producer
 |
Event
 |
Broker
 +--> Consumer A
 +--> Consumer B
 +--> Consumer C
```

Each consumer can independently react.

Useful for:

```text
analytics
notifications
search
audit
```

---

# 29. Command Queue

A command says:

```text
Do this.
```

Example:

```text
GenerateInvoice
```

A queue can distribute commands to workers.

The producer expects the command to be acted upon.

---

# 30. Event Stream

An event says:

```text
This happened.
```

Example:

```text
InvoiceGenerated
```

Multiple consumers may independently process the event.

Event streams are useful for:

```text
audit
analytics
materialized views
integration
```

---

# 31. Queue vs Event Stream

Queue model:

```text
message -> worker -> removed/acknowledged
```

Event stream:

```text
event -> retained log
             |
       +-----+-----+
       |     |     |
      C1    C2    C3
```

The second model enables replay and independent consumer progress.

---

# 32. Synchronous Retry

If a synchronous call fails:

```text
A -> B
X
A -> B retry
```

The retry consumes caller resources and can amplify load.

Use:

```text
deadline
bounded attempts
backoff
jitter
```

---

# 33. Asynchronous Retry

A message can be retried without keeping the original request open.

```text
Queue
 |
Consumer
 |
failure
 |
retry delay
 |
consumer again
```

This is useful for temporary dependency failures.

---

# 34. Retry Ownership

Avoid:

```text
API retries
Service retries
Worker retries
Database driver retries
```

all independently.

This can multiply traffic.

Define which layer owns each retry decision.

---

# 35. Timeout Budgets

Suppose:

```text
overall deadline = 2s
```

Do not allow:

```text
Service A timeout = 5s
```

The downstream timeout should fit inside the remaining budget.

This is deadline propagation.

---

# 36. Timeout and Retry Interaction

Bad:

```text
timeout = 2s
retries = 5
```

Potential total wait:

```text
10s+
```

even though the user may only tolerate 2 seconds.

Design retries within the overall deadline.

---

# 37. Retry Storm

```text
Dependency failure
 |
1,000 callers
 |
5 retries each
 |
5,000 additional calls
```

The dependency can be pushed further into failure.

Use:

```text
exponential backoff
jitter
bounded retries
circuit breakers
load shedding
```

---

# 38. Backpressure

Asynchronous systems require flow control.

```text
Producer
 |
fast
 v
Queue
 |
slow
 v
Consumer
```

If the producer remains faster forever:

```text
queue grows
storage grows
latency grows
```

Backpressure must eventually reach producers.

---

# 39. Producer Throttling

Possible controls:

```text
rate limits
bounded concurrency
queue capacity limits
token buckets
load shedding
```

The goal is to prevent unbounded work accumulation.

---

# 40. Consumer Scaling

```text
Queue
 |
+-- Worker 1
+-- Worker 2
+-- Worker 3
```

Scale consumers when:

```text
backlog grows
oldest message age grows
processing capacity is insufficient
```

But check downstream capacity before scaling indefinitely.

---

# 41. Message Ordering

Synchronous request chains naturally impose ordering for that request.

Asynchronous systems may not.

Example:

```text
Event 1: Create
Event 2: Update
```

Consumer may receive:

```text
Update
Create
```

if the architecture does not guarantee ordering.

---

# 42. Per-Key Ordering

Often the requirement is:

```text
order events for order_id=123
```

not:

```text
order every event globally
```

Partitioning by business key can provide scalable per-key ordering.

---

# 43. Delivery Semantics

Asynchronous systems must define:

```text
at-most-once
at-least-once
exactly-once
```

In practice, at-least-once delivery plus idempotent consumers is a common
production pattern.

---

# 44. Idempotent Consumer

```text
Message
 |
message_id
 |
processed?
 +-- yes -> return/skip
 |
 no
 |
process
 |
record
```

This protects against duplicate delivery.

---

# 45. Transactional Side Effects

Consider:

```text
consume payment event
 |
charge card
 |
ack message
```

If the consumer crashes after charging but before ack:

```text
message redelivered
 |
card charged again
```

Therefore external side effects require idempotency or reconciliation.

---

# 46. Synchronous External APIs

External APIs introduce:

```text
unknown latency
rate limits
provider outages
schema changes
network failures
```

Use:

```text
timeouts
bounded retries
circuit breakers
rate limits
fallbacks
```

Never assume a third-party API behaves like your internal service.

---

# 47. Webhooks

Webhooks are asynchronous callbacks:

```text
Provider
 |
HTTP callback
 |
Your endpoint
```

Webhook consumers should:

```text
authenticate
validate
deduplicate
ack quickly
process asynchronously
```

A robust pattern:

```text
Webhook
 |
validate
 |
persist/enqueue
 |
202/200
 |
worker
```

---

# 48. Webhook Duplicate Delivery

Providers may retry if they do not receive an expected response.

Therefore:

```text
event_id
 |
deduplication
 |
process once
```

is important.

---

# 49. Webhook Security

Use:

```text
signature verification
TLS
timestamp validation
replay protection
allow-listing where appropriate
```

Do not trust an arbitrary HTTP request simply because it comes to a webhook
endpoint.

---

# 50. Polling

Polling means repeatedly asking:

```text
GET /status
```

Example:

```text
client
 |
poll
 |
server

wait
 |
poll
 |
server
```

Simple but can create unnecessary load.

---

# 51. Long Polling

The server holds the request until data is available or timeout occurs.

```text
Client -------- request --------> Server
Client <------- response -------- Server
```

Useful in some systems but consumes long-lived connections.

---

# 52. Server-Sent Events

SSE allows a server to stream events to a client over HTTP.

```text
Server
 |
event
 |
Client
 |
event
 |
Client
```

Useful for:

```text
live status
notifications
progress updates
```

It is primarily server-to-client streaming.

---

# 53. WebSockets

WebSockets provide bidirectional communication:

```text
Client <=================> Server
```

Useful for:

```text
chat
live dashboards
collaboration
real-time applications
```

Operational considerations:

```text
connection management
load balancing
timeouts
reconnection
authentication
backpressure
```

---

# 54. Streaming RPC

Streaming allows multiple messages over one connection.

```text
Client <===========> Server
   message 1
   message 2
   message 3
```

Useful for:

```text
telemetry
large result sets
real-time processing
```

---

# 55. Batch vs Individual Requests

Individual:

```text
1 request -> 1 item
```

Batch:

```text
1 request -> 100 items
```

Batching can reduce:

```text
network overhead
connection overhead
serialization overhead
```

But batches can increase:

```text
latency
failure scope
memory
```

Choose batch size deliberately.

---

# 56. Request Coalescing

If many callers request the same expensive operation:

```text
A -> expensive request
B -> same request
C -> same request
```

coalesce into:

```text
one operation
 |
shared result
```

Useful for cache misses and expensive reads.

---

# 57. Priority Queues

Not all work has equal importance.

```text
HIGH:
payments

MEDIUM:
orders

LOW:
analytics
```

Priority processing can protect critical workloads.

But starvation of low-priority work must be prevented.

---

# 58. Fairness

A shared queue can allow one workload to dominate.

Use:

```text
per-tenant limits
weighted queues
quotas
fair scheduling
```

This is especially important in multi-tenant systems.

---

# 59. Deadlines in Async Systems

Asynchronous does not mean "process whenever."

Messages can have:

```text
expiry
deadline
business validity window
```

Example:

```text
OTP request
 |
expires in 5 minutes
```

Processing an expired command may be incorrect.

---

# 60. Message TTL

A message may have a time-to-live.

After expiration:

```text
message
 |
TTL expired
 |
discard/DLQ
```

TTL policies must match business semantics.

---

# 61. Queue Capacity

A queue needs capacity planning.

Consider:

```text
message size
messages/sec
retention
peak duration
storage
replication factor
```

Example:

```text
10,000 msg/s
10 KB/message
```

Raw ingress:

```text
~100 MB/s
```

before replication and overhead.

---

# 62. Burst Absorption

Suppose:

```text
normal = 1,000 msg/s
peak = 10,000 msg/s
peak duration = 60s
```

Extra work:

```text
9,000 × 60
= 540,000 messages
```

The queue must accommodate the burst or consumers must scale fast enough.

---

# 63. Queue Age

Queue depth alone can be misleading.

More useful:

```text
oldest message age
```

A queue with 1 million tiny messages may be healthier than a queue with 10,000
messages that are already 30 minutes old, depending on the SLA.

---

# 64. Consumer Lag

For streaming systems:

```text
producer position
       |
       | lag
       v
consumer position
```

Lag indicates how far consumers are behind.

Monitor both:

```text
lag
lag growth rate
```

---

# 65. Message Size

Large messages increase:

```text
network usage
broker storage
serialization time
consumer memory
latency
```

Prefer references for large objects:

```text
Message
 |
object_id
 |
Object Storage
```

instead of embedding huge payloads.

---

# 66. Asynchronous Large File Processing

Better pattern:

```text
Client
 |
Object Storage
 |
event
 |
Queue
 |
Worker
 |
processed object
```

Do not push multi-hundred-MB files through a general-purpose message broker
unless the architecture explicitly supports it.

---

# 67. Synchronous Large Payloads

Large synchronous responses can cause:

```text
high memory
long connections
timeouts
proxy limits
```

Prefer:

```text
upload/download from object storage
```

and return a reference.

---

# 68. Communication Security

Synchronous:

```text
TLS
authentication
authorization
```

Asynchronous:

```text
TLS
producer authorization
consumer authorization
topic/queue permissions
encryption
```

Both models require identity.

---

# 69. Service-to-Service Authentication

Use:

```text
service identity
short-lived credentials
mTLS where appropriate
IAM/workload identity
```

Do not rely only on source IP.

---

# 70. Authorization

Authentication asks:

```text
Who are you?
```

Authorization asks:

```text
What are you allowed to do?
```

Example:

```text
Order Service -> publish OrderCreated
Notification -> consume OrderCreated
```

Permissions should be scoped accordingly.

---

# 71. Observability of Synchronous Calls

Track:

```text
request rate
latency
error rate
timeouts
retries
dependency latency
```

Use distributed tracing to identify which dependency caused the delay.

---

# 72. Observability of Asynchronous Calls

Track:

```text
publish rate
consume rate
queue depth
oldest age
consumer lag
retry count
DLQ count
processing latency
```

Also correlate:

```text
producer event ID
consumer processing
business transaction
```

---

# 73. Correlation IDs

Example:

```text
request_id=abc123
```

Flow:

```text
API
 |
Order
 |
Event
 |
Worker
 |
Notification
```

The same correlation information lets operators follow the business operation.

---

# 74. Business IDs vs Technical IDs

Use both where appropriate.

```text
order_id = ORD123
event_id = EVT456
trace_id = TRACE789
```

Business IDs identify domain objects.

Technical IDs identify messages and execution paths.

---

# 75. Synchronous Failure Strategy

When dependency fails:

```text
timeout
 |
fallback OR controlled error
```

Do not automatically retry every operation.

Ask:

```text
Is it safe?
Is it useful?
Does the deadline permit it?
Can the dependency handle it?
```

---

# 76. Asynchronous Failure Strategy

When consumer fails:

```text
message
 |
retry
 |
retry
 |
DLQ
```

The exact strategy depends on:

```text
transient vs permanent failure
ordering
message TTL
business importance
```

---

# 77. Transient vs Permanent Failure

Transient:

```text
database temporarily unavailable
network timeout
rate limit
```

Retry may help.

Permanent:

```text
invalid schema
missing required field
invalid business state
```

Retrying indefinitely will not help.

Route permanent failures to DLQ or another remediation path.

---

# 78. Poison Message Handling

```text
message
 |
consumer
 |
failure
 |
retry count
 |
threshold
 |
DLQ
```

Alert owners.

A DLQ without operational ownership is just hidden failure.

---

# 79. Backpressure in Synchronous Systems

Synchronous systems also need backpressure.

Controls:

```text
connection limits
thread pools
request queues
rate limiting
load shedding
```

Without limits, traffic can exhaust resources.

---

# 80. Bulkheads

Separate resource pools:

```text
Payment calls      -> pool A
Recommendations    -> pool B
Reports            -> pool C
```

A slow report dependency should not consume payment capacity.

---

# 81. Circuit Breakers

Circuit breaker states:

```text
CLOSED
 |
failure threshold
 v
OPEN
 |
cooldown
 v
HALF-OPEN
 |
success -> CLOSED
failure -> OPEN
```

Useful for synchronous dependencies.

They can also be useful around asynchronous external calls.

---

# 82. Load Shedding

When capacity is exhausted:

```text
reject optional requests
```

Examples:

```text
429
503
disable recommendations
defer analytics
```

Protect critical workflows.

---

# 83. Graceful Degradation

Example:

```text
Checkout
 |
+--> Payment     REQUIRED
+--> Inventory   REQUIRED
+--> Reviews     OPTIONAL
+--> Recommend   OPTIONAL
```

If optional services fail:

```text
checkout continues
```

This is a strong production design.

---

# 84. Availability Comparison

Synchronous dependency chain:

```text
A -> B -> C
```

If B is unavailable, A may fail.

Asynchronous:

```text
A -> Broker
       |
       v
       B
```

A may continue if the broker remains available.

But the business workflow becomes eventually processed.

---

# 85. Asynchronous Failure Window

Async systems can hide failure temporarily.

Example:

```text
Producer healthy
Broker healthy
Consumer down
```

Producer may continue successfully while backlog grows.

Therefore asynchronous systems require strong backlog monitoring.

---

# 86. The Delayed Failure Problem

A synchronous failure is immediate:

```text
request -> error
```

An asynchronous failure may appear later:

```text
request -> accepted

minutes later:
consumer -> failure
```

Users may not immediately know the final business outcome.

The product must communicate processing state when necessary.

---

# 87. Job Status Pattern

```text
POST /jobs
 |
202
 |
job_id
 |
GET /jobs/{id}
 |
PENDING
 |
RUNNING
 |
COMPLETED / FAILED
```

This is useful for long-running asynchronous operations.

---

# 88. Notification Pattern

```text
Job
 |
status change
 |
event
 |
notification service
 |
email/SMS/push
```

Notification should not block the critical job if notification is noncritical.

---

# 89. Synchronous Authentication + Async Processing

A common production design:

```text
Client
 |
API
 |
authenticate
 |
validate
 |
enqueue
 |
202 Accepted
```

The expensive work happens asynchronously.

Authentication remains synchronous because the caller needs an immediate
authorization decision.

---

# 90. Payment Architecture

Payments require careful semantics.

Possible:

```text
Order API
 |
Payment authorization SYNC
 |
Order state
 |
Fulfillment ASYNC
```

The payment operation must use:

```text
idempotency
reconciliation
timeouts
provider status lookup
```

Never assume a timeout means payment failed.

---

# 91. Inventory Architecture

Inventory may require strong control over stock.

Example:

```text
Checkout
 |
reserve inventory
 |
reservation event
 |
fulfillment
```

The reservation itself may be synchronous if the caller needs an immediate
answer.

Downstream fulfillment can be asynchronous.

---

# 92. Notification Architecture

Email/SMS is often asynchronous:

```text
Business Event
 |
Notification Queue
 |
Workers
 |
Provider
```

Benefits:

```text
provider outage does not block business workflow
retry is possible
provider rate limits can be handled
```

---

# 93. Search Indexing

Database transaction:

```text
DB write
 |
Outbox
 |
Event
 |
Search Indexer
 |
Search
```

Search can become eventually consistent with the database.

This is often acceptable.

---

# 94. Analytics

Do not block user requests on analytics processing.

Prefer:

```text
application
 |
event
 |
stream
 |
analytics consumers
```

This separates operational latency from analytical processing.

---

# 95. Audit Logging

Audit events often need durable asynchronous processing:

```text
business action
 |
audit event
 |
durable storage
```

For security-sensitive actions, ensure the audit path itself has appropriate
durability and access controls.

---

# 96. Synchronous vs Asynchronous Decision Matrix

| Requirement | Preferred model |
|---|---|
| Immediate response | Synchronous |
| Long-running work | Asynchronous |
| Burst buffering | Asynchronous |
| Strong immediate validation | Synchronous |
| Independent scaling | Asynchronous |
| Simple CRUD | Synchronous |
| Background email | Asynchronous |
| Search indexing | Asynchronous |
| User login | Synchronous |
| Analytics | Asynchronous |
| Payment authorization | Often synchronous |
| Fulfillment | Often asynchronous |
| Real-time bidirectional UI | WebSocket |
| Service-to-service typed RPC | gRPC |
| Public resource API | REST |

This is a starting point, not a rigid rule.

---

# 97. Hybrid Architecture

Most mature systems are hybrid.

```text
                 Client
                   |
                  API
                   |
          +--------+--------+
          |                 |
       Sync path        Async path
          |                 |
       Payment            Broker
          |             /   |   \
       Response       Worker A B  C
```

The synchronous path handles what the user must know immediately.

The asynchronous path handles work that can be deferred.

---

# 98. Anti-Pattern: Everything Synchronous

```text
Client
 |
A
 |
B
 |
C
 |
D
 |
E
```

Problems:

```text
high latency
high coupling
large failure surface
cascading failures
hard scaling
```

Use asynchronous boundaries where business semantics permit.

---

# 99. Anti-Pattern: Everything Asynchronous

Asynchronous communication is not automatically better.

Problems:

```text
eventual consistency
complex debugging
delayed errors
ordering complexity
duplicate processing
more infrastructure
```

If a simple immediate lookup is required, an async workflow may be unnecessary.

---

# 100. Anti-Pattern: Queue as Database

Do not use a queue as permanent business storage unless the technology and
retention model explicitly support that requirement.

A queue usually represents work.

A durable event log may represent historical facts.

Choose storage semantics intentionally.

---

# 101. Anti-Pattern: Infinite Retry

```text
failure
 |
retry forever
```

This can create:

```text
queue starvation
resource exhaustion
duplicate effects
```

Use bounded retries and DLQs.

---

# 102. Anti-Pattern: Blind Retry

Do not retry:

```text
validation error
authorization failure
invalid payload
```

Retry only errors that are plausibly transient and safe to repeat.

---

# 103. Anti-Pattern: Synchronous Side Effects

Example:

```text
Checkout
 |
send email
 |
call analytics
 |
update recommendation
 |
response
```

The user waits for unnecessary side effects.

Prefer:

```text
Checkout
 |
commit
 |
event
 |
side effects asynchronously
```

---

# 104. Anti-Pattern: Asynchronous Critical Decision

If the user needs to know:

```text
"Is my payment authorized?"
```

making that decision entirely asynchronous may produce poor UX unless the
business process explicitly supports pending states.

Communication style must follow business semantics.

---

# 105. Exactly-Once Myth

No architecture should casually claim:

```text
everything is exactly once
```

Instead ask:

```text
What can duplicate?
What effect must be idempotent?
What state is authoritative?
How is reconciliation performed?
```

---

# 106. Request Idempotency

For synchronous APIs:

```text
POST /payment
Idempotency-Key: abc123
```

The server stores the result associated with the key.

Retry:

```text
same key -> same logical operation
```

This is essential for uncertain network outcomes.

---

# 107. Event Idempotency

For asynchronous processing:

```text
event_id = EVT123
```

Consumer records processed state.

Duplicate:

```text
EVT123
EVT123
```

does not create duplicate business effects.

---

# 108. Ordering vs Parallelism

More parallelism:

```text
worker A
worker B
worker C
worker D
```

can improve throughput.

But strict ordering may limit concurrency.

The design must determine:

```text
where ordering is actually required
```

rather than imposing global ordering unnecessarily.

---

# 109. Communication and Data Consistency

Synchronous:

```text
write
 |
read latest result
```

can make strong consistency easier to present to clients.

Asynchronous:

```text
write
 |
event
 |
consumer
 |
eventual state
```

introduces a consistency window.

That window must be acceptable to the business.

---

# 110. Communication and Deployment

During rolling deployment:

```text
v1
v1
v2
v2
```

Both versions may process requests/events.

Therefore:

```text
API compatibility
event compatibility
schema compatibility
```

are mandatory.

---

# 111. Backward-Compatible APIs

Prefer:

```text
add optional field
```

over:

```text
remove required field
```

During migration:

```text
old client + new server
new client + old server
```

should behave safely where required.

---

# 112. Message Schema Compatibility

Consumers may lag behind producers.

Therefore new event versions should avoid breaking older consumers.

Use:

```text
optional fields
schema registry where appropriate
versioning
compatibility testing
```

---

# 113. Observability Difference

Synchronous:

```text
request -> response
```

is relatively easy to trace.

Asynchronous:

```text
request -> event -> queue -> consumer -> downstream
```

requires correlation across time.

Therefore asynchronous architecture requires stronger tracing and event
metadata.

---

# 114. Monitoring Synchronous Systems

Alert on:

```text
error rate
P95/P99 latency
timeouts
dependency failures
connection pool saturation
thread pool saturation
```

---

# 115. Monitoring Asynchronous Systems

Alert on:

```text
queue depth
oldest message age
consumer lag
retry rate
DLQ growth
processing latency
publish failures
consumer failures
```

A consumer being "Running" is not enough.

---

# 116. SLO for Async Work

An asynchronous SLO might be:

```text
99.9% of messages processed within 60 seconds
```

This is more meaningful than simply saying:

```text
consumer uptime = 99.99%
```

Measure business completion time.

---

# 117. Async Error Visibility

A successful enqueue does not mean successful business processing.

Track:

```text
accepted
processing
completed
failed
dead-lettered
```

Expose status to users when necessary.

---

# 118. Async Cancellation

Long-running jobs may need cancellation.

Use:

```text
job state
 |
CANCEL_REQUESTED
 |
worker checks
 |
CANCELLED
```

Do not assume killing the worker automatically creates a clean cancellation.

---

# 119. At-Least-Once and External Providers

Example:

```text
Worker -> email provider
```

Provider accepts the email.

Worker crashes before ack.

Message returns.

The worker sends the email again.

If duplicates are unacceptable, use:

```text
provider idempotency support
deduplication
business reconciliation
```

where available.

---

# 120. Communication Contracts

Every integration should define:

```text
schema
authentication
authorization
timeout
retry
rate limit
delivery semantics
ordering
failure behavior
observability
ownership
```

This becomes an operational contract.

---

# 121. Service-Level Contract Example

```text
Payment API

Timeout: 1.5s
Retry: limited, only safe transient errors
Auth: workload identity
Rate limit: provider-defined
Idempotency: required
Fallback: reconciliation
SLO: 99.95%
Owner: Payments Platform
```

A contract like this is much more useful than merely documenting an endpoint.

---

# 122. Capacity Planning

For synchronous systems:

```text
RPS
concurrency
connection pools
CPU
memory
network
```

For asynchronous systems:

```text
messages/sec
message size
retention
queue depth
consumer throughput
partition/queue capacity
storage
```

Both require end-to-end capacity analysis.

---

# 123. Example Capacity Calculation

Suppose:

```text
arrival = 5,000 msg/s
consumer throughput = 1,000 msg/s per worker
```

Minimum workers:

```text
5,000 / 1,000 = 5
```

But production should account for:

```text
failure
headroom
bursts
downstream capacity
deployments
```

Therefore running exactly five workers may be insufficient.

---

# 124. Queue Recovery Calculation

Suppose backlog:

```text
500,000 messages
```

Arrival:

```text
2,000/s
```

Processing capacity:

```text
3,000/s
```

Excess drain rate:

```text
1,000/s
```

Approximate recovery:

```text
500,000 / 1,000
= 500 seconds
≈ 8.3 minutes
```

This is the kind of reasoning required in production operations.

---

# 125. Synchronous Capacity Calculation

Suppose:

```text
2,000 RPS
P95 = 250ms
```

Approximate concurrency:

```text
2,000 × 0.25
= 500
```

Thread pools, connection pools and downstream capacity must accommodate the
actual concurrency model.

---

# 126. Communication and Kubernetes

Kubernetes supports both models:

Synchronous:

```text
Pod -> Service -> Pod
```

Asynchronous:

```text
Pod -> Broker Service
             |
             v
          Worker Pods
```

Kubernetes provides placement and service discovery, but application-level
communication semantics remain your responsibility.

---

# 127. Communication and EKS

An EKS architecture may look like:

```text
Internet
 |
ALB
 |
EKS Services
 |
+------ synchronous ------+
|                         |
Database               Redis

EKS Services
 |
Kafka/RabbitMQ
 |
Workers
```

Network policies, security groups, IAM and observability apply to both paths.

---

# 128. Communication and AWS

Common choices:

```text
REST/gRPC -> API Gateway/ALB/EKS
SQS -> asynchronous work
SNS/EventBridge -> events
MSK -> Kafka workloads
Amazon MQ/RabbitMQ -> messaging
```

Selection should follow semantics rather than service popularity.

---

# 129. Synchronous vs Asynchronous in RoboShop

Example:

```text
Frontend
 |
Cart
 |
Checkout
 |
Payment
```

Potential asynchronous boundaries:

```text
Checkout
 |
OrderCreated
 |
+--> Shipping
+--> Notification
+--> Analytics
```

This keeps secondary processing away from the critical request path.

---

# 130. Production Architecture Pattern

```text
                         CLIENT
                            |
                       DNS / WAF
                            |
                           ALB
                            |
                     API / EKS Services
                       /          \
                      /            \
               SYNC PATH         ASYNC PATH
                  |                  |
             critical APIs        Broker
                  |             /    |    \
              Database       Worker Worker Worker
                  |               |
                Cache          External APIs
```

Cross-cutting:

```text
IAM
TLS
Secrets
Metrics
Logs
Traces
Alerts
Backup
DR
```

---

# 131. Communication Decision Questions

Before selecting a protocol, ask:

```text
Does caller need immediate answer?
Can work be delayed?
Can the operation be retried?
Can it duplicate?
Is ordering required?
What is the maximum acceptable latency?
What happens if consumer is down?
What happens if provider is slow?
How much backlog is acceptable?
What is the consistency requirement?
```

---

# 132. Production Decision Tree

```text
Need immediate result?
 |
 +-- YES --> synchronous
 |
 +-- NO
      |
      Need durable deferred work?
      |
      +-- YES --> queue/message
      |
      Need multiple independent consumers/replay?
      |
      +-- YES --> event stream
```

Then evaluate:

```text
ordering
throughput
retention
failure
security
cost
```

---

# 133. Senior-Level Trade-Off

A good answer:

```text
"I would keep payment authorization synchronous because the caller needs
an immediate business decision. I would publish the resulting order event
asynchronously for fulfillment, notifications and analytics because those
operations do not need to block checkout.

This gives the critical workflow predictable latency while isolating
secondary workloads from provider failures."
```

This demonstrates architecture reasoning.

---

# 134. Common Interview Question

### Why not make everything asynchronous?

Answer:

```text
Because some decisions require an immediate response. Making those workflows
asynchronous adds eventual consistency and state-management complexity without
providing a business benefit.
```

---

# 135. Common Interview Question

### Why not make everything synchronous?

Answer:

```text
Because long-running or noncritical work would increase request latency and
couple the critical path to additional dependencies. Queues can provide
buffering, independent scaling and failure isolation.
```

---

# 136. Common Interview Question

### How do you prevent duplicate processing?

Answer:

```text
Use idempotency keys/message IDs, durable deduplication state, transactional
boundaries where appropriate, and reconciliation for external side effects.
```

---

# 137. Common Interview Question

### How do you handle consumer failure?

Answer:

```text
Use acknowledgements/offset management, bounded retries, exponential backoff,
DLQ handling, alerting and idempotent consumers. Monitor backlog age and
consumer lag.
```

---

# 138. Common Interview Question

### What happens if a synchronous dependency becomes slow?

Answer:

```text
Enforce deadlines, bound concurrency, use controlled retries only for safe
transient errors, activate circuit protection and degrade optional features.
```

---

# 139. Common Interview Question

### What happens if a message broker is unavailable?

Answer:

It depends on business criticality.

Possible options:

```text
fail request
buffer locally within strict limits
degrade optional processing
use alternate path
```

Do not blindly continue without understanding durability requirements.

---

# 140. Common Interview Question

### How do you choose RabbitMQ vs Kafka?

Answer:

```text
RabbitMQ is often attractive for routing and work-queue patterns.

Kafka is often attractive for high-throughput durable event streams, partitioned
processing, independent consumer groups and replay.

I would choose based on delivery semantics, retention, ordering, throughput,
routing and operational requirements.
```

---

# 141. Common Interview Question

### How do you guarantee ordering?

Answer:

```text
First identify the business key requiring ordering. Then partition or route by
that key and ensure the consumer processes that key sequentially. Avoid global
ordering unless it is truly required.
```

---

# 142. Common Interview Question

### How do you design retries?

Answer:

```text
Classify failures as transient or permanent. Retry only safe transient
failures, use bounded attempts, exponential backoff and jitter, and send
persistent failures to a DLQ or remediation workflow.
```

---

# 143. Common Interview Question

### What is the biggest risk of asynchronous architecture?

Answer:

There is no single answer, but common risks are:

```text
eventual consistency
duplicate processing
delayed failures
ordering
replay safety
operational visibility
backlog growth
```

The architecture must explicitly address them.

---

# 144. Common Interview Question

### What is the biggest risk of synchronous architecture?

Common risks:

```text
tight coupling
latency propagation
cascading failures
resource exhaustion
dependency availability
retry storms
```

---

# 145. Production Golden Rules

```text
1. Use synchronous communication when the caller needs an immediate result.
2. Use asynchronous communication when work can safely be deferred.
3. Treat every remote call as failure-prone.
4. Use deadlines.
5. Bound concurrency.
6. Retry only safe transient failures.
7. Use backoff and jitter.
8. Design consumers for duplicates.
9. Do not assume timeout means operation failed.
10. Define ordering explicitly.
11. Monitor queue age and consumer lag.
12. Do not let queues grow without bounds.
13. Protect critical paths from optional dependencies.
14. Use graceful degradation.
15. Keep event schemas compatible.
16. Use correlation IDs.
17. Make external side effects idempotent where possible.
18. Test failure and recovery.
19. Design security for both request and message paths.
20. Choose technology based on semantics, not fashion.
```

---

# 146. Final Mental Model

Use this:

```text
                 BUSINESS REQUIREMENT
                         |
                         v
                Need result now?
                  /          \
                YES           NO
                 |             |
             SYNC          ASYNC
                 |             |
              REST/gRPC     Queue/Event
                 |             |
           immediate       durable work
             result        /     |     \
                          C1     C2     C3
```

Then evaluate:

```text
latency
availability
consistency
ordering
duplicates
retry
backpressure
observability
security
scaling
DR
cost
```

The communication mechanism is not the architecture by itself.

The architecture is the combination of:

```text
business semantics
+
communication model
+
failure behavior
+
data model
+
operational controls
```

---