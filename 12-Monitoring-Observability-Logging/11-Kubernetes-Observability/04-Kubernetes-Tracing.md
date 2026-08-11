# Kubernetes Tracing

## 1. Overview

Kubernetes tracing provides visibility into how a request travels through distributed services running inside a Kubernetes cluster.

Metrics can tell us:

```text
Latency increased
Error rate increased
CPU increased
```

Logs can tell us:

```text
Database connection timeout
Application exception
Authentication failure
```

Tracing can show:

```text
Which service handled the request?
Where did the request spend time?
Which downstream dependency was slow?
Which service generated the error?
```

A typical tracing architecture is:

```text
User Request
     ↓
Ingress / ALB
     ↓
Service A
     ↓
Service B
     ↓
Service C
     ↓
Database
```

Tracing represents this request as a distributed trace:

```text
Trace
├── Ingress Span
├── Service A Span
├── Service B Span
├── Service C Span
└── Database Span
```

---

# 2. Why Kubernetes Tracing Is Important

Modern Kubernetes applications are commonly composed of multiple microservices.

For example:

```text
Orders
   ↓
Payment
   ↓
Inventory
   ↓
Database
```

If the request takes 5 seconds, Kubernetes metrics alone may not identify the exact bottleneck.

Tracing can show:

```text
Orders       → 100ms
Payment      → 3.5s
Inventory    → 200ms
Database     → 1.1s
```

Now the slow dependency is much easier to identify.

---

# 3. What Is Distributed Tracing?

Distributed tracing follows one request across multiple services.

Example:

```text
User
 ↓
Orders
 ↓
Payment
 ↓
Inventory
 ↓
Database
```

A single trace represents the complete request.

Each operation creates a span:

```text
Trace
│
├── Orders
│
├──── Payment
│
├──────── Inventory
│
└──────────── Database
```

The trace shows the relationship and timing between these operations.

---

# 4. Trace

A trace represents the complete journey of a request through a distributed system.

Example:

```text
Trace ID:
abc123
```

The trace may contain:

```text
Span 1 → Orders
Span 2 → Payment
Span 3 → Inventory
Span 4 → Database
```

All spans belonging to the request are connected through the same trace context.

---

# 5. Span

A span represents a single operation.

Examples:

```text
HTTP request
Database query
Redis operation
Message processing
External API call
```

Example:

```text
Orders Service
     │
     └── Span: POST /orders
```

Another:

```text
Payment Service
     │
     └── Span: POST /payment
```

---

# 6. Trace vs Span

### Trace

Represents:

```text
Complete request journey
```

### Span

Represents:

```text
One operation within the request
```

Example:

```text
Trace
│
├── Span: Orders
├── Span: Payment
├── Span: Inventory
└── Span: Database
```

A trace contains multiple related spans.

---

# 7. Trace Context

For distributed tracing to work, tracing context must travel between services.

Conceptually:

```text
Service A
   ↓
Trace Context
   ↓
Service B
   ↓
Trace Context
   ↓
Service C
```

The context typically contains information that allows downstream services to associate their spans with the same trace.

---

# 8. Trace ID

The Trace ID identifies the complete distributed request.

Example:

```text
trace_id=abc123
```

Service A:

```text
trace_id=abc123
```

Service B:

```text
trace_id=abc123
```

Service C:

```text
trace_id=abc123
```

This allows all spans to be grouped into one trace.

---

# 9. Span ID

Each span has its own identifier.

Example:

```text
Trace ID = abc123

Orders:
span_id = span001

Payment:
span_id = span002

Database:
span_id = span003
```

The Trace ID connects the complete request.

The Span ID identifies an individual operation.

---

# 10. Parent-Child Relationship

Spans have relationships.

Example:

```text
Orders
  │
  ├── Payment
  │      │
  │      └── Database
  │
  └── Inventory
```

Here:

```text
Orders → Parent
Payment → Child
Database → Child of Payment
```

This creates a trace hierarchy.

---

# 11. Kubernetes Request Flow

A production request may look like:

```text
Internet
   ↓
ALB / Ingress
   ↓
Orders Service
   ↓
Payment Service
   ↓
Inventory Service
   ↓
Database
```

Tracing can represent:

```text
Trace
│
├── ALB / Ingress
├── Orders
├── Payment
├── Inventory
└── Database
```

The exact components that generate spans depend on instrumentation.

---

# 12. OpenTelemetry

OpenTelemetry provides a vendor-neutral framework for collecting telemetry.

For tracing:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
Trace
     ↓
OTLP
     ↓
OpenTelemetry Collector
     ↓
Jaeger
```

OpenTelemetry can provide:

```text
Instrumentation
Trace context propagation
Span creation
Export
Processing
```

---

# 13. OpenTelemetry SDK

Applications use OpenTelemetry SDKs to create traces.

Conceptually:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
Create Span
     ↓
Export Span
```

