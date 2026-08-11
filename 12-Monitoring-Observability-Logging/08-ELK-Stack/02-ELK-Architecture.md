# ELK Architecture

## 1. Overview

The ELK Stack consists of three primary components:

```text
Elasticsearch
Logstash
Kibana
```

The basic architecture is:

```text
Applications
     ↓
   Logs
     ↓
 Logstash
     ↓
Elasticsearch
     ↓
   Kibana
     ↓
  Engineers
```

In a real production environment, the architecture is usually more distributed:

```text
Applications
     ↓
Log Collectors
     ↓
Logstash Cluster
     ↓
Elasticsearch Cluster
     ↓
Kibana
     ↓
Users
```

---

# 2. Responsibilities of Each Component

## Log Collector

Responsible for:

```text
Collecting logs
Reading container logs
Adding basic metadata
Forwarding logs
```

Common examples:

```text
Filebeat
Fluent Bit
Fluentd
```

---

## Logstash

Responsible for:

```text
Parsing
Filtering
Transformation
Enrichment
Routing
Output
```

---

## Elasticsearch

Responsible for:

```text
Indexing
Storage
Searching
Aggregation
Distributed processing
```

---

## Kibana

Responsible for:

```text
Log exploration
Search
Visualization
Dashboards
Analysis
```

---

# 3. Basic ELK Architecture

```text
                    Applications
                         │
                         ↓
                       Logs
                         │
                         ↓
                     Logstash
                         │
                         ↓
                  Elasticsearch
                         │
                         ↓
                       Kibana
                         │
                         ↓
                    Engineers
```

This architecture is easy to understand but is usually too simple for a large production environment.

---

# 4. Production Architecture

A more realistic architecture is:

```text
                       Applications
                            │
                            ↓
                     Log Collectors
                            │
                 ┌──────────┴──────────┐
                 ↓                     ↓
            Logstash A            Logstash B
                 │                     │
                 └──────────┬──────────┘
                            ↓
                  Elasticsearch Cluster
                 ┌──────────┼──────────┐
                 ↓          ↓          ↓
                ES1        ES2        ES3
                 │          │          │
                 └──────────┼──────────┘
                            ↓
                         Kibana
                            │
                            ↓
                          Users
```

---

# 5. Kubernetes Production Architecture

For an EKS environment, the architecture can look like:

```text
                         EKS Cluster
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       Node 1              Node 2              Node 3
          │                   │                   │
       Pods                Pods                Pods
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                    Fluent Bit / Filebeat
                              ↓
                       Logstash Cluster
                              ↓
                    Elasticsearch Cluster
                              ↓
                           Kibana
```

---

# 6. Kubernetes Log Flow

Applications running inside containers generally write logs to:

```text
stdout
stderr
```

The container runtime and Kubernetes node make these logs available to a collector.

The flow becomes:

```text
Application Container
        ↓
stdout / stderr
        ↓
Node Log Files
        ↓
Fluent Bit / Filebeat
        ↓
Logstash
        ↓
Elasticsearch
        ↓
Kibana
```

---

# 7. Why Use a Collector?

Instead of sending every application directly to Logstash:

```text
Application A ──→ Logstash
Application B ──→ Logstash
Application C ──→ Logstash
```

use a node-level collector:

```text
Application A ─┐
Application B ─┤
Application C ─┘
       ↓
Log Collector
       ↓
Logstash
```

This creates a cleaner architecture.

---

# 8. DaemonSet Architecture

In Kubernetes, a collector such as Fluent Bit or Filebeat is commonly deployed as a DaemonSet.

Conceptually:

```text
Node 1
 ├── Application Pods
 └── Log Collector

Node 2
 ├── Application Pods
 └── Log Collector

Node 3
 ├── Application Pods
 └── Log Collector
```

Each node has a collector.

---

# 9. Why DaemonSet?

A DaemonSet ensures a collector runs on the required nodes.

When a new node joins:

```text
New Node
   ↓
DaemonSet
   ↓
Log Collector automatically scheduled
```

This is useful for cluster-wide log collection.

---

# 10. Log Collector Responsibilities

The collector can:

```text
Read container logs
Parse basic formats
Add Kubernetes metadata
Buffer events
Forward events
```

Example metadata:

```json
{
  "namespace": "production",
  "pod": "payment-7d89f",
  "container": "payment",
  "node": "ip-10-0-1-10"
}
```

---

# 11. Collector to Logstash

A common architecture:

```text
Node
 ↓
Fluent Bit
 ↓
Logstash Service
 ↓
Logstash Pods
```

