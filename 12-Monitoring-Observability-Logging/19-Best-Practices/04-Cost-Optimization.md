# Cost Optimization

> Production Monitoring & Observability Cost Optimization — Prometheus, Grafana, ELK, Kubernetes/EKS, Metrics, Logs, Storage, Network, Retention, Query Optimization, Capacity Planning, FinOps, Troubleshooting and DevOps/DevSecOps Interview Preparation

---

# 1. Purpose

Observability is essential for production reliability, but uncontrolled telemetry can become expensive.

Typical cost drivers include:

    Metric ingestion
    High-cardinality time series
    Log ingestion
    Log storage
    Elasticsearch nodes
    Persistent volumes
    Network transfer
    Cross-AZ traffic
    Query infrastructure
    High-availability replicas
    Long retention
    Unused dashboards and alerts

The goal is not:

> Collect the minimum possible telemetry.

The goal is:

> Collect the right telemetry at the right resolution, retain it for the right period, and operate the platform efficiently.

---

# 2. Cost Optimization Principles

1. Understand the workload before cutting resources.
2. Eliminate unnecessary telemetry first.
3. Control metric cardinality.
4. Control log volume.
5. Use appropriate retention.
6. Separate hot and historical data.
7. Optimize queries before scaling infrastructure.
8. Avoid unnecessary replicas.
9. Reduce cross-AZ and cross-region traffic where practical.
10. Right-size CPU, memory and storage.
11. Monitor unused dashboards, alerts and data sources.
12. Preserve telemetry required for incidents, security and compliance.
13. Measure cost per service/team where possible.
14. Treat observability as a shared production platform.
15. Review cost continuously as workloads change.

---

# 3. Cost Architecture

Typical production flow:

    Applications
        |
        +---- Metrics ----> Prometheus
        |                     |
        |                     v
        |                  Grafana
        |
        +---- Logs -------> Collector
                              |
                              v
                           Logstash
                              |
                              v
                        Elasticsearch
                              |
                              v
                           Kibana

Costs exist at every stage:

    Application telemetry
       |
       v
    Collection
       |
       v
    Processing
       |
       v
    Network
       |
       v
    Storage
       |
       v
    Query
       |
       v
    Visualization
       |
       v
    Retention / backup

---

# 4. Cost Categories

## Infrastructure Cost

    EC2
    EKS nodes
    Persistent volumes
    Load balancers
    Elasticsearch nodes
    Prometheus instances

## Storage Cost

    Metrics
    Logs
    Snapshots
    Backups
    Persistent volumes

## Network Cost

    Cross-AZ
    Cross-region
    Internet egress
    Centralized telemetry traffic

## Operational Cost

    High-maintenance clusters
    Excessive dashboards
    Unnecessary alerts
    Complex pipelines
    Manual administration

---

# 5. Cost Per Telemetry Type

## Metrics

Cost drivers:

    Number of series
    Samples/sec
    Retention
    Replication
    Remote write

## Logs

Cost drivers:

    Events/sec
    Average event size
    Retention
    Replication
    Indexing
    Query volume

## Traces

Cost drivers:

    Spans/sec
    Span size
    Sampling rate
    Retention

The optimization principle is the same:

    Remove unnecessary data
       |
       v
    Keep valuable data
       |
       v
    Store it efficiently

---

# 6. Observability Cost Lifecycle

Think of telemetry as:

    Generate
       |
       v
    Collect
       |
       v
    Transport
       |
       v
    Process
       |
       v
    Store
       |
       v
    Query
       |
       v
    Retain
       |
       v
    Delete/archive

Cost can be reduced at any stage.

The cheapest telemetry is telemetry that does not need to be generated.

---

# 7. Metric Cardinality and Cost

High cardinality increases:

    Memory
    CPU
    Disk
    Network
    Query cost
    Remote storage cost

Bad labels:

    user_id
    request_id
    session_id
    transaction_id

Better:

    service
    method
    route
    status_code
    environment

Cardinality control is both a performance and cost strategy.

---

# 8. Cardinality Example

Suppose:

    20 services
    5 HTTP methods
    10 status codes
    4 environments

Potential combinations:

    20 x 5 x 10 x 4
    = 4,000

Now add:

    100,000 users

The theoretical combination becomes enormous.

Do not put unbounded business identifiers into metric labels.

---

# 9. Metric Cost Review

For every metric ask:

    Is it used?
    Is it alerted?
    Is it dashboarded?
    Is it required for troubleshooting?
    Is it required for capacity planning?

If the answer is no:

    Consider dropping it.

But verify dependencies before deletion.

---

# 10. Scrape Interval Optimization

A shorter scrape interval produces more samples.

Example:

    1,000 targets

At 15 seconds:

    ~66.7 scrape cycles/sec

At 60 seconds:

    ~16.7 scrape cycles/sec

The exact sample volume depends on metrics per target, but the principle is:

> Higher frequency = higher ingestion and storage.

Use the lowest frequency that still meets the operational requirement.

