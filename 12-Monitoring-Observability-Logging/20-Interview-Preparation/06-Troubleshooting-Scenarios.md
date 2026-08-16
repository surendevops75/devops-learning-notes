# Troubleshooting Scenarios

> Monitoring, Observability & Logging — Production Troubleshooting Interview Preparation
>
> Focus: systematic diagnosis, commands, evidence collection, root-cause analysis, remediation, prevention, production communication, Kubernetes/EKS, Prometheus, Grafana, ELK, Alertmanager, SLOs and real incident reasoning.

---

# 1. Production Troubleshooting Framework

Never start by randomly running commands.

Use:

    ALERT
      |
      v
    IMPACT
      |
      v
    SCOPE
      |
      v
    TIMELINE
      |
      v
    RECENT CHANGES
      |
      v
    METRICS
      |
      v
    LOGS
      |
      v
    EVENTS
      |
      v
    DEPENDENCIES
      |
      v
    ROOT CAUSE
      |
      v
    MITIGATION
      |
      v
    VALIDATION
      |
      v
    PREVENTION

---

# 2. First Five Questions During an Incident

Before deep investigation, answer:

1. What is broken?
2. Who is affected?
3. When did it start?
4. Is it getting worse?
5. What changed immediately before the incident?

This prevents spending 30 minutes investigating the wrong system.

---

# 3. Severity Assessment

Classify impact.

## Critical

- Complete service outage
- Major customer impact
- Data loss
- Security incident
- Severe SLO breach

## High

- Major degradation
- Large customer segment affected
- Significant error-rate increase

## Medium

- Partial degradation
- Limited functionality affected

## Low

- No current user impact
- Capacity or maintenance warning

Severity should be based on business impact, not merely infrastructure utilization.

---

# 4. Scenario: API Suddenly Returns 5xx Errors

## Symptoms

    HTTP 5xx ↑
    Success rate ↓

## Investigation

Check:

    ALB
    Kubernetes Service
    Pods
    Application logs
    Dependencies

Commands:

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

Check recent deployment:

```bash
kubectl rollout history deployment/<deployment> -n <namespace>
kubectl rollout status deployment/<deployment> -n <namespace>
```

## Likely causes

- Bad deployment
- Application exception
- Dependency failure
- Database connection failure
- Configuration change
- Readiness issue

## Mitigation

If deployment is clearly responsible:

```bash
kubectl rollout undo deployment/<deployment> -n <namespace>
```

Validate:

- Error rate
- Latency
- Pod health
- Business transaction success

---

# 5. Scenario: API Latency Suddenly Increased

## Symptoms

    p50 normal
    p95 ↑
    p99 ↑

## Investigation

Check:

- Application CPU
- Memory
- Database latency
- External APIs
- Connection pools
- Queue latency
- Recent deployment

PromQL example:

```promql
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m]))
  by (le, service)
)
```

## Important insight

Average latency may remain normal while tail latency becomes unacceptable.

Focus on:

    p95
    p99

---

# 6. Scenario: CPU Is 100% But Application Looks Healthy

Do not immediately restart.

Investigate:

```bash
top
ps -ef
```

In Kubernetes:

```bash
kubectl top pod -n <namespace>
kubectl top node
```

Check:

- Traffic increase
- Deployment version
- Thread/process behavior
- Garbage collection
- Infinite loop
- Expensive query
- HPA behavior

CPU utilization is a symptom.

Determine why CPU increased.

---

# 7. Scenario: Memory Usage Keeps Increasing

Symptoms:

    Memory ↑
    Memory ↑
    Memory ↑
    OOMKilled

Check:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl top pod <pod> -n <namespace>
```

Look for:

    reason: OOMKilled

Possible causes:

- Memory leak
- Increased traffic
- Large payloads
- Cache growth
- Incorrect memory limit
- Application regression

Do not simply increase the limit without identifying the cause.

---

# 8. Scenario: Pod Is CrashLoopBackOff

Run:

```bash
kubectl get pod <pod> -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

Check:

- Exit code
- Events
- Environment variables
- Secrets
- ConfigMaps
- Probes
- Image
- Resource limits

Most important command during repeated crashes:

```bash
kubectl logs <pod> --previous
```

---

# 9. Scenario: Pod Is Running But Requests Fail

A `Running` pod does not guarantee application availability.

Check:

```bash
kubectl get pod <pod> -o wide
kubectl get endpoints <service>
kubectl describe service <service>
```

Check:

- Readiness
- Service selector
- Target port
- Container port
- Application listener
- Network policy

A pod can be running but not ready.

---

# 10. Scenario: Readiness Probe Failing

Symptoms:

    Pod Running
    READY = 0/1

Check:

```bash
kubectl describe pod <pod>
```

Inspect Events.

Common causes:

- Wrong endpoint
- Wrong port
- Slow startup
- Dependency required by probe
- Authentication mismatch

Fix the probe or application behavior rather than blindly increasing thresholds.

---

# 11. Scenario: Liveness Probe Keeps Restarting Pod

Symptoms:

    Restarts ↑

Check:

```bash
kubectl describe pod <pod>
```

Look at:

    Liveness probe failed

Potential causes:

- Probe endpoint too strict
- Application overloaded
- Startup takes longer
- Deadlock
- Dependency incorrectly included in liveness

Important distinction:

> Liveness should answer whether the process needs restarting. Readiness should answer whether it should receive traffic.

---

# 12. Scenario: Deployment Is Stuck

Check:

```bash
kubectl rollout status deployment/<deployment>
kubectl get rs
kubectl get pods
kubectl describe deployment/<deployment>
```

Look for:

- Insufficient resources
- Image pull failure
- Readiness failure
- Scheduling failure
- PDB restrictions
- Quota
- Admission policies

---

# 13. Scenario: ImagePullBackOff

Check:

```bash
kubectl describe pod <pod>
```

Look for:

    Failed to pull image

Possible causes:

- Wrong image tag
- Image does not exist
- Registry authentication
- Network issue
- ECR permissions
- Image architecture mismatch

For ECR/EKS, verify IAM permissions and image URI.

---

# 14. Scenario: Pods Are Pending

Run:

```bash
kubectl get pods
kubectl describe pod <pod>
```

Look at Events.

Common causes:

- Insufficient CPU
- Insufficient memory
- Node selector mismatch
- Taints/tolerations
- Affinity rules
- Resource quota

Then inspect:

```bash
kubectl get nodes
kubectl describe node <node>
```

---

# 15. Scenario: Kubernetes Node Is NotReady

Check:

```bash
kubectl get nodes
kubectl describe node <node>
```

Investigate:

- Disk pressure
- Memory pressure
- PID pressure
- Kubelet
- Container runtime
- Network
- Node health

Check workloads affected by the node.

---

# 16. Scenario: Disk Usage Is 100% on a Node

Start with:

```bash
df -h
df -i
```

Then:

```bash
du -xh /var | sort -h | tail
```

Potential causes:

- Container logs
- Images
- Temporary files
- Application logs
- Deleted-but-open files

Check deleted open files where appropriate:

```bash
lsof +L1
```

Do not immediately delete random files from `/var`.

---

# 17. Scenario: Inodes Are Exhausted

Symptoms:

    Disk space available
    File creation fails

Check:

```bash
df -i
```

Find directories with many files.

Common causes:

- Temporary files
- Small log files
- Cache
- Application-generated files

Disk capacity and inode capacity are separate problems.

---

# 18. Scenario: Kubernetes Node Has Memory Pressure

Check:

```bash
kubectl describe node <node>
kubectl top node
```

Look for:

    MemoryPressure=True

Investigate:

- Pod requests/limits
- Memory leaks
- Overcommit
- System processes
- Evictions

---

# 19. Scenario: Pods Are Being Evicted

Check:

```bash
kubectl get events --sort-by=.lastTimestamp
kubectl describe pod <pod>
kubectl describe node <node>
```

Common causes:

- Memory pressure
- Disk pressure
- Ephemeral storage exhaustion

Identify the resource causing eviction before changing limits.

---

# 20. Scenario: HPA Is Not Scaling

Check:

```bash
kubectl get hpa
kubectl describe hpa <hpa>
kubectl top pods
```

Investigate:

- Metrics availability
- CPU/memory requests
- Target thresholds
- Min/max replicas
- Metrics API

