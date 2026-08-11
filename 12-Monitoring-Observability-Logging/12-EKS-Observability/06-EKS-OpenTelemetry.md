# EKS OpenTelemetry

## 1. Overview

OpenTelemetry provides a vendor-neutral framework for collecting observability telemetry from applications running on Amazon EKS.

It supports:

```text
Metrics
Logs
Traces
```

For EKS, OpenTelemetry is particularly useful for distributed microservices because it can collect telemetry from workloads running across multiple Pods and Nodes.

A typical architecture is:

```text
                         EKS
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
   Application         Kubernetes         Nodes
      Pods                │                 │
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                OpenTelemetry Collector
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       Metrics           Logs           Traces
          │               │               │
          ↓               ↓               ↓
     Prometheus           ELK           Jaeger
          │               │               │
          ↓               ↓               ↓
       Grafana          Kibana         Jaeger UI
```

---

# 2. Why OpenTelemetry in EKS?

A production EKS environment can contain:

```text
Multiple microservices
Multiple namespaces
Multiple Pods
Multiple Nodes
Multiple dependencies
```

Without a standardized telemetry layer, every application may use different monitoring libraries and exporters.

OpenTelemetry provides a common approach for:

```text
Instrumentation
Telemetry collection
Context propagation
Processing
Export
```

---

# 3. OpenTelemetry Components

The major components are:

```text
OpenTelemetry API
OpenTelemetry SDK
Instrumentation
OpenTelemetry Collector
Exporters
```

Conceptually:

```text
Application
    ↓
Instrumentation
    ↓
OpenTelemetry SDK
    ↓
OpenTelemetry Collector
    ↓
Backend
```

---

# 4. OpenTelemetry Signals

OpenTelemetry supports three major observability signals:

```text
Metrics
Logs
Traces
```

Example:

```text
Application
   │
   ├── Metrics → Prometheus
   ├── Logs    → ELK
   └── Traces  → Jaeger
```

This allows a common instrumentation and collection layer while using specialized backends.

---

# 5. OpenTelemetry Architecture in EKS

A production architecture can look like:

```text
                    EKS Cluster
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
     Service A        Service B        Service C
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                OpenTelemetry SDK
                         ↓
                OpenTelemetry Collector
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Metrics          Logs           Traces
          ↓              ↓              ↓
    Prometheus           ELK           Jaeger
          ↓              ↓              ↓
      Grafana          Kibana        Jaeger UI
```

---

# 6. OpenTelemetry API

The OpenTelemetry API provides interfaces used by applications and instrumentation.

It allows application code to work with concepts such as:

```text
Tracer
Meter
Logger
Context
```

The API separates application instrumentation from the implementation details of telemetry processing.

---

# 7. OpenTelemetry SDK

The SDK provides the implementation behind the API.

It handles tasks such as:

```text
Creating telemetry
Processing telemetry
Sampling
Batching
Exporting
```

Conceptually:

```text
Application
    ↓
OpenTelemetry API
    ↓
OpenTelemetry SDK
    ↓
Telemetry
```

---

# 8. Instrumentation

Instrumentation generates telemetry from application activity.

Examples include:

```text
HTTP requests
Database calls
Messaging
Framework operations
External API calls
```

Instrumentation can be:

```text
Automatic
Manual
```

---

# 9. Automatic Instrumentation

Automatic instrumentation can capture common operations without requiring developers to manually create spans for every operation.

For example:

```text
HTTP request
     ↓
Automatic instrumentation
     ↓
Span created
```

This is useful for quickly introducing tracing into existing applications.

---

# 10. Manual Instrumentation

Manual instrumentation provides application-specific visibility.

Example:

```text
Process Order
     ↓
Create custom span
     ↓
Validate Order
     ↓
Process Payment
     ↓
End span
```

This allows important business operations to appear in traces.

---

# 11. OpenTelemetry Collector

The Collector is a vendor-neutral telemetry processing component.

It can:

```text
Receive
Process
Filter
Batch
Sample
Enrich
Export
```

Architecture:

```text
Applications
     ↓
OpenTelemetry Collector
     ↓
Backend
```

