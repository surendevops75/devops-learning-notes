# Prometheus Architecture

Prometheus architecture defines how metrics are discovered, collected, stored, queried, processed, and delivered to visualization and alerting systems.

A basic Prometheus architecture is:

```
Targets
   ↓
Service Discovery
   ↓
Prometheus
   ↓
TSDB
   ↓
PromQL
   ↓
Grafana
```

Alerting flow:

```
Prometheus
   ↓
Alerting Rules
   ↓
Alertmanager
   ↓
Notification Channels
```

In a production Kubernetes environment, the architecture becomes:

```
┌──────────────────────────────────────────────────────────────┐
│                         AWS EKS                              │
│                                                              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐             │
│  │ Order      │  │ Payment    │  │ Inventory  │             │
│  │ Service    │  │ Service    │  │ Service    │             │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘             │
│        │               │               │                    │
│        └───────────────┼───────────────┘                    │
│                        │                                    │
│                   /metrics                                  │
│                        │                                    │
│  Node Exporter         │       kube-state-metrics           │
│        │               │               │                    │
└────────┼───────────────┼───────────────┼────────────────────┘
         │               │               │
         └───────────────┼───────────────┘
                         ↓
                Service Discovery
                         ↓
                    Prometheus
                         ↓
                       TSDB
                         ↓
                       PromQL
                   ┌─────┴─────┐
                   ↓           ↓
                Grafana    Alertmanager
                               ↓
                          Notifications
```

---

# 1. Prometheus Architecture Overview

Prometheus consists of several important architectural components:

```
1. Prometheus Server

2. Service Discovery

3. Scrape Manager

4. Retrieval Layer

5. TSDB

6. PromQL Engine

7. Rule Evaluation

8. Alerting Rules

9. Recording Rules

10. Alertmanager

11. Exporters

12. Instrumented Applications

13. Remote Write / Remote Read

14. Web UI
```

Each component has a specific responsibility.

---

# 2. High-Level Architecture

```
┌─────────────────────────────────────────────┐
│              Prometheus Server              │
│                                             │
│  Service Discovery                          │
│       ↓                                     │
│  Target Manager                             │
│       ↓                                     │
│  Scrape Manager                             │
│       ↓                                     │
│  TSDB                                       │
│       ↓                                     │
│  PromQL Engine                              │
│       ↓                                     │
│  Rule Evaluation                            │
│                                             │
└───────────────┬─────────────────────────────┘
                │
         ┌──────┴──────┐
         ↓             ↓
      Grafana      Alertmanager
```

---

# 3. Prometheus Server

The Prometheus server is the central component.

It performs several major functions:

```
Service Discovery

Target Management

Scraping

Metric Ingestion

Time-Series Storage

PromQL Query Execution

Recording Rule Evaluation

Alerting Rule Evaluation
```

The Prometheus server is therefore responsible for most of the core monitoring workflow.

---

# 4. Prometheus Data Flow

The complete data flow is:

```
Target
   ↓
Service Discovery
   ↓
Target Selection
   ↓
Scrape
   ↓
Metric Parsing
   ↓
TSDB
   ↓
Query
   ↓
PromQL
   ↓
Grafana / API / Rules
```

---

# 5. Target

A target is an endpoint that exposes metrics.

Examples:

```
Application
Node Exporter
Database Exporter
Kubernetes Component
Blackbox Exporter
```

Example:

```
order-service:8080/metrics
```

Prometheus periodically scrapes the endpoint.

---

# 6. Instrumented Application

A modern application can expose Prometheus metrics directly.

Example:

```
Java Application
     ↓
Micrometer / OpenTelemetry / Prometheus Client
     ↓
/metrics
     ↓
Prometheus
```

The application can expose:

```
HTTP Requests

HTTP Errors

Request Duration

JVM Metrics

Business Metrics
```

---

# 7. Exporter Architecture

Some systems do not natively expose Prometheus metrics.

An exporter can collect information and expose it in Prometheus format.

Architecture:

```
System
   ↓
Exporter
   ↓
/metrics
   ↓
Prometheus
```

Examples:

```
Linux
   ↓
Node Exporter

PostgreSQL
   ↓
PostgreSQL Exporter

NGINX
   ↓
NGINX Exporter
```

---

# 8. Node Exporter Architecture

Node Exporter provides Linux host metrics.

```
Linux Node
    ↓
Node Exporter
    ↓
:9100/metrics
    ↓
Prometheus
```

Typical metrics include:

```
CPU

Memory

Disk

Filesystem

Network

Load
```

---

# 9. Kubernetes Metrics Architecture

A typical Kubernetes monitoring architecture contains multiple metric sources.

```
Kubernetes Nodes
     ↓
Node Exporter

Kubernetes Objects
     ↓
kube-state-metrics

Applications
     ↓
/metrics
```

All of these can be discovered and scraped by Prometheus.

---

# 10. Kubernetes Architecture

```
┌──────────────────── EKS ────────────────────┐
│                                             │
│  Node 1                                     │
│    ├── Application Pods                     │
│    └── Node Exporter                        │
│                                             │
│  Node 2                                     │
│    ├── Application Pods                     │
│    └── Node Exporter                        │
│                                             │
│  kube-state-metrics                         │
│                                             │
└───────────────────┬─────────────────────────┘
                    ↓
            Kubernetes Discovery
                    ↓
               Prometheus
```

---

# 11. Service Discovery

Service discovery allows Prometheus to dynamically discover targets.

This is especially important in:

```
Kubernetes

Cloud Environments

Dynamic Infrastructure

Auto Scaling Environments
```

Without service discovery, engineers would have to manually maintain target lists.

---

# 12. Static Discovery

Static discovery uses explicitly configured targets.

Example:

```
Prometheus
   |
   +-- server1:9100
   +-- server2:9100
   +-- server3:9100
```

This is simple but difficult to maintain in dynamic environments.

---

# 13. Dynamic Discovery

Dynamic discovery works like:

```
Infrastructure
     ↓
Discovery System
     ↓
Prometheus
     ↓
Automatically discovered targets
```

When new workloads appear, Prometheus can discover them according to the configured discovery mechanism.

---

# 14. Kubernetes Service Discovery

In Kubernetes:

```
Pod Created
    ↓
Kubernetes API
    ↓
Prometheus Discovery
    ↓
Target Discovered
    ↓
Scrape
```

When a pod disappears:

```
Pod Deleted
    ↓
Discovery Updates
    ↓
Target Removed
```

This is much better than maintaining static IP addresses.

---

# 15. Service Discovery Components

