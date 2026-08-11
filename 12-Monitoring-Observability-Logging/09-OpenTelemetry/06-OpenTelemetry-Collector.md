# OpenTelemetry Collector

## 1. Overview

The OpenTelemetry Collector is a vendor-neutral service that receives, processes, and exports telemetry.

It sits between applications and observability backends.

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTLP
     ↓
OpenTelemetry Collector
     ↓
Processing
     ↓
Backend
```

The Collector supports the three major telemetry signals:

```text
Metrics
Logs
Traces
```

A production architecture can use the Collector to centralize:

```text
Collection
Processing
Filtering
Batching
Sampling
Routing
Retry
Export
Security
```

---

# 2. Why the Collector Exists

Without a Collector:

```text
Application
     ├──→ Prometheus
     ├──→ Elasticsearch
     └──→ Jaeger
```

Every application must understand multiple backend systems.

With a Collector:

```text
Application
     ↓
OTel Collector
     ├──→ Prometheus
     ├──→ Elasticsearch
     └──→ Jaeger
```

The application only needs to understand the OpenTelemetry protocol.

This creates a separation between:

```text
Application instrumentation
```

and:

```text
Observability backend
```

---

# 3. Collector Responsibilities

The Collector can perform:

```text
Receive telemetry
Validate telemetry
Enrich telemetry
Transform telemetry
Filter telemetry
Batch telemetry
Sample traces
Queue telemetry
Retry exports
Route telemetry
Export telemetry
Expose health metrics
```

Architecture:

```text
                 OpenTelemetry Collector
                         │
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
    Receive            Process           Export
       │                 │                 │
    Receiver          Processor         Exporter
```

---

# 4. Collector Architecture

The core Collector architecture is:

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

Extensions provide additional functionality:

```text
Extensions
    │
    ├── Health Check
    ├── Authentication
    └── Diagnostics
```

The complete architecture:

```text
             ┌─────────────────────────────┐
             │     OpenTelemetry Collector │
             │                             │
Telemetry →  │ Receivers                   │
             │      ↓                      │
             │ Processors                  │
             │      ↓                      │
Backend  ←   │ Exporters                   │
             │                             │
             │ Extensions                  │
             └─────────────────────────────┘
```

---

# 5. Collector Components

The main components are:

```text
Receivers
Processors
Exporters
Connectors
Extensions
Service Pipelines
```

A simple mental model:

```text
Receiver
   ↓
Processor
   ↓
Exporter
```

A more advanced architecture:

```text
Receiver
   ↓
Processor
   ↓
Connector
   ↓
Another Pipeline
   ↓
Exporter
```

---

# 6. Receivers

Receivers accept telemetry into the Collector.

Examples:

```text
OTLP
Prometheus
Jaeger
Zipkin
Kafka
Filelog
Host Metrics
```

The receiver determines how telemetry enters the Collector.

Example:

```text
Application
     ↓
OTLP/gRPC
     ↓
OTLP Receiver
```

---

# 7. OTLP Receiver

The OTLP receiver is one of the most commonly used receivers.

It can support:

```text
OTLP/gRPC
OTLP/HTTP
```

Example:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:
```

Architecture:

```text
Application
     ↓
OTLP
     ↓
Collector
     ↓
OTLP Receiver
```

---

# 8. OTLP gRPC

OTLP over gRPC commonly uses:

```text
4317
```

Conceptually:

```text
Application
     ↓
gRPC
     ↓
Collector :4317
```

The application exporter configuration must match the Collector endpoint.

---

# 9. OTLP HTTP

OTLP over HTTP commonly uses:

```text
4318
```

Architecture:

```text
Application
     ↓
HTTP
     ↓
Collector :4318
```

The choice between gRPC and HTTP depends on:

```text
Application support
Network architecture
Proxy requirements
Operational requirements
```

---

# 10. Prometheus Receiver

The Prometheus receiver allows the Collector to scrape Prometheus-compatible targets.

Conceptually:

```text
Application / Exporter
       ↑
       │ Scrape
       │
OTel Collector
       ↓
Metrics Pipeline
```

Example concept:

```yaml
receivers:
  prometheus:
    config:
      scrape_configs:
        - job_name: node-exporter
          static_configs:
            - targets:
                - node-exporter:9100
```

