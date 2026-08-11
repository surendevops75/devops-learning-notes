# Logstash Installation

## 1. Overview

This document covers installing Logstash for real-world DevOps environments, including:

* Linux / EC2 installation
* Configuration directories
* Service management
* Initial pipeline validation
* Elasticsearch connectivity
* Security considerations
* Production installation architecture
* Kubernetes / EKS deployment concepts
* Installation troubleshooting

The overall ELK flow is:

```text
Application / Infrastructure
            ↓
       Log Collector
            ↓
         Logstash
            ↓
      Elasticsearch
            ↓
          Kibana
```

---

# 2. Installation Architecture

A basic VM-based installation:

```text
┌──────────────────────────────┐
│          Linux / EC2         │
│                              │
│  ┌────────────────────────┐  │
│  │       Logstash         │  │
│  │                        │  │
│  │ Input → Filter → Output│  │
│  └────────────────────────┘  │
└──────────────────────────────┘
                │
                ↓
        Elasticsearch
```

Production:

```text
                 Log Collectors
                       │
              ┌────────┴────────┐
              ↓                 ↓
          Logstash-01       Logstash-02
              │                 │
              └────────┬────────┘
                       ↓
             Elasticsearch Cluster
```

---

# 3. Installation Prerequisites

Before installing Logstash, determine:

```text
Operating system
Logstash version
CPU
Memory
Disk
Network
Input sources
Output destinations
Expected events/sec
Security requirements
```

For production, also determine:

```text
High availability
Persistent queue requirements
TLS
Authentication
Monitoring
Backup/replay requirements
```

---

# 4. Version Planning

Keep Logstash compatible with the Elasticsearch/Kibana stack.

Example:

```text
Elasticsearch
      ↓
Compatible Version
      ↓
Logstash
      ↓
Compatible Kibana
```

Do not randomly install different major versions across the ELK stack.

Before production deployment, verify the compatibility matrix for the exact versions you intend to use.

---

# 5. Infrastructure Requirements

A development Logstash server might be:

```text
4 vCPU
8 GB RAM
50 GB disk
Private IP
```

This is only an example.

Production sizing depends on:

```text
Events/sec
Average event size
Filter complexity
Grok usage
Output latency
Queue requirements
```

---

# 6. Network Requirements

A typical architecture is:

```text
Log Collector
      ↓
   Logstash
      ↓
Elasticsearch
```

Therefore Logstash requires connectivity to:

```text
Input source
Elasticsearch
Optional Kafka
Optional monitoring systems
```

---

# 7. Common Ports

Depending on the inputs configured, Logstash may listen on different ports.

Example:

```text
5044
   ↓
Beats input

5000
   ↓
TCP / UDP example

8080
   ↓
HTTP input example

9600
   ↓
Logstash monitoring API
```

Do not open every possible port.

Only expose ports required by your actual pipeline.

---

# 8. Security Group Example

For AWS:

```text
                    AWS VPC
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       Log Collector          Logstash
                                 │
                                 ↓
                          Elasticsearch
```

Security groups should allow only the required traffic.

Example:

```text
Collector SG
     ↓
Logstash SG : 5044

Logstash SG
     ↓
Elasticsearch SG : 9200
```

Avoid:

```text
0.0.0.0/0 → Logstash
```

unless there is a specific secured architecture requiring it.

---

# 9. Linux Preparation

Before installing:

```bash
sudo dnf update -y
```

or on Debian/Ubuntu:

```bash
sudo apt update
```

Check OS:

```bash
cat /etc/os-release
```

Check CPU:

```bash
nproc
```

Check memory:

```bash
free -h
```

Check disk:

```bash
df -h
```

---

# 10. Set Hostname

Give the Logstash server a meaningful hostname.

Example:

```bash
sudo hostnamectl set-hostname logstash-prod-01
```

Verify:

```bash
hostname
```

For multiple nodes:

```text
logstash-prod-01
logstash-prod-02
logstash-prod-03
```

---

# 11. Time Synchronization

Distributed logging systems depend on consistent timestamps.

Check:

```bash
timedatectl
```

Make sure the server's clock is synchronized.

This is important when correlating:

```text
Application logs
Logstash logs
Elasticsearch events
Prometheus metrics
Distributed traces
```

---

# 12. Storage Planning

Logstash may need disk for:

```text
Persistent queues
Dead letter queues
Temporary data
Application logs
Operating-system logs
```

