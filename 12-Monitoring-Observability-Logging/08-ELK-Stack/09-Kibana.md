# Kibana

## 1. Overview

Kibana is the visualization and analysis layer of the ELK stack.

The core ELK architecture is:

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

Kibana does **not** normally act as the primary log-storage system.

Its main responsibilities are:

```text
Search
Explore
Visualize
Analyze
Dashboard
Alert
Discover
Investigate
```

---

# 2. Kibana in the ELK Stack

Each component has a different responsibility:

```text
Logstash
   ↓
Collect
Parse
Transform
Route

Elasticsearch
   ↓
Index
Store
Search
Aggregate

Kibana
   ↓
Explore
Visualize
Dashboard
Investigate
```

Think of it as:

```text
Logs → Logstash → Elasticsearch → Kibana
```

---

# 3. Why Kibana?

Suppose your production application generates thousands of logs:

```text
INFO
INFO
INFO
ERROR
WARN
INFO
ERROR
...
```

Searching raw files becomes difficult.

Kibana allows engineers to ask questions such as:

```text
Which services are generating errors?
Which endpoint has the most 5xx responses?
When did the errors start?
Which Kubernetes namespace is affected?
Which pod is generating failures?
What happened immediately before the incident?
```

---

# 4. Kibana Architecture

A basic architecture:

```text
                    Users
                      │
                      ↓
                   Kibana
                      │
                      ↓
               Elasticsearch
                      │
                      ↓
                  Log Data
```

In production:

```text
Users
  │
  ↓
Internal Load Balancer / Ingress
  │
  ↓
Kibana
  │
  ↓
Elasticsearch Cluster
```

---

# 5. Production ELK Architecture

```text
                         USERS
                           │
                           ↓
                        Kibana
                           │
                           ↓
              Elasticsearch Cluster
             ┌────────────┼────────────┐
             ↓            ↓            ↓
           ES-01        ES-02        ES-03
             ↑            ↑            ↑
             └────────────┼────────────┘
                          ↑
                     Logstash
                          ↑
                     Fluent Bit
                          ↑
                       EKS Pods
```

Kibana provides the human-facing interface for the Elasticsearch data.

---

# 6. Kibana and Elasticsearch

Kibana depends heavily on Elasticsearch.

Architecture:

```text
Kibana
   │
   │ Queries
   ↓
Elasticsearch
   │
   │ Results
   ↓
Kibana
   │
   ↓
User
```

If Elasticsearch is unavailable:

```text
Kibana
   ↓
Elasticsearch X
```

Kibana cannot retrieve the underlying data.

---

# 7. Kibana Server

Kibana itself is a server-side application.

It provides a web interface such as:

```text
http://kibana.example.com
```

or, in production:

```text
https://kibana.example.com
```

Do not expose Kibana directly to the public internet unless there is a specific, strongly secured requirement.

---

# 8. Kibana Configuration File

The main Kibana configuration file is commonly:

```text
/etc/kibana/kibana.yml
```

This file controls settings such as:

```text
Server configuration
Elasticsearch connection
Security
Logging
Monitoring
TLS
```

---

# 9. Basic Kibana Configuration

A conceptual configuration:

```yaml
server.host: "0.0.0.0"

server.port: 5601

elasticsearch.hosts:
  - "https://elasticsearch.internal:9200"
```

The exact production configuration depends on your networking and security architecture.

---

# 10. Kibana Port

The default Kibana web port is:

```text
5601
```

Check whether Kibana is listening:

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

---

# 11. Kibana Host Binding

A development environment may use:

```yaml
server.host: "127.0.0.1"
```

A server accessed from other systems may use:

```yaml
server.host: "0.0.0.0"
```

However, binding to all interfaces does **not** mean Kibana should be publicly exposed.

Production network controls should restrict access.

---

# 12. Production Network Architecture

A better architecture is:

```text
User
 ↓
Corporate Network / VPN
 ↓
Internal Load Balancer
 ↓
Kibana
 ↓
Private Elasticsearch
```

Avoid:

```text
Internet
   ↓
Kibana
   ↓
Elasticsearch
```

without strong authentication, authorization, TLS, and network protection.

---

# 13. Kibana Installation

On Linux, the normal workflow is:

```text
Official Elastic Repository
          ↓
Package Installation
          ↓
kibana.yml
          ↓
Elasticsearch Connection
          ↓
Security Configuration
          ↓
Service Start
          ↓
Web Access
```

Install the Kibana version compatible with your Elasticsearch deployment.

---

# 14. RPM-Based Installation

For RHEL/Amazon Linux-style systems:

```bash
sudo dnf install kibana
```

The Elastic repository must first be configured.

Then verify:

```bash
rpm -qa | grep kibana
```

---

# 15. Debian / Ubuntu Installation

On Debian/Ubuntu:

```bash
sudo apt update
```

Then:

```bash
sudo apt install kibana
```

Again, use the official Elastic repository for the selected version.

---

# 16. Verify Installation

Check:

```bash
systemctl status kibana
```

If the service is not running:

```bash
sudo systemctl start kibana
```

Enable startup:

```bash
sudo systemctl enable kibana
```

---

# 17. Kibana Service Management

Start:

```bash
sudo systemctl start kibana
```

Stop:

```bash
sudo systemctl stop kibana
```

Restart:

```bash
sudo systemctl restart kibana
```

Status:

```bash
sudo systemctl status kibana
```

Enable:

```bash
sudo systemctl enable kibana
```

---

# 18. Kibana Logs

When Kibana fails to start:

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

These logs are usually the first place to look for:

```text
Configuration errors
Elasticsearch connection failures
TLS errors
Authentication problems
Plugin failures
Port conflicts
```

---

# 19. Kibana Configuration Directory

A common package installation uses:

```text
/etc/kibana/
```

Typical contents include:

```text
kibana.yml
node.options
```

The exact layout depends on the package/version.

---

# 20. Kibana Data Directory

A package installation commonly uses:

```text
/var/lib/kibana/
```

This location can contain Kibana-related runtime data depending on the version and configuration.

---

# 21. Kibana Log Directory

A package installation may use:

```text
/var/log/kibana/
```

Check:

```bash
sudo ls -la /var/log/kibana/
```

For systemd-managed installations, `journalctl` is also useful.

---

# 22. Connect Kibana to Elasticsearch

The most important Kibana configuration is the Elasticsearch connection.

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

Use the appropriate configuration recommended for your Elasticsearch version and deployment topology.

---

# 23. Elasticsearch Connectivity

The communication path is:

```text
Kibana
  ↓
HTTPS
  ↓
Elasticsearch
```

Test DNS:

```bash
getent hosts es-01.internal
```

Test connectivity:

```bash
nc -zv es-01.internal 9200
```

Test HTTPS:

```bash
curl https://es-01.internal:9200
```

Authentication may be required.

---

# 24. Authentication

Kibana needs to authenticate with Elasticsearch.

Modern secured deployments use a Kibana service identity/credential mechanism supported by the deployed Elastic Stack version.

Do not use the Elasticsearch superuser as the normal Kibana runtime identity.

---

# 25. Kibana Security Architecture

```text
User
 ↓
Kibana
 ↓
Authenticated Session
 ↓
Elasticsearch
 ↓
Authorized Data
```

Security should exist at multiple layers:

```text
Network
TLS
Authentication
Authorization
```

---

# 26. TLS

Production:

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
Session information
Queries
Log data
Dashboard data
```

---

# 27. Kibana HTTPS

Kibana can be configured to serve HTTPS directly.

Conceptually:

```yaml
server.ssl.enabled: true
```

The exact certificate and key settings depend on your deployment and Elastic Stack version.

---

# 28. TLS Certificate

Kibana requires a valid certificate when serving HTTPS.

Architecture:

```text
Certificate Authority
       │
       ↓
Kibana Certificate
       │
       ↓
HTTPS :443 / 5601
```

The certificate should match the hostname users access.

---

# 29. Reverse Proxy / Load Balancer

A common production architecture is:

```text
User
 ↓
ALB / Reverse Proxy
 ↓
HTTPS
 ↓
Kibana
 ↓
Elasticsearch
```

This allows centralized handling of:

```text
TLS
DNS
Access control
Network boundaries
```

If TLS terminates at the load balancer, traffic from the load balancer to Kibana should still be protected when required by your security model.

---

# 30. Kibana Behind AWS ALB

For your AWS environment:

```text
Internet / Corporate Network
           ↓
       AWS ALB
           ↓
        Kibana
           ↓
    Elasticsearch
