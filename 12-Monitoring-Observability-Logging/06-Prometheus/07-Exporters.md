# Prometheus Exporters

Exporters are one of the most important components in the Prometheus ecosystem.

An exporter collects metrics from a system, application, database, or service and exposes those metrics in a format that Prometheus can scrape.

The basic architecture is:

```text
Application / System
        ↓
     Exporter
        ↓
   /metrics endpoint
        ↓
    Prometheus
        ↓
       TSDB
        ↓
     Grafana
```

---

# 1. What Is an Exporter?

An exporter is a component that exposes metrics in the Prometheus exposition format.

For example, Linux itself does not normally expose all system metrics in Prometheus format.

Node Exporter solves this problem.

```text
Linux Server
     ↓
Node Exporter
     ↓
http://server:9100/metrics
     ↓
Prometheus
```

Prometheus periodically scrapes the `/metrics` endpoint.

---

# 2. Why Do We Need Exporters?

Prometheus works by scraping metrics.

But many systems do not natively expose Prometheus metrics.

Examples:

```text
Linux
MySQL
PostgreSQL
Redis
HAProxy
Nginx
Apache
Kafka
RabbitMQ
Blackbox endpoints
Windows
```

Exporters bridge the gap.

```text
System
  ↓
Exporter
  ↓
Prometheus metrics
```

---

# 3. Exporter Architecture

A typical exporter architecture is:

```text
                 ┌─────────────────┐
                 │   Target System │
                 └────────┬────────┘
                          │
                     Collect Data
                          │
                          ↓
                 ┌─────────────────┐
                 │    Exporter     │
                 └────────┬────────┘
                          │
                     /metrics
                          │
                          ↓
                 ┌─────────────────┐
                 │   Prometheus    │
                 └────────┬────────┘
                          │
                          ↓
                       Grafana
```

---

# 4. Exporter vs Application Instrumentation

There are two common ways to expose metrics.

## Application Instrumentation

The application itself exposes metrics.

```text
Java Application
      ↓
Prometheus Client Library
      ↓
/metrics
      ↓
Prometheus
```

Examples:

```text
Java
Node.js
Python
Go
```

---

## Exporter

An external exporter collects metrics from another system.

```text
Linux
  ↓
Node Exporter
  ↓
/metrics
  ↓
Prometheus
```

---

# 5. When Should You Use an Exporter?

Use an exporter when:

* The target system does not natively expose Prometheus metrics.
* You need infrastructure-level metrics.
* You need database metrics.
* You need network or endpoint probing.
* You need metrics from an existing third-party system.

---

# 6. Common Prometheus Exporters

Some commonly used exporters include:

| Exporter             | Purpose                  |
| -------------------- | ------------------------ |
| Node Exporter        | Linux host metrics       |
| Windows Exporter     | Windows metrics          |
| Blackbox Exporter    | Endpoint/network probing |
| MySQL Exporter       | MySQL metrics            |
| PostgreSQL Exporter  | PostgreSQL metrics       |
| Redis Exporter       | Redis metrics            |
| HAProxy Exporter     | HAProxy metrics          |
| SNMP Exporter        | SNMP devices             |
| NVIDIA DCGM Exporter | NVIDIA GPU metrics       |
| Kafka Exporters      | Kafka-related metrics    |
| JMX Exporter         | JVM/JMX metrics          |
| Nginx Exporter       | Nginx metrics            |

The exact exporter should be selected according to the target system and supported metrics.

---

# 7. Node Exporter

Node Exporter is one of the most important exporters for DevOps engineers.

It exposes hardware and operating-system metrics from Linux systems.

Architecture:

```text
Linux Server
     ↓
Node Exporter
     ↓
:9100/metrics
     ↓
Prometheus
```

---

# 8. What Does Node Exporter Monitor?

Node Exporter can expose metrics related to:

```text
CPU
Memory
Disk
Filesystem
Network
Load
Processes
System time
Filesystem statistics
Kernel-related statistics
```

The available metrics depend on the enabled collectors and operating system.

---

# 9. Node Exporter Installation

On a Linux server, download a release appropriate for your architecture.

Example:

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v<VERSION>/node_exporter-<VERSION>.linux-amd64.tar.gz
```

Extract it:

```bash
tar -xvf node_exporter-<VERSION>.linux-amd64.tar.gz
```

Move the binary:

```bash
sudo mv node_exporter-<VERSION>.linux-amd64/node_exporter /usr/local/bin/
```

Verify:

```bash
node_exporter --version
```

---

# 10. Create Node Exporter User

Do not normally run infrastructure exporters as root unless there is a specific requirement.

Create a dedicated user:

```bash
sudo useradd --no-create-home --shell /bin/false node_exporter
```

Set ownership:

```bash
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

---

# 11. Create Systemd Service

Create:

```text
/etc/systemd/system/node_exporter.service
```

Example:

```ini
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

---

# 12. Start Node Exporter

Reload systemd:

```bash
sudo systemctl daemon-reload
```

Enable:

```bash
sudo systemctl enable node_exporter
```

Start:

```bash
sudo systemctl start node_exporter
```

Check:

```bash
sudo systemctl status node_exporter
```

---

# 13. Verify Node Exporter

By default, Node Exporter commonly listens on:

```text
9100
```

Check:

```bash
ss -lntp | grep 9100
```

Test:

```bash
curl http://localhost:9100/metrics
```

You should see Prometheus-formatted metrics.

---

# 14. Example Node Exporter Metrics

Examples include:

```text
node_cpu_seconds_total
node_memory_MemAvailable_bytes
node_memory_MemTotal_bytes
node_filesystem_avail_bytes
node_filesystem_size_bytes
node_network_receive_bytes_total
node_network_transmit_bytes_total
```

Metric availability depends on Node Exporter version and enabled collectors.

---

# 15. Configure Prometheus for Node Exporter

Example:

```yaml
scrape_configs:
  - job_name: "node-exporter"

    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
