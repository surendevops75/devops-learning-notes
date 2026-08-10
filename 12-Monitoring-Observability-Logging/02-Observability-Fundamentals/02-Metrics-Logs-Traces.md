# Metrics, Logs and Traces

Metrics, Logs, and Traces are the three traditional pillars of observability.

They provide different views of the same production system:

```
Metrics
    ↓
What is happening?

Logs
    ↓
What happened?

Traces
    ↓
Where did it happen and how did the request flow?
```

A mature observability platform combines all three.

---

# 1. The Three Pillars

```
┌──────────────────────────────┐
│       Observability          │
└──────────────┬───────────────┘
               |
    +----------+----------+
    |          |          |
    ↓          ↓          ↓
 Metrics      Logs      Traces
    |          |          |
    ↓          ↓          ↓
 Trends      Events    Request Flow
    |          |          |
    +----------+----------+
               |
               ↓
         Root Cause
```

---

# 2. Why Three Signals Are Required

One telemetry signal rarely provides the complete picture.

Example:

```
Metric:
Error Rate = 8%
```

This tells us that something is wrong.

But it does not necessarily tell us why.

Logs may show:

```
Database connection timeout
```

Traces may show:

```
API
  |
  ↓
Order Service
  |
  ↓
Database
  |
  ↓
3.2 seconds
```

Together:

```
Metrics → Detect
Logs    → Explain
Traces  → Locate
```

---

# 3. Metrics

Metrics are numerical measurements collected over time.

Examples:

```
CPU Usage
Memory Usage
Request Count
Error Count
Request Rate
Latency
Queue Depth
Active Connections
```

Example:

```
cpu_usage = 72

http_requests_total = 150000

active_connections = 85
```

Metrics are particularly useful for:

```
Monitoring
Alerting
Dashboards
Trends
Capacity Planning
SLOs
```

---

# 4. Characteristics of Metrics

Metrics are generally:

```
Numerical
Time-Series Based
Aggregatable
Efficient to Store
Easy to Query
```

Example:

```
http_requests_total

10:00 → 1000
10:01 → 1200
10:02 → 1500
10:03 → 1900
```

This allows engineers to understand how a system changes over time.

---

# 5. Metric Types

Common Prometheus metric types include:

```
Counter
Gauge
Histogram
Summary
```

Each type represents a different kind of measurement.

---

# 6. Counter

A counter represents a value that generally increases over time.

Examples:

```
Total Requests
Total Errors
Total Jobs Processed
Total HTTP Responses
```

Example:

```
http_requests_total

100
200
300
400
```

Counters can reset when the application restarts.

---

# 7. Counter Example

Suppose an application receives:

```
100 requests
```

Then:

```
http_requests_total = 100
```

After another 50 requests:

```
http_requests_total = 150
```

After another 100:

```
http_requests_total = 250
```

Prometheus can calculate the rate of increase.

Example:

```
rate(http_requests_total[5m])
```

---

# 8. Gauge

A gauge represents a value that can increase or decrease.

Examples:

```
CPU Usage
Memory Usage
Active Connections
Queue Depth
Temperature
```

Example:

```
active_connections

50
70
40
90
60
```

Unlike a counter, a gauge can move in either direction.

---

# 9. Histogram

A histogram measures observations and groups them into buckets.

Common use:

```
Request Duration
Response Size
Query Latency
```

Example:

```
Request Duration

< 100ms
< 250ms
< 500ms
< 1s
< 2s
```

Histograms are useful for calculating latency distributions and
percentiles.

---

# 10. Histogram Example

Suppose request durations are:

```
50ms
80ms
120ms
300ms
700ms
1.2s
```

A histogram can group these observations into buckets.

Example:

```
<= 100ms → 2
<= 250ms → 3
<= 500ms → 4
<= 1s    → 5
<= 2s    → 6
```

This helps understand the distribution of request latency.

---

# 11. Summary

A summary also calculates observations and quantiles.

It can provide values such as:

```
P50
P90
P95
P99
```

However, histograms are often preferred when aggregation across
multiple instances is required.

---

# 12. Metric Labels

Metrics can include labels.

Example:

```
http_requests_total{
    service="order-service",
    method="POST",
    status="200"
}
```

Labels provide dimensions for querying.

For example:

```
service="order-service"
```

or:

```
status="500"
```

---

# 13. Metric Cardinality

Cardinality represents the number of unique label combinations.

Example:

```
service
method
status
```

usually produces manageable combinations.

But:

```
user_id
request_id
session_id
```

can create extremely high cardinality.

High cardinality can increase:

```
Memory Usage
Storage
Query Cost
Processing Cost
```

