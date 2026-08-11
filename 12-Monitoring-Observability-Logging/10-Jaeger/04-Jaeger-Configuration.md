# Jaeger Configuration

## 1. Overview

Jaeger configuration defines how the tracing platform:

```text
Receives traces
Processes traces
Stores traces
Queries traces
Exposes the UI
Handles security
Uses resources
```

In a production OpenTelemetry architecture:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OpenTelemetry Collector
     ↓
Jaeger
     ↓
Storage
     ↓
Jaeger Query
     ↓
Jaeger UI
```

Configuration must be designed according to:

```text
Traffic volume
Sampling strategy
Storage backend
Security requirements
Availability requirements
Kubernetes architecture
```

---

# 2. Configuration Layers

Jaeger configuration can be understood in several layers:

```text
Application
    ↓
OpenTelemetry Configuration
    ↓
Collector Configuration
    ↓
Jaeger Configuration
    ↓
Storage Configuration
    ↓
Query / UI Configuration
```

Each layer has a different responsibility.

---

# 3. Application Configuration

The application controls:

```text
Service name
OTLP endpoint
Instrumentation
Sampling
Resource attributes
Trace propagation
```

Example:

```bash
export OTEL_SERVICE_NAME=orders
export OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-gateway:4317
```

The exact environment variables depend on the language SDK and deployment.

---

# 4. Service Name

Every application should have a meaningful service name.

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
export OTEL_SERVICE_NAME=payment
```

Jaeger uses the service information to group and search traces.

---

# 5. Service Version

Include the application version where possible.

Example:

```text
service.name = payment
service.version = 2.4.1
```

This is especially useful during deployments.

Example:

```text
payment v2.3.0 → 250ms
payment v2.4.0 → 900ms
```

Jaeger can then help investigate the release that introduced the latency.

---

# 6. Deployment Environment

Configure the deployment environment.

Examples:

```text
production
staging
development
```

Conceptually:

```text
service.name = payment
service.version = 2.4.1
deployment.environment = production
```

This prevents traces from different environments from becoming difficult to distinguish.

---

# 7. Kubernetes Resource Attributes

In Kubernetes, useful attributes include:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
```

These can be added through OpenTelemetry resource detection and Kubernetes metadata enrichment.

---

# 8. OTLP Configuration

Applications commonly send traces using OTLP.

Two common protocols:

```text
OTLP/gRPC
OTLP/HTTP
```

Common ports:

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

# 9. OTLP Endpoint

Example:

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-gateway.observability.svc.cluster.local:4317
```

The endpoint should point to the appropriate Collector endpoint.

Do not hard-code Pod IP addresses.

Use:

```text
Kubernetes Service
```

instead.

---

# 10. Kubernetes Service Configuration

Example architecture:

```text
Application
     ↓
otel-gateway
     ↓
Gateway Pods
```

The application connects to:

```text
otel-gateway.observability.svc.cluster.local
```

Kubernetes DNS resolves the Service.

---

# 11. Collector Configuration

The Collector is usually configured with:

```text
Receivers
Processors
Exporters
Service Pipelines
```

Conceptually:

```yaml
receivers:
  otlp:

processors:
  memory_limiter:
  batch:

exporters:
  <jaeger-compatible-exporter>:

service:
  pipelines:
    traces:
```

The exact exporter depends on the current Collector distribution and Jaeger version.

---

# 12. OTLP Receiver

A typical Collector receiver:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:
```

This allows the Collector to receive:

```text
OTLP/gRPC
OTLP/HTTP
```

---

# 13. Batch Processor

Batching reduces the number of individual export operations.

Conceptually:

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

Batch configuration should be tuned according to telemetry volume.

---

# 14. Memory Limiter

Use the Memory Limiter Processor to protect the Collector.

Example:

```yaml
processors:
  memory_limiter:
    check_interval: 1s
    limit_percentage: 80
    spike_limit_percentage: 15
```

The exact values should be based on Collector resource allocation and workload.

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

---

# 15. Trace Sampling

Sampling controls how much trace data reaches Jaeger.

Example strategy:

```text
Errors → 100%
Slow traces → 100%
Normal traces → 5%
```

This is particularly important for high-volume production systems.

---

# 16. Head Sampling

Head sampling makes the decision early.

```text
Request
   ↓
Sampler
 ├── Keep
 └── Drop
