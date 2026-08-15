# 20 - Interview Preparation
# 08 - Mock Interview

> Monitoring, Observability & Logging — Complete DevOps / DevSecOps Mock Interview
>
> This file is designed for realistic interview practice from basic concepts through senior production scenarios. Answers are structured for spoken interviews: clear, technically accurate, production-oriented and focused on reasoning rather than memorized definitions.

---

# 1. How to Use This Mock Interview

Do not read the answer first.

For every question:

1. Read the question.
2. Pause.
3. Answer aloud.
4. Explain your reasoning.
5. Mention commands or metrics when relevant.
6. Give a production example.
7. Then compare your answer with the model answer.

A strong DevOps interview answer usually follows:

    WHAT
      |
      v
    WHY
      |
      v
    HOW
      |
      v
    PRODUCTION EXAMPLE
      |
      v
    TROUBLESHOOTING

---

# 2. Interview Answer Framework

For technical questions:

> "X is ..."

Then:

> "It is useful because ..."

Then:

> "In production, I would configure/use it by ..."

Then:

> "If it fails, I would troubleshoot it by ..."

Then:

> "For example, in an EKS environment ..."

This structure prevents answers from becoming definitions only.

---

# 3. Self-Introduction — Monitoring Focus

## Question

Tell me about your experience with monitoring and observability.

## Model Answer

> I have worked with Prometheus, Grafana and ELK for monitoring, visualization and centralized logging in Kubernetes and AWS environments. I use Prometheus for infrastructure and application metrics, Grafana for dashboards and visualization, and ELK for centralized application and system logs. My approach is to monitor the golden signals such as latency, traffic, errors and saturation, then correlate those metrics with logs and deployment or infrastructure changes during incidents. In Kubernetes environments, I also monitor pod health, node capacity, resource utilization, application availability and workload behavior. I focus not only on collecting telemetry but also on alerting, troubleshooting, SLOs and production incident response.

---

# 4. Basic Round — Monitoring Fundamentals

## Q1. What is monitoring?

### Model Answer

> Monitoring is the continuous collection and evaluation of system signals such as metrics, logs, availability and resource utilization to detect abnormal behavior and operational problems.

Examples:

- CPU
- Memory
- Disk
- Network
- Request rate
- Error rate
- Latency
- Pod restarts
- Database connections

Monitoring primarily answers:

> "Is something wrong?"

---

## Q2. What is observability?

### Model Answer

> Observability is the ability to understand the internal state and behavior of a system from its externally available telemetry, mainly metrics, logs and traces.

Monitoring tells me that:

> API latency increased.

Observability helps answer:

> Which endpoint, version, pod, dependency or database query caused the latency increase?

---

# 5. Monitoring vs Observability

## Q3. What is the difference?

### Model Answer

> Monitoring focuses on predefined known failure conditions and alerts. Observability gives deeper context for investigating unknown or complex failures.

Example:

Monitoring:

```text
HTTP 5xx > 5%
```

Observability:

```text
5xx increased
    |
    +--> checkout service
    |
    +--> version v2.4
    |
    +--> database latency increased
    |
    +--> deployment occurred 5 minutes earlier
```

---

# 6. Metrics, Logs and Traces

## Q4. What are the three pillars of observability?

### Model Answer

- Metrics
- Logs
- Traces

### Metrics

Numerical measurements over time.

Example:

```text
http_requests_total
http_request_duration_seconds
node_memory_utilization
```

### Logs

Detailed event records.

Example:

```text
2026-08-15 10:10:22 ERROR payment failed
```

### Traces

Follow a request across distributed services.

Example:

```text
ALB
 |
 v
Order Service
 |
 +--> Inventory
 |
 +--> Payment
 |
 +--> Database
```

---

# 7. Golden Signals

## Q5. What are the four golden signals?

### Model Answer

1. Latency
2. Traffic
3. Errors
4. Saturation

### Latency

How long requests take.

### Traffic

How much demand the system receives.

### Errors

How many requests fail.

### Saturation

How close the system is to resource or capacity limits.

---

# 8. Why Golden Signals Matter

## Q6. Give a production example.

Suppose:

```text
Traffic       ↑ 3x
Latency       ↑
Errors        ↑
CPU            ↑
DB connections ↑
```

This suggests a capacity/dependency problem.

If:

```text
Traffic normal
CPU normal
Latency ↑
DB latency ↑
```

The database becomes a stronger hypothesis.

---

# 9. Metrics

## Q7. What is a metric?

### Model Answer

> A metric is a numerical measurement collected over time that represents some property of a system.

Examples:

```text
CPU usage
Memory usage
Request rate
Error rate
Latency
Queue depth
```

---

# 10. Metric Types

## Q8. What are Prometheus metric types?

### Model Answer

Prometheus supports:

- Counter
- Gauge
- Histogram
- Summary

---

# 11. Counter

## Q9. What is a counter?

### Model Answer

> A counter is a monotonically increasing value that normally only increases or resets when the process restarts.

Example:

```text
http_requests_total
```

Use:

```promql
rate(http_requests_total[5m])
```

Do not directly interpret the raw counter value as requests per second.

---

# 12. Gauge

## Q10. What is a gauge?

### Model Answer

> A gauge represents a value that can increase or decrease.

Examples:

```text
CPU utilization
Memory usage
Queue depth
Active connections
```

---

# 13. Histogram

## Q11. Why are histograms useful?

### Model Answer

> Histograms collect observations into buckets and are useful for calculating distributions such as request latency and percentiles.

Example:

```promql
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m]))
  by (le)
)
```

This estimates p95 latency.

---

# 14. Summary vs Histogram

## Q12. Difference?

### Model Answer

> Summaries calculate quantiles on the client side, while histograms expose buckets that can be aggregated on the Prometheus side. Histograms are generally more flexible for distributed aggregation.

---

# 15. Prometheus

## Q13. What is Prometheus?

### Model Answer

> Prometheus is an open-source monitoring and time-series database system designed primarily for collecting and querying metrics.

It uses:

- Pull-based scraping
- PromQL
- Time-series storage
- Service discovery
- Alerting integration

---

# 16. Prometheus Architecture

## Q14. Explain Prometheus architecture.

```text
Applications
     |
     v
 /metrics
     |
     v
Prometheus
     |
     +--> TSDB
     |
     +--> PromQL
     |
     v
Alert Rules
     |
     v
Alertmanager
     |
     v
Notifications

Prometheus
     |
     v
Grafana
```

---

# 17. Pull vs Push

## Q15. Why does Prometheus generally use pull?

### Model Answer

> Pull allows Prometheus to control scrape intervals, discover targets and determine whether a target is reachable. It also makes monitoring infrastructure relatively simple.

Applications expose:

```text
/metrics
```

Prometheus scrapes them.

---

# 18. Pushgateway

## Q16. When would you use Pushgateway?

### Model Answer

> Pushgateway can be useful for short-lived batch jobs that cannot remain available long enough for Prometheus to scrape them.

It should not normally be used as a replacement for standard service scraping.

---

# 19. PromQL

## Q17. What is PromQL?

### Model Answer

> PromQL is Prometheus's query language used to select, aggregate and calculate information from time-series metrics.

Example:

```promql
rate(http_requests_total[5m])
```

---

# 20. PromQL — Error Rate

## Q18. How do you calculate HTTP 5xx rate?

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

Multiply by 100 if a percentage is required.

---

# 21. PromQL — CPU

## Q19. Give a CPU utilization example.

```promql
100 *
(
  1 -
  avg by(instance)
  (rate(node_cpu_seconds_total{mode="idle"}[5m]))
)
```

Always validate the exact labels exposed by your exporter.

