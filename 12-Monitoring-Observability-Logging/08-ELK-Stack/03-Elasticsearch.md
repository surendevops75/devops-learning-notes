# Elasticsearch

## 1. Introduction

Elasticsearch is a distributed search and analytics engine used to store, index, search, and analyze large volumes of data.

In the ELK architecture:

```text
Logstash
   ↓
Elasticsearch
   ↓
Kibana
```

Logstash sends processed events to Elasticsearch.

Elasticsearch indexes and stores those events.

Kibana queries Elasticsearch and provides the visualization and investigation interface.

---

# 2. Elasticsearch in the ELK Stack

The responsibilities are:

```text
Logstash
   ↓
Collect
Process
Transform
Enrich

Elasticsearch
   ↓
Index
Store
Search
Aggregate

Kibana
   ↓
Visualize
Explore
Analyze
```

A typical logging flow is:

```text
Application
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

# 3. Why Elasticsearch?

Traditional log storage often involves searching individual files:

```text
/var/log/application.log
```

For a large environment:

```text
Server 1
Server 2
Server 3
Server 4
...
Server 100
```

Searching manually becomes difficult.

Elasticsearch provides:

```text
Centralized storage
Fast search
Structured indexing
Filtering
Aggregations
Distributed processing
Horizontal scalability
```

---

# 4. Elasticsearch Example

Suppose the Payment service produces:

```json
{
  "service": "payment",
  "level": "ERROR",
  "message": "Payment failed",
  "order_id": "ORD-1001"
}
```

Elasticsearch stores this event as a document.

You can then search:

```text
service:payment
```

or:

```text
service:payment AND level:ERROR
```

or:

```text
order_id:ORD-1001
```

---

# 5. Elasticsearch Mental Model

The most important concepts are:

```text
Cluster
   ↓
Node
   ↓
Index
   ↓
Document
   ↓
Field
```

Data is distributed using:

```text
Index
   ↓
Shards
   ↓
Nodes
```

---

# 6. Elasticsearch Cluster

A cluster is a collection of Elasticsearch nodes working together.

Example:

```text
             Elasticsearch Cluster
              /        |        \
             /         |         \
           Node 1    Node 2    Node 3
```

The cluster provides:

```text
Distributed storage
Distributed search
High availability
Scalability
```

---

# 7. Elasticsearch Node

A node is an Elasticsearch instance.

Example:

```text
Cluster
 ├── Node A
 ├── Node B
 └── Node C
```

Each node has its own:

```text
CPU
Memory
Disk
Network
Elasticsearch process
```

Nodes communicate with each other to form the cluster.

---

# 8. Cluster Name

Elasticsearch nodes use cluster configuration to determine which cluster they belong to.

Conceptually:

```text
cluster.name = production-logging
```

Nodes configured for the same cluster and compatible discovery configuration can form the cluster.

---

# 9. Node Name

Each Elasticsearch node can have a unique node name.

Example:

```text
node.name = es-prod-01
```

Another:

```text
node.name = es-prod-02
```

This helps identify nodes operationally.

---

# 10. Elasticsearch Data Model

Elasticsearch uses JSON documents.

Example:

```json
{
  "@timestamp": "2026-08-11T10:30:00Z",
  "service": "payment",
  "environment": "production",
  "level": "ERROR",
  "message": "Database connection failed"
}
```

The document contains fields.

---

# 11. Document

A document represents an individual data object.

For logging:

```text
One log event
    ↓
One Elasticsearch document
```

Example:

```json
{
  "service": "orders",
  "level": "INFO",
  "message": "Order created"
}
```

---

# 12. Field

Fields are individual pieces of information inside a document.

Example:

```json
{
  "service": "payment",
  "level": "ERROR",
  "status_code": 500
}
```

Fields:

```text
service
level
status_code
```

Each field can have a mapping that controls how Elasticsearch indexes and searches it.

---

# 13. Index

An index is a logical collection of documents.

For example:

```text
logs-payment-production
```

may contain:

```text
Document 1
Document 2
Document 3
Document 4
...
```

Conceptually:

```text
logs-payment-production
        │
   ┌────┼────┐
   ↓    ↓    ↓
 Doc1 Doc2 Doc3
