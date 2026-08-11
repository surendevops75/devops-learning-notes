# OpenTelemetry Production Architecture

## 1. Overview

A production OpenTelemetry architecture must be designed for:

```text
High Availability
Scalability
Reliability
Security
Performance
Cost Control
Fault Isolation
Observability
```

The goal is not simply to install an OpenTelemetry Collector.

The production goal is to build a telemetry platform that can reliably collect:

```text
Metrics
Logs
Traces
```

from a large number of applications and infrastructure components.

A production architecture commonly follows:

```text
Applications
     ↓
OpenTelemetry SDK / Agents
     ↓
OpenTelemetry Collector
     ↓
Processing
     ↓
Sampling / Filtering
     ↓
Export
     ↓
Observability Backends
```

---

# 2. Production Architecture Overview

A scalable Kubernetes architecture:

```text
                              USERS
                                │
                                ↓
                               ALB
                                │
                                ↓
                           EKS CLUSTER
                                │
             ┌──────────────────┼──────────────────┐
             ↓                  ↓                  ↓
          Orders             Payment           Inventory
          Service            Service            Service
             │                  │                  │
          OTel SDK           OTel SDK           OTel SDK
             │                  │                  │
             └──────────────────┼──────────────────┘
                                ↓
                       OTel Agent DaemonSet
                                ↓
                       OTel Gateway Deployment
                                ↓
                  ┌─────────────┼─────────────┐
                  ↓             ↓             ↓
              Metrics          Logs         Traces
                  ↓             ↓             ↓
             Prometheus         ELK       Trace Backend
                  ↓             ↓             ↓
               Grafana        Kibana        Trace UI
```

---

# 3. Three-Layer Architecture

A production deployment can be divided into three layers.

```text
Layer 1
Telemetry Generation

Layer 2
Telemetry Collection & Processing

Layer 3
Telemetry Storage & Visualization
```

Architecture:

```text
Applications
     ↓
Generation
     ↓
Collectors
     ↓
Processing
     ↓
Backends
     ↓
Visualization
```

---

# 4. Layer 1 — Telemetry Generation

Applications generate telemetry using:

```text
OpenTelemetry SDK
Automatic Instrumentation
Manual Instrumentation
Existing Logging Frameworks
```

Example:

```text
Java
 ↓
OTel SDK
 ↓
Traces / Metrics / Logs
```

Node.js:

```text
Node.js
 ↓
OTel Instrumentation
 ↓
Telemetry
```

Python:

```text
Python
 ↓
OTel Instrumentation
 ↓
Telemetry
```

---

# 5. Layer 2 — Collection

The Collector receives telemetry.

```text
Application
     ↓
OTLP
     ↓
OTel Collector
```

The Collector can receive:

```text
OTLP
Prometheus
File Logs
Host Metrics
Kubernetes Metadata
Other supported receivers
```

---

# 6. Layer 3 — Processing and Export

The Collector processes telemetry:

```text
Receive
  ↓
Enrich
  ↓
Filter
  ↓
Transform
  ↓
Sample
  ↓
Batch
  ↓
Export
```

Then:

```text
Metrics → Prometheus
Logs    → Elasticsearch / ELK
Traces  → Trace Backend
```

---

# 7. Agent + Gateway Architecture

For production Kubernetes environments, Agent + Gateway is a strong architecture.

```text
                    EKS
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     Node 1        Node 2        Node 3
       │             │             │
    OTel Agent    OTel Agent    OTel Agent
       │             │             │
       └─────────────┼─────────────┘
                     ↓
               OTel Gateway
                     ↓
              Observability
                Backends
```

Agent:

```text
Local collection
```

Gateway:

```text
Central processing
Sampling
Routing
Batching
Export
```

---

# 8. Why Agent + Gateway?

A single Collector architecture may work for small environments:

```text
Applications
     ↓
Collector
     ↓
Backend
```

But larger environments benefit from separation:

```text
Applications
     ↓
Agents
     ↓
Gateways
     ↓
Backends
```

Benefits:

```text
Scalability
Centralized processing
Fault isolation
Simpler application configuration
Better sampling control
```

---

# 9. Agent Responsibilities

The Agent should generally perform local collection.

Examples:

```text
Application telemetry
Container logs
Node metrics
Local metadata
```

Typical deployment:

```text
DaemonSet
```

Architecture:

```text
Node
│
├── Application Pods
│
└── OTel Agent
```

---

# 10. Gateway Responsibilities

The Gateway provides centralized processing.

Typical responsibilities:

```text
Tail sampling
Filtering
Transformation
Enrichment
Routing
Batching
Export
```

Architecture:

```text
Agent 1 ──┐
Agent 2 ──┤
Agent 3 ──┼──→ Gateway
Agent 4 ──┤
Agent 5 ──┘
```

---

# 11. Gateway Deployment

Use multiple Gateway replicas.

```text
OTel Gateway
│
├── Gateway-01
├── Gateway-02
└── Gateway-03
```

Expose them through a Kubernetes Service:

```text
otel-gateway
      ↓
Gateway-01
Gateway-02
Gateway-03
```

This prevents a single Gateway Pod from becoming a single point of failure.

---

# 12. High Availability

A production Collector architecture should avoid:

```text
Application
    ↓
Single Collector
    ↓
Backend
```

Instead:

```text
Application
    ↓
Collector Service
    ↓
┌──────────┬──────────┬──────────┐
Gateway-01 Gateway-02 Gateway-03
```

If one Gateway fails:

```text
Gateway-01
    X

Gateway-02 → Available
Gateway-03 → Available
```

Telemetry can continue through healthy replicas.

---

# 13. Availability Zones

For production EKS, distribute Gateway replicas across Availability Zones where appropriate.

```text
AZ-A
 └── Gateway-01

AZ-B
 └── Gateway-02

AZ-C
 └── Gateway-03
```

If one AZ becomes unavailable:

```text
AZ-A
  X

AZ-B → Gateway-02
AZ-C → Gateway-03
```

Telemetry collection remains available if the rest of the architecture is healthy.

---

# 14. Pod Anti-Affinity

Gateway replicas should not all run on the same node.

Bad:

```text
Node-01
├── Gateway-01
├── Gateway-02
└── Gateway-03
```

Better:

```text
Node-01 → Gateway-01
Node-02 → Gateway-02
Node-03 → Gateway-03
```

Pod anti-affinity or topology spread constraints can help achieve this.

---

# 15. Topology Spread Constraints

Topology spread can distribute replicas across:

```text
Nodes
Availability Zones
Other topology domains
```

Example:

```text
AZ-A → Gateway-01
AZ-B → Gateway-02
AZ-C → Gateway-03
```

This improves resilience.

---

# 16. PodDisruptionBudget

A PodDisruptionBudget protects availability during voluntary disruptions.

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: otel-gateway
  namespace: observability
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: otel-gateway
```

If there are three replicas:

```text
Gateway-01
Gateway-02
Gateway-03
```

at least two should remain available during supported voluntary disruptions.

---

# 17. Resource Requests

Every production Collector should have resource requests.

Example:

```yaml
resources:
  requests:
    cpu: 200m
    memory: 256Mi
```

Requests allow Kubernetes to make better scheduling decisions.

---

# 18. Resource Limits

Limits prevent unlimited resource consumption.

Example:

```yaml
resources:
  limits:
    cpu: "1"
    memory: 1Gi
```

However, limits should be sized based on actual telemetry volume.

Do not blindly copy resource values from another environment.

---

# 19. Memory Limiter

Use the Memory Limiter Processor.

```text
Telemetry
    ↓
Memory Limiter
    ↓
Processing
```

During a telemetry spike:

```text
Telemetry volume ↑
       ↓
Memory pressure
       ↓
Memory limiter
       ↓
Protect Collector
```

This reduces the risk of Collector OOM failures.

---

# 20. Batch Processing

Batch processing improves efficiency.

```text
Span 1 ┐
Span 2 ├──→ Batch
Span 3 ┘
          ↓
       Exporter
```

Benefits:

```text
Lower network overhead
Better throughput
Fewer export operations
Better backend efficiency
```

Use batching carefully so telemetry is not delayed excessively.

---

# 21. Queueing

A queue can buffer telemetry before export.

```text
Telemetry
    ↓
Queue
    ↓
Exporter
    ↓
Backend
```

If the backend temporarily becomes unavailable:

```text
Backend
   X
   ↓
Queue
   ↓
Retry
```

Queues must be bounded.

---

# 22. Retry

Exporters can retry transient failures where supported.

Example:

```text
Collector
   ↓
Backend
   X
```

Then:

```text
Retry
   ↓
Backend recovers
   ↓
Export succeeds
```

Retry should not become an infinite memory-consuming process.

---

# 23. Backpressure

Suppose:

```text
Telemetry generation = 100,000 spans/sec
Backend capacity = 60,000 spans/sec
```

Then:

```text
Incoming > Outgoing
```

The queue grows.

Eventually:

```text
Queue ↑
Memory ↑
CPU ↑
```

The system must apply:

```text
Backpressure
Sampling
Scaling
Filtering
Dropping low-value telemetry
```

---

# 24. Telemetry Loss Strategy

When the system is overloaded, prioritize important telemetry.

Example:

```text
Highest priority
    ↓