The exact configuration depends on the environment.

---

# 11. Host Metrics Receiver

The host metrics receiver can collect infrastructure metrics.

Examples:

```text
CPU
Memory
Disk
Filesystem
Network
Load
```

Architecture:

```text
Kubernetes Node
      ↓
Host Metrics Receiver
      ↓
Collector
      ↓
Metrics Backend
```

This is useful when the Collector is deployed with appropriate host access.

---

# 12. Filelog Receiver

The filelog receiver can collect logs from files.

Conceptually:

```text
Application Log File
       ↓
Filelog Receiver
       ↓
Collector
       ↓
Logs Pipeline
```

This can be useful for applications that write logs to files rather than directly exporting OpenTelemetry logs.

---

# 13. Receivers Are Signal-Specific

A receiver can be used in one or more pipelines depending on its supported signal types.

For example:

```text
OTLP Receiver
   ├── Metrics
   ├── Logs
   └── Traces
```

Another receiver may support only metrics or logs.

Always verify the receiver's supported signals before designing the pipeline.

---

# 14. Processors

Processors modify, filter, enrich, or control telemetry after reception.

Common processors include:

```text
batch
memory_limiter
resource
attributes
filter
transform
probabilistic_sampler
tail_sampling
```

Architecture:

```text
Receiver
   ↓
Processor
   ↓
Exporter
```

---

# 15. Batch Processor

The batch processor groups telemetry before export.

```text
Span 1 ┐
Span 2 ├──→ Batch
Span 3 ┘
          ↓
       Exporter
```

Benefits:

```text
Reduced network overhead
Improved throughput
More efficient backend ingestion
```

Example:

```yaml
processors:
  batch:
    timeout: 5s
    send_batch_size: 1024
```

The values are examples and should be tuned.

---

# 16. Memory Limiter

The memory limiter helps prevent excessive memory consumption.

Example:

```yaml
processors:
  memory_limiter:
    limit_percentage: 80
    spike_limit_percentage: 15
```

Architecture:

```text
Telemetry
    ↓
Memory Limiter
    ↓
Batch
    ↓
Exporter
```

The Collector should also have appropriate Kubernetes memory requests and limits.

---

# 17. Resource Processor

The resource processor modifies resource attributes.

Example:

```yaml
processors:
  resource:
    attributes:
      - key: deployment.environment
        value: production
        action: upsert
```

This can standardize telemetry across services.

---

# 18. Attributes Processor

The attributes processor operates on telemetry attributes.

Possible operations include:

```text
Insert
Update
Delete
Hash
```

Example use cases:

```text
Normalize service metadata
Remove sensitive attributes
Add environment information
Modify inconsistent attributes
```

---

# 19. Filter Processor

The filter processor can remove telemetry that is not required.

Architecture:

```text
All Telemetry
      ↓
Filter
      ↓
Required Telemetry
      ↓
Exporter
```

Filtering can reduce:

```text
Network traffic
Backend ingestion
Storage
Cost
```

---

# 20. Transform Processor

The transform processor can modify telemetry according to defined transformation rules.

Potential uses:

```text
Rename attributes
Set attributes
Normalize data
Modify telemetry fields
```

Transformations should be kept understandable and documented because complex processing can increase Collector CPU usage.

---

# 21. Sampling Processors

Sampling is especially important for traces.

Architecture:

```text
All Traces
    ↓
Sampling
    ↓
Selected Traces
    ↓
Backend
```

Sampling reduces:

```text
Storage
Network
CPU
Backend cost
```

---

# 22. Tail Sampling

Tail sampling waits for enough information about a trace before deciding whether to retain it.

Architecture:

```text
Trace Spans
    ↓
Collector
    ↓
Trace assembled
    ↓
Tail Sampling
    ↓
Keep / Drop
```

Example policy:

```text
Error → Keep
Slow → Keep
Normal → Sample
```

Tail sampling requires careful resource planning.

---

# 23. Exporters

Exporters send telemetry out of the Collector.

Examples:

```text
OTLP
Prometheus-compatible destinations
Elasticsearch
Kafka
Debug
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

# 24. OTLP Exporter

The OTLP exporter sends telemetry to another OTLP-compatible endpoint.

Example:

```yaml
exporters:
  otlp:
    endpoint: backend:4317
