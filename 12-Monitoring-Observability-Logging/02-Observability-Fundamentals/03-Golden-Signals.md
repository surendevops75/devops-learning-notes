# Golden Signals

The Golden Signals are four key measurements that provide a practical
view of the health and performance of a production service.

They are:

```
Traffic
Latency
Errors
Saturation
```

These signals were popularized by the Google SRE approach and are widely
used for monitoring distributed systems, microservices, Kubernetes,
cloud platforms, and production applications.

A simple model is:

```
Traffic
   |
   ↓
Latency
   |
   ↓
Errors
   |
   ↓
Saturation
   |
   ↓
Service Health
```

The Golden Signals help engineers quickly answer:

```
Is the service receiving traffic?

Is the service responding quickly?

Are requests failing?

Is the service running out of capacity?
```

---

# 1. What Are the Golden Signals?

The four Golden Signals are:

## 1. Traffic

How much demand is the system receiving?

## 2. Latency

How long does the system take to respond?

## 3. Errors

How many requests are failing?

## 4. Saturation

How close is the system to its capacity?

Architecture:

```
┌───────────────────────┐
│    Golden Signals     │
└───────────┬───────────┘
            |
   +--------+--------+
   |        |        |
   ↓        ↓        ↓
Traffic  Latency   Errors
   |
   ↓
Saturation
```

All four signals should be considered together.

---

# 2. Why Golden Signals Matter

A production system can appear healthy from one perspective while
being unhealthy from another.

Example:

```
CPU = 40%
```

This does not automatically mean the application is healthy.

The service may have:

```
High Error Rate
High Latency
Database Saturation
Dependency Failure
```

Therefore, engineers should evaluate multiple signals.

Example:

```
Traffic       → Normal
Latency       → High
Errors        → High
Saturation    → Normal
```

This suggests the problem may not be local resource exhaustion.

---

# 3. Golden Signals and Observability

The Golden Signals are closely related to the three observability
signals.

```
Golden Signals
     |
     +--- Traffic
     +--- Latency
     +--- Errors
     +--- Saturation
             |
             ↓
      Metrics / Logs / Traces
```

Metrics usually provide the primary measurements for Golden Signals.

Logs provide detailed context.

Traces provide request-level investigation.

---

# 4. Traffic

Traffic measures how much demand is reaching a service.

Examples:

```
Requests per second
Requests per minute
Transactions per second
Messages per second
Jobs processed per minute
```

For an HTTP service:

```
Requests / second
```

may be the primary traffic metric.

For a message-processing service:

```
Messages / second
```

may be more appropriate.

---

# 5. Traffic Example

Suppose an order service normally receives:

```
500 requests/sec
```

Suddenly:

```
1,500 requests/sec
```

Traffic has increased significantly.

This can lead to:

```
Higher CPU
Higher Memory
Higher Database Connections
Higher Latency
Higher Error Rate
```

The traffic signal provides important context for interpreting the other
Golden Signals.

---

# 6. Traffic Does Not Mean Health

High traffic is not automatically a problem.

Example:

```
Traffic = 10,000 req/sec
Latency = 100ms
Errors = 0.1%
Saturation = 40%
```

The service may be operating perfectly.

The important question is:

```
Can the system handle the current traffic reliably?
```

---

# 7. Traffic Reduction Can Also Be Important

A sudden decrease in traffic may indicate:

```
Application Failure
Network Failure
Load Balancer Problem
DNS Problem
Authentication Failure
Client-Side Failure
```

Example:

```
Normal Traffic:
5,000 req/sec

Current:
500 req/sec
```

A dramatic drop may require investigation.

---

# 8. Traffic Metrics

Common traffic metrics include:

```
http_requests_total
requests_per_second
transactions_total
messages_received_total
jobs_processed_total
```

For microservices, useful dimensions may include:

```
service
method
route
status
environment
```

Be careful with high-cardinality labels.

---

# 9. Traffic by HTTP Method

Example:

```
GET     → 8,000 req/min
POST    → 3,000 req/min
PUT     → 500 req/min
DELETE  → 100 req/min
```

This can help identify workload patterns.

---

# 10. Traffic by Endpoint

Example:

```
/products     → 5,000 req/min
/orders       → 2,000 req/min
/payments     → 1,000 req/min
/inventory    → 500 req/min
```

This can help identify which operations generate the most load.

However, route labels should be normalized.

Avoid using arbitrary URL values such as:

```
/users/123456
/users/123457
/users/123458
```

as separate metric dimensions.

Prefer:

```
/users/{id}
```

where appropriate.

---

# 11. Traffic in Kubernetes

For Kubernetes workloads, traffic can be observed at multiple layers.

Example:

```
Internet
   |
   ↓
ALB
   |
   ↓
Kubernetes Service
   |
   ↓
Pods
```

Monitor traffic at:

```
Load Balancer
Ingress
Service
Application
Pod
```

This provides a layered view of demand.

---

# 12. Traffic in Microservices

A request may flow through multiple services.

Example:

```
User
  |
  ↓
Order Service
  |
  +------→ Payment Service
  |
  +------→ Inventory Service
```

Traffic should be considered at each important service.

Example:

```
Order Service:
5,000 req/sec

Payment Service:
1,500 req/sec

Inventory Service:
2,000 req/sec
```

Differences can reveal architectural behavior.

---

# 13. Traffic and Fan-Out

One request may generate multiple downstream requests.

Example:

```
1 Order Request
     |
     +----→ Payment
     |
     +----→ Inventory
     |
     +----→ Notification
```

One incoming request creates:

```
3 downstream operations
```

Therefore:

```
5,000 orders/sec
```

could produce:

```
15,000 downstream requests/sec
```

This is important when evaluating system capacity.

---

# 14. Latency

Latency measures how long an operation takes.

For HTTP applications:

```
Request
   |
   ↓
Processing
   |
   ↓
Response
```

