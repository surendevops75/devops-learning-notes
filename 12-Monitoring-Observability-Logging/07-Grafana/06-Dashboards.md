# Grafana Dashboards

## 1. Overview

A Grafana dashboard is a collection of visualizations that presents operational data in a form engineers can quickly understand.

A dashboard can contain:

```text
Panels
Variables
Queries
Annotations
Thresholds
Links
Transformations
Alerts
```

The purpose of a production dashboard is not simply to display as many metrics as possible.

A good dashboard should help an engineer answer:

```text
Is the system healthy?

What changed?

Where is the problem?

How severe is it?

Which service is affected?

What should I investigate next?
```

---

# 2. Dashboard Architecture

The basic flow is:

```text
User
  ↓
Grafana Dashboard
  ↓
Panel
  ↓
Query
  ↓
Data Source
  ↓
Backend
  ↓
Data
```

For our observability stack:

```text
                         Grafana
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
           Metrics         Logs         Traces
              ↓             ↓             ↓
         Prometheus    Loki/Elasticsearch  Jaeger
```

---

# 3. Dashboard Components

A typical dashboard contains:

```text
Dashboard
│
├── Variables
│
├── Rows / Sections
│
├── Panels
│   ├── Time Series
│   ├── Stat
│   ├── Gauge
│   ├── Table
│   └── Logs
│
├── Queries
│
├── Thresholds
│
└── Links
```

---

# 4. Dashboard Design Philosophy

A production dashboard should follow:

```text
High-level
    ↓
Service-level
    ↓
Component-level
    ↓
Detailed troubleshooting
```

For example:

```text
Kubernetes Overview
        ↓
Application Overview
        ↓
Payment Service
        ↓
Payment Pod
        ↓
Container / Request
```

---

# 5. Dashboard Types

In a real DevOps environment, maintain different dashboard categories.

```text
1. Executive / Business
2. Platform
3. Kubernetes
4. Infrastructure
5. Application
6. Service
7. Database
8. Incident / Troubleshooting
```

---

# 6. Executive Dashboard

An executive dashboard should be high-level.

Example:

```text
Application Availability
Error Rate
Request Volume
P95 Latency
Critical Services
Current Incidents
```

Avoid exposing hundreds of infrastructure metrics here.

---

# 7. Platform Dashboard

A platform dashboard can show:

```text
Cluster Health
Node Count
Node CPU
Node Memory
Pod Count
Pending Pods
Failed Pods
Restarting Pods
Deployment Availability
```

This is useful for the DevOps/platform team.

---

# 8. Kubernetes Dashboard

A Kubernetes overview dashboard:

```text
Kubernetes Cluster
│
├── Nodes
├── CPU
├── Memory
├── Pods
├── Namespaces
├── Deployments
├── ReplicaSets
├── Services
└── Persistent Volumes
```

---

# 9. Application Dashboard

For a microservices platform:

```text
Application
│
├── Request Rate
├── Error Rate
├── P50 Latency
├── P95 Latency
├── P99 Latency
├── Active Requests
├── Service Health
└── Dependency Health
```

This is much more useful during an application incident than a dashboard containing only infrastructure metrics.

---

# 10. Service Dashboard

For example, a Payment Service dashboard:

```text
Payment Service
│
├── Request Rate
├── Error Rate
├── P95 Latency
├── P99 Latency
├── CPU
├── Memory
├── Pod Restarts
├── Replica Availability
├── Database Errors
└── Dependency Errors
```

---

# 11. Database Dashboard

A database dashboard can contain:

```text
Connections
CPU
Memory
Storage
Read Operations
Write Operations
Latency
Errors
Connection Failures
Replication Health
```

The exact metrics depend on the database.

---

# 12. Incident Dashboard

During an incident, create a focused dashboard containing:

```text
Traffic
Errors
Latency
Saturation
Recent Deployments
Pod Restarts
Logs
Traces
```

The goal is rapid investigation.

---

# 13. Creating a Dashboard

Navigate to:

```text
Dashboards
    ↓
New
    ↓
New Dashboard
```

Then:

```text
Add visualization
```

Select a data source.

For metrics:

```text
Prometheus
```

For logs:

```text
Loki / Elasticsearch
```

For traces:

```text
Jaeger
```

---

# 14. Dashboard Panel

A panel consists of:

```text
Title
Query
Visualization
Units
Thresholds
Legend
Display settings
```

Example:

```text
Panel:
Payment Request Rate

Query:
rate(http_requests_total{service="payment"}[5m])

Visualization:
Time series

Unit:
requests/sec
```

---

# 15. Time Series Panel

Time series is one of the most useful visualizations.

Example:

```text
Requests/sec
│
│          /\
│     /\  /  \__
│____/  \/       \____
└────────────────────────→ Time
```

Use it for:

```text
Request rate
Latency
CPU
Memory
Errors
Network traffic
Database activity
```

---

# 16. Stat Panel

A Stat panel displays a single important value.

Example:

```text
┌────────────────────┐
│   Error Rate       │
│                    │
│      2.4%          │
└────────────────────┘
```

Good for:

```text
Availability
Current error rate
Pod count
Active users
Request rate
```

---

# 17. Gauge Panel

Gauge is useful when the value has a meaningful range.

Example:

```text
CPU Usage

     ┌─────────┐
     │   78%   │
     └─────────┘
```

Useful for:

```text
CPU
Memory
Disk
Capacity
Connection utilization
```

Avoid using gauges for every metric because they consume more visual space.

---

# 18. Table Panel

Tables are useful when engineers need detailed information.

Example:

```text
Pod                  CPU     Memory   Restarts
------------------------------------------------
payment-api-01       72%     610Mi    0
payment-api-02       68%     590Mi    1
payment-api-03       81%     700Mi    3
```

Useful during troubleshooting.

---

# 19. Logs Panel

A logs visualization can show:

```text
10:20:01 payment ERROR Database timeout
10:20:03 payment ERROR Connection refused
10:20:07 payment WARN Retry attempt
```

This is useful for connecting:

```text
Metric
  ↓
Logs
```

---

# 20. Dashboard Rows

Large dashboards should be organized into sections.

Example:

```text
Payment Service
│
├── Overview
│
├── Traffic
│
├── Errors
│
├── Latency
│
├── Resources
│
├── Logs
│
└── Traces
```

This makes dashboards easier to navigate.

---

# 21. Dashboard Variables

Variables allow one dashboard to work for multiple environments or services.

Example:

```text
Environment:
[ production ▼ ]

Namespace:
[ payments ▼ ]

Service:
[ payment-api ▼ ]

Pod:
[ payment-api-xxxxx ▼ ]
```

Without variables, you might create:

```text
Payment Dev Dashboard
Payment Stage Dashboard
Payment Production Dashboard
```

With variables:

```text
One Payment Dashboard
        ↓
Environment variable
```

---

# 22. Why Variables Matter

Variables reduce:

```text
Dashboard duplication
Maintenance
Configuration drift
```

They also improve incident investigation.

---

# 23. Environment Variable

Example:

```text
Environment
├── dev
├── staging
└── production
```

A PromQL query can then use:

```promql
{environment="$environment"}
```

The actual label must exist in your Prometheus metrics.

---

# 24. Namespace Variable

A Kubernetes dashboard can provide:

```text
Namespace
├── default
├── payments
├── orders
├── catalog
└── inventory
```

A query can use:

```promql
{namespace="$namespace"}
```

---

# 25. Pod Variable

A pod variable allows an engineer to inspect one pod.

Example:

```text
Pod:
[ payment-api-7d8f9cxxxxx ]
```

Then queries can filter:

```promql
{pod="$pod"}
```

---

# 26. Service Variable

For a microservices environment:

```text
Service
├── user
├── catalog
├── cart
├── order
├── payment
├── inventory
└── notification
```

This makes a common application dashboard reusable.

---

# 27. Variable Dependency

Variables can depend on each other.

Example:

```text
Cluster
   ↓
Namespace
   ↓
Service
   ↓
Pod
```

So:

```text
Cluster = production
        ↓
Namespace = payments
        ↓
Service = payment-api
        ↓
Pod = payment-api-xxxxx
```

This creates a dynamic troubleshooting dashboard.

---

# 28. Variable Query Example

A Prometheus variable can use a query such as:

```promql
label_values(kube_namespace_created, namespace)
```

The exact query depends on the metrics available in your environment.

---

# 29. Query Variables

Variables can also be populated using Prometheus label information.

Example concept:

```text
label_values(
  http_requests_total,
  service
)
```

This returns available service labels.

---

# 30. Multi-Select Variables

Allow users to select multiple services:

```text
Service:
☑ payment
☑ order
☐ catalog
```

The query can then use an appropriate regular-expression filter:

```promql
{service=~"$service"}
```

---

# 31. Include All Option

Variables can include:

```text
All
```

Example:

```text
Service:
[ All ▼ ]
```

The query then evaluates across all selected services.

Use this carefully because `All` can create very large queries.

---

# 32. Dashboard Time Range

Grafana dashboards support time ranges.

Examples:

```text
Last 5 minutes
Last 15 minutes
Last 1 hour
Last 6 hours
Last 24 hours
Last 7 days
```

For incident investigation:

```text
Last 30 minutes
```

is often a useful starting point.

---

# 33. Relative Time

A dashboard can use:

```text
now-1h
```

to:

```text
now
```

This makes dashboards dynamically follow the current time.

---

# 34. Custom Time Range

You can select:

```text
Start:
2026-08-11 09:00

End:
2026-08-11 12:00
```

