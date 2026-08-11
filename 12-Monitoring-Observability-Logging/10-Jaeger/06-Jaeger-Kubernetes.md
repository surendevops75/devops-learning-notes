# Jaeger Kubernetes

## 1. Overview

Jaeger can be deployed in Kubernetes to provide distributed tracing for microservices running inside the cluster.

A basic architecture is:

```text
Kubernetes Cluster
│
├── Application Pods
│      ↓
│   OpenTelemetry SDK
│      ↓
├── OpenTelemetry Collector
│      ↓
├── Jaeger
│      ↓
├── Trace Storage
│      ↓
├── Jaeger Query
│      ↓
└── Jaeger UI
```

For a production EKS environment, a more scalable architecture is:

```text
Application Pods
       ↓
OTel Agent
       ↓
OTel Gateway
       ↓
Sampling / Processing
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

# 2. Why Run Jaeger on Kubernetes?

Running Jaeger inside Kubernetes provides:

```text
Containerized deployment
Service discovery
Horizontal scaling
Self-healing
Rolling updates
Resource management
Namespace isolation
RBAC
NetworkPolicy
GitOps integration
```

It also allows Jaeger to run close to the applications generating traces.

---

# 3. Kubernetes Observability Namespace

Create a dedicated namespace:

```bash
kubectl create namespace observability
```

Verify:

```bash
kubectl get namespace observability
```

A dedicated namespace keeps observability components separate from application workloads.

Example:

```text
observability
├── OpenTelemetry Collector
├── Jaeger
├── Query
└── Supporting components
```

---

# 4. Kubernetes Architecture

A typical architecture:

```text
                     EKS Cluster
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
   Application-1     Application-2     Application-3
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                   OTel Collector
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

# 5. Jaeger Components

Depending on the deployment model and Jaeger version, the architecture can contain:

```text
Jaeger
├── Ingestion
├── Query
├── UI
└── Storage
```

In a production Kubernetes environment, these components may be independently scaled.

---

# 6. Kubernetes Deployment Models

Common deployment approaches include:

```text
Docker image
Kubernetes manifests
Helm
GitOps
Operator-based deployment where supported
```

For production Kubernetes:

```text
Git
 ↓
Helm / Kubernetes configuration
 ↓
ArgoCD
 ↓
EKS
```

GitOps provides repeatability and change tracking.

---

# 7. Helm-Based Deployment

Helm simplifies Kubernetes deployment.

Verify Helm:

```bash
helm version
```

Update repositories:

```bash
helm repo update
```

Search available Jaeger charts:

```bash
helm search repo jaeger
```

Always verify the chart version and application version before deploying.

---

# 8. Helm Values

Maintain production configuration in Git.

Example:

```text
observability/
└── jaeger/
    ├── values-dev.yaml
    ├── values-staging.yaml
    └── values-prod.yaml
```

Environment-specific configuration prevents accidental reuse of development settings in production.

---

# 9. Kubernetes Service Discovery

Applications should communicate with Jaeger or the Collector using Kubernetes Services.

Example:

```text
Application
     ↓
otel-gateway.observability.svc.cluster.local
```

Do not configure applications with Pod IP addresses.

Pod IPs can change when Pods are recreated.

---

# 10. Kubernetes DNS

A Kubernetes Service provides a stable DNS name.

Example:

```text
otel-gateway.observability.svc.cluster.local
```

General format:

```text
<service>.<namespace>.svc.cluster.local
```

This allows applications to communicate with observability components without knowing their Pod IPs.

---

# 11. OpenTelemetry Collector Architecture

For Kubernetes, a common design uses an Agent and Gateway.

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

The Agents forward telemetry to centralized Gateways.

```text
OTel Agents
     ↓
OTel Gateway
     ↓
Jaeger
```

---

# 12. Agent as DaemonSet

An OTel Agent is commonly deployed as a DaemonSet.

```text
Kubernetes Node
      ↓
OTel Agent
```

A DaemonSet ensures an Agent is scheduled on each eligible node.

Conceptually:

```text
Node-1 → Agent-1
Node-2 → Agent-2
Node-3 → Agent-3
```

---

# 13. Gateway as Deployment

The Gateway is normally deployed as a Deployment.

