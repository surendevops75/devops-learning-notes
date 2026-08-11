# OpenTelemetry Kubernetes

## 1. Overview

OpenTelemetry Kubernetes integration provides a standardized approach for collecting and exporting telemetry from Kubernetes workloads and the Kubernetes platform itself.

In a production Kubernetes environment, OpenTelemetry can collect:

```text
Application Traces
Application Metrics
Application Logs
Kubernetes Metadata
Node Metrics
Container Metrics
Kubernetes Events
```

A typical EKS architecture is:

```text
                         EKS Cluster
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
     Application Pods    Kubernetes Nodes    Cluster Components
          │                   │                   │
      OTel SDK            OTel Agent          Kubernetes APIs
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                       OTel Collector
                              ↓
                    OTel Gateway / Backend
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
          Prometheus          ELK       Trace Backend
              ↓               ↓               ↓
           Grafana          Kibana        Trace UI
```

---

# 2. Why OpenTelemetry in Kubernetes?

Kubernetes environments are highly dynamic.

Pods can:

```text
Start
Stop
Restart
Move between nodes
Scale horizontally
Be replaced during deployments
```

Therefore, observability must automatically identify changing workloads.

OpenTelemetry helps standardize telemetry collection across:

```text
Applications
Containers
Pods
Nodes
Namespaces
Clusters
Services
```

---

# 3. Kubernetes Observability Layers

A production Kubernetes environment can be observed at multiple layers.

```text
Cluster
  ↓
Node
  ↓
Pod
  ↓
Container
  ↓
Application
  ↓
Request
```

Each layer provides different information.

### Cluster

```text
Node availability
Scheduling
Cluster capacity
```

### Node

```text
CPU
Memory
Disk
Network
```

### Pod

```text
Restarts
Status
Resource usage
Availability
```

### Container

```text
CPU
Memory
Logs
Runtime behavior
```

### Application

```text
Requests
Errors
Latency
Business metrics
Traces
```

---

# 4. Kubernetes + OpenTelemetry Architecture

A scalable architecture is:

```text
                     Kubernetes Cluster
                            │
        ┌───────────────────┼───────────────────┐
        ↓                   ↓                   ↓
     Pod A               Pod B               Pod C
        │                   │                   │
     OTel SDK            OTel SDK            OTel SDK
        │                   │                   │
        └───────────────────┼───────────────────┘
                            ↓
                     OTel Agent
                            ↓
                     OTel Gateway
                            ↓
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
         Prometheus         ELK       Trace Backend
             ↓              ↓              ↓
          Grafana         Kibana        Trace UI
```

---

# 5. OpenTelemetry Collector in Kubernetes

The Collector is the central component responsible for:

```text
Receiving telemetry
Processing telemetry
Enriching telemetry
Filtering telemetry
Sampling traces
Batching telemetry
Exporting telemetry
```

Example:

```text
Application
     ↓
OTel SDK
     ↓
OTel Collector
     ↓
Backend
```

---

# 6. Collector Deployment Models

There are three important deployment patterns:

```text
Agent
Gateway
Sidecar
```

They can be used independently or together.

---

# 7. Agent Pattern

An Agent Collector runs close to workloads.

In Kubernetes, this is commonly implemented as a DaemonSet.

```text
Node 1
├── Application Pods
└── OTel Agent

Node 2
├── Application Pods
└── OTel Agent

Node 3
├── Application Pods
└── OTel Agent
```

The Agent provides local telemetry collection.

---

# 8. Gateway Pattern

A Gateway Collector runs centrally.

```text
OTel Agent
      │
OTel Agent
      │
OTel Agent
      ↓
OTel Gateway
      ↓
Backend
```

The Gateway can perform:

```text
Central processing
Tail sampling
Filtering
Routing
Batching
Export
```

---

# 9. Agent + Gateway Architecture

For production EKS:

```text
                 EKS
                  │
      ┌───────────┼───────────┐
      ↓           ↓           ↓
    Node 1      Node 2      Node 3
      │           │           │
   Agent       Agent       Agent
      │           │           │
      └───────────┼───────────┘
                  ↓
             Gateway
                  ↓
        ┌─────────┼─────────┐
        ↓         ↓         ↓
   Prometheus     ELK    Trace Backend
```

This provides separation between local collection and centralized processing.

---

# 10. DaemonSet Deployment

A Collector Agent is commonly deployed as a DaemonSet.

Conceptually:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: otel-agent
  namespace: observability
spec:
  selector:
    matchLabels:
      app: otel-agent
  template:
    metadata:
      labels:
        app: otel-agent
    spec:
      containers:
        - name: otel-agent
          image: otel/opentelemetry-collector-contrib:<version>
