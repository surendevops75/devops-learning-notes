# 20 - Interview Preparation
# 04 - Scenario-Based

> Monitoring, Observability & Logging — Scenario-Based DevOps Interview Preparation
>
> Focus: real production incidents, structured troubleshooting, metrics/logs/Kubernetes correlation, Prometheus, Grafana, ELK, EKS, SLOs, alerting, infrastructure and incident response.

---

# 1. How to Approach Any Production Scenario

Use this framework:

    USER IMPACT
        |
        v
    CONFIRM
        |
        v
    SCOPE
        |
        v
    TIMELINE
        |
        v
    SIGNALS
        |
        +---- Metrics
        +---- Logs
        +---- Kubernetes
        +---- Infrastructure
        +---- Dependencies
        +---- Recent changes
        |
        v
    HYPOTHESIS
        |
        v
    TEST
        |
        v
    MITIGATE
        |
        v
    VALIDATE
        |
        v
    ROOT CAUSE
        |
        v
    PREVENT

A strong interview answer should explain not only what command you run, but **why you run it and what decision the result enables**.

---

# 2. Scenario: Production API Error Rate Suddenly Reaches 20%

## Situation

The orders API normally has:

    Error rate = 0.5%

It suddenly reaches:

    Error rate = 20%

## Investigation

First establish:

    When did it start?
    Which endpoints?
    Which version?
    Which environment?
    Which users?
    Is traffic also changing?

Check:

    Request rate
    Error rate
    p95/p99 latency
    Pod restarts
    Application logs
    Dependency errors
    Recent deployment

## Commands

    kubectl get pods -n production
    kubectl get events -n production --sort-by=.lastTimestamp
    kubectl logs <pod> -n production --since=15m
    kubectl describe pod <pod> -n production

## PromQL example

    sum(rate(http_requests_total{
      service="orders",
      status=~"5.."
    }[5m]))
    /
    sum(rate(http_requests_total{
      service="orders"
    }[5m]))

## Likely causes

    Bad deployment
    Database failure
    Configuration error
    Dependency timeout
    Resource exhaustion
    Network issue

## Mitigation

If strongly correlated with a new release:

    Stop rollout
    Roll back if safe
    Validate recovery

## Interview answer

> I would first quantify the impact and establish the timeline. Then I would compare the error rate by service, endpoint and version, correlate it with latency and traffic, and check application logs, Kubernetes events and dependencies. If a recent deployment is strongly correlated and rollback is safe, I would mitigate first and then investigate the root cause.

---

# 3. Scenario: Error Rate Is Normal but Latency Is Extremely High

## Situation

    5xx = normal
    p95 latency = 4 seconds

## Investigation

Do not assume the service is healthy.

Check:

    Database latency
    Connection pool
    Thread pool
    External API latency
    Queue depth
    CPU
    Memory
    Network
    Locks

## Key insight

Requests can succeed while taking too long.

## Interview answer

> I would treat this as a latency incident rather than an error incident. I would inspect p95/p99, dependency latency, connection pools, thread pools and database performance. I would also check whether traffic increased or a recent release changed execution behavior.

---

# 4. Scenario: CPU Is Normal but API Requests Time Out

## Possible causes

    Database
    Network
    External API
    Thread pool exhaustion
    Connection pool exhaustion
    Lock contention
    Queueing

## Investigation

    API latency
    Dependency latency
    Timeout count
    Active connections
    Thread utilization
    Database waits

## Key interview point

> CPU is not a complete measure of application health.

---

# 5. Scenario: Memory Usage Increases Continuously

## Pattern

    40%
    50%
    60%
    70%
    80%
    90%

with relatively stable traffic.

## Investigation

Check:

    Pod memory
    Container limit
    Restarts
    OOMKilled
    Application version
    Heap if Java
    Garbage collection
    Traffic
    Deployment timeline

## Possible causes

    Memory leak
    Cache growth
    Unbounded queue
    Increased object retention
    Incorrect configuration

## Prevention

    Memory profiling
    Load testing
    Limits
    Regression tests
    Application fix

---

# 6. Scenario: Pod Is OOMKilled Every 10 Minutes

## First checks

    kubectl describe pod <pod> -n production
    kubectl logs <pod> -n production --previous
    kubectl get pod <pod> -n production -o json

Look for:

    OOMKilled
    Exit code
    Memory limit
    Restart count

## Important distinction

If the application legitimately needs more memory:

    Increase capacity appropriately

If memory usage keeps growing unexpectedly:

    Investigate leak

Do not simply increase the limit without evidence.

---

# 7. Scenario: CrashLoopBackOff After Deployment

## Investigation

    kubectl describe pod <pod>
    kubectl logs <pod>
    kubectl logs <pod> --previous

Check:

    Environment variables
    Secrets
    ConfigMaps
    Startup command
    Dependencies
    Probes
    Resource limits

## Common causes

    Application startup failure
    Missing secret
    Wrong configuration
    Dependency unavailable
    Bad image
    Probe failure

---

# 8. Scenario: Readiness Probe Starts Failing

## Impact

The pod may remain running but stop receiving traffic.

## Check

    Probe path
    Probe port
    Application startup
    Network
    Dependency
    Timeout
    Initial delay

## Key interview statement

> Readiness determines whether the workload should receive traffic; it is not primarily a restart mechanism.

---

# 9. Scenario: Liveness Probe Causes Constant Restarts

## Investigation

Check:

    Probe timeout
    Probe path
    Probe command
    Application startup
    CPU throttling
    Memory pressure

Ask:

> Is the application unhealthy, or is the liveness probe incorrectly detecting health?

A bad liveness probe can create the incident.

---

# 10. Scenario: ImagePullBackOff in Production

## Investigation

    kubectl describe pod <pod>

Check Events.

Possible causes:

    Wrong image name
    Wrong tag
    Registry unavailable
    Authentication failure
    Image deleted
    Network/DNS problem

For ECR:

    IAM
    Registry access
    Node/pod identity
    Network connectivity

---

# 11. Scenario: One Pod Is Much Slower Than All Others

## Situation

    Pod A: p99 = 200 ms
    Pod B: p99 = 250 ms
    Pod C: p99 = 8 sec

## Investigation

Compare:

    CPU
    Memory
    Node
    Pod version
    Request volume
    Logs
    Dependencies

Potential causes:

    Bad node
    Uneven traffic
    Connection pool problem
    Application-specific issue

Do not assume the entire deployment is broken.

---

# 12. Scenario: One Node Has Many Unhealthy Pods

## Investigation

Check:

    Node CPU
    Memory
    Disk
    Network
    Conditions
    Events

Commands:

    kubectl describe node <node>
    kubectl top node
    kubectl get pods -A -o wide

Possible causes:

    Memory pressure
    Disk pressure
    Network issue
    Runtime issue
    Resource overcommitment

---

# 13. Scenario: Node Goes Into Memory Pressure

## Investigation

    kubectl describe node <node>

Check:

    MemoryPressure
    Allocatable
    Requests
    Limits
    Evictions

Also inspect:

    kubectl top node
    kubectl top pod -A

## Root causes

    Overcommitment
    Memory leak
    Large workloads
    Incorrect requests

---

# 14. Scenario: Node Has Disk Pressure

## Symptoms

    Pods evicted
    Image pulls fail
    Container runtime problems

Check:

    df -h
    df -i

Investigate:

    Container images
    Container logs
    Temporary files
    Application files

## Important

Disk pressure can be caused by inode exhaustion even when disk capacity is not completely full.

---

# 15. Scenario: ALB Returns 5xx

## Investigation

Start outside-in:

    Client
      |
      v
    ALB
      |
      v
    Target
      |
      v
    Service
      |
      v
    Pod
      |
      v
    Application

Check:

    ALB metrics
    Target health
    Kubernetes endpoints
    Readiness
    Application logs

Determine whether the failure originates at the ALB or target.

---

# 16. Scenario: ALB Target Health Keeps Flapping

Possible causes:

    Unstable application
    Readiness issue
    Health endpoint dependency
    Network problem
    Timeout
    Resource pressure

Correlate:

    Target health
    Pod readiness
    Application health
    Node health

---

# 17. Scenario: DNS Works Intermittently

## Investigation

Check:

    DNS resolution
    Resolver
    CoreDNS
    Network
    Record TTL
    Application behavior

For Kubernetes:

    kubectl get pods -n kube-system
    kubectl logs -n kube-system <coredns-pod>

Also distinguish:

    DNS failure
    from
    downstream service failure.

---

# 18. Scenario: CoreDNS CPU Is High

Possible causes:

    High request volume
    Misbehaving clients
    DNS loop
    Large cluster
    Configuration issue

Investigate:

    DNS request rate
    CoreDNS CPU
    Errors
    Cache behavior
    Recent changes

Do not simply scale CoreDNS before understanding why traffic increased.

---

# 19. Scenario: Service Cannot Reach Another Service

Use:

    DNS
    Service
    Endpoints
    Network policy
    Ports
    Application logs

Commands:

    kubectl get svc
    kubectl get endpoints
    kubectl describe svc <service>