```text
OTel Gateway
├── Pod-1
├── Pod-2
└── Pod-3
```

The Gateway can be exposed through a Kubernetes Service.

```text
Agents
   ↓
Gateway Service
   ↓
Gateway Pods
```

---

# 14. Application to Collector Flow

Application configuration:

```text
Application
     ↓
OTLP
     ↓
OTel Agent
```

Then:

```text
OTel Agent
     ↓
OTel Gateway
     ↓
Jaeger
```

Complete flow:

```text
Application
    ↓
OTel SDK
    ↓
OTLP
    ↓
OTel Agent
    ↓
OTel Gateway
    ↓
Jaeger
```

---

# 15. Application Environment Variables

A Kubernetes application can receive OpenTelemetry configuration through environment variables.

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: orders

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: http://otel-gateway.observability.svc.cluster.local:4317
```

The exact configuration depends on the application language and instrumentation.

---

# 16. Service Name

Every microservice should have a meaningful service name.

Example:

```text
orders
payment
inventory
notification
user
```

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: payment
```

This makes services easy to identify in Jaeger.

---

# 17. Service Version

Add version metadata.

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: payment
```

The application should also provide:

```text
service.version=2.4.1
```

This allows trace comparison across deployments.

---

# 18. Deployment Environment

Add environment metadata:

```text
development
staging
production
```

Example:

```text
service.name=payment
service.version=2.4.1
deployment.environment=production
```

This is important when multiple environments share observability infrastructure.

---

# 19. Kubernetes Metadata

Useful Kubernetes attributes include:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
```

Example:

```text
service.name=payment
k8s.namespace.name=production
k8s.pod.name=payment-7c8f6b9d8f-x2abc
```

This allows engineers to move from a trace to the exact Kubernetes workload.

---

# 20. Kubernetes Attributes Processor

The OpenTelemetry Collector can enrich telemetry with Kubernetes metadata.

Conceptually:

```yaml
processors:
  k8sattributes:
```

Flow:

```text
Application
    ↓
OTel Agent
    ↓
Kubernetes Attributes
    ↓
OTel Gateway
```

The Collector requires appropriate Kubernetes permissions for metadata discovery.

---

# 21. RBAC for Collector

If the Collector needs Kubernetes metadata, create a dedicated ServiceAccount.

Architecture:

```text
Collector
    ↓
ServiceAccount
    ↓
RBAC
    ↓
Kubernetes API
```

Grant only the permissions required for the Collector's functionality.

Avoid unnecessarily granting:

```text
cluster-admin
```

---

# 22. OTLP Receiver

The Collector can receive OTLP:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:
```

Common ports:

```text
4317 → OTLP/gRPC
4318 → OTLP/HTTP
```

The application protocol must match the configured receiver.

---

# 23. Collector Service

Expose the Collector through a Kubernetes Service.

Conceptually:

```text
apiVersion: v1
kind: Service
metadata:
  name: otel-gateway
  namespace: observability
```

Applications can then use:

```text
otel-gateway.observability.svc.cluster.local
```

---

# 24. Jaeger Service

Jaeger components should also communicate through Kubernetes Services.

Conceptually:

```text
OTel Gateway
      ↓
Jaeger Service
      ↓
Jaeger Pods
```

The exact Service names depend on the deployment.

Always verify them:

```bash
kubectl get svc -n observability
```

---

# 25. Kubernetes Labels

Use consistent labels.

Example:

```yaml
labels:
  app: jaeger
  component: query
```

Labels are used by:

```text
Services
Deployments
NetworkPolicies
Monitoring
Selectors
```

A mismatch between Service selectors and Pod labels can cause connectivity problems.

---

# 26. Kubernetes Deployment

A simplified Deployment concept:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger-query
  namespace: observability
spec:
  replicas: 2
```

The actual container image, ports, environment variables, and configuration depend on the Jaeger version and deployment method.

---

# 27. Replica Configuration

For production:

```text
Jaeger Query
├── Replica 1
├── Replica 2
└── Replica 3
```

Multiple replicas improve availability.

However, scaling Query does not automatically make the storage backend highly available.

Storage must also be designed appropriately.

---

# 28. Resource Requests

