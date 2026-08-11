# Kibana Installation

## 1. Overview

Kibana is the visualization and investigation interface for Elasticsearch.

The installation architecture is:

```text
Applications
     ↓
Log Collector
     ↓
Logstash
     ↓
Elasticsearch
     ↓
Kibana
     ↓
Users / DevOps / Security Teams
```

This document covers installing Kibana in:

* Linux / EC2 environments
* Production AWS environments
* Kubernetes / EKS environments
* High-availability architectures
* GitOps-managed environments

The focus is on **real-world installation**, not just running Kibana locally.

---

# 2. Kibana Installation Architecture

A basic VM-based architecture:

```text
┌──────────────────────────────┐
│          Linux / EC2         │
│                              │
│       ┌──────────────┐       │
│       │    Kibana    │       │
│       │    :5601     │       │
│       └──────┬───────┘       │
└──────────────┼───────────────┘
               │
               ↓
       Elasticsearch
```

Production:

```text
                    Users
                      │
                      ↓
              Internal ALB
                      │
              ┌───────┴───────┐
              ↓               ↓
          Kibana-01       Kibana-02
              │               │
              └───────┬───────┘
                      ↓
             Elasticsearch
                Cluster
```

---

# 3. Installation Prerequisites

Before installing Kibana, determine:

```text
Elasticsearch version
Kibana version
Operating system
CPU
Memory
Disk
Network
DNS
TLS requirements
Authentication
Number of users
High availability requirements
```

The Kibana version should be compatible with the Elasticsearch version.

For production, avoid independently upgrading Kibana without considering the Elasticsearch cluster.

---

# 4. Infrastructure Requirements

A small development Kibana server could use:

```text
4 vCPU
8 GB RAM
50 GB disk
Private network
```

These are example values only.

Production sizing depends on:

```text
Concurrent users
Dashboard complexity
Query volume
Number of visualizations
Elasticsearch response time
Saved searches
Alerting workload
```

---

# 5. Network Architecture

Kibana generally requires connectivity to Elasticsearch.

```text
Kibana
   │
   ↓
Elasticsearch :9200
```

Users require access to Kibana:

```text
User
  ↓
Kibana :5601
```

A production architecture should keep Elasticsearch private.

```text
Users
  ↓
Internal ALB
  ↓
Kibana
  ↓
Private Elasticsearch
```

---

# 6. AWS Security Groups

For an AWS deployment:

```text
User / Corporate Network
          ↓
       ALB SG
          ↓
       Kibana SG
          ↓
 Elasticsearch SG
```

Example:

```text
ALB SG
   ↓
Kibana SG : 5601

Kibana SG
   ↓
Elasticsearch SG : 9200
```

Avoid exposing Elasticsearch directly to users.

---

# 7. DNS Planning

Use a meaningful hostname.

Example:

```text
kibana.prod.internal
```

Architecture:

```text
User
 ↓
Route53
 ↓
Internal ALB
 ↓
Kibana
```

For a simple EC2 deployment:

```text
kibana-prod-01.internal
```

can resolve directly to the private IP.

---

# 8. Time Synchronization

Before installation:

```bash
timedatectl
```

Make sure the system clock is synchronized.

Consistent time is important for:

```text
Logs
Dashboards
Alerts
Elasticsearch
Kibana
Incident investigation
```

---

# 9. Linux Preparation

For RPM-based systems:

```bash
sudo dnf update -y
```

For Debian/Ubuntu:

```bash
sudo apt update
```

Check the operating system:

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

Set a meaningful hostname:

```bash
sudo hostnamectl set-hostname kibana-prod-01
```

Verify:

```bash
hostname
```

For HA:

```text
kibana-prod-01
kibana-prod-02
```

---

# 11. Configure Elastic Repository

Kibana should be installed from the official Elastic package repository appropriate for the version you are deploying.

The repository should match the intended Elastic Stack release.

Architecture:

```text
Official Elastic Repository
          ↓
Package Manager
          ↓
Kibana
```

Do not mix unrelated major versions.

---

# 12. RPM-Based Installation

After configuring the appropriate Elastic repository:

```bash
sudo dnf install kibana
```

Verify:

```bash
rpm -qa | grep kibana
```

You should see the installed Kibana package.

---

# 13. Debian / Ubuntu Installation

Update package metadata:

```bash
sudo apt update
```