```

Prefer an internal ALB when Kibana is intended only for internal users.

---

# 31. Security Groups

Example:

```text
User / Corporate Network
          ↓
       ALB SG
          ↓
       Kibana SG
          ↓
 Elasticsearch SG
```

Allow only required traffic.

Example:

```text
ALB SG → Kibana SG : 5601
Kibana SG → Elasticsearch SG : 9200
```

Do not allow broad unrestricted access.

---

# 32. Kibana Authentication

Users should authenticate before accessing dashboards.

Depending on the Elastic Stack security configuration, supported authentication mechanisms can include:

```text
Username/password
Single Sign-On
SAML
OIDC
LDAP
Other supported providers
```

Enterprise environments often integrate Kibana with centralized identity providers.

---

# 33. Role-Based Access Control

Kibana should not give every engineer unrestricted access.

Example roles:

```text
Platform Admin
DevOps Engineer
Application Developer
Security Analyst
Read-only User
```

Permissions can be designed around:

```text
Spaces
Dashboards
Indices
Features
Actions
```

---

# 34. Example Access Model

```text
Platform Team
     ↓
All observability dashboards

Application Team
     ↓
Application logs

Security Team
     ↓
Security logs

Read-only Users
     ↓
Selected dashboards
```

This reduces unnecessary access to sensitive data.

---

# 35. Kibana Spaces

Spaces allow teams to organize Kibana content.

Example:

```text
Kibana
 │
 ├── Platform
 │
 ├── Applications
 │
 ├── Security
 │
 └── Production
```

This is useful in organizations with multiple teams.

---

# 36. Discover

Discover is one of Kibana's primary investigation tools.

It allows engineers to:

```text
Search logs
Filter events
Inspect fields
Change time ranges
View individual documents
Create saved searches
```

Example incident:

```text
Payment 5xx ↑
      ↓
Open Discover
      ↓
service:payment
      ↓
level:ERROR
```

---

# 37. Discover Workflow

A typical troubleshooting workflow:

```text
Incident Alert
      ↓
Grafana
      ↓
Open Kibana
      ↓
Discover
      ↓
Filter service
      ↓
Filter error level
      ↓
Inspect timestamps
      ↓
Inspect trace/request ID
```

---

# 38. KQL

Kibana Query Language, commonly called KQL, is used for filtering and searching data in Kibana.

Example:

```text
service.name : "payment"
```

Another:

```text
log.level : "ERROR"
```

Combined:

```text
service.name : "payment" and log.level : "ERROR"
```

KQL is designed primarily for filtering.

---

# 39. KQL Example

Suppose logs contain:

```json
{
  "service.name": "payment",
  "log.level": "ERROR",
  "status_code": 500
}
```

Search:

```text
service.name : "payment" and log.level : "ERROR"
```

This returns matching events.

---

# 40. Filtering by Kubernetes Namespace

If Kubernetes metadata exists:

```text
kubernetes.namespace : "production"
```

Combine:

```text
kubernetes.namespace : "production"
and log.level : "ERROR"
```

This is useful during EKS incidents.

---

# 41. Filtering by Pod

Example:

```text
kubernetes.pod.name : "payment-7d89f"
```

This helps isolate a specific Pod.

---

# 42. Filtering by HTTP Status

Example:

```text
http.response.status_code >= 500
```

This can identify server-side errors.

---

# 43. Searching for a Trace ID

If applications propagate tracing information:

```text
trace.id : "abc123"
```

This allows you to locate logs belonging to a particular distributed transaction.

---

# 44. Logs + Traces

A useful observability workflow is:

```text
Grafana
  ↓
5xx spike
  ↓
Kibana
  ↓
Find trace.id
  ↓
Jaeger
  ↓
Trace request
```

This connects the observability tools in your chapter.

---

# 45. Data Views

Before using Discover and dashboards, Kibana needs to know which Elasticsearch data it should search.

This is commonly done through a **data view**.

Example:

```text
application-logs-*
```

The data view can represent:

```text
application-logs-2026.08.11
application-logs-2026.08.12
application-logs-2026.08.13
```

---

# 46. Time Field

For time-series logs, select the appropriate timestamp field.

Usually:

```text
@timestamp
```

This allows Kibana to filter data by time.

Example:

```text
Last 15 minutes
Last 1 hour
Last 24 hours
Custom range
```

---

# 47. Why `@timestamp` Matters

Suppose an application generated an event at:

```text
10:30
```

but Elasticsearch indexed it at:

```text
10:35
```

If the original timestamp is not correctly mapped to `@timestamp`, Kibana can show the event at the wrong time.

Correct flow:

```text
Application Timestamp
        ↓
