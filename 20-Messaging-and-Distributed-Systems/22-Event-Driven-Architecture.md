# 20-Messaging-and-Distributed-Systems

# 22-Event-Driven-Architecture

> Deep, production-oriented notes covering event-driven architecture, Kafka/RabbitMQ patterns, reliability, consistency, retries, schemas, security, observability, Kubernetes, DR and senior-level design.

# 1. Event-Driven Architecture Fundamentals

Event-Driven Architecture (EDA) is an architectural style in which components communicate by producing and consuming events. An event represents something that happened in a domain or system.

```text
Producer
   |
   | Event
   v
Event Broker
   |
   +----> Consumer A
   +----> Consumer B
   +----> Consumer C
```

Unlike a tightly coupled request/response design, producers do not necessarily need to know which consumers will process an event. This separation enables independent scaling, asynchronous processing and easier integration between bounded systems.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 2. Event vs Message vs Command

These concepts should not be treated as interchangeable.

- Event: states that something happened.
- Command: asks a component to perform an action.
- Message: a general transport envelope that may contain an event or command.

Example:

```text
OrderPlaced        -> event
CreateShipment     -> command
Kafka record       -> transport message
```

A strong event name normally describes a completed fact rather than an instruction.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 3. Why Use EDA

EDA is valuable when systems need asynchronous workflows, loose coupling, fan-out, independent scaling, integration between services, or durable event history.

Typical benefits:
- loose coupling
- asynchronous processing
- independent consumer scaling
- event replay
- fan-out
- resilience to temporary consumer outages
- integration across teams and platforms

EDA also introduces complexity around delivery, ordering, duplication, schema evolution and observability.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 4. Synchronous vs Event-Driven

Synchronous:

```text
Service A -> HTTP -> Service B -> response
```

Event-driven:

```text
Service A -> Event Broker -> Service B
                         -> Service C
                         -> Service D
```

Synchronous calls are often appropriate when an immediate response is required. Events are useful when downstream work can happen asynchronously or when multiple consumers need the same business fact.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 5. Asynchronous Communication

Asynchronous communication decouples the producer's execution from consumer processing. The producer can publish an event and continue without waiting for every consumer.

This improves resilience but changes failure handling. A successful publish does not mean every downstream business operation has completed.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 6. Event Producer

A producer creates and publishes an event. Production responsibilities include:
- correct event semantics
- stable schema
- unique event identity
- appropriate key
- timestamps
- trace/correlation metadata
- delivery configuration
- error handling
- observability

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 7. Event Consumer

A consumer subscribes to events and performs business or technical processing. Production consumers should be:
- idempotent
- observable
- retry-aware
- able to handle duplicates
- tolerant of expected schema evolution
- designed for graceful shutdown
- horizontally scalable when required

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 8. Event Broker

A broker receives, stores, routes and delivers messages/events. Kafka and RabbitMQ provide different messaging models.

Kafka is optimized around durable distributed logs, partitions, consumer groups and replay. RabbitMQ emphasizes exchanges, queues, routing and message delivery patterns.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 9. Event Flow

A typical production flow is:

```text
Business Service
      |
      v
Event Producer
      |
      v
Kafka Topic
      |
      +------> Inventory Consumer
      |
      +------> Payment Consumer
      |
      +------> Notification Consumer
      |
      +------> Analytics Consumer
```

Each consumer can evolve and scale independently while consuming the same event stream.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 10. Pub/Sub

Publish/subscribe allows multiple subscribers to receive a published event. In Kafka, multiple consumer groups can independently consume the same topic.

This is different from multiple consumers within one consumer group, where partitions are distributed among group members for parallel processing.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 11. Point-to-Point

Point-to-point messaging generally means one logical consumer group processes each message once from the perspective of the queue/group. RabbitMQ queues are a common implementation.

Kafka consumer groups provide a related workload-sharing pattern while retaining Kafka's log-based storage semantics.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 12. Event Fan-Out

Fan-out occurs when one event drives multiple independent consumers.

```text
OrderPlaced
    |
    +--> Inventory
    +--> Payment
    +--> Email
    +--> Analytics
    +--> Fraud
```

Fan-out reduces direct service-to-service dependencies but increases the importance of schema contracts and consumer isolation.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 13. Loose Coupling

EDA reduces temporal and implementation coupling. A producer can publish a stable event contract without knowing how consumers implement their processing.

