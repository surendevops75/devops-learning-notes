# Production Observability — High Availability

## 1. Introduction

High Availability (HA) in observability means designing the monitoring, logging, alerting, and visualization platform so that the failure of an individual component does not result in complete loss of visibility.

This is a critical production requirement.

A common mistake is to build a highly available application while running observability on a single server.

For example:

```text
Production Application
        |
        v
       EKS
        |
        v
    Applications
        |
        v
   Observability
        |
        v
   Single Prometheus
```

If Prometheus fails during a production incident, the application may continue running, but engineers lose visibility exactly when they need it most.

Therefore:

> **The observability platform itself must be treated as production infrastructure.**

---

# 2. What Does High Availability Mean?

High availability means that a service remains available despite expected component failures.

For observability, this can include:

```text
Prometheus
Grafana
Alertmanager
Elasticsearch
Logstash
Kibana
Storage
Network
Nodes
Availability Zones
```

A highly available architecture should tolerate failures such as:

```text
Server failure
Pod failure
Node failure
Availability Zone failure
Application restart
Network interruption
Storage failure
Software failure
Configuration error
```

The exact level of HA depends on business requirements and the required observability SLO.

---

# 3. Why Observability HA Matters

Imagine a production incident:

```text
10:00 AM
Payment API starts returning 500 errors
```

Engineers open Grafana.

But:

```text
Grafana unavailable
```

They try Prometheus.

```text
Prometheus unavailable
```

They check Kibana.

```text
Elasticsearch unavailable
```

Now the application is failing and the monitoring platform is also failing.

This creates a much more serious operational problem:

```text
Application Incident
        +
Observability Failure
        =
Blind Production Incident
```

This is why observability HA is important.

---

# 4. Availability vs High Availability

These terms are related but not identical.

### Availability

A component is available when it is functioning and reachable.

Example:

```text
Prometheus
   |
   v
Running
```

### High Availability

A service continues functioning when one or more components fail.

Example:

```text
Prometheus-1
Prometheus-2

Prometheus-1 fails

Prometheus-2 continues
```

Therefore:

```text
Availability:
"Is it running?"

High Availability:
"Does it continue working when something fails?"
```

---

# 5. Single Point of Failure

A Single Point of Failure (SPOF) is a component whose failure can cause the entire service to become unavailable.

Example:

```text
                Grafana
                   |
                   v
              Prometheus
                   |
                   v
              Single Node
```

If the node fails:

```text
Node failure
     |
     v
Prometheus failure
     |
     v
No metrics
     |
     v
Grafana has no current data
```

The node is therefore a SPOF.

---

# 6. Basic HA Architecture

A simple HA metrics architecture can look like:

```text
                    Applications
                         |
                         v
                  +-------------+
                  | Kubernetes  |
                  +-------------+
                    |         |
                    v         v
              Prometheus-1  Prometheus-2
                    |         |
                    +----+----+
                         |
                         v
                    Query Layer
                         |
                         v
                      Grafana
```

Both Prometheus instances collect the same metrics.

If one instance fails:

```text
Prometheus-1
     X

Prometheus-2
     |
     v
Continues monitoring
```

However, this architecture introduces additional considerations such as duplicate data, querying, alert deduplication, and storage.

---

# 7. HA Is More Than Running Two Replicas

A common misconception is:

> "If I deploy two Prometheus replicas, I have solved HA."

Not necessarily.

You must consider:

```text
Compute
Storage
Networking
Data consistency
Querying
Alerting
Deduplication
Failure detection
Recovery
Backups
```

For example:

```text
Prometheus-1
     |
     v
Local storage

Prometheus-2
     |
     v
Local storage
```

If Prometheus-1 fails, Prometheus-2 may continue collecting data, but the historical data stored only on Prometheus-1 may not automatically exist on Prometheus-2.

Therefore:

> **HA and durability are different problems.**

---

# 8. HA vs Durability

### High Availability

Answers:

> Can I continue accessing the service?

### Durability

Answers:

> Will my data survive a failure?

Example:

```text
Prometheus-1
   |
   +--> Metrics data

Prometheus-1 fails
```

If the metrics were only stored locally:

```text
Service may recover
but
historical data may be lost
```

Therefore production architecture should consider:

```text
Availability
+
Durability
+
Recovery
```

---

# 9. HA Architecture Layers

A production observability platform can be divided into:

```text
                    Users / Engineers
                           |
                           v
                    Access Layer
                           |
                           v
                  Visualization Layer
                           |
                           v
                   Query / Data Layer
                           |
                           v
                  Collection Layer
                           |
                           v
                   Infrastructure
                           |
                           v
                       Storage
```

Each layer must be evaluated for SPOFs.

---

# 10. Observability HA Architecture for EKS

For an EKS environment:

```text
                         AWS
                          |
                    Availability Zone A
                          |
                     EKS Node 1
                          |
              +-----------+-----------+
              |                       |
              v                       v
        Prometheus-1              Grafana-1
              |
              |
                    Availability Zone B
                          |
                     EKS Node 2
                          |
              +-----------+-----------+
              |                       |
              v                       v
        Prometheus-2              Grafana-2
```

The goal is to avoid placing all observability components on the same node or availability zone.

---

# 11. Availability Zones

AWS production environments commonly use multiple Availability Zones.

Example:

```text
                    AWS Region
                         |
          +--------------+--------------+
          |                             |
          v                             v
        AZ-A                          AZ-B
          |                             |
       EKS Nodes                     EKS Nodes
          |                             |
    Observability                 Observability
```

If AZ-A becomes unavailable:

```text
AZ-A
  X

AZ-B
  |
  v
Observability continues
```

