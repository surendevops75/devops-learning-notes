# OpenTelemetry Traces

## 1. Overview

OpenTelemetry Traces provides a standardized way to create, collect, process, correlate, and export distributed tracing data.

The basic flow is:

```text
Application
     ↓
OpenTelemetry Tracing API
     ↓
OpenTelemetry SDK
     ↓
Tracer
     ↓
Spans
     ↓
OTLP Exporter
     ↓
OpenTelemetry Collector
     ↓
Tracing Backend
```

In a production microservices environment:

```text
User
 ↓
ALB
 ↓
Frontend
 ↓
Orders
 ↓
Payment
 ↓
Inventory
 ↓
Database
```

A distributed trace connects these individual operations into one request journey.

---

# 2. What Is a Trace?

A trace represents the complete journey of a request through a distributed system.

Example:

```text
Trace
│
├── Frontend HTTP request
│
├── Orders service
│
├── Payment service
│
├── Inventory service
│
└── Database query
```

The trace allows engineers to answer:

```text
Where did the request spend time?
Which service failed?
Which dependency is slow?
Where did the error originate?
```

---

# 3. What Is a Span?

A span represents one operation within a trace.

Example:

```text
Trace
│
├── GET /checkout
├── orders.process
├── payment.process
├── inventory.reserve
└── database.query
```

Each span contains information such as:

```text
Span name
Start time
End time
Duration
Attributes
Events
Status
Trace ID
Span ID
Parent relationship
```

---

# 4. Trace and Span Relationship

The easiest mental model is:

```text
Trace
  │
  ├── Span A
  │     ├── Child Span A1
  │     └── Child Span A2
  │
  └── Span B
        └── Child Span B1
```

A trace is the complete tree.

A span is one operation inside that tree.

---

# 5. Trace ID

Every trace has a Trace ID.

Example:

```text
trace_id =
4bf92f3577b34da6a3ce929d0e0e4736
```

All spans belonging to the same distributed request share the same Trace ID.

```text
Frontend
trace_id = ABC

Orders
trace_id = ABC

Payment
trace_id = ABC
```

This is what allows the backend to reconstruct the complete trace.

---

# 6. Span ID

Every span has its own Span ID.

Example:

```text
Trace ID = ABC

Frontend:
Span ID = 111

Orders:
Span ID = 222

Payment:
Span ID = 333
```

Therefore:

```text
Trace ID
   ↓
Identifies the entire request

Span ID
   ↓
Identifies one operation
```

---

# 7. Parent Span

A span can have a parent.

Example:

```text
Frontend Span
      │
      └── Orders Span
             │
             └── Payment Span
```

Relationships:

```text
Orders
parent = Frontend

Payment
parent = Orders
```

This creates the trace hierarchy.

---

# 8. Root Span

The first span in a trace is normally the root span.

Example:

```text
Root Span
GET /checkout
     │
     ├── Orders
     ├── Payment
     └── Inventory
```

The root span represents the overall operation from the perspective of the entry point.

---

# 9. Child Spans

Child spans represent operations performed as part of a parent operation.

Example:

```text
Orders
│
├── Validate Request
├── Database Query
├── Payment Request
└── Inventory Request
```

This provides detailed visibility inside a service.

---

# 10. Span Timing

Every span has timing information.

```text
Start
  ↓
Operation
  ↓
End
```

Example:

```text
Orders Span
Start = 10:00:00.000
End   = 10:00:00.250

Duration = 250ms
```

If a child span consumes most of the duration, it becomes a strong candidate for investigation.

---

# 11. Trace Waterfall

Tracing backends commonly display a waterfall.

```text
Frontend      |████████████████████| 1000ms
Orders             |██████████████| 700ms
Payment                   |███████| 350ms
Database                       |███| 150ms
```

This immediately shows where time was spent.

---

# 12. Distributed Trace Example

Consider:

```text
User
 ↓
ALB
 ↓
Orders Service
 ↓
Payment Service
 ↓
Inventory Service
 ↓
MongoDB
```

Trace:

```text
Trace ABC
│
├── Orders HTTP
│
├── Payment HTTP
│    └── Payment DB
│
├── Inventory HTTP
│    └── Inventory DB
│
└── Orders DB
```

One request can therefore contain many spans.

---

# 13. OpenTelemetry TracerProvider

The `TracerProvider` provides tracers.

Conceptually:

