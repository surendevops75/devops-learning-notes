# ELK Centralized Logging

> Production-style centralized logging project for an AWS EKS microservices platform using Elasticsearch, Logstash and Kibana.
>
> This project focuses on the complete logging lifecycle: application log generation, Kubernetes collection, transport, parsing, enrichment, indexing, visualization, retention, security, troubleshooting, scaling, disaster recovery and interview-ready production explanations.

---

# 1. Project Overview

## Project Name

**Centralized Logging for EKS Microservices using ELK**

## Objective

Build a centralized logging platform for applications running on Kubernetes/EKS.

The platform should allow engineers to:

- Collect logs from Kubernetes workloads
- Centralize logs from multiple pods and nodes
- Parse unstructured logs
- Convert logs into structured fields
- Enrich logs with Kubernetes metadata
- Search logs
- Build dashboards
- Investigate incidents
- Correlate application errors with deployments
- Control retention
- Secure access
- Scale log ingestion
- Troubleshoot logging failures

Primary stack:

```text
AWS EKS
Docker
Kubernetes
Elasticsearch
Logstash
Kibana
```

A production deployment may also use a lightweight collector such as Fluent Bit or another supported log shipper. The core ELK processing and storage flow remains:

```text
Applications
     |
     v
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
     |
     v
Engineers
```

---

# 2. Why Centralized Logging?

Without centralized logging:

```text
Engineer
   |
   +--> kubectl logs pod-A
   +--> kubectl logs pod-B
   +--> kubectl logs pod-C
   +--> SSH node
   +--> search multiple files
```

This becomes difficult when:

- There are many pods
- Pods are recreated
- Nodes scale dynamically
- Multiple services are involved
- An incident spans multiple services
- Logs must be retained after pods disappear

Centralized logging provides:

```text
Many workloads
     |
     v
Central log platform
     |
     +--> Search
     +--> Filtering
     +--> Aggregation
     +--> Dashboards
     +--> Retention
     +--> Incident investigation
```

---

# 3. Real-World Scenario

Assume an EKS microservices platform:

```text
                         Internet
                            |
                            v
                      AWS ALB / Ingress
                            |
             +--------------+--------------+
             |              |              |
             v              v              v
           User          Product         Order
          Service        Service        Service
             |              |              |
             +--------------+--------------+
                            |
                      Database / Queue
```

Every service produces logs:

```text
user-service
product-service
order-service
payment-service
inventory-service
notification-service
```

The goal is to search all logs centrally.

---

# 4. Production Architecture

A practical architecture:

```text
                         EKS Cluster
                              |
          +-------------------+-------------------+
          |                   |                   |
          v                   v                   v
       Node 1              Node 2              Node 3
          |                   |                   |
       Pods                Pods                Pods
          |                   |                   |
          +-------------------+-------------------+
                              |
                     Container Logs
                              |
                              v
                    Log Collector DaemonSet
                              |
                              v
                         Logstash
                              |
                    +---------+---------+
                    |                   |
                    v                   v
               Parsing              Enrichment
                    |                   |
                    +---------+---------+
                              |
                              v
                       Elasticsearch
                              |
                              v
                           Kibana
                              |
                              v
                         Engineers
```

---

# 5. Why a DaemonSet Collector?

Kubernetes nodes produce container logs locally.

A common pattern is:

```text
One collector pod
      |
      v
Each Kubernetes node
```

Therefore a DaemonSet is commonly used.

Architecture:

```text
Node 1 -> Collector 1
Node 2 -> Collector 2
Node 3 -> Collector 3
```

When a node is added:

```text
New Node
   |
   v
DaemonSet
   |
   v
New Collector Pod
```

This makes collection automatically follow cluster capacity.

---

# 6. Application Logging Strategy

Applications should preferably write logs to:

```text
stdout
stderr
```

rather than relying on application-specific files inside ephemeral containers.

Why?

```text
Application
   |
   v
stdout/stderr
   |
   v
Kubernetes/container runtime
   |
   v
Collector
```

This aligns application logging with Kubernetes operations.

---

# 7. Structured Logging

Prefer:

```json
{
  "timestamp": "2026-08-15T10:30:00Z",
  "level": "ERROR",
  "service": "payment-service",
  "message": "Payment failed",
  "order_id": "12345",
  "status": 500
}
```

over:

```text
2026-08-15 Payment failed for order 12345
```

Structured logs make filtering and aggregation easier.

---

# 8. Important Log Fields

Recommended fields:

```text
timestamp
level
message
service
environment
namespace
pod
container
node
version
request_id
trace_id
status_code
http_method
route
duration
```

Avoid putting secrets or unnecessary high-cardinality data into logs.

---

# 9. Log Levels

Common levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
FATAL
```

Production applications should use appropriate levels.

Avoid:

```text
DEBUG everything
```

because it can create excessive ingestion and storage cost.

---

# 10. Log Lifecycle

A production logging lifecycle:

```text
Generate
   |
   v
Collect
   |
   v
Transport
   |
   v
Parse
   |
   v
Enrich
   |
   v
Index
   |
   v
Search
   |
   v
Visualize
   |
   v
Retain
   |
   v
Archive/Delete
```

---

# 11. Elasticsearch Role

Elasticsearch provides:

- Document storage
- Indexing
- Search
- Filtering
- Aggregation
- Full-text queries
- Time-based log analysis

Conceptually:

```text
Log event
   |
   v
JSON document
   |
   v
Elasticsearch index
```

---

# 12. Logstash Role

Logstash is responsible for:

- Receiving events
- Parsing
- Filtering
- Transforming
- Enriching
- Routing
- Sending events to Elasticsearch

Architecture:

```text
Input
  |
  v
Filter
  |
  v
Output
```

Example:

```text
Container Log
     |
     v
JSON Parsing
     |
     v
Kubernetes Metadata
     |
     v
Field Normalization
     |
     v
Elasticsearch
```

---

# 13. Kibana Role

Kibana provides:

- Log search
- Discover
- Visualizations
- Dashboards
- Filtering
- Aggregations
- Operational investigation

Flow:

```text
Elasticsearch
      |
      v
    Kibana
      |
      v
 Engineers
```

---

# 14. ELK vs EFK

ELK:

```text
Elasticsearch
Logstash
Kibana
```

EFK commonly means:

```text
Elasticsearch
Fluent Bit / Fluentd
Kibana
```

A lightweight collector is often preferred for node-level collection, while Logstash can remain the central processing layer.

---

# 15. Recommended EKS Architecture

For a scalable Kubernetes implementation:

```text
Pods
 |
 v
Container Runtime Logs
 |
 v
Fluent Bit / Collector
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

This separates:

```text
Collection
```

from:

```text
Processing
```

and:

```text
Storage/Search
```

---

# 16. Namespace

Create:

```bash
kubectl create namespace logging
```

Verify:

```bash
kubectl get namespace logging
```

Production logging components should have clearly defined resource policies and security boundaries.

---

# 17. Prerequisites

Required:

```text
AWS account
EKS cluster
kubectl
Helm
Docker
Kubernetes knowledge
Storage knowledge
```

Verify:

```bash
kubectl get nodes
helm version
```

---

# 18. Elasticsearch Deployment Considerations

Before deploying Elasticsearch, decide:

```text
Cluster size
Memory
CPU
Storage
Replication
Retention
Index strategy
Network
Security
Backup
```

Do not treat Elasticsearch as a small stateless application.

---

# 19. Elasticsearch Architecture

A production Elasticsearch cluster may contain:

```text
+-------------------+
| Master-eligible   |
+-------------------+

+-------------------+
| Data nodes        |
+-------------------+

+-------------------+
| Ingest nodes      |
+-------------------+
```

