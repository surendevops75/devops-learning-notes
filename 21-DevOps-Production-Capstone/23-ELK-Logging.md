# 23 — ELK Logging — Production DevOps Capstone

> Deep production-focused chapter for centralized logging with Elasticsearch, Logstash, Kibana, and Kubernetes log collectors in AWS/EKS environments.

## Chapter Objective

A production logging platform must allow engineers to answer:

- What happened?
- Which service generated the event?
- Which pod/node/container produced it?
- When did it happen?
- How widespread is the problem?
- What changed immediately before it?
- Can the event be correlated with metrics and traces?
- Can the organization retain the required evidence?
- Can logs be searched safely and cost-effectively?

Logging is therefore part of the production reliability architecture, not simply a place to store application text.

---

# 1. Logging Architecture

A common EKS architecture is:

```text
Applications
    |
    v
stdout / stderr
    |
    v
Container Runtime
    |
    v
Node Log Files
    |
    v
Fluent Bit / Fluentd
    |
    +----> Parse / Enrich / Filter
    |
    v
Logstash
    |
    +----> Transform / Route
    |
    v
Elasticsearch
    |
    v
Kibana
    |
    v
DevOps / SRE / Security
```

An alternative architecture may send Fluent Bit directly to Elasticsearch, while Logstash is introduced only when advanced processing or routing is required.

---

# 2. Why Centralized Logging Matters

Without centralized logging:

```text
Engineer
   |
   +--> kubectl logs pod-a
   +--> kubectl logs pod-b
   +--> kubectl logs pod-c
```

With centralized logging:

```text
Engineer
   |
   v
Kibana
   |
   +--> cluster
   +--> namespace
   +--> service
   +--> pod
   +--> container
   +--> request ID
   +--> trace ID
   +--> severity
```

Centralization is especially important when pods are ephemeral.

---

# 3. Logging Requirements

A production logging system should provide:

- centralized collection
- structured logs
- metadata enrichment
- reliable delivery
- search
- filtering
- retention
- access control
- encryption
- auditing
- alerting
- backup/recovery strategy
- cost controls
- schema governance

---

# 4. Application Logging Strategy

Prefer structured JSON.

Example:

```json
{
  "timestamp": "2026-08-31T10:20:31.123Z",
  "level": "ERROR",
  "service": "checkout",
  "environment": "production",
  "message": "payment request failed",
  "request_id": "abc-123",
  "trace_id": "trace-456",
  "status_code": 502
}
```

Avoid unstructured messages such as:

```text
payment failed for customer 123
```

when structured fields could provide the same operational information more safely.

---

# 5. Never Log Secrets

Never intentionally log:

```text
Passwords
API keys
Access tokens
Session tokens
Private keys
Database credentials
Authorization headers
Full credit-card numbers
Sensitive personal information
```

Logging a secret can turn an application incident into a security incident.

---

# 6. Kubernetes Logging

Kubernetes applications commonly write logs to:

```text
stdout
stderr
```

The container runtime and node logging configuration make those logs available to collection agents.

Typical path:

```text
Application
   |
 stdout/stderr
   |
 container runtime
   |
 node log files
   |
 Fluent Bit
```

---

# 7. DaemonSet Collector

A common EKS design runs one Fluent Bit pod per node.

```text
Node 1                  Node 2
+-----------+           +-----------+
| App Pods  |           | App Pods  |
|           |           |           |
| FluentBit |           | FluentBit |
+-----------+           +-----------+
       |                       |
       +-----------+-----------+
                   |
                   v
                Backend
```

A DaemonSet ensures the collector is scheduled across nodes.

---

# 8. Why Fluent Bit

Fluent Bit is commonly used because it is:

- lightweight
- Kubernetes-friendly
- fast
- extensible
- capable of filtering
- capable of metadata enrichment
- suitable for node-level collection

It is often preferred as the first-stage collector in EKS.

---

# 9. Fluent Bit vs Fluentd

Fluent Bit:

- lightweight
- written in C
- low resource overhead
- strong fit for node agents

Fluentd:

- richer processing ecosystem
- Ruby-based
- generally heavier

A common architecture is:

```text
Fluent Bit
    |
    v
Logstash
    |
    v
Elasticsearch
```

But Logstash is not mandatory if Fluent Bit can perform the required processing and output.

---

# 10. Why Logstash

Logstash is useful for:

- complex parsing
- transformation
- routing
- enrichment
- conditional processing
- multiple outputs
- advanced filtering

Example:

```text
Fluent Bit
    |
    v
Logstash
    |
    +--> Elasticsearch
    +--> Security destination
    +--> Archive
```

---

# 11. Why Elasticsearch

Elasticsearch provides distributed indexing and search.

It is suitable for:

- full-text search
- structured field search
- aggregations
- log analytics
- high-volume indexed data

It should be operated as a production datastore with capacity planning, lifecycle management, security, and recovery procedures.

---

# 12. Why Kibana

Kibana provides:

- log search
- dashboards
- visualizations
- Discover
- aggregations
- operational investigation

A typical investigation starts in Kibana and then correlates findings with Grafana metrics and traces.

---

# 13. End-to-End Logging Flow

```text
Application
    |
    v
stdout
    |
    v
Container Runtime
    |
    v
Node log
    |
    v
Fluent Bit
    |
    +--> Parse JSON
    +--> Add Kubernetes metadata
    +--> Drop unwanted events
    +--> Add environment
    |
    v
Logstash
    |
    +--> Normalize
    +--> Route
    +--> Enrich
    |
    v
Elasticsearch
    |
    v
Kibana
```

---

# 14. Kubernetes Metadata

Enrich every event where practical with:

```text
cluster
environment
region
namespace
pod
container
node
workload
container_id
```

This allows:

```text
namespace="payments"
AND
pod="checkout-7d9..."
```

instead of searching blindly through raw text.

---

# 15. Application Metadata

Useful fields include:

```text
service
version
environment
request_id
trace_id
span_id
method
route
status_code
duration_ms
level
logger
```

Do not add high-cardinality fields without understanding their indexing impact.