```text
Application
     ↓
TracerProvider
     ↓
Tracer
     ↓
Span
```

The provider is responsible for configuring tracing behavior.

---

# 14. Tracer

A tracer creates spans.

Conceptually:

```text
Tracer
   ↓
startSpan()
   ↓
Span
   ↓
end()
```

Example:

```text
tracer
   ↓
process-payment
   ↓
span
```

The exact syntax depends on the programming language.

---

# 15. Manual Instrumentation

Manual instrumentation allows developers to create spans around business operations.

Example:

```text
process-order
validate-payment
reserve-inventory
send-notification
```

This provides visibility that automatic instrumentation may not provide.

---

# 16. Manual Span Example

Conceptually:

```text
span = tracer.startSpan("process-payment")

processPayment()

span.end()
```

A production implementation should also handle exceptions and ensure the span is ended correctly.

---

# 17. Automatic Instrumentation

Automatic instrumentation instruments supported libraries and frameworks.

Examples:

```text
HTTP
JDBC
Spring
Kafka
Database clients
Messaging frameworks
```

Architecture:

```text
Application
     ↓
Auto Instrumentation
     ↓
Spans
     ↓
OTel SDK
```

This significantly reduces manual code changes.

---

# 18. Automatic + Manual Instrumentation

Production applications often use both.

```text
Application
│
├── Automatic instrumentation
│     ├── HTTP
│     ├── Database
│     └── Messaging
│
└── Manual instrumentation
      ├── process-order
      ├── payment-validation
      └── inventory-reservation
```

This provides technical and business-level visibility.

---

# 19. Span Attributes

Attributes provide additional information about a span.

Example:

```text
http.request.method = POST
http.route = /api/orders
http.response.status_code = 200
service.name = orders
```

Database span:

```text
db.system = mongodb
db.operation = find
```

Attributes make traces searchable and useful.

---

# 20. Semantic Conventions

OpenTelemetry defines standardized attribute conventions.

Common areas include:

```text
HTTP
Database
Messaging
RPC
Cloud
Kubernetes
```

Using standard conventions makes traces consistent across different services.

---

# 21. Span Events

Events represent notable occurrences during a span.

Example:

```text
Payment Span
│
├── Start
│
├── Event: payment_retry
│
├── Event: gateway_response
│
└── End
```

Events are useful for recording important moments without creating unnecessary child spans.

---

# 22. Span Status

A span can indicate whether the operation succeeded or failed.

Conceptually:

```text
UNSET
OK
ERROR
```

Example:

```text
Payment Span
status = ERROR
```

An error status helps tracing backends identify failed operations.

---

# 23. Recording Exceptions

When an operation fails, the application can record an exception on the span.

Conceptually:

```text
try:
    processPayment()

catch error:
    span.recordException(error)
    span.setStatus(ERROR)
```

This provides error information directly inside the trace.

---

# 24. Span Links

Not every relationship is parent-child.

Sometimes one span needs to reference another trace or span without becoming its child.

Span links can represent such relationships.

Example:

```text
Trace A
   │
   └── Producer Span
          │
          ↓
       Queue
          │
          ↓
Trace B
   │
   └── Consumer Span
          ↑
       Link
```

This is useful for asynchronous and batch processing.

---

# 25. Context

Tracing depends on context.

Context can contain:

```text
Trace ID
Span ID
Trace flags
Baggage
```

Architecture:

```text
Current Span
     ↓
Context
     ↓
Next Operation
```

---

# 26. Context Propagation

Context propagation allows trace information to travel between services.

```text
Service A
   ↓
HTTP Request
   ↓
traceparent
   ↓
Service B
```

Service B extracts the context and continues the trace.

---

# 27. W3C Trace Context

OpenTelemetry commonly uses W3C Trace Context.

Important headers include:

```text
traceparent
tracestate
```

Example concept:

```text
traceparent:
00-TRACE_ID-SPAN_ID-01
```

The receiving service uses this information to establish the correct parent relationship.

---

# 28. Distributed Trace Without Propagation

Without propagation:

```text
Service A
Trace A

Service B
Trace B

Service C
Trace C
```

The backend cannot reliably connect them into one distributed trace.

---

# 29. Distributed Trace With Propagation

With propagation:

```text
Trace ABC
│
├── Service A
├── Service B
└── Service C
```

This is one of the most important concepts in distributed tracing.

---

# 30. HTTP Trace Propagation

Example:

```text
Frontend
   ↓
HTTP Request
   ↓
traceparent
   ↓
Orders
```

Orders extracts the trace context:

```text
traceparent
     ↓
Parent Span
     ↓
New Orders Span
```

---

# 31. Messaging Trace Propagation

For asynchronous communication:

```text
Orders
   ↓
RabbitMQ / Kafka
   ↓
Payment
```

The trace context can be injected into message metadata.

```text
Producer
   ↓
Message + Trace Context
   ↓
Queue
   ↓
Consumer
```

This preserves trace relationships across asynchronous processing.

---

# 32. Synchronous vs Asynchronous Tracing

Synchronous:

```text
Orders
  ↓
HTTP
  ↓
Payment
  ↓
Response
```

Asynchronous:

```text
Orders
  ↓
Queue
  ↓
Payment Consumer
```

Asynchronous systems may require span links rather than a simple parent-child structure depending on the processing model.

---

# 33. Trace Sampling

High-volume systems can generate enormous numbers of spans.

Example:

```text
1,000,000 requests
      ↓
Potentially millions of spans
```

Sampling reduces the amount retained.

```text
1,000,000 requests
      ↓
10% sampling
      ↓
100,000 traces
```

---

# 34. Head Sampling

Head sampling makes the sampling decision early.

```text
Request
   ↓
Sampler
   ├── Keep
   └── Drop
```

Advantages:

```text
Simple
Low overhead
Reduces telemetry early
```

Disadvantage:

```text
The sampler may not know whether the trace will later contain an error.
```

---

# 35. Tail Sampling

Tail sampling makes the decision after more or all of the trace is available.

```text
Trace
 ↓
Collect spans
 ↓
Analyze trace
 ↓
Sampling decision
 ├── Keep
 └── Drop
```

Example:

```text
Error trace → Keep
Slow trace  → Keep
Normal trace → Sample
```

Tail sampling is commonly handled at the Collector layer.

---

# 36. Parent-Based Sampling

A child service can follow the sampling decision made by its parent.

```text
Frontend
   ↓
Sampled
   ↓
Orders
   ↓
Payment
```

This helps keep distributed traces consistent.

---

# 37. Sampling Strategy for Production

A production environment may use:

```text
Normal traces
    ↓
Sample

Error traces
    ↓
Keep

Slow traces
    ↓
Keep
```

This preserves valuable diagnostic traces while controlling storage and ingestion costs.

---

# 38. SpanProcessor

The SpanProcessor receives spans and prepares them for export.

Architecture:

```text
Span
 ↓
SpanProcessor
 ↓
Exporter
```

Common processor concepts:

```text
Simple
Batch
```

---

# 39. Batch Span Processor

The batch processor collects multiple spans before export.

```text
Span 1 ┐
Span 2 ├──→ Batch
Span 3 ┘
          ↓
       Exporter
```

Benefits:

```text
Better throughput
Lower network overhead
More efficient export
```

Batch processing is commonly preferred for production.

---

# 40. Simple Span Processor

A simple processor can export spans individually.

```text
Span 1 → Export
Span 2 → Export
Span 3 → Export
```

This can be useful for development and debugging but can introduce more overhead in high-volume environments.

---

# 41. OTLP Trace Export

A common architecture:

```text
Application
     ↓
OTel SDK
     ↓
OTLP Exporter
     ↓
OTel Collector
     ↓
Tracing Backend
```

The application does not need to know the backend-specific protocol.

---

# 42. Collector Traces Pipeline

A typical traces pipeline:

```text
Traces
  ↓
OTLP Receiver
  ↓
Memory Limiter
  ↓
Resource Processor
  ↓
Tail Sampling
  ↓
Batch
  ↓
OTLP Exporter
```

This centralizes trace processing.

---

# 43. Trace Resource Attributes

Every trace should identify its source service.

Recommended:

```text
service.name
service.version
deployment.environment
```