---

# 11. Different Scrape Intervals

Do not treat every target equally.

Example:

    Critical application:
    15s

    Standard application:
    30s

    Slow infrastructure metric:
    60s

Exact values should be determined by:

    Alert requirements
    SLOs
    Detection time
    Cost

---

# 12. Metric Filtering

Exporters can expose large numbers of metrics.

Use:

    Target filtering
    Relabeling
    Metric relabeling
    Collector configuration

Goal:

    Keep useful metrics
    Drop irrelevant metrics

Important:

> Dropping a metric after scraping saves storage, but may not eliminate the cost of collecting it.

---

# 13. Application Instrumentation Cost

Developers can unintentionally create expensive telemetry.

Examples:

    One metric per user
    One metric per request ID
    Excessive histogram buckets
    Large log payloads
    DEBUG logging in production

Establish instrumentation standards before production.

---

# 14. Histogram Cost

Histograms can generate multiple time series per metric.

Example:

    request_duration_seconds_bucket

with many bucket boundaries can increase series count.

Use buckets that answer real questions.

Avoid excessive bucket counts without a clear requirement.

---

# 15. Summary vs Histogram

Choose the metric type based on the use case.

Consider:

    Aggregation
    Quantiles
    SLO calculations
    Cardinality
    Storage

Do not automatically use the most detailed metric type everywhere.

---

# 16. Recording Rules and Cost

Recording rules can reduce repeated query computation.

Benefits:

    Faster dashboards
    Faster alerts
    Lower repeated query CPU

But recording rules also create stored series.

Therefore:

    Use them for valuable repeated calculations
    Remove unused rules

Optimization is not simply:

> Create more recording rules.

---

# 17. PromQL Cost

Expensive queries can consume substantial compute.

Avoid:

    Broad selectors
    Huge time ranges
    Complex joins
    Unnecessary subqueries
    Expensive regex

Prefer:

    Narrow labels
    Appropriate aggregation
    Short operational ranges
    Recording rules

---

# 18. Dashboard Cost

Every dashboard can generate:

    Queries
    Data-source CPU
    Network traffic
    Browser workload

A dashboard that refreshes every 5 seconds creates much more load than one refreshing every minute.

Review:

    Panel count
    Refresh interval
    Query complexity
    User count

---

# 19. Unused Dashboards

Organizations often accumulate dashboards.

Examples:

    Old application dashboard
    Old migration dashboard
    Duplicate service dashboard
    Temporary incident dashboard

Review dashboards periodically.

Remove or archive obsolete ones.

---

# 20. Query Storm Cost

During an incident:

    Engineers
       |
       v
    Dashboards
       |
       v
    Query volume ↑
       |
       v
    Infrastructure CPU ↑

Design incident dashboards to be:

    Focused
    Lightweight
    Reusable

Cost optimization should not make incident investigation painful.

---

# 21. Alert Cost

Every alert rule consumes:

    Query CPU
    Memory
    Evaluation resources

Unused alerts are pure operational overhead.

Review:

    Alert firing frequency
    Query complexity
    Business value
    Ownership

Delete alerts that no longer represent actionable conditions.

---

# 22. Alert Noise

Noisy alerts create:

    Engineer fatigue
    Notification cost
    Query workload
    Incident confusion

A good alert should be:

    Actionable
    Relevant
    Owned
    Properly prioritized

Alert quality is a cost optimization.

---

# 23. Log Volume

Log volume is one of the largest observability cost drivers.

Reduce unnecessary volume through:

    Correct log levels
    Structured logs
    Sampling
    Deduplication
    Rate limiting
    Removing repetitive messages

Do not remove important error and security events.

---

# 24. Log Level Strategy

Typical production policy:

    ERROR:
    Always retained

    WARN:
    Retained according to policy

    INFO:
    Normal operational logs

    DEBUG:
    Usually disabled or selectively enabled

Avoid:

    DEBUG globally in production

unless temporarily required and controlled.

---

# 25. Structured Logging

Structured logs improve:

    Search
    Filtering
    Parsing
    Automation

They can also reduce processing overhead compared with repeatedly parsing arbitrary text.

Example:

    {
      "service": "orders",
      "level": "ERROR",
      "status": 500,
      "environment": "prod"
    }

---

# 26. Log Event Size

Large logs cost more to:

    Generate
    Serialize
    Transport
    Parse
    Store
    Query

Avoid unnecessarily logging:

    Full request bodies
    Large response payloads
    Duplicate stack traces
    Sensitive data

---

# 27. Duplicate Logging

A single error may be logged at:

    Application
    Middleware
    API gateway
    Sidecar
    Collector

This can multiply volume.

Review the logging architecture and eliminate unnecessary duplication.

---

# 28. Error Loop

A retry loop can generate:

    100 events/sec
    1,000 events/sec
    10,000 events/sec

The result can be:

    Application load ↑
    Log volume ↑
    Logstash load ↑
    Elasticsearch load ↑
    Storage cost ↑

