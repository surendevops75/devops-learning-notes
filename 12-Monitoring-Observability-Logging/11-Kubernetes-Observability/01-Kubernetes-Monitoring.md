# Kubernetes Monitoring

## 1. Overview

Kubernetes monitoring is the process of collecting, analyzing, and alerting on the health and performance of Kubernetes workloads and infrastructure.

A Kubernetes environment has multiple layers that need to be monitored:

```text
Kubernetes Monitoring
│
├── Applications
├── Pods
├── Containers
├── Nodes
├── Services
├── Deployments
├── StatefulSets
├── DaemonSets
├── Cluster Components
├── Kubernetes API
└── Control Plane
```

The goal is to answer:

```text
Is the application healthy?
Are Pods healthy?
Are Nodes healthy?
Is the cluster healthy?
Is the application consuming resources correctly?
Are workloads scaling correctly?
Are there failures or capacity problems?
```

---

# 2. Why Kubernetes Monitoring Is Important

A Kubernetes cluster can appear healthy while the application is unhealthy.

For example:

```text
Node        → Healthy
Pod         → Running
Container   → Running
Application → 503 Errors
```

Therefore, monitoring must cover more than Pod status.

A complete monitoring strategy looks at:

```text
Infrastructure
      ↓
Kubernetes
      ↓
Workloads
      ↓
Application
      ↓
User experience
```

---

# 3. Kubernetes Monitoring Layers

A useful monitoring model is:

```text
                Application
                     │
                Workloads
                     │
              Kubernetes Objects
                     │
                   Pods
                     │
                 Containers
                     │
                   Nodes
                     │
                  Cluster
                     │
                Infrastructure
```

Each layer provides different signals.

---

# 4. Monitoring Categories

Kubernetes monitoring can be divided into:

```text
1. Resource Monitoring
2. Workload Monitoring
3. Application Monitoring
4. Cluster Monitoring
5. Control Plane Monitoring
6. Network Monitoring
7. Storage Monitoring
8. Availability Monitoring
```

---

# 5. Resource Monitoring

Resource monitoring focuses on:

```text
CPU
Memory
Disk
Network
```

For example:

```text
Pod
├── CPU usage
├── Memory usage
├── Network receive
└── Network transmit
```

Resource monitoring helps identify capacity and performance problems.

---

# 6. CPU Monitoring

CPU usage can be checked with:

```bash
kubectl top pods
```

For a specific namespace:

```bash
kubectl top pods -n production
```

Node CPU:

```bash
kubectl top nodes
```

Example:

```text
NAME       CPU(cores)   MEMORY(bytes)
node-1     850m         4Gi
node-2     420m         3Gi
```

---

# 7. Memory Monitoring

Memory usage is equally important.

```bash
kubectl top pods -n production
```

Example:

```text
NAME              CPU    MEMORY
orders-xxx        200m   450Mi
payment-xxx       300m   900Mi
inventory-xxx     150m   300Mi
```

High memory usage can eventually result in:

```text
OOMKilled
Pod restart
Node memory pressure
Pod eviction
```

---

# 8. CPU vs CPU Requests

Suppose:

```yaml
resources:
  requests:
    cpu: 500m
```

and the Pod uses:

```text
200m
```

The application is currently using less CPU than its requested amount.

Requests are primarily used for scheduling and resource guarantees rather than representing a hard usage limit.

---

# 9. CPU Limits

Example:

```yaml
resources:
  limits:
    cpu: "1"
```

This establishes a CPU limit for the container.

If the container attempts to consume more CPU than allowed, it may experience CPU throttling.

Therefore, CPU throttling should be monitored rather than assuming that high CPU utilization is the only CPU problem.

---

# 10. Memory Requests

Example:

```yaml
resources:
  requests:
    memory: 512Mi
```

Kubernetes uses memory requests when scheduling the Pod.

If the node does not have sufficient allocatable memory:

```text
Pod
 ↓
Scheduler
 ↓
No suitable node
 ↓
Pod Pending
```

---

# 11. Memory Limits

Example:

```yaml
resources:
  limits:
    memory: 1Gi
```

If a container exceeds its memory limit, it can be terminated by the kernel and Kubernetes may report:

```text
Reason: OOMKilled
```

This is an important signal during troubleshooting.

---

# 12. kubectl Monitoring

`kubectl` provides basic operational visibility.

Useful commands:

```bash
kubectl get pods
kubectl get nodes
kubectl get deployments
kubectl get services
kubectl get events
```

For resource usage:

```bash
kubectl top pods
kubectl top nodes
```

`kubectl` is useful for immediate investigation, while a monitoring system such as Prometheus provides historical metrics and alerting.

---

# 13. Pod Monitoring

Monitor:

```text
Pod availability
Pod restarts
CPU
Memory
Readiness
Liveness
Container status
```

Check Pods:

```bash
kubectl get pods -A
```

Detailed information:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

---

# 14. Pod Restart Monitoring

A restart count increasing can indicate:

```text
Application crash
OOMKilled
Liveness probe failure
Configuration problem
Dependency failure
```

Check:

```bash
kubectl get pods -n production
```

Example:

```text
NAME          READY   STATUS    RESTARTS
payment-xxx   1/1     Running   7
```

A continuously increasing restart count requires investigation.

---

# 15. CrashLoopBackOff

A common Kubernetes monitoring problem is:

```text
CrashLoopBackOff
```

Typical investigation:

```bash
kubectl describe pod <pod-name> -n production
```

Then:

```bash
kubectl logs <pod-name> -n production
```

For the previous container:

```bash
kubectl logs <pod-name> -n production --previous
```

Monitor the restart rate to detect recurring crashes.

---

# 16. Pod Availability

For a Deployment:

```text
Desired replicas = 3
Available replicas = 3
```

Healthy:

```text
3 / 3
```

Problem:

```text
3 desired
2 available
```

This indicates that one replica is unavailable.

Monitor:

```text
Desired replicas
Available replicas
Ready replicas
Unavailable replicas
```

---

# 17. Deployment Monitoring

Important Deployment signals:

```text
Desired replicas
Updated replicas
Available replicas
Ready replicas
Unavailable replicas
Deployment conditions
```

Check:

```bash
kubectl get deployment -n production
```

Example:

```text
NAME       READY   UP-TO-DATE   AVAILABLE
orders     3/3     3            3
payment    3/3     3            3
```

---

# 18. StatefulSet Monitoring

Stateful applications require additional monitoring.

Check:

```text
Ready replicas
Current replicas
Updated replicas
Pod identity
Persistent volumes
```

Example:

```bash
kubectl get statefulset -n production
```

Storage failures can cause StatefulSet workloads to become unavailable.

---

# 19. DaemonSet Monitoring

DaemonSets are commonly used for:

```text
Node agents
Logging agents
Monitoring agents
Security agents
```

Monitor:

```text
Desired
Current
Ready
Available
```

Example:

```bash
kubectl get daemonset -n observability
```

If the DaemonSet is expected to run on 10 nodes but only 8 Pods are ready, investigate the missing two.

---

# 20. Service Monitoring

A Kubernetes Service provides network access to Pods.

Monitor:

```text
Service existence
Endpoints
EndpointSlices
Port configuration
Backend availability
```

Check:

```bash
kubectl get svc -n production
```

Then:

```bash
kubectl get endpoints -n production
```

A Service without healthy endpoints can result in application connectivity failures.

---

# 21. Service With No Endpoints

Example:

```text
Service
   ↓
No endpoints
```

Possible causes:

```text
Pod labels do not match Service selector
Pods are not Ready
Pods are not running
Incorrect namespace
Deployment failure
```

This is a common Kubernetes troubleshooting scenario.

---

# 22. Node Monitoring

Nodes provide the compute capacity for workloads.

Monitor:

```text
CPU
Memory
Disk
Network
Pod capacity
Node conditions
```

Check:

```bash
kubectl get nodes
```

Detailed information:

```bash
kubectl describe node <node-name>
```

Resource usage:

```bash
kubectl top nodes
```

---

