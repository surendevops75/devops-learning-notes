# Best Practices

> Production Monitoring, Observability & Logging Best Practices — Design Principles, Metrics, Prometheus, Grafana, ELK, Kubernetes/EKS, Applications, Alerting, SLI/SLO, Reliability, Security, Scalability, High Availability, Incident Response, Cost Awareness, Troubleshooting and DevOps/DevSecOps Interview Preparation

---

# 1. Purpose

Monitoring and observability should not be treated as a collection of tools.

A production observability system should answer:

    Is the system healthy?
    Are users affected?
    What changed?
    Where is the failure?
    Why did it happen?
    How severe is it?
    Is the system recovering?
    Can we prove recovery?
    How can we prevent recurrence?

Typical stack:

    Applications
        |
        +---- Metrics ----> Prometheus ----> Grafana
        |
        +---- Logs -------> ELK
        |
        +---- Traces -----> Tracing backend where implemented
        |
        v
    Kubernetes / EKS
        |
        v
    AWS Infrastructure

The goal is not:

> Collect everything.

The goal is:

> Collect the right signals, make them actionable, reliable, secure and cost-effective.

---

# 2. Core Observability Principles

A strong production design follows these principles:

1. Monitor user impact.
2. Prefer actionable signals.
3. Use metrics, logs and traces together.
4. Monitor dependencies, not only applications.
5. Design alerts around symptoms and service objectives.
6. Keep telemetry structured.
7. Control cardinality.
8. Protect observability infrastructure itself.
9. Retain data according to operational value.
10. Automate repetitive operational work.
11. Test monitoring and alerting.
12. Make dashboards useful for decisions.
13. Preserve enough historical data for troubleshooting.
14. Secure telemetry as production data.
15. Continuously improve based on incidents.

---

# 3. Monitoring vs Observability

Monitoring asks:

    Is CPU high?
    Is pod down?
    Is error rate high?

Observability asks:

    Why is error rate high?
    Which request path is failing?
    Which dependency caused it?
    Which deployment introduced the change?

Monitoring detects known failure conditions.

Observability helps investigate unknown failure modes.

A mature platform needs both.

---

# 4. Start With User Impact

Bad approach:

    CPU > 80%
    Therefore incident.

Better:

    User-facing errors ↑
       |
       v
    Latency ↑
       |
       v
    Dependency saturation
       |
       v
    Incident

Infrastructure metrics are useful, but service health and business impact should drive priorities.

---

# 5. Four Golden Signals

Use:

    Latency
    Traffic
    Errors
    Saturation

## Latency

How long requests take.

Track:

    p50
    p95
    p99

## Traffic

How much demand the system receives.

Examples:

    requests/sec
    messages/sec
    transactions/sec

## Errors

Failed requests or operations.

Examples:

    HTTP 5xx
    failed transactions
    queue processing errors

## Saturation

How close a resource is to capacity.

Examples:

    CPU
    memory
    DB connections
    queue depth
    thread pool utilization

---

# 6. RED Method

For request-driven services:

    Rate
    Errors
    Duration

Example:

    Rate:
    2,000 requests/sec

    Errors:
    0.5%

    Duration:
    p95 = 350ms

RED is especially useful for:

    APIs
    Microservices
    Web applications

---

# 7. USE Method

For infrastructure resources:

    Utilization
    Saturation
    Errors

Example:

    CPU utilization = 75%
    CPU run queue = high
    Hardware/network errors = 0

USE is useful for:

    Linux
    EC2
    Nodes
    Network
    Storage

---

# 8. Metrics Best Practice

Every important service should have:

    Availability
    Request rate
    Error rate
    Latency
    Resource utilization
    Dependency health

Avoid collecting metrics without a clear purpose.

Ask:

> What decision will this metric help me make?

---

# 9. Metric Naming

Use consistent names.

Example:

    http_requests_total
    http_request_duration_seconds
    process_cpu_seconds_total

Names should communicate:

    What
    Unit
    Type

Avoid ambiguous names such as:

    request_time

Prefer:

    http_request_duration_seconds

---

# 10. Counter Best Practices

Counters represent values that increase over time.

Examples:

    requests_total
    errors_total
    messages_processed_total

Counters should normally be queried using:

    rate()
    increase()

Do not treat a counter as a current instantaneous value.

---

# 11. Gauge Best Practices

Gauges represent values that can increase or decrease.

Examples:

    memory_usage_bytes
    active_connections
    queue_depth

Use gauges for current state.

---

# 12. Histogram Best Practices

Histograms are useful for distributions.

Typical use:

    Request latency

They allow analysis such as:

    p95
    p99

Prefer histograms when latency distribution matters.

---

# 13. Metric Units

Use base units consistently.

Examples:

    seconds
    bytes
    bytes/sec

Avoid mixing:

    milliseconds
    seconds

without clear naming.

Correct:

    request_duration_seconds

---

# 14. Label Design

Labels should have bounded cardinality.

Good:

    method
    endpoint
    status_code
    environment
    service

Dangerous:

    user_id
    request_id
    session_id
    email
    random UUID

High-cardinality labels can severely increase Prometheus resource usage.

---

# 15. Cardinality Rule

Think:

    Number of label combinations
        |
        v
    Number of time series

If every request creates a new label value:

    Series count ↑ dramatically.

Before adding a label ask:

> Does this label have a small, predictable set of values?

If not, do not use it as a Prometheus label.

---

# 16. Avoid Metric Explosion

Bad:

    request_id
    user_id
    transaction_id
    full_url

Better:

    service
    method
    route_template
    status_code

For example:

Bad:

    /users/12345

Better:

    /users/{id}

---

# 17. Metric Aggregation

Use recording rules for expensive repeated PromQL.

Example:

    service:http_error_rate:5m

This can reduce repeated query computation in dashboards and alerts.

---

# 18. Recording Rules

Useful for:

- Expensive queries
- Common dashboard queries
- SLO calculations
- Alert expressions

Benefits:

    Faster dashboards
    Faster alerts
    Consistent calculations
    Reduced query cost

---

# 19. PromQL Best Practices

Write queries that answer a specific operational question.

Example:

    Rate of HTTP 5xx responses

Conceptually:

    errors / total requests

Do not build dashboards containing dozens of expensive unrestricted queries.

---

# 20. Avoid Unbounded PromQL

Queries that scan excessive data can overload Prometheus.

Prefer:

    Appropriate time ranges
    Aggregation
    Recording rules
    Focused label matchers

---

# 21. Scrape Interval