Latency is the time between the request and the corresponding response.

Example:

```
Request → Response = 250ms
```

---

# 15. Why Latency Matters

High latency directly affects user experience.

Example:

```
100ms
  ↓
250ms
  ↓
500ms
  ↓
1s
  ↓
3s
```

As latency increases, users may experience:

```
Slow Pages
Slow APIs
Timeouts
Retries
Failed Transactions
```

---

# 16. Average Latency

Average latency is:

```
Total Request Duration
----------------------
   Total Requests
```

Example:

```
100 requests
```

Total duration:

```
20 seconds
```

Average:

```
20 / 100

= 200ms
```

Average latency is useful but can hide slow requests.

---

# 17. Why Average Can Be Misleading

Consider:

```
9 requests = 100ms
1 request  = 10 seconds
```

Average:

```
(9 × 100ms + 10,000ms) / 10

= 1,090ms
```

But most requests were actually very fast.

This is why percentile-based latency is often more useful.

---

# 18. Percentiles

Common latency percentiles:

```
P50
P90
P95
P99
```

They describe the distribution of request latency.

Example:

```
P50 = 100ms
P90 = 200ms
P95 = 300ms
P99 = 800ms
```

---

# 19. P50

P50 is the median.

Approximately:

```
50% of requests
are at or below this latency
```

Example:

```
P50 = 100ms
```

This represents typical request behavior.

---

# 20. P90

P90 means approximately:

```
90% of requests
are at or below this latency
```

Example:

```
P90 = 250ms
```

It provides visibility into slower requests.

---

# 21. P95

P95 means approximately:

```
95% of requests
are at or below this latency
```

Example:

```
P95 = 400ms
```

P95 is commonly useful for service-level performance monitoring.

---

# 22. P99

P99 means approximately:

```
99% of requests
are at or below this latency
```

Example:

```
P99 = 1.2 seconds
```

P99 is particularly useful for detecting tail latency.

---

# 23. Tail Latency

Tail latency refers to the slowest portion of requests.

Example:

```
P50 = 100ms
P95 = 300ms
P99 = 2s
```

The P99 value indicates that a small percentage of requests are much
slower than typical requests.

This can be caused by:

```
Database Queries
External APIs
Garbage Collection
CPU Contention
Network Problems
Lock Contention
```

---

# 24. Latency by Endpoint

Example:

```
/products
    P95 = 100ms

/orders
    P95 = 300ms

/payments
    P95 = 800ms
```

This allows engineers to identify slow operations.

---

# 25. Latency by Service

Example:

```
User Service:
P95 = 120ms

Order Service:
P95 = 300ms

Payment Service:
P95 = 900ms
```

Payment Service deserves further investigation.

---

# 26. Latency by Dependency

Example:

```
Order Service
     |
     +--- Database = 100ms
     |
     +--- Payment API = 800ms
     |
     +--- Inventory = 50ms
```

The payment dependency is the major contributor.

Distributed tracing can confirm this.

---

# 27. Latency and Traces

Suppose:

```
P99 = 2 seconds
```

A trace may show:

```
Order Service
    |
    | 100ms
    ↓
Payment Service
    |
    | 1.5s
    ↓
External Payment API
    |
    | 300ms
    ↓
Database
```

This identifies where the latency is occurring.

---

# 28. Latency Thresholds

Alert thresholds should be based on:

```
SLO
Historical Baseline
Business Requirements
User Experience
```

Example:

```
P95 > 500ms
for 10 minutes
```

may be appropriate for one service but not another.

Do not use identical thresholds for every application.

---

# 29. Errors

Errors measure unsuccessful operations.

Examples:

```
HTTP 5xx
Request Failures
Exceptions
Timeouts
Failed Transactions
Dependency Failures
```

Errors are one of the most important Golden Signals.

---

# 30. Error Rate

Error rate can be calculated as:

```
Failed Requests
---------------
Total Requests
```

Example:

```
Total Requests = 100,000

Failed Requests = 1,000

Error Rate = 1%
```

---

# 31. Error Rate Example

Suppose:

```
Requests = 10,000

Errors = 50
```

Then:

```
Error Rate =
50 / 10,000

= 0.5%
```

If errors increase:

```
0.5%
  ↓
1%
  ↓
3%
  ↓
8%
```

the service may be experiencing a significant incident.

---

# 32. HTTP Error Categories

Monitor:

```
2xx
3xx
4xx
5xx
```

Generally:

```
2xx → Successful
3xx → Redirect
4xx → Client-side request issue
5xx → Server-side failure
```

The exact interpretation depends on the application.

---

# 33. 4xx Errors

4xx errors may indicate:

```
Invalid Request
Unauthorized Request
Forbidden Request
Resource Not Found
Rate Limiting
```

Not every 4xx response represents a server incident.

For example:

```
404 /product/does-not-exist
```

may be expected behavior.

---

# 34. 5xx Errors

5xx errors generally indicate server-side failures.

Examples:

```
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

A sudden increase in 5xx responses is often a strong signal of service
degradation.

---

# 35. Application Errors

Not every failure is represented by an HTTP 5xx.

Applications may also experience:

```
Payment Failure
Database Failure
Queue Failure
Business Transaction Failure
Validation Failure
```

Therefore, application-specific error metrics are important.

---

# 36. Business Errors

Example:

```
Payment Success Rate

Total Payments = 10,000
Successful = 9,700
Failed = 300
```

Payment failure rate:

```
300 / 10,000

= 3%
```

The HTTP endpoint may still return:

```
HTTP 200
```

if the business transaction response is handled differently.

Therefore, business-level error signals can be valuable.

---

# 37. Error Budget

Errors consume the error budget associated with an SLO.

Example:

```
Availability SLO = 99.9%
```

Allowed failure:

```
0.1%
```

If error rate increases significantly, the error budget is consumed
faster.

---

# 38. Error Types

Errors can be categorized:

```
Application Errors
Infrastructure Errors
Dependency Errors
Network Errors
Database Errors
Authentication Errors
Authorization Errors
```

Classification helps troubleshooting.

---

# 39. Error by Service

Example:

```
User Service:
0.2%

