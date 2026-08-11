# Pod Monitoring

## 1. Overview

Pod monitoring focuses on the health, resource usage, lifecycle, availability, and behavior of individual Kubernetes Pods.

A Pod is the smallest deployable unit in Kubernetes and may contain one or more containers.

```text
Pod
├── Container
├── Container
└── Shared networking / storage
```

For production monitoring, a Pod should be evaluated across:

```text
Pod Monitoring
│
├── Availability
├── Status
├── CPU
├── Memory
├── Restarts
├── Probes
├── Events
├── Scheduling
├── Networking
├── Storage
└── Application behavior
```

---

# 2. Why Pod Monitoring Is Important

A Kubernetes cluster can appear healthy while individual workloads are failing.

Example:

```text
Cluster
   ↓
Healthy

Payment Pods
   ↓
2 Running
3 CrashLoopBackOff
```

The cluster itself may be available, but the application is degraded.

Pod monitoring helps answer:

```text
Is the Pod running?
Is it Ready?
Is it restarting?
Is it consuming too many resources?
Was it OOMKilled?
Is it Pending?
Can it reach dependencies?
Are probes failing?
Is it receiving traffic?
```

---

# 3. Pod Monitoring Architecture

A typical monitoring flow is:

```text
                    Kubernetes Pod
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
      Container        Kubelet        Pod State
          │               │               │
          ↓               ↓               ↓
    CPU / Memory      Runtime Data   kube-state-metrics
          │               │               │
          └───────────────┼───────────────┘
                          ↓
                      Prometheus
                          ↓
                        Grafana
                          ↓
                       Alerts
```

Different components provide different types of information.

---

# 4. Pod Health Dimensions

Pod health should not be determined only by the `Running` state.

A better model is:

```text
Pod Health
│
├── Scheduled?
├── Running?
├── Ready?
├── Containers healthy?
├── Restarting?
├── Resource usage normal?
├── Probes passing?
├── Dependencies reachable?
└── Receiving expected traffic?
```

A Pod can be:

```text
Running = True
Ready   = False
```

and therefore still be unavailable to application traffic.

---

# 5. Pod Lifecycle

A Pod generally moves through phases such as:

```text
Pending
   ↓
Running
   ↓
Succeeded / Failed
```

Common Pod phases:

```text
Pending
Running
Succeeded
Failed
Unknown
```

These phases provide high-level lifecycle information.

---

# 6. Pending Pods

A Pending Pod has not reached the Running phase.

Possible causes:

```text
Insufficient CPU
Insufficient memory
Taints
Node affinity
Pod affinity
Node selector
PVC problems
Image pull prerequisites
Scheduling constraints
```

Example:

```text
Deployment
   ↓
Desired replicas = 5
   ↓
Only 4 scheduled
   ↓
1 Pod Pending
```

A growing number of Pending Pods is an important monitoring signal.

---

# 7. Running Pods

A Running Pod has at least one container running.

However:

```text
Running ≠ Healthy
```

Example:

```text
Pod = Running
Ready = False
```

The application may still be unavailable.

Always combine Pod phase with:

```text
Readiness
Container status
Probe results
Restart count
Application metrics
```

---

# 8. Succeeded Pods

Succeeded usually means that all containers completed successfully.

This is common for:

```text
Jobs
Batch workloads
One-time tasks
Migration workloads
```

Example:

```text
Job
 ↓
Pod
 ↓
Task completes
 ↓
Succeeded
```

A Succeeded Pod is not necessarily a problem.

---

# 9. Failed Pods

A Failed Pod means its containers terminated and the Pod reached a failed phase.

Investigate:

```text
Exit code
Termination reason
Application logs
Events
Resource conditions
Dependencies
```

Example:

```bash
kubectl describe pod <pod-name>
```

and:

```bash
kubectl logs <pod-name>
```

---

# 10. Pod Ready Condition

Readiness is one of the most important Pod health indicators.

Example:

```text
Pod
├── Running = True
└── Ready = False
```

A Pod that is not Ready should normally not receive Service traffic.

Therefore monitor:

```text
Ready Pods
Not Ready Pods
Readiness failures
Ready percentage
```

---

# 11. Ready vs Running

Consider:

```text
5 Pods
```

Status:

```text
Running = 5
Ready   = 3
```

The application effectively has only three ready instances.

Possible consequences:

```text
Reduced capacity
Higher latency
Traffic failures
Insufficient redundancy
```

This is why alerts should not rely only on Running status.

---

# 12. Container Status

A Pod may contain multiple containers.

