# ELK Stack Fundamentals

## 1. Introduction

The ELK Stack is a centralized logging and log analysis platform built around three major components:

```text
E = Elasticsearch
L = Logstash
K = Kibana
```

The basic flow is:

```text
Application / Server
        ↓
      Logstash
        ↓
   Elasticsearch
        ↓
      Kibana
        ↓
   Engineers / Teams
```

The purpose of ELK is to collect logs from different systems, process and enrich them, store them centrally, and provide a searchable visualization interface.

---

# 2. Why Centralized Logging?

Without centralized logging, each server or container may have its own logs.

For example:

```text
EC2-1
 └── application.log

EC2-2
 └── application.log

EC2-3
 └── application.log
```

If an application is running across 20 containers, an engineer would have to inspect logs from multiple locations.

This becomes difficult during production incidents.

With centralized logging:

```text
Application 1 ─┐
Application 2 ─┤
Application 3 ─┤
Application 4 ─┤
                ↓
             Logstash
                ↓
         Elasticsearch
                ↓
              Kibana
```

All logs become searchable from one location.

---

# 3. Problems ELK Solves

ELK helps solve problems such as:

```text
Finding application errors
Searching logs across servers
Debugging production incidents
Correlating events
Filtering logs
Analyzing traffic
Investigating failures
Building log dashboards
Searching historical logs
```

Example:

```text
User reports:
"Payment failed"
```

Engineer can search:

```text
service:payment
status:500
```

and investigate the corresponding events.

---

# 4. ELK Components

The three primary components are:

```text
Elasticsearch
Logstash
Kibana
```

Each has a different responsibility.

```text
Logstash
   ↓
Collect + Process

Elasticsearch
   ↓
Store + Search

Kibana
   ↓
Visualize + Analyze
```

---

# 5. Elasticsearch

Elasticsearch is a distributed search and analytics engine.

It stores and indexes log data so that it can be searched efficiently.

Architecture:

```text
Logs
 ↓
Elasticsearch
 ↓
Index
 ↓
Search
```

Example search:

```text
service = payment
AND level = ERROR
```

Elasticsearch returns matching documents.

---

# 6. Logstash

Logstash is a data processing pipeline.

Its responsibility is commonly:

```text
Input
  ↓
Filter
  ↓
Output
```

Example:

```text
Application Logs
       ↓
     Input
       ↓
     Filter
       ↓
    Parse JSON
       ↓
    Add Fields
       ↓
    Elasticsearch
```

---

# 7. Kibana

Kibana is the visualization and exploration interface for Elasticsearch data.

It allows engineers to:

```text
Search logs
Create dashboards
Create visualizations
Investigate incidents
Filter events
Analyze trends
```

Architecture:

```text
Elasticsearch
      ↓
    Kibana
      ↓
  Engineers
```

---

# 8. Complete ELK Flow

The traditional architecture is:

```text
                 Applications
                      │
                      ↓
                  Log Files
                      │
                      ↓
                   Logstash
                      │
                Parse / Filter
                      │
                      ↓
                Elasticsearch
                      │
                 Index / Store
                      │
                      ↓
                    Kibana
                      │
                      ↓
                Engineers
```

---

# 9. Real-World Example

Suppose you have a microservices application:

```text
User Service
Product Service
Cart Service
Order Service
Payment Service
Inventory Service
Notification Service
```

Each service generates logs.

Without centralized logging:

```text
User Pod
 └── logs

Payment Pod
 └── logs

Order Pod
 └── logs
```

With ELK:

```text
User Logs ──────┐
Payment Logs ───┤
Order Logs ─────┤
Inventory Logs ─┤
                ↓
             Logstash
                ↓
         Elasticsearch
                ↓
              Kibana
```

---

# 10. Example Application Log

A simple application log might look like:

```text
2026-08-11 10:15:22 ERROR Payment failed for order 12345
```

This is readable but not ideal for large-scale log analysis.

Structured logging is better:

```json
{
  "timestamp": "2026-08-11T10:15:22Z",
  "level": "ERROR",
  "service": "payment",
  "environment": "production",
  "order_id": "12345",
  "message": "Payment failed"
}
```

Elasticsearch can index these fields separately.

---

# 11. Why Structured Logs Matter

With structured logs, you can search:

```text
service = payment
```

or:

```text
level = ERROR
```

or:

```text
order_id = 12345
```

instead of searching only through raw text.

This makes troubleshooting much faster.

---

# 12. Log Pipeline

A production pipeline may look like:

```text
Application
     ↓
JSON Logs
     ↓
Log Collector
     ↓
Logstash
     ↓
Parsing
     ↓
Enrichment
     ↓
Elasticsearch
     ↓
Kibana
```

The collector can be a separate agent depending on the architecture.

---

# 13. ELK vs Traditional Server Logs

Traditional:

```text
Server
 └── /var/log/application.log
```

ELK:

```text
Server
 └── Application
       ↓
      Logs
       ↓
    Central Pipeline
       ↓
 Elasticsearch
       ↓
     Kibana
```

Centralized logging provides a single place for investigation.

---

# 14. Log Collection

Logstash can receive logs through different inputs.

Examples include:

```text
File
Beats
TCP
UDP
HTTP
Kafka
Other supported inputs
```

A production architecture may use a lightweight log collector on each node and send logs to Logstash.

For Kubernetes, a common architecture is:

```text
Pod
 ↓
Container stdout/stderr
 ↓
Node log files
 ↓
Log Collector
 ↓
Logstash
```

---

# 15. Logstash Input

The input defines where Logstash receives data.

Example:

```text
Application
    ↓
HTTP
    ↓
Logstash Input
```

or:

```text
File
 ↓
Logstash
```

---

# 16. Logstash Filter

Filters transform incoming events.

Examples:

```text
Parse JSON
Grok
Mutate
Date parsing
Field extraction
Drop unwanted events
```

Architecture:

```text
Raw Log
   ↓
Filter
   ↓
Structured Event
```

---

# 17. Logstash Output

The output defines where processed events go.

Example:

```text
Logstash
   ↓
Elasticsearch
```

Other outputs may also be supported.

For ELK:

```text
Input
 ↓
Filter
 ↓
Elasticsearch
```

is the core pipeline.

---

# 18. Elasticsearch Index

Elasticsearch organizes documents into indexes.

Conceptually:

```text
payment-logs
order-logs
inventory-logs
```

Each index contains documents.

Example:

```text
payment-logs
 ├── document 1
 ├── document 2
 ├── document 3
 └── document 4
```

---

# 19. Elasticsearch Document

A log event is represented as a document.

Example:

```json
{
  "@timestamp": "2026-08-11T10:15:22Z",
  "service": "payment",
  "level": "ERROR",
  "message": "Payment failed",
  "environment": "production"
}
```

Elasticsearch indexes these fields.

---

# 20. Indexing

Indexing makes data searchable.

Flow:

```text
Log Event
   ↓
Document
   ↓
Elasticsearch
   ↓
Index
   ↓
Search
```

Instead of reading every log file manually, Elasticsearch can search indexed fields efficiently.

---

# 21. Elasticsearch Search

Example query concept:

```text
service:payment AND level:ERROR
```

Result:

```text
Payment service errors
```

Another example:

```text
environment:production
```

returns production logs.

---

# 22. Kibana Discover

Kibana's Discover interface is commonly used for log investigation.

Typical workflow:

```text
Kibana
 ↓
Discover
 ↓
Select Data View
 ↓
Choose Time Range
 ↓
Search
 ↓
Inspect Events
```

---

# 23. Kibana Dashboards

Kibana can provide dashboards such as:

```text
Production Logs
Application Errors
HTTP Status Codes
Traffic
Authentication Events
Payment Errors
Infrastructure Events
```

Example:

```text
Production Dashboard

Requests        50K
Errors           1.2K
5xx              300
4xx              900
Critical Logs     25
```

---

# 24. Kibana Visualization

Kibana can visualize indexed data.

Examples:

```text
Line charts
Bar charts
Pie charts
Tables
Metric panels
Maps
Other supported visualizations
```

The exact visualization capabilities depend on the Kibana version.

---

# 25. Log Levels

Common log levels include:

```text
TRACE
DEBUG
INFO
WARN
ERROR
FATAL
```

Example:

```text
INFO
Payment request received

WARN
Payment retry required

ERROR
Payment failed

FATAL
Application cannot start
```

---

# 26. Log Levels in Production

A common production configuration is:

```text
INFO
WARN
ERROR
```

Avoid excessive DEBUG logging in high-volume production environments unless temporarily enabled for troubleshooting.

---

# 27. Logging Architecture

A simple production architecture:

```text
                    EKS
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     Pod A         Pod B         Pod C
        │            │            │
        └────────────┼────────────┘
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

# 28. Kubernetes Logging

Containers commonly write application logs to:

```text
stdout
stderr
```

Kubernetes/container runtime infrastructure stores those logs on the node.

A log collector can then collect them.

Typical flow:

```text
Container
   ↓
stdout/stderr
   ↓
Node log files
   ↓
DaemonSet log collector
   ↓
Logstash
   ↓
Elasticsearch
   ↓
Kibana
```

---

# 29. Log Collector

In Kubernetes, it is common to use a lightweight agent such as:

```text
Filebeat
Fluent Bit
Fluentd
```

The collector's job is commonly:

```text
Collect
Add Kubernetes metadata
Forward
```

This reduces the need to run Logstash directly on every application node.

---

# 30. Why Use a Log Collector?

Logstash is powerful but relatively heavier than lightweight collection agents.

A common architecture is:

```text
Kubernetes Node
      ↓
Filebeat / Fluent Bit
      ↓
Logstash
      ↓