```

---

# 14. Index Example

Suppose the Payment service produces thousands of logs.

You could have:

```text
logs-payment-production
```

containing:

```text
Payment request received
Payment successful
Payment failed
Database timeout
Payment retry
```

Each event becomes a document.

---

# 15. Index Naming

Use predictable naming conventions.

Examples:

```text
logs-payment-production
logs-orders-production
logs-inventory-production
```

Another approach is date-based:

```text
logs-payment-2026.08.11
logs-payment-2026.08.12
```

The appropriate strategy depends on the ingestion volume, lifecycle policy, and operational requirements.

---

# 16. Avoid Random Index Names

Avoid uncontrolled patterns such as:

```text
logs-abc123
logs-xyz456
logs-random789
```

This can result in many unnecessary indexes.

A controlled naming strategy makes:

```text
Search
Retention
Access control
Capacity planning
```

easier.

---

# 17. Shards

Elasticsearch distributes index data using shards.

An index can contain multiple primary shards.

Example:

```text
Index
 ├── Primary Shard 1
 ├── Primary Shard 2
 └── Primary Shard 3
```

These shards can be distributed across nodes.

---

# 18. Why Shards?

Shards allow Elasticsearch to distribute data and search work.

Example:

```text
               Index
                 │
       ┌─────────┼─────────┐
       ↓         ↓         ↓
    Shard 1   Shard 2   Shard 3
       │         │         │
       ↓         ↓         ↓
      ES1       ES2       ES3
```

A search can execute across multiple shards.

---

# 19. Primary Shard

A primary shard is the original shard that stores a portion of an index's data.

Example:

```text
Index
 ├── Primary 1
 ├── Primary 2
 └── Primary 3
```

Each primary shard contains a subset of the index documents.

---

# 20. Replica Shard

A replica is a copy of a primary shard.

Example:

```text
Primary Shard 1
      ↓
Replica Shard 1
```

Replicas provide additional copies and can improve availability and search capacity.

---

# 21. Primary + Replica Example

```text
             Index
               │
       ┌───────┴───────┐
       ↓               ↓
   Primary 1       Primary 2
       ↓               ↓
   Replica 1       Replica 2
```

These shards can be distributed across different nodes.

---

# 22. Node Distribution

Example:

```text
ES1
 ├── Primary Shard 1
 └── Replica Shard 2

ES2
 ├── Replica Shard 1
 └── Primary Shard 2
```

This protects against a single-node failure.

---

# 23. Shard Allocation

Elasticsearch decides where shards should be placed based on cluster configuration and allocation rules.

Conceptually:

```text
Index
 ↓
Primary + Replica
 ↓
Shard Allocation
 ↓
Multiple Nodes
```

Elasticsearch tries to avoid placing a replica on the same node as its primary.

---

# 24. Shard Count

Shard count is an important design decision.

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

Therefore shard count should be based on:

```text
Data volume
Query workload
Node resources
Growth
Retention
```

---

# 25. Oversharding

Oversharding means creating too many shards.

For example:

```text
100 indexes
×
20 shards
=
2000 primary shards
```

This can create unnecessary cluster overhead.

Avoid creating shards simply because more shards appear to provide better performance.

---

# 26. Elasticsearch Mappings

Mappings define how fields are indexed.

Example:

```text
timestamp → date
service   → keyword
level     → keyword
message   → text
status    → integer
```

Mappings are important for:

```text
Search behavior
Sorting
Aggregations
Storage
Performance
```

---

# 27. Text Field

A `text` field is generally used for full-text search.

Example:

```text
message:
"Payment failed because database connection timed out"
```

A text field can be analyzed so individual terms can be searched.

---

# 28. Keyword Field

A `keyword` field is generally used for exact values.

Example:

```text
service = payment
environment = production
level = ERROR
```

You commonly want exact filtering:

```text
service:"payment"
```

rather than full-text analysis.

---

# 29. Text vs Keyword

Conceptually:

```text
message
   ↓
