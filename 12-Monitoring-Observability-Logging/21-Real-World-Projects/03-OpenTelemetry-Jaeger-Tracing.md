# OpenTelemetry + Jaeger Distributed Tracing

> Production-style distributed tracing project for an AWS EKS microservices platform using OpenTelemetry and Jaeger.
>
> This project explains how to instrument applications, propagate trace context, collect spans, process and export telemetry, visualize traces, correlate traces with logs and metrics, troubleshoot production failures, scale the tracing platform, and explain the architecture in DevOps interviews.

---

# 1. Project Overview

## Project Name

**Distributed Tracing for EKS Microservices using OpenTelemetry and Jaeger**

## Objective

Build a production-oriented distributed tracing platform for microservices running on Kubernetes/EKS.

The platform should help engineers answer:

```text
Which service received the request?
Which downstream service was slow?
Where did the request fail?
How long did each dependency take?
Which database/API caused the latency?
Was the problem introduced by a deployment?
```

Core technologies:

```text
AWS EKS
Kubernetes
Docker
OpenTelemetry
OpenTelemetry SDKs
OpenTelemetry Collector
Jaeger
```

---

# 2. Why Distributed Tracing?

Consider:

```text
Client
  |
  v
ALB / Ingress
  |
  v
Order Service
  |
  +----> User Service
  |
  +----> Payment Service
  |
  +----> Inventory Service
             |
             +----> Database
```

The user sees:

```text
Request took 4.8 seconds
```

Metrics can tell us:

```text
Order latency increased
```

Logs can tell us:

```text
Payment timeout
```

Tracing can show the complete request path:

```text
Order Service       120 ms
   |
   +--> User Service       80 ms
   |
   +--> Payment Service   3.9 sec
             |
             +--> Database 3.7 sec
```

This identifies the bottleneck much faster.

---

# 3. Metrics vs Logs vs Traces

## Metrics

Answer:

```text
What is happening?
```

Examples:

```text
CPU
Memory
Request rate
Error rate
Latency
```

## Logs

Answer:

```text
What happened?
```

Examples:

```text
Database timeout
Payment failed
Authentication error
```

## Traces

Answer:

```text
Where did the request spend time?
```

Examples:

```text
Order -> Payment -> Database
```

A production observability platform combines all three.

---

# 4. Real-World Scenario

Assume a microservices application:

```text
                         Client
                           |
                           v
                     AWS ALB / Ingress
                           |
                           v
                    Order Service
                    /     |      \
                   /      |       \
                  v       v        v
             User       Payment   Inventory
             Service    Service    Service
                          |
                          v
                       Database
```

A single user request may cross multiple services.

Tracing creates one distributed trace containing spans from those services.

---

# 5. Production Architecture

Recommended architecture:

```text
                         Users
                           |
                           v
                     ALB / Ingress
                           |
                           v
                    Microservices
                 /       |        \
                /        |         \
               v         v          v
             User      Order      Payment
               \         |          /
                \        |         /
                 +-------+---------+
                         |
                         v
                OpenTelemetry SDK
                         |
                         v
                OTLP instrumentation
                         |
                         v
              OpenTelemetry Collector
                         |
                         v
                       Jaeger
                         |
                         v
                    Jaeger UI
```

A more production-oriented design separates application instrumentation from backend storage:

```text
Application
    |
    v
OTel SDK / Agent
    |
    v
OTel Collector
    |
    +--> Sampling
    +--> Batching
    +--> Enrichment
    +--> Filtering
    +--> Routing
    |
    v
Jaeger
    |
    v
Trace Storage
    |
    v
Jaeger UI
```

---

# 6. Important Design Principle

Applications should not depend directly on Jaeger.

Prefer:

```text
Application
     |
     v
OpenTelemetry
     |
     v
Collector
     |
     v
Jaeger
```

instead of:

```text
Application
     |
     v
Jaeger directly
```

Why?

Because OpenTelemetry provides a vendor-neutral telemetry layer.

You can later route telemetry to:

```text
Jaeger
Tempo
Commercial observability platforms
Other OTLP-compatible backends
```

without completely redesigning application instrumentation.

---

# 7. OpenTelemetry Components

OpenTelemetry consists of several concepts:

```text
API
SDK
Instrumentation
Collector
Exporters
Semantic conventions
Context propagation
```

For this project:

```text
Application
 |
 +--> OTel instrumentation
 |
 +--> OTel SDK
 |
 v
OTel Collector
 |
 v
Jaeger
```

---

# 8. Jaeger Role

Jaeger provides:

```text
Trace collection
Trace storage integration
Trace search
Trace visualization
Trace analysis
```

The Jaeger UI allows engineers to inspect:

```text
Trace
 |
 +--> Span
 +--> Span
 +--> Span
```

and understand timing and dependencies.

---

# 9. What Is a Trace?

A trace represents one end-to-end request or transaction.

Example:

```text
Trace ID:
4f8a7c...
```

The trace contains multiple spans:

```text
Trace
 |
 +--> Order Service
 |
 +--> User Service
 |
 +--> Payment Service
 |
 +--> Database
```

---

# 10. What Is a Span?

A span represents one operation.

Example:

```text
Span:
payment-service POST /payments
```

A span usually contains:

```text
Trace ID
Span ID
Parent Span ID
Start time
Duration
Name
Attributes
Events
Status
Links
```

---

# 11. Parent and Child Spans

Example:

```text
Order Request
    |
    +--> User Service
    |
    +--> Payment Service
             |
             +--> Database
```

Relationships:

```text
Order span
 |
 +--> User span
 |
 +--> Payment span
          |
          +--> DB span
```

This forms the trace tree.

---

# 12. Trace ID

A Trace ID identifies the entire distributed request.

Example:

```text
trace_id=abc123
```

Every span in the same trace has the same Trace ID.

---

# 13. Span ID

Each span has its own Span ID.

Example:

```text
Trace ID: abc123

Order:
span_id=001

Payment:
span_id=002

Database:
span_id=003
```

---

# 14. Parent Span ID

The parent relationship identifies who initiated the current operation.

Example:

```text
Order
  |
  +--> Payment
         |
         +--> Database
```

Payment's parent:

```text
Order span
```

Database's parent:

```text
Payment span
```

---

# 15. Distributed Trace

A distributed trace is created when context is propagated across service boundaries.

Example:

```text
Order Service
      |
      | trace context
      v
Payment Service
      |
      | trace context
      v
Database
```

Without propagation:

```text
Trace A
Trace B
Trace C
```

instead of:

```text
One connected trace
```

---

# 16. Trace Context

OpenTelemetry commonly uses W3C Trace Context.

Important headers:

```text
traceparent
tracestate
```

The `traceparent` header carries trace context across HTTP boundaries.

Concept:

```text
Client
 |
 | traceparent
 v
Service A
 |
 | traceparent
 v
Service B
```

---

# 17. Why Context Propagation Matters

Suppose:

```text
Order Service = 100 ms
Payment Service = 4 sec
Database = 3.8 sec
```

If trace context is propagated correctly, Jaeger shows them as one request.

If not:

```text
Order trace
Payment trace
Database trace
```

The relationship is lost.

---

# 18. W3C Trace Context Concept