HPA decisions depend on configured metrics and resource requests.

---

# 21. Scenario: HPA Scales Too Aggressively

Symptoms:

    Pods ↑ rapidly
    Cost ↑
    Instability

Investigate:

- Target utilization
- Traffic pattern
- CPU spikes
- Startup behavior
- Stabilization settings
- Application capacity

Scaling itself can create load if new pods are expensive to initialize.

---

# 22. Scenario: HPA Does Not Reduce Replicas

Possible causes:

- Current metrics still above target
- Stabilization window
- Minimum replicas
- Traffic remains elevated

Check:

```bash
kubectl describe hpa <hpa>
```

Do not assume HPA is broken until the decision inputs are understood.

---

# 23. Scenario: Service Returns 503

Investigate:

    ALB
      |
      v
    Target group
      |
      v
    Kubernetes Service
      |
      v
    Endpoints
      |
      v
    Pods

Check:

```bash
kubectl get endpoints <service>
kubectl get pods -l <selector>
```

If endpoints are empty, inspect readiness and selectors.

---

# 24. Scenario: ALB Shows Unhealthy Targets

Check:

- Health-check path
- Port
- Security groups
- Pod readiness
- Service target port
- Application listener

Kubernetes:

```bash
kubectl get svc
kubectl get endpoints
kubectl describe ingress
```

---

# 25. Scenario: DNS Resolution Fails Inside Kubernetes

Test:

```bash
kubectl exec -it <pod> -- nslookup <service>
```

