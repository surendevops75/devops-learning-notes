# Prometheus Fundamentals

Prometheus is an open-source monitoring and alerting system designed primarily for collecting and querying time-series metrics.

It is widely used in:

```
DevOps
Kubernetes
Cloud-Native Applications
Microservices
Infrastructure Monitoring
Application Monitoring
SRE
Production Operations
```

Prometheus helps answer questions such as:

```
Is CPU usage increasing?

Is memory usage high?

How many requests is the application receiving?

What is the HTTP error rate?

What is application latency?

How many Kubernetes pods are running?

Are containers restarting?

Is a node running out of resources?

Is a service unavailable?
```

The fundamental Prometheus model is:

```
Target
   ↓
Metrics Endpoint
   ↓
Prometheus
   ↓
Time-Series Database
   ↓
PromQL
   ↓
Dashboard / Alert
```

---

# 1. What Is Prometheus?

Prometheus is a metrics monitoring and alerting platform.

It collects numerical measurements over time and stores them as time-series data.

For example:

```
CPU Usage
72%

Memory Usage
68%

HTTP Requests
2,500 requests/sec

HTTP Errors
15 requests/sec

Request Latency
250 ms
```

These values change over time.

Prometheus stores these measurements with timestamps and labels.

---

# 2. Why Prometheus Is Used

Modern applications generate a large amount of operational data.

For example:

```
Kubernetes
    ↓
Nodes
    ↓
Pods
    ↓
Containers
    ↓
Applications
    ↓
Databases
    ↓
Load Balancers
```

Prometheus can collect metrics from these components.

It helps engineers:

```
Monitor
Query
Analyze
Alert
Troubleshoot
Capacity Plan
```

---

# 3. Prometheus in a DevOps Environment

A typical DevOps monitoring architecture can be:

```
Application
    ↓
Metrics Endpoint
    ↓
Prometheus
    ↓
PromQL
    ↓
Grafana
```

For alerting:

```
Prometheus
    ↓
Alerting Rules
    ↓
Alertmanager
    ↓
Notification Channel
    ↓
Engineers
```

---

# 4. Prometheus Mental Model

The easiest way to understand Prometheus is:

```
"Prometheus periodically collects metrics from targets and stores them as labeled time-series data."
```

The main concepts are:

```
Metrics

Time Series

Labels

Targets

Scraping

Exporters

PromQL

Recording Rules

Alerting Rules

Alertmanager
```

---

# 5. What Is a Metric?

A metric is a numerical measurement that describes the state or behavior of a system.

Examples:

```
CPU usage

Memory usage

Request count

Request duration

Error count

Active connections

Disk usage

Network traffic
```

---

# 6. Real-World Metric Examples

Application:

```
http_requests_total
```

Infrastructure:

```
node_cpu_seconds_total
```

Kubernetes:

```
kube_pod_info
```

Database:

```
database_connections
```

These metrics can be queried and analyzed.

---

# 7. What Is a Time Series?

A time series is a sequence of metric values recorded over time.

Example:

```
Time        CPU

10:00       40%
10:01       45%
10:02       52%
10:03       68%
10:04       75%
```

Prometheus stores these values as time-series data.

---

# 8. Metric Name

Every Prometheus metric has a metric name.

Example:

```
http_requests_total
```

Another example:

```
node_memory_MemAvailable_bytes
```

The metric name describes what is being measured.

---

# 9. Labels

Labels provide dimensions for a metric.

Example:

```
http_requests_total{
    method="GET",
    status="200",
    service="order"
}
```

The metric name is:

```
http_requests_total
```

Labels provide additional information:

```
method
status
service
```

---

# 10. Why Labels Are Important

Without labels:

```
http_requests_total
```

You only know the total request count.

With labels:

```
http_requests_total{
    service="order",
    status="500"
}
```

you can answer:

```
How many 500 errors occurred in the Order Service?
```

Labels make Prometheus metrics highly queryable.

---

# 11. Time-Series Identity

In Prometheus, a time series is uniquely identified by:

```
Metric Name
+
Label Set
```

For example:

```
http_requests_total{
    service="order",
    status="200"
}
```

is a different time series from:

```
http_requests_total{
    service="payment",
    status="200"
}
```

---

# 12. Label Example

Suppose we have:

```
http_requests_total{
    service="order",
    method="GET",
    status="200"
}
```

and:

```
http_requests_total{
    service="order",
    method="GET",
    status="500"
}
```

These are two different time series.

---

# 13. Labels and Dimensions

Labels can represent dimensions such as:

```
service
environment
namespace
pod
instance
method
status
region
version
```

Example:

```
http_requests_total{
    environment="production",
    service="payment",
    region="ap-south-1"
}
```

---

# 14. Label Cardinality

Cardinality refers to the number of unique combinations of label values.

Low cardinality:

```
method
status
environment
```

Potentially high cardinality:

```
user_id
request_id
transaction_id
session_id
```

High-cardinality labels can create huge numbers of time series.

This can increase:

```
Memory Usage

Storage

Query Cost

Prometheus Resource Consumption
```

---

# 15. High-Cardinality Anti-Pattern

Avoid metrics such as:

```
http_requests_total{
    user_id="123456"
}
```

when there are millions of users.

This can generate an enormous number of time series.

Instead, use metrics with bounded dimensions.

For example:

```
http_requests_total{
    status="200"
}
```

---

# 16. Prometheus Pull Model

Prometheus primarily uses a pull-based monitoring model.

Architecture:

```
Prometheus
    |
    | HTTP GET
    ↓
Target /metrics
    |
    ↓
Metrics Response
```

Prometheus periodically scrapes the target.

---

# 17. Pull vs Push

Pull model:

```
Prometheus
    ↓
Scrape
    ↓
Target
```

Push model:

```
Application
    ↓
Push Metrics
    ↓
Monitoring Server
```

Prometheus primarily follows the pull model.

---

# 18. Why Prometheus Uses Pull

The pull model provides advantages such as:

```
Easy Target Discovery

Health Verification

Centralized Collection

Simple Debugging

Clear Scrape Status
```

If Prometheus cannot scrape a target, that itself becomes useful monitoring information.

---

# 19. Scraping

Scraping means Prometheus periodically requests metrics from a target.

Example:

```
Prometheus
    ↓
GET /metrics
    ↓
Application
```

The application responds with metrics.

---

# 20. Example /metrics Endpoint

A typical metrics endpoint might return:

```
# HELP http_requests_total Total HTTP requests
# TYPE http_requests_total counter
http_requests_total{
    method="GET",
    status="200"
} 15000
```

Prometheus parses this response and stores the metric.

---

# 21. Metrics Endpoint

A target commonly exposes:

```
/metrics
```

Example:

```
http://application:8080/metrics
```

Prometheus scrapes this endpoint.

The exact endpoint depends on the application or exporter.

---

# 22. Prometheus Target

A target is an endpoint from which Prometheus collects metrics.

Examples:

```
Linux Server

Kubernetes Node

Application

Database Exporter

NGINX Exporter

Blackbox Exporter
```

---

# 23. Static Targets

Targets can be explicitly configured.

Example:

```
scrape_configs:
  - job_name: "node"
    static_configs:
      - targets:
          - "server1:9100"
          - "server2:9100"
```

Prometheus then scrapes these endpoints.

---

# 24. Service Discovery

In dynamic environments such as Kubernetes, static targets are difficult to maintain.

Instead, Prometheus can discover targets automatically.

Example:

```
Kubernetes
    ↓
Service Discovery
    ↓
Pods / Services / Nodes
    ↓
Prometheus
```

---

# 25. Kubernetes Monitoring

Prometheus is heavily used with Kubernetes.

It can monitor:

```
Nodes

Pods

Containers

Services

Deployments

StatefulSets

Applications

Kubernetes Control Plane Components
```

---

# 26. Kubernetes Prometheus Architecture

A typical architecture:

```
Kubernetes Cluster
      |
      +-- Application Pods
      |
      +-- Node Exporter
      |
      +-- kube-state-metrics
      |
      +-- Application Metrics
      |
      ↓
  Prometheus
      |
      ↓
   PromQL
      |
      ↓
   Grafana
```

---

# 27. Node Metrics

Node-level metrics can include:

```
CPU

Memory

Disk

Network

Filesystem
```

Node Exporter is commonly used for Linux host metrics.

---

# 28. Application Metrics

Applications can expose metrics directly.

Example:

```
Order Service
    ↓
/metrics
    ↓
Prometheus
```

Application metrics can include:

```
Request Rate

Error Rate

Latency

Active Requests

Queue Size

Business Metrics
```

---

# 29. Exporters

Not every system exposes Prometheus metrics natively.

Exporters convert or expose metrics in Prometheus-compatible form.

Examples:

```
Node Exporter
Blackbox Exporter
MySQL Exporter
PostgreSQL Exporter
NGINX Exporter
```

Exporter architecture:

```
System
   ↓
Exporter
   ↓
/metrics
   ↓
Prometheus
```

---

# 30. Node Exporter

Node Exporter exposes hardware and operating system metrics.

Typical metrics include:

```
CPU

Memory

Disk

Filesystem

Network

Load
```

Example architecture:

```
Linux Server
     ↓
Node Exporter
     ↓
:9100/metrics
     ↓
Prometheus
```

---

# 31. kube-state-metrics

kube-state-metrics exposes Kubernetes object state as metrics.

It can provide information about:

```
Pods

Deployments

ReplicaSets

StatefulSets

Jobs

Nodes

Services
```

It focuses on Kubernetes object state rather than raw host resource metrics.

---

# 32. kube-state-metrics vs Node Exporter

Node Exporter:

```
Infrastructure / OS Metrics
```

Example:

```
CPU
Memory
Disk
```

kube-state-metrics:

```
Kubernetes Object State
```

Example:

```
Desired Replicas
Available Replicas
Pod Phase
Deployment Status
```

Both provide different types of information.

---

# 33. Prometheus Data Model

Prometheus stores data as labeled time series.

Example:

```
http_requests_total{
    job="order-service",
    instance="10.0.1.15:8080",
    method="GET",
    status="200"
}
```

Value:

```
15000
```

Timestamp:

```
2026-08-11T07:00:00
```

---

# 34. Metric Sample

A metric sample consists conceptually of:

```
Metric Name
+
Labels
+
Value
+
Timestamp
```

Example:

```
cpu_usage{
    instance="server01"
} 72
```

---

# 35. Counter

A counter represents a value that generally increases over time.

Examples:

```
HTTP Requests

Errors

Jobs Processed

Transactions
```

Example:

```
http_requests_total
```

Values:

```
100
120
150
200
```

Counters can reset when the application restarts.

---

# 36. Gauge

A gauge represents a value that can increase or decrease.

Examples:

```
CPU Usage

Memory Usage

Active Connections

Queue Size
```

Example:

```
active_connections
```

Values:

```
100
80
120
95
```

---

# 37. Histogram

A histogram measures observations and groups them into buckets.

Common use:

```
Request Latency
```

Example:

```
HTTP request duration
```

It can help answer:

```
How many requests completed under 100ms?

How many under 500ms?

How many under 1 second?
```

---

# 38. Summary

A summary also measures observations and calculates statistical information.

It can provide:

```
Quantiles

Count

Sum
```

However, histograms are often more flexible for aggregating latency across multiple instances.

---

# 39. Four Main Metric Types

The common Prometheus metric types are:

```
Counter

Gauge

Histogram

Summary
```

Each type serves a different monitoring purpose.

---

# 40. Counter Example

Use a counter for:

```
Number of requests
```

Example:

```
http_requests_total
```

Querying a counter directly gives the cumulative value.

For rate:

```
rate(http_requests_total[5m])
```

---

# 41. Gauge Example

Use a gauge for:

```
Current CPU utilization

Current memory usage

Active connections

Queue depth
```

Example:

```
active_connections
```

---

# 42. Histogram Example

Use a histogram for:

```
Request duration
```

Example:

```
http_request_duration_seconds_bucket
```

It creates buckets such as:

```
0.1s
0.5s
1s
2s
5s
```

---

# 43. Summary Example

A summary can expose metrics related to:

```
Quantiles

Count

Sum
```

Example conceptually:

```
http_request_duration_seconds
```

with summary-related series.

---

# 44. Prometheus Server

The Prometheus server is responsible for:

```
Scraping Targets

Storing Metrics

Evaluating Queries

Evaluating Alerting Rules

Evaluating Recording Rules
```

It is the central component of a traditional Prometheus deployment.

---

# 45. Prometheus Server Architecture

```
┌───────────────────────────────┐
│        Prometheus Server      │
│                               │
│  Service Discovery            │
│        ↓                      │
│  Scrape                      │
│        ↓                      │
│  TSDB                        │
│        ↓                      │
│  PromQL                      │
│        ↓                      │
│  Rules                       │
└───────────────┬───────────────┘
                ↓
             Clients
```

---

# 46. Prometheus TSDB

TSDB stands for:

```
Time Series Database
```

Prometheus stores collected metrics in its local time-series database.

The TSDB is optimized for time-series workloads.

---

# 47. Local Storage

Prometheus can store data locally.

Architecture:

```
Targets
   ↓
Prometheus
   ↓
Local TSDB
   ↓
Disk
```

Local storage is simple and useful for many deployments.

