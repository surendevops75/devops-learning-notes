# Prometheus High Availability

Prometheus is designed primarily as a monitoring and time-series database system, but a single Prometheus instance can become a critical dependency in a production monitoring platform.

If the only Prometheus instance fails:

```text
Prometheus
    ↓
      X
    Down
```

then:

```text
Metrics collection stops
Alerts stop being evaluated
Dashboards lose fresh data
Incident visibility is reduced
```

For production environments, especially critical Kubernetes and EKS platforms, Prometheus should be designed with appropriate high-availability and failure-recovery mechanisms.

---

# 1. Why Prometheus High Availability?

A single Prometheus instance creates a single point of failure.

Example:

```text
                     EKS
                      │
                      ↓
                 Prometheus
                      │
                 ┌────┴────┐
                 ↓         ↓
              Grafana   Alerting
```

If Prometheus fails:

```text
                     EKS
                      │
                      X
                 Prometheus
                      │
                 ┌────┴────┐
                 X         X
              Grafana   Alerting
```

Even if Kubernetes applications are healthy, your monitoring visibility is affected.

---

# 2. Basic HA Architecture

A common approach is to run two Prometheus replicas:

```text
                       Kubernetes
                            │
                 ┌──────────┴──────────┐
                 ↓                     ↓
           Prometheus A          Prometheus B
                 │                     │
                 │                     │
                 └──────────┬──────────┘
                            ↓
                       Alertmanager
                            │
                            ↓
                       Notifications
```

Both Prometheus instances scrape the same targets.

---

# 3. Prometheus HA Does Not Mean Shared Storage

A common misunderstanding is:

```text
Prometheus A
      │
      ↓
Shared filesystem
      ↑
      │
Prometheus B
```

Prometheus instances are generally designed to maintain their own local TSDB data.

A typical HA architecture is:

```text
Prometheus A ──→ Local TSDB A
Prometheus B ──→ Local TSDB B
```

Both independently collect the same metrics.

---

# 4. Independent Scraping

With two replicas:

```text
                  Kubernetes
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
    Prometheus A             Prometheus B
          │                       │
          ↓                       ↓
       Targets                 Targets
```

Both instances scrape:

```text
Node Exporter
Kube-State-Metrics
Applications
Exporters
ServiceMonitors
PodMonitors
```

This provides monitoring redundancy.

---

# 5. What Happens If Prometheus A Fails?

Suppose:

```text
Prometheus A → DOWN
Prometheus B → UP
```

Prometheus B continues collecting metrics.

Grafana can query Prometheus B.

Alerting can continue through the remaining Prometheus path.

Therefore:

```text
One Prometheus failure
        ↓
Monitoring continues
        ↓
Reduced risk of monitoring outage
```

---

# 6. What Happens to Data?

Suppose:

```text
Prometheus A
collected metrics for 10 days
```

and then fails.

Prometheus B does not automatically contain Prometheus A's local TSDB data simply because both were scraping the same targets.

Therefore HA provides:

```text
Availability
```

but does not automatically provide:

```text
Complete shared historical storage
```

This distinction is important.

---

# 7. HA vs Long-Term Storage

These solve different problems.

### High Availability

Protects against:

```text
Prometheus instance failure
```

### Long-Term Storage

Protects against:

```text
Limited local retention
Long-term historical analysis
Global querying
Large-scale metrics storage
```

A production architecture may need both.

---

# 8. HA + Long-Term Storage

Conceptually:

```text
                  Kubernetes
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       Prometheus A        Prometheus B
             │                   │
             └─────────┬─────────┘
                       ↓
                Long-Term Store
                       │
                       ↓
                    Grafana
```

Examples of long-term metric systems include:

```text
Thanos
Grafana Mimir
Cortex
VictoriaMetrics
```

The selected technology depends on organizational requirements.

---

# 9. Prometheus Replica Labels

When two Prometheus replicas scrape the same target, the resulting data can look similar.

To distinguish the replicas, use external labels.

Example:

```yaml
global:
  external_labels:
    cluster: production
    replica: prometheus-01
```

The second replica can use:

```yaml
global:
  external_labels:
    cluster: production
    replica: prometheus-02
```

The exact configuration depends on the deployment model.

---

# 10. Why Replica Identity Matters

Consider:

```text
Prometheus A
replica=prometheus-01

Prometheus B
replica=prometheus-02
```

Now a long-term metrics system can distinguish:

```text
same metric
+
same cluster
+
different Prometheus replica
```

This is useful for deduplication.

---

# 11. Duplicate Metrics

Because both Prometheus replicas scrape the same target:

```text
Application
    │
    ├──→ Prometheus A
    │
    └──→ Prometheus B
```

the same logical metric exists twice.

For example:

```text
http_requests_total
```

may be present from both replicas.

This is not automatically a problem.

The monitoring architecture must account for replica duplication when querying or storing data centrally.

---

# 12. HA Querying

Suppose Grafana queries both Prometheus instances independently:

```text
Grafana
   ├──→ Prometheus A
   └──→ Prometheus B
```

Without proper deduplication, dashboards may show duplicate series.

A common production approach is to place a query layer in front of multiple Prometheus replicas.

Conceptually:

```text
                  Grafana
                     │
                     ↓
                 Query Layer
                     │
            ┌────────┴────────┐
            ↓                 ↓
      Prometheus A      Prometheus B
```

---

# 13. Thanos Architecture

One common HA and long-term storage architecture is Thanos.

Conceptually:

```text
                 Grafana
                    │
                    ↓
               Thanos Query
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
    Prometheus A        Prometheus B
          │                   │
          ↓                   ↓
      Object Storage      Object Storage
```

Thanos can provide:

```text
Global querying
Deduplication
Long-term storage
HA-aware querying
```

---

# 14. Object Storage

Long-term metrics can be stored in object storage.

In AWS environments, this may be:

```text
Amazon S3
```

Architecture:

```text
Prometheus
    ↓
Thanos Sidecar
    ↓
Object Storage
    ↓
S3
```

Object storage provides durable long-term storage independent of the local Prometheus pod.

---

# 15. Thanos Sidecar

A Thanos sidecar runs alongside Prometheus.

Conceptually:

```text
Prometheus Pod
 ├── Prometheus
 └── Thanos Sidecar
```

The sidecar can:

```text
Expose Prometheus data
Upload blocks to object storage
Provide access to Prometheus data
```

---

# 16. Thanos Query

Thanos Query provides a unified query interface.

Architecture:

```text
                    Grafana
                       │
                       ↓
                  Thanos Query
                       │
             ┌─────────┼─────────┐
             ↓         ↓         ↓
        Prometheus A  Prom B   Store
                               Gateway
```

Grafana can query the Thanos endpoint instead of directly querying each Prometheus instance.

---

# 17. Thanos Store Gateway

The Store Gateway provides access to historical blocks stored in object storage.

Architecture:

```text
Grafana
   ↓
Thanos Query
   ↓
Store Gateway
   ↓
Object Storage
```

This allows historical metrics to be queried after they are no longer present in local Prometheus storage.

---

# 18. Thanos Compactor

Thanos Compactor manages long-term object-storage blocks.

Conceptually:

```text
Object Storage
      ↓
Thanos Compactor
      ↓
Compaction
      ↓
Downsampling
      ↓
Efficient historical queries
```

The exact architecture should be designed according to the Thanos deployment model.

---

# 19. Prometheus HA Without Thanos

Not every environment needs Thanos.

A smaller production cluster may use:

```text
Prometheus A
Prometheus B
Alertmanager HA
Persistent storage
Grafana
```

This can provide reasonable availability without introducing a full long-term metrics platform.

---

# 20. When Do You Need Thanos?

Consider Thanos or another long-term backend when you need:

```text
Multiple clusters
Long-term retention
Global querying
Large metric volumes
Cross-cluster dashboards
HA deduplication
Object-storage durability
```

For a small environment, this may be unnecessary complexity.

---

# 21. Multi-Cluster Monitoring

Suppose an organization has:

```text
Production EKS
Staging EKS
Development EKS
```

Each cluster can have Prometheus:

```text
Prod → Prometheus
Stage → Prometheus
Dev → Prometheus
```

A centralized architecture can provide one query layer:

```text
                   Grafana
                      │
                      ↓
                 Global Query
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
      Prod          Stage          Dev
   Prometheus     Prometheus    Prometheus
```

---

# 22. Multi-Cluster Labels

Each cluster should have a unique identity.

For example:

```text
cluster=production
cluster=staging
cluster=development
```

Then queries can distinguish environments.

Example:

```promql
up{cluster="production"}
```

---

# 23. AWS EKS Multi-Cluster Architecture

A larger environment could look like:

```text
                      Grafana
                         │
                         ↓
                   Global Query
                         │
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
    EKS Prod          EKS Stage         EKS Dev
        │                │                │
   Prometheus A/B   Prometheus A/B   Prometheus A/B
        │                │                │
        └────────────────┼────────────────┘
                         ↓
                  Object Storage
```

This provides a centralized observability view.

---

# 24. Alerting in HA Prometheus

Suppose:

```text
Prometheus A
    ↓
Alert fires

Prometheus B
    ↓
Same alert fires
```

You do not want:

```text
Two pages
```

for one incident.

This is why Alertmanager's grouping and deduplication behavior is important.

---

# 25. HA Alertmanager

Run multiple Alertmanager replicas:

```text
              Prometheus A
                   │
              Prometheus B
                   │
            ┌──────┴──────┐
            ↓             ↓
      Alertmanager A  Alertmanager B
            │             │
            └──────┬──────┘
                   ↓
              Notification
```

Alertmanager replicas form a cluster for high availability.

---

# 26. Alertmanager vs Prometheus HA

Prometheus HA:

```text
Multiple metric collectors
```

Alertmanager HA:

```text
Multiple alert-management instances
```

