# Grafana Installation

## 1. Overview

Grafana can be installed in several ways depending on the environment:

```text
Development
    ↓
Local package / Docker

Virtual Machine
    ↓
Linux package installation

Kubernetes
    ↓
Helm

Production EKS
    ↓
Helm + GitOps / ArgoCD
```

For the DevOps environment in this chapter, the most important installation method is **Kubernetes/Helm**, because the target environment is AWS EKS.

The installation process should be understood from basic installation through production deployment.

---

# 2. Installation Options

Common Grafana installation approaches include:

```text
1. Linux package
2. Docker
3. Kubernetes manifests
4. Helm
5. Grafana Operator
6. Managed Grafana services
```

For our environment:

```text
Primary → Helm on EKS
Production management → GitOps with ArgoCD
```

---

# 3. Basic Architecture

After installation:

```text
                    Engineer
                       │
                       ↓
                    Grafana
                       │
                       ↓
                  Prometheus
                       │
                       ↓
                    Metrics
```

Production:

```text
                         Users
                           │
                           ↓
                       ALB / Ingress
                           │
                           ↓
                    Grafana Service
                           │
                  ┌────────┴────────┐
                  ↓                 ↓
              Grafana A         Grafana B
                  │                 │
                  └────────┬────────┘
                           ↓
                       Data Sources
                    ┌──────┼──────┐
                    ↓      ↓      ↓
               Prometheus Logs   Jaeger
```

---

# 4. Prerequisites

Before installing Grafana, verify:

```text
Operating system
Kubernetes
Helm
kubectl
Network connectivity
DNS
Storage
Authentication requirements
Data sources
```

For EKS:

```bash
kubectl version --client
```

Check cluster access:

```bash
kubectl get nodes
```

Expected:

```text
NAME                          STATUS   ROLES
ip-10-0-1-10                  Ready    <none>
ip-10-0-2-20                  Ready    <none>
ip-10-0-3-30                  Ready    <none>
```

---

# 5. Verify Helm

Check Helm:

```bash
helm version
```

Example:

```text
version.BuildInfo{
  Version:"v3.x.x"
}
```

The exact version should be pinned and standardized for production.

---

# 6. Create Monitoring Namespace

Create a dedicated namespace:

```bash
kubectl create namespace monitoring
```

Verify:

```bash
kubectl get namespaces
```

You should see:

```text
monitoring
```

A dedicated namespace makes it easier to manage:

```text
Grafana
Prometheus
Alertmanager
Exporters
Monitoring resources
```

---

# 7. Add Grafana Helm Repository

Add the official Grafana Helm repository:

```bash
helm repo add grafana https://grafana.github.io/helm-charts
```

Update repositories:

```bash
helm repo update
```

Verify:

```bash
helm search repo grafana
```

You should see available Grafana charts.

---

# 8. Search for Grafana Chart

Run:

```bash
helm search repo grafana/grafana
```

Example output:

```text
NAME              CHART VERSION   APP VERSION
grafana/grafana   <version>       <version>
```

Do not blindly install the latest version in production.

---

# 9. Version Pinning

For production:

```bash
helm search repo grafana/grafana --versions
```

Select an approved chart version.

Then install using:

```bash
helm install grafana grafana/grafana \
  --version <approved-version> \
  -n monitoring
```

The actual approved version should come from your organization's compatibility testing.

---

# 10. Development Installation

For a simple development environment:

```bash
helm install grafana grafana/grafana \
  -n monitoring
```

Check:

```bash
kubectl get pods -n monitoring
```

Expected:

```text
NAME                       READY   STATUS
grafana-xxxxxxxxxx         1/1     Running
```

---

# 11. Check Grafana Service

Run:

```bash
kubectl get svc -n monitoring
```

Example:

```text
NAME      TYPE        CLUSTER-IP      PORT
grafana   ClusterIP   10.100.10.20    80
```

The exact Service name, IP and port depend on the Helm chart version and values.

---

# 12. Check Grafana Deployment

Run:

```bash
kubectl get deployment -n monitoring
```

Example:

```text
NAME      READY   UP-TO-DATE   AVAILABLE
grafana   1/1     1            1
```

---

# 13. Check Grafana Pod

```bash
kubectl get pods -n monitoring -l app.kubernetes.io/name=grafana
```

Then:

```bash
kubectl describe pod <grafana-pod> -n monitoring
```

Check:

```text
Events
Container
Image
Ports
Volumes
Environment
Probes
```

---

# 14. View Grafana Logs

Use:

```bash
kubectl logs deployment/grafana -n monitoring
```

For a specific pod:

```bash
kubectl logs <grafana-pod> -n monitoring
```

Follow logs:

```bash
kubectl logs -f deployment/grafana -n monitoring
```

Look for:

```text
Startup errors
Database errors
Plugin errors
Configuration errors
Authentication errors
Datasource errors
```

---

# 15. Access Grafana Using Port Forwarding

For development/testing:

```bash
kubectl port-forward svc/grafana 3000:80 -n monitoring
```

Then access:

```text
http://localhost:3000
```

Port forwarding is useful for testing.

It should not be treated as the production access architecture.

---

# 16. Retrieve Initial Admin Password

Depending on the Helm chart configuration, the initial admin password may be stored in a Kubernetes Secret.

First inspect secrets:

```bash
kubectl get secrets -n monitoring
```

Look for the Grafana-related Secret.

Then:

```bash
kubectl get secret <grafana-secret> \
  -n monitoring \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

The exact Secret name and key depend on the chart version and configuration.

---

# 17. Login

Open:

```text
http://localhost:3000
```

Use the configured credentials.

For a production deployment, prefer centralized authentication such as:

```text
OIDC
SAML
Enterprise SSO
```

instead of relying on a shared local admin account.

---

# 18. Verify Grafana

After login, verify:

```text
Dashboards
Explore
Connections / Data Sources
Alerting
Administration
```

The UI names can vary slightly by Grafana version.

---

# 19. Production Installation With Helm

For production, do not rely on:

```bash
helm install ...
```

with no values file.

Instead:

```text
Git
 │
 └── values.yaml
        │
        ↓
      Helm
        │
        ↓
      Grafana
```

Example:

```text
grafana/
├── values.yaml
├── dashboards/
├── provisioning/
└── argocd/
```

---

# 20. Production Values File

Create:

```text
values.yaml
```

Example:

```yaml
replicas: 2

resources:
  requests:
    cpu: 250m
    memory: 512Mi

  limits:
    cpu: 1
    memory: 1Gi
```

These values are examples only.

Production resources should be based on actual:

```text
Dashboard count
Concurrent users
Query volume
Plugins
Refresh frequency
```

---

# 21. Configure Grafana Service

Example:

```yaml
service:
  type: ClusterIP
  port: 80
```

Then expose Grafana through:

```text
ALB / Ingress
```

rather than exposing the Grafana Service directly unless there is a specific requirement.

---

# 22. Grafana Ingress

A typical production architecture:

```text
User
 ↓
Internal ALB
 ↓
Ingress
 ↓
Grafana Service
 ↓
Grafana Pods
```

Example configuration depends on the Kubernetes ingress controller and AWS load-balancer architecture.

---

# 23. AWS ALB Architecture

For an EKS environment using AWS Load Balancer Controller:

```text
                    User
                     │
                     ↓
              Internal / External
                    ALB
                     │
                  Ingress
                     │
              Grafana Service
                     │
            ┌────────┴────────┐
            ↓                 ↓
        Grafana A         Grafana B
```

For internal monitoring, an internal ALB is often appropriate.

---

# 24. Example Ingress

A simplified example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: grafana
  namespace: monitoring

  annotations:
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/target-type: ip

spec:
  ingressClassName: alb

  rules:
    - host: grafana.example.internal

      http:
        paths:
          - path: /
            pathType: Prefix

            backend:
              service:
                name: grafana
                port:
                  number: 80
```

