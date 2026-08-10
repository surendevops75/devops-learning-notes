# Monitoring Fundamentals

Monitoring is the process of continuously collecting, storing, analyzing,
and visualizing information about systems, infrastructure, applications,
and services.

In DevOps and cloud environments, monitoring helps teams understand:

```
Is the system healthy?
Is the application available?
Are users experiencing errors?
Is performance degrading?
Are resources being exhausted?
Did a recent deployment cause a problem?
Is the infrastructure operating normally?
```

Monitoring is one of the fundamental pillars of operating production
systems reliably.

---

# 1. What Is Monitoring?

Monitoring is the continuous observation of system behavior using
measurable signals.

A basic monitoring flow is:

```
System
   |
   ↓
Metrics / Events
   |
   ↓
Monitoring System
   |
   ↓
Storage
   |
   ↓
Visualization
   |
   ↓
Alerts
   |
   ↓
Engineer
```

For example:

```
Application
   |
   ↓
CPU = 85%
Memory = 90%
Error Rate = 8%
Latency = 2.5 sec
   |
   ↓
Monitoring System
   |
   ↓
Alert
   |
   ↓
DevOps Engineer
   |
   ↓
Investigation
```

---

# 2. Why Monitoring Is Important

Without monitoring, production problems may be discovered only when
users report them.

Example:

```
Application Failure
      |
      X
    No Monitoring
      |
      ↓
  Users Report
      |
      ↓
  Engineer Investigates
```

With monitoring:

```
Application Failure
      |
      ↓
   Metrics
      |
      ↓
    Alert
      |
      ↓
  Engineer
      |
      ↓
  Investigation
```

Monitoring changes the operating model from:

```
"Wait for users to report problems"
```

to:

```
"Detect problems as early as possible."
```

---

# 3. Monitoring in DevOps

Monitoring is part of the complete DevOps lifecycle.

```
PLAN
  |
  ↓
CODE
  |
  ↓
BUILD
  |
  ↓
TEST
  |
  ↓
RELEASE
  |
  ↓
DEPLOY
  |
  ↓
OPERATE
  |
  ↓
MONITOR
  |
  ↓
FEEDBACK
  |
  └──────────────→ PLAN
```

Monitoring provides feedback to development, operations, and platform
teams.

---

# 4. Monitoring vs Observability

Monitoring and observability are related but not identical.

## Monitoring

Monitoring answers:

```
"Is something wrong?"
```

Examples:

```
CPU is high
Memory is high
Error rate increased
Pod is down
Disk is full
```

## Observability

Observability helps answer:

```
"Why is it wrong?"
```

For example:

```
Error Rate Increased
        |
        ↓
    Which Service?
        |
        ↓
    Which Endpoint?
        |
        ↓
    Which Request?
        |
        ↓
    Which Dependency?
        |
        ↓
    Root Cause
```

Monitoring tells us that there is a problem.

Observability helps us investigate the internal state and determine
why the problem occurred.

---

# 5. Monitoring and Observability Relationship

A useful model is:

```
Monitoring
   |
   ↓
Detect
   |
   ↓
Observability
   |
   ↓
Investigate
   |
   ↓
Root Cause
   |
   ↓
Resolve
```

Monitoring and observability should work together.

---

# 6. What Should We Monitor?

A production environment has multiple layers.

```
Infrastructure
    |
    ↓
Kubernetes
    |
    ↓
Containers
    |
    ↓
Applications
    |
    ↓
Services
    |
    ↓
Dependencies
    |
    ↓
Users
```

Each layer can produce monitoring signals.

---

# 7. Infrastructure Monitoring

Infrastructure monitoring includes:

```
CPU
Memory
Disk
Network
Load
Processes
File Systems
System Availability
```

For an EC2 instance:

```
EC2
  |
  +--- CPU
  +--- Memory
  +--- Disk
  +--- Network
  +--- Processes
  +--- Availability
```

---

# 8. Server Monitoring

For Linux servers, common things to monitor include:

```
CPU Utilization
Memory Utilization
Disk Utilization
Disk I/O
Network Traffic
Load Average
Process Count
Open Connections
File Descriptors
```

Example:

```
Server
  |
  +--- CPU       → 65%
  +--- Memory    → 72%
  +--- Disk      → 68%
  +--- Network   → 400 Mbps
  +--- Load      → 2.4
```

---

# 9. Application Monitoring

Application monitoring focuses on application behavior.

Monitor:

```
Request Rate
Error Rate
Response Time
Availability
Throughput
Application Exceptions
Dependency Failures
Business Metrics
```

Example:

```
API
  |
  +--- Requests      → 10,000/min
  +--- Errors        → 120/min
  +--- Error Rate    → 1.2%
  +--- Latency       → 250 ms
  +--- Availability  → 99.95%
```

---

# 10. Database Monitoring

Databases are critical production components.

Monitor:

```
Connections
Query Latency
Query Rate
CPU
Memory
Storage
IOPS
Locks
Replication
Errors
```

Example:

```
Application
     |
     ↓
  Database
     |
     +--- Connections
     +--- Query Latency
     +--- CPU
     +--- Memory
     +--- Storage
     +--- IOPS
```

---

# 11. Network Monitoring

Network monitoring helps identify communication problems.

Monitor:

```
Network Throughput
Packet Loss
Connection Errors
Latency
Network Connections
DNS Failures
Load Balancer Health
```