Both layers need to be considered independently.

---

# 27. Grafana HA

Grafana can also become a single point of failure.

A larger production environment may use:

```text
               Load Balancer
                    │
             ┌──────┴──────┐
             ↓             ↓
          Grafana A     Grafana B
```

Grafana HA also requires appropriate handling of shared configuration and state.

---

# 28. Complete HA Monitoring Stack

A larger production architecture:

```text
                         Users
                           │
                           ↓
                     Load Balancer
                           │
                    ┌──────┴──────┐
                    ↓             ↓
                 Grafana A     Grafana B
                    │             │
                    └──────┬──────┘
                           ↓
                      Query Layer
                           │
              ┌────────────┼────────────┐
              ↓            ↓            ↓
        Prometheus A  Prometheus B  Long-Term Store
              │            │
              └─────┬──────┘
                    ↓
              Alertmanager HA
                    │
                    ↓
               Notifications
```

---

# 29. Kubernetes Deployment

Prometheus HA can be configured using the Prometheus Operator.

A simplified conceptual configuration:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus

metadata:
  name: production

spec:
  replicas: 2

  serviceMonitorSelector:
    matchLabels:
      monitoring: enabled
```

The actual configuration should be aligned with the installed Prometheus Operator and chart version.

---

# 30. Prometheus Replicas

With:

```yaml
spec:
  replicas: 2
```

the Operator creates two Prometheus replicas.

Conceptually:

```text
Prometheus CR
     │
     ↓
Prometheus Operator
     │
     ├──→ Prometheus A
     │
     └──→ Prometheus B
```

---

# 31. Pod Anti-Affinity

Running both replicas on the same node creates a failure risk.

Bad:

```text
Node 1
 ├── Prometheus A
 └── Prometheus B
```

If Node 1 fails:

```text
Prometheus A → DOWN
Prometheus B → DOWN
```

Better:

```text
Node 1
 └── Prometheus A

Node 2
 └── Prometheus B
```

---

# 32. Topology Spread

In larger clusters, spread Prometheus replicas across:

```text
Nodes
Availability Zones
```

For EKS:

```text
AZ-A
 └── Prometheus A

AZ-B
 └── Prometheus B
```

This protects against node and availability-zone failures.

---

# 33. Anti-Affinity Example

Conceptually:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - prometheus
        topologyKey: kubernetes.io/hostname
```

Production configurations should use labels and selectors matching the actual Prometheus workload.

---

# 34. Availability Zone Distribution

For AWS EKS:

```text
              EKS Cluster
                   │
       ┌───────────┼───────────┐
       ↓           ↓           ↓
      AZ-A        AZ-B        AZ-C
       │           │           │
      Node        Node        Node
       │           │           │
     Prom A      Prom B
```

This provides stronger resilience than putting replicas on the same node.

---

# 35. Persistent Storage for Prometheus

Each Prometheus replica should have its own persistent storage when local retention is required.

Conceptually:

```text
Prometheus A
     ↓
PVC A
     ↓
Storage A

Prometheus B
     ↓
PVC B
     ↓
Storage B
```

Do not assume both replicas should mount the same writable filesystem.

---

# 36. Storage Class in EKS

For AWS EKS, persistent storage may use an appropriate EBS-backed StorageClass.

For example:

```yaml
storageSpec:
  volumeClaimTemplate:
    spec:
      storageClassName: gp3
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 100Gi
```

The StorageClass name must exist in the cluster.

---

# 37. Prometheus Resource Requests

Prometheus itself needs resources.

Example:

```yaml
resources:
  requests:
    cpu: 500m
    memory: 2Gi

  limits:
    cpu: 2
    memory: 4Gi
```

These are only examples.

Real values should be based on:

```text
Target count
Scrape interval
Metric cardinality
Query load
Retention
Rule count
Cluster size
```

---

# 38. Why Prometheus Needs More Memory

Prometheus memory usage can increase with:

```text
High cardinality
Large number of active series
Frequent scrapes
Large rule sets
Heavy queries
Large ingestion volume
```

Therefore:

```text
More targets
+
More labels
+
More series
=
More resource requirements
```

---

# 39. High Cardinality and HA

Running multiple Prometheus replicas doubles much of the scraping and storage workload.

For example:

```text
10 million series
       ↓
Two replicas
       ↓
Potentially significant additional ingestion/storage
```

HA should therefore be designed together with cardinality management.

---

# 40. High Cardinality Example

Bad metric label:

```text
http_requests_total{
  user_id="93847293847"
}
```

If millions of users exist:

```text
Millions of time series
```

This can overload Prometheus.

Prefer bounded labels such as:

```text
service
method
route
status_code
```

where appropriate.

---

# 41. HA Does Not Fix Bad Cardinality

Important principle:

```text
Bad cardinality
     +
HA
     =
More expensive bad cardinality
```

Before scaling Prometheus horizontally, optimize metric design.

---

# 42. Query Performance

Large Prometheus environments can experience expensive queries.

Examples:

```text
Huge range queries
High-cardinality aggregations
Complex regex
Large dashboard panels
```

Solutions may include:

```text
Recording rules
Query optimization
Downsampling
Long-term storage
Query federation
Dedicated query layers
```

---

# 43. Recording Rules in HA

Recording rules should be evaluated consistently across replicas.

Example:

```yaml
groups:
  - name: application-recording-rules

    rules:

      - record: service:http_requests:rate5m

        expr: |
          sum by(service) (
            rate(http_requests_total[5m])
          )
```

Both Prometheus replicas may calculate the same recording rule.

A centralized query layer can deduplicate replicas where supported.

---

# 44. Federation vs Long-Term Storage

Prometheus federation can be useful for selected metrics.

Example:

```text
Cluster Prometheus
       ↓
Federation
       ↓
Central Prometheus
```

But federation is not equivalent to a full long-term storage and global-query architecture.

Use the approach that matches the scale and requirements.

---

# 45. HA and Disaster Recovery

High availability protects against:

```text
Single instance failure
```

Disaster recovery addresses larger failures such as:

```text
Cluster loss
Region failure
Storage failure
Configuration loss
```

A mature monitoring architecture should consider both.

---

# 46. Monitoring Configuration Backup

Store configuration in Git:

```text
Prometheus values
PrometheusRules
ServiceMonitors
PodMonitors
Alertmanager configuration
Grafana dashboards
```

Then:

```text
Cluster lost
   ↓
Create new cluster
   ↓
ArgoCD
   ↓
Restore monitoring stack
```

---

# 47. GitOps and Disaster Recovery

A strong architecture:

```text
Git
 │
 ├── Prometheus
 ├── Alertmanager
 ├── Grafana
 ├── Rules
 ├── Dashboards
 └── ServiceMonitors
          │
          ↓
        ArgoCD
          │
          ↓
         EKS
```

The cluster becomes reproducible.

---

# 48. Monitoring Configuration as Code

Treat monitoring like infrastructure.

Instead of manually changing:

```text
Prometheus UI
Alertmanager UI
Grafana UI
```

prefer:

```text
Git
 ↓
Pull Request
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
```

This provides:

```text
Version control
Auditability
Rollback
Consistency
```

---

# 49. HA Monitoring Failure Scenarios

## Scenario 1: Prometheus A fails

```text
Prom A → DOWN
Prom B → UP
```

Monitoring continues through B.

---

## Scenario 2: Node hosting Prometheus A fails

```text
Node A → DOWN
Prom A → DOWN
Prom B → UP
```

Kubernetes reschedules Prometheus A if configured appropriately.

---

## Scenario 3: Availability Zone fails

If Prometheus replicas are spread across AZs:

```text
AZ-A → DOWN
Prom A → DOWN
Prom B → UP
```

Monitoring can continue.

---

# 50. Failure Scenario: Both Replicas on Same Node

Bad architecture:

```text
Node 1
 ├── Prometheus A
 └── Prometheus B
```

Node failure:

```text
Node 1
   ↓
Both replicas lost
```

This is not meaningful HA.

---

# 51. Failure Scenario: Both Replicas in Same AZ

If an AZ outage occurs:

```text
AZ-A
 ├── Prometheus A
 └── Prometheus B
```

both can be lost.

Therefore distribute replicas across failure domains.

---

# 52. Failure Scenario: Shared Storage Dependency

If both Prometheus replicas depend on one storage system:

```text
Prom A ─┐
        ├── Shared dependency
Prom B ─┘
```

that dependency may become a common failure point.

HA design should identify and reduce common dependencies.

---

# 53. Common HA Failure Points

Look for:

```text
Same node
Same AZ
Same storage
Same network path
Same credentials
Same load balancer
Same DNS dependency
```

True HA requires considering the entire dependency chain.

---

# 54. Alertmanager HA Failure

If Alertmanager A fails:

```text
Alertmanager A → DOWN
Alertmanager B → UP
```

the cluster should continue handling notifications according to its HA configuration.

---

# 55. Notification Provider Failure

Even if Alertmanager is healthy:

```text
Alertmanager
     ↓
Slack
      X
```

notifications may fail because the external notification service is unavailable.

Therefore monitor notification delivery itself.

---

# 56. Alertmanager Notification Failure

Useful operational signals include:

```text
Notification failures
Notification latency
Receiver errors
```

If notifications fail silently, the monitoring system may appear healthy while the on-call channel is broken.

---

# 57. Grafana Failure

If Grafana A fails:

```text
Grafana A → DOWN
Grafana B → UP
```

the query backend can continue serving metrics.

This is another example of separating:

```text
Data collection
Querying
Visualization
Alerting
```

---

# 58. Separation of Responsibilities

A mature monitoring platform separates:

```text
Prometheus
→ Collect + Store + Evaluate

Alertmanager
→ Manage + Route Alerts

Grafana
→ Visualize + Investigate

Long-Term Store
→ Historical Metrics

Object Storage
→ Durable Metric Blocks
```

