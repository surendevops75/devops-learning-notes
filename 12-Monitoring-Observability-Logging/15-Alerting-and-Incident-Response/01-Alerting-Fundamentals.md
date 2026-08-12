# Alerting Fundamentals

## 1. Overview

Alerting is the process of automatically detecting abnormal or dangerous conditions in infrastructure, applications, databases, Kubernetes, networks, and other production systems and notifying the responsible engineers.

Monitoring answers:

```text
What is happening?
```

Alerting answers:

```text
What requires human attention?
```

Incident response answers:

```text
What should we do about it?
```

A production observability flow is:

```text
Infrastructure / Application
            ↓
          Metrics
            ↓
        Monitoring
            ↓
       Alert Rules
            ↓
          Alert
            ↓
       Alertmanager
            ↓
     Notification Channel
            ↓
        On-Call Engineer
            ↓
      Incident Response
            ↓
       Root Cause
            ↓
         Recovery
```

The goal of alerting is not to generate as many alerts as possible.

The goal is:

```text
Detect important problems
+
Notify the right people
+
At the right time
+
With enough context
+
Without unnecessary noise
```

---

# 2. Monitoring vs Alerting

Monitoring and alerting are related but different.

### Monitoring

Monitoring continuously collects and displays system information.

Example:

```text
CPU = 85%
Memory = 70%
Disk = 60%
```

### Alerting

Alerting determines whether a condition requires action.

Example:

```text
CPU > 90% for 10 minutes
```

Then:

```text
CPU Alert
    ↓
Notification
    ↓
Engineer Investigation
```

Therefore:

```text
Monitoring = Visibility

Alerting = Actionable Notification
```

---

# 3. Why Alerting Is Important

Production systems are too large to monitor manually.

Imagine:

```text
100 EC2 Instances
50 Kubernetes Nodes
200 Pods
20 Databases
30 Load Balancers
100+ Microservices
```

An engineer cannot continuously watch every metric.

Alerting automates detection.

Example:

```text
Application
    ↓
Error Rate = 12%
    ↓
Alert Rule
    ↓
Alert Triggered
    ↓
Engineer Notified
```

This reduces detection time.

---

# 4. Alerting Objectives

A good alerting system should help achieve:

```text
Fast Detection
Fast Notification
Fast Diagnosis
Fast Recovery
Reduced Downtime
Reduced Alert Noise
Clear Ownership
```

Important incident metrics include:

```text
MTTD
MTTR
```

Where:

```text
MTTD = Mean Time To Detect

MTTR = Mean Time To Recovery / Resolve
```

---

# 5. Alerting Architecture

A Prometheus-based alerting architecture commonly looks like:

```text
                   TARGETS
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
       EC2           EKS           DB
        │             │             │
        └─────────────┼─────────────┘
                      ↓
                  Prometheus
                      │
                 Alert Rules
                      │
                      ↓
                 Alertmanager
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
        Email       Slack       PagerDuty
```

Prometheus evaluates alerting rules.

Alertmanager handles:

```text
Grouping
Routing
Deduplication
Silencing
Inhibition
Notifications
```

---

# 6. Prometheus Alerting Flow

The basic flow is:

```text
Exporter / Application
        ↓
    Prometheus
        ↓
   Query Metric
        ↓
   Evaluate Rule
        ↓
Condition True
        ↓
      Alert
        ↓
  Alertmanager
        ↓
 Notification
```

Example:

```text
CPU Usage
   ↓
92%
   ↓
CPU > 90%
   ↓
Alert
```

---

# 7. What Should Trigger an Alert?

An alert should generally represent a condition that requires action.

Good examples:

```text
Application is unavailable
Error rate is critically high
Database is unreachable
Disk is almost full
Node is unavailable
Critical service has no healthy targets
Replication is severely delayed
Certificate is about to expire
```

Bad examples:

```text
CPU briefly reached 70%
One request was slow
One pod restarted
Memory changed slightly
```

Not every unusual metric requires an alert.

---

# 8. Alert vs Metric

A metric is an observation.

Example:

```text
CPU Usage = 91%
```

An alert is a decision based on that metric.

Example:

```text
CPU Usage > 90%
for 10 minutes
```

Therefore:

```text
Metric
   ↓
Condition
   ↓
Duration
   ↓
Alert
```

---

# 9. Alert States

Prometheus alert rules commonly have states such as:

```text
Inactive
Pending
Firing
```

### Inactive

The alert condition is false.

```text
CPU = 40%
Threshold = 90%

Alert = Inactive
```

### Pending

The condition is true but has not remained true for the configured duration.

```text
CPU = 92%
Threshold = 90%
Duration = 10 minutes

Current Duration = 2 minutes

Alert = Pending
```

