# 30 — Messaging Projects

## 1. Purpose

This chapter converts the messaging theory from the previous chapters into hands-on, production-oriented DevOps projects.

The projects progress from fundamentals to senior-level architecture.

Primary technologies:

- RabbitMQ
- Kafka
- Kubernetes
- Docker
- AWS
- Prometheus
- Grafana
- Elasticsearch
- Logstash
- Kibana
- OpenTelemetry
- Jaeger
- Argo CD
- Terraform
- CI/CD

The objective is not merely to make messages move between applications. The objective is to build systems that are:

```text
Reliable
Secure
Observable
Scalable
Recoverable
Automated
Production-ready
```

---

# PART I — Project Roadmap

## 2. Project Progression

Complete the projects in this order:

```text
Project 01
RabbitMQ Producer/Consumer
        |
Project 02
RabbitMQ Routing
        |
Project 03
RabbitMQ Retry/DLQ
        |
Project 04
RabbitMQ HA
        |
Project 05
Kafka Producer/Consumer
        |
Project 06
Kafka Consumer Groups
        |
Project 07
Kafka Ordering/Partitioning
        |
Project 08
Kafka Retry/DLQ
        |
Project 09
Idempotent Processing
        |
Project 10
Transactional Outbox
        |
Project 11
RoboShop Messaging
        |
Project 12
Kubernetes Messaging Platform
        |
Project 13
Observability
        |
Project 14
GitOps
        |
Project 15
Production Failure Testing
        |
Project 16
Multi-AZ / DR Architecture
```

---

# PART II — Project 01: RabbitMQ Producer/Consumer

## 3. Objective

Build a simple asynchronous worker system.

```text
Producer
   |
RabbitMQ
   |
Queue
   |
Consumer
```

---

## 4. Requirements

Create:

```text
producer
consumer
RabbitMQ
```

Producer publishes:

```json
{
  "job_id": "JOB-001",
  "task": "send-email"
}
```

Consumer receives and processes it.

---

## 5. Docker Setup

Use RabbitMQ in Docker for local development.

Conceptual architecture:

```text
Docker Network
 |
+-- rabbitmq
+-- producer
+-- consumer
```

---

## 6. Validation

Test:

```text
start broker
start consumer
start producer
publish message
verify consumer
```

Then stop the consumer:

```text
producer -> broker -> queue
```

Start consumer again:

```text
queue -> consumer
```

Verify pending work is processed.

---

# PART III — Project 02: RabbitMQ Routing

## 7. Objective

Implement multiple event types.

```text
order.created
order.cancelled
payment.completed
payment.failed
```

---

## 8. Exchange

Use a topic exchange:

```text
roboshop.events
```

Bindings:

```text
payment.queue
    |
payment.*

shipping.queue
    |
order.created
```

---

## 9. Test Routing

Publish:

```text
order.created
```

Verify shipping receives it.

Publish:

```text
payment.completed
```

Verify payment consumer receives it.

---

# PART IV — Project 03: Retry and DLQ

## 10. Objective

Build failure handling.

```text
Main Queue
    |
Consumer
    |
Failure
    |
Retry
    |
Failure
    |
DLQ
```

---

## 11. Failure Classification

Implement:

```text
transient failure -> retry
permanent failure -> DLQ
```

Example transient:

```text
HTTP 503
connection timeout
temporary database unavailable
```

Permanent:

```text
invalid schema
invalid business data
unsupported event
```

---

## 12. Retry Strategy

Use increasing delays:

```text
1st retry -> 5 sec
2nd retry -> 30 sec
3rd retry -> 5 min
```

These are example values, not universal production defaults.

Add jitter where appropriate.

---

## 13. DLQ Validation

Send an invalid message.

Expected:

```text
consume
 |
failure
 |
retry
 |
retry limit
 |
DLQ
```

Verify the message remains available for investigation.

---

# PART V — Project 04: RabbitMQ High Availability

## 14. Objective