Do not automatically choose:

    scrape_interval = 5s

for everything.

Choose based on:

    Signal importance
    Change frequency
    Storage capacity
    Alerting needs

Critical service metrics may need higher resolution than slow-changing infrastructure metrics.

---

# 22. Scrape Timeout

Timeout should be lower than or aligned with the scrape interval.

If scrape interval:

    15s

A target should not be allowed to hang indefinitely.

Monitor scrape failures.

---

# 23. Target Health Monitoring

Prometheus should monitor:

    up

for important targets.

If:

    up = 0

you may have:

    Application failure
    Network failure
    Service discovery issue
    Metrics endpoint failure

---

# 24. Exporter Best Practices

Use exporters when direct instrumentation is not practical.

Examples:

    node_exporter
    database exporters
    blackbox_exporter

Keep exporter scope clear.

Do not deploy unnecessary exporters everywhere.

---

# 25. Application Instrumentation

Instrument important application paths.

Capture:

    Request count
    Error count
    Request duration
    Dependency duration
    Business success/failure where appropriate

Avoid instrumenting every internal detail without operational value.

---

# 26. Business Metrics

Technical metrics may be healthy while business behavior is failing.

Examples:

    Orders created
    Payments successful
    Login success
    Checkout completion

Business metrics help identify:

> HTTP 200 but business failure.

---

# 27. Business Metrics Should Be Bounded

Good:

    payment_status="success|failed"

Bad:

    customer_id="123456"

Business telemetry must still respect cardinality and privacy.

---

# 28. Logging Best Practices

Production logs should be:

    Structured
    Consistent
    Searchable
    Correlated
    Secure
    Actionable

Prefer JSON or another structured format where the logging pipeline supports it.

---

# 29. Structured Logging

Example conceptual structure:

    timestamp
    level
    service
    environment
    message
    request_id
    trace_id
    endpoint
    status
    duration

Structured logs are easier to search and aggregate than arbitrary text.

---

# 30. Log Levels

Use levels intentionally:

    DEBUG
    INFO
    WARN
    ERROR

Do not use:

    ERROR

for normal business events.

Do not use:

    DEBUG

for every production request unless temporarily enabled under controlled conditions.

---

# 31. Logging Sensitive Data

Never log:

    Passwords
    API keys
    Access tokens
    Secret values
    Private keys
    Sensitive personal data

Use masking/redaction where necessary.

Logs often have broad access, so treat them as sensitive production data.

---

# 32. Log Volume Control

Excessive logging can cause:

    Application CPU ↑
    Network traffic ↑
    Storage ↑
    Elasticsearch load ↑
    Cost ↑

Logging must be balanced against troubleshooting needs.

---

# 33. Error Logging

A useful error should contain enough context to investigate:

    What failed?
    Which service?
    Which operation?
    What request?
    What dependency?
    What error category?

But avoid logging secrets or full sensitive payloads.

---

# 34. Stack Trace Best Practice

For unexpected application errors, stack traces are useful.

However:

    Do not expose stack traces to end users.

Return a safe error response while keeping detailed diagnostic information in protected logs.

---

# 35. Correlation IDs

Use a request/correlation identifier across service calls.

Example:

    Client
      |
      v
    Orders request_id=abc
      |
      v
    Payment request_id=abc
      |
      v
    Inventory request_id=abc

This makes log investigation easier.

---

# 36. Trace IDs

When distributed tracing is implemented, use trace IDs to connect:

    Logs
    Metrics context
    Traces

A trace can show:

    API
      |
      +---- Orders
      |
      +---- Payment
      |
      +---- Database

This is especially useful for latency troubleshooting.

---

# 37. Logging Architecture

Typical centralized architecture:

    Application
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

The architecture should avoid making the application directly dependent on Elasticsearch availability for normal request processing.

---

# 38. Kubernetes Logging

Prefer application logs to:

    stdout
    stderr

Then collect them centrally.

Avoid relying on local node files as the only copy of production logs.

Containers and nodes are replaceable.

---

# 39. Log Retention

Retention should be based on:

    Operational value
    Compliance requirements
    Incident investigation needs
    Storage cost

Do not retain everything forever.

Use different retention periods where appropriate:

    Hot
    Warm
    Cold/archive
    Delete

---

# 40. Elasticsearch Best Practices

Monitor:

    Cluster health
    Disk usage
    JVM memory
    Search latency
    Indexing rate
    Shard count
    Rejected operations

Avoid unlimited index growth.

---

# 41. Elasticsearch Shard Strategy

Too many shards cause:

    Metadata overhead
    Memory consumption
    Management complexity

Too few shards can limit:

    Parallelism
    Scaling

Shard design should reflect:

    Data volume
    Query patterns
    Retention
    Cluster size

---

# 42. Index Lifecycle Management

Use lifecycle policies to manage:

    Rollover
    Retention
    Deletion
    Storage tiers

This prevents old logs from consuming production storage indefinitely.

---

# 43. Logstash Best Practices

Keep pipelines:

    Simple
    Tested
    Observable

Monitor:

    Input rate
    Processing rate
    Queue size
    Errors
    Elasticsearch output

Avoid unnecessary expensive filters.

---

# 44. Kibana Dashboard Best Practices

Dashboards should answer operational questions.

Good dashboard sections:

    Service health
    Error rate
    Latency
    Traffic
    Dependency health
    Logs
    Infrastructure

Avoid dashboards that contain hundreds of unrelated graphs.

---

# 45. Dashboard Design

A useful dashboard should provide:

    Overview
       |
       v
    Problem identification
       |
       v
    Drill-down
       |
       v
    Root-cause evidence

Start with service-level health.

Then drill into:

    Pod
    Node
    Dependency
    Logs

---

# 46. Dashboard Time Range

Dashboards should allow:

    Last 15m
    Last 1h
    Last 6h
    Last 24h
    Custom range

During incidents compare:

    Current period
    Previous healthy period

---

# 47. Dashboard Naming

Use consistent names:

    Service Overview
    Kubernetes Cluster
    EKS Nodes
    Database
    ELK Health
    Prometheus Health

Avoid:

    Final Dashboard
    New Dashboard
    Test Dashboard

Operational dashboards should be maintainable.

---

# 48. Alerting Principles

A good alert should be:

    Actionable
    Relevant
    Timely
    Understandable

Ask:

> If this alert fires at 3 AM, what action should the engineer take?

If the answer is:

> Nothing.

It probably should not page.

---

# 49. Symptom-Based Alerts