```

Advantages:

```text
Low overhead
Lower network usage
Simple
```

Disadvantage:

```text
The final result of the request is not yet known.
```

Therefore, an important error could be sampled out before it is known to be an error.

---

# 17. Tail Sampling

Tail sampling waits until enough information about the trace is available.

```text
Trace
 ↓
Collector
 ↓
Evaluate
 ↓
Sampling Decision
 ├── Keep
 └── Drop
```

Example:

```text
Error → Keep
Slow → Keep
Normal → Sample
```

Tail sampling is usually implemented in the Collector layer.

---

# 18. Tail Sampling Configuration

Conceptually:

```yaml
processors:
  tail_sampling:
    decision_wait: 10s
```

Policies can then determine which traces are retained.

Examples:

```text
Status code = error
Latency > threshold
Specific service
Specific environment
Probability
```

Actual policy configuration depends on the Collector distribution and version.

---

# 19. Sampling Trade-Off

Higher sampling:

```text
More traces
More visibility
More storage
More network
Higher cost
```

Lower sampling:

```text
Less storage
Lower cost
Less complete visibility
```

Production sampling should balance:

```text
Incident investigation
Cost
Trace volume
Business requirements
```

---

# 20. Jaeger Storage Configuration

Jaeger requires a trace storage backend.

Conceptually:

```text
Jaeger
   ↓
Storage Backend
```

Depending on the Jaeger version and deployment, supported storage options can include:

```text
Elasticsearch
OpenSearch
Other supported backends
```

Always verify storage compatibility with the specific Jaeger release.

---

# 21. Storage Endpoint

The Jaeger storage configuration needs the appropriate backend endpoint.

Conceptually:

```text
Jaeger
   ↓
http://elasticsearch:9200
```

In Kubernetes:

```text
Jaeger
   ↓
Elasticsearch Service
```

Do not configure Pod IP addresses directly.

---

# 22. Storage Authentication

If the storage backend requires authentication, configure credentials securely.

Bad:

```text
username: admin
password: password123
```

inside a Git repository.

Better:

```text
Secret Management
       ↓
Jaeger
       ↓
Storage
```

Possible approaches:

```text
Kubernetes Secrets
AWS Secrets Manager
External Secrets
```

depending on the environment.

---

# 23. Storage TLS

For secured storage:

```text
Jaeger
   ↓
TLS
   ↓
Elasticsearch / OpenSearch
```

Configure:

```text
CA certificate
Client certificate where required
TLS verification
Credentials
```

Do not disable TLS verification simply to make connectivity work.

---

# 24. Storage Retention

Retention determines how long trace data remains available.

Example:

```text
Normal traces
   ↓
7 days

Important traces
   ↓
Longer retention
```

Actual retention depends on:

```text
Storage capacity
Incident requirements
Compliance
Cost
```

---

# 25. Query Configuration

Jaeger Query retrieves traces from storage.

Architecture:

```text
Jaeger UI
    ↓
Jaeger Query
    ↓
Storage
```

Query configuration should consider:

```text
Storage endpoint
Authentication
TLS
Query limits
Resources
```

---

# 26. Query Resources

Production Query components should have resource requests and limits.

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi

  limits:
    cpu: "1"
    memory: 1Gi
```

These values are examples.

Actual sizing should come from workload testing.

---

# 27. UI Configuration

The Jaeger UI connects to the Query component.

Conceptually:

```text
Browser
   ↓
Jaeger UI
   ↓
Jaeger Query
   ↓
Storage
```

The UI should normally remain inside a controlled network boundary.

---

# 28. UI Exposure

For development:

```text
kubectl port-forward
```

can provide temporary access.

For production:

```text
Engineer
   ↓
VPN / Secure Access
   ↓
Ingress / Proxy
   ↓
Jaeger UI
```

Avoid directly exposing the UI to the public internet.

---

# 29. Kubernetes ConfigMaps

Non-sensitive configuration can be stored in a ConfigMap.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: jaeger-config
  namespace: observability
data:
  environment: production
```

Do not put passwords or tokens into ConfigMaps.

---

# 30. Kubernetes Secrets

Sensitive configuration belongs in Secrets or an external secret-management system.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: jaeger-storage
  namespace: observability
type: Opaque
```

Use appropriate encryption and access controls.

---

# 31. Environment Variables