Elasticsearch
```

The collector handles local log collection.

Logstash handles more complex processing.

---

# 31. Filebeat Example Architecture

```text
             Kubernetes Node
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
      Pod A       Pod B       Pod C
        │           │           │
        └───────────┼───────────┘
                    ↓
                 Filebeat
                    ↓
                 Logstash
                    ↓
              Elasticsearch
                    ↓
                  Kibana
```

---

# 32. Fluent Bit Example

Another architecture:

```text
Pods
 ↓
Fluent Bit
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

Fluent Bit is commonly used as a lightweight Kubernetes log collector.

---

# 33. Logstash vs Log Collector

Log collector:

```text
Collect
Forward
Basic enrichment
```

Logstash:

```text
Parse
Transform
Enrich
Route
Process
```

This separation can improve scalability.

---

# 34. Logstash Processing Example

Raw log:

```text
2026-08-11 10:15:22 ERROR Payment failed
```

Logstash can transform it into:

```json
{
  "@timestamp": "2026-08-11T10:15:22Z",
  "level": "ERROR",
  "service": "payment",
  "message": "Payment failed"
}
```

---

# 35. Grok

Grok can extract fields from unstructured logs.

Example:

```text
2026-08-11 ERROR payment failed
```

can become:

```text
timestamp = 2026-08-11
level = ERROR
service = payment
message = failed
```

Grok is useful when applications produce text logs rather than JSON.

---

# 36. JSON Parsing

If the application already produces JSON:

```json
{
  "level": "ERROR",
  "service": "payment",
  "message": "Payment failed"
}
```

Logstash can parse the JSON into structured fields.

This is usually preferable to complicated regex parsing.

---

# 37. Enrichment

Logstash can add metadata such as:

```text
environment
team
application
region
cluster
```

Example:

```json
{
  "service": "payment",
  "environment": "production",
  "cluster": "eks-prod",
  "region": "ap-south-1"
}
```

This makes filtering and routing easier.

---

# 38. Kubernetes Metadata

A production log pipeline should ideally include:

```text
namespace
pod
container
node
deployment
service
cluster
```

Example:

```json
{
  "namespace": "production",
  "pod": "payment-7d9c8",
  "container": "payment",
  "node": "ip-10-0-1-20"
}
```

This dramatically improves troubleshooting.

---

# 39. Searching Kubernetes Logs

Instead of:

```text
grep payment.log
```

you can search:

```text
namespace:production
AND
service:payment
AND
level:ERROR
```

This is much faster during incidents.

---

# 40. Correlation Fields

Useful fields include:

```text
trace.id
transaction.id
request.id
order.id
user.id
```

For example:

```json
{
  "service": "payment",
  "trace.id": "abc123",
  "order.id": "ORD-1001",
  "message": "Payment failed"
}
```

These fields allow logs to be correlated with traces and application requests.

---

# 41. Logs + Traces

A mature observability architecture connects:

```text
Metrics
   ↓
Logs
   ↓
Traces
```

Example:

```text
Alert:
Payment error rate increased
        ↓
Kibana
        ↓
Search trace.id
        ↓
Jaeger
        ↓
Distributed trace
```

This is particularly useful for microservices.

---

# 42. Logs + Metrics

Example:

```text
Prometheus:
Payment 5xx rate ↑
        ↓
Grafana
        ↓
Kibana
        ↓
Payment ERROR logs
```

Metrics identify the problem.

Logs provide detailed event information.

---

# 43. ELK and Prometheus/Grafana

The roles are different:

```text
Prometheus
    ↓
Metrics

ELK
    ↓
Logs

Grafana
    ↓
Metrics / Observability UI

Kibana
    ↓
Log Search / Analysis
```

They complement each other.

---

# 44. ELK and Jaeger

The roles are:

```text
Elasticsearch
    ↓
Logs

Jaeger
    ↓
Traces

Prometheus
    ↓
Metrics
```

Together:

```text
Metrics + Logs + Traces
```

provide a complete observability platform.

---

# 45. Three Pillars

```text
             Observability
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
    Metrics      Logs      Traces
       ↓          ↓          ↓
 Prometheus       ELK      Jaeger
       │          │          │
       ↓          ↓          ↓
    Grafana     Kibana     Jaeger UI
```

---

# 46. Production Microservices Example

Suppose:

```text
Frontend
   ↓
ALB
   ↓
Orders
   ↓
Payment
   ↓
Inventory
```

A request fails.

Metrics show:

```text
Payment 5xx ↑
```

Logs show:

```text
Database connection timeout
```

Traces show:

```text
Orders
  ↓
Payment
  ↓
Database
      ↑
   3 seconds
```

Together, the root cause becomes easier to identify.

---

# 47. Elasticsearch Cluster

For production, Elasticsearch should generally run as a cluster rather than a single node.

Example:

```text
Elasticsearch Cluster

Node 1
Node 2
Node 3
```

Architecture:

```text
              Elasticsearch Cluster
           ┌────────┼────────┐
           ↓        ↓        ↓
        Node A    Node B    Node C
```