# 23. Node Conditions

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
Ready = True
MemoryPressure = False
DiskPressure = False
PIDPressure = False
```

Changes in these conditions can indicate node-level problems.

---

# 24. Node CPU Saturation

Suppose:

```text
node-1 CPU = 95%
```

Possible effects:

```text
Pod performance degradation
CPU throttling
Scheduling pressure
Slow application response
```

Investigate:

```bash
kubectl top nodes
kubectl top pods --all-namespaces
```

Identify which workloads are consuming the most CPU.

---

# 25. Node Memory Pressure

If:

```text
MemoryPressure = True
```

investigate:

```text
Node memory usage
Pod memory usage
Memory requests
Memory limits
Evictions
```

A memory-pressure event can result in Kubernetes evicting Pods.

---

# 26. Disk Pressure

Disk pressure can occur because of:

```text
Container logs
Container images
Temporary files
Ephemeral storage
Application data
```

Check:

```bash
kubectl describe node <node-name>
```

Look for:

```text
DiskPressure
```

Node disk monitoring is important because disk exhaustion can affect both applications and Kubernetes itself.

---

# 27. Network Monitoring

Monitor:

```text
Network receive
Network transmit
Packet errors
Connection failures
Service latency
DNS failures
```

At the application level:

```text
Request rate
Error rate
Latency
```

At the Kubernetes level:

```text
Pod-to-Pod connectivity
Service connectivity
DNS
Ingress
Load Balancer
```

---

# 28. DNS Monitoring

Kubernetes applications frequently depend on CoreDNS.

Architecture:

```text
Application
    ↓
CoreDNS
    ↓
Kubernetes Service
```

Monitor:

```text
DNS request rate
DNS latency
DNS errors
CoreDNS CPU
CoreDNS memory
CoreDNS availability
```

If DNS fails, many otherwise healthy applications can become unreachable.

---

# 29. CoreDNS Monitoring

Check:

```bash
kubectl get pods -n kube-system
```

Find CoreDNS Pods:

```text
coredns-xxxxx
coredns-yyyyy
```

Then inspect:

```bash
kubectl logs <coredns-pod> -n kube-system
```

Also monitor CoreDNS metrics through the cluster monitoring system.

---

# 30. Kubernetes API Server Monitoring

The API Server is a critical control-plane component.

Monitor:

```text
Request rate
Request latency
Request errors
HTTP status codes
API availability
```

High API Server latency can affect:

```text
kubectl commands
Controllers
Schedulers
Operators
Deployments
Autoscaling
```

---

# 31. Scheduler Monitoring

The Kubernetes Scheduler decides where Pods should run.

Monitor:

```text
Scheduling latency
Scheduling failures
Pending Pods
Scheduler errors
```

Example problem:

```text
Pod
 ↓
Pending
 ↓
No suitable node
```

Possible causes:

```text
Insufficient CPU
Insufficient memory
Taints
Node selectors
Affinity
Topology constraints
```

---

# 32. Controller Monitoring

Controllers continuously reconcile desired and actual state.

Examples:

```text
Deployment Controller
ReplicaSet Controller
StatefulSet Controller
DaemonSet Controller
Job Controller
```

Monitor controller health because controller failures can prevent Kubernetes resources from reaching the desired state.

---

# 33. kubelet Monitoring

The kubelet runs on each Kubernetes node.

It manages:

```text
Pod lifecycle
Container lifecycle
Health probes
Volume mounts
Node status
```

If kubelet becomes unhealthy:

```text
Node
 ↓
Kubelet problem
 ↓
Pod management affected
```

Monitor kubelet health and relevant node metrics.

---

# 34. Container Monitoring

Containers should be monitored for:

```text
CPU
Memory
Restart count
Network
Filesystem
Exit codes
OOMKilled
```

Container-level metrics help identify the exact workload consuming resources.

---

# 35. Application Metrics

Infrastructure metrics alone are not enough.

Monitor application-level metrics such as:

```text
Request rate
Error rate
Latency
Throughput
Active connections
Queue depth
Business transactions
```

A useful model is:

```text
Application Metrics
        ↓
Kubernetes Metrics
        ↓
Infrastructure Metrics
```

---

# 36. The Four Golden Signals

For applications running on Kubernetes, monitor:

```text
Latency
Traffic
Errors
Saturation
```

Example:

```text
Latency  → Request duration
Traffic  → Requests/sec
Errors   → HTTP 5xx
Saturation → CPU / memory / queue
```

These signals provide a high-level view of application health.

---

# 37. RED Method

For request-driven applications:

```text
R = Rate
E = Errors
D = Duration
```

Example:

```text
Rate      = 2,000 requests/sec
Errors    = 1.5%
Duration  = p95 450ms
```

This is useful for monitoring microservices.

---

# 38. USE Method

For infrastructure resources:

```text
U = Utilization
S = Saturation
E = Errors
```

Example for a node:

```text
Utilization → CPU 80%
Saturation  → Run queue high
Errors      → Network errors
```

RED is useful for services, while USE is useful for infrastructure resources.

---

# 39. Kubernetes Events

Events provide valuable troubleshooting information.

Check:

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

Events may show:

```text
FailedScheduling
FailedMount
BackOff
Unhealthy
Killing
Evicted
Pulling
Pulled
```

Events are useful for understanding what Kubernetes is doing at a particular point in time.

---

# 40. Event Monitoring Limitations

Kubernetes Events are useful but should not be treated as a complete historical monitoring system.

Events can be:

```text
Transient
Short-lived
Limited in retention
```

Therefore:

```text
Events → Immediate troubleshooting
Prometheus → Historical metrics
ELK → Centralized logs
```

---

# 41. Prometheus for Kubernetes Monitoring

Prometheus is commonly used to collect Kubernetes metrics.

Architecture:

```text
Kubernetes
    ↓
