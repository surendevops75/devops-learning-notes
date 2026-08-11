# Cluster Monitoring

## 1. Overview

Cluster monitoring focuses on the overall health, capacity, performance, availability, and behavior of a Kubernetes cluster.

While Pod monitoring focuses on individual workloads and Node monitoring focuses on individual worker machines, Cluster monitoring provides the broader view:

```text
Kubernetes Cluster
│
├── Control Plane
├── Nodes
├── Pods
├── Namespaces
├── Workloads
├── Networking
├── Storage
└── Cluster Capacity
```

The goal is to answer:

```text
Is the cluster healthy?
Does it have enough capacity?
Are workloads being scheduled?
Are Nodes healthy?
Are Pods available?
Is the control plane responding?
Are cluster resources being exhausted?
```

---

# 2. Why Cluster Monitoring Is Important

A Kubernetes cluster can have:

```text
Nodes = Ready
Pods = Running
```

and still experience problems such as:

```text
High API latency
Insufficient capacity
Scheduling failures
IP exhaustion
Storage problems
Control-plane issues
High resource utilization
Network degradation
```

Cluster monitoring identifies problems that are difficult to see from a single Pod or Node.

---

# 3. Three Monitoring Levels

A useful Kubernetes observability model is:

```text
                    Kubernetes
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
      Cluster           Node            Pod
        │               │               │
   Overall health    Infrastructure    Workload
```

### Cluster

```text
Capacity
Control plane
Scheduling
Networking
Storage
```

### Node

```text
CPU
Memory
Disk
Network
Kubelet
```

### Pod

```text
Readiness
Restarts
CPU
Memory
Probes
Application behavior
```

---

# 4. Cluster Monitoring Architecture

A typical monitoring architecture:

```text
                    Kubernetes Cluster
                           │
       ┌───────────────────┼───────────────────┐
       ↓                   ↓                   ↓
   Control Plane          Nodes              Objects
       │                   │                   │
       ↓                   ↓                   ↓
 API Server             Node Exporter     kube-state-metrics
 Scheduler                   │                   │
 Controller                  └─────────┬─────────┘
 Manager                             ↓
                              Prometheus
                                   ↓
                                Grafana
                                   ↓
                                 Alerts
```

Application observability can be added:

```text
Logs   → ELK
Traces → OpenTelemetry → Jaeger
```

---

# 5. Cluster Health

Cluster health should be evaluated across:

```text
Cluster Health
│
├── Control Plane
├── Nodes
├── Pods
├── Scheduling
├── Resource Capacity
├── Networking
├── Storage
├── API Availability
├── Workloads
└── Autoscaling
```

A healthy cluster should have:

```text
Nodes Ready
Pods available
Scheduling successful
API responsive
Adequate capacity
No sustained pressure
Healthy networking
Healthy storage
```

---

# 6. Control Plane Monitoring

The Kubernetes control plane manages cluster state.

Important components include:

```text
API Server
Scheduler
Controller Manager
etcd
```

In managed Kubernetes services such as EKS, many control-plane components are managed by the cloud provider, but their health and behavior still matter operationally.

---

# 7. Kubernetes API Server

The API Server is the main entry point for Kubernetes API operations.

It handles requests from:

```text
kubectl
Controllers
Scheduler
Operators
Applications
Monitoring systems
```

Conceptually:

```text
kubectl
   │
   ↓
API Server
   │
   ├── Authentication
   ├── Authorization
   └── Kubernetes API
```

---

# 8. API Server Monitoring

Important signals include:

```text
Request rate
Request latency
Request errors
HTTP status codes
Rejected requests
API availability
```

High API latency can affect:

```text
kubectl commands
Controllers
Operators
Deployments
Autoscaling
Monitoring
```

---

# 9. API Server Latency

Suppose:

```text
API latency
   ↓
Normal: 50 ms
Current: 2 seconds
```

This may indicate a control-plane problem or high API load.

Investigate:

```text
Request rate
API errors
etcd performance
Controllers
Clients generating excessive requests
```

---

# 10. API Server Error Rate

Monitor HTTP responses such as:

```text
2xx
4xx
5xx
```

A sudden increase in:

```text
5xx errors
```

can indicate a control-plane problem.

However, 4xx responses may also be caused by invalid requests or authorization failures, so the status code should be interpreted in context.

---

# 11. Kubernetes Scheduler

The scheduler determines which Node should run a Pod.

Flow:

```text
Pod created
   ↓
Scheduler
   ↓
Find suitable Node
   ↓
Assign Pod
```

Monitor:

```text
Scheduling latency
Scheduling failures
Pending Pods
Unschedulable Pods
```

---

# 12. Scheduling Failures

A cluster can have healthy Nodes but still fail to schedule Pods.

Possible causes:

```text
Insufficient CPU
Insufficient memory
Taints
Tolerations
Affinity
Anti-affinity
Topology constraints
Pod capacity
Storage constraints
```

Therefore cluster monitoring should track scheduling failures.

---

# 13. Pending Pods at Cluster Level

A few Pending Pods may be normal temporarily.