---

# 16. Correlation IDs

A request may travel:

```text
ALB
 |
 v
API
 |
 v
Checkout
 |
 v
Payment
 |
 v
Database
```

A common request ID allows engineers to correlate logs across services.

---

# 17. Trace Correlation

Where distributed tracing exists, include:

```text
trace_id
span_id
```

Example:

```json
{
  "service": "checkout",
  "level": "ERROR",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "message": "payment timeout"
}
```

Kibana can then help locate the event while the tracing platform shows the request path.

---

# 18. Log Levels

Common levels:

```text
TRACE
DEBUG
INFO
WARN
ERROR
FATAL
```

Production defaults should normally avoid DEBUG-level volume unless deliberately enabled for troubleshooting.

---

# 19. Log Level Strategy

Use:

```text
INFO  normal lifecycle events
WARN  unusual but recoverable condition
ERROR request/operation failure
FATAL process-level critical failure
DEBUG temporary diagnostic detail
```

Do not use ERROR for every expected validation failure.

---

# 20. Logging Anti-Patterns

Avoid:

```text
Logging entire request bodies
Logging secrets
Logging stack traces repeatedly
Logging inside tight loops
Generating millions of duplicate INFO messages
Using inconsistent timestamp formats
Using inconsistent severity names
```

---

# 21. Elasticsearch Architecture

A production cluster conceptually contains:

```text
                Load Balancer / Service
                        |
            +-----------+-----------+
            |           |           |
          Node-1      Node-2      Node-3
            |           |           |
            +-----------+-----------+
                        |
                 Cluster State
```

Production topology depends on workload, version, shard requirements, failure domains, and the selected deployment model.

---

# 22. Elasticsearch Roles

Modern Elasticsearch supports multiple node roles.

Conceptually:

```text
Cluster Manager Eligible
Data
Ingest
Coordinating
```

For larger environments, separating responsibilities may improve operational isolation.

For smaller clusters, role consolidation may be appropriate.

---

# 23. Elasticsearch Cluster Manager

Cluster-manager-eligible nodes participate in cluster state management.

A production design should provide enough eligible nodes to tolerate the intended failure scenario.

Avoid designing a cluster around a single control-plane node.

---

# 24. Data Nodes

Data nodes store indexed documents and execute searches and aggregations.

Sizing depends on:

```text
Ingest rate
Retention
Shard count
Query rate
Document size
Aggregation complexity
Storage type
Replication
```

---

# 25. Ingest Nodes

Ingest nodes can execute pipelines before documents are indexed.

Use ingest processing when it is operationally appropriate.

Heavy processing may instead belong in Logstash or the application.

---

# 26. Coordinating Nodes

Coordinating nodes route requests to data nodes and combine responses.

They can be useful in large clusters with significant query traffic.

Do not introduce additional roles without a demonstrated operational need.

---

# 27. Elasticsearch Index

An index is a logical collection of documents.

For logs:

```text
logs-application-2026.08.31
```

or data-stream-oriented designs may be used.

The naming strategy should support lifecycle management and ownership.

---

# 28. Shards

An index is divided into shards.

```text
Index
 |
 +--> Primary shard 0
 +--> Primary shard 1
 +--> Primary shard 2
```

Replicas provide additional copies.

---

# 29. Shard Count

More shards are not automatically better.

Too many shards cause:

- memory pressure
- cluster-state overhead
- slower operations
- more file handles
- more management overhead

Shard sizing should be based on observed workload and supported operational guidance.

---

# 30. Replica Shards

Example:

```text
Primary P0 --> Replica R0
Primary P1 --> Replica R1
Primary P2 --> Replica R2
```

Replicas provide resilience and can improve read capacity.

---

# 31. Failure Domains

Do not place all copies in one failure domain when availability requirements demand cross-domain resilience.

In AWS this commonly means distributing nodes across multiple Availability Zones.

---

# 32. Elasticsearch Storage

Log workloads are storage-intensive.

Consider:

```text
IOPS
throughput
latency
capacity
growth rate
replication
retention
snapshot overhead
```

Fast storage can improve indexing and query performance.

---

# 33. JVM Heap

Elasticsearch uses JVM heap plus filesystem cache.

Do not simply maximize JVM heap.

A production sizing exercise should account for:

```text
Heap
OS memory
Filesystem cache
Elasticsearch version
Workload
Shard count
```

---

# 34. Elasticsearch Memory Pressure

Symptoms include:

```text
Long GC pauses
Circuit breaker exceptions
Slow queries
Rejected requests
Node instability
Cluster instability
```

Investigate heap, shard count, query patterns, and indexing pressure together.

---

# 35. Logstash Architecture

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
Beats / Fluent Bit
       |
       v
    Logstash
       |
       +--> JSON parsing
       +--> Field normalization
       +--> PII filtering
       +--> Routing
       |
       v
 Elasticsearch
```

---

# 36. Logstash Input

Possible inputs include:

```text
Beats
TCP
HTTP
Kafka
Files
Cloud sources
```

Choose the input based on architecture and reliability requirements.

---

# 37. Logstash Filter

Typical filters:

```text
json
grok
mutate
date
drop
rename
convert
```

Use filters deliberately because complex pipelines consume CPU.

---

# 38. JSON Filter

If application logs are JSON:

```text
raw message
   |
   v
JSON parser
   |
   v
structured fields
```

Structured logging is usually preferable to using large numbers of regex patterns.

---

# 39. Grok

Grok can parse text patterns.

Example conceptual input:

```text
2026-08-31 ERROR checkout payment failed
```

The pipeline can extract:

```text
timestamp
level
service
message
```

Use grok selectively; poorly designed patterns can be expensive.

---

# 40. Logstash Conditional Routing

Example logic:

```text
if environment == "production"
    -> production index

if service == "security"
    -> security destination
```

Routing rules should be version controlled and tested.

---

# 41. Backpressure

A production logging pipeline must handle temporary downstream slowdown.

```text
Application
   |
Collector
   |
Buffer
   |
Logstash
   |