Metrics
    ↓
Prometheus
    ↓
Grafana
```

Prometheus can monitor:

```text
Nodes
Pods
Containers
Kubernetes objects
Applications
Cluster components
```

---

# 42. kube-state-metrics

`kube-state-metrics` exposes Kubernetes object state as Prometheus metrics.

It provides information about objects such as:

```text
Deployments
Pods
StatefulSets
DaemonSets
Nodes
Jobs
Namespaces
```

Conceptually:

```text
Kubernetes API
      ↓
kube-state-metrics
      ↓
Prometheus
      ↓
Grafana
```

---

# 43. Metrics Server vs Prometheus

These tools serve different purposes.

### Metrics Server

Primarily provides resource metrics used by Kubernetes features such as:

```text
kubectl top
HPA resource metrics
```

### Prometheus

Provides:

```text
Time-series storage
Historical metrics
Queries
Dashboards
Alerting integration
```

Therefore:

```text
Metrics Server ≠ Prometheus
```

---

# 44. Monitoring Stack

A typical Kubernetes monitoring stack can contain:

```text
Prometheus
Grafana
kube-state-metrics
Node Exporter
Alertmanager
```

Architecture:

```text
Kubernetes
│
├── kube-state-metrics
├── Node Exporter
├── Application Metrics
│
└───────────────→ Prometheus
                         ↓
                      Grafana
                         ↓
                    Alertmanager
```

---

# 45. Node Exporter

Node Exporter exposes host-level metrics.

Typical metrics include:

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
```

It is commonly deployed as a DaemonSet so each node has an exporter.

---

# 46. Application Metrics

Applications can expose Prometheus-compatible metrics.

Example endpoint:

```text
/metrics
```

Prometheus scrapes the endpoint.

```text
Application
   ↓
/metrics
   ↓
Prometheus
   ↓
Grafana
```

This allows Kubernetes infrastructure monitoring and application monitoring to be combined.

---

# 47. Service Discovery

Prometheus can discover Kubernetes targets dynamically.

Conceptually:

```text
Kubernetes API
      ↓
Prometheus Service Discovery
      ↓
Targets
      ↓
Metrics
```

This is useful because Pods and Services are constantly changing.

---

# 48. Pod Lifecycle Monitoring

Monitor Pod states:

```text
Pending
Running
Succeeded
Failed
Unknown
```

A healthy production workload should generally have the expected number of Pods in `Running` and `Ready` states.

---

# 49. Pending Pods

A Pod remaining Pending for a long time is an important alert.

Possible causes:

```text
Insufficient CPU
Insufficient memory
Taints
Affinity rules
Node selectors
PVC problems
Topology constraints
```

Investigation:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Check the Events section.

---

# 50. Container Restart Monitoring

An increasing restart count can indicate:

```text
CrashLoopBackOff
OOMKilled
Probe failures
Application crashes
Configuration errors
```

A useful monitoring strategy is to alert on restart rate rather than only absolute restart count.

---

# 51. OOMKilled Monitoring

Monitor for:

```text
Container memory limit exceeded
```

Investigation:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
Last State:
  Terminated
  Reason: OOMKilled
```

Then compare:

```text
Actual memory usage
Memory request
Memory limit
```

---

# 52. Probe Monitoring

Monitor:

```text
Liveness failures
Readiness failures
Startup failures
```

A readiness failure can cause:

```text
Pod
 ↓
Removed from Service endpoints
 ↓
Traffic decreases
```

A liveness failure can cause:

```text
Container
 ↓