Better:

    API availability < SLO

than:

    CPU > 80%

when CPU alone does not indicate user impact.

Infrastructure alerts are still useful for capacity and early warning, but paging should prioritize meaningful service impact.

---

# 50. Alert Severity

Example:

    Critical
    Warning
    Info

Define clear actions.

Example:

    Critical:
    Immediate response.

    Warning:
    Investigate during business hours.

Do not make every alert critical.

---

# 51. Alert Thresholds

Avoid arbitrary thresholds.

Instead of:

    CPU > 80%

ask:

    Does CPU > 80% predict service degradation?

Use historical data to establish meaningful thresholds.

---

# 52. `for` Duration

Avoid alerting on a one-second spike.

Use sustained conditions where appropriate.

Conceptually:

    condition true
       |
       v
    remains true
       |
       v
    alert

This reduces transient alert noise.

---

# 53. Alert Grouping

If 100 pods fail because one node is down:

Bad:

    100 pages

Better:

    One service/cluster incident

Use Alertmanager grouping appropriately.

---

# 54. Alert Inhibition

If a higher-level dependency failure causes many child alerts:

    Cluster outage
       |
       +---- Pod alerts
       +---- Service alerts
       +---- Application alerts

Inhibit redundant alerts where appropriate.

---

# 55. Alert Routing

Route based on:

    Service
    Severity
    Environment
    Team

Example:

    Production critical
        -> On-call

    Staging warning
        -> Team channel

Routing should be documented and tested.

---

# 56. Alert Testing

An alert that has never been tested is not reliable.

Test:

    Rule expression
    Evaluation
    Alertmanager routing
    Notification
    Escalation
    Recovery notification

Conduct controlled alert tests.

---

# 57. Alert Runbooks

Every important alert should link to a runbook.

Example:

    High API Error Rate

Runbook:

    1. Check deployment
    2. Check application logs
    3. Check dependency health
    4. Check recent changes
    5. Roll back if release-related

---

# 58. Alert Ownership

Every production alert should have:

    Owner
    Severity
    Meaning
    Action
    Runbook

Unowned alerts become operational debt.

---

# 59. Alert Fatigue

Symptoms:

    Too many alerts
    Engineers ignore notifications
    Important alerts get missed

Reduce noise by:

    Removing non-actionable alerts
    Grouping related alerts
    Adjusting thresholds
    Using SLO-based alerts
    Adding `for` durations
    Improving dependencies

---

# 60. SLI/SLO Integration

Observability should connect to service objectives.

Example:

    SLI:
    Successful requests / total requests

    SLO:
    99.9% successful requests

Alerting can then focus on:

    Error budget consumption
    Burn rate

This is more meaningful than arbitrary infrastructure thresholds.

---

# 61. SLO-Based Alerting

Suppose:

    SLO = 99.9%

A short but severe outage may burn the error budget rapidly.

A slow degradation may consume it gradually.

Burn-rate alerts detect both patterns.

---

# 62. Error Budget Best Practice

Error budget provides a balance between:

    Reliability
    Delivery speed

If the team continuously exhausts the budget:

    Improve reliability.

If the system consistently has a large unused budget:

    Carefully evaluate whether the SLO is appropriately defined.

---

# 63. Kubernetes Monitoring

Monitor:

    Nodes
    Pods
    Containers
    Deployments
    Services
    Ingress
    HPA
    Resource usage
    Restarts
    Scheduling
    Persistent volumes

---

# 64. Pod Monitoring

Important metrics:

    CPU
    Memory
    Restarts
    Ready state
    Container status
    Network
    Resource throttling

A Running pod can still be unhealthy.

Always consider:

    Ready

and:

    Application health.

---

# 65. Node Monitoring

Monitor:

    CPU
    Memory
    Disk
    Network
    Pressure conditions
    Pod density
    Runtime health

Important conditions:

    MemoryPressure
    DiskPressure
    PIDPressure

---

# 66. Kubernetes Resource Requests

Requests influence scheduling.

Best practice:

    Set realistic requests based on workload measurements.

Too low:

    Resource contention

Too high:

    Scheduling inefficiency

Use historical usage to tune requests.

---

# 67. Kubernetes Resource Limits

Limits provide boundaries.

Memory limits are particularly important because exceeding them can result in:

    OOMKilled

CPU limits can introduce throttling.

Do not copy arbitrary limits from another application.

---

# 68. HPA Best Practices

HPA should use metrics that reflect workload.

CPU may work for CPU-bound services.

For queue consumers:

    Queue depth

may be more meaningful.

Always configure:

    minReplicas
    maxReplicas
    target
    scale behavior

---

# 69. Autoscaling Safety

Before enabling aggressive autoscaling verify:

    Database capacity
    Cache capacity
    External API limits
    Network capacity
    Node capacity

Autoscaling without dependency capacity planning can create cascading failures.

---

# 70. Kubernetes Health Probes

Use:

    Startup probe
    Readiness probe
    Liveness probe

Understand the purpose of each.

Startup:

    Is application startup complete?

Readiness:

    Should receive traffic?

Liveness:

    Is the process unhealthy and should it be restarted?

---

# 71. Probe Best Practices

Avoid:

    Heavy database query
    External API call
    Expensive computation

inside frequent liveness probes.

A bad probe can create an outage.

---

# 72. Graceful Shutdown

Applications should handle:

    SIGTERM

Recommended flow:

    Readiness false
       |
       v
    Stop new traffic
       |
       v
    Finish active requests
       |
       v
    Close connections
       |
       v
    Exit

Configure:

    terminationGracePeriodSeconds

based on real shutdown time.

---

# 73. Kubernetes Deployment Strategy

Use appropriate:

    RollingUpdate
    Canary
    Blue-Green

For high-risk applications:

    Canary

can reduce blast radius.

---

# 74. Rollback Readiness

Before deploying:

    Know previous-good version
    Ensure image still exists
    Ensure schema compatibility
    Ensure configuration is compatible

A rollback plan that depends on a deleted image is not a real rollback plan.

---

# 75. GitOps Best Practice

Git should represent desired state.

Typical flow:

    Code
      |
      v
    CI
      |
      v
    Image
      |
      v
    Manifest update
      |
      v
    Git
      |
      v
    ArgoCD
      |
      v
    EKS

This provides:

    Auditability
    Repeatability
    Drift detection
    Easier rollback

---

# 76. Manual Changes in GitOps

Manual production changes create drift.

