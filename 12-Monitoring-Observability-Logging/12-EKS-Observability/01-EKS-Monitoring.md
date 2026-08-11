# EKS Monitoring

## 1. Overview

Amazon Elastic Kubernetes Service (EKS) monitoring focuses on the health, performance, capacity, and availability of the Kubernetes workloads and infrastructure running on AWS.

A production EKS monitoring architecture should provide visibility into:

```text
EKS Cluster
│
├── Control Plane
├── Worker Nodes
├── Pods
├── Deployments
├── Services
├── Load Balancers
├── Networking
├── Storage
├── Autoscaling
└── Applications
```

The objective is to answer:

```text
Is the EKS cluster healthy?
Are Nodes available?
Are Pods running?
Can new Pods be scheduled?
Does the cluster have enough capacity?
Are applications healthy?
Is networking working?
Is autoscaling working?
```

---

# 2. EKS Monitoring Architecture

A typical monitoring architecture is:

```text
                         AWS
                          │
                          ↓
                     EKS Cluster
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
   Control Plane        Nodes              Pods
       │                  │                  │
       ↓                  ↓                  ↓
  EKS Metrics        Node Metrics      Application Metrics
                          │                  │
                          └────────┬─────────┘
                                   ↓
                              Prometheus
                                   ↓
                                Grafana
                                   ↓
                                 Alerts
```

Kubernetes state can be collected using:

```text
kube-state-metrics
        ↓
   Prometheus
```

---

# 3. What Should Be Monitored in EKS?

A production EKS environment should be monitored at multiple levels:

```text
EKS Monitoring
│
├── Control Plane
├── Nodes
├── Pods
├── Containers
├── Kubernetes Objects
├── Networking
├── Storage
├── Load Balancers
├── Autoscaling
├── Applications
└── AWS Infrastructure
```

This layered approach helps determine where an incident is occurring.

---

# 4. Control Plane Monitoring

The EKS control plane includes Kubernetes components such as:

```text
API Server
Scheduler
Controller Manager
etcd
```

AWS manages the EKS control plane.

However, engineers still need visibility into control-plane behavior.

Important areas include:

```text
API availability
API request rate
API latency
API errors
Scheduler behavior
Cluster events
```

---

# 5. EKS API Server

The Kubernetes API Server is the primary interface to the cluster.

Commands such as:

```bash
kubectl get pods
```

communicate with the API Server.

Architecture:

```text
kubectl
   │
   ↓
EKS API Server
   │
   ├── Kubernetes Objects
   ├── Controllers
   └── Scheduler
```

If API Server performance degrades, many Kubernetes operations can be affected.

---

# 6. API Server Monitoring

Monitor:

```text
Request rate
Request latency
Request errors
HTTP status codes
Rejected requests
API availability
```

Example problem:

```text
kubectl commands
       ↓
Slow response
       ↓
API latency investigation
```

High API latency can affect:

```text
Deployments
Autoscaling
Controllers
CI/CD
Monitoring
kubectl operations
```

---

# 7. EKS Nodes

Worker Nodes provide compute capacity for workloads.

A typical EKS architecture:

```text
EKS Cluster
│
├── Node Group
│   ├── Node-1
│   ├── Node-2
│   └── Node-3
│
└── Pods
```

Monitor:

```text
Node Ready status
CPU
Memory
Disk
Network
Pod capacity
Kubelet
Container runtime
Pressure conditions
```

---

# 8. Node Ready Status

Check:

```bash
kubectl get nodes
```

Example:

```text
NAME       STATUS   ROLES
node-1     Ready    <none>
node-2     Ready    <none>
node-3     Ready    <none>
```

A Node that becomes:

```text
NotReady
```

requires investigation.

---

# 9. Node Conditions

Important Node conditions include:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

Check:

```bash
kubectl describe node <node-name>
```

Example:

```text
Ready            True
MemoryPressure   False
DiskPressure     False
PIDPressure      False
```

---

# 10. Node CPU Monitoring

Monitor:

```text
CPU capacity
CPU allocatable
CPU requests
CPU limits
CPU usage
CPU saturation
CPU throttling
```

