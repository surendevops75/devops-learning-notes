# OpenTelemetry Fundamentals

## 1. Overview

OpenTelemetry, commonly abbreviated as **OTel**, is an open-source observability framework for generating, collecting, processing, and exporting telemetry data.

The three primary telemetry signals are:

```text
Metrics
Logs
Traces
```

OpenTelemetry provides a common framework for instrumenting applications and sending telemetry to observability backends.

A simplified architecture is:

```text
Application
     │
     ↓
OpenTelemetry SDK
     │
     ↓
OpenTelemetry Collector
     │
     ├──────────────┬──────────────┐
     ↓              ↓              ↓
  Metrics         Logs          Traces
     │              │              │
     ↓              ↓              ↓
 Prometheus      ELK           Jaeger
```

OpenTelemetry does not have to be the final storage or visualization system. It acts primarily as a standardized telemetry generation and collection layer.

---

# 2. Why OpenTelemetry Exists

Modern applications are distributed across:

```text
Microservices
Containers
Kubernetes
Cloud infrastructure
Databases
Message queues
External APIs
Multiple regions
```

A single request can travel through many components:

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

Without distributed observability, understanding the complete request becomes difficult.

OpenTelemetry provides standardized instrumentation and telemetry collection.

---

# 3. Traditional Observability Problem

Historically, applications often used vendor-specific libraries.

For example:

```text
Application
    ├── Vendor A SDK
    ├── Vendor B SDK
    └── Vendor C SDK
```

Changing observability backends could require significant application changes.

OpenTelemetry aims to provide a vendor-neutral approach:

```text
Application
     ↓
OpenTelemetry
     ↓
Any compatible backend
```

This reduces dependency on a single observability vendor.

---

# 4. OpenTelemetry Components

OpenTelemetry consists of several major components:

```text
OpenTelemetry
│
├── APIs
├── SDKs
├── Instrumentation
├── Collector
├── Semantic Conventions
└── Protocols
```

Each component has a different responsibility.

---

# 5. OpenTelemetry API

The API defines interfaces that applications and instrumentation use to create telemetry.

Conceptually:

```text
Application
     ↓
OpenTelemetry API
     ↓
Telemetry
```

The API provides abstractions for:

```text
Tracing
Metrics
Logging
Context propagation
```

---

# 6. OpenTelemetry SDK

The SDK provides the implementation behind the API.

Conceptually:

```text
Application
     ↓
OTel API
     ↓
OTel SDK
     ↓
Telemetry processing/export
```

The SDK handles tasks such as:

```text
Sampling
Batching
Processing
Resource configuration
Exporting
```

---

# 7. Instrumentation

Instrumentation is how an application generates telemetry.

There are two major approaches:

```text
Automatic Instrumentation
Manual Instrumentation
```

---

# 8. Automatic Instrumentation

Automatic instrumentation uses libraries or agents that instrument supported frameworks without requiring developers to add telemetry code everywhere.

Example:

```text
Java Application
      ↓
OTel Java Agent
      ↓
HTTP requests
Database calls
Framework operations
      ↓
Telemetry
```

This is useful when teams want broad instrumentation with minimal application-code changes.

---

# 9. Manual Instrumentation

Developers can explicitly create telemetry.

Conceptually:

```text
Start Span
    ↓
Execute Business Logic
    ↓
Add Attributes
    ↓
Record Error
    ↓
End Span
```

Manual instrumentation is useful for business-specific operations that automatic instrumentation cannot understand.

---

# 10. Automatic + Manual Instrumentation

Production applications can use both.

```text
Application
    │
    ├── Automatic Instrumentation
    │       ↓
    │    HTTP / DB / Messaging
    │
    └── Manual Instrumentation
            ↓
        Business Operations
```

This provides broad technical visibility plus application-specific visibility.

---

# 11. Three Major Signals

OpenTelemetry works with:

```text
Metrics
Logs
Traces
```

### Metrics

Metrics represent numerical measurements.

Examples:

```text
HTTP request count
CPU usage
Request latency
Error rate
Queue depth
```

### Logs

Logs record individual events.

Example:

```text
Payment database connection failed
```

### Traces

Traces represent the journey of a request through distributed systems.

```text
Frontend
   ↓
Orders
   ↓
Payment
   ↓
Database
```

---

# 12. Metrics

Metrics answer:

> What is happening?

Example:

```text
HTTP requests = 10,000/min
Error rate = 4%
Latency = 800 ms
```

OpenTelemetry can generate and export metrics to compatible monitoring systems.

For example:

```text
Application
     ↓
OTel SDK
     ↓
OTel Collector
     ↓
Prometheus
     ↓
Grafana
```

---

# 13. Logs

Logs answer:

> What happened?

Example:

```text
2026-08-11 10:30:25 ERROR Database connection timeout
```

OpenTelemetry can standardize and transport log telemetry.

Example architecture:

```text
Application
     ↓
OTel
     ↓
Collector
     ↓
Logging Backend
```

The backend could be an observability platform such as Elasticsearch or another compatible system.

---

# 14. Traces

Traces answer:

> Where did the request travel and where did it fail?

Example:

```text
Trace
│
├── Frontend
│
├── Orders
│
├── Payment
│
└── Database
```

A trace is composed of spans.

```text
Trace
  ├── Span: Frontend
  ├── Span: Orders
  ├── Span: Payment
  └── Span: Database
```

---

# 15. Telemetry

Telemetry is the data generated to understand system behavior.

The major categories are:

```text
Metrics
Logs
Traces
```

Example:

```text
Metrics
    ↓
Latency increased

Logs
    ↓
Database timeout

Trace
    ↓
Payment → Database → Timeout
```

Together they provide much better visibility than any single signal.

---

# 16. OpenTelemetry Collector

The Collector is one of the most important OpenTelemetry components.

It can:

```text
Receive telemetry
Process telemetry
Transform telemetry
Filter telemetry
Batch telemetry
Export telemetry
```

Architecture:

```text
Applications
     ↓
OpenTelemetry Collector
     ↓
Backend
```

---

# 17. Collector Pipeline

A Collector pipeline generally follows:

```text
Receivers
    ↓
Processors
    ↓
Exporters
```

Conceptually:

```text
Telemetry
    ↓
Receiver
    ↓
Processor
    ↓
Exporter
    ↓
Backend
```

---

# 18. Receivers

Receivers accept telemetry from applications and systems.

Examples include protocols and formats such as:

```text
OTLP
Prometheus
Jaeger
Zipkin
```

A receiver is the entry point into a Collector pipeline.

---

# 19. Processors

Processors modify or control telemetry before export.

Common processing tasks include:

```text
Batching
Filtering
Sampling
Adding attributes
Removing attributes
Memory limiting
```

Example:

```text
Telemetry
   ↓
Batch Processor
   ↓
Resource Processor
   ↓
Exporter
```

---

# 20. Exporters

Exporters send telemetry to a backend.

Examples include destinations such as:

```text
Prometheus-compatible systems
Jaeger-compatible systems
OTLP endpoints
Elasticsearch
Other observability platforms
```

Conceptually:

```text
Collector
    ↓
Exporter
    ↓
Backend
```

---

# 21. OpenTelemetry Protocol

OTLP stands for:

**OpenTelemetry Protocol**

It is the primary protocol used by OpenTelemetry components to exchange telemetry.

Conceptually:

```text
Application
     ↓
OTel SDK
     ↓
OTLP
     ↓
Collector
```

OTLP supports OpenTelemetry telemetry signals.

---

# 22. OTLP Transport

OTLP can use supported transports such as:

```text
OTLP/gRPC
OTLP/HTTP
```

The specific endpoint and port depend on the deployment configuration.

The important concept is:

```text
OTel SDK
     ↓
OTLP
     ↓
OTel Collector
```

---

# 23. Resource Attributes

A resource describes the entity producing telemetry.

Useful attributes include:

```text
service.name
service.version
deployment.environment
host.name
cloud.region
k8s.namespace.name
k8s.pod.name
```

Example:

```json
{
  "service.name": "payment",
  "service.version": "v1.5.2",
  "deployment.environment": "production"
}
```

These attributes help identify the telemetry source.

---

# 24. Service Name

`service.name` is especially important.

Example:

```text
service.name = payment
```

Another service:

```text
service.name = orders
```

This allows observability systems to distinguish telemetry from different services.

Without service identity, centralized telemetry becomes difficult to analyze.

---

# 25. Semantic Conventions

OpenTelemetry defines standardized naming conventions for telemetry attributes.

Examples include:

```text
service.name
service.version
http.request.method
server.address
server.port
db.system
```

Standard naming makes telemetry more consistent across applications.

---

# 26. Why Semantic Conventions Matter

Without standardization:

```text
serviceName
service
application_name
app
```

Different applications may use different field names.

With a common convention:

```text
service.name
```

dashboards and queries become easier to reuse.

---

# 27. Context Propagation

Distributed tracing requires context propagation.

Example:

```text
Frontend
   ↓
Orders
   ↓
Payment
```

The trace context must travel with the request.

Conceptually:

```text
Request
  +
Trace Context
      ↓
Orders
      ↓
Payment
```

This allows spans generated by different services to belong to the same trace.

---

# 28. Trace Context

A trace context typically contains information used to correlate spans.

Important identifiers include:

```text
Trace ID
Span ID
Trace Flags
Trace State
```

Example:

```text
Trace ID:
abc123

Span ID:
def456
```

---

# 29. Distributed Trace Example

A user request:

```text
User
 ↓
Frontend
 ↓
Orders
 ↓
Payment
 ↓
Database
```

OpenTelemetry can represent it as:

```text
Trace: abc123

├── Frontend Span
│
├── Orders Span
│
├── Payment Span
│
└── Database Span
```

The trace ID connects all spans.

---

# 30. Span

A span represents a single operation.

Example:

```text
Span
│
├── Operation: HTTP POST /payments
├── Service: payment
├── Start Time
├── End Time
├── Attributes
└── Status
```

Multiple spans form a trace.

---

# 31. Trace Relationships

Example:

```text
Parent Span
    │
    ├── Child Span
    │
    ├── Child Span
    │
    └── Child Span
```

For a microservices request:

```text
Frontend Span
     │
     └── Orders Span
           │
           └── Payment Span
                 │
                 └── Database Span
```

---

# 32. Span Attributes

Spans can contain attributes.

Example:

```text
http.request.method = POST
server.address = payment
http.response.status_code = 500
```

Attributes provide additional context about the operation.

---

# 33. Span Events

A span can contain events.

Example:

```text
Payment Span
    │
    ├── Start
    ├── Database timeout event
    ├── Error event
    └── End
```

Events can provide detailed information without creating a separate span for every event.

---

# 34. Span Status

A span can indicate operation status.

Conceptually:

```text
OK
ERROR
UNSET
```

Example:

```text
Payment Span
    ↓
Database timeout
    ↓
Status = ERROR
```

This helps identify failed operations.

---

# 35. Error Recording

When an operation fails, instrumentation can record:

```text
Exception
Error message
Stack trace
Span status
```

Example:

```text
Payment
 ↓
Database timeout
 ↓
Record exception
 ↓
Span status = ERROR
```

---

# 36. Sampling

Large systems can generate enormous numbers of traces.

Example:

```text
1,000,000 requests/minute
```

Recording every trace may be expensive.

Sampling allows only a subset to be retained.

Conceptually:

```text
1,000,000 requests
        ↓
     Sampling
        ↓
100,000 traces
```

The actual sampling percentage depends on workload and requirements.

---

# 37. Head Sampling

Sampling decision is made early.

```text
Request
   ↓
Sampling Decision
   ↓
Keep / Drop
```

This reduces telemetry volume early.

---

# 38. Tail Sampling

Tail sampling makes the decision after more information about the trace is available.

Conceptually:

```text
Complete Trace
      ↓
Analyze
      ↓
Keep important traces
```

For example:

```text
Keep
ERROR traces
Slow traces
Important business traces
```

This can be useful for distributed systems.

---

# 39. Batching

Telemetry can be batched before export.

Instead of:

```text
Event
 ↓
Export
```

use:

```text
Events
 ↓
Batch
 ↓
Export
```

Benefits:

```text
Reduced network overhead
Improved throughput
Better backend efficiency
```

The Collector commonly provides batching processors.

---

# 40. Memory Limiting

Telemetry pipelines must protect themselves from excessive memory consumption.

Conceptually:

```text
Telemetry
    ↓
Memory Limiter
    ↓
Batch
    ↓
Exporter
```

This is especially important in Kubernetes.

---

# 41. OpenTelemetry Collector Modes

The Collector can be deployed in different architectural roles.

### Agent

Runs close to the application or host.

```text
Application
    ↓
Local Collector
```

