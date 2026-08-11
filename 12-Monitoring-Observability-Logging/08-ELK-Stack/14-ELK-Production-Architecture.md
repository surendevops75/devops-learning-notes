# ELK Production Architecture

## 1. Overview

A production ELK architecture must be designed for:

* High availability
* Scalability
* Security
* Reliable log ingestion
* Centralized storage
* Efficient search
* Fault tolerance
* Data retention
* Monitoring
* Disaster recovery
* Operational simplicity

The core architecture is:

```text
Applications
     ↓
Log Collectors
     ↓
Logstash
     ↓
Elasticsearch Cluster
     ↓
Kibana
     ↓
DevOps / Platform / Security Teams
```

For a Kubernetes and AWS environment:

```text
                         Users
                           │
                           ↓
                    Private Route53
                           │
                           ↓
                    Internal AWS ALB
                           │
                 ┌─────────┴─────────┐
                 ↓                   ↓
             Kibana-01           Kibana-02
                 │                   │
                 └─────────┬─────────┘
                           ↓
                 Elasticsearch Cluster
                           ↑
                           │
                    Logstash Cluster
                           ↑
                           │
                     Fluent Bit
                           ↑
                           │
                        EKS Pods
```

---

# 2. Production Requirements

Before designing ELK, determine:

```text
Daily log volume
Peak log volume
Retention period
Number of applications
Number of Kubernetes clusters
Number of users
Search workload
Dashboard workload
Availability requirements
Security requirements
Compliance requirements
Recovery requirements
```

Example:

```text
Daily ingestion:
100 GB/day

Retention:
30 days

Availability:
High availability

Deployment:
AWS EKS

Access:
Internal only

Visualization:
Kibana

Metrics:
Prometheus + Grafana
```

These values are examples. Production sizing must be based on measured workload.

---

# 3. Production Architecture Layers

Think about the architecture in layers:

```text
Layer 1 → Application Logging

Layer 2 → Log Collection

Layer 3 → Log Processing

Layer 4 → Log Storage

Layer 5 → Visualization

Layer 6 → Security

Layer 7 → Monitoring

Layer 8 → High Availability

Layer 9 → Disaster Recovery
```

Each layer has a different responsibility.

---

# 4. Application Layer

Applications generate logs.

Example microservices:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

Each service should produce structured logs.

Example:

```json
{
  "@timestamp": "2026-08-11T10:30:25Z",
  "service.name": "payment",
  "service.version": "v1.5.2",
  "environment": "production",
  "log.level": "ERROR",
  "message": "Database connection timeout"
}
```

---

# 5. Application Logging Standard

Applications should consistently provide:

```text
@timestamp
log.level
service.name
service.version
environment
message
request.id
trace.id
```

Kubernetes applications should additionally provide or allow enrichment with:

```text
namespace
pod
container
node
cluster
```

Consistent fields make centralized searches and dashboards much easier.

---

# 6. Log Collection Layer

Fluent Bit is commonly used as the collection layer in Kubernetes.

Architecture:

```text
EKS Node
│
├── Application Pod
│      ↓
│    stdout
│
└── Fluent Bit
       ↓
     Collect
```

Fluent Bit is commonly deployed as a DaemonSet.

```text
Node-01 → Fluent Bit
Node-02 → Fluent Bit
Node-03 → Fluent Bit
```

---

# 7. Fluent Bit Responsibilities

Fluent Bit can:

```text
Collect logs
Parse logs
Add Kubernetes metadata
Buffer logs
Filter events
Forward logs
```

It should generally remain lightweight.

Heavy processing can be delegated to Logstash.

---

# 8. Logstash Layer

Logstash provides centralized processing.

Architecture:

```text
Fluent Bit
     ↓
Logstash
     │
     ├── Input
     ├── Filter
     └── Output
             ↓
       Elasticsearch
```

Logstash can:

```text
Parse
Transform
Enrich
Normalize
Route
Filter
```

---

# 9. Logstash High Availability

Avoid a single Logstash instance:

```text
Fluent Bit
     ↓
Logstash-01
     ↓
Elasticsearch
```

Instead:

```text
                 Fluent Bit
                     │
              ┌──────┴──────┐
              ↓             ↓
         Logstash-01   Logstash-02
              │             │
              └──────┬──────┘
                     ↓
              Elasticsearch
```

This removes Logstash as a single point of failure.

---

# 10. Logstash Persistent Queues

If Elasticsearch temporarily becomes unavailable:

```text
Logstash
    ↓
Elasticsearch X
```

events can be buffered using appropriate queueing mechanisms.

Conceptually:

```text
Logstash
   ↓
Persistent Queue
   ↓
Retry
   ↓
Elasticsearch
```

This can reduce log loss during temporary Elasticsearch outages.

