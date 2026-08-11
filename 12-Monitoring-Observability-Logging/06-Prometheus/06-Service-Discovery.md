# Prometheus Service Discovery

Prometheus needs to know **what targets to scrape**.

In a small environment, we can manually configure targets:

```yaml
scrape_configs:
  - job_name: "node-exporter"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
          - "10.0.1.12:9100"
```

This works for a small number of servers.

But in a production environment:

```text
EC2 instances are created
Pods are created
Pods are destroyed
Nodes scale
Services change
Containers restart
IP addresses change
Applications are deployed
```

Manually maintaining Prometheus targets becomes difficult.

This is where **Service Discovery** becomes important.

---

# 1. What Is Service Discovery?

Service discovery is the process through which Prometheus automatically discovers targets that should be monitored.

Instead of manually configuring every target:

```text
Prometheus
    ↓
Service Discovery
    ↓
Discover Targets
    ↓
Scrape Metrics
```

For example, in Kubernetes:

```text
Kubernetes API
      ↓
Prometheus
      ↓
Discover Pods
      ↓
Discover Services
      ↓
Discover Nodes
      ↓
Scrape Metrics
```

---

# 2. Why Service Discovery Is Required

Consider a Kubernetes production environment.

Suppose we have:

```text
order-service
payment-service
inventory-service
user-service
```

Each service may have multiple pods.

Initially:

```text
order → pod-1
payment → pod-2
inventory → pod-3
user → pod-4
```

Later HPA scales:

```text
order → pod-1
        pod-5
        pod-6
```

Pods can also be recreated with new IP addresses.

If Prometheus used static IP addresses, we would need to constantly update its configuration.

With service discovery:

```text
Kubernetes
    ↓
Prometheus discovers new pod
    ↓
Prometheus discovers new IP
    ↓
Prometheus starts scraping
```

No manual target update is required.

---

# 3. Static Configuration vs Service Discovery

## Static Configuration

```text
Prometheus
    ↓
Hardcoded Targets
    ↓
10.0.1.10:9100
10.0.1.11:9100
10.0.1.12:9100
```

Advantages:

```
Simple
Easy to understand
Good for small environments
```

Disadvantages:

```
Manual maintenance
IP changes require updates
Poor scalability
Configuration drift
```

---

# 4. Dynamic Service Discovery

```text
Prometheus
      ↓
Service Discovery
      ↓
Target Registry / API
      ↓
Current Targets
      ↓
Scraping
```

Advantages:

```
Automatic discovery
Scales with infrastructure
Handles changing IP addresses
Better for cloud environments
Better for Kubernetes
```

---

# 5. Prometheus Service Discovery Architecture

The general architecture is:

```text
                    ┌───────────────────┐
                    │    Prometheus     │
                    └─────────┬─────────┘
                              │
                              ↓
                    ┌───────────────────┐
                    │ Service Discovery │
                    └─────────┬─────────┘
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
      Kubernetes             AWS               Consul
          ↓                   ↓                   ↓
       Targets             Targets             Targets
          └───────────────────┼───────────────────┘
                              ↓
                         Scrape Metrics
```

---

# 6. Prometheus Discovery Process

Prometheus generally follows this workflow:

```text
1. Contact discovery source
          ↓
2. Discover targets
          ↓
3. Attach metadata
          ↓
4. Apply relabeling
          ↓
5. Determine scrape address
          ↓
6. Scrape target
          ↓
7. Store metrics
```

---

# 7. What Is a Target?

A target is an endpoint from which Prometheus collects metrics.

Example:

```text
10.0.1.10:9100
```

If Node Exporter is running on that server:

```text
http://10.0.1.10:9100/metrics
```

is the metrics endpoint.

---

# 8. Target Labels

Prometheus associates labels with targets.

Example:

```text
job="node-exporter"
instance="10.0.1.10:9100"
environment="production"
region="ap-south-1"
```

Labels are extremely important because they allow PromQL filtering and aggregation.

Example:

```promql
up{environment="production"}
```

---

# 9. `static_configs`

Static configuration is the simplest approach.

Example:

```yaml
scrape_configs:
  - job_name: "node-exporter"

    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
```

Prometheus directly uses these addresses.

---

# 10. File-Based Service Discovery

Prometheus can also discover targets from files.

Example:

```yaml
scrape_configs:
  - job_name: "node-exporter"

    file_sd_configs:
      - files:
          - "/etc/prometheus/targets/*.yml"
```

The external file contains targets.

---

# 11. File SD Example

Example:

```yaml
- targets:
    - "10.0.1.10:9100"
    - "10.0.1.11:9100"

  labels:
    environment: "production"
    team: "platform"
```

Prometheus reads this file and dynamically updates its target list.

---

# 12. Why File SD Is Useful

File-based discovery is useful when:

```
An external system manages target information

Infrastructure automation generates target files

You do not have a native discovery integration

You want separation between Prometheus configuration and target inventory
```

For example:

```text
Terraform / Ansible
        ↓
Generate targets.yml
        ↓
Prometheus File SD
        ↓
Targets
```

---

# 13. Kubernetes Service Discovery

Kubernetes is one of the most important Prometheus service discovery integrations.

Prometheus can discover:

```text
Nodes
Pods
Services
Endpoints
EndpointSlices
Ingresses
```

depending on the configured discovery role and Prometheus version/environment.

---

# 14. Kubernetes Architecture

```text
                    Kubernetes Cluster
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
           Nodes          Pods        Services
             │             │             │
             └─────────────┼─────────────┘
                           ↓
                  Kubernetes API
                           ↓
                      Prometheus
                           ↓
                      Relabeling
                           ↓
                       Scraping
```

---

# 15. Kubernetes API

Prometheus communicates with the Kubernetes API to discover resources.

Conceptually:

```text
Prometheus
     ↓
Kubernetes API
     ↓
Pods / Services / Nodes
     ↓
Metadata
     ↓
Target List
```

Prometheus does not need to manually track every pod IP.

---

# 16. Kubernetes Roles

A Kubernetes discovery configuration can specify roles such as:

```text
node
pod
service
endpoints
endpointslice
ingress
```

The role determines what Kubernetes objects Prometheus discovers.

---

# 17. `role: pod`

Example:

```yaml
scrape_configs:
  - job_name: "kubernetes-pods"

    kubernetes_sd_configs:
      - role: pod
```

This allows Prometheus to discover pods.

---

# 18. `role: service`

Example:

```yaml
scrape_configs:
  - job_name: "kubernetes-services"

    kubernetes_sd_configs:
      - role: service
```

This discovers Kubernetes Services.

However, discovering a Service does not automatically mean the service is exposing Prometheus metrics.