For Kubernetes:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.container.name
```

---

# 44. Service Name

`service.name` is particularly important.

Example:

```text
service.name = orders
```

Other services:

```text
service.name = payment
service.name = inventory
service.name = notification
```

Without consistent service names, distributed traces become difficult to analyze.

---

# 45. Service Version

Include:

```text
service.version = v2.3.1
```

This allows engineers to compare trace behavior across releases.

Example:

```text
v2.2.0 → p95 = 250ms
v2.3.0 → p95 = 800ms
```

A release regression becomes easier to identify.

---

# 46. Environment

Include:

```text
deployment.environment = production
```

Then traces can be separated:

```text
production
staging
development
```

This prevents confusion when multiple environments send data to the same backend.

---

# 47. HTTP Server Tracing

Example:

```text
GET /api/orders
```

Span:

```text
HTTP GET /api/orders
duration = 125ms
status = 200
```

Possible attributes:

```text
http.request.method
http.route
http.response.status_code
server.address
```

---

# 48. HTTP Client Tracing

When Orders calls Payment:

```text
Orders
   ↓
HTTP Client Span
   ↓
Payment
```

Trace:

```text
Orders
│
└── HTTP POST /payments
       │
       └── Payment Server Span
```

The client and server spans represent the same cross-service request from different perspectives.

---

# 49. Database Tracing

Example:

```text
Orders
   ↓
Database Span
   ↓
MongoDB
```

Span:

```text
db.query
duration = 450ms
```

Useful information:

```text
Database system
Operation
Server
Duration
Status
```

Avoid recording sensitive query data unnecessarily.

---

# 50. External API Tracing

Example:

```text
Payment
   ↓
Payment Gateway
```

Trace:

```text
Payment
└── HTTP POST /gateway/payment
       duration = 900ms
```

If this span consumes most of the overall request time, the external dependency becomes a strong investigation candidate.

---

# 51. Trace Waterfall Example

```text
Checkout                    |████████████████████████| 1200ms
Orders                      |██████████████████████  | 1100ms
Payment                            |██████████         | 500ms
Payment Gateway                       |███████           | 350ms
Inventory                     |████                  | 200ms
Database                         |██                  | 100ms
```

The waterfall makes latency contribution visible.

---

# 52. Finding a Slow Service

Suppose:

```text
Checkout = 2 seconds
Orders   = 1.9 seconds
Payment  = 1.5 seconds
```

Tracing reveals:

```text
Payment
   ↓
External Gateway
   ↓
1.3 seconds
```

The trace provides evidence rather than requiring guesswork.

---

# 53. Finding a Failing Service

Trace:

```text
Frontend
   ↓ 200
Orders
   ↓ 200
Payment
   ↓ 500
Database
```

Payment is the first service returning an error.

Then inspect the Payment span:

```text
status = ERROR
error.type = timeout
```

This narrows the investigation.

---

# 54. Error Span

An error span might contain:

```text
status = ERROR
error.type = TimeoutError
error.message = Gateway timeout
```

Events can include:

```text
exception
```

This provides structured error context.

---

# 55. Trace Exceptions

A span can record an exception:

```text
Payment Span
│
├── Exception:
│     TimeoutError
│
└── Status:
      ERROR
```

The backend can display the exception alongside the span.

---

# 56. Trace and Logs

A trace can contain:

```text
trace_id = ABC
span_id = XYZ
```

A log generated during that operation can contain:

```text
trace_id = ABC
span_id = XYZ
```

Therefore:

```text
Trace
  ↓
Span
  ↓
Log
```

The engineer can move directly between related telemetry.

---

# 57. Trace and Metrics

Metrics can indicate:

```text
payment_latency_p95 ↑
```

Traces show:

```text
Payment
   ↓
Gateway
   ↓
Database
```

This creates a useful investigation path:

```text
Metrics
   ↓
Find problem
   ↓
Traces
   ↓
Find operation
   ↓
Logs
   ↓
Find detailed error
```

---

# 58. Exemplars

Metrics can sometimes link a measurement to a trace through exemplars.

Conceptually:

```text
Latency Metric
     ↓
Exemplar
     ↓
Trace ID
     ↓
Trace
```

This creates a direct path from a metric visualization to an individual trace.

Support depends on the selected metrics backend and integration.

---

# 59. Trace Backend

The Collector exports traces to a tracing backend.

Examples include:

```text
Jaeger
Grafana Tempo
Other OTLP-compatible backends
```

Architecture:

```text
Application
     ↓
Collector
     ↓
Tracing Backend
     ↓
Trace UI
```

---

# 60. Jaeger Architecture

A conceptual architecture:

```text
Application
     ↓
OTel SDK
     ↓
OTel Collector
     ↓
Jaeger
     ↓
Jaeger UI
```

The application remains OpenTelemetry-native rather than being tightly coupled to Jaeger.

---

# 61. OpenTelemetry Collector With Jaeger

```text
Traces
   ↓