Prometheus can discover targets through supported mechanisms such as:

```
Kubernetes

EC2

Consul

DNS

File-based Discovery
```

The choice depends on the environment.

---

# 16. Scrape Manager

The scrape manager controls the collection process.

Conceptually:

```
Target List
   ↓
Scrape Manager
   ↓
Periodic Scrapes
   ↓
Metrics
   ↓
Storage
```

It manages:

```
Which targets to scrape

When to scrape

Scrape configuration

Scrape timeouts

Scrape errors
```

---

# 17. Scrape Interval

Example:

```
scrape_interval: 15s
```

Prometheus attempts to scrape the configured target approximately every 15 seconds.

Shorter intervals provide more frequent measurements but increase:

```
Network Traffic

CPU

Storage

Query Data Volume
```

---

# 18. Scrape Timeout

The scrape timeout determines how long Prometheus waits for a target response.

Example:

```
Request
   ↓
Target
   ↓
Response
```

If the target takes too long:

```
Timeout
   ↓
Scrape Failure
```

Timeout values should be appropriate for the target.

---

# 19. Scrape Lifecycle

A scrape follows this general flow:

```
Target Selected
    ↓
HTTP Request
    ↓
/metrics
    ↓
Response
    ↓
Parse Metrics
    ↓
Apply Processing
    ↓
Store Samples
```

---

# 20. Scrape Failure

If scraping fails:

```
Prometheus
    ↓
Target
    X
Connection Failure
```

Prometheus records scrape-related health information.

The `up` metric is commonly used to determine whether a scrape succeeded.

Example:

```
up{job="order-service"} 1
```

or:

```
up{job="order-service"} 0
```

---

# 21. `up` Metric

The `up` metric is generated by Prometheus for scrape targets.

Typical meaning:

```
1 = scrape succeeded

0 = scrape failed
```

Example:

```
up{
    job="node-exporter",
    instance="10.0.1.10:9100"
} 1
```

---

# 22. Target Metadata

Prometheus maintains information about targets.

Important information includes:

```
Job

Instance

Address

Health

Last Scrape

Scrape Duration

Scrape Error
```

This is useful when troubleshooting monitoring failures.

---

# 23. Relabeling Architecture

Relabeling allows Prometheus to transform discovered target information.

Architecture:

```
Discovery
    ↓
Discovered Labels
    ↓
Target Relabeling
    ↓
Final Target
    ↓
Scrape
```

It can be used to:

```
Keep Targets

Drop Targets

Modify Labels

Modify Addresses
```

---

# 24. Metric Relabeling

Metric relabeling happens after scraping.

Architecture:

```
Target
   ↓
Scrape
   ↓
Metric Samples
   ↓
Metric Relabeling
   ↓
TSDB
```

It can be used to remove unnecessary metrics or modify labels.

---

# 25. Target Relabeling vs Metric Relabeling

Target relabeling:

```
Before scrape
```

Used for:

```
Target Selection

Target Label Modification
```

Metric relabeling:

```
After scrape
```

Used for:

```
Metric Filtering

Metric Label Modification
```

---

# 26. Time-Series Database

Prometheus stores metrics in its TSDB.

TSDB stands for:

```
Time Series Database
```

It is designed specifically for time-series data.

Each sample has:

```
Metric Identity

Labels

Value

Timestamp
```

---

# 27. TSDB Data Model

Example:

```
http_requests_total{
    service="order",
    status="200"
}
```

Value:

```
15000
```

Timestamp:

```
T1
```

Later:

```
15200
```

Timestamp:

```
T2
```

Prometheus stores the evolution of this metric over time.

---

# 28. Time-Series Identity

A time series is identified by:

```
Metric Name
+
Complete Label Set
```

Example:

```
http_requests_total{
    service="order",
    status="200"
}
```

is different from:

```
http_requests_total{
    service="order",
    status="500"
}
```

---

# 29. TSDB Components

Prometheus TSDB internally manages time-series data using structures optimized for:

```
Ingestion

Compression

Indexing

Querying

Retention
```

The implementation details are handled internally by Prometheus.

For operations, the important concepts are:

```
Samples

Series

Blocks

WAL

Compaction
```

---

# 30. Write-Ahead Log

Prometheus uses a write-ahead log, commonly referred to as WAL.

Conceptually:

```
Incoming Samples
      ↓
     WAL
      ↓
    TSDB
      ↓
   Blocks
```

The WAL helps provide durability for recently ingested samples.

---

# 31. Why WAL Is Important

If Prometheus unexpectedly stops:

```
Process Crash
    ↓
Restart
    ↓
WAL Replay
    ↓
Recover Recent Data
```

The WAL is therefore an important part of Prometheus storage reliability.

---

# 32. TSDB Blocks

Prometheus stores data in blocks.

Conceptually:

```
Recent Samples
     ↓
    WAL
     ↓
   Head
     ↓
  Blocks
     ↓
Compaction
```

Blocks contain time-series data and indexes required for querying.

---

# 33. Head Block

Recently ingested data is kept in the in-memory head portion of the TSDB.

Conceptually:

```
New Samples
    ↓
   Head
    ↓
Persistent Block
```

The exact internal implementation can evolve between Prometheus versions, but operationally the important idea is that recent data is actively ingested before becoming persisted blocks.

---

# 34. Compaction

Prometheus periodically compacts blocks.

Conceptually:

```
Small Blocks
   ↓
Compaction
   ↓
Larger Blocks
```

Compaction helps improve:

```
Storage Efficiency

Query Efficiency

Long-Term TSDB Organization
```

---

# 35. Retention

Prometheus can retain metrics based on configured retention settings.

Retention affects:

```
Disk Usage

Query Range

Storage Requirements
```

Example:

```
15 days
```

means Prometheus keeps approximately that amount of local historical data, subject to the configured retention policy and storage behavior.

---

# 36. Storage Growth

Storage usage depends on:

```
Number of Series

Samples Per Second

Scrape Interval

Retention

Metric Cardinality
```

The most important factor to control carefully is often active series count.

---

# 37. PromQL Engine

PromQL is the query engine used to retrieve and analyze Prometheus data.

Architecture:

```
User / Grafana
      ↓
   PromQL
      ↓
 Query Engine
      ↓
    TSDB
      ↓
  Query Result
```

---

# 38. Instant Query

An instant query evaluates an expression at a single point in time.

Example:

```
up
```

It can return the current value of each matching series.

---

# 39. Range Query

A range query evaluates an expression over a time range.

Example concept:

```
rate(http_requests_total[5m])
```

The query uses historical samples from the selected range.