Restart
```

Frequent probe failures should be investigated rather than simply increasing probe thresholds.

---

# 53. HPA Monitoring

Horizontal Pod Autoscaler should be monitored for:

```text
Current replicas
Desired replicas
CPU utilization
Memory utilization
Custom metrics
Scaling events
```

Example:

```text
Current replicas = 3
Desired replicas = 8
```

If the workload remains at 3 despite high demand, investigate the HPA configuration and metrics source.

---

# 54. HPA Failure Scenario

Example:

```text
CPU = 90%
HPA desired = 8
Current = 3
```

Possible causes:

```text
Metrics unavailable
Resource requests missing
HPA configuration issue
Maximum replicas reached
Cluster capacity unavailable
```

Monitoring should cover both the HPA and the cluster's ability to schedule new Pods.

---

# 55. Cluster Capacity Monitoring

Monitor:

```text
Total CPU
Allocatable CPU
Requested CPU
Used CPU
Total memory
Allocatable memory
Requested memory
Used memory
```

Example:

```text
Cluster CPU
Capacity   = 32 cores
Allocatable = 30 cores
Requested   = 27 cores
```

High requested capacity can prevent new workloads from being scheduled even if instantaneous CPU usage appears low.

---

# 56. Requests vs Actual Usage

This distinction is important.

Example:

```text
CPU request = 1 CPU
Actual usage = 200m
```

The scheduler considers the request when placing Pods.

Therefore, a cluster can experience:

```text
Low actual CPU usage
+
High requested CPU
=
Scheduling problems
```

Monitor both resource requests and actual utilization.

---

# 57. Cluster Autoscaler Monitoring

If using Cluster Autoscaler, monitor:

```text
Node count
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
New node
     ↓
Pods scheduled
```

If scale-up does not happen, investigate autoscaler logs and cluster constraints.

---

# 58. Node Capacity

Monitor:

```text
CPU capacity
Memory capacity
Pod capacity
Ephemeral storage
```

A node can have CPU and memory available but still be unable to schedule a Pod because it has reached its Pod capacity.

---

# 59. Storage Monitoring

Kubernetes storage monitoring includes:

```text
PersistentVolume
PersistentVolumeClaim
StorageClass
Disk usage
Volume health
Provisioning failures
```

Check:

```bash
kubectl get pvc -A
```

A PVC stuck in:

```text
Pending
```

requires investigation.

---

# 60. PVC Monitoring

Example:

```text
PVC
 ↓
Pending
```

Possible causes:

```text
No matching StorageClass
Provisioner failure
Insufficient storage
Access mode mismatch
Zone constraints
```

Storage monitoring is critical for stateful applications.

---

# 61. Ingress Monitoring

For an ingress/load balancer layer, monitor:

```text
Request rate
Latency
HTTP 4xx
HTTP 5xx
Backend health
Connection errors
```

Example:

```text
Users
  ↓
ALB / Ingress
  ↓
Service
  ↓
Pods
```

A 503 can originate from several layers.

---

# 62. 503 Troubleshooting

If users receive:

```text
503 Service Unavailable
```

check:

```text
Ingress
 ↓
Service
 ↓
Endpoints
 ↓
Pods
 ↓
Application
```

Commands:

```bash
kubectl get ingress -A
kubectl get svc -A
kubectl get endpoints -A
kubectl get pods -A
```

Then correlate with application metrics and logs.

---

# 63. Network Policy Monitoring

NetworkPolicies can intentionally block traffic.

If:

```text
Application A
      X
Application B
```

check:

```text
NetworkPolicy
Service
Endpoints
DNS
Ports
Protocols
```

Network observability should help distinguish policy failures from application failures.

---

# 64. Namespace Monitoring

Monitor resource consumption by namespace.

Example:

```text
production
├── CPU
├── Memory
├── Pods
└── Restarts

