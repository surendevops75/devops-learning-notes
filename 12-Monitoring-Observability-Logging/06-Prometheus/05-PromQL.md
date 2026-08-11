# PromQL

PromQL stands for **Prometheus Query Language**.

PromQL is the query language used by Prometheus to:

```
Select metrics
Filter metrics
Aggregate metrics
Calculate rates
Calculate percentages
Compare values
Analyze time series
Build Grafana dashboards
Create recording rules
Create alerting rules
```

The basic architecture is:

```
Application / Exporter
        ↓
    Metrics
        ↓
    Prometheus
        ↓
      TSDB
        ↓
      PromQL
        ↓
┌───────┼────────┐
↓       ↓        ↓
```

Grafana  Alerts   Analysis

---

# 1. Why PromQL Is Important

Prometheus stores time-series metrics, but storing metrics alone is not enough.

Engineers need to answer questions such as:

```
Is CPU high?

Which pods are consuming memory?

What is the request rate?

What is the error rate?

Which service has the highest latency?

How many pods are unavailable?

Is the application healthy?
```

PromQL provides the ability to answer these questions.

---

# 2. Basic PromQL Mental Model

Think of PromQL as:

```
Metric
   ↓
Select
   ↓
Filter
   ↓
Transform
   ↓
Aggregate
   ↓
Result
```

Example:

```promql
up
```

Then:

```promql
up{job="node-exporter"}
```

Then:

```promql
sum(up{job="node-exporter"})
```

Each step produces a different view of the same underlying time-series data.

---

# 3. Prometheus Data Model

Prometheus stores data as time series.

A time series is identified by:

```
Metric Name
+
Set of Labels
```

Example:

```text
http_requests_total{
  method="GET",
  service="order",
  status="200"
}
```

The metric name is:

```text
http_requests_total
```

The labels are:

```text
method="GET"
service="order"
status="200"
```

---

# 4. Metric Name

A simple query:

```promql
up
```

selects the metric:

```text
up
```

Another example:

```promql
node_cpu_seconds_total
```

This selects the CPU time metric.

---

# 5. Metric Selector

The simplest PromQL expression is a metric selector.

Example:

```promql
up
```

This returns all matching `up` time series.

Another:

```promql
node_memory_MemAvailable_bytes
```

This returns available memory for matching nodes.

---

# 6. Instant Vector

A metric selector without a range returns an **instant vector**.

Example:

```promql
up
```

Conceptually:

```text
Target A → 1
Target B → 1
Target C → 0
```

Each result has:

```
Labels
Current Value
```

---

# 7. Range Vector

A range selector retrieves samples over a time window.

Example:

```promql
http_requests_total[5m]
```

This means:

```
Give me the samples from the last 5 minutes.
```

Range vectors are commonly used with:

```
rate()

increase()

irate()

delta()

changes()
```

---

# 8. Range Selector

Common examples:

```promql
[1m]
```

```promql
[5m]
```

```promql
[15m]
```

```promql
[1h]
```

```promql
[1d]
```

For example:

```promql
http_requests_total[5m]
```

means the last five minutes of samples.

---

# 9. Time Units

PromQL duration units include:

```
ms = milliseconds
s  = seconds
m  = minutes
h  = hours
d  = days
w  = weeks
y  = years
```

Examples:

```promql
[30s]
```

```promql
[5m]
```

```promql
[1h]
```

---

# 10. Label Matching

PromQL allows filtering using labels.

Example:

```promql
up{job="node-exporter"}
```

This means:

```
Select up
where job equals node-exporter
```

---

# 11. Multiple Labels

Example:

```promql
http_requests_total{
  service="order",
  method="GET"
}
```

This selects:

```
service = order
```

AND:

```
method = GET
```

---

# 12. Exact Label Matching

The operator:

```text
=
```

means exact match.

Example:

```promql
up{environment="production"}
```

Only production targets are selected.

---

# 13. Negative Label Matching

The operator:

```text
!=
```

means not equal.

Example:

```promql
up{environment!="development"}
```

This excludes targets where:

```text
environment="development"
```

---

# 14. Regex Matching

The operator:

```text
=~
```

performs regex matching.

Example:

```promql
up{environment=~"prod|staging"}
```

This matches:

```
prod
```

OR:

```
staging
```

---

# 15. Regex Negative Matching

The operator:

```text
!~
```

excludes values matching a regex.

Example:

```promql
up{environment!~"dev|test"}
```

This excludes:

```
dev

test
```

---

# 16. Label Matching Summary

PromQL supports:

```text
=     Exact match
!=    Not equal
=~    Regex match
!~    Regex not match
```

Example:

```promql
up{job="node-exporter"}
```

```promql
up{job!="node-exporter"}
```

```promql
up{job=~"node.*"}
```

```promql
up{job!~"test.*"}
```

---

# 17. Multiple Label Filtering

Example:

```promql
up{
  environment="production",
  region="ap-south-1",
  job="node-exporter"
}
```

This selects only targets matching all specified conditions.

---

# 18. Metric Names and Labels

Instead of creating metrics such as:

```text
order_requests
payment_requests
inventory_requests
```

prefer a consistent metric with labels:

```text
http_requests_total{
  service="order"
}
```

```text
http_requests_total{
  service="payment"
}
```

```text
http_requests_total{
  service="inventory"
}
```

This makes PromQL more powerful and reusable.

---

# 19. Operators in PromQL

PromQL supports operators for:

```
Arithmetic

Comparison

Logical operations

Vector matching
```

Common arithmetic operators:

```text
+
-
*
/
%
^
```

---

# 20. Addition

Example:

```promql
node_memory_MemTotal_bytes
+
node_memory_MemFree_bytes
```

The two compatible vectors are added.

Arithmetic should be used carefully because vector label matching affects the result.

---

# 21. Subtraction

Example:

```promql
node_memory_MemTotal_bytes
-
node_memory_MemAvailable_bytes
```

This can help calculate memory usage.

---

# 22. Multiplication

Example:

```promql
http_requests_per_second * 60
```

This converts:

```
Requests per second
```

into:

```
Approximate requests per minute
```

---

# 23. Division

Division is heavily used in monitoring.

Example:

```promql
used_memory / total_memory
```

This produces a ratio.

To convert it into a percentage:

```promql
(
  used_memory / total_memory
) * 100
```

---

# 24. Modulo

PromQL supports:

```text
%
```

for modulo operations.

Example:

```promql
metric % 2
```

It is less common in everyday monitoring queries.

---

# 25. Power Operator

PromQL supports:

```text
^
```

for exponentiation.

Example:

```promql
2 ^ 3
```

Result:

```text
8
```

---

# 26. Comparison Operators

PromQL supports:

```text
==
!=
>
<
>=
<=
```

Example:

```promql
up == 0
```

This identifies targets whose `up` value is zero.

---

# 27. CPU Alert Example

Example:

```promql
cpu_usage_percent > 80
```

This returns series where CPU usage is above 80.

---

# 28. Memory Alert Example

```promql
memory_usage_percent > 90
```

This can be used to identify high-memory systems.

---

# 29. Equality

Example:

```promql
up == 0
```

This is useful for identifying unavailable targets.

---

# 30. Boolean Modifier

PromQL comparison operators can be used as filters or, with the `bool` modifier, to return 0/1 results.

Example:

```promql
up == 0
```

versus:

```promql
up == bool 0
```

The `bool` modifier changes the comparison result behavior.

---

# 31. Logical Operators

PromQL supports:

```text
and
or
unless
```

These operate on vectors.

---

# 32. `and`

Example:

```promql
up == 0
and
probe_success == 0
```

This can combine matching conditions.

Vector matching rules are important when using these operators.

---

# 33. `or`

Example:

```promql
up == 0
or
probe_success == 0
```

