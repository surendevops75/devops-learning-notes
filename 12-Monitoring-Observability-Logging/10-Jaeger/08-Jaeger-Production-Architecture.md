# Jaeger Production Architecture

## 1. Overview

Jaeger production architecture should be designed for:

```text
High availability
Scalability
Durability
Security
Controlled trace volume
Fault tolerance
Operational visibility
```

A production architecture should avoid placing the entire tracing system into a single Pod or single node.

A typical architecture is:

```text
Applications
      ↓
OpenTelemetry SDK
      ↓
OTel Collector Agents
      ↓
OTel Collector Gateways
      ↓
Processing / Sampling
      ↓
Jaeger
      ↓
Durable Trace Storage
      ↓
Jaeger Query
      ↓
Secure Jaeger UI
```

---

# 2. Production Architecture

A Kubernetes/EKS production design can look like:

```text
                           EKS Cluster
                                │
       ┌────────────────────────┼────────────────────────┐
       ↓                        ↓                        ↓
 Application Pods         Application Pods         Application Pods
       │                        │                        │
       └────────────────────────┼────────────────────────┘
                                ↓
                       OTel Agent DaemonSet
                                ↓
                       OTel Gateway Service
                                ↓
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
          Gateway-1         Gateway-2         Gateway-3
              └─────────────────┼─────────────────┘
                                ↓
                      Processing / Sampling
                                ↓
                             Jaeger
                                ↓
                       Durable Storage
                                ↓
                  ┌─────────────┼─────────────┐
                  ↓             ↓             ↓
               Query-1       Query-2       Query-3
                  └─────────────┼─────────────┘
                                ↓
                         Internal Ingress
                                ↓
                           Jaeger UI
```

---

# 3. Core Production Principles

A production Jaeger architecture should follow these principles:

```text
1. Do not rely on a single replica.
2. Separate collection from querying.
3. Use durable storage.
4. Control trace volume.
5. Protect the UI.
6. Distribute workloads across nodes/AZs.
7. Monitor the entire telemetry pipeline.
8. Use resource requests and limits.
9. Protect against memory pressure.
10. Maintain tested rollback and recovery procedures.
```

---

# 4. Application Layer

Applications generate traces using OpenTelemetry.

Example:

```text
Orders
Payment
Inventory
Notification
User
```

Each application should have:

```text
service.name
service.version
deployment.environment
```

Example:

```text
service.name=payment
service.version=2.4.1
deployment.environment=production
```

These attributes are important for production troubleshooting.

---

# 5. OpenTelemetry SDK

The SDK generates spans inside applications.

Example:

```text
HTTP Request
     ↓
Create Span
     ↓
Database Span
     ↓
Payment Span
     ↓
Complete Trace
```

The SDK exports telemetry using OTLP.

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTLP
```

---

# 6. Collector Agent Layer

In Kubernetes, a common architecture is to deploy an OpenTelemetry Collector Agent as a DaemonSet.

```text
Node-1 → OTel Agent
Node-2 → OTel Agent
Node-3 → OTel Agent
```

This provides a local collection layer.

The Agent then forwards telemetry to the Gateway.

```text
Application
    ↓
OTel Agent
    ↓
OTel Gateway
```

---

# 7. Collector Gateway Layer

The Gateway is deployed as a scalable Deployment.

Example:

```text
OTel Gateway
├── Pod-1
├── Pod-2
└── Pod-3
```

A Kubernetes Service distributes traffic:

```text
OTel Agents
     ↓
Kubernetes Service
     ↓
Gateway replicas
```

This avoids depending on a single Collector Pod.

---

# 8. Why Separate Agent and Gateway?

The separation provides:

```text
Local collection
Centralized processing
Centralized sampling
Centralized routing
Independent scaling
Better fault isolation
```

Architecture:

```text
Application
    ↓
Agent
    ↓
Gateway
    ↓
