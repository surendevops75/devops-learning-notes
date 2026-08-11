# OpenTelemetry–Jaeger Integration

## 1. Overview

OpenTelemetry and Jaeger are commonly used together in a distributed tracing architecture.

OpenTelemetry is responsible for:

```text
Instrumentation
Telemetry collection
Processing
Enrichment
Sampling
Export
```

Jaeger is responsible primarily for:

```text
Trace ingestion
Trace storage integration
Trace querying
Trace visualization
```

A typical architecture is:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTLP
     ↓
OpenTelemetry Collector
     ↓
Jaeger
     ↓
Trace Storage
     ↓
Jaeger Query
     ↓
Jaeger UI
```

---

# 2. Why Integrate OpenTelemetry with Jaeger?

OpenTelemetry provides a vendor-neutral telemetry pipeline.

Instead of applications directly depending on Jaeger-specific libraries:

```text
Application
     ↓
Jaeger SDK
     ↓
Jaeger
```

the preferred architecture is:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTLP
     ↓
Collector
     ↓
Jaeger
```

This provides greater flexibility.

The backend can potentially be changed without rewriting application instrumentation.

---

# 3. OpenTelemetry vs Jaeger

The responsibilities are different.

```text
OpenTelemetry
├── Instrumentation
├── SDKs
├── APIs
├── Context propagation
├── Collector
├── Processing
└── Export

Jaeger
├── Trace backend
├── Trace querying
└── Trace visualization
```

Think of it as:

```text
OpenTelemetry = telemetry generation and pipeline

Jaeger = tracing backend and UI
```

---

# 4. Complete Architecture

A production architecture can look like:

```text
                         APPLICATIONS
                              │
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
              Orders       Payment     Inventory
                 │            │            │
                 └────────────┼────────────┘
                              ↓
                     OpenTelemetry SDK
                              ↓
                             OTLP
                              ↓
                    OTel Collector Agent
                              ↓
                    OTel Collector Gateway
                              ↓
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
              Enrich       Sample        Batch
                 └────────────┼────────────┘
                              ↓
                           Jaeger
                              ↓
                       Trace Storage
                              ↓
                        Jaeger Query
                              ↓
                         Jaeger UI
```

---

# 5. OTLP as the Integration Protocol

OpenTelemetry applications commonly export traces using OTLP.

Two common protocols are:

```text
OTLP/gRPC
OTLP/HTTP
```

Typical ports:

```text
4317 → OTLP/gRPC
4318 → OTLP/HTTP
```

Example:

```text
Application
    ↓
OTLP/gRPC
    ↓
Collector
```

---

# 6. Application Configuration

An application needs:

```text
Service name
OTLP endpoint
Instrumentation
Resource attributes
Sampling configuration
Propagation configuration
```

Example:

```bash
export OTEL_SERVICE_NAME=payment
export OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-gateway:4317
```

The exact configuration depends on the language SDK.

---

# 7. Service Name

Every application should have a stable service name.

Example:

```text
orders
payment
inventory
notification
user
```

Example:

```bash
export OTEL_SERVICE_NAME=orders
```

The service name appears in Jaeger and is essential for trace searching.

---

# 8. Service Version

Include application version information.

Example:

```text
service.name=payment
service.version=2.4.1
```

This helps compare traces between deployments.

Example:

```text
payment v2.3 → 250ms
payment v2.4 → 900ms
```

---

# 9. Deployment Environment

Add environment information:

```text
deployment.environment=production
```

Possible values:

```text
development
staging
production
```

This is especially important when multiple environments send traces to the same backend.

---

# 10. Kubernetes Integration

For Kubernetes:

```text
Application Pods
      ↓
OTel Agent
      ↓
OTel Gateway
      ↓
Jaeger
```

The application should normally send telemetry to a Kubernetes Service instead of a Pod IP.

Example:

```text
otel-gateway.observability.svc.cluster.local
```

---

# 11. OpenTelemetry Agent/Gateway Model