This separation improves scalability and reliability.

---

# 59. Production HA Architecture for EKS

A realistic architecture:

```text
                              AWS
                               │
                              EKS
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
        AZ-A                 AZ-B                 AZ-C
          │                    │                    │
       Prom A               Prom B             Monitoring
          │                    │                Components
          │                    │
          └──────────┬─────────┘
                     ↓
                Alertmanager
                  HA Cluster
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        Email      Slack      Pager
                     │
                     ↓
                 Engineers

Prometheus replicas
       │
       ↓
Long-Term Metrics Backend
       │
       ↓
Object Storage
       │
       ↓
Grafana / Query Layer
```

---

# 60. Prometheus HA With Kubernetes Operator

In your monitoring stack, the Operator manages Prometheus.

Conceptually:

```text
Prometheus CR
     ↓
Prometheus Operator
     ↓
Prometheus StatefulSet
     ↓
Prometheus Pods
```

The Operator manages lifecycle operations such as configuration reconciliation.

---

# 61. Why StatefulSet?

Prometheus is commonly deployed using a StatefulSet because each replica benefits from stable identity and persistent storage.

Conceptually:

```text
StatefulSet
   │
   ├── prometheus-0
   │      ↓
   │     PVC
   │
   └── prometheus-1
          ↓
         PVC
```

This differs from a completely stateless Deployment model.

---

# 62. Prometheus Pod Identity

StatefulSet provides stable names:

```text
prometheus-0
prometheus-1
```

This is useful for:

```text
Persistent storage
Stable identity
Operational management
```

---

# 63. Pod Disruption Budget

For production monitoring, consider a PodDisruptionBudget.

Example:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget

metadata:
  name: prometheus-pdb

spec:
  minAvailable: 1

  selector:
    matchLabels:
      app: prometheus
```

The selector must match the actual Prometheus pod labels.

---

# 64. Why PDB?

A PodDisruptionBudget helps protect availability during voluntary disruptions such as:

```text
Node drain
Cluster maintenance
Node upgrades
```

It does not protect against every possible failure.

---

# 65. Kubernetes Upgrade Scenario

Suppose a node upgrade drains:

```text
Node 1
```

If Prometheus A is on Node 1:

```text
Prometheus A → evicted
```

Prometheus B remains:

```text
Prometheus B → running
```

This is why replicas plus scheduling constraints matter.

---

# 66. Production Resource Planning

Before enabling HA, estimate:

```text
Number of targets
Scrape interval
Number of active series
Samples per second
Retention
Query load
Rule evaluation load
Dashboard load
```

Then size:

```text
CPU
Memory
Storage
Network
```

---

# 67. Samples Per Second

Prometheus ingestion depends heavily on scrape frequency.

Conceptually:

```text
Targets
×
Metrics per target
÷
Scrape interval
```

Higher scrape frequency increases ingestion volume.

For example:

```text
1000 targets
1000 metrics each
30-second interval
```

creates a significantly different workload than:

```text
1000 targets
1000 metrics each
15-second interval
```

---

# 68. HA Cost

If one Prometheus consumes:

```text
2 CPU
8 GiB memory
```

two replicas roughly require:

```text
4 CPU
16 GiB memory
```

plus overhead.

Actual resource usage must be measured rather than assumed to scale perfectly linearly.

---

# 69. HA Monitoring Checklist

```text
[ ] Multiple Prometheus replicas
[ ] Replicas spread across nodes
[ ] Replicas spread across AZs where appropriate
[ ] Persistent storage
[ ] Resource requests
[ ] Resource limits
[ ] PodDisruptionBudget
[ ] Anti-affinity / topology spread
[ ] Alertmanager HA
[ ] Monitoring self-alerts
[ ] Long-term storage where required
[ ] Backup/recovery strategy
[ ] GitOps configuration
[ ] Cardinality management
[ ] Query optimization
```

---

# 70. HA Troubleshooting

If Prometheus A is down:

```bash
kubectl get pods -n monitoring
```

Check:

```bash
kubectl describe pod <prometheus-pod> -n monitoring
```

Then:

```bash
kubectl logs <prometheus-pod> -n monitoring
```

Investigate:

```text
Scheduling
Resources
PVC
Configuration
Readiness
Liveness
Storage
```

---

# 71. Prometheus PVC Troubleshooting

Check:

```bash
kubectl get pvc -n monitoring
```

If:

```text
STATUS = Pending
```

investigate:

```text
StorageClass
Provisioner
Availability Zone
Capacity
Access mode
Cloud provider storage
```

---

# 72. Prometheus Memory Troubleshooting

Symptoms:

```text
OOMKilled
Restarts
Slow queries
High memory usage
```

Investigate:

```text
Active series
Cardinality
Query complexity
Retention
Scrape volume
Recording rules
Resource limits
```

Do not simply increase memory without understanding the cause.

---

# 73. Prometheus CPU Troubleshooting

High CPU may come from:

```text
Heavy PromQL
High ingestion
Large rule groups
Frequent evaluations
High cardinality
Many dashboards
```

Optimize queries and rules before simply scaling resources.

---

# 74. Prometheus Storage Troubleshooting

If storage grows quickly:

```text
Disk usage ↑
```

investigate:

```text
Metric volume
Retention
Cardinality
Scrape interval
High-volume exporters
```

Then consider:

```text
Retention reduction
Metric filtering
Cardinality optimization
Long-term storage
```

---

# 75. Cardinality Management

Avoid unbounded labels such as:

```text
user_id
request_id
transaction_id
session_id
```

when they create huge numbers of unique time series.

Prefer bounded dimensions:

```text
service
method
route
status
environment
```

where appropriate.

---

# 76. Metric Relabeling

Metric relabeling can remove unwanted metrics or labels before storage.

Conceptually:

```yaml
metricRelabelings:
  - action: drop
    regex: "unnecessary_metric.*"