Configuration can also be passed using environment variables.

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: orders
```

Sensitive values should come from Secrets:

```yaml
env:
  - name: STORAGE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: jaeger-storage
        key: password
```

---

# 32. Helm Values

In Kubernetes, Helm values are commonly used to manage configuration.

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
```

Production values should be version controlled.

Example:

```text
Git
 ↓
values.yaml
 ↓
Pull Request
 ↓
Review
 ↓
ArgoCD
 ↓
EKS
```

---

# 33. Configuration Separation

Separate environments:

```text
values-dev.yaml
values-staging.yaml
values-prod.yaml
```

Example:

```text
Development
 ↓
Lower resources
Higher sampling

Production
 ↓
Higher resources
Controlled sampling
Production storage
```

This prevents development configuration from accidentally being used in production.

---

# 34. Environment-Specific Endpoints

Development:

```text
otel-gateway.observability.svc
```

Production:

```text
otel-gateway.observability.svc
```

The Service name can remain consistent while the underlying deployment differs.

This makes application configuration more portable.

---

# 35. Resource Attributes

Configure consistent resource attributes:

```text
service.name
service.version
deployment.environment
cloud.provider
cloud.region
k8s.cluster.name
```

Example:

```text
service.name=payment
service.version=2.4.1
deployment.environment=production
cloud.provider=aws
cloud.region=ap-south-1
```

These attributes improve trace filtering and investigation.

---

# 36. Service Naming Standard

Use a standardized naming strategy.

Good:

```text
orders
payment
inventory
notification
```

Avoid inconsistent naming:

```text
orders-service
OrdersService
orders_prod
orders-production-service
```

unless the organization deliberately defines those names.

---

# 37. Trace Propagation Configuration

Distributed tracing depends on context propagation.

Common propagation format:

```text
W3C Trace Context
```

Headers:

```text
traceparent
tracestate
```

Flow:

```text
Orders
  ↓
traceparent
  ↓
Payment
  ↓
Inventory
```

All services should use compatible propagation settings.

---

# 38. Propagation Failure

If propagation is broken:

```text
Orders
Trace A
   ↓
Payment
Trace B
```

Instead of:

```text
Orders
Trace A
   ↓
Payment
Trace A
```

The result is fragmented traces.

Check:

```text
Instrumentation
HTTP headers
Messaging headers
Context extraction
Context injection
```

---

# 39. HTTP Configuration

For HTTP services, instrumentation should capture useful information such as:

```text
HTTP method
Route
Status
Duration
Service
```

Avoid automatically collecting sensitive request bodies unless there is a clear requirement.

---

# 40. Database Configuration

Database spans can contain useful information:

```text
Database system
Operation
Duration
```

Example:

```text
db.system=postgresql
db.operation=SELECT
```

Avoid capturing sensitive SQL values or credentials.

---

# 41. External API Configuration

External calls should be represented as spans.

Example:

```text
Payment
   ↓
Payment Gateway
```

The trace can then show:

```text
Payment Gateway
Duration = 1.4s
Status = timeout
```

This helps distinguish internal and external latency.

---

# 42. Messaging Configuration

Distributed tracing should propagate context through messaging systems.

Example:

```text
Orders
   ↓
RabbitMQ
   ↓
Notification
```

Trace context should travel with the message where supported by the instrumentation.

---

# 43. Log Correlation

Configure application logging so that logs can include:

```text
trace_id
span_id
```

Then:

```text
Jaeger
   ↓
Trace ID
   ↓
ELK
   ↓
Related logs
```

This creates strong correlation between traces and logs.

---

# 44. Metrics Correlation

Prometheus can monitor tracing infrastructure.

Example metrics:

```text
Collector CPU
Collector memory
Spans received
Spans exported
Export failures
Queue size
```

Grafana can visualize these metrics.

```text
Prometheus
   ↓
Grafana
```

---

# 45. Collector Pipeline

A production pipeline commonly looks like:

```text
OTLP Receiver
     ↓
Memory Limiter
     ↓
Kubernetes Attributes
     ↓
Tail Sampling
     ↓
Batch
     ↓
Jaeger Export
```

Each processor has a purpose.

---

# 46. Processor Ordering

Processor order matters.

A conceptual order:

```text
Receive
  ↓
Memory Protection
  ↓
Enrichment
  ↓
Sampling
  ↓
Batching
  ↓
Export
```