---

# 11. Elasticsearch Layer

Elasticsearch is the central storage and search layer.

```text
Logstash
    ↓
Elasticsearch
    ↓
Indexes / Data Streams
    ↓
Documents
```

It provides:

```text
Storage
Indexing
Search
Filtering
Aggregation
Replication
```

---

# 12. Elasticsearch Cluster

Production should use a cluster rather than a single node.

```text
                 Elasticsearch Cluster
                 ┌────────┼────────┐
                 ↓        ↓        ↓
               ES-01    ES-02    ES-03
```

Multiple nodes improve:

```text
Availability
Capacity
Fault tolerance
```

The exact node roles and topology depend on the Elasticsearch version and workload.

---

# 13. Elasticsearch Availability Zones

In AWS:

```text
                 VPC
                  │
       ┌──────────┼──────────┐
       ↓          ↓          ↓
      AZ-1       AZ-2       AZ-3
       │          │          │
     ES-01      ES-02      ES-03
```

Distributing nodes across Availability Zones reduces the risk of losing the entire cluster because of a single AZ failure.

---

# 14. Elasticsearch Node Design

A production Elasticsearch deployment may separate responsibilities according to workload.

Conceptually:

```text
Cluster
│
├── Cluster-manager eligible nodes
├── Data nodes
├── Ingest nodes
└── Coordinating responsibilities
```

The exact topology should be selected based on:

```text
Data volume
Query volume
Ingestion rate
Shard count
Availability requirements
```

Do not blindly deploy the same topology for every environment.

---

# 15. Elasticsearch Data Nodes

Data nodes store indexed data.

```text
Logstash
    ↓
Elasticsearch
    ↓
Data Nodes
```

Data nodes require appropriate:

```text
CPU
Memory
Disk
Network
```

Storage performance is especially important for logging workloads.

---

# 16. Elasticsearch Storage

Logging environments can generate significant data.

Example:

```text
100 GB/day
```

30-day logical volume:

```text
100 × 30
= 3 TB
```

Actual storage requirements will be higher due to:

```text
Replication
Index overhead
Metadata
Segment files
Temporary space
Operational headroom
```

---

# 17. Storage Sizing

Consider:

```text
Daily ingestion
Retention
Replica count
Compression
Index overhead
Growth rate
Free-space requirement
```

Do not size the cluster only based on current ingestion.

Include future growth.

---

# 18. Elasticsearch Replication

Elasticsearch can maintain replicas of indexed data.

Conceptually:

```text
Primary Shard
     │
     └── Replica Shard
```

Example:

```text
ES-01
 └── Primary

ES-02
 └── Replica
```

If ES-01 fails, the replica can become available on another node according to cluster state and configuration.

---

# 19. Shard Distribution

A production cluster should distribute shards across nodes.

```text
              Index
                │
       ┌────────┼────────┐
       ↓        ↓        ↓
     Shard-1  Shard-2  Shard-3
       │        │        │
       ↓        ↓        ↓
     ES-01    ES-02    ES-03
```

Poor shard sizing can cause:

```text
High overhead
Slow queries
Uneven load
Cluster instability
```

---

# 20. Avoid Excessive Shards

More shards do not automatically mean better performance.

Example:

```text
1000 tiny shards
```

can be worse than:

```text
Appropriately sized shards
```

Plan shard counts based on:

```text
Data size
Query workload
Retention
Number of nodes
Growth
```

---

# 21. Index/Data Stream Strategy

A centralized logging platform should have a deliberate data organization strategy.

Possible logical categories:

```text
application-logs
kubernetes-logs
security-logs
audit-logs
infrastructure-logs
```

Data streams may be appropriate for time-series log workloads depending on the Elasticsearch architecture.

---

# 22. Environment Separation

Logs can be separated logically:

```text
development
staging
production
```

Example:

```text
environment = production
```

Then Kibana can filter:

```text
environment : "production"
```

For larger environments, separate data streams/index patterns and access policies may be appropriate.

---

# 23. Service Separation

Include:

```text
service.name
```

Example:

```text
service.name = payment
service.name = orders
service.name = inventory
```

This allows centralized storage while maintaining service-level filtering.

---

# 24. Elasticsearch Security

Production Elasticsearch should not be publicly accessible.

Bad architecture:

```text
Internet
   ↓
Elasticsearch :9200
```

Preferred:

```text
Private Network
      ↓
Elasticsearch
```

Applications and logging components access it through controlled network paths.

---

# 25. Security Group Architecture

AWS security groups:

```text
ALB SG
  ↓
Kibana SG
  ↓
Elasticsearch SG
```

For ingestion:

```text
Logstash SG
  ↓
Elasticsearch SG
```

Only required traffic should be allowed.

---

# 26. TLS Architecture

