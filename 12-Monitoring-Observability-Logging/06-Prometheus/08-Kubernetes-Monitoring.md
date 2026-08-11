# Prometheus Kubernetes Monitoring

Kubernetes monitoring is one of the most important areas for a DevOps engineer because a production Kubernetes cluster contains multiple layers that must be monitored.

A production EKS cluster can have:

```text
Cluster
 ├── Control Plane
 ├── Worker Nodes
 ├── Pods
 ├── Deployments
 ├── StatefulSets
 ├── DaemonSets
 ├── Services
 ├── Ingress
 ├── Persistent Volumes
 ├── Applications
 └── Databases
```

Prometheus can collect metrics from these different layers and provide the foundation for monitoring and alerting.

The overall architecture is:

```text
                    Kubernetes / EKS
                           │
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
        Nodes             Pods           Objects
          │                │                │
          ↓                ↓                ↓
   Node Exporter      App Metrics    Kube-State-Metrics
          │                │                │
          └────────────────┼────────────────┘
                           ↓
                 Prometheus Operator
                           ↓
                      Prometheus
                           ↓
                         TSDB
                           ↓
                        Grafana
```

---

# 1. What Should We Monitor in Kubernetes?

A production Kubernetes monitoring strategy should cover multiple layers.

```text
1. Cluster
2. Nodes
3. Pods
4. Containers
5. Deployments
6. Services
7. Applications
8. Ingress / Load Balancer
9. Persistent Volumes
10. Kubernetes object state
11. Resource utilization
12. Availability
13. Errors
14. Saturation
```

Monitoring only CPU and memory is not enough.

---

# 2. Kubernetes Monitoring Layers

A useful mental model is:

```text
                    Kubernetes Monitoring
                           │
       ┌───────────────────┼───────────────────┐
       ↓                   ↓                   ↓
   Infrastructure      Kubernetes State     Application
       │                   │                   │
       ↓                   ↓                   ↓
 Node Exporter       Kube-State-Metrics   App Metrics
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ↓
                       Prometheus
```

---

# 3. Node-Level Monitoring

Node-level monitoring tells us how the worker nodes are performing.

Typical metrics:

```text
CPU
Memory
Disk
Filesystem
Network
Load
Node availability
```

Node Exporter is commonly used for Linux nodes.

```text
Node
 ↓
Node Exporter
 ↓
Prometheus
```

---

# 4. Pod-Level Monitoring

Pod monitoring answers questions such as:

```text
How many pods are running?
How many are pending?
How many are restarting?
Which pods are failing?
Which pods are consuming resources?
```

For example:

```text
order-service
 ├── Pod 1
 ├── Pod 2
 └── Pod 3
```

Prometheus should monitor the health and resource behavior of these pods.

---

# 5. Container-Level Monitoring

A pod can contain one or more containers.

Example:

```text
Pod
 ├── application
 └── sidecar
```

Container-level metrics help identify:

```text
CPU usage
Memory usage
Container restarts
Memory limits
CPU limits
Container state
```

This is especially useful when one container inside a pod is causing resource pressure.

---

# 6. Kubernetes Object Monitoring

Kubernetes object state can be monitored through kube-state-metrics.

Examples:

```text
Deployment
StatefulSet
DaemonSet
ReplicaSet
Pod
Node
Job
CronJob
PersistentVolume
PersistentVolumeClaim
Namespace
```

Architecture:

```text
Kubernetes API
      ↓
Kube-State-Metrics
      ↓
/metrics
      ↓
Prometheus
```

---

# 7. Kube-State-Metrics

Kube-state-metrics exposes metrics about the state of Kubernetes objects.

It does not measure actual CPU usage directly.

Instead, it exposes information such as:

```text
Desired replicas
Available replicas
Ready replicas
Pod phase
Deployment status
Node conditions
PVC status
Job status
```

This distinction is important.

---

# 8. Node Exporter vs Kube-State-Metrics

## Node Exporter

Answers:

> What is happening on the operating system?

Examples:

```text
CPU
Memory
Disk
Network
```

## Kube-State-Metrics

Answers:

> What does Kubernetes know about the state of its objects?

Examples:

```text
Desired replicas
Available replicas
Pod phase
Deployment status
```

Both are required for comprehensive monitoring.

---

# 9. Application Monitoring

Infrastructure monitoring alone is not enough.

Applications should expose metrics such as:

```text
Request rate
Request latency
HTTP errors
Application errors
Database calls
Queue processing
Business metrics
```

Example:

```text
Java Application
       ↓
Prometheus Client / Micrometer
       ↓
/metrics
       ↓
Prometheus
```

---

# 10. Complete Kubernetes Monitoring Stack

A production monitoring stack can look like:

```text
                     EKS
                      │
       ┌──────────────┼───────────────┐
       ↓              ↓               ↓
     Nodes           Pods         Kubernetes API
       │              │               │
       ↓              ↓               ↓
Node Exporter    App Metrics    Kube-State-Metrics
       │              │               │
       └──────────────┼───────────────┘
                      ↓
               Prometheus Operator
                      ↓
                  Prometheus
                      ↓
                    Grafana
```

---

# 11. Prometheus Operator

Prometheus Operator makes running Prometheus on Kubernetes easier.

It provides Kubernetes custom resources such as:

```text
Prometheus
ServiceMonitor
PodMonitor
PrometheusRule
Alertmanager
```

Instead of manually managing a large Prometheus configuration file, monitoring configuration can be represented as Kubernetes resources.

---

# 12. Why Use Prometheus Operator?

Without Operator:

```text
Prometheus
   ↓
Large prometheus.yml
   ↓
Manual configuration
```

With Operator:

```text
Kubernetes Resources
   ↓
Prometheus Operator
   ↓
Prometheus Configuration
   ↓
Prometheus
```

This is especially useful in GitOps environments.

---

# 13. Prometheus Operator Architecture

```text
                       Kubernetes API
                              │
                              ↓
                    Prometheus Operator
                              │
       ┌──────────────────────┼──────────────────────┐
       ↓                      ↓                      ↓
ServiceMonitor           PodMonitor          PrometheusRule
       │                      │                      │
       └──────────────────────┼──────────────────────┘
                              ↓
                         Prometheus
                              ↓
                            TSDB
                              ↓
                           Grafana
```

