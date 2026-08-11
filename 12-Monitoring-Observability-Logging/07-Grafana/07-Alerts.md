# Grafana Alerts

## 1. Overview

Alerts are one of the most important parts of an observability platform.

Dashboards help engineers answer:

```text
"What is happening?"
```

Alerts help answer:

```text
"What needs my attention now?"
```

A production alerting system should detect meaningful problems and route them to the right people without creating unnecessary noise.

A basic architecture is:

```text
Metric / Log / Query
        ↓
   Alert Rule
        ↓
   Evaluation
        ↓
   Alert State
        ↓
Notification Policy
        ↓
  Contact Point
        ↓
Slack / Email / Pager
```

---

# 2. Why Alerting Is Important

Without alerting:

```text
Problem
   ↓
Users complain
   ↓
Engineer checks dashboard
   ↓
Engineer discovers issue
```

With alerting:

```text
Problem
   ↓
Monitoring detects it
   ↓
Alert fires
   ↓
Engineer receives notification
   ↓
Engineer investigates
```

The objective is to detect problems before or while they significantly affect users.

---

# 3. Alerting Is Not Monitoring

Monitoring provides visibility:

```text
Metrics
Logs
Traces
Dashboards
```

Alerting adds action:

```text
Condition
   ↓
Notification
   ↓
Human / Automated Response
```

Example:

```text
CPU = 85%
```

Monitoring shows:

```text
CPU is 85%
```

Alerting may decide:

```text
CPU > 85% for 10 minutes
        ↓
Warning alert
```

---

# 4. Grafana Alerting Architecture

Modern Grafana alerting can be represented as:

```text
                    Data Source
                        │
                        ↓
                  Alert Query
                        │
                        ↓
                   Expression
                        │
                        ↓
                   Alert Rule
                        │
                        ↓
                  Alert Instance
                        │
                        ↓
              Notification Policy
                        │
                        ↓
                  Contact Point
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
        Slack         Email        PagerDuty
```

The exact capabilities depend on the Grafana version and edition.

---

# 5. Main Alerting Components

Understand these components clearly:

```text
1. Alert Rule
2. Query
3. Expression
4. Condition
5. Evaluation Interval
6. Pending Period
7. Alert State
8. Contact Point
9. Notification Policy
10. Notification Template
11. Mute Timing
```

---

# 6. Alert Rule

An alert rule defines:

```text
What should be monitored?
What condition should trigger?
How frequently should it be evaluated?
What should happen when it fires?
```

Example:

```text
Rule:
High Payment Error Rate

Condition:
Error rate > 5%

For:
10 minutes

Severity:
Critical
```

---

# 7. Alert Query

The query retrieves the data used by the alert.

For Prometheus:

```promql
sum(
  rate(http_requests_total{
    service="payment",
    status=~"5.."
  }[5m])
)
```

The query might then be transformed into an alert condition.

---

# 8. Alert Expression

An expression processes query results.

Example:

```text
Query A
   ↓
Error Rate
   ↓
Reduce
   ↓
Last value
   ↓
Threshold
   ↓
> 5%
```

This allows complex alert logic.

---

# 9. Alert Condition

A condition determines whether the alert should fire.

Example:

```text
IF
error_rate > 5%
```

Another example:

```text
IF
P95 latency > 1 second
```

---

# 10. Evaluation Interval

The evaluation interval determines how frequently the alert rule is evaluated.

Example:

```text
Evaluation interval:
1 minute
```

Flow:

```text
10:00 → Evaluate
10:01 → Evaluate
10:02 → Evaluate
10:03 → Evaluate
```

Avoid unnecessarily aggressive evaluation intervals for every rule.

---

# 11. Pending Period

A condition does not always need to immediately create an incident.

Example:

```text
Condition:
CPU > 85%

Pending:
10 minutes
```

Flow:

```text
CPU > 85%
    ↓
Condition true
    ↓
Pending
    ↓
Still true after 10 minutes
    ↓
Alert fires
```

This prevents transient spikes from generating alerts.

---

# 12. Immediate vs Sustained Alerts

Immediate alert:

```text
Condition true
      ↓
Fire immediately
```

Sustained alert:

```text
Condition true
      ↓
Wait
      ↓
Condition remains true
      ↓
Fire
```

Use sustained conditions for noisy metrics.

---

# 13. Alert States

An alert can have states such as:

```text
Normal
Pending
Firing
No Data
Error
```

The exact state behavior depends on the Grafana version and alert configuration.

Conceptually:

```text
Normal
  ↓
Condition true
  ↓
Pending
  ↓
Condition remains true
  ↓
Firing
```

---

# 14. Normal State

The alert condition is not met.

Example:

```text
Error rate = 0.5%
Threshold = 5%
```

Therefore:

```text
State = Normal
```

---

# 15. Pending State

The condition is true but the required duration has not elapsed.

Example:

```text
Error rate = 6%
Threshold = 5%
Pending = 10 minutes
```

At minute 3:

```text
State = Pending
```

The alert has not yet become a firing incident.

---

# 16. Firing State

The condition remains true for the configured duration.

Example:

```text
Error rate > 5%
for 10 minutes
```

Then:

```text
State = Firing
```

Notification routing can then occur.

---

# 17. No Data State

Sometimes the query returns no usable data.

Example:

```text
Prometheus
   ↓
No response / no series
```

The alerting system must have a defined policy for this situation.

Possible behavior:

```text
No Data
```

or another configured state behavior.

---

# 18. Error State

If the alert query cannot be evaluated because of an error:

```text
Prometheus unavailable
Query invalid
Datasource failure
```