The SDK can automatically instrument supported frameworks or allow manual instrumentation.

---

# 14. Automatic Instrumentation

Automatic instrumentation can create spans without developers manually creating every span.

For example:

```text
HTTP Request
     ↓
Automatic Instrumentation
     ↓
HTTP Span
```

Database operations and supported framework calls can also be instrumented depending on the language and instrumentation package.

---

# 15. Manual Instrumentation

Manual instrumentation allows developers to create custom spans.

Example:

```text
Order Processing
     ↓
Create Span
     ↓
Validate Order
     ↓
Process Payment
     ↓
Complete Span
```

This is useful for important business operations that automatic instrumentation may not capture.

---

# 16. OpenTelemetry Collector

The Collector receives and processes telemetry.

Architecture:

```text
Applications
     ↓
OpenTelemetry SDK
     ↓
OTLP
     ↓
OpenTelemetry Collector
     ↓
Jaeger
```

The Collector can perform:

```text
Receive
Process
Filter
Sample
Batch
Retry
Export
```

---

# 17. Kubernetes Collector Architecture

A common Kubernetes design is:

```text
Application Pods
      ↓
OTel Collector Agent
      ↓
OTel Collector Gateway
      ↓
Jaeger
```

The Agent can run as a DaemonSet:

```text
Node-1 → OTel Agent
Node-2 → OTel Agent
Node-3 → OTel Agent
```

The Gateway can run as a scalable Deployment:

```text
Gateway-1
Gateway-2
Gateway-3
```

---

# 18. Why Use Collector Agents?

Agents provide a local collection layer.

Architecture:

```text
Application
    ↓
Local Collector Agent
    ↓
Collector Gateway
```

Advantages include:

```text
Local collection
Centralized processing
Reduced application dependency
Independent gateway scaling
Better failure isolation
```

---

# 19. Collector Gateway

The Gateway provides centralized processing.

Example:

```text
Agent-1 ──┐
Agent-2 ──┼──→ Gateway
Agent-3 ──┘
```

The Gateway can handle:

```text
Sampling
Batching
Filtering
Enrichment
Routing
Export
```

---

# 20. OTLP

OTLP stands for OpenTelemetry Protocol.

It is commonly used to transport OpenTelemetry telemetry.

Tracing flow:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTLP
     ↓
Collector
```

This separates instrumentation from the backend.

---

# 21. Why Use OTLP?

OTLP provides a standardized way to send OpenTelemetry telemetry.

Instead of:

```text
Application
     ↓
Jaeger-specific implementation
```

you can use:

```text
Application
     ↓
OpenTelemetry
     ↓
OTLP
     ↓
Collector
     ↓
Backend
```

This provides backend flexibility.

---

# 22. Jaeger

Jaeger is a distributed tracing platform.

It can provide:

```text
Trace storage
Trace querying
Trace visualization
Service dependency analysis
```

Architecture:

```text
Application
     ↓
OpenTelemetry
     ↓
Collector
     ↓
Jaeger
     ↓
Jaeger UI
```

---

# 23. Jaeger UI

Jaeger UI allows engineers to search and inspect traces.

Typical workflow:

```text
Service
   ↓
Trace
   ↓
Span
   ↓
Duration
   ↓
Attributes
   ↓
Error
```

This helps identify slow or failed operations.

---

# 24. Trace Visualization

A trace may appear conceptually as:

```text
Orders      |████████████████|
Payment           |████████|
Inventory              |███|
Database                  |████|
Time ─────────────────────────→
```

The length of each span indicates its duration.

This makes bottlenecks easier to identify.

---

# 25. Service Dependency Graph

Tracing can show relationships between services.

Example:

```text
              Orders
             /      \
            ↓        ↓
       Payment     Inventory
          ↓
       Database
```

This helps answer:

```text
Which service calls Payment?
Which services depend on Inventory?
Which dependency is generating failures?
```

---

# 26. Trace Duration

Suppose:

```text
Total request = 3 seconds
```

Trace breakdown:

```text
Orders       = 200ms
Payment      = 2.1s
Inventory    = 300ms
Database     = 400ms
```

The Payment service is the primary bottleneck.

Without tracing, this may be difficult to identify quickly.

---

# 27. Error Tracing

Traces can contain error information.

Example:

```text
Orders
   ↓
Payment
   ↓
Database
   X
