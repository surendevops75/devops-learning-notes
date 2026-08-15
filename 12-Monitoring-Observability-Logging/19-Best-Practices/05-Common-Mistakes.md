# Common Mistakes

> Production Monitoring & Observability Common Mistakes — Prometheus, Grafana, ELK, Kubernetes/EKS, Metrics, Logs, Alerting, Dashboards, Storage, Security, Performance, Cost, Troubleshooting and DevOps/DevSecOps Interview Preparation

---

# 1. Purpose

Observability failures are often caused not by missing tools, but by poor design and operational mistakes.

Common examples:

    Too many metrics
    High-cardinality labels
    Excessive logging
    Poor retention
    Expensive queries
    Alert fatigue
    Unprotected dashboards
    No monitoring of the monitoring platform
    Incorrect Kubernetes resource sizing
    Poor Elasticsearch shard design
    No backup/DR strategy
    No capacity planning
    No ownership

The goal of this file is to understand:

    What the mistake is
        |
        v
    Why it happens
        |
        v
    Production impact
        |
        v
    How to detect it
        |
        v
    How to fix it
        |
        v
    How to prevent it

---

# 2. Mistake — Treating Monitoring and Observability as the Same Thing

Monitoring generally answers:

    Is the system healthy?

Observability helps answer:

    Why is the system behaving this way?

Weak implementation:

    CPU dashboard
    Memory dashboard
    Disk dashboard

Better:

    Metrics
    Logs
    Context
    Service relationships
    SLOs
    Incident correlation

Production lesson:

> Monitoring tells you that something is wrong. Observability helps you investigate why.

---

# 3. Mistake — Installing Tools Without a Strategy

Common approach:

    Install Prometheus
    Install Grafana
    Install ELK
    Create dashboards

But nobody defines:

    What to monitor
    What to alert
    What to retain
    Who owns alerts
    What SLOs exist
    What data is required during incidents

Result:

    Large platform
    High cost
    Low operational value

Correct approach:

    Requirements
       |
       v
    Signals
       |
       v
    Architecture
       |
       v
    Collection
       |
       v
    Alerting
       |
       v
    Incident response
       |
       v
    Continuous improvement

---

# 4. Mistake — Collecting Everything

More telemetry does not automatically mean better observability.

Collecting everything creates:

    CPU usage
    Memory usage
    Network usage
    Storage usage
    Query complexity
    Cost

Ask:

    Is this data useful?
    Is it actionable?
    Will someone investigate it?
    Is it needed for security/compliance?

Remove unnecessary telemetry.

---

# 5. Mistake — High-Cardinality Prometheus Labels

Bad:

    http_requests_total{
      user_id="123456",
      request_id="abc..."
    }

Each unique value can create a new time series.

Impact:

    Prometheus memory ↑
    CPU ↑
    Disk ↑
    Query latency ↑
    Remote storage cost ↑

Better:

    service
    route
    method
    status_code

Use logs or traces for request-specific identifiers.

---

# 6. Mistake — Using request_id as a Metric Label

A request ID is almost always high cardinality.

Bad:

    http_request_duration_seconds{
      request_id="unique-per-request"
    }

This can create enormous series churn.

Better:

    Metric:
      service + route + method

    Log:
      request_id

This separates:

    Aggregated telemetry
    from
    Individual request context

---

# 7. Mistake — Using user_id as a Metric Label

Bad:

    api_requests_total{
      user_id="customer123"
    }

If the application has millions of users, Prometheus may receive an enormous number of series.

Better:

    Log user_id when appropriate
    Use bounded metric dimensions

Also ensure personal data handling follows security and privacy requirements.

---

# 8. Mistake — Excessive Histogram Buckets

A histogram can create multiple bucket series.

Example:

    request_duration_seconds_bucket

Too many buckets increase:

    Series count
    Storage
    Query cost

Use bucket boundaries that answer real SLO or troubleshooting questions.

---

# 9. Mistake — Scraping Too Frequently

Example:

    Every target:
    1 second

Possible result:

    Ingestion ↑
    CPU ↑
    Network ↑
    Storage ↑

Ask:

    Does the service actually need one-second resolution?

Choose intervals based on detection requirements.

---

# 10. Mistake — One Scrape Interval for Everything

Using:

    15 seconds

for every target may be unnecessary.

Better:

    Critical applications:
    Higher resolution

    Standard workloads:
    Moderate resolution

    Slow infrastructure:
    Lower resolution

Optimization should follow operational requirements.

---

# 11. Mistake — Ignoring Scrape Failures

Seeing:

    up == 0

is not the end of the investigation.

Check:

    Network
    Target health
    Metrics endpoint
    DNS
    Service discovery
    Authentication
    Prometheus load

A scrape failure may indicate either:

    Target failure
    or
    Monitoring-path failure

---

# 12. Mistake — Slow /metrics Endpoint

Bad design:

    /metrics
       |
       +---- Database query
       +---- External API
       +---- Heavy computation

Every scrape can trigger expensive work.

Better:

    Maintain metrics efficiently
       |
       v
    Expose metrics cheaply

Metrics collection should not become an application bottleneck.

---

# 13. Mistake — Keeping Every Exporter Metric

Exporters may expose thousands of metrics.

Keeping all of them can increase:

    Cardinality
    Storage
    Query cost

Review:

    Used?
    Alerted?
    Dashboarded?
    Required?

Drop unnecessary telemetry when safe.

---

# 14. Mistake — Ignoring Metric Churn

Short-lived pods and dynamic labels can create high series churn.

Symptoms:

    Prometheus memory pressure
    High CPU
    Increasing series creation
    Storage overhead

Investigate:

    Pod churn
    Jobs
    Autoscaling
    Dynamic labels

---

# 15. Mistake — Using Expensive PromQL Everywhere

Bad:

    Very broad selectors
    Large time ranges
    Complex regex
    Expensive joins
    Large subqueries

Better:

    Narrow labels
    Aggregation
    Recording rules
    Appropriate time ranges

---

# 16. Mistake — Querying 30 Days by Default

Operational dashboards usually do not need:

    30-day
    high-resolution
    real-time refresh

every few seconds.

Use:

    15m
    1h
    6h
    24h

for routine operations.

Use long ranges intentionally.

---

# 17. Mistake — Not Using Recording Rules

If the same expensive query is used by:

    Dashboard
    Alert
    SLO

compute it once using a recording rule where appropriate.

Benefits:

    Faster queries
    Lower repeated CPU
    Consistent calculation

---

# 18. Mistake — Creating Too Many Recording Rules

The opposite mistake also happens.

Creating thousands of unused recording rules creates:

    Storage
    CPU
    Operational complexity

Use recording rules for:

    Frequently reused
    Expensive
    Operationally valuable

