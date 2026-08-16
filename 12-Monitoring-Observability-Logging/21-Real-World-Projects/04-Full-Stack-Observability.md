# Full-Stack Observability

> Production-grade end-to-end observability architecture for AWS EKS microservices combining Prometheus, Grafana, ELK, OpenTelemetry and Jaeger.
>
> This project connects metrics, logs and traces into one operational workflow and focuses on architecture, implementation, production operations, security, scalability, incident response and DevOps interview preparation.

---

# 1. Project Overview

## Project Name

**Full-Stack Observability Platform for EKS Microservices**

## Objective

Build a unified observability platform for a production-style microservices application running on Amazon EKS.

The platform should answer three questions:

```text
Metrics -> What is happening?
Logs   -> What happened?
Traces -> Where did it happen?
```

Technology stack:

```text
AWS
  |
  +--> EKS
  +--> ALB
  +--> EC2 worker nodes
  +--> RDS / databases
  +--> ECR

Observability
  |
  +--> Prometheus
  +--> Grafana
  +--> ELK
  +--> OpenTelemetry
  +--> Jaeger
```

---

# 2. Why Full-Stack Observability?

Monitoring only infrastructure is not enough.

Example:

```text
Node CPU = 40%
Node Memory = 55%
Pod CPU = 35%
Pod Memory = 50%
```

Everything appears healthy.

But users report:

```text
Checkout is slow.
```

Metrics may show:

```text
P99 latency = 4 seconds
```

Tracing may show:

```text
Payment Service = 3.5 seconds
```

Logs may show:

```text
Payment provider timeout
```

The complete stack connects:

```text
Detect -> Locate -> Explain
```

---

# 3. Three Pillars

## Metrics

Metrics are numerical time-series data.

Examples:

```text
CPU
Memory
Request rate
Error rate
Latency
Pod restarts
Node availability
Database connections
```

Primary tools:

```text
Prometheus
Grafana
```

---

# 4. Logs

Logs contain detailed event information.

Examples:

```text
Application errors
Authentication failures
Database errors
Deployment messages
HTTP requests
Stack traces
```

Primary tools:

```text
Elasticsearch
Logstash
Kibana
```

---

# 5. Traces

Traces follow a request across services.

Example:

```text
ALB
 |
 v
Order
 |
 +--> User
 |
 +--> Payment
       |
       +--> Database
```

Primary tools:

```text
OpenTelemetry
Jaeger
```

---

# 6. Unified Observability Architecture

```text
                         USERS
                           |
                           v
                      AWS ALB
                           |
                           v
                    EKS APPLICATIONS
             +-------------+-------------+
             |             |             |
             v             v             v
          Metrics         Logs         Traces
             |             |             |
             v             v             v
        Prometheus      Log agents    OTel SDK
             |             |             |
             v             v             v
          Grafana      Logstash       OTel Collector
                           |             |
                           v             v
                    Elasticsearch      Jaeger
                           |             |
                           v             v
                         Kibana      Jaeger UI
             \              |             /
              \             |            /
               +------------+-----------+
                            |
                       OPERATIONS TEAM
```

---

# 7. Production Architecture

A more complete architecture:

```text
                         Internet
                            |
                            v
                      AWS ALB / Ingress
                            |
                            v
                  +---------------------+
                  | EKS Cluster          |
                  |                     |
                  | Order               |
                  | User                |
                  | Payment             |
                  | Inventory           |
                  | Notification        |
                  +---------------------+
                     |       |       |
             +-------+       |       +--------+
             |               |                |
             v               v                v
        Prometheus       OTel SDK        Container Logs
             |               |                |
             v               v                v
          Grafana        OTel Collector     Log Collector
                             |                |
                             v                v
                           Jaeger          Logstash
                                              |
                                              v
                                        Elasticsearch
                                              |
                                              v
                                            Kibana
```

---

# 8. Observability Data Flow

## Metrics

```text
Application / Node / Kubernetes
            |
            v
       Prometheus
            |
            v
         Grafana
```

## Logs

```text
Container stdout/stderr
            |
            v
       Log Collector
            |
            v
         Logstash
            |
            v
      Elasticsearch
            |
            v
          Kibana
```

## Traces

```text
Application
     |
     v
OpenTelemetry
     |
     v
OTel Collector
     |
     v
Jaeger
     |
     v
Jaeger UI
```

---

# 9. Observability Platform as a System

The observability stack itself must be production-grade.

Monitor:

```text
Prometheus
Grafana
Log collectors
Logstash
Elasticsearch
Kibana
OTel Collector
Jaeger
Storage
Network
```

Otherwise:

```text
Application fails
       |
       v
Observability platform also fails
       |
       v
No visibility
```

---

# 10. Project Goals

The platform should provide:

```text
Infrastructure monitoring
Kubernetes monitoring
Application metrics
Centralized logging
Distributed tracing
Alerting
Dashboards
Correlation
Incident response
SLO visibility
Security
Retention
Cost control
Disaster recovery
```

---

# 11. EKS Environment

Example:

```text
AWS Account
 |
 +--> VPC
      |
      +--> Public Subnets
      |
      +--> Private Subnets
             |
             +--> EKS
                  |
                  +--> Worker Nodes
                  |
                  +--> Applications
                  |
                  +--> Observability
```

Keep production workloads and observability components inside controlled private network boundaries where practical.

---

# 12. Kubernetes Namespaces

Example:

```text
production
monitoring
logging
tracing
```

Applications:

```text
production
```

Metrics:

```text
monitoring
```

Logging:

```text
logging
```

Tracing:

```text
tracing
```

This provides separation of responsibility and access control.

---

# 13. Namespace Design

Example:

```text
production
 ├── order-service
 ├── user-service
 ├── payment-service
 └── inventory-service

monitoring
 ├── prometheus
 └── grafana

logging
 ├── log collector
 ├── logstash
 ├── elasticsearch
 └── kibana

tracing
 ├── otel-collector
 └── jaeger
```

---

# 14. Metrics Architecture

```text
Kubernetes
   |
   +--> kube-state-metrics
   |
   +--> node exporter
   |
   +--> application /metrics
   |
   v
Prometheus
   |
   v
Grafana
```

Prometheus collects:

```text
Infrastructure metrics
Kubernetes object metrics
Application metrics
Custom metrics
```

---

# 15. Prometheus Responsibilities

Prometheus is responsible for:

```text
Scraping
Storage
PromQL
Recording rules
Alert rules
Service discovery
Metric evaluation
```

Grafana is primarily responsible for:

```text
Visualization
Dashboards
Exploration
Alert presentation
```

---

# 16. Application Metrics

Expose useful metrics such as:

```text
HTTP request count
HTTP error count
Request duration
Active requests
Database connection pool
Business transaction count
```

Use appropriate labels.

Avoid uncontrolled high-cardinality labels.

---

# 17. RED Method

For services:

```text
Rate
Errors
Duration
```

Example:

```text
http_requests_total
http_request_errors_total
http_request_duration_seconds
```

This is useful for application dashboards.

---

# 18. USE Method

For infrastructure:

```text
Utilization
Saturation
Errors
```

Examples:

```text
CPU utilization
Disk saturation
Network errors
Memory pressure
```

---

# 19. Golden Signals

The four golden signals:

```text
Latency
Traffic
Errors
Saturation
```

Use them as the foundation for service dashboards.

---

# 20. Kubernetes Metrics

Monitor:

```text
Pods
Deployments
Nodes
Namespaces
CPU
Memory
Restarts
Scheduling
Resource requests/limits
```

Important questions:

```text
Are pods restarting?
Are nodes under pressure?
Are replicas available?
Are workloads pending?
```

---

# 21. Node Monitoring

Monitor:

```text
CPU
Memory
Disk
Filesystem
Network
Load
I/O
```

Example:

```text
node_filesystem_avail_bytes
node_memory_MemAvailable_bytes
node_cpu_seconds_total
```

---

# 22. Application Logging Architecture

```text
Application
    |
    v
stdout/stderr
    |
    v
Container runtime
    |
    v
Node log files
    |
    v
Log collector
    |
    v
Logstash
    |
    v
Elasticsearch
    |
    v
Kibana
```

Prefer applications logging to stdout/stderr in Kubernetes rather than writing local application log files unless there is a specific requirement.

---

# 23. Structured Logging

Prefer:

```json
{
  "timestamp": "2026-08-15T14:00:00Z",
  "level": "ERROR",
  "service": "payment-service",
  "environment": "production",
  "trace_id": "abc123",
  "span_id": "def456",
  "message": "Payment provider timeout"
}
```

Instead of:

```text
Payment failed at 14:00
```

Structured logs are easier to search and correlate.

---

# 24. Log Levels

Use:

```text
DEBUG
INFO
WARN
ERROR
```

Production should avoid excessive DEBUG logging unless temporarily enabled in a controlled way.

---

# 25. Log Collector

A Kubernetes log collector can run as a DaemonSet.

Architecture:

```text
Node 1 -> Collector
Node 2 -> Collector
Node 3 -> Collector
```

Each collector reads node-local container logs.

---

# 26. Logstash Role

Logstash can process:

```text
Parsing
Filtering
Enrichment
Transformation
Routing
```

Example:

```text
Raw log
  |
  v
JSON parse
  |
  v
Kubernetes metadata
  |
  v
Redaction
  |
  v
Index
```

---

# 27. Elasticsearch Role

Elasticsearch provides:

```text
Indexing
Storage
Search
Aggregation
Filtering
```

Plan:

```text
Shards
Replicas
Retention
Storage
Index naming
Lifecycle
```

---

# 28. Kibana Role

Kibana provides:

```text
Search
Dashboards
Discover
Visualizations
Operational analysis
```

Example:

```text
service = payment-service
level = ERROR
trace_id = abc123
```

---

# 29. Tracing Architecture

```text
Application
 |
 v
OpenTelemetry SDK / Agent
 |
 v
OTLP
 |
 v
OTel Collector
 |
 +--> Batch
 +--> Sampling
 +--> Enrichment
 +--> Retry
 |
 v
Jaeger
 |
 v
Storage
 |
 v
Jaeger UI
```

---

# 30. Trace Context

Every distributed request should maintain:

```text
Trace ID
Span ID
Parent relationship
```

Example:

```text
Order
 |
 +--> Payment
       |
       +--> Database
```

All belong to the same trace.

---

# 31. Trace-to-Log Correlation

Application logs should contain:

```text
trace_id
span_id
```

Then:

```text
Jaeger
 |
 | trace_id
 v
Kibana
 |
 v
Detailed log
```

This is one of the most valuable integrations in the stack.

---

# 32. Metrics-to-Trace Correlation

Suppose Grafana shows:

```text
payment-service P99 = 4 sec
```

Use Jaeger to inspect:

```text
slow payment traces
```

Then:

```text
Find slow dependency
```

---

# 33. Logs-to-Trace Correlation

Suppose Kibana shows:

```text
Payment provider timeout
trace_id=abc123
```

Open:

```text
Jaeger -> trace abc123
```

Then identify:

```text
Payment span
External API span
Database span
```

---

# 34. Full Correlation Workflow

```text
Grafana
   |
   | Detect
   v
Jaeger
   |
   | Locate
   v
Kibana
   |
   | Explain
   v
Root Cause
```

---

# 35. Correlation Example

Incident:

```text
Checkout is slow.
```

Step 1:

```text
Grafana
P99 checkout = 4.2 sec
```

Step 2:

```text
Jaeger
Payment span = 3.8 sec
```

Step 3:

```text
Kibana
trace_id=abc123
Payment provider timeout
```

Root cause:

```text
External payment dependency latency
```

---

# 36. Dashboard Strategy

Create separate dashboards for:

```text
Executive overview
Platform
Kubernetes
Application
Database
Logs
Traces
SLO
Observability platform
```

Do not put every metric on one dashboard.

---

# 37. Executive Dashboard

Show:

```text
Availability
Request rate
Error rate
P95/P99 latency
SLO status
Active incidents
```

Keep it high level.

---

# 38. Platform Dashboard

Show:

```text
Node CPU
Node memory
Node disk
Network
Pod health
Cluster capacity
```

---

# 39. Kubernetes Dashboard

Show:

```text
Pod count
Running pods
Pending pods
Failed pods
Restarts
OOMKilled
CrashLoopBackOff
Node pressure
Deployment availability
```

---

# 40. Application Dashboard

For each service:

```text
Requests/sec
Errors/sec
P50
P95
P99
Active requests
Dependency latency
```

---

# 41. Database Dashboard

Monitor:

```text
Connections
CPU
Memory
Latency
Query rate
Slow queries
Locks
Storage
Replication health
```

---

# 42. Logging Dashboard

Show:

```text
Total logs
ERROR rate
WARN rate
Top services
Top error messages
Logs by namespace
Logs by pod
```

---

# 43. Tracing Dashboard

Show:

```text
Trace volume
Error traces
Slow traces
Top services
Top operations
P95/P99 duration
Collector health
Export failures
```

---

# 44. SLO Dashboard

Show:

```text
SLO target
Current performance
Error budget
Burn rate
Availability
Latency compliance
```

Example:

```text
Availability SLO = 99.9%
Current = 99.94%
Status = Healthy
```

---

# 45. Alerting Strategy

Alerts should represent actionable conditions.

Bad:

```text
CPU = 70%
```

Better:

```text
CPU saturation sustained
AND
application latency is increasing
```

---

# 46. Alert Categories

```text
Availability
Latency
Errors
Saturation
Capacity
Security
Observability platform
SLO burn
```

---

# 47. Alert Severity

Example:

```text
Critical
Warning
Info
```

Critical alerts should require immediate action.

---

# 48. Alert Routing

Example:

```text
Prometheus
   |
   v
Alertmanager
   |
   +--> Critical -> On-call
   |
   +--> Warning -> Team channel
```

---

# 49. Alert Fatigue

Avoid:

```text
100 alerts/day
```

with no action.

Prefer:

```text
Few high-value alerts
```

Every alert should answer:

```text
What is broken?
Why does it matter?
What should I do?
```

---

# 50. SLI Design

Examples:

```text
Availability
Latency
Successful requests
```

SLI:

```text
successful requests / total requests
```

Latency SLI:

```text
requests under threshold / eligible requests
```

---

# 51. SLO Example

```text
99.9% availability
99% of requests < 500ms
```

Metrics evaluate the SLO.

Traces explain violations.

Logs support investigation.

---

# 52. Error Budget

For:

```text
99.9% SLO
```

allowed failure is approximately:

```text
0.1%
```

Use the error budget to balance:

```text
Reliability
vs
Delivery speed
```

---

# 53. Burn Rate

Fast burn:

```text
SLO violation increasing quickly
```

Slow burn:

```text
Long-term gradual degradation
```

Use burn-rate alerts rather than only instantaneous threshold alerts.

---

# 54. Observability Platform Monitoring

Monitor:

```text
Prometheus
Grafana
Elasticsearch
Logstash
Kibana
OTel Collector
Jaeger
```

Key signals:

```text
CPU
Memory
Disk
Queue
Dropped data
Export errors
Latency
Storage
```

---

# 55. Prometheus Self-Monitoring

Monitor:

```text
Scrape failures
Target availability
Rule evaluation
TSDB storage
WAL behavior
Query latency
Memory
CPU
```

---

# 56. Logstash Self-Monitoring

Monitor:

```text
Events in
Events out
Pipeline latency
Queue
CPU
Memory
Pipeline errors
```

---

# 57. Elasticsearch Self-Monitoring

Monitor:

```text
Cluster health
Heap
CPU
Disk
Shard state
Indexing rate
Search latency
Rejected operations
```

---

# 58. OTel Collector Self-Monitoring

Monitor:

```text
Received spans
Exported spans
Dropped spans
Queue size
Export errors
CPU
Memory
```

---

# 59. Jaeger Self-Monitoring

Monitor:

```text
Ingestion
Query latency
Storage
Errors
CPU
Memory
```

---

# 60. Data Retention

Each telemetry type has different retention requirements.

Example:

```text
Metrics -> weeks/months
Logs -> days/weeks
Traces -> days/weeks
```

Actual policy depends on:

```text
Incident requirements
Compliance
Cost
Traffic
Business needs
```

---

# 61. Cost Optimization

Main cost drivers:

```text
Metrics cardinality
Log volume
Trace volume
Storage
Retention
Network traffic
Query workload
```

---

# 62. Metrics Cost Optimization

Reduce:

```text
Unnecessary metrics
High-cardinality labels
Duplicate scraping
Excessive retention
```

Use:

```text
Recording rules
Appropriate scrape intervals
Controlled label design
```

---

# 63. Logging Cost Optimization

Reduce:

```text
DEBUG volume
Duplicate logs
Large payloads
Unnecessary health-check logs
Long retention
```

Use:

```text
Structured logs
Filtering
Sampling where appropriate
Retention policies
```

---

# 64. Tracing Cost Optimization

Use:

```text
Sampling
Tail sampling
Error priority
Slow-trace priority
Attribute control
Retention policies
```

---

# 65. High Cardinality

Dangerous metric label:

```text
user_id
```

If millions of users exist:

```text
Millions of time series
```

Prefer bounded labels:

```text
service
route
method
status_class
```

---

# 66. High-Cardinality Logs

Logs can also explode in volume through:

```text
Request bodies
Large IDs
Debug payloads
Repeated stack traces
```

Filter carefully.

---

# 67. High-Cardinality Traces

Avoid large dynamic span attributes.

Bad:

```text
full request JSON
```

Better:

```text
order.id
```

provided it is allowed and appropriate.

---

# 68. Security Architecture

```text
Users
 |
 v
Authentication
 |
 v
Grafana / Kibana / Jaeger
 |
 v
RBAC
 |
 v
Observability data
```

Restrict production access.

---

# 69. TLS

Secure:

```text
Application -> Collector
Collector -> Backend
User -> UI
Backend -> Storage
```

TLS configuration depends on the deployment and trust model.

---

# 70. Secrets

Never commit:

```text
Passwords
API keys
TLS private keys
Cloud credentials
Backend credentials
```

Use:

```text
Kubernetes Secrets
AWS Secrets Manager
External Secrets
```

where appropriate.

---

# 71. Network Policies

Example:

```text
production
   |
   +--> monitoring
   +--> tracing
   +--> logging
```

Only required communication should be allowed.

---

# 72. RBAC

Examples:

```text
Developers -> application logs/traces
Platform team -> infrastructure
SRE -> full observability
Security -> security-related data
```

Use least privilege.

---

# 73. PII Protection

Do not collect:

```text
Passwords
Tokens
Payment card data
Sensitive request bodies
```

unless explicitly required and governed.

Redact sensitive fields.

---

# 74. Auditability

Track:

```text
Configuration changes
Dashboard changes
Alert changes
Access
Retention changes
Collector changes
```

GitOps provides strong configuration auditability.

---

# 75. GitOps Architecture

```text
Git
 |
 +--> Prometheus configuration
 +--> Grafana dashboards
 +--> Alert rules
 +--> ELK configuration
 +--> OTel Collector
 +--> Jaeger
 |
 v
CI
 |
 v
ArgoCD
 |
 v
EKS
```

---

# 76. Repository Structure