But:

```text
Pending Pods ↑
```

across multiple namespaces can indicate a cluster-wide capacity or scheduling problem.

Example:

```text
Production → 5 Pending
Development → 3 Pending
Testing → 4 Pending
```

This pattern deserves cluster-level investigation.

---

# 14. Controller Manager

Controllers continuously compare:

```text
Desired state
      vs
Current state
```

Example:

```text
Deployment desired replicas = 5
Current replicas = 3
```

The controller attempts to reconcile the difference.

Monitor controller behavior where metrics are available.

---

# 15. Desired vs Actual State

Kubernetes is fundamentally a reconciliation system.

```text
Desired State
     ↓
Controller
     ↓
Actual State
```

Monitoring should therefore compare:

```text
Desired
vs
Available
vs
Ready
```

for important workloads.

---

# 16. Cluster Node Count

Monitor:

```text
Total Nodes
Ready Nodes
NotReady Nodes
Scheduling-disabled Nodes
```

Example:

```text
Total = 12
Ready = 11
NotReady = 1
```

One Node failure may or may not be critical depending on cluster capacity and workload redundancy.

---

# 17. Cluster Node Capacity

Cluster capacity is the combined capacity of available Nodes.

Conceptually:

```text
Cluster Capacity
│
├── CPU
├── Memory
├── Storage
├── Pod capacity
└── Network capacity
```

Monitor both:

```text
Capacity
and
Actual available headroom
```

---

# 18. Cluster CPU Capacity

Example:

```text
6 Nodes
×
8 CPU each
=
48 CPU capacity
```

But actual Pod scheduling capacity depends on:

```text
System reservations
Kubernetes reservations
Allocatable resources
Existing requests
Scheduling constraints
```

Therefore raw capacity is not enough.

---

# 19. Cluster Allocatable CPU

Example:

```text
Raw capacity = 48 CPU
Allocatable  = 42 CPU
```

The difference is reserved for Node/system components.

Monitor:

```text
Allocatable CPU
Requested CPU
Actual CPU usage
Remaining CPU
```

---

# 20. Cluster Memory Capacity

Example:

```text
6 Nodes
×
32 GiB
=
192 GiB raw memory
```

After reservations:

```text
Allocatable
≈
less than raw capacity
```

Monitor:

```text
Memory capacity
Memory allocatable
Memory requested
Memory usage
Memory headroom
```

---

# 21. Cluster Resource Requests

Requests are particularly important for scheduling.

Example:

```text
Cluster allocatable CPU = 40 CPU
Current requests        = 35 CPU
```

Only:

```text
5 CPU
```

remains available for scheduling according to requests.

Even if actual usage is much lower, new Pods may not schedule if their requests cannot fit.

---

# 22. Cluster Resource Utilization

A cluster dashboard can show:

```text
CPU
├── Capacity
├── Allocatable
├── Requested
└── Used

Memory
├── Capacity
├── Allocatable
├── Requested
└── Used
```

This provides both:

```text
Scheduling perspective
+
Runtime perspective
```

---

# 23. Cluster Headroom

Headroom is capacity kept available for unexpected demand.

It supports:

```text
Traffic spikes
Pod rescheduling
Node failures
Rolling deployments
HPA scaling
Cluster upgrades
Maintenance
```

Example:

```text
Current utilization = 65%
```

may provide more operational flexibility than:

```text
Current utilization = 95%
```

---

# 24. Cluster Capacity Planning

Capacity planning asks:

```text
How much capacity do we need?
```

Consider:

```text
Current workloads
Growth
Peak traffic
Node failures
Deployment overhead
Autoscaling
Maintenance
Availability requirements
```

Capacity planning should not be based only on average utilization.

---

# 25. Cluster Pod Capacity

Monitor:

```text
Total Pod capacity
Current Pod count
Available Pod slots
```

Example:

```text
Maximum capacity = 1,000 Pods
Current Pods      = 850
```

Remaining:

```text
150 Pod slots
```

This can become a constraint even if CPU and memory are available.

---

# 26. Pod Density

Cluster-wide Pod density:

```text
Total Pods
──────────────
Total Nodes
```

Example:

```text
600 Pods
10 Nodes
```

Average:

```text
60 Pods per Node
```

Monitor distribution as well as averages.

---

# 27. Uneven Pod Distribution

Example:

```text
Node-1 → 100 Pods
Node-2 → 80 Pods
Node-3 → 20 Pods
Node-4 → 15 Pods
```

Possible causes:

```text
Affinity
Taints
Topology rules
Node labels
Different capacity
Pod scheduling behavior
```

Averages can hide this imbalance.

---

# 28. Cluster Scheduling Latency

Monitor how quickly Pods move from:

```text
Pending
   ↓
Scheduled
```

If scheduling latency increases:

```text
Pending duration ↑
```

investigate:

```text
Scheduler load
Cluster capacity
Scheduling constraints
API Server
Node availability
```

---

# 29. Cluster Autoscaling

A common scaling flow:

```text
Traffic ↑
   ↓
HPA
   ↓
Pods ↑
   ↓
Node capacity insufficient
   ↓
Cluster Autoscaler
   ↓
Nodes ↑
   ↓
Pods scheduled
```

Monitor:

```text
HPA desired replicas
Pending Pods
Node count
Autoscaler decisions
Node provisioning time
```

---

# 30. HPA Monitoring

Monitor:

```text
Current replicas
Desired replicas
Minimum replicas
Maximum replicas
Target metric
Actual metric
Scaling events
```

Example:

```text
Current replicas = 5
Desired replicas = 10
```

If new Pods remain Pending:

```text
Cluster capacity
```

may be the actual problem.

---

# 31. Cluster Autoscaler Monitoring

Monitor:

```text
Scale-out events
Scale-in events
Pending Pods
Node provisioning
Node termination
Unschedulable Pods
```

A scaling problem can occur at multiple layers:

```text
HPA
 ↓
Pods
 ↓
Scheduler
 ↓
Cluster Autoscaler
 ↓
Cloud infrastructure
```

---

# 32. Node Group Monitoring

In EKS, Nodes are commonly organized into Node groups.

Monitor:

```text
Node group size
Desired capacity
Minimum size
Maximum size
Healthy Nodes
Instance types
Availability Zones
```

Example:

```text
Application Node Group
Desired = 6
Min     = 3
Max     = 12
```

---

# 33. Availability Zones

Production clusters should monitor distribution across Availability Zones.

Example:

```text
Cluster
├── AZ-A → 4 Nodes
├── AZ-B → 4 Nodes
└── AZ-C → 4 Nodes
```

This improves resilience against a single-AZ failure.

---

# 34. AZ Imbalance

Example:

```text
AZ-A → 8 Nodes
AZ-B → 2 Nodes
AZ-C → 1 Node
```

This may indicate:

```text
Capacity imbalance
Scheduling constraints
Subnet capacity
Autoscaling configuration
```

Monitor Node and workload distribution by AZ.

---

# 35. Cluster Network Monitoring

Monitor:

```text
Network throughput
Packet errors
Packet drops
Connections
DNS
Service communication
Ingress traffic
Egress traffic
```

At cluster level, investigate both:

```text
Node network
+
Application network
```

---

# 36. Kubernetes DNS Monitoring

DNS is critical for Service discovery.

Typical flow:

```text
Pod
 ↓
DNS query
 ↓
CoreDNS
 ↓
Service IP
```

If DNS becomes unhealthy:

```text
Service communication
      ↓
Failures
```

Monitor:

```text
DNS request rate
DNS latency
DNS errors
CoreDNS health
```

---

# 37. CoreDNS Monitoring

CoreDNS handles DNS resolution inside Kubernetes.

Monitor:

```text
Request rate
Request latency
Error rate
CPU
Memory
Replica availability
Restarts
```

Example:

```text
CoreDNS replicas = 3
Healthy = 3
```

If:

```text
Healthy = 1
```

the cluster may have reduced DNS redundancy.

---

# 38. Cluster Storage Monitoring

Monitor:

```text
Persistent Volumes
Persistent Volume Claims
Storage Classes
Volume attachment
Volume provisioning
Capacity
Mount failures
```

Storage problems can cause Pods to remain:

```text
Pending
ContainerCreating
```

---

# 39. Persistent Volume Monitoring

Example:

```text
PVC
 ↓
Provisioning
 ↓
PV
 ↓
Pod
```

Monitor:

```text
PVC status
PV status
Provisioning failures
Volume capacity
Attachment errors
Mount errors
```

---

# 40. Storage Capacity

Cluster storage planning should consider:

```text
Current usage
Available capacity
Growth
Provisioning limits
AZ constraints
IOPS
Throughput
```

A storage system can have sufficient capacity but insufficient performance.

---

# 41. Cluster Events

Cluster events can provide useful operational context.

Query:

```bash
kubectl get events -A
```

Look for:

```text
FailedScheduling
FailedMount
BackOff
Unhealthy
Evicted
Failed
NodeNotReady
```

A sudden increase in Events can indicate a cluster-level problem.

---

# 42. Event Monitoring

Events are useful but are not a replacement for metrics.

Use:

```text
Metrics
→ Trends

Events
→ Kubernetes actions/problems

Logs
→ Detailed context
```

A complete observability system uses all three.

---

# 43. Cluster Workload Monitoring

Monitor major workload types:

```text
Deployments
StatefulSets
DaemonSets
Jobs
CronJobs
```

For each workload monitor:

```text
Desired
Ready
Available
Updated
Failed
```

---

# 44. Deployment Health

Example:

```text
Deployment
Desired = 10
Updated = 10
Available = 10
Ready = 10
```

Healthy.

Problem:

```text
Desired = 10
Updated = 10
Available = 7
Ready = 7
```

Investigate:

```text
Pods
Probes
Application
Resources
Nodes
```

---

# 45. StatefulSet Monitoring

Stateful workloads require monitoring of:

```text
Replica count
Ready replicas
Pod identity
Persistent volumes
Startup ordering
Storage health
```