The exact annotations depend on your AWS Load Balancer Controller configuration.

---

# 25. DNS

Production users should access Grafana using a DNS name:

```text
grafana.example.internal
```

rather than:

```text
http://10.x.x.x
```

Architecture:

```text
Engineer
   ↓
grafana.example.internal
   ↓
Route 53 / DNS
   ↓
ALB
   ↓
Grafana
```

---

# 26. HTTPS

Production Grafana should use HTTPS.

Architecture:

```text
Engineer
   ↓
HTTPS
   ↓
ALB
   ↓
Grafana
```

TLS certificates can be managed using AWS Certificate Manager when using an AWS ALB architecture.

---

# 27. Configure Grafana Root URL

When Grafana is accessed through a domain:

```text
https://grafana.example.internal
```

Grafana should know its external URL.

Example:

```yaml
grafana.ini:
  server:
    root_url: https://grafana.example.internal
```

The exact Helm configuration syntax depends on the chart version.

---

# 28. Why Root URL Matters

The root URL is important for:

```text
Login redirects
OAuth
OIDC
Links
Embedded dashboards
Alert URLs
Browser navigation
```

A wrong root URL can cause authentication and redirect problems.

---

# 29. Grafana Database

For simple development:

```text
Grafana
   ↓
SQLite
```

For production HA:

```text
Grafana A ──┐
            ├──→ PostgreSQL
Grafana B ──┘
```

This allows multiple Grafana replicas to share persistent application state.

---

# 30. Configure PostgreSQL

Conceptually:

```yaml
database:
  type: postgres
  host: postgres.example.internal:5432
  name: grafana
  user: grafana
```

Do not place the actual password directly into Git.

Use secret management.

---

# 31. Database Secret

Use Kubernetes Secrets or an external secret mechanism.

Conceptually:

```text
AWS Secrets Manager
        ↓
External Secrets
        ↓
Kubernetes Secret
        ↓
Grafana
        ↓
PostgreSQL
```

This keeps database credentials outside the Git repository.

---

# 32. Grafana HA Installation

Example production architecture:

```text
                       Internal ALB
                            │
                   ┌────────┴────────┐
                   ↓                 ↓
               Grafana A         Grafana B
                   │                 │
                   └────────┬────────┘
                            ↓
                       PostgreSQL
```

Helm values may include:

```yaml
replicas: 2
```

Again, actual production values depend on capacity and availability requirements.

---

# 33. Pod Anti-Affinity

Ensure Grafana replicas are not unnecessarily placed on the same node.

Example concept:

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchLabels:
              app.kubernetes.io/name: grafana
```

The labels must match the deployed chart.

---

# 34. Topology Spread

You can also distribute Grafana across availability zones.

Conceptually:

```text
AZ-A
 └── Grafana A

AZ-B
 └── Grafana B
```

This protects against a failure affecting one zone.

---

# 35. PodDisruptionBudget

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget

metadata:
  name: grafana
  namespace: monitoring

spec:
  minAvailable: 1

  selector:
    matchLabels:
      app.kubernetes.io/name: grafana
```

This helps maintain availability during voluntary disruptions.

---

# 36. Resource Requests and Limits

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 512Mi

  limits:
    cpu: 1
    memory: 1Gi
```

Monitor actual usage and adjust.

Do not copy resource values blindly into production.

---

# 37. Why Resource Requests Matter

Kubernetes uses requests during scheduling.

Example:

```text
Grafana request:
CPU    250m
Memory 512Mi
```

The scheduler looks for a node with sufficient allocatable resources.

Without appropriate requests, Grafana may be scheduled poorly or become difficult to operate reliably.

---

# 38. Why Resource Limits Matter

Limits protect the cluster from uncontrolled resource consumption.

However, overly restrictive limits can cause:

```text
OOMKilled
CPU throttling
Slow dashboards
```

Therefore resource limits should be based on measurements.

---

# 39. Grafana Health Probes

Configure appropriate:

```text
Startup probe
Readiness probe
Liveness probe
```

Conceptually:

```text
Startup
   ↓