Order Service:
0.8%

Payment Service:
7%

Inventory Service:
0.3%
```

Payment Service is clearly abnormal compared with the other services.

---

# 40. Error by Endpoint

Example:

```
/users
    0.1%

/orders
    0.5%

/payments
    8%
```

This helps isolate the failing operation.

---

# 41. Error by Dependency

Example:

```
Payment Service
     |
     +--- Database → 0.2%
     |
     +--- External API → 8%
     |
     +--- Cache → 0.1%
```

The external API may be the primary source of failure.

---

# 42. Errors and Logs

Metrics show:

```
Error Rate = 5%
```

Logs can show:

```
payment-service
ERROR
external payment API timeout
```

Metrics identify the scale.

Logs provide details.

---

# 43. Errors and Traces

Metrics:

```
Payment Error Rate = 5%
```

Logs:

```
External payment API timeout
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
  X
Timeout
```

Together they provide strong evidence of the failure path.

---

# 44. Saturation

Saturation measures how close a resource or service is to its capacity.

Examples:

```
CPU
Memory
Disk
Database Connections
Thread Pools
Connection Pools
Queue Capacity
```

Saturation is about capacity pressure.

---

# 45. CPU Saturation

Example:

```
CPU Utilization

40%
50%
60%
70%
85%
95%
```

High CPU may indicate:

```
Increased Traffic
Inefficient Code
CPU-Heavy Workload
Insufficient Capacity
```

---

# 46. Memory Saturation

Example:

```
Memory

50%
60%
70%
80%
90%
95%
```

High memory utilization can cause:

```
OOMKilled
Swapping
Application Slowdown
Pod Restarts
```

---

# 47. Disk Saturation

Monitor:

```
Disk Usage
Disk I/O
Disk Latency
Disk Queue
```

Example:

```
Disk Usage = 92%
```

This may eventually prevent:

```
Log Writes
Database Writes
Application Writes
```

---

# 48. Database Saturation

Database saturation may include:

```
Connection Pool Usage
Active Connections
CPU
Memory
IOPS
Storage
Query Queue
```

Example:

```
Max Connections = 500
Active Connections = 480
```

Connection capacity is almost exhausted.

---

# 49. Connection Pool Saturation

Example:

```
Maximum Connections = 100

Active = 95
```

Saturation:

```
95%
```

The application may start experiencing:

```
Connection Wait
Timeout
Increased Latency
Request Failure
```

---

# 50. Thread Pool Saturation

Example:

```
Thread Pool Size = 200

Active Threads = 195
```

The service may have very little remaining processing capacity.

This can lead to:

```
Request Queueing
Latency Increase
Timeouts
```

---

# 51. Queue Saturation

Example:

```
Queue Capacity = 10,000

Current Messages = 9,500
```

Saturation is high.

If producers continue faster than consumers:

```
Queue Depth
   |
   ↓
9,500
   ↓
10,000
   ↓
Capacity Exhausted
```

---

# 52. Kubernetes Saturation

Important Kubernetes saturation signals include:

```
Node CPU
Node Memory
Pod CPU
Pod Memory
Disk Pressure
Pod Density
Resource Requests
Resource Limits
```

Example:

```
Node
  |
  +--- CPU = 90%
  +--- Memory = 88%
  +--- Disk = 91%
```

This indicates capacity pressure.

---

# 53. Resource Requests and Limits

Kubernetes workloads should define appropriate:

```
CPU Requests
CPU Limits
Memory Requests
Memory Limits
```

Example:

```
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

Monitoring should verify whether these values are appropriate.

---

# 54. Saturation Is Not Just CPU

A common mistake is to define saturation only as CPU.

A service can have:

```
CPU = 40%
```

while:

```
Database Connections = 100%
Queue Depth = 95%
Thread Pool = 98%
```

Therefore, saturation should be monitored according to the service's
actual bottlenecks.

---

# 55. Finding the Bottleneck

Example:

```
CPU = 40%
Memory = 50%
DB Connections = 98%
```

The service is likely constrained by database connections rather than
CPU.

Another example:

```
CPU = 95%
Memory = 60%
DB Connections = 40%
```

CPU is likely the primary saturation point.

---

# 56. Golden Signals Relationship

The four signals influence each other.

Example:

```
Traffic ↑
   |
   ↓
Saturation ↑
   |
   ↓
Latency ↑
   |
   ↓
Errors ↑
```

This is a common failure pattern.

---

# 57. Example Production Scenario

Normal:

```
Traffic = 5,000 req/sec
P95 = 200ms
Errors = 0.2%
CPU = 50%
```

Incident:

```
Traffic = 12,000 req/sec
P95 = 1.2s
Errors = 5%
CPU = 95%
```

Interpretation:

```
Traffic increased significantly.

CPU became saturated.

Latency increased.

Errors increased.
```

This indicates capacity pressure caused by increased traffic.

---

# 58. Another Production Scenario

Normal:

```
Traffic = 5,000 req/sec
P95 = 200ms
Errors = 0.2%
CPU = 50%
```

Incident:

```
Traffic = 5,000 req/sec
P95 = 1.5s
Errors = 4%
CPU = 50%
```

Interpretation:

```
Traffic did not increase.

CPU is normal.

Latency increased.

Errors increased.
```

Potential causes:

```
Database
External Dependency
Network
Application Lock
Connection Pool
Configuration Change
```

Distributed tracing and logs can help identify the cause.

---

# 59. Dependency Failure Scenario

```
Traffic = Normal
CPU = Normal
Memory = Normal
```

But:

```
Latency = High
Errors = High
```

Logs:

```
Payment API Timeout
```

Traces:

```
Order
  |
  ↓
Payment
  |
  ↓
External API
  |
  X
Timeout
```

Conclusion:

```
Dependency failure.
```

---

# 60. Database Saturation Scenario

```
Traffic = Normal

CPU = 50%

Memory = 60%

Error Rate = 3%

P95 = 1.5s
```

Database:

```
Connections = 99%
```

The application may be healthy at the compute layer but saturated at
the database connection layer.

---

# 61. Queue Saturation Scenario

```
Producer Rate = 10,000 msg/sec

Consumer Rate = 7,000 msg/sec
```

Queue depth:

```
1,000
5,000
10,000
20,000
```

The queue is growing because consumers cannot keep up.

Possible actions:

```
Scale Consumers
Optimize Processing
Increase Capacity
Reduce Producer Load
```

---

# 62. Golden Signals Dashboard

A service dashboard should include:

```
Traffic
Latency
Errors
Saturation
```

Example:

```
┌────────────────────────────────────┐
│        ORDER SERVICE               │
├────────────────────────────────────┤
│ Traffic     │ 5,000 req/sec        │
│ P95         │ 250ms                │
│ P99         │ 600ms                │
│ Errors      │ 0.5%                 │
│ CPU         │ 55%                  │
│ Memory      │ 62%                  │
└────────────────────────────────────┘
```

This gives engineers a quick service-health view.

---

# 63. Golden Signals Dashboard in Grafana

A practical Grafana dashboard can have panels for:

```
Request Rate
Error Rate
P50
P95
P99
CPU
Memory
Restarts
Database Connections
Queue Depth
```

The dashboard should be focused rather than overloaded.

---

# 64. Traffic Panel

Example query concept:

```
rate(http_requests_total[5m])
```

Display:

```
Requests/sec
```

Useful dimensions:

```
service
method
route
```

---

# 65. Error Panel

Example query concept:

```
rate(http_requests_total{status=~"5.."}[5m])
```

This shows server-side HTTP error rate.

For an application-specific error metric:

```
rate(application_errors_total[5m])
```

Use the metric that best represents actual service health.

---

# 66. Latency Panel

For histogram-based latency:

```
histogram_quantile(
  0.95,
  rate(http_request_duration_seconds_bucket[5m])
)
```

This can provide P95 latency.

Similarly:

```
0.50 → P50

0.90 → P90

0.95 → P95

0.99 → P99
```

---

# 67. Saturation Panel

Possible measurements:

```
CPU Utilization
Memory Utilization
Disk Utilization
Connection Pool Usage
Queue Depth
```

The correct saturation signal depends on the application.

---

# 68. Golden Signals and Alerting

Alerts should be based on meaningful Golden Signal conditions.

Examples:

```
Error Rate > 5%

P95 Latency > SLO

CPU Saturation > 90%
for sustained period

Queue Depth continuously increasing
```

Avoid creating alerts for every small fluctuation.

---

# 69. Alert Example: Error Rate

Condition:

```
Error Rate > 5%
for 5 minutes
```

Possible alert:

```
HighErrorRate
```

Labels:

```
service
environment
severity
```

Annotations:

```
summary
description
runbook
```

---

# 70. Alert Example: Latency

Condition:

```
P95 Latency > 1 second
for 10 minutes
```

This may indicate:

```
Slow Database
External Dependency
Resource Saturation
Application Regression
```

Use traces and logs for investigation.

---

# 71. Alert Example: Saturation

Condition:

```
CPU > 90%
for 10 minutes
```

But consider whether CPU actually represents service saturation.

For some applications, better signals may be:

```
Connection Pool > 90%
```

or:

```
Queue Depth > Threshold
```

---

# 72. Alert Example: Queue Growth

Condition:

```
Queue depth continuously increasing
```

This can indicate:

```
Consumers cannot keep up
Consumer Failure
Increased Producer Traffic
Processing Bottleneck
```

---

# 73. Golden Signals and SLOs

Golden Signals support SLO measurement.

Example:

```
Availability SLO
    |
    ↓
Errors

Latency SLO
    |
    ↓
Latency
```

Traffic provides workload context.

Saturation provides capacity context.

---

# 74. Golden Signals and Error Budget

Suppose:

```
SLO = 99.9%
```

Error budget:

```
0.1%
```

If:

```
Error Rate = 5%
```

the service is consuming error budget very quickly.

Monitoring should show:

```
Current Error Rate
SLO
Error Budget Remaining
```

---

# 75. Golden Signals During Deployment

Before deployment:

```
Record Baseline
```

During deployment:

```
Monitor Golden Signals
```

After deployment:

```
Compare
```

Example:

```
Before:
P95 = 200ms
Errors = 0.2%

After:
P95 = 700ms
Errors = 3%
```

The deployment may have introduced a regression.

---

# 76. Golden Signals During Canary

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
P95 = 800ms
Error = 4%
```

Version B should not receive more traffic until the issue is
understood.

---

# 77. Golden Signals and Rollback

Deployment flow:

```
Deploy
  |
  ↓
Observe
  |
  ↓
Golden Signals
  |
  +---- Healthy
  |       |
  |       ↓
  |    Continue
  |
  └---- Unhealthy
          |
          ↓
       Rollback
```

---

# 78. Golden Signals for Microservices

Each critical service should have:

```
Traffic
Latency
Errors
Saturation
```

Example:

```
payment-service

Traffic:
2,000 req/sec

P95:
350ms

Error Rate:
0.5%

DB Connections:
70%
```

---

# 79. Golden Signals for APIs

For APIs:

```
Traffic
    ↓
Requests/sec

Latency
    ↓
P50/P95/P99

Errors
    ↓
4xx/5xx + Application Errors

Saturation
    ↓