the alert may enter an error-related state depending on configuration.

Do not treat every data-source failure as an application failure without understanding the alert rule.

---

# 19. Alert Labels

Labels identify and classify an alert.

Example:

```text
severity="critical"
service="payment"
environment="production"
team="payments"
```

These labels are extremely important for routing.

---

# 20. Alert Annotations

Annotations provide additional information.

Example:

```text
summary:
Payment service has high error rate

description:
5xx error rate has exceeded 5% for 10 minutes.

runbook_url:
Payment incident runbook
```

Labels answer:

```text
"How should this alert be classified?"
```

Annotations answer:

```text
"What information should the engineer see?"
```

---

# 21. Labels vs Annotations

Example:

```text
Labels:
severity=critical
service=payment
team=payments
environment=production
```

Annotations:

```text
summary=Payment API error rate is high
description=5xx rate exceeded 5%
runbook=Payment incident procedure
```

This distinction is important in production alert design.

---

# 22. Alert Severity

A common model:

```text
Info
Warning
Critical
```

Example:

```text
CPU > 70%
    ↓
Warning

CPU > 90%
    ↓
Critical
```

However, severity should be based on business and operational impact rather than arbitrary percentages.

---

# 23. Team Label

Use a team label:

```text
team="platform"
```

or:

```text
team="payments"
```

Then notification routing can send the alert to the correct team.

---

# 24. Environment Label

Include:

```text
environment="production"
```

This prevents confusion between:

```text
Development alert
```

and:

```text
Production alert
```

---

# 25. Service Label

Include:

```text
service="payment"
```

This makes alerts searchable and routable.

---

# 26. Example Alert Labels

```yaml
labels:
  severity: critical
  environment: production
  service: payment
  team: payments
```

These are examples; your organization should define a consistent label schema.

---

# 27. Contact Points

A contact point defines where notifications are sent.

Examples:

```text
Slack
Email
PagerDuty
Webhook
Microsoft Teams
Other supported integrations
```

Architecture:

```text
Alert
 ↓
Contact Point
 ↓
Notification Channel
```

---

# 28. Slack Contact Point

A common production setup:

```text
Grafana
   ↓
Alert
   ↓
Notification Policy
   ↓
Slack Contact Point
   ↓
#prod-alerts
```

Example message:

```text
🚨 Critical Alert

Service: payment
Environment: production
Alert: High Error Rate
Error Rate: 8.2%
```

The exact Slack integration depends on your Grafana configuration.

---

# 29. Email Contact Point

Architecture:

```text
Grafana
   ↓
Alert
   ↓
Notification Policy
   ↓
Email Contact Point
   ↓
On-call distribution list
```

Use email for alerts that do not necessarily require immediate paging.

---

# 30. PagerDuty Contact Point

For high-impact production incidents:

```text
Grafana
   ↓
Critical Alert
   ↓
Notification Policy
   ↓
PagerDuty
   ↓
On-call Engineer
```

This is more appropriate for urgent incidents than sending everything to email.

---

# 31. Contact Point Strategy

A practical model:

```text
Info
 ↓
Dashboard / low-priority channel

Warning
 ↓
Slack

Critical
 ↓
PagerDuty / on-call
```

The exact routing should follow your organization's incident-management process.

---

# 32. Notification Policies

A notification policy decides where alerts should go.

Example:

```text
Alert:
team=payments
severity=critical
environment=production
```

Policy:

```text
team=payments
      ↓
Payments PagerDuty
```

Another:

```text
severity=warning
      ↓
Platform Slack
```

---

# 33. Routing Tree

A routing hierarchy may look like:

```text
All Alerts
    │
    ├── production
    │      │
    │      ├── critical
    │      │      ↓
    │      │   PagerDuty
    │      │
    │      └── warning
    │             ↓
    │          Slack
    │
    └── non-production
           ↓
        Slack / Email
```

---

# 34. Grouping Alerts

Suppose 20 pods fail at the same time.

Without grouping:

```text
20 alerts
20 notifications
```

With grouping:

```text
20 alert instances
       ↓
Grouped
       ↓
1 notification
```

Grouping reduces alert noise.

---

# 35. Alert Grouping Labels

Group by labels such as:

```text
alertname
cluster
namespace
service
```

Example:

```text
service="payment"
```

can group multiple payment-related alert instances.

Choose grouping labels carefully.

---

# 36. Alert Storm

An alert storm occurs when one underlying incident generates hundreds of alerts.

Example:

```text
Node failure
 ↓
20 pods fail
 ↓
20 pod alerts
 ↓
20 service alerts
 ↓
5 application alerts
```

Result:

```text
45+ notifications
```

This creates alert fatigue.

---

# 37. Alert Fatigue

Alert fatigue occurs when engineers receive too many alerts.

Symptoms:

```text
Too many notifications
Frequent false positives
Repeated alerts
Low trust in alerts
Alerts ignored
```

A good alerting system should prioritize actionable alerts.

---

# 38. Actionable Alert

A good alert should answer:

```text
What is wrong?
Why does it matter?
Who owns it?
What should I do?
```

Example:

```text
Payment API 5xx rate is above SLO.

Service: payment
Environment: production
Severity: critical

Runbook:
<runbook>

Dashboard:
<payment-dashboard>
```

---

# 39. Bad Alert

Bad:

```text
CPU high
```

No:

```text
Service
Environment
Duration
Impact
Runbook
Owner
```

The engineer has to investigate from scratch.

---

# 40. Good Alert

Better:

```text
Payment API CPU saturation

Environment: production
Service: payment
CPU: 94%
Duration: 15 minutes
Severity: warning

Investigate:
Payment service dashboard
Payment runbook
```