Fix the application behavior first.

---

# 29. Log Sampling

Sampling can reduce cost for repetitive high-volume events.

Example:

    Successful requests:
    Low sample rate

    Errors:
    High retention

    Slow requests:
    High retention

Sampling must be designed carefully so that critical evidence is preserved.

---

# 30. Log Retention

Not every log needs the same retention.

Example:

    Hot:
    7 days

    Warm:
    30 days

    Archive:
    90+ days

Exact periods depend on:

    Operations
    Security
    Compliance
    Business requirements

---

# 31. Hot/Warm/Cold Strategy

Recent data:

    Fast storage

Older data:

    Lower-cost storage

Rarely accessed data:

    Archive

This reduces expensive storage usage while preserving historical information.

---

# 32. Elasticsearch ILM

Use Index Lifecycle Management for:

    Rollover
    Tier movement
    Retention
    Deletion

Without lifecycle management:

    Logs accumulate
       |
       v
    Disk grows
       |
       v
    Cluster grows
       |
       v
    Cost grows

---

# 33. Elasticsearch Shard Cost

Every shard has overhead.

Too many shards cause:

    Memory usage
    CPU overhead
    Metadata overhead
    Management complexity

Avoid creating unnecessary daily indexes with tiny volumes.

---

# 34. Small Shards

Example:

    1 GB/day

Creating many tiny shards can be inefficient.

Prefer a shard strategy based on:

    Data volume
    Query patterns
    Node count
    Retention

---

# 35. Elasticsearch Replicas

Replica shards improve availability and read scalability.

But replicas also consume:

    Storage
    Network
    CPU
    Recovery bandwidth

Use replica counts appropriate to:

    Availability requirements
    Query load
    Failure model

---

# 36. Elasticsearch Storage Tiers

Use appropriate storage for:

    Hot data
    Warm data
    Cold data

Do not keep all historical logs on expensive high-performance storage.

---

# 37. Elasticsearch Query Cost

Expensive queries increase:

    CPU
    Memory
    Search latency

Optimize:

    Time range
    Filters
    Aggregations
    Field types

Start with narrow queries.

---

# 38. Logstash Cost

Logstash consumes:

    CPU
    Memory
    Network
    Disk

Expensive filters such as complex Grok patterns increase processing cost.

Prefer structured input when possible.

---

# 39. Collector Cost

Collectors running on every Kubernetes node create aggregate resource consumption.

If:

    100 nodes

and each collector uses:

    200 MB memory

the fleet uses roughly:

    20 GB memory

before considering CPU and other overhead.

Measure aggregate collector cost.

---

# 40. DaemonSet vs Centralized Collection

## DaemonSet

Advantages:

    Local collection
    Simple routing
    Lower application-to-collector distance

Cost:

    Collector runs on every node

## Centralized

Advantages:

    Fewer collector instances

Risks:

    Central bottleneck
    More network traffic
    Failure concentration

Choose based on scale and reliability.

---

# 41. Prometheus High Availability Cost

Two Prometheus replicas provide better availability but approximately duplicate scrape workload unless architecture provides efficient deduplication.

Before enabling HA ask:

    Is it required?
    What is the failure impact?
    How is duplicate data handled?
    What is the storage cost?

---

# 42. Grafana High Availability Cost

Multiple Grafana replicas consume:

    CPU
    Memory
    Database connections

Use HA when availability requirements justify it.

Do not run replicas simply because:

> Production means three replicas.

Architecture must follow requirements.

---

# 43. Elasticsearch High Availability Cost

Multi-node clusters provide:

    Failure tolerance
    Shard distribution
    Query parallelism

But cost increases with:

    Nodes
    Replicas
    Storage
    Network
    Operational overhead

Design based on:

    Data volume
    Availability
    RTO/RPO

---

# 44. Prometheus Storage Cost

Local Prometheus storage costs depend on:

    Samples/sec
    Series
    Retention
    Replicas
    Storage performance

Avoid unnecessarily long local retention if historical data can be stored more economically elsewhere.

---

# 45. Remote Storage Cost

Remote metrics storage may introduce:

    Ingestion charges
    Storage charges
    Query charges
    Network charges

Evaluate total cost, not just local disk savings.

---

# 46. Backup Cost

Observability data may require backups depending on business needs.

Consider:

    Snapshot frequency
    Retention
    Cross-region copies
    Storage tier
    Restore requirements

Not every temporary metric needs the same backup policy as critical configuration or compliance logs.

---

# 47. Disaster Recovery Cost

DR can involve:

    Secondary clusters
    Cross-region storage
    Replication
    Backups
    Standby infrastructure

Choose:

    Active-active
    Active-passive
    Backup/restore

based on:

    RTO
    RPO
    Cost

---

# 48. AWS/EKS Cost Drivers

For an EKS observability platform consider:

    EKS worker nodes
    EBS volumes
    Load balancers
    Data transfer
    NAT gateway traffic
    Elasticsearch infrastructure
    S3/archive storage
    Cross-AZ traffic