This is useful when investigating a known incident window.

---

# 35. Dashboard Refresh

Common refresh intervals:

```text
5s
10s
30s
1m
5m
```

Do not use an aggressive refresh interval without considering backend load.

Example:

```text
Dashboard
 ↓
20 panels
 ↓
Each refreshes every 5 seconds
 ↓
Many queries
 ↓
Prometheus load increases
```

---

# 36. Dashboard Query Load

Suppose:

```text
20 panels
×
6 queries each
=
120 queries
```

If refreshed every 5 seconds:

```text
120 / 5
=
24 queries/sec
```

And this is only one dashboard/user.

With many users, the load can increase significantly.

---

# 37. Dashboard Performance

To improve performance:

```text
Reduce unnecessary panels
Reduce query complexity
Use appropriate time ranges
Avoid excessive refresh
Use recording rules where appropriate
Avoid extremely high-cardinality queries
```

---

# 38. Golden Signals Dashboard

A service dashboard should prominently show the four Golden Signals:

```text
Latency
Traffic
Errors
Saturation
```

Example:

```text
┌──────────────────────────────────────┐
│ Payment Service                      │
├──────────┬──────────┬────────┬───────┤
│ Traffic  │ Errors   │ P95    │ Satur.│
│ 450 rps  │ 1.2%     │ 240ms  │ 72%   │
└──────────┴──────────┴────────┴───────┘
```

---

# 39. Traffic Panel

Example:

```promql
sum(
  rate(http_requests_total{
    service="$service"
  }[5m])
)
```

This shows request rate if the application exposes the appropriate metric.

---

# 40. Error Rate Panel

Example:

```promql
sum(
  rate(http_requests_total{
    service="$service",
    status=~"5.."
  }[5m])
)
/
sum(
  rate(http_requests_total{
    service="$service"
  }[5m])
)
```

This calculates an approximate HTTP 5xx error ratio.

The label names must match your application metrics.

---

# 41. P95 Latency