```

Use carefully.

Dropping metrics can make troubleshooting impossible if important signals are removed.

---

# 77. Target Relabeling

Target relabeling modifies target labels before scraping.

It can be used for:

```text
Cluster identification
Environment labels
Target filtering
Standardized labels
```

---

# 78. HA and Security

Protect:

```text
Prometheus
Alertmanager
Grafana
Long-term storage
Object storage
```

Use:

```text
Authentication
Authorization
TLS
NetworkPolicies
IAM
Secrets
Private networking
```

---

# 79. AWS IAM for Object Storage

If using S3 for long-term metrics, avoid static access keys where possible.

For EKS workloads, prefer an appropriate AWS identity mechanism such as:

```text
IAM Roles for Service Accounts
```

or the current EKS-supported pod identity approach.

The exact implementation depends on the AWS architecture and platform version.

---

# 80. Network Security

Monitoring components should not be unnecessarily exposed.

Prefer:

```text
Private Prometheus
Private Alertmanager
Controlled Grafana access
Restricted service-to-service communication
```

Use Kubernetes NetworkPolicies where supported.

---

# 81. Monitoring Data Security

Metrics can sometimes contain sensitive information through:

```text
Labels
URLs
Application metadata
Tenant IDs
```

Avoid putting sensitive information into metric labels.

Never expose:

```text
Passwords
Tokens
Secrets
Personal data
```

through metrics.

---

# 82. HA and Maintenance

Before upgrading Prometheus:

```text
Check HA replica status
Check storage
Check rules
Check Alertmanager
Check dashboards
```

Then upgrade in a controlled manner.

The objective is:

```text
Upgrade one replica
while another remains available
```

where the deployment architecture supports that behavior.

---

# 83. Rolling Upgrade

Conceptually:

```text
Prometheus A + Prometheus B
          │
          ↓
Upgrade A
          │
          ↓
Prometheus B remains available
          │
          ↓
A healthy
          │
          ↓
Upgrade B
```

Always validate the exact upgrade behavior for the Operator and chart version being used.

---

# 84. HA and Disaster Recovery Test

A mature team should periodically test:

```text
Prometheus pod failure
Node failure
AZ failure
Alertmanager failure
Storage failure
Grafana failure
Cluster recovery
```

Do not assume HA works simply because replicas exist.

---

# 85. Chaos Testing

Controlled failure testing can validate:

```text
Rescheduling
Alerting
Data availability
Notification delivery
Grafana connectivity
Recovery time
```

Perform such tests only under an approved process.

---

# 86. Recovery Objectives

Define:

```text
RTO
RPO
```

for the monitoring platform.

### RTO

How quickly should monitoring be restored?

### RPO

How much monitoring history can be lost?

These requirements influence whether you need:

```text
HA
Long-term storage
Object storage
Backups
Multi-region architecture
```

---

# 87. Monitoring Platform RTO/RPO Example

Suppose:

```text
RTO = 15 minutes
RPO = 5 minutes
```

Your architecture must be capable of:

```text
Restoring monitoring within 15 minutes
```

while limiting metric-history loss to an acceptable window.

The exact design depends on the chosen storage and recovery architecture.

---

# 88. Multi-Region Prometheus

For extremely critical environments, monitoring may span regions.

Conceptually:

```text
Region A
 └── EKS
      └── Prometheus

Region B
 └── EKS
      └── Prometheus

          ↓
   Global Metrics Layer
          ↓
       Grafana
```

This is significantly more complex and should only be implemented when business requirements justify it.

---

# 89. Multi-Region Alerting

Critical alerts should not depend on a monitoring component that exists only in the failed region.

A highly resilient architecture may use:

```text
Region A Prometheus
       ↓
Alertmanager

Region B Prometheus
       ↓
Alertmanager

       ↓
