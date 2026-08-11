# Grafana Fundamentals

## 1. What Is Grafana?

Grafana is an open-source observability and visualization platform used to query, visualize, explore, and monitor data from different sources.

In a DevOps environment, Grafana is commonly used to visualize:

```text
Metrics
Logs
Traces
Infrastructure health
Application performance
Kubernetes resources
Business metrics
Alerts
```

Grafana itself is primarily a visualization and analysis layer. It does not replace the systems that collect and store the underlying observability data.

A common architecture is:

```text
Applications
     │
     ├── Metrics ──→ Prometheus
     │
     ├── Logs ─────→ Loki / Elasticsearch
     │
     └── Traces ───→ Jaeger / Tempo
                         │
                         ↓
                       Grafana
```

Grafana provides a common interface for engineers to investigate these signals.

---

# 2. Why Do We Need Grafana?

Prometheus stores metrics and provides PromQL, but engineers still need a convenient way to visualize those metrics.

Without Grafana:

```text
Engineer
   ↓
Prometheus
   ↓
PromQL
   ↓
Raw query results
```

With Grafana:

```text
Engineer
   ↓
Grafana
   ↓
Prometheus
   ↓
PromQL
   ↓
Charts / Tables / Gauges / Stat Panels
```

Grafana makes operational data easier to understand.

---

# 3. Grafana in a DevOps Environment

A typical monitoring stack might be:

```text
                    Kubernetes / AWS
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
     Metrics             Logs              Traces
        │                  │                  │
        ↓                  ↓                  ↓
   Prometheus        Elasticsearch          Jaeger
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ↓
                        Grafana
                           │
                           ↓
                      Engineers
```

Grafana acts as a unified visualization and investigation layer.

---

# 4. Grafana Is Not Prometheus

This distinction is important.

### Prometheus

Prometheus is responsible for:

```text
Metric collection
Metric storage
PromQL
Recording rules
Alert rules
Service discovery
```

### Grafana

Grafana is responsible for:

```text
Visualization
Dashboards
Queries
Exploration
Annotations
Dashboard variables
Alert visualization
Data-source integration
```

Simplified:

```text
Prometheus = Metrics backend

Grafana = Visualization and analysis layer
```

---

# 5. Grafana Is Not a Database

Grafana generally does not act as the primary storage system for your observability data.

Instead:

```text
Grafana
   │
   ├──→ Prometheus
   ├──→ Elasticsearch
   ├──→ Loki
   ├──→ Jaeger
   └──→ Other data sources
```

Grafana sends queries to those systems and visualizes the returned data.

---

# 6. Grafana Data Sources

Grafana supports many types of data sources.

For a modern observability platform, you may use:

```text
Prometheus
Loki
Elasticsearch
Jaeger
Tempo
InfluxDB
MySQL
PostgreSQL
Cloud databases
```

For the monitoring and observability stack in this chapter, the important integrations are:

```text
Prometheus → Metrics
Elasticsearch / Loki → Logs
Jaeger → Traces
```

---

# 7. Metrics With Grafana

Suppose Prometheus contains:

```text
http_requests_total
```

Grafana can query it using PromQL:

```promql
rate(http_requests_total[5m])
```

Grafana can then display the result as:

```text
Time series
Stat
Gauge
Bar chart
Table
```

---

# 8. Example CPU Dashboard

Suppose Prometheus collects node CPU metrics.

Grafana can display:

```text
+--------------------------------------+
|          CPU UTILIZATION             |
|                                      |
|  90% ┤                         ╭───   |
|  70% ┤              ╭──────────╯      |
|  50% ┤       ╭──────╯                |
|  30% ┤───────╯                       |
|   0% └────────────────────────────── |
|        Time →                        |
+--------------------------------------+
```

The engineer can immediately see whether CPU utilization is increasing.

---

# 9. Grafana Dashboard

A dashboard is a collection of panels.

For example:

```text
Production Kubernetes Dashboard
│
├── Cluster CPU
├── Cluster Memory
├── Node Health
├── Pod Count
├── Pod Restarts
├── Network Traffic
├── API Request Rate
├── Error Rate
└── P95 Latency
```

Each panel can execute its own query against a data source.

---

# 10. Grafana Panel

A panel is an individual visualization.

Examples:

```text
Time Series
Stat
Gauge
Table
Bar Chart
Heatmap
Logs
Geomap
```

Example:

```text
Dashboard
   │
   ├── Panel 1 → CPU
   ├── Panel 2 → Memory
   ├── Panel 3 → Requests
   └── Panel 4 → Errors
```

---