A common Kubernetes design is:

```text
Node-1
├── Application Pods
└── OTel Agent

Node-2
├── Application Pods
└── OTel Agent

Node-3
├── Application Pods
└── OTel Agent
```

Then:

```text
OTel Agents
      ↓
OTel Gateway
      ↓
Jaeger
```

The Agent provides local collection while the Gateway centralizes processing and export.

---

# 12. Collector Receiver

The Collector receives OTLP:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:
```

This allows applications to send:

```text
OTLP/gRPC
OTLP/HTTP
```

---

# 13. Collector Processor Pipeline

A production pipeline can contain:

```text
OTLP Receiver
      ↓
Memory Limiter
      ↓
Kubernetes Attributes
      ↓
Sampling
      ↓
Batch
      ↓
Exporter
```

Each processor has a specific purpose.

---

# 14. Memory Limiter

The Memory Limiter protects the Collector from excessive memory usage.

Example:

```yaml
processors:
  memory_limiter:
    check_interval: 1s
    limit_percentage: 80
    spike_limit_percentage: 15
```

This is particularly important when trace volume suddenly increases.

---

# 15. Kubernetes Attributes

The Kubernetes Attributes Processor can enrich telemetry with Kubernetes metadata.

Conceptually:

```yaml
processors:
  k8sattributes:
```

Useful information includes:

```text
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
k8s.cluster.name
```

---

# 16. Batch Processor

Batching groups spans before export.

```text
Span 1 ┐
Span 2 ├──→ Batch
Span 3 ┘
          ↓
        Jaeger
```

Example:

```yaml
processors:
  batch:
```

Batching can reduce export overhead and improve efficiency.

---

# 17. Sampling

Sampling controls how much trace data is retained.

Example:

```text
Errors       → 100%
Slow traces  → 100%
Normal       → 5%
```

This reduces:

```text
Storage
Network traffic
Collector load
Backend cost
```

while retaining important traces.

---

# 18. Head Sampling

Head sampling makes a decision early.

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
Less network traffic
```

Disadvantage:

```text
The final result of the request may not yet be known.
```

An error could therefore be dropped.

---

# 19. Tail Sampling

Tail sampling makes a decision after observing the trace.

```text
Trace
  ↓
Collector
  ↓
Evaluate trace
  ↓
Sampling decision
```

Example:

```text
Error      → Keep
Slow       → Keep
Normal     → Sample
```

Tail sampling is particularly useful when the requirement is to retain important traces based on their final outcome.

---

# 20. Jaeger Export

The Collector sends processed traces to the Jaeger backend using the exporter supported by the chosen Collector distribution and Jaeger architecture.

Conceptually:

```text
OTel Collector
      ↓
Jaeger-compatible export
      ↓
Jaeger
```

The exact exporter configuration should always match the OpenTelemetry Collector and Jaeger versions being deployed.

---

# 21. Collector Pipeline Example

Conceptually:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  memory_limiter:
    check_interval: 1s
    limit_percentage: 80
    spike_limit_percentage: 15

  batch:

exporters:
  <jaeger-compatible-exporter>:
    endpoint: <jaeger-endpoint>

service:
  pipelines:
    traces:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - <jaeger-compatible-exporter>
```

The exporter section must be adapted to the actual Collector distribution and Jaeger version.

---

# 22. Direct Application-to-Jaeger Integration

A simple architecture can be:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
Jaeger
```

However, production environments generally benefit from a Collector:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
Collector
     ↓
Jaeger
```

The Collector provides:

```text
Centralized processing
Sampling
Filtering
Enrichment
Retry
Batching
Routing
```

---

# 23. Why Use the Collector?

Without a Collector:

```text
Application
     ↓
Jaeger
```

Every application becomes responsible for backend-specific configuration.

With a Collector:

```text
Applications
     ↓
Collector
     ↓
Backend
```

Backend configuration becomes centralized.

This also makes future backend changes easier.

---

# 24. Vendor-Neutral Application Design

A major advantage is:

```text
Application
     ↓