A single unhealthy replica may have a larger impact than a stateless Pod failure.

---

# 46. DaemonSet Monitoring

DaemonSets usually run a Pod on each eligible Node.

Example:

```text
12 Nodes
 ↓
DaemonSet
 ↓
12 Pods expected
```

If:

```text
Desired = 12
Ready = 10
```

two Nodes may not have the required agent.

This can affect:

```text
Logging
Monitoring
Networking
Security agents
```

---

# 47. Job Monitoring

Jobs should be monitored for:

```text
Completions
Failures
Active Pods
Duration
Retries
```

A failed Job may indicate:

```text
Application failure
Configuration problem
Dependency failure
Resource issue
```

---

# 48. CronJob Monitoring

CronJobs should be monitored for:

```text
Schedule
Last successful execution
Last failed execution
Active Jobs
Missed schedules
Execution duration
```

A scheduled workload can fail silently without appropriate monitoring.

---

# 49. Namespace Monitoring

Namespaces provide logical isolation.

Monitor per namespace:

```text
CPU
Memory
Pods
Restarts
Pending Pods
Network
Storage
Workload availability
```

Example:

```text
production
├── CPU
├── Memory
├── Pods
└── Errors

staging
├── CPU
├── Memory
├── Pods
└── Errors
```

---

# 50. ResourceQuota Monitoring

ResourceQuota can limit namespace resources.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
```

Monitor:

```text
Used
Hard limit
Remaining
```

If a namespace reaches its quota:

```text
New workloads
      ↓
May fail to create/schedule
```

---

# 51. LimitRange Monitoring

LimitRange can define default or maximum resource requirements.

It affects:

```text
CPU
Memory
Requests
Limits
```

Incorrect configurations can cause workload creation failures or unexpected resource allocation.

---

# 52. Cluster API Objects

Monitor important object counts:

```text
Pods
Deployments
ReplicaSets
Services
Secrets
ConfigMaps
Jobs
CronJobs
PVCs
```

Unexpected growth can indicate:

```text
Automation problem
Controller issue
Resource leak
Failed cleanup
```

---

# 53. API Object Growth

Example:

```text
Jobs
100
 ↓
500
 ↓
2,000
```

If old Jobs are not cleaned up:

```text
Cluster object count ↑
API load ↑
Management complexity ↑
```

Use appropriate retention and cleanup policies.

---

# 54. API Server Load

Large clusters can generate significant API traffic from:

```text
Controllers
Operators
Monitoring
CI/CD
Automation
Users
```

Monitor:

```text
Request rate
Latency
Errors
Request volume by client/resource
```

Unexpected API traffic can indicate a misbehaving component.

---

# 55. etcd Monitoring

etcd stores Kubernetes cluster state.

Important signals include:

```text
Request latency
Request rate
Database size
Disk performance
Leader health
Errors
```

In managed Kubernetes services, etcd may be managed by the provider, but understanding its role is still important for troubleshooting control-plane behavior.

---

# 56. etcd Importance

Conceptually:

```text
API Server
    ↓
etcd
    ↓