calculations.

---

# 19. Mistake — Building Huge Grafana Dashboards

Bad:

    200 panels
    100 queries
    5-second refresh

Impact:

    Slow browser
    High Prometheus load
    Poor incident experience

Better:

    Overview
       |
       v
    Service
       |
       v
    Detailed troubleshooting

---

# 20. Mistake — Dashboard for Everything

A dashboard should answer a question.

Examples:

    Is production healthy?
    Which service is failing?
    Is latency increasing?
    Which node is under pressure?

Avoid dashboards that display:

    Every metric ever created

---

# 21. Mistake — Too-Frequent Dashboard Refresh

Example:

    Refresh every 1 second

for metrics that change slowly.

This creates unnecessary queries.

Use refresh intervals appropriate to:

    Data resolution
    Incident requirements

---

# 22. Mistake — Unoptimized Grafana Variables

Variables such as:

    cluster
    namespace
    service
    pod

can trigger expensive data-source queries.

Keep variable queries:

    Narrow
    Bounded
    Efficient

---

# 23. Mistake — Ignoring Browser Performance

A dashboard can be slow because of:

    Too many panels
    Large result sets
    Heavy rendering
    Excessive transformations

Do not assume the backend is always the problem.

---

# 24. Mistake — Treating Logs as a Database

Logs are excellent for:

    Events
    Context
    Errors
    Request details

But using logs as a replacement for every application metric can create:

    Huge volume
    High cost
    Slow queries

Use metrics for aggregation and logs for detailed context.

---

# 25. Mistake — Logging Everything at DEBUG

Bad:

    DEBUG enabled globally in production

Impact:

    Storage explosion
    Network load
    Elasticsearch pressure
    Higher cost

Better:

    INFO normally
    ERROR for failures
    Temporary targeted DEBUG only when required

---

# 26. Mistake — Logging Sensitive Information

Never casually log:

    Passwords
    Access tokens
    Session secrets
    Private keys
    Sensitive personal data

Logs often have:

    Long retention
    Many users
    Broad access

Treat logs as sensitive production data.

---

# 27. Mistake — Logging Full Request/Response Payloads

This can create:

    Large events
    Privacy risk
    Security risk
    Storage cost

Log:

    Request ID
    Route
    Status
    Duration
    Relevant business/error context

rather than entire payloads unless specifically required and protected.

---

# 28. Mistake — Duplicate Logs

The same event may be logged by:

    Application
    Middleware
    Sidecar
    Collector

Result:

    One error becomes several events.

Review the pipeline and eliminate unnecessary duplication.

---

# 29. Mistake — Unstructured Logs Everywhere

Raw text is harder to:

    Search
    Parse
    Aggregate
    Alert on

Prefer structured fields:

    timestamp
    service
    environment
    level
    request_id
    status
    duration
    message

---

# 30. Mistake — Poor Log Levels

If everything is:

    ERROR

then alerts become meaningless.

If everything is:

    DEBUG

volume becomes excessive.

Use levels according to operational meaning.

---

# 31. Mistake — No Log Retention Strategy

Keeping everything forever causes:

    Disk growth
    Search slowdown
    Higher storage cost
    Larger Elasticsearch cluster

Define:

    Hot retention
    Warm retention
    Archive
    Deletion

---

# 32. Mistake — No Elasticsearch ILM

Without lifecycle policies:

    New logs arrive
       |
       v
    Old logs remain
       |
       v
    Storage grows
       |
       v
    Cluster grows
       |
       v
    Cost grows

Use ILM where appropriate.

---

# 33. Mistake — Excessive Elasticsearch Shards

More shards do not automatically mean better performance.

Too many shards cause:

    JVM overhead
    Metadata overhead
    CPU overhead
    Management complexity

Design shard strategy from:

    Data volume
    Query pattern
    Node count
    Retention

---

# 34. Mistake — Too Many Tiny Elasticsearch Indexes

Creating a new index for every small workload/day can produce:

    Many tiny shards
    Poor resource utilization

Use appropriate rollover/index strategies.

---

# 35. Mistake — Wrong Elasticsearch Mappings

Allowing every dynamic field to become searchable can create:

    Mapping explosion
    Storage growth
    Indexing overhead

Define useful schemas.

Use:

    keyword

for exact filters/aggregations.

Use:

    text

for full-text search.

---

# 36. Mistake — Using text for Everything

If a field is primarily:

    service
    environment
    status
    hostname

you usually need exact matching, not full-text analysis.

Appropriate mapping improves:

    Storage
    Query performance
    Aggregation

---

# 37. Mistake — Ignoring Elasticsearch JVM

High JVM heap pressure can cause:

    GC pauses
    Slow searches
    Slow indexing
    Rejections
    OutOfMemoryError

Monitor:

    Heap
    GC
    CPU
    Disk

---

# 38. Mistake — Fixing Elasticsearch by Only Adding RAM

Adding memory may hide the symptom.

First investigate:

    Shards
    Queries
    Index volume
    Mappings
    Log volume
    Retention

Then decide whether additional resources are necessary.

---

# 39. Mistake — Ignoring Disk I/O

Elasticsearch and Prometheus both depend heavily on storage.

A disk can have:

    Free space

but still be slow because of:

    High latency
    Low IOPS
    Saturated throughput

Monitor both:

    Capacity
    Performance

---

# 40. Mistake — No Storage Headroom

Bad:

    Disk allowed to reach 95–100%

Production systems need headroom for:

    Spikes
    Recovery
    Compaction
    Temporary files
    Incident load

Alert before capacity becomes critical.

---

# 41. Mistake — Ignoring Logstash Backpressure

If Elasticsearch slows:

    Logstash queue grows

If you only look at Logstash:

    "Logstash is slow."

But the real cause may be:

    Elasticsearch indexing bottleneck

Trace the pipeline end-to-end.

---

# 42. Mistake — Overusing Grok

Complex Grok patterns can consume significant CPU.

Prefer structured application logs where possible.

If parsing is required:

    Keep patterns simple
    Test performance
    Avoid pathological regex

---

# 43. Mistake — No Queue Monitoring

For logging pipelines monitor:

    Queue depth
    Queue growth
    Oldest event
    Output failures
    Disk

A growing queue is an early warning.

---

# 44. Mistake — Treating Backpressure as a Permanent Solution

A queue can absorb:

    Short burst

It cannot solve:

    Permanent producer > consumer rate

If:

    Producer = 100k events/sec
    Consumer = 60k events/sec

the queue will eventually fill.

Fix the throughput mismatch.

---

# 45. Mistake — No Telemetry Loss Monitoring

Logs or metrics can be lost due to:

    Collector failure
    Network failure
    Queue overflow
    Storage failure
    Backend rejection

