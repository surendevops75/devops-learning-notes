# Performance

> Production Monitoring & Observability Performance — Prometheus, Grafana, ELK, Kubernetes/EKS, Metrics, Logs, Queries, Dashboards, Alerting, Storage, Network, Applications, Capacity Planning, Troubleshooting, Optimization and DevOps/DevSecOps Interview Preparation

---

# 1. Purpose

Observability systems are production systems. Poorly designed monitoring can itself create CPU, memory, disk, network, latency, storage and cost problems.

The objective is:

    Collect useful telemetry
        |
        v
    Process efficiently
        |
        v
    Store appropriately
        |
        v
    Query efficiently
        |
        v
    Alert quickly
        |
        v
    Maintain predictable cost

Performance must be considered across:

    Collection
    Processing
    Storage
    Querying
    Visualization
    Alerting
    Network
    Application overhead

---

# 2. Core Performance Principles

1. Measure before optimizing.
2. Control metric cardinality.
3. Keep scrape intervals appropriate.
4. Avoid unnecessary telemetry.
5. Optimize expensive PromQL.
6. Use recording rules for repeated calculations.
7. Keep Grafana dashboards focused.
8. Control log volume.
9. Design Elasticsearch indexes deliberately.
10. Monitor queues and backpressure.
11. Maintain storage and resource headroom.
12. Size observability for incident conditions.
13. Optimize before scaling when practical.
14. Scale when workload genuinely requires it.
15. Monitor the observability platform itself.

---

# 3. Production Architecture

Typical architecture:

    Applications
        |
        +---- Metrics ----> Prometheus ----> Grafana
        |
        +---- Logs -------> Collector ----> Logstash ----> Elasticsearch
        |                                               |
        |                                               v
        |                                             Kibana
        |
        +---- Traces ------> Trace Backend where implemented

Performance bottlenecks can occur at every stage.

Always ask:

    Is collection slow?
    Is processing slow?
    Is storage slow?
    Is querying slow?
    Is visualization slow?
    Is alert evaluation slow?

---

# 4. Application Telemetry Overhead

Instrumentation consumes:

    CPU
    Memory
    Network
    Serialization time
    Application processing time

Examples:

    Metric generation
    JSON logging
    Log formatting
    Trace/span creation
    Context propagation

Avoid:

    Logging every request unnecessarily
    Creating thousands of unnecessary metrics
    High-cardinality labels
    Expensive work inside /metrics endpoints

Telemetry should provide operational value without materially degrading the application.

---

# 5. Prometheus Performance

Prometheus performance is strongly influenced by:

    Active time series
    Samples/sec
    Scrape interval
    Label cardinality
    Query complexity
    Rule evaluation
    Retention
    Disk performance
    Remote write

A Prometheus performance problem is often a telemetry-design problem rather than simply a CPU problem.

---

# 6. Time Series and Cardinality

Every unique label combination creates a separate time series.

Example:

    http_requests_total{
      service="orders",
      method="GET",
      status="200"
    }

Dangerous labels:

    user_id
    request_id
    session_id
    transaction_id
    email

Prefer bounded labels:

    service
    method
    route
    status_code
    environment

High cardinality increases:

    Memory
    CPU
    Disk
    Query cost
    Startup/recovery time

---

# 7. Cardinality Calculation

Conceptually:

    Total combinations =
    product of possible values of every label

Example:

    10 services
    x 5 methods
    x 10 status codes
    x 4 environments

    = 2,000 combinations

Adding a label with thousands of values can multiply the number of series dramatically.

Before adding a label ask:

    Is it bounded?
    Does it grow with requests?
    Is it necessary for an operational decision?

If it grows with every request, prefer logs/traces for that context.

---

# 8. Metric Churn

Metric churn occurs when time series are constantly created and removed.

Common causes:

    Short-lived pods
    Jobs
    Aggressive autoscaling
    Dynamic label values
    High pod churn

High churn can increase:

    CPU
    Memory
    Storage
    Query overhead

Control unnecessary dynamic dimensions.

