# Kibana Configuration

## 1. Overview

This document covers configuring Kibana for real-world environments after installation.

Kibana configuration mainly controls:

```text
Server
Elasticsearch connectivity
Security
TLS
Authentication
Logging
Monitoring
Sessions
Data views
Spaces
Kibana behavior
```

The production flow is:

```text
Users
  ↓
Internal ALB / Ingress
  ↓
Kibana
  ↓
Elasticsearch
```

For your observability architecture:

```text
Metrics → Prometheus → Grafana

Logs → Logstash → Elasticsearch → Kibana

Traces → OpenTelemetry → Jaeger
```

---

# 2. Main Configuration File

The primary Kibana configuration file is commonly:

```text
/etc/kibana/kibana.yml
```

Edit it with:

```bash
sudo vi /etc/kibana/kibana.yml
```

Before changing production configuration, back it up:

```bash
sudo cp /etc/kibana/kibana.yml \
        /etc/kibana/kibana.yml.bak
```

---

# 3. Configuration Architecture

```text
                    kibana.yml
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
      Server       Elasticsearch      Security
        │               │               │
        ↓               ↓               ↓
      Port          Connection          TLS
      Host          Credentials         Auth
      URL           Certificates        RBAC
                        │
                        ↓
                      Kibana
```

---

# 4. Server Host

The server host determines which network interfaces Kibana listens on.

Local-only:

```yaml
server.host: "127.0.0.1"
```

Server accessible through a private network:

```yaml
server.host: "0.0.0.0"
```

For production, binding to `0.0.0.0` should be combined with:

```text
Security Groups
Firewall
Internal ALB
Authentication
TLS
```

---

# 5. Server Port

The default Kibana port is:

```yaml
server.port: 5601
```

Architecture:

```text
Client
  ↓
5601
  ↓
Kibana
```

Verify:

```bash
sudo ss -lntp | grep 5601
```

---

# 6. Server Name

Give the Kibana instance a meaningful identity where supported by your deployed version/configuration.

Example:

```yaml
server.name: "kibana-prod-01"
```

For multiple nodes:

```text
kibana-prod-01
kibana-prod-02
```

This helps operational identification.

---

# 7. Public Base URL

If users access Kibana through a stable external/internal hostname, configure the appropriate public base URL.

Example:

```yaml
server.publicBaseUrl: "https://kibana.prod.internal"
```

This is useful when Kibana needs to know the URL users actually access.

The value should match the real user-facing URL.

---

# 8. Elasticsearch Hosts

Kibana needs to connect to Elasticsearch.

Example:

```yaml
elasticsearch.hosts:
  - "https://es-01.internal:9200"
```

For a cluster:

```yaml
elasticsearch.hosts:
  - "https://es-01.internal:9200"
  - "https://es-02.internal:9200"
  - "https://es-03.internal:9200"
```

Use the topology and connection configuration appropriate to your Elasticsearch deployment.

---

# 9. Elasticsearch Connectivity

The architecture is:

```text
Kibana
   │
   │ HTTPS
   ↓
Elasticsearch
```

Test DNS:

```bash
getent hosts es-01.internal
```

Test port:

```bash
nc -zv es-01.internal 9200
```

Test API:

```bash
curl https://es-01.internal:9200
```

Authentication may be required.

---

# 10. Elasticsearch Service Identity

Kibana requires an Elasticsearch service identity/credential mechanism.

Do not use the Elasticsearch superuser as the normal Kibana runtime identity.

Separate:

```text
Kibana Server Identity
```

from:

```text
Human User Identity
```

---

# 11. Kibana Security Architecture

```text
Human User
    ↓
Authentication
    ↓
Kibana
    ↓
Kibana Server Identity
    ↓
Elasticsearch
```

This is different from:

```text
User
 ↓
Elasticsearch directly
```

Users should normally access Elasticsearch data through the authorized Kibana interface.

---

# 12. TLS Between Kibana and Elasticsearch

Production should use HTTPS:

```yaml
elasticsearch.hosts:
  - "https://es-prod.internal:9200"
```

Architecture:

```text
Kibana
  ↓
TLS
  ↓
Elasticsearch
```

This protects log queries and credentials in transit.

---

# 13. Elasticsearch CA Certificate

Kibana must trust the CA that signed the Elasticsearch certificate.

Conceptually:

```yaml
elasticsearch.ssl.certificateAuthorities:
  - "/etc/kibana/certs/ca.crt"
```