Telemetry can generate substantial network volume.

---

# 49. Cross-AZ Cost

Example:

    Application in AZ-A
        |
        v
    Collector in AZ-B
        |
        v
    Elasticsearch in AZ-C

High-volume telemetry can create unnecessary cross-AZ traffic.

Where practical:

    Prefer local collection
    Plan placement
    Keep high-volume flows efficient

Do not sacrifice HA simply to eliminate every cross-AZ transfer.

---

# 50. NAT Gateway Cost

If private EKS workloads send large telemetry volumes through NAT gateways:

    Application
       |
       v
    NAT Gateway
       |
       v
    External service

data processing and transfer costs can grow.

Review architecture for high-volume telemetry paths.

---

# 51. S3 Archive Strategy

For long-term logs or exports, object storage can provide lower-cost retention than keeping everything in Elasticsearch.

Typical architecture:

    Active logs
       |
       v
    Elasticsearch
       |
       v
    Archive
       |
       v
    S3/object storage

Use lifecycle policies for older objects.

---

# 52. Storage Lifecycle

Example:

    0–7 days:
    Hot searchable

    8–30 days:
    Warm

    31–90 days:
    Archive

    >90 days:
    Delete if policy allows

The exact values must follow business and compliance requirements.

---

# 53. Cost Allocation

For shared observability, identify cost by:

    Team
    Service
    Environment
    Cluster
    Data source

Useful dimensions:

    Metrics generated
    Log GB/day
    Query usage
    Storage consumed

This helps teams understand their telemetry footprint.

---

# 54. Showback and Chargeback

## Showback

Tell teams:

    "Your service generated 500 GB of logs."

No direct billing.

## Chargeback

Allocate actual platform cost to teams.

Use when organizational maturity supports it.

The objective is accountability, not punishment.

---

# 55. Cost per Service

A useful model:

    Service
       |
       +---- Metric volume
       +---- Log volume
       +---- Storage
       +---- Query usage
       +---- Infrastructure share

Then identify:

    Highest-cost services
    Fastest-growing services
    Unusual telemetry producers

---

# 56. Cost Anomaly Detection

Watch for sudden increases:

    Log GB/day ↑ 10x
    Metric series ↑ 5x
    Elasticsearch disk ↑ rapidly
    Cross-AZ traffic ↑

Possible causes:

    Deployment
    Logging configuration
    Cardinality explosion
    Retry storm
    Incident

Cost anomalies can reveal technical problems.

---

# 57. Cost and Performance Tradeoff

Do not optimize cost by:

    Removing critical metrics
    Removing security logs
    Removing important audit events
    Making dashboards unusably slow
    Reducing availability below requirements

Optimize waste first.

---

# 58. Cost Optimization Priority

Use this order:

    1. Remove unused telemetry
    2. Control cardinality
    3. Reduce unnecessary log volume
    4. Optimize retention
    5. Optimize storage tiers
    6. Optimize queries
    7. Optimize network paths
    8. Right-size infrastructure
    9. Review HA/replication
    10. Negotiate/optimize platform capacity

---

# 59. Right-Sizing

Monitor actual usage:

    CPU
    Memory
    Storage
    Network

Then adjust:

    Node types
    Pod requests
    Pod limits
    Elasticsearch nodes
    Prometheus resources

Do not right-size from a single quiet day.

Use:

    Normal
    Peak
    Incident

usage.

---

# 60. Kubernetes Requests and Cost

Requests influence scheduling and capacity planning.

If requests are too high:

    Nodes may be unnecessarily added.

If requests are too low:

    Workloads may compete for resources.

Use historical utilization to tune requests.

---

# 61. Cluster Autoscaling

Observability workloads affect cluster scaling.

Example:

    Prometheus requests ↑
       |
       v
    Scheduler needs more capacity
       |
       v
    Cluster adds nodes
       |
       v
    Cost ↑

Right-sizing observability workloads can reduce indirect infrastructure cost.

---

# 62. Dedicated Node Pool Cost

Dedicated monitoring nodes improve isolation but may have lower utilization.

Use them when the benefit is worth:

    Additional nodes
    Additional operational complexity

For smaller environments, shared nodes may be more cost-effective.

---

# 63. Spot Instances

Some observability workloads may be suitable for lower-cost interruptible capacity, but critical stateful components require careful analysis.

Consider:

    Collector agents
    Non-critical processing workers

Be cautious with:

    Primary storage
    Critical Elasticsearch nodes
    Sole Prometheus instance

Availability requirements come first.

---

# 64. Storage Optimization

Review:

    Volume size
    Volume type
    IOPS
    Throughput
    Retention
    Replica count

Do not pay for maximum IOPS if the workload does not require it.

But do not choose slow storage that causes production alert/log delays.

---

# 65. Compression

Compression can reduce:

    Storage
    Network

But requires:

    CPU

Choose based on:

    CPU availability
    Storage cost
    Network cost
    Query performance