OpenTelemetry
     ↓
OTLP
     ↓
Collector
```

The application does not need to be tightly coupled to the tracing backend.

For example:

```text
                  ┌──→ Jaeger
Application → OTel Collector
                  ├──→ Another tracing backend
                  └──→ Additional telemetry destination
```

The exact routing depends on the Collector configuration.

---

# 25. Trace Context Propagation

Distributed tracing requires context propagation.

Example:

```text
Orders
  ↓
traceparent
  ↓
Payment
  ↓
traceparent
  ↓
Inventory
```

All participating services need compatible instrumentation and propagation.

---

# 26. W3C Trace Context

A common propagation standard is W3C Trace Context.

Important headers include:

```text
traceparent
tracestate
```

Example conceptual flow:

```text
Client
  ↓
traceparent
  ↓
Orders
  ↓
traceparent
  ↓
Payment
```

This allows multiple services to participate in the same trace.

---

# 27. Broken Propagation

Correct:

```text
Orders
  └── Payment
       └── Inventory
```

Broken propagation:

```text
Orders → Trace A

Payment → Trace B

Inventory → Trace C
```

The UI will show separate traces instead of one distributed trace.

---

# 28. HTTP Instrumentation

OpenTelemetry can automatically or manually instrument HTTP applications.

Useful span information includes:

```text
HTTP method
HTTP route
HTTP status
Duration
Service
```

Example:

```text
POST /payments
Status = 200
Duration = 120ms
```

Avoid collecting sensitive request data unnecessarily.

---

# 29. Database Instrumentation

Database operations can appear as child spans.

Example:

```text
Payment
   ↓
PostgreSQL
   ↓
SELECT
   ↓
80ms
```

This makes database latency visible inside the overall request trace.

---

# 30. Messaging Integration

Distributed tracing should also propagate through messaging systems.

Example:

```text
Orders
   ↓
RabbitMQ
   ↓
Notification
```

Trace context should be propagated with the message when supported by the instrumentation.

This allows asynchronous operations to remain connected to the originating trace.

---

# 31. External API Integration

Example:

```text
Payment
   ↓
External Payment Gateway
```

The trace can show:

```text
Payment
   └── External Gateway
          Duration = 1.4s
```

This makes external dependency latency visible.

---

# 32. Trace-to-Log Correlation

The tracing system becomes much more powerful when Trace IDs are included in application logs.

Architecture:

```text
Jaeger
   ↓
Trace ID
   ↓
ELK
   ↓
Application logs
```

Example:

```text
trace_id=abc123
span_id=def456
```

Search the same Trace ID in Kibana.

---

# 33. Trace-to-Metric Correlation

Metrics identify abnormal behavior:

```text
Prometheus
   ↓
Grafana
   ↓
High latency
```

Then Jaeger identifies the affected operation:

```text
Grafana
   ↓
Payment latency ↑
   ↓
Jaeger
   ↓
Database span = 900ms
```

Then ELK can identify the application error or database warning.

---

# 34. Complete Observability Workflow

A production investigation can follow:

```text
Metric
  ↓
Trace
  ↓
Log
  ↓
Root Cause
```

Example:

```text
Prometheus
    ↓
High p95 latency
    ↓
Jaeger
    ↓
Slow Payment span
    ↓
ELK
    ↓
Database timeout
```

---

# 35. Jaeger UI

After traces reach Jaeger:

```text
Application
     ↓
OpenTelemetry
     ↓
Collector
     ↓
Jaeger
     ↓
Storage
     ↓
Query
     ↓
UI
```

The engineer can search by:

```text
Service
Operation
Time range
Duration
Trace ID
Tags / attributes
```

---

# 36. End-to-End Example

Consider:

```text
Frontend
   ↓
Orders
   ↓
Payment
   ↓
Inventory
```

OpenTelemetry creates spans:

```text
Frontend Span
    │
    └── Orders Span
          │
          ├── Payment Span
          │
          └── Inventory Span
