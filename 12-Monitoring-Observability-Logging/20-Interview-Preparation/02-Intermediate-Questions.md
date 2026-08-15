# 20 - Interview Preparation
# 02 - Intermediate Questions

> Monitoring, Observability & Logging — Intermediate DevOps Interview Preparation
>
> Focus: production reasoning, Prometheus/Grafana, ELK, Kubernetes/EKS, alerting, SLI/SLO, performance, security, cost, troubleshooting and architecture.

---

# 1. How Would You Design Monitoring for a Production Microservices Platform?

## What the interviewer is testing

Whether you can move from individual tools to an end-to-end production design.

## Strong answer

> I would design observability in layers. At the infrastructure layer I would monitor nodes, compute, storage and networking. At the Kubernetes layer I would monitor pods, workloads, cluster capacity and Kubernetes health. At the application layer I would collect RED metrics, structured logs and relevant dependency metrics. Prometheus would collect metrics, Grafana would provide dashboards, and ELK would provide centralized logging. Alerts would be tied to actionable conditions and SLOs. I would also monitor the observability platform itself for capacity, freshness, ingestion and alert latency.

## Architecture

    Users
      |
      v
    ALB
      |
      v
    EKS
      |
      +----------------------+
      |                      |
      v                      v
    Applications          Kubernetes
      |                      |
      | metrics              | node/pod metrics
      v                      v
    Prometheus <-------------+
      |
      v
    Grafana
      |
      v
    Alerts / Operations

    Applications
      |
      v
    stdout/stderr
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

---

# 2. How Do You Decide What to Monitor?

Use a layered approach.

## Layer 1 — User impact

    Availability
    Latency
    Error rate

## Layer 2 — Application

    Request rate
    Error rate
    Duration
    Queue depth
    Dependency latency

## Layer 3 — Kubernetes

    Pod restarts
    OOMKilled
    CPU
    Memory
    Readiness
    Liveness
    Scheduling

## Layer 4 — Infrastructure

    CPU
    Memory
    Disk
    Network
    I/O

## Layer 5 — Dependencies

    Database
    Cache
    Queue
    External APIs

## Layer 6 — Observability platform

    Scrape health
    Ingestion
    Query latency
    Storage
    Alert latency

---

# 3. How Do You Design a Production Dashboard?

Start with user impact.

    Service Health
        |
        +-- Availability
        +-- Request Rate
        +-- Error Rate
        +-- p50
        +-- p95
        +-- p99
        +-- Saturation
        +-- Dependencies
        +-- Recent Deployments

Then drill down:

    Service
      |
      v
    Pods
      |
      v
    Logs
      |
      v
    Dependency

A dashboard should support incident investigation, not simply display every available metric.

---

# 4. How Do You Design Alerts for a Production Application?

Start with:

    What can hurt users?
    What requires human action?
    How quickly must someone respond?

Examples:

    High error rate
    High latency
    Service unavailable
    Critical dependency unavailable
    Severe disk pressure
    Node unavailable
    Observability pipeline failure

Avoid:

    Every CPU spike
    Every warning
    Every transient error

---

# 5. What Is the Difference Between a Symptom Alert and a Cause Alert?

Symptom:

    Order service errors increased

Possible cause:

    Payment database unavailable

A good alerting system should expose the important symptom while helping responders identify the likely root cause.

Use:

    Grouping
    Inhibition
    Dependency context

to prevent alert storms.

---

# 6. How Do You Prevent Alert Fatigue in Production?

Use:

    Actionable alerts
    Appropriate severity
    Grouping
    Deduplication
    Inhibition
    `for` duration
    SLO-based alerting
    Runbooks
    Ownership

Review alerts periodically.

Ask:

> If this alert fires at 3 AM, what action should the engineer take?

If there is no clear action, the alert probably needs redesign.

---

# 7. What Is the Difference Between Threshold-Based and SLO-Based Alerting?

Threshold:

    CPU > 80%

SLO-based:

    Availability is degrading
    or
    Error budget is being consumed too quickly

Threshold alerts are useful for:

    Infrastructure
    Capacity
    Hard limits

SLO-based alerts are useful for:

    User-facing services
    Reliability

---

# 8. What Is a Burn Rate?

Burn rate measures how quickly a service is consuming its error budget.

Example:

    SLO:
    99.9%

Allowed unreliability:

    0.1%

If errors occur much faster than expected, the service is burning the error budget rapidly.

A high burn rate can justify immediate action even if the monthly SLO has not yet been violated.

---

# 9. How Would You Create a Good SLO for an API?

Example:

    SLI:
    Percentage of successful valid requests

    SLO:
    99.9% success over 30 days

Then define:

    Measurement source
    Exclusions
    Error budget
    Alert strategy

For latency:

    SLI:
    Percentage of requests below 500 ms

    SLO:
    99% of requests below 500 ms

The target should reflect actual user expectations.

---

# 10. What Is High Cardinality in a Real Production Example?

Suppose:

    100 pods
    10 endpoints
    5 methods
    5 status classes
    1,000,000 user IDs

Adding `user_id` creates a huge possible series space.

Better:

    Metrics:
    service + route + method + status

    Logs:
    user_id + request_id

This separates aggregation from detailed investigation.

---

# 11. How Do You Find the Source of Prometheus Cardinality Growth?

I would investigate:

    Active series
    Series created over time
    Recent deployments
    New exporters
    New labels
    Kubernetes pod churn
    Application instrumentation

Then identify:

    Which metric?
    Which label?
    Which service?

Typical root cause:

    New unbounded label

---

# 12. What Is Metric Churn?

Metric churn occurs when time series are frequently created and removed.

Common causes:

    Short-lived pods
    Jobs
    Dynamic labels
    Ephemeral workloads

High churn can increase:

    Memory
    CPU
    Storage
    TSDB overhead

Monitor series creation and workload behavior.

---

# 13. How Do You Optimize PromQL?

First identify expensive queries.

Then review:

    Selector scope
    Time range
    Regex
    Aggregation
    Joins
    Subqueries

Prefer:

    Narrow selectors
    Appropriate time ranges
    Recording rules for expensive repeated calculations

Example:

    rate(http_requests_total[5m])

can be aggregated:

    sum by (service) (
      rate(http_requests_total[5m])
    )

---

# 14. When Would You Use Recording Rules?

Use recording rules when a calculation is:

    Expensive
    Frequently queried
    Used by multiple dashboards
    Used by alerts
    Used for SLO calculations

Benefits:

    Faster dashboard queries
    Lower repeated computation
    Consistent calculations