staging
├── CPU
├── Memory
├── Pods
└── Restarts
```

This helps identify which teams or environments are consuming cluster resources.

---

# 65. Resource Quotas

Namespaces can have ResourceQuotas.

Monitor:

```text
CPU requests
CPU limits
Memory requests
Memory limits
Pod count
Storage
```

A workload may fail to schedule because the namespace quota is exhausted even when the cluster has available capacity.

---

# 66. LimitRanges

LimitRanges can define default resource requests and limits.

Monitoring should account for:

```text
Default requests
Default limits
Container limits
Namespace policies
```

Unexpected defaults can affect scheduling and resource utilization.

---

# 67. Monitoring Kubernetes Jobs

Jobs should be monitored for:

```text
Successful completions
Failed jobs
Active jobs
Retries
Execution duration
```

Example:

```bash
kubectl get jobs -A
```

A Job repeatedly failing can indicate an application or dependency problem.

---

# 68. CronJob Monitoring

CronJobs should be monitored for:

```text
Last schedule
Next schedule
Successful jobs
Failed jobs
Missed schedules
Execution duration
```

A scheduled workload can silently stop running if it is not monitored.

---

# 69. Cluster Health Dashboard

A useful Grafana dashboard can contain:

```text
Cluster Overview
│
├── Node Count
├── CPU Utilization
├── Memory Utilization
├── Pod Count
├── Pending Pods
├── Pod Restarts
├── API Server Latency
├── Scheduling Failures
└── Cluster Capacity
```

---

# 70. Node Dashboard

A node dashboard can contain:

```text
Node
├── CPU utilization
├── Memory utilization
├── Disk usage
├── Disk I/O
├── Network traffic
├── Pod count
├── Filesystem usage
└── Node conditions
```

This helps identify infrastructure bottlenecks.

---

# 71. Pod Dashboard

A Pod dashboard can contain:

```text
Pod
├── CPU
├── Memory
├── Restarts
├── Network
├── Status
├── Readiness
├── Container state
└── OOMKilled events
```

---

# 72. Application Dashboard

Application dashboards should contain:

```text
Request rate
Error rate
Latency
Saturation
Active requests
Dependency latency
```

For a microservice:

```text
Orders
├── RPS
├── p95 latency
├── p99 latency
├── 4xx
├── 5xx
└── Dependency latency
```

---

# 73. Monitoring vs Observability

Monitoring generally answers:

```text
Is something wrong?
```

Observability helps answer:

```text
Why is it wrong?
```

For example:

```text
Monitoring:
CPU = 95%

Observability:
Which Pod?
Which request?
Which trace?
Which dependency?
Which log?
Why did CPU increase?
```

---

# 74. Three Pillars

Kubernetes observability commonly combines:

```text
Metrics
Logs
Traces
```

Architecture:

```text
             Kubernetes
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
    Metrics     Logs     Traces
       ↓         ↓         ↓
 Prometheus      ELK      Jaeger
       ↓                   ↓
    Grafana             Jaeger UI
```

---

# 75. Metrics → Logs → Traces

A practical investigation:

```text
Metric:
High 5xx
   ↓
Log:
Application timeout
   ↓
Trace:
Payment dependency slow
```

This is more powerful than relying on a single telemetry source.

---

# 76. Kubernetes Monitoring Workflow

When an alert fires:

```text
1. Identify affected service
2. Check application metrics
3. Check Pod status
4. Check restarts
5. Check resource usage
6. Check Service endpoints
7. Check Node health
8. Check Kubernetes events
9. Check logs
10. Check traces
11. Identify root cause
12. Remediate
13. Verify recovery
```

---

# 77. Production Incident Example

Problem:

```text
Users receive 503 errors.
```

Start with:

```text
Grafana
 ↓
5xx rate increased
```

Check Kubernetes:

```text
Service
 ↓
Endpoints
 ↓
Pods
```

Suppose Pods are:

```text
Running
Ready
```

Then check:

```text
Application logs
```

Suppose logs show:

```text
Database connection timeout
```

Jaeger may show:

```text
Application
 ↓
Database
 ↓
1.8s
```

Now the investigation has moved from Kubernetes to the database dependency.

---

# 78. Production Incident: OOMKilled

Problem:

```text
Application restarting repeatedly.
```

Check:

```bash
kubectl get pods -n production
```

Then:

```bash
kubectl describe pod <pod-name> -n production
```

Find:

```text
OOMKilled
```

Then compare:

```text
Memory usage
Memory request
Memory limit
```

Next investigate:

```text
Memory leak
Traffic increase
Large workload
Incorrect limits
Application behavior
```

---

# 79. Production Incident: Pending Pod

Problem:

```text
Pod Pending
```

Run:

```bash
kubectl describe pod <pod-name> -n production
```

Events may show:

```text
Insufficient cpu
```

Check:

```bash
kubectl top nodes
kubectl get nodes
```

Then evaluate:

```text
Node capacity
Pod requests
Cluster Autoscaler
Scheduling constraints
```

---

# 80. Production Incident: Node Memory Pressure

Problem:

```text
Node MemoryPressure = True
```

Check:

```bash
kubectl top node <node-name>
kubectl top pods -A
```

Identify high-memory workloads.

Then inspect:

```text
Memory requests
Memory limits
Node allocatable memory
Evictions
```

---

# 81. Alert Design

Avoid alerting on every small change.

Bad:

```text
CPU > 50%
```

This can generate excessive noise.

Better:

```text
CPU > 90%
for sustained period
AND
application latency is increasing
```

The exact thresholds depend on workload behavior.

---

# 82. Alert Severity

A useful model:

```text
Critical
 ↓