Avoid putting highly unique values into metrics unless there is a
specific reason and the architecture supports it.

---

# 14. Metrics in Kubernetes

Important Kubernetes metrics include:

```
Node CPU
Node Memory
Pod CPU
Pod Memory
Container Restarts
Deployment Replicas
HPA Replicas
Network Traffic
Storage Usage
```

Example:

```
Pod
  |
  +--- CPU = 400m
  +--- Memory = 512Mi
  +--- Restarts = 0
```

---

# 15. Application Metrics

Application metrics should describe application behavior.

Examples:

```
Request Rate
Error Rate
Latency
Active Requests
Queue Processing Rate
Database Connections
Cache Hit Rate
```

Example:

```
order_requests_total

order_errors_total

order_request_duration_seconds
```

---

# 16. Golden Metrics

Important application measurements include:

```
Traffic
Errors
Latency
Saturation
```

These are commonly known as the Golden Signals.

Example:

```
Service
  |
  +--- Traffic
  +--- Errors
  +--- Latency
  +--- Saturation
```

These will be covered in detail in the next file.

---

# 17. Prometheus

Prometheus is a metrics monitoring and time-series database platform.

A common architecture is:

```
Application
    |
    ↓
/metrics
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

Prometheus can scrape metrics from:

```
Applications
Exporters
Kubernetes Targets
Infrastructure Components
```

---

# 18. Prometheus Scraping

Prometheus commonly uses a pull-based model.

Architecture:

```
Prometheus
    |
    | HTTP GET
    ↓
/metrics
    |
    ↓
Application
```

The application exposes metrics.

Prometheus periodically retrieves them.

---

# 19. Prometheus Querying

Prometheus uses PromQL.

Example:

```
rate(http_requests_total[5m])
```

This calculates request rate over a five-minute window.

Another example:

```
rate(http_requests_total{status="500"}[5m])
```

This focuses on HTTP 500 responses.

---

# 20. Metrics for Alerting

Metrics can be used to generate alerts.

Example:

```
CPU > 90%
for 10 minutes
```

or:

```
Error Rate > 5%
for 5 minutes
```

or:

```
P99 Latency > 1 second
```

Alerting should focus on actionable conditions.

---

# 21. Logs

Logs are records of events generated by systems.

Example:

```
2026-08-10 10:20:15
ERROR
payment-service
Database connection timeout
```

Logs provide detailed context.

---

# 22. What Logs Tell Us

Logs can answer:

```
What happened?

When did it happen?

Which service generated it?

Which operation failed?

What error occurred?

Which dependency was involved?

What was the application doing?
```

---

# 23. Log Levels

Common log levels:

```
DEBUG
INFO
WARN
ERROR
FATAL
```

Example:

```
INFO:
Payment request started

WARN:
Payment API response is slow

ERROR:
Payment API request failed
```

---

# 24. DEBUG Logs

DEBUG logs provide detailed information useful during development
and troubleshooting.

Example:

```
DEBUG:
Processing order ID 12345
```

Excessive DEBUG logging in production can create:

```
Large Log Volume
Higher Storage Cost
More Noise
```

Use it carefully.

---

# 25. INFO Logs

INFO logs describe normal application activity.

Examples:

```
Application Started
Deployment Completed
User Authenticated
Order Created
```

These provide operational context.

---

# 26. WARN Logs

WARN indicates an unusual condition that may require investigation.

Examples:

```
Connection Pool Near Limit
Retry Attempt
Slow Dependency
Cache Miss Spike
```

A warning does not necessarily mean the system is failing.

---

# 27. ERROR Logs

ERROR indicates a failed operation or significant problem.

Examples:

```
Database Connection Failed
Payment Request Failed
File Processing Failed
External API Timeout
```

Errors should contain enough context to investigate the problem.

---

# 28. Structured Logging

Structured logging represents log events in a structured format.

Example:

```
{
  "timestamp": "2026-08-10T10:30:00Z",
  "service": "payment-service",
  "level": "ERROR",
  "status": 500,
  "message": "Database timeout"
}
```

This is easier to search than an unstructured message.

---

# 29. Unstructured Logging

Example:

```
ERROR payment service database timeout
```

This can still be useful, but searching and parsing may be more
difficult.

Structured logging is generally preferable for modern distributed
applications.

---

# 30. Important Log Fields

Useful fields include:

```
timestamp
level
service
environment
version
message
status
endpoint
trace_id
span_id
```

Example:

```
{
  "timestamp": "...",
  "level": "ERROR",
  "service": "order-service",
  "environment": "production",
  "version": "v2.1",
  "trace_id": "abc123",
  "message": "Payment timeout"
}
```

---

# 31. Trace ID in Logs

Trace IDs connect logs to distributed traces.

Example:

```
Log:

service = payment-service
trace_id = abc123
error = timeout
```

Search for:

```
trace_id = abc123
```

Then inspect the corresponding trace.

This is one of the most useful forms of telemetry correlation.

---

# 32. Log Collection

A centralized logging architecture can be:

```
Application
    |
    ↓
Container stdout/stderr
    |
    ↓
Log Collector
    |
    ↓
Logstash
    |
    ↓
Elasticsearch
    |
    ↓
Kibana
```

---

# 33. ELK Stack

ELK means:

```
Elasticsearch
Logstash
Kibana
```

Architecture:

```
Logs
  |
  ↓
Logstash
  |
  ↓
Elasticsearch
  |
  ↓
Kibana
```

---

# 34. Elasticsearch

Elasticsearch stores and indexes logs.

Example:

```
{
  "service": "order-service",
  "level": "ERROR",
  "status": 500,
  "message": "Database timeout"
}
```

Engineers can search and filter the stored documents.

---

# 35. Logstash

Logstash processes logs.

Pipeline:

```
Input
  |
  ↓
Filter
  |
  ↓
Transform
  |
  ↓
Output
```

Example:

```
Raw Log
  |
  ↓
Parse
  |
  ↓
Add Service
  |
  ↓
Add Environment
  |
  ↓
Elasticsearch
```

---

# 36. Kibana

Kibana provides:

```
Log Search
Filtering
Dashboards
Visualizations
Investigation
```

Example query:

```
service:"payment-service"
AND level:"ERROR"
```

---

# 37. Log Retention

Log retention defines how long logs are stored.

Consider:

```
Storage Cost
Investigation Requirements
Compliance
Business Requirements
```

High-volume logs should not be retained indefinitely without a reason.

---

# 38. Sensitive Information in Logs

Never intentionally log sensitive information such as:

```
Passwords
Access Tokens
API Keys
Private Keys
Credentials
```

Example:

```
BAD:

password=MySecretPassword

BETTER:

password=[REDACTED]
```

---

# 39. Log Volume

Large applications can produce huge amounts of logs.

Example:

```
100 Services
    |
    ↓
Thousands of Containers
    |
    ↓
Millions of Log Events
```

Control volume using:

```
Appropriate Log Levels
Filtering
Sampling
Retention
Aggregation
```

---

# 40. Traces

A trace represents a complete logical request through a distributed
system.

Example:

```
User
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

This entire journey can be represented as one trace.

---

# 41. Why Traces Are Important

In a monolithic application:

```
Request
  |
  ↓
Application
  |
  ↓
Database
```

Troubleshooting can be relatively straightforward.

In microservices:

```
Request
  |
  ↓
Service A
  |
  ↓
Service B
  |
  ↓
Service C
  |
  ↓
Database
  |
  ↓
External API
```

Tracing helps identify where the request spent time.

---

# 42. Trace Components

A trace contains spans.

Example:

```
Trace
  |
  +--- Span A
  |
  +--- Span B
  |
  +--- Span C
  |
  +--- Span D
```

Each span represents an operation.

---

# 43. Span

A span can contain:

```
Span ID
Parent Span ID
Trace ID
Service Name
Operation
Start Time
End Time
Duration
Status
Attributes
```

Example:

```
service.name = payment-service

operation = POST /payments

duration = 850ms
```

---

# 44. Parent and Child Spans

Distributed operations form relationships.

Example:

```
Trace
  |
  +--- Order Service
         |
         +--- Payment Service
                |
                +--- Database
```

Order Service span can be the parent.

Payment Service span can be a child.

Database operation can be another child span.

---

# 45. Trace ID

Trace ID identifies the complete request.

Example:

```
trace_id = abc123
```

All spans associated with the same request use the same trace
context.

---

# 46. Span ID

Span ID identifies an individual operation.

Example:

```
trace_id = abc123

span_id = 001
span_id = 002
span_id = 003
```

Each span has its own identity.

---

# 47. Distributed Trace Example

Request:

```
POST /orders
```

Trace:

```
Trace ID = abc123

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

Timing:

```
ALB              10ms
Order Service    40ms
Payment          900ms
Inventory        50ms
Database         30ms
```

Payment Service is the main latency contributor.

---

# 48. OpenTelemetry

OpenTelemetry provides a standardized approach to collecting
observability telemetry.

It supports:

```
Metrics
Logs
Traces
```

A typical architecture:

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
Backend
```

---

# 49. OpenTelemetry SDK

Applications can be instrumented using OpenTelemetry SDKs.