---

# 48. Local Storage Limitation

A single Prometheus server has limitations related to:

```
Capacity

Retention

Availability

Query Scale

Failure Recovery
```

For larger environments, additional architecture may be required.

---

# 49. Remote Storage

Prometheus can integrate with remote storage systems.

Conceptually:

```
Prometheus
   |
   +----→ Local TSDB
   |
   +----→ Remote Storage
```

This can provide:

```
Longer Retention

Centralized Storage

Global Querying

Scalability
```

The exact remote-storage architecture depends on the selected platform.

---

# 50. Prometheus and Grafana

Prometheus is primarily a metrics collection and query system.

Grafana is commonly used for visualization.

Architecture:

```
Prometheus
    ↓
PromQL
    ↓
Grafana
    ↓
Dashboard
```

Grafana can display:

```
CPU

Memory

Request Rate

Error Rate

Latency

Kubernetes Metrics
```

---

# 51. Prometheus and Alertmanager

Prometheus can evaluate alerting rules.

Architecture:

```
Prometheus
    ↓
Alerting Rule
    ↓
Alert
    ↓
Alertmanager
    ↓
Notification
```

Alertmanager handles notification management.

---

# 52. Alertmanager

Alertmanager is responsible for handling alerts received from Prometheus.

It can provide capabilities such as:

```
Grouping

Deduplication

Routing

Silencing

Inhibition

Notification
```

---

# 53. Prometheus vs Alertmanager

Prometheus:

```
Collects Metrics

Stores Metrics

Queries Metrics

Evaluates Rules
```

Alertmanager:

```
Receives Alerts

Groups Alerts

Routes Alerts

Sends Notifications
```

---

# 54. Prometheus Scrape Interval

Prometheus periodically scrapes targets.

Example:

```
scrape_interval: 15s
```

This means Prometheus attempts to scrape the target every 15 seconds.

The appropriate interval depends on:

```
Application

Traffic

Metric Importance

Storage

Monitoring Requirements
```

---

# 55. Scrape Timeout

A scrape timeout controls how long Prometheus waits for a target response.

Example concept:

```
Scrape Started
    ↓
Target Response
    ↓
Success
```

or:

```
Scrape Started
    ↓
Timeout
    ↓
Failure
```

Timeout should be configured appropriately.

---

# 56. Scrape Success

Prometheus exposes information about scrape health.

A key metric is:

```
up
```

Example:

```
up{job="order-service"} 1
```

This indicates the target was successfully scraped.

---

# 57. Scrape Failure

Example:

```
up{job="order-service"} 0
```

This indicates that the target could not be successfully scraped.

Possible causes:

```
Application Down

Network Failure

Wrong Port

Wrong Endpoint

DNS Failure

TLS Problem

Timeout
```

---

# 58. Why `up` Is Important

The `up` metric is useful for availability monitoring.

Example:

```
up == 0
```

can indicate that a target is unreachable.

However, target reachability does not always mean that the application itself is healthy.

Application-specific health metrics should also be monitored.

---

# 59. Jobs

Prometheus groups scrape configurations into jobs.

Example:

```
job_name: "order-service"
```

A job represents a logical group of scrape targets.

Example:

```
order-service
    ↓
instance 1
instance 2
instance 3
```

---

# 60. Instance

An instance typically represents one scrape target.

Example:

```
instance="10.0.1.20:8080"
```

For a service with multiple replicas:

```
order-service
   |
   +-- 10.0.1.20:8080
   +-- 10.0.1.21:8080
   +-- 10.0.1.22:8080
```

Each can become a separate time series based on labels.

---

# 61. Job and Instance

Example metric:

```
http_requests_total{
    job="order-service",
    instance="10.0.1.20:8080"
}
```

Here:

```
job
    ↓
Logical service group

instance
    ↓
Individual target
```

---

# 62. Target Labels

Prometheus attaches labels to scraped metrics.

Common labels include:

```
job

instance
```

Other labels can come from:

```
Service Discovery

Relabeling

Exporters

Application Metrics
```

---

# 63. Relabeling

Relabeling allows Prometheus to modify target labels before scraping or modify metric labels during ingestion.

It can be used to:

```
Rename Labels

Drop Targets

Keep Targets

Modify Addresses

Control Scraping
```

Relabeling becomes very important in Kubernetes monitoring.

---

# 64. Target Relabeling

Target relabeling happens before scraping.

Conceptually:

```
Discovered Target
     ↓
Relabeling
     ↓
Final Target
     ↓
Scrape
```

It can determine which discovered targets should actually be scraped.

---

# 65. Metric Relabeling

Metric relabeling occurs after metrics are scraped.

Conceptually:

```
Target
   ↓
Scrape
   ↓
Metrics
   ↓
Metric Relabeling
   ↓
Storage
```

It can be used to drop unwanted metrics.

---

# 66. Why Metric Filtering Matters

Suppose an exporter exposes thousands of metrics.

You may not need all of them.

Filtering can reduce:

```
Storage

Memory

Query Complexity

Prometheus Load
```

Use filtering carefully so important metrics are not accidentally removed.

---

# 67. Prometheus Labels in Kubernetes

A Kubernetes metric may contain labels such as:

```
namespace

pod

container

node

service

deployment
```

Example:

```
container_cpu_usage_seconds_total{
    namespace="production",
    pod="order-7d8f9",
    container="order"
}
```

---

# 68. Prometheus in Microservices

Suppose the platform contains:

```
User Service

Product Service

Cart Service

Order Service

Payment Service

Inventory Service

Notification Service
```

Prometheus can collect metrics from each service.

---

# 69. Microservices Monitoring Architecture

```
User Service
      |
Product Service
      |
Cart Service
      |
Order Service
      |
Payment Service
      |
Inventory Service
      |
Notification Service
      |
      ↓
  Prometheus
      ↓
   PromQL
      ↓
   Grafana
```

---

# 70. RED Method

For request-driven services, a useful monitoring approach is RED:

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

---

# 71. RED Example

Order Service:

```
Rate:
1,500 requests/sec

Errors:
20 errors/sec

Duration:
p95 = 450 ms
```

This provides a quick view of application health.

---

# 72. USE Method

For infrastructure resources, USE is useful:

```
Utilization

Saturation

Errors
```

Example:

```
CPU Utilization

Memory Saturation

Disk Errors
```

This complements application-level monitoring.

---

# 73. Golden Signals

The four common Golden Signals are:

```
Latency

Traffic

Errors

Saturation
```

Prometheus can collect metrics required to monitor these signals.

---

# 74. Latency

Latency measures how long operations take.

Example:

```
HTTP Request
    ↓
250 ms
```

Important percentiles:

```
p50

p90

p95

p99
```

---