Monitor each container's state:

```text
Running
Waiting
Terminated
```

Example:

```text
Pod
├── application → Running
└── sidecar     → Waiting
```

The Pod may not be operational even though one container is running.

---

# 13. Waiting State

A container may be waiting because of:

```text
ImagePullBackOff
ErrImagePull
CrashLoopBackOff
ContainerCreating
CreateContainerConfigError
CreateContainerError
```

The exact reason should be inspected through:

```bash
kubectl describe pod <pod-name>
```

and Events.

---

# 14. Terminated State

A container may terminate because of:

```text
Completed
Error
OOMKilled
Signal
Application exit
```

Useful information includes:

```text
Exit code
Reason
Started time
Finished time
```

For unexpected termination, correlate this with logs and metrics.

---

# 15. Restart Count

Restart count is an important Pod signal.

Example:

```text
RESTARTS
0
1
2
3
20
```

A continuously increasing restart count is concerning.

However, an old Pod with one restart may not indicate a current problem.

Prefer monitoring:

```text
Restart rate
```

rather than only the absolute restart count.

---

# 16. Restart Rate

Conceptually:

```text
Restart count
      ↓
Time
      ↓
Restart rate
```

Example:

```text
10 restarts
in 5 minutes
```

is much more concerning than:

```text
10 restarts
over 30 days
```

Use time-based metrics and alerts where possible.

---

# 17. CrashLoopBackOff

CrashLoopBackOff occurs when a container repeatedly fails and Kubernetes backs off before restarting it.

Typical sequence:

```text
Container starts
     ↓
Application crashes
     ↓
Container exits
     ↓
Kubernetes restarts
     ↓
Application crashes again
     ↓
Backoff increases
```

Monitor:

```text
Restart rate
Container termination reason
Pod state
Logs
Events
```

---

# 18. CrashLoopBackOff Troubleshooting

Start with:

```bash
kubectl get pods -n production
```

Then:

```bash
kubectl describe pod <pod-name> -n production
```

Check logs:

```bash
kubectl logs <pod-name> -n production
```

If the container restarted:

```bash
kubectl logs <pod-name> -n production --previous
```

Then investigate:

```text
Configuration
Secrets
ConfigMaps
Dependencies
Probes
CPU
Memory
Application errors
```

---

# 19. OOMKilled

OOMKilled means the container was terminated because it exceeded available memory conditions.

A common sequence:

```text
Memory usage ↑
      ↓
Container reaches limit
      ↓
Container terminated
      ↓
Restart
```

Monitor:

```text
Memory usage
Memory limit
Restart rate
OOMKilled events
```

---

# 20. CPU Monitoring

For each Pod monitor:

```text
CPU usage
CPU request
CPU limit
CPU throttling
```

Example:

```text
CPU request = 250m
CPU limit   = 1
Usage       = 800m
```

This Pod is consuming significantly more CPU than its request but remains below its limit.

---

# 21. CPU Throttling

CPU limits can result in throttling.

Example:

```text
CPU demand
    ↓
Configured CPU limit
    ↓
Throttling
```

Symptoms may include:

```text
Higher latency
Slower processing
Reduced throughput
```

Therefore monitor CPU usage together with throttling rather than assuming high CPU usage is the only issue.

---

# 22. Memory Monitoring

Monitor:

```text
Memory working set
Memory request
Memory limit
Memory pressure
OOM events
```

Example:

```text
Memory request = 512Mi
Memory limit   = 1Gi
Usage          = 900Mi
```

The Pod is approaching its memory limit.

If this is a persistent pattern, investigate whether the workload is correctly sized.

---

# 23. Requests vs Actual Usage

Resource requests affect scheduling.

Example:

```text
CPU request = 2 CPU
Actual usage = 200m
```

This may result in inefficient capacity usage.

Conversely:

```text
CPU request = 200m
Actual usage = 900m
```

may indicate under-requesting.

Monitor both:

```text
Requested resources
Actual usage
```

---

# 24. Resource Limits

Limits provide an upper boundary for resource consumption.

Example:

```yaml
resources:
  limits:
    cpu: "1"
    memory: 1Gi
```

Incorrect limits can cause:

```text
CPU throttling
OOMKilled
Performance degradation
```

Resource monitoring should therefore be combined with workload behavior.

---

# 25. Pod Resource Efficiency

A useful dashboard can compare:

```text
CPU usage / CPU request
Memory usage / Memory request
```

Example:

```text
Payment
CPU:
Request = 500m
Usage   = 450m

Memory:
Request = 512Mi
Usage   = 480Mi
```