---

# 22. Prometheus Service Discovery

## Q20. What is service discovery?

### Model Answer

> Service discovery allows Prometheus to dynamically discover scrape targets rather than maintaining a static list manually.

In Kubernetes it can discover:

- Pods
- Services
- Nodes
- Endpoints
- ServiceMonitors/PodMonitors depending on the monitoring stack

---

# 23. Exporters

## Q21. What is an exporter?

### Model Answer

> An exporter converts metrics from a system into a format Prometheus can scrape.

Examples:

- Node Exporter — Linux
- Blackbox Exporter — endpoint probing
- Database exporters
- Application exporters

---

# 24. Node Exporter

## Q22. What does Node Exporter monitor?

### Model Answer

It exposes Linux host metrics such as:

- CPU
- Memory
- Disk
- Filesystem
- Network
- Load

---

# 25. Prometheus in Kubernetes

## Q23. What would you monitor in EKS?

### Model Answer

I would monitor:

### Cluster

- Nodes
- API availability
- Scheduling
- Capacity

### Nodes

- CPU
- Memory
- Disk
- Network

### Pods

- CPU
- Memory
- Restarts
- OOMKilled
- Readiness
- Liveness

### Applications

- Request rate
- Error rate
- Latency

### Dependencies

- Database
- Queue
- External APIs

---

# 26. Prometheus Target Is DOWN

## Q24. What do you check?

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get endpoints -A
kubectl get servicemonitor -A
```

Then verify:

- `/metrics`
- Network connectivity
- Port
- Labels
- Service discovery
- Authentication/TLS if applicable

---

# 27. Prometheus Has No Data

## Q25. Troubleshoot.

Check:

1. Target exists.
2. Target is UP.
3. Metric endpoint responds.
4. Service discovery works.
5. Metric name is correct.
6. Labels are correct.
7. Query time range is correct.

---

# 28. High Prometheus Memory

## Q26. What could cause it?

- High cardinality
- Too many targets
- Long retention
- Large scrape volume
- Expensive queries
- New labels

Strong answer:

> I would check active series and identify which metrics or labels are contributing to cardinality before simply increasing memory.

---

# 29. Cardinality

## Q27. What is high cardinality?

### Model Answer

> Cardinality is the number of unique time series generated by metric label combinations.

Bad:

```text
user_id
request_id
transaction_id
```

These can create huge numbers of unique series.

Better:

```text
service
method
status
environment
```

with controlled values.

---

# 30. Grafana

## Q28. What is Grafana?

### Model Answer

> Grafana is a visualization and observability platform used to query data sources and build dashboards, alerts and operational views.

It can visualize:

- Prometheus
- Elasticsearch
- Loki
- SQL databases
- Other supported sources

---

# 31. Grafana + Prometheus

## Q29. Explain the integration.

```text
Application
    |
    v
Prometheus
    |
    v
Grafana
    |
    v
Dashboard
```

Grafana queries Prometheus using PromQL.

Prometheus remains the metrics source.

---

# 32. Grafana Dashboard Design

## Q30. What should a production dashboard contain?

Recommended structure:

```text
Service Health
   |
   +--> Availability
   +--> Traffic
   +--> Error Rate
   +--> Latency
   |
   v
Resources
   |
   +--> CPU
   +--> Memory
   +--> Pods
   |
   v
Dependencies
   |
   +--> DB
   +--> Queue
   +--> External APIs
   |
   v
Deployments
```

---

# 33. Bad Dashboard

A dashboard with 100 unrelated panels is difficult during incidents.

Good dashboard:

- Starts with health
- Shows golden signals
- Shows dependencies
- Shows recent deployments
- Links to logs/runbooks

---

# 34. Grafana Shows No Data

## Q31. Troubleshooting

Check:

- Datasource
- Connectivity
- Query
- Labels
- Time range
- Refresh interval
- Prometheus health

Then test the same query directly in Prometheus.

---

# 35. ELK

## Q32. What is ELK?

### Model Answer

ELK stands for:

- Elasticsearch
- Logstash
- Kibana

Typical architecture:

```text
Application
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
```

---

# 36. Elasticsearch

## Q33. What is Elasticsearch?

### Model Answer

> Elasticsearch is a distributed search and analytics engine commonly used for storing and querying log data.

Important concepts:

- Index
- Document
- Shard
- Replica
- Node
- Cluster

---

# 37. Logstash

## Q34. What does Logstash do?

### Model Answer

> Logstash collects, processes, transforms and routes events.

Example:

```text
Input
  |
  v
Filter
  |
  v
Output
```

It can parse structured/unstructured logs and send them to Elasticsearch.

---

# 38. Kibana

## Q35. What is Kibana?

### Model Answer

> Kibana provides visualization and search capabilities for Elasticsearch data.

Use cases:

- Log search
- Dashboards
- Error analysis
- Incident investigation
- Operational analytics

---

# 39. Centralized Logging

## Q36. Why centralized logging?

Without centralized logging:

```text
Pod A -> logs
Pod B -> logs
Pod C -> logs
```

Engineers must inspect many systems.

Centralized:

```text
Pods
 |
 v
Collector
 |
 v
Log pipeline
 |
 v
Elasticsearch
 |
 v
Kibana
```

This makes cross-service investigation easier.

---

# 40. Structured Logging

## Q37. What is structured logging?

Example:

```json
{
  "timestamp": "2026-08-15T10:10:22Z",
  "level": "ERROR",
  "service": "payment",
  "environment": "production",
  "request_id": "abc123",
  "message": "payment failed"
}
```

Structured fields make search and correlation easier.

---

# 41. Log Levels

## Q38. Explain log levels.

Common levels:

```text
DEBUG
INFO
WARN
ERROR
```

Production should generally avoid excessive DEBUG logging unless temporarily enabled through a controlled mechanism.

---

# 42. Logging Incident — Logs Stop

## Q39. How do you troubleshoot?

Trace:

```text
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
```

Check each stage.

---

# 43. Elasticsearch Red Cluster

## Q40. What does red mean?

### Model Answer

> A red Elasticsearch cluster means at least some primary shards are unassigned, so data availability may be affected.

Investigate:

- Node failures
- Disk
- Shard allocation
- Storage
- Cluster health

---

# 44. Logstash Queue Growth

## Q41. What do you check?

Compare:

```text
Input rate
Processing rate
Output rate
```

Possible bottlenecks:

- CPU
- Filters
- Elasticsearch
- Network
- Storage

---

# 45. Distributed Tracing

## Q42. What is distributed tracing?

### Model Answer

> Distributed tracing follows a request as it travels across multiple services and records timing and relationships between operations.

Example:

```text
Client
 |
 v
ALB
 |
 v
Order Service
 |
 +--> Inventory
 |
 +--> Payment
 |
 +--> Database
```

---

# 46. Trace vs Log

## Q43. Difference?

### Model Answer

> A trace provides the end-to-end path and timing of a request. Logs provide detailed events generated by individual components.

They complement each other.

---

# 47. Span

## Q44. What is a span?

### Model Answer

> A span represents a single operation within a trace.

Example:

```text
Trace
 |
 +--> HTTP request
      |
      +--> DB query
      |
      +--> Payment API
```

Each operation can be represented as a span.

---

# 48. Trace Context

## Q45. Why is trace context important?

### Model Answer

> Trace context allows trace identity to propagate across service boundaries so operations from different services can be correlated into a single trace.

---

# 49. OpenTelemetry

## Q46. What is OpenTelemetry?

### Model Answer

> OpenTelemetry is a vendor-neutral observability framework for generating, collecting and exporting telemetry such as metrics, logs and traces.

Architecture:

```text
Application
    |
    v
