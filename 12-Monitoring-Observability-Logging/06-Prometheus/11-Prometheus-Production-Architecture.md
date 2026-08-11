# Prometheus Production Architecture

Prometheus can be easy to install in a development environment, but a production monitoring platform requires much more than simply running a Prometheus pod.

A production architecture must consider:

```text
High availability
Scalability
Storage
Security
Performance
Alerting
Service discovery
Kubernetes integration
Long-term retention
Disaster recovery
GitOps
Operational ownership
```

The goal is to build a monitoring platform that remains useful when the application itself is experiencing problems.

---

# 1. Production Monitoring Architecture

A basic setup may look like:

```text
Application
    ↓
Prometheus
    ↓
Grafana
```

A production setup is more complete:

```text
                           Users
                             │
                             ↓
                          Grafana
                             │
                             ↓
                      Query / Metrics Layer
                             │
              ┌──────────────┴──────────────┐
              ↓                             ↓
        Prometheus A                  Prometheus B
              │                             │
              └──────────────┬──────────────┘
                             ↓
                    Long-Term Storage
                             │
                             ↓
                        Object Store

EKS
 │
 ├── Node Exporter
 ├── Kube-State-Metrics
 ├── Application Metrics
 ├── ServiceMonitors
 └── PodMonitors
         │
         ↓
     Prometheus

Prometheus
    ↓
Alertmanager HA
    ↓
Slack / Email / Pager
```

---

# 2. Production Architecture Layers

A useful way to design the platform is to divide it into layers.

```text
Layer 1 → Applications
Layer 2 → Metric Collection
Layer 3 → Prometheus
Layer 4 → Alerting
Layer 5 → Visualization
Layer 6 → Long-Term Storage
Layer 7 → Security
Layer 8 → Operations
```

Each layer has a specific responsibility.

---

# 3. Application Layer

Applications should expose useful metrics.

For example:

```text
HTTP request count
HTTP error count
Request latency
Request duration
Active requests
Application-specific business metrics
```

A microservice might expose:

```text
/metrics
```

Prometheus can scrape that endpoint.

---

# 4. Infrastructure Layer

Infrastructure metrics provide visibility into the underlying platform.

For Kubernetes nodes, Node Exporter can provide:

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
Node Exporter
      ↓
Prometheus
```

---

# 5. Kubernetes State Layer

Kube-State-Metrics provides Kubernetes object-state information.

Examples:

```text
Deployments
Pods
Nodes
ReplicaSets
DaemonSets
StatefulSets
Jobs
CronJobs
PersistentVolumes
PersistentVolumeClaims
```

Architecture:

```text
Kubernetes API
      ↓
Kube-State-Metrics
      ↓
Prometheus
```

---

# 6. Application Metrics Layer

Application metrics may come from:

```text
Java
Node.js
Python
Go
Nginx
Databases
Message queues
Custom exporters
```

Example:

```text
Payment Service
      ↓
/metrics
      ↓
Prometheus
```

---

# 7. Prometheus Layer

Prometheus is responsible for:

```text
Metric scraping
Time-series storage
PromQL
Recording rules
Alerting rules
Service discovery
```

In production, use multiple Prometheus replicas when availability requirements justify it.

---

# 8. Prometheus HA

A typical production baseline:

```text
             Kubernetes
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
 Prometheus A          Prometheus B
        │                   │
      AZ-A                AZ-B
        │                   │
        ↓                   ↓
     Storage A           Storage B
```

The replicas independently scrape targets.

---

# 9. Alertmanager Layer

Prometheus detects alert conditions.

Alertmanager manages those alerts.

```text
Prometheus
    ↓
Alert Rules
    ↓
Alertmanager
    ↓
Grouping
    ↓
Routing
    ↓
Notifications
```

In production:

```text
Alertmanager A
Alertmanager B
```

can be used for HA.

---

# 10. Grafana Layer

Grafana provides:

```text
Dashboards
Visualization
Investigation
Ad-hoc querying
Operational views
```

Typical flow:

```text
Engineer
    ↓
Grafana
    ↓
Prometheus / Query Layer
    ↓
Metrics
```

Grafana should not be the primary storage system.

---

# 11. Long-Term Storage

Prometheus local storage is excellent for operational monitoring, but large environments may require longer retention.

Examples of long-term systems include:

```text
Thanos
Grafana Mimir
Cortex
VictoriaMetrics
```

Choose based on organizational requirements.

---

# 12. Object Storage

For architectures such as Thanos, metrics can be stored in durable object storage.

In AWS:

```text
Prometheus
    ↓
Thanos
    ↓
Amazon S3
```

This provides a durable historical metrics layer.

---

# 13. Why Object Storage?

Object storage provides:

```text
Durability
Scalability
Large capacity
Independent storage lifecycle
Long retention
```

It also separates historical data from the lifecycle of individual Prometheus pods.

---

# 14. Production Data Flow

The complete metric flow:

```text
Application
     ↓