CPU/Memory/Connections
```

---

# 80. Golden Signals for Databases

Databases are different from HTTP services.

Useful signals include:

```
Query Traffic
Query Latency
Query Errors
Connection Saturation
CPU
Memory
Storage
IOPS
```

The Golden Signal concept should be adapted to the system being
monitored.

---

# 81. Golden Signals for Message Queues

For queues:

```
Traffic
    ↓
Messages/sec

Latency
    ↓
Processing Latency

Errors
    ↓
Consumer Failures

Saturation
    ↓
Queue Depth / Consumer Capacity
```

---

# 82. Golden Signals for Kubernetes

For Kubernetes applications:

```
Traffic
    ↓
Requests/sec

Latency
    ↓
P95/P99

Errors
    ↓
Failed Requests / Restarts

Saturation
    ↓
CPU / Memory / Node Capacity
```

---

# 83. Golden Signals for Infrastructure

For infrastructure:

```
Traffic
    ↓
Network Traffic

Latency
    ↓
Network / Disk Latency

Errors
    ↓
Network Errors / Disk Errors

Saturation
    ↓
CPU / Memory / Disk / Network
```

---

# 84. Golden Signals and User Experience

The four signals should ultimately connect to user impact.

Example:

```
Traffic
   |
   ↓
Increased Load
   |
   ↓
Saturation
   |
   ↓
Latency
   |
   ↓
Errors
   |
   ↓
User Impact
```

This provides a useful production mental model.

---

# 85. Golden Signals and Business Transactions

Technical metrics alone may not show business impact.

Example:

```
Checkout Requests
   |
   ↓
Payment Success Rate
   |
   ↓
Order Completion Rate
```

A service may have:

```
HTTP 200 = 99.9%
```

but:

```
Payment Success = 95%
```

Therefore, business-specific signals should complement the Golden
Signals where necessary.

---

# 86. Combining Golden Signals With Business Metrics

Example:

```
Traffic
   |
   ↓
10,000 checkout requests

Latency
   |
   ↓
P95 = 300ms

Errors
   |
   ↓
HTTP Errors = 0.5%

Saturation
   |
   ↓
CPU = 60%
```

Business metric:

```
Payment Success = 94%
```

This reveals a problem that basic HTTP monitoring may miss.

---

# 87. Golden Signals and Dependencies

For a service:

```
Order Service
   |
   +--- Payment
   +--- Inventory
   +--- Database
```

Monitor the Golden Signals of the service and important dependencies.

Example:

```
Order:
P95 = 300ms

Payment:
P95 = 900ms
```

This suggests payment may contribute to order latency.

---

# 88. Dependency Golden Signals

For an external API:

```
Traffic
Latency
Errors
Saturation / Rate Limit
```

Example:

```
Requests = 2,000/sec
P95 = 800ms
Errors = 5%
Rate Limit = 95%
```

This can explain downstream application problems.

---

# 89. Golden Signals and Retries

Retries can increase traffic.

Example:

```
Original Requests = 1,000/sec
```

Failures cause retries:

```
Retry Requests = 500/sec
```

Total:

```
1,500 requests/sec
```

This increases traffic and may increase saturation.

The failure loop can become:

```
Errors
  ↓
Retries
  ↓
Traffic
  ↓
Saturation
  ↓
Latency
  ↓
More Errors
```

This is a dangerous production pattern.

---

# 90. Retry Storm

A retry storm occurs when many clients repeatedly retry failed
requests.

Example:

```
Dependency Failure
      |
      ↓
Requests Fail
      |
      ↓
Clients Retry
      |
      ↓
Traffic Increases
      |
      ↓
Service Saturates
      |
      ↓
More Requests Fail
```

Mitigation can include:

```
Exponential Backoff
Jitter
Retry Limits
Circuit Breakers
Rate Limiting
```

---

# 91. Golden Signals and Autoscaling

Golden Signals can provide context for autoscaling.

Example:

```
Traffic ↑
   |
   ↓
CPU ↑
   |
   ↓
HPA Scale Out
   |
   ↓
CPU ↓
   |
   ↓
Latency ↓
```

But autoscaling should not be based blindly on CPU alone.

Application-specific metrics may be more useful.

---

# 92. Golden Signals and HPA

Kubernetes HPA may use:

```
CPU
Memory
Custom Metrics
```

Example:

```
Request Rate ↑
   |
   ↓
Application Load ↑
   |
   ↓
HPA
   |
   ↓
Replicas ↑
```

Monitor:

```
Desired Replicas
Current Replicas
CPU
Request Rate
Latency
```

---

# 93. Saturation Before Failure

A good monitoring strategy should detect saturation before complete
failure.

Example:

```
60% → Healthy
70% → Normal
80% → Warning
90% → High
100% → Exhausted
```

Early detection provides time to:

```
Scale
Optimize
Reduce Load
Increase Capacity
```

---

# 94. Baseline-Based Monitoring

Golden Signal thresholds should consider normal behavior.

Example:

```
Normal Traffic:
5,000 req/sec

Current:
6,000 req/sec
```

This may be normal.

But:

```
Normal:
5,000 req/sec

Current:
50,000 req/sec
```

is a major deviation.

Historical baselines help interpret current values.

---

# 95. Dynamic Workloads

Cloud-native workloads change continuously.

Examples:

```
Traffic Spikes
Autoscaling
Deployments
Batch Jobs
Scheduled Jobs
```

Therefore, static thresholds may not always be sufficient.

Use:

```
Baselines
SLOs
Rate-of-Change
Sustained Conditions
```

where appropriate.

---

# 96. Rate of Change

Sometimes the rate of change is more important than the absolute
value.

Example:

```
Error Rate:
0.1%
0.2%
0.5%
2%
8%
```

The rapid increase is significant.

Similarly:

```
Queue Depth:
100
500
2,000
10,000
```

The growth trend indicates the system is falling behind.

---

# 97. Sustained Conditions

Avoid alerting on every short spike.

Example:

```
CPU = 95%
for 10 seconds
```

may not require immediate action.

But:

```
CPU = 95%
for 15 minutes
```

may indicate sustained saturation.

Alert duration should match the system's behavior.

---

# 98. Golden Signals and Alert Fatigue

Poor configuration:

```
Alert on every CPU spike
Alert on every 4xx
Alert on every latency spike
```

Result:

```
Too Many Alerts
```

Better:

```
Alert on meaningful service impact.
```

Example:

```
P95 latency above SLO
for sustained period
```

---

# 99. Golden Signals and Incident Response

During an incident:

```
1. Check Traffic