```

Then Prometheus scrapes:

```text
10.0.1.10:9100/metrics
10.0.1.11:9100/metrics
```

---

# 16. Production Node Exporter Architecture

For multiple EC2 instances:

```text
                AWS
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
     EC2-1     EC2-2     EC2-3
       │         │         │
       ↓         ↓         ↓
     Node       Node       Node
   Exporter   Exporter   Exporter
       │         │         │
       └─────────┼─────────┘
                 ↓
          Prometheus
                 ↓
              Grafana
```

---

# 17. Node Exporter with EC2 Service Discovery

Instead of static targets:

```text
EC2-1
EC2-2
EC2-3
```

use EC2 service discovery:

```text
AWS EC2
   ↓
EC2 Service Discovery
   ↓
Prometheus
   ↓
Node Exporter Targets
```

This is especially useful with Auto Scaling Groups.

---

# 18. Node Exporter and Auto Scaling

Suppose:

```text
ASG
Desired = 3
```

During high traffic:

```text
3 → 6 instances
```

New instances automatically run Node Exporter through the machine image or bootstrap process.

Prometheus discovers them through EC2 service discovery.

```text
ASG
 ↓
New EC2
 ↓
Node Exporter
 ↓
EC2 SD
 ↓
Prometheus
```

---

# 19. Node Exporter Collectors

Node Exporter uses collectors to gather different classes of metrics.

Examples include collectors for:

```text
CPU
Filesystem
Memory
Network
Load
System
Processes
```

Collectors can be enabled or disabled according to operational requirements.

---

# 20. Why Collectors Matter

Not every server needs every collector.

Unnecessary collectors can create:

```text
More CPU usage
More metrics
More storage
More cardinality
```

Production environments should enable only what is useful.

---

# 21. Node Exporter Security

Node Exporter exposes system information.

Therefore:

```text
Do not expose port 9100 directly to the public internet.
```

Prefer:

```text
Prometheus
   ↓
Private Network
   ↓
Node Exporter
```

For AWS:

```text
Prometheus Security Group
        ↓
Node Exporter Security Group
        ↓
TCP 9100
```

Only allow the monitoring system to access the exporter.

---

# 22. MySQL Exporter

MySQL Exporter collects metrics from MySQL.

Architecture:

```text
MySQL
  ↓
MySQL Exporter
  ↓
/metrics
  ↓
Prometheus
```

It can expose information related to:

```text
Connections
Queries
Threads
Replication
InnoDB
Locks
Errors
Performance
```

The exact metrics depend on exporter configuration and MySQL version.

---

# 23. MySQL Exporter User

Create a dedicated MySQL monitoring user with only the required permissions.

Example concept:

```sql
CREATE USER 'exporter'@'localhost'
IDENTIFIED BY 'strong-password';
```

Then grant only the permissions required by the exporter version and enabled collectors.

Avoid using:

```text
root
```

for monitoring.

---

# 24. MySQL Exporter Architecture

```text
              MySQL
                │
                ↓
        Monitoring User
                │
                ↓
        MySQL Exporter
                │
            /metrics
                │
                ↓
            Prometheus
                │
                ↓
             Grafana
```

---

# 25. PostgreSQL Exporter

PostgreSQL Exporter collects PostgreSQL metrics.

Architecture:

```text
PostgreSQL
    ↓
Postgres Exporter
    ↓
/metrics
    ↓
Prometheus
```

Common monitoring areas include:

```text
Connections
Transactions
Locks
Replication
Database activity
Table statistics
Query-related statistics
```

---

# 26. Redis Exporter

Redis Exporter collects metrics from Redis.

Architecture:

```text
Redis
 ↓
Redis Exporter
 ↓
/metrics
 ↓
Prometheus
```

Useful monitoring areas include:

```text
Memory
Connected clients
Commands
Keys
Evictions
Persistence
Replication
Latency-related statistics
```

---

# 27. Blackbox Exporter

Blackbox Exporter is different from Node Exporter.

Node Exporter asks:

```text
"How is this server performing?"
```

Blackbox Exporter asks:

```text
"Can I successfully reach this endpoint?"
```

Architecture:

```text
Prometheus
    ↓
Blackbox Exporter
    ↓
Probe Target
    ↓
Result
    ↓
Prometheus
```

---

# 28. What Can Blackbox Exporter Probe?

It can be used for probes such as:

```text
HTTP
HTTPS
TCP
ICMP
DNS
```

depending on the configured modules and permissions.

---

# 29. Blackbox Exporter Use Case

Suppose your application is:

```text
https://example.com
```

You can test:

```text
DNS resolution
TCP connection
TLS
HTTP status
HTTP response
```

Architecture:

```text
Prometheus
     ↓
Blackbox Exporter
     ↓
Internet / Internal Endpoint
     ↓
HTTP Response
```

---

# 30. Blackbox Exporter Installation

Download an appropriate release:

```bash
wget https://github.com/prometheus/blackbox_exporter/releases/download/v<VERSION>/blackbox_exporter-<VERSION>.linux-amd64.tar.gz
```

Extract:

```bash
tar -xvf blackbox_exporter-<VERSION>.linux-amd64.tar.gz
```

Move binary:

```bash
sudo mv blackbox_exporter-<VERSION>.linux-amd64/blackbox_exporter /usr/local/bin/
```

Verify:

```bash
blackbox_exporter --version
```

---

# 31. Blackbox Configuration

A basic configuration can define modules.

Example:

```yaml
modules:
  http_2xx:
    prober: http
    timeout: 5s

    http:
      preferred_ip_protocol: "ip4"
```

Start the exporter with the configuration file.

---

# 32. Blackbox Prometheus Configuration

Prometheus uses the blackbox exporter as an intermediary.

Conceptually:

```text
Prometheus
   ↓
Blackbox Exporter
   ↓
Target URL
```

A typical Prometheus configuration uses relabeling to pass the actual probe target to the exporter.

Example:

```yaml
scrape_configs:
  - job_name: "blackbox"

    metrics_path: /probe

    params:
      module:
        - http_2xx

    static_configs:
      - targets:
          - https://example.com

    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target

      - source_labels: [__param_target]
        target_label: instance

      - target_label: __address__
        replacement: blackbox-exporter:9115