If persistent queues are enabled:

```text
Logstash
   ↓
Persistent Queue
   ↓
Disk
```

The disk must be sized according to expected queue volume and outage duration.

---

# 13. Repository Installation

The recommended package installation process is:

```text
Official Elastic Repository
          ↓
Package Manager
          ↓
Logstash
```

Use the official Elastic repository appropriate for the Logstash version.

Avoid installing Logstash packages from random third-party repositories.

---

# 14. RHEL / Amazon Linux Installation

For RPM-based systems, the workflow is:

```text
Configure Elastic repository
          ↓
Refresh package metadata
          ↓
Install Logstash
          ↓
Configure
          ↓
Validate
          ↓
Start service
```

Example:

```bash
sudo dnf install logstash
```

The repository must be configured before this command.

---

# 15. Ubuntu / Debian Installation

For Debian/Ubuntu:

```bash
sudo apt update
```

Then install:

```bash
sudo apt install logstash
```

Again, use the official Elastic repository matching your selected version.

---

# 16. Verify Installation

On RPM systems:

```bash
rpm -qa | grep logstash
```

On Debian/Ubuntu:

```bash
dpkg -l | grep logstash
```

Then:

```bash
systemctl status logstash
```

---

# 17. Logstash Installation Directory

Package installations commonly place Logstash under:

```text
/usr/share/logstash/
```

This directory contains the Logstash installation and executable components.

Example:

```text
/usr/share/logstash/
├── bin/
├── config/
├── data/
├── lib/
└── vendor/
```

The exact layout can vary by version and packaging.

---

# 18. Logstash Configuration Directory

A common package configuration directory is:

```text
/etc/logstash/
```

Typical files include:

```text
logstash.yml
pipelines.yml
jvm.options
```

and:

```text
conf.d/
```

for pipeline configurations.

---

# 19. Recommended Configuration Structure

A clean structure:

```text
/etc/logstash/
│
├── logstash.yml
├── pipelines.yml
├── jvm.options
│
└── conf.d/
    ├── application/
    │   ├── input.conf
    │   ├── filter.conf
    │   └── output.conf
    │
    ├── infrastructure/
    │   ├── input.conf
    │   ├── filter.conf
    │   └── output.conf
    │
    └── security/
        ├── input.conf
        ├── filter.conf
        └── output.conf
```

This is easier to maintain than one enormous pipeline file.

---

# 20. Main Logstash Configuration

The main configuration file is commonly:

```text
/etc/logstash/logstash.yml
```

It can contain settings related to:

```text
Node identity
Pipeline management
API
Logging
Monitoring
Queue behavior
```

---

# 21. Node Name

A meaningful Logstash node name is useful in production.

Example:

```yaml
node.name: logstash-prod-01
```

Another node:

```yaml
node.name: logstash-prod-02
```

This helps identify nodes during troubleshooting.

---

# 22. API Configuration

Logstash exposes an HTTP API for operational information.

A commonly used port is:

```text
9600
```

Example:

```bash
curl http://localhost:9600
```

This can provide information about:

```text
Node
Pipelines
Events
Plugins
```

Restrict access to this API in production.

---

# 23. Pipeline Configuration

Pipeline definitions can be managed through:

```text
/etc/logstash/pipelines.yml
```

Example:

```yaml
- pipeline.id: application
  path.config: "/etc/logstash/conf.d/application/*.conf"

- pipeline.id: infrastructure
  path.config: "/etc/logstash/conf.d/infrastructure/*.conf"
```

This allows separate logical pipelines.

---

# 24. Basic Pipeline

A simple test pipeline:

```ruby
input {
  stdin {}
}

filter {
  mutate {
    add_field => {
      "environment" => "test"
    }
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

Flow:

```text
Terminal
   ↓
stdin
   ↓
mutate
   ↓
rubydebug
```

This is useful for validating the installation before connecting real log sources.

---

# 25. Running the Test Pipeline

The Logstash executable is commonly located under:

```text
/usr/share/logstash/bin/logstash
```

You can run a test configuration from the command line.

Example:

```bash
sudo /usr/share/logstash/bin/logstash \
  -f /etc/logstash/conf.d/test.conf
```

Then enter:

```text
hello
```

You should see the processed event.

---

# 26. Configuration Validation

Before starting production pipelines, validate the configuration.

A common validation command is:

```bash
sudo /usr/share/logstash/bin/logstash \
  --config.test_and_exit \
  -f /etc/logstash/conf.d/