OTLP Receiver
   ↓
Processors
   ↓
OTLP / Jaeger-compatible Export
   ↓
Jaeger
```

The exact exporter should match the supported backend and current Collector distribution.

---

# 62. Trace Sampling at Collector

A production gateway can perform tail sampling:

```text
Applications
     ↓
OTel Agents
     ↓
OTel Gateway
     ↓
Tail Sampling
     ├── Error → Keep
     ├── Slow → Keep
     └── Normal → Sample
     ↓
Tracing Backend
```

This prevents every application from needing complex sampling logic.

---

# 63. Trace Routing

Traces can be routed based on:

```text
service.name
environment
region
tenant
cluster
```

Example:

```text
production
   ↓
Production Trace Backend

staging
   ↓
Staging Trace Backend
```

Routing reduces backend mixing and can support multi-environment architectures.

---

# 64. Kubernetes Trace Architecture

```text
EKS
│
├── Orders
│    └── OTel SDK
│
├── Payment
│    └── OTel SDK
│
├── Inventory
│    └── OTel SDK
│
└── OTel Agent
        ↓
    OTel Gateway
        ↓
    Trace Backend
```

The application Pods export traces to the Collector layer.

---

# 65. Agent + Gateway

A scalable tracing architecture:

```text
Application
   ↓
OTel Agent
   ↓
OTel Gateway
   ↓
Tail Sampling
   ↓
Trace Backend
```

Agent:

```text
Local collection
```

Gateway:

```text
Central processing
Sampling
Routing
Export
```

---

# 66. Collector DaemonSet

A DaemonSet can provide a Collector on each Kubernetes node.

```text
Node 1
├── Applications
└── OTel Agent

Node 2
├── Applications
└── OTel Agent

Node 3
├── Applications
└── OTel Agent
```

This provides distributed collection.

---

# 67. Collector Gateway Deployment

The gateway can run as a Deployment:

```text
OTel Gateway
│
├── Replica 1
├── Replica 2
└── Replica 3
```

Applications or agents connect through a Kubernetes Service.

```text
otel-gateway.observability.svc
```

---

# 68. Gateway High Availability

Use multiple replicas:

```text
Gateway-01
Gateway-02
Gateway-03
```

Improve resilience with:

```text
Pod anti-affinity
Topology spread
PodDisruptionBudget
Multiple nodes
Multiple AZs where appropriate
```

---

# 69. Trace Pipeline Reliability

The tracing system should tolerate:

```text
Collector restart
Backend restart
Pod restart
Network interruption
Deployment
Node failure
```

Tracing failures should not stop application traffic.

---

# 70. Application Availability vs Trace Availability

Critical principle:

```text
Application availability
        >
Tracing availability
```

If tracing backend fails:

```text
Payment
   ↓
Should continue
```

not:

```text
Tracing backend unavailable
   ↓
Payment unavailable
```

Telemetry must be non-critical to business execution.

---

# 71. Trace Export Queues

The SDK and Collector may buffer telemetry.

```text
Spans
 ↓
Queue
 ↓
Exporter
 ↓
Backend
```

Queues should be bounded.

Otherwise:

```text
Backend failure
   ↓
Queue grows
   ↓
Memory grows
   ↓
Collector OOM
```

---

# 72. Trace Backpressure

When the backend cannot keep up:

```text
Trace volume
      ↓
Collector
      ↓
Backend slower
      ↓
Queue grows
```

Possible actions:

```text
Retry
Queue
Sampling
Scaling
Dropping low-value traces
```

---

# 73. Trace Performance Overhead

Tracing introduces overhead from:

```text
Span creation
Context propagation
Attribute processing
Event recording
Serialization
Export
```

Reduce unnecessary overhead through:

```text
Sampling
Batching
Selective instrumentation
Controlled attributes
Efficient exporters
```

---

# 74. High-Cardinality Span Attributes

Avoid excessive attributes such as:

```text
user_id
session_id
request_id
full request body
large response body
```

These can increase backend storage and query complexity.

Use only attributes that provide operational value.

---

# 75. Sensitive Data in Traces

Do not record:

```text
Passwords
Access tokens
Authorization headers
Credit card data
Private keys
Sensitive personal information
```

Example:

```text
Authorization: Bearer eyJ...
```

should never be captured as a normal span attribute.

---

# 76. Trace Attribute Redaction

Sensitive information should be removed or masked before reaching the backend.

Possible locations:

```text
Application
   ↓