Logstash Date Filter
        ↓
@timestamp
        ↓
Elasticsearch
        ↓
Kibana
```

---

# 48. Kibana Dashboards

Dashboards combine visualizations and saved searches.

Example:

```text
Production Dashboard
 ├── Request Rate
 ├── Error Rate
 ├── 5xx
 ├── Top Errors
 ├── Logs
 └── Kubernetes Pods
```

---

# 49. Application Dashboard

A production application dashboard might include:

```text
Requests
Errors
5xx
Top Exceptions
Top Services
Latency-related log events
Database errors
Deployment events
```

---

# 50. Infrastructure Dashboard

Example:

```text
Infrastructure Dashboard
 ├── Host Errors
 ├── Kernel Errors
 ├── Disk Errors
 ├── Network Errors
 ├── Authentication Failures
 └── System Events
```

---

# 51. Kubernetes Dashboard

Example:

```text
EKS Logging Dashboard
 ├── Logs by Namespace
 ├── Logs by Pod
 ├── Error Rate
 ├── CrashLoopBackOff Events
 ├── OOMKilled Events
 ├── ImagePullBackOff Events
 └── Top Error Messages
```

---

# 52. Security Dashboard

A security-focused dashboard might contain:

```text
Authentication Failures
Privilege Changes
Suspicious IPs
Security Events
Failed Requests
Access Denied Events
```

Access should be restricted to appropriate users.

---

# 53. Visualization Types

Kibana supports multiple visualization approaches.

Common examples include:

```text
Bar charts
Line charts
Pie/donut charts
Tables
Metric panels
Heatmaps
Maps
Lens visualizations
```

Choose visualizations based on the question you need to answer.

---

# 54. Kibana Lens

Lens provides a visual interface for creating charts.

Example:

```text
Field:
log.level

Aggregation:
Count

Breakdown:
service.name
```

This can create:

```text
Errors by Service
```

---

# 55. Example Error Visualization

Data:

```text
payment   ERROR  500
payment   ERROR  500
orders    ERROR  500
cart      WARN
```

Visualization:

```text
Errors
  │
  │       █
  │       █
  │   █   █
  │   █   █
  └────────────
    cart payment orders
```

This quickly identifies problematic services.

---

# 56. Top Error Messages

A useful visualization:

```text
Top Errors
────────────────────────────
Database timeout       1250
Connection refused      820
Payment failed          430
Redis timeout           210
```

This helps prioritize incidents.

---

# 57. Logs by Service

Example:

```text
Service       Log Count
-----------------------
payment        100,000
orders          80,000
inventory       60,000
cart            35,000
user            20,000
```

This can identify unusual traffic or noisy services.

---

# 58. Error Rate Over Time

Example:

```text
Errors
  │
  │          /\
  │         /  \
  │________/    \________
  └──────────────────────
              Time
```

A spike can indicate:

```text
Deployment
Dependency failure
Database issue
Traffic spike
Configuration change
```

---

# 59. Dashboard Filters

Dashboards should allow filtering by:

```text
Environment
Cluster
Namespace
Service
Pod
Container
Level
Region
Time
```

Example:

```text
Environment = production
Namespace   = payments
Service     = payment
Level       = ERROR
```

---

# 60. Dashboard Design Principle

Do not create dashboards containing every available field.

A good dashboard answers a specific operational question.

For example:

```text
"Why is payment failing?"
```

is better than:

```text
"Show me every field in Elasticsearch."
```

---

# 61. Kibana Alerts

Kibana can create rules/alerts based on data and conditions supported by the deployed version and license.

Example:

```text
Condition:
More than 100 ERROR events
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
Notification / Connector
```

---

# 62. Example Error Alert

Condition:

```text
service.name = payment
log.level = ERROR
count > 100
window = 5 minutes
```

This can indicate a production incident.

---

# 63. Alerting vs Prometheus

Use the right tool for the right problem.

Prometheus:

```text
Metrics
 ↓
CPU
Memory
Request rate
Error rate
Latency
```

Kibana:

```text
Logs
 ↓