However, events still create contract coupling. Poorly designed schemas can create hidden dependencies across many teams.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 14. Temporal Decoupling

A producer can publish while a consumer is temporarily unavailable, provided the broker retains the event long enough.

This is one of the major operational advantages of durable messaging over direct synchronous calls.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 15. Spatial Decoupling

The producer does not need a direct network connection to every consumer. The broker becomes the communication boundary.

This can reduce cascading failures caused by synchronous downstream dependencies.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 16. Domain Events

A domain event describes a meaningful business fact:

```text
CustomerRegistered
OrderPlaced
PaymentCaptured
ShipmentDispatched
```

The event should contain enough information for consumers to perform their responsibility without forcing them to synchronously call the producer for every field.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 17. Integration Events

An integration event is designed to communicate a domain fact across service or organizational boundaries. It should have a deliberately governed schema and compatibility policy.

Do not expose unstable internal database structures as integration contracts.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 18. Event Naming

Prefer past-tense business facts:

```text
OrderPlaced
PaymentAuthorized
InventoryReserved
ShipmentCreated
```

Avoid ambiguous names such as `OrderEvent` when the semantic meaning matters operationally.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 19. Event Envelope

A production event commonly contains:

```json
{
  "event_id": "uuid",
  "event_type": "OrderPlaced",
  "event_version": 1,
  "occurred_at": "timestamp",
  "producer": "order-service",
  "correlation_id": "id",
  "trace_id": "id",
  "key": "order-123",
  "data": {}
}
```

The exact envelope should be standardized by the platform or organization.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 20. Event ID

A globally unique event ID helps consumers detect duplicates, trace processing and investigate incidents.

It should remain stable when the same event is retried.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 21. Correlation ID

A correlation ID connects multiple operations belonging to the same business transaction.

Example:

```text
API request
  -> OrderPlaced
  -> PaymentAuthorized
  -> ShipmentCreated
```

The correlation ID can help reconstruct the end-to-end workflow during troubleshooting.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 22. Trace Context

Distributed tracing metadata can connect producer and consumer spans. Propagate trace context through the event envelope according to the organization's tracing standard.

This enables observability across asynchronous boundaries.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 23. Event Payload Design

Include information consumers genuinely need. Avoid both extremes:
- tiny events that force synchronous lookups everywhere
- huge events containing unrelated internal data

Payload design should consider bandwidth, storage, schema evolution, privacy and consumer independence.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 24. Event Key

Keys often determine partition placement in Kafka. A stable business key can preserve ordering for that entity.

Example:

```text
key = order_id
```

All events for the same order can then be routed consistently to a partition, subject to Kafka's partitioning configuration.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 25. Ordering

EDA does not automatically provide global ordering. Ordering must be defined at the appropriate scope.

For Kafka, partition-level ordering is the important baseline. If order-specific sequencing matters, use a suitable stable key so related events are assigned consistently.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 26. Delivery Semantics

Common delivery semantics are:
- at-most-once
- at-least-once
- effectively-once through idempotent processing
- exactly-once in supported transactional processing scenarios

Application correctness must not depend on assuming duplicates never occur.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 27. At-Most-Once

At-most-once processing favors avoiding duplicates at the cost of possible loss. It can be appropriate for non-critical telemetry where occasional loss is acceptable.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 28. At-Least-Once

At-least-once processing prioritizes not losing messages but can produce duplicates.

Production consumers should therefore implement idempotency.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 29. Exactly-Once

Exactly-once semantics are narrower than the phrase suggests. Kafka can provide transactional guarantees for specific processing patterns, but external side effects such as email, payment APIs or databases require additional design.

Never claim that a Kafka transaction automatically makes an entire distributed business workflow exactly once.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 30. Idempotency

An idempotent consumer produces the same intended business result when processing the same event multiple times.

Typical techniques:
- event ID table
- business-key uniqueness
- conditional database update
- state transition validation
- idempotency key
- transactional inbox pattern

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 31. Inbox Pattern

The consumer stores the received event ID in the same database transaction as its business change.

```text
Event
 |
 v
DB transaction
 |-- process business change
 |-- record event_id
 |
commit
```

If the same event arrives again, the stored event ID prevents duplicate business effects.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 32. Outbox Pattern

The transactional outbox solves the producer-side dual-write problem:

```text
Application transaction
 |-- update business DB
 |-- insert event into outbox
 |
commit
        |
        v
Outbox publisher / CDC
        |
        v
Kafka
```

