# Kubernetes Metrics

## 1. Overview

Kubernetes metrics provide quantitative information about the health, performance, and resource usage of Kubernetes clusters and workloads.

Metrics help answer questions such as:

```text
How much CPU is being used?
How much memory is being consumed?
How many Pods are running?
How many replicas are available?
Are Nodes under pressure?
Are applications receiving traffic?
Are requests failing?
Is the cluster running out of capacity?
```

A typical Kubernetes metrics architecture is:

```text
Kubernetes
    │
    ├── Node Metrics
    ├── Pod Metrics
    ├── Container Metrics
    ├── Object State Metrics
    └── Application Metrics
             │
             ↓
        Metrics Collection
             │
        ┌────┴────┐
        ↓         ↓
 Metrics Server  Prometheus
                    ↓
                 Grafana
                    ↓
                 Alerts
```

---

# 2. Types of Kubernetes Metrics

Kubernetes metrics can broadly be divided into:

```text
1. Resource Metrics
2. Object State Metrics
3. Node Metrics
4. Container Metrics
5. Application Metrics
6. Control Plane Metrics
7. Network Metrics
8. Storage Metrics
```

Each category provides a different view of the cluster.

---

# 3. Resource Metrics

Resource metrics primarily describe resource consumption.

Common examples:

```text
CPU usage
Memory usage
```

For Pods:

```text
Pod
├── CPU
└── Memory
```

For Nodes:

```text
Node
├── CPU
└── Memory
```

These metrics are commonly used for immediate resource inspection and autoscaling.

---

# 4. Metrics Server

Metrics Server is a lightweight component that collects resource usage metrics from Kubernetes nodes and exposes them through the Kubernetes resource metrics API.

It is commonly used by:

```text
kubectl top
Horizontal Pod Autoscaler
```

Architecture:

```text
Nodes / Kubelets
       ↓
Metrics Server
       ↓
Resource Metrics API
       ↓
kubectl top / HPA
```

---

# 5. kubectl top

`kubectl top` displays current resource consumption.

For Nodes:

```bash
kubectl top nodes
```

For Pods:

```bash
kubectl top pods
```

For a namespace:

```bash
kubectl top pods -n production
```

Example:

```text
NAME              CPU(cores)   MEMORY(bytes)
orders-7c8d       250m         420Mi
payment-5f9a      500m         780Mi
inventory-6a4d    120m         310Mi
```

These values are useful for current-state troubleshooting.

---

# 6. kubectl top nodes

Example:

```bash
kubectl top nodes
```

Output may look like:

```text
NAME       CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node-1     850m         42%    5Gi             63%
node-2     620m         31%    4Gi             50%
```

This helps identify nodes with unusually high resource usage.

---

# 7. kubectl top pods

Example:

```bash
kubectl top pods -A
```

This helps identify workloads consuming the most resources.

For example:

```text
payment     900m CPU
orders      700m CPU
inventory   150m CPU
```

If one Pod is significantly higher than others, investigate whether it is receiving more traffic or has abnormal application behavior.

---

# 8. Metrics Server vs Prometheus

These are frequently confused.

### Metrics Server

Primarily provides:

```text
Current CPU usage
Current memory usage
Resource Metrics API
kubectl top
HPA resource metrics
```

### Prometheus

Provides:

```text
Time-series storage
Historical metrics
PromQL
Dashboards
Alerting integration
Application metrics
Kubernetes metrics
```

Therefore:

```text
Metrics Server ≠ Prometheus
```

---

# 9. Prometheus

Prometheus is a time-series monitoring system commonly used for Kubernetes observability.

Architecture:

```text
Kubernetes
     ↓
Metrics endpoints
     ↓
Prometheus
     ↓
PromQL
     ↓
Grafana
```

Prometheus can collect:

```text
Node metrics
Container metrics
Kubernetes object metrics
Application metrics
Control-plane metrics
```

---

# 10. Prometheus Pull Model

Prometheus commonly uses a pull-based model.

```text
Prometheus
     │
     │ scrape
     ↓
Target
     │
     └── /metrics
```

Example:

```text
Prometheus
    ↓
http://application:8080/metrics
```

Prometheus periodically retrieves the exposed metrics.

---

# 11. Metrics Endpoint

Applications can expose a metrics endpoint:

```text
/metrics
```

Example:

```text
http_requests_total
http_request_duration_seconds
```

Prometheus scrapes the endpoint.

```text
Application
     ↓
/metrics
     ↓
Prometheus
```

---

# 12. Kubernetes Service Discovery

Kubernetes environments are dynamic.

Pods can:

```text
Start
Stop
Restart
Scale
Move between Nodes
```

Prometheus can use Kubernetes service discovery to discover monitoring targets dynamically.