Exceptions
Messages
Audit events
Detailed errors
```

Both can participate in the same incident workflow.

---

# 64. Observability Architecture

Your complete stack:

```text
                   Applications
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Metrics        Logs        Traces
          │            │            │
          ↓            ↓            ↓
    Prometheus     Logstash     OpenTelemetry
          │            │            │
          ↓            ↓            ↓
       Grafana    Elasticsearch    Jaeger
                       │
                       ↓
                     Kibana
```

This provides:

```text
Metrics
Logs
Traces
```

---

# 65. Kibana + Grafana

These tools serve different purposes.

Grafana:

```text
Metrics
Infrastructure
Time-series
Prometheus
```

Kibana:

```text
Logs
Elasticsearch
Log analysis
```

Example:

```text
Grafana
 ↓
Payment 5xx spike

Kibana
 ↓
Database timeout logs
```

---

# 66. Kibana + Jaeger

A distributed incident:

```text
Grafana
 ↓
Error rate ↑
 ↓
Kibana
 ↓
trace.id
 ↓
Jaeger
 ↓
Payment → Database
```

This gives engineers:

```text
What happened?
Why did it happen?
Which service caused it?
```

---

# 67. Kibana and OpenTelemetry

If applications emit OpenTelemetry telemetry:

```text
Application
   │
   ├── Metrics
   ├── Logs
   └── Traces
```

Kibana can be part of the broader observability ecosystem for telemetry that reaches Elasticsearch.

For your architecture, however, keep the responsibilities clear:

```text
Prometheus → Metrics
Elasticsearch/Kibana → Logs
Jaeger → Traces
```

---

# 68. Saved Searches

A commonly used investigation query can be saved.

Example:

```text
service.name : "payment"
and log.level : "ERROR"
```

Save it as:

```text
Payment Production Errors
```

This allows engineers to reuse the investigation query.

---

# 69. Saved Objects

Kibana can manage objects such as:

```text
Dashboards
Visualizations
Saved searches
Data views
Rules
Other supported objects
```

These can be managed through Kibana's UI and, in appropriate environments, exported/imported for controlled promotion.

---

# 70. Dev → Staging → Production

A good workflow is:

```text
Development
    ↓
Create Dashboard
    ↓
Test
    ↓
Export / Version
    ↓
Staging
    ↓
Validate
    ↓
Production
```

Avoid creating critical production dashboards only through undocumented manual changes.

---

# 71. Kibana Configuration as Code

For your DevOps environment:

```text
GitHub
   ↓
Kibana Configuration
   ↓
Pull Request
   ↓
GitHub Actions
   ↓
Validation
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
```

This gives reproducibility.

---

# 72. Kibana on EKS

Kibana can run as a Kubernetes workload.

Architecture:

```text
EKS
 │
 ├── Kibana Pod
 ├── Kibana Pod
 └── Service
       ↓
Internal ALB / Ingress
```

For high availability:

```text
             Internal ALB
                  │
          ┌───────┴───────┐
          ↓               ↓
      Kibana Pod       Kibana Pod
          │               │
          └───────┬───────┘
                  ↓
          Elasticsearch
```

---

# 73. Kibana Kubernetes Service

Typical flow:

```text
User
 ↓
ALB / Ingress
 ↓
Kibana Service
 ↓
Kibana Pods
 ↓
Elasticsearch
```

The Service provides stable internal connectivity to the Pods.

---

# 74. Kibana Resource Requests

Define Kubernetes resources:

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
Saved searches
Query volume
Elasticsearch response time
```

---

# 75. Kibana Readiness

Kubernetes should not send traffic to a Kibana Pod that is not ready.

Use:

```text
Readiness Probe
```

Conceptually:

```text
Kibana Pod
    ↓
Ready?
 ┌──┴──┐
No    Yes
↓      ↓
No     Receive traffic
traffic
```

---

# 76. Kibana Liveness

A liveness probe can help Kubernetes detect a stuck process.

Conceptually:

```text
Kibana
 ↓
Healthy?
 ↓
No
 ↓
Restart
```

Do not make probes overly aggressive.

---

# 77. Kibana High Availability

A production Kubernetes architecture:

```text
                    Internal ALB
                         │
              ┌──────────┴──────────┐
              ↓                     ↓
          Kibana-01             Kibana-02
              │                     │
              └──────────┬──────────┘
                         ↓
                Elasticsearch Cluster
```

Spread Pods across worker nodes/AZs where practical.

---

# 78. Kubernetes Anti-Affinity

Avoid:

```text
Worker-01
 ├── Kibana-01
 └── Kibana-02
```

Prefer:

```text
Worker-01
 └── Kibana-01

Worker-02
 └── Kibana-02
```

Use:

```text
Pod anti-affinity
Topology spread constraints
```

as appropriate.

---

# 79. Kibana Environment Separation

Maintain separate environments:

```text
Development
Staging
Production
```

Example:

```text
kibana-dev
kibana-staging
kibana-prod
```

Each should connect to the appropriate Elasticsearch environment.

---

# 80. Kibana Production Architecture on AWS

For your AWS/EKS environment:

```text
                   Corporate Users
                         │
                         ↓
                Internal Route53 DNS
                         │
                         ↓
                  Internal AWS ALB
                         │
                         ↓
                 Kibana Service
                         │
                  ┌──────┴──────┐
                  ↓             ↓
              Kibana-01     Kibana-02
                  │             │
                  └──────┬──────┘
                         ↓
                Elasticsearch Cluster
                         │
                         ↑
                     Logstash
                         ↑
                    Fluent Bit
                         ↑
                       EKS
```

---

# 81. Kibana Security Groups

A secure AWS design:

```text
ALB Security Group
       ↓
Kibana Security Group
       ↓
Elasticsearch Security Group
```

Only required traffic should be allowed.

---

# 82. DNS

A friendly internal DNS name:

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

This is cleaner than asking users to remember Pod IPs or node ports.

---

# 83. Authentication With Corporate Identity

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
 ↓
Authorized Dashboards
```

This allows centralized:

```text
User lifecycle
Authentication
MFA
Role assignment
Access revocation
```

---

# 84. Least Privilege

Example:

```text
Developer
 ↓
Application logs only

DevOps
 ↓
Platform + application logs

Security
 ↓
Security + audit logs

Admin
 ↓
Full administrative capabilities
```

This is safer than giving everyone admin access.

---

# 85. Sensitive Log Access

Some logs may contain:

```text
PII
Customer information
Authentication events
Security information
Internal infrastructure data
```

Kibana permissions should prevent unauthorized access.

---

# 86. Log Retention

Kibana itself is not the primary retention mechanism.

Retention is primarily handled through Elasticsearch data lifecycle/index management.

Architecture:

```text
Logs
 ↓
Elasticsearch
 ↓
Lifecycle Policy
 ↓
Hot
 ↓
Warm
 ↓
Cold/Frozen where applicable
 ↓
Delete
```

Kibana provides visibility into the data.

---

# 87. High Log Volume

If logs increase dramatically:

```text
Application
 ↓
Fluent Bit
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

Kibana may appear slow because the underlying Elasticsearch queries are expensive.

Do not assume Kibana itself is always the bottleneck.

Investigate:

```text
Query
Elasticsearch
Shard count
Data volume
Dashboard design
Time range
```

---

# 88. Slow Dashboard

If a dashboard is slow:

```text
Kibana
 ↓
Query
 ↓
Elasticsearch
 ↓
Slow response
```

Investigate:

```text
Large time range
High-cardinality fields
Too many panels
Expensive aggregations
Too much data
Elasticsearch cluster health
```

---

# 89. Dashboard Optimization

Prefer:

```text
Last 15 minutes
```

over:

```text
Last 5 years
```

when troubleshooting an active incident.

Use appropriate filters:

```text
service.name : "payment"
environment : "production"
```

This reduces unnecessary search scope.

---

# 90. High-Cardinality Fields

Examples:

```text
request.id
trace.id
user.id
session.id
```

These can be valuable for investigation but expensive for some aggregations.

Use them primarily for searching/correlation rather than indiscriminate aggregation.

---

# 91. Kibana Query Troubleshooting

If a query returns no data:

```text
1. Check time range.
2. Check data view.
3. Check field name.
4. Check field type.
5. Check environment.
6. Check namespace.
7. Check index/data stream.
8. Check Elasticsearch directly.
```

Do not immediately assume the logs are missing.

---

# 92. Kibana and Index Mapping

Suppose:

```text
status_code
```

