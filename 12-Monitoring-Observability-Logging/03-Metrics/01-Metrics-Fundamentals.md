# Metrics Fundamentals

Metrics are numerical measurements collected over time that help
engineers understand the health, performance, reliability, and behavior
of systems.

In modern DevOps and cloud-native environments, metrics are one of the
most important observability signals.

Metrics help answer questions such as:

```
Is the application receiving traffic?

Is the application responding quickly?

Are requests failing?

Is CPU usage increasing?

Is memory usage increasing?

Are Kubernetes pods restarting?

Is the database running out of connections?

Is the application approaching capacity?
```

A typical production monitoring architecture is:

```
Application
     |
     ↓
  Metrics
     |
     ↓
Prometheus
     |
     ↓
  PromQL
     |
     ↓
  Grafana
     |
     +------→ Dashboards
     |
     +------→ Alerts
     |
     ↓
Incident Response
```

---

# 1. What Is a Metric?

A metric is a numerical measurement associated with a specific point in
time.

Examples:

```
CPU Usage = 72%

Memory Usage = 4.5 GB

Requests/sec = 2,500

Error Rate = 1.2%

Request Latency = 250ms

Active Connections = 180
```

A metric normally contains:

```
Metric Name
Labels
Timestamp
Value
```

Example:

```
http_requests_total{
    service="order-service",
    method="GET",
    status="200"
}
```

The value might be:

```
125000
```

---

# 2. Metrics vs Logs

Metrics are numerical and optimized for aggregation and alerting.

Logs are event records containing detailed information.

Example metric:

```
http_requests_total = 125000
```

Example log:

```
2026-08-10 10:15:32
ERROR
payment-service
Database connection timeout
```

Metrics answer:

```
What is happening?
```

Logs help answer:

```
What exactly happened?
```

---

# 3. Metrics vs Traces

Metrics provide an aggregated view.

Traces provide request-level details.

Example:

```
Metric:

P95 latency = 800ms
```

This tells us that latency is high.

A trace can show:

```
API
  |
  ↓
Order Service
  |
  ↓
Payment Service
  |
  ↓
Database
```

And identify which operation caused the delay.

A practical observability model is:

```
Metrics
   ↓
Detect
   ↓
Logs
   ↓
Investigate
   ↓
Traces
   ↓
Locate
```

---

# 4. Why Metrics Matter

Metrics are useful because they are:

```
Numerical
Compact
Efficient to store
Easy to aggregate
Easy to visualize
Easy to alert on
Suitable for long-term trends
```

For example, storing every CPU measurement as a log entry would create
large volumes of data.

Metrics are designed specifically for this type of monitoring.

---

# 5. Metrics in DevOps

Metrics are used throughout the DevOps lifecycle.

```
Development
    |
    ↓
CI/CD
    |
    ↓
Deployment
    |
    ↓
Production
    |
    ↓
Monitoring
    |
    ↓
Optimization
```

Examples:

```
Build Duration
Deployment Frequency
Deployment Failure Rate
Application Traffic
Application Latency
Error Rate
Infrastructure Utilization
```

---

# 6. Metrics in Production

A production service should expose useful metrics around:

```
Traffic
Latency
Errors
Saturation
```

These are the Golden Signals.

Example:

```
Order Service

Traffic:
5,000 req/sec

P95 Latency:
250ms

Error Rate:
0.4%

CPU:
55%
```

These four measurements immediately provide useful information about
service health.

---

# 7. Metrics Architecture

A typical Prometheus-based architecture is:

```
┌───────────────────────┐
│      Application      │
│                       │
│  Java / Node / Python │
└───────────┬───────────┘
            |
            ↓
       /metrics
            |
            ↓
     ┌─────────────┐
     │ Prometheus  │
     └──────┬──────┘
            |
            ↓
         PromQL
            |
            ↓
     ┌─────────────┐
     │   Grafana   │
     └──────┬──────┘
            |
     +------+------+
     |             |
     ↓             ↓
  Dashboard      Alerts
```

---

# 8. Prometheus

Prometheus is a monitoring and time-series database system commonly
used in cloud-native environments.

It can:

```
Collect Metrics
Store Metrics
Query Metrics
Aggregate Metrics
Evaluate Alert Rules
```

Prometheus uses PromQL for querying.

---

# 9. Prometheus Data Model

Prometheus identifies a time series using:

```
Metric Name
+
Label Set
```

Example:

```
http_requests_total{
    service="order-service",
    method="GET",
    status="200"
}
```

This represents one time series.

Another:

```
http_requests_total{
    service="order-service",
    method="POST",
    status="200"
}
```

is a different time series.

---

# 10. Time Series

A time series is a sequence of values recorded over time.

Example:

```
Time        CPU
----------------
10:00       40%
10:01       42%
10:02       45%
10:03       50%
10:04       58%
```

The series shows how CPU usage changes over time.

---

# 11. Metric Name

Metric names should clearly describe what is being measured.

Examples:

```
http_requests_total

process_cpu_seconds_total

application_errors_total

http_request_duration_seconds
```

A good metric name should make its purpose clear.

---

# 12. Metric Naming Principles

Good names should:

```
Describe the measurement
Use consistent terminology
Include units where appropriate
Follow Prometheus naming conventions
```

Examples:

```
request_duration_seconds

memory_usage_bytes

requests_total
```

Avoid unclear names such as:

```
metric1

value

app_metric
```

---

# 13. Units

Metrics should use consistent units.

Examples:

```
seconds
bytes
bytes_per_second
```

Example:

```
http_request_duration_seconds
```

Instead of:

```
http_request_duration_milliseconds
```

using seconds follows common Prometheus conventions for duration
metrics.

The display layer can convert units for users.

---

# 14. Metric Labels

Labels provide additional dimensions.

Example:

```
http_requests_total{
    service="order-service",
    method="GET",
    status="200"
}
```

Labels allow queries such as:

```
All requests

GET requests

POST requests

5xx requests

Requests for a specific service
```

---

# 15. Label Example

Suppose:

```
http_requests_total
```

has:

```
service
method
status
```

Then:

```
service="order-service"
```

identifies the service.

```
method="GET"
```

identifies the HTTP method.

```
status="500"
```

identifies the response status.

---

# 16. Label Cardinality

Cardinality means the number of unique combinations of label values.

Example:

```
service
method
status
```

usually has manageable cardinality.

But consider:

```
user_id
```

If there are millions of users:

```
user_id="12345"
user_id="12346"
user_id="12347"
...
```

this can create a huge number of time series.

---

# 17. Why High Cardinality Is Dangerous

High-cardinality labels can cause:

```
Increased Memory Usage
Increased Storage
Higher Query Cost
Slower Queries
Prometheus Performance Problems
```

Avoid arbitrary user-controlled values as metric labels.

Bad example:

```
http_requests_total{
    user_id="123456"
}
```

Better:

```
http_requests_total{
    service="order-service",
    route="/orders/{id}"
}
```

---

# 18. Good Labels

Good metric dimensions commonly include:

```
service
environment
namespace
method
route
status
region
```

These should still be reviewed for cardinality.

---

# 19. Bad Labels

Avoid labels such as:

```
user_id
request_id
session_id
email
UUID
random token
full URL with IDs
```

These values are often highly unique.

Use logs or traces for high-cardinality investigation data.

---

# 20. Metrics and Metadata

Metrics should provide enough context for operational questions.

Example:

```
http_requests_total{
    service="payment-service",
    environment="production",
    method="POST",
    route="/payments",
    status="500"
}
```

This allows filtering by:

```
Service
Environment
Route
Status
```

---

# 21. Counter

A Counter is a metric that only increases, except when the process
restarts.

Examples:

```
Total Requests
Total Errors
Total Jobs Processed
Total Transactions
```

Example:

```
http_requests_total
```

Values:

```
100
150
200
250
```

Counters are useful for measuring cumulative events.

---

# 22. Counter Reset

Counters may reset when an application restarts.

Example:

```
Before restart:

http_requests_total = 50000

Application restarts.

After restart:

http_requests_total = 0
```

This is expected.

Prometheus functions such as:

```
rate()
```

and:

```
increase()
```

handle counter behavior appropriately.

---

# 23. Counter Example

Suppose:

```
http_requests_total
```

changes from:

```
10,000
```

to:

```
10,500
```

during five minutes.

The increase is:

```
500 requests
```

The average request rate is approximately:

```
500 / 300

= 1.67 requests/sec
```

---

# 24. Counter Use Cases

Counters are appropriate for:

```
Requests
Errors
Transactions
Messages
Jobs
Restarts
Authentication Attempts
```

Examples:

```
requests_total

errors_total

payments_processed_total

messages_received_total
```

---

# 25. Gauge

A Gauge represents a value that can increase or decrease.

Examples:

```
CPU Usage
Memory Usage
Active Connections
Queue Depth
Current Temperature
```

Example:

```
active_connections
```

Values:

```
50
75
100
80
60
```

---

# 26. Gauge Use Cases

Use gauges for current state.

Examples:

```
active_sessions

queue_depth

memory_usage_bytes

cpu_utilization

active_connections

running_jobs
```

---

# 27. Counter vs Gauge

Counter:

```
Monotonically increasing events.
```

Examples:

```
Requests
Errors
Transactions
```

Gauge:

```
Current state.
```

Examples:

```
CPU
Memory
Connections
Queue Depth
```

Simple rule:

```
"How many have happened?"
    ↓
  Counter

"What is the current value?"
    ↓
   Gauge
```

---

# 28. Histogram

A Histogram measures observations and groups them into buckets.

Common use case:

```
Request Latency
```

Example:

```
http_request_duration_seconds
```

A histogram can provide:

```
Count
Sum
Bucket Counts
```

It is especially useful for calculating percentiles.

---

# 29. Histogram Example

Suppose request latency observations are:

```
50ms
100ms
200ms
300ms
700ms
```

Buckets might be:

```
0.1s
0.25s
0.5s
1s
```

Prometheus records how many observations fall into each bucket.

---

# 30. Histogram Components

A Prometheus histogram creates related metrics such as:

```
request_duration_seconds_bucket

request_duration_seconds_sum

request_duration_seconds_count
```

The bucket metric represents cumulative counts.

The sum represents total observed duration.

The count represents total observations.

---

# 31. Histogram and Percentiles

Histograms can be used with:

```
histogram_quantile()
```

Example:

```
histogram_quantile(
  0.95,
  rate(
    http_request_duration_seconds_bucket[5m]
  )
)
```

This estimates P95 latency.

---

# 32. Histogram vs Gauge for Latency

A gauge can represent a current or sampled latency value, but it is
usually not enough to understand the full distribution.

A histogram can represent:

```
Fast Requests
Normal Requests
Slow Requests
Tail Requests
```

Therefore, histograms are commonly preferred for request latency.

---

# 33. Summary

A Summary also records observations and can provide quantiles.

It typically exposes:

```
Quantiles
Sum
Count
```

Summaries calculate quantiles on the client side.

---

# 34. Histogram vs Summary

Histogram:

```
Buckets
Server-side quantile calculation
Aggregation-friendly
```

Summary:

```
Client-side quantiles
Less suitable for aggregation across instances
```

For distributed systems, histograms are often preferred when
aggregation across multiple application instances is required.

---

# 35. Metric Type Overview

The four major Prometheus metric types are:

```
Counter
Gauge
Histogram
Summary
```

Quick reference:

```
Counter
→ Cumulative events

Gauge
→ Current value

Histogram
→ Distribution using buckets

Summary
→ Distribution with client-side quantiles
```

---

# 36. Metric Collection

Metrics can be collected through different approaches.

Common approaches include:

```
Pull
Push
Exporters
Application Instrumentation
OpenTelemetry
```