Error traces
Security events
Critical application logs
Slow traces
    ↓
Normal telemetry
    ↓
Low-value debug telemetry
```

The exact strategy should be defined by operational requirements.

---

# 25. Tail Sampling

Tail sampling is useful for production traces.

Architecture:

```text
Applications
     ↓
Agents
     ↓
Gateway
     ↓
Collect Trace
     ↓
Evaluate Trace
     ↓
Sampling Decision
```

Example:

```text
Error → Keep
Slow → Keep
Normal → Sample
```

---

# 26. Tail Sampling Architecture

```text
                    Agents
                       │
                       ↓
                 OTel Gateway
                       │
                 Tail Sampling
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      Error          Slow          Normal
        │              │              │
      KEEP           KEEP           SAMPLE
        │              │              │
        └──────────────┼──────────────┘
                       ↓
                 Trace Backend
```

---

# 27. Tail Sampling Consistency

A distributed trace contains multiple spans.

Therefore, tail sampling must be designed so that decisions apply to the complete trace rather than randomly keeping individual spans.

Otherwise:

```text
Trace ABC
├── Span 1 → kept
├── Span 2 → dropped
└── Span 3 → kept
```

can produce an incomplete trace.

---

# 28. Sampling Policies

Production policies may include:

```text
Error traces
Slow traces
Specific services
Specific environments
Specific HTTP routes
Probability sampling
```

Example:

```text
production + error → 100%
production + latency > threshold → 100%
production normal → 5%
development → higher sampling
```

---

# 29. Metrics Sampling

Metrics are fundamentally different from traces.

Do not treat metric samples like trace sampling.

Metrics usually need consistent time-series data for:

```text
Rates
Counters
Gauges
Histograms
SLOs
Alerts
```

Reduce metric cost primarily through:

```text
Filtering
Aggregation
Appropriate scrape intervals
Controlled cardinality
Retention
```

---

# 30. Log Volume Control

Control log volume using:

```text
Log levels
Filtering
Redaction
Deduplication
Retention
Indexing strategy
```

Avoid sending every low-value debug event into long-term production storage.

---

# 31. Cardinality Management

Cardinality is one of the biggest observability cost drivers.

Potentially problematic attributes:

```text
request_id
session_id
user_id
full URL
large arbitrary strings
```

Prefer controlled dimensions:

```text
service.name
http.route
status
environment
version
```

Review high-cardinality attributes before adding them globally.

---

# 32. Resource Attributes

Use consistent resource attributes.

Recommended:

```text
service.name
service.version
deployment.environment
cloud.provider
cloud.region
k8s.cluster.name
k8s.namespace.name
```

These provide common dimensions across telemetry.

---

# 33. Service Naming

Use consistent service names.

Example:

```text
orders
payment
inventory
notification
user
```

Avoid inconsistent names:

```text
orders-service
orders_service
OrdersService
orders-prod
```

unless those distinctions are intentionally part of the naming strategy.

---

# 34. Environment Naming

Use consistent values:

```text
production
staging
development
```

Avoid mixing:

```text
prod
production
PROD
production-env
```

unless there is a defined normalization strategy.

---

# 35. Version Tracking

Include:

```text
service.version
```

Example:

```text
payment
v2.3.1
```

Then incidents can be correlated with deployments.

```text
Version v2.3.1
   ↓
Error rate ↑
   ↓
Trace latency ↑
```

---

# 36. Kubernetes Metadata

Production telemetry should identify Kubernetes resources where useful.

Example:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
```

This enables:

```text
Service
   ↓
Pod
   ↓
Node
```

correlation.

---

# 37. Kubernetes Attributes Processor

Conceptually:

```yaml
processors:
  k8sattributes:
```

It can associate telemetry with Kubernetes metadata.

Architecture:

```text
Telemetry
   ↓
Kubernetes Attributes Processor
   ↓
Enriched Telemetry
```

The Collector requires appropriate Kubernetes API permissions for metadata lookup.

---

# 38. RBAC

Use least privilege.

```text
ServiceAccount
      ↓
ClusterRole / Role
      ↓
Binding
      ↓
Collector
```

Grant only required:

```text
Resources
Verbs
Namespaces
```

Do not use unrestricted permissions simply to make configuration easier.

---

# 39. Network Security

Telemetry should travel through controlled network paths.

```text
Application
   ↓
OTel Agent
   ↓
OTel Gateway
   ↓
Backend
```

Controls:

```text
NetworkPolicy
Security Groups
TLS
Authentication
Private networking
```

---

# 40. TLS

