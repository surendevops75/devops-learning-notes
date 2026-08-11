# Prometheus Installation

Prometheus can be installed in several ways depending on the environment.

Common installation approaches are:

```
1. Linux Binary Installation

2. Docker

3. Docker Compose

4. Kubernetes

5. Helm
```

For DevOps and production environments, the most important approaches to understand are:

```
Linux / systemd
Docker
Kubernetes / Helm
```

A typical production Kubernetes architecture is:

```
EKS
  ↓
kube-prometheus-stack
  ↓
Prometheus
  ↓
Persistent Storage
  ↓
Grafana
  ↓
Alertmanager
```

---

# 1. Installation Architecture

A basic Linux installation looks like:

```
Linux Server
    ↓
Prometheus Binary
    ↓
prometheus.yml
    ↓
Prometheus Service
    ↓
Port 9090
    ↓
Prometheus Web UI
```

---

# 2. Prerequisites

Before installing Prometheus, verify:

```
Linux Server

CPU

Memory

Disk

Network Connectivity

Required Port

Dedicated User

Configuration Directory
```

For a lab environment, a small Linux VM is enough.

For production, sizing should be based on:

```
Number of Targets

Active Series

Scrape Interval

Retention

Query Load
```

---

# 3. Recommended Linux Directory Structure

A common layout is:

```
/opt/prometheus/
/etc/prometheus/
/var/lib/prometheus/
```

For example:

```
/opt/prometheus/
    ├── prometheus
    └── promtool

/etc/prometheus/
    └── prometheus.yml

/var/lib/prometheus/
    └── TSDB data
```

The exact layout can vary by distribution and installation method.

---

# 4. Create Prometheus User

Prometheus should not normally run as root.

Create a dedicated system user:

```bash
sudo useradd --no-create-home --shell /bin/false prometheus
```

Verify:

```bash
id prometheus
```

Expected output will show the Prometheus user.

---

# 5. Create Directories

Create the application and configuration directories:

```bash
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
sudo mkdir -p /opt/prometheus
```

---

# 6. Set Ownership

Assign ownership:

```bash
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
sudo chown prometheus:prometheus /opt/prometheus
```

Verify:

```bash
ls -ld /etc/prometheus
ls -ld /var/lib/prometheus
ls -ld /opt/prometheus
```

---

# 7. Download Prometheus

Prometheus releases are distributed as compressed archives.

The production process should be:

```
Identify Approved Version
      ↓
Download Release
      ↓
Verify Archive
      ↓
Extract
      ↓
Install Binary
      ↓
Configure
      ↓
Start Service
      ↓
Verify
```

Use the official Prometheus release source for the version approved by your organization.

---

# 8. Extract Prometheus

After downloading the archive:

```bash
tar -xvf prometheus-<version>.linux-amd64.tar.gz
```

Example directory:

```text
prometheus-<version>.linux-amd64/
```

Inside the directory you normally find:

```
prometheus

promtool

consoles/

console_libraries/

prometheus.yml
```

---

# 9. Install Prometheus Binary

Copy the binaries:

```bash
sudo cp prometheus-<version>.linux-amd64/prometheus /opt/prometheus/
sudo cp prometheus-<version>.linux-amd64/promtool /opt/prometheus/
```

Set ownership:

```bash
sudo chown prometheus:prometheus /opt/prometheus/prometheus
sudo chown prometheus:prometheus /opt/prometheus/promtool
```

---

# 10. Install Console Files

Copy the console directories:

```bash
sudo cp -r prometheus-<version>.linux-amd64/consoles /etc/prometheus/
sudo cp -r prometheus-<version>.linux-amd64/console_libraries /etc/prometheus/
```

Set ownership:

```bash
sudo chown -R prometheus:prometheus /etc/prometheus/
```

---

# 11. Create Prometheus Configuration

Create:

```text
/etc/prometheus/prometheus.yml
```

Basic configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
```

This configuration tells Prometheus:

```
Scrape every 15 seconds

Scrape Prometheus itself
```

---

# 12. Understand the Basic Configuration

The configuration:

```yaml
global:
  scrape_interval: 15s
```

defines the default scrape interval.

Then:

```yaml
scrape_configs:
```

defines the targets Prometheus should scrape.

And:

```yaml
job_name: "prometheus"
```

creates a monitoring job called:

```
prometheus
```

---

# 13. Validate Configuration

Prometheus provides:

```
promtool
```

Use it to validate the configuration:

```bash
/opt/prometheus/promtool check config /etc/prometheus/prometheus.yml
```

Expected result should indicate that the configuration is valid.

Always validate configuration before restarting a production Prometheus server.

---

# 14. Test Prometheus Binary

Check the version:

```bash
/opt/prometheus/prometheus --version
```

This confirms that the binary can execute correctly.

---

# 15. Test Prometheus Manually

Before creating a systemd service, test Prometheus manually.

Example:

```bash
sudo -u prometheus /opt/prometheus/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus
```

Prometheus should start and listen on its HTTP port.

The default port is:

```
9090
```

---

# 16. Verify Port 9090

Check the listening port:

```bash
ss -lntp | grep 9090
```

Expected:

```
Prometheus listening on :9090
```

---

# 17. Access Prometheus

From the server:

```bash
curl http://localhost:9090/-/healthy
```

A healthy server should respond successfully.

You can also access the UI:

```text
http://<server-ip>:9090
```

For production, avoid exposing this directly to the public Internet.

---

# 18. Prometheus Health Endpoint

Prometheus provides health endpoints.

Example:

```bash
curl http://localhost:9090/-/healthy
```

This is useful for:

```
Health Checks

Load Balancers

Kubernetes Probes