---

# 9. Scrape Interval

Shorter scrape intervals mean:

    More samples
    More network traffic
    More CPU
    More storage

Do not use a very short interval for every target.

Choose based on:

    Detection requirement
    Signal importance
    Data value
    Storage capacity

Critical service metrics may need higher resolution than slow-changing infrastructure metrics.

---

# 10. Scrape Timeout

A target should respond within a reasonable timeout.

Monitor:

    up
    scrape_duration_seconds
    scrape_samples_scraped

If scrape duration approaches the scrape interval, investigate:

    Metrics endpoint
    Network
    Target CPU
    Prometheus load

---

# 11. Metrics Endpoint Performance

An application /metrics endpoint should be lightweight.

Avoid:

    Database queries
    External API calls
    Expensive calculations
    Large dynamic data generation

Metrics collection should ideally expose already-maintained application counters/histograms rather than perform expensive work during every scrape.

---

# 12. Exporter Performance

Exporters consume resources.

Examples:

    node_exporter
    database exporters
    blackbox_exporter

Best practices:

    Enable required collectors only
    Remove unnecessary metrics
    Monitor exporter CPU/memory
    Avoid duplicate exporters
    Protect exporter endpoints

---

# 13. Relabeling

Use relabeling to:

    Keep required targets
    Drop unnecessary targets
    Normalize labels
    Remove unwanted dimensions

For Kubernetes, relabeling can prevent every discovered target from becoming a production metric target.

---

# 14. Metric Relabeling

Metric relabeling can drop unnecessary metrics after scraping.

Example:

    Exporter exposes 5,000 metrics
       |
       v
    Keep useful metrics
       |
       v
    Drop irrelevant metrics

Important:

> Metric relabeling reduces what is stored, but the scrape itself still happened.

For major performance improvements, control the exporter and target configuration too.

---

# 15. Recording Rules

Use recording rules for expensive repeated PromQL.

Example:

    service:http_error_rate:5m

Good candidates:

    SLO calculations
    Frequently used dashboard queries
    Common aggregations
    Repeated alert expressions

Benefits:

    Faster dashboards
    Faster alerts
    Consistent calculations
    Lower repeated query cost

Do not create thousands of unnecessary recording rules.

---

# 16. PromQL Performance

Query cost depends on:

    Number of series
    Time range
    Aggregation
    Regex
    Joins
    Subqueries
    Functions

Prefer narrow selectors.

Instead of:

    {job=~".*"}

prefer:

    {job="kubernetes-pods"}

where appropriate.

Aggregate at the level actually required by the dashboard.

---

# 17. Query Time Range

Do not make every dashboard query:

    30 days

Use:

    15m
    1h
    6h
    24h

for operational views.

Use long ranges intentionally for:

    Capacity planning
    Trend analysis
    SLO analysis

---

# 18. Query Resolution

Long time ranges with extremely high resolution create unnecessary work.

Example:

    30 days
    second-level visualization

Instead choose a resolution appropriate to the visualization.

Operational dashboards should prioritize fast decision-making over unnecessary detail.

---

# 19. Prometheus Storage

Prometheus storage performance depends on:

    Active series
    Samples/sec
    Disk IOPS
    Disk latency
    Retention
    Query workload

Monitor:

    Disk usage
    Disk latency
    WAL behavior
    Ingestion rate

Do not wait for disk usage to reach 100%.

---

# 20. Prometheus WAL

Prometheus uses a write-ahead log for durable ingestion.

High ingestion can increase:

    WAL writes
    Disk I/O
    Recovery time

If WAL-related performance degrades, investigate:

    Disk performance
    Ingestion rate
    Series growth
    Storage capacity

---

# 21. Retention

Longer retention means:

    More disk
    More historical data
    Potentially larger queries

Choose retention based on:

    Incident investigation
    Capacity planning
    SLO analysis
    Compliance

For long-term storage, use an appropriate remote-storage architecture rather than keeping unlimited local data.

---

# 22. Remote Write