This is stronger than simply running multiple pods on one node.

---

# 12. Kubernetes Scheduling and HA

Kubernetes can distribute replicas across nodes.

For example:

```yaml
replicas: 2
```

does not automatically guarantee that replicas are in different Availability Zones.

You need scheduling policies such as:

```text
Pod Anti-Affinity
Topology Spread Constraints
Node Affinity
Taints and Tolerations
```

These help control placement.

---

# 13. Pod Anti-Affinity

Suppose we have:

```text
Prometheus-1
Prometheus-2
```

Without anti-affinity:

```text
Node-1
  |
  +-- Prometheus-1
  +-- Prometheus-2
```

If Node-1 fails:

```text
Both replicas lost
```

With anti-affinity:

```text
Node-1
  |
  +-- Prometheus-1

Node-2
  |
  +-- Prometheus-2
```

Now:

```text
Node-1 fails
     |
     v
Prometheus-2 continues
```

---

# 14. Topology Spread Constraints

Topology spread constraints allow Kubernetes workloads to be distributed across topology domains.

Possible topology domains include:

```text
Nodes
Availability Zones
Regions
```

Conceptually:

```text
AZ-A
  |
  +-- Prometheus-1

AZ-B
  |
  +-- Prometheus-2

AZ-C
  |
  +-- Prometheus-3
```

This reduces correlated failures.

---

# 15. Prometheus HA

Prometheus is fundamentally designed around a pull-based monitoring model.

A simplified flow:

```text
Targets
   |
   | scrape
   v
Prometheus
   |
   v
Local TSDB
   |
   v
PromQL
```

For HA:

```text
Targets
   |
   +------------+
   |            |
   v            v
Prometheus-1  Prometheus-2
   |            |
   +------+-----+
          |
          v
       Query
```

Both Prometheus servers can independently scrape the same targets.

---

# 16. Why Run Multiple Prometheus Instances?

Reasons include:

### Failure tolerance

If one Prometheus instance fails, another continues scraping.

### Maintenance

One instance can be restarted while another remains available.

### Upgrade safety

You can perform controlled upgrades.

### Reduced monitoring blind spots

A single Prometheus failure does not eliminate all metric collection.

---

# 17. Prometheus HA Challenge — Duplicate Data

Suppose both instances scrape:

```text
http_requests_total
```

They both produce:

```text
service="order"
instance="pod-1"
```

The query system may see duplicate series.

This can create:

```text
Duplicate metrics
Incorrect graphs
Incorrect aggregations
Duplicate alerts
```

Therefore HA Prometheus requires a strategy for handling replicas.

---

# 18. Prometheus HA Strategies

Common approaches include:

```text
1. Independent replicas
2. Query federation
3. Remote storage
4. External query layer
5. Long-term metrics platform
```

The exact design depends on scale and operational requirements.

---

# 19. Independent Prometheus Replicas

The simplest architecture:

```text
                Targets
                /     \
               /       \
              v         v
       Prometheus-1  Prometheus-2
```

Advantages:

```text
Simple
Easy to understand
Independent failure domains
```

Challenges:

```text
Duplicate data
Duplicate alerts
Separate local storage
Historical data differences
```

This can work for smaller environments, but large production environments often need an additional query/storage architecture.

---

# 20. Prometheus Remote Storage

Prometheus can send metrics to a remote storage system.

Conceptually:

```text
Prometheus
    |
    v
Remote Write
    |
    v
Long-Term Metrics Storage
```

This provides separation between:

```text
Collection
```

and:

```text
Long-term storage/querying
```

This becomes useful when retention, scalability, or HA requirements exceed local Prometheus storage.

---

# 21. Prometheus HA with External Query Layer

A more scalable architecture can look like:

```text
                    Grafana
                       |
                       v
                  Query Layer
                 /     |     \
                /      |      \
               v       v       v
             P-1      P-2     P-3
               \       |      /
                \      |     /
                 v     v    v
                  Metrics
```

The query layer can provide:

```text
Replica handling
Query routing
Aggregation
Deduplication
Long-term access
```

The exact technology depends on the organization's chosen architecture.

---

# 22. Prometheus Storage Considerations

Prometheus stores time-series data in its TSDB.

Important production considerations:

```text
Retention
Disk capacity
Write rate
Series count
Cardinality
Compaction
Disk performance
```

If storage fills:

```text
Prometheus
    |
    v
Disk 100%
    |
    v
Write failures
    |
    v
Missing metrics
```

Therefore storage monitoring is part of Prometheus HA.

---

# 23. Prometheus Disk Failure

Suppose:

```text
Prometheus-1
    |
    v
Disk full
```

Potential consequences:

```text
Failed writes
Compaction problems
Unavailable historical data
Prometheus instability
```

If Prometheus-2 exists:

```text
Prometheus-1
    X

Prometheus-2
    |
    v
Continues collection
```

This reduces monitoring downtime.

---

# 24. Prometheus Retention Strategy

Retention should be designed based on:

```text
Required historical analysis
Storage capacity
Query workload
Cost
Compliance requirements
```

Do not simply configure extremely long retention on local disks.

Instead consider:

```text
Short local retention
+
Long-term remote storage
```

when long historical analysis is required.

---

# 25. Grafana HA

Grafana is generally the visualization and dashboard layer.

Basic architecture:

```text
Users
  |
  v
Load Balancer
  |
  +--------+
  |        |
  v        v
Grafana-1 Grafana-2
    \       /
     \     /
      Data Sources
```

If Grafana-1 fails:

```text
Grafana-2
   |
   v
Users continue accessing dashboards
```