### Firing

The condition has remained true long enough.

```text
CPU = 92%
Threshold = 90%
Duration = 10 minutes

Alert = Firing
```

---

# 10. Why Use a Duration?

Without a duration:

```text
CPU > 90%
```

could trigger whenever CPU briefly spikes.

Example:

```text
CPU
 │
95%      ▲
 │       │
90% ─────┼──────── Threshold
 │       │
 │_______│____________
        30 sec
```

A short spike may not require human intervention.

Better:

```text
CPU > 90%
for 10 minutes
```

This reduces alert noise.

---

# 11. Example Alert Rule

A basic Prometheus alert rule:

```yaml
groups:
  - name: infrastructure-alerts

    rules:
      - alert: HighCPUUsage
        expr: 100 * (1 - avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m]))) > 90
        for: 10m

        labels:
          severity: warning

        annotations:
          summary: "High CPU usage"
          description: "CPU usage is above 90% for more than 10 minutes on {{ $labels.instance }}"
```

The rule contains:

```text
alert
expr
for
labels
annotations
```

---

# 12. Alert Name

The alert name should clearly describe the condition.

Good:

```text
HighCPUUsage
DatabaseConnectionPoolExhaustion
HighErrorRate
DiskSpaceLow
KubernetesNodeNotReady
```

Avoid vague names:

```text
Problem1
ServerIssue
SomethingWrong
HighMetric
```

A good alert name helps engineers quickly understand the problem.

---

# 13. Alert Expression

The expression defines the condition.

Example:

```yaml
expr: cpu_usage > 90
```

Conceptually:

```text
Metric
  ↓
Comparison
  ↓
Threshold
  ↓
Alert
```

More realistic alerts often use PromQL functions.

Example:

```promql
rate(http_requests_total{status=~"5.."}[5m])
```

This calculates the rate of HTTP 5xx requests.

---

# 14. Alert Duration

The `for` field defines how long the condition must remain true.

Example:

```yaml
for: 10m
```

Meaning:

```text
Condition becomes true
        ↓
Wait 10 minutes
        ↓
Still true?
        ↓
Alert fires
```

This prevents transient spikes from immediately triggering alerts.

---

# 15. Alert Labels

Labels provide metadata.

Example:

```yaml
labels:
  severity: critical
  team: platform
  service: payment
  environment: production
```

Labels can be used by Alertmanager for routing.

For example:

```text
severity = critical
        ↓
PagerDuty

severity = warning
        ↓
Slack
```

---

# 16. Alert Annotations

Annotations provide human-readable information.

Example:

```yaml
annotations:
  summary: "High CPU usage"
  description: "CPU usage is above 90% on {{ $labels.instance }}"
```

Common annotations include:

```text
summary
description
runbook_url
dashboard_url
```

Annotations should help the engineer understand what happened.

---

# 17. Runbook Links

A production alert should ideally link to a runbook.

Example:

```yaml
annotations:
  summary: "Database connection pool exhausted"
  description: "Database connection usage is above 90%"
  runbook_url: "https://internal.example/runbooks/database-connections"
```

The engineer receives:

```text
Alert
 ↓
Runbook
 ↓
Troubleshooting Steps
 ↓
Resolution
```

This reduces Mean Time To Recovery.

---

# 18. Dashboard Links

Alerts can also include a dashboard reference.

Example:

```yaml
annotations:
  dashboard_url: "Grafana Dashboard"
```

The engineer can quickly move from:

```text
Alert
 ↓
Dashboard
 ↓
Metrics
 ↓
Investigation
```

This is especially useful during production incidents.

---

# 19. Severity Levels

Organizations commonly classify alerts using severity.

Example:

```text
INFO
WARNING
CRITICAL
```

### INFO

Informational event.

Usually does not require immediate human action.

### WARNING

Potential problem.

Requires investigation but may not be an immediate outage.

### CRITICAL

Active production impact or high risk of immediate impact.

Requires urgent response.

---

# 20. Severity Example

Example:

```text
Disk Usage = 75%
```

Could be:

```text
WARNING
```

At:

```text
Disk Usage = 90%
```

could become:

```text
CRITICAL
```

However, thresholds should be based on:

```text
Workload
Growth Rate
Recovery Time
Business Impact
```

rather than arbitrary numbers.

---

# 21. Infrastructure Alerts

Common infrastructure alerts include:

```text
High CPU
High Memory
Disk Space Low
Disk I/O High
Network Errors
Network Packet Drops
Instance Down
Node Not Ready
High Load
Filesystem Read-Only
```

Example:

```text
EC2
 ↓
CPU > 90% for 10m
 ↓
HighCPUUsage
```

---