Install Kibana:

```bash
sudo apt install kibana
```

Verify:

```bash
dpkg -l | grep kibana
```

---

# 14. Verify Kibana Binary

Check the installed binary:

```bash
/usr/share/kibana/bin/kibana --version
```

This confirms the installed version.

Record the version as part of your production deployment information.

---

# 15. Kibana Directory Structure

A package installation commonly uses:

```text
/usr/share/kibana/
```

Example:

```text
/usr/share/kibana/
├── bin/
├── config/
├── node/
├── plugins/
└── src/
```

The exact internal structure can vary by version.

---

# 16. Kibana Configuration Directory

The main configuration directory is commonly:

```text
/etc/kibana/
```

The primary configuration file is:

```text
/etc/kibana/kibana.yml
```

This file contains the important server and Elasticsearch connection settings.

---

# 17. Kibana Configuration File

Open:

```bash
sudo vi /etc/kibana/kibana.yml
```

Important configuration areas include:

```text
Server
Elasticsearch
Security
TLS
Logging
Monitoring
```

---

# 18. Configure Server Host

For local-only testing:

```yaml
server.host: "127.0.0.1"
```

For a server that must accept traffic from an internal network:

```yaml
server.host: "0.0.0.0"
```

However:

```text
0.0.0.0
```

does not mean Kibana should be publicly exposed.

Use security groups, firewall rules, load balancers, and authentication.

---

# 19. Configure Server Port

Default Kibana port:

```yaml
server.port: 5601
```

Architecture:

```text
Client
  ↓
TCP 5601
  ↓
Kibana
```

If using an ALB, the ALB can forward traffic to the Kibana service/instance on this port.

---

# 20. Configure Elasticsearch

A basic configuration:

```yaml
elasticsearch.hosts:
  - "https://elasticsearch.internal:9200"
```

For a cluster:

```yaml
elasticsearch.hosts:
  - "https://es-01.internal:9200"
  - "https://es-02.internal:9200"
  - "https://es-03.internal:9200"
```

Use the topology and connection settings appropriate for your Elasticsearch deployment.

---

# 21. Verify Elasticsearch DNS

From the Kibana server:

```bash
getent hosts es-01.internal
```

You can also use:

```bash
nslookup es-01.internal
```

Verify that the hostname resolves to the intended private address.

---

# 22. Test Elasticsearch Port

Check connectivity:

```bash
nc -zv es-01.internal 9200
```

A successful result indicates that the network path is reachable.

If it times out, investigate:

```text
Security Groups
Network ACLs
Routing
Firewall
DNS
Network Policies
```

---

# 23. Test Elasticsearch API

If Elasticsearch uses HTTPS:

```bash
curl https://es-01.internal:9200
```

If authentication is required, use the appropriate credentials or authentication mechanism.

A response confirms that the endpoint is reachable.

---

# 24. TLS Architecture

Production should use secure communication:

```text
User
 ↓
HTTPS
 ↓
Kibana
 ↓
HTTPS
 ↓
Elasticsearch
```

This protects:

```text
Credentials
Sessions
Queries
Log data
Dashboard information
```

---

# 25. Kibana HTTPS

Kibana can terminate HTTPS itself.

Conceptually:

```yaml
server.ssl.enabled: true
```

The actual certificate and key configuration must match your certificate-management approach and Kibana version.

---

# 26. Certificate Requirements

The Kibana certificate should contain the hostname users access.

For example:

```text
kibana.prod.internal
```

If users access:

```text
https://kibana.prod.internal
```

the certificate must be valid for that hostname.

---

# 27. Certificate Management

A production certificate architecture:

```text
Certificate Authority
        │
        ↓
Kibana Certificate
        │
        ↓
Kibana HTTPS
```

Certificates should be:

```text
Valid
Trusted
Not expired
Properly named
Stored securely
```

---

# 28. Reverse Proxy Architecture

Instead of exposing Kibana directly:

```text
User
 ↓
Internal ALB
 ↓
Kibana
 ↓
Elasticsearch
```

The ALB can provide:

```text
TLS termination
DNS integration
Load balancing
Network boundary
```

Traffic from the ALB to Kibana should also be protected when required by the security model.

---

# 29. AWS ALB Architecture

For your AWS environment:

```text
Corporate User
      ↓
Route53
      ↓
Internal ALB
      ↓
Target Group
      ↓
Kibana
      ↓
Elasticsearch
```