Example:

```
User
  |
  ↓
ALB
  |
  ↓
Service
  |
  ↓
Pod
```

A problem at any layer can affect application availability.

---

# 12. Kubernetes Monitoring

In Kubernetes, monitor multiple levels.

```
Cluster
   |
   +--- Nodes
   |
   +--- Namespaces
   |
   +--- Pods
   |
   +--- Containers
   |
   +--- Deployments
   |
   +--- Services
   |
   +--- Ingress / ALB
```

---

# 13. Kubernetes Node Monitoring

Monitor:

```
CPU
Memory
Disk
Network
Node Status
Pod Capacity
Allocatable Resources
Pressure Conditions
```

Example:

```
Node
  |
  +--- CPU
  +--- Memory
  +--- Disk
  +--- Network
  +--- Pod Count
  +--- Conditions
```

---

# 14. Kubernetes Pod Monitoring

Monitor:

```
Pod Status
Container Status
CPU
Memory
Restarts
Readiness
Liveness
OOMKilled
CrashLoopBackOff
```

Example:

```
Pod
  |
  +--- Running
  +--- Ready
  +--- CPU
  +--- Memory
  +--- Restarts
  +--- Health Probes
```

---

# 15. Kubernetes Deployment Monitoring

A Deployment should be monitored for:

```
Desired Replicas
Current Replicas
Available Replicas
Ready Replicas
Rollout Status
Failed Pods
```

Example:

```
Desired = 5
Current = 5
Available = 5
Ready = 5
```

This indicates the deployment is currently healthy from a replica
availability perspective.

---

# 16. Container Monitoring

Containers can fail even when the Kubernetes node is healthy.

Monitor:

```
Container Restarts
CPU
Memory
Exit Codes
OOMKilled
Startup Failures
Health Checks
```

---

# 17. Load Balancer Monitoring

For an ALB-based application:

```
Users
   |
   ↓
  ALB
   |
   ↓
Target Group
   |
   ↓
Kubernetes Service
   |
   ↓
  Pods
```

Monitor:

```
Healthy Targets
Unhealthy Targets
Request Count
Response Codes
Latency
Connection Errors
```

---

# 18. Availability Monitoring

Availability measures whether a service is accessible and functioning.

Example:

```
Service Available
    |
    ↓
   YES
```

or:

```
Service Available
    |
    ↓
    NO
    |
    ↓
   Alert
```

A simple availability check can use a health endpoint.

Example:

```
GET /health
```

Expected:

```
HTTP 200
```

---

# 19. Health Checks

Health checks determine whether a service is healthy.

Common types:

```
Liveness
Readiness
Startup
Application Health
Dependency Health
```

---

# 20. Liveness Check

Liveness answers:

```
"Is this application still alive?"
```

If the application becomes unhealthy, Kubernetes may restart the
container depending on the probe configuration.

---

# 21. Readiness Check

Readiness answers:

```
"Is this application ready to receive traffic?"
```

A running container is not necessarily ready.

Example:

```
Container
   |
   ↓
Running
   |
   ↓
Readiness Check
  / \
Pass Fail
 |     |
 ↓     ↓
Traffic  No Traffic
```

---

# 22. Startup Check

Startup checks are useful for applications that take time to start.

Example:

```
Container Starts
      |
      ↓
Startup Probe
      |
      ↓
Application Initialization
      |
      ↓
Ready
      |
      ↓
Readiness / Liveness
```

---

# 23. Monitoring Signals

Monitoring systems generally collect different types of signals.

The three major observability signals are:

```
Metrics
Logs
Traces
```

These provide different perspectives.

---

# 24. Metrics

Metrics are numerical measurements collected over time.

Examples:

```
CPU Usage
Memory Usage
Request Count
Error Count
Request Latency
Disk Usage
Network Traffic
```

Example:

```
CPU:
10%
20%
40%
60%
80%
```

This allows us to understand trends.

---

# 25. Logs

Logs are records of events generated by systems and applications.

Example:

```
2026-08-10 09:15:22
ERROR
Payment service failed to connect to database
```

Logs provide detailed event information.

---

# 26. Traces

Traces follow a request across distributed services.

Example:

```
User
  |
  ↓
API Gateway / ALB
  |
  ↓
User Service
  |
  ↓
Order Service
  |
  ↓
Payment Service
  |
  ↓
Database
```

A trace can show how much time the request spent in each component.

---

# 27. Metrics vs Logs vs Traces

## Metrics

Best for:

```
Trends
Alerts
Capacity
Performance
Health
```

## Logs

Best for:

```
Events
Errors
Debugging
Detailed Context
```

## Traces

Best for:

```
Request Flow
Distributed Systems
Latency Analysis
Dependency Analysis
```

---

# 28. The Three Pillars

A common observability model is:

```
             OBSERVABILITY
                  |
      +-----------+-----------+
      |           |           |
      ↓           ↓           ↓
   Metrics      Logs       Traces
      |           |           |
      ↓           ↓           ↓
 Prometheus      ELK     OpenTelemetry
      |           |           |
      ↓           ↓           ↓
   Grafana   Elasticsearch   Jaeger
                Kibana
```

These tools can work together to provide a complete operational view.

---

# 29. Monitoring Architecture

A basic architecture is:

```
┌───────────────────────┐
│       Application     │
└───────────┬───────────┘
            |
   +--------+--------+
   |        |        |
   ↓        ↓        ↓
Metrics    Logs    Traces
   |        |        |
   ↓        ↓        ↓
```

Prometheus    ELK   OpenTelemetry
|                 |
|                 ↓
|               Jaeger
|
+--------+--------+
|
↓
Grafana
|
↓
Dashboards
|
↓
Alerts

---

# 30. Real-World EKS Architecture

For an AWS EKS environment:

```
Internet
    |
    ↓
   ALB
    |
    ↓
EKS Cluster
    |
    +-----------------------+
    |                       |
    ↓                       ↓
Microservices          Kubernetes
                          |
                          ↓
                     Observability
                          |
         +----------------+----------------+
         |                |                |
         ↓                ↓                ↓
      Metrics           Logs             Traces
         |                |                |
         ↓                ↓                ↓
    Prometheus           ELK        OpenTelemetry
         |                                 |
         ↓                                 ↓
      Grafana                           Jaeger
```

---

# 31. Monitoring Data Flow

Metrics:

```
Application
    |
    ↓
Metrics Endpoint
    |
    ↓
Prometheus
    |
    ↓
Grafana
```

Logs:

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
    |
    ↓
Kibana
```

Traces:

```
Application
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

---

# 32. Prometheus

Prometheus is a monitoring and metrics platform.

It collects and stores time-series metrics.

Typical architecture:

```
Application
    |
    ↓
/metrics
    |
    ↓
Prometheus
    |
    ↓
Time-Series Database
    |
    ↓
PromQL
    |
    ↓
Grafana
```

---

# 33. Prometheus Responsibilities

Prometheus can provide:

```
Metrics Collection
Time-Series Storage
PromQL Queries
Service Discovery
Alert Rules
Target Health
```

---

# 34. Grafana

Grafana is a visualization and dashboard platform.

Typical flow:

```
Prometheus
    |
    ↓
Grafana Data Source
    |
    ↓
Dashboard
    |
    ↓
Engineer
```

Grafana can visualize metrics and other supported data sources.

---

# 35. ELK Stack

ELK stands for:

```
E = Elasticsearch
L = Logstash
K = Kibana
```

Typical architecture:

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
    |
    ↓
Kibana
    |
    ↓
Engineer
```

---

# 36. Elasticsearch

Elasticsearch stores and searches log data.

Responsibilities include:

```
Indexing
Searching
Aggregation
Storage
Querying
```

---

# 37. Logstash

Logstash processes and transforms events.

Typical flow:

```
Input
  |
  ↓
Filter
  |
  ↓
Output
```

Example:

```
Application Log
     |
     ↓
   Input
     |
     ↓
   Filter
     |
     ↓
  Parse JSON
     |
     ↓
Elasticsearch
```

---

# 38. Kibana

Kibana provides a user interface for:

```
Searching Logs
Creating Visualizations
Building Dashboards
Investigating Errors
```

Example:

```
Elasticsearch
     |
     ↓
   Kibana
     |
     ↓
Search:
ERROR
payment
timeout
```

---

# 39. OpenTelemetry

OpenTelemetry is an open-source observability framework for
instrumenting, generating, collecting, and exporting telemetry data.

It supports:

```
Metrics
Logs
Traces
```

A common architecture is:

```
Application
    |
    ↓
OpenTelemetry SDK / Instrumentation
    |
    ↓
OpenTelemetry Collector
    |
    +------→ Metrics Backend
    |
    +------→ Logs Backend
    |
    +------→ Traces Backend
```

---

# 40. OpenTelemetry Collector

The Collector acts as a telemetry processing layer.

A simplified architecture:

```
Application
    |
    ↓
OTel Collector
    |
    +--- Receivers
    |
    +--- Processors
    |
    +--- Exporters
    |
    ↓
Backends
```

The Collector can receive telemetry, process it, and export it to
different destinations.

---

# 41. Jaeger

Jaeger is a distributed tracing platform.

It helps visualize request execution across multiple services.

Example:

```
Request
   |
   ↓
Service A
   |
   ↓
Service B
   |
   ↓
Service C
   |
   ↓
Database
```

Jaeger can show the trace and individual spans.

---

# 42. OpenTelemetry + Jaeger

A common tracing architecture is:

```
Application
    |
    ↓
OpenTelemetry SDK
    |
    ↓
OpenTelemetry Collector
    |
    ↓
Jaeger
    |
    ↓
Jaeger UI
```

OpenTelemetry provides telemetry collection and standardization.

Jaeger provides distributed tracing storage/querying and visualization.

---

# 43. Complete Observability Stack

Our stack in this chapter is:

```
┌─────────────────────────────────────────┐
│               Applications              │
└────────────────────┬────────────────────┘
                     |
      +--------------+--------------+
      |              |              |
      ↓              ↓              ↓
   Metrics          Logs          Traces
      |              |              |
      ↓              ↓              ↓
Prometheus           ELK      OpenTelemetry
      |              |              |
      |              |              ↓
      |              |        OTel Collector
      |              |              |
      |              |              ↓
      |              |           Jaeger
      |              |
      |              ↓
      |         Elasticsearch
      |              |
      |              ↓
      |            Kibana
      |
      +--------------+--------------+
                     |
                     ↓
                  Grafana