# 22. Kubernetes Alerts

Common Kubernetes alerts include:

```text
Node Not Ready
Pod CrashLoopBackOff
Pod Pending
High Pod Restart Rate
Container OOMKilled
Deployment Replicas Unavailable
High API Server Latency
Disk Pressure
Memory Pressure
CPU Saturation
```

Example:

```text
Node
 ↓
NotReady
 ↓
Alert
 ↓
On-Call Engineer
```

---

# 23. Application Alerts

Application alerts should focus on user impact.

Examples:

```text
High HTTP 5xx Rate
High Request Latency
Service Unavailable
High Error Rate
Request Throughput Drop
Queue Backlog
Failed Transactions
```

Example:

```text
HTTP 5xx Rate
     ↓
8%
     ↓
Threshold = 5%
     ↓
Alert
```

---

# 24. Database Alerts

Common database alerts include:

```text
Database Down
High CPU
High Connection Usage
Connection Failures
Slow Queries
High Lock Waits
Deadlocks
Low Storage
High Disk Latency
Replication Lag
Backup Failure
```

Example:

```text
DB Connections
      ↓
95% of Maximum
      ↓
Critical Alert
```

---

# 25. Network Alerts

Common network alerts include:

```text
High Packet Loss
High Network Latency
High Network Errors
High Network Drops
High Bandwidth Utilization
DNS Failure
Load Balancer Target Failure
Connection Failures
```

Example:

```text
Packet Loss > 5%
for 5 minutes
        ↓
Network Alert
```

---

# 26. Availability Alerts

Availability alerts should detect actual service impact.

Example:

```text
Application Health Check
        ↓
Failed
        ↓
Service Unavailable
        ↓
Critical Alert
```

A better availability alert is often:

```text
No healthy targets
```

rather than:

```text
CPU > 90%
```

because no healthy targets directly represents user impact.

---

# 27. Symptom-Based vs Cause-Based Alerts

There are two useful categories.

### Symptom Alert

Detects user-visible impact.

Example:

```text
HTTP 5xx > 5%
```

### Cause Alert

Detects a likely underlying cause.

Example:

```text
Database connections > 90%
```

A mature alerting system uses both.

```text
User Impact
+
Likely Cause
```

---

# 28. Golden Signals

A common approach for application alerting is the Four Golden Signals:

```text
Latency
Traffic
Errors
Saturation
```

### Latency

How long requests take.

### Traffic

How much demand the system receives.

### Errors

How many requests fail.

### Saturation

How close the system is to resource limits.

These provide a strong foundation for application alerting.

---

# 29. Latency Alert

Example:

```text
95th Percentile Latency > 1 second
for 10 minutes
```

Why percentile?

Because averages can hide slow requests.

Example:

```text
99 requests = 100 ms
1 request = 10 seconds
```

The average may not fully represent the user experience.

Percentiles such as:

```text
p50
p95
p99
```

can provide better visibility.

---

# 30. Error Rate Alert

Example:

```text
HTTP 5xx Error Rate > 5%
for 5 minutes
```

Conceptually:

```text
Total Requests
       ↓
Failed Requests
       ↓
Error Rate
       ↓
Threshold
       ↓
Alert
```

Error rate is often more useful than alerting on individual errors.

---

# 31. Traffic Alerts

Traffic alerts can identify unusual demand.

Example:

```text
Request Rate
     ↓
Expected = 1,000 req/s
Current = 5,000 req/s
```

Possible causes:

```text
Traffic Spike
Marketing Event
Bot Traffic
Attack
Application Retry Storm
```

Traffic anomalies may require investigation even when the application is still healthy.

---

# 32. Saturation Alerts

Saturation indicates that a resource is approaching its limit.

Examples:

```text
CPU > 90%
Memory > 90%
Disk > 85%
Connections > 90%
Queue Depth > Threshold
```

Saturation alerts help detect capacity problems before complete failure.

---

# 33. Alert Fatigue

Alert fatigue occurs when engineers receive too many alerts.

Example:

```text
100 Alerts / Day
       ↓
Most Are Not Actionable
       ↓
Engineers Ignore Alerts
       ↓
Critical Alert Appears
       ↓
Missed Response
```

This is dangerous.

The solution is:

```text
Fewer
+
Meaningful
+
Actionable
Alerts
```

---

# 34. Alert Noise

Alert noise can come from:

```text
Low thresholds
Short evaluation windows
Duplicate alerts
Non-actionable conditions
Incorrect severity
Poor grouping
Transient failures
```

Example:

```text
CPU briefly reaches 91%
       ↓
Alert
       ↓
CPU returns to 60%
       ↓
Alert resolves
       ↓
Repeated every few minutes
```

