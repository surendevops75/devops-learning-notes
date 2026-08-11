# Jaeger Installation

## 1. Overview

Jaeger can be installed in different environments depending on the purpose.

Common approaches include:

```text
Local Development
Docker
Docker Compose
Kubernetes
Helm
OpenTelemetry Collector
Managed / External Infrastructure
```

For production Kubernetes environments, the preferred architecture is generally:

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

For learning and local testing, a simpler deployment can be used.

---

# 2. Installation Strategy

Choose the installation method based on the environment.

| Environment      | Recommended Approach                          |
| ---------------- | --------------------------------------------- |
| Developer laptop | Docker                                        |
| Local Kubernetes | Helm / Kubernetes                             |
| Development EKS  | Helm / GitOps                                 |
| Production EKS   | Helm / GitOps + HA architecture               |
| Large production | Collector + scalable Jaeger + durable storage |

Do not use a single-container development deployment as the production architecture.

---

# 3. Jaeger Installation Components

A Jaeger deployment can contain:

```text
Jaeger Collector / Ingestion
Jaeger Query
Jaeger UI
Storage
```

When OpenTelemetry is used:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OpenTelemetry Collector
     ↓
Jaeger
```

The exact Jaeger component layout depends on the Jaeger release and deployment mode.

---

# 4. Verify Docker

For local installation, first verify Docker:

```bash
docker --version
```

Then:

```bash
docker info
```

Docker must be running before starting Jaeger.

---

# 5. Running Jaeger Locally

For local experimentation, Jaeger provides an all-in-one style deployment.

Conceptually:

```text
Docker
  ↓
Jaeger
  ├── Ingestion
  ├── Query
  └── UI
```

This is useful for:

```text
Learning
Instrumentation testing
Local application development
Trace visualization
```

---

# 6. Docker Installation

A local Jaeger container can be started with the official Jaeger image.

Example:

```bash
docker run --rm \
  --name jaeger \
  -p 16686:16686 \
  -p 4317:4317 \
  -p 4318:4318 \
  jaegertracing/jaeger:<tested-version>
```

Pin a tested version instead of using an unqualified `latest` tag in repeatable environments.

---

# 7. Exposed Ports

Common ports include:

```text
16686 → Jaeger UI
4317  → OTLP/gRPC
4318  → OTLP/HTTP
```

The exact ports exposed depend on the Jaeger version and configuration.

The important concept is:

```text
Application
    ↓
OTLP
    ↓
Jaeger
    ↓
16686
    ↓
Browser
```

---

# 8. Accessing Jaeger UI

After starting the container, open the Jaeger UI through:

```text
http://localhost:16686
```

The UI allows you to:

```text
Search traces
Select services
Select operations
Inspect spans
Analyze latency
Inspect errors
```

For production environments, the UI should not normally be exposed directly to the public internet.

---

# 9. Verify the Container

Check:

```bash
docker ps
```

Expected:

```text
jaeger
```

Inspect logs:

```bash
docker logs jaeger
```

If the container is not healthy, inspect the logs before troubleshooting the application.

---

# 10. Stop Jaeger

For local development:

```bash
docker stop jaeger
```

Because the container was started with:

```text
--rm
```

Docker removes the container after it stops.

This is suitable for temporary testing.

---

# 11. Docker Compose

For a repeatable local environment, Docker Compose can be used.

Example:

```yaml
services:
  jaeger:
    image: jaegertracing/jaeger:<tested-version>
    ports:
      - "16686:16686"
      - "4317:4317"
      - "4318:4318"
```

Start:

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

View logs:

```bash
docker compose logs jaeger
```

Stop:

```bash
docker compose down
```

---

# 12. Application Configuration

An OpenTelemetry application needs an exporter endpoint.

For OTLP/gRPC:

```text
http://localhost:4317
```

For OTLP/HTTP:

```text
http://localhost:4318
```

Example:

```bash
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

The application then sends telemetry through OTLP.

---

# 13. Service Name

Always configure a meaningful service name.

Example:

```bash
export OTEL_SERVICE_NAME=orders
```

For another service:

```bash
export OTEL_SERVICE_NAME=payment
```

