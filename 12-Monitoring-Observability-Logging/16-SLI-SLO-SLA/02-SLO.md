# SLO — Service Level Objective

## 1. Overview

SLO stands for **Service Level Objective**.

An SLO defines the target level of reliability or performance that a service is expected to achieve over a specific measurement period.

In simple terms:

```text
SLI = What we measure

SLO = What level we want to achieve
```

Example:

```text
SLI:
Request Success Rate

SLO:
99.9% of valid requests should succeed
```

The SLI provides the actual observed performance, while the SLO defines the desired target.

---

## 2. SLI vs SLO

The relationship is:

```text
SLI
 ↓
Actual Performance

SLO
 ↓
Target Performance
```

Example:

```text
SLI = 99.85% availability

SLO = 99.9% availability
```

Here:

```text
Actual Performance = 99.85%
Target Performance = 99.9%
```

The service is currently below its objective.

---

## 3. Why SLOs Are Important

Without an SLO, statements such as:

```text
"The application should be highly available."
"The API should be fast."
"The service should be reliable."
```

are difficult to measure.

An SLO converts these statements into measurable objectives.

For example:

```text
99.9% availability over 30 days
```

or:

```text
95% of requests complete within 300ms
```

Now the engineering team has a clear reliability target.

---

## 4. SLO Formula

An SLO can be represented as:

```text
Observed SLI >= SLO Target
```

Example:

```text
SLI = 99.95%

SLO = 99.9%
```

Therefore:

```text
99.95% >= 99.9%
```

The service is meeting the SLO.

If:

```text
SLI = 99.7%

SLO = 99.9%
```

then:

```text
99.7% < 99.9%
```

The service is violating the SLO.

---

## 5. Availability SLO

Availability is one of the most common SLOs.

Example:

```text
SLO = 99.9% availability
```

This means the service should successfully handle at least:

```text
99.9%
```

of the defined valid requests during the measurement period.

The exact definition of a successful request should be documented.

---

## 6. Availability SLO Example

Suppose:

```text
Total Valid Requests = 1,000,000
```

SLO:

```text
99.9%
```

Allowed failure percentage:

```text
100% - 99.9%
= 0.1%
```

Therefore:

```text
1,000,000 × 0.001
= 1,000
```

The service can have up to approximately:

```text
1,000 failed requests
```

within this request population and still achieve 99.9% availability.

---

## 7. Latency SLO

SLOs can also define latency targets.

Example:

```text
95% of valid requests should complete
within 300ms.
```

Here:

```text
SLI = Request Latency

SLO = 95% of requests <= 300ms
```

This is more meaningful than simply saying:

```text
Average latency < 300ms
```

because averages can hide slow requests.

---

## 8. Percentile-Based Latency SLO

A common latency SLO is:

```text
p95 latency <= 300ms
```

Example:

```text
Current p95 = 250ms

Target p95 = 300ms
```

The service is meeting the latency SLO.

If:

```text
Current p95 = 500ms
```

then the latency SLO is violated.

---

## 9. p50, p95 and p99 SLOs

Different percentiles represent different portions of traffic.

```text
p50
 ↓
Typical request experience

p95
 ↓
Broader user experience

p99
 ↓
Tail user experience
```

Example:

```text
p50 = 100ms
p95 = 300ms
p99 = 700ms
```

A service may choose:

```text
p95 <= 300ms
```

or:

```text
p99 <= 700ms
```

depending on its requirements.

---

## 10. Error Rate SLO

An SLO can be defined using error rate.

Example:

```text
SLO:
5xx error rate <= 0.1%
```

This can also be expressed as:

```text
Request Success Rate >= 99.9%
```

The important point is that the organization clearly defines which errors are included.

---

## 11. Success Rate SLO

Example:

```text
SLI:
Successful Requests / Total Valid Requests
```

SLO:

```text
99.9% successful requests
```

Observed:

```text
99.95%
```

Result:

```text
SLO Met
```

Observed:

```text
99.7%
```

Result:

```text
SLO Violated
```

---

## 12. SLO Measurement Window

An SLO must have a defined measurement period.

Common windows include:

```text
7 days
14 days
28 days
30 days
90 days
```

Example:

```text
Availability SLO:
99.9% over a rolling 30-day window
```

Without a measurement window, the SLO is incomplete.

---

## 13. Calendar-Based SLO

A calendar-based SLO uses a fixed time period.

Example:

```text
January 1
     ↓
January 31
```

The SLO is calculated for that specific calendar period.

At the beginning of February:

```text
February 1
```

a new measurement period begins.

---

## 14. Rolling SLO

A rolling SLO continuously evaluates a moving time window.

Example:

```text
Rolling 30 days
```

Today:

```text
Previous 30 days
```

Tomorrow:

```text
New previous 30 days
```

As time moves forward:

```text
Old data leaves
New data enters
```

This provides a continuously updated reliability view.

---

## 15. Calendar vs Rolling SLO

### Calendar SLO

```text
January 1 → January 31
February 1 → February 28
```

### Rolling SLO

```text
Last 30 days
```

The rolling window continuously moves forward.

Both approaches can be useful depending on reporting, operational, and business requirements.

---

## 16. SLO and Error Budget

SLO and error budget are directly connected.

Example:

```text
SLO = 99.9%
```

Allowed unreliability:

```text
100% - 99.9%
= 0.1%
```

Therefore:

```text
Error Budget = 0.1%
```

Mental model:

```text
SLO
 ↓
Allowed Failure
 ↓
Error Budget
```

---

## 17. Error Budget Example

Suppose:

```text
SLO = 99.9%
```

Then:

```text
Error Budget = 0.1%
```

For:

```text
1,000,000 valid requests
```

the allowed failures are:

```text
1,000,000 × 0.001
= 1,000
```

If the service has:

```text
300 failures
```

there are approximately:

```text
700 failures
```

remaining in the budget.

---

## 18. SLO Is Not SLA

SLO and SLA are different concepts.

```text
SLO
 ↓
Internal reliability target

SLA
 ↓
Formal customer or contractual commitment
```

Example:

```text
Internal SLO = 99.9%

Customer SLA = 99.5%
```

The SLO can intentionally be stricter than the SLA.

---

## 19. Why SLO Can Be Stricter Than SLA

Suppose:

```text
SLO = 99.9%

SLA = 99.5%
```

The engineering team targets:

```text
99.9%
```

instead of waiting until:

```text
99.5%
```

is reached.

This creates a safety margin before the contractual SLA is threatened.

---

## 20. SLO for APIs

For a production API, possible SLOs include:

```text
Availability:
99.9%

Latency:
95% of requests <= 300ms

Error Rate:
5xx <= 0.1%
```

The exact values should be based on:

```text
User Expectations
Business Requirements
Historical Performance
Cost
Risk
```

---

## 21. SLO for EKS Applications

For an application running on EKS:

```text
User
 ↓
ALB
 ↓
Kubernetes Service
 ↓
Pod
```

Example SLOs:

```text
Availability:
99.9%

Latency:
p95 <= 300ms

Error Rate:
5xx <= 0.1%
```

Infrastructure metrics such as:

```text
Node CPU
Node Memory
Pod CPU
Pod Memory
```

are supporting operational metrics.

---

## 22. SLO for Microservices

Consider:

```text
User
 ↓
ALB
 ↓
Service A
 ↓
Service B
 ↓
Service C
```

Each service can have its own SLO.

Example:

```text
User Service:
99.9%

Product Service:
99.9%

Order Service:
99.95%

Payment Service:
99.99%
```

Critical services may require stricter objectives.

---

## 23. SLO Should Reflect Criticality

Not every service needs:

```text
99.99%
```

For example:

```text
Internal Reporting Service
```

may tolerate more downtime than:

```text
Payment Service
```

SLOs should consider:

```text
Business Criticality
User Expectations
Risk
Dependencies
Cost
```

---

## 24. SLO and Business Requirements

