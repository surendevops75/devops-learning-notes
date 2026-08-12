# Alert Design

## 1. Overview

Alert design is the process of deciding:

```text
What should trigger an alert?
When should it trigger?
How severe is it?
Who should receive it?
What information should it contain?
What action should the engineer take?
```

A good alert is not simply a metric crossing a threshold.

A good alert represents a condition that requires human attention.

The objective is:

```text
Meaningful Signal
      ↓
Actionable Alert
      ↓
Correct Notification
      ↓
Fast Investigation
      ↓
Fast Recovery
```

Poor alert design creates:

```text
Alert Noise
Alert Fatigue
Missed Incidents
Slow Response
Unclear Ownership
```

---

# 2. Alert Design Principles

A production alert should follow these principles:

```text
Actionable
Relevant
Specific
Reliable
Understandable
Owned
Prioritized
Context-Rich
```

The engineer receiving the alert should immediately understand:

```text
What is wrong?
Where is it happening?
How serious is it?
Is the customer affected?
What should I check?
Who owns the service?
```

---

# 3. Monitoring vs Alert Design

Monitoring provides visibility:

```text
CPU = 92%
```

Alert design determines whether this should become an alert.

For example:

```text
CPU > 90%
for 10 minutes
```

But even this may not always be the best alert.

If the application is healthy and CPU spikes are normal:

```text
CPU > 90%
```

may not require paging.

A better alert might be based on:

```text
User Impact
+
Sustained Resource Saturation
```

---

# 4. Alert on Symptoms, Not Only Causes

A mature alerting strategy should prioritize user-visible symptoms.

Example:

```text
Application
    ↓
HTTP 5xx Rate ↑
    ↓
Users Experiencing Errors
    ↓
CRITICAL ALERT
```

This is often more valuable than:

```text
CPU > 90%
```

because high CPU does not necessarily mean users are affected.

Use both when appropriate:

```text
Symptom Alert
+
Cause Alert
```

---

# 5. Symptom-Based Alerting

A symptom alert represents a user-visible problem.

Examples:

```text
High HTTP 5xx Rate
High Request Latency
No Healthy Load Balancer Targets
Service Unavailable
Failed Transactions
Queue Backlog Affecting Processing
```

Example:

```text
5xx Error Rate
      ↓
8%
      ↓
Threshold = 5%
      ↓
Alert
```

The engineer knows immediately that the application is experiencing failures.

---

# 6. Cause-Based Alerting

Cause alerts detect infrastructure or dependency problems.

Examples:

```text
High CPU
High Memory
Disk Almost Full
Database Connection Exhaustion
Replication Lag
Node NotReady
High Network Packet Loss
```

Example:

```text
Database Connections
        ↓
95%
        ↓
Alert
```

This may help identify the root cause before users experience a complete outage.

---

# 7. Symptom + Cause Model

A strong alerting architecture uses both:

```text
                Production System
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
        User Symptoms          Causes
             │                   │
          5xx ↑              CPU ↑
          Latency ↑          Memory ↑
          Availability ↓     DB Connections ↑
             │                   │
             └─────────┬─────────┘
                       ↓
                  Investigation
```

The symptom tells you:

```text
Something is wrong.
```

The cause alert may tell you:

```text
Where to look.
```

---

# 8. Alert Actionability

Every alert should have a clear expected action.

Bad:

```text
CPU High
```

Better:

```text
CRITICAL
Production payment service CPU has remained above 90% for 15 minutes.
Check current traffic, expensive requests, recent deployments, and pod/node saturation.
```

The second alert provides an operational starting point.

---

# 9. Alert Thresholds

Thresholds define when an alert condition becomes true.

Example:

```text
CPU > 90%
```

Other examples:

```text
Memory > 90%
Disk > 85%
5xx Error Rate > 5%
Latency p95 > 1 second
Connections > 90%
Replication Lag > 60 seconds
```

Thresholds should be based on:

```text
Historical Behavior
Capacity
SLO
User Impact
Workload
Recovery Requirements
```

Avoid selecting thresholds arbitrarily.

---

# 10. Static Thresholds

A static threshold uses a fixed value.

Example:

```text
CPU > 90%
```

Advantages:

```text
Simple
Easy to understand
Easy to implement
```