Production communication should be encrypted.

```text
Fluent Bit
    ↓ TLS
Logstash
    ↓ TLS
Elasticsearch
    ↑ TLS
Kibana
    ↑ HTTPS
Users
```

TLS protects:

```text
Credentials
Log content
Queries
Sessions
Sensitive metadata
```

---

# 27. Authentication

Use authentication between components where required.

Conceptually:

```text
Logstash
   ↓
Elasticsearch Authentication

Kibana
   ↓
Elasticsearch Authentication

Users
   ↓
Kibana Authentication
```

Do not use a powerful administrator account for every component.

---

# 28. Least Privilege

Separate identities:

```text
Kibana Server Identity
Logstash Identity
Human Users
Administrators
```

Each should receive only the permissions required for its role.

---

# 29. Kibana Layer

Kibana is the visualization and investigation layer.

```text
Elasticsearch
      ↓
    Kibana
      ↓
Users
```

Kibana provides:

```text
Search
Discover
Dashboards
Visualizations
Alerting
Saved objects
Spaces
```

---

# 30. Kibana High Availability

Avoid:

```text
ALB
 ↓
Kibana-01
```

Use:

```text
                 Internal ALB
                      │
              ┌───────┴───────┐
              ↓               ↓
          Kibana-01       Kibana-02
              │               │
              └───────┬───────┘
                      ↓
               Elasticsearch
```

If one Kibana instance fails, the other can continue serving users.

---

# 31. Kibana Across Availability Zones

AWS architecture:

```text
                 Internal ALB
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
        AZ-1                    AZ-2
          │                       │
      Kibana-01              Kibana-02
```

This reduces the impact of an Availability Zone failure.

---

# 32. Route53

Use private DNS where Kibana is internal.

Example:

```text
kibana.prod.internal
```

Flow:

```text
User
 ↓
Private Route53
 ↓
Internal ALB
 ↓
Kibana
```

This is preferable to exposing Kibana directly through a public IP when it is intended for internal users.

---

# 33. Internal ALB

The ALB provides:

```text
TLS termination
Load balancing
Health checks
Stable endpoint
```

Architecture:

```text
User
 ↓
HTTPS
 ↓
Internal ALB
 ↓
Kibana
```

---

# 34. ALB Health Checks

The ALB should route traffic only to healthy Kibana targets.

```text
ALB
 │
 ├── Kibana-01 → Healthy
 │
 └── Kibana-02 → Unhealthy
```

Traffic goes to healthy targets.

The exact health-check configuration should match the Kibana deployment and authentication architecture.

---

# 35. Kubernetes Architecture

For EKS:

```text
EKS
│
├── Application Pods
├── Fluent Bit DaemonSet
├── Logstash
└── Kibana
```

Elasticsearch may run:

```text
Inside EKS
```

or:

```text
On dedicated infrastructure
```

depending on operational requirements.

---

# 36. Kubernetes Namespace

Use a dedicated namespace:

```bash
kubectl create namespace logging
```

Example:

```text
logging
│
├── Fluent Bit
├── Logstash
└── Kibana
```

Some organizations may use:

```text
observability
```

instead.

---

# 37. Kubernetes Resource Management

Define resource requests and limits.

Example:

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "1Gi"
  limits:
    cpu: "1"
    memory: "2Gi"
```

These are example values.

Production values should come from measurements.

---

# 38. Kubernetes Pod Distribution

For high availability:

```text
Node-01
 └── Kibana-01

Node-02
 └── Kibana-02
```

Use:

```text
Pod anti-affinity
Topology spread constraints
```

to reduce the probability that all replicas land on the same failure domain.

---

# 39. Persistent Storage

Elasticsearch requires durable storage.

Conceptually:

```text
Elasticsearch Pod
       ↓
Persistent Volume
       ↓
Durable Storage
```

Do not treat Elasticsearch data like stateless application data.

If Elasticsearch is deployed on Kubernetes, storage design is a critical production decision.

---

# 40. Storage Performance

Logging workloads can generate:

```text
High writes
High reads
Large sequential operations
Segment merges
```

Therefore evaluate:

```text
IOPS
Throughput
Latency
Capacity
Durability
```

Storage bottlenecks can affect the entire logging platform.

---

# 41. Log Ingestion Pipeline

Production flow:

```text
Application
     ↓
stdout
     ↓
Fluent Bit
     ↓
Logstash
     ↓
Elasticsearch
```

The ingestion path should be monitored continuously.

---

# 42. Backpressure

Suppose Elasticsearch slows down:

```text
Elasticsearch
     ↓
Slow
     ↓
Logstash
     ↓
Queue increases
     ↓