This can combine alternative conditions.

---

# 34. `unless`

Example:

```promql
up == 0
unless
maintenance_mode == 1
```

This can exclude series matching the second expression.

---

# 35. Counter Metrics

Counters represent values that generally increase over time.

Example:

```text
http_requests_total
```

A counter might look like:

```text
100
150
200
250
```

Counters are usually reset when the application restarts.

---

# 36. Counter Example

A request counter:

```promql
http_requests_total
```

does not directly represent:

```
Requests per second
```

It represents:

```
Total requests observed by the instrumented process.
```

To calculate rate, use:

```promql
rate(http_requests_total[5m])
```

---

# 37. `rate()`

`rate()` calculates the average per-second increase of a counter over a range.

Example:

```promql
rate(http_requests_total[5m])
```

Meaning:

```
Calculate average requests per second over the last 5 minutes.
```

---

# 38. Why Use `rate()`

Suppose:

```text
http_requests_total
```

has:

```text
10,000
```

then later:

```text
10,600
```

The counter increased by:

```text
600
```

PromQL calculates the increase over the selected time range and normalizes it to a per-second rate.

---

# 39. Rate for Multiple Services

Example:

```promql
rate(
  http_requests_total[5m]
)
```

This returns a rate for each matching series.

To aggregate:

```promql
sum(
  rate(http_requests_total[5m])
)
```

This gives the total request rate across the matching series.

---

# 40. `irate()`

`irate()` calculates an instant per-second rate based primarily on the most recent samples in the selected range.

Example:

```promql
irate(http_requests_total[5m])
```

It is more sensitive to short-term changes.

---

# 41. `rate()` vs `irate()`

Use:

```promql
rate()
```

for:

```
Dashboards

Alerting

Stable trends
```

Use:

```promql
irate()
```

when you specifically need a more instantaneous view of counter behavior.

For most production alerting, `rate()` is generally the safer default.

---

# 42. `increase()`

`increase()` calculates the total increase of a counter over a time range.

Example:

```promql
increase(http_requests_total[1h])
```

This answers:

```
Approximately how many requests occurred during the last hour?
```

---

# 43. `rate()` vs `increase()`

`rate()`:

```promql
rate(http_requests_total[5m])
```

returns:

```
Per-second rate
```

`increase()`:

```promql
increase(http_requests_total[1h])
```

returns:

```
Total increase over the range
```

---

# 44. Counter Reset Handling

Prometheus functions such as:

```promql
rate()
```

and:

```promql
increase()
```

are designed to handle counter resets.

This is important because application processes can restart.

---

# 45. Gauge Metrics

A gauge can move up and down.

Examples:

```text
memory_usage_bytes
temperature_celsius
active_connections
```

Example:

```promql
node_memory_MemAvailable_bytes
```

Gauges should generally not be passed through `rate()`.

---

# 46. Gauge Example

Current available memory:

```promql
node_memory_MemAvailable_bytes
```

Current CPU load:

```promql
node_load1
```

Current active connections:

```promql
active_connections
```

---

# 47. `avg_over_time()`

This calculates the average value over a range.

Example:

```promql
avg_over_time(
  node_load1[15m]
)
```

This gives the average load over the last 15 minutes.

---

# 48. `max_over_time()`

Example:

```promql
max_over_time(
  node_load1[15m]
)
```

This returns the maximum value observed during the range.

Useful for:

```
Peak CPU

Peak Memory

Maximum Queue Depth
```

---

# 49. `min_over_time()`

Example:

```promql
min_over_time(
  node_load1[15m]
)
```

This returns the minimum value over the range.

---

# 50. `sum_over_time()`

Example:

```promql
sum_over_time(
  some_gauge[1h]
)
```

This sums samples over the selected range.

Be careful with interpretation because summing gauge samples does not automatically mean "total amount" unless the metric semantics make that meaningful.

---

# 51. `count_over_time()`

Example:

```promql
count_over_time(
  up[1h]
)
```

This counts samples within the range.

---

# 52. `quantile_over_time()`

Example:

```promql
quantile_over_time(
  0.95,
  some_metric[1h]
)
```

This calculates a quantile over the samples of a range vector.

This is different from calculating a request latency percentile from histogram buckets.

---

# 53. Aggregation Operators

PromQL provides aggregation operators such as:

```text
sum
avg
min
max
count
stddev
stdvar
topk
bottomk
quantile
count_values
group
```

These are used to combine multiple series.

---

# 54. `sum()`

Example:

```promql
sum(
  rate(http_requests_total[5m])
)
```

This gives total request rate across matching series.

---

# 55. Sum by Label

Example:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

Result:

```text
service="order"      25
service="payment"    10
service="inventory"   8
```

This is extremely common in microservices monitoring.

---

# 56. Sum Without Label

You can remove selected dimensions using:

```promql
sum without(instance) (
  rate(http_requests_total[5m])
)
```

This aggregates across instances while retaining other labels.

---

# 57. `avg()`

Example:

```promql
avg(
  node_load1
)
```

This calculates the average across matching series.

---

# 58. Average by Service

```promql
avg by (service) (
  request_latency_seconds
)
```

This calculates the average value for each service.

Be careful with latency metrics: if the metric represents histogram buckets, use histogram-specific PromQL instead of averaging bucket counts.

---

# 59. `min()`

Example:

```promql
min(
  node_memory_MemAvailable_bytes
)
```

This identifies the lowest available-memory value among matching series.

---

# 60. `max()`

Example:

```promql
max(
  node_memory_MemAvailable_bytes
)
```

This identifies the highest available-memory value.

---

# 61. `count()`

Example:

```promql
count(
  up
)
```

This counts the number of matching series.

For Kubernetes:

```promql
count(
  kube_pod_info
)
```

can help count matching pod series, depending on metric labels and filters.

---

# 62. Count by Label

Example:

```promql
count by (namespace) (
  kube_pod_info
)
```

This gives a count grouped by namespace.

---

# 63. `topk()`

Example:

```promql
topk(
  5,
  rate(http_requests_total[5m])
)
```

This returns the top five matching series.

Useful for:

```
Highest Traffic

Highest CPU

Highest Memory

Highest Error Rate
```

---

# 64. Top Services by Request Rate

```promql
topk(
  5,
  sum by (service) (
    rate(http_requests_total[5m])
  )
)
```

This identifies the five busiest services.

---

# 65. `bottomk()`

Example:

```promql
bottomk(
  5,
  rate(http_requests_total[5m])
)
```

This returns the lowest five matching series.

---

# 66. `quantile()`

PromQL's aggregation operator:

```promql
quantile(
  0.95,
  metric
)
```

calculates a quantile across series.

This is different from `histogram_quantile()`, which is commonly used with histogram buckets.

---

# 67. Histogram Metrics

Prometheus histograms commonly produce:

```text
request_duration_seconds_bucket
request_duration_seconds_sum
request_duration_seconds_count
```

These are used to calculate latency distributions.

---

# 68. Histogram Buckets

Example:

```text
request_duration_seconds_bucket{
  le="0.1"
}

request_duration_seconds_bucket{
  le="0.5"
}

request_duration_seconds_bucket{
  le="1"
}
```

The `le` label means:

```
Less than or equal to
```

---

# 69. `histogram_quantile()`

To calculate the 95th percentile:

```promql
histogram_quantile(
  0.95,
  rate(request_duration_seconds_bucket[5m])
)
```