Jaeger
```

The Agent handles local collection while the Gateway handles centralized processing.

---

# 9. Collector Processing Pipeline

A production Gateway may use:

```text
OTLP Receiver
      ↓
Memory Limiter
      ↓
Kubernetes Attributes
      ↓
Filtering
      ↓
Sampling
      ↓
Batch
      ↓
Retry / Queue
      ↓
Jaeger
```

Each stage solves a different problem.

---

# 10. Memory Protection

A Collector must be protected against sudden traffic spikes.

Example:

```text
Trace volume ↑
      ↓
Collector memory ↑
      ↓
Memory protection
      ↓
Backpressure / controlled dropping
```

The Memory Limiter helps prevent uncontrolled memory consumption.

This is especially important in Kubernetes where excessive memory consumption can result in:

```text
OOMKilled
Pod restart
Trace loss
Node pressure
```

---

# 11. Batch Processing

Batching reduces the overhead of sending individual spans.

Without batching:

```text
Span → Export
Span → Export
Span → Export
Span → Export
```

With batching:

```text
Span ┐
Span ├──→ Batch → Export
Span │
Span ┘
```

Batching should be tuned according to workload and latency requirements.

---

# 12. Retry and Queue

Temporary backend failures can cause telemetry export errors.

Example:

```text
Collector
    ↓
Jaeger
    X
```

With retry:

```text
Collector
    ↓
Retry Queue
    ↓
Jaeger
    ✓
```

A bounded queue can provide temporary buffering.

The queue must be bounded because an unbounded queue can consume excessive memory.

---

# 13. Sampling Architecture

Production environments can generate enormous trace volumes.

Example:

```text
100,000 spans/sec
```

Storing every span may be unnecessary.

Sampling can reduce the volume:

```text
Errors      → Keep
Slow traces → Keep
Normal      → Sample
```

This reduces:

```text
Storage cost
Network traffic
Collector workload
Backend workload
```

---

# 14. Tail Sampling

Tail sampling is useful when the sampling decision depends on the complete trace.

Architecture:

```text
Trace
  ↓
Collector
  ↓
Observe complete trace
  ↓
Evaluate policy
  ├── Error → Keep
  ├── Slow → Keep
  └── Normal → Sample
```

This can improve the probability of retaining valuable incident traces.

---

# 15. Trace Storage

Storage is one of the most important parts of the production architecture.

```text
Jaeger
   ↓
Durable Storage
```

Storage should be designed for:

```text
High ingestion rate
Query performance
Retention
Replication
Capacity
Recovery
```

The exact storage backend depends on the Jaeger architecture and organization's infrastructure standards.

---

# 16. Storage Should Not Be a Single Point of Failure

Avoid:

```text
Jaeger
   ↓
Single storage node
```

A production design should use storage with appropriate:

```text
Replication
Failure recovery
Durability
Capacity
Backup
```

Jaeger Query availability does not compensate for unavailable trace storage.

---

# 17. External Storage Architecture

Storage can be operated outside the EKS cluster.

```text
EKS
 │
 └── Jaeger
       ↓
    Private Network
       ↓
 Durable Storage
```

Advantages can include:

```text
Independent scaling
Independent lifecycle
Dedicated storage resources
Simpler recovery strategy
```

---

# 18. Query Layer

Jaeger Query retrieves traces from storage.

Production architecture:

```text
Storage
   ↓
Query
├── Query-1
├── Query-2
└── Query-3
```

Multiple Query replicas prevent a single Query Pod from becoming a single point of failure.

---

# 19. Query Scaling

Query workload can increase when:

```text
More engineers use Jaeger
More traces are stored
Searches become more frequent
Incident investigation starts
```

Scale Query independently:

```text
Query load ↑
     ↓
Query replicas ↑
```

However, Query scaling does not solve an overloaded storage backend.

---

# 20. Jaeger UI

The UI should be treated as a user-facing observability component.

Production flow:

```text
Engineer
   ↓
Secure Access
   ↓
Internal Ingress
   ↓
Jaeger UI / Query
   ↓