Prometheus primarily uses a pull-based model.

---

# 37. Pull Model

In the pull model:

```
Prometheus
    |
    ↓
HTTP Request
    |
    ↓
Application /metrics
    |
    ↓
Metrics Response
```

Prometheus periodically scrapes the target.

---

# 38. Push Model

In a push model:

```
Application
    |
    ↓
Push Metrics
    |
    ↓
Metrics Backend
```

Push-based systems can be useful in specific architectures.

Prometheus itself generally expects scraping rather than direct
application pushes.

---

# 39. Why Prometheus Uses Pull

The pull model provides several advantages:

```
Easy Target Discovery
Centralized Scraping
Target Health Visibility
Simple Debugging
Consistent Collection
```

Prometheus can determine whether a target is reachable.

---

# 40. Prometheus Scrape

A scrape is the process of Prometheus retrieving metrics from a target.

Example:

```
Prometheus
    |
    | GET /metrics
    ↓
Application
```

Response:

```
http_requests_total 10000
```

Prometheus stores the result as time-series data.

---

# 41. Scrape Interval

Prometheus periodically scrapes targets.

Example:

```
scrape_interval: 15s
```

This means Prometheus attempts to collect metrics every 15 seconds.

A shorter interval provides more frequent observations but increases
collection load and storage requirements.

---

# 42. Scrape Timeout

Example:

```
scrape_timeout: 10s
```

If the target does not respond within the timeout, the scrape fails.

Monitor:

```
Scrape Success
Scrape Duration
Target Availability
```

---

# 43. Prometheus Configuration

A simplified configuration:

```
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "application"
    static_configs:
      - targets:
          - "application:8080"
```

Prometheus then scrapes:

```
http://application:8080/metrics
```

---

# 44. Static Target Discovery

Example:

```
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
```

This is simple but becomes difficult to maintain in dynamic
environments.

---

# 45. Kubernetes Service Discovery

Kubernetes environments are dynamic.

Pods can:

```
Start
Stop
Move
Scale
Restart
```

Prometheus can use Kubernetes service discovery to dynamically find
targets.

Example:

```
Kubernetes API
      |
      ↓
  Prometheus
      |
      ↓
   Targets
      |
      ↓
    Scrape
```

---

# 46. Kubernetes Metrics Collection

In Kubernetes, Prometheus can discover:

```
Pods
Services
Nodes
Endpoints
ServiceMonitors
PodMonitors
```

depending on the monitoring setup.

---

# 47. ServiceMonitor

In Prometheus Operator-based environments, a ServiceMonitor can define
how a Kubernetes service should be monitored.

Conceptually:

```
ServiceMonitor
     |
     ↓
Kubernetes Service
     |
     ↓
/metrics
     |
     ↓
Prometheus
```

---

# 48. PodMonitor

A PodMonitor can define scraping directly against pods.

Conceptually:

```
PodMonitor
    |
    ↓
  Pods
    |
    ↓
/metrics
    |
    ↓
Prometheus
```

This can be useful when direct pod scraping is preferred.

---

# 49. Exporters

Exporters expose metrics from systems that do not natively provide
Prometheus-compatible metrics.

Examples:

```
Node Exporter
Database Exporter
Blackbox Exporter
```

Architecture:

```
System
   |
   ↓
Exporter
   |
   ↓
/metrics
   |
   ↓
Prometheus
```

---

# 50. Node Exporter

Node Exporter provides system-level metrics.

Examples:

```
CPU
Memory
Disk
Filesystem
Network
Load
```

Architecture:

```
Linux Node
    |
    ↓
Node Exporter
    |
    ↓
Prometheus
```

---

# 51. Application Instrumentation

Applications can expose custom metrics.

Example:

```
Java Application
     |
     ↓
Metrics Library
     |
     ↓
/metrics
     |
     ↓
Prometheus
```

This allows application-specific monitoring.

---

# 52. Business Metrics

Technical metrics are not always enough.

Examples:

```
Orders Created
Payments Completed
Cart Checkouts
Successful Transactions
Failed Payments
```

Example:

```
orders_created_total
```

This can provide direct business visibility.

---

# 53. Application Metrics

Typical application metrics:

```
http_requests_total

http_request_duration_seconds

application_errors_total

active_connections

queue_depth
```

These metrics can be used for:

```
Dashboards
Alerts
Capacity Planning
SLOs
```

---

# 54. RED Method

The RED method is commonly used for request-driven services.

RED means:

```
Rate
Errors
Duration
```

Rate:

```
Requests per second
```

Errors:

```
Failed requests
```

Duration:

```
Request latency
```

Example:

```
Rate = 5,000 req/sec

Error Rate = 0.5%

P95 Duration = 250ms
```

---

# 55. USE Method

The USE method is commonly applied to infrastructure resources.

USE means:

```
Utilization
Saturation
Errors
```

Example for CPU:

```
Utilization = 70%

Saturation = Run Queue High

Errors = Hardware Errors
```

For infrastructure monitoring, USE can complement application-focused
methods such as RED.

---

# 56. RED vs USE

RED focuses primarily on services:

```
Rate
Errors
Duration
```

USE focuses primarily on resources:

```
Utilization
Saturation
Errors
```

Both approaches can be used together.

---

# 57. Golden Signals vs RED

Golden Signals:

```
Traffic
Latency
Errors
Saturation
```

RED:

```
Rate
Errors
Duration
```

They overlap significantly.

RED is especially useful for request-driven services.

Golden Signals provide a broader service-health model by explicitly
including saturation.

---

# 58. Infrastructure Metrics

Important infrastructure metrics include:

```
CPU
Memory
Disk
Network
Load
Filesystem
Process Count
```

Example:

```
node_cpu_seconds_total

node_memory_MemAvailable_bytes

node_filesystem_avail_bytes
```

---

# 59. Kubernetes Metrics

Important Kubernetes metrics include:

```
Pod CPU
Pod Memory
Pod Restarts
Node CPU
Node Memory
Desired Replicas
Available Replicas
HPA Status
```

These metrics help identify:

```
Resource Pressure
Scaling Problems
Scheduling Problems
Application Instability
```

---

# 60. Container Metrics

Monitor:

```
CPU Usage
Memory Usage
Network
Filesystem
Restart Count
```

Example:

```
Container
   |
   +--- CPU
   +--- Memory
   +--- Network
   +--- Restarts
```

---

# 61. Database Metrics

Important database metrics include:

```
Connections
Query Rate
Query Latency
CPU
Memory
Storage
IOPS
Locks
Errors
```

Example:

```
active_db_connections
```

A database can be the bottleneck even when application CPU is low.

---

# 62. Message Queue Metrics

For RabbitMQ or similar systems:

```
Messages Published
Messages Consumed
Queue Depth
Consumer Count
Processing Rate
Consumer Errors
```

Example:

```
queue_depth
```

If queue depth continuously increases, consumers may not be keeping
up.

---

# 63. Load Balancer Metrics

For an ALB, useful metrics include:

```
Request Count
Target Response Time
HTTP 4xx
HTTP 5xx
Healthy Targets
Unhealthy Targets
```

These metrics help identify issues at the ingress/load-balancing
layer.

---

# 64. Metrics for External APIs

Monitor:

```
Request Count
Success Rate
Error Rate
Latency
Timeout Count
Retry Count
```

Example:

```
payment_api_requests_total

payment_api_errors_total

payment_api_duration_seconds
```

---

# 65. Metrics and SLOs

Metrics provide the data required to calculate SLOs.

Example:

```
Availability SLO = 99.9%
```

Metrics:

```
Total Requests
Successful Requests
Failed Requests
```

Availability can be calculated from these metrics.

---

# 66. Error Rate Metric

Conceptually:

```
Error Rate =
Failed Requests
---------------
Total Requests
```

Example:

```
Failed = 100

Total = 10,000

Error Rate = 1%
```

This can be visualized in Grafana and used in alerting.

---

# 67. Latency Metrics

Latency metrics should ideally provide distributions.

Useful values:

```
P50
P90
P95
P99
```

Example:

```
P50 = 100ms
P95 = 300ms
P99 = 900ms
```

This provides more information than a single average.

---

# 68. Throughput Metrics

Throughput measures how much work the system processes.

Examples:

```
Requests/sec
Transactions/sec
Messages/sec
Jobs/sec
```

Example:

```
payment_transactions_total
```

combined with:

```
rate(payment_transactions_total[5m])
```

can estimate processing rate.

---

# 69. Availability Metrics

Availability can be measured using:

```
Successful Requests
Failed Requests
```

Example:

```
Total = 1,000,000

Failed = 500
```

Availability:

```
99.95%
```

This can support an availability SLO.

---

# 70. Saturation Metrics

Examples:

```
CPU Utilization
Memory Utilization
Disk Usage
Connection Pool Usage
Queue Depth
```

The right metric depends on the bottleneck.

---

# 71. Metric Aggregation

Metrics can be aggregated across dimensions.

Example:

```
Requests by service:

service A = 1,000/sec
service B = 2,000/sec
service C = 3,000/sec
```

Total:

```
6,000/sec
```

PromQL can perform this aggregation.

---

# 72. PromQL Aggregation

Example:

```
sum(
  rate(http_requests_total[5m])
)
```

This calculates total request rate across matching time series.

---

# 73. Aggregation by Service

Example:

```
sum(
  rate(http_requests_total[5m])
) by (service)
```

This produces request rate per service.

---

# 74. Aggregation by Status

Example:

```
sum(
  rate(http_requests_total[5m])
) by (status)
```

This can show:

```
200
400
404
500
503
```

request rates.

---

# 75. Filtering Metrics

PromQL supports label filtering.

Example:

```
http_requests_total{
  service="payment-service"
}
```

Another:

```
http_requests_total{
  status=~"5.."
}
```

This selects server-side error responses.

---

# 76. Exact Label Matching

Example:

```
service="payment-service"
```

This matches exactly.

---

# 77. Regex Matching

Example:

```
service=~"payment|order"
```

This matches services such as:

```
payment
```

or:

```
order
```

Regex should be used carefully because broad queries can increase
query cost.

---

# 78. rate()

The rate function calculates the per-second average increase of a
counter over a time range.

Example:

```
rate(http_requests_total[5m])
```

This is commonly used for request rate.

---

# 79. increase()

The increase function calculates how much a counter increased during a
time range.

Example:

```
increase(
  http_requests_total[1h]
)
```

This can show total requests during the last hour.

---

# 80. irate()

irate estimates the per-second rate based on the most recent samples.

Example:

```
irate(http_requests_total[5m])
```

It can be useful for rapidly changing counters but may be more noisy
than rate().

For most alerting and dashboards, rate() is often preferred.

---

# 81. sum()

Example:

```
sum(
  rate(http_requests_total[5m])
)
```

This aggregates matching series.

---

# 82. avg()

Example:

```
avg(
  node_cpu_utilization
)
```

This calculates an average across matching series.

Be careful with averages because they can hide important outliers.

---

# 83. max()

Example:

```
max(
  container_memory_usage_bytes
)
```

This can identify the highest observed value across matching series.

---

# 84. min()

Example:

```
min(
  available_disk_space_bytes
)
```

This can identify the lowest available disk capacity.

---

# 85. histogram_quantile()

Example:

```
histogram_quantile(
  0.95,
  sum(
    rate(
      http_request_duration_seconds_bucket[5m]
    )
  ) by (le)
)
```

This estimates P95 latency from histogram buckets.

---

# 86. Metric Recording Rules

Complex PromQL queries can be expensive when executed repeatedly.

Recording rules can precompute frequently used expressions.

Example:

```
service:http_requests_per_second
```

This can then be queried directly by dashboards or alerts.