Configure resource requests.

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
```

Requests help Kubernetes schedule Pods onto suitable nodes.

---

# 29. Resource Limits

Example:

```yaml
resources:
  limits:
    cpu: "1"
    memory: 1Gi
```

These are example values.

Production values should be based on:

```text
Trace volume
Span rate
Query workload
Sampling
Storage performance
```

---

# 30. Why Resource Configuration Matters

Without resource planning:

```text
Trace volume ↑
     ↓
Memory ↑
     ↓
Pod OOMKilled
```

or:

```text
Query load ↑
     ↓
CPU ↑
     ↓
Query latency ↑
```

Monitor actual usage and adjust resources accordingly.

---

# 31. Readiness Probe

A readiness probe tells Kubernetes whether the Pod is ready to receive traffic.

Conceptually:

```text
Pod starting
    ↓
Not Ready
    ↓
Initialization
    ↓
Ready
    ↓
Service sends traffic
```

This prevents traffic from being sent to an uninitialized component.

---

# 32. Liveness Probe

A liveness probe helps Kubernetes detect unhealthy containers.

Conceptually:

```text
Container
    ↓
Healthy
    ↓
Continue
```

If the container becomes unhealthy:

```text
Container
    ↓
Failure
    ↓
Kubernetes restart
```

Probe endpoints must match the component and version being deployed.

---

# 33. Startup Probe

Startup probes can be useful for components that take longer to initialize.

```text
Container starts
      ↓
Startup probe
      ↓
Initialization complete
      ↓
Readiness / Liveness
```

This prevents Kubernetes from restarting a slow-starting container prematurely.

---

# 34. PodDisruptionBudget

For multiple replicas:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: jaeger-query
  namespace: observability
spec:
  minAvailable: 1
```

A PDB helps protect availability during voluntary disruptions.

The selector must match the intended Pods.

---

# 35. Pod Anti-Affinity

Avoid placing all replicas on one node.

Bad:

```text
Node-1
├── Query-1
├── Query-2
└── Query-3
```

Better:

```text
Node-1 → Query-1
Node-2 → Query-2
Node-3 → Query-3
```

Use Pod anti-affinity or topology spread constraints.

---

# 36. Topology Spread

For a production EKS cluster:

```text
AZ-A → Query-1
AZ-B → Query-2
AZ-C → Query-3
```

This provides better resilience against node or Availability Zone failures.

---

# 37. Persistent Storage

Production tracing requires appropriate durable storage.

Avoid relying on:

```text
Pod local ephemeral storage
```

when persistent trace retention is required.

Architecture:

```text
Jaeger
   ↓
Durable Storage
```

The storage design depends on the supported Jaeger architecture and selected backend.

---

# 38. Elasticsearch / OpenSearch Architecture

A supported external search-storage architecture can look like:

```text
OTel Collector
      ↓
Jaeger
      ↓
Elasticsearch / OpenSearch
      ↓
Jaeger Query
      ↓
Jaeger UI
```

Storage should be independently planned for:

```text
Capacity
Replication
Retention
Performance
Backup
Recovery
```

---

# 39. Storage Outside Kubernetes

In AWS, storage can be operated separately from the EKS cluster.

Conceptually:

```text
EKS
│
└── Jaeger
      ↓
Private Network
      ↓
Trace Storage
```

Advantages:

```text
Independent lifecycle
Dedicated storage resources
Independent scaling
Potentially easier recovery
```

The actual storage service should be selected according to the organization's supported architecture.

---

# 40. Storage Credentials

If Jaeger requires credentials:

```text
Kubernetes Secret
       ↓
Jaeger
       ↓
Storage
```

Do not commit credentials into Git.

For GitOps:

```text
Git
 ↓
Secret reference
 ↓
External Secret System
 ↓
Kubernetes Secret
```

---

# 41. NetworkPolicy

A production namespace can restrict traffic.

Desired flow:

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

NetworkPolicy should prevent unnecessary communication between unrelated workloads.

---

# 42. Example Network Model

```text
Production Namespace
        │
        ↓
OTel Agent
        │
        ↓
Observability Namespace
        │
        ↓
OTel Gateway
        │
        ↓
Jaeger
        │
        ↓
Storage
```