This workload is using resources close to its requests.

---

# 26. Pod Scheduling

Pod monitoring should also consider scheduling.

Important signals:

```text
Scheduled
Node assigned
Scheduling failures
Pending duration
```

A Pod stuck in Pending may indicate:

```text
Capacity issue
Affinity issue
Taint
Toleration problem
Storage issue
```

---

# 27. Pod Placement

Monitor where Pods are running.

Example:

```text
Deployment = 6 Pods

Node-1 → 5 Pods
Node-2 → 1 Pod
Node-3 → 0 Pods
```

This may create poor distribution.

Potential risks:

```text
Node failure
Uneven resource consumption
Reduced availability
```

---

# 28. Pod Distribution

For production workloads, consider:

```text
Topology spread
Pod anti-affinity
Availability Zones
Node groups
```

Example:

```text
AZ-A → 2 Pods
AZ-B → 2 Pods
AZ-C → 2 Pods
```

This is generally more resilient than placing all replicas on one Node or AZ.

---

# 29. Node Failure Impact

Suppose:

```text
Payment replicas = 5

Node-1:
Payment-1
Payment-2
Payment-3

Node-2:
Payment-4

Node-3:
Payment-5
```

If Node-1 fails:

```text
3 Pods lost
```

The workload may become temporarily degraded.

Pod monitoring should therefore be combined with placement awareness.

---

# 30. Readiness Probes

Readiness probes determine whether a Pod is ready to receive traffic.

Example:

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
```

If the probe fails:

```text
Ready = False
```

The Service should stop sending new traffic to that endpoint.

---

# 31. Liveness Probes

Liveness probes determine whether a container should be restarted.

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

Repeated failures may cause:

```text
Container restart
```

Monitor:

```text
Probe failures
Restart rate
Application health
```

---

# 32. Startup Probes

Startup probes protect slow-starting applications.

Example:

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
```

This prevents liveness checks from restarting an application before it has completed startup.

Monitor:

```text
Startup duration
Startup failures
Restart during initialization
```

---

# 33. Probe Failure Analysis

Probe failures can be caused by:

```text
Application startup delay
Incorrect path
Incorrect port
Network issue
Resource pressure
Dependency failure
Aggressive timeout
Incorrect probe configuration
```

Do not automatically increase probe timeout without identifying the actual cause.

---

# 34. Pod Events

Events are extremely useful for troubleshooting.

Use:

```bash
kubectl describe pod <pod-name>
```

Look at:

```text
Events
```

Common events:

```text
FailedScheduling
FailedMount
Failed
BackOff
Unhealthy
Pulling
Pulled
Created
Started
Killing
```

---

# 35. Pod Logs vs Events

### Logs

Tell you:

```text
What the application reported
```

### Events

Tell you:

```text
What Kubernetes observed or did
```

Example:

```text
Application logs:
Database connection failed

Kubernetes event:
Readiness probe failed
```

Both are valuable.

---

# 36. Pod Logs vs Metrics vs Events

Use:

```text
Metrics
→ How much / how often?

Logs
→ What happened?

Events
→ What did Kubernetes do?

Traces
→ Where did the request spend time?
```

Together they provide much stronger troubleshooting capability.

---

# 37. Pod Network Monitoring

Monitor:

```text
Network receive
Network transmit
Packets
Errors
Drops
Connections
```

For application Pods also consider:

```text
Request rate
Network latency
Service connectivity
DNS
Ingress traffic
```

---

# 38. Network Errors

A Pod can be Running and Ready while experiencing network problems.

Example:

```text
Pod = Running
Ready = True

But:

Database connections = failing
```

Investigate:

```text
DNS
NetworkPolicy
Security groups
Service
Endpoints
CNI
Database connectivity
```

---

# 39. Pod DNS Troubleshooting

A common issue is inability to resolve a Service.

Inside an appropriate troubleshooting environment:

```bash
nslookup <service-name>
```

or:

```bash
dig <service-name>
```

Check:

```text
CoreDNS
Service
Namespace
Network connectivity
```

Pod monitoring should correlate DNS failures with application errors.

---

# 40. Service Endpoint Monitoring

Pods become Service endpoints when they satisfy the required readiness conditions.

Conceptually:

```text
Pod
 ↓
Ready
 ↓
Endpoint
 ↓
Service
 ↓
Traffic
```

If:

```text
Pod = Running
Ready = False
```

it may not be included as a ready endpoint.

---

# 41. Pod Traffic

Monitor:

```text
Requests/sec
Connections
HTTP status
Latency
Bytes
```