This creates unnecessary noise.

---

# 35. Reducing Alert Noise

Use:

```text
for duration
Appropriate thresholds
Grouping
Deduplication
Inhibition
Severity
Recording rules
Alert review
```

Example:

```text
CPU > 90%
for 10 minutes
```

is usually better than:

```text
CPU > 90%
instantaneously
```

when brief CPU spikes are normal.

---

# 36. Actionable Alerts

A good alert should answer:

```text
What is wrong?
Where is it happening?
How serious is it?
When did it start?
Who owns it?
What should I do?
```

Example:

```text
CRITICAL

Service: payment
Environment: production
Instance: node-01

Problem:
HTTP 5xx rate > 10% for 5 minutes.

Impact:
Payment requests are failing.

Runbook:
Database / Payment Service Troubleshooting
```

---

# 37. Non-Actionable Alerts

Avoid alerts like:

```text
CPU = 80%
```

without context.

Better:

```text
CPU > 90% for 15 minutes
AND
service is experiencing elevated latency
```

The second alert has more operational meaning.

---

# 38. Alert Ownership

Every important alert should have an owner.

Example:

```text
Payment Service
      ↓
Platform Team
```

or:

```text
Database Alert
      ↓
Database Team
```

Without ownership:

```text
Alert
 ↓
Nobody Responds
```

This creates operational risk.

---

# 39. Alert Routing Concept

Alert routing determines who receives the alert.

Example:

```text
Alert
  │
  ├── severity=critical
  │        ↓
  │      PagerDuty
  │
  ├── severity=warning
  │        ↓
  │       Slack
  │
  └── severity=info
           ↓
          Email
```

Alertmanager commonly handles this routing.

---

# 40. Alert Grouping Concept

Suppose one node failure causes:

```text
Pod Alert
Node Alert
Service Alert
Deployment Alert
```

Without grouping:

```text
4 Alerts
↓
4 Notifications
```

With grouping:

```text
4 Alerts
   ↓
Grouped Incident
   ↓
1 Notification
```

Grouping reduces noise.

---

# 41. Alert Deduplication

Sometimes the same problem is detected multiple times.

Example:

```text
Prometheus-1
   ↓
High CPU

Prometheus-2
   ↓
High CPU
```

Without deduplication:

```text
2 Notifications
```

With deduplication:

```text
1 Notification
```

This prevents repeated alerts for the same event.

---

# 42. Alert Inhibition

Inhibition prevents lower-level alerts when a higher-level alert already explains the problem.

Example:

```text
Node Down
   ↓
Pods Unhealthy
   ↓
Application Unavailable
```

Without inhibition:

```text
Node Alert
Pod Alert
Application Alert
```

With inhibition:

```text
Node Down
   ↓
Primary Alert
```

Related alerts can be suppressed temporarily.

---

# 43. Alert Silencing

Silencing temporarily prevents notifications for known conditions.

Example:

```text
Planned Maintenance
        ↓
Known Alerts
        ↓
Silence
```

This prevents expected maintenance activity from generating unnecessary notifications.

Silencing should be:

```text
Temporary
Scoped
Documented
```

Do not create permanent silences for recurring production problems.

---

# 44. Maintenance Window

During planned maintenance:

```text
Deployment
     ↓
Expected Restart
     ↓
Expected Alerts
```

A maintenance window or silence can prevent unnecessary notifications.

Example:

```text
Maintenance:
01:00 - 02:00

Alerts:
Silenced for affected services
```

After maintenance, verify that alerting is active again.

---

# 45. Alert Lifecycle

A typical lifecycle is:

```text
Metric
  ↓
Condition
  ↓
Pending
  ↓
Firing
  ↓
Notification
  ↓
Acknowledgement
  ↓
Investigation
  ↓
Mitigation
  ↓
Recovery
  ↓
Resolved
```

The lifecycle should be documented and understood by the on-call team.

---

# 46. Acknowledgement

Acknowledgement means that an engineer has accepted responsibility for investigating the incident.

Example:

```text
Alert Fired
    ↓
Engineer Acknowledges
    ↓
Investigation Starts
```

Acknowledgement does not mean the problem is fixed.

It means:

```text
Someone is actively handling it.
```

---

# 47. Incident Detection

Incident detection can come from:

```text
Monitoring Alert
Customer Report
Support Ticket
Synthetic Monitoring
Log Detection
Security Detection
Manual Observation
```

The fastest detection usually comes from automated monitoring.

---

# 48. MTTD

Mean Time To Detect measures how long it takes to detect an incident.

Example:

```text
Incident Starts
     ↓
10:00
     ↓
Alert Fires
     ↓
10:05
```

Then:

```text
MTTD = 5 minutes
```