---

# 14. Installing Kubernetes Monitoring

A common production approach is to install a maintained Prometheus Operator-based monitoring stack through Helm.

A commonly used chart is the Prometheus community monitoring stack.

The exact chart version should be pinned in production rather than using an unbounded latest version.

Example:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Update:

```bash
helm repo update
```

Create namespace:

```bash
kubectl create namespace monitoring
```

Install:

```bash
helm install monitoring \
  prometheus-community/kube-prometheus-stack \
  -n monitoring
```

For production, use a versioned chart and a values file stored in Git.

---

# 15. Verify Installation

Check pods:

```bash
kubectl get pods -n monitoring
```

You may see components such as:

```text
prometheus
grafana
alertmanager
kube-state-metrics
node-exporter
operator
```

The exact names depend on the chart release and configuration.

---

# 16. Check Services

```bash
kubectl get svc -n monitoring
```

Typical services include:

```text
Prometheus
Grafana
Alertmanager
Kube-State-Metrics
Node Exporter
```

---

# 17. Check Custom Resources

```bash
kubectl get prometheus -n monitoring
```

Check ServiceMonitors:

```bash
kubectl get servicemonitor -A
```

Check PodMonitors:

```bash
kubectl get podmonitor -A
```

Check PrometheusRules:

```bash
kubectl get prometheusrule -A
```

---

# 18. Prometheus Access

For temporary local access:

```bash
kubectl port-forward \
  svc/monitoring-kube-prometheus-prometheus \
  9090:9090 \
  -n monitoring
```

Then access Prometheus locally.

For production, expose Prometheus through an appropriate private ingress or internal load-balancing architecture rather than using port-forwarding.

---

# 19. Grafana Access

For temporary access:

```bash
kubectl port-forward \
  svc/monitoring-grafana \
  3000:80 \
  -n monitoring
```

Production environments should use an appropriate authenticated access path, usually through an internal or controlled ingress/load balancer.

---

# 20. Node Exporter in Kubernetes

Node Exporter is commonly deployed as a DaemonSet.

Architecture:

```text
EKS
 │
 ├── Node 1 → Node Exporter
 ├── Node 2 → Node Exporter
 └── Node 3 → Node Exporter
```

When a new node joins:

```text
New Node
   ↓
DaemonSet
   ↓
Node Exporter
```

This automatically provides node-level metrics.

---

# 21. Why DaemonSet?

A DaemonSet ensures that a pod runs on every eligible node.

This is ideal for infrastructure agents.

Examples:

```text
Node Exporter
Log agents
Security agents
Monitoring agents
```

---

# 22. Kube-State-Metrics Deployment

Kube-state-metrics normally runs as a Kubernetes workload and watches the Kubernetes API.

Architecture:

```text
Kubernetes API
       ↓
Kube-State-Metrics
       ↓
/metrics
       ↓
Prometheus
```

It provides Kubernetes object-state metrics.

---

# 23. ServiceMonitor

ServiceMonitor defines how Prometheus should scrape a Kubernetes Service.

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor

metadata:
  name: order-service
  namespace: monitoring

spec:
  selector:
    matchLabels:
      app: order

  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
```

---

# 24. Application Service

Suppose the application Service is:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: order-service
  labels:
    app: order

spec:
  selector:
    app: order

  ports:
    - name: metrics
      port: 9090
      targetPort: 9090
```

The ServiceMonitor selects this Service:

```text
Service
  app=order
     ↓
ServiceMonitor
  app=order
```

---

# 25. Prometheus Selection

One of the most common Kubernetes monitoring mistakes is forgetting that the Prometheus resource must also select the ServiceMonitor.

Conceptually:

```text
Prometheus
    ↓
Select ServiceMonitor
    ↓
ServiceMonitor
    ↓
Select Service
    ↓
Service
    ↓
Pod
```

There are therefore multiple selector relationships to verify.

---

# 26. Complete ServiceMonitor Flow

```text
Prometheus
    │
    │ selects
    ↓
ServiceMonitor
    │
    │ selects
    ↓
Service
    │
    │ selects
    ↓
Pod
    │
    ↓
/metrics
```

If any selector is wrong, the target may not be scraped.

---

# 27. PodMonitor

PodMonitor can directly select pods.

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor

metadata:
  name: order-pods

spec:
  selector:
    matchLabels:
      app: order

  podMetricsEndpoints:
    - port: metrics
      path: /metrics
```

Architecture:

```text
Pod
 ↓
PodMonitor
 ↓
Prometheus
```

---

# 28. ServiceMonitor vs PodMonitor

| ServiceMonitor             | PodMonitor                       |
| -------------------------- | -------------------------------- |
| Selects Services           | Selects Pods                     |
| Service-oriented           | Pod-oriented                     |
| Stable service abstraction | Direct pod scraping              |
| Common application pattern | Useful for direct pod monitoring |

Choose according to the application architecture.

---

# 29. Application Metrics Endpoint

Suppose a Java application exposes:

```text
/actuator/prometheus
```

instead of:

```text
/metrics
```

Then configure:

```yaml
endpoints:
  - port: metrics
    path: /actuator/prometheus
```

The scrape path must match the actual application endpoint.

---

# 30. Application Metrics Example

A microservice may expose:

```text
http_server_requests_seconds_count
http_server_requests_seconds_sum
jvm_memory_used_bytes
process_cpu_usage
```

The exact names depend on the instrumentation library.

These metrics allow Prometheus to monitor application behavior.

---

# 31. Kubernetes Resource Monitoring

Important resource concepts:

```text
CPU Requests
CPU Limits
Memory Requests
Memory Limits
Actual Usage
```

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi

  limits:
    cpu: 500m
    memory: 512Mi
```

Monitoring should compare requested, limited, and actual resource usage.

---

# 32. CPU Requests vs Actual Usage

Suppose:

```text
CPU Request = 250m
Actual CPU = 220m
```