The discovered target still needs to be configured correctly for scraping.

---

# 19. `role: node`

Example:

```yaml
scrape_configs:
  - job_name: "kubernetes-nodes"

    kubernetes_sd_configs:
      - role: node
```

This discovers Kubernetes nodes.

This is commonly used for node-level monitoring.

---

# 20. `role: endpoints`

Example:

```yaml
scrape_configs:
  - job_name: "kubernetes-endpoints"

    kubernetes_sd_configs:
      - role: endpoints
```

This discovers service endpoints.

---

# 21. `role: endpointslice`

Modern Kubernetes environments commonly use EndpointSlices.

Example:

```yaml
scrape_configs:
  - job_name: "kubernetes-endpointslices"

    kubernetes_sd_configs:
      - role: endpointslice
```

EndpointSlices are Kubernetes' scalable representation of service endpoints.

---

# 22. `role: ingress`

Example:

```yaml
scrape_configs:
  - job_name: "kubernetes-ingress"

    kubernetes_sd_configs:
      - role: ingress
```

This discovers Kubernetes Ingress resources.

The usefulness of this role depends on how the Ingress controller exposes metrics.

---

# 23. Kubernetes Pod Discovery

Suppose we have:

```text
Namespace: production

Pods:

order-7f6d9
payment-6b87f
inventory-54d92
```

Prometheus can discover these dynamically.

When a pod is deleted:

```text
order-7f6d9
```

Prometheus removes the corresponding target.

When a new pod appears:

```text
order-8c7f2
```

Prometheus discovers it.

---

# 24. Why Pod IP Discovery Is Important

Kubernetes pod IP addresses are ephemeral.

Example:

```text
order-pod
IP = 10.244.2.15
```

After restart:

```text
order-pod
IP = 10.244.3.21
```

Static configuration would break.

Kubernetes service discovery automatically follows the new target.

---

# 25. Kubernetes Labels

Kubernetes objects have labels.

Example:

```yaml
metadata:
  labels:
    app: order
    environment: production
    team: payments
```

Prometheus can use Kubernetes metadata during target discovery.

---

# 26. Kubernetes Annotations

Historically, Prometheus configurations often used annotations to control scraping.

Example:

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/path: "/metrics"
    prometheus.io/port: "8080"
```

These annotations can be consumed by an appropriately configured Prometheus scrape job.

---

# 27. Annotation-Based Scraping

Conceptually:

```text
Pod
 │
 ├── prometheus.io/scrape = true
 ├── prometheus.io/path = /metrics
 └── prometheus.io/port = 8080
          ↓
Kubernetes Discovery
          ↓
Prometheus
          ↓
Scrape
```

---

# 28. Important Modern Kubernetes Practice

In modern Kubernetes environments, **Prometheus Operator and PodMonitor/ServiceMonitor resources** are often preferred over manually maintaining annotation-heavy Prometheus configurations.

The architecture becomes:

```text
Application
    ↓
Service / Pod
    ↓
ServiceMonitor / PodMonitor
    ↓
Prometheus Operator
    ↓
Prometheus
```

This provides a more Kubernetes-native way to manage monitoring configuration.

---

# 29. ServiceMonitor

A `ServiceMonitor` tells Prometheus Operator how to discover and scrape Kubernetes Services.

Conceptually:

```text
Service
   ↓
ServiceMonitor
   ↓
Prometheus Operator
   ↓
Prometheus Configuration
   ↓
Scraping
```

---

# 30. Example ServiceMonitor

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

The exact namespace and selector configuration must match your cluster.

---

# 31. How ServiceMonitor Works

Suppose the Service:

```yaml
metadata:
  labels:
    app: order
```

ServiceMonitor:

```yaml
selector:
  matchLabels:
    app: order
```

The ServiceMonitor selects the Service.

Then:

```text
Service
   ↓
metrics port
   ↓
/metrics
   ↓
Prometheus
```

---

# 32. PodMonitor

`PodMonitor` can discover and scrape pods directly.

Conceptually:

```text
Pod
 ↓
PodMonitor
 ↓
Prometheus Operator
 ↓
Prometheus
```

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

---

# 33. ServiceMonitor vs PodMonitor

### ServiceMonitor

Used when:

```text
Service
  ↓
Application
```

is the desired discovery path.

### PodMonitor

Used when:

```text
Pod
  ↓
Application
```

should be scraped directly.

A common production setup uses ServiceMonitor when a Kubernetes Service provides a stable abstraction, while PodMonitor is useful when direct pod scraping is required.

---

# 34. Prometheus Operator Architecture

A common production architecture is:

```text
                 Kubernetes API
                       │
                       ↓
               Prometheus Operator
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
   ServiceMonitor  PodMonitor   PrometheusRule
          │            │            │
          └────────────┼────────────┘
                       ↓
                  Prometheus
                       ↓
                    TSDB
                       ↓
                    Grafana
```

---

# 35. What Prometheus Operator Does

Prometheus Operator manages Prometheus configuration using Kubernetes resources.

Instead of editing a large Prometheus configuration manually, engineers can create:

```text
Prometheus
ServiceMonitor
PodMonitor
PrometheusRule
Alertmanager
```

resources.

---

# 36. AWS EC2 Service Discovery

Prometheus can discover EC2 instances using AWS APIs.

Example:

```yaml
scrape_configs:
  - job_name: "ec2"

    ec2_sd_configs:
      - region: ap-south-1
```

Prometheus can obtain instance metadata from AWS.

---

# 37. EC2 Discovery Architecture

```text
AWS EC2
   │
   ↓
AWS API
   │
   ↓
Prometheus EC2 SD
   │
   ↓
Discovered Instances
   │
   ↓
Relabeling
   │
   ↓
Node Exporter
   │
   ↓
Prometheus
```

---

# 38. EC2 Discovery with Tags

EC2 instances commonly have tags:

```text
Environment=production
Application=order
Team=platform
```

Prometheus can use EC2 metadata and relabeling to turn this information into Prometheus labels.

This allows queries such as:

```promql
up{environment="production"}
```

---

# 39. EC2 Relabeling Concept

AWS discovery may expose metadata such as:

```text
instance_id
private_ip
availability_zone
instance_type
tags
```

Relabeling can convert metadata into useful Prometheus labels.

For example:

```text
AWS Tag
   ↓
Environment=production
   ↓
Prometheus Label
   ↓
environment="production"
```

---

# 40. Why EC2 Service Discovery Is Useful

Without discovery:

```text
New EC2
   ↓
Update Prometheus
   ↓
Reload configuration
```

With discovery:

```text
New EC2
   ↓
AWS API
   ↓
Prometheus discovers it
   ↓
