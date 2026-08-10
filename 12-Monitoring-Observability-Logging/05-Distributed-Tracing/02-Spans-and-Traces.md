# Spans and Traces

Spans and traces are the core building blocks of distributed tracing.

A distributed tracing system follows a request through multiple services by creating spans and connecting them together into a trace.

The basic relationship is:

```
Request
   ↓
Trace
   ↓
Spans
   ↓
Operations
```

A trace represents the complete request journey.

A span represents one operation within that journey.

---

# 1. What Is a Trace?

A trace represents the complete lifecycle of a request through a distributed system.

Example:

```
Client
  ↓
Order Service
  ↓
Payment Service
  ↓
Inventory Service
  ↓
Database
```

The complete journey can be represented as:

```
Trace ID = abc123

Trace
  |
  +-- Order Service
  |
  +-- Payment Service
  |
  +-- Inventory Service
  |
  +-- Database
```

All these operations belong to the same trace.

---

# 2. What Is a Span?

A span represents one unit of work inside a trace.

Examples:

```
HTTP Request
Database Query
Redis Operation
External API Call
Message Publish
Message Consume
Business Operation
```

Example:

```
Trace
  |
  +-- Span: POST /orders
  |
  +-- Span: Payment API
  |
  +-- Span: PostgreSQL Query
```

Each span contains information about the operation.

---

# 3. Trace vs Span

The easiest way to remember the difference:

```
Trace = Complete Journey

Span = Individual Operation
```

Example:

```
Trace
  |
  +-- Order API Span
  |
  +-- Payment API Span
  |
  +-- Database Span
```

The trace contains the spans.

---

# 4. Real-World Example

A user submits an order.

The request travels:

```
User
  ↓
ALB
  ↓
Order Service
  ↓
Payment Service
  ↓
Inventory Service
  ↓
PostgreSQL
```

The trace could contain:

```
Span 1:
POST /orders

Span 2:
POST /payment

Span 3:
POST /inventory/reserve

Span 4:
PostgreSQL query
```

All belong to one trace.

---

# 5. Trace ID

Every distributed trace has a unique Trace ID.

Example:

```
4bf92f3577b34da6a3ce929d0e0e4736
```

The Trace ID connects all related spans.

Example:

```
Trace ID = abc123

Span A → abc123
Span B → abc123
Span C → abc123
Span D → abc123
```

---

# 6. Span ID

Each span has its own unique Span ID.

Example:

```
Trace ID:
abc123

Span ID:
span001
```

Another span:

```
Trace ID:
abc123

Span ID:
span002
```

Therefore:

```
Same Trace ID
   +
Different Span IDs
```

means the spans belong to the same trace.

---

# 7. Parent Span

A span can have a parent.

Example:

```
Order Service
     |
     +---- Payment Service
```

The Order Service span is the parent.

The Payment Service span is the child.

---

# 8. Child Span

A child span represents work initiated by another span.

Example:

```
Order API
   |
   +-- Payment API
   |
   +-- Database Query
```

Here:

```
Order API = Parent

Payment API = Child

Database Query = Child
```

---

# 9. Span Tree

A trace can be represented as a tree.

Example:

```
Trace
  |
  +-- Order API
        |
        +-- Payment API
        |      |
        |      +-- Payment Database
        |
        +-- Inventory API
               |
               +-- Inventory Database
```

This is called the span hierarchy.

---

# 10. Root Span

The root span represents the beginning of the trace.

Example:

```
Root Span
POST /orders
     |
     +-- Payment
     |
     +-- Inventory
```

The root span usually represents the incoming operation that initiated the trace.

---

# 11. Nested Spans

Spans can be nested.

Example:

```
Order Request
    |
    +-- Validate Order
    |
    +-- Process Payment
    |      |
    |      +-- Payment API
    |
    +-- Reserve Inventory
```

This allows tracing at multiple levels.

---

# 12. Span Start Time

A span records when an operation started.

Example:

```
start_time:
2026-08-10T10:30:00.100Z
```

---

# 13. Span End Time

A span records when the operation finished.

Example:

```
end_time:
2026-08-10T10:30:00.500Z
```

---

# 14. Span Duration

Duration is calculated from:

```
End Time - Start Time
```

Example:

```
Start:
10:30:00.100

End:
10:30:00.500
```

Duration:

```
400 ms
```

Duration is one of the most important span properties.

---

# 15. Why Span Duration Matters

Suppose:

```
Order Service:
100 ms

Payment Service:
2500 ms

Inventory Service:
150 ms
```

The trace immediately shows:

```
Payment Service
```

as the largest contributor to latency.

---

# 16. Span Name

Every span should have a meaningful name.

Examples:

```
GET /orders/{id}

POST /orders

process_payment

reserve_inventory

SELECT orders
```

Avoid meaningless names:

```
operation1

function123

method2
```

---

# 17. Span Attributes

Attributes provide additional information about a span.

Examples:

```
service.name
service.version
deployment.environment
http.request.method
http.response.status_code
db.system
db.operation
```

Attributes allow engineers to understand the operation in more detail.

---

# 18. HTTP Span

An HTTP server request can create a span.

Example:

```
POST /orders
```

Possible information:

```
HTTP Method
Route
Status Code
Duration
Server
Client Context
```

Example:

```
Span:
POST /orders

Status:
200

Duration:
120 ms
```

---

# 19. HTTP Client Span

When one service calls another:

```
Order Service
   ↓
Payment Service
```

The outgoing request can create a client span.

Example:

```
POST payment-service/charge
```

This helps identify downstream latency.

---

# 20. Database Span

Database operations can be represented by spans.

Example:

```
Order Service
   |
   +-- PostgreSQL Query
```

Possible information:

```
Database System
Operation
Duration
Status
```

Example:

```
db.system:
postgresql

operation:
SELECT

duration:
80 ms
```

---

# 21. External API Span