Fluent Bit
```

The system should handle temporary pressure through:

```text
Buffering
Retries
Queues
Persistent queues
Batching
```

---

# 43. Application Impact

The logging system should not become a dependency that blocks application requests.

Prefer:

```text
Application
     ↓
stdout
     ↓
Asynchronous collection
```

rather than making every application request wait for Elasticsearch.

---

# 44. Log Loss Considerations

Potential loss points:

```text
Application
     ↓
Collector
     ↓
Network
     ↓
Logstash
     ↓
Elasticsearch
```

For critical environments, design:

```text
Buffers
Retries
Persistent queues
Replication
Monitoring
```

to reduce the probability of losing important events.

---

# 45. Log Volume Control

Do not send unnecessary logs.

Avoid:

```text
DEBUG
DEBUG
DEBUG
DEBUG
```

for every production request unless there is a justified reason.

Instead:

```text
INFO
WARN
ERROR
```

and temporarily increase verbosity when troubleshooting.

---

# 46. Sensitive Information

Centralized logs can become a security risk.

Never intentionally log:

```text
Passwords
Access tokens
Private keys
API secrets
Credit card information
Session secrets
```

Prefer:

```text
[REDACTED]
```

or avoid logging the field entirely.

---

# 47. Log Redaction

Pipeline:

```text
Application
    ↓
Sensitive field detected
    ↓
Redact / Remove
    ↓
Elasticsearch
```

Application-level prevention is preferable to relying entirely on Logstash filtering.

---

# 48. Access Control

Different teams need different permissions.

Example:

```text
Platform Team
    ↓
Infrastructure logs

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

Use appropriate Kibana spaces and Elasticsearch authorization controls.

---

# 49. Kibana Spaces

Example:

```text
Kibana
│
├── Platform
├── Applications
├── Security
└── Production
```

Each space can contain its own dashboards and saved objects.

---

# 50. Centralized Dashboards

A production ELK dashboard may show:

```text
Total Logs
Errors
Warnings
Logs by Service
Logs by Namespace
Logs by Environment
Top Exceptions
Recent Errors
```

---

# 51. Kubernetes Dashboard

Example:

```text
EKS Logging Dashboard
│
├── Logs by Namespace
├── Errors by Namespace
├── Errors by Pod
├── Errors by Container
├── Logs by Node
└── Recent Errors
```

---

# 52. Application Dashboard

Example:

```text
Payment Dashboard
│
├── Total Events
├── Error Count
├── Warning Count
├── Top Exceptions
├── Errors by Pod
├── Errors by Version
└── Recent Errors
```

---

# 53. Alerting

Useful log-based alerts include:

```text
Critical application exception
Authentication failure spike
Large increase in ERROR logs
No logs from critical service
Unexpected log volume increase
Repeated database errors
```

Avoid alerting on every warning.

---

# 54. Monitoring ELK With Prometheus

The logging platform itself must be monitored.

```text
Fluent Bit
     ↓
Metrics
     ↓
Prometheus
     ↓
Grafana
```

Monitor:

```text
Events received
Events processed
Events dropped
Buffer size
Queue size
Processing latency
Errors
```

---

# 55. Monitor Logstash

Important operational signals:

```text
Pipeline events
Queue size
Processing rate
Input rate
Output rate
Failed events
Elasticsearch connection errors
CPU
Memory
```

A sudden difference between input and output rates can indicate a pipeline problem.

---

# 56. Monitor Elasticsearch

Monitor:

```text
Cluster health
Node availability
CPU
Memory
JVM pressure
Disk usage
Disk latency
Shard health
Indexing rate
Search latency
Rejected requests
```

---

# 57. Monitor Kibana

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

# 58. Monitor Fluent Bit

Monitor:

```text
Input records
Output records
Dropped records
Buffer usage
Retry count
Errors
CPU
Memory
```

A collector failure can create an invisible logging outage.

---

# 59. Logging Pipeline Dashboard

Create a Grafana dashboard:

```text
ELK Pipeline
│
├── Fluent Bit
│   ├── Input Rate
│   ├── Output Rate
│   └── Errors
│
├── Logstash
│   ├── Events In
│   ├── Events Out
│   └── Queue Size
│
├── Elasticsearch
│   ├── Index Rate
│   ├── Search Rate
│   └── Cluster Health
│
└── Kibana
    ├── Availability
    └── Response Time
```

---

# 60. Elasticsearch Cluster Health