If emergency manual change is required:

    Make emergency change
       |
       v
    Restore service
       |
       v
    Document change
       |
       v
    Update Git
       |
       v
    Reconcile

Do not leave emergency changes undocumented.

---

# 77. CI/CD Observability

Monitor deployment pipeline metrics:

    Deployment frequency
    Deployment duration
    Failure rate
    Rollback rate
    Lead time

These metrics help identify delivery reliability.

---

# 78. Deployment Observability

After deployment compare:

    Before
    vs
    After

Metrics:

    Error rate
    Latency
    CPU
    Memory
    Restarts
    Dependency performance
    Business success

A deployment should have an observable validation window.

---

# 79. Production Change Correlation

Every incident investigation should ask:

    Was there a deployment?
    Config change?
    Terraform apply?
    Secret rotation?
    Certificate rotation?
    Feature flag change?
    Database migration?

Recent changes are often the highest-value clues.

---

# 80. Infrastructure Monitoring

For EC2 and Linux monitor:

    CPU
    Memory
    Disk
    Disk I/O
    Network
    Processes
    File descriptors
    Load
    Service state

Use:

    Prometheus exporters
    System tools
    AWS-native signals where appropriate

---

# 81. Linux Monitoring

Useful commands:

    top
    free -m
    df -h
    du -sh
    vmstat
    iostat
    ss -s
    ps -ef
    uptime

The objective is not to memorize commands.

Know what question each command answers.

---

# 82. Network Monitoring

Monitor:

    Connection count
    Errors
    Latency
    Packet drops
    Throughput
    DNS failures

For Kubernetes:

    Service connectivity
    Ingress
    CNI
    NetworkPolicy

---

# 83. Database Monitoring

Monitor:

    CPU
    Memory
    Connections
    Query latency
    Locks
    I/O
    Storage
    Replication
    Errors

Application monitoring without database monitoring creates blind spots.

---

# 84. Dependency Monitoring

For every critical dependency track:

    Availability
    Latency
    Error rate
    Saturation
    Timeouts
    Retry rate

Examples:

    RDS
    Redis
    RabbitMQ
    External APIs
    Authentication services

---

# 85. Dependency Budgeting

Application capacity should consider:

    DB connections
    Cache connections
    External API quota
    Queue throughput

A service is only as scalable as its critical dependencies.

---

# 86. High Availability for Observability

Observability itself can become a production dependency.

For critical environments consider:

    Prometheus HA
    Grafana HA
    Elasticsearch cluster
    Durable storage
    Alertmanager redundancy

Avoid making one monitoring node a single point of failure.

---

# 87. Monitoring Failure Must Not Cause Application Failure

The application should generally continue operating if:

    Prometheus unavailable
    Grafana unavailable
    Elasticsearch temporarily unavailable

Telemetry pipelines should be designed to fail safely.

---

# 88. Observability Data Pipeline Isolation

Application request path:

    User
      |
      v
    Application

Telemetry path:

    Application
      |
      v
    Observability pipeline

Keep telemetry failures from blocking core business requests.

---

# 89. Monitoring the Monitoring System

Monitor:

    Prometheus health
    Scrape success
    Scrape duration
    Alertmanager health
    Grafana availability
    Elasticsearch health
    Log ingestion
    Storage
    Collector health

Otherwise you may lose visibility during the exact incident when you need it most.

---

# 90. Observability Security

Protect:

    Metrics
    Logs
    Dashboards
    Alerting
    Trace data

Use:

    Authentication
    Authorization
    TLS
    Network restrictions
    Secret management

---

# 91. Least Privilege

Observability users do not all need:

    Admin

Use roles such as:

    Viewer
    Editor
    Administrator

Production access should follow least privilege.

---

# 92. Secrets in Telemetry

Potential leakage sources:

    Environment variables
    HTTP headers
    Authorization tokens
    Database URLs
    Request payloads

Implement:

    Redaction
    Filtering
    Secret detection
    Secure logging standards

---

# 93. Multi-Tenancy

If monitoring multiple teams/environments:

    Production
    Staging
    Development

Separate access appropriately.

Avoid allowing one team to view another team's sensitive logs or dashboards unless required.

---

# 94. Environment Separation

Use clear labels:

    environment="prod"
    environment="staging"

Avoid mixing production and non-production telemetry without obvious separation.

---

# 95. Production Data in Logs

Do not copy production payloads into development dashboards or debugging systems unnecessarily.

Use:

    Masking
    Sanitization
    Sampling
    Controlled access

---

# 96. Sampling Best Practices

For high-volume telemetry, sampling can reduce cost.

Use higher sampling for:

    Errors
    Slow requests
    Critical transactions

Use lower sampling for:

    Normal high-volume traffic

Sampling strategy should preserve enough data for incident investigation.

---

# 97. Log Sampling

If a service generates millions of identical successful requests:

    Do not necessarily retain every identical INFO event forever.

But preserve:

    Errors
    Warnings
    Important state changes
    Security events

---

# 98. Metrics Retention

Choose retention based on:

    Incident investigation
    Capacity planning
    SLO analysis
    Trend analysis

Short-term high-resolution data can coexist with longer-term aggregated data.

---

# 99. Observability Cost Optimization

Major cost drivers:

    Log volume
    Metric cardinality
    Trace volume
    Retention
    Storage
    Query frequency

Optimize these before simply reducing visibility.

---

# 100. Cost Optimization Principle

Do not ask:

> How do we collect less data?

Ask:

> Which data provides the highest operational value per unit cost?

---

# 101. Telemetry Tiers

Classify telemetry:

## Tier 1 — Critical

    Availability
    Error rate
    Latency
    Critical business metrics

## Tier 2 — Diagnostic

    Dependency metrics
    Detailed logs
    Runtime metrics

## Tier 3 — Deep Debugging

    High-detail logs
    Profiling
    Temporary instrumentation

This helps prioritize retention and cost.

---

# 102. Observability as Code

Version-control:

    Prometheus rules
    Grafana dashboards where practical
    Alertmanager configuration
    Log pipeline configuration
    Kubernetes monitoring manifests

Benefits:

    Review
    Audit
    Rollback
    Reproducibility

---

# 103. Dashboard as Code

Treat important dashboards like production artifacts.

Changes should be:

    Reviewed
    Tested
    Versioned

Avoid dashboards that only exist as undocumented manual UI changes.

---

# 104. Alert as Code

Alert rules should be version-controlled.

Benefits:

    Peer review
    Change history
    Automated deployment
    Reproducibility

---

# 105. Monitoring Configuration Testing