Monitor:

    Dropped samples
    Dropped events
    Queue failures
    Scrape failures

---

# 46. Mistake — Assuming "Green Dashboard" Means Healthy

A dashboard can be green while:

    Data is stale
    Alerts are delayed
    Some targets are missing
    Logs are not indexed

Monitor:

    Availability
    Freshness
    Ingestion
    Alert latency

---

# 47. Mistake — Not Monitoring the Monitoring System

Critical mistake.

You monitor:

    Applications

but forget:

    Prometheus
    Grafana
    Logstash
    Elasticsearch
    Collectors

If Prometheus fails silently, application dashboards may look normal because they are simply stale.

---

# 48. Mistake — No Observability SLO

The observability platform itself should have objectives.

Examples:

    Metrics freshness
    Alert delivery latency
    Log ingestion delay
    Dashboard availability

Observability is a production service.

---

# 49. Mistake — Alerting on Everything

If every warning creates an alert:

    Alert volume explodes
    Engineers ignore alerts
    Important alerts are missed

An alert should represent:

    Actionable condition

Not:

    Interesting metric

---

# 50. Mistake — No Alert Ownership

An alert without an owner creates:

    Delayed response
    Escalation confusion
    Repeated incidents

Every critical alert should have:

    Owner
    Severity
    Runbook
    Escalation path

---

# 51. Mistake — Alerting on Symptoms Without Context

Bad:

    CPU > 80%

without understanding:

    Which service?
    Is latency affected?
    Is traffic increased?
    Is there user impact?

Prefer alerts aligned to:

    User impact
    SLOs
    Critical infrastructure conditions

---

# 52. Mistake — Alerting on CPU Alone

High CPU may be normal during:

    Traffic spike
    Batch job
    Autoscaling

Better correlation:

    CPU
    + latency
    + error rate
    + saturation

Use multiple signals when appropriate.

---

# 53. Mistake — Alert Fatigue

Symptoms:

    Hundreds of notifications
    Engineers silence alerts
    Critical alert buried

Fix:

    Deduplication
    Grouping
    Inhibition
    Better thresholds
    Severity
    Ownership

---

# 54. Mistake — No Alert Testing

A rule may look correct but fail because:

    Query is wrong
    Label is wrong
    Routing is wrong
    Notification is broken

Test:

    Firing
    Routing
    Delivery
    Recovery

---

# 55. Mistake — No Runbook

Alert:

    "High error rate"

without instructions is incomplete.

Runbook should include:

    What it means
    What to check
    Useful commands
    Dashboards
    Logs
    Likely causes
    Escalation

---

# 56. Mistake — No SLO-Based Alerting

Infrastructure alerts alone can produce noise.

Example:

    CPU 85%

may not mean user impact.

SLO-based alerting asks:

    Is availability degrading?
    Is latency exceeding target?
    Is error budget being consumed?

---

# 57. Mistake — Confusing SLI, SLO and SLA

SLI:

    Measurement

SLO:

    Internal target

SLA:

    External/business commitment

Example:

    SLI:
    Successful requests

    SLO:
    99.9% success

    SLA:
    Contractual commitment

---

# 58. Mistake — No Error Budget

Without error budgets teams may:

    Ship risky changes during poor reliability
    Ignore reliability degradation
    Focus only on feature delivery

Error budgets connect:

    Reliability
    and
    Delivery velocity

---

# 59. Mistake — Ignoring Burn Rate

A service can still meet a monthly SLO while consuming error budget dangerously fast.

Burn-rate alerts detect:

    Rapid budget consumption

This allows earlier intervention.

---

# 60. Mistake — No Baseline

Without a normal baseline, engineers cannot easily determine:

    Is this CPU normal?
    Is log volume normal?
    Is query latency normal?

Maintain baseline trends.

---

# 61. Mistake — No Capacity Planning

Observability grows with:

    Services
    Pods
    Nodes
    Logs
    Users
    Queries

Without planning:

    Prometheus memory fills
    Elasticsearch disk fills
    Alerts slow
    Costs explode

Track growth.

---

# 62. Mistake — Scaling Before Optimizing

Bad response:

    Prometheus CPU 90%
    Add 4 CPUs

without checking:

    Cardinality
    Expensive queries
    Scrape rate
    New exporter

Optimize the workload first.

---

# 63. Mistake — Optimizing Only for Normal Load

Monitoring systems are often busiest during incidents.

Incident load includes:

    More engineers
    More dashboard queries
    More logs
    More alerts

Design for:

    Normal
    Peak
    Incident

---

# 64. Mistake — No Capacity Headroom

Running at:

    95% utilization

leaves little room for:

    Traffic spikes
    Recovery
    Deployment
    Query storms

Maintain appropriate headroom.

---

# 65. Mistake — No Resource Requests/Limits in Kubernetes

Observability pods without resource planning can:

    Consume too much
    Get evicted
    Compete with applications

Set appropriate:

    Requests
    Limits

based on measured usage.

---

# 66. Mistake — Arbitrary Kubernetes Limits

Setting:

    memory: 256Mi

because it "looks reasonable" can cause:

    OOMKilled

Similarly excessive requests waste nodes.

Use:

    Historical usage
    Peak usage
    JVM/application characteristics

---

# 67. Mistake — Running Observability on Unstable Nodes

If Prometheus and applications share a heavily loaded node:

    Application load spike
       |
       v
    Node pressure
       |
       v
    Prometheus impacted

For larger environments consider dedicated observability capacity.

---

# 68. Mistake — No Pod Anti-Affinity for Critical Replicas

If two critical monitoring replicas run on the same node:

    Node failure
       |
       v
    Both replicas lost

Use appropriate scheduling constraints for HA components.

---

# 69. Mistake — Single Point of Failure

Examples:

    One Prometheus
    One Logstash
    One Elasticsearch node
    One Grafana

Ask:

    What happens if this component fails?

Architecture should follow business criticality.

---

# 70. Mistake — Assuming More Replicas Always Means HA

Three replicas on:

    One node

is still a major failure-domain risk.

HA requires:

    Failure-domain awareness
    Storage considerations
    Correct routing
    Correct state management

---

# 71. Mistake — Ignoring Availability Zones

In EKS/AWS production:

    AZ placement matters.

Spread critical workloads where appropriate.

But remember:

    More distribution can increase network cost.

Balance:

    Reliability
    Performance
    Cost

---

# 72. Mistake — Ignoring Cross-AZ Telemetry Cost

Architecture:

    EKS node AZ-A
       |
       v
    Collector AZ-B
       |
       v
    Elasticsearch AZ-C

Large log volume can create:

    Network cost
    Latency

Review high-volume paths.

---

# 73. Mistake — Ignoring NAT Gateway Traffic