Use TLS when telemetry crosses a trust boundary.

Example:

```text
EKS
 ↓
TLS
 ↓
External Observability Backend
```

For multi-cluster architectures:

```text
Cluster A
   ↓
TLS
   ↓
Central Gateway
```

---

# 41. Authentication

Backend authentication may use:

```text
API tokens
Bearer tokens
mTLS
Cloud authentication
Other backend-specific mechanisms
```

Never place secrets directly in:

```text
Git repository
Container image
Source code
Plain-text configuration
```

---

# 42. Secrets Management

Possible architecture:

```text
AWS Secrets Manager
        ↓
Secret Integration
        ↓
EKS
        ↓
OTel Collector
```

For Kubernetes-native secrets:

```text
Secret
   ↓
Collector Pod
```

Use appropriate encryption and access controls.

---

# 43. Private Networking

For EKS:

```text
Internet
   ↓
ALB
   ↓
Private Subnets
   ↓
EKS Workloads
```

Telemetry can remain inside private networking:

```text
Application
   ↓
OTel Agent
   ↓
OTel Gateway
```

This reduces unnecessary exposure.

---

# 44. Multi-Cluster Architecture

Suppose there are:

```text
Production EKS
Staging EKS
Development EKS
```

Architecture:

```text
Prod EKS ──────┐
               │
Stage EKS ─────┼──→ Central Gateway
               │
Dev EKS ───────┘
                    ↓
              Backends
```

Add:

```text
k8s.cluster.name
deployment.environment
```

to distinguish sources.

---

# 45. Multi-Region Architecture

Example:

```text
AWS Region A
     ↓
Regional Gateway
     │
     └──────────┐
                ↓
             Central
             Backend
                ↑
     ┌──────────┘
     │
Regional Gateway
     ↑
AWS Region B
```

Consider:

```text
Latency
Network cost
Data residency
Backend availability
Regional failures
```

---

# 46. Centralized vs Regional Gateways

Centralized:

```text
All Clusters
     ↓
Central Gateway
```

Advantages:

```text
Simpler management
Centralized policies
Centralized sampling
```

Regional:

```text
Cluster
 ↓
Regional Gateway
 ↓
Central Backend
```

Advantages:

```text
Lower latency
Reduced cross-region traffic
Regional isolation
```

Choose based on architecture requirements.

---

# 47. Failure Domain Design

Do not allow one failure to affect the entire observability system.

Failure domains:

```text
Application
Node
AZ
Cluster
Region
Gateway
Backend
```

Example:

```text
Gateway-01 failure
      ↓
Gateway-02/03 continue
```

---

# 48. Collector Failure

If an Agent fails:

```text
Node
 ↓
Agent failure
 ↓
Kubernetes restarts Agent
```

If a Gateway fails:

```text
Gateway-01
    X
    ↓
Service
    ↓
Gateway-02/03
```

If the backend fails:

```text
Collector
 ↓
Queue / Retry
 ↓
Backend recovery
```

Telemetry may still be lost during extended failures, so define acceptable loss and buffering limits.

---

# 49. Application Dependency on Telemetry

Critical rule:

```text
Application
      │
      ├── Business logic
      │
      └── Telemetry
```

Telemetry should not become a critical synchronous dependency.

Bad:

```text
Payment
  ↓
Wait for telemetry backend
  ↓
Complete payment
```

Better:

```text
Payment
  ↓
Business operation
  ↓
Telemetry exported asynchronously
```

---

# 50. Telemetry Backpressure Should Not Break Applications

If the Collector is unavailable:

```text
Collector
   X
```

the application should continue operating where the instrumentation design allows.

Telemetry may be:

```text
Buffered
Dropped
Retried
```

according to the configured reliability strategy.

Business traffic should remain independent.

---

# 51. Collector Health Monitoring

Monitor the Collector itself.

Important metrics include:

```text
Received telemetry
Exported telemetry
Dropped telemetry
Refused telemetry
Queue size
Exporter failures
Processor errors
CPU
Memory
```

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

---

# 52. Backend Monitoring

Monitor:

```text
Elasticsearch
Prometheus
Trace Backend
```

Examples:

```text
Storage
Ingestion rate
Query latency
CPU
Memory
Errors
Capacity
```

Observability is only useful if the backend remains healthy.

---

# 53. End-to-End Telemetry Monitoring

Monitor the complete pipeline:

```text
Application
    ↓
Agent
    ↓
Gateway
    ↓
Exporter
    ↓
Backend
    ↓
Visualization
```

If telemetry disappears, determine the exact failing layer.

---

# 54. Telemetry Delivery SLO

A mature observability platform can define an internal SLO.

Example:

```text
99.9% of telemetry should reach the backend within an acceptable delay.
```

Measure:

```text
Generation timestamp
        ↓
Backend ingestion timestamp
```

This helps detect observability pipeline degradation.

---

# 55. Collector Deployment Strategy

Use version-controlled deployment.

Example:

```text
Git
 ↓
otel-values.yaml
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

Benefits:

```text
Auditability
Rollback
Consistency
Peer review
Drift detection
```

---

# 56. GitOps Repository Structure

A possible structure:

```text
observability/
├── otel-collector/
│   ├── values.yaml
│   ├── config.yaml
│   └── namespace.yaml
│
├── prometheus/
├── grafana/
└── elk/
```

The exact structure depends on the team's repository strategy.

---

# 57. Terraform + Helm + ArgoCD

A practical separation:

```text
Terraform
   ↓
AWS Infrastructure
   ↓
EKS / VPC / IAM
```

Then:

```text
Git
   ↓
Helm Values / Kubernetes Manifests
   ↓
ArgoCD
   ↓
OpenTelemetry
```

This separates infrastructure provisioning from application/platform deployment.

---

# 58. Production Change Management

Before changing Collector configuration:

```text
1. Modify Git
2. Create Pull Request
3. Review configuration
4. Validate syntax
5. Test in staging
6. Deploy
7. Monitor Collector
8. Verify telemetry
```

Avoid making undocumented production changes directly with `kubectl edit`.

---

# 59. Configuration Validation

Validate:

```text
Receivers
Processors
Exporters
Pipelines
Endpoints
TLS
Authentication
```

A configuration error can cause an entire telemetry pipeline to stop.

---

# 60. Staging Environment

Test changes in:

```text
Development
     ↓
Staging
     ↓
Production
```

Validate:

```text
Telemetry volume
Performance
Sampling
Backend ingestion
Security
Resource usage
```

---

# 61. Production Rollout

For major Collector changes:

```text
Old Configuration
       ↓
Canary Collector
       ↓
Observe
       ↓
Expand rollout
       ↓
Full production
```

This reduces the risk of breaking the entire observability pipeline.

---

# 62. Canary Collector

Example:

```text
Applications
    │
    ├── Existing Gateway
    │
    └── Canary Gateway
```

Send a controlled amount of traffic to the new configuration.

Monitor:

```text
Errors
Dropped telemetry
Latency
CPU
Memory
Backend ingestion
```

---

# 63. Rolling Updates

Kubernetes can perform rolling updates:

```text
Gateway v1
Gateway v1
Gateway v1
```

becomes:

```text
Gateway v2
Gateway v1
Gateway v1
```

then:

```text
Gateway v2
Gateway v2
Gateway v1
```

then:

```text
Gateway v2
Gateway v2
Gateway v2
```

Readiness probes help prevent traffic from reaching unready Pods.

---

# 64. Rollback

If a new Collector version causes problems:

```text
New Version
    ↓
Telemetry failure
    ↓
Rollback
    ↓
Previous Version
```

GitOps makes rollback easier:

```text
Git
 ↓
Previous commit
 ↓
ArgoCD
 ↓
EKS
```

---

# 65. Collector Upgrade

Before upgrading:

```text
Check release compatibility
Review configuration changes
Test in staging
Review deprecated components
Check exporter compatibility
```

Then:

```text
Canary
 ↓
Rolling upgrade
 ↓
Validation
```

Do not assume configuration from an older Collector version will always remain valid indefinitely.

---

# 66. Production Capacity Planning

Estimate:

```text
Number of services
Number of Pods
Requests per second
Spans per request
Log events per second
Metric cardinality
Telemetry size
Sampling rate
Retention
```

Example:

```text
10,000 requests/sec
×
5 spans/request
=
50,000 spans/sec
```

This gives a starting point for capacity planning.

---

# 67. Trace Volume Estimation

Suppose:

```text
20,000 requests/sec
```

and:

```text
8 spans/request
```

Then:

```text
20,000 × 8
=
160,000 spans/sec
```

If only 10% of normal traces are retained:

```text
160,000 × 10%
=
16,000 spans/sec
```

Error and slow-trace policies can retain additional traces.

---

# 68. Log Volume Estimation

Suppose:

```text
100 services
```

each produces:

```text
500 log events/sec
```

Then:

```text
100 × 500
=
50,000 logs/sec
```

This volume must be considered when sizing:

```text
Agents
Gateways
Network
Elasticsearch
Storage
```

---

# 69. Metrics Cardinality

A metric such as:

```text
http_requests_total
```

can become expensive if it contains uncontrolled labels.

Bad:

```text
user_id
request_id
session_id
```

Better:

```text
service
route
method
status
environment
```

Cardinality management is essential for production monitoring.

---

# 70. Cost Optimization

Control OpenTelemetry cost using:

```text
Trace sampling
Log filtering
Metric cardinality control
Retention policies
Batching
Compression
Tiered storage
Appropriate scrape intervals
```

Do not simply collect everything forever.

---

# 71. Observability Data Retention

Different telemetry types can have different retention.

Example:

```text
Metrics
   → Longer operational history