SLO design should involve the appropriate stakeholders.

Examples:

```text
Product
Engineering
DevOps
SRE
Business
Support
```

For example:

```text
Payment Service
```

may require a stricter SLO than:

```text
Internal Administration Tool
```

The target should reflect actual business needs.

---

## 25. SLO and User Experience

A good SLO represents something users care about.

Bad:

```text
CPU < 80%
```

Better:

```text
99.9% of requests succeed
```

Another example:

```text
95% of requests complete within 300ms
```

The objective should be connected to actual service behavior.

---

## 26. SLO and Infrastructure Metrics

Infrastructure metrics remain important:

```text
CPU
Memory
Disk
Network
Pod Restarts
Node Health
```

But they generally support the investigation of SLO violations.

Example:

```text
SLO Violation
 ↓
Latency Increased
 ↓
Pod CPU Increased
 ↓
CPU Saturation
 ↓
Root Cause
```

---

## 27. SLO and Monitoring

A typical monitoring architecture:

```text
Application
 ↓
Metrics
 ↓
Prometheus
 ↓
SLI
 ↓
SLO Calculation
 ↓
Grafana
```

Grafana can display:

```text
Current SLI
SLO Target
Error Budget
SLO Status
SLO Trend
```

---

## 28. SLO and Alerting

SLOs can be used as a foundation for reliability-focused alerting.

Instead of alerting only on:

```text
CPU > 80%
Memory > 80%
Disk > 70%
```

teams can also alert when:

```text
SLO is at risk
```

This focuses alerts on customer impact.

---

## 29. SLO-Based Alerting

Example:

```text
SLO = 99.9%
```

Then:

```text
Error Rate ↑
 ↓
Error Budget Consumption ↑
 ↓
SLO Risk ↑
 ↓
Alert
```

The alert becomes connected to service reliability rather than just infrastructure utilization.

---

## 30. SLO and Burn Rate

Burn rate measures how quickly the service is consuming its error budget.

Conceptually:

```text
Error Budget
 ↓
Errors Consumed
 ↓
Consumption Rate
 ↓
Burn Rate
```

If the service consumes the error budget much faster than expected:

```text
Burn Rate ↑
```

the team should investigate.

---

## 31. Fast Burn

Suppose:

```text
SLO = 99.9%
```

A deployment causes:

```text
20% of requests to fail
```

The error budget is being consumed extremely quickly.

This is:

```text
High Burn Rate
```

A fast-burning budget usually requires immediate attention.

---

## 32. Slow Burn

Suppose the error rate gradually increases:

```text
0.10%
0.12%
0.15%
0.18%
```

The service may be slowly consuming its error budget.

This is:

```text
Slow Burn
```

It may not cause an immediate outage, but it can still threaten the SLO over time.

---

## 33. SLO and Deployment Decisions

SLOs can influence release decisions.

Example:

```text
Error Budget Healthy
 ↓
Normal Release Activity
```

If:

```text
Error Budget Low
```

the team may:

```text
Reduce Risky Releases
Prioritize Reliability
Investigate Recurring Failures
```

If:

```text
Error Budget Exhausted
```

the organization may temporarily prioritize reliability work over high-risk feature releases.

---

## 34. Error Budget Policy

A team might define:

```text
Error Budget Healthy
 ↓
Normal Feature Development
```

```text
Error Budget Low
 ↓
Increase Reliability Work
```

```text
Error Budget Exhausted
 ↓
Pause High-Risk Releases
```

The exact policy depends on the organization.

---

## 35. SLO and CI/CD

SLOs can be used during deployment validation.

Example:

```text
Deploy
 ↓
Canary
 ↓
Measure SLI
 ↓
Compare with SLO / Baseline
 ↓
Healthy?
 ├── Yes → Continue
 └── No  → Stop / Rollback
```

This is useful for:

```text
Canary Deployments
Progressive Delivery
Automated Rollback
Blue-Green Deployments
```

---

## 36. SLO and GitOps

For a GitOps environment:

```text
Git
 ↓
ArgoCD
 ↓
EKS
 ↓
Application
 ↓
Prometheus
 ↓
SLI
 ↓
SLO
```

If the new version causes SLO degradation:

```text
SLO Degradation
 ↓
Investigation
 ↓
Git Revert
 ↓
ArgoCD Sync
 ↓
Previous Version
```

---

## 37. SLO for Java Applications

Example:

```text
Availability SLO:
99.9%

Latency SLO:
95% <= 500ms
```

Prometheus can collect:

```text
HTTP Requests
Request Duration
Errors
```

Grafana can visualize:

```text
SLI
SLO
Error Budget
```

---

## 38. SLO for Node.js Applications

Example:

```text
Availability:
99.9%

Latency:
p95 <= 300ms
```

If:

```text
p95 = 800ms
```

the latency SLO is violated.

Investigation may include:

```text
Application Logs
CPU
Memory
Database
External APIs
Application Runtime
```

---

## 39. SLO for Python Applications

Example:

```text
Success Rate:
99.95%

Latency:
p95 <= 500ms
```

If success rate drops:

```text
99.95%
 ↓
99.5%
```

the team investigates:

```text
Application
Dependencies
Database
Infrastructure
Recent Deployments
```

---

## 40. SLO for Database

Possible database SLOs:

```text
Availability
Query Latency
Transaction Success
```

Example:

```text
99.95% successful transactions
```

Supporting metrics may include:

```text
CPU
Memory
Connections
Storage
Query Performance
```

---

## 41. SLO for Message Processing

For a message-processing system:

```text
Producer
 ↓
RabbitMQ
 ↓
Consumer
```

Possible SLO:

```text
99% of messages processed within 30 seconds
```

SLI:

```text
Percentage of messages processed within 30 seconds
```

SLO:

```text
99%
```

---

## 42. SLO for Data Pipeline

Example:

```text
99% of daily jobs complete successfully.
```

Another possible objective:

```text
95% of data is available within 5 minutes.
```

Possible SLIs:

```text
Job Success Rate
Data Freshness
Processing Latency
Data Completeness
```

---

## 43. SLO for Notification Service

Example:

```text
99% of notifications delivered within 60 seconds.
```

SLI:

```text
Notifications Delivered Within 60 Seconds
------------------------------------------
Valid Notifications
```

SLO:

```text
99%
```

---

## 44. SLO for Search Service

Possible objectives:

```text
99.9% search request availability
```

and:

```text
95% of searches complete within 500ms
```

If correctness can be reliably measured, it may also be considered.

---

## 45. SLO for Authentication

Possible objectives:

```text
99.95% authentication success
```

and:

```text
95% of authentication requests <= 300ms
```

The exact targets should depend on business requirements and user experience.

---

## 46. SLO for Checkout

For a checkout service:

```text
99.9% successful checkout transactions
```

Supporting objective:

```text
95% of checkout requests <= 1 second
```

This focuses on a critical business user journey.

---

## 47. SLO for Payment

For a payment service:

```text
99.99% successful payment processing
```

Possible latency objective:

```text
95% of payment requests <= 1 second
```

The actual target should consider:

```text
Business Requirements
Payment Provider
Risk
Customer Expectations
```

---

## 48. Composite SLO

A user journey may depend on multiple services.

Example:

```text
Checkout
 ↓
Payment
 ↓
Inventory
 ↓
Order
```

A composite SLO can measure:

```text
Successful End-to-End Checkouts
--------------------------------
Valid Checkout Attempts
```

This can represent the customer's experience better than looking at each individual service separately.

---

## 49. Dependency SLO

A service may depend on:

```text
Database
External API
Message Queue
Authentication Service
```

The team may monitor dependency SLOs separately.

Example:

```text
Application SLO
      ↓
Database SLO
      ↓
External API SLO
```

Dependency failures can impact the application's SLO.

---

## 50. SLO and Third-Party Services

Suppose:

```text
Application
 ↓
External Payment API
```

