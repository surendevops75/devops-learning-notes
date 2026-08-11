# Prometheus Alerting

Prometheus alerting is used to detect conditions that require human or automated attention.

Monitoring answers:

```text
What is happening?
```

Alerting answers:

```text
What requires action?
```

A production monitoring platform should not alert on every abnormal metric. It should alert on conditions that are meaningful, actionable, and sustained.

The typical architecture is:

```text
                         Prometheus
                              │
                              ↓
                       PrometheusRule
                              │
                         Alert Rules
                              │
                              ↓
                         Alertmanager
                              │
               ┌──────────────┼──────────────┐
               ↓              ↓              ↓
             Email          Slack          Pager
```

---

# 1. What Is Prometheus Alerting?

Prometheus evaluates alerting rules against time-series data.

Example:

```text
CPU usage > 90%
```

Prometheus evaluates the expression periodically.

If the condition becomes true for the configured duration:

```text
Pending
   ↓
Firing
```

The firing alert is sent to Alertmanager.

---

# 2. Why Do We Need Alerting?

Without alerting:

```text
Prometheus
    ↓
Metrics
    ↓
Grafana
    ↓
Engineer manually checks dashboards
```

With alerting:

```text
Prometheus
    ↓
Alert Rule
    ↓
Condition detected
    ↓
Alertmanager
    ↓
Notification
    ↓
Engineer
```

This allows engineers to react to important incidents without continuously watching dashboards.

---

# 3. Prometheus Alerting Architecture

```text
                  Applications
                       │
                       ↓
                   Exporters
                       │
                       ↓
                   Prometheus
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
         Alert Rules           Metrics
             │                   │
             ↓                   ↓
        Alertmanager          Grafana
             │
       ┌─────┼─────┬───────────┐
       ↓     ↓     ↓           ↓
     Email Slack PagerDuty  Webhook
```

Prometheus is responsible for evaluating alert rules.

Alertmanager is responsible for managing and routing alerts.

---

# 4. Prometheus vs Alertmanager

This distinction is extremely important.

## Prometheus

Prometheus:

```text
Collects metrics
Stores metrics
Evaluates alert rules
Generates alerts
```

## Alertmanager

Alertmanager:

```text
Receives alerts
Groups alerts
Deduplicates alerts
Routes alerts
Silences alerts
Inhibits related alerts
Sends notifications
```

---

# 5. Alerting Flow

The complete flow is:

```text
Application
    ↓
Metrics
    ↓
Exporter / /metrics
    ↓
Prometheus
    ↓
PromQL expression
    ↓
Alert rule
    ↓
Pending
    ↓
Firing
    ↓
Alertmanager
    ↓
Grouping / Routing
    ↓
Notification
```

---

# 6. Basic Alert Rule

A simple Prometheus alert rule:

```yaml
groups:
  - name: example-alerts

    rules:
      - alert: InstanceDown

        expr: up == 0

        for: 5m

        labels:
          severity: critical

        annotations:
          summary: "Instance is down"
          description: "The instance has been unavailable for more than 5 minutes."
```

---

# 7. Understanding an Alert Rule

The important fields are:

```text
alert
expr
for
labels
annotations
```

Example:

```yaml
alert: InstanceDown
```

is the alert name.

```yaml
expr: up == 0
```

is the PromQL expression.

```yaml
for: 5m
```

means the condition must remain true for five minutes before the alert fires.

---

# 8. Labels

Labels classify alerts.

Example:

```yaml
labels:
  severity: critical
  team: platform
```

You can use labels for routing.

For example:

```text
severity=critical
        ↓
Pager

severity=warning
        ↓
Slack
```

---

# 9. Annotations

Annotations contain human-readable information.

Example:

```yaml
annotations:
  summary: "Node is down"

  description: >
    Node {{ $labels.instance }}
    has been unavailable for more than 5 minutes.
```

Annotations are intended to provide useful context to the person receiving the alert.

---

# 10. Alert Name

Use meaningful alert names.

Good:

```text
NodeDown
HighMemoryUsage
PodCrashLooping
DeploymentReplicasMismatch
HighApplicationErrorRate
HighApplicationLatency
```

Avoid vague names:

```text
Problem
Alert1
Issue
ServerProblem
```

---

# 11. Severity

A common severity model is:

```text
critical
warning
info
```

Example:

```yaml
labels:
  severity: critical
```

However, severity should be defined consistently across the organization.

---

# 12. Critical vs Warning

### Warning

Something requires attention but is not necessarily causing immediate customer impact.

Example:

```text
Filesystem usage > 80%
```

### Critical

A condition is actively affecting availability or requires immediate intervention.

Example:

```text
Production service unavailable
```

The actual thresholds should be based on service requirements.

---

# 13. The `for` Duration

Consider:

```yaml
expr: cpu_usage > 90
for: 5m
```

This means:

```text
CPU > 90%
     │
     ├── 1 minute → no firing
     ├── 2 minutes → no firing
     ├── 3 minutes → no firing
     ├── 4 minutes → no firing
     └── 5 minutes → firing
```

This helps reduce alerts caused by short-lived spikes.

---

# 14. Pending State

An alert normally moves through states such as:

```text
Inactive
   ↓
Pending
   ↓
Firing
```

Example:

```text
CPU > threshold
      ↓
Pending
      ↓
Condition remains true
      ↓
Firing
```

---

# 15. Why Use Pending?

Without a `for` duration:

```text
CPU spike
   ↓
Immediate alert
```

With:

```yaml
for: 10m
```

you can require the condition to remain true.

This helps reduce alert noise.

---

# 16. Alert Lifecycle

Conceptually:

```text
                 Condition false
                       ↑
                       │
Inactive ───────→ Pending ───────→ Firing
                    │                │
                    │                │
                    └────────────────┘
                      condition clears
```

When the condition clears, the alert becomes inactive again.

---

# 17. Example: Instance Down

```yaml
groups:
  - name: infrastructure

    rules:

      - alert: InstanceDown

        expr: up == 0

        for: 5m

        labels:
          severity: critical

        annotations:
          summary: "Instance down"
          description: >
            Instance {{ $labels.instance }}
            has been down for more than 5 minutes.
```

---

# 18. Example: High CPU