This allows Jaeger to distinguish services.

---

# 14. Development Trace Flow

A local application can use:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTLP
     ↓
Jaeger
     ↓
Jaeger UI
```

For example:

```text
orders
  ↓
POST /orders
  ↓
Jaeger
```

The trace should appear under the `orders` service.

---

# 15. Verify Application Telemetry

After generating traffic:

```bash
curl http://localhost:8080/orders
```

Then open Jaeger UI.

Search for:

```text
Service = orders
```

If instrumentation is working, traces should appear.

---

# 16. Kubernetes Installation

For Kubernetes, Jaeger can be deployed inside the cluster.

Architecture:

```text
Kubernetes
│
├── Application
│
├── OpenTelemetry Collector
│
└── Jaeger
```

A production architecture should normally separate:

```text
Telemetry generation
Telemetry collection
Trace backend
Trace storage
```

---

# 17. Create Observability Namespace

Create a dedicated namespace:

```bash
kubectl create namespace observability
```

Verify:

```bash
kubectl get namespace observability
```

A dedicated namespace provides:

```text
Resource organization
RBAC separation
NetworkPolicy control
Operational clarity
```

---

# 18. Helm Installation

Helm is commonly used for Kubernetes observability components.

Verify Helm:

```bash
helm version
```

Add the appropriate Jaeger Helm repository or use the current supported Jaeger deployment method for the version being installed.

Then:

```bash
helm repo update
```

Always verify the chart and application version before deploying.

---

# 19. Helm Chart Discovery

List configured repositories:

```bash
helm repo list
```

Search available charts:

```bash
helm search repo jaeger
```

Review available versions:

```bash
helm search repo jaeger --versions
```

Choose a tested version rather than blindly installing the newest version.

---

# 20. Helm Values

Production configuration should be maintained in a values file.

Example:

```yaml
# values.yaml

resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 1
    memory: 1Gi
```

The actual fields depend on the Jaeger chart and version.

Do not copy values from one chart version into another without checking the chart documentation.

---

# 21. Helm Release

A generic installation pattern is:

```bash
helm upgrade --install jaeger \
  <jaeger-chart> \
  -n observability \
  -f values.yaml
```

Using `upgrade --install` makes the command suitable for both initial installation and subsequent upgrades.

---

# 22. Verify Helm Release

Run:

```bash
helm list -n observability
```

Then:

```bash
helm status jaeger -n observability
```

Check Kubernetes resources:

```bash
kubectl get all -n observability
```

---

# 23. Verify Jaeger Pods

Run:

```bash
kubectl get pods -n observability
```

Look for:

```text
Running
Ready
```

If Pods are not ready:

```bash
kubectl describe pod <pod-name> -n observability
```

Then inspect logs:

```bash
kubectl logs <pod-name> -n observability
```

---

# 24. Jaeger Services

Check Services:

```bash
kubectl get svc -n observability
```

You may see services corresponding to Jaeger components.

Applications should communicate through Kubernetes Services rather than directly using Pod IPs.

---

# 25. Kubernetes DNS

A Service can be accessed using Kubernetes DNS.

Example:

```text
jaeger-query.observability.svc.cluster.local
```

or another Service name based on the deployment.

For Collector-to-Jaeger communication:

```text
OTel Collector
      ↓
Kubernetes DNS
      ↓
Jaeger Service
```

---

# 26. Port Forwarding

For development, you can access the Jaeger UI without exposing it externally.

Example:

```bash
kubectl port-forward \
  svc/<jaeger-query-service> \
  16686:16686 \
  -n observability
```

Then access the UI locally.

This is preferable to creating a public LoadBalancer just for development access.

---

# 27. Verify UI

After port forwarding:

```text
Browser
   ↓
localhost:16686
   ↓
Kubernetes
   ↓
Jaeger Query
```

Open the UI and verify that the interface loads successfully.

---

# 28. OpenTelemetry Collector Integration

A recommended Kubernetes architecture is:

```text
Application Pods
      ↓
OTel SDK
      ↓
OTLP
      ↓
OTel Collector
      ↓
