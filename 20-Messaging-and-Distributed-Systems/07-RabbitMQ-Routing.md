# RabbitMQ-Routing

## Purpose

RabbitMQ routing is the mechanism that determines which queues receive a
published message.

The production mental model is:

```text
Producer
   |
   | exchange + routing key + headers
   v
Exchange
   |
   | routing algorithm
   v
Matching bindings
   |
   +-------------------+
   |                   |
 Queue A             Queue B
   |                   |
Consumer A           Consumer B
```

Routing is not simply "send a message to a queue."

A production routing design must answer:

```text
What is the event?
Which exchange receives it?
Which routing key represents it?
Which bindings match it?
Which queues receive it?
What happens if nothing matches?
What happens if too many destinations match?
How is routing tested?
How is routing observed?
How does routing evolve?
How does routing behave during failures?
```

---

# 1. Routing Fundamentals

RabbitMQ routing consists of:

```text
publication
exchange selection
routing-key selection
binding evaluation
queue selection
delivery
```

---

# 2. Publication

A producer publishes to an exchange.

Conceptually:

```text
publish(
    exchange,
    routing_key,
    message,
    properties
)
```

The producer does not normally publish directly into a queue except through
the special default exchange mechanism.

---

# 3. Exchange Selection

The exchange is part of the producer's publication contract.

Example:

```text
exchange = orders.events
```

If the producer selects the wrong exchange:

```text
correct routing key
+
wrong exchange
=
wrong topology
```

---

# 4. Routing Key

The routing key is metadata used by applicable exchange types.

Example:

```text
order.created
```

It is not the payload itself.

---

# 5. Binding

A binding defines how a destination is associated with an exchange.

Conceptually:

```text
Exchange
 |
Binding
 |
Queue
```

For topic/direct routing, the binding key/pattern participates in matching.

---

# 6. Routing Algorithm

At a high level:

```text
receive message
 |
identify exchange
 |
evaluate exchange type
 |
evaluate bindings
 |
determine matching destinations
 |
route message
```

The exact internal implementation should not be assumed from this simplified
model, but the architectural behavior is the important part.

---

# 7. Direct Routing

Direct routing uses exact routing-key matching.

```text
message key:
order.created

binding:
order.created

=> match
```

---

# 8. Direct Routing Failure

```text
message:
order.created

binding:
order.updated

=> no match
```

The producer must handle the possibility of no destination.

---

# 9. Topic Routing

Topic routing uses dot-separated words.

Example:

```text
order.created
order.created.us
order.created.priority
```

Bindings can use:

```text
*
#
```

---

# 10. Topic Word

A topic routing key is interpreted as words separated by dots.

Example:

```text
order.created.us
```

contains:

```text
order
created
us
```

---

# 11. Star Wildcard

`*` matches exactly one word.

Example:

```text
order.*
```

matches:

```text
order.created
order.updated
order.cancelled
```

---

# 12. Star Does Not Match Multiple Words

```text
order.*
```

does not match:

```text
order.created.us
```

because there are two words after `order`.

---

# 13. Hash Wildcard

`#` matches zero or more words.

Example:

```text
order.#
```

can match:

```text
order
order.created
order.created.us
order.created.us.priority
```

---

# 14. Combined Wildcards

Example:

```text
*.created.#
```

This represents:

```text
one word
+
created
+
zero or more additional words
```

Design these patterns carefully.

---

# 15. Routing-Key Taxonomy

A strong routing-key taxonomy should be:

```text
predictable
documented
stable
searchable
human-readable
```

Example:

```text
<domain>.<entity/event>
```

or:

```text
<domain>.<event>
```

---

# 16. Good Routing Keys

Examples:

```text
order.created
order.updated
order.cancelled
payment.completed
payment.failed
inventory.reserved
shipment.created
```

---

# 17. Poor Routing Keys

Avoid:

```text
event1
message2
foo.bar
abc.xyz
```

unless there is a deliberate technical reason.

---

# 18. Routing Key as Contract

A routing key is effectively part of the event API.

Changing:

```text
order.created
```

to:

```text
orders.created
```

can break consumers.

Treat changes as compatibility events.

---

# 19. Routing Key Versioning

Possible strategies:

```text
payload schema version
event type version
routing-key version
```

Do not introduce versions everywhere without a reason.

---

# 20. Event vs Command Routing

Events:

```text
order.created
```

describe facts.

Commands:

```text
order.cancel
```

request an action.

Routing should reflect the semantic difference where useful.

---

# 21. Event Routing

Typical:

```text
orders.events
 |
topic
 |
order.*
```

Multiple consumers can subscribe.

---

# 22. Command Routing

Typical:

```text
orders.commands
 |
direct
 |
cancel.order
 |
Order Worker
```

Commands usually have an intended handler.

---

# 23. Direct Routing Design

Use direct routing when:

```text
destination is explicit
```

Example:

```text
billing.commands
 |
charge.order
 |
billing.worker
```

---

# 24. Topic Routing Design

Use topic routing when:

```text
consumers need categories
```

Example:

```text
orders.events
 |
order.#
```

---

# 25. Fanout Routing

Fanout routing sends the publication to all bound queues.

```text
fanout
 |
+--> A
+--> B
+--> C
```

Use for broadcast semantics.

---

# 26. Headers Routing

Headers routing evaluates message headers rather than relying primarily on
routing-key taxonomy.

Example:

```text
region=us
priority=high
```

Use only when metadata-based routing is genuinely useful.

---

# 27. Default Exchange Routing

The unnamed default exchange provides convenient direct routing using queue
names.

```text
exchange = ""
routing_key = queue-name
```

Example:

```text
orders.queue
```

is directly addressed by using:

```text
routing_key = orders.queue
```

---

# 28. Routing Graph

Think of routing as a graph:

```text
Producer
 |
Exchange A
 |
+--> Queue A
|
+--> Exchange B
       |
       +--> Queue B
       +--> Queue C
```

This is more accurate than thinking only in terms of producer-to-queue pairs.

---

# 29. Exchange-to-Exchange Routing

RabbitMQ supports exchange-to-exchange bindings.

Example:

```text
orders.events
 |
regional.events
 |
us.orders.queue
```

This enables composed routing.

---

# 30. Exchange Chain

A chain may look like:

```text
A -> B -> C -> Queue
```

Use chains deliberately.

Too many stages increase:

```text
debugging complexity
operational risk
topology coupling
```

---

# 31. Routing Layers

A layered architecture can be:

```text
Domain Exchange
      |
Regional Exchange
      |
Workload Exchange
      |
Queue
```

Useful when routing needs clear stages.

---

# 32. Routing Layer Trade-Off

Advantages:

```text
reusable topology
separation
central routing policy
```

Disadvantages:

```text
more hops
harder troubleshooting
more configuration
```

---

# 33. Routing Fanout

A single publication may reach many queues.

```text
1 publication
 |
+--> Queue A
+--> Queue B
+--> Queue C
+--> Queue D
```

This is routing amplification.

---

# 34. Fanout Factor

Define:

```text
fanout factor =
number of destination deliveries
/
number of publications
```

Example:

```text
100,000 publications
500,000 deliveries

fanout = 5x
```

---

# 35. Fanout Capacity Planning

At:

```text
100,000 messages/s
```

with:

```text
5 destinations/message
```

the system may need to handle approximately:

```text
500,000 destination deliveries/s
```

before other overhead.

---

# 36. Broad Topic Subscription

Binding:

```text
#
```

can receive almost all topic messages.

Useful for:

```text
audit
debugging
central event capture
```

but potentially expensive.

---

# 37. Narrow Topic Subscription

Prefer:

```text
order.#
```

when the consumer only needs order events.

This reduces unnecessary delivery.

---

# 38. Binding Overlap

Multiple bindings can match the same publication.

This must be considered when designing topology.

Avoid assuming:

```text
one matching pattern
=
one delivery
```

without understanding the exact binding topology and RabbitMQ behavior.

---

# 39. Routing Duplicate vs Redelivery

These are different concepts.

Routing duplication:

```text
topology causes multiple intended destinations/delivery paths
```

Redelivery:

```text
consumer did not successfully acknowledge and message is delivered again
```

Both can create repeated business processing.

---

# 40. Idempotency

Consumers should be designed to tolerate duplicates.

Use:

```text
event_id
message_id
idempotency key
inbox
unique constraint
```

where appropriate.

---

# 41. Routing Key and Business Key

Do not confuse:

```text
routing key
```

with:

```text
business key
```

Example:

```text
routing key = order.created
business key = order-123
```

---

# 42. Business-Key Routing

If ordering is required per order:

```text
order-123
```

can be used to select a consistent routing destination in a sharded design.

---

# 43. Routing Shards

Example:

```text
orders.0
orders.1
orders.2
orders.3
```

A routing function can assign events to shards.

```text
hash(order_id) % 4
```

---

# 44. Sharding Trade-Off

Benefits:

```text
parallelism
smaller queues
hot-queue reduction
```

Costs:

```text
more queues
more bindings
routing complexity
ordering considerations
```

---

# 45. Stable Shard Mapping

For per-key ordering:

```text
same order_id
 |
same shard
 |
same queue
```

This allows parallel processing across keys while preserving sequence within
a key when the consumer design supports it.

---

# 46. Hot Key

A single key can dominate a shard.

Example:

```text
customer-123
 |
70% traffic
```

Even perfect sharding can leave one hot key.

---

# 47. Hot-Key Mitigation

Options:

```text
split workload
rate limit
special handling
relax ordering if allowed
partition more intelligently
```

Never break a business ordering guarantee without approval.

---

# 48. Routing and Ordering

Routing determines destination.

Ordering determines processing sequence.

They are related but not identical.

---

# 49. Global Ordering

Global ordering is expensive and difficult to scale.

Ask whether the business actually requires:

```text
global order
```

or:

```text
per-customer order
per-order order
per-account order
```

---

# 50. Per-Key Ordering

Usually more scalable:

```text
order A:
1 -> 2 -> 3

order B:
1 -> 2 -> 3
```

A and B can process concurrently.

---

# 51. Routing and Consumer Concurrency

Even if routing preserves queue order:

```text
Queue
 |
Consumer A
Consumer B
```

processing completion can occur out of order.

---

# 52. Routing and Prefetch

High prefetch can cause consumers to hold many messages.

This affects:

```text
fairness
latency
recovery
ordering assumptions
```

Routing design should be reviewed together with consumer behavior.

---

# 53. Routing and Retry

Retry can change observed processing order.

Example:

```text
1 fails -> retry
2 succeeds
```

Business completion:

```text
2
1
```

Therefore retry architecture must be considered when ordering matters.

---

# 54. Routing and DLQ

A failed message may be routed:

```text
Main Queue
 |
DLX
 |
DLQ
```

This is separate from initial exchange routing.

---

# 55. Alternate Routing

Unroutable messages can be sent to an alternate exchange.

```text
Primary Exchange
 |
no match
 |
Alternate Exchange
 |
Fallback Queue
```

This creates an explicit unroutable-message path.

---

# 56. Alternate Exchange vs DLX

Alternate exchange:

```text
publication could not be routed
```

Dead-letter exchange:

```text
message already associated with a queue and later dead-lettered
```