Timeout
```

The trace can show:

```text
Payment → Error
Database → Timeout
```

This provides useful context during incident investigation.

---

# 28. Kubernetes Metadata

Tracing in Kubernetes becomes more useful when spans contain Kubernetes metadata.

Examples:

```text
cluster
namespace
pod
container
node
deployment
service
```

Example:

```text
service.name=payment
k8s.namespace.name=production
k8s.pod.name=payment-7d9f8c
```

This connects traces to Kubernetes infrastructure.

---

# 29. Service Name

Every service should have a consistent service name.

Example:

```text
service.name=orders
service.name=payment
service.name=inventory
```

Avoid inconsistent names such as:

```text
payment-service
Payment
payment-prod
payment-api
```

unless there is a deliberate naming convention.

Consistent naming makes trace searches easier.

---

# 30. Environment Metadata

Include environment information:

```text
deployment.environment=production
```

For example:

```text
service.name=payment
deployment.environment=production
```

This prevents confusion between:

```text
Production
Staging
Development
```

---

# 31. Kubernetes Namespace

A trace can include:

```text
k8s.namespace.name=production
```

This allows engineers to filter traces by namespace.

Example:

```text
namespace=production
service=payment
```

---

# 32. Pod Metadata

Useful Pod attributes include:

```text
pod name
pod UID
namespace
node
container
```

Example:

```text
pod=payment-7d9f8c
node=node-2
```

This allows engineers to connect an application trace to a specific Kubernetes workload.

---

# 33. Trace Sampling

Large Kubernetes environments can generate enormous trace volumes.

Example:

```text
100,000 requests/sec
```

If every request produces multiple spans, trace volume becomes very large.

Sampling reduces the amount of telemetry retained.

---

# 34. Head Sampling

Head sampling makes the decision near the beginning of the trace.

Example:

```text
Request
  ↓
Sampling Decision
  ├── Keep
  └── Drop
```

This is efficient but the decision is made before the entire trace is known.

---

# 35. Tail Sampling

Tail sampling waits until enough information about the trace is available.

Example:

```text
Trace
 ↓
Collect spans
 ↓
Evaluate trace
 ├── Error → Keep
 ├── Slow → Keep
 └── Normal → Sample
```

This can retain more valuable traces.

---

# 36. Sampling Policies

Useful policies can prioritize:

```text
Errors
Slow traces
Important services
Specific environments
Specific HTTP status codes
```

Example:

```text
Error traces → 100%
Slow traces  → 100%
Normal       → 10%
```

Actual percentages should be based on workload and storage capacity.

---

# 37. Trace Retention

Trace data consumes storage.

Retention depends on:

```text
Trace volume
Storage capacity
Operational requirements
Incident investigation requirements
Cost
```

A production environment may retain traces for a limited period and rely on metrics/logs for longer-term historical analysis.

---

# 38. Trace Attributes

Attributes provide context about spans.

Examples:

```text
http.request.method
http.response.status_code
server.address
db.system
db.operation.name
service.name
deployment.environment
```

Useful attributes make traces easier to filter and analyze.

---

# 39. Avoid High Cardinality

Like metrics, tracing attributes should be designed carefully.

Avoid unnecessary high-cardinality data such as:

```text
Passwords
Tokens
Sensitive IDs
Large request bodies
Large response bodies
```

Trace attributes should provide diagnostic value without exposing sensitive information or creating unnecessary storage costs.

---

# 40. Trace Context Propagation

Suppose:

```text
Orders
  ↓
Payment
```

The trace context must be propagated from Orders to Payment.

Conceptually:

```text
Orders
 ├── trace_id=abc123
 ↓
Payment
 ├── trace_id=abc123
 ↓
Inventory
 ├── trace_id=abc123
```

Without context propagation, services may create unrelated traces.

---

# 41. HTTP Context Propagation

For HTTP calls:

```text
Service A
   ↓
HTTP Request + Trace Context
   ↓
Service B
```

Service B extracts the context and continues the same trace.

This allows:

```text
Service A
   ↓
Service B
```

to appear as one distributed trace.

---

# 42. Asynchronous Messaging

Tracing is also important for asynchronous systems.

Example:

```text
Orders
   ↓
Message Queue
   ↓
Payment Consumer
```

Trace context can be propagated through the message where supported.

Then:

```text
Producer Span
     ↓
Messaging Span
     ↓
Consumer Span
```

This helps trace asynchronous workflows.

---

# 43. Database Tracing

A database operation can appear as a child span.

Example:

```text
Payment
   │
   └── Database Query
```

Trace:

```text
Payment
  └── SELECT payment
```

This can reveal:

```text
Slow queries
Connection delays
Database errors
```

Tracing does not replace database monitoring, but it provides request-level context.

---

# 44. External API Tracing

Suppose:

```text
Orders
   ↓
External Payment API
```

Tracing can show:

```text
Orders
   └── External API
         └── 1.8 seconds
```

This helps distinguish:

```text
Application latency
from
External dependency latency
```

---

# 45. Kubernetes Ingress Tracing

If the ingress layer supports tracing instrumentation, the request can begin at the ingress.

Conceptually:

```text
Client
  ↓
Ingress / ALB
  ↓
Orders
  ↓
Payment
```

Trace:

```text
Ingress Span
   ↓
Orders Span
   ↓