Example:

```text
Node CPU capacity = 8 CPU
CPU usage         = 6 CPU
```

Utilization:

```text
6 / 8 × 100 = 75%
```

Sustained high CPU usage should be investigated together with application latency and workload behavior.

---

# 11. Node Memory Monitoring

Monitor:

```text
Memory capacity
Memory allocatable
Memory requests
Memory limits
Memory usage
Memory pressure
OOM events
```

Example:

```text
Node memory = 32 GiB
Used        = 26 GiB
```

Also account for:

```text
Operating system
kubelet
Container runtime
System agents
Monitoring agents
```

---

# 12. Node Disk Monitoring

Monitor:

```text
Disk utilization
Available space
Filesystem usage
Inodes
Container images
Container logs
Ephemeral storage
Disk I/O
```

High disk usage can result in:

```text
DiskPressure
Pod eviction
Container failures
Image pull failures
```

---

# 13. Node Network Monitoring

Monitor:

```text
Network throughput
Packets
Errors
Drops
Connections
Network interface utilization
```

In EKS, also consider:

```text
AWS VPC networking
Elastic Network Interfaces
VPC CNI
Subnet IP availability
```

---

# 14. EKS Pod Monitoring

Pods are the primary execution units for applications.

Monitor:

```text
Pod status
Ready state
Restarts
CPU
Memory
Network
Probe failures
OOMKilled
Pending state
Evictions
```

Useful command:

```bash
kubectl get pods -A
```

---

# 15. Pod Status

Common states include:

```text
Running
Pending
Succeeded
Failed
Unknown
```

Monitor abnormal states such as:

```text
Pending
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
OOMKilled
```

---

# 16. Pod Restarts

A high restart count may indicate:

```text
Application crash
OOMKilled
Failed probes
Configuration problems
Dependency failures
```

Example:

```text
NAME       READY   STATUS    RESTARTS
payment    1/1     Running   25
```

The Pod may currently be Running but still have a serious stability problem.

---

# 17. Container CPU Monitoring

Monitor CPU usage by:

```text
Namespace
Deployment
Pod
Container
Node
```

Example:

```text
Payment
 ├── Pod-1 → 700m
 ├── Pod-2 → 650m
 └── Pod-3 → 680m
```

This helps identify resource-intensive workloads.

---

# 18. Container Memory Monitoring

Monitor:

```text
Current memory
Memory request
Memory limit
Memory growth
OOMKilled events
```

A useful investigation pattern:

```text
Memory usage ↑
      ↓
Limit reached
      ↓
OOMKilled
      ↓
Container restart
```

---

# 19. Kubernetes Object Monitoring

Monitor:

```text
Deployments
ReplicaSets
StatefulSets
DaemonSets
Jobs
CronJobs
Services
Ingress
ConfigMaps
Secrets
PVCs
```

For workloads, compare:

```text
Desired
Current
Ready
Available
Updated
```

---

# 20. Deployment Monitoring

Example:

```text
Desired replicas = 5
Updated replicas = 5
Available         = 5
Ready             = 5
```

Healthy.

Problem:

```text
Desired replicas = 5
Available         = 3
Ready             = 3
```

Investigate:

```text
Pods
Events
Readiness probes
Resources
Nodes
Application logs
```

---

# 21. StatefulSet Monitoring

For StatefulSets monitor:

```text
Desired replicas
Ready replicas
Pod identity
Persistent volumes
Storage health
Startup failures
```

A StatefulSet failure can affect stateful application availability and data access.

---

# 22. DaemonSet Monitoring

DaemonSets typically run on every eligible Node.

Example:

```text
EKS Nodes = 10
DaemonSet desired = 10
DaemonSet ready = 10
```

If:

```text
Desired = 10
Ready = 8
```

investigate the two Nodes that do not have healthy DaemonSet Pods.

This is particularly important for:

```text
Logging agents
Monitoring agents
Security agents
Networking components
```

---

# 23. Kubernetes Events

Events provide important troubleshooting context.