```text
Kubernetes API
      ↓
Prometheus Service Discovery
      ↓
Targets
      ↓
Scraping
```

---

# 13. kube-state-metrics

`kube-state-metrics` exposes metrics about the state of Kubernetes objects.

It does not measure raw resource consumption in the same way as Metrics Server.

Instead, it exposes information about objects such as:

```text
Pods
Deployments
StatefulSets
DaemonSets
Jobs
Nodes
Namespaces
PersistentVolumes
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

# 14. Example kube-state-metrics Information

For a Deployment, useful information includes:

```text
Desired replicas
Available replicas
Updated replicas
Unavailable replicas
```

For a Pod:

```text
Pod phase
Container status
Container readiness
Container restart information
```

For a Node:

```text
Node conditions
Node state
```

This makes Kubernetes object state queryable through Prometheus.

---

# 15. Node Exporter

Node Exporter provides host-level metrics.

Common metrics include:

```text
CPU
Memory
Disk
Filesystem
Network
Load
```

In Kubernetes it is commonly deployed as a DaemonSet.

```text
Node-1 → Node Exporter
Node-2 → Node Exporter
Node-3 → Node Exporter
```

Prometheus scrapes each exporter.

```text
Node Exporter
      ↓
Prometheus
```

---

# 16. Node Metrics Architecture

A typical architecture is:

```text
                     Kubernetes Node
                           │
              ┌────────────┴────────────┐
              ↓                         ↓
        Node Exporter              Kubelet
              │                         │
              ↓                         ↓
          Prometheus              Container Metrics
```

These sources provide different types of information.

---

# 17. CPU Metrics

CPU metrics can be used to understand:

```text
CPU utilization
CPU saturation
CPU throttling
CPU capacity
CPU requests
CPU limits
```

Example:

```text
Node CPU utilization = 85%
```

This does not automatically mean the node is unhealthy.

You should also check:

```text
Application latency
CPU saturation
Pod distribution
CPU requests
CPU limits
```

---

# 18. CPU Utilization vs Saturation

These concepts are different.

### Utilization

How much CPU is currently being used.

```text
CPU usage = 80%
```

### Saturation

How much work is waiting because available CPU is insufficient.

Conceptually:

```text
Work
 ↓
CPU
 ↓
Queue / waiting
```

A system can have high utilization and high saturation at the same time.

---

# 19. CPU Throttling

If a container has a CPU limit, Linux may throttle CPU usage when the configured limit is reached.

Example:

```yaml
resources:
  limits:
    cpu: "1"
```

If the application repeatedly hits the CPU limit, CPU throttling can contribute to latency.

Therefore, monitor both:

```text
CPU utilization
CPU throttling
```

---

# 20. Memory Metrics

Memory monitoring should cover:

```text
Memory usage
Memory working set
Memory requests
Memory limits
Memory pressure
OOM events
```

Example:

```text
Memory limit = 1Gi
Usage = 950Mi
```

This workload is close to its configured limit.

If usage continues increasing:

```text
950Mi
 ↓
1Gi
 ↓
OOMKilled
```

---

# 21. OOMKilled Metrics

OOMKilled is an important Kubernetes signal.

Typical sequence:

```text
Memory usage ↑
      ↓
Container reaches memory limit
      ↓
Container terminated
      ↓
Restart
```

Monitoring should identify repeated OOM events.

A high restart rate combined with OOMKilled is a strong signal of memory pressure inside the workload.

---

# 22. Memory Requests vs Usage

Example:

```text
Memory request = 512Mi
Memory usage   = 900Mi
```

The container may be using substantially more memory than its request.

This can create scheduling and capacity-planning concerns.

Always compare:

```text
Actual usage
Requests
Limits
Node allocatable capacity
```

---

# 23. Memory Pressure

Node-level memory pressure is different from a single container exceeding its limit.

Example:

```text
Node memory pressure
        ↓
Kubernetes may evict Pods
```

Monitor:

```text
Node memory utilization
MemoryPressure condition
Evictions
Pod memory usage
```

---

# 24. Disk Metrics

Disk monitoring should cover:

```text
Filesystem usage
Disk I/O
Disk latency
Available space
Inodes
Ephemeral storage
```

Disk problems can affect:

```text
Container runtime
Application writes
Container logs
Image pulls
Kubernetes components
```

---

# 25. Disk Pressure

Kubernetes can report:

```text
DiskPressure = True
```

Possible causes include:

```text
Large container logs
Unused images
Temporary files
Ephemeral storage usage
Application-generated files
```

Monitor disk usage before the node reaches critical levels.

---

# 26. Network Metrics

Network monitoring can include:

```text
Bytes received
Bytes transmitted
Packets received
Packets transmitted
Packet errors
Dropped packets
Connections
Network latency
```

At the Kubernetes level also monitor:

```text
Pod-to-Pod communication
Service traffic
Ingress traffic
DNS
Load balancers
```

---

# 27. Application Metrics

Infrastructure metrics alone are not sufficient.

A production application should expose useful business and technical metrics.

Examples:

```text
HTTP requests
HTTP errors
Request duration
Active connections
Queue depth
Database connections
Cache hits
Business transactions
```

---

# 28. RED Metrics

For request-based services, use:

```text
R = Rate
E = Errors
D = Duration
```

Example:

```text
Rate:
2,000 requests/sec

