# Metric Types

Metric types define how measurements are represented, stored, interpreted,
and queried.

Choosing the correct metric type is important because the wrong type can
produce incorrect dashboards, misleading alerts, and inaccurate
observability data.

In a Prometheus-based environment, the primary metric types are:

```
Counter
Gauge
Histogram
Summary
```

A practical understanding of these types is essential for:

```
Application Monitoring
Kubernetes Monitoring
Infrastructure Monitoring
SLOs
Alerting
Capacity Planning
Performance Analysis
```

---

# 1. Why Metric Types Matter

Different measurements behave differently.

For example:

```
Total HTTP Requests
```

is continuously increasing.

This should normally be represented as a:

```
Counter
```

But:

```
Current CPU Usage
```

can increase and decrease.

This should normally be represented as a:

```
Gauge
```

Similarly:

```
Request Latency
```

is a distribution of observations.

This is commonly represented using a:

```
Histogram
```

Choosing the correct type allows Prometheus to interpret the metric
properly.

---

# 2. Prometheus Metric Types

The four main Prometheus metric types are:

```
1. Counter
2. Gauge
3. Histogram
4. Summary
```

Quick overview:

```
Counter
    ↓
Cumulative events

Gauge
    ↓
Current state

Histogram
    ↓
Distribution using buckets

Summary
    ↓
Distribution with client-side quantiles
```

---

# 3. Counter

A Counter represents a cumulative value that normally only increases.

Examples:

```
Total Requests
Total Errors
Total Transactions
Total Jobs
Total Messages
Total Payments
```

Example:

```
http_requests_total
```

Values may look like:

```
100
250
500
750
1000
```

The value increases as events occur.

---

# 4. Counter Behavior

A Counter should not normally decrease during the lifetime of a
process.

Example:

```
Request 1
   ↓
Counter = 1

Request 2
   ↓
Counter = 2

Request 3
   ↓
Counter = 3

Request 4
   ↓
Counter = 4
```

---

# 5. Counter and Application Restart

Counters can reset when the application restarts.

Example:

```
Before Restart:

http_requests_total = 50,000
```

Application restarts.

After restart:

```
http_requests_total = 0
```

This is expected behavior.

Prometheus functions such as:

```
rate()
increase()
```

are designed to work with counters and account for resets.

---

# 6. Counter Example

Suppose an application has:

```
http_requests_total = 10,000
```

Five minutes later:

```
http_requests_total = 13,000
```

The counter increased by:

```
3,000 requests
```

Average request rate:

```
3,000
------
 300

= 10 requests/sec
```

PromQL can calculate this using:

```
rate(http_requests_total[5m])
```

---

# 7. Common Counter Metrics

Examples:

```
http_requests_total

http_errors_total

orders_created_total

payments_processed_total

messages_received_total

jobs_completed_total

authentication_failures_total
```

---

# 8. Counter Naming

Counters commonly use:

```
_total
```

Examples:

```
requests_total

errors_total

transactions_total
```

This makes it clear that the metric represents a cumulative count.

---

# 9. Counter Use Cases

Use a Counter when the question is:

```
"How many events have happened?"
```

Examples:

```
How many HTTP requests?

How many errors?

How many payments?

How many messages?

How many jobs completed?

How many database queries?
```

---

# 10. Counter with Labels

Counters can have labels.

Example:

```
http_requests_total{
    service="order-service",
    method="GET",
    status="200"
}
```

Another series:

```
http_requests_total{
    service="order-service",
    method="POST",
    status="200"
}
```

Labels create separate time series.

---

# 11. Counter for HTTP Requests

A common metric is:

```
http_requests_total
```

with labels:

```
service
method
route
status
```

Example:

```
http_requests_total{
    service="payment-service",
    method="POST",
    route="/payments",
    status="200"
}
```

This can be used to calculate:

```
Request Rate
Error Rate
Traffic by Route
Traffic by Status
```

---

# 12. Counter for Errors

Example:

```
http_errors_total
```

PromQL:

```
rate(http_errors_total[5m])
```

This provides the error rate per second.

Another useful approach is to calculate error percentage:

```
Error Rate =
Errors / Total Requests
```

---

# 13. Counter for Business Events

Counters are not limited to infrastructure.

Example:

```
orders_created_total

payments_completed_total

refunds_processed_total

notifications_sent_total
```

These provide business visibility.

---

# 14. Counter for Message Processing

For RabbitMQ or other queues:

```
messages_received_total

messages_processed_total

messages_failed_total
```

This helps measure:

```
Processing Rate
Failure Rate
Consumer Performance
```

---

# 15. Counter for CI/CD

Counters can also represent pipeline events.

Examples:

```
deployments_total

deployment_failures_total

builds_completed_total

builds_failed_total
```

This can help measure engineering performance.

---

# 16. Gauge

A Gauge represents a value that can increase or decrease.

Examples:

```
CPU Usage
Memory Usage
Queue Depth
Active Connections
Running Pods
Temperature
```

Example:

```
active_connections
```

Values can change:

```
20
30
50
35
10
```

---

# 17. Gauge Behavior

A Gauge can move in both directions.

Example:

```
Queue Depth

100
 ↓
150
 ↓
200
 ↓
120
 ↓
 50
```

This makes a Gauge appropriate for current state.

---

# 18. Gauge Use Cases

Use a Gauge when the question is:

```
"What is the current value?"
```

Examples:

```
How much memory is being used?

How many connections are active?

How many messages are currently queued?

How many replicas are running?

How much disk space remains?
```

---

# 19. Common Gauge Metrics

Examples:

```
active_connections

queue_depth

memory_usage_bytes

cpu_utilization

running_jobs

available_disk_space_bytes
```

---

# 20. CPU as a Gauge

CPU utilization can be represented as a current value.

Example:

```
cpu_utilization = 72
```

It may later become:

```
85
```

and then:

```
55
```

Because it can move in both directions, a Gauge is appropriate for
current utilization measurements.

---

# 21. Memory as a Gauge

Example:

```
memory_usage_bytes
```

Values:

```
2 GB
3 GB
4 GB
3.5 GB
```

