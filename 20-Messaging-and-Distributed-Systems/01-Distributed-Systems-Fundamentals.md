# 20-Messaging-and-Distributed-Systems
# 01-Distributed-Systems-Fundamentals

## Purpose

Distributed systems are the foundation of modern production platforms. This chapter builds the reasoning required to design and operate systems where independent processes, containers, nodes, availability zones, regions, databases and services communicate over networks.

The key mental model is:

```text
remote call != local function call
healthy component != healthy system
timeout != proof of failure
replication != backup
queue != infinite capacity
retry != free
availability != correctness
```

## 1. What Is a Distributed System?

A distributed system is a collection of independent computing components that communicate over a network to achieve a common purpose.

```text
Client
 |
API
 |
+----------+----------+
|          |          |
User     Order      Search
Service  Service    Service
            |
          Database
            |
          Broker
            |
         Workers
```

Every component and every network path can fail independently.

Therefore distributed systems must explicitly handle:

```text
latency
packet loss
timeouts
partial failure
duplication
ordering
concurrency
consistency
replication
coordination
scaling
security
recovery
```

## 2. Local vs Remote Calls

A local call:

```text
A -> function -> result
```

A remote call:

```text
A
 |
DNS
 |
TCP
 |
TLS
 |
Load Balancer
 |
B
 |
response
```

Possible failures include:

```text
DNS failure
connection failure
TLS failure
server overload
packet loss
timeout
response loss
proxy failure
```

Treat every remote call as failure-prone.

## 3. Partial Failure

Partial failure means some components work while another component does not.

```text
API --------> User Service      HEALTHY
 |
 +---------> Order Service     DOWN
```

The API may still pass its own health check while the business workflow is broken.

Important principle:

> Local health does not imply end-to-end health.

## 4. Failure Domains

Think in layers:

```text
process
pod
node
availability zone
region
account
```

Also consider dependency domains:

```text
DNS
load balancer
database
cache
broker
identity
registry
external API
observability
```

For every critical component ask:

```text
What fails?
Who depends on it?
What is the blast radius?
Can the system degrade?
How does it recover?
```

## 5. Fault, Failure and Recovery

A fault is an underlying problem.

A failure is an externally observable incorrect behavior.

Example:

```text
disk latency
 |
database latency
 |
service timeout
 |
API 5xx
 |
customer impact
```

A resilient architecture attempts to contain the fault before it becomes a
customer-visible failure.

## 6. Failure Detection

A timeout tells you:

```text
expected response did not arrive before deadline
```

It does NOT necessarily prove:

```text
remote service crashed
```

The remote service could be:

```text
slow
overloaded
partitioned
processing successfully but response was lost
```

This uncertainty is central to distributed systems.

## 7. Timeouts

Never allow important remote calls to wait forever.

Bad:

```text
call dependency
wait forever
```

Better:

```text
request
 |
deadline
 |
success OR controlled failure
```

Too-short timeouts cause false failures and retries.

Too-long timeouts cause resource exhaustion and cascading failures.

## 8. Deadline Propagation

Suppose the caller has a 2-second budget:

```text
Client
 |
API
 |
Service A
 |
Service B
 |
Database
```

The downstream calls should operate inside the remaining budget rather than
each receiving an independent unlimited timeout.

This prevents work from continuing after the caller has already abandoned it.

## 9. Latency and Tail Latency

Measure:

```text
P50
P90
P95
P99
P99.9
```

Average latency can hide severe tail latency.

Distributed fan-out amplifies tail latency because a request may wait for the
slowest required dependency.

## 10. Fan-Out and Fan-In

Fan-out:

```text
API
 +--> A
 +--> B
 +--> C
 +--> D
```

Fan-in:

```text
A B  C ---> consumer
D  /
```

Fan-out increases dependency exposure.

Fan-in can create a downstream bottleneck.

## 11. Throughput and Concurrency

Throughput examples:

```text
requests/sec
messages/sec
transactions/sec
MB/sec
```

Concurrency is work in progress.

A useful approximation is:

```text
concurrency ≈ throughput × latency
```

Example:

```text
1,000 requests/sec × 0.2 sec
≈ 200 concurrent requests
```

## 12. Backpressure

Backpressure prevents producers from overwhelming consumers.