```

Architecture:

```text
Collector
    ↓
OTLP Exporter
    ↓
Another Collector / Backend
```

TLS and authentication should be configured when required.

---

# 25. Debug Exporter

The debug exporter is useful for development and troubleshooting.

Conceptually:

```yaml
exporters:
  debug:
```

Architecture:

```text
Application
    ↓
Collector
    ↓
Debug Exporter
    ↓
Collector Logs
```

It allows engineers to verify whether telemetry is reaching the Collector pipeline.

---

# 26. Backend-Specific Exporters

Some environments require backend-specific exporters.

For example:

```text
Collector
   ↓
Elasticsearch Export
   ↓
Elasticsearch
```

or:

```text
Collector
   ↓
OTLP
   ↓
Backend
```

The available exporters depend on the selected Collector distribution.

---

# 27. Connectors

Connectors can connect one Collector pipeline to another pipeline.

Conceptually:

```text
Pipeline A
    ↓
Connector
    ↓
Pipeline B
```

This is useful for advanced routing and signal transformation architectures.

For example:

```text
Metrics Pipeline
      ↓
Connector
      ↓
Logs / Metrics / Traces Pipeline
```

The exact behavior depends on the connector.

---

# 28. Extensions

Extensions provide capabilities that are not part of the normal telemetry pipeline.

Common examples:

```text
health_check
pprof
zpages
authentication extensions
```

Example:

```yaml
extensions:
  health_check:
```

Enable it through the service configuration:

```yaml
service:
  extensions:
    - health_check
```

---

# 29. Health Check Extension

The health check extension allows monitoring systems and Kubernetes to determine whether the Collector is healthy.

Architecture:

```text
Kubernetes Probe
      ↓
Health Check
      ↓
Collector
```

This is important for production deployments.

---

# 30. pprof Extension

The pprof extension can help investigate Collector performance.

It can provide profiling information such as:

```text
CPU
Memory
Goroutines
```

It should not be exposed publicly without appropriate protection.

---

# 31. zPages Extension

zPages provides runtime diagnostic information about Collector processing.

It can be useful during:

```text
Troubleshooting
Development
Performance analysis
```

It should be protected and restricted in production.

---

# 32. Service Pipelines

The `service` section connects components together.

Example:

```yaml
service:
  pipelines:
    traces:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - otlp
```

This means:

```text
OTLP Receiver
      ↓
Memory Limiter
      ↓
Batch
      ↓
OTLP Exporter
```

---

# 33. Metrics Pipeline

A metrics pipeline:

```yaml
service:
  pipelines:
    metrics:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - otlp
```

Architecture:

```text
Metrics
   ↓
OTLP Receiver
   ↓
Memory Limiter
   ↓
Batch
   ↓
Exporter
   ↓
Metrics Backend
```

---

# 34. Logs Pipeline

A logs pipeline:

```yaml
service:
  pipelines:
    logs:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - otlp
```

Architecture:

```text
Logs
   ↓
OTLP Receiver
   ↓
Memory Limiter
   ↓
Batch
   ↓
Exporter
   ↓
Logging Backend
```

---

# 35. Traces Pipeline

A traces pipeline:

```yaml
service:
  pipelines:
    traces:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - otlp
```

Architecture:

```text
Traces
   ↓
OTLP Receiver
   ↓
Memory Limiter
   ↓
Batch
   ↓
Exporter
   ↓
Tracing Backend
```

---

# 36. Complete Multi-Signal Configuration

A Collector can have all three pipelines:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  memory_limiter:
  batch:

exporters:
  otlp:
    endpoint: backend:4317

service:
  pipelines:

    metrics:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - otlp

    logs:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - otlp

    traces:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - otlp
```

This is a conceptual example.

---

# 37. Collector Pipeline Isolation

Each signal can have different processing requirements.

For example:

```text
Metrics
   ↓
Resource
   ↓
Batch
   ↓
Prometheus

Logs
   ↓
Filter
   ↓
Resource
   ↓
Batch
   ↓
ELK

Traces
   ↓
Tail Sampling
   ↓
Batch
   ↓
Jaeger
```

This allows signal-specific optimization.

---

# 38. Multiple Receivers