Disadvantages:

```text
May create noise
May not work for variable workloads
May not represent user impact
```

Static thresholds are useful for straightforward resource limits.

---

# 11. Dynamic Thresholds

Dynamic thresholds consider normal system behavior.

For example:

```text
Normal Request Rate
      ↓
1,000 req/s

Current
      ↓
5,000 req/s
```

Instead of using a fixed threshold, the alert can detect unusual deviation from normal behavior.

Dynamic alerting can be useful when:

```text
Traffic Changes Throughout the Day
Workload Is Highly Variable
Seasonality Exists
Normal Baseline Changes
```

---

# 12. Threshold Selection

Consider a disk example.

Suppose:

```text
Disk Usage = 80%
```

This may be acceptable if disk growth is slow.

But:

```text
Disk Usage = 80%
Growth = 10% per hour
```

could be dangerous.

Therefore alert design should consider:

```text
Current Value
+
Rate of Change
+
Available Recovery Time
```

---

# 13. Alert Duration

Use a duration to avoid transient alerts.

Example:

```yaml
for: 10m
```

Flow:

```text
Condition True
      ↓
Pending
      ↓
Condition Remains True
      ↓
10 Minutes
      ↓
Firing
```

Without a duration:

```text
Brief Spike
   ↓
Alert
   ↓
Noise
```

---

# 14. Choosing the Right Duration

Different conditions require different durations.

Example:

```text
Database Down
    ↓
Short duration

CPU Saturation
    ↓
Longer duration

Disk Growth
    ↓
Longer duration

HTTP 5xx Spike
    ↓
Short but sustained duration
```

Do not use one duration for every alert.

The duration should match:

```text
How quickly the condition becomes dangerous
+
How long transient behavior is normal
```

---

# 15. Alert Severity

Severity determines how urgently an alert should be handled.

Example:

```text
INFO
WARNING
CRITICAL
```

A common model:

```text
INFO
 ↓
No immediate action

WARNING
 ↓
Investigation required

CRITICAL
 ↓
Immediate response required
```

Severity should be based on:

```text
Impact
Urgency
Customer Effect
Recovery Requirements
```

---

# 16. Severity Should Not Be Based Only on Metric Value

Example:

```text
CPU = 95%
```

This could be:

```text
WARNING
```

if the application is healthy.

But:

```text
CPU = 95%
+
Application Error Rate = 15%
```

could be:

```text
CRITICAL
```

Therefore:

```text
Severity = Impact + Urgency
```

not simply:

```text
Severity = Metric Value
```

---

# 17. Alert Labels

Labels provide structured metadata.

Example:

```yaml
labels:
  severity: critical
  service: payment
  environment: production
  team: platform
```

Labels can be used for:

```text
Routing
Grouping
Filtering
Silencing
Dashboards
Incident Analysis
```

---

# 18. Essential Alert Labels

Useful labels include:

```text
severity
service
environment
team
region
cluster
namespace
instance
```

Example:

```yaml
labels:
  severity: critical
  service: payment
  environment: production
  cluster: production-eks
  namespace: payment
  team: platform
```

Avoid unnecessary labels that create excessive alert cardinality or fragmentation.

---

# 19. Alert Annotations

Annotations contain human-readable context.

Example:

```yaml
annotations:
  summary: "High payment API error rate"
  description: "5xx error rate is above 5% for 5 minutes."
```

Useful annotations include:

```text
summary
description
runbook_url
dashboard_url
```

---

# 20. Alert Summary

The summary should be short.

Good:

```text
High payment API error rate
```

Bad:

```text
Something appears to be wrong with the payment service because a metric exceeded a threshold
```

The summary should be readable in:

```text
Slack
PagerDuty
Email
Incident Dashboard
```

---

# 21. Alert Description

The description should provide enough context to begin investigation.

Example:

```yaml
description: |
  Payment API 5xx error rate is above 5%
  for more than 5 minutes in production.
```

A useful description can include:

```text
Observed Value
Expected Value
Affected Service
Environment
Duration
Potential Impact
```

---

# 22. Runbook Integration

Every important production alert should ideally have a runbook.

Example:

```text
Alert
 ↓
Runbook
 ↓
Troubleshooting Steps
 ↓
Commands
 ↓
Expected Results
 ↓
Remediation
```