```

---

# 33. Blackbox Architecture in Production

```text
                       Prometheus
                           │
                           │ /probe
                           ↓
                  Blackbox Exporter
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
          Website       API          TCP Service
              │            │            │
              └────────────┼────────────┘
                           ↓
                         Result
                           ↓
                       Prometheus
```

---

# 34. Important Blackbox Metrics

Common metrics include:

```text
probe_success
probe_duration_seconds
probe_http_status_code
probe_ssl_earliest_cert_expiry
```

The exact metrics depend on the probe type.

---

# 35. Blackbox Exporter for SSL Monitoring

You can use Blackbox Exporter to monitor certificate expiry.

Conceptually:

```text
Prometheus
    ↓
Blackbox Exporter
    ↓
HTTPS endpoint
    ↓
TLS certificate
    ↓
Expiry metric
```

Then alert before certificates expire.

---

# 36. Nginx Exporter

Nginx Exporter exposes Nginx metrics in Prometheus format.

Architecture:

```text
Nginx
  ↓
Nginx status endpoint
  ↓
Nginx Exporter
  ↓
Prometheus
```

Monitoring can include:

```text
Connections
Requests
Active connections
Reading
Writing
Waiting
```

The available metrics depend on the Nginx status interface and exporter implementation.

---

# 37. Nginx Exporter Architecture

```text
                 Client
                   ↓
                 Nginx
                   │
              Status Data
                   ↓
             Nginx Exporter
                   ↓
                /metrics
                   ↓
               Prometheus
                   ↓
                Grafana
```

---

# 38. JMX Exporter

JMX Exporter is commonly used for Java applications exposing JVM/JMX metrics.

Architecture:

```text
Java Application
      ↓
      JMX
      ↓
JMX Exporter
      ↓
Prometheus Metrics
      ↓
Prometheus
```

It is useful for applications and platforms such as:

```text
Java applications
Kafka
JVM-based services
```

depending on deployment architecture.

---

# 39. JMX Exporter Deployment Models

JMX Exporter can be deployed in different ways depending on the target system.

Common models include:

```text
Java agent
Standalone exporter
```

For modern Java applications, the Java agent approach is commonly used when you control the application JVM.

---

# 40. Java Agent Concept

```text
Java Application
      │
      ├── Application Code
      │
      └── JMX Exporter Agent
                 ↓
              /metrics
                 ↓
             Prometheus
```

The agent can expose selected JMX metrics in Prometheus format.

---

# 41. RabbitMQ Monitoring

RabbitMQ can expose metrics through its Prometheus integration.

In such a case, an external exporter may not always be necessary.

Architecture:

```text
RabbitMQ
   ↓
Prometheus Metrics Endpoint
   ↓
Prometheus
```

This illustrates an important point:

```text
Not every system needs an external exporter.
```

If the application or platform already provides a suitable Prometheus endpoint, scrape it directly.

---

# 42. Kubernetes Metrics

Kubernetes components may expose metrics directly.

For example:

```text
Kubernetes components
       ↓
Prometheus metrics
       ↓
Prometheus
```

Additional exporters may be used for node-level or application-specific monitoring.

---

# 43. Exporter Pattern in Kubernetes

A common Kubernetes pattern is:

```text
                 Kubernetes
                     │
         ┌───────────┼───────────┐
         ↓           ↓           ↓
      Node Pods    DB Pods    App Pods
         │           │           │
         ↓           ↓           ↓
      Exporter     Exporter   Native Metrics
         │           │           │
         └───────────┼───────────┘
                     ↓
                  Prometheus
```

---

# 44. Running Exporters as Sidecars

Some applications use a sidecar exporter.

Architecture:

```text
Pod
┌─────────────────────────────┐
│                             │
│  Application                │
│                             │
│  Exporter                   │
│                             │
└─────────────────────────────┘
          ↓
      /metrics
```

This is useful when the exporter needs local access to the application.

---

# 45. Sidecar Example

Conceptually:

```yaml
containers:
  - name: application
    image: application:latest

  - name: exporter
    image: exporter:latest
```

The two containers share the pod's network namespace.

The exporter can communicate with the application through localhost when appropriate.

---

# 46. Exporter as a Separate Deployment

Another pattern:

```text
Application
     ↓
Service
     ↓
Exporter
     ↓
Prometheus
```

This can make scaling and lifecycle management easier.

---

# 47. DaemonSet Exporters

Node Exporter is commonly deployed as a DaemonSet in Kubernetes.

Architecture:

```text
Node 1 → Node Exporter
Node 2 → Node Exporter
Node 3 → Node Exporter
```

This ensures each node gets an exporter instance.

---

# 48. Node Exporter DaemonSet Architecture

```text
                 EKS Cluster
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      Node 1       Node 2       Node 3
        │            │            │
    Node Exporter Node Exporter Node Exporter
        │            │            │
        └────────────┼────────────┘
                     ↓
                 Prometheus
```

---

# 49. Why DaemonSet?

A DaemonSet ensures a pod runs on each eligible Kubernetes node.

This is ideal for node-level monitoring.

```text
1 Node
 ↓
1 Node Exporter
```

If a new node joins:

```text
New Node
   ↓
DaemonSet
   ↓
Node Exporter automatically scheduled
```

---

# 50. Node Exporter Kubernetes Deployment

In a production Kubernetes environment, you commonly install Node Exporter through the Prometheus Operator ecosystem or Helm-based monitoring stack rather than manually creating every resource.

The architecture is:

```text
Helm / GitOps
     ↓
Node Exporter DaemonSet
     ↓
Nodes
     ↓
Prometheus
```

---

# 51. Exporters and Service Discovery

Exporters work together with Prometheus service discovery.

Example:

```text
Kubernetes
   ↓
Discover Node Exporter Pods
   ↓