The application is using most of its requested CPU.

But:

```text
CPU Limit = 1000m
```

means the container can potentially use more CPU before reaching its configured limit, subject to cluster scheduling and runtime behavior.

---

# 33. Memory Monitoring

Memory is especially important because exceeding a container memory limit can lead to:

```text
OOMKilled
```

Monitoring should identify:

```text
Memory usage
Memory requests
Memory limits
Memory working set
OOM events
```

---

# 34. CPU Throttling

A container can experience CPU throttling when it reaches its CPU limit.

Monitoring CPU usage alone may not reveal the complete picture.

For performance troubleshooting, also investigate:

```text
CPU throttling
CPU limits
CPU requests
Node CPU pressure
Application latency
```

---

# 35. Container Restart Monitoring

A rising restart count is an important signal.

Example:

```text
order-service
restarts = 0

After deployment:
restarts = 25
```

Investigate:

```text
CrashLoopBackOff
OOMKilled
Application crash
Probe failure
Configuration error
Dependency failure
```

---

# 36. CrashLoopBackOff Monitoring

A pod may enter:

```text
CrashLoopBackOff
```

Prometheus can expose Kubernetes state information that helps identify the affected pod.

The troubleshooting flow is:

```text
Prometheus
 ↓
Identify problematic pod
 ↓
kubectl describe pod
 ↓
kubectl logs
 ↓
Investigate application/container
```

Prometheus is the monitoring system; Kubernetes commands remain useful for detailed diagnosis.

---

# 37. Pod Readiness

A pod can be:

```text
Running
```

but not:

```text
Ready
```

This is an important distinction.

Example:

```text
Pod:
Running = true
Ready = false
```

The application process may be alive but not ready to receive traffic.

---

# 38. Readiness Monitoring

Monitor:

```text
Ready replicas
Desired replicas
Available replicas
Unavailable replicas
```

For a Deployment:

```text
Desired = 5
Available = 5
```

is healthy.

But:

```text
Desired = 5
Available = 3
```

requires investigation.

---

# 39. Deployment Monitoring

A production Deployment should be monitored for:

```text
Desired replicas
Updated replicas
Available replicas
Unavailable replicas
Ready replicas
Rollout progress
```

---

# 40. Deployment Availability

Suppose:

```text
Deployment:
desired = 10
available = 10
```

Healthy.

After a deployment:

```text
desired = 10
available = 7
```

Potential causes:

```text
Image issue
Probe failure
Resource shortage
Scheduling problem
Application startup failure
Configuration issue
```

---

# 41. StatefulSet Monitoring

StatefulSets are important for workloads such as:

```text
Databases
Kafka
Stateful applications
```

Monitor:

```text
Desired replicas
Ready replicas
Current replicas
Pod availability
Persistent volumes
```

---

# 42. DaemonSet Monitoring

For a DaemonSet:

```text
Desired nodes = 10
Ready pods = 10
```

Healthy.

If:

```text
Desired = 10
Ready = 7
```

investigate:

```text
Scheduling
Taints
Tolerations
Resource availability
Node conditions
Pod failures
```

---

# 43. Kubernetes Scheduling Monitoring

A pod may remain Pending because:

```text
CPU insufficient
Memory insufficient
Node selector mismatch
Affinity rules
Taints
Tolerations
PVC unavailable
```

Prometheus can help identify symptoms, while:

```bash
kubectl describe pod
```

usually provides the detailed scheduling event.

---

# 44. Node Health Monitoring

Important node signals include:

```text
CPU utilization
Memory utilization
Filesystem utilization
Network
Node readiness
Disk pressure
Memory pressure
PID pressure
```

---

# 45. Node Conditions

Kubernetes nodes can report conditions such as:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

These are useful for identifying node-level problems.

---

# 46. Node NotReady

If a node becomes:

```text
NotReady
```

the impact can include:

```text
Pods unavailable
Rescheduling
Traffic disruption
Capacity reduction
```

Alerting on important node conditions is essential.

---

# 47. Node Capacity Monitoring

Monitor:

```text
CPU capacity
Memory capacity
Pod capacity
Allocatable resources
Actual utilization
```

A cluster can have low CPU utilization but still encounter pod scheduling problems if allocatable resources are constrained by requests.

---

# 48. Requests and Scheduling

Kubernetes scheduling uses resource requests when determining whether a pod can fit on a node.

Example:

```text
Node allocatable CPU = 4 CPU

Existing requests = 3.5 CPU

New pod request = 1 CPU
```

The scheduler may not place the pod because:

```text
3.5 + 1 > 4
```

even if actual CPU utilization is currently much lower.

This is why monitoring actual utilization alone is insufficient.

---

# 49. Cluster Capacity Monitoring

A production monitoring platform should monitor:

```text
Total nodes
Ready nodes
CPU capacity
CPU allocatable
Memory capacity
Memory allocatable
Pod capacity
Pod usage
```

This helps with capacity planning.

---

# 50. Namespace Monitoring

Namespaces are useful for organizational and operational separation.

Monitor:

```text
Pods
CPU
Memory
Deployments
Services
Errors
Restarts
```

per namespace.

Example:

```promql
sum by (namespace) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

The exact query depends on which container metrics are available in your deployment.

---

# 51. Application Monitoring by Namespace

You can filter metrics:

```promql
up{namespace="production"}
```

This allows production-only monitoring.

For staging:

```promql
up{namespace="staging"}
```

---

# 52. Kubernetes Monitoring by Environment

Use labels such as:

```text
environment=production
environment=staging
environment=development
```

Then dashboards can be separated.

Example:

```promql
up{environment="production"}
```

---

# 53. Kubernetes Service Monitoring

A Kubernetes Service provides a stable networking abstraction.

But monitoring a Service does not automatically guarantee that every backend pod is individually visible.

Understand the difference:

```text
Service
 ↓
Load balances
 ↓
Pods
```

while direct pod discovery can expose each pod as a target.

---

# 54. Ingress Monitoring

For an EKS environment using an ALB Ingress Controller / AWS Load Balancer Controller, monitoring can exist at multiple layers:

```text
Internet
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