---

# 26. Grafana HA Requirements

Running multiple Grafana replicas introduces shared-state considerations.

Grafana may contain:

```text
Users
Organizations
Dashboards
Data source configuration
Alert configuration
Preferences
```

Therefore production HA should consider how Grafana state is shared.

A common architecture uses:

```text
Grafana replicas
       |
       v
Shared database
```

rather than treating each Grafana pod as completely independent.

---

# 27. Grafana Database

For production deployments, Grafana can use an external relational database for shared configuration/state.

Conceptually:

```text
Grafana-1
     |
     +------+
            |
Grafana-2 --+--> Shared Database
            |
Grafana-3 --+
```

This allows multiple Grafana instances to work with common state.

The database itself then becomes part of the HA design.

---

# 28. Grafana Behind a Load Balancer

Example:

```text
                   Users
                     |
                     v
                  ALB / LB
                     |
             +-------+-------+
             |               |
             v               v
         Grafana-1       Grafana-2
             |               |
             +-------+-------+
                     |
                     v
                 Prometheus
```

Benefits:

```text
Load distribution
Failure tolerance
Centralized access
```

---

# 29. Grafana Session Considerations

When running multiple Grafana replicas, session handling and shared state must be considered.

A user might access:

```text
Grafana-1
```

and later:

```text
Grafana-2
```

The architecture must ensure authentication/session behavior works correctly across replicas.

This is another reason production HA is more than simply increasing:

```text
replicas: 2
```

---

# 30. Alertmanager HA

Alertmanager handles alerts generated by Prometheus.

Basic flow:

```text
Prometheus
     |
     v
Alert Rule
     |
     v
Alertmanager
     |
     +---- Email
     +---- Slack
     +---- Incident System
```

If Alertmanager is a single instance:

```text
Alertmanager failure
        |
        v
Notification failure
```

The alert may exist in Prometheus but fail to reach engineers.

---

# 31. Alertmanager HA Architecture

Example:

```text
             Prometheus-1
                  |
             Prometheus-2
                  |
            +-----+-----+
            |           |
            v           v
     Alertmanager-1  Alertmanager-2
            \           /
             \         /
              +-------+
                  |
                  v
            Notification
```

Alertmanager instances can form a cluster to coordinate alert state and avoid duplicate notifications.

---

# 32. Why Alert Deduplication Matters

Suppose two Prometheus replicas detect:

```text
High error rate
```

Without proper coordination:

```text
Prometheus-1 → Alertmanager → Pager
Prometheus-2 → Alertmanager → Pager
```

An engineer might receive two pages for the same incident.

HA alerting should therefore support:

```text
Deduplication
Grouping
Silencing
Routing
Inhibition
```

---

# 33. Alert Grouping

Suppose 20 pods fail.

Without grouping:

```text
Alert 1
Alert 2
Alert 3
...
Alert 20
```

With grouping:

```text
Production
Order Service
High Error Rate
```

One notification can represent the broader incident.

This reduces alert fatigue.

---

# 34. Elasticsearch HA

Elasticsearch is distributed by design.

A production Elasticsearch cluster can contain:

```text
Node-1
Node-2
Node-3
```

Conceptually:

```text
                 Elasticsearch Cluster
                /        |        \
               /         |         \
              v          v          v
           Node-1      Node-2     Node-3
```

Data is distributed across shards and replicas.

---

# 35. Elasticsearch Shards

An Elasticsearch index is divided into shards.

Conceptually:

```text
Index
 |
 +-- Primary Shard 1
 +-- Primary Shard 2
 +-- Primary Shard 3
```

This allows Elasticsearch to distribute data and search operations.

---

# 36. Elasticsearch Replica Shards

A replica provides another copy of a shard.

Example:

```text
Primary Shard
      |
      +---- Replica
```

If the node holding the primary fails:

```text
Primary
   X

Replica
   |
   v
Promoted / available
```

This improves availability.

---

# 37. Elasticsearch HA Architecture

A simplified production architecture:

```text
                   Logstash
                      |
                      v
              Elasticsearch
             /      |       \
            /       |        \
           v        v         v
        ES-1      ES-2      ES-3
           \        |        /
            \       |       /
             +-----+------+
                   |
                   v
                 Kibana
```

Kibana queries the Elasticsearch cluster.

---

# 38. Elasticsearch Availability Zone Awareness

For production AWS environments:

```text
AZ-A
  |
  +-- ES-1

AZ-B
  |
  +-- ES-2

AZ-C
  |
  +-- ES-3
```

This reduces the possibility that one AZ failure removes the entire Elasticsearch cluster.

Shard allocation should also be designed to avoid placing all copies of a shard in the same failure domain.

---

# 39. Elasticsearch Storage

Elasticsearch is highly dependent on disk performance.

Important metrics include:

```text
Disk usage
Disk I/O
Shard count
Index size
Heap usage
JVM pressure
Search latency
Indexing latency
Cluster health
```

If disk usage becomes too high:

```text
Elasticsearch
      |
      v
Disk pressure
      |
      v
Shard allocation problems
      |
      v
Indexing issues
      |
      v
Log ingestion problems
```

---

# 40. Elasticsearch Cluster Health

Important cluster health states include:

```text
Green
Yellow
Red
```

### Green

All primary and replica shards are allocated.

### Yellow

All primary shards are available, but one or more replicas are not allocated.

### Red

One or more primary shards are unavailable.

A production engineer should investigate yellow/red states rather than ignoring them.

---

# 41. Logstash HA

Logstash can also become a SPOF.

Single instance:

```text
Applications
     |
     v
Logstash
     |
     v
Elasticsearch
```