Logs
   → Moderate retention

Traces
   → Shorter retention with intelligent sampling
```

Actual retention should follow:

```text
Operational requirements
Compliance
Incident investigation needs
Cost
```

---

# 72. Data Classification

Telemetry should be classified before collection.

```text
Operational
Security-sensitive
Personal
Business-sensitive
Public/non-sensitive
```

Then define:

```text
Collection
Redaction
Retention
Access control
```

accordingly.

---

# 73. Access Control

Not everyone should have unrestricted access to observability data.

Examples:

```text
Developers
Operations
Security
Platform Engineering
Management
```

can have different permissions.

This is especially important for logs containing sensitive operational information.

---

# 74. Auditability

Production observability changes should be auditable.

Track:

```text
Who changed configuration
What changed
When it changed
Why it changed
Who approved it
```

GitOps provides a strong foundation for this.

---

# 75. Disaster Recovery

Consider what happens if the observability backend is lost.

Questions:

```text
Can telemetry be buffered?
Can the backend be restored?
How much telemetry loss is acceptable?
Is configuration backed up?
Are dashboards backed up?
Are alert rules backed up?
```

Telemetry should be treated as operational infrastructure.

---

# 76. Backup Strategy

Back up important configuration:

```text
Collector configuration
Helm values
Kubernetes manifests
Alert rules
Dashboard definitions
Backend configuration
```

Use Git as the source of truth where appropriate.

---

# 77. Observability During Cluster Upgrade

During an EKS upgrade:

```text
Node draining
   ↓
Pods rescheduled
   ↓
OTel Agents recreated
   ↓
Gateway remains available
```

Use:

```text
DaemonSet
Multiple Gateway replicas
PDB
Topology spreading
```

to reduce telemetry disruption.

---

# 78. Observability During Node Failure

If:

```text
Node-01
   X
```

the DaemonSet Agent on that node disappears.

Kubernetes reschedules workloads onto another node.

Gateway replicas on other nodes continue processing telemetry.

This demonstrates the importance of separating:

```text
Agent availability
Gateway availability
Backend availability
```

---

# 79. Observability During AZ Failure

If:

```text
AZ-A
  X
```

then:

```text
Gateway-01
  X

Gateway-02 → AZ-B
Gateway-03 → AZ-C
```

The remaining Gateway replicas can continue.

Application workloads should also be distributed appropriately across AZs.

---

# 80. Observability During Backend Failure

If Elasticsearch is unavailable:

```text
OTel Gateway
     ↓
Elasticsearch
     X
```

Collector behavior should be controlled by:

```text
Queue
Retry
Backpressure
Memory limits
Sampling
```

Do not allow backend failure to cause unbounded Collector memory growth.

---

# 81. Observability During Network Failure

If the network path fails:

```text
Application
    ↓
Collector
    X
Backend
```

The Collector should protect itself using:

```text
Bounded queues
Retries
Timeouts
Memory limiter
```

The application should remain independent from backend availability.

---

# 82. Timeouts

Telemetry exporters should have appropriate timeouts.

Without timeouts:

```text
Backend unavailable
   ↓
Exporter waits indefinitely
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

# 83. Collector Security Hardening

Production Collector Pods should follow security best practices.

Consider:

```text
Non-root execution where supported
Read-only filesystem where supported
Minimal capabilities
Restricted RBAC
Network restrictions
Secret management
Pinned images
Regular updates
```

Test compatibility before applying restrictive Pod Security settings.

---

# 84. Container Image Security

Use:

```text
Pinned versions
Trusted registries
Image scanning
Regular upgrades
```

Avoid:

```text
latest
```

in production deployment definitions.

Example:

```text
otel/opentelemetry-collector-contrib:<tested-version>
```

Pin the version through your deployment process.

---

# 85. Supply Chain Security

Protect the observability platform itself.

Pipeline:

```text
Git
 ↓
CI
 ↓
Image / Configuration Scanning
 ↓
Approval
 ↓
ArgoCD
 ↓
EKS
```

Security tools can check:

```text
Container vulnerabilities
Configuration problems
Dependency vulnerabilities
Secrets
```

---

# 86. Production Monitoring Dashboard