Example:

```
Payment Service
   ↓
External Payment Provider
```

Span:

```
POST /charge
```

Possible information:

```
Endpoint
Duration
Status
Error
```

Example:

```
duration:
2800 ms

status:
timeout
```

This immediately highlights the external dependency.

---

# 22. Messaging Span

For message-based systems:

```
Order Service
   ↓
RabbitMQ
   ↓
Notification Service
```

Spans can represent:

```
Message Publish
```

and:

```
Message Consume
```

This allows engineers to understand asynchronous workflows.

---

# 23. Producer Span

A producer span represents message publishing.

Example:

```
Order Service
    |
    +-- Publish order.created
                |
                ↓
             RabbitMQ
```

The span represents the producer operation.

---

# 24. Consumer Span

A consumer span represents message processing.

Example:

```
RabbitMQ
    |
    ↓
Notification Service
    |
    +-- Process order.created
```

The consumer span represents the processing operation.

---

# 25. Span Kind

Spans can describe different roles in a distributed system.

Common span kinds include:

```
Internal
Server
Client
Producer
Consumer
```

These help describe how the span participates in the operation.

---

# 26. Server Span

A server span represents handling an incoming request.

Example:

```
Client
   ↓
Order Service
```

The Order Service creates a server span.

Example:

```
Span:
POST /orders
```

---

# 27. Client Span

A client span represents an outgoing request.

Example:

```
Order Service
   ↓
Payment Service
```

Order Service creates a client span for the outgoing request.

---

# 28. Internal Span

An internal span represents an operation inside a service.

Example:

```
Order Service
    |
    +-- Validate Order
    |
    +-- Calculate Total
    |
    +-- Apply Discount
```

These operations can be represented as internal spans when they provide meaningful visibility.

---

# 29. Producer Span

A producer span represents sending a message.

Example:

```
Order Service
    |
    ↓
RabbitMQ
```

The publishing operation can be represented by a producer span.

---

# 30. Consumer Span

A consumer span represents receiving or processing a message.

Example:

```
RabbitMQ
    |
    ↓
Notification Service
```

The message-processing operation can be represented by a consumer span.

---

# 31. Span Status

A span can represent whether an operation succeeded or failed.

Conceptually:

```
UNSET
OK
ERROR
```

Example:

```
Payment API

Status:
ERROR
```

This allows tracing systems to identify failed operations.

---

# 32. Error Span

When an operation fails:

```
Payment Service
    |
    ↓
External API
    |
    ↓
  ERROR
```

The span can contain error-related information.

---

# 33. Span Events

A span can contain events that occurred during the operation.

Example:

```
Span:
process_payment

Events:

retry_started

timeout_detected

fallback_triggered
```

Events provide additional context without creating unnecessary separate spans.

---

# 34. Span Event vs Span

A span represents an operation.

A span event represents something that happened during that operation.

Example:

```
Span:
process_payment
Duration: 3 seconds

Events:
retry_started
retry_started
timeout_detected
```

The events belong to the span.

---

# 35. Span Links

Some distributed workflows do not naturally fit a simple parent-child hierarchy.

Span links allow a span to reference another span context.

This is particularly useful for:

```
Asynchronous Processing
Batch Processing
Fan-In
Fan-Out
```

The exact design depends on the workflow.

---

# 36. Parent-Child Relationship

Simple synchronous request:

```
Service A
   |
   ↓
Service B
   |
   ↓
Service C
```

Span hierarchy:

```
A
 |
 +-- B
      |
      +-- C
```

This represents the normal parent-child relationship.

---

# 37. Fan-Out

One request can trigger multiple operations.

Example:

```
Order Service
   |
   +------→ Payment
   |
   +------→ Inventory
   |
   +------→ Notification
```

The trace may contain:

```
Order
  |
  +-- Payment
  +-- Inventory
  +-- Notification
```

These are parallel child operations.

---

# 38. Fan-In

Multiple operations can contribute to one process.

Example:

```
Service A
   |
   +------+
          |
Service B |----→ Aggregator
          |
Service C |
   |
   +------+
```

The aggregator combines results.

Tracing can help understand the relationship between these operations.

---

# 39. Sequential vs Parallel Spans

Sequential:

```
A
|
↓
B
|
↓
C
```

Parallel:

```
A
|
+----→ B
|
+----→ C
```

The waterfall view makes these differences visible.

---

# 40. Trace Waterfall

A waterfall visualization may look like:

```
Order API       |-----------------------|

  Payment API      |----------------|

  Database             |------|

  Inventory API              |------|
```

The horizontal length represents duration.

The position shows when the operation occurred.

---

# 41. Reading a Waterfall

When investigating a slow trace:

```
1. Find the longest span.

2. Check whether it is sequential or parallel.

3. Identify its parent.

4. Identify its downstream dependencies.

5. Check errors.

6. Check attributes.

7. Correlate with logs.
```

---

# 42. Trace Duration

Trace duration represents the overall request duration.

Example:

```
Trace Duration:
3.2 seconds
```

Individual spans:

```
Order:
100 ms

Payment:
2.8 sec

Inventory:
200 ms
```

The total trace duration provides the user-visible perspective.

---

# 43. Span Duration vs Trace Duration

Trace duration:

```
Complete request duration
```

Span duration:

```
Individual operation duration
```

Example:

```
Trace:
5 seconds

Payment Span:
3 seconds

Database Span:
2 seconds
```

The spans explain where the trace time was spent.

---

# 44. Overlapping Spans

Spans can overlap.

Example:

```
Order
|--------------------------|

Payment
   |-----------|

Inventory
   |---------|
```

Payment and Inventory may execute concurrently.

Therefore:

```
Total Trace Duration
```

is not necessarily:

```
Sum of Every Span Duration
```

---

# 45. Span Timing Example

Suppose:

```
Order:
0 ms → 1000 ms

Payment:
100 ms → 700 ms

Inventory:
200 ms → 800 ms
```

Payment and Inventory overlap.

Trace duration:

```
1000 ms
```

not:

```
1000 + 600 + 600
```

---

# 46. Span Attributes vs Events

Attributes describe the span.

Example:

```
http.request.method=POST
```

Events represent something that happened during the span.

Example:

```
retry_started
```

Both provide context but serve different purposes.

---

# 47. Span Attributes vs Logs

Span attributes:

```
Operation metadata
```

Logs:

```
Detailed application events
```

Example:

```
Span:
payment-service

Attribute:
status_code=500

Log:
payment failed because provider timed out
```

The two can complement each other.

---

# 48. Trace ID in Logs

A strong observability architecture includes:

```
trace_id
```

in application logs.

Example:

```
{
  "service": "payment-service",
  "event": "payment_failed",
  "trace_id": "abc123"
}
```

Then:

```
Log
  ↓
trace_id
  ↓
Jaeger
  ↓
Trace
```

---

# 49. Span ID in Logs

You can also include:

```
span_id
```

Example:

```
{
  "trace_id": "abc123",
  "span_id": "span456"
}
```

This allows even more precise correlation between logs and spans.

---

# 50. Span Context

Span context identifies the current tracing position.

Conceptually:

```
Trace ID
Span ID
Trace Flags
Trace State
```

The context is propagated between services.

---

# 51. Current Span

At any point during request processing, an application can have a current span.

Example:

```
Incoming Request
   ↓
Current Span
   ↓
Business Logic
   ↓
Database Span
```

The database span can be created as a child of the current span.

---

# 52. Creating a Child Span

Conceptually:

```
Parent Span
   |
   +-- Start Child Span
   |
   +-- Perform Operation
   |
   +-- End Child Span
   |
   +-- Continue Parent
   |
   +-- End Parent
```

The tracing SDK manages the relationship.

---

# 53. Span Lifecycle

A typical span lifecycle is:

```
Start Span
   ↓
Set Attributes
   ↓
Execute Operation
   ↓
Record Events
   ↓
Record Error if Required
   ↓
Set Status
   ↓
End Span
```

---

# 54. Span Creation

A span should start close to the beginning of the operation it represents.

Example:

```
Start Span
   ↓
Database Operation
   ↓
End Span
```

This allows accurate duration measurement.

---

# 55. Ending a Span

A span should end when the operation completes.

Success:

```
Operation
   ↓
End Span
   ↓
Status OK
```

Failure:

```
Operation
   ↓
Error
   ↓
Record Error
   ↓
End Span
   ↓
Status ERROR
```

---

# 56. Recording Errors

When an operation fails, useful information can include:

```
Error Type
Error Message
Stack Trace
Status
```

Avoid exposing sensitive information in error attributes.

---

# 57. Exceptions and Spans

An application exception can be associated with the current span.

Example:

```
Span:
process_payment

Exception:
TimeoutError

Status:
ERROR
```

This helps identify failures directly inside the trace.

---

# 58. Stack Traces

For application failures, stack traces can provide useful debugging information.

However:

```
Stack traces
    +
Sensitive Data
```

must be reviewed carefully before being exported.

---

# 59. Span Recording

A tracing SDK can record:

```
Span Attributes
Span Events
Span Status
Exceptions
```

The completed span is then exported to the tracing pipeline.

---

# 60. Span Export

Typical flow:

```
Application
   ↓
OpenTelemetry SDK
   ↓
Span
   ↓
OTLP
   ↓
OpenTelemetry Collector
   ↓
Jaeger
```

---

# 61. OTLP

OTLP stands for:

```
OpenTelemetry Protocol
```

It is used to transport OpenTelemetry telemetry.

It can be used for:

```
Traces
Metrics
Logs
```

For tracing:

```
Application
   ↓
OTLP
   ↓
Collector
```

---

# 62. Batch Processing

Sending every span individually can create unnecessary overhead.

Collectors commonly use batching.

Conceptually:

```
Span
Span
Span
Span
   ↓
Batch
   ↓
Backend
```

Batching can improve efficiency.

---

# 63. Span Sampling

Not every span necessarily needs to be stored.

Sampling can reduce:

```
Network
CPU
Storage
Backend Load
```

Sampling strategies must preserve important traces.

---

# 64. Trace-Based Sampling

When possible, make sampling decisions with awareness of the complete trace.

Useful criteria:

```
Error
High Latency
Specific Service
Specific Route
Critical Transaction
```

---

# 65. Error Trace Priority

Example:

```
Successful Request
    ↓
Sample 10%

Failed Request
    ↓
Keep 100%
```

This preserves useful troubleshooting information.

---

# 66. Slow Trace Priority

Example:

```
Normal:
< 500 ms

Slow:
> 2 seconds
```

A production sampling strategy could prioritize traces exceeding the latency threshold.

The exact threshold depends on application SLOs.

---

# 67. Business-Critical Trace Priority

Some requests are more important.

Examples:

```
Payment
Checkout
Order Creation
Authentication
```

These workflows may deserve higher trace retention or sampling priority.

---

# 68. Span Attributes and Cardinality

Attributes with many unique values can increase storage and query cost.

Examples:

```
user_id
transaction_id
request_id
```

Use high-cardinality values carefully.

---

# 69. Do Not Put Secrets in Span Attributes

Never add:

```
Password
API Token
Access Token
Private Key
Secret
```

to spans.

Tracing data must follow the same security principles as logs.

---

# 70. Span Size

Avoid creating extremely large spans.

Large spans can contain:

```
Huge Request Bodies
Huge Response Bodies
Large Objects
Large Stack Traces
```

This increases telemetry cost.

---

# 71. Meaningful Span Attributes

Good:

```
service.name
service.version
deployment.environment
http.request.method
http.response.status_code
db.system
```