Cluster state
```

If the control-plane datastore becomes unhealthy, Kubernetes control-plane operations can be affected.

Applications already running may continue temporarily, but management and reconciliation can degrade.

---

# 57. Cluster API Availability

A healthy application cluster also needs a healthy management plane.

Monitor:

```text
API availability
API latency
API error rate
```

If:

```text
kubectl get pods
```

becomes slow or fails consistently, investigate control-plane health and connectivity.

---

# 58. Cluster Monitoring Dashboard

A useful top-level dashboard:

```text
┌──────────────────────────────────────────────┐
│            KUBERNETES CLUSTER               │
├──────────────────────────────────────────────┤
│ Nodes          12       Ready       12       │
│ Pods          420       Ready      405       │
│ Pending Pods    3                            │
│ Failed Pods     2                            │
├──────────────────────────────────────────────┤
│ CPU Usage      62%       Memory      68%      │
│ CPU Requested  71%       Memory Req  74%      │
├──────────────────────────────────────────────┤
│ API Latency     80ms     API Errors   0.1%    │
│ DNS Errors       2%      DNS Latency  20ms    │
├──────────────────────────────────────────────┤
│ Node Pressure   0        Evictions     1      │
│ Storage Alerts  0        Network Alerts 0     │
└──────────────────────────────────────────────┘
```

---

# 59. Cluster Dashboard Sections

A production Grafana dashboard can have:

```text
Cluster Overview
│
├── Control Plane
├── Nodes
├── Pods
├── Workloads
├── CPU
├── Memory
├── Storage
├── Network
├── Scheduling
├── Autoscaling
├── DNS
└── Alerts
```

---

# 60. Cluster CPU Dashboard

Useful panels:

```text
Total CPU Usage
CPU Capacity
CPU Allocatable
CPU Requests
CPU Limits
CPU Headroom
CPU by Namespace
CPU by Node
CPU by Workload
```

This helps identify both capacity and hotspots.

---

# 61. Cluster Memory Dashboard

Useful panels:

```text
Total Memory Usage
Memory Capacity
Memory Allocatable
Memory Requests
Memory Limits
Memory Headroom
Memory by Namespace
Memory by Node
Top Memory-consuming Pods
```

Look for sustained growth rather than isolated spikes.

---

# 62. Cluster Storage Dashboard

Include:

```text
Total Storage
Used Storage
Available Storage
Filesystem utilization
Inodes
PVC usage
Ephemeral storage
Disk I/O
DiskPressure
```

This gives both capacity and performance visibility.

---

# 63. Cluster Network Dashboard

Include:

```text
Ingress traffic
Egress traffic
Node traffic
Packets
Errors
Drops
Connections
DNS requests
DNS errors
```

Investigate sudden changes and correlate with application traffic.

---

# 64. Cluster Scheduling Dashboard

Useful panels:

```text
Pending Pods
Scheduling failures
Scheduling latency
Unschedulable Pods
Requested CPU
Requested memory
Available capacity
Node count
```

This dashboard answers:

```text
Can the cluster place new workloads?
```

---

# 65. Cluster Autoscaling Dashboard

Include:

```text
Current Nodes
Desired Nodes
Pending Pods
Scale-out events
Scale-in events
Node provisioning time
HPA replicas
HPA desired replicas
```

This helps identify scaling bottlenecks.

---

# 66. Cluster Availability Dashboard

Track:

```text
Ready Nodes
Ready Pods
Desired replicas
Available replicas
Unavailable replicas
Failed workloads
API availability
```

A high-level availability dashboard should focus on user impact.

---

# 67. Cluster Alerts

Important alerts:

```text
Cluster Node availability reduced
High Pending Pods
High scheduling failures
Low cluster capacity
High CPU utilization
High memory utilization
DiskPressure
MemoryPressure
High API latency
High API error rate
CoreDNS failures
Storage failures
Network errors
High eviction rate
Autoscaler failure
```

---

# 68. Avoid Alert Fatigue

Do not alert on every small variation.

Bad:

```text
CPU > 70%
```

for every Node.

Better:

```text
Cluster capacity is critically low
```

or:

```text
Important workload replicas are unavailable
```

Alerts should lead to a meaningful action.

---

# 69. Symptom-Based Cluster Alerts

Good alerts focus on impact.

Examples:

```text
Ready Nodes < expected
```

```text
Pending Pods continuously increasing
```

```text
Available replicas < desired replicas
```

```text
API latency above SLO
```

```text
Cluster scheduling failures increasing
```

These are more actionable than isolated infrastructure metrics.

---

# 70. Cluster Monitoring With Prometheus

Prometheus can collect metrics from:

```text
Kubernetes components
Node Exporter
kube-state-metrics
Applications
Exporters
```

Architecture:

```text
Metrics Sources
      │
      ↓
Prometheus
      │
      ├── Queries
      ├── Rules
      └── Alerts
           │
           ↓
        Grafana
```

---

# 71. kube-state-metrics

kube-state-metrics exposes Kubernetes object state.

Useful cluster-level information includes:

```text
Node state
Pod state
Deployment state
Replica counts
DaemonSet state
StatefulSet state
Job state
Resource quotas
```

Prometheus can scrape these metrics.

---

# 72. Node Exporter

Node Exporter provides host-level information:

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
Node
 ↓
Node Exporter
 ↓
Prometheus
 ↓
Grafana
```

This complements Kubernetes object-state monitoring.

---

# 73. Metrics Server

Metrics Server is commonly used for:

```text
kubectl top
HPA resource metrics
```

Example:

```bash
kubectl top nodes
```

and:

```bash
kubectl top pods -A
```

Prometheus provides a broader observability and historical monitoring capability.

---

# 74. Cluster Logs

Centralized logging helps investigate cluster-wide problems.

Typical flow:

```text
Pods / Nodes
     ↓
Log Collector
     ↓
ELK
     ↓
Kibana
```

Use logs for:

```text
Application failures
Kubelet problems
System errors
Container runtime errors
Network problems
```

---

# 75. Cluster Tracing

Distributed tracing provides request-level visibility.

Example:

```text
Client
 ↓
Ingress
 ↓
Service
 ↓
Pod
 ↓
Database
```

OpenTelemetry can collect traces and Jaeger can provide trace visualization.

This is particularly useful when cluster infrastructure appears healthy but application latency is high.

---

# 76. Three Pillars at Cluster Level

```text
Metrics
  ↓
Prometheus / Grafana

Logs
  ↓
ELK

Traces
  ↓
OpenTelemetry / Jaeger
```

Together:

```text
Metrics
 ↓
Detect problem

Logs
 ↓
Explain problem

Traces
 ↓
Locate problem
```

---

# 77. Cluster Monitoring During Deployment

During a production deployment monitor:

```text
API health
Node capacity
Pod scheduling
Pod readiness
Replica availability
CPU
Memory
Restarts
5xx
Latency
```

Deployment flow:

```text
New version
    ↓
Pods created
    ↓
Scheduler
    ↓
Nodes
    ↓
Readiness
    ↓
Traffic
```