Grafana commonly sends range queries to build time-series graphs.

---

# 40. PromQL Execution

A simplified query flow is:

```
Query
  ↓
Parse
  ↓
Select Series
  ↓
Read Samples
  ↓
Apply Functions
  ↓
Aggregate
  ↓
Return Result
```

---

# 41. PromQL Example

Query:

```
http_requests_total
```

This selects matching time series.

Label filter:

```
http_requests_total{
    service="order"
}
```

Rate:

```
rate(http_requests_total[5m])
```

Aggregation:

```
sum by (service) (
  rate(http_requests_total[5m])
)
```

---

# 42. Query Performance

PromQL queries can become expensive when:

```
Many Series

Long Time Ranges

Complex Aggregations

High Cardinality

Large Dashboards
```

are involved.

Production environments should monitor and optimize expensive queries.

---

# 43. Recording Rules

Recording rules precompute expressions.

Architecture:

```
Expensive PromQL
     ↓
Recording Rule
     ↓
Stored Result Metric
     ↓
Dashboard
```

Example:

```
service:http_requests_per_second
```

This can be queried later instead of repeatedly calculating the original expensive expression.

---

# 44. Recording Rule Architecture

```
Raw Metrics
    ↓
PromQL Expression
    ↓
Rule Evaluation
    ↓
New Time Series
    ↓
TSDB
```

Recording rules are useful for frequently used queries.

---

# 45. Alerting Rules

Alerting rules evaluate PromQL expressions and generate alerts when conditions are met.

Architecture:

```
Metrics
   ↓
PromQL
   ↓
Alert Rule
   ↓
Condition True
   ↓
Alert
   ↓
Alertmanager
```

---

# 46. Alert Evaluation

Example:

```
CPU > 80%
```

Prometheus evaluates the expression periodically.

If:

```
CPU = 70%
   ↓
No Alert
```

If:

```
CPU = 85%
   ↓
Condition True
```

If a configured `for` duration is satisfied:

```
Alert Firing
```

---

# 47. Alertmanager Architecture

Prometheus sends alerts to Alertmanager.

Architecture:

```
Prometheus
    ↓
Alert
    ↓
Alertmanager
    ↓
Grouping
    ↓
Deduplication
    ↓
Routing
    ↓
Notification
```

---

# 48. Alertmanager Responsibilities

Alertmanager handles:

```
Alert Grouping

Alert Deduplication

Alert Routing

Silencing

Inhibition

Notification Delivery
```

Prometheus detects the condition.

Alertmanager manages the operational notification workflow.

---

# 49. Alert Routing

Example:

```
Critical Alert
    ↓
Pager / On-Call

Warning Alert
    ↓
Team Channel

Development Alert
    ↓
Development Team
```

Routing can be based on alert labels.

---

# 50. Alert Grouping

Suppose 20 pods fail at once.

Without grouping:

```
Alert
Alert
Alert
Alert
Alert
...
```

With grouping:

```
Kubernetes Deployment Failure
   ↓
One Grouped Notification
```

This reduces alert noise.

---

# 51. Alert Deduplication

If Prometheus repeatedly sends the same alert:

```
Alert
Alert
Alert
```

Alertmanager can recognize duplicates and prevent unnecessary repeated notifications according to its configured behavior.

---

# 52. Silencing

During planned maintenance:

```
Alert
   ↓
Silence
```

This prevents selected alerts from generating notifications during the maintenance window.

Silences should be used carefully.

---

# 53. Inhibition

Inhibition prevents certain alerts from notifying when another related alert is already firing.

Example:

```
Entire Node Down
    ↓
Inhibit
    ↓
Multiple Pod Alerts
```

This reduces noise during cascading failures.

---

# 54. Grafana Architecture

Grafana commonly connects to Prometheus as a data source.

Architecture:

```
Grafana
   ↓
Prometheus API
   ↓
PromQL
   ↓
TSDB
   ↓
Query Result
   ↓
Grafana Panel
```

Grafana does not replace Prometheus storage.

---

# 55. Grafana Dashboard Flow

```
User
  ↓
Grafana Dashboard
  ↓
PromQL Query
  ↓
Prometheus
  ↓
TSDB
  ↓
Result
  ↓
Graph
```

---

# 56. Prometheus HTTP API

Prometheus exposes an HTTP API.

It allows clients such as:

```
Grafana

Automation

Custom Applications
```

to query Prometheus.

Conceptually:

```
Client
   ↓
Prometheus API
   ↓
PromQL
   ↓
Query Engine
```

---

# 57. Prometheus Web UI

Prometheus provides its own web interface.

It can be used for:

```
Querying Metrics

Inspecting Targets

Checking Rules

Debugging
```

Grafana is generally more suitable for production dashboards.

---

# 58. Prometheus Web UI Architecture

```
Browser
   ↓
Prometheus Web UI
   ↓
Prometheus Server
   ↓
PromQL
   ↓
TSDB
```

---

# 59. Remote Write

Prometheus can send samples to an external remote storage system using remote write.

Architecture:

```
Targets
   ↓
Prometheus
   ↓
Local TSDB
   |
   └────→ Remote Write
              ↓
         Remote Storage
```

This can support:

```
Long-Term Storage

Centralized Metrics

Multi-Cluster Monitoring
```

---

# 60. Remote Read

Remote read allows Prometheus to query data from a compatible external storage system.

Conceptually:

```
PromQL
   ↓
Local Data
   +
Remote Data
   ↓
Query Result
```

The exact behavior depends on the configured remote storage architecture.

---

# 61. Local vs Remote Storage

Local storage:

```
Prometheus
    ↓
Local TSDB
```

Advantages:

```
Simple

Fast Local Access

Easy Deployment
```

Limitations:

```
Node-Level Failure Impact

Limited Long-Term Retention

Limited Horizontal Scaling
```

Remote storage:

```
Prometheus
    ↓
Remote Storage
```

Advantages:

```
Long Retention

Centralization

Cross-Cluster Architecture
```

Additional complexity:

```
Network

Cost

Backend Operations
```

---

# 62. Prometheus Federation

Federation allows one Prometheus server to collect selected metrics from another Prometheus server.

Architecture:

```
Cluster Prometheus
      ↓
   /federate
      ↓
Global Prometheus
```

This can be useful for hierarchical monitoring architectures.

---

# 63. Federation Example

Suppose:

```
Production Cluster
   ↓
Prometheus A

Staging Cluster
   ↓
Prometheus B
```

A central Prometheus can collect selected metrics from each environment.

