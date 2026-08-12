# Error Budgets

## 1. Overview

An **Error Budget** represents the amount of unreliability a service can experience while still meeting its SLO.

The basic relationship is:

```text
Error Budget = 100% - SLO
```

Example:

```text
SLO = 99.9%

Error Budget = 100% - 99.9%
             = 0.1%
```

The error budget gives engineering teams a measurable amount of acceptable failure.

---

## 2. Why Error Budgets Matter

Without an error budget, teams may argue about:

```text
Should we release features faster?
Should we stop deployments?
Should we focus on reliability?
```

Error budgets provide an objective way to make these decisions.

Example:

```text
Healthy Error Budget
        ↓
Normal Feature Development
```

```text
Low Error Budget
        ↓
More Reliability Focus
```

```text
Exhausted Error Budget
        ↓
Reduce Risky Changes
```

---

## 3. Relationship Between SLI, SLO and Error Budget

```text
SLI
 ↓
Actual Performance

SLO
 ↓
Target Performance

Error Budget
 ↓
Allowed Unreliability
```

Example:

```text
SLI = 99.95%

SLO = 99.9%

Error Budget = 0.1%
```

The service is currently performing above the SLO.

---

## 4. Error Budget Formula

For an availability SLO:

```text
Error Budget = 100% - SLO
```

Examples:

```text
SLO = 99%
Error Budget = 1%
```

```text
SLO = 99.5%
Error Budget = 0.5%
```

```text
SLO = 99.9%
Error Budget = 0.1%
```

```text
SLO = 99.99%
Error Budget = 0.01%
```

The higher the SLO, the smaller the error budget.

---

## 5. Error Budget Example

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
1,000,000 requests
```

the allowed failures are approximately:

```text
1,000,000 × 0.001
= 1,000 failures
```

Therefore:

```text
Allowed Failures = 1,000
```

---

## 6. Error Budget With Availability

Suppose a service has:

```text
30-day SLO = 99.9%
```

The error budget represents the amount of downtime or failed service that can occur while still meeting the SLO.

For a 30-day month:

```text
30 × 24 × 60
= 43,200 minutes
```

Allowed unreliability:

```text
43,200 × 0.001
= 43.2 minutes
```

So approximately:

```text
43.2 minutes
```

of unavailability corresponds to a 99.9% availability target over a 30-day period.

---

## 7. Error Budget for Different SLOs

Approximate 30-day availability budgets:

```text
99% SLO
≈ 7 hours 12 minutes

99.5% SLO
≈ 3 hours 36 minutes

99.9% SLO
≈ 43 minutes

99.95% SLO
≈ 22 minutes

99.99% SLO
≈ 4.3 minutes

99.999% SLO
≈ 26 seconds
```

These are approximate values for a 30-day period.

---

## 8. Error Budget Is Not Only Downtime

Error budgets can represent more than availability.

Depending on the SLO, the budget may represent:

```text
Failed Requests
Slow Requests
Failed Transactions
Late Messages
Stale Data
Unsuccessful Jobs
```

Example:

```text
SLO:
99.9% successful requests
```

The error budget represents the allowed unsuccessful requests.

---

## 9. Error Budget for Latency

Suppose:

```text
SLO:
95% of requests must complete within 300ms
```

The remaining:

```text
5%
```

represents the allowed portion of requests that can exceed the defined latency threshold.

The exact error-budget calculation depends on the SLO definition.

---

## 10. Error Budget for Error Rate

Suppose:

```text
SLO:
5xx error rate <= 0.1%
```

Then:

```text
Allowed Error Rate = 0.1%
```

For:

```text
1,000,000 requests
```

approximately:

```text
1,000 errors
```

are allowed.

---

## 11. Error Budget for Successful Requests

Suppose:

```text
SLO = 99.9% success rate
```

Then:

```text
Error Budget = 0.1%
```

If there are:

```text
500,000 valid requests
```

the approximate allowed failures are:

```text
500,000 × 0.001
= 500
```

---

## 12. Remaining Error Budget

Suppose:

```text
Total Error Budget = 1,000 failures
```

The service has experienced:

```text
300 failures
```

Remaining:

```text
1,000 - 300
= 700 failures
```

Therefore:

```text
Remaining Error Budget = 700 failures
```

---

## 13. Error Budget Percentage

Suppose:

```text
Total Budget = 1,000 failures
Consumed = 300 failures
```

Then:

```text
Budget Consumed = 30%
```

and:

```text
Budget Remaining = 70%
```

This is often useful for dashboards.

---

## 14. Error Budget Status

A simple model:

```text
100% Remaining
      ↓