Jaeger
```

This prevents applications from becoming tightly coupled to the Jaeger backend.

---

# 29. Collector OTLP Receiver

Conceptual Collector configuration:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:
```

This allows applications to send OTLP telemetry to the Collector.

---

# 30. Collector Jaeger Export

The Collector can be configured with an appropriate Jaeger-compatible export path depending on the installed Collector and Jaeger versions.

Conceptually:

```yaml
exporters:
  <jaeger-compatible-exporter>:
    endpoint: <jaeger-endpoint>
```

The exact exporter must match the current OpenTelemetry Collector component availability.

Do not assume every historical Jaeger exporter remains available in every current Collector distribution.

---

# 31. Traces Pipeline

Conceptual pipeline:

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
        - <jaeger-exporter>
```

Flow:

```text
Application
    ↓
OTLP
    ↓
Collector
    ↓
Memory Limiter
    ↓
Batch
    ↓
Jaeger
```

---

# 32. Modern OTLP Architecture

A strong architecture is:

```text
Application
     ↓
OTLP
     ↓
OpenTelemetry Collector
     ↓
OTLP
     ↓
Jaeger
```

This provides a standardized telemetry protocol between components.

The exact Jaeger ingestion endpoint depends on the Jaeger version and configuration.

---

# 33. Why Use a Collector?

Without a Collector:

```text
Application
     ↓
Jaeger
```

With a Collector:

```text
Application
     ↓
Collector
     ↓
Jaeger
```

The Collector provides:

```text
Batching
Filtering
Sampling
Transformation
Enrichment
Retry
Routing
```

This becomes especially important in production.

---

# 34. Kubernetes Application Configuration

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: orders

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: http://otel-gateway.observability.svc.cluster.local:4317
```

The exact endpoint should match your Agent/Gateway architecture.

---

# 35. Agent Architecture

For larger Kubernetes environments:

```text
Node 1
├── Applications
└── OTel Agent

Node 2
├── Applications
└── OTel Agent

Node 3
├── Applications
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

The Agent is normally deployed as a DaemonSet.

---

# 36. Gateway Architecture

The Gateway is normally deployed as a Deployment.

```text
OTel Agent
     ↓
Kubernetes Service
     ↓
Gateway-1
Gateway-2
Gateway-3
     ↓
Jaeger
```

Multiple replicas provide better availability.

---

# 37. Jaeger Storage

Jaeger needs a storage backend appropriate for the deployment.

Production storage planning should consider:

```text
Trace volume
Sampling
Retention
Replication
Query workload
Storage capacity
Recovery
```

A development installation may use simplified storage, while production requires durable storage.

---

# 38. Storage Selection

A production design should choose storage based on:

```text
Scale
Availability
Operational expertise
Query performance
Retention
Cost
Backup requirements
```

Examples of storage technologies supported by Jaeger deployments have included:

```text
Elasticsearch
OpenSearch
Other supported backends depending on Jaeger version
```

Always verify compatibility with the specific Jaeger release.

---

# 39. Persistent Storage

If the selected deployment requires persistent storage, do not rely on ephemeral Pod storage.

Bad:

```text
Jaeger Pod
    ↓
Ephemeral storage
```

Better:

```text
Jaeger
   ↓
Durable Storage
```

Trace data should survive Pod restarts when the architecture requires persistent retention.

---

# 40. Production Storage Architecture

A conceptual production architecture:

```text
OTel Collector
      ↓
Jaeger Ingestion
      ↓
Durable Storage
      ↓
Jaeger Query
      ↓
Jaeger UI
```

Storage should have its own:

```text
High Availability
Capacity planning
Backup
Monitoring
Recovery strategy
```

---

# 41. Development vs Production

### Development

```text
Application
 ↓
Jaeger
 ↓
UI
```

Simple and fast.

### Production

```text
Applications
 ↓
OTel Agents
 ↓
OTel Gateways
 ↓
Sampling / Processing
 ↓
Jaeger
 ↓
Durable Storage
 ↓
Query
 ↓
UI
```

Production requires additional reliability and security controls.

---

# 42. EKS Installation Architecture

For an EKS microservices platform:

```text
                         AWS
                          │
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