```

---

# 44. Monitoring Architecture in Microservices

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
  +------→ Product Service
  |
  +------→ Order Service
                |
                +------→ Payment Service
                |
                +------→ Inventory Service
```

Monitoring:

```
All Services
     |
     +------→ Metrics → Prometheus
     |
     +------→ Logs → ELK
     |
     +------→ Traces → OpenTelemetry → Jaeger
```

---

# 45. Why Microservices Need Monitoring

In a monolithic application:

```
Request
  |
  ↓
Application
  |
  ↓
Database
```

In microservices:

```
Request
  |
  ↓
Service A
  |
  ↓
Service B
  |
  ↓
Service C
  |
  ↓
Database
  |
  ↓
External API
```

A failure can occur anywhere.

Therefore, distributed monitoring becomes critical.

---

# 46. Example Production Problem

Suppose users report:

```
"Orders are taking too long."
```

Monitoring can show:

```
Request Rate
    |
    ↓
Normal

Error Rate
    |
    ↓
Normal

Latency
    |
    ↓
Increased
```

Now investigate with traces:

```
User Request
    |
    ↓
Order Service
    |
    ↓
Payment Service
    |
    ↓
Payment API
    |
    ↓
3.2 seconds
```

The trace identifies the slow dependency.

Then inspect logs:

```
Payment API timeout
Connection retry
Response delayed
```

This demonstrates how:

```
Metrics
    +
Traces
    +
Logs
```

work together.

---

# 47. Monitoring Workflow

A practical troubleshooting workflow is:

```
1. Detect
   |
   ↓
2. Alert
   |
   ↓
3. Check Metrics
   |
   ↓
4. Identify Affected Component
   |
   ↓
5. Check Logs
   |
   ↓
6. Check Traces
   |
   ↓
7. Identify Root Cause
   |
   ↓
8. Fix
   |
   ↓
9. Validate
   |
   ↓
10. Document
```

---

# 48. Monitoring Dashboard

A production dashboard should provide a quick health overview.

Example:

```
┌──────────────────────────────────────────┐
│         Production Overview              │
├──────────────────────────────────────────┤
│ Availability       99.95%                │
│ Request Rate       12K/min               │
│ Error Rate         0.4%                  │
│ P95 Latency        240ms                 │
│ CPU                61%                   │
│ Memory             68%                   │
│ Healthy Pods       28/28                 │
│ Unhealthy Pods     0                     │
└──────────────────────────────────────────┘
```

---

# 49. What Should a Production Dashboard Show?

At minimum:

```
Availability
Request Rate
Error Rate
Latency
CPU
Memory
Pod Health
Restart Count
Deployment Status
```

For important services, also include:

```
Dependency Health
Database Health
Queue Health
Business Metrics
```

---

# 50. Monitoring Layers

A useful monitoring model is:

```
Layer 1
Infrastructure

    ↓

Layer 2
Kubernetes

    ↓

Layer 3
Containers

    ↓

Layer 4
Application

    ↓

Layer 5
Services

    ↓

Layer 6
Dependencies

    ↓

Layer 7
User Experience
```

Each layer should provide useful signals.

---

# 51. Infrastructure Layer

Monitor:

```
EC2
VPC
Network
Storage
Load Balancers
Databases
```

---

# 52. Platform Layer

For Kubernetes:

```
EKS
Nodes
Pods
Deployments
Services
Ingress
HPA
Cluster Resources
```

---

# 53. Application Layer

Monitor:

```
Request Rate
Errors
Latency
Exceptions
Dependency Calls
Business Operations
```

---

# 54. User Layer

Monitor:

```
Availability
Response Time
Error Rate
Successful Transactions
Critical User Journeys
```

---

# 55. Monitoring Frequency

Different signals can have different collection intervals.

Example:

```
CPU
   → Every few seconds

Application Metrics
   → Short interval

Logs
   → Event based

Traces
   → Request based
```

The correct interval depends on the signal and operational need.

---

# 56. Monitoring Data Retention

Retention depends on:

```
Operational Requirements
Compliance
Storage Cost
Troubleshooting Requirements
```

Example:

```
High-Resolution Metrics
    |
    ↓
Shorter Retention

Aggregated Metrics
    |
    ↓
Longer Retention

Logs
    |
    ↓
Policy-Based Retention
```

---

# 57. Monitoring Cardinality

Metric cardinality is the number of unique combinations of metric
labels.

Example:

```
http_requests_total{
    service="order",
    method="GET",
    status="200"
}
```

Too many unique labels can increase memory and storage usage.

Avoid uncontrolled high-cardinality labels such as:

```
user_id
request_id
session_id
```

as metric labels unless there is a carefully designed reason.

These values are generally better suited to logs or traces.

---

# 58. Monitoring Data Quality

Good monitoring should provide:

```
Accurate Data
Useful Labels
Consistent Naming
Correct Units
Appropriate Resolution
Appropriate Retention
```

Bad monitoring can create noise instead of useful information.

---

# 59. Monitoring Naming

Use consistent metric naming.

Examples:

```
http_requests_total

http_request_duration_seconds

process_cpu_seconds_total
```

The exact naming convention should be consistent across services.

---

# 60. Monitoring Labels

Labels provide dimensions for metrics.

Example:

```
http_requests_total{
    service="orders",
    method="GET",
    status="200"
}
```

Labels allow queries such as:

```
Requests by Service
Requests by HTTP Method
Requests by Status
```