Healthy
```

```text
50% Remaining
      ↓
Monitor
```

```text
10% Remaining
      ↓
High Risk
```

```text
0% Remaining
      ↓
Exhausted
```

The exact thresholds should be defined by the organization.

---

## 15. Healthy Error Budget

When the error budget is healthy:

```text
Error Budget
      ↓
Healthy
      ↓
Normal Release Velocity
```

The team can generally continue:

```text
Feature Development
Deployments
Experiments
Performance Improvements
```

while continuing to monitor reliability.

---

## 16. Low Error Budget

When the error budget becomes low:

```text
Error Budget ↓
       ↓
Risk ↑
```

The team may choose to:

```text
Reduce Risky Deployments
Prioritize Reliability
Investigate Recurring Issues
Improve Testing
Improve Capacity
```

This is a policy decision rather than an automatic rule.

---

## 17. Exhausted Error Budget

When:

```text
Error Budget = 0
```

the service has consumed the allowed unreliability for the measurement period.

A common policy may be:

```text
Stop High-Risk Releases
        ↓
Focus on Reliability
        ↓
Fix Root Causes
        ↓
Improve Testing
        ↓
Improve Architecture
```

The exact policy depends on the organization.

---

## 18. Error Budget Policy

A mature organization can define a policy such as:

```text
Budget > 50%
    ↓
Normal Development
```

```text
Budget 10–50%
    ↓
Increase Reliability Focus
```

```text
Budget < 10%
    ↓
Restrict Risky Changes
```

```text
Budget = 0%
    ↓
Reliability Work Takes Priority
```

These values are examples, not universal standards.

---

## 19. Error Budget and Release Velocity

Error budgets create a balance between:

```text
Reliability
        ↕
Feature Velocity
```

If reliability is healthy:

```text
More Room for Change
```

If reliability is poor:

```text
More Focus on Stability
```

This avoids treating every incident as a reason to permanently slow down development.

---

## 20. Error Budget and DevOps

In a DevOps environment:

```text
Code
 ↓
CI/CD
 ↓
Deployment
 ↓
Monitoring
 ↓
SLI
 ↓
SLO
 ↓
Error Budget
```

The error budget provides feedback on whether deployment velocity is affecting reliability.

---

## 21. Error Budget and CI/CD

A pipeline can use reliability information:

```text
Build
 ↓
Test
 ↓
Security Scan
 ↓
Deploy
 ↓
Monitor
 ↓
Evaluate SLI
 ↓
Evaluate SLO
```

If a deployment causes significant reliability degradation:

```text
Stop
 ↓
Rollback
```

---

## 22. Error Budget and Canary Deployment

Canary deployment:

```text
Small Percentage of Traffic
          ↓
New Version
          ↓
Monitor
          ↓
SLI / SLO
```

If the new version causes excessive errors:

```text
Error Budget Burn ↑
          ↓
Stop Canary
          ↓
Rollback
```

This reduces the potential impact of a bad release.

---

## 23. Error Budget and Blue-Green Deployment

Example:

```text
Blue
 ↓
Current Version

Green
 ↓
New Version
```

After deploying Green:

```text
Monitor SLI
 ↓
Compare With SLO
```

If reliability degrades:

```text
Traffic
 ↓
Blue
```

This helps prevent prolonged customer impact.

---

## 24. Error Budget and GitOps

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
 ↓
Error Budget
```

If a deployment consumes the budget rapidly:

```text
Investigate
 ↓
Revert Git Change
 ↓
ArgoCD Sync
 ↓
Previous Version
```

---

## 25. Error Budget and Burn Rate

Burn rate tells us how quickly the error budget is being consumed.

Example:

```text
Normal Error Consumption
        ↓
Low Burn Rate
```

```text
Rapid Error Consumption
        ↓
High Burn Rate
```

A high burn rate can indicate an active incident.

---

## 26. Fast Burn

Suppose:

```text
SLO = 99.9%
```

and suddenly:

```text
20% requests fail
```

The error budget is being consumed very quickly.

This is:

```text
Fast Burn
```

The team should investigate immediately.

---

## 27. Slow Burn

Suppose error rate gradually increases:

```text
0.10%
0.12%
0.15%
0.18%
```

The budget is slowly being consumed.

This is:

```text
Slow Burn
```

A slow burn may not trigger an immediate major incident but can eventually exhaust the error budget.

---

## 28. Error Budget Burn Example

Suppose:

```text
Monthly Error Budget = 43 minutes
```

An incident consumes:

```text
20 minutes
```

Remaining:

```text
43 - 20
= 23 minutes
```

Another incident consumes:

```text
15 minutes
```

Remaining:

```text
23 - 15
= 8 minutes
```

The service is now operating with very little remaining budget.

---

## 29. Error Budget and Incident Management

During an incident:

```text
Incident
 ↓
Customer Impact
 ↓
SLI Degradation
 ↓
Error Budget Consumption
 ↓
Burn Rate Increase
```

This gives incident responders another way to understand the severity and reliability impact.

---

## 30. Error Budget and Incident Severity

A high burn rate can indicate that the incident is consuming reliability budget quickly.

For example:

```text
Error Rate = 20%
```

may be much more concerning than:

```text
Error Rate = 0.2%
```

because the first scenario can consume the error budget very rapidly.

Incident severity should still consider:

```text
Customer Impact
Business Impact
Scope
Duration
```

---

## 31. Error Budget and Postmortem

After an incident:

```text
Incident
 ↓
Calculate SLO Impact
 ↓
Calculate Error Budget Consumption
 ↓
Identify Root Cause
 ↓
Create Corrective Actions
```

Example:

```text
Incident Duration = 25 minutes
Error Budget Consumed = 58%
```

This shows the reliability impact more clearly than incident duration alone.

---

## 32. Error Budget and Reliability Work

When the budget is repeatedly consumed, engineering should look for systemic problems.

Examples:

```text
Frequent Bad Deployments
Insufficient Capacity
Application Bugs
Database Bottlenecks
Dependency Failures
Poor Alerting
Weak Testing
Configuration Errors
```

The goal is not simply to recover from incidents.

The goal is to reduce future budget consumption.

---

## 33. Error Budget and Technical Debt

Repeated incidents may indicate technical debt.

Example:

```text
Old Application
 ↓
Frequent Failures
 ↓
Error Budget Repeatedly Consumed
 ↓
Reliability Work
 ↓
Refactoring
```

Error budget data can help justify reliability improvements.

---

## 34. Error Budget and Capacity Planning

Suppose:

```text
Traffic ↑
CPU ↑
Latency ↑
```

and:

```text
Error Budget Consumption ↑
```

This may indicate insufficient capacity.

Possible actions:

```text
Increase Pod Replicas
Increase Node Capacity
Tune HPA
Optimize Application
Optimize Database
```

---

## 35. Error Budget and Auto Scaling

Example:

```text
Traffic ↑
 ↓
CPU ↑
 ↓
HPA Scales Pods
 ↓
Latency Stable
 ↓
SLO Maintained
```

If scaling is insufficient:

```text
Traffic ↑
 ↓
CPU ↑
 ↓
Latency ↑
 ↓
SLO Violation
 ↓
Error Budget Burn
```

The error budget provides evidence that scaling or architecture may need improvement.

---

## 36. Error Budget and Kubernetes

For Kubernetes:

```text
User
 ↓
ALB
 ↓
Service
 ↓
Pods
```

Potential causes of budget consumption include:

```text
Pod Crash
OOMKilled
Node Failure
Deployment Failure
Readiness Failure
Network Problems
Application Errors
Database Issues
```

Kubernetes metrics help identify the cause after the SLO detects the impact.

---

## 37. Error Budget and EKS

For EKS:

```text
ALB
 ↓
EKS
 ↓
Pods
 ↓
Application
```

Monitoring can include:

```text
Prometheus
Grafana
ELK
```

Example:

```text
SLO Degradation
 ↓
Prometheus
 ↓
Error Rate
 ↓
Grafana
 ↓
Investigation
 ↓
ELK Logs
```

---

## 38. Error Budget Dashboard

A useful dashboard can show:

```text
Service:
Checkout API

SLO:
99.9%

Current SLI:
99.95%

Error Budget:
0.1%

Budget Consumed:
30%

Budget Remaining:
70%

Burn Rate:
Normal
```

This provides a quick reliability overview.

---

## 39. Error Budget Dashboard for EKS

Example:

```text
Service
 ↓
Availability SLO
 ↓
Current Availability
 ↓
Error Budget Remaining
 ↓
Burn Rate
```

Supporting panels:

```text
Pod CPU
Pod Memory
Pod Restarts
Node CPU
Node Memory
HTTP Errors
Request Latency
```

---

## 40. Error Budget Monitoring Architecture

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
Error Budget
     ↓
Grafana
     ↓
Alerts
```

For investigation:

```text
Alert
 ↓
Metrics
 ↓
Logs
 ↓
Traces
 ↓
Root Cause
```

---

## 41. Error Budget and Prometheus

Prometheus can collect:

```text
Request Count
Successful Requests
Error Count
Request Duration
```

These metrics can be used to calculate:

```text
SLI
 ↓
SLO
 ↓
Error Budget
```

Example conceptual query:

```text
successful_requests
/
total_valid_requests
```

The exact PromQL depends on the application's metric names and labels.

---

## 42. Error Budget and Grafana

Grafana can visualize:

```text
SLO Target
Current SLI
Budget Consumed
Budget Remaining
Burn Rate
Trend
```

Example:

```text
SLO = 99.9%

Current = 99.96%

Budget Remaining = 60%
```

This allows engineers to quickly understand the reliability state.

---

## 43. Error Budget and ELK

ELK is primarily useful for investigating why the budget is being consumed.

Example:

```text
Error Budget Burn
 ↓
Kibana
 ↓
5xx Errors
 ↓
Application Logs
 ↓
Database Timeout
```

Prometheus identifies the reliability degradation.

ELK can help investigate the cause.

---

## 44. Error Budget and Tracing

Tracing can help identify dependency-related budget consumption.

Example:

```text
Request
 ↓
Service A
 ↓
Service B
 ↓
Database
```

If Service B becomes slow:

```text
End-to-End Latency ↑
 ↓
Latency SLO Degradation
 ↓
Error Budget Consumption
```

Tracing helps identify where the delay occurred.

---

## 45. Error Budget and Application Monitoring

For Java:

```text
Request Errors
Request Duration
JVM Metrics
```

For Node.js:

```text
Request Errors
Request Duration
Runtime Metrics
```

For Python:

```text
Request Errors
Request Duration
Runtime Metrics
```

These metrics can contribute to SLI calculations.

---

## 46. Error Budget and Microservices

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

Each service may have its own:

```text
SLO
Error Budget
Burn Rate
```

A critical service may consume its budget faster than others.

This helps identify which service needs attention.

---

## 47. Error Budget and Dependencies

Suppose:

```text
Order Service
 ↓
Payment Service
```

Payment becomes unavailable:

```text
Payment Failure
 ↓
Order Failure
 ↓
Order SLI ↓
 ↓
Order Error Budget Consumed
```

The team should investigate the dependency as part of the incident.

---

## 48. Error Budget and Third-Party APIs

Suppose:

```text
Application
 ↓
External API
```

The external API becomes slow.

Result:

```text
Application Latency ↑
 ↓
SLO Degradation
 ↓
Error Budget Consumption
```

The team should determine whether the issue is:

```text
Application
Dependency
Network
External Provider
```

---

## 49. Error Budget and Deployment Frequency

Error budgets can support a healthy balance between:

```text
Deployment Frequency
Reliability
```

If releases are stable:

```text
Healthy Budget
 ↓
Continue Delivery
```

If releases repeatedly cause failures:

```text
Budget Consumption
 ↓
