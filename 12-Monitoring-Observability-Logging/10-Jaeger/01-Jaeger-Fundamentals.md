# Jaeger Fundamentals

## 1. Overview

Jaeger is a distributed tracing platform used to monitor and troubleshoot requests as they travel through distributed applications and microservices.

It helps engineers understand:

```text
Request
   ↓
Service A
   ↓
Service B
   ↓
Service C
   ↓
Database / External API
```

Instead of investigating each service independently, Jaeger provides a view of the complete request flow.

A typical OpenTelemetry-based architecture is:

```text
Application
     ↓
OpenTelemetry SDK / Instrumentation
     ↓
Trace / Spans
     ↓
OpenTelemetry Collector
     ↓
Jaeger
     ↓
Jaeger UI
```

---

# 2. What Problem Does Jaeger Solve?

In a monolithic application, troubleshooting a request can be relatively straightforward:

```text
User
 ↓
Application
 ↓
Database
```

In a microservices architecture:

```text
User
 ↓
Frontend
 ↓
Orders
 ↓
Payment
 ↓
Inventory
 ↓
Notification
 ↓
Database
```

A single request can cross many services.

If the request takes 5 seconds, engineers need to determine:

```text
Which service was slow?
Which dependency was slow?
Where did the error occur?
Was the database responsible?
Was an external API responsible?
```

Jaeger helps answer these questions by visualizing distributed traces.

---

# 3. What Is Distributed Tracing?

Distributed tracing follows a request across multiple services.

Example:

```text
User
 ↓
Frontend
 ↓
Orders
 ↓
Payment
 ↓
Inventory
```

The complete request is represented as a:

```text
Trace
```

Individual operations inside that trace are represented as:

```text
Spans
```

Therefore:

```text
Trace
 ├── Span
 ├── Span
 ├── Span
 └── Span
```

---

# 4. Trace

A trace represents the complete journey of one request.

Example:

```text
Trace ID: abc123

Frontend
   ↓
Orders
   ↓
Payment
   ↓
Database
```

All related spans belong to the same trace.

A trace can therefore answer:

```text
What happened to this request?
```

---

# 5. Span

A span represents one operation.

Examples:

```text
HTTP request
Database query
External API call
Message processing
Business operation
```

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

Each span has timing and contextual information.

---

# 6. Trace ID

A Trace ID identifies the complete distributed request.

Example:

```text
trace_id = 4bf92f3577b34da6a3ce929d0e0e4736
```

Multiple spans can share the same Trace ID:

```text
Frontend → Trace ID ABC
Orders   → Trace ID ABC
Payment  → Trace ID ABC
```

Jaeger uses this relationship to reconstruct the complete trace.

---

# 7. Span ID

Each span has its own Span ID.

Example:

```text
Trace ID = ABC

Frontend
Span ID = 111

Orders
Span ID = 222

Payment
Span ID = 333
```

Therefore:

```text
Trace ID
   ↓
Entire request

Span ID
   ↓
Individual operation
```

---

# 8. Parent-Child Relationship

Spans can have parent-child relationships.

Example:

```text
Frontend
   │
   └── Orders
          │
          └── Payment
```

Relationship:

```text
Orders
parent = Frontend

Payment
parent = Orders
```

This creates the trace hierarchy.

---

# 9. Root Span

The root span is the starting span of a trace.

Example:

```text
Root Span
GET /checkout
      │
      ├── Orders
      ├── Payment
      └── Inventory
```

The root span normally represents the incoming request or top-level operation.

---

# 10. Child Spans

Child spans represent operations performed under another operation.

Example:

```text
Orders
│
├── Validate Order
├── Database Query
├── Payment Request
└── Inventory Request
```

This gives deeper visibility into the service.

---

# 11. Trace Waterfall

One of Jaeger's most useful visualizations is the trace waterfall.

Example:

```text
Checkout        |████████████████████| 1000ms
Orders             |███████████████  | 750ms
Payment                   |███████   | 350ms
Database                     |███     | 150ms
```

This helps identify where time was spent.