Do not use these terms interchangeably.

---

# 57. Unroutable Message

Common causes:

```text
wrong exchange
wrong routing key
missing binding
wrong vhost
wrong exchange type
topology drift
```

---

# 58. Unroutable Message Detection

Use:

```text
mandatory publishing
returned-message handling
application metrics
alternate exchange
```

depending on the required behavior.

---

# 59. Mandatory Publishing

With mandatory publishing:

```text
Producer
 |
publish
 |
no queue route
 |
return
 |
Producer
```

The producer can decide what to do.

---

# 60. Returned Message Handling

For critical messages:

```text
return
 |
log
 |
metric
 |
persist/retry
 |
alert
```

Do not silently drop.

---

# 61. Publisher Confirms

Publisher confirms provide producer-side acknowledgement of broker publication
according to RabbitMQ confirmation semantics.

They are not consumer acknowledgements.

---

# 62. Confirm and Routing

Important:

```text
publisher confirm
!=
consumer processed
```

Also do not assume:

```text
confirm
=
intended queue matched
```

Use routing detection separately where required.

---

# 63. Publication Reliability

For important events:

```text
persistent message
+
durable topology
+
publisher confirms
+
routing validation
+
idempotent consumers
```

This creates a stronger end-to-end design.

---

# 64. Routing Contract Test

Test:

```text
publish order.created
 |
expect orders.queue
```

This can detect missing bindings.

---

# 65. Routing Regression Test

A topology change should verify:

```text
old event
new event
unroutable event
unexpected event
```

---

# 66. Negative Routing Test

Publish:

```text
order.invalid
```

and verify:

```text
not delivered to payment queue
```

This prevents over-broad subscriptions.

---

# 67. Routing Security Test

Verify:

```text
producer A
```

cannot publish to unauthorized exchanges.

Verify:

```text
consumer B
```

cannot consume unauthorized queues.

---

# 68. Routing Topology as Code

Represent:

```text
exchange
type
binding
routing key
queue
policy
```

in version control.

---

# 69. Topology Review

Pull requests should answer:

```text
Why is this binding required?
What traffic will it receive?
What is its owner?
What is its expected fanout?
Does it change security?
Does it change capacity?
```

---

# 70. Routing Drift

Drift:

```text
Git topology
      !=
RabbitMQ topology
```

Examples:

```text
missing binding
extra binding
wrong exchange
unexpected queue
```

---

# 71. Routing Reconciliation

A platform process can compare:

```text
desired state
```

against:

```text
actual state
```

and correct supported differences.

Use careful change control for production.

---

# 72. Routing Migration

Example:

```text
old.exchange
 |
new.exchange
```

Migration:

```text
create new
 |
create bindings
 |
deploy consumers
 |
validate
 |
switch producer
 |
drain old
 |
delete old
```

---

# 73. Dual Routing

A producer can temporarily publish to:

```text
old exchange
+
new exchange
```

This can help migration but may create duplicate business events.

Consumers need idempotency.

---

# 74. Routing Compatibility

During migration:

```text
Producer v2
Consumer v1
Consumer v2
```

may coexist.

Maintain compatibility across the transition.

---

# 75. Routing-Key Migration

Old:

```text
order.created
```

New:

```text
orders.created
```

A safer migration can temporarily support both:

```text
old binding
+
new binding
```

or dual publication, depending on requirements.

---

# 76. Routing Contract Deprecation

Document:

```text
deprecated routing key
replacement
migration date
affected consumers
```

Remove only after usage is confirmed absent.

---

# 77. Routing Ownership

Every routing key should ideally have:

```text
producer owner
event owner
consumer inventory
documentation
```

---

# 78. Routing Documentation

Maintain:

```text
Exchange
Routing Key
Queue
Consumer
Purpose
Criticality
```

Example:

```text
orders.events | order.created | inventory.queue | Inventory Worker
```

---

# 79. Routing Inventory

A large RabbitMQ installation should maintain an inventory of:

```text
exchanges
queues
bindings
routing keys
owners
```

This reduces operational uncertainty.

---

# 80. Routing Blast Radius

Broad binding:

```text
#
```

can dramatically increase the blast radius of an event-volume spike.

---

# 81. Routing Isolation

Separate:

```text
critical
normal
batch
analytics
```

when their operational characteristics differ.

---

# 82. Critical Event Routing

Critical events should have:

```text
dedicated queue
clear ownership
strict monitoring
bounded retry
DLQ
idempotency
```

---

# 83. Best-Effort Routing

For best-effort events:

```text
analytics
telemetry
recommendations
```

loss may be acceptable depending on business requirements.

Do not apply critical-event guarantees unnecessarily.

---

# 84. Routing and Data Classification

If routing payloads contain sensitive data, minimize exposure.

A consumer subscribed broadly may receive data it does not need.

Use:

```text
narrow bindings
separate exchanges
separate queues
```

where required.

---

# 85. Routing and Least Privilege

Example:

```text
Order Producer
 |
write -> orders.events
```

The producer does not need access to:

```text
payments.events
```

---

# 86. Routing and Vhosts

Use vhosts for logical isolation.

Example:

```text
/production
/staging
/development
```

But choose separate RabbitMQ infrastructure when stronger isolation or
failure-domain separation is required.

---

# 87. Multi-Tenant Routing

Options:

```text
tenant-specific exchange
tenant-specific queue
shared exchange with tenant routing key
separate vhost
separate broker
```

Choose based on:

```text
security
scale
cost
operational complexity
```

---

# 88. Shared Tenant Exchange

Example:

```text
events
 |
tenant.a.order.created
tenant.b.order.created
```

Consumers must use carefully scoped bindings.

---

# 89. Tenant Routing Key