The Collector reduces direct coupling between applications and observability backends.

---

# 12. Collector Architecture

The Collector pipeline is built around:

```text
Receivers
    ↓
Processors
    ↓
Exporters
```

Example:

```text
OTLP Receiver
      ↓
Batch Processor
      ↓
Jaeger Exporter
```

---

# 13. Receivers

Receivers accept telemetry from applications and other sources.

Common examples include:

```text
OTLP
Jaeger
Zipkin
Prometheus
```

For OpenTelemetry-native applications, OTLP is commonly used.

---

# 14. Processors

Processors modify or control telemetry.

Examples:

```text
Batch
Memory limiting
Filtering
Sampling
Resource enrichment
Attribute modification
```

Example:

```text
Incoming spans
      ↓
Filter
      ↓
Batch
      ↓
Export
```

---

# 15. Exporters

Exporters send telemetry to destinations.

Examples include:

```text
Prometheus-compatible systems
Elasticsearch
Jaeger
Other observability backends
```

Architecture:

```text
Collector
    ↓
Exporter
    ↓
Backend
```

---

# 16. OpenTelemetry in Kubernetes

OpenTelemetry can be deployed in different ways depending on the workload.

Common Collector deployment patterns include:

```text
DaemonSet
Deployment
Sidecar
```

Each pattern solves different operational requirements.

---

# 17. DaemonSet Collector

A DaemonSet runs a Collector on each eligible Node.

Example:

```text
Node-1 → Collector
Node-2 → Collector
Node-3 → Collector
```

When a new Node is added:

```text
New Node
   ↓
DaemonSet
   ↓
Collector
```

Advantages:

```text
Node-local collection
Automatic scaling with Nodes
Distributed collection
```

---

# 18. Deployment Collector

A Deployment provides centralized Collector instances.

Example:

```text
Applications
     │
 ┌───┼───┐
 ↓   ↓   ↓
 C1  C2  C3
  \   |  /
   \  | /
    Backend
```

Advantages:

```text
Independent scaling
Centralized processing
Simpler centralized management
```

---

# 19. Sidecar Collector

A sidecar places a Collector alongside an application.

Example:

```text
Pod
│
├── Application
└── OTel Collector
```

This provides close coupling between the application and Collector.

However, using a sidecar for every application Pod can increase:

```text
CPU
Memory
Operational complexity
```

Therefore the deployment model should be selected based on requirements.

---

# 20. EKS Collector Architecture

A scalable architecture may use:

```text
Application Pods
       ↓
Node-local Collectors
       ↓
Gateway Collectors
       ↓
Backends
```

Conceptually:

```text
Pod
 ↓
DaemonSet Collector
 ↓
Gateway Collector
 ├──→ Prometheus
 ├──→ Elasticsearch
 └──→ Jaeger
```

This separates local collection from centralized processing.

---

# 21. OTLP

OTLP stands for OpenTelemetry Protocol.

It is designed for transporting OpenTelemetry telemetry.

It can transport:

```text
Metrics
Logs
Traces
```

Conceptually:

```text
Application
    ↓
OTLP
    ↓
Collector
```

---

# 22. Trace Context Propagation

Distributed tracing requires trace context propagation.

Example:

```text
Service A
   │
   │ Trace Context
   ↓
Service B
   │
   │ Trace Context
   ↓
Service C
```

This allows all operations to remain part of the same trace.

---

# 23. Trace ID

A Trace ID identifies the complete distributed request.

Example:

```text
trace_id=abc123
```

The same Trace ID can appear across:

```text
Frontend
Orders
Payment
Inventory
Database
```

---

# 24. Span ID

A Span ID identifies an individual operation.

Example:

```text
Trace ID: abc123

Span 001 → Orders
Span 002 → Payment
Span 003 → Database
```

The Trace ID connects the spans.

---

# 25. Parent and Child Spans

Example:

```text
Orders
│
├── Payment
│    └── Database
│
└── Inventory
     └── Database
```

This hierarchy shows the request dependency structure.

---

# 26. OpenTelemetry Metrics

Applications can expose metrics through OpenTelemetry.

Examples:

```text
Request rate
Request duration
Error count
Active requests
Database connections
Queue size
```

Architecture:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
Collector
     ↓
Metrics Backend
```

---

# 27. OpenTelemetry Logs

OpenTelemetry can also provide a standardized approach to logs.

Conceptually:

```text
Application
     ↓
OpenTelemetry
     ↓
Collector
     ↓
Log Backend
```

In an existing EKS environment, logs can be forwarded to an ELK-based backend.

---

# 28. OpenTelemetry Traces

Tracing is one of the most common OpenTelemetry use cases.

Example:

```text
Client
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

OpenTelemetry can connect these operations into one distributed trace.

---

# 29. OpenTelemetry and Jaeger

A common architecture is:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
Collector
     ↓
Jaeger
     ↓
Jaeger UI
```

Jaeger provides:

```text
Trace storage
Trace search
Trace visualization
Span analysis
```

OpenTelemetry provides the instrumentation and collection framework.

---

# 30. OpenTelemetry and Prometheus

Metrics architecture:

```text
Application
     ↓
OpenTelemetry
     ↓
Collector
     ↓
Prometheus-compatible backend
     ↓
Grafana
```

This provides a standardized instrumentation layer while retaining Prometheus/Grafana for metrics operations.

---

# 31. OpenTelemetry and ELK

Logs can flow through:

```text
Application
     ↓
OpenTelemetry
     ↓
Collector
     ↓
Elasticsearch
     ↓
Kibana
```

In some architectures, a separate log collector such as Fluent Bit or another agent may feed the ELK pipeline.

The architecture should be selected based on log volume and operational requirements.

---

# 32. OpenTelemetry and Kubernetes Metadata

Telemetry can be enriched with:

```text
Cluster
Namespace
Pod
Container
Node
Deployment
Service
Environment
```

Example:

```text
service.name=payment
deployment.environment=production
k8s.namespace.name=payments
k8s.pod.name=payment-abc
```

This makes telemetry much easier to filter and analyze.

---

# 33. Resource Attributes

Resource attributes describe the entity producing telemetry.

Examples:

```text
service.name
service.version
deployment.environment
host.name
```

In Kubernetes:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.node.name
```

These attributes provide important context.

---

# 34. Span Attributes

Span attributes describe the operation.

Examples:

```text
http.request.method
http.response.status_code
url.path
server.address
db.system
db.operation
```

Useful attributes help engineers understand what happened without storing unnecessary payload data.

---

# 35. Avoid Sensitive Data

Telemetry can accidentally expose:

```text
Passwords
Access tokens
API keys
Personal information
Payment information
Secrets
```

Avoid putting sensitive information into:

```text
Span attributes
Log fields
Metric labels
```

Use filtering and application-level controls where appropriate.

---

# 36. Sampling

High-volume applications can generate enormous trace volumes.

Example:

```text
100,000 requests
       ↓
Sampling
       ↓
10,000 traces
```

Sampling helps control:

```text
Storage
Network
Collector CPU
Backend cost
```

---

# 37. Error-Aware Sampling

Production systems can prioritize failed requests.

Conceptually:

```text
Successful requests
→ Lower sampling

Failed requests
→ Higher sampling
```

This preserves more useful diagnostic information.

---

# 38. Latency-Aware Sampling

Slow requests can also receive higher priority.

Example:

```text
Normal request
→ Lower sampling

Slow request
→ Higher sampling
```

This helps investigate:

```text
Latency spikes
Performance regressions
Slow dependencies
```

---

# 39. Batch Processing

Collectors can batch telemetry before exporting.

Example:

```text
Span
Span
Span
Span
 ↓
Batch
 ↓
Exporter
```

Batching can reduce:

```text
Network overhead
Backend request overhead
Processing overhead
```

---

# 40. Memory Limiting

A Collector must be protected from excessive memory consumption.

Monitor:

```text
Memory usage
Queue size
Incoming telemetry
Export throughput
```

Resource limits should be configured appropriately for the expected telemetry volume.

---

# 41. Collector Backpressure

Suppose:

```text
Incoming = 20,000 spans/sec
Export = 12,000 spans/sec
```