Multiple Logstash Pods can be used:

```text
Fluent Bit
    │
    ↓
Logstash Service
    │
 ┌──┴──┐
 ↓     ↓
LS-A  LS-B
```

---

# 12. Logstash Cluster

For production, avoid relying on a single Logstash instance.

Instead:

```text
               Log Collectors
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
      Logstash A            Logstash B
          │                     │
          └──────────┬──────────┘
                     ↓
             Elasticsearch
```

If Logstash A fails:

```text
Logstash A
    X

Logstash B
    ✓
```

the logging pipeline can continue if the collector and service configuration support failover.

---

# 13. Logstash Processing Pipeline

Logstash internally follows:

```text
Input
  ↓
Filter
  ↓
Output
```

Example:

```text
Fluent Bit
    ↓
  INPUT
    ↓
 Parse JSON
    ↓
 Add Metadata
    ↓
 Filter Fields
    ↓
  OUTPUT
    ↓
Elasticsearch
```

---

# 14. Logstash Input

The input receives events.

Examples:

```text
Beats
TCP
UDP
HTTP
Kafka
File
```

A Kubernetes architecture may use a Beats-compatible input:

```text
Filebeat
   ↓
Logstash Beats Input
```

or an HTTP-based architecture:

```text
Fluent Bit
   ↓
HTTP
   ↓
Logstash
```

The exact input depends on the selected collector and deployment design.

---

# 15. Logstash Filter

The filter layer processes events.

Typical operations:

```text
JSON parsing
Grok parsing
Field extraction
Field renaming
Field removal
Date parsing
Enrichment
Conditional routing
```

Example:

```text
Raw Event
   ↓
JSON Parse
   ↓
Add environment
   ↓
Add service
   ↓
Normalize timestamp
   ↓
Elasticsearch
```

---

# 16. Logstash Output

The output sends processed events to a destination.

For ELK:

```text
Logstash
   ↓
Elasticsearch
```

A production pipeline may route different event types to different indexes.

Example:

```text
Payment Logs
      ↓
logs-payment-*

Security Logs
      ↓
logs-security-*

Infrastructure Logs
      ↓
logs-infrastructure-*
```

---

# 17. Elasticsearch Cluster

Production Elasticsearch should generally be distributed.

Example:

```text
               Elasticsearch Cluster
                /        |        \
               /         |         \
             ES1        ES2        ES3
```

Each node contributes resources to the cluster.

---

# 18. Why Multiple Elasticsearch Nodes?

Multiple nodes provide:

```text
High availability
Horizontal scalability
Shard distribution
Replica placement
Failure tolerance
```

If one node fails:

```text
ES1
 X

ES2
 ✓

ES3
 ✓
```

the cluster can potentially continue serving data depending on shard allocation and cluster state.

---

# 19. Elasticsearch Index

Logs are stored in indexes.

Example:

```text
logs-payment-production
logs-orders-production
logs-inventory-production
```

An index contains documents:

```text
logs-payment-production
 ├── Document 1
 ├── Document 2
 ├── Document 3
 └── Document 4
```

---

# 20. Elasticsearch Documents

Each log event becomes a document.

Example:

```json
{
  "@timestamp": "2026-08-11T10:30:00Z",
  "service": "payment",
  "environment": "production",
  "level": "ERROR",
  "message": "Payment failed"
}
```

Elasticsearch indexes the fields so they can be searched and aggregated.

---

# 21. Shard Architecture

An index can be divided into shards.

```text
Index
 ├── Primary Shard 1
 ├── Primary Shard 2
 └── Primary Shard 3
```

The shards can be distributed across nodes.

```text
ES1
 └── Shard 1

ES2
 └── Shard 2

ES3
 └── Shard 3
```

---

# 22. Replica Architecture

Replicas provide additional copies.

Example:

```text
ES1
 └── Primary Shard 1

ES2
 └── Replica Shard 1
```

If ES1 fails, the replica can potentially become the active copy.

---

# 23. Shard Distribution

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

With replicas:

```text
ES1 → Primary 1
ES2 → Replica 1

ES2 → Primary 2
ES3 → Replica 2

ES3 → Primary 3
ES1 → Replica 3
```

Actual allocation is controlled by Elasticsearch.

---

# 24. Elasticsearch Node Roles

Depending on cluster size, Elasticsearch nodes can have different roles.

Examples include:

```text
Master-eligible
Data
Ingest
Coordinating
```

A small environment may combine roles.

A large environment may separate them.

---

# 25. Small Elasticsearch Deployment

For a small production environment:

```text
        Elasticsearch Cluster
          ┌────┬────┐
          ↓    ↓    ↓
         ES1  ES2  ES3
```

Each node can perform multiple roles.

This keeps the architecture relatively simple.

---

# 26. Large Elasticsearch Deployment

A larger environment may use:

```text
Dedicated Master Nodes
          ↓
      Data Nodes
          ↓
     Ingest Nodes
          ↓
 Coordinating Nodes
```

This provides more control over resource isolation.

---

# 27. Data Nodes

Data nodes handle operations related to stored data.

They are responsible for workloads such as:

```text
Indexing
Searching
Aggregations
Shard storage
```

They generally require significant:

```text
CPU
Memory
Disk
```

depending on workload.

---

# 28. Master-Eligible Nodes

Master-eligible nodes participate in cluster management.

They help manage:

```text
Cluster state
Index metadata
Shard allocation decisions
Node membership
```

For larger clusters, dedicated master nodes can help isolate cluster-management workloads.

---

# 29. Ingest Nodes

Ingest nodes can perform preprocessing before documents are indexed.

Conceptually:

```text
Logstash
   ↓
Elasticsearch Ingest Pipeline
   ↓
Data Node
```

Depending on the architecture, preprocessing can happen in Logstash, Elasticsearch ingest pipelines, or both.

Avoid duplicating expensive processing unnecessarily.

---

# 30. Coordinating Nodes

Coordinating nodes can receive search requests and coordinate work across data nodes.

Example:

```text
Kibana
  ↓
Coordinating Node
  ↓
┌───┼───┐
↓   ↓   ↓
D1  D2  D3
```

The coordinating layer combines responses before returning the result.

---

# 31. Kibana Architecture

Kibana is the visualization layer.

Basic:

```text
User
 ↓
Kibana
 ↓
Elasticsearch
```

Production:

```text
              ALB
               │
        ┌──────┴──────┐
        ↓             ↓
    Kibana A      Kibana B
        │             │
        └──────┬──────┘
               ↓
      Elasticsearch Cluster
```

---

# 32. Kibana Does Not Store Application Logs

The primary log data is stored in Elasticsearch.

Kibana queries Elasticsearch.

```text
Kibana
   ↓
Search Request
   ↓
Elasticsearch
   ↓
Search Results
   ↓
Kibana
```

Kibana is primarily the UI and analytics layer.

---

# 33. User Access

A production access path can be:

```text
User
 ↓
Route 53
 ↓
ALB
 ↓
Kibana
 ↓
Elasticsearch
```

Elasticsearch itself remains private.

---

# 34. ALB Architecture

For EKS:

```text
                   Internet
                       │
                       ↓
                    Route 53
                       │
                       ↓
                      ALB
                       │
                       ↓
                  Kibana Ingress
                       │
                       ↓
                 Kibana Service
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
         Kibana Pod A        Kibana Pod B
```

---

# 35. TLS

Production Kibana should use HTTPS.

```text
User
 ↓
HTTPS
 ↓
ALB
 ↓
Kibana
```

With AWS:

```text
Route 53
   ↓
ALB
   ↓
ACM Certificate
   ↓
HTTPS
   ↓
Kibana
```

---

# 36. Authentication

A production Kibana deployment should use appropriate authentication.

Conceptually:

```text
User
 ↓
Kibana
 ↓
Identity Provider
 ↓
Authentication
 ↓
Kibana
```

Possible enterprise approaches include:

```text
SSO
OIDC
SAML
LDAP
```

depending on the deployment and supported configuration.

---

# 37. Authorization

Authentication identifies the user.

Authorization controls what the user can access.

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
Security indexes
```

Use role-based access control and least privilege.

---

# 38. Complete Kubernetes Architecture

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
                  ┌───────────┼───────────┐
                  ↓           ↓           ↓
                 ES1         ES2         ES3
                  ↑           ↑           ↑
                  └───────────┼───────────┘
                              ↑
                         Logstash
                       ┌──────┴──────┐
                       ↓             ↓
                   Logstash A    Logstash B
                       ↑             ↑
                       └──────┬──────┘
                              ↑
                       Log Collectors
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                  Node 1    Node 2    Node 3
                    │         │         │
                   Pods      Pods      Pods
```

---

# 39. Multi-AZ Architecture

For higher availability, distribute components across Availability Zones.

```text
                    ALB
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
         AZ-A                  AZ-B
          │                     │
       Kibana A              Kibana B
          │                     │
      Logstash A            Logstash B
          │                     │
        ES Node A            ES Node B
```

