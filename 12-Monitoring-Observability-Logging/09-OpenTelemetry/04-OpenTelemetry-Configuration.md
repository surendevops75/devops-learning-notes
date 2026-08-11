# OpenTelemetry Configuration

## 1. Overview

OpenTelemetry configuration controls how telemetry is generated, identified, processed, transported, and exported.

The overall flow is:

```text
Application
    ↓
OTel SDK / Auto-Instrumentation
    ↓
Telemetry
    ↓
OTel Collector
    ↓
Receivers
    ↓
Processors
    ↓
Exporters
    ↓
Backend
```

A production configuration must address:

```text
Service identity
Resource attributes
Receivers
Processors
Exporters
Pipelines
Sampling
Batching
Memory limits
TLS
Authentication
Networking
Resource limits
```

---

# 2. Configuration Layers

OpenTelemetry configuration normally exists at multiple layers:

```text
Application
│
├── SDK configuration
├── Instrumentation configuration
├── Resource configuration
└── Exporter configuration

Collector
│
├── Receivers
├── Processors
├── Exporters
├── Extensions
└── Service pipelines
```

Both layers must work together.

---

# 3. Application Configuration

Application configuration commonly defines:

```text
service.name
service.version
environment
OTLP endpoint
protocol
sampling
resource attributes
```

Example:

```bash
export OTEL_SERVICE_NAME=payment
export OTEL_SERVICE_VERSION=v1.5.2
export OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-gateway:4317
```

---

# 4. Standard Environment Variables

OpenTelemetry provides standardized environment variables.

Common examples:

```text
OTEL_SERVICE_NAME
OTEL_RESOURCE_ATTRIBUTES
OTEL_EXPORTER_OTLP_ENDPOINT
OTEL_EXPORTER_OTLP_PROTOCOL
OTEL_TRACES_EXPORTER
OTEL_METRICS_EXPORTER
OTEL_LOGS_EXPORTER
OTEL_TRACES_SAMPLER
```

The exact supported variables depend on the language SDK.

---

# 5. Service Name Configuration

Every service should have a consistent service name.

Example:

```bash
export OTEL_SERVICE_NAME=payment
```

Another service:

```bash
export OTEL_SERVICE_NAME=orders
```

Then telemetry can be grouped:

```text
payment
orders
inventory
notification
```

---

# 6. Service Version

Configure the application version:

```bash
export OTEL_SERVICE_VERSION=v1.5.2
```

This helps correlate telemetry with deployments.

Example:

```text
payment v1.5.1 → healthy
payment v1.5.2 → latency increased
```

---

# 7. Deployment Environment

Use an environment attribute:

```bash
export OTEL_RESOURCE_ATTRIBUTES="deployment.environment=production"
```

Then telemetry can be filtered by:

```text
production
staging
development
```

This is particularly important when multiple environments share observability infrastructure.

---

# 8. Multiple Resource Attributes

Multiple attributes can be configured together.

Example:

```bash
export OTEL_RESOURCE_ATTRIBUTES="deployment.environment=production,service.version=v1.5.2"
```

The final resource might contain:

```text
service.name = payment
service.version = v1.5.2
deployment.environment = production
```

---

# 9. Kubernetes Resource Attributes

In Kubernetes, useful resource information includes:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
```

Example:

```text
service.name = payment
k8s.namespace.name = production
k8s.pod.name = payment-7d8f
```

This makes Kubernetes troubleshooting much easier.

---

# 10. OTLP Endpoint

Applications can send telemetry through OTLP.

Example:

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-gateway:4317
```

Architecture:

```text
Application
     ↓
OTLP
     ↓
OTel Gateway
```

The endpoint must match the Collector's exposed protocol and port.

---

# 11. OTLP Protocol

Common OTLP transport options include:

```text
gRPC
HTTP
```

Configuration may specify the protocol explicitly.

Example:

```bash
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
```

The application and Collector must use compatible settings.

---

# 12. Signal-Specific Endpoints

Applications can use a common OTLP endpoint or signal-specific endpoints depending on SDK capabilities.

Conceptually:

```text
Metrics → Collector
Logs    → Collector
Traces  → Collector
```

The Collector then routes each signal through its corresponding pipeline.

---