```

The exact command depends on how your pipelines are structured.

The purpose is:

```text
Configuration
      ↓
Syntax validation
      ↓
Exit
```

rather than immediately starting the production pipeline.

---

# 27. Why Configuration Validation Matters

Suppose you deploy:

```text
Grok pattern
   ↓
Syntax error
```

Without validation:

```text
Production
   ↓
Logstash restart
   ↓
Pipeline fails
```

With validation:

```text
Git
 ↓
Validation
 ↓
Error detected
 ↓
Deployment blocked
```

This is especially important in CI/CD and GitOps environments.

---

# 28. Systemd Service

After package installation, manage Logstash through systemd.

Check:

```bash
sudo systemctl status logstash
```

Start:

```bash
sudo systemctl start logstash
```

Stop:

```bash
sudo systemctl stop logstash
```

Restart:

```bash
sudo systemctl restart logstash
```

---

# 29. Enable Logstash at Boot

Use:

```bash
sudo systemctl enable logstash
```

Verify:

```bash
systemctl is-enabled logstash
```

Expected:

```text
enabled
```

---

# 30. Check Service Logs

If Logstash fails:

```bash
sudo journalctl -u logstash
```

Recent logs:

```bash
sudo journalctl -u logstash -n 100
```

Follow logs:

```bash
sudo journalctl -u logstash -f
```

This should be your first troubleshooting step when the service does not start.

---

# 31. Logstash Application Logs

Package installations commonly store Logstash logs under:

```text
/var/log/logstash/
```

Check:

```bash
sudo ls -l /var/log/logstash/
```

These logs can reveal:

```text
Pipeline errors
Plugin errors
Elasticsearch connection failures
Configuration errors
JVM problems
```

---

# 32. Verify the API

After starting Logstash:

```bash
curl http://localhost:9600
```

You should receive JSON containing node information.

You can also inspect pipeline information through the monitoring API.

---

# 33. Check Listening Ports

Use:

```bash
sudo ss -lntp
```

For example:

```bash
sudo ss -lntp | grep 9600
```

If using Beats input:

```bash
sudo ss -lntp | grep 5044
```

This confirms the process is listening on the expected port.

---

# 34. Basic Installation Validation

Run:

```text
systemctl status logstash
        ↓
API check
        ↓
Port check
        ↓
Test pipeline
        ↓
Elasticsearch connectivity
```

Do not consider installation complete simply because the service is running.

---

# 35. Install and Connect to Elasticsearch

The next validation is:

```text
Logstash
   ↓
Elasticsearch
```

Verify network connectivity from the Logstash host.

Example:

```bash
curl https://elasticsearch.example.internal:9200
```

If security is enabled, use the appropriate authentication and CA configuration.

---

# 36. DNS Validation

Check Elasticsearch DNS:

```bash
getent hosts elasticsearch.example.internal
```

or:

```bash
nslookup elasticsearch.example.internal
```

Make sure the hostname resolves to the intended private address.

---

# 37. Port Connectivity

Test:

```bash
nc -zv elasticsearch.example.internal 9200
```

Possible outcomes:

```text
Connection succeeded
```

or:

```text
Connection refused
```

or:

```text
Connection timed out
```

These indicate different network problems.

---

# 38. Connection Refused

If:

```text
Connection refused
```

check:

```text
Elasticsearch service
Port 9200
Elasticsearch binding
Security configuration
```

---

# 39. Connection Timeout

If:

```text
Connection timed out
```

check:

```text
AWS Security Groups
Network ACLs
Routing
Firewall
Network Policy
Private subnet connectivity
```

A timeout is often a network-path problem.

---

# 40. TLS Connection

If Elasticsearch uses HTTPS:

```text
Logstash
   ↓
HTTPS/TLS
   ↓