A third AZ may be used for additional resilience and quorum considerations.

---

# 40. Three-AZ Elasticsearch Example

```text
              Elasticsearch Cluster

       AZ-A          AZ-B          AZ-C
        │             │             │
       ES1           ES2           ES3
        │             │             │
        └─────────────┼─────────────┘
                      │
                  Cluster
```

This reduces dependence on a single Availability Zone.

---

# 41. Complete AWS Architecture

```text
                             INTERNET
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
                     Private VPC / EKS
                                │
        ┌───────────────────────┼───────────────────────┐
        ↓                       ↓                       ↓
    Logstash A              Logstash B             Logstash C
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                ↓
                     Elasticsearch Cluster
                        ┌───────┼───────┐
                        ↓       ↓       ↓
                       ES1     ES2     ES3
```

---

# 42. Application-to-ELK Flow

Suppose the Payment service produces:

```json
{
  "level": "ERROR",
  "service": "payment",
  "order_id": "ORD-1001",
  "message": "Payment failed"
}
```

The flow is:

```text
Payment Pod
    ↓
stdout
    ↓
Fluent Bit
    ↓
Logstash
    ↓
JSON parsing
    ↓
Metadata enrichment
    ↓
Elasticsearch
    ↓
logs-payment-production
    ↓
Kibana
```

---

# 43. Adding Kubernetes Metadata

The collector can add:

```text
cluster
namespace
pod
container
node
```

The final event could look like:

```json
{
  "cluster": "prod-eks",
  "namespace": "production",
  "pod": "payment-7d89f",
  "container": "payment",
  "service": "payment",
  "level": "ERROR",
  "order_id": "ORD-1001",
  "message": "Payment failed"
}
```

This makes production troubleshooting much easier.

---

# 44. Application Logs vs Infrastructure Logs

You may have:

```text
Application Logs
Infrastructure Logs
Kubernetes Logs
Security Logs
Load Balancer Logs
Database Logs
```

They can be routed separately.

Example:

```text
Application Logs
       ↓
logs-app-*

Security Logs
       ↓
logs-security-*

Infrastructure Logs
       ↓
logs-infra-*
```

---

# 45. Routing Architecture

Logstash can use conditional routing.

Conceptually:

```text
                    Logstash
                       │
             ┌─────────┼─────────┐
             ↓         ↓         ↓
        Application  Security   Infra
             ↓         ↓         ↓
          Index A    Index B    Index C
```

This helps isolate workloads and retention policies.

---

# 46. Index Strategy

A practical strategy might be:

```text
logs-app-production
logs-app-staging
logs-security-production
logs-infrastructure-production
```

Avoid creating unnecessary indexes for every small category.

The index strategy should support:

```text
Retention
Search
Access control
Capacity planning
```

---

# 47. Index Lifecycle

A typical lifecycle:

```text
New Logs
   ↓
Hot Data
   ↓
Warm Data
   ↓
Cold / Archive
   ↓
Delete
```

Example:

```text
7 days  → Hot
30 days → Warm
90 days → Delete
```

The actual retention should be based on organizational requirements.

---

# 48. Storage Architecture

Elasticsearch storage must be planned carefully.

```text
Logs
 ↓
Elasticsearch
 ↓
Persistent Storage
```

On Kubernetes:

```text
Elasticsearch Pod
       ↓
Persistent Volume
       ↓
Storage Class
       ↓
Underlying Storage
```

Never treat Elasticsearch data as disposable container filesystem data in production.

---

# 49. Stateful Elasticsearch

Elasticsearch is stateful.

Therefore Kubernetes deployments need:

```text
Persistent Volumes
Stable identity
Appropriate StatefulSet/operator architecture
Storage planning
```

A production Elasticsearch deployment should use an architecture designed for stateful workloads.

---

# 50. StatefulSet Concept

Conceptually:

```text
Elasticsearch
     ↓
StatefulSet
     ↓
┌────┬────┬────┐
↓    ↓    ↓
ES-0 ES-1 ES-2
│    │    │
PV   PV   PV
```

Each instance maintains persistent storage.

---

# 51. Elasticsearch Storage Failure

If a Pod restarts:

```text
ES-0
 X
 ↓
New ES-0
 ↓
Same persistent storage
```

Persistent storage allows the node to recover its local data where appropriate.

However, persistence is not a substitute for Elasticsearch replicas and backups.

---

# 52. Elasticsearch Backup

Production Elasticsearch should have a backup strategy.

Backups can be used for:

```text
Disaster recovery
Accidental deletion
Corruption recovery
Migration
Compliance
```