Storage
```

The UI should not normally be exposed directly to the public internet.

---

# 21. Secure UI Architecture

A secure design can be:

```text
Engineer
   ↓
VPN / Corporate Access
   ↓
Internal ALB / Ingress
   ↓
Authentication
   ↓
Jaeger UI
```

Possible controls:

```text
TLS
Authentication
Authorization
Security Groups
NetworkPolicy
Private networking
```

---

# 22. Kubernetes Namespace

Use a dedicated namespace:

```bash
kubectl create namespace observability
```

Architecture:

```text
observability
├── OTel Agent
├── OTel Gateway
├── Jaeger
├── Query
└── Supporting resources
```

This simplifies:

```text
RBAC
NetworkPolicy
Resource management
Monitoring
Operations
```

---

# 23. Kubernetes Service Discovery

Applications should use Kubernetes Services.

Example:

```text
otel-gateway.observability.svc.cluster.local
```

Avoid hardcoding Pod IP addresses.

Pod IPs can change after:

```text
Pod restart
Deployment rollout
Node replacement
Scaling
```

---

# 24. Resource Requests

Production components should have resource requests.

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
```

The actual values should be based on workload measurements.

Requests help Kubernetes make scheduling decisions.

---

# 25. Resource Limits

Example:

```yaml
resources:
  limits:
    cpu: "1"
    memory: 1Gi
```

Limits should be chosen carefully.

Too low:

```text
OOMKilled
Throttling
Restart
```

Too high:

```text
Poor cluster utilization
Scheduling difficulties
```

---

# 26. Horizontal Scaling

Collector Gateways can be scaled horizontally.

```text
                Service
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   Gateway-1   Gateway-2   Gateway-3
```

This improves:

```text
Capacity
Availability
Fault tolerance
```

---

# 27. Pod Distribution

Do not place every replica on one node.

Bad:

```text
Node-1
├── Gateway-1
├── Gateway-2
└── Gateway-3
```

Better:

```text
Node-1 → Gateway-1
Node-2 → Gateway-2
Node-3 → Gateway-3
```

Use:

```text
Pod anti-affinity
Topology spread constraints
```

where appropriate.

---

# 28. Multi-AZ Architecture

For EKS production:

```text
Availability Zone A
├── Gateway
└── Query

Availability Zone B
├── Gateway
└── Query

Availability Zone C
├── Gateway
└── Query
```

This improves resilience against an Availability Zone failure.

---

# 29. High Availability

A production tracing platform should avoid:

```text
One Agent
One Gateway
One Jaeger instance
One Query
One storage node
```

Instead:

```text
Multiple Agents
Multiple Gateways
Multiple Query replicas
Highly available storage
```

Every component should be evaluated for failure scenarios.

---

# 30. Failure Scenario: Gateway Pod Failure

Suppose:

```text
Gateway-1
    X
```

The Service can route traffic to:

```text
Gateway-2
Gateway-3
```

This requires:

```text
Multiple replicas
Readiness probes
Correct Service selectors
```

---

# 31. Failure Scenario: Query Pod Failure

Suppose:

```text
Query-1
   X
```

Remaining replicas continue:

```text
Query-2
Query-3
```

The UI can continue working if:

```text
Storage is healthy
Other Query replicas are healthy
Ingress is healthy
```

---

# 32. Failure Scenario: Collector Gateway Overload

Suppose:

```text
Trace volume ↑
     ↓
Gateway CPU ↑
     ↓
Queue ↑
     ↓
Memory ↑
```

Possible actions:

```text
Increase Gateway replicas
Tune sampling
Tune batching
Control queue size
Investigate traffic increase
```

---

# 33. Failure Scenario: Storage Unavailable

```text
Collector
    ↓
Jaeger
    ↓
Storage
    X
```

Possible effects:

```text
Export failures
Trace loss
Queue growth
Query failures
```

Storage availability is therefore critical.

---

# 34. Failure Scenario: Node Failure