```text
Producer
 |
Queue
 |
Consumer
```

A queue absorbs bursts but cannot create infinite capacity.

If:

```text
arrival rate > processing rate
```

backlog grows.

Therefore monitor:

```text
queue depth
oldest message age
processing rate
consumer errors
```

## 13. Synchronous Communication

```text
A -> request -> B
A <- response <- B
```

Advantages:

```text
simple
immediate result
easy request/response semantics
```

Disadvantages:

```text
tight temporal coupling
latency propagation
dependency availability
retry complexity
```

## 14. Asynchronous Communication

```text
Producer
 |
Broker
 |
Consumer
```

Advantages:

```text
decoupling
buffering
independent scaling
retry
burst absorption
```

Costs:

```text
eventual processing
duplicates
ordering complexity
replay complexity
observability complexity
```

## 15. Choosing Communication Style

Use synchronous calls when:

```text
caller needs immediate result
operation is short
dependency availability is acceptable
```

Use asynchronous messaging when:

```text
work can happen later
burst buffering is useful
processing is expensive
independent scaling is useful
```

Production systems commonly use both.

Example:

```text
Checkout
 |
synchronous payment authorization
 |
asynchronous fulfillment
```

## 16. Coupling

Types include:

```text
temporal coupling
spatial coupling
data coupling
behavioral coupling
```

Asynchronous messaging can reduce temporal coupling.

Service discovery reduces spatial coupling.

Stable contracts reduce data and behavioral coupling.

## 17. Availability

Availability describes whether a service is usable.

Approximate annual downtime:

```text
99%      ~3.65 days
99.9%    ~8.76 hours
99.99%   ~52.6 minutes
99.999%  ~5.26 minutes
```

Higher targets require stronger architecture and operations.

## 18. End-to-End Availability

If:

```text
API -> Database
```

and the API cannot function without the database, the database becomes part of
the availability path.

Component availability cannot be considered independently from the business
workflow.

## 19. Reliability and Durability

Reliability includes:

```text
correctness
availability
durability
recoverability
```

Durability asks whether committed data survives defined failures.

Availability and durability are different properties.

## 20. Consistency

Consistency concerns what values clients observe.

Example:

```text
write balance = 100
read balance = 80
```

Whether this is acceptable depends on business requirements.

Financial state may require stronger consistency than search indexing.

## 21. Strong Consistency

Strong consistency aims for reads to observe the latest committed state under
the system's defined model.

Useful for:

```text
financial state
inventory constraints
critical transactional decisions
```

It can require additional coordination and latency.

## 22. Eventual Consistency

Temporary divergence is permitted:

```text
Region A -> value 101
Region B -> value 100

replication

Region B -> value 101
```

Useful for:

```text
catalog
search
analytics
recommendations
some caches
```

The business must tolerate stale data.

## 23. CAP Theorem

CAP describes behavior during network partitions:

```text
C = consistency
A = availability
P = partition tolerance
```

The practical lesson is:

> Network partitions must be considered, and behavior during partitions must
> be explicitly designed.

Do not reduce CAP to a simplistic slogan about choosing any two properties.

## 24. Network Partition

```text
Node A     X     Node B
```

Both nodes may still be alive.

Questions become:

```text
Who accepts writes?
Should reads continue?
How is conflict handled?
Who is leader?
How is recovery performed?
```

## 25. Split Brain

Split brain occurs when multiple nodes believe they are authoritative.

```text
Node A: "I am leader"
Node B: "I am leader"
```

Potential consequences:

```text
conflicting writes
duplicate processing
corruption
```

Use quorum, leader election and fencing where appropriate.

## 26. Quorum

For:

```text
N = 5
```

a majority quorum is:

```text
floor(5/2)+1 = 3
```

The objective is preventing two independent groups from simultaneously
making conflicting decisions.

## 27. Leader Election

A system may select one leader:

```text
        Leader
       /            B        C
```

The design must handle:

```text
leader crash
network partition
slow leader
stale leader
re-election
```

## 28. Fencing

Fencing prevents a stale leader from continuing to modify shared state.

```text
Old leader -> fenced -> cannot write

New leader -> authorized -> write
```

This is particularly important for stateful distributed systems.

## 29. Replication

Replication maintains multiple copies:

```text
Primary
 /    R1     R2
```

Benefits:

```text
availability
durability
read scaling
DR
```

Costs:

```text
replication traffic
lag
storage
consistency complexity
failover complexity
```

## 30. Synchronous vs Asynchronous Replication

Synchronous replication can improve durability but increases coordination and
write latency.

Asynchronous replication can reduce write latency but introduces lag and
potential RPO exposure.

Therefore:

```text
replication strategy <-> RPO <-> latency
```

must be designed together.

## 31. Replication Lag

If:

```text
primary commit = T0
replica apply = T0 + 5s
```

then approximately:

```text
lag = 5 seconds
```

Consequences:

```text
stale reads
potential failover data loss
```

Monitor lag continuously.

## 32. Partitioning and Sharding

Partitioning distributes data:

```text
Partition 0
Partition 1
Partition 2
Partition 3
```

A shard key should distribute load.

Bad keys create hotspots.

## 33. Hot Partitions

If:

```text
99% traffic -> partition 0
```

adding more partitions does not automatically solve the problem.

Solutions may include:

```text
better key
key salting
workload redesign
traffic distribution
```

## 34. Consistent Hashing

A hash ring can distribute keys while limiting movement when nodes change.

Useful for:

```text
distributed caches
some partitioned systems
```

It introduces its own operational considerations around rebalancing and
hotspots.

## 35. Idempotency

An idempotent operation can safely be repeated without creating additional
business effects.

Example:

```text
request_id = abc123

first -> process
retry -> return existing result
```

This is essential when response delivery is uncertain.

## 36. The Uncertain Outcome Problem

Consider:

```text
Client -> Payment Provider
          |
          | payment succeeds
          |
          X response lost
```

The client does not know whether payment succeeded.

Correct response:

```text
reconcile
or
retry idempotently
```

Do not simply assume failure.

## 37. Delivery Semantics

Common models:

```text
at-most-once
at-least-once
exactly-once
```

At-most-once may lose messages.

At-least-once may duplicate messages.

"Exactly once" must be evaluated in the context of the entire business effect,
not just broker delivery.

## 38. At-Least-Once Processing

```text
consume
 |
process
 |
ack
```

If the consumer crashes after processing but before acknowledgement:

```text
process
 |
CRASH
 |
redelivery
```

Therefore consumers must tolerate duplicates.

## 39. Ordering

Business workflows may require:

```text
CREATE
 |
UPDATE
 |
CLOSE
```

Out-of-order events can produce invalid state.

Global ordering is expensive.

Per-key ordering is often a better design:

```text
customer A -> ordered
customer B -> ordered
customer C -> ordered
```

## 40. Duplicate Messages

Duplicates can result from:

```text
producer retry
consumer crash
network uncertainty
broker redelivery
```

Treat duplicates as a normal condition in at-least-once architectures.

## 41. Deduplication

Conceptual pattern:

```text
message_id
 |
processed?
 +---- yes -> skip
 |
 no
 |
process
 |
record ID
```

The deduplication record must itself have appropriate durability and
transactional guarantees.

## 42. Poison Messages

A poison message repeatedly fails.

Bad:

```text
consume -> fail -> retry -> fail -> retry forever
```

Better:

```text
bounded retry
 |
DLQ
 |
investigate
 |
fix
 |
replay
```

## 43. Dead-Letter Queues

A DLQ should have:

```text
owner
retention
alerting
inspection
replay process
```

A DLQ is not a garbage bin.

## 44. Retry Storms

If 100 clients each retry 5 times:

```text
100 initial
+
500 retries
```

An already unhealthy dependency receives even more traffic.

Use:

```text
bounded retries
exponential backoff
jitter
deadlines
circuit breakers
bulkheads
```

## 45. Exponential Backoff

Example:

```text
1s
2s
4s
8s
16s
```

This spreads retries over time.

## 46. Jitter

Without jitter:

```text
many clients
 |
retry at same instant
 |
traffic spike
```

With jitter:

```text
retry over a time window
 |
load distributed
```

## 47. Circuit Breaker

Typical states:

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

Circuit breakers can protect callers and dependencies.

## 48. Bulkheads

Separate resources by workload:

```text
Service
 |
+-- payment pool
+-- reporting pool
+-- recommendation pool
```