# 13. Traces Exporter

A trace exporter can be configured through the SDK.

Conceptually:

```bash
export OTEL_TRACES_EXPORTER=otlp
```

Architecture:

```text
Application
     ↓
Trace SDK
     ↓
OTLP
     ↓
Collector
```

---

# 14. Metrics Exporter

Conceptually:

```bash
export OTEL_METRICS_EXPORTER=otlp
```

Architecture:

```text
Application
     ↓
Metrics SDK
     ↓
OTLP
     ↓
Collector
```

The backend can then be Prometheus-compatible or another metrics platform depending on the Collector configuration.

---

# 15. Logs Exporter

Conceptually:

```bash
export OTEL_LOGS_EXPORTER=otlp
```

Architecture:

```text
Application
     ↓
Logs SDK
     ↓
OTLP
     ↓
Collector
```

The Collector then sends the logs to the configured logging backend.

---

# 16. SDK Configuration Flow

The application configuration can be visualized as:

```text
Environment Variables
        ↓
OTel SDK
        ↓
Resource
        ↓
Instrumentation
        ↓
Telemetry
        ↓
Exporter
```

The SDK configuration controls the application-side telemetry generation and export behavior.

---

# 17. Collector Configuration

The Collector configuration has five important areas:

```text
receivers
processors
exporters
extensions
service
```

Conceptually:

```yaml
receivers:
  ...

processors:
  ...

exporters:
  ...

extensions:
  ...

service:
  pipelines:
    ...
```

---

# 18. Receivers

Receivers define how telemetry enters the Collector.

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
OTLP Receiver
```

---

# 19. OTLP gRPC Receiver

Conceptually:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
```

The Collector then accepts OTLP telemetry over gRPC.

A common OTLP gRPC port is:

```text
4317
```

---

# 20. OTLP HTTP Receiver

Conceptually:

```yaml
receivers:
  otlp:
    protocols:
      http:
```

A common OTLP HTTP port is:

```text
4318
```

Architecture:

```text
Application
     ↓
OTLP/HTTP
     ↓
Collector
```

---

# 21. Multiple Receivers

A Collector can have multiple receivers.

Example:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

  prometheus:
    config:
      scrape_configs:
        ...
```

Different receivers can feed different pipelines.

---

# 22. Processors

Processors transform or control telemetry.

Common processors:

```text
batch
memory_limiter
resource
attributes
filter
transform
sampling
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

# 23. Batch Processor

Batching reduces the overhead of sending telemetry individually.

Example:

```yaml
processors:
  batch:
```

Architecture:

```text
Event 1 ┐
Event 2 ├──→ Batch
Event 3 ┘
           ↓
        Export
```

Benefits:

```text
Lower network overhead
Better throughput
More efficient backend ingestion
```

---

# 24. Batch Tuning

Batch processing can be tuned according to workload.

Conceptually:

```yaml
processors:
  batch:
    timeout: 5s
    send_batch_size: 1024
```

These values are examples.

Do not blindly use the same values for every environment.

---

# 25. Memory Limiter

The memory limiter protects the Collector from excessive memory usage.

Conceptually:

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

The exact configuration should be sized against the Collector's container memory limit.

---

# 26. Processor Ordering

Processor order matters.

A common conceptual sequence is:

```text
Receiver
   ↓
Memory Limiter
   ↓
Resource
   ↓
Filter
   ↓
Sampling
   ↓
Batch
   ↓
Exporter
```

The correct ordering depends on the processing requirements.

---

# 27. Resource Processor

The resource processor can add or modify resource attributes.

Conceptually:

```yaml
processors:
  resource:
    attributes:
      - key: deployment.environment
        value: production
        action: upsert
```

This can standardize metadata across telemetry.

---

# 28. Attributes Processor

The attributes processor works with telemetry attributes.

It can be used to:

```text
Insert attributes
Update attributes
Delete attributes
Hash attributes
```

This is useful when normalizing telemetry.

---

# 29. Filtering

Not all telemetry needs to be retained.