Elasticsearch
```

Without buffering or flow control, downstream failure can cause data loss or resource exhaustion.

---

# 42. Buffering

Possible buffering layers:

```text
Fluent Bit filesystem buffering
Logstash persistent queues
Kafka
Object storage archive
```

The right choice depends on reliability and scale requirements.

---

# 43. Logstash Persistent Queue

Persistent queues can protect events when downstream systems temporarily fail.

Consider:

```text
queue size
disk capacity
recovery time
retention
failure scenarios
```

Do not enable a persistent queue without planning its storage requirements.

---

# 44. Kafka as Logging Buffer

At larger scale:

```text
Fluent Bit
    |
    v
Kafka
    |
    v
Logstash
    |
    v
Elasticsearch
```

Advantages:

- buffering
- replay
- decoupling
- multiple consumers

Trade-offs:

- additional infrastructure
- operational complexity
- storage requirements

---

# 45. Elasticsearch Bulk Indexing

Log processors should generally use bulk ingestion rather than one request per document.

Bulk ingestion improves throughput but must be tuned carefully.

---

# 46. Ingestion Rate

Track:

```text
events/sec
bytes/sec
documents/sec
bulk request latency
bulk failures
queue depth
```

---

# 47. Logging Backpressure

If Elasticsearch slows:

```text
Elasticsearch
   |
   v
Logstash queue grows
   |
   v
Fluent Bit buffer grows
   |
   v
Node disk usage grows
```

Monitor every stage.

---

# 48. Data Loss Scenarios

Potential loss can occur when:

```text
collector crashes
node disappears
buffer fills
Logstash queue fills
Elasticsearch rejects writes
network fails
```

A production design must explicitly decide what loss is acceptable.

---

# 49. At-Least-Once Delivery

Logging systems commonly favor at-least-once behavior.

This can produce duplicates during retries.

Consumers and indexing strategies should tolerate or manage duplicate events where necessary.

---

# 50. Exactly-Once Logging

Do not assume exactly-once delivery across a distributed logging pipeline.

Instead design for:

```text
duplicates
retries
partial failures
replay
```

---

# 51. Elasticsearch Refresh

Documents may not become searchable immediately after indexing.

This is normal.

Do not build operational assumptions around zero-latency search unless the architecture explicitly requires and supports it.

---

# 52. Index Lifecycle Management

Logging retention should be automated.

Typical lifecycle:

```text
Hot
 |
 v
Warm
 |
 v
Cold
 |
 v
Delete
```

Exact tiers depend on the Elasticsearch architecture and storage strategy.

---

# 53. Hot Phase

Hot data is frequently searched.

Use faster storage and adequate resources.

Typical use:

```text
Recent hours/days
Incident investigation
Active operations
```

---

# 54. Warm Phase

Warm data is less frequently queried.

It can use lower-cost storage where supported.

---

# 55. Cold / Frozen Strategy

Older data may be moved to cheaper storage or searchable snapshots depending on requirements.

The design should consider:

```text
Search frequency
Recovery time
Compliance
Cost
```

---

# 56. Retention

Example policy:

```text
Production application logs: 14–30 days
Security logs: organization-defined
Audit logs: organization-defined
Debug logs: short retention
```

Never choose retention only because storage is available. Define it from operational, legal, security, and cost requirements.

---

# 57. Index Lifecycle Example

```text
Day 0–7     HOT
Day 8–30    WARM
Day 31–90   COLD / ARCHIVE
Day 91+     DELETE
```

This is only an example; actual retention must be approved.

---

# 58. Data Streams

For high-volume time-series-style logs, data streams can simplify lifecycle management.

They work particularly well when documents are append-oriented and time-based.

---

# 59. Index Templates

Use templates to enforce:

```text
mappings
settings
aliases
lifecycle policies
```

This prevents every index from evolving differently.

---

# 60. Field Mappings

Common types:

```text
keyword
text
date
integer
long
double
boolean
ip
```

Choose mappings deliberately.

---

# 61. Keyword vs Text

Use `keyword` for exact matching and aggregation:

```text
service.keyword
environment.keyword
status.keyword
```

Use `text` for full-text search.

A field may have both forms depending on the mapping.

---

# 62. Mapping Explosion

Uncontrolled dynamic fields can create huge numbers of mappings.

Example:

```text
request.headers.*
```

If arbitrary header names become fields, mappings can grow rapidly.

Prefer controlled schemas.

---

# 63. High Cardinality

Fields such as:

```text
request_id
session_id
user_id
```

may have extremely high cardinality.

They can be useful for search but may be poor choices for aggregations or dashboards.

---

# 64. PII Protection

Apply data minimization.

Options include:

```text
Do not collect
Redact
Hash
Mask
Drop
Restrict access
```

The best sensitive data is often data that never enters the logging system.

---

# 65. PII Redaction

Example:

```text
john@example.com
```

may become:

```text
[REDACTED_EMAIL]
```

depending on requirements.

Do not depend solely on Kibana permissions after sensitive data has already been indexed.

---

# 66. Security Architecture

```text
Users
  |
  v
Kibana Authentication
  |
  v
RBAC
  |
  v
Elasticsearch
  |
  +--> TLS
  +--> Encryption
  +--> Audit
```

---

# 67. TLS

Use TLS for:

```text
User -> Kibana
Kibana -> Elasticsearch
Logstash -> Elasticsearch
Collector -> Logstash
```

where the architecture requires those network paths.

---

# 68. Authentication

Use centralized identity where practical.

Examples include:

```text
OIDC
SAML
LDAP
Cloud identity integration
```

Avoid shared administrator credentials.

---

# 69. Authorization

Apply least privilege.

Example:

```text
Application Team
  -> application logs

Platform Team
  -> platform logs

Security Team
  -> security logs