Only required ports and sources should be allowed.

---

# 43. Security Groups in EKS

If the trace backend is outside the cluster:

```text
EKS Nodes / Pods
      ↓
Security Group
      ↓
Trace Storage
```

Allow only required traffic.

Do not open the storage backend broadly to the internet.

---

# 44. Private Subnets

For production:

```text
Internet
   X
   │
Private EKS
   │
Jaeger
   │
Private Storage
```

Tracing components generally do not need to be publicly accessible.

---

# 45. TLS

Use TLS where required by the security architecture.

```text
Collector
    ↓
TLS
    ↓
Jaeger / Storage
```

For external storage:

```text
Jaeger
    ↓
TLS
    ↓
Elasticsearch / OpenSearch
```

Certificates should be managed securely.

---

# 46. Jaeger UI Access

For development:

```text
kubectl port-forward
```

For production:

```text
Engineer
   ↓
VPN / Secure Access
   ↓
Internal Ingress
   ↓
Jaeger UI
```

Do not expose Jaeger UI directly through a public LoadBalancer without appropriate authentication and security controls.

---

# 47. Ingress Architecture

A production UI can use an internal ingress:

```text
Engineer
    ↓
Internal ALB / Ingress
    ↓
Jaeger Query / UI
```

Security controls can include:

```text
TLS
Authentication
Authorization
Security Groups
Network restrictions
```

---

# 48. Horizontal Scaling

Suppose:

```text
Trace volume = 10,000 spans/sec
```

and one Collector cannot handle the workload.

Scale:

```text
Gateway-1
Gateway-2
Gateway-3
```

Flow:

```text
OTel Agents
     ↓
Service
     ↓
Gateway replicas
```

Kubernetes distributes traffic among healthy Pods.

---

# 49. Autoscaling

HPA can be considered for components whose workload changes significantly.

Possible signals:

```text
CPU
Memory
Custom metrics
```

For example:

```text
Trace volume ↑
    ↓
Collector CPU ↑
    ↓
HPA
    ↓
More replicas
```

Autoscaling should be tested carefully because telemetry workloads can change rapidly.

---

# 50. Collector Queue

Use a bounded queue where appropriate.

```text
Collector
   ↓
Queue
   ↓
Jaeger
```

If Jaeger is temporarily unavailable:

```text
Jaeger
   X
```

the Collector can temporarily buffer data.

The queue must remain bounded to avoid uncontrolled memory consumption.

---

# 51. Collector Retry

Transient failures can use retry:

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

Configure reasonable:

```text
Retry interval
Maximum retry interval
Maximum elapsed time
```

---

# 52. Collector Backpressure

If incoming telemetry exceeds backend capacity:

```text
Incoming spans
       ↓
Collector
       ↓
Backend capacity exceeded
```

Possible controls:

```text
Sampling
Batching
Queue
Scaling
Backpressure
Filtering
```

Without controls, memory usage can grow rapidly.

---

# 53. Sampling in Kubernetes

A production Kubernetes environment may use:

```text
Applications
     ↓
OTel Agent
     ↓
OTel Gateway
     ↓
Tail Sampling
     ↓
Jaeger
```

Example:

```text
HTTP 500 → Keep
Latency > 1s → Keep
Normal requests → Sample
```

This reduces trace storage while retaining important traces.

---

# 54. Kubernetes Health Checks

Verify:

```bash
kubectl get pods -n observability
```

Then:

```bash
kubectl get svc -n observability
```

Then:

```bash
kubectl get endpoints -n observability
```

Then:

```bash
kubectl get events -n observability --sort-by=.lastTimestamp
```

These commands provide a quick health overview.

---

# 55. Check Jaeger Pods

Run:

```bash
kubectl get pods -n observability
```

Expected:

```text
NAME                       READY   STATUS
jaeger-query-xxxxx         1/1     Running
```

The actual resource names depend on the deployment.

---

# 56. Describe a Pod

If a Pod is not healthy:

```bash
kubectl describe pod <pod-name> -n observability
```

Check:

```text
Events
Container status
Readiness
Liveness
Resource issues
Mounts
Scheduling
```

---

# 57. Check Logs

Run:

```bash
kubectl logs <pod-name> -n observability
```

For the previous crashed container:

```bash
kubectl logs <pod-name> -n observability --previous
```

Look for:

```text
Configuration errors
Storage errors
Connection failures
Authentication failures
TLS errors
Port errors
```

---

# 58. Check Services

Run:

```bash
kubectl get svc -n observability
```

Verify:

```text
Service name
ClusterIP
Ports
Selectors
```

Then:

```bash
kubectl describe svc <service-name> -n observability
```

---

# 59. Check Endpoints

Run:

```bash
kubectl get endpoints -n observability
```

If a Service has no endpoints:

```text
Service
   ↓
No endpoints
```

Check:

```text
Pod labels
Service selector
Pod readiness
Pod status
```

---

# 60. DNS Troubleshooting

Create a temporary diagnostic Pod if needed.

Conceptually:

```bash
kubectl run dns-test \
  --image=busybox \
  -n observability \
  --rm -it -- sh
```

Then test:

```bash
nslookup <service-name>
```

or use another available DNS utility.

This verifies Kubernetes DNS resolution.

---

# 61. Connectivity Testing

From a diagnostic container:

```text
Application
    ↓
Collector Service
```

Test the appropriate port using a tool available in the container.

For example:

```bash
wget -qO- http://<service>:<port>
```

or another suitable connectivity test.

The test must use the correct protocol for the endpoint.

---

# 62. No Traces in Jaeger

Use the following path:

```text
Application
   ↓
OTel SDK
   ↓
OTLP
   ↓
Agent
   ↓
Gateway
   ↓
Jaeger
   ↓
Storage
   ↓
Query
   ↓
UI
```

Check each stage independently.

---

# 63. Application Cannot Send Traces

Check:

```text
OTEL_EXPORTER_OTLP_ENDPOINT
OTLP protocol
Service DNS
Service port
NetworkPolicy
Collector receiver
Application instrumentation
```

Example:

```text
Application
    ↓
otel-gateway.observability.svc.cluster.local:4317
```

Verify that the Service actually exposes the expected port.

---

# 64. Collector Cannot Reach Jaeger

Check:

```text
Jaeger Service
Service port
Endpoints
DNS
NetworkPolicy
TLS
Authentication
Jaeger health
```

Commands:

```bash
kubectl get svc -n observability
kubectl get endpoints -n observability
```

Then inspect Collector logs.

---

# 65. Jaeger Cannot Reach Storage

Check:

```text
Storage endpoint
DNS
Port
Credentials
TLS
Security Groups
NetworkPolicy
Storage health
```

A Jaeger Pod being `Running` does not guarantee that storage connectivity is healthy.

---

# 66. Jaeger UI Shows No Data

Check:

```text
Query health
Storage
Time range
Service filter
Sampling
Trace ingestion
```

Follow:

```text
UI
 ↓
Query
 ↓
Storage
```

Then trace backward:

```text
Storage
 ↓
Jaeger
 ↓
Collector
 ↓
Application
```

---

# 67. CrashLoopBackOff

If Jaeger or Collector is crashing:

```bash
kubectl logs <pod> -n observability --previous
```

Then:

```bash
kubectl describe pod <pod> -n observability
```

Common causes:

```text
Invalid configuration
Incorrect image
Storage configuration
Missing secret
Permission issue
Resource exhaustion
Version incompatibility
```

---

# 68. OOMKilled

If:

```text
Reason: OOMKilled
```

check:

```text
Trace volume
Sampling
Queue size
Batch size
Memory limiter
Container memory limit
```

Do not simply increase memory without investigating why memory usage increased.

---

# 69. Pending Pods

If a Jaeger or Collector Pod is:

```text
Pending
```

run:

```bash
kubectl describe pod <pod-name> -n observability
```

Common causes:

```text
Insufficient CPU
Insufficient memory
Node taints
Node selectors
Affinity rules
Topology constraints
```

---

# 70. ImagePullBackOff

Check:

```bash
kubectl describe pod <pod-name> -n observability
```

Possible causes:

```text
Incorrect image
Incorrect tag
Registry access
Authentication
Network connectivity
```

Use version-pinned images.

---

# 71. Service Selector Problem