Conceptual header:

```text
traceparent:
00-<trace-id>-<parent-id>-<flags>
```

The application framework or OpenTelemetry library normally manages this rather than engineers manually constructing it.

---

# 19. Sampling

Tracing every request can be expensive.

Suppose:

```text
10,000 requests/sec
```

Tracing all requests may produce significant telemetry volume.

Sampling decides which traces are retained.

Common approaches:

```text
Head sampling
Tail sampling
```

---

# 20. Head Sampling

Decision is made near the beginning of the trace.

Example:

```text
Receive request
      |
      v
Sampling decision
      |
      +--> Keep
      |
      +--> Drop
```

Advantages:

```text
Simple
Low overhead
Predictable volume
```

Disadvantage:

```text
May drop an interesting trace before knowing whether it becomes an error.
```

---

# 21. Tail Sampling

Decision is made after more of the trace is available.

Example:

```text
Request
 |
 v
Collect spans
 |
 v
Inspect trace
 |
 +--> Error? Keep
 |
 +--> Slow? Keep
 |
 +--> Normal? Maybe sample
```

This is powerful for production.

For example:

```text
Keep 100% errors
Keep slow traces
Keep 5% normal traces
```

The OpenTelemetry Collector is commonly used for this type of processing.

---

# 22. Sampling Strategy

A practical production policy:

```text
Errors        -> 100%
Very slow     -> 100%
Normal        -> 5-10%
Health checks -> low/filtered
```

Exact percentages depend on traffic and storage capacity.

---

# 23. Span Attributes

Attributes provide context.

Example:

```text
service.name = payment-service
deployment.environment = production
http.request.method = POST
http.response.status_code = 500
server.address = payment-service
```

Use standardized OpenTelemetry semantic conventions where supported.

---

# 24. Avoid High-Cardinality Problems

Do not blindly attach:

```text
full request body
large response body
password
token
entire user object
```

to every span.

This increases:

```text
Telemetry volume
Storage
Privacy risk
Query cost
```

---

# 25. Span Events

A span can contain events.

Example:

```text
payment span
 |
 +--> event: payment_authorization_started
 |
 +--> event: payment_provider_timeout
```

Events are useful for significant points inside an operation.

---

# 26. Span Status

A span can represent:

```text
Unset
Ok
Error
```

For failures, record useful error information without leaking sensitive data.

---

# 27. Error Recording

Example conceptual span:

```text
Service:
payment-service

Status:
ERROR

Error type:
TimeoutError

Message:
Payment provider timeout
```

Avoid putting secrets into exception attributes.

---

# 28. Application Instrumentation

There are several approaches:

```text
Automatic instrumentation
Manual instrumentation
Framework instrumentation
SDK instrumentation
```

Prefer automatic instrumentation where it provides sufficient visibility.

Use manual spans for important business operations that automatic instrumentation cannot capture.

---

# 29. Automatic Instrumentation

Automatic instrumentation can capture common operations such as:

```text
HTTP server
HTTP client
Database
Messaging
Framework calls
```

This reduces application code changes.

---

# 30. Manual Instrumentation

Useful when you need:

```text
Business operation
Custom workflow
External dependency
Important internal function
```

Example:

```text
createOrder
validateInventory
processPayment
```

---

# 31. Java Application

For Java, OpenTelemetry can be introduced through supported Java instrumentation mechanisms.

Conceptual architecture:

```text
Java Application
      |
      v
OTel Java Agent / SDK
      |
      v
OTLP
      |
      v
OTel Collector
```

The agent approach can provide broad automatic instrumentation with limited source-code changes.

---

# 32. Node.js Application

Typical architecture:

```text
Node.js Application
       |
       v
OTel Node.js SDK
       |
       v
Instrumentation
       |
       v
OTLP
       |
       v
Collector
```

Instrument:

```text
HTTP
Express
Database
Redis
Messaging
```

according to supported instrumentation packages.

---

# 33. Python Application

Typical architecture:

```text
Python Application
       |
       v
OpenTelemetry Python
       |
       v
Instrumentation
       |
       v
OTLP
       |
       v
Collector
```

Common instrumentation targets:

```text
Flask
Django
FastAPI
Requests
Database clients
```

---

# 34. Instrumentation Strategy

Start with:

```text
HTTP ingress
HTTP downstream calls
Database
Messaging
```

Then add business-specific spans where required.

Do not instrument every function automatically without a reason.

---

# 35. Service Naming

Every service should have a stable:

```text
service.name
```

Example:

```text
user-service
order-service
payment-service
inventory-service
```

Do not use unstable pod names as the primary service identity.

---

# 36. Resource Attributes

Useful resource metadata:

```text
service.name
service.version
deployment.environment
cloud.provider
cloud.region
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.node.name
```

This allows filtering by environment and Kubernetes workload.

---

# 37. EKS Architecture

Example:

```text
AWS
 |
 +--> EKS
       |
       +--> Namespace: production
       |      |
       |      +--> order-service
       |      +--> payment-service
       |      +--> inventory-service
       |
       +--> OTel Collector
       |
       +--> Jaeger
```

---

# 38. Collector Deployment Models

OpenTelemetry Collector can be deployed as:

```text
DaemonSet
Deployment
Sidecar
Gateway
```

A common production architecture uses:

```text
DaemonSet / local collector
          |
          v
Gateway collector
          |
          v
Backend
```

This separates node-local collection from centralized processing.

---

# 39. Collector as DaemonSet

Advantages:

```text
Local collection
Node affinity
Low network hops
Automatic scaling with nodes
```

Useful when telemetry is generated by workloads on each node.

---

# 40. Collector as Deployment

Advantages:

```text
Centralized processing
Independent scaling
Gateway architecture
```

Useful for:

```text
Tail sampling
Routing
Exporting
Central policy
```

---

# 41. Recommended Production Collector Architecture

```text
Application
    |
    v
OTel SDK
    |
    v
Local Collector
    |
    v
Gateway Collector
    |
    +--> Sampling
    +--> Enrichment
    +--> Filtering
    +--> Batching
    |
    v
Jaeger
```

For smaller environments, a single collector tier may be sufficient.

---

# 42. Collector Pipeline

Concept:

```text
Receivers
    |
    v
Processors
    |
    v
Exporters
```

Example:

```text
OTLP Receiver
     |
     v
Memory Limiter
     |
     v
Batch
     |
     v
Tail Sampling
     |
     v
Jaeger Exporter
```

The exact component names and support depend on the collector distribution/version.

---

# 43. Receiver

Receives telemetry.

Common protocol:

```text
OTLP
```

Transport options may include:

```text
gRPC
HTTP
```

---

# 44. Processor

Processors can:

```text
Batch
Sample
Filter
Transform
Add attributes
Limit memory
```

Processors are critical for production control.

---

# 45. Batch Processor

Instead of sending every span immediately:

```text
Span
Span
Span
Span
```

the collector batches them:

```text
Batch
 |
 v
Exporter
```

Benefits:

```text
Better throughput
Reduced network overhead
More efficient backend ingestion
```

---

# 46. Memory Limiter

Protects the collector from uncontrolled memory growth.

Concept:

```text
Memory increases
      |
      v
Memory limiter
      |
      +--> Apply protection
```