Conceptually:

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
Storage
Network traffic
Backend load
Cost
```

---

# 30. Sensitive Data Filtering

Telemetry may contain sensitive information.

Examples:

```text
Authorization headers
Tokens
Email addresses
Personal information
Secrets
```

Prefer preventing sensitive data from being generated.

Collector-level filtering can provide an additional protection layer.

---

# 31. Sampling Configuration

Sampling reduces trace volume.

Conceptually:

```text
1,000,000 traces
       ↓
Sampling
       ↓
100,000 traces
```

A simple sampling strategy can be configured at the SDK or Collector layer depending on requirements.

---

# 32. Head Sampling

Head sampling makes a decision near the beginning of a trace.

```text
Request
   ↓
Sampling Decision
   ↓
Keep / Drop
```

Advantages:

```text
Lower resource usage
Lower network traffic
Simpler processing
```

---

# 33. Tail Sampling

Tail sampling makes a decision after more information about a trace is available.

```text
Trace
 ↓
Collector
 ↓
Complete Trace
 ↓
Sampling Decision
```

This allows policies such as:

```text
Keep errors
Keep slow traces
Sample successful traces
```

---

# 34. Tail Sampling Example

Conceptually:

```yaml
processors:
  tail_sampling:
    decision_wait: 10s
    policies:
      - name: errors
        type: status_code
        status_code:
          status_codes:
            - ERROR
```

The exact configuration depends on the Collector distribution and supported processor configuration.

---

# 35. Exporters

Exporters send processed telemetry to destinations.

Examples:

```text
OTLP
Prometheus-compatible destination
Elasticsearch
Jaeger-compatible destination
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

# 36. Debug Exporter

During development, a debug exporter can help verify telemetry.

Conceptually:

```yaml
exporters:
  debug:
```

Pipeline:

```text
Application
    ↓
Collector
    ↓
Debug Exporter
    ↓
Collector Output
```

This is useful for troubleshooting before connecting production backends.

---

# 37. OTLP Exporter

An OTLP exporter sends telemetry to another OTLP endpoint.

Conceptually:

```yaml
exporters:
  otlp:
    endpoint: backend.example:4317
```

Architecture:

```text
Collector
    ↓
OTLP
    ↓
Another Collector / Backend
```

TLS and authentication should be configured when required.

---

# 38. Prometheus Metrics Configuration

Prometheus integration can be designed in different ways.

One architecture is:

```text
Application
     ↓
OTel Collector
     ↓
Prometheus-compatible output
     ↓
Prometheus
```

Another architecture can use Prometheus to scrape a Collector endpoint.

The correct approach depends on the chosen OpenTelemetry and Prometheus architecture.

---

# 39. Elasticsearch Configuration

For logging:

```text
Application
     ↓
OTel Collector
     ↓
Elasticsearch
```

Alternatively:

```text
Application
     ↓
OTel Collector
     ↓
Logstash
     ↓
Elasticsearch
```

The second architecture is useful when existing Logstash pipelines provide required processing.

---

# 40. Jaeger Configuration

For tracing:

```text
Application
     ↓
OTel Collector
     ↓
Jaeger
```

The Collector can export trace data to the selected Jaeger endpoint according to the supported exporter architecture.

---

# 41. Service Pipelines

Processors and exporters do nothing unless referenced by pipelines.

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

The pipeline connects the components.

---

# 42. Metrics Pipeline

Example:

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
        - prometheus
```

Conceptually:

```text
OTLP
 ↓
Memory Limiter
 ↓
Batch
 ↓
Prometheus Export
```

---

# 43. Logs Pipeline

Example:

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

Conceptually:

```text
OTLP
 ↓
Memory Limiter
 ↓
Batch
 ↓
Log Export
```

---

# 44. Traces Pipeline

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

For advanced production tracing:

```text
OTLP
 ↓
Memory Limiter
 ↓
Resource
 ↓
Tail Sampling
 ↓
Batch
 ↓
OTLP
```

---

# 45. Extensions

Extensions provide functionality outside the normal telemetry pipelines.

Common examples include:

```text
health_check
pprof
zpages
```

Example:

```yaml
extensions:
  health_check:
```

Then enable it:

```yaml
service:
  extensions:
    - health_check
```

---

# 46. Health Check Configuration

A health check endpoint allows Kubernetes or monitoring systems to verify Collector health.

Conceptually:

```text
Kubernetes
    ↓