Remote write can provide:

    Long-term retention
    Centralized metrics
    Multi-cluster aggregation

But adds:

    Network traffic
    CPU
    Memory
    Queueing
    Failure modes

Monitor remote-write queues and failures.

---

# 23. Prometheus Query Concurrency

Too many simultaneous queries can cause:

    CPU saturation
    Memory pressure
    Slow dashboards
    Delayed alerts

During incidents query load may increase because many engineers investigate simultaneously.

Use:

    Efficient dashboards
    Recording rules
    Appropriate refresh intervals
    Adequate capacity

---

# 24. Alert Rule Performance

Every alert expression consumes evaluation resources.

Hundreds of expensive rules can overload Prometheus.

Best practices:

    Keep expressions focused
    Use recording rules
    Avoid unnecessarily long ranges
    Review evaluation intervals
    Remove obsolete alerts

---

# 25. Alert Delays

Alert path:

    Metric scraped
       |
       v
    Rule evaluated
       |
       v
    Alertmanager
       |
       v
    Notification

If Prometheus is overloaded, rule evaluation can be delayed.

Therefore monitor:

    Rule evaluation duration
    Prometheus CPU
    Query load
    Alertmanager queue
    Notification failures

---

# 26. Alert Storms

One root cause can create hundreds of alerts.

Example:

    Node failure
       |
       +---- Pods fail
       +---- Services fail
       +---- Applications fail
       +---- Dependency alerts fire

Use:

    Grouping
    Inhibition
    Routing
    Sensible severity

A single root cause should not become an alert flood.

---

# 27. Grafana Performance

Slow dashboards can be caused by:

    Too many panels
    Too many queries
    Expensive PromQL
    Long time ranges
    Frequent refresh
    Slow data source
    Expensive Elasticsearch aggregations

Troubleshoot the entire chain:

    Browser
       |
       v
    Grafana
       |
       v
    Query
       |
       v
    Data source
       |
       v
    Storage

---

# 28. Dashboard Design

Prefer:

    Overview
       |
       v
    Service
       |
       v
    Dependency
       |
       v
    Detailed troubleshooting

Avoid one dashboard containing hundreds of unrelated panels.

A dashboard should help an engineer make a decision quickly.

---

# 29. Dashboard Refresh

Refreshing every second can create unnecessary query load.

Choose refresh based on:

    Incident requirements
    Metric resolution
    Query cost

For normal operations, longer refresh intervals are usually sufficient.

---

# 30. Grafana Variables

Variables such as:

    namespace
    service
    pod

are useful, but variable queries themselves can be expensive.

Keep variable queries:

    Narrow
    Bounded
    Cached where appropriate
    Relevant to the dashboard

---

# 31. Data Source Performance

When Grafana is slow determine whether the bottleneck is:

    Grafana
    Prometheus
    Elasticsearch
    Network
    Storage

Do not assume Grafana is the cause simply because the browser is showing a slow page.

---

# 32. Elasticsearch Performance

Monitor:

    CPU
    JVM heap
    Garbage collection
    Disk I/O
    Disk usage
    Indexing rate
    Search latency
    Rejected requests
    Shard count
    Segment/index growth

---

# 33. Elasticsearch JVM

High heap pressure can produce:

    Long GC
    Slow searches
    Slow indexing
    Rejected operations
    OutOfMemoryError

Do not automatically keep increasing heap.

Investigate:

    Shards
    Queries
    Mappings
    Index size
    Log volume

---

# 34. Elasticsearch Shards

Too many shards cause:

    Memory overhead
    Metadata overhead
    Management complexity

Too few shards can limit:

    Parallelism
    Distribution

Shard design must consider:

    Data volume
    Node count
    Query patterns
    Retention
    Growth

---

# 35. Small-Shard Problem

Thousands of tiny indexes/shards can degrade cluster performance.

Common causes:

    Excessive daily indexes
    Poor rollover design
    Too many environments
    Over-segmentation

Review shard sizing and index lifecycle.

---

# 36. Index Lifecycle Management