Check:

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
NodeNotReady
Failed
```

Events are especially useful when a Pod is Pending or repeatedly failing.

---

# 24. EKS Cluster Capacity

Monitor cluster-wide:

```text
CPU capacity
CPU allocatable
CPU requests
CPU usage
Memory capacity
Memory allocatable
Memory requests
Memory usage
Pod capacity
Storage
Network capacity
```

Example:

```text
Cluster CPU allocatable = 40 CPU
CPU requested           = 34 CPU
```

Only about:

```text
6 CPU
```

remains available for scheduling based on requests.

---

# 25. Cluster Headroom

Headroom is spare capacity available for:

```text
Traffic spikes
Node failures
Pod rescheduling
Rolling deployments
HPA scaling
Cluster upgrades
Maintenance
```

Avoid operating a production cluster continuously at near-total capacity.

---

# 26. EKS Node Groups

Monitor Node Groups for:

```text
Desired capacity
Minimum capacity
Maximum capacity
Current Nodes
Healthy Nodes
Instance types
Availability Zones
Scaling events
```

Example:

```text
Application Node Group

Min     = 3
Desired = 6
Max     = 12
```

---

# 27. EKS Managed Node Groups

With Managed Node Groups, AWS manages parts of the Node lifecycle.

Monitor:

```text
Node health
Node group status
Scaling
Instance availability
Upgrade state
```

Even though AWS manages the infrastructure lifecycle, Kubernetes workload health still needs to be monitored.

---

# 28. EKS Auto Scaling

A common architecture:

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

Monitor every stage.

---

# 29. HPA Monitoring

Monitor:

```text
Current replicas
Desired replicas
Minimum replicas
Maximum replicas
Target metric
Current metric
Scaling events
```

Example:

```text
Current replicas = 4
Desired replicas = 8
```

If the additional Pods remain Pending, investigate cluster capacity.

---

# 30. Cluster Autoscaler Monitoring

Monitor:

```text
Pending Pods
Unschedulable Pods
Node count
Scale-out events
Scale-in events
Node provisioning time
Node group limits
```

A typical problem:

```text
Pods Pending
     ↓
Cluster Autoscaler should scale
     ↓
No new Node
```

Investigate autoscaler configuration and AWS infrastructure constraints.

---

# 31. EKS Networking Monitoring

EKS networking commonly involves:

```text
VPC
Subnets
Security Groups
Route Tables
VPC CNI
ENIs
Pod IPs
Load Balancers
```

Monitor:

```text
Pod connectivity
Network errors
Packet drops
IP availability
CNI health
Load Balancer health
```

---

# 32. AWS VPC CNI

The AWS VPC CNI provides networking for Pods.

Conceptually:

```text
Pod
 ↓
VPC CNI
 ↓
ENI / IP
 ↓
VPC
```

Monitor:

```text
CNI health
IP allocation
ENI capacity
Subnet IP availability
Pod networking errors
```

---

# 33. Pod IP Exhaustion

A cluster may have sufficient:

```text
CPU
Memory
```

but still fail to create new Pods because there are not enough IP addresses.

Example:

```text
Pod Pending
     ↓
CPU available
Memory available
     ↓
No available Pod IP
```

In EKS, investigate:

```text
Subnet available IPs
VPC CNI
ENI allocation
Node Pod limits
```

---

# 34. Subnet Monitoring

Monitor important subnet conditions:

```text
Available IP addresses
Subnet utilization
Availability Zone
Route configuration
```

A nearly exhausted subnet can affect:

```text
Node provisioning
Pod networking
Load Balancer resources
```

---

# 35. EKS Load Balancer Monitoring

Applications exposed externally may use:

```text
Application Load Balancer
Network Load Balancer
```

Monitor:

```text
Healthy targets
Unhealthy targets
Request count
Latency
HTTP errors
Connection errors
```

The load balancer layer should be correlated with Kubernetes Service/Ingress health.

---

# 36. Load Balancer Troubleshooting

Example:

```text
Users receive 503
```

Check:

```text
Load Balancer
 ↓
Target health
 ↓
Kubernetes Service
 ↓