A Collector dashboard should include:

```text
Collector CPU
Collector memory
Received telemetry
Exported telemetry
Dropped telemetry
Exporter failures
Queue size
Export latency
Pod restarts
```

A Gateway dashboard can additionally show:

```text
Replica health
Throughput
Sampling rates
Backend errors
```

---

# 87. Alerting

Important Collector alerts:

```text
Collector unavailable
High memory usage
OOMKilled
High dropped telemetry
Exporter failure
Queue near capacity
High export latency
Gateway replica count below desired level
```

These alerts protect the observability platform itself.

---

# 88. Example Alert Scenario

Suppose:

```text
otelcol_exporter_send_failed ↑
```

Alert:

```text
OpenTelemetry Gateway exporter failures are increasing.
```

Investigation:

```text
Gateway
 ↓
Exporter
 ↓
Backend connectivity
 ↓
TLS
 ↓
Authentication
 ↓
Backend health
```

---

# 89. Log-Based Collector Troubleshooting

Collector logs may show:

```text
exporter failed
connection refused
authentication failed
TLS handshake failed
pipeline error
```

Use:

```bash
kubectl logs <collector-pod> -n observability
```

Then inspect the relevant pipeline configuration.

---

# 90. Kubernetes Troubleshooting Commands

Useful commands:

```bash
kubectl get pods -n observability
```

```bash
kubectl get svc -n observability
```

```bash
kubectl describe pod <pod> -n observability
```

```bash
kubectl logs <pod> -n observability
```

```bash
kubectl get events -n observability --sort-by=.lastTimestamp
```

These Kubernetes commands complement OpenTelemetry telemetry.

---

# 91. Collector Configuration Inspection

Use:

```bash
kubectl get configmap -n observability
```

Then:

```bash
kubectl describe configmap <configmap-name> -n observability
```

For a production GitOps environment, prefer checking the version-controlled configuration as the source of truth.

---

# 92. Testing OTLP Connectivity

From a suitable diagnostic environment, verify:

```text
DNS
Port
Protocol
TLS
Authentication
```

Example:

```text
otel-gateway.observability.svc
```

Common OTLP ports:

```text
4317
4318
```

Do not expose OTLP endpoints publicly unless there is a specific, secured architecture requiring it.

---

# 93. Production Namespace Structure

A possible structure:

```text
observability
│
├── otel-agent
├── otel-gateway
├── prometheus
├── grafana
└── other observability components
```

Application namespaces remain separate:

```text
production
staging
development
```

---

# 94. Example Production Signal Flow

### Traces

```text
Application
 ↓
OTel SDK
 ↓
Agent
 ↓
Gateway
 ↓
Tail Sampling
 ↓
Trace Backend
```

### Logs

```text
Application
 ↓
stdout
 ↓
Agent
 ↓
Gateway
 ↓
Filtering
 ↓
Elasticsearch
```

### Metrics

```text
Application
 ↓
OTel SDK / Prometheus-compatible source
 ↓
Agent / Collector
 ↓
Prometheus
```

---

# 95. Complete EKS Architecture

```text
                                      AWS
                                       │
                                    Route53
                                       │
                                       ↓
                                      ALB
                                       │
                                       ↓
                               Private EKS Cluster
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        ↓                              ↓                              ↓
   Application Pods               Application Pods               Application Pods
        │                              │                              │
   OTel SDK                       OTel SDK                       OTel SDK
        │                              │                              │
        └──────────────────────────────┼──────────────────────────────┘
                                       ↓
                              OTel Agent DaemonSet
                                       │
                                       ↓
                              OTel Gateway Service
                                       │
                ┌──────────────────────┼──────────────────────┐
                ↓                      ↓                      ↓
           Gateway-01              Gateway-02              Gateway-03
                │                      │                      │
                └──────────────────────┼──────────────────────┘
                                       ↓
                             Processing / Sampling
                                       │
                  ┌────────────────────┼────────────────────┐
                  ↓                    ↓                    ↓
              Prometheus         Elasticsearch          Trace Backend
                  ↓                    ↓                    ↓
               Grafana               Kibana              Trace UI
```

---

# 96. Production Design Principles

The architecture should follow these principles:

```text
1. No single point of failure
2. Collect close to workloads
3. Process centrally when appropriate
4. Protect Collector memory
5. Batch telemetry
6. Sample intelligently
7. Control cardinality
8. Secure telemetry
9. Monitor the observability pipeline
10. Manage configuration through Git
11. Test changes before production
12. Keep telemetry independent from application availability
```

---

# 97. Real-World Incident Example

Suppose users report slow checkout.

Start with Grafana:

```text
Checkout latency ↑
```