Errors:
2% HTTP 5xx

Duration:
p95 = 600ms
```

This gives a useful high-level picture of application health.

---

# 29. USE Metrics

For infrastructure resources:

```text
U = Utilization
S = Saturation
E = Errors
```

Example:

```text
CPU utilization
CPU saturation
CPU-related errors
```

This is particularly useful for:

```text
Nodes
Storage
Network
Infrastructure resources
```

---

# 30. Golden Signals

Another useful monitoring model is the four Golden Signals:

```text
Latency
Traffic
Errors
Saturation
```

Example:

```text
Latency     → p95 response time
Traffic     → Requests/sec
Errors      → HTTP 5xx
Saturation  → CPU / memory / queue
```

These signals connect infrastructure health with user experience.

---

# 31. Kubernetes Object Metrics

Object-state monitoring can answer:

```text
How many replicas are desired?
How many are available?
How many Pods are ready?
How many Pods are pending?
How many Jobs failed?
```

This information is commonly provided to Prometheus through kube-state-metrics.

---

# 32. Deployment Metrics

Important Deployment metrics include:

```text
Desired replicas
Available replicas
Ready replicas
Updated replicas
Unavailable replicas
```

Healthy example:

```text
Desired   = 5
Available = 5
Ready     = 5
Updated   = 5
```

Problem:

```text
Desired   = 5
Available = 3
Ready     = 3
```

This should generate investigation or alerting based on the service's availability requirements.

---

# 33. StatefulSet Metrics

Monitor:

```text
Desired replicas
Ready replicas
Current replicas
Updated replicas
```

Also monitor associated storage:

```text
PVC status
Volume health
Storage capacity
```

A StatefulSet can be healthy at the Pod level while its storage is approaching capacity.

---

# 34. DaemonSet Metrics

Monitor:

```text
Desired
Current
Ready
Available
Updated
```

Example:

```text
Desired = 10
Ready   = 8
```

This indicates that two nodes may not have healthy DaemonSet Pods.

---

# 35. Job Metrics

For Jobs:

```text
Active
Succeeded
Failed
Completion time
Duration
```

Repeated Job failures should be investigated.

---

# 36. CronJob Metrics

For CronJobs monitor:

```text
Last schedule
Successful executions
Failed executions
Missed schedules
Execution duration
```

A CronJob can stop working while the rest of the cluster remains healthy.

Therefore, scheduled workloads need their own monitoring.

---

# 37. Pod Phase Metrics

Pod phases include:

```text
Pending
Running
Succeeded
Failed
Unknown
```

Monitoring a large number of:

```text
Pending Pods
```

may indicate:

```text
Capacity problem
Scheduling constraints
Storage issue
Image problem
```

---

# 38. Container Status Metrics

Monitor container states such as:

```text
Running
Waiting
Terminated
```

Also investigate:

```text
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
OOMKilled
```

These states can provide early indicators of workload failure.

---

# 39. Restart Metrics

A restart count alone is not always meaningful.

Instead monitor the rate of restarts.

Conceptually:

```text
Restart count:
0 → 1 → 2 → 3 → 4 → 5
```

A continuously increasing rate is more concerning than an old Pod that restarted once months ago.

---

# 40. Readiness Metrics

Readiness determines whether a Pod should receive traffic.

Monitor:

```text
Ready Pods
Not Ready Pods
Readiness failures
Endpoint availability
```

Example:

```text
Deployment replicas = 5
Ready replicas      = 3
```

Potential impact:

```text
Reduced capacity
Higher latency
Traffic failures
```

---

# 41. Liveness Metrics

Liveness probes determine whether a container should be restarted.

Monitor:

```text
Liveness failures
Container restarts
Restart frequency
```

If liveness failures occur frequently, the problem may be:

```text
Application instability
Probe configuration
Resource pressure
Dependency behavior
```

Simply increasing the probe timeout may hide the underlying problem.

---

# 42. Startup Probe Metrics

Startup probes protect slow-starting applications.

Monitor:

```text
Startup failures
Startup duration
Container restarts during startup
```

This is especially useful for:

```text
Large Java applications
Applications with migrations
Slow initialization
Heavy dependency loading
```

---

# 43. HPA Metrics

Horizontal Pod Autoscaler can use metrics such as:

```text
CPU utilization
Memory utilization
Custom metrics
External metrics
```

Monitor:

```text
Current replicas
Desired replicas
Metric value
Target value
Scaling events
```

Example:

```text
CPU = 85%
Target = 60%