Private workloads sending large telemetry streams through NAT can create unnecessary processing/transfer cost.

Review:

    Routing
    Endpoints
    Internal destinations
    Architecture

---

# 74. Mistake — No Data Lifecycle

Data should move through:

    Active
       |
       v
    Older
       |
       v
    Archive
       |
       v
    Delete

Without lifecycle:

    Storage becomes the permanent system of record for everything.

---

# 75. Mistake — Keeping All Logs in Hot Storage

Old logs rarely require the same query speed as recent logs.

Use:

    Hot
    Warm
    Cold
    Archive

based on access patterns.

---

# 76. Mistake — No Backup/Restore Testing

Having:

    Snapshot configured

does not prove recovery works.

Test:

    Backup
    Restore
    Data integrity
    Recovery time

---

# 77. Mistake — No Disaster Recovery Plan

Ask:

    What happens if Prometheus storage is lost?
    What happens if Elasticsearch cluster is lost?
    What happens if an entire AZ fails?
    What happens if a region fails?

Define:

    RTO
    RPO
    Recovery procedure

---

# 78. Mistake — Treating Development Like Production

Development usually does not need:

    Long retention
    Multiple replicas
    Large Elasticsearch clusters
    High-resolution metrics

Use environment-specific sizing.

---

# 79. Mistake — Leaving Temporary Environments Running

Temporary EKS/ELK/Prometheus resources can become long-term cost.

Use:

    Automated cleanup
    Expiration tags
    Infrastructure lifecycle

---

# 80. Mistake — No Resource Ownership

Every:

    Dashboard
    Alert
    Data source
    Index
    Collector

should have an owner.

Without ownership:

    Resources become abandoned.

---

# 81. Mistake — No Naming Standards

Inconsistent names create:

    Confusing dashboards
    Difficult queries
    Hard automation
    Poor ownership

Standardize:

    Metric names
    Labels
    Index names
    Dashboard names
    Alert names

---

# 82. Mistake — Poor Label Naming

Avoid mixing:

    env
    environment
    ENV

Use consistent conventions.

Example:

    environment
    service
    namespace
    cluster

Consistency improves query and automation reliability.

---

# 83. Mistake — Mixing Environments Without Clear Labels

If:

    prod
    staging
    dev

share a data source without clear labels, engineers may investigate the wrong environment.

Always make environment context obvious.

---

# 84. Mistake — No Cluster Label in Multi-Cluster Monitoring

If multiple EKS clusters feed central monitoring, include a bounded cluster identifier.

Example:

    cluster="prod-east"

This enables:

    Filtering
    Aggregation
    Ownership

---

# 85. Mistake — Unbounded Kubernetes Labels in Metrics

Kubernetes metadata can create excessive dimensions.

Review:

    Pod labels
    Deployment labels
    Custom application labels

Only expose labels needed for operational decisions.

---

# 86. Mistake — Scraping Every Kubernetes Endpoint

Service discovery can discover many endpoints.

Without filtering:

    Target count ↑
    Scrape load ↑
    Storage ↑

Use relabeling and explicit discovery rules.

---

# 87. Mistake — Ignoring Pod Churn

Highly dynamic environments can produce:

    Frequent target changes
    Metric churn
    Service discovery load

Monitor:

    Pod creation/deletion
    Target count
    Series churn

---

# 88. Mistake — Logging Kubernetes Metadata Blindly

Adding every:

    Label
    Annotation
    Namespace
    Pod field

to every log can create huge events.

Select useful metadata.

---

# 89. Mistake — No Correlation Between Metrics and Logs

When metrics say:

    Error rate ↑

engineers should quickly identify:

    Which service?
    Which endpoint?
    Which instance?
    Which error?

Use consistent:

    service
    environment
    cluster
    namespace
    request/correlation ID

where appropriate.

---

# 90. Mistake — No Correlation IDs

For distributed applications, without correlation IDs:

    Request
       |
       v
    Service A
       |
       v
    Service B
       |
       v
    Service C

is difficult to reconstruct from logs.

Use correlation/request IDs in logs where appropriate.

Do not put them into high-cardinality Prometheus labels.

---

# 91. Mistake — Confusing Logs With Traces

Logs show:

    Events

Traces show:

    Request path and timing across services

Metrics show:

    Aggregated behavior

Use each for its strength.

---

# 92. Mistake — Assuming Tracing Fixes Everything

Tracing is useful for:

    Distributed latency
    Dependency relationships
    Request paths

But tracing does not replace:

    Metrics
    Logs
    SLOs

Use a complete observability strategy.

---

# 93. Mistake — Sampling All Traces Equally

High trace volume can be expensive.

Use intelligent sampling where appropriate:

    Keep errors
    Keep slow requests
    Sample normal traffic

Exact policy depends on operational requirements.

---

# 94. Mistake — No Application-Level Metrics

Infrastructure metrics alone may show:

    CPU
    Memory

but not:

    Request rate
    Error rate
    Latency
    Business-critical operations

Instrument applications with useful RED-style metrics.

---

# 95. Mistake — Monitoring Infrastructure but Not User Impact

Example:

    EC2 CPU = 40%
    Memory = 50%

but:

    Checkout errors = 20%

Infrastructure looks healthy while users are failing.

Monitor:

    Availability
    Error rate
    Latency
    Saturation

---

# 96. Mistake — Monitoring Only CPU and Memory

CPU and memory are important but incomplete.

Include:

    Request rate
    Error rate
    Latency
    Saturation
    Queue depth
    Disk
    Network
    Dependencies

---

# 97. Mistake — No Dependency Monitoring

Application health depends on:

    Database
    Cache
    Message queue
    External APIs

A service may be healthy internally but failing because a dependency is unavailable.

Monitor critical dependencies.

---

# 98. Mistake — No Database Observability

Monitor:

    Connections
    Query latency
    Errors
    Locks
    CPU
    Memory
    Storage
    Replication
    Connection pool

Database issues often appear first as application latency/errors.

---

# 99. Mistake — No Queue Monitoring

For RabbitMQ or other queues monitor:

    Queue depth
    Consumer count
    Publish rate
    Consume rate
    Message age
    Errors

A growing queue is often an early saturation signal.

---

# 100. Mistake — No Network Observability

Application failures may come from:

    DNS
    Security groups
    Network ACLs
    Routing
    Load balancer
    Connection exhaustion

Monitor:

    Latency
    Errors
    Connections
    Throughput

---

# 101. Mistake — Ignoring DNS

A service can appear healthy but clients fail because:

    DNS resolution fails
    DNS latency increases
    Wrong record

Include DNS in troubleshooting.

---

# 102. Mistake — Ignoring Load Balancer Metrics

For AWS ALB, monitor:

    Request count
    Target response time
    HTTP errors
    Target health

