# OpenTelemetry Metrics

## 1. Overview

OpenTelemetry Metrics provides a standardized way to generate, collect, process, and export application and infrastructure metrics.

The basic flow is:

```text
Application
     ↓
OpenTelemetry Metrics API
     ↓
OpenTelemetry SDK
     ↓
Metric Instruments
     ↓
Metric Reader
     ↓
Exporter
     ↓
OpenTelemetry Collector
     ↓
Metrics Backend
     ↓
Prometheus / Grafana
```

Metrics answer questions such as:

```text
How many requests are arriving?
How many requests are failing?
How long are requests taking?
How many active connections exist?
How much memory is being used?
How many orders are processed?
```

---

# 2. What Is a Metric?

A metric is a numerical measurement collected over time.

Examples:

```text
HTTP request count
CPU utilization
Memory usage
Request latency
Database connections
Queue depth
Error count
```

Example:

```text
http.server.request.count = 125000
```

The value becomes useful when analyzed across:

```text
Time
Service
Environment
Endpoint
Status
Region
```

---

# 3. Metrics vs Logs vs Traces

The three observability signals answer different questions.

```text
Metrics
   ↓
What is happening?

Logs
   ↓
What happened?

Traces
   ↓
Where did it happen and why?
```

Example:

```text
Metric:
HTTP error rate = 8%

Log:
Payment API returned HTTP 500

Trace:
Frontend → Orders → Payment → Database
                         ↑
                    Slow operation
```

Together they provide stronger observability.

---

# 4. OpenTelemetry Metrics Architecture

```text
                    Application
                         │
                         ↓
                 Metrics API / SDK
                         │
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          Counter     Histogram   UpDownCounter
             │           │           │
             └───────────┼───────────┘
                         ↓
                    Metric Reader
                         ↓
                      Exporter
                         ↓
                  OTel Collector
                         ↓
                     Prometheus
                         ↓
                      Grafana
```

---

# 5. MeterProvider

The `MeterProvider` is the SDK component responsible for providing meters.

Conceptually:

```text
Application
     ↓
MeterProvider
     ↓
Meter
     ↓
Metric Instrument
```

The application obtains a `Meter` and uses it to create metric instruments.

---

# 6. Meter

A meter creates metric instruments.

Example concept:

```text
payment-meter
      │
      ├── payment_requests
      ├── payment_errors
      └── payment_duration
```

The meter is normally associated with an instrumentation scope.

---

# 7. Metric Instruments

OpenTelemetry provides several metric instruments.

Important instruments include:

```text
Counter
UpDownCounter
Histogram
Observable Counter
Observable UpDownCounter
Observable Gauge
```

The exact API syntax differs between programming languages.

---

# 8. Counter

A counter records a value that normally increases.

Examples:

```text
HTTP requests
Orders processed
Payment attempts
Errors
Messages consumed
```

Conceptually:

```text
100
 ↓
101
 ↓
102
 ↓
103
```

Example metric:

```text
payment_requests_total
```

---

# 9. Counter Example

Suppose a payment service receives requests.

```text
Request 1 → Counter = 1
Request 2 → Counter = 2
Request 3 → Counter = 3
```

Architecture:

```text
Payment Request
      ↓
Counter
      ↓
payment_requests_total
```

The backend can calculate rates from the accumulated counter values.

---

# 10. UpDownCounter

An UpDownCounter can increase and decrease.

Examples:

```text
Active connections
Active requests
Queue depth
Number of running jobs
```

Conceptually:

```text
100
 ↓
120
 ↓
110
 ↓
95
```

This differs from a normal Counter because the value can move in both directions.

---

# 11. Observable Gauge

An Observable Gauge reports a current measurement.

Examples:

```text
Memory usage
CPU temperature
Current queue depth
Current active connections
```

Conceptually:

```text
Application
    ↓
Observe current value
    ↓
Metric
```

It is useful when the application needs to report a value that represents the current state.

---

# 12. Observable Counter

An Observable Counter can report a cumulative value through a callback/observation mechanism.

For example:

```text
Total bytes processed
Total requests received
Total records consumed
```