Automation
```

---

# 19. Prometheus Readiness Endpoint

You can also check readiness:

```bash
curl http://localhost:9090/-/ready
```

This can be useful for determining whether Prometheus is ready to serve requests.

---

# 20. Create systemd Service

For a Linux production installation, Prometheus should normally run as a systemd service.

Create:

```text
/etc/systemd/system/prometheus.service
```

Example:

```ini
[Unit]
Description=Prometheus Monitoring System
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple

ExecStart=/opt/prometheus/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus

Restart=on-failure

[Install]
WantedBy=multi-user.target
```

---

# 21. Reload systemd

After creating the service:

```bash
sudo systemctl daemon-reload
```

---

# 22. Start Prometheus

```bash
sudo systemctl start prometheus
```

Check status:

```bash
sudo systemctl status prometheus
```

---

# 23. Enable Prometheus at Boot

Enable the service:

```bash
sudo systemctl enable prometheus
```

This ensures Prometheus starts automatically after a server reboot.

---

# 24. Verify Service

Check:

```bash
sudo systemctl status prometheus
```

You want:

```
Active: active (running)
```

If it fails:

```bash
sudo journalctl -u prometheus -f
```

---

# 25. View Prometheus Logs

Use:

```bash
sudo journalctl -u prometheus
```

Follow logs:

```bash
sudo journalctl -u prometheus -f
```

This is useful for troubleshooting:

```
Configuration Errors

Permission Problems

Storage Problems

Scrape Problems

Startup Failures
```

---

# 26. Check Prometheus Process

Use:

```bash
ps -ef | grep prometheus
```

You should see Prometheus running under the:

```
prometheus
```

user.

---

# 27. Verify Prometheus Endpoint

Run:

```bash
curl http://localhost:9090
```

You should receive an HTTP response from Prometheus.

---

# 28. Verify Targets

Open the Prometheus UI and navigate to:

```
Status
   ↓
Targets
```

You should see:

```
prometheus
```

with:

```
State = UP
```

---

# 29. Query the `up` Metric

Open the Prometheus query interface and run:

```promql
up
```

You should see something similar to:

```text
up{instance="localhost:9090",job="prometheus"} 1
```

This confirms that Prometheus is successfully scraping itself.

---

# 30. Query Prometheus Build Information

You can query:

```promql
prometheus_build_info
```

This can provide information about the running Prometheus build.

---

# 31. Query Scrape Duration

You can inspect scrape duration using Prometheus's scrape metrics.

For example:

```promql
scrape_duration_seconds
```

This helps determine how long target scrapes are taking.

---

# 32. Basic Installation Flow

The complete Linux installation is:

```
Create User
    ↓
Create Directories
    ↓
Download Prometheus
    ↓
Extract Archive
    ↓
Install Binaries
    ↓
Configure prometheus.yml
    ↓
Validate Configuration
    ↓
Create systemd Service
    ↓
Start Prometheus
    ↓
Enable at Boot
    ↓
Verify /-/healthy
    ↓
Check Targets
    ↓
Query up
```

---

# 33. Firewall Configuration

Prometheus commonly listens on:

```
TCP 9090
```

If remote access is required, allow access only from trusted monitoring networks.

Do not blindly expose:

```text
0.0.0.0:9090
```

to the public Internet.

---

# 34. AWS Security Group

If Prometheus is running on EC2 and remote access is required:

```
Security Group
     ↓
TCP 9090
     ↓
Trusted Source
```

Prefer:

```
VPN

Bastion

Internal Network

Private Load Balancer
```

rather than public exposure.

---

# 35. Private Subnet Architecture

A production AWS architecture may look like:

```
Internet
   X
   |
   X
Prometheus
```

Instead:

```
Engineer
   ↓
VPN / Corporate Network
   ↓
Private Subnet
   ↓
Prometheus
```

This reduces attack surface.

---

# 36. Prometheus Storage

The TSDB data directory is:

```text
/var/lib/prometheus
```

It contains Prometheus metric data.

For production:

```
Use Persistent Storage

Monitor Disk Usage

Plan Retention

Plan Recovery
```

---

# 37. Persistent Disk

Prometheus should have sufficient disk space.

For AWS EC2:

```
Prometheus
    ↓
EBS Volume
    ↓
/var/lib/prometheus
```

The disk size should be based on actual:

```
Samples/sec

Active Series

Retention

Growth
```

---

# 38. Disk Monitoring

Monitor:

```
Filesystem Usage

Disk I/O

Available Space
```

Alert before the disk reaches capacity.

A full Prometheus disk can cause serious monitoring failures.

---

# 39. Prometheus Retention

Retention controls how long local Prometheus data is retained.

Example:

```bash
--storage.tsdb.retention.time=15d
```

This means approximately 15 days of local time-based retention.

The correct value depends on:

```
Disk Capacity

Monitoring Requirements

Long-Term Storage Architecture
```

---

# 40. Retention by Size

Prometheus can also use a size-based retention limit.

Example concept:

```bash
--storage.tsdb.retention.size=50GB
```

In production, retention policies should be chosen based on actual storage requirements and tested behavior.

---

# 41. Time and Size Retention

You can configure retention using both:

```
Time

Size
```

This helps prevent unlimited storage growth.

The exact flags and supported behavior should be checked against the Prometheus version being deployed.

---

# 42. Production Resource Planning

Prometheus should have:

```
CPU

Memory

Disk
```

sized according to monitoring workload.

Do not choose resources only based on the number of servers.

Consider:

```
Active Series

Scrape Frequency

Query Load

Retention

Number of Rules
```

---

# 43. Docker Installation

Prometheus can also run as a Docker container.

Architecture:

```
Docker Host
   ↓
Prometheus Container
   ↓
prometheus.yml
   ↓
Persistent Volume
   ↓