A slow optional workload should not consume all resources required by critical
operations.

## 49. Connection Pools

Unbounded connections can create:

```text
traffic spike
 |
connection explosion
 |
database overload
 |
timeouts
 |
retries
 |
more load
```

Connection limits are a reliability control.

## 50. Distributed Transactions

A transaction spanning:

```text
Order
Payment
Inventory
Shipping
```

cannot be treated like one local database transaction without additional
coordination.

Common patterns:

```text
Saga
outbox
compensation
workflow orchestration
```

## 51. Saga Pattern

A saga uses local transactions:

```text
Create Order
 |
Reserve Inventory
 |
Authorize Payment
 |
Create Shipment
```

If payment fails:

```text
release inventory
 |
cancel order
```

Compensating actions restore business consistency.

## 52. Orchestration vs Choreography

Orchestration:

```text
Workflow
 |
+-> Order
+-> Inventory
+-> Payment
```

Choreography:

```text
OrderCreated
 |
InventoryReserved
 |
PaymentAuthorized
 |
ShipmentCreated
```

Orchestration offers centralized visibility.

Choreography can reduce central coordination but may make workflows harder to
understand as systems grow.

## 53. Transactional Outbox

Problem:

```text
database transaction succeeds
message publication fails
```

Outbox:

```text
DB transaction
 |
+-- business record
+-- outbox record
 |
publisher
 |
broker
```

Business state and the intent to publish are committed together.

## 54. Change Data Capture

Concept:

```text
Database
 |
transaction log
 |
CDC
 |
broker
 |
consumers
```

Useful for:

```text
search indexing
analytics
event propagation
cache updates
```

## 55. Event-Driven Architecture

Events represent facts:

```text
OrderCreated
PaymentAuthorized
InventoryReserved
```

Benefits:

```text
decoupling
independent consumers
replay
asynchronous processing
```

Costs:

```text
schema evolution
ordering
duplicates
debugging
eventual consistency
```

## 56. Command vs Event

Command:

```text
"Do this."
```

Event:

```text
"This happened."
```

Example:

```text
ReserveInventory(order)
```

versus:

```text
InventoryReserved(order)
```

This distinction clarifies ownership and workflow design.

## 57. Schema Evolution

Events can outlive producer releases.

Prefer compatibility:

```text
add optional field
 |
deploy compatible consumers
 |
start producing field
```

Avoid breaking existing consumers during mixed-version operation.

## 58. Distributed Clocks

Machines do not have perfectly synchronized clocks.

Problems include:

```text
clock drift
network delay
NTP changes
different timestamps
```

Do not use wall-clock timestamps as the only source of causal ordering.

Use:

```text
sequence numbers
offsets
logical clocks
```

where appropriate.

## 59. Logical Clocks

Logical clocks represent ordering rather than physical time.

Examples:

```text
Lamport clocks
vector clocks
sequence numbers
```

The practical lesson is that timestamps alone may not prove causality.

## 60. Consensus

Consensus lets distributed nodes agree despite failures.

Core concepts:

```text
leader
quorum
term/epoch
log
commit
membership
```

Algorithms such as Raft and Paxos implement consensus approaches.

DevOps engineers should understand their operational effects even when a managed
service hides the implementation.

## 61. Service Discovery

Services need stable ways to find one another.

Options:

```text
DNS
service registry
Kubernetes Service
load balancer
```

Avoid coupling applications directly to ephemeral pod IPs.

## 62. Load Balancing

```text
Load Balancer
 /    |    A     B     C
```

Strategies include:

```text
round robin
least connections
weighted
hash-based
```

Health checks determine eligible targets.

## 63. Service Mesh

A service mesh may provide:

```text
mTLS
traffic routing
retries
timeouts
telemetry
authorization
```

It also adds operational complexity.

Use it when the value exceeds the additional failure and debugging surface.

## 64. Distributed Cache

```text
Application
 |
Cache
 |
Database
```

Problems:

```text
stale data
cache stampede
hot keys
eviction
cache outage
```

Design a safe fallback.

## 65. Cache Stampede

```text
popular key expires
 |
1,000 requests miss
 |
1,000 database requests
```

Controls:

```text
request coalescing
TTL jitter
stale-while-revalidate
prewarming
rate limiting
```

