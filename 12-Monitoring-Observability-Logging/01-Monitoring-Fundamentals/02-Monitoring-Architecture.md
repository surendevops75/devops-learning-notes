# Monitoring Architecture

Monitoring architecture defines how monitoring data is generated, collected, processed, stored, visualized, and used for alerting.

A production monitoring architecture should not simply answer:

> "Is the server running?"

It should provide visibility across:

```
Infrastructure
Kubernetes
Containers
Applications
Services
Databases
Networks
Load Balancers
Dependencies
User-facing services
```

The overall objective is:

```
Collect
   ↓
Process
   ↓
Store
   ↓
Query
   ↓
Visualize
   ↓
Alert
   ↓
Investigate
   ↓
Resolve
```

---

# 1. What Is Monitoring Architecture?

Monitoring architecture is the design of the complete monitoring pipeline.

It defines:

```
Where telemetry is generated
How telemetry is collected
Where telemetry is processed
Where telemetry is stored
How engineers query it
How dashboards are created
How alerts are generated
How incidents are investigated
```

A simple architecture is:

```
Application
    |
    ↓
Telemetry
    |
    ↓
Collector
    |
    ↓
Monitoring Backend
    |
    ↓
Visualization
    |
    ↓
Alerting
    |
    ↓
Engineer
```

---

# 2. Monitoring Architecture Layers

A production architecture can be divided into layers:

```
Layer 1 → Sources
Layer 2 → Collection
Layer 3 → Processing
Layer 4 → Storage
Layer 5 → Query
Layer 6 → Visualization
Layer 7 → Alerting
Layer 8 → Incident Response
```

Architecture:

```
Sources
   ↓
Collection
   ↓
Processing
   ↓
Storage
   ↓
Query
   ↓
Visualization
   ↓
Alerting
   ↓
Incident Response
```

---

# 3. Monitoring Sources

Monitoring data can originate from many sources.

Examples:

```
Linux Servers
EC2
Kubernetes Nodes
Kubernetes Pods
Containers
Applications
Databases
Load Balancers
Network Devices
Microservices
External Dependencies
```

Example:

```
┌───────────────┐
│ Linux / EC2   │
└───────┬───────┘
        |
┌───────┴────────┐
│                │
↓                ↓
```

Metrics           Logs

---

# 4. Application as a Monitoring Source

Applications can expose metrics and generate logs.

Example:

```
Java Application
     |
     +---- /metrics
     |
     +---- Application Logs
     |
     +---- Traces
```

A microservice can therefore produce multiple telemetry signals.

---

# 5. Kubernetes as a Monitoring Source

Kubernetes produces information about:

```
Nodes
Pods
Containers
Deployments
Services
ReplicaSets
Jobs
HPA
Resource Usage
Events
```

Architecture:

```
EKS
 |
 +--- Nodes
 |
 +--- Pods
 |
 +--- Containers
 |
 +--- Deployments
 |
 +--- Services
```

---

# 6. Collection Layer

The collection layer gathers monitoring data from sources.

Examples:

```
Prometheus
Exporters
OpenTelemetry Collector
Log Collectors
Application Agents
```

For metrics:

```
Application
    |
    ↓
/metrics
    |
    ↓
Prometheus
```

For traces:

```
Application
    |
    ↓
OpenTelemetry
    |
    ↓
OTel Collector
```

For logs:

```
Application
    |
    ↓
Log Collector
    |
    ↓
Logstash
    |
    ↓
Elasticsearch
```

---

# 7. Pull-Based Metrics Collection

Prometheus commonly uses a pull-based model.

Architecture:

```
Application
    |
    ↓
/metrics
    |
    ↑
  Scrape
    |
    |
Prometheus
```

Prometheus periodically scrapes metrics endpoints.

Example:

```
Prometheus
    |
    | HTTP GET
    ↓
http://service:8080/metrics
```

The application responds with metrics.

---

# 8. Push-Based Telemetry

Some systems use a push model.

Architecture:

```
Application
    |
    ↓
Telemetry Agent
    |
    ↓
Collector
    |
    ↓
Backend
```