Test:

    PromQL syntax
    Rule logic
    Recording rules
    Dashboard queries
    Alert routing

A broken monitoring configuration can create a silent failure.

---

# 106. Synthetic Monitoring

Use synthetic checks for critical user journeys.

Example:

    Login
      |
      v
    Product
      |
      v
    Cart
      |
      v
    Checkout

This catches:

    DNS issues
    Routing issues
    Auth issues
    Application issues
    Dependency failures

---

# 107. Black-Box vs White-Box Monitoring

## Black-box

Checks from outside:

    HTTP endpoint
    DNS
    TCP
    User journey

Question:

> Is it working from the user's perspective?

## White-box

Looks inside:

    CPU
    Memory
    Internal metrics
    Application internals

Question:

> Why is it behaving this way?

Use both.

---

# 108. Monitoring Coverage

Create coverage for:

    Infrastructure
    Kubernetes
    Applications
    Dependencies
    Business transactions
    Monitoring platform

A service is not fully monitored if only CPU and memory are available.

---

# 109. Critical Path Monitoring

Identify:

    User
      |
      v
    API
      |
      v
    Orders
      |
      +---- Payment
      |
      +---- Inventory
      |
      v
    Database

This is the critical business path.

Prioritize deeper observability here.

---

# 110. Dependency Graph

Maintain a logical dependency map.

Example:

    Orders
      |
      +---- Payment
      +---- Inventory
      +---- RabbitMQ
      +---- RDS
      +---- Redis

When Orders fails, engineers can immediately inspect the dependency chain.

---

# 111. Service Ownership

Every production service should have:

    Owner
    Repository
    Deployment method
    Dashboard
    Alerts
    Runbook
    Dependencies
    Escalation path

Unknown ownership increases incident duration.

---

# 112. Service Catalog

Maintain metadata such as:

    Service name
    Team
    Environment
    Criticality
    SLO
    Dependencies
    Repository
    Dashboard
    Runbook

This turns observability into an operational system rather than a collection of dashboards.

---

# 113. Runbook Best Practices

A runbook should include:

    Symptoms
    Impact
    Checks
    Commands
    Metrics
    Logs
    Decision points
    Mitigation
    Rollback
    Validation
    Escalation

Avoid runbooks that say only:

    "Restart pod."

---

# 114. Runbook Decision Trees

Better:

    High error rate
       |
       +-- Recent deployment?
       |       |
       |       +-- Yes -> Compare version -> Rollback if justified
       |
       +-- No
               |
               +-- Dependency failing?
                       |
                       +-- Yes -> Protect dependency
                       |
                       +-- No -> Inspect application

Decision trees reduce incident response time.

---

# 115. Incident Response Integration

Monitoring should directly support:

    Detection
    Triage
    Investigation
    Mitigation
    Recovery
    RCA

A dashboard that cannot help during incidents has limited operational value.

---

# 116. Incident Timeline Correlation

Correlate:

    Deployment
    Config change
    Traffic
    Error rate
    Latency
    Resource usage
    Dependency events

Example:

    10:00 deployment
    10:02 error spike
    10:03 latency spike

This correlation is stronger than simply saying:

> "Application was unhealthy."

---

# 117. Post-Incident Improvements

Every major incident should ask:

    Did we detect quickly?
    Did we have enough telemetry?
    Was the alert actionable?
    Did we have a runbook?
    Was rollback easy?
    Did observability help?
    Was the root cause obvious?
    What should be automated?

---

# 118. Mean Time to Detect

MTTD:

    Incident detected time
    -
    Incident start time

Reduce MTTD through:

    Good alerts
    Synthetic monitoring
    SLOs
    Business metrics

---

# 119. Mean Time to Recover

MTTR:

    Recovery time
    -
    Incident detection/start

Reduce MTTR through:

    Runbooks
    Automation
    Rollbacks
    Clear ownership
    Good observability

---

# 120. Alert-to-Action Time

An alert should minimize:

    Alert
      |
      v
    Confusion
      |
      v
    Investigation

A good alert provides:

    What happened
    Where
    Severity
    Impact
    Link to dashboard
    Link to runbook

---

# 121. Production Readiness Checklist

Before onboarding a service:

    [ ] Metrics
    [ ] Logs
    [ ] Health checks
    [ ] Dashboard
    [ ] Alerts
    [ ] SLO
    [ ] Runbook
    [ ] Ownership
    [ ] Dependency map
    [ ] Deployment rollback
    [ ] Security review
    [ ] Capacity baseline

---

# 122. New Service Observability Checklist

## Metrics

    [ ] Rate
    [ ] Errors
    [ ] Duration
    [ ] Saturation

## Logs

    [ ] Structured
    [ ] Correlation ID
    [ ] Error context
    [ ] Sensitive data protected

## Alerts

    [ ] Actionable
    [ ] Correct severity
    [ ] Owner
    [ ] Runbook

## Dashboard

    [ ] Overview
    [ ] Dependencies
    [ ] Resources
    [ ] Drill-down

---

# 123. Production Deployment Checklist

Before deployment:

    [ ] Dashboard exists
    [ ] Alerts exist
    [ ] Baseline known
    [ ] Rollback tested
    [ ] Dependency capacity checked
    [ ] Resource requests reviewed
    [ ] Health probes validated

After deployment:

    [ ] Error rate
    [ ] Latency
    [ ] Traffic
    [ ] Restarts
    [ ] CPU
    [ ] Memory
    [ ] Dependency health
    [ ] Business success

---

# 124. Monitoring Change Checklist

Before changing monitoring:

    [ ] What problem does change solve?
    [ ] Is query correct?
    [ ] Is cardinality acceptable?
    [ ] Is cost acceptable?
    [ ] Is alert actionable?
    [ ] Is ownership clear?
    [ ] Is rollback possible?

---

# 125. Logging Change Checklist

Before increasing logs:

    [ ] Why is more detail required?
    [ ] Is sensitive data exposed?
    [ ] Expected volume?
    [ ] Elasticsearch capacity?
    [ ] Retention impact?
    [ ] Query usefulness?
    [ ] Can logging be temporary?

---

# 126. Alert Review Checklist

For every alert:

    [ ] What does it mean?
    [ ] Who owns it?
    [ ] What action is required?
    [ ] Is it tied to user impact?
    [ ] Is threshold justified?
    [ ] Is it noisy?
    [ ] Is runbook linked?
    [ ] Was it tested?

---

# 127. Dashboard Review Checklist