### Gateway

Centralized Collector layer.

```text
Applications
     ↓
Collectors
     ↓
Gateway Collector
     ↓
Backends
```

---

# 42. Agent + Gateway Architecture

A production architecture can use both:

```text
Application
    ↓
Node/Agent Collector
    ↓
Gateway Collector
    ↓
Backend
```

For Kubernetes:

```text
Pods
 ↓
DaemonSet Collector
 ↓
Gateway Collector
 ↓
Observability Backend
```

This provides more centralized control over processing and export.

---

# 43. OpenTelemetry in Kubernetes

A typical architecture:

```text
                         EKS
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       Node-01         Node-02         Node-03
          │               │               │
        OTel            OTel            OTel
       Collector        Collector       Collector
          │               │               │
          └───────────────┼───────────────┘
                          ↓
                  OTel Gateway
                          ↓
              ┌───────────┼───────────┐
              ↓           ↓           ↓
          Prometheus    ELK         Jaeger
```

---

# 44. OpenTelemetry With Prometheus

OpenTelemetry can participate in a metrics pipeline.

Example:

```text
Application
     ↓
OTel SDK
     ↓
OTel Collector
     ↓
Prometheus-compatible endpoint
     ↓
Prometheus
     ↓
Grafana
```

The exact architecture depends on the chosen Collector exporter and Prometheus integration.

---

# 45. OpenTelemetry With Elasticsearch

Logs can be routed toward Elasticsearch-compatible logging architectures.

Conceptually:

```text
Application
     ↓
OTel
     ↓
Collector
     ↓
Elasticsearch
     ↓
Kibana
```

Another architecture may use:

```text
Application
     ↓
OTel
     ↓
Collector
     ↓
Logstash
     ↓
Elasticsearch
     ↓
Kibana
```

The choice depends on processing and compatibility requirements.

---

# 46. OpenTelemetry With Jaeger

For tracing:

```text
Application
     ↓
OTel SDK
     ↓
OTel Collector
     ↓
Jaeger
```

The Collector acts as a routing and processing layer between applications and the tracing backend.

---

# 47. OpenTelemetry as a Vendor-Neutral Layer

One major benefit is separation:

```text
Application
      ↓
OpenTelemetry
      ↓
Observability Backend
```

The application does not have to be tightly coupled to one backend.

For example:

```text
                    OpenTelemetry
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
      Prometheus        ELK             Jaeger
```

---

# 48. OpenTelemetry vs Backend

OpenTelemetry:

```text
Generate
Collect
Process
Export
```

Backend:

```text
Store
Query
Visualize
Alert
```

For example:

```text
OpenTelemetry
      ↓
Collector
      ↓
Prometheus / Elasticsearch / Jaeger
      ↓
Grafana / Kibana / Jaeger UI
```

---

# 49. OpenTelemetry vs Prometheus

They are not exactly competing products.

Prometheus is primarily a metrics monitoring system.

OpenTelemetry is a broader observability framework.

Comparison:

```text
OpenTelemetry
 ├── Metrics
 ├── Logs
 └── Traces

Prometheus
 └── Metrics-focused monitoring
```

OpenTelemetry can feed metrics into Prometheus-compatible architectures.

---

# 50. OpenTelemetry vs ELK

ELK is primarily a logging platform:

```text
Logstash
Elasticsearch
Kibana
```

OpenTelemetry supports multiple signals:

```text
Metrics
Logs
Traces
```

A possible architecture is:

```text
Application
    ↓
OpenTelemetry
    ↓
Collector
    ├── Metrics → Prometheus
    ├── Logs → Elasticsearch
    └── Traces → Jaeger
```

---

# 51. OpenTelemetry vs Jaeger

Jaeger is primarily a distributed tracing backend and visualization system.

OpenTelemetry provides:

```text
Instrumentation
Telemetry APIs
SDKs
Collection
Processing
Export
```

A common architecture:

```text
Application
    ↓
OpenTelemetry
    ↓
Collector
    ↓
Jaeger
```

---

# 52. Observability Signal Correlation

The real value comes from connecting signals.

Example:

```text
Metric
 ↓
Payment latency increased
 ↓
Log
 ↓
Database timeout
 ↓
Trace
 ↓
Payment → Database
```

This provides:

```text
Detection
+
Investigation
+
Root-cause analysis
```

---