```

The exact image and configuration should be pinned to a tested version in production.

---

# 11. Why DaemonSet?

DaemonSet ensures that an Agent runs on each eligible Kubernetes node.

```text
Node 1 → Agent
Node 2 → Agent
Node 3 → Agent
```

When a new node is added:

```text
New Node
   ↓
DaemonSet
   ↓
New Agent Pod
```

This makes the architecture automatically adapt to cluster scaling.

---

# 12. Deployment for Gateway

The Gateway is commonly deployed as a Deployment.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: otel-gateway
  namespace: observability
spec:
  replicas: 3
```

Multiple replicas provide higher availability.

---

# 13. Kubernetes Service for Gateway

Agents need a stable endpoint.

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
    - name: otlp-http
      port: 4318
```

Applications or Agents can then send telemetry to the Service.

---

# 14. OTLP in Kubernetes

A common architecture:

```text
Application
     ↓
OTLP/gRPC
     ↓
OTel Agent
     ↓
OTLP/gRPC
     ↓
OTel Gateway
```

Common OTLP ports:

```text
4317 → OTLP/gRPC
4318 → OTLP/HTTP
```

Ensure NetworkPolicies and security controls permit only the required communication.

---

# 15. Application-to-Collector Communication

Suppose the Collector Service is:

```text
otel-agent.observability.svc.cluster.local
```

An application can be configured to send telemetry to:

```text
http://otel-agent.observability.svc.cluster.local:4318
```

or an appropriate gRPC endpoint.

The exact endpoint depends on the protocol and Collector configuration.

---

# 16. Environment Variables

OpenTelemetry applications can commonly be configured using environment variables.

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: orders

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: http://otel-agent.observability.svc.cluster.local:4317
```

Additional configuration can specify:

```text
Protocol
Headers
TLS
Sampling
Resource attributes
```

---

# 17. Kubernetes Resource Attributes

Kubernetes workloads should carry useful resource metadata.

Examples:

```text
service.name
service.version
deployment.environment
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
```

Example:

```text
service.name = payment
k8s.namespace.name = production
k8s.pod.name = payment-7d9f8c
```

---

# 18. Kubernetes Metadata Enrichment

The OpenTelemetry Collector can enrich telemetry with Kubernetes metadata.

Architecture:

```text
Application Telemetry
        ↓
OTel Collector
        ↓
Kubernetes Metadata
        ↓
Enriched Telemetry
```

This allows operators to answer:

```text
Which pod produced this log?
Which namespace?
Which node?
Which workload?
Which cluster?
```

---

# 19. Kubernetes Attributes Processor

The Kubernetes Attributes Processor can associate telemetry with Kubernetes resources.

Conceptually:

```yaml
processors:
  k8sattributes:
```

It can enrich telemetry with Kubernetes-related attributes based on the workload that generated it.

---

# 20. Kubernetes RBAC

Kubernetes metadata collection may require permissions to query Kubernetes APIs.

A typical setup includes:

```text
ServiceAccount
     ↓
Role / ClusterRole
     ↓
RoleBinding / ClusterRoleBinding
```

The Collector should receive only the permissions it actually needs.

---

# 21. Least Privilege

Do not give the Collector unrestricted Kubernetes permissions.

Use:

```text
Read-only permissions
Required resources only
Required namespaces only where possible
```

Security principle:

```text
Minimum required permissions
        ↓
Lower blast radius
```

---

# 22. ServiceAccount

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: otel-agent
  namespace: observability
```

The Collector Pod can use:

```yaml
serviceAccountName: otel-agent
```

This identity can then be bound to the required Kubernetes permissions.

---

# 23. ClusterRole

A conceptual example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: otel-agent
rules:
  - apiGroups: [""]
    resources:
      - pods
      - namespaces
      - nodes
    verbs:
      - get
      - list
      - watch
```

Only grant resources and verbs required by the Collector configuration.

---

# 24. Kubernetes Logs

A common Kubernetes logging model is:

```text
Application
     ↓
stdout / stderr
     ↓
Container Runtime
     ↓
Node Log Files
     ↓
OTel Agent
     ↓
Log Backend
```

The Collector can use a file-based receiver such as `filelog` to collect these logs.

---

# 25. Filelog Receiver

Conceptual configuration:

```yaml
receivers:
  filelog:
    include:
      - /var/log/pods/*/*/*.log
```

The exact paths depend on the Kubernetes node runtime and logging configuration.

---

# 26. Host Filesystem Mount

For an Agent to read node-level log files, the required host path must be mounted into the Collector Pod.

Conceptually:

```text
Kubernetes Node
│
└── /var/log/pods
        │
        ↓
   OTel Agent Pod
        │
        ↓
   filelog receiver
```

Without the required mount, the Collector cannot read those files.

---

# 27. Kubernetes Log Flow

```text
Pod
 ↓
stdout/stderr
 ↓
Container runtime
 ↓
/var/log/pods
 ↓
OTel Agent
 ↓
filelog receiver
 ↓
processors
 ↓
ELK
```