Possible taxonomy:

```text
tenant.<tenant-id>.<domain>.<event>
```

Be careful about exposing sensitive tenant identifiers.

---

# 90. Tenant Isolation Risk

Broad binding:

```text
#
```

can accidentally receive multiple tenants' data.

Security review must include routing patterns.

---

# 91. Tenant-Specific Queues

Example:

```text
tenant-a.orders
tenant-b.orders
```

This improves isolation but increases resource count.

---

# 92. Routing Resource Explosion

Thousands of tenants can create:

```text
thousands of queues
thousands of bindings
```

Evaluate whether shared topology is more appropriate.

---

# 93. Routing Cost

Routing cost includes:

```text
broker CPU
network
storage
replication
consumer work
observability
```

Fanout is especially important.

---

# 94. Cross-AZ Routing

If destination queues are associated with workloads across AZs, traffic may
cross AZ boundaries.

Consider:

```text
resilience
latency
cost
```

together.

---

# 95. Cross-Region Routing

Do not assume normal exchange routing provides global replication.

For multi-region systems, use an explicitly designed mechanism such as
federation or shovel where appropriate.

---

# 96. Active-Active Routing

Active-active messaging requires explicit decisions about:

```text
duplicate events
ordering
conflicts
regional ownership
failover
```

---

# 97. Region-Local Routing

A common design:

```text
Region A
 |
orders.events
 |
local queues

Region B
 |
orders.events
 |
local queues
```

Cross-region synchronization is a separate concern.

---

# 98. Routing DR

DR should restore:

```text
exchanges
bindings
queues
policies
permissions
```

Then validate:

```text
publication
routing
consumption
```

---

# 99. Routing Restore Test

```text
restore RabbitMQ
 |
restore topology
 |
publish test event
 |
verify target queue
 |
consume
 |
verify business effect
```

---

# 100. Routing Observability

Observe:

```text
publish rate
unroutable rate
returns
confirm failures
queue delivery
consumer processing
```

---

# 101. Routing Metrics

Application metrics:

```text
messages_published_total
messages_unroutable_total
publisher_confirm_failures_total
```

Broker metrics should complement these.

---

# 102. Routing Logs

Log:

```text
exchange
routing key
message ID
correlation ID
publication result
```

Avoid full sensitive payloads.

---

# 103. Routing Tracing

Trace:

```text
HTTP request
 |
producer
 |
RabbitMQ exchange
 |
queue
 |
consumer
 |
database
```

Propagate:

```text
trace ID
correlation ID
```

---

# 104. Routing SLO

Example:

```text
99.99% of valid order events must be routed to their intended destination.
```

Measure:

```text
unroutable events
publication failures
consumer completion
```

---

# 105. End-to-End Routing SLO

Better:

```text
publish
 |
route
 |
deliver
 |
process
 |
business effect
```

Measure the complete workflow.

---

# 106. Routing Latency

Measure:

```text
publication time
 |
delivery time
```

Then:

```text
routing/delivery waiting time
```

can be analyzed separately from application processing.

---

# 107. Routing Failure Framework

When messages disappear:

```text
1. Confirm producer publication.
2. Confirm exchange.
3. Confirm vhost.
4. Confirm exchange type.
5. Confirm routing key.
6. Confirm bindings.
7. Confirm destination queues.
8. Check mandatory returns.
9. Check publisher confirms.
10. Check consumer state.
```

---

# 108. Wrong Routing Key Incident

Example:

```text
expected:
order.created

actual:
orders.created
```

Fix:

```text
producer contract
or
binding
```

Do not guess which side is wrong.

---

# 109. Wrong Exchange Incident

Producer:

```text
payments.events
```

Expected:

```text
orders.events
```

Check application configuration and deployment environment.

---

# 110. Wrong Vhost Incident

Producer connects to:

```text
/staging
```

while production topology exists in:

```text
/production
```

The resources appear "missing."

---

# 111. Wrong Exchange Type Incident

Expected:

```text
topic
```

Runtime:

```text
direct
```

Wildcard behavior will not work as intended.

---

# 112. Missing Binding Incident

```text
Exchange
 |
X
 |
Queue
```

Restore the binding through the approved topology mechanism.

---

# 113. Broad Binding Incident

A new:

```text
#
```

binding is deployed.

Result:

```text
unexpected traffic
queue growth
consumer overload
```

Remove or narrow it after assessing impact.

---

# 114. Fanout Incident

A high-volume exchange receives a new queue.

Every event now enters the queue.

Result:

```text
storage growth
consumer load
network growth
```

---

# 115. Unroutable Incident

Symptoms:

```text
producer believes publish succeeded
consumer sees nothing
```

Check:

```text
mandatory returns
alternate exchange
bindings
routing key
```

---

# 116. Confirm Incident

Publisher confirms fail.

Investigate:

```text
connection
channel
broker health
resource alarms
permissions
```

---

# 117. Routing During Broker Restart

Clients may experience:

```text
connection loss
publish retry
```

Potential duplicates exist if the producer cannot determine whether a previous
publication succeeded.

Use:

```text
message IDs
idempotency
confirm-aware retry
```

---

# 118. Routing During Consumer Restart

The exchange remains available.

The queue may accumulate messages temporarily.

After consumer recovery:

```text
backlog drains
```

provided capacity is sufficient.

---

# 119. Routing During Queue Failure

Investigate:

```text
queue state
node
storage
replication
consumer
```

Exchange health alone does not prove delivery health.

---

# 120. Routing During Node Failure

Check:

```text
remaining cluster
queue availability
client recovery
bindings
publication confirms
```

---

# 121. Routing During AZ Failure

Check:

```text
remaining RabbitMQ nodes
queue quorum
network
client endpoints
consumer capacity
```