---

# 78. Cluster Monitoring During Rollout Failure

Example:

```text
Deployment
 ↓
New Pods
 ↓
Pods Pending
```

Check:

```text
Cluster capacity
Scheduler
Node resources
Affinity
Pod requests
```

If:

```text
Pods Running
but
Ready = False
```

check:

```text
Probes
Application
Dependencies
```

---

# 79. Cluster Monitoring During Traffic Spike

Typical flow:

```text
Traffic ↑
   ↓
Application metrics ↑
   ↓
HPA scales Pods
   ↓
Cluster capacity ↓
   ↓
Autoscaler adds Nodes
```

Monitor:

```text
Traffic
HPA
Pods
Nodes
Cluster capacity
Latency
Errors
```

---

# 80. Cluster Monitoring During Node Failure

Example:

```text
Node-1
 ↓
Failure
```

Then:

```text
Pods
 ↓
Rescheduling
 ↓
Remaining Nodes
```

Monitor:

```text
Node availability
Pending Pods
Available replicas
Remaining capacity
Autoscaler
Application SLOs
```

---

# 81. Cluster Monitoring During AZ Failure

Suppose:

```text
AZ-A
 ↓
Unavailable
```

Monitor:

```text
Nodes remaining
Pods remaining
Cross-AZ workload distribution
Cluster capacity
Pending Pods
Application availability
```

A properly designed cluster should have enough redundancy to survive expected AZ-level failures.

---

# 82. Cluster Monitoring During Kubernetes Upgrade

During an upgrade:

```text
Control Plane
      ↓
Worker Nodes
      ↓
Applications
```

Monitor:

```text
API availability
Node readiness
Pod availability
Scheduling
Evictions
Application errors
Latency
```

Do not rely only on successful upgrade commands.

---

# 83. Cluster Monitoring During Autoscaler Failure

Suppose:

```text
Pods Pending
```

but:

```text
Cluster Autoscaler
does not add Nodes
```

Investigate:

```text
Autoscaler logs
Node group limits
Cloud provider permissions
Subnet capacity
Instance availability
Scheduling constraints
```

The monitoring chain is:

```text
Pending Pods
 ↓
Autoscaler
 ↓
Node provisioning
```

---

# 84. Cluster Monitoring During Resource Exhaustion

Example:

```text
Cluster memory requests = 95%
```

Potential result:

```text
New Pods Pending
```

Monitor:

```text
Requests
Allocatable
Pending Pods
Autoscaler
Node count
```

Resource exhaustion should be detected before it causes production failures.

---

# 85. Cluster Monitoring During DNS Failure

Symptoms:

```text
Application errors
Service connectivity failures
Timeouts
```

Cluster investigation:

```text
CoreDNS health
DNS latency
DNS error rate
Pod network
Service discovery
```

If multiple unrelated workloads fail DNS resolution simultaneously, investigate DNS at cluster level.

---

# 86. Cluster Monitoring During Network Failure

Symptoms:

```text
Multiple services unreachable
Connection timeouts
Packet drops
High latency
```

Investigate:

```text
Node network
CNI
DNS
NetworkPolicy
Cloud networking
Load balancers
Security controls
```

---

# 87. Cluster Monitoring During Storage Failure

Symptoms:

```text
Pods Pending
Mount failures
PVC errors
Application I/O errors
```

Investigate:

```text
PVC
PV
StorageClass
CSI driver
Volume attachment
Storage backend
```

---

# 88. Cluster Monitoring Troubleshooting Workflow

When a cluster-wide problem occurs:

```text
1. Check cluster Nodes
2. Check Node conditions
3. Check Pending Pods
4. Check workload availability
5. Check cluster CPU
6. Check cluster memory
7. Check storage
8. Check networking
9. Check DNS
10. Check scheduling
11. Check API health
12. Check Events
13. Check autoscaling
14. Check application metrics
15. Check logs
16. Check traces
```

---

# 89. Start With Scope

First determine:

```text
One Pod?
One workload?
One Node?
Multiple Nodes?
One Namespace?
Multiple Namespaces?
Entire Cluster?
```

Example:

```text
Payment only
→ Application / Pod investigation

All workloads on Node-3
→ Node investigation

All workloads across cluster
→ Cluster investigation
```

Scope prevents unnecessary changes.

---

# 90. Cluster Troubleshooting Decision Tree

```text
Application problem
       │
       ↓
How broad?
       │
 ┌─────┼─────┐
 ↓     ↓     ↓
Pod   Node  Cluster
 │     │      │
 ↓     ↓      ↓
Logs  CPU    API
      Memory DNS
      Disk   Storage
      Net    Capacity
```

This is a practical incident-response approach.

---

# 91. Cluster Monitoring Best Practices