A Kubernetes application can be healthy while ALB routing is failing.

---

# 103. Mistake — No Blackbox Monitoring

Internal metrics can say:

    Application healthy

while users cannot reach:

    DNS
    Load balancer
    Public endpoint

Blackbox checks test actual reachability.

---

# 104. Mistake — No Synthetic Monitoring

Critical user journeys may need:

    Login
    Search
    Checkout
    API request

Synthetic checks can detect failures before users report them.

---

# 105. Mistake — Alerting Without Maintenance Windows

Planned maintenance can generate expected alerts.

Without suppression:

    Alert flood

Use:

    Maintenance windows
    Silences
    Planned change handling

---

# 106. Mistake — No Alert Grouping

If one node failure causes:

    500 pod alerts

without grouping, responders receive unnecessary noise.

Group related alerts.

---

# 107. Mistake — No Inhibition

If:

    Database is down

then:

    50 applications fail

Inhibit lower-level dependent alerts when the root cause is already known.

---

# 108. Mistake — Wrong Alert Severity

If everything is:

    Critical

then nothing is truly critical.

Define:

    Critical
    Warning
    Info

based on action and user impact.

---

# 109. Mistake — Thresholds Without Context

Example:

    CPU > 80%

may be normal.

Better:

    CPU saturation + latency increase + sustained duration

Thresholds should reflect real operational conditions.

---

# 110. Mistake — Alerts That Fire Too Quickly

A short spike may not require an incident.

Use:

    for:
    sustained condition

when appropriate.

This reduces noise.

---

# 111. Mistake — Alerts That Fire Too Slowly

The opposite problem:

    30-minute evaluation

for a critical failure.

Balance:

    Detection speed
    Noise
    Business impact

---

# 112. Mistake — No Recovery Notification

An incident workflow should communicate:

    Fired
    Recovered

where appropriate.

Recovery signals help close incidents correctly.

---

# 113. Mistake — No Post-Incident Review

After major incidents ask:

    What happened?
    Why did monitoring miss it?
    Which alert was noisy?
    Which signal was missing?
    What should change?

Use incidents to improve observability.

---

# 114. Mistake — Fixing the Dashboard Instead of the Problem

Example:

    Dashboard slow

Engineer immediately increases:

    Grafana CPU

But the actual problem is:

    Expensive PromQL

Always identify the bottleneck first.

---

# 115. Mistake — Scaling Before Removing Waste

Example:

    Elasticsearch disk fills

Immediate response:

    Add another node

But logs contain:

    Duplicate DEBUG events
    No retention policy

Fix the source of waste before permanently scaling.

---

# 116. Mistake — No Recent-Change Correlation

Performance problems often follow:

    Deployment
    Configuration change
    New exporter
    New dashboard
    New alert

Always ask:

> What changed?

---

# 117. Mistake — No Baseline Before Changes

Before major observability changes capture:

    CPU
    Memory
    Ingestion
    Query latency
    Disk
    Cost

Then compare:

    Before
    vs
    After

---

# 118. Mistake — Testing Only Normal Conditions

Test:

    Normal
    Peak
    Incident
    Backend failure
    Network failure
    Storage pressure

Observability must survive realistic failure conditions.

---

# 119. Mistake — No Failure Testing

Ask:

    What if Prometheus fails?
    What if Elasticsearch fails?
    What if Logstash fails?
    What if storage becomes unavailable?
    What if network connectivity is lost?

Test recovery paths.

---

# 120. Mistake — No Monitoring Runbooks

A dashboard tells you:

    What is happening.

A runbook tells you:

    What to do.

Critical alerts should link to operational guidance.

---

# 121. Mistake — Runbooks With No Commands

A useful runbook should include examples such as:

    kubectl get pods
    kubectl describe pod
    kubectl logs
    kubectl top
    df -h
    du -sh
    systemctl status
    journalctl

Use commands appropriate to the actual incident.

---

# 122. Mistake — No Standard Troubleshooting Flow

Without a standard process engineers jump between tools.

Use:

    Detect
       |
       v
    Scope
       |
       v
    Correlate
       |
       v
    Investigate
       |
       v
    Mitigate
       |
       v
    Recover
       |
       v
    Prevent

---

# 123. Mistake — Troubleshooting Only From One Tool

Example:

    Prometheus says CPU high

Do not stop there.

Correlate with:

    Logs
    Kubernetes
    Application metrics
    Database
    Network
    Recent deployment

Observability is about correlation.

---

# 124. Mistake — Ignoring Time Correlation

If:

    Deployment at 10:05

and:

    Error rate increased at 10:06

that relationship matters.

Always correlate incidents against:

    Deployments
    Configuration changes
    Scaling
    Infrastructure changes

---

# 125. Mistake — No Change Tracking

Monitoring should ideally correlate:

    Git commit
    Deployment
    Infrastructure change
    Alert

This reduces mean time to diagnosis.

---

# 126. Mistake — No Ownership in Dashboards

A dashboard without ownership becomes stale.

Include:

    Team
    Service
    Environment
    Purpose

---

# 127. Mistake — No Dashboard Lifecycle

Dashboards accumulate.

Review:

    Usage
    Ownership
    Accuracy
    Query cost

Archive obsolete dashboards.

---

# 128. Mistake — No Alert Lifecycle

Alerts also accumulate.

Review:

    Firing history
    Ownership
    Actionability
    False positives

Delete or redesign obsolete rules.

---

# 129. Mistake — No Metric Lifecycle

Metrics can remain forever after:

    Feature removal
    Service removal
    Migration

Remove obsolete instrumentation.

---

# 130. Mistake — No Index Lifecycle

Indexes can become:

    Old
    Unused
    Expensive

Apply:

    Rollover
    Retention
    Deletion

---

# 131. Mistake — No Environment Isolation

Mixing:

    Dev
    Stage
    Prod

without clear boundaries can cause:

    Wrong investigation
    Wrong alert
    Data leakage
    Accidental changes

Separate or clearly isolate environments.

---

# 132. Mistake — Weak Access Control

Monitoring systems may contain:

    Application logs
    Security information
    User identifiers
    Infrastructure details

Use:

    RBAC
    Authentication
    Least privilege
    Network restrictions

---

# 133. Mistake — Exposing Grafana Publicly Without Protection

Bad:

    Internet
       |
       v
    Grafana
       |
       v
    Internal metrics/logs

Use:

    Authentication
    HTTPS
    Network controls
    RBAC

---

# 134. Mistake — Storing Secrets in Dashboards

Do not put:

    Passwords
    Tokens
    API keys

into:

    Dashboard variables
    Panel queries
    Documentation
    Logs

Use secure secret management.

---

# 135. Mistake — Logging Secrets