OpenTelemetry SDK/Instrumentation
    |
    v
Collector
    |
    +--> Metrics backend
    +--> Logs backend
    +--> Trace backend
```

---

# 50. OpenTelemetry Collector

## Q47. What does the Collector do?

### Model Answer

It can:

- Receive telemetry
- Process telemetry
- Filter
- Batch
- Enrich
- Export

This separates application instrumentation from backend destinations.

---

# 51. Jaeger

## Q48. What is Jaeger?

### Model Answer

> Jaeger is a distributed tracing platform used to collect, store and visualize traces.

Typical architecture:

```text
Application
    |
    v
OpenTelemetry
    |
    v
Jaeger
    |
    v
UI
```

---

# 52. Kubernetes Observability

## Q49. What should be monitored at pod level?

- CPU
- Memory
- Restarts
- OOMKilled
- Readiness
- Liveness
- Container status
- Network
- Ephemeral storage

---

# 53. Kubernetes Node Monitoring

## Q50. What do you monitor?

- CPU
- Memory
- Disk
- Disk inode usage
- Network
- Kubelet health
- Node readiness
- Pressure conditions

Commands:

```bash
kubectl get nodes
kubectl describe node <node>
kubectl top nodes
```

---

# 54. Kubernetes Cluster Monitoring

## Q51. What would you monitor?

- Node availability
- Pod scheduling
- API server availability
- Resource utilization
- Pending pods
- Evictions
- Cluster capacity
- Control-plane signals
- CoreDNS
- CNI

---

# 55. EKS Observability

## Q52. Explain your EKS monitoring approach.

### Model Answer

> I would use Prometheus for Kubernetes and application metrics, Grafana for dashboards and alert visualization, and ELK for centralized application and infrastructure logs. I would monitor EKS nodes, pods, workloads, ingress, resource utilization, application golden signals and important dependencies such as databases and queues. Alerts should be aligned with user impact and SLOs rather than simply alerting on every resource threshold.

---

# 56. Application Monitoring

## Q53. What should application monitoring include?

- Request rate
- Error rate
- Latency
- Saturation
- Dependency latency
- Connection pools
- Thread pools
- Queue depth
- Business transactions

---

# 57. Java Monitoring

## Q54. What would you monitor in Java?

- JVM heap
- Non-heap
- GC
- Threads
- CPU
- Memory
- Request latency
- Errors
- Connection pools

Potential issue:

```text
Heap ↑
GC ↑
Latency ↑
```

Possible memory pressure.

---

# 58. Node.js Monitoring

## Q55. What would you monitor?

- Event loop lag
- Heap
- GC
- CPU
- Memory
- Request rate
- Latency
- Errors
- Active connections

---

# 59. Python Monitoring

## Q56. What would you monitor?

- CPU
- Memory
- Request latency
- Errors
- Worker count
- Queue depth
- Database connections
- Application-specific metrics

---

# 60. Alerting

## Q57. What makes a good alert?

A good alert should be:

- Actionable
- Specific
- Owned
- Severity-based
- Based on meaningful symptoms
- Linked to a runbook
- Resistant to temporary noise

---

# 61. Bad Alert

Bad:

```text
CPU > 80%
```

Better:

```text
Checkout service CPU saturation is high for 15 minutes
AND request latency is above the SLO.
```

The second is closer to user impact.

---

# 62. Alertmanager

## Q58. What does Alertmanager do?

### Model Answer

Alertmanager handles alerts from Prometheus and provides:

- Grouping
- Routing
- Silencing
- Inhibition
- Notification

---

# 63. Alert Grouping

## Q59. Why grouping?

Instead of:

```text
500 alerts
```

send:

```text
EKS production cluster outage
```

with related alerts grouped together.

---

# 64. Inhibition

## Q60. What is inhibition?

### Model Answer

> Inhibition suppresses lower-priority or dependent alerts when a known higher-level failure is already active.

Example:

```text
Database outage
     |
     +--> Payment errors
     +--> Order errors
     +--> Inventory errors
```

The database outage may be the primary alert.

---

# 65. SLI

## Q61. What is an SLI?

### Model Answer

> An SLI is a quantitative measurement of a service's reliability or performance from the user's perspective.

Example:

```text
successful requests / valid requests
```

---

# 66. SLO

## Q62. What is an SLO?

### Model Answer

> An SLO is the target value for an SLI over a defined period.

Example:

```text
99.9% successful requests over 30 days
```

---

# 67. SLA

## Q63. What is an SLA?

### Model Answer

> An SLA is a formal agreement, often contractual, defining expected service levels and potentially consequences if they are not met.

---

# 68. Error Budget

## Q64. What is an error budget?

For a 99.9% SLO:

```text
Allowed failure = 0.1%
```

The error budget represents the amount of unreliability that can occur while still meeting the SLO.

---

# 69. Burn Rate

## Q65. What is burn rate?

### Model Answer

> Burn rate describes how quickly a service is consuming its error budget.

High burn rate:

> Reliability is deteriorating rapidly.

---

# 70. SLO Design

## Q66. Design an availability SLO.

Example:

```text
SLI:
successful valid requests / total valid requests

SLO:
99.9% over 30 days
```

Then define:

- Exclusions
- Measurement source
- Alerting
- Ownership
- Error budget policy

---

# 71. Production Architecture

## Q67. Design observability for an EKS microservices platform.

### Model Answer

```text
                    Users
                      |
                     ALB
                      |
                    EKS
          +-----------+-----------+
          |           |           |
       Service A   Service B   Service C
          |           |           |
          +-----------+-----------+
                      |
              Application Metrics
                      |
                 Prometheus
                      |
                    Grafana

Applications
    |
    v
Central Log Collection
    |
    v
Logstash
    |
    v
Elasticsearch
    |
    v
Kibana

Optional tracing:
Applications
    |
    v
OpenTelemetry
    |
    v
Tracing backend
```

---

# 72. Production Observability Design Principles

A production platform should have:

- High availability
- Retention policies
- Access control
- Encryption
- Backup/recovery
- Capacity planning
- Alert routing
- SLOs
- Runbooks
- Cost controls

---

# 73. Monitoring High Availability

## Q68. How would you make monitoring highly available?

### Model Answer

Use:

- Multiple Prometheus instances where appropriate
- Redundant alerting
- Durable/remote storage where required
- Multiple notification paths
- External blackbox monitoring
- Failure-domain-aware deployment

Monitoring should not depend on one node.

---

# 74. Monitoring Security

## Q69. How do you secure observability systems?

- Authentication
- RBAC
- TLS
- Network controls
- Encryption
- Secret management
- Least privilege
- Sensitive-log redaction
- Retention policies

---

# 75. Observability Cost Optimization

## Q70. How do you control observability costs?

### Metrics

- Reduce unnecessary cardinality
- Tune scrape intervals
- Remove unused metrics

### Logs

- Reduce DEBUG volume
- Retention policies
- Filtering
- Compression

### Dashboards

- Optimize expensive queries
- Use recording rules where appropriate

---

# 76. Incident Scenario 1 — API 100% Down

## Question

Production API is returning 100% 5xx. What do you do?

## Model Answer

> First I confirm the alert and external customer impact. Then I determine whether the failure is global or limited to a service, region, version or endpoint. I check ALB health, Kubernetes pods and services, recent deployments and major dependencies. If the incident correlates strongly with a recent deployment and rollback is safe, I would pause the rollout and roll back. I would then validate recovery using error rate, latency, traffic and business transaction success.

---

# 77. Incident Scenario 2 — API Latency Doubles

## Question

Latency doubled but CPU is normal.

### Model Answer

> I would not assume a CPU problem. I would check p50/p95/p99, database latency, external dependency latency, connection pools, queue depth and recent changes. I would segment latency by endpoint, version and dependency. A healthy CPU does not mean the application is healthy.

---

# 78. Incident Scenario 3 — Pods Pending

## Question

30 production pods are Pending.

### Model Answer

```bash
kubectl get pods -A
kubectl describe pod <pod>
kubectl get nodes
```

I would inspect scheduler events and determine whether the cause is:

- CPU/memory shortage
- Taints
- Affinity
- Quotas
- Node capacity

Then I would scale capacity or correct scheduling constraints as appropriate.

---

# 79. Incident Scenario 4 — OOMKilled

## Question

A production service repeatedly gets OOMKilled.

### Model Answer

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod <pod>
```