# 53. Production Example

Suppose the payment service suddenly becomes slow.

Prometheus:

```text
payment_latency ↑
```

Grafana:

```text
Latency = 2.5 seconds
```

Kibana:

```text
Database connection timeout
```

Trace:

```text
Payment
  ↓
Database
  ↓
2.3 second delay
```

OpenTelemetry can help correlate these telemetry signals.

---

# 54. OpenTelemetry Instrumentation Lifecycle

A typical application lifecycle:

```text
Application Starts
       ↓
OTel SDK Initializes
       ↓
Instrumentation Initialized
       ↓
Telemetry Generated
       ↓
Telemetry Processed
       ↓
Telemetry Exported
```

---

# 55. Application Startup

The application configures:

```text
Service Name
Environment
Exporter
Endpoint
Sampling
Resource Attributes
```

Example conceptual configuration:

```text
service.name = payment
environment = production
exporter = OTLP
endpoint = otel-collector
```

---

# 56. Telemetry Export

Application:

```text
Payment Service
     ↓
OTel SDK
     ↓
OTLP
     ↓
Collector
```

Collector:

```text
Receiver
   ↓
Processor
   ↓
Exporter
```

Backend:

```text
Exporter
   ↓
Prometheus / Elasticsearch / Jaeger
```

---

# 57. OpenTelemetry Configuration Concepts

Common configuration areas include:

```text
Resource attributes
Receivers
Processors
Exporters
Service pipelines
Sampling
Endpoints
TLS
Authentication
```

These are typically configured through environment variables, SDK configuration, Collector configuration, or deployment manifests depending on the component.

---

# 58. Collector Configuration Structure

A Collector configuration conceptually contains:

```yaml
receivers:
  otlp:
    ...

processors:
  batch:
    ...

exporters:
  ...

service:
  pipelines:
    traces:
      receivers:
      processors:
      exporters:
```

The exact configuration depends on the Collector distribution and installed components.

---

# 59. Telemetry Pipelines

Separate pipelines can be created for different signals:

```text
Metrics Pipeline
    ↓
Receiver
    ↓
Processors
    ↓
Exporter

Logs Pipeline
    ↓
Receiver
    ↓
Processors
    ↓
Exporter

Traces Pipeline
    ↓
Receiver
    ↓
Processors
    ↓
Exporter
```

This provides independent control.

---

# 60. Multi-Signal Collector

A single Collector can handle multiple signals:

```text
                    OTel Collector
                         │
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
    Metrics             Logs             Traces
       │                 │                 │
       ↓                 ↓                 ↓
  Prometheus            ELK              Jaeger
```

This is one reason the Collector is useful in centralized observability architectures.

---

# 61. Collector Gateway

A centralized gateway:

```text
Applications
     ↓
Node Collectors
     ↓
Gateway Collector
     ↓
Backends
```

The gateway can centralize:

```text
Filtering
Sampling
Batching
Routing
Security
Export
```

---

# 62. Telemetry Routing

Different signals can go to different backends.

Example:

```text
                    Collector
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
       Metrics         Logs         Traces
          ↓             ↓             ↓
     Prometheus        ELK          Jaeger
```

This provides a vendor-neutral collection layer while allowing specialized backends.

---

# 63. Production Security

Secure telemetry communication using:

```text
TLS
Authentication
Authorization
Network restrictions
Secrets management
```

Example:

```text
Application
    ↓ TLS
Collector
    ↓ TLS
Backend
```

Do not expose telemetry endpoints publicly without appropriate protection.

---

# 64. Collector Resource Management

The Collector is itself a production workload.

Configure:

```text
CPU requests
CPU limits
Memory requests
Memory limits
Replicas
Health checks
```

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

These are example values and should be tuned based on telemetry volume.

---

# 65. Collector High Availability

Avoid a single gateway Collector:

```text
Applications
     ↓
Collector-01
     ↓
Backend
```

Prefer:

```text
Applications
     ↓
      ┌──────────────┐
      ↓              ↓
Collector-01    Collector-02
      │              │
      └──────┬───────┘
             ↓
          Backend
```

Multiple replicas reduce the impact of a Collector failure.

---

# 66. Kubernetes Collector Deployment Models

Common patterns include:

```text
DaemonSet
Deployment
Gateway Deployment
```

DaemonSet:

```text
One Collector per node
```