Collector
   ↓
Backend
```

The earlier the data is removed, the better.

---

# 77. Trace Sampling and Cost

Suppose:

```text
1 million requests/day
```

Each request generates multiple spans.

Without sampling:

```text
Millions of spans
   ↓
High storage
High network
High backend cost
```

With appropriate sampling:

```text
Normal traces → Sample
Errors → Keep
Slow traces → Keep
```

This provides useful visibility while controlling cost.

---

# 78. SLO and Traces

SLOs are generally evaluated from metrics, but traces can help investigate violations.

Example:

```text
SLO:
99.9% requests < 500ms
```

Metric:

```text
p99 latency = 1.2s
```

Trace:

```text
Orders
   ↓
Payment
   ↓
External Gateway
   ↓
900ms
```

The trace helps explain the SLO violation.

---

# 79. Traces During Deployment

Before deployment:

```text
p95 = 200ms
error rate = 0.5%
```

After deployment:

```text
p95 = 700ms
error rate = 4%
```

Trace comparison can identify:

```text
New database query
New external API call
New processing step
Configuration issue
```

---

# 80. Canary Tracing

For a canary:

```text
Version A → 95%
Version B → 5%
```

Compare traces for:

```text
Latency
Errors
Dependency calls
Span duration
```

If the new version produces slower traces:

```text
Rollback
```

---

# 81. Trace-Based Troubleshooting

A typical interview troubleshooting flow:

```text
1. Identify affected service
       ↓
2. Check metrics
       ↓
3. Find slow/error traces
       ↓
4. Inspect trace waterfall
       ↓
5. Identify slow span
       ↓
6. Inspect span attributes/events
       ↓
7. Correlate logs using trace ID
       ↓
8. Identify root cause
       ↓
9. Fix
       ↓
10. Verify metrics/traces
```

---

# 82. Example: 503 Error

Suppose users receive:

```text
HTTP 503
```

Start with:

```text
Metric:
HTTP 5xx rate ↑
```

Then trace:

```text
ALB
 ↓
Orders
 ↓
Payment
 X
```

Then inspect logs:

```text
Payment service unavailable
```

Then Kubernetes:

```text
Payment Pods
CrashLoopBackOff
```

Tracing narrows the affected service; logs and Kubernetes confirm the root cause.

---

# 83. Example: High Latency

Metric:

```text
p95 = 1.5s
```

Trace:

```text
Orders = 1.4s
Payment = 1.1s
Database = 950ms
```

Root cause candidate:

```text
Database query
```

Then inspect:

```text
Database logs
Database metrics
Query performance
Connection pool
```

---

# 84. Example: Intermittent Failure

Suppose:

```text
Error rate = 1%
```

Most requests succeed.

Tracing allows engineers to find only failing traces:

```text
Error traces
   ↓
Payment
   ↓
External Gateway
   ↓
Timeout
```

Sampling strategies should ensure important error traces are retained.

---

# 85. Trace Search Strategy

Start with:

```text
service.name
environment
time range
status
```

Then narrow with:

```text
operation
http.route
duration
trace ID
version
```

Example:

```text
service.name = payment
AND
status = ERROR
AND
environment = production
```

---

# 86. Trace Backend Investigation

When investigating a trace:

```text
Trace Overview
   ↓
Root Span
   ↓
Child Spans
   ↓
Slow Span
   ↓
Attributes
   ↓
Events
   ↓
Exceptions
   ↓
Logs
```

This provides a structured debugging workflow.

---

# 87. Trace Naming

Span names should describe operations clearly.

Good:

```text
GET /api/orders
POST /payments
orders.process
inventory.reserve
```

Poor:

```text
operation1
request
process
span
```

Clear span names make traces easier to understand.

---

# 88. Avoid Dynamic Span Names

Avoid:

```text
GET /users/12345
GET /users/12346
GET /users/12347
```

Prefer a normalized route:

```text
GET /users/{id}
```

Dynamic span names can create unnecessary backend complexity.

---

# 89. Span Duration

Duration is one of the most important span properties.

Example:

```text
Payment Span
duration = 900ms
```

Compare:

```text
Orders = 100ms
Payment = 900ms
Inventory = 80ms
```

Payment is clearly contributing most of the latency.

---

# 90. Span Status vs HTTP Status

HTTP:

```text
HTTP 500
```

should generally be represented appropriately in the span.

But not every non-2xx response necessarily represents an application error from the tracing system's perspective.

Instrumentation and semantic conventions should be followed rather than manually marking every unusual response incorrectly.

---

# 91. Trace Context Security

Trace context should not contain sensitive information.

Be careful when propagating:

```text
Baggage
Custom headers
User metadata
Tenant information
```

Do not propagate secrets.

---

# 92. Trace Context Across ALB

A typical request:

```text
User
 ↓