Suppose:

```text
Node-1
   X
```

If Gateway replicas are distributed:

```text
Node-2 → Gateway-2
Node-3 → Gateway-3
```

the tracing pipeline can continue.

This is why Pod distribution matters.

---

# 35. Failure Scenario: Availability Zone Failure

Suppose:

```text
AZ-A
   X
```

A multi-AZ architecture keeps workloads running in:

```text
AZ-B
AZ-C
```

provided the underlying storage and networking architecture also support the required failure model.

---

# 36. PodDisruptionBudget

A PodDisruptionBudget can protect availability during voluntary disruptions.

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: otel-gateway
  namespace: observability
spec:
  minAvailable: 2
```

The exact value depends on replica count and availability requirements.

---

# 37. Health Probes

Use:

```text
Startup Probe
Readiness Probe
Liveness Probe
```

Conceptually:

```text
Container starts
      ↓
Startup
      ↓
Ready
      ↓
Receive traffic
```

If a Pod becomes unhealthy:

```text
Readiness → Remove from Service
Liveness → Restart if appropriate
```

---

# 38. Readiness Is Important

Suppose a Gateway is running but cannot process traffic.

```text
Pod Status = Running
Application Health = Bad
```

Without readiness:

```text
Service
  ↓
Unhealthy Pod
```

With readiness:

```text
Service
  ↓
Healthy Pods only
```

---

# 39. Network Security

A production architecture should restrict communication.

Example:

```text
Applications
     ↓
OTel Gateway
     ↓
Jaeger
     ↓
Storage
```

NetworkPolicy should allow only required traffic.

---

# 40. Security Groups

If storage is outside EKS:

```text
EKS
 ↓
Security Group
 ↓
Private Storage
```

Allow only the required source, destination, protocol, and port.

Avoid:

```text
0.0.0.0/0
```

for internal tracing infrastructure unless there is an explicitly justified requirement.

---

# 41. TLS

Secure communication where required:

```text
Application
    ↓
TLS
    ↓
Collector
```

and:

```text
Jaeger
    ↓
TLS
    ↓
Storage
```

Certificates should be managed through the organization's approved certificate-management solution.

---

# 42. Secrets Management

Credentials should not be hardcoded.

Bad:

```yaml
password: mypassword123
```

Better:

```text
Secret
   ↓
Jaeger
```

For GitOps environments:

```text
Git
 ↓
Secret reference
 ↓
External Secret mechanism
 ↓
Kubernetes Secret
```

---

# 43. RBAC

Create dedicated ServiceAccounts.

Example:

```text
OTel Collector
     ↓
ServiceAccount
     ↓
RBAC
```

Grant only required Kubernetes permissions.

Avoid:

```text
cluster-admin
```

unless there is a specific justified requirement.

---

# 44. Kubernetes Metadata

Production traces should ideally contain useful Kubernetes metadata.

Example:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
```

Then a trace can lead directly to:

```text
Service
 ↓
Pod
 ↓
Node
```

This improves incident investigation.

---

# 45. Trace-to-Log Correlation

Example:

```text
Jaeger
  ↓
Trace ID
  ↓
Kibana
  ↓
Application logs
```

A production log entry might contain:

```text
trace_id=abc123
span_id=def456
```

This allows engineers to correlate the same request across systems.

---

# 46. Trace-to-Metric Correlation

Metrics:

```text
Prometheus
   ↓
High latency
```

Tracing:

```text
Jaeger
   ↓
Slow Payment span
```

Logs:

```text
ELK
   ↓
Database timeout
```

Complete investigation:

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

# 47. Prometheus Monitoring

Monitor the tracing infrastructure itself.

Important metrics include:

```text
Collector CPU
Collector memory
Collector queue size
Export failures
Dropped spans
Jaeger query latency
Jaeger errors
Storage health
Pod restarts
```

The observability platform must itself be observable.

---

# 48. Grafana Dashboards

A production dashboard can contain:

```text
Collector
├── CPU
├── Memory
├── Receive rate
├── Export rate
├── Export errors
├── Dropped spans
└── Queue size

Jaeger
├── Query latency
├── Query errors
├── Ingestion
└── Storage health
```

---

# 49. Alerting

Useful alerts:

```text
Collector unavailable
Gateway replicas below minimum
High export error rate
High dropped span rate
Collector memory high
Collector queue near capacity
Jaeger Query unavailable
Storage unavailable
Frequent Pod restarts
```

Alerts should correspond to actionable operational problems.

---

# 50. Capacity Planning

Estimate trace volume.

Example:

```text
10,000 requests/sec
×
10 spans/request
=
100,000 spans/sec
```

If sampling retains 10%:

```text
100,000 × 10%
=
10,000 spans/sec
```

Then estimate:

```text
Storage/day
Storage/retention period
Collector capacity
Query capacity
Network bandwidth
```

Actual sizing depends on span size and workload characteristics.

---

# 51. Retention Planning

Retention depends on:

```text
Business requirements
Incident investigation needs
Storage cost
Compliance requirements
```

Example:

```text
Production → 7–14 days
Staging → shorter
Development → minimal
```

These are examples, not universal requirements.

---

# 52. Cost Optimization

Reduce unnecessary cost by:

```text
Sampling
Filtering
Reducing unnecessary spans
Reducing high-cardinality attributes
Retention management
Right-sizing resources
```

Do not solve every capacity problem by simply adding infrastructure.

---

# 53. Trace Data Security

Trace data can contain sensitive application information.

Avoid collecting:

```text
Passwords
Tokens
Authorization headers
Cookies
Payment information
Sensitive request bodies
```

Apply filtering at the instrumentation or Collector layer.

---

# 54. Multi-Environment Architecture

One architecture can support:

```text
Development
Staging
Production
```

Use metadata:

```text
deployment.environment
```

Example:

```text
payment
  ├── development
  ├── staging
  └── production
```

Production access should be more restricted than development access.

---

# 55. Multi-Cluster Architecture

For multiple EKS clusters:

```text
EKS Cluster A
      ↓
OTel Gateway
      │
      ├─────────┐
      ↓         ↓
EKS Cluster B  EKS Cluster C
      ↓         ↓
OTel Gateway  OTel Gateway
      └────┬────┘
           ↓
     Central Tracing
```

Include:

```text
k8s.cluster.name
deployment.environment
```

to distinguish telemetry sources.

---

# 56. Multi-Region Architecture

Example:

```text
AWS Region A
   ↓
OTel Gateway
   ↓
Tracing Backend

AWS Region B
   ↓
OTel Gateway
   ↓
Tracing Backend
```

Consider:

```text
Latency
Cross-region traffic
Cost
Data residency
Failure isolation
```

---

# 57. Production GitOps

A DevOps team can manage the architecture using GitOps.

```text
Git
 ↓
Helm Values
 ↓
OpenTelemetry Configuration
 ↓
Jaeger Configuration
 ↓
Pull Request
 ↓
Review
 ↓
ArgoCD
 ↓
EKS
```

This provides:

```text
Version control
Audit trail
Repeatability
Rollback
Drift detection
```

---

# 58. Repository Structure

Example:

```text
observability/
├── otel/
│   ├── agent/
│   ├── gateway/
│   └── sampling/
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

The exact repository structure should follow the team's GitOps standards.

---

# 59. Deployment Strategy

Use:

```text
Development
    ↓
Staging
    ↓
Production
```

Validate:

```text
Configuration
Version compatibility
Storage
Sampling
Security
Resource usage
```

before production rollout.

---

# 60. Rolling Upgrade

For replicated components:

```text
Old-1
Old-2
Old-3
```

Upgrade gradually:

```text
New-1
Old-2
Old-3
```

then:

```text
New-1
New-2
Old-3
```

finally:

```text
New-1
New-2
New-3
```

Readiness checks help prevent traffic from reaching unready Pods.

---

# 61. Rollback

If the upgrade introduces a problem:

```text
New Version
     ↓
