# Node Monitoring

## 1. Overview

Node monitoring focuses on the health, resource utilization, capacity, pressure conditions, and workload distribution of Kubernetes worker Nodes.

A Kubernetes Node provides the compute resources required to run Pods.

```text
Kubernetes Cluster
│
├── Node-1
│   ├── Pod
│   ├── Pod
│   └── Pod
│
├── Node-2
│   ├── Pod
│   └── Pod
│
└── Node-3
    ├── Pod
    └── Pod
```

If a Node becomes unhealthy, multiple Pods can be affected simultaneously.

Therefore Node monitoring is critical for Kubernetes reliability.

---

# 2. Why Node Monitoring Is Important

A single Node can host many application Pods.

Example:

```text
Node-1
│
├── Orders
├── Payment
├── Inventory
├── Notification
└── Frontend
```

If Node-1 fails:

```text
Node-1
   ↓
Node failure
   ↓
Multiple Pods affected
```

Possible consequences:

```text
Application downtime
Pod eviction
Traffic reduction
Scheduling pressure
Resource contention
```

Node monitoring helps identify these conditions before they become larger incidents.

---

# 3. Node Monitoring Architecture

A typical monitoring architecture is:

```text
                         Kubernetes
                              │
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
          Node-1           Node-2           Node-3
             │                │                │
        Node Metrics     Node Metrics     Node Metrics
             │                │                │
             └────────────────┼────────────────┘
                              ↓
                         Prometheus
                              ↓
                           Grafana
                              ↓
                            Alerts
```

Additional components can provide Kubernetes object state:

```text
kube-state-metrics
        ↓
    Prometheus
```

---

# 4. Node Health Dimensions

A production Node should be monitored across:

```text
Node Health
│
├── Ready condition
├── CPU
├── Memory
├── Disk
├── Network
├── Filesystem
├── PID usage
├── Kubelet
├── Container runtime
├── Pod capacity
├── Allocatable resources
├── Pressure conditions
└── Workload distribution
```

---

# 5. Node Ready Condition

The Node Ready condition is one of the most important health indicators.

Check:

```bash
kubectl get nodes
```

Example:

```text
NAME      STATUS   ROLES
node-1    Ready    <none>
node-2    Ready    <none>
node-3    Ready    <none>
```

A healthy production cluster should have the expected Nodes in the Ready state.

---

# 6. NotReady Node

If a Node becomes:

```text
NotReady
```

investigate immediately.

Possible causes:

```text
Kubelet failure
Network failure
Node resource pressure
Container runtime failure
Operating system problem
Disk problem
Cloud infrastructure issue
```

Check:

```bash
kubectl describe node <node-name>
```

---

# 7. Node Conditions

Kubernetes exposes several Node conditions.

Important conditions include:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

Example:

```text
Ready            True
MemoryPressure   False
DiskPressure     False
PIDPressure      False
```

This represents a generally healthy Node.

---

# 8. MemoryPressure

MemoryPressure indicates that the Node is experiencing memory pressure.

Typical flow:

```text
Node Memory
    ↓
Available memory decreases
    ↓
MemoryPressure
    ↓
Kubernetes may evict Pods
```

Investigate:

```text
Node memory usage
Pod memory usage
OOM events
Memory requests
Memory limits
```

---

# 9. DiskPressure

DiskPressure indicates that the Node is experiencing filesystem or ephemeral-storage pressure.

Typical flow:

```text
Disk usage ↑
    ↓
Available storage ↓
    ↓
DiskPressure
    ↓
Possible Pod eviction
```

Common causes:

```text
Container logs
Container images
Temporary files
EmptyDir usage
Application-generated files
```

---

# 10. PIDPressure

PIDPressure occurs when the Node is running low on available process IDs.

Possible causes:

```text
Too many processes
Process leaks
Large numbers of containers
Misbehaving workloads
```

Monitor:

```text
Process count
PID availability
Node condition
```

---

# 11. NetworkUnavailable

NetworkUnavailable can indicate that the Node networking is not correctly configured or available.

Investigate:

```text
CNI
Node networking
Routes
Cloud networking
Network interfaces
```

In managed Kubernetes environments, the exact cause depends on the networking implementation.

---

# 12. CPU Monitoring

Monitor Node CPU:

```text
CPU usage
CPU utilization
CPU capacity
CPU allocatable
CPU requested
CPU limits
CPU throttling
```

Example:

```text
Node CPU capacity = 8 CPU
Actual usage       = 6 CPU
```

Utilization:

```text
6 / 8 × 100 = 75%
```

The appropriate alert threshold depends on workload characteristics.

---

# 13. CPU Saturation

High CPU utilization does not automatically mean an incident.

Example:

```text
CPU = 90%
Latency = normal
Errors = normal
```

This may be acceptable.

But:

```text
CPU = 90%
Latency = high
Errors = increasing
```

is more concerning.

Always correlate CPU with workload behavior.

---

# 14. CPU Requests

Pod CPU requests determine scheduling requirements.

Example:

```yaml
resources:
  requests:
    cpu: 500m
```

If a Node has:

```text
Allocatable CPU = 8
```

and Pods request:

```text
7.5 CPU
```

the Node has very little remaining scheduling capacity.

Monitor:

```text
Requested CPU
Allocatable CPU
Actual CPU usage
```

---

# 15. CPU Overcommitment

Kubernetes can schedule workloads based on requests even when limits allow higher usage.