Relabeling
   ↓
Prometheus
   ↓
Scrape
```

This means exporters do not eliminate the need for service discovery.

They provide the metrics endpoint.

---

# 52. Exporter Architecture: Two Different Problems

Think about two separate questions.

### Question 1

Where is the target?

```text
Service Discovery
```

### Question 2

How does the target expose metrics?

```text
Application / Exporter
```

Example:

```text
EC2
 ↓
Node Exporter
 ↓
EC2 Service Discovery
 ↓
Prometheus
```

---

# 53. Exporter Labels

Exported metrics can contain labels.

Example:

```text
node_cpu_seconds_total{
  instance="10.0.1.10:9100",
  job="node-exporter",
  mode="idle"
}
```

Prometheus also adds target-level labels.

---

# 54. `job` and `instance`

Two important Prometheus labels are:

```text
job
instance
```

Example:

```text
job="node-exporter"
instance="10.0.1.10:9100"
```

`job` identifies the scrape job.

`instance` commonly identifies the scraped target.

---

# 55. Exporter Authentication

Some exporters expose sensitive system information.

If an exporter supports authentication or TLS, use it where required.

Do not assume:

```text
Internal network = automatically secure
```

Security should include:

```text
Network controls
TLS
Authentication
Authorization
Least privilege
```

according to the environment.

---

# 56. Exporter Network Security

Example AWS architecture:

```text
                 Internet
                    X
                    │
              Node Exporter
                 TCP 9100
                    │
                    ↓
              Private Subnet
                    │
                    ↓
                Prometheus
```

Security group rule:

```text
Source:
Prometheus Security Group

Destination:
Exporter Security Group

Port:
9100
```

Do not allow:

```text
0.0.0.0/0 → 9100
```

unless there is a very specific, justified architecture.

---

# 57. Exporter Resource Usage

Exporters consume resources.

Monitor:

```text
CPU
Memory
Scrape duration
Metric count
Network traffic
```

For large deployments, exporter overhead can become significant.

---

# 58. Exporter Cardinality

Exporters may expose many metrics and labels.

For example, a database exporter can expose metrics across:

```text
Databases
Tables
Connections
Queries
Indexes
Replication
```

Blindly enabling every possible collector may increase cardinality.

Use only metrics required for operational visibility.

---

# 59. Exporter Collector Selection

Many exporters support collector configuration.

Conceptually:

```text
Exporter
 ├── collector A
 ├── collector B
 ├── collector C
 └── collector D
```

Enable the collectors that provide meaningful monitoring value.

---

# 60. Exporter Version Management

Treat exporters as production software.

Track:

```text
Exporter version
Configuration version
Prometheus compatibility
Target-system compatibility
Security advisories
```

Do not randomly upgrade exporters in production without testing.

---

# 61. Exporter Deployment Through Automation

Do not manually install exporters on hundreds of servers.

Use:

```text
Terraform
Ansible
Packer
AMI
Helm
Kubernetes manifests
GitOps
```

depending on the environment.

---

# 62. EC2 + Ansible Example

A common architecture:

```text
Ansible
   ↓
Install Node Exporter
   ↓
Configure systemd
   ↓
Enable service
   ↓
Open private firewall rule
   ↓
Prometheus discovers instance
```

This is repeatable and auditable.

---

# 63. EC2 + Golden AMI

Another production pattern:

```text
Packer
   ↓
Golden AMI
   ↓
Node Exporter preinstalled
   ↓
Launch Template
   ↓
Auto Scaling Group
```

New EC2 instances automatically contain the exporter.

---

# 64. Kubernetes + Helm

For Kubernetes, Helm is commonly used to deploy exporter components.

Example conceptual flow:

```text
Helm
  ↓
Exporter Chart
  ↓
Deployment / DaemonSet
  ↓
Service
  ↓
ServiceMonitor
  ↓
Prometheus
```

The exact chart and values depend on the exporter.

---

# 65. Kubernetes + GitOps

A production approach can be:

```text
Git
 ↓
Exporter Helm Values
 ↓
ArgoCD
 ↓
EKS
 ↓
Exporter
 ↓
ServiceMonitor
 ↓
Prometheus
```

This gives:

```text
Version control
Review
Audit
Rollback
Consistency
```

---

# 66. Exporter Upgrade Strategy

Before upgrading:

```text
1. Check release notes.
2. Check compatibility.
3. Test in development.
4. Test in staging.
5. Validate metrics.
6. Deploy to production.
7. Monitor exporter health.
8. Verify dashboards and alerts.
```

---

# 67. Exporter Failure

What happens if an exporter crashes?

Example:

```text
Node Exporter
     X
Prometheus
```

Prometheus will show:

```text
up{job="node-exporter"} = 0
```

for that target if it can no longer scrape it.

The server itself may still be healthy.

Therefore, exporter health should be monitored.

---

# 68. Exporter Troubleshooting

Start with:

```bash
systemctl status node_exporter
```

Then:

```bash
journalctl -u node_exporter
```

Check port:

```bash
ss -lntp | grep 9100
```

Test endpoint:

```bash
curl http://localhost:9100/metrics
```

---

# 69. Remote Exporter Troubleshooting

From Prometheus host:

```bash
curl http://<target-ip>:9100/metrics
```

If it fails:

```text
Check routing
Check Security Groups
Check firewall
Check NetworkPolicy
Check exporter process
Check port
```

---

# 70. Prometheus Target Troubleshooting

Open the Prometheus Targets page.

Check:

```text
State
Endpoint
Labels
Last Scrape
Scrape Duration
Error
```

If:

```text
UP
```

the exporter is reachable and returning valid Prometheus metrics.

If:

```text
DOWN
```

investigate the scrape error.

---

# 71. Exporter Metrics vs Application Metrics

For a microservices platform:

```text
Node Exporter
     ↓
Infrastructure metrics
```

Application instrumentation:

```text
Application
     ↓