A conceptual alert:

```yaml
- alert: HighNodeCPU

  expr: |
    100 *
    (1 - avg by(instance)
      (rate(node_cpu_seconds_total{mode="idle"}[5m])))
    > 90

  for: 10m

  labels:
    severity: warning

  annotations:
    summary: "High node CPU"
    description: >
      Node {{ $labels.instance }}
      has CPU utilization above 90%.
```

The exact query should be adapted to the labels and recording rules in your environment.

---

# 19. Example: High Memory

A conceptual rule:

```yaml
- alert: HighNodeMemory

  expr: |
    (
      1 -
      node_memory_MemAvailable_bytes
      /
      node_memory_MemTotal_bytes
    ) * 100 > 90

  for: 10m

  labels:
    severity: warning

  annotations:
    summary: "High node memory"
    description: >
      Node {{ $labels.instance }}
      has memory utilization above 90%.
```

---

# 20. Example: Filesystem Almost Full

```yaml
- alert: FilesystemAlmostFull

  expr: |
    (
      node_filesystem_size_bytes
      -
      node_filesystem_avail_bytes
    )
    /
    node_filesystem_size_bytes
    > 0.85

  for: 15m

  labels:
    severity: warning

  annotations:
    summary: "Filesystem usage is high"
    description: >
      Filesystem {{ $labels.mountpoint }}
      on {{ $labels.instance }}
      is more than 85% full.
```

In production, exclude pseudo-filesystems and other irrelevant mounts using appropriate label filters.

---

# 21. Kubernetes Alerting

Kubernetes alerts should cover:

```text
Node failures
Pod failures
Deployment availability
Container restarts
OOMKilled
Pending pods
PVC issues
Resource pressure
Application errors
Application latency
```

---

# 22. Node Not Ready Alert

A conceptual alert:

```yaml
- alert: KubernetesNodeNotReady

  expr: |
    kube_node_status_condition{
      condition="Ready",
      status="true"
    } == 0

  for: 10m

  labels:
    severity: critical

  annotations:
    summary: "Kubernetes node is not ready"
    description: >
      Kubernetes node {{ $labels.node }}
      has been NotReady for more than 10 minutes.
```

Metric labels can vary depending on the kube-state-metrics version and configuration.

---

# 23. Deployment Replica Mismatch

A useful alert is:

```yaml
- alert: DeploymentReplicasMismatch

  expr: |
    kube_deployment_status_replicas_available
    <
    kube_deployment_spec_replicas

  for: 10m

  labels:
    severity: warning

  annotations:
    summary: "Deployment replicas are unavailable"
    description: >
      Deployment {{ $labels.namespace }}/{{ $labels.deployment }}
      has fewer available replicas than desired.
```

---

# 24. Pod Restart Alert

A simple example:

```yaml
- alert: PodRestartingFrequently

  expr: |
    increase(
      kube_pod_container_status_restarts_total[15m]
    ) > 3

  for: 5m

  labels:
    severity: warning

  annotations:
    summary: "Pod restarting frequently"
    description: >
      Container {{ $labels.container }}
      in pod {{ $labels.namespace }}/{{ $labels.pod }}
      has restarted multiple times.
```

Thresholds should be adjusted based on workload behavior.

---

# 25. Pending Pod Alert

Conceptually:

```yaml
- alert: KubernetesPodPending

  expr: |
    kube_pod_status_phase{
      phase="Pending"
    } == 1

  for: 15m

  labels:
    severity: warning

  annotations:
    summary: "Pod remains pending"
    description: >
      Pod {{ $labels.namespace }}/{{ $labels.pod }}
      has remained Pending for more than 15 minutes.
```

---

# 26. OOMKilled Alert

OOM-related monitoring can be built around container termination reasons.

For example:

```yaml
- alert: ContainerOOMKilled

  expr: |
    kube_pod_container_status_last_terminated_reason{
      reason="OOMKilled"
    } == 1

  for: 5m

  labels:
    severity: warning

  annotations:
    summary: "Container was OOMKilled"
    description: >
      Container {{ $labels.container }}
      in pod {{ $labels.namespace }}/{{ $labels.pod }}
      was terminated because of OOMKilled.
```

In production, consider whether you want to alert on every occurrence or only repeated/recent occurrences.

---

# 27. Kubernetes Disk Pressure

A node experiencing disk pressure can cause:

```text
Pod eviction
Scheduling problems
Container failures
Image-pull issues
```

Monitor the node's DiskPressure condition.

---

# 28. Kubernetes Memory Pressure

Memory pressure can cause:

```text
Pod eviction
OOM events
Scheduling problems
Application instability
```

Monitor both:

```text
Node MemoryPressure
Container memory usage
Container memory limits
```

---

# 29. Application Error Rate Alert

Suppose the application exposes HTTP metrics.

Conceptually:

```yaml
- alert: HighApplicationErrorRate

  expr: |
    (
      sum(rate(http_requests_total{
        status_code=~"5.."
      }[5m]))
      /
      sum(rate(http_requests_total[5m]))
    ) > 0.05

  for: 10m

  labels:
    severity: critical

  annotations:
    summary: "High application error rate"
    description: >
      Application 5xx error rate is above 5%.
```

The metric and label names depend on your application instrumentation.

---

# 30. Application Latency Alert

For histogram-based metrics:

```yaml
- alert: HighApplicationLatency

  expr: |
    histogram_quantile(
      0.95,
      sum by (le) (
        rate(http_request_duration_seconds_bucket[5m])
      )
    ) > 1

  for: 10m

  labels:
    severity: warning

  annotations:
    summary: "High application latency"
    description: >
      Application P95 latency is above 1 second.
```

Choose latency thresholds based on the service's actual SLO.

---

# 31. SLO-Based Alerting

Threshold alerts are useful, but mature organizations often use SLO-based alerting.

Example:

```text
SLO:
99.9% successful requests
```

The alert can be based on error budget consumption rather than:

```text
CPU > 80%
```

This focuses alerting on user impact.

---

# 32. Symptom vs Cause Alerts

Bad alerting:

```text
CPU > 80%
Memory > 80%
Disk > 80%
Load > 5
```

This may generate many alerts during one incident.

Better:

```text
Production API availability degraded
```