---

# 66. Query Caching

Caching can reduce repeated query cost.

Useful for:

    Capacity dashboards
    Historical trends
    Frequently viewed panels

Less useful when:

    Real-time data is required

Understand freshness requirements.

---

# 67. Dashboard Optimization for Cost

For every panel ask:

    Is this used?
    Is it actionable?
    Is the query expensive?
    Does it need real-time refresh?
    Can a recording rule help?

Remove unused panels.

---

# 68. Alert Optimization for Cost

For every alert ask:

    Is it actionable?
    Who owns it?
    Does it fire too often?
    Is the query expensive?
    Is the condition already covered elsewhere?

A noisy alert is both an operational and compute cost.

---

# 69. Retention Policy Design

Retention should be based on:

    Incident investigation
    Compliance
    Security
    SLO reporting
    Capacity planning

Do not choose:

    "Keep everything forever."

unless there is a specific requirement.

---

# 70. Different Data, Different Retention

Example:

    Critical security logs:
    Longer

    Application INFO:
    Shorter

    Debug logs:
    Very short

    Operational metrics:
    Medium

    Long-term capacity metrics:
    Aggregated

This reduces storage without losing useful information.

---

# 71. Downsampling

For long-term metrics, storing lower-resolution data can reduce cost.

Example:

    Recent:
    High resolution

    Historical:
    Aggregated resolution

This preserves trends without storing every raw sample forever.

---

# 72. Aggregation

Instead of retaining every detailed series forever:

    Pod-level data
       |
       v
    Service-level aggregation
       |
       v
    Historical trend

This can dramatically reduce long-term storage.

Be careful not to remove detail required for incident analysis.

---

# 73. Metrics vs Logs for High-Cardinality Context

If you need:

    user_id
    request_id
    transaction_id

consider putting that context in:

    Logs
    Traces

rather than Prometheus labels.

This is often both cheaper and more appropriate.

---

# 74. Cost-Aware Instrumentation

During development review:

    Metric count
    Label count
    Histogram buckets
    Log event size
    Log frequency

Make cost visible before production.

---

# 75. Production Example — Metric Cost Explosion

A developer adds:

    user_id

to:

    request_duration_seconds

Result:

    Active series ↑ dramatically
    Prometheus memory ↑
    Disk ↑
    Query latency ↑
    Remote storage cost ↑

Correct solution:

    Remove user_id from metric labels
    Keep it in logs/traces

---

# 76. Production Example — Log Cost Explosion

Application changes from:

    100 MB/hour

to:

    10 GB/hour

Investigation finds:

    DEBUG enabled globally

Fix:

    Restore production log level
    Keep targeted debugging only
    Add controls around temporary DEBUG

---

# 77. Production Example — Elasticsearch Cost Growth

Symptoms:

    Disk grows faster every month
    Elasticsearch nodes increase
    Cost increases

Investigation:

    No lifecycle policy
    Old logs remain hot
    Excessive replicas

Fix:

    ILM
    Hot/warm strategy
    Appropriate replica count
    Retention enforcement

---

# 78. Production Example — Cross-AZ Cost

Architecture:

    EKS nodes
       |
       v
    Collector in another AZ
       |
       v
    Elasticsearch in third AZ

Large log volume creates:

    High network traffic
    Additional transfer cost
    Extra latency

Optimization:

    Review placement
    Use local collection
    Balance with HA requirements

---

# 79. Production Example — Dashboard Query Cost

A dashboard runs:

    30-day
    high-resolution
    complex PromQL

every:

    5 seconds

Result:

    Query CPU ↑
    Prometheus latency ↑
    Cost ↑

Fix:

    Narrow default range
    Increase refresh interval
    Recording rule
    Aggregation

---

# 80. Production Example — Unused Data Sources

Over time:

    Old Prometheus
    Old Elasticsearch
    Test cluster

remain connected to Grafana.

Consequences:

    Infrastructure cost
    Operational complexity
    Security surface

Remove obsolete resources after dependency verification.

---

# 81. Cost Optimization Workflow

When cost increases:

    1. Identify which component increased
    2. Identify the resource causing cost
    3. Compare with historical baseline
    4. Identify recent changes
    5. Find highest-volume producers
    6. Remove unnecessary workload
    7. Optimize retention/query/storage
    8. Right-size infrastructure
    9. Validate reliability
    10. Add preventive controls

---

# 82. Cost Investigation Questions

Ask:

    Which service generates the most logs?
    Which service creates the most time series?
    Which cluster consumes the most storage?
    Which dashboards generate the most queries?
    Which indexes are growing fastest?
    Which environment is least efficient?
    Is cost growth aligned with business growth?
    Did a recent deployment change telemetry?

---

# 83. Cost Dashboard

Create:

    Observability Cost & Usage

dashboard.

Include:

    Log GB/day
    Metric samples/sec
    Active series
    Elasticsearch disk
    Storage growth
    Query volume
    Collector resource usage
    Network traffic
    Cost trend