Deploy RabbitMQ in a clustered architecture.

Concept:

```text
          RabbitMQ
       /     |     \
    Node1  Node2  Node3
```

Use an appropriate replicated queue architecture such as quorum queues for critical workloads.

---

## 15. Failure Test

Stop one broker node.

Validate:

```text
application reconnect
queue availability
message continuity
consumer recovery
```

Record:

```text
failure time
recovery time
messages affected
```

---

# PART VI — Project 05: Kafka Producer/Consumer

## 16. Objective

Build:

```text
Producer
   |
Kafka Topic
   |
Consumer
```

Topic:

```text
roboshop.orders
```

---

## 17. Producer

Publish:

```json
{
  "event_type": "order.created",
  "order_id": "ORD-10001"
}
```

---

## 18. Consumer

Consumer should:

```text
receive
validate
process
commit offset
```

Test consumer restart.

Verify it resumes according to its committed offset.

---

# PART VII — Project 06: Kafka Consumer Groups

## 19. Objective

Create independent consumers.

```text
                 orders
                   |
       +-----------+-----------+
       |           |           |
    billing      audit     analytics
      CG1         CG2          CG3
```

Each group receives the event independently.

---

## 20. Scaling Experiment

Create:

```text
6 partitions
```

Run:

```text
1 consumer
3 consumers
6 consumers
8 consumers
```

Measure throughput.

Expected observation:

```text
1 -> useful parallelism
3 -> more parallelism
6 -> maximum partition-level concurrency
8 -> some consumers may be idle
```

---

# PART VIII — Project 07: Kafka Partitioning and Ordering

## 21. Objective

Understand ordering.

Use:

```text
key = order_id
```

Publish:

```text
order.created
order.paid
order.shipped
order.completed
```

All events for the same order should map consistently to the same partition while the partitioning scheme remains stable.

---

## 22. Ordering Test

Create multiple orders:

```text
ORD-001
ORD-002
ORD-003
```

Publish events concurrently.

Verify:

```text
ORD-001 events preserve their partition order
ORD-002 events preserve their partition order
```

Do not assume global ordering across partitions.

---

# PART IX — Project 08: Kafka Retry and DLQ

## 23. Architecture

```text
orders.v1
    |
consumer
    |
failure
    |
retry topic
    |
consumer
    |
failure
    |
DLQ topic
```

Topics:

```text
orders.v1
orders.retry.1
orders.retry.2
orders.dlq
```

---

## 24. Retry Metadata

Include metadata such as:

```text
event_id
attempt
original_topic
original_partition
original_offset
failure_reason
failed_at
```

This makes investigation easier.

---

# PART X — Project 09: Idempotent Consumer

## 25. Objective

Prove duplicate delivery can be handled safely.

Send:

```text
event_id = EVT-001
```

twice.

Expected:

```text
first -> business operation
second -> no duplicate business effect
```

---

## 26. Idempotency Store

Example:

```text
processed_events

event_id
consumer_name
processed_at
result
```

Use a unique constraint on the relevant identity.

---

## 27. Important Race Condition

This is unsafe:

```text
check event_id
       |
not found
       |
process
       |
insert event_id
```

Two consumers or retries can race.

Prefer an atomic database mechanism or transaction that makes the idempotency decision and business update consistent.

---

# PART XI — Project 10: Transactional Outbox

## 28. Objective

Build:

```text
Application
    |
Database Transaction
    |
+-----------+-----------+
| Order     | Outbox    |
+-----------+-----------+
                  |
              Relay
                  |
               Broker
```

---

## 29. Failure Experiment

Force the application to fail:

```text
after database commit
before message publication
```

Expected:

```text
order exists
outbox exists
relay eventually publishes
```

This demonstrates why the outbox pattern is useful.

---

# PART XII — Project 11: RoboShop Messaging

## 30. Objective

Create an asynchronous checkout architecture.