ALB
 ↓
Frontend
```

The application instrumentation should establish and propagate tracing context from the incoming request.

Then:

```text
Frontend
 ↓
Orders
 ↓
Payment
```

continues the trace.

---

# 93. Trace Context Across Microservices

```text
Frontend
   │
   │ traceparent
   ↓
Orders
   │
   │ traceparent
   ↓
Payment
   │
   │ traceparent
   ↓
Inventory
```

All services participate in the same trace.

---

# 94. Trace Context Across RabbitMQ

```text
Orders
   ↓
Producer Span
   ↓
RabbitMQ
   ↓
Consumer Span
   ↓
Payment
```

Trace context is propagated through message metadata.

For asynchronous fan-out:

```text
One producer
   ↓
Multiple consumers
```

span links can be useful for representing relationships.

---

# 95. Trace Context Across Kafka

```text
Producer
   ↓
Kafka Topic
   ↓
Consumer Group
   ↓
Consumer
```

Trace context can be injected into message headers.

This helps connect producer and consumer operations.

---

# 96. Trace Context Across Database

Database calls generally occur within the current service span.

```text
Orders Span
    │
    └── Database Span
```

The database span inherits the current trace context.

---

# 97. Trace Context Across External Services

For supported HTTP clients:

```text
Payment
   ↓
HTTP Client
   ↓
External Gateway
```

Trace context may be propagated to the external service.

However, external systems may not support or honor OpenTelemetry tracing.

In that case, the client-side span still provides useful visibility.

---

# 98. Tracing a Microservices Transaction

Example checkout:

```text
Trace
│
├── Frontend
│
├── Orders
│   ├── Validate Order
│   └── Database
│
├── Payment
│   ├── Payment Gateway
│   └── Database
│
├── Inventory
│   └── Database
│
└── Notification
```

This gives an end-to-end view of the transaction.

---

# 99. Production Trace Checklist

```text
[ ] TracerProvider configured
[ ] Service name configured
[ ] Service version configured
[ ] Environment configured
[ ] Automatic instrumentation enabled where appropriate
[ ] Manual instrumentation for important business operations
[ ] Semantic conventions followed
[ ] Context propagation enabled
[ ] W3C Trace Context configured
[ ] Sampling configured
[ ] Batch processing configured
[ ] OTLP exporter configured
[ ] Collector traces pipeline configured
[ ] Trace backend configured
[ ] Error traces retained
[ ] Slow traces retained
[ ] Sensitive attributes excluded
[ ] High-cardinality attributes reviewed
[ ] Graceful shutdown configured
```

---

# 100. EKS Production Checklist

```text
[ ] OTel SDK configured in microservices
[ ] OTel Agent deployed
[ ] OTel Gateway deployed
[ ] Gateway replicas configured
[ ] Kubernetes Service configured
[ ] OTLP ports configured correctly
[ ] NetworkPolicy configured
[ ] TLS configured where required
[ ] RBAC reviewed
[ ] Resource requests/limits configured
[ ] Memory limiter configured
[ ] Batch processor configured
[ ] Tail sampling evaluated
[ ] Error traces retained
[ ] Slow traces retained
[ ] Trace backend monitored
[ ] Collector monitored
[ ] Trace-to-log correlation verified
[ ] Deployment versions visible
[ ] Production rollback validated
```

---

# 101. Complete Production Trace Architecture

```text
                                  USERS
                                    │
                                    ↓
                                   ALB
                                    │
                                    ↓
                             EKS Microservices
                                    │
             ┌──────────────────────┼──────────────────────┐
             ↓                      ↓                      ↓
          Frontend                Orders                Payment
             │                      │                      │
          OTel SDK               OTel SDK               OTel SDK
             │                      │                      │
             └──────────────────────┼──────────────────────┘
                                    ↓
                                OTel Agent
                                    ↓
                              OTel Gateway
                                    │
                              Trace Processing
                                    │
                         ┌──────────┴──────────┐
                         ↓                     ↓
                    Tail Sampling           Batch
                         │                     │
                         └──────────┬──────────┘
                                    ↓
                              Trace Backend
                                    ↓
                                Trace UI