# 75. Traffic

Traffic measures demand.

Examples:

```
Requests/sec

Transactions/sec

Messages/sec
```

Example:

```
http_requests_total
```

can be converted into request rate using PromQL.

---

# 76. Errors

Errors represent failed operations.

Examples:

```
HTTP 5xx

HTTP 4xx

Failed Jobs

Database Errors

Application Exceptions
```

---

# 77. Saturation

Saturation indicates how close a resource is to its capacity.

Examples:

```
CPU

Memory

Disk

Network

Connection Pool

Queue
```

---

# 78. Infrastructure Monitoring

Prometheus can monitor:

```
Linux

Kubernetes

Containers

Databases

Network Services

Cloud Components
```

through appropriate exporters and integrations.

---

# 79. Application Monitoring

Prometheus can monitor:

```
Request Rate

Error Rate

Latency

JVM Metrics

Node.js Metrics

Python Metrics

Queue Depth

Cache Performance
```

---

# 80. Business Metrics

Prometheus can also monitor selected business-level metrics.

Examples:

```
Orders Processed

Payments Completed

Failed Payments

Inventory Reservations

Checkout Attempts
```

Example:

```
orders_processed_total
```

Business metrics should be designed with controlled cardinality.

---

# 81. Technical vs Business Metrics

Technical metric:

```
CPU Usage
```

Business metric:

```
orders_processed_total
```

Technical metric:

```
HTTP 500 Rate
```

Business metric:

```
payment_failures_total
```

Both can help understand production behavior.

---

# 82. Prometheus Querying

Prometheus provides:

```
PromQL
```

PromQL stands for:

```
Prometheus Query Language
```

It allows engineers to:

```
Select Time Series

Filter Labels

Aggregate Data

Calculate Rates

Calculate Percentiles

Compare Metrics

Build Alerts
```

---

# 83. Simple PromQL Example

Query:

```
up
```

This returns the current `up` metric for available targets.

---

# 84. Metric Selector

Query:

```
http_requests_total
```

returns matching time series for that metric.

---

# 85. Label Filtering

Query:

```
http_requests_total{
    status="500"
}
```

This selects HTTP request metrics where:

```
status = 500
```

---

# 86. Multiple Label Filters

Example:

```
http_requests_total{
    service="order",
    status="500"
}
```

This selects 500-status requests for the Order Service.

---

# 87. Rate Function

For a counter:

```
rate(http_requests_total[5m])
```

This calculates the per-second average rate of increase over the last five minutes.

---

# 88. Increase Function

Example:

```
increase(http_requests_total[1h])
```

This estimates how much the counter increased over the selected time range.

Useful for questions such as:

```
How many requests occurred during the last hour?
```

---

# 89. Aggregation

Suppose multiple service instances exist.

Example:

```
http_requests_total{
    instance="1"
}

http_requests_total{
    instance="2"
}
```

You may want total traffic.

Conceptually:

```
sum(rate(http_requests_total[5m]))
```

---

# 90. Grouping

You can aggregate by labels.

Example:

```
sum by (service) (
  rate(http_requests_total[5m])
)
```

This gives request rate grouped by service.

---

# 91. Error Rate

Example:

```
sum(rate(http_requests_total{
    status=~"5.."
}[5m]))
```

This calculates the rate of HTTP 5xx responses.

---

# 92. Error Percentage

Conceptually:

```
Error Rate
/
Total Request Rate
× 100
```

PromQL can implement this calculation.

Example:

```
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) * 100
```

---

# 93. Availability

A simple target availability signal can use:

```
up
```

For example:

```
up == 0
```

can identify unreachable scrape targets.

Application availability should use additional application-specific metrics where necessary.

---

# 94. CPU Monitoring

Example concept:

```
CPU Usage
```

Prometheus can collect CPU metrics through exporters or Kubernetes monitoring components.

The exact PromQL depends on the metric source.

---

# 95. Memory Monitoring

Memory metrics can show:

```
Used Memory

Available Memory

Working Set

Cache

Container Memory
```

This is especially important in Kubernetes.

---

# 96. Disk Monitoring

Monitor:

```
Disk Usage

Filesystem Usage

Disk I/O

Disk Errors
```

Example alert:

```
Filesystem usage > threshold
```

---

# 97. Kubernetes Pod Monitoring

Important signals include:

```
Pod Running

Pod Pending

Pod Failed

Container Restarts

CPU Usage

Memory Usage

OOMKilled
```

---

# 98. Container Restart Monitoring

A high restart count can indicate:

```
CrashLoopBackOff

Application Failure

OOMKilled

Probe Failure

Configuration Error
```

Prometheus can alert on abnormal restart behavior.

---

# 99. Kubernetes Deployment Monitoring

Important metrics include:

```
Desired Replicas

Available Replicas

Ready Replicas

Updated Replicas

Unavailable Replicas
```

Example:

```
Desired:
5

Available:
3
```

This may indicate a deployment problem.

---

# 100. Node Monitoring

Monitor:

```
CPU

Memory

Disk

Network

Filesystem

Load
```

Node-level issues can affect many workloads simultaneously.

---

# 101. Prometheus Alerting

Prometheus can evaluate alerting rules.

Example:

```
CPU > 80%
```

Then:

```
Alert
```

The alert can be sent to Alertmanager.

---

# 102. Example Alert Flow

```
Prometheus
    ↓
Evaluate Rule
    ↓
Condition True
    ↓
Alert Firing
    ↓
Alertmanager
    ↓
Notification
```

---

# 103. Alerting Rule Example

Conceptually:

```
alert: HighCPUUsage

expression:
cpu_usage > 80

for:
5m
```

This means the condition must remain true for the specified period before the alert fires.

---

# 104. Why Use `for`

Without `for`:

```
CPU temporarily spikes
    ↓
Alert immediately
```

With:

```
for: 5m
```

the condition must remain true for five minutes.

This helps reduce alerts caused by short-lived spikes.

---

# 105. Alert Severity

Alerts can contain labels such as:

```
severity="warning"
```

or:

```
severity="critical"
```

Example:

```
severity="critical"
```

Alertmanager can route alerts based on these labels.

---

# 106. Alert Annotations

Annotations can provide useful information.

Examples:

```
summary

description

runbook_url
```

A good alert should tell the engineer:

```
What happened?

Where?

How severe?

What should I check?
```

---

# 107. Recording Rules

Recording rules precompute frequently used PromQL expressions.

Example:

```
sum(rate(http_requests_total[5m]))
```

can be stored as a new metric.

Benefits:

```
Faster Dashboards

Faster Queries

Reduced Query CPU

Reusable Expressions
```

---

# 108. Why Recording Rules Matter

Suppose Grafana repeatedly executes an expensive query.