```text
observability/
│
├── prometheus/
│   ├── values.yaml
│   ├── rules/
│   └── servicemonitors/
│
├── grafana/
│   ├── dashboards/
│   └── datasources/
│
├── logging/
│   ├── collector/
│   ├── logstash/
│   ├── elasticsearch/
│   └── kibana/
│
├── tracing/
│   ├── otel-collector/
│   └── jaeger/
│
├── alerts/
│
├── policies/
│
└── README.md
```

---

# 77. CI Pipeline

A production observability pipeline can validate:

```text
YAML
Helm
Kubernetes manifests
Prometheus rules
Collector config
Security policies
```

Then:

```text
Deploy to staging
Generate telemetry
Run smoke tests
Verify dashboards
Verify alerts
```

---

# 78. Deployment Strategy

Recommended:

```text
Git
 |
 v
Pull Request
 |
 v
Review
 |
 v
CI validation
 |
 v
Staging
 |
 v
Observability tests
 |
 v
ArgoCD production
```

---

# 79. Smoke Tests

After deployment verify:

```text
Prometheus targets UP
Grafana datasource healthy
Logs reaching Elasticsearch
Kibana searches working
Traces visible in Jaeger
Trace IDs in logs
Alerts evaluating
```

---

# 80. End-to-End Health Check

Test:

```text
Client
 |
 v
ALB
 |
 v
Application
 |
 +--> Metrics
 |
 +--> Logs
 |
 +--> Traces
```

Expected:

```text
Metric appears in Prometheus
Log appears in Kibana
Trace appears in Jaeger
```

---

# 81. Correlation Test

Generate one request.

Capture:

```text
trace_id
```

Then verify:

```text
Jaeger -> trace
Kibana -> same trace_id
Grafana -> service latency
```

This validates the integration.

---

# 82. Incident Scenario — High Latency

Alert:

```text
P99 > 2 seconds
```

Workflow:

```text
Grafana
 |
 v
payment-service
 |
 v
Jaeger
 |
 v
external payment API
 |
 v
Kibana
 |
 v
provider timeout
```

Action:

```text
Failover / rollback / dependency mitigation
```

---

# 83. Incident Scenario — Error Spike

Metrics:

```text
HTTP 5xx ↑
```

Trace:

```text
Payment span ERROR
```

Log:

```text
Database connection timeout
```

Root cause:

```text
Database connectivity issue
```

---

# 84. Incident Scenario — Pod CrashLoopBackOff

Kubernetes:

```text
Pod restarting
```

Metrics:

```text
Restart count ↑
```

Logs:

```text
Configuration error
```

Tracing:

```text
Requests fail before downstream calls
```

The three signals provide complementary evidence.

---

# 85. Incident Scenario — OOMKilled

Metrics:

```text
Container memory ↑
```

Kubernetes:

```text
OOMKilled
```

Logs:

```text
Application terminated
```

Tracing:

```text
Requests fail / traces incomplete
```

Investigate:

```text
Memory limit
Heap
Traffic
Leak
Recent deployment
```

---

# 86. Incident Scenario — Node Disk Pressure

Metrics:

```text
Filesystem available ↓
```

Kubernetes:

```text
DiskPressure=True
```

Logs:

```text
Container/logging errors
```

Possible causes:

```text
Container logs
Elasticsearch storage
Images
Temporary files
```

---

# 87. Incident Scenario — Elasticsearch Disk Full

Metrics:

```text
Disk usage > threshold
```

Logs:

```text
Indexing failures
```

Impact:

```text
Logs missing
```

Tracing/metrics may continue working, creating partial observability.

Action:

```text
Control log volume
Expand storage
Review retention
Restore indexing
```

---

# 88. Incident Scenario — Prometheus Storage Full

Impact:

```text
Metrics retention problem
Queries fail
Alerts may be affected
```

Action:

```text
Check TSDB
Retention
Cardinality
Storage
Scrape configuration
```

---

# 89. Incident Scenario — Collector Overload

Signals:

```text
Collector CPU ↑
Memory ↑
Queue ↑
Dropped spans ↑
```

Possible causes:

```text
Traffic spike
Tail sampling
Backend outage
Large spans
```

Action:

```text
Scale collector
Control sampling
Restore backend
```

---

# 90. Incident Scenario — Log Pipeline Backpressure

```text
Application
 |
 v
Collector
 |
 v
Logstash
 |
 X
Elasticsearch
```

Symptoms:

```text
Queue growth
Log delay
Indexing errors
```

Investigate from the backend backward.

---

# 91. Backpressure Principle

When a downstream component slows:

```text
Application
    |
    v
Collector
    |
    v
Queue
    |
    v
Backend
```

Bound the queue.

Monitor it.

Do not let memory grow indefinitely.

---

# 92. Observability Failure Matrix

| Component | Failure | Impact |
|---|---|---|
| Prometheus | Down | Metrics/alerts affected |
| Grafana | Down | Visualization affected |
| Log collector | Down | Logs may be lost/delayed |
| Logstash | Down | Log processing affected |
| Elasticsearch | Down | Log storage/search affected |
| Kibana | Down | Log UI affected |
| OTel Collector | Down | Traces affected |
| Jaeger | Down | Trace analysis affected |

A mature platform designs for each failure.

---

# 93. High Availability

## Prometheus

Use an HA strategy appropriate to scale and requirements.

## Grafana

Use multiple replicas where required.

## Elasticsearch

Use multiple nodes and replicas.

## Logstash

Use multiple pipeline instances.

## OTel Collector

Use multiple replicas.

## Jaeger

Use a production-supported highly available architecture.

---

# 94. Failure Domain Design

Avoid putting all replicas on one node.

Spread across:

```text
Nodes
Availability Zones
Failure domains
```

Use:

```text
Pod anti-affinity
Topology spread constraints
PodDisruptionBudget
```

where appropriate.

---

# 95. Disaster Recovery

Back up:

```text
Elasticsearch data/configuration as appropriate
Grafana configuration
Dashboards
Alert rules
Prometheus configuration
Collector configuration
Jaeger storage/configuration
```

GitOps repositories should contain recoverable configuration.

---

# 96. DR Priority

During a major incident:

```text
1. Application availability
2. Critical metrics
3. Critical logs
4. Critical traces
5. Full historical data
```

Prioritize according to business requirements.

---

# 97. Backup Testing

A backup is not enough.

Test:

```text
Restore
Validation
Data integrity
Recovery time
Recovery point
```

Measure:

```text
RTO
RPO
```

---

# 98. Observability RTO

Example:

```text
Restore critical dashboards and alerts within 30 minutes.
```

The exact target should be defined by the organization.

---

# 99. Observability RPO

Example:

```text
Maximum acceptable telemetry loss = 15 minutes
```

Again, this depends on business and incident requirements.

---

# 100. Scaling Metrics

Prometheus scaling depends on:

```text
Samples/sec
Series count
Query load
Retention
Storage
```

---

# 101. Scaling Logs

ELK scaling depends on:

```text
Events/sec
Average log size
Indexing rate
Shard count
Search rate
Retention
Storage
```

---

# 102. Scaling Traces

Tracing depends on:

```text
Requests/sec
Spans/request
Sampling
Span size
Collector throughput
Backend ingestion
Retention
```

---

# 103. Capacity Planning

## Metrics

```text
samples/sec
series count
storage/day
```

## Logs

```text
events/sec
bytes/sec
storage/day
```

## Traces

```text
spans/sec
bytes/sec
storage/day
```

Plan each independently.

---

# 104. Cost Model

Observability cost roughly comes from:

```text
Compute
+
Storage
+
Network
+
Retention
+
Queries
```

Control:

```text
Data volume
Cardinality
Sampling
Retention
Replica count
```

---

# 105. Observability Data Classification

Classify data:

```text
Metrics -> operational
Logs -> operational + potentially sensitive
Traces -> operational + potentially sensitive
```

Security policies should reflect this.

---

# 106. Access Control

Example:

```text
Developer
  |
  +--> Application dashboards
  +--> Application logs
  +--> Application traces

Platform Engineer
  |
  +--> Infrastructure
  +--> Kubernetes
  +--> Observability platform

Security
  |
  +--> Security logs
  +--> Audit data
```

Least privilege remains important.

---

# 107. Production Best Practice — Standard Naming

Use consistent:

```text
service.name
namespace
cluster
environment
version
region
```

This makes cross-tool filtering easier.

---

# 108. Production Best Practice — Standard Labels

Metrics:

```text
service
namespace
environment
method
route
status
```

Logs:

```text
service
namespace
pod
container
level
trace_id
```

Traces:

```text
service.name
service.version
deployment.environment
```

---

# 109. Production Best Practice — Avoid Duplicates

Do not accidentally:

```text
scrape same target twice
ship same logs twice
instrument same operation twice
```

Duplicate telemetry increases:

```text
Cost
Storage
Query complexity
```

---

# 110. Production Best Practice — Standard Dashboards

Every production service should have:

```text
Request rate
Error rate
P95/P99
Saturation
Dependency latency
Pod health
```

---

# 111. Production Best Practice — Standard Runbooks

Every critical alert should link to:

```text
Runbook
Dashboard
Logs
Traces
Relevant service owner
```

---

# 112. Production Best Practice — Service Ownership

Maintain:

```text
Service
Owner
Repository
Dashboard
Runbook
SLO
Dependencies
```

This reduces incident response time.

---

# 113. Production Best Practice — Change Correlation

Always correlate incidents with:

```text
Deployment
Configuration
Infrastructure changes
Database changes
Dependency changes
```

Use:

```text
service.version
deployment timestamps
Git commit
```

where appropriate.

---

# 114. Production Best Practice — Annotations

Mark dashboards with:

```text
Deployment
Rollback
Incident
Maintenance
```

Then engineers can visually correlate changes with metric behavior.

---

# 115. Production Best Practice — Synthetic Monitoring

For critical user journeys:

```text
Login
Search
Checkout
Payment
```

run synthetic checks.

This detects:

```text
User-facing failures
```

even when internal infrastructure looks healthy.

---

# 116. Synthetic + Real User Monitoring

Where appropriate:

```text
Synthetic
+
Real traffic
+
Infrastructure
+
Logs
+
Traces
```

gives broader coverage.

---

# 117. Dependency Monitoring

Track:

```text
Service -> Service
Service -> Database
Service -> External API
```

Metrics and traces together can identify dependency health.

---

# 118. Service Dependency Map

Example:

```text
Order
 |
 +--> User
 |
 +--> Payment
 |      |
 |      +--> Provider
 |
 +--> Inventory
        |
        +--> Database
```

Maintain this conceptually through telemetry rather than relying only on manually drawn diagrams.

---

# 119. External Dependency

Monitor:

```text
Latency
Errors
Timeouts
Retries
Availability
```

Tracing should capture external dependency calls where appropriate.

---

# 120. Database Dependency

Monitor:

```text
Connection pool
Latency
Errors
Query rate
Slow queries
```

Correlate:

```text
Database metric
+
Trace database span
+
Application log
```

---

# 121. Queue Dependency

For asynchronous systems:

```text
Producer
 |
 v
Queue
 |
 v
Consumer
```

Monitor:

```text
Queue depth
Age of oldest message
Consumer rate
Failures
Retries
```

Trace context should be propagated through supported messaging instrumentation where applicable.

---

# 122. Async Trace Challenges

Unlike synchronous HTTP:

```text
Producer
   |
   v
Queue
   |
   v
Consumer
```

there is a time gap.

Context propagation must be handled through message metadata.

---

# 123. Full-Stack Async Workflow

```text
Order Service
    |
    v
RabbitMQ / Queue
    |
    v
Inventory Consumer
    |
    v
Database
```

Metrics:

```text
queue depth
```

Logs:

```text
consumer errors
```

Traces:

```text
message processing path
```

Together they explain delayed processing.

---

# 124. Incident Scenario — Queue Backlog

Metrics:

```text
queue depth ↑
```

Tracing:

```text
consumer processing duration ↑
```

Logs:

```text
database timeout
```

Root cause:

```text
Database dependency slowing consumers.
```

---

# 125. Full-Stack Troubleshooting Method

Always move from:

```text
Symptom
 |
 v
Metric
 |
 v
Trace
 |
 v
Log
 |
 v
Dependency
 |
 v
Root cause
```

Do not immediately jump into random logs.

---

# 126. Troubleshooting Example

Symptom:

```text
Users report slow checkout.
```

Metric:

```text
Checkout P99 = 4 sec
```

Trace:

```text
Payment = 3.5 sec
```

Log:

```text
Provider timeout
```

Dependency:

```text
External provider latency
```

Action:

```text
Failover / mitigation
```

---

# 127. Kubernetes Incident Workflow

For a failing service:

```bash
kubectl get pods -n production
kubectl describe pod <pod> -n production
kubectl logs <pod> -n production
kubectl logs <pod> --previous -n production
```

Then correlate:

```text
Prometheus
Grafana
Kibana
Jaeger
```

---

# 128. CrashLoopBackOff

Check:

```text
Events
Previous logs
Exit code
OOMKilled
Environment variables
Secrets
ConfigMaps
Probes
```

Observability should accelerate this investigation.

---

# 129. OOMKilled

Check:

```text
Container memory
Memory limits
Application heap
Traffic
Recent release
```

Metrics show:

```text
Memory trend
```

Logs show:

```text
Application behavior
```

Traces show:

```text
Request impact
```

---

# 130. ImagePullBackOff

This is mainly a deployment/infrastructure issue.

Check:

```text
Image name
Tag
ECR access
ImagePullSecrets
IAM
Network
```

Observability may show deployment failure but does not replace Kubernetes investigation.

---

# 131. Readiness Failure

Metrics:

```text
Available replicas ↓
```

Logs:

```text
Readiness failure
```

Trace:

```text
Requests routed elsewhere / fail
```

Investigate:

```text
Probe
Dependency
Application startup
```

---

# 132. Node Failure

Metrics:

```text
Node unavailable
```

Kubernetes:

```text
Pods rescheduled
```

Logs:

```text
Application interruptions
```

Traces:

```text
Error/latency increase
```

Use multiple signals to establish impact.

---

# 133. Network Incident

Symptoms:

```text
Latency ↑
5xx ↑
Timeouts ↑
```

Trace:

```text
Downstream span timeout
```

Logs:

```text
connection timeout
```

Infrastructure metrics:

```text
network errors
```

Investigate:

```text
Security groups
NetworkPolicy
DNS
Load balancer
Service endpoints
```

---

# 134. DNS Incident

Trace:

```text
External call slow
```

Logs:

```text
DNS resolution failure
```

Infrastructure:

```text
DNS errors
```

Investigate DNS independently from application code.

---

# 135. Deployment Incident

Correlate:

```text
Deployment timestamp
```

with:

```text
Metric change
Log error increase
Trace latency change
```

This is one of the fastest ways to identify regressions.

---

# 136. Canary Analysis

Compare:

```text
version A
vs
version B
```

Signals:

```text
Error rate
P99
Trace duration
Log error rate
Resource usage
```

Stop rollout if the new version violates defined thresholds.

---

# 137. Rollback Strategy

If a release causes:

```text
Error rate ↑
Latency ↑
SLO burn ↑
```

follow the approved rollback process.

After rollback verify:

```text
Metrics recover
Logs normalize
Traces normalize
SLO recovers
```

---

# 138. Post-Incident Analysis

Capture:

```text
Detection
Timeline
Impact
Root cause
Contributing factors
Mitigation
Resolution
Follow-up actions
```

Add:

```text
Dashboard improvements
Alert improvements
Instrumentation improvements
Runbook improvements
```

---

# 139. Observability Maturity Levels

## Level 1

```text
Basic infrastructure monitoring
```

## Level 2

```text
Centralized logs
```

## Level 3

```text
Application metrics
```

## Level 4

```text
Distributed tracing
```

## Level 5

```text
Full correlation + SLO-driven operations
```

---

# 140. Mature Observability

A mature platform has:

```text
Metrics
Logs
Traces
Correlation
SLOs
Alerting
Runbooks
Automation
Ownership
Security
Cost controls
DR
```

---

# 141. Automation Opportunities

Automate:

```text
Dashboard provisioning
Alert rule deployment
Collector deployment
Index lifecycle
Retention
Runbook links
Service onboarding
```

Use GitOps.

---

# 142. New Service Onboarding

A new microservice should provide:

```text
Metrics endpoint
Structured logs
OpenTelemetry instrumentation
Service metadata
Dashboard
Alerts
SLO
Runbook
```

This creates an observability contract.

---

# 143. Observability Contract

Every production service should define:

```text
Service name
Owner
SLO
Metrics
Log format
Trace instrumentation
Dependencies
Alerts
Dashboard
Runbook
```

---

# 144. Service Template

Example:

```text
service-name/
 |
 +--> /metrics
 |
 +--> structured JSON logs
 |
 +--> OpenTelemetry instrumentation
 |
 +--> dashboard
 |
 +--> alerts
 |
 +--> SLO
 |
 +--> runbook
```

---

# 145. Observability Review Checklist

Before production:

```text
Can we detect failure?
Can we identify affected users?
Can we find the slow dependency?
Can we search detailed logs?
Can we correlate trace IDs?
Can we measure SLO?
Can we alert the owner?
Can we recover the observability platform?
```

---

# 146. Production Readiness Checklist

## Metrics

- [ ] Prometheus HA strategy
- [ ] Service discovery
- [ ] Application metrics
- [ ] Kubernetes metrics
- [ ] Recording rules
- [ ] Alerts
- [ ] Retention

## Grafana

- [ ] Datasources
- [ ] Dashboards
- [ ] RBAC
- [ ] Alerts
- [ ] Backup/GitOps

## Logging

- [ ] Structured logs
- [ ] Collector
- [ ] Logstash
- [ ] Elasticsearch HA
- [ ] Retention
- [ ] Redaction
- [ ] Kibana access

## Tracing

- [ ] OpenTelemetry
- [ ] Context propagation
- [ ] Collector HA
- [ ] Sampling
- [ ] Jaeger
- [ ] Storage
- [ ] Trace correlation

---

# 147. Production Security Checklist

- [ ] TLS
- [ ] RBAC
- [ ] NetworkPolicy
- [ ] Secrets management
- [ ] PII protection
- [ ] Least privilege
- [ ] Auditability
- [ ] Retention policy
- [ ] Access reviews

---

# 148. Production Reliability Checklist

- [ ] Multi-replica critical components
- [ ] AZ distribution
- [ ] PDB
- [ ] Topology spread
- [ ] Resource limits
- [ ] Queue protection
- [ ] Retry policies
- [ ] Backup
- [ ] Restore testing
- [ ] Capacity planning

---

# 149. Production Cost Checklist

- [ ] Metric cardinality reviewed
- [ ] Log volume reviewed
- [ ] Trace sampling configured
- [ ] Retention reviewed
- [ ] Storage monitored
- [ ] Duplicate telemetry removed
- [ ] Query cost reviewed
- [ ] Resource sizing reviewed

---

# 150. End-to-End Architecture