# 11. Time-Series Panel

The most common panel for Prometheus metrics is a time-series graph.

Example:

```text
Requests/sec

100 ┤                    ╭─────
 80 ┤              ╭─────╯
 60 ┤        ╭─────╯
 40 ┤   ╭────╯
 20 ┤───╯
  0 └──────────────────────────
       10:00  10:05  10:10
```

This is useful for:

```text
CPU
Memory
Request rate
Latency
Error rate
Network traffic
Disk usage
```

---

# 12. Stat Panel

A Stat panel displays a single important value.

Example:

```text
┌─────────────────────┐
│     ERROR RATE      │
│                     │
│       2.4%          │
│                     │
└─────────────────────┘
```

Useful for:

```text
Current error rate
Available replicas
Active alerts
Request rate
Current CPU
Current memory
```

---

# 13. Gauge Panel

A gauge displays a value against a threshold.

Example:

```text
        CPU

      ┌───────┐
     /         \
    /    78%    \
   |             |
    \           /
     \_________/

```

Useful for:

```text
CPU utilization
Memory utilization
Disk utilization
SLO percentage
Capacity
```

---

# 14. Table Panel

A table is useful when comparing multiple services.

Example:

```text
Service       Requests/s    Errors    P95
------------------------------------------------
payment          250          2%     420ms
orders           380          1%     180ms
catalog          720        0.3%     110ms
user             190        0.5%     150ms
```

This is particularly useful for service-level dashboards.

---

# 15. Dashboard Variables

Grafana variables make dashboards reusable.

Instead of creating separate dashboards for:

```text
Production
Staging
Development
```

you can create one dashboard with a variable:

```text
Environment:
[ production ▼ ]
```

Then the engineer can select:

```text
production
staging
development
```

---

# 16. Kubernetes Dashboard Variables

A Kubernetes dashboard might provide:

```text
Cluster:
[ production ▼ ]

Namespace:
[ payments ▼ ]

Service:
[ payment-api ▼ ]

Pod:
[ payment-api-7f9c8 ▼ ]
```

The same dashboard can then be used for different workloads.

---

# 17. Why Variables Matter

Without variables:

```text
Production Dashboard
Staging Dashboard
Development Dashboard
Payment Dashboard
Order Dashboard
User Dashboard
Catalog Dashboard
```

This can become difficult to maintain.

With variables:

```text
One reusable dashboard
        ↓
Multiple environments
Multiple namespaces
Multiple services
Multiple pods
```

---

# 18. Grafana Query Model

A dashboard panel generally follows this flow:

```text
Panel
  ↓
Data Source
  ↓
Query
  ↓
Backend
  ↓
Results
  ↓
Visualization
```

For Prometheus:

```text
Grafana
   ↓
PromQL
   ↓
Prometheus
   ↓
Time Series
   ↓
Grafana Panel
```

---

# 19. Example PromQL Query

Suppose we want request rate:

```promql
sum(
  rate(http_requests_total[5m])
)
```

Grafana sends this query to Prometheus.

Prometheus calculates the result.

Grafana visualizes it.

---

# 20. Service-Level Query

For a microservices environment:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

This can produce:

```text
payment    250 req/s
orders     380 req/s
catalog    720 req/s
user       190 req/s
```

Grafana can display this as a table or graph.

---

# 21. Error Rate Dashboard

A useful application dashboard includes error rate.

For example:

```promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
```

This allows engineers to identify services generating server errors.

The exact metric and labels depend on the application's instrumentation.

---

# 22. Latency Dashboard

For histogram-based metrics, Grafana can visualize latency percentiles.

Example:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

This represents an approximate P95 latency calculation when the underlying histogram metric is correctly instrumented.

---

# 23. Golden Signals Dashboard

A production service dashboard should commonly include:

```text
Traffic
Errors
Latency
Saturation
```

Example:

```text
┌────────────────────────────────────────────┐
│           PAYMENT SERVICE                  │
├────────────┬────────────┬──────────────────┤
│ Traffic    │ Errors     │ P95 Latency      │
│ 250 req/s  │ 0.8%       │ 180ms            │
├────────────┴────────────┴──────────────────┤
│ Request Rate                                │
│ ────────────────────────────────            │
├─────────────────────────────────────────────┤
│ Error Rate                                  │
│ ────────────────────────────────            │
├─────────────────────────────────────────────┤
│ Latency                                     │
│ ────────────────────────────────            │
└─────────────────────────────────────────────┘
```

---

# 24. Infrastructure Dashboard

A production Kubernetes infrastructure dashboard might contain:

```text
Cluster Overview

Nodes
 ├── CPU
 ├── Memory
 ├── Disk
 └── Network

Pods
 ├── Running
 ├── Pending
 ├── Failed
 └── Restarting

Workloads
 ├── Deployments
 ├── StatefulSets
 └── DaemonSets
```

---

# 25. Kubernetes Dashboard

A useful Kubernetes dashboard can contain:

```text
Cluster
 ├── Node CPU
 ├── Node Memory
 ├── Node Status
 │
 ├── Pod Count
 ├── Pod Restarts
 ├── Pending Pods
 │
 ├── Deployment Availability
 ├── Replica Mismatch
 │
 └── PVC Usage
```

---

# 26. Grafana Explore

Grafana Explore is used for interactive investigation.

Instead of opening a dashboard:

```text
Engineer
   ↓
Explore
   ↓
Select Data Source
   ↓
Write Query
   ↓
Investigate
```

This is particularly useful during incidents.

---

# 27. Example Incident Investigation

Suppose users report:

```text
Application is slow.
```

Engineer opens Grafana Explore.

First query:

```promql
rate(http_requests_total[5m])
```

Then:

```text
Request rate
```

Next:

```text
Error rate
```

Next:

```text
P95 latency
```

Then:

```text
CPU
Memory
Pod restarts
```

This helps narrow down the problem.

---

# 28. Grafana and Logs

Grafana can also visualize logs if a supported log data source is configured.

For example:

```text
Grafana
   ↓
Elasticsearch
   ↓
Application Logs
```

or:

```text
Grafana
   ↓
Loki
   ↓
Application Logs
```

This allows engineers to move from metrics to logs during investigation.

---

# 29. Metrics → Logs Investigation

A useful workflow:

```text
High Error Rate
      ↓
Grafana Metric Dashboard
      ↓
Identify Service
      ↓
Open Logs
      ↓
Find Error
      ↓
Identify Root Cause
```

For example:

```text
Payment error rate ↑
       ↓
Payment pod identified
       ↓
Logs show DB connection error
       ↓
Investigate database
```

---

# 30. Metrics → Traces Investigation

Grafana can also be used as an investigation interface for traces when a tracing data source such as Jaeger is configured.

Flow:

```text
High latency
    ↓
Grafana
    ↓
Identify service
    ↓
Find trace
    ↓
Open Jaeger trace
    ↓
Identify slow dependency
```

This is especially valuable in microservices architectures.

---

# 31. Three Observability Signals

Your observability platform can connect:

```text
Metrics
   ↓
Logs
   ↓
Traces
```

Conceptually:

```text
                Grafana
              /    |    \
             /     |     \
            ↓      ↓      ↓
       Prometheus  Logs  Jaeger
         Metrics          Traces
```

This creates a unified investigation experience.

---

# 32. Grafana Annotations

Annotations allow events to appear on dashboards.

For example:

```text
Deployment
    │
    ↓
──────────────────────────────
             │
             ↓
        Deployment v1.4
```

An engineer can correlate:

```text
Deployment
+
Latency increase
+
Error increase
```

---

# 33. Deployment Correlation

Suppose:

```text
10:00 → Deployment
10:03 → Error rate increases
10:05 → Latency increases
```

A dashboard annotation makes the relationship visible.

This is extremely useful during production incidents.

---

# 34. Grafana Alerting

Grafana also has its own alerting capabilities.

Conceptually:

```text
Data Source
    ↓
Query
    ↓
Condition
    ↓
Alert Rule
    ↓
Contact Point
    ↓
Notification
```

However, in a Prometheus-based architecture, alerting responsibilities should be designed consistently rather than creating duplicate alerting systems without a clear reason.

---

# 35. Prometheus Alerting vs Grafana Alerting

Prometheus commonly handles:

```text
Prometheus
   ↓
Alert Rules
   ↓
Alertmanager
```

Grafana can also evaluate alert rules:

```text
Grafana
   ↓
Alert Rule
   ↓
Contact Point
```

Choose the architecture intentionally.

Avoid creating the same alert in both systems unless there is a clear operational requirement.

---

# 36. Grafana Authentication

Production Grafana should not be anonymously accessible.

Common authentication methods include:

```text
Local users
LDAP
OAuth
OIDC
SAML
Enterprise SSO
```

For enterprise environments, centralized identity is generally preferred.

---

# 37. Grafana Authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to access?
```

Grafana can use:

```text
Organizations
Teams
Roles
Folder permissions
Dashboard permissions
Data-source permissions
```

depending on the deployment and edition.

---

# 38. Grafana Organizations

An organization provides separation between groups of Grafana resources.

For example:

```text
Organization A
 ├── Dashboards
 ├── Users
 └── Data Sources