Suppose:

```text
Service
   ↓
No endpoints
```

Inspect Service:

```bash
kubectl describe svc <service> -n observability
```

Inspect Pod labels:

```bash
kubectl get pods --show-labels -n observability
```

The Service selector must match the intended Pod labels.

---

# 72. NetworkPolicy Problem

If the Service exists but traffic fails:

```text
Application
    X
Collector
```

Check:

```text
NetworkPolicy
Security Groups
Pod-to-Pod networking
Service port
Protocol
```

Temporarily testing connectivity in a controlled staging environment can help isolate NetworkPolicy issues.

---

# 73. Production Monitoring

Monitor Kubernetes Jaeger components using Prometheus.

Important signals:

```text
CPU
Memory
Pod restarts
Request rate
Query latency
Export errors
Dropped spans
Queue size
Storage health
```

Architecture:

```text
Jaeger / Collector
       ↓
Metrics
       ↓
Prometheus
       ↓
Grafana
```

---

# 74. Alerting

Useful alerts:

```text
Jaeger Pod unavailable
Collector exporter failures
High dropped spans
High Collector memory
High queue utilization
Storage unavailable
Query latency high
Query error rate high
Frequent Pod restarts
```

Alerts should be actionable.

---

# 75. Kubernetes Upgrade Strategy

Before upgrading Jaeger:

```text
Development
    ↓
Staging
    ↓
Production
```

Validate:

```text
Image version
Helm chart
Storage compatibility
Configuration
API compatibility
Collector compatibility
```

---

# 76. Rolling Upgrade

For replicated components:

```text
Old Pod
Old Pod
Old Pod
```

Upgrade gradually:

```text
New Pod
Old Pod
Old Pod
```

then:

```text
New Pod
New Pod
Old Pod
```

finally:

```text
New Pod
New Pod
New Pod
```

Readiness checks are important during the transition.

---

# 77. GitOps Deployment

A production EKS environment can use:

```text
Git
 ↓
Jaeger Helm Values
 ↓
Pull Request
 ↓
CI
 ↓
ArgoCD
 ↓
EKS
```

ArgoCD continuously compares desired state with cluster state.

---

# 78. GitOps Repository Structure

Example:

```text
observability/
├── jaeger/
│   ├── values-dev.yaml
│   ├── values-staging.yaml
│   └── values-prod.yaml
│
├── otel/
│   ├── agent-values.yaml
│   └── gateway-values.yaml
│
└── argocd/
    ├── jaeger.yaml
    └── otel.yaml
```

The exact repository structure depends on the team's GitOps conventions.

---

# 79. Configuration Change Through GitOps

Process:

```text
Engineer
   ↓
Modify values
   ↓
Git commit
   ↓
Pull Request
   ↓
Review
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
```

This provides:

```text
Audit trail
Version control
Rollback
Repeatability
Drift detection
```

---

# 80. Rollback

If a Jaeger deployment causes problems:

```text
Current version
      ↓
Problem
      ↓
Git revert
      ↓
ArgoCD
      ↓
Previous version
```

After rollback:

```text
Generate test traces
Verify UI
Check Collector
Check storage
```

---

# 81. Production EKS Architecture

A realistic architecture:

```text
                              AWS
                               │
                              EKS
                               │
       ┌───────────────────────┼───────────────────────┐
       ↓                       ↓                       ↓
  Application Pods        Application Pods        Application Pods
       │                       │                       │
       └───────────────────────┼───────────────────────┘
                               ↓
                     OTel Agent DaemonSet
                               ↓
                      OTel Gateway Service
                               ↓
                ┌──────────────┼──────────────┐
                ↓              ↓              ↓
             Gateway-1      Gateway-2      Gateway-3
                └──────────────┼──────────────┘
                               ↓
                         Tail Sampling
                               ↓
                             Jaeger
                               ↓
                       Durable Storage
                               ↓
                      Jaeger Query Pods
                               ↓
                         Internal UI
```

---

# 82. Multi-AZ Deployment

For production EKS:

```text
AZ-A
├── Application
├── OTel Agent
└── Gateway-1

AZ-B
├── Application
├── OTel Agent
└── Gateway-2

AZ-C
├── Application
├── OTel Agent
└── Gateway-3
```