Review Deployment Process
 ↓
Improve Testing / Canary / Rollback
```

---

## 50. Error Budget and Testing

Repeated budget consumption after deployments may indicate insufficient testing.

Possible improvements:

```text
Unit Testing
Integration Testing
Load Testing
Security Testing
Canary Testing
Smoke Testing
Automated Rollback
```

The objective is to reduce production failures.

---

## 51. Error Budget and Security

Security incidents can also affect availability and reliability.

Examples:

```text
DDoS
Credential Compromise
Malicious Traffic
Security Misconfiguration
```

If the incident causes customer-facing service degradation:

```text
SLI ↓
 ↓
SLO Impact
 ↓
Error Budget Consumption
```

Reliability and security can therefore intersect.

---

## 52. Error Budget and Maintenance

Planned maintenance may or may not consume the budget depending on the SLO definition.

For example:

```text
Scheduled Maintenance
```

may be excluded from availability calculations.

The measurement rules should be documented clearly.

---

## 53. Error Budget and Business Decisions

Error budgets can help product and engineering teams make decisions.

Example:

```text
Feature Release
        vs
Reliability Improvement
```

If the budget is healthy:

```text
Feature Development
```

If the budget is exhausted:

```text
Reliability Work
```

This creates a shared decision-making framework.

---

## 54. Error Budget and Product Teams

Product teams often want:

```text
More Features
Faster Releases
```

Engineering teams need:

```text
Reliability
Stability
Maintainability
```

Error budgets provide a common language:

```text
Healthy Budget
 ↓
More Room for Feature Delivery
```

```text
Exhausted Budget
 ↓
Reliability Takes Priority
```

---

## 55. Error Budget and Management

Management can use error budget trends to understand:

```text
Service Reliability
Release Risk
Operational Health
Engineering Investment
```

Example:

```text
Budget Consumption:
80% every month
```

This may indicate that the current architecture or SLO target needs review.

---

## 56. Error Budget and SLO Review

If the error budget is consistently:

```text
Almost Always Unused
```

the SLO may be too loose.

If the budget is:

```text
Almost Always Exhausted
```

possible explanations include:

```text
SLO Too Strict
Service Reliability Too Low
Architecture Problems
Insufficient Capacity
Unstable Deployments
```

The team should investigate rather than automatically changing the SLO.

---

## 57. Error Budget and SLO Adjustment

Do not change the SLO simply to hide reliability problems.

Bad approach:

```text
SLO = 99.9%
 ↓
Frequent Violations
 ↓
Change SLO to 99%
```

Better:

```text
Investigate Reliability
 ↓
Understand User Expectations
 ↓
Review Business Requirements
 ↓
Review Historical Performance
 ↓
Decide Whether SLO Should Change
```

---

## 58. Error Budget Anti-Pattern — Ignoring the Budget

Bad:

```text
Error Budget Exhausted
```

but the team continues releasing high-risk changes without considering reliability.

Better:

```text
Error Budget Exhausted
 ↓
Review Reliability
 ↓
Prioritize Root Cause Fixes
```

---

## 59. Error Budget Anti-Pattern — Using It as a Punishment

Error budgets should not be used to blame teams.

Bad:

```text
Team consumed budget
 ↓
Team is responsible
```

Better:

```text
Budget Consumed
 ↓
Understand Systemic Cause
 ↓