A secret in a log can propagate into:

    Logstash
    Elasticsearch
    Backups
    Archives
    Downloads

Secret removal must happen at the source whenever possible.

---

# 136. Mistake — No TLS

Telemetry often crosses:

    Nodes
    AZs
    Networks
    Users

Use encryption where required:

    HTTPS
    TLS
    Secure transport

---

# 137. Mistake — Overprivileged Service Accounts

Prometheus, collectors and applications should not automatically receive broad Kubernetes permissions.

Use:

    Minimal RBAC
    Dedicated service accounts
    Explicit permissions

---

# 138. Mistake — Ignoring Elasticsearch Security

Elasticsearch may contain:

    Production logs
    User information
    Security events

Protect:

    Authentication
    Authorization
    TLS
    Network access

---

# 139. Mistake — No Audit Trail

For sensitive monitoring platforms track:

    Who changed dashboards?
    Who changed alerts?
    Who changed retention?
    Who accessed sensitive logs?

Use audit mechanisms where required.

---

# 140. Mistake — Ignoring Cost

Observability cost can grow silently.

Track:

    Log GB/day
    Active series
    Storage
    Query volume
    Infrastructure
    Network

---

# 141. Mistake — No Cost Ownership

If nobody owns cost:

    Everyone sends logs
    Everyone creates metrics
    Everyone keeps data forever

Create:

    Ownership
    Budgets
    Usage visibility
    Review process

---

# 142. Mistake — Optimizing Cost by Removing Critical Data

Bad:

    "Delete all logs after one day."

This may violate:

    Incident needs
    Security
    Compliance

Optimize intelligently.

---

# 143. Mistake — No Data Classification

Not all telemetry has the same value.

Classify:

    Critical security
    Audit
    Application errors
    Operational metrics
    Debug data

Then apply appropriate:

    Retention
    Access
    Storage

---

# 144. Mistake — No Capacity Forecast

Current capacity may look fine.

But if:

    Log volume grows 20% monthly

storage can become a problem quickly.

Forecast:

    30 days
    90 days
    6 months
    1 year

where appropriate.

---

# 145. Mistake — Ignoring Incident Load

Normal dashboard usage may be:

    5 engineers

Incident usage may be:

    30 engineers

Design for the higher query load.

---

# 146. Mistake — No Observability Ownership Model

Define:

    Platform team:
    Monitoring infrastructure

    Application team:
    Application metrics/logs

    Security:
    Security/audit telemetry

    On-call:
    Incident response

Clear ownership prevents gaps.

---

# 147. Mistake — Tool-Centric Thinking

Bad:

> "We have Prometheus, so monitoring is solved."

Tools do not create observability automatically.

You need:

    Instrumentation
    Architecture
    Standards
    Alerts
    SLOs
    Runbooks
    Ownership
    Incident process

---

# 148. Mistake — Copying Production Architecture Everywhere

A small development environment may not need:

    Large Elasticsearch cluster
    Multiple Prometheus replicas
    Long retention
    Complex HA

Architecture should match:

    Scale
    Criticality
    Environment

---

# 149. Mistake — No Documentation

Without documentation engineers may not know:

    Where metrics are
    Where logs are
    How alerts route
    Who owns systems
    How to recover components

Maintain:

    Architecture diagrams
    Runbooks
    Configuration documentation
    Ownership information

---

# 150. Mistake — No Version Control for Observability

Monitoring configuration should ideally be managed through:

    Git
    CI/CD
    Code review

Examples:

    Prometheus rules
    Grafana provisioning
    Logstash configuration
    Kubernetes manifests

This provides:

    History
    Review
    Rollback

---

# 151. Mistake — Manual Production Changes

Manual changes create:

    Drift
    Unknown configuration
    Difficult rollback

Prefer:

    GitOps
    Infrastructure as Code
    Configuration as Code

where appropriate.

---

# 152. Mistake — No Validation Before Deployment

A bad PromQL expression or malformed alert rule can break monitoring.

Use:

    Validation
    Testing
    CI
    Staging

before production rollout.

---

# 153. Mistake — No Rollback Plan

If a new monitoring configuration causes:

    CPU spike
    Alert storm
    Log explosion

you need:

    Fast rollback

Version-controlled configuration makes this easier.

---

# 154. Mistake — No Canary for Monitoring Changes

Large changes can be introduced gradually.

Example:

    Test
       |
       v
    Small scope
       |
       v
    Measure
       |
       v
    Expand

Useful for:

    New exporters
    New log pipelines
    Large alert-rule changes

---

# 155. Mistake — Ignoring Upgrade Compatibility

Upgrading:

    Prometheus
    Grafana
    Elasticsearch
    Logstash

can affect:

    APIs
    Plugins
    Queries
    Dashboards
    Storage
    Configurations

Test upgrades.

---

# 156. Mistake — No Capacity Test After Upgrade

A version upgrade can change:

    Resource consumption
    Query behavior
    Storage behavior

Compare before/after metrics.

---

# 157. Mistake — Ignoring Plugin Dependencies

Grafana plugins and integrations can affect:

    Security
    Performance
    Compatibility

Keep only required plugins and review them during upgrades.

---

# 158. Mistake — No Central Ownership of Standards

Without standards:

    Every team names metrics differently
    Every dashboard looks different
    Alerts have inconsistent severity
    Logs use different fields

Create platform standards.

---

# 159. Mistake — No Golden Signals

For applications, start with:

    Latency
    Traffic
    Errors
    Saturation

Then add service-specific signals.

---

# 160. Mistake — No RED Metrics

For request-driven services:

    Rate
    Errors
    Duration

provide a strong baseline.

---

# 161. Mistake — No USE Metrics

For infrastructure resources consider:

    Utilization
    Saturation
    Errors

This complements application-level monitoring.

---

# 162. Mistake — Monitoring Without Business Context

Technical metrics alone may not explain impact.

Example:

    Payment API error rate

should be correlated with:

    Failed transactions

Business-critical metrics help prioritize incidents.

---

# 163. Mistake — No Synthetic User Journey

A service can expose:

    HTTP 200

while the real workflow is broken.

Test critical journeys.

---

# 164. Mistake — No Blackbox Monitoring

Internal monitoring may miss:

    DNS
    Load balancer
    Network path
    External accessibility

Blackbox probes provide an external perspective.

---

# 165. Mistake — Ignoring Dependency Failures

A service may show:

    CPU normal
    Memory normal

but:

    Database unavailable

Monitor dependency health.

---

# 166. Mistake — No Correlation Between Deployment and Incident

If every deployment is not visible in observability:

    Engineers spend more time guessing.

Expose:

    Deployment markers
    Version
    Commit
    Environment

where practical.

---

# 167. Mistake — No Incident Timeline