If Logstash fails:

```text
Log ingestion stops
```

A more resilient architecture:

```text
Applications
     |
     v
Load Balancer / Log Pipeline
     |
     +----------+
     |          |
     v          v
Logstash-1  Logstash-2
     |          |
     +-----+----+
           |
           v
    Elasticsearch
```

---

# 42. Logstash Pipeline Design

Logstash pipelines commonly include:

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
      |
      v
Input
      |
      v
Parse / Transform
      |
      v
Elasticsearch
```

For HA, multiple Logstash instances can process the incoming log stream.

---

# 43. Log Loss Consideration

High availability does not automatically guarantee zero log loss.

Consider:

```text
Application
    |
    v
Logstash
    X
```

If the log pipeline fails and there is no buffering:

```text
Logs may be lost
```

Production architectures may use buffering or durable queues depending on requirements.

Therefore:

```text
HA
+
Buffering
+
Durable storage
```

should be considered for critical logging pipelines.

---

# 44. Kibana HA

Kibana is primarily the visualization/query interface for Elasticsearch.

Basic HA architecture:

```text
Users
  |
  v
Load Balancer
  |
  +---------+
  |         |
  v         v
Kibana-1 Kibana-2
  |         |
  +----+----+
       |
       v
Elasticsearch
```

If one Kibana instance fails, traffic can be sent to another.

---

# 45. Complete HA Observability Architecture

A production-grade architecture can look like:

```text
                              USERS
                                |
                                v
                         Load Balancer
                                |
                     +----------+----------+
                     |                     |
                     v                     v
                 Grafana-1             Grafana-2
                     |                     |
                     +----------+----------+
                                |
                         Query / Metrics
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
               Prometheus-1           Prometheus-2
                    |                       |
                    +-----------+-----------+
                                |
                         Long-Term Storage
```

Logging:

```text
Applications / EKS
        |
        v
   Log Collection
        |
   +----+----+
   |         |
   v         v
Logstash-1 Logstash-2
   |         |
   +----+----+
        |
        v
 Elasticsearch Cluster
   |       |       |
   v       v       v
 ES-1    ES-2    ES-3
        |
        v
   +----+----+
   |         |
   v         v
Kibana-1  Kibana-2
```

Alerting:

```text
Prometheus-1
      |
Prometheus-2
      |
      v
Alertmanager Cluster
      |
      v
Notification Channels
```

---

# 46. End-to-End Production HA Architecture

Combining metrics and logs:

```text
                              USERS
                                |
                                v
                          AWS ALB / LB
                                |
                                v
                              EKS
                                |
          +---------------------+---------------------+
          |                     |                     |
          v                     v                     v
     Application A        Application B        Application C
          |                     |                     |
          |                     |                     |
       Metrics                Metrics                Metrics
          |                     |                     |
          +---------------------+---------------------+
                                |
                    +-----------+-----------+
                    |                       |
                    v                       v
              Prometheus-1           Prometheus-2
                    |                       |
                    +-----------+-----------+
                                |
                                v
                         Query / Storage
                                |
                                v
                         Grafana Cluster


Application / Container Logs
             |
             v
       Log Pipeline
             |
       +-----+-----+
       |           |
       v           v
  Logstash-1   Logstash-2
       |           |
       +-----+-----+
             |
             v
    Elasticsearch Cluster
       |       |       |
       v       v       v
     ES-1    ES-2    ES-3
             |
             v
      Kibana Cluster
```

---

# 47. Failure Scenario — Prometheus-1 Fails

Initial state:

```text
Prometheus-1   HEALTHY
Prometheus-2   HEALTHY
```

Failure:

```text
Prometheus-1
      X
```

Expected behavior:

```text
Prometheus-2
      |
      v
Continues scraping
```

Engineers should still have access to current metrics.

After recovery:

```text
Prometheus-1
      |
      v
Restarts
      |
      v
Resumes monitoring
```

The exact historical data behavior depends on the storage architecture.

---

# 48. Failure Scenario — Grafana-1 Fails

Initial:

```text
ALB
 |
 +-- Grafana-1
 +-- Grafana-2
```

Failure:

```text
Grafana-1
    X
```

Load balancer health checks detect the unhealthy instance.

Traffic goes to:

```text
Grafana-2
```

Users continue accessing dashboards.

---

# 49. Failure Scenario — Elasticsearch Node Failure

Initial:

```text
ES-1
ES-2
ES-3
```

Suppose:

```text
ES-2 fails
```

If shard replicas are correctly configured:

```text
ES-1
ES-3
   |
   v
Cluster continues serving
```

The cluster may temporarily become yellow while Elasticsearch reallocates replicas.

After recovery:

```text
ES-2 rejoins
      |
      v
Shard recovery
      |
      v
Cluster returns healthy
```

---

# 50. Failure Scenario — Logstash Failure

Initial:

```text
Applications
      |
 +----+----+
 |         |
 v         v
L-1       L-2
```

If:

```text
L-1 fails
```

traffic should continue through:

```text
L-2
```

If a load balancer or durable queue is present, the impact can be reduced further.

---

# 51. Failure Scenario — Entire Availability Zone Failure

This is a more advanced production scenario.

Suppose:

```text
AZ-A
 |
 +-- Prometheus-1
 +-- Grafana-1
 +-- ES-1
```

and:

```text
AZ-B
 |
 +-- Prometheus-2
 +-- Grafana-2
 +-- ES-2
```

AZ-A fails:

```text
AZ-A
 X