Monitor:

```text
ALB metrics
Ingress/controller metrics where available
Service health
Pod health
Application metrics
```

---

# 55. ALB Monitoring

AWS ALB metrics are commonly available through AWS monitoring integrations.

Prometheus should not be forced to scrape every AWS-managed service through an exporter when AWS already provides an appropriate metric source.

The architecture can be:

```text
AWS ALB
   ↓
AWS metrics
   ↓
Prometheus integration / exporter
   ↓
Prometheus
```

The exact integration depends on the AWS monitoring architecture.

---

# 56. Kubernetes and AWS Monitoring

A production EKS architecture may combine:

```text
Kubernetes Metrics
+
Application Metrics
+
Node Metrics
+
AWS Metrics
```

Architecture:

```text
                       EKS
                        │
        ┌───────────────┼────────────────┐
        ↓               ↓                ↓
   Kubernetes        Application       AWS
      State            Metrics        Services
        │               │                │
        ↓               ↓                ↓
Kube-State-Metrics   /metrics      AWS Metrics
        │               │                │
        └───────────────┼────────────────┘
                        ↓
                    Prometheus
```

---

# 57. Persistent Volume Monitoring

Stateful workloads depend on storage.

Monitor:

```text
PVC status
PV status
Storage capacity
Available space
I/O
Volume errors
```

---

# 58. PVC Problems

A PVC can remain:

```text
Pending
```

because:

```text
StorageClass issue
Provisioner issue
Capacity issue
Access mode issue
Cloud volume problem
```

Kubernetes state metrics can help identify the problem.

Detailed investigation still requires:

```bash
kubectl describe pvc <name>
```

---

# 59. Storage Usage

For database workloads, monitor:

```text
Filesystem usage
Volume capacity
Disk I/O
IOPS
Latency
```

A database may remain healthy while its disk is approaching capacity, making storage monitoring essential.

---

# 60. Kubernetes Network Monitoring

Monitor:

```text
Network receive
Network transmit
Packet errors
Dropped packets
Application latency
Service connectivity
```

For deeper network troubleshooting, additional network observability tools may be required.

Prometheus provides metrics, but it is not a complete packet-level network debugging tool.

---

# 61. Kubernetes DNS Monitoring

DNS is critical for Kubernetes service communication.

Applications may depend on:

```text
service.namespace.svc.cluster.local
```

If cluster DNS fails:

```text
Application
   ↓
DNS lookup
   X
Service
```

Applications can appear unhealthy even when their pods are running.

Monitor DNS components and relevant application errors.

---

# 62. Application Dependency Monitoring

For a microservices platform:

```text
Order
 ↓
Payment
 ↓
Database
```

monitor each dependency.

For example:

```text
Order latency ↑
Payment latency ↑
Database healthy
```

This may indicate a problem between Order and Payment rather than the database.

---

# 63. Microservices Monitoring Architecture

For a platform containing:

```text
User
Product
Cart
Order
Payment
Inventory
Notification
```

the monitoring architecture can be:

```text
                  Prometheus
                       │
     ┌─────────────────┼─────────────────┐
     ↓                 ↓                 ↓
Node Metrics      App Metrics      Dependency Metrics
     │                 │                 │
Node Exporter       /metrics        DB/Redis Exporters
                       │
                       ↓
                    Grafana
```

---

# 64. Request Rate

Application traffic can be measured using request counters.

Conceptually:

```promql
rate(http_requests_total[5m])
```

This gives request rate over the selected interval.

The exact metric name depends on application instrumentation.

---

# 65. Error Rate

Suppose the application exposes:

```text
http_requests_total
```

with:

```text
status_code
```

You can calculate errors using labels.

Conceptually:

```promql
sum(rate(http_requests_total{status_code=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

This calculates an approximate 5xx error ratio when the metric structure supports it.

---

# 66. Latency

Histogram metrics can be used to calculate request latency.

For example:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

This estimates the 95th percentile latency.

The exact metric name depends on your instrumentation.

---

# 67. Saturation

Saturation tells us whether a resource is approaching its capacity.

Examples:

```text
CPU saturation
Memory pressure
Disk capacity
Connection pool saturation
Thread pool saturation
Queue depth
```

This is an important part of Kubernetes monitoring.

---

# 68. Golden Signals in Kubernetes

The four Golden Signals are:

```text
Latency
Traffic
Errors
Saturation
```

For a Kubernetes application:

```text
Latency → application metrics
Traffic → request metrics
Errors → HTTP/application errors
Saturation → CPU/memory/queue/resource metrics
```

---

# 69. Kubernetes Monitoring Dashboard

A useful cluster dashboard can contain:

```text
Cluster Health
Node Health
CPU Usage
Memory Usage
Pod Count
Pending Pods
Restart Count
Deployment Availability
Container Restarts
Filesystem Usage
Network Traffic
```

---

# 70. Application Dashboard

For each microservice:

```text
Request Rate
Error Rate
P95 Latency
P99 Latency
Pod Count
CPU
Memory
Restarts
Dependency Latency
```

---

# 71. Node Dashboard

A node dashboard can contain:

```text
CPU
Memory
Load
Disk
Filesystem
Network
Node Ready
Pod Count
Container Count
```

---

# 72. Namespace Dashboard

A namespace dashboard can show:

```text
CPU Usage
Memory Usage
Pod Count
Restarts
Deployment Availability
Network Traffic
Errors
```

---

# 73. Production Alert Categories

A mature Kubernetes monitoring setup should alert on:

```text
Node failures
Pod failures
Deployment availability
High CPU
High memory
OOMKilled
Disk pressure
Filesystem capacity
Pending pods
Restart spikes
High application error rate
High application latency
Service availability
PVC problems
```

---

# 74. Alert: Node Not Ready

Conceptually:

```promql
kube_node_status_condition{
  condition="Ready",
  status="true"
} == 0
```

The exact expression should be adapted to the metric labels available in your environment.

---

# 75. Alert: Deployment Replicas Missing

Conceptually:

```promql
kube_deployment_status_replicas_available
<
kube_deployment_spec_replicas
```

This identifies deployments where available replicas are below desired replicas.

---

# 76. Alert: Pod Restart Spike

A restart counter can be evaluated over time.

Conceptually:

```promql
increase(kube_pod_container_status_restarts_total[15m]) > 3
```

Use sensible thresholds based on application behavior to avoid noisy alerts.

---

# 77. Alert: High CPU

A common pattern is:

```text
Actual CPU usage
/
CPU capacity or relevant request
```

The correct denominator depends on what you are trying to measure.

Do not blindly use one CPU formula for every environment.

---

# 78. Alert: High Memory

Monitor:

```text
Actual memory
vs
Memory limit
```

or:

```text
Actual memory
vs
Node capacity
```

depending on whether you are monitoring the container or node.

---

# 79. Alert: OOMKilled

A pod/container that repeatedly reaches its memory limit can be affected by OOM termination.

Monitor OOM-related container state and restart behavior.

Then investigate:

```text
Memory limit
Application memory leak
Traffic increase
Heap configuration
Node pressure
```

---

# 80. Alert: Disk Pressure

Node disk pressure can affect scheduling and pod stability.

Monitor:

```text
Filesystem usage
Node DiskPressure
Container image filesystem
Ephemeral storage
```

---

# 81. Alert: Pending Pods

A large increase in Pending pods may indicate:

```text
Insufficient resources
Taints
Affinity rules
PVC issues
Scheduling constraints
Node group capacity
```

This is an important capacity signal.

---

# 82. Alert Noise

Do not alert on every metric.

Bad monitoring:

```text
CPU > 70%
→ Alert