---

# 84. Cost SLOs / Guardrails

Examples:

    Maximum expected log volume
    Maximum metric cardinality
    Maximum storage growth rate
    Maximum telemetry CPU overhead

These are guardrails rather than universal limits.

Tune them to platform capacity.

---

# 85. Cost Alerts

Useful alerts:

    Log volume anomaly
    Cardinality spike
    Disk growth anomaly
    Storage threshold
    Queue growth
    Network traffic anomaly

A cost alert can also be a reliability alert.

---

# 86. Cost and Security

Never reduce:

    Required audit logs
    Security events
    Authentication logs
    Compliance-required retention

Cost optimization must respect:

    Security policy
    Compliance
    Legal retention

---

# 87. Cost and Incident Response

During incidents, do not aggressively shut down observability just to reduce cost.

The telemetry may be necessary to:

    Find root cause
    Validate recovery
    Establish impact
    Produce post-incident evidence

Cost optimization must preserve incident capability.

---

# 88. Cost and Disaster Recovery

DR architecture costs money.

Choose based on:

    RTO
    RPO
    Criticality

Example:

    Non-critical dev:
    Backup/restore

    Critical production:
    Multi-AZ

    Extremely critical:
    Cross-region strategy

Do not apply the same DR cost to every environment.

---

# 89. Environment-Based Optimization

## Development

    Short retention
    Lower resolution
    Smaller clusters
    Limited replicas

## Staging

    Production-like but controlled
    Moderate retention

## Production

    Reliability-focused
    Appropriate HA
    Required retention

## Temporary environments

    Automatic cleanup
    Short retention
    Minimal telemetry

---

# 90. Non-Production Cost

Temporary environments can become a major source of waste.

Examples:

    Test EKS cluster
    Temporary Elasticsearch
    Old Prometheus
    Unused dashboards
    Development log retention

Use:

    Automatic cleanup
    Short retention
    Scheduled shutdown where appropriate

---

# 91. Environment Lifecycle

For temporary environments:

    Create
       |
       v
    Use
       |
       v
    Test
       |
       v
    Destroy

Do not let observability infrastructure outlive the environment that needed it.

---

# 92. Resource Scheduling

Non-production observability can use lower-cost capacity where acceptable.

Examples:

    Smaller nodes
    Lower replica counts
    Interruptible capacity
    Reduced retention

Production reliability requirements should remain separate.

---

# 93. Cost Ownership

Define owners for:

    Prometheus
    Grafana
    Elasticsearch
    Logstash
    Collectors
    Dashboards
    Alerts
    Retention

Without ownership:

    Unused resources accumulate.

---

# 94. Lifecycle Governance

Every observability resource should have:

    Owner
    Purpose
    Environment
    Retention
    Review date

Examples:

    Dashboard
    Alert
    Index
    Data source
    Collector
    Storage

---

# 95. Monthly Cost Review

Review:

    Total cost
    Cost by environment
    Cost by team
    Log growth
    Metric growth
    Storage growth
    Network growth
    Unused resources

Compare:

    Current month
    Previous month
    Same period previously

---

# 96. Cost Trend Analysis

If cost grows:

    10% month over month

but application traffic grows:

    5%

investigate the difference.

Possible causes:

    Telemetry inefficiency
    Cardinality
    Log growth
    Retention
    Query workload

Cost should generally be understood relative to business workload.

---

# 97. Unit Economics

Useful metrics:

    Cost per GB of logs
    Cost per million metric samples
    Cost per service
    Cost per cluster
    Cost per application

This helps identify inefficient producers.

---

# 98. Cost Efficiency Ratio

Example concept:

    Observability cost
    ------------------
    Application traffic

Track the trend rather than relying on one absolute number.

If traffic doubles but observability cost triples, investigate.

---

# 99. Cost Optimization and Reliability

The target is:

    Lower waste

not:

    Lowest possible cost

A cheaper monitoring system that misses alerts is not optimized.

A better objective is:

    Reliability
       +
    Performance
       +
    Security
       +
    Cost efficiency

---

# 100. Cost Optimization Decision Tree

When something is expensive:

    Is the telemetry required?
          |
       NO | YES
          | 
          v
      Remove it
             |
             v
      Can volume be reduced?
             |
          YES|NO
             |
             v
      Filter/sample/aggregate
             |
             v
      Can retention be reduced?
             |
          YES|NO
             |
             v
      Apply lifecycle policy
             |
             v
      Can storage be optimized?
             |
             v
      Right-size infrastructure
             |
             v
      Validate reliability

---

# 101. Prometheus Cost Checklist

    [ ] Cardinality reviewed
    [ ] Unbounded labels removed
    [ ] Scrape intervals justified
    [ ] Unused metrics dropped
    [ ] Recording rules reviewed
    [ ] Expensive queries optimized
    [ ] Retention reviewed
    [ ] Remote write reviewed
    [ ] Storage right-sized
    [ ] HA justified

---