Use lifecycle policies for:

    Rollover
    Hot storage
    Warm storage
    Cold storage
    Deletion

This prevents old logs from consuming active storage indefinitely.

---

# 37. Elasticsearch Mappings

Poor mappings increase:

    Indexing cost
    Storage
    Query cost

Avoid dynamically indexing every possible application field.

Define fields based on actual search/aggregation requirements.

---

# 38. Keyword vs Text

Use:

    keyword

for:

    Exact matching
    Filtering
    Aggregation

Use:

    text

for:

    Full-text search

Correct mappings improve query performance and storage efficiency.

---

# 39. Log Field Explosion

A large JSON event may contain hundreds of fields.

If every field becomes searchable:

    Mapping count ↑
    Storage ↑
    Indexing cost ↑

Prefer a controlled schema.

---

# 40. Kibana Performance

Slow Kibana dashboards may result from:

    Too many panels
    Broad time range
    Expensive aggregations
    High-cardinality fields
    Slow Elasticsearch

Start queries with:

    Time range
    Environment
    Service
    Log level

Then narrow further.

---

# 41. Logstash Performance

Logstash performance depends on:

    Events/sec
    Filter complexity
    Worker count
    Batch size
    Output speed
    Queue configuration

Monitor:

    Events in
    Events out
    Queue size
    CPU
    Memory
    Output failures

---

# 42. Grok and Regex Performance

Complex Grok/regex processing can consume significant CPU.

Prefer:

    Structured application logs

over:

    Parsing large arbitrary text

where the application can produce structured fields directly.

---

# 43. Logstash Backpressure

Architecture:

    Collector
       |
       v
    Logstash
       |
       v
    Elasticsearch

If Elasticsearch slows:

    Output slows
       |
       v
    Queue grows
       |
       v
    Memory/disk pressure

The queue is a symptom of a throughput mismatch.

---

# 44. Persistent Queue

Persistent queues can protect logs during short downstream failures.

But they require:

    Disk
    I/O
    Capacity planning

Monitor queue size and available disk.

A queue cannot solve a permanent ingestion mismatch.

---

# 45. Log Volume

Track:

    Events/sec
    Bytes/sec
    Events by service
    Error logs/sec

A sudden increase may indicate:

    Deployment problem
    Error loop
    Retry storm
    Attack
    Configuration change

---

# 46. Log Storm

Example:

    Application error loop
       |
       v
    50x log volume
       |
       v
    Logstash overloaded
       |
       v
    Elasticsearch overloaded
       |
       v
    Troubleshooting becomes harder

Mitigation:

    Fix root cause
    Reduce unnecessary logs
    Rate-limit where safe
    Sample repetitive events
    Protect the logging pipeline

---

# 47. Telemetry Reliability

Telemetry can be lost because of:

    Network failure
    Collector failure
    Queue overflow
    Disk exhaustion
    Elasticsearch rejection
    Prometheus overload

Monitor:

    Dropped samples
    Dropped events
    Queue depth
    Ingestion failures
    Storage capacity

Never assume:

> "The application wrote the log, so it must be searchable."

---

# 48. Kubernetes Observability Performance

Kubernetes monitoring load comes from:

    Pods
    Nodes
    Kubelets
    API server
    Controllers
    Services
    Exporters
    Applications

Performance depends heavily on:

    Cluster size
    Pod count
    Target count
    Scrape frequency
    Cardinality
    Pod churn

---

# 49. Kubernetes API Server Load

Excessive discovery or monitoring requests can increase API server load.

Use:

    Efficient discovery
    Appropriate watch mechanisms
    Target filtering
    Avoid unnecessary polling

Monitor API server performance.

---

# 50. Pod Churn

High pod creation/deletion rates can create metric churn.

Examples:

    Jobs
    Short-lived workloads
    Aggressive autoscaling

Review:

    Series churn
    Target churn
    Kubernetes API activity

---

# 51. Observability Workload Isolation

In larger EKS environments consider:

    Dedicated observability node pool