CPU > 71%
→ Alert

CPU > 72%
→ Alert
```

Better:

```text
CPU consistently high
+
Service impact
+
Sustained duration
```

Alert design should focus on actionable conditions.

---

# 83. Kubernetes Monitoring and HPA

Horizontal Pod Autoscaler uses metrics to scale workloads.

Architecture:

```text
Application
    ↓
Metrics
    ↓
Metrics API
    ↓
HPA
    ↓
More / fewer pods
```

Prometheus metrics may be integrated into autoscaling architectures through adapters or other supported mechanisms.

Monitoring and autoscaling are related but should not be treated as the same system.

---

# 84. HPA Monitoring

Monitor:

```text
Current replicas
Desired replicas
CPU utilization
Memory utilization
Scaling events
Maximum replicas
Minimum replicas
```

If HPA is constantly at maximum replicas:

```text
Investigate capacity
```

Do not simply increase `maxReplicas` without understanding the workload.

---

# 85. HPA Troubleshooting

If HPA does not scale:

Check:

```text
Metrics available?
Metrics API healthy?
Target configured?
Current utilization?
Min/max replicas?
Resource requests defined?
HPA events?
```

Useful command:

```bash
kubectl describe hpa <name> -n <namespace>
```

---

# 86. Kubernetes Monitoring and Cluster Autoscaler

HPA scales pods.

Cluster Autoscaler or an equivalent node autoscaling mechanism can scale nodes.

Architecture:

```text
Traffic
  ↓
HPA
  ↓
More Pods
  ↓
Insufficient Node Capacity
  ↓
Cluster Autoscaler
  ↓
More Nodes
```

Prometheus can monitor both sides.

---

# 87. Autoscaling Monitoring

Monitor:

```text
Pod desired replicas
Pod actual replicas
Node count
Node capacity
Pending pods
CPU
Memory
Scaling events
```

This helps determine whether scaling is functioning correctly.

---

# 88. Production EKS Monitoring Architecture

A realistic architecture for an AWS EKS microservices platform:

```text
                              AWS
                               │
                    ┌──────────┴──────────┐
                    │         EKS         │
                    └──────────┬──────────┘
                               │
       ┌───────────────────────┼────────────────────────┐
       ↓                       ↓                        ↓
    Worker Nodes          Applications            Kubernetes API
       │                       │                        │
       ↓                       ↓                        ↓
 Node Exporter             /metrics             Kube-State-Metrics
       │                       │                        │
       └───────────────────────┼────────────────────────┘
                               ↓
                      Prometheus Operator
                               │
             ┌─────────────────┼─────────────────┐
             ↓                 ↓                 ↓
       ServiceMonitor      PodMonitor      PrometheusRule
             │                 │                 │
             └─────────────────┼─────────────────┘
                               ↓
                           Prometheus
                               │
                      ┌────────┴────────┐
                      ↓                 ↓
                   Grafana           Alerting
```

---

# 89. GitOps-Based Monitoring

In a production environment, monitoring configuration can be managed through Git.

Example repository:

```text
monitoring/
├── prometheus/
├── grafana/
├── alertmanager/
├── servicemonitors/
├── podmonitors/
├── alerts/
└── dashboards/
```

ArgoCD can deploy these resources to EKS.

Flow:

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
EKS
 ↓
Monitoring Stack
```

---

# 90. Why GitOps for Monitoring?

Advantages:

```text
Version control
Audit trail
Peer review
Rollback
Consistency
Environment promotion
```

For example:

```text
dev
 ↓
staging
 ↓
production
```

monitoring changes can follow the same controlled deployment process.

---

# 91. Multi-Environment Monitoring

A typical architecture:

```text
             Monitoring Configuration
                       │
             ┌─────────┼─────────┐
             ↓         ↓         ↓
           Dev       Stage      Prod
             │         │         │
         Prometheus Prometheus Prometheus
             │         │         │
             └─────────┼─────────┘
                       ↓
                 Central Grafana
```

Depending on scale and security requirements, environments may use separate Prometheus instances and dashboards.

---

# 92. Production Monitoring Isolation

Production monitoring should be isolated appropriately.

For example:

```text
Production EKS
   ↓
Production Prometheus
```

Staging:

```text
Staging EKS
   ↓
Staging Prometheus
```

This prevents a staging issue from directly affecting production monitoring.

---

# 93. High Availability Prometheus

A single Prometheus instance can become a monitoring single point of failure.

For important production environments, consider:

```text
Prometheus A
Prometheus B
```

scraping the same targets.