Check CoreDNS:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system <coredns-pod>
```

Investigate:

- CoreDNS health
- Network policy
- DNS configuration
- Service name
- Namespace

---

# 26. Scenario: CoreDNS Is CPU Saturated

Symptoms:

    DNS latency ↑
    Application latency ↑

Check:

```bash
kubectl top pods -n kube-system
kubectl get deployment coredns -n kube-system
```

Investigate:

- Query volume
- Pod count
- DNS caching
- Node distribution
- CoreDNS resources

DNS can become a hidden dependency for many microservices.

---

# 27. Scenario: Service-to-Service Communication Fails

Check:

```bash
kubectl get svc
kubectl get endpoints
kubectl exec -it <pod> -- curl <service>:<port>
```

Investigate:

- DNS
- Service
- Endpoint
- Network policy
- Port
- Application listener

Use the network path rather than jumping directly to application code.

---

# 28. Scenario: NetworkPolicy Blocks Traffic

Symptoms:

    Pods healthy
    Connection timeout

Compare:

- Source namespace
- Destination namespace
- Pod labels
- Ports
- NetworkPolicy rules

A healthy pod can still be unreachable.

---

# 29. Scenario: ALB 5xx Increased

Separate:

    ALB-generated errors
    Target-generated errors

Investigate:

- Target health
- Application response
- Connection errors
- Timeouts
- Deployment
- Security groups

Do not assume all 5xx originate from the application.

---

# 30. Scenario: ALB 4xx Increased

Potential causes:

- Client behavior
- Authentication
- Routing
- Invalid requests
- WAF/rules where applicable

Determine whether the increase is expected or a regression.

---

# 31. Scenario: Database Latency Increased

Check application:

    API latency ↑

Then database:

- Connections
- CPU
- Memory
- I/O
- Slow queries
- Locks
- Storage
- Replication

Correlate timestamps.

Do not assume the database is the root cause simply because application latency increased.

---

# 32. Scenario: Database Connection Pool Exhausted

Symptoms:

    Connection timeout
    Requests waiting
    API latency ↑

Investigate:

- Pool size
- Active connections
- Idle connections
- Database max connections
- Long-running queries

Potential fix:

- Reduce connection leaks
- Tune pool
- Optimize queries
- Increase database capacity only when justified

---

# 33. Scenario: RabbitMQ Queue Depth Keeps Growing

Compare:

    Publish rate
    Consume rate

If:

    Publish > Consume

backlog grows.

Investigate:

- Consumer count
- Consumer errors
- Processing latency
- Downstream database
- Worker CPU
- Retries

Queue depth alone does not explain the cause.

---

# 34. Scenario: RabbitMQ Consumers Are Running But Queue Grows

Check:

- Consumer processing time
- Unacked messages
- Worker errors
- Database latency
- External dependencies

The workers may be alive but unable to process fast enough.

---

# 35. Scenario: Retry Storm

Symptoms:

    Dependency errors ↑
    Requests ↑ unexpectedly
    CPU ↑
    Latency ↑

Pattern:

    Dependency failure
         |
         v
       Retry
         |
         v
    More requests
         |
         v
    More dependency load
         |
         v
    Larger failure

Mitigation:

- Exponential backoff
- Jitter
- Retry limits
- Circuit breaker
- Fallback

---

# 36. Scenario: Cascading Failure

Example:

    Database slows
       |
       v
    API waits
       |
       v
    Threads/connection pools fill
       |
       v
    Requests timeout
       |
       v
    Clients retry
       |
       v
    Load increases

Look for the earliest abnormal dependency signal.

---

# 37. Scenario: Prometheus Target Is Down

Check:

```promql
up{job="my-service"}
```

Then inspect target status in Prometheus.

Investigate:

- Endpoint
- Service discovery
- Network
- Authentication/TLS
- Application metrics endpoint
- ServiceMonitor/PodMonitor if using Prometheus Operator

---

# 38. Scenario: Prometheus Scrape Is Timing Out

Check:

```promql
scrape_duration_seconds
```

Investigate:

- Target response time
- Metric volume
- Network latency
- Exporter performance
- Scrape timeout

A slow metrics endpoint can consume Prometheus resources.

---

# 39. Scenario: Prometheus Storage Is Filling

Check:

- Disk usage
- TSDB size
- Retention
- Active series
- Ingestion rate

Investigate cardinality:

```promql
prometheus_tsdb_head_series
```

Possible fixes:

- Reduce cardinality
- Adjust retention
- Increase storage
- Reduce unnecessary metrics
- Add long-term storage architecture

---

# 40. Scenario: Prometheus Memory Usage Is Very High

Potential causes:

- High active series
- High cardinality
- Large queries
- Recording rules
- High ingestion rate

Check:

    Active series
    Samples/sec
    Query duration
    Rule evaluation

Do not blindly increase memory without identifying the driver.

---

# 41. Scenario: Prometheus Query Is Very Slow

Investigate:

- Large time range
- High-cardinality aggregation
- Expensive regex
- Too many series
- Complex joins

Prefer:

- Recording rules
- Narrow label filters
- Appropriate aggregation
- Smaller query windows

---

# 42. Scenario: Grafana Dashboard Is Slow

Possible causes:

- Expensive PromQL
- Too many panels
- Large time range
- Elasticsearch query inefficiency
- Datasource latency
- Too much refresh frequency

Troubleshoot the datasource query first.

---

# 43. Scenario: Grafana Shows "No Data"

Check:

1. Datasource availability.
2. Query.
3. Time range.
4. Labels.
5. Metric existence.
6. Permissions.
7. Network.

Test the query directly in the datasource.

---

# 44. Scenario: Grafana Cannot Reach Prometheus

Check:

- Datasource URL
- DNS
- Network
- Authentication
- TLS
- Prometheus availability

From the Grafana environment, test connectivity where appropriate.

---

# 45. Scenario: Grafana Dashboard Worked Yesterday but Not Today

Investigate:

- Metric renamed
- Label changed
- Dashboard variable changed
- Datasource changed
- Prometheus retention
- Application deployment
- Query incompatibility

This is why dashboards should be version-controlled.

---

# 46. Scenario: Grafana Alert Is Not Firing

Check:

- Alert query
- Evaluation interval
- Condition
- Datasource
- Rule state
- Labels
- Notification routing

Separate:

    Rule does not fire

from:

    Rule fires but notification fails

---

# 47. Scenario: Alert Is Firing But No Notification Arrives

Check:

    Alertmanager
        |
        v
    Route
        |
        v
    Receiver
        |
        v
    Notification provider

Investigate:

- Routing labels
- Silences
- Inhibition
- Receiver configuration
- Provider API
- Network

---

# 48. Scenario: Alertmanager Has Too Many Alerts

Symptoms:

    Alert storm

Possible causes:

- Broad alert rules
- Dependency outage
- Missing inhibition
- Duplicate alerts
- Incorrect thresholds

Use grouping and inhibition carefully.

Do not silence everything permanently.

---

# 49. Scenario: Alert Fatigue

Symptoms:

    Engineers ignore alerts

Fix:

- Remove non-actionable alerts
- Tune thresholds
- Add severity
- Group related alerts
- Align pages with SLOs
- Add runbooks

Goal:

> When an engineer is paged, something requiring attention should actually be happening.

---

# 50. Scenario: SLO Is Being Violated But No Alert Fires

Investigate:

- SLI calculation
- Recording rule
- Alert expression
- Evaluation interval
- Alert routing
- Error-budget logic

Test the SLI independently before debugging the alert.

---

# 51. Scenario: Error Budget Is Decreasing Rapidly

Calculate burn rate.

Investigate:

- Error rate
- Traffic
- SLO window
- Recent deployments
- Dependency failures

High burn rate means the service is consuming reliability budget too quickly.

---

# 52. Scenario: ELK Logs Are Delayed

Architecture:

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

Find where latency occurs.

Check:

    Collector queue
    Logstash queue
    Elasticsearch indexing
    Kibana query

Do not restart the whole stack without identifying the bottleneck.

---

# 53. Scenario: Logstash CPU Is High

Potential causes:

- Complex grok
- Regex
- Huge events
- High ingestion
- Too many pipeline workers
- Expensive filters

Optimize processing and scale horizontally if necessary.

---

# 54. Scenario: Logstash Queue Is Growing

Compare:

    Input rate
    Processing rate
    Output rate

If output is slower:

    Elasticsearch may be bottlenecked.

If processing is slow:

    Filters may be expensive.

---

# 55. Scenario: Elasticsearch Disk Is Full

Immediate concern:

    Indexing may stop or cluster health may degrade.

Check:

- Disk usage
- Watermarks
- Index sizes
- Retention
- Replica count

Short-term mitigation may include removing unnecessary old data according to approved retention policy.

Long-term fix:

- Lifecycle policies
- Capacity
- Storage tiers
- Better retention

---

# 56. Scenario: Elasticsearch Cluster Is Yellow

Yellow usually indicates:

    Primary shards allocated
    Replica shards not fully allocated

Investigate:

```text
Cluster health
Shard allocation
Node availability
Disk watermarks
```

A yellow cluster may still serve data but has reduced redundancy.

---

# 57. Scenario: Elasticsearch Cluster Is Red

Red indicates one or more primary shards are unavailable.

Treat as serious.

Investigate:

- Failed nodes
- Disk
- Shard allocation
- Corruption
- Storage
- Recovery

Prioritize data availability and cluster stability.

---

# 58. Scenario: Elasticsearch Search Is Slow

Possible causes:

- Large time range
- Expensive query
- Too many shards
- High indexing load
- Disk I/O
- Insufficient memory
- Poor mappings

Investigate both:

    Query workload

and:

    Cluster workload

---

# 59. Scenario: Kibana Cannot Connect to Elasticsearch

Check:

- Elasticsearch health
- Kibana configuration
- DNS
- Network
- Authentication
- TLS
- Version compatibility

First prove Elasticsearch is healthy.

---

# 60. Scenario: Logs Are Missing From Kibana

Trace backwards:

    Kibana
      |
      v
    Elasticsearch
      |
      v
    Logstash
      |
      v
    Collector
      |
      v
    Application

Find the first broken stage.

---

# 61. Scenario: Logs Are Duplicated

Possible causes:

- Multiple collectors
- Duplicate pipeline
- Reprocessing
- Incorrect file offsets
- Application writing duplicate logs

Use event identifiers/timestamps to establish where duplication begins.

---

# 62. Scenario: Logs Have Wrong Timestamps

Check:

- Application timezone
- Collector parsing
- Logstash date filter
- Elasticsearch timestamp mapping

Prefer UTC internally for distributed systems.

---

# 63. Scenario: Structured Logs Cannot Be Parsed

Check:

- JSON validity
- Escaping
- Field types
- Parser configuration
- Nested objects

Avoid brittle regex parsing when applications can emit valid structured JSON directly.

---

# 64. Scenario: One Application Produces Huge Log Volume

Investigate:

- DEBUG logging
- Exception loops
- Request/response payload logging
- Retry loops
- Duplicate logging

Do not solve only by increasing Elasticsearch capacity.

Fix the producer.

---

# 65. Scenario: Application Logs Contain Secrets

Immediate actions:

1. Stop further exposure.
2. Rotate affected secrets where appropriate.
3. Restrict access.
4. Assess retention/backups.
5. Fix application logging.
6. Follow incident/security procedures.

Never assume deleting the visible log index fully removes all copies.

---

# 66. Scenario: Kubernetes Logs Disappear After Pod Restart

Container-local logs may be ephemeral.

Centralized collection is required for durable operational investigation.

Ensure collectors capture logs before pod/node lifecycle removes local data.

---

# 67. Scenario: Node Log Collector Is Down

Symptoms:

    Logs missing for one node

Check:

```bash
kubectl get pods -n <logging-namespace> -o wide
```

Identify whether the DaemonSet pod is absent or failing.

Investigate:

- Resource pressure
- Permissions
- Mounts
- Container runtime
- Collector configuration

---

# 68. Scenario: One Node Produces Far More Logs

Compare:

    Logs by node
    Logs by namespace
    Logs by service

Possible causes:

- Hot workload
- Error loop
- Misconfigured logging
- Traffic imbalance

This can overload both node disk and centralized logging.

---

# 69. Scenario: Prometheus Alerts Stop During Prometheus Failure

This is an architectural failure.

Ask:

> What monitors the monitoring system?

Use:

- HA Prometheus
- External checks
- Platform monitoring
- Independent critical alerts

Critical alerting should not have a single point of failure.

---

# 70. Scenario: Grafana Is Down During an Incident

Do not assume observability is completely unavailable.

Prometheus and alerting may still function.

Use:

- Prometheus UI
- Alertmanager
- Logs
- CLI
- External monitoring

Then restore Grafana.

---

# 71. Scenario: Elasticsearch Is Down but Application Is Healthy

Determine whether:

    Logging is lost

or:

    Application is impacted

Applications should ideally not become unavailable merely because centralized logging is temporarily unavailable.

Use asynchronous logging and buffering where appropriate.

---

# 72. Scenario: Logging Pipeline Failure Causes Application Failure

This is a design problem.

Avoid synchronous dependency:

    Application
        |
        | blocks on logging
        v
    Logging backend

Prefer:

    Application
        |
        v
    Local/asynchronous logging
        |
        v
    Collector
        |
        v
    Backend

---

# 73. Scenario: Monitoring Backend Is Consuming Excessive CPU

Identify component:

    Prometheus?
    Grafana?
    Logstash?
    Elasticsearch?

Then identify workload:

    Ingestion
    Query
    Rule evaluation
    Indexing
    Parsing

Scale only after identifying the bottleneck.

---

# 74. Scenario: Monitoring Costs Suddenly Increase

Check:

- Metric cardinality
- Log volume
- Retention
- Query volume
- Storage
- Ingestion spikes

Compare current behavior with previous baseline.

Look for recent:

    Deployment
    Instrumentation change
    Debug logging
    New labels

---

# 75. Scenario: New Deployment Causes Monitoring Cost Spike

Typical pattern:

    New metric label
        |
        v
    Series explosion
        |
        v
    Prometheus memory/storage ↑

Or:

    DEBUG logging
        |
        v
    Log volume ↑
        |
        v
    Elasticsearch cost ↑

Correlate cost changes with deployment metadata.

---

# 76. Scenario: Prometheus Has Millions of Series

Do not immediately increase memory.

Find:

- Top metrics
- Top labels
- Recent cardinality changes

Remove or redesign unnecessary dimensions.

---

# 77. Scenario: Prometheus Rule Evaluation Is Slow

Check:

- Number of rules
- Query complexity
- Evaluation interval
- Cardinality

Use recording rules to precompute frequently queried expressions.

---

# 78. Scenario: Recording Rule Is Wrong

Validate:

1. Raw metric.
2. Aggregation.
3. Labels.
4. Time window.
5. Recording rule output.

Do not trust a dashboard simply because it displays a number.

---

# 79. Scenario: Alert Fires During Every Deployment

Possible causes:

- Temporary traffic change
- Readiness behavior
- Replica reduction
- Scrape gaps
- Threshold too sensitive

Use deployment-aware alert design and appropriate windows.

Do not blindly silence alerts during deployments.

---

# 80. Scenario: Alert Fires Because One Pod Restarted

Ask:

> Does one restart represent user impact?

If not, use:

- Warning
- Trend alert
- Aggregation
- Restart-rate threshold

Paging should usually represent meaningful impact.

---

# 81. Scenario: Application Is Healthy But Monitoring Shows Errors

Validate the monitoring signal.

Check:

- Metric definition
- Exporter
- Scrape
- Query
- Labels
- Dashboard transformation

Observability itself can have bugs.

---

# 82. Scenario: Monitoring Says Healthy But Users Report Failure

Treat user reports as valid evidence.

Check:

- Blackbox monitoring
- DNS
- CDN/load balancer
- Authentication
- Region-specific traffic
- Business transaction

Possible problem:

> Internal health is green while the user journey is broken.

---

# 83. Scenario: Only One Region Has High Error Rate

Compare:

    Region A
    Region B
    Region C

Investigate:

- Regional deployment
- Dependency
- Network
- DNS
- Capacity
- Infrastructure

This is why region should be a standard dimension.

---

# 84. Scenario: Only One Availability Zone Is Failing

Compare:

    AZ-A
    AZ-B
    AZ-C

Check:

- Nodes
- Targets
- Network
- Capacity
- Application distribution

Kubernetes workloads should be distributed across failure domains where appropriate.

---

# 85. Scenario: Only One Version Has High Error Rate

Query by:

    version

Example:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
by (service, version)
```