This is a common production pattern for an internal observability platform.

---

# 30. Security Group Example

Example:

```text
Corporate Network
       ↓
ALB Security Group
       ↓
Kibana Security Group
       ↓
Elasticsearch Security Group
```

Rules:

```text
ALB SG
  → Kibana SG : 5601

Kibana SG
  → Elasticsearch SG : 9200
```

Only required sources should be permitted.

---

# 31. Start Kibana

After configuration:

```bash
sudo systemctl start kibana
```

Check:

```bash
sudo systemctl status kibana
```

If successful:

```text
Active: active (running)
```

---

# 32. Enable Kibana at Boot

Use:

```bash
sudo systemctl enable kibana
```

Verify:

```bash
systemctl is-enabled kibana
```

Expected:

```text
enabled
```

---

# 33. Restart Kibana

Whenever configuration changes:

```bash
sudo systemctl restart kibana
```

Then immediately verify:

```bash
sudo systemctl status kibana
```

Do not assume a restart succeeded simply because the command returned successfully.

---

# 34. Check Kibana Port

Use:

```bash
sudo ss -lntp | grep 5601
```

Expected:

```text
LISTEN
   ↓
5601
   ↓
Kibana
```

If nothing is listening, inspect Kibana logs.

---

# 35. Check Kibana Logs

Use:

```bash
sudo journalctl -u kibana
```

Recent logs:

```bash
sudo journalctl -u kibana -n 100
```

Follow logs:

```bash
sudo journalctl -u kibana -f
```

Look for:

```text
Configuration errors
Elasticsearch connection failures
TLS errors
Authentication failures
Port conflicts
Plugin failures
```

---

# 36. Check Kibana Log Files

Depending on the package configuration, Kibana logs may also exist under:

```text
/var/log/kibana/
```

Check:

```bash
sudo ls -la /var/log/kibana/
```

Systemd logs remain useful even when file-based logging is not configured.

---

# 37. Local Health Check

From the Kibana server:

```bash
curl http://localhost:5601
```

If HTTPS is configured:

```bash
curl https://localhost:5601
```

Certificate validation may need to be handled correctly rather than bypassed.

---

# 38. Remote Health Check

From a permitted client:

```bash
curl https://kibana.prod.internal
```

If this fails:

```text
DNS
 ↓
ALB
 ↓
Security Group
 ↓
Target
 ↓
Kibana
```

must be checked.

---

# 39. Browser Validation

Open the configured Kibana URL:

```text
https://kibana.prod.internal
```

Verify:

```text
Kibana login page
        ↓
Authentication
        ↓
Kibana home page
```

Do not use a public IP when the production architecture is intended to be private.

---

# 40. Elasticsearch Dependency

Kibana can start but still fail to operate correctly if Elasticsearch is unavailable.

Architecture:

```text
Kibana
  │
  X
  ↓
Elasticsearch
```

Therefore always verify:

```text
Kibana → Elasticsearch
```

not just:

```text
Browser → Kibana
```

---

# 41. Elasticsearch Authentication

Kibana requires an appropriate Elasticsearch service identity/credential mechanism.

Use the mechanism supported by your deployed Elastic Stack version.

Avoid using:

```text
Elasticsearch superuser
```

as a general-purpose runtime identity.

---

# 42. Least Privilege

The Kibana runtime identity should have only the privileges required for Kibana's operation.

The design should distinguish:

```text
Kibana Server Identity
```

from:

```text
Human User Identity
```

These are different security concerns.

---

# 43. Human User Authentication

Users can authenticate through mechanisms supported by the Elastic Stack deployment.

Examples include:

```text
Username/password
SAML
OIDC
LDAP
Other supported identity providers
```

Enterprise architecture:

```text
User
 ↓
Identity Provider
 ↓
Authentication
 ↓
Kibana
 ↓
Role Mapping
```

---

# 44. Kibana Roles

Example:

```text
Platform Admin
Application Developer
DevOps Engineer
Security Analyst
Read Only
```

Use least privilege.

For example:

```text
Application Team
     ↓
Application logs

Security Team
     ↓
Security logs

Platform Team
     ↓
Platform + application observability
```

---

# 45. Kibana Spaces

Organize teams and dashboards:

```text
Kibana
│
├── Platform
├── Applications
├── Security
└── Production
```

This helps separate operational content.

---

# 46. Data Access

Users should not automatically have access to every Elasticsearch index.

For example:

```text
Developer
   ↓
application-logs-*

Security Analyst
   ↓
security-logs-*

Platform Team
   ↓
application-logs-*
infrastructure-logs-*
kubernetes-logs-*
```

Access should be implemented using the appropriate Elasticsearch/Kibana authorization model.

---

# 47. Kibana Data View

After installation, create a data view for the logs.

Example:

```text
application-logs-*
```

This tells Kibana which Elasticsearch data should be available for exploration.

---

# 48. Time Field

Select the correct timestamp field.

Usually:

```text
@timestamp
```

Architecture:

```text
Application
 ↓
Logstash
 ↓
@timestamp
 ↓
Elasticsearch
 ↓
Kibana
```

Without correct timestamp handling, time-based investigations can become misleading.

---

# 49. First Validation Search

After logs are available:

```text
service.name : "payment"
```

Then:

```text
log.level : "ERROR"
```

Then:

```text
service.name : "payment"
and log.level : "ERROR"
```

This validates that:

```text
Elasticsearch
   ↓
Kibana
   ↓
Data View
   ↓
Search
```

are working.

---

# 50. Kibana Installation Validation

Verify all layers:

```text
[ ] Package installed
[ ] Correct version
[ ] Configuration file exists
[ ] Elasticsearch DNS works
[ ] Elasticsearch port reachable
[ ] Elasticsearch authentication works
[ ] TLS works
[ ] Kibana service running
[ ] Port 5601 listening
[ ] Browser access works
[ ] Login works
[ ] Data view works
[ ] Logs searchable
```

---

# 51. Production Installation With EC2

A simple AWS architecture:

```text
                   AWS VPC
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
   Logstash        Kibana       Elasticsearch
      EC2             EC2          Cluster
        │              │              │
        └──────────────┴──────────────┘
```

More resilient:

```text
                  Internal ALB
                       │
               ┌───────┴───────┐
               ↓               ↓
           Kibana-01       Kibana-02
               │               │
               └───────┬───────┘
                       ↓
              Elasticsearch Cluster
```

---

# 52. Multi-AZ Architecture

For production AWS:

```text
             Internal ALB
                  │
        ┌─────────┴─────────┐
        ↓                   ↓
      AZ-1                 AZ-2
        │                   │
    Kibana-01            Kibana-02
        │                   │
        └─────────┬─────────┘
                  ↓
       Elasticsearch Cluster
```

Distribute Kibana instances across Availability Zones where practical.

---

# 53. Kibana High Availability

A single Kibana instance creates a single point of failure:

```text
User
 ↓
Kibana X
```

With multiple instances:

```text
                 ALB
                  │
          ┌───────┴───────┐
          ↓               ↓
       Kibana-01       Kibana-02
          │               │
          └───────┬───────┘
                  ↓
             Elasticsearch
```

If one Kibana instance fails, the load balancer can route users to another healthy instance.

---

# 54. Load Balancer Health Checks

The ALB should determine whether a Kibana target is healthy.

Conceptually:

```text
ALB
 │
 ├── Kibana-01 → Healthy
 │
 └── Kibana-02 → Unhealthy
```

Traffic should go only to healthy targets.

Configure health checks according to the Kibana version and authentication/network architecture.

---

# 55. Kibana on EKS

Kibana can also be deployed as a Kubernetes workload.

Architecture:

```text
EKS
 │
 ├── Kibana Pod
 ├── Kibana Pod
 └── Service
       ↓
     ALB
```

Production:

```text
Internal ALB
      ↓
Kibana Service
      ↓
┌─────┴─────┐
↓           ↓
Pod-01     Pod-02
      \     /
       \   /
        ↓
 Elasticsearch
```

---

# 56. Kubernetes Namespace

Keep observability components together where appropriate:

```bash
kubectl create namespace logging
```

Then:

```text
logging
│
├── Kibana
├── Logstash
└── Related logging components
```

Your organization may instead use a dedicated `observability` namespace.

---

# 57. Kubernetes Service

Kibana Pods should normally be exposed through a Kubernetes Service.

Architecture:

```text
ALB / Ingress
      ↓
Kibana Service
      ↓
Kibana Pods
```