with supporting dashboards showing:

```text
CPU
Memory
Latency
Errors
Database
```

Alert on symptoms that require action, while metrics help diagnose the cause.

---

# 33. Alert Fatigue

Alert fatigue happens when engineers receive too many alerts.

Example:

```text
100 alerts/day
```

If most are not actionable:

```text
Engineer
   ↓
Ignores alerts
   ↓
Real incident occurs
   ↓
Critical alert ignored
```

This is dangerous.

---

# 34. Characteristics of a Good Alert

A good alert should be:

```text
Actionable
Specific
Relevant
Timely
Stable
Understandable
Low-noise
```

---

# 35. Every Alert Should Answer

An alert should make it easy to understand:

```text
What is broken?
Where is it broken?
How severe is it?
Who owns it?
What should I check?
```

Example:

```text
HighApplicationErrorRate

Service:
payment

Namespace:
production

Severity:
critical
```

---

# 36. Alert Annotations

Use useful annotations:

```yaml
annotations:
  summary: "Payment service error rate is high"

  description: >
    Payment service in production has a 5xx error rate
    above the configured threshold.

  runbook_url: "https://internal.example/runbooks/payment-errors"
```

A runbook link is extremely valuable in production.

---

# 37. Runbooks

A runbook explains what an engineer should do when an alert fires.

Example:

```text
Alert:
PaymentServiceHighErrorRate

Runbook:
1. Check Grafana payment dashboard.
2. Check recent deployment.
3. Check application logs.
4. Check dependency health.
5. Check database connectivity.
6. Check pod restarts.
7. Roll back if the release caused the issue.
```

---

# 38. Alert Ownership

Alerts should identify an owner.

Example:

```yaml
labels:
  team: payments
  service: payment
  severity: critical
```

Then Alertmanager can route the alert appropriately.

---

# 39. Alertmanager

Alertmanager receives alerts from Prometheus.

Its responsibilities include:

```text
Grouping
Deduplication
Routing
Silencing
Inhibition
Notification
```

---

# 40. Alertmanager Architecture

```text
Prometheus
    │
    │ alerts
    ↓
Alertmanager
    │
    ├── Group
    ├── Deduplicate
    ├── Route
    ├── Silence
    └── Inhibit
          │
          ↓
   Notifications
```

---

# 41. Why Alertmanager?

Suppose ten pods on the same node fail.

Without grouping:

```text
Pod1 failed
Pod2 failed
Pod3 failed
Pod4 failed
...
Pod10 failed
```

You might receive ten notifications.

Alertmanager can group related alerts into one notification.

---

# 42. Alert Grouping

Example:

```yaml
route:
  group_by:
    - alertname
    - namespace
    - service
```

Then related alerts can be grouped.

Example:

```text
HighPodRestarting
namespace=production
service=payment
```

can become one notification group.

---

# 43. Alert Routing

Example:

```text
severity=critical
        ↓
Pager

severity=warning
        ↓
Slack

team=database
        ↓
Database team
```

Alertmanager routes based on labels.

---

# 44. Basic Alertmanager Configuration

Example:

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: default

receivers:
  - name: default
```

Production configuration should define real notification integrations and appropriate routing.

---

# 45. Routing by Severity

Example:

```yaml
route:
  receiver: default

  routes:

    - matchers:
        - severity="critical"
      receiver: critical

    - matchers:
        - severity="warning"
      receiver: warning
```

---

# 46. Team-Based Routing

Example:

```yaml
route:
  receiver: default

  routes:

    - matchers:
        - team="platform"
      receiver: platform-team

    - matchers:
        - team="payments"
      receiver: payments-team

    - matchers:
        - team="database"
      receiver: database-team
```

---

# 47. Notification Channels

Alertmanager can integrate with supported notification mechanisms such as:

```text
Email
Slack
PagerDuty
Webhook
Opsgenie
Microsoft Teams through supported integration patterns
```

The exact integration depends on your organization's tooling.

---

# 48. Email Notifications

Conceptually:

```yaml
receivers:
  - name: email-team

    email_configs:
      - to: devops@example.com
```

Production environments should keep credentials and sensitive configuration secure.

---

# 49. Slack Notifications

Conceptually:

```yaml
receivers:
  - name: slack-team

    slack_configs:
      - channel: "#alerts"
```

Webhook URLs or credentials should not be committed to public repositories.

Use secrets or secure configuration management.

---

# 50. Pager Notifications

Critical alerts should reach an on-call system.

Example flow:

```text
Prometheus
   ↓
Alertmanager
   ↓
Critical Alert
   ↓
On-call system
   ↓
Engineer
```

This is preferable to sending every warning directly to a pager.

---

# 51. Webhooks

Alertmanager can send alerts to an HTTP endpoint.

Architecture:

```text
Alertmanager
     ↓
Webhook
     ↓
Incident / Automation System
```

This can integrate monitoring with internal automation.

---

# 52. Grouping

Grouping reduces notification noise.

Example:

```text
NodeDown
PodDown
DeploymentUnavailable
```

from the same cluster can potentially be grouped according to the routing strategy.

---

# 53. Deduplication

Suppose Prometheus evaluates the same alert repeatedly.

Alertmanager avoids sending the exact same notification continuously.

Conceptually:

```text
Prometheus
 ↓
Same alert
 ↓
Same alert
 ↓
Same alert
 ↓
Alertmanager
 ↓
One notification
```

---

# 54. Repeat Notifications

Alertmanager can resend an unresolved alert after an interval.

Example:

```yaml
route:
  repeat_interval: 4h
```

This is useful for long-running incidents.

Do not set extremely short repeat intervals because they can create alert fatigue.

---

# 55. Group Wait

Alertmanager can wait briefly before sending a grouped notification.

Example:

```yaml
route:
  group_wait: 30s
```

This allows related alerts arriving close together to be grouped.

---

# 56. Group Interval

Example:

```yaml
route:
  group_interval: 5m
```

This controls how often updates for an existing group are sent.

---

# 57. Silence

A silence temporarily prevents notifications for matching alerts.

Example:

```text
Planned maintenance
      ↓
Create silence
      ↓
Expected alerts
      ↓