The SDK can generate telemetry such as:

```
Spans
Metrics
Logs
```

Example:

```
Java Application
    |
    ↓
OpenTelemetry SDK
    |
    ↓
Telemetry
```

The implementation depends on the programming language and framework.

---

# 50. OpenTelemetry Collector

The Collector receives, processes, and exports telemetry.

Architecture:

```
Application
    |
    ↓
Receiver
    |
    ↓
Processor
    |
    ↓
Exporter
    |
    ↓
Backend
```

---

# 51. Collector Receivers

Receivers accept telemetry from sources.

Examples:

```
OTLP
Prometheus
Jaeger
Host Metrics
Filelog
```

The receiver depends on the telemetry source and deployment design.

---

# 52. Collector Processors

Processors modify or process telemetry.

Examples:

```
Batch
Filter
Transform
Attributes
Memory Limiter
Sampling
```

Example:

```
Telemetry
   |
   ↓
Batch
   |
   ↓
Filter
   |
   ↓
Export
```

---

# 53. Collector Exporters

Exporters send telemetry to backends.

Example:

```
OTel Collector
     |
     +------→ Jaeger
     |
     +------→ Metrics Backend
     |
     +------→ Other Supported Backend
```

---

# 54. Jaeger

Jaeger is used for distributed tracing.

It provides a way to:

```
Search Traces
Inspect Spans
Analyze Latency
View Service Dependencies
Investigate Errors
```

Architecture:

```
Application
    |
    ↓
OpenTelemetry
    |
    ↓
OTel Collector
    |
    ↓
Jaeger
    |
    ↓
Jaeger UI
```

---

# 55. Metrics vs Logs vs Traces

| Signal  | Main Purpose            | Example              |
| ------- | ----------------------- | -------------------- |
| Metrics | Measure system behavior | CPU = 80%            |
| Logs    | Record events           | Database timeout     |
| Traces  | Follow request flow     | Order → Payment → DB |

---

# 56. Metrics vs Logs

Metrics:

```
Compact
Numerical
Time-Series
Good for Dashboards
Good for Alerts
```

Logs:

```
Detailed
Event-Based
Text / Structured
Good for Investigation
```

Example:

```
Metric:
error_rate = 8%

Log:
payment-service database timeout
```

---

# 57. Logs vs Traces

Logs:

```
Detailed event information
```

Traces:

```
Distributed request flow
```

Example:

```
Log:
Payment timeout
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
4 seconds
```

The trace provides the context around where the delay occurred.

---

# 58. Metrics vs Traces

Metrics:

```
Error Rate
Request Rate
P95 Latency
```

Traces:

```
Individual Request
Service Flow
Span Duration
```

Metrics tell you:

```
"The service is slow."
```

Traces can tell you:

```
"The payment dependency caused the slow request."
```

---

# 59. Using All Three Together

Example problem:

```
Users report checkout failures.
```

Metrics:

```
checkout_error_rate = 8%
```

Logs:

```
payment-service:
external API timeout
```

Traces:

```
checkout
   |
   ↓
payment
   |
   ↓
external API
   |
   ↓
5 seconds
```

Conclusion:

```
External payment API is causing checkout failures.
```

---

# 60. Correlation Workflow

A useful troubleshooting workflow:

```
Metrics
   |
   ↓
Detect
   |
   ↓
Logs
   |
   ↓
Understand Error
   |
   ↓
Trace
   |
   ↓
Locate Dependency
   |
   ↓
Root Cause
```

---

# 61. Telemetry Context

Useful common attributes include:

```
service.name
service.version
environment
region
namespace
pod
container
host
trace_id
span_id
```

Example:

```
service.name = order-service
service.version = v2.1
environment = production
```

---

# 62. Service Name

Use consistent service names.

Example:

```
user-service
order-service
payment-service
inventory-service
```

Consistent naming makes cross-signal searching easier.

---

# 63. Environment

Always distinguish environments.

Example:

```
environment = development

environment = staging

environment = production
```

This prevents engineers from accidentally analyzing the wrong
environment.

---

# 64. Version

Include application version where practical.

Example:

```
service.version = v3.2
```

This allows investigation such as:

```
Version v3.1 → Normal

Version v3.2 → High Latency
```

---

# 65. Kubernetes Metadata

Useful Kubernetes context includes:

```
cluster
namespace
pod
container
node
deployment
```

Example:

```
cluster = production-eks
namespace = payments
pod = payment-service-7d9f
container = payment-service
```

---

# 66. Trace Context Propagation

When a request moves between services, trace context should be
propagated.

Example:

```
Service A
   |
   ↓
Trace Context
   |
   ↓
Service B
   |
   ↓
Trace Context
   |
   ↓
Service C
```

This allows all operations to belong to the same distributed trace.

---

# 67. Without Trace Context

Without propagation:

```
Service A
   |
   ↓
Log A

Service B
   |
   ↓
Log B

Service C
   |
   ↓
Log C
```

It can be difficult to determine which events belong to the same
request.

---

# 68. With Trace Context

With propagation:

```
Trace ID = abc123

Service A
   |
   ↓
Service B
   |
   ↓
Service C
```

All operations can be connected.

---

# 69. Metrics-to-Logs Correlation

Suppose Grafana shows:

```
payment-service
Error Rate = 7%
```

Engineer identifies:

```
service = payment-service
```

Then searches Kibana:

```
service:"payment-service"
AND level:"ERROR"
```

Result:

```
Database connection timeout
```

The metric identifies the affected service.

The logs provide the error details.

---

# 70. Logs-to-Traces Correlation

Suppose Kibana shows:

```
trace_id = abc123
error = payment timeout
```

Engineer searches Jaeger:

```
Trace ID = abc123
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
4.2 seconds
```

The trace identifies where the timeout occurred.

---

# 71. Metrics-to-Traces Correlation

Suppose Grafana shows:

```
P99 latency = 3 seconds
```

Engineer identifies:

```
payment-service
```

Then uses Jaeger to inspect slow traces.

Trace shows:

```
Payment
   |
   ↓
External API
   |
   ↓
2.8 seconds
```

This narrows the investigation.

---

# 72. Full Correlation

The ideal workflow:

```
Grafana
   |
   ↓
High Latency
   |
   ↓
Service Name
   |
   ↓
Kibana
   |
   ↓
Error Log
   |
   ↓
Trace ID
   |
   ↓
Jaeger
   |
   ↓
Slow Span
   |
   ↓
Dependency
   |
   ↓
Root Cause
```

---

# 73. Example: High CPU

Metrics:

```
CPU = 95%
```

Logs:

```
Application processing large batch
```

Traces:

```
Long-running requests
```

Conclusion:

```
High workload caused increased CPU.
```

Possible actions:

```
Scale
Optimize Application
Reduce Workload
Investigate Request Pattern
```

---

# 74. Example: OOMKilled

Metrics:

```
Memory increasing
```

Kubernetes:

```
Restart Count increasing
```

Events:

```
OOMKilled
```

Logs:

```
Application terminated
```

Traces:

```
Requests become slower before termination
```

Possible root cause:

```
Memory leak or excessive memory consumption.
```

---

# 75. Example: Database Latency

Metrics:

```
API P99 increased
```

Logs:

```
Database timeout
```

Traces:

```
API
  |
  ↓
Service
  |
  ↓
Database
  |
  ↓
4 seconds
```

Database metrics:

```
Query latency increased
```

Root cause:

```
Database performance degradation.
```

---

# 76. Example: External API Failure

Metrics:

```
Payment errors increased
```

Logs:

```
External payment timeout
```

Traces:

```
Payment Service
     |
     ↓
External API
     |
     ↓
5 seconds
```

Root cause:

```
External dependency failure or latency.
```

---

# 77. Metrics, Logs and Traces During Deployment

Before deployment:

```
Capture Baseline
```

During deployment:

```
Metrics
Logs
Traces
```

After deployment:

```
Compare
```

Example:

```
Version A

P95 = 250ms
Errors = 0.5%
```

After Version B:

```
P95 = 900ms
Errors = 5%
```

Tracing and logs can then identify why Version B is performing worse.

---

# 78. Canary Deployment Observability

Example:

```
Version A → 95%
Version B → 5%
```

Monitor Version B:

```
Request Rate
Error Rate
P95
P99
CPU
Memory
Logs
Traces
```

If healthy:

```
Increase traffic.
```

If unhealthy:

```
Rollback.
```

---

# 79. Observability and SLOs

Metrics can be used to calculate SLOs.

Example:

```
SLO = 99.9% availability
```

Metrics:

```
Successful Requests
Total Requests
```

Logs:

```
Failure Details
```

Traces:

```
Request-Level Investigation
```

Together they provide:

```
Measurement
Investigation
Root Cause
```

---

# 80. Observability and Error Budgets

If:

```
SLO = 99.9%
```

Then:

```
Error Budget = 0.1%
```

Metrics track budget consumption.

Logs and traces help explain why the budget was consumed.

---

# 81. Observability Cost

Telemetry volume can become very large.

Example:

```
500 Services
   |
   ↓
Millions of Metrics
   |
   ↓
Billions of Logs
   |
   ↓
Millions of Traces
```