/metrics
     ↓
ServiceMonitor / PodMonitor
     ↓
Prometheus Service Discovery
     ↓
Prometheus Scrape
     ↓
TSDB
     ↓
Recording Rules
     ↓
PromQL
     ↓
Grafana
```

For alerting:

```text
Prometheus
     ↓
Alert Rule
     ↓
Alertmanager
     ↓
Notification
```

For long-term storage:

```text
Prometheus
     ↓
Long-Term Metrics Backend
     ↓
Object Storage
```

---

# 15. Production EKS Architecture

For an EKS-based microservices platform:

```text
                             AWS
                              │
                             EKS
                              │
       ┌──────────────────────┼──────────────────────┐
       ↓                      ↓                      ↓
   Application Pods      Kubernetes Objects       Nodes
       │                      │                      │
       ↓                      ↓                      ↓
   /metrics            Kube-State-Metrics      Node Exporter
       │                      │                      │
       └──────────────────────┼──────────────────────┘
                              ↓
                     Prometheus HA
                       ┌──────┴──────┐
                       ↓             ↓
                   Prom A         Prom B
                       │             │
                       └──────┬──────┘
                              ↓
                      Alertmanager HA
                              │
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                  Slack      Email     Pager

Prometheus
     ↓
Long-Term Metrics Backend
     ↓
Amazon S3

Grafana
     ↓
Query Layer
     ↓
Metrics
```

---

# 16. Availability Zones

In AWS, avoid placing all monitoring components in one Availability Zone.

Better:

```text
AZ-A
 ├── Prometheus A
 └── Grafana A

AZ-B
 ├── Prometheus B
 └── Alertmanager A

AZ-C
 └── Alertmanager B
```

The exact placement depends on capacity and failure requirements.

---

# 17. Node Distribution

Use Kubernetes scheduling controls such as:

```text
Pod Anti-Affinity
Topology Spread Constraints
PodDisruptionBudget
```

to avoid concentrating replicas on one node.

---

# 18. Pod Anti-Affinity

Without anti-affinity:

```text
Node 1
 ├── Prometheus A
 └── Prometheus B
```

With anti-affinity:

```text
Node 1
 └── Prometheus A

Node 2
 └── Prometheus B
```

This improves resilience against node failure.

---

# 19. Topology Spread

Topology spread can distribute replicas across:

```text
Nodes
Availability Zones
Other topology domains
```

Conceptually:

```text
AZ-A → Prometheus A
AZ-B → Prometheus B
```

This is stronger than simply ensuring different pod names.

---

# 20. Persistent Storage

Prometheus requires storage for local TSDB data.

Example architecture:

```text
Prometheus A
     ↓
PVC A
     ↓
EBS Volume A

Prometheus B
     ↓
PVC B
     ↓
EBS Volume B
```

For EKS, choose an appropriate EBS CSI-backed StorageClass according to the cluster architecture.

---

# 21. Storage Retention

Prometheus retention can be based on:

```text
Time
Size
```

Example concept:

```text
15 days
```

or:

```text
100GB
```

Production retention should be based on:

```text
Operational requirements
Disk capacity
Ingestion rate
Query needs
Recovery requirements
Cost
```

---

# 22. Local Retention vs Long-Term Retention

Use local Prometheus storage for:

```text
Recent operational data
Fast queries
Real-time dashboards
Alert evaluation
```

Use long-term storage for:

```text
Historical analysis
Multiple clusters
Long retention
Capacity beyond local disks
Cross-cluster queries
```

---

# 23. Storage Growth

Prometheus storage growth depends on:

```text
Number of active series
Scrape interval
Retention
Metric size
Label cardinality
```

A production team should monitor storage usage continuously.

---

# 24. Storage Monitoring

Monitor:

```text
Disk usage
TSDB size
WAL size
Compaction
Storage errors
Write failures
```

A nearly full Prometheus disk can become a serious monitoring incident.

---

# 25. High Cardinality

High cardinality is one of the biggest Prometheus production risks.

Example:

```text
http_requests_total{
    user_id="12345"
}
```

If millions of users exist:

```text
Millions of time series
```

This can consume significant:

```text
Memory
CPU
Storage
Query resources
```

---

# 26. Cardinality Rules

Avoid unbounded labels such as:

```text
user_id
request_id
session_id
transaction_id
```

unless there is a very strong reason and the resulting cardinality is controlled.

Prefer bounded dimensions:

```text
service
method
route
status
environment
```

---

# 27. Metric Naming

Use consistent metric names.

Examples:

```text
http_requests_total
http_request_duration_seconds
process_cpu_seconds_total
```

Use standard Prometheus naming conventions.

---

# 28. Labels

Good:

```text
service="payment"
environment="production"
method="POST"
status="500"
```

Potentially dangerous:

```text
request_id="8f7a..."
user_id="918273..."
```

The number of possible label values matters.

---

# 29. Scrape Interval

Common intervals may include:

```text
15s
30s
60s
```

Do not automatically use the shortest possible interval.

A shorter interval means:

```text
More samples
More storage
More CPU
More network traffic
```

Choose based on the monitoring requirement.

---

# 30. Evaluation Interval

Alert and recording rules are evaluated periodically.

Example:

```yaml
global:
  evaluation_interval: 30s