A Pod receiving significantly more traffic than other replicas may indicate:

```text
Load-balancing issue
Uneven endpoints
Connection behavior
Long-lived connections
```

---

# 42. Uneven Pod Traffic

Suppose:

```text
Pod-1 → 10,000 requests
Pod-2 → 2,000 requests
Pod-3 → 1,800 requests
```

Investigate:

```text
Service endpoints
Connection reuse
Load balancing behavior
Pod readiness
Ingress configuration
Application behavior
```

Do not immediately assume Kubernetes is distributing traffic incorrectly.

---

# 43. Pod Storage Monitoring

Monitor Pods that use:

```text
PVC
Ephemeral storage
EmptyDir
Temporary files
```

Important signals:

```text
Storage usage
Volume availability
Mount failures
Filesystem capacity
```

A Pod can fail because storage is unavailable even when CPU and memory look healthy.

---

# 44. Ephemeral Storage

Containers may consume ephemeral storage through:

```text
Temporary files
Container logs
Writable container filesystem
EmptyDir volumes
```

Excessive usage can contribute to:

```text
DiskPressure
Pod eviction
Container failure
```

Monitor workloads that generate large temporary files or logs.

---

# 45. Pod Evictions

Pods can be evicted when Nodes experience resource pressure.

Possible conditions:

```text
MemoryPressure
DiskPressure
PIDPressure
```

Example:

```text
Node
 ↓
MemoryPressure
 ↓
Pod eviction
```

Monitor:

```text
Eviction count
Eviction reason
Affected workloads
Node conditions
```

---

# 46. Pod Disruption

Pods can also be disrupted through:

```text
Node maintenance
Cluster upgrades
Voluntary disruptions
Autoscaling
Application deployment
```

Production workloads should have enough replicas and appropriate disruption policies.

---

# 47. PodDisruptionBudget

A PodDisruptionBudget can protect application availability during voluntary disruptions.

Example:

```yaml
minAvailable: 2
```

If:

```text
Replicas = 3
```

then Kubernetes should avoid voluntarily disrupting too many Pods simultaneously, subject to the applicable disruption mechanisms.

Monitor:

```text
Allowed disruptions
Current healthy Pods
Desired healthy Pods
```

---

# 48. Deployment Availability

For a Deployment:

```text
Desired replicas
Available replicas
Ready replicas
Updated replicas
Unavailable replicas
```

Example:

```text
Desired    = 5
Available  = 5
Ready      = 5
Updated    = 5
```

Healthy.

Problem:

```text
Desired    = 5
Available  = 3
Ready      = 3
```

Investigate immediately according to the application's availability requirements.

---

# 49. Pod Monitoring With HPA

HPA changes the number of Pods.

```text
Traffic ↑
   ↓
CPU / Custom Metric ↑
   ↓
HPA
   ↓
Replicas ↑
```

Monitor:

```text
Current replicas
Desired replicas
CPU
Memory
Custom metrics
Pending Pods
```

If replicas increase but Pods remain Pending, investigate cluster capacity and scheduling.

---

# 50. Pod Monitoring With Cluster Autoscaler

Combined behavior:

```text
Traffic ↑
   ↓
HPA
   ↓
More Pods
   ↓
Insufficient Node capacity
   ↓
Pods Pending
   ↓
Cluster Autoscaler
   ↓
New Nodes
   ↓
Pods scheduled
```

Monitor both Pod and Node metrics to understand the complete scaling process.

---

# 51. Pod Monitoring Dashboard

A useful Pod dashboard can contain:

```text
Pod Overview
├── Total Pods
├── Running Pods
├── Ready Pods
├── Pending Pods
├── Failed Pods
├── Restart Rate
├── OOMKilled
├── CPU Usage
├── Memory Usage
├── CPU Throttling
├── Probe Failures
└── Evictions
```

Filters:

```text
Cluster
Namespace
Deployment
Service
Pod
Node
Environment
```

---

# 52. Namespace Pod Dashboard

Example:

```text
Production
│
├── Orders
│   ├── 5 Ready
│   └── 0 Restarts
│
├── Payment
│   ├── 4 Ready
│   └── 8 Restarts
│
└── Inventory
    ├── 3 Ready
    └── 0 Restarts
```

The Payment workload deserves immediate investigation.

---

# 53. Pod Health Score

A conceptual Pod health model can combine:

```text
Ready
+
Low restart rate
+
Normal CPU
+
Normal memory
+
Healthy probes
+
No critical events
+
Healthy network
```

Do not rely on one signal.