text

service
   ↓
keyword

environment
   ↓
keyword

status_code
   ↓
integer
```

Correct mappings improve query behavior.

---

# 30. Date Fields

Timestamps should generally be mapped as dates.

Example:

```json
{
  "@timestamp": "2026-08-11T10:30:00Z"
}
```

This allows:

```text
Time filtering
Sorting
Time-based aggregations
```

Kibana relies heavily on timestamp fields for log exploration.

---

# 31. Numeric Fields

Numeric values should use appropriate numeric mappings.

Example:

```json
{
  "status_code": 500,
  "response_time_ms": 250
}
```

These can then be used for:

```text
Range queries
Sorting
Aggregations
```

---

# 32. Boolean Fields

Boolean fields represent:

```text
true
false
```

Example:

```json
{
  "payment_success": false
}
```

This can be filtered efficiently.

---

# 33. Dynamic Mapping

Elasticsearch can dynamically infer mappings for new fields depending on configuration.

For example:

```json
{
  "status_code": 500
}
```

may result in a numeric mapping.

Dynamic mapping is convenient but should be controlled carefully in production.

---

# 34. Mapping Explosion

If applications continuously generate new field names:

```text
field_1
field_2
field_3
...
field_100000
```

the number of mapped fields can grow excessively.

This can create:

```text
Memory pressure
Cluster instability
Management complexity
```

Structured logging should use stable field names.

---

# 35. Inverted Index

Elasticsearch uses indexing structures designed for fast search.

A simplified mental model is:

```text
Term
 ↓
Documents containing that term
```

Example:

```text
"payment"
   ↓
Doc 1
Doc 7
Doc 15
Doc 30
```

This allows searches without scanning every document like a simple text-file grep.

---

# 36. Search Architecture

A search request can look like:

```text
Kibana
   ↓
Elasticsearch
   ↓
Coordinating Node
   ↓
Multiple Shards
   ↓
Search Results
   ↓
Coordinating Node
   ↓
Kibana
```

The coordinating layer collects and combines responses.

---

# 37. Query Example

A simple Elasticsearch query:

```json
{
  "query": {
    "match": {
      "message": "payment failed"
    }
  }
}
```

This searches the `message` field.

The actual results depend on field mapping and analysis.

---

# 38. Term Query

For exact-value fields such as `keyword`, a term query can be used.

Example:

```json
{
  "query": {
    "term": {
      "service": "payment"
    }
  }
}
```

This is appropriate when `service` is mapped as a keyword field.

---

# 39. Boolean Query

You can combine conditions.

Conceptually:

```text
service = payment
AND
level = ERROR
AND
environment = production
```

Elasticsearch supports boolean query structures for this purpose.

---

# 40. Range Query

Numeric and date fields can be searched using ranges.

Example:

```text
response_time_ms > 1000
```

or:

```text
timestamp between
10:00 and 11:00
```

This is useful for finding slow requests or events in a particular time window.

---

# 41. Aggregations

Elasticsearch is not only for searching.

It can aggregate data.

Example:

```text
Count errors by service
```

Result:

```text
payment       500
orders        320
inventory     120
```

This is useful for dashboards.

---

# 42. Aggregation Example

Conceptually:

```text
Logs
 ↓
Group by service
 ↓
Count documents
 ↓