Example:

```yaml
runbook_url: "Internal Payment API Runbook"
```

A runbook should explain:

```text
What does this alert mean?
What should I check first?
What commands should I run?
What are common causes?
How can I mitigate?
When should I escalate?
```

---

# 23. Dashboard Integration

The alert should link to the relevant dashboard.

Example:

```text
Alert
 ↓
Grafana Dashboard
 ↓
CPU
Memory
Requests
Latency
Errors
Database
```

This allows the engineer to move from:

```text
Detection
```

to:

```text
Investigation
```

quickly.

---

# 24. Alert Context

A strong alert contains enough context to avoid unnecessary investigation before basic diagnosis.

Example:

```text
CRITICAL

Service: Payment
Environment: Production
Cluster: EKS-Prod
Namespace: payment

Problem:
HTTP 5xx rate = 12%

Threshold:
5%

Duration:
8 minutes

Impact:
Payment requests are failing.

Dashboard:
Payment Dashboard

Runbook:
Payment Incident Runbook
```

---

# 25. Alert Ownership

Every alert should have a responsible team.

Example:

```text
Payment Service
      ↓
Payment Team

Database
      ↓
Database / Platform Team

EKS Node
      ↓
Platform Team
```

Ownership should be represented in alert labels.

Example:

```yaml
labels:
  team: platform
```

---

# 26. Alert Routing

Alert routing determines where notifications go.

Example:

```text
                Alert
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
     Critical   Warning     Info
        │         │          │
        ↓         ↓          ↓
    PagerDuty   Slack      Email
```

Routing can also depend on:

```text
Team
Service
Environment
Region
Severity
```

---

# 27. Alert Grouping

Multiple alerts can originate from one incident.

Example:

```text
Node Failure
     │
     ├── Pod Unhealthy
     ├── Deployment Unavailable
     ├── Service Errors
     └── Application Latency
```

Without grouping:

```text
4 Alerts
```

With grouping:

```text
1 Incident Notification
```

Grouping reduces notification noise.

---

# 28. Alert Grouping Strategy

Alerts can be grouped by:

```text
Alert Name
Cluster
Namespace
Service
Team
Incident
```

Example:

```yaml
group_by:
  - alertname
  - cluster
  - namespace
```

The grouping strategy should match how engineers investigate incidents.

---

# 29. Alert Deduplication

Deduplication prevents repeated notifications for the same alert.

Example:

```text
Prometheus A
    ↓
HighErrorRate

Prometheus B
    ↓
HighErrorRate
```

Without deduplication:

```text
2 notifications
```

With deduplication:

```text
1 notification
```

This is especially important in highly available monitoring architectures.

---

# 30. Alert Inhibition

Inhibition suppresses secondary alerts when a primary alert explains them.

Example:

```text
Node Down
   ↓
Pods Unhealthy
   ↓
Application Unavailable
```

Instead of:

```text
Node Alert
Pod Alerts
Application Alert
```

the system can prioritize:

```text
Node Down
```

and inhibit dependent alerts.

---

# 31. Alert Silencing

Silencing temporarily prevents notifications.

Use cases:

```text
Planned Maintenance
Known Incident
Controlled Testing
Migration
```

Example:

```text
01:00 - 02:00
Maintenance
     ↓
Silence affected alerts
```

Silences should have:

```text
Owner
Reason
Start Time
End Time
Scope
```

---

# 32. Silencing Best Practice

Do not use silencing to hide recurring problems.

Bad:

```text
Alert is noisy
 ↓
Silence permanently
```

Better:

```text
Alert is noisy
 ↓
Investigate
 ↓
Fix Alert Rule
 ↓
Tune Threshold
 ↓
Review
```

Silence is temporary operational control, not a permanent solution.

---

# 33. Alert Noise

Alert noise is one of the biggest alerting problems.

Example:

```text
CPU 90%
 ↓
Alert

CPU 89%
 ↓
Resolved

CPU 91%
 ↓
Alert

CPU 89%
 ↓
Resolved
```

This can repeat continuously.

The engineer receives:

```text
Alert
Resolve
Alert
Resolve
Alert
Resolve
```

This creates fatigue.

---

# 34. Alert Fatigue