If only the new version is failing:

    Rollback or stop rollout.

---

# 86. Scenario: Errors Started Immediately After Deployment

Strong hypothesis:

> Deployment-related regression.

But prove it.

Compare:

    Before deployment
    After deployment

Check:

- Error rate
- Latency
- Logs
- Dependencies
- Configuration

Then rollback if customer impact is significant and rollback is safe.

---

# 87. Scenario: Errors Started Hours After Deployment

Do not assume deployment caused it.

Investigate:

- Traffic increase
- Cache expiration
- Batch job
- Dependency change
- Resource exhaustion
- Delayed failure
- Data-dependent behavior

Timeline matters.

---

# 88. Scenario: No Deployment Occurred But Service Failed

Possible causes:

- Dependency failure
- Infrastructure issue
- Certificate expiry
- DNS issue
- Traffic spike
- Database problem
- Configuration change
- External provider failure

"Nothing changed" does not mean "nothing changed."

---

# 89. Scenario: Certificate Expired

Symptoms:

    TLS errors
    HTTPS unavailable

Check:

- Certificate expiry
- ALB/listener
- Ingress
- Secret
- Renewal automation

Monitor certificate expiry proactively.

---

# 90. Scenario: DNS Record Changed and Traffic Broke

Investigate:

- DNS record
- TTL
- Resolver behavior
- Target availability
- Regional routing

Correlate DNS changes with incident timeline.

---

# 91. Scenario: Sudden Traffic Spike

Metrics:

    Request rate ↑
    CPU ↑
    Latency ↑

Determine:

- Legitimate traffic
- Marketing event
- Client bug
- Bot traffic
- Attack

Then check autoscaling and capacity.

---

# 92. Scenario: Traffic Drops to Zero

Do not immediately assume application failure.

Check:

    DNS
    ALB
    Routing
    Client traffic
    Deployment
    Upstream service

A zero-traffic service can be either healthy or unreachable.

---

# 93. Scenario: Metrics Suddenly Drop to Zero

Potential causes:

- Application failure
- Exporter failure
- Prometheus scrape failure
- Label change
- Query problem
- Service discovery failure

Check:

```promql
up
```

before concluding that the application is healthy or unhealthy.

---

# 94. Scenario: Metric Name Changed After Deployment

Dashboard:

    No data

Application:

    Healthy

Cause:

    Instrumentation change

Fix:

- Version metric changes
- Update dashboards
- Update alerts
- Maintain compatibility where possible

---

# 95. Scenario: Labels Changed After Deployment

Example:

Before:

    app="orders"

After:

    service="orders"

Queries may return no data.

Treat metric schemas as APIs.

---

# 96. Scenario: Service Discovery Stops Working

Prometheus may show:

    Target disappeared

Investigate:

- Kubernetes API
- ServiceMonitor
- PodMonitor
- Labels
- Namespace
- RBAC

A discovery configuration change can silently remove targets.

---

# 97. Scenario: Exporter Is Down

Check:

```bash
kubectl get pods
kubectl logs <exporter>
kubectl describe pod <exporter>
```

Determine:

- Process failure
- Port issue
- Permission
- Resource limit
- Target backend unavailable

---

# 98. Scenario: Node Exporter Is Missing From One Node

Check DaemonSet:

```bash
kubectl get daemonset
kubectl get pods -o wide
```

Investigate:

- Taints
- Node selector
- Scheduling
- Resource constraints
- Pod failure

One missing exporter can create blind spots.

---

# 99. Scenario: Monitoring Data Has Gaps

Determine:

    Single target?
    Entire Prometheus?
    Entire cluster?

If one target:

    Exporter/application

If all targets:

    Prometheus/network/storage

If one time window:

    Infrastructure incident or scrape failure

---

# 100. Scenario: Prometheus Restarted and Historical Data Is Missing

Investigate:

- Persistent volume
- TSDB
- Retention
- WAL
- Pod replacement
- Storage mount

Prometheus local storage is not equivalent to a long-term durable observability architecture.

---

# 101. Scenario: Alertmanager Restarted

Verify:

- Configuration
- Routes
- Silences
- Notification delivery
- Cluster state where applicable

Then test a controlled alert.

---

# 102. Scenario: Duplicate Alerts Are Sent

Possible causes:

- HA Prometheus instances
- Duplicate alert rules
- Missing deduplication
- Multiple Alertmanagers
- Label differences

Compare alert labels and fingerprints.

---

# 103. Scenario: Alerts Are Grouped Too Aggressively

Symptoms:

    One notification hides multiple independent failures.

Review:

- Group-by labels
- Group wait
- Group interval

Alert grouping should reduce noise without hiding distinct incidents.

---

# 104. Scenario: Alert Inhibition Hides Important Alerts

Example:

    Node alert inhibits application alert

But application alert contains critical user impact.

Review inhibition rules carefully.

Inhibition should suppress secondary noise, not hide primary impact.

---

# 105. Scenario: Alert Is Permanently Silenced

Check:

- Silence owner
- Expiration
- Reason

Permanent silences are dangerous.

Every silence should have:

    Reason
    Scope
    Owner
    Expiration

---

# 106. Scenario: Logs Show an Error But Metrics Are Normal

Possible causes:

- Low-frequency error
- Non-critical path
- Logging bug
- Metric not instrumented
- Metric aggregation hides the error

Correlate both signals rather than assuming one is wrong.

---

# 107. Scenario: Metrics Show Errors But Logs Are Empty

Possible causes:

- Logging failure
- Different component generated metric
- Sampling
- Log collection gap
- Wrong namespace/index

Trace the signal path.

---

# 108. Scenario: Logs and Metrics Have Different Timestamps

Check:

- Time synchronization
- Timezone
- Timestamp parsing
- Collection delay
- Buffering

Use UTC and synchronized clocks where possible.

---

# 109. Scenario: Logs Arrive 10 Minutes Late

Find delay at each stage:

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

Measure event timestamp vs ingestion timestamp.

---

# 110. Scenario: Elasticsearch Indexing Lag Is High

Investigate:

- Ingestion rate
- Indexing rate
- CPU
- Disk I/O
- Heap
- Shards
- Refresh settings
- Cluster health

Do not increase shards blindly.

---

# 111. Scenario: Elasticsearch JVM Heap Is High

Check:

- Heap pressure
- Fielddata
- Query workload
- Shard count
- Mapping
- Aggregations

High heap usage can result from excessive shards and expensive queries.

---

# 112. Scenario: Elasticsearch Shards Are Unbalanced

Potential causes:

- Uneven shard sizes
- Allocation rules
- Hot indices
- Node capacity