Always monitor cluster health.

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
Critical data availability issue
```

The exact cause must be investigated rather than assuming the color alone identifies the root cause.

---

# 61. Disk Monitoring

Elasticsearch requires sufficient free disk.

Monitor:

```text
Disk usage
Disk growth
Disk latency
```

A full disk can cause severe Elasticsearch problems.

Create alerts before the system reaches critical capacity.

---

# 62. JVM Memory

Elasticsearch uses JVM memory.

Monitor:

```text
Heap usage
GC activity
JVM pressure
```

Avoid blindly increasing heap size.

Memory sizing must account for:

```text
Heap
Operating system
Filesystem cache
Other processes
```

---

# 63. Query Performance

Kibana dashboards can generate expensive Elasticsearch queries.

Monitor:

```text
Query latency
Search rate
Rejected searches
CPU
Memory
```

If Kibana becomes slow:

```text
Kibana
   ↓
Check Elasticsearch
```

The bottleneck may be Elasticsearch rather than Kibana.

---

# 64. Dashboard Optimization

Avoid dashboards with:

```text
Large time ranges
Many panels
Expensive aggregations
High-cardinality aggregations
Unnecessary refresh frequency
```

Prefer:

```text
Focused queries
Reasonable time ranges
Useful panels
```

---

# 65. Retention Architecture

Logs should move through lifecycle stages:

```text
Hot
 ↓
Recent operational logs

Warm
 ↓
Older logs

Cold
 ↓
Long-term retention

Delete
 ↓
Expired data
```

The exact architecture depends on Elasticsearch capabilities, storage design, and business requirements.

---

# 66. Retention Policy

Example:

```text
Hot:
7 days

Warm:
23 days

Long-term:
Optional

Delete:
After required retention
```

These values are examples only.

Retention should be based on:

```text
Business
Compliance
Security
Incident response
Cost
```

---

# 67. Cost Optimization

Main cost drivers:

```text
Log volume
Retention
Replication
Storage
Query workload
Compute
```

Optimization:

```text
Reduce unnecessary logs
Use appropriate retention
Use lifecycle policies
Optimize mappings
Optimize dashboards
Use suitable storage
```

---

# 68. Disaster Recovery

A production logging platform should have a recovery strategy.

Consider:

```text
Configuration backup
Index/data recovery
Snapshot strategy
Cluster rebuild
DNS recovery
Kibana configuration
Logstash pipelines
Fluent Bit configuration
```

---

# 69. Elasticsearch Snapshots

Elasticsearch supports snapshot-based backup mechanisms.

Conceptually:

```text
Elasticsearch
      ↓
Snapshot
      ↓
Object Storage
```

For AWS environments:

```text
Elasticsearch
      ↓
Snapshot Repository
      ↓
S3
```

The exact implementation depends on the Elasticsearch deployment and supported repository configuration.

---

# 70. Disaster Recovery Architecture

Conceptually:

```text
Primary Region
     │
     ↓
Elasticsearch
     │
     ↓
Snapshots
     │
     ↓
S3
     │
     ↓
Recovery Region
```

The required design depends on:

```text
RPO
RTO
Cost
Compliance
Business criticality
```

---

# 71. RPO

Recovery Point Objective answers:

```text
How much data can we afford to lose?
```

Example:

```text
RPO = 1 hour
```

means the organization accepts a potential recovery point up to one hour behind the failure.

---

# 72. RTO

Recovery Time Objective answers:

```text
How quickly must the logging platform recover?
```

Example:

```text
RTO = 2 hours
```

means the organization aims to restore the service within two hours.

---

# 73. Failure Domains

Production architecture should account for:

```text
Pod failure
Node failure
AZ failure
Network failure
Instance failure
Storage failure
Elasticsearch node failure
Logstash failure
Kibana failure
```

---

# 74. AZ Failure

Example:

```text
AZ-1
 └── Kibana-01
 └── ES-01

AZ-2
 └── Kibana-02
 └── ES-02

AZ-3
 └── ES-03
```

If AZ-1 fails:

```text
Kibana-01 → Lost
ES-01 → Lost
```

the remaining architecture should continue operating within its designed failure tolerance.

---

# 75. Network Failure

Suppose:

```text
Logstash
   X
Elasticsearch
```

The system should:

```text
Buffer
Retry
Alert
Recover
```

rather than silently dropping every event.

---

# 76. Elasticsearch Failure

Suppose ES-02 fails:

```text
ES-01 ✓
ES-02 X
ES-03 ✓
```

If the cluster has appropriate replicas and capacity:

```text
Cluster
 ↓