```

However, many enterprise deployments prefer a centralized organization with Teams and RBAC rather than creating many isolated organizations.

---

# 39. Grafana Teams

Teams allow access to be organized around responsibilities.

Example:

```text
Platform Team
 ├── Kubernetes dashboards
 └── Infrastructure dashboards

Payments Team
 ├── Payment dashboards
 └── Payment alerts

Application Team
 ├── Application dashboards
 └── Service dashboards
```

This provides clearer ownership.

---

# 40. Grafana Folders

Folders organize dashboards.

Example:

```text
Dashboards
│
├── Kubernetes
│   ├── Cluster Overview
│   ├── Node Health
│   └── Pod Health
│
├── Applications
│   ├── Payment
│   ├── Orders
│   └── Catalog
│
└── Infrastructure
    ├── EC2
    ├── RDS
    └── Load Balancers
```

---

# 41. Dashboard Permissions

A production dashboard may contain sensitive infrastructure information.

Control who can:

```text
View
Edit
Administer
```

the dashboard.

Do not give every user administrative access.

---

# 42. Grafana Data-Source Permissions

A data source may provide access to:

```text
Production metrics
Production logs
Infrastructure data
Business metrics
```

Protect access to sensitive data sources.

---

# 43. Grafana Configuration

Grafana configuration can be managed through:

```text
grafana.ini
Environment variables
Helm values
Provisioning files
```

For Kubernetes environments, Helm values and provisioning are commonly used.

---

# 44. Grafana Configuration File

A traditional installation uses:

```text
/etc/grafana/grafana.ini
```

Important configuration areas include:

```text
[server]
[security]
[database]
[auth]
[users]
[analytics]
[alerting]
```

The exact settings depend on the Grafana version and deployment model.

---

# 45. Grafana Database

Grafana stores its own configuration and metadata.

It may use:

```text
SQLite
MySQL
PostgreSQL
```

For a production HA deployment, an external database such as PostgreSQL is commonly preferred over local SQLite because multiple Grafana instances need shared persistent state.

---

# 46. Grafana HA

A production HA architecture can look like:

```text
                  Load Balancer
                       │
              ┌────────┴────────┐
              ↓                 ↓
          Grafana A         Grafana B
              │                 │
              └────────┬────────┘
                       ↓
                 PostgreSQL
```

Both Grafana instances share the database.

---

# 47. Grafana in Kubernetes

A Kubernetes deployment may look like:

```text
                    Ingress
                       │
                       ↓
                   Grafana Service
                       │
              ┌────────┴────────┐
              ↓                 ↓
          Grafana Pod A      Grafana Pod B
              │                 │
              └────────┬────────┘
                       ↓
                  PostgreSQL
```

The actual architecture depends on whether HA is required.

---

# 48. Grafana Persistent Storage

If Grafana uses local storage for configuration or plugins, persistent storage may be required.

However, for a properly designed HA deployment, important state should be externalized or provisioned so that Grafana pods can be recreated safely.

---

# 49. Grafana Provisioning

Provisioning allows configuration to be managed as code.

You can provision:

```text
Data sources
Dashboards
Alert rules
Contact points
Notification policies
```

This is extremely useful in GitOps environments.

---

# 50. Data Source Provisioning

Conceptually:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus-server:9090
    isDefault: true
```

The exact URL depends on the Kubernetes Service and deployment.

---

# 51. Why Provision Data Sources?

Without provisioning:

```text
Engineer
   ↓
Login to Grafana
   ↓
Create data source manually
```

With provisioning:

```text
Git
 ↓
Configuration
 ↓
ArgoCD
 ↓
Grafana
 ↓
Data source automatically configured
```

This supports repeatable environments.

---

# 52. Dashboard as Code

Dashboards can also be stored in Git.

Example:

```text
grafana/
├── dashboards/
│   ├── kubernetes-cluster.json
│   ├── node-health.json
│   └── payment-service.json
```

Then:

```text
Git
 ↓
ArgoCD
 ↓
Grafana
 ↓
Dashboards
```

---

# 53. Why Dashboard as Code?

Advantages:

```text
Version control
Code review
Rollback
Environment consistency
Auditability
Disaster recovery
```

If a dashboard is accidentally deleted, it can be recreated from Git.

---

# 54. Grafana and GitOps

A production architecture can be:

```text
Git
│
├── Grafana Helm values
├── Data source provisioning
├── Dashboard definitions
├── Alert configuration
└── Folder configuration
       │
       ↓
     ArgoCD
       │
       ↓
      EKS
       │
       ↓
    Grafana
```

