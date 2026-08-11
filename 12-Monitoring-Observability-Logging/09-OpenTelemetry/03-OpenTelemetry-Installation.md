# OpenTelemetry Installation

## 1. Overview

Installing OpenTelemetry depends on where telemetry is generated and where the OpenTelemetry Collector is deployed.

A production environment typically has two installation layers:

```text
Application
     ↓
OpenTelemetry SDK / Auto-Instrumentation
     ↓
OpenTelemetry Collector
     ↓
Observability Backends
```

The application layer installs the appropriate OpenTelemetry SDK or auto-instrumentation mechanism.

The infrastructure layer installs the OpenTelemetry Collector.

---

# 2. Installation Components

The main components that may need to be installed are:

```text
OpenTelemetry SDK
OpenTelemetry Auto-Instrumentation
OpenTelemetry Collector
OpenTelemetry Collector Contrib / Distribution
```

The exact installation method depends on:

```text
Application language
Operating system
Container platform
Kubernetes
Deployment architecture
Observability backend
```

---

# 3. Installation Architecture

A typical Kubernetes environment:

```text
                         EKS Cluster
                              │
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
         Application      Application      Application
             │                │                │
          OTel SDK         OTel SDK         OTel SDK
             │                │                │
             └────────────────┼────────────────┘
                              ↓
                       OTel Collector
                              ↓
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
             Prometheus       ELK         Jaeger
```

---

# 4. Installation Strategy

Before installing anything, determine:

```text
Where will telemetry be generated?
Where will the Collector run?
Which signals are required?
Where will telemetry be stored?
How much telemetry will be generated?
Is high availability required?
```

For example:

```text
Signals:
Metrics + Logs + Traces

Application:
Java / Node.js / Python

Platform:
AWS EKS

Collector:
Kubernetes

Backends:
Prometheus + Elasticsearch + Jaeger
```

---

# 5. OpenTelemetry SDK Installation

SDK installation is language-specific.

Common application languages include:

```text
Java
Python
Node.js
.NET
Go
JavaScript
PHP
Ruby
C++
```

The general process is:

```text
Select language
     ↓
Install OTel API / SDK
     ↓
Install instrumentation
     ↓
Configure service
     ↓
Configure exporter
     ↓
Send telemetry
```

---

# 6. Java Application Installation

Java applications can use OpenTelemetry SDKs or the OpenTelemetry Java Agent.

For many existing Java applications, the Java Agent is a practical starting point because it can provide automatic instrumentation with minimal application-code changes.

Architecture:

```text
Java Application
       +
OTel Java Agent
       ↓
Telemetry
       ↓
OTel Collector
```

---

# 7. Java Agent Installation

Download the OpenTelemetry Java Agent from the official OpenTelemetry distribution.

The application is then started with the Java agent:

```bash
java -javaagent:/opt/opentelemetry/opentelemetry-javaagent.jar \
     -jar application.jar
```

The important concept is:

```text
-javaagent
```

The agent is loaded into the JVM before the application starts.

---

# 8. Java Agent Configuration

Configuration can be provided using environment variables.

Example:

```bash
export OTEL_SERVICE_NAME=payment
export OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

Then:

```bash
java -javaagent:/opt/opentelemetry/opentelemetry-javaagent.jar \
     -jar application.jar
```

The exact exporter protocol and endpoint should match the Collector configuration.

---

# 9. Java Kubernetes Deployment

A Kubernetes Deployment can provide the agent to the application.

Conceptually:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: "payment"

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: "http://otel-collector:4317"
```

The Java process must also load the agent.

A common production approach is to package the agent with the application image or inject it through an approved Kubernetes instrumentation mechanism.

---

# 10. Node.js Installation

Node.js applications can use the OpenTelemetry Node.js SDK and supported instrumentation packages.

General flow:

```text
Node.js Application
        ↓
OTel Node SDK
        ↓
Instrumentation
        ↓
OTLP
        ↓
Collector
```

The required packages depend on the signals and libraries being instrumented.

---

# 11. Node.js Initialization