Prometheus TSDB
```

---

# 44. Docker Directory Structure

Example:

```text
prometheus/
├── prometheus.yml
└── data/
```

The configuration file is mounted into the container.

The data directory is mounted as persistent storage.

---

# 45. Docker Prometheus Configuration

Example:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"
```

For container-to-container communication, the target should normally use the appropriate Docker network address rather than blindly using localhost.

---

# 46. Docker Run

A simplified Docker example:

```bash
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml:ro \
  -v $(pwd)/data:/prometheus \
  prom/prometheus
```

This provides:

```
Port Mapping

Configuration Mount

Persistent Data Mount
```

---

# 47. Docker Configuration Explanation

```text
-p 9090:9090
```

maps:

```
Host 9090
    ↓
Container 9090
```

Configuration:

```text
prometheus.yml
        ↓
/etc/prometheus/prometheus.yml
```

Data:

```text
./data
   ↓
/prometheus
```

---

# 48. Verify Docker Container

```bash
docker ps
```

Check logs:

```bash
docker logs prometheus
```

Check health:

```bash
curl http://localhost:9090/-/healthy
```

---

# 49. Docker Compose Installation

A simple Docker Compose architecture:

```text
docker-compose.yml
        |
        +-- Prometheus
        |
        +-- Persistent Volume
```

Example:

```yaml
services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus-data:/prometheus
    restart: unless-stopped

volumes:
  prometheus-data:
```

---

# 50. Start Docker Compose

```bash
docker compose up -d
```

Check:

```bash
docker compose ps
```

Logs:

```bash
docker compose logs -f prometheus
```

---

# 51. Docker Production Considerations

For production, consider:

```
Persistent Volumes

Resource Limits

Container Security

Network Isolation

Version Pinning

Backup / Recovery Strategy

Storage Monitoring
```

Do not use an uncontrolled:

```text
latest
```

image tag in a production deployment process.

Pin an approved version according to your organization's release process.

---

# 52. Kubernetes Installation

In Kubernetes, Prometheus is commonly installed using Helm.

A popular approach is:

```
kube-prometheus-stack
```

It can deploy:

```
Prometheus

Alertmanager

Grafana

Node Exporter

kube-state-metrics
```

and related monitoring resources.

---

# 53. Why Helm Is Commonly Used

Installing the complete monitoring stack manually can require many Kubernetes objects.

Helm provides:

```
Packaging

Configuration

Versioning

Upgrades

Rollbacks
```

This makes it practical for production Kubernetes deployments.

---

# 54. Kubernetes Prerequisites

Before installing Prometheus in EKS, verify:

```bash
kubectl cluster-info
```

Check nodes:

```bash
kubectl get nodes
```

Check Helm:

```bash
helm version
```

Make sure the Kubernetes context points to the intended cluster.

---

# 55. Create Monitoring Namespace

Create:

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

---

# 56. Add Prometheus Community Repository

Add the Helm repository:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

Update repositories:

```bash
helm repo update
```

Verify:

```bash
helm repo list
```

---

# 57. Search Helm Chart

You can search:

```bash
helm search repo prometheus-community
```

You should see the available Prometheus-related charts.

---

# 58. Install kube-prometheus-stack

A basic installation:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring
```

This creates a monitoring stack in the:

```
monitoring
```

namespace.

---

# 59. Verify Helm Installation

Run:

```bash
helm list -n monitoring
```

You should see the release:

```
monitoring
```

---

# 60. Check Pods

```bash
kubectl get pods -n monitoring
```

Depending on the chart configuration, you should see components such as:

```
Prometheus

Alertmanager

Grafana

Node Exporter

kube-state-metrics
```

---

# 61. Check Services

```bash
kubectl get svc -n monitoring
```

You should see services associated with the monitoring components.

---

# 62. Check StatefulSets

Prometheus commonly runs using a StatefulSet when persistent identity/storage is required by the deployment architecture.

Check:

```bash
kubectl get statefulset -n monitoring
```

---

# 63. Check PersistentVolumeClaims

If persistence is enabled:

```bash
kubectl get pvc -n monitoring
```

You should see the PVC associated with Prometheus storage.

---

# 64. Access Prometheus in Kubernetes

For a temporary test:

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
```

Then access:

```text
http://localhost:9090
```

The exact service name can vary depending on the Helm release name.

Check:

```bash
kubectl get svc -n monitoring
```

before running the command.

---

# 65. Port Forwarding

Port forwarding is useful for:

```
Development

Testing

Troubleshooting
```

It is not normally the preferred production access method.

Production access should use an appropriate internal access architecture.

---

# 66. Access Grafana

A temporary port-forward can also be used for Grafana.

First identify the service:

```bash
kubectl get svc -n monitoring
```

Then forward the Grafana service to a local port.

Example:

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Then:

```text
http://localhost:3000
```

---

# 67. Production Grafana Access

For production, consider:

```
Internal Load Balancer

Ingress

HTTPS

Authentication

SSO

Network Restrictions
```

Do not expose Grafana directly to the public Internet without a properly designed security architecture.

---

# 68. Production Prometheus Access

Prometheus itself can remain internal.

Example:

```
Engineer
   ↓
VPN
   ↓
Internal Network
   ↓
Grafana
   ↓
Prometheus
```

Grafana can query Prometheus without exposing Prometheus publicly.

---

# 69. Prometheus Service in Kubernetes

The Prometheus service allows other components to communicate with Prometheus.

Architecture:

```
Grafana
   ↓
Kubernetes Service
   ↓
Prometheus Pod
```

The service provides stable discovery even when Prometheus pods change.

---

# 70. Prometheus Operator

The kube-prometheus-stack installation commonly uses the Prometheus Operator.

The Operator introduces Kubernetes-native resources for monitoring.

Examples include:

```
Prometheus

ServiceMonitor

PodMonitor

PrometheusRule
```