Elasticsearch
```

Do not configure Logstash to use plain HTTP against a TLS-only Elasticsearch endpoint.

---

# 41. Elasticsearch Output Configuration

Example:

```ruby
output {
  elasticsearch {
    hosts => ["https://elasticsearch.example.internal:9200"]
    index => "application-logs"
  }
}
```

Production configuration also needs appropriate authentication and certificate trust.

---

# 42. Authentication

Logstash needs permission to write to Elasticsearch.

Possible mechanisms include:

```text
API key
Username/password
Other supported authentication mechanisms
```

Use a dedicated Logstash identity.

Do not use the Elasticsearch superuser for routine ingestion.

---

# 43. Least Privilege

A Logstash identity should have only the permissions required to:

```text
Write logs
Create/manage required indices or data streams
Use required templates/policies
```

Avoid:

```text
Full cluster administrator privileges
```

unless there is a specific justified operational requirement.

---

# 44. Secret Management

Do not write:

```ruby
password => "MySecretPassword"
```

into a Git repository.

Instead use:

```text
Secret manager
Secure keystore
Kubernetes Secret
External Secrets
Environment-specific secret injection
```

depending on your deployment.

---

# 45. Logstash Keystore

Logstash provides a keystore mechanism for sensitive values.

Conceptually:

```text
Secret
   ↓
Logstash Keystore
   ↓
Pipeline
   ↓
Elasticsearch
```

This allows credentials to be separated from ordinary pipeline configuration.

---

# 46. Example Secure Output

Conceptually:

```ruby
output {
  elasticsearch {
    hosts => ["https://elasticsearch.example.internal:9200"]
    api_key => "${ELASTIC_API_KEY}"
  }
}
```

The actual secret should be supplied securely rather than committed to Git.

---

# 47. Test Pipeline to Elasticsearch

A simple test:

```ruby
input {
  stdin {}
}

output {
  elasticsearch {
    hosts => ["https://elasticsearch.example.internal:9200"]
    index => "logstash-installation-test"
  }

  stdout {
    codec => rubydebug
  }
}
```

Enter:

```text
Logstash installation test
```

Then verify the index in Elasticsearch.

---

# 48. Verify Test Index

From Elasticsearch:

```bash
curl "https://elasticsearch.example.internal:9200/_cat/indices?v"
```

Look for:

```text
logstash-installation-test
```

Then search:

```bash
curl "https://elasticsearch.example.internal:9200/logstash-installation-test/_search"
```

---

# 49. Remove Test Data

After validation:

```bash
curl -X DELETE \
  "https://elasticsearch.example.internal:9200/logstash-installation-test"
```

Only execute deletion commands after verifying the target index.

Never run destructive commands against production indexes blindly.

---

# 50. Install Logstash with Beats Input

For a common logging architecture:

```text
Filebeat / Fluent Bit
          ↓
       Logstash
          ↓
    Elasticsearch
```

Example:

```ruby
input {
  beats {
    port => 5044
  }
}
```

Then configure the collector to send logs to the Logstash private address.

---

# 51. Verify Beats Port

After starting the pipeline:

```bash
sudo ss -lntp | grep 5044
```

Expected:

```text
LISTEN
  ↓
5044
  ↓
Logstash
```

---

# 52. Collector Connectivity

From the collector host:

```bash
nc -zv logstash.example.internal 5044
```

If successful:

```text
Collector
   ↓
Network
   ↓
Logstash
```

is reachable.

---

# 53. TLS Between Collector and Logstash

Production architecture should preferably secure collector-to-Logstash traffic:

```text
Collector
   ↓
TLS
   ↓
Logstash
```

The exact TLS configuration depends on the input plugin and deployment.

---

# 54. Production Input Architecture

```text
                   EKS Nodes
                       │
                 Fluent Bit
                       │
                       ↓
                TLS / Network
                       │
              ┌────────┴────────┐
              ↓                 ↓
          Logstash-01       Logstash-02
              │                 │
              └────────┬────────┘
                       ↓
              Elasticsearch
```

---

# 55. Logstash Installation on Multiple Nodes

For HA:

```text
Install Logstash
      ↓
Configure identical pipeline logic
      ↓
Configure node-specific identity
      ↓
Configure TLS
      ↓
Configure Elasticsearch access
      ↓
Start
      ↓
Validate
```

Each node should be independently testable.

---

# 56. High Availability

A single Logstash server:

```text
Collector
   ↓
Logstash
   ↓
Elasticsearch
```

has a single point of failure.

Production:

```text
             Collector
                 │
        ┌────────┴────────┐
        ↓                 ↓
   Logstash-01       Logstash-02
        │                 │
        └────────┬────────┘
                 ↓
          Elasticsearch
```

---

# 57. Load Balancing

A load balancer can distribute collector traffic:

```text
                  Collector
                      │
                      ↓
                 Internal LB
                 /          \
                ↓            ↓
             LS-01         LS-02
                \            /
                 \          /
                  ↓        ↓
                Elasticsearch