Do not create recording rules for every possible query.

---

# 15. How Do You Troubleshoot a Slow Grafana Dashboard?

I would determine where the latency exists:

    Browser
       |
       v
    Grafana
       |
       v
    Data source
       |
       v
    Query
       |
       v
    Storage

Then check:

    Number of panels
    Number of queries
    Refresh interval
    Time range
    PromQL
    Elasticsearch queries
    Transformations

Optimize before scaling.

---

# 16. Why Can a Grafana Dashboard Increase Prometheus Load?

Every panel can generate queries.

For example:

    30 panels
    x
    10-second refresh
    x
    multiple users

can create a large query load.

Use:

    Fewer panels
    Recording rules
    Appropriate refresh
    Appropriate time ranges
    Focused dashboards

---

# 17. How Do You Design Grafana Dashboards for Incidents?

Use progressive detail.

## Level 1

    Overall health

## Level 2

    Service health

## Level 3

    Instance/pod

## Level 4

    Logs/dependencies

This prevents responders from starting with hundreds of graphs.

---

# 18. What Is the Difference Between p50, p95 and p99?

Example:

    p50 = 100 ms
    p95 = 300 ms
    p99 = 900 ms

Interpretation:

    50% <= 100 ms
    95% <= 300 ms
    99% <= 900 ms

p95 and p99 reveal tail latency.

---

# 19. Why Is Average Latency Dangerous?

Suppose:

    990 requests = 100 ms
    10 requests = 10 seconds

The average can hide the poor experience of a subset of users.

Percentiles show the distribution more clearly.

---

# 20. How Would You Monitor an ALB in Front of EKS?

Monitor:

    Request count
    Target response time
    HTTP 4xx
    HTTP 5xx
    Target health
    Connection behavior

Correlate:

    ALB metrics
       |
       v
    Kubernetes service
       |
       v
    Pods
       |
       v
    Application logs

This helps determine whether failure is at the load balancer, service routing or application layer.

---

# 21. What Would You Check If ALB 5xx Errors Increase?

Check:

    ALB metrics
    Target health
    Kubernetes endpoints
    Pod readiness
    Application errors
    Recent deployments
    Dependency failures

Determine whether the ALB itself is returning the error or the target application is.

---

# 22. What Would You Monitor in an EKS Cluster?

## Nodes

    CPU
    Memory
    Disk
    Network
    Pressure

## Pods

    CPU
    Memory
    Restarts
    OOMKilled
    Readiness
    Liveness

## Cluster

    API server
    Scheduling
    Capacity
    Events

## Applications

    RED metrics
    Dependencies
    Logs

---

# 23. How Do You Troubleshoot a Pod With High CPU?

Start:

    kubectl top pod <pod>

Then:

    kubectl describe pod <pod>

Check:

    Requests
    Limits
    Container
    Recent deployment

Then correlate:

    Request rate
    Latency
    Application behavior

Potential causes:

    Traffic increase
    Infinite loop
    Expensive operation
    Bad deployment
    CPU limit/throttling

---

# 24. How Do You Troubleshoot a Pod With High Memory?

Check:

    kubectl top pod <pod>

Then:

    kubectl describe pod <pod>

Look for:

    OOMKilled
    Memory limits
    Restarts

Investigate:

    Traffic
    Application memory usage
    JVM heap if Java
    Recent release
    Memory leak symptoms

---

# 25. What Is CPU Throttling?

CPU throttling occurs when a container is constrained by its CPU limit.

Symptoms can include:

    Increased latency
    Reduced throughput
    Application slowdown

Do not confuse:

    High CPU utilization

with:

    CPU throttling

Investigate both.

---

# 26. What Is Memory Pressure in Kubernetes?

Memory pressure occurs when a node has insufficient available memory.

Potential effects:

    Pod eviction
    OOM events
    Scheduling problems

Monitor:

    Node memory
    Pod requests
    Pod limits
    Evictions
    OOMKilled

---

# 27. What Is Disk Pressure?

Disk pressure occurs when a node approaches filesystem or inode limits.

Potential causes:

    Container logs
    Image layers
    Temporary files
    Application data

Monitor:

    Filesystem capacity
    Inodes
    Image storage
    Log growth

---

# 28. How Would You Troubleshoot `ImagePullBackOff`?

Check:

    kubectl describe pod <pod>

Look at Events.

Common causes:

    Wrong image name
    Wrong tag
    Registry unavailable
    Authentication failure
    Image does not exist
    Network issue

For private ECR repositories also check:

    IAM
    Node/pod identity
    Registry connectivity

---

# 29. How Would You Troubleshoot Readiness Probe Failures?

Check:

    kubectl describe pod <pod>
    kubectl logs <pod>

Then validate:

    Probe path
    Probe port
    Application startup
    Network binding
    Dependencies
    Timeout
    Initial delay

Important:

> Readiness failure should generally stop traffic rather than restart the application.

---

# 30. How Would You Troubleshoot Liveness Probe Failures?

Check:

    Probe endpoint
    Timeout
    Initial delay
    Application logs
    CPU/memory
    Application deadlock

Ask:

> Is the application actually unhealthy, or is the probe badly designed?

A bad liveness probe can create a restart loop.

---

# 31. How Do You Monitor Kubernetes Control Plane Health?

For managed EKS, much of the control plane is AWS-managed.

You still monitor:

    API server availability/latency where exposed
    Kubernetes events
    Scheduling behavior
    Workload deployment failures
    API-related application symptoms

Also monitor AWS-side control-plane signals available for the EKS service.

---

# 32. How Do You Monitor Kubernetes Events?

Events can reveal:

    Scheduling failures
    Image pull errors
    Probe failures
    Evictions
    Mount errors

Command:

    kubectl get events -A --sort-by=.lastTimestamp

Events are especially useful during incident investigation.

---

# 33. How Would You Design Centralized Logging for EKS?

Architecture:

    Pods
      |
      v
    stdout/stderr
      |
      v
    Node-level collector
      |
      v
    Logstash
      |
      v
    Elasticsearch
      |
      v
    Kibana

Collector deployment:

    DaemonSet

Advantages:

    One collector per node
    Captures node-local container logs
    Works with dynamic pods

---

# 34. Why Use a DaemonSet for Log Collection?

A DaemonSet ensures a collector runs on each eligible node.

Architecture:

    Node 1 -> Collector
    Node 2 -> Collector
    Node 3 -> Collector

Each collector reads local container logs and forwards them centrally.

---

# 35. What Happens If Logstash Becomes Slow?