Benefits:

    Resource isolation
    Predictable performance
    Easier capacity planning

Tradeoff:

    Additional infrastructure cost

Use:

    Taints/tolerations
    Node affinity
    Requests/limits

when justified.

---

# 52. Resource Requests and Limits

Observability components need realistic resources.

Important workloads:

    Prometheus
    Grafana
    Logstash
    Elasticsearch

Too-low requests cause:

    Contention
    Scheduling problems

Too-low memory limits can cause:

    OOMKilled

Too-high requests can waste capacity.

Use historical usage to tune.

---

# 53. Prometheus CPU Troubleshooting

High CPU can be caused by:

    Expensive queries
    High ingestion
    High cardinality
    Many scrapes
    Rule evaluation

Investigation:

    Check recent changes
    Check series growth
    Check query load
    Check scrape rate
    Check recording rules

Optimize before scaling.

---

# 54. Prometheus Memory Troubleshooting

High memory can be caused by:

    High active series
    High cardinality
    Ingestion increase
    Query workload
    Rule evaluation

Do not immediately add memory.

First identify the workload.

---

# 55. Elasticsearch Disk Troubleshooting

If disk usage is high:

    Identify index growth
    Check log ingestion rate
    Check retention
    Check large indexes
    Check shard design
    Check lifecycle policies

Then:

    Apply eligible retention
    Expand capacity
    Fix ingestion source

Never blindly delete active production data.

---

# 56. Network Performance

Telemetry consumes:

    Metrics bandwidth
    Log bandwidth
    Dashboard query traffic
    Search results

Monitor:

    Throughput
    Latency
    Packet errors
    Connections

In AWS, consider cross-AZ traffic because high-volume telemetry can increase both latency and cost.

---

# 57. Availability-Zone Placement

For production:

    Spread critical observability components across failure domains where appropriate.

Consider placement of:

    Prometheus replicas
    Elasticsearch nodes
    Collectors
    Grafana replicas

Balance:

    Availability
    Network performance
    Cost

---

# 58. High Availability Tradeoff

HA improves availability but adds:

    CPU
    Memory
    Network
    Storage
    Operational complexity

Choose HA based on:

    Service criticality
    RTO/RPO
    Scale
    Budget

---

# 59. Capacity Planning

Track:

    Samples/sec
    Active series
    Log events/sec
    Log GB/day
    Query rate
    Dashboard users
    Alert rate
    Storage growth

Then forecast:

    CPU
    Memory
    Disk
    Network

requirements.

---

# 60. Capacity Headroom

Do not run the observability platform at sustained near-maximum capacity.

Leave room for:

    Traffic spikes
    Deployment events
    Incident query storms
    Log storms
    Recovery
    Unexpected growth

---

# 61. Incident Load

During an incident:

    More engineers
       |
       v
    More dashboard loads
       |
       v
    More queries
       |
       v
    More logs searched
       |
       v
    More alerts

Therefore:

> Size observability for incident conditions, not just normal traffic.

---

# 62. Query Storm

A dashboard used by 20 engineers can create a large number of repeated queries.

Mitigation:

    Efficient queries
    Recording rules
    Sensible refresh
    Focused dashboards
    Caching where appropriate
    Adequate backend capacity

---

# 63. Observability SLOs

The monitoring platform can have its own SLOs.

Examples:

    Metrics freshness
    Alert delivery latency
    Dashboard availability
    Log ingestion delay

Example:

    Critical alerts should reach the on-call team within the defined target.

---

# 64. Data Freshness

A dashboard can be:

    Available

but show:

    10-minute-old data.

Therefore measure:

    Availability
    Freshness
    Ingestion delay

A monitoring system is useful only if its data is sufficiently current.

---

# 65. Performance Testing

Before a major production rollout test:

    Normal ingestion
    Peak ingestion
    Query load
    Dashboard load
    Alert load
    Storage growth
    Failure behavior
    Recovery

Test at:

    1x
    2x
    Peak
    Incident-like load

where practical.

---

# 66. Performance Regression