The database transaction makes the business update and event creation atomic from the application's perspective.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 33. Dual-Write Problem

A dangerous pattern is:

```text
update database
publish Kafka event
```

If the database commit succeeds but publishing fails, state changes without an event. If publishing succeeds but the database commit fails, consumers see an event for a transaction that did not persist.

Use an outbox or an architecture specifically designed to solve this consistency boundary.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 34. Eventual Consistency

EDA frequently creates eventual consistency. A downstream service may temporarily show old state while consuming and processing events.

Business processes must define acceptable consistency windows and user-facing behavior.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 35. Eventual Consistency Trade-Off

Advantages:
- resilience
- decoupling
- scalability

Costs:
- stale reads
- asynchronous failure
- harder debugging
- duplicate processing
- ordering concerns
- more complex testing

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 36. Saga Pattern

A saga coordinates a distributed business workflow without requiring one global database transaction.

```text
OrderCreated
   -> Payment
   -> Inventory
   -> Shipment
```

If a later step fails, compensating actions may be required.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 37. Choreography Saga

In choreography, services react to events without a central coordinator.

```text
OrderPlaced -> Payment
PaymentCaptured -> Inventory
InventoryReserved -> Shipment
```

It is decentralized but can become difficult to understand when workflows grow large.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 38. Orchestration Saga

An orchestrator coordinates commands and observes events.

```text
Saga Orchestrator
 |--> Payment
 |--> Inventory
 |--> Shipment
```

Orchestration provides centralized workflow visibility but introduces another important component.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 39. Compensation

Distributed transactions often require compensating actions rather than rollback.

Example:

```text
Payment captured
Inventory reservation fails
        |
        v
Issue refund
```

Compensation must itself be reliable and idempotent.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 40. Retry

Transient failures should be retried carefully. Retry design needs:
- maximum attempts
- exponential backoff
- jitter
- classification of transient vs permanent errors
- retry topic/queue strategy
- observability
- dead-letter handling

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 41. Retry Storm

Immediate retries can amplify an outage:

```text
Consumer failure
 -> retry
 -> retry
 -> retry
 -> downstream overloaded
 -> more failures
```

Use bounded retries and backoff. Protect downstream dependencies.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 42. Dead-Letter Events

A dead-letter topic or queue stores messages that cannot be processed after the defined retry policy.

DLQs require ownership, retention, alerting, replay procedures and security controls. A DLQ is not a place to permanently hide failures.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 43. Poison Message

A poison message repeatedly fails processing because of malformed data, incompatible schema, invalid business state or a deterministic application bug.

Do not allow one poison message to permanently block an entire partition when the business workflow permits isolation.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 44. Backpressure

Backpressure occurs when producers generate work faster than consumers can process it.

Monitor:
- producer rate
- consumer throughput
- consumer lag
- processing latency
- broker capacity

Scale consumers when partition parallelism permits it; otherwise address the partitioning or processing bottleneck.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 45. Consumer Lag

Consumer lag represents unprocessed backlog. In production, monitor both lag count and lag age.

A backlog of 1 million messages may be harmless for a very fast consumer or dangerous for a slow consumer near retention expiry.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 46. Replay

Durable event streams allow replay for:
- rebuilding projections
- recovering failed consumers
- backfilling new consumers
- correcting processing logic

Replay must be controlled to prevent duplicate external side effects.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 47. Replay Safety

Before replaying:
- confirm consumer idempotency
- isolate or protect external side effects
- determine offset range
- understand schema versions
- estimate processing load
- monitor downstream dependencies
- communicate expected impact

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 48. Event Versioning

Schemas evolve. Additive changes are usually easier to make compatible than breaking field changes.

Versioning can be represented in the event type, schema metadata, envelope or registry strategy.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 49. Schema Registry

A schema registry can centralize event schemas and compatibility rules. It helps prevent producers from publishing incompatible changes.

The registry is part of the event platform and should have availability, security, backup and governance plans.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 50. Backward Compatibility

Backward compatibility means newer consumers can handle older producer data or the selected compatibility direction.

Always define the compatibility contract explicitly instead of assuming all consumers upgrade simultaneously.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 51. Contract Testing

Consumer-driven or producer contract testing can detect breaking event changes before production deployment.

Tests should cover required fields, data types, enum evolution and semantic compatibility.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 52. Event Governance