## 66. Distributed Locks

A distributed lock can provide exclusive ownership:

```text
Worker A -> lock -> allowed
Worker B -> lock -> denied
```

Failure cases:

```text
holder crash
partition
lease expiry
stale owner
```

Use established coordination mechanisms and fencing for critical state.

## 67. Leases

A lease grants ownership for a limited time.

```text
worker
 |
lease 30s
 |
renew
```

If the worker disappears, the lease eventually expires.

Carefully handle stale holders and renewal failures.

## 68. Heartbeats

```text
Node -> heartbeat -> coordinator
```

Missing heartbeat means:

```text
"communication was not observed"
```

not necessarily:

```text
"node definitely crashed"
```

Network congestion can produce false suspicions.

## 69. Gossip

Gossip spreads membership/state through peers.

```text
A <-> B
|\   /|
| \ / |
C <-> D
```

It can scale well but information propagates rather than appearing instantly
everywhere.

## 70. Distributed Rate Limiting

Local:

```text
each pod -> own limit
```

Shared:

```text
all pods -> shared state -> limit
```

Shared limits introduce consistency, latency and failure considerations.

## 71. Work Queues

```text
Producer
 |
Queue
 |
+---+---+
|   |   |
W1  W2  W3
```

Useful for:

```text
background processing
burst absorption
parallel work
```

Monitor queue depth and oldest-message age, not only worker CPU.

## 72. Consumer Scaling

Increasing workers helps until another bottleneck appears:

```text
database
network
partition count
downstream API
CPU
```

Always evaluate downstream capacity before unlimited scaling.

## 73. Load Shedding

When capacity is exhausted, reject lower-priority work:

```text
429
503
disable optional feature
defer background work
```

Protecting critical workflows is better than allowing the entire system to
collapse.

## 74. Graceful Degradation

Example:

```text
Checkout -> available
Recommendations -> unavailable
```

Critical workflows should not unnecessarily depend on optional features.

## 75. Dependency Classification

Classify dependencies:

```text
critical
important
optional
```

For each define:

```text
timeout
retry
fallback
owner
failure impact
```

## 76. Cascading Failure

Classic chain:

```text
dependency slows
 |
requests wait
 |
threads exhausted
 |
queue grows
 |
timeouts
 |
retries
 |
more traffic
 |
system collapse
```

Prevent with:

```text
timeouts
bounded concurrency
backpressure
bulkheads
circuit breakers
load shedding
```

## 77. Observability

Distributed debugging needs:

```text
metrics
logs
traces
correlation IDs
deployment markers
business metrics
```

The key question is:

```text
Where did the request go?
Where did it slow down?
Which dependency failed?
Which version handled it?
Who was affected?
```

## 78. Correlation IDs

```text
request_id=abc123

API -> abc123
Order -> abc123
Payment -> abc123
```

This connects activity across services.

## 79. Distributed Tracing

```text
Trace
 |
+-- API
    |
    +-- Order
        |
        +-- Payment
        +-- Database
```

Tracing helps identify dependency latency, fan-out and failures.

## 80. Distributed Metrics

Important metrics:

```text
traffic
errors
latency
saturation
queue depth
consumer lag
retry count
timeouts
connection usage
replication lag
```

The four golden signals:

```text
latency
traffic
errors
saturation
```

## 81. Distributed Logging

Use structured logs containing:

```text
timestamp
service
environment
severity
request ID
trace ID
message
```

Never put passwords, tokens or credentials into logs.

## 82. Distributed Security

Security boundaries include:

```text
identity
authentication
authorization
network
encryption
secrets
artifact integrity
audit
```

Network reachability must not automatically imply trust.

## 83. Zero Trust

Service A being inside the same VPC as Service B should not automatically
grant access.

Use:

```text
identity
authentication
authorization
least privilege
```

## 84. Configuration Changes

Central configuration can have a large blast radius.

Use:

```text
versioning
validation
review
staged rollout
audit
rollback
```

## 85. Feature Flags

Feature flags separate deployment from activation:

```text
deploy
 |
flag OFF
 |
validate
 |
flag ON
```

Useful for:

```text
canary
rollback
gradual release
experimentation
```

## 86. Mixed Versions

During deployments:

```text
v1 pods
v2 pods
```

Both may receive traffic.