Then use an appropriate highly available metrics architecture or long-term backend.

---

# 94. HA Kubernetes Monitoring

Conceptually:

```text
                 Kubernetes
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
    Prometheus A            Prometheus B
          │                       │
          └───────────┬───────────┘
                      ↓
               Long-Term Backend
                      ↓
                   Grafana
```

The exact HA implementation depends on the chosen architecture.

---

# 95. Prometheus Storage

Prometheus stores time-series data locally by default.

For production Kubernetes environments:

```text
Prometheus
   ↓
Persistent Volume
   ↓
Prometheus TSDB
```

Do not assume pod-local ephemeral storage is appropriate for production Prometheus.

---

# 96. Prometheus Persistent Storage

A Helm-based deployment can configure persistent storage.

Conceptually:

```yaml
prometheus:
  prometheusSpec:
    storageSpec:
      volumeClaimTemplate:
        spec:
          storageClassName: gp3
          accessModes:
            - ReadWriteOnce
          resources:
            requests:
              storage: 100Gi
```

The exact storage class and capacity should be designed according to workload requirements.

---

# 97. Prometheus Retention

Prometheus retention determines how long local metrics remain available.

Example concept:

```text
Retention:
15 days
```

or based on storage size:

```text
Retention size:
100GB
```

Production retention should be selected based on:

```text
Metric volume
Storage capacity
Query requirements
Compliance
Long-term storage architecture
```

---

# 98. Long-Term Storage

For long-term metrics, architectures may use:

```text
Thanos
Mimir
Cortex
VictoriaMetrics
```

depending on organizational requirements.

The local Prometheus instance can continue handling scraping while long-term storage handles extended retention and global querying.

---

# 99. Monitoring Prometheus Itself

Prometheus is also an application that needs monitoring.

Monitor:

```text
CPU
Memory
Disk
Active series
Scrape failures
Query latency
Rule evaluation failures
Target count
WAL behavior
Storage usage
```

---

# 100. Monitoring Grafana

Grafana should also be monitored.

Important areas include:

```text
Availability
CPU
Memory
Datasource health
Dashboard query latency
Authentication
```

---

# 101. Monitoring the Monitoring Stack

A production principle:

```text
You must monitor the monitoring system itself.
```

Architecture:

```text
Applications
     ↓
Prometheus
     ↓
Grafana

Prometheus
     ↓
Prometheus self-metrics
     ↓
Grafana
```

If Prometheus stops working, you need another mechanism or HA design to detect the failure.

---

# 102. Kubernetes Monitoring Troubleshooting Framework

When someone reports:

> "Prometheus is not showing my Kubernetes application."

Follow this sequence:

```text
1. Is the application running?
2. Does the pod expose metrics?
3. Does /metrics work?
4. Is the Service correct?
5. Is the ServiceMonitor/PodMonitor correct?
6. Is Prometheus selecting it?
7. Is the target discovered?
8. Is the target UP?
9. Are metrics stored?
10. Is Grafana querying the correct metric?
```

---

# 103. Check Pod

```bash
kubectl get pods -n production
```

Then:

```bash
kubectl describe pod <pod-name> -n production
```

Check:

```text
Status
Events
Container ports
Readiness
Restarts
```

---

# 104. Check Service

```bash
kubectl get svc -n production
```

Then:

```bash
kubectl describe svc order-service -n production
```

Check:

```text
Selector
Port
TargetPort
Endpoints
```

---

# 105. Check ServiceMonitor

```bash
kubectl get servicemonitor -A
```

Then:

```bash
kubectl describe servicemonitor order-service -n monitoring
```

Check:

```text
Selector
Endpoint
Path
Port
Namespace
Labels
```

---

# 106. Check PodMonitor

```bash
kubectl get podmonitor -A
```

Then:

```bash
kubectl describe podmonitor order-service -n monitoring
```

Verify:

```text
Pod selector
Metrics port
Metrics path
Namespace behavior
```

---

# 107. Check Metrics Endpoint

From inside the cluster:

```bash
curl http://<service>:<port>/metrics
```

For example:

```bash
curl http://order-service.production.svc.cluster.local:9090/metrics
```

If this fails, solve the application/service/network problem before debugging Prometheus.

---

# 108. Check Prometheus Targets

Open the Prometheus Targets page.

Find:

```text
order-service
```

Check:

```text
State
Endpoint
Labels
Last Scrape
Error
```

---

# 109. Target Not Found

If the application does not appear:

```text
Service Discovery Problem
```

Investigate:

```text
ServiceMonitor
PodMonitor
Prometheus selector
Namespace selector
Labels
RBAC
Relabeling
```

---

# 110. Target DOWN

If the application appears but is DOWN:

```text
Scrape Problem
```

Investigate:

```text
Port
Path
NetworkPolicy
DNS
TLS
Application
Timeout
```

---

# 111. Target UP but Metric Missing

If the target is UP but the expected metric is missing:

```text
Metric Problem
```

Investigate:

```text
Application instrumentation
Metric name
Metric relabeling
Collector configuration
PromQL
```

---

# 112. Grafana Dashboard Empty

If Prometheus contains the metric but Grafana is empty:

```text
Grafana Problem
```

Check:

```text
Datasource
Prometheus URL
Dashboard query
Variables
Time range
Labels
```

---

# 113. Real-World Incident: Deployment Succeeded but Pods Are Unhealthy

Suppose CI/CD reports:

```text
Deployment successful
```

but monitoring shows:

```text
Available replicas = 2
Desired replicas = 5
```

Investigate:

```text
Pod events
Readiness probes
Liveness probes
Application logs
Resource limits
Image
Configuration
Dependencies
```

Monitoring detects the symptom; Kubernetes and application tooling provide the detailed root cause.

---

# 114. Real-World Incident: Pods Are Running but Users Get 503

Architecture:

```text
User
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pods
```

Prometheus can show:

```text
Pod Ready status
Service availability
Application error rate
Application latency
```

Possible root causes:

```text
Readiness failure
No ready endpoints
Service selector mismatch
Ingress issue
Application failure
ALB target health issue
```