---

# 54. Pod Availability Percentage

For a workload:

```text
Ready Pods
────────────── × 100
Desired Pods
```

Example:

```text
Ready = 9
Desired = 10
```

Availability:

```text
90%
```

The acceptable threshold depends on the service's SLO and redundancy requirements.

---

# 55. Pod Age

Pod age can help identify:

```text
Frequent restarts
Unexpected recreation
Deployment behavior
Autoscaling
Evictions
```

Example:

```text
Pod age:
5 minutes
```

If Pods are repeatedly recreated, investigate:

```text
Deployment
Node
Probes
Application
OOM
Evictions
```

---

# 56. Pod Churn

Pod churn means Pods are repeatedly created and terminated.

Example:

```text
Pod-1 created
Pod-1 deleted
Pod-2 created
Pod-2 deleted
Pod-3 created
```

Possible causes:

```text
CrashLoopBackOff
Deployment changes
HPA
Node failures
Evictions
Preemption
```

High Pod churn can indicate instability.

---

# 57. Pod Preemption

Pods can sometimes be preempted based on scheduling priorities.

Investigate:

```text
PriorityClass
Scheduling events
Node capacity
Higher-priority workloads
```

A sudden disappearance of Pods should not always be interpreted as an application crash.

---

# 58. Image Pull Monitoring

A Pod may remain unavailable because its image cannot be pulled.

Common states:

```text
ImagePullBackOff
ErrImagePull
```

Check:

```bash
kubectl describe pod <pod-name>
```

Investigate:

```text
Image name
Tag
Registry
Authentication
Network
Registry availability
```

---

# 59. Pod Configuration Errors

Pods can fail before the application starts because of:

```text
Missing ConfigMap
Missing Secret
Invalid environment variable
Invalid volume
Incorrect command
Incorrect arguments
```

Common status:

```text
CreateContainerConfigError
```

Use:

```bash
kubectl describe pod <pod-name>
```

and inspect Events.

---

# 60. Pod Volume Monitoring

Monitor:

```text
Volume mounts
PVC status
Mount errors
Storage capacity
I/O
```

A Pod stuck in:

```text
ContainerCreating
```

may have a volume or image-related issue.

---

# 61. Pod Scheduling Metrics

Monitor:

```text
Pending Pod count
Scheduling latency
Failed scheduling attempts
Unschedulable Pods
```

A sudden increase in Pending Pods can indicate:

```text
Cluster capacity exhaustion
Scheduling constraints
Node failures
Storage limitations
```

---

# 62. Pod Monitoring Alerts

Useful alerts include:

```text
Pod unavailable
Pod not Ready
High restart rate
OOMKilled
CrashLoopBackOff
Long Pending duration
High CPU
High memory
High CPU throttling
Probe failures
Pod eviction
High ephemeral storage usage
```

Alerts should have sensible thresholds and durations.

---

# 63. Avoid Noisy Alerts

Bad alert:

```text
CPU > 70%
for every Pod
```

This can generate excessive alerts.

Better:

```text
Critical service CPU saturation
for sustained period
```

Combine:

```text
Resource signal
+
Duration
+
Workload importance
```

---

# 64. Symptom-Based Pod Alerts

A useful alert:

```text
Ready replicas < desired replicas
```

can be more actionable than:

```text
CPU > 80%
```

because it directly indicates reduced application capacity.

---

# 65. Restart Alert

A useful conceptual alert:

```text
Restart rate is continuously increasing
```

rather than:

```text
Restart count > 0
```

The first identifies ongoing instability.

---

# 66. OOM Alert

Alert when:

```text
OOMKilled events
```

occur for important workloads.

Also correlate with:

```text
Memory usage
Memory limits
Restart rate
Application behavior
```

---

# 67. Pending Pod Alert

Example condition:

```text
Pod remains Pending
for several minutes
```

This can identify:

```text
Scheduling failure
Capacity shortage
Storage issue
Taints
Affinity
```

The exact duration should be based on workload startup expectations.

---

# 68. Readiness Alert

A workload alert can compare:

```text
Ready replicas
vs
Desired replicas
```

Example:

```text
Desired = 10
Ready = 7
```

This directly reflects reduced capacity.

---

# 69. Pod Monitoring With Prometheus

Prometheus can collect:

```text
Container metrics
Kubernetes object metrics
Node metrics
Application metrics
```

A typical architecture:

```text
Pod
│
├── Application Metrics
│       ↓
│   Prometheus
│
└── Kubernetes State
        ↓
  kube-state-metrics
        ↓
    Prometheus
```