---

# 41. Alert Naming

Use descriptive names.

Good:

```text
PaymentHighErrorRate
PaymentHighP95Latency
KubernetesNodeMemoryPressure
PaymentDeploymentUnavailable
```

Avoid:

```text
Alert1
CPU Alert
Test Alert
Problem
```

---

# 42. Alert Naming Convention

A useful pattern:

```text
<service><condition><severity>
```

Example:

```text
PaymentHighErrorRate
PaymentHighLatency
PaymentPodCrashLooping
```

Or:

```text
KubernetesNodeMemoryPressure
KubernetesPodHighRestartRate
```

Consistency is more important than the exact naming style.

---

# 43. High Error Rate Alert

Example PromQL:

```promql
(
  sum(
    rate(http_requests_total{
      service="payment",
      status=~"5.."
    }[5m])
  )
/
  sum(
    rate(http_requests_total{
      service="payment"
    }[5m])
  )
) * 100
```

Condition:

```text
> 5
```

Meaning:

```text
5xx error rate > 5%
```

The metric labels must match the application's instrumentation.

---

# 44. High Latency Alert

For P95 latency:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(
      http_request_duration_seconds_bucket{
        service="payment"
      }[5m]
    )
  )
)
```

Condition:

```text
> 1
```

if the metric is measured in seconds.

---

# 45. High CPU Alert

Example:

```promql
100 *
(
  1 -
  avg by(instance) (
    rate(node_cpu_seconds_total{
      mode="idle"
    }[5m]
  )
)
)
```

Condition:

```text
> 85
```

Again, the threshold should be based on workload behavior.

---

# 46. High Memory Alert

Conceptually:

```promql
100 *
(
  1 -
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
)
```

Condition:

```text
> 90
```

Use a sustained duration to avoid transient spikes.

---

# 47. Pod Restart Alert

Example:

```promql
increase(
  kube_pod_container_status_restarts_total[15m]
) > 3
```

This can identify containers restarting repeatedly.

However, alerting on restarts should consider:

```text
Deployment behavior
Pod lifecycle
Expected restarts
Application type
```

---

# 48. Deployment Availability Alert

Conceptually:

```promql
kube_deployment_status_replicas_available
<
kube_deployment_spec_replicas
```

This detects a mismatch between desired and available replicas.

---

# 49. Pending Pod Alert

Example:

```promql
sum(
  kube_pod_status_phase{
    phase="Pending"
  }
) > 0
```

Do not necessarily page for a single short-lived Pending pod.

A more useful alert may require:

```text
Pending
for 10 minutes
```

and perhaps only for production workloads.

---

# 50. Node Not Ready Alert

Conceptually:

```promql
kube_node_status_condition{
  condition="Ready",
  status="true"
} == 0
```

A production node becoming NotReady may be important.

But alert severity should consider:

```text
Cluster capacity
Number of nodes
Workload distribution
Auto Scaling
```

---

# 51. Disk Usage Alert

Example:

```promql
100 *
(
  1 -
  node_filesystem_avail_bytes
  /
  node_filesystem_size_bytes
)
```

Condition:

```text
> 85
```

The exact filesystem filters should be added to avoid alerting on irrelevant mounts.

---

# 52. Certificate Expiration Alert

TLS certificate expiry can cause outages.

Conceptually:

```text
Certificate expires in:
< 30 days
```

Then:

```text
Warning
```

and:

```text
Certificate expires in:
< 7 days
```

could become:

```text
Critical
```

The actual metric depends on how certificates are monitored.

---

# 53. Availability Alert

If service availability is below the target:

```text
Availability < SLO
```

trigger an alert according to the desired SLO strategy.

For example:

```text
SLO = 99.9%
```

The alert should be designed around actual error budget consumption rather than arbitrary availability thresholds when possible.

---

# 54. SLO-Based Alerting

Instead of:

```text
CPU > 80%
```

a user-impacting alert may be:

```text
Error budget burn rate is too high
```

This focuses alerts on reliability impact.

Architecture:

```text
SLO
 ↓
Error Budget
 ↓
Burn Rate
 ↓
Alert
```

---

# 55. Burn Rate

Burn rate measures how quickly a service is consuming its error budget.

Conceptually:

```text
Normal burn
    ↓
No alert

Fast burn
    ↓
Alert
```

This is generally more meaningful for reliability than monitoring infrastructure metrics alone.

---

# 56. Infrastructure vs User Impact

Not every infrastructure anomaly requires a page.

Example:

```text
CPU = 85%
```

but:

```text
Latency = normal
Errors = normal
Availability = normal
```

This may be:

```text
Warning
```

rather than:

```text
PagerDuty
```

---

# 57. Symptom-Based Alerting

Alert on user-visible symptoms:

```text
High error rate
High latency
Availability failure
Queue backlog
Request failures
```

Then use infrastructure metrics for diagnosis.

This is an important production alerting principle.

---

# 58. Cause-Based Alerts

Cause-based alerts can still be useful.

Examples:

```text
Node NotReady
Disk nearly full
Certificate expiring
Database connection exhaustion
```

But not every possible cause should page the on-call engineer.

---

# 59. Alert Severity Model

Example:

```text
INFO
  ↓
Useful information

WARNING
  ↓
Needs investigation

CRITICAL
  ↓
Immediate action required
```

The severity should map to operational response.

---

# 60. Notification Policy Example

Conceptually:

```text
IF environment=production
AND severity=critical
AND team=payments
        ↓
Payments PagerDuty
```

Another:

```text
IF environment=production
AND severity=warning
        ↓