---

# 115. Real-World Incident: Node CPU High

Prometheus shows:

```text
Node CPU = 95%
```

Do not immediately scale nodes.

Investigate:

```text
Which pods consume CPU?
Are CPU requests correct?
Is HPA scaling?
Is there a runaway process?
Is a deployment causing the spike?
```

Use pod/container metrics to identify the workload.

---

# 116. Real-World Incident: Node Memory Pressure

Symptoms:

```text
Memory usage high
Pods evicted
OOMKilled
Node MemoryPressure
```

Investigate:

```text
Pod memory usage
Memory limits
Requests
Node capacity
Application behavior
Memory leaks
```

---

# 117. Real-World Incident: Pods Pending

Prometheus shows increasing Pending pods.

Investigate:

```text
Node capacity
CPU requests
Memory requests
Taints
Tolerations
Affinity
PVC
Node groups
```

Then:

```bash
kubectl describe pod <pod>
```

to identify scheduler events.

---

# 118. Real-World Incident: Monitoring Suddenly Stops

Suppose dashboards show no new data.

First determine:

```text
Prometheus down?
Targets down?
Prometheus storage problem?
Grafana problem?
Network problem?
```

Check:

```bash
kubectl get pods -n monitoring
```

Then inspect Prometheus logs.

---

# 119. Monitoring Deployment Strategy

For production:

```text
Git
 ↓
Pull Request
 ↓
Review
 ↓
CI validation
 ↓
ArgoCD
 ↓
EKS
```

Monitoring configuration should be treated like application infrastructure.

---

# 120. Monitoring Configuration Repository

A practical repository structure:

```text
monitoring/
├── prometheus/
│   ├── values.yaml
│   └── rules/
├── grafana/
│   ├── dashboards/
│   └── datasources/
├── exporters/
│   ├── node-exporter/
│   ├── blackbox-exporter/
│   └── database-exporters/
├── servicemonitors/
├── podmonitors/
└── alerts/
```

---

# 121. Production Kubernetes Monitoring Checklist

```text
[ ] Prometheus installed
[ ] Prometheus Operator configured
[ ] Persistent storage configured
[ ] Node Exporter deployed
[ ] Kube-State-Metrics deployed
[ ] Application metrics enabled
[ ] ServiceMonitors configured
[ ] PodMonitors configured where required
[ ] RBAC configured
[ ] NetworkPolicy configured
[ ] Grafana configured
[ ] Alerting configured
[ ] Dashboards created
[ ] Retention configured
[ ] Resource requests/limits configured
[ ] Prometheus itself monitored
```

---

# 122. Application Monitoring Checklist

For every production microservice:

```text
[ ] /metrics endpoint
[ ] Request rate
[ ] Error rate
[ ] Latency
[ ] CPU
[ ] Memory
[ ] Restart count
[ ] Pod availability
[ ] Dependency metrics
[ ] Alerts
[ ] Dashboard
```

---

# 123. Node Monitoring Checklist

```text
[ ] CPU
[ ] Memory
[ ] Disk
[ ] Filesystem
[ ] Network
[ ] Node Ready
[ ] DiskPressure
[ ] MemoryPressure
[ ] PIDPressure
[ ] Pod capacity
```

---

# 124. Kubernetes Object Monitoring Checklist

```text
[ ] Deployment replicas
[ ] StatefulSet replicas
[ ] DaemonSet availability
[ ] Pod phases
[ ] Pod restarts
[ ] Job failures
[ ] CronJob status
[ ] PVC status
[ ] Node conditions
```

---

# 125. Interview Question: How Do You Monitor Kubernetes?

A strong answer:

```text
"I monitor Kubernetes at multiple layers.

For node-level infrastructure metrics, I use Node Exporter.

For Kubernetes object state such as deployments, pods, replica availability and node conditions, I use kube-state-metrics.

For application-level monitoring, applications expose Prometheus metrics.

In Kubernetes, I use Prometheus Operator with ServiceMonitor and PodMonitor resources to manage scraping declaratively.

Prometheus collects and stores the metrics, Grafana provides dashboards, and alerting handles actionable conditions.

In an EKS environment I also integrate AWS service metrics where appropriate, especially for infrastructure such as ALB and other AWS-managed services."
```

---

# 126. Interview Question: Why Do You Need Kube-State-Metrics?

Answer:

```text
"Kube-state-metrics exposes metrics about the state of Kubernetes objects.

For example, it can tell me the desired and available replicas of a Deployment, the phase of Pods, node conditions, and PVC state.

It does not replace Node Exporter.

Node Exporter gives me operating-system and node-level metrics, while kube-state-metrics gives me Kubernetes object-state information."
```

---

# 127. Interview Question: How Do You Monitor Pods?

Answer:

```text
"I monitor pods from multiple perspectives.

First, I monitor Kubernetes object state such as pod phase, readiness and restart counts.

Second, I monitor container resource usage such as CPU and memory.

Third, I monitor application-level metrics such as request rate, latency and errors.

For troubleshooting, I correlate these metrics with Kubernetes events and application logs."
```

---

# 128. Interview Question: How Do You Monitor a Deployment?

Answer:

```text
"I monitor desired replicas, updated replicas, available replicas and unavailable replicas.

For example, if a deployment has five desired replicas but only three are available, I investigate the affected pods, readiness probes, scheduling, image issues, resources and application startup.

I also alert when the replica gap remains for an appropriate duration."
```

---

# 129. Interview Question: How Do You Monitor Kubernetes Nodes?

Answer:

```text
"I deploy Node Exporter as a DaemonSet so that each node exposes host-level metrics.

I monitor CPU, memory, filesystem, disk, network and node conditions such as Ready, DiskPressure and MemoryPressure.

I also monitor allocatable resources and pod capacity because scheduling depends on resource requests, not only actual utilization."
```

---

# 130. Interview Question: What Is the Difference Between Node Metrics and Kubernetes State Metrics?

Answer:

```text
"Node metrics tell me how the underlying operating system is behaving.

For example:
CPU, memory, disk and network.

Kubernetes state metrics tell me what Kubernetes believes about its objects.

For example:
desired replicas, available replicas, pod phase and node conditions.

I need both to understand the complete health of the cluster."
```