Reliable notification platform
```

The exact architecture depends on the organization's availability requirements.

---

# 90. Prometheus HA vs Global Observability

These are different scales.

### Single cluster HA

```text
Prometheus A
Prometheus B
```

### Multi-cluster

```text
Prod
Stage
Dev
```

### Global observability

```text
Region A
Region B
Region C
Multiple clusters
Central query layer
Long-term storage
```

Choose the simplest architecture that satisfies the requirements.

---

# 91. When Not to Use Complex HA

Do not introduce:

```text
Thanos
Mimir
Cortex
Multi-region
Complex federation
```

just because they are available.

For a small environment:

```text
Prometheus
+
Grafana
+
Alertmanager
+
Persistent storage
```

may be sufficient.

Architecture should follow actual scale and reliability requirements.

---

# 92. Production Design Decision

A practical decision model:

```text
Small environment
    ↓
Single Prometheus + persistent storage

Important production environment
    ↓
Prometheus replicas + Alertmanager HA

Multiple clusters / long retention
    ↓
HA + long-term metrics backend

Large enterprise
    ↓
Centralized query + long-term object storage
+ multi-cluster / multi-region strategy where required
```

---

# 93. Recommended EKS Architecture for a Production Microservices Platform

For a production EKS microservices environment, a strong baseline is:

```text
                    EKS
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
     Prometheus A          Prometheus B
          │                     │
       AZ-A                   AZ-B
          │                     │
          └──────────┬──────────┘
                     ↓
               Alertmanager HA
                     │
          ┌──────────┼──────────┐
          ↓          ↓          ↓
        Slack      Email      Pager
                     │
                     ↓
                 Engineers

Prometheus
    ↓
Persistent Storage

Prometheus
    ↓
Optional Long-Term Backend
    ↓
Object Storage

Grafana
    ↓
Query Layer
    ↓
Metrics
```

---

# 94. Production Implementation Sequence

When implementing Prometheus HA:

```text
1. Install Prometheus Operator
        ↓
2. Configure Prometheus replicas
        ↓
3. Configure persistent storage
        ↓
4. Spread replicas across nodes/AZs
        ↓
5. Configure ServiceMonitors
        ↓
6. Configure PodMonitors
        ↓
7. Configure recording rules
        ↓
8. Configure alert rules
        ↓
9. Configure Alertmanager HA
        ↓
10. Configure Grafana
        ↓
11. Add long-term storage if required
        ↓
12. Test failure scenarios
```

---

# 95. Real-World Failure Scenario

Imagine your production EKS cluster has:

```text
3 AZs
20 nodes
100 microservices
2 Prometheus replicas
2 Alertmanager replicas
Grafana
```

Prometheus A is running in AZ-A.

Prometheus B is running in AZ-B.

AZ-A experiences an outage.

Result:

```text
Prometheus A → unavailable
Prometheus B → continues scraping
```

The monitoring platform remains operational.

This is the purpose of distributing replicas across failure domains.

---

# 96. Real-World Node Failure Scenario

Prometheus A runs on:

```text
Node-01
```

Node-01 fails.

Kubernetes detects the node failure.

Prometheus B continues running.

If the scheduler can place Prometheus A on another suitable node:

```text
Node-03
   ↓
Prometheus A
```

the HA pair is restored.

---

# 97. Real-World Prometheus Storage Failure

Suppose Prometheus A's PVC becomes unavailable.

Prometheus A may fail to operate correctly.

Prometheus B continues collecting metrics.

This illustrates why:

```text
Multiple replicas
+
independent storage
```

can improve availability.

---

# 98. Real-World Alerting Scenario

Suppose both replicas detect:

```text
Payment error rate = 15%
```

Both may generate the same alert.

Alertmanager receives both alert instances.

Its grouping/deduplication behavior prevents engineers from receiving unnecessary duplicate pages when the alerts are configured consistently.

---

# 99. Real-World Monitoring Scenario

A deployment introduces a memory leak.

Timeline:

```text
Deployment
    ↓
Memory gradually increases
    ↓
Pod restarts
    ↓
Error rate increases
    ↓
Alert fires
    ↓
Alertmanager
    ↓
On-call engineer
    ↓
Grafana investigation
    ↓
Deployment identified
    ↓
Rollback
    ↓
Memory returns to normal
```

This demonstrates why monitoring, alerting, dashboards, logs and deployment systems must work together.

---

# 100. Interview Answer: How Would You Implement Prometheus HA?

A strong production answer:

```text
"I would run at least two Prometheus replicas using Prometheus Operator.

I would ensure the replicas are distributed across different worker nodes and preferably different availability zones so that a node or AZ failure does not take down both replicas.

Each replica would have its own persistent storage.

I would configure Alertmanager in HA as well because duplicate alerts from multiple Prometheus replicas need to be grouped and deduplicated.

For larger environments with multiple clusters or long-term retention requirements, I would add a long-term metrics backend such as Thanos or Mimir, using object storage where appropriate.

Finally, I would manage the configuration through GitOps and regularly test node, pod, storage and monitoring-component failure scenarios."
```

---

# 101. Interview Answer: Does Prometheus HA Mean No Data Loss?

No.

A strong answer:

```text
"Not necessarily.