The exact backup mechanism depends on the Elasticsearch deployment.

---

# 53. Snapshot Architecture

Conceptually:

```text
Elasticsearch
      ↓
Snapshot
      ↓
Object Storage
```

For AWS environments, object storage such as Amazon S3 is commonly used for snapshots where supported by the chosen Elasticsearch deployment.

---

# 54. Snapshot vs Replica

A replica:

```text
Protects against some node failures
```

A snapshot:

```text
Protects against broader recovery scenarios
```

Therefore:

```text
Replicas
+
Snapshots
```

should be considered separately.

---

# 55. Elasticsearch Monitoring

Monitor:

```text
Cluster health
Node health
CPU
Memory
JVM heap
Disk
Shard allocation
Indexing rate
Search latency
Rejected requests
```

Prometheus can be used to collect Elasticsearch metrics through an appropriate exporter/integration.

---

# 56. Logstash Monitoring

Monitor:

```text
Events received
Events processed
Pipeline throughput
Pipeline latency
Queue size
Dropped events
Processing errors
CPU
Memory
```

A Logstash bottleneck can cause ingestion delays.

---

# 57. Kibana Monitoring

Monitor:

```text
Availability
HTTP response codes
Latency
CPU
Memory
Pod restarts
Elasticsearch connectivity
```

If Kibana fails while Elasticsearch remains healthy:

```text
Log ingestion
     ✓

Log storage
     ✓

Log UI
     X
```

---

# 58. End-to-End Monitoring

The entire pipeline should be monitored:

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

A failure at any point can affect the user experience.

---

# 59. Log Pipeline Lag

A useful operational concept is ingestion delay.

Example:

```text
Application generated:
10:00:00

Elasticsearch received:
10:00:03
```

Pipeline delay:

```text
3 seconds
```

If this grows to minutes:

```text
10:00:00 → 10:05:00
```

the pipeline has a serious performance problem.

---

# 60. Backpressure

Backpressure occurs when downstream processing cannot keep up.

Example:

```text
Application
   ↓
Collector
   ↓
Logstash
   ↓
Elasticsearch
   X
```

If Elasticsearch slows down:

```text
Logstash queue
     ↑
     │
Events accumulate
```

Monitor queue growth.

---

# 61. Buffering

Buffering can protect against temporary downstream problems.

```text
Collector
   ↓
Buffer
   ↓
Logstash
   ↓
Buffer
   ↓
Elasticsearch
```

Buffering should have capacity limits and operational monitoring.

It does not eliminate the need to fix a permanently overloaded backend.

---

# 62. High Log Volume

Suppose:

```text
500 GB/day
```

of logs are generated.

The architecture must account for:

```text
500 GB ingestion
+
Replication
+
Index overhead
+
Retention
+
Search workload
```

Therefore capacity planning must happen before production deployment.

---

# 63. Elasticsearch Capacity Planning

Important inputs:

```text
Daily ingestion
Retention period
Replica count
Shard strategy
Growth rate
Query workload
Storage overhead
Failure headroom
```

Example:

```text
200 GB/day
× 30 days
= 6 TB raw
```

Then add replication and operational overhead.

---

# 64. Logstash Capacity Planning

Consider:

```text
Events per second
Average event size
Parsing complexity
Grok usage
CPU
Memory
Pipeline workers
Queue size
```

Complex Grok patterns can consume significant CPU.

Prefer structured JSON when possible.

---

# 65. JSON vs Grok

Structured:

```json
{
  "service": "payment",
  "level": "ERROR",
  "message": "Payment failed"
}
```

Usually easier to process.

Unstructured:

```text
2026-08-11 payment ERROR Payment failed
```

May require Grok or other parsing.

Therefore:

```text
Application structured logging
        ↓
Less parsing complexity
        ↓
Simpler pipeline
```

---

# 66. Failure Scenario: Collector Down

Suppose:

```text
Node 1
 └── Collector X
```

Logs may not reach Logstash from that node.

A DaemonSet automatically attempts to keep the collector scheduled.

Check:

```text
kubectl get daemonset -n logging
kubectl get pods -n logging
```

---

# 67. Failure Scenario: Logstash Down

If:

```text
Logstash A
   X
```

and:

```text
Logstash B
   ✓
```

the collector should be configured to use the available Logstash endpoint(s) according to the chosen topology.

---

# 68. Failure Scenario: Elasticsearch Node Down

Example:

```text
ES1
 X

ES2
 ✓

ES3
 ✓
```

Check:

```text
Cluster health
Shard allocation
Replica status
Node availability
Disk
Network
```

Do not immediately restart everything.

---

# 69. Failure Scenario: Elasticsearch Cluster Red

Troubleshooting:

```text
1. Check cluster health
2. Identify unassigned shards
3. Check node health
4. Check disk
5. Check allocation explanation
6. Check recent changes
7. Recover affected nodes/shards
```

Avoid destructive actions unless the recovery procedure explicitly requires them.

---

# 70. Failure Scenario: Kibana Down

Check:

```text
kubectl get pods -n logging
kubectl logs <kibana-pod> -n logging
kubectl get svc -n logging
kubectl get ingress -n logging
```

Then verify:

```text
Kibana → Elasticsearch connectivity
```

---

# 71. Failure Scenario: Logs Missing

Use the pipeline:

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

At every stage verify:

```text
Data entering?
Data leaving?
Errors?
```

This is the fastest troubleshooting method.

---

# 72. ELK Security Architecture

```text
                      Users
                        │
                        ↓
                      ALB
                        │
                        ↓
                     Kibana
                        │
                  Authentication
                        │
                     RBAC
                        │
                        ↓
              Elasticsearch Cluster
                        ↑
                        │
                    Logstash
                        ↑
                        │
                Log Collectors
```

Elasticsearch should remain private.

---

# 73. Network Segmentation

A good architecture separates:

```text
Public Layer
    ↓
ALB

Application Layer
    ↓
Collectors / Logstash

Data Layer
    ↓
Elasticsearch
```

Only required traffic should be allowed.

---

# 74. Example Network Rules

Conceptually:

```text
ALB → Kibana
Kibana → Elasticsearch
Logstash → Elasticsearch
Collectors → Logstash
```

Avoid:

```text
Internet → Elasticsearch
Internet → Logstash
```

unless there is an explicitly secured architecture requiring it.

---

# 75. TLS Between Components

For security-sensitive environments:

```text
Collector
   ↓ TLS
Logstash
   ↓ TLS
Elasticsearch
   ↑ TLS
Kibana
```

TLS protects data in transit.

---

# 76. Secrets

Credentials may include:

```text
Elasticsearch credentials
Kibana credentials
TLS certificates
API keys
```

Do not store these directly in Git.

Use:

```text
Kubernetes Secrets
AWS Secrets Manager
External Secrets
Other approved secret-management systems
```

---

# 77. Sensitive Log Data

Logging systems may contain highly sensitive information.

Examples:

```text
Authentication events
User information
Internal IP addresses
Application errors
Request information
Security events
```

Apply:

```text
RBAC
Retention controls
Encryption
Audit controls
Data masking
```

---

# 78. Log Retention and Compliance

Retention should be defined by:

```text
Business requirements
Security requirements
Compliance
Storage cost
Incident investigation requirements
```

Do not simply keep every log forever.

---

# 79. ELK With GitOps

For your EKS environment:

```text
GitHub
   ↓
Configuration
   ↓
GitHub Actions
   ↓
Validation / Security Checks
   ↓
ArgoCD
   ↓
EKS
   ↓
ELK Components
```

This makes deployment reproducible.

---

# 80. Repository Structure

A practical GitOps structure could be:

```text
logging/
│
├── elasticsearch/
│   ├── values.yaml
│   └── manifests/
│
├── logstash/
│   ├── values.yaml
│   └── pipelines/
│
├── kibana/
│   ├── values.yaml
│   └── ingress/
│
├── collectors/
│   └── fluent-bit/
│
└── argocd/
    └── applications/
```

This keeps components separated rather than placing everything in one folder.

---

# 81. Environment Structure

You can further separate:

```text
logging/
├── dev/
├── staging/
└── prod/
```

Example:

```text
logging/
├── dev/
│   ├── elasticsearch-values.yaml
│   ├── logstash-values.yaml
│   └── kibana-values.yaml
│
├── staging/
│   ├── elasticsearch-values.yaml
│   ├── logstash-values.yaml
│   └── kibana-values.yaml
│
└── prod/
    ├── elasticsearch-values.yaml
    ├── logstash-values.yaml
    └── kibana-values.yaml
```

---

# 82. Deployment Flow

```text
Developer
    ↓
Modify logging configuration
    ↓
Git Pull Request
    ↓
Review
    ↓
GitHub Actions
    ↓
Validation / Security
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

# 83. Production Upgrade

Upgrade components independently where possible.

Example:

```text
Elasticsearch
    ↓
Test

Logstash
    ↓
Test

Kibana
    ↓