A backlog can develop.

Possible causes:

```text
Backend slow
Network issue
Collector resource constraints
Excessive telemetry volume
```

Investigate the entire pipeline.

---

# 42. Collector High Availability

Production architecture should avoid relying on one Collector.

Example:

```text
Applications
     │
 ┌───┼───┐
 ↓   ↓   ↓
 C1  C2  C3
  \   |  /
   \  | /
    Backend
```

Multiple Collectors provide:

```text
Resilience
Load distribution
Independent scaling
```

---

# 43. Collector Monitoring

Monitor:

```text
CPU
Memory
Received telemetry
Exported telemetry
Dropped telemetry
Export errors
Queue size
Export latency
```

The observability pipeline itself must be observable.

---

# 44. Dropped Telemetry

Dropped telemetry can occur because of:

```text
Collector overload
Backend failure
Network problems
Queue exhaustion
Sampling
Configuration errors
```

Monitor dropped:

```text
Spans
Metrics
Logs
```

to identify observability gaps.

---

# 45. OpenTelemetry in Microservices

Example:

```text
                         Client
                           │
                           ↓
                        Frontend
                           │
                           ↓
                         Orders
                      /           \
                     ↓             ↓
                 Payment       Inventory
                     │             │
                     └──────┬──────┘
                            ↓
                         Database
```

Telemetry:

```text
All services
     ↓
OpenTelemetry
     ↓
Collector
     ↓
Metrics + Logs + Traces
```

---

# 46. EKS Service-to-Service Tracing

Example:

```text
Orders Pod
    │
    │ HTTP
    ↓
Payment Service
    │
    │ HTTP
    ↓
Payment Pod
```

OpenTelemetry propagates context across the service boundary.

The resulting trace may look like:

```text
Orders
 └── Payment
      └── Database
```

---

# 47. EKS Async Tracing

Microservices may communicate through queues.

Example:

```text
Orders
  ↓
RabbitMQ
  ↓
Notification Worker
```

Trace context can be propagated through supported messaging instrumentation.

This allows asynchronous operations to remain correlated.

---

# 48. OpenTelemetry and Kubernetes Service Discovery

Kubernetes provides dynamic service discovery.

Pods can:

```text
Start
Stop
Scale
Move
Restart
```

Telemetry systems should therefore use dynamic discovery and Kubernetes metadata rather than relying on static Pod addresses.

---

# 49. OpenTelemetry Operator

The OpenTelemetry Operator can help manage OpenTelemetry resources in Kubernetes.

It can simplify:

```text
Collector deployment
Configuration management
Instrumentation management
Kubernetes integration
```

The Operator is an optional component and should be introduced when its operational benefits justify the added complexity.

---

# 50. Instrumentation Strategy

A practical rollout can be:

```text
Phase 1
Instrument critical services

Phase 2
Add database and external dependency visibility

Phase 3
Add Kubernetes metadata

Phase 4
Correlate logs and traces

Phase 5
Introduce advanced sampling

Phase 6
Expand coverage
```

Start with the most important workloads rather than instrumenting everything immediately.

---

# 51. Production Telemetry Architecture

```text
                         EKS
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
   Application A     Application B     Application C
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                  OpenTelemetry SDK
                          ↓
                 Local Collector
                          ↓
                 Gateway Collector
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       Metrics           Logs           Traces
          ↓               ↓               ↓
    Prometheus            ELK           Jaeger
          ↓               ↓               ↓
       Grafana          Kibana         Jaeger UI
```

---

# 52. EKS OpenTelemetry With Prometheus

```text
Application
    ↓
OpenTelemetry SDK
    ↓
OTel Collector
    ↓
Metrics Export
    ↓
Prometheus
    ↓
Grafana
```

Use this architecture when you want a common instrumentation layer while retaining Prometheus/Grafana as the metrics platform.

---

# 53. EKS OpenTelemetry With ELK

```text
Application
    ↓
OpenTelemetry
    ↓
Collector
    ↓
Log Processing
    ↓
Elasticsearch
    ↓
Kibana
```

This provides centralized application logging.