---

# 122. Routing During Network Partition

Check:

```text
node connectivity
client connectivity
cluster state
Kubernetes networking
security controls
DNS
```

Avoid making multiple topology changes before understanding the failure.

---

# 123. Routing and Backpressure

Routing can continue while consumers slow down.

```text
Exchange
 |
Queue
 |
slow consumer
 |
backlog
```

Backpressure must eventually reach producers or workload controls.

---

# 124. Producer Throttling

If downstream cannot keep up:

```text
Producer
 |
rate limit
 |
Exchange
```

This protects broker and queues.

---

# 125. Routing and Circuit Breaker

For command/event producers that depend on downstream health, application-level
circuit breakers and rate limits can reduce overload.

RabbitMQ should not be expected to solve every distributed-systems backpressure
problem.

---

# 126. Routing and Retry Storm

Bad:

```text
failure
 |
retry
 |
failure
 |
retry
```

Thousands of messages can circulate rapidly.

Use:

```text
bounded retry
backoff
jitter
DLQ
```

---

# 127. Routing Retry Topology

```text
Main Exchange
 |
Main Queue
 |
failure
 |
Retry Exchange
 |
Retry Queue
 |
delay
 |
Main Exchange
```

---

# 128. Routing Retry and Ordering

Retries can cause:

```text
event 2 completes
event 1 waits
```

If ordering matters, use a design that explicitly handles this.

---

# 129. Routing and Poison Messages

A poison message should not repeatedly traverse the same routing path forever.

Use:

```text
attempt tracking
DLQ
manual inspection
```

---

# 130. Routing and Dead-Letter

A final failure path:

```text
Main Queue
 |
DLX
 |
DLQ
```

The DLQ itself may have a separate routing topology for operational tooling.

---

# 131. Routing DLQ Replay

Replay:

```text
DLQ
 |
inspect
 |
fix
 |
sample replay
 |
monitor
 |
full replay
```

Republish through the intended exchange rather than bypassing the normal
routing contract unless there is a documented operational reason.

---

# 132. Routing and Replay Safety

Replay can cause:

```text
duplicate business effects
```

Require:

```text
idempotency
```

and controlled rate.

---

# 133. Replay Routing Key

When replaying, preserve the original event semantics where possible.

Do not accidentally publish:

```text
order.created
```

as:

```text
order.updated
```

during a recovery operation.

---

# 134. Routing and Message Metadata

Useful properties:

```text
message_id
correlation_id
content_type
content_encoding
headers
timestamp
delivery mode
```

Use them intentionally.

---

# 135. Routing and Correlation ID

Correlation IDs allow operators to trace:

```text
request
 |
event
 |
consumer
 |
downstream call
```

---

# 136. Routing and Causation ID

Causation ID can link:

```text
command
 |
event
```

This helps reconstruct distributed workflows.

---

# 137. Routing and Content Type

Set content metadata appropriately.

Example:

```text
application/json
```

Consumers should not assume every queue contains identical serialization.

---

# 138. Routing and Schema Evolution

A queue may receive events from multiple producer versions.

Routing does not guarantee schema compatibility.

Use:

```text
backward-compatible schema
versioning
consumer tolerance
contract testing
```

---

# 139. Routing and Rolling Deployment

During deployment:

```text
Producer v1
Producer v2
Consumer v1
Consumer v2
```

may coexist.

Routing contracts must remain compatible.

---

# 140. Routing and Blue-Green

Blue and green consumers can both consume from the same queue.

If only one version should process messages, use controlled handoff or separate
queues/routing.

---

# 141. Routing and Canary

Canary routing can use:

```text
separate exchange
specific routing key
dedicated queue
controlled publisher
```

Avoid arbitrary message splitting for state-changing commands.

---

# 142. Routing by Region

Possible:

```text
order.created.us
order.created.eu
```

with:

```text
order.created.us -> US queue
order.created.eu -> EU queue
```

This is useful when regional processing is required.

---

# 143. Routing by Environment

Do not normally put:

```text
prod
staging
```

inside routing keys as the primary isolation mechanism.

Use:

```text
vhosts
separate clusters
environment-specific infrastructure
```

for stronger isolation.

---

# 144. Routing by Priority

Possible patterns:

```text
order.critical
order.normal
order.batch
```

This can be simpler than priority queue semantics when workloads require
different scaling and isolation.

---

# 145. Routing by Workload

Example:

```text
orders.events
 |
+--> order.realtime
+--> order.batch
+--> order.audit
```

This isolates workloads.

---

# 146. Routing by Consumer Capability

Consumers should subscribe only to events they understand.

Avoid:

```text
consumer -> #
```

unless it intentionally handles the complete event domain.

---

# 147. Routing by Version

Possible:

```text
order.created.v1
order.created.v2
```

Use only when explicit routing-level version isolation is required.

---

# 148. Version Routing Trade-Off

Advantages:

```text
clear separation
independent migration
```

Costs:

```text
more bindings
more queues
version management
```

Prefer compatibility when practical.

---

# 149. Routing Governance

Define standards for:

```text
exchange naming
routing-key naming
wildcards
ownership
versioning
deprecation
security
```

---

# 150. Routing Change Management

Any production routing change should include:

```text
impact assessment
capacity assessment
security assessment
rollback plan
monitoring plan
```

---

# 151. Routing Rollback

If a routing change causes problems:

```text
restore previous binding
 |
restore previous producer config
 |
verify traffic
```

Keep topology definitions versioned.

---

# 152. Routing Audit

Periodically audit:

```text
unused exchanges
unused bindings
unroutable events
broad bindings
orphaned queues
unknown owners
```

---