Instead of recalculating it every time:

```
Complex Query
    ↓
Recording Rule
    ↓
Precomputed Metric
    ↓
Grafana
```

This can improve performance.

---

# 109. Prometheus and Grafana Dashboards

A production dashboard might include:

```
CPU

Memory

Request Rate

Error Rate

Latency

Pod Restarts

Node Health

Disk Usage

Application Health
```

---

# 110. Service Dashboard

For a microservice:

```
Service: Order

Request Rate:
1,200 req/s

Error Rate:
0.4%

p95 Latency:
320 ms

Active Pods:
5

Restarts:
0
```

This provides a service-level operational view.

---

# 111. Infrastructure Dashboard

For Kubernetes:

```
Nodes:
10

CPU:
62%

Memory:
71%

Disk:
58%

Pods:
180

Restarts:
3
```

---

# 112. Prometheus Use in Production

A production monitoring stack can look like:

```
AWS
  ↓
EKS
  ↓
Applications
  ↓
Prometheus
  ↓
PromQL
  ↓
Grafana
```

Alerting:

```
Prometheus
  ↓
Alertmanager
  ↓
Notification Channel
```

---

# 113. Real-World Microservices Example

Suppose an application has:

```
User
Product
Cart
Order
Payment
Inventory
Notification
```

Prometheus monitors:

```
Request Rate

Error Rate

Latency

CPU

Memory

Pod Restarts

Deployment Availability
```

---

# 114. Example Production Incident

Problem:

```
Users report slow checkout.
```

Monitoring shows:

```
Order p95 latency ↑
```

Prometheus dashboard:

```
Order:
p95 = 2.5 seconds
```

Further investigation:

```
Payment latency ↑
```

Then:

```
Payment → External Provider
```

is slow.

Prometheus helps identify the service-level symptoms.

Distributed tracing can then identify the exact request-level dependency.

---

# 115. Metrics and Traces Together

Prometheus answers:

```
"Is the service slow?"
```

Tracing answers:

```
"Which request and dependency caused the slowness?"
```

Example:

```
Prometheus
    ↓
Payment p95 latency ↑
    ↓
Jaeger
    ↓
External API span = 2.1 seconds
```

This is a powerful observability workflow.

---

# 116. Metrics and Logs Together

Prometheus:

```
Error Rate ↑
```

ELK:

```
Search payment-service errors
```

Logs:

```
Connection timeout
```

Then tracing:

```
External API span slow
```

Each signal provides a different level of detail.

---

# 117. Three Pillars in the Stack

Metrics:

```
Prometheus
```

Logs:

```
ELK
```

Traces:

```
OpenTelemetry + Jaeger
```

Visualization:

```
Grafana
```

This creates a complete observability platform.

---

# 118. Prometheus Strengths

Prometheus is strong for:

```
Time-Series Metrics

Kubernetes Monitoring

Cloud-Native Applications

PromQL

Alerting Rules

Service Discovery

Exporter Ecosystem

Infrastructure Monitoring
```

---

# 119. Prometheus Limitations

Prometheus is primarily focused on metrics.

It is not a replacement for:

```
Log Management

Distributed Tracing

Full Application Profiling
```

Therefore:

```
Prometheus
    +
ELK
    +
OpenTelemetry / Jaeger
```

can provide broader observability.

---

# 120. Prometheus vs Logging

Prometheus:

```
Numerical Time-Series Data
```

Example:

```
CPU = 75%
```

Logs:

```
Text / Structured Events
```

Example:

```
Payment timeout connecting to provider
```

Use the appropriate signal for the question.

---

# 121. Prometheus vs Distributed Tracing

Prometheus:

```
Aggregated Metrics
```

Example:

```
p95 latency = 400 ms
```

Tracing:

```
Individual Request Path
```

Example:

```
Order
  ↓
Payment
  ↓
External API
```

Tracing explains the request path.

---

# 122. Prometheus vs Grafana

Prometheus:

```
Collection

Storage

Query

Alert Rules
```

Grafana:

```
Visualization

Dashboards

Data Exploration
```

They work together but serve different roles.

---

# 123. Prometheus vs Alertmanager

Prometheus:

```
Detects alert conditions.
```

Alertmanager:

```
Handles alert delivery and routing.
```

Example:

```
Prometheus
   ↓
High CPU Alert
   ↓
Alertmanager
   ↓
Notification
```

---

# 124. Pull Model Troubleshooting

If a metric is missing:

```
Check Target

   ↓

Check /metrics

   ↓

Check Network

   ↓

Check Prometheus Target Status

   ↓

Check Scrape Errors

   ↓

Check Relabeling

   ↓

Check Metric Filtering

   ↓

Check PromQL
```

---

# 125. Target Troubleshooting

Check whether the target exists.

Then verify:

```
DNS

IP

Port

Endpoint

Network Policy

Security Group

Service

Pod
```

---

# 126. /metrics Troubleshooting

Test manually:

```
curl http://target:port/metrics
```

Check whether metrics are returned.

If the endpoint does not respond:

```
Application / Exporter problem
```

If it responds:

```
Check Prometheus configuration.
```

---

# 127. Prometheus Target Status

Prometheus provides target information.

Important fields:

```
UP

DOWN

Last Scrape

Scrape Duration

Error
```

If a target is DOWN, investigate the scrape error.

---

# 128. Common Scrape Problems

Common causes:

```
Wrong Port

Wrong Endpoint

DNS Failure

Network Policy

Firewall

TLS Configuration

Authentication

Timeout

Application Down
```

---

# 129. Missing Metric Troubleshooting

If:

```
Target = UP
```

but:

```
Metric Missing
```

check:

```
/metrics

Metric Name

Labels

Relabeling

Metric Relabeling

Exporter Configuration

Application Instrumentation
```

---

# 130. High Prometheus Memory Usage

Possible causes:

```
High Cardinality

Too Many Targets

Too Many Metrics

Long Retention

Large Query Load

Excessive Labels
```

Investigation should begin with time-series growth and cardinality.

---

# 131. High Prometheus CPU Usage

Possible causes:

```
Expensive PromQL

High Scrape Rate

Large Number of Series

Complex Recording Rules

Frequent Queries

Large Dashboards
```

Optimize queries and reduce unnecessary metric volume.

---

# 132. High Cardinality Troubleshooting

Check labels.

Bad example:

```
user_id

request_id

session_id
```

Potentially dangerous because each unique value creates additional series.

Use bounded labels wherever possible.

---

# 133. Prometheus Storage Growth

Storage can grow because of:

```
More Targets

More Metrics

More Labels

Higher Scrape Frequency

Longer Retention

Increased Application Traffic
```

Monitor series count and storage usage.

---

# 134. Scrape Interval Trade-Off