Application metrics
```

Both are important.

---

# 72. Example Microservices Monitoring

Suppose your EKS platform has:

```text
User
Product
Cart
Order
Payment
Inventory
Notification
```

You can have:

```text
Node Exporter
     ↓
Node CPU / Memory / Disk

Application Metrics
     ↓
Request rate / Error rate / Latency

Database Exporter
     ↓
Database health

Blackbox Exporter
     ↓
Endpoint availability
```

All are collected by Prometheus.

---

# 73. Complete Monitoring Architecture

```text
                          EKS
                           │
       ┌───────────────────┼───────────────────┐
       ↓                   ↓                   ↓
   App Services         Worker Pods          Nodes
       │                   │                   │
       ↓                   ↓                   ↓
App Metrics            App Metrics       Node Exporter
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ↓
                     Service Discovery
                           ↓
                       Prometheus
                           │
       ┌───────────────────┼───────────────────┐
       ↓                   ↓                   ↓
    MySQL               Redis             Blackbox
   Exporter            Exporter            Exporter
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ↓
                       Prometheus
                           ↓
                         Grafana
```

---

# 74. Exporters in AWS

For an AWS environment, you may monitor:

```text
EC2 → Node Exporter
MySQL → MySQL Exporter
Redis → Redis Exporter
Application → Native metrics
Endpoints → Blackbox Exporter
```

Cloud-provider-native services may expose metrics through their own AWS integrations, so do not install exporters where they provide no additional value.

---

# 75. Exporters in Kubernetes

A production Kubernetes monitoring architecture may include:

```text
Node Exporter
Kube-state-metrics
Application instrumentation
Database exporters
Blackbox Exporter
JMX Exporter
```

Note that kube-state-metrics is not a traditional exporter in the same sense as Node Exporter, but it serves a similar Prometheus metrics exposure role for Kubernetes object state.

---

# 76. Kube-State-Metrics

Kube-state-metrics exposes metrics about Kubernetes object state.

Examples:

```text
Deployments
Pods
DaemonSets
StatefulSets
Jobs
Nodes
Namespaces
```

It answers questions such as:

```text
How many desired replicas?
How many available replicas?
Which pods are pending?
Which deployments are unavailable?
```

---

# 77. Node Exporter vs Kube-State-Metrics

### Node Exporter

Measures:

```text
Actual node/system behavior
```

Example:

```text
CPU
Memory
Disk
Network
```

### Kube-State-Metrics

Measures:

```text
Kubernetes object state
```

Example:

```text
Desired replicas
Available replicas
Pod phase
Deployment status
```

They solve different monitoring problems.

---

# 78. Kube-State-Metrics Architecture

```text
Kubernetes API
      ↓
Kube-State-Metrics
      ↓
/metrics
      ↓
Prometheus
```

It watches Kubernetes API objects and exposes their state as metrics.

---

# 79. Exporter Selection Strategy

Do not install every exporter.

Ask:

```text
1. Does the target already expose Prometheus metrics?
2. If not, is an exporter available?
3. Which metrics are actually required?
4. What is the exporter overhead?
5. How will it be secured?
6. How will it be discovered?
7. How will it be deployed and upgraded?
```

---

# 80. Production Exporter Standardization

A good organization can standardize:

```text
Exporter image/version
Resource requests
Resource limits
Security context
Service configuration
ServiceMonitor
Labels
Dashboards
Alerts
```

This creates consistent monitoring across teams.

---

# 81. Exporter Resource Configuration

For Kubernetes exporters, define resources.

Example:

```yaml
resources:
  requests:
    cpu: 50m
    memory: 64Mi

  limits:
    cpu: 200m
    memory: 256Mi
```

The correct values should be based on actual exporter behavior and production measurements rather than blindly copying these numbers.

---

# 82. Exporter Health Checks

Where supported, configure:

```text
Liveness probe
Readiness probe
```

For example:

```text
Exporter
   ↓
/metrics
```

can be checked through an appropriate Kubernetes health mechanism.

---

# 83. Exporter Availability

For critical exporters, consider:

```text
Multiple replicas
Pod anti-affinity
PodDisruptionBudget
Node distribution
Resource requests
```

The exact architecture depends on the exporter.

For Node Exporter, a DaemonSet already provides one instance per eligible node.

---

# 84. Exporter and TLS

For sensitive environments:

```text
Prometheus
    ↓ HTTPS
Exporter
```

TLS can protect metrics in transit.

Use TLS where required by security policy or network architecture.

---

# 85. Exporter Authentication

Some target systems require credentials.

Do not hardcode credentials in Git.

Use:

```text
Kubernetes Secrets
AWS Secrets Manager
External Secrets
Vault
```

according to your organization's architecture.

---

# 86. Secret Management Example

Instead of:

```yaml
password: mypassword
```

use:

```text
Secret Manager
      ↓
Kubernetes Secret
      ↓
Exporter
      ↓
Database
```

Ensure the secret is not exposed through logs, manifests, or command-line arguments.

---

# 87. Exporter and NetworkPolicy

For Kubernetes:

```text
Prometheus
   ↓
NetworkPolicy
   ↓