Poor:

```
complete_request_body
complete_response_body
password
authorization_token
```

---

# 72. Service Name

Every service should have a consistent service name.

Example:

```
order-service
payment-service
inventory-service
```

The tracing backend uses service names to identify service boundaries.

---

# 73. Service Version

Include application version where useful.

Example:

```
service.version=1.5.2
```

This allows traces to be correlated with deployments.

---

# 74. Deployment Environment

Include:

```
development
staging
production
```

Example:

```
deployment.environment=production
```

This prevents confusion when investigating multiple environments.

---

# 75. Resource Attributes

Resource attributes describe the entity producing telemetry.

Examples:

```
service.name
service.version
deployment.environment
cloud.provider
cloud.region
```

They provide context for the telemetry source.

---

# 76. Kubernetes Resource Context

In Kubernetes, useful information may include:

```
Namespace
Pod
Container
Node
Cluster
```

Example:

```
service.name=payment-service
namespace=production
cluster=prod-eks
```

---

# 77. AWS Context

For AWS environments, useful resource context can include:

```
Cloud Provider
Region
Account / Environment Context
Kubernetes Cluster
```

Avoid exposing sensitive infrastructure information unnecessarily.

---

# 78. Span Naming Best Practices

Good span names should be:

```
Consistent
Meaningful
Low Cardinality
```

Good:

```
GET /orders/{id}
```

Bad:

```
GET /orders/12345
```

The second creates a unique span name for every order.

---

# 79. Avoid High-Cardinality Span Names

Bad:

```
GET /orders/10001
GET /orders/10002
GET /orders/10003
```

Better:

```
GET /orders/{id}
```

Dynamic identifiers should normally be attributes rather than part of the operation name.

---

# 80. Span Naming for Database Operations

Prefer meaningful low-cardinality names.

Example:

```
SELECT orders
```

rather than including every unique query parameter.

The exact naming depends on instrumentation and database technology.

---

# 81. Span Naming for Messaging

Examples:

```
publish order.created
consume order.created
```

Use stable operation names.

Avoid:

```
publish order.created.12345
```

because the dynamic ID creates unnecessary cardinality.

---

# 82. Span Naming for Business Operations

Good:

```
process_order

reserve_inventory

process_payment
```

Avoid:

```
process_order_12345
```

Keep identifiers in attributes when appropriate.

---

# 83. Trace Sampling and Business Operations

Suppose:

```
process_payment
```

is business-critical.

You may choose a sampling policy that preserves a higher percentage of payment traces.

This can be valuable during payment-related incidents.

---

# 84. Trace Relationships

Common relationships include:

```
Parent → Child
```

and:

```
Span Link
```

Parent-child relationships are useful for normal request flows.

Links are useful when operations have more complex relationships.

---

# 85. Span Links in Asynchronous Systems

Example:

```
Request A
   ↓
Queue
   ↓
Batch Consumer
```

A consumer may process work originating from multiple requests.

Span links can help represent those relationships without forcing an incorrect single-parent model.

---

# 86. Batch Processing

Consider:

```
100 Messages
     ↓
Batch Consumer
     ↓
One Processing Operation
```

The tracing model needs to represent relationships between the batch processing operation and the messages that triggered it.

Span links can be useful here.

---

# 87. Trace Across RabbitMQ

Example:

```
Order Service
   |
   | Producer Span
   ↓
RabbitMQ
   |
   | Consumer Span
   ↓
Notification Service
```

The trace context can be propagated through message metadata.

---

# 88. Trace Across External APIs

Example:

```
Payment Service
    |
    | Client Span
    ↓
External Payment Provider
```

The client span can record:

```
Operation
Duration
Status
```

Sensitive request information should not be recorded unnecessarily.

---

# 89. Trace Across Databases

Example:

```
Order Service
   |
   +-- SQL Span
          |
          +-- PostgreSQL
```

Useful information:

```
Database System
Operation
Duration
Status
```

---

# 90. Trace Across Cache

Example:

```
Order Service
   |
   +-- Redis GET
```

A cache span can help identify:

```
Cache Latency
Cache Errors
Slow Cache Operations
```

---

# 91. Trace Across Kubernetes Services

Example:

```
Order Pod
   ↓
Kubernetes Service
   ↓
Payment Pod
```

Application-level trace context should propagate through the service-to-service request.

Kubernetes networking itself does not automatically create an end-to-end application trace.

---

# 92. Trace Through ALB / Ingress

Architecture:

```
Client
  ↓
ALB
  ↓
Kubernetes Ingress / Service
  ↓
Application
```

The application tracing layer should preserve incoming trace context where supported.

The exact capabilities depend on the ingress/load-balancer configuration and instrumentation.

---

# 93. Trace Context Boundary

Every distributed boundary is important.

Examples:

```
HTTP
gRPC
Messaging
External APIs
```

At each boundary:

```
Extract Context
    ↓
Create / Continue Span
    ↓
Inject Context
```

---

# 94. Extract Context

When a service receives a request, it extracts trace context from the incoming request.

Conceptually:

```
Incoming Request
    ↓
traceparent
    ↓
Extract Context
```

The service can then continue the trace.

---

# 95. Inject Context

When a service makes an outgoing request, it injects trace context.

Example:

```
Order Service
    ↓
Inject Trace Context
    ↓
Payment Service
```

This allows the downstream service to continue the trace.

---

# 96. Context Propagation Flow

Complete flow:

```
Client
   |
   | traceparent
   ↓
Order Service
   |
   | traceparent
   ↓
Payment Service
   |
   | traceparent
   ↓
Inventory Service
```

The same trace can therefore span multiple services.

---

# 97. Broken Context Propagation

Problem:

```
Order Service
Trace = abc123

Payment Service
Trace = xyz789

Inventory Service
Trace = qwe456
```

The tracing backend sees unrelated traces.

Root cause:

```
Context propagation failure.
```

---

# 98. Debugging Broken Trace Context

Check:

```
Incoming Headers
traceparent
Instrumentation
Context Extraction
Context Injection
HTTP Client
HTTP Server
Messaging Metadata
```

Also verify that middleware is correctly configured.

---

# 99. Trace Context Through Proxies

Proxies and gateways can affect headers.

Architecture:

```
Client
   ↓
Proxy
   ↓
Service
```

Verify that trace propagation headers are preserved and correctly handled.

---

# 100. Trace Context Security

Trace context is not a secret.

However, it should still be treated as trusted application metadata.

Do not use tracing headers to transport:

```
Passwords
Tokens
Sensitive User Data
```

---

# 101. Span Events for Retries

Example:

```
Payment Span
    |
    +-- retry_started
    |
    +-- retry_started
    |
    +-- payment_success
```

This can show retry behavior without creating unnecessary top-level spans.

---

# 102. Span Events for Timeouts

Example:

```
External API Span
    |
    +-- timeout_detected
```

Span status:

```
ERROR
```

Attributes may describe the operation and timeout condition.

---

# 103. Span Events for Cache Behavior

Example:

```
Order Processing Span
    |
    +-- cache_miss
```

Then:

```
Database Span
```

This helps explain why a database operation occurred.

---

# 104. Span Events for Business Decisions

Example:

```
Order Span
    |
    +-- inventory_reserved
    |
    +-- payment_authorized
```

Business events can be useful when they provide meaningful troubleshooting context.

---

# 105. Avoid Excessive Span Events

Do not record every trivial application event.

Good:

```
retry_started

timeout_detected
```

Bad:

```
variable_created

loop_started

loop_ended
```

Keep telemetry operationally useful.

---

# 106. Trace Error Investigation

Example:

```
Trace
  |
  +-- Order
  |
  +-- Payment
         |
         +-- External API
                |
                ERROR
```

The failed child span helps identify the likely failure boundary.

---

# 107. Trace Latency Investigation

Example:

```
Trace:
4.5 seconds

Order:
100 ms

Payment:
3.8 seconds

Inventory:
200 ms
```

The Payment span is the primary latency candidate.

---

# 108. Trace Dependency Investigation

Example:

```
Order
  |
  +-- Payment
  |      |
  |      +-- PostgreSQL
  |
  +-- Inventory
         |
         +-- Redis
```

The trace shows the dependency structure.

---

# 109. Service Dependency Graph

Repeated traces can reveal:

```
Order
  ↓
Payment
  ↓
External Provider
```

and:

```
Order
  ↓
Inventory
  ↓
Redis
```

This helps teams understand the architecture.

---

# 110. Trace-Based SLO Investigation

Suppose:

```
Checkout SLO:
99% < 2 seconds
```

Metrics:

```
p99 = 3 seconds
```

Traces can show:

```
Payment dependency
    |
    ↓
2.5 seconds
```

This helps identify why the SLO is being violated.

---

# 111. Trace-Based Incident Investigation

During an incident:

```
1. Identify affected endpoint.

2. Search slow or failed traces.

3. Inspect root span.

4. Identify longest child span.

5. Inspect dependency.

6. Check errors.

7. Correlate logs.

8. Check metrics.

9. Check deployment.

10. Mitigate.
```

---

# 112. Traces During Deployment

Before deployment:

```
version=1.5.1
```

After deployment:

```
version=1.5.2
```

Compare:

```
Trace Duration
Error Rate
Dependency Latency
```

This can expose regressions.

---

# 113. Canary Tracing

During a canary:

```
v1 → 95%
v2 → 5%
```

Compare traces for:

```
v1
v2
```

Check:

```
Latency
Errors
Dependency Behavior
```

If v2 performs significantly worse, investigate before increasing traffic.

---

# 114. Blue-Green Tracing

Example:

```
Blue:
v1.5.1

Green:
v1.5.2
```

Trace metadata can identify which environment/version handled the request.

This helps compare behavior before switching traffic.

---

# 115. Rollback Investigation

Suppose:

```
v1.5.2
```

introduced:

```
Payment latency
```

Tracing shows:

```
payment-service
    |
    +-- External API
          |
          +-- 4 sec
```

After rollback:

```
v1.5.1
    |
    +-- External API
          |
          +-- 300 ms
```

The traces provide evidence for the rollback decision.

---

# 116. Trace Retention

Not every trace needs long-term retention.

A practical strategy may be:

```
Normal traces:
Short retention

Error traces:
Longer retention

Critical traces:
Special retention
```

The exact policy depends on:

```
Cost
Business Requirements
Compliance
Incident Needs
```

---

# 117. Trace Storage Growth

Trace storage depends on:

```
Requests/sec
Spans/request
Average span size
Sampling rate
Retention
```

Example:

```
10,000 requests/sec
   ×
8 spans/request
   =
80,000 spans/sec
```

Even modest span sizes can create significant telemetry volume.

---

# 118. Trace Cost Optimization

Use:

```
Sampling

Retention Policies

Efficient Attributes

Batch Processing

Filtering

Appropriate Instrumentation
```

Avoid:

```
Huge Spans

Excessive Events

Excessive Custom Instrumentation
```

---

# 119. Instrumentation Overhead

Tracing can introduce overhead through:

```
CPU
Memory
Network
Serialization
```

Monitor application performance after enabling instrumentation.

---

# 120. Tracing Should Be Non-Critical

A key reliability principle:

```
Application
   ↓
Should continue operating
```

even if:

```
Tracing Backend
   ↓
Becomes unavailable
```

Telemetry is important, but it should not become a critical runtime dependency for normal request processing.

---

# 121. Collector Backpressure

Suppose:

```
Application:
50,000 spans/sec

Collector:
40,000 spans/sec
```

Then:

```
Backlog increases
```

The collector should be monitored for:

```
Queue Size
Export Failures
Dropped Spans
CPU
Memory
```

---

# 122. Collector Scaling

If trace volume increases:

```
Collector
   ↓
Scale Replicas
```

For Kubernetes:

```
Deployment
   ↓
Horizontal Scaling
```

The exact scaling mechanism depends on the deployment architecture.

---

# 123. Collector High Availability

Production design:

```
Application
   |
   +----→ Collector 1
   |
   +----→ Collector 2
   |
   +----→ Collector 3
             |
             ↓
           Jaeger
```

Multiple collectors reduce the risk of a single collector failure.

---

# 124. Jaeger Availability

The tracing backend should also be designed according to production requirements.

Consider:

```
Multiple Components
Persistent Storage
Replication
Backup
Access Control
Monitoring
```

Avoid a single-instance design for critical production tracing unless the availability requirement permits it.

---

# 125. Trace Backend Monitoring

Monitor:

```
Ingestion Rate
Storage
Query Latency
Errors
Availability
```

The tracing platform itself needs observability.

---

# 126. Trace Search

Useful search dimensions include:

```
Service
Operation
Duration
Status
Trace ID
Attributes
```

Example investigation:

```
Service:
payment-service

Status:
ERROR

Duration:
> 2s
```

---

# 127. Search Slow Traces

Example:

```
service=payment-service

duration > 2s
```

Then inspect:

```
Longest Spans

Failed Spans

Dependencies

Attributes
```

---

# 128. Search Error Traces

Example:

```
service=order-service

status=ERROR
```

Then:

```
Open Trace
   ↓
Identify Failed Span
   ↓
Inspect Error
   ↓
Check Logs
```

---

# 129. Search by Trace ID

If a log contains:

```
trace_id=abc123
```

search for:

```
abc123
```

in the tracing backend.

This directly connects:

```
Log
```

to:

```
Trace
```

---

# 130. Trace and Logging Example

Log:

```
{
  "service": "payment-service",
  "level": "ERROR",
  "event": "payment_timeout",
  "trace_id": "abc123",
  "span_id": "span456"
}
```

Jaeger:

```
Trace:
abc123

Span:
span456
```

The engineer can directly correlate the two.

---

# 131. Trace and Metrics Example

Prometheus:

```
payment_request_duration_seconds
↑
```

Logs:

```
payment_timeout
```

Trace:

```
payment-service
    |
    +-- external-payment-api
            |
            +-- 3.5 sec
```

This provides three levels of visibility.

---

# 132. Trace and Grafana

A unified Grafana workflow can allow:

```
Metrics
   ↓
Logs
   ↓
Traces
```

For example:

```
Grafana Alert
     ↓
High Latency
     ↓
Open Related Logs
     ↓
Open Related Trace
     ↓
Investigate
```

---

# 133. Tracing in Java Applications

For Java microservices, tracing can be implemented using OpenTelemetry instrumentation.

Typical targets:

```
HTTP Server
HTTP Client
JDBC
Messaging
Framework Operations
```

The goal is to capture important service boundaries.

---

# 134. Tracing in Node.js Applications

Node.js services can be instrumented with OpenTelemetry.

Typical targets:

```
HTTP
Express / Framework
Database
HTTP Client
Messaging
```

Context propagation should be preserved across asynchronous operations.

---

# 135. Tracing in Python Applications

Python services can use OpenTelemetry instrumentation.

Typical targets:

```
HTTP Server
HTTP Client
Database
Messaging
```

As with other languages, trace context must propagate correctly.

---

# 136. Multi-Language Distributed Tracing

Example:

```
Java
  ↓
Node.js
  ↓
Python
  ↓
PostgreSQL
```

OpenTelemetry provides a common observability model across languages.

All services can participate in the same trace.

---

# 137. Cross-Language Trace Example

Trace:

```
abc123
```

Java:

```
Order Service
trace_id=abc123
```

Node.js:

```
Payment Service
trace_id=abc123
```

Python:

```
Notification Service
trace_id=abc123
```

This creates one end-to-end distributed trace.

---

# 138. Trace Context Across Languages

The propagation format should be standardized.

For example:

```
Java
   ↓
W3C Trace Context
   ↓
Node.js
   ↓
W3C Trace Context
   ↓
Python
```

This prevents fragmented traces.

---

# 139. Testing Distributed Tracing

Test:

```
HTTP Request

Service-to-Service Call

Database Operation

External API

Message Queue

Error

Timeout

Retry

Context Propagation
```

---

# 140. Trace Testing

For a test request:

```
POST /orders
```

Verify:

```
Trace Created

Root Span Created

Payment Span Created

Inventory Span Created

Database Span Created

Trace IDs Match

Parent-Child Relationships Correct
```

---

# 141. Context Propagation Test

Test:

```
Service A
   ↓
Service B
   ↓
Service C
```

Expected:

```
Trace ID A = Trace ID B = Trace ID C
```

If not:

```
Context propagation is broken.
```

---

# 142. Error Trace Test

Force:

```
Payment Failure
```

Expected:

```
Payment Span
   ↓
ERROR
```

Verify:

```
Error Recorded

Trace Searchable

Trace ID Available in Logs
```

---

# 143. Timeout Trace Test

Force:

```
External API Timeout
```

Expected:

```
External API Span
   ↓
Long Duration
   ↓
Error / Timeout
```

This verifies latency investigation capabilities.

---

# 144. Database Trace Test

Execute:

```
Database Query
```

Verify:

```
Database Span

Correct Duration

Database Context

Correct Parent
```

---

# 145. Messaging Trace Test

Test:

```
Producer
   ↓
RabbitMQ
   ↓
Consumer
```

Verify:

```
Producer Span

Message Context

Consumer Span

Trace Relationship
```

---

# 146. Load Testing Tracing

During load testing measure:

```
Requests/sec
Spans/sec
Collector CPU
Collector Memory
Export Errors
Dropped Spans
Backend Storage
```

This identifies tracing capacity limits.

---

# 147. Distributed Tracing Troubleshooting

If no traces appear:

```
1. Check instrumentation.

2. Check SDK configuration.

3. Check OTLP endpoint.

4. Check Collector.

5. Check Collector receiver.

6. Check Exporter.

7. Check Jaeger.

8. Check network connectivity.

9. Check sampling.

10. Check backend storage.
```

---

# 148. If Only One Service Appears

Suppose:

```
Order Service
   ↓
Payment Service
   ↓
Inventory Service
```

Jaeger shows only:

```
Order Service
```

Possible causes:

```
Missing Instrumentation

Context Propagation Failure

Export Failure

Sampling

Incorrect SDK Configuration
```

---

# 149. If Trace Is Broken Between Services

Check:

```
traceparent header

HTTP client instrumentation

HTTP server instrumentation

Context extraction

Context injection

Middleware

Proxy behavior
```

---

# 150. If Traces Are Too Large

Check:

```
Number of Spans

Span Events

Attributes

Request Bodies

Response Bodies

Custom Instrumentation
```

Then reduce unnecessary telemetry.

---

# 151. If Trace Latency Looks Wrong

Check:

```
Clock Synchronization

Span Start Time

Span End Time

Parent-Child Relationships

Instrumentation

Async Operations
```

Distributed systems can produce confusing timing if clocks or instrumentation are incorrect.

---

# 152. If Database Spans Are Missing

Check:

```
Database Instrumentation

Database Client Library

OpenTelemetry SDK

Export Pipeline

Sampling
```

The application may have HTTP tracing enabled but database instrumentation disabled.

---

# 153. If Logs Cannot Be Correlated

Check:

```
trace_id in logs

span_id where useful

Logging Framework

OpenTelemetry Context

Log Formatter
```

The logging system should retrieve the current tracing context.

---

# 154. Production Span Design

A production span should generally contain:

```
Operation Name

Start Time

End Time

Duration

Trace ID

Span ID

Parent Context

Status

Relevant Attributes

Relevant Events
```

It should avoid:

```
Secrets

Huge Payloads

Unnecessary High-Cardinality Data
```

---

# 155. Production Trace Design

A production trace should provide:

```
Complete Request Flow

Service Relationships

Latency

Errors

Dependency Information

Trace Context

Deployment Context
```

The trace should be useful during an incident without producing unnecessary telemetry.

---

# 156. Production Span Checklist

```
[ ] Meaningful span name

[ ] Correct parent

[ ] Correct Trace ID

[ ] Unique Span ID

[ ] Start time

[ ] End time

[ ] Duration

[ ] Status

[ ] Useful attributes

[ ] Error information

[ ] Relevant events

[ ] No secrets

[ ] No unnecessary payloads
```

---

# 157. Production Trace Checklist

```
[ ] Root span exists

[ ] Child spans are connected

[ ] Context propagation works

[ ] HTTP tracing works

[ ] Database tracing works

[ ] Messaging tracing works

[ ] External dependency tracing works

[ ] Logs contain Trace ID

[ ] Sampling configured

[ ] Retention configured

[ ] Collector monitored

[ ] Backend monitored
```

---

# 158. Interview Question: What Is a Span?

### Answer

A span represents one unit of work inside a distributed trace.

For example, if an order request calls:

```
Order Service
   ↓
Payment Service
   ↓
PostgreSQL
```

the trace can contain spans for:

```
Order API
Payment API
Database Query
```

Each span contains information such as:

```
Start Time
Duration
Operation
Status
Attributes
```

---

# 159. Interview Question: What Is the Difference Between a Span and a Trace?

### Answer

A trace represents the complete journey of a request.

A span represents one operation within that request.

For example:

```
Trace
  |
  +-- Order API
  +-- Payment API
  +-- Database
```

The trace is the complete request.

Each item is an individual span.

---

# 160. Interview Question: What Is a Root Span?

### Answer

The root span is the span at the beginning of a trace.

For an HTTP request:

```
Client
   ↓
POST /orders
```

the server handling the incoming request may create the root span.

Child operations then appear below it.

---

# 161. Interview Question: What Is a Child Span?

### Answer

A child span represents an operation performed as part of a parent span.

Example:

```
Order API
   |
   +-- Payment API
   |
   +-- Database
```

Payment and Database operations can be child spans of the Order operation.

---

# 162. Interview Question: What Are Span Attributes?

### Answer

Span attributes provide additional metadata about an operation.

Examples:

```
HTTP Method
HTTP Status
Service Name
Database System
Deployment Environment
Service Version
```

They allow engineers to search and analyze traces.

---

# 163. Interview Question: What Are Span Events?

### Answer

Span events represent important events that happen during a span.

For example:

```
Span:
Payment

Events:

retry_started

timeout_detected

fallback_triggered
```

They provide additional diagnostic context.

---

# 164. Interview Question: How Do You Find the Slowest Service?

### Answer

I would open the trace and inspect the waterfall.

I would identify:

```
Longest Span
```

Then check:

```
Parent Span

Child Spans

External Dependencies

Database Operations

Errors
```

For example:

```
Order:
100 ms

Payment:
3 seconds

Inventory:
200 ms
```

The Payment span is the primary latency candidate.

---

# 165. Interview Question: Why Are Span Names Important?

### Answer

Span names are used for:

```
Search
Aggregation
Dashboards
Analysis
```

They should be meaningful and low-cardinality.

For example:

```
GET /orders/{id}
```

is better than:

```
GET /orders/12345
```

because the second creates a unique operation name for every order.

---

# 166. Interview Question: What Is Span Sampling?

### Answer

Span or trace sampling reduces the amount of tracing telemetry that must be processed and stored.

At high traffic levels, collecting every trace may be expensive.