This fits naturally into the DevOps workflow.

---

# 55. Grafana Dashboard Lifecycle

Use:

```text
Create
 ↓
Test
 ↓
Review
 ↓
Commit
 ↓
Deploy
 ↓
Validate
 ↓
Maintain
```

Avoid unmanaged production dashboards.

---

# 56. Dashboard Naming

Use consistent names.

Examples:

```text
Kubernetes / Cluster Overview
Kubernetes / Node Health
Kubernetes / Pod Health
Application / Payment Service
Application / Orders Service
Infrastructure / RDS
Infrastructure / Load Balancer
```

---

# 57. Dashboard Design Principles

A good dashboard should answer:

```text
What is happening?
Why is it happening?
Where is it happening?
How severe is it?
What should I investigate next?
```

Avoid dashboards containing dozens of unrelated graphs.

---

# 58. Executive Dashboard

An executive or high-level dashboard might contain:

```text
Service Availability
SLO Compliance
Critical Incidents
Error Rate
Request Volume
Major Infrastructure Health
```

Keep it simple.

---

# 59. Operations Dashboard

An operations dashboard should contain:

```text
CPU
Memory
Disk
Network
Pod health
Node health
Alerts
Errors
Latency
```

This is intended for active troubleshooting.

---

# 60. Application Dashboard

An application dashboard should contain:

```text
Request rate
Error rate
P50 latency
P95 latency
P99 latency
Active requests
Dependency health
Pod health
```

---

# 61. Kubernetes Dashboard

For Kubernetes:

```text
Cluster
 ↓
Nodes
 ↓
Namespaces
 ↓
Deployments
 ↓
Pods
 ↓
Containers
```

The dashboard should allow engineers to move from high-level health to individual workloads.

---

# 62. Drill-Down Design

A strong Grafana experience allows:

```text
Cluster
  ↓
Namespace
  ↓
Service
  ↓
Pod
  ↓
Container
  ↓
Logs
  ↓
Trace
```

This is much more useful than isolated dashboards.

---

# 63. Grafana Links

Dashboard links can connect related dashboards.

Example:

```text
Kubernetes Cluster
      ↓
Namespace Dashboard
      ↓
Payment Service
      ↓
Pod Dashboard
```

This reduces manual navigation during incidents.

---

# 64. Metrics-to-Logs Correlation

A panel showing:

```text
Payment Error Rate
```

can be used as the starting point for log investigation.

Workflow:

```text
Metric spike
   ↓
Identify service
   ↓
Open logs
   ↓
Filter by service/pod
   ↓
Find error
```

---

# 65. Metrics-to-Traces Correlation

For distributed tracing:

```text
Latency spike
   ↓
Identify service
   ↓
Trace query
   ↓
Jaeger
   ↓
Trace spans
   ↓
Slow dependency
```

This is especially useful for microservices.

---

# 66. Grafana Explore for Prometheus

Example workflow:

```text
Explore
 ↓
Prometheus
 ↓
Query
```

Example:

```promql
sum by (namespace) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

This can help identify which namespaces consume the most CPU.

---

# 67. Grafana Explore for Logs

With Elasticsearch or another log source:

```text
Explore
 ↓
Elasticsearch
 ↓
Search logs
```

Example search concept:

```text
service="payment"
AND level="ERROR"
```

The exact query language depends on the configured log data source.

---

# 68. Grafana Explore for Jaeger

With Jaeger:

```text
Explore
 ↓
Jaeger
 ↓
Service
 ↓
Operation
 ↓
Trace
```

The engineer can inspect:

```text
Trace duration
Span duration
Service dependencies
Errors
Tags
```

---

# 69. Observability Investigation Example

Imagine users report:

```text
Checkout is slow.
```

Engineer starts with Grafana.

### Step 1 — Metrics

```text
P95 latency ↑
```

### Step 2 — Service

```text
checkout-service
```

### Step 3 — Logs

```text
Database timeout
```

### Step 4 — Trace

```text
checkout
 ↓
payment
 ↓
database
```

### Step 5 — Root cause

```text
Database query latency
```

Grafana becomes the central investigation interface.

---

# 70. Grafana Alerting Fundamentals

Grafana alerts consist conceptually of:

```text
Query
 ↓
Condition / Expression
 ↓
Evaluation
 ↓
Alert State
 ↓
Notification Policy
 ↓