is stored as:

```text
keyword
```

instead of:

```text
integer
```

Numeric aggregations may not behave as expected.

This is why correct mappings are important.

---

# 93. Kibana and Elasticsearch Mapping

Flow:

```text
Application
 ↓
Logstash
 ↓
Elasticsearch Mapping
 ↓
Kibana
```

The field type determines how Kibana can use the data.

Examples:

```text
integer
date
keyword
text
boolean
```

---

# 94. Operational Workflow

When a production incident occurs:

```text
1. Alert fires
        ↓
2. Grafana shows metric anomaly
        ↓
3. Open Kibana
        ↓
4. Search logs
        ↓
5. Filter service / namespace / pod
        ↓
6. Identify error
        ↓
7. Find trace/request ID
        ↓
8. Investigate trace in Jaeger
        ↓
9. Identify root cause
        ↓
10. Remediate
```

This is a practical observability workflow.

---

# 95. Example Production Incident

Suppose:

```text
Payment 5xx rate ↑
```

Prometheus/Grafana shows:

```text
5xx = 20%
```

Open Kibana.

Search:

```text
service.name : "payment"
and log.level : "ERROR"
```

You discover:

```text
Database connection timeout
```

The log contains:

```text
trace.id = abc123
```

Search the trace:

```text
trace.id : "abc123"
```

Then inspect Jaeger.

You discover:

```text
Payment
  ↓
Database
  ↓
Timeout
```

Now the incident has moved from:

```text
Symptom
```

to:

```text
Root cause
```

---

# 96. Kibana Monitoring

Monitor Kibana itself.

Important areas:

```text
Kibana availability
Response time
CPU
Memory
Pod restarts
Elasticsearch connectivity
Query performance
```

For Kubernetes:

```text
Kibana Pod
 ↓
Prometheus
 ↓
Grafana
```

---

# 97. Kibana Logs

Kibana logs can reveal:

```text
Elasticsearch connection failures
Authentication problems
Plugin errors
Configuration problems
Slow requests
Startup failures
```

Monitor these logs as part of the overall observability architecture.

---

# 98. Kibana Troubleshooting Flow

If Kibana is unavailable:

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

Check each layer.

---

# 99. Kibana Pod Not Ready

Check:

```bash
kubectl get pods -n logging
```

Then:

```bash
kubectl describe pod <kibana-pod> -n logging
```

Then:

```bash
kubectl logs <kibana-pod> -n logging
```

Look for:

```text
Configuration
Elasticsearch
TLS
Authentication
Memory
Probe failures
```

---

# 100. Kibana CrashLoopBackOff

Check:

```bash
kubectl logs <kibana-pod> -n logging --previous
```

Then:

```bash
kubectl describe pod <kibana-pod> -n logging
```

Check:

```text
Configuration
Environment variables
Secrets
Elasticsearch connection
Memory
```

---

# 101. Kibana Cannot Reach Elasticsearch

Check:

```text
Kibana Pod
 ↓
DNS
 ↓
Service
 ↓
NetworkPolicy
 ↓
Security Group
 ↓
Elasticsearch
```

Test from the Kibana Pod if appropriate:

```bash
kubectl exec -it <kibana-pod> -n logging -- sh
```

Then test connectivity using tools available in the container.

---

# 102. Kibana Authentication Failure

Symptoms may include:

```text
401
403
```

Check:

```text
Kibana credentials
Service identity
Elasticsearch security
Certificate
Permissions
```

---

# 103. Kibana Memory Pressure

Symptoms:

```text
Pod restarts
OOMKilled
Slow UI
Long garbage collection
```

Check:

```bash
kubectl describe pod <kibana-pod> -n logging
```

Look for:

```text
Reason: OOMKilled
```

Then review:

```text
Memory requests
Memory limits
Dashboard complexity
Concurrent users
Query volume
```

---

# 104. Kibana High CPU

Possible causes:

```text
Heavy dashboards
Many concurrent users
Expensive searches
Large aggregations
Elasticsearch latency
```

Investigate both:

```text
Kibana
```

and:

```text
Elasticsearch
```

because Kibana often waits for Elasticsearch query results.

---

# 105. Kibana Dashboard Empty

Check:

```text
1. Time range
2. Data view
3. Index/data stream
4. Elasticsearch data
5. Field names
6. Environment
7. Dashboard filters
```