Results
```

Example:

```text
payment     500
orders      300
inventory   100
```

Kibana can visualize these results.

---

# 43. Average Aggregation

Suppose logs contain:

```text
response_time_ms
```

Elasticsearch can calculate:

```text
Average response time
```

Example:

```text
Average = 245 ms
```

---

# 44. Maximum Aggregation

You can calculate:

```text
Maximum response time
```

Example:

```text
Maximum = 4.2 seconds
```

This can help identify latency spikes.

---

# 45. Cardinality

Cardinality represents the number of unique values.

Example:

```text
Unique users
Unique orders
Unique IP addresses
```

High-cardinality fields should be designed carefully because aggregating them can consume significant resources.

---

# 46. Elasticsearch REST API

Elasticsearch provides HTTP APIs.

Examples of API operations include:

```text
GET
POST
PUT
DELETE
```

Example:

```text
GET /_cluster/health
```

This can be used to inspect cluster health.

---

# 47. Cluster Health

A cluster health response can indicate:

```text
green
yellow
red
```

Conceptually:

```text
Green
 ↓
Healthy

Yellow
 ↓
Primary shards available,
some replicas not allocated

Red
 ↓
One or more primary shards unavailable
```

Always investigate degraded cluster health.

---

# 48. Cluster Health Example

Conceptual request:

```text
GET /_cluster/health
```

Example response:

```json
{
  "cluster_name": "production-logging",
  "status": "green",
  "number_of_nodes": 3
}
```

The actual response contains many additional fields.

---

# 49. Node Information

Elasticsearch APIs can provide information about nodes.

Conceptually:

```text
GET /_cat/nodes
```

This can help identify:

```text
Node name
Roles
CPU
Heap
Memory
```

The CAT APIs are useful operational tools.

---

# 50. Index Information

You can inspect indexes using Elasticsearch APIs.

Conceptually:

```text
GET /_cat/indices
```

This can show information such as:

```text
Index
Health
Status
Document count
Storage size
```

---

# 51. Shard Information

You can inspect shard allocation.

Conceptually:

```text
GET /_cat/shards
```

This is useful when troubleshooting:

```text
Unassigned shards
Shard distribution
Primary/replica placement
```

---

# 52. Elasticsearch Disk

Disk usage is extremely important.

Monitor:

```text
Disk used
Disk available
Index size
Shard size
Growth rate
```

If disks approach configured watermarks, Elasticsearch can restrict shard allocation or indexing behavior.

---

# 53. Disk Watermarks

Elasticsearch uses disk allocation watermarks to manage shard placement and protect nodes from running out of storage.

Conceptually:

```text
Normal
 ↓
High disk usage
 ↓
High watermark
 ↓
Allocation restrictions
```

The exact thresholds depend on Elasticsearch configuration and version.

---

# 54. JVM Heap

Elasticsearch runs on the JVM.

Important JVM metrics include:

```text
Heap usage
Garbage collection
Old generation
Memory pressure
```

Excessive heap pressure can cause:

```text
Slow searches
Long GC pauses
Performance degradation
Out-of-memory failures
```

---

# 55. Elasticsearch Memory

Do not assume that all available server memory should be assigned to JVM heap.

Elasticsearch also benefits from operating-system filesystem cache.

Therefore memory planning should consider:

```text
JVM heap
+
Filesystem cache
+
Other processes
```

---

# 56. CPU

Monitor:

```text
CPU utilization
Search CPU
Indexing CPU
Merge activity
```

High CPU can result from:

```text
Heavy indexing
Complex queries
Aggregations
Grok processing upstream
Too many concurrent requests
```

---

# 57. Indexing Performance

When Logstash sends documents:

```text
Logstash
   ↓
Bulk requests
   ↓
Elasticsearch
   ↓
Indexing
```

Important factors include:

```text
Document size
Bulk size
Shard count
Disk speed
CPU
Mapping complexity
Replication
```

---

# 58. Bulk Indexing

Sending documents individually can create unnecessary overhead.

A more efficient architecture uses bulk operations:

```text
Documents
 ┌───┬───┬───┬───┐
 ↓   ↓   ↓   ↓   ↓
      Bulk Request
            ↓
      Elasticsearch