---

# 54. EKS OpenTelemetry With Jaeger

```text
Application
    ↓
OpenTelemetry SDK
    ↓
Collector
    ↓
Trace Export
    ↓
Jaeger
    ↓
Jaeger UI
```

This provides distributed tracing across microservices.

---

# 55. Complete EKS Observability

```text
                              EKS
                               │
              ┌────────────────┼────────────────┐
              ↓                ↓                ↓
           Metrics            Logs            Traces
              │                │                │
              └────────────────┼────────────────┘
                               ↓
                     OpenTelemetry Layer
                               │
                        Collector
                               │
          ┌────────────────────┼────────────────────┐
          ↓                    ↓                    ↓
     Prometheus           Elasticsearch           Jaeger
          ↓                    ↓                    ↓
      Grafana               Kibana              Jaeger UI
```

---

# 56. Metrics + Logs + Traces Correlation

Suppose:

```text
Grafana:
Payment p95 latency ↑
```

Then:

```text
Jaeger:
Payment → Database span = 1.7 seconds
```

Then:

```text
Kibana:
Database connection timeout
```

The investigation becomes:

```text
Metric
  ↓
Trace
  ↓
Log
  ↓
Root Cause
```

---

# 57. OpenTelemetry During Deployment

Before deployment:

```text
v1
p95 = 250 ms
```

After deployment:

```text
v2
p95 = 800 ms
```

Use traces to compare:

```text
v1 → Database span = 100 ms
v2 → Database span = 600 ms
```

Use logs to investigate the underlying error or behavior.

---

# 58. OpenTelemetry During 503 Errors

Suppose:

```text
Users
  ↓
503
```

Investigation:

```text
Grafana
 ↓
5xx rate ↑
 ↓
Jaeger
 ↓
Failed backend span
 ↓
Kibana
 ↓
Application error
```

This creates a complete incident path.

---

# 59. OpenTelemetry During High Latency

Problem:

```text
Request = 2 seconds
```

Trace:

```text
Frontend       100 ms
Orders         200 ms
Payment       1,400 ms
Database      1,200 ms
```

Focus:

```text
Payment
   ↓
Database
```

instead of investigating every service equally.

---

# 60. OpenTelemetry During Node Failure

Example:

```text
Node-2
   ↓
NotReady
```

Workloads move to other Nodes.

OpenTelemetry continues to capture application telemetry because telemetry is associated with application execution rather than a fixed Node.

Kubernetes metadata allows engineers to identify where workloads were running.

---

# 61. OpenTelemetry Security

Secure the telemetry pipeline using:

```text
TLS
Authentication
Authorization
RBAC
Network policies
Secret management
Sensitive-data filtering
```

Telemetry backends should not be unnecessarily exposed to the public internet.

---

# 62. OpenTelemetry Cost Management

Major cost factors:

```text
Telemetry volume
Trace sampling
Log volume
Storage
Retention
Collector resources
Backend resources
```

Optimize with:

```text
Sampling
Filtering
Batching
Retention
Useful attributes
Appropriate log levels
```

---

# 63. High Cardinality

Avoid unnecessary high-cardinality telemetry.

For metrics, be particularly careful with:

```text
user_id
request_id
session_id
```

as labels.

For traces and logs, avoid unnecessarily storing:

```text
Large payloads
Entire responses
Sensitive data
Unbounded metadata
```

---

# 64. OpenTelemetry Troubleshooting Workflow

If telemetry is missing:

```text
1. Check application instrumentation.
2. Verify SDK configuration.
3. Verify telemetry generation.
4. Check context propagation.
5. Check Collector receiver.
6. Check Collector processors.
7. Check Collector exporter.
8. Check network connectivity.
9. Check backend ingestion.
10. Check backend storage.
11. Check UI/query filters.
```

---

# 65. Missing Traces

If traces are missing:

```text
Application
    ↓
Are spans generated?
    ↓
Collector
    ↓
Are spans received?
    ↓
Exporter
    ↓
Are spans exported?
    ↓
Jaeger
    ↓
Are traces stored?
```

Troubleshoot sequentially.

---

# 66. Missing Metrics