Improve System
```

Reliability is a shared engineering responsibility.

---

## 60. Error Budget Anti-Pattern — Only Tracking Availability

A service can be available but still provide poor user experience.

Example:

```text
Availability = 99.99%
```

but:

```text
p95 latency = 10 seconds
```

Users may still consider the service unreliable.

Therefore error budgets should reflect meaningful SLOs such as:

```text
Availability
Latency
Success Rate
Transaction Success
```

---

## 61. Error Budget Anti-Pattern — Ignoring Burn Rate

A team may see:

```text
70% Budget Remaining
```

and assume everything is fine.

But if the service is burning the remaining budget extremely quickly:

```text
Burn Rate ↑
```

the budget may soon be exhausted.

Therefore track both:

```text
Budget Remaining
Burn Rate
```

---

## 62. Error Budget Anti-Pattern — No Ownership

Someone should own:

```text
SLO
Error Budget
Dashboard
Alerts
Incident Response
Reliability Improvements
```

Without ownership, budget data may not result in action.

---

## 63. Error Budget Production Example — API

Suppose:

```text
API SLO = 99.9%
```

Therefore:

```text
Error Budget = 0.1%
```

Monthly request volume:

```text
10,000,000 requests
```

Allowed failures:

```text
10,000,000 × 0.001
= 10,000 failures
```

During the month:

```text
Failures = 7,000
```

Remaining:

```text
3,000 failures
```

Budget remaining:

```text
30%
```

---

## 64. Error Budget Production Example — Deployment

Before deployment:

```text
Budget Remaining = 80%
```

New deployment causes:

```text
5xx Errors ↑
Latency ↑
```

After the incident:

```text
Budget Remaining = 35%
```

The deployment consumed a significant portion of the monthly reliability budget.

The team should investigate:

```text
Why did the deployment fail?
Why did testing not catch it?
Can canary analysis improve?
Can rollback be automated?
```

---

## 65. Error Budget Production Example — EKS

Architecture:

```text
User
 ↓
ALB
 ↓
EKS
 ├── Pod A
 ├── Pod B
 └── Pod C
 ↓
RDS
```

Suppose:

```text
SLO = 99.9%
```

A node failure causes:

```text
Pods Unavailable
 ↓
5xx Errors
 ↓
SLI Degradation
 ↓
Error Budget Consumption
```

Recovery:

```text
Replace / Recover Node
 ↓
Pods Rescheduled
 ↓
Traffic Restored
 ↓
Monitor SLI
```

---

## 66. Error Budget Production Example — HPA

Traffic increases:

```text
Traffic ↑
 ↓
CPU ↑
 ↓
HPA Scales
 ↓
Pods Increase
 ↓
Latency Stable
```

Result:

```text
SLO Maintained
Error Budget Protected
```

If HPA cannot scale fast enough:

```text
Latency ↑
 ↓
SLO Violation
 ↓
Error Budget Burn
```

---

## 67. Error Budget Production Example — Database

Suppose:

```text
Application SLO = 99.9%
```

Database connections become exhausted:

```text
DB Connections ↑
 ↓
Requests Timeout
 ↓
5xx Errors ↑
 ↓
Availability ↓
 ↓
Error Budget Consumption
```

The team investigates:

```text
Connection Pool
Database Capacity
Slow Queries
Traffic
Application Behavior
```

---

## 68. Error Budget Production Example — External API

Application:

```text
Order Service
 ↓
External Payment API
```

External API becomes slow:

```text
Payment Latency ↑
 ↓
Order Latency ↑
 ↓
Order SLO Degradation
 ↓
Error Budget Burn
```

Tracing and logs can help confirm the dependency impact.

---

## 69. Error Budget Production Example — Incident

Suppose:

```text
Monthly Error Budget = 43 minutes
```

Incident:

```text
25 minutes downtime
```

Remaining:

```text
43 - 25
= 18 minutes
```

The team has consumed approximately:

```text
25 / 43
≈ 58%
```

of the availability budget.

This incident should trigger a reliability review if the remaining budget is now considered risky.

---

## 70. Error Budget Production Example — Multiple Incidents

Suppose:

```text
Monthly Budget = 43 minutes
```

Incident 1:

```text
10 minutes
```

Incident 2:

```text
15 minutes
```

Incident 3:

```text
12 minutes
```

Total:

```text
10 + 15 + 12
= 37 minutes
```

Remaining:

```text
43 - 37
= 6 minutes
```

The service is close to exhausting its monthly budget.

---

## 71. Error Budget Production Example — Budget Exhaustion

Suppose:

```text
Monthly Budget = 43 minutes
```

Total downtime:

```text
50 minutes
```

Then:

```text
Budget Exhausted
```

and the service has exceeded the allowed unreliability for the SLO period.

The organization may now prioritize:

```text
Reliability Improvements
Root Cause Fixes
Risk Reduction
```

---

## 72. Error Budget Production Example — Canary

Current version:

```text
p95 = 200ms
```

New version:

```text
p95 = 700ms
```

SLO:

```text
p95 <= 300ms
```

Result:

```text
Canary SLO Failure
 ↓