# 153. Orphaned Binding

A binding may remain after its consumer/queue is removed.

Clean it up after verification.

---

# 154. Orphaned Exchange

An exchange with no active publishers or consumers may be obsolete.

Confirm usage before deletion.

---

# 155. Routing Capacity Review

Review:

```text
publication rate
fanout factor
binding count
destination count
message size
cross-AZ traffic
consumer capacity
```

---

# 156. Routing Load Test

Test:

```text
normal traffic
peak traffic
burst traffic
fanout
wildcard subscriptions
unroutable events
consumer slowdown
```

---

# 157. Routing Chaos Test

Examples:

```text
remove test binding
stop consumer
restart broker
simulate network failure
introduce downstream latency
```

Validate:

```text
routing correctness
retry
DLQ
recovery
```

---

# 158. Routing Synthetic Monitoring

Publish a synthetic event:

```text
synthetic.order.created
```

Then verify:

```text
expected exchange
expected queue
expected consumer
```

This provides continuous topology validation.

---

# 159. Routing Canary Event

A canary event can include:

```text
synthetic=true
test_id
timestamp
```

Consumers should recognize and handle it safely.

---

# 160. Routing Production Dashboard

Include:

```text
Publications
Confirms
Unroutable
Returns
Queue Depth
Oldest Age
Consumer Count
Processing Rate
DLQ
```

---

# 161. Routing Alert

Alert on:

```text
unexpected unroutable rate
confirm failures
sudden fanout traffic
queue age
consumer disappearance
```

---

# 162. Routing Error Budget

If routing SLO is:

```text
99.99%
```

the error budget is:

```text
0.01%
```

Use the budget to prioritize topology reliability work.

---

# 163. Routing Incident: Wrong Routing Key

Response:

```text
identify first bad publication
 |
confirm intended contract
 |
restore compatibility
 |
reconcile affected messages
 |
test
```

---

# 164. Routing Incident: Missing Binding

Response:

```text
confirm scope
 |
restore binding
 |
synthetic publish
 |
verify destination
 |
reconcile affected events
```

---

# 165. Routing Incident: Excessive Fanout

Response:

```text
identify new binding
 |
measure amplification
 |
protect consumers/storage
 |
narrow/remove binding
 |
review change
```

---

# 166. Routing Incident: Unroutable Spike

Response:

```text
check producer deployment
check routing keys
check exchange
check bindings
check vhost
check topology changes
```

---

# 167. Routing Incident: Consumer Overload

Response:

```text
identify unexpected routing
 |
reduce unnecessary destinations
 |
protect downstream
 |
scale safely
```

---

# 168. Routing Incident: Cross-Tenant Leakage Risk

If a broad binding causes unauthorized tenant traffic:

```text
contain
 |
restrict binding
 |
audit affected messages
 |
review permissions
 |
validate isolation
```

Treat this as a security incident when applicable.

---

# 169. Routing Incident: Retry Storm

Response:

```text
stop uncontrolled requeue
 |
reduce producer pressure
 |
bounded retry
 |
DLQ poison messages
 |
restore dependency
```

---

# 170. Routing Interview: Basic

### What is RabbitMQ routing?

Answer:

```text
Routing is the process by which RabbitMQ evaluates an incoming publication
against the selected exchange's routing behavior and bindings to determine
which queues or exchanges receive the message.
```

---

# 171. Routing Interview: Direct

### How does direct routing work?

Answer:

```text
A direct exchange routes using exact routing-key matches between the published
message and queue bindings.
```

---

# 172. Routing Interview: Topic

### How does topic routing work?

Answer:

```text
A topic exchange evaluates dot-separated routing keys against binding patterns.
'*' matches one word and '#' matches zero or more words.
```

---

# 173. Routing Interview: Fanout

### How does fanout routing work?

Answer:

```text
A fanout exchange routes the publication to all queues bound to that exchange,
providing broadcast semantics.
```

---

# 174. Routing Interview: Headers

### When would you use headers routing?

Answer:

```text
When routing decisions depend naturally on message metadata or headers rather
than a stable routing-key taxonomy.
```

---

# 175. Routing Interview: Default Exchange

### How does the default exchange work?

Answer:

```text
The special unnamed exchange provides direct addressing to queues using the
queue name as the routing key through automatically created bindings.
```

---

# 176. Routing Interview: Unroutable

### What is an unroutable message?

Answer:

```text
A publication is unroutable when it does not reach a queue through the
available exchange routing and bindings.
```

---

# 177. Routing Interview: Mandatory

### Why use mandatory publishing?

Answer:

```text
It allows the producer to receive a return when a message cannot be routed to
a queue, making routing failures visible to the application.
```

---

# 178. Routing Interview: Confirm

### What does a publisher confirm tell you?

Answer:

```text
It confirms broker acceptance of the publication according to RabbitMQ's
publisher-confirm semantics. It is different from queue delivery and consumer
processing confirmation.
```

---

# 179. Routing Interview: Confirm vs Mandatory

Answer:

```text
Publisher confirms address broker-side publication acknowledgement, while
mandatory publishing addresses the case where the publication has no matching
queue route. They solve different parts of publication reliability.
```

---

# 180. Routing Interview: Alternate Exchange

### Why use an alternate exchange?

Answer:

```text
It provides an explicit fallback route for publications that would otherwise
be unroutable, allowing operators to detect and inspect routing mistakes.
```

---

# 181. Routing Interview: AE vs DLX

Answer:

```text
An alternate exchange handles unroutable publications at the exchange-routing
stage. A dead-letter exchange handles messages that have already reached a
queue and later become eligible for dead-lettering.
```

---

# 182. Routing Interview: Wildcards

### Difference between '*' and '#'