Health Probe
    ↓
Collector
```

If the Collector becomes unhealthy, Kubernetes can take appropriate action.

---

# 47. pprof

`pprof` can help diagnose Collector performance issues.

It can provide profiling information for:

```text
CPU
Memory
Goroutines
```

It should be protected and not unnecessarily exposed publicly.

---

# 48. zPages

zPages can provide runtime diagnostic information for Collector pipelines.

It can help during troubleshooting.

Like profiling endpoints, it should be appropriately protected and restricted.

---

# 49. Complete Collector Configuration

A conceptual production configuration:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  memory_limiter:
    limit_percentage: 80
    spike_limit_percentage: 15

  batch:
    timeout: 5s
    send_batch_size: 1024

  resource:
    attributes:
      - key: deployment.environment
        value: production
        action: upsert

exporters:
  otlp:
    endpoint: backend:4317

extensions:
  health_check:

service:
  extensions:
    - health_check

  pipelines:
    traces:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - resource
        - batch
      exporters:
        - otlp

    metrics:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - resource
        - batch
      exporters:
        - otlp

    logs:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - resource
        - batch
      exporters:
        - otlp
```

This is a conceptual template. Production configurations must be adapted to the installed Collector distribution and backend.

---

# 50. Kubernetes ConfigMap

A Collector configuration can be stored in a Kubernetes ConfigMap.

Conceptually:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: otel-collector-config
  namespace: observability
data:
  collector.yaml: |
    receivers:
      otlp:
        protocols:
          grpc:
          http:
```

The Collector Pod mounts or receives the configuration through the deployment mechanism.

---

# 51. Configuration Through Operator

When using the OpenTelemetry Operator, the configuration can be embedded in the `OpenTelemetryCollector` resource.

Conceptually:

```yaml
apiVersion: opentelemetry.io/v1beta1
kind: OpenTelemetryCollector
metadata:
  name: otel-gateway
  namespace: observability
spec:
  mode: deployment
  config:
    receivers:
      otlp:
        protocols:
          grpc:
          http:
```

The exact API version and schema depend on the installed Operator version.

---

# 52. Kubernetes Service Configuration

Expose the Collector through a Kubernetes Service.

Conceptually:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: otel-gateway
  namespace: observability
spec:
  selector:
    app: otel-gateway
  ports:
    - name: otlp-grpc
      port: 4317
      targetPort: 4317
```

Applications then connect through:

```text
otel-gateway.observability.svc.cluster.local:4317
```

---

# 53. Service Discovery

Applications should use the Kubernetes Service rather than Pod IPs.

Bad:

```text
10.20.3.45:4317
```

Preferred:

```text
otel-gateway.observability.svc.cluster.local:4317
```

Why?

```text
Pod IP
 ↓
Can change

Service DNS
 ↓
Stable
```

---

# 54. DaemonSet Configuration

An agent Collector can run as a DaemonSet.

Conceptually:

```yaml
spec:
  mode: daemonset
```

Architecture:

```text
Node-01 → Agent
Node-02 → Agent
Node-03 → Agent
```

This is useful for node-local collection.

---

# 55. Gateway Deployment Configuration

A gateway Collector can run as a Deployment:

```yaml
spec:
  mode: deployment
```

Example architecture:

```text
Gateway
│
├── Pod-01
├── Pod-02
└── Pod-03
```

Use a Kubernetes Service in front of the gateway.

---

# 56. Resource Configuration

Collector resources must be defined.

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

These values are examples.

Measure actual workload before finalizing them.

---

# 57. Memory Limiter and Container Memory

Suppose:

```text
Container memory limit = 1 GiB
```

The Collector memory limiter should be configured with enough headroom for the process itself and normal runtime behavior.

Do not configure memory usage right up to the container limit.

Otherwise:

```text
Memory pressure
    ↓
OOMKilled
    ↓
Telemetry interruption
```

---

# 58. Replica Configuration

For a production gateway:

```text
2 or more replicas
```

depending on availability requirements.

Example:

```text
Gateway
├── Pod-01
├── Pod-02
└── Pod-03
```

Then distribute them across nodes or availability zones where practical.

---

# 59. Pod Anti-Affinity

A production Collector gateway should avoid placing all replicas on the same node.