For smaller environments, node roles may be combined.

For larger environments, roles can be separated.

---

# 20. Elasticsearch Data Flow

```text
Logstash
   |
   v
Elasticsearch
   |
   +--> Index
   |
   +--> Shard
   |
   +--> Replica
   |
   v
Search
```

---

# 21. Elasticsearch Index

An index is a logical collection of documents.

Example:

```text
logs-2026.08.15
```

A document represents one log event.

Concept:

```json
{
  "@timestamp": "2026-08-15T10:30:00Z",
  "service": "order-service",
  "level": "ERROR",
  "message": "Database connection failed"
}
```

---

# 22. Time-Based Indices

For logs, time-based indexing is common:

```text
logs-2026.08.13
logs-2026.08.14
logs-2026.08.15
```

Benefits:

- Easier retention
- Easier deletion
- Operational isolation
- Better time-range management

Exact index lifecycle design should match the Elasticsearch version and deployment model.

---

# 23. Index Naming Strategy

Example:

```text
logs-production-app-2026.08.15
```

Possible dimensions:

```text
environment
application
date
```

Avoid creating an excessive number of tiny indices.

---

# 24. Elasticsearch Shards

An index can be divided into shards.

Concept:

```text
Index
 |
 +--> Primary Shard 1
 +--> Primary Shard 2
 +--> Primary Shard 3
```

Sharding allows data and query workload to be distributed.

---

# 25. Elasticsearch Replicas

Example:

```text
Primary
   |
   +--> Replica
```

Replicas provide:

- Redundancy
- Failover
- Additional read capacity

Replica count should match availability requirements and cluster capacity.

---

# 26. Shard Sizing

Too many shards:

```text
Metadata overhead
Memory overhead
Management complexity
```

Too few or oversized shards:

```text
Slow recovery
Uneven distribution
Large segments
```

Shard sizing must be based on actual data volume and workload.

---

# 27. Elasticsearch JVM Memory

Elasticsearch relies heavily on JVM memory.

Monitor:

```text
Heap
GC
CPU
Disk
Thread pools
Search latency
Indexing latency
```

Do not allocate all node memory to the JVM heap.

Leave memory for the operating system and filesystem cache.

---

# 28. Elasticsearch Disk Watermarks

Elasticsearch uses disk thresholds to protect the cluster.

As disk utilization becomes high, Elasticsearch may restrict shard allocation.

Therefore:

```text
Disk usage
```

is a critical production metric.

---

# 29. Elasticsearch Persistent Storage

For EKS, persistent storage should be used.

Common pattern:

```text
Elasticsearch Pod
       |
       v
PersistentVolume
       |
       v
AWS-backed storage
```

Storage type and configuration should be selected based on workload, performance and availability requirements.

---

# 30. Kubernetes StatefulSet

Elasticsearch is stateful.

Therefore a Kubernetes deployment commonly uses:

```text
StatefulSet
```

rather than a simple Deployment.

Benefits:

- Stable identities
- Persistent volumes
- Ordered lifecycle behavior
- Stateful workload semantics

---

# 31. Elasticsearch Installation with Helm

A production implementation should use a maintained chart/operator appropriate to the Elasticsearch version and organizational standards.

The important installation principle is:

```text
Version pinning
+
Persistent storage
+
Resource configuration
+
Security
+
Cluster topology
```

Do not deploy an unreviewed latest version directly into production.

---

# 32. Verify Elasticsearch

Example:

```bash
kubectl get pods -n logging
kubectl get svc -n logging
kubectl get pvc -n logging
```

Then verify cluster health using the supported Elasticsearch API endpoint.

Typical health states:

```text
green
yellow
red
```

---

# 33. Elasticsearch Cluster Health

## Green

All primary and replica shards are allocated.

## Yellow

Primaries are allocated but one or more replicas are not.

## Red

One or more primary shards are unavailable.

Production incident priority:

```text
Red > Yellow > Green
```

---

# 34. Elasticsearch Troubleshooting — Yellow

Possible causes:

- Not enough nodes
- Replica count too high
- Allocation constraints
- Disk watermark
- Node failure

Check:

```text
Cluster health
Shard allocation
Node availability
Disk
```

---

# 35. Elasticsearch Troubleshooting — Red

Immediate checks:

```text
Cluster health
Failed shards
Node health
Disk
Storage
Recent changes
```

A red cluster can cause missing or unavailable log data.

---

# 36. Logstash Architecture

```text
Input
  |
  v
Filter
  |
  v
Output
```

Example:

```text
Beats / TCP / HTTP
       |
       v
    Logstash
       |
  +----+----+
  |         |
Parse     Enrich
  |         |
  +----+----+
       |
       v
Elasticsearch
```

---

# 37. Logstash Pipeline

A pipeline typically contains:

```text
input {
}

filter {
}

output {
}
```