During postmortem, establish:

    10:00 deployment
    10:03 latency increased
    10:05 errors increased
    10:07 alert fired
    10:12 rollback
    10:15 recovery

A clear timeline helps identify:

    Detection gaps
    Alert delays
    Root cause

---

# 168. Mistake — Measuring Only Availability

A system can be:

    Available

but:

    Slow
    Error-prone
    Stale
    Degraded

Measure:

    Availability
    Latency
    Error rate
    Freshness
    Saturation

---

# 169. Mistake — No Monitoring of Alert Latency

An alert can be correct but arrive late.

Measure:

    Condition detected
       |
       v
    Rule evaluation
       |
       v
    Notification

Alert latency is a reliability metric.

---

# 170. Mistake — No Monitoring of Log Freshness

Logs can exist but arrive late.

Measure:

    Event timestamp
    Ingestion timestamp
    Searchable timestamp

Track ingestion delay.

---

# 171. Mistake — No Monitoring of Metric Freshness

Prometheus may be running while a target has stopped updating.

Check:

    Scrape success
    Last sample
    Target health

---

# 172. Mistake — No Monitoring of Data Loss

Monitor:

    Dropped logs
    Dropped metrics
    Failed scrapes
    Queue overflow
    Backend rejection

Observability quality includes data completeness.

---

# 173. Mistake — No Production Validation

After deploying monitoring changes:

    Verify dashboards
    Verify alerts
    Verify logs
    Verify ingestion
    Verify queries
    Verify permissions

Do not assume deployment success means observability success.

---

# 174. Mistake — No Security Validation

After changes verify:

    Authentication
    RBAC
    TLS
    Network access
    Secrets

Monitoring systems contain sensitive production information.

---

# 175. Mistake — No Performance Validation

After changes measure:

    CPU
    Memory
    Disk
    Query latency
    Ingestion
    Queue

Compare with baseline.

---

# 176. Mistake — No Cost Validation

After onboarding a new service measure:

    Metric growth
    Log growth
    Storage growth
    Query growth
    Infrastructure impact

A service can be functionally correct but financially inefficient.

---

# 177. Mistake — No End-to-End Test

A monitoring pipeline should be tested from:

    Application
       |
       v
    Collector
       |
       v
    Backend
       |
       v
    Dashboard/Alert

This catches failures hidden by component-level health checks.

---

# 178. Common Production Anti-Pattern

Bad architecture:

    Everything
       |
       v
    One Prometheus
       |
       v
    One Elasticsearch
       |
       v
    One Grafana
       |
       v
    Long retention
       |
       v
    No capacity planning

Problems:

    Single points of failure
    High cardinality
    Storage growth
    Query bottlenecks
    Alert delays
    High cost

---

# 179. Better Production Pattern

Use:

    Well-designed instrumentation
          |
          v
    Controlled collection
          |
          v
    Filtered telemetry
          |
          v
    Scalable backends
          |
          v
    Efficient dashboards
          |
          v
    Actionable alerts
          |
          v
    Lifecycle management
          |
          v
    Continuous review

---

# 180. Production Review Checklist

## Metrics

    [ ] Cardinality controlled
    [ ] Labels bounded
    [ ] Scrape intervals justified
    [ ] Unused metrics removed
    [ ] Queries optimized
    [ ] Recording rules reviewed

## Logs

    [ ] Structured
    [ ] Correct log levels
    [ ] Secrets removed
    [ ] Volume controlled
    [ ] Retention configured
    [ ] ILM configured
    [ ] Parsing optimized

## Dashboards

    [ ] Focused
    [ ] Fast
    [ ] Owned
    [ ] Version controlled
    [ ] Useful during incidents

## Alerts

    [ ] Actionable
    [ ] Owned
    [ ] Severity defined
    [ ] Grouped
    [ ] Inhibited where appropriate
    [ ] Tested
    [ ] Runbook linked

## Kubernetes/EKS

    [ ] Requests/limits
    [ ] HA scheduling
    [ ] Resource isolation where required
    [ ] AZ placement
    [ ] API load controlled

## Security

    [ ] Authentication
    [ ] RBAC
    [ ] TLS
    [ ] Secrets protected
    [ ] Audit where required

## Cost

    [ ] Retention reviewed
    [ ] Storage reviewed
    [ ] Log volume reviewed
    [ ] Cardinality reviewed
    [ ] Non-prod resources cleaned
    [ ] Network cost reviewed

---

# 181. Production Troubleshooting Mental Model

When observability fails:

    1. Is the component running?
    2. Is it receiving data?
    3. Is it processing data?
    4. Is it storing data?
    5. Is the data queryable?
    6. Is it fresh?
    7. Are alerts evaluating?
    8. Are notifications delivered?
    9. Did a recent change cause the issue?
    10. Is the observability platform itself overloaded?

This avoids assuming:

> "The dashboard is broken."

---

# 182. Incident Example — Prometheus Suddenly Uses High Memory

Symptoms:

    Memory ↑
    CPU ↑
    Queries slow

Investigation:

    Check active series
    Check cardinality
    Check new deployment
    Check exporters
    Check pod churn
    Check recording rules

Likely mistake:

    New unbounded label

Fix:

    Remove label
    Reduce series
    Validate memory recovery

Prevention:

    Cardinality review

---

# 183. Incident Example — Elasticsearch Disk Fills

Symptoms:

    Disk ↑ rapidly
    Indexing errors

Investigation:

    Log volume
    DEBUG level
    Index growth
    Retention
    ILM
    Shards

Possible mistake:

    No lifecycle policy

Fix:

    Apply retention
    Move older data
    Remove eligible data
    Add capacity if necessary

Prevention:

    ILM + disk alerts

---

# 184. Incident Example — Alert Storm

Symptoms:

    Hundreds of alerts

Investigation:

    Identify root cause
    Review grouping
    Review inhibition
    Review dependencies

Possible mistake:

    Alerting on every downstream symptom

Fix:

    Group/inhibit
    Improve root-cause alerting

---

# 185. Incident Example — Dashboard Slow During Incident

Symptoms:

    Normal:
    2 seconds

    Incident:
    30 seconds

Investigation:

    More users
    More refreshes
    More queries
    Expensive panels

Possible mistake:

    Dashboard designed only for normal usage

Fix:

    Optimize panels
    Recording rules
    Lower refresh
    Focused incident dashboard

---

# 186. Incident Example — Logs Missing

Trace:

    Application
       |
       v
    stdout
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

Check:

    Each stage
    Queue
    Network
    Backend rejection
    Storage

Do not troubleshoot only Kibana.

---

# 187. Incident Example — Application Latency Increases After Logging Change