Example:

```text
Node allocatable = 8 CPU

Pod requests:
2 + 2 + 2 + 2 = 8 CPU

Limits:
4 + 4 + 4 + 4 = 16 CPU
```

The Node is fully committed by requests but could experience contention if all workloads attempt to use their limits simultaneously.

Therefore monitor both:

```text
Requests
and
Actual usage
```

---

# 16. CPU Throttling

CPU limits can cause container CPU throttling.

At the Node level, monitor whether workloads are experiencing:

```text
CPU saturation
CPU contention
CPU throttling
```

Symptoms may include:

```text
Higher latency
Lower throughput
Slow application processing
```

---

# 17. Memory Monitoring

Monitor:

```text
Total memory
Used memory
Available memory
Memory pressure
Pod memory usage
System memory
```

Example:

```text
Node memory = 32 GiB
Used        = 27 GiB
```

The important question is not simply whether memory is high, but whether the Node has sufficient available memory for system and workload needs.

---

# 18. Memory Requests

Example:

```yaml
resources:
  requests:
    memory: 512Mi
```

The scheduler uses memory requests when deciding whether a Pod can fit on a Node.

Monitor:

```text
Total allocatable memory
Requested memory
Actual memory usage
Remaining capacity
```

---

# 19. Memory Limits

Memory limits define the maximum memory a container can consume before Kubernetes/container runtime behavior may result in termination due to memory constraints.

Example:

```yaml
resources:
  limits:
    memory: 1Gi
```

If a container repeatedly reaches its limit:

```text
Memory usage ↑
      ↓
OOMKilled
      ↓
Restart
```

Node and Pod monitoring should be correlated.

---

# 20. Node Memory vs Pod Memory

Suppose:

```text
Node memory = 32 GiB
```

Pod usage:

```text
Payment = 5 GiB
Orders  = 4 GiB
Inventory = 3 GiB
```

Total application usage:

```text
12 GiB
```

But the Node also requires memory for:

```text
kubelet
container runtime
OS
system processes
networking
monitoring agents
```

Therefore:

```text
Node memory ≠ Sum of application memory alone
```

---

# 21. Disk Monitoring

Monitor Node storage:

```text
Root filesystem
Container filesystem
Image filesystem
Ephemeral storage
Log directories
```

Important signals:

```text
Disk utilization
Available bytes
Inodes
I/O
DiskPressure
```

---

# 22. Disk Utilization

Example:

```text
Disk = 500 GB
Used = 450 GB
```

Utilization:

```text
90%
```

High disk usage can become a serious issue if logs, images, or temporary files continue growing.

---

# 23. Inode Monitoring

A filesystem can run out of inodes even when disk space remains.

Example:

```text
Disk usage = 60%
Inode usage = 98%
```

The Node can still experience filesystem problems.

Monitor:

```text
Inode usage
Inode availability
Filesystem usage
```

---

# 24. Container Images and Disk

Nodes store container images.

Example:

```text
Node
│
├── image:v1
├── image:v2
├── image:v3
├── image:v4
└── image:v5
```

If old images accumulate:

```text
Disk usage ↑
```

Image garbage collection helps control this.

---

# 25. Container Logs and Disk

High-volume logs can consume Node storage.

Example:

```text
Application
    ↓
Huge log volume
    ↓
Node filesystem
    ↓
Disk usage ↑
    ↓
DiskPressure
```

Monitor both:

```text
Application log volume
Node disk usage
```

---

# 26. Ephemeral Storage

Pods can consume ephemeral storage through:

```text
Container writable layer
EmptyDir
Temporary files
Container logs
```

Monitor:

```text
Ephemeral storage requests
Ephemeral storage limits
Actual usage
Node available storage
```

---

# 27. Network Monitoring

Monitor Node network:

```text
Receive throughput
Transmit throughput
Packets
Errors
Drops
Connections
Network interface utilization
```

High network usage can affect application performance.

---

# 28. Network Errors

Monitor:

```text
RX errors
TX errors
Dropped packets
Interface failures
Connection failures
```

A Node can be:

```text
Ready = True
```

while still experiencing network degradation.

Correlate Node network metrics with application latency and errors.

---

# 29. Network Bandwidth Saturation

Example:

```text
Node network capacity = 10 Gbps
Current traffic        = 9.8 Gbps
```

Potential consequences:

```text
Increased latency
Packet drops
Connection failures
Slow application responses
```

Investigate:

```text
High-traffic Pods
Ingress traffic
Service traffic
Network interfaces
CNI behavior
```

---

# 30. Kubelet Monitoring

The kubelet runs on each Node and manages Pod lifecycle operations.

Monitor:

```text
Kubelet availability
Kubelet errors
Pod lifecycle operations
Probe handling
Runtime communication
Resource pressure
```

If kubelet becomes unhealthy:

```text
Node
 ↓
Kubelet failure
 ↓
Node health problems
```

---

# 31. Container Runtime Monitoring

The container runtime is responsible for running containers.

Monitor:

```text
Runtime availability
Container creation failures
Container startup
Container termination
Runtime errors
```

A runtime problem can affect many Pods on the same Node.

---

# 32. Node Capacity

Node capacity represents the total resources available on the Node.

Example:

```text
CPU:    8
Memory: 32 GiB
```

But workloads are generally scheduled based on allocatable resources rather than raw capacity.

---

