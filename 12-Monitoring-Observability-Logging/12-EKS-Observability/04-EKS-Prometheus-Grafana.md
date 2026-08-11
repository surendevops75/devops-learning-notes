# EKS Prometheus & Grafana

## 1. Overview

Prometheus and Grafana provide a powerful monitoring and visualization stack for Amazon EKS.

A typical architecture is:

```text
EKS Cluster
    │
    ├── Nodes
    ├── Pods
    ├── Kubernetes Objects
    └── Applications
            │
            ↓
       Metrics Sources
            │
       ┌────┴─────┐
       ↓          ↓
Node Exporter  kube-state-metrics
       │          │
       └────┬─────┘
            ↓
        Prometheus
            │
            ↓
         Grafana
            │
      ┌─────┴─────┐
      ↓           ↓
 Dashboards     Alerts
```

Prometheus is primarily responsible for **collecting and storing metrics**, while Grafana is used for **visualization, dashboards, and alert presentation**.

---

# 2. Why Prometheus and Grafana for EKS?

EKS environments contain multiple layers:

```text
AWS Infrastructure
        ↓
EKS Cluster
        ↓
Nodes
        ↓
Pods
        ↓
Containers
        ↓
Applications
```

Prometheus can collect metrics from these layers, while Grafana provides a unified view.

The combination helps answer:

```text
Are Nodes healthy?
Are Pods healthy?
Is the cluster running out of capacity?
Which application is consuming resources?
Are workloads scaling correctly?
Is application latency increasing?
```

---

# 3. Prometheus Architecture in EKS

A typical architecture:

```text
                         EKS
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
   Node Exporter    kube-state-metrics   Applications
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                     Prometheus
                          │
                          ↓
                       Grafana
```

Prometheus can scrape:

```text
Node metrics
Kubernetes object metrics
Application metrics
Kubelet metrics
Custom metrics
```

---

# 4. Grafana Architecture

Grafana connects to Prometheus as a data source.

```text
Prometheus
    │
    │ PromQL
    ↓
 Grafana
    │
    ├── Dashboards
    ├── Panels
    ├── Variables
    └── Alerts
```

Grafana does not need to collect the metrics itself.

Instead:

```text
Prometheus → Stores metrics
Grafana    → Queries and visualizes metrics
```

---

# 5. Metrics Sources in EKS

Important sources include:

```text
EKS Metrics
│
├── Node Exporter
├── kube-state-metrics
├── kubelet
├── cAdvisor/container metrics
├── Application metrics
└── Kubernetes control-plane metrics
```

Different sources provide different information.

---

# 6. Node Exporter

Node Exporter provides host-level metrics.

Examples:

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
EKS Node
   │
   ↓
Node Exporter
   │
   ↓
Prometheus
```

It is commonly deployed as a DaemonSet.

---

# 7. Why Node Exporter Uses a DaemonSet

Suppose the cluster has:

```text
Node-1
Node-2
Node-3
```

A DaemonSet creates:

```text
Node-1 → Node Exporter
Node-2 → Node Exporter
Node-3 → Node Exporter
```

When another Node is added:

```text
Node-4
   ↓
DaemonSet
   ↓
Node Exporter automatically scheduled
```

This makes Node monitoring scale with the cluster.

---

# 8. kube-state-metrics

kube-state-metrics exposes metrics about Kubernetes object state.

Examples:

```text
Deployment replicas
Pod status
DaemonSet status
StatefulSet status
Job status
Node status
PersistentVolumeClaim status
```

Architecture:

```text
Kubernetes API
      ↓
kube-state-metrics
      ↓
Prometheus
```

---

# 9. Node Exporter vs kube-state-metrics

They solve different problems.

| Component          | Primary Purpose                   |
| ------------------ | --------------------------------- |
| Node Exporter      | Linux/Node infrastructure metrics |
| kube-state-metrics | Kubernetes object state           |
| Prometheus         | Collection and storage            |
| Grafana            | Visualization                     |

Example:

```text
Node Exporter
→ Node CPU = 80%