```

Collector processes them:

```text
Receive
 ↓
Enrich
 ↓
Sample
 ↓
Batch
```

Then:

```text
Collector
   ↓
Jaeger
```

Jaeger stores and displays the trace.

---

# 37. Kubernetes End-to-End Flow

In EKS:

```text
Orders Pod
    ↓
OTel SDK
    ↓
OTLP
    ↓
OTel Agent DaemonSet
    ↓
OTel Gateway Service
    ↓
Gateway Pods
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

# 38. Service Configuration in Kubernetes

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: orders

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: http://otel-gateway.observability.svc.cluster.local:4317
```

The endpoint should match:

```text
Protocol
Service
Namespace
Port
```

---

# 39. Kubernetes Service Verification

Check the Collector Service:

```bash
kubectl get svc -n observability
```

Check endpoints:

```bash
kubectl get endpoints -n observability
```

Check Pods:

```bash
kubectl get pods -n observability
```

The Service should have healthy endpoints.

---

# 40. Testing the Integration

After deploying the application and tracing stack:

```text
1. Generate application traffic
2. Verify application instrumentation
3. Verify Collector receives telemetry
4. Verify Collector exports telemetry
5. Verify Jaeger receives traces
6. Verify storage
7. Open Jaeger UI
8. Search for the service
9. Open a trace
10. Inspect spans
```

Do not consider the integration successful merely because all Pods show `Running`.

---

# 41. Generate Test Traffic

For an HTTP application:

```bash
curl http://<application-endpoint>/orders
```

Generate multiple requests:

```bash
for i in {1..20}; do
  curl -s http://<application-endpoint>/orders > /dev/null
done
```

Then search for the service in Jaeger.

---

# 42. Verify Collector Logs

Check:

```bash
kubectl logs <otel-collector-pod> -n observability
```

Look for:

```text
Receiver activity
Exporter activity
Connection errors
Authentication errors
TLS errors
Dropped spans
```

Avoid enabling excessive debug logging permanently in production.

---

# 43. Verify Jaeger

Check:

```bash
kubectl get pods -n observability
```

Then inspect Jaeger logs if necessary:

```bash
kubectl logs <jaeger-pod> -n observability
```

Look for:

```text
Storage errors
Connection errors
Configuration errors
Authentication failures
```

---

# 44. Verify Storage

If Jaeger receives traces but the UI shows nothing, investigate the storage path.

```text
Jaeger
   ↓
Storage
```

Check:

```text
Storage endpoint
Connectivity
Credentials
TLS
Storage health
Retention
```

---

# 45. Verify Jaeger UI

Search:

```text
Service = orders
```

Set an appropriate time range.

Then open a trace.

Expected:

```text
Orders
   ↓
Child operations
   ↓
Database / downstream services
```

If the trace appears, the integration is working end to end.

---

# 46. Troubleshooting: No Traces

Follow the pipeline:

```text
Application
   ↓
SDK
   ↓
OTLP
   ↓
Collector
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

Check each stage rather than randomly changing configuration.

---

# 47. Troubleshooting: Collector Receives Nothing

Check:

```text
OTEL_EXPORTER_OTLP_ENDPOINT
OTLP protocol
Collector Service
Service port
NetworkPolicy
Application instrumentation
```

Verify the application can resolve the Collector Service.

---

# 48. Troubleshooting: Collector Receives but Cannot Export

Possible causes:

```text
Incorrect Jaeger endpoint
Incorrect port
TLS mismatch
Authentication failure
NetworkPolicy
Storage/backend problem
Exporter configuration
```

Inspect Collector logs.

---

# 49. Troubleshooting: Jaeger Receives but UI Is Empty

Check:

```text
Storage
Query
Time range
Service name
Sampling
```

Also verify that the trace was not dropped by the sampling policy.

---

# 50. Troubleshooting: Partial Trace

Suppose:

```text
Orders
  ↓
Payment
```

but Jaeger shows only:

```text
Orders
```

Check:

```text
Trace propagation
Payment instrumentation
Payment OTLP endpoint
Payment Collector route
Sampling
```

---

# 51. Troubleshooting: Broken Trace Context

Symptoms:

```text
Multiple independent traces
Missing child spans
Different Trace IDs between services
```

Check:

```text
W3C propagation
HTTP header forwarding
Messaging propagation
Instrumentation
Proxy configuration
```

---

# 52. Troubleshooting: High Collector Memory

Check:

```text
Trace volume
Sampling
Queue
Batch
Large traces
High-cardinality attributes
Backend availability
```

Architecture:

```text
Backend unavailable
       ↓
Queue grows
       ↓
Memory increases
```

Memory protection should be configured.

---

# 53. Troubleshooting: High Storage Usage

Possible causes:

```text
High trace volume
High sampling rate
Large traces
Long retention
High-cardinality attributes
Unnecessary spans
```

Actions:

```text
Adjust sampling
Filter low-value telemetry
Reduce retention
Review instrumentation
Scale storage
```

---

# 54. Production Sampling Strategy

A practical strategy:

```text
Error traces       → 100%
Slow traces        → 100%
Normal traces      → 5%
```

For example:

```text
10,000 requests/sec
×
10 spans/request
=
100,000 spans/sec
```

At 5% normal sampling:

```text
Much lower retained volume
```

The exact sampling policy should be based on production requirements.

---

# 55. Security

Protect the complete pipeline:

```text
Application
   ↓
Collector
   ↓
Jaeger
   ↓
Storage
```

Security controls can include:

```text
TLS
RBAC
NetworkPolicy
Security Groups
Private subnets
Secrets
Authentication
Authorization
```

---

# 56. Sensitive Data Filtering

Do not blindly collect:

```text
Passwords
Tokens
Authorization headers
Cookies
Payment credentials
Sensitive request bodies
Personal information
```

Use instrumentation configuration and Collector processors to remove sensitive information.

---

# 57. High Cardinality

Avoid unnecessary attributes such as:

```text
random UUID
session ID
full URL with unique query parameters
user-specific values
```

High-cardinality telemetry can increase storage and query costs.

Prefer controlled attributes such as:

```text
service.name
service.version
http.route
http.method
status
environment
```

---

# 58. Production High Availability

A production architecture should avoid single points of failure.

```text
Applications
    ↓
Multiple Collectors
    ↓
Multiple Jaeger components
    ↓
Highly available storage
    ↓
Multiple Query replicas
```

Distribute replicas across nodes and Availability Zones where appropriate.

---

# 59. Collector Scaling

If trace volume increases:

```text
Trace volume ↑
     ↓
Collector CPU ↑
     ↓
Add replicas
```

Kubernetes Service distributes traffic among healthy Collector Pods.

Use HPA where appropriate and where scaling signals are reliable.

---

# 60. Jaeger Query Scaling

If many engineers are searching traces:

```text
Query traffic ↑
      ↓
Query CPU ↑
      ↓
Scale Query replicas
```

Storage performance must also be sufficient.

---

# 61. Storage Scaling

Trace storage is often the largest infrastructure component.

Consider:

```text
Ingestion rate
Retention
Replication
Disk capacity
Query rate
Recovery
```

Do not scale only the Jaeger application while ignoring storage capacity.

---

# 62. GitOps Integration

For an EKS production environment:

```text
Git
 ↓
OpenTelemetry configuration
 ↓
Jaeger configuration
 ↓
Helm values
 ↓
Pull Request
 ↓
Review
 ↓
ArgoCD
 ↓
EKS
```

This keeps tracing configuration version controlled.

---

# 63. Configuration Repository

Example:

```text
observability/
├── otel/
│   ├── agent-values.yaml
│   ├── gateway-values.yaml
│   └── sampling.yaml
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

The exact layout can vary by organization.

---

# 64. Deployment Workflow

```text
Developer
    ↓