I would prioritize:

```
Error Traces

Slow Traces

Critical Transactions
```

and sample normal successful traffic where appropriate.

---

# 167. Interview Question: How Do You Correlate a Span With Logs?

### Answer

I include:

```
trace_id
```

and optionally:

```
span_id
```

in structured logs.

Example:

```
{
  "service": "payment-service",
  "trace_id": "abc123",
  "span_id": "span456"
}
```

Then I can search the trace backend using the Trace ID and correlate the exact operation with application logs.

---

# 168. Interview Question: How Do You Trace an Async Workflow?

### Answer

For asynchronous systems, I propagate trace context through message metadata.

Example:

```
Producer
   ↓
RabbitMQ
   ↓
Consumer
```

The producer creates a producer span.

The consumer creates a consumer span.

For more complex relationships such as batch processing, span links may be appropriate.

---

# 169. Interview Question: What Happens If a Downstream Service Creates a New Trace?

### Answer

The end-to-end trace becomes fragmented.

For example:

```
Order Service
Trace A

Payment Service
Trace B
```

I would investigate:

```
Context Propagation

traceparent

HTTP Client Instrumentation

HTTP Server Instrumentation

Middleware

Context Extraction / Injection
```

The goal is for the downstream service to continue the original trace.

---

# 170. Interview Question: How Would You Implement Spans in an EKS Environment?

### Answer

I would instrument the application services using OpenTelemetry.

Applications would export traces using OTLP.

The traces would be sent to an OpenTelemetry Collector running in EKS.

The collector would process and export them to Jaeger.

Architecture:

```
Java / Node.js / Python
        ↓
OpenTelemetry SDK
        ↓
       OTLP
        ↓
OpenTelemetry Collector
        ↓
      Jaeger
```

I would monitor both the collector and tracing backend.

---

# 171. Real-World Example: Slow Payment

Request:

```
POST /orders
```

Trace:

```
Order API
0 ms → 4000 ms
```

Child spans:

```
Payment API
100 ms → 3900 ms

External Provider
200 ms → 3800 ms

Database
100 ms → 300 ms
```

Investigation:

```
Payment Service
    ↓
External Provider
    ↓
High Latency
```

Logs using the same Trace ID:

```
payment_timeout
```

Metrics:

```
Payment latency ↑
```

This gives strong evidence that the external dependency is the primary latency contributor.

---

# 172. Real-World Example: Database Failure

Trace:

```
Order API
   |
   +-- Database
          |
          +-- ERROR
```

Span:

```
duration=3000 ms
```

Status:

```
ERROR
```

Log:

```
database_timeout
```

Metric:

```
DB latency ↑
```

The engineer can correlate:

```
Trace
Logs
Metrics
```

to investigate the database problem.

---

# 173. Real-World Example: Service-to-Service Failure

Request:

```
Client
  ↓
Order
  ↓
Payment
  ↓
Inventory
```

Trace:

```
Order
  |
  +-- Payment
         |
         +-- ERROR
```

Payment logs:

```
error_code=PAYMENT_TIMEOUT
```

The trace identifies the failure boundary.

The logs provide detailed error information.

---

# 174. Real-World Example: Retry Storm

Trace:

```
Payment
   |
   +-- External API
   |
   +-- retry_started
   |
   +-- retry_started
   |
   +-- timeout_detected
```

The trace shows:

```
Initial Request
Retry
Retry
Timeout
```

Metrics:

```
External API latency ↑
```

Logs:

```
dependency_retry
```

This helps identify a retry storm.

---

# 175. Real-World Example: Deployment Regression

Before deployment:

```
version=1.5.1
p95 trace latency=400 ms
```

After deployment:

```
version=1.5.2
p95 trace latency=1800 ms
```

Trace analysis:

```
payment-service
    |
    +-- external API
          |
          +-- 1400 ms
```

This points to a regression introduced or exposed by the new version.

---

# 176. Real-World Example: Multi-Language Trace

Architecture:

```
Java Order Service
      ↓
Node.js Payment Service
      ↓
Python Notification Service
      ↓
RabbitMQ
```

Trace:

```
trace_id=abc123
```

Java:

```
abc123
```

Node.js:

```
abc123
```

Python:

```
abc123
```

The common OpenTelemetry model allows the request to remain one distributed trace across different languages.

---

# 177. Spans and Traces Mental Model

Remember:

```
Trace
   |
   +-- Root Span
          |
          +-- Child Span
          |      |
          |      +-- Database Span
          |
          +-- Child Span
                 |
                 +-- External API Span
```

Each span contains:

```
Name
Start
End
Duration
Status
Attributes
Events
Context
```

The trace connects everything.

---

# 178. Complete Span Lifecycle

```
Incoming Request
      ↓
Extract Context
      ↓
Create Span
      ↓
Set Attributes
      ↓
Execute Operation
      ↓
Create Child Spans
      ↓
Record Events
      ↓
Record Error if Needed
      ↓
Set Status
      ↓
End Span
      ↓
Export Trace
      ↓
OpenTelemetry Collector
      ↓
Jaeger
```

---

# 179. Final Mental Model

The most important concepts are:

```
Trace
Trace ID
Span
Span ID
Parent Span
Child Span
Root Span
Span Context
Span Attributes
Span Events
Span Status
Span Links
Span Kind
Span Duration
Sampling
Context Propagation
```

Think of the system as:

```
One User Request
       ↓
    One Trace
       ↓
Multiple Operations
       ↓
   Multiple Spans
       ↓
Parent / Child Relationships
       ↓
Context Propagation
       ↓
OpenTelemetry
       ↓
Collector
       ↓
   Jaeger
```

The purpose of spans is not simply to record that an operation happened.

The purpose is to understand:

```
Where the request went

How long each operation took

Which dependency was involved

Where an error occurred

How operations are related

Why the complete request was slow or failed
```