```

Logstash commonly uses bulk-oriented delivery to Elasticsearch.

---

# 59. Search Performance

Search performance depends on:

```text
Query complexity
Number of shards
Data volume
Field mappings
Aggregations
Hardware
Concurrent queries
```

Poorly designed queries can be expensive.

---

# 60. Expensive Queries

Examples of potentially expensive operations include:

```text
Large aggregations
High-cardinality aggregations
Broad time ranges
Wildcard-heavy searches
Leading wildcard patterns
Deep pagination
```

Use appropriate mappings and query design.

---

# 61. Time-Based Searching

Logging systems are naturally time-oriented.

A typical search uses:

```text
Last 15 minutes
Last 1 hour
Last 24 hours
Custom time range
```

Kibana uses timestamp fields to filter events.

This reduces the amount of data that needs to be searched.

---

# 62. Time-Based Indexing

A common logging strategy is to use rollover or time-oriented indexes.

Example:

```text
logs-production-000001
logs-production-000002
logs-production-000003
```

or a date-oriented scheme:

```text
logs-production-2026.08.11
```

The exact strategy depends on the lifecycle design.

---

# 63. Index Lifecycle

As logs age:

```text
New
 ↓
Hot
 ↓
Warm
 ↓
Cold
 ↓
Delete
```

Lifecycle policies can automate these transitions where supported.

---

# 64. Hot Data

Hot data is:

```text
Recent
Frequently searched
Performance-sensitive
```

It should generally use fast storage.

Example:

```text
Today's logs
```

---

# 65. Warm Data

Warm data is:

```text
Older
Less frequently searched
Still operationally useful
```

It can potentially use lower-cost resources depending on the architecture.

---

# 66. Cold / Archived Data

Older logs may be:

```text
Rarely accessed
Kept for compliance
Kept for historical investigation
```

They can be moved to lower-cost storage or archived according to organizational requirements.

---

# 67. Retention

A retention policy defines when logs should be deleted or archived.

Example:

```text
Application logs:
30 days

Security logs:
90 days
```

These are examples only.

Retention should be determined by:

```text
Compliance
Business requirements
Security
Cost
Investigation requirements
```

---

# 68. Elasticsearch Security

Production Elasticsearch should be protected.

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
```

Access should be limited to trusted components.

---

# 69. Authentication

Authentication determines:

```text
Who are you?
```

Example:

```text
User
 ↓
Kibana
 ↓
Authentication
 ↓
Elasticsearch
```

The exact authentication mechanism depends on the Elasticsearch deployment and security configuration.

---

# 70. Authorization

Authorization determines:

```text
What are you allowed to access?
```

Example:

```text
User A
 ↓
Application Logs

User B
 ↓
Security Logs
```

Apply least privilege.

---

# 71. Encryption

Use encryption for:

```text
Data in transit
Data at rest
```

Typical architecture:

```text
Client
 ↓ HTTPS/TLS
Kibana
 ↓ TLS
Elasticsearch
```

Storage encryption can be provided by the underlying infrastructure or Elasticsearch deployment, depending on the architecture.

---

# 72. Elasticsearch and Secrets

Never hard-code:

```text
Passwords
API keys
Certificates
Private keys
```

into application repositories.

Use an approved secret-management solution.

For Kubernetes environments, options may include:

```text
Kubernetes Secrets
AWS Secrets Manager
External Secrets
```

---

# 73. Elasticsearch Backups

Production Elasticsearch should have a tested backup strategy.

Conceptually:

```text
Elasticsearch
      ↓
Snapshot
      ↓
Object Storage
```

Snapshots provide recovery capability beyond ordinary shard replicas.

---

# 74. Replica vs Backup

A replica helps with:

```text
Node failure
Availability
Search capacity
```

A snapshot helps with:

```text
Disaster recovery
Accidental deletion
Data restoration
Migration
```

Therefore:

```text
Replica ≠ Backup
```

Both can be important.

---

# 75. Elasticsearch on Kubernetes

Because Elasticsearch is stateful, Kubernetes deployment requires careful planning.

Conceptually:

```text
Stateful Elasticsearch
       ↓
Persistent Volumes
       ↓
Storage
```

Example:

```text
ES-0 → PV-0
ES-1 → PV-1
ES-2 → PV-2
```

---

# 76. Elasticsearch Stateful Architecture