Deployment:

```text
Multiple centralized Collector replicas
```

Gateway:

```text
Centralized processing/export layer
```

---

# 67. Agent + Gateway

A production Kubernetes architecture:

```text
Node-01
 └── OTel Agent
       │
Node-02
 └── OTel Agent
       │
Node-03
 └── OTel Agent
       │
       └──────────┐
                  ↓
            OTel Gateway
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
   Prometheus    ELK      Jaeger
```

This separates local collection from centralized processing.

---

# 68. Collector Failure

If an agent fails:

```text
Application
    ↓
Collector X
```

telemetry may be lost or delayed depending on buffering and retry configuration.

Monitor Collector health.

If a gateway fails:

```text
Agent
  ↓
Gateway X
```

multiple gateway replicas and appropriate retry/buffering mechanisms can improve resilience.

---

# 69. Observability of the Collector

Monitor:

```text
Received telemetry
Exported telemetry
Dropped telemetry
Export failures
Queue size
Processing latency
CPU
Memory
```

Example:

```text
Received = 100,000
Exported = 99,950
Dropped = 50
```

A non-zero drop rate should be investigated when unexpected.

---

# 70. Collector Backpressure

When a backend is slow:

```text
Backend
   ↓
Slow
   ↓
Collector
   ↓
Queue grows
```

Use:

```text
Batching
Queues
Retry
Memory limits
Backpressure controls
```

The goal is to prevent the Collector from consuming unbounded memory.

---

# 71. OpenTelemetry and Microservices

For your microservices architecture:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

Each service can be instrumented:

```text
Service
   ↓
OTel SDK
   ↓
Collector
```

The Collector centralizes telemetry processing.

---

# 72. Service-to-Service Tracing

Example:

```text
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

Trace:

```text
Trace ID = abc123

Frontend
 └── Orders
      └── Payment
           └── Inventory
                └── Database
```

This allows engineers to see the complete request path.

---

# 73. Database Instrumentation

Applications can generate database spans.

Example:

```text
Payment Service
      ↓
DB Query
      ↓
PostgreSQL
```

Trace:

```text
Payment
   └── SELECT database operation
```

The span can include useful attributes while avoiding sensitive query parameters.

---

# 74. HTTP Instrumentation

HTTP instrumentation can capture:

```text
HTTP method
Route
Status code
Duration
Server
Client
```

Example:

```text
POST /payments
Status = 500
Duration = 1.8s
```

This is useful for identifying slow or failing endpoints.

---

# 75. Messaging Instrumentation

Microservices often use asynchronous messaging.

Example:

```text
Orders
   ↓
RabbitMQ
   ↓
Notification
```

OpenTelemetry can provide telemetry around messaging operations where supported by the instrumentation.

Trace relationship:

```text
Orders
   ↓
Message Publish
   ↓
RabbitMQ
   ↓
Message Consume
   ↓
Notification
```

---

# 76. External API Calls

A service may call an external API:

```text
Payment
   ↓
Payment Gateway
```

Instrumentation can provide visibility into:

```text
Request duration
Status
Errors
Dependency behavior
```

This helps distinguish application problems from external dependency failures.

---

# 77. Error Analysis

Suppose:

```text
Payment API = 500
```

Trace:

```text
Payment
  ↓
External Gateway
  ↓
Timeout
```

Log:

```text
Gateway timeout
```

Metric:

```text
payment_error_rate ↑
```

Combined telemetry points toward the external dependency.

---

# 78. OpenTelemetry and Kubernetes Metadata

Kubernetes telemetry should identify:

```text
Cluster
Namespace
Pod
Container
Node
Deployment
Service
```

Example:

```text
service.name = payment
k8s.namespace.name = production
k8s.pod.name = payment-7d8f
```

This makes traces and metrics easier to filter.

---

# 79. Production Naming Standards

Use consistent service names.

Good:

```text
payment
orders
inventory
```

Avoid inconsistent names such as:

```text
payment-service-prod
PaymentService
payment_app
paymentsvc
```

unless the organization's naming standard requires them.

---

# 80. Environment Attributes

Include environment information:

```text
deployment.environment = production
```

Then telemetry can be filtered:

```text
deployment.environment = production
```

This prevents mixing telemetry from different environments.

---

# 81. Version Attributes

Include service version:

```text
service.version = v1.5.2
```

This is extremely useful during deployment investigations.

Example:

```text
v1.5.1 → healthy
v1.5.2 → latency increased
```

---

# 82. Deployment Correlation

A production incident:

```text
Release v1.5.2
      ↓