Conceptually:

```text
Node-01 → Gateway-01
Node-02 → Gateway-02
Node-03 → Gateway-03
```

Kubernetes scheduling controls can help achieve this.

---

# 60. Pod Disruption Budget

For a multi-replica gateway, a PodDisruptionBudget can preserve minimum availability during voluntary disruptions.

Example concept:

```text
3 replicas
minimum available = 2
```

The exact setting depends on:

```text
Replica count
Cluster capacity
Maintenance strategy
Availability requirements
```

---

# 61. TLS Configuration

For secure OTLP communication:

```text
Application
    ↓ TLS
Collector
```

Conceptually:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        tls:
          cert_file: /certs/tls.crt
          key_file: /certs/tls.key
```

Certificate paths and configuration depend on how certificates are mounted and managed.

---

# 62. TLS Certificates

Certificates can be provided through:

```text
Kubernetes Secret
Certificate manager
External secret system
Cloud certificate infrastructure
```

Example:

```text
Secret
 ↓
Collector Pod
 ↓
/certs/tls.crt
/certs/tls.key
```

Never commit private keys to Git.

---

# 63. Authentication Configuration

Authentication can be implemented using appropriate Collector extensions or exporter/receiver mechanisms supported by the installed distribution.

Architecture:

```text
Application
     ↓
Authenticated OTLP
     ↓
Collector
```

The authentication method should be selected based on:

```text
Trust boundary
Network architecture
Backend
Identity platform
Security requirements
```

---

# 64. Network Policy

Restrict Collector access.

Conceptually:

```text
Application Namespace
       ↓
OTel Collector
       ↓
Allowed

Internet
       ↓
Blocked
```

NetworkPolicy can limit which workloads communicate with the Collector.

---

# 65. Production Configuration Management

Collector configuration should be version controlled.

Example:

```text
observability/
└── opentelemetry/
    ├── base/
    │   ├── collector.yaml
    │   └── service.yaml
    │
    └── overlays/
        ├── dev/
        ├── staging/
        └── production/
```

This supports repeatable deployment.

---

# 66. GitOps Configuration

For your environment:

```text
GitHub
   ↓
OpenTelemetry configuration
   ↓
GitHub Actions
   ↓
Validation / Security
   ↓
ArgoCD
   ↓
EKS
```

ArgoCD manages the desired Kubernetes state.

---

# 67. Configuration Validation

Before applying a Collector configuration:

```text
YAML validation
      ↓
Collector configuration validation
      ↓
Helm validation
      ↓
Kubernetes validation
      ↓
Security scan
```

This reduces production configuration errors.

---

# 68. Collector Configuration Validation

After deployment:

```bash
kubectl get pods -n observability
```

Then:

```bash
kubectl logs <collector-pod> -n observability
```

Look for:

```text
Configuration loaded
Receiver started
Pipeline started
Exporter started
```

---

# 69. Configuration Change Process

A safe production change:

```text
Developer
   ↓
Modify Git configuration
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
Sync
   ↓
Monitor
```

Avoid direct production editing unless there is a controlled emergency process.

---

# 70. Configuration Rollback

If a configuration change breaks telemetry:

```text
Current configuration
       ↓
Broken
```

Rollback:

```text
Git revert
   ↓
Previous configuration
   ↓
ArgoCD
   ↓
Collector
   ↓
Healthy
```

This is safer and more auditable than manually reconstructing the previous configuration.

---

# 71. Environment-Specific Configuration

Development:

```text
debug exporter
low telemetry volume
simpler security
```

Staging:

```text
production-like pipeline
load testing
failure testing
```

Production:

```text
HA
TLS
authentication
sampling
resource limits
monitoring
```

---

# 72. Development Configuration

A simple development pipeline:

```text
Application
    ↓
OTLP
    ↓
Collector
    ↓
Debug Exporter
```

This makes it easy to confirm telemetry generation.

---

# 73. Staging Configuration

Staging should resemble production:

```text
Application
    ↓
OTel Collector
    ↓
Processing
    ↓
Backend
```

Validate:

```text
Batching
Sampling
TLS
Authentication
Failure handling
Scaling
```

---

# 74. Production Configuration

Production architecture:

```text
Applications
     ↓