OpenTelemetry can support telemetry pipelines where applications or
agents send telemetry to an OpenTelemetry Collector.

---

# 9. Push vs Pull

## Pull

```
Prometheus
    |
    ↓
Target
```

Prometheus controls collection.

## Push

```
Application
    |
    ↓
Collector
```

The source sends telemetry.

Both models have valid use cases.

---

# 10. Processing Layer

Telemetry may require processing before storage.

Examples:

```
Parsing
Filtering
Enrichment
Transformation
Aggregation
Sampling
Redaction
```

Example:

```
Raw Log
   |
   ↓
Parse
   |
   ↓
Extract Fields
   |
   ↓
Add Metadata
   |
   ↓
Store
```

---

# 11. Log Processing

Suppose an application produces:

```
2026-08-10 ERROR payment timeout
```

A log processing pipeline may convert it into structured data:

```
timestamp = 2026-08-10
level     = ERROR
service   = payment
message   = timeout
```

This makes searching and filtering easier.

---

# 12. Metrics Processing

Metrics can also be transformed or aggregated.

Example:

```
Individual Requests
      |
      ↓
Request Duration
      |
      ↓
Aggregation
      |
      ↓
P95 Latency
```

This allows engineers to understand service behavior without
examining every individual request.

---

# 13. Trace Processing

Tracing pipelines may perform:

```
Filtering
Sampling
Attribute Processing
Batching
Exporting
```

Example:

```
Application
    |
    ↓
OTel Collector
    |
    ↓
Batch
    |
    ↓
Sample
    |
    ↓
Export
    |
    ↓
Jaeger
```

---

# 14. Storage Layer

Monitoring data needs a backend.

Examples:

```
Prometheus
    → Metrics

Elasticsearch
    → Logs

Jaeger-compatible backend
    → Traces
```

Storage must be designed for:

```
Capacity
Performance
Retention
Availability
Backup
Recovery
```

---

# 15. Metrics Storage

Prometheus stores time-series metrics.

Conceptually:

```
Metric
   |
   ↓
Timestamp
   |
   ↓
Value
```

Example:

```
cpu_usage
   |
   +--- 09:00 → 50
   +--- 09:01 → 55
   +--- 09:02 → 60
   +--- 09:03 → 65
```

---

# 16. Log Storage

Elasticsearch stores logs as searchable documents.

Example:

```
{
  timestamp: "2026-08-10T09:10:00Z",
  service: "order-service",
  level: "ERROR",
  message: "Database timeout"
}
```

This allows queries such as:

```
service = order-service
level = ERROR
```

---

# 17. Trace Storage

Trace storage contains information about:

```
Trace
Span
Duration
Service
Operation
Attributes
Relationships
```

Example:

```
Trace
  |
  +--- Span: API
  |
  +--- Span: Order Service
  |
  +--- Span: Payment Service
  |
  +--- Span: Database
```

---

# 18. Query Layer

Engineers need query mechanisms to investigate telemetry.

Examples:

```
PromQL
Elasticsearch Query
Kibana Search
Jaeger Trace Search
```

Query layer:

```
Stored Data
    |
    ↓
Query
    |
    ↓
Result
```

---

# 19. PromQL

PromQL is Prometheus Query Language.

Example:

```
rate(http_requests_total[5m])
```

This can be used to calculate request rate over a five-minute
window.

Another example:

```
rate(http_requests_total{status="500"}[5m])
```

This filters for HTTP 500 responses.

---

# 20. Log Queries

Kibana can be used to search centralized logs.

Example:

```
service: "payment-service"
AND level: "ERROR"
```

This can identify payment service errors.

---

# 21. Trace Queries

Jaeger can be used to search traces by:

```
Service
Operation
Duration
Tags / Attributes
Trace ID
```

Example:

```
Service = order-service
Duration > 1s
```

This can help find slow traces.

---

# 22. Visualization Layer

Visualization converts telemetry into dashboards.

The primary visualization tool in our stack is:

```
Grafana
```

Architecture:

```
Prometheus
    |
    ↓
Grafana
    |
    ↓
Dashboard
```

Grafana can display:

```
CPU
Memory
Request Rate
Error Rate
Latency
Kubernetes Health
```

---

# 23. Logging Visualization

Kibana provides visualization and investigation of log data.

Architecture:

```
Elasticsearch
     |
     ↓
   Kibana
     |
     ↓
Search / Dashboard
```

---

# 24. Tracing Visualization

Jaeger provides a UI for examining distributed traces.

Architecture:

```
Trace Backend
     |
     ↓
   Jaeger UI
     |
     ↓
Trace Timeline
```

---

# 25. Alerting Layer

Alerting converts monitoring conditions into notifications.

Example:

```
Metric
   |
   ↓
Rule
   |
   ↓
Condition Met
   |
   ↓
Alert
   |
   ↓
Notification
```

Example:

```
CPU > 90%
for 10 minutes

↓

Alert
```

---

# 26. Alerting Architecture

A common Prometheus architecture is:

```
Prometheus
    |
    ↓
Alert Rule
    |
    ↓
Alertmanager
    |
    +--- Email
    |
    +--- Slack
    |
    +--- Pager
    |
    +--- Other Notification
```

---

# 27. Alertmanager

Alertmanager is responsible for handling alerts generated by
Prometheus.

It can perform:

```
Grouping
Deduplication
Routing
Silencing
Inhibition
```

Example:

```
100 Pod Alerts
     |
     ↓
Alertmanager
     |
     ↓
Group Related Alerts
     |
     ↓
One Incident Notification
```

---

# 28. Incident Response Layer

The final layer is incident response.

Architecture:

```
Alert
  |
  ↓
Engineer
  |
  ↓
Metrics
  |
  ↓
Logs
  |
  ↓
Traces
  |
  ↓
Root Cause
  |
  ↓
Resolution
```

---

# 29. Complete Monitoring Architecture

The complete architecture is:

```
┌──────────────────────────────────────────────┐
│                   SOURCES                    │
│                                              │
│ EC2 | EKS | Pods | Containers | Applications │
│ Databases | ALB | Network | Dependencies    │
└──────────────────────┬───────────────────────┘
                       |
         +-------------+-------------+
         |             |             |
         ↓             ↓             ↓
      Metrics         Logs         Traces
         |             |             |
         ↓             ↓             ↓
    Prometheus      Log Pipeline   OpenTelemetry
         |             |             |
         |             ↓             ↓
         |        Elasticsearch   Collector
         |             |             |
         |             ↓             ↓
         |           Kibana        Jaeger
         |             |
         +-------------+-------------+
                       |
                       ↓
                   Visualization
                       |
          +------------+------------+
          |            |            |
          ↓            ↓            ↓
       Grafana      Kibana       Jaeger UI
          |
          ↓
       Alerting
          |
          ↓
     Alertmanager
          |
          ↓
   Notification
          |
          ↓
      Engineer
```

---

# 30. Real-World AWS EKS Architecture

A production EKS environment can look like:

```
AWS
 |
 +------------------------------------------------+
 |                                                |
 |                    EKS                         |
 |                                                |
 |   +------------+  +------------+               |
 |   | Node Group |  | Node Group |               |
 |   +-----+------+  +------+-----+               |
 |         |                |                     |
 |         +-------+--------+                     |
 |                 |                              |
 |                 ↓                              |
 |          Microservices                         |
 |                                                  |
 +------------------------------------------------+
                   |
         +---------+---------+
         |         |         |
         ↓         ↓         ↓
      Metrics    Logs      Traces
         |         |         |
         ↓         ↓         ↓
    Prometheus    ELK    OpenTelemetry
         |                   |
         ↓                   ↓
      Grafana              Jaeger
```

---

# 31. EKS Monitoring Components

For Kubernetes monitoring, the architecture can include:

```
Prometheus
    |
    +--- Kubernetes API
    |
    +--- Nodes
    |
    +--- Pods
    |
    +--- Services
    |
    +--- Applications
```

Prometheus can discover Kubernetes targets and scrape their metrics.

---

# 32. Kubernetes Service Discovery

Kubernetes environments are dynamic.

Pods can:

```
Start
Stop
Restart
Scale
Move between nodes
```

Therefore, static monitoring targets are difficult to maintain.

Prometheus can use Kubernetes service discovery.

Architecture:

```
Kubernetes API
      |
      ↓
Service Discovery
      |
      ↓
Prometheus
      |
      ↓
Dynamic Targets
```

---

# 33. Dynamic Monitoring

Suppose:

```
order-service
   |
   +--- Pod A
   +--- Pod B
   +--- Pod C
```

HPA scales it:

```
order-service
   |
   +--- Pod A
   +--- Pod B
   +--- Pod C
   +--- Pod D
   +--- Pod E
```

Monitoring should automatically discover new targets.

This is one reason service discovery is important.

---

# 34. Kubernetes Logging Architecture

Containers generate logs.

A common architecture is:

```
Application
    |
    ↓
Container stdout/stderr
    |
    ↓
Node Log Files
    |
    ↓
Log Collector
    |
    ↓
Logstash / Backend
    |
    ↓
Elasticsearch
    |
    ↓
Kibana
```

The exact collection component can vary depending on the production
design.

---

# 35. Kubernetes Tracing Architecture

A cloud-native tracing architecture can be:

```
Microservice A
     |
     ↓
OpenTelemetry
     |
     ↓
OTel Collector
     |
     ↓
   Jaeger

Microservice B
     |
     ↓
OpenTelemetry
     |
     ↓
OTel Collector
     |
     ↓
   Jaeger
```

All services can send telemetry through a common collector layer.

---

# 36. Why Use a Collector?

Without a collector:

```
Service A ─────────→ Backend A
Service B ─────────→ Backend A
Service C ─────────→ Backend A
```

With a collector:

```
Service A ──┐
Service B ──┼──→ OTel Collector ──→ Backend
Service C ──┘
```

The collector provides a central processing layer.

---

# 37. Collector Responsibilities

An OpenTelemetry Collector can perform:

```
Receive
Process
Filter
Transform
Batch
Sample
Export
```

Example:

```
Applications
     |
     ↓
OTel Collector
     |
     +--- Batch
     |
     +--- Filter
     |
     +--- Sampling
     |
     ↓
   Backend
```

---

# 38. Agent vs Collector

An agent is generally deployed close to the workload.

A collector can provide a centralized telemetry processing layer.

Example:

```
Pod
  |
  ↓
Agent
  |
  ↓
Gateway Collector
  |
  ↓
Backend
```

This can provide a scalable telemetry architecture.

---

# 39. Monitoring Architecture for Microservices

Consider:

```
User
  |
  ↓
ALB
  |
  ↓
User Service
  |
  ↓
Product Service
  |
  ↓
Order Service
  |
  +------→ Payment Service
  |
  +------→ Inventory Service
```

Monitoring:

```
All Services
    |
    +------ Metrics ------→ Prometheus
    |
    +------ Logs --------→ ELK
    |
    +------ Traces ------→ OTel → Jaeger
```

Visualization:

```
Prometheus
    |
    ↓
Grafana

Elasticsearch
    |
    ↓
Kibana

Jaeger
    |
    ↓
Jaeger UI
```

---

# 40. Correlation Between Metrics, Logs and Traces

The real power of observability comes from correlation.

Example:

```
Grafana
   |
   ↓
Error Rate Increased
   |
   ↓
Identify Service
   |
   ↓
Kibana
   |
   ↓
Application Error
   |
   ↓
Trace ID
   |
   ↓
Jaeger
   |
   ↓
Slow Dependency
```

This produces a much faster investigation process.

---

# 41. Example Incident

Problem:

```
Users report checkout failures.
```

Metrics show:

```
checkout_error_rate = 8%
```

Engineer opens Grafana:

```
Error rate increased after deployment.
```

Next:

```
Kibana
```

Search:

```
service = checkout-service
level = ERROR
```

Logs show:

```
payment dependency timeout
```

Next:

```
Jaeger
```

Trace:

```
checkout
   |
   ↓
payment-service
   |
   ↓
external-payment-api
   |
   ↓
5.2 seconds
```

Root cause:

```
External payment dependency latency.
```

This is an example of using all three signals together.

---

# 42. Monitoring During Deployment

A production deployment should be monitored.

Before deployment:

```
Capture Baseline
    |
    ↓
Error Rate
Latency
Traffic
CPU
Memory
```

Deploy:

```
Version B
    |
    ↓
Monitor
```

After deployment:

```
Compare
    |
    ↓
Baseline vs New Version
```

If metrics degrade:

```
Investigate
    |
    ↓
Rollback if necessary
```

---

# 43. Canary Monitoring

For canary deployments:

```
95% → Version A
 5% → Version B
```

Monitor Version B carefully.

Compare:

```
Error Rate
Latency
CPU
Memory
Request Success
```

If healthy:

```
5%
 ↓
25%
 ↓
50%
 ↓
100%
```

If unhealthy:

```
Rollback
```

Monitoring is therefore an important part of safe deployment strategies.

---

# 44. Blue-Green Monitoring

Architecture:

```
Users
  |
  ↓
Load Balancer
  |
  +------→ Blue
  |
  +------→ Green
```

Monitor both environments.

Before switching:

```
Green Health
    |
    ↓
Validate
    |
    ↓
Switch Traffic
```

After switching:

```
Monitor Green
```

---

# 45. Monitoring and SLOs

Monitoring architecture should support service objectives.

Example:

```
SLO:
Availability = 99.9%
```

Monitoring collects:

```
Successful Requests
Failed Requests
```

Then calculate availability.

Example:

```
Availability =
Successful Requests / Total Requests
```

Monitoring provides the data required to evaluate SLOs.

---

# 46. Monitoring and Error Budgets

If the SLO is:

```
99.9%
```

The allowed error budget is:

```
0.1%
```

Monitoring tracks whether the system is consuming that budget.

Example:

```
Error Budget
    |
    ↓
100%
    |
    ↓
 70%
    |
    ↓
 30%
    |
    ↓
  0%
```

If the budget is exhausted, teams may need to prioritize reliability
work.

---

# 47. Monitoring Storage Design

Storage planning should consider:

```
Data Volume
Retention
Query Load
Ingestion Rate
Replication
Backup
Recovery
```

Example:

```
High Ingestion
     |
     ↓
High Storage
     |
     ↓
Capacity Planning
```

---

# 48. Monitoring Retention

Not every signal needs identical retention.

Example:

```
High-resolution metrics
    → shorter retention

Aggregated metrics
    → longer retention

Application logs
    → policy-based retention

Traces
    → retention based on volume and troubleshooting needs
```

Retention should balance:

```
Cost
Performance
Operational Need
Compliance
```

---

# 49. Monitoring Scalability

As the environment grows:

```
10 Services
   ↓
50 Services
   ↓
100 Services
   ↓
500 Services
```

Telemetry volume grows as well.

Architecture must scale:

```
Collection
Processing
Storage
Query
Visualization
```

---

# 50. Monitoring High Availability

Monitoring is itself a production service.

If the monitoring platform is unavailable:

```
Production Problem
     |
     ↓
Monitoring Down
     |
     ↓
No Visibility
```

Therefore, critical monitoring components should have suitable HA
designs.

---

# 51. Monitoring Security Architecture

A secure architecture may look like:

```
Application Network
       |
       ↓
Private Monitoring Network
       |
       +--- Prometheus
       +--- Elasticsearch
       +--- Jaeger
       |
       ↓
Controlled Access
       |
       ↓
    Grafana / Kibana
```

Use:

```
Authentication
Authorization
TLS
Network Policies
Security Groups
Private Endpoints
Secrets Management
```

where appropriate.

---

# 52. Monitoring Backup Strategy

Backup requirements depend on the component.

Important data may include:

```
Dashboards
Alert Rules
Configuration
Elasticsearch Data
Important Metadata
```

Configuration should ideally be version controlled where practical.

Example:

```
Git
  |
  +--- Prometheus Config
  +--- Alert Rules
  +--- Grafana Provisioning
  +--- OTel Config
  +--- Kubernetes Manifests
```

---