Current replicas = 3
Desired replicas = 5
```

---

# 44. HPA Troubleshooting With Metrics

Suppose:

```text
CPU = 90%
Current replicas = 3
Desired replicas = 3
```

Investigate:

```text
Maximum replicas
HPA configuration
Metrics availability
Resource requests
Scaling behavior
```

If:

```text
Desired replicas = 8
Current replicas = 3
```

but Pods remain Pending, the problem may be cluster capacity rather than HPA.

---

# 45. VPA Metrics

If Vertical Pod Autoscaler is used, monitor:

```text
Current requests
Recommended requests
Target requests
Update behavior
```

VPA recommendations can help identify workloads that are consistently over- or under-provisioned.

---

# 46. Cluster Autoscaler Metrics

Monitor:

```text
Current node count
Desired node count
Scale-up events
Scale-down events
Unschedulable Pods
Autoscaler errors
```

Example:

```text
Pending Pods
      ↓
Cluster Autoscaler
      ↓
Scale-up
      ↓
New Node
      ↓
Pod Scheduled
```

---

# 47. API Server Metrics

The Kubernetes API Server is critical.

Important metrics include:

```text
Request rate
Request latency
Request errors
Response status
In-flight requests
```

High API latency can affect:

```text
kubectl
Controllers
Scheduler
Operators
Autoscalers
Deployments
```

---

# 48. API Server Request Errors

Monitor HTTP status categories such as:

```text
2xx
4xx
5xx
```

A sudden increase in:

```text
5xx
```

may indicate control-plane problems.

A high number of:

```text
4xx
```

may indicate authentication, authorization, or client configuration problems.

---

# 49. Scheduler Metrics

Monitor:

```text
Scheduling latency
Scheduling attempts
Scheduling failures
Pending Pods
```

If scheduling latency increases:

```text
Pod creation
    ↓
Scheduler delay
    ↓
Application deployment delay
```

This can become especially important during cluster scaling events.

---

# 50. Controller Manager Metrics

Monitor:

```text
Controller work queue
Queue latency
Reconciliation errors
Controller processing rate
```

Controllers are responsible for continuously reconciling Kubernetes resources.

A controller problem can result in:

```text
Desired state ≠ Actual state
```

---

# 51. Kubelet Metrics

Kubelet metrics can provide visibility into:

```text
Pod lifecycle
Container operations
Runtime interactions
Node health
```

Monitor:

```text
Kubelet errors
Operation latency
Pod startup
Container operations
```

A kubelet problem can affect multiple workloads on a node.

---

# 52. Container Runtime Metrics

The container runtime is responsible for managing containers.

Problems can appear as:

```text
Container creation failures
Image pull failures
Container startup delays
Container runtime errors
```

These should be correlated with:

```text
Kubelet
Node
Container
Application
```

---

# 53. Storage Metrics

Monitor:

```text
PVC usage
PV capacity
Volume I/O
Volume latency
Provisioning failures
Filesystem usage
```

Example:

```text
PVC capacity = 100Gi
Used          = 92Gi
```

This should trigger capacity planning before the volume becomes full.

---

# 54. Storage Provisioning

Monitor StorageClass and CSI components.

Architecture:

```text
PVC
 ↓
StorageClass
 ↓
CSI Provisioner
 ↓
Storage Backend
```

If provisioning fails:

```text
PVC = Pending
```

Check CSI controller and node components.

---

# 55. Network Metrics in Kubernetes

Monitor:

```text
Pod network traffic
Node network traffic
Service traffic
Ingress traffic
DNS traffic
Network errors
```

For CNI-related issues, also monitor:

```text
CNI health
IP allocation
Network interface capacity
Packet drops
```

---

# 56. Service Metrics

For Services, monitor:

```text
Request traffic
Backend availability
Endpoint count
Connection errors
Latency
```

A useful correlation is:

```text
Service
 ↓
Endpoints
 ↓