This reduces the impact of an AZ failure.

---

# 83. Multi-Cluster Architecture

If multiple EKS clusters exist:

```text
EKS Production
      ↓
OTel Gateway
      │
      ├──────────┐
      ↓          ↓
EKS Staging   EKS Development
      ↓          ↓
    OTel        OTel
      └────┬─────┘
           ↓
      Central Tracing
```

Include cluster metadata:

```text
k8s.cluster.name
deployment.environment
```

---

# 84. Multi-Region Architecture

For multiple AWS regions:

```text
Region A
Applications
     ↓
OTel Gateway
     ↓
Tracing Backend


Region B
Applications
     ↓
OTel Gateway
     ↓
Tracing Backend
```

Consider:

```text
Cross-region traffic
Latency
Data residency
Cost
Failure isolation
```

---

# 85. Kubernetes Security Architecture

Production security:

```text
Applications
     ↓
Private Network
     ↓
OTel Collector
     ↓
Jaeger
     ↓
Private Storage
```

Controls:

```text
RBAC
NetworkPolicy
TLS
Security Groups
Private subnets
Secrets
Authentication
Authorization
```

---

# 86. Sensitive Trace Data

Do not collect:

```text
Passwords
Access tokens
Authorization headers
Private keys
Payment credentials
Sensitive request bodies
```

Use Collector processors and instrumentation configuration to filter sensitive information.

---

# 87. Capacity Planning

Estimate:

```text
Requests/sec
×
Spans/request
×
Sampling rate
```

Example:

```text
10,000 requests/sec
×
10 spans/request
=
100,000 spans/sec
```

At 10% sampling:

```text
100,000
×
10%
=
10,000 spans/sec
```

This determines Collector and storage capacity requirements.

---

# 88. Trace Retention

Retention should consider:

```text
Incident investigation
Storage cost
Compliance
Business requirements
```

Example:

```text
Production → 7–14 days
Staging → shorter
Development → minimal
```

These are examples only. Actual retention should be defined by the organization.

---

# 89. Kubernetes Resource Planning

Plan resources independently for:

```text
OTel Agent
OTel Gateway
Jaeger
Query
Storage
```

Example:

```text
Node
│
├── Application
├── OTel Agent
└── Other workloads
```

Avoid allocating so many observability resources that they compete aggressively with application workloads.

---

# 90. Node Pressure

If the node becomes resource constrained:

```text
CPU pressure
Memory pressure
Disk pressure
```

the Collector or Jaeger components may become unstable.

Monitor:

```bash
kubectl top nodes
kubectl top pods -n observability
```

Use these together with Prometheus/Grafana for historical analysis.

---

# 91. Pod Eviction

Under node pressure, Kubernetes may evict Pods.

If tracing components are evicted:

```text
Application
   ↓
Collector unavailable
   ↓
Traces dropped
```

Use:

```text
Resource requests
Multiple replicas
Pod distribution
Priority planning
```

where appropriate.

---

# 92. High Availability Design

Avoid:

```text
One Collector
One Jaeger
One Query
One storage node
```

A better design:

```text
Multiple Agents
Multiple Gateways
Multiple Query replicas
Highly available storage
```

Every critical dependency should be evaluated for single points of failure.

---

# 93. Failure Scenario: Gateway Pod Failure

Suppose:

```text
Gateway-1
   X
```

The Kubernetes Service routes traffic to:

```text
Gateway-2
Gateway-3
```

Applications should continue sending telemetry.

This requires:

```text
Multiple replicas
Readiness probes
Service discovery
Proper load balancing
```

---

# 94. Failure Scenario: Jaeger Query Failure

Suppose:

```text
Query-1
   X
```

Traffic can continue to:

```text
Query-2
Query-3
```

The UI should remain available if the remaining Query replicas and storage are healthy.

---

# 95. Failure Scenario: Storage Failure

Storage failure is more serious:

```text
Collector
   ↓
Jaeger
   ↓
Storage
   X
```

Possible consequences:

```text
Trace ingestion failures
Query failures
Dropped traces
Queue growth
```

Storage must therefore be treated as a critical production dependency.

---