Scrape
```

This is especially useful with:

```text
Auto Scaling Groups
```

---

# 41. EC2 Auto Scaling Example

Suppose an ASG initially has:

```text
3 EC2 instances
```

During high traffic:

```text
3 → 6 instances
```

Static Prometheus configuration would need to be updated.

With EC2 service discovery:

```text
ASG
 ↓
New Instances
 ↓
AWS API
 ↓
Prometheus
 ↓
New Targets
```

---

# 42. Consul Service Discovery

Prometheus can also integrate with Consul.

Architecture:

```text
Applications
    ↓
Consul
    ↓
Service Registry
    ↓
Prometheus
    ↓
Targets
```

Example:

```yaml
scrape_configs:
  - job_name: "consul-services"

    consul_sd_configs:
      - server: "consul.example.com:8500"
```

---

# 43. Consul Use Case

Suppose applications register:

```text
order-service
payment-service
inventory-service
```

in Consul.

Prometheus discovers those services through Consul.

This is useful in environments where Consul acts as the service registry.

---

# 44. DNS Service Discovery

Prometheus also supports DNS-based discovery.

Example:

```yaml
scrape_configs:
  - job_name: "dns"

    dns_sd_configs:
      - names:
          - "_metrics._tcp.example.com"
```

DNS can return dynamically changing targets.

---

# 45. DNS Discovery Architecture

```text
Prometheus
    ↓
DNS
    ↓
Service Records
    ↓
Targets
    ↓
Scraping
```

This can be useful when DNS is already the source of truth for service locations.

---

# 46. Kubernetes vs EC2 vs Consul

| Environment              | Discovery      |
| ------------------------ | -------------- |
| Kubernetes               | Kubernetes SD  |
| AWS EC2                  | EC2 SD         |
| Consul                   | Consul SD      |
| DNS-managed systems      | DNS SD         |
| External inventory       | File SD        |
| Small static environment | Static configs |

---

# 47. Service Discovery and Relabeling

Service discovery tells Prometheus:

```text
"What targets exist?"
```

Relabeling determines:

```text
"Which targets should I scrape and how should I label them?"
```

This distinction is extremely important.

---

# 48. Relabeling Architecture

```text
Service Discovery
       ↓
Discovered Targets
       ↓
relabel_configs
       ↓
Filtered / Modified Targets
       ↓
Scraping
```

---

# 49. `relabel_configs`

Example:

```yaml
relabel_configs:
  - source_labels: [__meta_kubernetes_namespace]
    target_label: namespace
```

This takes Kubernetes metadata and creates a Prometheus label.

---

# 50. Internal Discovery Labels

Service discovery generates internal labels beginning with:

```text
__meta_
```

Examples:

```text
__meta_kubernetes_namespace
__meta_kubernetes_pod_name
__meta_kubernetes_pod_label_app
```

These can be used during relabeling.

---

# 51. `__address__`

The target address is stored internally as:

```text
__address__
```

Example:

```text
10.244.1.20:8080
```

Relabeling can modify the target address.

---

# 52. `__metrics_path__`

The metrics path is represented internally as:

```text
__metrics_path__
```

Typically:

```text
/metrics
```

Relabeling can modify it if necessary.

---

# 53. `__scheme__`

The scrape scheme can be:

```text
http
```

or:

```text
https
```

The internal label is:

```text
__scheme__
```

---

# 54. Kubernetes Metadata Labels

Kubernetes discovery can expose metadata such as:

```text
namespace
pod name
pod labels
pod annotations
node name
container name
container port
```

These metadata values can be used to construct target labels.

---

# 55. Example: Kubernetes Namespace Label

```yaml
relabel_configs:
  - source_labels:
      - __meta_kubernetes_namespace

    target_label: namespace
```

Result:

```text
namespace="production"
```

---

# 56. Example: Kubernetes Pod Label

Suppose pod has:

```yaml
labels:
  app: order
```

A discovery label can represent the pod label.

Relabeling can expose it as:

```text
app="order"
```

---

# 57. Keep Only Production Targets

Suppose discovered targets contain:

```text
environment=production
environment=staging
environment=development
```

You can filter targets with relabeling.

Conceptually:

```yaml
relabel_configs:
  - source_labels:
      - __meta_kubernetes_namespace
    regex: production
    action: keep
```

This keeps only matching targets.

---

# 58. `action: keep`

`keep` keeps targets whose source label matches the configured regex.

Example:

```yaml
- source_labels: [environment]
  regex: production
  action: keep
```

---

# 59. `action: drop`

`drop` removes matching targets.

Example:

```yaml
- source_labels: [environment]
  regex: development
  action: drop
```

This prevents development targets from being scraped.

---

# 60. Why Drop Development Targets?

Suppose Prometheus monitors:

```text
production
staging
development
```

but production Prometheus should only monitor production.

Instead of scraping everything:

```text
Prometheus
    ↓
Discover All
    ↓
Drop Development
    ↓
Scrape Production
```

This reduces unnecessary monitoring load.

---

# 61. `action: replace`

`replace` can create or modify labels.

Example:

```yaml
- source_labels: [__meta_kubernetes_namespace]
  target_label: namespace
  action: replace
```

---

# 62. `action: labelmap`

`labelmap` can copy matching discovery labels into Prometheus labels.

Conceptually:

```yaml
- regex: __meta_kubernetes_pod_label_(.+)
  action: labelmap
```

This can expose Kubernetes pod labels.

Use label mapping carefully because copying many Kubernetes labels can increase cardinality.

---

# 63. `action: labeldrop`

Example:

```yaml
- regex: temporary_.*
  action: labeldrop
```

This removes unwanted labels.

---

# 64. `action: labelkeep`

Example:

```yaml
- regex: service|namespace|environment
  action: labelkeep
```

This keeps only selected labels.

This can help control label growth.

---

# 65. Target Relabeling vs Metric Relabeling

There are two important concepts:

```text
relabel_configs
```

and:

```text
metric_relabel_configs
```

They operate at different stages.

---

# 66. Target Relabeling

```text
Discovery
   ↓
relabel_configs
   ↓
Target
   ↓
Scrape
```

Target relabeling can:

```
Keep targets

Drop targets

Change target address

Create labels

Modify discovery metadata
```

---

# 67. Metric Relabeling

Metric relabeling happens after scraping.

```text
Target
  ↓
Scrape
  ↓
Metrics
  ↓
metric_relabel_configs
  ↓
Store
```

This can filter or modify individual metrics.

---

# 68. Why Metric Relabeling Matters

Suppose an exporter exposes:

```text
1000 metrics
```

but you only need:

```text
200 metrics
```

Metric relabeling can drop unnecessary metrics before storage.

---

# 69. Example Metric Drop

Conceptually:

```yaml
metric_relabel_configs:
  - source_labels: [__name__]
    regex: "unwanted_metric_.*"
    action: drop