The application observes the current cumulative value when the metric reader collects it.

---

# 13. Histogram

A histogram records a distribution of measurements.

It is especially useful for:

```text
Request latency
Database latency
Response size
Processing time
```

Example:

```text
HTTP request duration

20ms
35ms
42ms
50ms
120ms
900ms
```

The histogram allows the backend to analyze the distribution.

---

# 14. Why Histogram Is Important

Average latency can hide problems.

Example:

```text
Request latencies:

20ms
20ms
25ms
30ms
2000ms
```

Average latency may look acceptable while one request is extremely slow.

A histogram allows engineers to analyze:

```text
p50
p90
p95
p99
```

This is extremely useful in production.

---

# 15. Percentiles

Common latency percentiles:

```text
p50
p90
p95
p99
```

For example:

```text
p50 = 80ms
p95 = 300ms
p99 = 900ms
```

This means the long-tail latency is much higher than the median.

---

# 16. Metric Attributes

Metrics can have attributes.

Example:

```text
http.server.request.duration
```

with:

```text
http.request.method = GET
http.response.status_code = 200
service.name = orders
deployment.environment = production
```

This allows metrics to be analyzed by dimensions.

---

# 17. Dimensions

Dimensions allow a metric to be broken down.

Example:

```text
HTTP requests
```

can be grouped by:

```text
service
method
route
status
environment
```

Architecture:

```text
HTTP Requests
     ↓
Metric
     ↓
Attributes
     ├── service
     ├── route
     └── status
```

---

# 18. Cardinality

Every unique combination of metric attributes can create a distinct time series.

Example:

```text
service
route
status
region
```

If these have many possible values, the number of time series can grow rapidly.

This is called high cardinality.

---

# 19. High-Cardinality Example

Avoid using unrestricted identifiers such as:

```text
user_id
request_id
session_id
transaction_id
```

as metric dimensions unless there is a very strong reason.

Example:

```text
http_requests{user_id="123"}
http_requests{user_id="124"}
http_requests{user_id="125"}
...
```

Millions of users can create enormous metric cardinality.

---

# 20. Recommended Metric Attributes

Prefer stable dimensions such as:

```text
service.name
deployment.environment
http.request.method
http.route
http.response.status_code
cloud.region
k8s.namespace.name
```

Use dimensions that engineers actually need for operational analysis.

---

# 21. Resource Attributes

Metrics can also contain resource information.

Example:

```text
service.name = payment
service.version = v2.3.1
deployment.environment = production
k8s.namespace.name = production
```

This helps identify which service generated the metric.

---

# 22. Instrumentation Scope

Metrics can be associated with the instrumentation scope that produced them.

Conceptually:

```text
Application
   ↓
Instrumentation Library
   ↓
Meter
   ↓
Metric
```

This helps distinguish telemetry generated by different instrumentation libraries.

---

# 23. Metric Reader

The Metric Reader determines how metric data is collected from the SDK.

Conceptually:

```text
Metric Instrument
      ↓
Metric Reader
      ↓
Exporter
```

The exact implementation differs by language SDK.

---

# 24. Periodic Export

A common architecture is periodic metric export.

```text
Application
     ↓
Metric
     ↓
Metric Reader
     ↓
Every N seconds
     ↓
Exporter
     ↓
Collector
```

The interval should be selected based on:

```text
Monitoring requirements
Telemetry volume
Application overhead
Backend capacity
```

---

# 25. Push vs Pull

Metrics systems commonly use two models.

### Push

```text
Application
     ↓
Collector
     ↓
Backend
```

### Pull

```text
Prometheus
     ↓ scrape
Metrics Endpoint
```

OpenTelemetry can participate in architectures using either approach depending on the SDK, Collector, and backend configuration.

---

# 26. OpenTelemetry Metrics With Prometheus

A common architecture is:

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTel Collector
     ↓
Prometheus-compatible metrics
     ↓
Prometheus
     ↓