```
Prometheus A ──┐
               ├──→ Global Prometheus
Prometheus B ──┘
```

---

# 64. Federation vs Remote Write

Federation:

```
Prometheus
   ↓
Selective Metrics
   ↓
Another Prometheus
```

Remote write:

```
Prometheus
   ↓
Samples
   ↓
Remote Storage
```

The architecture should be selected based on the monitoring requirement.

---

# 65. Multi-Cluster Monitoring

Suppose an organization has:

```
EKS Production
EKS Staging
EKS Development
```

Possible architecture:

```
Production Prometheus
      |
Staging Prometheus
      |
Development Prometheus
      |
      ↓
Central Metrics Platform
```

A central metrics architecture can provide cross-cluster visibility.

---

# 66. Prometheus High-Level Production Architecture

```
┌──────────────────────────────────────────────────────┐
│                     EKS Cluster                      │
│                                                      │
│  Applications ───────────────→ /metrics              │
│       │                                              │
│  Node Exporter ─────────────→ /metrics               │
│       │                                              │
│  kube-state-metrics ────────→ /metrics               │
│                                                      │
└───────────────────────┬──────────────────────────────┘
                        ↓
                Kubernetes Discovery
                        ↓
                   Prometheus
                        │
           ┌────────────┼────────────┐
           ↓            ↓            ↓
          TSDB       Rule Engine   HTTP API
           │            │            │
           │            ↓            ↓
           │       Alertmanager    Grafana
           │
           ↓
      Remote Write
           ↓
   Long-Term Storage
```

---

# 67. Prometheus HA Architecture

A common high-availability model uses multiple Prometheus instances.

```
Targets
   ↓
┌───────────────┐
│               │
↓               ↓
```

Prometheus A    Prometheus B
│               │
└───────┬───────┘
↓
Consumers

Both Prometheus instances may scrape the same targets.

This improves availability but introduces duplicate samples.

A complete HA architecture must therefore consider:

```
Deduplication

Query Layer

Remote Storage

Alert Deduplication
```

---

# 68. Why Two Prometheus Servers?

Suppose:

```
Prometheus A
    ↓
Failure
```

Without HA:

```
Monitoring unavailable
```

With:

```
Prometheus A
Prometheus B
```

if A fails:

```
Prometheus B
    ↓
Continue Monitoring
```

The exact HA behavior depends on how dashboards, alerts, and long-term storage are configured.

---

# 69. Duplicate Data in HA

If two Prometheus servers scrape the same targets:

```
Target
  ↓
┌──────────────┐
↓              ↓
```

Prometheus A   Prometheus B

Both collect the same metric.

Therefore a downstream system may need to identify replicas and deduplicate data where appropriate.

---

# 70. HA Labels

A common HA architecture adds identifying labels such as:

```
cluster

replica
```

For example:

```
cluster="production"

replica="prometheus-01"
```

These labels can help downstream systems identify duplicate samples.

---

# 71. Thanos-Style Architecture

For large-scale Prometheus environments, a long-term scalable architecture may include a system such as Thanos.

Conceptually:

```
Prometheus A ──┐
               │
Prometheus B ──┼──→ Object Storage
               │
Prometheus C ──┘
                     ↓
                 Query Layer
                     ↓
                  Grafana
```

The exact architecture depends on the selected long-term storage/query platform.

---

# 72. Object Storage

In AWS environments, object storage such as Amazon S3 can be used by compatible long-term Prometheus architectures.

Conceptually:

```
Prometheus
    ↓
Long-Term Metrics System
    ↓
S3
    ↓
Historical Metrics
```

This is useful when local Prometheus retention is insufficient.

---

# 73. Production AWS Architecture

A larger AWS architecture could be:

```
AWS
 |
 +-- EKS Production
 |      |
 |      +-- Prometheus
 |
 +-- EKS Staging
 |      |
 |      +-- Prometheus
 |
 +-- EKS Development
        |
        +-- Prometheus
```

Then:

```
Cluster Prometheus
      ↓
Central Metrics Layer
      ↓
    S3 / Long-Term Storage
      ↓
    Query Layer
      ↓
    Grafana
```

---

# 74. Prometheus Security Architecture

Prometheus should be treated as internal infrastructure.

A secure architecture is:

```
Applications
     ↓
Private Network
     ↓
Prometheus
     ↓
Internal Grafana
```

Avoid unnecessary public exposure.

---

# 75. Kubernetes Network Security

Use controls such as:

```
NetworkPolicies

Security Groups

Private Subnets

Internal Load Balancers
```

to restrict access.

Example:

```
Application
    ↓
Prometheus
```

Allow only required communication.

---

# 76. Authentication

Prometheus itself may need to be protected behind:

```
Authentication

Reverse Proxy

Network Access Controls

Identity-Aware Access
```

The exact security architecture depends on the deployment environment.

---

# 77. TLS

For sensitive environments, secure communication may use:

```
HTTPS

TLS
```

Example:

```
Client
   ↓
  TLS
   ↓
Prometheus
```

Similarly, secure connections may be required between components depending on the architecture.

---

# 78. Prometheus Resource Management

Prometheus is itself a production workload.

Configure:

```
CPU Requests

CPU Limits

Memory Requests

Memory Limits

Persistent Storage
```

based on actual workload requirements.

---

# 79. Prometheus Memory Considerations

Memory usage is influenced by:

```
Active Time Series

Label Cardinality

Scrape Volume

Query Load

Rule Evaluation
```

Large environments require capacity planning.

---

# 80. Prometheus CPU Considerations

CPU usage can increase because of:

```
High Scrape Rate

Many Targets

Complex PromQL

Recording Rules

Alert Rules

Query Concurrency
```

Monitor Prometheus itself with appropriate metrics.

---

# 81. Prometheus Disk Considerations

Disk usage depends on:

```
Samples

Series

Retention

Block Size

Compaction
```

Monitor:

```
Disk Utilization

Disk I/O

Storage Growth
```

---

# 82. Prometheus Monitoring Itself

Prometheus exposes its own metrics.

This allows Prometheus to monitor:

```
Scrape Duration

Scrape Errors

Query Performance

TSDB Health

Rule Evaluation

Storage

Process Resources
```

---

# 83. Important Self-Monitoring Signals

Examples include metrics related to:

```
Prometheus Process CPU

Prometheus Memory

Scrape Duration

Scrape Samples

TSDB Storage

Rule Evaluation

Query Performance
```

The exact metric names can vary by Prometheus version and configuration.

---

# 84. Monitoring Scrape Health

Monitor:

```
up

scrape duration

scrape samples

scrape errors
```