Production Slack
```

Another:

```text
IF environment!=production
        ↓
Development Slack
```

---

# 61. Mute Timings

Sometimes alerts should not notify during planned maintenance.

Example:

```text
Maintenance:
02:00 - 03:00
```

During that period:

```text
Expected alerts
    ↓
Muted
```

This prevents unnecessary notifications.

---

# 62. Maintenance Windows

A planned deployment may temporarily cause:

```text
Pod restarts
Temporary latency
Expected replica changes
```

Mute or suppress only the appropriate alerts.

Do not broadly silence every alert unless necessary.

---

# 63. Mute vs Disable

These are different concepts.

Mute:

```text
Alert rule still evaluates
Notifications are suppressed
```

Disable:

```text
Alert rule is not actively evaluating
```

Prefer temporary muting for planned maintenance where appropriate.

---

# 64. Alert Silence

A silence can suppress notifications matching specific labels.

Example:

```text
service=payment
environment=staging
```

This avoids muting unrelated production alerts.

---

# 65. Alert Templates

Notification messages should be useful.

Example:

```text
🚨 {{ alertname }}

Severity: {{ severity }}
Environment: {{ environment }}
Service: {{ service }}

Summary:
{{ summary }}

Description:
{{ description }}

Dashboard:
{{ dashboard_url }}

Runbook:
{{ runbook_url }}
```

The exact template syntax depends on the Grafana notification system.

---

# 66. What Should an Alert Message Contain?

At minimum:

```text
Alert name
Severity
Environment
Service
Summary
Description
Current value
Threshold
Dashboard
Runbook
```

Where supported, also include:

```text
Instance
Namespace
Pod
Team
```

---

# 67. Example Production Alert

```text
🚨 PaymentHighErrorRate

Severity: Critical
Environment: Production
Service: payment
Team: payments

Current error rate: 8.2%
Threshold: 5%

Condition has remained true for 10 minutes.

Dashboard:
Payment Service

Runbook:
Payment Incident Runbook
```

This is much more actionable than:

```text
ERROR!!!
```

---

# 68. Alert Routing by Team

Example:

```text
Alert Labels
│
├── team=platform
│       ↓
│    Platform Slack
│
├── team=payments
│       ↓
│    Payments PagerDuty
│
└── team=orders
        ↓
     Orders Slack
```

This allows the monitoring system to route alerts automatically.

---

# 69. Production Alert Architecture

```text
                         Prometheus
                             │
                             ↓
                         Alert Rule
                             │
                             ↓
                        Alert Instance
                             │
                             ↓
                    Notification Policy
                             │
             ┌───────────────┼───────────────┐
             ↓               ↓               ↓
         Platform          Payments        Orders
             │               │               │
             ↓               ↓               ↓
          Slack          PagerDuty         Slack
```

---

# 70. Grafana-Centric Alerting

Grafana can evaluate alert rules against configured data sources.

Architecture:

```text
Grafana
  │
  ├── Prometheus
  ├── Loki
  └── Other Data Sources
  │
  ↓
Alert Rules
  ↓
Notification Policies
  ↓
Contact Points
```

This can provide a centralized alerting experience.

---

# 71. Prometheus Alertmanager vs Grafana Alerting

In a Prometheus ecosystem, there are two common patterns.

### Pattern 1: Prometheus Alerting

```text
Prometheus
   ↓
Alert Rules
   ↓
Alertmanager
   ↓
Notifications
```

### Pattern 2: Grafana Alerting

```text
Grafana
   ↓
Alert Rules
   ↓
Notification Policies
   ↓
Contact Points
```

Both are valid architectures.

---

# 72. Avoid Duplicate Alerting Systems

Do not accidentally configure the same alert in both systems without a clear reason.

For example:

```text
Prometheus:
HighCPU → PagerDuty

Grafana:
HighCPU → PagerDuty
```

Result:

```text
2 notifications
```

for one incident.

Define ownership clearly.

---

# 73. Recommended Alerting Strategy

For a Prometheus-centric Kubernetes environment:

```text
Metrics
   ↓
Prometheus
   ↓
Alert evaluation
   ↓
Alertmanager
   ↓
Routing
```

Grafana:

```text
Dashboard
Visualization
Exploration
Alert visibility
Additional alerting where appropriate
```

Alternatively, use Grafana Alerting as the central alert evaluation and routing layer.

The key is consistency.

---

# 74. Alert Rule as Code

Production alert rules should be version-controlled.

Example repository:

```text
observability/
├── prometheus/
│   └── rules/
│       ├── kubernetes.yaml
│       ├── applications.yaml
│       └── infrastructure.yaml
│
└── grafana/
    └── alerting/
        └── rules/
```

Choose one source of truth for each alerting system.

---

# 75. GitOps Alerting

Recommended flow:

```text
Developer
   ↓
Modify alert rule
   ↓
Git
   ↓
Pull Request
   ↓
Review
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Prometheus / Grafana
```

This provides:

```text
Version control
Auditability
Review
Rollback
Consistency
```

---

# 76. Alert Rule Testing

Before production:

```text
Test query
Test condition
Test labels
Test routing
Test notification
Test recovery
```

Do not only test whether the rule can be created.

Verify that the actual notification reaches the correct destination.

---

# 77. Alert Testing Example

Test:

```text
Error rate > threshold
```

Then verify:

```text
Alert enters Pending
        ↓
Alert enters Firing
        ↓
Slack/Pager receives notification
        ↓
Condition clears
        ↓
Alert returns to Normal
        ↓
Recovery notification if configured
```

---

# 78. Recovery Notifications

When an alert clears:

```text
Firing
   ↓