Endpoints
 ↓
Pods
 ↓
Readiness
```

A healthy Load Balancer does not guarantee healthy backend Pods.

---

# 37. Service Monitoring

Monitor:

```text
Service existence
Endpoints
EndpointSlices
Port configuration
Selector configuration
```

A Service with no healthy endpoints can result in application connectivity failures.

Conceptually:

```text
Service
   ↓
Endpoints
   ↓
Ready Pods
```

---

# 38. Ingress Monitoring

For ALB-based ingress:

```text
Internet
   ↓
ALB
   ↓
Ingress
   ↓
Service
   ↓
Pods
```

Monitor each layer:

```text
ALB health
Ingress configuration
Service endpoints
Pod readiness
Application health
```

---

# 39. EKS Storage Monitoring

Monitor:

```text
PersistentVolume
PersistentVolumeClaim
StorageClass
CSI driver
Volume attachment
Mount failures
Storage capacity
```

Example:

```text
Pod
 ↓
PVC
 ↓
PV
 ↓
Storage backend
```

A failure anywhere in this chain can prevent a workload from starting.

---

# 40. EBS Volume Monitoring

When EBS-backed storage is used, monitor relevant infrastructure and workload signals such as:

```text
Volume availability
IOPS
Throughput
Latency
Volume attachment
Filesystem utilization
```

Correlate storage performance with application latency.

---

# 41. EKS DNS Monitoring

CoreDNS provides cluster DNS.

Typical flow:

```text
Pod
 ↓
CoreDNS
 ↓
Service name
 ↓
Service IP
```

Monitor:

```text
CoreDNS replicas
CPU
Memory
Request rate
Latency
Errors
Restarts
```

---

# 42. CoreDNS Failure

Symptoms can include:

```text
Service discovery failures
Connection timeouts
Application errors
Dependency failures
```

Check:

```bash
kubectl get pods -n kube-system
```

Then inspect CoreDNS Pods and Events.

---

# 43. EKS Monitoring With Prometheus

A common architecture:

```text
EKS
│
├── Node Exporter
├── kube-state-metrics
├── kubelet
└── Applications
        │
        ↓
    Prometheus
        │
        ↓
     Grafana
```

Prometheus provides:

```text
Metrics collection
Time-series storage
PromQL queries
Alerting rules
```

---

# 44. kube-state-metrics in EKS

Use kube-state-metrics to monitor Kubernetes object state.

Examples:

```text
Node status
Pod status
Deployment replicas
DaemonSet replicas
StatefulSet replicas
Job status
PVC status
Resource quotas
```

It complements infrastructure metrics.

---

# 45. Node Exporter in EKS

Node Exporter provides host-level metrics:

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
   ↓
Node Exporter
   ↓
Prometheus
   ↓
Grafana
```

---

# 46. Metrics Server vs Prometheus

Metrics Server is commonly used for:

```text
kubectl top
HPA resource metrics
```

Prometheus is commonly used for:

```text
Historical monitoring
Dashboards
Alerting
Detailed queries
Infrastructure metrics
Application metrics
```

They serve different purposes.

---

# 47. EKS Grafana Dashboard

A useful EKS dashboard can include:

```text
Cluster Overview
│
├── Node Count
├── Ready Nodes
├── Pod Count
├── Pending Pods
├── CPU
├── Memory
├── Disk
├── Network
├── API Health
├── HPA
├── Autoscaler
├── DNS
├── Storage
└── Alerts
```

---

# 48. Cluster Overview Dashboard

Example:

```text
┌───────────────────────────────────────────┐
│              EKS CLUSTER                 │
├───────────────────────────────────────────┤
│ Nodes              12                    │
│ Ready Nodes        12                    │
│ Pods               420                   │
│ Pending Pods         2                   │
├───────────────────────────────────────────┤
│ CPU Usage           62%                   │
│ Memory Usage        68%                   │
│ Disk Usage          55%                   │
├───────────────────────────────────────────┤
│ HPA Desired         25                    │
│ Current Pods        23                    │
│ Alerts                1                   │
└───────────────────────────────────────────┘
```