Then test from the affected pod where appropriate.

---

# 20. Scenario: Application Can Reach Some Dependencies but Not One

Example:

    Orders -> Database = OK
    Orders -> Payment = OK
    Orders -> Inventory = FAIL

Focus on:

    Inventory service
    Network path
    DNS
    Service endpoint
    Port
    Authentication
    Dependency health

Avoid investigating healthy dependencies indefinitely.

---

# 21. Scenario: Prometheus Target Is Down

Check:

    up{job="..."} == 0

Then:

    Target address
    Service discovery
    Network
    `/metrics`
    Port
    TLS
    Authentication

For Kubernetes:

    Labels
    ServiceMonitor/PodMonitor if used
    Namespace selection
    RBAC

---

# 22. Scenario: Prometheus Target Is Flapping

Possible causes:

    Application restarts
    Network instability
    Slow metrics endpoint
    Timeout
    Resource pressure

Check:

    Scrape duration
    Scrape samples
    Pod restarts
    Application CPU
    Network

---

# 23. Scenario: Prometheus Memory Doubles Overnight

## Investigation

Compare:

    Active series
    Samples/sec
    Target count
    Scrape frequency
    New metrics
    New labels
    Pod churn

Likely causes:

    Cardinality spike
    New exporter
    Instrumentation change
    Increased targets

## Key interview answer

> I would correlate the memory increase with changes in active series and ingestion rather than immediately increasing Prometheus memory.

---

# 24. Scenario: Prometheus CPU Is High

Check:

    Query load
    Rule evaluation
    Scrape processing
    Cardinality
    Remote write
    Dashboards

Determine whether CPU is consumed by:

    Ingestion
    Queries
    Rules

---

# 25. Scenario: Grafana Dashboard Becomes Slow

## Investigation

Check:

    Number of panels
    Refresh rate
    Time range
    PromQL
    Elasticsearch queries
    Transformations
    Data source latency

## Fixes

    Recording rules
    Query optimization
    Smaller time ranges
    Fewer panels
    Lower refresh frequency

---

# 26. Scenario: Grafana Shows "No Data"

Check:

    Data source
    Query
    Time range
    Variables
    Labels
    Authentication

For Prometheus:

    Run the query directly.

For Elasticsearch:

    Verify index/data view and timestamp field.

---

# 27. Scenario: Dashboard Shows Wrong Environment

Possible cause:

    Missing environment filter

Check:

    Query
    Variables
    Labels
    Data source

Use:

    environment
    cluster
    region

as explicit dimensions.

---

# 28. Scenario: Alert Fires in Prometheus but No Notification Arrives

Trace:

    Prometheus
       |
       v
    Alertmanager
       |
       v
    Route
       |
       v
    Receiver
       |
       v
    Notification

Check:

    Alertmanager connectivity
    Routing
    Matchers
    Silences
    Inhibition
    Receiver configuration

---

# 29. Scenario: Alertmanager Receives Alerts but Notification Fails

Check:

    Receiver
    Authentication
    Network
    Endpoint
    Rate limits
    External service

The alert pipeline has progressed successfully up to Alertmanager.

---

# 30. Scenario: Alertmanager Sends Duplicate Alerts

Check:

    Grouping
    Group interval
    Repeat interval
    Alert labels
    Multiple routes
    Duplicate alert sources

Use stable alert identity.

---

# 31. Scenario: Alert Volume Suddenly Explodes

## Investigation

Identify:

    Which alert?
    Which service?
    Which label?
    Which deployment?
    Which dependency?

Common causes:

    Alert per pod
    High cardinality
    Dependency outage
    Missing inhibition
    New rule

## Response

Group and suppress secondary symptoms where appropriate, but preserve root-cause alerts.

---

# 32. Scenario: Alert Is Too Noisy

Ask:

    Is it actionable?
    Does it represent user impact?
    Is threshold appropriate?
    Is duration appropriate?
    Is there a better SLO signal?

Tune:

    Threshold
    `for`
    Aggregation
    Severity
    Routing

Do not simply silence it permanently.

---

# 33. Scenario: Alert Fires During Every Deployment

Possible causes:

    Temporary startup behavior
    Probe behavior
    Alert too sensitive
    Metric discontinuity
    Expected rollout state

Improve:

    Alert condition
    `for`
    Deployment-aware logic
    Readiness monitoring

---

# 34. Scenario: CPU Alert Fires but Users Are Fine

This is a classic false-positive scenario.

CPU may be:

    High
    but
    no user impact

Question:

> Does this require immediate human action?

If not, consider:

    Lower severity
    Capacity alert
    SLO-independent notification

---

# 35. Scenario: Users Are Failing but CPU Alert Never Fires

Possible causes:

    Database failure
    DNS
    Network
    External API
    Certificate
    Authentication
    Application logic

This demonstrates why CPU alone is insufficient.

---

# 36. Scenario: Logs Suddenly Stop Appearing in Kibana

Trace:

    Application
       |
       v
    stdout/stderr
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

Check each stage.

---

# 37. Scenario: Log Collector Is Running but No Logs Are Arriving

A running process does not prove successful ingestion.

Check:

    File paths
    Container runtime
    Permissions
    Collector configuration
    Network
    Output errors

---

# 38. Scenario: Logstash Queue Is Growing

Interpretation:

    Input rate > processing/output rate

Check:

    CPU
    Memory
    Parsing
    Grok
    Output
    Elasticsearch
    Network

If Elasticsearch is slow, Logstash may accumulate backlog.

---

# 39. Scenario: Elasticsearch Is Slow to Index Logs

Check:

    Indexing rate
    CPU
    JVM
    Disk
    Shards
    Replicas
    Mapping
    Log volume

Determine:

    Producer overload
    or
    Backend capacity issue.

---

# 40. Scenario: Elasticsearch Search Is Slow

Check:

    Query
    Time range
    Aggregation
    Shards
    JVM
    Disk
    Search queue

A cluster can be healthy while a particular query is poorly designed.

---

# 41. Scenario: Elasticsearch Disk Usage Jumps 40%

Investigate:

    Which indices?
    Which services?
    Log volume?
    DEBUG logging?
    Duplicate logs?
    Replica changes?
    Retention?
    ILM?

Then calculate:

    Growth rate
    Days to exhaustion

---

# 42. Scenario: Elasticsearch Turns Yellow

Check:

    Unassigned replicas
    Node count
    Allocation rules
    Disk watermarks
    Failure domains

Common simple example:

    One node
    replica count = 1

The replica cannot be placed on the same node.

---

# 43. Scenario: Elasticsearch Turns Red

Prioritize:

    Identify unassigned primary shards
    Identify affected indices
    Check node failures
    Check disk
    Check allocation

Red means some primary data is unavailable and requires immediate investigation.

---

# 44. Scenario: Elasticsearch Is Green but Logs Are Missing

Green only means shard allocation is healthy.

Still check:

    Collector
    Logstash
    Indexing
    Timestamp
    Kibana filters
    Index pattern

---

# 45. Scenario: Kibana Shows No Logs but Elasticsearch Contains Data

Possible causes:

    Wrong data view
    Wrong timestamp field
    Wrong time range
    Query filter
    Index pattern

Verify directly in Elasticsearch before blaming ingestion.

---

# 46. Scenario: Logs Arrive 20 Minutes Late

Measure:

    Event timestamp
    Collector timestamp
    Logstash processing
    Elasticsearch indexing

Find where delay accumulates.

Likely:

    Backpressure
    Queue
    Slow Elasticsearch
    Network

---

# 47. Scenario: Log Volume Doubles After a Deployment

Compare:

    Before deployment
    After deployment

Check:

    Log level
    Retry loops
    Exception loops
    Request payload logging
    Duplicate messages

Fix at source.

---

# 48. Scenario: Application Is Logging Passwords

Immediate priority:

    Stop exposure
    Rotate compromised credentials if necessary
    Remove sensitive logging
    Restrict access
    Review retention

Then:

    Add automated log scanning
    Secure logging standards
    Code review

---

# 49. Scenario: Logs Contain Request IDs but Metrics Do Not

Do not add request IDs to metric labels.

Use request IDs in:

    Logs
    Traces

Use bounded dimensions in metrics:

    service
    route
    method
    status

This prevents cardinality explosion.

---

# 50. Scenario: Cannot Correlate Logs From Multiple Services

Standardize:

    correlation_id
    service
    version
    environment
    timestamp

Ensure these fields survive the entire logging pipeline.

---

# 51. Scenario: Database CPU Is 30% but Application Has DB Timeouts

Check:

    Connection pool
    Max connections
    Lock contention
    Query latency
    Network
    Database connection errors

CPU alone cannot prove database health.

---

# 52. Scenario: Database Connections Are Exhausted

Architecture:

    Many pods
       |
       v
    Connection pools
       |
       v
    Database
       |
       v
    Connection limit

Possible causes:

    Too many pods
    Pool too large
    Connection leak
    Traffic spike