I would determine whether the memory growth is caused by traffic, a leak, payload size, cache growth or an incorrect limit. If a recent deployment caused it, rollback may be the fastest mitigation. I would then investigate the application-level root cause.

---

# 80. Incident Scenario 5 — Database Connection Exhaustion

### Model Answer

> I would check connection count, application connection pools, slow queries and recent releases. If the application is leaking connections, increasing the database connection limit may only postpone the problem. I would protect the database and fix the connection behavior.

---

# 81. Incident Scenario 6 — Queue Backlog

### Model Answer

> I would compare producer rate, consumer rate and queue age. Then I would determine whether consumers are failing, slow or blocked by a downstream dependency. I would scale consumers only if the downstream systems can handle the additional load.

---

# 82. Incident Scenario 7 — Prometheus Memory Explosion

### Model Answer

> I would investigate active series and recent metric changes, especially labels that create high cardinality. I would identify the offending metric, reduce cardinality or ingestion load and only then consider increasing capacity.

---

# 83. Incident Scenario 8 — Elasticsearch Red

### Model Answer

> I would check cluster health, unassigned primary shards, node health, disk and shard allocation. Because red indicates primary shard availability problems, I would prioritize data availability and recovery rather than blindly deleting data.

---

# 84. Incident Scenario 9 — Alert Storm

### Model Answer

> I would identify the primary failure and use grouping and inhibition to reduce secondary noise. I would preserve the important alert and investigate the dependency chain rather than treating hundreds of alerts as separate incidents.

---

# 85. Incident Scenario 10 — Monitoring Platform Down

### Model Answer

> I would use external monitoring, load-balancer checks, direct application tests and other independent signals to determine application health. Then I would restore the monitoring platform and investigate why its HA failed.

---

# 86. Advanced Scenario — Deployment Failure

## Question

A deployment completed successfully, but five minutes later error rate increased.

### Answer

> I would correlate the deployment timestamp with the error increase, compare old and new versions, segment metrics by version and inspect application logs. If the new version is responsible and rollback is safe, I would stop the rollout and roll back. I would then validate the rollback and preserve evidence for root-cause analysis.

---

# 87. Advanced Scenario — Rollback Does Not Fix It

### Answer

> I would reconsider the assumption that the deployment was the root cause. I would check database schema changes, configuration, external dependencies, infrastructure and traffic patterns. A rollback can fail to restore service if a shared dependency or persistent state has already changed.

---

# 88. Advanced Scenario — One Region Fails

### Answer

> I would confirm regional scope by comparing health and SLOs across regions. If another region is healthy, I would assess its spare capacity before shifting traffic. I would fail over according to the tested DR procedure and monitor the surviving region closely to avoid cascading failure.

---

# 89. Advanced Scenario — Retry Storm

### Answer

> I would identify which dependency is failing and whether clients are retrying aggressively. I would reduce amplification through exponential backoff, jitter, retry limits, circuit breakers and controlled traffic. Scaling the dependency may not be enough if retries continue increasing load.

---

# 90. Advanced Scenario — Cache Failure Causes DB Outage

### Answer

> I would recognize the dependency chain: cache failure reduces hit rate, database traffic increases and the database saturates. I would protect the database, restore or bypass the cache safely, control request load and validate both cache and database recovery.

---

# 91. Advanced Scenario — Customers Detect Incident First

### Answer

> That indicates a detection gap. I would investigate whether the missing signal is availability, latency, a business SLI or a user journey. I would add synthetic or business-level monitoring and an actionable alert rather than simply adding another infrastructure metric.

---

# 92. Advanced Scenario — Monitoring and Application Fail Together

### Answer

> I would use independent monitoring outside the affected failure domain. This is exactly why observability infrastructure needs its own HA and external detection path.

---

# 93. Advanced Scenario — Logs Contain Secrets

### Answer

> I would stop further exposure, rotate affected credentials, restrict access, assess retention and follow security procedures. I would then fix the logging path so sensitive fields are redacted or never emitted.

---

# 94. Advanced Scenario — Data Corruption

### Answer

> My first priority is containment. I would stop further corruption, preserve evidence, determine affected scope and protect backups. Recovery would follow the organization's approved data recovery procedure. I would not run destructive cleanup commands during an uncertain data incident.

---

# 95. Advanced Scenario — DR Failover Causes Second Outage

### Answer

> This indicates insufficient DR capacity or an untested failover assumption. I would stabilize the surviving region first, then review capacity planning, load testing, autoscaling and dependency limits. DR should be tested under realistic failover load.

---

# 96. Advanced Scenario — SLO Burn Rate Is High

### Answer

> I would identify which SLI is consuming the error budget and determine whether the cause is errors, latency or availability. Then I would prioritize mitigation based on user impact and burn rate. A high burn rate may justify pausing risky releases until reliability is restored.

---

# 97. Advanced Scenario — Alert Has No Owner

### Answer

> I would treat ownership as part of alert quality. The alert should identify service, team, severity and runbook. Without ownership, even a technically correct alert can fail operationally.

---

# 98. Advanced Scenario — Alert Is Too Noisy

### Answer

> I would examine whether the alert is actionable and aligned with customer impact or SLO risk. I would tune thresholds, duration and grouping, remove non-actionable pages and add runbook links. The goal is not fewer alerts; it is higher-quality alerts.

---

# 99. Advanced Scenario — Metrics and Logs Disagree

### Answer

> I would validate timestamp, query, data freshness, collection path and scope. I would check the raw datasource and compare it with the dashboard. I would not assume either signal is automatically correct.

---

# 100. Advanced Scenario — No Root Cause Yet

### Answer

> During an active incident I would focus on mitigation and evidence collection. I would maintain a list of hypotheses and explicitly distinguish the leading hypothesis from confirmed root cause. Once service is stable, I would continue the investigation using the preserved timeline and telemetry.

---

# 101. Linux Monitoring Interview

## Q101. How do you investigate high disk usage?

```bash
df -h
df -i
du -xh /var | sort -h | tail
```

Approach:

1. Identify full filesystem.
2. Identify large directories.
3. Identify large files.
4. Check logs.
5. Check deleted-but-open files.
6. Check application growth.
7. Clean only according to approved procedures.

---

# 102. High CPU

## Q102. What do you check?

```bash
top
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head
```

Then determine:

- Process
- Application behavior
- Traffic
- Threads
- Recent deployment

---

# 103. High Memory

Commands:

```bash
free -h
top
ps -eo pid,cmd,%mem --sort=-%mem | head
```

Then correlate with application metrics.

---

# 104. Network Troubleshooting

## Q104. Application cannot connect to database.

Check:

```bash
ss -lntp
curl -v <endpoint>
```

Then inspect:

- DNS
- Route
- Security group
- Network ACL
- Port
- Application configuration
- Database listener

---