Immediate user/business impact

Warning
 ↓
Potential future problem

Info
 ↓
Operational information
```

Example:

```text
Critical:
Multiple replicas unavailable

Warning:
Node CPU sustained above threshold

Info:
Deployment completed
```

---

# 83. SLO-Based Monitoring

Instead of monitoring only infrastructure thresholds, monitor service objectives.

Example:

```text
Availability SLO = 99.9%
Latency SLO = p95 < 500ms
```

Then monitor:

```text
Error budget
Latency
Availability
Burn rate
```

This connects Kubernetes monitoring to actual user experience.

---

# 84. Monitoring Anti-Patterns

Avoid:

```text
Monitoring only Nodes
Monitoring only CPU
Monitoring only Pod status
No historical metrics
No alerting
No application metrics
No logs
No traces
No capacity planning
```

A Pod being `Running` does not mean the application is healthy.

---

# 85. Kubernetes Monitoring Best Practices

```text
1. Monitor every important layer.
2. Monitor both utilization and saturation.
3. Monitor resource requests and actual usage.
4. Monitor Pod restarts.
5. Monitor readiness and liveness failures.
6. Monitor node conditions.
7. Monitor cluster capacity.
8. Monitor application RED metrics.
9. Use Prometheus for time-series metrics.
10. Use Grafana for visualization.
11. Use ELK for centralized logs.
12. Use Jaeger for distributed tracing.
13. Correlate metrics, logs, and traces.
14. Alert on symptoms that require action.
15. Continuously review alert quality.
```

---

# 86. Kubernetes Monitoring Commands

### Cluster

```bash
kubectl get nodes
kubectl top nodes
```

### Pods

```bash
kubectl get pods -A
kubectl top pods -A
```

### Deployments

```bash
kubectl get deployments -A
```

### Services

```bash
kubectl get svc -A
```

### Endpoints

```bash
kubectl get endpoints -A
```

### Events

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

### Resource Details

```bash
kubectl describe pod <pod-name> -n <namespace>
kubectl describe node <node-name>
```

---

# 87. Interview Question

### How do you monitor a Kubernetes cluster?

**Answer:**

I monitor the cluster at multiple layers. At the node level, I monitor CPU, memory, disk, network, node conditions, and capacity. At the workload level, I monitor Pod availability, restarts, resource usage, readiness, liveness, Deployments, StatefulSets, and DaemonSets. At the application level, I monitor request rate, errors, latency, and saturation. I use Prometheus for metrics, Grafana for dashboards and alerts, ELK for logs, and Jaeger for distributed tracing.

---

# 88. Interview Question

### What would you monitor for Kubernetes Pods?

**Answer:**

I would monitor Pod status, readiness, restart count, CPU, memory, network usage, container exit codes, OOMKilled events, and probe failures. I would also monitor desired versus available replicas at the Deployment level.

---

# 89. Interview Question

### How would you troubleshoot a Pod that keeps restarting?

**Answer:**

I would first check the Pod status and restart count:

```bash
kubectl get pod <pod-name>
```

Then inspect:

```bash
kubectl describe pod <pod-name>
```

and:

```bash
kubectl logs <pod-name> --previous
```

I would check for OOMKilled, application crashes, liveness probe failures, configuration errors, missing secrets, or dependency failures. Then I would correlate the issue with application metrics and logs.

---

# 90. Interview Question

### What is the difference between Metrics Server and Prometheus?

**Answer:**

Metrics Server primarily provides resource metrics such as CPU and memory for Kubernetes use cases like `kubectl top` and resource-based HPA. Prometheus is a full monitoring system that collects and stores time-series metrics, supports PromQL, historical analysis, dashboards through Grafana, and alerting integrations.

---

# 91. Interview Question

### How would you troubleshoot a Pending Pod?

**Answer:**

I would run:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

and check the Events section. I would look for insufficient CPU or memory, taints, node selectors, affinity rules, topology constraints, PVC issues, or namespace resource quotas. Then I would compare Pod resource requests with available node capacity.

---

# 92. Interview Question

### How would you monitor Kubernetes nodes?

**Answer:**

I monitor CPU, memory, disk, filesystem, network, Pod count, node conditions, and allocatable capacity. I also monitor `MemoryPressure`, `DiskPressure`, `PIDPressure`, node readiness, and resource saturation. Prometheus and Node Exporter can provide historical metrics, while `kubectl top nodes` is useful for immediate inspection.

---

# 93. Interview Question

### How do you monitor Kubernetes application health?

**Answer:**

I don't rely only on Kubernetes Pod status. I monitor application RED metrics—request rate, error rate, and duration—along with Pod readiness, resource usage, restart rates, dependency latency, logs, and distributed traces. This allows me to determine whether a running Pod is actually serving users correctly.

---

# 94. Interview Question

### How would you troubleshoot a 503 from Kubernetes?

**Answer:**

I would trace the request path from the ingress or load balancer to the Service, then to its endpoints and Pods:

```text
Ingress / ALB
     ↓