```

AZ-B continues:

```text
Prometheus-2
Grafana-2
ES-2
```

However, whether the overall system remains fully functional depends on:

```text
Replica count
Quorum requirements
Storage
Load balancing
Shard replicas
Network
Database availability
```

---

# 52. Quorum and Distributed Systems

Distributed systems often use quorum concepts.

For example, with three nodes:

```text
Node-1
Node-2
Node-3
```

A majority is:

```text
2 of 3
```

If one node fails:

```text
Node-1
Node-2
Node-3 X
```

the remaining two may still maintain a quorum.

This concept is especially important when designing distributed storage and coordination systems.

---

# 53. Monitoring the Monitoring System

One of the most important production practices is:

> **Monitor the observability platform itself.**

You should monitor:

```text
Prometheus
Grafana
Alertmanager
Elasticsearch
Logstash
Kibana
```

For Prometheus:

```text
Scrape failures
Target count
Storage usage
TSDB health
Rule evaluation
Query latency
Memory usage
CPU usage
```

For Elasticsearch:

```text
Cluster health
Heap usage
Disk usage
Shard health
Indexing rate
Search latency
Node availability
```

For Logstash:

```text
Events in
Events out
Pipeline errors
Queue size
CPU
Memory
```

---

# 54. Meta-Monitoring

Meta-monitoring means monitoring the monitoring system.

Example:

```text
Production Application
       |
       v
Prometheus
       |
       v
Grafana
```

But who monitors Prometheus?

You need an external mechanism or independent monitoring layer for critical failures.

Conceptually:

```text
Primary Observability
        |
        v
Monitoring

Independent Monitoring
        |
        v
Detects Primary Monitoring Failure
```

This prevents:

```text
Monitoring failure
      ↓
No monitoring
      ↓
Nobody knows monitoring failed
```

---

# 55. Observability HA and Backup

HA does not replace backup.

For example:

```text
Grafana
  |
  v
Shared Database
```

If the database is corrupted:

```text
All Grafana replicas
       |
       v
Affected
```

Therefore backups should cover critical configuration and data.

Potential backup targets include:

```text
Grafana database
Grafana configuration
Dashboard definitions
Alert rules
Elasticsearch snapshots
Configuration repositories
Infrastructure-as-Code
```

---

# 56. Configuration as Code

Production observability configuration should preferably be version controlled.

Examples:

```text
Prometheus configuration
Alert rules
Grafana dashboards
Logstash pipelines
Kubernetes manifests
Helm values
Terraform configuration
```

Architecture:

```text
Git
 |
 +-- Prometheus Config
 +-- Alert Rules
 +-- Grafana
 +-- Logstash
 +-- Kubernetes
 |
 v
CI/CD / GitOps
 |
 v
Production
```

This provides:

```text
Version history
Rollback
Peer review
Auditability
Reproducibility
```

---

# 57. Kubernetes Deployment Strategy

Observability components should be deployed using production-safe Kubernetes configurations.

Typical considerations:

```text
Deployment / StatefulSet
Resource requests
Resource limits
PodDisruptionBudget
Anti-affinity
Topology spread
Persistent volumes
Readiness probes
Liveness probes
SecurityContext
Service accounts
```

---

# 58. PodDisruptionBudget

A PodDisruptionBudget can help ensure that voluntary disruptions do not remove too many replicas simultaneously.

Example concept:

```text
Prometheus replicas = 3
```

You may configure availability requirements so that enough replicas remain during planned maintenance.

This is especially important during:

```text
Node drain
Cluster upgrade
Maintenance
Autoscaling
```

---

# 59. Resource Requests and Limits

Observability components can consume significant resources.

For example:

```text
Prometheus
```

may consume high memory because of:

```text
Large active series count
High cardinality
Long retention
Heavy queries
Many scrape targets
```

Elasticsearch can also consume significant:

```text
CPU
Memory
Disk
I/O
```

Therefore resource requests and limits should be based on actual workload measurements.

---

# 60. Capacity Planning for HA

Adding replicas increases resilience but also increases resource consumption.

Example:

```text
1 Prometheus
   |
   v
100% baseline

2 Prometheus
   |
   v
Approximately 2x collection workload
```

The exact overhead depends on architecture.

Similarly:

```text
3 Elasticsearch nodes
```

require more:

```text
CPU
Memory
Storage
Network
```

HA therefore requires capacity planning.

---

# 61. HA and Cost

High availability increases cost.

Example:

```text
Single Prometheus
      ↓
Low cost
High risk

Multiple Prometheus replicas
      ↓
Higher cost
Lower risk
```

The correct architecture should balance:

```text
Reliability
+
Business impact
+
Cost
+
Operational complexity
```

Do not blindly make every component maximally redundant.

---

# 62. Production HA Decision Matrix

Consider:

| Component     | SPOF Risk | HA Requirement |
| ------------- | --------- | -------------- |
| Prometheus    | High      | High           |
| Grafana       | Medium    | High           |
| Alertmanager  | High      | High           |
| Elasticsearch | Very High | High           |
| Logstash      | Medium    | High           |
| Kibana        | Medium    | Medium/High    |
| Storage       | Very High | Very High      |

The exact requirement depends on the criticality of the environment.

---

# 63. Observability HA vs Application HA

Application HA:

```text
Application
   |
   +-- Replica 1
   +-- Replica 2
   +-- Replica 3
```

Observability HA:

```text
Metrics
   |
   +-- Prometheus-1
   +-- Prometheus-2

Logging
   |
   +-- Logstash-1
   +-- Logstash-2
   +-- Elasticsearch Cluster

Visualization
   |
   +-- Grafana-1
   +-- Grafana-2