Problem
     ↓
Git Revert
     ↓
ArgoCD
     ↓
Previous Version
```

Then verify:

```text
Collector
Jaeger
Query
Storage
UI
Trace ingestion
```

---

# 62. Disaster Recovery

A production tracing platform should define recovery procedures.

Consider:

```text
Cluster failure
Storage failure
AZ failure
Configuration corruption
Accidental deletion
```

Recovery should address:

```text
Configuration
Secrets
Storage
Deployment manifests
GitOps state
```

---

# 63. Backup Strategy

Important items may include:

```text
Git repository
Helm values
Kubernetes manifests
Storage configuration
Critical secrets through the approved secret-management process
```

Trace data backup requirements depend on business needs.

Not every organization needs long-term backup of raw trace data.

---

# 64. Failure Testing

Test scenarios such as:

```text
Gateway Pod failure
Query Pod failure
Node failure
AZ failure
Storage interruption
Collector overload
Network interruption
```

Observe whether:

```text
Traces continue
Queues behave correctly
Pods recover
Alerts trigger
Recovery is automatic
```

---

# 65. Production Troubleshooting Workflow

When tracing breaks:

```text
1. Check application instrumentation
2. Check OTLP endpoint
3. Check Collector Agents
4. Check Gateway
5. Check Collector logs
6. Check Jaeger ingestion
7. Check storage
8. Check Query
9. Check UI
10. Check sampling
```

Use:

```bash
kubectl get pods -n observability
kubectl get svc -n observability
kubectl get endpoints -n observability
kubectl get events -n observability
```

---

# 66. No Traces Scenario

Architecture:

```text
Application
   ↓
OTel SDK
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

Check from left to right.

Do not immediately restart everything.

Identify the first broken component.

---

# 67. High Latency in Jaeger Queries

Possible causes:

```text
Large trace volume
Large time range
Storage latency
Query resource constraints
Large traces
Insufficient Query replicas
```

Actions:

```text
Narrow search range
Scale Query
Optimize storage
Review trace size
Review retention
```

---

# 68. High Collector CPU

Possible causes:

```text
High trace volume
Expensive processing
Tail sampling
Large attribute sets
High export rate
```

Actions:

```text
Scale Collector
Review processors
Tune sampling
Reduce unnecessary telemetry
```

---

# 69. High Collector Memory

Check:

```text
Queue size
Batch configuration
Trace size
Sampling
Backend availability
Memory limiter
```

Potential sequence:

```text
Backend unavailable
       ↓
Queue grows
       ↓
Memory grows
       ↓
OOMKilled
```

This is why bounded queues and memory protection are important.

---

# 70. High Dropped Span Rate

Investigate:

```text
Collector overload
Backend failure
Sampling configuration
Queue capacity
Network issues
Resource limits
```

A high dropped span rate can mean important traces are being lost.

---

# 71. Jaeger UI Security

Production UI should have:

```text
Authentication
Authorization
TLS
Private access
```

Possible access model:

```text
Engineer
   ↓
Corporate VPN
   ↓
Internal ALB
   ↓
Authentication
   ↓
Jaeger UI
```

Use least privilege.

---

# 72. Network Architecture

A secure production model:

```text
Internet
   X
   │
Private EKS
   │
   ├── Application
   ├── OTel Collector
   └── Jaeger
          │
          ↓
      Private Storage
```

Keep tracing infrastructure on private network paths where practical.

---

# 73. Observability Stack Integration

A complete platform:

```text
                  Applications
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
     Metrics          Logs          Traces
        ↓              ↓              ↓
   Prometheus          ELK       OpenTelemetry
        ↓                             ↓
     Grafana                        Jaeger
```

Each tool has a different role.

```text
Prometheus → Metrics
ELK        → Logs
Jaeger     → Traces
Grafana    → Visualization / dashboards
```