The Service provides stable connectivity even when Pods are recreated.

---

# 58. Kubernetes Deployment

A simplified conceptual Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: logging
spec:
  replicas: 2
```

The exact deployment configuration depends on the selected Kibana version and packaging method.

---

# 59. Resource Requests

Define CPU and memory requests:

```yaml
resources:
  requests:
    cpu: ...
    memory: ...
  limits:
    cpu: ...
    memory: ...
```

Size based on:

```text
Concurrent users
Dashboard complexity
Query volume
Elasticsearch latency
```

Do not blindly copy resource values from another environment.

---

# 60. Readiness Probe

A readiness probe prevents traffic from being sent to an unavailable Kibana Pod.

Conceptually:

```text
Kibana Pod
    ↓
Ready?
 ┌──┴──┐
No    Yes
↓      ↓
No    Receive traffic
traffic
```

Use a probe appropriate for the deployed Kibana version and security configuration.

---

# 61. Liveness Probe

A liveness probe can allow Kubernetes to restart a stuck process.

```text
Kibana
 ↓
Healthy?
 ↓
No
 ↓
Restart Pod
```

Avoid aggressive probe settings that cause healthy Kibana instances to restart during temporary Elasticsearch delays.

---

# 62. Kubernetes Configuration

Non-sensitive configuration can be provided using a ConfigMap.

Architecture:

```text
Git
 ↓
ConfigMap
 ↓
Kibana Pod
 ↓
kibana.yml
```

Do not place passwords or private keys in a normal ConfigMap.

---

# 63. Kubernetes Secrets

Sensitive configuration should use an appropriate secret-management mechanism:

```text
AWS Secrets Manager
       ↓
External Secrets
       ↓
Kubernetes Secret
       ↓
Kibana
```

or another approved enterprise secret-management approach.

---

# 64. Helm Deployment

If using Helm:

```text
kibana/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    └── ingress.yaml
```

Helm makes environment-specific deployment easier.

---

# 65. Environment Values

Example:

```text
values-dev.yaml
values-staging.yaml
values-prod.yaml
```

Production might configure:

```text
replicas
resources
Elasticsearch endpoint
TLS
Ingress
security
```

without changing the base chart.

---

# 66. GitOps Architecture

For your environment:

```text
GitHub
   ↓
Kibana Helm Configuration
   ↓
Pull Request
   ↓
GitHub Actions
   ↓
Validation
   ↓
Security Checks
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Kibana
```

This keeps the deployment reproducible.

---

# 67. GitHub Actions Validation

A CI pipeline can validate:

```text
YAML
Helm
Kubernetes manifests
Security configuration
Container image
```

Example flow:

```text
Pull Request
     ↓
YAML Validation
     ↓
Helm Lint
     ↓
Security Scan
     ↓
Review
     ↓
Merge
```

---

# 68. ArgoCD Deployment

After the configuration is merged:

```text
Git Repository
      ↓
ArgoCD
      ↓
Desired State
      ↓
EKS
      ↓
Kibana
```

ArgoCD continuously compares:

```text
Git desired state
        vs
Kubernetes actual state
```

---

# 69. Configuration Drift

Example:

```text
Git:
replicas = 2

Cluster:
replicas = 1
```

This is configuration drift.

With GitOps:

```text
Git
 ↓
Desired State
 ↓
ArgoCD
 ↓
Reconcile
 ↓
replicas = 2
```

---

# 70. Kibana Production Repository

A clean structure:

```text
observability/
│
├── kibana/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   └── ingress.yaml
│   │
│   └── overlays/
│       ├── dev/
│       ├── staging/
│       └── prod/
```

This keeps environments separated.

---

# 71. Terraform Integration

Infrastructure can be provisioned using Terraform:

```text
Terraform
   ↓
VPC
   ↓
Security Groups
   ↓
EKS
   ↓
ALB
   ↓
Route53
   ↓
Kibana Deployment
```

Terraform should manage infrastructure.

ArgoCD should manage the GitOps application deployment.

---

# 72. Responsibility Separation

A useful model:

```text
Terraform
 ↓
AWS Infrastructure

GitHub Actions
 ↓
Validation / CI

ArgoCD
 ↓
Kubernetes Deployment

Kibana
 ↓