Review shard allocation and workload distribution.

---

# 113. Scenario: Elasticsearch Recovery Is Taking Too Long

Consider:

- Shard size
- Disk speed
- Network
- Node count
- Concurrent recoveries
- Replica count

Recovery time is part of HA design.

---

# 114. Scenario: Log Search Works for Recent Data But Not Old Data

Check:

- Retention
- Lifecycle policy
- Index deletion
- Cold storage
- Search configuration

Understand where older data is stored before declaring it lost.

---

# 115. Scenario: Application Error Rate Increased Only During Peak Traffic

Possible causes:

- Capacity limit
- Connection pool
- Database
- Queue
- Thread pool
- Autoscaling lag

Compare:

    Traffic
    Capacity
    Latency
    Errors

---

# 116. Scenario: Autoscaling Happens Too Late

Possible causes:

- Slow metric window
- Wrong metric
- Long pod startup
- Insufficient baseline replicas

Improve:

- Scaling signal
- Startup time
- Min replicas
- Predictive capacity where justified

---

# 117. Scenario: Autoscaling Creates Instability

Pattern:

    Load ↑
      |
      v
    Scale ↑
      |
      v
    Cold pods start
      |
      v
    Load/latency changes
      |
      v
    Scale down
      |
      v
    Repeat

Tune:

- Stabilization
- Thresholds
- Startup probes
- Resource requests

---

# 118. Scenario: Pods Restart Only During Traffic Peaks

Investigate:

- Memory
- CPU
- Connection pools
- File descriptors
- Thread pools
- Timeouts

Traffic peaks can expose resource leaks that are invisible during normal load.

---

# 119. Scenario: File Descriptors Exhausted

Linux:

```bash
ulimit -n
lsof | wc -l
```

Application may show:

    Too many open files

Investigate:

- Connection leaks
- File leaks
- Socket count
- Process limits

Do not increase limits without finding the leak.

---

# 120. Scenario: Connection Count Keeps Increasing

Potential causes:

- Connection leak
- Slow database
- Requests not completing
- Pool misconfiguration

Correlate:

    Connection count
    Request latency
    Error rate

---

# 121. Scenario: Thread Pool Exhausted

Symptoms:

- Requests queue
- Latency ↑
- Timeouts

Investigate:

- Slow dependencies
- Database
- Thread pool size
- Blocking operations

Increasing threads can make a saturated dependency worse.

---

# 122. Scenario: External API Is Slow

Application:

    Dependency latency ↑
    API latency ↑

Check:

- Timeout
- Retry
- Circuit breaker
- Request rate
- Provider health

Mitigation may involve:

- Timeout
- Retry control
- Fallback
- Circuit breaker

---

# 123. Scenario: External Dependency Returns 429

This means rate limiting may be occurring.

Check:

- Request rate
- Retry behavior
- Client concurrency
- Provider limits

Do not retry immediately without backoff.

---

# 124. Scenario: External Dependency Returns 503

Determine:

- Provider outage
- Network failure
- Client overload
- Authentication
- Regional issue

Use dependency-specific dashboards where possible.

---

# 125. Scenario: One Microservice Is Causing Platform-Wide Latency

Find:

    Service error/latency
       |
       v
    Dependency calls
       |
       v
    Shared database/queue
       |
       v
    Other services

Look for common dependency saturation.

---

# 126. Scenario: One Service Consumes Excessive Memory

Check:

```bash
kubectl top pod
kubectl describe pod
```

Compare:

    Requests
    Limits
    Actual usage

Then investigate application behavior.

---

# 127. Scenario: One Namespace Is Consuming Most Cluster Resources

Aggregate:

    CPU by namespace
    Memory by namespace
    Pod count

Then inspect workloads.

Possible controls:

- ResourceQuota
- LimitRange
- Requests/limits
- Autoscaling

---

# 128. Scenario: Cluster Has Enough CPU But Pods Cannot Schedule

Possible cause:

    Fragmentation

Example:

    Total free CPU = 8 cores

but:

    Largest available node = 1 core

A pod requiring 2 cores cannot schedule.

Check scheduling constraints and node distribution.

---

# 129. Scenario: Pod Has Huge Memory Request and Cannot Schedule

Check:

```bash
kubectl describe pod <pod>
```

The scheduler uses requests, not actual current usage.

Optimize resource requests based on measured workload.

---

# 130. Scenario: Resource Limits Cause OOMKilled

Pattern:

    Application needs 1.5 GiB
    Limit = 1 GiB
       |
       v
    OOMKilled

Fix:

- Correct limit
- Investigate memory growth
- Optimize application

Do not remove limits blindly.

---

# 131. Scenario: Resource Requests Are Too Low

Symptoms:

- CPU throttling
- Scheduling issues
- Noisy neighbor behavior

Requests should represent realistic baseline needs.

---

# 132. Scenario: CPU Throttling Is High

Investigate:

- CPU limit
- Workload behavior
- Container runtime metrics
- Request/limit configuration

A high CPU limit can still be insufficient for bursty workloads.

---

# 133. Scenario: Monitoring Agent Uses Too Much CPU

Potential causes:

- Too many metrics
- Too many logs
- High scrape frequency
- Expensive parsing
- Excessive enrichment

Reduce unnecessary telemetry before simply allocating more CPU.

---

# 134. Scenario: Observability Agent Causes Node Pressure

Remember:

> Monitoring consumes resources too.

Set:

- Resource requests
- Resource limits
- Priority appropriately
- DaemonSet placement strategy

Monitor monitoring-agent resource consumption.

---

# 135. Scenario: Incident Started During Certificate Renewal

Check:

- Renewal logs
- Secret updates
- ALB/Ingress reload
- Certificate chain

Automation failures often create delayed outages.

---

# 136. Scenario: Service Fails After Secret Rotation

Investigate:

- New secret value
- Secret mount
- Pod restart
- Application authentication
- Dependency credentials

Compare:

    Old credential behavior
    New credential behavior

---

# 137. Scenario: ConfigurationMap Change Breaks Application

Check:

```bash
kubectl describe configmap <configmap>
kubectl describe pod <pod>
```

Determine:

- What changed?
- Which pods consumed it?
- Did pods reload automatically?
- Was restart required?

Configuration changes must be treated like deployments.

---

# 138. Scenario: Application Has Correct Image But Wrong Configuration

Image correctness does not guarantee runtime correctness.

Check:

- Environment variables
- Secrets
- ConfigMaps
- Command/args
- Mounted files

---

# 139. Scenario: Deployment Rollback Does Not Fix Incident

Possible causes:

- Database migration
- External dependency
- Configuration change
- Infrastructure issue
- Data corruption
- Certificate/DNS issue

Rollback only reverses the application version.

---

# 140. Scenario: Database Migration Caused Outage

Timeline:

    Deployment
       |
       v
    Schema change
       |
       v
    Query failure
       |
       v
    API errors

Recovery must consider database compatibility and rollback safety.

Never blindly roll back an application if the schema migration is not backward compatible.

---

# 141. Scenario: New Code Increased Database Query Volume

Metrics:

    Request rate normal
    DB query rate ↑
    DB latency ↑

Investigate:

- N+1 queries
- Missing cache
- Query regression
- Code path change

Application metrics and database metrics together reveal the problem.

---

# 142. Scenario: Cache Failure Causes Database Overload

Pattern:

    Cache hit rate ↓
       |
       v
    DB requests ↑
       |
       v
    DB latency ↑
       |
       v
    API latency ↑

This is a cascading dependency failure.

---

# 143. Scenario: Cache Hit Rate Is Low After Deployment

Check:

- Cache key
- TTL
- Serialization
- Namespace/version
- Application code

Do not immediately scale the database.

---

# 144. Scenario: Queue Backlog Causes User-Visible Delay

Metrics:

    Queue age ↑
    Queue depth ↑
    Processing time ↑

Define an SLI around:

    Time from event creation to processing

This is often more meaningful than queue depth alone.

---

# 145. Scenario: Scheduled Job Did Not Run

Check:

- CronJob
- Scheduler
- Job history
- Pod
- Application logs

Commands:

```bash
kubectl get cronjobs
kubectl get jobs
kubectl describe cronjob <name>
```

Also monitor expected execution time.

---

# 146. Scenario: CronJob Runs but Produces No Data