---

# 61. Alerting

Monitoring without useful alerts can still fail operationally.

Example:

```
Metric
  |
  ↓
Threshold
  |
  ↓
Alert
  |
  ↓
Notification
  |
  ↓
Engineer
```

---

# 62. Good Alerts

A good alert should be:

```
Actionable
Relevant
Clear
Specific
Time Sensitive
```

Example:

```
"Production order-service error rate exceeded 5%
 for 10 minutes."
```

This is better than:

```
"Something is wrong."
```

---

# 63. Alert Fatigue

Too many alerts create alert fatigue.

Example:

```
500 Alerts
    |
    ↓
Engineer
    |
    ↓
Alerts Ignored
```

Good monitoring should focus on meaningful alerts.

---

# 64. Alert Severity

A common model is:

```
Critical
    |
    ↓
Immediate Action

Warning
    |
    ↓
Investigation

Informational
    |
    ↓
Awareness
```

The exact severity model depends on the organization.

---

# 65. Monitoring and Incident Management

Monitoring should connect to incident response.

```
Monitoring
    |
    ↓
Alert
    |
    ↓
Incident
    |
    ↓
Investigation
    |
    ↓
Recovery
    |
    ↓
Post-Incident Review
```

---

# 66. Monitoring During Deployment

Monitoring is especially important during deployments.

Before deployment:

```
Baseline Metrics
```

During deployment:

```
Watch Metrics
Watch Logs
Watch Health
```

After deployment:

```
Compare Metrics
Validate Logs
Validate Traces
```

---

# 67. Deployment Monitoring

Example:

```
Version A
    |
    ↓
Baseline

Deploy Version B
    |
    ↓
Monitor
    |
    +--- Error Rate
    +--- Latency
    +--- CPU
    +--- Memory
    +--- Logs
    +--- Traces
```

If the release causes degradation:

```
Stop
   |
   ↓
Investigate
   |
   ↓
Rollback if required
```

---

# 68. Monitoring and CI/CD

Monitoring completes the CI/CD feedback loop.

```
GitHub
   |
   ↓
GitHub Actions
   |
   ↓
Build
   |
   ↓
Test
   |
   ↓
Security
   |
   ↓
Deploy
   |
   ↓
Production
   |
   ↓
Monitoring
   |
   ↓
Feedback
```

---

# 69. Monitoring and GitOps

With GitOps:

```
Git
  |
  ↓
ArgoCD
  |
  ↓
EKS
  |
  ↓
Application
  |
  ↓
Monitoring
  |
  ↓
Metrics / Logs / Traces
  |
  ↓
Engineer
```

Monitoring helps verify whether the desired state resulted in a
healthy runtime state.

---

# 70. Production Monitoring Architecture

A production architecture can look like:

```
┌─────────────────────────────────────────┐
│                  AWS                    │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │              EKS                  │  │
│  │                                   │  │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ │  │
│  │  │Service │ │Service │ │Service │ │  │
│  │  │   A    │ │   B    │ │   C    │ │  │
│  │  └───┬────┘ └───┬────┘ └───┬────┘ │  │
│  │      │          │          │       │  │
│  └──────┼──────────┼──────────┼───────┘  │
│         │          │          │          │
└─────────┼──────────┼──────────┼──────────┘
          │          │          │
   +------+----------+----------+------+
   |               |                  |
   ↓               ↓                  ↓
Metrics           Logs              Traces
   |               |                  |
   ↓               ↓                  ↓
Prometheus         ELK         OpenTelemetry
   |               |                  |
   ↓               ↓                  ↓
Grafana       Elasticsearch          |
                  |                  ↓
                  ↓                Jaeger
                Kibana
```

---

# 71. Real-World Data Flow

## Metrics

```
Application
    |
    ↓
/metrics
    |
    ↓
Prometheus
    |
    ↓
PromQL
    |
    ↓
Grafana
```

## Logs

```
Application
    |
    ↓
Container Logs
    |
    ↓
Log Collection
    |
    ↓
Logstash
    |
    ↓
Elasticsearch
    |
    ↓
Kibana
```

## Traces

```
Application
    |
    ↓
OpenTelemetry
    |
    ↓
OTel Collector
    |
    ↓
Jaeger
    |
    ↓
Trace UI
```

---

# 72. Example: User Request Investigation

A user reports:

```
"Checkout is slow."
```

Start with metrics:

```
Checkout Request Rate
    |
    ↓
Error Rate
    |
    ↓
P95 / P99 Latency
```

Suppose:

```
Error Rate = Normal
Traffic = Normal
P99 Latency = High
```

Next:

```
Distributed Trace
```

Trace:

```
Checkout
   |
   ↓
Cart Service       100ms
   |
   ↓
Order Service      150ms
   |
   ↓
Payment Service    3000ms
   |
   ↓
Database           100ms
```

The trace identifies Payment Service as the likely latency source.

Then inspect logs:

```
Payment Service
   |
   ↓
Timeout
   |
   ↓
External Payment API
```

Now the investigation has moved from:

```
"Checkout is slow"
```

to:

```
"Payment Service is waiting on an external dependency."
```

---

# 73. Monitoring Tools in This Chapter

The primary tools are:

```
Prometheus
    |
    ↓
Metrics

Grafana
    |
    ↓
Visualization / Dashboards

Elasticsearch
    |
    ↓
Log Storage / Search

Logstash
    |
    ↓
Log Processing

Kibana
    |
    ↓
Log Visualization

OpenTelemetry
    |
    ↓
Telemetry Instrumentation / Collection

Jaeger
    |
    ↓
Distributed Tracing
```

---

# 74. Tool Responsibilities

| Tool           | Primary Responsibility                        |
| -------------- | --------------------------------------------- |
| Prometheus     | Metrics collection and time-series monitoring |
| Grafana        | Visualization and dashboards                  |
| Elasticsearch  | Log indexing and search                       |
| Logstash       | Log ingestion and processing                  |
| Kibana         | Log search and visualization                  |
| OpenTelemetry  | Telemetry instrumentation and collection      |
| OTel Collector | Telemetry receiving, processing, exporting    |
| Jaeger         | Distributed tracing                           |

---

# 75. Why Use Multiple Tools?

Each tool solves a different problem.

```
Prometheus
    ↓
"What is happening?"

ELK
    ↓
"What events/errors occurred?"

OpenTelemetry
    ↓
"How do we collect standardized telemetry?"

Jaeger
    ↓
"Where did this request spend time?"

Grafana
    ↓
"How do we visualize operational data?"
```

Together they provide a more complete operational picture.

---

# 76. Installation Strategy

In this chapter, installation will be covered in two major environments.

## Environment 1: Linux / EC2

Example:

```
EC2
  |
  +--- Prometheus
  +--- Grafana
  +--- Elasticsearch
  +--- Logstash
  +--- Kibana
  +--- OpenTelemetry
  +--- Jaeger
```

This helps understand installation and configuration fundamentals.

## Environment 2: Kubernetes / EKS

Example:

```
EKS
  |
  +--- Prometheus
  +--- Grafana
  +--- ELK
  +--- OpenTelemetry Collector
  +--- Jaeger
```

This represents a more realistic cloud-native deployment model.

---

# 77. Installation Learning Path

For each major tool, we will cover:

```
1. Prerequisites
2. Installation
3. Directory Structure
4. Configuration
5. Service Startup
6. Health Verification
7. Integration
8. Kubernetes Deployment
9. EKS Deployment
10. Security
11. Persistence
12. High Availability
13. Scaling
14. Backup / Recovery
15. Troubleshooting
16. Production Best Practices
```

---

# 78. Configuration Learning Path

For each tool:

```
Installation
    |
    ↓
Configuration File
    |
    ↓
Service
    |
    ↓
Data Source
    |
    ↓
Integration
    |
    ↓
Dashboard / Query
    |
    ↓
Alert
    |
    ↓
Production
```

---

# 79. Linux Installation Model

A typical Linux-based installation follows:

```
Package / Binary
    |
    ↓
Configuration
    |
    ↓
Systemd Service
    |
    ↓
Start
    |
    ↓
Enable
    |
    ↓
Verify
    |
    ↓
Monitor
```

---

# 80. Kubernetes Installation Model

A Kubernetes deployment generally follows:

```
Namespace
    |
    ↓
ConfigMap
    |
    ↓
Secret
    |
    ↓
ServiceAccount
    |
    ↓
RBAC
    |
    ↓
Deployment / StatefulSet
    |
    ↓
Service
    |
    ↓
Persistent Storage
    |
    ↓
Ingress / ALB
    |
    ↓
Monitoring
```

---

# 81. Persistence

Monitoring systems can generate large amounts of data.

Persistent storage may be required for:

```
Prometheus
Elasticsearch
Jaeger
```

Stateless components can be treated differently depending on their
architecture.

Storage design should consider:

```
Capacity
Performance
Retention
Backup
Recovery
```

---

# 82. High Availability

Production monitoring itself must be reliable.

If monitoring fails:

```
Application
    |
    ↓
Problem
    |
    X
Monitoring Down
```

The team may not detect the problem.

Therefore, critical monitoring systems should be designed with
appropriate availability.

---

# 83. Monitoring System Failure

The monitoring platform can itself become a production dependency.

Monitor:

```
Prometheus Health
Grafana Health
Elasticsearch Health
Logstash Health
Kibana Health
OTel Collector Health
Jaeger Health
```

---

# 84. Monitoring the Monitoring System

A mature platform monitors its observability infrastructure.

Example:

```
Prometheus
   |
   ↓
Monitoring
   |
   ↓
Prometheus Health

Elasticsearch
   |
   ↓
Monitoring
   |
   ↓
Cluster Health
```

---

# 85. Security

Monitoring systems can contain sensitive information.

Examples:

```
Application Logs
User Information
API Errors
Infrastructure Details
Request Metadata
Security Events
```

Therefore:

```
Authentication
    +
Authorization
    +
Encryption
    +
Network Controls
    +
Access Logging
```

are important.

---

# 86. Sensitive Data in Logs

Never intentionally log:

```
Passwords
API Keys
Access Tokens
Private Keys
Sensitive Credentials
```

Example of bad logging:

```
password=MySecretPassword
```

Better:

```
password=[REDACTED]
```

---

# 87. Monitoring Access Control

Different users may require different access.

Example:

```
Developer
    |
    ↓
Application Dashboards

DevOps
    |
    ↓
Infrastructure + Application

Security Team
    |
    ↓
Security Logs
```

Access should follow least privilege.

---

# 88. Monitoring Network Security