This is a common centralized logging architecture.

---

# 28. Container Runtime Considerations

Kubernetes clusters can use different container runtimes.

The log format and filesystem behavior should therefore be validated for the actual runtime.

Do not blindly assume every cluster has identical log paths or formats.

---

# 29. Kubernetes Metrics

OpenTelemetry can collect Kubernetes-related metrics.

Examples:

```text
Pod CPU
Pod memory
Node CPU
Node memory
Container restarts
Pod status
```

Additional Kubernetes components may provide metrics that complement application telemetry.

---

# 30. Host Metrics

The Host Metrics Receiver can collect node-level metrics.

Conceptually:

```text
Kubernetes Node
      ↓
Host Metrics Receiver
      ↓
OTel Collector
      ↓
Prometheus
```

Useful measurements include:

```text
CPU
Memory
Disk
Filesystem
Network
Load
```

---

# 31. Host Metrics in DaemonSet

A DaemonSet Collector is useful for node-level metrics.

```text
Node 1 → Agent → Host Metrics
Node 2 → Agent → Host Metrics
Node 3 → Agent → Host Metrics
```

Each Agent observes the node on which it runs.

---

# 32. Kubernetes Metrics vs Application Metrics

Application metrics:

```text
HTTP requests
Errors
Latency
Business operations
```

Kubernetes metrics:

```text
Pod CPU
Pod memory
Restarts
Node capacity
Pod status
```

Infrastructure metrics:

```text
Disk
Network
Filesystem
Host CPU
```

A complete monitoring solution combines these layers.

---

# 33. Application Traces in Kubernetes

An instrumented application can generate spans.

```text
Orders Pod
     ↓
OTel SDK
     ↓
OTLP
     ↓
OTel Agent
     ↓
OTel Gateway
     ↓
Trace Backend
```

The Kubernetes platform should not change the basic OpenTelemetry tracing model.

---

# 34. Application Metrics in Kubernetes

```text
Payment Pod
     ↓
OTel SDK
     ↓
Metric Reader
     ↓
OTLP
     ↓
OTel Agent
     ↓
Prometheus
```

Resource attributes identify:

```text
Pod
Namespace
Service
Version
Cluster
```

---

# 35. Application Logs in Kubernetes

```text
Payment Pod
     ↓
stdout
     ↓
Node log file
     ↓
OTel Agent
     ↓
OTel Gateway
     ↓
Elasticsearch
     ↓
Kibana
```

Trace IDs can connect these logs to distributed traces.

---

# 36. Unified Kubernetes Telemetry

```text
                         EKS
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
    Metrics              Logs              Traces
       │                  │                  │
    OTel SDK           stdout             OTel SDK
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                     OTel Agent
                          ↓
                     OTel Gateway
                          ↓
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
      Prometheus          ELK       Trace Backend
          ↓               ↓               ↓
       Grafana          Kibana         Trace UI
```

---

# 37. OpenTelemetry Operator

The OpenTelemetry Operator helps manage OpenTelemetry Collector deployments and can also assist with workload instrumentation.

Conceptually:

```text
Kubernetes API
      ↓
OpenTelemetry Operator
      ↓
Collector Resources
      ↓
Kubernetes Workloads
```

It uses Kubernetes-native custom resources.

---

# 38. OpenTelemetryCollector Custom Resource

A conceptual resource:

```yaml
apiVersion: opentelemetry.io/v1beta1
kind: OpenTelemetryCollector
metadata:
  name: otel
  namespace: observability
spec:
  mode: deployment
```

The exact API version and fields should match the Operator version installed in the cluster.

---

# 39. Collector Modes

The Operator can support deployment modes such as:

```text
Deployment
DaemonSet
StatefulSet
Sidecar
```

The correct mode depends on the collection requirement.

Examples:

```text
Node logs → DaemonSet
Central gateway → Deployment
Stateful processing → StatefulSet where appropriate
Pod-specific collector → Sidecar
```

---

# 40. Operator Benefits

The Operator provides:

```text
Declarative configuration
Kubernetes-native management
Automated Collector deployment
Lifecycle management
Instrumentation automation support
```

Instead of manually maintaining every Collector Pod definition.

---

# 41. Auto-Instrumentation

The OpenTelemetry Operator can support automatic instrumentation for supported languages.

Conceptually:

```text
Application Deployment
       ↓
Instrumentation Resource
       ↓
Operator
       ↓
Instrumentation injected/configured
       ↓
Application
       ↓
OTel telemetry
```

This can reduce manual application changes.

---

# 42. Instrumentation Resource

Conceptually:

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: otel-instrumentation
  namespace: production
spec:
  exporter:
    endpoint: http://otel-agent.observability.svc:4317