Visualization / Investigation
```

This avoids mixing infrastructure provisioning with application deployment.

---

# 73. Monitoring Kibana

Kibana itself should be monitored.

Monitor:

```text
Availability
CPU
Memory
Pod restarts
Response time
Elasticsearch connectivity
Query performance
```

Architecture:

```text
Kibana
 ↓
Metrics
 ↓
Prometheus
 ↓
Grafana
```

---

# 74. Kibana Alerts

Useful operational alerts include:

```text
Kibana unavailable
Kibana Pod restarting
Kibana OOMKilled
High CPU
High memory
Elasticsearch connection failures
High response latency
```

---

# 75. Installation Troubleshooting: Service Won't Start

Start with:

```bash
sudo systemctl status kibana
```

Then:

```bash
sudo journalctl -u kibana -n 100
```

Look for:

```text
Invalid configuration
Port conflict
Elasticsearch connection failure
TLS failure
Authentication failure
Memory problem
```

---

# 76. Troubleshooting: Port Conflict

Check:

```bash
sudo ss -lntp | grep 5601
```

Then:

```bash
sudo lsof -i :5601
```

If another process is using the port, Kibana may fail to bind to it.

---

# 77. Troubleshooting: Elasticsearch Unreachable

Check:

```bash
getent hosts es-01.internal
```

Then:

```bash
nc -zv es-01.internal 9200
```

Then:

```bash
curl https://es-01.internal:9200
```

Investigate:

```text
DNS
Routing
Security Group
Firewall
TLS
Authentication
Elasticsearch health
```

---

# 78. Troubleshooting: TLS Failure

Common causes:

```text
Wrong CA
Expired certificate
Hostname mismatch
Incorrect trust configuration
Wrong protocol
```

Correct approach:

```text
TLS Error
   ↓
Check CA
   ↓
Check certificate
   ↓
Check hostname
   ↓
Check expiration
   ↓
Fix trust configuration
```

Do not disable TLS verification as a permanent workaround.

---

# 79. Troubleshooting: Authentication Failure

Possible symptoms:

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
Authorization problem
```

Check:

```text
Credentials
Service identity
Role
Permissions
Elasticsearch security
```

---

# 80. Troubleshooting: Kibana Page Does Not Open

Follow the network path:

```text
Browser
 ↓
DNS
 ↓
ALB
 ↓
Target Group
 ↓
Security Group
 ↓
Kibana
 ↓
Elasticsearch
```

Check each layer independently.

---

# 81. Troubleshooting: ALB Target Unhealthy

Check:

```bash
kubectl get pods -n logging
```

Then:

```bash
kubectl get svc -n logging
```

Then:

```bash
kubectl describe pod <kibana-pod> -n logging
```

Check:

```text
Readiness probe
Service port
Target port
Security Group
ALB health check
Kibana startup
```

---

# 82. Troubleshooting: Kibana CrashLoopBackOff

Check:

```bash
kubectl get pods -n logging
```

Then:

```bash
kubectl logs <kibana-pod> -n logging --previous
```

Then:

```bash
kubectl describe pod <kibana-pod> -n logging
```

Look for:

```text
Configuration errors
Elasticsearch connectivity
TLS
Authentication
OOMKilled
Probe failures
```

---

# 83. Troubleshooting: OOMKilled

Check:

```bash
kubectl describe pod <kibana-pod> -n logging
```

If you see:

```text
Reason: OOMKilled
```

review:

```text
Memory request
Memory limit
Dashboard workload
Concurrent users
Query volume
```

Then size resources based on measurements.

---

# 84. Troubleshooting: Empty Discover Page

Check:

```text
Data View
Time Range
Elasticsearch Data
@timestamp
Index/Data Stream
KQL Filter
```

For example:

```text
Last 15 minutes
```

may be incorrect if the logs were generated yesterday.

---

# 85. Troubleshooting: Dashboard Slow

Check:

```text
Dashboard panels
Time range
Queries
Aggregations
High-cardinality fields
Elasticsearch response time
Elasticsearch cluster health
```

Do not immediately scale Kibana.

The bottleneck may be Elasticsearch.

---

# 86. Troubleshooting: Elasticsearch Slow

The path is:

```text
Kibana
 ↓
Elasticsearch Query
 ↓
Slow Elasticsearch Response
 ↓
Slow Kibana Dashboard
```

Check:

```text
Shard count
Data volume
Query complexity
Cluster CPU
Memory
Disk
Index mappings
```