The exact certificate configuration depends on your deployed Elastic Stack version.

---

# 14. Certificate Validation

Do not solve TLS problems by disabling certificate validation.

Correct troubleshooting:

```text
TLS Failure
    ↓
Check CA
    ↓
Check certificate
    ↓
Check hostname
    ↓
Check expiration
    ↓
Check trust configuration
```

Not:

```text
TLS Failure
    ↓
Disable verification
```

---

# 15. Kibana HTTPS

Kibana itself can serve HTTPS.

Conceptually:

```yaml
server.ssl.enabled: true
```

Then configure the appropriate certificate and private key.

Example structure:

```text
/etc/kibana/certs/
├── kibana.crt
└── kibana.key
```

Private keys must be protected.

---

# 16. Kibana Certificate

The certificate should match the hostname users access.

If users access:

```text
https://kibana.prod.internal
```

the certificate should be valid for:

```text
kibana.prod.internal
```

Otherwise browsers may report certificate errors.

---

# 17. Certificate Permissions

Private keys should not be readable by arbitrary users.

Check:

```bash
sudo ls -l /etc/kibana/certs/
```

Use restrictive permissions appropriate for the Kibana service account.

---

# 18. TLS Architecture With ALB

A common AWS design:

```text
User
 ↓
HTTPS
 ↓
Internal ALB
 ↓
Kibana
 ↓
HTTPS
 ↓
Elasticsearch
```

TLS can terminate at the ALB, while the ALB-to-Kibana connection can remain encrypted when required by the security model.

---

# 19. AWS Security Groups

Recommended flow:

```text
Corporate Network
       ↓
ALB Security Group
       ↓
Kibana Security Group
       ↓
Elasticsearch Security Group
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

Do not allow:

```text
0.0.0.0/0 → Elasticsearch : 9200
```

---

# 20. Authentication

Kibana can integrate with authentication mechanisms supported by the deployed Elastic Stack.

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
```

---

# 21. Role-Based Access Control

RBAC should separate users based on responsibilities.

Example:

```text
Platform Admin
      ↓
Full observability administration

DevOps Engineer
      ↓
Platform + application observability

Developer
      ↓
Application logs

Security Analyst
      ↓
Security / audit data

Read Only
      ↓
Selected dashboards
```

---

# 22. Kibana Spaces

Spaces help organize Kibana content.

Example:

```text
Kibana
│
├── Platform
├── Applications
├── Security
└── Production
```

This prevents all dashboards and saved objects from becoming one large collection.

---

# 23. Application Space

Example:

```text
Applications
│
├── Payment
├── Orders
├── Inventory
├── Cart
└── User
```

Each team can access the content relevant to them.

---

# 24. Platform Space

Example:

```text
Platform
│
├── EKS
├── Nodes
├── Kubernetes
├── Infrastructure
└── Logging
```

This can be used by the DevOps/platform team.

---

# 25. Security Space

Example:

```text
Security
│
├── Authentication
├── Authorization
├── Audit
└── Security Events
```

Restrict access to appropriate security personnel.

---

# 26. Data Views

Kibana needs to know which Elasticsearch data should be queried.

Example:

```text
application-logs-*
```

Another:

```text
kubernetes-logs-*
```

Another:

```text
security-logs-*
```

---

# 27. Time Field

For log data, the primary time field is normally:

```text
@timestamp
```

The flow should be:

```text
Application
 ↓
Logstash
 ↓
Date parsing
 ↓
@timestamp
 ↓
Elasticsearch
 ↓
Kibana
```

Correct timestamps are critical during incident investigation.

---

# 28. KQL

Kibana Query Language is commonly used for filtering.

Example:

```text
service.name : "payment"
```

Multiple conditions:

```text
service.name : "payment"
and log.level : "ERROR"
```

Kubernetes example:

```text
kubernetes.namespace : "production"
```

---

# 29. Production Log Search

A common production investigation:

```text
environment : "production"
and service.name : "payment"
and log.level : "ERROR"
```

Then narrow the time range:

```text
Last 15 minutes
```

This helps avoid unnecessarily querying huge amounts of historical data.

---

# 30. Kubernetes Log Search

Example:

```text
kubernetes.namespace : "production"
and kubernetes.container.name : "payment"
```

Add errors:

```text
kubernetes.namespace : "production"
and kubernetes.container.name : "payment"
and log.level : "ERROR"
```

---

# 31. Trace Correlation

If your logs contain tracing information:

```text
trace.id
span.id
```

you can search:

```text
trace.id : "abc123"
```

Architecture:

```text
Grafana
  ↓
Error Spike
  ↓
Kibana
  ↓
trace.id
  ↓
Jaeger
```

---

# 32. Request Correlation

Applications should ideally include a request/correlation identifier.

Example:

```json
{
  "service.name": "payment",
  "request.id": "req-123",
  "trace.id": "abc123",
  "log.level": "ERROR"
}
```

This allows an engineer to follow one request across services.

---

# 33. Dashboard Configuration

A production dashboard should answer a specific question.

Example:

```text
Payment Production Dashboard
```

Panels:

```text
Error Count
5xx Count
Top Errors
Errors by Pod
Errors by Namespace
Recent Errors
```

---

# 34. Dashboard Time Range

Prefer operational time ranges such as:

```text
Last 15 minutes
Last 30 minutes
Last 1 hour
```

during active incidents.

Avoid defaulting every dashboard to very large ranges.

Large queries can increase Elasticsearch load.

---

# 35. Dashboard Filters

Useful filters:

```text
Environment
Cluster
Namespace
Service
Pod
Container
Log Level
Region
```

Example:

```text
Environment = production
Namespace = payments
Service = payment
```

---

# 36. Visualization Strategy

Use visualizations based on the question.

Examples:

```text
Line chart
    ↓
Errors over time

Bar chart
    ↓
Errors by service

Table
    ↓
Recent error events

Metric
    ↓
Current error count
```

Avoid adding visualizations simply because Kibana supports them.

---

# 37. Saved Searches

A frequently used query can be saved.

Example:

```text
environment : "production"
and log.level : "ERROR"
```

Save it as:

```text
Production Errors
```

This gives engineers a reusable investigation starting point.

---

# 38. Alerts

Kibana rules can detect conditions based on the capabilities of the deployed version and license.

Example:

```text
service.name = payment
log.level = ERROR
count > 100
within 5 minutes
```

Flow:

```text
Elasticsearch
     ↓
Kibana Rule
     ↓
Condition
     ↓
Alert
     ↓
Notification
```

---

# 39. Kibana Alerting vs Prometheus Alerting

Use metrics for metric-oriented conditions:

```text
CPU > 80%
Memory > 90%
Error rate > 5%
Latency > threshold
```

Prometheus/Grafana is generally well suited for these.

Use Kibana for log-oriented conditions:

```text
Authentication failure
Database timeout
Security event
Specific exception
Log volume spike
```

---

# 40. Logging Configuration

Kibana itself produces operational logs.

Useful logging configuration areas include:

```text
Log level
Log format
Output destination
Component logging
```

A normal production level is commonly:

```text
info
```

Use debug/trace only during troubleshooting when appropriate.

---

# 41. Kibana Log Level

Conceptually:

```yaml
logging.root.level: info
```

Higher verbosity:

```text
debug
trace
```

should not normally be enabled permanently because it can generate significant log volume.

---

# 42. Logging to JSON

Structured Kibana logs can be useful in centralized logging environments.

Architecture:

```text
Kibana
 ↓
Structured Logs
 ↓
Fluent Bit
 ↓
Logstash
 ↓
Elasticsearch
```

This allows Kibana's own operational logs to be monitored through the same logging platform.

---

# 43. Monitoring Kibana

Kibana should be monitored using your observability stack.

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

Monitor:

```text
Availability
CPU
Memory
Response time
Pod restarts
Elasticsearch connectivity
```

---

# 44. Kibana Availability

A simple operational target:

```text
Kibana
 ↓
Available?
```

If Kibana becomes unavailable:

```text
Users
 ↓
Cannot investigate logs
```

This can significantly affect incident response even when the application itself is still running.

---

# 45. Kibana and Elasticsearch Monitoring

Always monitor both.

```text
Kibana
   ↓
Elasticsearch
```

A slow Kibana dashboard may actually be an Elasticsearch problem.

Monitor:

```text
Kibana
CPU
Memory
Response time

Elasticsearch
CPU
Memory
Disk
Shard health
Query latency
```

---

# 46. Kibana Session Configuration

Kibana maintains user sessions.

Session-related configuration must be treated as security-sensitive.

Important considerations:

```text
Session timeout
Cookie security
HTTPS
Authentication provider
Load balancing
```

When multiple Kibana instances are used, session behavior must be compatible with the chosen deployment architecture.

---

# 47. Production Session Architecture