```

The exact schema depends on the installed Operator version and language instrumentation.

---

# 43. Java Auto-Instrumentation

A Java workload can use OpenTelemetry Java auto-instrumentation.

Conceptually:

```text
Java Application
      ↓
Java Agent
      ↓
HTTP / JDBC / Messaging instrumentation
      ↓
OTLP
      ↓
Collector
```

This can provide traces and metrics with minimal application code changes.

---

# 44. Node.js Auto-Instrumentation

Node.js applications can use supported OpenTelemetry instrumentation libraries.

Typical flow:

```text
Node.js
   ↓
OTel Instrumentation
   ↓
HTTP / DB / Messaging
   ↓
OTLP
   ↓
Collector
```

---

# 45. Python Auto-Instrumentation

Python applications can use OpenTelemetry instrumentation for supported frameworks and libraries.

```text
Python
   ↓
Auto Instrumentation
   ↓
HTTP / DB / Framework
   ↓
OTLP
   ↓
Collector
```

Always test auto-instrumentation against the application's actual framework and dependencies.

---

# 46. Auto-Instrumentation vs Manual

Auto-instrumentation:

```text
Fast adoption
Less code
Standard library instrumentation
```

Manual instrumentation:

```text
Business operations
Custom workflows
Domain-specific spans
```

Production systems often use both.

---

# 47. Namespace Organization

A dedicated observability namespace is commonly used:

```text
observability
```

Example:

```text
kubectl create namespace observability
```

Then deploy:

```text
OTel Agent
OTel Gateway
OpenTelemetry Operator
Monitoring components
```

according to the platform design.

---

# 48. Helm Installation

The OpenTelemetry Collector can be deployed using Helm.

Conceptually:

```bash
helm repo add opentelemetry-helm https://open-telemetry.github.io/opentelemetry-helm-charts
```

Then:

```bash
helm repo update
```

The actual chart name and version should be pinned and validated before production use.

---

# 49. Helm Values

A production deployment commonly manages configuration through:

```text
values.yaml
```

Example conceptual settings:

```yaml
mode: daemonset

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

Production values should be based on measured telemetry volume.

---

# 50. GitOps Deployment

For an EKS environment using GitOps:

```text
Git
 ↓
Collector Helm Values
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
 ↓
OTel Collector
```

This provides:

```text
Version control
Review
Rollback
Auditability
Drift detection
```

---

# 51. Terraform + OpenTelemetry

Infrastructure can be provisioned using Terraform.

For example:

```text
Terraform
   ↓
EKS
   ↓
Observability Namespace
   ↓
Helm Release
   ↓
OTel Collector
```

Terraform can manage infrastructure while Helm/GitOps manages Kubernetes applications, depending on the team's operating model.

---

# 52. EKS Architecture

A production AWS architecture can look like:

```text
                         AWS
                          │
                        Route53
                          │
                          ↓
                         ALB
                          │
                          ↓
                         EKS
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
    Orders             Payment           Inventory
       │                  │                  │
    OTel SDK           OTel SDK           OTel SDK
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                     OTel Agent
                          ↓
                     OTel Gateway
```

---

# 53. Private EKS Networking

In a production architecture, worker nodes may be placed in private subnets.

```text
Internet
   ↓
ALB
   ↓
Private EKS Nodes
   ↓
Application Pods
```

Telemetry traffic can remain inside the VPC:

```text
Application
   ↓
OTel Agent
   ↓
OTel Gateway
```

This reduces unnecessary exposure.

---

# 54. Network Security

Protect Collector communication using:

```text
Security Groups
NetworkPolicies
TLS
Authentication
Private networking
```

Only required sources should be allowed to reach the Collector.

---

# 55. NetworkPolicy

Example conceptual policy:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: otel-gateway
  namespace: observability
spec:
  podSelector:
    matchLabels:
      app: otel-gateway
  policyTypes:
    - Ingress
```

A production policy should explicitly define allowed sources and ports.

---

# 56. TLS

Telemetry traffic may contain operational information.

For communication outside a trusted network boundary, use TLS.

```text
Application
     ↓
TLS
     ↓
OTel Collector
```

For multi-cluster or external backend communication:

```text
Collector
     ↓
TLS
     ↓
Backend
```

---

# 57. Authentication

Depending on the backend, authentication may use:

```text
API keys
Bearer tokens
mTLS
Cloud IAM-based mechanisms
Basic authentication
```

Credentials should be stored securely.

Do not hard-code credentials inside container images or Git repositories.

---

# 58. Kubernetes Secrets

Sensitive Collector configuration can use Kubernetes Secrets.

```text
Secret
  ↓
Collector Pod
  ↓
Environment / mounted configuration
```

For production, consider integrating with a centralized secrets management solution where appropriate.

---

# 59. AWS Secrets Manager

An EKS environment can use AWS Secrets Manager for sensitive values.

Conceptually:

```text
AWS Secrets Manager
        ↓