# 105. Kubernetes Commands You Should Know

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl get events --sort-by=.lastTimestamp
kubectl get nodes
kubectl describe node <node>
kubectl top pods
kubectl top nodes
kubectl get svc
kubectl get endpoints
kubectl get ingress
kubectl get deployment
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
```

---

# 106. Prometheus Commands / Queries You Should Know

Examples:

```promql
up
rate(http_requests_total[5m])
sum(rate(http_requests_total[5m])) by (service)
sum(rate(http_requests_total{status=~"5.."}[5m]))
histogram_quantile(0.95, ...)
```

Know what each query means rather than memorizing syntax.

---

# 107. Grafana Interview Questions

## Q107. Why use Grafana if Prometheus has a UI?

### Answer

> Prometheus provides querying and basic visualization, while Grafana provides richer dashboards, multiple data sources, operational views, variables, annotations and visualization capabilities.

---

# 108. ELK Interview Questions

## Q108. Why Elasticsearch instead of plain files?

### Answer

> Centralized search allows engineers to correlate logs from multiple services, search large datasets and build operational dashboards.

---

# 109. Observability Architecture Question

## Q109. Design monitoring for 50 microservices.

### Model Answer

```text
                 EKS
                  |
       +----------+----------+
       |          |          |
    Service A  Service B  Service N
       |          |          |
       +----------+----------+
                  |
        +---------+---------+
        |                   |
      Metrics              Logs
        |                   |
        v                   v
   Prometheus            Logstash
        |                   |
        v                   v
     Grafana           Elasticsearch
                            |
                            v
                          Kibana
```

Use standardized:

- Metric names
- Labels
- Log fields
- Service names
- Environment labels
- Alert ownership
- SLO definitions

---

# 110. Architecture Follow-Up

## Q110. What happens if Prometheus fails?

### Answer

> I would use redundant monitoring where required, durable storage for appropriate use cases, external monitoring and redundant alert delivery. Monitoring should not be a single point of failure.

---

# 111. Architecture Follow-Up

## Q111. What happens if Elasticsearch fails?

### Answer

> The application should continue operating independently of centralized logging. The logging layer should have buffering or resilient collection where required. I would restore the backend while preserving logs that can still be buffered.

---

# 112. Architecture Follow-Up

## Q112. How do you scale observability?

### Metrics

- Reduce cardinality
- Shard/scale collectors where required
- Use durable remote storage architecture when appropriate

### Logs

- Scale ingestion
- Partition storage
- Retention policies

### Dashboards

- Query optimization
- Recording rules

---

# 113. Scenario-Based Rapid Fire

Answer each in 30–60 seconds.

## Q113. Pod is Running but application is unavailable.

Think:

- Readiness
- Service
- Endpoints
- Port
- Ingress
- Application listener

---

## Q114. Pod restarts every few minutes.

Think:

- CrashLoopBackOff
- Logs
- OOMKilled
- Liveness
- Application exit
- Dependency failure

---

## Q115. Node is Ready but pods have high latency.

Think:

- CPU throttling
- Network
- Disk
- Noisy neighbor
- Application
- Dependency

---

## Q116. CPU is low but latency is high.

Think:

- Database
- External API
- Lock contention
- Queue
- Network
- Thread/event-loop blocking

---

## Q117. Error rate is normal but users report slow application.

Think:

- Latency SLI
- p95/p99
- Dependency latency
- Client/network
- Synthetic monitoring

---

## Q118. Error rate suddenly drops to zero.

This is not automatically good.

Check:

- Traffic
- Metrics pipeline
- Application availability
- Monitoring freshness

Zero traffic can produce zero errors.

---

## Q119. Traffic drops to zero.

Investigate:

- DNS
- ALB
- Routing
- Client traffic
- Deployment
- External dependency
- Monitoring pipeline

---

## Q120. Queue depth is zero but processing is failing.

Check whether producers are also failing.

Zero queue depth does not necessarily mean healthy processing.

---

# 121. Interview Trick Questions

## Q121. Is CPU above 80% always an incident?

### Answer

No.

High CPU can be normal during healthy utilization.

The important question is:

> Is the system approaching saturation and causing user impact?

---

## Q122. Is 100% CPU always bad?

No.

A CPU-bound service can intentionally use high CPU while meeting its SLO.

---

## Q123. Is high memory always a problem?

No.

Memory usage should be evaluated against capacity, workload behavior and symptoms such as OOM or swapping.

---

## Q124. Is a pod in Running state healthy?

No.

Running only means the container is running.

Check:

- Readiness
- Errors
- Latency
- Business health

---

## Q125. Is HTTP 200 enough to say the service is healthy?

No.

The request could return 200 while the business operation fails or returns invalid data.

---

## Q126. Is a green dashboard enough?

No.

Dashboards can contain incorrect queries, stale data or incomplete signals.

---

## Q127. Is no alert equal to no incident?

No.

Monitoring can have blind spots.

---

## Q128. Should every metric have an alert?

No.

Not every metric is actionable.

---

## Q129. Should every log be stored forever?

No.

Retention should balance operational, security, compliance and cost requirements.

---

## Q130. Should we use DEBUG logs in production permanently?

Usually no.

Use controlled mechanisms when detailed diagnostics are temporarily required.

---

# 131. Senior Scenario — What Would You Do First?

## Question

You receive:

```text
Pager alert:
Checkout availability below SLO.
```

What do you do?

### Strong Answer

> First I confirm the alert and establish customer impact. I check traffic, errors and latency for checkout, then segment by version, region and endpoint. I check recent deployments and dependencies such as payment, inventory and database. If a recent change clearly correlates with the incident and rollback is safe, I prioritize rollback. Throughout the response I document actions and communicate the current state. After mitigation I validate both technical recovery and checkout success rate before closing the incident.

---

# 132. Senior Scenario — Why Not Start With Logs?

## Question

Why not immediately search logs?

### Answer

> Logs are valuable, but starting with logs without understanding scope can produce a huge amount of noise. I prefer to establish impact, time range and affected service first using metrics, then use logs to investigate the specific failure window.

---

# 133. Senior Scenario — Why Metrics First?

### Answer

Metrics provide fast aggregation and help answer:

- When?
- How much?
- Where?
- Which service?
- Which version?
- Which region?

Then logs provide detailed context.

---

# 134. Senior Scenario — Why Not Start With Kubernetes?

### Answer

> Kubernetes may be healthy while the real failure is in the database, DNS, external API or application. I start with customer impact and the complete request path rather than assuming the infrastructure layer is the problem.

---

# 135. Senior Scenario — Why Check Recent Changes?

### Answer

> Recent changes are high-value evidence because many production incidents correlate with deployments, configuration, infrastructure or feature-flag changes. However, correlation is not automatically causation, so I validate it against telemetry.

---

# 136. Senior Scenario — What Is the First Metric You Look At?

There is no universal first metric.

A strong answer:

> I start with the service's primary SLI and golden signals: availability/error rate, traffic and latency. Then I use saturation and dependency metrics to narrow the cause.

---

# 137. Senior Scenario — How Do You Prioritize Dependencies?

Look at:

- Dependency criticality
- Latency
- Error rate
- Request volume
- Recent changes
- Business path

For checkout:

```text
Checkout
 |
 +--> Payment
 +--> Inventory
 +--> Order DB
```

Payment may have greater direct business impact than a non-critical dependency.

---

# 138. Senior Scenario — How Do You Handle Uncertainty?

Use:

```text
Known
Unknown
Hypothesis
Evidence
Next test
```

Example:

```text
Known:
Checkout errors increased.

Unknown:
Whether DB or payment caused it.

Hypothesis:
Payment latency caused request timeout.