An enterprise event platform needs ownership:
- event owner
- schema owner
- producer
- consumers
- retention policy
- classification
- compatibility policy
- deprecation date
- documentation

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 53. Event Catalog

An event catalog provides searchable information about events, schemas, producers, consumers and ownership.

Without discoverability, teams frequently create duplicate events or consume undocumented internal topics.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 54. Topic Design

Kafka topic boundaries should reflect operational and business needs. Avoid one enormous topic containing unrelated event types unless the architecture deliberately supports it.

Consider:
- throughput
- retention
- security
- ownership
- partitioning
- consumer patterns
- schema lifecycle

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 55. One Event Per Topic vs Multi-Event Topics

Both patterns can be valid.

Dedicated topics can simplify retention, ownership and consumer behavior. Multi-event topics can reduce topic proliferation but require filtering and schema discipline.

Choose based on operational boundaries rather than a universal rule.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 56. Partition Strategy

Partitioning determines scalability and ordering boundaries.

Choose partition keys based on:
- required ordering
- load distribution
- consumer parallelism
- hot-key risk
- expected growth

Never choose a key only because it is convenient in application code.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 57. Hot Partition

A poor key can send disproportionate traffic to one partition.

Example:

```text
99% traffic -> key A -> partition 0
1% traffic  -> other partitions
```

This can create a throughput bottleneck even when the cluster has unused capacity.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 58. Consumer Groups

A consumer group represents one logical processing application. Kafka distributes partitions among members.

If a topic has fewer partitions than desired consumer instances, some instances remain idle.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 59. Independent Consumers

Different business applications should normally use separate consumer groups when each must receive all relevant events.

```text
Topic
 |--> payment-group
 |--> inventory-group
 |--> analytics-group
```

One group is for parallelism within the same logical application, not for unrelated applications.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 60. Event Ordering Across Services

A producer may preserve order within a Kafka partition, but asynchronous consumers can process events at different speeds.

Business workflows must define whether strict sequencing is actually required and where it must be enforced.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 61. Event Time vs Processing Time

`occurred_at` describes when the business event happened. Processing time describes when a consumer handled it.

Late events make this distinction important for analytics, windows and business logic.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 62. Duplicate Events

Duplicates can arise from producer retries, consumer retries, network failures, transaction boundaries or reprocessing.

Design duplicate handling as a normal operating condition rather than an exceptional impossibility.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 63. Missing Events

Missing events require investigation across producer logs, broker health, acknowledgements, retention, consumer offsets and downstream processing.

Do not assume a consumer bug until publication and retention have been verified.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 64. Event Ordering vs Idempotency

Ordering and idempotency solve different problems.

- Ordering controls sequence.
- Idempotency controls repeated application.

A production consumer may need both.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 65. Timeouts

Every synchronous dependency inside an event consumer needs bounded timeouts. An unbounded database or HTTP call can stall consumer processing and cause growing lag.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 66. Circuit Breaker

Circuit breakers can prevent an unhealthy downstream dependency from consuming all consumer capacity. When combined with retry and DLQ policies, they help control cascading failures.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 67. Bulk Processing

Consumers can improve throughput through batching, but batch size affects latency, memory, failure scope and retry behavior.

A failed batch should not accidentally duplicate all successfully processed records unless processing is idempotent.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 68. Concurrency

Increase consumer concurrency only when the partition model and downstream dependencies support it. More threads do not automatically increase throughput if the topic has limited partitions or the database is the bottleneck.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 69. Database Consumers

A common pattern is:

```text
Kafka
  |
Consumer
  |
DB transaction
```

The database transaction should establish the correct relationship between event processing and business state. Consider inbox/idempotency when retries are possible.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 70. External API Consumers

External APIs create an especially difficult exactly-once boundary. A consumer can send a successful request and crash before recording completion.

Use idempotency keys supported by the external API where possible.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 71. Event-Driven Notifications

Notifications are good candidates for asynchronous processing. However, duplicate email/SMS/push delivery may be user-visible.

Store a notification idempotency key or use a provider-side idempotency feature where available.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 72. Event-Driven Payments

Payment workflows require stronger correctness controls. Never rely solely on Kafka delivery semantics to guarantee a payment operation occurs exactly once.

Use payment-provider idempotency, durable transaction state, reconciliation and carefully designed event processing.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 73. Event-Driven Inventory

Inventory changes often require ordering and idempotency by SKU or inventory entity. Concurrent events can produce overselling if state transitions are not atomically controlled.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 74. Event-Driven Search Indexing