Monitor changes after:

    New exporter
    New application
    New metric label
    New dashboard
    New alert rule
    New log parser
    New retention policy

Compare:

    Before
    vs
    After

A small configuration change can produce a large performance regression.

---

# 67. Performance Optimization Priority

Use this order:

    1. Remove unnecessary telemetry
    2. Reduce cardinality
    3. Filter unnecessary targets
    4. Optimize queries
    5. Optimize dashboards
    6. Optimize retention/storage
    7. Improve processing pipeline
    8. Scale infrastructure

Do not make:

    "Add more CPU"

the first troubleshooting step.

---

# 68. Production Incident — Prometheus Memory Spike

Symptoms:

    Prometheus memory ↑
    Query latency ↑
    Alerts delayed

Investigation:

    Check active series
    Check new labels
    Check exporters
    Check dashboard changes
    Check recording rules
    Check ingestion

Likely cause:

    High-cardinality metric introduced.

Resolution:

    Remove/drop offending dimension
    Reduce series
    Recover Prometheus if necessary
    Add cardinality review

Prevention:

    Instrumentation standards
    Cardinality monitoring
    Code review

---

# 69. Production Incident — Grafana Slow

Symptoms:

    Dashboard takes 20 seconds

Investigation:

    Check panel count
    Identify slow panel
    Inspect PromQL
    Check time range
    Check series count
    Check data source

Resolution:

    Narrow query
    Recording rule
    Reduce panels
    Reduce refresh rate

---

# 70. Production Incident — Elasticsearch Disk Full

Symptoms:

    Disk > 90%
    Indexing errors

Investigation:

    Index growth
    Log volume
    Retention
    Shards
    Large indexes

Immediate mitigation:

    Remove eligible old data
    Expand storage
    Reduce unnecessary ingestion

Prevention:

    ILM
    Capacity alerts
    Growth forecasting

---

# 71. Production Incident — Logstash Queue Growth

Symptoms:

    Queue growing
    Elasticsearch indexing slow

Trace:

    Collector
       |
       v
    Logstash
       |
       v
    Elasticsearch

Check Elasticsearch:

    JVM
    Disk
    CPU
    Indexing latency
    Rejected requests

Fix the downstream bottleneck instead of endlessly increasing queue size.

---

# 72. Production Incident — Alerts Arrive Late

Example:

    Condition: 10:00
    Alert:     10:08

Trace:

    Scrape
       |
       v
    Rule evaluation
       |
       v
    Alertmanager
       |
       v
    Notification

Check each stage for delay.

---

# 73. Production Incident — Log Storm

Symptoms:

    Errors increase
    Log volume spikes
    Elasticsearch slows

Investigate:

    Deployment
    Retry loop
    Dependency failure
    Application bug

Mitigate:

    Fix root cause
    Reduce repetitive logging
    Protect pipeline
    Preserve important errors

---

# 74. Production Incident — Kubernetes Monitoring Overload

Symptoms:

    Prometheus CPU high
    API server load high

Check:

    Target count
    Service discovery
    Pod churn
    Scrape interval
    Exporters
    Cardinality

Fix:

    Filter targets
    Reduce scrape frequency
    Reduce unnecessary metrics
    Control labels

---

# 75. Production Incident — Disk 100%

Always determine:

    Which partition?
    Which component?
    What is consuming it?
    What is the growth rate?

Use Linux tools where appropriate:

    df -h
    du -sh
    iostat

Then correlate with observability data.

---

# 76. Troubleshooting Workflow

For any observability performance incident:

    1. Define the symptom
    2. Measure impact
    3. Compare with baseline
    4. Check recent changes
    5. Identify bottleneck
    6. Reduce unnecessary workload
    7. Optimize configuration/query
    8. Scale if required
    9. Validate recovery
    10. Document prevention

---

# 77. Application Performance vs Observability Performance

If application latency increases after adding logging:

    Application
       |
       v
    More logs
       |
       v
    Serialization/CPU
       |
       v
    Application latency ↑

Observability must not become the cause of application degradation.