```text
1. Monitor cluster health continuously.
2. Monitor control-plane signals.
3. Monitor Node availability.
4. Monitor Pod availability.
5. Monitor CPU and memory capacity.
6. Monitor scheduling.
7. Monitor Pending Pods.
8. Monitor storage.
9. Monitor networking.
10. Monitor DNS.
11. Monitor autoscaling.
12. Monitor workload availability.
13. Monitor resource quotas.
14. Maintain capacity headroom.
15. Monitor across Availability Zones.
16. Centralize logs.
17. Collect distributed traces.
18. Use actionable alerts.
19. Correlate metrics, logs, and traces.
20. Monitor user-facing SLOs.
```

---

# 92. Production Cluster Dashboard

```text
                         CLUSTER
                            │
       ┌────────────────────┼────────────────────┐
       ↓                    ↓                    ↓
   CONTROL PLANE          CAPACITY            WORKLOADS
       │                    │                    │
   API Server              CPU                  Pods
   Scheduler               Memory               Deployments
   Controllers             Storage              StatefulSets
   etcd                     Network              Jobs
       │                    │                    │
       └────────────────────┼────────────────────┘
                            ↓
                        PROMETHEUS
                            ↓
                         GRAFANA
                            ↓
                          ALERTS
```

---

# 93. Cluster Monitoring Architecture With Observability

```text
                         KUBERNETES
                              │
       ┌──────────────────────┼──────────────────────┐
       ↓                      ↓                      ↓
    Metrics                 Logs                  Traces
       │                      │                      │
       ↓                      ↓                      ↓
 Prometheus                  ELK              OpenTelemetry
       │                      │                      │
       ↓                      ↓                      ↓
   Grafana                 Kibana                  Jaeger
       │                      │                      │
       └──────────────────────┼──────────────────────┘
                              ↓
                    Incident Investigation
```

---

# 94. Cluster Health Mental Model

Remember:

```text
CLUSTER
│
├── CONTROL PLANE
│   ├── API Server
│   ├── Scheduler
│   ├── Controllers
│   └── etcd
│
├── CAPACITY
│   ├── CPU
│   ├── Memory
│   ├── Storage
│   ├── Network
│   └── Pod capacity
│
├── NODES
│   ├── Ready
│   ├── Pressure
│   └── Resources
│
├── WORKLOADS
│   ├── Pods
│   ├── Deployments
│   ├── StatefulSets
│   └── Jobs
│
├── PLATFORM
│   ├── DNS
│   ├── Networking
│   └── Storage
│
└── SCALING
    ├── HPA
    └── Cluster Autoscaler
```

---

# 95. Interview Question

### How would you monitor a Kubernetes cluster?

**Answer:**

I would monitor the cluster at control-plane, Node, workload, networking, storage, and capacity levels. For control-plane behavior I would monitor API availability, latency, errors, and scheduling behavior. For Nodes I would monitor readiness, CPU, memory, disk, network, and pressure conditions. For workloads I would monitor Pod availability, restarts, resource usage, and deployment health. I would use Prometheus and Grafana for metrics and dashboards, kube-state-metrics for Kubernetes object state, ELK for centralized logs, and OpenTelemetry with Jaeger for distributed tracing.

---

# 96. Interview Question

### How do you identify whether a problem is cluster-wide or application-specific?

**Answer:**

I first determine the scope. If only one Pod or workload is affected, I investigate the application and Pod. If multiple workloads on the same Node are affected, I investigate Node resources, networking, disk, and kubelet health. If workloads across multiple Nodes and namespaces are affected, I investigate cluster-level components such as API Server, DNS, networking, storage, scheduling, and overall capacity.

---

# 97. Interview Question

### What metrics are important for Kubernetes cluster capacity planning?

**Answer:**

I monitor CPU and memory capacity, allocatable resources, resource requests, actual utilization, Pod capacity, storage capacity, network capacity, Node count, and headroom. I also consider workload growth, peak traffic, Node failures, rolling deployments, autoscaling, and maintenance requirements.

---

# 98. Interview Question

### What would you check if many Pods are Pending?

**Answer:**

I would check the Pending Pod count and inspect scheduling Events. Then I would verify available CPU and memory, Pod requests, Node capacity, Pod capacity, taints and tolerations, affinity and topology constraints, storage requirements, networking/IP capacity, and Cluster Autoscaler behavior. I would determine whether the problem is insufficient capacity or a scheduling constraint.

---

# 99. Interview Question

### How do you monitor Kubernetes DNS?

**Answer:**

I monitor CoreDNS availability, CPU, memory, request rate, latency, and DNS error rate. If multiple workloads report service-resolution failures, I check CoreDNS health, Pod networking, Service configuration, and cluster DNS connectivity. I correlate DNS metrics with application errors and latency.

---

# 100. Interview Question

### How do you monitor Kubernetes during a production deployment?

**Answer:**

I monitor Node capacity, Pod scheduling, desired versus available replicas, readiness, restart rate, probe failures, application latency, error rate, and API health. I make sure new Pods become Ready before old Pods are removed and verify application-level health rather than relying only on successful deployment completion.

---

# 101. Interview Question

### What is cluster headroom and why is it important?

**Answer:**