# 102. Grafana Cost Checklist

    [ ] Unused dashboards removed
    [ ] Panel count controlled
    [ ] Refresh rates reviewed
    [ ] Expensive queries optimized
    [ ] Variables optimized
    [ ] Data sources cleaned up
    [ ] HA justified
    [ ] Plugins reviewed

---

# 103. ELK Cost Checklist

    [ ] Log volume monitored
    [ ] DEBUG controlled
    [ ] Duplicate logs removed
    [ ] Log event size reviewed
    [ ] Grok optimized
    [ ] ILM configured
    [ ] Hot/warm/cold strategy
    [ ] Shard count reviewed
    [ ] Replica count justified
    [ ] Elasticsearch storage right-sized

---

# 104. Kubernetes/EKS Cost Checklist

    [ ] Observability requests tuned
    [ ] Limits reviewed
    [ ] Node pools right-sized
    [ ] Collector footprint measured
    [ ] Cross-AZ traffic reviewed
    [ ] NAT traffic reviewed
    [ ] Temporary clusters cleaned
    [ ] Non-production retention reduced
    [ ] HA matched to environment

---

# 105. Senior DevOps Interview — How Do You Reduce Observability Cost?

Strong answer:

> I start by identifying the major cost drivers across metrics, logs, storage, compute and network. I reduce unnecessary telemetry first, control Prometheus cardinality, optimize scrape intervals and queries, control log volume, implement appropriate Elasticsearch lifecycle policies, review storage tiers and right-size infrastructure. I also examine cross-AZ traffic and non-production resources. I validate that cost reductions do not compromise alerting, incident response, security or compliance.

---

# 106. Interview — How Do You Reduce Prometheus Cost?

Strong answer:

> I focus on active series and sample ingestion. I remove unbounded labels, drop unused metrics, filter unnecessary targets, use appropriate scrape intervals, optimize expensive PromQL and use recording rules for repeated calculations. I also review retention, remote write and HA requirements.

---

# 107. Interview — How Do You Reduce Elasticsearch Cost?

Strong answer:

> I control log volume and event size, optimize parsing, use ILM, move older data to lower-cost storage, review shard and replica counts, use appropriate mappings and right-size nodes. I also investigate cross-AZ traffic and archive older logs where appropriate.

---

# 108. Interview — How Do You Reduce Logging Cost Without Losing Important Data?

Strong answer:

> I keep critical errors, security and audit events while reducing unnecessary INFO or DEBUG volume, removing duplicate logs, controlling event size and using sampling for repetitive non-critical events. I apply different retention policies to different classes of logs rather than deleting everything uniformly.

---

# 109. Interview — What Is More Important: Performance or Cost?

Strong answer:

> Neither should be optimized independently. The objective is cost-efficient reliability. I first remove waste, then optimize workload and storage, and finally scale resources when required. I would never reduce observability to a point where it compromises production detection, troubleshooting, security or compliance.

---

# 110. Interview — How Do You Control High Cardinality?

Strong answer:

> I identify labels with rapidly growing value sets and remove identifiers such as user IDs and request IDs from metric labels. I use bounded dimensions such as service, route and status, and move detailed request-level context into logs or traces. I also review cardinality during instrumentation changes.

---

# 111. Interview — How Do You Optimize Log Retention?

Strong answer:

> I classify logs based on operational, security and compliance requirements. Recent logs remain highly searchable, older data can move to lower-cost storage, and data beyond the required retention period is deleted. I implement lifecycle policies rather than relying on manual cleanup.

---

# 112. Interview — How Does AWS Networking Affect Observability Cost?

Strong answer:

> High-volume telemetry crossing Availability Zones, regions or NAT gateways can create significant transfer and processing cost. I review collector placement, Elasticsearch placement and routing paths, while balancing cost against high availability and failure-domain requirements.

---

# 113. Interview — How Do You Handle Non-Production Observability Cost?

Strong answer:

> I use shorter retention, lower-resolution metrics, smaller infrastructure, fewer replicas and automated cleanup for temporary environments. I also ensure test clusters and their monitoring infrastructure are destroyed together rather than allowing orphaned resources.

---

# 114. Interview — How Do You Detect a Cost Anomaly?

Strong answer:

> I monitor log volume, metric samples, active series, storage growth, query volume and network traffic. If one suddenly increases, I correlate it with deployments and configuration changes. A cost anomaly often indicates a technical issue such as a logging loop, cardinality explosion or unexpected workload.

---

# 115. Interview — What Would You Do If Management Asked You to Cut Observability Cost by 30%?

Strong answer:

> I would first establish the current cost breakdown and identify the highest-value savings. I would remove unused telemetry and infrastructure, control cardinality, reduce unnecessary log volume, optimize retention and storage tiers, review non-production environments and right-size resources. I would protect critical metrics, alerts, security logs and compliance data. I would then measure the savings and validate that observability SLOs remain healthy.

---

# 116. Practical Cost Optimization Scenario