Cost can grow rapidly.

Control:

```
Cardinality
Log Volume
Trace Sampling
Retention
Storage
```

---

# 82. Metric Cardinality Control

Avoid labels such as:

```
user_id
request_id
session_id
```

when they create excessive unique combinations.

Prefer dimensions such as:

```
service
method
status
environment
```

where appropriate.

---

# 83. Log Volume Control

Reduce unnecessary logs by:

```
Choosing appropriate log levels
Filtering repetitive messages
Removing unnecessary DEBUG logs
Aggregating events
Defining retention
```

---

# 84. Trace Sampling

Tracing can be sampled.

Example:

```
1,000,000 requests
      |
      ↓
   Sampling
      |
      ↓
  Selected Traces
```

A production strategy should preserve important requests where
possible.

Examples:

```
Errors
High-Latency Requests
Important Business Transactions
```

---

# 85. Data Retention Strategy

Different telemetry can have different retention.

Example:

```
Metrics
    |
    ↓
Long-Term Trends

Logs
    |
    ↓
Operational Investigation

Traces
    |
    ↓
Detailed Request Investigation
```

Retention should be based on:

```
Business Need
Cost
Compliance
Operational Requirements
```

---

# 86. Security of Telemetry

Telemetry can contain sensitive information.

Protect:

```
Logs
Metrics
Traces
Dashboards
Monitoring APIs
```

Use:

```
Authentication
Authorization
TLS
Least Privilege
Network Controls
```

---

# 87. Observability Platform

Our observability stack consists of:

```
Metrics:
    Prometheus

Visualization:
    Grafana

Logs:
    Elasticsearch
    Logstash
    Kibana

Telemetry:
    OpenTelemetry

Tracing:
    Jaeger
```

Architecture:

```
Applications
    |
    +--- Metrics → Prometheus → Grafana
    |
    +--- Logs → Logstash → Elasticsearch → Kibana
    |
    +--- Traces → OpenTelemetry → Collector → Jaeger
```

---

# 88. Real-World EKS Architecture

```
AWS
 |
 ↓
EKS
 |
 +-----------------------------+
 |                             |
 ↓                             ↓
Pods                         Nodes
 |
 ↓
Microservices
 |
 +----------------+----------------+
 |                |                |
 ↓                ↓                ↓
```

Metrics            Logs            Traces
|                |                |
↓                ↓                ↓
Prometheus          ELK        OpenTelemetry
|                               |
↓                               ↓
Grafana                          Collector
|
↓
Jaeger

---

# 89. Observability Architecture With Correlation

```
┌───────────────────────────────────────────┐
│              Applications                 │
└────────────────────┬──────────────────────┘
                     |
      +--------------+--------------+
      |              |              |
      ↓              ↓              ↓
   Metrics          Logs          Traces
      |              |              |
      ↓              ↓              ↓
 Prometheus        Logstash    OpenTelemetry
      |              |              |
      ↓              ↓              ↓
   Grafana      Elasticsearch     Collector
                     |              |
                     ↓              ↓
                   Kibana          Jaeger
      |              |              |
      +--------------+--------------+
                     |
                     ↓
                Correlation
                     |
                     ↓
              Root Cause Analysis
```

---

# 90. Practical Investigation

Problem:

```
Users report:
"The application is slow."
```

Step 1:

```
Check Metrics.

P95 = 1.5 seconds
```

Step 2:

```
Identify affected service.

order-service
```

Step 3:

```
Check Logs.

payment timeout
```

Step 4:

```
Extract Trace ID.

abc123
```

Step 5:

```
Search Jaeger.

order
  |
  ↓
payment
  |
  ↓
external API
  |
  ↓
1.2 seconds
```

Step 6:

```
Identify root cause.

External payment dependency is slow.
```

---

# 91. When to Use Metrics

Use metrics when you need:

```
Trends
Dashboards
Alerts
Capacity Planning
SLO Calculations
Aggregated System Health
```

Examples:

```
CPU
Memory
Error Rate
Request Rate
Latency
Queue Depth
```

---

# 92. When to Use Logs

Use logs when you need:

```
Detailed Events
Error Information
Debugging Context
Security Events
Configuration Information
```

Examples:

```
Database Error
Authentication Failure
Application Exception
Deployment Event
```

---

# 93. When to Use Traces

Use traces when you need:

```
Distributed Request Flow
Service Dependencies
Latency Breakdown
Request-Level Investigation
Dependency Analysis
```

Examples:

```
Order → Payment → Inventory → Database
```

---

# 94. Choosing the Correct Signal

Question:

```
"Is the system becoming slow?"
```

Use:

```
Metrics
```

Question:

```
"What error occurred?"
```

Use:

```
Logs
```

Question:

```
"Which service caused the latency?"
```

Use:

```
Traces
```

Question:

```
"Why is the system slow?"
```

Use:

```
Metrics + Logs + Traces
```

---

# 95. Signal Selection Example

Problem:

```
CPU is high.
```

Use:

```
Metrics
```

Problem:

```
Application throws exception.
```

Use:

```
Logs
```

Problem:

```
Microservice request is slow.
```

Use:

```
Traces
```

Problem:

```
Customer checkout fails intermittently.
```

Use:

```
Metrics + Logs + Traces
```

---

# 96. Best Practices

```
1. Use metrics for aggregated system health.

2. Use logs for detailed events.

3. Use traces for distributed requests.

4. Standardize service names.

5. Include environment metadata.

6. Include application version.

7. Preserve trace context.

8. Add trace IDs to logs where appropriate.

9. Avoid high-cardinality metric labels.

10. Use structured logs.

11. Protect sensitive telemetry.

12. Define retention policies.

13. Control log volume.

14. Use trace sampling appropriately.

15. Correlate metrics, logs, and traces.

16. Monitor the observability platform itself.
```

---

# 97. Common Mistakes

## Mistake 1

Collecting metrics without meaningful labels.

Result:

```
Limited Investigation
```

---

## Mistake 2

Using user_id as a metric label.

Result:

```
High Cardinality
```

---

## Mistake 3

Logging secrets.

Result:

```
Security Risk
```

---

## Mistake 4

Generating excessive logs.

Result:

```
High Cost
High Noise
```

---

## Mistake 5

Using traces without context propagation.

Result:

```
Broken Distributed Traces
```

---

## Mistake 6

Using metrics without logs.

Result:

```
Detection Without Context
```

---

## Mistake 7

Using logs without metrics.

Result:

```
Difficult Trend Analysis
```

---

## Mistake 8

Using traces without metrics.

Result:

```
Difficult to Know Which Requests Need Investigation
```

---

# 98. Production Checklist

```
[ ] Application metrics implemented
[ ] Infrastructure metrics implemented
[ ] Kubernetes metrics implemented
[ ] Prometheus configured
[ ] Grafana configured
[ ] Structured logging implemented
[ ] Centralized logging implemented
[ ] Elasticsearch configured
[ ] Logstash configured
[ ] Kibana configured
[ ] OpenTelemetry instrumentation configured
[ ] OTel Collector configured
[ ] Jaeger configured
[ ] Trace context propagated
[ ] Trace IDs available in logs
[ ] Service names standardized
[ ] Environment metadata available
[ ] Version metadata available
[ ] Metric cardinality reviewed
[ ] Log volume reviewed
[ ] Trace sampling reviewed
[ ] Retention policies defined
[ ] Security controls implemented
[ ] Metrics, logs, and traces correlated
[ ] Dashboards created
[ ] Alerts created
[ ] Runbooks created
```

---

# 99. Interview Questions

## What are the three pillars of observability?

### Answer

The three traditional pillars are:

```
Metrics
Logs
Traces
```

Metrics provide numerical measurements.

Logs provide detailed event information.

Traces show the flow of individual requests across distributed
services.

Together they provide a more complete view of system behavior.

---

# 100. What is the difference between metrics and logs?

### Answer

Metrics are numerical measurements collected over time.

Examples:

```
CPU
Request Rate
Error Rate
Latency
```

Logs are detailed records of events.

Examples:

```
Database timeout
Application exception
Authentication failure
```

Metrics are generally better for dashboards, trends, and alerts.

Logs are generally better for detailed investigation.

---

# 101. What is the difference between logs and traces?

### Answer

Logs represent individual events.

Traces represent the journey of a request across services.

Example:

```
Log:

payment-service database timeout

Trace:

Order
  ↓
Payment
  ↓
Database
  ↓
3 seconds
```

Logs provide event details.

Traces provide distributed request context.

---

# 102. What is the difference between metrics and traces?

### Answer

Metrics provide aggregated measurements.

Example:

```
P95 latency = 500ms
```

A trace provides request-level information.

Example:

```
Order
  |
  ↓
Payment
  |
  ↓
Database
```

Metrics tell us that latency is high.

Traces can help identify where the latency occurs.

---

# 103. What is a counter in Prometheus?

### Answer

A counter is a metric that generally increases over time.

Examples:

```
Total Requests
Total Errors
Total Jobs Processed
```

PromQL can calculate the rate of increase using functions such as:

```
rate()
```

---