Exporter
```

Allow only the monitoring namespace or Prometheus pods to reach exporter ports.

This follows least-privilege networking.

---

# 88. Exporter Monitoring

Monitor the exporters themselves.

For example:

```promql
up{job="node-exporter"}
```

can identify targets that are unavailable.

Also monitor:

```text
Scrape duration
Scrape failures
Exporter process health
Exporter resource usage
Target count
```

---

# 89. Exporter Alerting

Examples of useful alert conditions:

```text
Exporter target DOWN
Exporter unavailable for several intervals
Scrape failures increasing
Scrape duration unusually high
Exporter resource usage high
```

Avoid alerting on a single transient scrape failure unless the use case requires it.

---

# 90. Exporter Dashboards

A useful infrastructure dashboard can contain:

```text
CPU Usage
Memory Usage
Disk Usage
Filesystem Usage
Network Traffic
Load
Exporter Availability
Scrape Failures
```

For databases:

```text
Connections
Transactions
Replication
Locks
Memory
Query activity
```

For Blackbox:

```text
Endpoint availability
Latency
HTTP status
TLS certificate expiry
```

---

# 91. Golden Signals with Exporters

Exporters help provide infrastructure signals, while application instrumentation provides application signals.

For example:

```text
Latency
Traffic
Errors
Saturation
```

can be constructed from application metrics and infrastructure metrics.

---

# 92. Exporters and SRE Troubleshooting

Suppose users report:

```text
Application is slow.
```

Start with:

```text
Application latency
```

Then investigate:

```text
CPU
Memory
Disk
Network
Database
Redis
External endpoint
```

Exporters provide the supporting infrastructure data.

---

# 93. Example Production Incident

Users report:

```text
Checkout is slow.
```

Application metrics show:

```text
Latency ↑
```

Node Exporter shows:

```text
CPU = 95%
```

Database exporter shows:

```text
Connections normal
```

Redis exporter shows:

```text
Healthy
```

This points toward node/resource pressure rather than immediately blaming the database.

---

# 94. Example Incident: Disk Full

Node Exporter reports:

```text
Filesystem available ↓
```

PromQL can identify filesystem usage.

Conceptually:

```promql
(
  node_filesystem_size_bytes
  - node_filesystem_avail_bytes
)
/
node_filesystem_size_bytes
```

Then alert when usage exceeds an appropriate threshold.

---

# 95. Example Incident: Endpoint Down

Blackbox Exporter reports:

```text
probe_success = 0
```

This tells you that the probe failed.

Then investigate:

```text
DNS
Network
Load Balancer
Application
TLS
HTTP response
```

---

# 96. Example Incident: Database Connection Saturation

Database exporter reports:

```text
Connections ↑
```

Application metrics show:

```text
Request latency ↑
```

This correlation can help identify database connection pressure.

---

# 97. Exporter Data Flow

Remember the complete path:

```text
Target
  ↓
Exporter
  ↓
/metrics
  ↓
Service Discovery
  ↓
Prometheus
  ↓
TSDB
  ↓
PromQL
  ↓
Grafana / Alerting
```

Depending on architecture, discovery occurs before the actual scrape.

---

# 98. Pull Model

Prometheus normally uses a pull model.

```text
Exporter
    ↑
    │ scrape
    │
Prometheus
```

Prometheus initiates the HTTP request.

The exporter does not normally push metrics to Prometheus.

---

# 99. Why Pull Model Is Useful

Advantages include:

```text
Centralized scraping
Target health visibility
Simple debugging
Consistent scrape intervals
Service discovery integration
```

Prometheus knows whether the endpoint is reachable.

---

# 100. Exporter vs Push

Exporter architecture:

```text
Prometheus
     ↓
Exporter
```

Push architecture:

```text
Application
     ↓
Metrics Gateway
```

Prometheus itself is primarily pull-based, although related systems such as Pushgateway can support specific push use cases.

Do not use Pushgateway as a general replacement for exporters or normal service discovery.

---

# 101. Pushgateway Warning

Pushgateway is primarily intended for short-lived batch jobs that cannot be scraped directly.

It should not be used to turn every long-running application into a push-based monitoring architecture.

For long-running services:

```text
Prefer:
Application / Exporter
       ↓
Prometheus scrape
```

---

# 102. Exporter Naming

Use consistent job names.

Examples:

```text
node-exporter
mysql-exporter
redis-exporter
blackbox-exporter
jmx-exporter
```

This makes PromQL and dashboards easier to maintain.

---

# 103. Environment Labels

Add useful environment metadata:

```text
environment="production"
cluster="prod-eks"
region="ap-south-1"
```

Then queries can filter:

```promql
up{environment="production"}
```

---

# 104. Avoid Unnecessary Labels

Do not add:

```text
request_id
user_id
session_id
```

to exporter metrics.

High-cardinality labels can create huge numbers of time series.

---

# 105. Exporter Configuration Management

Store exporter configuration in Git where appropriate:

```text
monitoring/
├── exporters/
│   ├── node-exporter/
│   ├── blackbox-exporter/
│   ├── mysql-exporter/
│   └── redis-exporter/
├── servicemonitors/
└── alerts/
```

Then deploy through CI/CD or GitOps.

---

# 106. Real-World GitOps Flow

A production change might look like:

```text
Developer
   ↓
Git Commit
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
Exporter Configuration
   ↓
Prometheus
```

This provides traceability for monitoring changes.

---

# 107. Exporter Production Checklist

Before deploying an exporter:

```text
[ ] Confirm exporter purpose
[ ] Confirm supported target version
[ ] Choose stable version
[ ] Configure dedicated user
[ ] Configure least privilege
[ ] Secure exporter endpoint
[ ] Configure resource requests
[ ] Configure health checks
[ ] Configure service discovery
[ ] Configure ServiceMonitor if using Operator
[ ] Configure alerts
[ ] Create dashboard
[ ] Test failure scenarios
[ ] Document ownership
```

---

# 108. Node Exporter Checklist

```text
[ ] Dedicated user
[ ] Systemd service
[ ] Port 9100
[ ] Private network access
[ ] Firewall/Security Group
[ ] Service discovery
[ ] Required collectors
[ ] Prometheus scrape
[ ] CPU dashboard
[ ] Memory dashboard
[ ] Disk dashboard
[ ] Network dashboard
[ ] Node availability alert
```

---

# 109. Blackbox Exporter Checklist

```text
[ ] Probe modules
[ ] HTTP/HTTPS configuration
[ ] Timeout
[ ] Target list
[ ] TLS validation
[ ] DNS probing where required
[ ] Alerting
[ ] Certificate expiry monitoring
[ ] Endpoint availability dashboard
```

---

# 110. Database Exporter Checklist

```text
[ ] Dedicated DB monitoring user
[ ] Least privilege
[ ] Secure credentials
[ ] Network connectivity
[ ] Exporter configuration
[ ] Required collectors
[ ] Service discovery
[ ] Scrape validation
[ ] Database dashboard
[ ] Connection alerts
[ ] Replication alerts
```

---

# 111. Exporter Troubleshooting Framework

Use this sequence:

```text
1. Is exporter running?
       ↓