Memory can increase and decrease.

Therefore, current memory usage is represented as a Gauge.

---

# 22. Queue Depth as a Gauge

Example:

```
queue_depth = 500
```

Messages are consumed:

```
queue_depth = 450
```

New messages arrive:

```
queue_depth = 700
```

Because queue depth can move up and down, it is a Gauge.

---

# 23. Active Connections as a Gauge

Example:

```
active_connections = 100
```

Connections close:

```
active_connections = 80
```

New connections arrive:

```
active_connections = 150
```

A Gauge represents this current state.

---

# 24. Kubernetes Replica Count as a Gauge

Examples:

```
kube_deployment_spec_replicas

kube_deployment_status_replicas_available
```

Replica counts can increase and decrease.

Therefore, they are naturally represented as Gauges.

---

# 25. Gauge Operations

Gauges can be queried directly.

Example:

```
memory_usage_bytes
```

They can also be aggregated:

```
sum(memory_usage_bytes)
```

or:

```
max(memory_usage_bytes)
```

---

# 26. Counter vs Gauge

Counter:

```
Represents cumulative events.
```

Gauge:

```
Represents current state.
```

Example:

```
Total Requests
    ↓
Counter

Active Requests
    ↓
Gauge
```

---

# 27. Simple Counter vs Gauge Rule

Ask:

```
"How many have happened?"
```

Use:

```
Counter
```

Ask:

```
"What is the current value?"
```

Use:

```
Gauge
```

Example:

```
Requests received
    → Counter

Active requests
    → Gauge
```

---

# 28. Histogram

A Histogram measures the distribution of observed values.

It is commonly used for:

```
Request Latency
Response Size
Processing Time
Database Query Duration
Job Execution Time
```

Example:

```
http_request_duration_seconds
```

---

# 29. Why Histograms Matter

Suppose we have 1,000 requests.

Most requests:

```
100ms
```

But some requests:

```
2 seconds
```

An average may hide the slow requests.

A histogram helps understand the distribution.

---

# 30. Latency Distribution

Example observations:

```
50ms
70ms
90ms
100ms
120ms
200ms
500ms
1000ms
```

A histogram groups these observations into buckets.

---

# 31. Histogram Buckets

Example buckets:

```
0.1 seconds
0.25 seconds
0.5 seconds
1 second
2.5 seconds
5 seconds
```

Each bucket counts observations up to that boundary.

---

# 32. Cumulative Histogram Buckets

Prometheus histogram buckets are cumulative.

Suppose:

```
<= 0.1s = 100 requests

<= 0.5s = 500 requests

<= 1s = 900 requests

<= 2.5s = 990 requests

<= 5s = 1000 requests
```

The 1-second bucket includes all observations that were 1 second or
less.

---

# 33. Histogram Metrics

A Prometheus histogram creates related metrics.

Example:

```
http_request_duration_seconds_bucket

http_request_duration_seconds_sum

http_request_duration_seconds_count
```

These represent:

```
Bucket Counts
Total Observed Duration
Number of Observations
```

---

# 34. Histogram Bucket Metric

Example:

```
http_request_duration_seconds_bucket{
    le="0.1"
}
```

The `le` label means:

```
less than or equal to
```

So:

```
le="0.1"
```

means:

```
<= 0.1 seconds
```

---

# 35. Histogram Count

Example:

```
http_request_duration_seconds_count
```

If the value is:

```
10000
```

then the histogram has observed:

```
10,000 requests
```

---

# 36. Histogram Sum

Example:

```
http_request_duration_seconds_sum
```

If the value is:

```
2500
```

then the total observed duration is:

```
2,500 seconds
```

assuming the metric unit is seconds.

---

# 37. Histogram Percentiles

Histograms can be used to calculate:

```
P50
P90
P95
P99
```

using:

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

---

# 38. P50

P50 represents the median.

Approximately:

```
50% of observations
are at or below this value.
```

Example:

```
P50 = 100ms
```

This means roughly half of requests are at or below 100ms.

---

# 39. P90

P90 represents the 90th percentile.

Example:

```
P90 = 250ms
```

Approximately 90% of requests are at or below 250ms.

---

# 40. P95

P95 represents the 95th percentile.

Example:

```
P95 = 400ms
```

Approximately 95% of requests are at or below 400ms.

P95 is commonly used for service latency monitoring.

---

# 41. P99

P99 represents the 99th percentile.

Example:

```
P99 = 1.2s
```

Approximately 99% of requests are at or below 1.2 seconds.

P99 is useful for understanding tail latency.

---

# 42. Tail Latency

Tail latency represents the slowest portion of requests.

Example:

```
P50 = 100ms
P95 = 400ms
P99 = 2s
```

Most users may experience relatively low latency, while a small
percentage experience very slow requests.

Monitoring only averages can hide this behavior.

---

# 43. Histogram for API Latency

Example:

```
http_request_duration_seconds
```

Labels:

```
service
method
route
```

Example:

```
http_request_duration_seconds_bucket{
    service="order-service",
    method="POST",
    route="/orders",
    le="0.5"
}
```

This can be used to calculate latency percentiles.

---

# 44. Histogram for Database Queries

Example:

```
db_query_duration_seconds
```

Possible labels:

```
service
operation
```

Avoid putting highly unique SQL text directly into metric labels.

---

# 45. Histogram for Job Processing

Example:

```
job_processing_duration_seconds
```

This can show:

```
Fast Jobs
Normal Jobs
Slow Jobs
```

Percentiles can help identify performance degradation.

---

# 46. Histogram for Response Size

Example:

```
http_response_size_bytes
```

This can measure:

```
Small Responses
Medium Responses
Large Responses
```

Useful for understanding network and application behavior.

---

# 47. Histogram Aggregation

Histograms can be aggregated across instances.

Suppose:

```
Pod A
Pod B
Pod C
```

all expose the same histogram.

Prometheus can aggregate the histogram buckets and calculate a
service-level percentile.

This is one of the major advantages of histograms over client-side
quantiles.

---

# 48. Histogram Aggregation Example

Conceptually:

```
Pod A
   |
   +--- Histogram

Pod B
   |
   +--- Histogram

Pod C
   |
   +--- Histogram

    ↓

Prometheus

    ↓

Aggregated Histogram

    ↓

P95 / P99
```

---

# 49. Histogram and Microservices

For a microservice running many replicas:

```
order-service
   |
   +--- Pod 1
   +--- Pod 2
   +--- Pod 3
   +--- Pod 4
```

Each pod generates latency observations.

Histograms allow these observations to be aggregated for the service.

---

# 50. Summary

A Summary also measures distributions.

It generally exposes:

```
Quantiles
Sum
Count
```

Example:

```
http_request_duration_seconds
```

A Summary can calculate quantiles on the application side.

---

# 51. Summary Quantiles

A Summary may expose:

```
quantile="0.5"

quantile="0.9"

quantile="0.95"

quantile="0.99"
```

These represent:

```
P50
P90
P95
P99
```

---

# 52. Histogram vs Summary

Histogram:

```
Uses buckets
Quantiles calculated using PromQL
Aggregation-friendly
Flexible bucket configuration
```

Summary:

```
Quantiles calculated at client
Less aggregation-friendly
Quantile objectives configured at instrumentation level
```

---

# 53. Why Histograms Are Often Preferred

In distributed environments:

```
Service
   |
   +--- Pod 1
   +--- Pod 2
   +--- Pod 3
   +--- Pod 4
```

If each pod produces its own Summary quantiles, combining them correctly
is difficult.

Histograms can be aggregated through their bucket counts.

Therefore, histograms are often preferred for service-level latency
monitoring.

---

# 54. Histogram vs Summary Example

Suppose:

```
Pod A P95 = 200ms

Pod B P95 = 300ms

Pod C P95 = 500ms
```

You cannot simply calculate:

```
(200 + 300 + 500) / 3
```

to obtain the service P95.

This is why client-side quantiles from Summaries are difficult to
aggregate.

---

# 55. Histogram Aggregation Example

With Histograms:

```
Pod A
   |
   ↓
Buckets

Pod B
   |
   ↓
Buckets

Pod C
   |
   ↓
Buckets
```

Prometheus can combine bucket counts and calculate a service-level
quantile.

---

# 56. Native Histograms

Modern Prometheus versions support native histograms.

Native histograms can represent distributions without relying on the
traditional fixed bucket approach.

Potential advantages include:

```
More Flexible Resolution
Reduced Need for Manually Defined Buckets
Better Representation of Distributions
```

Native histogram support should be evaluated against:

```
Prometheus Version
Client Library Support
Storage Requirements
Query Compatibility
Operational Requirements
```

---

# 57. Traditional Histogram vs Native Histogram

Traditional histogram:

```
Explicit Buckets
    |
    ↓
_bucket Series
```

Native histogram:

```
Histogram Data Structure
    |
    ↓
More Flexible Distribution Representation
```

Traditional histograms remain widely used and are highly compatible
with existing Prometheus tooling.

---

# 58. Choosing Histogram Buckets

Bucket selection is important.

For HTTP latency, possible buckets might be:

```
0.005
0.01
0.025
0.05
0.1
0.25
0.5
1
2.5
5
10
```

These represent:

```
5ms
10ms
25ms
50ms
100ms
250ms
500ms
1s
2.5s
5s
10s
```

The correct buckets depend on the application's expected latency.

---

# 59. Poor Histogram Buckets

Suppose the application normally responds in:

```
50ms - 500ms
```

but buckets are:

```
10s
30s
60s
```

The buckets provide poor resolution around the important range.

Bucket boundaries should reflect actual system behavior and SLOs.

---

# 60. Histogram Bucket Design

Consider:

```
Historical Latency
Expected Latency
SLO Target
Tail Latency
```

Example:

```
SLO = 500ms
```

Useful buckets should provide enough resolution around:

```
100ms
250ms
500ms
1s
```

---

# 61. Counter for Request Count

Use:

```
http_requests_total
```

Question:

```
How many requests occurred?
```

---

# 62. Gauge for Active Requests

Use:

```
active_requests
```

Question:

```
How many requests are currently active?
```

---

# 63. Histogram for Request Duration

Use:

```
http_request_duration_seconds
```

Question:

```
How long do requests take?
```

---

# 64. Summary for Request Duration

A Summary can also represent request duration.

Question:

```
What are the client-side quantiles?
```

However, for distributed service-level aggregation, Histogram is often
more appropriate.

---

# 65. Metric Type Decision Tree

Use this simple decision process:

```
Is it a cumulative event?
      |
     Yes
      ↓
   Counter

      No
      |
      ↓
Is it a current state?
      |
     Yes
      ↓
    Gauge

      No
      |
      ↓
Is it a distribution?
      |
     Yes
      ↓
Histogram / Summary
```

---

# 66. Metric Type Decision Examples

```
Total Requests
    ↓
Counter

Current CPU
    ↓
Gauge

Queue Depth
    ↓
Gauge

Request Latency
    ↓
Histogram

Database Query Duration
    ↓
Histogram

Client-Side Quantile
    ↓
Summary
```

---

# 67. Counter vs Gauge Real-World Example

Consider an order service.

Metrics:

```
orders_created_total

active_orders
```

The first is cumulative.

Use:

```
Counter
```

The second represents current state.

Use:

```
Gauge
```

---

# 68. Counter vs Gauge for Kubernetes

Consider pods.

Metric:

```
pods_started_total
```

This represents cumulative events.

Use:

```
Counter
```

Metric:

```
running_pods
```

This represents current state.

Use:

```
Gauge
```

---

# 69. Counter vs Gauge for RabbitMQ

Metric:

```
messages_published_total
```

Use:

```
Counter
```

Metric:

```
queue_depth
```

Use:

```
Gauge
```

Metric:

```
messages_processed_total
```

Use:

```
Counter
```

---

# 70. Counter vs Gauge for Databases

Metric:

```
queries_total
```

Use:

```
Counter
```

Metric:

```
active_connections
```

Use:

```
Gauge
```

Metric:

```
database_connection_errors_total
```

Use:

```
Counter
```