Application code
    ↓
OpenTelemetry instrumentation
    ↓
Build
    ↓
Container image
    ↓
Deploy to EKS
    ↓
Application generates traces
    ↓
Collector
    ↓
Jaeger
```

For GitOps:

```text
Application deployment
        +
Observability configuration
        ↓
Git
        ↓
ArgoCD
        ↓
EKS
```

---

# 65. Deployment Validation

After deployment:

```text
[ ] Application is healthy
[ ] Collector Pods are healthy
[ ] Jaeger Pods are healthy
[ ] Storage is healthy
[ ] Test traffic generated
[ ] Traces appear
[ ] Trace IDs correlate with logs
[ ] Service metadata is correct
[ ] Sampling works
[ ] No excessive errors
[ ] Resource usage is acceptable
```

---

# 66. Real-World Microservices Example

Consider:

```text
Frontend
   ↓
Orders
   ↓
Payment
   ↓
Inventory
   ↓
Notification
```

OpenTelemetry creates:

```text
Frontend Span
     ↓
Orders Span
     ├── Payment Span
     │      └── Payment Gateway
     │
     ├── Inventory Span
     │
     └── Notification Span
```

Collector:

```text
Receive
 ↓
Enrich
 ↓
Sample
 ↓
Batch
 ↓
Export
```

Jaeger:

```text
Store
 ↓
Query
 ↓
Visualize
```

---

# 67. Real-World Latency Investigation

Problem:

```text
Checkout p95:
300ms → 1.5s
```

Grafana:

```text
High latency detected
```

Jaeger:

```text
Checkout
   ↓
Orders
   ↓
Payment
   ↓
External Gateway
   ↓
1.2s
```

ELK:

```text
Payment timeout warning
```

Kubernetes:

```text
Payment Pods healthy
```

Conclusion:

```text
External Payment dependency is the likely bottleneck.
```

This demonstrates the value of integrating:

```text
Prometheus + Grafana
ELK
OpenTelemetry
Jaeger
```

---

# 68. Real-World Deployment Regression

Before release:

```text
payment v2.3
p95 = 250ms
```

After release:

```text
payment v2.4
p95 = 900ms
```

Jaeger trace:

```text
payment v2.4
   ↓
Database
   ↓
750ms
```

ELK:

```text
Slow database query
```

The team can investigate the release and roll back if necessary.

---

# 69. Interview Question

### Why use OpenTelemetry with Jaeger?

**Answer:**

OpenTelemetry provides vendor-neutral instrumentation, telemetry collection, processing, sampling, and OTLP export, while Jaeger provides distributed trace storage, querying, and visualization. Using OpenTelemetry with Jaeger separates application instrumentation from the backend and provides a flexible production telemetry pipeline.

---

# 70. Interview Question

### Why use an OpenTelemetry Collector between the application and Jaeger?

**Answer:**

The Collector provides a centralized place for processing telemetry. I can use it for batching, memory protection, filtering, Kubernetes metadata enrichment, sampling, retries, and routing. This avoids putting backend-specific configuration and processing responsibilities into every application.

---

# 71. Interview Question

### How would you integrate OpenTelemetry with Jaeger in EKS?

**Answer:**

I would instrument the microservices with OpenTelemetry SDKs and export traces using OTLP. In Kubernetes, I would deploy OpenTelemetry Agents as a DaemonSet and centralized Gateways as a Deployment. The Gateway would process, enrich, sample, batch, and export traces to Jaeger. Jaeger would use durable storage, with Query and UI components exposed through secure internal Kubernetes networking. I would monitor the entire pipeline using Prometheus and Grafana and correlate Trace IDs with ELK logs.

---

# 72. Interview Question

### What would you do if traces are not appearing in Jaeger?

**Answer:**

I would troubleshoot the complete path:

```text
Application
 ↓
OpenTelemetry SDK
 ↓
OTLP
 ↓