```

This should be chosen based on:

```text
Alert requirements
Rule count
Prometheus resource capacity
Operational urgency
```

---

# 31. Service Discovery

In Kubernetes, Prometheus should dynamically discover workloads.

Common mechanisms include:

```text
ServiceMonitor
PodMonitor
Kubernetes service discovery
```

This avoids manually listing every application endpoint.

---

# 32. ServiceMonitor Architecture

```text
Application Service
       ↓
ServiceMonitor
       ↓
Prometheus Operator
       ↓
Prometheus
       ↓
Scrape target
```

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor

metadata:
  name: payment

spec:
  selector:
    matchLabels:
      app: payment

  endpoints:
    - port: metrics
      interval: 30s
```

---

# 33. PodMonitor

PodMonitor can scrape pods directly.

Architecture:

```text
Pod
 ↓
PodMonitor
 ↓
Prometheus
```

This can be useful when the metrics endpoint is exposed directly on pods rather than through a Kubernetes Service.

---

# 34. Application Instrumentation

For production application monitoring, applications should expose meaningful metrics.

For example:

```text
HTTP requests
HTTP errors
Latency
Database calls
Queue operations
Business transactions
```

This allows Prometheus to understand application behavior rather than only infrastructure health.

---

# 35. RED Method

For request-driven services, monitor:

```text
Rate
Errors
Duration
```

Example:

```text
Requests/sec
5xx rate
P95 latency
```

This is particularly useful for microservices.

---

# 36. USE Method

For infrastructure resources:

```text
Utilization
Saturation
Errors
```

Example for a node:

```text
CPU utilization
CPU saturation
Disk errors
Network errors
```

---

# 37. Golden Signals

A production service should usually monitor:

```text
Latency
Traffic
Errors
Saturation
```

These provide a strong starting point for alerting and dashboards.

---

# 38. Recording Rules

Recording rules precompute expensive queries.

Example:

```yaml
groups:
  - name: service-recording-rules

    rules:

      - record: service:http_requests:rate5m

        expr: |
          sum by(service) (
            rate(http_requests_total[5m])
          )
```

This can make frequently used dashboard and alert queries cheaper.

---

# 39. Production Recording Rule Strategy

Create recording rules for:

```text
Frequently used queries
Expensive aggregations
SLO calculations
Dashboard queries
Common alert expressions
```

Avoid creating recording rules for every metric.

---

# 40. Alerting Strategy

A production alerting strategy should include:

```text
Infrastructure alerts
Kubernetes alerts
Application alerts
Database alerts
External endpoint alerts
SLO alerts
Monitoring-system alerts
```

---

# 41. Alert Severity

A common model:

```text
critical
warning
info
```

Example:

```yaml
labels:
  severity: critical
```

Severity should determine routing.

---

# 42. Alert Ownership

Include ownership where appropriate:

```yaml
labels:
  team: payments
  service: payment
  environment: production
```

Then Alertmanager can route:

```text
team=payments
    ↓
Payments Team
```

---

# 43. Runbooks

Critical alerts should have runbooks.

Example:

```text
PaymentHighErrorRate
       ↓
Runbook
       ↓
1. Check Grafana
2. Check application logs
3. Check recent deployment
4. Check dependencies
5. Check database
6. Roll back if necessary
```

---

# 44. Alertmanager Routing

Production routing might look like:

```text
                         Alertmanager
                              │
                    severity=critical?
                         /          \
                       yes           no
                       ↓              ↓
                    Pager          Slack
                       │
                 team=payments?
                    /       \
                  yes        no
                  ↓           ↓
              Payments     Platform
```

This prevents every alert from reaching every engineer.

---

# 45. Alert Grouping

If 50 pods fail during one node outage:

```text
50 individual alerts
```

should not necessarily become:

```text
50 pages
```

Group by useful labels such as:

```text
alertname
cluster
namespace
service
```

---

# 46. Alert Inhibition

Example:

```text
NodeDown
   ↓
PodDown
PodDown
PodDown
```

NodeDown may inhibit related PodDown alerts.

This reduces noise.

---

# 47. Monitoring the Monitoring System

A production observability platform must monitor itself.

Monitor:

```text
Prometheus
Alertmanager
Grafana
Exporters
Long-term storage
Notification delivery
```

---

# 48. Prometheus Self-Monitoring

Important signals:

```text
Prometheus availability
Scrape failures
Rule evaluation failures
TSDB health
WAL failures
Storage usage
Memory
CPU
```

---

# 49. Scrape Failure Monitoring

If:

```text
up == 0
```

for important targets, investigate.

But avoid paging on every temporary scrape failure.

For example:

```text
Target scrape failure
      ↓
warning
      ↓
Persistent failure
      ↓
critical
```

The exact strategy depends on service importance.

---

# 50. Prometheus Rule Failures

Monitor rule evaluation errors.

A broken recording rule can affect:

```text
Dashboards
Alerts
SLO calculations
```

Therefore rule failures themselves should be observable.

---

# 51. Alertmanager Self-Monitoring

Monitor:

```text
Notification failures
Notification latency
Active alerts
Alert processing
Cluster health
```

Otherwise an Alertmanager outage can go unnoticed.

---

# 52. Grafana Self-Monitoring

Monitor:

```text
Grafana availability
Datasource errors
Dashboard loading issues
Resource utilization
```

---

# 53. Exporter Monitoring

Exporters can fail.

Examples:

```text
Node Exporter down
Blackbox Exporter down
Database Exporter down
Redis Exporter down
```

Monitor exporter availability as part of the platform.

---

# 54. Security Architecture

Do not expose Prometheus directly to the public internet.

Preferred:

```text
Internet
   X
Prometheus
```

Instead:

```text
Engineer
   ↓
VPN / SSO / Internal Network
   ↓
Grafana
   ↓
Prometheus
```

---

# 55. Grafana Access

Grafana should provide controlled access through:

```text
SSO
RBAC
Authentication
Authorization
```

Teams should only access the dashboards and data appropriate to their role.

---

# 56. Secrets

Never place secrets directly into:

```text
Git
Prometheus configuration
Grafana dashboards
Alert rules
```

Use:

```text
Kubernetes Secrets
AWS Secrets Manager
External Secrets
Vault
```

according to the organization's architecture.

---

# 57. AWS Security

For EKS-based monitoring:

```text
Private subnets
Security groups
IAM
Pod identity
NetworkPolicies
TLS
```

should be considered.

---

# 58. NetworkPolicy

Monitoring components should communicate only with required workloads.

For example:

```text
Prometheus
   ↓
Allowed
   ↓
Metrics endpoints
```

Unnecessary inbound and outbound traffic should be restricted.

---

# 59. TLS

Use TLS where required for:

```text
Grafana
Prometheus endpoints
Alertmanager
Long-term storage
External integrations
```

The exact implementation depends on the organization's ingress and certificate architecture.

---

# 60. GitOps Architecture

Your monitoring stack can be managed through Git.

Example:

```text
Git Repository
│
├── prometheus
├── alertmanager
├── grafana
├── rules
├── servicemonitors
├── podmonitors
└── dashboards
        │
        ↓
      ArgoCD
        │
        ↓
       EKS
```

This matches the GitOps model used elsewhere in your DevOps platform.

---

# 61. Monitoring Repository Structure

A practical repository could be:

```text
monitoring/
├── prometheus/
│   ├── values.yaml
│   ├── recording-rules/
│   └── alerts/
│
├── alertmanager/
│   └── values.yaml
│
├── grafana/
│   ├── values.yaml
│   └── dashboards/
│
├── exporters/
│   ├── node-exporter/
│   ├── blackbox/
│   └── database/
│
├── servicemonitors/
│
├── podmonitors/
│
└── kustomization.yaml
```

---

# 62. ArgoCD Deployment Flow

```text
Developer
    ↓
Git Pull Request
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
    ↓
Prometheus / Grafana / Alertmanager
```

This provides auditability and rollback.

---

# 63. Monitoring Configuration Changes

Suppose someone changes:

```text
CPU alert threshold
```

Instead of editing production manually:

```text
Git
 ↓
Pull Request
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
 ↓
Production
```

Now the change is traceable.

---

# 64. Production Upgrade Strategy

Before upgrading Prometheus:

```text
1. Review release notes
2. Validate compatibility
3. Test in development
4. Test in staging
5. Backup configuration
6. Verify HA
7. Upgrade production
8. Monitor health
```

---

# 65. Helm-Based Deployment

Prometheus is commonly deployed using Helm charts.

A popular Kubernetes monitoring stack is:

```text
kube-prometheus-stack
```

It commonly packages:

```text
Prometheus
Alertmanager
Grafana
Prometheus Operator
Node Exporter
Kube-State-Metrics
```

The exact components and defaults depend on the chart version.

---

# 66. Example Helm Installation

A typical installation begins with:

```bash
helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

helm repo update
```

Then:

```bash
helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  -n monitoring \
  --create-namespace
```

For production, use a version-pinned chart and a version-controlled values file rather than relying on implicit defaults.

---

# 67. Production Helm Values

Do not keep all configuration on the command line.