Secrets integration
        ↓
EKS
        ↓
OTel Collector
```

This can reduce the need to store long-lived credentials directly in Kubernetes manifests.

---

# 60. Collector Resource Requests

Collectors consume CPU and memory.

Configure requests:

```yaml
resources:
  requests:
    cpu: 200m
    memory: 256Mi
```

and limits:

```yaml
  limits:
    cpu: 1
    memory: 1Gi
```

The correct values depend on telemetry volume and processing complexity.

---

# 61. Memory Limiter

A production Collector should use a memory limiter.

Conceptually:

```yaml
processors:
  memory_limiter:
```

It protects the Collector from uncontrolled memory growth.

```text
Telemetry spike
      ↓
Memory pressure
      ↓
Memory limiter
      ↓
Protect Collector
```

---

# 62. Batch Processor

Batching improves export efficiency.

```yaml
processors:
  batch:
```

Flow:

```text
Span 1 ┐
Span 2 ├──→ Batch
Span 3 ┘
          ↓
       Exporter
```

This reduces network overhead.

---

# 63. Queue and Retry

Exporters can use queues and retry behavior where supported.

```text
Telemetry
   ↓
Queue
   ↓
Exporter
   ↓
Backend
```

If the backend temporarily fails:

```text
Retry
   ↓
Backend recovers
   ↓
Export continues
```

Queues must remain bounded.

---

# 64. Collector Scaling

Scale the Gateway horizontally:

```text
OTel Gateway
│
├── Pod 1
├── Pod 2
├── Pod 3
└── Pod 4
```

Scaling should consider:

```text
CPU
Memory
Telemetry throughput
Queue size
Export latency
Backend capacity
```

---

# 65. Horizontal Pod Autoscaler

A Gateway may be scaled based on resource utilization or suitable custom metrics.

Conceptually:

```text
Telemetry Volume ↑
       ↓
CPU / Memory ↑
       ↓
HPA
       ↓
More Gateway Pods
```

Autoscaling must be tested carefully because telemetry pipelines can have bursty traffic.

---

# 66. Pod Anti-Affinity

Do not place every Gateway replica on one node.

Conceptually:

```text
Node 1 → Gateway-1
Node 2 → Gateway-2
Node 3 → Gateway-3
```

This improves resilience against node failure.

---

# 67. Topology Spread

Topology spread constraints can distribute Collector replicas across:

```text
Nodes
Availability Zones
Other topology domains
```

Example:

```text
AZ-A → Gateway
AZ-B → Gateway
AZ-C → Gateway
```

This is useful for production EKS clusters.

---

# 68. PodDisruptionBudget

A PodDisruptionBudget can help preserve Collector availability during voluntary disruptions.

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: otel-gateway
spec:
  minAvailable: 2
```

The value should match the number of replicas and availability requirements.

---

# 69. Collector High Availability

Production:

```text
                Service
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
    Gateway-1  Gateway-2  Gateway-3
        │          │          │
        └──────────┼──────────┘
                   ↓
                Backend
```

Avoid a single Collector Gateway replica for critical production telemetry.

---

# 70. Failure Scenario: Agent Failure

If an Agent Pod fails:

```text
Node
 ↓
Agent crashes
```

DaemonSet recreates it:

```text
Agent
 ↓
Failure
 ↓
Kubernetes
 ↓
New Agent
```

Telemetry during the failure window may still be lost depending on buffering and application configuration.

---

# 71. Failure Scenario: Gateway Failure

With multiple replicas:

```text
Gateway-1 → FAILED
Gateway-2 → ACTIVE
Gateway-3 → ACTIVE
```

The Kubernetes Service can continue routing traffic to healthy replicas.

---

# 72. Failure Scenario: Backend Failure

If the backend fails:

```text
Collector
   ↓
Backend
   X
```

The Collector should:

```text
Retry where configured
Queue where configured
Apply backpressure
Drop telemetry when necessary
```

It should not consume unlimited memory.

---

# 73. Observability of the Collector

Monitor:

```text
Collector CPU
Collector memory
Received telemetry
Exported telemetry
Dropped telemetry
Exporter failures
Queue size
Export latency
```

Use Prometheus and Grafana to monitor the Collector itself.

---

# 74. Collector Self-Monitoring

```text
OTel Collector
      ↓
Internal Metrics
      ↓
Prometheus
      ↓
Grafana
```

This creates an important feedback loop:

```text
Applications
   ↓
Observability Pipeline
   ↓
Monitor Pipeline
```

---

# 75. Collector Health Checks

The Collector can expose health information.

Conceptually:

```text
Collector
   ↓
Health Check
   ↓
Kubernetes Probe
```

Kubernetes can then restart or stop routing traffic to unhealthy Pods.

---

# 76. Liveness Probe

A liveness probe answers:

```text
Is the Collector process alive?
```