---

# 71. Counter vs Gauge for Infrastructure

Metric:

```
node_cpu_seconds_total
```

This represents cumulative CPU time.

It behaves as a Counter.

Metric:

```
current_memory_usage_bytes
```

This represents current memory usage.

It behaves as a Gauge.

---

# 72. Counter vs Gauge for Network

Metric:

```
network_receive_bytes_total
```

Use:

```
Counter
```

Metric:

```
current_bandwidth_utilization
```

Use:

```
Gauge
```

The distinction depends on whether the measurement represents
cumulative activity or current state.

---

# 73. Metric Type and PromQL

Metric type influences how you query the metric.

Counter:

```
rate()
increase()
```

Gauge:

```
Direct Query
avg()
max()
min()
```

Histogram:

```
histogram_quantile()
```

Summary:

```
Query exposed quantiles directly
```

---

# 74. Counter Query

Example:

```
rate(
  http_requests_total[5m]
)
```

This calculates request rate.

---

# 75. Counter Increase Query

Example:

```
increase(
  orders_created_total[1h]
)
```

This estimates the number of orders created during the last hour.

---

# 76. Gauge Query

Example:

```
memory_usage_bytes
```

This returns the current value.

---

# 77. Gauge Average

Example:

```
avg(
  memory_usage_bytes
)
```

This calculates the average across matching time series.

---

# 78. Gauge Maximum

Example:

```
max(
  memory_usage_bytes
)
```

This identifies the highest current value among matching series.

---

# 79. Gauge Minimum

Example:

```
min(
  available_disk_space_bytes
)
```

This identifies the lowest available disk space.

---

# 80. Histogram Quantile Query

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

This estimates service-level P95 latency.

---

# 81. Histogram Aggregation by Service

Suppose we have:

```
service
le
```

labels.

Query:

```
histogram_quantile(
  0.95,
  sum(
    rate(
      http_request_duration_seconds_bucket[5m]
    )
  ) by (service, le)
)
```

This calculates P95 per service.

---

# 82. Histogram Aggregation by Route

Example:

```
histogram_quantile(
  0.95,
  sum(
    rate(
      http_request_duration_seconds_bucket[5m]
    )
  ) by (route, le)
)
```

This can show P95 latency for different routes.

Be careful with route labels to avoid dynamic route values.

---

# 83. Histogram Aggregation by Status

You can also analyze distributions by status if the metric includes
that dimension.

Example:

```
status="200"

status="500"
```

This can help compare successful and failed requests.

---

# 84. Summary Query

A Summary may expose:

```
http_request_duration_seconds{
    quantile="0.95"
}
```

The value represents the configured client-side quantile.

The exact quantiles depend on the instrumentation configuration.

---

# 85. Why Not Use Gauge for Request Rate?

A common mistake is to represent request rate directly as a Gauge.

For example:

```
request_rate = 5000
```

This can become problematic because the rate is derived from cumulative
events.

A better design is:

```
http_requests_total
```

as a Counter.

Then calculate:

```
rate(http_requests_total[5m])
```

This preserves the underlying event count and allows flexible queries.

---

# 86. Why Not Use Gauge for Total Requests?

Suppose:

```
total_requests = 10000
```

and later:

```
total_requests = 12000
```

This is cumulative event data.

A Gauge is inappropriate because the metric represents a monotonically
increasing total.

Use:

```
Counter
```

---

# 87. Why Not Use Counter for CPU Usage?

CPU utilization can increase and decrease.

Example:

```
30%
70%
90%
40%
```

This is current state.

Use:

```
Gauge
```

Some raw CPU time metrics are counters, such as cumulative CPU seconds.

---

# 88. CPU Counter Example

Example:

```
process_cpu_seconds_total
```

This metric represents cumulative CPU time consumed by a process.

Because CPU time accumulates:

```
100s
120s
150s
200s
```

it behaves like a Counter.

To calculate CPU usage rate, PromQL can use:

```
rate(process_cpu_seconds_total[5m])
```

---

# 89. Current CPU vs CPU Time

Current CPU utilization:

```
Gauge
```

Cumulative CPU time:

```
Counter
```

This is an important distinction.

---

# 90. Network Bytes

Example:

```
node_network_receive_bytes_total
```

This is cumulative received bytes.

Use:

```
Counter
```

To calculate receive rate:

```
rate(
  node_network_receive_bytes_total[5m]
)
```

---

# 91. Disk Space

Example:

```
available_disk_space_bytes
```

This represents current available space.

Use:

```
Gauge
```

---

# 92. Disk Bytes Written

Example:

```
disk_written_bytes_total
```

This represents cumulative bytes written.

Use:

```
Counter
```

Then calculate write rate using:

```
rate(
  disk_written_bytes_total[5m]
)
```

---

# 93. Queue Depth vs Messages Processed

Queue depth:

```
queue_depth
```

Use:

```
Gauge
```

Messages processed:

```
messages_processed_total
```

Use:

```
Counter
```

This combination gives both:

```
Current Backlog
Processing Throughput
```

---

# 94. Active Users vs Total Users

Current active users:

```
active_users
```

Use:

```
Gauge
```

Total registered users:

```
users_registered_total
```

If it represents cumulative registrations over time:

```
Counter
```

---

# 95. Current Connections vs Connection Attempts

Current connections:

```
active_connections
```

Use:

```
Gauge
```

Connection attempts:

```
connection_attempts_total
```

Use:

```
Counter
```

Connection failures:

```
connection_failures_total
```

Use:

```
Counter
```

---

# 96. HTTP Metrics Model

A typical HTTP application exposes:

```
http_requests_total
    → Counter

http_request_duration_seconds
    → Histogram

active_http_requests
    → Gauge
```

This combination provides:

```
Traffic
Latency
Concurrency
```

---

# 97. Database Metrics Model

A database-aware application can expose:

```
db_queries_total
    → Counter

db_query_duration_seconds
    → Histogram

db_active_connections
    → Gauge
```

This provides:

```
Query Volume
Query Latency
Current Connections
```

---

# 98. RabbitMQ Metrics Model

Example:

```
messages_published_total
    → Counter

messages_consumed_total
    → Counter

queue_depth
    → Gauge

message_processing_duration_seconds
    → Histogram
```

This provides:

```
Traffic
Processing
Backlog
Latency
```

---

# 99. Kubernetes Metrics Model

Example:

```
container_cpu_usage_seconds_total
    → Counter

container_memory_usage_bytes
    → Gauge

container_restarts_total
    → Counter

running_pods
    → Gauge
```

Different Kubernetes measurements require different metric types.

---

# 100. JVM Metrics Model

For Java applications:

```
jvm_gc_pause_seconds
    → Histogram

jvm_memory_used_bytes
    → Gauge

jvm_threads_live
    → Gauge

jvm_gc_collections_total
    → Counter
```

This provides:

```
Memory
Threads
Garbage Collection
GC Duration
```

---

# 101. Node.js Metrics Model

Example:

```
nodejs_eventloop_lag_seconds
    → Gauge / Histogram depending on instrumentation

process_cpu_seconds_total
    → Counter

process_resident_memory_bytes
    → Gauge

http_request_duration_seconds
    → Histogram
```

---

# 102. Python Metrics Model

Example:

```
process_cpu_seconds_total
    → Counter

process_resident_memory_bytes
    → Gauge

http_requests_total
    → Counter

http_request_duration_seconds
    → Histogram
```

---

# 103. Metric Types and Golden Signals

Golden Signals:

```
Traffic
Latency
Errors
Saturation
```

Possible metric types:

```
Traffic
    → Counter

Latency
    → Histogram

Errors
    → Counter

Saturation
    → Gauge
```

This is not an absolute rule, but it is a useful starting point.

---

# 104. Metric Types and RED

RED:

```
Rate
Errors
Duration
```

Possible implementation:

```
Rate
    → Counter + rate()

Errors
    → Counter + rate()

Duration
    → Histogram
```

This provides a strong foundation for service monitoring.

---

# 105. Metric Types and USE

USE:

```
Utilization
Saturation
Errors
```

Possible implementation:

```
Utilization
    → Gauge

Saturation
    → Gauge

Errors
    → Counter
```

Again, the exact type depends on how the underlying measurement is
defined.

---

# 106. Metric Type Mistakes

Common mistakes:

```
Using Gauge for cumulative events

Using Counter for current state

Using Gauge for latency distribution

Using Summary when cross-instance aggregation is required

Choosing poor histogram buckets

Adding high-cardinality labels
```

---

# 107. Mistake: Gauge for Total Requests

Bad:

```
total_requests
```

as a Gauge.

Why?

Because total requests are cumulative.

Better:

```
http_requests_total
```

as a Counter.

Then:

```
rate(http_requests_total[5m])
```

---

# 108. Mistake: Counter for Active Connections

Bad:

```
active_connections_total
```

as a Counter.

Why?

Active connections can increase and decrease.

Better:

```
active_connections
```

as a Gauge.

---

# 109. Mistake: Gauge for Latency Distribution

Bad:

```
request_latency
```

as a Gauge.

Why?

A single current value does not represent the distribution of request
latency.

Better:

```
http_request_duration_seconds
```

as a Histogram.

---

# 110. Mistake: Poor Histogram Buckets

Bad buckets:

```
10s
30s
60s
```

for an application where normal latency is:

```
50ms - 500ms
```

The histogram will not provide useful resolution.

Design buckets around actual system behavior.

---

# 111. Mistake: Too Many Histogram Buckets

Creating excessive bucket boundaries increases the number of time
series.

This can increase:

```
Memory
Storage
Query Cost
```

Use enough buckets to answer operational questions without unnecessary
complexity.

---

# 112. Metric Type and Cardinality

Metric type does not eliminate cardinality problems.

A Histogram with labels:

```
user_id
request_id
```

can still create huge numbers of time series.

Always evaluate:

```
Metric Type
Labels
Number of Series
Collection Frequency
```

together.

---

# 113. Metric Type and Labels

Example:

```
http_requests_total{
    service="order-service",
    method="GET",
    status="200"
}
```

Counter:

```
Appropriate
```

Labels:

```
Bounded and useful
```

Another example:

```
http_request_duration_seconds{
    service="order-service",
    route="/orders/{id}"
}
```

Histogram:

```
Appropriate
```

Route:

```
Normalized
```

---

# 114. Normalized Routes

Bad:

```
/orders/12345

/orders/12346

/orders/12347
```

These can create many unique label values.

Better:

```
/orders/{id}
```

This provides a stable route dimension.

---

# 115. Metric Types and SLOs

Metric type selection affects SLO calculations.

Availability:

```
Successful Requests
Total Requests
```

Counters can provide these values.

Latency SLO:

```
Requests below latency threshold
```

Histograms are useful for this type of analysis.

---

# 116. Latency SLO Example

Suppose:

```
SLO:
95% of requests should complete below 500ms.
```

A histogram can track:

```
<= 0.5s
```

Then PromQL can calculate the proportion of requests satisfying the
latency target.

---

# 117. Error Budget and Metric Types

Error budget calculations often rely on counters.

Example:

```
Total Requests
    → Counter

Failed Requests
    → Counter
```

Then calculate:

```
Error Rate
```

and compare it with the SLO target.

---

# 118. Metric Type and Alerting

Counter-based alert:

```
rate(http_errors_total[5m]) > threshold
```

Gauge-based alert:

```
memory_usage_bytes > threshold
```

Histogram-based alert:

```
P95 latency > threshold
```

Each metric type supports different operational questions.

---

# 119. Counter Alert Example

Alert condition:

```
Error Rate > 5%
```

Using:

```
rate(
  http_errors_total[5m]
)
```

combined with total request rate.

---

# 120. Gauge Alert Example

Alert condition:

```
Memory Usage > 90%
```

Example:

```
memory_usage_bytes
/
memory_limit_bytes
```

This produces current utilization.

---

# 121. Histogram Alert Example

Alert condition:

```
P95 latency > 500ms
```

Example:

```
histogram_quantile(
  0.95,
  rate(
    http_request_duration_seconds_bucket[5m]
  )
) > 0.5
```