Instead:

```text
monitoring-values.yaml
```

Example:

```yaml
prometheus:
  prometheusSpec:
    replicas: 2

    retention: 15d

    resources:
      requests:
        cpu: 500m
        memory: 2Gi
```

Actual resource and retention values should be based on workload measurements.

---

# 68. Version Pinning

Pin versions for production.

For example:

```bash
helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  --version <approved-version> \
  -n monitoring \
  -f monitoring-values.yaml
```

This prevents unexpected chart changes.

---

# 69. GitOps Helm Deployment

With ArgoCD:

```text
Git
 ├── Chart version
 ├── values.yaml
 └── monitoring resources
        ↓
      ArgoCD
        ↓
       EKS
```

This is preferable to manually running Helm commands repeatedly in production.

---

# 70. Production Configuration Validation

Before deployment:

```text
Validate YAML
Validate Helm values
Validate PromQL
Validate PrometheusRule
Validate ServiceMonitor
Validate PodMonitor
Validate Alertmanager configuration
```

Then deploy.

---

# 71. Health Validation

After installation:

```bash
kubectl get pods -n monitoring
```

Check:

```text
Prometheus
Alertmanager
Grafana
Operator
Exporters
Kube-State-Metrics
```

Then:

```bash
kubectl get svc -n monitoring
```

---

# 72. Target Validation

In Prometheus, check:

```text
Status → Targets
```

Verify:

```text
UP
```

for expected targets.

A production installation is not complete just because pods are running.

---

# 73. Rule Validation

Check:

```text
Prometheus → Rules
```

Verify:

```text
Recording rules
Alerting rules
Rule groups
Evaluation status
```

---

# 74. Alert Validation

Test:

```text
Alert fires
Alert reaches Alertmanager
Alert routes correctly
Notification arrives
Alert resolves
Resolved notification arrives if configured
```

---

# 75. Grafana Validation

Check:

```text
Datasource
Dashboards
Queries
Variables
Panels
Alerts if applicable
```

The datasource should return expected Prometheus metrics.

---

# 76. Production Readiness Test

A useful test sequence:

```text
Deploy
 ↓
Prometheus healthy
 ↓
Targets UP
 ↓
Metrics visible
 ↓
Rules loaded
 ↓
Alert test
 ↓
Notification test
 ↓
Grafana dashboards
 ↓
Failure simulation
 ↓
Recovery validation
```

---

# 77. Disaster Recovery

Store monitoring configuration in Git.

If the cluster is lost:

```text
Cluster destroyed
      ↓
New EKS cluster
      ↓
ArgoCD
      ↓
Monitoring manifests
      ↓
Prometheus
Alertmanager
Grafana
Exporters
Rules
```

The monitoring stack can be recreated.

---

# 78. Historical Data Recovery

Configuration recovery does not automatically recover local Prometheus history.

For long-term historical data, use:

```text
Long-term metrics backend
+
Object storage
```

This separates:

```text
Configuration recovery
```

from:

```text
Historical data recovery
```

---

# 79. Backup Strategy

Back up or version-control:

```text
Helm values
PrometheusRules
ServiceMonitors
PodMonitors
Grafana dashboards
Alertmanager configuration
Recording rules
```

For long-term metrics, rely on the durability and backup characteristics of the chosen storage architecture.

---

# 80. Production Cost Optimization

Monitoring can become expensive.

Control costs through:

```text
Metric filtering
Cardinality management
Scrape interval optimization
Retention policies
Recording rules
Long-term storage tiering
Object-storage lifecycle policies
Dashboard optimization
```

---

# 81. Metric Filtering

Do not scrape every metric just because an exporter exposes it.

Ask:

```text
Do we use this metric?
Does it support an alert?
Does it support a dashboard?
Does it support troubleshooting?
```

If not, consider whether it should be retained.

---

# 82. Dashboard Optimization

A dashboard with dozens of panels, each running expensive queries, can overload Prometheus.

Prefer:

```text
Recording rules
Efficient PromQL
Appropriate time ranges
Useful panels
```

Avoid unnecessary high-cardinality queries.

---

# 83. Query Optimization

Avoid unnecessarily broad queries.

Example:

```promql
rate(http_requests_total[30d])
```

may be expensive for large datasets.

Use appropriate ranges:

```promql
rate(http_requests_total[5m])
```

for real-time operational dashboards when appropriate.

---

# 84. Production Capacity Planning

Monitor:

```text
Active series
Samples/sec
Query latency
CPU
Memory
Disk
WAL
Rule evaluation duration
```

Use these measurements to decide when the platform needs scaling.

---

# 85. Prometheus Scaling Options

When Prometheus reaches its limits, possible strategies include:

```text
Optimize metrics
Optimize queries
Reduce cardinality
Use recording rules
Shard Prometheus
Add long-term storage
Use a scalable metrics backend
```