Monitoring systems should not necessarily be publicly accessible.

Example:

```
Internet
   |
   X
Prometheus
```

Preferred:

```
Internal Network
   |
   ↓
Prometheus
```

Access can be provided through controlled mechanisms such as VPN,
private networking, authentication, or secure ingress.

---

# 89. Production Architecture Principle

Do not treat monitoring as:

```
"Install Prometheus and Grafana."
```

Treat it as:

```
Collection
    +
Storage
    +
Processing
    +
Visualization
    +
Alerting
    +
Security
    +
Retention
    +
High Availability
    +
Backup
    +
Troubleshooting
```

---

# 90. Monitoring Best Practices

Follow these principles:

```
1. Monitor critical systems
2. Monitor user-facing services
3. Monitor dependencies
4. Use meaningful metrics
5. Centralize logs
6. Use distributed tracing for microservices
7. Create actionable alerts
8. Avoid alert fatigue
9. Protect monitoring systems
10. Protect sensitive log data
11. Control metric cardinality
12. Define retention policies
13. Use persistent storage where required
14. Design monitoring for high availability
15. Monitor the monitoring platform
16. Validate dashboards regularly
17. Test alerts
18. Test incident procedures
19. Correlate metrics, logs, and traces
20. Continuously improve observability
```

---

# 91. Common Monitoring Mistakes

## Mistake 1

Monitoring only CPU and memory.

### Problem

The infrastructure may look healthy while the application is failing.

### Better

Monitor:

```
Availability
Errors
Latency
Traffic
Dependencies
```

---

## Mistake 2

Creating hundreds of alerts.

### Problem

Alert fatigue.

### Better

Create actionable alerts.

---

## Mistake 3

No centralized logging.

### Problem

Engineers must manually inspect logs on individual servers or pods.

### Better

Centralize logs with an appropriate logging architecture.

---

## Mistake 4

No distributed tracing.

### Problem

Microservice latency becomes difficult to investigate.

### Better

Use OpenTelemetry and a tracing backend such as Jaeger.

---

## Mistake 5

Using high-cardinality metric labels.

### Problem

High memory and storage usage.

### Better

Use carefully designed labels and move high-cardinality details into
logs or traces when appropriate.

---

## Mistake 6

Exposing monitoring systems publicly.

### Problem

Security risk.

### Better

Use private networking and authentication.

---

## Mistake 7

Logging secrets.

### Problem

Credential exposure.

### Better

Redact sensitive values before logging.

---

## Mistake 8

No retention strategy.

### Problem

Storage grows continuously.

### Better

Define retention based on operational and compliance requirements.

---

## Mistake 9

No monitoring of monitoring infrastructure.

### Problem

The observability platform can fail silently.

### Better

Monitor the health of the observability platform itself.

---

# 92. Production Monitoring Checklist

```
[ ] Infrastructure monitored
[ ] Kubernetes monitored
[ ] Application monitored
[ ] Database monitored
[ ] Network monitored
[ ] Load balancer monitored
[ ] Metrics collected
[ ] Logs centralized
[ ] Traces available
[ ] Dashboards created
[ ] Alerts configured
[ ] Alert severity defined
[ ] Alert routing tested
[ ] Retention defined
[ ] Storage capacity monitored
[ ] Access control configured
[ ] Sensitive data protected
[ ] Monitoring platform secured
[ ] Backup / recovery considered
[ ] Monitoring platform health monitored
```

---

# 93. Real-World DevOps Monitoring Workflow

As a DevOps Engineer, a practical workflow can be:

```
1. Monitor infrastructure
       |
       ↓
2. Monitor Kubernetes
       |
       ↓
3. Monitor applications
       |
       ↓
4. Monitor dependencies
       |
       ↓
5. Centralize logs
       |
       ↓
6. Collect metrics
       |
       ↓
7. Collect traces
       |
       ↓
8. Build dashboards
       |
       ↓
9. Configure alerts
       |
       ↓
10. Investigate incidents
       |
       ↓
11. Resolve issues
       |
       ↓
12. Improve monitoring
```

---

# 94. Interview Question

## What is monitoring?

### Answer

Monitoring is the continuous collection and analysis of system,
infrastructure, and application signals to determine whether a system
is operating normally.

It helps detect problems such as:

```
High CPU
High Memory
High Error Rate
High Latency
Pod Failures
Disk Exhaustion
Service Unavailability
```

Monitoring primarily helps answer:

```
"Is something wrong?"
```

---

# 95. Interview Question

## What is the difference between monitoring and observability?

### Answer

Monitoring focuses on detecting known failure conditions using
predefined signals and thresholds.

Observability focuses on understanding the internal state of a system
and investigating unknown problems using telemetry such as:

```
Metrics
Logs
Traces
```

A simple way to explain it is:

```
Monitoring → Detect the problem

Observability → Understand why the problem happened
```

---

# 96. Interview Question

## What are the three major observability signals?

### Answer

The three major signals are:

```
Metrics
Logs
Traces
```

Metrics provide numerical measurements.

Logs provide detailed event information.

Traces show how requests move through distributed systems.

---

# 97. Interview Question

## Why do we need distributed tracing in microservices?

### Answer

In a microservices architecture, a single user request can travel
through multiple services and dependencies.

Without distributed tracing, identifying the slow or failing service
can be difficult.

Tracing allows us to see:

```
Request
   |
   ↓
Service A
   |
   ↓
Service B
   |
   ↓
Service C
   |
   ↓
Database
```

and identify which component contributed most to latency or failure.

---

# 98. Interview Question

## Why use Prometheus?

### Answer

Prometheus is designed for monitoring and time-series metrics.

It provides:

```
Metrics Collection
Time-Series Storage
PromQL
Service Discovery
Alert Rules
Target Monitoring
```

It is particularly useful for Kubernetes and cloud-native
environments.

---

# 99. Interview Question

## Why use Grafana?

### Answer

Grafana provides visualization and dashboards for monitoring data.

It can connect to data sources such as Prometheus and help engineers
visualize:

```
CPU
Memory
Request Rate
Error Rate
Latency
Kubernetes Health
```

It also supports alerting capabilities depending on the configured
architecture.

---

# 100. Interview Question

## What is ELK?

### Answer

ELK stands for:

```
Elasticsearch
Logstash
Kibana
```

Elasticsearch stores and searches log data.

Logstash processes and transforms log events.

Kibana provides visualization and log investigation.

---

# 101. Interview Question

## What is OpenTelemetry?

### Answer

OpenTelemetry is an open-source observability framework that provides
standardized instrumentation and collection for:

```
Metrics
Logs
Traces
```

It can collect telemetry and export it to different observability
backends.

---

# 102. Interview Question

## What is Jaeger?

### Answer

Jaeger is a distributed tracing platform used to collect, store,
query, and visualize traces.

It helps engineers understand request flows across microservices and
identify latency and dependency problems.

---

# 103. Interview Question

## How do OpenTelemetry and Jaeger work together?

### Answer

OpenTelemetry can instrument the application and collect trace
telemetry.

The telemetry can be sent through the OpenTelemetry Collector and
exported to Jaeger.

The flow is:

```
Application
    |
    ↓
OpenTelemetry
    |
    ↓
OTel Collector
    |
    ↓
Jaeger
    |
    ↓
Jaeger UI
```

OpenTelemetry is the telemetry instrumentation and collection layer,
while Jaeger is used for distributed tracing.

---

# 104. Interview Question

## How would you monitor an EKS cluster?

### Answer

I would monitor the EKS environment at multiple levels.

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
Nodes
Restarts
HPA
Resource Usage
```

Application:

```
Request Rate
Error Rate
Latency
Availability
```

I would use Prometheus for metrics, Grafana for visualization, ELK
for centralized logs, and OpenTelemetry with Jaeger for distributed
tracing.

---

# 105. Interview Question

## A production application is slow. How would you troubleshoot it?

### Answer

I would first check metrics to determine whether the problem is:

```
High Latency
High Error Rate
High CPU
High Memory
High Traffic
```

Then I would identify the affected service.

Next I would inspect application logs in the centralized logging
system.

For a microservices application, I would use distributed tracing to
identify which service or dependency is contributing to the latency.

The investigation would be:

```
Metrics
    |
    ↓
Identify Problem
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
Fix
    |
    ↓
Validate
```

---

# 106. Final Monitoring Architecture

The complete architecture for this chapter is:

```
┌─────────────────────────────────────────────────────┐
│                    USERS                            │
└───────────────────────┬─────────────────────────────┘
                        │
                        ↓
                       ALB
                        │
                        ↓
┌─────────────────────────────────────────────────────┐
│                     AWS EKS                         │
│                                                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│   │ Service A│  │ Service B│  │ Service C│         │
│   └────┬─────┘  └────┬─────┘  └────┬─────┘         │
│        │             │             │               │
└────────┼─────────────┼─────────────┼───────────────┘
         │             │             │
   ┌─────┴─────────────┴─────────────┴─────┐
   │                                        │
   ↓                                        ↓
Metrics                                   Logs
   │                                        │
   ↓                                        ↓
Prometheus                              Log Pipeline
   │                                        │
   ↓                                        ↓
Grafana                               Elasticsearch
                                            │
                                            ↓
                                          Kibana

         Traces
            │
            ↓
      OpenTelemetry
            │
            ↓
   OTel Collector
            │
            ↓
         Jaeger
            │
            ↓
       Jaeger UI
```

---

# 107. Final Monitoring Model

The complete model is:

```
INFRASTRUCTURE
      |
      ↓
KUBERNETES
      |
      ↓
APPLICATION
      |
      ↓
METRICS + LOGS + TRACES
      |
      +-------------------+
      |                   |
      ↓                   ↓
  MONITORING         OBSERVABILITY
      |                   |
      ↓                   ↓
   DETECT              INVESTIGATE
      |                   |
      +---------+---------+
                |
                ↓
             ROOT CAUSE
                |
                ↓
              FIX
                |
                ↓
             VALIDATE
                |
                ↓
            CONTINUOUS
            IMPROVEMENT
```

The tools used in this chapter provide the implementation:

```
Prometheus
    → Metrics

Grafana
    → Visualization

Elasticsearch
    → Log Storage / Search

Logstash
    → Log Processing

Kibana
    → Log Visualization

OpenTelemetry
    → Telemetry Instrumentation / Collection

OTel Collector
    → Telemetry Processing / Export

Jaeger
    → Distributed Tracing
```

The ultimate goal is:

```
"Detect problems early, understand what is happening inside the
 system, identify the root cause quickly, recover production
 safely, and continuously improve system reliability."
```