Contact Point
```

Example:

```text
CPU > 80%
for 10 minutes
```

can trigger an alert.

---

# 71. Alert States

An alert may be:

```text
Normal
Pending
Firing
No Data
Error
```

The exact behavior depends on the configured alert rule.

---

# 72. Alert Evaluation

Example:

```text
CPU > 80%
```

If the condition is true for:

```text
10 minutes
```

the alert can transition into a firing state.

This prevents short spikes from immediately paging engineers.

---

# 73. Alert Labels

Use labels to identify ownership.

Example:

```yaml
labels:
  severity: critical
  team: platform
  service: kubernetes
  environment: production
```

These labels can be used for routing and filtering.

---

# 74. Contact Points

A contact point defines where notifications go.

Examples:

```text
Slack
Email
PagerDuty
Webhook
Microsoft Teams
```

Use the notification mechanism appropriate to your organization.

---

# 75. Notification Policies

Notification policies determine where alerts should be sent.

Example:

```text
team=payments
    ↓
Payments Slack

severity=critical
    ↓
PagerDuty

severity=warning
    ↓
Slack
```

---

# 76. Alert Noise

A production Grafana environment should avoid:

```text
Too many alerts
Duplicate alerts
Non-actionable alerts
Alerts without owners
Alerts without runbooks
```

The goal is:

```text
Fewer
+
Actionable
+
Reliable
alerts
```

---

# 77. Grafana Security

Production Grafana should have:

```text
Authentication
Authorization
TLS
Secure cookies
Secret management
Network restrictions
```

Do not expose administrative interfaces publicly without appropriate protection.

---

# 78. Admin Access

Limit Grafana administrator privileges.

Use:

```text
Admin
Editor
Viewer
Team-based permissions
```

where supported.

Most users should not need administrator access.

---

# 79. Secret Management

Avoid:

```text
password: mypassword
```

in Git.

Use:

```text
Kubernetes Secrets
External Secrets
AWS Secrets Manager
Vault
```

depending on the architecture.

---

# 80. Production Grafana Architecture on EKS

A practical architecture:

```text
                        Users
                          │
                          ↓
                    ALB / Ingress
                          │
                          ↓
                    Grafana Service
                          │
                 ┌────────┴────────┐
                 ↓                 ↓
             Grafana A         Grafana B
                 │                 │
                 └────────┬────────┘
                          ↓
                     PostgreSQL
                          │
          ┌───────────────┼────────────────┐
          ↓               ↓                ↓
     Prometheus       Elasticsearch       Jaeger
       Metrics            Logs            Traces
```

This provides a centralized observability UI.

---

# 81. Grafana Availability

For production HA:

```text
Grafana A
Grafana B
```

should ideally run on separate nodes and, where appropriate, separate availability zones.

Use:

```text
Pod anti-affinity
Topology spread
PodDisruptionBudget
```

as appropriate.

---

# 82. Grafana Database HA

If Grafana uses PostgreSQL:

```text
Grafana A
Grafana B
    │
    ↓
PostgreSQL HA
```

The database itself becomes an important dependency.

Therefore:

```text
Grafana HA
without
database resilience
```

does not provide complete HA.

---

# 83. Grafana Plugins

Grafana can use plugins for:

```text
Data sources
Panels
Applications
```

In production:

```text
Pin versions
Review plugins
Install only required plugins
Track them in configuration
```

Avoid uncontrolled plugin installation.

---

# 84. Grafana Upgrade

A safe production upgrade:

```text
Review release notes
        ↓
Check compatibility
        ↓
Backup configuration
        ↓
Test in development
        ↓
Test staging
        ↓
Upgrade production
        ↓
Verify dashboards
        ↓
Verify data sources
        ↓
Verify alerts
```

---

# 85. Grafana Backup

Back up or version-control:

```text
Dashboards
Data source definitions
Alert rules
Provisioning
Grafana configuration
```

If using an external Grafana database, ensure its backup and recovery strategy is also defined.

---

# 86. Grafana Disaster Recovery

If the Grafana pods are destroyed:

```text
Git
 ↓
Provisioning
 ↓
Grafana
 ↓
Dashboards
Data sources
Configuration
```

can be recreated.

If the Grafana database contains important state, the database must also be recoverable.

---

# 87. Grafana Performance

Grafana performance can be affected by:

```text
Large dashboards
Many panels
Expensive queries
Frequent refreshes
Large time ranges
Slow data sources
Too many concurrent users
```

---

# 88. Dashboard Refresh Rate

Do not refresh dashboards every second unless absolutely necessary.

For most operational dashboards:

```text
5s
10s
30s
1m
```

may be more appropriate depending on the use case.

Faster refreshes increase query load.

---

# 89. Dashboard Time Range

Avoid unnecessarily querying months of data for a real-time incident dashboard.

For example:

```text
Last 15 minutes
Last 1 hour
Last 6 hours
```

may be appropriate for active troubleshooting.

Historical analysis can use longer ranges when necessary.

---

# 90. Query Load

Suppose:

```text
Dashboard
 ├── 20 panels
 ├── 10-second refresh
 └── complex PromQL