# 43. Private EKS Deployment

Keep internal tracing traffic private where possible.

```text
Internet
   ↓
ALB
   ↓
Application
```

Tracing:

```text
Application
   ↓
OTel Agent
   ↓
OTel Gateway
   ↓
Jaeger
```

Telemetry does not need to be exposed to the public internet.

---

# 44. Security Groups

If the storage backend is outside the cluster or VPC boundaries, control access using AWS networking controls.

Conceptually:

```text
EKS
 ↓
Security Controls
 ↓
Trace Backend
```

Allow only the required ports and sources.

---

# 45. Kubernetes NetworkPolicy

Restrict communication to required components.

Example concept:

```text
Application
   ↓
OTel Agent
   ↓
OTel Gateway
   ↓
Jaeger
```

Applications should not necessarily have direct access to every Jaeger component.

---

# 46. RBAC

If Jaeger or the Collector needs Kubernetes API access, configure a dedicated ServiceAccount.

Conceptually:

```text
ServiceAccount
      ↓
RBAC
      ↓
Collector / Component
```

Grant only required permissions.

---

# 47. Secrets

Credentials should not be embedded in:

```text
Dockerfile
Git repository
Helm values committed as plaintext
Application source code
```

Use:

```text
Kubernetes Secrets
AWS Secrets Manager
External secret management
```

according to the platform architecture.

---

# 48. TLS

For communication across trust boundaries:

```text
Collector
   ↓
TLS
   ↓
Jaeger / Backend
```

Use certificates and validate the server identity.

For internal cluster traffic, whether TLS is required depends on the organization's security model and network trust boundaries.

---

# 49. Jaeger UI Security

Do not expose the Jaeger UI directly to the public internet without an appropriate security layer.

Preferred:

```text
User
 ↓
VPN / Private Network / Secure Access
 ↓
Ingress / Proxy
 ↓
Jaeger UI
```

or an organization-approved authentication gateway.

---

# 50. Resource Requests

Set resource requests for production components.

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
```

The correct values depend on:

```text
Trace volume
Number of services
Sampling rate
Storage performance
Query workload
```

---

# 51. Resource Limits

Example:

```yaml
resources:
  limits:
    cpu: "1"
    memory: 1Gi
```

Do not copy these values blindly.

Measure actual usage in staging and production.

---

# 52. Health Probes

Kubernetes components should have appropriate:

```text
Liveness probe
Readiness probe
Startup probe where needed
```

Readiness is particularly important for components that should receive traffic only after initialization.

---

# 53. PodDisruptionBudget

For production replicas:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: jaeger-query
  namespace: observability
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: jaeger-query
```

The exact selector and values depend on the deployment.

---

# 54. Pod Distribution

Avoid placing all replicas on a single node.

Bad:

```text
Node-1
├── Jaeger Query 1
├── Jaeger Query 2
└── Jaeger Query 3
```

Better:

```text
Node-1 → Query 1
Node-2 → Query 2
Node-3 → Query 3
```

Use:

```text
Pod anti-affinity
Topology spread constraints
```

where appropriate.

---

# 55. Availability Zones

For production EKS:

```text
AZ-A → Jaeger component
AZ-B → Jaeger component
AZ-C → Jaeger component
```

This reduces the impact of a single Availability Zone failure.

The storage backend must also support the desired availability model.

---

# 56. Horizontal Scaling

Scale Query or ingestion components based on:

```text
CPU
Memory
Request rate
Trace ingestion
Query latency
Queue depth
```

Example:

```text
Trace volume ↑
     ↓
Gateway load ↑
     ↓
Scale Gateway
```

Scaling should be validated under realistic workloads.

---

# 57. Sampling During Installation

Do not initially configure unlimited trace collection in a high-volume production environment.

Start with an intentional sampling strategy.

Example:

```text
Errors → Keep
Slow traces → Keep
Normal traces → Sample
```

Then adjust based on:

```text
Storage
Cost
Incident requirements
Trace volume
```

---

# 58. Verification Workflow

After installation:

```text
1. Verify Pods
2. Verify Services
3. Verify endpoints
4. Verify Collector
5. Generate application traffic
6. Search Jaeger
7. Inspect trace
8. Verify storage
9. Verify Query
10. Verify UI
```

---

# 59. Verify Kubernetes Resources

Run:

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

Endpoints should point to healthy Pods.

---

# 60. Verify Collector

Check Collector Pods:

```bash
kubectl get pods -n observability -l app=otel-collector
```

Then:

```bash
kubectl logs <collector-pod> -n observability
```

Look for:

```text
Receiver started
Pipeline started
Exporter connected
```

and investigate any repeated errors.

---

# 61. Verify Jaeger

Check:

```bash
kubectl get pods -n observability -l app=jaeger
```

Then:

```bash
kubectl logs <jaeger-pod> -n observability
```

Verify:

```text
Pod ready
Service available
Storage reachable
```

---

# 62. Generate Test Traffic

Example:

```bash
curl http://<application-endpoint>/orders
```

Generate several requests:

```bash
for i in {1..10}; do
  curl -s http://<application-endpoint>/orders > /dev/null
done
```

Then search the Jaeger UI for the service.

---

# 63. Verify Trace Structure

A successful trace should look conceptually like:

```text
Frontend
   ↓
Orders
   ↓
Payment
   ↓
Database
```

Check:

```text
Trace ID
Span IDs
Parent-child relationships
Duration
Service names
Attributes
Status
```

---

# 64. Verify Trace Context

If multiple services are involved:

```text
Orders
   ↓
Payment
```

verify that both belong to the same Trace ID.

Expected:

```text
Trace ABC
├── Orders Span
└── Payment Span
```

If separate Trace IDs appear, investigate context propagation.

---

# 65. Installation Troubleshooting

### Pods Pending

Check:

```bash
kubectl describe pod <pod-name> -n observability
```

Look for:

```text
Insufficient CPU
Insufficient memory
Node selector mismatch
Taints
Affinity rules
```

---

# 66. CrashLoopBackOff

Run:

```bash
kubectl logs <pod-name> -n observability --previous
```

Then:

```bash
kubectl describe pod <pod-name> -n observability
```

Check:

```text
Configuration
Environment variables
Storage
Permissions
Resource limits
Version compatibility
```

---

# 67. ImagePullBackOff

Check:

```bash
kubectl describe pod <pod-name> -n observability
```

Common causes:

```text
Incorrect image
Incorrect tag
Registry access problem
Authentication failure
Network problem
```

Use tested, pinned image versions.

---

# 68. Service Not Reachable

Check:

```bash
kubectl get svc -n observability
```

Then:

```bash
kubectl get endpoints -n observability
```

If there are no endpoints:

```text
Service selector
      ↓
Does not match Pod labels
```

Fix the selector or labels.

---

# 69. Collector Cannot Reach Jaeger

Check:

```text
DNS
Service
Port
Protocol
TLS
NetworkPolicy
Jaeger health
```

From a diagnostic Pod:

```bash
kubectl exec -it <pod> -n observability -- sh
```

Then test connectivity to the appropriate Service and port.

---

# 70. No Traces in Jaeger

Follow the path:

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
 ↓
Storage
 ↓
Query
 ↓
UI
```

Check every stage.

Do not assume the UI is the problem just because no traces appear.

---

# 71. No Traces From One Service

Check:

```text
OTEL_SERVICE_NAME
OTEL_EXPORTER_OTLP_ENDPOINT
Instrumentation
Sampling
Application startup
Collector routing
```

Compare the working service with the failing service.

---

# 72. Traces Arrive but Are Incomplete

Check:

```text
Context propagation
Sampling
Instrumentation
Collector processing
Multiple telemetry pipelines
```

Example:

```text
Orders
 ↓
Payment
```

If Payment is missing:

```text
traceparent
```

propagation should be one of the first things investigated.

---

# 73. Installation Through GitOps

Production deployment:

```text
Git
 ↓
Jaeger configuration
 ↓
Pull Request
 ↓
Review
 ↓
CI validation
 ↓
ArgoCD
 ↓