These resources allow monitoring configuration to be represented declaratively in Kubernetes.

---

# 71. Prometheus Custom Resources

The Prometheus Operator can manage:

```
Prometheus Instances

Alertmanager Instances

ServiceMonitors

PodMonitors

PrometheusRules
```

This integrates Prometheus configuration with Kubernetes objects.

---

# 72. ServiceMonitor

A ServiceMonitor tells the Prometheus Operator how to monitor a Kubernetes Service.

Conceptually:

```
Kubernetes Service
      ↓
ServiceMonitor
      ↓
Prometheus
      ↓
/metrics
```

---

# 73. ServiceMonitor Example

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
      app: order-service
  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
```

The exact labels, namespace, and port must match the application Service and the Prometheus Operator configuration.

---

# 74. PodMonitor

PodMonitor can monitor pods directly.

Architecture:

```
Pod
  ↓
PodMonitor
  ↓
Prometheus
```

This can be useful when monitoring pods without going through a Kubernetes Service.

---

# 75. ServiceMonitor vs PodMonitor

ServiceMonitor:

```
Monitors Kubernetes Services
```

PodMonitor:

```
Monitors Pods
```

Use the resource that matches the application's discovery and monitoring architecture.

---

# 76. PrometheusRule

PrometheusRule allows alerting and recording rules to be managed as Kubernetes resources.

Architecture:

```
PrometheusRule
      ↓
Prometheus Operator
      ↓
Prometheus
      ↓
Rule Evaluation
```

---

# 77. Example PrometheusRule

Conceptual example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: application-alerts
  namespace: monitoring
spec:
  groups:
    - name: application.rules
      rules:
        - alert: HighErrorRate
          expr: |
            rate(http_requests_total{status=~"5.."}[5m]) > 1
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "High HTTP error rate"
```

The actual expression should be adapted to the metric names and requirements of the application.

---

# 78. Kubernetes Application Metrics

Suppose an application exposes:

```text
/metrics
```

The deployment may look like:

```
Application Pod
     ↓
  :8080
     ↓
 /metrics
     ↓
Service
     ↓
ServiceMonitor
     ↓
Prometheus
```

---

# 79. Application Metrics Port

A Kubernetes application can expose a dedicated metrics port.

Example:

```yaml
ports:
  - name: http
    containerPort: 8080

  - name: metrics
    containerPort: 9090
```

The metrics endpoint could then be:

```text
/metrics
```

---

# 80. Real-World Microservice Example

Suppose:

```
order-service
```

has:

```
HTTP Port: 8080

Metrics Port: 9090
```

Architecture:

```
order-service
    |
    +-- :8080 application
    |
    +-- :9090 metrics
              ↓
           /metrics
              ↓
         Prometheus
```

---

# 81. Monitoring Multiple Microservices

Suppose the platform contains:

```
user-service

product-service

cart-service

order-service

payment-service
```

Each service exposes:

```text
/metrics
```

Prometheus discovers them through Kubernetes monitoring resources.

Architecture:

```
Services
   ↓
ServiceMonitors
   ↓
Prometheus
   ↓
Grafana
```

---

# 82. EKS Monitoring Architecture

A practical EKS architecture:

```
┌──────────────────────────────────────────────┐
│                    EKS                       │
│                                              │
│  Node 1             Node 2                  │
│    │                  │                     │
│  Node Exporter      Node Exporter           │
│    │                  │                     │
│    └──────────┬───────┘                     │
│               │                             │
│       Application Pods                      │
│               │                             │
│            /metrics                         │
│                                              │
│       kube-state-metrics                    │
│                                              │
└───────────────┬──────────────────────────────┘
                ↓
         Prometheus Operator
                ↓
           Prometheus
                ↓
            Persistent
              Storage
                ↓
              Grafana
```

---

# 83. EKS Storage

For Prometheus persistence in EKS, use an appropriate Kubernetes storage class.

Typical architecture:

```
Prometheus
    ↓
PVC
    ↓
EBS-backed Storage
    ↓
Persistent Volume
```

The exact StorageClass depends on the EKS environment and storage driver configuration.

---

# 84. Check Storage Classes

Run:

```bash
kubectl get storageclass
```

Identify the storage class intended for Prometheus.

---

# 85. Persistent Volume Claim

A simplified PVC:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Gi
```

The actual production storage size should be calculated from monitoring workload requirements.

---

# 86. Helm Values

Production Helm deployments should generally use a dedicated values file.

Example:

```text
prometheus-values.yaml
```

This makes configuration:

```
Version Controlled

Reviewable

Repeatable

Auditable
```

---

# 87. Example Production Values

A simplified conceptual values file:

```yaml
prometheus:
  prometheusSpec:
    retention: 15d

    resources:
      requests:
        cpu: 500m
        memory: 2Gi
      limits:
        cpu: "2"
        memory: 4Gi

    storageSpec:
      volumeClaimTemplate:
        spec:
          resources:
            requests:
              storage: 50Gi
```

These are examples only.

Production values must be sized according to actual workload.

---

# 88. Install with Values

Example:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f prometheus-values.yaml
```

---

# 89. Upgrade with Values

When configuration changes:

```bash
helm upgrade monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f prometheus-values.yaml
```

---

# 90. Helm Rollback

If a deployment causes a problem:

```bash
helm history monitoring -n monitoring
```

Then:

```bash
helm rollback monitoring <REVISION> -n monitoring
```

This is one reason version-controlled Helm values are useful.

---

# 91. Verify Helm Deployment

After installation:

```bash
helm status monitoring -n monitoring
```

Then:

```bash
kubectl get pods -n monitoring
```

Then:

```bash
kubectl get svc -n monitoring
```

Then:

```bash
kubectl get pvc -n monitoring
```

---

# 92. Check Prometheus CR

With the Prometheus Operator:

```bash
kubectl get prometheus -n monitoring
```

---

# 93. Check ServiceMonitors

```bash
kubectl get servicemonitor -A
```

This helps determine whether Kubernetes monitoring resources exist.

---

# 94. Check PodMonitors

```bash
kubectl get podmonitor -A
```

---

# 95. Check PrometheusRules

```bash
kubectl get prometheusrule -A
```

This shows configured recording and alerting rule resources.

---

# 96. Check Prometheus Logs in Kubernetes

Find the Prometheus pod:

```bash
kubectl get pods -n monitoring
```

Then:

```bash
kubectl logs <prometheus-pod> -n monitoring
```

Follow:

```bash
kubectl logs -f <prometheus-pod> -n monitoring
```

---

# 97. Describe Prometheus Pod

```bash
kubectl describe pod <prometheus-pod> -n monitoring
```

Check:

```
Events

Volumes

Mounts

Resource Configuration

Container State
```

---

# 98. Check Prometheus Targets

After installation, access Prometheus and check:

```
Status
   ↓
Targets
```

You should see targets such as:

```
Prometheus

Node Exporter

kube-state-metrics

Kubernetes components
```

The exact target list depends on configuration and chart version.

---

# 99. Query Kubernetes Metrics

After installation, test:

```promql
up
```

Then:

```promql
prometheus_build_info
```

You can also query application-specific metrics after instrumentation is configured.

---

# 100. Node Exporter Verification

Check:

```bash
kubectl get pods -n monitoring
```

Look for Node Exporter pods.

Then query relevant node metrics in Prometheus.

---

# 101. kube-state-metrics Verification

Check:

```bash
kubectl get pods -n monitoring
```

Look for kube-state-metrics.

Then query Kubernetes object metrics in Prometheus.

---

# 102. Grafana Verification

Check:

```bash
kubectl get pods -n monitoring
```

Then:

```bash
kubectl get svc -n monitoring
```

Access Grafana using an approved internal access method.

Verify:

```
Prometheus Data Source

Dashboards

Kubernetes Metrics
```

---

# 103. Alertmanager Verification

Check:

```bash
kubectl get pods -n monitoring
```

Find the Alertmanager pod.

Then verify the Alertmanager service:

```bash
kubectl get svc -n monitoring
```

---

# 104. Installation Verification Checklist

After installation verify:

```
[ ] Prometheus pod running

[ ] Prometheus service exists

[ ] Prometheus UI accessible

[ ] Prometheus health endpoint works

[ ] Prometheus target is UP

[ ] Node Exporter running

[ ] kube-state-metrics running

[ ] Grafana running

[ ] Alertmanager running

[ ] PVC bound

[ ] PromQL query works
```

---

# 105. Linux Installation Troubleshooting

If Prometheus does not start:

```bash
sudo systemctl status prometheus
```

Then:

```bash
sudo journalctl -u prometheus -n 100
```

Check configuration:

```bash
/opt/prometheus/promtool check config /etc/prometheus/prometheus.yml
```

Check permissions:

```bash
ls -l /etc/prometheus/
ls -ld /var/lib/prometheus/
```

---

# 106. Permission Problem

If Prometheus cannot write to storage:

Check:

```bash
ls -ld /var/lib/prometheus
```

Correct ownership:

```bash
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

Restart:

```bash
sudo systemctl restart prometheus
```

---

# 107. Port Already in Use

Check:

```bash
sudo ss -lntp | grep 9090
```

If another process is using the port:

```
Identify Process

Stop or Reconfigure It

Start Prometheus
```

Do not kill processes blindly in production.

---

# 108. Invalid Configuration

If Prometheus fails to start because of YAML errors:

```bash
/opt/prometheus/promtool check config /etc/prometheus/prometheus.yml
```

Fix the configuration.

Then:

```bash
sudo systemctl restart prometheus
```

---

# 109. Docker Troubleshooting

Check:

```bash
docker ps -a
```

Logs:

```bash
docker logs prometheus
```

Inspect:

```bash
docker inspect prometheus
```

Check volume mounts and configuration paths.

---

# 110. Kubernetes Troubleshooting

Check:

```bash
kubectl get pods -n monitoring
```

If a pod is not running:

```bash
kubectl describe pod <pod> -n monitoring
```

Check logs:

```bash
kubectl logs <pod> -n monitoring
```

Check events:

```bash
kubectl get events -n monitoring --sort-by=.lastTimestamp
```

---

# 111. CrashLoopBackOff During Installation

If Prometheus enters:

```
CrashLoopBackOff
```

Check:

```bash
kubectl logs <pod> -n monitoring --previous
```

Then:

```bash
kubectl describe pod <pod> -n monitoring
```

Investigate:

```
Configuration

Storage

Permissions

Resource Limits

Startup Errors
```

---

# 112. Pending Prometheus Pod

If:

```
STATUS = Pending
```

check:

```bash
kubectl describe pod <pod> -n monitoring
```

Common causes:

```
Insufficient CPU

Insufficient Memory

PVC Pending

Node Selector

Taints / Tolerations

Affinity Rules
```

---

# 113. PVC Pending

Check:

```bash
kubectl get pvc -n monitoring
```

Then:

```bash
kubectl describe pvc <pvc-name> -n monitoring
```

Check:

```bash
kubectl get storageclass
```

Possible causes:

```
No StorageClass

Wrong StorageClass

CSI Driver Problem

Capacity Problem

Access Mode Problem
```

---

# 114. Prometheus OOMKilled

Check:

```bash
kubectl describe pod <prometheus-pod> -n monitoring
```

Look for:

```
OOMKilled
```

Then investigate:

```
Active Series

Cardinality

Scrape Volume

Query Load

Resource Limits
```

Increase resources only after understanding the workload.

---

# 115. Prometheus Not Scraping Application

Troubleshooting:

```
Application Pod
     ↓