```

Both must be designed.

A highly available application with unavailable monitoring is still an operational risk.

---

# 64. Disaster Scenario — Monitoring Platform Completely Lost

Suppose:

```text
Prometheus
Grafana
Elasticsearch
Kibana
```

are all unavailable.

Questions to answer:

```text
How do we detect application failure?
How do we receive alerts?
How do we recover observability?
Where are configurations stored?
Where are backups?
How quickly can the platform be rebuilt?
```

This leads directly into disaster recovery planning.

---

# 65. Recovery Through Infrastructure as Code

If the observability platform is defined using:

```text
Terraform
Helm
Kubernetes manifests
Git
```

you can rebuild infrastructure more reliably.

Example:

```text
Git Repository
      |
      v
Terraform
      |
      v
AWS Infrastructure
      |
      v
EKS
      |
      v
Helm
      |
      v
Observability Stack
```

This is much safer than manually rebuilding servers under pressure.

---

# 66. Production Deployment Strategy

A safe HA observability deployment should follow:

```text
Development
    ↓
Testing
    ↓
Staging
    ↓
Production
    ↓
Health verification
```

After deployment:

```text
Check Pods
Check Services
Check Storage
Check Metrics
Check Logs
Check Alerts
Check Dashboards
```

---

# 67. HA Troubleshooting Method

When an observability component becomes unavailable:

## Step 1 — Check Kubernetes

```bash
kubectl get pods -n monitoring
```

## Step 2 — Check pod details

```bash
kubectl describe pod <pod> -n monitoring
```

## Step 3 — Check logs

```bash
kubectl logs <pod> -n monitoring
```

## Step 4 — Check services

```bash
kubectl get svc -n monitoring
```

## Step 5 — Check endpoints

```bash
kubectl get endpoints -n monitoring
```

## Step 6 — Check nodes

```bash
kubectl get nodes
```

## Step 7 — Check events

```bash
kubectl get events -n monitoring --sort-by=.lastTimestamp
```

## Step 8 — Check resources

```bash
kubectl top pods -n monitoring
kubectl top nodes
```

---

# 68. Prometheus HA Troubleshooting

If Prometheus is unavailable:

```text
1. Check pod status
2. Check container logs
3. Check PVC
4. Check disk usage
5. Check memory
6. Check TSDB
7. Check scrape targets
8. Check configuration
9. Check rule evaluation
10. Check network connectivity
```

Useful commands:

```bash
kubectl get pods -n monitoring
kubectl describe pod <prometheus-pod> -n monitoring
kubectl logs <prometheus-pod> -n monitoring
kubectl get pvc -n monitoring
kubectl get events -n monitoring
```

---

# 69. Grafana HA Troubleshooting

Check:

```text
Pod health
Service
Load balancer
Database
Authentication
Data source connectivity
```

Commands:

```bash
kubectl get pods -n monitoring
kubectl logs <grafana-pod> -n monitoring
kubectl describe pod <grafana-pod> -n monitoring
kubectl get svc -n monitoring
```

If Grafana is running but dashboards show no data:

```text
Grafana
   |
   v
Data Source
   |
   v
Prometheus
```

Check the connection between them.

---

# 70. Elasticsearch HA Troubleshooting

Check:

```text
Cluster health
Node availability
Disk pressure
Heap usage
Shard allocation
Indexing failures
Network connectivity
```

Important conceptual flow:

```text
Logs not visible
      ↓
Kibana?
      ↓
Elasticsearch?
      ↓
Logstash?
      ↓
Log source?
```

Do not assume Elasticsearch is the problem just because Kibana shows no logs.

---

# 71. Logstash HA Troubleshooting

Check:

```text
Input
Filter
Output
Pipeline
Queue
Elasticsearch connectivity
Resource utilization
```

If logs stop:

```text
Application
    ↓
Are logs generated?
    ↓
Logstash
    ↓
Are events received?
    ↓
Filter
    ↓
Are events dropped?
    ↓
Elasticsearch
    ↓
Are events indexed?
```

This prevents random troubleshooting.

---

# 72. Common HA Mistakes

### Mistake 1

Running multiple replicas on the same node.

```text
Node-1
 |
 +-- P-1
 +-- P-2
```

Node failure removes both.

---

### Mistake 2

Running replicas in the same AZ.

```text
AZ-A
 |
 +-- P-1
 +-- P-2
```

AZ failure removes both.

---

### Mistake 3

No persistent storage.

A pod restart can cause data loss.

---

### Mistake 4

No backups.

HA does not protect against:

```text
Data corruption
Accidental deletion
Configuration mistakes
```

---

### Mistake 5

No monitoring of observability.

You can have:

```text
Broken Prometheus
```

without realizing it.

---

### Mistake 6

Ignoring cardinality.

High metric cardinality can destabilize Prometheus even when replicas exist.

---

### Mistake 7

Ignoring Elasticsearch disk pressure.

High disk utilization can eventually affect indexing and cluster health.

---

# 73. Production Best Practices

## Prometheus

```text
Use redundant replicas where required
Spread replicas across failure domains
Monitor TSDB
Monitor cardinality
Monitor disk
Plan retention
Consider remote storage for long-term retention
```

## Grafana

```text
Run multiple replicas for critical environments
Use shared persistent state
Place behind a load balancer
Back up dashboards/configuration
Monitor data source health
```

## Alertmanager

```text
Use HA where alerting is critical
Configure deduplication
Group related alerts
Design routing carefully
Test notification channels
```

## Elasticsearch

```text
Use multiple nodes
Use shard replicas
Spread nodes across AZs
Monitor disk
Monitor heap
Monitor cluster health
Use snapshots
Control shard count
```

## Logstash

```text
Use multiple instances
Use durable buffering where necessary
Monitor pipeline health
Monitor events in/out
Monitor Elasticsearch connectivity
```

## Kibana

```text
Use multiple replicas where required
Place behind a load balancer
Monitor Elasticsearch connectivity
```

---

# 74. Production HA Validation

Do not assume HA works because replicas exist.

Test it.

Examples:

```text
Delete Prometheus pod
Delete Grafana pod
Delete Logstash pod
Stop an Elasticsearch node
Drain a Kubernetes node
Simulate AZ failure where practical
Restart components
Test alert delivery
Test recovery
```

Observe:

```text
Did monitoring continue?
Were alerts delivered?
Was data lost?
How long was the interruption?
Did workloads recover?
```

---

# 75. Chaos Testing for Observability

Controlled failure testing can validate HA.

Example:

```text
Normal
  ↓