If it fails:

```text
Kubernetes
   ↓
Restart Pod
```

---

# 77. Readiness Probe

Readiness answers:

```text
Can this Collector receive traffic?
```

If not ready:

```text
Service
   ↓
Stops routing traffic
```

This is especially important for Gateway replicas.

---

# 78. Startup Probe

Startup probes can help when the Collector requires additional initialization time.

```text
Pod starts
   ↓
Startup Probe
   ↓
Application initializes
   ↓
Readiness begins
```

This avoids premature liveness failures.

---

# 79. Kubernetes Scheduling

Collector Pods consume cluster resources.

Use:

```text
Requests
Limits
Node selectors
Tolerations
Affinity
Topology spread
```

to control placement.

---

# 80. Tolerations

If application nodes have taints:

```text
node
  taint = workload=application:NoSchedule
```

the Collector may require a corresponding toleration if it must run there.

For a DaemonSet, verify scheduling across all required nodes.

---

# 81. Node Selectors

A Collector can be scheduled to specific node types.

Example:

```yaml
nodeSelector:
  workload: observability
```

This can isolate observability workloads.

However, dedicated nodes increase infrastructure cost, so use them when operational requirements justify it.

---

# 82. Dedicated Observability Nodes

For large clusters:

```text
Application Nodes
│
├── Orders
├── Payment
└── Inventory

Observability Nodes
│
├── OTel Gateway
├── Prometheus
└── Grafana
```

Benefits:

```text
Isolation
Predictable capacity
Reduced contention
```

---

# 83. Namespace Isolation

A typical structure:

```text
production
staging
development
observability
```

Observability components run in:

```text
observability
```

This provides:

```text
Resource organization
RBAC separation
NetworkPolicy control
Operational clarity
```

---

# 84. ResourceQuota

A namespace can have resource limits.

Example:

```text
observability namespace
   ↓
ResourceQuota
   ↓
CPU / Memory limits
```

Ensure the quota is large enough for the expected Collector workload.

---

# 85. LimitRange

LimitRange can define default resource behavior.

```text
Namespace
   ↓
LimitRange
   ↓
Default requests/limits
```

Explicit resource requests and limits are still preferred for production-critical components.

---

# 86. OpenTelemetry Collector Configuration

A production configuration usually has:

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
    endpoint: trace-backend:4317

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

The pipeline must be explicitly declared.

---

# 87. Logs Pipeline

Conceptually:

```yaml
service:
  pipelines:
    logs:
      receivers:
        - otlp
        - filelog
      processors:
        - memory_limiter
        - batch
      exporters:
        - elasticsearch
```

The exact exporter and configuration depend on the ELK architecture.

---

# 88. Metrics Pipeline

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

The exporter configuration must match the chosen Prometheus architecture.

---

# 89. Three Signal Pipelines

A unified Collector can process all three signals:

```text
               OTel Collector
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      Traces       Metrics       Logs
        │            │            │
        ↓            ↓            ↓
     Backend      Prometheus       ELK
```

This is one of the major benefits of OpenTelemetry.

---

# 90. Kubernetes Events

Kubernetes Events can provide useful operational information:

```text
Pod scheduled
Image pulled
Container started
Probe failed
Pod evicted
Container restarted
```

These events complement application logs and metrics.

---

# 91. Kubernetes Event Troubleshooting

Suppose a Pod is unavailable.

Application telemetry may show:

```text
No application traces
```

Kubernetes Events may show:

```text
FailedScheduling
FailedMount
Back-off restarting container
Unhealthy
```

Therefore, Kubernetes platform telemetry is important alongside application telemetry.

---

# 92. Pod Restart Investigation

Metric:

```text
container_restart_count ↑
```

Logs:

```text
Application crashed
```

Kubernetes:

```text
Reason: OOMKilled
```

Trace:

```text
Requests failing before restart
```

This provides a complete picture.

---

# 93. OOMKilled Investigation

Flow:

```text
Pod Memory ↑
     ↓
Memory Limit
     ↓
OOMKilled
     ↓
Container Restart
     ↓
Error Rate ↑
```

Use:

```text
Prometheus
Kubernetes metadata
Application logs
Traces
```

to investigate.

---

# 94. CrashLoopBackOff Investigation

```text
CrashLoopBackOff
       ↓
kubectl describe pod
       ↓
Events
       ↓
Application logs
       ↓
Metrics
       ↓
Traces if application starts
```

OpenTelemetry cannot replace Kubernetes-native troubleshooting commands.

It complements them.

---

# 95. ImagePullBackOff

If a container never starts:

```text
ImagePullBackOff
```

there may be:

```text
No application logs
No application traces
No application metrics
```

Kubernetes Events become critical.

This demonstrates why observability must include both application and platform layers.

---

# 96. Kubernetes Deployment Observability

During a deployment:

```text
New ReplicaSet
      ↓
New Pods
      ↓
Application telemetry
      ↓
Metrics
      ↓
Logs
      ↓
Traces
```

Track:

```text
Pod readiness
Error rate
Latency
Restart count
Version
```

---

# 97. ArgoCD + OpenTelemetry

For a GitOps workflow:

```text
Git
 ↓
Application manifests
 ↓
OTel configuration
 ↓
ArgoCD
 ↓
EKS
```

After deployment:

```text
ArgoCD
 ↓
New application version
 ↓
OpenTelemetry telemetry
 ↓
Prometheus / ELK / Trace Backend
```

Telemetry can verify whether the deployment behaves correctly.

---

# 98. Canary Deployment

Example:

```text
Version v1 → 90%
Version v2 → 10%
```

Compare:

```text
v1
 ├── Error Rate
 ├── Latency
 └── Traces

v2
 ├── Error Rate
 ├── Latency
 └── Traces
```

If v2 performs poorly:

```text
Rollback
```

---

# 99. Kubernetes HPA and OpenTelemetry

Application metrics can contribute to autoscaling architectures.

```text
Application
     ↓
Metric
     ↓
Metrics Pipeline
     ↓
Metrics Backend / Adapter
     ↓
HPA
     ↓
More Pods
```

The exact integration depends on the Kubernetes metrics architecture.

Do not assume every OpenTelemetry metric is automatically available to HPA.

---

# 100. Production EKS Architecture

```text
                                AWS
                                 │
                              Route53
                                 │
                                 ↓
                                ALB
                                 │
                                 ↓
                         Private EKS Nodes
                                 │
          ┌──────────────────────┼──────────────────────┐
          ↓                      ↓                      ↓
       Orders                 Payment               Inventory
          │                      │                      │
       OTel SDK               OTel SDK               OTel SDK
          │                      │                      │
          └──────────────────────┼──────────────────────┘
                                 ↓
                         OTel Agent DaemonSet
                                 ↓
                         OTel Gateway Deployment
                                 │
              ┌──────────────────┼──────────────────┐
              ↓                  ↓                  ↓
          Prometheus        Elasticsearch       Trace Backend
              ↓                  ↓                  ↓
           Grafana             Kibana             Trace UI
```

---

# 101. Production Security Architecture

```text
Application Pods
      │
      │ Internal Cluster Network
      ↓
OTel Agent
      │
      │ NetworkPolicy
      ↓
OTel Gateway
      │
      │ TLS / Authentication
      ↓
Telemetry Backend
```

Security controls:

```text
RBAC
NetworkPolicies
TLS
Secrets
Private networking
Least privilege
```

---

# 102. Production High Availability

```text
                    EKS
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
       Node/AZ     Node/AZ     Node/AZ
          │          │          │
       Agent       Agent       Agent
          │          │          │
          └──────────┼──────────┘
                     ↓
              Gateway Service
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
   Gateway-1     Gateway-2     Gateway-3
       │             │             │
       └─────────────┼─────────────┘
                     ↓
                  Backends
```

---

# 103. Production Reliability

A production OpenTelemetry deployment should handle:

```text
Pod restart
Node failure
AZ failure
Collector failure
Backend failure
Traffic spikes
Application deployment
Network interruption
```

Use:

```text
Multiple replicas
DaemonSets
Queues
Retries
Memory limits
Batching
Sampling
Health probes
PodDisruptionBudgets
Topology spreading
```

---

# 104. Production Monitoring

Monitor both applications and the observability system.

Application:

```text
Request rate
Error rate
Latency
CPU
Memory
Restarts
```

Collector:

```text
Received telemetry
Exported telemetry
Dropped telemetry
Queue size
CPU
Memory
Exporter failures
```

Backend:

```text
Ingestion rate
Storage
Query latency
Errors
Capacity
```

---

# 105. Troubleshooting: Application Telemetry Missing

Check:

```text
1. Application instrumentation
2. OTEL_SERVICE_NAME
3. OTLP endpoint
4. Network connectivity
5. Collector receiver
6. Collector pipeline
7. Processor filtering
8. Exporter
9. Backend
```

Test each stage independently.

---

# 106. Troubleshooting: Collector Not Receiving Data

Check:

```text
Application
   ↓
OTLP endpoint
   ↓
Service
   ↓
Collector receiver
```

Verify:

```text
Port
Protocol
DNS
Service
NetworkPolicy
TLS
Authentication
```

---

# 107. Troubleshooting: Collector Receiving but Not Exporting

Check:

```text
Receiver
   ↓
Processor
   ↓
Exporter
```

Look for:

```text
Processor filtering
Exporter configuration
Backend connectivity
Authentication
TLS
Queue failures
Memory pressure
```

---

# 108. Troubleshooting: Logs Missing

Check:

```text
stdout
 ↓
node log file
 ↓
host mount
 ↓
filelog receiver
 ↓
processor
 ↓
exporter
 ↓
ELK
```

Validate each stage.

---

# 109. Troubleshooting: Kubernetes Metadata Missing

Check:

```text
ServiceAccount
 ↓
RBAC
 ↓
Kubernetes API access
 ↓
k8sattributes processor
```

Common problems:

```text
Insufficient permissions
Wrong namespace
Incorrect processor configuration
Pod identification failure
```

---

# 110. Troubleshooting: Traces Not Connected

If:

```text
Orders Trace
```

and:

```text
Payment Trace
```

appear as separate traces, check:

```text
traceparent propagation
HTTP instrumentation
Messaging instrumentation
Context extraction
Context injection
Sampling
```

The most common conceptual problem is broken context propagation.

---

# 111. Troubleshooting: High Collector Memory

Symptoms:

```text
Memory ↑
OOMKilled
Telemetry dropped
Export delays
```

Investigate:

```text
Telemetry volume
Queue size
Batch settings
Tail sampling
Backend availability
Large attributes
High-cardinality data
```

Then tune:

```text
Memory limiter
Queue
Batch
Sampling
Resources
```

---

# 112. Troubleshooting: High Collector CPU

Possible causes:

```text
High telemetry volume
Complex processors
Tail sampling
Heavy transformations
Large number of attributes
```

Solutions:

```text
Scale Gateway
Reduce unnecessary telemetry
Optimize processors
Filter low-value data
Tune sampling
```

---

# 113. Troubleshooting: Backend Slow

If:

```text
Backend latency ↑
```

then:

```text
Collector queues ↑
       ↓
Memory ↑
       ↓
Telemetry drops
```

Investigate the backend before continuously increasing Collector memory.

---

# 114. OpenTelemetry Kubernetes Best Practices

```text
1. Use Agent + Gateway for larger clusters.
2. Deploy Agents as DaemonSets where appropriate.
3. Run Gateway with multiple replicas.
4. Use resource requests and limits.
5. Enable memory limiting.
6. Use batching.
7. Control trace sampling.
8. Avoid high-cardinality attributes.
9. Enrich telemetry with Kubernetes metadata.
10. Use least-privilege RBAC.
11. Secure telemetry with TLS where required.
12. Use NetworkPolicies.
13. Monitor the Collector itself.
14. Version-control Collector configuration.
15. Use GitOps for production changes.
```

---

# 115. OpenTelemetry Kubernetes Checklist

```text
[ ] Observability namespace created
[ ] OpenTelemetry Collector deployed
[ ] Agent DaemonSet configured
[ ] Gateway Deployment configured
[ ] Gateway Service configured
[ ] OTLP ports configured
[ ] ServiceAccount configured
[ ] RBAC configured
[ ] Kubernetes metadata enrichment configured
[ ] Resource attributes configured
[ ] Application instrumentation configured
[ ] Logs collection configured
[ ] Metrics collection configured
[ ] Traces collection configured
[ ] Memory limiter configured
[ ] Batch processor configured
[ ] Sampling configured
[ ] TLS configured where required
[ ] NetworkPolicy configured
[ ] Resource requests/limits configured
[ ] Health probes configured
[ ] PDB configured
[ ] Topology spreading configured
[ ] Collector monitoring configured
[ ] Backend monitoring configured
[ ] GitOps deployment configured
```

---

# 116. Final Production Mental Model

Remember OpenTelemetry Kubernetes as:

```text
KUBERNETES
    ↓
APPLICATIONS
    ↓
OTel SDK
    ↓
OTel Agent
    ↓
OTel Gateway
    ↓
PROCESS
    ↓
FILTER
    ↓
ENRICH
    ↓
BATCH
    ↓
SAMPLE
    ↓
EXPORT
    ↓
OBSERVABILITY BACKENDS
```

For an EKS microservices platform:

```text
                         EKS
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
     Metrics             Logs             Traces
        │                 │                 │
     OTel SDK          stdout/file        OTel SDK
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                    OTel Agent
                     DaemonSet
                          ↓
                    OTel Gateway
                    Deployment
                          ↓
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
      Prometheus          ELK        Trace Backend
          ↓               ↓               ↓
       Grafana          Kibana         Trace UI
```

The key principle is:

**OpenTelemetry Kubernetes integration provides a scalable, Kubernetes-native observability architecture in which application telemetry is generated through OpenTelemetry SDKs, collected close to workloads through Collector Agents, centrally processed through Gateway Collectors, enriched with Kubernetes metadata, protected with resource and security controls, and exported to Prometheus, ELK, and tracing backends. In production EKS environments, the Collector layer should be highly available, resource-controlled, observable itself, and managed through GitOps with appropriate RBAC, NetworkPolicies, TLS, sampling, batching, and failure-handling strategies.**