---

# 12. Latency Investigation

Suppose users report that checkout is slow.

Metrics show:

```text
Checkout p95 latency = 1.8 seconds
```

Jaeger shows:

```text
Checkout
   ↓
Orders       200ms
Payment      1.4s
Inventory    100ms
```

Then Payment:

```text
Payment
   ↓
External Gateway
   ↓
1.2 seconds
```

The external gateway becomes the primary investigation target.

---

# 13. Error Investigation

Suppose users receive:

```text
HTTP 500
```

Jaeger may show:

```text
Frontend       200
Orders         200
Payment        500
Database       Error
```

The trace immediately identifies the failing portion of the request.

---

# 14. Jaeger Components

A modern Jaeger deployment can be understood as several logical components:

```text
Instrumentation
      ↓
Trace Collection
      ↓
Jaeger
      ↓
Storage
      ↓
Jaeger Query / UI
```

With OpenTelemetry:

```text
Application
      ↓
OpenTelemetry
      ↓
Collector
      ↓
Jaeger
      ↓
UI
```

The exact deployment architecture depends on the Jaeger version and selected storage/ingestion configuration.

---

# 15. Jaeger UI

Jaeger provides a web interface for exploring traces.

The UI allows engineers to:

```text
Search traces
Filter traces
View trace duration
Inspect spans
Inspect tags/attributes
View errors
Inspect service relationships
Analyze dependencies
```

---

# 16. Jaeger Trace Search

Typical search dimensions include:

```text
Service
Operation
Tags
Duration
Time range
Trace status
```

Example:

```text
Service:
payment

Operation:
POST /payments

Duration:
> 1s
```

This helps locate slow operations.

---

# 17. Service Selection

A large microservices environment may contain:

```text
frontend
orders
payment
inventory
notification
user
```

Jaeger allows engineers to focus on a specific service.

Example:

```text
Service = payment
```

Then investigate traces involving that service.

---

# 18. Operation Selection

A service can have multiple operations:

```text
GET /payments
POST /payments
GET /payments/{id}
POST /refund
```

Searching by operation helps narrow the investigation.

Example:

```text
Service:
payment

Operation:
POST /payments
```

---

# 19. Trace Duration

Duration is an important search criterion.

Example:

```text
Duration > 2s
```

This can identify slow traces.

Then the engineer can inspect:

```text
Root span
Child spans
Database spans
External API spans
```

---

# 20. Span Details

Selecting a span can reveal information such as:

```text
Operation name
Start time
Duration
Tags / attributes
Events
Process information
References
Status / errors
```

This provides detailed context for troubleshooting.

---

# 21. Span Attributes

Attributes provide additional information about a span.

Examples:

```text
http.request.method = POST
http.route = /api/orders
http.response.status_code = 500
service.name = orders
```

Database example:

```text
db.system = postgresql
db.operation = SELECT
```

The exact attributes depend on instrumentation and semantic conventions.

---

# 22. Span Events

Events represent notable occurrences during a span.

Example:

```text
Payment Span
│
├── Start
├── payment_retry
├── gateway_response
└── End
```

Events can provide additional context without creating another child span.

---

# 23. Span Status

A span can indicate whether an operation succeeded or failed.

Conceptually:

```text
OK
ERROR
UNSET
```

Example:

```text
Payment
status = ERROR
```

Jaeger can then display the failed operation within the trace.

---

# 24. Exception Information

Instrumentation can record exceptions on spans.

Example:

```text
Payment Span
│
├── Exception
│     └── TimeoutError
│
└── Status = ERROR
```

This makes application failures easier to investigate.

---

# 25. Service Dependency Graph

Distributed tracing can provide service dependency information.

Example:

```text
Frontend
   ↓
Orders
   ├── Payment
   └── Inventory

Payment
   ↓
Payment Gateway

Orders
   ↓
Database
```

This helps engineers understand the architecture and identify dependencies.

---

# 26. Microservices Architecture With Jaeger

Consider:

```text
                    ALB
                     │
                     ↓
                 Frontend
                     │
                     ↓
                  Orders
                /        \
               ↓          ↓
           Payment     Inventory
              │            │
              ↓            ↓
          Gateway        Database
```

Jaeger can represent a request across these services as one distributed trace.

---

# 27. Trace Context

Distributed tracing requires context propagation.

Important information includes:

```text
Trace ID
Span ID
Trace flags
```

The context travels between services.

Example:

```text
Orders
   ↓
HTTP Request
   ↓
traceparent
   ↓
Payment
```

Payment extracts the context and creates a child span.

---

# 28. W3C Trace Context

OpenTelemetry commonly uses W3C Trace Context.

Important headers:

```text
traceparent
tracestate
```

Example concept:

```text
traceparent:
00-TRACE_ID-SPAN_ID-01
```

This allows different services and instrumentation libraries to participate in the same trace.

---

# 29. Jaeger and OpenTelemetry

OpenTelemetry and Jaeger have different roles.

```text
OpenTelemetry
   ↓
Telemetry generation and collection standard
```

Jaeger:

```text
Tracing backend and UI
```

Therefore:

```text
Application
   ↓
OpenTelemetry
   ↓
Jaeger
```

is a common architecture.

---

# 30. OpenTelemetry + Jaeger Architecture

```text
Application
     ↓
OpenTelemetry SDK
     ↓
Spans
     ↓
OTLP
     ↓
OpenTelemetry Collector
     ↓
Jaeger
     ↓
Jaeger UI
```

This separates application instrumentation from the tracing backend.

---

# 31. Why Use OpenTelemetry With Jaeger?

Without OpenTelemetry:

```text
Application
     ↓
Jaeger-specific instrumentation
     ↓
Jaeger
```

With OpenTelemetry:

```text
Application
     ↓
OpenTelemetry
     ↓
Collector
     ↓
Jaeger
```

The second architecture provides greater backend flexibility.

The same OpenTelemetry telemetry can potentially be routed to another compatible tracing backend later.

---

# 32. OTLP

OTLP is the OpenTelemetry Protocol.

Application:

```text
Application
     ↓
OTLP
     ↓
Collector
```

The Collector can then export the telemetry to Jaeger or another supported backend.

Common OTLP ports are:

```text
4317 → OTLP/gRPC
4318 → OTLP/HTTP
```

The actual endpoint depends on the deployment configuration.

---

# 33. Jaeger Data Flow

A modern OpenTelemetry-based flow:

```text
Application
    ↓
OTel SDK
    ↓
OTLP
    ↓
Collector
    ↓
Jaeger ingestion
    ↓
Storage
    ↓
Jaeger Query
    ↓
Jaeger UI
```

Each stage has a different responsibility.

---

# 34. Instrumentation

Instrumentation generates spans.

Examples:

```text
HTTP
Database
Messaging
RPC
Frameworks
```

Automatic instrumentation can cover common libraries.

Manual instrumentation can add business-specific operations.

---

# 35. Automatic Instrumentation

Automatic instrumentation can create spans for supported libraries.

Example:

```text
Java Application
     ↓
OTel Java Agent
     ↓
HTTP Span
JDBC Span
Messaging Span
```

This reduces application code changes.

---

# 36. Manual Instrumentation

Manual instrumentation creates custom spans.

Example conceptual flow:

```text
Start Span
    ↓
Validate Order
    ↓
Process Payment
    ↓
End Span
```

Manual spans are useful for important business operations that automatic instrumentation cannot understand.

---

# 37. Combining Automatic and Manual Instrumentation

Production applications can use:

```text
Automatic
    +
Manual
```

Example:

```text
HTTP request
     ↓
Automatic Span
     ↓
process-order
     ↓
Manual Span
     ↓
Database
     ↓
Automatic DB Span
```

This gives both infrastructure-level and business-level visibility.

---

# 38. Trace Sampling

High-volume applications can generate large numbers of traces.

Example:

```text
1,000,000 requests/day
```

If every request is retained:

```text
Large trace volume
High storage
High network usage
High backend cost
```