This is especially important in Kubernetes.

---

# 47. Tail Sampling Processor

Concept:

```text
Receive complete traces
       |
       v
Evaluate policy
       |
       +--> Error -> Keep
       +--> Slow -> Keep
       +--> Normal -> Sample
```

This can reduce storage while retaining valuable traces.

---

# 48. Filter Processor

Use filtering carefully.

Potentially filter:

```text
Health-check spans
Low-value internal traffic
Known noisy endpoints
```

Do not accidentally drop:

```text
Critical business requests
Security events
Errors
```

---

# 49. Resource Detection

Collector processors can enrich telemetry with infrastructure metadata.

For Kubernetes:

```text
Cluster
Namespace
Pod
Node
Container
```

This makes traces easier to search.

---

# 50. Jaeger Deployment

Jaeger can be deployed using a supported Kubernetes deployment model appropriate to the chosen Jaeger version.

Consider:

```text
Collector
Query/UI
Storage
Networking
Authentication
Persistence
```

Do not treat a production tracing backend as a simple UI deployment.

---

# 51. Jaeger Components

Conceptually:

```text
Trace Producers
      |
      v
Jaeger Collector / OTel Collector
      |
      v
Storage
      |
      v
Jaeger Query
      |
      v
Jaeger UI
```

The exact architecture depends on the Jaeger release and deployment mode.

---

# 52. Jaeger Storage

Jaeger requires a trace storage backend.

Depending on the chosen deployment/version, storage can use supported backends such as:

```text
Elasticsearch-compatible storage
OpenSearch-compatible storage
Other supported storage options
```

Always verify compatibility for the exact Jaeger version.

---

# 53. Jaeger UI

The UI should allow engineers to:

```text
Search traces
Filter service
Filter operation
Filter duration
Inspect spans
Inspect errors
View service dependencies
```

---

# 54. Jaeger Service Search

Example:

```text
Service:
payment-service
```

Then:

```text
Operation:
POST /payments
```

Then filter:

```text
Duration > 2s
```

---

# 55. Trace Search by Duration

Suppose:

```text
Normal = 100 ms
Incident = 4 sec
```

Search for:

```text
duration > 2s
```

This quickly isolates slow requests.

---

# 56. Trace Search by Error

Search for traces containing:

```text
error
status = ERROR
HTTP 5xx
```

Then inspect:

```text
Failed span
Parent span
Downstream spans
```

---

# 57. Trace Waterfall

A trace may look like:

```text
Order Service       |---------|
User Service            |---|
Payment Service          |-------------|
Database                      |--------|
```

This makes latency visually obvious.

---

# 58. Reading a Trace

Start from the root span.

Ask:

```text
How long was the complete request?
```

Then:

```text
Which child span consumed most time?
```

Then:

```text
Did it fail?
```

Then:

```text
Which downstream dependency caused it?
```

---

# 59. Critical Path

The critical path is the sequence of operations contributing most directly to total request latency.

Example:

```text
Order
 |
 +--> User          50 ms
 |
 +--> Payment      900 ms
        |
        +--> DB    800 ms
```

Critical path:

```text
Order -> Payment -> DB
```

---

# 60. Parallel Spans

Some services may execute concurrently:

```text
Order
 |
 +--> User       100 ms
 |
 +--> Inventory  120 ms
 |
 +--> Pricing     80 ms
```

Total may be close to the longest parallel branch rather than the sum.

Tracing helps reveal this.

---

# 61. N+1 Dependency Problem

Example:

```text
Order
 |
 +--> DB query 1
 +--> DB query 2
 +--> DB query 3
 ...
 +--> DB query 100
```

Tracing can reveal repeated database calls.

This is often difficult to see from application-level metrics alone.

---

# 62. Slow Database Query

Trace:

```text
Order
 |
 +--> Payment
        |
        +--> DB query
                |
                +--> 3.8 seconds
```

This points the investigation toward the database layer.

---

# 63. External API Latency

Trace:

```text
Order
 |
 +--> Payment
        |
        +--> External Payment API
                |
                +--> 4 sec
```

The service itself may be healthy while the dependency is slow.

---

# 64. Error Propagation

Example:

```text
Payment API timeout
       |
       v
Payment service error
       |
       v
Order service error
       |
       v
HTTP 500
```

The trace shows the original failure and propagated impact.

---

# 65. Retry Storm

Example:

```text
Order
 |
 +--> Payment attempt 1
 +--> Payment attempt 2
 +--> Payment attempt 3
 +--> Payment attempt 4
```

Tracing reveals repeated downstream calls.

This can identify retry amplification.

---

# 66. Timeout Budget

Suppose:

```text
API timeout = 5 seconds
```

Trace:

```text
Order      4.9 sec
Payment    4.8 sec
Database   4.7 sec
```

This indicates the downstream dependency is consuming almost the entire request budget.

---

# 67. Trace-to-Log Correlation

Logs should include:

```text
trace_id
span_id
```

Example:

```json
{
  "level": "ERROR",
  "service": "payment-service",
  "trace_id": "abc123",
  "span_id": "def456",
  "message": "Payment provider timeout"
}
```

Then:

```text
Jaeger trace
     |
     v
trace_id
     |
     v
Kibana
     |
     v
Detailed log
```

---

# 68. Trace-to-Metrics Correlation

Example:

```text
Prometheus:
payment latency P99 = 4.2s
```

Then:

```text
Jaeger:
payment-service traces
```

Then:

```text
Kibana:
payment provider timeout
```

Together:

```text
Metric -> Detect
Trace  -> Locate
Log    -> Explain
```

---

# 69. Full Observability Workflow

```text
Alert
 |
 v
Grafana
 |
 v
Identify service
 |
 v
Jaeger
 |
 v
Find slow/error span
 |
 v
Kibana
 |
 v
Search trace_id
 |
 v
Find detailed error
 |
 v
Mitigate
```

---

# 70. Application Deployment

A production application should receive tracing configuration through environment variables or supported runtime configuration.

Concept:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: payment-service

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: http://otel-collector:4317
```

The exact endpoint and protocol depend on the collector configuration.

---

# 71. Kubernetes Service

The collector can be exposed internally:

```text
otel-collector.logging.svc.cluster.local
```

Then applications send telemetry to the internal Kubernetes Service.

---

# 72. OTLP Ports

Common OTLP ports:

```text
4317 -> OTLP gRPC
4318 -> OTLP HTTP
```

Use the protocol configured by the collector.

---

# 73. Kubernetes NetworkPolicy

Restrict which namespaces can send telemetry to the collector.

Concept:

```text
production namespaces
       |
       +--> OTel Collector

untrusted namespace
       X
```

This reduces unnecessary access.

---

# 74. Resource Requests and Limits

Collectors require explicit resources.

Example conceptual configuration:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 1
    memory: 1Gi
```

Actual sizing should be based on telemetry volume.

---

# 75. Collector Horizontal Scaling

```text
Applications
    |
    +--> Collector 1
    |
    +--> Collector 2
    |
    +--> Collector 3
```

Scale based on:

```text
Spans/sec
CPU
Memory
Queue
Export latency
```

---

# 76. Collector High Availability

Avoid:

```text
One collector pod
```