Condition normal
   ↓
Resolved
```

A recovery notification can tell the team:

```text
Payment error rate has returned to normal.
```

This closes the incident loop.

---

# 79. Alert Lifecycle

Complete lifecycle:

```text
Normal
  ↓
Condition becomes true
  ↓
Pending
  ↓
Firing
  ↓
Notification
  ↓
Engineer investigates
  ↓
Condition resolves
  ↓
Normal
```

---

# 80. Alert Lifecycle During Incident

Example:

```text
10:00
Error rate = 1%
       ↓
Normal

10:05
Error rate = 7%
       ↓
Pending

10:15
Error rate = 8%
       ↓
Firing

10:16
PagerDuty notification

10:30
Rollback completed

10:35
Error rate = 1%
       ↓
Resolved
```

---

# 81. Alert Rule for Deployment Failure

A useful deployment alert:

```text
Desired replicas = 3
Available replicas = 1
```

Condition:

```text
Available < Desired
for 10 minutes
```

Then:

```text
DeploymentUnavailable
```

---

# 82. Alert Rule for CrashLoopBackOff

A direct alert may inspect Kubernetes state metrics.

Conceptually:

```text
Container waiting reason = CrashLoopBackOff
```

Alert:

```text
PodCrashLooping
```

But route severity based on:

```text
Production
Service criticality
Number of replicas
User impact
```

---

# 83. Alert Rule for OOMKilled

Monitor OOMKilled events and restarts.

Example concept:

```text
OOMKilled count > 0
```

For production services, you may want:

```text
Repeated OOMKilled events
```

rather than paging on every isolated event.

---

# 84. Alert Rule for Node Pressure

Monitor:

```text
MemoryPressure
DiskPressure
PIDPressure
```

A node pressure alert should provide:

```text
Node
Cluster
Pressure type
Duration
Current condition
```

---

# 85. Alert Rule for Service Availability

Example:

```text
Available replicas < desired replicas
```

or:

```text
HTTP availability < SLO
```

The second is generally closer to user impact.

---

# 86. Alert Rule for API Errors

Example:

```promql
sum(
  rate(http_requests_total{
    status=~"5.."
  }[5m])
)
```

Then calculate the error ratio.

Alert based on:

```text
Error ratio
AND
Sustained duration
```

---

# 87. Alert Rule for Queue Backlog

For asynchronous systems:

```text
Queue depth
```

can be critical.

Example:

```text
RabbitMQ queue depth > threshold
for 10 minutes
```

This can indicate:

```text
Consumers too slow
Consumer failure
Traffic spike
Downstream dependency failure
```

---

# 88. Alert Rule for Database Connections

Monitor:

```text
Active connections
Maximum connections
Connection utilization
Connection errors
```

Example:

```text
Connection utilization > 90%
```

Use sustained conditions and appropriate severity.

---

# 89. Alert Rule for Disk Capacity

Example:

```text
Disk available < 15%
```

Warning:

```text
< 20%
```

Critical:

```text
< 10%
```

Exact thresholds should be based on storage growth and recovery time.

---

# 90. Alert Rule for Certificate Expiry

Example:

```text
Expiry < 30 days
```

Warning.

```text
Expiry < 7 days
```

Critical.

This gives the team time to renew certificates before an outage.

---

# 91. Alert Rule for ALB Errors

For AWS workloads, monitor:

```text
ALB 4xx
ALB 5xx
Target response time
Unhealthy targets
Request count
```

Correlate ALB alerts with:

```text
Ingress
Service
Pod
Application
```

---

# 92. 503 Alerting Architecture

For a 503 incident:

```text
ALB 5xx
   ↓
Grafana Alert
   ↓
Payment Service Dashboard
   ↓
Service 5xx
   ↓
Pod Availability
   ↓
Readiness
   ↓
Logs
   ↓
Traces
```

This is much more useful than only alerting:

```text
ALB 5xx > 0
```

---

# 93. Alerting and SLOs

A mature alerting system should prioritize:

```text
User impact
SLO violations
Error budget burn
Critical infrastructure failures
```

rather than:

```text
Every unusual metric
```

---

# 94. Alerting and SRE Principles

Good alerts are:

```text
Actionable
Relevant
Reliable
Low-noise
Owned
Documented
```

Bad alerts are:

```text
Noisy
Unclear
Non-actionable
Duplicate
Ownerless
```

---

# 95. Alert Documentation

Every important production alert should document:

```text
Purpose
Trigger
Severity
Owner
Impact
Runbook
Dashboard
Escalation
```

Example:

```text
Alert:
PaymentHighErrorRate

Trigger:
5xx > 5% for 10 minutes

Owner:
Payments Team

Runbook:
Payment incident procedure
```

---

# 96. Alert Runbook

A runbook should tell the engineer:

```text
1. Check dashboard
2. Check recent deployment
3. Check pods
4. Check logs
5. Check dependencies
6. Check traces
7. Roll back if required
8. Escalate if necessary
```

---

# 97. Alert and Deployment Correlation

Suppose:

```text
Deployment v2.4
       ↓
Error rate increases
       ↓
Alert fires
```

The alert notification should ideally include:

```text
Service
Version
Deployment timestamp
Dashboard
Logs
Traces
Runbook
```

This reduces investigation time.

---

# 98. Alert and GitOps

If deployment is managed through GitOps:

```text
Git commit
   ↓
ArgoCD
   ↓
Deployment
   ↓
Metrics change
   ↓
Alert
```

The engineer can correlate the Git change with the alert.

---

# 99. Alert Routing by Environment

Example:

```text
Production
   ↓