```

This can generate significant query traffic.

Optimize:

```text
Panel count
Refresh interval
PromQL
Recording rules
Time range
```

---

# 91. Grafana Production Checklist

```text
[ ] Authentication configured
[ ] Authorization configured
[ ] TLS configured
[ ] Admin access restricted
[ ] Data sources configured
[ ] Dashboards version-controlled
[ ] Provisioning configured
[ ] Alerting configured
[ ] Contact points configured
[ ] Notification policies configured
[ ] Grafana HA evaluated
[ ] External database evaluated for HA
[ ] Persistent state protected
[ ] Plugins reviewed
[ ] Resource requests configured
[ ] Resource limits configured
[ ] Pod anti-affinity configured
[ ] PDB configured
[ ] Backup strategy
[ ] Disaster recovery tested
[ ] GitOps configured
```

---

# 92. Grafana + Prometheus Production Flow

The core monitoring flow is:

```text
Kubernetes
   │
   ├── Applications
   ├── Nodes
   └── Kubernetes Objects
          │
          ↓
      Prometheus
          │
          ↓
       PromQL
          │
          ↓
       Grafana
          │
          ↓
      Dashboards
          │
          ↓
       Engineers
```

---

# 93. Full Observability Flow

The complete observability platform is:

```text
                         Kubernetes / AWS
                                │
          ┌─────────────────────┼─────────────────────┐
          ↓                     ↓                     ↓
       Metrics                Logs                  Traces
          │                     │                     │
          ↓                     ↓                     ↓
     Prometheus           Elasticsearch/Loki        Jaeger
          │                     │                     │
          └─────────────────────┼─────────────────────┘
                                ↓
                             Grafana
                                │
             ┌──────────────────┼──────────────────┐
             ↓                  ↓                  ↓
          Dashboards         Explore             Alerts
             │                  │                  │
             └──────────────────┼──────────────────┘
                                ↓
                            Engineers
```

---

# 94. Real-World Microservices Example

Suppose your EKS platform contains:

```text
user-service
catalog-service
cart-service
order-service
payment-service
inventory-service
notification-service
```

Grafana can provide:

```text
Cluster Dashboard
        ↓
Service Dashboard
        ↓
Payment Dashboard
        ↓
Pod Dashboard
        ↓
Logs
        ↓
Traces
```

---

# 95. Payment Service Dashboard

A practical dashboard might show:

```text
Payment Service
──────────────────────────────────────

Request Rate       250 req/s
Error Rate         0.8%
P95 Latency        180 ms
P99 Latency        420 ms
Active Pods        5/5

──────────────────────────────────────
Request Rate Graph

──────────────────────────────────────
Error Rate Graph

──────────────────────────────────────
Latency Graph

──────────────────────────────────────
Pod CPU / Memory

──────────────────────────────────────
Recent Errors
```

---

# 96. Incident Example

Suppose payment failures increase.

Grafana:

```text
Error Rate
    ↑
    │
    └── Payment Service
```

Engineer checks:

```text
CPU → normal
Memory → normal
Pods → healthy
```

Then checks logs:

```text
Database connection timeout
```

Then traces:

```text
payment-service
      ↓
database
      ↓
slow query
```

Now the root cause becomes much easier to identify.

---

# 97. Grafana as the Investigation Hub

The key idea is:

```text
Prometheus
    ↓
Metrics

Elasticsearch / Loki
    ↓
Logs

Jaeger
    ↓
Traces

       ↓
     Grafana
       ↓
Investigation
```

Grafana does not necessarily replace each specialized backend.

Instead, it provides a common interface for working with them.

---

# 98. Common Grafana Mistakes

### Mistake 1

Creating dashboards without understanding the underlying metrics.

### Mistake 2

Using expensive PromQL everywhere.

### Mistake 3

Creating hundreds of dashboards without ownership.

### Mistake 4

Giving everyone admin access.

### Mistake 5

Manually configuring production dashboards.

### Mistake 6

No dashboard version control.

### Mistake 7

No alert ownership.

### Mistake 8

Too many panels.

### Mistake 9

Very frequent dashboard refreshes.

### Mistake 10

Treating Grafana as the metrics database.

---

# 99. Grafana Production Principles

Remember:

```text
Grafana
=
Visualization
+
Exploration
+
Dashboarding
+
Observability Integration
```

Not:

```text
Grafana
=
Primary Metrics Storage
```

---

# 100. Interview Answer: What Is Grafana?

A strong production answer:

```text
"Grafana is an open-source observability visualization and
analysis platform.