# 96. Failure Scenario: Collector Overload

```text
Trace volume
     ↑
Collector CPU
     ↑
Queue
     ↑
Memory
```

Response:

```text
Increase replicas
Adjust sampling
Tune batching
Control queue
Investigate traffic spike
```

Do not blindly increase memory without addressing the underlying workload.

---

# 97. Jaeger Kubernetes Troubleshooting Workflow

Use:

```text
1. kubectl get pods
2. kubectl get svc
3. kubectl get endpoints
4. kubectl describe pod
5. kubectl logs
6. Check Collector
7. Check Jaeger
8. Check storage
9. Generate test traffic
10. Check Jaeger UI
```

Then correlate with:

```text
Prometheus
Grafana
ELK
Kubernetes events
```

---

# 98. Real-World EKS Example

Suppose an EKS microservices application contains:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

Tracing architecture:

```text
                   EKS
                    │
     ┌──────────────┼──────────────┐
     ↓              ↓              ↓
  Orders          Payment       Inventory
     │              │              │
     └──────────────┼──────────────┘
                    ↓
              OTel Agent
                    ↓
              OTel Gateway
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

# 99. Real-World Incident

Users report:

```text
Checkout is slow.
```

Metrics:

```text
orders p95:
300ms → 1.4s
```

Jaeger:

```text
Orders
   ↓
Payment
   ↓
External Payment API
   ↓
1.1 seconds
```

ELK:

```text
Payment timeout warnings
```

Kubernetes:

```text
Payment Pods healthy
```

Conclusion:

```text
External Payment dependency
```

The tracing architecture helps isolate the problem without incorrectly blaming Kubernetes.

---

# 100. Production Deployment Checklist

```text
KUBERNETES
[ ] Dedicated observability namespace
[ ] Services configured
[ ] DNS verified
[ ] Resource requests configured
[ ] Resource limits configured
[ ] Readiness probes
[ ] Liveness probes
[ ] Startup probes where required
[ ] PDB
[ ] Pod anti-affinity
[ ] Topology spread

COLLECTOR
[ ] Agent DaemonSet
[ ] Gateway Deployment
[ ] OTLP receiver
[ ] Batch
[ ] Memory limiter
[ ] Sampling
[ ] Retry
[ ] Queue
[ ] Kubernetes metadata
[ ] RBAC

JAEGER
[ ] Ingestion configured
[ ] Query configured
[ ] UI configured
[ ] Storage configured
[ ] Multiple replicas where required

SECURITY
[ ] NetworkPolicy
[ ] TLS
[ ] Secrets
[ ] RBAC
[ ] Private networking
[ ] UI authentication
[ ] Sensitive-data filtering

OPERATIONS
[ ] Prometheus monitoring
[ ] Grafana dashboards
[ ] Alerts
[ ] Capacity planning
[ ] Retention
[ ] Backup/recovery
[ ] Failure testing

DEPLOYMENT
[ ] GitOps
[ ] Version pinning
[ ] Staging validation
[ ] Rolling upgrade
[ ] Rollback
```

---

# 101. Final Mental Model

The Kubernetes Jaeger architecture can be remembered as:

```text
                         EKS
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
       Application Pods          Application Pods
              │                       │
              └───────────┬───────────┘
                          ↓
                   OTel Agent
                    DaemonSet
                          ↓
                  OTel Gateway
                   Deployment
                          ↓
                 Process / Sample
                          ↓
                       Jaeger
                          ↓
                  Durable Storage
                          ↓
                    Jaeger Query
                          ↓
                     Jaeger UI
```

For a production DevOps environment, the important design is:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTel Agent
     ↓
OTel Gateway
     ↓
Batch / Enrich / Sample
     ↓
Jaeger
     ↓
Durable Storage
     ↓
Jaeger Query
     ↓
Secure Jaeger UI
```

The key production principles are **Kubernetes Service-based discovery, dedicated observability namespaces, Agent/Gateway Collector architecture, resource controls, health probes, Pod disruption protection, multi-replica deployment, multi-AZ distribution, durable storage, RBAC, NetworkPolicy, TLS, secure UI access, controlled sampling, monitoring, GitOps deployment, and tested rollback/failure recovery**.