A Collector can accept telemetry from different sources.

```text
OTLP Applications
       ↓
OTLP Receiver
       │
Prometheus Targets
       ↓
Prometheus Receiver
       │
Log Files
       ↓
Filelog Receiver
```

These can then enter different pipelines.

---

# 39. Multiple Exporters

A pipeline can export to more than one destination.

Conceptually:

```text
Traces
   ↓
Collector
   ├──→ Backend A
   └──→ Backend B
```

This can be useful during:

```text
Migration
Testing
Dual-backend operation
Backend comparison
```

But exporting everything to multiple backends increases cost and operational complexity.

---

# 40. Pipeline Routing

Telemetry can be routed based on attributes.

For example:

```text
service.name = payment
       ↓
Payment Backend

service.name = orders
       ↓
Orders Backend
```

Routing can also use:

```text
Environment
Region
Tenant
Signal
Service
```

---

# 41. Collector Agent

A Collector can run close to applications.

In Kubernetes:

```text
Node-01
├── Application Pods
└── OTel Collector

Node-02
├── Application Pods
└── OTel Collector
```

A DaemonSet is commonly used for node-level Collector deployment.

---

# 42. Collector Gateway

A gateway is a centralized Collector.

```text
Applications
      ↓
Agent Collectors
      ↓
Gateway
      ↓
Backends
```

The gateway can provide:

```text
Centralized processing
Sampling
Filtering
Routing
Authentication
Export
```

---

# 43. Agent + Gateway Architecture

A scalable production architecture:

```text
                         EKS
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
    Node-01            Node-02            Node-03
       │                  │                  │
    OTel Agent         OTel Agent         OTel Agent
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                   OTel Gateway
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
         Prometheus       ELK         Jaeger
```

---

# 44. Why Agent + Gateway

The agent handles:

```text
Local collection
Node-level telemetry
Local enrichment
```

The gateway handles:

```text
Central processing
Sampling
Routing
Backend export
```

This separates responsibilities.

---

# 45. Collector Deployment Modes

Common deployment modes:

```text
DaemonSet
Deployment
Sidecar
```

### DaemonSet

```text
One Collector per node
```

### Deployment

```text
Centralized Collector replicas
```

### Sidecar

```text
Collector alongside application
```

Choose the deployment model based on the telemetry source and architecture.

---

# 46. DaemonSet Architecture

```text
Node-01
  ├── Application
  └── OTel Collector

Node-02
  ├── Application
  └── OTel Collector

Node-03
  ├── Application
  └── OTel Collector
```

Advantages:

```text
Local collection
Node-level visibility
Distributed collection
```

---

# 47. Deployment Architecture

```text
OTel Gateway Deployment
│
├── Gateway-01
├── Gateway-02
└── Gateway-03
```

Applications connect through:

```text
Kubernetes Service
        ↓
Gateway Pods
```

Advantages:

```text
Centralized processing
Easy horizontal scaling
Centralized configuration
```

---

# 48. Sidecar Architecture

A sidecar Collector runs beside an application:

```text
Pod
│
├── Application
└── OTel Collector
```

This provides isolation but can increase resource consumption significantly.

For large Kubernetes environments, DaemonSet and gateway architectures are often more resource-efficient.

---

# 49. Collector Networking

A production flow:

```text
Application
     ↓
Private Network
     ↓
OTel Collector
     ↓
Private Network
     ↓
Backend
```

Avoid unnecessary public exposure.

---

# 50. Kubernetes Service

The gateway should generally be accessed through a stable Service.

```text
Application
     ↓
otel-gateway.observability.svc
     ↓
Gateway Pods
```

Do not configure applications with individual Collector Pod IP addresses.

---

# 51. Collector DNS

Example:

```text
otel-gateway.observability.svc.cluster.local
```

Application configuration:

```text
OTEL_EXPORTER_OTLP_ENDPOINT
        ↓
otel-gateway.observability.svc.cluster.local:4317
```

Kubernetes handles Pod replacement behind the Service.

---

# 52. Collector TLS

Production communication may use TLS:

```text
Application
     ↓ TLS
Collector
     ↓ TLS
Gateway
     ↓ TLS
Backend
```

Certificates should be managed securely.

Possible mechanisms include:

```text
Kubernetes Secrets
Certificate management
External secret systems
Cloud certificate infrastructure
```

---

# 53. Collector Authentication

Authentication may be required at:

```text
Application → Collector
Collector → Gateway
Gateway → Backend
```

The exact mechanism depends on the deployment and backend.

Use:

```text
TLS
Authentication
Authorization
Network policies
```

as appropriate for the trust boundary.

---

# 54. Collector RBAC

Kubernetes Collector workloads should use least privilege.

Avoid:

```text
Cluster-wide administrator permissions
```

unless genuinely required.

Use:

```text
ServiceAccount
Role
RoleBinding
ClusterRole
ClusterRoleBinding
```

only according to the Collector's actual Kubernetes metadata or resource discovery requirements.

---

# 55. Kubernetes Metadata Enrichment

A Collector may need Kubernetes API access to enrich telemetry.

For example:

```text
Pod
 ↓
Kubernetes metadata
 ↓
Collector
 ↓
Telemetry
```

This can provide:

```text
Namespace
Pod
Container
Node
Deployment
Cluster
```

The required permissions should be reviewed carefully.

---

# 56. Collector Resource Requests

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

These are examples only.

Production values should be determined using:

```text
Telemetry rate
CPU usage
Memory usage
Processor complexity
Export latency
```

---

# 57. Collector Scaling

Scale horizontally when:

```text
CPU remains high
Memory remains high
Queue grows
Export latency increases
Dropped telemetry increases
```

Architecture:

```text
                Collector Service
                       ↓
           ┌───────────┼───────────┐
           ↓           ↓           ↓
       Gateway-01  Gateway-02  Gateway-03
```

---

# 58. High Availability

Production gateway:

```text
Gateway-01
Gateway-02
Gateway-03
```

Distribute replicas across nodes and availability zones where practical.

Use:

```text
Pod anti-affinity
Topology spread constraints
PodDisruptionBudget
```

to improve resilience.

---

# 59. Collector Health Monitoring

Monitor:

```text
Telemetry received
Telemetry exported
Telemetry dropped
Exporter failures
Queue size
CPU
Memory
Pipeline errors
```

Architecture:

```text
Collector
    ↓
Collector Metrics
    ↓
Prometheus
    ↓
Grafana
```

---

# 60. Collector Alerts

Useful alerts:

```text
Collector unavailable
Exporter failure
Dropped telemetry
High CPU
High memory
Queue growth
Backend unavailable
No telemetry received
```

Thresholds should be based on the environment's normal baseline.

---

# 61. Collector Self-Observability

The observability system itself must be observable.

```text
Applications
    ↓
Collector
    ↓
Backend
```

But also:

```text
Collector
    ↓
Collector Metrics
    ↓
Prometheus
    ↓
Grafana
```

This prevents the observability layer from becoming a blind spot.

---

# 62. Collector Failure Scenario

Suppose:

```text
Gateway-01 → Down
Gateway-02 → Healthy
Gateway-03 → Healthy
```

Applications should continue sending telemetry through the Kubernetes Service to the healthy replicas.

Business traffic should remain independent from telemetry availability.

---

# 63. Backend Failure

Suppose:

```text
Collector
     ↓
Backend
     X
```

The Collector can use configured retry and queue mechanisms where supported.

Architecture:

```text
Collector
    ↓
Queue
    ↓
Retry
    ↓
Backend
```

The queue must be bounded and monitored.

---

# 64. High Telemetry Volume

Suppose the platform generates:

```text
High request volume
High log volume
High trace volume
```

Architecture:

```text
Applications
      ↓
Agents
      ↓
Regional Gateways
      ↓
Central / Regional Backends
```

Scale the Collector layer horizontally rather than relying on one Collector instance.

---

# 65. Multi-Cluster Architecture

For multiple EKS clusters:

```text
prod-eks
   ↓
OTel Agent
   ↓
Gateway
   ↓
Central Backend

staging-eks
   ↓
OTel Agent
   ↓
Gateway
   ↓
Central Backend
```

The telemetry should include cluster and environment identity.

---

# 66. Multi-Region Architecture

For multiple AWS regions:

```text
Region-1
   ↓
OTel Gateway
   ↓
Regional Backend

Region-2
   ↓
OTel Gateway
   ↓
Regional Backend
```