Grafana initializes
   ↓
Readiness
   ↓
Traffic allowed
   ↓
Liveness
   ↓
Continuous health
```

---

# 40. Grafana Readiness

When Grafana is not ready:

```text
Grafana Pod
     ↓
Readiness = Failed
     ↓
Service removes endpoint
     ↓
Traffic stops
```

This prevents requests from being sent to an unhealthy pod.

---

# 41. Grafana Liveness

If Grafana becomes unhealthy:

```text
Grafana Pod
     ↓
Liveness fails
     ↓
Kubernetes restarts container
```

Use probes carefully to avoid unnecessary restart loops.

---

# 42. Install Grafana With a Values File

Example:

```bash
helm install grafana grafana/grafana \
  --version <approved-version> \
  -n monitoring \
  -f values.yaml
```

For upgrades:

```bash
helm upgrade grafana grafana/grafana \
  --version <approved-version> \
  -n monitoring \
  -f values.yaml
```

---

# 43. Validate Helm Before Installing

Use:

```bash
helm lint ./chart
```

If using the remote chart:

```bash
helm template grafana grafana/grafana \
  --version <approved-version> \
  -n monitoring \
  -f values.yaml
```

Review the generated Kubernetes resources.

---

# 44. Dry-Run

You can use:

```bash
helm upgrade --install grafana grafana/grafana \
  --version <approved-version> \
  -n monitoring \
  -f values.yaml \
  --dry-run
```

This helps identify configuration problems before changing the cluster.

---

# 45. Check Helm Release

After installation:

```bash
helm list -n monitoring
```

Example:

```text
NAME      NAMESPACE    STATUS
grafana   monitoring   deployed
```

---

# 46. Check Helm Release Details

```bash
helm status grafana -n monitoring
```

This can show:

```text
Revision
Resources
Notes
Status
```

---

# 47. Check All Monitoring Resources

```bash
kubectl get all -n monitoring
```

Then inspect:

```bash
kubectl get pods -n monitoring
kubectl get svc -n monitoring
kubectl get ingress -n monitoring
```

---

# 48. Verify Grafana Endpoint

If using port-forward:

```bash
kubectl port-forward svc/grafana 3000:80 -n monitoring
```

Then:

```text
http://localhost:3000
```

For production:

```text
https://grafana.example.internal
```

---

# 49. Configure Prometheus Data Source

Once Grafana is installed, connect it to Prometheus.

Architecture:

```text
Grafana
   ↓
Prometheus Data Source
   ↓
Prometheus
```

Inside Kubernetes, Grafana should use the Prometheus Kubernetes Service DNS name.

---

# 50. Find Prometheus Service

Run:

```bash
kubectl get svc -n monitoring
```

Example:

```text
NAME                  TYPE        PORT
prometheus-operated   ClusterIP   9090
grafana               ClusterIP   80
```

The actual Service name depends on your Prometheus deployment.

---

# 51. Prometheus URL

A typical internal Kubernetes URL may look like:

```text
http://prometheus-operated.monitoring.svc:9090
```

Do not assume the Service name.

Verify it using:

```bash
kubectl get svc -n monitoring
```

---

# 52. Add Prometheus Through Grafana UI

Navigate to:

```text
Connections
   ↓
Data Sources
   ↓
Add data source
   ↓
Prometheus
```

Set:

```text
URL:
http://<prometheus-service>:9090
```

Then:

```text
Save & Test
```

---

# 53. Provision Prometheus Data Source

For production, configure the data source as code.

Example:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus-operated.monitoring.svc:9090
    isDefault: true
```

The actual URL should match the Prometheus Service deployed in your environment.

---

# 54. Why Provision Data Sources?

Manual:

```text
Engineer
 ↓
Grafana UI
 ↓
Create Data Source
```

GitOps:

```text
Git
 ↓
Data Source Definition
 ↓
ArgoCD
 ↓
Grafana
```

GitOps provides:

```text
Version control
Auditability
Repeatability
Rollback
```