Next:
Compare payment latency with checkout timeout timestamps.
```

---

# 139. Senior Scenario — What Is a Safe Change?

A safe change is generally:

- Reversible
- Small blast radius
- Well understood
- Observable
- Documented

Examples:

- Pause rollout
- Disable feature flag
- Shift traffic
- Roll back known-safe release

---

# 140. Senior Scenario — What Is an Unsafe Change?

Examples:

- Delete production data
- Restart entire cluster
- Change many components simultaneously
- Disable all security controls
- Delete all logs
- Blindly increase retries

---

# 141. Mock Interview — Basic Round

Answer without looking at the model answers.

1. What is monitoring?
2. What is observability?
3. What are metrics, logs and traces?
4. What are golden signals?
5. What is Prometheus?
6. Why does Prometheus use pull?
7. What is PromQL?
8. Counter vs gauge?
9. What is histogram?
10. What is an exporter?
11. What is Grafana?
12. What is ELK?
13. What is Elasticsearch?
14. What is Logstash?
15. What is Kibana?
16. What is structured logging?
17. What is distributed tracing?
18. What is a span?
19. What is OpenTelemetry?
20. What is Jaeger?

---

# 142. Mock Interview — Intermediate Round

1. Explain Prometheus architecture.
2. Explain Prometheus service discovery.
3. How do you monitor Kubernetes?
4. How do you monitor EKS?
5. How do you troubleshoot a DOWN target?
6. How do you troubleshoot missing Prometheus data?
7. What causes Prometheus high memory?
8. What is metric cardinality?
9. How do you design Grafana dashboards?
10. How do you troubleshoot Grafana?
11. Explain centralized logging.
12. How do you troubleshoot Logstash?
13. How do you troubleshoot Elasticsearch?
14. Explain structured logging.
15. How do you monitor Java?
16. How do you monitor Node.js?
17. How do you monitor Python?
18. What is Alertmanager?
19. What is alert grouping?
20. What is inhibition?
21. What is SLI?
22. What is SLO?
23. What is SLA?
24. What is error budget?
25. What is burn rate?

---

# 143. Mock Interview — Advanced Round

1. Design observability for an EKS microservices platform.
2. How would you make Prometheus highly available?
3. How do you control Prometheus cardinality?
4. How do you scale centralized logging?
5. How do you handle monitoring backend failure?
6. How do you design SLOs?
7. How do you design burn-rate alerts?
8. How do you reduce alert fatigue?
9. How do you troubleshoot cascading failures?
10. How do you handle regional outages?
11. How do you design DR observability?
12. How do you protect observability data?
13. How do you optimize observability cost?
14. How do you detect business-level failures?
15. How do you integrate observability into CI/CD?
16. How do you validate monitoring after deployment?
17. How do you prevent telemetry from becoming a bottleneck?
18. How do you handle high-cardinality metrics?
19. How do you handle log-volume explosions?
20. How do you build production-ready incident response?

---

# 144. Mock Interview — Scenario Round

## Scenario 1

Production API is returning 503.

Answer using:

```text
Confirm
Scope
Timeline
Dependencies
Mitigate
Validate
Prevent
```

---

## Scenario 2

Prometheus memory increases continuously.

Discuss:

- Cardinality
- Active series
- Scrape volume
- Queries
- Retention

---

## Scenario 3

Elasticsearch turns red.

Discuss:

- Cluster health
- Primary shards
- Nodes
- Disk
- Allocation
- Recovery

---

## Scenario 4

Kubernetes pods are Pending.

Discuss:

- Scheduler events
- Requests
- Capacity
- Taints
- Affinity
- Quotas

---

## Scenario 5

One region is failing.

Discuss:

- Scope
- Regional comparison
- Failover
- Capacity
- Dependencies
- Validation

---

# 145. Mock Interview — Architecture Round

## Question

Design an observability platform for:

```text
AWS
 |
 v
EKS
 |
 +--> 30 microservices
 +--> RDS
 +--> RabbitMQ
 +--> ALB
```

### Expected discussion

Metrics:

```text
Services
   |
   v
Prometheus
   |
   v
Grafana
```

Logs:

```text
Services
   |
   v
Collectors
   |
   v
Logstash
   |
   v
Elasticsearch
   |
   v
Kibana
```

Alerting:

```text
Prometheus
   |
   v
Alertmanager
   |
   v
Notification
```

Include:

- HA
- RBAC
- TLS
- Retention
- SLOs
- Runbooks
- Cost management
- DR

---

# 146. Mock Interview — Production Architecture Challenge

## Question

Your company has:

- 3 EKS clusters
- 50 microservices
- Multiple environments
- RDS
- RabbitMQ
- ALB
- Jenkins
- ArgoCD

Design observability.

### Strong discussion

Standardize:

```text
environment
cluster
namespace
service
version
team
region
```

Metrics:

- Prometheus
- Exporters
- Service discovery
- Recording rules

Visualization:

- Grafana

Logs:

- Centralized ELK

Alerting:

- Alertmanager

SLO:

- Service-level reliability

Incident:

- Runbooks
- On-call
- Postmortems

---

# 147. Mock Interview — Failure of Observability Platform

## Question

Your production application is healthy, but Prometheus and Grafana are down. Is this a production incident?

### Answer

> Yes, it is an operational incident because the organization has lost important visibility and possibly alerting. The application may still be healthy, but monitoring availability itself needs to be restored. I would use independent monitoring to confirm application health while restoring the observability platform.

---

# 148. Mock Interview — Logging Failure

## Question

ELK is down. Should you stop the application?

### Answer

> Normally no. The application should not depend synchronously on centralized logging. I would restore the logging pipeline and use buffering or local collection mechanisms where appropriate.

---

# 149. Mock Interview — Alert Failure

## Question

Prometheus is healthy but no alerts are reaching engineers.

### Answer

Trace:

```text
Metric
  |
  v
Rule
  |
  v
Alert
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
```

Find the first broken stage.

---

# 150. Mock Interview — SLO Failure

## Question

Availability is 99.7% and SLO is 99.9%. What do you do?

### Answer

> I would quantify the error-budget consumption, identify the cause of the failure, determine whether the issue is ongoing and prioritize remediation according to the burn rate and business impact. If the budget is exhausted, release-risk policy may require slowing or pausing risky changes.

---

# 151. Mock Interview — Monitoring vs Business Health

## Question

All infrastructure metrics are green, but customers report checkout failures.

### Answer

> I would suspect a monitoring blind spot or business-level failure. I would check checkout success as a business SLI, inspect application logs and dependencies and validate the complete user journey. Infrastructure health does not automatically equal business health.

---

# 152. Mock Interview — Debugging Mindset

## Question

What is your general troubleshooting philosophy?

### Model Answer

> I start from impact and scope rather than assumptions. I establish a timeline, use metrics to narrow the problem, use logs and events for detail, inspect dependencies and recent changes, then choose a safe mitigation. After recovery I validate the system and investigate root cause. I avoid random changes because they can increase blast radius and destroy evidence.

---

# 153. Mock Interview — Explain a Production Incident

Use this format:

```text
Situation:
What happened?

Impact:
What users/services were affected?

Detection:
How did you detect it?

Investigation:
What signals did you inspect?

Root cause:
What actually failed?

Mitigation:
How did you restore service?

Validation:
How did you know it recovered?