for production-critical tracing.

Prefer:

```text
Collector 1
Collector 2
Collector 3
```

with suitable Kubernetes scheduling and disruption controls.

---

# 77. Pod Anti-Affinity

Collectors should ideally not all run on the same node.

Concept:

```text
Node 1 -> Collector 1
Node 2 -> Collector 2
Node 3 -> Collector 3
```

This reduces a single-node failure impact.

---

# 78. PodDisruptionBudget

For critical collector tiers, use a PDB so planned node operations do not remove all replicas simultaneously.

Example concept:

```text
minimum available collectors >= 2
```

Exact settings depend on replica count.

---

# 79. Collector Queueing

If the Jaeger backend slows:

```text
Collector
 |
 v
Queue
 |
 v
Retry
 |
 v
Jaeger
```

Queueing can protect against short-lived downstream failures.

Do not allow an unbounded queue.

---

# 80. Retry Strategy

A collector can retry transient failures.

Concept:

```text
Export
 |
 X
Backend unavailable
 |
 v
Retry with backoff
 |
 v
Success
```

Retry must be combined with bounded queueing and monitoring.

---

# 81. Circuit Protection

When downstream systems are failing:

```text
Backend failure
      |
      v
Collector pressure
```

Use memory limits, queues and retry policies to prevent the collector from becoming unstable.

---

# 82. Jaeger Backend Scaling

Scale according to:

```text
Spans/sec
Retention
Search rate
Storage
Query concurrency
```

Do not scale only the UI.

The backend storage and ingestion path are usually more important.

---

# 83. Trace Retention

Trace retention should consider:

```text
Incident investigation
Storage cost
Traffic volume
Compliance
Business requirements
```

A practical policy may keep:

```text
High-value traces longer
Normal sampled traces shorter
```

---

# 84. Why Traces Can Be Expensive

Every trace can contain many spans.

Example:

```text
1 request
 |
 +--> 20 spans
```

At:

```text
10,000 requests/sec
```

that can become:

```text
200,000 spans/sec
```

Sampling is therefore important.

---

# 85. Span Volume Calculation

Approximate:

```text
requests/sec
x
average spans/request
=
spans/sec
```

Example:

```text
5,000 requests/sec
x
12 spans/request
=
60,000 spans/sec
```

Then estimate:

```text
spans/sec
x
average span size
x
retention
```

for capacity planning.

---

# 86. Trace Sampling Capacity

If:

```text
100,000 spans/sec
```

and normal traffic is sampled at:

```text
10%
```

the retained volume can be significantly lower, but tail sampling must still process enough information to make its decision.

---

# 87. Error Trace Priority

A production policy can prioritize:

```text
HTTP 5xx
Exceptions
High latency
Specific business operations
```

and sample routine successful traffic more aggressively.

---

# 88. Health Check Filtering

Health endpoints can generate large trace volumes:

```text
GET /health
GET /ready
GET /health
```

Consider filtering or aggressively sampling them.

Do not remove meaningful availability signals from the metrics system.

---

# 89. Security — Trace Data

Trace attributes may accidentally contain:

```text
Authorization headers
Cookies
Email addresses
User IDs
Request bodies
Payment information
```

Treat trace data as sensitive operational data.

---

# 90. Redaction

Prefer preventing sensitive data at instrumentation.

Also consider collector-level filtering/redaction as defense in depth.

Example:

```text
Authorization
      |
      v
remove/redact
      |
      v
export
```

---

# 91. Authentication

Secure:

```text
Application -> Collector
Collector -> Backend
User -> Jaeger UI
```

Use the supported authentication and TLS mechanisms of the chosen deployment.

---

# 92. Network Security

Keep:

```text
Collector
Jaeger
Storage
```

inside controlled network boundaries.

Expose the UI only through an approved access layer.

---

# 93. TLS

Production architecture:

```text
Application
   |
  TLS
   |
Collector
   |
  TLS
   |
Jaeger/backend
```

Exact TLS requirements depend on cluster trust boundaries and deployment.

---

# 94. Secrets

Store credentials and certificates using:

```text
Kubernetes Secrets
AWS Secrets Manager
External secret-management solutions
```

Do not commit private keys to Git.

---

# 95. Trace Data Governance

Define:

```text
Who can search traces?
How long are traces retained?
What data may be recorded?
Which teams can access production traces?
```

Tracing is not exempt from data governance.

---

# 96. Troubleshooting — No Traces

Trace pipeline:

```text
Application
 |
 v
Instrumentation
 |
 v
OTLP
 |
 v
Collector
 |
 v
Jaeger
 |
 v
Storage
 |
 v
UI
```

Find the first broken layer.

---

# 97. Check Application Instrumentation

Verify:

```text
Instrumentation loaded
Service name configured
Exporter configured
Endpoint correct
Protocol correct
```

Application logs may show exporter errors.

---

# 98. Check Collector

```bash
kubectl get pods -n logging
kubectl logs <otel-collector-pod> -n logging
```

Look for:

```text
connection refused
authentication failure
export failure
queue full
memory pressure
```

---

# 99. Check Collector Service

```bash
kubectl get svc -n logging
kubectl describe svc <otel-service> -n logging
```

Verify:

```text
Port
TargetPort
Endpoints
Selector
```

---

# 100. Check Network Connectivity

From an application pod, verify connectivity to the collector service using approved diagnostic tools.

Check:

```text
DNS
Port
NetworkPolicy
Service endpoints
```

---

# 101. Troubleshooting — Collector Receives but Cannot Export

Possible causes:

```text
Jaeger unavailable
Wrong endpoint
Wrong protocol
TLS mismatch
Authentication failure
NetworkPolicy
Backend overloaded
```

Check:

```text
Collector exporter metrics/logs
Backend health
Network connectivity
```

---

# 102. Troubleshooting — Traces Delayed

Possible causes:

```text
Batch timeout
Collector queue
Backend indexing
Storage pressure
High query/ingest load
Network latency
```

Compare:

```text
Application span timestamp
Collector processing
Backend ingestion
UI arrival
```

---

# 103. Troubleshooting — Partial Trace

Example:

```text
Order span
Payment span
```

but no:

```text
Database span
```

Possible causes:

```text
Database instrumentation missing
Context not propagated
DB operation not supported by instrumentation
Sampling
Exporter failure
```

---

# 104. Troubleshooting — Broken Trace Context

Symptoms:

```text
Multiple unrelated traces
```

Check:

```text
HTTP propagation
Framework middleware
Instrumentation
Headers
Service compatibility
```

---

# 105. Troubleshooting — Trace IDs Change Unexpectedly

Possible causes:

```text
Context lost
New root span created
Instrumentation misconfiguration
Manual propagation bug
Async context problem
```

The first step is to identify where the Trace ID changes.

---

# 106. Troubleshooting — Missing Service

If Jaeger does not show:

```text
inventory-service
```

check:

```text
Instrumentation
service.name
Exporter
Collector
Sampling
Traffic
```

---

# 107. Troubleshooting — High Collector CPU

Check:

```text
Spans/sec
Processors
Tail sampling
Attribute processing
Batching
Export retries
```

Tail sampling can require substantial resources at high traffic.

---