Sampling can reduce the retained volume.

---

# 39. Head Sampling

Head sampling makes the sampling decision near the beginning of the trace.

```text
Request
   ↓
Sampler
 ┌─┴─┐
Keep Drop
```

Advantages:

```text
Simple
Low overhead
Early reduction
```

Disadvantage:

```text
The sampler may not yet know whether the request will later fail or become slow.
```

---

# 40. Tail Sampling

Tail sampling waits until sufficient trace information is available.

```text
Trace
 ↓
Collect
 ↓
Analyze
 ↓
Decision
 ├── Keep
 └── Drop
```

Useful policies:

```text
Errors → Keep
Slow traces → Keep
Normal traces → Sample
```

Tail sampling is commonly performed at the Collector layer.

---

# 41. Jaeger and Sampling

Jaeger receives the traces that survive the configured collection and sampling strategy.

A common architecture:

```text
Application
   ↓
OTel Agent
   ↓
OTel Gateway
   ↓
Tail Sampling
   ↓
Jaeger
```

This reduces the volume sent to Jaeger.

---

# 42. Trace Retention

Trace retention depends on:

```text
Trace volume
Storage capacity
Incident requirements
Cost
Compliance
```

Typical strategy:

```text
Normal traces
   ↓
Shorter retention

Error / important traces
   ↓
Longer retention where required
```

The exact retention policy should be defined according to operational requirements.

---

# 43. Jaeger Storage

Jaeger requires storage for trace data.

Depending on the deployment/version, storage can use supported backends such as:

```text
Elasticsearch
OpenSearch
Other supported storage options
```

The specific storage architecture should be selected based on scale and operational requirements.

---

# 44. Jaeger and Elasticsearch

A common architecture:

```text
Application
     ↓
OTel Collector
     ↓
Jaeger
     ↓
Elasticsearch
     ↓
Jaeger Query
     ↓
Jaeger UI
```

Elasticsearch provides persistent storage and search capabilities.

---

# 45. Trace Storage Considerations

Storage planning should consider:

```text
Traces per second
Spans per trace
Average span size
Sampling rate
Retention period
Replication
Indexing
Query workload
```

Example:

```text
High traffic
+
Long retention
+
100% sampling
=
Large storage requirement
```

---

# 46. Jaeger Query

Jaeger Query is responsible for retrieving trace data for the UI.

Conceptually:

```text
Jaeger UI
    ↓
Jaeger Query
    ↓
Trace Storage
```

The Query component retrieves matching traces and presents them to the UI.

---

# 47. Jaeger UI

The UI provides:

```text
Trace search
Trace visualization
Span details
Service information
Operation information
Latency analysis
Error inspection
```

A typical workflow:

```text
Select Service
      ↓
Select Operation
      ↓
Search traces
      ↓
Open trace
      ↓
Inspect waterfall
      ↓
Inspect spans
```

---

# 48. Finding Slow Requests

Example search:

```text
Service:
orders

Operation:
POST /orders

Duration:
> 1s
```

Open a result:

```text
Orders
   ↓
Payment
   ↓
Database
```

The waterfall identifies which operation contributed most to latency.

---

# 49. Finding Errors

Search for:

```text
Service = payment
```

Then inspect failed traces.

Example:

```text
Payment
   ↓
External Gateway
   ↓
Timeout
```

Inspect:

```text
Span status
Exception
Attributes
Duration
Logs
```

---

# 50. Trace-to-Log Correlation

If logs contain:

```text
trace_id
span_id
```

then Jaeger and ELK can be used together.

Workflow:

```text
Jaeger
  ↓
Trace ID
  ↓
Kibana
  ↓
Related logs
```

This is extremely useful during incidents.

---

# 51. Trace-to-Metric Correlation

Metrics detect abnormal behavior.

Example:

```text
payment_latency_p95 ↑
```

Then Jaeger:

```text
Payment
   ↓
Gateway
   ↓
Slow
```

Workflow:

```text
Grafana
   ↓
Problem detected
   ↓
Jaeger
   ↓
Root cause investigation
```