Then tracing:

```text
Checkout
   ↓
Orders
   ↓
Payment
   ↓
External Gateway
```

Trace shows:

```text
External Gateway = 2.1 seconds
```

Then Kibana:

```text
Payment gateway timeout
```

Then infrastructure:

```text
Payment Pods healthy
CPU normal
Memory normal
```

Conclusion:

```text
External payment dependency latency
```

The investigation used:

```text
Metrics
   ↓
Traces
   ↓
Logs
```

---

# 98. Real-World Deployment Example

A new Payment version is deployed:

```text
v2.3.0
```

After deployment:

```text
HTTP error rate ↑
Latency ↑
```

Trace:

```text
Payment
   ↓
Database
   ↓
Slow query
```

Logs:

```text
Database timeout
```

Deployment metadata:

```text
service.version = v2.3.0
```

Rollback:

```text
v2.3.0
   ↓
Rollback
   ↓
v2.2.9
```

Telemetry confirms:

```text
Latency ↓
Errors ↓
```

This is how OpenTelemetry supports production deployment validation.

---

# 99. Production Readiness Checklist

```text
ARCHITECTURE
[ ] Agent deployment designed
[ ] Gateway deployment designed
[ ] Multiple Gateway replicas
[ ] AZ distribution
[ ] Pod anti-affinity / topology spread
[ ] PDB configured

APPLICATION
[ ] OTel SDK configured
[ ] Automatic instrumentation configured
[ ] Manual instrumentation where required
[ ] service.name configured
[ ] service.version configured
[ ] environment configured

COLLECTOR
[ ] Receivers configured
[ ] Processors configured
[ ] Exporters configured
[ ] Pipelines configured
[ ] Memory limiter enabled
[ ] Batch processor enabled
[ ] Queues configured where required
[ ] Retry behavior configured
[ ] Timeouts configured

KUBERNETES
[ ] ServiceAccount
[ ] RBAC
[ ] NetworkPolicy
[ ] Resource requests
[ ] Resource limits
[ ] Health probes
[ ] Node scheduling strategy
[ ] Kubernetes metadata enrichment

SECURITY
[ ] TLS
[ ] Authentication
[ ] Secrets management
[ ] Least privilege
[ ] Image scanning
[ ] Version pinning
[ ] Sensitive data filtering

RELIABILITY
[ ] Backend failure tested
[ ] Collector failure tested
[ ] Node failure tested
[ ] AZ failure considered
[ ] Queue behavior tested
[ ] Backpressure tested
[ ] Rollback tested

OPERATIONS
[ ] Collector dashboard
[ ] Collector alerts
[ ] Backend monitoring
[ ] Telemetry delivery monitoring
[ ] Capacity planning
[ ] Retention policies
[ ] Cost monitoring

DEPLOYMENT
[ ] GitOps
[ ] CI validation
[ ] Staging testing
[ ] Canary strategy
[ ] Rolling upgrade
[ ] Rollback procedure
```

---

# 100. Final Production Mental Model

Remember OpenTelemetry Production Architecture as:

```text
                    APPLICATIONS
                         │
                         ↓
                    OTel SDKs
                         │
                         ↓
                 OTel Agent Layer
                  DaemonSet
                         │
                         ↓
                OTel Gateway Layer
                 Multiple Replicas
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
       Metrics          Logs          Traces
          │              │              │
          ↓              ↓              ↓
     Processing       Processing     Sampling
          │              │              │
          └──────────────┼──────────────┘
                         ↓
                    Export Layer
                         │
          ┌──────────────┼──────────────┐
          ↓              ↓              ↓
      Prometheus         ELK       Trace Backend
          ↓              ↓              ↓
       Grafana         Kibana         Trace UI
```

For a production EKS environment:

```text
Git
 ↓
Terraform
 ↓
AWS / EKS Infrastructure
 ↓
Helm / Kubernetes
 ↓
ArgoCD
 ↓
OTel Agent + Gateway
 ↓
Metrics + Logs + Traces
 ↓
Prometheus + ELK + Trace Backend
```

The most important production principles are:

**Use Agents for local collection, Gateways for centralized processing, multiple replicas for high availability, Kubernetes metadata for workload correlation, memory limits and bounded queues for reliability, tail sampling for trace cost control, filtering and cardinality management for telemetry efficiency, TLS/RBAC/NetworkPolicies for security, and GitOps for controlled configuration and deployment.**

The observability platform itself must also be monitored. A production OpenTelemetry architecture is successful only when it can continue collecting useful telemetry during application failures, Pod restarts, node failures, deployments, traffic spikes, and backend interruptions without becoming a dependency that can take down the application.