```text
Frontend
   |
Checkout
   |
Order
   |
Broker
   |
+--------+---------+
|                  |
Payment          Shipping
Worker           Worker
```

---

## 31. Business Flow

Implement:

```text
order.created
payment.completed
shipping.created
order.completed
```

Failure:

```text
payment.failed
```

Compensation:

```text
order.cancelled
```

---

## 32. Add Idempotency

Each worker must safely handle duplicate events.

```text
event_id
order_id
business operation
```

should be designed so repeated delivery does not produce duplicate side effects.

---

# PART XIII — Project 12: Kubernetes Messaging Platform

## 33. Objective

Deploy the messaging workloads on Kubernetes.

Example:

```text
Namespace: messaging
 |
+-- broker
+-- producer
+-- consumer
+-- retry worker
+-- DLQ processor
```

---

## 34. Kubernetes Resources

Practice:

```text
Deployment
StatefulSet
Service
ConfigMap
Secret
ServiceAccount
Role
RoleBinding
NetworkPolicy
PodDisruptionBudget
HorizontalPodAutoscaler
```

Use each only where appropriate.

---

## 35. Resource Management

Configure:

```yaml
resources:
  requests:
    cpu: ...
    memory: ...
  limits:
    cpu: ...
    memory: ...
```

Measure actual usage before selecting production values.

---

# PART XIV — Project 13: Messaging Observability

## 36. Objective

Build a complete monitoring stack.

```text
Messaging
 |
Prometheus
 |
Grafana
```

Logs:

```text
Applications
 |
Log Collector
 |
Elasticsearch
 |
Kibana
```

Traces:

```text
Applications
 |
OpenTelemetry
 |
Jaeger
```

---

## 37. Metrics Dashboard

Create panels:

```text
message rate
consume rate
consumer lag
queue depth
unacked messages
retry count
DLQ count
processing latency
error rate
broker CPU
broker memory
broker disk
```

---

## 38. Alerts

Create alerts for:

```text
high consumer lag
growing queue
DLQ growth
broker unavailable
disk near capacity
high error rate
consumer unavailable
under-replicated Kafka partitions
```

Tune thresholds from real baseline data.

---

# PART XV — Project 14: Distributed Tracing

## 39. Objective

Trace an asynchronous request.

```text
HTTP
 |
Checkout
 |
Broker
 |
Payment Worker
 |
Database
```

Propagate:

```text
trace context
correlation ID
```

through message metadata.

---

## 40. Trace Investigation

Create a deliberate delay in Payment Worker.

Use Jaeger to identify:

```text
request latency
message wait time
consumer processing time
database time
```

---

# PART XVI — Project 15: GitOps

## 41. Objective

Deploy messaging applications using GitOps.

Repository:

```text
messaging-gitops/
|
+-- base/
|   +-- producer/
|   +-- consumer/
|   +-- config/
|
+-- overlays/
    +-- dev/
    +-- staging/
    +-- production/
```

---

## 42. Argo CD Flow

```text
Git Commit
    |
Argo CD
    |
Kubernetes
    |
Messaging Workloads
```

Practice:

```text
sync
rollback
drift detection
health status
```

---

# PART XVII — Project 16: CI/CD

## 43. Pipeline

Build:

```text
Git Push
 |
Lint
 |
Unit Tests
 |
Integration Tests
 |
Contract Tests
 |
Security Scan
 |
Build Image
 |
Push Registry
 |
Deploy
 |
Smoke Test
```

---

## 44. Messaging Integration Tests

Test:

```text
producer -> broker
broker -> consumer
ack/commit
retry
DLQ
duplicate
restart
schema validation
```

---

# PART XVIII — Project 17: Security

## 45. Objective

Secure the messaging platform.

Implement:

```text
TLS
authentication
authorization
Kubernetes Secrets
NetworkPolicy
RBAC
least privilege
```

---

## 46. Access Model

Example:

```text
payment-worker
   |
READ payment queue
WRITE payment events
```

It should not receive administrative broker permissions.

---