```text
                              USERS
                                |
                                v
                         AWS ALB / Ingress
                                |
                                v
                     +----------------------+
                     | EKS Microservices    |
                     |                      |
                     | Order                |
                     | User                 |
                     | Payment              |
                     | Inventory            |
                     | Notification         |
                     +----------------------+
                         |      |      |
              +----------+      |      +-----------+
              |                 |                  |
              v                 v                  v
          METRICS             TRACES             LOGS
              |                 |                  |
              v                 v                  v
         Prometheus        OTel SDK          Container stdout
              |                 |                  |
              v                 v                  v
           Grafana         OTel Collector      Log Collector
                                |                  |
                                v                  v
                              Jaeger            Logstash
                                |                  |
                                v                  v
                             Storage          Elasticsearch
                                                   |
                                                   v
                                                 Kibana
```

---

# 151. Complete Incident Workflow

```text
                     INCIDENT
                         |
                         v
                    Grafana Alert
                         |
                         v
                 Identify Service
                         |
                         v
                     Jaeger
                         |
                         v
                 Identify Bad Span
                         |
                         v
                     trace_id
                         |
                         v
                      Kibana
                         |
                         v
                  Detailed Error
                         |
                         v
                 Check Dependency
                         |
                         v
                Metrics / Logs / Traces
                         |
                         v
                      Mitigate
                         |
                         v
                     Validate
                         |
                         v
                    SLO Recovery
                         |
                         v
                   Post-Incident
```

---

# 152. Interview — Explain the Architecture

### 30-second answer

> "I designed a full-stack observability platform for an EKS microservices environment using Prometheus and Grafana for metrics, ELK for centralized logs, and OpenTelemetry with Jaeger for distributed tracing. Prometheus collects infrastructure, Kubernetes and application metrics. Container logs are collected and processed through the logging pipeline into Elasticsearch and Kibana. Applications are instrumented with OpenTelemetry and export traces through collectors to Jaeger. The important part is correlation: metrics detect the problem, traces identify the slow or failing dependency, and logs provide the detailed reason."

---

# 153. Interview — 60-second Answer

> "For a production EKS microservices platform, I treat observability as three connected telemetry pipelines. Prometheus handles infrastructure, Kubernetes and application metrics, with Grafana providing dashboards and alert visibility. For logs, applications write structured JSON logs to stdout, a Kubernetes log collector forwards them through Logstash, and Elasticsearch stores them for Kibana search and dashboards. For tracing, applications use OpenTelemetry instrumentation and export OTLP data to OpenTelemetry Collectors, which handle batching, sampling, enrichment and retries before sending traces to Jaeger. I correlate the systems using service metadata, deployment versions and trace IDs. During incidents, I start with Grafana to detect the issue, use Jaeger to locate the slow or failed dependency, and then use Kibana to explain the failure. The production design also includes HA, security, retention, cost control, GitOps and disaster recovery."

---

# 154. Interview — Why Use Three Pillars?

> "Metrics provide efficient time-series detection, logs provide detailed event context, and traces provide request-level dependency visibility. None of the three completely replaces the others."

---

# 155. Interview — How Do You Correlate Them?

Use common metadata:

```text
service
environment
namespace
version
trace_id
```

Example:

```text
Grafana
  |
  v
payment-service
  |
  v
Jaeger trace_id=abc123
  |
  v
Kibana trace_id=abc123
```

---

# 156. Interview — Which Tool Do You Open First?

> "It depends on the symptom, but for a production alert I usually start with metrics because they provide the fastest high-level signal. Once I identify the affected service or time window, I use traces to locate the dependency or critical path and logs to understand the detailed failure."

---

# 157. Interview — What If Metrics Are Healthy but Users Report Errors?

Check:

```text
Application logs
Traces
Load balancer
Synthetic checks
External dependencies
```

Metrics may be incomplete or aggregate away a small but important failure.

---

# 158. Interview — What If Logs Are Healthy but Traces Show Errors?

Possible:

```text
Log level/filtering
Missing error logs
Different request path
Sampling
Trace instrumentation
```

Do not assume one signal is always complete.

---

# 159. Interview — What If Traces Are Missing but Metrics Work?

Investigate:

```text
OpenTelemetry instrumentation
Exporter
Collector
Sampling
Jaeger
```

Metrics and traces are independent pipelines.

---

# 160. Interview — What If Elasticsearch Is Down?

Impact:

```text
Log search unavailable
```

But:

```text
Prometheus may continue
Jaeger may continue
```

This is why independent telemetry pipelines are valuable.

---

# 161. Interview — What If Prometheus Is Down?

Impact:

```text
Metrics unavailable
Alerts may be affected
Grafana metric panels fail
```

Logs and traces may still provide incident evidence.

---

# 162. Interview — What If Jaeger Is Down?

Impact:

```text
Trace visualization unavailable
```

Metrics and logs can still support investigation.

---

# 163. Interview — Why Should Pipelines Be Independent?

> "Metrics, logs and traces have different volume characteristics, storage requirements and failure modes. Keeping pipelines logically independent prevents one backend failure from taking down the entire observability platform."

---

# 164. Interview — How Do You Avoid Alert Fatigue?

Answer:

```text
Use SLO-based alerts
Use burn rates
Alert on actionable symptoms
Tune thresholds
Deduplicate
Route by severity
Attach runbooks
```

---

# 165. Interview — How Do You Design Dashboards?

Start with:

```text
Golden signals
SLOs
Service health
Dependencies
```

Then drill down:

```text
Infrastructure
Pods
Logs
Traces
```

---

# 166. Interview — How Do You Control Observability Costs?

Answer:

> "I control metric cardinality, reduce unnecessary logs, sample traces intelligently, tune retention, remove duplicate telemetry and continuously monitor storage and query workload."

---

# 167. Interview — What Is High Cardinality?

> "High cardinality occurs when a metric label creates a very large number of unique time series. Labels such as user IDs or request IDs can create massive cardinality and should generally not be used as Prometheus labels."

---

# 168. Interview — How Do You Handle Production PII?

Answer:

```text
Prevent collection
Redact sensitive fields
Restrict access
Encrypt transport/storage
Define retention
Audit access
```

---

# 169. Interview — How Do You Make Observability Highly Available?

Answer:

```text
Multiple replicas
AZ distribution
PDB
Topology spread
Storage replication
Bounded queues
Retry
Monitoring of monitoring
```

---

# 170. Interview — How Do You Test Observability?

Test:

```text
Normal traffic
Error
Latency
Collector failure
Backend failure
Node failure
Storage pressure
Deployment regression
```

Verify that:

```text
Metrics
Logs
Traces
Alerts
```

behave as expected.

---

# 171. Interview — What Is Your Incident Process?