```

---

# 70. Kubernetes Secrets

Do not place Elasticsearch credentials directly in:

```text
ConfigMap
Git
Dashboard JSON
Container command line
```

Use Kubernetes Secrets integrated with an appropriate external secret-management solution where possible.

---

# 71. NetworkPolicy

Restrict:

```text
Fluent Bit -> Logstash
Logstash -> Elasticsearch
Kibana -> Elasticsearch
Users -> Kibana
```

Do not permit unrestricted access.

---

# 72. Elasticsearch Authentication

Use service credentials with narrowly scoped permissions.

Avoid using a superuser account for Logstash.

---

# 73. Snapshot Strategy

Snapshots protect against:

```text
cluster failure
operator error
data corruption
accidental deletion
```

Snapshots should be stored independently of the primary cluster.

---

# 74. Snapshot Repository

A common AWS architecture uses object storage.

Conceptually:

```text
Elasticsearch
     |
     v
Snapshot Repository
     |
     v
S3 / compatible object storage
```

Use appropriate IAM permissions and encryption.

---

# 75. Disaster Recovery

A DR strategy should define:

```text
RPO
RTO
Snapshot frequency
Retention
Restore procedure
Infrastructure recreation
DNS / endpoint recovery
Credential recovery
Validation
```

---

# 76. Restore Testing

A backup is not proven until restored.

Test:

```text
Create recovery environment
Restore snapshot
Validate cluster health
Validate indexes
Search known events
Validate mappings
Validate application access
Measure recovery duration
```

---

# 77. Kibana Dashboards

Useful dashboards include:

```text
Application Errors
HTTP Status Codes
Top Error Messages
Logs by Service
Logs by Namespace
Logs by Pod
Security Events
Authentication Failures
Deployment Events
Infrastructure Logs
```

---

# 78. Kibana Discover

Discover is ideal for interactive investigation.

Typical workflow:

```text
Select time range
   |
Filter cluster
   |
Filter namespace
   |
Filter service
   |
Filter severity
   |
Search request_id
   |
Inspect event
```

---

# 79. KQL

Kibana Query Language can provide readable filters.

Conceptual examples:

```text
service : "checkout"
```

```text
level : "ERROR"
```

```text
namespace : "payments" AND level : "ERROR"
```

Use the syntax supported by the deployed Kibana version.

---

# 80. Search by Trace ID

Example concept:

```text
trace_id : "4bf92f3577b34da6a3ce929d0e0e4736"
```

This can quickly identify all logs associated with a trace.

---

# 81. Search by Request ID

Example:

```text
request_id : "abc-123"
```

This is particularly useful when tracing is not available.

---

# 82. Error Dashboard

Recommended panels:

```text
Errors/sec
Errors by service
Errors by namespace
Top error messages
HTTP 5xx
HTTP 4xx
Recent deployment
Error trend
```

---

# 83. Logging Alerts

Useful alerts:

```text
Elasticsearch cluster unhealthy
Disk usage high
Indexing failures
Collector down
Logstash queue growing
No logs from critical service
Authentication failures
Error volume spike
```

Avoid alerting on every individual application ERROR.

---

# 84. Missing Logs Alert

A powerful operational check is:

```text
Critical service expected logs
        |
        v
No events for N minutes
        |
        v
Investigate collector/application/backend
```

The threshold must account for low-traffic services.

---

# 85. Log Volume Spike

Alerting can detect sudden increases.

Example concept:

```text
current log rate
        /
baseline log rate
```

A huge increase may indicate:

- application failure loop
- traffic spike
- retry storm
- bad logging configuration

---

# 86. Elasticsearch Health

Monitor:

```text
cluster health
node availability
disk usage
heap
GC
CPU
memory
JVM pressure
indexing rate
search rate
rejections
thread pools
```

---

# 87. Cluster Health

A production cluster should not remain in a degraded state unnoticed.

Monitor:

```text
green
yellow
red
```

Understand that the exact meaning depends on primary/replica allocation.

---

# 88. Disk Watermarks

Elasticsearch uses disk allocation thresholds.

If disk usage becomes too high:

```text
allocation can change
writes may be blocked
cluster may become unstable
```

Disk monitoring is therefore critical.

---

# 89. Thread Pools

Watch for rejected operations.

Examples:

```text
search rejections
write rejections
bulk rejections
```

Rejections indicate the cluster cannot keep up with incoming work.

---

# 90. Indexing Failures

Track:

```text
bulk failures
mapping failures
rejected writes
pipeline errors
authentication failures
```

A healthy application does not guarantee successful log ingestion.

---

# 91. Logstash Monitoring

Track:

```text
events in
events out
pipeline throughput
queue size
persistent queue
CPU
memory
GC
output failures
```

---

# 92. Fluent Bit Monitoring

Track:

```text
input records
output records
retry count
buffer size
dropped records
CPU
memory
filesystem buffer
```

---

# 93. Collector Resource Requests

Fluent Bit must have enough resources to handle node log volume.

If under-sized:

```text
CPU throttling
memory pressure
buffer growth
dropped records
```

---

# 94. Collector DaemonSet Example

Illustrative:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluent-bit
  namespace: logging
spec:
  selector:
    matchLabels:
      app: fluent-bit
  template:
    metadata:
      labels:
        app: fluent-bit
    spec:
      serviceAccountName: fluent-bit
      containers:
        - name: fluent-bit
          image: fluent/fluent-bit:<pinned-version>
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
```

The actual image, mounts, configuration, RBAC, and output must match the production design.

---

# 95. Fluent Bit Configuration Concepts

A collector commonly contains:

```text
[SERVICE]
[INPUT]
[FILTER]
[OUTPUT]
```

Example conceptual flow:

```text
INPUT: tail container logs
       |
FILTER: Kubernetes metadata
       |
FILTER: JSON parser
       |
FILTER: remove sensitive fields
       |
OUTPUT: Logstash
```

---

# 96. Tail Input

A typical collector tails Kubernetes container log files.

Important settings include:

```text
path
parser
tag
database/state file
buffer
mem_buf_limit
```

State tracking helps prevent unnecessary rereading after restart.

---

# 97. Filesystem Buffering

For important logs, filesystem buffering can provide greater resilience than memory-only buffering.

Trade-offs include:

```text
disk consumption
write I/O
recovery behavior
node disk pressure
```