```text
                 Elasticsearch
                      │
                  StatefulSet
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      ES-0          ES-1          ES-2
        │             │             │
       PV-0          PV-1          PV-2
```

A production deployment should use an appropriate Elasticsearch operator or supported deployment mechanism where practical.

---

# 77. Availability Zones

For production EKS:

```text
             Elasticsearch Cluster

         AZ-A        AZ-B        AZ-C
          │           │           │
         ES1         ES2         ES3
```

Distributing nodes across Availability Zones reduces the impact of a single-AZ failure.

---

# 78. Elasticsearch Failure

Suppose:

```text
ES1
 X
```

If replica shards are available:

```text
ES2
 ✓

ES3
 ✓
```

Elasticsearch can recover affected shard copies depending on cluster state.

After recovery:

```text
ES1
 ↓
Rejoin cluster
 ↓
Shard recovery
```

Monitor the recovery process.

---

# 79. Cluster Recovery

After a node failure:

```text
Node failure
     ↓
Cluster detects failure
     ↓
Replica promotion where applicable
     ↓
Cluster health changes
     ↓
Node returns
     ↓
Shard recovery
     ↓
Cluster stabilizes
```

The exact behavior depends on cluster state and shard availability.

---

# 80. Elasticsearch Production Monitoring

Monitor:

```text
Cluster health
Node availability
JVM heap
CPU
Memory
Disk
Shard allocation
Indexing rate
Search latency
Rejected requests
GC
Thread pools
```

This monitoring should be part of your observability platform.

---

# 81. Prometheus Integration

Your monitoring stack can monitor Elasticsearch:

```text
Elasticsearch
      ↓
Metrics Export / Integration
      ↓
Prometheus
      ↓
Grafana
```

Example dashboard metrics:

```text
Cluster health
JVM heap
CPU
Disk
Indexing rate
Search rate
```

---

# 82. Elasticsearch Logs

Elasticsearch also generates its own logs.

Example categories:

```text
Startup
Cluster formation
Shard allocation
Indexing
Search
Errors
Security
```

These logs can themselves be collected by the ELK pipeline.

This creates a useful distinction:

```text
Application Logs
       ↓
       ELK

Elasticsearch Logs
       ↓
       ELK
```

---

# 83. Self-Logging

The logging platform can monitor its own logs.

Example:

```text
Elasticsearch error
       ↓
Log collector
       ↓
Logstash
       ↓
Elasticsearch
       ↓
Kibana
```

This creates an operational feedback loop.

---

# 84. ELK + Kubernetes Metadata

For Kubernetes logs, useful fields include:

```text
cluster
namespace
pod
container
node
deployment
service
```

Example:

```json
{
  "cluster": "prod-eks",
  "namespace": "production",
  "pod": "payment-7d89f",
  "container": "payment",
  "service": "payment",
  "level": "ERROR"
}
```

---

# 85. ELK + Trace ID

If applications use distributed tracing, logs should ideally include:

```text
trace.id
span.id
```

Example:

```json
{
  "service": "payment",
  "level": "ERROR",
  "trace.id": "abc123",
  "message": "Payment failed"
}
```

Then:

```text
Kibana
 ↓
trace.id
 ↓
Jaeger
 ↓
Distributed Trace
```

This connects logs and traces.

---

# 86. ELK + Metrics

Metrics identify patterns.

Example:

```text
Prometheus
 ↓
payment_5xx_rate ↑
```

Then:

```text
Kibana
 ↓
service:payment
level:ERROR
```

Logs provide the detailed reason.

---

# 87. Three-Pillar Architecture

Your complete environment:

```text
                    Observability
                         │
           ┌─────────────┼─────────────┐
           ↓             ↓             ↓
        Metrics         Logs         Traces
           ↓             ↓             ↓
      Prometheus         ELK          Jaeger
           ↓             ↓             ↓
        Grafana        Kibana       Jaeger UI
```

Elasticsearch is the central storage/search engine for the logging side.

---

# 88. Real-World Incident Example