If the external API becomes unavailable:

```text
Application SLI ↓
```

During investigation, distinguish:

```text
Application Reliability
```

from:

```text
Dependency Reliability
```

This helps identify the actual source of degradation.

---

## 51. SLO and Multi-Region Architecture

For a multi-region application:

```text
Users
 ↓
Route 53
 ↓
Region A / Region B
 ↓
EKS
```

SLOs can be measured:

```text
Globally
Per Region
```

Example:

```text
Global Availability SLO = 99.99%
```

Individual regions can still be monitored independently.

---

## 52. SLO and High Availability

A highly available architecture may include:

```text
Multiple Availability Zones
Multiple Nodes
Pod Replicas
Auto Scaling
Load Balancing
Multi-Region Deployment
```

However:

```text
Architecture
≠
SLO Achievement
```

We still need to measure actual user experience.

---

## 53. SLO and Auto Scaling

Suppose:

```text
Traffic ↑
```

HPA scales:

```text
3 Pods
 ↓
8 Pods
```

If:

```text
Latency remains within SLO
```

the scaling strategy is working.

If:

```text
Latency exceeds SLO
```

investigate:

```text
Scaling Limits
Node Capacity
Application Bottleneck
Database
Dependencies
```

---

## 54. SLO During an Incident

Suppose:

```text
SLO = 99.9%

Current SLI = 98.5%
```

The service is significantly below target.

Incident response should focus on:

```text
Restore Service
Stop Error Budget Burn
Identify Root Cause
Prevent Recurrence
```

---

## 55. SLO and Incident Severity

SLO impact can help provide context for incident severity.

Example:

```text
Minor SLO Degradation
```

versus:

```text
Major SLO Degradation
```

A large customer-facing SLO violation may indicate a high-severity incident.

The exact severity model should be defined by the organization.

---

## 56. SLO and Postmortem

After an incident:

```text
Incident
 ↓
Postmortem
 ↓
SLO Impact
 ↓
Error Budget Consumed
 ↓
Root Cause
 ↓
Corrective Actions
```

This connects reliability incidents with measurable service performance.

---

## 57. SLO and Capacity Planning

Historical SLO data can reveal trends.

Example:

```text
Traffic ↑
 ↓
Latency ↑
```

If latency gradually approaches the SLO threshold, capacity planning may be required.

SLO trends can therefore help guide:

```text
Scaling
Infrastructure Investment
Architecture Changes
Performance Optimization
```

---

## 58. SLO and Cost

Higher reliability usually requires additional investment.

For example:

```text
99.9%
```

may require less redundancy than:

```text
99.99%
```

A stricter SLO may require:

```text
More Redundancy
More Capacity
More Automation
More Testing
More Operational Effort
```

Therefore SLOs should balance:

```text
Reliability
Cost
Business Value
Risk
```

---

## 59. Do Not Automatically Target 99.99%

A common mistake is assuming:

```text
Higher SLO = Always Better
```

The correct target depends on:

```text
User Expectations
Business Requirements
Historical Performance
Risk
Cost
```

A 99.9% SLO may be completely appropriate for one service while another may require 99.99%.

---

## 60. SLO Precision

Avoid unnecessary precision.

For example:

```text
99.9873%
```

may create false precision if the measurement itself is not that accurate.

Prefer targets that are:

```text
Meaningful
Understandable
Measurable
Defensible
```

---

## 61. SLO Documentation

Every SLO should document:

```text
Service
SLI
Target
Measurement Window
Request Population
Formula
Owner
Data Source
Dashboard
Alerting Policy
```

Example:

```text
Service:
Checkout API

SLI:
Successful Checkout Rate

SLO:
99.9%

Window:
Rolling 30 days

Source:
Prometheus
```

---

## 62. SLO Review

SLOs should be reviewed periodically.

Review them when:

```text
User Expectations Change
Architecture Changes
Traffic Changes
Business Requirements Change
Major Incidents Occur
New Dependencies Are Added
Service Becomes More Critical
```