---

# 48. Elasticsearch Distributed Architecture

Elasticsearch distributes data across:

```text
Indices
Shards
Replicas
Nodes
```

This provides scalability and resilience.

---

# 49. Shards

An Elasticsearch index can be divided into shards.

Conceptually:

```text
Index
 ├── Primary Shard 1
 ├── Primary Shard 2
 └── Primary Shard 3
```

Different shards can be placed on different nodes.

---

# 50. Replicas

A replica is a copy of a primary shard.

Example:

```text
Primary Shard
      ↓
Replica Shard
```

If a node fails, replicas can help maintain availability.

---

# 51. Elasticsearch Cluster Example

```text
             Cluster
                │
      ┌─────────┼─────────┐
      ↓         ↓         ↓
    Node A    Node B    Node C
      │         │         │
   Primary    Replica   Primary
```

The actual shard placement is controlled by Elasticsearch.

---

# 52. Elasticsearch Index Lifecycle

Logs grow continuously.

Therefore:

```text
Logs
 ↓
Index
 ↓
Retention
 ↓
Delete / Archive
```

You should define:

```text
Retention period
Index lifecycle
Storage tiers
Deletion policy
```

---

# 53. Why Retention Matters

Suppose logs arrive at:

```text
100 GB/day
```

After 30 days:

```text
≈ 3 TB
```

Before replication and overhead.

Therefore centralized logging requires capacity planning.

---

# 54. Log Retention Strategy

Example:

```text
Hot:
7 days

Warm:
30 days

Cold:
90 days

Delete:
After retention period
```

The exact policy depends on:

```text
Compliance
Cost
Business requirements
Incident investigation needs
```

---

# 55. Hot/Warm Architecture

Recent logs:

```text
Hot
 ↓
Fast storage
```

Older logs:

```text
Warm
 ↓
Lower-cost storage
```

Very old logs:

```text
Cold / Archive
 ↓
Low-cost storage
```

This can reduce storage costs.

---

# 56. Index Lifecycle Management

Elasticsearch supports lifecycle management features that can automate:

```text
Rollover
Retention
Tier movement
Deletion
```

The exact implementation depends on the Elasticsearch version and license/features in use.

---

# 57. Index Naming

Use consistent names.

Example:

```text
logs-payment-production
logs-orders-production
logs-inventory-production
```

Or date-oriented patterns:

```text
logs-payment-2026.08.11
```

Choose a strategy that works with your lifecycle and operational model.

---

# 58. Avoid Uncontrolled Index Creation

Bad:

```text
logs-<random-id>
```

This can create thousands of indexes.

Better:

```text
logs-payment-production
```

with controlled rollover and lifecycle policies.

---

# 59. Index Templates

Index templates define consistent mappings and settings for new indexes.

Conceptually:

```text
Template
   ↓
New Index
   ↓
Consistent Fields / Mappings
```

This helps prevent inconsistent field types.

---

# 60. Mapping

Mappings define how fields are stored and indexed.

Example:

```text
timestamp → date
service   → keyword
level     → keyword
message   → text
```

Correct mappings improve search and storage behavior.

---

# 61. Text vs Keyword

A field such as:

```text
service = payment
```

is usually useful for exact filtering.

A field such as:

```text
message = Payment failed because database timed out
```

may need full-text search capabilities.

Understanding field types is important for Elasticsearch performance and query behavior.

---

# 62. Elasticsearch Query

A simple query concept:

```json
{
  "query": {
    "match": {
      "message": "payment failed"
    }
  }
}
```

The exact query depends on the field mapping and Elasticsearch version.

---

# 63. Filtering

Example concept:

```text
service = payment
environment = production
level = ERROR
```

Filtering structured fields is usually more efficient and reliable than searching arbitrary raw text.

---

# 64. Kibana Data View

Kibana needs a way to identify the Elasticsearch data it should display.

A data view can match indexes such as:

```text
logs-*
```

Then Kibana can search matching data.

---

# 65. Kibana Discover Workflow

```text
Kibana
 ↓
Discover
 ↓
Data View
 ↓
Time Range
 ↓
Filter
 ↓
Log Event
 ↓
Inspect Fields
```

---

# 66. Kibana Query Example

Search:

```text
service : "payment"
```

Then:

```text
level : "ERROR"
```

Then:

```text
environment : "production"
```

Combined:

```text
service:"payment"
AND level:"ERROR"
AND environment:"production"
```

The exact query syntax depends on the Kibana query language/configuration.

---

# 67. Production Incident Investigation

Example:

```text
Alert:
Payment error rate > 5%
```

Engineer opens Kibana:

```text
service:payment
level:ERROR
```

Then discovers:

```text
Database timeout
```

Next:

```text
trace.id = abc123
```

Engineer opens Jaeger.

This gives:

```text
Metrics → Logs → Traces
```

---

# 68. ELK Security