This helps identify whether Prometheus is successfully collecting data.

---

# 85. Monitoring Query Health

Monitor:

```
Query Duration

Query Failures

Query Concurrency

Expensive Queries
```

This is especially important when many Grafana dashboards query the same Prometheus server.

---

# 86. Monitoring TSDB Health

Monitor:

```
Disk Usage

Active Series

Samples Ingested

Compaction

WAL

Block Health
```

This helps detect storage problems early.

---

# 87. Production Failure Scenario: Prometheus Down

Problem:

```
Prometheus
   X
```

Impact:

```
No New Local Metrics

Dashboards May Lose Data

Local Alert Evaluation Stops
```

Response:

```
Detect Prometheus Failure

    ↓

Check Pod / Instance

    ↓

Check CPU / Memory

    ↓

Check Disk

    ↓

Check Logs

    ↓

Restart / Recover

    ↓

Verify Scraping

    ↓

Verify Alerts
```

---

# 88. Production Failure Scenario: Prometheus OOM

Symptoms:

```
Prometheus Restarting

OOMKilled

Memory Usage High
```

Possible causes:

```
High Cardinality

Large Number of Series

Expensive Queries

Too Many Targets

Insufficient Resources
```

Investigation:

```
Check Active Series

Check Query Load

Check Metric Cardinality

Check Resource Limits
```

---

# 89. Production Failure Scenario: Disk Full

Symptoms:

```
Prometheus Cannot Write

Compaction Problems

Storage Errors
```

Response:

```
Check Disk

Check Retention

Check Series Growth

Check WAL

Check Blocks

Expand Storage

Review Cardinality
```

---

# 90. Production Failure Scenario: Target Down

Problem:

```
up == 0
```

Troubleshooting:

```
Check Pod

Check Service

Check Endpoint

Check Port

Check Network

Check /metrics

Check Prometheus Target Status
```

---

# 91. Production Failure Scenario: High Query Latency

Symptoms:

```
Grafana dashboards slow
```

Possible causes:

```
High Cardinality

Large Query Range

Expensive Aggregations

Too Many Dashboards

Insufficient Prometheus Resources
```

Response:

```
Optimize PromQL

Use Recording Rules

Reduce Query Range

Reduce Cardinality

Scale Architecture
```

---

# 92. Production Failure Scenario: Alert Not Firing

Troubleshooting:

```
Check Metric Exists

   ↓
Check PromQL

   ↓
Check Alert Rule

   ↓
Check Evaluation

   ↓
Check `for`

   ↓
Check Alert State

   ↓
Check Alertmanager

   ↓
Check Notification Route
```

---

# 93. Production Failure Scenario: Alert Firing but No Notification

Architecture:

```
Prometheus
   ↓
Alert Firing
   ↓
Alertmanager
   ↓
Notification
```

If no notification:

```
Check Alertmanager

Check Routing

Check Grouping

Check Silences

Check Inhibition

Check Receiver Configuration
```

---

# 94. Prometheus Capacity Planning

Before production deployment, estimate:

```
Number of Targets

Scrape Interval

Metrics per Target

Active Series

Samples per Second

Retention

Query Load

Alert Rules

Recording Rules
```

---

# 95. Sample Rate

A simplified model:

```
Samples Per Second
    ≈
Active Series
    ÷
Scrape Interval
```

For example, if:

```
100,000 active series
```

and:

```
15-second scrape interval
```

then the approximate sample ingestion rate is:

```
100,000 / 15
≈ 6,667 samples/sec
```

This is a simplified estimate; actual ingestion depends on target behavior and metric availability.

---

# 96. Storage Planning

A simplified conceptual model:

```
Storage
   ≈
Samples/sec
   ×
Retention
   ×
Bytes/sample
```

Actual Prometheus storage usage depends on:

```
Compression

Series Metadata

Labels

Indexes

Blocks

WAL
```

Therefore, benchmark real workloads rather than relying only on theoretical calculations.

---

# 97. Cardinality Planning

Before deploying application metrics:

```
Review Labels

Identify Unbounded Values

Estimate Series Count

Remove Unnecessary Dimensions
```

For example:

Bad:

```
request_id
```

Better:

```
method

status

endpoint
```

---

# 98. Prometheus Architecture for Microservices

Example:

```
┌──────────────┐
│ User Service │
└──────┬───────┘
       │
┌──────▼───────┐
│ Product      │
│ Service      │
└──────┬───────┘
       │
┌──────▼───────┐
│ Order        │
│ Service      │
└──────┬───────┘
       │
┌──────▼───────┐
│ Payment      │
│ Service      │
└──────┬───────┘
       │
┌──────▼───────┐
│ Inventory    │
│ Service      │
└──────┬───────┘
       │
       ↓
  Prometheus
       ↓
    Grafana
```

Each service exposes standardized metrics.

---

# 99. Standard Microservice Metrics

A microservice should ideally expose metrics for:

```
Requests

Errors

Latency

Dependencies

Resource Usage
```

Examples:

```
http_requests_total

http_request_duration_seconds

active_connections

database_queries_total

queue_messages_total
```

---

# 100. Service-Level Architecture

For every service:

```
Application
   |
   +-- Metrics
   |
   +-- Logs
   |
   +-- Traces
   |
   ↓
Observability Platform
```

Prometheus handles the metrics layer.

---

# 101. Prometheus and ELK

Metrics and logs complement each other.

Example:

```
Prometheus
   ↓
Error Rate ↑
```

Then:

```
ELK
   ↓
Search Logs
```

Logs may show:

```
Database connection timeout
```

This provides more detail than the metric alone.

---

# 102. Prometheus and Jaeger

Prometheus:

```
Service p95 latency ↑
```

Jaeger:

```
Which request is slow?
```

Example:

```
Order
   ↓
Payment
   ↓
External API
         ↑
      2.5 sec
```

Prometheus detects the problem.

Jaeger helps identify the request path.

---

# 103. Complete Observability Architecture

```
┌──────────────────────────────────────────────────────┐
│                     EKS                              │
│                                                      │
│ Applications                                          │
│     │                                                │
│     ├──────── Metrics ─────→ Prometheus               │
│     │                           │                     │
│     ├──────── Logs ─────────→ ELK                    │
│     │                           │                     │
│     └──────── Traces ───────→ OTel → Jaeger           │
│                                                      │
└──────────────────────────┬───────────────────────────┘
                           │
                           ↓
                       Grafana
                           │
                           ↓
                    Engineering Team
```

---

# 104. Production Architecture Example

Consider:

```
AWS
  ↓
EKS
  ↓
ALB
  ↓
Microservices
  ↓
Prometheus
  ↓
Grafana
```

Supporting components:

```
Node Exporter

kube-state-metrics

Alertmanager
```

For traces:

```
OpenTelemetry

Jaeger
```

For logs:

```
ELK
```

---

# 105. End-to-End Production Monitoring

Request:

```
User
  ↓
ALB
  ↓
Order Service
  ↓
Payment Service
  ↓
Inventory Service
  ↓
PostgreSQL
```

Metrics:

```
Prometheus
  ↓
Request Rate
Error Rate
Latency
```

Logs:

```
ELK
  ↓
Application Errors
```

Traces:

```
OpenTelemetry
  ↓
Jaeger
  ↓
Request Dependency
```

---

# 106. Monitoring Incident Workflow

Incident:

```
Checkout is slow.
```

Step 1:

```
Prometheus
```

Check:

```
Error Rate

Request Rate

p95

p99
```

Step 2:

```
Identify affected service.
```

Step 3:

```
ELK
```

Search:

```
Errors

Timeouts

Exceptions
```

Step 4:

```
Jaeger
```

Find:

```
Slow Span
```

Step 5:

```
Identify dependency.
```

Step 6:

```
Check recent deployment.
```

Step 7:

```
Fix / Rollback.
```

Step 8:

```
Verify metrics.
```

---

# 107. Prometheus Architecture Best Practices

Use:

```
Service Discovery

Appropriate Scrape Intervals

Controlled Cardinality

Recording Rules

Alerting Rules

Resource Limits

Persistent Storage

Monitoring of Prometheus

High Availability where required

Long-Term Storage where required
```

---

# 108. Prometheus Architecture Anti-Pattern

Avoid:

```
One Prometheus
    ↓
Thousands of High-Cardinality Metrics
    ↓
Unlimited Retention
    ↓
Huge Grafana Queries
    ↓
No Resource Planning
```

This can result in:

```
High Memory

High CPU

Slow Queries

Storage Problems

Monitoring Failure
```

---

# 109. Prometheus Architecture Anti-Pattern

Avoid:

```
Every Metric
   ↓
user_id
   ↓
request_id
   ↓
session_id
   ↓
Unlimited Series
```

This can create severe cardinality problems.

---

# 110. Prometheus Architecture Anti-Pattern

Avoid:

```
Grafana
   ↓
Extremely Expensive PromQL
   ↓
Every 5 Seconds
   ↓
Hundreds of Dashboards
```

This can create excessive query load.

Use:

```
Recording Rules

Query Optimization

Appropriate Dashboard Refresh
```

---

# 111. Prometheus Architecture Anti-Pattern

Avoid:

```
Single Prometheus
   ↓
No Persistent Storage
   ↓
No Backup / Recovery Strategy
   ↓
Critical Monitoring
```

Production architecture should consider failure and recovery requirements.

---

# 112. Prometheus Architecture Anti-Pattern

Avoid:

```
Public Internet
    ↓
Prometheus
    ↓
Internal Infrastructure Metrics
```

Prometheus should generally be protected as internal infrastructure.

---

# 113. Production Architecture Layers

A mature Prometheus deployment can be understood in layers:

```
Layer 1
Applications / Exporters

Layer 2
Service Discovery

Layer 3
Scraping

Layer 4
TSDB

Layer 5
PromQL

Layer 6
Rules

Layer 7
Alertmanager

Layer 8
Visualization

Layer 9
Long-Term Storage
```

---

# 114. Layer 1: Metric Sources

Sources include:

```
Applications

Node Exporter

kube-state-metrics

Database Exporters

Blackbox Exporter
```

---

# 115. Layer 2: Discovery

Discovery identifies:

```
What should be monitored?

Where is it?

What labels should it have?
```

Kubernetes discovery is especially important in EKS.

---

# 116. Layer 3: Collection

Prometheus:

```
Scrapes Targets

Parses Metrics

Applies Relabeling

Stores Samples
```

---

# 117. Layer 4: Storage

TSDB stores:

```
Time Series

Samples

Labels

Timestamps
```

Storage must be sized according to:

```
Series

Sample Rate

Retention
```

---

# 118. Layer 5: Query

PromQL allows:

```
Selection

Filtering

Aggregation

Rate Calculation

Histogram Analysis

Comparisons
```

---

# 119. Layer 6: Rules

Rules include:

```
Recording Rules

Alerting Rules
```

Recording rules improve query efficiency.

Alerting rules detect operational conditions.

---

# 120. Layer 7: Alerting

Alertmanager handles:

```
Grouping

Routing

Deduplication

Silencing

Inhibition

Notifications
```

---

# 121. Layer 8: Visualization

Grafana provides:

```
Dashboards

Panels

Queries

Alerts / Alert Views

Exploration
```

---

# 122. Layer 9: Long-Term Storage

Large environments may require:

```
Remote Write

Long-Term Metrics Storage

Object Storage

Centralized Query
```

This allows metrics to remain available beyond local Prometheus retention.

---

# 123. Real-World Production Architecture

```
┌─────────────────────────────────────────────────────────┐
│                         AWS                             │
│                                                         │
│  ┌──────────────────── EKS ─────────────────────────┐  │
│  │                                                   │  │
│  │ Applications                                      │  │
│  │   │                                               │  │
│  │   ├── /metrics                                   │  │
│  │   │                                               │  │
│  │   ├── Node Exporter                              │  │
│  │   │                                               │  │
│  │   └── kube-state-metrics                         │  │
│  │                                                   │  │
│  └───────────────────┬───────────────────────────────┘  │
│                      │                                  │
│                      ↓                                  │
│              Kubernetes Discovery                       │
│                      │                                  │
│                      ↓                                  │
│                 Prometheus                              │
│                      │                                  │
│              ┌───────┼────────┐                         │
│              ↓       ↓        ↓                         │
│             TSDB   PromQL    Rules                      │
│              │       │        │                         │
│              │       ↓        ↓                         │
│              │    Grafana  Alertmanager                 │
│              │                 │                        │
│              ↓                 ↓                        │
│        Remote Storage       Notifications                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

# 124. Architecture Decision: Static vs Dynamic Discovery

## Static

Use when:

```
Small Infrastructure

Stable Targets

Simple Environment
```

## Dynamic

Use when:

```
Kubernetes

Auto Scaling

Cloud

Frequently Changing Targets
```

For EKS, dynamic discovery is generally the preferred approach.

---

# 125. Architecture Decision: Local vs Remote Storage

## Local

Use when:

```
Short/Moderate Retention