---

# 98. Log Rotation

Container logs are rotated by the runtime/platform configuration.

The collector must be compatible with the rotation behavior.

Otherwise you can encounter:

```text
missing events
duplicate events
stale file handles
```

---

# 99. Kubernetes Metadata Filter

The metadata filter can enrich records with:

```text
namespace_name
pod_name
container_name
container_id
labels
annotations
```

Use only metadata required for operations.

---

# 100. Label Explosion

Do not blindly index every Kubernetes label and annotation.

Applications may create arbitrary labels such as:

```text
build_id
timestamp
random_id
request_hash
```

This can create unnecessary fields and cardinality.

---

# 101. Drop Unnecessary Logs

Drop or reduce:

```text
health-check noise
debug logs
known repetitive messages
non-actionable events
```

Do this carefully; do not remove evidence needed for incident investigation or compliance.

---

# 102. Health Check Logs

Endpoints such as:

```text
GET /health
GET /ready
```

can generate huge log volume.

Options:

```text
disable access logging for health endpoints
sample
filter
aggregate
```

---

# 103. Sampling

Sampling can reduce volume for high-frequency low-value logs.

However:

```text
security logs
audit logs
critical failure logs
```

may require full capture.

Sampling must be policy-driven.

---

# 104. Log Compression

Compression can reduce network and storage usage.

Consider CPU cost versus bandwidth/storage savings.

---

# 105. Time Synchronization

Accurate timestamps are critical.

Ensure:

```text
nodes
containers
applications
Elasticsearch
Logstash
```

have reliable time synchronization.

Incorrect clocks can make distributed incident timelines misleading.

---

# 106. Timestamp Normalization

Use a consistent standard such as UTC.

Example:

```text
2026-08-31T10:20:31.123Z
```

Do not mix local time zones across services.

---

# 107. Parsing Failures

If JSON parsing fails, preserve the original message.

A robust pipeline should not silently discard the event merely because parsing failed.

Useful fields:

```text
message
parse_error
raw_message
```

---

# 108. Dead-Letter Logging

Malformed events can be routed separately:

```text
Collector
   |
   v
Parser
   |
   +--> valid --> Elasticsearch
   |
   +--> invalid --> dead-letter stream
```

This allows investigation without blocking the main pipeline.

---

# 109. Schema Governance

Define required fields:

```text
@timestamp
level
service
environment
message
cluster
namespace
pod
```

Optional fields:

```text
request_id
trace_id
status_code
duration_ms
error_type
```

---

# 110. ECS Consideration

Elastic Common Schema can standardize fields across logs.

Benefits:

- consistent search
- reusable dashboards
- easier correlation
- common field names

Use a compatible schema strategy rather than inventing different names for each service.

---

# 111. Schema Versioning

When changing logging fields:

```text
old field
new field
migration period
dashboard compatibility
```

Do not break all dashboards by changing field names without coordination.

---

# 112. Production Logging Repository

Recommended:

```text
logging/
├── fluent-bit/
│   ├── daemonset.yaml
│   ├── configmap.yaml
│   ├── rbac.yaml
│   └── serviceaccount.yaml
├── logstash/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── pipeline.conf
│   └── configmap.yaml
├── elasticsearch/
│   ├── values.yaml
│   ├── policies/
│   └── templates/
├── kibana/
│   ├── values.yaml
│   └── dashboards/
└── README.md
```

---

# 113. GitOps Logging Deployment

```text
Git
 |
 v
CI
 |
 +--> YAML validation
 +--> Helm lint
 +--> Policy checks
 +--> Security checks
 |
 v
Argo CD
 |
 +--> Fluent Bit
 +--> Logstash
 +--> Elasticsearch
 +--> Kibana
```

---

# 114. Environment Strategy

Separate:

```text
dev
stage
production
```

Use environment-specific:

```text
retention
capacity
replicas
storage
access policies
```

Do not copy production credentials into lower environments.

---

# 115. Production Logging Security

Checklist:

```text
TLS
RBAC
NetworkPolicy
Secret management
Audit logging
Encryption at rest
Encryption in transit
Least privilege
PII filtering
Retention policy
```

---

# 116. IAM Strategy on AWS

For AWS resources, prefer:

```text
IAM roles
IRSA / workload identity mechanisms
short-lived credentials
least privilege
```

Avoid static AWS access keys inside pods.

---

# 117. EKS Logging Architecture

```text
                EKS
+-------------------------------------+
| Node                                |
|                                     |
| App --> stdout --> container logs   |
|                       |             |
|                   Fluent Bit        |
+-----------------------|-------------+
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

For AWS-native alternatives, CloudWatch or OpenSearch may be selected depending on organizational requirements.

---

# 118. Multi-Cluster Logging

For multiple EKS clusters:

```text
Cluster A --\
Cluster B ----> Central Logging Platform
Cluster C --/
```

Every event should carry:

```text
cluster
environment
region
account
```

This prevents ambiguity.

---

# 119. Multi-Account Logging

Example:

```text
Dev Account ----\
Stage Account ----> Central Logging
Prod Account ----/
```

Use cross-account IAM and network controls according to organizational architecture.

---

# 120. Centralized Logging Trade-Off

Benefits:

```text
single search
cross-service correlation
central operations
central security analysis
```

Risks:

```text
blast radius
network dependency
central capacity bottleneck
cross-account complexity
```

Design for failure and capacity.

---

# 121. Tenant Isolation

If teams share Elasticsearch/Kibana:

```text
Team A -> Application A indices
Team B -> Application B indices
Security -> Security indices
```

Use roles and index-level privileges.

---

# 122. Cost Model

Logging cost comes from:

```text
collection
network
processing
indexing
storage
replication
queries
snapshots
```

The cheapest event is the event you do not generate unnecessarily.

---

# 123. Cost Optimization

Use:

```text
appropriate retention
log filtering
sampling
compression
lifecycle tiers
right-sized shards
controlled replicas
efficient queries
```

Do not sacrifice required evidence simply to reduce cost.

---

# 124. Query Cost

Expensive searches include:

```text
huge time range
wildcard-heavy searches
high-cardinality aggregations
unbounded regex
large result sets
```

Train engineers to search narrowly first.

---

# 125. Kibana Investigation Strategy

Start with:

```text
Time range
+
Service
+
Environment
+
Severity
```

Then narrow:

```text
Namespace
Pod
Request ID
Trace ID
Error type
```

Avoid immediately searching the entire cluster for millions of events.

---

# 126. Incident Workflow

```text
Alert in Grafana
      |
      v