```

This prevents matching metrics from being stored.

---

# 70. Target Drop vs Metric Drop

### Target Drop

```text
Don't scrape the target.
```

### Metric Drop

```text
Scrape the target,
but don't store selected metrics.
```

This distinction is important for production tuning.

---

# 71. Service Discovery and Security

Prometheus needs permission to discover targets.

For Kubernetes, this commonly means RBAC permissions.

Architecture:

```text
Prometheus Pod
      ↓
ServiceAccount
      ↓
RBAC
      ↓
Kubernetes API
```

---

# 72. Kubernetes RBAC

Prometheus may need access to resources such as:

```text
pods
services
endpoints
endpointslices
nodes
ingresses
```

The exact permissions depend on the discovery roles being used.

---

# 73. Why RBAC Is Important

Do not give Prometheus unnecessary permissions.

Follow:

```text
Least Privilege
```

Prometheus should have only the Kubernetes API permissions needed for monitoring.

---

# 74. Service Discovery in EKS

A real-world EKS monitoring architecture may look like:

```text
                    AWS
                     │
             ┌───────┴────────┐
             │                │
            EKS              EC2
             │                │
      Kubernetes API      EC2 Metadata
             │                │
             ↓                ↓
         Prometheus ──────────┘
             │
             ↓
          Scraping
             │
      ┌──────┴──────┐
      ↓             ↓
   Metrics       Exporters
      │
      ↓
   Prometheus
      │
      ↓
    Grafana
```

---

# 75. EKS Pod Discovery

Suppose:

```text
namespace = production

order
payment
inventory
```

Prometheus discovers pods.

Each pod may have:

```text
pod IP
namespace
pod name
labels
container
ports
```

Prometheus uses this metadata to determine whether and where to scrape.

---

# 76. EKS Node Discovery

Prometheus can discover Kubernetes nodes.

The node exporter may expose:

```text
CPU
Memory
Disk
Filesystem
Network
Load
```

Prometheus can scrape node metrics.

---

# 77. EKS Service Discovery with ServiceMonitor

A production Kubernetes stack commonly uses:

```text
Application
    ↓
Service
    ↓
ServiceMonitor
    ↓
Prometheus Operator
    ↓
Prometheus
```

This avoids maintaining a large manually edited scrape configuration.

---

# 78. Example Production Application

Suppose we deploy:

```text
order-service
```

with:

```text
port: 8080
metrics port: 9090
metrics path: /metrics
```

Kubernetes Service:

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
    - name: http
      port: 8080
      targetPort: 8080

    - name: metrics
      port: 9090
      targetPort: 9090
```

---

# 79. ServiceMonitor for the Application

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor

metadata:
  name: order-service

spec:
  selector:
    matchLabels:
      app: order

  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
```

Flow:

```text
Order Pod
   ↓
Service
   ↓
ServiceMonitor
   ↓
Prometheus Operator
   ↓
Prometheus
   ↓
/metrics
```

---

# 80. ServiceMonitor Namespace Consideration

A ServiceMonitor does not necessarily discover Services in every namespace automatically.

The Prometheus resource configuration controls which namespaces and ServiceMonitors it watches.

Therefore, in production, verify:

```text
Prometheus namespace selector
ServiceMonitor namespace
Service selector
Service labels
```

---

# 81. ServiceMonitor Selector Problems

A common issue:

```text
Service exists
ServiceMonitor exists
But Prometheus doesn't scrape it.
```

Possible causes:

```text
Wrong namespace
Wrong Service selector
Wrong ServiceMonitor selector
Prometheus not selecting ServiceMonitor
Wrong port name
Wrong path
RBAC issue
Target does not expose metrics
```

---

# 82. Troubleshooting Service Discovery

Start from:

```text
Is target discovered?
```

Then:

```text
Is target up?
```

Then:

```text
Can Prometheus reach it?
```

Then:

```text
Does /metrics exist?
```

Then:

```text
Are metrics valid?
```

---

# 83. Prometheus Targets Page

Prometheus provides a Targets page.

Typically:

```text
Status
   ↓
Targets
```

You can see:

```text
Endpoint
State
Labels
Last Scrape
Scrape Duration
Error
```

This is one of the first places to troubleshoot service discovery.

---

# 84. Target States

A target can commonly appear as:

```text
UP
DOWN
UNKNOWN / pending discovery behavior
```

The most important distinction:

```text
Discovered ≠ Successfully Scraped
```

A target can be discovered but still be DOWN.

---

# 85. Discovered but DOWN

Example:

```text
Target:
10.244.1.20:9090

State:
DOWN

Error:
connection refused
```

This means service discovery worked.

The problem is now with:

```text
Connectivity
Port
Application
NetworkPolicy
Endpoint
```

---

# 86. Target Not Discovered

If the target does not appear at all:

Investigate:

```text
Service Discovery
RBAC
Selectors
Namespace
Relabeling
ServiceMonitor / PodMonitor
Prometheus configuration
```

---

# 87. Target Discovery Debugging Workflow

```text
Application
    ↓
Pod
    ↓
Service
    ↓
ServiceMonitor
    ↓
Prometheus Operator
    ↓
Prometheus Target
    ↓
Scrape
```

Check every layer.

---

# 88. Kubernetes Monitoring Debug Commands

Check pods:

```bash
kubectl get pods -n production
```

Check services:

```bash
kubectl get svc -n production
```

Check ServiceMonitor:

```bash
kubectl get servicemonitor -A
```

Check PodMonitor:

```bash
kubectl get podmonitor -A
```

---

# 89. Inspect ServiceMonitor

```bash
kubectl describe servicemonitor order-service -n monitoring
```

Check:

```text
Selector
Endpoints
Namespace
Labels
```

---

# 90. Inspect Service

```bash
kubectl describe svc order-service -n production
```

Verify:

```text
Selector
Ports
TargetPort
Endpoints
```

---

# 91. Check Endpoints

Depending on your Kubernetes environment:

```bash
kubectl get endpoints -n production
```

or:

```bash
kubectl get endpointslice -n production
```

Verify that the Service actually has backend endpoints.

---

# 92. Test Metrics Endpoint

From a reachable environment:

```bash
curl http://<pod-ip>:9090/metrics
```

or through an appropriate Service endpoint.

If `/metrics` does not respond, Prometheus cannot scrape it successfully.

---

# 93. Test Inside Kubernetes

You can use a temporary troubleshooting pod:

```bash
kubectl run curl-test \
  --image=curlimages/curl \
  -it \
  --rm \
  -- sh