Grafana
```

Another architecture can expose a metrics endpoint that Prometheus scrapes.

The correct design depends on the environment and chosen integration.

---

# 27. Prometheus as Metrics Backend

Prometheus stores time-series metrics.

Example:

```text
http_requests_total{
  service="orders",
  method="GET",
  status="200"
}
```

Prometheus can then query the metric using PromQL.

---

# 28. Grafana Visualization

Grafana can visualize OpenTelemetry-generated metrics after they reach Prometheus or another supported metrics backend.

Architecture:

```text
Application
     ↓
OTel SDK
     ↓
Collector
     ↓
Prometheus
     ↓
Grafana
```

Grafana can display:

```text
Request rate
Error rate
Latency
CPU
Memory
Saturation
```

---

# 29. Application Request Counter

A microservice can record:

```text
http_requests_total
```

Conceptually:

```text
HTTP Request
      ↓
Counter
      ↓
http_requests_total
```

Prometheus can calculate request rate from the counter.

---

# 30. Error Counter

A service can maintain:

```text
http_errors_total
```

Architecture:

```text
HTTP 500
   ↓
Error Counter
   ↓
http_errors_total
```

Then calculate:

```text
Error Rate =
Errors / Total Requests
```

---

# 31. Latency Histogram

A service can record:

```text
http_server_request_duration
```

Conceptually:

```text
Request
   ↓
Measure duration
   ↓
Histogram
   ↓
Collector
   ↓
Prometheus
```

Grafana can then display latency percentiles.

---

# 32. Business Metrics

OpenTelemetry metrics are not limited to infrastructure.

A microservice can expose business metrics such as:

```text
orders_created
payments_completed
payments_failed
inventory_reservations
notifications_sent
```

Example:

```text
orders_created_total
```

This helps connect technical health with business behavior.

---

# 33. Business Metric Example

Suppose:

```text
Application
     ↓
Order created
     ↓
Counter++
```

Metric:

```text
orders_created_total
```

Then Grafana can display:

```text
Orders per minute
Orders per hour
Orders by service
Orders by environment
```

---

# 34. Database Metrics

Applications can expose metrics such as:

```text
db_connections_active
db_query_duration
db_errors_total
db_queries_total
```

These can help identify database-related bottlenecks.

Example:

```text
Application latency ↑
       ↓
Database duration ↑
       ↓
Database becomes suspect
```

---

# 35. Queue Metrics

For RabbitMQ or Kafka-style systems, useful metrics include:

```text
messages_published
messages_consumed
queue_depth
consumer_lag
processing_duration
```

Example:

```text
Queue depth ↑
      ↓
Consumers cannot keep up
```

This can reveal asynchronous processing problems.

---

# 36. Kubernetes Metrics

Metrics can also identify Kubernetes behavior.

Examples:

```text
Pod CPU
Pod memory
Container restarts
Node CPU
Node memory
Network traffic
Pod availability
```

These metrics can come from Kubernetes-aware components and exporters.

---

# 37. EKS Metrics Architecture

A production EKS architecture might be:

```text
                         EKS
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       Orders          Payment         Inventory
          │               │               │
       OTel SDK         OTel SDK         OTel SDK
          │               │               │
          └───────────────┼───────────────┘
                          ↓
                     OTel Agent
                          ↓
                     OTel Gateway
                          ↓
                      Prometheus
                          ↓
                       Grafana
```

---

# 38. Metrics Collection From Kubernetes

A Collector can receive metrics through different mechanisms.

Examples:

```text
OTLP
Prometheus scraping
Host metrics
Kubernetes metadata
```

Architecture:

```text
Application Metrics
        ↓
OTLP

Node Metrics
        ↓
Host Metrics

Exporter Metrics
        ↓
Prometheus Scraping
```

The Collector can combine these sources.

---

# 39. Metric Pipeline

A production metric pipeline can be:

```text
Metrics
   ↓
OTLP Receiver
   ↓
Memory Limiter
   ↓
Resource Processor
   ↓
Filter
   ↓
Batch
   ↓
Prometheus Export
```

This provides centralized metric processing.

---

# 40. Metric Filtering

Not every metric needs to reach the backend.

Example:

```text
All Metrics
    ↓
Filter
    ↓
Operational Metrics
    ↓