Identify service
      |
      v
Open Kibana
      |
      v
Filter time window
      |
      v
Filter service
      |
      v
Filter ERROR
      |
      v
Find request/trace ID
      |
      v
Correlate deployment
      |
      v
Check metrics/traces
      |
      v
Root cause
```

---

# 127. Incident Example — HTTP 500 Spike

Suppose:

```text
checkout 5xx = 8%
```

Investigation:

```text
Grafana
  |
  +--> Error spike
  +--> Deployment at 10:20
  |
  v
Kibana
  |
  +--> service=checkout
  +--> level=ERROR
  |
  v
Exception found
  |
  v
Trace ID
  |
  v
Database timeout
```

Now inspect database metrics and dependency health.

---

# 128. Incident Example — Pods Restarting

```text
Grafana
  |
  v
Restart spike
  |
  v
Kibana
  |
  v
Search pod
  |
  v
OOM / exception / termination reason
```

Correlate with Kubernetes events and resource metrics.

---

# 129. Incident Example — Missing Logs

Symptoms:

```text
Application appears healthy
Kibana shows no new logs
```

Investigate:

```text
Application stdout
Node log files
Fluent Bit pod
Fluent Bit input metrics
Fluent Bit buffers
Network
Logstash
Elasticsearch
Index
Kibana datasource
```

---

# 130. Incident Example — Elasticsearch Red

Investigate:

```text
Cluster health
Unassigned shards
Node availability
Disk
Allocation explanations
Network
Resource pressure
Recent changes
```

Do not restart all nodes blindly.

---

# 131. Incident Example — Logstash Queue Growth

Check:

```text
Elasticsearch output latency
Elasticsearch rejections
Logstash CPU
Logstash memory
Persistent queue
Network
Bulk failures
```

Determine whether the bottleneck is upstream, Logstash, or Elasticsearch.

---

# 132. Incident Example — Node Disk Full

Logging itself may cause disk exhaustion.

Check:

```text
container logs
Fluent Bit buffers
container runtime logs
application temporary files
Prometheus
system logs
```

Freeing disk must be done carefully to avoid deleting needed evidence.

---

# 133. Production Runbook — Collector Down

```text
1. Identify affected node(s).
2. Check Fluent Bit DaemonSet.
3. Check pod status.
4. Check logs.
5. Check node disk.
6. Check memory/CPU.
7. Check mounted log path.
8. Check RBAC.
9. Check configuration.
10. Restart only after identifying the cause.
11. Confirm events resume.
12. Check for backlog/loss.
```

---

# 134. Production Runbook — Elasticsearch Disk High

```text
1. Check filesystem usage.
2. Identify largest indices.
3. Check lifecycle policy.
4. Check retention.
5. Check snapshots.
6. Check shard distribution.
7. Remove only approved expired data.
8. Scale storage if required.
9. Verify allocation.
10. Confirm indexing recovery.
```

---

# 135. Production Runbook — Kibana No Data

```text
1. Verify datasource.
2. Verify index/data view.
3. Verify time range.
4. Verify @timestamp.
5. Search directly in Elasticsearch.
6. Check permissions.
7. Check ingestion.
8. Check index lifecycle.
```

---

# 136. Production Runbook — High Log Volume

```text
1. Identify top-producing services.
2. Compare with traffic.
3. Check recent deployment.
4. Check error loops.
5. Check health-check logging.
6. Check DEBUG level.
7. Check retry storms.
8. Apply temporary filtering only if safe.
9. Fix source application.
10. Verify volume returns to baseline.
```

---

# 137. Observability Correlation

The strongest workflow combines:

```text
Metrics
  |
  +--> "Something is wrong"

Logs
  |
  +--> "What happened?"

Traces
  |
  +--> "Where did the request spend time?"
```

Use all three.

---

# 138. Metrics-to-Logs

Example:

```text
5xx spike
  |
  v
Grafana
  |
  v
checkout service
  |
  v
Kibana
  |
  v
exception
```

---

# 139. Logs-to-Traces

```text
Error log
  |
  +--> trace_id
          |
          v
        Trace
          |
          v
      dependency
```

This can reduce incident investigation time significantly.

---

# 140. Logging SLOs

The logging platform itself should have objectives.

Examples:

```text
Collector availability
Log ingestion success
Search availability
Search latency
Maximum acceptable ingestion delay
```

A logging platform that silently loses production events is a reliability risk.

---

# 141. Log Ingestion Delay

Track:

```text
event timestamp
        vs
index/search timestamp
```

A large difference means logs may be delayed even though they eventually arrive.

---

# 142. Search SLO

Example:

```text
99% of normal operational searches complete within target latency.
```

Define the query class and time range before setting a target.

---

# 143. Logging Alert Severity

Example:

```text
P1:
Central logging unavailable during critical incident

P2:
Production ingestion degraded

P3:
One service missing logs

P4:
Non-production logging issue
```

Severity must follow business impact.

---

# 144. Chaos Testing Logging

Test:

```text
Logstash unavailable
Elasticsearch unavailable
Network interruption
Collector restart
Node failure
Disk pressure
High log volume
Index rejection
```

Observe:

```text
buffer behavior
data loss
recovery
alerting
```

---

# 145. Production Capacity Test

Measure:

```text
events/sec
bytes/sec
peak rate
burst rate
search concurrency
retention volume
```

Do not size only from average traffic.

---

# 146. Peak Log Volume

A deployment can unexpectedly increase logs by:

```text
10x
50x
100x
```

Examples:

```text
retry loop
DEBUG enabled
exception loop
health-check logging
```

Capacity planning must account for bursts.

---

# 147. Elasticsearch Scaling

Scale based on:

```text
CPU
heap
disk
IOPS
ingestion
query latency
rejections
shard pressure
```

Do not scale based on CPU alone.

---

# 148. Logstash Scaling

Logstash can be scaled horizontally.

```text
              Load Balancer
                   |
        +----------+----------+
        |          |          |
     LS-1       LS-2       LS-3
        |          |          |
        +----------+----------+
                   |
             Elasticsearch