Service
     ↓
Endpoints
     ↓
Pods
     ↓
Application
```

I would verify that the Service has healthy endpoints, Pods are Ready, readiness probes are passing, and the application is listening on the expected port. Then I would check application logs, metrics, and traces to identify the underlying failure.

---

# 95. Interview Question

### How do you monitor Kubernetes capacity?

**Answer:**

I monitor total and allocatable CPU and memory, resource requests, actual utilization, Pod capacity, and node count. I also monitor Pending Pods and cluster autoscaling activity. This helps distinguish actual resource saturation from scheduling problems caused by high resource requests or other constraints.

---

# 96. Interview Question

### What is the difference between monitoring and observability?

**Answer:**

Monitoring tells me that a system is behaving abnormally, while observability helps me investigate why. For example, monitoring might show that latency increased, while observability allows me to correlate metrics with a specific trace, application log, Pod, and downstream dependency to identify the root cause.

---

# 97. Production Monitoring Architecture

```text
                         Kubernetes / EKS
                                │
         ┌──────────────────────┼──────────────────────┐
         ↓                      ↓                      ↓
      Nodes                  Workloads             Applications
         │                      │                      │
         └──────────────────────┼──────────────────────┘
                                ↓
                         Metrics Collection
                                │
                ┌───────────────┼───────────────┐
                ↓               ↓               ↓
          Node Exporter   kube-state-metrics   App Metrics
                └───────────────┼───────────────┘
                                ↓
                            Prometheus
                                ↓
                             Grafana
                                ↓
                            Alerting
```

---

# 98. Complete Kubernetes Observability Model

```text
                         KUBERNETES
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       Metrics               Logs               Traces
          ↓                   ↓                   ↓
     Prometheus              ELK              OpenTelemetry
          ↓                                       ↓
       Grafana                                  Jaeger
          │                                       │
          └───────────────────┬───────────────────┘
                              ↓
                       Incident Response
                              ↓
                          Root Cause
```

---

# 99. Final Mental Model

Remember Kubernetes monitoring using four layers:

```text
1. CLUSTER
   ├── Nodes
   ├── API Server
   ├── Scheduler
   └── Controllers

2. WORKLOADS
   ├── Pods
   ├── Deployments
   ├── StatefulSets
   └── DaemonSets

3. RESOURCES
   ├── CPU
   ├── Memory
   ├── Disk
   └── Network

4. APPLICATION
   ├── Rate
   ├── Errors
   ├── Latency
   └── Saturation
```

Then combine the telemetry:

```text
Metrics
   ↓
Prometheus
   ↓
Grafana

Logs
   ↓
ELK

Traces
   ↓
OpenTelemetry
   ↓
Jaeger
```

The key principle is:

**Kubernetes monitoring should not stop at checking whether Pods are Running. A production monitoring strategy must cover cluster health, node resources, workload availability, container behavior, Kubernetes objects, application performance, networking, storage, and control-plane health. Prometheus provides historical metrics, Grafana provides visualization and alerting, ELK provides centralized logs, and Jaeger provides distributed traces. Together, these signals provide the visibility required to detect problems quickly and perform reliable root-cause analysis.**