---

# 49. EKS Monitoring Alerts

Useful alerts include:

```text
NodeNotReady
MemoryPressure
DiskPressure
High CPU
High memory
High disk usage
High restart count
Pending Pods
Failed Pods
Deployment unavailable
High API latency
High API error rate
CoreDNS failures
High network errors
Low subnet IP availability
Load Balancer unhealthy targets
PVC failures
Autoscaler failure
```

---

# 50. Avoid Alert Fatigue

Not every metric requires an alert.

Bad example:

```text
CPU > 70%
```

on every Node for a short period.

Better:

```text
Cluster capacity critically low
```

or:

```text
Production workload replicas unavailable
```

Alerts should represent actionable problems.

---

# 51. EKS Monitoring During Deployment

During a production deployment monitor:

```text
Deployment replicas
Pod readiness
Pod restarts
CPU
Memory
Node capacity
Pending Pods
Application latency
Application errors
Load Balancer targets
```

Deployment flow:

```text
New Image
   ↓
Pod Created
   ↓
Scheduled
   ↓
Container Started
   ↓
Readiness Passed
   ↓
Traffic
```

---

# 52. EKS Monitoring During Rolling Update

Monitor:

```text
Old replicas
New replicas
Available replicas
Unavailable replicas
Pending Pods
Readiness failures
Node capacity
Application errors
```

A successful Kubernetes rollout command does not automatically mean users are receiving healthy responses.

---

# 53. EKS Monitoring During Traffic Spike

Example:

```text
Traffic ↑
   ↓
Application CPU ↑
   ↓
HPA scales Pods
   ↓
Cluster capacity decreases
   ↓
Autoscaler adds Nodes
```

Monitor:

```text
Request rate
CPU
Memory
HPA
Pending Pods
Node count
Latency
Errors
```

---

# 54. EKS Monitoring During Node Failure

Suppose:

```text
Node-2
   ↓
Failure
```

Monitor:

```text
Node count
Ready Nodes
Pod eviction
Pending Pods
Remaining capacity
Autoscaler
Application availability
```

The key question is:

```text
Can the remaining cluster capacity handle the workloads?
```

---

# 55. EKS Monitoring During AZ Failure

Example:

```text
AZ-A
├── Node-1
├── Node-2
└── Node-3
```

If AZ-A becomes unavailable:

```text
Remaining Nodes
       ↓
Remaining capacity
       ↓
Pod rescheduling
```

Monitor:

```text
Node distribution
Pod distribution
Remaining capacity
Load Balancer health
Application availability
```

---

# 56. EKS Monitoring During Upgrade

During an EKS upgrade monitor:

```text
Control-plane health
Node health
Pod availability
Pending Pods
Evictions
Scheduling
Application errors
Latency
```

For Node upgrades:

```text
Cordon
 ↓
Drain
 ↓
Replace
 ↓
Ready
 ↓
Workload recovery
```

---

# 57. EKS Monitoring and AWS Metrics

EKS monitoring should also consider AWS infrastructure.

Examples include:

```text
EC2
EBS
Load Balancers
VPC
Network interfaces
Subnets
```

This creates a complete view:

```text
AWS Infrastructure
       ↓
EKS Infrastructure
       ↓
Kubernetes
       ↓
Applications
```

---

# 58. CloudWatch and EKS

AWS CloudWatch can provide AWS service and infrastructure telemetry.

For example:

```text
EC2
EBS
ALB/NLB
AWS infrastructure
EKS-related AWS metrics
```

Prometheus can provide Kubernetes and application-level metrics.

A production environment may use both depending on monitoring requirements.

---

# 59. Prometheus + AWS Monitoring

A common architecture can combine:

```text
AWS Metrics
   ↓
CloudWatch

Kubernetes Metrics
   ↓
Prometheus

Visualization
   ↓
Grafana
```

This gives visibility across AWS and Kubernetes.

---

# 60. EKS Observability Layers

Think of EKS monitoring as:

```text
Layer 1
AWS Infrastructure
│
├── EC2
├── EBS
├── VPC
└── Load Balancer

Layer 2
EKS Infrastructure
│
├── Control Plane
├── Node Groups
└── Nodes

Layer 3
Kubernetes
│
├── Pods
├── Deployments
├── Services
└── Ingress

Layer 4
Application
│
├── Metrics
├── Logs
└── Traces
```

---

# 61. EKS Monitoring Mental Model

```text
AWS
 ↓
EKS
 ↓
Nodes
 ↓
Pods
 ↓
Containers
 ↓
Applications
```

Monitor every layer.

If users report an issue:

```text
User
 ↓
Load Balancer
 ↓
Ingress
 ↓
Service
 ↓
Pod
 ↓
Node
 ↓
AWS Infrastructure
```

Trace the problem through this path.

---

# 62. EKS Incident Troubleshooting Workflow

When an EKS application is unhealthy:

```text
1. Check application symptoms
2. Check Load Balancer
3. Check Ingress
4. Check Service
5. Check Pod readiness
6. Check Pod logs
7. Check Pod resources
8. Check Node health
9. Check cluster capacity
10. Check networking
11. Check DNS
12. Check storage
13. Check autoscaling
14. Check AWS infrastructure
15. Correlate metrics and events
```

---

# 63. Example: EKS 503 Error

User receives:

```text
503 Service Unavailable
```

Investigation:

```text
ALB
 ↓
Unhealthy targets
 ↓
Service
 ↓
No Ready endpoints
 ↓
Pods
 ↓
Readiness probe failure
```

Then inspect:

```text
Pod logs
Application metrics
Dependencies
```

This gives a complete troubleshooting chain.

---

# 64. Example: EKS Pod Pending

Problem:

```text
Pod Pending
```

Check:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
FailedScheduling
```

Then investigate:

```text
CPU
Memory
Node capacity
Taints
Affinity
Topology
Pod capacity
IP availability
Autoscaler
```

---

# 65. Example: EKS Node NotReady

Problem:

```text
Node-3 NotReady
```

Run:

```bash
kubectl describe node <node-name>
```

Check:

```text
Conditions
Events
Kubelet
MemoryPressure
DiskPressure
Network
Container runtime
```

Then determine whether affected Pods were rescheduled successfully.

---

# 66. Example: EKS High Memory

Problem:

```text
Node memory = 95%
```

Investigate:

```text
Top Pods
 ↓
Pod memory usage
 ↓
Memory limits
 ↓
OOMKilled
 ↓
Application behavior
```

If workload demand genuinely increased:

```text
HPA / Cluster Autoscaler
```

may be required.

---

# 67. Example: EKS IP Exhaustion

Problem:

```text
Pods Pending
```

CPU:

```text
Available
```

Memory:

```text
Available
```

Events indicate networking/IP limitations.

Investigate:

```text
Subnet available IPs
VPC CNI
ENIs
Node Pod limits
```

This is an important EKS-specific troubleshooting scenario.

---

# 68. Example: EKS DNS Problem

Symptoms:

```text
Multiple applications cannot reach Services
```

Check:

```text
CoreDNS
 ↓
DNS request rate
 ↓
DNS errors
 ↓