Environment:

    EKS
    Multiple microservices
    Prometheus
    Grafana
    ELK

Observed:

    Elasticsearch cost rising
    Prometheus memory rising
    Storage growing quickly

Investigation:

    Application DEBUG logging enabled
    user_id used as metric label
    90-day hot log retention
    Many duplicate dashboards
    Excessive Elasticsearch replicas

Optimization:

    Disable global DEBUG
    Remove user_id metric label
    Apply ILM
    Move older logs to cheaper storage
    Remove duplicate dashboards
    Reassess replicas

Result:

    Lower ingestion
    Lower cardinality
    Lower storage
    Lower compute
    Lower query load

---

# 117. Practical Cost Review Workflow

Every month:

    1. Measure total platform cost
    2. Break down by component
    3. Identify top consumers
    4. Compare growth to application traffic
    5. Review cardinality
    6. Review log volume
    7. Review storage
    8. Review dashboards/alerts
    9. Review HA/replicas
    10. Review network
    11. Apply optimizations
    12. Validate reliability

---

# 118. Production Cost Review Questions

Ask:

    What changed this month?
    Which service produces the most telemetry?
    Which metric family has the highest cardinality?
    Which index is growing fastest?
    Which dashboards are heavily queried?
    Which alerts are noisy?
    Which resources are underutilized?
    Which data can move to cheaper storage?
    Which temporary resources can be deleted?
    Are we paying for unnecessary HA?
    Are cross-AZ costs growing?
    Are cost increases justified by business growth?

---

# 119. Cost Optimization Guardrails

Never optimize by blindly:

    Deleting production data
    Removing security logs
    Disabling alerts
    Reducing replicas below availability requirements
    Moving critical workloads to unreliable storage
    Disabling monitoring during incidents

Always:

    Understand requirements
    Measure impact
    Make controlled changes
    Validate recovery and observability

---

# 120. Final Production Architecture

A cost-efficient observability architecture looks like:

    APPLICATIONS
         |
         +---- Metrics ----+
         |                 |
         +---- Logs -------+----> COLLECTION
         |                 |
         +---- Traces -----+
                           |
                           v
                       FILTERING
                           |
                    +------+------+
                    |             |
                    v             v
                 Metrics         Logs
                    |             |
                    v             v
                Prometheus     Logstash
                    |             |
                    v             v
                 Grafana     Elasticsearch
                                  |
                                  v
                               Kibana
                                  |
                                  v
                         Lifecycle / Archive

Across the platform:

    Reduce unnecessary data
           |
           v
    Control cardinality
           |
           v
    Optimize processing
           |
           v
    Use appropriate storage
           |
           v
    Apply retention
           |
           v
    Review cost continuously

---

# 121. Final Cost Mental Model

When cost increases:

    WHAT resource increased?
          |
          v
    WHICH component caused it?
          |
          v
    WHICH service/team produces the workload?
          |
          v
    IS THE TELEMETRY REQUIRED?
          |
          v
    CAN VOLUME BE REDUCED?
          |
          v
    CAN STORAGE/RETENTION BE OPTIMIZED?
          |
          v
    CAN QUERIES/PROCESSING BE OPTIMIZED?
          |
          v
    CAN INFRASTRUCTURE BE RIGHT-SIZED?
          |
          v
    DOES RELIABILITY REMAIN ACCEPTABLE?

The goal is not:

> Spend the least possible.

The goal is:

> Spend efficiently while preserving production reliability.

---

# 122. Final Principles to Remember

> The cheapest telemetry is telemetry that does not need to be generated.

> High cardinality creates both performance and cost problems.

> Logs should be retained according to their value and requirements.

> Hot storage should contain data that actually needs hot access.

> Old data can often move to cheaper storage.

> Every replica has a cost.

> Every query has a cost.

> Every dashboard refresh has a cost.

> Every metric label has a cost.

> Every log event has a cost.

> Cross-AZ telemetry can create significant network cost.

> Non-production observability should not consume production-level resources by default.

> Cost optimization must never compromise critical alerting, security or compliance.

> Cost anomalies can reveal technical incidents.

> Optimize waste before reducing useful telemetry.

> Observability is a production platform and should have measurable reliability and cost objectives.

---

# 123. Final DevOps Mental Model

For production observability cost optimization:

    MEASURE
       |
       v
    ATTRIBUTE
       |
       v
    REMOVE WASTE
       |
       v
    CONTROL CARDINALITY
       |
       v
    CONTROL LOG VOLUME
       |
       v
    OPTIMIZE RETENTION
       |
       v
    OPTIMIZE STORAGE
       |
       v
    OPTIMIZE QUERIES
       |
       v
    OPTIMIZE NETWORK
       |
       v
    RIGHT-SIZE INFRASTRUCTURE
       |
       v
    VALIDATE RELIABILITY
       |
       v
    CONTINUOUSLY REVIEW

This completes:

    19-Best-Practices/
        04-Cost-Optimization.md

Next:

    05-Common-Mistakes.md