Collector
 ↓
Jaeger
 ↓
Storage
 ↓
Query
 ↓
UI
```

I would first verify the application's OTLP endpoint and instrumentation, then check Collector receivers and logs, exporter connectivity, Jaeger health, storage connectivity, Query health, and finally the UI time range and service filters.

---

# 73. Interview Question

### How do you reduce tracing cost in production?

**Answer:**

I would use appropriate sampling, especially retaining errors and slow traces while sampling normal traffic. I would filter unnecessary spans, avoid excessive high-cardinality attributes, use batching, control retention, and scale the infrastructure based on actual trace volume.

---

# 74. Interview Question

### How do you correlate Jaeger and ELK?

**Answer:**

I configure application logs to contain the Trace ID and Span ID. When I identify a problematic trace in Jaeger, I copy the Trace ID and search for it in Kibana. This lets me correlate the distributed request with application logs and identify exceptions, timeouts, retries, or other failures.

---

# 75. Interview Question

### How does tracing help with Kubernetes troubleshooting?

**Answer:**

I can use Kubernetes metadata such as namespace, Pod, container, and cluster information in spans. If Jaeger shows that a specific service or Pod is slow, I can use the Pod information to inspect Kubernetes logs, resource usage, events, and deployment details. This connects application-level behavior with Kubernetes infrastructure.

---

# 76. Production Checklist

```text
OPENTELEMETRY
[ ] SDK instrumentation
[ ] OTLP configured
[ ] Service name
[ ] Service version
[ ] Environment
[ ] Trace propagation
[ ] Kubernetes metadata

COLLECTOR
[ ] OTLP receiver
[ ] Memory limiter
[ ] Kubernetes attributes
[ ] Sampling
[ ] Batch
[ ] Retry
[ ] Queue
[ ] Jaeger exporter
[ ] Multiple replicas

JAEGER
[ ] Ingestion
[ ] Storage
[ ] Query
[ ] UI
[ ] Resource limits
[ ] Health probes
[ ] High availability

KUBERNETES
[ ] Services
[ ] DNS
[ ] RBAC
[ ] NetworkPolicy
[ ] PDB
[ ] Anti-affinity / topology spread
[ ] Multi-AZ where required

SECURITY
[ ] TLS
[ ] Authentication
[ ] Authorization
[ ] Secrets
[ ] Sensitive-data filtering
[ ] Private networking

OPERATIONS
[ ] Prometheus metrics
[ ] Grafana dashboards
[ ] Alerts
[ ] Trace-to-log correlation
[ ] Capacity planning
[ ] Retention
[ ] Backup/recovery
[ ] GitOps
[ ] Rollback
```

---

# 77. Final Mental Model

Remember the integration as:

```text
                         APPLICATION
                              │
                       OTel SDK / API
                              │
                             OTLP
                              │
                              ↓
                    OTel Collector Agent
                              │
                              ↓
                    OTel Collector Gateway
                              │
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
              Enrich       Sample        Batch
                 └────────────┼────────────┘
                              ↓
                           Jaeger
                              │
                              ↓
                       Trace Storage
                              │
                              ↓
                        Jaeger Query
                              │
                              ↓
                          Jaeger UI
```

The broader production observability architecture is:

```text
                     Application
                          │
          ┌───────────────┼────────────────┐
          ↓               ↓                ↓
       Metrics           Logs            Traces
          ↓               ↓                ↓
    Prometheus            ELK       OpenTelemetry
          ↓                                ↓
       Grafana                           Jaeger
```

The key principle is:

**Use OpenTelemetry as the standardized instrumentation and telemetry pipeline, OTLP as the transport, the OpenTelemetry Collector as the centralized processing layer, and Jaeger as the distributed tracing backend and visualization platform. In Kubernetes/EKS, combine this with Agent/Gateway deployment, Kubernetes metadata enrichment, controlled sampling, secure networking, durable storage, GitOps, and correlation with Prometheus/Grafana and ELK.**