Prevention:
What did you change afterward?
```

---

# 154. Sample Production Incident Answer

> We had a production service where application errors increased after a deployment. I first checked the alert and confirmed that customer-facing error rate had increased. I compared the timing with the deployment and segmented the errors by application version. The new version showed significantly higher errors. We paused the rollout and rolled back to the previous version. After rollback, error rate and latency returned to normal and we validated the business transaction. Later analysis identified a database query regression. We added a query regression test and improved deployment validation so the same issue would be caught before production.

---

# 155. How to Answer "What Did You Personally Do?"

Avoid:

> "We checked everything."

Say:

> "I checked..."

> "I identified..."

> "I compared..."

> "I proposed..."

> "I implemented..."

> "I validated..."

Use "we" for team outcomes but clearly explain your contribution.

---

# 156. How to Answer When You Don't Know

Bad:

> "I don't know."

Better:

> "I have not handled that exact failure directly, but my investigation would start by establishing scope, checking the relevant metrics and dependencies, and then validating the most likely failure points."

Never invent production experience.

---

# 157. How to Handle an Interviewer Challenge

Interviewer:

> "Why would you scale instead of restart?"

Answer:

> "I would choose based on the failure mode. If pods are overloaded due to legitimate traffic, scaling may increase capacity. If the application has a memory leak, scaling may only postpone failure. I would inspect evidence before selecting the mitigation."

---

# 158. Interviewer Challenge — Why Not Increase CPU?

Answer:

> "If CPU is actually the bottleneck, increasing capacity can help. But if the bottleneck is database latency, network, locks or an external dependency, more CPU will not solve the root cause."

---

# 159. Interviewer Challenge — Why Prometheus?

Answer:

> "Prometheus provides strong Kubernetes integration, dimensional time-series metrics, PromQL and a mature alerting ecosystem. It works well for infrastructure and application monitoring in cloud-native environments."

---

# 160. Interviewer Challenge — Why Grafana?

Answer:

> "Grafana provides flexible visualization and operational dashboards across multiple data sources. It is useful for service health, incident investigation, SLO dashboards and cross-system correlation."

---

# 161. Interviewer Challenge — Why ELK?

Answer:

> "ELK provides centralized search and analysis of logs. It helps correlate events across multiple microservices and infrastructure components, which is difficult when logs remain distributed across individual pods or servers."

---

# 162. Interviewer Challenge — Why SLO Instead of CPU Alerts?

Answer:

> "CPU is an infrastructure signal, while an SLO represents service reliability. CPU can be high without customer impact, and customer impact can occur while CPU is normal. SLOs align alerting and operational decisions with user experience."

---

# 163. Interviewer Challenge — What Is More Important: Logs or Metrics?

Answer:

> "Neither is universally more important. Metrics are usually faster for detecting and scoping problems, while logs provide detailed context. During incidents I typically use metrics to narrow the failure and logs to understand the exact events."

---

# 164. Interviewer Challenge — What About Tracing?

Answer:

> "Tracing is especially valuable for distributed systems when a request crosses multiple services. It helps identify which service or dependency contributed to latency or failure."

---

# 165. Interviewer Challenge — Your Stack Does Not Include Tracing

A safe answer:

> "In the environments I have worked with, my primary observability stack has been Prometheus, Grafana and ELK. I understand distributed tracing concepts and the OpenTelemetry/Jaeger architecture, but I would not claim hands-on production experience with a tracing platform if I have not operated one."

This is better than exaggerating experience.

---

# 166. Interviewer Challenge — What Is Your Monitoring Stack?

### Model Answer

> "My practical monitoring stack is Prometheus for metrics, Grafana for visualization and ELK for centralized logging. In Kubernetes/EKS, I use metrics for infrastructure and application health, Grafana dashboards for operational visibility and ELK for detailed log investigation. I also apply SLO-based alerting and incident troubleshooting principles."

---

# 167. DevOps Interview — Monitoring in CI/CD

## Question

How would you integrate observability into CI/CD?

### Answer

Pipeline:

```text
Git
 |
 v
Build
 |
 v
Test
 |
 v
Security Scan
 |
 v
Deploy
 |
 v
Smoke Test
 |
 v
Observability Validation
 |
 v
SLO Check
 |
 v
Promote / Rollback
```

Validate:

- Metrics available
- Logs available
- Health checks
- Error rate
- Latency
- Deployment health

---

# 168. DevSecOps Interview — Observability and Security

Observability can detect:

- Authentication anomalies
- Unexpected traffic
- Privilege errors
- Failed access
- Secret exposure
- Suspicious behavior

But observability data itself must be protected.

---

# 169. Kubernetes Deployment + Observability

Before deployment:

- Dashboard exists
- Alerts exist
- Logs available
- SLO defined

During deployment:

- Watch error rate
- Watch latency
- Watch restarts
- Watch readiness

After deployment:

- Validate business transaction
- Compare old/new version
- Confirm telemetry

---

# 170. Production Release Gate

Example:

```text
Deploy 10%
   |
   v
Check:
  Error rate
  Latency
  Availability
   |
   +--> Healthy --> 50%
   |
   +--> Unhealthy --> Rollback