Production Elasticsearch should not be exposed publicly without strong controls.

Avoid:

```text
Internet
   ↓
Elasticsearch
```

Prefer:

```text
Private Network
   ↓
Elasticsearch
   ↑
Logstash
   ↑
Kibana
```

---

# 69. Elasticsearch Network Architecture

```text
                    Private VPC
                        │
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
    Logstash        Elasticsearch      Kibana
                        │
                    Cluster
```

Users access Kibana rather than directly accessing Elasticsearch.

---

# 70. Authentication

Production ELK environments should use appropriate authentication and authorization.

Depending on deployment and version:

```text
Users
 ↓
Kibana
 ↓
Authentication
 ↓
Elasticsearch
```

Centralized identity integration can be used where supported.

---

# 71. Authorization

Not every user should have access to all logs.

Example:

```text
Platform Team
 ↓
Infrastructure logs

Payments Team
 ↓
Payment logs

Security Team
 ↓
Security logs
```

Apply least privilege.

---

# 72. Sensitive Data

Logs can accidentally contain:

```text
Passwords
Tokens
API keys
Credit card information
Personal data
Session IDs
```

Do not log secrets.

If sensitive data is accidentally logged:

```text
Collect
 ↓
Detect
 ↓
Redact / Drop
 ↓
Store
```

where practical.

---

# 73. Log Redaction

Logstash can potentially remove or transform sensitive fields.

Example:

```text
password
token
authorization
```

should not be forwarded in plaintext.

A better solution is to prevent sensitive data from being logged at the application layer.

---

# 74. Log Volume

High log volume can cause:

```text
High storage usage
High network traffic
Elasticsearch pressure
Logstash CPU usage
Kibana slow queries
```

Therefore:

```text
Log volume
+
Retention
+
Cardinality
+
Query load
```

must be planned together.

---

# 75. Logging Strategy

Do not log everything at maximum verbosity.

A practical approach:

```text
INFO
Important application events

WARN
Unexpected conditions

ERROR
Failures

DEBUG
Temporary troubleshooting
```

---

# 76. Logging in Production

Good:

```json
{
  "timestamp": "...",
  "service": "payment",
  "level": "ERROR",
  "environment": "production",
  "message": "Database connection failed"
}
```

Bad:

```text
password=secret123
token=abc123
card_number=...
```

---

# 77. Log Sampling

For extremely high-volume systems, sampling can reduce volume.

Example:

```text
100,000 requests
      ↓
Log every request
```

may produce too much data.

Instead:

```text
Normal traffic
 ↓
Sample logs

Errors
 ↓
Keep all relevant logs
```

The exact strategy depends on the application's requirements.

---

# 78. Error Logs

Do not aggressively sample critical error events without understanding the impact.

A common strategy is:

```text
Normal requests
 ↓
Sample

Errors
 ↓
Keep
```

---

# 79. Logstash Scalability

One Logstash instance can become a bottleneck.

Instead:

```text
Collectors
    ↓
┌───────────────┐
│ Logstash A    │
│ Logstash B    │
│ Logstash C    │
└───────────────┘
    ↓
Elasticsearch
```

Load can be distributed across multiple Logstash instances.

---

# 80. Logstash Queue

Logstash can use buffering mechanisms to handle temporary downstream failures.

Conceptually:

```text
Collector
   ↓
Logstash
   ↓
Queue
   ↓
Elasticsearch
```

If Elasticsearch is temporarily unavailable, buffering can reduce immediate data loss depending on the configured architecture.

---

# 81. Persistent Queue

For production, Logstash persistent queues can provide additional durability for events awaiting processing.

Architecture:

```text
Input
 ↓
Logstash
 ↓
Persistent Queue
 ↓
Elasticsearch
```

This is not a replacement for proper Elasticsearch durability.

---

# 82. Elasticsearch Backpressure

If Elasticsearch becomes slow:

```text
Logstash
   ↓
Queue grows
   ↓
Elasticsearch pressure
```

Monitor:

```text
Queue size
Processing latency
Elasticsearch health
Disk
CPU
Memory
```

---

# 83. Elasticsearch Health

Elasticsearch cluster health has states such as:

```text
Green
Yellow
Red
```

Conceptually:

```text
Green
 ↓
Healthy

Yellow
 ↓
Replica allocation issue

Red
 ↓
Primary shard issue
```

Investigate cluster health immediately when it becomes degraded.

---

# 84. Elasticsearch Disk Usage

Disk pressure is critical.

Monitor:

```text
Disk utilization
Available storage
Shard allocation
Index growth
```

If disks become full:

```text
Elasticsearch
      ↓
Indexing problems
      ↓
Log ingestion failure
```

---

# 85. Elasticsearch JVM Memory

Elasticsearch uses the JVM.

Monitor:

```text
Heap usage
GC activity
CPU
Memory pressure
```

Incorrect heap sizing can cause:

```text
Slow queries
Long garbage collection
Out-of-memory failures
```

---