# 104. What is a gauge?

### Answer

A gauge represents a value that can increase or decrease.

Examples:

```
CPU Usage
Memory Usage
Active Connections
Queue Depth
```

---

# 105. What is a histogram?

### Answer

A histogram measures observations and places them into buckets.

It is commonly used for:

```
Request Duration
Latency
Response Size
```

Histograms are useful for understanding distributions and calculating
percentiles.

---

# 106. What is high cardinality?

### Answer

High cardinality occurs when a metric has a very large number of unique
label combinations.

Examples of potentially high-cardinality labels:

```
user_id
request_id
session_id
```

High cardinality can significantly increase memory, storage, and query
costs.

---

# 107. Why use structured logs?

### Answer

Structured logs contain fields that can be searched and filtered.

Example:

```
{
  "service": "payment-service",
  "level": "ERROR",
  "status": 500
}
```

This is easier to analyze than unstructured text.

---

# 108. Why should Trace ID be included in logs?

### Answer

Trace ID allows engineers to connect log events to distributed traces.

Example:

```
Log
  |
  ↓
trace_id = abc123
  |
  ↓
Jaeger
  |
  ↓
Complete Request Trace
```

This improves troubleshooting and correlation.

---

# 109. What is OpenTelemetry?

### Answer

OpenTelemetry is a vendor-neutral observability framework for
generating, collecting, processing, and exporting telemetry.

It supports:

```
Metrics
Logs
Traces
```

---

# 110. What is the OpenTelemetry Collector?

### Answer

The OpenTelemetry Collector receives telemetry, processes it, and
exports it to supported backends.

Architecture:

```
Receiver
   ↓
Processor
   ↓
Exporter
```

It provides a central telemetry processing layer.

---

# 111. What is Jaeger?

### Answer

Jaeger is a distributed tracing platform.

It allows engineers to inspect:

```
Traces
Spans
Service Dependencies
Latency
Errors
```

It is particularly useful for microservices architectures.

---

# 112. How would you troubleshoot a high error rate?

### Answer

First check metrics:

```
Error Rate
Request Rate
Latency
```

Then identify the affected service.

Next check logs:

```
Application Errors
Dependency Errors
Exceptions
Timeouts
```

Then inspect traces:

```
Which span failed?
Which dependency failed?
Where did latency increase?
```

The flow is:

```
Metrics
   ↓
Logs
   ↓
Traces
   ↓
Root Cause
```

---

# 113. How would you troubleshoot high latency?

### Answer

I would:

```
1. Check P95/P99 latency.

2. Identify the affected service.

3. Check resource utilization.

4. Check application logs.

5. Inspect distributed traces.

6. Identify slow dependencies.

7. Check database performance.

8. Check external APIs.

9. Review recent deployments.

10. Mitigate and validate recovery.
```

---

# 114. How would you correlate metrics, logs and traces?

### Answer

I would use consistent metadata such as:

```
Service Name
Environment
Version
Trace ID
Span ID
```

Example:

```
Grafana
   |
   ↓
High Error Rate
   |
   ↓
payment-service
   |
   ↓
Kibana
   |
   ↓
trace_id
   |
   ↓
Jaeger
   |
   ↓
Failed Span
```

This allows engineers to move from high-level symptoms to detailed
request-level investigation.

---

# 115. Final Metrics, Logs and Traces Architecture

```
┌──────────────────────────────────────────────┐
│                Production                    │
│                                              │
│ Applications / Kubernetes / Infrastructure  │
└──────────────────────┬───────────────────────┘
                       |
          +------------+------------+
          |            |            |
          ↓            ↓            ↓
       Metrics        Logs        Traces
          |            |            |
          ↓            ↓            ↓
     Prometheus       ELK      OpenTelemetry
          |            |            |
          ↓            ↓            ↓
       Grafana   Elasticsearch    Collector
                       |             |
                       ↓             ↓
                     Kibana        Jaeger
          |            |             |
          +------------+-------------+
                       |
                       ↓
                  Correlation
                       |
                       ↓
                Investigation
                       |
                       ↓
                  Root Cause
                       |
                       ↓
                   Recovery
```

The three signals have different responsibilities:

```
Metrics
    ↓
Detect trends and abnormal behavior

Logs
    ↓
Provide detailed event context

Traces
    ↓
Show distributed request flow
```

The real value comes from combining them:

```
Metrics
   +
Logs
   +
Traces
   +
Context
   +
Correlation
   ↓
Complete Observability
```

In a production AWS/EKS microservices environment, this combination
provides the foundation for monitoring application health,
investigating incidents, understanding dependencies, analyzing
performance, and improving reliability.