Pods
```

If endpoint count drops unexpectedly, application availability may also decrease.

---

# 57. Ingress Metrics

Monitor:

```text
Requests
4xx
5xx
Latency
Connections
Backend response
```

Example:

```text
Requests/sec = 5,000
5xx = 3%
p95 = 800ms
```

These signals help distinguish ingress problems from application problems.

---

# 58. Metrics Labels

Prometheus metrics contain labels.

Example concept:

```text
http_requests_total{
  service="payment",
  namespace="production",
  method="POST",
  status="200"
}
```

Labels make metrics filterable.

However, excessive label cardinality can create serious performance and storage problems.

---

# 59. High Cardinality

Avoid labels containing highly unique values such as:

```text
user_id
request_id
session_id
random UUID
```

For example:

```text
request_id=abc123
request_id=def456
request_id=ghi789
```

This can create huge numbers of unique time series.

Prefer controlled dimensions such as:

```text
service
namespace
method
route
status
environment
```

---

# 60. Metric Cardinality

Cardinality means the number of unique combinations of label values.

Example:

```text
service × method × status
```

is usually manageable.

But:

```text
service × user_id × request_id × session_id
```

can explode the number of time series.

Monitor and control cardinality in production.

---

# 61. PromQL

Prometheus Query Language is used to query metrics.

Example:

```promql
up
```

CPU:

```promql
rate(node_cpu_seconds_total[5m])
```

The exact query should depend on the metric names exposed by the deployed exporters and Kubernetes monitoring stack.

---

# 62. Rate

`rate()` calculates the per-second average increase of a counter over a time range.

Example:

```promql
rate(http_requests_total[5m])
```

This can be used to estimate:

```text
Requests per second
```

---

# 63. Increase

`increase()` calculates the total increase of a counter over a period.

Example:

```promql
increase(http_requests_total[1h])
```

This can help answer:

```text
How many requests occurred during the last hour?
```

---

# 64. Aggregation

PromQL can aggregate metrics.

Example:

```promql
sum(rate(http_requests_total[5m]))
```

This can provide a total request rate across matching series.

You can also aggregate by labels:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

This gives request rate per service.

---

# 65. CPU Monitoring With Prometheus

A common approach is to calculate CPU usage from cumulative CPU time.

Conceptually:

```promql
rate(container_cpu_usage_seconds_total[5m])
```

The exact query should be adjusted for the metrics exposed by the container runtime and monitoring stack.

---

# 66. Memory Monitoring With Prometheus

Container memory metrics can be queried from the container metrics exposed to Prometheus.

Conceptually:

```promql
container_memory_working_set_bytes
```

Then filter by:

```text
namespace
pod
container
```

This allows engineers to identify high-memory workloads.

---

# 67. Grafana Dashboards

Grafana can visualize Prometheus metrics.

Architecture:

```text
Prometheus
     ↓
   PromQL
     ↓
  Grafana
     ↓
Dashboards
```

Useful dashboards include:

```text
Cluster Overview
Node Overview
Pod Overview
Namespace Overview
Application Overview
Kubernetes Control Plane
```

---

# 68. Dashboard Design

Avoid dashboards containing hundreds of unrelated panels.

A useful dashboard should answer a specific question.

Example:

```text
Cluster Overview
├── Node health
├── CPU
├── Memory
├── Pending Pods
├── Restarts
├── API latency
└── Capacity
```

Detailed dashboards can then drill into:

```text
Node
 ↓
Namespace
 ↓
Deployment
 ↓
Pod
```

---

# 69. Alerting

Metrics become operationally useful when important conditions trigger alerts.

Examples:

```text
Node unavailable
Pod unavailable
High restart rate
High memory pressure
High CPU saturation
Persistent Pending Pods
High 5xx rate
High latency
PVC nearing capacity
Collector unavailable
```

---

# 70. Alert Duration

Avoid alerting on very short spikes unless the problem is genuinely critical.

Example:

```text
CPU > 90%
for 10 minutes
```

rather than:

```text
CPU > 90%
for 1 second
```

The correct duration depends on the workload.

---

# 71. Alert Symptoms Instead of Causes

A useful alert:

```text
Service availability below SLO
```

can be more actionable than:

```text
CPU > 80%
```

because high CPU is not necessarily a user-facing problem.

Use infrastructure alerts for conditions that require operational action.

---

# 72. SLO Metrics

For production services, monitor:

```text
Availability
Latency
Error rate
Throughput
```

Example:

```text
Availability SLO = 99.9%
Latency SLO = p95 < 500ms
```

Then track:

```text
Error budget
Burn rate
SLO compliance
```

---

# 73. Metrics Correlation

A single metric rarely gives the complete answer.

Example:

```text
CPU ↑
```

Check:

```text
CPU
 ↓
Request rate
 ↓
Latency
 ↓
Errors
```

If:

```text
CPU ↑
Traffic ↑
Latency stable
Errors stable
```

the increase may simply reflect higher demand.

---

# 74. Metric Correlation Example

Suppose:

```text
CPU = 95%
```

But:

```text
Traffic = normal
Latency = high
Errors = high
```

This is more concerning.

Possible causes:

```text
Application inefficiency
CPU throttling
Infinite loop
Dependency processing
Unexpected workload
```

Metrics should be analyzed together.

---

# 75. Kubernetes Capacity Model

Monitor:

```text
Cluster capacity
       ↓
Allocatable capacity
       ↓
Requested capacity
       ↓