PagerDuty / Production Slack

Staging
   ↓
Staging Slack

Development
   ↓
Development Slack
```

Do not page the production on-call for every development alert.

---

# 100. Alert Routing by Severity

Example:

```text
Info
 ↓
No notification / low-priority channel

Warning
 ↓
Slack

Critical
 ↓
PagerDuty
```

The exact policy should be based on incident-management requirements.

---

# 101. Alert Grouping Strategy

Group alerts using meaningful dimensions:

```text
cluster
namespace
service
alertname
```

Example:

```text
service=payment
```

can group several pod-level alerts into a single service-level notification.

---

# 102. Alert Inhibition

Some alerting systems can suppress secondary symptoms when a higher-level failure is already known.

Example:

```text
Node Down
   ↓
Many Pods Unavailable
```

Instead of:

```text
1 Node alert
+
30 Pod alerts
```

the alerting strategy can prioritize the root infrastructure failure.

The exact inhibition mechanism depends on the alerting architecture.

---

# 103. Alert Correlation

Correlation can reduce noise.

Example:

```text
Database unavailable
        ↓
Payment errors
        ↓
Order errors
        ↓
Checkout errors
```

Instead of treating every symptom as an independent incident, identify the common dependency.

---

# 104. Alert Noise Reduction

Methods include:

```text
Sustained conditions
Grouping
Inhibition
Severity levels
SLO-based alerting
Appropriate thresholds
Maintenance muting
Deduplication
```

---

# 105. False Positive

Example:

```text
CPU spikes to 95%
for 10 seconds
```

If the alert fires immediately:

```text
False / low-value alert
```

Better:

```text
CPU > 85%
for 10 minutes
```

when appropriate.

---

# 106. False Negative

The opposite problem:

```text
Error rate = 20%
```

but no alert exists.

Users discover the outage before engineers.

A production alert review should identify important gaps.

---

# 107. Alert Coverage

Review whether you cover:

```text
Availability
Errors
Latency
Saturation
Capacity
Dependencies
Security-sensitive failures
Certificate expiry
Infrastructure health
```

But do not create an alert for every possible metric.

---

# 108. Alert Review

Periodically review:

```text
Which alerts fired?
Which were actionable?
Which were false positives?
Which were ignored?
Which had no owner?
Which should be removed?
Which important incidents were not detected?
```

Alert quality should improve over time.

---

# 109. Alert Metrics

Monitor the alerting system itself.

Useful metrics include:

```text
Alert count
Firing alerts
Notification failures
Notification latency
Alert evaluation errors
```

This helps ensure the alerting platform is reliable.

---

# 110. Alert Delivery Failure

Suppose:

```text
Alert fires
   ↓
Slack unavailable
```

The monitoring system may successfully detect the incident but fail to notify engineers.

Therefore notification channels should be treated as production dependencies.

---

# 111. Multi-Channel Strategy

For critical alerts, consider appropriate redundancy.

Example:

```text
Critical Alert
   ├── PagerDuty
   └── Slack
```

The exact redundancy depends on your incident-management process.

---

# 112. Grafana Alerting High Availability

In a production Grafana deployment:

```text
              Load Balancer
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
      Grafana A           Grafana B
          │                   │
          └─────────┬─────────┘
                    ↓
                PostgreSQL
```

Alerting behavior and HA requirements depend on the Grafana architecture and version.

Do not assume simply adding replicas automatically provides every form of alerting HA.

---

# 113. Alert State Persistence

For production, understand where alert state is stored and how it behaves during:

```text
Grafana restart
Pod failure
Node failure
Deployment
Database failure
```

Test recovery behavior before relying on the system for critical paging.

---

# 114. Alerting in Kubernetes

A common Kubernetes observability architecture is:

```text
Kubernetes
    │
    ├── Metrics
    │     ↓
    │  Prometheus
    │     ↓
    │  Alert Rules
    │
    ├── Logs
    │     ↓
    │  Loki / Elasticsearch
    │
    └── Traces
          ↓
        Jaeger

Grafana
    ↓
Visualization + Alerting
```

---

# 115. Production Alerting Architecture

```text
                         EKS
                          │
          ┌───────────────┼────────────────┐
          ↓               ↓                ↓
      Applications      Nodes           Services
          │               │                │
          └───────────────┼────────────────┘
                          ↓
                     Prometheus
                          │
                    Alert Evaluation
                          │
                          ↓
                    Alert Routing
                          │
             ┌────────────┼────────────┐
             ↓            ↓            ↓
          Platform      Payments      Orders
             ↓            ↓            ↓
           Slack       PagerDuty      Slack
```

---

# 116. Grafana + Prometheus Alerting Architecture

If Grafana Alerting is used:

```text
Prometheus
    ↓
Grafana Data Source
    ↓
Grafana Alert Rule
    ↓
Notification Policy
    ↓
Contact Point
```

---

# 117. Prometheus + Alertmanager Architecture

If Prometheus Alertmanager is used:

```text
Prometheus
    ↓
Alert Rule
    ↓
Alertmanager
    ↓
Grouping
    ↓
Routing
    ↓
Slack / PagerDuty / Email
```

Grafana can then visualize the alert state.

---

# 118. Choosing the Architecture

Choose one clear responsibility model.

For example:

```text
Prometheus
    ↓
Infrastructure / Kubernetes alerting

Grafana
    ↓
Visualization + application-oriented alerting
```

Or:

```text
Grafana
    ↓
Central alerting layer
```

The important point is to avoid accidental duplication.

---

# 119. Alert Rule as Code Example

Conceptually:

```yaml
apiVersion: 1