## 47. Secret Rotation

Test:

```text
old credential
     |
rotate
     |
new credential
     |
application reconnect
```

Document how zero/minimal downtime is achieved.

---

# PART XIX — Project 18: Load Testing

## 48. Objective

Determine messaging capacity.

Generate:

```text
100 msg/sec
500 msg/sec
1,000 msg/sec
5,000 msg/sec
```

Measure:

```text
throughput
latency
lag
CPU
memory
disk
network
```

---

## 49. Find the Bottleneck

Potential bottlenecks:

```text
broker CPU
broker disk
network
producer
consumer
database
external API
partition count
```

Do not assume the broker is always the bottleneck.

---

# PART XX — Project 19: Backpressure

## 50. Objective

Simulate a slow database.

```text
Broker
 |
Consumer
 |
Slow DB
```

Observe:

```text
processing latency increases
lag increases
```

Then implement bounded concurrency.

---

## 51. Verify Recovery

Restore database performance.

Expected:

```text
processing rate increases
lag decreases
system returns to normal
```

Do not allow uncontrolled catch-up to overload the recovered dependency.

---

# PART XXI — Project 20: Chaos Testing

## 52. Kill Consumer Pods

```text
kubectl delete pod
```

Observe:

```text
message recovery
rebalancing
lag
duplicates
```

---

## 53. Kill Broker Instance

Observe:

```text
failover
reconnection
availability
message continuity
```

---

## 54. Network Failure

Introduce controlled network disruption.

Observe:

```text
timeouts
retry
reconnect
lag
recovery
```

---

# PART XXII — Project 21: Disaster Recovery

## 55. Define RPO/RTO

Example:

```text
RPO = 5 minutes
RTO = 30 minutes
```

These are example objectives only.

Design the system to meet the actual agreed business requirements.

---

## 56. DR Plan

Document:

```text
backup
replication
DNS/failover
credentials
broker restoration
application restoration
data validation
replay strategy
```

---

## 57. DR Exercise

Simulate:

```text
primary messaging platform unavailable
```

Execute:

```text
declare incident
activate DR
restore/redirect
validate consumers
validate data
replay required events
measure RTO/RPO
```

---

# PART XXIII — Project 22: Multi-AZ Architecture

## 58. Objective

Deploy across availability zones.

```text
             Region
        /       |       \
      AZ-A     AZ-B     AZ-C
       |        |        |
     Broker   Broker   Broker
       |        |        |
     Worker   Worker   Worker
```

Spread critical workloads across failure domains.

---

## 59. Validate AZ Failure

Simulate loss of one zone.

Verify:

```text
broker quorum/replication
consumer recovery
application availability
latency
```

---

# PART XXIV — Project 23: Production Runbook

## 60. High Lag

Run:

```text
1. Check broker health
2. Check partition distribution
3. Check consumer count
4. Check consumer errors
5. Check processing latency
6. Check downstream dependencies
7. Check recent deployment
8. Scale only if safe
9. Monitor recovery
```

---

## 61. DLQ Growth

Run:

```text
1. Identify event type
2. Identify consumer
3. Inspect failure
4. Classify failure
5. Check recent changes
6. Fix root cause
7. Test fix
8. Replay controlled messages
9. Monitor
```

---

# PART XXV — Project 24: Senior-Level Capstone

## 62. Capstone Architecture

Build:

```text
                    USERS
                      |
                 Load Balancer
                      |
                   Frontend
                      |
                   Checkout
                      |
               Transactional DB
                      |
                    Outbox
                      |
            +---------+---------+
            |                   |
        RabbitMQ              Kafka
            |                   |
       Worker Queues       Event Streams
            |                   |
      +-----+-----+       +-----+-----+
      |           |       |           |
   Payment     Shipping  Audit     Analytics
      |           |       |           |
      +-----------+-------+-----------+
                      |
                Data Services
```

Observability:

```text
Prometheus
Grafana
ELK
OpenTelemetry
Jaeger
```