```text
                   Internal ALB
                        │
              ┌─────────┴─────────┐
              ↓                   ↓
          Kibana-01           Kibana-02
              │                   │
              └─────────┬─────────┘
                        ↓
                   Elasticsearch
```

Users should be able to move between healthy Kibana instances without unexpected authentication behavior.

---

# 48. Kibana and Load Balancing

The load balancer should distribute requests across healthy Kibana instances.

```text
                 ALB
                  │
         ┌────────┴────────┐
         ↓                 ↓
      Kibana-01         Kibana-02
```

Health checks must be configured appropriately.

---

# 49. Kibana on Kubernetes

Configuration architecture:

```text
Git
 ↓
Helm Values / Manifest
 ↓
ConfigMap
 ↓
Kibana Pod
 ↓
Elasticsearch
```

Sensitive information should come from:

```text
Kubernetes Secret
```

or an external secret-management system.

---

# 50. Kubernetes Configuration Structure

A clean repository:

```text
kibana/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── ingress.yaml
│
└── overlays/
    ├── dev/
    ├── staging/
    └── prod/
```

This follows the folder organization approach you are using throughout your notes.

---

# 51. Helm Configuration

If using Helm:

```text
kibana/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
└── values-prod.yaml
```

Environment-specific values can control:

```text
Replicas
Resources
Elasticsearch endpoint
Ingress
TLS
Security
```

---

# 52. Production Kubernetes Resources

Kibana normally requires:

```text
Deployment
Service
ConfigMap
Secret
Ingress / ALB
RBAC where required
```

Depending on the architecture, additional resources may be required.

---

# 53. Resource Configuration

Example structure:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
  limits:
    cpu: "1"
    memory: "2Gi"
```

These are example values only.

Real production sizing should be based on:

```text
Users
Dashboards
Queries
Elasticsearch latency
Observed CPU
Observed memory
```

---

# 54. Pod Anti-Affinity

For HA:

```text
Worker-01
 └── Kibana-01

Worker-02
 └── Kibana-02
```

Avoid:

```text
Worker-01
 ├── Kibana-01
 └── Kibana-02
```

Use appropriate:

```text
Pod anti-affinity
Topology spread constraints
```

---

# 55. Readiness Probe

Kibana should receive traffic only after it is ready.

Conceptually:

```text
Kibana Pod
    ↓
Readiness
    ↓
Ready?
 ┌──┴──┐
No    Yes
↓      ↓
No     Traffic
traffic
```

Configure the probe according to the Kibana version and security settings.

---

# 56. Liveness Probe

A liveness probe can restart an unhealthy Kibana process.

```text
Kibana
 ↓
Health check
 ↓
Failure
 ↓
Kubernetes restart
```

Avoid probes that restart Kibana simply because Elasticsearch is temporarily slow.

---

# 57. EKS Network Policy

Where appropriate, restrict traffic:

```text
Kibana
 ↓
Allowed
 ↓
Elasticsearch
```

Do not allow every Pod in the cluster to communicate freely with Kibana.

Network policies should match your CNI/networking design.

---

# 58. EKS ALB Architecture

Your AWS architecture can be:

```text
Corporate User
      ↓
Private Route53
      ↓
Internal ALB
      ↓
Kibana Service
      ↓
Kibana Pods
      ↓
Elasticsearch
```

This provides a controlled entry point.

---

# 59. Terraform Responsibilities

Terraform can manage:

```text
VPC
Subnets
Security Groups
ALB
Route53
EKS
IAM
```

Then GitOps manages Kibana:

```text
Terraform
 ↓
Infrastructure
 ↓
EKS
 ↓
ArgoCD
 ↓
Kibana
```

---

# 60. GitHub Actions Responsibilities

GitHub Actions can validate the Kibana deployment configuration.

Example:

```text
Pull Request
    ↓
YAML validation
    ↓
Helm lint
    ↓
Security scan
    ↓
Manifest validation
    ↓
Review
```

Then merge.

---

# 61. ArgoCD Responsibilities

ArgoCD:

```text
Git
 ↓
Desired State
 ↓
ArgoCD
 ↓
EKS
 ↓
Kibana
```

ArgoCD handles:

```text
Synchronization
Drift detection
Deployment
Rollback
```

---

# 62. Configuration Change Workflow

Suppose you change:

```text
Elasticsearch endpoint
```

Correct process:

```text
Developer
 ↓
Git change
 ↓
Pull Request
 ↓