---

# 87. Why Use Recording Rules?

Benefits:

```
Faster Queries
Reduced Query Load
Consistent Calculations
Easier Dashboards
Easier Alerts
```

Example:

```
Complex Query
     |
     ↓
Recording Rule
     |
     ↓
Precomputed Metric
     |
     ↓
Grafana
```

---

# 88. Alerting Rules

Prometheus can evaluate alerting rules.

Example:

```
Alert:

HighErrorRate
```

Condition:

```
error_rate > 0.05
```

for:

```
5m
```

The alert can then be routed to the organization's notification
system.

---

# 89. Alert Example

Conceptual rule:

```
alert: HighErrorRate

expr:
  error_rate > 0.05

for:
  5m

labels:
  severity: critical

annotations:
  summary: "High error rate"
```

The exact implementation depends on the Prometheus configuration.

---

# 90. Metrics and Grafana

Grafana can visualize:

```
Time Series
Gauges
Tables
Stat Panels
Heatmaps
Histograms
```

Useful dashboards include:

```
Infrastructure
Kubernetes
Application
Microservices
SLO
Business Metrics
```

---

# 91. Service Dashboard

A production service dashboard may contain:

```
Request Rate
Error Rate
P50
P95
P99
CPU
Memory
Restarts
Connections
Queue Depth
```

Example:

```
┌──────────────────────────────────┐
│ ORDER SERVICE                    │
├──────────────────────────────────┤
│ Request Rate    5,000 req/sec    │
│ Error Rate      0.4%             │
│ P95             250ms            │
│ P99             600ms            │
│ CPU             55%              │
│ Memory          60%              │
└──────────────────────────────────┘
```

---

# 92. Metrics During Deployment

Before deployment:

```
Capture baseline.
```

During deployment:

```
Monitor metrics.
```

After deployment:

```
Compare with baseline.
```

Example:

```
Before:
P95 = 200ms
Error = 0.2%

After:
P95 = 800ms
Error = 3%
```

This may indicate a deployment regression.

---

# 93. Metrics During Canary Deployment

Example:

```
Version A = 95%
Version B = 5%
```

Compare:

```
Version A:
P95 = 200ms
Error = 0.2%

Version B:
P95 = 700ms
Error = 3%
```

Version B should be investigated before increasing traffic.

---

# 94. Metrics and Autoscaling

Metrics can drive autoscaling.

Example:

```
Request Rate ↑
     |
     ↓
CPU ↑
     |
     ↓
HPA
     |
     ↓
Replicas ↑
     |
     ↓
CPU ↓
     |
     ↓
Latency ↓
```

Autoscaling can use resource metrics or custom metrics depending on
the architecture.

---

# 95. Custom Metrics

Custom metrics allow scaling based on application-specific behavior.

Examples:

```
Requests per second
Queue depth
Active sessions
Messages waiting
Transactions per second
```

Example:

```
queue_depth > 1000
```

could be a better scaling signal than CPU for a queue-consuming
application.

---

# 96. Metrics and Capacity Planning

Historical metrics can identify trends.

Example:

```
CPU Usage

January = 50%
February = 58%
March = 65%
April = 72%
```

The trend suggests increasing resource requirements.

Metrics can support:

```
Capacity Planning
Scaling Decisions
Infrastructure Sizing
Cost Optimization
```

---

# 97. Metrics and Cost Optimization

Metrics can reveal unused resources.

Example:

```
CPU Request = 2 cores
Average Usage = 0.3 cores
```

This may indicate over-provisioning.

But resource rightsizing should consider:

```
Peak Usage
SLO
Traffic Spikes
Scaling Behavior
```

Do not size solely from average usage.

---

# 98. Metrics and Incident Response

During an incident, metrics provide the initial overview.

Example:

```
Alert
  |
  ↓
Grafana
  |
  +--- Traffic
  +--- Latency
  +--- Errors
  +--- Saturation
  |
  ↓
Identify affected service
  |
  ↓
Logs
  |
  ↓
Traces
```

---

# 99. Metrics and Logs Correlation

Metrics:

```
Error Rate = 8%
```

Then:

```
Search logs for affected service.
```

Example:

```
service="payment-service"
level="ERROR"
```

Result:

```
Database timeout
```

Metrics show scale.

Logs show event details.

---

# 100. Metrics and Traces Correlation

Metric:

```
P99 = 2 seconds
```

Trace:

```
Order
  |
  ↓
Payment
  |
  ↓
External API
  |
  ↓
1.8 seconds
```

Metrics identify the problem.

Traces identify the slow path.

---

# 101. Metric Collection Architecture with OpenTelemetry

OpenTelemetry can also participate in metric collection.

Example:

```
Application
    |
    ↓
OpenTelemetry SDK
    |
    ↓
OTel Collector
    |
    ↓
Metrics Backend
```

Depending on the chosen architecture, Prometheus can remain the
primary metrics system while OpenTelemetry handles instrumentation
and/or collection.

---

# 102. Prometheus + OpenTelemetry

A practical architecture can be:

```
Application
    |
    +--- Prometheus Metrics
    |
    +--- OpenTelemetry Traces
    |
    ↓
Prometheus
    |
    ↓
Grafana
```

and:

```
Application
    |
    ↓
OpenTelemetry
    |
    ↓
Collector
    |
    ↓
Jaeger
```

This allows metrics and traces to be operated through appropriate
backends.

---

# 103. Metric Reliability

Metrics must be trustworthy.

Problems can occur because of:

```
Incorrect Instrumentation
Wrong Units
Wrong Labels
Duplicate Metrics
Missing Metrics
Counter Misuse
Incorrect Aggregation
```

Always validate metric behavior before using it for critical alerts.

---

# 104. Metric Validation

For a new metric:

```
1. Verify the metric name.

2. Verify the type.

3. Verify labels.

4. Verify units.

5. Generate known traffic.

6. Confirm metric changes.

7. Query using PromQL.

8. Verify Grafana.

9. Verify alerts.
```