No notification
```

The alerts can still exist in the monitoring system.

Silencing controls notification behavior.

---

# 58. Silence Example

Suppose you are intentionally restarting:

```text
production-node-01
```

Create a silence matching:

```text
instance="production-node-01"
```

for the maintenance window.

---

# 59. Silence vs Delete Alert Rule

Do not delete an alert rule simply because maintenance is happening.

Use:

```text
Silence
```

for temporary expected conditions.

Keep the monitoring rule intact.

---

# 60. Inhibition

Inhibition suppresses certain alerts when another alert indicates a higher-level failure.

Example:

```text
NodeDown
   ↓
Many PodDown alerts
```

If the node is down, individual pod-down notifications may be redundant.

Alertmanager can inhibit lower-level alerts.

---

# 61. Inhibition Example

Conceptually:

```yaml
inhibit_rules:

  - source_matchers:
      - alertname="NodeDown"

    target_matchers:
      - alertname="PodDown"

    equal:
      - cluster
      - node
```

The exact labels must match your environment.

---

# 62. Alert Hierarchy

A useful model is:

```text
Infrastructure
      ↓
Cluster
      ↓
Node
      ↓
Pod
      ↓
Application
      ↓
Request
```

Higher-level failures can cause many lower-level symptoms.

Alertmanager inhibition can reduce redundant notifications.

---

# 63. Alertmanager High Availability

In production, Alertmanager can run as multiple replicas.

Architecture:

```text
             Prometheus
                 │
        ┌────────┴────────┐
        ↓                 ↓
 Alertmanager A      Alertmanager B
        │                 │
        └────────┬────────┘
                 ↓
           Notifications
```

The exact HA setup should follow the Alertmanager version and deployment model.

---

# 64. Prometheus HA and Alertmanager

With multiple Prometheus replicas:

```text
Prometheus A ───┐
                ├──→ Alertmanager
Prometheus B ───┘
```

Both may evaluate the same alert.

Alertmanager deduplication and HA behavior are important in this architecture.

---

# 65. Alert Rule Management in Kubernetes

With Prometheus Operator, use:

```text
PrometheusRule
```

instead of manually editing Prometheus configuration.

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule

metadata:
  name: application-alerts
  namespace: monitoring

spec:
  groups:
    - name: application.rules

      rules:
        - alert: HighApplicationErrorRate
          expr: ...
          for: 10m

          labels:
            severity: critical

          annotations:
            summary: "High application error rate"
```

---

# 66. PrometheusRule Architecture

```text
Git
 ↓
PrometheusRule YAML
 ↓
ArgoCD
 ↓
Kubernetes
 ↓
Prometheus Operator
 ↓
Prometheus
 ↓
Alert Evaluation
```

This is an excellent GitOps pattern.

---

# 67. GitOps Alert Management

A monitoring repository might contain:

```text
monitoring/
├── alerts/
│   ├── node-alerts.yaml
│   ├── kubernetes-alerts.yaml
│   ├── application-alerts.yaml
│   └── database-alerts.yaml
```

Then:

```text
Pull Request
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
 ↓
EKS
```

---

# 68. Alert Rule Testing

Do not deploy alert rules without validation.

Validate:

```text
YAML syntax
PromQL syntax
Label behavior
Threshold
Evaluation interval
Annotations
Routing
Runbook
```

---

# 69. PromQL Testing

Before using:

```yaml
expr: some_expression
```

test it in Prometheus.

Verify:

```text
Does it return data?
Does it return expected labels?
Does it trigger under the intended condition?
Does it remain silent when healthy?
```

---

# 70. Alert Simulation

A useful testing approach:

```text
Normal
   ↓
Create controlled failure
   ↓
Metric changes
   ↓
Alert becomes Pending
   ↓
Alert becomes Firing
   ↓
Alertmanager receives it
   ↓
Notification arrives
   ↓
Recover
   ↓
Alert resolves
```

---

# 71. Test NodeDown

In a non-production environment:

```text
Stop Node Exporter
```

or isolate the target.

Then verify:

```text
up == 0
```

and:

```text
InstanceDown
```

fires after the configured duration.

---

# 72. Test High CPU

In a controlled environment:

```text
Generate CPU load
```

Then verify:

```text
CPU metric increases
      ↓
PromQL condition true
      ↓
Alert pending
      ↓
Alert firing
```

Do not run destructive load tests in production without an approved procedure.

---

# 73. Test Alertmanager Routing

Create a temporary test alert:

```yaml
- alert: AlertmanagerRoutingTest

  expr: vector(1)

  for: 1m

  labels:
    severity: warning
    team: platform

  annotations:
    summary: "Alertmanager routing test"
```

This should fire and verify the expected notification route.

Remove the test after validation.

---

# 74. Alert Resolution

When the expression becomes false:

```text
Firing
  ↓
Resolved
```

Alertmanager can send a resolved notification depending on receiver configuration.

---

# 75. Resolved Notifications

A useful notification can show:

```text
ALERT:
HighApplicationErrorRate

STATUS:
Resolved

SERVICE:
payment

DURATION:
12 minutes
```

This tells engineers that the incident condition has cleared.

---

# 76. Alert Labels for Routing

Recommended labels may include:

```text
severity
team
service
namespace
environment
cluster
```

Example:

```yaml
labels:
  severity: critical
  team: platform
  service: payment
  environment: production
```

Do not create unnecessary labels.

---

# 77. Environment-Based Routing

You may want:

```text
production
    ↓
Pager / critical notification

staging
    ↓
Slack

development
    ↓
No notification / low-priority channel
```

This can prevent development alerts from interrupting production on-call engineers.

---

# 78. Production vs Non-Production Alerts

Do not necessarily use identical thresholds everywhere.

For example:

```text
Production:
5% error rate → critical

Staging:
10% error rate → warning
```

Thresholds should reflect the purpose and expected behavior of each environment.

---

# 79. Alert Routing Tree

Example:

```text
                         Alertmanager
                              │
                        severity?
                              │
              ┌───────────────┴───────────────┐
              ↓                               ↓
           critical                         warning
              │                               │
              ↓                               ↓
         team/service                       Slack
              │
       ┌──────┴──────┐
       ↓             ↓
   platform       payments
       │             │
       ↓             ↓
     Pager          Pager
```