Actual utilization
       ↓
Saturation
```

These are not the same thing.

For example:

```text
Capacity     = 100 CPU
Allocatable  = 90 CPU
Requested    = 85 CPU
Actual usage = 45 CPU
```

The cluster may look underutilized based on actual usage but have little schedulable capacity because requests are high.

---

# 76. Monitoring Resource Efficiency

Compare:

```text
Actual usage
       vs
Requested resources
```

If:

```text
Request = 2 CPU
Usage   = 200m
```

the workload may be over-requesting CPU.

If:

```text
Request = 200m
Usage   = 900m
```

the workload may be under-requesting CPU.

Resource efficiency improves when requests reflect realistic workload behavior.

---

# 77. Monitoring Node Utilization

A node should be evaluated using:

```text
CPU utilization
Memory utilization
Disk utilization
Network utilization
Pod count
Resource requests
```

Do not judge node health using CPU alone.

---

# 78. Monitoring Namespace Consumption

A namespace dashboard can show:

```text
CPU requested
CPU used
Memory requested
Memory used
Pod count
Restart count
PVC usage
```

This is useful for:

```text
Capacity planning
Team ownership
Environment comparison
Quota management
```

---

# 79. Monitoring Multi-Tenant Clusters

For shared clusters, include:

```text
Namespace
Team
Environment
Application
```

Use controlled labels and resource quotas.

This helps identify:

```text
Which workload consumes resources?
Which namespace is growing?
Which team is approaching quota?
```

---

# 80. Kubernetes Metrics Collection Architecture

A common architecture:

```text
                     Kubernetes
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
  Node Exporter     kube-state-metrics   App / Service
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                     Prometheus
                          ↓
                        PromQL
                          ↓
                       Grafana
                          ↓
                      Alerting
```

Metrics Server may separately provide:

```text
Kubelet
  ↓
Metrics Server
  ↓
Resource Metrics API
  ↓
kubectl top / HPA
```

---

# 81. Monitoring Stack Responsibilities

A useful separation is:

```text
Metrics Server
    → Current resource metrics / Kubernetes resource API

kube-state-metrics
    → Kubernetes object state

Node Exporter
    → Host-level metrics

Prometheus
    → Collection and time-series storage

Grafana
    → Visualization

Alertmanager
    → Alert routing and notification
```

---

# 82. Production Metrics Flow

```text
Application
    ↓
/metrics
    ↓
Prometheus

Node
    ↓
Node Exporter
    ↓
Prometheus

Kubernetes API
    ↓
kube-state-metrics
    ↓
Prometheus

Kubelet
    ↓
Container / node metrics
    ↓
Monitoring stack
```

Then:

```text
Prometheus
    ↓
Grafana
    ↓
Alerts
```

---

# 83. Metrics Retention

Prometheus retention determines how long historical metrics are available.

Retention depends on:

```text
Metric volume
Storage capacity
Operational requirements
Query workload
Long-term monitoring requirements
```

For longer-term retention, organizations may use remote storage systems designed for long-term metrics.

---

# 84. Remote Write

Prometheus can send metrics to a remote time-series backend.

Conceptually:

```text
Prometheus
    ├── Local Storage
    │
    └── Remote Write
            ↓
      Long-term Storage
```

This can be useful for:

```text
Long-term retention
Centralized monitoring
Multi-cluster metrics
```

---

# 85. Multi-Cluster Metrics

For multiple Kubernetes clusters:

```text
EKS Cluster A
      ↓
Prometheus A
      ↓
Central Metrics Backend

EKS Cluster B
      ↓
Prometheus B
      ↓
Central Metrics Backend
```

Include labels such as:

```text
cluster
environment
region
```

This allows Grafana to distinguish metrics from different clusters.

---

# 86. Multi-Environment Metrics

Use labels such as:

```text
environment=production
environment=staging
environment=development
```

Then dashboards can filter:

```text
Environment
Cluster
Namespace
Service
Pod
```

This is useful when the same application runs in multiple environments.

---

# 87. Metrics Security

Protect monitoring endpoints and systems.

Consider:

```text
Authentication
Authorization
TLS
NetworkPolicy
Private networking
RBAC
```

Do not expose internal metrics endpoints publicly without a valid security requirement.

---

# 88. Sensitive Metrics

Metrics can accidentally contain sensitive information through labels.

Avoid labels containing:

```text
Passwords
Tokens
Email addresses
User IDs
Session IDs
Request IDs
```

Metrics should use controlled dimensions.

---

# 89. Monitoring Failure

The monitoring system itself can fail.

Example:

```text
Prometheus
    X
```

If nobody notices, the organization may lose visibility.

Therefore monitor:

```text
Prometheus availability
Scrape failures
Target health
Storage health
Grafana availability
Alertmanager availability
```

---

# 90. Prometheus Target Health

Prometheus exposes target health.

Conceptually:

```text
Target
 ├── UP
 └── DOWN