Ask:

    [ ] Does it show service health?
    [ ] Can I identify errors?
    [ ] Can I identify latency?
    [ ] Can I see traffic?
    [ ] Can I identify saturation?
    [ ] Can I inspect dependencies?
    [ ] Can I compare time periods?
    [ ] Is it readable during an incident?

---

# 128. Common Observability Anti-Pattern — Dashboard Overload

Problem:

    200 graphs

Engineer cannot find:

    Error rate

Better:

    Overview
       |
       v
    Problem
       |
       v
    Dependency
       |
       v
    Detail

---

# 129. Common Anti-Pattern — Alert Everything

Problem:

    Every metric has an alert.

Result:

    Alert fatigue

Better:

    Page on actionable symptoms.

Use lower-severity monitoring for early-warning conditions.

---

# 130. Common Anti-Pattern — Monitor Only Infrastructure

CPU:

    Healthy

Memory:

    Healthy

But:

    Checkout failure = 100%

Infrastructure-only monitoring misses business failures.

---

# 131. Common Anti-Pattern — Monitor Only Applications

Application:

    Healthy

Database:

    Storage full

Queue:

    Backlog

Network:

    Packet loss

Application metrics alone are insufficient.

---

# 132. Common Anti-Pattern — No Dependency Monitoring

Service A:

    Error rate ↑

Without dependency metrics:

    Root cause unclear.

Monitor:

    DB
    Cache
    Queue
    External APIs

---

# 133. Common Anti-Pattern — High Cardinality

Using:

    user_id
    request_id

as metric labels creates potentially enormous time-series counts.

Use logs/traces for high-cardinality investigation context instead.

---

# 134. Common Anti-Pattern — Secrets in Logs

Logging:

    Authorization: Bearer <token>

is a security incident waiting to happen.

Use redaction.

---

# 135. Common Anti-Pattern — Infinite Retention

Keeping all logs forever:

    Storage ↑
    Cost ↑
    Query performance ↓

Use lifecycle and retention policies.

---

# 136. Common Anti-Pattern — No Monitoring Runbook

Alert:

    High error rate

Engineer:

    "What should I do?"

Every critical alert should have a documented response.

---

# 137. Common Anti-Pattern — No Alert Testing

Configuration exists but notification is never tested.

During outage:

    Alert fires
       |
       X
    Notification fails

Test the complete path.

---

# 138. Common Anti-Pattern — No Monitoring of Monitoring

Prometheus fails.

No one notices.

Application outage occurs.

Engineers have no metrics.

Monitor the monitoring stack itself.

---

# 139. Common Anti-Pattern — Treating Running as Healthy

Kubernetes:

    STATUS = Running

But:

    Ready = false

or:

    Application returns 500

Running means the container process exists.

It does not guarantee service health.

---

# 140. Common Anti-Pattern — Restart as First Response

Restart may:

    Restore temporarily
    Destroy evidence
    Hide leaks

Use restart when it is an appropriate mitigation, but continue root-cause investigation.

---

# 141. Common Anti-Pattern — Scale as First Response

Scaling can help capacity incidents.

But if:

    Database is saturated

more pods may make things worse.

Always identify the bottleneck.

---

# 142. Common Anti-Pattern — Blind Timeout Increase

If API takes:

    20 seconds

increasing timeout from:

    10 -> 60 seconds

may create:

    More waiting requests
    More threads
    More connections
    More resource usage

Fix the underlying latency where possible.

---

# 143. Common Anti-Pattern — Retry Without Backoff

Bad:

    Retry immediately 100 times.

Better:

    Limited attempts
    Exponential backoff
    Jitter

Retries should protect the system, not amplify failure.

---

# 144. Common Anti-Pattern — Ignoring Business Metrics

HTTP:

    200

Business:

    Payment failed

Include business success indicators for critical workflows.

---

# 145. Common Anti-Pattern — No Baseline

Without a baseline:

    Is CPU 60% normal?

    Is 500 requests/sec normal?

    Is p95 500ms normal?

Establish normal behavior before incidents.

---

# 146. Baseline Monitoring

Record:

    Normal traffic
    Normal latency
    Normal CPU
    Normal memory
    Normal DB connections
    Normal queue depth

Use baseline comparisons during incidents.

---

# 147. Capacity Planning

Use historical trends:

    Traffic growth
    CPU growth
    Memory growth
    DB storage growth
    Log volume growth

Predict:

    When capacity will become a problem.

---

# 148. Capacity Headroom

Do not operate permanently at:

    99% capacity.

Maintain appropriate headroom for:

    Traffic spikes
    Node failures
    Deployments
    Maintenance
    Unexpected workloads

---

# 149. Failure Testing

Observability should be tested through controlled failure scenarios.

Examples:

    Kill pod
    Stop dependency in test environment
    Simulate latency
    Fill test queue
    Test node drain

Verify:

    Detection
    Alert
    Dashboard
    Runbook
    Recovery

---

# 150. Game Days

A game day simulates production failures.

Example:

    Database becomes unavailable.

Team practices:

    Detection
    Communication
    Investigation
    Mitigation
    Recovery

Game days expose gaps before real incidents.

---

# 151. Chaos Engineering

Chaos experiments should be:

    Controlled
    Hypothesis-driven
    Observable
    Reversible

Example hypothesis:

> If one application pod fails, the service remains within its SLO.

Then test.

---

# 152. Disaster Recovery Observability

During DR, verify:

    Metrics
    Logs
    Alerts
    Dashboards
    Synthetic tests

DR is incomplete if the application runs but the team cannot observe it.

---

# 153. Multi-AZ Observability

Dashboards should allow breakdown by:

    Availability Zone
    Node
    Service
    Pod

This helps identify localized failures.

---

# 154. EKS Observability Best Practice

For EKS monitor:

    Cluster
    Nodes
    Pods
    Services
    Ingress/ALB
    Applications
    RDS
    Network
    ECR/deployments
    Logging
    Prometheus
    Alerting

Do not treat EKS as only a Kubernetes problem.

It is a complete AWS application platform.

---

# 155. Production Architecture Best Practice

A mature platform looks conceptually like:

    Users
       |
       v
    Route53
       |
       v
    ALB
       |
       v
    EKS
       |
       +------------------------+
       |                        |
       v                        v
    Services                Observability
       |                        |
       +---- RDS                +---- Prometheus
       +---- Redis              +---- Grafana
       +---- RabbitMQ           +---- ELK
       +---- External APIs

Every critical dependency has:

    Metrics
    Logs where appropriate
    Alerts
    Ownership
    Runbook