---

# 105. Testing a Counter

Suppose:

```
orders_created_total
```

Generate:

```
10 orders
```

Expected:

```
Counter increases by approximately 10
```

Then restart the application.

The counter may reset locally.

Prometheus functions should account for this when calculating rates.

---

# 106. Testing a Gauge

Suppose:

```
active_connections
```

Create:

```
20 connections
```

Expected:

```
Gauge ≈ 20
```

Close:

```
10 connections
```

Expected:

```
Gauge ≈ 10
```

---

# 107. Testing a Histogram

Generate requests with known latency ranges.

Example:

```
100 requests around 100ms

50 requests around 500ms

10 requests around 1s
```

Then inspect histogram buckets and percentile calculations.

---

# 108. Missing Metrics

Possible causes:

```
Application Not Exposing /metrics
Incorrect Port
Incorrect Path
Network Policy
Service Discovery Failure
Prometheus Configuration Error
Authentication
Target Down
```

Troubleshooting flow:

```
Application
    |
    ↓
/metrics
    |
    ↓
Network
    |
    ↓
Prometheus Target
    |
    ↓
PromQL
    |
    ↓
Grafana
```

---

# 109. Prometheus Target Down

If a target is down:

```
Check target address.

Check application port.

Check /metrics endpoint.

Check Kubernetes Service.

Check NetworkPolicy.

Check DNS.

Check Prometheus discovery.

Check scrape logs.
```

---

# 110. Scrape Failure

Possible reasons:

```
Timeout
Connection Refused
DNS Failure
HTTP Error
TLS Error
Authentication Failure
```

Prometheus target health can help identify the problem.

---

# 111. Metrics Duplication

Duplicate scraping can produce confusing results.

Example:

```
Target A
    |
    ↓
Prometheus
```

and:

```
Target A
    |
    ↓
Prometheus
```

through two discovery mechanisms.

This may cause duplicated time series or unexpected aggregation.

Review scrape configuration carefully.

---

# 112. Duplicate Time Series

Duplicate series can happen when label sets collide or multiple
scrapers collect the same metric without distinguishing labels.

Avoid ambiguous monitoring configurations.

Use appropriate labels such as:

```
job
instance
cluster
environment
```

where appropriate.

---

# 113. Metric Naming Mistakes

Bad:

```
cpu
```

Better:

```
process_cpu_seconds_total
```

Bad:

```
latency
```

Better:

```
http_request_duration_seconds
```

Clear naming improves usability.

---

# 114. Unit Mistakes

Bad:

```
request_latency = 250
```

What is 250?

```
milliseconds?
seconds?
microseconds?
```

Better:

```
request_duration_seconds
```

The unit is explicit.

---

# 115. Label Mistakes

Bad:

```
request_id
```

Better:

```
route
method
status
service
```

High-cardinality information should generally be handled in logs and
traces instead.

---

# 116. Over-Instrumentation

Collecting every possible metric can create:

```
High Storage
High Memory
High Query Cost
High Operational Complexity
```

Monitor what is operationally useful.

---

# 117. Under-Instrumentation

Collecting too few metrics can make troubleshooting difficult.

At minimum, critical services should provide useful measurements for:

```
Traffic
Latency
Errors
Saturation
```

---

# 118. Metrics Best Practices

```
1. Use clear metric names.

2. Use correct metric types.

3. Use consistent units.

4. Avoid high-cardinality labels.

5. Use counters for cumulative events.

6. Use gauges for current state.

7. Use histograms for distributions.

8. Prefer histograms when aggregation across instances is needed.

9. Monitor Golden Signals.

10. Use RED for request-driven services.

11. Use USE for infrastructure.

12. Create recording rules for expensive repeated queries.

13. Use meaningful alerts.

14. Validate metrics before production.

15. Monitor the metrics platform itself.
```

---

# 119. Production Metrics Checklist

```
[ ] Application metrics implemented
[ ] Infrastructure metrics implemented
[ ] Kubernetes metrics implemented
[ ] Database metrics implemented
[ ] Queue metrics implemented
[ ] Load balancer metrics implemented
[ ] Golden Signals available
[ ] RED metrics available
[ ] USE metrics available
[ ] Metric names standardized
[ ] Units standardized
[ ] Labels reviewed
[ ] Cardinality reviewed
[ ] Prometheus scraping configured
[ ] Service discovery configured
[ ] Recording rules configured
[ ] Alert rules configured
[ ] Grafana dashboards created
[ ] SLO metrics defined
[ ] Capacity metrics available
```

---

# 120. Production Metric Architecture

```
┌────────────────────────────────────────────────────────┐
│                    Production                           │
│                                                        │
│  EKS | EC2 | Applications | Databases | Queues | ALB │
└──────────────────────────┬─────────────────────────────┘
                           |
         +-----------------+-----------------+
         |                 |                 |
         ↓                 ↓                 ↓
    App Metrics      Infrastructure      Exporters
         |               Metrics             |
         |                 |                 |
         +-----------------+-----------------+
                           |
                           ↓
                     Prometheus
                           |
                     +-----+-----+
                     |           |
                     ↓           ↓
                 PromQL       Rules
                     |           |
                     ↓           ↓
                  Grafana     Alerts
                     |
                     ↓
              Incident Response
```

---

# 121. Metrics Investigation Model

When a metric becomes abnormal:

```
Metric Alert
     |
     ↓
Confirm Metric
     |
     ↓
Check Time Range
     |
     ↓
Check Labels
     |
     ↓
Compare Baseline
     |
     ↓
Check Related Metrics
     |
     ↓
Check Logs
     |
     ↓
Check Traces
     |
     ↓
Identify Root Cause
```

---

# 122. Example Production Investigation

Alert:

```
HighErrorRate
```

First query:

```
Error Rate = 8%
```

Next:

```
Traffic = 10,000 req/sec
```

Then:

```
CPU = 50%

Memory = 60%
```

No obvious resource saturation.

Next:

```
Check latency.

P95 = 2 seconds
```

Then:

```
Search logs.
```

Result:

```
Database timeout
```

Then:

```
Check database metrics.
```

Result:

```
Connections = 99%
```

Root cause:

```
Database connection saturation.
```

---

# 123. Another Production Investigation

Alert:

```
HighCPU
```

Metrics:

```
CPU = 95%

Traffic = 3x normal

Error Rate = 4%

P95 = 1.5 seconds
```

Interpretation:

```
Increased traffic is causing resource saturation.
```

Possible mitigation:

```
Scale out.
```

Then validate:

```
CPU ↓

Error Rate ↓

Latency ↓
```

---

# 124. Metrics and Capacity Planning Example

Historical data:

```
January:
4,000 req/sec

February:
5,000 req/sec

March:
6,500 req/sec

April:
8,000 req/sec
```

CPU:

```
January:
45%

April:
78%
```

The trend suggests capacity planning may be required.

Possible actions:

```
Optimize Application
Scale Horizontally
Increase Capacity
Review Database
Review Caching
```

---

# 125. Metrics and Cost Optimization Example

Suppose a Kubernetes deployment has:

```
CPU Request = 2 cores
```

Average usage:

```
0.4 cores
```

Peak:

```
1.2 cores
```

Before reducing the request, consider:

```
Peak Traffic
SLO
Scaling
Burst Behavior
Scheduling
```

Metrics should guide rightsizing rather than averages alone.

---

# 126. Interview Questions

## What is a metric?

### Answer

A metric is a numerical measurement collected over time that
represents the state or behavior of a system.

Examples include:

```
CPU
Memory
Request Rate
Error Rate
Latency
Active Connections
```

Metrics are useful for dashboards, alerting, capacity planning, and
SLO measurement.

---

# 127. What is a time series?

### Answer

A time series is a sequence of metric values recorded over time.

For example:

```
10:00 → 50%
10:01 → 55%
10:02 → 60%
```

The combination of a metric name and label set identifies a specific
Prometheus time series.

---

# 128. What are the four Prometheus metric types?

### Answer

The four main Prometheus metric types are:

```
Counter
Gauge
Histogram
Summary
```

Counter measures cumulative events.

Gauge measures current state.

Histogram measures distributions using buckets.

Summary records observations and can expose quantiles calculated by the
client.

---

# 129. What is the difference between Counter and Gauge?

### Answer

A Counter represents a cumulative value that normally increases.

Examples:

```
Requests
Errors
Transactions
```

A Gauge represents a value that can increase or decrease.

Examples:

```
CPU
Memory
Queue Depth
Active Connections
```

---

# 130. When would you use a Histogram?

### Answer

I would use a histogram when I need to measure the distribution of
observations, especially request latency.

For example:

```
P50
P95
P99
```

Histograms are also useful when aggregating observations across
multiple application instances.

---

# 131. Histogram vs Summary?

### Answer

A Histogram stores observations in buckets and allows quantiles to be
calculated from aggregated data.

A Summary calculates quantiles on the client side.

For distributed systems, histograms are often more useful because
histogram data can be aggregated across instances.

---

# 132. What is metric cardinality?

### Answer

Cardinality is the number of unique time-series combinations created
by metric labels.

For example:

```
service
method
status
```

usually has manageable cardinality.

But:

```
user_id
```

can create millions of unique combinations.

High cardinality can cause memory, storage, and query performance
problems.

---

# 133. Why should request_id not usually be a metric label?

### Answer

A request ID is usually unique for every request.

Using it as a metric label creates a huge number of unique time series.

This increases:

```
Memory Usage
Storage
Query Cost
Prometheus Load
```

Request IDs are better suited to logs and traces.

---

# 134. What is Prometheus?

### Answer

Prometheus is a monitoring and time-series database system.

It collects metrics, stores time-series data, provides PromQL for
querying, and supports alert rule evaluation.

It is widely used for Kubernetes and cloud-native monitoring.

---

# 135. How does Prometheus collect metrics?

### Answer

Prometheus primarily uses a pull model.

It periodically scrapes HTTP endpoints such as:

```
/metrics
```

The architecture is:

```
Prometheus
    |
    ↓
HTTP Scrape
    |
    ↓
Application / Exporter
```

---

# 136. What is a scrape?

### Answer

A scrape is when Prometheus retrieves metrics from a target.

For example:

```
Prometheus
    |
    ↓
GET /metrics
    |
    ↓
Application
```

Prometheus stores the returned metrics as time series.

---

# 137. What is scrape interval?

### Answer

Scrape interval determines how frequently Prometheus collects metrics.

Example:

```
scrape_interval: 15s
```

means Prometheus attempts to scrape the target every 15 seconds.

Shorter intervals provide more frequent observations but increase
collection and storage costs.

---

# 138. What is ServiceMonitor?

### Answer

ServiceMonitor is a Prometheus Operator resource used to define how
Kubernetes services should be monitored.

Conceptually:

```
ServiceMonitor
     |
     ↓
Kubernetes Service
     |
     ↓
/metrics
     |
     ↓
Prometheus
```

---

# 139. What is an exporter?

### Answer

An exporter exposes metrics from a system in a format Prometheus can
scrape.

Examples:

```
Node Exporter
Database Exporter
Blackbox Exporter
```

Architecture:

```
System
   |
   ↓
Exporter
   |
   ↓
Prometheus
```

---

# 140. What is Node Exporter?

### Answer

Node Exporter exposes Linux system metrics such as:

```
CPU
Memory
Disk
Network
Filesystem
```

Prometheus scrapes these metrics for infrastructure monitoring.

---

# 141. What is PromQL?

### Answer

PromQL is Prometheus's query language.

It is used to:

```
Filter Metrics
Aggregate Metrics
Calculate Rates
Calculate Percentiles
Build Dashboards
Create Alerts
```