# 33. Node Allocatable

Allocatable resources are the resources Kubernetes makes available for Pods after reserving resources for system components.

Conceptually:

```text
Node Capacity
      ↓
System Reservations
      ↓
Kubernetes Reservations
      ↓
Allocatable
      ↓
Pods
```

This distinction is important for capacity planning.

---

# 34. System Reserved Resources

Some Node resources should be reserved for the operating system and system processes.

Conceptually:

```text
Node
├── OS / System
├── Kubernetes components
└── Pods
```

Without appropriate reservations, workloads can consume resources needed by the Node itself.

---

# 35. Kube Reserved Resources

Resources may also be reserved for Kubernetes components such as:

```text
kubelet
container runtime
other Kubernetes processes
```

This helps maintain Node stability under workload pressure.

---

# 36. Allocatable Capacity Monitoring

A useful capacity dashboard can show:

```text
CPU Capacity
CPU Allocatable
CPU Requested
CPU Used

Memory Capacity
Memory Allocatable
Memory Requested
Memory Used
```

This provides a complete picture of Node capacity.

---

# 37. Node Resource Fragmentation

A Node can have enough total resources but still fail to schedule a Pod because the required resources cannot be accommodated according to scheduling constraints.

Example:

```text
Node:
Available CPU = 2 CPU
Available memory = 1 GiB

New Pod requires:
CPU = 1 CPU
Memory = 4 GiB
```

The Pod cannot fit despite available CPU.

Monitor both:

```text
CPU capacity
Memory capacity
```

---

# 38. Node Pod Capacity

Nodes have a maximum number of Pods they can support, depending on configuration and platform limitations.

Monitor:

```text
Maximum Pods
Current Pods
Available Pod capacity
```

Example:

```text
Maximum Pods = 110
Current Pods = 105
```

Only a small amount of Pod capacity remains.

---

# 39. Pod Density

Pod density refers to the number of Pods running on a Node.

Example:

```text
Node-1 → 90 Pods
Node-2 → 40 Pods
Node-3 → 35 Pods
```

High Pod density can create:

```text
Resource contention
Network complexity
IP exhaustion risk
Higher kubelet workload
```

---

# 40. EKS Pod Density

In EKS, Pod capacity can be affected by networking and available IP addresses.

Conceptually:

```text
Node
 ↓
Network interfaces
 ↓
Available IP addresses
 ↓
Maximum Pod placement
```

Therefore monitor:

```text
Node Pod capacity
Available IPs
CNI health
Subnet IP availability
```

---

# 41. Node IP Exhaustion

Suppose:

```text
Pods Pending
```

while:

```text
CPU = available
Memory = available
```

A networking/IP limitation may still prevent scheduling.

In EKS, investigate:

```text
VPC subnet IP availability
AWS VPC CNI
Node ENI/IP allocation
Pod networking configuration
```

---

# 42. Node Conditions and Scheduling

Node conditions affect whether new Pods can be scheduled.

Example:

```text
Node
 ↓
MemoryPressure
 ↓
Scheduling / eviction behavior
```

Monitoring Node conditions helps explain unexpected Pod Pending or eviction events.

---

# 43. Taints

A Node may have a taint:

```text
key=value:NoSchedule
```

Only Pods with matching tolerations can be scheduled.

Monitoring should include:

```text
Node taints
Pod tolerations
Scheduling failures
```

---

# 44. Node Labels

Node labels can influence workload placement.

Examples:

```text
workload=compute
environment=production
nodegroup=application
```

Monitor workload distribution based on these labels.

Incorrect labels can cause:

```text
Pods Pending
Uneven placement
Unexpected workload distribution
```

---

# 45. Node Affinity

Node affinity can force Pods onto particular Nodes.

Example:

```text
Pod
 ↓
node affinity
 ↓
Specific Node group
```

If insufficient Nodes satisfy the requirement:

```text
Pod → Pending
```

Monitoring scheduling failures should therefore include affinity constraints.

---

# 46. Node Failure

A Node can fail because of:

```text
Operating system failure
Cloud instance failure
Network failure
Disk failure
Kubelet failure
Container runtime failure
Resource exhaustion
```

Typical sequence:

```text
Node healthy
    ↓
Failure
    ↓
NotReady
    ↓
Pods affected
    ↓
Pods rescheduled where possible
```

---

# 47. Node Heartbeat

Kubernetes relies on Node health information and kubelet communication to determine Node status.

If the Node stops reporting correctly:

```text
Node communication lost
      ↓
Node becomes unhealthy
```

Monitor:

```text
Kubelet health
Node Ready condition
Node heartbeat-related signals
```

---

# 48. Node Drain

During maintenance, Nodes may be drained.

Example:

```bash
kubectl drain <node-name>
```

This causes eligible workloads to be evicted and rescheduled.

Monitor:

```text
Pods being evicted
Available replicas
Pending Pods
Node status
```

A drain should not unintentionally reduce critical application availability.

---

# 49. Node Cordon

A Node can be cordoned:

```bash
kubectl cordon <node-name>
```

This prevents new Pods from being scheduled there while existing Pods remain.

Useful for:

```text
Maintenance
Troubleshooting
Controlled workload movement
```

Monitor:

```text
Scheduling state
Existing workload health
```

---

# 50. Node Maintenance

Typical maintenance workflow:

```text
Check workload redundancy
        ↓
Cordon Node
        ↓
Drain Node
        ↓
Perform maintenance
        ↓
Validate Node
        ↓
Uncordon
        ↓
Monitor workloads
```

Node monitoring is essential throughout this process.

---

# 51. Node Reboot

After a reboot:

```text
Node unavailable
   ↓
Pods disrupted
   ↓
Node returns
   ↓
Kubelet starts
   ↓
Node Ready
```

Monitor:

```text
Node Ready
Pod recovery
Pod restart count
Application availability
```

---

# 52. Node Autoscaling

Cluster Autoscaler or other autoscaling mechanisms may add or remove Nodes.

Example:

```text
Pods Pending
    ↓
Insufficient capacity
    ↓
Autoscaler
    ↓
New Node
    ↓
Pods scheduled
```

Monitor:

```text
Node count
Pending Pods
Scaling events
Node provisioning time
```

---

# 53. Node Scale-In

When capacity is no longer required:

```text
Unused capacity
     ↓
Autoscaler
     ↓
Node removal
     ↓
Pods rescheduled
```

Monitor:

```text
Pod availability
Disruption
Node utilization
Scale-in events
```

Poorly configured scaling can cause unnecessary churn.

---

# 54. Node Utilization

A simple utilization view:

```text
Node-1 → CPU 70%, Memory 60%
Node-2 → CPU 30%, Memory 40%
Node-3 → CPU 85%, Memory 80%
```

Node-3 may require investigation depending on workload behavior.

But utilization alone should not determine scaling decisions.

---

# 55. Uneven Node Utilization

Example:

```text
Node-1 → 20%
Node-2 → 25%
Node-3 → 90%
```

Possible causes:

```text
Pod affinity
Taints
Topology constraints
Different Pod sizes
Uneven scheduling
Long-running workloads
```

Investigate placement before changing Node capacity.

---

# 56. Node Resource Pressure

Monitor:

```text
CPU pressure
Memory pressure
Disk pressure
PID pressure
```

A pressure condition can affect:

```text
Scheduling
Pod eviction
Application availability
Node stability
```

---

# 57. Pod Eviction From Node Pressure

Typical sequence:

```text
Node resource usage ↑
       ↓
Pressure condition
       ↓
Kubernetes eviction
       ↓
Pod terminated
       ↓
Pod rescheduled
```

Monitor:

```text
Eviction count
Eviction reason
Affected namespace
Affected workload
Node condition
```

---

# 58. Eviction Priority

Kubernetes uses resource requests and other factors when making eviction decisions.

Therefore workloads should define appropriate:

```text
CPU requests
Memory requests
Priority
QoS configuration
```

Poor resource configuration can increase operational risk.

---

# 59. Kubernetes QoS Classes

Pods generally fall into:

```text
Guaranteed
Burstable
BestEffort
```

Resource configuration affects the QoS class.

Example:

```text
Guaranteed
→ Requests and limits defined appropriately for containers
```

```text
Burstable
→ Some resource requests/limits defined
```

```text
BestEffort
→ No requests or limits
```

QoS can influence behavior during resource pressure.

---

# 60. Guaranteed Workloads

Example:

```yaml
resources:
  requests:
    cpu: "1"
    memory: 1Gi
  limits:
    cpu: "1"
    memory: 1Gi
```

For a Pod meeting the relevant Kubernetes conditions, this contributes to Guaranteed QoS.

This can provide stronger predictability for critical workloads.

---

# 61. BestEffort Workloads

If containers have no CPU or memory requests/limits:

```text
BestEffort
```

Such workloads can be more vulnerable during resource pressure.

Production workloads should intentionally define resource requirements rather than relying on accidental defaults.

---

# 62. Burstable Workloads

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 1Gi
```

This provides a guaranteed baseline with room to burst.

Many production workloads use this model, depending on their requirements.

---

# 63. Node Monitoring Dashboard

A useful Node dashboard can include:

```text
Node Overview
├── Total Nodes
├── Ready Nodes
├── NotReady Nodes
├── CPU Usage
├── Memory Usage
├── Disk Usage
├── Network Usage
├── Pod Count
├── Pod Capacity
├── CPU Requested
├── Memory Requested
├── DiskPressure
├── MemoryPressure
├── PIDPressure
└── Evictions
```

---

# 64. Node Dashboard Example

```text
┌──────────────────────────────────────────────┐
│              NODE OVERVIEW                   │
├──────────────────────────────────────────────┤
│ Nodes              12                        │
│ Ready              12                        │
│ NotReady            0                        │
│                                                  │
│ CPU Usage          63%                        │
│ Memory Usage       71%                        │
│ Disk Usage         58%                        │
│ Network Usage      42%                        │
│                                                  │
│ Pods              420                        │
│ Pod Capacity      528                        │
│                                                  │
│ MemoryPressure      0                        │
│ DiskPressure        0                        │
│ PIDPressure         0                        │
└──────────────────────────────────────────────┘
```

---

# 65. Node Alerts

Useful alerts include:

```text
Node NotReady
MemoryPressure
DiskPressure
PIDPressure
High CPU
High memory
High disk usage
Low filesystem space
High inode usage
Network errors
Network drops
High Pod density
Low Pod capacity
Kubelet unavailable
Container runtime errors
```

Alert thresholds should reflect workload and operational requirements.

---

# 66. Node NotReady Alert

A Node NotReady condition should generally trigger an alert when it persists beyond a short transient period.

Example:

```text
Node
 ↓