Kill Prometheus-1
  ↓
Prometheus-2 continues
  ↓
Metrics available
  ↓
Recovery
```

Another:

```text
Normal
  ↓
Terminate Elasticsearch node
  ↓
Cluster remains available
  ↓
Replica recovery
  ↓
Cluster healthy
```

The objective is:

> **Prove resilience rather than assuming resilience.**

---

# 76. Recovery Time Objective

RTO is:

> The maximum acceptable time to restore a service after failure.

For an observability platform:

```text
Prometheus RTO = X
Grafana RTO = X
Logging RTO = X
Alerting RTO = X
```

Critical alerting may require a shorter RTO than historical log search.

---

# 77. Recovery Point Objective

RPO is:

> The maximum acceptable amount of data loss measured in time.

For example:

```text
RPO = 5 minutes
```

means losing more than approximately five minutes of data may be unacceptable.

RPO matters especially for:

```text
Logs
Metrics
Dashboards
Configuration
Elasticsearch data
```

---

# 78. RTO and RPO Example

Suppose:

```text
Observability RTO = 15 minutes
Observability RPO = 5 minutes
```

Then the architecture should be designed so that:

```text
Failure
  ↓
Restore service ≤ 15 min
```

and:

```text
Data loss ≤ approximately 5 min
```

The actual targets should be established according to business requirements.

---

# 79. Security and HA

HA must not weaken security.

For example:

```text
Grafana replicas
```

should still have:

```text
Authentication
Authorization
TLS
Network controls
Secrets management
Least privilege
```

Similarly:

```text
Elasticsearch cluster
```

should not be exposed directly to the public internet simply because multiple nodes are required.

---

# 80. Production Architecture Summary

A mature observability HA architecture should provide:

```text
                     HIGH AVAILABILITY
                            |
        +-------------------+-------------------+
        |                   |                   |
        v                   v                   v
     Metrics             Logging            Alerting
        |                   |                   |
        v                   v                   v
 Prometheus HA         Log Pipeline HA     Alertmanager HA
        |                   |                   |
        v                   v                   v
 Long-Term Storage     Elasticsearch HA     Notifications
        |                   |
        v                   v
    Grafana HA          Kibana HA
```

Across infrastructure:

```text
Multiple Pods
     +
Multiple Nodes
     +
Multiple AZs
     +
Persistent Storage
     +
Backups
     +
Monitoring
     +
Testing
```

---

# 81. Interview Questions

## Q1. How would you make Prometheus highly available?

A strong answer:

> I would run multiple Prometheus replicas and distribute them across different Kubernetes nodes and availability zones using anti-affinity or topology spread constraints. Both replicas can scrape the required targets. For larger environments, I would separate local collection from long-term storage and querying using an appropriate external metrics architecture. I would also design alerting carefully so duplicate alerts are deduplicated.

---

## Q2. Is two Prometheus replicas enough for HA?

No.

I would also consider:

```text
Node placement
AZ placement
Storage
Query layer
Duplicate samples
Alerting
Retention
Recovery
Capacity
```

HA is an architectural property, not simply a replica count.

---

## Q3. How would you make Grafana highly available?

I would deploy multiple Grafana replicas behind a load balancer and use shared persistent state, typically through a shared database. I would also ensure dashboards, configuration, authentication, and data sources remain consistent across replicas.

---

## Q4. How would you make Elasticsearch highly available?

I would deploy a multi-node Elasticsearch cluster, distribute nodes across availability zones, configure appropriate shard replicas, monitor disk and JVM utilization, and maintain snapshots for disaster recovery.

---

## Q5. What happens if one Elasticsearch node fails?

If shard replicas are correctly configured and the cluster retains the necessary quorum and resources, Elasticsearch can continue serving requests while reallocating or recovering affected shards.

I would check cluster health, shard allocation, disk capacity, node availability, and recovery status.

---

## Q6. How do you prevent duplicate alerts with HA Prometheus?

When multiple Prometheus replicas evaluate the same alert, the alerting architecture must support deduplication and grouping. Alertmanager clustering can coordinate alert handling so that engineers do not receive multiple notifications for the same underlying incident.

---

## Q7. How would you design observability across multiple AWS Availability Zones?

I would distribute observability workloads across multiple AZs using Kubernetes topology constraints. For stateful systems such as Elasticsearch, I would distribute nodes and shard replicas across failure domains. For stateless systems such as Grafana, I would use multiple replicas behind a load balancer.

I would also ensure storage, networking, backups, and recovery mechanisms are designed to tolerate the expected failure scenarios.

---

# 82. Scenario-Based Interview Question

### Scenario

Your production EKS cluster has:

```text
Prometheus
Grafana
Elasticsearch
Kibana
```

All observability components are running as single pods.

The production application is highly available.

An EKS worker node suddenly fails.

What happens?

### Weak answer

> Kubernetes will restart the pods.

That is incomplete.

### Strong answer

If all observability components are scheduled on the failed node, the observability platform can become unavailable until Kubernetes reschedules those workloads.

I would redesign the architecture using:

```text
Multiple replicas
Pod anti-affinity
Topology spread constraints
Multiple AZs
Persistent storage
PodDisruptionBudgets
Proper resource requests
```

For Elasticsearch, I would additionally use:

```text
Multiple nodes
Shard replicas
AZ-aware placement
Snapshots
```

For Prometheus, I would consider:

```text
Multiple replicas
External/long-term storage where required
HA-aware querying
```

The key point is that the observability platform should have failure tolerance comparable to the production application.

---

# 83. Scenario — Prometheus Is Down During an Incident

### Interviewer:

> Your application is returning 500 errors, but Prometheus is also down. How do you troubleshoot?

### Strong answer:

I would first determine whether the issue is limited to Prometheus or whether the broader observability infrastructure is affected.

I would use:

```text
Kubernetes
Application logs
Elasticsearch/Kibana
Application health endpoints
Load balancer information
```

to investigate the application.

For Prometheus itself, I would check:

```text
Pod status
Events
Logs
PVC
Node health
Resource utilization
Configuration
```

If a redundant Prometheus instance exists, I would use it for current metrics while recovering the failed instance.

After resolving the incident, I would investigate why Prometheus became unavailable and whether the observability HA design needs improvement.

---

# 84. Scenario — Elasticsearch Is Red

### Interviewer:

> Kibana is showing Elasticsearch cluster health as red. What do you do?

### Strong answer:

I would treat red status as a serious issue because it indicates one or more primary shards are unavailable.

I would investigate:

```text
1. Cluster health
2. Node availability
3. Unassigned shards
4. Disk utilization
5. JVM/heap pressure
6. Recent node failures
7. Index allocation
8. Network connectivity
```

Then I would determine whether the cause is:

```text
Node failure
Disk pressure
Allocation issue
Corruption
Resource exhaustion
Configuration issue
```

I would avoid making destructive changes before understanding the shard state and recovery path.

---

# 85. Scenario — Logs Suddenly Stop Appearing in Kibana

A strong troubleshooting chain is:

```text
Kibana
  ↓