For histogram-based metrics:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(
      http_request_duration_seconds_bucket{
        service="$service"
      }[5m]
    )
  )
)
```

This calculates P95 latency.

The metric name and labels depend on your application instrumentation.

---

# 42. Saturation Panel

Saturation could represent:

```text
CPU
Memory
Disk
Connection pools
Queue depth
Thread pools
```

For Kubernetes, you might show:

```text
CPU utilization
Memory utilization
Pod resource limits
```

---

# 43. Kubernetes Cluster Dashboard

A production cluster dashboard should provide:

```text
Cluster Overview
│
├── Node Count
├── Ready Nodes
├── CPU Usage
├── Memory Usage
├── Pod Count
├── Pending Pods
├── Failed Pods
├── Restarting Pods
├── Deployment Availability
└── PVC Usage
```

---

# 44. Node CPU Panel

Example:

```promql
100 *
(
  1 -
  avg by(instance) (
    rate(node_cpu_seconds_total{
      mode="idle"
    }[5m])
  )
)
```

This provides approximate CPU utilization per instance.

---

# 45. Node Memory Panel

Example:

```promql
100 *
(
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

This shows approximate memory utilization.

---

# 46. Pod Restart Panel

A useful query:

```promql
sum by(namespace, pod) (
  increase(kube_pod_container_status_restarts_total[1h])
)
```

This helps identify pods with recent restarts.

---

# 47. Pending Pods

Example:

```promql
sum(
  kube_pod_status_phase{
    phase="Pending"
  }
)
```

A sudden increase can indicate:

```text
Insufficient resources
Scheduling constraints
Node problems
PVC issues
Taints
Affinity problems
```

---

# 48. Deployment Availability

Example:

```promql
kube_deployment_status_replicas_available
```

Compare:

```text
Desired replicas
Available replicas
```

For example:

```text
Desired:   3
Available: 2
```

This indicates a degraded deployment.

---

# 49. Pod OOM Dashboard

Monitor:

```text
Memory usage
Memory limits
OOMKilled events
Restart count
```

A dashboard can help detect memory pressure before repeated crashes occur.

---

# 50. Application Dashboard

For your microservices platform, build one reusable service dashboard.

Example:

```text
Service Dashboard
│
├── Request Rate
├── Error Rate
├── P50
├── P95
├── P99
├── Active Pods
├── CPU
├── Memory
├── Restarts
├── Logs
└── Traces
```

Then use:

```text
$environment
$namespace
$service
```

variables.

---

# 51. Application Error Dashboard

Useful panels:

```text
5xx Rate
4xx Rate
Top Error Endpoints
Top Error Services
Error Logs
Recent Deployments
```

---

# 52. Top Error Services

A dashboard can show:

```text
Service          Error Rate
--------------------------------
payment          4.2%
orders           2.1%
catalog          0.8%
inventory        0.4%
```

This immediately tells engineers where to investigate.

---

# 53. Latency Dashboard

Display:

```text
P50
P90
P95
P99
```

Example:

```text
Latency
│
│          P99
│         /
│  P95   /
│   /\  /
│__/  \/________
└────────────────→
```

P99 spikes can reveal problems hidden by average latency.

---

# 54. Why Average Latency Is Not Enough

Suppose:

```text
99 requests = 100ms
1 request  = 10 seconds
```

Average latency may not adequately communicate the user experience.

Percentiles reveal tail latency.

Therefore production dashboards should often include:

```text
P50
P95
P99
```

---

# 55. Deployment Dashboard

A deployment dashboard can show:

```text
Deployment
│
├── Current Version
├── Desired Replicas
├── Available Replicas
├── Unavailable Replicas
├── Restart Count
├── CPU
├── Memory
└── Error Rate
```

---

# 56. Release Correlation

A very useful dashboard feature is showing deployments on the same timeline as application metrics.

Example:

```text
Error Rate
│
│                    /\
│                   /  \
│__________________/    \____
                  ↑
               Deployment
```

This helps answer:

```text
Did the problem begin after a deployment?
```

---

# 57. Annotations

Grafana annotations can mark events such as:

```text
Deployment
Rollback
Incident
Configuration change
Infrastructure change
```

Example:

```text
10:00 ──────── 10:30 ──────── 11:00
                  │
                  ↓
             Deployment
                  │
                  ↓
             Error rate ↑
```

---

# 58. Deployment Annotation

If your CI/CD system can send deployment events to Grafana, engineers can correlate:

```text
GitHub Actions
     ↓
Deployment
     ↓
Grafana Annotation
```

This is particularly useful in production troubleshooting.

---

# 59. GitHub Actions + Grafana

A practical flow:

```text
Developer
   ↓
GitHub
   ↓
GitHub Actions
   ↓
Build / Test / Security
   ↓
Deploy
   ↓
Kubernetes
   ↓
Grafana Annotation
```

The annotation can contain:

```text
Service
Version
Commit
Environment
Deployment time
```

---

# 60. Dashboard Links

Dashboards can link to:

```text
Related dashboard
Runbook
Git repository
Deployment
Logs
Traces
Incident system
```

Example:

```text
Payment Dashboard
   ↓
Payment Logs
Payment Traces
Payment Runbook
Payment Git Repository
```

This reduces investigation time.

---

# 61. Dashboard Links for Incident Response

A production dashboard should ideally provide:

```text
Runbook
Logs
Traces
Deployment History
Service Dashboard
```

Example:

```text
High Error Rate
     ↓
[View Logs]
     ↓
[View Traces]
     ↓
[View Deployment]
     ↓
[Open Runbook]
```

---

# 62. Drill-Down Dashboards

A dashboard can lead from broad information to detailed information.

```text
Cluster Dashboard
       ↓
Namespace Dashboard
       ↓
Service Dashboard
       ↓
Pod Dashboard
       ↓
Logs
       ↓
Trace
```

This creates a natural troubleshooting workflow.

---

# 63. Dashboard Navigation

Organize dashboards into folders:

```text
Dashboards
│
├── Kubernetes
│
├── Applications
│
├── Infrastructure
│
├── Databases
│
└── Troubleshooting
```

---

# 64. Dashboard Folders

For your environment:

```text
Kubernetes
├── Cluster Overview
├── Nodes
├── Pods
└── Workloads

Applications
├── Platform Overview
├── User
├── Catalog
├── Cart
├── Orders
├── Payment
├── Inventory
└── Notification
```

---

# 65. Dashboard Naming Convention

Use predictable names.

Good:

```text
Kubernetes - Cluster Overview
Kubernetes - Node Health
Application - Platform Overview
Application - Payment
Application - Orders
```

Avoid:

```text
Dashboard1
Test
New Dashboard
Copy of Dashboard
```

---

# 66. Dashboard Folder Permissions

Use folder-level access controls where appropriate.

Example:

```text
Platform Dashboards
      ↓
Platform Team

Application Dashboards
      ↓
Application Teams
```

Production access should follow least privilege.

---

# 67. Dashboard as Code

Manual dashboards can become difficult to manage.

Instead:

```text
Dashboard JSON
      ↓
Git
      ↓
Review
      ↓
ArgoCD
      ↓
Grafana
```

This gives:

```text
Version control
Review
Auditability
Rollback
Consistency
```

---

# 68. Dashboard Repository

A practical structure:

```text
grafana/
│
├── dashboards/
│   │
│   ├── kubernetes/
│   │   ├── cluster.json
│   │   ├── nodes.json
│   │   └── pods.json
│   │
│   ├── applications/
│   │   ├── overview.json
│   │   ├── payment.json
│   │   ├── orders.json
│   │   └── catalog.json
│   │
│   └── infrastructure/
│       ├── rds.json
│       └── alb.json
│
└── provisioning/
    └── dashboards/
        └── providers.yaml
```

---

# 69. Dashboard Provisioning

Grafana can automatically load dashboard definitions from files.

Conceptually:

```yaml
apiVersion: 1

providers:
  - name: Kubernetes
    type: file
    folder: Kubernetes
    options:
      path: /var/lib/grafana/dashboards/kubernetes
```

---

# 70. Dashboard Lifecycle

A production dashboard should follow:

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
Monitor
  ↓
Improve
```

Avoid unmanaged dashboards that only exist inside the production UI.

---

# 71. Dashboard Version Control

Git provides:

```text
Commit history
Pull requests
Review
Rollback
Ownership
```

Example:

```text
Commit:
Improve Payment P95 panel

PR:
Reviewed by Platform Team

Merge:
main

ArgoCD:
Deploy

Grafana:
Updated dashboard
```

---

# 72. Dashboard Rollback

If a dashboard change is incorrect:

```text
Bad dashboard commit
        ↓
Git rollback
        ↓
ArgoCD
        ↓
Previous dashboard
```

This is much safer than manually reconstructing a previous dashboard.

---

# 73. Dashboard Provisioning and Deletion

Be careful with automated dashboard provisioning.

A dashboard removed from Git may:

```text
Remain in Grafana
```

or:

```text
Be deleted automatically
```

depending on provisioning settings.

Understand the behavior before enabling automatic deletion.

---

# 74. Dashboard Design: Avoid Noise

Bad dashboard:

```text
50 panels
20 graphs
15 gauges
10 tables
No hierarchy
No ownership
```

Good dashboard:

```text
Health
 ↓
Traffic
 ↓
Errors
 ↓
Latency
 ↓
Saturation
 ↓
Details
```

---

# 75. Dashboard Design: Focus on Decisions

Every panel should answer a question.

Examples:

```text
Is traffic increasing?
Is error rate increasing?
Is latency increasing?
Are pods restarting?
Are nodes saturated?
Did deployment cause the problem?
```

If a panel does not help answer an operational question, consider removing it.

---

# 76. Dashboard Color Semantics

Use consistent semantics.

For example:

```text
Healthy
Warning
Critical
```

Avoid using many colors simply for visual decoration.

The meaning of a color should remain consistent across dashboards.

---

# 77. Thresholds

Thresholds can make important values easier to identify.

Example:

```text
CPU

0-70%    → Normal
70-85%   → Warning
>85%     → Critical
```

These values are examples.

Actual thresholds should be based on workload behavior and operational requirements.

---

# 78. Error Rate Threshold

Example:

```text
Error Rate

<1%       Normal
1-5%      Warning
>5%       Critical
```

Do not blindly use these values for production alerting.

The correct threshold depends on the service's SLO.

---

# 79. SLO Dashboard

A production service dashboard can include:

```text
Availability SLO
Latency SLO
Error Budget
Current Error Rate
Burn Rate
```

Example:

```text
SLO:
99.9%

Current:
99.97%

Error Budget:
72% remaining
```

---

# 80. Error Budget

If a service has a 99.9% availability target, the allowed unavailability is approximately:

```text
0.1%
```

A dashboard can show:

```text
SLO
Error Budget
Budget Burn
```

This connects observability with reliability objectives.

---

# 81. Dashboard for Kubernetes Troubleshooting

A troubleshooting dashboard can show:

```text
Node CPU
Node Memory
Pod Restarts
OOMKilled
Pending Pods
Container CPU
Container Memory
Deployment Availability
Events
```

Then the engineer can investigate:

```text
Cluster
 ↓
Node
 ↓
Pod
 ↓
Container
```

---

# 82. Dashboard for 503 Errors

If users receive 503 errors, build a dashboard containing:

```text
ALB Request Count
ALB 5xx
Ingress Errors
Service Request Rate
Service 5xx
Pod Availability
Pod Restarts
Readiness Failures
CPU
Memory
```

This allows investigation across layers.

---

# 83. 503 Troubleshooting Flow

```text
User
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
 ↓
Application
```

Dashboard should help determine:

```text
Which layer is returning 503?
```

---

# 84. Dashboard for High Latency

Show:

```text
P50
P95
P99
Request Rate
Error Rate
CPU
Memory
Database Latency
External Dependency Latency
Trace Duration
```

Then:

```text
Metric
 ↓
Trace
 ↓
Slow Span
 ↓
Dependency
```

---

# 85. Dashboard for OOMKilled

Show:

```text
Container Memory
Memory Limit
Memory Working Set
OOMKilled Count
Pod Restarts
Deployment Availability
```

This helps determine whether the problem is:

```text
Application memory leak
Insufficient limit
Traffic increase
Configuration problem
```

---

# 86. Dashboard for Node Pressure

Show:

```text
Node CPU
Node Memory
Memory Available
Disk Usage
Disk Pressure
PID Pressure
Pod Count
Evictions
```

This helps identify node-level resource exhaustion.

---

# 87. Dashboard for Deployment Failures

Show:

```text
Desired replicas
Available replicas
Unavailable replicas
Pod phase
Pod restarts
Image pull errors
Readiness failures
Application errors
Deployment timestamp
```

---

# 88. Logs in a Dashboard

A useful application dashboard can include a log panel:

```text
┌────────────────────────────────────────────┐
│ Payment Service Logs                       │
├────────────────────────────────────────────┤
│ 10:21 ERROR DB timeout                     │
│ 10:22 ERROR DB connection refused          │
│ 10:23 WARN  Retry attempt                  │
└────────────────────────────────────────────┘
```

The logs should be filtered using dashboard variables.

---

# 89. Traces in a Dashboard

A dashboard can provide links to traces or trace exploration.

Example:

```text
High P95 Latency
       ↓
Payment Service
       ↓
View traces
       ↓
Jaeger
```

This turns a dashboard into an investigation entry point.

---

# 90. Metrics → Logs → Traces Workflow

A mature dashboard supports:

```text
1. Detect
   ↓
2. Metrics
   ↓
3. Logs
   ↓
4. Traces
   ↓
5. Root Cause
```

Example:

```text
P95 latency ↑
     ↓
Payment error logs
     ↓
trace_id
     ↓
Jaeger
     ↓
Database span slow
```

---

# 91. Dashboard Annotations

Useful annotations include:

```text
Deployment
Rollback
Infrastructure change
Configuration change
Incident
Maintenance
```

This provides temporal context.

---

# 92. Deployment Annotation Example

```text
10:00
Version v1.4.2 deployed
       │
       ↓
10:08
Error rate increases
       │
       ↓
10:10
Rollback
```

A dashboard makes this relationship immediately visible.

---

# 93. Dashboard Reuse

Instead of creating separate dashboards:

```text
Payment Dev
Payment Stage
Payment Prod
```

create:

```text
Payment Service Dashboard
```

with:

```text
Environment
Namespace
Service
Pod
```

variables.

---

# 94. Dashboard Standardization

Every service dashboard should follow a common layout:

```text
Row 1:
Service Health

Row 2:
Traffic

Row 3:
Errors

Row 4:
Latency

Row 5:
Resources

Row 6:
Dependencies

Row 7:
Logs

Row 8:
Traces
```

This makes engineers faster because every dashboard follows the same mental model.

---

# 95. Recommended Service Dashboard Layout

```text
┌─────────────────────────────────────────────┐
│ SERVICE OVERVIEW                            │
│ Status | Traffic | Errors | P95 | P99       │
├─────────────────────────────────────────────┤
│ TRAFFIC                                     │
│ Requests/sec                                │
├─────────────────────────────────────────────┤
│ ERRORS                                      │
│ 4xx / 5xx                                   │
├─────────────────────────────────────────────┤
│ LATENCY                                     │
│ P50 / P95 / P99                             │
├─────────────────────────────────────────────┤
│ RESOURCES                                   │
│ CPU / Memory / Restarts                     │
├─────────────────────────────────────────────┤
│ DEPENDENCIES                                │
│ DB / External APIs / Queues                 │
├─────────────────────────────────────────────┤
│ LOGS                                        │
├─────────────────────────────────────────────┤
│ TRACES                                      │
└─────────────────────────────────────────────┘
```

---

# 96. Production Dashboard Repository

For a real project:

```text
grafana/
│
├── dashboards/
│   ├── kubernetes/
│   │   ├── cluster-overview.json
│   │   ├── node-health.json
│   │   └── pod-health.json
│   │
│   ├── applications/
│   │   ├── platform-overview.json
│   │   ├── payment.json
│   │   ├── orders.json
│   │   ├── catalog.json
│   │   └── inventory.json
│   │
│   └── infrastructure/
│       ├── alb.json
│       └── rds.json
│
├── provisioning/
│   └── dashboards/
│       └── providers.yaml
│
└── argocd/
    └── application.yaml
```

---

# 97. GitOps Dashboard Flow

```text
Engineer
   ↓
Modify Dashboard
   ↓
Git Commit
   ↓
Pull Request
   ↓
Code Review
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Grafana
```

---

# 98. Dashboard Change Example

Suppose the Payment dashboard currently shows:

```text
P50
P95
```

You want to add:

```text
P99
```

Workflow:

```text
Edit dashboard JSON
       ↓
Git commit
       ↓
Pull request
       ↓
Review
       ↓
Merge
       ↓
ArgoCD sync
       ↓
Grafana updated
```

---

# 99. Dashboard Testing

Before production:

```text
Query works
Data source works
Variables work
Time range works
Panels load
Links work
Thresholds make sense
No unnecessary expensive queries
```

Test with:

```text
Development
       ↓
Staging
       ↓
Production
```

---

# 100. Dashboard Performance Testing

Measure:

```text
Dashboard load time
Query execution time
Prometheus query cost
Number of queries
Refresh frequency
Concurrent users
```

A dashboard that works for one engineer may perform poorly for 100 engineers.

---

# 101. Dashboard Cardinality Consideration

High-cardinality labels can create expensive queries.

Example:

```text
user_id
request_id
trace_id
```

Using these dimensions indiscriminately in Prometheus metrics can create huge time-series counts.

Dashboards should avoid queries that scan unnecessary high-cardinality data.

---

# 102. Dashboard Security

Do not expose sensitive information unnecessarily.

Examples:

```text
Customer identifiers
Internal IP addresses
Secrets
Tokens
Personal data
Sensitive logs
```

Dashboards should follow the organization's security and data-access requirements.

---

# 103. Dashboard Access Control

Example:

```text
Platform Team
   ↓
Kubernetes + Infrastructure

Application Team
   ↓
Application dashboards

Developers
   ↓
Development dashboards
```

Use folders, teams, organizations, or appropriate RBAC features depending on your Grafana deployment.

---

# 104. Dashboard Ownership

Every important dashboard should have:

```text
Owner
Purpose
Data sources
Runbook
Environment
Last reviewed date
```

Example:

```text
Dashboard:
Payment Service

Owner:
Payments Team

Purpose:
Monitor payment API health

Data:
Prometheus + Loki + Jaeger

Runbook:
Payment incident runbook
```

---

# 105. Dashboard Documentation

Document:

```text
What does this dashboard show?
What does each panel mean?
What thresholds are important?
What should an engineer do when a panel is abnormal?
```

This is particularly useful for on-call engineers.

---

# 106. Dashboard Anti-Patterns

Avoid:

```text
Too many panels
No variables
No ownership
No documentation
Hardcoded environments
Very aggressive refresh
Expensive queries
Duplicate dashboards
Manual production-only dashboards
No rollback strategy
```

---

# 107. Good vs Bad Dashboard

Bad:

```text
100 panels
Random metrics
No hierarchy
No variables
No owner
No links
```

Good:

```text
Service health
    ↓
Golden signals
    ↓
Resources
    ↓
Dependencies
    ↓
Logs
    ↓
Traces
    ↓
Runbook
```

---

# 108. Production Dashboard Strategy

For your environment, use:

```text
1. Platform Overview
2. Kubernetes Cluster
3. Node Health
4. Pod Health
5. Application Overview
6. Service Dashboards
7. Infrastructure Dashboards
8. Incident Dashboard
```

---

# 109. Platform Overview

Example:

```text
Platform Overview
│
├── Cluster Health
├── Node Health
├── Pod Health
├── Application Availability
├── Global Error Rate
├── Global P95
└── Active Incidents
```

This should be the first dashboard an on-call engineer opens.

---

# 110. Application Overview

For your microservices platform:

```text
Application Overview
│
├── Request Rate
├── Error Rate
├── P95
├── P99
├── Service Availability
│
├── User
├── Catalog
├── Cart
├── Orders
├── Payment
├── Inventory
└── Notification
```

---

# 111. Service Dashboard

Each service follows the same layout:

```text
Service Overview
        ↓
Traffic
        ↓
Errors
        ↓
Latency
        ↓
Saturation
        ↓
Dependencies
        ↓
Logs
        ↓
Traces
```

This standardization is extremely valuable in production.

---

# 112. Example Payment Investigation

Engineer sees:

```text
Payment Error Rate = 8%
```

They open:

```text
Payment Dashboard
```

Then:

```text
P95 = 2.3 seconds
```

Next:

```text
Logs
 ↓
Database timeout
```

Then:

```text
Trace
 ↓
Database span = 1.9 seconds
```

Then:

```text
Database dashboard
 ↓
Connection utilization = 98%
```

The dashboard chain helps identify the likely root cause.

---

# 113. Dashboard and Incident Response

During an incident:

```text
Alert
 ↓
Grafana
 ↓
Service Dashboard
 ↓
Metrics
 ↓
Logs
 ↓
Traces
 ↓
Root Cause
 ↓
Remediation
```

The dashboard should reduce the number of systems an engineer must manually search.

---

# 114. Dashboard and Runbooks

Add a link:

```text
Payment Service Dashboard
       ↓
Payment Incident Runbook
```

The runbook can explain:

```text
Common errors
Rollback procedure
Dependency checks
Database checks
Escalation path
Recovery procedure
```

---

# 115. Dashboard and GitOps

Recommended:

```text
Dashboard JSON
       ↓
Git
       ↓
Pull Request
       ↓
Review
       ↓
ArgoCD
       ↓
Grafana
```

This makes dashboards part of the engineering lifecycle instead of unmanaged UI configuration.

---

# 116. Dashboard Production Checklist

```text
[ ] Clear purpose
[ ] Consistent naming
[ ] Folder organization
[ ] Variables
[ ] Appropriate time range
[ ] Appropriate refresh
[ ] Golden signals
[ ] Resource metrics
[ ] Dependency metrics
[ ] Logs
[ ] Traces
[ ] Deployment annotations
[ ] Dashboard links
[ ] Runbook links
[ ] Ownership
[ ] Access control
[ ] Git version control
[ ] Provisioning
[ ] Performance tested
[ ] Security reviewed
```

---

# 117. Interview Answer: How Do You Design a Grafana Dashboard?

```text
"I start with the operational questions the dashboard needs to
answer.

For a service dashboard I normally start with the Golden Signals:
traffic, errors, latency and saturation.

Then I add resource metrics such as CPU, memory and pod health,
followed by dependency metrics.

For troubleshooting, I add links or panels for logs and traces.

I also use variables for environment, namespace, service and pod so
the dashboard can be reused instead of creating separate dashboards
for every service."
```

---

# 118. Interview Answer: How Do You Manage Dashboards in Production?

```text
"I don't want production dashboards to exist only as manual UI
changes.

I manage important dashboards as code.

The dashboard definitions are stored in Git, reviewed through pull
requests and provisioned into Grafana.

ArgoCD deploys the configuration to EKS.

This gives us version control, auditability, consistency and
rollback."
```

---

# 119. Interview Answer: How Do You Avoid Grafana Dashboard Performance Problems?

```text
"I look at the number and complexity of queries, refresh intervals,
time ranges and backend query performance.

I avoid unnecessary panels and expensive high-cardinality queries.

For Prometheus, I optimize PromQL and use recording rules where
appropriate.

I also avoid very aggressive refresh intervals on large dashboards.

The goal is to make dashboards useful without putting unnecessary
load on the observability backend."
```

---

# 120. Interview Answer: What Dashboards Would You Create for Kubernetes?

```text
"I would create a cluster overview, node health, workload or pod
health, and application/service dashboards.

The cluster dashboard would cover node readiness, CPU, memory,
pod counts and pending or failed pods.

The workload dashboard would cover deployment availability,
restarts, resource utilization and container health.

For applications, I would use the Golden Signals and then provide
logs and traces for deeper troubleshooting."
```

---

# 121. Interview Answer: How Do You Correlate Metrics, Logs and Traces?

```text
"I start with metrics to detect the problem.

For example, if Payment Service P95 latency increases, I move to the
corresponding logs to understand the errors.

If the logs contain trace IDs, I use those to inspect the distributed
trace in Jaeger.

The trace can then show which downstream service or database call
is causing the latency.

So the workflow becomes:

Metrics → Logs → Traces → Root Cause."
```

---

# 122. Final Dashboard Architecture

```text
                              Grafana
                                 │
       ┌─────────────────────────┼─────────────────────────┐
       ↓                         ↓                         ↓
 Kubernetes Dashboard     Application Dashboard     Infrastructure
       │                         │                         │
       ↓                         ↓                         ↓
 Prometheus Metrics       Golden Signals            CPU / Memory
       │                         │                   ALB / RDS
       │                         │
       └──────────────┬──────────┘
                      ↓
                 Logs / Traces
                      │
               ┌──────┴──────┐
               ↓             ↓
           Loki / ES       Jaeger
```

---

# 123. Complete Production Dashboard Flow

```text
                         Engineer
                            │
                            ↓
                         Grafana
                            │
                            ↓
                  Platform Overview
                            │
                 ┌──────────┼──────────┐
                 ↓          ↓          ↓
              Metrics      Logs      Traces
                 │          │          │
                 ↓          ↓          ↓
            Prometheus    Loki/ES    Jaeger
                 │          │          │
                 └──────────┼──────────┘
                            ↓
                       Investigation
                            │
                            ↓
                         Root Cause
                            │
                            ↓
                         Runbook
                            │
                            ↓
                        Remediation
```

---

# 124. Final Mental Model

Remember this hierarchy:

```text
Dashboard
   ↓
Panel
   ↓
Query
   ↓
Data Source
   ↓
Backend
```

And for production troubleshooting:

```text
Overview
   ↓
Golden Signals
   ↓
Resources
   ↓
Dependencies
   ↓
Logs
   ↓
Traces
   ↓
Root Cause
```

The best Grafana dashboard is not the one with the most graphs.

It is the one that helps an engineer **detect a problem quickly, understand its impact, drill down to the affected component, correlate metrics with logs and traces, and reach the root cause with minimal manual investigation**.