# 108. Troubleshooting — Collector Memory Growth

Check:

```text
Queue size
Tail sampling traces in memory
Batch size
Backend outage
Exporter retries
Large spans
```

Memory limiter protection is important.

---

# 109. Troubleshooting — Jaeger Storage High

Check:

```text
Trace volume
Retention
Sampling
Span size
Replication
Query workload
```

First control unnecessary telemetry before blindly adding storage.

---

# 110. Troubleshooting — UI Shows No Results

Check:

```text
Correct service
Correct time range
Correct backend
Storage health
Sampling
Index/data source
```

A common issue is simply searching the wrong time window.

---

# 111. Production Incident — API Latency Spike

Alert:

```text
P99 latency = 4.5 sec
```

Workflow:

```text
Grafana
 |
 v
Identify service
 |
 v
Jaeger
 |
 v
Find slow span
 |
 v
Payment dependency
 |
 v
Kibana
 |
 v
Find timeout logs
```

This is the full observability workflow.

---

# 112. Production Incident — Payment Failure

Trace:

```text
Order
 |
 +--> Payment
        |
        +--> Provider
```

If provider span is:

```text
4.2 sec
ERROR
```

the trace identifies the dependency.

Then logs can provide:

```text
Provider timeout
HTTP status
request_id
trace_id
```

---

# 113. Production Incident — Database Latency

Trace:

```text
Order
 |
 +--> Database
       |
       +--> 3.5 sec
```

Then check:

```text
Database metrics
Database logs
Query details
Connection pool
```

Tracing narrows the investigation.

---

# 114. Production Incident — Retry Storm

Trace waterfall:

```text
Payment
 |
 +--> Attempt 1
 +--> Attempt 2
 +--> Attempt 3
 +--> Attempt 4
```

Metrics:

```text
Request rate ↑
Dependency calls ↑
Latency ↑
```

Root cause may be:

```text
Aggressive retry policy
```

---

# 115. Production Incident — Deployment Regression

Timeline:

```text
10:00 version 10 healthy
10:15 version 11 deployed
10:17 P99 ↑
10:18 errors ↑
```

Trace comparison:

```text
version 10 -> payment 100 ms
version 11 -> payment 1.8 sec
```

This is strong evidence for deployment correlation.

---

# 116. Production Incident — One Service Slow

Suppose:

```text
Order P99 = 2 sec
User P99 = 100 ms
Payment P99 = 1.8 sec
```

Tracing shows:

```text
Order
 |
 +--> Payment
       |
       +--> External API
```

The bottleneck is isolated quickly.

---

# 117. Production Incident — Missing Traces During Outage

If the collector is down:

```text
Application
    |
    X
Collector
```

Tracing data may be lost depending on application/exporter buffering.

For critical environments:

```text
HA collectors
bounded queues
retry
multiple replicas
```

should be considered.

---

# 118. Observability Failure Is Also an Incident

Do not assume:

```text
Application is healthy
```

when:

```text
Monitoring is blind
```

An observability platform should have its own monitoring.

---

# 119. Monitor the Collector

Track:

```text
Received spans
Exported spans
Dropped spans
Queue size
CPU
Memory
Export failures
```

---

# 120. Monitor Jaeger

Track:

```text
Ingestion
Storage
Query latency
Errors
CPU
Memory
Disk
```

---

# 121. Monitor Trace Completeness

Potential signal:

```text
Expected service spans
vs
Observed service spans
```

A sudden drop may indicate:

```text
Instrumentation failure
Collector failure
Sampling change
```

---

# 122. Tracing SLO

Example:

> 99% of sampled error traces should become searchable within 60 seconds.

This makes tracing reliability measurable.

---

# 123. Trace Loss

Trace loss can happen because of:

```text
Exporter failure
Collector overload
Queue overflow
Sampling
Backend rejection
Network failure
```

Not every missing trace is necessarily a platform failure because sampling is intentional.

---

# 124. Sampling vs Data Loss

Important distinction:

```text
Sampling:
Intentional selection

Data loss:
Unexpected missing telemetry
```

Interviewers often test this difference.

---

# 125. Production Collector Architecture

Recommended:

```text
                 EKS Applications
                  /      |      \
                 /       |       \
                v        v        v
            OTel SDK  OTel SDK  OTel SDK
                 \       |       /
                  \      |      /
                   v     v     v
               Local Collectors
                     |
                     v
               Gateway Collectors
                /       |       \
               /        |        \
        Sampling     Routing    Enrichment
               \        |        /
                \       |       /
                     v
                   Jaeger
                     |
                     v
                   Storage
                     |
                     v
                  Jaeger UI
```

---

# 126. Why Gateway Collectors?

Gateway collectors centralize:

```text
Sampling
Routing
Security
Enrichment
Export policy
```

They also provide an abstraction between applications and backend vendors.

---

# 127. Collector Scaling Strategy

Scale local collectors based on:

```text
Node count
Application telemetry
CPU
Memory
```

Scale gateway collectors based on:

```text
Spans/sec
Tail sampling workload
Export rate
Queue size
```

---

# 128. Gateway Load Balancing

Applications should not depend on one gateway pod.

Use:

```text
Kubernetes Service
 |
 +--> Gateway 1
 +--> Gateway 2
 +--> Gateway 3
```

---

# 129. Availability Zones

For production EKS:

```text
AZ-A -> Collector
AZ-B -> Collector
AZ-C -> Collector
```

Use Kubernetes scheduling rules to spread critical replicas.

---

# 130. Disaster Recovery

DR should cover:

```text
Collector configuration
Jaeger configuration
Storage
Dashboards/configuration
Secrets/certificates
```

Application instrumentation should remain deployable even if the tracing backend changes.

---

# 131. Trace Storage Backup

If the selected Jaeger backend supports snapshots/backups, configure:

```text
Frequency
Retention
Encryption
Restore testing
```

The exact backup method depends on the storage backend.

---

# 132. Recovery Scenario

If Jaeger UI fails but storage remains healthy:

```text
Restore Query/UI
```

If storage fails:

```text
Restore storage
 |
 v
Restore Jaeger
 |
 v
Verify traces
```

---

# 133. GitOps Structure

Recommended repository:

```text
otel-jaeger/
│
├── collector/
│   ├── values.yaml
│   ├── daemonset.yaml
│   └── gateway.yaml
│
├── jaeger/
│   ├── values.yaml
│   └── config.yaml
│
├── instrumentation/
│   ├── java/
│   ├── nodejs/
│   └── python/
│
├── dashboards/
│
├── alerts/
│
├── runbooks/
│   ├── collector-down.md
│   ├── missing-traces.md
│   └── high-latency.md
│
└── README.md
```

---

# 134. GitOps Flow

```text
Developer
   |
   v
Git
   |
   v
CI validation
   |
   v
ArgoCD
   |
   v
EKS
   |
   v
OTel Collector
   |
   v
Jaeger
```

Benefits:

```text
Version control
Review
Audit
Rollback
Repeatability
```

---

# 135. CI Validation

Validate:

```text
YAML
Helm
Collector configuration
Kubernetes manifests
Security policies
```

Then:

```text
Deploy test
Generate traces
Verify Jaeger
```

---

# 136. Test Application