Mitigation:

    Control pool size
    Fix leaks
    Scale database if justified
    Reduce unnecessary connections

---

# 53. Scenario: RabbitMQ Queue Depth Keeps Growing

Check:

    Publish rate
    Consume rate
    Consumer count
    Consumer errors
    Message processing time
    Unacked messages

If:

    publish rate > consume rate

backlog will grow.

---

# 54. Scenario: RabbitMQ Queue Depth Is Normal but Users Are Slow

Check:

    Message age
    Processing latency
    Downstream dependencies

Queue depth alone can hide latency problems.

---

# 55. Scenario: One Consumer Is Slow

Compare:

    Consumer throughput
    CPU
    Memory
    Processing time
    Errors
    Dependency latency

Possible causes:

    Slow database
    Bad message
    Resource issue
    Code regression

---

# 56. Scenario: Retry Count Suddenly Increases

Check:

    Dependency errors
    Timeout rate
    Application release
    Retry policy
    Circuit breaker state

A retry increase may be the symptom of a dependency incident.

---

# 57. Scenario: Retry Storm Overloads a Dependency

Flow:

    Dependency failure
       |
       v
    Retries
       |
       v
    More dependency load
       |
       v
    More failures

Mitigate with:

    Exponential backoff
    Jitter
    Retry limits
    Circuit breaker
    Rate limiting

---

# 58. Scenario: External API Is Slow

Measure:

    Request rate
    Error rate
    p95/p99 latency
    Timeout rate
    Retry rate

Separate:

    Your application latency

from:

    External dependency latency.

---

# 59. Scenario: One Microservice Causes Latency Across the Platform

Example:

    Payment slow
       |
       v
    Orders waits
       |
       v
    Checkout waits
       |
       v
    User-facing latency

Use:

    Dependency metrics
    Logs
    Tracing where available
    Timeouts
    Queue metrics

This is a cascading dependency scenario.

---

# 60. Scenario: Deployment Causes Latency Increase but No Errors

Check:

    Version
    Endpoint latency
    CPU
    Database queries
    External dependencies
    Connection pools

Possible cause:

    New code performs slower operations without failing.

---

# 61. Scenario: Deployment Causes Error Spike

Immediate approach:

    Confirm version correlation
    Stop rollout
    Compare old/new
    Roll back if safe
    Validate

Then investigate:

    Code
    Config
    Secrets
    Dependency compatibility
    Database migration

---

# 62. Scenario: Deployment Is Successful but Application Is Not Healthy

Deployment status:

    Success

does not mean:

    Application health = success

Check:

    Readiness
    Error rate
    Latency
    Business metrics
    Dependency health

---

# 63. Scenario: HPA Is Scaling but Latency Is Still Increasing

Check:

    HPA metric
    Current replicas
    Desired replicas
    Pending pods
    Pod startup time
    Cluster capacity
    Database capacity
    Connection limits

Possible root cause:

    Downstream dependency bottleneck.

---

# 64. Scenario: HPA Does Not Scale During Traffic Spike

Check:

    Metric availability
    HPA configuration
    Target value
    Current metric
    Max replicas
    Resource requests
    Metrics pipeline

Do not assume Kubernetes itself is broken.

---

# 65. Scenario: Pods Scale Rapidly Up and Down

This is scaling thrash.

Possible causes:

    Threshold too sensitive
    Short-lived spikes
    Poor stabilization
    Incorrect metric

Use:

    Appropriate scaling windows
    Stabilization
    Better metric design

---

# 66. Scenario: Cluster Has Capacity but Pods Cannot Schedule

Check:

    Requests
    Taints
    Tolerations
    Affinity
    Node selectors
    Pod topology constraints
    Resource availability

Commands:

    kubectl describe pod <pod>

Look at Events.

---

# 67. Scenario: Pods Are Pending for CPU

Check:

    Node allocatable
    Requests
    Current usage
    Autoscaling
    Node groups

Important:

> Scheduling is based primarily on resource requests, not simply current CPU usage.

---

# 68. Scenario: Pods Are Evicted Despite Low Application CPU

Possible causes:

    Memory pressure
    Disk pressure
    Ephemeral storage
    Node conditions

Check node state and pod resource usage.

---

# 69. Scenario: Monitoring Data Is Missing After Prometheus Restart

Check:

    Persistent storage
    Retention
    WAL recovery
    Storage mount
    Configuration

Distinguish:

    Data loss

from:

    Query time range problem.

---

# 70. Scenario: Prometheus Disk Is Filling Rapidly

Check:

    Active series
    Samples/sec
    Retention
    Scrape interval
    New targets
    WAL

Do not delete TSDB files manually as a first response.

---

# 71. Scenario: Remote Write Queue Is Growing

Interpretation:

    Local ingestion > remote backend throughput

Check:

    Backend health
    Network
    Queue capacity
    Failed samples
    Retry rate

Long outages can eventually cause data loss.

---

# 72. Scenario: Prometheus and Grafana Are Healthy but Dashboards Show Stale Data

Check:

    Scrape freshness
    Prometheus ingestion
    Query cache
    Dashboard refresh
    Data source

A healthy process can still serve stale information.

---

# 73. Scenario: Alert Fires 10 Minutes After the Actual Incident

Trace timing:

    Incident
       |
       v
    Metric change
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

Find the largest delay.

---

# 74. Scenario: Alert Is Correct but Comes Too Frequently

Check:

    Rule interval
    Alert duration
    Grouping
    Repeat interval
    Recovery behavior

Tune the alerting pipeline instead of changing the actual incident signal blindly.

---

# 75. Scenario: SLO Is Violated but No Alert Fired

This is a serious alerting failure.

Check:

    SLI calculation
    Recording rules
    Alert expression
    Evaluation interval
    Alertmanager
    Routing
    Silences

Then test the entire SLO path.

---

# 76. Scenario: Error Budget Is Being Consumed Rapidly

Check:

    Error rate
    Latency SLI
    Incident cause
    Deployment
    Dependency

If burn rate is severe:

    Mitigate
    Reduce risky changes
    Prioritize reliability work

---

# 77. Scenario: SLO Is Healthy but Customers Complain

Possible reasons:

    SLI does not represent user experience
    Business metric missing
    Wrong traffic excluded
    Blackbox failure
    Measurement gap

Review the SLI itself.

---

# 78. Scenario: SLO Looks Excellent Because Errors Are Excluded

This is SLO gaming.

Check:

    Exclusion rules
    Client errors
    Dependency errors
    Invalid requests

The SLI should reflect meaningful user experience.

---

# 79. Scenario: Blackbox Monitoring Says Site Is Down but Internal Metrics Are Healthy

This is a valuable scenario.

Check:

    DNS
    TLS
    ALB
    Network
    Routing
    External path

Internal health does not guarantee external availability.

---

# 80. Scenario: Internal Metrics Are Failing but Blackbox Is Healthy

Possible causes:

    Prometheus outage
    Scrape failure
    Exporter failure
    Metrics pipeline issue

The application may still be healthy.

This demonstrates why monitoring the monitoring platform matters.

---

# 81. Scenario: Certificate Expires Tomorrow

Priority:

    Confirm certificate
    Confirm affected endpoints
    Verify renewal mechanism
    Renew/rotate
    Validate TLS
    Monitor expiration

Do not wait for expiry.

---

# 82. Scenario: Monitoring Platform Has a Certificate Problem

Impact:

    Metrics access
    Dashboard access
    Log access
    Alert delivery

Check:

    Certificate chain
    Expiry
    Trust
    Endpoint configuration

Monitoring infrastructure is production infrastructure.

---

# 83. Scenario: One Availability Zone Loses Nodes

Check:

    Pod distribution
    Node capacity
    PDB
    Autoscaling
    ALB targets
    Workload health

If workloads are properly distributed:

    Other AZs should absorb traffic.

---

# 84. Scenario: All Pods of a Critical Service Are in One AZ

This is an architecture risk.

Use:

    Topology spread
    Pod anti-affinity
    Multi-AZ nodes
    Capacity planning

Observability should reveal workload concentration.

---

# 85. Scenario: Entire EKS Cluster Becomes Unreachable

Outside-in approach:

    External monitor
       |
       v
    DNS
       |
       v
    ALB
       |
       v
    EKS
       |
       v
    Application

Use independent monitoring where possible.

---

# 86. Scenario: Entire Region Fails

Questions:

    Did external monitoring detect it?
    Can alerts be delivered?
    Is DR available?
    Are critical dependencies replicated?
    Is data recoverable?

Evaluate:

    RPO
    RTO

---

# 87. Scenario: Observability System Fails During a Major Incident

This is a priority incident.

Use:

    External monitoring
    Cloud/provider signals
    Application logs
    Secondary monitoring
    Local system data

Restore observability while maintaining service recovery.

---

# 88. Scenario: Elasticsearch Node Fails

Check:

    Cluster health
    Primary/replica assignment
    Remaining capacity
    Disk
    Recovery

If replicas exist across failure domains:

    Service can continue with reduced redundancy.

---

# 89. Scenario: Elasticsearch Replica Is Not Assigned