Alert fatigue occurs when engineers receive too many alerts and begin treating them as background noise.

Consequences:

```text
Ignored Alerts
Delayed Response
Missed Critical Incidents
Burnout
Reduced Trust in Monitoring
```

The solution is:

```text
Fewer
+
Better
+
Actionable
Alerts
```

---

# 35. Alert Quality Formula

A useful mental model:

```text
Alert Quality
=
Actionability
+
Accuracy
+
Context
+
Ownership
+
Priority
```

An alert with no action:

```text
Low Quality
```

An alert with:

```text
Clear Problem
+
Impact
+
Owner
+
Runbook
+
Dashboard
```

is much more useful.

---

# 36. Alert on SLOs

Service Level Objectives can provide strong alert signals.

Example:

```text
SLO:
99.9% successful requests
```

If failures increase:

```text
Error Rate ↑
      ↓
Error Budget Consumption ↑
      ↓
SLO Risk ↑
      ↓
Alert
```

SLO-based alerts focus more directly on service reliability.

---

# 37. Error Budget

Suppose:

```text
SLO = 99.9%
```

Allowed failure:

```text
0.1%
```

This represents the error budget.

If the service consumes the budget too quickly:

```text
Error Budget Burn
       ↓
Risk of SLO Violation
       ↓
Alert
```

This is often more meaningful than a raw CPU alert.

---

# 38. Burn Rate Alerting

Burn rate measures how quickly the error budget is being consumed.

Example:

```text
Normal Burn Rate
      ↓
1x

Fast Burn
      ↓
10x
```

A high burn rate indicates that the service could violate its SLO quickly.

Conceptually:

```text
Error Rate
    ↓
Error Budget
    ↓
Burn Rate
    ↓
Alert
```

---

# 39. Golden Signal Alert Design

Use the Four Golden Signals:

```text
Latency
Traffic
Errors
Saturation
```

Example:

```text
Latency
  ↓
p95 > 1s

Errors
  ↓
5xx > 5%

Traffic
  ↓
Unexpected spike/drop

Saturation
  ↓
CPU / Memory / Connections near limits
```

This provides balanced application alert coverage.

---

# 40. Latency Alert Design

Avoid:

```text
Average latency > 1 second
```

without understanding workload behavior.

Consider:

```text
p95
p99
```

Example:

```text
p95 latency > 1 second
for 10 minutes
```

This better represents the slower portion of user requests.

---

# 41. Error Alert Design

Use error rate rather than only error count where possible.

Bad:

```text
100 errors
```

Context is missing.

Better:

```text
100 errors
out of 500 requests
=
20% error rate
```

Compared with:

```text
100 errors
out of 1,000,000 requests
=
0.01%
```

The error rate provides better context.

---

# 42. Traffic Alert Design

Traffic anomalies can reveal:

```text
Traffic Spike
Traffic Drop
Bot Activity
DDoS
Broken Client
Deployment Issue
Dependency Failure
```

Example:

```text
Expected:
1,000 req/s

Current:
50 req/s
```

A sudden traffic drop may indicate:

```text
DNS Problem
Load Balancer Problem
Application Routing Problem
Client Failure
```

---

# 43. Saturation Alert Design

Saturation alerts should identify resources approaching limits.

Examples:

```text
CPU
Memory
Disk
Database Connections
Thread Pool
Connection Pool
Queue
```

Example:

```text
Database Connection Pool
      ↓
95% utilized
      ↓
Warning
```

If:

```text
100%
```

and requests are failing:

```text
Critical
```

---

# 44. Multi-Condition Alerts

Some alerts become more meaningful when multiple conditions are combined.

Example:

```text
CPU > 90%
AND
HTTP Error Rate > 5%
```

This can reduce false positives when high CPU is normal.

However, avoid making alert expressions unnecessarily complex.

The alert should remain:

```text
Understandable
Maintainable
Actionable
```

---

# 45. Rate-Based Alerts

Rates are often more useful than absolute counters.

Example:

```text
http_requests_total
```

is a counter.

To understand recent request rate:

```promql
rate(http_requests_total[5m])
```

For errors:

```promql
rate(http_requests_total{status=~"5.."}[5m])
```

Alerting on rates helps identify current system behavior.