# 53. Infrastructure as Code

Monitoring infrastructure can be managed using:

```
Terraform
Helm
Kubernetes Manifests
GitOps
```

Example:

```
Git
  |
  ↓
Terraform / Helm
  |
  ↓
EKS
  |
  ↓
Monitoring Stack
```

This provides repeatability.

---

# 54. GitOps Monitoring Architecture

A production GitOps model:

```
Developer
   |
   ↓
Git Repository
   |
   ↓
ArgoCD
   |
   ↓
EKS
   |
   ↓
Monitoring Stack
```

Configuration changes are reviewed through Git.

---

# 55. Monitoring Configuration Management

Configuration should be:

```
Version Controlled
Reviewed
Tested
Auditable
Reproducible
```

Avoid manually changing production monitoring configuration whenever
possible.

---

# 56. Monitoring Architecture Failure Scenarios

## Scenario 1

Prometheus is down.

Impact:

```
Metrics unavailable
Dashboards incomplete
Alerts may fail
```

Response:

```
Check Prometheus
Check storage
Check resource usage
Check configuration
Restore service
```

---

## Scenario 2

Elasticsearch disk is full.

Impact:

```
Logs may stop indexing.
```

Response:

```
Check storage
Check retention
Remove/roll old data according to policy
Increase capacity if required
```

---

## Scenario 3

OpenTelemetry Collector is overloaded.

Impact:

```
Telemetry may be delayed or dropped depending on configuration.
```

Response:

```
Check CPU
Check memory
Check queue
Check export failures
Scale collectors
```

---

## Scenario 4

Jaeger is unavailable.

Impact:

```
Distributed tracing unavailable.
```

Metrics and logs may still work if independently designed.

This demonstrates why observability should avoid unnecessary
single points of failure.

---

# 57. Monitoring Dependencies

Monitoring systems have dependencies.

Example:

```
Grafana
   |
   ↓
Prometheus
   |
   ↓
Kubernetes / Applications
```

If Prometheus fails:

```
Grafana
   |
   X
No Metrics
```

Therefore, monitoring dependencies should also be monitored.

---

# 58. Monitoring the Monitoring Stack

Example:

```
Prometheus
   |
   ↓
Prometheus Health

Grafana
   |
   ↓
Grafana Health

Elasticsearch
   |
   ↓
Cluster Health

OTel Collector
   |
   ↓
Collector Metrics

Jaeger
   |
   ↓
Jaeger Health
```

This creates a self-observing monitoring platform.

---

# 59. Real-World Enterprise Architecture

A larger environment may look like:

```
┌─────────────────────────────────────────────────────┐
│                    Production                       │
│                                                     │
│   EKS Cluster                                      │
│                                                     │
│   Services                                          │
│      │                                              │
│      ├──── Metrics ────→ Prometheus                │
│      │                        │                    │
│      │                        ↓                    │
│      │                     Grafana                  │
│      │                                             │
│      ├──── Logs ────────→ Log Pipeline             │
│      │                        │                    │
│      │                        ↓                    │
│      │                   Elasticsearch              │
│      │                        │                    │
│      │                        ↓                    │
│      │                      Kibana                  │
│      │                                             │
│      └──── Traces ─────→ OpenTelemetry              │
│                               │                    │
│                               ↓                    │
│                           OTel Collector             │
│                               │                    │
│                               ↓                    │
│                             Jaeger                  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

# 60. Recommended Operational Model

A mature team should operate monitoring as a platform.

Responsibilities include:

```
Monitoring Design
Instrumentation
Collection
Storage
Dashboards
Alerting
Access Control
Retention
Capacity Planning
Troubleshooting
Incident Response
```

---

# 61. Monitoring Design Questions

Before deploying a monitoring system, ask:

```
What do we need to monitor?

What signals do we need?

Where will data come from?

How will data be collected?

Where will it be stored?

How long will it be retained?

Who needs access?

What alerts are required?

What happens when the monitoring system fails?

How will the system scale?

How will it be secured?