The OpenTelemetry SDK should be initialized before the application loads the libraries that need instrumentation.

Conceptually:

```javascript
initializeOpenTelemetry();

startApplication();
```

The important principle is:

```text
Initialize instrumentation
        ↓
Load application
```

If instrumentation is initialized too late, some libraries may not be instrumented correctly.

---

# 12. Node.js Configuration

Typical configuration includes:

```text
OTEL_SERVICE_NAME
OTEL_EXPORTER_OTLP_ENDPOINT
OTEL_RESOURCE_ATTRIBUTES
```

Example:

```bash
export OTEL_SERVICE_NAME=orders
export OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
```

---

# 13. Python Installation

Python applications can use OpenTelemetry Python packages.

General architecture:

```text
Python Application
       ↓
OTel SDK
       ↓
Instrumentation
       ↓
OTLP
       ↓
Collector
```

Python supports both SDK-based instrumentation and automatic instrumentation approaches.

---

# 14. Python Automatic Instrumentation

The OpenTelemetry Python ecosystem provides an auto-instrumentation approach for supported libraries.

Conceptually:

```text
Python Application
       ↓
OTel Auto-Instrumentation
       ↓
HTTP / DB / Framework telemetry
       ↓
Collector
```

This can reduce the amount of manual instrumentation required.

---

# 15. Python Manual Instrumentation

Manual instrumentation can be used for application-specific operations.

Conceptually:

```text
Business Operation
       ↓
Create Span
       ↓
Add Attributes
       ↓
Record Error
       ↓
End Span
```

Use manual instrumentation when automatic instrumentation does not provide sufficient business context.

---

# 16. Installing the Collector

The OpenTelemetry Collector can be installed in several environments:

```text
Linux
Docker
Kubernetes
VMs
Cloud infrastructure
```

For production Kubernetes environments, Kubernetes deployment is often the preferred model.

---

# 17. Collector Deployment Options

The Collector can run as:

```text
DaemonSet
Deployment
Gateway
Sidecar
```

Each model has a different purpose.

### DaemonSet

```text
One Collector per node
```

### Deployment

```text
Multiple centralized Collector replicas
```

### Gateway

```text
Centralized telemetry processing
```

### Sidecar

```text
Collector alongside an application
```

---

# 18. Kubernetes Installation

A production EKS architecture can use the OpenTelemetry Operator.

Conceptually:

```text
Kubernetes
     ↓
OpenTelemetry Operator
     ↓
OpenTelemetry Collector
     ↓
Applications
```

The Operator simplifies deployment and lifecycle management of Collector resources and can also support instrumentation workflows.

---

# 19. Install OpenTelemetry Operator

A common Helm-based approach is:

```bash
helm repo add open-telemetry \
  https://open-telemetry.github.io/opentelemetry-helm-charts

helm repo update
```

Then install the Operator using the current chart and version selected by the organization.

The exact chart values should be pinned in production rather than relying on unreviewed defaults.

---

# 20. Create Observability Namespace

Create a dedicated namespace:

```bash
kubectl create namespace observability
```

Verify:

```bash
kubectl get namespaces
```

Expected conceptually:

```text
observability
```

---

# 21. Verify Operator

Check:

```bash
kubectl get pods -n observability
```

You should see the Operator components running.

For more details:

```bash
kubectl get all -n observability
```

---

# 22. Why Use a Dedicated Namespace

A dedicated namespace provides logical separation:

```text
observability
│
├── OTel Operator
├── OTel Collector
├── OTel Gateway
└── Supporting resources
```

It simplifies:

```text
Access control
Resource management
Troubleshooting
Monitoring
Configuration management
```

---

# 23. Collector Custom Resource

With the OpenTelemetry Operator, the Collector can be defined using an `OpenTelemetryCollector` custom resource.

Conceptually:

```yaml
apiVersion: opentelemetry.io/v1beta1
kind: OpenTelemetryCollector
metadata:
  name: otel-gateway
  namespace: observability
spec:
  mode: deployment
```

The exact API version and supported fields should be checked against the installed Operator version.

---

# 24. Collector Configuration

A basic Collector configuration contains:

```text
receivers
processors
exporters
service
pipelines
```

Example:

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
  debug:

service:
  pipelines:
    traces:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - debug
```

This is a conceptual starting configuration for validating Collector behavior.

---

# 25. Receiver Configuration

The OTLP receiver can expose:

```text
OTLP/gRPC
OTLP/HTTP
```

Conceptually:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:
```

The application exporter must use a compatible protocol and endpoint.

---

# 26. Processor Configuration

Production pipelines commonly include:

```text
Memory Limiter
Batch
Resource
Attributes
Filter
Sampling
```

Example:

```yaml
processors:
  memory_limiter:
    limit_percentage: 80
    spike_limit_percentage: 15

  batch:
```

The exact values must be tuned according to workload and available resources.

---

# 27. Exporter Configuration

The exporter depends on the backend.

For example:

```text
Prometheus-compatible backend
Elasticsearch
Jaeger
OTLP backend
```

Conceptually:

```yaml
exporters:
  otlp:
    endpoint: backend:4317
```

The exact endpoint and security configuration depend on the backend.

---

# 28. Pipeline Configuration

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

The flow is:

```text
Application
    ↓
OTLP
    ↓
Receiver
    ↓
Memory Limiter
    ↓
Batch
    ↓
Exporter
    ↓
Backend
```

---

# 29. Metrics Pipeline

Conceptually:

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

The exact exporter configuration depends on the chosen Prometheus architecture.

---

# 30. Logs Pipeline

Conceptually:

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

The exporter could send telemetry toward an intermediate processor such as Logstash or directly to a compatible backend, depending on the architecture.

---

# 31. Apply Collector Configuration

Once the Collector resource is defined:

```bash
kubectl apply -f otel-collector.yaml
```

Then check:

```bash
kubectl get opentelemetrycollectors -n observability
```

Check Pods:

```bash
kubectl get pods -n observability
```

---

# 32. Verify Collector Service

Check Services:

```bash
kubectl get svc -n observability
```

You may see a service such as:

```text
otel-gateway
```

Applications or node-level Collectors can send telemetry to this stable Kubernetes Service.

---

# 33. Verify Collector Logs

Check Collector logs:

```bash
kubectl logs -n observability \
  deployment/otel-gateway
```

Depending on the deployment mode, the resource name may differ.

Look for:

```text
Receiver started
Pipeline started
Exporter started
Configuration errors
Connection failures
```

---

# 34. Collector Health

The Collector should expose health information.

A health-check extension can be configured conceptually:

```yaml
extensions:
  health_check:
```

Then:

```text
Kubernetes
    ↓
Health Check
    ↓
Collector
```

This can be used for operational monitoring and Kubernetes health probes.

---

# 35. Collector Metrics

The Collector can expose its own metrics.

These metrics can be scraped by Prometheus.

Conceptually:

```text
OTel Collector
      ↓
Collector Metrics
      ↓
Prometheus
      ↓
Grafana
```

Useful metrics include:

```text
Received telemetry
Exported telemetry
Dropped telemetry
Export failures
Queue size
Processing activity
```

---

# 36. Installing OTel in EKS

A production EKS deployment can look like:

```text
AWS
│
└── EKS
    │
    ├── Node-01
    │   ├── Applications
    │   └── OTel Agent
    │
    ├── Node-02
    │   ├── Applications
    │   └── OTel Agent
    │
    └── Node-03
        ├── Applications
        └── OTel Agent
```

Then:

```text
OTel Agents
      ↓
OTel Gateway
      ↓
Backends
```

---

# 37. Agent Installation

Deploy the Collector as a DaemonSet when node-local collection is required.

Conceptually:

```yaml
spec:
  mode: daemonset
```

Architecture:

```text
Node-01 → OTel Agent
Node-02 → OTel Agent
Node-03 → OTel Agent
```

This allows telemetry to be collected close to workloads.

---

# 38. Gateway Installation

Deploy a separate Collector as a Deployment:

```yaml
spec:
  mode: deployment
```

Architecture:

```text
                    OTel Gateway
                         │
              ┌──────────┼──────────┐
              ↓          ↓          ↓
          Gateway-01 Gateway-02 Gateway-03
```

The Kubernetes Service provides a stable endpoint.

---

# 39. Agent + Gateway Installation

A production installation can deploy:

```text
DaemonSet
   ↓
OTel Agent

Deployment
   ↓
OTel Gateway
```

Architecture:

```text
Node-01 → Agent ─┐
Node-02 → Agent ─┼→ Gateway → Backend
Node-03 → Agent ─┘
```

This separates local collection from centralized processing.

---

# 40. Application Connection

Applications should send telemetry to a stable Collector endpoint.

Example:

```text
otel-gateway.observability.svc.cluster.local:4317
```

Conceptually:

```text
Application
     ↓
Kubernetes Service
     ↓
OTel Gateway
```

Avoid hardcoding individual Collector Pod IP addresses.

---

# 41. DNS-Based Service Discovery

Kubernetes Service DNS provides stable addressing:

```text
otel-gateway.observability.svc.cluster.local
```

The application does not need to know:

```text
Gateway Pod IP
Gateway Pod Name
Gateway Node
```

This improves resilience when Pods are replaced.

---

# 42. Application Environment Variables

OpenTelemetry supports standard environment variables.

Common examples include:

```bash
OTEL_SERVICE_NAME
OTEL_EXPORTER_OTLP_ENDPOINT
OTEL_EXPORTER_OTLP_PROTOCOL
OTEL_RESOURCE_ATTRIBUTES
```

Example:

```bash
OTEL_SERVICE_NAME=payment
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-gateway.observability.svc.cluster.local:4317
```

---

# 43. Kubernetes Application Configuration

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: "payment"

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: "http://otel-gateway.observability.svc.cluster.local:4317"
```

For production, use TLS and authentication when required by the environment.

---

# 44. Java Application With Kubernetes

A Java application might use:

```text
payment Deployment
       ↓
OTel Java Agent
       ↓
OTLP
       ↓
OTel Gateway
```

Conceptually:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: "payment"

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: "http://otel-gateway.observability.svc.cluster.local:4317"
```

The agent can then generate telemetry from supported application frameworks.

---

# 45. Node.js Application With Kubernetes

Architecture:

```text
orders Deployment
       ↓
OTel Node SDK
       ↓
OTLP
       ↓
OTel Gateway
```

The Node.js process should initialize OpenTelemetry before the instrumented libraries are loaded.

---

# 46. Python Application With Kubernetes

Architecture:

```text
inventory Deployment
       ↓
OTel Python
       ↓
OTLP
       ↓
OTel Gateway
```

Configuration can be provided through environment variables and application startup configuration.

---

# 47. Collector-to-Collector Communication

In an agent + gateway architecture:

```text
Agent
  ↓
OTLP
  ↓
Gateway
```

Example:

```text
OTel Agent
    ↓
otel-gateway:4317
```

This keeps the gateway endpoint stable while allowing the gateway Pods to scale.

---

# 48. TLS Installation

Production telemetry should generally use encrypted transport where required.

Architecture:

```text
Application
     ↓ TLS
OTel Agent
     ↓ TLS
OTel Gateway
     ↓ TLS
Backend
```

Certificates should be managed through an appropriate certificate-management and secret-management process.

---

# 49. Authentication Installation

Where authentication is required:

```text
Application
     ↓
Authenticated OTLP
     ↓
Collector
```

Similarly:

```text
Collector
     ↓
Authenticated Export
     ↓
Backend
```

Do not rely only on network isolation when the backend requires application-level authentication.

---

# 50. Secret Management

Do not store credentials directly in Git.

Bad:

```yaml
password: super-secret-password
```

Preferred:

```text
Secret Manager
      ↓
Kubernetes Secret
      ↓
Collector
```

For AWS environments, an approved AWS secret-management solution can be used.

---

# 51. OpenTelemetry Operator and Instrumentation

The OpenTelemetry Operator can also support Kubernetes instrumentation workflows.

Conceptually:

```text
OpenTelemetry Operator
       ↓
Instrumentation Resource
       ↓
Application
       ↓
Auto-Instrumentation
```

This can reduce manual changes to individual application deployments.

---

# 52. Instrumentation Resource

Conceptually:

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: default
  namespace: observability
spec:
  exporter:
    endpoint: http://otel-gateway:4317
```

The exact API version and fields must match the installed Operator version.

---

# 53. Auto-Instrumentation Architecture

```text
                         Operator
                            │
                            ↓
                   Instrumentation Config
                            │
                            ↓
                     Application Pod
                            │
                            ↓
                    Auto-Instrumentation
                            │
                            ↓
                       OTel Collector
```

This is useful for standardizing instrumentation across many workloads.

---

# 54. Production Instrumentation Strategy

For a microservices platform:

```text
User
   → Automatic instrumentation

Orders
   → Automatic + manual

Payment
   → Automatic + manual

Inventory
   → Automatic + manual
```

Use automatic instrumentation for common technical operations and manual instrumentation for important business operations.

---

# 55. Verify Application Telemetry

After instrumentation:

```text
Application
     ↓
OTel SDK
     ↓
Collector
```

Verify Collector logs and metrics.

Then verify the backend:

```text
Collector
     ↓
Backend
     ↓
Search / Query
```

For traces:

```text
Jaeger
```

For logs:

```text
Kibana
```

For metrics:

```text
Prometheus / Grafana
```

---

# 56. Verify Traces

A trace validation workflow:

```text
Deploy application
       ↓
Generate request
       ↓
Application creates span
       ↓
Collector receives span
       ↓
Collector exports span
       ↓
Jaeger receives span
       ↓
Search trace
```

Check:

```text
Trace ID
Service name
Span name
Duration
Status
```

---

# 57. Verify Metrics

Workflow:

```text
Generate traffic
      ↓
Application metrics
      ↓
Collector
      ↓
Prometheus
      ↓
Grafana
```

Check:

```text
Request count
Latency
Error rate
```

---

# 58. Verify Logs

Workflow:

```text
Generate application event
       ↓
Application log
       ↓
Collector
       ↓
Log processing
       ↓
Elasticsearch
       ↓
Kibana
```

Search using:

```text
service.name
environment
log.level
```

---

# 59. End-to-End Installation Test

Use a simple test service.

```text
Test Application
      ↓
OTel SDK
      ↓
OTel Collector
      ↓
Backend
```

Generate traffic:

```bash
curl http://application.example
```

Then verify:

```text
Metrics → Prometheus
Logs → Elasticsearch / Kibana
Traces → Jaeger
```

This confirms the complete telemetry path.

---

# 60. Collector Connectivity Test

From a Pod:

```bash
kubectl exec -it <pod> -n <namespace> -- \
  sh
```

Then verify DNS:

```bash
nslookup otel-gateway.observability.svc.cluster.local
```

or:

```bash
getent hosts otel-gateway.observability.svc.cluster.local
```

The exact diagnostic command depends on the container image.

---

# 61. Check Collector Services

```bash
kubectl get svc -n observability
```

Example:

```text
NAME
otel-agent
otel-gateway
```

Then inspect:

```bash
kubectl describe svc otel-gateway -n observability
```

Verify:

```text
Endpoints
Ports
Selector
```

---

# 62. Check Collector Pods

```bash
kubectl get pods -n observability -o wide
```

Check:

```text
READY
STATUS
RESTARTS
NODE
```

A production Collector should not continuously restart.

---

# 63. Check Collector Events

```bash
kubectl get events -n observability --sort-by=.lastTimestamp
```

Look for:

```text
ImagePullBackOff
CrashLoopBackOff
FailedScheduling
Unhealthy
OOMKilled
```

---

# 64. Check Collector Logs

```bash
kubectl logs <collector-pod> -n observability
```

Look for:

```text
Configuration error
Receiver failure
Exporter failure
TLS error
Authentication failure
Connection refused
```

---

# 65. Debugging Configuration

If the Collector does not start:

```text
Collector Pod
     ↓
Configuration validation
     ↓