# 86. Elasticsearch Capacity Planning

Consider:

```text
Daily log volume
Retention
Replication factor
Shard count
Query volume
Growth rate
Node resources
```

Example:

```text
200 GB/day
× 30 days
= 6 TB raw
```

Then account for:

```text
Replica copies
Index overhead
Segment overhead
Operational headroom
```

---

# 87. Shard Strategy

Too few shards:

```text
Large shards
 ↓
Limited parallelism
```

Too many shards:

```text
Many small shards
 ↓
Cluster overhead
```

Choose shard counts based on actual data volume and workload.

---

# 88. Avoid Oversharding

A common Elasticsearch operational problem is creating too many small shards.

This increases:

```text
Memory usage
Cluster state
Management overhead
Query overhead
```

Use lifecycle and rollover strategies carefully.

---

# 89. Elasticsearch Nodes

Production clusters can separate responsibilities depending on architecture.

Examples:

```text
Master-eligible nodes
Data nodes
Ingest nodes
Coordinating nodes
```

The exact node-role architecture depends on cluster size and workload.

---

# 90. Small Production Cluster

A smaller environment may use:

```text
Node A
Node B
Node C
```

with appropriate roles.

This can provide basic resilience without unnecessarily complicated topology.

---

# 91. Large Production Cluster

A larger environment may separate roles:

```text
Dedicated Master Nodes
        ↓
Data Nodes
        ↓
Coordinating Nodes
        ↓
Kibana / Clients
```

The architecture should be driven by workload.

---

# 92. Kibana Architecture

Kibana is the UI layer:

```text
User
 ↓
Kibana
 ↓
Elasticsearch
```

For high availability:

```text
            ALB
             │
      ┌──────┴──────┐
      ↓             ↓
  Kibana A      Kibana B
      │             │
      └──────┬──────┘
             ↓
       Elasticsearch
```

---

# 93. Kibana Production Deployment

Kibana can run:

```text
EC2
VM
Kubernetes
Managed service
```

For EKS:

```text
Kibana Deployment
 ↓
Kibana Service
 ↓
ALB / Ingress
```

---

# 94. Kibana High Availability

Multiple replicas:

```text
Kibana A
Kibana B
```

behind:

```text
ALB
```

can improve availability.

However, Elasticsearch remains a critical backend dependency.

---

# 95. ELK on Kubernetes

A complete Kubernetes architecture:

```text
                         ALB
                          │
                ┌─────────┴─────────┐
                ↓                   ↓
             Kibana A           Kibana B
                │                   │
                └─────────┬─────────┘
                          ↓
                  Elasticsearch
                 ┌────────┼────────┐
                 ↓        ↓        ↓
              ES Node A ES Node B ES Node C
                 ↑        ↑        ↑
                 └────────┼────────┘
                          ↑
                      Logstash
                          ↑
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Collector    Collector    Collector
             ↓            ↓            ↓
            Pods         Pods         Pods
```

---

# 96. Centralized Logging Architecture

The full flow:

```text
Application
    ↓
stdout/stderr
    ↓
Node
    ↓
Log Collector
    ↓
Logstash
    ↓
Elasticsearch
    ↓
Kibana
    ↓
Engineer
```

This is the core centralized logging model.

---

# 97. Centralized Logging Benefits

```text
Single search interface
Central retention
Cross-service investigation
Historical analysis
Structured search
Incident troubleshooting
Security investigation
Audit support
```

---

# 98. Centralized Logging Challenges

ELK also introduces operational challenges:

```text
Storage cost
Network bandwidth
Cluster management
Shard management
Retention
Log volume
Security
Backup
Scaling
Query performance
```

A production logging platform must manage these deliberately.

---

# 99. Logging Pipeline Failure

Suppose:

```text
Application
 ↓
Collector
 ↓
Logstash
 X
```

Logs may accumulate locally depending on the collector and runtime.

Therefore:

```text
Collector buffering
+
Logstash queue
+
Elasticsearch durability
```

should be considered together.

---

# 100. Elasticsearch Failure

If Elasticsearch is unavailable:

```text
Logstash
 ↓
Queue
 ↓
Elasticsearch X
```

The system should have enough buffering and recovery capability to handle expected outages.

Do not assume indefinite buffering.

---

# 101. Logstash Failure

If one Logstash instance fails:

```text
Collector
 ↓
Logstash A X
```

A resilient architecture can route to:

```text
Logstash B
```

Example:

```text
Collectors
 ├──→ Logstash A
 └──→ Logstash B
```

---

# 102. Kibana Failure

If Kibana fails:

```text
Elasticsearch
   ✓
Kibana
   X
```

Log ingestion can potentially continue, but engineers lose the normal UI.

Run multiple Kibana replicas where required.

---

# 103. End-to-End Availability

A production ELK architecture should consider:

```text
Collector HA
Logstash HA
Elasticsearch HA
Kibana HA
Network HA
Storage HA
```