```

---

# 102. Unified Metrics + Logs + Traces

```text
                           EKS
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
           Metrics         Logs         Traces
              │             │             │
              └─────────────┼─────────────┘
                            ↓
                       OTel Agent
                            ↓
                       OTel Gateway
                            │
            ┌───────────────┼───────────────┐
            ↓               ↓               ↓
        Prometheus          ELK        Trace Backend
            ↓               ↓               ↓
         Grafana          Kibana         Trace UI
```

The common correlation fields are:

```text
service.name
service.version
deployment.environment
trace_id
span_id
```

---

# 103. Incident Response With OpenTelemetry

```text
Incident
   ↓
Grafana
   ↓
Metric indicates latency/error increase
   ↓
Trace Backend
   ↓
Find affected trace
   ↓
Inspect slow/error span
   ↓
Extract trace_id
   ↓
Kibana
   ↓
Find correlated logs
   ↓
Kubernetes / Infrastructure
   ↓
Confirm root cause
   ↓
Fix / Rollback
   ↓
Verify metrics
```

---

# 104. Trace Cost Optimization

Control tracing cost through:

```text
Sampling
Batching
Tail sampling
Attribute control
Selective instrumentation
Retention policies
Filtering
```

Recommended strategy:

```text
Normal traces
     ↓
Sample

Slow traces
     ↓
Keep

Error traces
     ↓
Keep

Critical business traces
     ↓
Keep
```

---

# 105. Trace Performance Optimization

To reduce application overhead:

```text
Use appropriate sampling
Avoid unnecessary spans
Avoid huge attributes
Avoid capturing request/response bodies
Use batch exporting
Use asynchronous export
Keep Collector processing centralized
```

Tracing should provide useful visibility without becoming a significant application bottleneck.

---

# 106. Trace Testing

Before production:

```text
Generate request
     ↓
Verify root span
     ↓
Verify child spans
     ↓
Verify Trace ID propagation
     ↓
Verify attributes
     ↓
Verify error status
     ↓
Verify Collector
     ↓
Verify backend
```

Test:

```text
Normal request
Error request
Slow request
External API failure
Database failure
Pod restart
Service restart
```

---

# 107. Trace Validation During Deployment

After deploying a new version:

```text
1. Generate test request
2. Open trace
3. Verify service names
4. Verify parent-child relationships
5. Verify durations
6. Verify errors
7. Verify logs contain Trace ID
8. Verify metrics correlate
```

This validates the complete observability path.

---

# 108. Trace Rollback Validation

After rollback:

```text
Previous Version
      ↓
Generate request
      ↓
Trace
      ↓
Compare latency
      ↓
Compare errors
```

The rollback should be confirmed through telemetry rather than assuming the issue is resolved.

---

# 109. OpenTelemetry Trace Mental Model

Remember:

```text
REQUEST
   ↓
TRACE
   ↓
SPANS
   ↓
CONTEXT
   ↓
PROPAGATION
   ↓
COLLECTOR
   ↓
SAMPLING
   ↓
BACKEND
```

More specifically:

```text
Request
  ↓
Root Span
  ↓
Child Spans
  ↓
Trace ID
  ↓
Context Propagation
  ↓
Distributed Services
  ↓
OTLP
  ↓
Collector
  ↓
Trace Backend
```

---

# 110. Final Key Concepts

The most important OpenTelemetry Tracing concepts are:

```text
Trace
Span
Trace ID
Span ID
Parent Span
Root Span
Child Span
Span Attributes
Span Events
Span Status
Exceptions
Span Links
TracerProvider
Tracer
Context
Propagation
W3C Trace Context
Sampling
Tail Sampling
SpanProcessor
Batching
OTLP
Collector
Trace Backend
```

The key production principle is:

**OpenTelemetry Traces provides end-to-end visibility across distributed applications by connecting individual operations into a single trace through spans and propagated context. In an EKS microservices environment, applications should use OpenTelemetry SDKs and automatic/manual instrumentation, propagate W3C trace context across HTTP and messaging boundaries, export through scalable Collector agents and gateways, use controlled sampling and batching, retain important error and slow traces, and correlate traces with Prometheus metrics and ELK logs for complete incident investigation.**