For aggregation across instances:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(request_duration_seconds_bucket[5m])
  )
)
```

---

# 70. Histogram Quantile by Service

Example:

```promql
histogram_quantile(
  0.95,
  sum by (service, le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

This returns a 95th percentile latency per service.

---

# 71. Important Histogram Rule

When aggregating histogram buckets, retain:

```text
le
```

Therefore:

```promql
sum by (service, le)
```

is generally needed before:

```promql
histogram_quantile()
```

---

# 72. `_sum` and `_count`

For a histogram:

```text
request_duration_seconds_sum
```

represents the sum of observed durations.

```text
request_duration_seconds_count
```

represents the number of observations.

Average latency can be calculated:

```promql
rate(request_duration_seconds_sum[5m])
/
rate(request_duration_seconds_count[5m])
```

---

# 73. Average Request Latency

Example:

```promql
sum(
  rate(http_request_duration_seconds_sum[5m])
)
/
sum(
  rate(http_request_duration_seconds_count[5m])
)
```

This calculates average request duration over the selected period.

---

# 74. Counter Example: HTTP Requests

Metric:

```text
http_requests_total
```

Request rate:

```promql
sum(
  rate(http_requests_total[5m])
)
```

---

# 75. Error Rate

Suppose:

```text
http_requests_total{
  status="500"
}
```

Error requests:

```promql
sum(
  rate(http_requests_total{status=~"5.."}[5m])
)
```

Total requests:

```promql
sum(
  rate(http_requests_total[5m])
)
```

---

# 76. Error Percentage

```promql
(
  sum(
    rate(http_requests_total{status=~"5.."}[5m])
  )
  /
  sum(
    rate(http_requests_total[5m])
  )
) * 100
```

This produces a percentage of 5xx traffic.

---

# 77. Error Rate by Service

```promql
(
  sum by (service) (
    rate(http_requests_total{status=~"5.."}[5m])
  )
  /
  sum by (service) (
    rate(http_requests_total[5m])
  )
) * 100
```

This is useful for microservices dashboards.

---

# 78. Request Rate by Service

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

This answers:

```
How much traffic is each service receiving?
```

---

# 79. Request Rate by HTTP Method

```promql
sum by (method) (
  rate(http_requests_total[5m])
)
```

---

# 80. Request Rate by Status Code

```promql
sum by (status) (
  rate(http_requests_total[5m])
)
```

---

# 81. 5xx Requests by Service

```promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
```

---

# 82. 4xx Requests by Service

```promql
sum by (service) (
  rate(http_requests_total{status=~"4.."}[5m])
)
```

---

# 83. CPU Utilization

A common node CPU calculation uses:

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

This estimates CPU utilization per instance.

---

# 84. CPU by Kubernetes Node

Depending on metric labels:

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

Then map instance/node labels appropriately in your environment.

---

# 85. Memory Utilization

A common node memory utilization query:

```promql
100 * (
  1 -
  (
    node_memory_MemAvailable_bytes
    /
    node_memory_MemTotal_bytes
  )
)
```

This calculates approximately:

```
Used Memory Percentage
```

---

# 86. Memory Available

```promql
node_memory_MemAvailable_bytes
```

Useful for identifying nodes under memory pressure.

---

# 87. Disk Utilization

A common Linux filesystem calculation:

```promql
100 * (
  1 -
  (
    node_filesystem_avail_bytes
    /
    node_filesystem_size_bytes
  )
)
```

Filter filesystems as appropriate for your environment.

---

# 88. Disk Usage by Mountpoint

```promql
100 * (
  1 -
  (
    node_filesystem_avail_bytes
    /
    node_filesystem_size_bytes
  )
)
```

Group or filter using labels such as:

```
instance

mountpoint

device

fstype
```

---

# 89. Filesystem Filtering

A production query should avoid pseudo filesystems.

Example:

```promql
node_filesystem_size_bytes{
  fstype!~"tmpfs|overlay"
}
```

The exact filters should be adapted to the exporter and operating system.

---

# 90. Node Load

Example:

```promql
node_load1
```

This shows the one-minute load average exposed by Node Exporter.

Load average should be interpreted relative to:

```
CPU Count

Workload

Scheduler Behavior
```

It is not identical to CPU utilization.

---

# 91. Kubernetes Pod Count

Depending on kube-state-metrics:

```promql
count(
  kube_pod_info
)
```

can be used to count pod series.

For namespaces:

```promql
count by (namespace) (
  kube_pod_info
)
```

---

# 92. Running Pods

Depending on the available metrics:

```promql
sum by (namespace) (
  kube_pod_status_phase{phase="Running"}
)
```

This can help identify running pods.

---

# 93. Pending Pods

```promql
sum by (namespace) (
  kube_pod_status_phase{phase="Pending"}
)
```

This can identify pending pods.

---

# 94. Failed Pods

```promql
sum by (namespace) (
  kube_pod_status_phase{phase="Failed"}
)
```

---

# 95. Pod Restart Rate

Container restart counts are counters.

A common query:

```promql
rate(
  kube_pod_container_status_restarts_total[15m]
)
```

This shows restart rate.

For identifying containers with recent restarts:

```promql
increase(
  kube_pod_container_status_restarts_total[15m]
) > 0
```

---

# 96. OOMKilled Containers

Depending on available kube-state-metrics:

```promql
kube_pod_container_status_last_terminated_reason{
  reason="OOMKilled"
}
```

This can identify containers whose last termination reason was OOMKilled.

---

# 97. Pod CPU Usage

Depending on the metrics available from cAdvisor/kubelet:

```promql
sum by (namespace, pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

You should normally filter out irrelevant infrastructure series.

---

# 98. Pod Memory Usage

Example:

```promql
sum by (namespace, pod) (
  container_memory_working_set_bytes
)
```

Again, apply appropriate filters for your environment.

---

# 99. Kubernetes CPU Requests

Depending on kube-state-metrics:

```promql
sum by (namespace, pod) (
  kube_pod_container_resource_requests{
    resource="cpu"
  }
)
```

Metric naming can vary by kube-state-metrics version, so verify the actual metric in your cluster.

---

# 100. Kubernetes Memory Requests

Conceptually:

```promql
sum by (namespace, pod) (
  kube_pod_container_resource_requests{
    resource="memory"
  }
)
```

Verify the exact metric and labels available in your deployment.

---

# 101. CPU Usage vs CPU Requests

A useful capacity query compares:

```text
Actual Usage
     /
Requested CPU
```

For example:

```promql
sum by (namespace, pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
/
sum by (namespace, pod) (
  kube_pod_container_resource_requests{
    resource="cpu"
  }
)
```

The exact join may require label alignment depending on your metrics.

---

# 102. CPU Usage vs Limits

Similarly:

```text
Actual Usage
     /
CPU Limit
```

This can help identify containers approaching configured limits.

---

# 103. Kubernetes Node Readiness

Depending on kube-state-metrics:

```promql
kube_node_status_condition{
  condition="Ready",
  status="true"
}
```

This can be used to monitor node readiness.

---

# 104. Not Ready Nodes

A common approach:

```promql
kube_node_status_condition{
  condition="Ready",
  status="true"
} == 0
```

Verify metric semantics and labels in your installed kube-state-metrics version.

---

# 105. Deployment Available Replicas

Depending on kube-state-metrics:

```promql
kube_deployment_status_replicas_available
```

This shows available replicas.

---

# 106. Deployment Desired Replicas

```promql
kube_deployment_spec_replicas
```

This represents the desired number of replicas.

---

# 107. Deployment Replica Difference

```promql
kube_deployment_spec_replicas
-
kube_deployment_status_replicas_available
```

This can help identify replica shortages.

---

# 108. Deployment Availability Check

Conceptually:

```promql
kube_deployment_status_replicas_available
<
kube_deployment_spec_replicas
```

This identifies deployments with fewer available replicas than desired.

---

# 109. StatefulSet Replicas

Depending on kube-state-metrics:

```promql
kube_statefulset_status_replicas_ready
```

can be compared with desired replicas.

---

# 110. DaemonSet Availability

Depending on metrics available:

```promql
kube_daemonset_status_number_ready
```

can be compared with:

```promql
kube_daemonset_status_desired_number_scheduled
```

---

# 111. HPA Current Replicas

Depending on kube-state-metrics:

```promql
kube_horizontalpodautoscaler_status_current_replicas
```

This can show current HPA replica count.

---

# 112. HPA Desired Replicas

```promql
kube_horizontalpodautoscaler_status_desired_replicas
```

This can show desired replicas.

---

# 113. HPA Maximum Replicas

```promql
kube_horizontalpodautoscaler_spec_max_replicas
```

This can help determine whether HPA has reached its configured maximum.

---

# 114. HPA at Maximum

Conceptually:

```promql
kube_horizontalpodautoscaler_status_current_replicas
==
kube_horizontalpodautoscaler_spec_max_replicas
```

This can indicate that the HPA has reached its maximum configured replica count.

---

# 115. Querying with `by`

Example:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

`by` specifies which labels should remain in the aggregated result.

---

# 116. Querying with `without`

Example:

```promql
sum without(instance) (
  rate(http_requests_total[5m])
)
```

This removes the specified label from the grouping dimensions.

---

# 117. `sum by` vs `sum without`

Use:

```promql
sum by (service)
```

when you explicitly want:

```
service
```

to remain.

Use:

```promql
sum without(instance)
```

when you want to aggregate away:

```
instance
```

while retaining other labels.

---

# 118. Vector Matching

PromQL becomes more advanced when combining different metrics.

Example:

```text
Metric A
service="order"

Metric B
service="order"
```

PromQL can match these based on labels.

---

# 119. One-to-One Vector Matching

Example:

```promql
metric_a
/
metric_b
```

Prometheus tries to match series based on their labels.

If label sets do not match appropriately, the result may be empty or unexpected.

---

# 120. `on()`

`on()` specifies the labels used for vector matching.

Example:

```promql
metric_a
/
on(service)
metric_b
```

This means:

```
Match series using the service label.
```

---

# 121. `ignoring()`

`ignoring()` tells Prometheus to ignore specified labels during matching.

Example:

```promql
metric_a
/
ignoring(instance)
metric_b
```

This can be useful when two metrics differ only by instance.

---

# 122. Many-to-One Matching

Sometimes one side contains more series than the other.

PromQL supports modifiers such as:

```text
group_left
group_right
```

Example:

```promql
metric_a
/
on(service)
group_left(team)
metric_b
```

Use these carefully because incorrect many-to-one matching can produce unexpected results or duplicate series.

---

# 123. Example: Usage vs Request

Suppose:

```text
CPU Usage
```

has:

```
namespace

pod

container
```

while resource requests contain:

```
namespace

pod

container
```

If the labels align, they can be divided directly.

If additional labels exist, vector matching may be required.

---

# 124. `abs()`

Returns absolute value.

Example:

```promql
abs(metric)
```

---

# 125. `ceil()`

Rounds values upward.

Example:

```promql
ceil(metric)
```

---

# 126. `floor()`

Rounds values downward.

Example:

```promql
floor(metric)
```

---

# 127. `round()`

Rounds values.

Example:

```promql
round(metric)
```

---

# 128. `clamp_min()`

Example:

```promql
clamp_min(metric, 0)
```

This ensures values do not go below zero.

---

# 129. `clamp_max()`

Example:

```promql
clamp_max(metric, 100)
```

This limits values to a maximum of 100.

---

# 130. `absent()`

`absent()` can be used to detect when a series is missing.

Example:

```promql
absent(up{job="critical-service"})
```

This can help identify missing monitoring data.

---

# 131. `absent_over_time()`

Example:

```promql
absent_over_time(
  up{job="critical-service"}[10m]
)
```

This can help detect when a series has been absent throughout a time range.

---

# 132. `changes()`

Example:

```promql
changes(
  kube_deployment_status_replicas_available[30m]
)
```

This counts changes in a gauge over the selected range.

---

# 133. `delta()`

For gauges:

```promql
delta(
  node_load1[1h]
)
```

This calculates the difference between the first and last value in the range, with PromQL's range semantics.

Do not use `delta()` as a substitute for counter functions.

---

# 134. `deriv()`

`deriv()` calculates the per-second derivative of a gauge using linear regression.

Example:

```promql
deriv(
  node_load1[30m]
)
```

This can help identify trends in gauge values.

---

# 135. `predict_linear()`

Example:

```promql
predict_linear(
  node_filesystem_avail_bytes[6h],
  4 * 3600
)
```

This estimates a future value using linear regression.

It can be useful for disk capacity forecasting, but should be treated as an estimate rather than a guarantee.

---

# 136. Disk Full Forecast

A common example:

```promql
predict_linear(
  node_filesystem_avail_bytes{
    fstype!~"tmpfs|overlay"
  }[6h],
  4 * 3600
) < 0
```

This can identify filesystems projected to run out of available space within approximately four hours, assuming the recent linear trend continues.

---

# 137. `resets()`

For counters:

```promql
resets(
  http_requests_total[1h]
)
```

This counts counter resets during the range.

Useful for detecting frequent application restarts or exporter restarts.

---

# 138. `last_over_time()`

Example:

```promql
last_over_time(
  metric[15m]
)
```

This retrieves the most recent sample in the range.

---

# 139. `present_over_time()`

Example:

```promql
present_over_time(
  metric[15m]
)
```

This can help determine whether a series had samples during the selected period.

---

# 140. `stddev()`

Example:

```promql
stddev(
  metric
)
```

This calculates standard deviation across series.

---

# 141. `stdvar()`

Example:

```promql
stdvar(
  metric
)
```

This calculates variance across series.

---

# 142. `count_values()`

Example:

```promql
count_values(
  "version",
  application_build_info
)
```

This can count how many series have each distinct value of the selected label/value dimension.

This is useful for identifying version distribution across instances.

---

# 143. Version Distribution

Suppose:

```text
application_build_info{
  version="1.0"
}

application_build_info{
  version="1.1"
}

application_build_info{
  version="1.1"
}
```

A query can help determine:

```text
Version 1.0 → 1 instance
Version 1.1 → 2 instances
```

This is useful during deployments.

---

# 144. `sort()`

Example:

```promql
sort(
  metric
)
```

Sorts values ascending.

---

# 145. `sort_desc()`

Example:

```promql
sort_desc(
  metric
)
```

Sorts values descending.

Useful for dashboard tables and top-value views.

---

# 146. `label_replace()`

`label_replace()` can create or modify labels using regex.

Conceptual example:

```promql
label_replace(
  metric,
  "service",
  "$1",
  "job",
  "(.*)"
)
```

This can transform label information.

Use it carefully because label manipulation can increase query complexity.

---

# 147. `label_join()`

`label_join()` combines label values into another label.

Example concept:

```promql
label_join(
  metric,
  "target",
  ":",
  "namespace",
  "pod"
)
```

This can create a combined label such as:

```text
namespace:pod
```

---

# 148. Time Functions

PromQL provides functions related to time.

Examples include:

```promql
time()
```

and:

```promql
day_of_week()
```

These can be useful in specialized queries.

---

# 149. `time()`

Example:

```promql
time()
```

Returns the current evaluation timestamp in Unix time.

---

# 150. `vector()`

Example:

```promql
vector(1)
```

This creates a vector containing a scalar value.

It can be useful in more advanced PromQL expressions.

---

# 151. `scalar()`

Example:

```promql
scalar(
  sum(up)
)
```

This converts a single-element vector into a scalar.

It should be used only when the expression actually returns one meaningful series.

---

# 152. `clamp()`

PromQL supports clamping between minimum and maximum values.

Example:

```promql
clamp(metric, 0, 100)
```

This limits the result to:

```
Minimum = 0

Maximum = 100
```

---

# 153. Subqueries

PromQL supports subqueries for advanced analysis.

Example:

```promql
rate(
  http_requests_total[5m]
)[1h:1m]
```

A subquery evaluates an expression over a range with a specified resolution.

Subqueries are powerful but can be computationally expensive.

---

# 154. Subquery Use Case

A subquery can be useful when you want to analyze:

```
Rate over time

Maximum rate

Average rate
```

For example:

```promql
max_over_time(
  rate(http_requests_total[5m])[1h:1m]
)
```

This finds the maximum five-minute rate observed over approximately the last hour.

---

# 155. Recording Rules

Recording rules precompute frequently used PromQL expressions.

Example:

```yaml
groups:
  - name: service-recording
    rules:
      - record: service:http_requests_rate5m
        expr: |
          sum by (service) (
            rate(http_requests_total[5m])
          )
```

Then query:

```promql
service:http_requests_rate5m
```

---

# 156. Why Use Recording Rules

Recording rules are useful when:

```
Queries are expensive

Queries are used frequently

Dashboards use the same expression

Alerts use the same expression

Large datasets are involved
```

Instead of recalculating:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

every time, Prometheus can store the result as a new time series.

---

# 157. Recording Rule Naming

A common naming convention:

```text
level:metric:operations
```

Example:

```text
service:http_requests_rate5m
```

Another:

```text
namespace:container_cpu_usage:sum_rate5m
```

Use consistent naming across your monitoring platform.

---

# 158. Alerting Rules

Example:

```yaml
groups:
  - name: service-alerts
    rules:
      - alert: HighErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High HTTP error rate"
```

---

# 159. PromQL and Alerting

PromQL powers alerts.

The flow is:

```text
Metrics
   ↓
PromQL
   ↓
Condition
   ↓
Alerting Rule
   ↓
Alertmanager
   ↓
Notification
```

---

# 160. PromQL and Grafana

Grafana can use Prometheus as a data source.

Architecture:

```text
Prometheus
    ↓
PromQL
    ↓
Grafana
    ↓
Dashboard
```

A Grafana panel may contain:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

---

# 161. Dashboard Query Example

Panel:

```
Request Rate by Service
```

PromQL:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

Visualization:

```
Time Series
```

This allows engineers to see traffic trends across services.

---

# 162. Dashboard: Error Rate

PromQL:

```promql
(
  sum by (service) (
    rate(http_requests_total{status=~"5.."}[5m])
  )
  /
  sum by (service) (
    rate(http_requests_total[5m])
  )
) * 100
```

Visualization:

```
Time Series

Table

Stat
```

depending on the dashboard requirement.

---

# 163. Dashboard: CPU

PromQL:

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

---

# 164. Dashboard: Memory

PromQL:

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

---

# 165. Dashboard: Disk

PromQL:

```promql
100 * (
  1 -
  node_filesystem_avail_bytes
  /
  node_filesystem_size_bytes
)
```

Filter pseudo filesystems as appropriate.

---

# 166. Dashboard: Pod Restarts

PromQL:

```promql
sum by (namespace, pod) (
  increase(
    kube_pod_container_status_restarts_total[1h]
  )
)
```

This shows restart counts over the last hour.

---

# 167. Dashboard: Pod CPU

```promql
sum by (namespace, pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

---

# 168. Dashboard: Pod Memory

```promql
sum by (namespace, pod) (
  container_memory_working_set_bytes
)
```

---

# 169. Dashboard: Deployment Health

A conceptual query:

```promql
kube_deployment_status_replicas_available
<
kube_deployment_spec_replicas
```

This can identify deployments where available replicas are below desired replicas.

---

# 170. Dashboard: Node Availability

A common query:

```promql
kube_node_status_condition{
  condition="Ready",
  status="true"
}
```

---

# 171. Golden Signals with PromQL

The four Golden Signals are:

```
Latency
Traffic
Errors
Saturation
```

PromQL can calculate all four.

---

# 172. Traffic

Example:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

---

# 173. Errors

Example:

```promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
```

---

# 174. Error Percentage

```promql
(
  sum by (service) (
    rate(http_requests_total{status=~"5.."}[5m])
  )
  /
  sum by (service) (
    rate(http_requests_total[5m])
  )
) * 100
```

---

# 175. Latency

For histogram metrics:

```promql
histogram_quantile(
  0.95,
  sum by (service, le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

---

# 176. Saturation

Saturation depends on the resource.

CPU:

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

Memory:

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

---

# 177. SLI Example

Suppose the SLI is:

```
Successful HTTP requests
```

PromQL:

```promql
sum(
  rate(http_requests_total{status=~"2.."}[5m])
)
/
sum(
  rate(http_requests_total[5m])
)
```

This gives an approximate success ratio.

---

# 178. Availability Percentage

```promql
(
  sum(
    rate(http_requests_total{status!~"5.."}[5m])
  )
  /
  sum(
    rate(http_requests_total[5m])
  )
) * 100
```

The exact definition of "available" must match your organization's SLI.

---

# 179. SLO Alert Example

Suppose the objective is:

```
99.9% successful requests
```

Then the allowed error ratio is:

```
0.1%
```

A PromQL expression can calculate the current error ratio.

Example:

```promql
sum(
  rate(http_requests_total{status=~"5.."}[5m])
)
/
sum(
  rate(http_requests_total[5m])
)
```

Then compare it with:

```text
0.001
```

---

# 180. Burn Rate Concept

SLO burn-rate alerting compares:

```
Current Error Rate
```

against:

```
Allowed Error Budget Consumption
```

PromQL can be used to calculate burn rate.

A production implementation normally uses multiple windows, such as:

```
Short Window

Long Window
```

to reduce false positives.

---

# 181. Multi-Window Burn Rate

Conceptually:

```promql
short_window_error_rate
/
allowed_error_rate
```

and:

```promql
long_window_error_rate
/
allowed_error_rate
```

Then combine them using an alerting rule.

The exact thresholds should be designed according to the SLO.

---

# 182. PromQL Query Performance

PromQL queries can become expensive.

Expensive queries may:

```
Consume CPU

Consume Memory

Increase Query Latency

Affect Grafana

Affect Prometheus Stability
```

---

# 183. Avoid Extremely Broad Queries

Instead of:

```promql
rate(http_requests_total[5m])
```

across an enormous environment, narrow it when possible:

```promql
rate(
  http_requests_total{
    environment="production",
    service="order"
  }[5m]
)
```

This can reduce unnecessary work.

---

# 184. Avoid High-Cardinality Aggregation

Be careful with:

```promql
sum by (
  user_id
) (
  rate(http_requests_total[5m])
)
```

if `user_id` has huge cardinality.

This can create expensive query results.

---

# 185. Use Recording Rules

If a query is repeatedly used:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

consider a recording rule.

Then dashboards query:

```promql
service:http_requests_rate5m
```

---

# 186. Query Range Selection

Avoid unnecessarily large ranges.

Instead of:

```promql
rate(http_requests_total[30d])
```

when you only need a short-term rate, use an appropriate range such as:

```promql
rate(http_requests_total[5m])
```

Long ranges can be expensive.

---

# 187. Query Optimization Workflow

When a query is slow:

```text
Query
 ↓
Check Selector
 ↓
Check Cardinality
 ↓
Check Range
 ↓
Check Aggregation
 ↓
Check Vector Matching
 ↓
Consider Recording Rule
```

---

# 188. PromQL Anti-Pattern: Huge Regex

Avoid overly broad regex expressions such as:

```promql
metric{
  service=~".*"
}
```

when they are unnecessary.

Prefer precise selectors where possible.

---

# 189. PromQL Anti-Pattern: Unbounded Labels

Avoid queries based on high-cardinality labels such as:

```text
user_id
request_id
session_id
```

unless there is a very specific and controlled use case.

---

# 190. PromQL Anti-Pattern: Using `irate()` Everywhere

Do not automatically use:

```promql
irate()
```

for every counter.

For most dashboards and alerts:

```promql
rate()
```

provides a more stable signal.

---

# 191. PromQL Anti-Pattern: Rate on Gauges

Do not use:

```promql
rate(node_memory_MemAvailable_bytes[5m])
```

just because you need a trend.

Memory available is a gauge.

Use functions appropriate for gauges.

---

# 192. PromQL Anti-Pattern: Average of Percentages

Suppose two services have:

```text
Service A → 1% error
Service B → 20% error
```

A simple average:

```text
(1 + 20) / 2
```

may not represent the actual platform error percentage if traffic volumes differ.

Prefer aggregating numerator and denominator first:

```promql
sum(errors)
/
sum(total_requests)
```

---

# 193. PromQL Anti-Pattern: Incorrect Histogram Aggregation

Do not calculate latency percentile like:

```promql
quantile(
  0.95,
  request_duration_seconds
)
```

if the metric is represented as histogram buckets.

Use:

```promql
histogram_quantile()
```

with `_bucket` metrics.

---

# 194. PromQL Best Practice: Use Meaningful Labels

Good:

```text
service
environment
namespace
cluster
region
method
status
```

Avoid uncontrolled dimensions.

---

# 195. PromQL Best Practice: Use Consistent Metric Names

For example:

```text
http_requests_total
http_request_duration_seconds
```

rather than different names for each service.

Consistency makes dashboards reusable.

---

# 196. PromQL Best Practice: Use `rate()` for Counters

For example:

```promql
rate(http_requests_total[5m])
```

rather than interpreting the raw counter value as traffic rate.

---

# 197. PromQL Best Practice: Aggregate Intentionally

Instead of:

```promql
sum(metric)
```

ask:

```
What dimensions should remain?
```

For example:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

---

# 198. PromQL Best Practice: Test Queries in Prometheus

Before using a query in Grafana or an alert:

```text
Prometheus UI
     ↓
Execute Query
     ↓
Check Result
     ↓
Validate Labels
     ↓
Check Time Range
```

---

# 199. PromQL Best Practice: Test Alerts

An alert query should be tested against real metrics.

Check:

```
Normal State

Warning State

Critical State

Recovery State
```

---

# 200. PromQL Best Practice: Use Recording Rules for Expensive Queries

If the same query appears in:

```
Grafana

Alerts

Reports

SLO calculations
```

consider creating a recording rule.

---

# 201. Real-World Microservices PromQL

Suppose the platform contains:

```text
user
product
cart
order
payment
inventory
```

Request rate:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

Error rate:

```promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
```

Error percentage:

```promql
(
  sum by (service) (
    rate(http_requests_total{status=~"5.."}[5m])
  )
  /
  sum by (service) (
    rate(http_requests_total[5m])
  )
) * 100
```

---

# 202. Real-World EKS Monitoring Queries

Node CPU:

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

Node memory:

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

Pod restarts:

```promql
sum by (namespace, pod) (
  increase(kube_pod_container_status_restarts_total[1h])
)
```

---

# 203. Production Incident: 5xx Spike

Suppose users report:

```
HTTP 500 errors
```

First query:

```promql
sum(
  rate(http_requests_total{status=~"5.."}[5m])
)
```

Then:

```promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
```

This identifies which service is generating the errors.

Then inspect:

```
Logs

Deployment

Pods

Dependencies

Database

Recent Releases
```

---

# 204. Production Incident: Latency Increase

Start with:

```promql
histogram_quantile(
  0.95,
  sum by (service, le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

Then identify the affected service.

Next investigate:

```
CPU

Memory

Database

Network

External APIs

Recent Deployment

Pod Restarts
```

---

# 205. Production Incident: High CPU

Start with:

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

Then identify affected nodes.

Next:

```promql
sum by (namespace, pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

This helps identify high-CPU pods.

Then inspect:

```
Deployment

Requests/Limits

HPA

Application behavior
```

---

# 206. Production Incident: High Memory

Node level:

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

Pod level:

```promql
sum by (namespace, pod) (
  container_memory_working_set_bytes
)
```

Then inspect:

```
OOMKilled

Memory Limits

Application Memory Usage

Node Pressure
```

---

# 207. Production Incident: Pod Restarts

Query:

```promql
sum by (namespace, pod) (
  increase(kube_pod_container_status_restarts_total[15m])
)
```

Then investigate:

```
CrashLoopBackOff

OOMKilled

Liveness Probe

Readiness Probe

Application Logs
```

---

# 208. Production Incident: Node Not Ready

Query:

```promql
kube_node_status_condition{
  condition="Ready",
  status="true"
} == 0
```

Then investigate:

```
Node health

Kubelet

Network

Disk pressure

Memory pressure

AWS infrastructure
```

---

# 209. Production Incident: Deployment Not Available

Query:

```promql
kube_deployment_status_replicas_available
<
kube_deployment_spec_replicas
```

Then inspect:

```
Deployment

ReplicaSet

Pods

Scheduling

ImagePullBackOff

Resource Constraints

Probes
```

---

# 210. PromQL and DevOps Troubleshooting

PromQL should not be used alone.

A real incident workflow is:

```text
PromQL
 ↓
Identify Problem
 ↓
Grafana
 ↓
Logs
 ↓
Kubernetes
 ↓
Application
 ↓
Root Cause
```

Metrics tell you:

```
What is happening?
```

Logs often help explain:

```
Why is it happening?
```

Traces can help determine:

```
Where is the request spending time?
```

---

# 211. PromQL and Observability Stack

Your monitoring stack can be:

```text
                 Applications
                 /     |      \
                /      |       \
           Metrics    Logs     Traces
              ↓        ↓         ↓
         Prometheus    ELK   OpenTelemetry
              ↓                  ↓
           Grafana             Jaeger
```

PromQL is the query language for the metrics side.

---

# 212. PromQL and Grafana Variables

Grafana can create variables such as:

```text
$cluster
$namespace
$service
```

Then PromQL can use them.

Example:

```promql
sum by (service) (
  rate(
    http_requests_total{
      cluster="$cluster",
      namespace="$namespace"
    }[5m]
  )
)
```

This allows one dashboard to work across multiple environments.

---

# 213. Dashboard Variable Example

Suppose:

```text
cluster = prod-eks
namespace = production
```

The query becomes:

```promql
sum by (service) (
  rate(
    http_requests_total{
      cluster="prod-eks",
      namespace="production"
    }[5m]
  )
)
```

---

# 214. Multi-Cluster Dashboard

Use:

```promql
sum by (cluster) (
  rate(http_requests_total[5m])
)
```

This gives traffic per cluster.

---

# 215. Multi-Region Dashboard

Use:

```promql
sum by (region) (
  rate(http_requests_total[5m])
)
```

This gives traffic per AWS region.

---

# 216. Multi-Environment Dashboard

Use:

```promql
sum by (environment) (
  rate(http_requests_total[5m])
)
```

This compares:

```
Development

Staging

Production
```

---

# 217. PromQL for Deployment Version

Suppose:

```text
application_build_info
```

contains:

```text
version
```

You can use it to inspect deployed versions.

Example:

```promql
count by (version) (
  application_build_info
)
```

This helps during rolling deployments.

---

# 218. Detect Multiple Versions

A useful deployment check is:

```promql
count by (version) (
  application_build_info
)
```

If two versions appear during a rollout:

```text
1.0 → 4 pods
1.1 → 6 pods
```

the deployment is partially rolled out.

---

# 219. Detect Version Rollout

During deployment:

```text
Old Version
    ↓
New Version
    ↓
Gradual Replacement
```

PromQL can help observe:

```
Version distribution

Error rate by version

Latency by version

Traffic by version
```

---

# 220. Error Rate by Version

```promql
sum by (version) (
  rate(
    http_requests_total{
      status=~"5.."
    }[5m]
  )
)
```

This is useful if the request metric includes a version label.

---

# 221. Canary Deployment Analysis

For a canary:

```text
Version 1.0 → 90%
Version 1.1 → 10%
```

Compare:

```
Error Rate

Latency

Traffic
```

between versions.

Example:

```promql
sum by (version) (
  rate(http_requests_total[5m])
)
```

---

# 222. Canary Error Rate

```promql
(
  sum by (version) (
    rate(http_requests_total{status=~"5.."}[5m])
  )
  /
  sum by (version) (
    rate(http_requests_total[5m])
  )
) * 100
```

This helps compare error percentages across versions.

---

# 223. PromQL and CI/CD

PromQL can support deployment verification.

Deployment flow:

```text
Deploy
 ↓
Wait
 ↓
Query Prometheus
 ↓
Check Error Rate
 ↓
Check Latency
 ↓
Check Availability
 ↓
Promote or Rollback
```

---

# 224. Automated Deployment Verification

A CI/CD pipeline can evaluate conditions such as:

```text
5xx rate < threshold

Latency < threshold

Pods healthy

Availability > threshold
```

If the conditions fail:

```text
Rollback
```

---

# 225. PromQL in Jenkins/GitHub Actions

A deployment pipeline can query the Prometheus HTTP API.

Conceptually:

```text
Jenkins / GitHub Actions
          ↓
     Prometheus API
          ↓
        PromQL
          ↓
      Deployment
      Validation
```

This enables automated post-deployment checks.

---

# 226. PromQL and ArgoCD

A GitOps deployment can also integrate metrics into progressive delivery.

Conceptually:

```text
ArgoCD
  ↓
Deployment
  ↓
Prometheus
  ↓
PromQL
  ↓
Health Metrics
  ↓
Promotion / Rollback
```

The exact implementation depends on the progressive delivery tooling used.

---

# 227. PromQL Query Naming

For frequently reused queries, prefer recording rules with meaningful names.

Example:

```text
service:http_requests_rate5m
service:http_5xx_rate5m
service:http_error_ratio5m
```

This makes dashboards easier to maintain.

---

# 228. PromQL Documentation

For production teams, document important queries.

Example:

```text
Query:
service:http_error_ratio5m

Meaning:
Percentage of HTTP requests returning 5xx responses.

Used By:
Grafana
Alerting
SLO Dashboard
```

---

# 229. PromQL Code Review

Review PromQL like application code.

Check:

```
Correct Metric

Correct Labels

Correct Aggregation

Correct Time Window

Correct Units

Cardinality

Performance

Business Meaning
```

---

# 230. PromQL Testing

Before production alerting:

```text
Normal Traffic
      ↓
Query
      ↓
Expected Result
      ↓
High Error Traffic
      ↓
Query
      ↓
Alert
```

Use realistic test data and rule testing where applicable.

---

# 231. PromQL Units

Always understand the unit returned by the query.

Examples:

```text
rate(counter[5m])
    ↓
per second
```

```text
node_memory_MemAvailable_bytes
    ↓
bytes
```

```text
error ratio
    ↓
0 to 1
```

```text
error percentage
    ↓
0 to 100
```

Incorrect units can produce misleading dashboards and alerts.

---

# 232. Percentage vs Ratio

Ratio:

```promql
errors / total
```

returns:

```text
0.05
```

which means:

```text
5%
```

Percentage:

```promql
(errors / total) * 100
```

returns:

```text
5
```

Always label Grafana panels correctly.

---

# 233. Bytes to GiB

PromQL can convert bytes to GiB:

```promql
node_memory_MemTotal_bytes
/
1024
/
1024
/
1024
```

This produces approximately:

```
GiB
```

---

# 234. Bytes to GB

Decimal GB:

```promql
node_memory_MemTotal_bytes
/
1000000000
```

Use the unit that matches the dashboard requirement.

---

# 235. Seconds to Milliseconds

Example:

```promql
http_latency_seconds * 1000
```

This converts:

```
seconds
```

to:

```
milliseconds
```

---

# 236. Query Result Validation

When building a query, ask:

```
What metric am I selecting?

What labels exist?

What type is the metric?

What is the unit?

What range should I use?

What aggregation do I need?

What dimensions should remain?

Is cardinality safe?
```

---

# 237. PromQL Production Checklist

## Metric Selection

```
[ ] Correct metric

[ ] Correct labels

[ ] Correct metric type
```

---

## Time Range

```
[ ] Appropriate range

[ ] Not unnecessarily large
```

---

## Functions

```
[ ] rate for counters

[ ] Appropriate functions for gauges

[ ] histogram_quantile for histogram buckets
```

---

## Aggregation

```
[ ] Correct by labels

[ ] Correct without labels

[ ] No accidental high cardinality
```

---

## Performance

```
[ ] Query is reasonably efficient

[ ] Recording rule considered

[ ] Range is appropriate
```

---

## Dashboard

```
[ ] Unit correct

[ ] Legend meaningful

[ ] Environment filter available

[ ] Service filter available
```

---

# 238. Interview Question: What Is PromQL?

### Answer

PromQL is the query language used by Prometheus to select, filter, transform, aggregate, and analyze time-series metrics.

I use PromQL for:

```
Grafana dashboards

Alerting rules

Recording rules

Troubleshooting

SLO calculations
```

For example:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

gives request rate grouped by service.

---

# 239. Interview Question: What Is the Difference Between Instant Vector and Range Vector?

### Answer

An instant vector represents the current value of multiple time series at a specific evaluation time.

Example:

```promql
up
```

A range vector contains samples over a time range.

Example:

```promql
up[5m]
```

Range vectors are commonly passed to functions such as:

```promql
rate()
```

and:

```promql
increase()
```

---

# 240. Interview Question: What Is the Difference Between `rate()` and `irate()`?

### Answer

`rate()` calculates an average per-second increase over a selected range and is generally more stable.

Example:

```promql
rate(http_requests_total[5m])
```

`irate()` focuses more heavily on the most recent samples and is more sensitive to short-term changes.

For most dashboards and alerting, I prefer `rate()` unless there is a specific reason to use `irate()`.

---

# 241. Interview Question: What Is the Difference Between `rate()` and `increase()`?

### Answer

`rate()` gives a per-second rate:

```promql
rate(http_requests_total[5m])
```

`increase()` gives the total increase over the selected range:

```promql
increase(http_requests_total[1h])
```

So:

```
rate = per-second rate

increase = total increase
```

---

# 242. Interview Question: How Do You Calculate HTTP Error Rate?

### Answer

I first calculate the 5xx request rate:

```promql
sum(
  rate(http_requests_total{status=~"5.."}[5m])
)
```

Then divide it by total request rate:

```promql
sum(
  rate(http_requests_total{status=~"5.."}[5m])
)
/
sum(
  rate(http_requests_total[5m])
)
```

If I need a percentage, I multiply by 100.

---

# 243. Interview Question: How Do You Calculate P95 Latency?

### Answer

If the application exposes Prometheus histogram buckets, I use `histogram_quantile()`.

Example:

```promql
histogram_quantile(
  0.95,
  sum by (service, le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

The important point is that histogram buckets must be aggregated while retaining the `le` label.

---

# 244. Interview Question: Why Do You Use `sum by (service)`?

### Answer

A microservice can have multiple pods or instances.

For example:

```text
order-pod-1
order-pod-2
order-pod-3
```

I may want the total request rate for the entire service rather than each pod.

So I use:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

This aggregates across instances while retaining the service dimension.

---

# 245. Interview Question: What Is the Difference Between `by` and `without`?

### Answer

`by` specifies which labels should remain after aggregation.

Example:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

`without` specifies which labels should be removed from the grouping.

Example:

```promql
sum without(instance) (
  rate(http_requests_total[5m])
)
```

---

# 246. Interview Question: How Do You Calculate CPU Utilization?

### Answer

Using Node Exporter CPU seconds, I can calculate utilization by taking one minus the idle CPU rate.

Example:

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

This gives an approximate CPU utilization percentage per instance.

---

# 247. Interview Question: How Do You Calculate Memory Utilization?

### Answer

I use total memory and available memory.

Example:

```promql
100 * (
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

This gives approximate memory utilization percentage.

---

# 248. Interview Question: How Do You Find Pods With Frequent Restarts?

### Answer

I use the container restart counter and calculate its increase over a time range.

Example:

```promql
sum by (namespace, pod) (
  increase(
    kube_pod_container_status_restarts_total[1h]
  )
) > 0
```

Then I correlate the result with:

```
Pod Events

Logs

OOMKilled

Probe Failures

Application Errors
```

---

# 249. Interview Question: How Do You Find High-CPU Pods?

### Answer

I calculate container CPU usage and aggregate it by namespace and pod.

Example:

```promql
sum by (namespace, pod) (
  rate(container_cpu_usage_seconds_total[5m])
)
```

Then I can use `topk()`:

```promql
topk(
  10,
  sum by (namespace, pod) (
    rate(container_cpu_usage_seconds_total[5m])
  )
)
```

This gives the top CPU-consuming pods.

---

# 250. Interview Question: How Do You Find High-Memory Pods?

### Answer

I use container memory working set:

```promql
topk(
  10,
  sum by (namespace, pod) (
    container_memory_working_set_bytes
  )
)
```

Then I compare the result with:

```
Memory Requests

Memory Limits
```

and investigate OOMKilled containers if necessary.

---

# 251. Interview Question: How Do You Optimize a Slow PromQL Query?

### Answer

I would:

```
1. Narrow the metric selector.

2. Avoid unnecessary high-cardinality labels.

3. Reduce the query range where appropriate.

4. Review aggregation.

5. Review vector matching.

6. Avoid expensive regex where unnecessary.

7. Use recording rules for frequently used expensive queries.
```

I would also inspect Prometheus resource utilization and query behavior.

---

# 252. Interview Question: When Would You Use Recording Rules?

### Answer

I use recording rules when a PromQL expression is:

```
Expensive

Frequently queried

Used by many dashboards

Used by alerts

Used in SLO calculations
```

For example:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

can be precomputed as:

```text
service:http_requests_rate5m
```

---

# 253. Interview Question: How Do You Use PromQL During a Production Incident?

### Answer

I start from the symptom and narrow the scope.

For example, if users report 500 errors:

```text
5xx Rate
   ↓
Service
   ↓
Pod
   ↓
Node
   ↓
Dependency
```

I might use:

```promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
```

Then I correlate the affected service with:

```
Grafana

Logs

Kubernetes events

Deployment history

Traces
```

The goal is to use metrics to narrow the investigation rather than randomly checking components.

---

# 254. Interview Question: How Do You Use PromQL for Deployment Validation?

### Answer

After deployment I check:

```
Request Rate

Error Rate

Latency

Pod Restarts

CPU

Memory
```

For example:

```promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
```

If error rate or latency increases significantly after deployment, I would investigate or trigger the rollback strategy.

---

# 255. Interview Question: How Would You Build a Golden Signals Dashboard?

### Answer

I would create four primary panels:

### Traffic

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

### Errors

```promql
sum by (service) (
  rate(http_requests_total{status=~"5.."}[5m])
)
```

### Latency

```promql
histogram_quantile(
  0.95,
  sum by (service, le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

### Saturation

CPU:

```promql
100 * (
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
```

I would then add filters for:

```
Cluster

Namespace

Service

Environment
```

---

# 256. Real-World PromQL Workflow

A DevOps engineer typically works like this:

```text
Incident
   ↓
Check Grafana
   ↓
Open PromQL Query
   ↓
Filter Environment
   ↓
Filter Service
   ↓
Check Traffic
   ↓
Check Errors
   ↓
Check Latency
   ↓
Check Saturation
   ↓
Identify Component
   ↓
Check Logs
   ↓
Check Traces
   ↓
Find Root Cause
```

---

# 257. PromQL Production Mental Model

Remember:

```text
SELECT
    Metric
        ↓
FILTER
    Labels
        ↓
TRANSFORM
    rate / increase / histogram_quantile
        ↓
AGGREGATE
    sum / avg / max / min
        ↓
COMPARE
    > / < / ==
        ↓
VISUALIZE
    Grafana
        ↓
ALERT
    PrometheusRule
```

---

# 258. Final PromQL Cheat Sheet

## Basic Selectors

```promql
up
```

```promql
up{job="prometheus"}
```

```promql
up{environment!="dev"}
```

```promql
up{job=~"node.*"}
```

---

## Range Selectors

```promql
metric[5m]
```

```promql
metric[1h]
```

---

## Counter Functions

```promql
rate(metric_total[5m])
```

```promql
irate(metric_total[5m])
```

```promql
increase(metric_total[1h])
```

---

## Aggregation

```promql
sum(metric)
```

```promql
sum by (service) (metric)
```

```promql
avg(metric)
```

```promql
max(metric)
```

```promql
min(metric)
```

```promql
count(metric)
```

---

## Top/Bottom

```promql
topk(5, metric)
```

```promql
bottomk(5, metric)
```

---

## Time Functions

```promql
avg_over_time(metric[1h])
```

```promql
max_over_time(metric[1h])
```

```promql
min_over_time(metric[1h])
```

```promql
sum_over_time(metric[1h])
```

---

## Histogram

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(metric_bucket[5m])
  )
)
```

---

## Arithmetic

```promql
metric_a + metric_b
```

```promql
metric_a - metric_b
```

```promql
metric_a * 100
```

```promql
metric_a / metric_b
```

---

## Comparison

```promql
metric > 80
```

```promql
metric < 20
```

```promql
metric == 0
```

---

## Vector Matching

```promql
metric_a / on(service) metric_b
```

```promql
metric_a / ignoring(instance) metric_b
```

---

## Recording Rule

```text
service:http_requests_rate5m
```

---

# 259. Final PromQL Production Checklist

Before using a PromQL query in production:

```
[ ] Correct metric

[ ] Correct metric type

[ ] Correct labels

[ ] Correct range

[ ] Correct function

[ ] Correct aggregation

[ ] Correct units

[ ] Correct vector matching

[ ] Cardinality reviewed

[ ] Query performance reviewed

[ ] Tested in Prometheus

[ ] Tested in Grafana

[ ] Alert behavior tested if applicable
```

---

# 260. Final PromQL Mental Model

```text
                 PROMETHEUS TSDB
                       │
                       ↓
                 PromQL Selector
                       │
                       ↓
                 Label Filtering
                       │
                       ↓
              ┌────────┴────────┐
              ↓                 ↓
          Range Data        Instant Data
              │                 │
              ↓                 ↓
       rate / increase      Aggregation
              │                 │
              └────────┬────────┘
                       ↓
                  PromQL Result
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Grafana       Alerts      Analysis
```

The most important production skill is not memorizing every PromQL function.

It is being able to take a monitoring question such as:

```text
"Why are users seeing errors?"
```

and convert it into a sequence of measurable questions:

```text
Are errors increasing?
        ↓
Which service?
        ↓
Which endpoint?
        ↓
Which status code?
        ↓
Which pods?
        ↓
Which version?
        ↓
Which node?
        ↓
Which dependency?
        ↓
What changed?
```

That is where PromQL becomes a real **DevOps troubleshooting tool**, rather than just a query language.