Pipeline:

    Collector
       |
       v
    Logstash
       |
       v
    Elasticsearch

If Logstash cannot process fast enough:

    Queue grows
       |
       v
    Backpressure
       |
       v
    Potential log delay/loss

Investigate:

    CPU
    Memory
    Pipeline configuration
    Grok processing
    Elasticsearch output

---

# 36. What Happens If Elasticsearch Becomes Slow?

Possible result:

    Logstash output slows
       |
       v
    Queue grows
       |
       v
    Log ingestion delay

Investigate Elasticsearch:

    JVM
    CPU
    Disk
    Shards
    Indexing
    Rejections

Do not automatically blame Logstash.

---

# 37. How Do You Reduce ELK Cost?

Use:

    Appropriate log levels
    Structured logs
    Filtering
    Retention policies
    ILM
    Hot/warm/cold strategy
    Right-sized nodes
    Appropriate replicas
    Efficient mappings
    Removal of duplicate logs

First identify the largest cost driver.

---

# 38. What Is Elasticsearch ILM?

Index Lifecycle Management automates index lifecycle actions such as:

    Rollover
    Hot phase
    Warm phase
    Cold phase
    Delete

It helps control:

    Storage
    Performance
    Retention
    Cost

---

# 39. Why Are Too Many Elasticsearch Shards a Problem?

Every shard has overhead.

Too many shards can cause:

    Memory pressure
    JVM overhead
    Query overhead
    Cluster-state overhead

Shard design should consider:

    Data volume
    Query pattern
    Retention
    Node capacity

---

# 40. What Is a Mapping in Elasticsearch?

A mapping defines how fields are stored and indexed.

Examples:

    keyword
    text
    date
    integer
    boolean

Correct mapping improves:

    Search
    Aggregation
    Storage
    Query behavior

---

# 41. `text` vs `keyword` in Elasticsearch

Use `text` for:

    Full-text search

Use `keyword` for:

    Exact matching
    Filtering
    Aggregation

Example:

    service="orders"

is generally an exact-value field.

---

# 42. What Is Mapping Explosion?

Mapping explosion occurs when excessive dynamic fields are created.

Causes:

    Uncontrolled JSON fields
    Dynamic user-generated fields
    Arbitrary metadata

Impact:

    Memory
    Cluster state
    Index complexity

Control mappings intentionally.

---

# 43. How Would You Troubleshoot Missing Logs?

Trace end-to-end:

    Application
       |
       v
    Container stdout
       |
       v
    Node collector
       |
       v
    Logstash
       |
       v
    Elasticsearch
       |
       v
    Kibana

At each stage ask:

    Are events entering?
    Are events leaving?
    Are errors occurring?
    Is there backpressure?

---

# 44. What Is Log Sampling?

Sampling reduces the amount of telemetry retained or processed.

For logs it may be appropriate for:

    High-volume repetitive informational events

Be careful with:

    Errors
    Security events
    Audit logs

Do not sample critical evidence blindly.

---

# 45. How Do You Prevent Sensitive Data From Entering Logs?

Prefer prevention at the application layer.

Do not log:

    Passwords
    Tokens
    Private keys
    Session secrets

Use:

    Masking
    Redaction
    Structured logging
    Code review

Also restrict access to the centralized logging system.

---

# 46. What Is a Correlation ID?

A correlation ID identifies a request or transaction across components.

Example:

    Client
      |
      | correlation_id=abc123
      v
    Order Service
      |
      | correlation_id=abc123
      v
    Payment Service
      |
      | correlation_id=abc123
      v
    Inventory Service

It makes distributed log investigation easier.

Do not use the correlation ID as an unbounded Prometheus label.

---

# 47. How Do You Correlate Metrics and Logs?

Use common bounded context:

    service
    environment
    cluster
    namespace

Then:

    Metric:
    orders error rate increased

    Log:
    orders service
    ERROR
    payment timeout
    correlation_id=abc123

Metrics detect.

Logs explain.

---

# 48. What Is the Difference Between Availability and Reliability?

Availability:

    Is the service accessible/working at a point in time?

Reliability:

    How consistently does it perform correctly over time?

SLOs help quantify reliability.

---

# 49. How Do You Define Availability?

A common model:

    Availability =
    Successful time or successful requests
    ---------------------------------------
    Total expected time or requests

For APIs, request-based availability is often more useful.

---

# 50. How Would You Monitor a Database?

Monitor:

    CPU
    Memory
    Connections
    Query latency
    Slow queries
    Locks
    Storage
    Replication
    Errors

Correlate with application:

    API latency
    Error rate
    Connection pool

---

# 51. How Would You Monitor RabbitMQ?

Monitor:

    Queue depth
    Message age
    Publish rate
    Consume rate
    Consumer count
    Unacked messages
    Errors

A queue that grows continuously indicates:

    Producer rate > Consumer capacity

---

# 52. How Do You Identify a Dependency Bottleneck?

Suppose:

    API latency ↑

Check:

    Database latency
    External API latency
    Queue delay
    Cache latency

Tracing or dependency metrics can identify the slow component.

---

# 53. How Would You Troubleshoot High Application Latency?

Use:

    1. Check traffic
    2. Check p95/p99 latency
    3. Check errors
    4. Check CPU/memory
    5. Check database
    6. Check external dependencies
    7. Check queues
    8. Check logs
    9. Check recent deployment
    10. Compare with baseline

Do not assume:

    "High latency = high CPU."

---

# 54. How Would You Troubleshoot Increased Error Rate After Deployment?

Timeline:

    Deployment
       |
       v
    Error rate increases

Check:

    Version
    Logs
    Configuration
    Environment variables
    Secrets
    Dependency compatibility
    Database migrations

If impact is severe:

    Mitigate first
       |
       v
    Roll back if appropriate
       |
       v
    Investigate root cause

---

# 55. What Is the First Priority During a Production Incident?

The first priority is:

> Restore or protect user-facing service.

Do not spend 30 minutes proving the root cause while users are actively impacted if a safe mitigation is available.

Sequence:

    Detect
      |
      v
    Scope
      |
      v
    Mitigate
      |
      v
    Investigate
      |
      v
    Recover
      |
      v
    Prevent

---

# 56. Why Is "Restart the Pod" Not Always a Good Answer?

Restarting may:

    Temporarily hide the symptom
    Destroy evidence
    Repeat the failure
    Mask a memory leak
    Increase downtime

Restart when:

    It is an appropriate mitigation
    and
    you continue investigating the root cause.

---

# 57. How Do You Monitor the Monitoring System?