---

# 131. Interview Question: What Happens When a Pod Is Pending?

Answer:

```text
"I first check whether the issue is related to scheduling.

I look at CPU and memory requests, node capacity, taints and tolerations, affinity rules, node selectors and PVC availability.

I then use kubectl describe pod to inspect scheduler events.

From Prometheus, I can correlate the issue with cluster capacity and node utilization."
```

---

# 132. Interview Question: How Do You Monitor CrashLoopBackOff?

Answer:

```text
"I monitor pod restart counts and container state.

When I see repeated restarts, I check the affected pod using kubectl describe pod and kubectl logs --previous.

I investigate OOMKilled, application startup errors, configuration problems, dependency failures and liveness probe failures.

Prometheus tells me that the problem is occurring and helps establish when it started; Kubernetes events and logs help identify the cause."
```

---

# 133. Interview Question: How Do You Monitor Kubernetes Applications?

Answer:

```text
"I instrument the application to expose Prometheus metrics.

For each microservice, I monitor request rate, error rate and latency, along with CPU, memory, pod availability and restarts.

I use ServiceMonitor or PodMonitor through Prometheus Operator to discover the metrics endpoint.

Then I create Grafana dashboards and alerts around the Golden Signals and infrastructure saturation."
```

---

# 134. Interview Question: How Do You Monitor EKS?

Answer:

```text
"For EKS, I use layered monitoring.

At the node level, I use Node Exporter.

For Kubernetes object state, I use kube-state-metrics.

For applications, I use application Prometheus instrumentation.

Prometheus Operator manages ServiceMonitors and PodMonitors.

I also integrate AWS-native metrics for services such as ALB where appropriate.

Grafana provides dashboards and Prometheus-based alerting handles actionable conditions."
```

---

# 135. Interview Question: How Would You Troubleshoot a Missing ServiceMonitor Target?

Answer:

```text
"I would check the monitoring chain from the top down.

First, I verify that Prometheus is selecting the ServiceMonitor.

Then I verify that the ServiceMonitor selector matches the intended Service.

Then I check that the Service selects the correct Pods and has endpoints.

After that I verify the metrics port and path.

Finally, I check RBAC, namespace selectors and relabeling.

If the target appears but is DOWN, I move to connectivity and application-level troubleshooting."
```

---

# 136. Interview Question: How Do You Monitor Kubernetes Capacity?

Answer:

```text
"I monitor both actual resource utilization and Kubernetes scheduling capacity.

I look at node CPU and memory capacity, allocatable resources, resource requests, pod capacity and Pending pods.

This distinction is important because a node can have relatively low actual CPU utilization while still being unable to schedule a new pod if the requested resources do not fit."
```

---

# 137. Interview Question: How Do You Monitor a Production Microservices Platform?

Answer:

```text
"I use layered monitoring.

At the infrastructure layer I monitor nodes using Node Exporter.

At the Kubernetes layer I monitor object state with kube-state-metrics.

At the application layer I collect request rate, latency and errors from application instrumentation.

For databases and other systems without native Prometheus metrics, I use appropriate exporters.

For external endpoint availability, I can use Blackbox Exporter.

Prometheus collects the metrics, Grafana visualizes them, and alerting detects actionable failures.

I also manage the monitoring configuration through GitOps so changes are version-controlled and auditable."
```

---

# 138. Production Monitoring Mental Model

Remember this architecture:

```text
                         KUBERNETES
                              │
        ┌─────────────────────┼──────────────────────┐
        ↓                     ↓                      ↓
      NODES                  PODS                OBJECTS
        │                     │                      │
        ↓                     ↓                      ↓
 Node Exporter         Application Metrics    Kube-State-Metrics
        │                     │                      │
        └─────────────────────┼──────────────────────┘
                              ↓
                       Service Discovery
                              ↓
                     Prometheus Operator
                              ↓
                           Prometheus
                              │
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
               TSDB        Alerting      PromQL
                 │                         │
                 └────────────┬────────────┘
                              ↓
                           Grafana
```

---

# 139. The Most Important Kubernetes Monitoring Concepts

For interviews and production work, remember:

```text
Prometheus Operator
ServiceMonitor
PodMonitor
Kube-State-Metrics
Node Exporter
Application Metrics
Kubernetes API
RBAC
Resource Requests
Resource Limits
CPU
Memory
Restarts
OOMKilled
Pod Availability
Deployment Replicas
Node Conditions
Persistent Volumes
HPA
Cluster Autoscaling
Service Discovery
Prometheus
Grafana
Alerting
```

---

# 140. Final Production Architecture

A production-grade EKS monitoring architecture can be summarized as:

```text
                              AWS
                               │
                ┌──────────────┴──────────────┐
                │                             │
               EKS                           ALB
                │                             │
      ┌─────────┼─────────┐             AWS Metrics
      ↓         ↓         ↓                   │
    Nodes      Pods     Objects               │
      │         │         │                   │
      ↓         ↓         ↓                   │
Node Exporter  Apps   Kube-State-Metrics      │
      │         │         │                   │
      └─────────┼─────────┴───────────────────┘
                ↓
        Kubernetes Discovery
                ↓
       Prometheus Operator
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
 ServiceMonitor PodMonitor PrometheusRule
       │        │        │
       └────────┼────────┘
                ↓
            Prometheus
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
      TSDB    Alerting  PromQL
                │
                ↓
             Grafana
                │
                ↓
          DevOps / SRE Team
```

The key principle is:

```text
Kubernetes monitoring is not just
"install Prometheus."

It is:

Infrastructure
    +
Kubernetes state
    +
Application metrics
    +
Service discovery
    +
Dashboards
    +
Alerting
    +
Capacity planning
    +
Troubleshooting
```

A production DevOps engineer should be able to move from:

```text
User symptom
     ↓
Application metrics
     ↓
Pod metrics
     ↓
Kubernetes state
     ↓
Node metrics
     ↓
Infrastructure
```

and identify where the failure is occurring.