Continue operating
```

Then investigate:

```text
Node failure
Disk
Network
JVM
Storage
```

---

# 77. Kibana Failure

Suppose:

```text
Kibana-01 X
Kibana-02 ✓
```

The internal ALB can route traffic to:

```text
Kibana-02
```

Users may continue using the logging interface.

---

# 78. Logstash Failure

Suppose:

```text
Logstash-01 X
Logstash-02 ✓
```

A resilient ingestion architecture should continue processing through the healthy instance, assuming the traffic distribution and buffering architecture are correctly designed.

---

# 79. Fluent Bit Failure

Because Fluent Bit runs as a DaemonSet:

```text
Node-01 → Fluent Bit X
Node-02 → Fluent Bit ✓
Node-03 → Fluent Bit ✓
```

only workloads associated with the affected collector may experience log collection problems.

Kubernetes should restart the failed Fluent Bit Pod according to its workload configuration.

---

# 80. Security Architecture

Production ELK:

```text
                         Corporate Users
                               │
                               ↓
                         Internal ALB
                               │
                              TLS
                               ↓
                            Kibana
                               │
                              TLS
                               ↓
                       Elasticsearch
                               ↑
                              TLS
                               │
                           Logstash
                               ↑
                              TLS
                               │
                         Fluent Bit
                               ↑
                         Applications
```

All major communication paths should be explicitly secured.

---

# 81. Secret Management Architecture

```text
AWS Secrets Manager
        ↓
External Secrets / Secret Mechanism
        ↓
Kubernetes Secret
        ↓
ELK Components
```

Avoid:

```text
Git
 ↓
Plaintext password
```

---

# 82. IAM Architecture

AWS resources should use IAM roles rather than hardcoded access keys where possible.

For EKS workloads:

```text
Kubernetes Service Account
        ↓
AWS IAM Role
        ↓
AWS API
```

Use the AWS-native workload identity mechanism appropriate for the cluster configuration.

---

# 83. Network Segmentation

A production VPC may contain:

```text
Public Subnets
Private Application Subnets
Private Data Subnets
```

ELK components should generally reside in private networking where appropriate.

Example:

```text
Public
  ↓
Internal boundary / controlled entry
  ↓
Private Kibana
  ↓
Private Elasticsearch
```

---

# 84. Elasticsearch Should Remain Private

Avoid:

```text
Internet
   ↓
Elasticsearch
```

Prefer:

```text
Application Network
        ↓
Private Elasticsearch
```

Only approved systems should be able to connect to Elasticsearch.

---

# 85. Kibana Should Be Restricted

Kibana is an operational interface and should normally not be openly exposed.

Prefer:

```text
Corporate Network
      ↓
VPN / Private Connectivity
      ↓
Internal ALB
      ↓
Kibana
```

Authentication and authorization should still be enabled.

---

# 86. Production Configuration Management

Configuration should be version controlled.

Example:

```text
GitHub
│
├── fluent-bit/
├── logstash/
├── elasticsearch/
└── kibana/
```

Changes should follow:

```text
Pull Request
 ↓
Validation
 ↓
Security Checks
 ↓
Review
 ↓
Merge
 ↓
Deployment
```

---

# 87. GitHub Actions

GitHub Actions can perform:

```text
YAML validation
Helm lint
Kubernetes validation
Configuration tests
Security scanning
```

Then:

```text
Merge
 ↓
ArgoCD
```

---

# 88. ArgoCD

ArgoCD maintains desired state:

```text
Git
 ↓
ArgoCD
 ↓
EKS
```

If the cluster changes unexpectedly:

```text
Git desired state
       vs
Cluster actual state
```

ArgoCD can detect drift.

---

# 89. GitOps ELK Architecture

```text
GitHub
   │
   ├── Fluent Bit
   ├── Logstash
   ├── Elasticsearch
   └── Kibana
          │
          ↓
       ArgoCD
          │
          ↓
         EKS
          │
          ↓
       ELK Stack
```

This provides:

```text
Version control
Auditability
Repeatability
Rollback
Drift detection
```

---

# 90. Terraform Responsibilities

Terraform should manage infrastructure:

```text
VPC
Subnets
Security Groups
IAM
EKS
ALB
Route53
S3
Supporting AWS resources
```

Then:

```text
Terraform
   ↓
Infrastructure
   ↓
ArgoCD
   ↓
ELK workloads
```

---

# 91. Responsibility Separation

A clean production model:

```text
Terraform
   ↓
Cloud infrastructure

GitHub Actions
   ↓
Validation and CI

ArgoCD
   ↓
Kubernetes deployment

ELK
   ↓
Centralized logging

Prometheus
   ↓
Metrics

Grafana
   ↓
Visualization and alerting
```

---

# 92. Production Repository

Example:

```text
observability/
│
├── fluent-bit/
│   ├── base/
│   └── overlays/
│
├── logstash/
│   ├── base/
│   └── overlays/
│
├── elasticsearch/
│   ├── base/
│   └── overlays/
│
└── kibana/
    ├── base/
    └── overlays/
```

Environment overlays:

```text
dev
staging
prod
```

---

# 93. Production Deployment Flow

```text
Developer
   ↓
GitHub
   ↓
Pull Request
   ↓
GitHub Actions
   ├── YAML validation
   ├── Helm lint
   ├── Security scan
   └── Tests
   ↓