Search indexes are commonly eventual-consistency projections. Kafka replay can rebuild an index after schema or application changes.

Consumers should tolerate duplicate indexing operations.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 75. Event-Driven Cache Invalidation

Events can invalidate or update caches asynchronously. Consumers should tolerate missed or reordered invalidation events through periodic reconciliation or authoritative database refresh where required.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 76. Event Sourcing

Event sourcing stores business state transitions as events and rebuilds current state by replaying them.

This makes event retention, schema compatibility, ordering and replay performance architectural requirements rather than optional operational features.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 77. CQRS

CQRS separates write and read models. Events can update read projections asynchronously.

```text
Command -> Write Model
              |
              v
            Event
              |
       +------+------+
       v             v
 Read Model A     Read Model B
```

Projection rebuild is a major operational consideration.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 78. Eventual Consistency in CQRS

Immediately after a write, a read model may not yet reflect the change. User interfaces should handle this intentionally rather than presenting stale data as an unexpected system defect.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 79. Transactional Boundaries

Define which operations must be atomic. A distributed workflow should not pretend that separate databases and Kafka form one ACID transaction unless the exact technology guarantees support that claim.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 80. Exactly-Once Business Effect

A practical design often achieves an effectively-once business effect using:

```text
at-least-once delivery
+
idempotent consumer
+
atomic state transition
+
reconciliation
```

This is frequently more realistic than trying to make every external interaction globally exactly once.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 81. Event Security

Secure events in transit and at rest. Apply authentication and authorization at the broker, restrict topic access, protect credentials and avoid exposing sensitive data unnecessarily.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 82. Sensitive Data in Events

Events can persist for days or years. Do not place secrets, unnecessary personal data or credentials into durable event streams.

Data minimization should be part of schema design.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 83. Encryption

Use TLS for network protection and storage encryption according to platform requirements. Encryption does not replace authorization or data minimization.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 84. Access Control

Authorize producers and consumers by topic and operation. Separate administrative privileges from application privileges.

A consumer that only needs to read one topic should not have broad cluster permissions.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 85. Event Observability

Production observability should answer:

```text
Who produced the event?
When?
Which topic?
Which partition?
Which event ID?
Which consumer group?
Which processing attempt?
What downstream operation happened?
Did it fail or retry?
```

Correlation and trace metadata make this possible.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 86. Metrics

Useful EDA metrics include:
- producer throughput
- publish failures
- consumer throughput
- lag and lag age
- processing latency
- retry count
- DLQ rate
- duplicate rate
- event processing errors
- downstream dependency latency

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 87. Distributed Tracing

Trace asynchronous workflows across producer and consumer boundaries. Include trace IDs in event metadata and create consumer spans around message processing.

This turns a multi-service workflow into an observable causal chain.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 88. Logging

Log structured fields:
- event_id
- event_type
- correlation_id
- trace_id
- topic
- partition
- offset
- consumer_group
- processing_status

Never log sensitive payload fields merely for convenience.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 89. Alerting

Alert on business-impacting symptoms:
- sustained consumer lag
- lag age near retention
- high processing failures
- retry spikes
- DLQ growth
- broker errors
- producer failures
- downstream dependency failures

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 90. Disaster Recovery

DR for EDA must consider both event data and application state.

Questions:
- Can events be recovered?
- Is the required history retained?
- Can schemas be restored?
- Can consumers restart safely?
- Are offsets recoverable or reconstructable?
- Can downstream databases be reconciled?
- How is producer/consumer traffic redirected?

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 91. Multi-Region EDA

Multi-region event architecture adds replication, latency, ordering and conflict considerations.

Choose active/passive or active/active deliberately. Active/active requires especially careful ownership and idempotency design.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 92. Event Replay During DR

Replay can rebuild downstream projections after recovery, but replaying business side effects such as payments or notifications may be unsafe.

Separate state-rebuilding consumers from irreversible side-effect consumers when possible.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 93. Kubernetes EDA Deployment

A production Kubernetes architecture commonly contains:

```text
Ingress/API
   |
Producer Services
   |
Kafka cluster
   |
+--+-------------+-------------+
|                |             |
Consumers     Consumers     Stream processors
|                |             |
DB             APIs          Search/Analytics
```

Use Kubernetes for workload lifecycle while Kafka remains the durable event platform.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 94. Autoscaling Consumers

Consumer workloads can scale based on CPU, processing latency or lag-related metrics. Scaling must respect partition count and downstream capacity.