Check:

    Node count
    Allocation rules
    Disk watermarks
    Awareness settings

Do not immediately increase replica count.

---

# 90. Scenario: Elasticsearch Storage Is Near Capacity

Immediate:

    Confirm growth
    Protect cluster
    Review retention
    Review ILM
    Remove only data that policy permits
    Add capacity if needed

Then:

    Fix the growth source.

---

# 91. Scenario: Logstash Goes Down

Potential impact:

    Logs stop flowing

Recovery:

    Restart/replace collector
    Check queue
    Check backlog
    Validate ingestion

If buffering exists:

    Replay safely.

---

# 92. Scenario: Node-Level Log Collector Goes Down

Only that node's log collection may be affected, depending on architecture.

Check:

    DaemonSet status
    Collector logs
    Node
    File access

This demonstrates why DaemonSets reduce blast radius.

---

# 93. Scenario: Collector Is Healthy but Logs Are Missing From One Node

Compare:

    Container runtime logs
    File path
    Permissions
    Collector configuration
    Node health

Focus on the affected node rather than the whole platform.

---

# 94. Scenario: One Service Produces Huge Log Events

Large events can affect:

    Network
    Logstash
    Elasticsearch
    Storage
    Query performance

Investigate:

    Payload logging
    Stack traces
    Serialization
    Request bodies

Reduce event size at source.

---

# 95. Scenario: Elasticsearch Mapping Explosion

Possible causes:

    Dynamic fields
    Arbitrary JSON
    User metadata

Mitigation:

    Explicit mappings
    Dynamic templates
    Field restrictions
    Application schema control

---

# 96. Scenario: Elasticsearch Query Uses High-Cardinality Aggregation

Possible effects:

    High memory
    Slow query
    Circuit breaker
    Cluster instability

Investigate:

    Query
    Aggregation
    Time range
    Field type

Use appropriate `keyword` fields and bounded queries.

---

# 97. Scenario: Monitoring Cost Doubles Without Traffic Doubling

Investigate:

    Metric cardinality
    Log volume
    Retention
    Scrape interval
    Query volume
    Replica count
    Storage

Telemetry growth should be compared with workload growth.

---

# 98. Scenario: One Team Generates Most Observability Cost

Use:

    Service attribution
    Log GB/day
    Metric series
    Query usage

Then work with the team to optimize:

    Debug logging
    Labels
    Retention
    Dashboards

Avoid cost optimization without evidence.

---

# 99. Scenario: Developers Want Request ID as Prometheus Label

Explain:

    Request IDs are high-cardinality.

Use:

    Logs
    Traces

instead.

Metrics should remain aggregatable.

---

# 100. Scenario: Developers Want User ID in Every Metric

Same principle.

Avoid:

    user_id
    email
    session_id

in metric labels.

Use logs or traces for request-level investigation.

---

# 101. Scenario: New Metric Creates Millions of Series

Immediate:

    Identify metric
    Identify labels
    Stop rollout if necessary
    Remove unbounded label
    Verify Prometheus recovery

Then add:

    Instrumentation review
    Cardinality testing

---

# 102. Scenario: Dashboard Has 100 Panels

Symptoms:

    Slow loading
    High Prometheus CPU
    Poor incident usability

Redesign:

    Executive summary
    Service health
    Detailed drill-down

Use recording rules for repeated calculations.

---

# 103. Scenario: On-Call Engineer Is Overwhelmed by Alerts

Measure:

    Alert volume
    Duplicate alerts
    False positives
    Severity

Then:

    Remove non-actionable pages
    Group related alerts
    Add inhibition
    Align critical alerts to SLOs
    Add runbooks

---

# 104. Scenario: Alert Has No Owner

Do not leave it as a generic platform alert.

Assign:

    Team
    Service
    Severity
    Runbook
    Escalation

Ownership is part of alert design.

---

# 105. Scenario: Permanent Silence Exists for a Critical Alert

Treat as a risk.

Investigate:

    Why was it silenced?
    Who owns it?
    Is the underlying problem fixed?

Replace permanent silence with:

    Correct alert
    Appropriate threshold
    Root-cause fix

---

# 106. Scenario: New Monitoring Rule Causes Alert Storm

Mitigate:

    Disable/rollback the bad rule if safe
    Group alerts
    Protect notification systems

Then:

    Fix rule
    Validate in staging
    Reintroduce gradually

---

# 107. Scenario: Monitoring Upgrade Causes Data Loss

Investigate:

    Storage
    Compatibility
    Configuration
    Migration
    Backup

Recovery:

    Restore from backup if required
    Re-establish collection
    Validate data flow

Prevention:

    Upgrade testing
    Backup
    Rollback plan

---

# 108. Scenario: Grafana Upgrade Breaks Dashboards

Check:

    Datasources
    Plugins
    Variables
    Queries
    Dashboard compatibility

Rollback if production impact is high and safe rollback exists.

---

# 109. Scenario: Prometheus Upgrade Causes Rule Errors

Check:

    Rule syntax
    Query behavior
    Version compatibility

Validate:

    Rules
    Alerts
    Dashboards

before broad rollout.

---

# 110. Scenario: Observability Configuration Differs Between Staging and Production

This creates configuration drift.

Use:

    Git
    Code review
    CI validation
    GitOps

Compare:

    Prometheus config
    Alert rules
    Grafana provisioning
    Log pipeline

---

# 111. Scenario: Production Dashboard Works Manually but Fails for On-Call

Check:

    RBAC
    Datasource permissions
    Authentication
    Network access
    Variables

Operational usability is part of production readiness.

---

# 112. Scenario: Monitoring Data Is Fresh in One Region but Stale in Another

Compare:

    Collectors
    Network
    Storage
    Remote write
    Time synchronization
    Regional capacity

Use explicit:

    cluster
    region
    environment

labels.

---

# 113. Scenario: Metrics From Two Clusters Look Identical

Possible cause:

    Missing cluster label

Add:

    cluster
    region
    environment

This is especially important for centralized dashboards.

---

# 114. Scenario: Central Metrics Backend Becomes Unavailable

Questions:

    Does local collection continue?
    Is data buffered?
    How long?
    What is the RPO?
    What happens when backend returns?

Monitor:

    Queue
    Backlog
    Storage

---

# 115. Scenario: Central Logging Backend Becomes Unavailable

Desired behavior:

    Collector
       |
       v
    Buffer
       |
       v
    Backend recovery
       |
       v
    Replay

But buffers have finite capacity.

Define:

    Maximum outage duration tolerated.

---

# 116. Scenario: Observability Platform Is Under Heavy Load

Use graceful degradation.

Prioritize:

    Critical metrics
    Critical alerts
    Security logs

Reduce:

    Debug telemetry
    Noncritical queries
    Excessive dashboards

---

# 117. Scenario: Monitoring Platform Itself Has High Memory

Apply the same troubleshooting process:

    Which component?
    What changed?
    What workload?
    What resource?
    What dependency?

Do not give the monitoring system special treatment.

---

# 118. Scenario: Incident Has Multiple Simultaneous Symptoms

Example:

    API errors
    Database latency
    Queue growth
    Pod CPU

Build a causal chain.

Possible:

    Database
       |
       v
    API latency
       |
       v
    Queue
       |
       v
    Retries
       |
       v
    CPU

Avoid treating every symptom as an independent root cause.

---

# 119. Scenario: Two Teams Blame Each Other

Use evidence.

Correlate:

    Request metrics
    Dependency metrics
    Logs
    Timelines
    Version
    Network

Ask:

> What signal proves where the latency/error originates?

Avoid opinion-based incident management.

---

# 120. Scenario: You Cannot Prove the Root Cause

Be precise:

    Confirmed:
    API latency increased at 10:03.

    Confirmed:
    Database latency increased at 10:04.

    Hypothesis:
    Database degradation caused API latency.

Need:

    Additional evidence

Do not claim certainty without evidence.

---

# 121. Scenario: Incident Is Resolved

Do not stop when:

    Alert clears.

Validate:

    Error rate
    Latency
    Traffic
    Queue
    Dependencies
    Business metric
    SLO
    Logs

Then document:

    Timeline
    Root cause
    Mitigation
    Preventive action

---

# 122. Scenario: Post-Incident Review

Include:

    Impact
    Detection
    Timeline
    Root cause
    Contributing factors
    Mitigation
    Recovery
    What went well
    What failed
    Action items

Avoid blame.

Focus on system improvement.

---

# 123. Scenario: MTTD Is High

Improve:

    Detection signals
    SLO alerts
    Blackbox monitoring
    Alert thresholds
    Telemetry freshness

Do not solve MTTD by creating hundreds of alerts.

---

# 124. Scenario: MTTR Is High

Improve:

    Dashboards
    Logs
    Runbooks
    Ownership
    Dependency visibility
    Rollback capability
    Incident procedures

The problem may not be detection.

---

# 125. Scenario: Service Has Excellent Monitoring but High MTTR

Possible reasons:

    Too many dashboards
    Poor runbooks
    No ownership
    Difficult logs
    No correlation
    Slow deployment rollback