---

# 122. Metric Type and Grafana

Grafana visualization should match the metric type.

Counter-derived:

```
Request Rate
Error Rate
```

Gauge:

```
CPU
Memory
Queue Depth
```

Histogram-derived:

```
P95
P99
Latency Distribution
```

This creates more meaningful dashboards.

---

# 123. Counter Dashboard

Example:

```
HTTP Request Rate

10:00 → 1,000 req/sec
10:05 → 1,500 req/sec
10:10 → 2,000 req/sec
```

The underlying metric is:

```
http_requests_total
```

The dashboard displays:

```
rate()
```

---

# 124. Gauge Dashboard

Example:

```
CPU Usage

10:00 → 40%
10:05 → 65%
10:10 → 80%
```

The underlying metric is a Gauge.

---

# 125. Histogram Dashboard

Example:

```
Request Latency

P50 = 100ms
P95 = 350ms
P99 = 800ms
```

The underlying metric is a Histogram.

---

# 126. Summary Dashboard

Example:

```
Client-side quantiles:

P50 = 100ms
P95 = 400ms
P99 = 900ms
```

These values are exposed directly by the Summary.

---

# 127. Choosing Between Histogram and Summary

Ask:

```
Do I need to aggregate latency across instances?
```

If:

```
Yes
```

prefer:

```
Histogram
```

If:

```
No
```

and client-side quantiles are appropriate:

```
Summary
```

In distributed microservices, Histograms are often the preferred
default for request latency.

---

# 128. Metric Type Selection for Microservices

For a production microservice:

```
Request Count
    → Counter

Request Errors
    → Counter

Request Duration
    → Histogram

Active Requests
    → Gauge

Database Connections
    → Gauge

Database Queries
    → Counter
```

This combination provides strong operational visibility.

---

# 129. Complete Microservice Metrics

Example:

```
order-service

http_requests_total
    → Counter

http_errors_total
    → Counter

http_request_duration_seconds
    → Histogram

active_requests
    → Gauge

db_queries_total
    → Counter

db_query_duration_seconds
    → Histogram

db_active_connections
    → Gauge
```

---

# 130. Complete Kubernetes Metrics

Example:

```
container_cpu_usage_seconds_total
    → Counter

container_memory_usage_bytes
    → Gauge

container_network_receive_bytes_total
    → Counter

container_network_transmit_bytes_total
    → Counter

container_restarts_total
    → Counter
```

---

# 131. Complete Infrastructure Metrics

Example:

```
process_cpu_seconds_total
    → Counter

process_resident_memory_bytes
    → Gauge

node_network_receive_bytes_total
    → Counter

node_network_transmit_bytes_total
    → Counter

node_filesystem_avail_bytes
    → Gauge
```

---

# 132. Complete Database Metrics

Example:

```
db_queries_total
    → Counter

db_query_duration_seconds
    → Histogram

db_active_connections
    → Gauge

db_connection_errors_total
    → Counter

db_storage_bytes
    → Gauge
```

---

# 133. Complete Queue Metrics

Example:

```
messages_published_total
    → Counter

messages_consumed_total
    → Counter

queue_depth
    → Gauge

message_processing_duration_seconds
    → Histogram

consumer_errors_total
    → Counter
```

---

# 134. Metric Type Testing

Before production, validate each metric.

For Counter:

```
Generate events
    |
    ↓
Counter increases
```

For Gauge:

```
Increase state
    |
    ↓
Gauge increases

Decrease state
    |
    ↓
Gauge decreases
```

For Histogram:

```
Generate observations
    |
    ↓
Bucket counts change
```

For Summary:

```
Generate observations
    |
    ↓
Quantiles change
```

---

# 135. Counter Testing

Example:

```
Create 100 orders.
```

Expected:

```
orders_created_total
```

increases by approximately:

```
100
```

---

# 136. Gauge Testing

Example:

```
Start 10 workers.
```

Expected:

```
running_workers = 10
```

Stop 4 workers.

Expected:

```
running_workers = 6
```

---

# 137. Histogram Testing

Generate:

```
100 requests < 100ms

50 requests around 500ms

10 requests around 1s
```

Verify:

```
Bucket counts
Sum
Count
P95
P99
```

---

# 138. Summary Testing

Generate known latency observations.

Verify:

```
P50
P90
P95
P99
```

match the expected approximate distribution.

---

# 139. Metric Type During Application Restart

Counter:

```
May reset
```

Gauge:

```
Returns current state after restart
```

Histogram:

```
Application-local observations may reset
```

Summary:

```
Application-local observations may reset
```

Prometheus should interpret time-series continuity carefully across
process restarts.

---

# 140. Metric Type and Recording Rules

Recording rules can transform metrics into operational measurements.

Example:

```
http_requests_total
     |
     ↓
rate()
     |
     ↓
Recording Rule
     |
     ↓
service:http_requests_per_second
```

The underlying metric remains a Counter.

The derived metric represents a rate.

---

# 141. Derived Metrics

A derived metric can be calculated from another metric.

Examples:

```
Counter
   ↓
rate()
   ↓
Request Rate

Counter
   ↓
increase()
   ↓
Total Events in Window

Histogram
   ↓
histogram_quantile()
   ↓
P95 Latency
```

---

# 142. Do Not Change the Meaning of the Original Metric

Keep:

```
http_requests_total
```

as the original cumulative Counter.

Do not overwrite its meaning with a rate.

Instead derive:

```
rate(http_requests_total[5m])
```

This preserves flexibility.

---

# 143. Metric Type and Data Semantics

The most important rule is:

```
The metric type should reflect the meaning of the measurement.
```

Not:

```
"Which type is easiest?"
```

But:

```
"What does this value represent?"
```

Ask:

```
Is it cumulative?

Is it current state?

Is it a distribution?
```

Then select the appropriate type.

---

# 144. Metric Type Decision Table