For Prometheus:

    Scrape failures
    Active series
    Samples/sec
    Query latency
    Rule evaluation
    CPU
    Memory
    Disk

For Elasticsearch:

    JVM
    CPU
    Disk
    Search latency
    Indexing
    Rejections
    Cluster health

For Logstash:

    Events in/out
    Queue
    Pipeline latency
    CPU
    Memory

For Grafana:

    Availability
    Query latency
    Errors
    Resource usage

---

# 58. What Is Observability Platform Capacity Planning?

Forecast:

    Targets
    Active series
    Samples/sec
    Log GB/day
    Query volume
    Storage growth
    Users
    Dashboards

Then plan:

    CPU
    Memory
    Storage
    Network
    HA capacity

---

# 59. How Do You Decide Prometheus Retention?

Consider:

    Incident investigation requirements
    Storage capacity
    Query needs
    Compliance
    Long-term trends

Do not select retention simply because:

    "30 days is standard."

Choose based on actual requirements.

---

# 60. Local Prometheus vs Long-Term Metrics Storage

Prometheus local storage is useful for:

    Recent operational data

Long-term systems can provide:

    Extended retention
    Centralized multi-cluster data
    Larger historical analysis

Examples of architectural approaches include remote storage systems.

The key interview point:

> Separate short-term operational monitoring requirements from long-term analytical retention.

---

# 61. How Do You Monitor Multiple EKS Clusters?

Possible architecture:

    EKS Cluster A
          |
          v
    Prometheus
          |
          |
    EKS Cluster B
          |
          v
    Prometheus
          |
          v
    Central Metrics Platform
          |
          v
        Grafana

Use labels such as:

    cluster
    environment
    region

Keep labels bounded.

---

# 62. How Do You Avoid Cross-Cluster Metric Confusion?

Include explicit dimensions:

    cluster
    environment
    region

Example:

    cluster="prod-eks-01"
    environment="prod"
    region="ap-south-1"

This makes dashboards and queries safer.

---

# 63. How Would You Monitor a Multi-AZ EKS Platform?

Monitor:

    Node distribution
    Pod distribution
    ALB targets
    Network traffic
    Cross-AZ traffic
    Capacity per AZ

Ensure critical workloads are not accidentally concentrated in one failure domain.

---

# 64. What Is the Tradeoff Between HA and Cost?

More HA generally means:

    More replicas
    More storage
    More network
    More compute

But insufficient HA creates:

    Downtime
    Data loss
    Monitoring blind spots

Design HA according to:

    Business impact
    RTO
    RPO
    SLO
    Budget

---

# 65. How Do You Secure Monitoring Systems?

Use:

    Authentication
    RBAC
    TLS
    Network restrictions
    Least privilege
    Secret management
    Audit logging where required

Monitoring systems can contain sensitive production information.

---

# 66. How Do You Secure Prometheus in Kubernetes?

Consider:

    Service account
    Minimal RBAC
    Network policy
    Restricted access
    TLS where appropriate
    Secure Grafana access

Do not expose Prometheus publicly without strong controls.

---

# 67. How Do You Secure Elasticsearch?

Consider:

    Authentication
    Authorization
    TLS
    Network isolation
    Index-level permissions
    Secure credentials
    Audit capabilities where required

---

# 68. What Is Observability as Code?

Managing:

    Dashboards
    Alert rules
    Prometheus configuration
    Log pipelines
    Kubernetes manifests

through:

    Git
    CI/CD
    Code review

Benefits:

    Version history
    Review
    Rollback
    Repeatability

---

# 69. Why Use GitOps for Observability Configuration?

Example:

    Git repository
        |
        v
    Review
        |
        v
    CI validation
        |
        v
    Deployment
        |
        v
    Monitoring platform

This reduces manual configuration drift.

---

# 70. How Do You Validate Prometheus Rules Before Production?

Validate:

    YAML syntax
    PromQL
    Rule names
    Labels
    Alert expressions

Then:

    Test in non-production
    Review changes
    Deploy
    Verify evaluation
    Verify alert routing

---

# 71. What Is Configuration Drift in Monitoring?

Example:

    Git:
    Alert threshold = 5%

    Production:
    Alert threshold = 10%

Now:

    Source of truth != running configuration

This can create unpredictable incident behavior.

Use configuration as code.

---

# 72. How Do You Handle a Bad Monitoring Deployment?

If a new configuration causes:

    High Prometheus CPU
    Alert storm
    Log explosion

I would:

    1. Identify impact
    2. Stop further rollout
    3. Roll back
    4. Verify recovery
    5. Analyze cause
    6. Fix in test environment
    7. Re-deploy gradually

---

# 73. What Is a Monitoring Canary?

Instead of changing every target at once:

    Small scope
       |
       v
    Validate
       |
       v
    Measure
       |
       v
    Expand

Useful for:

    Exporters
    Scrape configuration
    Alert rules
    Log pipeline changes

---

# 74. How Do You Optimize Monitoring Cost Without Losing Visibility?

Use a priority model.

## Tier 1

    Critical SLO metrics
    Security logs
    Audit data

Keep strongly protected.

## Tier 2

    Operational metrics
    Application logs

Optimize retention and volume.

## Tier 3

    Debug telemetry

Short retention or temporary collection.

This avoids deleting valuable data blindly.

---

# 75. What Is Data Lifecycle Management?

Example:

    New data
       |
       v
    Hot
       |
       v
    Warm
       |
       v
    Cold
       |
       v
    Archive
       |
       v
    Delete

Use different storage classes based on access frequency and business requirements.

---

# 76. What Is Observability Data Freshness?

Freshness answers:

> How recently was the data generated and ingested?

For metrics:

    Last successful scrape

For logs:

    Event time vs ingestion time

For alerts:

    Condition time vs notification time

Freshness is important because stale data can create false confidence.

---

# 77. How Do You Detect Monitoring Blind Spots?

Ask:

    Which services have no metrics?
    Which services have no logs?
    Which alerts have no owners?
    Which dependencies are unmonitored?
    Which dashboards have stale data?
    Which environments are not covered?

Create an observability coverage matrix.

---

# 78. What Is an Observability Coverage Matrix?

Example:

| Service | Metrics | Logs | Alerts | SLO | Owner |
|---|---|---|---|---|---|
| Orders | Yes | Yes | Yes | Yes | Team A |
| Payment | Yes | Yes | Yes | Yes | Team B |
| Inventory | Yes | Yes | Yes | No | Team C |

This quickly reveals gaps.

---

# 79. How Do You Review Monitoring Quality?