Suppose:

```text
Payment error rate increased.
```

Prometheus detects:

```text
payment_5xx_rate = 8%
```

Grafana displays the alert.

Engineer opens Kibana:

```text
service:payment
level:ERROR
```

Logs show:

```text
Database connection timeout
```

The engineer finds:

```text
trace.id = abc123
```

Then opens Jaeger:

```text
Order
 ↓
Payment
 ↓
Database
 ↓
Timeout
```

This gives a complete incident investigation path.

---

# 89. Elasticsearch Capacity Planning Example

Suppose:

```text
Daily logs = 200 GB
Retention = 30 days
```

Raw storage:

```text
200 GB × 30
=
6 TB
```

But actual capacity needs to account for:

```text
Replica copies
Index overhead
Segment overhead
System overhead
Growth
Failure headroom
```

Therefore 6 TB is not the final production storage requirement.

---

# 90. Elasticsearch Architecture for Your Project

For your DevOps/DevSecOps environment, a practical architecture could be:

```text
                         EKS
                          │
                 Application Pods
                          │
                          ↓
                 Fluent Bit / Filebeat
                          │
                          ↓
                  Logstash Cluster
                    ┌─────┴─────┐
                    ↓           ↓
                  LS-A        LS-B
                    │           │
                    └─────┬─────┘
                          ↓
               Elasticsearch Cluster
                  ┌───────┼───────┐
                  ↓       ↓       ↓
                 ES1     ES2     ES3
                  │       │       │
                  └───────┼───────┘
                          ↓
                       Kibana
                          │
                         ALB
                          │
                          ↓
                        Users
```

---

# 91. Production Elasticsearch Design

For production, focus on:

```text
High availability
Persistent storage
Shard strategy
Replica strategy
Capacity planning
Security
Backups
Monitoring
Retention
Disaster recovery
```

Do not treat Elasticsearch as simply another stateless application.

---

# 92. Common Elasticsearch Mistakes

### Mistake 1: One node in production

```text
Elasticsearch
    ↓
One node
```

This creates a single point of failure.

---

### Mistake 2: No persistent storage

```text
Pod
 ↓
Container filesystem
```

A Pod restart can result in data loss.

---

### Mistake 3: Too many shards

```text
Thousands of tiny shards
```

This creates unnecessary cluster overhead.

---

### Mistake 4: Unlimited retention

```text
Keep everything forever
```

Storage will continuously grow.

---

### Mistake 5: Exposing Elasticsearch publicly

```text
Internet
 ↓
Elasticsearch
```

This creates a major security risk.

---

# 93. Common Elasticsearch Mistakes

### Mistake 6: Logging secrets

```text
password=...
token=...
api_key=...
```

Prevent sensitive information from entering logs.

---

### Mistake 7: Uncontrolled dynamic fields

```text
user_field_1
user_field_2
user_field_3
...
```

This can cause mapping explosion.

---

### Mistake 8: Treating replicas as backups

```text
Replica = Backup
```

This is incorrect.

Use snapshots for recovery.

---

# 94. Elasticsearch Troubleshooting Flow

When Elasticsearch has a problem:

```text
1. Cluster Health
        ↓
2. Node Health
        ↓
3. Disk
        ↓
4. JVM / Memory
        ↓
5. Shards
        ↓
6. Indexing
        ↓
7. Search
        ↓
8. Network
        ↓
9. Recent Changes
```

Avoid making destructive changes before identifying the failure.

---

# 95. Logs Not Indexed

If logs are generated but not searchable:

```text
Application
     ↓
Collector
     ↓
Logstash
     ↓
Elasticsearch X
```

Check:

```text
Logstash output
Elasticsearch connectivity
Authentication
Index permissions
Index existence
Indexing errors
Cluster health
```

---

# 96. Elasticsearch Cluster Red

Troubleshooting:

```text
Cluster Health
      ↓
Unassigned Shards
      ↓
Allocation Explanation
      ↓
Node Health
      ↓
Disk
      ↓
Replica / Primary Status
```