Monitoring quantity does not guarantee troubleshooting quality.

---

# 126. Scenario: You Need to Monitor a New Microservice

Production checklist:

    Metrics
    Logs
    SLO
    Dashboard
    Alerts
    Dependencies
    Runbook
    Ownership
    Security
    Retention
    Cost

Validate before production traffic.

---

# 127. Scenario: A New Service Has No SLO

Start with:

    User journey
    Critical operations
    Availability
    Latency

Then define:

    SLI
    SLO
    Error budget

Do not choose arbitrary targets without business context.

---

# 128. Scenario: Business Team Says "Checkout Is Slow"

Translate business language into technical signals:

    Checkout latency
    Payment latency
    Inventory latency
    Order latency
    Error rate
    Conversion rate

This is an important observability skill.

---

# 129. Scenario: Revenue Drops but Infrastructure Looks Healthy

Check:

    Business metrics
    Payment success
    Checkout completion
    Order creation
    External integrations

Technical infrastructure can remain healthy while business functionality fails.

---

# 130. Scenario: Monitoring Says Everything Is Healthy but Users Cannot Log In

Use blackbox testing.

Check:

    DNS
    TLS
    Authentication service
    Identity provider
    ALB
    Network

Also test the actual user journey.

---

# 131. Scenario: Synthetic Login Test Fails

Check:

    DNS
    TLS
    Authentication endpoint
    Credentials/test account
    Dependency
    Application logs

Separate:

    Test account problem

from:

    Production authentication failure.

---

# 132. Scenario: Observability Data Has Inconsistent Timestamps

Check:

    NTP/time synchronization
    Application timezone
    Collector parsing
    Elasticsearch timestamp mapping

Incident timelines depend on reliable time.

---

# 133. Scenario: Metrics and Logs Use Different Service Names

Example:

    Metrics:
    order-service

    Logs:
    orders

This makes correlation difficult.

Standardize:

    service name
    environment
    version

---

# 134. Scenario: One Application Has 50 Different Log Formats

This creates operational complexity.

Standardize:

    JSON
    Timestamp
    Level
    Service
    Environment
    Message
    Correlation ID

Centralized logging becomes much easier to query.

---

# 135. Scenario: A Debugging Question Cannot Be Answered From Existing Telemetry

Ask:

    What data is missing?
    Why was it missing?
    Could it be added safely?

Then improve instrumentation.

This is observability-driven engineering.

---

# 136. Scenario: Engineers Want More Logs to Solve Every Incident

Explain:

> More logs are not automatically better. We need structured, relevant and searchable logs with appropriate retention and correlation.

Use:

    Metrics for detection
    Logs for detail
    Traces for request path where available

---

# 137. Scenario: Engineers Want Every Possible Metric

Ask:

> What operational question does this metric answer?

If none:

    Do not collect it by default.

This protects cardinality and cost.

---

# 138. Scenario: Monitoring Has Too Much Data but Little Insight

This is a signal-design problem.

Improve:

    Service dashboards
    SLOs
    Alerts
    Ownership
    Correlation
    Retention
    Metric governance

The goal is useful information, not maximum telemetry.

---

# 139. Scenario: You Need to Choose Between Two Monitoring Designs

Compare:

    Reliability
    Scale
    Failure isolation
    Cost
    Security
    Operations
    DR
    Team capability

Do not choose solely because:

    "Tool X is popular."

---

# 140. Scenario: Observability Platform Has No DR

Start by defining:

    Critical telemetry
    RPO
    RTO

Then design:

    Configuration backup
    Data backup
    Replication
    Restore testing

Prioritize critical alerting and configuration recovery.

---

# 141. Scenario: Backup Exists but Restore Has Never Been Tested

State:

> The backup is not yet a proven recovery mechanism.

Perform:

    Restore test
    Integrity validation
    Application validation
    Recovery timing

Record:

    Actual RTO
    Actual data loss

---

# 142. Scenario: Observability Platform Must Survive an AZ Failure

Use:

    Multi-AZ nodes
    Failure-domain-aware scheduling
    Redundant collectors
    Adequate storage
    Independent notification path

Validate:

    One AZ failure

before calling it HA.

---

# 143. Scenario: Observability Platform Must Survive Region Failure

Evaluate:

    Cross-region storage
    Backup
    Secondary monitoring
    External alerts
    DR dashboards
    Configuration reproducibility

Not every telemetry type requires the same DR level.

---

# 144. Scenario: Cost Is More Important Than Long Retention

Do not delete data blindly.

Classify:

    Critical
    Operational
    Debug

Then design:

    Hot
    Warm
    Cold
    Archive
    Delete

based on access requirements.

---

# 145. Scenario: Security Requires Longer Audit Retention

Separate:

    Audit logs

from:

    Normal application logs.

Apply:

    Restricted access
    Encryption
    Retention
    Integrity
    Backup

---

# 146. Scenario: You Need to Reduce Observability Cost by 30%

Approach:

    1. Measure current cost
    2. Identify largest source
    3. Control telemetry at source
    4. Optimize retention
    5. Optimize queries
    6. Right-size infrastructure
    7. Re-measure

Do not reduce visibility blindly.

---

# 147. Scenario: Log Volume Is 5 TB/Day

Ask:

    Which services?
    Which log levels?
    Which event types?
    What percentage is useful?
    What is retention?
    What is compliance requirement?

Then:

    Reduce noise
    Filter
    Sample where safe
    Compress
    Tier storage

---

# 148. Scenario: Metric Volume Is Too High

Check:

    Series
    Labels
    Scrape interval
    Targets
    Exporters

The first question should often be:

> Which labels created the growth?

---

# 149. Scenario: Dashboard Query Takes 30 Seconds

Check:

    Time range
    Series count
    Regex
    Aggregations
    Joins
    Query frequency

Use:

    Recording rules
    Better selectors
    Smaller windows

---

# 150. Scenario: Alert Query Takes Too Long

A slow alert can delay incident detection.

Optimize:

    Query
    Recording rule
    Evaluation interval

Then verify:

    Alert latency

---

# 151. Scenario: Alert Is Correct but Notification Provider Is Down

Need:

    Secondary notification path
    Retry
    Escalation

For critical alerts, avoid a single notification dependency where business requirements justify redundancy.

---

# 152. Scenario: Monitoring Platform Loses Network Connectivity

Determine:

    Collection continues?
    Local buffering?
    Alerting continues?
    External monitoring works?

Network partitions should be part of resilience testing.

---

# 153. Scenario: Collector Has Backlog for Two Hours

Estimate:

    Current backlog
    Processing rate
    Incoming rate

If:

    processing rate < incoming rate

the backlog will continue growing.

Fix the throughput mismatch.

---

# 154. Scenario: Log Pipeline Has Backpressure

Identify bottleneck:

    Source
    Collector
    Logstash
    Elasticsearch
    Network

Then:

    Reduce load
    Increase processing
    Scale bottleneck
    Protect critical data

---

# 155. Scenario: One Dependency Causes a Cascade of Alerts

Root alert:

    DependencyDown

Secondary:

    ServiceLatency
    ServiceErrors
    QueueGrowth

Use inhibition/grouping to reduce noise while retaining the dependency alert.

---

# 156. Scenario: On-Call Wants One "Everything Is Broken" Alert

Explain that a single generic alert is not actionable.

Better:

    Service SLO
    Dependency
    Infrastructure
    Critical platform alerts

with:

    Ownership
    Severity
    Runbook

---

# 157. Scenario: You Need to Explain an Incident to Management

Use:

    Impact
    Duration
    Cause
    Mitigation
    Current status
    Prevention

Avoid:

    Deep PromQL
    Kubernetes internals
    Tool-specific details

unless asked.

---

# 158. Scenario: You Need to Explain the Same Incident to Engineers

Include:

    Timeline
    Metrics
    Logs
    Dependencies
    Deployment
    Root cause
    Evidence
    Preventive action

Tailor communication to the audience.

---

# 159. Scenario: Incident Is Caused by a Bad Configuration

Example:

    DB connection pool = 500
    DB limit = 200

Result:

    Connection exhaustion

Prevention:

    Configuration validation
    Capacity checks
    CI/CD policy
    Runtime monitoring

---

# 160. Scenario: Configuration Drift Causes Different Alert Thresholds

Example:

    Git:
    threshold = 5%

    Production:
    threshold = 10%

Fix:

    Configuration as code
    GitOps
    Drift detection

---

# 161. Scenario: A Monitoring Rule Works in Staging but Not Production

Compare:

    Metric names
    Labels
    Targets
    Traffic
    RBAC
    Data availability
    Configuration

Environment-specific assumptions are common causes.

---

# 162. Scenario: Production Has More Pods and Prometheus Becomes Slow

Check:

    Series growth
    Pod labels
    Scrape count
    Churn
    Query load

Dynamic Kubernetes environments can generate large series churn.

---

# 163. Scenario: Short-Lived Jobs Create Huge Metric Churn