Answer:

```text
Detect
 |
Identify affected service
 |
Measure impact
 |
Trace request
 |
Inspect logs
 |
Check dependencies
 |
Mitigate
 |
Verify recovery
 |
Document
```

---

# 172. Interview — How Do You Investigate a 500 Error?

```text
1. Check error rate.
2. Identify service.
3. Find error traces.
4. Inspect failed span.
5. Search trace_id in logs.
6. Check dependency.
7. Check recent deployment.
8. Mitigate.
```

---

# 173. Interview — How Do You Investigate High CPU?

```text
1. Node/container CPU metric.
2. Identify workload.
3. Check request rate.
4. Check recent deployment.
5. Check application traces.
6. Search application logs.
7. Determine whether CPU is expected or abnormal.
```

---

# 174. Interview — How Do You Investigate Disk Pressure?

```text
1. Check filesystem metrics.
2. Identify node.
3. Check Kubernetes DiskPressure.
4. Check container logs.
5. Check images/temp files.
6. Check observability storage.
7. Clean/expand according to runbook.
```

---

# 175. Interview — How Do You Investigate Missing Logs?

Pipeline:

```text
Application
 |
stdout
 |
Node log
 |
Collector
 |
Logstash
 |
Elasticsearch
 |
Kibana
```

Find the first broken layer.

---

# 176. Interview — How Do You Investigate Missing Traces?

Pipeline:

```text
Application
 |
Instrumentation
 |
OTLP
 |
Collector
 |
Jaeger
 |
Storage
 |
UI
```

Again, find the first broken layer.

---

# 177. Interview — How Do You Investigate Missing Metrics?

Pipeline:

```text
Target
 |
Service discovery
 |
Prometheus scrape
 |
Prometheus storage
 |
Grafana
```

Check:

```text
Target
Endpoint
Labels
Scrape errors
Prometheus
Datasource
```

---

# 178. Interview — How Do You Correlate a Deployment?

Use:

```text
Deployment timestamp
service.version
Git commit
dashboard annotations
trace attributes
log fields
```

Compare:

```text
Before
vs
After
```

---

# 179. Interview — What Is the Most Important Design Principle?

> "Observability should reduce mean time to detect and mean time to resolve. A tool is valuable only if it helps engineers move from symptom to root cause quickly."

---

# 180. Project Implementation Sequence

Build in this order:

```text
1. EKS
2. Prometheus
3. Grafana
4. Structured application logs
5. Log collector
6. Logstash
7. Elasticsearch
8. Kibana
9. OpenTelemetry instrumentation
10. OTel Collector
11. Jaeger
12. Correlation
13. Dashboards
14. Alerts
15. SLOs
16. Security
17. HA
18. DR
19. Load testing
20. Incident testing
```

---

# 181. Project Validation Sequence

## Metrics

```text
Prometheus target UP
```

## Logs

```text
Log visible in Kibana
```

## Traces

```text
Trace visible in Jaeger
```

## Correlation

```text
Same trace_id in logs and traces
```

## Alerting

```text
Test alert fires
```

## Incident

```text
Inject failure
Detect
Locate
Explain
Recover
```

---

# 182. Production Architecture Review

Ask:

```text
Can we detect a failure?
Can we identify the affected service?
Can we see the request path?
Can we find detailed logs?
Can we correlate telemetry?
Can we alert the owner?
Can we survive a component failure?
Can we restore the platform?
Can we control cost?
```

---

# 183. Final Mental Model

```text
                         USER
                           |
                           v
                         ALB
                           |
                           v
                     APPLICATION
                      /    |    \
                     /     |     \
                    v      v      v
                METRICS  LOGS   TRACES
                   |       |       |
                   v       v       v
              Prometheus Logstash OTel
                   |       |       |
                   v       v       v
                Grafana Elasticsearch Jaeger
                           |       |
                           v       v
                         Kibana  Jaeger UI
                              \    /
                               \  /
                                \/
                          INCIDENT TEAM
```

---

# 184. Final Production Workflow

```text
                    PRODUCTION
                        |
                        v
                Telemetry Generated
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
       Metrics        Logs         Traces
          |             |             |
          v             v             v
     Prometheus     Elasticsearch    Jaeger
          |             |             |
          v             v             v
       Grafana        Kibana        Trace UI
          \             |             /
           \            |            /
            +-----------+-----------+
                        |
                        v
                   Correlation
                        |
                        v
                   Root Cause
                        |
                        v
                    Mitigation
                        |
                        v
                    SLO Recovery
```

---

# 185. Final Project Summary

This project combines the three observability pillars into one production platform:

```text
Prometheus
    |
    +--> Metrics
    +--> PromQL
    +--> Alerting

Grafana
    |
    +--> Dashboards
    +--> Visualization
    +--> SLO views

ELK
    |
    +--> Centralized logs
    +--> Search
    +--> Analysis

OpenTelemetry
    |
    +--> Instrumentation
    +--> Context propagation
    +--> Collection

Jaeger
    |
    +--> Distributed tracing
    +--> Trace search
    +--> Dependency analysis
```

The strongest operational workflow is:

```text
Metrics detect
      |
      v
Traces locate
      |
      v
Logs explain
      |
      v
SLO validates impact
      |
      v
Incident response mitigates
      |
      v
Post-incident improvements
```

---

# 186. Key Interview Takeaway

A strong production answer is:

> "I designed observability as an integrated platform rather than deploying isolated tools. Prometheus and Grafana provide infrastructure, Kubernetes and application metrics; ELK provides centralized structured logs; OpenTelemetry provides vendor-neutral instrumentation and collection; and Jaeger provides distributed tracing. I standardize service, environment and version metadata and propagate trace IDs into logs. During an incident, metrics identify the affected service, traces expose the critical dependency or slow span, and logs provide the detailed error context. The production design also covers HA, security, retention, sampling, cost optimization, GitOps, SLOs, disaster recovery and monitoring the observability platform itself."

---

# 187. Next Project

The next file is:

```text
05-Microservices-Observability.md
```

It will focus specifically on observability for a real microservices platform, including:

```text
Service-level instrumentation
Service dependency mapping
API gateway / ALB observability
Java / Node.js / Python services
Synchronous communication
Asynchronous communication
RabbitMQ-style messaging
Database observability
Redis/cache observability
Business metrics
Distributed tracing
Trace/log correlation
Kubernetes workload observability
SLOs per microservice
Failure isolation
Production incident scenarios
Capacity planning
Security
Cost optimization
GitOps
Interview architecture and scenario questions
```