Or:

```text
Region-1 ─┐
Region-2 ─┼→ Central Gateway → Central Backend
Region-3 ─┘
```

The choice depends on:

```text
Latency
Data residency
Network cost
Resilience
Backend architecture
```

---

# 67. OpenTelemetry Collector With ELK

For the ELK stack:

```text
Application
     ↓
OTel SDK
     ↓
OTel Collector
     ↓
Logstash
     ↓
Elasticsearch
     ↓
Kibana
```

The Collector can handle:

```text
Collection
Filtering
Enrichment
Batching
Routing
```

Logstash can continue to handle existing log processing requirements.

---

# 68. OpenTelemetry Collector With Prometheus

A metrics architecture:

```text
Application
     ↓
OTel SDK
     ↓
OTel Collector
     ↓
Prometheus-compatible metrics path
     ↓
Prometheus
     ↓
Grafana
```

The exact architecture depends on whether Prometheus scrapes an endpoint or the Collector exports metrics through another supported mechanism.

---

# 69. OpenTelemetry Collector With Jaeger

Tracing:

```text
Application
     ↓
OTel SDK
     ↓
OTLP
     ↓
Collector
     ↓
Jaeger
```

This keeps the application independent from the Jaeger-specific backend interface.

---

# 70. Full Observability Architecture

```text
                           Users
                             │
                             ↓
                            ALB
                             │
                             ↓
                       EKS Services
                             │
                 ┌───────────┼───────────┐
                 ↓           ↓           ↓
              Metrics       Logs       Traces
                 │           │           │
                 └───────────┼───────────┘
                             ↓
                         OTel Agent
                             ↓
                         OTel Gateway
                             │
                 ┌───────────┼───────────┐
                 ↓           ↓           ↓
             Prometheus      ELK        Jaeger
                 ↓           ↓           ↓
              Grafana      Kibana      Trace UI
```

---

# 71. Collector and GitOps

For your EKS environment, Collector configuration can be managed through GitOps:

```text
GitHub
   ↓
Collector Configuration
   ↓
Pull Request
   ↓
Validation
   ↓
Security Checks
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
```

This provides:

```text
Version control
Audit trail
Review
Rollback
Drift detection
Repeatability
```

---

# 72. Terraform and Collector

Terraform can provision the infrastructure:

```text
VPC
Subnets
EKS
IAM
Security Groups
```

Then GitOps can deploy the observability components:

```text
Terraform
   ↓
EKS
   ↓
ArgoCD
   ↓
OTel Collector
```

This cleanly separates infrastructure provisioning from application and observability deployment.

---

# 73. Collector Configuration Repository

A possible structure:

```text
observability/
│
├── opentelemetry/
│   ├── base/
│   │   ├── collector.yaml
│   │   └── service.yaml
│   │
│   └── overlays/
│       ├── dev/
│       ├── staging/
│       └── production/
│
├── prometheus/
├── grafana/
├── elasticsearch/
├── logstash/
└── kibana/
```

Environment-specific settings can be managed through overlays or Helm values.

---

# 74. Collector Deployment Flow

A production change:

```text
Developer
   ↓
Modify Collector configuration
   ↓
Pull Request
   ↓
Review
   ↓
CI validation
   ↓
Merge
   ↓
ArgoCD
   ↓
Collector rollout
   ↓
Health check
   ↓
Telemetry validation
```

---

# 75. Rolling Update

A gateway Deployment can be updated gradually.

```text
Old:
Gateway-01
Gateway-02
Gateway-03

       ↓

New:
Gateway-01
Gateway-02
Gateway-03
```

Kubernetes should maintain sufficient available replicas during the rollout.

---

# 76. Collector Rollback

If a new configuration causes telemetry failures:

```text
New Configuration
       ↓
Problem
```

Rollback:

```text
Git Revert
     ↓
Previous Configuration
     ↓
ArgoCD
     ↓
Collector
```

This is safer than manually editing production Pods.

---

# 77. Collector Configuration Validation

Before deployment:

```text
YAML validation
Collector configuration validation
Helm validation
Kubernetes validation
Security scanning
```

After deployment:

```text
Pod health
Service endpoints
Collector logs
Collector metrics
Backend telemetry
```