Do not immediately jump to distributed architecture without measuring the bottleneck.

---

# 86. Prometheus Sharding

At larger scale, targets can be divided among multiple Prometheus instances.

Conceptually:

```text
                   Targets
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Prom A       Prom B      Prom C
      targets 1    targets 2    targets 3
```

This reduces the workload handled by one Prometheus instance.

Sharding introduces operational complexity and should be used when justified.

---

# 87. HA + Sharding

A larger architecture may use:

```text
Target Shard 1
 ├── Prom A1
 └── Prom B1

Target Shard 2
 ├── Prom A2
 └── Prom B2

Target Shard 3
 ├── Prom A3
 └── Prom B3
```

This provides:

```text
Scaling
+
HA
```

but significantly increases operational complexity.

---

# 88. Central Query Layer

For multiple Prometheus instances:

```text
Grafana
   ↓
Query Layer
   ├── Prom A
   ├── Prom B
   ├── Prom C
   └── Long-term store
```

This provides a unified interface.

---

# 89. Enterprise Architecture

A mature enterprise platform may look like:

```text
                         Engineers
                             │
                             ↓
                           Grafana
                             │
                             ↓
                      Global Query Layer
                             │
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
     EKS Prod             EKS Stage             EKS Dev
        │                    │                    │
    Prom HA               Prom HA               Prom HA
        │                    │                    │
        └────────────────────┼────────────────────┘
                             ↓
                     Long-Term Metrics
                             │
                             ↓
                           S3

Each cluster:
Prometheus
Alertmanager
Node Exporter
Kube-State-Metrics
ServiceMonitors
PodMonitors
Application Metrics
```

---

# 90. Monitoring Platform Ownership

Define ownership for:

```text
Prometheus platform
Alertmanager
Grafana
Exporters
Dashboards
Alert rules
Metric instrumentation
Long-term storage
```

For example:

```text
Platform Team
    ↓
Prometheus / Grafana / Alertmanager

Application Teams
    ↓
Application metrics / dashboards / alerts
```

---

# 91. Platform vs Application Responsibilities

Platform team:

```text
Prometheus
Alertmanager
Grafana
Exporters
Service discovery
Storage
Security
```

Application team:

```text
Instrumentation
Business metrics
Application dashboards
Application alerts
Runbooks
SLOs
```

This creates clear operational ownership.

---

# 92. Production Change Management

Monitoring changes should follow the same engineering process as application changes:

```text
Change
 ↓
Pull Request
 ↓
Review
 ↓
Testing
 ↓
Approval
 ↓
Deployment
 ↓
Validation
```

Avoid undocumented manual changes.

---

# 93. Observability Platform SLO

The monitoring platform itself should have reliability goals.

For example:

```text
Prometheus availability
Alert delivery availability
Grafana availability
Metric ingestion health
```

The exact SLOs should be defined according to business requirements.

---

# 94. Why Monitoring Availability Matters

Imagine the application is down.

At the same time:

```text
Prometheus → down
Alertmanager → down
Grafana → down
```

Now engineers have very little visibility.

This is why the monitoring platform itself must be treated as production infrastructure.

---

# 95. Monitoring Dependency Principle

Avoid a situation where:

```text
Application
    ↓
Monitoring
    ↓
Application
```

creates a circular dependency.

Monitoring should remain as independent as practical from the application being monitored.

---

# 96. Production Troubleshooting Flow

When monitoring stops working:

```text
1. Check Kubernetes
        ↓
2. Check Prometheus pods
        ↓
3. Check Prometheus targets
        ↓
4. Check storage
        ↓
5. Check rules
        ↓
6. Check Alertmanager
        ↓
7. Check Grafana
        ↓
8. Check long-term backend
        ↓
9. Check network/security
```

---

# 97. Scenario: Metrics Missing

If a dashboard suddenly has no data:

```text
Grafana
   ↓
Datasource
   ↓
Prometheus
   ↓
Query
   ↓
Target
   ↓
Exporter/Application
```

Troubleshoot from the top down.

---

# 98. Scenario: Target DOWN

Check:

```text
Service
Endpoint
Port
NetworkPolicy
SecurityGroup
Pod
Application
Metrics endpoint
ServiceMonitor
PodMonitor
```

---

# 99. Scenario: Prometheus Running but No Metrics

A running Prometheus pod does not guarantee successful scraping.

Check:

```text
Targets
Service discovery
Scrape configuration
RBAC
Network connectivity
TLS
Authentication
Exporter
```

---

# 100. Scenario: High Prometheus Memory

Check:

```text
Active series
High-cardinality labels
Scrape volume
Retention
Query load
Rule evaluation
Dashboard queries
```

Then optimize before scaling.

---

# 101. Scenario: High Prometheus CPU

Check:

```text
Ingestion
Queries
Recording rules
Alert rules
Dashboard refresh rates
Cardinality
Scrape frequency
```