More Pods than partitions cannot create additional Kafka partition parallelism.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 95. Graceful Consumer Shutdown

Consumers should stop accepting new work, complete or safely abandon in-flight processing, commit offsets only at the correct point and close connections.

Incorrect shutdown handling can cause duplicates or message loss depending on the delivery model.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 96. Production EDA Failure Model

Assume:
- producer retries
- broker restarts
- consumer crashes
- duplicate delivery
- delayed events
- out-of-order events across entities
- downstream timeouts
- schema evolution
- DLQ growth
- network partitions
- replay requirements

A production architecture is one that remains correct under these conditions.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 97. Anti-Patterns

Avoid:
- synchronous calls everywhere disguised as EDA
- one giant topic for unrelated domains
- no event ownership
- no schema governance
- no idempotency
- infinite retries
- unbounded DLQs
- logging sensitive payloads
- treating Kafka as a database without a deliberate model
- assuming exactly-once solves external side effects

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 98. Production Architecture Pattern

A mature platform can look like:

```text
                 API / Services
                       |
                 Outbox / Producer
                       |
                       v
                 Kafka Cluster
                       |
        +--------------+--------------+
        |              |              |
     Domain A       Domain B       Analytics
     consumer       consumer       consumer
        |              |              |
       DB             DB          Data Platform

Platform:
  Schema Registry
  Observability
  Security
  DLQ/Retry
  GitOps
  DR/Replication
```

The architecture separates business processing from platform concerns while preserving clear ownership.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 99. Production Readiness

Before production, validate:
- event semantics
- schema compatibility
- partition strategy
- retention
- idempotency
- retry policy
- DLQ handling
- observability
- security
- consumer scaling
- replay
- DR
- ownership
- runbooks
- failure testing

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# 100. Senior Engineering Mental Model

The most important EDA question is not "How do I publish to Kafka?"

It is:

```text
What business fact occurred?
Who needs it?
How long must it exist?
What happens if delivery is duplicated?
What happens if processing fails?
What happens if the consumer is offline?
Can it be replayed?
Can schemas evolve?
Can the business state be reconciled?
Can the entire workflow be observed?
What happens during disaster recovery?
```

That mindset separates a basic message-driven implementation from production-grade event architecture.

## Production Questions

```text
- What happens when the producer retries?
- What happens when the consumer crashes?
- What happens when the same event arrives twice?
- What happens when the consumer is offline?
- What happens when the downstream database is unavailable?
- Can the event be replayed safely?
- How is schema compatibility enforced?
- How is the workflow observed end-to-end?
- What happens during disaster recovery?
```

# Production Scenario 1: Producer publishes an event but the database transaction fails

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 2: Database commits but event publishing fails

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 3: Consumer crashes after the database update

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 4: Consumer receives the same event three times

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 5: Consumer is offline for two days

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 6: Consumer lag approaches retention

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 7: Downstream payment API times out after accepting the request

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 8: Notification consumer sends duplicate messages

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 9: One Kafka partition becomes a hot partition

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 10: One event repeatedly fails processing

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 11: Retry traffic overwhelms a downstream service

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 12: DLQ grows continuously

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 13: New event schema breaks an old consumer

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 14: New consumer needs historical replay

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 15: Projection rebuild must not send duplicate payments

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 16: Two events for the same order arrive out of sequence

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 17: Inventory receives duplicate reservation events

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 18: Search index falls behind Kafka

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 19: Kafka broker becomes unavailable

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 20: Consumer group has more consumers than partitions

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 21: Consumer processing is slower than producer throughput

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 22: Database connection pool becomes exhausted

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 23: Event contains sensitive information

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 24: Trace is lost between producer and consumer

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 25: One region becomes unavailable

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 26: DR replay would repeat external side effects

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 27: Operator restarts consumer Pods during processing

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 28: Autoscaler scales consumers beyond useful parallelism

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 29: Schema registry becomes unavailable

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Production Scenario 30: Business team asks for exactly-once processing

## Incident / Design Response

```text
1. Establish the business impact.
2. Identify the event, producer, consumer group and processing stage.
3. Determine whether the event exists in the broker.
4. Check delivery and acknowledgement behavior.
5. Check consumer offset and processing state.
6. Check downstream transaction state.
7. Determine whether the operation is idempotent.
8. Prevent retry amplification.
9. Preserve recoverable data.
10. Apply the smallest safe remediation.
11. Validate business state, not only infrastructure health.
12. Record the root cause and preventive control.
```