A successful process does not guarantee a successful business outcome.

Check:

- Exit code
- Records processed
- Output
- Destination
- Data freshness

Business metrics are essential for batch monitoring.

---

# 147. Scenario: Monitoring Shows Job Success But Users See Stale Data

Possible cause:

    Job success metric only measures process completion.

Improve monitoring with:

    Data freshness
    Record count
    Business validation

---

# 148. Scenario: Kubernetes Event Volume Is Extremely High

Check:

```bash
kubectl get events --sort-by=.lastTimestamp
```

Find recurring failures.

High event volume can be a symptom of:

- Crash loops
- Scheduling problems
- Probe failures
- Image errors

---

# 149. Scenario: API Server Requests Are Slow

Investigate:

- Client behavior
- API server load where visible
- Large list operations
- Controllers
- Watch behavior

Avoid polling the API excessively.

---

# 150. Scenario: One Controller Causes API Load

Possible causes:

- Aggressive reconciliation
- Large resource watches
- Frequent updates
- Buggy controller

Check controller metrics/logs and recent changes.

---

# 151. Scenario: Observability Data Is Incomplete After Network Partition

Identify:

    Collection loss
    Transmission loss
    Backend loss

Decide whether:

    Buffering

or:

    Local persistence

is required for critical telemetry.

---

# 152. Scenario: Monitoring Backend Is Unreachable From One Cluster

Check:

- DNS
- Routing
- Security groups
- Network policy
- TLS
- Endpoint availability

Compare with a healthy cluster.

---

# 153. Scenario: Cross-Region Monitoring Has High Latency

Consider:

- Network distance
- Query architecture
- Remote write
- Centralized collection

For real-time alerting, keep critical detection close to workloads when possible.

---

# 154. Scenario: Central Monitoring Is a Single Point of Failure

Mitigate through:

- Local alerting
- HA collectors
- Regional monitoring
- External checks
- Redundant storage

Centralization should not eliminate failure isolation.

---

# 155. Scenario: Monitoring Platform Itself Has an SLO

Example:

    Monitoring availability = 99.95%

Track:

- Data collection availability
- Query availability
- Alert delivery availability

Monitoring is production infrastructure and should have reliability objectives.

---

# 156. Scenario: Alert Delivery Is Delayed

Measure:

    Alert generated
       |
       v
    Alertmanager
       |
       v
    Notification provider
       |
       v
    Engineer

Find where latency occurs.

Alerting latency is an operational metric.

---

# 157. Scenario: On-Call Engineer Receives 500 Alerts

Do not treat each alert independently.

Group by:

    Incident
    Service
    Dependency
    Region

Find the primary failure.

Use inhibition to suppress predictable secondary symptoms.

---

# 158. Scenario: One Database Failure Creates 100 Alerts

Likely pattern:

    DB down
      |
      +--> Service A errors
      +--> Service B errors
      +--> Service C errors
      +--> Queue errors

Design alert hierarchy:

    Primary dependency failure
           |
           v
    Secondary alerts suppressed/grouped

---

# 159. Scenario: SLO Burn Rate Alert Fires Before Infrastructure Alert

This can be correct.

User impact:

    Error rate ↑

Infrastructure may still look healthy.

SLO alerting detects service reliability degradation rather than infrastructure saturation alone.

---

# 160. Scenario: CPU Is Normal But SLO Is Violated

Possible causes:

- Dependency latency
- Application bugs
- Database
- Network
- External provider
- Authentication

Infrastructure health is not equivalent to service health.

---

# 161. Scenario: CPU Is High But SLO Is Healthy

This may be:

- Normal traffic
- Efficient scaling
- Temporary workload

If no user impact exists, it may be a capacity warning rather than a page.

---

# 162. Scenario: Memory Is High But No OOM Occurs

Investigate:

- Working set
- Cache
- Heap
- Limits
- Trend

A high percentage alone does not prove an incident.

Trend and behavior matter.

---

# 163. Scenario: One Pod Is Slow While Others Are Healthy

Compare by pod:

    latency
    errors
    CPU
    memory
    node
    version

Possible causes:

- Bad node
- Noisy neighbor
- Application state
- Uneven traffic
- Connection issue

---

# 164. Scenario: One Node Has High Pod Latency

Compare all pods on that node.

If multiple unrelated services show degradation:

> Suspect node-level issue.

Check:

- CPU
- Memory
- Disk
- Network
- Runtime
- Node conditions

---

# 165. Scenario: One AZ Has Higher Latency

Compare:

    AZ
    Region
    Service
    Dependency

Potential causes:

- Network path
- Capacity
- Infrastructure
- Dependency localization

---

# 166. Scenario: One Region Has Lower Traffic

Possible:

- DNS routing
- Load-balancer health
- Regional outage
- Application capacity
- Client distribution

Traffic distribution itself is an observability signal.

---

# 167. Scenario: Service Has Zero Errors but Users Cannot Login

Investigate the entire user journey:

    DNS
      |
      v
    ALB
      |
      v
    Auth
      |
      v
    Identity provider
      |
      v
    Session/token

A service's internal error metric may not capture an external authentication failure.

---

# 168. Scenario: Monitoring Dashboard Looks Green During an Outage

Potential reasons:

- Wrong datasource
- Wrong environment
- Wrong labels
- Stale data
- Missing blackbox monitoring
- Dashboard not covering the failing path

Dashboards must be validated against real incidents.

---

# 169. Scenario: Alert Threshold Was Too High

Example:

    Error rate > 50%

But users are already impacted at:

    5%

Tune alerts around:

- SLO
- User impact
- Historical baseline

---

# 170. Scenario: Alert Threshold Was Too Low

Example:

    CPU > 50%

causes constant warnings.

Use:

- Baseline
- Duration
- Saturation
- SLO impact

Avoid alerting on normal fluctuations.

---

# 171. Scenario: Alert Uses a Single Instantaneous Value

Problem:

    CPU briefly spikes
       |
       v
    Page

Better:

    CPU high
    for sustained duration

Use time windows appropriate to the failure mode.

---

# 172. Scenario: Dashboard Has 100 Panels

Problems:

- Slow loading
- Hard to interpret
- Expensive queries

Create layered dashboards:

    Executive
       |
    Service
       |
    Dependency
       |
    Infrastructure
       |
    Detailed troubleshooting

---

# 173. Scenario: Runbook Does Not Help During Incident

A useful runbook should contain:

- Symptoms
- Commands
- Dashboard
- Expected result
- Decision tree
- Mitigation
- Rollback
- Escalation

Avoid:

> Check the application.

Be specific.

---

# 174. Scenario: Incident Has No Clear Owner

Every critical service should have:

    Team
    On-call
    Escalation
    Runbook

Ownership should be encoded in alert labels or service metadata.

---

# 175. Scenario: Monitoring Configuration Was Changed Manually

Risk:

- Drift
- No review
- Unknown owner
- Difficult rollback

Move configuration to:

    Git
      |
      v
    Review
      |
      v
    Deployment

---

# 176. Scenario: Alert Rule Deployment Breaks All Alerts

Validate rules before deployment.

Use:

- Syntax validation
- Unit tests where available
- Staging
- Controlled rollout

Observability configuration must have CI/CD too.

---

# 177. Scenario: Dashboard Query Causes Prometheus Overload

Symptoms:

    Query latency ↑
    CPU ↑
    Memory ↑

Fix:

- Optimize query
- Reduce time range
- Reduce cardinality
- Add recording rule
- Limit refresh frequency

Do not let a dashboard become a production DoS against the monitoring system.

---

# 178. Scenario: One User Query Is Extremely Expensive

Investigate:

- Query range
- Regex
- Cardinality
- Aggregations

Use query controls and dashboard standards.

---

# 179. Scenario: Elasticsearch Query Causes High CPU

Identify:

- Query
- Index
- Time range
- Aggregation

Optimize query and index design.

---

# 180. Scenario: Log Search Is Slow During Incident

Avoid repeatedly searching:

    Entire index
    Entire year
    All fields

Start narrow:

    service
    time range
    severity
    request/correlation ID

Then expand.

---

# 181. Scenario: Correlation ID Is Missing

Possible causes:

- Application doesn't generate it
- Proxy doesn't propagate it
- Service doesn't forward it
- Logging doesn't include it