groups:
  - name: payment-alerts
    rules:
      - alert: PaymentHighErrorRate
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
          ) > 0.05

        for: 10m

        labels:
          severity: critical
          service: payment
          environment: production
          team: payments

        annotations:
          summary: Payment service error rate is high
          description: Payment 5xx error rate has exceeded 5%.
```

This is illustrative; the exact provisioning format depends on the alerting system being used.

---

# 120. Alert Rule Testing in CI

A production GitOps workflow can validate alert rules before deployment.

Conceptually:

```text
Git
 ↓
Pull Request
 ↓
CI
 ├── YAML validation
 ├── Rule validation
 └── Query validation
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
```

This reduces invalid alert configurations reaching production.

---

# 121. Alert Testing with PromQL

Before creating an alert:

```promql
up
```

or the actual production query should be tested in:

```text
Grafana Explore
```

Verify:

```text
Expected result
Labels
Time range
Cardinality
Performance
```

---

# 122. Alert Threshold Testing

Do not select thresholds randomly.

Analyze historical data.

Example:

```text
Normal P95:
150-250ms

Peak P95:
300ms

Alert:
P95 > 1s
```

This is more meaningful than:

```text
P95 > 200ms
```

which could constantly fire during normal traffic.

---

# 123. Alert Duration Testing

Choose a duration based on the problem.

Examples:

```text
CPU spike:
5-15 minutes

Certificate expiry:
Days

Deployment unavailable:
5-10 minutes

Service outage:
Possibly immediate
```

There is no universal duration.

---

# 124. Alert Threshold + Duration

The two work together:

```text
Threshold
    +
Duration
    ↓
Alert quality
```

Example:

```text
CPU > 85%
for 10 minutes
```

is very different from:

```text
CPU > 85%
for 10 seconds
```

---

# 125. Critical vs Warning

Example:

```text
CPU > 80%
for 15m
    ↓
Warning

CPU > 95%
for 5m
    ↓
Critical
```

This creates escalation levels.

But again, use workload-specific thresholds.

---

# 126. Alert Escalation

A critical alert can follow:

```text
Alert
 ↓
Primary on-call
 ↓
No acknowledgement
 ↓
Secondary on-call
 ↓
Escalation
```

The escalation itself is usually handled by an incident-management platform such as PagerDuty rather than Grafana alone.

---

# 127. Alert Ownership

Every production alert should have an owner.

Bad:

```text
High Error Rate
Owner: Nobody
```

Good:

```text
PaymentHighErrorRate
Owner: Payments Team
```

---

# 128. Alert Runbook Link

Every critical alert should ideally include:

```text
runbook_url
```

Example:

```text
PaymentHighErrorRate
      ↓
Payment Incident Runbook
```

The engineer should not need to search the documentation repository manually.

---

# 129. Alert Dashboard Link

Also include:

```text
dashboard_url
```

Example:

```text
Alert
 ↓
Payment Dashboard
```

This makes the first investigation step immediate.

---

# 130. Alert + Trace Link

For trace-aware systems, the alert can direct the engineer toward trace investigation when appropriate.

Flow:

```text
Alert
 ↓
Service Dashboard
 ↓
Logs
 ↓
Trace
```

---

# 131. Alerting and Observability Correlation

The ideal workflow is:

```text
Alert
 ↓
Dashboard
 ↓
Metrics
 ↓
Logs
 ↓
Traces
 ↓
Runbook
 ↓
Remediation
```

This is the complete operational loop.

---

# 132. Alerting Best Practices

Follow these principles:

```text
1. Alert on actionable conditions.
2. Prefer user-impacting symptoms.
3. Use appropriate thresholds.
4. Use sustained durations.
5. Add ownership labels.
6. Add severity.
7. Include runbook links.
8. Include dashboard links.
9. Group related alerts.
10. Avoid duplicate alerts.
11. Review alert quality regularly.
12. Manage alerts as code.
```

---

# 133. Common Alerting Mistakes

Avoid:

```text
Alert on every metric
No duration
No ownership
No severity
No runbook
No dashboard
Duplicate alerts
Too many notifications
Hardcoded environments
Manual production rules
No alert testing
```

---

# 134. Alerting Checklist

```text
[ ] Alert rule defined
[ ] Query tested
[ ] Threshold validated
[ ] Duration configured
[ ] Severity defined
[ ] Service label defined
[ ] Team label defined
[ ] Environment label defined
[ ] Summary added
[ ] Description added
[ ] Dashboard link added
[ ] Runbook link added
[ ] Contact point configured
[ ] Notification policy configured
[ ] Grouping configured
[ ] Maintenance mute strategy defined
[ ] Recovery behavior tested
[ ] Git version control
[ ] Production test completed
```

---

# 135. Interview Answer: What Is Grafana Alerting?

```text
"Grafana alerting allows us to define alert rules against our
observability data sources.

A rule evaluates a query or expression and moves into an alert state
when a defined condition remains true.

The alert is then processed by notification policies and routed to
appropriate contact points such as Slack, email or PagerDuty.

In production, I manage alert rules as code and use labels such as
service, environment, severity and team for routing."
```

---

# 136. Interview Answer: How Do You Prevent Alert Fatigue?

```text
"I focus on actionable alerts rather than alerting on every metric.

I use appropriate thresholds and pending durations to avoid
transient spikes.

I group related alerts and use severity-based routing.

For critical alerts I include the service, environment, impact,
dashboard and runbook information.

I also periodically review alerts to remove noisy or non-actionable
rules."
```

---

# 137. Interview Answer: How Do You Route Alerts?

```text
"I use labels such as team, service, environment and severity.