## Senior Reasoning

The infrastructure symptom is only one part of an event-driven incident. A healthy Kafka broker does not prove that a business operation completed, and a failed consumer does not prove that the event was lost. Trace the event from producer transaction through broker storage, consumer offset, business transaction and downstream side effects.

# Senior-Level Interview Preparation

## 1. What is Event-Driven Architecture?

An architecture where components communicate through events representing facts or state changes, usually asynchronously through a broker or event platform.

## 2. Why use EDA instead of synchronous calls?

EDA can reduce temporal and spatial coupling, support fan-out, buffer outages, enable independent scaling and provide durable replay.

## 3. What is the biggest disadvantage?

Operational and application complexity around eventual consistency, ordering, retries, duplicates, schemas, observability and failure handling.

## 4. What is an event?

A durable representation of something that happened, such as OrderPlaced or PaymentCaptured.

## 5. Event vs command?

An event describes a fact; a command requests an action.

## 6. What is at-least-once delivery?

The system prioritizes avoiding loss, so the same event may be delivered more than once.

## 7. How do you handle duplicates?

Use idempotency through event IDs, business keys, uniqueness constraints, inbox processing or equivalent controls.

## 8. What is the outbox pattern?

It stores the business change and event record in the same database transaction, then publishes the event asynchronously.

## 9. Why is dual-write dangerous?

Database and broker operations are separate transactions, so one can succeed while the other fails.

## 10. What is the inbox pattern?

The consumer records processed event identity atomically with its business transaction to prevent repeated effects.

## 11. How do you handle poison messages?

Bound retries, isolate failed events through retry/DLQ mechanisms, alert and provide controlled replay.

## 12. How do you prevent retry storms?

Use bounded retries, exponential backoff, jitter, circuit breakers and downstream protection.

## 13. What is a saga?

A distributed workflow composed of local transactions coordinated through events and/or commands, with compensation for failures.

## 14. Choreography vs orchestration?

Choreography relies on decentralized event reactions; orchestration uses a central coordinator.

## 15. How do you guarantee event ordering?

Define the required ordering scope and, in Kafka, use a suitable partition key so related events share a partition.

## 16. Does Kafka guarantee global ordering?

No. Kafka provides ordering within a partition.

## 17. Why is idempotency still required if Kafka has transactions?

External side effects and systems outside the Kafka transaction boundary can still be repeated.

## 18. How do you design event schemas?

Use stable business semantics, explicit ownership, compatibility rules, versioning and consumer contract testing.

## 19. Why use a schema registry?

To centrally manage schemas and enforce compatibility rules.

## 20. How do you safely replay events?

Verify idempotency, isolate irreversible side effects, select the required offset range, control load and monitor downstream systems.

## 21. What is consumer lag?

The backlog between the producer's available data and the consumer's processing position.

## 22. Why monitor lag age as well as lag count?

A large count may be harmless if processing is fast, while a smaller backlog can be dangerous if events are close to retention expiry.

## 23. What is eventual consistency?

A state model where distributed components converge asynchronously rather than becoming immediately consistent in one transaction.

## 24. How do you handle external API exactly-once requirements?

Use provider-supported idempotency keys, durable state, reconciliation and carefully designed retry behavior.

## 25. What makes EDA production-grade?

Clear event contracts, durable delivery, idempotency, bounded retries, DLQs, observability, security, replay, capacity planning and tested DR.

# Production Checklist

```text
EVENT DESIGN
[ ] event semantics defined
[ ] event owner defined
[ ] event type stable
[ ] event ID included
[ ] correlation ID included
[ ] trace context included
[ ] schema version defined
[ ] sensitive data minimized

KAFKA / BROKER
[ ] topic ownership defined
[ ] partition strategy validated
[ ] ordering requirement defined
[ ] retention defined
[ ] replication designed
[ ] consumer groups documented

RELIABILITY
[ ] delivery semantics defined
[ ] idempotency implemented
[ ] outbox/inbox evaluated
[ ] retry policy bounded
[ ] exponential backoff
[ ] jitter
[ ] DLQ policy
[ ] replay procedure

APPLICATION
[ ] downstream timeouts
[ ] circuit breaker where appropriate
[ ] transaction boundaries defined
[ ] duplicate handling tested
[ ] out-of-order behavior tested
[ ] eventual consistency accepted by business

SECURITY
[ ] TLS
[ ] authentication
[ ] authorization
[ ] secrets protected
[ ] sensitive fields minimized
[ ] retention reviewed for data exposure

OBSERVABILITY
[ ] producer metrics
[ ] consumer metrics
[ ] lag
[ ] lag age
[ ] processing latency
[ ] retry rate
[ ] DLQ rate
[ ] event IDs in logs
[ ] distributed tracing
[ ] alerts

OPERATIONS
[ ] replay tested
[ ] poison event tested
[ ] consumer outage tested
[ ] broker failure tested
[ ] schema compatibility tested
[ ] DR tested
[ ] runbooks documented
[ ] ownership documented
```