NotReady
 ↓
Alert
```

The alert should include:

```text
Node name
Cluster
Availability Zone
Node group
Condition
Duration
```

This gives responders immediate context.

---

# 67. MemoryPressure Alert

Example:

```text
Node
 ↓
MemoryPressure = True
 ↓
Alert
```

Then investigate:

```text
Top memory-consuming Pods
Node system memory
Memory requests
Memory limits
OOM events
```

---

# 68. DiskPressure Alert

Example:

```text
Node
 ↓
DiskPressure = True
 ↓
Alert
```

Investigate:

```text
Disk usage
Logs
Container images
Ephemeral storage
Large files
Inodes
```

---

# 69. High Node CPU Alert

A useful alert should account for duration.

For example:

```text
CPU > high threshold
for sustained period
```

Then correlate with:

```text
Pod traffic
Pod CPU
Latency
Throttling
Node capacity
```

Avoid paging solely on short CPU spikes.

---

# 70. Node Capacity Alert

A useful capacity alert can detect:

```text
Allocatable CPU almost fully requested
```

or:

```text
Allocatable memory almost fully requested
```

This warns that the cluster may soon have difficulty scheduling new workloads.

---

# 71. Pod Capacity Alert

For EKS Nodes:

```text
Current Pods
     ↓
Maximum Pod capacity
```

If:

```text
Current = 95
Maximum = 110
```

only 15 Pod slots remain.

This can become a scheduling constraint even when CPU and memory remain available.

---

# 72. Node Network Monitoring

Monitor:

```text
Bytes received
Bytes transmitted
Packets received
Packets transmitted
Errors
Drops
```

Investigate sudden changes.

Example:

```text
Network traffic suddenly 3× higher
```

Possible causes:

```text
Traffic increase
Log/trace explosion
Large data transfer
Misbehaving workload
Network attack
```

---

# 73. Node I/O Monitoring

Disk I/O can affect application performance.

Monitor:

```text
Read IOPS
Write IOPS
Read throughput
Write throughput
I/O latency
```

A Node can have:

```text
CPU = 40%
Memory = 50%
Disk I/O = saturated
```

and applications may still experience high latency.

---

# 74. Node Load Average

For Linux Nodes, load average can provide additional OS-level context.

Example:

```bash
uptime
```

or:

```bash
top
```

Load should be interpreted relative to:

```text
CPU count
Runnable processes
I/O waits
Workload behavior
```

Do not treat load average as a direct equivalent of CPU utilization.

---

# 75. Node Filesystem Monitoring

Monitor important filesystems:

```text
/
 /var
 /var/lib/containerd
 /var/log
```

depending on the Node operating system and container runtime.

Look for:

```text
High usage
Low free space
High inode usage
Unexpected file growth
```

---

# 76. Kubelet Disk Usage

Kubelet and container runtime data can consume significant disk space.

Investigate:

```text
Container images
Container logs
Pod writable layers
Volumes
Temporary data
```

This is especially important on long-lived Nodes.

---

# 77. Container Runtime Cleanup

Container runtimes maintain image and container data.

If cleanup is not working correctly:

```text
Unused images
     ↓
Disk usage ↑
     ↓
DiskPressure
```

Monitor:

```text
Image filesystem
Container runtime health
Disk usage
Garbage collection behavior
```

---

# 78. Node Monitoring During Cluster Upgrade

During a Kubernetes upgrade:

```text
Node
 ↓
Cordon
 ↓
Drain
 ↓
Upgrade
 ↓
Rejoin
 ↓
Ready
```

Monitor:

```text
Node Ready
Pod availability
Pod Pending
Evictions
Application errors
Cluster capacity
```

Do not upgrade Nodes faster than the workload can tolerate.

---

# 79. Node Monitoring During AZ Failure

Suppose:

```text
AZ-A
├── Node-1
├── Node-2
└── Node-3
```

becomes unavailable.

Monitor:

```text
Node count
Ready Nodes
Pod availability
Pending Pods
Cross-AZ capacity
Autoscaler behavior
```

A well-distributed workload should continue operating with sufficient remaining capacity.

---

# 80. Node Monitoring During Application Incident

Suppose:

```text
Payment latency ↑
```

Before changing the application, check:

```text
Node CPU
Node memory
Node network
Node disk
Pod CPU
Pod memory
Pod restarts
```

This determines whether the problem is:

```text
Application-level
Pod-level
Node-level
Cluster-level
```

---

# 81. Node vs Pod Troubleshooting

### Node Problem

```text
Many Pods affected
CPU high across Node
MemoryPressure
DiskPressure
Network errors
```

### Pod Problem

```text
One workload affected
One Pod high CPU
One Pod restarting
One Pod failing probes
```

This distinction helps avoid incorrect remediation.

---

# 82. Example: Multiple Pods Become Slow

Suppose:

```text
Node-1
├── Orders → slow
├── Payment → slow
├── Inventory → slow
└── Notification → slow
```

This pattern suggests investigating the Node.

Check:

```text
CPU
Memory
Disk I/O
Network
Kubelet
Container runtime
```

If only Payment is slow:

```text
Node-1
├── Orders → normal
├── Payment → slow
├── Inventory → normal
└── Notification → normal
```

investigate the Payment workload first.

---

# 83. Example: Node DiskPressure

Problem:

```text
Node-2
DiskPressure = True
```

Investigation:

```text
Disk usage
 ↓