---

# 87. Production Installation Validation

After installation:

```text
1. Kibana package installed
2. Version verified
3. kibana.yml configured
4. Elasticsearch reachable
5. TLS validated
6. Authentication validated
7. Service started
8. Port 5601 listening
9. DNS working
10. ALB working
11. Login working
12. Data view created
13. Logs searchable
14. Dashboard tested
15. Monitoring configured
```

---

# 88. Security Checklist

```text
[ ] Kibana not unnecessarily public
[ ] HTTPS enabled
[ ] Elasticsearch communication secured
[ ] Authentication enabled
[ ] RBAC configured
[ ] Least privilege
[ ] Secrets protected
[ ] Security groups restricted
[ ] Internal ALB where appropriate
[ ] Elasticsearch remains private
[ ] Certificates monitored
```

---

# 89. High-Availability Checklist

```text
[ ] Multiple Kibana replicas
[ ] Multiple Availability Zones where practical
[ ] Internal load balancer
[ ] Health checks
[ ] Pod anti-affinity
[ ] Topology spread
[ ] Adequate resources
[ ] Elasticsearch cluster available
[ ] Failure testing
```

---

# 90. Operational Checklist

```text
[ ] Prometheus monitoring
[ ] Grafana dashboard
[ ] Kibana logs monitored
[ ] Pod restart alerts
[ ] Memory alerts
[ ] CPU alerts
[ ] Elasticsearch connectivity alerts
[ ] Certificate expiry monitoring
[ ] GitOps deployment
[ ] Rollback procedure
```

---

# 91. Complete Production Architecture

For your AWS/EKS environment:

```text
                         USERS
                           │
                           ↓
                  Route53 Private DNS
                           │
                           ↓
                    Internal AWS ALB
                           │
              ┌────────────┴────────────┐
              ↓                         ↓
         Kibana Pod-01             Kibana Pod-02
              │                         │
              └────────────┬────────────┘
                           ↓
                  Elasticsearch Cluster
                 ┌─────────┼─────────┐
                 ↓         ↓         ↓
               ES-01     ES-02     ES-03
                 ↑         ↑         ↑
                 └─────────┼─────────┘
                           ↑
                       Logstash
                           ↑
                      Fluent Bit
                           ↑
                       EKS Pods
```

---

# 92. Complete Observability Architecture

Your observability environment can be viewed as:

```text
                         APPLICATIONS
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
           Metrics           Logs           Traces
              │               │               │
              ↓               ↓               ↓
         Prometheus        Logstash       OpenTelemetry
              │               │               │
              ↓               ↓               ↓
           Grafana       Elasticsearch         Jaeger
                              │
                              ↓
                           Kibana
```

This separates the three major observability signals:

```text
Metrics → Prometheus → Grafana

Logs → Logstash → Elasticsearch → Kibana

Traces → OpenTelemetry → Jaeger
```

---

# 93. GitOps Production Flow

For your environment:

```text
Developer
   ↓
GitHub
   ↓
Kibana Configuration
   ↓
Pull Request
   ↓
GitHub Actions
   ├── YAML validation
   ├── Helm validation
   ├── Security checks
   └── Tests
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Kibana
```

This provides:

```text
Version control
Auditability
Repeatability
Drift detection
Rollback
```

---

# 94. Final Installation Mental Model

Remember Kibana installation as:

```text
                 KIBANA INSTALLATION
                         │
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
 Infrastructure      Installation      Configuration
       │                 │                 │
    EC2/EKS          Repository         kibana.yml
    Network          Package            Elasticsearch
    Security         Service            TLS
    DNS              Version            Security
       │                 │                 │
       └─────────────────┼─────────────────┘
                         ↓
                    Connectivity
                         │
               ┌─────────┴─────────┐
               ↓                   ↓
             Users            Elasticsearch
               │                   │
               ↓                   ↓
             HTTPS              HTTPS
               │                   │
               └─────────┬─────────┘
                         ↓
                    Validation
                         ↓
                    Monitoring
                         ↓
                    High Availability
                         ↓
                      GitOps
```

The key principle is:

**Kibana installation in production is not simply installing a package and opening port 5601. You must establish compatible versions, private networking, secure Elasticsearch connectivity, TLS, authentication, RBAC, DNS, load balancing, high availability, monitoring, and GitOps-based deployment.**