| Measurement        | Metric Type | Reason                |
| ------------------ | ----------- | --------------------- |
| Total Requests     | Counter     | Cumulative            |
| Total Errors       | Counter     | Cumulative            |
| Total Payments     | Counter     | Cumulative            |
| CPU Usage          | Gauge       | Current State         |
| Memory Usage       | Gauge       | Current State         |
| Queue Depth        | Gauge       | Current State         |
| Active Connections | Gauge       | Current State         |
| Request Latency    | Histogram   | Distribution          |
| DB Query Duration  | Histogram   | Distribution          |
| Job Duration       | Histogram   | Distribution          |
| Client Quantiles   | Summary     | Client-Side Quantiles |

---

# 145. Metric Type Cheat Sheet

```
COUNTER

Use for:
Total events
Requests
Errors
Transactions
Messages
Jobs

Query with:
rate()
increase()
```

---

```
GAUGE

Use for:
CPU
Memory
Queue Depth
Connections
Replicas
Current State

Query with:
Direct Query
avg()
min()
max()
```

---

```
HISTOGRAM

Use for:
Latency
Duration
Size
Distributions

Query with:
histogram_quantile()
```

---

```
SUMMARY

Use for:
Client-side quantiles
Local distribution measurements

Consider:
Aggregation limitations
```

---

# 146. Real-World DevOps Example

Consider an EKS-based microservices platform:

```
ALB
  |
  ↓
EKS
  |
  +--- User Service
  +--- Product Service
  +--- Cart Service
  +--- Order Service
  +--- Payment Service
  +--- Inventory Service
  +--- Notification Service
```

Metrics:

```
Request Count
    → Counter

Error Count
    → Counter

Request Duration
    → Histogram

Active Connections
    → Gauge

CPU
    → Gauge

Memory
    → Gauge

Queue Depth
    → Gauge

Messages Processed
    → Counter
```

---

# 147. Real-World Request Flow

A request enters:

```
ALB
  |
  ↓
Order Service
  |
  ↓
Payment Service
  |
  ↓
Inventory Service
  |
  ↓
Database
```

Metrics:

```
http_requests_total
    → Counter

http_request_duration_seconds
    → Histogram

active_requests
    → Gauge

db_queries_total
    → Counter

db_query_duration_seconds
    → Histogram
```

---

# 148. Production Metric Model

A strong application metric model combines:

```
Counters
   +
Gauges
   +
Histograms
```

Example:

```
Counter
    ↓
How much traffic?

Gauge
    ↓
What is the current state?

Histogram
    ↓
How is the distribution behaving?
```

Together they provide a much more complete picture.

---

# 149. Metric Types and Observability

Metrics are one part of the larger observability model:

```
Metrics
    |
    ↓
Detect

Logs
    |
    ↓
Explain

Traces
    |
    ↓
Locate
```

Metric types determine how the detection layer is built.

---

# 150. Production Best Practices

```
1. Use Counter for cumulative events.

2. Use Gauge for current state.

3. Use Histogram for distributions.

4. Use Summary only when client-side quantiles are appropriate.

5. Prefer Histograms for aggregatable service-level latency.

6. Use meaningful metric names.

7. Include units in metric names where appropriate.

8. Avoid high-cardinality labels.

9. Normalize dynamic routes.

10. Choose histogram buckets based on real latency behavior.

11. Align latency buckets with SLOs.

12. Preserve the original Counter and derive rates with PromQL.

13. Test metrics before production.

14. Monitor metric cardinality.

15. Document the meaning of custom metrics.
```

---

# 151. Common Mistakes

## Mistake 1

Using a Gauge for total requests.

Correct:

```
Counter
```

---

## Mistake 2

Using a Counter for active connections.

Correct:

```
Gauge
```

---

## Mistake 3

Using a Gauge for request latency distribution.

Correct:

```
Histogram
```

---

## Mistake 4

Using Summary quantiles when aggregation across many instances is
required.

Consider:

```
Histogram
```

---

## Mistake 5

Using request IDs as labels.

Avoid:

```
request_id
```

Use:

```
Logs
Traces
```

for request-level identification.

---

## Mistake 6

Using dynamic URLs as labels.

Bad:

```
/orders/12345
```

Better:

```
/orders/{id}
```

---

## Mistake 7

Choosing histogram buckets without looking at actual latency.

Always consider:

```
Historical Latency
Expected Latency
SLO
Tail Latency
```

---

# 152. Interview Questions

## What are the four main Prometheus metric types?

### Answer

The four main Prometheus metric types are:

```
Counter
Gauge
Histogram
Summary
```

Counter represents cumulative events.

Gauge represents current state.

Histogram represents distributions using buckets.

Summary represents distributions using client-side quantiles.

---

# 153. What is a Counter?

### Answer

A Counter is a cumulative metric that normally only increases during
the lifetime of a process.

Examples:

```
Requests
Errors
Transactions
Messages
```

For example:

```
http_requests_total
```

I would normally use:

```
rate()
```

or:

```
increase()
```

to analyze it.

---

# 154. What is a Gauge?

### Answer

A Gauge represents a value that can increase or decrease.

Examples:

```
CPU Usage
Memory Usage
Queue Depth
Active Connections
Running Replicas
```

It represents current state.

---

# 155. What is a Histogram?

### Answer

A Histogram measures a distribution of observations using buckets.

It is commonly used for:

```
Request Latency
Database Query Duration
Job Duration
Response Size
```

It can be used with:

```
histogram_quantile()
```

to estimate percentiles.

---

# 156. What is a Summary?

### Answer

A Summary measures observations and can expose client-side quantiles.

It typically provides:

```
Quantiles
Sum
Count
```

However, client-side quantiles are difficult to aggregate correctly
across multiple instances, so Histograms are often preferred for
distributed service-level latency analysis.

---

# 157. Counter vs Gauge?

### Answer

I use a Counter for cumulative events and a Gauge for current state.

For example:

```
http_requests_total
    → Counter

active_connections
    → Gauge
```

The key question is:

```
Is this a cumulative event or current state?
```

---

# 158. Histogram vs Summary?

### Answer

Histogram uses buckets and allows quantile calculation from aggregated
data.

Summary calculates quantiles at the client.

For a distributed microservice environment, I generally prefer
Histograms for latency because bucket data can be aggregated across
instances.