```

Use an internal load balancer for private logging infrastructure where appropriate.

---

# 58. Logstash on EKS

Logstash can also be deployed in Kubernetes.

Conceptually:

```text
EKS
 │
 ├── Logstash Pod
 ├── Logstash Pod
 └── Logstash Pod
```

But Logstash is stateful when persistent queues are enabled, so storage and scheduling need careful planning.

---

# 59. Kubernetes Deployment Options

Possible approaches include:

```text
Deployment
StatefulSet
Helm
Operator
GitOps
```

The appropriate method depends on the pipeline architecture and persistence requirements.

---

# 60. Logstash Kubernetes Architecture

```text
                     EKS
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      Node-1        Node-2        Node-3
        │             │             │
    Collector      Collector      Collector
        │             │             │
        └─────────────┼─────────────┘
                      ↓
                 Logstash
                /         \
               ↓           ↓
            LS Pod       LS Pod
               \           /
                \         /
                 ↓       ↓
              Elasticsearch
```

---

# 61. Persistent Queue on EKS

If persistent queues are enabled:

```text
Logstash Pod
     ↓
PVC
     ↓
Persistent Storage
```

For multiple replicas:

```text
LS-01 → PVC-01
LS-02 → PVC-02
```

Do not assume multiple Pods can safely share one writable queue volume.

---

# 62. Pod Scheduling

For production, avoid placing all Logstash replicas on one worker node.

Example:

```text
Worker-01
 └── Logstash-01

Worker-02
 └── Logstash-02

Worker-03
 └── Logstash-03
```

Use:

```text
Pod anti-affinity
Topology spread constraints
Node affinity
```

as appropriate.

---

# 63. EKS Resource Requests

Define resources for Logstash Pods.

Conceptually:

```yaml
resources:
  requests:
    cpu: ...
    memory: ...
  limits:
    cpu: ...
    memory: ...
```

Sizing depends on:

```text
Event rate
Filter complexity
Workers
Batch size
Memory requirements
```

Do not use arbitrary values.

---

# 64. Logstash JVM in Kubernetes

Logstash runs on the JVM.

Therefore:

```text
Container Memory
       │
       ├── JVM Heap
       ├── Native Memory
       └── Other Process Memory
```

Do not allocate the entire container memory limit to the JVM heap.

Leave headroom for the process and operating system/container requirements.

---

# 65. Kubernetes Service

Collectors can send traffic to a Kubernetes Service:

```text
Fluent Bit
    ↓
Logstash Service
    ↓
Logstash Pods
```

Example DNS concept:

```text
logstash.logging.svc.cluster.local
```

The exact service name depends on the deployment.

---

# 66. Logstash Configuration With GitOps

For your environment:

```text
GitHub
   ↓
Logstash Helm / Kubernetes Configuration
   ↓
Pull Request
   ↓
GitHub Actions
   ↓
Validation + Security
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Logstash
```

This is the preferred operational model for a GitOps-managed Kubernetes environment.

---

# 67. Suggested Repository Structure

A deployment repository could use:

```text
observability/
│
├── logstash/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   └── rbac.yaml
│   │
│   └── overlays/
│       ├── dev/
│       ├── staging/
│       └── prod/
```

If using Helm:

```text
logstash/
├── Chart.yaml
├── values.yaml
└── templates/
```

---

# 68. Configuration Separation

Keep separate:

```text
Application Pipeline
Infrastructure Pipeline
Security Pipeline
```

Example:

```text
prod/
├── application.conf
├── infrastructure.conf
└── security.conf
```

This makes changes easier to review.

---

# 69. Configuration as Code

Your configuration should be:

```text
Version controlled
Peer reviewed
Validated
Security scanned
Tested
Deployable
Rollback capable
```

Avoid:

```text
SSH → Production → Edit file manually
```

---

# 70. CI/CD Validation

Your GitHub Actions workflow can perform:

```text
Pull Request
    ↓
YAML validation
    ↓
Logstash configuration validation
    ↓
Security checks
    ↓
Review
    ↓
Merge
```

Then ArgoCD deploys to EKS.

---

# 71. Production Installation Flow

A complete VM-based flow:

```text
Terraform
   ↓
EC2
   ↓
Security Group
   ↓
EBS
   ↓
OS Preparation
   ↓
Elastic Repository
   ↓
Logstash Installation
   ↓
Configuration
   ↓