---

# 52. Metrics + Logs + Traces

A complete troubleshooting workflow:

```text
             Observability
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
    Metrics       Logs       Traces
       │           │           │
       ↓           ↓           ↓
    Grafana      Kibana      Jaeger
```

Each signal provides different information.

```text
Metrics → What is wrong?
Logs    → What happened?
Traces  → Where did it happen?
```

---

# 53. Jaeger in Kubernetes

A basic Kubernetes architecture:

```text
EKS
 │
 ├── Application Pods
 │      ↓
 │   OTel SDK
 │
 ├── OTel Collector
 │
 └── Jaeger
        ↓
      Storage
```

A more scalable architecture:

```text
Application Pods
      ↓
OTel Agent
      ↓
OTel Gateway
      ↓
Jaeger
      ↓
Storage
      ↓
Jaeger Query
      ↓
Jaeger UI
```

---

# 54. Jaeger Deployment Types

Jaeger can be deployed in different ways depending on requirements.

For development:

```text
Single Jaeger deployment
```

For production:

```text
Separate components
Multiple replicas
External durable storage
Load balancing
```

The exact components depend on the Jaeger release and deployment model.

---

# 55. Development Architecture

For local testing:

```text
Application
     ↓
OpenTelemetry
     ↓
Jaeger
     ↓
Jaeger UI
```

This is useful for:

```text
Instrumentation testing
Trace visualization
Developer debugging
```

It is not automatically a production architecture.

---

# 56. Production Architecture

A production design:

```text
Applications
      ↓
OTel Agents
      ↓
OTel Gateways
      ↓
Jaeger Ingestion
      ↓
Durable Storage
      ↓
Jaeger Query
      ↓
Jaeger UI
```

Components should be independently scalable where required.

---

# 57. Kubernetes Service Discovery

Applications need a stable Collector endpoint.

Example:

```text
otel-gateway.observability.svc.cluster.local
```

Architecture:

```text
Application
     ↓
Kubernetes DNS
     ↓
OTel Gateway Service
     ↓
Gateway Pods
```

Kubernetes automatically distributes traffic across healthy Service endpoints.

---

# 58. Jaeger Service Discovery

Similarly, applications or Collectors need a stable Jaeger endpoint.

```text
Collector
   ↓
Kubernetes Service
   ↓
Jaeger
```

Avoid hard-coding individual Pod IP addresses.

Pods are ephemeral.

---

# 59. Kubernetes Metadata

Traces should identify their Kubernetes origin where useful.

Examples:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
```

Combined with:

```text
service.name
service.version
deployment.environment
```

this provides strong deployment context.

---

# 60. Trace Example in EKS

Suppose:

```text
service.name = payment
service.version = v2.4.0
deployment.environment = production
k8s.namespace.name = production
k8s.pod.name = payment-7d9f8c
```

A Jaeger trace can therefore identify:

```text
Which service?
Which version?
Which environment?
Which Pod?
```

This is extremely useful after deployments.

---

# 61. Jaeger During Deployment

Before deployment:

```text
payment v2.3.0
p95 = 250ms
```

After deployment:

```text
payment v2.4.0
p95 = 900ms
```

Jaeger can show:

```text
v2.4.0
   ↓
Database span increased
```

This helps identify release regressions.

---

# 62. Canary Deployment With Jaeger

Suppose:

```text
v1 → 90%
v2 → 10%
```

Compare traces:

```text
v1
 ├── Latency
 └── Errors

v2
 ├── Latency
 └── Errors
```

If v2 shows significantly worse behavior:

```text
Rollback
```

---

# 63. Jaeger and GitOps

Jaeger configuration should be managed through GitOps where practical.

```text
Git
 ↓
Jaeger / OTel configuration
 ↓
Pull Request
 ↓
Review
 ↓
CI
 ↓
ArgoCD
 ↓
EKS
```

Benefits:

```text
Version control
Review
Auditability
Rollback
Drift detection
```

---

# 64. Jaeger and Terraform

Terraform can provision infrastructure required for Jaeger.

Example:

```text
Terraform
   ↓