Example:

```
rate(http_requests_total[5m])
```

---

# 142. What does rate() do?

### Answer

rate() calculates the average per-second increase of a counter over a
specified time range.

Example:

```
rate(http_requests_total[5m])
```

This can be used to calculate requests per second.

---

# 143. What does increase() do?

### Answer

increase() calculates the total increase of a counter over a selected
time range.

Example:

```
increase(http_requests_total[1h])
```

This can show how many requests occurred during the last hour.

---

# 144. What is a recording rule?

### Answer

A recording rule precomputes a PromQL expression and stores the result
as a new time series.

It is useful for:

```
Expensive Queries
Frequently Used Queries
Dashboards
Alerts
```

This can improve query performance.

---

# 145. How do you monitor Kubernetes with Prometheus?

### Answer

I would collect metrics from:

```
Nodes
Pods
Containers
Kubernetes Components
Applications
Exporters
```

I would use Kubernetes service discovery and, where appropriate,
Prometheus Operator resources such as ServiceMonitor and PodMonitor.

Then I would visualize the metrics using Grafana.

---

# 146. How do you monitor an application?

### Answer

I would expose application metrics such as:

```
Request Rate
Error Rate
Latency
Active Connections
Business Transactions
```

Then expose them through a metrics endpoint or appropriate
instrumentation.

Prometheus collects the metrics and Grafana visualizes them.

---

# 147. How do you monitor microservices?

### Answer

For every important service I would monitor:

```
Traffic
Latency
Errors
Saturation
```

Then add:

```
Dependencies
Business Metrics
Logs
Traces
```

I would standardize service and environment labels so the metrics can
be filtered and correlated.

---

# 148. How do you troubleshoot missing metrics?

### Answer

I would check the complete path:

```
Application
   ↓
/metrics
   ↓
Network
   ↓
Prometheus Target
   ↓
Scrape Status
   ↓
PromQL
   ↓
Grafana
```

I would verify the endpoint, port, service discovery, network
connectivity, scrape configuration, and query.

---

# 149. How do you reduce Prometheus memory usage?

### Answer

I would first investigate cardinality.

Then:

```
Remove unnecessary labels
Avoid high-cardinality labels
Reduce unnecessary metrics
Review scrape intervals
Review retention
Optimize queries
Use recording rules
Review target configuration
```

I would avoid blindly reducing collection frequency without
understanding the operational impact.

---

# 150. How do you monitor latency?

### Answer

I would use a histogram-based duration metric.

Example:

```
http_request_duration_seconds
```

Then calculate:

```
P50
P95
P99
```

using PromQL.

For example:

```
histogram_quantile(
  0.95,
  rate(
    http_request_duration_seconds_bucket[5m]
  )
)
```

---

# 151. How do you monitor error rate?

### Answer

I would use a counter for requests and errors.

Conceptually:

```
Error Rate =
Failed Requests
---------------
Total Requests
```

In PromQL, I can calculate the rate of error requests and divide it by
the total request rate.

---

# 152. How do metrics help autoscaling?

### Answer

Metrics provide signals that can trigger scaling.

For example:

```
CPU
Memory
Request Rate
Queue Depth
```

For a queue-based application, queue depth may be a better scaling
signal than CPU.

---

# 153. What is the RED method?

### Answer

RED stands for:

```
Rate
Errors
Duration
```

It is useful for monitoring request-driven services.

Rate tells us how much traffic the service receives.

Errors tell us how many requests fail.

Duration tells us how long requests take.

---

# 154. What is the USE method?

### Answer

USE stands for:

```
Utilization
Saturation
Errors
```

It is commonly used for infrastructure and resource monitoring.

For example:

```
CPU Utilization
CPU Saturation
CPU Errors
```

---

# 155. How would you design a production metrics architecture?

### Answer

I would use:

```
Applications
    |
    ↓
Metrics Endpoints / Exporters
    |
    ↓
Prometheus
    |
    ↓
PromQL
    |
    ↓
Grafana
    |
    +--- Dashboards
    |
    +--- Alerts
```

I would include:

```
Kubernetes Metrics
Infrastructure Metrics
Application Metrics
Database Metrics
Queue Metrics
Load Balancer Metrics
```

I would also implement cardinality controls, retention, security,
backup, and high availability according to production requirements.

---

# 156. Final Metrics Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Production                           │
│                                                         │
│  EKS | EC2 | Applications | RDS | Queues | ALB        │
└───────────────────────────┬─────────────────────────────┘
                            |
         +------------------+------------------+
         |                  |                  |
         ↓                  ↓                  ↓
   Application         Infrastructure      Exporters
      Metrics             Metrics              |
         |                  |                  |
         +------------------+------------------+
                            |
                            ↓
                       Prometheus
                            |
             +--------------+--------------+
             |                             |
             ↓                             ↓
          PromQL                        Rules
             |                             |
             ↓                             ↓
          Grafana                       Alerts
             |
      +------+------+
      |             |
      ↓             ↓
  Dashboards    Investigation
                    |
                    ↓
                   Logs
                    |
                    ↓
                  Traces
```

---

# 157. Final Metrics Mental Model

Think about metrics in this order:

```
What are we measuring?
        |
        ↓
What metric type should we use?
        |
        ↓
What labels are required?
        |
        ↓
What is the cardinality?
        |
        ↓
How will the metric be collected?
        |
        ↓
Where will it be stored?
        |
        ↓
How will it be queried?
        |
        ↓
How will it be visualized?
        |
        ↓
Should it trigger an alert?
        |
        ↓
What action will the alert cause?
```

The goal of metrics is not to collect as many numbers as possible.

The goal is to collect the right measurements that allow engineers to:

```
Detect Problems
    ↓
Understand System Behavior
    ↓
Troubleshoot Incidents
    ↓
Measure Reliability
    ↓
Plan Capacity
    ↓
Optimize Cost
    ↓
Improve Production Systems
```