Answer:

```text
'*' matches one topic word. '#' matches zero or more topic words.
```

---

# 183. Routing Interview: Broad Subscription

### Why is '#' dangerous?

Answer:

```text
It can subscribe a queue to a very large portion of the event space, causing
unexpected traffic, storage growth and consumer overload.
```

---

# 184. Routing Interview: Fanout

### What is fanout amplification?

Answer:

```text
It is the increase in destination deliveries caused by routing each publication
to multiple queues. I calculate it as destination deliveries divided by
publications.
```

---

# 185. Routing Interview: Ordering

### Does routing guarantee processing order?

Answer:

```text
No. Routing selects destinations. Multiple consumers, prefetch, processing
latency, retry and redelivery can change completion order. Ordering requirements
must be explicitly designed.
```

---

# 186. Routing Interview: Sharding

### How would you route a hot workload?

Answer:

```text
I would first measure whether consumer or downstream capacity is the actual
bottleneck. If the queue itself is the bottleneck, I would consider partitioning
by a stable business key across multiple queues while preserving required
per-key ordering.
```

---

# 187. Routing Interview: Multi-Tenant

### How would you route multiple tenants?

Answer:

```text
I would choose between separate vhosts, tenant-specific exchanges or queues,
or shared topology with tenant-scoped routing based on security, scale and
operational requirements. I would avoid broad bindings that can cross tenant
boundaries.
```

---

# 188. Routing Interview: Migration

### How do you migrate a routing key?

Answer:

```text
Create the new binding/topology, maintain compatibility, migrate consumers,
optionally dual-publish when necessary, validate traffic, switch producers,
then remove the old route after confirming it is unused.
```

---

# 189. Routing Interview: Duplicate

### How do you handle duplicate events?

Answer:

```text
Use stable event/message IDs and idempotent business processing. An inbox,
unique constraint or provider-supported idempotency key can protect the business
operation.
```

---

# 190. Routing Interview: Routing Failure

### Producer says the event was published but the consumer sees nothing.

Answer:

```text
I would check the exchange, vhost, exchange type, routing key, bindings,
mandatory returns and destination queue. I would distinguish broker publication
confirmation from successful routing and consumer processing.
```

---

# 191. Routing Interview: Binding Failure

### A binding was accidentally removed.

Answer:

```text
Restore the binding through the approved topology mechanism, validate with a
synthetic event, identify the affected publication window and reconcile only
the required events using an idempotent process.
```

---

# 192. Routing Interview: Exchange Chain

### Would you use five exchange-to-exchange hops?

Answer:

```text
Only if the routing requirements justify the layers. I prefer the simplest
topology that provides the required isolation and reuse because every additional
routing layer increases operational complexity.
```

---

# 193. Routing Interview: Production Event Bus

### Design an order event bus.

Answer:

```text
Use a durable topic exchange such as orders.events, define stable routing keys
such as order.created and order.cancelled, create isolated queues per consumer
workload, use publisher confirms, detect unroutable publications, apply
bounded retry and DLQ behavior at the queue layer, propagate event IDs and
correlation IDs, and make consumers idempotent.
```

---

# 194. Routing Interview: Audit

### How would you implement an audit subscriber?

Answer:

```text
Use a dedicated durable queue with a deliberately broad binding such as
order.# for the required domain, capacity-plan the resulting traffic and
storage, and treat the audit consumer as an independent workload.
```

---

# 195. Routing Interview: Analytics

### How would you protect analytics from payment traffic?

Answer:

```text
Use domain-specific exchanges or narrowly scoped bindings so analytics receives
only required events. Do not place unrelated workloads on one shared queue.
```

---

# 196. Routing Interview: Critical vs Batch

### How do you isolate critical events?

Answer:

```text
Use separate queues and, where appropriate, separate routing paths so batch or
analytics backlog cannot consume the resources required by critical workloads.
```

---

# 197. Routing Interview: Routing Observability

### What metrics matter?

Answer:

```text
publication rate, publisher-confirm failures, unroutable/returned messages,
queue depth, oldest message age, consumer count, processing rate and DLQ growth.
```

---

# 198. Routing Interview: Capacity

### What would you calculate before adding a broad binding?

Answer:

```text
publication rate × expected matched destinations × average message size,
plus consumer processing capacity, storage, network and cross-AZ implications.
```

---

# 199. Routing Interview: Failure Domain

### How does routing relate to HA?

Answer:

```text
Routing determines destinations, but queue availability and message durability
depend on the queue and broker architecture. I would design routing separately
from queue replication and then validate the complete failure path.
```

---

# 200. Routing Interview: Kubernetes

### How do you manage RabbitMQ routing in EKS?

Answer:

```text
I would manage exchanges, bindings and queues declaratively where practical,
version them in Git, validate topology in CI/staging, use private networking
and TLS, and test routing after deployment.
```

---

# 201. Routing Interview: Security

### How do you prevent unauthorized routing?

Answer:

```text
Use least-privilege RabbitMQ permissions, protected vhosts, private networking,
TLS and narrow bindings. Producers should only publish to authorized exchanges.
```

---

# 202. Routing Interview: Retry Ordering

### Why can retries break ordering?

Answer:

```text
If event 1 fails and is delayed while event 2 succeeds, completion order becomes
2 then 1. If business ordering is required, retry and consumer concurrency must
be designed around that requirement.
```

---

# 203. Routing Interview: Poison Message

### How do you prevent a poison message from causing a routing storm?

Answer:

```text
Bound retries, apply exponential backoff and jitter, track attempts and send
persistent failures to a DLQ instead of continuously requeueing them.
```

---

# 204. Routing Interview: Replay

### How do you safely replay a DLQ?

Answer:

```text
Identify and fix the root cause, validate with a small sample, replay through
the intended routing contract, rate-limit the replay and rely on idempotent
consumers to protect against duplicate business effects.
```

---

# 205. Routing Interview: Global vs Domain

### Global exchange or domain exchanges?

Answer:

```text
A global exchange can simplify publication but can become a large blast radius.
Domain exchanges improve ownership, isolation and routing clarity. I choose
based on scale, security, ownership and routing complexity.
```

---

# 206. Routing Interview: Wildcard Design

### How do you choose between order.* and order.#?

Answer:

```text
I choose based on the event taxonomy. If exactly one word follows order is
valid, order.* is more restrictive. If deeper subcategories are intentionally
part of the contract, order.# may be appropriate.
```

---

# 207. Routing Interview: Security Incident

### A tenant received another tenant's event.

Answer:

```text
Contain the route, restrict the affected binding, audit the affected messages,
verify permissions and tenant routing, identify the topology change that caused
the exposure, and validate isolation before reopening traffic.
```

---

# 208. Routing Interview: Senior Design Structure

For a senior system-design answer:

```text
Requirements
   |
Event/command semantics
   |
Exchange type
   |
Routing-key taxonomy
   |
Bindings
   |
Queue isolation
   |
Failure handling
   |
Idempotency
   |
Observability
   |
Security
   |
Capacity
   |
DR
```

---

# 209. Production Routing Checklist

```text
[ ] exchange selected correctly
[ ] exchange type documented
[ ] routing-key taxonomy documented
[ ] bindings documented
[ ] queue ownership documented
[ ] unroutable behavior defined
[ ] mandatory publishing considered
[ ] publisher confirms considered
[ ] alternate exchange considered
[ ] DLX defined
[ ] retry defined
[ ] idempotency defined
[ ] ordering requirement defined
[ ] fanout factor calculated
[ ] broad wildcard reviewed
[ ] security reviewed
[ ] topology as code
[ ] contract tests
[ ] synthetic routing test
[ ] observability
[ ] alerting
[ ] capacity plan
[ ] DR plan
[ ] rollback plan
```

---

# 210. Routing Change Checklist

Before production:

```text
[ ] identify affected exchanges
[ ] identify affected routing keys
[ ] identify matched queues
[ ] estimate additional traffic
[ ] estimate fanout
[ ] check consumer capacity
[ ] check storage
[ ] check network
[ ] review permissions
[ ] test positive route
[ ] test negative route
[ ] prepare rollback
```

---

# 211. Routing Incident Checklist

```text
[ ] confirm business impact
[ ] identify first bad event
[ ] check producer
[ ] check exchange
[ ] check vhost
[ ] check routing key
[ ] check binding
[ ] check queue
[ ] check consumer
[ ] check confirms
[ ] check mandatory returns
[ ] check DLQ
[ ] check recent topology changes
[ ] recover
[ ] reconcile
[ ] document root cause
```

---

# 212. Routing Golden Rules

```text
1. Routing is a contract.
2. Treat routing keys as API metadata.
3. Select exchange type based on semantics.
4. Use direct for exact routing.
5. Use topic for pattern routing.
6. Use fanout for broadcast.
7. Use headers only when metadata routing is justified.
8. Understand the default exchange.
9. Keep routing keys predictable.
10. Keep bindings narrow where possible.
11. Avoid accidental '#' subscriptions.
12. Calculate fanout amplification.
13. Distinguish routing duplication from redelivery.
14. Detect unroutable messages.
15. Understand mandatory publishing.
16. Use publisher confirms for publication reliability.
17. Do not confuse confirms with consumer processing.
18. Use alternate exchanges for explicit unroutable fallback when appropriate.
19. Use DLX for queue-level dead lettering.
20. Design retries separately from initial routing.
21. Make consumers idempotent.
22. Define ordering explicitly.
23. Use business-key sharding when needed.
24. Avoid unnecessary exchange chains.
25. Isolate critical workloads.
26. Apply least privilege.
27. Protect tenant boundaries.
28. Manage topology as code.
29. Test routing regressions.
30. Monitor business-level routing SLOs.
31. Capacity-plan fanout.
32. Test failure and replay.
33. Keep migration paths reversible.
34. Remove obsolete bindings.
35. Document ownership.
```

---

# 213. Final Routing Mental Model

```text
                         PRODUCER
                            |
                  exchange + routing key
                            |
                            v
                     +-------------+
                     |   EXCHANGE  |
                     +-------------+
                            |
                    routing algorithm
                            |
          +-----------------+-----------------+
          |                 |                 |
       Binding           Binding           Binding
          |                 |                 |
          v                 v                 v
       Queue A            Queue B            Queue C
          |                 |                 |
      Consumer A        Consumer B        Consumer C
```

For topic routing:

```text
order.created
order.updated
order.cancelled
        |
     orders.events
        |
   +----+----+
   |         |
order.*   order.#
   |         |
Queue A   Queue B
```

For unroutable protection:

```text
                 Primary Exchange
                       |
             +---------+---------+
             |                   |
          matched             no match
             |                   |
           Queue            Alternate Exchange
                                 |
                            Unrouted Queue
```

For production reliability:

```text
Publication
    |
Publisher Confirm
    |
Routing Verification
    |
Queue
    |
Consumer
    |
Idempotent Processing
    |
Business Effect
```

The senior-level question is therefore not:

```text
"How does RabbitMQ route messages?"
```

It is:

```text
"How do I design a routing topology that remains correct,
observable, secure, scalable and recoverable when traffic,
consumers, dependencies, deployments and failure domains change?"
```

That is the production RabbitMQ routing mindset.

---

# 214. Section Progression

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

---