If metrics are missing:

```text
Application
    ↓
Metric generated?
    ↓
Collector
    ↓
Metric received?
    ↓
Exporter
    ↓
Prometheus
    ↓
Metric query
```

Check each stage.

---

# 67. Missing Logs

If logs are missing:

```text
Application
    ↓
Log generated?
    ↓
Collector
    ↓
Log received?
    ↓
Exporter
    ↓
Elasticsearch
    ↓
Kibana
```

This is the same pipeline troubleshooting principle.

---

# 68. Collector Configuration

A simplified conceptual configuration is:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  batch:

exporters:
  otlp:
    endpoint: <backend>

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp]
```

The exact exporter and endpoint depend on the selected backend.

---

# 69. Collector Pipeline Separation

Different signals can have different pipelines.

```text
                Collector
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
    Traces         Logs        Metrics
       │            │            │
       ↓            ↓            ↓
    Jaeger         ELK       Prometheus
```

This provides independent processing and export paths.

---

# 70. Resource Management

Collectors should have Kubernetes resource requests and limits.

Monitor:

```text
CPU
Memory
Network
Queue
Telemetry rate
```

Example concept:

```yaml
resources:
  requests:
    cpu: ...
    memory: ...
  limits:
    cpu: ...
    memory: ...
```

The values should be based on actual workload measurements.

---

# 71. Collector Scaling

If telemetry volume increases:

```text
Traffic ↑
   ↓
Telemetry ↑
   ↓
Collector load ↑
```

Scale Collector instances appropriately.

Example:

```text
Collector-1
Collector-2
Collector-3
```

Autoscaling can be considered when workload patterns justify it.

---

# 72. OpenTelemetry in Production

Production considerations include:

```text
High availability
Sampling
Resource limits
Telemetry security
Data retention
Backend capacity
Collector scaling
Monitoring
Alerting
Cost management
```

Do not treat telemetry as an afterthought.

---

# 73. OpenTelemetry Best Practices

```text
1. Standardize instrumentation.
2. Use OpenTelemetry where appropriate.
3. Instrument critical services first.
4. Propagate trace context.
5. Add useful resource attributes.
6. Avoid sensitive data.
7. Use appropriate sampling.
8. Batch telemetry.
9. Monitor Collector health.
10. Monitor dropped telemetry.
11. Scale Collectors according to volume.
12. Secure telemetry transport.
13. Correlate logs with traces.
14. Correlate metrics with traces.
15. Define retention policies.
16. Control telemetry cardinality.
17. Monitor backend capacity.
18. Test observability during deployments.
19. Design for failure.
20. Monitor the monitoring pipeline.
```

---

# 74. Interview Question

### Why would you use OpenTelemetry in EKS?

**Answer:**

I would use OpenTelemetry to provide a standardized, vendor-neutral instrumentation and telemetry collection layer across microservices running in EKS. It can collect metrics, logs, and traces and export them to backends such as Prometheus, Elasticsearch, and Jaeger. This reduces application coupling to individual observability vendors and simplifies distributed observability.

---

# 75. Interview Question

### What is the role of the OpenTelemetry Collector?

**Answer:**

The Collector receives telemetry from applications, processes it, and exports it to observability backends. It can perform batching, filtering, sampling, enrichment, and routing. This creates a separate telemetry processing layer between applications and backend systems.

---

# 76. Interview Question

### What are the different Collector deployment models in Kubernetes?

**Answer:**

The main deployment models are DaemonSet, Deployment, and Sidecar. A DaemonSet is useful for Node-local collection, a Deployment is useful for centralized processing and gateway-style architectures, and a Sidecar provides a Collector alongside an individual application Pod. The correct model depends on telemetry volume and operational requirements.

---

# 77. Interview Question

### How does OpenTelemetry support distributed tracing?

**Answer:**

OpenTelemetry instruments applications and creates spans. Trace context is propagated across service boundaries so downstream operations remain part of the same trace. The spans are sent to a Collector, which processes and exports them to a tracing backend such as Jaeger.

---

# 78. Interview Question

### How would you troubleshoot missing telemetry?

**Answer:**

I would troubleshoot the telemetry pipeline from source to backend. First I would verify that the application generates telemetry and that instrumentation is configured correctly. Then I would check Collector receivers, processors, queues, and exporters. Finally I would verify network connectivity, backend ingestion, storage, and query or UI configuration.

---

# 79. Interview Question

### How would you handle high telemetry volume?

**Answer:**

I would first identify which services and signals generate the most telemetry. Then I would use appropriate sampling for traces, filtering for unnecessary telemetry, batching for efficient export, and retention policies for storage control. I would also scale the Collector and backend based on measured throughput and monitor dropped telemetry.

---

# 80. Interview Question

### How do you correlate OpenTelemetry traces with logs?

**Answer:**

I include Trace ID and, when useful, Span ID in application logs. When I find an error in Kibana, I can search using the Trace ID and locate the corresponding trace in Jaeger. The trace identifies the failing or slow operation, while the log provides the detailed application error.

---

# 81. Interview Question

### How would you design OpenTelemetry for high availability?

**Answer:**

I would avoid a single Collector instance and use multiple Collectors. Depending on the architecture, I could use DaemonSet Collectors for local collection and multiple gateway Collectors for centralized processing. I would monitor Collector queues, export failures, dropped telemetry, resource usage, and backend health.

---

# 82. EKS OpenTelemetry Checklist

```text
APPLICATION
[ ] OpenTelemetry API
[ ] OpenTelemetry SDK
[ ] Automatic instrumentation
[ ] Manual instrumentation
[ ] Trace context propagation
[ ] Resource attributes
[ ] Span attributes
[ ] Trace ID in logs
[ ] Sensitive-data protection