A common mistake is:

```text
Last 15 minutes
```

when the logs were generated:

```text
Yesterday
```

---

# 106. Kibana Dashboard Slow

Check:

```text
Time range
Number of panels
Query complexity
Aggregations
Elasticsearch performance
Shard count
High-cardinality fields
```

Optimize the dashboard before simply increasing Kibana resources.

---

# 107. Production Checklist

```text
[ ] Compatible Elastic Stack versions
[ ] Kibana installed
[ ] kibana.yml configured
[ ] Elasticsearch connectivity
[ ] TLS configured
[ ] Authentication configured
[ ] RBAC configured
[ ] Internal networking
[ ] ALB / Ingress configured
[ ] DNS configured
[ ] Data views created
[ ] Time field configured
[ ] Dashboards created
[ ] Alerts configured
[ ] Monitoring configured
[ ] High availability
[ ] Backup/export strategy
[ ] GitOps deployment
```

---

# 108. Recommended Kibana Repository Structure

For your GitOps repository:

```text
observability/
│
├── kibana/
│   ├── base/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap.yaml
│   │   ├── ingress.yaml
│   │   └── rbac.yaml
│   │
│   └── overlays/
│       ├── dev/
│       ├── staging/
│       └── prod/
```

If using Helm:

```text
kibana/
├── Chart.yaml
├── values.yaml
└── templates/
```

---

# 109. GitHub Actions Integration

A pipeline can validate:

```text
Pull Request
    ↓
YAML validation
    ↓
Helm validation
    ↓
Security scan
    ↓
Kibana configuration validation
    ↓
Merge
```

Then:

```text
ArgoCD
   ↓
EKS
   ↓
Kibana
```

---

# 110. GitOps Deployment

Your production flow:

```text
Developer
   ↓
GitHub
   ↓
Pull Request
   ↓
GitHub Actions
   ├── Validate
   ├── Test
   └── Security scan
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Kibana
```

This avoids manual production changes.

---

# 111. Kibana Production Architecture

```text
                         USERS
                           │
                           ↓
                   Internal Route53
                           │
                           ↓
                    Internal AWS ALB
                           │
                   ┌───────┴───────┐
                   ↓               ↓
               Kibana-01       Kibana-02
                   │               │
                   └───────┬───────┘
                           ↓
                 Elasticsearch Cluster
                ┌──────────┼──────────┐
                ↓          ↓          ↓
              ES-01      ES-02      ES-03
                ↑          ↑          ↑
                └──────────┼──────────┘
                           ↑
                       Logstash
                           ↑
                       Fluent Bit
                           ↑
                         EKS
```

---

# 112. Complete Observability Architecture

Your overall architecture becomes:

```text
                          USERS
                            │
              ┌─────────────┴─────────────┐
              ↓                           ↓
           Grafana                     Kibana
              │                           │
              ↓                           ↓
         Prometheus                Elasticsearch
              ↑                           ↑
              │                           │
          Metrics                    Logstash
              ↑                           ↑
              │                       Fluent Bit
              │                           ↑
              │                          EKS
              │                           ↑
              └──────── Applications ─────┘
                            │
                            ↓
                     OpenTelemetry
                            │
                            ↓
                          Jaeger
```

This gives you:

```text
Prometheus
   ↓
Metrics

Elasticsearch + Kibana
   ↓
Logs

OpenTelemetry + Jaeger
   ↓
Traces
```

---

# 113. Final Mental Model

Remember Kibana as:

```text
                    KIBANA
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Discover    Dashboards    Alerts
          │            │            │
          └────────────┼────────────┘
                       ↓
                Elasticsearch
                       ↓
                     Logs
```

And your production logging architecture:

```text
EKS
 ↓
Fluent Bit
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

With the full observability stack:

```text
Metrics  → Prometheus → Grafana

Logs     → Logstash → Elasticsearch → Kibana

Traces   → OpenTelemetry → Jaeger
```

The key principle is:

**Kibana is the investigation and visualization layer for Elasticsearch data. In production, secure it with TLS, authentication and RBAC, keep it on private networking, connect it to Elasticsearch through secure service identities, organize dashboards around operational questions, deploy it through GitOps, and use it together with Prometheus/Grafana and OpenTelemetry/Jaeger to investigate incidents across metrics, logs, and traces.**