Lower MTTD generally means faster incident detection.

---

# 49. MTTR

Mean Time To Recovery measures how long it takes to restore service.

Example:

```text
Incident Starts
10:00

Detected
10:05

Recovered
10:30
```

Then:

```text
MTTR ≈ 30 minutes
```

Improving:

```text
Detection
Diagnosis
Mitigation
Recovery
```

can reduce MTTR.

---

# 50. Alerting and Incident Response

Alerting is only the beginning.

```text
Alert
 ↓
Detection
 ↓
Notification
 ↓
On-Call
 ↓
Investigation
 ↓
Mitigation
 ↓
Recovery
 ↓
Root Cause Analysis
 ↓
Preventive Action
```

A mature DevOps organization treats alerting and incident response as one operational process.

---

# 51. Alert Quality

A useful alert should have:

```text
Clear Name
Correct Severity
Useful Labels
Useful Description
Relevant Dashboard
Runbook
Owner
Expected Action
```

Example:

```yaml
annotations:
  summary: "High HTTP error rate"
  description: "Payment API 5xx rate is above 5% for 5 minutes."
  runbook_url: "..."
  dashboard_url: "..."
```

---

# 52. Alert Naming Convention

A consistent naming convention improves operations.

Example:

```text
HighCPUUsage
HighMemoryUsage
DiskSpaceLow
NodeNotReady
PodCrashLooping
HighHTTPErrorRate
HighRequestLatency
DatabaseConnectionPoolExhausted
DatabaseReplicationLag
```

Avoid ambiguous names.

---

# 53. Environment Labels

Alerts should identify the environment.

Example:

```yaml
labels:
  environment: production
  service: payment
  severity: critical
```

This helps distinguish:

```text
Production
Staging
Development
```

For example:

```text
HighErrorRate
environment=production
```

is much more urgent than:

```text
HighErrorRate
environment=development
```

---

# 54. Service Labels

Include the affected service.

Example:

```yaml
labels:
  service: payment
  team: platform
```

This enables routing:

```text
Payment
  ↓
Payment Team

Database
  ↓
Database Team
```

---

# 55. Alert Rule Example: High Error Rate

```yaml
groups:
  - name: application-alerts

    rules:
      - alert: HighHTTPErrorRate
        expr: |
          (
            sum(rate(http_requests_total{status=~"5.."}[5m]))
            /
            sum(rate(http_requests_total[5m]))
          ) * 100 > 5

        for: 5m

        labels:
          severity: critical
          service: application
          environment: production

        annotations:
          summary: "High HTTP 5xx error rate"
          description: "HTTP 5xx error rate is above 5% for more than 5 minutes."
```

The important design elements are:

```text
Meaningful Metric
Meaningful Threshold
Duration
Severity
Service
Environment
Description
```

---

# 56. Alert Rule Example: Disk Usage

```yaml
groups:
  - name: infrastructure-alerts

    rules:
      - alert: DiskSpaceLow
        expr: |
          (
            node_filesystem_avail_bytes{fstype!=""}
            /
            node_filesystem_size_bytes{fstype!=""}
          ) * 100 < 15

        for: 10m

        labels:
          severity: warning

        annotations:
          summary: "Low disk space"
          description: "Filesystem available space is below 15% on {{ $labels.instance }}"
```

The actual threshold should be adapted to workload requirements.

---

# 57. Alert Rule Example: Node Down

A common infrastructure concept is:

```text
Target Unreachable
```

Example:

```yaml
groups:
  - name: infrastructure-alerts

    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 5m

        labels:
          severity: critical

        annotations:
          summary: "Instance is down"
          description: "{{ $labels.instance }} has been unreachable for more than 5 minutes."
```

This is generally more actionable than alerting only on high CPU.

---

# 58. Alert Testing

Alert rules should be tested before production.

Validate:

```text
Rule Syntax
Expression
Labels
Annotations
Threshold
Duration
Routing
Notification
```

A useful flow is:

```text
Write Rule
   ↓
Validate
   ↓
Trigger Test Condition
   ↓
Alert Fires
   ↓
Notification Received
   ↓
Resolve Condition
   ↓
Alert Resolves
```

---

# 59. Alert Validation

Before deploying alert rules, validate the Prometheus configuration.

Example:

```bash
promtool check rules alert-rules.yml
```

Also validate the Prometheus configuration:

```bash
promtool check config prometheus.yml
```

Then test the actual notification path.

---

# 60. Alert Testing in Production

Never intentionally create a dangerous production failure just to test alerting.

Prefer controlled testing:

```text
Non-production Environment
        ↓
Simulated Metric
        ↓
Alert
        ↓
Notification
        ↓
Verification
```