```

Persistent queues and ordering requirements must be considered.

---

# 149. Fluent Bit Scaling

DaemonSet scaling follows node count.

Therefore:

```text
More nodes
  =
More collectors
```

Resource requests should account for node-level log volume variation.

---

# 150. Noisy Neighbor Problem

One node may generate huge log volume while another is quiet.

Monitor collector performance per node.

Do not assume all nodes have equal workload.

---

# 151. Security Logging

Security-relevant events may include:

```text
authentication failures
authorization failures
privilege changes
configuration changes
admin actions
network security events
```

These may require separate retention and access policies.

---

# 152. Audit Logging

Audit logs should be:

```text
tamper-resistant
access-controlled
retained appropriately
searchable
backed up
```

Do not treat security audit data exactly like disposable application debug logs.

---

# 153. Immutable Archive

For compliance-sensitive records, an immutable object-storage archive may be appropriate.

Conceptually:

```text
Security Logs
    |
    +--> Search platform
    |
    +--> Immutable archive
```

---

# 154. Elasticsearch Upgrade Strategy

Before upgrading:

```text
1. Read compatibility requirements.
2. Test in non-production.
3. Validate plugins.
4. Validate mappings.
5. Validate lifecycle policies.
6. Validate snapshots.
7. Perform controlled rollout.
8. Monitor cluster health.
9. Validate ingestion.
10. Validate Kibana searches.
```

---

# 155. Logstash Upgrade Strategy

Test:

```text
pipeline syntax
plugins
throughput
memory
persistent queue
Elasticsearch compatibility
```

Deploy gradually where possible.

---

# 156. Fluent Bit Upgrade Strategy

Test:

```text
input parsing
Kubernetes metadata
filters
buffering
output
resource usage
```

A collector upgrade affects every node.

---

# 157. Kibana Upgrade Strategy

Validate:

```text
saved searches
dashboards
data views
authentication
plugins
Elasticsearch compatibility
```

---

# 158. Rollback Strategy

Every component should have a rollback plan:

```text
Fluent Bit
Logstash
Elasticsearch
Kibana
```

Version-control configuration and pin images.

---

# 159. Production Readiness Checklist

### Collection

- [ ] DaemonSet deployed.
- [ ] Logs collected from every intended node.
- [ ] Metadata enrichment works.
- [ ] Buffering configured.
- [ ] Resource requests set.
- [ ] Rotation tested.

### Processing

- [ ] Parsing validated.
- [ ] Sensitive data filtering implemented.
- [ ] Schema standardized.
- [ ] Routing tested.
- [ ] Backpressure tested.

### Elasticsearch

- [ ] HA architecture.
- [ ] Storage sized.
- [ ] Shards reviewed.
- [ ] Replicas configured.
- [ ] Lifecycle policy enabled.
- [ ] Snapshots tested.
- [ ] TLS enabled.
- [ ] RBAC configured.

### Kibana

- [ ] Authentication configured.
- [ ] Dashboards created.
- [ ] Access controls configured.
- [ ] Saved searches tested.

### Operations

- [ ] Alerts configured.
- [ ] Runbooks documented.
- [ ] DR tested.
- [ ] Capacity tested.
- [ ] Cost reviewed.

---

# 160. Production Logging Reference Architecture

```text
                           USERS
                             |
                             v
                           KIBANA
                             |
                             v
                     Elasticsearch
                  +----------+----------+
                  |          |          |
                Node       Node       Node
                  ^          ^          ^
                  |          |          |
               Logstash / Processing Layer
                  ^
                  |
        +---------+---------+
        |         |         |
      EKS-A     EKS-B     EKS-C
        |         |         |
     FluentBit FluentBit FluentBit
        |         |         |
        +---------+---------+
                  |
            Optional Kafka
                  |
                  v
             Logstash
```

---

# 161. Complete DevOps Incident Workflow

```text
                    ALERT
                      |
                      v
                Grafana Metrics
                      |
             +--------+--------+
             |                 |
           Logs              Traces
             |                 |
             +--------+--------+
                      |
                      v
                 Root Cause
                      |
                      v
                Remediation
                      |
                      v
                  Rollback
                      |
                      v
                 Verification
                      |
                      v
               Post-Incident