Short interval:

```
5 seconds
```

Advantages:

```
Faster Detection
```

Disadvantages:

```
More Scrapes

More Storage

More CPU
```

Long interval:

```
60 seconds
```

Advantages:

```
Lower Resource Usage
```

Disadvantages:

```
Slower Detection
```

Choose based on monitoring requirements.

---

# 135. Metric Design Best Practices

Good metric design should provide:

```
Clear Names

Useful Labels

Controlled Cardinality

Correct Metric Type

Meaningful Units

Consistent Naming
```

---

# 136. Metric Naming

Prefer descriptive names.

Example:

```
http_requests_total

process_cpu_seconds_total

http_request_duration_seconds
```

Metric names should clearly communicate what is measured.

---

# 137. Units

Use consistent units.

Examples:

```
seconds

bytes

ratios
```

Avoid ambiguous units.

For example:

```
request_duration_seconds
```

is clearer than:

```
request_duration
```

---

# 138. Counters and `_total`

Counters commonly use:

```
_total
```

Example:

```
http_requests_total
```

This communicates that the value is cumulative.

---

# 139. Histogram Naming

A histogram commonly produces related series.

Example:

```
http_request_duration_seconds_bucket

http_request_duration_seconds_sum

http_request_duration_seconds_count
```

These allow latency analysis.

---

# 140. Labels Best Practices

Good labels:

```
service

method

status

environment

namespace
```

Potentially dangerous:

```
user_id

request_id

session_id
```

unless carefully controlled and justified.

---

# 141. Avoid Metric Explosion

Do not expose thousands of unnecessary metrics without a reason.

Metric volume should be reviewed regularly.

Monitor:

```
Number of Series

Number of Metrics

Cardinality

Storage Growth
```

---

# 142. Production Naming Convention

A consistent naming approach can make dashboards easier to maintain.

Example:

```
http_requests_total

http_request_duration_seconds

database_connections_active

queue_messages_pending
```

Names should clearly represent the measured behavior.

---

# 143. Prometheus Security

Prometheus should not be exposed publicly without appropriate protection.

Consider:

```
Network Controls

Authentication

Authorization

TLS

Kubernetes NetworkPolicies

Security Groups
```

Prometheus contains operational information about the infrastructure.

---

# 144. Prometheus in AWS

A production AWS architecture may look like:

```
Internet
   ↓
ALB
   ↓
EKS
   ↓
Applications
   ↓
Prometheus
   ↓
Grafana
```

Infrastructure may include:

```
EC2

EKS

RDS

Load Balancers

S3

Network Components
```

Appropriate exporters and integrations can expose metrics.

---

# 145. Prometheus in EKS

A typical EKS monitoring environment includes:

```
Prometheus

Node Exporter

kube-state-metrics

Application Metrics

Grafana

Alertmanager
```

Architecture:

```
EKS
  |
  +-- Nodes
  |
  +-- Pods
  |
  +-- Services
  |
  +-- Applications
  |
  ↓
Prometheus
  |
  +-- Grafana
  |
  +-- Alertmanager
```

---

# 146. Production Monitoring Layers

A mature monitoring platform monitors several layers.

Infrastructure:

```
Nodes
CPU
Memory
Disk
Network
```

Kubernetes:

```
Pods
Deployments
Nodes
Containers
```

Application:

```
Rate
Errors
Latency
```

Business:

```
Orders
Payments
Transactions
```

---

# 147. Layered Monitoring Architecture

```
Business
    ↓
Application
    ↓
Kubernetes
    ↓
Infrastructure
    ↓
Cloud
```

Prometheus can collect metrics across multiple layers when appropriate exporters and instrumentation are available.

---

# 148. Monitoring Strategy

Do not monitor only CPU and memory.

A production monitoring strategy should cover:

```
Availability

Performance

Reliability

Capacity

Errors

Saturation

Dependencies

Business-Critical Operations
```

---

# 149. SLO Monitoring

Prometheus can be used to calculate SLI/SLO-related metrics.

Example:

```
Availability SLI

Error Rate

Latency
```

Example objective:

```
99.9% successful requests
```

PromQL can calculate the relevant measurement.

---

# 150. Error Budget

An SLO creates an error budget.

For example:

```
SLO:
99.9%
```

Allowed failure:

```
0.1%
```

Prometheus can calculate service behavior against the objective.

---

# 151. Capacity Planning

Historical Prometheus metrics can help answer:

```
Is CPU usage growing?

Is memory usage increasing?

Are requests increasing?

When will capacity become insufficient?
```

This supports infrastructure planning.

---

# 152. Scaling Decisions

Prometheus metrics can feed scaling systems.

Example:

```
CPU ↑
   ↓
HPA
   ↓
More Pods
```

Application metric:

```
Requests/sec ↑
   ↓
HPA
   ↓
Scale Pods
```

Prometheus is commonly used as a source of monitoring metrics for Kubernetes autoscaling architectures.

---

# 153. Prometheus and HPA

An application may scale based on:

```
CPU

Memory
```

or custom metrics.

Example:

```
Requests per second
    ↑
HPA
    ↓
Pod Replicas ↑
```

The exact implementation depends on the Kubernetes metrics architecture.

---

# 154. Prometheus and Incident Response

During an incident:

```
Alert
   ↓
Prometheus
   ↓
Dashboard
   ↓
Identify Service
   ↓
Logs
   ↓
Trace
   ↓
Root Cause
```

Prometheus provides the initial quantitative view.

---

# 155. Prometheus Production Checklist

## Collection

```
[ ] Targets configured

[ ] Service discovery configured

[ ] Scrape intervals reviewed

[ ] Scrape timeouts reviewed

[ ] Exporters configured

[ ] Application instrumentation configured
```

---

## Metrics

```
[ ] Metric names are clear

[ ] Metric types are correct

[ ] Units are consistent

[ ] Labels are meaningful

[ ] Cardinality is controlled
```

---

## Kubernetes

```
[ ] Node monitoring

[ ] Pod monitoring

[ ] Container monitoring

[ ] Deployment monitoring

[ ] kube-state-metrics

[ ] Application metrics
```

---

## Querying

```
[ ] PromQL understood

[ ] Rate queries

[ ] Aggregation

[ ] Error rate

[ ] Latency

[ ] Saturation
```

---

## Alerting

```
[ ] Alert rules

[ ] Severity

[ ] Alertmanager

[ ] Routing

[ ] Notification

[ ] Runbooks
```

---

## Production

```
[ ] Storage capacity

[ ] Retention

[ ] Resource limits

[ ] High availability strategy

[ ] Backup / recovery strategy where applicable

[ ] Security

[ ] Monitoring of Prometheus itself
```

---

# 156. Interview Question: What Is Prometheus?