An SLO that was appropriate in the past may not represent the current service.

---

## 63. SLO Anti-Pattern — Arbitrary Targets

Bad:

```text
Let's use 99.99% because it sounds good.
```

Better:

```text
Analyze:
User Expectations
Business Requirements
Historical Performance
Risk
Cost
```

Then select an appropriate target.

---

## 64. SLO Anti-Pattern — Infrastructure Target

Bad:

```text
CPU SLO = 70%
```

CPU is usually an operational metric rather than a user-facing reliability objective.

Better:

```text
Availability SLO
Latency SLO
Success Rate SLO
```

CPU remains useful for troubleshooting.

---

## 65. SLO Anti-Pattern — Too Many Objectives

A service with:

```text
30 different SLOs
```

can become difficult to manage.

Focus on:

```text
Critical User Journeys
```

and the service characteristics that matter most.

---

## 66. SLO Anti-Pattern — No Measurement Window

Bad:

```text
Availability SLO = 99.9%
```

without specifying:

```text
Over what period?
```

Better:

```text
99.9% availability over a rolling 30-day window.
```

---

## 67. SLO Anti-Pattern — Ignoring User Experience

Bad:

```text
All pods are Running.
```

This does not prove:

```text
Users can successfully use the service.
```

Better:

```text
99.9% successful user requests.
```

---

## 68. SLO and Golden Signals

The traditional Golden Signals are:

```text
Latency
Traffic
Errors
Saturation
```

SLOs commonly focus on user-facing characteristics such as:

```text
Latency
Errors
Availability
```

Traffic and saturation are valuable supporting signals.

---

## 69. SLO and Observability Stack

A practical observability flow:

```text
Application
     ↓
Metrics
     ↓
Prometheus
     ↓
SLI
     ↓
SLO
     ↓
Grafana
```

For investigation:

```text
SLO Degradation
     ↓
ELK
     ↓
Logs
```

For latency and dependency investigation:

```text
Latency Degradation
     ↓
Tracing
     ↓
Dependency Analysis
```

---

## 70. SLO Dashboard

A production SLO dashboard might contain:

```text
Service: Checkout API

Availability SLO:
99.9%

Current:
99.95%

Status:
Healthy
```

Latency:

```text
Target:
p95 <= 500ms

Current:
320ms

Status:
Healthy
```

Error Budget:

```text
Remaining:
65%
```

---

## 71. SLO Status

Basic evaluation:

```text
SLI >= SLO
    ↓
Objective Met
```

```text
SLI < SLO
    ↓
Objective Violated
```

Production reliability monitoring may additionally track:

```text
Error Budget Remaining
Burn Rate
Trend
Risk
```

---

## 72. SLO and Reliability Engineering

SLOs provide a common language between:

```text
Developers
DevOps
SRE
Product
Management
Support
```

Instead of:

```text
"The service feels unstable."
```

the team can say:

```text
"Availability dropped to 99.7%
against a 99.9% SLO."
```

This makes reliability measurable.

---

## 73. SLO and Engineering Priorities

Suppose:

```text
Error Budget Healthy
```

The team can continue normal feature delivery.

If:

```text
Error Budget Low
```

the team may prioritize:

```text
Reliability Improvements
Bug Fixes
Capacity
Automation
Testing
```

This creates a data-driven approach to engineering priorities.

---

## 74. SLO and Release Velocity

SLOs can help balance:

```text
Feature Velocity
Reliability
```

Example:

```text
Healthy Error Budget
 ↓
Normal Release Velocity
```

```text
Low Error Budget
 ↓
Higher Release Caution
```

```text
Exhausted Error Budget
 ↓
Reliability Focus
```

---

## 75. Production Example — API

Suppose:

```text
Service:
Product API

SLO:
99.9% availability

Latency SLO:
95% <= 300ms

Measurement:
Rolling 30 days
```

Current:

```text
Availability = 99.95%
p95 = 250ms
```

Result:

```text
Availability SLO → Met
Latency SLO → Met
```

---