EKS
   ↓
Networking
   ↓
IAM
   ↓
Storage
```

Kubernetes application deployment can then be managed through:

```text
Helm
ArgoCD
Kubernetes manifests
```

depending on the team's platform model.

---

# 65. Jaeger Resource Requirements

Jaeger components require:

```text
CPU
Memory
Network
Storage
```

Resource requirements depend heavily on:

```text
Trace volume
Sampling
Retention
Storage backend
Query volume
```

Production sizing should come from measured workload data.

---

# 66. Jaeger High Availability

Avoid:

```text
Single Jaeger component
```

for critical production environments.

Use multiple replicas where supported and appropriate:

```text
Jaeger Query
├── Replica 1
├── Replica 2
└── Replica 3
```

Similarly, ingestion components and storage should have appropriate redundancy.

---

# 67. Storage High Availability

If Jaeger uses an external storage backend:

```text
Jaeger
   ↓
Storage
```

the storage backend itself must be highly available.

Otherwise:

```text
Jaeger healthy
      ↓
Storage unavailable
      ↓
Trace queries fail
```

The tracing platform is only as reliable as its storage architecture.

---

# 68. Trace Backend Failure

If Jaeger becomes unavailable:

```text
Application
   ↓
OTel Collector
   ↓
Jaeger
   X
```

The application should continue functioning.

Collector-side:

```text
Queue
Retry
Backpressure
Sampling
```

can reduce impact during temporary backend failures.

---

# 69. Jaeger Query Failure

If Query is unavailable:

```text
Trace ingestion
     ↓
May continue
```

but:

```text
Jaeger UI
     ↓
Cannot retrieve traces
```

Therefore ingestion and query should be treated as separate availability concerns.

---

# 70. Jaeger UI Failure

If the UI is unavailable:

```text
Trace data
     ↓
May still exist in storage
```

Restoring the UI/Query layer can restore access to the stored traces.

This separation is useful in production architecture.

---

# 71. Monitoring Jaeger

Monitor:

```text
Jaeger CPU
Jaeger memory
Request rate
Query latency
Error rate
Ingestion rate
Storage health
Pod restarts
```

Also monitor:

```text
OTel Collector
Storage backend
Kubernetes
```

The complete tracing system needs observability.

---

# 72. Jaeger Alerts

Potential alerts:

```text
Jaeger unavailable
High query error rate
High query latency
Ingestion failures
Storage unavailable
Pod restart rate high
CPU saturation
Memory saturation
```

Alerts should be based on meaningful production thresholds.

---

# 73. Security

Tracing data can contain:

```text
URLs
HTTP attributes
Database metadata
Error messages
Service information
```

It can potentially expose sensitive information if instrumentation is poorly configured.

Protect Jaeger using:

```text
Authentication
Authorization
TLS
NetworkPolicy
Private networking
Access control
```

---

# 74. Sensitive Trace Data

Do not capture:

```text
Passwords
Access tokens
Authorization headers
Private keys
Credit card data
Sensitive request bodies
```

Review automatic instrumentation settings carefully.

---

# 75. Trace Attribute Control

Avoid unnecessarily capturing:

```text
Full HTTP body
Large responses
Sensitive headers
Uncontrolled user data
```

Prefer useful operational attributes:

```text
HTTP route
Status
Service
Operation
Duration
Error type
```

---

# 76. Jaeger Access Control

Different teams may require different levels of access.

For example:

```text
Developers
Platform Engineers
Security Team
Operations
```

Use the organization's identity and access-control mechanisms to restrict access appropriately.

---

# 77. Network Isolation

Jaeger should normally remain inside controlled network boundaries.

Example:

```text
Internet
   X
   │
   │ no direct public access
   ↓
Jaeger
```

Instead:

```text
Internal Network
     ↓