/var/lib/containerd large
 ↓
Old images
 ↓
Cleanup / garbage collection issue
```

Or:

```text
Disk usage
 ↓
/var/log large
 ↓
High-volume application logs
 ↓
Logging configuration issue
```

---

# 84. Example: Node MemoryPressure

Problem:

```text
Node-3
MemoryPressure = True
```

Check:

```text
Top Pods by memory
 ↓
Application A = 8 GiB
Application B = 6 GiB
```

Then:

```text
Application A
 ↓
Memory usage continuously increasing
 ↓
Possible memory leak
```

This is more useful than simply adding memory to the Node without investigation.

---

# 85. Example: Node NotReady

Problem:

```text
node-4 = NotReady
```

Check:

```bash
kubectl describe node node-4
```

Then investigate:

```text
Kubelet
Container runtime
Network
OS
Disk
Cloud instance
```

If kubelet is unavailable:

```text
Node
 ↓
Kubelet
 X
 ↓
Node health degraded
```

---

# 86. Example: Pods Pending After Node Failure

Suppose:

```text
12 Nodes
 ↓
1 Node fails
 ↓
Pods need rescheduling
```

But:

```text
Remaining capacity = insufficient
```

Then:

```text
Pods Pending
```

Investigate:

```text
Remaining CPU
Remaining memory
Pod requests
Topology constraints
Autoscaler
```

This demonstrates why Node monitoring must include spare capacity.

---

# 87. Node Monitoring and High Availability

A production cluster should maintain enough spare capacity for expected failures.

Example:

```text
Current usage = 65%
```

If one Node fails:

```text
Remaining Nodes = 80%+
```

Capacity may still be sufficient.

But if:

```text
Current usage = 90%
```

a single Node failure may cause:

```text
Pods Pending
Resource contention
Application degradation
```

Capacity planning is therefore part of availability engineering.

---

# 88. Node Headroom

Node headroom is the unused capacity available for:

```text
Traffic spikes
Pod rescheduling
Node failures
Rolling deployments
Autoscaling
Maintenance
```

Monitor:

```text
CPU headroom
Memory headroom
Pod headroom
Network headroom
Storage headroom
```

---

# 89. Node Monitoring Best Practices

```text
1. Monitor Node Ready state.
2. Monitor MemoryPressure.
3. Monitor DiskPressure.
4. Monitor PIDPressure.
5. Monitor CPU.
6. Monitor memory.
7. Monitor disk and inodes.
8. Monitor network traffic and errors.
9. Monitor kubelet.
10. Monitor container runtime.
11. Monitor Pod capacity.
12. Monitor allocatable resources.
13. Monitor resource requests.
14. Monitor workload distribution.
15. Monitor evictions.
16. Maintain sufficient headroom.
17. Monitor Node failures.
18. Distribute Nodes across Availability Zones.
19. Correlate Node metrics with Pod metrics.
20. Use alerts based on actionable symptoms.
```

---

# 90. Prometheus Node Monitoring

A common architecture:

```text
Node
│
├── kubelet
├── container runtime
├── node-exporter
└── Pods
       │
       ↓
   Metrics
       ↓
   Prometheus
       ↓
    Grafana
```

Node Exporter can provide operating-system-level metrics.

Kubelet/container metrics provide additional Kubernetes/container context.

---

# 91. Node Exporter

Node Exporter commonly provides host-level metrics such as:

```text
CPU
Memory
Filesystem
Network
Load
Disk
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

This complements Kubernetes object-state metrics.

---

# 92. kube-state-metrics vs Node Exporter

### kube-state-metrics

Provides Kubernetes object state such as:

```text
Node conditions
Pod state
Deployment state
Replica state
```

### Node Exporter

Provides host-level OS metrics such as:

```text
CPU
Memory
Disk
Network
Filesystem
```

They answer different questions.

---

# 93. Metrics Server vs Prometheus

Metrics Server is commonly used for:

```text
Resource metrics
HPA
kubectl top
```

Prometheus is commonly used for:

```text
Historical metrics
Dashboards
Alerting
Rich querying
Longer-term observability
```

They are not interchangeable.

---

# 94. kubectl top Nodes

A quick view of Node resource usage:

```bash
kubectl top nodes
```

Example:

```text
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
node-1   1200m        30%    8Gi             25%
node-2   2500m        62%    20Gi            62%
node-3   900m         22%    7Gi             21%
```

This is useful for immediate troubleshooting.

---

# 95. kubectl describe Node

Use:

```bash
kubectl describe node <node-name>
```

Useful sections include:

```text
Conditions
Capacity
Allocatable
System Info
Non-terminated Pods
Events
```

This is one of the most useful commands for Node troubleshooting.

---

# 96. Node Monitoring Workflow

When a Node is unhealthy:

```text
1. kubectl get nodes
2. Identify unhealthy Node
3. kubectl describe node
4. Check Conditions
5. Check Capacity
6. Check Allocatable
7. Check Pod distribution
8. Check CPU
9. Check memory
10. Check disk
11. Check network
12. Check kubelet
13. Check container runtime
14. Check Events
15. Check affected Pods
16. Correlate with Prometheus/Grafana
```

---

# 97. Node Monitoring With Logs

Metrics can identify:

```text
Node NotReady
```

Node logs can help identify:

```text
Kubelet error
Container runtime error
Network issue
Filesystem problem
```

Therefore:

```text
Metrics
 ↓
Identify Node
 ↓
Logs
 ↓
Root cause
```

---

# 98. Node Monitoring With Traces

Suppose:

```text
Application latency ↑
```

Tracing identifies:

```text
Payment Pod
 ↓
Node-3
```

Then Node monitoring shows:

```text
Node-3 CPU = 95%
```

This correlation can reveal infrastructure-level impact on application performance.

---

# 99. Node Monitoring With Logs

Suppose:

```text
Node DiskPressure
```

Node metrics show:

```text
Disk = 92%
```

ELK shows:

```text
Application producing huge log volume
```

The root cause is clearer:

```text
Application log volume
 ↓
Node disk growth
 ↓
DiskPressure
```

---

# 100. Complete Node Observability

```text
                         NODE
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
     State             Resources           Workloads
       │                  │                  │
       ↓                  ↓                  ↓
kube-state-metrics   Node Exporter      kubelet/container
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                      Prometheus
                          ↓
                        Grafana
                          ↓
                        Alerts
```

---

# 101. Production Node Architecture

A production EKS environment may look like:

```text
                         EKS CLUSTER
                              │
       ┌──────────────────────┼──────────────────────┐
       ↓                      ↓                      ↓
    AZ-A                   AZ-B                   AZ-C
       │                      │                      │
   ┌───┴───┐              ┌───┴───┐              ┌───┴───┐
   ↓       ↓              ↓       ↓              ↓       ↓
 Node-1  Node-2        Node-3  Node-4        Node-5  Node-6
   │       │              │       │              │       │
  Pods    Pods           Pods    Pods           Pods    Pods
```

Monitor:

```text
Node health
Resource headroom
Pod distribution
AZ distribution
Network
Storage
Autoscaling
```

---

# 102. Node Monitoring During Traffic Spike

Traffic increases:

```text
Traffic ↑
   ↓
Pod CPU ↑
   ↓
HPA scales Pods
   ↓
Node capacity ↓
   ↓
Cluster Autoscaler
   ↓
New Nodes
```

Monitor every stage:

```text
Traffic
Pods
Nodes
HPA
Autoscaler
Application latency
```

---

# 103. Node Monitoring During Cluster Scaling

Scale-out:

```text
New Node requested
       ↓
Cloud instance launched
       ↓
Node joins cluster
       ↓
Node Ready
       ↓
Pods scheduled
```

Monitor:

```text
Node provisioning time
Node join failures
Node Ready time
Pending Pods
Application recovery
```

---

# 104. Node Monitoring During Node Group Upgrade

For a rolling Node group upgrade:

```text
Old Node
   ↓
Cordon
   ↓
Drain
   ↓
Terminate
   ↓
New Node
   ↓
Ready
   ↓
Pods scheduled
```

Monitor:

```text
Ready Nodes
Available replicas
Pending Pods
Evictions
Scheduling latency
Application errors
```

---

# 105. Node Monitoring and SLOs

Node health ultimately matters because it affects application SLOs.

Example:

```text
Node issue
   ↓
Pod unavailable
   ↓
Application errors
   ↓
SLO degradation
```

Therefore Node alerts should be connected to application impact where possible.

---

# 106. Node Monitoring Mental Model

Remember:

```text
NODE
│
├── HEALTH
│   ├── Ready
│   ├── MemoryPressure
│   ├── DiskPressure
│   └── PIDPressure
│
├── RESOURCES
│   ├── CPU
│   ├── Memory
│   ├── Disk
│   └── Network
│
├── CAPACITY
│   ├── Capacity
│   ├── Allocatable
│   ├── Requests
│   └── Pod density
│
├── KUBERNETES
│   ├── Kubelet
│   ├── Runtime
│   └── Events
│
└── WORKLOADS
    ├── Pods
    ├── Distribution
    ├── Evictions
    └── Availability
```

---

# 107. Interview Question

### How do you monitor Kubernetes Nodes?

**Answer:**

I monitor Node Ready status, CPU, memory, disk, filesystem and inode usage, network traffic and errors, Pod capacity, allocatable resources, resource requests, MemoryPressure, DiskPressure, PIDPressure, kubelet health, container runtime health, and Pod distribution. I use Prometheus and Grafana for historical metrics and alerting, Node Exporter for host-level metrics, and kube-state-metrics for Kubernetes object state.

---

# 108. Interview Question

### A Node becomes NotReady. How would you troubleshoot it?

**Answer:**

I would first run `kubectl describe node` and inspect the Conditions and Events. Then I would check kubelet health, container runtime health, CPU and memory pressure, disk and filesystem usage, network connectivity, and operating-system health. I would also identify which Pods were running on that Node and determine whether they were rescheduled successfully. Finally, I would correlate the issue with infrastructure or cloud-provider events.

---

# 109. Interview Question

### What is the difference between capacity and allocatable?

**Answer:**

Capacity represents the total resources available on the Node, while allocatable represents the resources Kubernetes makes available for Pods after accounting for resources reserved for system and Kubernetes components. Scheduling is based primarily on allocatable resources rather than raw Node capacity.

---

# 110. Interview Question

### How do you troubleshoot DiskPressure?

**Answer:**

I would check filesystem usage, available space, inode usage, container images, container logs, ephemeral storage, and large temporary files. I would determine what is consuming the disk before taking cleanup actions. I would also check whether log rotation and container image garbage collection are functioning correctly.