Test
```

Always verify version compatibility across the stack.

Do not blindly upgrade all components simultaneously.

---

# 84. Upgrade Checklist

Before upgrade:

```text
[ ] Check compatibility
[ ] Backup / snapshot
[ ] Test in staging
[ ] Review release notes
[ ] Validate configuration
[ ] Validate plugins
[ ] Prepare rollback
```

After upgrade:

```text
[ ] Cluster healthy
[ ] Logs flowing
[ ] Indexing working
[ ] Kibana accessible
[ ] Dashboards working
[ ] Searches working
[ ] Alerts working
```

---

# 85. Disaster Recovery

A production logging platform needs a recovery strategy.

```text
Elasticsearch
      ↓
Snapshots
      ↓
Object Storage
```

Configuration:

```text
Git
 ↓
ELK configuration
```

Recovery:

```text
New infrastructure
       ↓
Deploy ELK
       ↓
Restore snapshots
       ↓
Apply configuration
       ↓
Validate
```

---

# 86. Disaster Recovery Architecture

```text
                 Production
                     │
              Elasticsearch
                     │
                  Snapshot
                     │
                     ↓
                Object Storage
                     │
                     ↓
              Disaster Recovery
                     │
                     ↓
            New Elasticsearch
                     │
                     ↓
               Restore Data
```

---

# 87. RPO and RTO

Define:

```text
RPO
```

How much log data can be lost?

And:

```text
RTO
```

How quickly must the logging platform be restored?

Example:

```text
RPO = 1 hour
RTO = 2 hours
```

These are examples only. Production requirements should determine the actual values.

---

# 88. ELK Monitoring With Prometheus

Your monitoring architecture can be:

```text
                 Prometheus
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
      Logstash   Elasticsearch  Kibana
          │          │          │
          └──────────┼──────────┘
                     ↓
                  Grafana
```

Grafana provides operational dashboards for the logging platform.

---

# 89. Complete Observability Architecture

Your final observability platform becomes:

```text
                            EKS
                             │
        ┌────────────────────┼────────────────────┐
        ↓                    ↓                    ↓
     Metrics                Logs                Traces
        │                    │                    │
   Prometheus             ELK Stack             Jaeger
        │                    │                    │
        ↓                    ↓                    ↓
     Grafana               Kibana             Jaeger UI
```

Where:

```text
Prometheus → Metrics
ELK        → Logs
Jaeger     → Traces
```

---

# 90. Metrics + Logs + Traces

Example incident:

```text
Prometheus
    ↓
Payment error rate increased
    ↓
Grafana
    ↓
Kibana
    ↓
Payment ERROR logs
    ↓
trace.id
    ↓
Jaeger
    ↓
Distributed request
    ↓
Root cause
```

This is the real value of an integrated observability architecture.

---

# 91. Production ELK Architecture

A mature production architecture can be represented as:

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
                  ┌────────────┼────────────┐
                  ↓            ↓            ↓
                 ES1          ES2          ES3
                  ↑            ↑            ↑
                  └────────────┼────────────┘
                               ↑
                       Logstash Cluster
                     ┌─────────┼─────────┐
                     ↓         ↓         ↓
                   LS-A      LS-B      LS-C
                     ↑         ↑         ↑
                     └─────────┼─────────┘
                               ↑
                       Log Collector Layer
                     ┌─────────┼─────────┐
                     ↓         ↓         ↓
                   Node 1    Node 2    Node 3
                     │         │         │
                    Pods      Pods      Pods
```

---

# 92. Component Failure Model

| Component             | Failure Impact                           | Production Protection     |
| --------------------- | ---------------------------------------- | ------------------------- |
| Log Collector         | Logs from affected node may stop flowing | DaemonSet / restart       |
| Logstash              | Processing/forwarding interruption       | Multiple replicas         |
| Elasticsearch Node    | Shards may become unavailable            | Replicas + multiple nodes |
| Elasticsearch Cluster | Search/indexing impact                   | HA cluster                |
| Kibana                | UI unavailable                           | Multiple replicas         |
| Storage               | Data availability risk                   | Replication + snapshots   |
| Network               | Pipeline interruption                    | HA networking             |
| Identity Provider     | Login may fail                           | Enterprise IdP HA         |

---

# 93. What Happens During an Application Failure?

Suppose Payment starts returning HTTP 500.

```text
Payment
   ↓
ERROR logs
   ↓
Collector
   ↓
Logstash
   ↓
Elasticsearch
   ↓
Kibana
```

At the same time:

```text
Prometheus
   ↓
5xx metric
   ↓
Grafana Alert
```