# 100 Golden Rules

1. Design events around business facts.
2. Do not confuse events with commands.
3. Prefer meaningful event names.
4. Define event ownership.
5. Define consumer ownership.
6. Treat schemas as APIs.
7. Version deliberately.
8. Use compatibility rules.
9. Use contract testing.
10. Do not expose unstable database internals as contracts.
11. Include a unique event ID.
12. Use correlation IDs.
13. Propagate trace context.
14. Minimize sensitive data.
15. Assume duplicates can occur.
16. Make important consumers idempotent.
17. Use the outbox pattern for dual-write problems.
18. Use inbox processing when appropriate.
19. Do not claim global exactly-once casually.
20. Define transaction boundaries.
21. Use bounded retries.
22. Use exponential backoff.
23. Use jitter.
24. Protect downstream dependencies.
25. Do not create retry storms.
26. Use DLQs for isolated failures.
27. Alert on DLQ growth.
28. Provide controlled replay.
29. Do not let poison messages block healthy traffic unnecessarily.
30. Monitor lag age.
31. Monitor processing latency.
32. Partition based on ordering and scale requirements.
33. Avoid hot partitions.
34. Do not add consumers beyond useful partition parallelism.
35. Use separate consumer groups for independent applications.
36. Define ordering scope explicitly.
37. Do not assume cross-partition ordering.
38. Treat eventual consistency as a business design choice.
39. Use sagas for distributed workflows where appropriate.
40. Design compensating actions.
41. Do not pretend multiple databases form one ACID transaction.
42. Use provider idempotency for external APIs.
43. Reconcile critical side effects.
44. Make notification processing idempotent.
45. Make payment processing especially defensive.
46. Protect inventory state transitions.
47. Use replay for projections when appropriate.
48. Separate replayable state rebuilds from irreversible side effects.
49. Test replay before production incidents.
50. Define retention from business replay requirements.
51. Retention is not backup.
52. DR is not just Kubernetes YAML.
53. Monitor producers and consumers.
54. Monitor broker health.
55. Monitor downstream dependencies.
56. Trace asynchronous boundaries.
57. Log event IDs, not unnecessary payloads.
58. Use structured logs.
59. Alert on meaningful business-impacting symptoms.
60. Secure producer and consumer access.
61. Use least privilege.
62. Protect credentials.
63. Encrypt data in transit.
64. Review encryption and retention together.
65. Avoid unbounded event growth.
66. Classify event data.
67. Define deprecation procedures.
68. Catalog important events.
69. Avoid undocumented topics.
70. Avoid one giant topic for unrelated domains.
71. Choose topic boundaries deliberately.
72. Measure consumer throughput.
73. Measure producer throughput.
74. Measure retry rate.
75. Measure duplicate rate when possible.
76. Test consumer crashes.
77. Test producer retries.
78. Test downstream timeouts.
79. Test schema evolution.
80. Test long consumer outages.
81. Test broker failures.
82. Test partition skew.
83. Test DR.
84. Test compensation workflows.
85. Test replay under load.
86. Test DLQ recovery.
87. Test certificate and credential rotation.
88. Use Kubernetes autoscaling carefully.
89. Respect partition limits.
90. Do not equate Pod count with Kafka parallelism.
91. Use graceful consumer shutdown.
92. Commit offsets at the correct point.
93. Do not acknowledge before durable business processing unless the semantics permit it.
94. Do not process external side effects without idempotency.
95. Correlate infrastructure and business telemetry.
96. Build runbooks.
97. Assign owners.
98. Review event contracts regularly.
99. Treat EDA as a system, not merely a Kafka producer and consumer.
100. Production correctness matters more than message throughput alone.

# END OF 22-Event-Driven-Architecture.md