---

# 111. Interview Question

### How do you troubleshoot MemoryPressure?

**Answer:**

I would check Node memory usage and identify the highest memory-consuming Pods. Then I would inspect memory requests and limits, OOMKilled events, Pod restart rates, and memory trends. If one application is continuously increasing memory, I would investigate a possible memory leak. If overall workload demand has increased, I would evaluate capacity and scaling.

---

# 112. Interview Question

### Why can a Node be CPU healthy but still unable to schedule a Pod?

**Answer:**

Scheduling depends on multiple resources and constraints. The Node may have sufficient CPU but insufficient memory, ephemeral storage, Pod capacity, available IP addresses, or may not satisfy taints, affinity, topology, or other scheduling constraints. Therefore I evaluate all relevant scheduling conditions rather than CPU alone.

---

# 113. Interview Question

### How do you monitor Node capacity?

**Answer:**

I compare Node capacity and allocatable resources against current requests and actual usage. I monitor CPU, memory, storage, Pod capacity, and network headroom. I also maintain enough spare capacity for Node failures, rolling deployments, traffic spikes, and workload rescheduling.

---

# 114. Interview Question

### How do you detect a Node causing application latency?

**Answer:**

I first identify whether multiple Pods on the same Node are experiencing elevated latency. Then I compare Node CPU, memory, disk I/O, network utilization, and pressure conditions with Pod-level metrics. I also correlate application traces and logs. If multiple unrelated workloads on the same Node degrade simultaneously, I investigate the Node as a likely common factor.

---

# 115. Interview Question

### How do you monitor Nodes during a Kubernetes upgrade?

**Answer:**

I monitor Node Ready status, Pod availability, eviction events, Pending Pods, scheduling capacity, and application-level metrics during each Node replacement. I cordon and drain Nodes carefully, ensure sufficient workload redundancy, verify the replacement Node becomes Ready, and confirm that workloads recover before continuing the upgrade.

---

# 116. Production Node Monitoring Checklist

```text
NODE HEALTH
[ ] Ready
[ ] NotReady
[ ] MemoryPressure
[ ] DiskPressure
[ ] PIDPressure
[ ] NetworkUnavailable

CPU
[ ] CPU usage
[ ] CPU capacity
[ ] CPU allocatable
[ ] CPU requested
[ ] CPU headroom
[ ] CPU contention

MEMORY
[ ] Memory usage
[ ] Memory capacity
[ ] Memory allocatable
[ ] Memory requested
[ ] Memory headroom
[ ] OOM events

STORAGE
[ ] Disk usage
[ ] Free space
[ ] Inodes
[ ] Ephemeral storage
[ ] Container images
[ ] Container logs
[ ] Disk I/O

NETWORK
[ ] RX traffic
[ ] TX traffic
[ ] Packets
[ ] Errors
[ ] Drops
[ ] Bandwidth
[ ] IP availability

KUBERNETES
[ ] Kubelet
[ ] Container runtime
[ ] Node conditions
[ ] Events
[ ] Capacity
[ ] Allocatable
[ ] Pod capacity

WORKLOADS
[ ] Pod count
[ ] Pod distribution
[ ] Pod requests
[ ] Pod evictions
[ ] Pending Pods
[ ] Ready replicas

OPERATIONS
[ ] Cordon
[ ] Drain
[ ] Maintenance
[ ] Upgrades
[ ] Autoscaling
[ ] Node replacement

OBSERVABILITY
[ ] Node Exporter
[ ] kube-state-metrics
[ ] Prometheus
[ ] Grafana
[ ] Logs
[ ] Traces
[ ] Alerts
```

---

# 117. Final Architecture

```text
                         KUBERNETES CLUSTER
                                  │
              ┌───────────────────┼───────────────────┐
              ↓                   ↓                   ↓
           NODE-1              NODE-2              NODE-3
              │                   │                   │
        ┌─────┼─────┐       ┌─────┼─────┐       ┌─────┼─────┐
        ↓     ↓     ↓       ↓     ↓     ↓       ↓     ↓     ↓
       Pod   Pod   Pod      Pod   Pod   Pod      Pod   Pod   Pod
              │                   │                   │
              └───────────────────┼───────────────────┘
                                  ↓
                     ┌────────────────────────┐
                     │   Node-Level Metrics   │
                     │ CPU / Memory / Disk    │
                     │ Network / Filesystem   │
                     └───────────┬────────────┘
                                 ↓
                           Prometheus
                                 ↓
                              Grafana
                                 ↓
                              Alerts
```

Combined with Kubernetes state:

```text
Node Exporter ──────┐
                    ├──→ Prometheus ──→ Grafana
kube-state-metrics ─┘
                    │
                    └──→ Alerts
```

The key principle is:

**Node monitoring provides visibility into the infrastructure layer that hosts Kubernetes workloads. A production Node should be monitored for readiness, CPU, memory, disk, network, filesystem, PID pressure, kubelet and runtime health, allocatable capacity, Pod density, and workload distribution. The most effective approach combines host-level metrics from Node Exporter, Kubernetes object state from kube-state-metrics, Prometheus/Grafana dashboards and alerts, and correlation with Pod metrics, logs, and traces. Maintaining sufficient Node headroom and distributing Nodes across failure domains are essential for reliable Kubernetes operations.**