Stop Rollout
 ↓
Investigate
 ↓
Rollback
```

This protects the remaining error budget.

---

## 73. Error Budget Production Example — GitOps

Flow:

```text
Git Commit
 ↓
ArgoCD
 ↓
EKS
 ↓
New Version
 ↓
Prometheus
 ↓
SLO
 ↓
Error Budget
```

If the new version causes a large budget burn:

```text
Revert Git Commit
 ↓
ArgoCD Sync
 ↓
Previous Version
```

---

## 74. Error Budget and Production Architecture

A reliability-oriented architecture may include:

```text
Route 53
 ↓
ALB
 ↓
EKS
 ↓
Multiple Pods
 ↓
RDS
```

Supporting components:

```text
Auto Scaling
Health Checks
Rolling Deployment
Monitoring
Alerting
Backup
Disaster Recovery
```

The architecture helps protect the error budget, but actual SLI measurements determine whether the SLO is achieved.

---

## 75. Error Budget and Observability Architecture

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
Error Budget
     ↓
Grafana
```

Investigation:

```text
Error Budget Burn
     ↓
Metrics
     ↓
Logs
     ↓
Traces
     ↓
Root Cause
```

---

## 76. Error Budget Production Checklist

```text
☐ Define SLO
☐ Calculate Error Budget
☐ Define Measurement Window
☐ Define Valid Request Population
☐ Calculate Budget Consumption
☐ Track Remaining Budget
☐ Track Burn Rate
☐ Create Grafana Dashboard
☐ Configure Alerts
☐ Define Release Policy
☐ Define Budget Exhaustion Policy
☐ Review Incidents
☐ Review Budget Trends
☐ Prioritize Reliability Improvements
```

---

## 77. Interview Question — What Is an Error Budget?

**Answer:**

An error budget is the amount of unreliability allowed by an SLO.

For example:

```text
SLO = 99.9%

Error Budget = 0.1%
```

It gives engineering teams a measurable amount of failure they can tolerate while still meeting the reliability objective.

---

## 78. Interview Question — How Do You Calculate an Error Budget?

**Answer:**

For an availability-based SLO:

```text
Error Budget = 100% - SLO
```

For example:

```text
SLO = 99.9%

Error Budget = 0.1%
```

For one million requests:

```text
1,000,000 × 0.001
= 1,000 allowed failures
```

---

## 79. Interview Question — What Happens When the Error Budget Is Exhausted?

**Answer:**

When the error budget is exhausted, the team may reduce risky changes and prioritize reliability work.

For example:

```text
Error Budget Exhausted
 ↓
Reduce High-Risk Releases
 ↓
Investigate Root Causes
 ↓
Improve Testing
 ↓
Improve Capacity / Architecture
```

The exact policy depends on the organization.

---

## 80. Interview Question — What Is Burn Rate?

**Answer:**

Burn rate describes how quickly a service is consuming its error budget.

A high burn rate means the service is consuming the budget faster than expected.

Example:

```text
Normal Error Consumption
 ↓
Low Burn Rate
```

```text
Large Incident
 ↓
Rapid Error Consumption
 ↓
High Burn Rate
```

---

## 81. Interview Question — What Is Fast Burn vs Slow Burn?

**Answer:**

Fast burn means the service is consuming the error budget very quickly, usually due to a severe incident.

Slow burn means the service is gradually consuming the budget over a longer period.

Example:

```text
Fast Burn:
Large spike in 5xx errors

Slow Burn:
Small but continuous increase in errors
```

Both should be monitored.

---

## 82. Interview Question — How Does Error Budget Affect Deployments?

**Answer:**

If the error budget is healthy, the team can generally maintain normal release velocity.

If the budget is low or exhausted, the team may reduce risky deployments and prioritize reliability.

Example:

```text
Healthy Budget
 ↓
Normal Deployment

Low Budget
 ↓
More Caution

Exhausted Budget
 ↓
Reliability Focus
```