Fix propagation across service boundaries.

---

# 182. Scenario: Request ID Changes Between Services

This breaks correlation.

Standardize:

    Incoming request ID
       |
       v
    Service A
       |
       v
    Service B
       |
       v
    Service C

Propagate the agreed identifier.

---

# 183. Scenario: Logs Cannot Be Correlated With Metrics

Ensure common dimensions:

    service
    environment
    version
    cluster
    namespace

For individual requests, use correlation IDs in logs rather than metric labels.

---

# 184. Scenario: One Request Is Slow Across 5 Services

Use the timeline:

    Gateway
      |
      v
    Service A
      |
      v
    Service B
      |
      v
    Service C
      |
      v
    Database

Identify where latency accumulates.

This is where distributed tracing would provide additional visibility when available.

---

# 185. Scenario: Tracing Is Available but Traces Are Missing

Check:

- Instrumentation
- Sampling
- Collector
- Network
- Backend
- Propagation

Do not assume missing traces mean missing requests.

---

# 186. Scenario: Sampling Hides Rare Errors

If traces are sampled too aggressively:

    Rare error
       |
       v
    No trace

Use targeted sampling or error-biased sampling where appropriate.

---

# 187. Scenario: Observability Pipeline Is Itself Generating Errors

Monitor:

- Collector failures
- Scrape failures
- Log drops
- Queue depth
- Backend failures
- Alert delivery failures

A production observability architecture needs internal telemetry.

---

# 188. Scenario: Monitoring Blind Spot Discovered

Example:

    One namespace has no metrics.

Do:

1. Identify why.
2. Assess current risk.
3. Restore visibility.
4. Add validation.
5. Prevent recurrence.

Monitoring gaps are reliability risks.

---

# 189. Scenario: Production Incident With No Alert

Ask:

- Was the signal collected?
- Was the SLI defined?
- Did the query detect it?
- Did the alert rule fire?
- Did Alertmanager route it?
- Did notification deliver?
- Did anyone acknowledge?

This decomposes "alert failure" into testable stages.

---

# 190. Scenario: Alert Fired But Engineer Responded Late

Potential causes:

- Notification delay
- Wrong team
- Alert fatigue
- Poor severity
- Missing runbook
- No escalation

Improve the entire incident path, not just the alert query.

---

# 191. Scenario: Incident Was Detected by Customers First

This is a strong signal that monitoring coverage is incomplete.

Add:

- Synthetic checks
- Business SLIs
- User-journey monitoring
- Better SLO alerts

---

# 192. Scenario: Postmortem Finds Monitoring Gap

Action items should be specific:

Bad:

> Improve monitoring.

Good:

> Add an SLO burn-rate alert for checkout availability and validate notification routing in staging.

---

# 193. Scenario: Production Incident During Deployment

Immediate questions:

1. Is user impact active?
2. Is rollout still progressing?
3. Can deployment be paused?
4. Is rollback safe?
5. Is database schema compatible?
6. Are dependencies healthy?

Then mitigate.

---

# 194. Scenario: Production Incident During Database Maintenance

Investigate:

- Connection failures
- Query latency
- Failover
- Replication
- Application retries

Compare timeline with maintenance event.

---

# 195. Scenario: Production Incident During Traffic Surge

Determine:

    Demand problem
    Capacity problem
    Dependency problem

Scale application only if the bottleneck is actually application capacity.

---

# 196. Scenario: Production Incident During AWS Infrastructure Event

Use:

- Application metrics
- Kubernetes metrics
- AWS service status/events where available
- Regional comparison
- Dependency health

Separate provider-level failure from application-level symptoms.

---

# 197. Scenario: Production Incident Is Resolved But Metrics Remain Abnormal

Do not close immediately.

Check:

- Backlog
- Queues
- Error budget
- Resource recovery
- Replication
- Data freshness

The service may be accepting requests while still recovering.

---

# 198. Scenario: Queue Continues Draining After Recovery

This is recovery behavior.

Monitor:

    Queue age
    Queue depth
    Processing rate

Estimate time to return to normal.

---

# 199. Scenario: Error Rate Recovered But SLO Still Shows Breach

Expected if historical SLO window includes previous errors.

Explain:

    Current health != historical SLO compliance

Error budget recovers based on the configured window.

---

# 200. Scenario: Incident Is Resolved but Root Cause Is Unknown

Do not invent certainty.

State:

> Service has been mitigated and validated, but root cause remains under investigation.

Collect:

- Timeline
- Logs
- Metrics
- Changes
- Evidence

Then perform post-incident analysis.

---

# 201. Senior-Level Troubleshooting Framework

When asked:

> "How would you troubleshoot this production issue?"

Answer:

## 1. Confirm impact

> I first confirm whether the alert represents real user impact and determine the affected scope.

## 2. Establish timeline

> I identify when the issue started and compare it with deployments, configuration changes and dependency events.

## 3. Check golden signals

> I examine traffic, errors, latency and saturation.

## 4. Narrow scope

> I segment by service, version, namespace, pod, node, AZ and region.

## 5. Correlate logs

> I inspect structured logs around the same timestamp using service and correlation identifiers.

## 6. Check dependencies

> I investigate databases, queues, external APIs, DNS and networking.

## 7. Mitigate

> If a recent deployment is strongly correlated and rollback is safe, I roll back or otherwise reduce user impact.

## 8. Validate

> I confirm recovery through metrics, logs and business success signals.

## 9. Prevent recurrence

> I add the appropriate alert, dashboard, test, capacity change or engineering fix.

---

# 202. Troubleshooting Decision Tree

    Alert
      |
      v
    Real impact?
      |
      +-- NO --> Validate alert
      |
      +-- YES
            |
            v
        Recent change?
            |
       +----+----+
       |         |
      YES        NO
       |         |
       v         v
    Compare    Check dependencies
    versions   and infrastructure
       |         |
       +----+----+
            |
            v
       Metrics
            |
            v
          Logs
            |
            v
       Events/config
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

# 203. Essential Kubernetes Troubleshooting Commands

## Pods