Possible effects:

    Series creation/deletion
    TSDB overhead
    Memory pressure

Mitigate:

    Reduce unnecessary metrics
    Avoid unbounded labels
    Monitor job behavior
    Use appropriate metric strategy

---

# 164. Scenario: Every Pod Restart Creates New Series

Check labels containing:

    Pod name
    UID
    Instance identifiers

Some pod-level dimensions are useful, but excessive ephemeral dimensions increase churn.

---

# 165. Scenario: You Need Per-Request Debugging

Do not solve with metric labels.

Use:

    Structured logs
    Correlation ID
    Trace ID where tracing is available

This keeps metrics scalable.

---

# 166. Scenario: You Need to Find Which Endpoint Causes Errors

Use:

    route
    method
    status

Metric:

    Error rate by route

Then drill into:

    Logs
    Dependency
    Deployment

Use route templates rather than raw URLs to avoid cardinality.

---

# 167. Scenario: Raw URL Is Used as a Metric Label

Example:

    /users/123
    /users/456
    /users/789

This creates unbounded cardinality.

Prefer:

    /users/:id

or a normalized route label.

---

# 168. Scenario: Alert Is Based on Raw URL Labels

This can generate thousands of alerts.

Aggregate:

    service
    route
    method

rather than:

    individual request path.

---

# 169. Scenario: You Need to Monitor Java Application Performance

Check:

    Heap
    GC
    Threads
    CPU
    Request latency
    Errors
    Connection pools

Correlate:

    GC pauses
    latency
    memory

Do not monitor only JVM heap.

---

# 170. Scenario: Java Application Has High GC but CPU Is Normal

Investigate:

    Allocation rate
    Heap size
    GC frequency
    Pause time
    Object retention

Possible:

    Memory pressure
    Allocation spike
    Heap configuration

---

# 171. Scenario: Node.js Application Uses Increasing Memory

Check:

    Heap
    Event loop lag
    Requests
    Dependencies
    Application version

Potential:

    Memory leak
    Cache growth
    Unbounded objects

---

# 172. Scenario: Node.js Event Loop Lag Is High

CPU can be moderate while the event loop is blocked.

Investigate:

    Synchronous operations
    CPU-heavy code
    Large JSON processing
    Blocking libraries

Monitor:

    Event loop lag
    Request latency

---

# 173. Scenario: Python Application Has High Latency

Check:

    CPU
    Memory
    Thread/process model
    Database
    External APIs
    Application logs

Determine whether latency is:

    Application
    or
    dependency.

---

# 174. Scenario: Application Metrics Are Missing

Check:

    Instrumentation
    `/metrics`
    Port
    Service discovery
    Network
    RBAC
    Deployment

A complete monitoring chain is:

    Instrument
      |
      v
    Expose
      |
      v
    Discover
      |
      v
    Scrape
      |
      v
    Store
      |
      v
    Query

---

# 175. Scenario: Logs Exist but Are Not Structured

Migration strategy:

    Identify required fields
    Update application format
    Validate parser
    Roll out gradually
    Verify Kibana queries

Do not break production logging during the migration.

---

# 176. Scenario: You Need to Introduce SLOs to a Team

Start small:

    One critical service
    One availability SLO
    One latency SLO

Measure:

    Current baseline

Then:

    Agree target
    Define error budget
    Create alerts
    Review monthly

Avoid creating dozens of SLOs immediately.

---

# 177. Scenario: Team Says SLOs Are Too Strict

Discuss:

    Current reliability
    User expectations
    Business impact
    Cost of reliability

SLO is a service objective, not a number selected only by infrastructure engineers.

---

# 178. Scenario: Team Has 100% Availability Goal

Explain:

    100% is expensive and usually unrealistic.

SLOs provide:

    Explicit reliability target
    Error budget
    Tradeoff between reliability and delivery

The correct target depends on the service.

---

# 179. Scenario: Error Budget Is Exhausted

Potential policy:

    Pause risky releases
    Prioritize reliability
    Fix recurring incidents
    Improve capacity
    Review SLO assumptions

Do not automatically freeze every engineering activity unless the agreed policy says so.

---

# 180. Scenario: Error Budget Is Healthy but Incidents Are Increasing

Possible issue:

    SLO window hides recent instability.

Use:

    Shorter burn-rate windows
    Incident metrics
    Reliability trend

SLO compliance alone does not mean operational excellence.

---

# 181. Scenario: A Service Has Low Traffic but High Error Rate

Percentage can be misleading with tiny sample sizes.

Check:

    Request count
    Absolute failures
    Business impact

Example:

    1 failure out of 2 requests = 50%

but not necessarily a major incident.

Alert design should consider traffic volume.

---

# 182. Scenario: Service Has High Traffic but Small Error Percentage

Example:

    1% errors
    1 million requests

means:

    10,000 failed requests

This can be severe.

Always consider:

    Rate
    Volume
    User impact

---

# 183. Scenario: Error Rate Drops but Business Impact Remains

Possible:

    Error classification changed
    Users abandon before error
    Latency is too high
    Business transaction silently fails

Correlate:

    Technical metrics
    Business metrics

---

# 184. Scenario: Application Returns HTTP 200 but Business Operation Fails

Example:

    HTTP 200
    response:
    payment_status="failed"

HTTP success is not necessarily business success.

Create:

    Business success metrics

where appropriate.

---

# 185. Scenario: Monitoring Team Wants to Alert on Every Error Log

Do not automatically page.

Reasons:

    Expected errors
    Client errors
    Retries
    Known failures

Instead:

    Aggregate
    Rate-limit
    Classify
    Map to SLO/user impact

---

# 186. Scenario: Logs Are Full of 404s

Determine:

    Expected client behavior?
    Broken frontend?
    Bot traffic?
    Missing endpoint?
    Deployment issue?

Do not page simply because 404 exists.

---

# 187. Scenario: Logs Are Full of Connection Timeout Errors

Aggregate by:

    Service
    Dependency
    Error type

Then correlate with:

    Dependency latency
    Error rate
    Network

A single timeout is different from a timeout storm.

---

# 188. Scenario: You Need to Monitor a Third-Party Dependency

Monitor:

    Availability
    Latency
    Error rate
    Timeout rate

Also track:

    Your application's fallback behavior

If the third party fails, your system should ideally degrade safely.

---

# 189. Scenario: Dependency Fails but Circuit Breaker Protects Your Service

Internal:

    Dependency errors ↑

User-facing:

    Service remains available

Monitoring should capture both:

    Dependency failure
    Successful fallback

Otherwise the dependency incident can remain invisible.

---

# 190. Scenario: Monitoring Shows a Dependency Failure but Users Are Unaffected

Do not automatically page.

If:

    Fallback works
    SLO remains healthy

the alert may be:

    Warning
    or
    informational

Severity should reflect impact.

---

# 191. Scenario: A Critical Dependency Has No Fallback

Then dependency failure can directly become:

    User impact

It deserves stronger monitoring and alerting.

---

# 192. Scenario: Service Has No Dependency Timeout

Potential result:

    Requests wait indefinitely
       |
       v
    Thread pool exhausted
       |
       v
    Latency
       |
       v
    Outage

Monitor:

    Request duration
    Thread pool
    Dependency latency

Design explicit timeouts.

---

# 193. Scenario: Database Query Becomes Slow After Release

Correlate:

    Deployment
       |
       v
    Query latency
       |
       v
    DB CPU/I/O
       |
       v
    Application latency

Possible cause:

    Query regression
    Missing index
    Plan change

Use database-level evidence.

---

# 194. Scenario: Database Is Healthy but Application Connection Pool Is Exhausted

Investigate:

    Pool size
    Connection leak
    Long-running requests
    Timeout
    Pod count

This is an application-side bottleneck.

---

# 195. Scenario: Application Is Healthy but Kubernetes Service Has No Endpoints

Check:

    Pod labels
    Service selector
    Pod readiness

Commands:

    kubectl get svc
    kubectl get endpoints
    kubectl get pods --show-labels

A selector mismatch can cause complete traffic failure.

---

# 196. Scenario: Service Exists but Traffic Does Not Reach Pods

Check:

    Service
    Endpoints
    Readiness
    Network policy
    Port
    Target registration

Work from:

    Load balancer
    to
    service
    to
    pod.

---

# 197. Scenario: Kubernetes Events Show Failed Scheduling

Read the exact reason.

Possible:

    Insufficient CPU
    Insufficient memory
    Taint
    Affinity
    Topology constraint

Never infer from pod status alone.

---

# 198. Scenario: Deployment Has Available Replicas but Users Still Fail

Possible:

    Wrong Service selector
    Bad readiness
    ALB target issue
    Network policy
    Application routing

"Available replicas" is not equivalent to "user traffic works."

---

# 199. Scenario: Monitoring Dashboard Says All Pods Are Healthy

Yet:

    Users cannot access service.

Use:

    Blackbox test
    ALB
    DNS
    Application endpoint

This demonstrates monitoring-layer independence.

---

# 200. Scenario: Production Incident Starts During a Database Maintenance Window