For production, use carefully controlled and approved testing procedures.

---

# 61. Alert Review

Alert rules should be reviewed periodically.

Ask:

```text
Did this alert require action?
Was it noisy?
Was the severity correct?
Did the right team receive it?
Was the description useful?
Was the runbook useful?
Was the threshold appropriate?
```

If an alert repeatedly creates unnecessary pages:

```text
Investigate
   ↓
Tune
   ↓
Reduce Noise
```

Do not simply ignore it.

---

# 62. Alert Debt

Alert debt is the accumulation of:

```text
Unused Alerts
Noisy Alerts
Incorrect Alerts
Missing Owners
Outdated Runbooks
Wrong Thresholds
```

Alert debt should be treated like technical debt.

Regularly:

```text
Review
Remove
Tune
Document
Assign Ownership
```

---

# 63. Alerting Anti-Patterns

Avoid:

```text
Alert on every metric
Alert on every log
No severity
No owner
No runbook
No duration
No grouping
No deduplication
Permanent silences
Alerts nobody responds to
Alerts without user impact
```

These patterns create alert fatigue.

---

# 64. Alerting Best Practices

```text
1. Alert on actionable conditions.
2. Focus on user impact.
3. Use appropriate thresholds.
4. Use evaluation durations.
5. Assign severity.
6. Add service and environment labels.
7. Include useful descriptions.
8. Add dashboard links.
9. Add runbook links.
10. Route alerts to the correct team.
11. Group related alerts.
12. Deduplicate duplicate alerts.
13. Use inhibition carefully.
14. Use temporary silences.
15. Review noisy alerts.
16. Test notification paths.
17. Measure MTTD.
18. Measure MTTR.
19. Keep runbooks updated.
20. Remove obsolete alerts.
```

---

# 65. Production Alerting Architecture

A complete production architecture can look like:

```text
                          PRODUCTION
                              │
          ┌───────────────────┼───────────────────┐
          ↓                   ↓                   ↓
       EC2 / EKS          Applications        Databases
          │                   │                   │
          └───────────────────┼───────────────────┘
                              ↓
                         Prometheus
                              │
                       Alerting Rules
                              │
                              ↓
                         Alertmanager
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
             Slack          Email          PagerDuty
              │               │               │
              └───────────────┼───────────────┘
                              ↓
                         On-Call Engineer
                              │
                              ↓
                       Incident Response
                              │
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                Dashboard   Logs      Traces
                    │         │         │
                 Grafana      ELK    OpenTelemetry
                                      │
                                    Jaeger
                              │
                              ↓
                           Recovery
                              │
                              ↓
                        Root Cause Analysis
```

---

# 66. Alerting Flow During an Incident

Example:

```text
Application
    ↓
HTTP 5xx ↑
    ↓
Prometheus
    ↓
Alert Rule
    ↓
Alert Firing
    ↓
Alertmanager
    ↓
PagerDuty / Slack
    ↓
On-Call Engineer
    ↓
Open Grafana
    ↓
Check Logs
    ↓
Check Traces
    ↓
Identify Root Cause
    ↓
Mitigate
    ↓
Service Recovered
    ↓
Alert Resolved
```

---

# 67. Production Alert Example

Scenario:

```text
Payment API
5xx Error Rate = 12%
```

Rule:

```text
5xx Error Rate > 5%
for 5 minutes
```

Flow:

```text
12%
 ↓
Condition True
 ↓
Pending
 ↓
5 Minutes
 ↓
Firing
 ↓
Critical Alert
 ↓
Payment Team
```

The engineer investigates:

```text
Application Logs
Database
Dependencies
Recent Deployment
Network
```

---

# 68. Alerting and Recent Deployments

Many incidents occur shortly after deployments.

Therefore correlate alerts with deployment events.

Example:

```text
Deployment
    ↓
5 Minutes Later
    ↓
HTTP 5xx ↑
    ↓
Alert
```

Investigation should include:

```text
Recent Code Changes
Configuration Changes
Image Version
Database Changes
Infrastructure Changes
```

This can significantly reduce investigation time.

---

# 69. Alerting and Kubernetes

A Kubernetes production alerting flow:

```text
Kubernetes
   │
   ├── Nodes
   ├── Pods
   ├── Services
   ├── Deployments
   └── Applications
        │
        ↓
    Prometheus
        ↓
   Alert Rules
        ↓
   Alertmanager
        ↓
 Notifications
```

Example alerts:

```text
NodeNotReady
PodCrashLooping
DeploymentReplicasUnavailable
HighPodRestartRate
HighCPUUsage
HighMemoryUsage
```

---

# 70. Alerting and Databases

Database alerts should focus on conditions that can affect application availability.

Example:

```text
Database
   │
   ├── Connections
   ├── Query Latency
   ├── CPU
   ├── Storage
   ├── Locks
   └── Replication
        │
        ↓
      Alerts
```

For example:

```text
Replication Lag > Critical Threshold
```

should be routed to the responsible database/platform team.

---

# 71. Alerting and Network

Network alerts should detect meaningful connectivity problems.

Example:

```text
Application
      ↓
Network
      ↓
Packet Loss ↑
      ↓
Latency ↑
      ↓
Application Errors ↑
```

A mature alerting system should correlate the network condition with application impact.

---

# 72. Alerting and SLOs

Alerting can be aligned with Service Level Objectives.

Example:

```text
SLO:
99.9% Successful Requests
```

If the error budget is being consumed rapidly:

```text
Error Rate ↑
     ↓
Error Budget Burn ↑
     ↓
Alert
```

SLO-based alerting can be more meaningful than alerting on individual infrastructure metrics.

---

# 73. Error Budget Concept

Suppose:

```text
SLO = 99.9%
```

The allowed failure budget is:

```text
0.1%
```

If the service begins consuming the budget rapidly:

```text
Errors ↑
   ↓
Error Budget Burn ↑
   ↓
Operational Risk ↑
```

This can trigger an alert before the overall SLO is permanently violated.

---

# 74. Alert Escalation

If the first responder does not acknowledge a critical incident:

```text
Alert
 ↓
Primary On-Call
 ↓
No Response
 ↓
Escalation
 ↓
Secondary On-Call
 ↓
Team Lead
```

Escalation policies should be clearly defined.

---

# 75. Alert Notification Channels

Common notification channels include:

```text
Slack
Email
PagerDuty
Microsoft Teams
SMS
Phone
```

Critical alerts should use a channel that reliably wakes or reaches the on-call engineer.

Non-critical alerts can use less intrusive channels.

---

# 76. Alert Routing Strategy

Example:

```text
Severity
   │
   ├── Critical → PagerDuty
   │
   ├── Warning  → Slack
   │
   └── Info     → Email
```

Then:

```text
Service
   │
   ├── Payment → Payment Team
   ├── Database → Database Team
   └── Platform → Platform Team
```

Alertmanager can combine these routing dimensions.

---

# 77. Alerting Maturity Levels

### Level 1 — Basic

```text
Simple Thresholds
Email Notifications
```

### Level 2 — Structured

```text
Severity
Labels
Routing
Dashboards
```

### Level 3 — Advanced

```text
Grouping
Deduplication
Inhibition
Runbooks
SLO Alerts
Escalation
```

### Level 4 — Mature

```text
SLO-Based Alerting
Error Budget
Automated Remediation
Incident Analytics
Continuous Alert Review
```

---

# 78. Alerting Checklist

```text
ALERT RULE
├── Clear Name
├── Correct Expression
├── Threshold
├── Duration
├── Severity
├── Service
├── Environment
├── Description
├── Dashboard
└── Runbook

ALERTMANAGER
├── Routing
├── Grouping
├── Deduplication
├── Inhibition
├── Silencing
└── Notifications

OPERATIONS
├── On-Call
├── Escalation
├── Acknowledgement
├── Incident Response
├── MTTD
└── MTTR
```

---

# 79. Interview Question

## What is alerting?

**Answer:**

Alerting is the process of detecting conditions that require human attention and sending notifications to the appropriate team.

For example:

```text
HTTP 5xx Rate > 5%
for 5 minutes
```

can trigger an alert.

The alert can then be routed through Alertmanager to the appropriate notification channel.

---

# 80. Interview Question

## What is the difference between monitoring and alerting?

**Answer:**

Monitoring provides visibility into system behavior.

For example:

```text
CPU = 90%
```

Alerting determines whether that condition requires action.

For example:

```text
CPU > 90%
for 10 minutes
```

So:

```text
Monitoring = Visibility

Alerting = Actionable Notification
```

---

# 81. Interview Question

## What makes a good alert?

**Answer:**

A good alert should be:

```text
Actionable
Specific
Relevant
Reliable
Clearly owned
```

It should contain:

```text
What happened?
Where?
Severity?
Impact?
When?
Dashboard?
Runbook?
```

It should help the engineer begin troubleshooting immediately.

---

# 82. Interview Question

## What is alert fatigue?

**Answer:**

Alert fatigue occurs when engineers receive too many alerts, especially alerts that are not actionable.

This can cause engineers to ignore alerts, including critical ones.

I reduce alert fatigue using:

```text
Appropriate Thresholds
for Durations
Grouping
Deduplication
Inhibition
Correct Severity
Alert Review
```

---

# 83. Interview Question