```

A target being down may mean:

```text
Application unavailable
Pod unavailable
Network problem
Service discovery problem
TLS problem
Metrics endpoint problem
```

Monitor scrape failures.

---

# 91. Scrape Failures

A scrape can fail because:

```text
Target unavailable
Connection timeout
DNS failure
Wrong port
Wrong path
TLS problem
Authentication problem
```

A sudden increase in scrape failures reduces monitoring coverage.

---

# 92. Monitoring Gaps

A dashboard may look healthy simply because metrics are no longer being collected.

Therefore:

```text
No data
```

should not automatically be interpreted as:

```text
Everything is healthy
```

Always distinguish:

```text
Zero
vs
Missing data
```

This is an important production monitoring principle.

---

# 93. Alert on Monitoring Failure

Examples:

```text
Prometheus unavailable
Target scrape failures
Metrics ingestion stopped
Grafana unavailable
Alertmanager unavailable
```

The monitoring platform should have monitoring of its own.

---

# 94. Troubleshooting Metrics

If a metric is missing:

```text
1. Is the target running?
2. Is the metrics endpoint available?
3. Can Prometheus discover it?
4. Is the target being scraped?
5. Is the scrape successful?
6. Does the metric exist?
7. Is the PromQL query correct?
8. Is the Grafana data source healthy?
```

This avoids assuming the application itself is broken.

---

# 95. Example: Missing Application Metrics

Architecture:

```text
Application
   ↓
/metrics
   X
Prometheus
```

Check:

```bash
kubectl get pods -n production
kubectl get svc -n production
```

Then test the metrics endpoint from an appropriate location:

```bash
curl http://<service>:<port>/metrics
```

If the endpoint works but Prometheus cannot scrape it, investigate discovery, configuration, network, or authentication.

---

# 96. Example: Missing Pod Metrics

If:

```text
kubectl top pods
```

does not return expected data, investigate the resource metrics pipeline:

```text
Kubelet
   ↓
Metrics Server
   ↓
Resource Metrics API
   ↓
kubectl top
```

This is different from Prometheus scraping application metrics.

---

# 97. Example: HPA Metrics Missing

If HPA cannot scale:

```text
HPA
 ↓
Metrics unavailable
```

Check:

```text
Metrics API
Resource requests
HPA configuration
Metrics Server
Custom metrics adapter if applicable
```

The exact troubleshooting path depends on the metric source configured for the HPA.

---

# 98. Metrics and Autoscaling

Metrics are often used by:

```text
HPA
VPA
Cluster Autoscaler
```

But each uses different signals and mechanisms.

Conceptually:

```text
Application Metrics
       ↓
HPA
       ↓
Pod replicas

Node / Scheduling State
       ↓
Cluster Autoscaler
       ↓
Node count
```

Do not confuse Pod scaling with Node scaling.

---

# 99. Pod Scaling vs Node Scaling

### HPA

```text
Load ↑
 ↓
Pod replicas ↑
```

### Cluster Autoscaler

```text
Pods cannot be scheduled
 ↓
Node count ↑
```

Both may operate together:

```text
Traffic ↑
 ↓
HPA
 ↓
More Pods
 ↓
Insufficient capacity
 ↓
Cluster Autoscaler
 ↓
More Nodes
```

---

# 100. Production Monitoring Checklist

```text
CLUSTER
[ ] Node count
[ ] CPU capacity
[ ] Memory capacity
[ ] Allocatable resources
[ ] Pending Pods
[ ] API Server
[ ] Scheduler
[ ] Controllers

NODES
[ ] CPU
[ ] Memory
[ ] Disk
[ ] Network
[ ] Node conditions
[ ] Pod capacity
[ ] Filesystem
[ ] Pressure conditions

PODS
[ ] Status
[ ] Readiness
[ ] Restarts
[ ] CPU
[ ] Memory
[ ] OOMKilled
[ ] Probe failures

WORKLOADS
[ ] Deployments
[ ] StatefulSets
[ ] DaemonSets
[ ] Jobs
[ ] CronJobs
[ ] HPA

NETWORK
[ ] Services
[ ] Endpoints
[ ] DNS
[ ] Ingress
[ ] Network errors

STORAGE
[ ] PVCs
[ ] PVs
[ ] Capacity
[ ] I/O
[ ] Provisioning

APPLICATION
[ ] Rate
[ ] Errors
[ ] Latency
[ ] Saturation
[ ] Dependencies