---

# 80. Alert Ownership Model

Every critical alert should have:

```text
Owner
Runbook
Dashboard
Severity
Service
Environment
```

Example:

```text
Alert:
PaymentHighErrorRate

Owner:
Payments Team

Dashboard:
Payment Service

Runbook:
Payment Error Runbook

Severity:
Critical
```

---

# 81. Alert Documentation

A production alert should be understandable even by an engineer who did not create it.

Bad:

```text
HighCPU
```

Better:

```text
Node CPU utilization has remained above
90% for 10 minutes in production.
```

---

# 82. Alert Naming Convention

A consistent convention might be:

```text
<Node/Application><Condition>
```

Examples:

```text
NodeDown
NodeHighCPU
NodeHighMemory
PodCrashLooping
PodOOMKilled
DeploymentReplicasMismatch
ApplicationHighErrorRate
ApplicationHighLatency
DatabaseConnectionSaturation
```

---

# 83. Recording Rules vs Alert Rules

Recording rules calculate and store frequently used expressions.

Alert rules detect conditions.

Example recording rule:

```yaml
- record: job:http_requests:rate5m

  expr: |
    sum by(job) (
      rate(http_requests_total[5m])
    )
```

Then an alert can use the recording rule.

---

# 84. Why Use Recording Rules?

Recording rules are useful when:

```text
Complex query
Frequently used query
Large dashboard
Large number of alerts
Expensive PromQL
```

Instead of recalculating the same expensive expression repeatedly, precompute it.

---

# 85. Recording Rule Architecture

```text
Raw Metrics
     ↓
Recording Rule
     ↓
Precomputed Time Series
     ↓
Dashboard
     ↓
Alert
```

This can improve query efficiency.

---

# 86. Alert Rule Evaluation Interval

Prometheus evaluates rules periodically.

For example:

```yaml
global:
  evaluation_interval: 30s
```

This means rule evaluation occurs every 30 seconds.

Choose intervals based on operational requirements and system scale.

---

# 87. Scrape Interval vs Evaluation Interval

These are different.

### Scrape interval

How often Prometheus collects metrics.

```text
30s
```

### Evaluation interval

How often Prometheus evaluates rules.

```text
30s
```

They do not have to be identical.

---

# 88. Alert Timing

Suppose:

```text
Scrape interval = 30s
Evaluation = 30s
for = 5m
```

A condition may need to remain continuously true long enough to satisfy the `for` duration before the alert becomes firing.

Alert timing should account for scrape and evaluation behavior.

---

# 89. Alert Flapping

Alert flapping occurs when an alert repeatedly changes:

```text
Firing
 ↓
Resolved
 ↓
Firing
 ↓
Resolved
```

Causes can include:

```text
Threshold too close to normal
Noisy metric
Short evaluation period
Insufficient `for`
Application instability
```

---

# 90. Preventing Flapping

Use:

```text
for duration
Appropriate thresholds
Recording rules
Hysteresis where applicable
Smoothing
Correct metric selection
```

Do not make thresholds unnecessarily sensitive.

---

# 91. Multi-Window Alerting

For important SLO alerts, use multiple windows.

Conceptually:

```text
Short window
+
Long window
```

This can detect both:

```text
Fast severe incidents
Slow sustained degradation
```

This is more advanced than simple threshold alerting.

---

# 92. Burn Rate Alerting

For SLO-based monitoring, burn rate represents how quickly an application is consuming its error budget.

Example concept:

```text
SLO:
99.9%

Error budget:
0.1%

Current burn rate:
20x
```

A high burn rate means the service is consuming its allowed error budget much faster than expected.

---

# 93. Why Burn Rate Is Useful

CPU alerts tell you:

```text
Resource is high.
```

Burn-rate alerts tell you:

```text
User-facing reliability is deteriorating
fast enough to threaten the SLO.
```

This is usually more meaningful for critical services.

---

# 94. Alerting for Microservices

For a microservices platform:

```text
User Service
Product Service
Cart Service
Order Service
Payment Service
Inventory Service
Notification Service
```

Each service can have:

```text
Availability
Error rate
Latency
Traffic
Resource saturation
Dependency health
```

---

# 95. Example Microservices Alert Architecture

```text
                     Prometheus
                         │
         ┌───────────────┼────────────────┐
         ↓               ↓                ↓
       User            Order           Payment
      Alerts           Alerts           Alerts
         │               │                │
         └───────────────┼────────────────┘
                         ↓
                    Alertmanager
                         │
              ┌──────────┼──────────┐
              ↓          ↓          ↓
          Platform     Orders     Payments
             Team       Team        Team
```

---

# 96. Database Alerting

Database alerts can include:

```text
Database unavailable
Connection saturation
Replication lag
High query latency
Storage nearly full
Too many connections
Locks
```

The exact alerts depend on the database.

---

# 97. Redis Alerting

Useful Redis alerts may include:

```text
Redis unavailable
Memory pressure
Evictions increasing
Replication problems
Connection problems
```

Use the metrics provided by the Redis exporter or native metrics.

---

# 98. Blackbox Alerting

Blackbox Exporter enables alerts such as:

```text
Endpoint unavailable
HTTP status unexpected
High endpoint latency
TLS certificate expiry approaching
DNS probe failure
TCP connectivity failure
```

---

# 99. Certificate Expiry Alert

A conceptual rule:

```promql
probe_ssl_earliest_cert_expiry - time() < 604800
```

This checks whether a certificate expires in less than seven days.

The exact probe metric must be available and the threshold should match your certificate-management process.

---

# 100. Endpoint Availability Alert

A simple rule:

```yaml
- alert: EndpointDown

  expr: probe_success == 0

  for: 5m

  labels:
    severity: critical

  annotations:
    summary: "Endpoint is unavailable"
    description: >
      Endpoint {{ $labels.instance }}
      has failed probing for more than 5 minutes.
```

---

# 101. Alerting for Kubernetes Ingress

Monitor:

```text
ALB availability
Target health
HTTP 4xx
HTTP 5xx
Latency
Application availability
```

A useful architecture is:

```text
ALB metrics
      +
Ingress/application metrics
      ↓
Prometheus
      ↓
Alertmanager
```

---