kube-state-metrics
→ Deployment desired replicas = 5
→ Available replicas = 3
```

---

# 10. Application Metrics

Applications can expose metrics such as:

```text
HTTP request rate
HTTP errors
Request latency
Database connections
Queue depth
Business metrics
```

A common endpoint is:

```text
/metrics
```

Prometheus scrapes the endpoint.

```text
Application
     │
     ↓
 /metrics
     │
     ↓
Prometheus
```

---

# 11. Prometheus Scraping

Prometheus works primarily through a pull-based model.

Conceptually:

```text
Prometheus
    │
    │ HTTP GET
    ↓
Target /metrics
    │
    ↓
Metrics response
```

Example:

```text
http_requests_total 1500
```

Prometheus stores the resulting time series.

---

# 12. Prometheus Targets

Targets can include:

```text
Nodes
Pods
Services
Applications
Exporters
Kubernetes components
```

Prometheus discovers targets using configuration and Kubernetes service discovery mechanisms.

---

# 13. Kubernetes Service Discovery

In EKS, Prometheus can discover Kubernetes resources dynamically.

Conceptually:

```text
Kubernetes API
      ↓
Prometheus Discovery
      ↓
Targets
      ↓
Scraping
```

This is important because Pods are dynamic.

A Pod can:

```text
Start
Stop
Move
Scale
Be replaced
```

Prometheus must discover the current targets automatically.

---

# 14. Prometheus Labels

Prometheus metrics use labels to identify dimensions.

Example:

```text
http_requests_total{
  namespace="production",
  pod="payment-abc",
  method="GET",
  status="200"
}
```

Labels allow filtering and aggregation.

For example:

```text
namespace="production"
```

can isolate production metrics.

---

# 15. Labels in EKS

Useful labels include:

```text
cluster
namespace
pod
container
node
service
deployment
environment
```

These labels make dashboards and queries more useful.

---

# 16. PromQL

PromQL is Prometheus Query Language.

Example:

```promql
up
```

shows whether targets are available.

CPU example:

```promql
rate(node_cpu_seconds_total[5m])
```

Memory example:

```promql
node_memory_MemAvailable_bytes
```

PromQL allows filtering, aggregation, mathematical operations, and time-window calculations.

---

# 17. CPU Query Example

A simplified CPU utilization query:

```promql
100 *
(1 - avg by(instance)
(rate(node_cpu_seconds_total{mode="idle"}[5m])))
```

This calculates approximate CPU utilization by subtracting idle CPU from total CPU.

---

# 18. Memory Query Example

Available memory:

```promql
node_memory_MemAvailable_bytes
```

Total memory:

```promql
node_memory_MemTotal_bytes
```

A percentage calculation can be built using:

```promql
100 *
(
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

---

# 19. Pod CPU Monitoring

Pod CPU usage can be queried from container metrics.

Conceptually:

```text
Prometheus
   ↓
Container metrics
   ↓
Pod CPU
```

Group by:

```text
namespace
pod
container
```

This helps identify resource-intensive workloads.

---

# 20. Pod Memory Monitoring

Monitor:

```text
Current memory
Requests
Limits
Memory growth
OOM events
```

A useful dashboard can show:

```text
Namespace
Pod
Current Memory
Memory Limit
Utilization
```

---

# 21. Node Monitoring Dashboard

A useful Node dashboard includes:

```text
Node
│
├── CPU utilization
├── Memory utilization
├── Disk usage
├── Disk I/O
├── Network traffic
├── Load
├── Filesystem
└── Node status
```

This provides infrastructure-level visibility.

---

# 22. Cluster Monitoring Dashboard

A cluster overview can show:

```text
Cluster
│
├── Total Nodes
├── Ready Nodes
├── Total Pods
├── Running Pods
├── Pending Pods
├── Failed Pods
├── CPU usage
├── Memory usage
├── Storage
└── Network
```

---

# 23. Workload Monitoring Dashboard

For Deployments:

```text
Deployment
│
├── Desired replicas
├── Current replicas
├── Available replicas
├── Ready replicas
├── CPU
├── Memory
├── Restarts
└── Error rate
```

This helps detect application availability problems.

---

# 24. Pod Restart Monitoring

A high restart count is an important signal.

Possible causes:

```text
Application crash
OOMKilled
Failed liveness probe
Configuration problem
Dependency failure
```

A dashboard can show:

```text
Pod
Restart Count
Last Restart
Namespace
Container
```

---

# 25. Pending Pod Monitoring

Pending Pods should be monitored.

Possible causes:

```text
Insufficient CPU
Insufficient memory
Taints
Affinity
Topology constraints
Storage
Pod IP exhaustion
Node limits
```

A useful alert:

```text
Production Pending Pods > 0
```

with an appropriate duration.

---

# 26. Deployment Availability

Monitor:

```text
Desired replicas
Available replicas
Unavailable replicas
```

Example:

```text
Desired = 5
Available = 5
```

Healthy.

Problem:

```text
Desired = 5
Available = 3
```

The application may have reduced availability.

---

# 27. HPA Monitoring

Monitor:

```text
Current replicas
Desired replicas
Min replicas
Max replicas
CPU utilization
Memory utilization
Scaling events
```

Example:

```text
Current = 4
Desired = 8
Max = 10
```

If desired replicas remain high but Pods remain Pending, cluster capacity may be insufficient.

---

# 28. Cluster Autoscaler Monitoring

Monitor:

```text
Node count
Desired Node count
Pending Pods
Scale-up events
Scale-down events
Node provisioning time
```

A useful relationship is:

```text
Traffic ↑
 ↓
HPA ↑
 ↓
Pods ↑
 ↓
Capacity insufficient
 ↓
Cluster Autoscaler
 ↓
Nodes ↑
```

---

# 29. EKS Network Monitoring

Prometheus dashboards can monitor relevant network metrics such as:

```text
Network traffic
Packet errors
Packet drops
Connections
Pod network activity
```

For EKS, also consider:

```text
AWS VPC CNI
ENIs
Pod IP availability
Subnet capacity
```

---

# 30. CoreDNS Monitoring

CoreDNS is critical for Kubernetes service discovery.

Monitor:

```text
CPU
Memory
Request rate
Errors
Latency
Restarts
Pod availability
```

Example:

```text
Application
    ↓
Service name
    ↓
CoreDNS
    ↓
Service IP
```

DNS problems can affect many applications simultaneously.

---

# 31. Prometheus Data Model

Prometheus stores metrics as time series.

A time series is identified by:

```text
Metric name + labels
```

Example:

```text
http_requests_total{
  service="payment",
  status="500"
}
```

The value changes over time.

---

# 32. Counter Metrics

Counters generally increase over time.

Example:

```text
http_requests_total
```

Use `rate()` to calculate the rate of increase:

```promql
rate(http_requests_total[5m])
```

This can show requests per second.

---

# 33. Gauge Metrics

Gauges represent values that can increase or decrease.

Examples:

```text
CPU utilization
Memory usage
Active connections
Queue depth
```

A gauge can move in either direction.

---

# 34. Histogram Metrics

Histograms are useful for measuring distributions such as:

```text
Request latency
Response size
Processing duration
```

They can help calculate:

```text
p50
p95
p99
```

latency values.

---

# 35. Application Latency

A production dashboard should monitor:

```text
p50
p95
p99
```

Example:

```text
p50 = 100 ms
p95 = 400 ms
p99 = 800 ms
```

If p99 suddenly increases, a subset of requests may be experiencing severe latency.

---

# 36. Error Rate

A common application SLI is error rate.

Conceptually:

```text
Error Rate =
Errors / Total Requests
```

Example:

```text
Errors = 50
Requests = 10,000
```

Error rate:

```text
0.5%
```

Monitor this alongside latency and traffic.

---

# 37. Request Rate

Request rate measures incoming traffic.

Example:

```text
1,000 requests/sec
```

A sudden increase may cause:

```text
CPU ↑
Memory ↑
Pod count ↑
Node count ↑
Latency ↑
```

Traffic should therefore be correlated with resource usage.

---

# 38. Golden Signals

A useful application monitoring model is the four Golden Signals:

```text
Latency
Traffic
Errors
Saturation
```

In EKS:

```text
Traffic
→ Requests/sec

Latency
→ p95/p99

Errors
→ HTTP 5xx

Saturation
→ CPU / Memory / Capacity
```

---

# 39. Grafana Dashboard Design

Avoid creating one huge dashboard containing everything.

Instead use layers:

```text
Grafana
│
├── EKS Overview
├── Cluster
├── Nodes
├── Pods
├── Workloads
├── Networking
├── Storage
├── Applications
└── Autoscaling
```

This makes troubleshooting easier.

---

# 40. EKS Overview Dashboard

A high-level dashboard should answer:

```text
Is the cluster healthy?
```

Example:

```text
Nodes Ready          12
Pending Pods          0
Failed Pods           0
CPU                  58%
Memory               63%
Disk                 51%
Alerts                 0
```

---

# 41. Node Dashboard

A Node dashboard should allow selection of:

```text
Node
Availability Zone
Node Group
Instance Type
```

Panels can include:

```text
CPU
Memory
Disk
Network
Filesystem
Load
```

---

# 42. Pod Dashboard

Useful filters:

```text
Namespace
Deployment
Pod
Container
Node
```

Panels:

```text
CPU
Memory
Restarts
Network
Status
```

---

# 43. Namespace Dashboard

A namespace-level dashboard can show:

```text
Pod count
CPU requests
CPU usage
Memory requests
Memory usage
Restarts
Errors
Network
```

This is useful for separating:

```text
production
staging
development
```

---

# 44. Application Dashboard

For a microservice:

```text
Payment Service
│
├── Request rate
├── Error rate
├── p95 latency
├── p99 latency
├── Active requests
├── CPU
├── Memory
├── Pod count
└── Restarts
```

This connects application health with infrastructure resources.

---

# 45. Grafana Variables

Grafana variables allow dynamic dashboards.

Example:

```text
Cluster: production
Namespace: payments
Pod: payment-abc
```

One dashboard can then be reused for multiple environments.

Typical variables:

```text
cluster
namespace
node
pod
deployment
container
```

---

# 46. Dashboard Drill-Down

A good dashboard supports:

```text
Cluster
 ↓
Node
 ↓
Namespace
 ↓
Deployment
 ↓
Pod
 ↓
Container
```

Example:

```text
Cluster CPU high
      ↓
Node-3 CPU high
      ↓
payment namespace
      ↓
payment deployment
      ↓
payment-abc
      ↓
container CPU high
```

This makes troubleshooting much faster.

---

# 47. Grafana Alerting

Grafana can visualize Prometheus data and can also provide alerting capabilities.

Examples:

```text
Node NotReady
High CPU
High memory
High disk
Pending Pods
Deployment unavailable
High error rate
High latency
```

Alerts should be actionable.

---

# 48. Prometheus Alert Rules

Prometheus can evaluate alerting rules.

Conceptually:

```text
Metric
 ↓
PromQL condition
 ↓
Threshold
 ↓
Alert
```

Example:

```text
CPU utilization > 90%
for 10 minutes
```

This reduces noise from short-lived spikes.

---

# 49. Alert Severity

Use meaningful severity levels.

Example:

```text
warning
critical
```

For example:

```text
warning:
CPU > 80%

critical:
CPU > 95% for 10 minutes
```

The exact thresholds should be based on workload behavior.

---

# 50. Alert Examples

### Node NotReady

```text
Node status = NotReady
```

### Pending Pods

```text
Production Pending Pods > 0
```

### High Memory

```text
Memory utilization > 90%
```

### Deployment Failure

```text
Available replicas < desired replicas
```

### Application Error Rate

```text
5xx rate > defined threshold
```

---

# 51. Alert Fatigue

Avoid alerts for every small metric fluctuation.

Bad:

```text
CPU > 70%
```

for a few seconds.

Better:

```text
Production Node CPU > 90%
for 10 minutes
```

when that condition requires action.

---

# 52. Prometheus High Availability

A single Prometheus instance can become a single point of failure.

Production architectures may use:

```text
Prometheus-1
Prometheus-2
```

with appropriate replication or federation/remote-storage architecture.

The exact design depends on scale and availability requirements.

---

# 53. Prometheus Storage

Prometheus stores time-series data locally by default.

Important considerations:

```text
Retention
Disk capacity
Scrape volume
Number of series
Query load
```

High-cardinality metrics can increase memory and storage consumption significantly.

---

# 54. Prometheus High Cardinality

Avoid labels with unbounded values.

Problematic examples can include:

```text
user_id
request_id
session_id
```

if used as metric labels.

Each unique combination can create another time series.

This can significantly increase Prometheus resource consumption.

---

# 55. Metric Cardinality Example

Suppose:

```text
10 services
100 endpoints
10 status codes
```

The number of combinations can grow quickly.

Adding:

```text
100,000 user IDs
```

as a metric label can create an enormous number of time series.

Use controlled dimensions for metrics.

---

# 56. Prometheus Resource Monitoring

Monitor Prometheus itself:

```text
CPU
Memory
Disk
Scrape duration
Scrape failures
Target count
Active series
Query latency
Rule evaluation
WAL behavior
```

The monitoring system itself must be monitored.

---

# 57. Scrape Failures

If Prometheus cannot scrape a target:

```text
Target
   X
Prometheus
```

Possible causes:

```text
Network issue
Target down
Service configuration
Authentication
TLS
Wrong endpoint
Pod failure
```

Monitor:

```promql
up
```

Targets with:

```text
up = 0
```

require investigation.

---

# 58. Prometheus Target Health

A dashboard can show:

```text
Target
Status
Endpoint
Job
Namespace
Pod
```

Example:

```text
payment-api       UP
orders-api        UP
inventory-api     DOWN
```

This quickly identifies missing metrics.

---

# 59. Grafana Data Source

Prometheus is configured in Grafana as a data source.

Conceptually:

```text
Grafana
   │
   ↓
Prometheus Data Source
   │
   ↓
PromQL
   │
   ↓
Visualization
```

Grafana sends PromQL queries to Prometheus.

---

# 60. Grafana Panels

Common panel types include:

```text
Time series
Stat
Gauge
Table
Bar chart
Heatmap
```

Use the visualization that best represents the metric.

Examples:

```text
CPU over time → Time series
Node count    → Stat
Pod status    → Table
Latency       → Time series / histogram
```

---

# 61. EKS Production Dashboard Layout

```text
┌───────────────────────────────────────────────┐
│              EKS OVERVIEW                    │
├───────────────────────────────────────────────┤
│ Nodes │ Pods │ Pending │ Alerts │ CPU │ RAM │
├───────────────────────────────────────────────┤
│                 CPU Usage                     │
├───────────────────────────────────────────────┤
│                Memory Usage                   │
├───────────────────────────────────────────────┤
│           Pod / Deployment Health             │
├───────────────────────────────────────────────┤
│       Network / Storage / Autoscaling          │
└───────────────────────────────────────────────┘
```

---

# 62. EKS Monitoring During Deployment

Before deployment:

```text
Check baseline
```

During deployment:

```text
Watch:
Pods
Replicas
Errors
Latency
CPU
Memory
```

After deployment:

```text
Compare:
Old version
New version
```

This helps detect regressions.

---

# 63. EKS Monitoring During Traffic Spike

Example:

```text
Traffic ↑
   ↓
CPU ↑
   ↓
HPA ↑
   ↓
Pods ↑
   ↓
Node capacity ↓
   ↓
Cluster Autoscaler
   ↓
Nodes ↑
```

Grafana should make this sequence visible.

---

# 64. EKS Monitoring During Node Failure

Problem:

```text
Node-2 NotReady
```

Dashboard should show:

```text
Ready Nodes ↓
Pending Pods ↑
CPU utilization on remaining Nodes ↑
```

Then verify:

```text
Autoscaling
Pod rescheduling
Application availability
```

---

# 65. EKS Monitoring During OOMKilled

Metric signals:

```text
Memory ↑
Restarts ↑
```

Kubernetes state:

```text
Container terminated
Reason = OOMKilled
```

Application logs can then provide context.

This demonstrates why:

```text
Metrics + Kubernetes State + Logs
```

should be correlated.

---

# 66. Prometheus + Grafana + ELK

A complete monitoring stack can be:

```text
                     EKS
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      Metrics        Logs         Traces
        │             │             │
        ↓             ↓             ↓
   Prometheus         ELK       OpenTelemetry
        │             │             │
        ↓             ↓             ↓
     Grafana        Kibana          Jaeger
```

Each system has a different responsibility.

---

# 67. Metrics vs Logs vs Traces

| Signal  | Main Question        |
| ------- | -------------------- |
| Metrics | Is something wrong?  |
| Logs    | What happened?       |
| Traces  | Where did it happen? |

Example:

```text
Grafana:
Payment latency ↑

Jaeger:
Database span = 1.5 sec

Kibana:
Database timeout
```

Together they provide a strong troubleshooting path.

---

# 68. EKS Monitoring Troubleshooting Workflow

```text
Alert
 ↓
Grafana
 ↓
Identify affected layer
 ↓
Cluster / Node / Pod / Application
 ↓
PromQL investigation
 ↓
Kubernetes Events
 ↓
Logs
 ↓
Traces
 ↓
Root Cause
```

---

# 69. Example: 503 Error

Grafana:

```text
HTTP 5xx ↑
```

Check:

```text
ALB
 ↓
Service
 ↓
Endpoints
 ↓
Pods
```

Then:

```text
Prometheus
 ↓
Pod readiness / restart metrics
```

Then:

```text
ELK
 ↓
Application errors
```

Then:

```text
Jaeger
 ↓
Failed dependency
```

---

# 70. Example: High CPU

Grafana shows:

```text
Node CPU = 95%
```

Drill down:

```text
Node
 ↓
Pods
 ↓
Deployment
 ↓
Container
```

Then determine:

```text
Expected workload increase?
Application problem?
Missing HPA?
Insufficient capacity?
```

---

# 71. Example: High Memory

Grafana:

```text
Pod memory = 95%
```

Check:

```text
Memory limit
Restarts
OOMKilled
Application behavior
```

If the Pod is repeatedly OOMKilled:

```text
Resource limit
or
Application memory behavior
```

needs investigation.

---

# 72. Example: Pending Pods

Grafana:

```text
Pending Pods > 0
```

Then:

```bash
kubectl describe pod <pod>
```

Check:

```text
FailedScheduling
```

Possible causes:

```text
CPU
Memory
Taints
Affinity
Storage
Pod IPs
Node limits
Autoscaling
```

---

# 73. Example: CoreDNS Failure

Grafana:

```text
CoreDNS errors ↑
```

Then:

```text
CoreDNS Pods
 ↓
CPU / Memory
 ↓
Restarts
 ↓
DNS query failures
```

Correlate with application errors.

---

# 74. Example: Deployment Regression

Before deployment:

```text
p95 = 250 ms
```

After deployment:

```text
p95 = 900 ms
```

Grafana identifies the regression.

Then:

```text
Jaeger
 ↓
Slow span
```

and:

```text
Kibana
 ↓
Related error
```

This provides a complete deployment investigation.

---

# 75. Production Best Practices

```text
1. Monitor the EKS cluster at multiple layers.
2. Deploy Node Exporter appropriately.
3. Use kube-state-metrics for Kubernetes object state.
4. Monitor application metrics.
5. Use Prometheus for time-series metrics.
6. Use Grafana for visualization.
7. Build cluster-level dashboards.
8. Build Node-level dashboards.
9. Build Pod-level dashboards.
10. Build application dashboards.
11. Monitor HPA.
12. Monitor Cluster Autoscaler.
13. Monitor CoreDNS.
14. Monitor networking and storage.
15. Monitor Prometheus itself.
16. Control metric cardinality.
17. Define actionable alerts.
18. Avoid alert fatigue.
19. Use dashboard drill-down.
20. Correlate metrics with logs and traces.
```

---

# 76. Production Architecture

```text
                              AWS
                               │
                              VPC
                               │
                              EKS
                               │
       ┌───────────────────────┼───────────────────────┐
       ↓                       ↓                       ↓
     Nodes                    Pods                Kubernetes
       │                       │                       │
       ↓                       ↓                       ↓
Node Exporter          Application Metrics      kube-state-metrics
       │                       │                       │
       └───────────────────────┼───────────────────────┘
                               ↓
                           Prometheus
                               │
                     ┌─────────┴─────────┐
                     ↓                   ↓
                PromQL Queries       Alert Rules
                     │                   │
                     └─────────┬─────────┘
                               ↓
                            Grafana
                               │
              ┌────────────────┼────────────────┐
              ↓                ↓                ↓
          Dashboards        Alerts          Drill-down
```

---

# 77. Prometheus and Grafana Security

Secure the monitoring platform with:

```text
Authentication
Authorization
RBAC
TLS
Network policies
Secret management
```

Avoid exposing Grafana or Prometheus directly to the public internet unless there is a deliberate and secured architecture.

---

# 78. Monitoring Multi-Environment EKS

For:

```text
Development
Staging
Production
```

use labels such as:

```text
environment
cluster
namespace
```

Example:

```text
environment="production"
cluster="prod-eks"
namespace="payments"
```

Grafana variables can then switch between environments.

---

# 79. Monitoring Multiple EKS Clusters

For multiple clusters:

```text
Cluster A
Cluster B
Cluster C
```

Prometheus architecture may use:

```text
Separate Prometheus
or
Centralized metrics architecture
```

depending on scale and operational requirements.

Dashboards should identify:

```text
cluster
environment
region
```

to prevent confusion.

---

# 80. Monitoring the Monitoring Stack

Prometheus and Grafana themselves must be monitored.

Monitor:

```text
Prometheus
├── CPU
├── Memory
├── Disk
├── Scrape failures
├── Active series
├── Query latency
└── Rule evaluation

Grafana
├── Availability
├── CPU
├── Memory
├── Query failures
└── Dashboard performance
```

A broken monitoring system can create an operational blind spot.

---

# 81. Interview Question

### How do you monitor EKS using Prometheus and Grafana?

**Answer:**

I use Prometheus to collect Kubernetes, Node, and application metrics. Node Exporter provides infrastructure metrics, while kube-state-metrics exposes Kubernetes object state such as Pod and Deployment health. Prometheus stores the metrics and supports PromQL queries. Grafana connects to Prometheus and provides dashboards and alerting views for cluster, Node, Pod, workload, networking, storage, and application health.

---

# 82. Interview Question

### What is the difference between Node Exporter and kube-state-metrics?

**Answer:**

Node Exporter provides host-level infrastructure metrics such as CPU, memory, disk, and network usage. kube-state-metrics provides metrics representing the state of Kubernetes objects, such as desired versus available Deployment replicas, Pod status, and Node conditions.

---

# 83. Interview Question

### How would you troubleshoot high CPU in EKS using Grafana?

**Answer:**

I would first determine whether the CPU increase is at the cluster, Node, namespace, Pod, or container level. Then I would identify the workloads consuming the most CPU using Prometheus metrics. I would correlate the increase with traffic, deployment changes, HPA behavior, and application metrics. If required, I would inspect logs and traces to determine whether the CPU increase is caused by an application issue.

---

# 84. Interview Question

### How would you troubleshoot a Pending Pod using Prometheus and Kubernetes?

**Answer:**

I would use Grafana to identify the Pending Pod and check cluster capacity, Node utilization, and Pod counts. Then I would run `kubectl describe pod` and inspect the Events for scheduling failures. I would check CPU and memory requests, taints, affinity, topology constraints, storage, IP availability, and autoscaling behavior.

---

# 85. Interview Question

### Why is metric cardinality important in Prometheus?

**Answer:**

Every unique combination of metric labels creates a time series. If labels contain high-cardinality values such as unique user IDs or request IDs, the number of time series can grow rapidly. This increases Prometheus memory, storage, and query requirements. Therefore, metric labels should use controlled and meaningful dimensions.

---

# 86. Interview Question

### How do you monitor HPA and Cluster Autoscaler?

**Answer:**

For HPA, I monitor current versus desired replicas, target metrics, minimum and maximum replicas, and scaling behavior. For Cluster Autoscaler, I monitor Pending Pods, Node Group capacity, Node count, scale-up and scale-down events, and Node provisioning time. I correlate these signals with application traffic and resource utilization.

---

# 87. Interview Question

### What would you alert on in production EKS?

**Answer:**

I would prioritize actionable conditions such as NodeNotReady, sustained high resource utilization, critical disk pressure, Pending Pods, unavailable Deployment replicas, high application error rates, high latency, failed Prometheus targets, CoreDNS problems, and autoscaling failures. I would avoid creating alerts for every temporary metric fluctuation.

---

# 88. Interview Question

### How do Prometheus and Grafana work together?

**Answer:**

Prometheus collects and stores time-series metrics and provides PromQL for querying them. Grafana connects to Prometheus as a data source and uses those queries to build dashboards, visualizations, and alerting views. Prometheus is the metrics backend, while Grafana is primarily the visualization and dashboard layer.

---

# 89. EKS Prometheus & Grafana Checklist

```text
PROMETHEUS
[ ] Installation
[ ] Scrape configuration
[ ] Kubernetes service discovery
[ ] Retention
[ ] Storage
[ ] High availability
[ ] Resource limits
[ ] Target health
[ ] Scrape failures
[ ] Query performance
[ ] Cardinality

EXPORTERS
[ ] Node Exporter
[ ] kube-state-metrics
[ ] Application exporters
[ ] Kubelet metrics

GRAFANA
[ ] Prometheus data source
[ ] Cluster dashboard
[ ] Node dashboard
[ ] Pod dashboard
[ ] Workload dashboard
[ ] Application dashboard
[ ] Variables
[ ] Drill-down
[ ] Alerts

EKS
[ ] Nodes
[ ] Pods
[ ] Deployments
[ ] HPA
[ ] Cluster Autoscaler
[ ] CoreDNS
[ ] Networking
[ ] Storage
[ ] Load Balancer

APPLICATION
[ ] Request rate
[ ] Error rate
[ ] p50 latency
[ ] p95 latency
[ ] p99 latency
[ ] CPU
[ ] Memory
[ ] Restarts

OPERATIONS
[ ] Alerting
[ ] Alert severity
[ ] Alert fatigue
[ ] Capacity
[ ] Security
[ ] RBAC
[ ] TLS
[ ] Monitoring the monitoring stack
```

---

# 90. Final Mental Model

```text
                         EKS
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
      Nodes              Pods          Kubernetes State
        │                 │                 │
        ↓                 ↓                 ↓
Node Exporter      App Metrics       kube-state-metrics
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                      Prometheus
                          │
                         PromQL
                          │
                          ↓
                       Grafana
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
    Dashboards          Alerts          Drill-down
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                 Incident Investigation
                          │
            ┌─────────────┼─────────────┐
            ↓             ↓             ↓
         Metrics         Logs         Traces
            ↓             ↓             ↓
       Prometheus          ELK       OpenTelemetry
            ↓             ↓             ↓
         Grafana         Kibana       Jaeger
```

**Key principle:** Prometheus and Grafana form the core metrics visualization layer for EKS. Prometheus collects and stores metrics from Nodes, Kubernetes objects, and applications, while Grafana turns those metrics into dashboards, drill-down views, and actionable alerts. A strong EKS monitoring setup combines **Node Exporter + kube-state-metrics + application metrics + Prometheus + Grafana**, then correlates those metrics with centralized logs and distributed traces for complete troubleshooting.