2. Is exporter listening?
       ↓
3. Does /metrics respond?
       ↓
4. Can Prometheus reach it?
       ↓
5. Is target discovered?
       ↓
6. Is target UP?
       ↓
7. Are expected metrics present?
       ↓
8. Are labels correct?
       ↓
9. Are metrics stored?
       ↓
10. Does Grafana query them?
```

---

# 112. Common Exporter Problems

## Problem: Exporter Not Running

Check:

```bash
systemctl status node_exporter
```

Then:

```bash
journalctl -u node_exporter
```

---

# 113. Problem: Port Not Listening

Check:

```bash
ss -lntp | grep 9100
```

If nothing is listening:

```text
Exporter process
Configuration
Systemd
Port
```

must be investigated.

---

# 114. Problem: `/metrics` Returns Error

Check:

```bash
curl http://localhost:9100/metrics
```

Then inspect:

```text
Exporter logs
Collector configuration
Target system
Permissions
```

---

# 115. Problem: Prometheus Cannot Reach Exporter

Check:

```text
Security Group
Firewall
NetworkPolicy
Routing
DNS
Port
TLS
```

Test from the Prometheus network location.

---

# 116. Problem: Target Is DOWN

Check the Prometheus target error.

Examples:

```text
connection refused
connection timed out
context deadline exceeded
404
401
403
TLS error
```

Each points toward a different troubleshooting path.

---

# 117. Problem: 404 From Exporter

Usually indicates the scrape path is wrong.

For example:

```text
Expected:
/metrics

Configured:
/probe
```

Check exporter-specific configuration.

---

# 118. Problem: 401 / 403

This may indicate:

```text
Authentication
Authorization
Credentials
RBAC
```

Do not simply disable security to make scraping work.

---

# 119. Problem: Too Many Metrics

Investigate:

```text
Collectors
Metric relabeling
Target count
Exporter configuration
High-cardinality labels
```

Disable unnecessary collectors or drop unnecessary metrics where appropriate.

---

# 120. Problem: Prometheus Memory Increased

Investigate:

```text
Active series
Scrape targets
Scrape frequency
Metric cardinality
Exporter metric count
Duplicate scraping
```

Do not immediately increase Prometheus memory without identifying the cause.

---

# 121. Exporters and High Availability

Exporters themselves are usually deployed close to the target.

For example:

```text
Each node
   ↓
Node Exporter
```

For centralized exporters, availability requirements depend on the exporter architecture.

Avoid introducing a single exporter instance that becomes a critical bottleneck without evaluating redundancy.

---

# 122. Exporter Scaling

Scaling strategy depends on exporter type.

### Node Exporter

```text
One per node
```

### Database Exporter

Usually:

```text
One exporter per database target
```

or an architecture supported by the exporter.

### Blackbox Exporter

Can scale horizontally if probe volume requires it.

---

# 123. Exporter as Monitoring Infrastructure

Treat exporters as part of the monitoring platform.

They require:

```text
Deployment
Configuration
Security
Upgrades
Monitoring
Alerting
Documentation
```

They are not just small helper binaries.

---

# 124. Real-World Architecture for Your DevOps Environment

A practical AWS + EKS architecture can be:

```text
                         AWS
                          │
          ┌───────────────┴────────────────┐
          │                                │
         EKS                              EC2
          │                                │
    ┌─────┼─────┐                    Node Exporter
    │     │     │                         │
   Apps  Nodes  DB                        │
    │     │     │                         │
    │     │  DB Exporter                  │
    │     │     │                         │
    │  Node Exporter                      │
    │     │                               │
    └─────┼──────────────┬────────────────┘
          │              │
          ↓              ↓
     ServiceMonitor   EC2 SD
          │              │
          └──────┬───────┘
                 ↓
             Prometheus
                 │
        ┌────────┼─────────┐
        ↓        ↓         ↓
      Grafana  Alerting  Long-term
                         Storage
```

---

# 125. Complete Monitoring Stack

For a production microservices platform:

```text
                   ┌─────────────────────┐
                   │      Grafana        │
                   └──────────┬──────────┘
                              │
                         PromQL Queries
                              │
                              ↓
                   ┌─────────────────────┐
                   │     Prometheus     │
                   └──────────┬──────────┘
                              │
                 ┌────────────┼────────────┐
                 ↓            ↓            ↓
             Kubernetes      EC2        External
              Discovery       SD         Probes
                 │            │            │
                 ↓            ↓            ↓
             Exporters     Exporters    Blackbox
                 │            │            │
       ┌─────────┼────────────┼────────────┤
       ↓         ↓            ↓            ↓
    Node       MySQL        Redis      Application
   Exporter    Exporter     Exporter     Metrics
```

---

# 126. Key Difference: Exporter vs Service Discovery

This is an important interview point.

Exporter:

```text
Converts/exposes metrics.
```

Service discovery:

```text
Finds the targets.
```

Prometheus:

```text
Scrapes and stores the metrics.
```

Example:

```text
EC2
 ↓
Node Exporter
 ↓
EC2 Service Discovery
 ↓
Prometheus
```

---

# 127. Key Difference: Node Exporter vs Kube-State-Metrics

Node Exporter:

```text
"What is happening on the node?"
```

Kube-state-metrics:

```text
"What does Kubernetes think the object state is?"
```

Example:

```text
Node Exporter:
CPU = 80%

Kube-state-metrics:
Deployment desired replicas = 5
Deployment available replicas = 3
```

Both are valuable.

---

# 128. Key Difference: Node Exporter vs Blackbox Exporter

Node Exporter:

```text
Infrastructure health
```

Blackbox Exporter:

```text
Reachability and probe behavior
```

Example:

```text
Node Exporter:
CPU / Memory / Disk