The exact ordering should be tested according to the desired behavior.

---

# 47. Filtering

Filter unnecessary telemetry before sending it to Jaeger.

Examples:

```text
Health checks
Internal noise
Known low-value endpoints
Development-only operations
```

Example:

```text
/health
/readiness
/liveness
```

If these generate huge volumes of traces, filtering can reduce unnecessary telemetry.

---

# 48. Attribute Filtering

Remove sensitive or unnecessary attributes.

Example:

```text
Authorization
Cookie
Password
Token
```

should not be blindly propagated into tracing data.

A production tracing system should explicitly define what attributes are allowed.

---

# 49. High Cardinality

Avoid uncontrolled attributes:

```text
user_id
session_id
request_id
random UUID
```

These can increase storage and query costs.

Prefer controlled dimensions:

```text
service
route
method
status
environment
version
```

---

# 50. Sampling Configuration Strategy

A production policy might be:

```text
production:
  errors = 100%
  slow traces = 100%
  normal traces = 5%

staging:
  higher sampling

development:
  high or full sampling
```

This provides better visibility where needed without storing every production trace.

---

# 51. Queue Configuration

When the backend is temporarily unavailable:

```text
Collector
   ↓
Queue
   ↓
Jaeger
```

The queue should be bounded.

Bad:

```text
Unlimited queue
   ↓
Memory exhaustion
```

Good:

```text
Bounded queue
   ↓
Retry
   ↓
Backpressure
```

---

# 52. Retry Configuration

Transient backend failures may be retried.

Conceptually:

```text
Collector
   ↓
Jaeger
   X
   ↓
Retry
   ↓
Jaeger
   ✓
```

Configure:

```text
Initial interval
Maximum interval
Maximum elapsed time
```

according to the workload.

---

# 53. Timeout Configuration

Set appropriate timeouts.

Without timeouts:

```text
Backend unavailable
   ↓
Requests wait indefinitely
   ↓
Resources consumed
```

With timeouts:

```text
Backend unavailable
   ↓
Timeout
   ↓
Retry / Queue / Drop
```

---

# 54. Compression

For high-volume telemetry, compression can reduce network usage.

Conceptually:

```text
Collector
   ↓
Compressed telemetry
   ↓
Jaeger
```

Evaluate:

```text
CPU overhead
Network savings
Backend compatibility
```

before enabling it broadly.

---

# 55. TLS Configuration

For secured communication:

```text
Collector
   ↓
TLS
   ↓
Jaeger
```

Configure:

```text
CA
Certificate
Private key
Verification
```

Store certificates and keys securely.

---

# 56. Authentication

Depending on the architecture, authentication can use:

```text
Bearer tokens
API credentials
mTLS
Proxy authentication
Cloud-specific identity
```

Never hard-code credentials into application images.

---

# 57. NetworkPolicy

A possible traffic model:

```text
Application
    ↓
OTel Agent
    ↓
OTel Gateway
    ↓
Jaeger
    ↓
Storage
```

NetworkPolicy should allow only required communication paths.

For example:

```text
Application → Collector
Collector → Jaeger
Jaeger → Storage
Query → Storage
```

---

# 58. RBAC

Use least privilege.

```text
Collector
   ↓
Required Kubernetes permissions only
```

If Kubernetes metadata is required, grant only the resources and verbs needed.

Avoid:

```text
cluster-admin
```

unless there is a verified requirement.

---

# 59. Resource Requests and Limits

Jaeger and Collector components need resource controls.

Example:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 512Mi

  limits:
    cpu: "2"
    memory: 2Gi
```

These are examples, not universal production values.

Sizing must be based on:

```text
Spans/sec
Sampling
Query rate
Storage
```

---

# 60. Horizontal Scaling

Scale components according to workload.

Example:

```text
Trace volume ↑
      ↓
Gateway CPU ↑
      ↓
Add Gateway replicas
```

Query:

```text
Query requests ↑
      ↓
Query latency ↑
      ↓
Scale Query replicas
```

Storage scaling must be handled independently.

---

# 61. Pod Distribution

Spread replicas across nodes and Availability Zones.

Example:

```text
AZ-A → Gateway-1
AZ-B → Gateway-2
AZ-C → Gateway-3
```

Use:

```text
Pod anti-affinity
Topology spread constraints
```

where appropriate.

---

# 62. PodDisruptionBudget

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: jaeger-query
  namespace: observability
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: jaeger-query
```