Review:

    Alert accuracy
    Alert volume
    MTTD
    MTTR
    Data freshness
    Dashboard usage
    Metric cardinality
    Log volume
    Storage growth
    Cost

Monitoring quality should be measurable.

---

# 80. What Is MTTD?

MTTD:

    Mean Time To Detect

It measures how quickly a problem is detected.

Better monitoring should generally reduce MTTD.

---

# 81. What Is MTTR?

MTTR:

    Mean Time To Recover/Repair

It measures how quickly service is restored.

Observability can reduce MTTR by helping engineers:

    Find the affected service
    Identify root cause
    Validate mitigation

---

# 82. How Do Metrics Reduce MTTR?

Metrics quickly show:

    When the problem started
    Scope
    Severity
    Affected service

Then engineers use:

    Logs
    Traces
    Kubernetes state

for deeper investigation.

---

# 83. How Do Logs Reduce MTTR?

Logs provide:

    Error message
    Stack trace
    Request ID
    Dependency error
    Configuration issue

This can turn:

    "Service is failing"

into:

    "Payment dependency connection pool is exhausted."

---

# 84. How Does Tracing Reduce MTTR?

Tracing can reveal:

    Which service
    Which endpoint
    Which dependency
    Where latency occurred

This is especially useful for distributed systems.

---

# 85. What Is an Observability Runbook?

A runbook documents:

    Meaning of alert
    Initial checks
    Commands
    Dashboards
    Logs
    Likely causes
    Mitigation
    Escalation

Example:

    Alert:
    High API latency

    Check:
    p95 latency
    traffic
    CPU
    database
    deployment

---

# 86. What Is a Production Readiness Review for Monitoring?

Before production:

    Metrics present
    Logs present
    Alerts tested
    Dashboard created
    SLO defined
    Runbook available
    Ownership assigned
    Security reviewed
    Retention defined
    Cost reviewed

This is an important DevOps practice.

---

# 87. What Is the Difference Between Monitoring Coverage and Observability Coverage?

Monitoring coverage:

    Are important health metrics collected?

Observability coverage:

    Can we understand failures across metrics, logs, traces, dependencies and context?

A service can have CPU metrics and still have poor observability.

---

# 88. How Would You Monitor a New Microservice Before Production?

Checklist:

    1. Define service SLO
    2. Add request metrics
    3. Add error metrics
    4. Add latency metrics
    5. Add structured logs
    6. Define important dependencies
    7. Create dashboard
    8. Create actionable alerts
    9. Add runbook
    10. Test failure scenarios
    11. Review security
    12. Review cost

---

# 89. What Is a Good Production Logging Standard?

Minimum fields:

    timestamp
    level
    service
    environment
    message

Recommended where appropriate:

    version
    hostname/pod
    namespace
    request_id
    status
    duration
    error type

Avoid:

    passwords
    tokens
    secrets

---

# 90. How Do You Handle Log Volume Spikes?

First identify source:

    Which service?
    Which pod?
    Which log level?
    Which error?

Then:

    Reduce accidental volume
    Fix retry/error loop
    Filter duplicates
    Protect backend
    Scale consumers if needed

After mitigation:

    Investigate root cause

---

# 91. What Is a Retry Storm?

A dependency fails:

    Service A
       |
       v
    Service B fails
       |
       v
    A retries
       |
       v
    More requests
       |
       v
    B becomes more overloaded

This can cause:

    Error amplification
    Log storm
    CPU increase
    Latency increase

Monitoring should detect:

    Retry rate
    Error rate
    Dependency latency

---

# 92. What Is Saturation in a Database?

Examples:

    Connection pool full
    CPU saturated
    Disk I/O saturated
    Lock contention

Application symptoms:

    High latency
    Timeout
    Errors

Do not monitor only database CPU.

---

# 93. How Do You Identify a Memory Leak?

Look for:

    Memory increasing over time
    Stable traffic
    Repeated growth
    Restarts/OOMKilled
    Heap behavior

Correlate:

    Deployment
    Traffic
    Application version

For Java:

    Heap
    GC
    Old generation

---

# 94. How Do You Identify a CPU Regression After Deployment?

Compare:

    Before deployment
    vs
    After deployment

Check:

    CPU
    Request rate
    Latency
    Application version
    Endpoint distribution

If traffic is unchanged but CPU increases significantly after a release, investigate the new code/configuration.

---

# 95. How Would You Monitor a CI/CD Deployment?

Track:

    Deployment success/failure
    Duration
    Version
    Rollout progress
    Pod readiness
    Error rate
    Latency

Important:

> A deployment is successful only if the application remains healthy after rollout.

---

# 96. What Is Deployment Correlation?

Expose deployment metadata in observability.

Example:

    Version 2.5 deployed
       |
       v
    Error rate ↑
       |
       v
    Latency ↑

This provides immediate incident context.

---

# 97. What Is an Observability Anti-Pattern?

Example:

    1,000 metrics
    100 dashboards
    500 alerts
    30-day retention
    No ownership

This may look sophisticated but can be operationally poor.

Better:

    Useful telemetry
    Actionable alerts
    Focused dashboards
    Clear ownership

---

# 98. What Would You Improve First in a Noisy Monitoring Environment?

I would prioritize:

    1. Critical alert accuracy
    2. Alert ownership
    3. User-impact alerts
    4. Alert grouping/inhibition
    5. Remove noisy rules
    6. Improve dashboards
    7. Review telemetry volume
    8. Establish SLOs

Do not start by rebuilding the entire monitoring stack.

---

# 99. What Would You Do If Monitoring Is Consuming Too Many Resources?

Use an investigation sequence:

    Resource increase
       |
       v
    Identify component
       |
       v
    Identify workload
       |
       v
    Identify recent change
       |
       v
    Optimize
       |
       v
    Scale if needed

Examples:

    Prometheus:
    cardinality/query/scrape

    Elasticsearch:
    log volume/query/shards

    Grafana:
    dashboard/query load

---

# 100. How Do You Explain Observability to a Non-Technical Manager?

Strong answer:

> Monitoring tells us when something is going wrong. Observability gives the engineering team enough information to understand why it is happening and how users are affected. The goal is faster detection, faster diagnosis and faster recovery.

---

# 101. Intermediate Scenario — API Error Rate Is 10%

Situation:

    Error rate = 10%

Strong approach:

    1. Confirm affected service
    2. Check traffic
    3. Check latency
    4. Identify affected endpoint
    5. Check application logs
    6. Check dependencies
    7. Check recent deployment
    8. Mitigate if necessary