---

# 102. Scenario: Storage Filling Quickly

Check:

```text
Metric cardinality
Scrape interval
Retention
High-volume exporters
Unexpected targets
WAL growth
```

---

# 103. Scenario: Alerts Not Arriving

Check:

```text
Prometheus alert state
Alertmanager targets
Alertmanager routing
Silences
Inhibition
Receiver
Notification provider
```

---

# 104. Scenario: Duplicate Alerts

Check:

```text
Prometheus replica labels
Alertmanager grouping
Alertmanager HA configuration
Label consistency
```

---

# 105. Scenario: Grafana Dashboard Slow

Check:

```text
PromQL complexity
Time range
Number of panels
Cardinality
Prometheus CPU
Query latency
Recording rules
```

---

# 106. Production Monitoring Standards

Establish standards for:

```text
Metric names
Labels
Scrape intervals
Alert names
Severity
Ownership
Runbooks
Dashboard naming
Service discovery
Retention
Security
```

Consistency becomes increasingly important as the environment grows.

---

# 107. Recommended Naming Standards

Metrics:

```text
<domain>_<object>_<measurement>_<unit>
```

Examples:

```text
http_request_duration_seconds
http_requests_total
database_connections
```

Alerts:

```text
<Application><Condition>
<Node><Condition>
<KubernetesObject><Condition>
```

Examples:

```text
PaymentServiceHighErrorRate
NodeDown
DeploymentReplicasMismatch
```

---

# 108. Production Dashboard Structure

A service dashboard might contain:

```text
Overview
 ├── Availability
 ├── Traffic
 ├── Error Rate
 ├── Latency
 ├── Saturation
 ├── Pod Health
 ├── Resource Usage
 └── Dependencies
```

---

# 109. Service Overview

The first dashboard row should answer:

```text
Is the service healthy?
```

Display:

```text
Availability
Error rate
Request rate
P95/P99 latency
Active pods
```

---

# 110. Infrastructure Dashboard

A node dashboard can show:

```text
CPU
Memory
Disk
Filesystem
Network
Load
Pod count
Pressure conditions
```

---

# 111. Kubernetes Dashboard

A cluster dashboard can show:

```text
Node count
Ready nodes
Pod count
Pending pods
Restarting pods
CPU utilization
Memory utilization
PVC usage
Deployment health
```

---

# 112. Alert Dashboard

Create a dashboard for:

```text
Active critical alerts
Active warnings
Alerts by team
Alerts by service
Alert frequency
Alert resolution time
```

This helps identify noisy alert rules.

---

# 113. Monitoring Platform Dashboard

Monitor the monitoring platform:

```text
Prometheus health
Scrape success
Active series
Storage usage
Rule evaluation
Alertmanager health
Notification failures
Grafana health
```

---

# 114. Production Architecture Decision Matrix

```text
Requirement                  Solution

Single cluster                Prometheus

Production HA                 Prometheus replicas

HA alerting                   Alertmanager replicas

Long retention                Long-term metrics backend

Multi-cluster                 Central query layer

Large-scale metrics           Sharding / scalable backend

Durable historical storage    Object storage

Visualization                 Grafana

Kubernetes discovery          ServiceMonitor / PodMonitor

Infrastructure metrics        Node Exporter

Kubernetes state              Kube-State-Metrics
```

---

# 115. Practical Architecture for Your DevOps Environment

For a real-world AWS/EKS microservices project, a practical architecture is:

```text
                         Git
                          │
                     ArgoCD GitOps
                          │
                          ↓
                         EKS
                          │
      ┌───────────────────┼────────────────────┐
      ↓                   ↓                    ↓
Application Pods      Kubernetes State       Nodes
      │                   │                    │
   /metrics         Kube-State-Metrics    Node Exporter
      │                   │                    │
      └───────────────────┼────────────────────┘
                          ↓
                  Prometheus HA
                   ┌──────┴──────┐
                   ↓             ↓
                Prom A         Prom B
                   │             │
                   └──────┬──────┘
                          ↓
                  Alertmanager HA
                          │
                 ┌────────┼────────┐
                 ↓        ↓        ↓
               Slack    Email    Pager

Prometheus
    ↓
Optional Thanos / Mimir
    ↓
S3 / Object Storage

Grafana
    ↓
Prometheus / Query Layer
```

This architecture fits well with a Kubernetes + AWS + GitOps + DevSecOps environment.

---

# 116. Production Implementation Roadmap

Implement the monitoring platform in this order:

```text
Phase 1
Prometheus
    ↓
Phase 2
Node Exporter
    ↓
Phase 3
Kube-State-Metrics
    ↓
Phase 4
ServiceMonitor / PodMonitor
    ↓
Phase 5
Grafana
    ↓
Phase 6
Alertmanager
    ↓
Phase 7
Application Metrics
    ↓
Phase 8
Recording Rules
    ↓
Phase 9
Production Alerts
    ↓
Phase 10
Prometheus HA
    ↓
Phase 11
Alertmanager HA
    ↓
Phase 12
Long-Term Storage
    ↓
Phase 13
GitOps
    ↓
Phase 14
Disaster Recovery
```