TLS / Credentials
   ↓
Pipeline Validation
   ↓
Systemd
   ↓
Monitoring
```

---

# 72. Production EKS Flow

For your Kubernetes architecture:

```text
Terraform
   ↓
EKS
   ↓
Storage / Networking
   ↓
GitHub
   ↓
Logstash Configuration
   ↓
GitHub Actions
   ↓
ArgoCD
   ↓
Logstash Deployment
   ↓
Persistent Queue if required
   ↓
Monitoring
```

---

# 73. Installation Validation Checklist

```text
[ ] Correct version
[ ] Official repository
[ ] Package installed
[ ] Service enabled
[ ] Service running
[ ] Configuration directory verified
[ ] Pipeline configuration verified
[ ] API accessible internally
[ ] Required input ports listening
[ ] Elasticsearch reachable
[ ] TLS working
[ ] Authentication working
[ ] Test event processed
[ ] Test event indexed
[ ] Test data removed
```

---

# 74. Security Checklist

```text
[ ] Private network
[ ] No unnecessary public exposure
[ ] TLS
[ ] Authentication
[ ] Least privilege
[ ] API protected
[ ] Secrets not in Git
[ ] Sensitive logs filtered
[ ] Security groups restricted
[ ] Kubernetes NetworkPolicies where appropriate
```

---

# 75. Production Reliability Checklist

```text
[ ] Multiple Logstash instances
[ ] Load balancing
[ ] Persistent queue if required
[ ] Adequate disk
[ ] Elasticsearch HA
[ ] Monitoring
[ ] Alerting
[ ] Capacity planning
[ ] Backpressure handling
[ ] Failure testing
```

---

# 76. Installation Troubleshooting: Service Fails

Start with:

```bash
sudo systemctl status logstash
```

Then:

```bash
sudo journalctl -u logstash -n 100
```

Look for:

```text
Configuration error
Plugin error
Permission error
JVM error
Port conflict
Elasticsearch connection error
```

---

# 77. Troubleshooting: Configuration Error

Validate:

```bash
sudo /usr/share/logstash/bin/logstash \
  --config.test_and_exit \
  -f /etc/logstash/conf.d/
```

Look for:

```text
Expected one of
Unknown setting
Invalid plugin
Syntax error
```

Fix the configuration before restarting production.

---

# 78. Troubleshooting: Port Already in Use

Check:

```bash
sudo ss -lntp | grep 5044
```

If another process is using the port:

```text
Logstash
   ↓
Cannot bind port
```

Identify the process:

```bash
sudo lsof -i :5044
```

Then correct the port conflict.

---

# 79. Troubleshooting: Elasticsearch Unreachable

Check:

```bash
getent hosts elasticsearch.example.internal
```

Then:

```bash
nc -zv elasticsearch.example.internal 9200
```

Then:

```bash
curl https://elasticsearch.example.internal:9200
```

If HTTPS is configured, validate TLS and authentication.

---

# 80. Troubleshooting: Authentication Failure

Symptoms may include:

```text
401 Unauthorized
403 Forbidden
```

Interpretation:

```text
401
 ↓
Authentication problem

403
 ↓
Authenticated but insufficient permissions
```

Check:

```text
API key
Username/password
Role
Index privileges
Elasticsearch security
```

---

# 81. Troubleshooting: TLS Failure

Common causes:

```text
Wrong CA
Expired certificate
Hostname mismatch
Wrong certificate
Incorrect trust configuration
Protocol mismatch
```

Do not solve TLS problems by disabling certificate validation in production.

---

# 82. Troubleshooting: No Events

Check:

```text
Collector
   ↓
Network
   ↓
Input
   ↓
Pipeline
   ↓
Filter
   ↓
Output
   ↓