```

Then:

```bash
curl http://order-service.production.svc.cluster.local:9090/metrics
```

This verifies Kubernetes network connectivity.

---

# 94. NetworkPolicy and Scraping

A Kubernetes NetworkPolicy can block Prometheus.

Architecture:

```text
Prometheus
    ↓
NetworkPolicy
    X
Application
```

The target may exist but remain unreachable.

Check:

```text
Ingress policy
Namespace
Pod selector
Prometheus namespace
Prometheus pod labels
Port
```

---

# 95. Service Discovery and DNS

Kubernetes Services provide DNS names.

Example:

```text
order-service.production.svc.cluster.local
```

However, Prometheus Kubernetes service discovery can discover target addresses directly rather than requiring every target to be manually represented as a DNS hostname.

---

# 96. Service Discovery and Load Balancing

Prometheus should generally scrape application instances individually when the monitoring architecture is designed that way.

Avoid assuming that scraping a load-balanced application Service gives visibility into every backend instance.

For example:

```text
Service
 ├── Pod A
 ├── Pod B
 └── Pod C
```

A load-balanced Service endpoint can distribute requests, but monitoring requirements may call for discovering each pod directly.

This is one reason PodMonitor or endpoint-based discovery can be useful.

---

# 97. ServiceMonitor vs Application Service

A ServiceMonitor selects a Kubernetes Service using labels.

Example:

```text
Service:
app=order
```

ServiceMonitor:

```text
matchLabels:
  app=order
```

The selector relationship must be correct.

---

# 98. Label Mismatch Example

Service:

```yaml
labels:
  app: order-service
```

ServiceMonitor:

```yaml
selector:
  matchLabels:
    app: order
```

Result:

```text
No match
```

Prometheus will not discover the intended Service through that ServiceMonitor.

---

# 99. Port Name Mismatch

Service:

```yaml
ports:
  - name: metrics
    port: 9090
```

ServiceMonitor:

```yaml
endpoints:
  - port: prometheus
```

Result:

```text
No matching port
```

The ServiceMonitor must reference the appropriate port name.

---

# 100. Metrics Path Mismatch

Application exposes:

```text
/actuator/prometheus
```

but ServiceMonitor uses:

```yaml
path: /metrics
```

Result:

```text
404 Not Found
```

The scrape path must match the application's metrics endpoint.

---

# 101. Scrape Interval

Example:

```yaml
interval: 30s
```

means Prometheus attempts to scrape the target every 30 seconds.

Common values include:

```text
15s
30s
60s
```

The appropriate interval depends on:

```
Monitoring requirements
Target count
Prometheus capacity
Metric importance
```

---

# 102. Scrape Timeout

A scrape timeout limits how long Prometheus waits for the target.

Example:

```yaml
scrape_timeout: 10s
```

The timeout must be compatible with the scrape interval.

---

# 103. Scrape Interval vs Timeout

Example:

```text
interval = 30s
timeout  = 10s
```

Prometheus can spend up to 10 seconds waiting for the scrape, then retry on the next interval according to its normal scheduling behavior.

Do not configure unnecessarily long timeouts.

---

# 104. Large Kubernetes Cluster

Suppose:

```text
500 nodes
10,000 pods
1,000 services
```

Service discovery can generate a very large target set.

Therefore:

```text
Discovery
   ↓
Filtering
   ↓
Relabeling
   ↓
Scraping
```

must be designed carefully.

---

# 105. Why Filtering Matters

Imagine Prometheus discovers:

```text
10,000 pods
```

but only:

```text
2,000 pods
```

expose metrics.

If you scrape everything unnecessarily, you increase:

```text
CPU
Memory
Network
Storage
Query load
```

Use appropriate discovery and filtering.

---

# 106. High Cardinality

Service discovery itself is not the only source of cardinality.

Labels can create enormous numbers of time series.

Avoid using labels such as:

```text
request_id
user_id
session_id
transaction_id
```

unless there is a carefully controlled reason.

---

# 107. Discovery Metadata and Cardinality

Kubernetes has many labels and annotations.

Do not blindly copy every Kubernetes metadata field into Prometheus labels.

For example:

```text
pod_uid
controller_revision_hash
random deployment labels
temporary labels
```

may increase cardinality unnecessarily.

---

# 108. Production Label Strategy

Prefer stable labels such as:

```text
cluster
environment
namespace
service
app
region
team
```

Use carefully controlled labels for:

```text
version
instance
pod
```

depending on the monitoring use case.

---

# 109. Service Discovery in Multi-Environment Architecture

A common architecture:

```text
                    Git / Terraform
                          │
            ┌─────────────┼─────────────┐
            ↓             ↓             ↓
        Dev EKS       Stage EKS     Prod EKS
            │             │             │
            ↓             ↓             ↓
       Prometheus     Prometheus     Prometheus
            │             │             │
            └─────────────┼─────────────┘
                          ↓
                       Grafana
```

Each environment can have its own discovery configuration.

---

# 110. Multi-Cluster Monitoring

For larger organizations:

```text
Cluster A
   ↓
Prometheus

Cluster B
   ↓
Prometheus

Cluster C
   ↓
Prometheus

      ↓

Centralized / Global Metrics Architecture
```

Service discovery happens within each cluster.

---

# 111. Service Discovery with Federation

Prometheus federation can allow one Prometheus server to scrape selected metrics from another Prometheus.

Conceptually:

```text
Cluster Prometheus
        ↓
     /federate
        ↓
Global Prometheus
```

This is different from Kubernetes target discovery.

It is a separate Prometheus architecture pattern.

---

# 112. Service Discovery and Remote Write

Another architecture is:

```text
Local Prometheus
      ↓
Remote Write
      ↓
Long-Term Metrics Backend
```

Examples of long-term architectures include systems such as:

```text
Thanos
Cortex / Mimir
VictoriaMetrics
```

The discovery process still occurs at the local Prometheus layer.

---

# 113. Service Discovery in Production

A production design should answer:

```text
What is the source of truth?
```

Examples:

```text
Kubernetes → Kubernetes API
AWS → AWS API
Consul → Consul
DNS → DNS
External inventory → File SD
```

---

# 114. Source of Truth

A good architecture is:

```text
Infrastructure Source of Truth
             ↓
       Service Discovery
             ↓
          Prometheus
             ↓
           Metrics