Simple Architecture

Single Cluster
```

## Remote

Consider when:

```
Long-Term Retention

Multiple Clusters

Centralized Monitoring

Large Scale
```

---

# 126. Architecture Decision: Single vs HA Prometheus

## Single

Suitable for:

```
Development

Small Environments

Non-Critical Monitoring
```

## HA

Consider when:

```
Production Monitoring Is Critical

Monitoring Availability Is Important

Multiple Prometheus Replicas Are Required

Centralized Long-Term Metrics Architecture Exists
```

---

# 127. Architecture Decision: Recording Rules

Use recording rules when:

```
Queries Are Expensive

Dashboards Repeat the Same Query

Alerts Reuse Complex Expressions

High Query Load Exists
```

Avoid creating recording rules for every simple query without a reason.

---

# 128. Architecture Decision: Federation

Consider federation when:

```
Multiple Prometheus Servers

Hierarchical Monitoring

Selective Metric Aggregation

Global Overview
```

is required.

---

# 129. Architecture Decision: Remote Write

Consider remote write when:

```
Long-Term Storage

Centralized Metrics

Multi-Cluster Metrics

External Metrics Platform
```

is required.

---

# 130. Prometheus Architecture Checklist

## Metric Sources

```
[ ] Application metrics

[ ] Node Exporter

[ ] kube-state-metrics

[ ] Database exporters

[ ] Blackbox monitoring where required
```

---

## Discovery

```
[ ] Kubernetes discovery

[ ] Dynamic target discovery

[ ] Relabeling

[ ] Target filtering
```

---

## Scraping

```
[ ] Scrape interval

[ ] Scrape timeout

[ ] Target health

[ ] Scrape errors
```

---

## Storage

```
[ ] TSDB

[ ] Persistent volume

[ ] Retention

[ ] Disk capacity

[ ] WAL

[ ] Compaction
```

---

## Query

```
[ ] PromQL

[ ] Query optimization

[ ] Recording rules

[ ] Dashboard optimization
```

---

## Alerting

```
[ ] Alerting rules

[ ] Alertmanager

[ ] Routing

[ ] Grouping

[ ] Deduplication

[ ] Silencing

[ ] Inhibition
```

---

## Production

```
[ ] Resource requests

[ ] Resource limits

[ ] HA requirements

[ ] Remote storage requirements

[ ] Security

[ ] Monitoring Prometheus itself

[ ] Failure recovery
```

---

# 131. Interview Question: Explain Prometheus Architecture.

### Answer

Prometheus follows a pull-based architecture.

Prometheus discovers monitoring targets through static or dynamic service discovery and periodically scrapes their metrics endpoints.

The scraped metrics are stored in the Prometheus TSDB as labeled time series.

PromQL is used to query and analyze the stored data.

Prometheus can also evaluate recording and alerting rules.

For visualization, Grafana commonly queries Prometheus.

For alert notifications, Prometheus sends alerts to Alertmanager.

The architecture is:

```
Targets
   ↓
Service Discovery
   ↓
Prometheus
   ↓
TSDB
   ↓
PromQL
   ↓
Grafana
```

and:

```
Prometheus
   ↓
Alerting Rules
   ↓
Alertmanager
   ↓
Notifications
```

---

# 132. Interview Question: Explain Prometheus Data Flow.

### Answer

The data flow is:

```
Application / Exporter
      ↓
Metrics Endpoint
      ↓
Service Discovery
      ↓
Prometheus Scrape
      ↓
Metric Parsing
      ↓
Relabeling
      ↓
TSDB
      ↓
PromQL
      ↓
Grafana / API / Rules
```

This separates collection, storage, querying, and visualization.

---

# 133. Interview Question: How Does Prometheus Discover Kubernetes Targets?

### Answer

Prometheus integrates with the Kubernetes API for service discovery.

It can discover resources such as:

```
Pods

Services

Nodes
```

and other supported Kubernetes objects.

Prometheus then uses relabeling rules to determine which discovered targets should be scraped and how their labels should be assigned.

This is important because Kubernetes workloads are dynamic.

---

# 134. Interview Question: What Happens During a Prometheus Scrape?

### Answer

The general flow is:

```
1. Prometheus selects a target.

2. It sends an HTTP request to the metrics endpoint.

3. The target returns metrics.

4. Prometheus parses the response.

5. Relabeling/filtering is applied where configured.

6. Samples are written to the TSDB.

7. PromQL, recording rules, and alerting rules can use the stored data.
```

---

# 135. Interview Question: What Is Prometheus TSDB?

### Answer

TSDB stands for Time Series Database.

Prometheus stores metric samples in its local time-series database.

Each series is identified by:

```
Metric Name
+
Labels
```

Prometheus also uses mechanisms such as the WAL and block storage to manage recent and persisted data.

---

# 136. Interview Question: What Is the Prometheus WAL?

### Answer

WAL stands for Write-Ahead Log.

Prometheus uses the WAL to record recent incoming samples before they are fully persisted into regular TSDB blocks.

If Prometheus restarts unexpectedly, the WAL can be replayed to recover recent data.

---

# 137. Interview Question: What Is Prometheus Compaction?

### Answer

Prometheus stores data in TSDB blocks.

Over time, blocks are compacted into larger blocks.

Compaction helps manage:

```
Storage

Indexes

Query Efficiency
```

It is an internal part of Prometheus TSDB management.

---

# 138. Interview Question: Why Does Prometheus Memory Increase?

### Answer

Common reasons include:

```
High Cardinality

Large Number of Active Series

Many Targets

High Ingestion Rate

Expensive Queries

Recording Rules

Large Query Ranges
```

I would first investigate active series and label cardinality because poorly designed labels can cause rapid time-series growth.

---

# 139. Interview Question: How Would You Design Prometheus for EKS?

### Answer

I would deploy Prometheus inside the EKS monitoring architecture.

Metric sources would include:

```
Application Metrics

Node Exporter

kube-state-metrics
```

Prometheus would use Kubernetes service discovery to dynamically discover targets.

The architecture would be:

```
EKS
   ↓
Kubernetes Service Discovery
   ↓
Prometheus
   ↓
TSDB
   ↓
Grafana
```

For alerting:

```
Prometheus
   ↓
Alertmanager
```

For large production environments I would additionally evaluate:

```
HA Prometheus

Remote Write

Long-Term Storage

Centralized Metrics
```

---

# 140. Interview Question: How Would You Troubleshoot a Prometheus Target Showing `DOWN`?

### Answer

I would check:

```
1. Target address.