Prometheus
```

Filtering can reduce:

```text
Storage
Network
Backend load
Cardinality
Cost
```

---

# 41. Metric Transformation

Metric data may need normalization.

Possible operations:

```text
Rename attributes
Add resource information
Remove unwanted attributes
Normalize labels
```

This creates consistency across multiple services.

---

# 42. Metric Aggregation

Metrics can be aggregated at different stages.

For example:

```text
Individual Requests
      ↓
Histogram
      ↓
Aggregated Distribution
      ↓
Backend
```

Aggregation reduces the amount of raw information that needs to be stored while retaining useful statistical information.

---

# 43. Metric Temporality

OpenTelemetry metrics can represent measurements using different temporality concepts.

Two important concepts are:

```text
Cumulative
Delta
```

### Cumulative

The value represents the accumulation from a starting point.

```text
100
 ↓
150
 ↓
200
```

### Delta

The value represents the change during an interval.

```text
+50
+50
+25
```

Backend compatibility matters when choosing or configuring temporality.

---

# 44. Cumulative Metrics

Counters are commonly represented cumulatively.

Example:

```text
Request count:

1000
1100
1200
1300
```

Each measurement includes the total accumulated value.

Prometheus commonly works with cumulative counters.

---

# 45. Delta Metrics

Delta represents the change during an interval.

Example:

```text
Minute 1 → 100 requests
Minute 2 → 150 requests
Delta    → 50 requests
```

The consuming backend must understand the selected metric semantics.

---

# 46. Metric Aggregation

Different instruments use different aggregation concepts.

Examples:

```text
Counter
   ↓
Sum

Histogram
   ↓
Histogram distribution

Gauge
   ↓
Current value
```

The SDK and backend must interpret the metric correctly.

---

# 47. Metric Naming

Metric names should clearly describe what is measured.

Good:

```text
http.server.request.duration
orders.created
payment.errors
```

Avoid confusing names such as:

```text
metric1
value
test_metric
```

Follow OpenTelemetry semantic conventions where applicable.

---

# 48. Semantic Conventions

OpenTelemetry semantic conventions standardize commonly used metric names and attributes.

Examples:

```text
HTTP
Database
Messaging
RPC
Kubernetes
Cloud
```

Using semantic conventions helps different teams produce consistent telemetry.

---

# 49. HTTP Metrics

Useful HTTP metrics include:

```text
Request count
Request duration
Active requests
Response status
```

Example dimensions:

```text
method
route
status
service
```

Architecture:

```text
HTTP
 ↓
Instrumentation
 ↓
Metrics
 ↓
Collector
 ↓
Prometheus
```

---

# 50. RED Method

The RED method is useful for service monitoring.

```text
R = Rate
E = Errors
D = Duration
```

Example:

```text
Rate:
500 requests/sec

Errors:
2%

Duration:
p95 = 250ms
```

OpenTelemetry metrics can provide the underlying measurements needed for RED dashboards.

---

# 51. USE Method

For infrastructure:

```text
U = Utilization
S = Saturation
E = Errors
```

Example:

```text
CPU utilization
Memory saturation
Disk errors
Network errors
```

Together with application metrics, this helps troubleshoot service performance.

---

# 52. Metric Correlation

Metrics become much more powerful when correlated with traces and logs.

Example:

```text
Metric:
Payment latency ↑
       ↓
Trace:
Database span slow
       ↓
Log:
Database connection timeout
```

This gives a complete troubleshooting path.

---

# 53. Metrics and Trace Correlation

Suppose Grafana shows:

```text
payment latency p99 = 2.5s
```

The engineer can investigate traces:

```text
Payment Request
      ↓
Database Span = 2.2s
```

Then inspect logs:

```text
Connection timeout
```

This is the practical value of unified observability.

---

# 54. Metrics and Logs Correlation

Example:

```text
Metric:
payment_errors_total ↑
       ↓
Logs:
"Payment gateway timeout"
```

Use common resource attributes:

```text
service.name
service.version
deployment.environment
```

to make correlation easier.

---

# 55. Metric Export

The SDK can export metrics using OTLP.

```text
Application
     ↓
Metric SDK
     ↓
OTLP Exporter
     ↓