Test telemetry overhead.

---

# 78. Logging Performance Best Practices

Prefer:

    Structured logs
    Appropriate log levels
    Asynchronous collection where suitable
    Efficient parsing
    Controlled event size
    Retention policies
    Sampling for repetitive high-volume events

Avoid:

    Full request payloads everywhere
    DEBUG globally
    Complex regex for every event
    Logging secrets

---

# 79. Log Event Size

Large events increase:

    Network bandwidth
    Parsing CPU
    Storage
    Search cost

Log only information necessary for:

    Debugging
    Operations
    Security
    Business requirements

---

# 80. Sampling

Sampling can reduce high-volume telemetry.

For example:

    Normal successful requests:
    Lower sample rate

    Errors:
    Higher retention

    Slow requests:
    Higher retention

Sampling should preserve information required for incidents.

---

# 81. Backpressure

Backpressure occurs when:

    Producer rate > Consumer rate

Example:

    Logs generated:
    100,000 events/sec

    Logs indexed:
    60,000 events/sec

The difference accumulates in queues.

Backpressure is a capacity signal.

---

# 82. Queue Monitoring

For collectors and Logstash monitor:

    Queue depth
    Queue growth rate
    Oldest queued event
    Output failures
    Disk usage

A growing queue indicates the downstream system cannot keep up.

---

# 83. Cost and Performance

Performance optimization often reduces cost.

Reducing:

    Metric cardinality
    Log volume
    Query frequency
    Retention

can reduce:

    CPU
    Memory
    Storage
    Network
    Cloud spend

But do not remove critical telemetry solely to reduce cost.

---

# 84. Performance Governance

Establish standards for:

    Metric naming
    Label design
    Scrape intervals
    Log levels
    Event size
    Dashboard refresh
    Query complexity
    Retention

Governance prevents repeated performance problems.

---

# 85. Performance Budget

For larger platforms define budgets such as:

    Maximum expected metric cardinality
    Maximum log GB/day
    Maximum telemetry CPU overhead
    Maximum query latency

Exact limits should come from platform capacity and business requirements.

---

# 86. Service Onboarding Performance Review

Before onboarding a service ask:

## Metrics

    How many series?
    What labels?
    What scrape interval?

## Logs

    Events/sec?
    Average event size?
    Peak volume?

## Dashboards

    Number of queries?
    Refresh interval?
    Expected users?

## Alerts

    How expensive?
    Evaluation frequency?

## Storage

    Retention?
    Growth rate?

---

# 87. Production Observability Checklist

## Prometheus

    [ ] Cardinality controlled
    [ ] Scrape intervals justified
    [ ] Targets filtered
    [ ] Recording rules used
    [ ] PromQL optimized
    [ ] Disk sized
    [ ] WAL monitored
    [ ] Query load monitored

## Grafana

    [ ] Panels controlled
    [ ] Refresh interval reasonable
    [ ] Variables optimized
    [ ] Queries efficient
    [ ] Data sources healthy

## ELK

    [ ] Log volume controlled
    [ ] Parsing efficient
    [ ] Logstash queues monitored
    [ ] Elasticsearch JVM monitored
    [ ] Shards reviewed
    [ ] ILM configured
    [ ] Disk headroom maintained

## Kubernetes/EKS

    [ ] Requests/limits
    [ ] Observability isolation where required
    [ ] API load controlled
    [ ] Target discovery optimized
    [ ] Network capacity planned

## Reliability

    [ ] Metrics freshness
    [ ] Alert latency
    [ ] Log ingestion latency
    [ ] Dashboard availability
    [ ] Incident load tested
    [ ] Recovery tested

---

# 88. Senior DevOps Interview — How Do You Improve Prometheus Performance?

Strong answer:

> I start with active series, cardinality, ingestion rate, scrape duration, query latency and rule evaluation. I look for high-cardinality labels, unnecessary targets, expensive PromQL, excessive scrape frequency and expensive rules. I use relabeling and recording rules where appropriate, optimize dashboards and only scale CPU, memory or storage after understanding the workload.