---

# 117. Production Readiness Checklist

```text
Infrastructure
[ ] Prometheus HA
[ ] Persistent storage
[ ] Resource requests
[ ] Resource limits
[ ] Node/AZ distribution
[ ] PDB
[ ] Topology spread

Collection
[ ] Service discovery
[ ] ServiceMonitors
[ ] PodMonitors
[ ] Exporters
[ ] Application metrics

Performance
[ ] Cardinality reviewed
[ ] Recording rules
[ ] Query optimization
[ ] Storage monitoring
[ ] Capacity planning

Alerting
[ ] Alert rules
[ ] Severity
[ ] Ownership
[ ] Runbooks
[ ] Alertmanager HA
[ ] Grouping
[ ] Deduplication
[ ] Inhibition

Visualization
[ ] Grafana
[ ] Service dashboards
[ ] Infrastructure dashboards
[ ] Kubernetes dashboards

Storage
[ ] Retention configured
[ ] Long-term storage evaluated
[ ] Object storage configured if required

Security
[ ] Authentication
[ ] Authorization
[ ] TLS
[ ] Secrets
[ ] Network policies
[ ] Private access

Operations
[ ] GitOps
[ ] Version pinning
[ ] Upgrade process
[ ] Backup/recovery
[ ] Failure testing
```

---

# 118. Final Production Architecture

The complete production monitoring platform can be visualized as:

```text
                              USERS
                                │
                                ↓
                         SSO / VPN / IAM
                                │
                                ↓
                             Grafana
                                │
                                ↓
                       Query / Metrics Layer
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
         Prometheus A      Prometheus B      Long-Term Store
              │                 │                 │
            AZ-A              AZ-B                │
              │                 │                 ↓
              │                 │              Amazon S3
              │                 │
              └────────┬────────┘
                       ↓
                Alert Evaluation
                       │
                       ↓
                Alertmanager HA
                       │
            ┌──────────┼──────────┐
            ↓          ↓          ↓
          Slack      Email      Pager
                       │
                       ↓
                   On-Call Team


                         EKS
                          │
        ┌─────────────────┼──────────────────┐
        ↓                 ↓                  ↓
 Application Pods   Kube-State-Metrics   Node Exporter
        │                 │                  │
        │                 │                  │
        └─────────────────┼──────────────────┘
                          ↓
                     Prometheus
```

---

# 119. Final Production Principles

The most important principles to remember are:

```text
1. Monitor the application, not just the infrastructure.

2. Use metrics that represent user impact.

3. Control metric cardinality.

4. Use ServiceMonitor and PodMonitor for Kubernetes discovery.

5. Use recording rules for expensive, frequently used queries.

6. Alert on actionable conditions.

7. Use Alertmanager for grouping, routing, deduplication,
   silencing and inhibition.

8. Run Prometheus and Alertmanager with HA where required.

9. Spread replicas across failure domains.

10. Use persistent storage for local retention.

11. Use long-term storage when scale or retention requires it.

12. Protect monitoring components with authentication,
    authorization and network controls.

13. Manage configuration through GitOps.

14. Monitor the monitoring system itself.

15. Test failure scenarios instead of assuming HA works.

16. Keep dashboards focused on investigation and operations.

17. Define ownership and runbooks for critical alerts.

18. Optimize before scaling.

19. Design disaster recovery separately from basic HA.

20. Treat observability infrastructure as production infrastructure.
```

---

# 120. Final Mental Model

Think of the complete Prometheus platform like this:

```text
                    APPLICATION
                         │
                         ↓
                      METRICS
                         │
                         ↓
                 SERVICE DISCOVERY
                         │
                         ↓
                    PROMETHEUS
                  ┌──────┴──────┐
                  ↓             ↓
              TSDB/RULES     ALERT RULES
                  │             │
                  ↓             ↓
               GRAFANA      ALERTMANAGER
                  │             │
                  ↓        ┌────┼────┐
              DASHBOARDS   ↓    ↓    ↓
                         Slack Email Pager
                  
                  Prometheus
                       │
                       ↓
              LONG-TERM STORAGE
                       │
                       ↓
                   OBJECT STORE
```

The production goal is not simply:

```text
"Install Prometheus."
```

It is:

```text
Collect reliable metrics
        +
Store them efficiently
        +
Query them efficiently
        +
Alert on meaningful conditions
        +
Visualize them clearly
        +
Survive component failures
        +
Retain historical data when required
        +
Secure the platform
        +
Manage everything as code
        +
Test the entire system
```

That is the foundation of a **production-grade Prometheus architecture**.