```

Observability can become a deployment safety mechanism.

---

# 171. Incident Response Interview

## Question

What is your first priority during an outage?

### Answer

> Restore safe service and protect customers.

Not:

> Find the perfect root cause immediately.

---

# 172. Incident Response Interview

## Question

What is the difference between MTTD and MTTR?

### Answer

MTTD:

> Mean Time To Detect.

MTTR:

> Mean Time To Restore/Resolve, depending on organizational definition.

A good observability platform reduces detection and diagnosis time, while automation and reliable recovery mechanisms reduce restoration time.

---

# 173. Incident Response Interview

## Question

How do dashboards reduce MTTR?

### Answer

They reduce investigation time by presenting:

- Scope
- Golden signals
- Dependencies
- Resource health
- Deployment history
- Relevant context

A dashboard should support a decision, not just display graphs.

---

# 174. Incident Response Interview

## Question

What makes a runbook useful?

A good runbook contains:

```text
Alert
Impact
Checks
Commands
Expected results
Mitigation
Rollback
Validation
Escalation
```

---

# 175. Production Incident Checklist

## Before Incident

- [ ] Monitoring deployed
- [ ] Dashboards ready
- [ ] Alerts tested
- [ ] Runbooks available
- [ ] Ownership defined
- [ ] SLOs defined
- [ ] Backup/recovery tested

## During Incident

- [ ] Confirm impact
- [ ] Establish scope
- [ ] Establish timeline
- [ ] Check recent changes
- [ ] Check dependencies
- [ ] Mitigate safely
- [ ] Communicate

## After Incident

- [ ] Validate recovery
- [ ] Capture timeline
- [ ] Identify root cause
- [ ] Create corrective actions
- [ ] Update monitoring
- [ ] Update runbook
- [ ] Review SLO impact

---

# 176. Final 60-Second Monitoring Answer

If an interviewer asks:

> "Explain your monitoring and observability approach."

Use:

> "I approach observability from the service and user perspective. For metrics, I use Prometheus to collect infrastructure and application telemetry and Grafana for visualization and operational dashboards. For logs, I use ELK for centralized collection, search and troubleshooting. I focus on golden signals such as traffic, errors, latency and saturation, along with Kubernetes health, dependencies and resource utilization. For alerting, I prefer actionable alerts aligned with service health and SLOs rather than alerting on every infrastructure threshold. During incidents, I establish impact and scope first, correlate metrics with logs and recent changes, use safe mitigation such as rollback when appropriate, validate recovery using both technical and business signals, and then perform root-cause analysis and preventive improvements."

---

# 177. Final 60-Second Production Incident Answer

> "During a production incident, my first priority is customer impact and safe service restoration. I confirm the alert, determine scope and establish a timeline. Then I check golden signals, recent deployments, infrastructure and dependencies. I maintain a small set of evidence-based hypotheses and choose the safest reversible mitigation. Once service is restored, I validate error rate, latency, availability, dependency health and business transactions. After the incident, I document the timeline, identify root cause and contributing factors, and implement corrective actions such as better monitoring, tests, automation or architecture changes."

---

# 178. Final 60-Second EKS Observability Answer

> "For an EKS microservices environment, I would use Prometheus for cluster, node, pod and application metrics and Grafana for dashboards and operational visibility. I would centralize application and infrastructure logs through ELK. I would monitor Kubernetes scheduling, pod restarts, OOMKilled, resource utilization, ingress health and application golden signals. I would also monitor dependencies such as RDS and RabbitMQ. Alerts should be actionable and aligned with SLOs. For production readiness I would add HA, access control, retention, capacity planning, runbooks, alert routing and independent monitoring so the observability platform itself does not become a single point of failure."

---

# 179. Final Interview Cheat Sheet

## Monitoring

```text
CPU
Memory
Disk
Network
Traffic
Errors
Latency
Saturation
```

## Observability

```text
Metrics
Logs
Traces
```

## Prometheus

```text
Scrape
Store
PromQL
Alert
```

## Grafana

```text
Query
Visualize
Dashboard
Alert
```

## ELK

```text
Collect
Process
Store
Search
Visualize
```

## Kubernetes

```text
Pods
Nodes
Services
Ingress
Resources
Events
Dependencies
```

## Alerting

```text
Actionable
Owned
Severity
Runbook
SLO
```

## SRE

```text
SLI
SLO
SLA
Error Budget
Burn Rate
```

## Incidents

```text
Impact
Scope
Timeline
Evidence
Mitigation
Recovery
Validation
Root Cause
Prevention
```

---

# 180. Final Interview Rules

Remember these principles:

> Do not memorize commands without understanding the failure they investigate.

> Do not say "CPU is high, so CPU is the problem."

> Do not treat pod Running state as application health.

> Do not treat a green dashboard as absolute truth.

> Do not confuse monitoring with observability.

> Do not confuse mitigation with root cause.

> Do not restart everything blindly.

> Do not scale everything blindly.

> Do not silence all alerts during an incident.

> Do not claim tracing experience you do not have.

> Do not invent production incidents.

> Be honest about what you have personally operated.

> Explain your reasoning.

> Start with customer impact.

> Use metrics to establish scope.

> Use logs for detailed evidence.

> Use dependencies to understand distributed failures.

> Use SLOs to connect technical health to reliability.

> Prefer safe and reversible changes.

> Validate recovery with multiple signals.

> Preserve incident evidence.

> Finish incidents with prevention.

---

# 181. Final Self-Evaluation

Score yourself from 1–5.

| Skill | Score |
|---|---:|
| Monitoring fundamentals | /5 |
| Observability concepts | /5 |
| Metrics | /5 |
| Prometheus | /5 |
| PromQL | /5 |
| Grafana | /5 |
| ELK | /5 |
| Kubernetes monitoring | /5 |
| EKS observability | /5 |
| Application monitoring | /5 |
| Alerting | /5 |
| SLI/SLO/SLA | /5 |
| Incident response | /5 |
| Production troubleshooting | /5 |
| Architecture | /5 |
| Communication | /5 |
| Root-cause analysis | /5 |

### Target

```text
Basic:
3/5

Interview-ready:
4/5

Strong production candidate:
4.5/5+

Senior-level:
5/5 with clear real-world reasoning
```

---

# 182. Final Mock Interview Sequence

Use this sequence for repeated practice.

## Round 1 — Fundamentals

10 questions.

## Round 2 — Prometheus/Grafana

15 questions.

## Round 3 — ELK

15 questions.

## Round 4 — Kubernetes/EKS

15 questions.

## Round 5 — SLO/Alerting

15 questions.

## Round 6 — Production Scenarios

10 incidents.

## Round 7 — Architecture

5 architecture questions.

## Round 8 — Rapid Fire

20 questions.

## Round 9 — Project Experience

Explain your actual monitoring architecture.

## Round 10 — Final Senior Round

Five production incidents with no preparation.

---

# 183. Project-Based Interview Preparation

For your own project, prepare answers for:

1. What did you monitor?
2. Why Prometheus?
3. Why Grafana?
4. How did you collect logs?
5. How did you troubleshoot production issues?
6. What alerts did you configure?
7. What was your most difficult incident?
8. How did you reduce MTTR?
9. How did you handle Kubernetes failures?
10. How did you monitor application health?
11. How did you monitor infrastructure?
12. How did you handle log volume?
13. How did you avoid noisy alerts?
14. How did you define SLOs?
15. How did monitoring help deployment?
16. What would you improve in your observability architecture?

---

# 184. Final Project Answer Template

Use your real experience and fill in:

```text
Project:
Environment:
Cloud:
Kubernetes:
Services:

Metrics:
Prometheus configuration:
Exporters:
Dashboards:

Logging:
Collection:
Processing:
Storage:
Search:

Alerting:
Critical alerts:
Routing:
Runbooks:

Incident:
What happened:
How detected:
Investigation:
Mitigation:
Root cause:
Prevention:

My contribution:
```

---

# 185. Final Master Mental Model

When the interviewer gives you any production monitoring problem, think:

```text
                    USER IMPACT
                         |
                         v
                    WHAT FAILED?
                         |
                         v
                       SCOPE
                         |
          +--------------+--------------+
          |              |              |
        Region         Version        Service
          |              |              |
          +--------------+--------------+
                         |
                         v
                      TIMELINE
                         |
                         v
                 RECENT CHANGES
                         |
                         v
                    GOLDEN SIGNALS
                         |
          +--------------+--------------+
          |              |              |
       Traffic         Errors         Latency
                         |
                         v
                    SATURATION
                         |
                         v
                   DEPENDENCIES
                         |
          +--------------+--------------+
          |              |              |
          DB            Queue         External API
                         |
                         v
                      LOGS
                         |
                         v
                     EVENTS
                         |
                         v
                   HYPOTHESES
                         |
                         v
                    MITIGATION
                         |
                         v
                     RECOVERY
                         |
                         v
                    VALIDATION
                         |
                         v
                   ROOT CAUSE
                         |
                         v
                   PREVENTION
```

---

# 186. Completion of Monitoring Interview Preparation

This completes:

```text
20-Interview-Preparation/
├── 01-Basic-Questions.md
├── 02-Intermediate-Questions.md
├── 03-Advanced-Questions.md
├── 04-Scenario-Based.md
├── 05-Architecture-Questions.md
├── 06-Troubleshooting-Scenarios.md
├── 07-Production-Incident-Scenarios.md
└── 08-Mock-Interview.md
```

The complete Monitoring & Observability interview preparation now covers:

```text
Fundamentals
    |
    v
Metrics
    |
    v
Prometheus
    |
    v
Grafana
    |
    v
ELK
    |
    v
Tracing Concepts
    |
    v
Kubernetes / EKS
    |
    v
Application Monitoring
    |
    v
Infrastructure Monitoring
    |
    v
Alerting
    |
    v
SLI / SLO / SLA
    |
    v
Production Architecture
    |
    v
Troubleshooting
    |
    v
Production Incidents
    |
    v
Mock Interviews
```

---

# 187. Final Rule for Your Interview Preparation

Do not try to sound like you memorized an observability textbook.

Sound like an engineer who can operate production.

The strongest answer is usually:

> "First I would confirm the customer impact and scope. Then I would establish the timeline and correlate it with recent changes. I would use metrics to narrow the problem, logs and events for evidence, and inspect dependencies. I would choose the safest reversible mitigation, validate recovery using both technical and business signals, and then perform root-cause analysis and implement preventive improvements."

That is the mindset interviewers expect from a production DevOps / DevSecOps engineer.