Blackbox:
HTTP status / latency / TLS
```

---

# 129. Key Difference: Exporter vs Instrumentation

Instrumentation:

```text
Application exposes its own metrics.
```

Exporter:

```text
External component exposes metrics about another system.
```

For a Java microservice:

```text
Java Application
   ↓
Micrometer / Prometheus instrumentation
   ↓
/metrics
```

For Linux:

```text
Linux
   ↓
Node Exporter
   ↓
/metrics
```

---

# 130. Interview Answer: What Is a Prometheus Exporter?

A strong answer:

```text
"A Prometheus exporter is a component that exposes metrics from a system in a format that Prometheus can scrape.

I use exporters when the target system does not natively expose Prometheus metrics.

For example, Node Exporter exposes Linux host metrics, MySQL Exporter exposes MySQL metrics, and Blackbox Exporter performs HTTP, TCP, DNS, or ICMP-style probes.

In production, I deploy exporters close to their targets, secure their endpoints, use service discovery to find them, and monitor the exporters themselves through Prometheus."
```

---

# 131. Interview Answer: How Would You Monitor EC2?

```text
"For EC2 instances, I would install Node Exporter to expose host-level CPU, memory, filesystem, disk and network metrics.

I would avoid manually maintaining instance IPs and use EC2 service discovery in Prometheus.

The instances would be secured using private networking and security-group rules that allow Prometheus to access the Node Exporter port.

For Auto Scaling Groups, I would ensure the exporter is installed through a golden AMI, bootstrap process, or configuration management so that new instances automatically become monitorable."
```

---

# 132. Interview Answer: How Would You Monitor Kubernetes Nodes?

```text
"I would deploy Node Exporter as a DaemonSet so that each Kubernetes node gets an exporter instance.

Prometheus would discover those exporter pods through Kubernetes service discovery, typically managed through the Prometheus Operator ecosystem.

I would then build dashboards and alerts for CPU, memory, filesystem, network and node availability."
```

---

# 133. Interview Answer: How Would You Monitor an External Website?

```text
"I would use Blackbox Exporter.

Prometheus would call the Blackbox Exporter probe endpoint, and the exporter would probe the target website.

I would monitor probe_success, response latency, HTTP status and TLS certificate expiry where applicable.

I would alert when the endpoint remains unavailable for an appropriate duration."
```

---

# 134. Interview Answer: How Do You Secure Exporters?

```text
"I don't expose exporter ports publicly.

For EC2, I restrict the exporter security-group access to the Prometheus security group.

For Kubernetes, I use NetworkPolicies where appropriate and expose exporters only to the monitoring components.

If authentication or TLS is required, I configure it according to the exporter capabilities.

Database exporters use dedicated monitoring users with least-privilege permissions, and credentials are stored in a secret-management system rather than Git."
```

---

# 135. Interview Answer: How Do You Deploy Exporters in Production?

```text
"I prefer automation rather than manual installation.

For EC2, I can use Ansible, Packer, or bootstrap automation to install Node Exporter.

For Kubernetes, I use Helm or Kubernetes manifests, preferably managed through GitOps with ArgoCD.

I also standardize resource limits, security configuration, ServiceMonitors, dashboards and alerts.

Before production rollout, I test the exporter version and verify that the expected metrics are available."
```

---

# 136. Interview Answer: What Happens If an Exporter Goes Down?

```text
"If an exporter goes down, Prometheus cannot scrape that target and the corresponding up metric becomes zero.

I would first check the Prometheus Targets page to determine the scrape error.

Then I would check the exporter process, logs, listening port, network connectivity, firewall or NetworkPolicy, and the exporter endpoint itself.

For critical exporters, I would also design appropriate availability and alerting around them."
```

---

# 137. Interview Answer: How Do You Troubleshoot Missing Exporter Metrics?

Use:

```text
Exporter running?
       ↓
Port listening?
       ↓
/metrics works?
       ↓
Prometheus can reach it?
       ↓
Target discovered?
       ↓
Target UP?
       ↓
Metric exists?
       ↓
Metric relabeled/dropped?
       ↓
PromQL correct?
```

This is a strong real-world troubleshooting sequence.

---

# 138. Production Best Practices

## 1. Use Dedicated Users

For system exporters:

```text
node_exporter
```

For database exporters:

```text
monitoring user
```

---

## 2. Never Expose Exporter Ports Publicly

Use:

```text
Private networking
Security Groups
Firewalls
NetworkPolicies
```

---

## 3. Automate Installation

Use:

```text
Ansible
Packer
Terraform
Helm
GitOps
```

---

## 4. Pin Versions

Do not blindly use:

```text
latest
```

for production exporter images.

Use controlled versions and upgrade deliberately.

---

## 5. Monitor Exporters

Monitor:

```text
up
scrape failures
scrape duration
resource usage
```

---

## 6. Control Cardinality

Avoid unnecessary labels and metrics.

---

## 7. Use Service Discovery

Avoid maintaining large static target lists.

---

## 8. Use GitOps for Kubernetes

Keep:

```text
Exporter configuration
ServiceMonitors
PodMonitors
Alerts
Dashboards
```

in version control.

---

# 139. Final Exporter Architecture

```text
                              USERS
                                │
                                ↓
                         Applications
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
          App Metrics      Database           External
              │            Exporters            Endpoints
              │                 │                 │
              │                 │            Blackbox
              │                 │             Exporter
              │                 │                 │
              └─────────────────┼─────────────────┘
                                │
                         Node Exporter
                                │
                                ↓
                       Service Discovery
                                │
                                ↓
                           Prometheus
                                │
                    ┌───────────┼───────────┐
                    ↓           ↓           ↓
                 PromQL      Alerting      TSDB
                    │
                    ↓
                 Grafana
```

The complete mental model is:

```text
Exporter = exposes metrics

Service Discovery = finds targets

Prometheus = scrapes + stores metrics

PromQL = queries metrics

Grafana = visualizes metrics

Alerting = notifies when conditions require action
```

That separation is fundamental to designing a production Prometheus monitoring platform.