/metrics
     ↓
Service
     ↓
ServiceMonitor
     ↓
Prometheus
```

Check each layer.

---

# 116. Test Application Metrics

From an appropriate network location:

```bash
curl http://<service>:<port>/metrics
```

If metrics are not returned:

```
Application instrumentation problem
```

If metrics are returned:

```
Continue with ServiceMonitor / discovery troubleshooting.
```

---

# 117. Check ServiceMonitor

Run:

```bash
kubectl get servicemonitor -A
```

Then inspect:

```bash
kubectl describe servicemonitor <name> -n <namespace>
```

Verify:

```
Selector

Namespace

Port

Path

Interval
```

---

# 118. Label Matching

A common problem is that ServiceMonitor selectors do not match the Service labels.

For example:

Service:

```yaml
labels:
  app: order-service
```

ServiceMonitor:

```yaml
selector:
  matchLabels:
    app: payment-service
```

This will not match.

The selector must correspond to the intended Service.

---

# 119. Port Name Matching

If the ServiceMonitor specifies:

```yaml
endpoints:
  - port: metrics
```

the Service should expose a port named:

```yaml
ports:
  - name: metrics
```

A mismatch can prevent scraping.

---

# 120. Metrics Path

The default metrics path is commonly:

```text
/metrics
```

If the application exposes:

```text
/actuator/prometheus
```

then configure the appropriate path.

Example:

```yaml
endpoints:
  - port: metrics
    path: /actuator/prometheus
```

---

# 121. Real-World Java Application Example

Spring Boot applications can expose Prometheus-compatible metrics through an appropriate metrics library and endpoint configuration.

Architecture:

```
Java Application
    ↓
Micrometer
    ↓
Prometheus Endpoint
    ↓
Kubernetes Service
    ↓
ServiceMonitor
    ↓
Prometheus
```

---

# 122. Real-World Node.js Application Example

Node.js applications can use a Prometheus client library.

Architecture:

```
Node.js
   ↓
Prometheus Client
   ↓
/metrics
   ↓
Kubernetes Service
   ↓
ServiceMonitor
   ↓
Prometheus
```

---

# 123. Real-World Python Application Example

Python applications can use a Prometheus client library.

Architecture:

```
Python
   ↓
Prometheus Client
   ↓
/metrics
   ↓
Service
   ↓
ServiceMonitor
   ↓
Prometheus
```

---

# 124. Application Instrumentation Responsibility

The application team should define:

```
Metric Names

Metric Types

Labels

Business Metrics

HTTP Metrics

Dependency Metrics
```

The platform team should provide:

```
Prometheus

Discovery

Scraping

Storage

Dashboards

Alerting
```

---

# 125. Production Installation Model

A mature organization might use:

```
Git Repository
     ↓
Helm Values
     ↓
CI/CD
     ↓
Kubernetes
     ↓
kube-prometheus-stack
     ↓
Prometheus
```

Configuration should be version controlled.

---

# 126. GitOps Installation

In a GitOps environment:

```
Git
  ↓
Helm Values
  ↓
ArgoCD
  ↓
EKS
  ↓
Prometheus
```

This provides:

```
Version Control

Review

Auditability

Reproducibility

Drift Detection
```

---

# 127. Prometheus Installation Through ArgoCD

A production workflow can be:

```
Developer / DevOps Engineer
         ↓
      Git Commit
         ↓
      Pull Request
         ↓
    Review / Approval
         ↓
       ArgoCD
         ↓
    EKS Monitoring
         ↓
     Prometheus
```

This fits well with a GitOps-based DevOps environment.

---

# 128. Production Namespace Strategy

Keep monitoring components in a dedicated namespace.

Example:

```text
monitoring
```

Benefits:

```
Easier Management

Resource Isolation

RBAC Organization

Monitoring Operations
```

---

# 129. RBAC

Prometheus Operator and Kubernetes discovery require appropriate Kubernetes permissions.

The Helm chart manages required Kubernetes resources according to its configuration.

In production:

```
Review RBAC

Follow Least Privilege

Avoid Unnecessary Cluster Permissions
```

---

# 130. NetworkPolicy

A production cluster can restrict monitoring traffic.

Example concept:

```
Prometheus
   ↓
Allow
   ↓
Application Metrics
```

But:

```
Unrelated Pods
   X
Prometheus
```

Network policies should be tested carefully to avoid breaking discovery or scraping.

---

# 131. Resource Requests and Limits

Prometheus should have explicit resource configuration.

Example:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 2Gi
  limits:
    cpu: "2"
    memory: 4Gi
```

These values are examples and should be tuned based on workload.

---

# 132. Anti-Pattern: No Persistence

Avoid:

```
Prometheus
   ↓
EmptyDir
   ↓
Pod Restart
   ↓
Historical Metrics Lost
```

For production, use appropriate persistent storage if local retention is required.

---

# 133. Anti-Pattern: No Resource Limits

Avoid deploying Prometheus without understanding resource consumption.

A large Prometheus workload can compete with application workloads.

Use:

```
Requests

Limits

Dedicated Nodes where appropriate

Capacity Planning
```

---

# 134. Anti-Pattern: Public Prometheus

Avoid:

```
Internet
   ↓
Prometheus:9090
```

Instead:

```
Engineer
   ↓
Secure Access
   ↓
Internal Prometheus / Grafana
```

---

# 135. Anti-Pattern: Unpinned Versions

Avoid uncontrolled production upgrades.

Use:

```
Approved Version

Version-Controlled Configuration

Testing

Staging Validation

Production Rollout
```

---

# 136. Installation Validation Flow

After installation:

```
Prometheus Running?
      ↓
    YES
      ↓
Health Endpoint?
      ↓
    YES
      ↓
Targets UP?
      ↓
    YES
      ↓
Metrics Available?
      ↓
    YES
      ↓
PromQL Works?
      ↓
    YES
      ↓
Grafana Connected?
      ↓
    YES
      ↓
Alerts Working?
      ↓
    YES
      ↓
Production Ready
```

---

# 137. Linux Installation Checklist

```
[ ] Dedicated prometheus user

[ ] /etc/prometheus created

[ ] /var/lib/prometheus created

[ ] Binary installed

[ ] promtool installed

[ ] prometheus.yml configured

[ ] Configuration validated

[ ] systemd service created

[ ] Service started

[ ] Service enabled

[ ] Port 9090 verified

[ ] Health endpoint verified

[ ] Target UP

[ ] Persistent storage configured
```

---

# 138. Docker Installation Checklist

```
[ ] Prometheus image selected

[ ] Version pinned

[ ] Configuration mounted

[ ] Persistent volume configured

[ ] Port configured

[ ] Container running

[ ] Logs checked

[ ] Health endpoint verified
```

---

# 139. Kubernetes Installation Checklist

```
[ ] EKS context verified

[ ] Helm installed

[ ] Monitoring namespace created

[ ] Helm repository added

[ ] Chart version approved

[ ] Values file created

[ ] Prometheus installed

[ ] Prometheus pod running

[ ] Alertmanager running

[ ] Grafana running

[ ] Node Exporter running

[ ] kube-state-metrics running

[ ] PVC bound

[ ] Targets UP

[ ] PromQL verified
```

---

# 140. Production Installation Checklist

## Infrastructure

```
[ ] CPU capacity

[ ] Memory capacity

[ ] Persistent storage

[ ] Network connectivity

[ ] DNS
```

---

## Security

```
[ ] Private access

[ ] RBAC

[ ] NetworkPolicy

[ ] TLS where required

[ ] Authentication
```

---

## Monitoring

```
[ ] Prometheus self-monitoring

[ ] Target monitoring

[ ] Storage monitoring

[ ] Query monitoring

[ ] Alerting
```

---

## Operations

```
[ ] Version controlled configuration

[ ] Backup / recovery plan

[ ] Upgrade procedure

[ ] Rollback procedure

[ ] Capacity planning

[ ] Runbooks
```

---

# 141. Interview Question: How Would You Install Prometheus on Linux?

### Answer

I would install Prometheus using a dedicated non-root user.

My process would be:

```
1. Create the Prometheus user.

2. Create configuration and data directories.

3. Download the approved Prometheus release.

4. Install Prometheus and promtool.

5. Create prometheus.yml.

6. Validate the configuration with promtool.

7. Create a systemd service.

8. Start and enable Prometheus.

9. Verify port 9090.

10. Check the health endpoint.

11. Verify targets.

12. Query the up metric.
```

For production, I would also configure persistent storage, resource sizing, retention, security, and monitoring of Prometheus itself.

---

# 142. Interview Question: How Would You Install Prometheus in EKS?

### Answer

For EKS, I would normally use Helm and the Prometheus Operator-based kube-prometheus-stack.

The flow would be:

```
EKS
  ↓
monitoring namespace
  ↓
Helm
  ↓
kube-prometheus-stack
  ↓
Prometheus
Alertmanager
Grafana
Node Exporter
kube-state-metrics
```

I would maintain the Helm values in Git and preferably deploy through GitOps using ArgoCD.

---

# 143. Interview Question: Why Use Helm for Prometheus?

### Answer

Prometheus monitoring in Kubernetes involves multiple components and Kubernetes resources.

Helm provides:

```
Packaging

Configuration

Versioning

Upgrades

Rollbacks
```

It also makes it easier to maintain production configuration through a values file.

---

# 144. Interview Question: What Is kube-prometheus-stack?

### Answer

kube-prometheus-stack is a Helm chart that provides a Kubernetes monitoring stack based on the Prometheus Operator.

It commonly includes:

```
Prometheus

Alertmanager

Grafana

Node Exporter

kube-state-metrics
```

The exact components and versions depend on the chart configuration.

---

# 145. Interview Question: What Is Prometheus Operator?

### Answer

Prometheus Operator manages Prometheus-related monitoring resources using Kubernetes-native custom resources.

It allows engineers to define monitoring configuration declaratively using resources such as:

```
Prometheus

ServiceMonitor

PodMonitor

PrometheusRule
```

This makes Prometheus monitoring integrate naturally with Kubernetes.

---

# 146. Interview Question: What Is a ServiceMonitor?

### Answer

A ServiceMonitor is a Kubernetes custom resource used by the Prometheus Operator to define how a Kubernetes Service should be scraped.

It specifies information such as:

```
Service Selector

Metrics Port

Path

Scrape Interval
```

Prometheus Operator then configures Prometheus accordingly.

---

# 147. Interview Question: What Is a PodMonitor?

### Answer

PodMonitor is similar to ServiceMonitor but discovers pods directly.

Use:

```
ServiceMonitor
    ↓
Service-based discovery
```

and:

```
PodMonitor
    ↓
Pod-based discovery
```

The appropriate choice depends on the application architecture.

---

# 148. Interview Question: Prometheus Pod Is Pending. How Do You Troubleshoot?

### Answer

I would check:

```bash
kubectl describe pod <pod> -n monitoring
```

Then investigate:

```
Node Capacity

PVC Status

StorageClass

Taints

Tolerations

Node Selectors

Affinity

Resource Requests
```

I would then fix the specific scheduling or storage issue rather than blindly restarting the pod.

---

# 149. Interview Question: Prometheus Is CrashLoopBackOff. What Do You Check?

### Answer

First:

```bash
kubectl logs <pod> -n monitoring --previous
```

Then:

```bash
kubectl describe pod <pod> -n monitoring
```

I would check:

```
Configuration

Prometheus Arguments

Storage

Permissions

Memory

Invalid Rules

Startup Errors
```

Then I would validate the configuration and correct the underlying problem.

---

# 150. Interview Question: Prometheus Is OOMKilled in EKS. What Would You Do?

### Answer

I would not immediately increase memory.

First I would investigate:

```
Active Series

Cardinality

Scrape Volume

Number of Targets

Query Load

Recording Rules

Retention
```

Then I would:

```
Remove unnecessary high-cardinality metrics

Optimize queries

Adjust scrape configuration

Increase resources if the workload genuinely requires it

Consider scaling the monitoring architecture for larger environments
```

---

# 151. Interview Question: How Do You Make Prometheus Installation Reproducible?

### Answer

I would store:

```
Helm Values

Kubernetes Manifests

Prometheus Rules

ServiceMonitors

PodMonitors
```

in Git.

Then use:

```
CI/CD
```

or:

```
ArgoCD
```

to deploy the monitoring stack.

This provides:

```
Version Control

Review

Auditability

Reproducibility

Rollback
```

---

# 152. Interview Question: How Would You Secure Prometheus?

### Answer

I would avoid exposing Prometheus directly to the public Internet.

I would use:

```
Private Subnets

Network Security

Kubernetes NetworkPolicies

RBAC

Authentication

TLS where required

Restricted Access
```

For Kubernetes, I would also review the permissions granted to the Prometheus Operator and monitoring components.

---

# 153. Interview Question: What Would You Check After Installing Prometheus?

### Answer

My validation sequence would be:

```
1. Check Prometheus process/pod.

2. Check logs.

3. Check health endpoint.

4. Open Prometheus UI.

5. Check Targets.

6. Verify `up`.

7. Check Node Exporter.

8. Check kube-state-metrics.

9. Verify application metrics.

10. Verify Grafana.

11. Verify Alertmanager.

12. Verify persistent storage.
```

This ensures the monitoring stack is functioning end-to-end.

---

# 154. Real-World Production Installation Architecture

A production EKS deployment can look like:

```
Git Repository
     ↓
Helm Values
     ↓
ArgoCD
     ↓
EKS
     ↓
kube-prometheus-stack
     │
     ├──────── Prometheus
     │              │
     │              ↓
     │           PVC / EBS
     │
     ├──────── Alertmanager
     │
     ├──────── Grafana
     │
     ├──────── Node Exporter
     │
     └──────── kube-state-metrics
```

Applications:

```
Microservices
     ↓
/metrics
     ↓
ServiceMonitor
     ↓
Prometheus
```

---

# 155. Complete Installation Workflow

For a real production Kubernetes environment:

```
1. Create monitoring namespace.

2. Verify EKS cluster access.

3. Configure Helm repository.

4. Select approved chart version.

5. Create production values file.

6. Configure Prometheus resources.

7. Configure retention.

8. Configure persistent storage.

9. Install kube-prometheus-stack.

10. Verify Prometheus.

11. Verify Alertmanager.

12. Verify Grafana.

13. Verify Node Exporter.

14. Verify kube-state-metrics.

15. Configure application ServiceMonitors.

16. Configure PrometheusRules.

17. Verify targets.

18. Verify PromQL.

19. Configure dashboards.

20. Configure alerts.

21. Integrate with GitOps.

22. Monitor Prometheus itself.
```

---

# 156. Final Installation Mental Model

Linux:

```
Linux Server
    ↓
Prometheus Binary
    ↓
prometheus.yml
    ↓
systemd
    ↓
Prometheus
    ↓
TSDB
    ↓
Grafana
```

Docker:

```
Docker
   ↓
Prometheus Container
   ↓
Config Volume
   ↓
Persistent Data Volume
```

Kubernetes:

```
EKS
   ↓
Helm
   ↓
kube-prometheus-stack
   ↓
Prometheus Operator
   ↓
Prometheus
   ↓
PVC
   ↓
Grafana / Alertmanager
```

Application monitoring:

```
Application
   ↓
/metrics
   ↓
Service
   ↓
ServiceMonitor
   ↓
Prometheus
```

Production GitOps:

```
Git
  ↓
Helm Values
  ↓
ArgoCD
  ↓
EKS
  ↓
Monitoring Stack
```

---

# 157. Final Prometheus Installation Checklist

```
[ ] Choose installation method

[ ] Choose approved Prometheus version

[ ] Create dedicated user where applicable

[ ] Create configuration directory

[ ] Create persistent data directory

[ ] Install Prometheus binary or Helm chart

[ ] Configure prometheus.yml / Helm values

[ ] Validate configuration

[ ] Configure storage

[ ] Configure retention

[ ] Configure resources

[ ] Start Prometheus

[ ] Enable automatic startup where applicable

[ ] Verify health

[ ] Verify port

[ ] Verify targets

[ ] Verify `up`

[ ] Install exporters

[ ] Configure Kubernetes discovery

[ ] Configure ServiceMonitors / PodMonitors

[ ] Configure PrometheusRules

[ ] Verify Grafana

[ ] Verify Alertmanager

[ ] Secure access

[ ] Monitor Prometheus itself

[ ] Version-control configuration

[ ] Integrate with GitOps
```

The important production principle is:

```
INSTALL
   ↓
CONFIGURE
   ↓
PERSIST
   ↓
SECURE
   ↓
VERIFY
   ↓
MONITOR
   ↓
AUTOMATE
```

Prometheus installation is not complete just because the Prometheus process is running.

A production-ready installation must also have:

```
Working Scrapes

Correct Metrics

Persistent Storage

Resource Planning

Secure Access

Alerting

Dashboards

Monitoring of Prometheus

Recovery Strategy

Version-Controlled Configuration
```