Deployment:

```text
Terraform
   |
AWS
   |
Kubernetes/EKS
   |
Argo CD
```

---

# PART XXVI — Capstone Requirements

## 63. Functional

Implement:

```text
order creation
payment
shipping
notification
order completion
```

---

## 64. Reliability

Implement:

```text
retry
DLQ
idempotency
outbox
graceful shutdown
HA
```

---

## 65. Security

Implement:

```text
TLS
authentication
authorization
NetworkPolicy
secret management
RBAC
```

---

## 66. Observability

Implement:

```text
metrics
logs
traces
dashboards
alerts
```

---

## 67. Operations

Document:

```text
deployment
rollback
incident response
backup
DR
capacity planning
scaling
failure recovery
```

---

# PART XXVII — Interview Project Explanation

## 68. One-Minute Explanation

> I built a production-oriented asynchronous microservices platform using RabbitMQ and Kafka. RabbitMQ handled worker-style asynchronous workflows, while Kafka handled durable event streams and independent consumer groups. I implemented retries, DLQs, idempotency and transactional outbox to address delivery and consistency problems. The workloads were containerized and deployed to Kubernetes with secure secret handling, resource management, health probes and graceful shutdown. I added Prometheus and Grafana for metrics, ELK for logs and OpenTelemetry with Jaeger for distributed tracing. CI/CD and GitOps automated deployments, and I performed failure, load and recovery testing to validate the platform.

---

# PART XXVIII — Troubleshooting Scenarios

## 69. Scenario: Queue Is Growing

Possible causes:

```text
consumer down
consumer slow
downstream dependency slow
message burst
insufficient consumers
broker issue
```

Approach:

```text
observe -> identify bottleneck -> remediate -> validate
```

---

## 70. Scenario: Kafka Lag Suddenly Increases

Check:

```text
producer rate
consumer rate
partition assignment
consumer errors
rebalance activity
CPU
memory
GC
database latency
external API latency
```

---

## 71. Scenario: Duplicate Payment

Check:

```text
ack timing
offset commit timing
consumer crash
retry logic
idempotency implementation
payment provider semantics
```

Never simply increase retries.

---

## 72. Scenario: DLQ Suddenly Spikes

Check:

```text
deployment
schema change
dependency failure
credential expiry
certificate expiry
database change
application exception
```

---

# PART XXIX — Production Checklist

## 73. Architecture

```text
[ ] workload classified correctly
[ ] broker selected appropriately
[ ] HA design
[ ] failure domains
[ ] capacity plan
[ ] DR design
```

---

## 74. Reliability

```text
[ ] durable messages
[ ] replication
[ ] retry
[ ] backoff
[ ] DLQ
[ ] idempotency
[ ] outbox where required
[ ] graceful shutdown
```

---

## 75. Security

```text
[ ] TLS
[ ] authentication
[ ] authorization
[ ] least privilege
[ ] secret manager
[ ] NetworkPolicy
[ ] audit logging
```

---

## 76. Observability

```text
[ ] metrics
[ ] dashboards
[ ] alerts
[ ] logs
[ ] tracing
[ ] correlation IDs
[ ] consumer lag
[ ] DLQ monitoring
```

---

## 77. Operations

```text
[ ] runbooks
[ ] deployment procedure
[ ] rollback procedure
[ ] backup procedure
[ ] DR procedure
[ ] failure testing
[ ] load testing
[ ] ownership
```

---

# PART XXX — Portfolio Presentation

## 78. GitHub Structure

Recommended:

```text
messaging-projects/
|
+-- rabbitmq/
|   +-- producer-consumer/
|   +-- routing/
|   +-- retry-dlq/
|   +-- ha/
|
+-- kafka/
|   +-- producer-consumer/
|   +-- consumer-groups/
|   +-- partitioning/
|   +-- retry-dlq/
|
+-- patterns/
|   +-- idempotency/
|   +-- outbox/
|
+-- kubernetes/
|   +-- manifests/
|   +-- helm/
|
+-- observability/
|   +-- prometheus/
|   +-- grafana/
|   +-- otel/
|
+-- gitops/
|   +-- argocd/
|
+-- docs/
    +-- architecture/
    +-- runbooks/
    +-- disaster-recovery/
```