Create a small test workflow:

```text
Frontend
 |
 v
Order
 |
 +--> User
 |
 +--> Payment
       |
       +--> Database
```

Generate requests and confirm one trace spans the complete path.

---

# 137. Trace Validation Test

Expected:

```text
One Trace ID
 |
 +--> frontend
 +--> order
 +--> user
 +--> payment
 +--> database
```

If the Trace ID changes:

```text
Investigate context propagation.
```

---

# 138. Failure Injection Test

In non-production:

```text
Delay Payment Service
```

Expected:

```text
Payment span duration ↑
Order trace duration ↑
```

Then:

```text
Grafana
 |
 v
Jaeger
```

should expose the problem.

---

# 139. Error Injection Test

Return:

```text
HTTP 500
```

Expected:

```text
Error span
Trace status ERROR
Application log
```

Then correlate:

```text
trace_id
```

between Jaeger and Kibana.

---

# 140. Collector Failure Test

Temporarily stop one collector replica in a test environment.

Verify:

```text
Other collector replicas remain available
```

and:

```text
Application telemetry recovers
```

---

# 141. Backend Failure Test

Temporarily make the Jaeger backend unavailable.

Verify:

```text
Collector queue
Retry
Memory protection
Recovery
```

Measure:

```text
Lost traces
Recovery time
```

---

# 142. Load Test

Generate:

```text
1,000 requests/sec
```

Then measure:

```text
Spans/sec
Collector CPU
Collector memory
Jaeger ingestion
Storage growth
```

Repeat with higher rates until the capacity boundary is understood.

---

# 143. Capacity Planning

Estimate:

```text
Requests/sec
x
Spans/request
=
Spans/sec
```

Then:

```text
Spans/sec
x
Average span size
=
Bytes/sec
```

Then:

```text
Bytes/sec
x
Retention
=
Storage requirement
```

Add:

```text
Replication
Index/storage overhead
Operational headroom
```

---

# 144. Trace Retention Strategy

Example conceptual strategy:

```text
Errors:
Longer retention

Slow traces:
Longer retention

Normal traces:
Aggressive sampling / shorter retention
```

This provides more value per byte stored.

---

# 145. Cost Optimization

Main drivers:

```text
Span volume
Span size
Retention
Sampling
Storage
Query workload
Network
```

Optimization:

```text
1. Reduce unnecessary spans
2. Sample normal traffic
3. Keep error traces
4. Reduce oversized attributes
5. Tune retention
6. Right-size collectors
7. Right-size storage
```

---

# 146. Avoid Instrumentation Explosion

Do not automatically create spans for every internal function.

Prefer meaningful operations:

```text
createOrder
processPayment
reserveInventory
```

rather than:

```text
functionA
functionB
functionC
functionD
```

unless those spans provide diagnostic value.

---

# 147. Trace Attribute Standards

Prefer semantic conventions for:

```text
HTTP
Database
Messaging
RPC
Cloud
Kubernetes
```

This makes traces more consistent across languages.

---

# 148. Versioning

Include:

```text
service.name
service.version
deployment.environment
```

Then compare:

```text
Version A
Version B
```

during deployment incidents.

---

# 149. Blue/Green Deployment

Trace:

```text
version A
version B
```

Compare:

```text
Latency
Errors
Dependency behavior
```

before increasing traffic.

---

# 150. Canary Deployment

For canary:

```text
95% -> version A
5%  -> version B
```

Use traces to compare:

```text
P95
P99
Error rate
Dependency latency
```

for the two versions.

---

# 151. Tracing and SLOs

Example SLO:

```text
99.9% of checkout requests < 1 second
```

Metrics detect SLO violations.

Tracing explains:

```text
Why checkout exceeded 1 second.
```

---

# 152. Error Budget Investigation

If error budget is burning quickly:

```text
Metrics
 |
 v
High latency / errors
 |
 v
Tracing
 |
 v
Slow dependency
 |
 v
Logs
 |
 v
Root cause
```

This connects tracing to SRE practices.

---

# 153. Production Incident Runbook — High Latency

```text
1. Check Grafana latency.
2. Identify affected service.
3. Open Jaeger.
4. Search slow traces.
5. Identify longest span.
6. Determine dependency.
7. Search trace_id in Kibana.
8. Check deployment/version.
9. Check dependency metrics.
10. Mitigate.
11. Document root cause.
```

---

# 154. Production Incident Runbook — Error Spike

```text
1. Check error-rate metric.
2. Identify service.
3. Search error traces.
4. Identify failed span.
5. Inspect parent/child spans.
6. Search trace_id in logs.
7. Check recent deployment.
8. Check dependency.
9. Roll back or mitigate.
10. Verify recovery.
```

---

# 155. Production Incident Runbook — Missing Traces

```text
1. Confirm expected traffic.
2. Check application instrumentation.
3. Check exporter.
4. Check collector service.
5. Check collector logs.
6. Check collector metrics.
7. Check backend.
8. Check sampling.
9. Check UI time range.
10. Verify trace recovery.
```

---

# 156. Production Incident Runbook — Collector Memory

```text
1. Check memory usage.
2. Check queue.
3. Check backend availability.
4. Check span volume.
5. Check tail sampling.
6. Check oversized spans.
7. Check memory limiter.
8. Scale collectors.
9. Restore downstream capacity.
```

---

# 157. Production Incident Runbook — Storage Growth

```text
1. Measure spans/sec.
2. Identify largest services.
3. Check sampling.
4. Check span size.
5. Check retention.
6. Check duplicate telemetry.
7. Reduce unnecessary data.
8. Scale storage if required.
```

---

# 158. Observability Architecture Integration

Combined platform:

```text
                         EKS
                          |
             +------------+------------+
             |            |            |
             v            v            v
         Prometheus       ELK         OTel
             |            |            |
          Metrics        Logs        Traces
             |            |            |
             v            v            v
          Grafana       Kibana      Jaeger
             \            |           /
              \           |          /
               +----------+---------+
                          |
                     Engineers
```

---

# 159. The Three-Pillar Workflow

## Metrics

```text
Detect
```

## Traces

```text
Locate
```

## Logs

```text
Explain
```

Example:

```text
Metric:
P99 latency ↑

Trace:
Payment span = 3.8 sec

Log:
Payment provider timeout
```

This is a strong production observability pattern.

---

# 160. Interview — Explain the Project

### 30-second answer

> "I implemented distributed tracing for an EKS microservices platform using OpenTelemetry and Jaeger. Applications were instrumented with OpenTelemetry and exported traces using OTLP to OpenTelemetry Collectors. The collectors handled batching, enrichment, retries and sampling before forwarding traces to Jaeger. Jaeger provided trace search and visualization. I also designed trace-to-log correlation using trace IDs and used metrics from Prometheus/Grafana to identify incidents before drilling into Jaeger."

---

# 161. Interview — 60-second answer

> "For an EKS-based microservices environment, I used OpenTelemetry as the instrumentation and telemetry layer and Jaeger as the tracing backend. Services such as order, payment, inventory and user were instrumented using language-specific OpenTelemetry libraries or agents. Trace context was propagated across HTTP calls so a single request could be visualized across services. Telemetry was exported through OTLP to an OpenTelemetry Collector, where I could apply batching, memory protection, enrichment, retries and sampling before exporting to Jaeger. For production, I considered highly available collectors, tail sampling for error and slow traces, TLS, RBAC, retention, storage capacity and failure handling. Trace IDs were also propagated into application logs so engineers could move from Grafana to Jaeger and then Kibana during incidents."