Code Review
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
ELK
```

---

# 94. Production Change Management

Do not manually edit production Pods.

Bad:

```text
kubectl edit deployment kibana
```

followed by leaving the change undocumented.

Preferred:

```text
Change Git
 ↓
Pull Request
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
 ↓
Production
```

This provides traceability.

---

# 95. Production Rollback

Suppose a new Logstash configuration breaks ingestion.

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
Pull Request
   ↓
Validation
   ↓
Merge
   ↓
ArgoCD
   ↓
Version 1
```

---

# 96. End-to-End Production Validation

After deployment:

```text
[ ] Applications generate logs
[ ] Fluent Bit collects logs
[ ] Logstash receives logs
[ ] Logstash processes logs
[ ] Elasticsearch indexes logs
[ ] Kibana searches logs
[ ] Dashboards work
[ ] Alerts work
[ ] TLS works
[ ] Authentication works
[ ] RBAC works
[ ] Monitoring works
```

---

# 97. Production Incident Workflow

During an incident:

```text
Alert
  ↓
Grafana
  ↓
Identify affected service
  ↓
Kibana
  ↓
Filter environment
  ↓
Filter service
  ↓
Filter ERROR
  ↓
Find request/trace ID
  ↓
Trace investigation
  ↓
Identify root cause
  ↓
Remediate
  ↓
Verify recovery
```

---

# 98. Example Incident

Suppose:

```text
Payment API latency ↑
```

Grafana:

```text
Payment latency
       ↑
```

Kibana:

```text
environment : "production"
and service.name : "payment"
and log.level : "ERROR"
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

Deployment history:

```text
v1.5.2 deployed 10 minutes ago
```

Now the investigation can correlate:

```text
Metric
+
Log
+
Trace
+
Deployment
```

---

# 99. Observability Integration

Complete platform:

```text
                         APPLICATION
                              │
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
          Metrics            Logs            Traces
             │                │                │
             ↓                ↓                ↓
        Prometheus         Fluent Bit      OpenTelemetry
             │                │                │
             ↓                ↓                ↓
          Grafana          Logstash           Jaeger
                              │
                              ↓
                       Elasticsearch
                              │
                              ↓
                           Kibana
```

---

# 100. Production ELK Architecture

The complete architecture:

```text
                              USERS
                                │
                                ↓
                       Private Route53
                                │
                                ↓
                         Internal ALB
                                │
                    ┌───────────┴───────────┐
                    ↓                       ↓
                Kibana-01               Kibana-02
                    │                       │
                    └───────────┬───────────┘
                                ↓
                      Elasticsearch Cluster
                  ┌─────────────┼─────────────┐
                  ↓             ↓             ↓
                ES-01         ES-02         ES-03
                  │             │             │
                  └─────────────┼─────────────┘
                                ↑
                       ┌────────┴────────┐
                       ↓                 ↓
                  Logstash-01       Logstash-02
                       ↑                 ↑
                       └────────┬────────┘
                                ↑
                           Fluent Bit
                                ↑
                ┌───────────────┼───────────────┐
                ↓               ↓               ↓
             EKS Node        EKS Node        EKS Node
                │               │               │
                ↓               ↓               ↓
          Application Pods Application Pods Application Pods
```

---

# 101. Production Architecture by Responsibility

```text
Application
    ↓
Generate structured logs

Fluent Bit
    ↓
Collect + enrich + forward

Logstash
    ↓
Process + transform + route

Elasticsearch
    ↓
Index + store + search

Kibana
    ↓
Visualize + investigate

Prometheus
    ↓
Monitor ELK

Grafana
    ↓
Monitor + alert
```

---

# 102. Production Architecture Security

```text
                    Corporate Users
                           │
                         HTTPS
                           │
                           ↓
                    Internal ALB
                           │
                         HTTPS
                           ↓
                         Kibana
                           │
                         TLS
                           ↓
                   Elasticsearch
                           ↑
                         TLS
                           │
                       Logstash
                           ↑
                         TLS
                           │
                     Fluent Bit
                           ↑
                     EKS Workloads
```

Security controls:

```text
Private networking
TLS
Authentication
RBAC
Least privilege
Secrets management
Security groups
Network policies
Sensitive-data protection
```

---

# 103. Production Architecture Reliability

```text
                High Availability
                       │
       ┌───────────────┼───────────────┐
       ↓               ↓               ↓
    Kibana          Logstash     Elasticsearch
   Multiple         Multiple        Cluster
    replicas         replicas        nodes
       │               │               │
       └───────────────┼───────────────┘
                       ↓
                 Failure tolerance
```

---

# 104. Production Architecture Monitoring

Monitor:

```text
Fluent Bit
    ↓
Collection health

Logstash
    ↓
Pipeline health

Elasticsearch
    ↓