OTel Agent
     ↓
OTel Gateway HA
     ↓
Signal Pipelines
     ↓
Backends
```

Controls:

```text
Memory limiting
Batching
Sampling
Filtering
TLS
Authentication
Monitoring
```

---

# 75. Environment Variables in Kubernetes

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: "payment"

  - name: OTEL_RESOURCE_ATTRIBUTES
    value: "deployment.environment=production"

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: "http://otel-gateway.observability.svc.cluster.local:4317"
```

For sensitive values, use Kubernetes Secrets or external secret-management mechanisms.

---

# 76. ConfigMap vs Secret

Use ConfigMap for:

```text
Collector configuration
Non-sensitive settings
Endpoints that are not secret
```

Use Secret for:

```text
Passwords
API tokens
Private keys
Credentials
Certificates
```

Never store sensitive credentials in a ConfigMap.

---

# 77. Configuration Precedence

When configuring OpenTelemetry, multiple configuration mechanisms may exist.

For example:

```text
Application defaults
      ↓
SDK configuration
      ↓
Environment variables
      ↓
Deployment-specific settings
```

The exact precedence depends on the language SDK and component.

Always verify the SDK documentation when multiple settings conflict.

---

# 78. Common Configuration Mistakes

### Mistake 1

Incorrect service name:

```text
service.name = payment-service-prod-v2-random
```

Use a stable service identity.

---

### Mistake 2

Wrong Collector endpoint:

```text
otel-gateway:4318
```

when the application is configured for gRPC on:

```text
4317
```

Protocol and port must match.

---

# 79. Common Configuration Mistakes

### Mistake 3

Receiver exists but is not in pipeline.

```text
Receiver
   ↓
Not referenced
```

Therefore no telemetry flows through it.

---

### Mistake 4

Exporter exists but is not referenced.

```text
Exporter
   ↓
Not referenced
```

No telemetry is exported through it.

---

# 80. Common Configuration Mistakes

### Mistake 5

No memory limiter.

```text
High telemetry volume
      ↓
Memory growth
      ↓
OOMKilled
```

---

### Mistake 6

No batching.

```text
Huge number of individual exports
      ↓
Network overhead
      ↓
Higher CPU
```

---

# 81. Common Configuration Mistakes

### Mistake 7

No sampling in a very high-volume tracing environment.

```text
Millions of traces
      ↓
Huge storage
      ↓
High cost
```

---

### Mistake 8

Sensitive data exported without filtering.

```text
Tokens
Passwords
PII
      ↓
Telemetry backend
```

This creates a security risk.

---

# 82. Configuration Troubleshooting

When the Collector fails:

```text
Check YAML
   ↓
Check component names
   ↓
Check receiver
   ↓
Check processors
   ↓
Check exporters
   ↓
Check pipelines
```

Then inspect:

```bash
kubectl logs <collector-pod> -n observability
```

---

# 83. Telemetry Not Arriving

Check application configuration:

```text
service.name
endpoint
protocol
exporter
```

Then:

```text
Application
   ↓
Network
   ↓
Collector Service
   ↓
Receiver
```

---

# 84. Telemetry Dropped

Check:

```text
Memory limiter
Filter processor
Sampling processor
Queue
Exporter failures
Backend availability
```

Collector operational metrics are especially useful here.

---

# 85. High CPU Configuration Troubleshooting

Possible causes:

```text
Complex processors
High telemetry volume
Tail sampling
Transformations
Large number of pipelines
```

Actions:

```text
Measure
 ↓
Identify expensive processing
 ↓
Reduce unnecessary telemetry
 ↓
Scale Collector
```

---

# 86. High Memory Configuration Troubleshooting

Possible causes:

```text
Large queues
Slow backend
High volume
Large batches
Tail sampling
```

Actions:

```text
Memory limiter
 ↓
Queue tuning
 ↓
Batch tuning
 ↓
Backend troubleshooting
 ↓
Horizontal scaling
```

---

# 87. Export Failure Configuration Troubleshooting

Check:

```text
Exporter endpoint
TLS
Authentication
Network
Backend availability
Retry
Queue
```

Example:

```text
Collector
   ↓
Exporter
   ↓
Connection refused
```