---

# 89. Interview — What Causes Prometheus Memory Growth?

Strong answer:

> Common causes include increased active time series, high-cardinality labels, higher ingestion rates, expensive queries and rule workloads. In Kubernetes I also check pod churn and newly deployed exporters. I compare the change with recent deployments and series growth before simply increasing memory.

---

# 90. Interview — How Do You Optimize Grafana?

Strong answer:

> I reduce unnecessary panels and expensive queries, narrow time ranges, avoid excessive refresh rates, optimize variables, use recording rules for repeated calculations and aggregate data at the level the dashboard needs. I also determine whether the bottleneck is Grafana, Prometheus, Elasticsearch, the network or storage.

---

# 91. Interview — How Do You Improve Elasticsearch Performance?

Strong answer:

> I monitor JVM heap, GC, CPU, disk I/O, indexing rate, search latency, rejected operations and shard count. I control log volume, use appropriate mappings and lifecycle policies, avoid excessive small shards, optimize queries and ensure storage has adequate performance and capacity.

---

# 92. Interview — How Do You Handle a Log Storm?

Strong answer:

> I first identify why the log volume increased, such as an error loop, retry storm or deployment. I protect the logging pipeline using buffering, rate control or sampling where safe, while preserving critical error information. Then I fix the application or dependency causing the storm rather than indefinitely scaling Elasticsearch.

---

# 93. Interview — How Do You Handle High Cardinality?

Strong answer:

> I identify the labels creating series growth, especially user IDs, request IDs or other unbounded values. I replace them with bounded dimensions or move high-cardinality context into logs or traces. I can also drop unnecessary metrics with relabeling and enforce cardinality review before production.

---

# 94. Interview — Why Can Observability Become a Production Problem?

Strong answer:

> Observability consumes CPU, memory, network and storage. High-cardinality metrics, excessive logs, expensive queries and large Elasticsearch workloads can compete with applications or delay alerts. Therefore I monitor the monitoring platform itself and design it with resource isolation, capacity, backpressure and failure handling.

---

# 95. Interview — How Do You Design Observability for a Large EKS Cluster?

Strong answer:

> I control Kubernetes target discovery, metric cardinality and scrape frequency, deploy collectors efficiently, isolate observability workloads where needed, plan storage and network capacity, optimize dashboards and use high availability for critical components. I also track ingestion and query performance as the cluster grows.

---

# 96. Interview — How Do You Plan Observability Capacity?

Strong answer:

> I establish baselines for samples per second, active series, log events per second, bytes per day, query volume and peak incident load. I track growth trends, maintain headroom and forecast CPU, memory, storage and network requirements. I size for incident conditions rather than only normal traffic.

---

# 97. Final Performance Mental Model

When something is slow:

    WHAT changed?
          |
          v
    WHERE is the bottleneck?
          |
          v
    HOW MUCH workload exists?
          |
          v
    CAN the workload be reduced?
          |
          v
    CAN the design/query be optimized?
          |
          v
    DOES it require more capacity?
          |
          v
    HOW DO WE PREVENT RECURRENCE?

Do not start with:

> "Add more CPU."

Start with:

> "What workload is consuming the resource, and why?"

---

# 98. Final Principles

> Measure before optimizing.

> High cardinality is one of the most important Prometheus performance risks.

> Every metric label has a cost.

> Every log event has ingestion and storage cost.

> Every dashboard query consumes resources.

> Every alert rule consumes evaluation resources.

> Observability must be sized for incident conditions.

> A buffer can absorb a burst, but it cannot solve a permanent throughput mismatch.

> Optimize before scaling whenever practical.

> Scale when workload genuinely requires it.

> Monitor freshness, not only availability.

> Treat Prometheus, Grafana and ELK as production services.

> Performance, reliability and cost are connected.

> The best observability platform provides useful information quickly without becoming a production bottleneck.

This completes:

    19-Best-Practices/
        03-Performance.md

Next:

    04-Cost-Optimization.md