---

# 74. Incident Response Flow

Example:

```text
Alert
 ↓
Grafana
 ↓
High latency
 ↓
Jaeger
 ↓
Slow span identified
 ↓
Trace ID
 ↓
Kibana
 ↓
Application error
 ↓
Kubernetes
 ↓
Root cause
```

This is a practical DevOps/SRE troubleshooting workflow.

---

# 75. Production Example

Suppose:

```text
Checkout latency:
300ms → 2s
```

Grafana shows:

```text
Orders p95 ↑
```

Jaeger shows:

```text
Orders
  ↓
Payment
  ↓
External API
  ↓
1.7s
```

ELK shows:

```text
Payment timeout
```

Kubernetes shows:

```text
Payment Pods healthy
```

The likely bottleneck is the external dependency rather than the Kubernetes cluster.

---

# 76. Production Architecture Example

```text
                           USERS
                             │
                             ↓
                     Application ALB
                             │
                             ↓
                    Microservices on EKS
                             │
             ┌───────────────┼───────────────┐
             ↓               ↓               ↓
          Orders          Payment        Inventory
             │               │               │
             └───────────────┼───────────────┘
                             ↓
                     OpenTelemetry SDK
                             ↓
                      OTel Agent DS
                             ↓
                    OTel Gateway Service
                             ↓
                  Gateway replicas across AZs
                             ↓
                    Sampling / Processing
                             ↓
                           Jaeger
                             ↓
                     Durable Storage
                             ↓
                      Jaeger Query
                             ↓
                     Internal Ingress
                             ↓
                         Jaeger UI
```

---

# 77. Production Architecture with Observability

```text
                         EKS
                          │
                 ┌────────┼────────┐
                 ↓        ↓        ↓
             Metrics    Logs     Traces
                 ↓        ↓        ↓
            Prometheus   ELK      OTel
                 ↓                 ↓
              Grafana            Jaeger
                 │                 │
                 └────────┬────────┘
                          ↓
                    Incident Response
```

This gives engineers three complementary views:

```text
Metrics → What is wrong?
Traces  → Where is it wrong?
Logs    → Why is it wrong?
```

---

# 78. Production Checklist

```text
ARCHITECTURE
[ ] Agent/Gateway architecture
[ ] Multiple Collector replicas
[ ] Multiple Query replicas
[ ] Durable storage
[ ] Multi-AZ distribution
[ ] No critical single point of failure

COLLECTOR
[ ] OTLP receiver
[ ] Memory limiter
[ ] Kubernetes metadata
[ ] Sampling
[ ] Batch
[ ] Retry
[ ] Bounded queue
[ ] Resource requests/limits

JAEGER
[ ] Ingestion configured
[ ] Storage configured
[ ] Query replicas
[ ] UI configured
[ ] Health probes
[ ] Resource management

KUBERNETES
[ ] Dedicated namespace
[ ] Services
[ ] DNS
[ ] RBAC
[ ] NetworkPolicy
[ ] PDB
[ ] Anti-affinity
[ ] Topology spread

SECURITY
[ ] TLS
[ ] Authentication
[ ] Authorization
[ ] Secrets management
[ ] Private networking
[ ] Sensitive-data filtering

OBSERVABILITY
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] Collector monitoring
[ ] Jaeger monitoring
[ ] Alerts
[ ] Trace-to-log correlation

OPERATIONS
[ ] Capacity planning
[ ] Retention
[ ] Backup strategy
[ ] Disaster recovery
[ ] Failure testing
[ ] Upgrade strategy
[ ] Rollback
[ ] GitOps
```

---

# 79. Interview Question

### How would you design Jaeger for production on EKS?

**Answer:**

I would use OpenTelemetry SDKs in the microservices and export traces using OTLP. In EKS, I would deploy OpenTelemetry Agents as a DaemonSet and scalable Collector Gateways as Deployments. The Gateway would handle memory protection, Kubernetes metadata enrichment, batching, sampling, retry, and queueing before exporting traces to Jaeger. Jaeger would use durable storage, and Query would run with multiple replicas distributed across nodes and Availability Zones. I would expose the UI only through secure internal access and monitor the entire tracing infrastructure using Prometheus and Grafana.