Jaeger
```

Expose the UI externally only through an appropriately secured access path if required.

---

# 78. Performance Considerations

Tracing overhead comes from:

```text
Span creation
Context propagation
Attribute processing
Serialization
Network export
Storage
```

Reduce overhead through:

```text
Sampling
Batching
Efficient instrumentation
Controlled attributes
Collector processing
```

---

# 79. High Cardinality

Avoid uncontrolled trace attributes such as:

```text
user_id
session_id
request_id
```

when they are not needed for the operational use case.

Prefer:

```text
service.name
route
method
status
environment
version
```

and other controlled dimensions.

---

# 80. Trace Retention Strategy

Example:

```text
Normal traces
     ↓
7 days

Important error traces
     ↓
Longer retention if required
```

The actual period should be based on:

```text
Incident investigation
Compliance
Cost
Storage capacity
Business requirements
```

---

# 81. Trace Cost Optimization

Main controls:

```text
Sampling
Retention
Span volume
Attribute size
Storage tier
Query workload
```

A common production strategy:

```text
Normal traces
   ↓
Low sampling

Slow traces
   ↓
Keep

Error traces
   ↓
Keep
```

---

# 82. Troubleshooting: No Traces

Check:

```text
1. Is instrumentation enabled?
2. Is the application creating spans?
3. Is OTLP endpoint correct?
4. Is Collector receiving traces?
5. Is the traces pipeline enabled?
6. Is sampling dropping traces?
7. Is the exporter working?
8. Is Jaeger receiving data?
9. Is storage healthy?
10. Is Jaeger Query working?
11. Is the UI searching the correct service/time range?
```

---

# 83. Troubleshooting: Traces Missing Between Services

Suppose:

```text
Orders Trace
   ↓
Payment Trace
```

appear as separate traces.

Check:

```text
Trace context propagation
traceparent
HTTP instrumentation
Messaging instrumentation
Context extraction
Context injection
```

The likely problem is broken propagation.

---

# 84. Troubleshooting: Slow Jaeger UI

Possible causes:

```text
Large query range
High trace volume
Slow storage
Large traces
High query concurrency
Insufficient Query resources
```

Actions:

```text
Narrow time range
Filter service
Filter operation
Check storage
Scale Query
Review retention
```

---

# 85. Troubleshooting: High Storage Usage

Possible causes:

```text
100% trace sampling
Long retention
Large spans
Too many attributes
High request volume
```

Solutions:

```text
Sampling
Retention policies
Attribute reduction
Storage optimization
Capacity planning
```

---

# 86. Troubleshooting: High Collector Memory

Possible causes:

```text
Backend unavailable
Queue growth
Tail sampling
High trace volume
Large span attributes
```

Check:

```text
Collector metrics
Collector logs
Queue size
Backend health
Sampling policy
```

---

# 87. Troubleshooting: Trace IDs Missing From Logs

Check:

```text
Logging integration
Active span context
Context propagation
OpenTelemetry instrumentation
Log bridge
```

Expected:

```text
Trace
 ↓
Span
 ↓
Log
trace_id = same
span_id = same
```

---

# 88. Troubleshooting: Wrong Service Name

If Jaeger shows:

```text
unknown-service
```

check:

```text
OTEL_SERVICE_NAME
Resource attributes
SDK configuration
Collector resource processor
Instrumentation configuration
```

Each microservice should have a meaningful service name.

---

# 89. Troubleshooting: Duplicate Traces

Possible causes:

```text
Duplicate instrumentation
Multiple telemetry paths
Application exports directly to backend
Application exports through Collector
Multiple collectors collecting the same data
```

Example:

```text
Application
 ├──→ Collector
 └──→ Jaeger