---

# 70. kube-state-metrics for Pods

kube-state-metrics provides Kubernetes object state.

For Pods, useful information can include:

```text
Pod phase
Container waiting state
Container termination state
Ready condition
Restart information
```

Prometheus can then query these metrics.

---

# 71. Container Metrics

Container-level metrics provide resource information such as:

```text
CPU usage
Memory usage
Network
Filesystem
```

These are useful for identifying resource-heavy containers.

---

# 72. Application Metrics Inside Pods

A Pod may also expose application metrics:

```text
/metrics
```

Example:

```text
http_requests_total
http_request_duration_seconds
```

This provides application-level visibility.

---

# 73. Three Layers of Pod Monitoring

Think of Pod monitoring as:

```text
Layer 1
Kubernetes State
→ Is the Pod healthy?

Layer 2
Container Resources
→ Is the Pod consuming resources normally?

Layer 3
Application Metrics
→ Is the application actually serving requests correctly?
```

All three should be considered.

---

# 74. Pod Monitoring and Logs

Metrics tell you:

```text
Restart rate increased
```

Logs tell you:

```text
Database connection failed
```

Workflow:

```text
Metric
 ↓
Identify Pod
 ↓
kubectl logs
 ↓
Root cause
```

---

# 75. Pod Monitoring and Tracing

Metrics:

```text
Payment p95 latency ↑
```

Tracing:

```text
Payment
 ↓
Database
 ↓
1.8 seconds
```

Then logs:

```text
Database timeout
```

This creates:

```text
Metrics
 ↓
Trace
 ↓
Logs
```

---

# 76. Pod Monitoring During Deployment

During deployment monitor:

```text
New Pods
Ready Pods
Old Pods
Restart rate
Probe failures
Image pull
Application errors
Latency
```

Typical flow:

```text
Deployment
 ↓
New ReplicaSet
 ↓
New Pods
 ↓
Startup
 ↓
Readiness
 ↓
Traffic
```

A deployment is successful only when the application is healthy, not merely when Pods are created.

---

# 77. Zero-Downtime Deployment Monitoring

For zero-downtime deployments:

```text
Old Pods
   ↓
Serving traffic
   ↓
New Pods become Ready
   ↓
Traffic shifts
   ↓
Old Pods terminate
```

Monitor:

```text
Ready replicas
Available replicas
5xx
Latency
Connection errors
```

---

# 78. Pod Monitoring During Rollback

During rollback:

```text
New version
    ↓
Failure
    ↓
Rollback
    ↓
Previous version
```

Monitor:

```text
Pod readiness
Restart rate
Application errors
Latency
Traffic
```

The rollback should restore application health, not simply restore the old Pod count.

---

# 79. Pod Monitoring in EKS

A practical EKS monitoring stack can be:

```text
EKS
│
├── Metrics Server
│
├── kube-state-metrics
│
├── Prometheus
│
├── Grafana
│
├── ELK
│
├── OpenTelemetry
│
└── Jaeger
```

For Pod monitoring:

```text
Prometheus
   ↓
Pod / container metrics
   ↓
Grafana
```

and:

```text
ELK
 ↓
Pod logs
 ↓
Kibana
```

and:

```text
OpenTelemetry
 ↓
Pod traces
 ↓
Jaeger
```

---

# 80. Pod Monitoring Troubleshooting Workflow

When a Pod is unhealthy:

```text
1. Check Pod status
2. Check Ready condition
3. Check container states
4. Check restart count
5. Check Events
6. Check current logs
7. Check previous logs
8. Check CPU
9. Check memory
10. Check probes
11. Check networking
12. Check storage
13. Check dependencies
14. Check Deployment / ReplicaSet
15. Correlate metrics, logs, and traces
```

---

# 81. Example: Pod Is Running but Users Get Errors

Problem:

```text
Pod = Running
```

Users:

```text
HTTP 503
```

Check:

```text
Ready = ?
```

If:

```text
Ready = False
```

investigate:

```text
Readiness probe
Application health
Dependencies
Service endpoints
```

Running alone does not prove availability.

---

# 82. Example: Pod Keeps Restarting

Check:

```bash
kubectl get pod <pod-name>
```

Then:

```bash
kubectl describe pod <pod-name>
```

Then:

```bash
kubectl logs <pod-name> --previous
```

Look for:

```text
OOMKilled
Exit code
Application exception
Probe failure
Configuration issue
Dependency failure
```

---

# 83. Example: Pod Is Pending

Check:

```bash
kubectl describe pod <pod-name>
```