EKS
```

This provides:

```text
Version control
Auditability
Rollback
Drift detection
Repeatability
```

---

# 74. Example Repository Structure

A possible repository:

```text
observability/
├── jaeger/
│   ├── values.yaml
│   ├── namespace.yaml
│   └── application.yaml
│
├── opentelemetry/
│   ├── collector-values.yaml
│   └── instrumentation.yaml
│
└── argocd/
    └── jaeger-application.yaml
```

The exact repository layout depends on the organization's GitOps standards.

---

# 75. CI Validation

Before deployment:

```text
Git
 ↓
CI
 ↓
YAML validation
 ↓
Helm validation
 ↓
Security scanning
 ↓
Review
 ↓
ArgoCD
```

Validate:

```text
Configuration syntax
Chart compatibility
Resource configuration
Security configuration
```

---

# 76. Staging Installation

Install first in staging:

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
Trace search
Sampling
Storage
Query performance
Failure scenarios
Resource consumption
```

---

# 77. Production Rollout

A safe rollout:

```text
New Version
    ↓
Canary
    ↓
Observe
    ↓
Expand
    ↓
Full rollout
```

Monitor:

```text
Ingestion
Errors
Latency
CPU
Memory
Storage
```

---

# 78. Rollback

If the new version causes problems:

```text
New Jaeger version
       ↓
Problem
       ↓
Rollback
       ↓
Previous version
```

With GitOps:

```text
Git
 ↓
Previous commit
 ↓
ArgoCD
 ↓
EKS
```

This makes rollback reproducible.

---

# 79. Installation Best Practices

```text
1. Pin Jaeger versions.
2. Pin container images.
3. Use Helm/GitOps for Kubernetes.
4. Use a dedicated observability namespace.
5. Use OpenTelemetry for application instrumentation.
6. Put a Collector between applications and backend where appropriate.
7. Configure resource requests and limits.
8. Use durable storage for production.
9. Secure the Jaeger UI.
10. Use TLS where required.
11. Protect sensitive trace attributes.
12. Configure sampling.
13. Monitor Jaeger itself.
14. Test failure scenarios.
15. Validate upgrades in staging.
16. Maintain rollback capability.
```

---

# 80. Production Installation Checklist

```text
[ ] Jaeger version selected
[ ] Storage architecture selected
[ ] Kubernetes namespace created
[ ] Helm/GitOps deployment configured
[ ] Resource requests configured
[ ] Resource limits configured
[ ] Health probes configured
[ ] PDB configured
[ ] Pod distribution configured
[ ] Multiple replicas configured where required
[ ] Storage persistence configured
[ ] Collector integration configured
[ ] OTLP endpoint configured
[ ] Sampling configured
[ ] Kubernetes metadata configured
[ ] TLS configured where required
[ ] Authentication configured where required
[ ] NetworkPolicy configured
[ ] UI access secured
[ ] Sensitive attributes filtered
[ ] Monitoring configured
[ ] Alerts configured
[ ] Backup/recovery considered
[ ] Staging validation completed
[ ] Rollback tested
```

---

# 81. Final Installation Architecture

For local development:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
Jaeger
     ↓
Jaeger UI
```

For Kubernetes development:

```text
Application
     ↓
OTel SDK
     ↓
OTel Collector
     ↓
Jaeger
     ↓
Jaeger UI
```

For production EKS:

```text
                         EKS
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Orders       Payment      Inventory
             │            │            │
          OTel SDK     OTel SDK     OTel SDK
             │            │            │
             └────────────┼────────────┘
                          ↓
                   OTel Agent
                    DaemonSet
                          ↓
                   OTel Gateway
                  Multiple Replicas
                          ↓
                     Sampling
                          ↓
                       Jaeger
                          ↓
                  Durable Storage
                          ↓
                    Jaeger Query
                          ↓
                      Jaeger UI
```

The key principle is:

**Install Jaeger differently depending on the environment. Docker or a simple all-in-one deployment is appropriate for learning and local development, while production Kubernetes should use version-pinned deployments, scalable Collector and Jaeger components, durable storage, secure networking, resource controls, sampling, monitoring, GitOps, and a tested rollback strategy.**