2. Check Latency

3. Check Errors

4. Check Saturation
```

Then:

```
5. Check Logs

6. Check Traces

7. Check Dependencies

8. Check Recent Changes

9. Mitigate

10. Validate
```

---

# 100. Practical Incident Example

Incident:

```
Users report:
"Orders are failing."
```

Step 1:

```
Traffic = Normal
```

Step 2:

```
Latency = Increased
```

Step 3:

```
Errors = 7%
```

Step 4:

```
CPU = 45%
```

Conclusion:

```
Not obviously CPU saturation.
```

Step 5:

```
Check logs.
```

Result:

```
Payment timeout.
```

Step 6:

```
Check traces.
```

Result:

```
Payment dependency is taking 4 seconds.
```

Step 7:

```
Check dependency.
```

Result:

```
External payment API degraded.
```

Root cause:

```
External dependency failure.
```

---

# 101. Another Incident Example

Incident:

```
API is slow.
```

Golden Signals:

```
Traffic = 3x normal
Latency = High
Errors = Increasing
CPU = 95%
```

Interpretation:

```
Increased traffic is causing CPU saturation.
```

Potential mitigation:

```
Scale out
```

Then validate:

```
CPU ↓
Latency ↓
Errors ↓
```

---

# 102. Another Incident Example

Incident:

```
API is slow.
```

Golden Signals:

```
Traffic = Normal
Latency = High
Errors = Normal
CPU = Normal
```

Possible causes:

```
Database Latency
External API
Network
Lock Contention
Connection Pool
```

Next:

```
Use traces to identify slow spans.
```

---

# 103. Another Incident Example

Incident:

```
Checkout failures.
```

Golden Signals:

```
Traffic = Normal
Latency = Normal
Errors = Normal
```

But:

```
Payment Success Rate = 90%
```

This demonstrates why business metrics should supplement Golden Signals.

---

# 104. Dashboard Design

A good Golden Signals dashboard should prioritize:

```
1. Traffic
2. Error Rate
3. Latency
4. Saturation
```

Then provide supporting information:

```
Dependencies
Logs
Traces
Deployments
Business Metrics
```

---

# 105. Recommended Service Dashboard

```
┌─────────────────────────────────────────────┐
│              SERVICE HEALTH                 │
├─────────────────────────────────────────────┤
│ Traffic     │ 5,000 req/sec                 │
│ Error Rate  │ 0.5%                          │
│ P50         │ 100ms                         │
│ P95         │ 250ms                         │
│ P99         │ 600ms                         │
│ CPU         │ 55%                           │
│ Memory      │ 60%                           │
│ Connections │ 65%                           │
└─────────────────────────────────────────────┘
```

---

# 106. Golden Signals Dashboard Sections

## Traffic

```
Request Rate
Transaction Rate
Message Rate
```

## Latency

```
P50
P95
P99
```

## Errors

```
Error Rate
4xx
5xx
Application Errors
```

## Saturation

```
CPU
Memory
Connections
Queue Depth
```

---

# 107. Prometheus Metrics

Example metrics:

```
http_requests_total

http_request_duration_seconds_bucket

process_cpu_seconds_total

process_resident_memory_bytes

database_connections_active
```

Application-specific metrics should also be defined.

---

# 108. PromQL Traffic

Example:

```
rate(
  http_requests_total[5m]
)
```

This estimates requests per second over the selected window.

---

# 109. PromQL Error Rate

Example:

```
sum(
  rate(http_requests_total{status=~"5.."}[5m])
)
/
sum(
  rate(http_requests_total[5m])
)
```

This provides the proportion of 5xx responses.

The exact query should be adapted to the application's metric
structure.

---

# 110. PromQL P95 Latency

For a histogram:

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

For production dashboards, the query should usually be grouped by
appropriate service or route dimensions.

---

# 111. PromQL Saturation

Example CPU utilization concept:

```
1 -
avg(
  rate(node_cpu_seconds_total{
    mode="idle"
  }[5m])
)
```

This can help estimate CPU utilization.

The exact query depends on the metric source and infrastructure
architecture.

---

# 112. Alerting Strategy

Golden Signal alerts should focus on service impact.

Example:

```
HighErrorRate
```

Condition:

```
Error Rate > 5%
for 5 minutes
```

Another:

```
HighLatency
```

Condition:

```
P95 > SLO
for 10 minutes
```

Another:

```
HighSaturation
```

Condition:

```
Resource saturation remains high
for sustained duration
```

---

# 113. Golden Signals and Runbooks

Each critical alert should have a runbook.

Example:

```
Alert:
HighLatency
```

Runbook:

```
1. Check Traffic
2. Check Error Rate
3. Check Saturation
4. Check Recent Deployment
5. Check Logs
6. Check Traces
7. Check Dependencies
8. Mitigate
9. Validate
```

---

# 114. Golden Signals and Recent Deployments

During an incident, check whether the Golden Signals changed after a
release.

Example:

```
Deployment
    |
    ↓
Version v2.5
    |
    ↓
P95 increases
    |
    ↓
Error rate increases
```

This may indicate a deployment regression.

---

# 115. Golden Signals and Rollback

If the deployment is confirmed as the cause:

```
Current Version
      |
      ↓
Golden Signals Degraded
      |
      ↓
   Rollback
      |
      ↓
Golden Signals Recover
      |
      ↓
  Validate