Symptoms:

    Deployment
       |
       v
    Logging volume ↑
       |
       v
    Application CPU ↑
       |
       v
    Request latency ↑

Possible mistake:

    Synchronous or excessive logging

Fix:

    Reduce volume
    Optimize serialization
    Use appropriate asynchronous collection
    Validate overhead

---

# 188. Interview — What Are Common Prometheus Mistakes?

Strong answer:

> The biggest mistakes I watch for are high-cardinality labels, excessive scrape frequency, scraping unnecessary targets, expensive PromQL, poor recording-rule design and insufficient storage planning. I also monitor Prometheus itself for memory, CPU, ingestion, query latency and series growth.

---

# 189. Interview — What Are Common ELK Mistakes?

Strong answer:

> Common mistakes include uncontrolled log volume, DEBUG logging in production, excessive Grok processing, poor Elasticsearch mappings, too many small shards, no lifecycle policy, excessive replicas and insufficient disk/JVM monitoring. I focus on both data design and infrastructure capacity.

---

# 190. Interview — What Is the Most Dangerous Observability Mistake?

Strong answer:

> Treating the monitoring platform as an afterthought. If observability is overloaded, misconfigured or stale during an incident, engineers lose the information needed to diagnose production. I therefore treat observability as a production service with its own reliability, performance, security and cost requirements.

---

# 191. Interview — How Do You Avoid Alert Fatigue?

Strong answer:

> I design alerts around actionable conditions and user impact, use severity appropriately, group related alerts, inhibit dependent symptoms, tune thresholds and durations, assign ownership and link runbooks. I also review alert history regularly and remove alerts that are not actionable.

---

# 192. Interview — How Do You Prevent Monitoring From Becoming Expensive?

Strong answer:

> I control metric cardinality, scrape frequency and unused metrics; control log volume and retention; optimize queries and dashboards; use lifecycle policies; right-size resources; review replicas and network paths; and continuously measure telemetry growth. I protect critical observability data while removing waste.

---

# 193. Interview — How Do You Troubleshoot a Slow Dashboard?

Strong answer:

> I first identify whether the delay is in the browser, Grafana, the query, the data source or storage. Then I inspect panel count, time range, refresh frequency and query complexity. For Prometheus I review PromQL and series count; for Elasticsearch I review search latency, shards and JVM. I optimize before scaling.

---

# 194. Interview — How Do You Handle High-Cardinality Metrics?

Strong answer:

> I identify the label responsible for the growth and determine whether it is bounded. If it contains values such as request ID or user ID, I remove it from the metric and use logs for detailed context. I can also drop unnecessary metrics and establish cardinality checks in code review.

---

# 195. Interview — Why Should Observability Have SLOs?

Strong answer:

> Because observability is part of the production reliability system. If metrics are stale, logs arrive late or alerts are delayed, engineers may not detect or diagnose incidents in time. I would define objectives around metrics freshness, alert delivery, log ingestion and dashboard availability.

---

# 196. Interview — How Do You Design Monitoring for EKS?

Strong answer:

> I monitor nodes, pods, workloads, Kubernetes components, application RED metrics and critical dependencies. I control target discovery and cardinality, size Prometheus appropriately, manage logs through a centralized pipeline, isolate monitoring workloads when needed, spread critical components across failure domains and monitor the observability platform itself.

---

# 197. Interview — What Would You Check If Prometheus Is OOMKilled?

Strong answer:

> I would check active series and cardinality first, then ingestion rate, recent instrumentation changes, exporter additions, pod churn, recording rules and expensive queries. I would also review resource requests and limits. I would not immediately increase memory without identifying the workload causing the growth.

---

# 198. Interview — What Would You Check If Elasticsearch Is Slow?

Strong answer:

> I would check JVM heap and GC, CPU, disk I/O, indexing and search latency, rejected requests, shard count, index growth, mappings and log volume. I would determine whether the problem is ingestion, search, storage or resource pressure before changing the cluster size.

---

# 199. Interview — What Would You Check If Logs Are Delayed?

Strong answer:

> I would trace the pipeline from application stdout through the collector, Logstash and Elasticsearch. I would check queue depth, events in/out, network connectivity, backend indexing latency, disk and rejection metrics. This identifies whether the delay is at collection, processing or storage.

---

# 200. Interview — What Would You Check If Alerts Are Delayed?

Strong answer:

> I would trace the alert path from scraping through Prometheus rule evaluation, Alertmanager routing and notification delivery. I would check scrape freshness, Prometheus CPU and query load, rule evaluation duration, Alertmanager queues and notification failures.

---

# 201. Final Common-Mistakes Framework

When reviewing an observability system, ask:

    DESIGN
      |
      +---- Is the architecture appropriate?
      |
    DATA
      |
      +---- Are we collecting the right telemetry?
      |
    PERFORMANCE
      |
      +---- Is the platform efficient?
      |
    RELIABILITY
      |
      +---- Can it survive failure and incidents?
      |
    SECURITY
      |
      +---- Is sensitive data protected?
      |
    COST
      |
      +---- Are we paying for useful telemetry?
      |
    OPERATIONS
      |
      +---- Are alerts actionable?
      |
    GOVERNANCE
      |
      +---- Is everything owned and version controlled?

---

# 202. Final Principles

> Do not collect everything simply because you can.

> Do not put unbounded identifiers into metric labels.

> Do not use logs as a replacement for metrics.

> Do not build dashboards without a purpose.

> Do not create alerts without an action.

> Do not let Elasticsearch grow without lifecycle management.

> Do not allow queues to hide permanent throughput problems.

> Do not scale before understanding the workload.

> Do not expose monitoring systems without security controls.

> Do not treat observability as separate from production reliability.

> Do not optimize cost by deleting critical operational or security data.

> Do not assume a green dashboard means the data is fresh.

> Do not assume a running component means the pipeline is healthy.

> Always correlate metrics, logs, Kubernetes state and recent changes.

> Always monitor the monitoring platform.

---

# 203. Final Production Mental Model

A mature observability platform follows:

    DEFINE
      |
      v
    INSTRUMENT
      |
      v
    COLLECT
      |
      v
    FILTER
      |
      v
    STORE
      |
      v
    VISUALIZE
      |
      v
    ALERT
      |
      v
    RESPOND
      |
      v
    MEASURE
      |
      v
    IMPROVE

And continuously:

    Security
       +
    Performance
       +
    Reliability
       +
    Cost
       +
    Ownership

This completes:

    19-Best-Practices/
        05-Common-Mistakes.md

Folder complete:

    19-Best-Practices/
    ├── 01-Best-Practices.md
    ├── 02-Security.md
    ├── 03-Performance.md
    ├── 04-Cost-Optimization.md
    └── 05-Common-Mistakes.md

Next folder:

    20-Interview-Preparation/