Collector
```

The Collector can then process and export the metrics to the selected backend.

---

# 56. Collector Metric Pipeline

Example:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

processors:
  memory_limiter:
  batch:

exporters:
  otlp:
    endpoint: metrics-backend:4317

service:
  pipelines:
    metrics:
      receivers:
        - otlp
      processors:
        - memory_limiter
        - batch
      exporters:
        - otlp
```

This is a conceptual configuration.

---

# 57. Prometheus Scraping Architecture

Another common approach:

```text
Application
     ↓
OTel SDK
     ↓
Metrics Endpoint
     ↑
Prometheus
     ↓
Grafana
```

Here Prometheus directly scrapes the metrics endpoint.

The choice between OTLP push and Prometheus pull should be based on the architecture and operational requirements.

---

# 58. Collector as Metrics Gateway

The Collector can centralize metrics:

```text
Service A ─┐
Service B ─┤
Service C ─┼→ OTel Collector → Prometheus
Service D ─┘
```

Benefits:

```text
Central filtering
Central enrichment
Central routing
Central security
```

---

# 59. Metric Resource Enrichment

Suppose a metric arrives as:

```text
http_requests_total
```

The Collector can enrich it with:

```text
service.name=payment
environment=production
cluster=prod-eks
namespace=production
```

Then dashboards can filter:

```text
environment="production"
service.name="payment"
```

---

# 60. Metric Cardinality Management

A production team should regularly inspect cardinality.

Dangerous:

```text
request_id
user_id
session_id
```

Better:

```text
service
route
method
status
environment
```

Cardinality should be considered before adding any new metric attribute.

---

# 61. Metric Sampling vs Trace Sampling

Metrics and traces are handled differently.

Trace sampling:

```text
Keep / Drop traces
```

Metrics:

```text
Aggregate measurements
Control dimensions
Filter metrics
```

Do not treat metric collection as if it were trace sampling.

---

# 62. Metric Resolution

Metrics may be collected at different intervals.

Example:

```text
5 seconds
10 seconds
30 seconds
60 seconds
```

Shorter intervals provide more detail but increase:

```text
CPU
Network
Storage
Backend ingestion
```

Choose a resolution appropriate to the monitoring requirement.

---

# 63. SLO Metrics

Metrics can support Service Level Objectives.

Example:

```text
SLO:
99.9% successful requests
```

Measure:

```text
Successful requests
Total requests
```

Then calculate:

```text
Availability =
Successful Requests / Total Requests
```

---

# 64. Error Budget

Suppose:

```text
SLO = 99.9%
```

Allowed failure:

```text
0.1%
```

Metrics can track:

```text
Error budget remaining
Error budget consumed
Burn rate
```

This connects observability with reliability engineering.

---

# 65. Alerting From Metrics

Metrics can trigger alerts.

Example:

```text
Error rate > 5%
       ↓
Prometheus
       ↓
Alert rule
       ↓
Alertmanager
       ↓
Notification
```

OpenTelemetry generates or transports the metric; the alerting system evaluates it.

---

# 66. Useful Production Alerts

Examples:

```text
High error rate
High latency
Low request rate
High CPU
High memory
High queue depth
Database connection exhaustion
Pod restart rate
Collector export failures
```

Alerts should represent actionable conditions.

---

# 67. Avoid Noisy Alerts

Bad:

```text
CPU > 60%
```

for every short spike.

Better:

```text
CPU > 90%
for 10 minutes
```

when that threshold reflects a real operational problem.

The exact threshold should be based on service behavior.

---

# 68. Metric Dashboard Structure

A production Grafana dashboard can be organized as:

```text
Service Overview
│
├── Request Rate
├── Error Rate
├── p50 Latency
├── p95 Latency
├── p99 Latency
├── CPU
├── Memory
├── Pod Restarts
└── Dependency Health
```

---

# 69. Microservice Dashboard

For a payment service:

```text
Payment Service
│
├── Requests/sec
├── Error %
├── p95 latency
├── p99 latency
├── Payment success rate
├── Payment failure count
├── DB latency
├── Active connections
└── Pod CPU / Memory
```