In our Kubernetes environment, I use Grafana primarily with
Prometheus for metrics visualization.

Prometheus collects and stores the metrics, while Grafana
queries Prometheus using PromQL and presents the results through
dashboards.

Grafana can also integrate with log and tracing backends such as
Elasticsearch, Loki and Jaeger, allowing engineers to correlate
metrics, logs and traces during troubleshooting."
```

---

# 101. Interview Answer: Why Use Grafana With Prometheus?

```text
"Prometheus provides the metric collection, storage and PromQL
query engine, but Grafana provides a much richer visualization
and dashboarding experience.

I can create reusable dashboards for Kubernetes, infrastructure
and applications and use variables to filter by cluster,
namespace, service and pod.

During incidents, Grafana also provides Explore functionality
that allows engineers to investigate metrics and, when configured,
correlate them with logs and traces."
```

---

# 102. Interview Answer: How Do You Deploy Grafana in Kubernetes?

```text
"I commonly deploy Grafana using Helm or the kube-prometheus-stack
depending on the monitoring architecture.

For production, I manage the Helm values through Git and deploy
using GitOps with ArgoCD.

I configure authentication, data sources, dashboards, resources,
persistent state and, where HA is required, multiple Grafana
replicas with an external database.

I also distribute the pods across nodes or availability zones
and validate the dashboards and data sources after deployment."
```

---

# 103. Interview Answer: How Do You Manage Grafana Dashboards?

```text
"I prefer dashboard-as-code for production.

Dashboards are stored in Git, reviewed through pull requests and
deployed through ArgoCD.

Grafana provisioning can automatically load the dashboards and
data sources.

This gives us version control, auditability, rollback and
consistent configuration across environments."
```

---

# 104. Interview Answer: How Would You Troubleshoot a Grafana Dashboard With No Data?

```text
"First I would check whether Grafana itself is healthy.

Then I would verify the configured data source.

If the data source is Prometheus, I would test the connection and
run the same query directly in Prometheus.

Then I would check whether the target is being scraped, whether
the metric exists, and whether the dashboard variables are
filtering the query incorrectly.

So my flow would be:

Grafana
→ Data source
→ Query
→ Prometheus
→ Target
→ Exporter/application."
```

---

# 105. Interview Answer: How Do You Troubleshoot a Slow Grafana Dashboard?

```text
"I would first identify which panel is slow.

Then I would inspect the PromQL query, time range, refresh interval
and cardinality.

I would check whether the query can be optimized or moved into a
recording rule.

I would also check Prometheus CPU, memory and query latency.

Finally, I would look at whether the dashboard has too many panels
or is refreshing too frequently."
```

---

# 106. Interview Answer: How Do You Make Grafana Highly Available?

```text
"I would run multiple Grafana replicas behind a load balancer.

For production HA, I would avoid relying on local SQLite and use
a shared external database such as PostgreSQL for Grafana state.

I would distribute Grafana replicas across nodes and availability
zones where appropriate.

I would also manage dashboards and configuration through GitOps so
that instances can be recreated consistently."
```

---

# 107. Interview Answer: How Do Metrics, Logs and Traces Work Together?

```text
"I use metrics to detect that something is wrong, logs to understand
what happened, and traces to understand where the request spent time
across distributed services.

For example, if Grafana shows increased latency for the payment
service, I would identify the affected service from metrics, inspect
the application logs for errors, and then use distributed tracing
through Jaeger to identify which downstream service or database
operation is causing the latency."
```

---

# 108. Final Grafana Mental Model

Remember this:

```text
                  GRAFANA
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     Metrics        Logs        Traces
        │            │            │
        ↓            ↓            ↓
  Prometheus    Elasticsearch   Jaeger
                    / Loki
        │            │            │
        └────────────┼────────────┘
                     ↓
              Investigation
                     ↓
                 Engineers
```

And the most important distinction:

```text
Prometheus
→ Collects and stores metrics

Grafana
→ Queries and visualizes metrics

Alertmanager
→ Routes and manages Prometheus alerts

Elasticsearch / Loki
→ Stores and searches logs

Jaeger
→ Stores and analyzes traces
```

This separation is the foundation for building a clean, scalable observability platform.