The exact design depends on business requirements.

---

# 104. Logging Security Architecture

```text
Applications
     ↓
Collectors
     ↓
Logstash
     ↓
Private Elasticsearch Cluster
     ↓
Kibana
     ↓
SSO / RBAC
     ↓
Authorized Users
```

Keep the storage layer protected.

---

# 105. Observability Security

Logs can contain sensitive operational information.

Therefore protect:

```text
Logs
Indexes
Dashboards
Kibana
Elasticsearch APIs
Credentials
```

Use:

```text
Authentication
Authorization
TLS
Network controls
Secret management
```

---

# 106. ELK and DevSecOps

Logging should also be integrated into the DevSecOps lifecycle.

Example:

```text
Developer
 ↓
GitHub
 ↓
GitHub Actions
 ↓
Deploy
 ↓
EKS
 ↓
Logs
 ↓
ELK
 ↓
Security / Operations
```

Logs can help investigate:

```text
Deployment failures
Authentication failures
Application errors
Infrastructure issues
Security events
```

---

# 107. Logging and Incident Response

During an incident:

```text
Alert
 ↓
Grafana
 ↓
Kibana
 ↓
Search logs
 ↓
Identify error
 ↓
Trace request
 ↓
Find root cause
 ↓
Remediate
```

This is the operational value of centralized logging.

---

# 108. Example Production Incident

Alert:

```text
Payment error rate > 5%
```

Grafana shows:

```text
Payment 5xx = 8%
```

Kibana search:

```text
service:payment
level:ERROR
```

Result:

```text
Database connection timeout
```

Trace:

```text
Payment
 ↓
Database
 ↓
Timeout
```

Root cause:

```text
Database connection exhaustion
```

This demonstrates the power of combining:

```text
Metrics
Logs
Traces
```

---

# 109. ELK Monitoring

Monitor the ELK stack itself.

### Logstash

```text
Events received
Events processed
Pipeline throughput
Queue size
Pipeline latency
Errors
```

### Elasticsearch

```text
Cluster health
CPU
Memory
Heap
Disk
Shard status
Indexing rate
Search latency
```

### Kibana

```text
Availability
CPU
Memory
HTTP errors
Response latency
```

---

# 110. Self-Monitoring Architecture

```text
                Prometheus
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
    Logstash    Elasticsearch  Kibana
        │           │           │
        └───────────┼───────────┘
                    ↓
                 Grafana
```

Prometheus can monitor the ELK components, while Kibana handles log exploration.

---

# 111. Production Architecture Summary

A mature architecture for your stack can look like:

```text
                             USERS
                               │
                               ↓
                            Route 53
                               │
                               ↓
                              ALB
                               │
                               ↓
                            Kibana
                               │
                               ↓
                     Elasticsearch Cluster
                       ┌───────┼───────┐
                       ↓       ↓       ↓
                      ES1     ES2     ES3
                       ↑       ↑       ↑
                       └───────┼───────┘
                               ↑
                           Logstash
                         ┌─────┴─────┐
                         ↓           ↓
                    Logstash A   Logstash B
                         ↑           ↑
                         └─────┬─────┘
                               ↑
                       Log Collectors
                         ┌─────┼─────┐
                         ↓     ↓     ↓
                       Node  Node  Node
```

---

# 112. ELK + Prometheus + Grafana + Jaeger

Your complete observability platform becomes:

```text
                         EKS
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
     Metrics             Logs             Traces
        ↓                 ↓                 ↓
   Prometheus          Logstash           Jaeger
        ↓                 ↓                 ↓
     Grafana        Elasticsearch       Jaeger UI
        │                 │
        └─────────────────┼─────────────────┐
                          ↓                 │
                       Kibana               │
                          │                 │
                          └─────────────────┘
```

A cleaner conceptual model:

```text
Metrics → Prometheus → Grafana
Logs    → ELK        → Kibana
Traces  → Jaeger     → Jaeger UI
```

---

# 113. ELK Production Principles

Remember these principles:

```text
1. Collect centrally.
2. Prefer structured logs.
3. Add Kubernetes metadata.
4. Separate collection from processing.
5. Keep Elasticsearch private.
6. Use HA for production.
7. Plan retention.
8. Control log volume.
9. Monitor storage.
10. Monitor cluster health.
11. Protect sensitive data.
12. Manage configuration as code.
13. Test failure recovery.
14. Integrate logs with metrics and traces.
```

---

# 114. Interview Answer: What Is ELK?

```text
"ELK stands for Elasticsearch, Logstash and Kibana.

Logstash is responsible for collecting and processing log events,
Elasticsearch stores and indexes the processed events, and Kibana
provides the UI for searching, analyzing and visualizing the logs.

In a Kubernetes environment, I would commonly use a lightweight
collector such as Fluent Bit or Filebeat to collect container logs
and forward them to Logstash, which then processes and sends them
to Elasticsearch."
```

---

# 115. Interview Answer: Explain the ELK Architecture