Check receiver
     ↓
Check processor
     ↓
Check exporter
     ↓
Check pipeline references
```

Common configuration mistake:

```text
Receiver defined
```

but not referenced by:

```text
service.pipelines
```

---

# 66. Receiver Configuration Error

Example:

```yaml
receivers:
  otlp:
```

but the pipeline does not contain:

```yaml
receivers:
  - otlp
```

The receiver exists but is not active in the required pipeline.

Always verify:

```text
Receiver
   ↓
Pipeline
   ↓
Processor
   ↓
Exporter
```

---

# 67. Exporter Configuration Error

A common issue is:

```text
Exporter defined
```

but not referenced in the pipeline.

Example:

```yaml
exporters:
  otlp:
```

The pipeline must reference the exporter:

```yaml
exporters:
  - otlp
```

---

# 68. Processor Configuration Error

A processor may be defined but unused.

Example:

```yaml
processors:
  batch:
```

The pipeline must reference it:

```yaml
processors:
  - batch
```

This distinction is important when troubleshooting Collector configuration.

---

# 69. Installation With Helm

Helm can simplify deployment.

Add repository:

```bash
helm repo add open-telemetry \
  https://open-telemetry.github.io/opentelemetry-helm-charts
```

Update:

```bash
helm repo update
```

Search available charts:

```bash
helm search repo open-telemetry
```

Then install the selected chart and pinned version according to the production deployment design.

---

# 70. Helm Values

Store production configuration in Git.

Example:

```text
otel/
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
└── values-production.yaml
```

This allows environment-specific configuration without manually changing production Pods.

---

# 71. Helm Installation Flow

```text
values-production.yaml
        ↓
Helm
        ↓
Kubernetes
        ↓
OTel Collector
```

GitOps can then manage the Helm release:

```text
Git
 ↓
ArgoCD
 ↓
Helm
 ↓
EKS
```

---

# 72. GitOps Installation

For your environment:

```text
GitHub
   ↓
OpenTelemetry Helm / Manifests
   ↓
GitHub Actions
   ↓
Validation
   ↓
ArgoCD
   ↓
EKS
   ↓
OTel Collector
```

This is preferable to manually applying configuration repeatedly in production.

---

# 73. ArgoCD Application

Conceptually:

```text
Git Repository
     ↓
ArgoCD Application
     ↓
Observability Namespace
     ↓
OTel Collector
```

ArgoCD continuously compares:

```text
Git desired state
       vs
Cluster actual state
```

---

# 74. Installation Through ArgoCD

The workflow:

```text
Developer
   ↓
Change Collector configuration
   ↓
Pull Request
   ↓
Review
   ↓
Merge
   ↓
ArgoCD detects change
   ↓
Sync
   ↓
EKS
```

This provides:

```text
Version control
Auditability
Repeatability
Rollback
Drift detection
```

---

# 75. Production Resource Configuration

Collector resources should be explicitly defined.

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

Do not copy these values blindly.

Measure actual:

```text
CPU
Memory
Telemetry rate
Queue size
```

and tune accordingly.

---

# 76. Production Replica Configuration

Gateway example:

```yaml
replicaCount: 3
```

Conceptually:

```text
Gateway
│
├── Pod-01
├── Pod-02
└── Pod-03
```

Distribute replicas across failure domains.

---

# 77. Pod Anti-Affinity

For production:

```text
Gateway-01 → Node-01
Gateway-02 → Node-02
Gateway-03 → Node-03
```

Use Kubernetes scheduling controls such as:

```text
Pod Anti-Affinity
Topology Spread Constraints
```

to reduce correlated failures.

---

# 78. Pod Disruption Budget

A PodDisruptionBudget can help preserve availability during voluntary disruptions.

Conceptually:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
```

For example:

```text
3 replicas
minimum available = 2
```

The exact value should match the deployment's availability requirements.

---

# 79. Horizontal Scaling

If the gateway becomes overloaded:

```text
Current:
2 replicas

Load increases

Scale:
3 replicas
```

Monitor:

```text
CPU
Memory
Telemetry throughput
Export latency
Queue size
Dropped telemetry
```