Cluster headroom is the spare capacity available for unexpected traffic, Pod rescheduling, Node failures, rolling deployments, autoscaling, and maintenance. Without sufficient headroom, a single Node failure or traffic spike can cause Pods to remain Pending or create resource contention.

---

# 102. Interview Question

### How would you troubleshoot high API Server latency?

**Answer:**

I would first confirm whether the latency is persistent and whether API errors are increasing. Then I would examine API request rate, control-plane health, scheduler and controller behavior, and any components generating unusually high API traffic. In a managed service such as EKS, I would also check the cloud provider's control-plane health information and relevant service events.

---

# 103. Interview Question

### How would you troubleshoot a cluster where CPU and memory look normal but Pods remain Pending?

**Answer:**

I would inspect the scheduler Events and check constraints beyond CPU and memory. I would investigate taints, tolerations, affinity, topology constraints, Pod capacity, storage requirements, available IP addresses, node selectors, and other admission or scheduling conditions. Resource utilization alone does not guarantee that a Pod can be scheduled.

---

# 104. Interview Question

### How do you monitor cluster availability?

**Answer:**

I monitor Ready Nodes, available workload replicas, Pending and Failed Pods, API availability, DNS health, storage health, and networking. I also monitor application-level error rate and latency because infrastructure can appear healthy while the application is unavailable.

---

# 105. Interview Question

### How do you use Prometheus and Grafana for cluster monitoring?

**Answer:**

I use Prometheus to collect and store metrics from Kubernetes components, kube-state-metrics, Node Exporter, and applications. I use PromQL to analyze cluster health, capacity, workload availability, and resource utilization. Grafana provides dashboards for cluster, Node, Pod, workload, networking, and storage visibility, while alerting rules identify actionable problems.

---

# 106. Production Cluster Monitoring Checklist

```text
CONTROL PLANE
[ ] API availability
[ ] API latency
[ ] API errors
[ ] Scheduler
[ ] Controller behavior
[ ] etcd health where accessible

NODES
[ ] Total Nodes
[ ] Ready Nodes
[ ] NotReady Nodes
[ ] MemoryPressure
[ ] DiskPressure
[ ] PIDPressure

CAPACITY
[ ] CPU capacity
[ ] CPU allocatable
[ ] CPU requested
[ ] CPU usage
[ ] Memory capacity
[ ] Memory allocatable
[ ] Memory requested
[ ] Memory usage
[ ] Storage capacity
[ ] Network capacity
[ ] Pod capacity
[ ] Headroom

WORKLOADS
[ ] Pending Pods
[ ] Ready Pods
[ ] Failed Pods
[ ] Deployments
[ ] StatefulSets
[ ] DaemonSets
[ ] Jobs
[ ] CronJobs

SCHEDULING
[ ] Scheduling latency
[ ] Failed scheduling
[ ] Taints
[ ] Affinity
[ ] Topology
[ ] Resource constraints

NETWORK
[ ] DNS
[ ] CoreDNS
[ ] Network traffic
[ ] Errors
[ ] Drops
[ ] IP availability

STORAGE
[ ] PV
[ ] PVC
[ ] StorageClass
[ ] Volume provisioning
[ ] Mount failures
[ ] Capacity

SCALING
[ ] HPA
[ ] Cluster Autoscaler
[ ] Node groups
[ ] Pending Pods
[ ] Scale-out
[ ] Scale-in

OBSERVABILITY
[ ] Prometheus
[ ] Grafana
[ ] kube-state-metrics
[ ] Node Exporter
[ ] ELK
[ ] OpenTelemetry
[ ] Jaeger
[ ] Alerts
```

---

# 107. Final Mental Model

```text
                       KUBERNETES CLUSTER
                               │
          ┌────────────────────┼────────────────────┐
          ↓                    ↓                    ↓
      CONTROL PLANE         CAPACITY             WORKLOADS
          │                    │                    │
      API Server              CPU                  Pods
      Scheduler               Memory               Deployments
      Controllers             Storage              StatefulSets
      etcd                    Network              Jobs
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ↓
                         PLATFORM HEALTH
                               │
                    ┌──────────┼──────────┐
                    ↓          ↓          ↓
                   DNS      Network     Storage
                    │          │          │
                    └──────────┼──────────┘
                               ↓
                          OBSERVABILITY
                               │
              ┌────────────────┼────────────────┐
              ↓                ↓                ↓
           Metrics            Logs             Traces
              ↓                ↓                ↓
         Prometheus            ELK         OpenTelemetry
              ↓                                 ↓
           Grafana                            Jaeger
              │
              ↓
            Alerts
```

**The key principle is:** Cluster monitoring provides the highest-level view of Kubernetes health. It combines control-plane health, Node availability, workload state, resource capacity, scheduling, networking, DNS, storage, and autoscaling. Prometheus and Grafana provide the central metrics and visualization layer, while kube-state-metrics and Node Exporter provide Kubernetes-state and host-level signals. ELK adds centralized logs, and OpenTelemetry with Jaeger adds distributed tracing. Effective cluster monitoring does not focus only on infrastructure utilization—it connects infrastructure conditions to workload availability and user-facing application health.**