2. Port.

3. Metrics endpoint.

4. DNS.

5. Network connectivity.

6. Kubernetes Service.

7. Pod status.

8. NetworkPolicy.

9. Security controls.

10. Prometheus scrape configuration.

11. TLS or authentication.
```

I would also test the endpoint manually.

For example:

```
curl http://target:port/metrics
```

If the endpoint is unreachable, I would investigate the application or network.

If the endpoint works manually, I would inspect Prometheus configuration and discovery.

---

# 141. Interview Question: Prometheus Is Running but Metrics Are Missing. What Do You Check?

### Answer

I would separate the problem into:

```
Collection

Storage

Query
```

First:

```
Is the target UP?
```

Then:

```
Does /metrics contain the metric?
```

Then:

```
Is the metric being filtered?
```

Then:

```
Is the PromQL correct?
```

I would check:

```
Service Discovery

Relabeling

Metric Relabeling

Metric Name

Labels

Prometheus Logs
```

---

# 142. Interview Question: How Would You Handle Prometheus Storage Growth?

### Answer

I would first identify the reason for growth.

I would check:

```
Active Series

Cardinality

Samples/sec

Scrape Interval

Retention

New Metrics
```

Then I would:

```
Remove unnecessary metrics

Reduce unbounded labels

Adjust scrape intervals where appropriate

Review retention

Expand storage
```

For large environments, I would evaluate long-term remote storage.

---

# 143. Interview Question: How Would You Handle High-Cardinality Metrics?

### Answer

I would identify the labels creating excessive combinations.

For example:

```
user_id

request_id

session_id
```

I would replace them with bounded dimensions such as:

```
service

method

status

endpoint
```

I would also review whether the information belongs in:

```
Logs
```

or:

```
Traces
```

rather than Prometheus metrics.

---

# 144. Interview Question: How Would You Make Prometheus Highly Available?

### Answer

I would deploy multiple Prometheus replicas that can scrape the same targets.

For example:

```
Targets
   ↓
┌─────────────┐
↓             ↓
```

Prometheus A  Prometheus B

Then I would design the downstream architecture to handle duplicate samples and failover.

For larger environments, I would evaluate a long-term metrics/query layer that supports HA and deduplication.

---

# 145. Interview Question: Why Do We Need Alertmanager?

### Answer

Prometheus is responsible for evaluating alert conditions.

Alertmanager handles the notification workflow.

It provides:

```
Grouping

Deduplication

Routing

Silencing

Inhibition

Notification Delivery
```

Therefore:

```
Prometheus = Detect

Alertmanager = Manage and Notify
```

---

# 146. Interview Question: Why Use Recording Rules?

### Answer

Recording rules precompute frequently used or expensive PromQL expressions.

For example:

```
Complex Query
    ↓
Recording Rule
    ↓
Precomputed Metric
```

This improves:

```
Dashboard Performance

Query Performance

Alert Evaluation
```

and reduces repeated computation.

---

# 147. Interview Question: What Happens If Prometheus Crashes?

### Answer

If Prometheus crashes:

```
Local metric collection stops

Local rule evaluation stops

Local querying may become unavailable
```

If persistent storage is used, Prometheus can recover stored data after restart.

The WAL helps recover recent samples.

In a highly available architecture, another Prometheus replica may continue monitoring.

---

# 148. Interview Question: How Would You Monitor Prometheus Itself?

### Answer

I would monitor:

```
CPU

Memory

Disk

Active Series

Samples Ingested

Scrape Failures

Query Latency

Rule Evaluation

TSDB Health

WAL
```

I would create alerts for:

```
High Memory

Disk Almost Full

Scrape Failures

Prometheus Down

High Query Latency
```

---

# 149. Interview Question: How Does Prometheus Fit Into an Observability Platform?

### Answer

Prometheus is the metrics component.

In a complete platform:

```
Metrics
   ↓
Prometheus

Logs
   ↓
ELK

Traces
   ↓
OpenTelemetry + Jaeger

Visualization
   ↓
Grafana
```

Prometheus can detect:

```
High Error Rate

High Latency

Resource Saturation
```

Then logs and traces can be used for deeper investigation.

---

# 150. Final Prometheus Architecture Mental Model

Remember the complete architecture as:

```
┌────────────────────────────────────────────────────────────┐
│                         TARGETS                            │
│                                                            │
│ Applications | Nodes | Kubernetes | Databases | Exporters  │
└────────────────────────────┬───────────────────────────────┘
                             │
                             ↓
                   SERVICE DISCOVERY
                             │
                             ↓
                      TARGET SELECTION
                             │
                             ↓
                          SCRAPE
                             │
                             ↓
                     RELABEL / FILTER
                             │
                             ↓
                          TSDB
                             │
                ┌────────────┴────────────┐
                ↓                         ↓
             PromQL                    Rules
                │                    ┌────┴────┐
                ↓                    ↓         ↓
             Grafana            Recording   Alerting
                                  Rules      Rules
                                               │
                                               ↓
                                          Alertmanager
                                               │
                                               ↓
                                         Notifications
```

For large production environments:

```
Prometheus
    │
    ├──── Local TSDB
    │
    └──── Remote Write
              ↓
       Long-Term Storage
              ↓
        Query Layer
              ↓
           Grafana
```

The most important architecture principles are:

```
Prometheus primarily uses a pull model.

Targets expose metrics.

Service discovery dynamically finds targets.

Scraping collects metrics.

Relabeling controls target and metric metadata.

TSDB stores time-series data.

WAL protects recent ingested data.

Blocks organize persisted data.

Compaction manages TSDB blocks.

PromQL queries the data.

Recording rules precompute expensive expressions.

Alerting rules detect operational conditions.

Alertmanager manages alert notifications.

Grafana visualizes Prometheus data.

Remote storage can provide long-term retention and centralized monitoring.

HA Prometheus deployments improve monitoring availability.

Cardinality must be controlled.

Prometheus itself must be monitored.
```

The complete production mental model is:

```
DISCOVER
    ↓
SCRAPE
    ↓
STORE
    ↓
QUERY
    ↓
VISUALIZE
    ↓
ALERT
    ↓
INVESTIGATE
    ↓
RESOLVE
```

And in a complete observability platform:

```
PROMETHEUS
    ↓
  METRICS
    ↓
DETECTION
    ↓
  ELK
    ↓
   LOGS
    ↓
OPENTELEMETRY
    ↓
  TRACES
    ↓
  JAEGER
    ↓
ROOT CAUSE
    ↓
 RESOLUTION
```