# 102. Alerting on 503 Errors

If users receive 503 errors:

```text
HTTP 503 rate ↑
```

alert on application or ingress metrics.

Then investigate:

```text
Service endpoints
Pod readiness
Application health
ALB target health
Ingress configuration
```

---

# 103. Alerting on Deployment Rollouts

You can alert when:

```text
Desired replicas != Available replicas
```

for a sustained duration.

This catches deployments that technically completed but did not result in a healthy application.

---

# 104. Alerting on New Deployments

Avoid blindly alerting every time a deployment occurs.

Instead, correlate:

```text
Deployment
+
Error rate increase
+
Latency increase
```

This can help identify release-related incidents.

---

# 105. Alerting and CI/CD

A useful production flow:

```text
GitHub Actions
      ↓
Build
      ↓
Test
      ↓
Security Scan
      ↓
Image
      ↓
ArgoCD
      ↓
EKS
      ↓
Prometheus
      ↓
Health Metrics
      ↓
Alert
```

This creates feedback from deployment into production observability.

---

# 106. Deployment Failure Detection

A deployment pipeline may report:

```text
Deployment succeeded
```

but Prometheus may later detect:

```text
High error rate
High latency
Pod restarts
Replica mismatch
```

This is why CI/CD success is not equivalent to production health.

---

# 107. Alerting and Rollback

If a deployment causes:

```text
Error rate ↑
Latency ↑
Pods restarting
```

an incident process can be:

```text
Alert
 ↓
Investigate recent deployment
 ↓
Confirm correlation
 ↓
Rollback
 ↓
Monitor recovery
```

Rollback should be based on evidence and established deployment procedures, not on a single noisy metric.

---

# 108. Alertmanager Configuration in Kubernetes

With `kube-prometheus-stack`, Alertmanager is commonly deployed as part of the stack.

Check:

```bash
kubectl get pods -n monitoring
```

You may see:

```text
alertmanager-...
prometheus-...
grafana-...
```

---

# 109. Alertmanager Service

Check:

```bash
kubectl get svc -n monitoring
```

Find the Alertmanager service.

Temporary access can be done with:

```bash
kubectl port-forward \
  svc/monitoring-kube-prometheus-alertmanager \
  9093:9093 \
  -n monitoring
```

The exact service name depends on the Helm release.

---

# 110. Alertmanager Configuration Through Helm

Production configuration should be managed through a version-controlled values file.

Conceptually:

```yaml
alertmanager:
  enabled: true

  alertmanagerSpec:
    replicas: 2

  config:
    global:
      resolve_timeout: 5m

    route:
      receiver: default
```

The exact chart schema changes across versions, so always validate the values against the installed chart version.

---

# 111. Alertmanager Secrets

Do not commit sensitive notification credentials directly to Git.

For example:

```text
Slack webhook
SMTP password
Pager credentials
API tokens
```

should be managed using:

```text
Kubernetes Secrets
External Secrets
AWS Secrets Manager
Vault
```

according to your environment.

---

# 112. Alertmanager Security

Protect the Alertmanager UI.

Do not expose it publicly without authentication and appropriate access controls.

Production architecture:

```text
Engineer
   ↓
Authenticated Access
   ↓
Internal Alertmanager
```

---

# 113. Alertmanager Monitoring

Alertmanager itself should be monitored.

Useful areas include:

```text
Availability
Notification failures
Notification latency
Alerts received
Alerts pending
Cluster state
```

---

# 114. Prometheus Alerting Self-Monitoring

Prometheus should alert when:

```text
Prometheus is unavailable
Scrape failures increase
Rule evaluation fails
Alertmanager is unavailable
Storage is near capacity
```

This creates a monitoring feedback loop.

---

# 115. Monitoring Stack Alert Hierarchy

```text
                    Monitoring
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
      Application    Kubernetes     Monitoring
          │             │             │
          ↓             ↓             ↓
       Errors         Nodes        Prometheus
       Latency        Pods         Alertmanager
       Traffic        Storage      Grafana
       Saturation     Capacity
```

---

# 116. Common Alerting Mistakes

## Mistake 1: Alerting on Everything

Result:

```text
Alert fatigue
```

---

## Mistake 2: No `for`

Result:

```text
Transient spikes trigger alerts.
```

---

## Mistake 3: No Ownership

Nobody knows who should respond.

---

## Mistake 4: No Runbook

Engineers receive an alert but do not know what to do.

---

## Mistake 5: No Severity

Everything looks equally important.

---

## Mistake 6: No Grouping

One incident generates hundreds of notifications.

---

## Mistake 7: Public Alertmanager

This creates unnecessary security exposure.

---

# 117. Production Alert Design

For each important service, define:

```text
SLI
SLO
Critical alerts
Warning alerts
Runbooks
Dashboard
Owner
Escalation path
```

---

# 118. Example Production Alert Catalog

```text
Platform:
  NodeDown
  NodeHighCPU
  NodeHighMemory
  NodeDiskPressure

Kubernetes:
  PodCrashLooping
  PodOOMKilled
  DeploymentReplicasMismatch
  PodPending
  PVCProblem

Application:
  ApplicationHighErrorRate
  ApplicationHighLatency
  ApplicationUnavailable

Database:
  DatabaseDown
  DatabaseConnectionSaturation
  DatabaseStorageLow
  DatabaseReplicationLag

External:
  EndpointDown
  EndpointLatencyHigh
  CertificateExpiring
```

---

# 119. Alert Testing in Staging

Before production:

```text
Development
    ↓
Staging
    ↓
Test alerts
    ↓
Test routing
    ↓
Test silences
    ↓
Test resolution
    ↓
Production
```

Never assume an alert works simply because the YAML is valid.

---

# 120. Production Alert Review

Review alerts periodically.

Ask:

```text
Did this alert fire recently?
Was it actionable?
Did someone respond?
Was it noisy?
Was the threshold correct?
Was the runbook useful?
Should it be removed?
```

Monitoring systems should evolve with the applications.

---

# 121. Alert Quality Metrics

Organizations can measure:

```text
Alert volume
Alert-to-incident ratio
False positive rate
Mean time to acknowledge
Mean time to resolve
Repeated alerts
Noisy alerts
```

This helps improve the alerting system itself.