Cluster health

Kibana
    ↓
Visualization health
```

Prometheus:

```text
ELK Components
    ↓
Prometheus
    ↓
Grafana
```

---

# 105. Production Architecture Disaster Recovery

```text
Primary Region
     │
     ↓
ELK Cluster
     │
     ↓
Snapshots
     │
     ↓
S3
     │
     ↓
Recovery Region
```

Test restoration regularly.

A backup that has never been restored should not be considered fully validated.

---

# 106. Production Architecture Checklist

## Infrastructure

```text
[ ] Private VPC
[ ] Multi-AZ design
[ ] Security groups
[ ] Route53
[ ] Internal ALB
[ ] IAM
[ ] Durable storage
```

## Collection

```text
[ ] Fluent Bit DaemonSet
[ ] Buffering
[ ] Retry
[ ] Kubernetes metadata
```

## Processing

```text
[ ] Logstash cluster
[ ] Pipeline configuration
[ ] Parsing
[ ] Enrichment
[ ] Persistent queues where required
```

## Storage

```text
[ ] Elasticsearch cluster
[ ] Appropriate node topology
[ ] Replica strategy
[ ] Shard strategy
[ ] Storage capacity
[ ] Retention
```

## Visualization

```text
[ ] Multiple Kibana replicas
[ ] Internal ALB
[ ] Dashboards
[ ] Data views
[ ] Spaces
[ ] RBAC
```

---

# 107. Security Checklist

```text
[ ] Elasticsearch private
[ ] Kibana private
[ ] TLS enabled
[ ] Authentication enabled
[ ] RBAC configured
[ ] Least privilege
[ ] Secrets protected
[ ] Sensitive data redacted
[ ] Security groups restricted
[ ] Network policies where appropriate
[ ] Certificate monitoring
```

---

# 108. Reliability Checklist

```text
[ ] Multi-AZ
[ ] Multiple Kibana instances
[ ] Multiple Logstash instances
[ ] Elasticsearch cluster
[ ] Replicas
[ ] Persistent queues where required
[ ] Buffering
[ ] Retry
[ ] Health checks
[ ] Monitoring
[ ] Alerting
[ ] Failure testing
```

---

# 109. Performance Checklist

```text
[ ] Structured logs
[ ] Appropriate log levels
[ ] Efficient parsing
[ ] Appropriate shard sizing
[ ] Appropriate mappings
[ ] Query optimization
[ ] Dashboard optimization
[ ] Retention strategy
[ ] Storage performance
[ ] Capacity monitoring
```

---

# 110. Operations Checklist

```text
[ ] GitOps
[ ] GitHub Actions
[ ] ArgoCD
[ ] Version-controlled configuration
[ ] Change approval
[ ] Rollback process
[ ] Runbooks
[ ] Incident procedures
[ ] Ownership
[ ] Disaster recovery testing
```

---

# 111. Production Readiness

The ELK platform should not be considered production-ready merely because:

```text
Kibana opens
```

Production readiness means:

```text
Logs are collected
        ↓
Logs are processed
        ↓
Logs are stored reliably
        ↓
Logs are searchable
        ↓
Access is secured
        ↓
Failures are tolerated
        ↓
Capacity is planned
        ↓
The platform is monitored
        ↓
Backups are tested
        ↓
Deployment is reproducible
```

---

# 112. Final Production Mental Model

Remember the production ELK architecture as:

```text
                         PRODUCTION ELK
                              │
       ┌──────────────────────┼──────────────────────┐
       ↓                      ↓                      ↓
    Collect                 Process                 Store
       │                      │                      │
  Fluent Bit              Logstash             Elasticsearch
       │                      │                      │
       └──────────────────────┼──────────────────────┘
                              ↓
                          Visualize
                              │
                            Kibana
                              │
                              ↓
                            Users
```

Then add the production controls:

```text
                         ELK
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
    Security          Availability       Monitoring
       │                  │                  │
      TLS              Multi-AZ          Prometheus
      RBAC             Replicas           Grafana
      IAM              Clustering         Alerts
      Secrets           Backups
       │                  │
       └──────────────────┼──────────────────┘
                          ↓
                       GitOps
                          │
                 GitHub Actions
                          ↓
                       ArgoCD
                          ↓
                         EKS
```

The key principle is:

**A production ELK architecture is a distributed logging platform, not simply Elasticsearch + Logstash + Kibana installed on three machines. It must reliably collect logs from distributed workloads, process and enrich them, store them across a resilient Elasticsearch cluster, provide secure and highly available Kibana access, handle backpressure and failures, control retention and cost, protect sensitive data, integrate with Prometheus/Grafana and tracing, and use Terraform + GitHub Actions + ArgoCD for reproducible infrastructure and deployment management.**