Then adjust replicas based on observed workload.

---

# 80. Collector Monitoring

Prometheus can monitor the Collector.

Architecture:

```text
OTel Collector
      ↓
Collector Metrics
      ↓
Prometheus
      ↓
Grafana
```

Create alerts for:

```text
Exporter failures
Dropped telemetry
High memory
High CPU
Queue growth
Unavailable replicas
```

---

# 81. Installation Security Checklist

```text
[ ] Collector endpoints private
[ ] TLS configured
[ ] Authentication configured where required
[ ] Secrets stored securely
[ ] Network access restricted
[ ] Kubernetes RBAC configured
[ ] Service accounts use least privilege
[ ] Sensitive telemetry filtered
[ ] Production configuration version controlled
```

---

# 82. Installation Reliability Checklist

```text
[ ] Multiple Collector replicas
[ ] Multi-node distribution
[ ] Health checks
[ ] Resource requests
[ ] Resource limits
[ ] Memory limiter
[ ] Batch processor
[ ] Retry configuration
[ ] Monitoring
[ ] Alerting
```

---

# 83. Installation Validation Checklist

```text
[ ] Operator healthy
[ ] Collector Pods healthy
[ ] Collector Service available
[ ] OTLP endpoint reachable
[ ] Application instrumentation working
[ ] Metrics arriving
[ ] Logs arriving
[ ] Traces arriving
[ ] Backend receiving data
[ ] Dashboards displaying data
```

---

# 84. Production Installation Architecture

A complete installation:

```text
                              EKS
                               │
          ┌────────────────────┼────────────────────┐
          ↓                    ↓                    ↓
       Node-01              Node-02              Node-03
          │                    │                    │
    Applications          Applications          Applications
          │                    │                    │
          ↓                    ↓                    ↓
      OTel Agent            OTel Agent            OTel Agent
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ↓
                        OTel Gateway Service
                               │
                 ┌─────────────┼─────────────┐
                 ↓             ↓             ↓
            Gateway-01     Gateway-02     Gateway-03
                 │             │             │
                 └─────────────┼─────────────┘
                               ↓
                ┌──────────────┼──────────────┐
                ↓              ↓              ↓
            Prometheus        ELK           Jaeger
                ↓              ↓              ↓
             Grafana         Kibana         Trace UI
```

---

# 85. Installation Troubleshooting Flow

When telemetry does not appear:

```text
No Telemetry
     ↓
Is application instrumented?
     ↓
Is SDK / agent running?
     ↓
Is service name configured?
     ↓
Is exporter configured?
     ↓
Can application reach Collector?
     ↓
Is Collector receiver running?
     ↓
Is pipeline configured?
     ↓
Is processor dropping data?
     ↓
Is exporter healthy?
     ↓
Is backend healthy?
```

---

# 86. Common Installation Problems

### Problem 1: Collector Pod CrashLoopBackOff

Check:

```bash
kubectl logs <pod> -n observability
```

Likely causes:

```text
Invalid YAML
Unsupported component
Invalid receiver
Invalid exporter
Pipeline configuration error
```

---

# 87. Problem 2: Connection Refused

Application:

```text
OTLP → Connection refused
```

Check:

```bash
kubectl get svc -n observability
kubectl get endpoints -n observability
kubectl get pods -n observability
```

Potential causes:

```text
Wrong service name
Wrong port
No endpoints
Collector not running
Network policy
```

---

# 88. Problem 3: No Traces

Check:

```text
Instrumentation
 ↓
SDK
 ↓
Exporter
 ↓
Collector
 ↓
Trace Pipeline
 ↓
Trace Exporter
 ↓
Backend
```

Check Collector logs and metrics.

---

# 89. Problem 4: No Metrics

Check:

```text
Metrics instrumentation
 ↓
SDK
 ↓
Collector
 ↓
Metrics pipeline
 ↓
Exporter
 ↓
Prometheus
```

Then verify the Prometheus side of the architecture.

---

# 90. Problem 5: No Logs

Check:

```text
Application logs
 ↓
OTel logging integration
 ↓
Collector
 ↓
Logs pipeline
 ↓
Exporter
 ↓
Logstash / Elasticsearch
```

Also verify filtering processors.

---

# 91. Problem 6: High Memory

Check:

```text
Collector memory
Queue
Batch size
Backend latency
Telemetry rate
```

Then:

```text
Enable/tune memory limiter
 ↓
Tune batch
 ↓
Fix backend bottleneck
 ↓
Scale Collector
```

---

# 92. Problem 7: High CPU

Check:

```text
Telemetry rate
Processor complexity
Sampling
Transformations
Number of pipelines
```

Then:

```text
Reduce unnecessary processing
 ↓
Optimize configuration
 ↓
Scale horizontally
```

---

# 93. Problem 8: Export Failures

Check:

```text
Backend health
Network
TLS
Authentication
Endpoint
Collector exporter
```

Example:

```text
Collector
   ↓
TLS error
   ↓
Backend
```

Certificate configuration should be investigated before changing application code.

---

# 94. OpenTelemetry Installation in a DevOps Pipeline

A production deployment can be integrated into CI/CD:

```text
Developer
   ↓
GitHub
   ↓
Pull Request
   ↓
GitHub Actions
   ├── YAML validation
   ├── Helm lint
   ├── Security scan
   └── Kubernetes validation
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
```

---

# 95. Security Validation

Before production deployment, validate:

```text
Collector configuration
TLS configuration
RBAC
Secrets
Network policies
Container image
Helm chart
Kubernetes manifests
```

Security scanning can be integrated into the CI/CD pipeline.

---

# 96. Production Installation Strategy

A recommended progression:

```text
Development
   ↓
Install Collector
   ↓
Test telemetry
   ↓
Staging
   ↓
Load test
   ↓
Validate failure handling
   ↓
Production
```

Do not install directly into production without validating the telemetry path.

---

# 97. Staging Validation

In staging:

```text
Generate traffic
      ↓
Check metrics
      ↓
Check logs
      ↓
Check traces
      ↓
Test Collector restart
      ↓
Test backend failure
      ↓
Test scaling
```

This helps expose problems before production deployment.

---

# 98. Production Deployment Sequence

A controlled rollout:

```text
Deploy Collector
      ↓
Check health
      ↓
Send test telemetry
      ↓
Verify backend
      ↓
Deploy instrumentation
      ↓
Generate traffic
      ↓
Verify all signals
      ↓
Monitor
```

Avoid changing every production service simultaneously if the organization requires a staged rollout.

---

# 99. Final Production Installation

The production installation should provide:

```text
Application Instrumentation
          ↓
OTel Agent / SDK
          ↓
OTel Gateway
          ↓
Signal-specific Pipelines
          ↓
Backends
```

With production controls:

```text
TLS
Authentication
RBAC
Resource Limits
Memory Limiting
Batching
Retry
Sampling
Monitoring
High Availability
GitOps
```

---

# 100. OpenTelemetry Installation Mental Model

Remember the installation process as:

```text
1. PREPARE
   Select signals
   Select languages
   Select backends
        ↓

2. INSTALL
   SDK / Auto-Instrumentation
        ↓

3. DEPLOY
   OTel Collector
        ↓

4. CONFIGURE
   Receivers
   Processors
   Exporters
   Pipelines
        ↓

5. CONNECT
   Applications → Collector
        ↓

6. EXPORT
   Collector → Backends
        ↓

7. VERIFY
   Metrics + Logs + Traces
        ↓

8. PRODUCTIONIZE
   HA + Security + Monitoring + GitOps
```

The key principle is:

**OpenTelemetry installation is not simply installing a Collector. A complete production implementation requires application instrumentation, SDK or auto-instrumentation configuration, Collector deployment, receivers, processors, exporters, signal-specific pipelines, secure networking, resource management, high availability, monitoring, and validation. In an EKS environment, a practical architecture is to run node-level Collectors for local collection and a highly available Collector gateway for centralized processing and export to Prometheus, Elasticsearch/ELK, Jaeger, or other observability backends.**