## Why should we use the `for` duration in Prometheus alerts?

**Answer:**

The `for` duration prevents temporary metric spikes from immediately triggering alerts.

For example:

```text
CPU > 90%
for 10 minutes
```

means CPU must remain above the threshold for 10 minutes before the alert fires.

This reduces transient alerts and alert noise.

---

# 84. Interview Question

## What is the difference between Pending and Firing?

**Answer:**

When an alert condition becomes true, the alert can enter the Pending state.

If the condition remains true for the configured `for` duration, it transitions to Firing.

Example:

```text
Condition True
     ↓
Pending
     ↓
Duration Completed
     ↓
Firing
```

---

# 85. Interview Question

## What are the Four Golden Signals?

**Answer:**

The Four Golden Signals are:

```text
Latency
Traffic
Errors
Saturation
```

They provide a high-level view of application health and user experience.

I use them as a foundation for application monitoring and alerting.

---

# 86. Interview Question

## How would you reduce alert noise?

**Answer:**

I would first identify which alerts are noisy and determine why they are firing.

Then I would consider:

```text
Threshold Tuning
Evaluation Duration
Grouping
Deduplication
Inhibition
Severity
Alert Consolidation
Removing Non-Actionable Alerts
```

I would also review alerts regularly instead of allowing alert rules to accumulate indefinitely.

---

# 87. Interview Question

## What should a critical production alert contain?

**Answer:**

A critical alert should contain:

```text
Alert Name
Severity
Environment
Service
Affected Instance / Pod
Problem Description
Potential Impact
Dashboard Link
Runbook Link
```

For example:

```text
CRITICAL
Service: Payment
Environment: Production

HTTP 5xx rate is above 10% for 5 minutes.

Impact:
Payment requests may be failing.

Dashboard:
Payment Service Dashboard

Runbook:
Payment API Incident Runbook
```

---

# 88. Interview Question

## What is MTTD?

**Answer:**

MTTD means Mean Time To Detect.

It measures how long it takes to detect an incident after it starts.

Example:

```text
Incident Starts → 10:00
Alert Fires     → 10:05

MTTD = 5 minutes
```

Reducing MTTD helps teams detect production problems faster.

---

# 89. Interview Question

## What is MTTR?

**Answer:**

MTTR generally refers to Mean Time To Recovery or Mean Time To Resolve.

It measures how long it takes to restore service or resolve an incident.

Example:

```text
Incident Starts → 10:00
Recovery        → 10:30

MTTR ≈ 30 minutes
```

Runbooks, dashboards, automation, and good alert context can help reduce MTTR.

---

# 90. Final Mental Model

```text
                         PRODUCTION SYSTEM
                                │
        ┌───────────────────────┼───────────────────────┐
        ↓                       ↓                       ↓
   Infrastructure          Application              Database
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                ↓
                            Metrics
                                │
                                ↓
                           Prometheus
                                │
                                ↓
                          Alert Rules
                                │
                     ┌──────────┴──────────┐
                     ↓                     ↓
                  Inactive               Pending
                                           │
                                    Condition Persists
                                           │
                                           ↓
                                        Firing
                                           │
                                           ↓
                                     Alertmanager
                                           │
                         ┌─────────────────┼─────────────────┐
                         ↓                 ↓                 ↓
                       Slack            Email           PagerDuty
                         │                 │                 │
                         └─────────────────┼─────────────────┘
                                           ↓
                                    On-Call Engineer
                                           │
                                           ↓
                                    Incident Response
                                           │
                           ┌───────────────┼───────────────┐
                           ↓               ↓               ↓
                       Dashboard         Logs           Traces
                           │               │               │
                        Grafana            ELK        OpenTelemetry
                                                           │
                                                         Jaeger
                                           │
                                           ↓
                                      Investigation
                                           │
                                           ↓
                                        Mitigation
                                           │
                                           ↓
                                        Recovery
                                           │
                                           ↓
                                   Root Cause Analysis
                                           │
                                           ↓
                                   Preventive Actions
```

---

# 91. Key Takeaway

Alerting is not about creating alerts for every abnormal metric.

The objective is:

```text
Detect
 ↓
Notify
 ↓
Understand
 ↓
Act
 ↓
Recover
 ↓
Prevent
```

A production alert should be:

```text
Actionable
+
Relevant
+
Reliable
+
Owned
+
Context-Rich
```

The complete DevOps operational model is:

```text
Monitoring
    ↓
Alerting
    ↓
Incident Response
    ↓
Root Cause Analysis
    ↓
Remediation
    ↓
Prevention
```

The strongest alerting systems reduce both:

```text
Time To Detect
+
Time To Recover
```

while avoiding unnecessary alert noise.