Example conceptual configuration:

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  json {
    source => "message"
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

Production configurations should use secure transport and credentials where required.

---

# 38. Logstash Input

Common inputs include:

```text
Beats
TCP
HTTP
Kafka
Files
```

In Kubernetes architectures, a collector can forward events to Logstash through a supported protocol.

---

# 39. Logstash Filters

Useful filters:

```text
json
grok
mutate
date
dissect
drop
geoip
```

Use the simplest parser that reliably matches the log format.

---

# 40. JSON Filter

If application logs are JSON:

```json
{
  "level": "ERROR",
  "service": "payment",
  "message": "Payment failed"
}
```

Logstash can parse the JSON into structured fields.

---

# 41. Grok

Grok parses semi-structured text.

Example:

```text
2026-08-15 10:30:00 ERROR payment failed
```

A Grok pattern can extract:

```text
timestamp
level
message
```

Avoid unnecessarily complex Grok patterns when application-side structured logging is possible.

---

# 42. Dissect

Dissect is useful when logs have a predictable delimiter-based structure.

Compared with Grok:

```text
Grok
Pattern matching

Dissect
Position-based parsing
```

Dissect can be simpler and faster for predictable formats.

---

# 43. Mutate

Mutate can:

```text
Rename fields
Remove fields
Convert fields
Add fields
Change fields
```

Example conceptual transformation:

```text
status_code
    |
    v
status
```

---

# 44. Date Filter

Normalize event timestamps.

Why?

Without correct timestamps:

```text
Log event occurred at 10:00
Elasticsearch sees 10:10
```

Incident investigation becomes misleading.

---

# 45. Kubernetes Metadata

Logs should ideally contain metadata such as:

```text
namespace
pod
container
node
labels
deployment
```

This enables queries such as:

```text
namespace = production
service = payment
level = ERROR
```

---

# 46. Log Enrichment

Example:

```text
Raw log
  |
  v
Add:
  environment
  cluster
  namespace
  pod
  container
  node
  service
  version
```

Then:

```text
Search becomes much easier.
```

---

# 47. Collector DaemonSet

A typical collector architecture:

```text
Node
 |
 +--> /var/log/containers
 |
 v
Collector
 |
 v
Logstash
```

The collector must have appropriate permissions and volume mounts.

---

# 48. Kubernetes Container Logs

Container logs are commonly available under node log paths such as:

```text
/var/log/containers/
```

and related runtime paths.

Exact paths and formats depend on the container runtime and Kubernetes distribution.

---

# 49. Collector Volume Mounts

A collector generally needs access to host log files.

Conceptually:

```yaml
volumeMounts:
  - name: varlog
    mountPath: /var/log
```

and:

```yaml
volumes:
  - name: varlog
    hostPath:
      path: /var/log
```

Production configurations should follow the collector's documented security requirements.

---

# 50. Collector Configuration

Typical collector responsibilities:

```text
Tail files
Parse container format
Add Kubernetes metadata
Buffer
Forward
Retry
```

---

# 51. Why Buffering Matters

If Logstash is temporarily unavailable:

```text
Collector
   |
   X
Logstash unavailable
```

Without buffering:

```text
Logs may be lost.
```

With buffering:

```text
Collector
   |
   v
Buffer
   |
   v
Retry
   |
   v
Logstash
```

Buffer sizing should reflect expected outage duration and available disk/memory.

---

# 52. Backpressure

If Elasticsearch slows down:

```text
Elasticsearch slow
       |
       v
Logstash backlog
       |
       v
Collector backlog
```

A production system must handle backpressure deliberately.

Possible strategies:

- Buffering
- Persistent queues
- Rate control
- Scaling
- Temporary sampling where appropriate

---

# 53. Logstash Persistent Queue

Persistent queues can help protect events during temporary downstream failures.

Concept:

```text
Input
 |
 v
Persistent Queue
 |
 v
Processing
 |
 v
Elasticsearch
```

This uses disk and should be sized and monitored carefully.

---

# 54. Logstash Multiple Pipelines

For different workloads:

```text
Application logs
Security logs
Audit logs
Infrastructure logs
```

separate pipelines may be useful.

Example:

```text
Pipeline A -> Application
Pipeline B -> Security
Pipeline C -> Audit
```

---

# 55. Logstash Scaling

Logstash can be scaled horizontally:

```text
Collector
   |
   +--> Logstash 1
   |
   +--> Logstash 2
   |
   +--> Logstash 3
```

Use a load-balancing or durable transport design appropriate to the deployment.

---

# 56. Logstash Monitoring

Monitor:

```text
Events in
Events out
Pipeline throughput
Queue size
Processing latency
CPU
Memory
JVM heap
Failed events
```

---

# 57. Kibana Installation

Kibana should be deployed in a way compatible with the Elasticsearch version.

Important configuration:

```text
Elasticsearch endpoint
Authentication
TLS
Resource limits
Persistent configuration
Ingress
```

---

# 58. Kibana Access

For local testing:

```bash
kubectl port-forward \
  svc/kibana \
  -n logging 5601:5601
```

Then:

```text
http://localhost:5601
```

Production access should use controlled authentication and HTTPS.

---

# 59. Kibana Data View

Create a data view matching the log index pattern.

Example:

```text
logs-production-*
```

Select the timestamp field:

```text
@timestamp
```

Then use Discover for investigation.

---

# 60. Kibana Discover

Discover can filter:

```text
service
namespace
pod
level
status
message
```

Example:

```text
service : "payment-service"
AND level : "ERROR"
```

---

# 61. Search by Namespace

Example concept:

```text
kubernetes.namespace_name : "production"
```

Exact field names depend on the collector/enrichment format.

---

# 62. Search by Pod

Example concept:

```text
kubernetes.pod_name : "payment-service-*"
```

Use wildcard behavior supported by the Kibana query language and field type.

---

# 63. Search Errors

Concept:

```text
level : "ERROR"
```

Then narrow by:

```text
service
namespace
version
time
```

---

# 64. Search a Deployment Incident

Suppose a deployment happened at:

```text
10:30
```

Investigate:

```text
10:20 - 10:40
```

Filter:

```text
service
version
ERROR
WARN
```

Compare before and after deployment.

---

# 65. Log Dashboards

Useful dashboards:

```text
Application Overview
Error Overview
Infrastructure Logs
Security Events
API Logs
Deployment Investigation
```

---

# 66. Application Log Dashboard

Panels:

```text
Total log events
ERROR count
WARN count
Top services
Top errors
HTTP status
Response time
Recent deployments
```

---

# 67. Error Dashboard

Track:

```text
Errors/min
Errors by service
Errors by endpoint
Errors by version
Errors by namespace
Top error messages
```

---

# 68. Log Volume Dashboard

Track:

```text
Events/sec
Events/min
GB/day
Logs by service
Logs by namespace
```

This is important for cost management.

---

# 69. High-Cardinality Log Fields

Logs can tolerate more dimensions than metrics, but excessive unique fields can still cause:

```text
Large storage
Expensive indexing
Slow searches
Large documents
```

Avoid logging unnecessary:

```text
request payloads
large objects
tokens
session data
```

---

# 70. Sensitive Data

Never log:

```text
Passwords
Access tokens
Private keys
Credit card data
Secrets
Session cookies
```

Use application-level filtering and Logstash/collector filtering as defense in depth.

---

# 71. PII

Personal data requires special handling.

Potential examples:

```text
Email
Phone
Address
Government identifiers
Customer data
```

Use:

```text
Masking
Redaction
Filtering
Access control
Retention policies
```

---

# 72. Log Redaction

Concept:

```text
Original:
email=user@example.com

Stored:
email=u***@example.com
```

Where appropriate, redact before storage so sensitive data does not become permanently indexed.

---

# 73. Elasticsearch Security

Production security should include:

```text
Authentication
Authorization
TLS
Network isolation
Least privilege
Secret management
Audit controls
```

Do not expose Elasticsearch directly to the public Internet.

---

# 74. Network Architecture

Preferred:

```text
Internet
   |
   v
Kibana / controlled access
   |
   v
Elasticsearch
```

not:

```text
Internet
   |
   v
Elasticsearch:9200
```

Elasticsearch should remain protected inside the trusted network boundary.

---

# 75. Kubernetes RBAC

Define access by role.

Example:

```text
Platform team
    |
    +--> Full logging administration

Application team
    |
    +--> Read application logs

Security team
    |
    +--> Security/audit logs
```

Use least privilege.

---

# 76. Multi-Tenancy

If multiple teams share the logging platform, separate access by:

```text
Namespace
Application
Team
Environment
```

Use Elasticsearch/Kibana authorization mechanisms appropriate to the deployment.

---

# 77. Environment Separation

Log data should clearly identify:

```text
development
staging
production
```

A common field:

```text
environment
```

This prevents engineers from investigating the wrong environment.

---

# 78. Retention Strategy

Retention should be based on:

```text
Operational need
Compliance
Cost
Incident requirements
Business value
```

Example conceptual policy:

```text
Hot
  |
  v
Recent searchable logs

Warm
  |
  v
Older but searchable logs

Cold/archive
  |
  v
Long-term low-cost retention

Delete
```

The exact lifecycle depends on Elasticsearch capabilities and organizational requirements.

---

# 79. Hot Data

Recent logs:

```text
High search frequency
High performance requirement
Fast storage
```

---

# 80. Warm Data

Older logs:

```text
Less frequent searches
Lower performance requirement
Potentially lower-cost storage
```

---

# 81. Cold / Archived Data

Used when:

```text
Compliance
Audit
Historical investigations
```

requires longer retention without keeping all data on high-performance storage.

---

# 82. Index Lifecycle Management

Use lifecycle policies where supported to:

```text
Roll over
Move
Shrink
Delete
```

based on:

```text
Age
Size
Storage tier
```

---

# 83. Log Rotation

Avoid a single forever-growing index.

Prefer controlled time/size-based rollover.

Benefits:

```text
Manageable indices
Predictable storage
Simpler deletion
Better recovery
```

---

# 84. Storage Capacity Planning

Estimate:

```text
Logs/day
Average event size
Compression
Replication
Retention
```

Concept:

```text
Daily raw volume
      x
Retention days
      x
Replication factor
      +
Overhead
```

This gives an approximate capacity requirement.

---

# 85. Example Capacity Calculation

Suppose:

```text
Raw logs = 100 GB/day
Retention = 14 days
Replication = 1 replica
```

Very rough raw replicated requirement:

```text
100 GB
x 14
x 2
= 2.8 TB
```

Actual Elasticsearch disk requirements differ because of:

- Index overhead
- Segment overhead
- Compression
- Mappings
- Replicas
- Free-space requirements

Never provision exactly to the raw calculation.

---

# 86. Elasticsearch Disk Headroom

Always maintain operational headroom.

Do not run:

```text
95-100% disk
```

Plan for:

```text
Recovery
Merges
Shard movement
Temporary spikes
```

---

# 87. Log Ingestion Rate

Monitor:

```text
events/sec
bytes/sec
events by service
failed events
```

A sudden spike may indicate:

```text
Application error loop
Debug logging
Traffic spike
Attack
Deployment problem
```

---

# 88. Logging Cost Optimization

Control:

```text
Log level
Retention
Event size
Duplicate logs
Unnecessary fields
Debug logging
High-volume health checks
```

Do not blindly send every possible log to long-term storage.

---

# 89. Sampling Logs

For extremely high-volume systems, sampling can be considered for specific low-value events.

Never sample away:

```text
Security events
Critical failures
Audit events
Required compliance logs
```

Sampling policy should be explicit.

---

# 90. Health Checks and Log Noise

Kubernetes probes may generate many requests.

If every probe produces an application log:

```text
/health
/ready
/health
/ready
...
```

log volume can grow unnecessarily.

Consider filtering low-value probe logs while preserving useful health signals.

---

# 91. Application Logging Best Practice

Instead of:

```text
Starting function
Starting function
Starting function
```

prefer meaningful events:

```text
Order created
Payment failed
Database timeout
External API timeout
Deployment version
```

Logs should help answer:

```text
What happened?
Where?
When?
Why?
For which request?
```

---

# 92. Request ID

A request ID helps correlate logs within a request path.

Example:

```text
request_id = abc123
```

Search:

```text
request_id : "abc123"
```

and retrieve related events.

---

# 93. Trace ID

Even when a tracing platform is not currently deployed, applications may propagate a trace ID.

Example:

```text
trace_id = 4bf92f...
```

This prepares the logging platform for future distributed tracing integration.

---

# 94. Correlation Across Services

Example:

```text
User
 |
 v
Order
 |
 v
Payment
 |
 v
Inventory
```

If all services propagate:

```text
request_id / trace_id
```

engineers can search related logs across services.

---

# 95. Logging During Deployments

Add:

```text
version
deployment_id
commit_sha
```

where appropriate.

Then incident investigation becomes:

```text
Error increase
    |
    v
Version changed
    |
    v
Deployment correlated
```

---

# 96. Logging and GitOps

For ArgoCD-based deployments:

```text
Git
 |
 v
Manifest change
 |
 v
ArgoCD
 |
 v
EKS deployment
 |
 v
Application logs
```

Include deployment/version metadata so log investigations can correlate with Git changes.

---

# 97. Log Pipeline Failure

If Logstash fails:

```text
Collector
   |
   X
Logstash
```

Questions:

```text
Is the collector buffering?
Is there backpressure?
Is data being dropped?
Is Logstash restarting?
Is Elasticsearch healthy?
```

---

# 98. Collector Failure

If a collector pod fails on one node:

```text
Node
 |
 X
Collector
```

Logs from that node may be affected.

Because collectors are usually DaemonSets:

```text
Check desired vs ready pods
```

---

# 99. Elasticsearch Failure

If Elasticsearch becomes unavailable:

```text
Logstash
   |
   v
Queue / Buffer
   |
   X
Elasticsearch
```

Monitor:

```text
Queue depth
Disk
Cluster health
Node health
Indexing failures
```

---

# 100. Kibana Failure

If Kibana is unavailable:

```text
Elasticsearch still stores logs
```

Therefore:

> Kibana should not be considered the log storage layer.

Restore Kibana separately while preserving Elasticsearch availability.

---

# 101. Troubleshooting — No Logs in Kibana

Use the pipeline:

```text
Application
   |
   v
Container stdout
   |
   v
Node log file
   |
   v
Collector
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

Find the first broken stage.

---

# 102. Check Application Logs

```bash
kubectl logs <pod> -n <namespace>
```

If logs exist:

```text
Application -> Kubernetes
```

is working.

---

# 103. Check Container Log Files

On the relevant node, verify that container logs are being generated according to the runtime's logging configuration.

Do not depend on direct node access as the primary operational method in production.

---

# 104. Check Collector

```bash
kubectl get daemonset -n logging
kubectl get pods -n logging -o wide
kubectl logs <collector-pod> -n logging
```

Look for:

```text
Permission denied
File not found
Connection refused
Backpressure
Parsing errors
```

---

# 105. Check Logstash

```bash
kubectl get pods -n logging
kubectl logs <logstash-pod> -n logging
```

Check:

```text
Input
Pipeline
Filter errors
Output
Elasticsearch connectivity
Queue
```

---

# 106. Check Elasticsearch

Verify:

```text
Cluster health
Indices
Documents
Disk
```

Use the Elasticsearch APIs appropriate to your version and authentication model.

---

# 107. Check Kibana

If Elasticsearch contains logs but Kibana shows nothing:

```text
Check data view
Check index pattern
Check timestamp field
Check time range
Check permissions
```

---

# 108. Troubleshooting — Logs Delayed

Possible causes:

```text
Collector backlog
Logstash queue
Elasticsearch indexing delay
Network latency
CPU saturation
Disk pressure
Large parsing workload
```

Measure:

```text
Event timestamp
Ingestion timestamp
Kibana arrival time
```

---

# 109. Troubleshooting — Duplicate Logs

Possible causes:

- Two collectors reading the same file
- Collector restart state issue
- Duplicate pipeline
- Multiple forwarding paths
- Retry semantics

Check:

```text
Collector configuration
File offsets
Pipeline topology
Event IDs
```

---

# 110. Troubleshooting — Missing Logs

Possible causes:

```text
Collector failure
Parsing drop
Output failure
Buffer overflow
Indexing failure
Disk pressure
Incorrect filter
Retention deletion
```

Trace the event end to end.

---

# 111. Troubleshooting — Elasticsearch Red

Flow:

```text
Red
 |
 v
Check failed shards
 |
 v
Check nodes
 |
 v
Check disk
 |
 v
Check storage
 |
 v
Check allocation
```

Do not immediately delete data to make the cluster green.

---

# 112. Troubleshooting — Elasticsearch High CPU

Investigate:

```text
Indexing rate
Search rate
Merge activity
Shard count
Query patterns
Aggregation workload
GC
```

Potential mitigation:

```text
Reduce expensive queries
Scale appropriately
Control ingestion
Optimize index design
```

---

# 113. Troubleshooting — Elasticsearch High Heap

Possible causes:

```text
Too many shards
High query load
Large aggregations
High indexing load
Mapping explosion
Large field sets
```

Do not only increase heap.

Find the workload causing the pressure.

---

# 114. Mapping Explosion

If applications dynamically create many fields:

```text
user_field_1
user_field_2
user_field_3
...
```

the mapping can grow excessively.

Prevent with:

```text
Controlled schemas
Dynamic mapping strategy
Field filtering
Application log standards
```

---

# 115. Too Many Indices

Bad architecture:

```text
One index per pod
One index per request
One index per user
```

This can create massive index counts.

Prefer controlled index strategies:

```text
environment
application
time
```

where justified.

---

# 116. Troubleshooting — Logstash CPU High

Check:

```text
Grok complexity
Event size
Throughput
Filter count
JSON parsing
Regex patterns
Output latency
```

Prefer structured logging to reduce expensive parsing.

---

# 117. Troubleshooting — Logstash Memory High

Check:

```text
Queue
Batch size
Large events
Persistent queue
JVM heap
Pipeline configuration
```

Large payloads should be controlled.

---

# 118. Troubleshooting — Collector CPU High

Possible causes:

```text
Huge log volume
Complex parsing
Metadata enrichment
Many files
High-frequency logs
```

Scale or simplify collection.

---

# 119. Production Incident — Error Spike

Scenario:

```text
5xx errors increased from 0.5% to 15%.
```

Logging response:

```text
Filter ERROR
   |
   v
Identify service
   |
   v
Identify endpoint
   |
   v
Identify version
   |
   v
Identify common error
```

Then correlate with:

```text
Deployment
Database
External API
Infrastructure
```

---

# 120. Production Incident — Application CrashLoopBackOff

Use:

```bash
kubectl logs <pod> --previous
```

Then search centralized logs by:

```text
pod
service
version
request_id
```

Central logging provides historical context after the pod has restarted.

---

# 121. Production Incident — Node Failure

A node may disappear while engineers are investigating.

Centralized logs remain searchable if they were already indexed.

This is one of the major advantages of centralized logging over relying only on local container logs.

---

# 122. Production Incident — Deployment Regression

Timeline:

```text
10:00 healthy
10:15 deployment
10:17 errors increase
10:18 latency increases
```

Use Kibana to compare:

```text
version A
version B
```

Then determine whether the deployment caused the regression.

---

# 123. Production Incident — Database Timeout

Application logs:

```text
database timeout
```

Search by:

```text
service
database operation
request_id
timestamp
```

Then correlate with infrastructure/database monitoring.

---

# 124. Production Incident — External API Failure

Example:

```text
Payment provider timeout
```

Filter:

```text
service = payment
status = timeout
dependency = payment-provider
```

Measure:

```text
error count
latency
affected requests
```

---

# 125. Production Incident — Log Flood

Scenario:

```text
Logs/day:
100 GB -> 2 TB
```

Possible cause:

```text
Debug enabled
Error loop
Traffic spike
Application bug
Attack
```

Immediate actions:

1. Identify source.
2. Reduce unnecessary logging.
3. Protect Elasticsearch capacity.
4. Preserve critical logs.
5. Investigate root cause.

---

# 126. Production Incident — Elasticsearch Disk Full

Symptoms:

```text
Indexing failures
Cluster health degradation
Shard allocation issues
```

Immediate:

```text
Identify largest indices
Check retention
Check disk
Stop unnecessary ingestion if required
Expand capacity safely
```

Then fix retention and capacity planning.

---

# 127. Production Incident — Logstash Backlog

Symptoms:

```text
Queue increasing
Logs delayed
Elasticsearch slow
```

Investigate:

```text
Elasticsearch throughput
Logstash CPU
Pipeline filters
Collector rate
Network
```

Scale or reduce pressure according to the root cause.

---

# 128. Production Incident — Missing Security Logs

This is higher severity than losing low-value debug logs.

Security/audit pipelines should have:

```text
Dedicated retention
Access controls
Monitoring
Failure alerts
Reliable buffering
```

---

# 129. Logging Reliability

Logging should have its own SLOs.

Example:

```text
99.9% of critical application logs indexed within 30 seconds
```

This separates:

```text
Application SLO
```

from:

```text
Logging pipeline SLO
```

---

# 130. Logging Pipeline Monitoring

Monitor every layer:

```text
Collector
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

Signals:

```text
Throughput
Latency
Errors
Backlog
CPU
Memory
Disk
```

---

# 131. Monitor Collector

Metrics:

```text
Records read
Records sent
Records failed
Buffer usage
CPU
Memory
```

---

# 132. Monitor Logstash

Metrics:

```text
Events in
Events out
Events failed
Queue size
Pipeline latency
JVM
CPU
Memory
```

---

# 133. Monitor Elasticsearch

Metrics:

```text
Cluster health
Node count
CPU
Heap
Disk
Indexing rate
Search latency
Rejected requests
Shard health
```

---

# 134. Monitor Kibana

Metrics:

```text
Availability
Response time
CPU
Memory
Active requests
Errors
```

---

# 135. Logging Backpressure Architecture

```text
Applications
     |
     v
Collectors
     |
     v
Buffer
     |
     v
Logstash
     |
     v
Elasticsearch
```

If Elasticsearch slows:

```text
Buffer grows
```

The system should alert before the buffer is exhausted.

---

# 136. Durable Queue Architecture

At larger scale, an intermediate durable messaging system can be used:

```text
Collectors
    |
    v
Kafka / Durable Queue
    |
    +--> Logstash
    |
    +--> Other Consumers
```

Benefits:

- Decoupling
- Replay
- Buffering
- Independent scaling

This adds operational complexity and should be justified by requirements.

---

# 137. Large-Scale Architecture

```text
                 EKS Clusters
             /        |        \
            /         |         \
           v          v          v
      Collectors  Collectors  Collectors
           \          |          /
            \         |         /
             v        v        v
               Durable Queue
                     |
             +-------+-------+
             |               |
             v               v
         Logstash         Other Consumers
             |
             v
       Elasticsearch
             |
             v
           Kibana
```

---

# 138. Multi-Cluster Logging

Example:

```text
EKS Dev
   |
EKS Stage
   |
EKS Prod
   |
   +----> Central Logging
```

Every event should identify:

```text
cluster
environment
region
```

---

# 139. Multi-Region Logging

Example:

```text
Region A ----\
              \
Region B ------> Central Logging
              /
Region C ----/
```

Consider:

```text
Network cost
Latency
Data residency
Failure domains
Security
```

---

# 140. Cross-Region Disaster Recovery

For critical logs:

```text
Primary
   |
   v
Central Elasticsearch
   |
   v
Backup / Archive
```

Recovery requirements should define:

```text
RPO
RTO
Retention
Archive
```

---

# 141. Backup Strategy

Back up:

```text
Elasticsearch snapshots
Kibana configuration
Index templates
Lifecycle policies
Ingest pipelines
Logstash configuration
Collector configuration
```

Do not assume Kubernetes YAML alone protects Elasticsearch data.

---

# 142. Elasticsearch Snapshots

Snapshots provide a recovery mechanism for Elasticsearch data.

A production snapshot strategy should define:

```text
Frequency
Retention
Repository
Encryption
Restore testing
```

A backup that has never been restored is not fully validated.

---

# 143. Restore Testing

Test:

```text
Create snapshot
   |
   v
Simulate failure
   |
   v
Restore
   |
   v
Verify indices
   |
   v
Verify searches
```

Document recovery time.

---

# 144. Security Architecture

```text
Users
 |
 v
Authenticated Kibana
 |
 v
Elasticsearch
 |
 +--> TLS
 +--> RBAC
 +--> Network restrictions
 +--> Audit
```

Collectors and Logstash should also use secure communication where supported.

---

# 145. TLS

Protect:

```text
Collector -> Logstash
Logstash -> Elasticsearch
Kibana -> Elasticsearch
User -> Kibana
```

Avoid plaintext transport in production unless the network and architecture explicitly justify it and compensating controls exist.

---

# 146. Secret Management

Credentials should come from:

```text
Kubernetes Secrets
AWS Secrets Manager
External secret-management system
```

Do not hard-code credentials into:

```text
Dockerfiles
Git
Helm values committed in plaintext
Logstash configs
```

---

# 147. Logging Secrets Accidentally

A dangerous pattern:

```text
Authorization: Bearer eyJ...
```

appears in logs.

Mitigation:

```text
Application redaction
Collector filtering
Logstash filtering
```

Best solution:

> Prevent secrets from being logged at the source.

---

# 148. Compliance Logging

Some environments require:

```text
Audit logs
Access logs
Administrative actions
Security events
Retention
Immutability
```

Do not mix compliance requirements with ordinary application debug logging without a clear policy.

---

# 149. Cost Model

Main cost drivers:

```text
Log volume
Retention
Replication
Storage
Indexing CPU
Search workload
Network transfer
Backup
```

The most powerful optimization is often:

```text
Reduce unnecessary log volume
```

before adding more hardware.

---

# 150. Cost Optimization Strategy

Step 1:

```text
Measure GB/day
```

Step 2:

```text
Identify top log producers
```

Step 3:

```text
Reduce noisy logs
```

Step 4:

```text
Optimize retention
```

Step 5:

```text
Tune shard/index strategy
```

Step 6:

```text
Right-size infrastructure
```

---

# 151. Production Logging Checklist

## Application

- [ ] Structured logs
- [ ] Correct levels
- [ ] Request ID
- [ ] Version
- [ ] No secrets
- [ ] No unnecessary payloads

## Collector

- [ ] DaemonSet
- [ ] Metadata enrichment
- [ ] Buffering
- [ ] Retry
- [ ] Resource limits
- [ ] Security context

## Logstash

- [ ] Pipeline configuration
- [ ] Parsing
- [ ] Filtering
- [ ] Persistent queue if required
- [ ] Monitoring
- [ ] Resource limits

## Elasticsearch

- [ ] Stateful architecture
- [ ] Persistent storage
- [ ] Replication
- [ ] Shard strategy
- [ ] Retention
- [ ] Snapshots
- [ ] TLS
- [ ] Authentication

## Kibana

- [ ] Authentication
- [ ] RBAC
- [ ] Data views
- [ ] Dashboards
- [ ] TLS

---

# 152. Logging Anti-Patterns

Avoid:

```text
Logging everything at DEBUG
Logging secrets
One index per pod
One index per request
Unbounded mappings
No retention policy
No backups
No disk monitoring
No buffering
No access control
```

---

# 153. Common Design Mistake — Elasticsearch as a Stateless Deployment

Incorrect:

```text
Deployment
 |
 +--> Elasticsearch
```

without durable storage.

Better:

```text
Stateful workload
 |
 +--> Persistent storage
 +--> Stable identity
 +--> Replication
```

---

# 154. Common Design Mistake — Direct Application to Elasticsearch

Possible:

```text
Application
   |
   v
Elasticsearch
```

But this tightly couples the application to the storage platform.

A more flexible architecture:

```text
Application
   |
   v
stdout
   |
   v
Collector
   |
   v
Processing
   |
   v
Storage
```

---

# 155. Common Design Mistake — No Backpressure

If downstream storage fails:

```text
Collector -> Logstash -> Elasticsearch X
```

without buffering, logs can be lost.

Design for temporary downstream failure.

---

# 156. Common Design Mistake — No Timestamp Normalization

Logs from multiple systems may have different timestamps.

Normalize to a common event timestamp such as:

```text
@timestamp
```

and preserve original timestamps when useful.

---

# 157. Common Design Mistake — Parsing Everything in Logstash

If applications can emit structured JSON:

```text
Application
   |
   v
JSON
```

prefer that over complex regex parsing.

This reduces:

```text
CPU
Complexity
Parsing failures
```

---

# 158. Common Design Mistake — Huge Log Documents

Avoid storing:

```text
Entire request body
Entire response body
Large stack payloads
Binary data
```

unless there is a justified requirement.

Large documents increase:

```text
Storage
Network
Indexing
Query cost
```

---

# 159. Common Design Mistake — No Ownership

An alert says:

```text
Elasticsearch disk 90%
```

but nobody owns it.

Every critical logging component needs:

```text
Owner
Runbook
Escalation
```

---

# 160. Project Implementation Plan

## Phase 1

Prepare EKS and logging namespace.

## Phase 2

Deploy Elasticsearch.

## Phase 3

Validate storage and cluster health.

## Phase 4

Deploy Logstash.

## Phase 5

Deploy collector DaemonSet.

## Phase 6

Send test logs.

## Phase 7

Parse and enrich logs.

## Phase 8

Index logs into Elasticsearch.

## Phase 9

Deploy Kibana.

## Phase 10

Create data views.

## Phase 11

Create dashboards.

## Phase 12

Implement retention and security.

## Phase 13

Test failure scenarios.

## Phase 14

Implement backup and DR.

---

# 161. Phase 1 — Namespace

```bash
kubectl create namespace logging
```

Verify:

```bash
kubectl get ns logging
```

---

# 162. Phase 2 — Elasticsearch

Validate:

```text
Version
Node count
Storage
Resources
Security
```

Then deploy using the selected supported method.

---

# 163. Phase 3 — Elasticsearch Validation

Check:

```bash
kubectl get pods -n logging
kubectl get pvc -n logging
kubectl get svc -n logging
```

Then verify cluster health through its API.

---

# 164. Phase 4 — Logstash

Deploy with:

```text
Input
Filter
Output
```

Validate pipeline syntax before production rollout.

---

# 165. Phase 5 — Collector

Deploy collector as:

```text
DaemonSet
```

Verify:

```bash
kubectl get daemonset -n logging
kubectl get pods -n logging -o wide
```

Expected:

```text
One collector per eligible node
```

---

# 166. Phase 6 — Test Log

Deploy a test application:

```text
test-log-generator
```

Generate:

```text
INFO
WARN
ERROR
```

Verify the event appears in the collector.

---

# 167. Phase 7 — Parsing

Test:

```text
JSON
Grok
Dissect
Date
Mutate
```

Prefer JSON where possible.

---

# 168. Phase 8 — Elasticsearch Index

Verify:

```text
Index exists
Documents exist
Timestamp correct
Fields mapped correctly
```

---

# 169. Phase 9 — Kibana

Deploy and connect it to Elasticsearch.

Verify:

```text
Kibana -> Elasticsearch
```

---

# 170. Phase 10 — Data View

Create:

```text
logs-*
```

or the organization's controlled index pattern.

Select:

```text
@timestamp
```

---

# 171. Phase 11 — Dashboards

Create:

```text
Log volume
Errors
Services
Namespaces
Pods
Versions
```

---

# 172. Phase 12 — Retention and Security

Configure:

```text
Lifecycle
Snapshots
RBAC
TLS
Authentication
Secret management
```

---

# 173. Phase 13 — Failure Testing

Test safely in non-production:

```text
Stop Logstash
Stop collector
Fill test index
Generate high log volume
Break Elasticsearch connectivity
Restart Kibana
```

Observe:

```text
Buffer
Retry
Alert
Recovery
Data loss
```

---

# 174. Phase 14 — DR

Test:

```text
Snapshot
Failure
Restore
Search
Dashboard recovery
```

Record:

```text
RTO
RPO
```

---

# 175. Repository Structure

A practical Git repository:

```text
elk-logging/
│
├── elasticsearch/
│   ├── values.yaml
│   └── policies/
│
├── logstash/
│   ├── pipelines/
│   │   ├── application.conf
│   │   └── infrastructure.conf
│   └── values.yaml
│
├── collector/
│   ├── daemonset.yaml
│   └── config.yaml
│
├── kibana/
│   ├── values.yaml
│   └── dashboards/
│
├── alerts/
│   └── logging-alerts.yaml
│
├── runbooks/
│   ├── elasticsearch-red.md
│   ├── disk-pressure.md
│   └── log-pipeline-delay.md
│
└── README.md
```

---

# 176. GitOps Deployment

For ArgoCD:

```text
Git
 |
 +--> Elasticsearch configuration
 +--> Logstash pipelines
 +--> Collector configuration
 +--> Kibana configuration
 |
 v
ArgoCD
 |
 v
EKS
```

This provides:

```text
Version control
Review
Audit
Rollback
Repeatability
```

---

# 177. CI Validation

Pipeline:

```text
Git commit
   |
   v
YAML validation
   |
   v
Helm lint
   |
   v
Manifest validation
   |
   v
Pipeline syntax validation
   |
   v
Deploy to test
   |
   v
Generate test logs
   |
   v
Verify Elasticsearch
   |
   v
Verify Kibana
```

---

# 178. Production Logging Architecture — Mature

```text
                    EKS Clusters
                /       |       \
               /        |        \
              v         v         v
         Collectors  Collectors  Collectors
              \         |         /
               \        |        /
                +-------+-------+
                        |
                  Durable Buffer
                        |
                        v
                   Logstash Tier
                 /      |       \
                /       |        \
               v        v         v
          Pipeline  Pipeline  Pipeline
               \       |        /
                +------+------+
                       |
                       v
               Elasticsearch
              /      |       \
             /       |        \
        Data Node Data Node Data Node
                       |
                       v
                    Kibana
                       |
                       v
                  Engineering
```

---

# 179. Enterprise Logging Considerations

At enterprise scale, consider:

```text
Kafka
Multiple Logstash pipelines
Multiple Elasticsearch tiers
Cross-region replication/backup
Object storage archive
Security analytics
SIEM integration
Dedicated audit pipelines
Data classification
Cost controls
```

The architecture should evolve based on actual requirements rather than adding every component by default.

---

# 180. Logging and Observability Integration

Prometheus/Grafana:

```text
Metrics
```

ELK:

```text
Logs
```

Combined:

```text
Metrics
   |
   +--> "Something is wrong"
              |
              v
            Logs
              |
              +--> "Why?"
```

Example:

```text
Prometheus:
5xx ↑

Kibana:
Database timeout errors ↑
```

---

# 181. Metrics + Logs Incident Workflow

```text
Alert
 |
 v
Grafana
 |
 v
Identify service
 |
 v
Kibana
 |
 v
Search errors
 |
 v
Identify version
 |
 v
Check deployment
 |
 v
Check dependency
 |
 v
Mitigate
```

This is a practical DevOps troubleshooting workflow.

---

# 182. Kubernetes Observability Architecture

```text
                    EKS
                     |
        +------------+------------+
        |                         |
        v                         v
    Prometheus                   ELK
        |                         |
        v                         v
     Metrics                    Logs
        |                         |
        +------------+------------+
                     |
                     v
               Engineers
```

Later projects can combine these with distributed tracing.

---

# 183. Interview Project Explanation — 30 Seconds

> "I designed centralized logging for an EKS microservices environment using Elasticsearch, Logstash and Kibana. Application containers wrote structured logs to stdout, a node-level collector collected Kubernetes container logs and forwarded them to Logstash for parsing and enrichment. Elasticsearch stored the structured events and Kibana provided search and dashboards. I focused on Kubernetes metadata, retention, buffering, Elasticsearch health, security and troubleshooting log pipeline failures."

---

# 184. Interview Project Explanation — 60 Seconds

> "For an EKS microservices platform, I implemented a centralized logging architecture where applications emitted structured JSON logs to stdout. A collector running as a DaemonSet collected container logs from each node and forwarded them to a Logstash processing tier. Logstash parsed and enriched the events with fields such as environment, namespace, pod, service and version before indexing them into Elasticsearch. Kibana was used for centralized search and operational dashboards. For production readiness I considered persistent Elasticsearch storage, shard and replica strategy, retention, snapshots, backpressure, Logstash queues, RBAC, TLS, sensitive-data redaction and monitoring the logging pipeline itself."

---

# 185. Interview Question — Why Centralized Logging?

### Answer

> "Kubernetes workloads are dynamic. Pods and nodes can disappear, so relying only on local container logs makes historical investigation difficult. Centralized logging preserves logs independently of individual pods and provides one searchable location across services and nodes."

---

# 186. Interview Question — Why Use a Collector?

### Answer

> "A node-level collector can efficiently read container logs and enrich them with Kubernetes metadata. It decouples application workloads from the central processing and storage systems."

---

# 187. Interview Question — Why DaemonSet?

### Answer

> "A DaemonSet ensures a collector runs on each eligible Kubernetes node. When the cluster scales and a new node is added, Kubernetes automatically schedules the collector there."

---

# 188. Interview Question — Why Logstash?

### Answer

> "Logstash provides a flexible processing layer for parsing, transformation, enrichment and routing. It is especially useful when logs come in different formats or require normalization before indexing."

---

# 189. Interview Question — Why Structured Logging?

### Answer

> "Structured logging gives us fields instead of requiring regex parsing for every event. It makes filtering, aggregation and incident investigation more reliable and usually reduces processing complexity."

---

# 190. Interview Question — What If Elasticsearch Goes Down?

### Answer

> "The collector and Logstash layers should provide buffering and retry according to the required durability. I would monitor queue growth, Elasticsearch health and indexing failures, restore Elasticsearch capacity, and then verify that buffered events are processed without unacceptable loss."

---

# 191. Interview Question — What If Logs Are Missing?

Answer:

```text
Application
   |
stdout
   |
collector
   |
Logstash
   |
Elasticsearch
   |
Kibana
```

> "I trace the event through each layer and identify the first point where it disappears."

---

# 192. Interview Question — How Do You Prevent Elasticsearch from Running Out of Disk?

Answer:

> "I monitor disk utilization and growth, implement retention and lifecycle policies, control index and shard design, maintain operational headroom, use snapshots for required retention and alert before disk pressure becomes critical."

---

# 193. Interview Question — How Do You Scale ELK?

Answer:

> "I first identify the bottleneck: collection, Logstash processing, Elasticsearch indexing, search or storage. Then I scale the affected layer independently where possible. At larger scale, a durable queue such as Kafka can decouple ingestion from processing, while Elasticsearch can be scaled with appropriate node roles and shard allocation."

---

# 194. Interview Question — How Do You Reduce Logging Costs?

Answer:

> "I start by reducing unnecessary log volume through appropriate log levels and removing duplicate or low-value events. Then I tune retention, event size, index strategy and storage tiers. I measure GB per day and identify the biggest producers before scaling infrastructure."

---

# 195. Interview Question — How Do You Secure ELK?

Answer:

> "I keep Elasticsearch private, use TLS, authentication and RBAC, protect credentials through secret management, restrict network access and implement redaction so sensitive data does not get indexed."

---

# 196. Interview Question — What Is the Difference Between Metrics and Logs?

### Metrics

```text
Numeric time series
Good for:
Health
Trends
Alerting
SLOs
```

### Logs

```text
Detailed events
Good for:
Debugging
Root cause
Context
Audit
```

Use both.

---

# 197. Interview Question — When Would You Use Metrics Instead of Logs?

Example:

```text
CPU > 90%
```

Metrics are better.

For:

```text
Database connection failed because timeout occurred
```

Logs provide more context.

---

# 198. Interview Question — What Is the Biggest ELK Risk?

A strong answer:

> "Uncontrolled log volume and Elasticsearch resource pressure. A logging platform can fail because applications suddenly produce huge volumes, Elasticsearch runs out of disk, or excessive shards and mappings consume resources. I therefore treat logging capacity, retention, cardinality, disk and backpressure as production concerns."

---

# 199. Interview Question — What Is a Logging SLO?

Example:

> "For critical application logs, we could define an objective such as 99.9% of events becoming searchable within 30 seconds. This makes the reliability of the observability platform measurable."

---

# 200. Interview Question — How Do You Correlate Logs Across Microservices?

Answer:

> "I propagate a request ID or trace ID across service boundaries and include it in structured logs. Then engineers can search that identifier in Kibana to reconstruct the request path."

---

# 201. Interview Question — How Do You Correlate a Deployment With Errors?

Answer:

> "I include application version or commit metadata in logs and correlate the timestamp with deployment events. If error volume changes immediately after a new version is deployed, I investigate that version and compare its behavior with the previous release."

---

# 202. Interview Question — Why Shouldn't Elasticsearch Be Public?

Answer:

> "Elasticsearch contains potentially sensitive operational and application data and provides powerful administrative APIs. Exposing it publicly increases the attack surface. Access should go through controlled authenticated interfaces such as Kibana or approved APIs."

---

# 203. Interview Question — What Happens When a Node Is Added?

Answer:

> "The collector DaemonSet schedules a collector on the new node, allowing container logs from workloads on that node to enter the centralized pipeline."

---

# 204. Interview Question — What Happens When a Pod Is Deleted?

Answer:

> "The local container log disappears with the workload eventually, but events already collected and indexed remain in centralized storage according to retention policy. That allows historical investigation after pod recreation."

---

# 205. Interview Question — Why Are Timestamps Important?

Answer:

> "Incident investigation depends on accurate timelines. If timestamps are inconsistent or incorrect, correlating application logs with deployments, metrics and dependency failures becomes unreliable."

---

# 206. Interview Question — How Would You Investigate a 2 TB Log Spike?

Answer:

```text
1. Identify top producer.
2. Check deployment timeline.
3. Check traffic.
4. Check log level.
5. Check repeated error.
6. Check attack/security indicators.
7. Protect Elasticsearch capacity.
8. Reduce unnecessary volume.
9. Preserve critical logs.
10. Fix source.
```

---

# 207. Interview Question — How Would You Handle a Logstash Bottleneck?

Answer:

```text
Measure events in/out
        |
        v
Check queue
        |
        v
Check CPU/memory
        |
        v
Check filter complexity
        |
        v
Check Elasticsearch throughput
        |
        v
Scale or optimize
```

---

# 208. Interview Question — How Would You Handle Elasticsearch Red State?

Answer:

> "I would immediately inspect cluster health and failed primary shards, then check node availability, disk watermarks, persistent volumes and recent changes. I would avoid destructive actions until I understand the failed allocation and protect data before attempting recovery."

---

# 209. Interview Question — How Would You Design Logging for 100+ Microservices?

Answer:

> "I would standardize structured logging, deploy collectors as DaemonSets, enrich events with Kubernetes metadata, decouple collection from processing, scale Logstash horizontally, use a durable queue if required, design Elasticsearch indices and lifecycle policies carefully, centralize Kibana access and enforce security and ownership."

---

# 210. Interview Question — How Would You Prevent One Team from Flooding the Cluster?

Possible controls:

```text
Application log standards
Per-team monitoring
Ingestion limits
Rate controls
Log level policies
Capacity alerts
```

The exact control should be implemented at the layer where the organization has the most reliable enforcement.

---

# 211. Interview Question — How Do Logs Help Kubernetes Troubleshooting?

Example:

```text
Prometheus:
Pod restart count ↑

Kibana:
Application ERROR
"Database connection timeout"
```

Together:

```text
Metric = symptom
Log = context
```

---

# 212. Interview Question — What Would You Improve Later?

Possible improvements:

```text
Durable Kafka buffer
Long-term archive
Automated PII detection
Better SLOs
Cross-cluster logging
Security analytics
Trace correlation
Automated incident enrichment
```

Choose improvements according to scale and business requirements.

---

# 213. Production Readiness Scorecard

## Collection

```text
[ ] DaemonSet
[ ] Metadata
[ ] Buffering
[ ] Retry
[ ] Resource limits
```

## Processing

```text
[ ] Structured parsing
[ ] Timestamp normalization
[ ] Field normalization
[ ] Redaction
[ ] Failure handling
```

## Storage

```text
[ ] Stateful architecture
[ ] Persistent storage
[ ] Replicas
[ ] Shards
[ ] Lifecycle
[ ] Snapshots
[ ] Disk monitoring
```

## Access

```text
[ ] Kibana authentication
[ ] RBAC
[ ] TLS
[ ] Network controls
```

## Operations

```text
[ ] Alerts
[ ] Runbooks
[ ] SLO
[ ] Capacity planning
[ ] DR testing
```

---

# 214. End-to-End Troubleshooting Mental Model

When a log is missing:

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
Node log
    |
    v
Collector
    |
    v
Buffer
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

At each stage ask:

```text
Is the event present?
Is it delayed?
Is it being rejected?
Is it being dropped?
Is it being transformed incorrectly?
```

---

# 215. Production Logging Mental Model

Think in four layers:

```text
Collection
    |
    v
Processing
    |
    v
Storage
    |
    v
Access
```

And four operational dimensions:

```text
Reliability
Security
Performance
Cost
```

A production logging design must balance all four.

---

# 216. Final Project Validation

```text
[ ] EKS healthy
[ ] Logging namespace created
[ ] Elasticsearch deployed
[ ] Persistent storage configured
[ ] Elasticsearch health verified
[ ] Logstash deployed
[ ] Collector deployed as DaemonSet
[ ] Test logs generated
[ ] Logs collected
[ ] Logs parsed
[ ] Kubernetes metadata added
[ ] Logs indexed
[ ] Kibana connected
[ ] Data view created
[ ] Dashboards created
[ ] Retention configured
[ ] Disk monitoring configured
[ ] Snapshot strategy configured
[ ] RBAC configured
[ ] TLS configured
[ ] Secrets protected
[ ] Backpressure tested
[ ] Failure scenarios tested
[ ] DR tested
[ ] Configuration stored in Git
```

---

# 217. Project Summary

The completed architecture is:

```text
                  EKS
                   |
          Application Pods
                   |
            stdout/stderr
                   |
                   v
             Collector
             DaemonSet
                   |
                   v
               Logstash
          +--------+--------+
          |                 |
       Parsing          Enrichment
          |                 |
          +--------+--------+
                   |
                   v
             Elasticsearch
                   |
                   v
                 Kibana
                   |
                   v
              Engineers
```

The key production principle is:

> Applications should remain independent of the logging platform. Logging must collect and preserve useful operational context without becoming a dependency in the application request path.

---

# 218. Relationship to the Previous Project

Previous project:

```text
01-Prometheus-Grafana-EKS
```

provided:

```text
Metrics
Dashboards
Alerts
SLO signals
```

This project provides:

```text
Logs
Search
Context
Historical investigation
```

Combined:

```text
                 Production EKS
                       |
          +------------+------------+
          |                         |
          v                         v
     Prometheus                    ELK
          |                         |
       Metrics                     Logs
          |                         |
          +------------+------------+
                       |
                       v
                 Troubleshooting
```

Example:

```text
Prometheus:
payment error rate = 12%

Kibana:
payment-service
ERROR
database timeout
version=2026.08.15
```

Metrics tell you **that** something is wrong.

Logs help explain **what happened and where to investigate**.

---

# 219. Next Project

The next file is:

```text
03-OpenTelemetry-Jaeger-Tracing.md
```

It will build a distributed tracing project for the same kind of microservices environment and cover:

```text
OpenTelemetry
SDKs
Instrumentation
Trace context
Spans
Collectors
Jaeger
Kubernetes
EKS
Sampling
Trace storage
Trace-to-log correlation
Trace-to-metrics correlation
Production scaling
Security
Troubleshooting
Interview scenarios
```

This will complete the tracing layer before combining metrics, logs and traces in:

```text
04-Full-Stack-Observability.md
```