Do not immediately restart all pods.

---

# 102. Intermediate Scenario — Prometheus Memory Suddenly Doubles

Investigate:

    Active series
    New metrics
    New labels
    Pod churn
    Exporters
    Scrape rate
    Recording rules

Likely causes:

    Cardinality increase
    More targets
    Higher ingestion

---

# 103. Intermediate Scenario — Elasticsearch Storage Grows 30% in One Day

Check:

    Log ingestion rate
    Index sizes
    DEBUG logging
    Duplicate events
    Retention
    ILM
    Replica count

Find:

> Which service caused the growth?

Then fix the source.

---

# 104. Intermediate Scenario — Grafana Works but Dashboards Are Slow

Check:

    Prometheus query latency
    Elasticsearch query latency
    Panel count
    Refresh rate
    Time range
    Query complexity

Optimize before increasing Grafana resources.

---

# 105. Intermediate Scenario — Logs Arrive 15 Minutes Late

Trace:

    Application
       |
       v
    Collector
       |
       v
    Logstash
       |
       v
    Elasticsearch

Check:

    Queue depth
    CPU
    Network
    Indexing latency
    Disk
    Backend rejection

The likely problem is backpressure somewhere in the pipeline.

---

# 106. Intermediate Scenario — Alerts Are Correct but Arrive Late

Trace:

    Metric
       |
       v
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

Check latency at each stage.

---

# 107. Intermediate Scenario — Only One Pod Has High Latency

Do not immediately blame the whole service.

Compare:

    Pod CPU
    Pod memory
    Node health
    Pod version
    Traffic distribution
    Logs
    Dependencies

Possible causes:

    Bad node
    Uneven traffic
    Application issue
    Dependency connection problem

---

# 108. Intermediate Scenario — One Node Has Many OOMKilled Pods

Investigate:

    Node memory
    Pod requests
    Pod limits
    Evictions
    Workload distribution

Possible cause:

    Overcommitment

Then review:

    Scheduling
    Resource requests
    Limits
    Workload placement

---

# 109. Intermediate Scenario — Error Rate Is Normal but Latency Is High

This is an important interview scenario.

Possible causes:

    Database slowdown
    External dependency latency
    CPU saturation
    Connection pool exhaustion
    Queue delay
    Network latency

Do not assume:

    No errors = healthy.

---

# 110. Intermediate Scenario — CPU Is Normal but Application Is Slow

Possible causes:

    Database
    Network
    External API
    Lock contention
    Queue
    Connection pool
    Thread pool
    Disk I/O

This demonstrates why infrastructure metrics alone are insufficient.

---

# 111. Intermediate Scenario — Disk Is 90% Full but Application Is Healthy

Do not wait for 100%.

Investigate:

    Growth rate
    Large directories
    Logs
    Temporary files
    Retention
    Container images

Then:

    Clean safely
    Increase capacity if required
    Fix source of growth

---

# 112. Intermediate Scenario — Monitoring Works in Staging but Fails in Production

Compare:

    Network
    IAM
    RBAC
    Service discovery
    Endpoints
    DNS
    Resource limits
    Configuration
    Scale

Do not assume application behavior is the only difference.

---

# 113. Intermediate Scenario — New Exporter Causes Prometheus Load

Compare:

    Before
    vs
    After

Check:

    Metrics count
    Active series
    Scrape duration
    Scrape frequency

Possible solution:

    Drop unnecessary metrics
    Reduce labels
    Adjust scrape configuration
    Remove exporter metrics not needed

---

# 114. Intermediate Scenario — New Dashboard Causes Prometheus CPU Spike

Likely cause:

    Expensive PromQL
    +
    Frequent refresh
    +
    Many users

Fix:

    Optimize query
    Use recording rules
    Reduce refresh
    Reduce panel count

---

# 115. Intermediate Scenario — New Application Generates Huge Logs

Check:

    Log level
    Retry loops
    Exceptions
    Request payload logging
    Duplicate logging

Fix at source whenever possible.

Do not simply keep increasing Elasticsearch capacity.

---

# 116. Intermediate Scenario — Alert Fires During Every Deployment

Possible causes:

    Temporary startup state
    Readiness not accounted for
    Alert too sensitive
    Deployment changes metric behavior

Use:

    Appropriate `for`
    Deployment-aware alert design
    Correct readiness signals

Do not blindly silence production alerts permanently.

---

# 117. Intermediate Scenario — Prometheus Target Is Intermittently Down

Check:

    Scrape duration
    Timeout
    Network
    Application load
    Endpoint performance
    Pod restarts

Possible cause:

    `/metrics` endpoint is too expensive.

---

# 118. Intermediate Scenario — Log Collector Uses High CPU

Check:

    Parsing
    Grok
    Regex
    Log volume
    Metadata enrichment
    Output retries

Optimize parsing before scaling.

---

# 119. Intermediate Scenario — Elasticsearch Search Is Slow but Indexing Is Normal

Investigate:

    Search queries
    Aggregations
    Shards
    JVM
    Disk
    Large time ranges
    High-cardinality aggregations

The bottleneck may be query-side rather than ingestion-side.

---

# 120. Intermediate Scenario — Elasticsearch Indexing Is Slow

Investigate:

    Log volume
    CPU
    Disk
    JVM
    Refresh behavior
    Shards
    Replicas
    Mapping complexity

Then identify whether:

    Producer rate
    exceeds
    Cluster capacity

---

# 121. Intermediate Scenario — Error Rate Increased but No Deployment Occurred

Check:

    Traffic
    Dependency
    Database
    Network
    External API
    Infrastructure
    Certificate/DNS changes
    Configuration changes

Not every incident is caused by application deployment.

---

# 122. Intermediate Scenario — Latency Increased During Traffic Spike

Determine:

    Traffic increase
       |
       v
    Resource saturation?
       |
       +---- CPU
       +---- DB
       +---- Connections
       +---- Queue
       +---- Network

Then check whether:

    Autoscaling responded correctly.

---

# 123. Intermediate Scenario — HPA Scales But Latency Remains High

Possible causes:

    Scaling metric is wrong
    Application bottleneck is downstream
    Database is saturated
    Connection pool is limited
    Pods are CPU-bound by limits
    New pods cannot become ready quickly

Scaling the application cannot solve every bottleneck.

---

# 124. Intermediate Scenario — Pods Scale but Database Connections Exhaust

Classic scaling problem:

    More pods
       |
       v
    More DB connections
       |
       v
    DB connection limit
       |
       v
    Errors/latency

Monitor:

    Connection pool
    DB connections
    Pod count