---

# 162. Interview — Why OpenTelemetry?

### Answer

> "OpenTelemetry provides a vendor-neutral standard for instrumentation and telemetry collection. It reduces application coupling to a specific observability vendor or backend and gives us a consistent approach across Java, Node.js and Python services."

---

# 163. Interview — Why Jaeger?

### Answer

> "Jaeger provides a dedicated distributed tracing experience for searching and visualizing traces and spans. It is useful for understanding request flow and latency across microservices."

---

# 164. Interview — Why Use an OTel Collector?

### Answer

> "The collector provides a centralized telemetry processing layer. It can batch, filter, enrich, retry, sample and route telemetry without putting all that responsibility into application code."

---

# 165. Interview — Why Not Send Directly to Jaeger?

### Answer

> "Direct export tightly couples applications to the tracing backend. Using OpenTelemetry and a Collector gives us a buffer and processing layer and makes it easier to change or add backends later."

---

# 166. Interview — What Is Trace Context?

### Answer

> "Trace context carries the identity of a distributed request across service boundaries. With W3C Trace Context, services propagate identifiers such as the Trace ID and parent context so downstream spans become part of the same trace."

---

# 167. Interview — What Is a Span?

### Answer

> "A span represents one operation within a trace. It contains timing, identity, parent-child relationships, attributes and status. Multiple spans form the complete distributed trace."

---

# 168. Interview — Head vs Tail Sampling

### Answer

> "Head sampling makes the sampling decision early, which is efficient but can discard a trace before we know whether it becomes important. Tail sampling makes the decision after collecting enough of the trace to evaluate conditions such as errors or latency, which is more useful for retaining problematic traces."

---

# 169. Interview — How Would You Sample in Production?

Answer:

```text
100% errors
100% very slow traces
5-10% normal traffic
Low/filtered health checks
```

Then adjust according to:

```text
Traffic
Storage
Incident requirements
Cost
```

---

# 170. Interview — How Do You Find a Slow Service?

Answer:

```text
Grafana
 |
 v
Latency alert
 |
 v
Jaeger
 |
 v
Slow traces
 |
 v
Longest span
 |
 v
Dependency
```

Then use logs and metrics for deeper investigation.

---

# 171. Interview — How Do You Correlate Logs and Traces?

Answer:

> "I propagate the Trace ID into application logging context. The logs then contain the trace ID, allowing engineers to open a trace in Jaeger and search the same identifier in Kibana."

---

# 172. Interview — What If Only One Service Is Missing From a Trace?

Answer:

> "I would check whether that service is instrumented, whether it receives and propagates context, whether its exporter is configured correctly, whether the collector receives its spans, and whether sampling or filtering is dropping them."

---

# 173. Interview — What If Trace IDs Change Between Services?

Answer:

> "That usually indicates a context propagation problem or a new root span being created. I would inspect HTTP middleware, instrumentation, asynchronous context handling and the request headers between the affected services."

---

# 174. Interview — How Do You Scale the Collector?

Answer:

> "I monitor spans per second, CPU, memory, queue size and export latency. I then run multiple replicas behind a Kubernetes Service and spread them across nodes or availability zones. For high-volume environments, I separate local collection from gateway processing."

---

# 175. Interview — Why Use DaemonSet Collectors?

Answer:

> "A DaemonSet provides a collector on each eligible node, which is useful for node-local telemetry collection and scales automatically with the Kubernetes node count."

---

# 176. Interview — Why Use Gateway Collectors?

Answer:

> "Gateway collectors centralize processing such as tail sampling, routing, enrichment and export policies. They can also be scaled independently from node-level collectors."

---

# 177. Interview — How Do You Protect Collectors From OOM?

Answer:

```text
Memory limiter
Bounded queues
Batching
Controlled retries
Resource limits
Horizontal scaling
```

Then investigate the source of telemetry growth.

---

# 178. Interview — What Causes Collector Memory Growth?

Answer:

```text
Backend outage
Queue growth
Tail sampling
Large spans
High traffic
Large batches
Exporter retries
```

---

# 179. Interview — What Causes High Trace Storage?

Answer:

```text
High traffic
Many spans/request
Large attributes
Long retention
Low sampling
Duplicate instrumentation
```

---

# 180. Interview — How Do You Reduce Trace Cost?

Answer:

> "I first reduce unnecessary span volume and oversized attributes. Then I apply intelligent sampling, especially retaining error and slow traces, and tune retention and storage. I also monitor which services generate the most telemetry."

---

# 181. Interview — What Is a Good Trace Attribute?

Good:

```text
service.name
service.version
HTTP method
HTTP route
status
dependency name
```

Bad:

```text
password
token
full request body
large response
```

---

# 182. Interview — How Do Traces Help Kubernetes?

Example:

```text
Pod looks healthy
CPU normal
Memory normal
```

but:

```text
Request latency high
```

Trace can reveal:

```text
External dependency slow
```

This demonstrates why infrastructure health does not guarantee application health.

---

# 183. Interview — What If Jaeger Is Down?

Answer:

> "I would first determine whether collectors are buffering and retrying telemetry. I would restore the backend or route telemetry to an available destination where supported, monitor queue growth and verify recovery. I would also check whether the failure caused unacceptable trace loss."

---

# 184. Interview — What If the Collector Is Down?

Answer:

> "With multiple collector replicas and proper service routing, applications should continue exporting to healthy collectors. I would verify service endpoints, collector availability, exporter retries and queue behavior."

---

# 185. Interview — What Is Trace Loss?

Trace loss means:

```text
Telemetry was expected
but was not retained
```

Possible causes:

```text
Collector overload
Queue overflow
Backend rejection
Exporter failure
Network failure
Sampling
```

Sampling is intentional; unexpected drops are operational failures.

---

# 186. Interview — How Do You Monitor the Observability Platform?

Monitor:

```text
Collector
Jaeger
Storage
```

with:

```text
Prometheus
Grafana
```

This creates:

```text
Application observability
+
Observability-platform observability
```

---

# 187. Interview — How Would You Troubleshoot a 5-Second API?

Answer:

```text
1. Check P95/P99 in Grafana.
2. Open slow traces in Jaeger.
3. Identify longest span.
4. Check child dependency.
5. Search trace_id in Kibana.
6. Check recent deployment.
7. Check dependency metrics.
8. Mitigate.
```

---

# 188. Interview — How Would You Troubleshoot Missing Traces?

Answer:

```text
Application instrumentation
       |
Exporter
       |
Collector service
       |
Collector receiver
       |
Collector processors
       |
Collector exporter
       |
Jaeger
       |
Storage
       |
UI
```

Find the first failing layer.

---

# 189. Interview — How Do You Handle PII?

Answer:

> "I avoid collecting sensitive data at instrumentation time. Where necessary, I apply collector-level filtering or redaction as defense in depth, restrict access through RBAC and retain trace data only for the required period."