COLLECTOR
[ ] Receivers
[ ] Processors
[ ] Exporters
[ ] DaemonSet / Deployment strategy
[ ] Resource requests
[ ] Resource limits
[ ] Batching
[ ] Sampling
[ ] Filtering
[ ] Queue monitoring
[ ] Dropped telemetry
[ ] High availability

METRICS
[ ] Prometheus
[ ] Grafana
[ ] Application metrics
[ ] Kubernetes metrics

LOGS
[ ] ELK
[ ] Elasticsearch
[ ] Kibana
[ ] Structured logs
[ ] Kubernetes metadata

TRACES
[ ] Jaeger
[ ] Trace IDs
[ ] Span IDs
[ ] Context propagation
[ ] Sampling
[ ] Trace retention

SECURITY
[ ] TLS
[ ] Authentication
[ ] Authorization
[ ] RBAC
[ ] Network policies
[ ] Secret protection
[ ] Sensitive-data filtering

OPERATIONS
[ ] Capacity
[ ] Scaling
[ ] Storage
[ ] Retention
[ ] Cost
[ ] Monitoring
[ ] Alerting
```

---

# 83. Final Mental Model

```text
                              EKS
                               │
             ┌─────────────────┼─────────────────┐
             ↓                 ↓                 ↓
        Microservices       Kubernetes          Nodes
             │                 │                 │
             └─────────────────┼─────────────────┘
                               ↓
                     OpenTelemetry SDK
                               │
                               ↓
                    Trace Context / Telemetry
                               │
                               ↓
                    OpenTelemetry Collector
                               │
             ┌─────────────────┼─────────────────┐
             ↓                 ↓                 ↓
          Metrics             Logs              Traces
             │                 │                  │
             ↓                 ↓                  ↓
       Prometheus              ELK               Jaeger
             │                 │                  │
             ↓                 ↓                  ↓
          Grafana            Kibana           Jaeger UI
             │                 │                  │
             └─────────────────┼──────────────────┘
                               ↓
                     Complete Observability
```

**Key principle:** OpenTelemetry provides a standardized observability layer for EKS applications. Applications generate telemetry through OpenTelemetry instrumentation, Collectors receive and process that telemetry, and specialized backends provide storage and visualization. In a production architecture, **Prometheus/Grafana handles metrics, ELK handles logs, and Jaeger handles traces**, while OpenTelemetry provides the common instrumentation and telemetry pipeline connecting the application layer to those systems.