Therefore:

```text
API contracts
events
database schemas
```

should remain compatible during the transition.

## 87. Expand-and-Contract

For database schema changes:

```text
expand
 |
compatible application
 |
backfill
 |
switch
 |
contract
```

Avoid destructive schema changes before old versions stop using them.

## 88. Graceful Shutdown

A distributed worker should:

```text
stop accepting new work
 |
finish current work
 |
commit/ack safely
 |
close connections
 |
exit
```

This reduces message loss and duplicates.

## 89. Control Plane vs Data Plane

Control plane:

```text
configuration
scheduling
orchestration
management
```

Data plane:

```text
actual customer traffic
```

A control-plane failure does not necessarily mean existing data-plane traffic
immediately stops.

## 90. Blast Radius

Reduce blast radius through:

```text
accounts
clusters
AZs
namespaces
tenant isolation
rate limits
bulkheads
least privilege
progressive delivery
```

Shared components require stronger safeguards.

## 91. Availability vs Degradation

A service does not always have only:

```text
UP
DOWN
```

Design states such as:

```text
FULL
DEGRADED
READ-ONLY
QUEUED
RECOVERING
```

This can preserve critical workflows during dependency failures.

## 92. Scalability

Vertical:

```text
bigger machine
```

Horizontal:

```text
more machines
```

Distributed systems often use horizontal scaling, but stateful components may
need partitioning, replication or specialized scaling.

## 93. Elasticity

Autoscaling changes capacity based on load.

But scaling has delay.

If traffic rises in seconds and capacity takes minutes to appear, use:

```text
headroom
queues
rate limiting
load shedding
```

## 94. Capacity Headroom

Do not operate critical infrastructure at permanent maximum utilization.

Reserve capacity for:

```text
traffic spikes
node failures
deployments
AZ loss
background recovery
```

## 95. Queue-Based Load Leveling

```text
Burst
 |
Queue
 |
steady workers
```

Useful for:

```text
notifications
media processing
analytics
batch workloads
```

## 96. Data Ownership

Explicit ownership reduces hidden coupling.

Prefer:

```text
Order Service -> owns order data
Payment Service -> owns payment data
Inventory -> owns inventory data
```

Integration occurs through contracts or events.

## 97. Distributed Transactions vs Local Transactions

Prefer local transactions plus asynchronous coordination where business
semantics allow it.

Avoid distributed transactions merely because they appear simpler initially.

## 98. Two-Phase Commit

Concept:

```text
Coordinator
 |
prepare
 |
participants
 |
commit
```

It can provide strong coordination but introduces blocking and availability
complexity.

Many microservice systems use sagas instead.

## 99. Reconciliation

Reconciliation compares desired and actual state:

```text
desired
 |
compare
 |
actual
 |
repair
```

This is central to:

```text
Kubernetes
GitOps
cloud controllers
payment reconciliation
inventory systems
```

## 100. Eventual Convergence

A system may temporarily diverge but repeatedly reconcile until:

```text
desired state == actual state
```

Convergence is a powerful model for distributed control systems.

## 101. State Machines

Business workflows can be modeled as:

```text
CREATED
 |
PAYMENT_PENDING
 |
PAID
 |
SHIPPED
 |
COMPLETED
```

Reject invalid transitions.

This makes retries and duplicate events easier to reason about.

## 102. Testing Distributed Systems

Test:

```text
latency
packet loss
dependency outage
node failure
AZ failure
broker failure
database failure
duplicates
out-of-order messages
partial deployments
```

Success-path testing alone is insufficient.

## 103. Chaos Engineering

Controlled failure injection can test assumptions:

```text
kill pod
 |
kill node
 |
add dependency latency
 |
block network path
```

Measure whether the system behaves as designed.

## 104. Game Days

A game day exercises:

```text
detection
communication
containment
failover
recovery
```

Measure:

```text
MTTD
MTTR
time to containment
manual steps
unexpected dependencies
```

## 105. Failure Injection Safety

Define:

```text
scope
owner
abort condition
monitoring
rollback
communication
```

Never perform uncontrolled failure experiments.

## 106. Messaging Mental Models

RabbitMQ:

```text
Producer
 |
Exchange
 |
Binding
 |
Queue
 |
Consumer
```

Kafka:

```text
Producer
 |
Topic
 |
Partitions
 |
Consumer Groups
 |
Consumers
```

These systems solve overlapping but different messaging problems.

## 107. RabbitMQ vs Kafka

Do not simply say:

```text
RabbitMQ = queue
Kafka = queue
```

Consider:

```text
routing
retention
replay
partitioning
ordering
throughput
consumer model
delivery semantics
```

RabbitMQ commonly emphasizes message routing and work distribution.

Kafka commonly emphasizes durable partitioned event streams and replay.

## 108. Event Replay

Durable event streams allow multiple independent consumers:

```text
event log
 |
+--> Consumer A
+--> Consumer B
+--> Consumer C
```

A new consumer may process historical events if retention allows.

Replay requires idempotent business effects.

## 109. Production Readiness Checklist

```text
[ ] requirements
[ ] scale assumptions
[ ] SLO
[ ] RTO
[ ] RPO
[ ] failure domains
[ ] dependency map
[ ] timeout policy
[ ] retry policy
[ ] idempotency
[ ] ordering
[ ] backpressure
[ ] observability
[ ] security
[ ] capacity
[ ] deployment
[ ] rollback
[ ] backup
[ ] restore
[ ] DR
[ ] ownership
[ ] cost
```

## 110. Senior Troubleshooting Framework

During an incident:

```text
1. Confirm customer impact
2. Establish timeline
3. Identify scope
4. Check recent changes
5. Trace request path
6. Check dependencies
7. Check saturation
8. Check network
9. Check data consistency
10. Contain
11. Mitigate
12. Validate recovery
13. Investigate root cause
14. Prevent recurrence
```

Avoid random restarts. Use evidence.

## 111. Example: API Latency Incident

Symptom:

```text
P95 = 200ms
P95 after incident = 5s
```

Investigation:

```text
API CPU -> normal
API memory -> normal
Database -> normal
Redis -> normal
Payment Service -> P99 = 4.5s
```

Likely bottleneck:

```text
Payment Service
```

Correct response:

```text
identify dependency degradation
 |
protect API with deadline
 |
use safe fallback if possible
 |
control retries
 |
restore dependency
 |
validate customer latency
```

Adding API replicas alone does not fix a downstream bottleneck.

## 112. Example: Payment Response Lost

```text
Client -> Payment
          |
          | SUCCESS
          |
          X response lost
```

The client should use:

```text
idempotency key
provider status lookup
reconciliation
```

rather than blindly charging again.

## 113. Example: Queue Backlog

```text
arrival = 1,000 msg/s
processing = 800 msg/s
```

Backlog grows by:

```text
200 msg/s
```

If processing later reaches:

```text
1,200 msg/s
```

the system has:

```text
200 msg/s
```

of backlog-draining capacity.

Recovery time must be calculated, not assumed.

## 114. Example: Database Saturation

Do not immediately increase application retries.

Check:

```text
slow queries
locks
connections
I/O
CPU
traffic
recent releases
```

Then:

```text
protect database
reduce load
fix expensive work
validate
```

## 115. Example: Node Failure

For Kubernetes:

```text
node failure
 |
pods reschedule
 |
remaining capacity
 |
PDB/topology
 |
application availability
```

If all replicas were on the failed node, replicas do not provide real
redundancy.

## 116. Example: AZ Failure

Verify distribution of:

```text
nodes
pods
load balancers
databases
dependencies
```

"Configured for three AZs" is not proof that workload capacity is actually
distributed across three AZs.

## 117. Example: Region Failure

A real multi-region design needs:

```text
traffic routing
data strategy
secrets
IAM
artifacts
deployment
observability
recovery procedure
```

A second region without tested recovery is not dependable DR.

## 118. Example: Security Compromise

If a pod is compromised:

```text
isolate
 |
identify identity
 |
inspect permissions
 |
rotate credentials
 |
inspect activity
 |
replace trusted workload
 |
validate
```

The objective is to contain lateral movement.

## 119. Example: Observability Failure

Telemetry systems should not become an unlimited dependency.

Use:

```text
buffers
bounded queues
sampling
drop policies
resource limits
```

Applications should continue operating within defined observability-degraded
behavior.

## 120. Example: CI Failure

CI failure may block:

```text
new builds
security scanning
new deployments
```