This points toward endpoint, network, or backend availability.

---

# 88. Production Configuration Monitoring

Monitor the Collector itself:

```text
Received telemetry
Exported telemetry
Dropped telemetry
Exporter errors
Queue size
CPU
Memory
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

# 89. Configuration Alerts

Useful alerts include:

```text
Collector unavailable
Exporter failures
Dropped telemetry
High memory
High CPU
Queue growth
No telemetry received
```

The exact thresholds should be based on normal baseline behavior.

---

# 90. Configuration Security Checklist

```text
[ ] TLS enabled where required
[ ] Authentication configured
[ ] Secrets stored securely
[ ] Network policies configured
[ ] RBAC least privilege
[ ] Sensitive attributes filtered
[ ] Public exposure avoided
[ ] Certificates managed securely
[ ] Configuration reviewed
```

---

# 91. Configuration Reliability Checklist

```text
[ ] Memory limiter
[ ] Batch processor
[ ] Retry
[ ] Queue where required
[ ] Multiple replicas
[ ] Multi-node placement
[ ] Health checks
[ ] Resource requests
[ ] Resource limits
[ ] Monitoring
```

---

# 92. Configuration Deployment Checklist

```text
[ ] Git repository updated
[ ] Pull request created
[ ] YAML validated
[ ] Helm validated
[ ] Security scanned
[ ] Reviewed
[ ] Merged
[ ] ArgoCD synchronized
[ ] Collector healthy
[ ] Telemetry verified
```

---

# 93. Production Configuration Example

A simplified production architecture:

```text
                         EKS
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
   Application         Application        Application
       │                  │                  │
    OTel SDK            OTel SDK            OTel SDK
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                    OTel Agents
                          ↓
                  OTel Gateway HA
                          │
            ┌─────────────┼─────────────┐
            ↓             ↓             ↓
        Metrics          Logs         Traces
            ↓             ↓             ↓
       Prometheus          ELK          Jaeger
            ↓             ↓             ↓
         Grafana         Kibana       Trace UI
```

---

# 94. Recommended Configuration Flow

For a production EKS microservices environment:

```text
1. Configure service identity
        ↓
2. Configure resource attributes
        ↓
3. Configure SDK / instrumentation
        ↓
4. Configure OTLP endpoint
        ↓
5. Deploy Collector
        ↓
6. Configure receivers
        ↓
7. Configure processors
        ↓
8. Configure exporters
        ↓
9. Configure pipelines
        ↓
10. Configure security
        ↓
11. Configure resource limits
        ↓
12. Configure HA
        ↓
13. Monitor Collector
        ↓
14. Validate telemetry
```

---

# 95. Final Production Configuration Architecture

```text
                       APPLICATION
                            │
                    OTel SDK / Agent
                            │
                ┌───────────┼───────────┐
                ↓           ↓           ↓
             Metrics       Logs       Traces
                │           │           │
                └───────────┼───────────┘
                            ↓
                       OTel Agent
                            ↓
                      OTLP / TLS
                            ↓
                    OTel Gateway HA
                            │
                     ┌──────┼──────┐
                     ↓      ↓      ↓
                  Receive Process Export
                            │
                            ↓
                     Signal Backends
```

---

# 96. OpenTelemetry Configuration Mental Model

Remember:

```text
APPLICATION CONFIG
│
├── Service Name
├── Service Version
├── Environment
├── Resource Attributes
├── Exporter
└── OTLP Endpoint

COLLECTOR CONFIG
│
├── Receivers
├── Processors
├── Exporters
├── Extensions
└── Pipelines

PRODUCTION CONFIG
│
├── TLS
├── Authentication
├── Memory Limits
├── Batching
├── Sampling
├── Filtering
├── High Availability
├── Monitoring
└── GitOps
```

The key principle is:

**OpenTelemetry configuration connects application instrumentation to the telemetry pipeline. At the application layer, configure service identity, resource attributes, instrumentation, exporters, and OTLP endpoints. At the Collector layer, configure receivers, processors, exporters, extensions, and signal-specific pipelines. In production EKS environments, configuration should additionally include batching, memory protection, sampling, secure transport, authentication, resource limits, high availability, monitoring, and GitOps-based deployment.**