Correlate:

    Maintenance event
    DB latency
    Application latency
    Errors
    Connection behavior

If correlation is strong:

    Confirm dependency impact
    Apply agreed mitigation
    Communicate

---

# 201. Scenario: No Recent Deployment but Incident Starts Suddenly

Investigate external changes:

    Dependency
    Infrastructure
    DNS
    Certificate
    Network
    Traffic
    Cloud service

No deployment does not mean no change.

---

# 202. Scenario: Traffic Doubles Suddenly

Check:

    Autoscaling
    CPU
    Memory
    Database
    Queue
    Connection pools
    Rate limits

Then ask:

> Is the traffic legitimate?

Potential causes:

    Campaign
    Client bug
    Retry storm
    Attack
    Misconfiguration

---

# 203. Scenario: Traffic Drops to Zero

This can be more serious than a traffic spike.

Check:

    DNS
    ALB
    Routing
    Application availability
    Client behavior
    Deployment

If business traffic is normally continuous, zero traffic is an important signal.

---

# 204. Scenario: ALB Traffic Drops but Pods Are Healthy

Check:

    DNS
    ALB listener
    Target registration
    Security/network
    External routing

This is an example of using multiple layers.

---

# 205. Scenario: Pods Receive No Traffic but ALB Targets Are Healthy

Check:

    Kubernetes Service
    Endpoints
    Ingress/ALB controller
    Service selector
    Ports

---

# 206. Scenario: Service Receives Traffic but Database Requests Are Zero

Possible:

    Application code path changed
    Cache serving responses
    Feature flag
    Queue behavior

Do not assume database failure.

---

# 207. Scenario: Cache Hit Rate Drops Suddenly

Potential:

    Cache eviction
    Key change
    TTL change
    Deployment
    Traffic pattern

Impact:

    Database load
    Latency

Correlate cache and database metrics.

---

# 208. Scenario: Database Load Increases Without Traffic Increase

Possible:

    Cache miss
    Query regression
    Retry
    Background job
    New endpoint behavior

Check:

    Query metrics
    Application version
    Cache hit rate
    Background workload

---

# 209. Scenario: Background Job Saturates the Database

Separate:

    User traffic
    Background traffic

Use:

    Job-specific metrics
    Database query attribution
    Scheduling

Protect user-facing workload.

---

# 210. Scenario: Scheduled Job Runs Every Hour and Causes Incidents

Correlation:

    Job starts
       |
       v
    DB load ↑
       |
       v
    API latency ↑

Solutions:

    Reschedule
    Throttle
    Batch
    Optimize query
    Resource isolation

Observability makes periodic patterns visible.

---

# 211. Scenario: Memory Leak Only Happens After 12 Hours

Use:

    Long-term memory trend

not:

    5-minute dashboard

Monitor:

    Memory slope
    Restart count
    Version

This is why retention and historical dashboards matter.

---

# 212. Scenario: Error Occurs Only During Peak Hours

Compare:

    Traffic
    Resource usage
    Queue
    DB
    Dependency

Look for:

    Capacity threshold

The system may be healthy under normal load but unstable near saturation.

---

# 213. Scenario: Incident Happens Only in One Region

Compare:

    Region
    Cluster
    Version
    Network
    Dependency
    Capacity

Use region labels to isolate the failure.

---

# 214. Scenario: Incident Happens Only in One Availability Zone

Compare:

    Node group
    Node health
    Network
    Workload placement

This can identify an infrastructure failure domain.

---

# 215. Scenario: One Node Has Much Higher Network Traffic

Investigate:

    Workload placement
    Traffic imbalance
    Logging
    Data transfer
    Network issue

A single node can become a hotspot.

---

# 216. Scenario: Network Latency Increases Between Services

Check:

    Network metrics
    Application latency
    DNS
    Node placement
    Cross-AZ traffic

Determine whether the increase is:

    Infrastructure
    or
    application/dependency.

---

# 217. Scenario: Cross-AZ Traffic Is Very High

Potential causes:

    Poor workload placement
    Service routing
    Stateful dependency location

Review:

    Topology
    Scheduling
    Architecture

Cost and latency can both be affected.

---

# 218. Scenario: Monitoring Queries Are Expensive During Incidents

This is dangerous because engineers query more during incidents.

Use:

    Recording rules
    Focused dashboards
    Query limits
    Prebuilt incident views

Observability should remain usable under stress.

---

# 219. Scenario: On-Call Dashboard Itself Causes Monitoring Load

Possible:

    Too many panels
    Fast refresh
    Large queries

Create:

    Lightweight incident dashboard

Separate from:

    Deep investigation dashboards.

---

# 220. Scenario: Incident Requires Searching 30 Days of Logs

Long time-range searches can be expensive.

Use:

    Narrow time window first
    Known service
    Known environment
    Error field
    Correlation ID

Expand only when necessary.

---

# 221. Scenario: Log Search Is Slow Because Everything Is Stored in One Index

Use appropriate:

    Index strategy
    Time-based organization
    Lifecycle policy

Balance:

    Search performance
    Shards
    Retention

---

# 222. Scenario: Log Retention Is Longer Than Required

Reduce:

    Hot storage

Move data to:

    Warm
    Cold
    Archive

based on requirements.

---

# 223. Scenario: Audit Data Must Never Be Deleted Before Retention Ends

Use:

    Retention controls
    Access restrictions
    Immutable storage where required
    Backup

Ensure operational teams cannot accidentally bypass policy.

---

# 224. Scenario: Observability Platform Is Used by Multiple Teams

Implement:

    RBAC
    Team dashboards
    Ownership
    Naming standards
    Quotas
    Cost attribution

Multi-tenancy requires governance.

---

# 225. Scenario: One Team's Bad Query Overloads Shared Prometheus

Protect:

    Query scope
    Recording rules
    Dashboard design
    Access controls

A shared platform needs workload isolation.

---

# 226. Scenario: One Team's Logs Fill Shared Elasticsearch

Implement:

    Per-team volume visibility
    Retention policies
    Ingestion controls
    Cost allocation

Then fix the source.

---

# 227. Scenario: New Service Is Deployed Without Monitoring

Production readiness should block or flag:

    Missing metrics
    Missing logs
    Missing ownership
    Missing alerts

Observability should be integrated into the delivery process.

---

# 228. Scenario: Developer Says "The Logs Are There, So Monitoring Is Done"

Explain:

    Logs alone do not provide:
    Fast detection
    Aggregated health
    SLOs
    Capacity visibility
    User-impact monitoring

Use multiple signals.

---

# 229. Scenario: Developer Says "CPU Is 20%, So Application Is Healthy"

Challenge the assumption.

Check:

    Error rate
    Latency
    Dependency
    Business metrics
    Queue
    Connections

Resource utilization is not equivalent to service health.

---

# 230. Scenario: Developer Says "Restart Fixed It"

Ask:

    What caused it?
    Did the symptom return?
    Was evidence lost?
    Why did restart work?

Treat restart as mitigation, not necessarily root cause.

---

# 231. Scenario: Production Is Healthy but Monitoring Is Broken

This is still an incident.

Why?

    Future incidents may become invisible.

Restore:

    Metrics
    Logs
    Alerts
    Dashboards

based on criticality.

---

# 232. Scenario: Monitoring Is Healthy but Alert Delivery Is Broken

Also a serious observability incident.

Check:

    Alertmanager
    Routing
    Receiver
    Notification provider

A detection system without notification can still fail operationally.

---

# 233. Scenario: Alert Delivery Works but Alert Content Is Wrong

Example:

    Alert says:
    "Database down"

but database is healthy.

Investigate:

    Query
    Labels
    Threshold
    Rule logic

False information is dangerous during incidents.

---

# 234. Scenario: Alert Fires for Wrong Environment

Check:

    Labels
    Variables
    Rule selector
    Routing

Explicitly filter:

    environment="prod"

where appropriate.

---

# 235. Scenario: Production Alert Is Missing a Runbook

Add:

    Dashboard
    Initial checks
    Commands
    Likely causes
    Mitigation
    Escalation

This improves MTTR.

---

# 236. Scenario: Incident Requires Manual Investigation Every Time

Automate:

    Dashboard
    Queries
    Runbook
    Diagnostic scripts
    Deployment correlation

Repeated incidents should become easier over time.

---

# 237. Scenario: Same Incident Happens Three Times

This is a reliability problem.

Perform:

    Root cause analysis
    Corrective action
    Alert improvement
    Capacity review
    Test coverage

Do not accept recurring incidents as normal operations.

---

# 238. Scenario: Postmortem Says "Human Error"

Go deeper.

Ask:

    Why was the mistake easy to make?
    Was there validation?
    Was rollback easy?
    Was configuration reviewed?
    Was automation missing?

Improve the system rather than only blaming the operator.

---

# 239. Scenario: Alert Was Missed Because Engineer Ignored It

Possible issue:

    Alert fatigue

Review:

    Alert quality
    Severity
    Volume
    Ownership

The solution may be alert redesign, not simply telling engineers to pay more attention.

---