This combines technical and business metrics.

---

# 70. Metric Naming in a Microservices Platform

Example:

```text
orders_created
orders_failed
payment_requests
payment_errors
inventory_reservations
notification_sent
```

Use consistent naming conventions across services.

---

# 71. SDK Metric Initialization

A typical conceptual startup sequence:

```text
Application starts
      ↓
Create Resource
      ↓
Create MeterProvider
      ↓
Configure Metric Reader
      ↓
Configure Exporter
      ↓
Register Provider
      ↓
Create Meter
      ↓
Create Instruments
```

---

# 72. Graceful Shutdown

Metric exporters may have pending data during application shutdown.

Use graceful shutdown:

```text
SIGTERM
   ↓
Stop application
   ↓
Flush metrics
   ↓
Shutdown exporter
   ↓
Exit
```

This reduces telemetry loss during deployments.

---

# 73. Kubernetes Shutdown

In EKS:

```text
Pod
 ↓
SIGTERM
 ↓
Application shutdown
 ↓
Metric flush
 ↓
Exporter shutdown
 ↓
Container exits
```

Configure an appropriate termination grace period.

---

# 74. Metric Reliability

Metrics should not become a critical dependency for the business application.

If Prometheus or the Collector is unavailable:

```text
Payment
   ↓
Should continue working
```

rather than:

```text
Telemetry failure
   ↓
Payment failure
```

Observability must be isolated from business availability.

---

# 75. Metric Export Failure

If the Collector is temporarily unavailable:

```text
Application
     ↓
Metric Export
     X
Collector
```

Depending on SDK behavior, telemetry may be queued, retried, or dropped.

The system should have bounded resource usage.

---

# 76. Collector Backpressure

If the metrics backend is slow:

```text
Collector
    ↓
Backend
    ↓
Slow
```

Then:

```text
Queue
   ↓
Retry
   ↓
Export
```

If the queue becomes excessive:

```text
Memory ↑
CPU ↑
Telemetry loss
```

Monitor the Collector closely.

---

# 77. Metrics Performance

Metrics can introduce overhead through:

```text
Metric recording
Attribute processing
Aggregation
Exporting
Serialization
```

Reduce overhead by:

```text
Reasonable metric count
Controlled cardinality
Appropriate collection interval
Efficient batching
Filtering unnecessary metrics
```

---

# 78. High-Cardinality Incident

Example:

A developer adds:

```text
user_id
```

to a frequently recorded metric.

Result:

```text
Millions of time series
        ↓
Prometheus memory ↑
        ↓
Query latency ↑
        ↓
Storage ↑
```

Prevention:

```text
Review metric dimensions
Use stable labels
Avoid unbounded identifiers
```

---

# 79. Metrics Cost Optimization

Reduce unnecessary cost through:

```text
Metric filtering
Cardinality control
Appropriate collection intervals
Aggregation
Removing unused metrics
Retention policies
```

Do not optimize by blindly deleting important operational metrics.

---

# 80. Production Metric Architecture

```text
                    EKS
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
     Service A    Service B    Service C
        │            │            │
      OTel SDK     OTel SDK     OTel SDK
        │            │            │
        └────────────┼────────────┘
                     ↓
                 OTel Agent
                     ↓
                OTel Gateway
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
      Processing             Export
          │                     │
          └──────────→ Prometheus
                            ↓
                         Grafana
```

---

# 81. Production Metric Pipeline

Recommended conceptual flow:

```text
Application
     ↓
Metric Instrument
     ↓
MeterProvider
     ↓
Metric Reader
     ↓
OTLP Exporter
     ↓
OTel Agent
     ↓
OTel Gateway
     ↓
Memory Limiter
     ↓
Resource Enrichment
     ↓
Filtering
     ↓
Batch
     ↓
Prometheus
     ↓
Grafana
```

---

# 82. Metric Troubleshooting

When a metric is missing:

```text
1. Is the instrument created?
2. Is the MeterProvider configured?
3. Is the metric being recorded?
4. Is the MetricReader active?
5. Is the exporter configured?
6. Is the Collector reachable?
7. Is the metrics receiver running?
8. Is the metrics pipeline configured?
9. Is the metric being filtered?
10. Is the backend receiving it?
```