If a primary shard is unavailable, prioritize recovery of that data path.

---

# 97. Elasticsearch Disk Full

Troubleshooting:

```text
Disk usage
    ↓
Largest indexes
    ↓
Retention policy
    ↓
Lifecycle policy
    ↓
Available capacity
    ↓
Add capacity / archive / delete according to policy
```

Never blindly delete indexes.

---

# 98. Elasticsearch Performance Problems

If searches become slow:

```text
Check CPU
Check JVM heap
Check GC
Check disk
Check shard count
Check query complexity
Check aggregation
Check concurrent searches
```

Then optimize the actual bottleneck.

---

# 99. Interview Answer: What Is Elasticsearch?

```text
"Elasticsearch is a distributed search and analytics engine.

In an ELK-based logging platform, Logstash sends processed log events
to Elasticsearch. Elasticsearch indexes those events and distributes
the data across shards and nodes.

This allows engineers to search, filter and aggregate large volumes
of logs efficiently.

Kibana then queries Elasticsearch and provides the interface for
log investigation and visualization."
```

---

# 100. Interview Answer: Explain Shards and Replicas

```text
"An Elasticsearch index is divided into primary shards, which allow
the data to be distributed across multiple nodes.

Replica shards are copies of primary shards. They provide additional
data copies and can improve availability and search capacity.

For example, an index may have three primary shards and one replica
for each primary. Elasticsearch distributes those shards across the
cluster according to its allocation rules."
```

---

# 101. Interview Answer: Why Elasticsearch Cluster?

```text
"I use an Elasticsearch cluster instead of a single node for
production because logs can grow quickly and the platform needs
availability and scalability.

Multiple nodes allow shards and replicas to be distributed across
the infrastructure.

If a node fails, replicas can provide recovery paths where
available, and additional nodes allow the platform to scale as log
volume grows."
```

---

# 102. Interview Answer: Elasticsearch Disk Is Full

```text
"First I would identify the disk usage and determine which indexes
are consuming the most space.

Then I would check retention and lifecycle policies and verify
whether old data can be archived or deleted according to policy.

I would also check Elasticsearch disk watermarks and shard
allocation.

If the growth is expected, I would increase storage capacity or
scale the cluster.

I would not blindly delete production indexes because that could
cause permanent data loss."
```

---

# 103. Interview Answer: Elasticsearch Cluster Is Red

```text
"I would first inspect cluster health and identify the affected
primary or replica shards.

Then I would check node availability, disk watermarks, JVM memory,
allocation failures and recent cluster changes.

If a primary shard is unavailable, I would prioritize restoring the
node or data path responsible for that shard.

I would use snapshots for disaster recovery if normal shard recovery
is not sufficient."
```

---

# 104. Interview Answer: How Do You Scale Elasticsearch?

```text
"I first determine whether the bottleneck is indexing, searching,
storage, CPU, memory or shard distribution.

If the workload is growing, I can add data nodes and rebalance
shards.

I also review shard sizing, replica count, index lifecycle and
retention.

For very large environments, I may separate node roles so that
cluster-management, ingest, data and coordinating workloads do not
compete unnecessarily."
```

---

# 105. Final Elasticsearch Mental Model

Remember:

```text
                  CLUSTER
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        NODE       NODE       NODE
          │          │          │
          └──────────┼──────────┘
                     │
                   INDEX
                     │
              ┌──────┴──────┐
              ↓             ↓
          PRIMARY        REPLICA
           SHARDS         SHARDS
              │             │
              └──────┬──────┘
                     ↓
                 DOCUMENTS
                     │
                   FIELDS
```

And the logging architecture:

```text
Application
     ↓
Collector
     ↓
Logstash
     ↓
Elasticsearch
     ↓
Kibana
```

The most important production concept is:

**Elasticsearch is a distributed, stateful search and analytics engine. In a production logging platform, you must design the cluster, shards, replicas, storage, security, retention, monitoring, and disaster recovery together—not independently.**