---

# 78. Troubleshooting Flow

When telemetry is missing:

```text
Application
    ↓
SDK
    ↓
Network
    ↓
Collector Service
    ↓
Receiver
    ↓
Processor
    ↓
Exporter
    ↓
Backend
```

Check each layer in order.

---

# 79. Troubleshooting Collector Pod

Check:

```bash
kubectl get pods -n observability
```

Then:

```bash
kubectl logs <collector-pod> -n observability
```

Check events:

```bash
kubectl get events -n observability --sort-by=.lastTimestamp
```

Look for:

```text
CrashLoopBackOff
OOMKilled
FailedScheduling
Configuration errors
ImagePullBackOff
```

---

# 80. Troubleshooting Collector Service

Check:

```bash
kubectl get svc -n observability
```

Then:

```bash
kubectl get endpoints -n observability
```

Verify:

```text
Service name
Port
Target port
Endpoints
Selectors
```

A Service with no endpoints indicates that no matching healthy Pods are currently available.

---

# 81. Troubleshooting Connection Refused

If an application reports:

```text
connection refused
```

check:

```text
Collector Pod
Collector Service
Service port
Target port
Receiver
NetworkPolicy
```

Common problem:

```text
Application
   ↓
4318
   X
Collector listening on 4317
```

The protocol and port must match.

---

# 82. Troubleshooting No Traces

Check:

```text
Instrumentation
TracerProvider
Sampler
OTLP exporter
Collector receiver
Traces pipeline
Processors
Trace exporter
Backend
```

A processor can also intentionally or accidentally drop traces.

---

# 83. Troubleshooting No Metrics

Check:

```text
MeterProvider
Metric instrument
Metric reader
Exporter
Collector receiver
Metrics pipeline
Metrics exporter
Prometheus/backend
```

Then verify the expected metric name and resource attributes.

---

# 84. Troubleshooting No Logs

Check:

```text
LoggerProvider
Logging integration
Logs exporter
Collector logs receiver
Logs pipeline
Exporter
Elasticsearch / Logstash
```

Also check filters.

---

# 85. Troubleshooting High CPU

Possible causes:

```text
High telemetry volume
Complex transformations
Tail sampling
Large attribute processing
High-cardinality data
Too many pipelines
```

Approach:

```text
Measure
   ↓
Identify expensive component
   ↓
Optimize
   ↓
Reduce unnecessary telemetry
   ↓
Scale
```

---

# 86. Troubleshooting High Memory

Possible causes:

```text
High telemetry volume
Large queues
Slow backend
Large batches
Tail sampling
Exporter failures
```

Approach:

```text
Memory limiter
   ↓
Queue analysis
   ↓
Backend health
   ↓
Batch tuning
   ↓
Horizontal scaling
```

---

# 87. Troubleshooting Export Failures

Check:

```text
Endpoint
DNS
Network
TLS
Authentication
Backend health
Exporter configuration
Queue
Retry
```

Example:

```text
Collector
   ↓
Exporter
   ↓
TLS handshake failure
   ↓
Backend
```

Investigate certificates and trust configuration.

---

# 88. Collector Security

Production security should include:

```text
TLS
Authentication
Authorization
Network policies
Private endpoints
RBAC
Secret management
Sensitive-data filtering
```

Do not expose Collector endpoints publicly without a strong reason and appropriate controls.

---

# 89. Secret Management

Avoid:

```text
Git
 ↓
Plaintext password
```

Preferred:

```text
Secret Manager
      ↓
External Secret / Kubernetes Secret
      ↓
Collector
```

Private keys and credentials should never be committed to the repository.

---

# 90. Network Security

A secure architecture:

```text
Application
     ↓
Private Cluster Network
     ↓
OTel Agent
     ↓
Private Cluster Network
     ↓
OTel Gateway
     ↓
Private Backend
```

Use security groups, NetworkPolicies, and backend access controls according to the environment.

---

# 91. Resource Protection

Every production Collector should have:

```text
CPU request
Memory request
CPU limit
Memory limit
Memory limiter
```

This protects the Kubernetes cluster from uncontrolled Collector resource consumption.

---

# 92. Collector Backpressure

If the backend cannot keep up:

```text
Telemetry
    ↓
Collector
    ↓
Backend slower
    ↓
Queue grows
```

The system needs a defined backpressure strategy.

Possible actions:

```text
Retry
Queue
Drop low-value telemetry
Sample
Scale backend
Scale Collector
```

---

# 93. Observability Cost Control

Collector processing can reduce cost.

For example:

```text
Raw telemetry
      ↓
Filter
      ↓
Sampling
      ↓
Batch
      ↓
Export
```

This reduces unnecessary:

```text
Network usage
Storage
Backend ingestion
```

Cost control should not remove telemetry required for incident investigation.

---

# 94. Collector Capacity Planning

Consider:

```text
Services
Requests per second
Spans per second
Log events per second
Metric cardinality
Average telemetry size
Processors
Sampling
Backend latency
```

Then determine:

```text
Collector CPU
Collector memory
Replica count
Network capacity
Backend capacity
```

---

# 95. Collector High Availability Checklist

```text
[ ] Multiple replicas
[ ] Multiple nodes
[ ] Multiple availability zones where appropriate
[ ] Pod anti-affinity / topology spread
[ ] PodDisruptionBudget
[ ] Kubernetes Service
[ ] Health checks
[ ] Resource requests
[ ] Resource limits
[ ] Memory limiter
[ ] Monitoring
```

---

# 96. Collector Production Security Checklist

```text
[ ] TLS
[ ] Authentication
[ ] Authorization
[ ] Private endpoints
[ ] Network policies
[ ] Least-privilege RBAC
[ ] Secret management
[ ] Sensitive data filtering
[ ] Restricted diagnostic endpoints
[ ] Secure backend connections
```

---

# 97. Collector Production Operations Checklist

```text
[ ] GitOps deployment
[ ] Configuration version control
[ ] CI validation
[ ] Security scanning
[ ] Health monitoring
[ ] Collector metrics
[ ] Alerts
[ ] Capacity planning
[ ] Rollback procedure
[ ] Upgrade procedure
[ ] Failure testing
```

---

# 98. Complete EKS Production Architecture

```text
                                  AWS
                                   │
                                  EKS
                                   │
          ┌────────────────────────┼────────────────────────┐
          ↓                        ↓                        ↓
       Node-01                  Node-02                  Node-03
          │                        │                        │
    Applications             Applications             Applications
          │                        │                        │
      OTel Agent               OTel Agent               OTel Agent
          │                        │                        │
          └────────────────────────┼────────────────────────┘
                                   ↓
                           OTel Gateway Service
                                   │
                    ┌──────────────┼──────────────┐
                    ↓              ↓              ↓
                Gateway-01     Gateway-02     Gateway-03
                    │              │              │
                    └──────────────┼──────────────┘
                                   ↓
                    ┌──────────────┼──────────────┐
                    ↓              ↓              ↓
                Prometheus        ELK            Jaeger
                    ↓              ↓              ↓
                 Grafana         Kibana        Trace UI
```

---

# 99. Collector Mental Model

Remember the Collector using:

```text
RECEIVE
   ↓
PROCESS
   ↓
EXPORT
```

More specifically:

```text
Receivers
   ↓
Memory Limiter
   ↓
Resource / Attributes
   ↓
Filter / Transform
   ↓
Sampling
   ↓
Batch
   ↓
Exporters
   ↓
Backends
```

And for Kubernetes:

```text
Application
   ↓
OTel Agent
   ↓
OTel Gateway
   ↓
Backend
```

---

# 100. Final Key Concepts

The most important OpenTelemetry Collector concepts are:

```text
Receiver
Processor
Exporter
Connector
Extension
Pipeline
Agent
Gateway
OTLP
Batching
Memory Limiting
Filtering
Sampling
Routing
Retry
Queueing
Health Checks
High Availability
```

The production principle is:

**The OpenTelemetry Collector provides the central telemetry processing layer between applications and observability backends. Applications generate telemetry through OpenTelemetry SDKs, while Collectors receive, enrich, filter, batch, sample, route, and export that telemetry. In an EKS production environment, an agent-plus-gateway architecture provides scalable collection and centralized processing, while Kubernetes Services, resource limits, memory protection, high availability, TLS, authentication, monitoring, and GitOps provide the operational foundation required for reliable observability.**