---

# 122. Alert Fatigue Reduction Strategy

Use:

```text
1. Better thresholds
2. `for` durations
3. Grouping
4. Deduplication
5. Inhibition
6. Silencing
7. SLO-based alerting
8. Runbooks
9. Ownership
10. Regular alert reviews
```

---

# 123. Alerting Best Practices

### Keep alerts actionable

If nobody can act on an alert, reconsider whether it should page someone.

### Prefer symptoms for paging

User-facing failures are usually more important than isolated infrastructure anomalies.

### Use warnings for early signals

Warnings can provide advance notice before a critical incident.

### Add context

Include:

```text
service
namespace
cluster
instance
severity
runbook
```

where useful.

---

# 124. Production Alert Example

A strong alert:

```yaml
- alert: PaymentServiceHighErrorRate

  expr: |
    (
      sum(rate(http_requests_total{
        service="payment",
        status_code=~"5.."
      }[5m]))
      /
      sum(rate(http_requests_total{
        service="payment"
      }[5m]))
    ) > 0.05

  for: 10m

  labels:
    severity: critical
    team: payments
    service: payment
    environment: production

  annotations:
    summary: "Payment service has a high 5xx error rate"

    description: >
      Payment service in production has maintained
      a 5xx error rate above 5% for 10 minutes.

    runbook_url: "https://internal.example/runbooks/payment-errors"
```

---

# 125. Alerting Architecture for Your EKS Environment

A practical architecture is:

```text
                         EKS
                          │
        ┌─────────────────┼──────────────────┐
        ↓                 ↓                  ↓
      Nodes             Pods              Objects
        │                 │                  │
        ↓                 ↓                  ↓
 Node Exporter       App Metrics       Kube-State-Metrics
        │                 │                  │
        └─────────────────┼──────────────────┘
                          ↓
                    Prometheus
                          │
                    Alert Rules
                          │
                          ↓
                    Alertmanager
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       Platform         Payments        Database
          │               │               │
          ↓               ↓               ↓
        Pager           Pager           Pager
          │
          ↓
       Engineers
```

---

# 126. Complete GitOps Alerting Flow

For a GitOps-managed EKS platform:

```text
Developer
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
PrometheusRule
    ↓
Prometheus Operator
    ↓
Prometheus
    ↓
Alert
    ↓
Alertmanager
    ↓
Notification
```

This provides a controlled and auditable alerting workflow.

---

# 127. Recommended Monitoring Repository

A production repository can be structured as:

```text
monitoring/
├── prometheus/
│   ├── values.yaml
│   └── recording-rules/
│
├── alerts/
│   ├── infrastructure.yaml
│   ├── kubernetes.yaml
│   ├── applications.yaml
│   ├── databases.yaml
│   └── external-services.yaml
│
├── alertmanager/
│   └── values.yaml
│
├── servicemonitors/
│
├── podmonitors/
│
└── dashboards/
```

---

# 128. Production Alerting Checklist

```text
[ ] Alert rules are version controlled
[ ] PromQL expressions tested
[ ] Appropriate `for` duration configured
[ ] Severity defined
[ ] Ownership defined
[ ] Runbook available
[ ] Dashboard available
[ ] Alertmanager configured
[ ] Routing configured
[ ] Grouping configured
[ ] Deduplication enabled
[ ] Inhibition configured where appropriate
[ ] Silencing process defined
[ ] Notification channels tested
[ ] Critical alerts tested
[ ] Resolved notifications tested
[ ] Alertmanager secured
[ ] Alertmanager monitored
[ ] Prometheus monitored
```

---

# 129. Troubleshooting Alert Not Firing

Use this sequence:

```text
1. Does the metric exist?
        ↓
2. Does the PromQL expression return data?
        ↓
3. Is the expression actually true?
        ↓
4. Has the `for` duration elapsed?
        ↓
5. Is the rule loaded?
        ↓
6. Is the rule evaluation succeeding?
        ↓
7. Is Alertmanager reachable?
        ↓
8. Is the alert routed?
        ↓
9. Is the receiver configured?
        ↓
10. Did the notification provider accept it?
```

---

# 130. Check Prometheus Rules

In Kubernetes:

```bash
kubectl get prometheusrule -A
```

Then:

```bash
kubectl describe prometheusrule <name> -n <namespace>
```

Also check the Prometheus UI for loaded rules.

---

# 131. Rule Exists but Does Not Fire

Test the expression directly in Prometheus.

For example:

```promql
up == 0
```

If it returns no series:

```text
The condition is currently false.
```

If it returns a series:

```text
Investigate alert state,
for duration,
labels,
and rule evaluation.
```

---

# 132. Alert Fires but No Notification

Check:

```text
Prometheus
   ↓
Alertmanager
   ↓
Routing
   ↓
Receiver
   ↓
Notification provider
```

Investigate each stage.

---

# 133. Alertmanager Routing Troubleshooting

Check:

```text
Alert labels
Route matchers
Receiver name
Grouping
Silence
Inhibition
Receiver configuration
```

A perfectly valid alert can still produce no notification if routing intentionally suppresses it.

---

# 134. Alert Is Silenced

If the alert exists but no notification is sent:

Check Alertmanager silences.

A maintenance silence may be matching:

```text
alertname
cluster
namespace
service
instance
```

Remove or allow the silence to expire when appropriate.

---

# 135. Alert Is Inhibited

If an alert is not notifying:

Check whether another higher-level alert is suppressing it.

Example:

```text
NodeDown
    ↓
PodDown inhibited
```

This is intentional in many production architectures.

---

# 136. Alertmanager Logs

Check:

```bash
kubectl logs \
  <alertmanager-pod> \
  -n monitoring
```

Look for:

```text
Routing errors
Receiver errors
Webhook errors
SMTP errors
Configuration errors
```

---

# 137. Prometheus Logs

Check:

```bash
kubectl logs \
  <prometheus-pod> \
  -n monitoring
```

Look for:

```text
Rule evaluation errors
Configuration errors
Remote write problems
Storage issues
```

---

# 138. Alertmanager UI

The Alertmanager UI helps identify:

```text
Active alerts
Silences
Groups
Labels
Receivers
```

Use it during troubleshooting instead of assuming the problem is with Prometheus.