Pod networking
```

If CoreDNS is unhealthy, application connectivity can degrade across the cluster.

---

# 69. Example: EKS Storage Problem

Symptoms:

```text
Pod stuck in ContainerCreating
```

Check Events:

```text
FailedMount
```

Then investigate:

```text
PVC
PV
CSI driver
Volume attachment
EBS
Availability Zone
```

---

# 70. EKS Monitoring Best Practices

```text
1. Monitor control-plane health.
2. Monitor Node readiness.
3. Monitor CPU and memory.
4. Monitor disk and filesystem.
5. Monitor network and IP capacity.
6. Monitor Pod status.
7. Monitor workload availability.
8. Monitor Kubernetes Events.
9. Monitor HPA.
10. Monitor Cluster Autoscaler.
11. Monitor Node Groups.
12. Monitor Load Balancers.
13. Monitor CoreDNS.
14. Monitor persistent storage.
15. Monitor AWS infrastructure.
16. Maintain cluster headroom.
17. Distribute workloads across Availability Zones.
18. Use Prometheus and Grafana for Kubernetes metrics.
19. Correlate metrics with application behavior.
20. Alert on actionable production conditions.
```

---

# 71. Production EKS Monitoring Architecture

```text
                              AWS
                               │
        ┌──────────────────────┼──────────────────────┐
        ↓                      ↓                      ↓
      VPC                     EC2                  Load Balancer
        │                      │                      │
        │                      ↓                      │
        │                   EKS Nodes                 │
        │                      │                      │
        │              ┌───────┼───────┐              │
        │              ↓       ↓       ↓              │
        │             Pods    Pods    Pods            │
        │              │       │       │              │
        │              └───────┼───────┘              │
        │                      ↓                      │
        │               Kubernetes Metrics            │
        │                      │                      │
        │          ┌───────────┴───────────┐          │
        │          ↓                       ↓          │
        │     kube-state-metrics      Node Exporter   │
        │          │                       │          │
        │          └───────────┬───────────┘          │
        │                      ↓                      │
        │                 Prometheus                  │
        │                      ↓                      │
        │                   Grafana                   │
        │                      ↓                      │
        └─────────────────── Alerts ──────────────────┘
```

---

# 72. EKS Monitoring With Application Observability

```text
                    EKS
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    Metrics         Logs         Traces
       │             │             │
       ↓             ↓             ↓
 Prometheus          ELK       OpenTelemetry
       │             │             │
       ↓             ↓             ↓
    Grafana         Kibana       Jaeger
       │             │             │
       └─────────────┼─────────────┘
                     ↓
              Root Cause Analysis
```

This allows infrastructure and application signals to be investigated together.

---

# 73. Interview Question

### How do you monitor an EKS cluster?

**Answer:**

I monitor EKS at multiple layers: control plane, Nodes, Pods, Kubernetes workloads, networking, storage, autoscaling, and AWS infrastructure. For Kubernetes metrics I use Prometheus with kube-state-metrics and Node Exporter, and Grafana for dashboards and alerting. I also monitor EKS networking, VPC CNI, subnet IP availability, Load Balancer health, CoreDNS, Node Groups, and cluster capacity. For application troubleshooting I correlate metrics with centralized logs and distributed traces.

---

# 74. Interview Question

### How would you troubleshoot a Pod that is Pending in EKS?

**Answer:**

I would run `kubectl describe pod` and inspect the Events, especially `FailedScheduling`. Then I would check CPU and memory requests, Node allocatable capacity, taints, tolerations, affinity, topology constraints, Pod capacity, storage requirements, and EKS-specific networking/IP capacity. I would also check whether Cluster Autoscaler is able to add capacity.

---

# 75. Interview Question

### How would you troubleshoot an EKS Node that becomes NotReady?

**Answer:**

I would check `kubectl describe node` and inspect Node conditions and Events. Then I would investigate kubelet health, container runtime, CPU and memory pressure, disk usage, network connectivity, and the underlying EC2 instance. I would also verify whether Pods were successfully rescheduled onto other Nodes and whether sufficient cluster capacity remains.

---

# 76. Interview Question

### How do you monitor EKS autoscaling?

**Answer:**

I monitor HPA desired versus current replicas, Pending Pods, Node capacity, Cluster Autoscaler decisions, Node Group desired and current capacity, scale-out and scale-in events, and Node provisioning time. I correlate these metrics with application traffic and latency to verify that scaling is responding appropriately.

---

# 77. Interview Question

### What is an EKS-specific issue that CPU and memory monitoring may not detect?

**Answer:**

Pod IP exhaustion is an important example. A cluster can have sufficient CPU and memory but still be unable to schedule new Pods because there are not enough available IP addresses. In EKS I would investigate the AWS VPC CNI, subnet available IPs, ENI allocation, and Node Pod limits.

---

# 78. Interview Question

### How would you troubleshoot an EKS 503 error?

**Answer:**

I would trace the request path from the Load Balancer to the Ingress, Service, endpoints, and Pods. I would check Load Balancer target health, Service endpoints, Pod readiness, readiness probe failures, application logs, application metrics, and dependencies. If the infrastructure looks healthy, I would use tracing to identify where the request is failing or becoming slow.

---

# 79. Interview Question

### How do you monitor EKS Node capacity?

**Answer:**

I compare Node capacity and allocatable CPU and memory against requests and actual usage. I also monitor Pod capacity, disk, network, and IP availability. At the cluster level I monitor total capacity, headroom, Pending Pods, Node Group limits, and autoscaling behavior.

---

# 80. Interview Question

### Why do you need both Prometheus and AWS monitoring for EKS?

**Answer:**

They provide different perspectives. Prometheus is well suited for Kubernetes and application metrics such as Pods, Nodes, workloads, and custom application metrics. AWS monitoring provides visibility into AWS infrastructure and services such as EC2, EBS, Load Balancers, and VPC-related resources. Combining them provides end-to-end visibility from AWS infrastructure through Kubernetes to the application.

---

# 81. EKS Monitoring Checklist

```text
CONTROL PLANE
[ ] API availability
[ ] API latency
[ ] API errors
[ ] Scheduler
[ ] Controller behavior