MONITORING STACK
[ ] Metrics Server
[ ] kube-state-metrics
[ ] Node Exporter
[ ] Prometheus
[ ] Grafana
[ ] Alertmanager
```

---

# 101. Interview Question

### What is the difference between Metrics Server and Prometheus?

**Answer:**

Metrics Server provides current CPU and memory resource metrics through the Kubernetes resource metrics API. It is commonly used by `kubectl top` and resource-based HPA. Prometheus is a full time-series monitoring system that collects, stores, queries, and evaluates metrics and is commonly used with Grafana for dashboards and alerting.

---

# 102. Interview Question

### What is kube-state-metrics?

**Answer:**

kube-state-metrics exposes metrics representing the state of Kubernetes objects such as Pods, Deployments, StatefulSets, DaemonSets, Jobs, and Nodes. It reads information from the Kubernetes API and exposes it in a Prometheus-compatible format. It is different from Metrics Server because it focuses primarily on Kubernetes object state rather than current resource consumption.

---

# 103. Interview Question

### How would you monitor Kubernetes Nodes?

**Answer:**

I would monitor CPU, memory, disk, filesystem, network, Pod capacity, and node conditions such as Ready, MemoryPressure, DiskPressure, and PIDPressure. I would use Node Exporter and Prometheus for historical metrics and `kubectl top nodes` for immediate resource inspection.

---

# 104. Interview Question

### How would you identify a Pod consuming too much CPU?

**Answer:**

I would first use:

```bash
kubectl top pods -A
```

to identify the workload. Then I would compare actual usage with its CPU request and limit, check CPU throttling, inspect application metrics and logs, and determine whether the increase is caused by higher traffic, an application problem, or an inefficient workload.

---

# 105. Interview Question

### How would you investigate high memory usage?

**Answer:**

I would identify the affected Pod using resource metrics, compare actual memory usage with requests and limits, and check whether the container has been OOMKilled or restarted. I would then inspect application behavior and logs for memory leaks, traffic changes, large payloads, or abnormal workload patterns.

---

# 106. Interview Question

### How do you monitor Kubernetes object health?

**Answer:**

I use kube-state-metrics to expose Kubernetes object state to Prometheus. I monitor desired versus available replicas for Deployments, ready replicas for StatefulSets and DaemonSets, Pod phases, restart conditions, Jobs, CronJobs, PVC states, and node conditions. Grafana dashboards and Prometheus alerts can then provide historical visibility and notifications.

---

# 107. Interview Question

### How would you monitor HPA?

**Answer:**

I would monitor the current and desired replica counts, the configured target metric, actual metric values, scaling events, and whether newly requested replicas can be scheduled. If HPA wants more replicas but Pods remain Pending, I would investigate cluster capacity, resource requests, taints, affinity, and Cluster Autoscaler behavior.

---

# 108. Interview Question

### How do you avoid Prometheus high-cardinality problems?

**Answer:**

I avoid putting highly unique values such as user IDs, request IDs, session IDs, or UUIDs into metric labels. I prefer controlled labels such as service, namespace, route, method, status, and environment. I also monitor time-series growth and review application instrumentation before introducing new labels.

---

# 109. Interview Question

### What would you monitor in a production Kubernetes cluster?

**Answer:**

I would monitor the cluster at four major levels:

```text
Cluster
 → API Server, Scheduler, Controllers, capacity

Nodes
 → CPU, memory, disk, network, pressure

Workloads
 → Pods, Deployments, StatefulSets, restarts, probes

Applications
 → Rate, Errors, Duration, Saturation
```

I would use Prometheus and Grafana for metrics, ELK for logs, and OpenTelemetry with Jaeger for distributed tracing.

---

# 110. Final Mental Model

Remember Kubernetes metrics as:

```text
                         KUBERNETES
                              │
        ┌─────────────────────┼─────────────────────┐
        ↓                     ↓                     ↓
   Resource Metrics      Object Metrics       Application Metrics
        │                     │                     │
        ↓                     ↓                     ↓
 Metrics Server        kube-state-metrics       /metrics
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ↓
                         Prometheus
                              ↓
                            PromQL
                              ↓
                           Grafana
                              ↓
                          Alerting
```

And remember the supporting components:

```text
Metrics Server
    → Current CPU / memory resource metrics

kube-state-metrics
    → Kubernetes object state

Node Exporter
    → Node / host metrics

Prometheus
    → Time-series collection and storage

Grafana
    → Dashboards and visualization

Alertmanager
    → Alert routing and notifications
```

The most important principle is:

**Kubernetes metrics should provide visibility from the cluster level down to individual applications. Monitor resource usage, capacity, Kubernetes object state, Pod health, node conditions, networking, storage, autoscaling, and application RED/Golden Signals. Use Metrics Server for current resource metrics, kube-state-metrics for Kubernetes object state, Node Exporter for host metrics, and Prometheus as the central time-series monitoring system. Grafana then turns these metrics into dashboards and operational visibility, while alerting converts important metric conditions into actionable notifications.**