---

# 46. Percentage-Based Alerts

Percentages often provide better context.

Example:

```text
Error Rate =

5xx Requests
--------------
Total Requests
```

PromQL concept:

```promql
(
  sum(rate(http_requests_total{status=~"5.."}[5m]))
  /
  sum(rate(http_requests_total[5m]))
) * 100
```

Then:

```text
> 5%
```

can trigger the alert.

---

# 47. Avoid Alerting on Raw Counters

Suppose:

```text
http_errors_total = 1,000,000
```

This number alone does not tell us whether the system is currently unhealthy.

The counter could have accumulated over several months.

Instead use:

```promql
rate(http_errors_total[5m])
```

or calculate an error percentage.

---

# 48. Alert on Availability

Availability is often one of the most important signals.

Example:

```text
Service Health Check
        ↓
Failed
        ↓
No Healthy Targets
        ↓
Critical Alert
```

This is usually more actionable than:

```text
CPU > 80%
```

because it directly indicates service availability.

---

# 49. Alert Dependencies Carefully

Suppose:

```text
Database Down
```

causes:

```text
Payment Service Down
Order Service Down
Inventory Service Down
```

You may receive:

```text
Database Alert
Payment Alert
Order Alert
Inventory Alert
```

A dependency-aware alert design should identify:

```text
Primary Failure
```

and avoid unnecessary secondary notifications.

---

# 50. Alert Dependency Model

```text
Database
   ↓
Payment
   ↓
Orders
   ↓
Checkout
```

If database fails:

```text
Database Failure
      ↓
Primary Incident
      ↓
Dependent Alerts
      ↓
Inhibit / Group
```

This helps engineers focus on the root problem.

---

# 51. Alert Design for Kubernetes

Kubernetes alerting should consider:

```text
Node
Pod
Deployment
Service
Application
Cluster
```

Example:

```text
Node NotReady
   ↓
Pod Scheduling Problems
   ↓
Deployment Replicas Unavailable
   ↓
Application Errors
```

The alerting system should distinguish:

```text
Root Infrastructure Problem
```

from:

```text
Downstream Symptoms
```

---

# 52. Pod Alert Design

Avoid alerting immediately on every pod restart.

Example:

```text
Pod Restart = 1
```

may be normal.

Better:

```text
Pod restart rate is continuously increasing
```

or:

```text
Container repeatedly entering CrashLoopBackOff
```

This focuses on persistent problems.

---

# 53. Node Alert Design

A node becoming unavailable is generally important.

Example:

```text
Node Ready = False
for 5 minutes
```

Possible impact:

```text
Pods Evicted
Scheduling Failure
Reduced Capacity
Application Degradation
```

Node alerts should include:

```text
Cluster
Node
Environment
Severity
Runbook
```

---

# 54. Database Alert Design

Example:

```text
Database Connections > 90%
for 10 minutes
```

Include:

```text
Database
Environment
Current Connections
Maximum Connections
Service
Runbook
```

The engineer can then investigate:

```text
Connection Pool
Slow Queries
Long Transactions
Traffic
Connection Leaks
```

---

# 55. Disk Alert Design

Instead of only:

```text
Disk > 90%
```

consider:

```text
Disk Available < 15%
AND
Growth Rate Indicates Less Than N Hours Remaining
```

This can provide more useful early warning.

The exact expression depends on workload and operational requirements.

---

# 56. Certificate Expiry Alert

Certificates should be monitored before expiration.

Example:

```text
Certificate
     ↓
30 days remaining
     ↓
Warning

7 days remaining
     ↓
Critical
```

This gives the team time to renew.

Certificate alerts should include:

```text
Domain
Environment
Expiration Date
Owner
Renewal Procedure
```

---

# 57. Queue Alert Design

For asynchronous systems:

```text
Producer
   ↓
Queue
   ↓
Consumer
```

Monitor:

```text
Queue Depth
Message Age
Consumer Rate
Producer Rate
Failed Messages
```

Example:

```text
Queue Depth ↑
+
Oldest Message Age ↑
```

may indicate consumer problems.

---

# 58. Queue Saturation

Example:

```text
Producer Rate = 1,000 msg/s
Consumer Rate = 500 msg/s
```

Then:

```text
Queue
 ↓
Backlog Increasing
```

The alert should focus on:

```text
Backlog Growth
+
Message Age
```

rather than only queue size.

---

# 59. Alert Design During Deployments

Deployments can temporarily create:

```text
Pod Restarts
CPU Spikes
Latency
Connection Changes
Health Check Failures
```

Alert design should distinguish:

```text
Expected Deployment Behavior
```

from:

```text
Actual Service Degradation
```

Use:

```text
Deployment Events
+
Alerting
+
Observability
```

to correlate changes.

---

# 60. Deployment-Aware Alerting

Example:

```text
Deployment
    ↓
Application Restart
    ↓
Temporary Error Spike
    ↓
Recovery
```

If the error spike is expected and very short:

```text
No Critical Page
```

If errors remain elevated:

```text
Critical Alert
```

This is another reason evaluation duration matters.

---

# 61. Alert Testing Strategy

Test the complete path:

```text
Metric
 ↓
Rule
 ↓
Prometheus
 ↓
Alertmanager
 ↓
Routing
 ↓
Notification
 ↓
Engineer
```

Validate:

```text
Alert Fires
Alert Contains Context
Correct Team Receives It
Correct Severity
Runbook Works
Dashboard Works
Alert Resolves
```

---

# 62. Alert Failure Testing

Monitoring itself can fail.

Consider:

```text
Prometheus Down
Alertmanager Down
Exporter Down
Notification Channel Down
```

Therefore monitor the monitoring system.

Example:

```text
Prometheus
    ↓
Monitoring Health
    ↓
Alert
```

A production observability system should not silently fail.

---

# 63. Watchdog Alert

A watchdog-style alert can confirm that the alerting pipeline itself is alive.

Conceptually:

```text
Prometheus
   ↓
Watchdog
   ↓
Alertmanager
   ↓
Notification System
```

If the expected watchdog signal disappears, the monitoring pipeline may require investigation.

---

# 64. Alert Recovery

An alert should automatically resolve when the underlying condition returns to normal.

Example:

```text
CPU > 90%
      ↓
Alert Firing
      ↓
CPU = 60%
      ↓
Condition False
      ↓
Alert Resolved
```

Do not require manual intervention to resolve alerts that can be automatically evaluated.

---

# 65. Alert Resolution Notification

For important incidents, notify when the alert resolves.

Example:

```text
CRITICAL
Payment API 5xx > 10%
```

Later:

```text
RESOLVED
Payment API 5xx returned to normal.
```

This provides closure to the incident.

---

# 66. Alert Review Process

Regularly review:

```text
Which alerts fired?
Which were actionable?
Which were noisy?
Which were ignored?
Which caused pages?
Which had no owner?
Which had outdated runbooks?
```

Then:

```text
Tune
Remove
Consolidate
Document
Assign Ownership
```

---

# 67. Alert Metrics

The alerting system itself should be measured.

Useful metrics:

```text
Number of Alerts
Alerts per Service
Alerts per Team
Critical Alerts
Warning Alerts
Duplicate Alerts
Noisy Alerts
Acknowledgement Time
MTTD
MTTR
```

Track trends over time.

---

# 68. Alert-to-Incident Ratio

Not every alert should become a major incident.

For example:

```text
100 Alerts
 ↓
20 Require Human Action
 ↓
5 Become Incidents
```

If almost every alert becomes an incident, alerting may be too aggressive.

If almost no alerts require action, the system may be too passive.

---

# 69. Paging Policy

Paging should generally be reserved for conditions requiring immediate human response.

Examples:

```text
Production Service Down
Critical Error Rate
Critical Database Failure
No Healthy Application Targets
Severe SLO Burn
```

Do not page engineers for:

```text
Normal CPU Variation
Minor Disk Growth
Expected Deployment Events
Low-Severity Development Issues
```

---

# 70. Warning vs Paging

A useful model:

```text
Warning
 ↓
Slack / Email
 ↓
Business Hours Investigation
```

Critical:

```text
Critical
 ↓
PagerDuty / Phone
 ↓
Immediate On-Call Response
```

The exact policy depends on the organization.

---

# 71. Alert Design Checklist

Before creating an alert, ask:

```text
1. Is this condition actionable?
2. Does it represent user impact?
3. Is the threshold meaningful?
4. Is the duration appropriate?
5. Is the severity correct?
6. Who owns the alert?
7. Where should it be routed?
8. Can related alerts be grouped?
9. Can duplicate alerts be deduplicated?
10. Should dependent alerts be inhibited?
11. Is there a runbook?
12. Is there a dashboard?
13. Does the alert contain enough context?
14. Has the alert been tested?
15. Is there a clear recovery condition?
```

---

# 72. Example Production Alert

```yaml
groups:
  - name: production-application-alerts

    rules:

      - alert: HighPaymentErrorRate

        expr: |
          (
            sum(rate(http_requests_total{
              service="payment",
              status=~"5.."
            }[5m]))
            /
            sum(rate(http_requests_total{
              service="payment"
            }[5m]))
          ) * 100 > 5

        for: 5m

        labels:
          severity: critical
          team: platform
          service: payment
          environment: production

        annotations:
          summary: "High payment API error rate"

          description: |
            Payment API 5xx error rate is above 5%
            for more than 5 minutes.

          runbook_url: "Payment API Incident Runbook"

          dashboard_url: "Payment API Grafana Dashboard"
```

---

# 73. What Makes This Alert Good?

It contains:

```text
Clear Name
      ↓
Meaningful Expression
      ↓
Threshold
      ↓
Duration
      ↓
Severity
      ↓
Team
      ↓
Service
      ↓
Environment
      ↓
Description
      ↓
Runbook
      ↓
Dashboard
```

The engineer receives both:

```text
Signal
+
Context
```

---

# 74. Poor Alert Example

```yaml
- alert: CPUAlert
  expr: cpu > 80
```

Problems:

```text
No Duration
No Severity
No Service
No Environment
No Team
No Description
No Runbook
No Dashboard
No Clear Impact
```

This alert may technically work but is operationally weak.

---

# 75. Improved Alert Example

```yaml
- alert: HighPaymentCPU
  expr: payment_cpu_usage > 90
  for: 10m

  labels:
    severity: warning
    service: payment
    environment: production
    team: platform

  annotations:
    summary: "High CPU usage on payment service"
    description: "Payment service CPU has remained above 90% for 10 minutes."
    runbook_url: "Payment CPU Troubleshooting Runbook"
    dashboard_url: "Payment Service Dashboard"
```

This is much easier to operate.

---

# 76. Alert Design Mental Model

```text
                    METRIC
                       │
                       ↓
                 IS IT IMPORTANT?
                       │
              ┌────────┴────────┐
              ↓                 ↓
             NO                YES
              │                 │
          No Alert              ↓
                         IS IT ACTIONABLE?
                               │
                       ┌───────┴───────┐
                       ↓               ↓
                      NO              YES
                       │               │
                  No Alert             ↓
                               DEFINE THRESHOLD
                                       │
                                       ↓
                                DEFINE DURATION
                                       │
                                       ↓
                                  DEFINE SEVERITY
                                       │
                                       ↓
                                    OWNER
                                       │
                                       ↓
                                    ROUTING
                                       │
                                       ↓
                                   CONTEXT
                                       │
                              ┌────────┼────────┐
                              ↓        ↓        ↓
                           Runbook  Dashboard  Impact
                              │        │        │
                              └────────┼────────┘
                                       ↓
                                    TEST
                                       │
                                       ↓
                                   PRODUCTION
```

---

# 77. Final Key Takeaway

Good alert design is about turning observability data into reliable operational signals.

The complete process is:

```text
Metric
  ↓
Meaningful Condition
  ↓
Threshold
  ↓
Duration
  ↓
Severity
  ↓
Ownership
  ↓
Routing
  ↓
Context
  ↓
Notification
  ↓
Investigation
  ↓
Recovery
```

The most important principle is:

```text
Do not alert because something is interesting.

Alert because someone needs to take action.
```

A strong production alert should tell the engineer:

```text
WHAT happened
WHERE it happened
HOW severe it is
WHO owns it
WHAT impact exists
WHERE to investigate
WHAT to do next
```

The ultimate goal is:

```text
Less Noise
+
Better Signals
+
Faster Detection
+
Faster Response
+
Faster Recovery
=
Reliable Production Operations
```