# 240. Scenario: You Have Hundreds of Alerts but Few Incidents Detected

Measure:

    Alert-to-incident ratio
    False positives
    Missed incidents
    SLO coverage

Optimize toward useful detection.

---

# 241. Scenario: Monitoring Has No Documentation

Start with:

    Architecture
    Data flow
    Ownership
    Runbooks
    Recovery
    Configuration

Documentation is part of operational resilience.

---

# 242. Scenario: Only One Engineer Understands the Monitoring Stack

This is operational risk.

Improve:

    Documentation
    Cross-training
    Runbooks
    Standardization
    Automation

Avoid single-person dependency.

---

# 243. Scenario: Monitoring Platform Is Manually Configured

Migrate to:

    Git
    Code review
    CI validation
    GitOps

Start with:

    Alerts
    Dashboards
    Prometheus config
    Log pipelines

---

# 244. Scenario: Observability Change Needs Emergency Rollback

Have:

    Previous version
    Backup
    Git revision
    Rollback procedure

After rollback:

    Validate data
    Validate alerts
    Validate dashboards

---

# 245. Scenario: Monitoring Platform Upgrade Is Planned During Peak Hours

Avoid if possible.

Choose:

    Low-risk window

Ensure:

    Backup
    Rollback
    On-call availability
    Validation checklist

---

# 246. Scenario: You Need to Prove Monitoring Coverage

Build a matrix:

| Service | Metrics | Logs | Alerts | SLO | Dashboard | Owner |
|---|---|---|---|---|---|---|
| Orders | Yes | Yes | Yes | Yes | Yes | Team A |
| Payment | Yes | Yes | Yes | Yes | Yes | Team B |
| Inventory | Yes | Yes | Yes | No | Yes | Team C |

Identify gaps.

---

# 247. Scenario: You Need to Measure Observability Maturity

Assess:

    Coverage
    Signal quality
    Alert quality
    SLO adoption
    HA
    DR
    Security
    Cost
    Automation
    Incident outcomes

Maturity should be measured by operational capability, not tool count.

---

# 248. Scenario: Management Wants "100% Monitoring"

Clarify what it means.

Possible dimensions:

    Infrastructure
    Kubernetes
    Application
    Dependencies
    User journeys
    Business metrics
    Logs
    Alerts

No system can realistically monitor every possible failure automatically.

Define coverage based on risk and business impact.

---

# 249. Scenario: You Must Explain Why Observability Is a Platform Capability

Strong answer:

> Observability affects every production service, so it needs standards, automation, security, capacity planning and lifecycle management just like any other platform. Treating it as a shared platform improves consistency, reduces operational duplication and makes incident response faster.

---

# 250. Scenario: Final Production Incident

## Situation

A production microservices platform has:

    High API latency
    Increased 5xx
    Database latency
    Queue growth
    High retry rate

## Investigation

Do not treat these as five unrelated incidents.

Build the chain:

    Database latency
          |
          v
    API dependency waits
          |
          v
    Request latency
          |
          v
    Timeouts/errors
          |
          v
    Retries
          |
          v
    More database load
          |
          v
    Queue growth

## Mitigation

Depending on evidence:

    Reduce retries
    Protect database
    Scale safe components
    Disable expensive background work
    Roll back recent changes
    Apply traffic controls

## Validate

Check:

    Error rate
    p95/p99
    DB latency
    Retry rate
    Queue age
    Business success

## Prevention

    Timeouts
    Backoff
    Circuit breaker
    Capacity planning
    Better alerts
    Load testing
    Dependency SLO

This is the type of scenario where senior-level reasoning matters more than memorizing commands.

---

# 251. Scenario-Based Interview Answer Template

For almost any interview scenario:

## 1. Acknowledge the impact

> I would first confirm whether users are affected and determine the severity.

## 2. Establish timeline

> I would identify when the symptom started and correlate it with deployments, traffic or infrastructure changes.

## 3. Scope

> I would determine whether it affects one service, endpoint, pod, node, AZ, region or the entire platform.

## 4. Use signals

> I would correlate metrics, logs, Kubernetes state, infrastructure and dependencies.

## 5. Form a hypothesis

> Based on the evidence, I would identify the most likely cause.

## 6. Test

> I would validate that hypothesis using targeted checks.

## 7. Mitigate

> If there is a safe mitigation, I would restore service first.

## 8. Validate

> I would confirm recovery through user-facing metrics and business signals.

## 9. Prevent

> Finally, I would implement the root-cause fix and improve monitoring or architecture to prevent recurrence.

---

# 252. Scenario-Based Interview Mistakes

Avoid:

> "I will restart the pod."

> "I will increase memory."

> "I will check CPU."

> "I will check logs."

> "I will scale the cluster."

These answers are incomplete.

Instead say:

> "I will check X because it distinguishes hypothesis A from hypothesis B. If the evidence confirms A, I will mitigate using Y and then validate recovery using Z."

That demonstrates engineering reasoning.

---

# 253. Scenario-Based Command Cheat Sheet

## Kubernetes

    kubectl get pods -A
    kubectl get pods -o wide
    kubectl describe pod <pod>
    kubectl logs <pod>
    kubectl logs <pod> --previous
    kubectl get events -A --sort-by=.lastTimestamp
    kubectl top pod -A
    kubectl top node
    kubectl describe node <node>
    kubectl get svc
    kubectl get endpoints
    kubectl get nodes

## Linux

    top
    ps -ef
    free -h
    df -h
    df -i
    du -sh *
    ss -lntp
    journalctl -u <service>
    curl
    ping
    nslookup
    dig

Use commands as evidence-gathering tools, not as a memorized checklist.

---

# 254. Scenario-Based PromQL Cheat Sheet

## Request rate

    sum(rate(http_requests_total[5m]))

## Error rate

    sum(rate(http_requests_total{status=~"5.."}[5m]))
    /
    sum(rate(http_requests_total[5m]))

## p95 latency

    histogram_quantile(
      0.95,
      sum by (le) (
        rate(http_request_duration_seconds_bucket[5m])
      )
    )

## Service-level p95

    histogram_quantile(
      0.95,
      sum by (le, service) (
        rate(http_request_duration_seconds_bucket[5m])
      )
    )

## Target health

    up

## CPU

    rate(process_cpu_seconds_total[5m])

The exact metric names depend on instrumentation.

---

# 255. Scenario-Based ELK Investigation Path

Use:

    Application
       |
       v
    Container stdout
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

At every stage ask:

    Is data entering?
    Is data leaving?
    Is there a queue?
    Are there errors?
    Is latency increasing?

---

# 256. Scenario-Based EKS Investigation Path

Use:

    User
      |
      v
    DNS
      |
      v
    ALB
      |
      v
    Target
      |
      v
    Service
      |
      v
    Pod
      |
      v
    Application
      |
      v
    Dependency
      |
      v
    Database/Queue

This outside-in and dependency-aware approach prevents random troubleshooting.

---

# 257. Scenario-Based Observability Decision Tree

    Is user impacted?
       |
      Yes
       |
       v
    What signal changed?
       |
       +---- Error
       +---- Latency
       +---- Traffic
       +---- Saturation
       +---- Business metric
       |
       v
    Where?
       |
       +---- Service
       +---- Pod
       +---- Node
       +---- AZ
       +---- Region
       +---- Dependency
       |
       v
    What changed?
       |
       +---- Deployment
       +---- Config
       +---- Traffic
       +---- Dependency
       +---- Infrastructure
       |
       v
    Mitigate
       |
       v
    Validate
       |
       v
    Prevent

---

# 258. Final Scenario-Based Interview Mental Model

When the interviewer gives you a production problem, do not jump directly to a command.

Think:

    IMPACT
       ↓
    SCOPE
       ↓
    TIMELINE
       ↓
    SIGNALS
       ↓
    CORRELATION
       ↓
    HYPOTHESIS
       ↓
    EVIDENCE
       ↓
    MITIGATION
       ↓
    VALIDATION
       ↓
    ROOT CAUSE
       ↓
    PREVENTION

---

# 259. Final Key Takeaways

> Start with user impact.

> Establish the timeline before forming a root-cause theory.

> Scope the incident before changing anything.

> Metrics tell you where and when to look.

> Logs provide detailed evidence.

> Kubernetes state explains platform behavior.

> Dependencies often explain application symptoms.

> CPU and memory are only part of application health.

> Queue age can be more useful than queue depth.

> A successful deployment does not necessarily mean a healthy release.

> Restarting a pod is mitigation, not automatically root cause.

> High cardinality can damage the monitoring platform itself.

> Alerting must be tested end-to-end.

> Monitoring the monitoring system is mandatory for mature production environments.

> SLOs should represent meaningful user reliability.

> Business metrics can reveal failures technical metrics miss.

> A good incident answer explains why each investigation step matters.

> Always validate recovery using evidence.

> Every recurring incident should produce preventive engineering work.

---

# 260. Completion

This completes:

    20-Interview-Preparation/
        04-Scenario-Based.md

Next:

    05-Architecture-Questions.md