```

---

# 162. What a Senior DevOps Engineer Should Own

A senior engineer should be able to:

- design centralized logging
- select collection architecture
- configure Fluent Bit
- configure Logstash
- design Elasticsearch topology
- manage Kibana
- define schemas
- protect sensitive logs
- troubleshoot ingestion
- troubleshoot Elasticsearch
- build operational dashboards
- integrate metrics/logs/traces
- design retention
- optimize cost
- perform upgrades
- execute disaster recovery
- define logging SLOs
- create incident runbooks

---

# 163. Senior Interview Question — Why Fluent Bit?

A strong answer:

> I use Fluent Bit as a lightweight node-level collector in Kubernetes. It can tail container logs, enrich them with Kubernetes metadata, parse structured logs, buffer records, and forward them to the centralized backend. I keep the collector lightweight and introduce Logstash when advanced transformation or routing is justified.

---

# 164. Senior Interview Question — Why Logstash?

Answer:

> Logstash is useful when I need more complex parsing, transformation, enrichment, conditional routing, multiple outputs, or a persistent processing layer. I do not automatically introduce it when Fluent Bit can satisfy the requirements because every additional component increases operational complexity.

---

# 165. Senior Interview Question — How Do You Prevent Log Loss?

Answer:

> I design buffering at the collector and processing layers, monitor downstream backpressure, use durable queues where required, and define acceptable RPO for logs. I test Elasticsearch failure, Logstash failure, collector restarts, node failures, and disk pressure rather than assuming retries guarantee delivery.

---

# 166. Senior Interview Question — How Do You Handle High Log Volume?

Answer:

> First I identify the producer responsible and determine whether the increase is caused by traffic, retries, DEBUG logging, exception loops, or health checks. Then I reduce unnecessary source volume, use filtering or sampling where appropriate, optimize processing, and scale the backend based on ingestion rate, storage, heap, I/O, and query load.

---

# 167. Senior Interview Question — How Do You Troubleshoot Missing Logs?

Answer:

> I trace the pipeline end-to-end: application stdout, node log files, Fluent Bit input and output metrics, buffers, network, Logstash queues and outputs, Elasticsearch indexing, and finally Kibana search/data views. This identifies exactly where the event stopped rather than restarting components blindly.

---

# 168. Senior Interview Question — How Do You Handle PII?

Answer:

> I prefer not to log sensitive data at all. Where required, I redact or mask it before indexing, restrict access, apply retention controls, and audit access. I do not rely only on Kibana permissions because the sensitive data has already entered the platform.

---

# 169. Senior Interview Question — How Do You Design Elasticsearch for HA?

Answer:

> I distribute nodes across failure domains, maintain appropriate replicas, use enough cluster-manager-eligible nodes for the intended failure tolerance, size storage and memory from ingestion and query requirements, and continuously monitor disk, heap, indexing pressure, shard allocation, and rejections. I also maintain tested snapshots and a documented recovery process.

---

# 170. Senior Interview Question — What Causes Elasticsearch to Become Unhealthy?

Potential causes:

```text
disk exhaustion
heap pressure
GC pauses
too many shards
high indexing rate
expensive queries
node failure
network issues
allocation problems
mapping explosion
```

Investigate evidence before taking action.

---

# 171. Senior Interview Question — Why Are Too Many Shards Bad?

Answer:

> Every shard carries memory, file, cluster-state, and management overhead. Excessive shard counts can increase heap pressure and slow cluster operations. I size shards from actual data volume, retention, query patterns, and supported Elasticsearch guidance rather than creating large numbers of tiny shards.

---

# 172. Senior Interview Question — What Is Your Logging Retention Strategy?

Answer:

> I classify logs by operational and compliance value. Recent operational logs stay in fast storage, older data moves to lower-cost tiers where appropriate, and data is deleted or archived according to approved retention policies. Security and audit logs may have separate requirements.

---

# 173. Senior Interview Question — How Do You Integrate Logs with Grafana?

Answer:

> Grafana can query a supported logging datasource such as Elasticsearch or Loki. I use dashboards to correlate metrics with logs and provide drill-down links. For example, an HTTP 5xx panel can link an engineer directly to logs filtered by service, namespace, pod, and time range.

---

# 174. Senior Interview Question — How Do You Correlate Logs and Traces?

Answer:

> I propagate trace and span identifiers through services and include them in structured logs. During an incident, I use metrics to detect the problem, logs to identify the error, and the trace to understand the request path and dependency responsible.

---

# 175. Senior Interview Question — What Happens When Elasticsearch Goes Down?

Strong answer:

> I expect buffering upstream. Fluent Bit filesystem buffers, Logstash persistent queues, or Kafka can temporarily absorb the outage depending on the architecture. I monitor queue depth and disk consumption, recover Elasticsearch, verify backlog processing, and determine whether any events were lost or duplicated.

---

# 176. Senior Interview Question — What Would You Alert On?

Alert on operational conditions such as:

```text
collector unavailable
ingestion failure
queue growth
Elasticsearch unhealthy
disk near capacity
heap pressure
search failure
critical service log silence
unexpected volume spike
```

Do not create an alert for every ERROR log.

---

# 177. Final Production Principles

1. Structured logs are superior to uncontrolled text.
2. Never intentionally log secrets.
3. Collect Kubernetes logs close to the source.
4. Use Fluent Bit when a lightweight collector is appropriate.
5. Use Logstash when its processing capabilities justify the complexity.
6. Treat Elasticsearch as a production distributed datastore.
7. Control shard count.
8. Plan storage before production launch.
9. Use lifecycle policies.
10. Define retention explicitly.
11. Use snapshots and test restores.
12. Secure every network hop.
13. Apply least privilege.
14. Protect PII before indexing.
15. Control field cardinality and mapping growth.
16. Design for backpressure.
17. Monitor the logging pipeline itself.
18. Correlate logs with metrics and traces.
19. Keep configuration in Git.
20. Deploy through CI/CD and GitOps.
21. Test collector, processor, and backend failure.
22. Treat missing logs as an operational signal.
23. Optimize cost by reducing unnecessary volume.
24. Build dashboards around operational questions.
25. Make the logging platform recoverable.

---

# Final Capstone Outcome

After completing this section, the production capstone should contain a logging architecture capable of:

```text
EKS
 |
 +--> Fluent Bit
        |
        +--> Kubernetes metadata
        +--> JSON parsing
        +--> filtering
        +--> buffering
        |
        v
     Logstash
        |
        +--> transformation
        +--> routing
        +--> enrichment
        |
        v
  Elasticsearch
        |
        +--> lifecycle
        +--> replicas
        +--> snapshots
        +--> security
        |
        v
      Kibana
        |
        +--> Search
        +--> Dashboards
        +--> Investigation
        +--> Security analysis
```

The complete observability workflow becomes:

```text
Prometheus/Grafana
       |
       | detect
       v
Elasticsearch/Kibana
       |
       | investigate
       v
OpenTelemetry/Jaeger
       |
       | trace
       v
Root Cause
       |
       v
Remediation / Rollback
       |
       v
Grafana
       |
       | verify recovery
       v
Incident Closed
```

**Production mindset:** Logging is successful only when an engineer can reliably reconstruct what happened during an incident, correlate it with metrics and traces, protect sensitive information, recover the logging platform after failure, and operate the entire pipeline at predictable cost and performance.