```

Avoid manually maintaining duplicate target inventories when the infrastructure already provides a reliable discovery source.

---

# 115. Service Discovery Security

Important security considerations:

```text
RBAC
IAM
TLS
NetworkPolicy
Authentication
Authorization
Least Privilege
```

For AWS discovery:

```text
IAM permissions
```

may be required depending on the deployment model.

For Kubernetes:

```text
ServiceAccount
RBAC
```

are central.

---

# 116. AWS IAM for EC2 Discovery

When Prometheus discovers EC2 instances using AWS APIs, it needs appropriate AWS permissions.

In EKS, a production design may use:

```text
IRSA
```

or the current AWS-supported pod identity approach to provide AWS permissions without embedding long-lived credentials.

The exact mechanism depends on the EKS authentication design being used.

---

# 117. Service Discovery Failure Scenarios

Common failures:

```text
1. Target not discovered
2. Wrong namespace
3. Wrong selector
4. Wrong port
5. Wrong metrics path
6. RBAC denied
7. NetworkPolicy blocked
8. DNS problem
9. Target down
10. TLS problem
11. Authentication failure
12. Relabeling dropped target
```

---

# 118. Troubleshooting: Target Not Appearing

Check:

```text
1. Is ServiceMonitor selected by Prometheus?
2. Is the namespace allowed?
3. Does the selector match the Service?
4. Is the Service correctly labeled?
5. Is the metrics port correct?
6. Did relabeling drop the target?
```

---

# 119. Troubleshooting: Target DOWN

Check:

```text
1. Endpoint exists?
2. Port reachable?
3. Metrics path correct?
4. Application listening?
5. NetworkPolicy?
6. TLS?
7. Authentication?
8. Timeout?
```

---

# 120. Troubleshooting: No Metrics

If the target is UP but expected application metrics are missing:

Check:

```text
1. /metrics response
2. Metric name
3. Exporter configuration
4. Application instrumentation
5. Metric relabeling
6. Scrape configuration
```

---

# 121. Troubleshooting: Metrics Suddenly Disappear

Possible causes:

```text
Application deployment
Pod restart
Service changed
Port changed
Metrics endpoint changed
ServiceMonitor changed
Label changed
Prometheus configuration changed
NetworkPolicy changed
RBAC changed
```

---

# 122. Troubleshooting Workflow

Use:

```text
Prometheus Targets
        ↓
Target exists?
        ↓
YES
 ↓
Target UP?
        ↓
YES
 ↓
Metrics present?
        ↓
YES
 ↓
PromQL query correct?
        ↓
YES
 ↓
Grafana problem?
```

If target doesn't exist:

```text
Service Discovery / Selector / RBAC
```

If target is DOWN:

```text
Connectivity / Endpoint / Application
```

---

# 123. Production Architecture: EKS

A realistic architecture for an EKS-based microservices platform:

```text
                         AWS
                          │
                    ┌─────┴─────┐
                    │    EKS    │
                    └─────┬─────┘
                          │
          ┌───────────────┼────────────────┐
          ↓               ↓                ↓
      User Service    Order Service    Payment Service
          │               │                │
          ↓               ↓                ↓
       /metrics        /metrics         /metrics
          │               │                │
          └───────────────┼────────────────┘
                          ↓
                    Kubernetes API
                          ↓
                 Prometheus Operator
                          ↓
               ServiceMonitor / PodMonitor
                          ↓
                      Prometheus
                          ↓
                        TSDB
                          ↓
                       Grafana
```

---

# 124. Production Architecture: EC2

For EC2-based infrastructure:

```text
                  AWS
                   │
          ┌────────┴────────┐
          ↓                 ↓
       EC2 ASG          Other EC2
          │                 │
          ↓                 ↓
   Node Exporter      Node Exporter
          │                 │
          └────────┬────────┘
                   ↓
               AWS EC2 SD
                   ↓
               Prometheus
                   ↓
                Grafana
```

---

# 125. Production Architecture: Hybrid

An organization may have:

```text
             Prometheus
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
       EKS       EC2      Consul
        ↓         ↓         ↓
 Kubernetes     AWS      Services
    SD           SD         SD
        └─────────┼─────────┘
                  ↓
              Prometheus
                  ↓
               Grafana
```

---

# 126. Service Discovery Best Practices

## 1. Prefer Dynamic Discovery

Use:

```text
Kubernetes SD
EC2 SD
Consul SD
DNS SD
```

instead of manually maintaining large target lists.

---

# 127. Best Practice: Use Kubernetes-Native Monitoring

For Kubernetes environments, consider:

```text
Prometheus Operator
ServiceMonitor
PodMonitor
PrometheusRule
```

This makes monitoring configuration easier to manage through Kubernetes manifests and GitOps.

---

# 128. Best Practice: Use GitOps

Store monitoring configuration in Git:

```text
monitoring/
├── prometheus/
├── servicemonitors/
├── podmonitors/
├── alerts/
└── dashboards/
```

Then:

```text
Git
 ↓
ArgoCD
 ↓
Kubernetes
 ↓
Prometheus Operator
```

This provides:

```text
Version Control
Auditability
Rollback
Review
Consistency
```

---

# 129. Best Practice: Avoid Hardcoded IPs

Avoid:

```yaml
targets:
  - "10.244.2.15:9090"
```

for ephemeral Kubernetes workloads.

Prefer:

```text
Kubernetes discovery
```

or:

```text
ServiceMonitor / PodMonitor
```

---

# 130. Best Practice: Use Stable Labels

Use labels such as:

```text
cluster
environment
namespace
service
region
team
```

Avoid uncontrolled dynamic labels.

---

# 131. Best Practice: Filter Early

If you do not need a target:

```text
Don't scrape it.
```

Target filtering is generally preferable to discovering and scraping unnecessary targets.

---

# 132. Best Practice: Control Metric Cardinality

Do not expose or retain unnecessary high-cardinality dimensions.

Monitor:

```text
Active Series
Scrape Size
Prometheus Memory
Query Performance
```

---

# 133. Best Practice: Monitor Prometheus Itself

Prometheus should monitor itself.

Important metrics include:

```text
prometheus_tsdb_head_series
prometheus_target_scrapes_exceeded_sample_limit_total
prometheus_tsdb_head_samples_appended_total
prometheus_engine_query_duration_seconds
prometheus_rule_evaluation_failures_total
```

The exact metrics available depend on your Prometheus version.

---

# 134. Monitor Service Discovery

You should monitor whether Prometheus is successfully discovering and scraping targets.

Important concepts include:

```text
Target count
UP targets
DOWN targets
Scrape failures
Scrape duration
Discovery errors
```

---

# 135. Alert on Scrape Failures

A common metric:

```promql
up
```

can be used to identify targets that cannot currently be scraped.

For example:

```promql
up == 0
```

can identify failed targets.

In production, add appropriate labels and filtering to avoid alert noise.

---

# 136. Alert on Target Availability

Example:

```promql
up{job="node-exporter"} == 0
```

This can alert when a Node Exporter target is unavailable.

---

# 137. Service Discovery vs Scraping

Remember:

```text
Service Discovery
    ↓