This helps maintain availability during voluntary disruptions.

---

# 63. Health Probes

Use:

```text
Startup probe
Readiness probe
Liveness probe
```

Readiness:

```text
Pod ready
   ↓
Receive traffic
```

Not ready:

```text
Pod not ready
   ↓
Do not send traffic
```

This is important during rolling updates.

---

# 64. Configuration Validation

Before applying a configuration:

```text
1. Validate YAML
2. Validate Helm values
3. Validate Collector configuration
4. Validate endpoints
5. Validate credentials
6. Test in staging
```

Do not introduce configuration changes directly into production without validation.

---

# 65. GitOps Configuration

Production configuration should be stored in Git.

Example:

```text
observability/
├── jaeger/
│   ├── values-prod.yaml
│   └── application.yaml
│
└── otel/
    ├── collector-prod.yaml
    └── values-prod.yaml
```

Workflow:

```text
Git
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

---

# 66. Configuration Drift

If someone changes production manually:

```text
kubectl edit
```

Git may no longer match the cluster.

GitOps can detect this drift.

```text
Git
  ≠
Cluster
  ↓
ArgoCD detects drift
```

The preferred production approach is to change configuration through Git.

---

# 67. Configuration Change Process

Use:

```text
1. Modify configuration
2. Commit
3. Create Pull Request
4. Review
5. Validate
6. Merge
7. ArgoCD sync
8. Monitor
```

After deployment verify:

```text
Traces
Collector health
Jaeger health
Storage
Query
```

---

# 68. Staging Configuration

Before production:

```text
Development
     ↓
Staging
     ↓
Production
```

Test:

```text
Trace generation
Sampling
Storage
Query performance
Security
Resource consumption
Failure handling
```

---

# 69. Canary Configuration

For major changes:

```text
Existing configuration
        +
Canary configuration
```

Send controlled traffic to the new configuration.

Monitor:

```text
Export errors
Dropped spans
Latency
CPU
Memory
Storage
```

Then expand gradually.

---

# 70. Configuration Rollback

If a configuration causes problems:

```text
New Configuration
       ↓
Problem
       ↓
Previous Git Commit
       ↓
ArgoCD
       ↓
Previous Configuration
```

This is one of the major benefits of GitOps.

---

# 71. Production Configuration Example

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

# 72. Production Configuration Flow

```text
                    Application
                         │
                         ↓
                       OTLP
                         │
                         ↓
                  OTel Collector
                         │
                ┌────────┼────────┐
                ↓        ↓        ↓
             Memory   Enrich   Sampling
             Limit              │
                └────────┬──────┘
                         ↓
                       Batch
                         ↓
                    Jaeger Export
                         ↓
                       Jaeger
                         ↓
                      Storage
```

---

# 73. Configuration Monitoring

Monitor configuration-related failures.

Examples:

```text
Invalid exporter
TLS failure
Authentication failure
Storage unavailable
Receiver failure
Pipeline failure
```

Collector logs should be inspected after every significant configuration change.

---

# 74. Troubleshooting Configuration

### No traces

Check:

```text
OTEL_EXPORTER_OTLP_ENDPOINT
OTLP receiver
Collector pipeline
Sampling
Exporter
Jaeger
Storage
```

### Partial traces

Check:

```text
Trace propagation
Sampling
Instrumentation
```

### High memory

Check:

```text
Queue
Sampling
Trace volume
Batch size
Backend availability
```

### Jaeger UI empty

Check:

```text
Query
Storage
Service selection
Time range
Trace ingestion
```

---

# 75. Configuration Troubleshooting Commands

Check Pods:

```bash
kubectl get pods -n observability
```

Check Services:

```bash
kubectl get svc -n observability
```

Check configuration:

```bash
kubectl get configmap -n observability
```

Check Secrets:

```bash
kubectl get secrets -n observability
```

Check events:

```bash
kubectl get events -n observability --sort-by=.lastTimestamp
```

Check logs:

```bash
kubectl logs <pod> -n observability
```

---

# 76. Verify Collector Configuration

After changing the Collector configuration:

```text
1. Apply configuration
2. Verify Pod readiness
3. Check Collector logs
4. Generate test traffic
5. Verify traces in Jaeger
6. Check exported span metrics
7. Monitor resource usage
```

Do not stop after confirming that the Pod is merely `Running`.

---

# 77. Verify Jaeger Configuration

Check:

```text
Jaeger Pods
Jaeger Services
Storage connectivity
Query health
UI access
```

Then generate traces.

A configuration is not considered successful until the complete flow works:

```text
Application
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