```

If both paths export the same spans, duplicates can appear.

Prefer one intentional telemetry path.

---

# 90. Production Jaeger Checklist

```text
[ ] OpenTelemetry instrumentation configured
[ ] OTLP configured
[ ] Collector configured
[ ] Agent/Gateway architecture defined
[ ] Sampling configured
[ ] Batch processing configured
[ ] Resource attributes configured
[ ] Kubernetes metadata available
[ ] Jaeger components deployed
[ ] Storage configured
[ ] Storage capacity planned
[ ] Query replicas configured where required
[ ] UI access secured
[ ] TLS configured where required
[ ] RBAC/access control configured
[ ] NetworkPolicy configured
[ ] Sensitive data filtered
[ ] High-cardinality attributes reviewed
[ ] Retention configured
[ ] Collector monitored
[ ] Jaeger monitored
[ ] Storage monitored
[ ] Backup/recovery strategy defined
[ ] GitOps deployment configured
[ ] Upgrade strategy defined
[ ] Rollback strategy tested
```

---

# 91. Real-World EKS Troubleshooting Flow

Suppose the application has high latency.

Start:

```text
Grafana
   ↓
Latency increase
```

Then:

```text
Jaeger
   ↓
Find slow trace
```

Trace:

```text
Orders
   ↓
Payment
   ↓
External Gateway
   ↓
1.8 seconds
```

Then:

```text
Kibana
   ↓
Payment gateway timeout logs
```

Then:

```text
Kubernetes
   ↓
Payment Pods healthy
```

Conclusion:

```text
External dependency latency
```

This is a practical observability workflow.

---

# 92. Jaeger With Prometheus and Grafana

A complete monitoring stack can be:

```text
                Applications
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    Metrics         Logs         Traces
       │             │             │
       ↓             ↓             ↓
  Prometheus          ELK         Jaeger
       ↓             ↓             ↓
    Grafana         Kibana      Jaeger UI
```

Use each tool for its strongest purpose.

```text
Prometheus → Metrics
Grafana    → Visualization
ELK        → Logs
Jaeger     → Traces
```

---

# 93. Jaeger With OpenTelemetry

The recommended mental model is:

```text
OpenTelemetry
   ↓
Generate / Collect / Process
   ↓
Jaeger
   ↓
Store / Search / Visualize traces
```

OpenTelemetry is the telemetry framework.

Jaeger is the tracing platform.

---

# 94. Migration Flexibility

With OpenTelemetry:

```text
Application
     ↓
OTel
     ↓
Collector
     ↓
Jaeger
```

Later, the backend can potentially change:

```text
Application
     ↓
OTel
     ↓
Collector
     ↓
Another Trace Backend
```

The application does not need to be rewritten around a backend-specific instrumentation model.

---

# 95. Production Architecture Summary

```text
                           EKS
                            │
        ┌───────────────────┼───────────────────┐
        ↓                   ↓                   ↓
     Orders              Payment            Inventory
        │                   │                   │
     OTel SDK            OTel SDK            OTel SDK
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ↓
                     OTel Agent
                      DaemonSet
                            ↓
                     OTel Gateway
                     Multiple Pods
                            ↓
                       Sampling
                            ↓
                         Jaeger
                            ↓
                         Storage
                            ↓
                      Jaeger Query
                            ↓
                         Jaeger UI
```

---

# 96. Final Mental Model

Remember Jaeger as:

```text
REQUEST
   ↓
TRACE
   ↓
SPANS
   ↓
TRACE CONTEXT
   ↓
DISTRIBUTED SERVICES
   ↓
OTLP
   ↓
OTEL COLLECTOR
   ↓
SAMPLING / PROCESSING
   ↓
JAEGER
   ↓
STORAGE
   ↓
JAEGER QUERY
   ↓
JAEGER UI
```

For a production EKS microservices platform:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTel Agent
     ↓
OTel Gateway
     ↓
Tail Sampling
     ↓
Jaeger
     ↓
Durable Storage
     ↓
Jaeger Query
     ↓
Jaeger UI
```

The key principle is:

**Jaeger provides distributed tracing visibility, while OpenTelemetry provides the standardized instrumentation and telemetry pipeline. In a production Kubernetes environment, applications should generate traces through OpenTelemetry, propagate trace context across HTTP and messaging boundaries, send telemetry through scalable Collector Agents and Gateways, apply controlled sampling and batching, securely deliver traces to Jaeger, use durable and highly available storage, and correlate traces with Prometheus metrics and ELK logs to investigate latency, failures, deployment regressions, and distributed-system behavior.**