---

## 83. Interview Question — How Would You Protect the Error Budget During a Deployment?

**Answer:**

I would use:

```text
Automated Testing
Canary Deployment
Progressive Delivery
Monitoring
SLO Checks
Burn-Rate Monitoring
Automated Rollback
```

Example:

```text
Deploy Canary
 ↓
Monitor SLI
 ↓
Check SLO
 ↓
Healthy → Increase Traffic
Unhealthy → Rollback
```

---

## 84. Interview Question — How Would You Monitor Error Budget in Kubernetes?

**Answer:**

I would collect application-level metrics using Prometheus and calculate the relevant SLI and SLO.

Then I would visualize:

```text
SLO
Current SLI
Budget Remaining
Budget Consumed
Burn Rate
```

in Grafana.

I would use Kubernetes metrics and logs to investigate the cause of budget consumption.

---

## 85. Interview Question — What If the Error Budget Is Always Exhausted?

**Answer:**

I would not immediately increase the error budget or lower the SLO.

I would investigate:

```text
Application Reliability
Deployment Failures
Capacity
Dependencies
Database
Infrastructure
Monitoring
```

Then determine whether:

```text
The service is unreliable
```

or:

```text
The SLO itself needs to be reviewed
```

The SLO should only be changed based on user and business requirements.

---

## 86. Interview Question — Can Error Budgets Be Used for Latency?

**Answer:**

Yes.

For example:

```text
SLO:
95% of requests <= 300ms
```

The error budget represents the allowed portion of requests that can exceed the defined latency objective.

Error budgets can therefore apply to:

```text
Availability
Latency
Success Rate
Data Freshness
Transaction Processing
```

---

## 87. Interview Question — Why Are Error Budgets Useful?

**Answer:**

Error budgets provide a measurable way to balance:

```text
Reliability
```

with:

```text
Development Velocity
```

Instead of making release decisions based only on opinions, teams can use reliability data to determine how much risk they can reasonably take.

---

## 88. Interview Question — How Does Error Budget Support DevOps?

**Answer:**

Error budgets connect development and operations.

```text
Development
 ↓
Deployment
 ↓
Production
 ↓
Monitoring
 ↓
SLO
 ↓
Error Budget
```

If releases repeatedly consume the budget, the team can improve:

```text
Testing
Deployment Strategy
Rollback
Architecture
Capacity
```

---

## 89. Interview Question — What Metrics Would You Put on an Error Budget Dashboard?

**Answer:**

I would include:

```text
SLO Target
Current SLI
Error Budget Total
Error Budget Consumed
Error Budget Remaining
Burn Rate
Error Rate
Latency
Availability
SLO Trend
```

For Kubernetes services, I would also include supporting infrastructure metrics.

---

## 90. Interview Question — How Does Error Budget Help During Incidents?

**Answer:**

It helps quantify the reliability impact.

During an incident I would look at:

```text
Current SLI
SLO Target
Budget Consumed
Budget Remaining
Burn Rate
Customer Impact
```

This helps the team understand how seriously the incident is affecting reliability.

---

## 91. Final Error Budget Cheat Sheet

```text
Error Budget
=
Allowed Unreliability
```

Basic formula:

```text
Error Budget = 100% - SLO
```

Example:

```text
SLO = 99.9%

Error Budget = 0.1%
```

For:

```text
1,000,000 requests
```

approximately:

```text
1,000 failures
```

are allowed.

Relationship:

```text
SLI
 ↓
Actual Performance

SLO
 ↓
Target

Error Budget
 ↓
Allowed Unreliability

Burn Rate
 ↓
Speed of Budget Consumption
```

Production reliability flow:

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
Error Budget
     ↓
Burn Rate
     ↓
Alerting
     ↓
Incident Response
     ↓
Reliability Improvements
```

Key principle:

```text
A healthy error budget gives teams room to move fast.

A low error budget signals increasing reliability risk.

An exhausted error budget means reliability work should
become a priority according to the organization's policy.
```

Most important interview statement:

```text
An error budget is the amount of unreliability allowed by
an SLO. It provides a quantitative way to balance service
reliability with engineering and release velocity.
```