---

# 78. Production Security Checklist

```text
[ ] TLS configured where required
[ ] Authentication configured
[ ] Authorization configured
[ ] Secrets protected
[ ] Jaeger UI access restricted
[ ] NetworkPolicy configured
[ ] Security Groups configured where applicable
[ ] Sensitive headers filtered
[ ] Sensitive request data excluded
[ ] High-cardinality attributes reviewed
[ ] RBAC follows least privilege
```

---

# 79. Production Performance Checklist

```text
[ ] Sampling configured
[ ] Batch processor configured
[ ] Memory limiter configured
[ ] Queues bounded
[ ] Retry configured
[ ] Timeouts configured
[ ] Resource requests configured
[ ] Resource limits configured
[ ] Storage capacity planned
[ ] Query workload measured
[ ] High-cardinality attributes controlled
```

---

# 80. Production Configuration Checklist

```text
APPLICATION
[ ] service.name
[ ] service.version
[ ] deployment.environment
[ ] OTLP endpoint
[ ] Trace propagation

COLLECTOR
[ ] OTLP receiver
[ ] Memory limiter
[ ] Kubernetes attributes
[ ] Sampling
[ ] Batch
[ ] Exporter
[ ] Retry
[ ] Queue
[ ] TLS

JAEGER
[ ] Ingestion
[ ] Storage
[ ] Query
[ ] UI
[ ] Resource configuration
[ ] Health probes

STORAGE
[ ] Endpoint
[ ] Authentication
[ ] TLS
[ ] Retention
[ ] Capacity
[ ] Backup

KUBERNETES
[ ] Service
[ ] ConfigMap
[ ] Secrets
[ ] RBAC
[ ] NetworkPolicy
[ ] PDB
[ ] Pod distribution

OPERATIONS
[ ] Monitoring
[ ] Alerts
[ ] GitOps
[ ] Staging validation
[ ] Rollback
[ ] Upgrade strategy
```

---

# 81. Real-World Configuration Example

Consider a production EKS Payment service.

Application:

```text
service.name = payment
service.version = 2.4.0
environment = production
```

Flow:

```text
Payment
   ↓
OTel SDK
   ↓
OTLP
   ↓
OTel Agent
   ↓
OTel Gateway
   ↓
Tail Sampling
   ↓
Jaeger
```

Important traces:

```text
Payment error → KEEP
Payment > 1s → KEEP
Normal Payment → SAMPLE
```

Then:

```text
Jaeger
   ↓
Storage
   ↓
Jaeger Query
   ↓
Jaeger UI
```

---

# 82. Real-World Deployment Regression

Before deployment:

```text
payment v2.3.0
p95 = 300ms
```

After deployment:

```text
payment v2.4.0
p95 = 1.2s
```

Jaeger trace:

```text
Payment
   ↓
Database
   ↓
950ms
```

Kibana:

```text
Slow database query
```

Configuration and deployment metadata identify:

```text
service.version = 2.4.0
```

The team can then investigate or rollback the release.

---

# 83. Final Mental Model

Remember Jaeger configuration as:

```text
APPLICATION
    ↓
Service Name
Version
Environment
OTLP Endpoint
Propagation
    ↓
COLLECTOR
    ↓
Receiver
    ↓
Memory Limiter
    ↓
Enrichment
    ↓
Sampling
    ↓
Batch
    ↓
Exporter
    ↓
JAEGER
    ↓
Storage
    ↓
Query
    ↓
UI
```

The most important configuration principles are:

**Use consistent service and environment metadata, send telemetry through OTLP, use the OpenTelemetry Collector for processing and routing, configure memory protection and batching, apply intentional sampling, secure storage and UI access, protect sensitive trace data, control cardinality, use Kubernetes Services instead of Pod IPs, store production configuration in Git, deploy through GitOps, and validate every configuration change through the complete Application → Collector → Jaeger → Storage → Query → UI path.**