Payment Span
```

The exact tracing capabilities depend on the ingress/load-balancer implementation.

---

# 46. Service Mesh Tracing

A service mesh can provide tracing instrumentation through sidecars or ambient data-plane components depending on the technology.

Conceptually:

```text
Application
   ↓
Proxy
   ↓
Network
   ↓
Proxy
   ↓
Application
```

Tracing can then capture service-to-service traffic.

However, a service mesh is not required for distributed tracing.

OpenTelemetry SDK instrumentation can provide application-level tracing without introducing a service mesh.

---

# 47. Kubernetes Tracing Without Service Mesh

A simple architecture is:

```text
Application
    ↓
OpenTelemetry SDK
    ↓
OTLP
    ↓
OTel Collector
    ↓
Jaeger
```

This is often easier to operate when a service mesh is not otherwise required.

---

# 48. Sidecar Collector Pattern

Another architecture is:

```text
Pod
├── Application
└── OTel Collector Sidecar
```

Flow:

```text
Application
    ↓
Sidecar Collector
    ↓
Gateway
    ↓
Jaeger
```

Advantages:

```text
Local processing
Isolation
Per-Pod configuration
```

Disadvantages:

```text
More containers
More resource consumption
More operational complexity
```

The Agent/Gateway model is often more resource-efficient for large clusters.

---

# 49. DaemonSet Collector Pattern

A common Kubernetes design is:

```text
Node
├── Application Pods
└── OTel Agent
```

Flow:

```text
Application
     ↓
Agent
     ↓
Gateway
     ↓
Jaeger
```

This reduces the number of Collector instances compared with a sidecar-per-Pod model.

---

# 50. Collector Gateway Scaling

Gateway replicas can be scaled horizontally.

```text
                 Service
                    │
          ┌─────────┼─────────┐
          ↓         ↓         ↓
      Gateway-1 Gateway-2 Gateway-3
```

Monitor:

```text
CPU
Memory
Trace ingestion rate
Export failures
Queue size
Dropped spans
```

---

# 51. Trace Pipeline Backpressure

Suppose the Jaeger backend slows down:

```text
Collector
   ↓
Jaeger
   X
```

The Collector may experience:

```text
Queue growth
Memory growth
Export failures
Dropped telemetry
```

Production configuration should include appropriate:

```text
Memory limits
Batching
Retry
Bounded queues
Sampling
```

---

# 52. Collector Memory Protection

A tracing pipeline can experience sudden traffic spikes.

Example:

```text
Trace volume ↑
      ↓
Collector memory ↑
```

Memory protection helps prevent:

```text
OOMKilled
Pod restart
Node memory pressure
```

Monitor Collector memory and queue utilization.

---

# 53. Kubernetes Resource Requests

Tracing components need appropriate resource requests.

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
```

Actual values should be determined through workload testing.

Under-requesting resources can cause scheduling and performance problems.

---

# 54. Kubernetes Resource Limits

Example:

```yaml
resources:
  limits:
    cpu: "1"
    memory: 1Gi
```

Limits should be tested carefully.

Too low:

```text
OOMKilled
CPU throttling
```

Too high:

```text
Poor resource utilization
Scheduling challenges
```

---

# 55. High Availability

Production tracing should avoid a single point of failure.

Bad:

```text
One Collector
One Jaeger instance
One Query
One storage node
```

Better:

```text
Multiple Collector replicas
Multiple Query replicas
Highly available storage
Multi-node distribution
```

---

# 56. Multi-AZ Tracing

For production EKS:

```text
AZ-A
├── Collector
└── Query

AZ-B
├── Collector
└── Query

AZ-C
├── Collector
└── Query
```

Use appropriate topology spread or anti-affinity rules.

The storage architecture must also support the desired availability model.

---

# 57. Monitoring Tracing Infrastructure

The tracing platform itself must be monitored.

Monitor:

```text
Collector
├── CPU
├── Memory
├── Queue
├── Receive rate
├── Export rate
└── Dropped spans

Jaeger
├── Ingestion
├── Query latency
├── Query errors
└── Storage health
```

Prometheus and Grafana can provide this visibility.

---

# 58. Trace-to-Log Correlation

Suppose a trace has:

```text
trace_id=abc123
```

The application logs should ideally contain:

```text
trace_id=abc123
```

Then:

```text
Jaeger
   ↓
Trace ID
   ↓
Kibana
   ↓
Matching logs
```

This dramatically improves incident investigation.

---

# 59. Trace-to-Metric Correlation

Suppose Grafana shows:

```text
Payment p95 latency ↑
```

The engineer can open Jaeger and investigate:

```text
Payment
   ↓
Database
   ↓
External API
```

This allows the engineer to move from:

```text
Metric
 ↓
Trace
 ↓
Root Cause
```

---

# 60. Complete Observability Workflow

A production incident might follow:

```text
Grafana
   ↓
High 5xx rate
   ↓
Kibana
   ↓
Application timeout
   ↓
Jaeger
   ↓
Slow database span
   ↓
Database investigation
```

The three telemetry types complement each other.

---

# 61. Example: Slow Microservice

Suppose users report:

```text
Checkout is slow.
```

Grafana:

```text
checkout p95 = 2.5s
```

Jaeger:

```text
Checkout
 ├── Orders = 100ms
 ├── Payment = 1.9s
 └── Inventory = 200ms
```

Then Payment:

```text
Payment
 └── Database = 1.7s
```

Kibana:

```text
Database connection timeout
```

Root cause investigation is now focused on the database path.

---

# 62. Example: 503 Error

Suppose:

```text
Users receive HTTP 503.
```

Metrics:

```text
5xx rate ↑
```

Logs:

```text
Service unavailable
```

Tracing:

```text
Orders
  ↓
Payment
  X
```

The Payment span shows:

```text
connection refused
```

Kubernetes:

```text
Payment Pods
 ↓
Restarting
```

Now the issue can be traced across:

```text
Application
Kubernetes
Logs
Traces
```

---

# 63. Example: High Latency After Deployment

Suppose latency increases immediately after deployment.

Check:

```text
Deployment
   ↓
New Pods
   ↓
Trace comparison
```

Compare:

```text
Previous version
vs
New version
```

Look for:

```text
New slow spans
New database calls
New external API calls
New retries
```

Then correlate with application logs.

---

# 64. Trace Sampling Strategy

A practical strategy may be:

```text
Production
├── Errors → Keep
├── Slow traces → Keep
├── Important business flows → Keep
└── Normal requests → Sample
```

This prioritizes traces that are most useful during incidents.

---

# 65. Sampling Trade-Off

More sampling:

```text
More traces
↓
Higher storage
↓
Higher cost
```

Less sampling:

```text
Fewer traces
↓
Lower cost
↓
Less diagnostic information
```

The correct balance depends on:

```text
Traffic
Incident requirements
Storage
Cost
Business importance
```

---

# 66. Trace Retention Strategy

Example:

```text
Production
 → Short-term detailed traces

Staging
 → Moderate traces

Development
 → Minimal traces
```

Metrics and logs may have different retention periods.

Do not assume every telemetry type needs the same retention period.

---

# 67. Security and Privacy

Trace data may contain:

```text
Request attributes
URLs
Database information
User-related identifiers
Headers
Error messages
```

Do not automatically capture sensitive information.

Review:

```text
Instrumentation
Attributes
Request headers
Payload capture
Collector processors
Storage access
```

---

# 68. Authentication Headers

Avoid capturing sensitive headers such as:

```text
Authorization
Cookie
Set-Cookie
API keys
```

If headers are collected for diagnostic reasons, sensitive values should be appropriately excluded or sanitized.

---

# 69. Request and Response Bodies

Capturing complete request/response bodies can create:

```text
High storage usage
Privacy risks
Security risks
High network traffic
```

Only collect them when there is a clear and controlled requirement.

---

# 70. RBAC for Jaeger

Production Jaeger access should be controlled.

Example:

```text
Engineer
   ↓
Authentication
   ↓
Authorization
   ↓
Jaeger UI
```

Restrict production trace access to appropriate users and teams.

---

# 71. Private Network Access

A production Jaeger UI should generally not be directly exposed to the public internet.

Example:

```text
Engineer
   ↓
VPN / Corporate Network
   ↓
Internal Ingress / ALB
   ↓
Jaeger UI
```

Use appropriate security controls for the environment.

---

# 72. Kubernetes NetworkPolicy

NetworkPolicies can restrict traffic.

Example concept:

```text
Applications
      ↓
OTel Gateway
      ↓
Jaeger
      ↓
Storage
```

Only required communication paths should be permitted.

---

# 73. Jaeger Health Monitoring

Monitor:

```text
Jaeger availability
Ingestion rate
Query latency
Query error rate
Storage connectivity
Pod restarts
CPU
Memory
```

A tracing system that is unavailable during an incident provides little value.

---

# 74. Collector Health Monitoring

Monitor:

```text
Receiver traffic
Processor throughput
Exporter traffic
Export errors
Dropped spans
Queue size
CPU
Memory
```

Example:

```text
Receive = 50k spans/sec
Export = 49k spans/sec
```

If:

```text
Receive = 50k
Export = 20k
```

the pipeline is falling behind.

---

# 75. Trace Loss

Possible causes include:

```text
Collector overload
Sampling
Backend failure
Network failure
Queue overflow
Pod restart
Storage failure
```

Monitor:

```text
Dropped spans
Export failures
Queue utilization
Backend health
```

---

# 76. Kubernetes Events and Tracing

Tracing can be combined with Kubernetes events.

Example:

```text
10:00 → Deployment
10:01 → New Pods
10:02 → Latency increases
10:03 → Trace errors
10:04 → Pod restarts
```

This timeline helps identify whether a deployment or infrastructure event correlates with application behavior.

---

# 77. GitOps Deployment

Tracing infrastructure can be managed using GitOps.

```text
Git
 ↓
OpenTelemetry Configuration
 ↓
Jaeger Configuration
 ↓
Helm / Kubernetes Manifests
 ↓
ArgoCD
 ↓
EKS
```

Benefits:

```text
Version control
Audit trail
Repeatability
Rollback
Drift detection
```

---

# 78. Production Tracing Repository

Example:

```text
observability/
├── opentelemetry/
│   ├── agent/
│   ├── gateway/
│   └── sampling/
│
├── jaeger/
│   ├── values-dev.yaml
│   ├── values-staging.yaml
│   └── values-prod.yaml
│
└── argocd/
    ├── otel.yaml
    └── jaeger.yaml
```

The exact repository structure depends on the organization's GitOps conventions.

---

# 79. Tracing Deployment Strategy

Use:

```text
Development
    ↓
Staging
    ↓
Production
```

Validate:

```text
Instrumentation
Collector
Sampling
Storage
Query
UI
Security
Resource consumption
```

before production rollout.

---

# 80. Rolling Upgrade

For replicated tracing components:

```text
Old-1
Old-2
Old-3
```

Gradually replace them:

```text
New-1
Old-2
Old-3
```

then:

```text
New-1
New-2
Old-3
```

finally:

```text
New-1
New-2
New-3
```

Readiness checks should prevent unready Pods from receiving traffic.

---

# 81. Rollback

If a new tracing configuration causes problems:

```text
New configuration
      ↓
Problem
      ↓
Git revert
      ↓
ArgoCD
      ↓
Previous configuration
```

Verify:

```text
Trace ingestion
Collector health
Jaeger health
Storage
Query
UI
```

---

# 82. Disaster Recovery

A production tracing platform should consider:

```text
Cluster failure
Node failure
AZ failure
Storage failure
Configuration corruption
Accidental deletion
```

Recovery plans should cover:

```text
Collector configuration
Jaeger configuration
Storage
Secrets
GitOps manifests
```

---

# 83. Trace Data Recovery

Trace data recovery requirements depend on business needs.

Some organizations may prioritize:

```text
Configuration recovery
```

over:

```text
Long-term raw trace recovery
```

because traces are often retained for shorter periods than logs or metrics.

The recovery strategy should be explicitly defined.

---

# 84. Capacity Planning

Estimate:

```text
Requests/sec
×
Spans/request
=
Spans/sec
```

Example:

```text
10,000 requests/sec
×
8 spans/request
=
80,000 spans/sec
```

Then account for:

```text
Sampling
Average span size
Retention
Replication
Query workload
```

---

# 85. Storage Planning

Conceptually:

```text
Daily Storage
=
Spans/sec
×
Average Span Size
×
Seconds/Day
×
Sampling Percentage
```

Then add:

```text
Replication
Index overhead
Metadata
Operational overhead
```

Actual storage requirements should be validated through real workload measurements.

---

# 86. Trace Cardinality

Tracing can also suffer from excessive attribute cardinality.

Avoid unnecessary attributes such as:

```text
Unique user IDs
Random UUIDs
Large payloads
Unbounded URLs
```

Prefer controlled attributes such as:

```text
service.name
environment
HTTP method
HTTP route
status code
database system
```

---

# 87. URL Cardinality

Avoid using raw URLs with unique identifiers as a high-volume grouping dimension.

Bad:

```text
/api/users/938472/orders/827362
```

Prefer normalized routes:

```text
/api/users/{user_id}/orders/{order_id}
```

This keeps telemetry more useful and manageable.

---

# 88. Kubernetes Service Names

Tracing should use stable service names.

Example:

```text
orders
payment
inventory
notification
```

Avoid using dynamically generated Pod names as the primary service identity.

Pod names are useful metadata, but the logical service should remain stable.

---

# 89. Tracing and Kubernetes Deployments

Include version information where practical:

```text
service.name=payment
service.version=2.4.1
deployment.environment=production
```

This allows comparisons between application versions.

Example:

```text
Payment v2.3
→ p95 = 300ms

Payment v2.4
→ p95 = 900ms
```

Tracing can help identify what changed.

---

# 90. Trace-Based Troubleshooting Workflow

Use this sequence:

```text
1. Identify affected service
2. Check metrics
3. Find slow/error traces
4. Inspect trace spans
5. Identify dependency
6. Search logs using trace ID
7. Check Kubernetes Pod
8. Check Node/resource metrics
9. Confirm root cause
10. Remediate
11. Verify recovery
```

---

# 91. Production Incident Example

Problem:

```text
Payment latency increased.
```