Design scaling with dependency capacity.

---

# 125. Intermediate Scenario — Application Is Healthy but Users Cannot Access It

Check from outside inward:

    DNS
       |
       v
    ALB
       |
       v
    Target health
       |
       v
    Kubernetes service
       |
       v
    Pod
       |
       v
    Application

This is a strong blackbox troubleshooting approach.

---

# 126. Intermediate Scenario — Kubernetes Says Pod Is Ready but Requests Fail

Readiness may be insufficient.

Check:

    Application endpoint
    Service
    Target registration
    Network policy
    ALB
    Dependencies

A readiness probe can be too shallow.

---

# 127. Intermediate Scenario — Monitoring Data Disappears After Pod Restart

Ask:

    Is the data stored locally?
    Is storage persistent?
    Is remote storage configured?
    Is the retention appropriate?

Stateful monitoring components need appropriate storage design.

---

# 128. Intermediate Scenario — Prometheus Storage Fills Quickly

Check:

    Active series
    Samples/sec
    Retention
    Scrape interval
    New targets

Calculate growth:

    Daily storage growth
       |
       v
    Available capacity
       |
       v
    Days remaining

Capacity planning should be proactive.

---

# 129. Intermediate Scenario — Alertmanager Receives Alerts but No Notification

Check:

    Receiver
    Routing
    Matchers
    Inhibition
    Silences
    Notification endpoint
    Authentication
    Network

Trace:

    Alertmanager
       |
       v
    Route
       |
       v
    Receiver
       |
       v
    External notification system

---

# 130. Intermediate Scenario — Alert Fires in Prometheus but Not in Alertmanager

Check:

    Prometheus alert configuration
    Alertmanager URL
    Network connectivity
    Alertmanager health
    Prometheus logs

The failure is between:

    Prometheus
       |
       v
    Alertmanager

---

# 131. Intermediate Scenario — Alertmanager Has Duplicate Notifications

Check:

    Grouping
    Group interval
    Repeat interval
    Multiple routes
    Multiple receivers
    Deduplication labels

Fix routing rather than silencing everything.

---

# 132. Intermediate Scenario — Same Alert Fires for Every Pod

Group by appropriate dimensions.

Example:

    cluster
    namespace
    alertname

Do not group so aggressively that unrelated incidents become hidden.

---

# 133. Intermediate Scenario — One Alert Masks Another

Review:

    Inhibition rules
    Severity
    Labels
    Root-cause assumptions

Inhibition should reduce noise without hiding independent failures.

---

# 134. Intermediate Scenario — Dashboard Shows Wrong Environment

Common causes:

    Incorrect variable
    Missing environment label
    Shared data source
    Query not filtering environment

Always include explicit:

    environment
    cluster

dimensions in multi-environment systems.

---

# 135. Intermediate Scenario — Production Has Missing Metrics for One Service

Check:

    Instrumentation
    `/metrics`
    ServiceMonitor/discovery configuration
    Labels
    RBAC
    Network policy
    Scrape configuration

Do not assume Prometheus is broken globally.

---

# 136. Intermediate Scenario — New Pods Are Not Being Monitored

Check:

    Service discovery
    Labels
    Namespace selection
    Scrape configuration
    Pod annotations/monitoring resources

Dynamic Kubernetes environments depend heavily on correct discovery configuration.

---

# 137. Intermediate Scenario — Metrics Exist but Alert Query Returns Nothing

Check:

    Metric name
    Labels
    Aggregation
    Time range
    Query syntax

Run the query manually before debugging the alerting platform.

---

# 138. Intermediate Scenario — Logs Exist but Kibana Search Returns Nothing

Check:

    Correct index
    Time range
    Field mapping
    Data view
    Timestamp parsing

A query issue can look like missing data.

---

# 139. Intermediate Scenario — Log Timestamp Is Wrong

Possible causes:

    Timezone handling
    Timestamp parsing
    Collector transformation
    Application clock

Check:

    Application timestamp
    Ingestion timestamp
    Elasticsearch timestamp

Time synchronization matters for incident timelines.

---

# 140. Intermediate Scenario — Distributed Logs Cannot Be Correlated

Add consistent:

    service
    environment
    version
    request/correlation ID

Then ensure the fields survive:

    Application
       |
       v
    Collector
       |
       v
    Logstash
       |
       v
    Elasticsearch

---

# 141. Intermediate Scenario — Monitoring Cost Doubled

Do not guess.

Compare:

    Metrics volume
    Log volume
    Query volume
    Storage
    Compute
    Network

Then compare:

    Yesterday
    vs
    Today

Look for recent changes.

---

# 142. Intermediate Scenario — Cost Increased After Adding a New Service

Check:

    Metrics cardinality
    Log GB/day
    Number of pods
    Scrape frequency
    Dashboard usage
    Retention

Calculate telemetry cost per service.

---

# 143. Intermediate Scenario — Security Team Wants Longer Log Retention

Do not simply increase hot Elasticsearch storage.

Design:

    Hot
       |
       v
    Warm
       |
       v
    Cold/archive

Meet retention requirements while controlling operational cost.

---

# 144. Intermediate Scenario — Compliance Requires Audit Logs

Treat audit logs differently from normal application logs.

Consider:

    Longer retention
    Restricted access
    Integrity
    Encryption
    Audit trail
    Controlled deletion

Do not apply the same retention policy blindly to all logs.

---

# 145. Intermediate Scenario — Observability System Itself Has an Incident

Example:

    Prometheus high memory
    Elasticsearch disk full
    Logstash queue full

Treat it as a production incident.

Prioritize:

    Restore telemetry
    Protect data
    Reduce load
    Recover capacity
    Investigate root cause
    Prevent recurrence

---

# 146. Intermediate Question — How Do You Balance Reliability and Cost?

Strong answer:

> I classify telemetry by business value. Critical SLO, security and audit data receive stronger retention and availability. Operational telemetry is optimized through cardinality control, appropriate scrape intervals and retention. Debug data has shorter retention. I also review compute, storage, query and network costs continuously.

---

# 147. Intermediate Question — How Do You Handle a Noisy Alert?

Strong answer:

> I first determine whether the alert represents real user or system impact. If it does, I tune the condition using appropriate duration, aggregation and severity. If it is a symptom of another alert, I use grouping or inhibition. I also add a runbook and assign ownership. I would not simply silence a recurring alert permanently.

---

# 148. Intermediate Question — How Do You Troubleshoot a Monitoring Failure?

Strong answer:

> I trace the telemetry pipeline end-to-end. For metrics I check target availability, scraping, ingestion, storage and queries. For logs I check application output, collectors, processing queues, backend indexing and search. For alerts I trace rule evaluation through Alertmanager to notification delivery. I also check the monitoring platform's own resource utilization.

---

# 149. Intermediate Question — How Do You Approach a Production Incident?

Strong answer:

> I first establish impact and scope, then stabilize the service if there is a safe mitigation. I use metrics to identify when and where the problem started, logs for detailed evidence, Kubernetes and infrastructure data for platform context, and recent changes/dependencies for correlation. After recovery I validate the service, document the timeline and create preventive actions.

---

# 150. Intermediate Question — What Is Your Observability Philosophy?

Strong answer:

> I prefer useful, actionable and sustainable telemetry rather than maximum telemetry. Every metric, log and alert should have a purpose, owner and appropriate retention. I focus on user impact, SLOs and fast troubleshooting while controlling cardinality, log volume, performance, security and cost.

---

# 151. Intermediate Rapid-Fire — Prometheus

## Q: Pull or push?

> Primarily pull.

## Q: Query language?

> PromQL.

## Q: High-cardinality example?

> user_id or request_id as metric labels.

## Q: Common exporter?

> node_exporter.

## Q: What does `up == 0` indicate?

> Scrape failure.

## Q: How do you reduce Prometheus load?

> Control cardinality, scrape rate, targets and expensive queries.

## Q: How do you speed repeated expensive queries?

> Recording rules.

---

# 152. Intermediate Rapid-Fire — Grafana

## Q: Why dashboards become slow?

> Too many panels, expensive queries, large time ranges, frequent refresh.

## Q: First optimization?

> Identify expensive queries and panel load before scaling.

## Q: What makes a good dashboard?

> Focused, actionable, fast and owned.

---

# 153. Intermediate Rapid-Fire — ELK

## Q: What does Logstash do?

> Collect/process/transform/forward data.

## Q: What does Elasticsearch do?

> Index/search/analyze data.

## Q: What does Kibana do?

> Search and visualize Elasticsearch data.

## Q: What causes disk growth?

> Log volume, retention, replicas, large events, lack of ILM.

## Q: What causes JVM pressure?

> Too much data, queries, shards, heap workload.

---

# 154. Intermediate Rapid-Fire — Kubernetes

## Q: CrashLoopBackOff?

> Repeated container failures with restart backoff.

## Q: OOMKilled?

> Memory exhaustion.

## Q: Readiness?

> Traffic eligibility.

## Q: Liveness?

> Restart health.

## Q: DaemonSet?

> Ensures a pod runs on eligible nodes; useful for node-level log collectors.

## Q: Common node problems?

> CPU pressure, memory pressure, disk pressure, network issues.

---

# 155. Intermediate Rapid-Fire — SLO

## Q: SLI?

> Measurement.

## Q: SLO?

> Target.

## Q: SLA?

> Contract.

## Q: Error budget?

> Allowed unreliability.

## Q: Burn rate?

> Speed of error-budget consumption.

---

# 156. Intermediate Rapid-Fire — Troubleshooting

## Q: API latency high?

> Check traffic, latency distribution, errors, resources, dependencies, database, logs and recent changes.

## Q: Logs delayed?

> Trace collector, queue, Logstash and Elasticsearch.

## Q: Alert delayed?

> Trace scrape, rule evaluation, Alertmanager and notification.

## Q: Prometheus memory high?

> Check cardinality, ingestion, targets, exporters, churn and queries.

## Q: Elasticsearch disk high?

> Check ingestion, index growth, retention, ILM, replicas and shards.

---

# 157. Interview Answer Pattern for Intermediate Questions

Use:

    Situation
       |
       v
    Signals
       |
       v
    Investigation
       |
       v
    Root cause
       |
       v
    Mitigation
       |
       v
    Prevention

Example:

> Prometheus memory increased suddenly.

    Situation:
    Prometheus memory doubled.

    Signals:
    Active series and ingestion increased.

    Investigation:
    Compared recent deployment and metric cardinality.

    Root cause:
    New request_id label.

    Mitigation:
    Removed the label and rolled back instrumentation.

    Prevention:
    Added cardinality review to instrumentation changes.

This demonstrates production thinking.

---

# 158. Intermediate Interview Mistakes to Avoid

Avoid answers such as:

    "I will restart it."

    "I will increase memory."

    "I will add more nodes."

    "I will check CPU."

These may be valid actions, but they are not complete troubleshooting strategies.

Instead explain:

    What signal you check
    Why you check it
    What evidence confirms the hypothesis
    What mitigation you choose
    How you prevent recurrence

---

# 159. Seniority Signal in an Intermediate Answer

Weak:

> I know Prometheus and Grafana.

Better:

> I use Prometheus for metrics collection and PromQL for analysis, Grafana for operational dashboards, and I monitor Prometheus itself for cardinality, ingestion, query load and storage.

Stronger:

> I also design the telemetry model to control cardinality, align alerts with SLOs, manage retention and capacity, and make the platform resilient enough to support production incidents.

---

# 160. Final Intermediate Mental Model

For any production observability problem:

    USER IMPACT
        |
        v
    DETECT
        |
        v
    SCOPE
        |
        v
    CORRELATE
        |
        +---- Metrics
        +---- Logs
        +---- Kubernetes
        +---- Infrastructure
        +---- Dependencies
        +---- Recent changes
        |
        v
    ROOT CAUSE
        |
        v
    MITIGATE
        |
        v
    VALIDATE
        |
        v
    PREVENT

---

# 161. Key Intermediate Takeaways

> Design observability around user impact, not tools.

> Prometheus performance depends heavily on cardinality, ingestion and query behavior.

> Grafana dashboards must be designed for incident use, not just visualization.

> ELK performance depends on log volume, mappings, shards, storage and queries.

> Kubernetes monitoring must cover infrastructure, workloads and applications.

> Alerts should represent actionable conditions.

> SLOs are more meaningful for user-facing reliability than arbitrary infrastructure thresholds.

> Metrics detect, logs explain, and traces correlate distributed requests.

> Backpressure is useful for short bursts but cannot solve a permanent throughput mismatch.

> Scaling should follow investigation, not replace it.

> Observability systems need their own monitoring, security, capacity planning and DR strategy.

> A strong DevOps engineer thinks in terms of signals, dependencies, failure domains, user impact and recovery.

---

# 162. Completion

This completes:

    20-Interview-Preparation/
        02-Intermediate-Questions.md

Next:

    03-Advanced-Questions.md