Find target
```

while:

```text
Scraping
    ↓
Collect metrics
```

A service discovery problem and a scrape problem are not necessarily the same problem.

---

# 138. Important Interview Concept

If an interviewer asks:

> "Prometheus is not scraping a Kubernetes application. How do you troubleshoot?"

Answer systematically:

```text
1. Check Prometheus Targets page.
2. Determine whether target is discovered.
3. If not discovered, check ServiceMonitor/PodMonitor.
4. Verify namespace and selectors.
5. Verify Prometheus is selecting the monitor resource.
6. Verify Service and endpoint configuration.
7. Verify metrics port and path.
8. Check RBAC.
9. Check NetworkPolicy.
10. Test /metrics manually.
11. Check relabeling.
12. Check target logs/errors.
```

---

# 139. Interview Question: What Is Service Discovery?

### Answer

Service discovery allows Prometheus to dynamically discover monitoring targets instead of maintaining static IP addresses manually.

For example, in Kubernetes, Prometheus can query the Kubernetes API to discover pods, services, nodes, and endpoints.

This is important because Kubernetes workloads are dynamic and pod IP addresses can change frequently.

---

# 140. Interview Question: Why Is Service Discovery Important in Kubernetes?

### Answer

Kubernetes workloads are dynamic.

Pods can:

```text
Start
Stop
Restart
Scale
Move between nodes
Receive new IP addresses
```

Static target configuration would require continuous manual maintenance.

Kubernetes service discovery allows Prometheus to automatically follow these changes.

---

# 141. Interview Question: What Kubernetes Resources Can Prometheus Discover?

### Answer

Depending on the configured discovery role and architecture, Prometheus can discover resources such as:

```text
Nodes
Pods
Services
Endpoints
EndpointSlices
Ingresses
```

When using Prometheus Operator, monitoring resources such as:

```text
ServiceMonitor
PodMonitor
```

provide Kubernetes-native scrape configuration.

---

# 142. Interview Question: What Is ServiceMonitor?

### Answer

ServiceMonitor is a custom resource provided by Prometheus Operator.

It defines how Prometheus should discover and scrape Kubernetes Services.

For example:

```text
Service
   ↓
ServiceMonitor
   ↓
Prometheus Operator
   ↓
Prometheus
```

It allows monitoring configuration to be managed declaratively through Kubernetes.

---

# 143. Interview Question: What Is PodMonitor?

### Answer

PodMonitor is a Prometheus Operator resource used to define scraping directly against pods.

It is useful when direct pod discovery is appropriate instead of selecting a Kubernetes Service.

---

# 144. Interview Question: ServiceMonitor vs PodMonitor?

### Answer

`ServiceMonitor` selects Kubernetes Services.

```text
Service
   ↓
ServiceMonitor
```

`PodMonitor` selects Pods.

```text
Pod
   ↓
PodMonitor
```

I choose based on the desired monitoring architecture and how the application exposes its metrics.

---

# 145. Interview Question: What Is Relabeling?

### Answer

Relabeling allows Prometheus to modify, filter, or enrich discovered targets before scraping.

For example, I can:

```text
Keep production targets
Drop development targets
Create environment labels
Map Kubernetes labels
Modify target addresses
```

This is especially useful in dynamic environments.

---

# 146. Interview Question: What Is the Difference Between `relabel_configs` and `metric_relabel_configs`?

### Answer

`relabel_configs` operates on discovered targets before scraping.

```text
Discovery
 ↓
relabel_configs
 ↓
Scrape
```

`metric_relabel_configs` operates on scraped metrics before they are stored.

```text
Scrape
 ↓
metric_relabel_configs
 ↓
Storage
```

So target relabeling controls targets, while metric relabeling controls individual metrics.

---

# 147. Interview Question: How Would You Monitor Dynamic EC2 Instances?

### Answer

I would use EC2 service discovery instead of maintaining static IP addresses.

Architecture:

```text
EC2 Auto Scaling Group
       ↓
AWS API
       ↓
Prometheus EC2 SD
       ↓
Relabeling
       ↓
Node Exporter
       ↓
Prometheus
```

This allows newly created instances to be discovered automatically.

---

# 148. Interview Question: How Would You Monitor a Kubernetes Microservices Application?

### Answer

I would expose Prometheus metrics from each application and use Kubernetes-native discovery.

For example:

```text
Application
    ↓
/metrics
    ↓
Kubernetes Service
    ↓
ServiceMonitor
    ↓
Prometheus Operator
    ↓
Prometheus
    ↓
Grafana
```

For applications requiring direct pod scraping, I can use PodMonitor.

---

# 149. Interview Question: A Pod Is Running but Prometheus Cannot Scrape It. What Do You Check?

### Answer

I would check:

```text
1. Is the pod discovered?
2. Is the target UP or DOWN?
3. Is the metrics port correct?
4. Is the metrics path correct?
5. Is the application actually listening?
6. Does /metrics return data?
7. Does the Service selector match the pods?
8. Does ServiceMonitor select the Service?
9. Does Prometheus select the ServiceMonitor?
10. Is RBAC correct?
11. Is NetworkPolicy blocking traffic?
12. Did relabeling drop the target?
```

---

# 150. Interview Question: The Target Is Discovered but DOWN. What Does That Mean?

### Answer

It means service discovery is working, but Prometheus cannot successfully scrape the target.

I would investigate:

```text
Network connectivity
Port
Metrics endpoint
Application process
TLS
Authentication
NetworkPolicy
Timeout
```

This is different from a target not being discovered at all.

---

# 151. Interview Question: How Do You Reduce Unnecessary Targets?

### Answer

I use target relabeling to filter targets before scraping.

For example:

```text
Discover
   ↓
Keep production
   ↓
Drop development
   ↓