```text
"In a production Kubernetes environment, application containers
write logs to stdout and stderr.

A node-level log collector such as Fluent Bit or Filebeat collects
those logs and adds Kubernetes metadata.

The collector forwards the events to Logstash.

Logstash parses and enriches the events and sends them to an
Elasticsearch cluster.

Kibana connects to Elasticsearch and provides centralized log
search, analysis and dashboards.

The flow is:

Application → Collector → Logstash → Elasticsearch → Kibana."
```

---

# 116. Interview Answer: Why Use Logstash?

```text
"I use Logstash when I need more advanced log processing.

It can parse unstructured logs, transform fields, enrich events,
route events and send them to different outputs.

For example, if an application produces JSON logs, Logstash can
parse the JSON, add environment and service metadata, and send the
structured event to Elasticsearch."
```

---

# 117. Interview Answer: Why Use Filebeat or Fluent Bit With Logstash?

```text
"Logstash is more resource-intensive than lightweight log collectors.

In Kubernetes, I prefer a node-level collector such as Fluent Bit or
Filebeat for local log collection and forwarding.

Logstash can then focus on more complex parsing, transformation and
enrichment.

This separates collection from processing and makes the logging
pipeline easier to scale."
```

---

# 118. Interview Answer: How Do You Handle Large Log Volumes?

```text
"First I control the log volume at the application level by using
appropriate log levels and structured logging.

Then I scale the collection and Logstash layer horizontally.

On Elasticsearch, I plan shard allocation, retention and lifecycle
policies based on daily ingestion volume and growth.

I also monitor disk, JVM heap, indexing latency and search latency.

For long retention, I use appropriate storage tiers or archival
architecture."
```

---

# 119. Interview Answer: How Do You Secure ELK?

```text
"I keep Elasticsearch on private networking and do not expose it
directly to the internet.

Users access logs through Kibana, which is protected using
authentication and role-based authorization.

I use TLS where required, store credentials securely and ensure
sensitive data such as passwords and tokens is not written into
logs.

I also apply network controls and least-privilege access."
```

---

# 120. Interview Answer: Elasticsearch Cluster Is Red

```text
"I would first check cluster health and identify whether primary or
replica shards are affected.

Then I would check node availability, disk usage, JVM heap,
allocation failures and recent cluster changes.

If primary shards are unavailable, the issue is more serious because
the affected data may not be searchable or writable.

I would investigate shard allocation and node health before making
any destructive changes."
```

---

# 121. Interview Answer: Elasticsearch Disk Is Full

```text
"First I would stop the disk from continuing to grow if possible.

I would identify the largest indexes and review retention and
lifecycle policies.

I would check shard allocation and Elasticsearch disk watermarks.

Then I would remove or archive data according to the approved
retention policy and add capacity if required.

I would not randomly delete production indexes without checking
retention and recovery requirements."
```

---

# 122. Interview Answer: Logs Are Not Appearing in Kibana

Use this troubleshooting chain:

```text
Application
    ↓
Logs generated?
    ↓
Collector
    ↓
Collector receiving?
    ↓
Logstash
    ↓
Pipeline working?
    ↓
Elasticsearch
    ↓
Documents indexed?
    ↓
Kibana
    ↓
Correct data view?
```

Commands and checks should be performed at each layer.

---

# 123. Logs Missing From Elasticsearch

Check:

```text
Application logs
Collector logs
Logstash logs
Logstash pipeline
Elasticsearch cluster health
Index existence
Indexing errors
Network connectivity
Authentication
```

Do not start troubleshooting only from Kibana.

---

# 124. ELK Troubleshooting Method

Always trace the pipeline from left to right:

```text
1. Application
2. Collector
3. Logstash
4. Elasticsearch
5. Kibana
```

At each stage ask:

```text
Is data entering?
Is data leaving?
Are there errors?
```

This prevents random troubleshooting.

---

# 125. Final Mental Model

The most important ELK flow to remember is:

```text
                 COLLECT
                    ↓
               LOG COLLECTOR
                    ↓
                 PROCESS
                    ↓
                LOGSTASH
                    ↓
                  INDEX
                    ↓
             ELASTICSEARCH
                    ↓
                 SEARCH
                    ↓
                  KIBANA
                    ↓
                ANALYZE
```

For Kubernetes:

```text
Pod
 ↓
stdout/stderr
 ↓
Node log files
 ↓
Fluent Bit / Filebeat
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

And for the complete observability platform:

```text
               OBSERVABILITY
                    │
        ┌───────────┼───────────┐
        ↓           ↓           ↓
     Metrics       Logs       Traces
        ↓           ↓           ↓
   Prometheus      ELK        Jaeger
        ↓           ↓           ↓
     Grafana      Kibana     Jaeger UI
```

The key production principle is:

**Collect logs reliably, structure and enrich them consistently, store them with controlled retention, secure the logging platform, and make logs easy to correlate with metrics and traces during incidents.**