```

Rollback should be based on evidence and the organization's deployment
procedures.

---

# 116. Golden Signals and Change Correlation

Useful changes to correlate include:

```
Application Deployment
Configuration Change
Infrastructure Change
Database Change
Scaling Event
Network Change
```

A timeline can help:

```
10:00 → Deployment
10:03 → Latency increases
10:04 → Errors increase
10:05 → Alert fires
```

This is strong evidence for investigation.

---

# 117. Golden Signals and Observability Correlation

The complete workflow:

```
Golden Signal
     |
     ↓
Identify Problem
     |
     ↓
Metrics
     |
     ↓
Logs
     |
     ↓
Trace
     |
     ↓
Dependency
     |
     ↓
Root Cause
```

---

# 118. Golden Signals and OpenTelemetry

OpenTelemetry can provide telemetry supporting Golden Signal
measurements.

For example:

```
Application
    |
    ↓
OpenTelemetry
    |
    +--- Request Metrics
    +--- Request Traces
    +--- Context
    |
    ↓
Collector
```

The metrics backend can be used for dashboards and alerts, while
traces provide detailed investigation.

---

# 119. Golden Signals and Jaeger

Jaeger primarily supports the tracing side of the investigation.

Example:

```
Golden Signal:
P99 latency increased

    ↓

Jaeger:
Inspect slow traces

    ↓

Slow Span:
Payment Service

    ↓

Dependency:
External Payment API
```

This provides request-level context.

---

# 120. Golden Signals and ELK

ELK supports the detailed log investigation.

Example:

```
Error Signal:
Error Rate = 8%

    ↓

Kibana

    ↓

Search:
service="payment-service"
level="ERROR"

    ↓

Result:
External API timeout
```

This complements metrics and traces.

---

# 121. Complete Golden Signals Architecture

```
Applications
      |
      ↓
+-----------------------+
|    Golden Signals     |
+-----------------------+
      |
 +----+----+----+
 |         |    |
 ↓         ↓    ↓
```

Traffic   Latency Errors
|         |    |
+---------+----+
|
↓
Saturation
|
↓
Metrics
|
↓
Prometheus
|
↓
Grafana

```
Detailed Investigation:

Logs
  |
  ↓
ELK
  |
  ↓
Kibana

Traces
  |
  ↓
OpenTelemetry
  |
  ↓
OTel Collector
  |
  ↓
Jaeger
```

---

# 122. Golden Signals in an EKS Environment

```
Users
  |
  ↓
ALB
  |
  ↓
EKS
  |
  +------------------------------+
  |                              |
  ↓                              ↓
Services                      Nodes
  |
  +----------+----------+
  |          |          |
  ↓          ↓          ↓
```

Traffic    Latency     Errors
|          |          |
+----------+----------+
|
↓
Saturation
|
↓
Prometheus
|
↓
Grafana

```
Logs → ELK

Traces → OpenTelemetry → Collector → Jaeger
```

---

# 123. Golden Signals for Production Readiness

Before deploying a critical service, verify:

```
[ ] Traffic metric exists
[ ] Latency metric exists
[ ] Error metric exists
[ ] Saturation metrics exist
[ ] P50 available
[ ] P95 available
[ ] P99 available
[ ] Dashboards created
[ ] Alerts configured
[ ] SLO defined
[ ] Runbook created
[ ] Logs available
[ ] Trace instrumentation available
[ ] Dependency monitoring available
[ ] Deployment correlation available
```

---

# 124. Common Golden Signal Mistakes

## Mistake 1: Monitoring Only CPU

Problem:

```
Application may be slow for reasons unrelated to CPU.
```

Better:

```
Monitor all four Golden Signals.
```

---

## Mistake 2: Using Only Average Latency

Problem:

```
Tail latency can remain hidden.
```

Better:

```
Monitor P50, P95 and P99 where appropriate.
```

---

## Mistake 3: Alerting on Every 4xx

Problem:

```
Many 4xx responses may be expected.
```

Better:

```
Identify meaningful application error conditions.
```

---

## Mistake 4: Ignoring Traffic Drops

Problem:

```
A sudden traffic decrease can indicate a serious issue.
```

Better:

```
Monitor both increases and decreases in traffic.
```

---

## Mistake 5: Defining Saturation Only as CPU

Problem:

```
Database connections or queues may saturate first.
```

Better:

```
Monitor actual service bottlenecks.
```

---

## Mistake 6: Ignoring Dependencies

Problem:

```
A service may be healthy locally but failing because of a
dependency.
```

Better:

```
Monitor dependency latency and errors.
```

---

## Mistake 7: Static Thresholds Everywhere

Problem:

```
Different services have different normal behavior.
```

Better:

```
Use SLOs, baselines, sustained conditions and service-specific
thresholds.
```

---

## Mistake 8: No Business Metrics

Problem:

```
Technical health may not reflect business health.
```

Better:

```
Add business-critical signals such as payment success rate or
order completion rate.
```

---

# 125. Golden Signals Best Practices

```
1. Monitor Traffic.

2. Monitor Latency.

3. Monitor Errors.

4. Monitor Saturation.

5. Use percentiles for latency.

6. Monitor application-specific errors.

7. Monitor business-critical transactions.

8. Monitor dependencies.

9. Monitor both traffic increases and decreases.

10. Identify actual saturation points.

11. Use SLOs to define important thresholds.

12. Avoid noisy alerts.

13. Use sustained evaluation windows.

14. Correlate metrics with logs.

15. Correlate logs with traces.

16. Monitor deployments and configuration changes.

17. Create focused dashboards.

18. Create runbooks for critical alerts.

19. Review Golden Signal behavior after incidents.