---

# 83. Troubleshooting: No Data in Prometheus

Check:

```text
Application
   ↓
SDK
   ↓
OTLP Exporter
   ↓
Collector
   ↓
Metrics Pipeline
   ↓
Prometheus Export
   ↓
Prometheus
```

Verify each layer rather than immediately assuming Prometheus is broken.

---

# 84. Troubleshooting: Metric Exists but Dashboard Is Empty

Possible causes:

```text
Wrong metric name
Wrong labels
Wrong datasource
Incorrect PromQL
Wrong time range
Wrong environment filter
Metric renamed
Metric filtered
```

Check the metric directly in Prometheus before debugging Grafana.

---

# 85. Troubleshooting: Missing Labels

If:

```text
service.name
```

is missing, check:

```text
Resource configuration
SDK configuration
Collector resource processor
Exporter mapping
Backend mapping
```

The problem may occur at any stage of the pipeline.

---

# 86. Troubleshooting: High Cardinality

Check:

```text
Metric labels
Attribute values
Number of unique time series
Recent metric changes
```

Identify recently introduced attributes.

Typical offenders:

```text
user_id
request_id
session_id
full URL
dynamic path
```

---

# 87. Metrics Security

Metrics can contain sensitive information if poorly designed.

Avoid metric attributes containing:

```text
Passwords
Tokens
Email addresses
Personal information
Sensitive business identifiers
```

Metrics should expose operationally useful information without exposing secrets.

---

# 88. Metrics and PII

Bad:

```text
payment_requests{
    customer_email="user@example.com"
}
```

Better:

```text
payment_requests{
    service="payment",
    status="success"
}
```

Keep personally identifiable information out of metric dimensions.

---

# 89. Metrics Testing

Before production:

```text
Generate traffic
      ↓
Record metrics
      ↓
Verify Collector
      ↓
Verify Prometheus
      ↓
Verify Grafana
      ↓
Verify alerts
```

Test:

```text
Normal traffic
Error traffic
High traffic
Application restart
Collector restart
Backend interruption
```

---

# 90. Load Testing Metrics

During load testing:

```text
Requests/sec ↑
       ↓
CPU ↑
       ↓
Latency ↑
       ↓
Error rate ↑
```

Metrics help identify system limits.

Observe:

```text
Application
Collector
Prometheus
Kubernetes
Database
```

---

# 91. Metrics During Deployment

A deployment should be observable.

Before deployment:

```text
Error rate = 0.5%
p95 = 200ms
```

After deployment:

```text
Error rate = 5%
p95 = 800ms
```

Metrics can identify a release regression quickly.

---

# 92. Canary Deployment Metrics

For canary deployment:

```text
Version A → 95%
Version B → 5%
```

Compare:

```text
Error rate
Latency
Request rate
Business success rate
```

If version B performs poorly:

```text
Rollback
```

This makes metrics part of the deployment safety mechanism.

---

# 93. Metrics for HPA

Metrics can also support Kubernetes autoscaling.

Conceptually:

```text
Application Load
      ↓
Metric
      ↓
Prometheus
      ↓
HPA
      ↓
More Pods
```

The exact HPA integration depends on the metric source and Kubernetes metrics architecture.

---

# 94. Metrics and Capacity Planning

Historical metrics can answer:

```text
When does traffic peak?
How fast is traffic growing?
How much CPU does each service require?
When does the database saturate?
How much capacity is required?
```

Example:

```text
Traffic
  ↓
Capacity Trend
  ↓
Infrastructure Planning
```

---

# 95. Metrics Retention

Metric retention determines how far back historical data is available.

Example:

```text
Short-term:
Detailed operational metrics

Long-term:
Aggregated trend metrics
```

Retention should consider:

```text
Incident investigation
Capacity planning
SLO analysis
Cost
Storage
Compliance
```

---

# 96. Metrics and Recording Rules

Prometheus recording rules can precompute frequently used expressions.

Example concept:

```text
Raw Metrics
    ↓
PromQL Expression
    ↓
Recording Rule
    ↓
Precomputed Metric
```