Grafana:

```text
Payment p95 ↑
```

Jaeger:

```text
Payment
  ↓
Database
  ↓
1.8s
```

Kibana:

```text
Database connection timeout
```

Kubernetes:

```text
Payment Pods
→ Healthy
```

Conclusion:

```text
Application Pods are healthy.
Database dependency is the bottleneck.
```

This prevents unnecessary Kubernetes changes.

---

# 92. Production Incident: Pod Restart

Suppose:

```text
Payment Pod
 ↓
Restart count ↑
```

Metrics:

```text
Memory ↑
```

Kubernetes:

```text
OOMKilled
```

Logs:

```text
Application processing large request
```

Trace:

```text
Large transaction
 ↓
Payment
```

The combined signals provide a much stronger diagnosis than any one source.

---

# 93. Production Incident: External API

Suppose:

```text
Orders latency = 3s
```

Trace:

```text
Orders
 ├── Database = 200ms
 ├── Inventory = 300ms
 └── External API = 2.4s
```

This immediately points to the external API.

Without tracing, engineers may incorrectly scale the Kubernetes workload.

---

# 94. Tracing Best Practices

```text
1. Use OpenTelemetry for standardized instrumentation.
2. Use consistent service names.
3. Propagate trace context correctly.
4. Include environment metadata.
5. Include useful Kubernetes metadata.
6. Avoid sensitive attributes.
7. Avoid unnecessary high-cardinality attributes.
8. Use sampling for high-volume systems.
9. Monitor Collector health.
10. Monitor Jaeger health.
11. Use durable storage.
12. Run multiple replicas in production.
13. Distribute workloads across Nodes/AZs.
14. Correlate traces with logs and metrics.
15. Manage configuration through GitOps.
16. Test failure and recovery scenarios.
```

---

# 95. Kubernetes Tracing Architecture

```text
                         EKS CLUSTER
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       Orders              Payment             Inventory
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                    OpenTelemetry SDK
                              ↓
                             OTLP
                              ↓
                     OTel Agent DaemonSet
                              ↓
                   OTel Gateway Service
                              ↓
                ┌─────────────┼─────────────┐
                ↓             ↓             ↓
            Gateway-1     Gateway-2     Gateway-3
                └─────────────┼─────────────┘
                              ↓
                    Processing / Sampling
                              ↓
                           Jaeger
                              ↓
                     Durable Storage
                              ↓
                       Jaeger Query
                              ↓
                        Jaeger UI
```

---

# 96. Complete Kubernetes Observability

```text
                         KUBERNETES
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       Metrics               Logs              Traces
          ↓                   ↓                   ↓
     Prometheus               ELK            OpenTelemetry
          ↓                   ↓                   ↓
       Grafana              Kibana              Jaeger
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                       Incident Response
                              ↓
                          Root Cause
```

---

# 97. Interview Question

### How would you implement distributed tracing in Kubernetes?

**Answer:**

I would instrument the microservices using OpenTelemetry SDKs and export traces using OTLP. In Kubernetes, I would use OpenTelemetry Collectors, typically an Agent DaemonSet for local collection and a scalable Gateway Deployment for centralized processing, sampling, batching, and exporting. I would send the processed traces to Jaeger with durable storage. I would also include Kubernetes metadata, propagate trace context between services, monitor the tracing infrastructure using Prometheus and Grafana, and correlate trace IDs with centralized logs in ELK.

---

# 98. Interview Question

### What is the difference between a trace and a span?

**Answer:**

A trace represents the complete journey of a request across distributed services. A span represents one operation within that trace. For example, a checkout trace might contain Orders, Payment, Inventory, and Database spans. Each span has its own Span ID, while all spans belonging to the same request share the same Trace ID.

---

# 99. Interview Question

### How do you troubleshoot high latency using distributed tracing?

**Answer:**

I first confirm the latency increase through application metrics. Then I search Jaeger for slow traces and inspect the individual spans. I identify which service or dependency consumes most of the request duration. For example, if Payment takes 2 seconds and the database span takes 1.8 seconds, I would investigate the database path rather than immediately scaling Kubernetes Pods. I would then correlate the Trace ID with application logs in ELK.

---

# 100. Interview Question

### Why use OpenTelemetry with Jaeger?

**Answer:**

OpenTelemetry provides vendor-neutral instrumentation, trace context propagation, collection, processing, and OTLP export. Jaeger provides distributed tracing storage, querying, and visualization. This separation avoids tightly coupling application instrumentation to a specific backend and allows the telemetry pipeline to evolve independently.

---

# 101. Interview Question

### How do you reduce tracing cost?

**Answer:**

I would use appropriate sampling, prioritize error and slow traces, reduce unnecessary span attributes, avoid high-cardinality data, control retention, and right-size the Collector and storage infrastructure. I would not simply collect every trace indefinitely because high-volume microservices can generate a very large amount of telemetry.