Latency ↑
      ↓
Error Rate ↑
      ↓
Trace failures
      ↓
Database timeout
```

OpenTelemetry can provide the telemetry required to correlate the application version with the failure.

---

# 83. OpenTelemetry and GitOps

A GitOps architecture:

```text
GitHub
   ↓
Configuration
   ↓
GitHub Actions
   ↓
Validation / Security
   ↓
ArgoCD
   ↓
EKS
   ↓
OpenTelemetry
```

Collector configurations should be version controlled.

---

# 84. Terraform + OpenTelemetry

Terraform can provision infrastructure:

```text
VPC
EKS
IAM
Security Groups
ALB
S3
```

Then:

```text
Terraform
    ↓
Infrastructure
```

and:

```text
ArgoCD
    ↓
OpenTelemetry Collector
```

This keeps infrastructure and Kubernetes application deployment responsibilities separate.

---

# 85. Production Architecture Example

```text
                              USERS
                                │
                                ↓
                             AWS ALB
                                │
                                ↓
                         EKS Applications
                                │
                 ┌──────────────┼──────────────┐
                 ↓              ↓              ↓
              Service A      Service B      Service C
                 │              │              │
                 └──────────────┼──────────────┘
                                ↓
                         OTel SDK / Agent
                                ↓
                       OTel Collector
                                ↓
                     OTel Gateway Cluster
                                │
               ┌────────────────┼────────────────┐
               ↓                ↓                ↓
           Prometheus          ELK             Jaeger
               ↓                ↓                ↓
            Grafana          Kibana          Trace UI
```

---

# 86. Production Collector Pipeline

Conceptually:

```text
                  OTLP
                   ↓
                Receiver
                   ↓
             Memory Limiter
                   ↓
                 Batch
                   ↓
               Resource
                   ↓
               Sampling
                   ↓
                Exporter
                   ↓
                Backend
```

The actual processors should be selected based on requirements.

---

# 87. Collector Configuration Principles

A production Collector configuration should consider:

```text
Receiver compatibility
Processor ordering
Memory limits
Batch sizes
Retry behavior
Export timeouts
TLS
Authentication
Sampling
Routing
```

Incorrect configuration can cause:

```text
Dropped telemetry
High memory usage
High CPU
Backend overload
```

---

# 88. Telemetry Filtering

Not every telemetry event needs to be exported.

Example:

```text
All telemetry
      ↓
Filter
      ↓
Remove unnecessary data
      ↓
Export
```

Filtering can reduce:

```text
Network traffic
Storage cost
Backend load
```

---

# 89. Sampling Strategy

A production tracing strategy may use:

```text
Keep 100% errors
Keep slow traces
Sample normal requests
```

Conceptually:

```text
Normal requests
     ↓
Sample

Errors
     ↓
Keep

Very slow requests
     ↓
Keep
```

The actual policy depends on business and troubleshooting requirements.

---

# 90. Cost Management

OpenTelemetry can generate significant telemetry.

Control cost through:

```text
Sampling
Filtering
Aggregation
Batching
Retention
Appropriate log levels
```

Do not collect every possible signal at maximum detail without a capacity plan.

---

# 91. Security Considerations

Telemetry may contain:

```text
User information
URLs
Headers
Database information
Error messages
Service metadata
```

Avoid sending sensitive information unnecessarily.

Security controls:

```text
TLS
Authentication
Authorization
Network isolation
Data filtering
Secret management
```

---

# 92. Production Anti-Patterns

Avoid:

```text
Single Collector
Single Elasticsearch node
Public Elasticsearch
Hardcoded credentials
Unlimited DEBUG logs
No sampling
No resource limits
No monitoring
No retry/buffering
No retention policy
No backup strategy
```

---

# 93. Good Production Architecture

Prefer:

```text
Multiple Collectors
        +
HA Gateway
        +
Secure transport
        +
Resource limits
        +
Batching
        +
Sampling
        +
Monitoring
        +
Reliable backends
```

---

# 94. Troubleshooting OpenTelemetry

When telemetry is missing:

```text
Application
   ↓
Check SDK
   ↓
