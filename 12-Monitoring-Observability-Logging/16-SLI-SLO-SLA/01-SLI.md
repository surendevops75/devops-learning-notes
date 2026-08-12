# SLI — Service Level Indicator

## 1. Overview

SLI stands for **Service Level Indicator**.

An SLI is a quantitative measurement that tells us how well a service is performing from the user's or customer's perspective.

In simple terms:

```text
SLI = What we measure
```

Examples:

```text
Availability
Latency
Error Rate
Throughput
Success Rate
Durability
```

An SLI provides the actual observed performance of a service.

---

# 2. SLI vs SLO vs SLA

These three concepts are closely related but different.

```text
SLI
 ↓
What are we measuring?

SLO
 ↓
What level of performance do we want?

SLA
 ↓
What level of service do we promise?
```

Example:

```text
SLI = Availability

SLO = 99.9% availability

SLA = 99.5% availability with contractual commitment
```

---

# 3. Why SLIs Are Important

Traditional monitoring often focuses on infrastructure:

```text
CPU
Memory
Disk
Network
```

These metrics are useful, but they do not always tell us whether users are receiving a good service.

Example:

```text
CPU = 40%
Memory = 50%
Nodes = Healthy
```

Everything looks healthy.

But:

```text
HTTP Requests
↓
20% failing
```

Customers are experiencing a problem.

Therefore we need measurements that represent actual service behavior.

---

# 4. Infrastructure Metrics vs Service Indicators

Infrastructure metric:

```text
CPU = 80%
```

Service indicator:

```text
Successful Requests / Total Requests
= 99.2%
```

The second measurement is closer to user experience.

This leads to an important principle:

```text
System Health
≠
User Experience
```

A service can have healthy infrastructure while still providing poor user experience.

---

# 5. Common SLI Categories

Common SLIs include:

```text
Availability
Latency
Error Rate
Throughput
Correctness
Durability
Freshness
```

The correct SLI depends on the type of service.

---

# 6. Availability SLI

Availability measures whether a service is successfully serving requests.

A common formula is:

```text
Availability =
Successful Requests
-------------------
Total Valid Requests
```

Example:

```text
Total Requests = 1,000,000

Successful Requests = 999,000
```

Therefore:

```text
Availability = 99.9%
```

---

# 7. Availability Example

Suppose an API receives:

```text
100,000 requests
```

and:

```text
99,800 requests succeed
```

Then:

```text
Availability =
99,800 / 100,000

= 99.8%
```

This gives us the observed service performance.

The SLI itself is:

```text
99.8%
```

The target comes later when we define the SLO.

---

# 8. Request-Based Availability

For APIs, availability can be measured using successful HTTP responses.

Example:

```text
Total Requests
        ↓
200 / 300 / 400 / 500
```

Depending on the service definition, successful responses might be:

```text
2xx
3xx
```

while server failures might include:

```text
5xx
```

The exact definition should be chosen according to the service's expected behavior.

---

# 9. Error Rate SLI

Error rate measures the percentage of requests that fail.

Formula:

```text
Error Rate =
Failed Requests
--------------
Total Requests
```

Example:

```text
Failed Requests = 500
Total Requests = 100,000
```

Therefore:

```text
Error Rate = 0.5%
```

Availability based on the same request population would be:

```text
99.5%
```

---

# 10. Success Rate SLI

Success rate is the inverse perspective of error rate.

```text
Success Rate =
Successful Requests
-------------------
Total Requests
```

Example:

```text
Successful = 99,500
Total = 100,000
```

Then:

```text
Success Rate = 99.5%
```

For many services:

```text
Success Rate
```

can be used as the primary availability SLI.

---

# 11. Latency SLI

Latency measures how quickly a service responds.

Example:

```text
Request
   ↓
Application
   ↓
Response
```

If the request takes:

```text
200 ms
```

then:

```text
Latency = 200 ms
```

But measuring only average latency can hide important problems.

---

# 12. Average Latency Problem

Suppose we have:

```text
99 requests = 100 ms
1 request  = 10 seconds
```

The average may look acceptable.

But one user experienced:

```text
10 seconds
```

Therefore average latency alone is often insufficient for user-facing services.

---

# 13. Percentile Latency

Common latency percentiles:

```text
p50
p90
p95
p99
```

Example:

```text
p50 = 100 ms
p95 = 300 ms
p99 = 800 ms
```

This tells us how latency is distributed.

---

# 14. p50 Latency

p50 represents the median.

Approximately:

```text
50% of requests
```

are faster than or equal to the p50 value.

Example:

```text
p50 = 120 ms
```

This represents typical request latency better than an extreme tail.

---

# 15. p95 Latency

p95 represents the 95th percentile.

Example:

```text
p95 = 400 ms
```