---

# 79. README Requirements

Each project README should contain:

```text
Problem
Architecture
Prerequisites
Installation
Configuration
Deployment
Testing
Failure scenarios
Observability
Troubleshooting
Production considerations
```

Include architecture diagrams.

---

# PART XXXI — Evidence for Job Interviews

## 80. What to Demonstrate

Do not only say:

```text
I know Kafka.
```

Show:

```text
architecture diagram
deployment manifests
CI/CD pipeline
Grafana dashboard
DLQ implementation
failure test
load test results
runbook
```

---

## 81. Strong Portfolio Story

Explain:

```text
Problem
   |
Design
   |
Implementation
   |
Failure
   |
Investigation
   |
Fix
   |
Measurement
   |
Production improvement
```

This demonstrates engineering maturity.

---

# PART XXXII — Final Capstone Questions

## 82. Why Kafka and RabbitMQ Together?

Answer:

> I would not introduce two brokers without a reason. RabbitMQ is useful for queue-oriented worker workloads and routing, while Kafka is useful for durable event streams, replay and multiple independent consumer groups. If the organization already standardizes on one platform and it can satisfy both requirements, reducing operational complexity may be preferable.

---

## 83. What Is the Biggest Messaging Risk?

Answer:

> The biggest risk is treating messaging as a simple transport mechanism. Production systems must explicitly handle duplicate delivery, ordering, retries, poison messages, schema evolution, backpressure, observability, security and recovery.

---

## 84. How Do You Know the System Is Production Ready?

Answer:

> I would not determine readiness from a successful happy-path test alone. I would validate capacity, HA, security, observability, failure recovery, retry/DLQ behavior, idempotency, deployment and rollback, backup and DR, and verify the agreed SLO, RPO and RTO targets.

---

# PART XXXIII — Final Learning Path

## 85. Recommended Sequence

Study and implement:

```text
1. RabbitMQ fundamentals
2. RabbitMQ routing
3. Acknowledgements
4. Retry/DLQ
5. HA
6. Kafka fundamentals
7. Partitions
8. Consumer groups
9. Offsets
10. Retention
11. Event-driven architecture
12. Delivery semantics
13. Idempotency
14. Outbox
15. RoboShop integration
16. Kubernetes
17. Security
18. Observability
19. GitOps
20. Failure testing
21. DR
22. Capstone
```

---

# 86. Completion Criteria

You should be able to independently explain and demonstrate:

```text
Producer -> Broker -> Consumer

Queue vs Topic

Exchange vs Routing Key

Partition vs Consumer Group

Offset vs Acknowledgement

At-most-once
At-least-once
Effectively-once business processing

Retry vs DLQ

Idempotency

Outbox

Saga

Backpressure

Ordering

Schema Evolution

HA

DR

Observability

Security

Kubernetes Deployment

GitOps

Production Troubleshooting
```

---

# 87. Final Mental Model

Messaging projects should always be evaluated across five dimensions:

```text
                 MESSAGING
                     |
      +--------------+--------------+
      |              |              |
 Reliability       Scale         Security
      |              |              |
 Retry/DLQ       Partitions      TLS/RBAC
 Idempotency     Consumers       Secrets
 Outbox          Backpressure    NetworkPolicy
      |              |              |
      +--------------+--------------+
                     |
               Observability
                     |
             Metrics/Logs/Traces
                     |
                  Recovery
                     |
                HA / DR / RTO
```

The final objective is not:

```text
"Can I send a message?"
```

It is:

```text
"Can I operate a distributed messaging system
safely and reliably in production?"
```

That is the standard expected from a strong DevOps/SRE engineer.