---

# 159. Why is Histogram commonly used for latency?

### Answer

Latency is a distribution, not just a single value.

A histogram allows us to understand:

```
P50
P95
P99
```

and aggregate observations across multiple application instances.

This is especially useful for Kubernetes microservices.

---

# 160. Why can't you simply average P95 values from multiple pods?

### Answer

Percentiles are not generally additive or linearly averageable.

For example:

```
Pod A P95 = 200ms

Pod B P95 = 500ms
```

You cannot simply calculate:

```
(200 + 500) / 2
```

and call the result the service P95.

Histograms allow the underlying distributions to be aggregated before
calculating the percentile.

---

# 161. Why should request IDs not be metric labels?

### Answer

Request IDs are highly unique.

If every request generates a unique label value, Prometheus creates a
large number of time series.

This can cause:

```
High Memory Usage
High Storage
High Query Cost
Performance Problems
```

Request IDs belong more naturally in logs and traces.

---

# 162. How would you monitor HTTP traffic?

### Answer

I would use:

```
http_requests_total
    → Counter
```

Then calculate:

```
rate(http_requests_total[5m])
```

for request rate.

For latency:

```
http_request_duration_seconds
    → Histogram
```

For active requests:

```
active_requests
    → Gauge
```

This provides traffic, latency, and concurrency visibility.

---

# 163. How would you monitor a database?

### Answer

I would use:

```
db_queries_total
    → Counter

db_query_duration_seconds
    → Histogram

db_active_connections
    → Gauge

db_connection_errors_total
    → Counter
```

This provides query volume, latency, connections, and failures.

---

# 164. How would you monitor RabbitMQ?

### Answer

I would monitor:

```
messages_published_total
    → Counter

messages_consumed_total
    → Counter

queue_depth
    → Gauge

message_processing_duration_seconds
    → Histogram
```

This provides:

```
Throughput
Backlog
Processing Latency
```

---

# 165. How would you monitor Kubernetes?

### Answer

I would use the appropriate metric type for each measurement.

Examples:

```
container_cpu_usage_seconds_total
    → Counter

container_memory_usage_bytes
    → Gauge

container_restarts_total
    → Counter

running_pods
    → Gauge
```

This allows Prometheus to calculate rates and current state correctly.

---

# 166. How do metric types affect alerting?

### Answer

Different metric types require different queries.

Counter:

```
rate(errors_total[5m])
```

Gauge:

```
memory_usage_bytes > threshold
```

Histogram:

```
histogram_quantile(
  0.95,
  ...
) > latency_threshold
```

Using the wrong metric type can lead to incorrect alerts.

---

# 167. How do metric types affect SLOs?

### Answer

Counters are useful for availability and error-rate calculations.

Histograms are useful for latency-based SLOs.

For example:

```
Total Requests
    → Counter

Failed Requests
    → Counter

Request Duration
    → Histogram
```

These can be used to calculate availability and latency objectives.

---

# 168. How would you choose histogram buckets?

### Answer

I would use real application behavior and SLO requirements.

I would review:

```
Historical Latency
Expected Latency
P95
P99
SLO Threshold
```

If the SLO is:

```
500ms
```

I would ensure there are useful bucket boundaries around that value.

I would avoid both excessively broad and unnecessarily large numbers
of buckets.

---

# 169. Can a Counter decrease?

### Answer

A Counter should normally only increase during the lifetime of the
process.

It can appear to decrease when the application restarts because the
process-local counter resets.

Prometheus functions such as rate() are designed to handle counter
resets.

---

# 170. Can a Gauge decrease?

### Answer

Yes.

A Gauge can increase and decrease.

Example:

```
Queue Depth:

100
200
150
50
```

This is exactly the type of measurement for which a Gauge is intended.

---

# 171. Can a Histogram be aggregated?

### Answer

Yes.

Histogram bucket counts can be aggregated across instances when the
histograms use compatible bucket boundaries.

This makes Histograms useful for distributed microservices.

---

# 172. What is the most important rule when choosing metric types?

### Answer

The metric type should reflect the semantic meaning of the measurement.

Ask:

```
Is it cumulative?
    → Counter

Is it current state?
    → Gauge

Is it a distribution?
    → Histogram / Summary
```

This is more important than simply memorizing metric names.

---

# 173. Final Metric Type Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Applications                         │
│                                                         │
│ Java | Node.js | Python | Kubernetes | Databases       │
└────────────────────────────┬────────────────────────────┘
                             |
         +-------------------+-------------------+
         |                   |                   |
         ↓                   ↓                   ↓
      Counter              Gauge             Histogram
         |                   |                   |
         ↓                   ↓                   ↓
  Cumulative Events      Current State        Distribution
         |                   |                   |
         +-------------------+-------------------+
                             |
                             ↓
                        Prometheus
                             |
                             ↓
                           PromQL
                             |
             +---------------+---------------+
             |                               |
             ↓                               ↓
          Grafana                         Alerts
             |                               |
             ↓                               ↓
        Dashboards                    Incident Response
```

---

# 174. Final Metric Type Mental Model

Remember:

```
COUNTER

"How many have happened?"

        ↓

GAUGE

"What is the current value?"

        ↓

HISTOGRAM

"How are observations distributed?"

        ↓

SUMMARY

"What client-side quantiles are being observed?"
```

For a real-world microservices platform, a typical design is:

```
Request Count
    → Counter

Error Count
    → Counter

Request Latency
    → Histogram

Active Requests
    → Gauge

CPU
    → Gauge

Memory
    → Gauge

Database Queries
    → Counter

Database Query Duration
    → Histogram

Database Connections
    → Gauge

Queue Messages
    → Counter

Queue Depth
    → Gauge
```

This combination gives Prometheus the correct semantics for
calculating:

```
Rates
Increases
Current State
Percentiles
Error Rates
Saturation
SLOs
Alerts
```

The key principle is:

```
Choose the metric type based on what the measurement means,
not simply on how the number looks.
```

A correctly designed metric model becomes the foundation for reliable
dashboards, alerts, SLOs, troubleshooting, capacity planning, and
production observability.