Look at Events.

Possible output:

```text
0/5 nodes are available:
Insufficient cpu
```

Then investigate:

```text
Node capacity
Pod requests
Cluster Autoscaler
Scheduling constraints
```

---

# 84. Example: Pod Has High CPU

Suppose:

```text
CPU = 95%
```

Check:

```text
Traffic
CPU request
CPU limit
CPU throttling
Application latency
```

If:

```text
Traffic ↑
CPU ↑
Latency normal
Errors normal
```

it may be expected load.

If:

```text
Traffic normal
CPU ↑
Latency ↑
```

investigate application behavior.

---

# 85. Example: Pod Has High Memory

Suppose:

```text
Memory = 95% of limit
```

Check:

```text
Memory trend
Restarts
OOMKilled
Traffic
Application logs
```

If memory continuously increases:

```text
Possible memory leak
```

If memory spikes only during large workloads:

```text
Possible workload-driven growth
```

---

# 86. Example: Pod Has No Logs

Check:

```text
Container actually started?
Container state?
Correct container?
stdout/stderr?
Logging agent?
```

For multi-container Pods:

```bash
kubectl logs <pod-name> -c <container-name>
```

If the container never started, application logs may not exist.

Check Events.

---

# 87. Example: Pod Not Receiving Traffic

Check:

```text
Pod Ready?
Service selector correct?
Endpoint exists?
Ingress configuration?
NetworkPolicy?
```

Flow:

```text
Pod
 ↓
Ready
 ↓
Endpoint
 ↓
Service
 ↓
Ingress
 ↓
Client
```

Find where the traffic path breaks.

---

# 88. Pod Monitoring Best Practices

```text
1. Monitor Ready state, not only Running state.
2. Track restart rate.
3. Monitor CPU and memory.
4. Monitor OOMKilled events.
5. Monitor CPU throttling.
6. Monitor probe failures.
7. Monitor Pending Pods.
8. Monitor Pod evictions.
9. Monitor ephemeral storage.
10. Monitor scheduling failures.
11. Monitor Pod distribution.
12. Monitor application metrics.
13. Centralize logs.
14. Use distributed tracing.
15. Correlate metrics, logs, and traces.
16. Alert on symptoms that require action.
```

---

# 89. Pod Monitoring Dashboard Example

```text
┌──────────────────────────────────────────────┐
│              POD OVERVIEW                    │
├──────────────────────────────────────────────┤
│ Total Pods       120                         │
│ Running          115                         │
│ Ready            110                         │
│ Pending            3                         │
│ Failed             2                         │
│ Restarts/min      12                         │
├──────────────────────────────────────────────┤
│ CPU Usage         62%                        │
│ Memory Usage      68%                        │
│ CPU Throttling     4%                        │
├──────────────────────────────────────────────┤
│ Probe Failures     5                         │
│ OOMKilled           2                         │
│ Evictions           1                         │
└──────────────────────────────────────────────┘
```

Filters:

```text
Cluster
Namespace
Deployment
Service
Node
Pod
Environment
```

---

# 90. Interview Question

### How would you monitor Kubernetes Pods?

**Answer:**

I would monitor Pod phase, Ready condition, container states, restart rate, CPU, memory, CPU throttling, OOMKilled events, probe failures, scheduling status, evictions, network behavior, and application metrics. I would use Prometheus and Grafana for historical metrics, kube-state-metrics for Kubernetes object state, centralized logs for detailed troubleshooting, and OpenTelemetry with Jaeger for request-level tracing.

---

# 91. Interview Question

### A Pod is Running but users receive 503. What do you check?

**Answer:**

I would first check whether the Pod is Ready because Running does not necessarily mean it can receive traffic. Then I would inspect readiness probe failures, Service endpoints, application logs, ingress behavior, and application metrics. I would trace the request path from the ingress through the Service to the Pod to identify where the failure occurs.

---

# 92. Interview Question

### How would you troubleshoot CrashLoopBackOff?

**Answer:**

I would check the Pod description and Events, then inspect current and previous container logs. I would verify the exit reason, resource limits, OOMKilled status, environment variables, Secrets, ConfigMaps, probes, startup commands, and dependencies. I would then correlate the Pod restart behavior with CPU and memory metrics to identify the root cause.

---

# 93. Interview Question

### How would you troubleshoot a Pending Pod?

**Answer:**

I would run `kubectl describe pod` and inspect the Events section for scheduling failures. Then I would check available Node resources, Pod resource requests, taints and tolerations, node selectors, affinity rules, topology constraints, PVC status, and Cluster Autoscaler behavior. The goal is to identify why the scheduler cannot place the Pod.