NODES
[ ] Node count
[ ] Ready Nodes
[ ] NotReady Nodes
[ ] CPU
[ ] Memory
[ ] Disk
[ ] Network
[ ] Kubelet
[ ] Runtime
[ ] Pressure conditions

PODS
[ ] Running
[ ] Pending
[ ] Failed
[ ] Restarts
[ ] OOMKilled
[ ] Readiness
[ ] Liveness
[ ] CPU
[ ] Memory

WORKLOADS
[ ] Deployments
[ ] StatefulSets
[ ] DaemonSets
[ ] Jobs
[ ] CronJobs

NETWORK
[ ] VPC CNI
[ ] Pod IP availability
[ ] Subnet IPs
[ ] DNS
[ ] CoreDNS
[ ] Services
[ ] Ingress
[ ] Load Balancer

STORAGE
[ ] PV
[ ] PVC
[ ] CSI
[ ] EBS
[ ] Mount failures
[ ] Capacity

SCALING
[ ] HPA
[ ] Cluster Autoscaler
[ ] Node Groups
[ ] Desired capacity
[ ] Current capacity
[ ] Pending Pods

AWS
[ ] EC2
[ ] EBS
[ ] Load Balancers
[ ] VPC
[ ] Subnets
[ ] ENIs

OBSERVABILITY
[ ] Prometheus
[ ] Grafana
[ ] kube-state-metrics
[ ] Node Exporter
[ ] Alerts
[ ] Logs
[ ] Traces
```

---

# 82. Final EKS Monitoring Mental Model

```text
                         AWS
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
      VPC                 EC2             Load Balancer
       │                  │                  │
       │                  ↓                  │
       │              EKS Nodes              │
       │                  │                  │
       │             ┌────┼────┐             │
       │             ↓    ↓    ↓             │
       │            Pods Pods Pods            │
       │             │    │    │             │
       └─────────────┼────┼────┼─────────────┘
                     ↓
             Kubernetes Metrics
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
   kube-state-metrics      Node Exporter
          │                     │
          └──────────┬──────────┘
                     ↓
                Prometheus
                     ↓
                  Grafana
                     ↓
                   Alerts
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
         Logs                 Traces
          ↓                     ↓
         ELK              OpenTelemetry
                                ↓
                              Jaeger
```

**Key principle:** EKS monitoring must cover both the AWS infrastructure and the Kubernetes platform running on top of it. Monitor the control plane, Nodes, Pods, workloads, networking, storage, autoscaling, Load Balancers, VPC CNI, DNS, and application health. Prometheus and Grafana provide the core Kubernetes monitoring layer, while AWS monitoring provides infrastructure visibility. When an incident occurs, follow the request and infrastructure path from **Load Balancer → Ingress → Service → Pod → Node → AWS infrastructure** and correlate metrics, Kubernetes Events, logs, and traces to identify the root cause.