Notification policies match those labels and route the alert to the
appropriate contact point.

For example, a production critical payment alert can go to the
Payments PagerDuty service, while a warning alert can go to the
Payments Slack channel.

This keeps alert ownership clear."
```

---

# 138. Interview Answer: What Is the Difference Between Labels and Annotations?

```text
"Labels classify and route alerts.

For example:

severity=critical
team=payments
service=payment
environment=production

Annotations provide human-readable information such as the summary,
description, dashboard URL and runbook URL.

So labels are mainly for classification and routing, while
annotations provide context."
```

---

# 139. Interview Answer: How Do You Handle Alert Storms?

```text
"First I identify the common root cause.

Then I use grouping and, where supported, inhibition to avoid sending
many notifications for the same incident.

For example, if a node goes down and causes many pod failures, I
would prioritize the node-level failure rather than paging separately
for every affected pod.

I also review alert hierarchy and severity."
```

---

# 140. Interview Answer: Grafana Alerting vs Alertmanager?

```text
"Prometheus Alertmanager is commonly used with Prometheus for
alert routing, grouping, silencing and notification management.

Grafana also provides an alerting and notification framework that
can evaluate rules against configured data sources.

The important production consideration is to define clear ownership
so the same condition is not evaluated and notified through both
systems unintentionally."
```

---

# 141. Interview Answer: How Would You Alert on High Error Rate?

```text
"I would calculate the ratio of 5xx requests to total requests
rather than simply alerting on the raw number of errors.

For example, I could evaluate the error ratio over a five-minute
window and trigger when it exceeds the service's defined threshold
for a sustained period.

I would add labels for service, environment, severity and team and
route critical production alerts to the appropriate on-call channel."
```

---

# 142. Interview Answer: Why Use Duration in Alerts?

```text
"Duration prevents short-lived spikes from becoming incidents.

For example, if CPU briefly reaches 90% for 20 seconds, I may not
need an alert.

If CPU remains above 90% for 10 minutes, it is much more likely to
represent a real capacity or workload problem.

The exact duration depends on the type of alert."
```

---

# 143. Interview Answer: How Do You Design Production Alerts?

```text
"I start from user impact and service objectives.

I define alerts around availability, error rate, latency, saturation,
capacity and critical dependencies.

Then I assign severity and ownership, configure an appropriate
evaluation window, and provide dashboard and runbook links.

Finally, I test both the firing and recovery paths and manage the
rules through GitOps."
```

---

# 144. Real-World EKS Alerting Architecture

```text
                           EKS
                            │
          ┌─────────────────┼─────────────────┐
          ↓                 ↓                 ↓
      Applications         Nodes          Kubernetes
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ↓
                       Prometheus
                            │
                            ↓
                     Alert Evaluation
                            │
                            ↓
                       Alert State
                            │
                            ↓
                    Notification Policy
                            │
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
          Platform       Payments       Orders
              ↓             ↓             ↓
            Slack       PagerDuty       Slack
```

---

# 145. Real-World Microservices Alerting

For your microservices platform:

```text
User Service
Catalog Service
Cart Service
Orders Service
Payment Service
Inventory Service
Notification Service
```

Each service should have meaningful alerts around:

```text
Availability
Error Rate
Latency
Saturation
Dependency Failure
```

---

# 146. Example Payment Alert Set

```text
PaymentHighErrorRate
PaymentHighP95Latency
PaymentDeploymentUnavailable
PaymentPodCrashLooping
PaymentOOMKilled
PaymentDatabaseConnectionFailure
PaymentQueueBacklog
```

Not every one should necessarily page the on-call engineer.

---

# 147. Example Kubernetes Alert Set

```text
KubernetesNodeNotReady
KubernetesNodeMemoryPressure
KubernetesNodeDiskPressure
KubernetesPodCrashLooping
KubernetesPodOOMKilled
KubernetesDeploymentUnavailable
KubernetesPendingPods
KubernetesPVCAlmostFull
```

Severity should reflect actual impact.

---

# 148. Example Infrastructure Alert Set

```text
ALBHigh5xx
ALBHighLatency
RDSHighCPU
RDSHighConnections
RDSLowStorage
CertificateExpiring
```

These alerts should correlate with application-level symptoms.

---

# 149. Alerting Flow During a Production Incident

```text
Production issue
      ↓
Prometheus detects metric change
      ↓
Alert rule evaluates
      ↓
Condition remains true
      ↓
Alert fires
      ↓
Notification policy matches
      ↓
PagerDuty / Slack
      ↓
Engineer opens Grafana
      ↓
Service dashboard
      ↓
Logs
      ↓
Traces
      ↓
Root cause
      ↓
Remediation
      ↓
Alert resolves
```

---

# 150. Final Alerting Mental Model

Remember:

```text
DATA
 ↓
QUERY
 ↓
CONDITION
 ↓
ALERT RULE
 ↓
ALERT STATE
 ↓
LABELS
 ↓
NOTIFICATION POLICY
 ↓
CONTACT POINT
 ↓
ENGINEER
```

And the production principle is:

```text
Detect problems early.
Alert only when action is required.
Route to the correct owner.
Provide enough context to investigate.
Avoid duplicate and noisy notifications.
Manage alert rules as code.
Continuously review alert quality.
```

A mature alerting system should allow an engineer to go from:

```text
Alert
  ↓
Impact
  ↓
Dashboard
  ↓
Logs
  ↓
Trace
  ↓
Root Cause
  ↓
Runbook
  ↓
Resolution
```

with as little manual searching as possible.