Approximately:

```text
95% of requests
```

are at or below 400 ms.

The remaining:

```text
5%
```

are slower.

p95 is often useful for understanding broader user experience.

---

# 16. p99 Latency

p99 focuses on the tail.

Example:

```text
p99 = 1 second
```

Approximately:

```text
99% of requests
```

are at or below one second.

The slowest:

```text
1%
```

are above that value.

Tail latency can be particularly important for high-scale services.

---

# 17. Why Latency Percentiles Matter

Consider:

```text
p50 = 100 ms
p95 = 300 ms
p99 = 5 seconds
```

Most requests are fast.

But:

```text
1% of requests
```

are extremely slow.

For a service receiving:

```text
10 million requests
```

1% represents:

```text
100,000 requests
```

So a small percentage can still represent a large number of affected users.

---

# 18. Throughput SLI

Throughput measures how much work a system processes.

Examples:

```text
Requests per second
Messages per second
Transactions per second
Orders per minute
```

Example:

```text
Requests = 10,000 / second
```

The SLI can be:

```text
Request Throughput
```

---

# 19. Throughput Example

Suppose a payment system normally processes:

```text
5,000 transactions/minute
```

During an incident:

```text
1,000 transactions/minute
```

Throughput has dropped significantly.

However, throughput by itself does not necessarily mean the service is unhealthy.

It must be interpreted with:

```text
Traffic
Demand
Errors
Latency
```

---

# 20. Correctness SLI

Some systems need to measure whether responses are correct.

Example:

```text
Search Service
```

A request may return:

```text
HTTP 200
```

but produce incorrect results.

Therefore:

```text
HTTP Success
≠
Business Correctness
```

A correctness SLI may measure:

```text
Correct Responses
-----------------
Total Responses
```

---

# 21. Freshness SLI

For data-processing systems, freshness can be an important SLI.

Example:

```text
Data Pipeline
```

Expected:

```text
Data less than 5 minutes old
```

Observed:

```text
Data is 20 minutes old
```

The freshness SLI could measure:

```text
Age of Data
```

or:

```text
Percentage of Data
within freshness requirement
```

---

# 22. Durability SLI

For storage systems, durability may be important.

The objective is to measure:

```text
Probability that stored data remains intact and retrievable.
```

Examples:

```text
Successful Data Writes
Successful Data Reads
Data Integrity
Recovery Success
```

The exact SLI depends on the storage system and business requirement.

---

# 23. Queue Processing SLI

For asynchronous systems, request latency may not be the best SLI.

Consider:

```text
Producer
   ↓
Queue
   ↓
Consumer
```

Useful SLIs include:

```text
Message Processing Success
Queue Processing Latency
Message Age
Consumer Throughput
Backlog
```

Example:

```text
95% of messages processed within 30 seconds
```

This can represent user-visible service quality better than CPU usage.

---

# 24. Database SLI

For a database-backed service, possible SLIs include:

```text
Query Success Rate
Query Latency
Transaction Success Rate
Connection Availability
Read Success
Write Success
```

Example:

```text
Successful Queries
------------------
Total Queries
```

This is more meaningful than simply saying:

```text
Database CPU = 70%
```

---

# 25. Kubernetes SLI

For Kubernetes workloads, possible service-level indicators include:

```text
Request Success Rate
Request Latency
Pod Availability
Application Availability
Deployment Success
```

Node CPU is generally an infrastructure metric rather than a direct user-facing SLI.

---

# 26. EKS SLI

For an application running on EKS:

```text
User
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Useful SLIs include:

```text
HTTP Success Rate
HTTP Error Rate
Request Latency
Availability
```

Infrastructure metrics such as:

```text
Node CPU
Node Memory
Pod CPU
Pod Memory
```

are useful for troubleshooting the SLI but are not necessarily the SLI itself.

---

# 27. SLI from Prometheus

Prometheus can provide the measurements used to calculate SLIs.

For example:

```text
HTTP Requests
HTTP Errors
Request Duration
```

A common metric pattern might be:

```text
http_requests_total
```

and:

```text
http_request_duration_seconds
```

The exact metric names depend on the application instrumentation.

---

# 28. Availability SLI with Prometheus

Conceptually:

```text
Successful Requests
-------------------
Total Requests
```

For example:

```promql
sum(rate(http_requests_total{status=~"2.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

This calculates an observed success ratio for the selected request population.

The exact labels and metric names depend on the application's instrumentation.

---

# 29. Error Rate SLI with Prometheus

Conceptually:

```text
Failed Requests
---------------
Total Requests
```

Example:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

This provides the observed 5xx error ratio.

---

# 30. Latency SLI with Prometheus

If request duration is exposed as a histogram, percentiles can be calculated.

Example:

```promql
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

This gives an estimated:

```text
p95 latency
```

---

# 31. SLI Measurement Window

An SLI is normally evaluated over a defined period.

Examples:

```text
5 minutes
1 hour
24 hours
7 days
28 days
30 days
```

Example:

```text
Availability over the last 30 days
```

The measurement window becomes especially important when calculating an SLO and error budget.

---

# 32. Good SLI Characteristics

A useful SLI should be:

```text
User-Centric
Measurable
Consistent
Reliable
Relevant
Understandable
Actionable
```

It should answer:

```text
How well is the service actually serving users?
```

---

# 33. Bad SLI Example

Suppose a team defines:

```text
CPU < 80%
```

as the service SLI.

Problem:

```text
CPU = 50%
```

but:

```text
HTTP 5xx = 20%
```

Users are still experiencing failures.

CPU is useful for troubleshooting, but it is not necessarily a good user-facing SLI.

---

# 34. Another Bad SLI Example

Suppose:

```text
Number of Pods = 10
```

This does not tell us whether users can successfully use the service.

You could have:

```text
10 Running Pods
```

but:

```text
0 Successful Requests
```

Therefore:

```text
Pod Count
≠
Service Availability
```

---

# 35. SLI Should Represent User Experience

Consider an online shopping application.

Infrastructure:

```text
EC2 Healthy
EKS Healthy
RDS Healthy
ALB Healthy
```

But:

```text
Checkout Success Rate = 90%
```

From the customer's perspective:

```text
Service is unhealthy.
```

Therefore the checkout success rate can be a much more meaningful SLI.

---

# 36. Choosing the Right SLI

Start with:

```text
What does the user care about?
```

Then determine:

```text
What can we measure?
```

Example:

```text
User wants:
Fast API response

SLI:
p95 API latency
```

Another:

```text
User wants:
Successful checkout

SLI:
Checkout success rate
```

Another:

```text
User wants:
Fresh analytics

SLI:
Data freshness
```

---

# 37. SLI and User Journeys

A service may expose many technical metrics.

Instead, identify important user journeys:

```text
Login
Search
Add to Cart
Checkout
Payment
Order Tracking
```

Then define SLIs around those journeys.

Example:

```text
Checkout Success Rate
```

may be more valuable than:

```text
CPU Utilization
```

---

# 38. SLI and Business Transactions

For business-critical operations:

```text
Login
Payment
Order Creation
Money Transfer
```

measure successful business transactions where possible.

Example:

```text
Successful Payments
-------------------
Payment Attempts
```

This directly reflects business functionality.

---

# 39. SLI and HTTP Status Codes

For HTTP services, status codes can help define request success.

Example:

```text
2xx → Success
5xx → Server Failure
```

But blindly treating all:

```text
4xx
```

as service failures may be incorrect.

For example:

```text
400 Bad Request
```

may indicate invalid customer input rather than an infrastructure failure.

The SLI definition should reflect what the service is responsible for.

---

# 40. Valid Request Population

An important concept is:

```text
Which requests should be included?
```

Example:

```text
Total Requests
```

may include:

```text
Health Checks
Bots
Invalid Requests
Internal Traffic
Customer Requests
```

A useful SLI may need to define the valid request population.

---

# 41. Excluding Health Checks

Suppose Kubernetes performs:

```text
10,000 health checks
```

and users make:

```text
1,000 real requests
```

If health checks are included blindly, they can distort the service-level measurement.

Therefore define the SLI population carefully.

---

# 42. SLI Aggregation

When calculating SLIs, understand what is being aggregated.

Example:

```text
Service A
Service B
Service C
```

You may need:

```text
Overall Service Availability
```

or:

```text
Availability Per Service
```

The aggregation method should reflect the user experience.

---

# 43. Availability Calculation

Suppose over one hour:

```text
Total Valid Requests = 1,000,000
Failed Requests = 1,000
```

Successful requests:

```text
999,000
```

SLI:

```text
999,000 / 1,000,000

= 99.9%
```

So the observed availability SLI is:

```text
99.9%
```

---

# 44. Latency Calculation

Suppose:

```text
p50 = 100 ms
p95 = 250 ms
p99 = 700 ms
```

These are three different measurements.

If the service requirement is concerned with most users:

```text
p95
```

may be appropriate.

If tail behavior is critical:

```text
p99
```

may be more useful.

The SLI should match the service's user experience.

---

# 45. SLI for Asynchronous Processing

For a notification service:

```text
Application
 ↓
RabbitMQ
 ↓
Notification Consumer
 ↓
Email/SMS Provider
```

Possible SLI:

```text
Percentage of notifications delivered within 60 seconds
```

This is more meaningful than:

```text
RabbitMQ CPU
```

or:

```text
Consumer Pod Count
```

---

# 46. SLI for Batch Processing

For a batch pipeline:

```text
Input
 ↓
Processing
 ↓
Output
```

Possible SLIs:

```text
Successful Job Completion
Job Duration
Data Freshness
Data Completeness
```

Example:

```text
Percentage of daily jobs completed successfully
```

---

# 47. SLI for Data Pipeline

Example:

```text
Application
 ↓
Kafka
 ↓
Processing
 ↓
Data Warehouse
```

Possible SLIs:

```text
Event Processing Success
Processing Latency
Data Freshness
Data Completeness
```

Example:

```text
95% of events processed within 5 minutes
```

---

# 48. SLI for Storage

For a storage service:

```text
Write Request
 ↓
Storage
 ↓
Read Request
```

Possible SLIs:

```text
Read Success Rate
Write Success Rate
Read Latency
Write Latency
Data Integrity
```

---

# 49. SLI for DNS

For a DNS service:

```text
DNS Query
 ↓
Response
```

Possible SLIs:

```text
Successful Resolution Rate
DNS Response Latency
```

---

# 50. SLI for Load Balancer

For an ALB-backed application:

```text
Request
 ↓
ALB
 ↓
Target
```

Potential SLIs:

```text
Successful Requests
Request Latency
HTTP Error Rate
Availability
```

ALB metrics can help calculate or validate these indicators.

---

# 51. SLI for Authentication

For an authentication service:

```text
Login Request
 ↓
Authentication
 ↓
Token
```

Possible SLIs:

```text
Successful Authentication Rate
Authentication Latency
Token Issuance Success
```

A login service being available does not necessarily mean authentication is working correctly.

---

# 52. SLI for Payment

For a payment service:

```text
Payment Request
 ↓
Validation
 ↓
Payment Provider
 ↓
Payment Result
```

Possible SLI:

```text
Successful Payment Transactions
--------------------------------
Valid Payment Attempts
```

This is a business-oriented SLI.

---

# 53. SLI for Search

For a search service:

```text
Search Request
 ↓
Search Engine
 ↓
Results
```

Possible SLIs:

```text
Search Success Rate
Search Latency
Result Correctness
```

Returning HTTP 200 without useful results may not represent a successful search experience.

---

# 54. SLI for E-Commerce Checkout

For checkout:

```text
Cart
 ↓
Checkout
 ↓
Payment
 ↓
Order
```

A business SLI could be:

```text
Successful Checkouts
--------------------
Valid Checkout Attempts
```

Supporting technical SLIs might include:

```text
Checkout Latency
Payment Success Rate
Order Creation Success Rate
```

---

# 55. Composite User Journey

Sometimes a user journey depends on multiple services:

```text
Checkout
   ↓
Payment
   ↓
Inventory
   ↓
Order
```

If any critical dependency fails:

```text
Checkout may fail
```

Therefore a user-journey SLI can sometimes be more meaningful than individual service metrics.

---

# 56. SLI vs Monitoring Metric

Not every metric is an SLI.

Examples:

```text
CPU
Memory
Disk
Pod Count
Node Count
```

are commonly:

```text
Operational Metrics
```

Examples:

```text
Request Success Rate
Latency
Data Freshness
Transaction Success
```

are more likely:

```text
SLIs
```

The distinction depends on how the organization defines the service objective.

---

# 57. SLI vs Health Check

Health check:

```text
Is the process alive?
```

SLI:

```text
How well is the service serving users?
```

Example:

```text
/health = 200
```

but:

```text
/checkout = 500
```

The health check says:

```text
Process is alive.
```

The SLI says:

```text
Customer transaction is failing.
```

---

# 58. SLI vs KPI

KPI:

```text
Business performance indicator
```

Examples:

```text
Revenue
Orders
Conversion Rate
Customer Retention
```

SLI:

```text
Service reliability indicator
```

Examples:

```text
Availability
Latency
Success Rate
Freshness
```

They can overlap.

For example:

```text
Checkout Success Rate
```

can be both:

```text
Business KPI
```

and:

```text
Service SLI
```

depending on the context.

---

# 59. SLI Measurement Quality

A poor SLI can create misleading conclusions.

For example:

```text
Average Latency
```

may hide tail latency.

Or:

```text
Pod Availability
```

may hide failed requests.

Or:

```text
CPU
```

may not represent user experience.

Therefore:

```text
Choose the measurement carefully.
```

---

# 60. SLI Should Be Actionable

A useful SLI should help answer:

```text
Is the service meeting user expectations?
```

and:

```text
Are we getting closer to an incident?
```

Example:

```text
p95 latency = 2 seconds
```

can directly indicate degraded user experience.

---

# 61. SLI and Alerting

An SLI can be used as the foundation for reliability alerting.

Flow:

```text
Service
 ↓
SLI
 ↓
SLO
 ↓
Error Budget
 ↓
Alerting
```

Example:

```text
HTTP Success Rate
 ↓
99.9% SLO
 ↓
Error Budget
 ↓
Burn Rate Alert
```

This is more reliability-focused than alerting on every infrastructure metric.

---

# 62. SLI and SLO Relationship

The SLI is the measurement.

The SLO is the target.

Example:

```text
Observed Availability:
99.85%

SLO:
99.9%
```

Therefore:

```text
SLI < SLO
```

The service is currently below its objective.

---

# 63. SLI and Error Budget

Suppose:

```text
SLO = 99.9%
```

Then allowed unreliability is:

```text
0.1%
```

The SLI tells us:

```text
How much reliability we actually achieved.
```

The error budget tells us:

```text
How much unreliability is allowed.
```

---

# 64. SLI Over Time

An SLI should usually be observed as a time series.

Example:

```text
Monday     99.99%
Tuesday    99.95%
Wednesday  99.80%
Thursday   99.97%
Friday     99.99%
```

This helps identify:

```text
Incidents
Trends
Degradation
Recovery
```

---

# 65. SLI During Deployment

Suppose:

```text
Before Deployment:
p95 = 200ms

After Deployment:
p95 = 700ms
```

The SLI changed significantly.

This can indicate a regression even if:

```text
Deployment = Successful
```

---

# 66. SLI During Scaling

Suppose traffic increases:

```text
Traffic ↑
```

and:

```text
Latency remains stable
Error rate remains low
```

The service is handling the load well.

But if:

```text
Traffic ↑
Latency ↑
Errors ↑
```

the SLI indicates degradation.

---

# 67. SLI and Capacity Planning

Historical SLI data can help identify capacity problems.

Example:

```text
Traffic ↑
 ↓
Latency gradually ↑
```

This may indicate:

```text
Capacity Limit
```

SLI trends can therefore support capacity planning.

---

# 68. SLI and Incident Detection

Suppose:

```text
CPU = 60%
```

but:

```text
Checkout Success = 85%
```

The service is clearly experiencing a customer-facing problem.

An SLI-based alert can detect this even when infrastructure metrics appear normal.

---

# 69. SLI and Root Cause Analysis

SLIs help identify:

```text
When user experience changed.
```

Example:

```text
10:00 Deployment
10:02 p95 latency ↑
10:04 Error rate ↑
10:05 Alert
```

This provides useful evidence during RCA.

---

# 70. SLI Design Principles

When designing an SLI:

```text
1. Start from the user journey.
2. Identify important service behavior.
3. Choose a measurable indicator.
4. Define success clearly.
5. Define the measurement population.
6. Choose an appropriate aggregation.
7. Choose an appropriate time window.
8. Validate that the metric represents user experience.
```

---

# 71. SLI Design Example — API

Requirement:

```text
Users need the API to respond successfully and quickly.
```

SLIs:

```text
Availability SLI:
Successful Requests / Total Valid Requests

Latency SLI:
p95 Request Latency
```

This gives two dimensions:

```text
Can users use it?
How fast does it respond?
```

---

# 72. SLI Design Example — Payment

Requirement:

```text
Users need successful payments.
```

SLI:

```text
Successful Payments
-------------------
Valid Payment Attempts
```

Supporting SLI:

```text
Payment Latency
```

Possible dependency measurements:

```text
Payment Provider Success Rate
```

---

# 73. SLI Design Example — Notification

Requirement:

```text
Notifications should be delivered quickly.
```

SLI:

```text
Percentage of notifications delivered within target time
```

Example:

```text
Notifications delivered within 60 seconds
------------------------------------------------
Valid Notifications
```

---

# 74. SLI Design Example — Data Pipeline

Requirement:

```text
Analytics data should be fresh.
```

SLI:

```text
Data Freshness
```

Example:

```text
Current Data Age = 3 minutes
```

Another SLI:

```text
Percentage of records processed successfully
```

---

# 75. SLI Design Example — Kubernetes Application

Requirement:

```text
Application should remain available to users.
```

SLIs:

```text
HTTP Success Rate
p95 Latency
```

Supporting operational metrics:

```text
Pod CPU
Pod Memory
Node CPU
Node Memory
Pod Restarts
```

The operational metrics help explain the SLI but are not necessarily the primary user-facing indicators.

---

# 76. SLI Design Example — EKS Microservices

For an EKS microservices platform:

```text
User
 ↓
ALB
 ↓
Ingress
 ↓
Microservice
 ↓
Database / Dependency
```

Useful SLIs:

```text
Request Success Rate
Request Latency
Service Availability
Transaction Success Rate
```

Supporting metrics:

```text
Pod Resources
Node Resources
Restarts
ALB Target Health
Database Resources
```

---

# 77. Multiple SLIs

A service rarely needs only one SLI.

For an API:

```text
SLI 1:
Availability

SLI 2:
Latency

SLI 3:
Correctness
```

For a data pipeline:

```text
SLI 1:
Freshness

SLI 2:
Processing Success

SLI 3:
Completeness
```

Use only indicators that represent meaningful service behavior.

---

# 78. Avoid Too Many SLIs

Having:

```text
50 SLIs
```

does not automatically improve reliability.

Too many indicators can create:

```text
Complexity
Confusion
Alert Noise
Difficult Objectives
```

Start with the most important user-facing characteristics.

---

# 79. SLI Naming

SLIs should have clear names.

Good:

```text
http_request_success_rate
checkout_success_rate
p95_request_latency
notification_delivery_success
data_freshness
```

Poor:

```text
metric1
service_health
system_good
```

Clear names make dashboards and alerts easier to understand.

---

# 80. SLI Documentation

Document:

```text
SLI Name
Purpose
Formula
Metric Source
Included Requests
Excluded Requests
Time Window
Aggregation
Owner
Dashboard
```

Example:

```text
SLI:
Checkout Success Rate

Formula:
Successful Checkouts / Valid Checkout Attempts

Source:
Application Metrics

Owner:
Checkout Team
```

---

# 81. SLI Review

SLIs should be reviewed when:

```text
Architecture Changes
User Journey Changes
New Features
New Dependencies
Monitoring Changes
Incident Findings
Business Requirements Change
```

An SLI that was useful six months ago may not represent the current service correctly.

---

# 82. Common SLI Mistakes

### Mistake 1

Using infrastructure metrics as the primary service indicator.

```text
CPU = 70%
```

does not automatically mean the service is healthy.

### Mistake 2

Using average latency only.

```text
Average = 100ms
```

may hide tail latency.

### Mistake 3

Including invalid traffic.

```text
Bots
Health Checks
Invalid Requests
```

can distort the measurement.

### Mistake 4

Choosing an SLI that nobody can reliably measure.

### Mistake 5

Defining an SLI without considering the user journey.

---

# 83. SLI Anti-Pattern

Bad:

```text
SLO:
CPU < 80%
```

Better:

```text
SLI:
Successful Requests / Valid Requests
```

Then:

```text
SLO:
99.9% successful requests
```

CPU can still be monitored as an operational metric.

---

# 84. SLI Anti-Pattern — Pod Count

Bad:

```text
10 Pods Running
```

This does not guarantee:

```text
Users Can Successfully Use Service
```

Better:

```text
Request Success Rate
```

---

# 85. SLI Anti-Pattern — Average Latency

Bad:

```text
Average Latency < 500ms
```

without understanding the distribution.

Better:

```text
p95 Latency
```

or:

```text
p99 Latency
```

when tail performance matters.

---

# 86. SLI Anti-Pattern — Monitoring Everything

Collecting every possible metric does not mean you have good SLIs.

The objective is:

```text
Measure What Matters
```

rather than:

```text
Measure Everything
```

---

# 87. SLI and Observability

Observability provides the signals needed to understand service behavior.

Common signals:

```text
Metrics
Logs
Traces
```

SLIs are generally derived from measurements that represent important service behavior.

Example:

```text
Prometheus
 ↓
Request Metrics
 ↓
SLI

ELK
 ↓
Error Investigation
 ↓
Root Cause

Tracing
 ↓
Latency Investigation
 ↓
Dependency Analysis
```

---

# 88. SLI and Prometheus + Grafana

A typical architecture:

```text
Application
      ↓
Metrics
      ↓
Prometheus
      ↓
SLI Queries
      ↓
Grafana
      ↓
SLO Dashboard
```

Grafana can show:

```text
Availability
Latency
Error Rate
SLI Trend
SLO Status
```

---

# 89. SLI and ELK

ELK is often more useful for understanding why an SLI degraded.

Example:

```text
SLI:
Error Rate ↑
```

Then:

```text
Grafana
 ↓
Error Rate Increase
 ↓
Kibana
 ↓
Database Timeout Errors
```

The SLI detects the problem.

Logs help investigate the cause.

---

# 90. SLI and Tracing

Tracing is useful when an SLI such as latency degrades.

Example:

```text
p95 latency ↑
      ↓
Trace
      ↓
Service A
      ↓
Service B
      ↓
Database
      ↓
Slow Query
```

Tracing helps identify where the latency originates.

---

# 91. SLI and DevOps

SLIs connect monitoring with operational reliability.

Instead of asking only:

```text
Is infrastructure healthy?
```

we ask:

```text
Is the service delivering the expected experience?
```

This shifts monitoring toward:

```text
User-Centric Reliability
```

---

# 92. SLI and DevSecOps

Security can also affect service-level behavior.

Examples:

```text
Authentication Success
Authorization Success
Security Service Availability
Certificate Validity
```

Security controls should not unintentionally create unacceptable service degradation.

---

# 93. SLI and CI/CD

SLIs can be used to evaluate deployments.

Flow:

```text
Deploy
 ↓
Monitor SLI
 ↓
Compare Before / After
 ↓
Healthy?
 ├── Yes → Continue
 └── No  → Rollback
```

This is especially useful for:

```text
Canary Deployments
Progressive Delivery
Automated Rollback
```

---

# 94. SLI and GitOps

For GitOps environments:

```text
Git
 ↓
ArgoCD
 ↓
Kubernetes
 ↓
Application
 ↓
SLI
 ↓
Monitoring
```

If a deployment causes SLI degradation:

```text
SLI Degradation
 ↓
Investigation
 ↓
Rollback / Git Change
```

SLIs can therefore become part of the deployment reliability process.

---

# 95. SLI and Production Readiness

Before releasing a production service, define:

```text
What is the user journey?
What does success mean?
What should latency be?
What failures matter?
How will we measure them?
```

Then create:

```text
SLIs
Dashboards
Alerts
SLOs
Runbooks
```

---

# 96. SLI Production Checklist

```text
☐ Identify critical user journeys
☐ Define service success
☐ Define valid request population
☐ Select meaningful SLIs
☐ Define formulas
☐ Select metric sources
☐ Create dashboards
☐ Validate data quality
☐ Review aggregation
☐ Review latency percentiles
☐ Exclude irrelevant traffic
☐ Document ownership
☐ Connect SLIs to SLOs
```

---

# 97. Real Production Example

Consider an application running on:

```text
AWS
 ↓
EKS
 ↓
ALB
 ↓
Microservices
 ↓
RDS
```

Monitoring stack:

```text
Prometheus
Grafana
ELK
```

For the application:

```text
SLI 1:
HTTP Success Rate

SLI 2:
p95 Request Latency

SLI 3:
Checkout Success Rate
```

Infrastructure metrics:

```text
Node CPU
Node Memory
Pod CPU
Pod Memory
RDS CPU
```

The infrastructure metrics help explain why the SLIs changed.

---

# 98. Real Production Incident

Suppose:

```text
Node CPU = 90%
```

but:

```text
HTTP Success Rate = 99.99%
p95 = 200ms
```

The service may still be meeting user expectations.

Another situation:

```text
Node CPU = 50%
```

but:

```text
HTTP Success Rate = 92%
p95 = 3 seconds
```

The service is clearly degraded.

This demonstrates why user-facing SLIs are important.

---

# 99. SLI Investigation During an Incident

Suppose:

```text
Availability SLI ↓
```

Follow:

```text
SLI
 ↓
Metric
 ↓
Time of Change
 ↓
Recent Deployment
 ↓
Logs
 ↓
Infrastructure
 ↓
Dependency
```

The SLI tells you:

```text
Something is wrong.
```

Other observability signals help determine:

```text
Why?
```

---

# 100. SLI Mental Model

Remember:

```text
SLI = Measurement
```

Examples:

```text
99.95% availability
250ms p95 latency
98.5% payment success
3-minute data freshness
```

Then:

```text
SLI
 ↓
Compare Against SLO
 ↓
Determine Reliability
```

---

# 101. Interview Question

## What is an SLI?

**Answer:**

An SLI, or Service Level Indicator, is a quantitative measurement of a service's actual performance from the user's perspective.

Examples include:

```text
Availability
Request Success Rate
Latency
Throughput
Data Freshness
```

For example:

```text
Successful Requests / Total Valid Requests
```

can be used as an availability SLI.

---

# 102. Interview Question

## What is the difference between SLI and SLO?

**Answer:**

An SLI is the actual measurement, while an SLO is the target we want to achieve.

Example:

```text
SLI:
Current availability = 99.85%

SLO:
Target availability = 99.9%
```

So:

```text
SLI = What we observe
SLO = What we target
```

---

# 103. Interview Question

## Is CPU an SLI?

**Answer:**

Usually CPU is an infrastructure or operational metric rather than a primary user-facing SLI.

For example:

```text
CPU = 90%
```

does not necessarily mean users are experiencing an outage.

A better service-level indicator might be:

```text
Request Success Rate
```

or:

```text
p95 Request Latency
```

CPU can still be extremely useful for diagnosing why an SLI is degrading.

---

# 104. Interview Question

## What SLIs would you define for an API?

**Answer:**

I would typically start with:

```text
1. Availability / Request Success Rate
2. Latency
```

For example:

```text
Success Rate:
Successful Requests / Valid Requests

Latency:
p95 Request Latency
```

Depending on the API, I may also consider:

```text
Correctness
Throughput
```

---

# 105. Interview Question

## What SLIs would you define for a payment service?

**Answer:**

For a payment service, I would prioritize business-facing indicators:

```text
Payment Success Rate
Payment Latency
```

For example:

```text
Successful Payments
-------------------
Valid Payment Attempts
```

I would also monitor supporting infrastructure and dependency metrics, but the primary SLIs should represent the actual payment experience.

---

# 106. Interview Question

## Why use p95 or p99 instead of average latency?

**Answer:**

Average latency can hide tail latency.

For example:

```text
Average = 200ms
```

may look healthy while a significant number of users experience several seconds of latency.

Percentiles such as:

```text
p95
p99
```

show the tail behavior and provide a better understanding of user experience.

---

# 107. Interview Question

## How would you define an SLI for an EKS application?

**Answer:**

I would start from the user request path:

```text
User
 ↓
ALB
 ↓
Service
 ↓
Pod
```

I would define:

```text
Request Success Rate
p95 Request Latency
```

Then use:

```text
Prometheus
Grafana
ELK
```

to measure and investigate those indicators.

Infrastructure metrics such as node and pod CPU would be supporting signals.

---

# 108. Interview Question

## How do you choose the right SLI?

**Answer:**

I start with the user journey and ask:

```text
What does successful service delivery mean?
```

Then I identify a measurable indicator that represents that experience.

For example:

```text
Checkout → Checkout Success Rate
API → Request Success Rate + Latency
Data Pipeline → Freshness + Processing Success
Notification → Delivery Success + Delivery Latency
```

---

# 109. Interview Question

## Can one service have multiple SLIs?

**Answer:**

Yes.

For an API:

```text
Availability
Latency
Correctness
```

For a data pipeline:

```text
Freshness
Completeness
Processing Success
```

The number of SLIs should be limited to the indicators that meaningfully represent service reliability.

---

# 110. Interview Question

## What makes a good SLI?

**Answer:**

A good SLI should be:

```text
User-Centric
Measurable
Reliable
Relevant
Understandable
Actionable
```

Most importantly, it should represent something users actually care about.

---

# 111. Interview Question

## How are SLIs related to observability?

**Answer:**

Observability provides the telemetry needed to measure and investigate service behavior.

For example:

```text
Prometheus
 ↓
Request Metrics
 ↓
SLI

Grafana
 ↓
SLI Visualization

ELK
 ↓
Root Cause Investigation

Tracing
 ↓
Latency / Dependency Investigation
```

The SLI tells us how the service is performing, while observability helps us understand why.

---

# 112. Interview Question

## How can SLIs help during deployments?

**Answer:**

I can compare SLIs before and after deployment.

Example:

```text
Before:
p95 = 200ms

After:
p95 = 900ms
```

If the degradation correlates with the deployment, I can investigate or roll back.

SLIs can therefore be used as deployment health signals.

---

# 113. Interview Question

## What is a common mistake when defining SLIs?

**Answer:**

A common mistake is choosing infrastructure metrics instead of user-facing measurements.

For example:

```text
CPU < 80%
```

does not directly represent user experience.

A better approach is:

```text
Request Success Rate
+
Latency
```

and use CPU, memory, and node metrics as supporting troubleshooting signals.

---

# 114. Interview Question

## How would you explain SLI to a non-technical stakeholder?

**Answer:**

I would say:

> An SLI is a measurement that tells us how well our service is actually working for users.

For example:

```text
99.9% of customer requests succeeded.
```

That number is the SLI.

---

# 115. Final SLI Cheat Sheet

```text
SLI
=
Service Level Indicator

Meaning:
Actual measurement of service performance
```

Common SLIs:

```text
Availability
Success Rate
Latency
Throughput
Correctness
Freshness
Durability
```

Key formulas:

```text
Availability =
Successful Requests / Total Valid Requests

Error Rate =
Failed Requests / Total Valid Requests

Success Rate =
Successful Requests / Total Valid Requests
```

Latency:

```text
p50
p95
p99
```

Important distinction:

```text
SLI = Measurement
SLO = Target
SLA = Commitment
Error Budget = Allowed Unreliability
```

Core principle:

```text
Measure what users experience,
not only what infrastructure is doing.
```

Production mental model:

```text
User Journey
     ↓
Service Behavior
     ↓
SLI
     ↓
SLO
     ↓
Error Budget
     ↓
Alerting / Reliability Decisions
```

The most important interview statement:

```text
An SLI is the actual quantitative measurement of
service performance, while an SLO defines the desired
target for that measurement.
```