---

# 94. Interview Question

### How do you identify whether a Pod has a memory problem?

**Answer:**

I would monitor its memory usage and compare it with the configured request and limit. I would check whether the Pod has been OOMKilled or repeatedly restarted and inspect application logs for memory-related errors. I would also examine the memory trend over time to distinguish a steady leak from normal workload-driven usage.

---

# 95. Interview Question

### Why is restart count alone not enough?

**Answer:**

Restart count is cumulative and does not show when the restarts occurred. A Pod with ten restarts over several months may be healthy now, while a Pod with three restarts in five minutes may be unstable. Therefore I prefer monitoring restart rate over a time window and correlating it with container termination reasons.

---

# 96. Interview Question

### What is the difference between readiness and liveness?

**Answer:**

Readiness determines whether a Pod should receive traffic. If readiness fails, the Pod can remain running but should be removed from the ready endpoints. Liveness determines whether the container is still functioning sufficiently to continue running; repeated liveness failures can cause Kubernetes to restart the container.

---

# 97. Interview Question

### How do you monitor Pod resource efficiency?

**Answer:**

I compare actual CPU and memory usage with configured requests and limits. If requests are consistently much higher than actual usage, the workload may be over-provisioned. If usage frequently approaches or exceeds the request, I investigate whether requests are too low. I also check CPU throttling and OOMKilled events.

---

# 98. Interview Question

### How do you monitor Pod availability during deployment?

**Answer:**

I monitor desired, updated, available, and ready replicas along with restart rate, probe failures, application error rate, and latency. New Pods should become Ready before old Pods are removed. I also verify user-facing metrics such as 5xx rate and latency because a deployment can technically complete while the application is still unhealthy.

---

# 99. Production Pod Monitoring Checklist

```text
POD STATE
[ ] Pending
[ ] Running
[ ] Succeeded
[ ] Failed
[ ] Unknown

AVAILABILITY
[ ] Ready
[ ] Not Ready
[ ] Desired replicas
[ ] Available replicas
[ ] Ready replicas

CONTAINERS
[ ] Running
[ ] Waiting
[ ] Terminated
[ ] Exit codes
[ ] Restart rate
[ ] OOMKilled

RESOURCES
[ ] CPU usage
[ ] CPU request
[ ] CPU limit
[ ] CPU throttling
[ ] Memory usage
[ ] Memory request
[ ] Memory limit
[ ] Memory pressure
[ ] Ephemeral storage

PROBES
[ ] Readiness
[ ] Liveness
[ ] Startup
[ ] Probe failures

SCHEDULING
[ ] Pending duration
[ ] Scheduling failures
[ ] Node placement
[ ] Taints
[ ] Affinity
[ ] Topology spread

NETWORK
[ ] Traffic
[ ] Connections
[ ] Errors
[ ] DNS
[ ] Service endpoints
[ ] NetworkPolicy

STORAGE
[ ] PVC
[ ] Mounts
[ ] Capacity
[ ] I/O
[ ] Ephemeral storage

OPERATIONS
[ ] Events
[ ] Logs
[ ] Previous logs
[ ] Deployments
[ ] ReplicaSets
[ ] HPA
[ ] Evictions
[ ] PDB

OBSERVABILITY
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] OpenTelemetry
[ ] Jaeger
```

---

# 100. Final Mental Model

Remember Pod monitoring as:

```text
                         POD
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
      STATE            RESOURCES         HEALTH
        │                 │                 │
   Pending/Running    CPU/Memory        Probes
   Ready/Failed       Requests/Limits    Restarts
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                     PROMETHEUS
                          ↓
                       GRAFANA
                          ↓
                        ALERTS
```

Then combine it with troubleshooting signals:

```text
Pod
 │
 ├── Metrics
 │      ↓
 │   Prometheus
 │
 ├── Logs
 │      ↓
 │     ELK
 │
 └── Traces
        ↓
   OpenTelemetry
        ↓
      Jaeger
```

The key principle is:

**A healthy Kubernetes Pod is more than a Pod in the Running state. Production Pod monitoring must evaluate readiness, container health, restart behavior, resource consumption, probes, scheduling, networking, storage, and application behavior. Prometheus and Grafana provide historical metrics and dashboards, kube-state-metrics provides Kubernetes object state, centralized logging provides detailed failure information, and distributed tracing shows request-level behavior. The strongest troubleshooting approach correlates all of these signals rather than relying on a single Pod status.**