Elasticsearch
```

At each stage verify whether events exist.

---

# 83. Troubleshooting: Events Enter but Don't Leave

Check:

```text
Filter failures
Output failures
Elasticsearch connectivity
Authentication
Queue
Pipeline workers
```

The Logstash monitoring API can help identify event counts.

---

# 84. Troubleshooting: Queue Growing

If:

```text
Queue size ↑
```

check:

```text
Elasticsearch latency
Elasticsearch health
Output errors
Input rate
CPU
Memory
Workers
```

A growing queue is usually a symptom, not the root cause.

---

# 85. Troubleshooting: CPU High

Possible causes:

```text
Complex Grok
Ruby filters
High event rate
Large events
Too many pipelines
Too many workers
GeoIP processing
```

Solutions may include:

```text
Structured logging
Dissect
Simpler filters
Horizontal scaling
Pipeline redesign
```

---

# 86. Troubleshooting: Memory High

Check:

```text
JVM heap
Event size
Batch size
Persistent queue
Pipeline count
Input rate
```

Do not simply increase memory without understanding the workload.

---

# 87. Troubleshooting: Disk Full

Check:

```bash
df -h
```

Then inspect:

```text
Persistent queue
Logstash logs
Dead letter queue
Temporary data
```

If persistent queues are filling, investigate the downstream Elasticsearch bottleneck.

---

# 88. Production Monitoring

Monitor Logstash using your observability stack:

```text
Logstash
   ↓
Metrics
   ↓
Prometheus
   ↓
Grafana
```

Track:

```text
Events in
Events out
Pipeline health
Queue size
CPU
Memory
JVM
Output failures
```

---

# 89. Alerting

Useful alerts:

```text
Logstash down
Pipeline stopped
Events out = 0
Queue > threshold
Persistent queue disk high
Elasticsearch output failures
CPU high
Memory high
```

These alerts help detect logging pipeline failures before they become blind spots.

---

# 90. Production Installation Architecture

```text
                         EKS
                          │
                Application Pods
                          │
                          ↓
                    Fluent Bit
                          │
                    TLS / Network
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
        Logstash-01              Logstash-02
              │                       │
              ↓                       ↓
         Persistent Q             Persistent Q
              │                       │
              └───────────┬───────────┘
                          ↓
                Elasticsearch Cluster
                 ┌────────┼────────┐
                 ↓        ↓        ↓
                ES-01   ES-02    ES-03
                          │
                          ↓
                       Kibana
```

---

# 91. Installation Strategy for Your Environment

For your DevOps notes, remember this production flow:

```text
Terraform
   ↓
AWS Infrastructure
   ↓
EKS
   ↓
GitHub
   ↓
Logstash Configuration
   ↓
GitHub Actions
   ↓
Security + Validation
   ↓
ArgoCD
   ↓
EKS
   ↓
Logstash
   ↓
Elasticsearch
   ↓
Kibana
```

This connects the ELK installation to the GitHub Actions and GitOps topics you already completed.

---

# 92. Final Installation Checklist

```text
Infrastructure
[ ] CPU
[ ] Memory
[ ] Disk
[ ] Network
[ ] Security groups

Installation
[ ] Official repository
[ ] Correct version
[ ] Package installed
[ ] Service enabled

Configuration
[ ] logstash.yml
[ ] pipelines.yml
[ ] Pipeline files
[ ] Input
[ ] Filter
[ ] Output

Connectivity
[ ] Collector → Logstash
[ ] Logstash → Elasticsearch
[ ] DNS
[ ] Ports
[ ] TLS

Security
[ ] Authentication
[ ] Authorization
[ ] API key / credentials
[ ] Secrets protected
[ ] No public exposure

Reliability
[ ] Multiple instances
[ ] Load balancing
[ ] Persistent queue if required
[ ] Disk sizing
[ ] Backpressure handling

Validation
[ ] Service running
[ ] API working
[ ] Test event received
[ ] Test event processed
[ ] Test event indexed
[ ] Kibana search verified

Operations
[ ] Prometheus monitoring
[ ] Grafana dashboards
[ ] Alerts
[ ] Logs
[ ] Capacity planning
```

---

# 93. Final Mental Model

The Logstash installation process should be remembered as:

```text
                 LOGSTASH INSTALLATION
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
   Infrastructure     Installation      Configuration
        │                 │                 │
     CPU/RAM          Repository       logstash.yml
     Storage          Package          pipelines.yml
     Network          Service          Pipeline configs
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                     Connectivity
                          │
                ┌─────────┴─────────┐
                ↓                   ↓
             Inputs             Elasticsearch
                │                   │
                └─────────┬─────────┘
                          ↓
                       Security
                          ↓
                    TLS + Auth
                          ↓
                     Validation
                          ↓
                     Monitoring
                          ↓
                    Production HA
```

The key principle is:

**Installing Logstash in production is not just installing the package. You must prepare the infrastructure, configure pipelines, secure the connections, validate the input and output paths, plan buffering and backpressure, provide high availability where required, and integrate the installation with your GitHub Actions + ArgoCD workflow.**