---

# 102. Interview Question

### How do you make tracing highly available?

**Answer:**

I would run multiple OpenTelemetry Gateway replicas, distribute them across Nodes and Availability Zones, use multiple Query replicas where applicable, use durable highly available storage, configure resource limits and health probes, and monitor the entire tracing pipeline. I would also use bounded queues and retry mechanisms to handle temporary backend failures.

---

# 103. Interview Question

### What happens if the tracing backend becomes unavailable?

**Answer:**

The Collector may begin experiencing export failures and queue growth. I would check backend health, Collector queues, memory usage, and dropped spans. A production Collector should have appropriate batching, retry, bounded queueing, and memory protection. I would also verify that the backend has sufficient capacity and recovery mechanisms.

---

# 104. Interview Question

### How do you correlate logs and traces?

**Answer:**

I include the Trace ID in structured application logs. During an incident, I can identify a slow or failed trace in Jaeger, copy its Trace ID, and search Kibana for the same ID. This connects the distributed request path with the detailed application logs.

---

# 105. Interview Question

### Can you implement tracing without a service mesh?

**Answer:**

Yes. A service mesh is not required for distributed tracing. Applications can be instrumented with OpenTelemetry SDKs and export traces to an OpenTelemetry Collector, which can then send them to Jaeger. A service mesh can provide additional network-level telemetry, but it introduces additional infrastructure and operational complexity.

---

# 106. Interview Question

### How would you troubleshoot a trace that is missing?

**Answer:**

I would check the complete pipeline:

```text
Application
 ↓
OpenTelemetry SDK
 ↓
Trace Context
 ↓
OTLP
 ↓
Collector
 ↓
Sampling
 ↓
Exporter
 ↓
Jaeger
 ↓
Storage
 ↓
Query
 ↓
UI
```

I would determine where the trace disappears instead of assuming Jaeger is the problem.

---

# 107. Production Checklist

```text
INSTRUMENTATION
[ ] OpenTelemetry SDK
[ ] Automatic instrumentation where appropriate
[ ] Manual instrumentation where needed
[ ] Consistent service names
[ ] Version metadata
[ ] Environment metadata
[ ] Trace context propagation

COLLECTOR
[ ] OTel Agent
[ ] OTel Gateway
[ ] OTLP
[ ] Batching
[ ] Sampling
[ ] Retry
[ ] Memory protection
[ ] Bounded queues
[ ] Resource requests/limits

KUBERNETES
[ ] DaemonSet
[ ] Gateway Deployment
[ ] Multiple replicas
[ ] Pod distribution
[ ] Health probes
[ ] PDB
[ ] NetworkPolicy
[ ] RBAC

JAEGER
[ ] Ingestion
[ ] Query
[ ] UI
[ ] Durable storage
[ ] High availability
[ ] Retention

SECURITY
[ ] TLS
[ ] Authentication
[ ] Authorization
[ ] Sensitive-data filtering
[ ] Private access
[ ] Least privilege

OBSERVABILITY
[ ] Collector metrics
[ ] Jaeger metrics
[ ] Prometheus
[ ] Grafana
[ ] ELK integration
[ ] Trace ID in logs

OPERATIONS
[ ] Capacity planning
[ ] Sampling strategy
[ ] Retention strategy
[ ] Disaster recovery
[ ] Failure testing
[ ] GitOps
[ ] Rollback
```

---

# 108. Final Mental Model

Remember Kubernetes tracing as:

```text
                    USER REQUEST
                         │
                         ↓
                    INGRESS / ALB
                         │
                         ↓
                    SERVICE A
                         │
                     TRACE ID
                         │
                         ↓
                    SERVICE B
                         │
                     TRACE ID
                         │
                         ↓
                    SERVICE C
                         │
                         ↓
                    DATABASE
```

The telemetry pipeline:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTLP
     ↓
OTel Agent
     ↓
OTel Gateway
     ↓
Sampling / Processing
     ↓
Jaeger
     ↓
Storage
     ↓
Jaeger UI
```

And the complete observability model:

```text
Metrics
   ↓
Prometheus
   ↓
Grafana
   │
   ├───────────────┐
   ↓               ↓
Logs             Traces
   ↓               ↓
ELK          OpenTelemetry
   ↓               ↓
Kibana           Jaeger
   └───────┬───────┘
           ↓
      Root Cause Analysis
```

The key principle is:

**Kubernetes tracing follows an individual request across distributed services and shows where time is spent, where failures occur, and which dependencies are involved. OpenTelemetry provides standardized instrumentation, context propagation, and telemetry collection, while Jaeger provides trace storage, querying, and visualization. In production EKS, use an Agent/Gateway Collector architecture, appropriate sampling, durable storage, high availability, Kubernetes metadata, secure access, and correlation with Prometheus metrics and ELK logs.**