Scrape
```

I also ensure ServiceMonitor and PodMonitor selectors are scoped appropriately.

---

# 152. Interview Question: How Do You Reduce Cardinality in Kubernetes?

### Answer

I avoid blindly copying all Kubernetes labels into Prometheus.

I prefer stable labels such as:

```text
cluster
environment
namespace
service
region
team
```

I avoid uncontrolled dimensions such as:

```text
request_id
user_id
session_id
```

I also use metric relabeling when specific unnecessary metrics need to be dropped.

---

# 153. Real-World Incident: New Pods Are Not Appearing in Prometheus

Suppose:

```text
order-service
```

scaled from:

```text
3 pods → 8 pods
```

but Prometheus only shows three targets.

Troubleshooting:

```text
1. Check Prometheus Targets.
2. Check PodMonitor / ServiceMonitor.
3. Check selector.
4. Check namespace.
5. Check pod labels.
6. Check Prometheus monitor selection.
7. Check relabeling.
8. Check RBAC.
```

---

# 154. Real-World Incident: All Production Pods Suddenly Become DOWN

Possible causes:

```text
NetworkPolicy change
Prometheus configuration change
Prometheus RBAC issue
Application metrics endpoint changed
Service port changed
Cluster networking issue
Security group / network routing issue
TLS certificate issue
```

Do not immediately assume the applications themselves are broken.

First determine whether:

```text
Discovery failed
```

or:

```text
Scraping failed
```

---

# 155. Real-World Incident: ServiceMonitor Exists but Nothing Is Scraped

Check:

```text
ServiceMonitor
    ↓
Selector
    ↓
Service
    ↓
Service Endpoints
    ↓
Metrics Port
    ↓
Metrics Path
```

Then check whether Prometheus is configured to select:

```text
that ServiceMonitor
```

---

# 156. Real-World Incident: Service Has No Endpoints

Run:

```bash
kubectl get endpoints order-service -n production
```

or:

```bash
kubectl get endpointslice -n production
```

If no backend endpoints exist, investigate:

```text
Service selector
Pod labels
Pod readiness
Pod availability
```

A ServiceMonitor cannot magically create backend endpoints.

---

# 157. Real-World Incident: Metrics Endpoint Works Manually but Prometheus Cannot Scrape

Suppose:

```bash
curl http://pod-ip:9090/metrics
```

works.

But Prometheus reports:

```text
connection refused
```

Investigate:

```text
NetworkPolicy
Prometheus network path
Port
ServiceMonitor target
Target address
TLS
Authentication
```

The fact that curl works from one location does not prove that Prometheus can reach the same endpoint.

---

# 158. Real-World Incident: Prometheus Has Too Many Targets

Suppose:

```text
Expected targets = 2,000
Actual targets = 10,000
```

Investigate:

```text
ServiceMonitor selectors
PodMonitor selectors
Kubernetes discovery roles
Target relabeling
Duplicate monitoring configurations
Namespaces being monitored
```

Then reduce the discovery scope.

---

# 159. Real-World Incident: Prometheus Memory Increased After Adding Kubernetes Discovery

Possible causes:

```text
More targets
More metrics
Higher scrape frequency
High-cardinality labels
Too many Kubernetes resources
Duplicate scraping
Unnecessary exporters
```

Investigate:

```text
Target count
Active series
Scrape samples
Label cardinality
Query load
```

---

# 160. Production Service Discovery Checklist

Before enabling discovery:

```text
[ ] Identify source of truth
[ ] Define target scope
[ ] Define namespaces
[ ] Define selectors
[ ] Configure RBAC
[ ] Configure authentication
[ ] Configure TLS if needed
[ ] Configure relabeling
[ ] Control cardinality
[ ] Validate target count
[ ] Test scraping
[ ] Add alerts
```

---

# 161. Kubernetes Production Checklist

```text
[ ] Prometheus Operator installed
[ ] Prometheus resource configured
[ ] ServiceMonitor configured
[ ] PodMonitor configured where needed
[ ] Correct namespace selector
[ ] Correct label selectors
[ ] Correct metrics port
[ ] Correct metrics path
[ ] RBAC configured
[ ] NetworkPolicy verified
[ ] Targets visible in Prometheus
[ ] Targets UP
[ ] Metrics visible in PromQL
```

---

# 162. AWS Production Checklist

```text
[ ] EC2 service discovery configured
[ ] Correct AWS region
[ ] IAM permissions configured
[ ] Instance tags standardized
[ ] Node Exporter installed
[ ] Target filtering configured
[ ] Environment labels configured
[ ] Production instances verified
[ ] Auto Scaling behavior tested
```

---

# 163. Service Discovery Mental Model

Remember:

```text
                    SOURCE OF TRUTH
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
         Kubernetes       AWS        Consul
             │            │            │
             └────────────┼────────────┘
                          ↓
                  SERVICE DISCOVERY
                          ↓
                       TARGETS
                          ↓
                     RELABELING
                          ↓
                       SCRAPING
                          ↓
                       METRICS
                          ↓
                     PROMETHEUS
                          ↓
                       GRAFANA
```

---

# 164. Most Important Concepts

For interviews and production work, remember these concepts:

```text
Service Discovery
Kubernetes SD
EC2 SD
Consul SD
DNS SD
File SD
Static Config
Relabeling
Target Relabeling
Metric Relabeling
ServiceMonitor
PodMonitor
RBAC
Selectors
Target States
Scrape Failures
Cardinality
```

---

# 165. Final Interview Summary

If asked:

> How does Prometheus discover targets in a production Kubernetes environment?

A strong answer is:

```text
"In a production Kubernetes environment, I prefer dynamic service discovery rather than static IP configuration.

Prometheus can use the Kubernetes API to discover resources such as pods, services, nodes, and endpoints. In environments using Prometheus Operator, I commonly use ServiceMonitor and PodMonitor resources to declaratively define what should be scraped.

The discovery metadata is processed through relabeling to filter targets and create useful labels such as namespace, service, environment, and cluster.

Prometheus then scrapes the selected targets and stores the metrics in its TSDB.

For troubleshooting, I first check whether the target is discovered in the Prometheus Targets page. If it is discovered but DOWN, I investigate the endpoint, port, metrics path, network connectivity, RBAC, NetworkPolicy, TLS, and application metrics endpoint.

This approach works well with dynamic Kubernetes environments because pods can scale, restart, and receive new IP addresses without requiring manual Prometheus configuration changes."
```

---

# 166. Final Architecture

```text
                           GIT
                            │
                            ↓
                  Monitoring Manifests
                            │
                            ↓
                          ArgoCD
                            │
                            ↓
                       Kubernetes
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
           Pods          Services        Nodes
             │              │              │
             └──────────────┼──────────────┘
                            ↓
                    Kubernetes API
                            │
                            ↓
                   Prometheus Operator
                            │
              ┌─────────────┴─────────────┐
              ↓                           ↓
        ServiceMonitor                PodMonitor
              │                           │
              └─────────────┬─────────────┘
                            ↓
                     Prometheus
                            │
                      Service Discovery
                            │
                       Relabeling
                            │
                         Scrape
                            │
                         Metrics
                            ↓
                         TSDB
                            │
                            ↓
                         Grafana
```

The key production principle is:

```text
Do not manually manage dynamic infrastructure.

Let the infrastructure source of truth
provide the target information.

Then use service discovery + relabeling
to control what Prometheus actually monitors.
```