Elasticsearch
  ↓
Logstash
  ↓
Log source
```

Check:

```text
Is Kibana connected?
       ↓
Is Elasticsearch healthy?
       ↓
Is Logstash receiving events?
       ↓
Is Logstash output successful?
       ↓
Are applications generating logs?
```

This avoids assuming:

```text
"Kibana is broken."
```

when the real problem may be:

```text
Logstash
or
Elasticsearch
or
Application logging
```

---

# 86. Senior DevOps Mental Model

When designing HA observability, think in failure domains.

Ask:

```text
What if the pod fails?

What if the node fails?

What if the AZ fails?

What if the storage fails?

What if the network fails?

What if the database fails?

What if configuration is corrupted?

What if the entire observability stack is lost?

How do we detect that failure?

How do we recover?

How much data can we lose?

How quickly must we recover?
```

This produces a real production architecture instead of simply adding replicas.

---

# 87. Final Production Architecture

The complete mental model is:

```text
                         AWS REGION
                              |
             +----------------+----------------+
             |                                 |
            AZ-A                              AZ-B
             |                                 |
         EKS Nodes                          EKS Nodes
             |                                 |
       +-----+------+                    +-----+------+
       |            |                    |            |
       v            v                    v            v
   Prometheus-1  Grafana-1          Prometheus-2  Grafana-2
       |            |                    |            |
       +------------+--------------------+------------+
                              |
                       Metrics Platform
                              |
                              v
                       Long-Term Storage


Applications / Containers
          |
          v
     Log Pipeline
          |
     +----+----+
     |         |
     v         v
 Logstash-1 Logstash-2
     |         |
     +----+----+
          |
          v
 Elasticsearch Cluster
     |       |       |
     v       v       v
   ES-1    ES-2    ES-3
          |
          v
      Kibana HA


Prometheus
     |
     v
Alertmanager HA
     |
     v
Notifications
     |
     v
Incident Response
```

The architecture should be backed by:

```text
Git
 |
 v
Infrastructure as Code
 |
 +-- Terraform
 +-- Helm
 +-- Kubernetes manifests
 |
 v
Deployment / GitOps
```

And protected by:

```text
Backups
Monitoring
Security
Capacity Planning
Disaster Recovery
Failure Testing
```

---

# 88. Final Key Takeaways

Remember these principles for interviews and production work:

### 1. Observability is production infrastructure

It must itself be highly available.

### 2. Replicas alone do not guarantee HA

Consider:

```text
Nodes
AZs
Storage
Networking
Quorum
Data
Alerting
```

### 3. Separate HA from durability

A service can remain available while historical data is lost.

### 4. Avoid correlated failures

Do not place all replicas:

```text
on one node
or
in one AZ
```

### 5. Monitor the monitoring system

Use meta-monitoring.

### 6. Test failure scenarios

Do not assume HA works.

### 7. Use IaC

Observability should be reproducible.

### 8. Plan RTO and RPO

Know:

```text
How quickly can we recover?
How much data can we lose?
```

### 9. HA has a cost

Design according to business criticality.

### 10. Design around failure domains

Think:

```text
Pod
 ↓
Node
 ↓
AZ
 ↓
Region
 ↓
Data
 ↓
Recovery
```

The central production principle is:

> **A production application is only as operable as the observability system that allows engineers to understand and recover it.**