Check instrumentation
   ↓
Check exporter
   ↓
Check Collector receiver
   ↓
Check Collector processors
   ↓
Check Collector exporter
   ↓
Check backend
```

Do not immediately assume the backend is broken.

---

# 95. Missing Traces

Troubleshooting flow:

```text
No Trace
   ↓
Is instrumentation enabled?
   ↓
Is SDK configured?
   ↓
Is exporter configured?
   ↓
Is Collector reachable?
   ↓
Is OTLP receiver running?
   ↓
Is pipeline configured?
   ↓
Is exporter working?
   ↓
Is backend receiving?
```

---

# 96. Missing Metrics

Check:

```text
Application
 ↓
Metrics instrumentation
 ↓
OTel SDK
 ↓
Collector
 ↓
Metrics pipeline
 ↓
Exporter
 ↓
Prometheus
```

Then check:

```text
Prometheus target / ingestion
```

according to the chosen architecture.

---

# 97. Missing Logs

Check:

```text
Application
 ↓
Log generation
 ↓
OTel logging integration
 ↓
Collector
 ↓
Logs pipeline
 ↓
Exporter
 ↓
Logging backend
```

---

# 98. High Collector CPU

Possible causes:

```text
High telemetry volume
Complex processors
Expensive transformations
High sampling workload
Too many pipelines
```

Actions:

```text
Measure
Filter unnecessary data
Batch
Scale Collector
Optimize processors
```

---

# 99. High Collector Memory

Possible causes:

```text
Large queues
High telemetry volume
Large batches
Insufficient memory limits
Slow backend
```

Actions:

```text
Enable memory limiting
Tune queues
Reduce telemetry
Scale Collector
Fix backend bottleneck
```

---

# 100. Collector Export Failures

Possible causes:

```text
Backend unavailable
Network failure
TLS problem
Authentication failure
Incorrect endpoint
Backend overloaded
```

Troubleshooting:

```text
Collector logs
 ↓
Export metrics
 ↓
Network
 ↓
TLS
 ↓
Authentication
 ↓
Backend health
```

---

# 101. Production Readiness Checklist

## Instrumentation

```text
[ ] Services instrumented
[ ] Automatic instrumentation configured where appropriate
[ ] Manual instrumentation for important business operations
[ ] Service names standardized
[ ] Versions included
[ ] Environment included
```

## Collector

```text
[ ] Receivers configured
[ ] Processors configured
[ ] Exporters configured
[ ] Memory limiting
[ ] Batching
[ ] Retry
[ ] Resource limits
[ ] Health checks
```

## Security

```text
[ ] TLS
[ ] Authentication
[ ] Authorization
[ ] Network restrictions
[ ] Secrets management
[ ] Sensitive data filtering
```

---

# 102. Reliability Checklist

```text
[ ] Multiple Collector replicas
[ ] Gateway HA
[ ] Backend HA
[ ] Buffering
[ ] Retry
[ ] Failure testing
[ ] Capacity planning
[ ] Disaster recovery
[ ] Backup strategy
```

---

# 103. Observability Checklist

```text
[ ] Metrics
[ ] Logs
[ ] Traces
[ ] Correlation IDs
[ ] Trace IDs
[ ] Service metadata
[ ] Kubernetes metadata
[ ] Dashboards
[ ] Alerts
```

---

# 104. OpenTelemetry Mental Model

Remember OpenTelemetry as:

```text
                APPLICATION
                     │
                     ↓
             OTel API / SDK
                     │
                     ↓
                TELEMETRY
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Metrics      Logs      Traces
          │          │          │
          └──────────┼──────────┘
                     ↓
             OTel Collector
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Metrics      Logs      Traces
          ↓          ↓          ↓
    Prometheus       ELK       Jaeger
          ↓          ↓          ↓
       Grafana     Kibana    Trace UI
```

The key idea is:

**OpenTelemetry provides a standardized, vendor-neutral framework for instrumenting applications and collecting metrics, logs, and traces. The OpenTelemetry SDK generates telemetry, the Collector receives and processes it, and exporters send it to observability backends such as Prometheus-compatible systems, Elasticsearch, or Jaeger. In a production Kubernetes environment, OpenTelemetry should be designed with proper instrumentation, resource attributes, context propagation, batching, sampling, security, high availability, resource limits, monitoring, and failure handling.**