### Answer

Prometheus is an open-source monitoring and alerting system designed primarily for time-series metrics.

It collects metrics from targets, stores them as labeled time-series data, provides PromQL for querying, and supports alerting rules.

A typical architecture is:

```
Targets
   ↓
Prometheus
   ↓
PromQL
   ↓
Grafana
```

For alerting:

```
Prometheus
   ↓
Alertmanager
```

---

# 157. Interview Question: Is Prometheus Pull or Push Based?

### Answer

Prometheus primarily uses a pull-based model.

Prometheus periodically scrapes metrics from targets, usually through an HTTP metrics endpoint.

For example:

```
Prometheus
   ↓
GET /metrics
   ↓
Application
```

This allows Prometheus to centrally control scraping and identify scrape failures.

---

# 158. Interview Question: What Is a Prometheus Target?

### Answer

A target is an endpoint from which Prometheus collects metrics.

Examples:

```
Application
Node Exporter
Database Exporter
Kubernetes Component
```

A target normally exposes metrics through a Prometheus-compatible endpoint.

---

# 159. Interview Question: What Is a Prometheus Job?

### Answer

A job is a logical group of scrape targets.

For example:

```
job="order-service"
```

may contain:

```
order-pod-1
order-pod-2
order-pod-3
```

The `job` label identifies the logical monitoring group.

---

# 160. Interview Question: What Is an Instance?

### Answer

An instance generally identifies an individual scrape target.

For example:

```
instance="10.0.1.10:8080"
```

If a service has three replicas, each replica can have its own instance label.

---

# 161. Interview Question: What Are Labels?

### Answer

Labels are key-value dimensions attached to metrics.

Example:

```
http_requests_total{
    service="order",
    status="500"
}
```

Labels allow us to filter and aggregate time series.

However, labels should be designed carefully to avoid high cardinality.

---

# 162. Interview Question: What Is High Cardinality?

### Answer

High cardinality occurs when a metric produces a very large number of unique label combinations.

For example:

```
user_id

request_id

session_id
```

can create huge numbers of time series.

This increases Prometheus memory, storage, and query load.

I prefer bounded labels such as:

```
service

method

status

environment
```

---

# 163. Interview Question: What Are the Four Prometheus Metric Types?

### Answer

The common Prometheus metric types are:

```
Counter
Gauge
Histogram
Summary
```

Counter is useful for cumulative values.

Gauge is useful for values that increase or decrease.

Histogram is useful for distributions such as latency.

Summary provides observations and quantile-related information.

---

# 164. Interview Question: Counter vs Gauge?

### Answer

A counter generally increases over time and may reset when the process restarts.

Example:

```
http_requests_total
```

A gauge can increase or decrease.

Examples:

```
active_connections

memory_usage

queue_size
```

---

# 165. Interview Question: What Is a Histogram?

### Answer

A histogram records observations into configurable buckets.

It is commonly used for:

```
Request Latency

Response Size

Processing Duration
```

For example:

```
<100ms
<500ms
<1s
<2s
```

Histograms are useful for calculating latency distributions and aggregating observations across instances.

---

# 166. Interview Question: What Is PromQL?

### Answer

PromQL is Prometheus Query Language.

It allows engineers to:

```
Select Metrics

Filter Labels

Aggregate Time Series

Calculate Rates

Calculate Ratios

Analyze Histograms

Create Recording Rules

Create Alerting Rules
```

---

# 167. Interview Question: What Is the `up` Metric?

### Answer

`up` is a metric that indicates whether Prometheus successfully scraped a target.

Typically:

```
up = 1
```

means the scrape succeeded.

```
up = 0
```

means the scrape failed.

However, `up=1` does not necessarily mean the application is functionally healthy, so application-level health metrics should also be monitored.

---

# 168. Interview Question: What Is an Exporter?

### Answer

An exporter exposes metrics from a system in a format Prometheus can scrape.

For example:

```
Linux
   ↓
Node Exporter
   ↓
/metrics
   ↓
Prometheus
```

Common exporters include:

```
Node Exporter

Database Exporters

Blackbox Exporter

NGINX Exporter
```

---

# 169. Interview Question: What Is Node Exporter?

### Answer

Node Exporter exposes Linux host-level metrics such as:

```
CPU

Memory

Disk

Filesystem

Network
```

Prometheus scrapes Node Exporter's metrics endpoint.

---

# 170. Interview Question: What Is kube-state-metrics?

### Answer

kube-state-metrics exposes metrics based on the state of Kubernetes objects.

It can provide information about:

```
Pods

Deployments

ReplicaSets

StatefulSets

Jobs

Nodes
```

It is different from Node Exporter because it focuses on Kubernetes object state rather than primarily operating-system metrics.

---

# 171. Interview Question: How Does Prometheus Monitor Kubernetes?

### Answer

A typical Kubernetes monitoring setup includes:

```
Prometheus

Service Discovery

Node Exporter

kube-state-metrics

Application Metrics
```

Prometheus discovers Kubernetes targets and scrapes their metrics.

Architecture:

```
Kubernetes
    ↓
Service Discovery
    ↓
Prometheus
    ↓
PromQL
    ↓
Grafana
```

---

# 172. Interview Question: How Do You Monitor Application Latency?

### Answer

I would expose application request-duration metrics, preferably using histograms when I need distribution analysis.

For example:

```
http_request_duration_seconds
```

Then I would use PromQL to calculate:

```
p50

p95

p99
```

I would dashboard these values in Grafana and create alerts based on service-level objectives or operational thresholds.

---

# 173. Interview Question: How Do You Monitor HTTP Error Rate?

### Answer

I would expose a request counter with an HTTP status label.

For example:

```
http_requests_total{
    status="500"
}
```

Then calculate the 5xx rate using:

```
rate()
```

and aggregate across instances.

Conceptually:

```
5xx Rate
/
Total Request Rate
```

This gives the error percentage.

---

# 174. Interview Question: Prometheus vs Grafana?

### Answer

Prometheus is responsible for:

```
Collecting Metrics

Storing Metrics

Querying Metrics

Evaluating Rules
```

Grafana is primarily responsible for:

```
Visualization

Dashboards

Exploration
```

A common architecture is:

```
Prometheus
    ↓
PromQL
    ↓
Grafana
```

---

# 175. Interview Question: Prometheus vs Alertmanager?

### Answer

Prometheus evaluates alerting rules.

Alertmanager manages alerts after they are generated.

Prometheus:

```
Detect
```

Alertmanager:

```
Group
Deduplicate
Route
Notify
```

---

# 176. Interview Question: How Would You Troubleshoot a Missing Prometheus Metric?

### Answer

I would troubleshoot in this order:

```
1. Check whether the target exists.

2. Check target status.

3. Check /metrics manually.

4. Verify network connectivity.

5. Verify scrape configuration.

6. Check service discovery.

7. Check relabeling.

8. Check metric relabeling.

9. Verify metric name.

10. Verify labels.

11. Check the PromQL query.
```

This helps determine whether the problem is collection, filtering, or querying.

---

# 177. Interview Question: Prometheus Memory Usage Is Very High. What Would You Check?

### Answer

I would check:

```
High Cardinality

Number of Active Time Series

Number of Targets

Number of Metrics

Scrape Frequency

Retention

Query Load

Recording Rules
```

I would especially look for unbounded labels such as:

```
user_id

request_id

session_id
```

Then I would reduce unnecessary cardinality and optimize collection and queries.

---

# 178. Interview Question: Prometheus CPU Usage Is High. How Would You Troubleshoot?

### Answer

I would check:

```
Expensive PromQL Queries

Number of Time Series

Scrape Frequency

Recording Rules

Dashboard Queries

Rule Evaluation

Target Count
```

I would identify expensive queries and move frequently used expensive calculations into recording rules where appropriate.

---

# 179. Interview Question: How Would You Monitor a Production EKS Cluster?

### Answer

I would monitor multiple layers.

Infrastructure:

```
Node CPU

Node Memory

Disk

Network
```

Kubernetes:

```
Pod State

Container Restarts

Deployment Availability

Node Health
```

Application:

```
Request Rate

Error Rate

Latency
```

Dependencies:

```
Database

Redis

RabbitMQ

External APIs
```

I would use Prometheus for metrics, Grafana for dashboards, and Alertmanager for alert routing.

---

# 180. Interview Question: How Would You Monitor a Microservices Platform?

### Answer

I would standardize application metrics across services.

For every service I would monitor:

```
Request Rate

Error Rate

Latency

Saturation

Resource Usage
```

I would also monitor:

```
Pod Availability

Restarts

Dependencies

Database Connections

Queue Depth
```

Then I would build service-level dashboards and alerts in Grafana using Prometheus data.

---

# 181. Interview Question: How Would You Design Prometheus for Production?

### Answer

I would first understand:

```
Number of Targets

Scrape Frequency

Metric Cardinality

Retention

Query Load
```

Then I would configure:

```
Service Discovery

Appropriate Scrape Intervals

Resource Requests and Limits

Recording Rules

Alerting Rules

Alertmanager

Storage
```

For larger environments, I would also evaluate:

```
High Availability

Remote Storage

Long-Term Retention

Global Querying
```

---

# 182. Prometheus Production Architecture Mental Model

Remember:

```
TARGETS
   ↓
SERVICE DISCOVERY
   ↓
SCRAPING
   ↓
PROMETHEUS
   ↓
TSDB
   ↓
PROMQL
   ↓
┌───────────────┐
│               │
↓               ↓
GRAFANA      ALERTING RULES
                ↓
           ALERTMANAGER
                ↓
            NOTIFICATION
```

---

# 183. Prometheus + Kubernetes Mental Model

Remember:

```
EKS
 |
 +-- Nodes
 |     ↓
 |   Node Exporter
 |
 +-- Kubernetes Objects
 |     ↓
 |   kube-state-metrics
 |
 +-- Applications
       ↓
   /metrics
       |
       ↓
   Prometheus
       ↓
    PromQL
       ↓
    Grafana
```

---

# 184. Prometheus + Observability Mental Model

Remember:

```
METRICS
   ↓
PROMETHEUS
   ↓
DETECT PROBLEM
   ↓
LOGS
   ↓
ELK
   ↓
TRACE ID
   ↓
OPENTELEMETRY
   ↓
JAEGER
   ↓
ROOT CAUSE
```

Prometheus is often the first signal that something is wrong.

---

# 185. Final Prometheus Fundamentals Summary

Prometheus is a time-series monitoring and alerting system.

Its core concepts are:

```
Metrics

Time Series

Labels

Targets

Scraping

Service Discovery

Exporters

PromQL

Recording Rules

Alerting Rules

Alertmanager
```

The standard operational flow is:

```
Target
   ↓
/metrics
   ↓
Prometheus
   ↓
TSDB
   ↓
PromQL
   ↓
Grafana
```

Alerting flow:

```
Prometheus
   ↓
Alerting Rule
   ↓
Alertmanager
   ↓
Notification
```

Kubernetes flow:

```
EKS
   ↓
Service Discovery
   ↓
Applications / Exporters
   ↓
Prometheus
   ↓
Grafana
```

Production observability flow:

```
Metrics
   ↓
Prometheus
   ↓
Detect Anomaly
   ↓
Logs
   ↓
ELK
   ↓
Trace
   ↓
OpenTelemetry + Jaeger
   ↓
Root Cause
```

The most important Prometheus principles to remember are:

```
Prometheus primarily uses a pull model.

Metrics are stored as labeled time series.

Labels provide dimensions for querying.

High-cardinality labels can become expensive.

Counters are used for cumulative values.

Gauges represent values that can increase or decrease.

Histograms are useful for distributions such as latency.

PromQL is used to query and analyze metrics.

Exporters expose metrics from systems that do not natively expose Prometheus metrics.

Service discovery is essential for dynamic environments such as Kubernetes.

Alertmanager handles alert routing and notification.

Grafana provides visualization.

Prometheus provides the metrics foundation for the monitoring layer.
```

A strong production monitoring architecture therefore looks like:

```
┌─────────────────────────────────────────────────────────┐
│                         EKS                             │
│                                                         │
│  Applications ──────── /metrics                         │
│       │                                                 │
│  Nodes ────────────── Node Exporter                     │
│       │                                                 │
│  Kubernetes Objects ── kube-state-metrics               │
│       │                                                 │
└───────┼─────────────────────────────────────────────────┘
        │
        ↓
Service Discovery
        │
        ↓
   Prometheus
        │
   ┌────┴────┐
   ↓         ↓
PromQL    Alert Rules
   │         │
   ↓         ↓
Grafana   Alertmanager
   │         │
   ↓         ↓
Dashboards Notifications
```

And when combined with the rest of the observability stack:

```
┌────────────────────────────────────────────────────────┐
│                    Production System                    │
└───────────────────────────┬────────────────────────────┘
                            │
          ┌─────────────────┼─────────────────┐
          ↓                 ↓                 ↓
       Metrics            Logs             Traces
          ↓                 ↓                 ↓
     Prometheus             ELK          OpenTelemetry
          ↓                 ↓                 ↓
       Grafana           Kibana             Jaeger
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ↓
                     Engineering Team
                            ↓
                     Detect → Investigate
                            ↓
                        Resolve
                            ↓
                        Verify
```