GitHub Actions
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
 ↓
Kibana
```

Avoid editing production Pods manually.

---

# 63. Configuration Drift

Example:

```text
Git:
replicas = 2

EKS:
replicas = 1
```

ArgoCD detects the difference.

Desired:

```text
2 replicas
```

Actual:

```text
1 replica
```

This is configuration drift.

---

# 64. Rollback

Suppose a configuration change breaks Kibana.

Git history:

```text
Version 1
   ↓
Working

Version 2
   ↓
Broken
```

Rollback:

```text
Git revert
 ↓
GitHub Actions
 ↓
Merge
 ↓
ArgoCD
 ↓
Version 1
```

This is safer than manually editing production.

---

# 65. Production Configuration Example

A conceptual configuration:

```yaml
server.host: "0.0.0.0"
server.port: 5601
server.publicBaseUrl: "https://kibana.prod.internal"

elasticsearch.hosts:
  - "https://es-01.internal:9200"
  - "https://es-02.internal:9200"
  - "https://es-03.internal:9200"

server.ssl.enabled: true
```

Authentication, certificate trust, service identity, and other security settings must be configured according to the exact Elastic Stack version and deployment.

---

# 66. Configuration Validation

After editing:

```bash
sudo systemctl restart kibana
```

Immediately check:

```bash
sudo systemctl status kibana
```

Then:

```bash
sudo journalctl -u kibana -n 100
```

Then:

```bash
sudo ss -lntp | grep 5601
```

Then test:

```bash
curl https://kibana.prod.internal
```

---

# 67. Configuration Change Checklist

Before deploying:

```text
[ ] YAML syntax valid
[ ] Correct Elasticsearch endpoint
[ ] DNS works
[ ] Port reachable
[ ] TLS certificate valid
[ ] CA trusted
[ ] Authentication valid
[ ] Security groups correct
[ ] Server URL correct
[ ] Port correct
[ ] Resource capacity sufficient
```

---

# 68. Troubleshooting: Kibana Won't Start

Run:

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
Unknown setting
YAML formatting
Port conflict
TLS failure
Elasticsearch failure
Authentication failure
```

---

# 69. Troubleshooting: Elasticsearch Connection

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

If the port works but Kibana cannot connect:

```text
TLS
Authentication
Certificate trust
Configuration
```

become the next areas to investigate.

---

# 70. Troubleshooting: 401

If Elasticsearch returns:

```text
401 Unauthorized
```

check:

```text
Kibana service credentials
Authentication configuration
Credential validity
```

---

# 71. Troubleshooting: 403

If Elasticsearch returns:

```text
403 Forbidden
```

check:

```text
Role
Privileges
Index permissions
Kibana privileges
```

Authentication succeeded, but authorization is insufficient.

---

# 72. Troubleshooting: Certificate Error

Check:

```text
Certificate hostname
CA
Expiration
Trust chain
File permissions
```

Use:

```bash
openssl s_client \
  -connect es-01.internal:9200 \
  -servername es-01.internal
```

This can help inspect the TLS handshake and certificate chain.

---

# 73. Troubleshooting: Kibana Not Accessible

Check:

```text
1. DNS
2. ALB
3. Security Group
4. Target Group
5. Service
6. Pod
7. Kibana port
8. Kibana logs
9. Elasticsearch
```

Architecture:

```text
User
 ↓
DNS
 ↓
ALB
 ↓
Service
 ↓
Pod
 ↓
Kibana
 ↓
Elasticsearch
```

---

# 74. Troubleshooting: Empty Dashboards

Check:

```text
Data View
Time range
@timestamp
Index/Data Stream
Elasticsearch data
KQL filter
Dashboard filters
```

A dashboard can be healthy while returning no data because the query or time range is wrong.

---

# 75. Troubleshooting: Slow Dashboards

Investigate:

```text
Time range
Number of panels
Query complexity
Aggregations
High-cardinality fields
Elasticsearch performance
```

Example:

```text
Last 7 days
```

may be unnecessarily expensive during an incident.

Use:

```text
Last 15 minutes
```

when appropriate.

---

# 76. Troubleshooting: High Memory

For Kubernetes:

```bash
kubectl describe pod <kibana-pod> -n logging
```

Look for:

```text
OOMKilled
```

Then check:

```text
Memory requests
Memory limits
Dashboard complexity
Concurrent users
Query volume
```

---

# 77. Troubleshooting: High CPU

Possible causes:

```text
Heavy dashboards
Many users
Expensive queries
Large aggregations
Elasticsearch latency
```

Investigate both Kibana and Elasticsearch.

---

# 78. Production Security Checklist

```text
[ ] Private Kibana
[ ] HTTPS
[ ] TLS to Elasticsearch
[ ] Valid CA
[ ] Authentication
[ ] RBAC
[ ] Least privilege
[ ] Protected secrets
[ ] Restricted security groups
[ ] Internal ALB
[ ] Elasticsearch not publicly exposed
[ ] Certificate monitoring
```

---

# 79. Production Reliability Checklist

```text
[ ] Multiple Kibana instances
[ ] Multiple AZs where practical
[ ] Internal load balancer
[ ] Health checks
[ ] Pod anti-affinity
[ ] Topology spread
[ ] Resource requests
[ ] Resource limits
[ ] Monitoring
[ ] Alerting
[ ] Failure testing
```

---

# 80. Production Operations Checklist

```text
[ ] Git-managed configuration
[ ] GitHub Actions validation
[ ] ArgoCD deployment
[ ] Drift detection
[ ] Rollback procedure
[ ] Dashboard ownership
[ ] Data view management
[ ] RBAC review
[ ] Certificate renewal
[ ] Capacity review
```

---

# 81. Complete Real-World Configuration Flow

```text
                     GitHub
                        │
                        ↓
                Kibana Configuration
                        │
                        ↓
                  Pull Request
                        │
                        ↓
                 GitHub Actions
                ┌───────┼───────┐
                ↓       ↓       ↓
              YAML    Helm    Security
            Validation Validation  Scan
                └───────┼───────┘
                        ↓
                      Merge
                        ↓
                     ArgoCD
                        ↓
                       EKS
                        ↓
                  Kibana Cluster
                  ┌─────┴─────┐
                  ↓           ↓
              Kibana-01   Kibana-02
                  │           │
                  └─────┬─────┘
                        ↓
               Elasticsearch
```

---

# 82. Complete ELK Architecture

```text
                         EKS
                          │
                    Applications
                          │
                          ↓
                     Fluent Bit
                          │
                          ↓
                       Logstash
                          │
                          ↓
               Elasticsearch Cluster
                  ┌───────┼───────┐
                  ↓       ↓       ↓
                ES-01   ES-02   ES-03
                          │
                          ↓
                    Kibana Cluster
                  ┌───────┴───────┐
                  ↓               ↓
              Kibana-01       Kibana-02
                  │               │
                  └───────┬───────┘
                          ↓
                     Internal ALB
                          ↓
                        Users
```

---

# 83. Complete Observability Architecture

```text
                         APPLICATIONS
                              │
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
          Metrics            Logs            Traces
             │                │                │
             ↓                ↓                ↓
        Prometheus         Logstash       OpenTelemetry
             │                │                │
             ↓                ↓                ↓
          Grafana       Elasticsearch          Jaeger
                              │
                              ↓
                           Kibana
```

The three signals work together:

```text
Metrics
  ↓
Detect the problem

Logs
  ↓
Understand the error

Traces
  ↓
Locate the failing dependency
```

---

# 84. Incident Investigation Example

Suppose Grafana shows:

```text
Payment error rate ↑
```

Then:

```text
Grafana
   ↓
5xx spike
   ↓
Kibana
   ↓
payment ERROR logs
   ↓
Database timeout
   ↓
trace.id
   ↓
Jaeger
   ↓
Database dependency slow
```

This is how the complete observability stack works together.

---

# 85. Final Mental Model

Remember Kibana configuration in six layers:

```text
                    KIBANA
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      Server      Elasticsearch     Security
        │              │              │
     Host/Port       Hosts           TLS
     URL             CA              Auth
        │              │              RBAC
        └──────────────┼──────────────┘
                       ↓
                    Users
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Spaces       Data Views   Dashboards
          │            │            │
          └────────────┼────────────┘
                       ↓
                  Investigation
                       │
             ┌─────────┼─────────┐
             ↓         ↓         ↓
           Logs      Alerts    Search
```

The key principle is:

**Treat Kibana configuration as production code. Keep Elasticsearch private, secure both user-to-Kibana and Kibana-to-Elasticsearch communication, use authentication and least-privilege access, organize data through data views and spaces, build operational dashboards around real incidents, monitor Kibana itself with Prometheus/Grafana, and manage Kubernetes configuration through GitHub Actions + ArgoCD rather than manual production changes.**