How will it be backed up?
```

---

# 62. Production Architecture Checklist

```
[ ] Sources identified
[ ] Metrics collection designed
[ ] Logging architecture designed
[ ] Tracing architecture designed
[ ] Collection layer designed
[ ] Processing layer designed
[ ] Storage selected
[ ] Retention defined
[ ] Dashboards created
[ ] Alerts designed
[ ] Alert routing configured
[ ] Access control implemented
[ ] TLS considered
[ ] Persistent storage configured
[ ] High availability considered
[ ] Capacity planning completed
[ ] Backup strategy defined
[ ] Disaster recovery considered
[ ] Monitoring stack monitored
[ ] Troubleshooting procedures documented
```

---

# 63. Interview Questions

## What is monitoring architecture?

### Answer

Monitoring architecture is the design of how monitoring data is
generated, collected, processed, stored, queried, visualized, and
used for alerting and incident response.

A typical flow is:

```
Sources
   ↓
Collection
   ↓
Processing
   ↓
Storage
   ↓
Visualization
   ↓
Alerting
   ↓
Incident Response
```

---

# 64. How would you design monitoring for an EKS cluster?

### Answer

I would design monitoring at multiple layers.

Infrastructure:

```
Nodes
CPU
Memory
Disk
Network
```

Kubernetes:

```
Pods
Deployments
Services
Node Health
Resource Utilization
```

Application:

```
Request Rate
Error Rate
Latency
Availability
```

I would use Prometheus for metrics, Grafana for visualization, ELK
for centralized logging, and OpenTelemetry with Jaeger for
distributed tracing.

---

# 65. How would you design observability for microservices?

### Answer

I would collect three major signals:

```
Metrics
Logs
Traces
```

Metrics:

```
Prometheus
```

Visualization:

```
Grafana
```

Logs:

```
Log Pipeline
    ↓
Elasticsearch
    ↓
Kibana
```

Traces:

```
OpenTelemetry
    ↓
OTel Collector
    ↓
Jaeger
```

I would also correlate these signals using service names, timestamps,
and trace/request identifiers where appropriate.

---

# 66. Why is service discovery important in Kubernetes monitoring?

### Answer

Kubernetes workloads are dynamic.

Pods can be created, deleted, restarted, or scaled automatically.

Static monitoring configuration would require constant manual changes.

Service discovery allows the monitoring system to dynamically discover
new targets and remove targets that no longer exist.

---

# 67. What happens if Prometheus goes down?

Potential impacts include:

```
Metrics stop being collected
Dashboards lose current metrics
Prometheus-based alerts may stop evaluating
Troubleshooting visibility is reduced
```

In production, I would design the monitoring platform with suitable
availability, persistence, alerting, and recovery mechanisms.

---

# 68. How do you prevent monitoring from becoming a single point of failure?

I would consider:

```
High Availability
Persistent Storage
Multiple Collector Instances
Appropriate Replication
Backup
Disaster Recovery
Monitoring the Monitoring Stack
Capacity Planning
```

The exact design depends on scale and business requirements.

---

# 69. Final Architecture

The complete architecture we will build throughout this chapter is:

```
USERS
  |
  ↓
ALB
  |
  ↓
EKS
  |
  +-------------------------------+
  |                               |
  ↓                               ↓
SERVICES                       KUBERNETES
  |                               |
  +---------------+---------------+
                  |
      +-----------+-----------+
      |           |           |
      ↓           ↓           ↓
   Metrics       Logs       Traces
      |           |           |
      ↓           ↓           ↓
 Prometheus      ELK     OpenTelemetry
      |           |           |
      ↓           ↓           ↓
  Grafana    Elasticsearch  Collector
                 |             |
                 ↓             ↓
               Kibana        Jaeger
                               |
                               ↓
                          Jaeger UI

              +----------------+
              |
              ↓
           Alerting
              |
              ↓
         Alertmanager
              |
              ↓
          Engineers
```

The goal of this architecture is not simply to collect data.

The goal is to provide:

```
Visibility
   +
Detection
   +
Investigation
   +
Root Cause Analysis
   +
Faster Recovery
   +
Reliability
```

This architecture will be used as the foundation for the remaining
Monitoring, Observability & Logging sections.