20. Continuously improve monitoring.
```

---

# 126. Interview Questions

## What are the four Golden Signals?

### Answer

The four Golden Signals are:

```
Traffic
Latency
Errors
Saturation
```

Traffic measures demand.

Latency measures response time.

Errors measure unsuccessful operations.

Saturation measures how close the system is to capacity.

Together they provide a practical view of service health.

---

# 127. Why are Golden Signals important?

### Answer

They provide a simple framework for understanding production service
health.

Instead of monitoring hundreds of unrelated metrics, engineers can
first evaluate:

```
Traffic
Latency
Errors
Saturation
```

Then use logs and traces to investigate the cause of abnormalities.

---

# 128. What is the difference between P50, P95 and P99?

### Answer

P50 is the median.

Approximately 50% of requests are at or below the P50 value.

P95 means approximately 95% of requests are at or below that latency.

P99 means approximately 99% of requests are at or below that latency.

P95 and P99 are useful for understanding slower and tail requests.

---

# 129. Why is average latency not enough?

### Answer

Average latency can hide tail latency.

For example, most requests may be fast while a small percentage are
extremely slow.

P95 and P99 provide additional visibility into these slower requests.

---

# 130. What is saturation?

### Answer

Saturation represents how close a system or resource is to its
capacity.

Examples:

```
CPU
Memory
Database Connections
Thread Pools
Queue Capacity
Disk
```

The correct saturation metric depends on the service.

---

# 131. Can a service have low CPU but high saturation?

### Answer

Yes.

For example:

```
CPU = 40%
```

but:

```
Database Connections = 100%
```

The application can still be saturated because database connection
capacity is exhausted.

Saturation should be measured according to the actual bottleneck.

---

# 132. How would you troubleshoot high latency using Golden Signals?

### Answer

I would first check:

```
Traffic
Latency
Errors
Saturation
```

Then determine whether the latency is associated with:

```
Increased Traffic
Resource Saturation
Database
External Dependency
Network
Application
```

Next I would use:

```
Logs
```

and:

```
Distributed Traces
```

to identify the slow component.

---

# 133. How would you troubleshoot a sudden increase in errors?

### Answer

First check the error rate and traffic.

Then determine whether errors correlate with:

```
Increased Traffic
Saturation
Deployment
Configuration Change
Dependency Failure
```

Next:

```
Check Logs
Check Traces
Check Dependencies
```

Finally:

```
Mitigate
Validate
Document
```

---

# 134. How would you monitor a microservice?

### Answer

I would monitor:

```
Traffic
Latency
Errors
Saturation
```

Then add:

```
Structured Logs
Distributed Traces
Dependency Metrics
Business Metrics
```

For the platform:

```
Prometheus
Grafana
ELK
OpenTelemetry
Jaeger
```

---

# 135. How would you design a Golden Signals dashboard?

### Answer

I would create a focused dashboard containing:

```
Request Rate
Error Rate
P50
P95
P99
CPU
Memory
Connection Usage
Queue Depth
```

Then include links or correlation to:

```
Logs
Traces
Deployments
Dependencies
```

The dashboard should help engineers quickly determine whether the
service is healthy.

---

# 136. How do Golden Signals help during deployment?

### Answer

I would capture a baseline before deployment.

Then monitor:

```
Traffic
Latency
Errors
Saturation
```

during and after the deployment.

If the new version causes significant degradation, I would investigate
and roll back according to the deployment strategy.

---

# 137. How do Golden Signals support SLOs?

### Answer

Errors can be used to measure availability.

Latency can be used to measure performance SLOs.

Traffic provides workload context.

Saturation provides capacity context.

Together they provide the telemetry required to evaluate service
reliability.

---

# 138. How do you avoid Golden Signal alert fatigue?

### Answer

I would:

```
Use SLO-based thresholds
Use meaningful evaluation windows
Avoid alerts for every fluctuation
Alert on user-impacting conditions
Use service-specific thresholds
Route alerts correctly
Review noisy alerts regularly
```

The goal is actionable alerts rather than maximum alert volume.

---

# 139. Golden Signals Troubleshooting Flow

A practical production workflow is:

```
Alert
  |
  ↓
Traffic
  |
  ↓
Latency
  |
  ↓
Errors
  |
  ↓
Saturation
  |
  ↓
Recent Changes
  |
  ↓
Logs
  |
  ↓
Traces
  |
  ↓
Dependencies
  |
  ↓
Root Cause
  |
  ↓
Mitigation
  |
  ↓
Validation
```

---

# 140. Final Golden Signals Model

```
┌─────────────────────────────────────────────┐
│                SERVICE                     │
└──────────────────────┬──────────────────────┘
                       |
          +------------+------------+
          |            |            |
          ↓            ↓            ↓
       Traffic      Latency       Errors
          |            |            |
          +------------+------------+
                       |
                       ↓
                   Saturation
                       |
                       ↓
                    Metrics
                       |
                       ↓
                  Prometheus
                       |
                       ↓
                     Grafana
                       |
          +------------+------------+
          |                         |
          ↓                         ↓
        Logs                      Traces
          |                         |
          ↓                         ↓
         ELK                  OpenTelemetry
          |                         |
          ↓                         ↓
        Kibana                  Collector
                                    |
                                    ↓
                                  Jaeger
                                    |
                                    ↓
                              Root Cause
```

The Golden Signals provide the first layer of production visibility:

```
Traffic
    ↓
Is the system receiving the expected workload?

Latency
    ↓
Is the system responding within acceptable time?

Errors
    ↓
Are requests or transactions failing?

Saturation
    ↓
Is the system approaching a capacity limit?
```

When combined with:

```
Metrics
Logs
Traces
SLOs
Dependency Monitoring
Business Metrics
```

they provide a powerful foundation for operating production
microservices, Kubernetes clusters, and cloud-native platforms.

The practical troubleshooting model is:

```
Golden Signals
      ↓
Identify the symptom
      ↓
Metrics
      ↓
Logs
      ↓
Traces
      ↓
Dependency
      ↓
Root Cause
      ↓
Mitigation
      ↓
Validation
      ↓
Continuous Improvement
```