```bash
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

## Nodes

```bash
kubectl get nodes
kubectl describe node <node>
kubectl top nodes
```

## Services

```bash
kubectl get svc
kubectl get endpoints
kubectl describe svc <service>
```

## Deployments

```bash
kubectl get deployment
kubectl rollout status deployment/<deployment>
kubectl rollout history deployment/<deployment>
kubectl rollout undo deployment/<deployment>
```

## Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

## Resources

```bash
kubectl top pod
kubectl top node
```

---

# 204. Essential Linux Troubleshooting Commands

## CPU

```bash
top
ps -ef
uptime
```

## Memory

```bash
free -h
top
```

## Disk

```bash
df -h
df -i
du -xh /path | sort -h
```

## Processes

```bash
ps -ef
pgrep <process>
kill <PID>
```

## Network

```bash
ss -lntp
curl -v <url>
```

## Open files

```bash
lsof
lsof +L1
```

---

# 205. Prometheus Troubleshooting Checklist

Check:

- Prometheus process/pod
- Target health
- `up`
- Scrape duration
- Scrape failures
- Active series
- Ingestion rate
- Storage
- Query latency
- Rule evaluation
- Remote-write status

Key questions:

> Is Prometheus healthy?

> Is the target healthy?

> Is the metric present?

> Is the query correct?

> Is the alert correct?

---

# 206. Grafana Troubleshooting Checklist

Check:

- Grafana availability
- Datasource
- Authentication
- Network
- Query
- Time range
- Variables
- Dashboard version
- Alert state

Always separate:

    Dashboard problem

from:

    Datasource problem

---

# 207. ELK Troubleshooting Checklist

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
      |
      v
    Kibana

At every layer ask:

- Is data entering?
- Is data processing?
- Is data being stored?
- Can data be queried?

---

# 208. Alerting Troubleshooting Checklist

Trace:

    Metric
      |
      v
    PromQL
      |
      v
    Alert rule
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

Find the first broken stage.

---

# 209. Production Troubleshooting Mistakes

Avoid:

1. Restarting everything immediately.
2. Deleting pods without collecting evidence.
3. Increasing resources without finding the cause.
4. Silencing alerts permanently.
5. Assuming the latest deployment is always responsible.
6. Looking at only one metric.
7. Ignoring user impact.
8. Ignoring dependencies.
9. Searching all logs without narrowing the scope.
10. Making production changes without documenting them.

---

# 210. Incident Communication

During an incident communicate:

### What happened?

> Checkout error rate increased significantly.

### Impact?

> Approximately the checkout workflow is affected.

### When?

> Started at 14:05 UTC.

### Current action?

> Investigating the new deployment and payment dependency.

### Mitigation?

> Rollout paused; rollback in progress.

### Validation?

> Error rate is returning to baseline.

Good communication reduces operational confusion.

---

# 211. Example Senior-Level Incident Answer

Question:

> "Production checkout latency increased and users are reporting failures. What do you do?"

Answer:

> First, I would confirm the customer impact and establish when the issue started. I would check the checkout service's traffic, error rate, p95/p99 latency and saturation, then segment the metrics by version, pod, region and availability zone.
>
> Next I would compare the timeline against recent deployments, configuration changes and dependency events. I would inspect structured application logs using the service, version and correlation ID around the incident window.
>
> Because checkout usually depends on databases, payment providers and queues, I would check those dependencies for latency, errors, connection exhaustion and backlog.
>
> If the issue started immediately after a deployment and the new version shows significantly worse error or latency metrics, I would pause the rollout and perform a safe rollback while continuing to monitor recovery.
>
> After mitigation, I would validate not only infrastructure health but also checkout success rate, latency and user-facing SLOs.
>
> Finally, I would document the root cause and add the appropriate prevention, such as a canary check, SLO alert, regression test, dashboard or capacity improvement.

---

# 212. Scenario-Based Questions for Practice

## Kubernetes

1. Pod is CrashLoopBackOff.
2. Pod is Pending.
3. Node becomes NotReady.
4. Pods are evicted.
5. HPA does not scale.
6. HPA scales too aggressively.
7. Service has no endpoints.
8. ALB targets are unhealthy.
9. CoreDNS is failing.
10. NetworkPolicy blocks traffic.
11. ImagePullBackOff occurs.
12. Deployment is stuck.
13. Rollout causes errors.
14. Rollback does not fix the problem.
15. One node has high latency.

## Prometheus

16. Target is down.
17. Scrapes time out.
18. Storage fills.
19. Memory increases.
20. Queries are slow.
21. Cardinality explodes.
22. Rules are slow.
23. Alerts stop.
24. Metrics disappear.
25. Service discovery fails.

## Grafana

26. Dashboard shows no data.
27. Dashboard is slow.
28. Datasource fails.
29. Alert does not fire.
30. Alert fires but notification is missing.

## ELK

31. Logs disappear.
32. Logs are delayed.
33. Logstash CPU is high.
34. Logstash queue grows.
35. Elasticsearch disk fills.
36. Cluster turns yellow.
37. Cluster turns red.
38. Elasticsearch queries become slow.
39. Kibana cannot connect.
40. Logs are duplicated.

## Production

41. API 5xx increases.
42. API latency increases.
43. Database slows.
44. Queue grows.
45. External API returns 429.
46. External API returns 503.
47. Retry storm begins.
48. Cache fails.
49. Traffic spikes.
50. Traffic drops to zero.
51. Certificate expires.
52. Secret rotation breaks service.
53. Configuration change breaks application.
54. Region fails.
55. AZ fails.
56. SLO burns rapidly.
57. Alert storm begins.
58. Customers detect incident first.
59. Monitoring fails during outage.
60. Incident resolves but root cause is unknown.

---

# 213. Advanced Interview Scenarios

## Scenario A — Everything Looks Healthy, Users Report Failure

Reasoning:

    Infrastructure
       |
       v
    Kubernetes
       |
       v
    Application
       |
       v
    User journey

Investigate outside-in.

Check:

- DNS
- TLS
- ALB
- Authentication
- Business transaction
- External dependencies

---

## Scenario B — CPU Is Normal, Latency Is High

Investigate:

    Database
    Network
    External API
    Queue
    Locks
    Connection pools

Do not equate low CPU with healthy service.

---

## Scenario C — Error Rate Is Normal, Availability Is Poor

Possible:

- Requests never reach application
- DNS
- Load balancer
- Authentication
- Routing

Use blackbox monitoring.

---

## Scenario D — Prometheus Is Healthy but Metrics Are Missing

Investigate:

    Service discovery
    Target
    Exporter
    Metric name
    Labels
    Query

The Prometheus process itself can be healthy while collection is broken.

---

## Scenario E — Elasticsearch Is Healthy but Logs Are Missing

Trace backwards:

    Kibana
      |
      v
    Elasticsearch
      |
      v
    Logstash
      |
      v
    Collector
      |
      v
    Application

---

# 214. Root Cause Analysis Framework

Use:

## What happened?

Describe the failure.

## Why did it happen?

Identify technical cause.

## Why was it possible?

Identify design weakness.

## Why wasn't it detected earlier?

Identify monitoring gap.

## Why wasn't recovery faster?

Identify operational weakness.

## What prevents recurrence?

Define specific actions.

---

# 215. Five Whys Example

Problem:

> Checkout errors increased.

Why?

> Payment calls timed out.

Why?

> Payment provider response latency increased.

Why?

> Client retry policy multiplied requests.

Why?

> Retry policy had no effective backoff/jitter.

Why?

> Resilience configuration was not reviewed for failure scenarios.

Root prevention:

- Retry policy standard
- Load test
- Circuit breaker
- Dependency SLO
- Alert on retry rate

---

# 216. Troubleshooting Priorities

During an active incident:

1. Protect users.
2. Stop further damage.
3. Restore service.
4. Preserve evidence.
5. Find root cause.
6. Prevent recurrence.

Do not spend 45 minutes proving the root cause while users remain impacted if a safe mitigation is available.

---

# 217. Mitigation vs Root Cause

Example:

    Restart pods

may restore service temporarily.

But root cause could be:

    Memory leak

Therefore:

    Restart = mitigation
    Memory leak fix = root-cause remediation

Interviewers expect you to distinguish these.

---

# 218. Evidence-Based Troubleshooting

Good:

> Error rate increased immediately after version v42 deployment, and only v42 pods show elevated latency.

Weak:

> I think the deployment caused it.

Always support hypotheses with:

    Metrics
    Logs
    Events
    Timeline

---

# 219. Avoid Confirmation Bias

If you suspect deployment:

Check:

- Previous version
- New version
- Dependency health
- Infrastructure
- Traffic

If all versions are failing, deployment is probably not the complete explanation.

---

# 220. Final Troubleshooting Mental Model

Think:

    USER IMPACT
         |
         v
      SCOPE
         |
         v
      TIMELINE
         |
         v
      CHANGE
         |
         v
     GOLDEN SIGNALS
         |
         v
      SEGMENT
         |
         v
       LOGS
         |
         v
    DEPENDENCIES
         |
         v
       EVENTS
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

---

# 221. Final Interview Takeaways

> Do not troubleshoot by guessing. Troubleshoot by evidence.

> Start with user impact and scope.

> Establish the timeline before forming a strong hypothesis.

> Always check recent changes, but do not assume they are the cause.

> Use metrics to identify abnormal behavior.

> Use logs to understand detailed failures.

> Use Kubernetes events to understand scheduling and lifecycle problems.

> Use dependency metrics to find cascading failures.

> Segment metrics by service, version, namespace, pod, node, AZ and region.

> A Running pod is not necessarily a healthy pod.

> CPU and memory are symptoms; determine why they changed.

> Prometheus being healthy does not mean all targets are healthy.

> Grafana being healthy does not mean its datasource is healthy.

> Elasticsearch being healthy does not mean logs are flowing.

> Alertmanager can be healthy while notification delivery is broken.

> SLOs should guide user-impact alerting.

> Blackbox monitoring catches failures internal monitoring can miss.

> Rollback is mitigation, not automatically root cause.

> Increasing resources is not a root-cause analysis.

> Restarting a service may hide the actual problem.

> During incidents, restore service first when a safe mitigation exists.

> After recovery, perform root-cause analysis and prevent recurrence.

> The strongest senior-level answer connects symptoms across the entire architecture rather than troubleshooting components in isolation.

---

# 222. Completion

This completes:

    20-Interview-Preparation/
        06-Troubleshooting-Scenarios.md

Next:

    07-Production-Incident-Scenarios.md