---

# 80. Interview Question

### How do you make Jaeger highly available?

**Answer:**

I avoid single replicas for critical components. I run multiple Collector Gateway replicas, multiple Query replicas, distribute Pods across nodes and Availability Zones, use durable highly available storage, configure readiness probes and PodDisruptionBudgets, and ensure Kubernetes Services can route traffic to healthy replicas.

---

# 81. Interview Question

### How would you handle a sudden increase in trace volume?

**Answer:**

I would first check Collector CPU, memory, queue size, export rate, and dropped spans. Then I would scale Collector replicas if required, review sampling policies, tune batching and queue limits, and investigate the reason for the traffic increase. I would avoid simply increasing memory without addressing the underlying trace-volume problem.

---

# 82. Interview Question

### What happens if the Jaeger storage backend goes down?

**Answer:**

Trace exports can begin failing and Collector queues may grow. I would check the storage health and connectivity, monitor queue and memory usage, and rely on bounded retry/queue mechanisms for temporary failures. For a production system, storage should have its own high-availability and recovery strategy because Jaeger Query cannot retrieve traces if the required storage is unavailable.

---

# 83. Interview Question

### How do you secure Jaeger in production?

**Answer:**

I would keep Jaeger and its storage on private network paths, restrict access using Kubernetes NetworkPolicies and AWS security controls, use TLS where required, manage credentials through Secrets or an approved external secret-management solution, apply least-privilege RBAC, and protect the Jaeger UI with authentication and authorization.

---

# 84. Interview Question

### How do you monitor Jaeger itself?

**Answer:**

I would monitor Collector and Jaeger metrics using Prometheus and visualize them in Grafana. Important signals include CPU, memory, Pod restarts, ingestion rate, export failures, dropped spans, queue utilization, Query latency, Query errors, and storage health. I would create alerts for conditions that can cause trace loss or make the tracing platform unavailable.

---

# 85. Final Mental Model

The complete production architecture can be remembered as:

```text
                         MICROSERVICES
                              │
                         OTel SDKs
                              │
                             OTLP
                              │
                    ┌─────────┴─────────┐
                    ↓                   ↓
              OTel Agent           OTel Agent
              DaemonSet            DaemonSet
                    │                   │
                    └─────────┬─────────┘
                              ↓
                     OTel Gateway
                       Deployment
                              │
                   ┌──────────┼──────────┐
                   ↓          ↓          ↓
                Gateway-1  Gateway-2  Gateway-3
                   └──────────┼──────────┘
                              ↓
                  Memory / Enrichment
                              ↓
                    Sampling / Batch
                              ↓
                       Retry / Queue
                              ↓
                           Jaeger
                              ↓
                    Durable Trace Storage
                              ↓
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                 Query-1   Query-2   Query-3
                    └─────────┼─────────┘
                              ↓
                    Secure Internal Access
                              ↓
                         Jaeger UI
```

The complete observability platform is:

```text
                    APPLICATIONS
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Metrics          Logs          Traces
          ↓              ↓              ↓
    Prometheus           ELK      OpenTelemetry
          ↓                             ↓
       Grafana                        Jaeger
          │                             │
          └──────────────┬──────────────┘
                         ↓
                  INCIDENT RESPONSE
```

The key production principle is:

**OpenTelemetry provides the standardized instrumentation and telemetry pipeline, while Jaeger provides distributed trace storage, querying, and visualization. For production EKS, use an Agent/Gateway architecture, controlled sampling, bounded queues, durable storage, multiple replicas, multi-AZ placement, secure private access, Kubernetes RBAC and NetworkPolicies, Prometheus/Grafana monitoring, ELK correlation, GitOps deployment, and tested failure recovery.**