Prometheus HA primarily improves availability by running multiple independent replicas.

Both replicas scrape the same targets, but they maintain their own local storage.

If one replica fails, the other can continue collecting metrics, but historical data that existed only on the failed replica is not automatically available from the other replica.

For stronger long-term durability and centralized querying, I would use a system such as Thanos or another long-term metrics backend."
```

---

# 102. Interview Answer: Why Run Two Prometheus Instances?

```text
"I run multiple Prometheus replicas to avoid having a single monitoring collector become a single point of failure.

If one Prometheus instance fails, another instance can continue scraping the cluster and evaluating alerts.

For this to provide real resilience, I also spread replicas across nodes and availability zones and ensure Alertmanager is highly available."
```

---

# 103. Interview Answer: How Do You Avoid Duplicate Alerts?

```text
"With multiple Prometheus replicas, both replicas may evaluate the same alert.

I configure Alertmanager appropriately so that duplicate alert instances are grouped and deduplicated.

For larger HA architectures using long-term metrics systems such as Thanos, replica labels allow the query layer to identify and deduplicate identical data from multiple Prometheus replicas."
```

---

# 104. Interview Answer: What Is the Difference Between HA and Long-Term Storage?

```text
"HA protects availability.

For example, if Prometheus A fails, Prometheus B continues collecting metrics.

Long-term storage addresses durability and historical retention.

For example, if I need months or years of metrics across multiple clusters, I would use a long-term backend such as Thanos or Mimir.

So HA and long-term storage solve different problems, although they are often used together in enterprise environments."
```

---

# 105. Interview Answer: How Do You Make Prometheus Highly Available on EKS?

```text id="qj8x3m"
"I use Prometheus Operator and configure multiple Prometheus replicas.

I use persistent volumes for local retention and configure pod anti-affinity or topology spread so replicas are distributed across worker nodes and availability zones.

I also run Alertmanager in HA and ensure alert routing and deduplication are configured.

For larger environments, I add long-term storage backed by durable object storage and use a query layer for centralized access.

All monitoring configuration is managed through GitOps so the platform can be recreated consistently."
```

---

# 106. Final HA Mental Model

Remember:

```text
                    HIGH AVAILABILITY
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
   Prometheus HA      Alertmanager HA      Grafana HA
        │                  │                  │
        ↓                  ↓                  ↓
  Metric Collection     Alert Routing      Visualization
        │
        ↓
 Long-Term Storage
        │
        ↓
 Object Storage
```

And:

```text
HA
=
multiple instances
+
failure-domain distribution
+
independent resources
+
failure detection
+
recovery

Long-term storage
=
durable history
+
extended retention
+
centralized querying
```

---

# 107. Final Production Checklist

Before calling a Prometheus deployment production-ready:

```text
[ ] Multiple Prometheus replicas
[ ] Replicas spread across nodes
[ ] Replicas spread across AZs
[ ] Persistent volumes configured
[ ] Correct StorageClass
[ ] Resource requests configured
[ ] Resource limits configured
[ ] PodDisruptionBudget configured
[ ] Anti-affinity/topology spread configured
[ ] ServiceMonitors tested
[ ] PodMonitors tested
[ ] PrometheusRules tested
[ ] Alertmanager HA configured
[ ] Alert deduplication verified
[ ] Alert routing tested
[ ] Grafana availability considered
[ ] Prometheus self-monitoring configured
[ ] Alertmanager self-monitoring configured
[ ] Cardinality reviewed
[ ] Recording rules optimized
[ ] Long-term storage evaluated
[ ] Disaster recovery documented
[ ] GitOps configuration stored in Git
[ ] Failure scenarios tested
```

---

# 108. Final Architecture

The complete production model is:

```text
                              USERS
                                │
                                ↓
                            Grafana
                                │
                                ↓
                         Global Query Layer
                                │
                    ┌───────────┴───────────┐
                    ↓                       ↓
              Prometheus A            Prometheus B
                    │                       │
                  AZ-A                    AZ-B
                    │                       │
                    └───────────┬───────────┘
                                │
                         Metrics Collection
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
        Node Exporter      Applications     Kube-State-Metrics
              │                 │                 │
              └─────────────────┼─────────────────┘
                                ↓
                           Alert Rules
                                │
                                ↓
                         Alertmanager HA
                                │
                  ┌─────────────┼─────────────┐
                  ↓             ↓             ↓
                Slack         Email          Pager
                                │
                                ↓
                           On-Call Team

Prometheus
    │
    ↓
Long-Term Metrics Backend
    │
    ↓
Object Storage
```

The most important production principle is:

```text
Do not design Prometheus HA as
"just run two Prometheus pods."

Real HA requires:

Multiple replicas
+
Failure-domain distribution
+
Persistent storage
+
Alertmanager HA
+
Duplicate handling
+
Resource planning
+
Cardinality management
+
Long-term storage when required
+
Disaster recovery
+
Failure testing
```

That is what turns a basic Prometheus deployment into a **production-grade monitoring platform**.