---

# 55. Verify Prometheus Data Source

In Grafana:

```text
Connections
 ↓
Data Sources
 ↓
Prometheus
 ↓
Save & Test
```

You should receive a successful connection response.

---

# 56. Test Prometheus Query

Open:

```text
Explore
```

Select:

```text
Prometheus
```

Run:

```promql
up
```

Expected result should include active scrape targets.

Example:

```text
up{job="grafana"} 1
up{job="node-exporter"} 1
up{job="kube-state-metrics"} 1
```

The exact labels depend on your Prometheus configuration.

---

# 57. Test CPU Query

Example:

```promql
100 *
(
  1 -
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

This can show approximate CPU utilization by instance.

---

# 58. Test Memory Query

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

This can show approximate memory utilization.

The available metrics depend on Node Exporter configuration.

---

# 59. Create First Dashboard

Go to:

```text
Dashboards
 ↓
New Dashboard
 ↓
Add Visualization
```

Select:

```text
Prometheus
```

Then create a query.

Example:

```promql
up
```

Choose:

```text
Time series
```

or:

```text
Stat
```

---

# 60. First Production-Useful Panel

Create:

```text
Panel:
Node CPU Utilization
```

Query:

```promql
100 *
(
  1 -
  avg by(instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

Set:

```text
Unit → Percent
```

Then save the dashboard.

---

# 61. Create Kubernetes Dashboard

A useful starting dashboard:

```text
Kubernetes Cluster
│
├── Node CPU
├── Node Memory
├── Pod Count
├── Pod Restarts
├── Pending Pods
├── Deployment Availability
└── PVC Usage
```

---

# 62. Dashboard Variables

Create variables:

```text
cluster
namespace
pod
container
```

For example:

```text
Cluster:
[ production ▼ ]

Namespace:
[ payments ▼ ]

Pod:
[ payment-api-xxxx ▼ ]
```

This allows one dashboard to serve multiple workloads.

---

# 63. Dashboard Provisioning

Instead of manually creating dashboards:

```text
Grafana UI
```

store dashboard definitions:

```text
Git
└── dashboards/
    ├── cluster.json
    ├── node.json
    └── payment.json
```

Then provision them automatically.

---

# 64. Grafana Dashboard Provider

A provider tells Grafana where dashboard files are located.

Conceptually:

```yaml
apiVersion: 1

providers:
  - name: default
    type: file
    options:
      path: /var/lib/grafana/dashboards
```

The exact Helm configuration depends on the chart version.

---

# 65. Grafana Plugins

Plugins can add:

```text
Data sources
Panels
Applications
```

For production:

```text
Only install required plugins
Pin versions where possible
Test upgrades
Track plugins in Git
```

Avoid installing random plugins directly in production.

---

# 66. Install Plugin

The exact Helm configuration depends on the chart version.

Conceptually:

```yaml
plugins:
  - <plugin-name>
```

Then deploy:

```bash
helm upgrade grafana grafana/grafana \
  -n monitoring \
  -f values.yaml
```

Verify:

```bash
kubectl logs deployment/grafana -n monitoring
```

---

# 67. Grafana Authentication

For development:

```text
Local username/password
```

For production:

```text
OIDC
SAML
LDAP
Enterprise SSO
```

A typical architecture:

```text
Engineer
   ↓
Grafana
   ↓
OIDC
   ↓
Identity Provider
```

---

# 68. OIDC Architecture

Conceptually:

```text
Engineer
   ↓
Grafana
   ↓
Redirect
   ↓
Identity Provider
   ↓
Authentication
   ↓
Authorization claims
   ↓
Grafana
```

Grafana can map identity-provider information to teams or roles depending on the configuration.

---

# 69. Secure Authentication Secrets

OIDC credentials should not be stored in plain Git.

Use:

```text
AWS Secrets Manager
        ↓
External Secrets
        ↓
Kubernetes Secret
        ↓
Grafana
```

---

# 70. Production Domain

A production Grafana deployment should have a stable URL.

Example:

```text
https://grafana.company.internal
```

Architecture:

```text
Route 53
   ↓
ALB
   ↓
Grafana Ingress
   ↓
Grafana Service
   ↓
Grafana Pods
```

---

# 71. Production Installation Through ArgoCD

Instead of manually installing:

```bash
helm install
```

production can use:

```text
Git
 │
 ├── Grafana Helm values
 ├── ArgoCD Application
 ├── Data Sources
 └── Dashboards
       │
       ↓
     ArgoCD
       │
       ↓
      EKS
```

---

# 72. ArgoCD Application

Conceptually:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: grafana

spec:
  destination:
    namespace: monitoring
    server: https://kubernetes.default.svc

  source:
    repoURL: <git-repository>
    path: grafana
```

The exact ArgoCD configuration depends on your repository structure.

---

# 73. Git Repository Structure

A practical structure:

```text
07-Grafana/
│
├── helm/
│   └── values.yaml
│
├── dashboards/
│   ├── kubernetes/
│   ├── applications/
│   └── infrastructure/
│
├── provisioning/
│   ├── datasources/
│   ├── dashboards/
│   └── alerting/
│
└── argocd/
    └── application.yaml
```

---

# 74. Production Deployment Flow

```text
Developer
   ↓
Modify Grafana configuration
   ↓
Git Pull Request
   ↓
Code Review
   ↓
Merge
   ↓
ArgoCD detects change
   ↓
Helm deployment
   ↓
EKS
   ↓
Grafana
```

---

# 75. Installation Validation

After deployment:

```bash
kubectl get pods -n monitoring
```

Check:

```text
Grafana pods Running
```

Then:

```bash
kubectl get svc -n monitoring
```

Then:

```bash
kubectl get ingress -n monitoring
```

Then test:

```text
Grafana URL
```

---

# 76. Validate Grafana Logs

```bash
kubectl logs deployment/grafana -n monitoring
```

Look for:

```text
HTTP server started
Database connected
Plugins loaded
Provisioning completed
```

And investigate:

```text
Database connection failures
Datasource failures
Plugin failures
Authentication failures
```

---

# 77. Validate Data Source

Open:

```text
Grafana
 ↓
Connections
 ↓
Data Sources
 ↓
Prometheus
 ↓
Save & Test
```

Then run:

```promql
up
```

If results are returned, the basic integration is working.

---

# 78. Validate Dashboard

Open:

```text
Dashboards
```

Verify:

```text
CPU
Memory
Pods
Nodes
Requests
Errors
Latency
```

depending on what metrics are available.

---

# 79. Validate Grafana Through Kubernetes

Check endpoints:

```bash
kubectl get endpoints -n monitoring
```

Or:

```bash
kubectl get endpointslices -n monitoring
```

Verify Grafana Service has healthy endpoints.

---

# 80. Validate Ingress

```bash
kubectl describe ingress grafana -n monitoring
```

Check:

```text
Address
Rules
Backend
Events
Annotations
```

If the ALB does not become healthy, inspect the AWS Load Balancer Controller logs and Kubernetes events.

---

# 81. Common Installation Problem: Pod Pending

Check:

```bash
kubectl describe pod <grafana-pod> -n monitoring
```

Look at:

```text
Events
```

Possible causes:

```text
Insufficient CPU
Insufficient memory
Node affinity
Taints
Topology constraints
PVC issues
```

---

# 82. Common Problem: CrashLoopBackOff

Run:

```bash
kubectl logs <grafana-pod> -n monitoring
```

Previous container:

```bash
kubectl logs <grafana-pod> \
  -n monitoring \
  --previous
```

Then:

```bash
kubectl describe pod <grafana-pod> -n monitoring
```

Check:

```text
Database
Configuration
Permissions
Plugins
Environment variables
Secrets
```

---

# 83. Common Problem: Grafana Shows Login But No Data

Check:

```text
1. Data source
2. Prometheus Service
3. Network connectivity
4. Prometheus availability
5. PromQL query
6. Time range
7. Dashboard variables
```

Start with:

```promql
up
```

If `up` returns data, the basic Prometheus connection is likely working.

---

# 84. Common Problem: Data Source Connection Failed

Check Prometheus Service:

```bash
kubectl get svc -n monitoring
```

Then inspect:

```bash
kubectl get endpoints -n monitoring
```

Check Prometheus:

```bash
kubectl get pods -n monitoring
```

Then test DNS/connectivity from Grafana if needed.

---

# 85. Test DNS From Grafana Pod

If the image contains suitable tools:

```bash
kubectl exec -it <grafana-pod> -n monitoring -- \
  nslookup <prometheus-service>
```

If `nslookup` is not available, use an appropriate diagnostic pod instead.

---

# 86. Test HTTP Connectivity

From a suitable troubleshooting container:

```bash
curl http://<prometheus-service>:9090/-/healthy
```

Expected:

```text
Prometheus is Healthy.
```

The exact health endpoint behavior depends on the Prometheus version.

---

# 87. Common Problem: Dashboard Empty

Check:

```text
Time range
Data source
Variables
Metric name
Label filters
Prometheus targets
```

For example:

```promql
up
```

may work while:

```promql
some_custom_metric
```

returns nothing because the application is not exposing that metric.

---

# 88. Common Problem: Grafana Slow

Check:

```text
Dashboard panel count
Query complexity
PromQL
Refresh interval
Time range
Prometheus query performance
Grafana resources
Database performance
```

---

# 89. Common Problem: Grafana Restarts

Check:

```bash
kubectl get pods -n monitoring
```

Then:

```bash
kubectl describe pod <grafana-pod> -n monitoring
```

Look for:

```text
OOMKilled
Probe failures
Container errors
Node pressure
```

---

# 90. OOMKilled

If:

```text
Reason: OOMKilled
```

check:

```text
Grafana memory usage
Dashboard complexity
Concurrent users
Query volume
Plugins
Container memory limit
```

Then adjust resources based on measured usage.

---

# 91. Common Problem: Ingress Not Working

Check:

```bash
kubectl get ingress -n monitoring
```

Then:

```bash
kubectl describe ingress grafana -n monitoring
```

Check:

```text
IngressClass
Annotations
Service
Target type
Health checks
Security groups
Subnets
Certificate
DNS
```

---

# 92. Common Problem: ALB Unhealthy

Check:

```text
Grafana readiness
Service endpoints
Target group
Security groups
Health check path
Health check port
```

Architecture:

```text
ALB
 ↓
Target
 ↓
Grafana Service
 ↓
Grafana Pod
```

Every layer must be healthy.

---

# 93. Common Problem: HTTPS Redirect Loop

Check:

```text
ALB TLS configuration
Grafana root_url
Proxy configuration
X-Forwarded-Proto
Ingress annotations
```

Grafana must correctly understand the external protocol and URL.

---

# 94. Production Installation Checklist

```text
[ ] Kubernetes cluster access
[ ] Helm installed
[ ] Monitoring namespace
[ ] Grafana chart repository
[ ] Approved chart version
[ ] Version-pinned deployment
[ ] Production values file
[ ] Resource requests
[ ] Resource limits
[ ] Health probes
[ ] HA replicas if required
[ ] Pod distribution
[ ] PDB
[ ] External database if HA required
[ ] Secrets management
[ ] Ingress
[ ] ALB
[ ] DNS
[ ] TLS
[ ] Authentication
[ ] Authorization
[ ] Prometheus data source
[ ] Dashboard provisioning
[ ] GitOps
[ ] ArgoCD
[ ] Monitoring
[ ] Backup
[ ] Disaster recovery
```

---

# 95. Recommended Installation Sequence

For a real EKS environment:

```text
Step 1
Create monitoring namespace
        ↓
Step 2
Install Grafana through Helm
        ↓
Step 3
Configure resources
        ↓
Step 4
Configure authentication
        ↓
Step 5
Configure PostgreSQL if required
        ↓
Step 6
Configure Ingress / ALB
        ↓
Step 7
Configure DNS / TLS
        ↓
Step 8
Configure Prometheus data source
        ↓
Step 9
Provision dashboards
        ↓
Step 10
Configure alerts
        ↓
Step 11
Move configuration into Git
        ↓
Step 12
Deploy through ArgoCD
        ↓
Step 13
Validate
        ↓
Step 14
Test failure scenarios
```

---

# 96. Production Architecture After Installation

The resulting architecture:

```text
                         Engineers
                             │
                             ↓
                    SSO / VPN / IAM
                             │
                             ↓
                        Route 53
                             │
                             ↓
                    Internal AWS ALB
                             │
                             ↓
                      Grafana Ingress
                             │
                 ┌───────────┴───────────┐
                 ↓                       ↓
             Grafana A               Grafana B
                 │                       │
                 └───────────┬───────────┘
                             ↓
                        PostgreSQL
                             │
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
   Prometheus          Elasticsearch             Jaeger
     Metrics               Logs                 Traces
```

---

# 97. Interview Answer: How Do You Install Grafana?

```text
"I normally deploy Grafana on Kubernetes using Helm.

First I create a dedicated monitoring namespace and add the
Grafana Helm repository.

For production I use a version-pinned chart and maintain the
configuration in a values file stored in Git.

I configure resources, authentication, ingress, TLS, data sources,
dashboards and, when HA is required, multiple replicas with an
external database.

In our GitOps model, ArgoCD deploys the Helm release to EKS.

After deployment I validate the pods, service, ingress, data source,
Prometheus connectivity, dashboards and Grafana health."
```

---

# 98. Interview Answer: How Do You Deploy Grafana in EKS?

```text
"I deploy Grafana using Helm into a dedicated monitoring namespace.

For production I expose it through an ALB or internal ALB depending
on the access requirement.

I configure HTTPS, DNS, authentication and RBAC.

If high availability is required, I run multiple Grafana replicas
across nodes or availability zones and use an external PostgreSQL
database for shared Grafana state.

The configuration, dashboards and data sources are managed through
GitOps with ArgoCD."
```

---

# 99. Interview Answer: How Do You Connect Grafana to Prometheus?

```text
"First I identify the Prometheus Kubernetes Service.

Then I configure Prometheus as a Grafana data source using the
internal Kubernetes DNS name.

For example, the URL could be:

http://<prometheus-service>:9090

I prefer provisioning the data source through configuration rather
than manually creating it in the UI.

After configuration I use Save & Test and then run a simple PromQL
query such as 'up' in Grafana Explore to verify connectivity."
```

---

# 100. Interview Answer: How Do You Install Grafana in Production?

```text
"In production I would avoid a manual one-off Helm installation.

I would keep the Helm chart version and values in Git and deploy
them through ArgoCD.

I would configure authentication, TLS, ingress, resource requests
and limits, health probes, data sources and dashboards.

For HA I would run multiple replicas, distribute them across failure
domains and use a resilient external database.

Finally, I would validate Grafana, Prometheus connectivity,
dashboards, alerts and failure recovery."
```

---

# 101. Final Installation Mental Model

Remember the complete installation flow:

```text
                    GIT
                     │
                     ↓
              Helm Values
                     │
                     ↓
                   ArgoCD
                     │
                     ↓
                    EKS
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
      Grafana A             Grafana B
          │                     │
          └──────────┬──────────┘
                     ↓
                PostgreSQL
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
   Prometheus   Elasticsearch    Jaeger
```

The most important production principle is:

```text
Install
  ↓
Configure
  ↓
Secure
  ↓
Integrate
  ↓
Version-control
  ↓
Deploy through GitOps
  ↓
Monitor
  ↓
Test recovery
```

A Grafana installation is not production-ready simply because the Grafana pod is `Running`.

A production-ready deployment must also have the required **security, data sources, availability, configuration management, ingress, authentication, monitoring, backup, and recovery strategy**.