---

# 190. Interview — How Do You Handle a Deployment Regression?

Answer:

> "I compare traces by service version and look for changes in latency, error spans or dependency behavior. If version B has a significantly slower critical path than version A, I correlate it with the deployment and roll back or mitigate according to the incident plan."

---

# 191. Interview — What Is the Critical Path?

Answer:

> "The critical path is the sequence of operations that directly determines the total request completion time. Parallel work does not necessarily add its durations together, so traces are useful for seeing which dependency actually controls latency."

---

# 192. Interview — How Do You Identify an N+1 Problem?

Answer:

> "I inspect repeated database or downstream spans inside a single request. If one request creates dozens or hundreds of similar database calls, the trace waterfall exposes that pattern."

---

# 193. Interview — How Do You Detect Retry Amplification?

Answer:

> "I look for repeated downstream spans within a single trace and compare dependency request rate with incoming request rate. If one incoming request causes several downstream attempts, retries may be amplifying the load."

---

# 194. Interview — What Would You Improve?

Possible improvements:

```text
Tail sampling
Gateway collectors
Kafka buffering
Better trace-to-log correlation
Automated SLO dashboards
Cross-region tracing
Advanced dependency mapping
Automated incident enrichment
```

Choose improvements based on scale and requirements.

---

# 195. Production Readiness Checklist

## Instrumentation

- [ ] OpenTelemetry configured
- [ ] Stable service names
- [ ] Version metadata
- [ ] Environment metadata
- [ ] Context propagation
- [ ] HTTP instrumentation
- [ ] Database instrumentation
- [ ] Messaging instrumentation where required

## Collector

- [ ] HA replicas
- [ ] Resource requests/limits
- [ ] Memory limiter
- [ ] Batch processor
- [ ] Retry
- [ ] Queue where required
- [ ] Sampling
- [ ] Monitoring

## Jaeger

- [ ] Supported deployment
- [ ] Persistent/appropriate storage
- [ ] Retention
- [ ] Authentication
- [ ] TLS
- [ ] Backup/DR
- [ ] Capacity planning

## Security

- [ ] No secrets in spans
- [ ] RBAC
- [ ] Network controls
- [ ] TLS
- [ ] Secret management
- [ ] Data retention policy

## Operations

- [ ] Collector alerts
- [ ] Backend alerts
- [ ] Trace SLO
- [ ] Runbooks
- [ ] Failure testing
- [ ] Load testing
- [ ] DR testing

---

# 196. End-to-End Troubleshooting Model

When a trace is missing:

```text
Application
     |
     v
Instrumentation
     |
     v
Exporter
     |
     v
OTLP
     |
     v
Collector Receiver
     |
     v
Processors
     |
     v
Collector Exporter
     |
     v
Jaeger
     |
     v
Storage
     |
     v
Jaeger UI
```

At each stage ask:

```text
Is the telemetry present?
Is it delayed?
Is it dropped?
Is it rejected?
Is it sampled?
Is it transformed?
```

---

# 197. Production Tracing Mental Model

Think in five layers:

```text
Instrumentation
      |
      v
Propagation
      |
      v
Collection
      |
      v
Processing
      |
      v
Storage / Visualization
```

And four operational dimensions:

```text
Reliability
Security
Performance
Cost
```

---

# 198. Final Project Architecture

```text
                         Users
                           |
                           v
                     AWS ALB / Ingress
                           |
                           v
                 +----------------------+
                 | EKS Microservices    |
                 |                      |
                 | Order                |
                 | User                 |
                 | Payment              |
                 | Inventory            |
                 +----------+-----------+
                            |
                      OTel SDK/Agent
                            |
                            v
                  Local OTel Collectors
                            |
                            v
                  Gateway OTel Collectors
                    /       |       \
                   /        |        \
            Batch       Sampling   Enrichment
                   \        |        /
                    \       |       /
                         Jaeger
                            |
                            v
                         Storage
                            |
                            v
                       Jaeger UI
                            |
             +--------------+--------------+
             |                             |
             v                             v
          Engineers                    Incident Team
```

---

# 199. Complete Observability Workflow

```text
User reports:
"Checkout is slow"
          |
          v
Prometheus/Grafana
          |
          v
P99 checkout latency increased
          |
          v
Jaeger
          |
          v
Payment span = 3.8 sec
          |
          v
Kibana
          |
          v
trace_id search
          |
          v
Payment provider timeout
          |
          v
Check dependency metrics
          |
          v
Mitigate / rollback / fail over
          |
          v
Verify SLO recovery
```

This is the practical workflow expected from a production DevOps/DevSecOps engineer.

---

# 200. Project Summary

The project establishes:

```text
OpenTelemetry
      |
      +--> Standard instrumentation
      +--> Context propagation
      +--> OTLP
      |
      v
OpenTelemetry Collector
      |
      +--> Batching
      +--> Sampling
      +--> Filtering
      +--> Enrichment
      +--> Retry
      +--> Queuing
      |
      v
Jaeger
      |
      +--> Trace search
      +--> Trace visualization
      +--> Dependency analysis
      |
      v
Engineers
```

The most important production principle is:

> Use OpenTelemetry as the standard instrumentation and collection layer, use the Collector as the control point for processing and reliability, and use Jaeger as the tracing analysis backend. Combine traces with metrics and logs so incidents can be detected, localized and explained quickly.

---

# 201. What This Project Adds to the Observability Stack

Previous project:

```text
01-Prometheus-Grafana-EKS
```

gave:

```text
Metrics
Dashboards
Alerts
SLO signals
```

Second project:

```text
02-ELK-Centralized-Logging
```

gave:

```text
Logs
Search
Historical context
Incident investigation
```

This project:

```text
03-OpenTelemetry-Jaeger-Tracing
```

adds:

```text
Distributed request flow
Latency breakdown
Dependency analysis
Trace correlation
Critical-path analysis
```

Together:

```text
                Production EKS
                      |
        +-------------+-------------+
        |             |             |
        v             v             v
    Prometheus       ELK          OTel
        |             |             |
     Metrics         Logs        Traces
        |             |             |
        v             v             v
     Grafana        Kibana       Jaeger
        \             |             /
         \            |            /
          +-----------+-----------+
                      |
                      v
             Full Observability
```

---

# 202. Key Interview Takeaway

A strong production explanation is:

> "Metrics tell me that a problem exists, traces show me where the request is spending time or failing across service boundaries, and logs provide the detailed event context. In my EKS observability architecture, I use Prometheus and Grafana for metrics, ELK for centralized logs, and OpenTelemetry with Jaeger for distributed tracing. I correlate them using service, version, environment and trace IDs, which gives the incident team an end-to-end path from alert to root cause."

---

# 203. Next Project

The next file is:

```text
04-Full-Stack-Observability.md
```

It will combine the three completed layers:

```text
Prometheus + Grafana
        +
ELK
        +
OpenTelemetry + Jaeger
```

into one production-grade EKS observability architecture covering:

```text
Metrics
Logs
Traces
Correlation
Dashboards
Alerting
SLI/SLO
Incident response
Security
HA
Scaling
DR
Cost optimization
GitOps
Production troubleshooting
End-to-end incident scenarios
Interview architecture questions
```