---

# 156. Observability Data Flow Best Practice

Metrics:

    Application
       |
       v
    Prometheus
       |
       v
    Grafana
       |
       v
    Alerting

Logs:

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
       |
       v
    Kibana

Keep data flows simple enough to troubleshoot.

---

# 157. Reliability Best Practice

Production reliability depends on:

    Good architecture
    Good code
    Good deployment
    Good observability
    Good incident response
    Good capacity planning

Observability does not fix unreliable architecture by itself.

It makes failures visible and diagnosable.

---

# 158. Security Best Practice

Apply least privilege to:

    AWS
    Kubernetes
    Grafana
    Elasticsearch
    Prometheus
    CI/CD

Protect:

    Credentials
    Logs
    Metrics
    Dashboards
    Alert channels

---

# 159. Automation Best Practice

Automate:

    Dashboard deployment
    Alert deployment
    Rule validation
    Log pipeline configuration
    Health checks
    Synthetic checks
    Incident data collection where safe

Automation reduces human error.

---

# 160. Documentation Best Practice

Document:

    Architecture
    Dependencies
    Dashboards
    Alerts
    Runbooks
    SLOs
    Escalation
    Recovery
    Retention

Documentation should be version-controlled where possible.

---

# 161. Ownership Best Practice

For every critical alert:

    Service
      |
      v
    Team
      |
      v
    On-call
      |
      v
    Runbook

No owner means slow response.

---

# 162. Operational Simplicity

Prefer:

    Fewer meaningful alerts

over:

    Hundreds of noisy alerts.

Prefer:

    Simple architecture

over:

    Unnecessary components.

Prefer:

    Clear dashboards

over:

    Graph overload.

Operational simplicity is a reliability feature.

---

# 163. Observability Maturity Model

## Level 1 — Basic Monitoring

    CPU
    Memory
    Pod status

## Level 2 — Service Monitoring

    Rate
    Errors
    Latency
    Dashboards
    Alerts

## Level 3 — Centralized Observability

    Metrics
    Logs
    Traces
    Dependency monitoring

## Level 4 — Reliability Engineering

    SLI
    SLO
    Error budget
    Burn rate
    Incident automation

## Level 5 — Proactive Reliability

    Capacity forecasting
    Synthetic monitoring
    Game days
    Chaos testing
    Automated remediation where safe

---

# 164. Production Readiness Standard

A service should not be considered production-ready only because:

    Container builds
    Deployment succeeds

Production readiness should include:

    Observability
    Security
    Capacity
    Reliability
    Recovery
    Ownership
    Documentation

---

# 165. Practical DevOps Workflow

When onboarding a microservice:

    1. Build application
    2. Containerize
    3. Deploy to EKS
    4. Expose through Service/Ingress
    5. Add metrics
    6. Add structured logs
    7. Create dashboard
    8. Define alerts
    9. Define SLO
    10. Create runbook
    11. Test failure
    12. Validate recovery
    13. Document ownership

This is production thinking.

---

# 166. Observability Review Questions

Before production ask:

    Can we detect failure?
    Can we identify affected users?
    Can we identify the failing service?
    Can we identify dependencies?
    Can we investigate latency?
    Can we search logs?
    Can we correlate requests?
    Can we alert the correct team?
    Can we roll back?
    Can we validate recovery?

If several answers are "no", observability is incomplete.

---

# 167. Senior DevOps Interview — What Are Monitoring Best Practices?

Strong answer:

> I focus on actionable, service-oriented monitoring rather than collecting every possible metric. I use RED for applications, USE for infrastructure, and the four golden signals for overall service health. I keep Prometheus label cardinality controlled, use structured logs with correlation identifiers, design alerts around user impact and SLOs, maintain runbooks, monitor dependencies, and continuously test the monitoring and alerting system itself.

---

# 168. Interview — How Do You Avoid Prometheus High Cardinality?

Strong answer:

> I avoid unbounded labels such as request IDs, user IDs and session IDs. I use bounded dimensions such as service, method, route template and status code. For high-cardinality debugging information, I prefer logs or traces rather than Prometheus labels. I also monitor time-series growth and review new instrumentation before production.

---

# 169. Interview — How Do You Design Production Alerts?

Strong answer:

> I first identify the service-level symptom and expected operational action. I prefer SLO or user-impact-based alerts for paging, use appropriate durations to avoid transient noise, group related alerts through Alertmanager, and provide severity, ownership, dashboards and runbooks. I also test the complete notification path.

---

# 170. Interview — What Should You Monitor in Kubernetes?

Strong answer:

> I monitor node health, CPU, memory, disk, network, pod readiness, restarts, resource requests and limits, scheduling failures, deployments, services, ingress, HPA, persistent volumes and application-level RED metrics. I also monitor the dependencies behind the cluster, such as databases, caches and queues.

---

# 171. Interview — Why Is Logging Important if We Already Have Metrics?

Strong answer:

> Metrics tell me that something is abnormal, but logs often provide the detailed event or exception explaining what happened. Metrics are excellent for detection and trends, while logs provide high-detail context. I use them together rather than treating one as a replacement for the other.

---

# 172. Interview — Why Are Dashboards Not Enough?

Strong answer:

> Dashboards are useful for visualization and investigation, but they do not automatically detect incidents, route alerts or provide operational procedures. A production observability system also needs alerts, ownership, runbooks, SLOs and tested recovery processes.

---

# 173. Interview — What Is the Difference Between Monitoring and Observability?

Strong answer:

> Monitoring tells us whether known conditions are healthy, such as error rate, CPU or availability. Observability gives us enough telemetry to investigate why a system behaves unexpectedly. In practice I combine metrics, logs and traces with dependency and infrastructure signals.

---

# 174. Interview — How Do You Reduce Alert Fatigue?

Strong answer:

> I remove non-actionable alerts, group related failures, use appropriate thresholds and durations, prioritize service-level and SLO-based signals, add inhibition for dependent alerts, and review alert history. Every paging alert should have a clear action and owner.

---

# 175. Interview — How Do You Monitor a Microservices Application?

Strong answer:

> I define service-level RED metrics, structured application logs, correlation IDs, dependency metrics and distributed request visibility where tracing is implemented. On Kubernetes I also monitor pod, node, service and ingress health. I connect the telemetry to dashboards, alerts, SLOs and runbooks.

---

# 176. Interview — What Is a Good Production Dashboard?

Strong answer:

> A good dashboard starts with service health: availability, traffic, errors and latency. It then provides resource saturation and dependency health, followed by drill-down views for pods, nodes and logs. During an incident an engineer should be able to identify the problem and move toward root cause without searching through dozens of unrelated graphs.

---

# 177. Interview — What Is Monitoring as Code?

Strong answer:

> Monitoring as code means managing alerts, recording rules, dashboards and monitoring configuration through version control and automated deployment. It provides reviewability, audit history, reproducibility and rollback, similar to infrastructure as code.

---

# 178. Interview — How Do You Make Observability Cost-Effective?

Strong answer:

> I control metric cardinality, avoid unnecessary telemetry, use appropriate scrape intervals, manage log retention, use lifecycle policies, sample high-volume telemetry where appropriate, aggregate expensive metrics and prioritize high-value signals. I optimize cost without removing the telemetry required for incidents and SLOs.

---

# 179. Interview — How Do You Know a Service Is Production Ready?

Strong answer:

> I look beyond whether the deployment succeeds. The service needs meaningful metrics, structured logs, health checks, dashboards, actionable alerts, an SLO, runbook, ownership, dependency visibility, capacity baselines, security controls and a tested rollback or recovery path.

---

# 180. Interview — What Do You Do When Monitoring Itself Fails?

Strong answer:

> I first verify whether the application is actually affected independently of the monitoring stack. Then I investigate Prometheus, Grafana, Alertmanager and ELK health. I also ensure that monitoring systems have their own health alerts and appropriate redundancy so an observability failure does not become a blind spot during an application incident.

---

# 181. Production Best-Practice Checklist

## Monitoring

    [ ] Golden signals
    [ ] RED
    [ ] USE
    [ ] Business metrics
    [ ] Dependency metrics
    [ ] Capacity metrics

## Prometheus

    [ ] Controlled cardinality
    [ ] Appropriate scrape intervals
    [ ] Recording rules
    [ ] Target health
    [ ] Retention
    [ ] Storage monitoring

## Grafana

    [ ] Service dashboards
    [ ] Dependency dashboards
    [ ] Clear time ranges
    [ ] Drill-down
    [ ] Versioned dashboards where practical

## ELK

    [ ] Structured logs
    [ ] Sensitive data protection
    [ ] Retention
    [ ] Index lifecycle
    [ ] Elasticsearch health
    [ ] Logstash health

## Alerting

    [ ] Actionable
    [ ] Severity
    [ ] Ownership
    [ ] Runbook
    [ ] Grouping
    [ ] Inhibition
    [ ] Tested notifications

## Kubernetes

    [ ] Pod
    [ ] Node
    [ ] Service
    [ ] Ingress
    [ ] HPA
    [ ] Storage
    [ ] Resource requests/limits
    [ ] Probes

## Reliability

    [ ] SLI
    [ ] SLO
    [ ] Error budget
    [ ] Burn rate
    [ ] Incident response
    [ ] RCA

## Security

    [ ] Least privilege
    [ ] Authentication
    [ ] Authorization
    [ ] TLS
    [ ] Secret protection
    [ ] Log redaction

## Operations

    [ ] Ownership
    [ ] Runbooks
    [ ] Change correlation
    [ ] Capacity planning
    [ ] Failure testing
    [ ] DR validation

---

# 182. Final Production Observability Architecture

A mature architecture can be understood as:

    ================= USERS =================
                       |
                       v
                  Route53 / ALB
                       |
                       v
                 EKS / Kubernetes
                       |
          +------------+-------------+
          |            |             |
          v            v             v
       Service A    Service B    Service C
          |            |             |
          +------------+-------------+
                       |
          +------------+-------------+
          |            |             |
          v            v             v
         RDS         Redis        RabbitMQ
          |
          v
    External Dependencies


    APPLICATION TELEMETRY
          |
    +-----+------+----------------+
    |            |                |
    v            v                v
 Metrics       Logs            Traces
    |            |                |
    v            v                v
Prometheus    Logstash       Trace Backend
    |            |                |
    v            v                |
 Grafana    Elasticsearch         |
              |                   |
              v                   v
            Kibana          Correlation


    ALERTING
       |
       v
    Alert Rules
       |
       v
    Alertmanager
       |
       v
    On-Call / Team


    GOVERNANCE
       |
       +---- SLI/SLO
       +---- Runbooks
       +---- Ownership
       +---- Security
       +---- Cost
       +---- Capacity
       +---- DR
       +---- Incident Response

---

# 183. Final Best-Practice Mental Model

Do not think:

    "Which monitoring tool should I install?"

Think:

    USER
      |
      v
    SERVICE OBJECTIVE
      |
      v
    GOLDEN SIGNALS
      |
      v
    METRICS + LOGS + TRACES
      |
      v
    DEPENDENCIES
      |
      v
    ALERTS
      |
      v
    RUNBOOK
      |
      v
    INCIDENT RESPONSE
      |
      v
    RCA
      |
      v
    PREVENTION
      |
      v
    CONTINUOUS IMPROVEMENT

The strongest DevOps/DevSecOps observability practice is not collecting the most telemetry.

It is building a system where:

    Failure is detected quickly.
    Impact is understood quickly.
    Root cause can be investigated quickly.
    Recovery is safe.
    Data is secure.
    Telemetry is affordable.
    Ownership is clear.
    Lessons become engineering improvements.

---

# 184. Final Principles to Remember

> Monitor the user experience, not only infrastructure.

> Use metrics for detection and trends.

> Use logs for detailed event context.

> Use traces to understand distributed request paths when tracing is implemented.

> Use RED for services and USE for infrastructure.

> Keep Prometheus cardinality under control.

> Never put unbounded identifiers such as request IDs or user IDs into metric labels.

> Treat logs as sensitive production data.

> Alerts should trigger action, not curiosity.

> Every critical alert should have ownership and a runbook.

> A Running pod is not necessarily a healthy application.

> Autoscaling one layer can overload another.

> Monitor dependencies, not only services.

> Monitor the monitoring system.

> Version-control observability configuration where practical.

> Test alerts, dashboards and recovery procedures.

> Use SLOs to connect telemetry to reliability.

> Optimize telemetry cost without destroying incident visibility.

> After every serious incident, improve detection, diagnosis, mitigation or prevention.

The production observability mindset is:

    DETECT
       +
    UNDERSTAND
       +
    ACT
       +
    VERIFY
       +
    LEARN
       +
    IMPROVE

This completes:

    19-Best-Practices/
        01-Best-Practices.md

Next:

    02-Security.md