## 76. Production Example — Incident

Before incident:

```text
Availability = 99.97%
```

A deployment occurs.

After deployment:

```text
Availability = 98.9%
```

SLO:

```text
99.9%
```

Result:

```text
SLO Violation
```

Next steps:

```text
Stop Rollout
 ↓
Investigate
 ↓
Check Logs
 ↓
Check Metrics
 ↓
Check Dependencies
 ↓
Rollback if Required
```

---

## 77. Production Example — Canary

Suppose:

```text
Production:
p95 = 200ms

Canary:
p95 = 900ms
```

Target:

```text
p95 <= 300ms
```

The canary is violating the latency objective.

Possible action:

```text
Stop Canary
```

or:

```text
Rollback
```

depending on the deployment strategy.

---

## 78. Production Example — GitOps

Flow:

```text
Developer
 ↓
Git Commit
 ↓
ArgoCD
 ↓
EKS Deployment
 ↓
Prometheus
 ↓
SLI
 ↓
SLO
```

If SLO performance degrades:

```text
SLO Degradation
 ↓
Investigation
 ↓
Revert Git Change
 ↓
ArgoCD Sync
 ↓
Previous Version
```

---

## 79. Production Example — Microservices

Consider:

```text
Frontend
 ↓
User Service
 ↓
Product Service
 ↓
Order Service
 ↓
Payment Service
```

Each service can have:

```text
Availability SLO
Latency SLO
```

The most critical services may have stricter objectives.

An additional end-to-end SLO can measure the complete user journey.

---

## 80. Production Example — Dependency

Suppose:

```text
Order Service
 ↓
Payment Service
 ↓
Database
```

If Payment becomes unavailable:

```text
Order SLI ↓
```

The investigation should identify:

```text
Order Service SLO
Payment Service SLO
Database SLO
```

This helps determine where the reliability problem originated.

---

## 81. Production SLO Checklist

```text
☐ Identify critical user journeys
☐ Define the SLI
☐ Define success criteria
☐ Choose the target
☐ Define measurement window
☐ Define valid request population
☐ Define the formula
☐ Create Prometheus queries
☐ Create Grafana dashboards
☐ Calculate error budget
☐ Define alerting
☐ Define burn-rate monitoring
☐ Document ownership
☐ Review periodically
```

---

## 82. Interview Question — What is an SLO?

**Answer:**

An SLO, or Service Level Objective, is a target level of service reliability or performance that an organization aims to maintain over a defined measurement period.

For example:

```text
99.9% availability over a rolling 30-day window.
```

The SLI provides the actual measurement, while the SLO defines the target.

---

## 83. Interview Question — Difference Between SLI, SLO and SLA

**Answer:**

```text
SLI:
Actual measurement

SLO:
Internal reliability target

SLA:
Formal customer commitment
```

Example:

```text
SLI = 99.95% availability

SLO = 99.9%

SLA = 99.5%
```

---

## 84. Interview Question — Why Should an SLO Be Measurable?

**Answer:**

An SLO should be measurable so the team can objectively determine whether the service is meeting its reliability target.

Instead of:

```text
"Service should be fast."
```

define:

```text
95% of requests should complete within 300ms.
```

Now the target can be measured using observability data.

---

## 85. Interview Question — How Do You Choose an SLO?

**Answer:**

I would consider:

```text
User Expectations
Business Criticality
Historical Performance
Architecture
Dependencies
Risk
Cost
```

I would avoid choosing an arbitrary number simply because it sounds good.

---

## 86. Interview Question — Why Not Make Every Service 99.99%?

**Answer:**

Higher reliability usually requires additional engineering and infrastructure investment.

Not every service has the same business criticality.

For example:

```text
Payment Service
```

may need a stricter objective than:

```text
Internal Reporting Service
```

The SLO should balance:

```text
Reliability
Cost
Risk
Business Value
```

---

## 87. Interview Question — How Does SLO Relate to Error Budget?

**Answer:**

The error budget represents the amount of unreliability allowed by the SLO.

For example:

```text
SLO = 99.9%
```

Therefore:

```text
Error Budget = 0.1%
```

For one million valid requests:

```text
Allowed failures = 1,000
```

---

## 88. Interview Question — What Happens When the Error Budget Is Exhausted?

**Answer:**

The organization may reduce risky releases and prioritize reliability work.

For example:

```text
Error Budget Exhausted
 ↓
Reduce Release Risk
 ↓
Fix Reliability Issues
 ↓
Improve Testing / Capacity / Architecture
```

The exact policy depends on the organization.

---

## 89. Interview Question — How Would You Define an SLO for an EKS Application?

**Answer:**

I would start with user-facing service behavior.

For example:

```text
Availability:
99.9%

Latency:
95% of requests <= 300ms
```

I would measure the SLIs using application metrics collected by Prometheus and visualize the SLOs in Grafana.

Infrastructure metrics such as node and pod CPU would be supporting signals for troubleshooting.

---

## 90. Interview Question — How Can SLOs Help With Deployments?

**Answer:**

I can monitor the SLI during a canary or progressive deployment and compare it against the SLO or baseline.

Example:

```text
Before:
p95 = 200ms

Canary:
p95 = 800ms

Target:
p95 <= 300ms
```

The canary is unhealthy, so I can stop the rollout or roll back.

---

## 91. Interview Question — Should an SLO Be Higher Than an SLA?

**Answer:**

Often, yes.

Example:

```text
Internal SLO = 99.9%

Customer SLA = 99.5%
```

The stricter internal target provides a buffer before the external contractual commitment is violated.

---

## 92. Interview Question — What Is a Rolling SLO?

**Answer:**

A rolling SLO continuously evaluates service performance over a moving time window.

For example:

```text
Rolling 30 days
```

Today:

```text
Previous 30 days
```

Tomorrow:

```text
New previous 30 days
```

The window continuously moves forward.

---

## 93. Interview Question — What Makes a Good SLO?

**Answer:**

A good SLO should be:

```text
User-Centric
Measurable
Relevant
Understandable
Achievable
Actionable
```

It should represent something users actually care about.

---

## 94. Interview Question — How Would You Use SLOs During an Incident?

**Answer:**

I would first determine whether the incident is affecting the service SLO.

Then I would evaluate:

```text
Current SLI
SLO Target
Error Budget
Burn Rate
Customer Impact
```

After that I would investigate logs, metrics, traces, dependencies, and recent deployments to identify and resolve the root cause.

---

## 95. Interview Question — How Can SLOs Help With Release Management?

**Answer:**

SLOs provide an objective signal for determining whether a release is negatively affecting reliability.

For example:

```text
Deploy
 ↓
Canary
 ↓
Monitor SLI
 ↓
Compare Against SLO
 ↓
Healthy → Continue
Unhealthy → Stop / Rollback
```

This allows reliability to be considered directly during deployment.

---

## 96. Final SLO Cheat Sheet

```text
SLO
=
Service Level Objective
```

Meaning:

```text
The target level of service reliability or performance
that should be achieved over a defined measurement period.
```

Relationship:

```text
SLI
 ↓
Actual Measurement

SLO
 ↓
Target

SLA
 ↓
Customer Commitment
```

Example:

```text
SLI:
99.95% availability

SLO:
99.9% availability

SLA:
99.5% availability
```

Error budget:

```text
100% - SLO
```

For:

```text
99.9% SLO
```

the error budget is:

```text
0.1%
```

Common SLOs:

```text
Availability
Request Success Rate
Latency
Error Rate
Data Freshness
Transaction Success
Message Processing
```

Production flow:

```text
User Journey
     ↓
SLI
     ↓
SLO
     ↓
Error Budget
     ↓
Burn Rate
     ↓
Alerting
     ↓
Incident / Reliability Decision
```

Most important interview statement:

```text
An SLO defines the target level of service reliability or
performance over a specific measurement period. The SLI
provides the actual measurement used to determine whether
the service is meeting that objective.
```