but should not automatically stop existing production workloads.

Separate delivery-plane availability from runtime availability.

## 121. Example: GitOps Failure

If Argo CD is unavailable:

```text
existing workloads may continue serving
new reconciliation is delayed
new changes may be blocked
```

Maintain emergency procedures and reconcile manual changes afterward.

## 122. Example: Registry Failure

Existing workloads may continue if images are already present.

Potential impact:

```text
new deployments
node replacement
pod scheduling requiring missing images
```

Use immutable artifacts and appropriate image availability strategies.

## 123. Example: DNS Failure

DNS failures can affect the entire request path.

Protect with:

```text
appropriate TTL
health checks
tested records
monitoring
controlled changes
```

Remember that cached DNS means changes are not necessarily instantaneous.

## 124. Example: Certificate Expiry

Automate:

```text
issuance
renewal
deployment
expiry monitoring
```

Do not wait until expiration.

## 125. Example: Secret Rotation

Safe rotation:

```text
create new credential
 |
deploy compatible application
 |
switch
 |
validate
 |
revoke old credential
```

Avoid treating rotation as an instantaneous destructive operation.

## 126. Example: Noisy Neighbor

If one tenant consumes all resources:

```text
rate limits
quotas
priority
resource isolation
queue partitioning
```

Protect other tenants' SLOs.

## 127. Example: Cost Explosion

Break cost into:

```text
compute
database
storage
network
NAT
observability
CI/CD
DR
```

Then correlate cost with:

```text
traffic
resource usage
retention
deployment volume
```

Do not cut reliability blindly.

## 128. Senior Design Checklist

For every architecture, ask:

```text
What is the critical path?
What can fail?
What is the largest blast radius?
What is the bottleneck?
What is the consistency requirement?
What is the delivery guarantee?
What happens on duplicate work?
What happens on out-of-order work?
What happens when the dependency is slow?
What happens when it disappears?
How is backpressure implemented?
How is recovery tested?
How is security enforced?
How is customer impact observed?
How does the design scale?
What does it cost?
```

## 129. Golden Rules

```text
1. Remote calls can fail.
2. Timeouts must be intentional.
3. Retries must be bounded.
4. Use backoff and jitter.
5. Consumers should be idempotent.
6. Queues require capacity planning.
7. Backpressure protects systems.
8. Replication is not backup.
9. Backup is not proven until restore is tested.
10. Availability is end-to-end.
11. Partial failure is normal.
12. Dependencies need explicit failure policies.
13. Shared components have larger blast radius.
14. State ownership must be explicit.
15. Compatibility matters during deployment.
16. Observability must cross service boundaries.
17. Network location does not equal trust.
18. Simplicity is a reliability feature.
19. Test failure, not only success.
20. Architecture must match business requirements.
```

## 130. Final Mental Model

Think about every distributed interaction as:

```text
REQUEST
  |
  +--> network
  |
  +--> remote processing
  |
  +--> response
  |
  +--> uncertainty
```

Then ask:

```text
What if it is slow?
What if it fails?
What if the response is lost?
What if the request is duplicated?
What if the message is duplicated?
What if it arrives out of order?
What if the network partitions?
What if the leader dies?
What if the database is unavailable?
What if the broker is unavailable?
What if the region fails?
What if credentials are compromised?
What if traffic increases 10x?
What if recovery takes longer than expected?
```

The purpose of distributed-systems engineering is to turn those questions into
explicit architecture, safe communication, controlled failure behavior,
observability, security and tested recovery.

## 131. Section 20 Progression

This foundation leads into:

```text
Distributed Systems
        |
Synchronous vs Asynchronous
        |
Messaging Fundamentals
        |
RabbitMQ
        |
RabbitMQ Production
        |
Kafka
        |
Kafka Production
        |
Event-Driven Architecture
        |
Retry / DLQ
        |
Idempotency
        |
Ordering / Delivery Semantics
        |
Messaging Security
        |
Troubleshooting
        |
Production Architecture
        |
RoboShop Integration
        |
Projects
        |
Interview Preparation
```

The next files will move from theory into concrete RabbitMQ architecture,
configuration, Kubernetes deployment, production operations and failure
handling.

# END OF 01-Distributed-Systems-Fundamentals.md