And if tracing is enabled:

```text
Jaeger
   ↓
Trace
   ↓
Slow / failed dependency
```

This provides three different perspectives of the same incident.

---

# 94. Architecture Principles

Remember these principles:

```text
1. Collect logs close to the source.
2. Use lightweight collectors.
3. Separate collection from processing.
4. Scale Logstash horizontally.
5. Run Elasticsearch as a distributed cluster.
6. Use persistent storage.
7. Use replicas for resilience.
8. Use snapshots for disaster recovery.
9. Keep Elasticsearch private.
10. Expose Kibana through controlled access.
11. Use TLS.
12. Apply RBAC.
13. Control retention.
14. Monitor the logging pipeline itself.
15. Manage configuration through GitOps.
```

---

# 95. Interview Question: Explain ELK Architecture

A strong answer:

```text
"ELK consists of Elasticsearch, Logstash and Kibana.

In a Kubernetes production environment, applications write logs to
stdout and stderr. A node-level collector such as Fluent Bit or
Filebeat runs as a DaemonSet and collects those logs.

The collector forwards events to a Logstash cluster. Logstash parses,
filters and enriches the events and sends them to an Elasticsearch
cluster.

Elasticsearch stores and indexes the logs across multiple nodes with
appropriate shard and replica configuration.

Kibana connects to Elasticsearch and provides centralized log search,
visualization and dashboards.

For production I would use multiple Logstash and Elasticsearch
instances, persistent storage, controlled retention, security,
backups and monitoring.

The overall flow is:

Application → Collector → Logstash → Elasticsearch → Kibana."
```

---

# 96. Interview Question: Why Do You Need Multiple Elasticsearch Nodes?

```text
"Multiple Elasticsearch nodes allow data and workload to be
distributed across the cluster.

Shards can be distributed across nodes and replicas provide
additional copies.

This improves scalability and resilience because the cluster can
continue operating when an individual node fails, depending on shard
availability and cluster configuration."
```

---

# 97. Interview Question: Why Use Fluent Bit/Filebeat?

```text
"I use a lightweight collector at the node level to collect
container logs.

Running the collector as a Kubernetes DaemonSet ensures each node
has a log collection agent.

The collector can add Kubernetes metadata and forward events to
Logstash.

This separates lightweight collection from heavier log processing."
```

---

# 98. Interview Question: How Do You Handle Elasticsearch Failure?

```text
"I first check cluster health and determine whether the problem is
with a node, shard allocation, disk, memory, network or another
dependency.

I check whether primary or replica shards are affected.

If a node has failed, Elasticsearch can use replica shards where
available.

I would also check disk watermarks, JVM heap and allocation
failures.

For disaster recovery, I rely on tested snapshot backups rather than
treating replicas as a replacement for backups."
```

---

# 99. Interview Question: How Do You Troubleshoot Missing Logs?

```text
"I trace the pipeline from the application to Kibana.

First I verify that the application is actually generating logs.

Then I check the node-level collector, followed by the Logstash
pipeline.

After that I verify Elasticsearch indexing and index health.

Finally I check Kibana's data view and time range.

The troubleshooting path is:

Application → Collector → Logstash → Elasticsearch → Kibana."
```

---

# 100. Final Architecture to Remember

```text
                       APPLICATIONS
                            │
                            ↓
                     stdout / stderr
                            │
                            ↓
                   LOG COLLECTOR
                  Fluent Bit/Filebeat
                            │
                            ↓
                    LOGSTASH CLUSTER
                  ┌─────────┴─────────┐
                  ↓                   ↓
              Logstash A          Logstash B
                  │                   │
                  └─────────┬─────────┘
                            ↓
                 ELASTICSEARCH CLUSTER
                  ┌─────────┼─────────┐
                  ↓         ↓         ↓
                 ES1       ES2       ES3
                  │         │         │
                  └─────────┼─────────┘
                            ↓
                         KIBANA
                            │
                            ↓
                         USERS
```

The complete observability relationship is:

```text
              OBSERVABILITY PLATFORM
                       │
        ┌──────────────┼──────────────┐
        ↓              ↓              ↓
      Metrics         Logs          Traces
        ↓              ↓              ↓
   Prometheus         ELK           Jaeger
        ↓              ↓              ↓
     Grafana         Kibana        Jaeger UI
```

The key production idea is that **ELK is not simply three applications installed on one server**. It is a distributed logging pipeline where collection, processing, storage, search, visualization, security, retention, scaling, and disaster recovery are designed as separate concerns.