This can improve dashboard performance for expensive queries.

---

# 97. Metrics Alerting Architecture

```text
OpenTelemetry
      ↓
Collector
      ↓
Prometheus
      ↓
PromQL
      ↓
Alert Rules
      ↓
Alertmanager
      ↓
Slack / Email / Pager
```

OpenTelemetry provides the telemetry; Prometheus and Alertmanager can handle evaluation and notification in a Prometheus-based architecture.

---

# 98. Golden Signals

A production service should generally monitor:

```text
Latency
Traffic
Errors
Saturation
```

Example:

```text
Traffic:
800 req/s

Errors:
1.2%

Latency:
p95 = 220ms

Saturation:
CPU = 75%
```

These provide a strong first-level service health view.

---

# 99. Production Checklist

```text
[ ] MeterProvider configured
[ ] Service resource configured
[ ] Standard semantic conventions used
[ ] Counters configured
[ ] Histograms configured
[ ] Appropriate gauges configured
[ ] Metric attributes reviewed
[ ] Cardinality controlled
[ ] OTLP exporter configured
[ ] MetricReader configured
[ ] Collector metrics pipeline configured
[ ] Batching configured
[ ] Memory protection configured
[ ] Prometheus integration verified
[ ] Grafana dashboards created
[ ] Alerts configured
[ ] SLO metrics defined
[ ] Sensitive data excluded
[ ] Graceful shutdown configured
```

---

# 100. EKS Production Checklist

```text
[ ] OTel SDK in application
[ ] OTel Agent deployed
[ ] OTel Gateway deployed
[ ] Gateway replicas configured
[ ] Kubernetes Service configured
[ ] Resource requests/limits configured
[ ] Memory limiter configured
[ ] Network policies configured
[ ] TLS configured where required
[ ] RBAC reviewed
[ ] Prometheus integration verified
[ ] Grafana dashboards verified
[ ] Collector monitored
[ ] Metric cardinality monitored
[ ] HPA metrics validated
[ ] Deployment metrics available
[ ] Rollback metrics available
```

---

# 101. Complete Metrics Architecture

```text
                                  USERS
                                    │
                                    ↓
                                   ALB
                                    │
                                    ↓
                             EKS Microservices
                                    │
                 ┌──────────────────┼──────────────────┐
                 ↓                  ↓                  ↓
              Orders             Payment           Inventory
                 │                  │                  │
              OTel SDK           OTel SDK           OTel SDK
                 │                  │                  │
                 └──────────────────┼──────────────────┘
                                    ↓
                                OTel Agent
                                    ↓
                               OTel Gateway
                                    │
                          ┌─────────┴─────────┐
                          ↓                   ↓
                    Metric Processing     Collector Metrics
                          │                   │
                          ↓                   ↓
                      Prometheus          Prometheus
                          │                   │
                          └─────────┬─────────┘
                                    ↓
                                 Grafana
                                    │
              ┌─────────────────────┼─────────────────────┐
              ↓                     ↓                     ↓
         Service Health         Kubernetes           SLO / Alerts
```

---

# 102. Final Mental Model

Remember OpenTelemetry Metrics as:

```text
MEASURE
   ↓
MeterProvider
   ↓
Instrument
   ↓
Record Value
   ↓
MetricReader
   ↓
Exporter
   ↓
Collector
   ↓
Prometheus
   ↓
Grafana
```

For production:

```text
Metrics
   ↓
Stable dimensions
   ↓
Controlled cardinality
   ↓
Resource enrichment
   ↓
Batching
   ↓
Secure transport
   ↓
Prometheus
   ↓
Dashboards + Alerts
```

The key principle is:

**OpenTelemetry Metrics provides a standardized application-side framework for generating and exporting numerical measurements. In a production EKS environment, metrics should use stable resource attributes and semantic conventions, avoid high-cardinality dimensions, use appropriate instruments such as Counters and Histograms, and flow through an OpenTelemetry Collector before reaching Prometheus and Grafana. Metrics should then support service health, RED/USE monitoring, SLOs, alerting, capacity planning, and safe production deployments.**