---

# 139. Interview Question: What Is Alertmanager?

Strong answer:

```text
"Alertmanager is the component that handles alerts generated by Prometheus.

Prometheus evaluates alerting rules and sends firing alerts to Alertmanager.

Alertmanager then groups related alerts, deduplicates them, routes them based on labels, supports silences and inhibition, and sends notifications to systems such as email, Slack or an on-call platform.

So Prometheus decides that an alert condition exists, while Alertmanager manages how that alert is delivered."
```

---

# 140. Interview Question: What Is the Difference Between Prometheus and Alertmanager?

```text
Prometheus:
- Collects metrics
- Stores time series
- Evaluates PromQL
- Evaluates alert rules
- Sends alerts

Alertmanager:
- Receives alerts
- Groups alerts
- Deduplicates alerts
- Routes alerts
- Silences alerts
- Inhibits alerts
- Sends notifications
```

---

# 141. Interview Question: What Is a PrometheusRule?

```text
"A PrometheusRule is a Kubernetes custom resource provided by the Prometheus Operator ecosystem.

It allows us to define recording rules and alerting rules declaratively in Kubernetes.

In a GitOps environment, I can store PrometheusRule manifests in Git and use ArgoCD to deploy them to EKS.

Prometheus Operator then makes those rules available to Prometheus."
```

---

# 142. Interview Question: What Is the Difference Between Alert Labels and Annotations?

```text
"Labels identify and classify the alert.

They are useful for routing, grouping and filtering.

For example:
severity=critical
team=payments

Annotations provide human-readable context.

For example:
summary
description
runbook URL

So labels are mainly for alert metadata and routing, while annotations are mainly for notification context."
```

---

# 143. Interview Question: Why Do We Use `for` in Alert Rules?

```text
"The `for` duration prevents short-lived metric spikes from immediately triggering an alert.

For example, if CPU temporarily reaches 95% for 20 seconds, I may not want a page.

If CPU remains above the threshold for 10 minutes, then the alert becomes actionable.

This reduces alert noise and flapping."
```

---

# 144. Interview Question: What Is Alert Fatigue?

```text
"Alert fatigue occurs when engineers receive too many non-actionable or noisy alerts.

Eventually they may start ignoring notifications, which is dangerous during a real incident.

I reduce alert fatigue using meaningful thresholds, `for` durations, alert grouping, deduplication, inhibition, severity-based routing, and SLO-based alerting."
```

---

# 145. Interview Question: What Is Alert Inhibition?

```text
"Inhibition allows Alertmanager to suppress lower-level alerts when a higher-level alert is already firing.

For example, if an entire Kubernetes node is down, many pods on that node may also appear down.

Instead of sending separate notifications for every pod, I can configure Alertmanager so that NodeDown inhibits related PodDown alerts.

This reduces noise while preserving the primary incident signal."
```

---

# 146. Interview Question: What Is Alert Silencing?

```text
"A silence temporarily prevents matching alerts from generating notifications.

I use it for planned maintenance or other expected temporary conditions.

I do not delete the alert rule because the monitoring condition should continue to be evaluated.

The silence only controls notification behavior."
```

---

# 147. Interview Question: How Would You Design Production Alerting?

```text
"I start with user impact and service objectives rather than creating alerts for every metric.

For each critical service I define SLIs and SLOs, then create actionable alerts for availability, error rate, latency and important saturation conditions.

I classify alerts by severity and ownership, add runbook links, and route them through Alertmanager.

I use grouping, deduplication and inhibition to reduce noise.

All alert rules are stored in Git and deployed through GitOps, and I periodically review alert quality based on false positives and operational usefulness."
```

---

# 148. Interview Question: A Deployment Succeeded but Alert Fired. Why?

```text
"CI/CD success only means the deployment process completed successfully.

It does not guarantee that the application is healthy in production.

After deployment, the application may have high error rates, latency, readiness failures, dependency problems or resource issues.

Prometheus provides production health signals, so I would correlate the alert with the deployment timestamp, application metrics, pod health, logs and dependencies before deciding whether a rollback is required."
```

---

# 149. Interview Question: How Would You Design Alerts for Kubernetes?

```text
"I would use multiple layers.

At the node level:
NodeDown, CPU, memory, disk and pressure conditions.

At the Kubernetes layer:
Pod restart spikes, OOMKilled, Pending pods, deployment replica mismatch and PVC issues.

At the application layer:
Error rate, latency, availability and traffic.

I would avoid paging directly on every resource threshold. Critical alerts should represent actionable conditions, while warning alerts can provide early signals."
```

---

# 150. Final Prometheus Alerting Architecture

```text
                              EKS
                               │
       ┌───────────────────────┼────────────────────────┐
       ↓                       ↓                        ↓
    Node Metrics          Application Metrics      Kubernetes State
       │                       │                        │
       ↓                       ↓                        ↓
 Node Exporter             /metrics             Kube-State-Metrics
       │                       │                        │
       └───────────────────────┼────────────────────────┘
                               ↓
                          Prometheus
                               │
                  ┌────────────┴────────────┐
                  ↓                         ↓
             Alert Rules                Metrics
                  │                         │
                  ↓                         ↓
             Alertmanager                Grafana
                  │
       ┌──────────┼───────────┐
       ↓          ↓           ↓
     Slack       Email       Pager
       │          │           │
       └──────────┼───────────┘
                  ↓
             DevOps Team
```

---

# 151. Final Alerting Mental Model

Remember:

```text
Prometheus
    =
metrics + rule evaluation

PrometheusRule
    =
declarative alert/recording rules

Alertmanager
    =
grouping + routing + deduplication
+ silencing + inhibition + notifications

Grafana
    =
visualization + investigation
```

And the production flow:

```text
Metric
  ↓
PromQL
  ↓
Alert Rule
  ↓
Pending
  ↓
Firing
  ↓
Alertmanager
  ↓
Grouping
  ↓
Routing
  ↓
Notification
  ↓
Engineer
  ↓
Runbook
  ↓
Investigation
  ↓
Recovery
```

The goal of production alerting is **not to generate as many alerts as possible**.

The goal is to generate **the right alert, at the right time, with enough context for the right engineer to take action**.
