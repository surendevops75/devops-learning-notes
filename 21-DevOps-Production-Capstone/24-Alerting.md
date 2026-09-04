# Production Alerting

> **Project:** DevOps Production Capstone  
> **Environment:** AWS + EKS + Kubernetes + Helm + Argo CD + Prometheus + Grafana + ELK  
> **Application:** RoboShop-style microservices platform  
> **Purpose:** Production-grade metrics, alerting, routing, escalation, incident detection, and operational response

---

## 1. Chapter Objectives

This chapter builds a production alerting system around:

- Amazon EKS
- Kubernetes
- Prometheus
- Prometheus Operator / kube-prometheus-stack style deployment
- PrometheusRule
- Alertmanager
- Grafana
- ELK
- AWS ALB
- Application SLI/SLO metrics
- Argo CD
- CI/CD
- Security signals
- On-call and incident management

The goal is not simply to create alerts.

The goal is to create **actionable alerts that detect meaningful production problems, route them to the correct owners, reduce noise, provide enough context for responders, and support reliable incident response.**

---

# 2. Alerting Fundamentals

## 2.1 What Is Alerting?

Alerting is the process of automatically detecting an abnormal or dangerous condition and notifying the appropriate operational team.

A monitoring system answers:

> "What is happening?"

An alerting system answers:

> "What requires human attention now?"

For example:

```text
Prometheus:
checkout request error rate = 18%

Alert rule:
error rate > 5% for 5 minutes

Alertmanager:
severity = critical
team = payments
environment = prod

Notification:
on-call receives incident
```

Alerting is therefore an operational decision layer above observability data.

---

# 3. Monitoring vs Alerting

| Monitoring | Alerting |
|---|---|
| Observes systems | Detects actionable conditions |
| Provides metrics/logs | Generates notifications |
| Used for dashboards | Used for response |
| Usually passive | Usually event-driven |
| Helps investigation | Initiates investigation |
| Can contain huge amounts of data | Should be selective |

A dashboard can contain hundreds of metrics.

An alert should normally represent something that requires action.

### Bad alert

```text
CPU > 50%
```

This may be completely normal.

### Better alert

```text
CPU > 90% for 15 minutes
AND
CPU throttling is high
AND
request latency is increasing
```

The second alert is closer to an operational problem.

---

# 4. Why Alerting Is Required

Without alerting, operators may discover failures through:

- customer complaints
- dashboards
- log searches
- business teams
- failed deployment reports
- manual health checks

This creates delayed detection.

Production alerting provides:

1. Faster detection
2. Faster incident response
3. Clear ownership
4. Reduced mean time to detect (MTTD)
5. Reduced mean time to acknowledge (MTTA)
6. Reduced mean time to recovery (MTTR)
7. Better SLA/SLO protection
8. Automated escalation
9. Operational visibility
10. Evidence for incident analysis

---

# 5. Alerting Philosophy

A production alert should satisfy a simple test:

> **If nobody acts on this alert, can the system, customer experience, security posture, or business objective become materially worse?**

If the answer is no, consider making it a dashboard metric rather than an alert.

Good alerts should be:

- actionable
- specific
- owned
- prioritized
- deduplicated
- routed
- documented
- linked to a runbook
- stable
- measurable
- testable

---

# 6. Alert Lifecycle

A typical lifecycle is:

```text
Metric / Log / Event
        |
        v
Detection
        |
        v
Alert rule evaluation
        |
        v
Pending
        |
        v
Firing
        |
        v
Alertmanager
        |
        +--> Group
        +--> Deduplicate
        +--> Inhibit
        +--> Route
        |
        v
Notification
        |
        v
On-call acknowledgement
        |
        v
Investigation
        |
        v
Mitigation
        |
        v
Recovery
        |
        v
Resolution
        |
        v
Post-incident analysis
```

---

# 7. Alert States

Prometheus alert rules commonly move through:

```text
Inactive
   |
   | condition becomes true
   v
Pending
   |
   | condition remains true for "for" duration
   v
Firing
   |
   | condition becomes false
   v
Inactive
```

Example:

```yaml
for: 10m
```

means the expression must remain true for approximately ten minutes before the alert becomes firing.

This is extremely useful for reducing transient noise.

---

# 8. Pending vs Firing

Suppose:

```promql
node_memory_MemAvailable_bytes /
node_memory_MemTotal_bytes < 0.10
```

and:

```yaml
for: 10m
```

If memory availability drops below 10% for only two minutes:

```text
Pending -> recovers -> Inactive
```

No production page is required.

If it remains below 10% for ten minutes:

```text
Pending -> Firing
```

The alert is sent to Alertmanager.

---

# 9. Alert Sources

Production alerts can originate from multiple systems.

## 9.1 Metrics

Examples:

- CPU
- memory
- disk
- latency
- request rate
- error rate
- queue depth
- replica availability
- HTTP status codes

Prometheus is the primary metrics alerting system in this capstone.

---

## 9.2 Logs

ELK can detect patterns such as:

```text
OutOfMemoryError
authentication failure
database connection refused
payment gateway timeout
TLS certificate failure
```

Log-based detection can be useful for conditions that are not represented cleanly by metrics.

However, metric-based alerting should generally be preferred for high-volume, predictable operational signals.

---

## 9.3 Infrastructure

Examples:

- EC2 unavailable
- EKS node NotReady
- EBS capacity issues
- NAT Gateway problems
- ALB unhealthy targets
- network errors
- DNS failures

---

## 9.4 Kubernetes

Examples:

- pod crash loops
- OOMKilled
- deployment unavailable
- replica mismatch
- node NotReady
- pending pods
- failed jobs
- excessive restarts

---

## 9.5 Application

Examples:

- HTTP 5xx rate
- p95 latency
- checkout failures
- authentication failures
- inventory API errors
- payment errors

---

## 9.6 Security

Examples:

- suspicious authentication failures
- unexpected privilege changes
- anomalous API activity
- critical vulnerability detection
- unauthorized deployment
- suspicious Kubernetes events

Security alerts should be integrated with the organization's security response process.

---

# 10. Infrastructure Alert Categories

A production alerting taxonomy can be:

```text
Infrastructure
├── Compute
├── Memory
├── Disk
├── Network
├── Storage
├── Load balancer
├── DNS
└── Cloud service health

Kubernetes
├── Nodes
├── Pods
├── Deployments
├── StatefulSets
├── Jobs
├── Services
└── Ingress

Application
├── Availability
├── Error rate
├── Latency
├── Saturation
└── Business SLIs

Platform
├── Prometheus
├── Alertmanager
├── Argo CD
├── CI/CD
└── GitOps

Security
├── Authentication
├── Authorization
├── Vulnerabilities
└── Policy violations
```

---

# 11. Golden Signals

The four classic Golden Signals are:

1. Latency
2. Traffic
3. Errors
4. Saturation

---

## 11.1 Latency

Measures how long requests take.

Examples:

```text
p50 = 80ms
p95 = 240ms
p99 = 900ms
```

Alerting should normally focus on user-impacting percentiles rather than averages alone.

---

## 11.2 Traffic

Measures demand.

Examples:

```text
requests/sec
messages/sec
transactions/sec
```

A sudden traffic drop can also be an incident.

For example:

```text
Normal:
1000 req/s

Current:
80 req/s
```

The application may be healthy technically, but traffic may have stopped due to an ALB, DNS, deployment, or upstream issue.

---

## 11.3 Errors

Examples:

```text
HTTP 5xx
HTTP 4xx
application exceptions
database errors
```

A useful alert is usually a rate or ratio:

```promql
rate(http_requests_total{status=~"5.."}[5m])
/
rate(http_requests_total[5m])
```

---

## 11.4 Saturation

Shows how close a resource is to its usable limit.

Examples:

- CPU
- memory
- disk
- connection pools
- worker queues
- Kubernetes pod capacity
- database connections

---

# 12. SLI, SLO and SLA

## 12.1 SLI

Service Level Indicator.

Example:

```text
Successful requests / total requests
```

---

## 12.2 SLO

Service Level Objective.

Example:

```text
99.9% of checkout requests should succeed.
```

---

## 12.3 SLA

Service Level Agreement.

A contractual commitment to customers.

---

# 13. SLO-Based Alerting

A production alert should not only monitor infrastructure.

It should protect service objectives.

For example:

```text
Checkout SLO:
99.9% successful requests
```

If the service is consuming its error budget too quickly, alerting should escalate.

A useful model is:

```text
SLI
 |
 v
SLO
 |
 v
Error budget
 |
 v
Burn rate
 |
 v
Alert
```

---

# 14. Error Budget

For a 99.9% SLO:

```text
Allowed failure:
0.1%
```

For a 30-day period:

```text
30 days × 24 × 60
= 43,200 minutes
```

Allowed downtime is approximately:

```text
43,200 × 0.001
= 43.2 minutes
```

The exact calculation depends on the SLO measurement model.

---

# 15. Fast and Slow Burn Alerts

A robust SLO strategy often uses multiple windows.

Example concept:

```text
Fast burn:
very high error rate
short detection window

Slow burn:
moderate error rate
longer detection window
```

This catches:

- sudden outages
- sustained degradation

---

# 16. Prometheus Alerting Architecture

```text
Kubernetes / EKS
       |
       +----------------+
       |                |
       v                v
 Application        kube-state-metrics
 metrics                 |
       |                 |
       +--------+--------+
                |
                v
            Prometheus
                |
                v
         Prometheus Rules
                |
                v
           Alertmanager
          /      |       \
         /       |        \
       Email    Slack    PagerDuty
```

Grafana consumes Prometheus metrics for visualization.

ELK handles centralized logs.

---

# 17. Prometheus Responsibilities

Prometheus:

- scrapes metrics
- stores time series
- evaluates recording rules
- evaluates alert rules
- sends firing alerts to Alertmanager

Prometheus should not be treated as the complete notification system.

Alertmanager handles notification concerns.

---

# 18. Alertmanager Responsibilities

Alertmanager handles:

- grouping
- deduplication
- routing
- inhibition
- silencing
- receiver selection
- notification timing
- repeat notifications

---

# 19. Prometheus vs Alertmanager

| Prometheus | Alertmanager |
|---|---|
| Collects metrics | Handles alerts |
| Evaluates rules | Routes alerts |
| Stores time series | Groups alerts |
| Executes PromQL | Deduplicates notifications |
| Detects condition | Sends notifications |
| Sends alert event | Handles silences |

---

# 20. PrometheusRule

When using the Prometheus Operator, alert rules can be represented as a Kubernetes resource:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
```

This allows alert rules to be managed through Kubernetes and GitOps.

That fits the capstone architecture.

---

# 21. Production PrometheusRule Example

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-production-alerts
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
    app.kubernetes.io/part-of: roboshop
    environment: prod
    managed-by: argocd
spec:
  groups:

    - name: roboshop.infrastructure
      interval: 30s
      rules:

        - alert: NodeCPUHigh
          expr: |
            (
              100 *
              (1 - avg by (instance) (
                rate(node_cpu_seconds_total{mode="idle"}[5m])
              ))
            ) > 85
          for: 15m
          labels:
            severity: warning
            team: platform
            environment: prod
            category: infrastructure
          annotations:
            summary: "High CPU usage on Kubernetes node"
            description: |
              Node {{ $labels.instance }} has CPU utilization above 85%
              for more than 15 minutes.
            runbook_url: "https://runbooks.example.internal/kubernetes/node-cpu-high"

        - alert: NodeMemoryPressure
          expr: |
            (
              node_memory_MemAvailable_bytes
              /
              node_memory_MemTotal_bytes
            ) < 0.10
          for: 10m
          labels:
            severity: critical
            team: platform
            environment: prod
            category: infrastructure
          annotations:
            summary: "Kubernetes node memory critically low"
            description: |
              Node {{ $labels.instance }} has less than 10% available memory
              for more than 10 minutes.
            runbook_url: "https://runbooks.example.internal/kubernetes/node-memory"

        - alert: NodeFilesystemAlmostFull
          expr: |
            (
              node_filesystem_avail_bytes{
                fstype!~"tmpfs|overlay",
                mountpoint="/"
              }
              /
              node_filesystem_size_bytes{
                fstype!~"tmpfs|overlay",
                mountpoint="/"
              }
            ) < 0.10
          for: 15m
          labels:
            severity: critical
            team: platform
            environment: prod
            category: storage
          annotations:
            summary: "Node root filesystem almost full"
            description: |
              Node {{ $labels.instance }} has less than 10% filesystem
              capacity remaining on /.
            runbook_url: "https://runbooks.example.internal/kubernetes/node-disk"

    - name: roboshop.kubernetes
      interval: 30s
      rules:

        - alert: KubernetesNodeNotReady
          expr: |
            kube_node_status_condition{
              condition="Ready",
              status="true"
            } == 0
          for: 10m
          labels:
            severity: critical
            team: platform
            environment: prod
            category: kubernetes
          annotations:
            summary: "Kubernetes node is not ready"
            description: |
              Kubernetes node {{ $labels.node }} has not reported Ready
              for more than 10 minutes.
            runbook_url: "https://runbooks.example.internal/kubernetes/node-not-ready"

        - alert: PodCrashLooping
          expr: |
            increase(
              kube_pod_container_status_restarts_total[15m]
            ) > 5
          for: 10m
          labels:
            severity: warning
            team: application
            environment: prod
            category: kubernetes
          annotations:
            summary: "Pod is restarting repeatedly"
            description: |
              Pod {{ $labels.namespace }}/{{ $labels.pod }} has restarted
              more than five times within 15 minutes.
            runbook_url: "https://runbooks.example.internal/kubernetes/crashloop"

        - alert: ContainerOOMKilled
          expr: |
            kube_pod_container_status_last_terminated_reason{
              reason="OOMKilled"
            } == 1
          for: 5m
          labels:
            severity: critical
            team: application
            environment: prod
            category: kubernetes
          annotations:
            summary: "Container was OOMKilled"
            description: |
              Container {{ $labels.container }} in pod
              {{ $labels.namespace }}/{{ $labels.pod }} was terminated
              because of an out-of-memory condition.
            runbook_url: "https://runbooks.example.internal/kubernetes/oomkilled"

        - alert: DeploymentReplicasUnavailable
          expr: |
            kube_deployment_status_replicas_available
            <
            kube_deployment_spec_replicas
          for: 10m
          labels:
            severity: critical
            team: application
            environment: prod
            category: deployment
          annotations:
            summary: "Deployment has unavailable replicas"
            description: |
              Deployment {{ $labels.namespace }}/{{ $labels.deployment }}
              has fewer available replicas than desired.
            runbook_url: "https://runbooks.example.internal/kubernetes/deployment"

    - name: roboshop.application
      interval: 30s
      rules:

        - alert: ApplicationHighErrorRate
          expr: |
            (
              sum(rate(http_requests_total{
                environment="prod",
                status=~"5.."
              }[5m]))
              /
              sum(rate(http_requests_total{
                environment="prod"
              }[5m]))
            ) > 0.05
          for: 5m
          labels:
            severity: critical
            team: application
            environment: prod
            category: availability
          annotations:
            summary: "Application 5xx error rate is high"
            description: |
              Production application 5xx error rate is above 5% for
              more than 5 minutes.
            runbook_url: "https://runbooks.example.internal/application/error-rate"

        - alert: ApplicationHighLatency
          expr: |
            histogram_quantile(
              0.95,
              sum by (le) (
                rate(http_request_duration_seconds_bucket{
                  environment="prod"
                }[5m])
              )
            ) > 1
          for: 10m
          labels:
            severity: warning
            team: application
            environment: prod
            category: latency
          annotations:
            summary: "Application p95 latency is high"
            description: |
              Production p95 request latency is above one second
              for more than ten minutes.
            runbook_url: "https://runbooks.example.internal/application/latency"

    - name: roboshop.slo
      interval: 30s
      rules:

        - alert: CheckoutSLOErrorBudgetBurn
          expr: |
            (
              sum(rate(http_requests_total{
                service="checkout",
                status=~"5..",
                environment="prod"
              }[5m]))
              /
              sum(rate(http_requests_total{
                service="checkout",
                environment="prod"
              }[5m]))
            ) > 0.10
          for: 5m
          labels:
            severity: critical
            team: checkout
            environment: prod
            category: slo
          annotations:
            summary: "Checkout SLO is burning error budget rapidly"
            description: |
              Checkout is experiencing a sustained error ratio above 10%.
              Investigate immediately because the service error budget
              is being consumed rapidly.
            runbook_url: "https://runbooks.example.internal/slo/checkout"
```

---

# 22. Important PrometheusRule Fields

## apiVersion

```yaml
apiVersion: monitoring.coreos.com/v1
```

Uses the Prometheus Operator CRD.

---

## kind

```yaml
kind: PrometheusRule
```

Defines Prometheus recording/alerting rules.

---

## metadata.labels

Labels may be used by the Prometheus Operator's rule selector.

For example:

```yaml
release: kube-prometheus-stack
```

must match the relevant Prometheus configuration.

---

## groups

Rules are organized into groups.

Example:

```yaml
groups:
  - name: roboshop.application
```

This makes rule organization and ownership easier.

---

## interval

```yaml
interval: 30s
```

Controls how often rules in that group are evaluated.

Do not choose extremely aggressive intervals without a reason.

---

## alert

Defines the alert name.

Example:

```yaml
alert: ApplicationHighErrorRate
```

Use stable, descriptive names.

---

## expr

Contains PromQL.

---

## for

Controls how long a condition must remain true.

Example:

```yaml
for: 5m
```

---

## labels

Labels determine classification and routing.

Example:

```yaml
severity: critical
team: application
environment: prod
```

---

## annotations

Annotations contain human-readable context.

Typical annotations:

- summary
- description
- runbook_url
- dashboard_url
- query
- impact

---

# 23. PromQL Alerting Patterns

## 23.1 CPU

```promql
100 *
(
  1 -
  avg by (instance) (
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
> 85
```

---

## 23.2 Memory

```promql
(
  node_memory_MemAvailable_bytes
  /
  node_memory_MemTotal_bytes
) < 0.10
```

---

## 23.3 Disk

```promql
(
  node_filesystem_avail_bytes
  /
  node_filesystem_size_bytes
) < 0.10
```

---

## 23.4 Pod restarts

```promql
increase(
  kube_pod_container_status_restarts_total[15m]
) > 5
```

---

## 23.5 HTTP error ratio

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
> 0.05
```

---

## 23.6 p95 latency

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
) > 1
```

---

# 24. Avoiding Incorrect PromQL

A common production mistake is writing an alert without understanding the metric's labels and semantics.

Before deploying an alert:

```bash
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
```

Then test the expression in Prometheus.

Also inspect available metrics:

```promql
{__name__=~"http_.*"}
```

Do not assume a metric exists.

Different applications expose different metric names.

---

# 25. Alert Labels

Labels are machine-oriented metadata.

Example:

```yaml
labels:
  severity: critical
  team: payments
  environment: prod
  service: checkout
  category: availability
```

Labels should be consistent.

Avoid excessive high-cardinality labels.

---

# 26. Annotation Design

Annotations are human-oriented.

Example:

```yaml
annotations:
  summary: "Checkout error rate is high"
  description: |
    Checkout is returning more than 5% HTTP 5xx responses
    for five minutes.
  runbook_url: "https://runbooks.example.internal/checkout/error-rate"
```

The responder should be able to understand:

1. What failed?
2. Where?
3. How serious is it?
4. What should I do next?

---

# 27. Runbook URLs

Every page-worthy alert should ideally have a runbook.

Example:

```text
ApplicationHighErrorRate
        |
        v
Runbook
        |
        +--> Verify alert
        +--> Check Grafana
        +--> Check logs
        +--> Check recent deployments
        +--> Check dependencies
        +--> Mitigate
        +--> Escalate
```

A runbook should not merely say:

```text
Check logs.
```

It should provide concrete commands and decision points.

---

# 28. Alert Ownership

Each production alert should have an owner.

Example:

```yaml
team: platform
```

or:

```yaml
team: checkout
```

Possible ownership model:

```text
platform
├── EKS
├── nodes
├── Prometheus
├── Alertmanager
└── ingress infrastructure

application
├── checkout
├── cart
├── catalog
└── frontend

database
├── MongoDB
├── Redis
└── MySQL-compatible services
```

---

# 29. Severity Levels

A practical model:

## Critical

Immediate action required.

Examples:

- production service unavailable
- severe error rate
- multiple EKS nodes unavailable
- SLO breach
- security incident
- data loss risk

Usually page on-call.

---

## Warning

Investigation required, but immediate page may not be necessary.

Examples:

- CPU > 85%
- disk > 80%
- increased latency
- elevated restart count

Usually notify team channels.

---

## Info

Useful operational information.

Examples:

- deployment completed
- certificate nearing renewal
- planned maintenance event

Avoid using informational alerts for events that don't require attention.

---

# 30. Critical vs Warning Example

Bad:

```yaml
severity: critical
expr: cpu > 70%
```

Better:

```text
warning:
CPU > 85% for 15 minutes

critical:
CPU > 95% for 10 minutes
AND
application latency is degrading
```

The exact thresholds should be tuned using real workload data.

---

# 31. Alertmanager Architecture

```text
Prometheus
    |
    | alert events
    v
Alertmanager
    |
    +--> Group
    |
    +--> Deduplicate
    |
    +--> Inhibit
    |
    +--> Route
    |
    +--> Receiver
    |
    +--> Notification
```

---

# 32. Production Alertmanager Configuration

Example configuration:

```yaml
global:
  resolve_timeout: 5m

route:
  group_by:
    - alertname
    - cluster
    - environment
    - team

  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

  receiver: default

  routes:

    - matchers:
        - severity="critical"
        - environment="prod"
      receiver: prod-critical
      continue: false

    - matchers:
        - severity="warning"
        - environment="prod"
      receiver: prod-warning
      continue: false

    - matchers:
        - team="platform"
      receiver: platform-team
      continue: false

    - matchers:
        - team="application"
      receiver: application-team
      continue: false

    - matchers:
        - environment="qa"
      receiver: nonprod-team
      continue: false

receivers:

  - name: default
    email_configs:
      - to: "ops@example.internal"
        send_resolved: true

  - name: prod-critical
    email_configs:
      - to: "oncall@example.internal"
        send_resolved: true
    # Add PagerDuty-style integration through a secret-managed
    # integration key in the production deployment.

  - name: prod-warning
    email_configs:
      - to: "devops@example.internal"
        send_resolved: true

  - name: platform-team
    email_configs:
      - to: "platform@example.internal"
        send_resolved: true

  - name: application-team
    email_configs:
      - to: "application@example.internal"
        send_resolved: true

  - name: nonprod-team
    email_configs:
      - to: "devops@example.internal"
        send_resolved: false

inhibit_rules:

  - source_matchers:
      - severity="critical"
    target_matchers:
      - severity="warning"
    equal:
      - alertname
      - cluster
      - namespace

  - source_matchers:
      - alertname="KubernetesNodeNotReady"
    target_matchers:
      - category="pod"
    equal:
      - node
```

The email addresses are placeholders.

Real credentials and integration keys must be stored in Kubernetes Secrets or an external secret-management system.

---

# 33. Alertmanager Routing

Routing determines where alerts go.

A route can match:

```yaml
matchers:
  - severity="critical"
  - environment="prod"
```

Then send to:

```yaml
receiver: prod-critical
```

This separates notification behavior from alert detection logic.

---

# 34. Routing Strategy

A useful production hierarchy:

```text
root
 |
 +-- production
 |     |
 |     +-- critical -> PagerDuty/on-call
 |     +-- warning  -> team channel/email
 |
 +-- non-production
       |
       +-- warning -> engineering
       +-- critical -> engineering
```

Then optionally route by ownership:

```text
platform
application
database
security
```

---

# 35. Grouping

Without grouping, one failure can create dozens of notifications.

Example:

```text
NodeNotReady
PodCrashLoop
PodPending
DeploymentUnavailable
HighLatency
HighErrorRate
```

A single node failure may cause all of these.

Grouping can combine related alerts into one notification.

---

# 36. Grouping Fields

Example:

```yaml
group_by:
  - alertname
  - cluster
  - environment
  - team
```

Choose grouping labels carefully.

Too broad:

```yaml
group_by:
  - environment
```

could combine unrelated incidents.

Too narrow:

```yaml
group_by:
  - pod
  - container
  - instance
  - namespace
  - deployment
  - service
```

could create excessive notifications.

---

# 37. Deduplication

Alertmanager identifies repeated alert instances.

If the same alert is received repeatedly, it should not create a new notification every time.

Deduplication is essential during:

- alert storms
- Prometheus retries
- temporary network issues
- repeated evaluation cycles

---

# 38. Alert Storm Prevention

An alert storm can occur when:

```text
one root cause
   |
   +--> 100 pods
   +--> 20 deployments
   +--> 10 services
   +--> 5 nodes
   |
   v
thousands of alerts
```

Mitigation strategies:

1. Group alerts
2. Inhibit dependent alerts
3. Alert on symptoms rather than every consequence
4. Tune thresholds
5. Use `for`
6. Use recording rules
7. Establish ownership
8. Rate-limit notifications
9. Fix noisy rules

---

# 39. Inhibition

Inhibition suppresses lower-priority alerts when a higher-level alert is already firing.

Example:

```text
NodeNotReady = critical
        |
        v
Suppress related pod warning alerts
```

This helps responders focus on the root cause.

---

# 40. Inhibition Example

```yaml
inhibit_rules:

  - source_matchers:
      - alertname="KubernetesNodeNotReady"
        severity="critical"

    target_matchers:
      - severity="warning"

    equal:
      - cluster
      - node
```

This should be designed carefully.

Bad inhibition can hide important independent failures.

---

# 41. Silences

A silence tells Alertmanager:

> Do not notify me for alerts matching these conditions during this period.

Use cases:

- planned maintenance
- known migration
- approved load test
- controlled infrastructure change

Example:

```text
maintenance_window=eks-node-upgrade
```

Silences should have:

- owner
- reason
- start time
- expiration
- change/incident reference

Never create indefinite silences for recurring noise.

Fix the alert instead.

---

# 42. Alert Flapping

Flapping means an alert repeatedly changes:

```text
Firing
Inactive
Firing
Inactive
```

Causes:

- threshold too close to normal value
- unstable workload
- insufficient evaluation window
- intermittent network
- transient dependency failure

Mitigation:

```yaml
for: 10m
```

and better expressions.

---

# 43. Dead Man's Switch

A dead-man's-switch alert verifies that the monitoring pipeline itself is alive.

Concept:

```text
Prometheus evaluates a heartbeat alert
        |
        v
Alertmanager receives it
        |
        v
External notification path expects it
```

If the expected heartbeat disappears, investigate monitoring failure.

This is especially useful because:

> "No alerts" does not necessarily mean "No incidents."

It can mean:

```text
Prometheus failed
Alertmanager failed
network failed
notification integration failed
```

---

# 44. Watchdog Pattern

Many Prometheus deployments use a `Watchdog` alert.

Example:

```yaml
- alert: Watchdog
  expr: vector(1)
  labels:
    severity: none
  annotations:
    summary: "Monitoring heartbeat"
```

The notification receiver can verify that this expected signal continues to arrive.

---

# 45. Recording Rules

Recording rules precompute frequently used PromQL expressions.

Example:

```yaml
groups:
  - name: roboshop.recording
    interval: 30s
    rules:

      - record: roboshop:http_requests_per_second
        expr: |
          sum(
            rate(http_requests_total[5m])
          )

      - record: roboshop:http_5xx_ratio
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))
```

Benefits:

- simpler alert expressions
- faster queries
- consistent calculations
- lower dashboard/query cost

---

# 46. Alert Evaluation

Prometheus evaluates rules at configured intervals.

Example:

```yaml
interval: 30s
```

This means the rule group is evaluated approximately every 30 seconds.

If:

```yaml
for: 10m
```

the condition must persist across the required evaluation period before firing.

Do not confuse:

```text
evaluation interval
```

with:

```text
for duration
```

---

# 47. Alertmanager HA

For production, Alertmanager should be deployed redundantly.

Example:

```text
              Prometheus
                  |
          +-------+-------+
          |               |
          v               v
   Alertmanager-1   Alertmanager-2
          |               |
          +-------+-------+
                  |
          Notification systems
```

Alertmanager instances can form a cluster.

Benefits:

- reduced single point of failure
- continued alert processing
- better availability

Prometheus should be configured appropriately to send alerts to the Alertmanager cluster.

---

# 48. Prometheus HA Considerations

Prometheus itself may also be deployed redundantly.

Possible architecture:

```text
             EKS
              |
      +-------+-------+
      |               |
 Prometheus-A     Prometheus-B
      |               |
      +-------+-------+
              |
         Alertmanager
```

Running two Prometheus replicas does not automatically solve every long-term metrics availability problem.

Production environments may also use:

- remote write
- long-term storage
- managed monitoring
- object storage based systems

The capstone keeps the core alerting model centered on Prometheus + Alertmanager.

---

# 49. Kubernetes Alerting

Important Kubernetes alerts include:

```text
NodeNotReady
NodeMemoryPressure
NodeDiskPressure
PodCrashLooping
ContainerOOMKilled
PodPending
DeploymentUnavailable
ReplicaMismatch
DaemonSetUnavailable
StatefulSetUnavailable
JobFailed
```

---

# 50. Node Alerts

## CPU

Use a sustained threshold.

```promql
100 *
(
  1 -
  avg by(instance)(
    rate(node_cpu_seconds_total{mode="idle"}[5m])
  )
)
> 85
```

---

## Memory

```promql
node_memory_MemAvailable_bytes
/
node_memory_MemTotal_bytes
< 0.10
```

---

## Disk

```promql
node_filesystem_avail_bytes
/
node_filesystem_size_bytes
< 0.10
```

---

# 51. Kubernetes Node NotReady

Investigate:

```bash
kubectl get nodes
```

Then:

```bash
kubectl describe node <node-name>
```

Check:

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

AWS/EKS checks:

```bash
aws eks describe-nodegroup \
  --cluster-name roboshop-prod \
  --nodegroup-name application
```

Then inspect:

- EC2 status
- Auto Scaling Group
- security groups
- subnet capacity
- kubelet health
- node conditions

---

# 52. Pod CrashLoop Alert

Useful expression:

```promql
increase(
  kube_pod_container_status_restarts_total[15m]
) > 5
```

Investigation:

```bash
kubectl get pods -A
```

```bash
kubectl describe pod <pod> -n <namespace>
```

```bash
kubectl logs <pod> -n <namespace>
```

Previous container:

```bash
kubectl logs <pod> -n <namespace> --previous
```

Check deployment:

```bash
kubectl get deployment <deployment> -n <namespace>
```

---

# 53. OOMKilled Alert

Check:

```bash
kubectl get pod <pod> -n <namespace> \
  -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
```

Then:

```bash
kubectl describe pod <pod> -n <namespace>
```

Look for:

```text
Reason: OOMKilled
Exit Code: 137
```

Possible root causes:

- memory limit too low
- memory leak
- traffic spike
- inefficient application
- excessive concurrency

Do not blindly increase memory limits.

First determine why memory usage increased.

---

# 54. Deployment Alerts

Useful conditions:

```promql
kube_deployment_status_replicas_available
<
kube_deployment_spec_replicas
```

Investigate:

```bash
kubectl rollout status deployment/<name> -n <namespace>
```

```bash
kubectl describe deployment <name> -n <namespace>
```

```bash
kubectl get rs -n <namespace>
```

```bash
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

---

# 55. Pending Pods

A pod may remain pending because:

- insufficient CPU
- insufficient memory
- node selector mismatch
- taints
- affinity constraints
- PVC unavailable
- topology constraints

Investigate:

```bash
kubectl describe pod <pod> -n <namespace>
```

Look at scheduler events.

---

# 56. EKS Alerting

EKS is a managed control plane, but the production platform still requires monitoring.

Monitor:

- worker node health
- pod health
- Kubernetes API behavior
- cluster capacity
- ALB
- networking
- application metrics
- AWS service metrics
- IAM failures
- ECR/image pull failures

AWS-native signals can complement Prometheus.

---

# 57. Kubernetes API Server

API server problems can affect:

- deployments
- scheduling
- controllers
- Argo CD
- autoscaling
- kubectl

Production teams should monitor API server availability and latency using the metrics exposed by the EKS monitoring model available to the environment.

Investigate:

```bash
kubectl cluster-info
```

```bash
kubectl get --raw='/readyz?verbose'
```

For EKS:

```bash
aws eks describe-cluster \
  --name roboshop-prod
```

---

# 58. ALB Alerting

ALB-related signals include:

- unhealthy targets
- target response time
- 4xx
- 5xx
- rejected connections
- request count anomalies

AWS CloudWatch metrics are useful here.

Typical investigation:

```text
ALB
 |
 +--> Target health
 |
 +--> Listener
 |
 +--> Target group
 |
 +--> Security groups
 |
 +--> Kubernetes Service
 |
 +--> Pods
```

---

# 59. ALB + Kubernetes Investigation

Check ingress:

```bash
kubectl get ingress -A
```

Describe:

```bash
kubectl describe ingress <name> -n <namespace>
```

Check services:

```bash
kubectl get svc -n <namespace>
```

Check endpoints:

```bash
kubectl get endpoints -n <namespace>
```

Check pods:

```bash
kubectl get pods -o wide -n <namespace>
```

The ALB can be healthy while pods are unhealthy, or vice versa.

---

# 60. Application Availability Alerts

Availability should be measured from the application's perspective.

Example:

```text
Successful requests / total requests
```

A service may have:

```text
pods = Running
CPU = normal
memory = normal
```

but:

```text
HTTP 500 = 15%
```

This is still an availability incident.

---

# 61. Application Error Rate

Example:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

Alert threshold:

```text
> 5%
```

for:

```text
5 minutes
```

Thresholds should be tuned using real traffic and SLO requirements.

---

# 62. Application Latency

Use histogram metrics when available.

Example:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket[5m])
  )
)
```

Alert:

```text
p95 > 1 second for 10 minutes
```

For customer-facing services, latency should be evaluated against the service's actual SLO.

---

# 63. Database-Related Alerts

Where databases are relevant, monitor:

- connection count
- connection failures
- query latency
- replication lag
- storage capacity
- CPU
- memory
- cache hit ratio
- lock contention
- error rate

The exact metrics depend on the database.

Example conceptual alert:

```text
database connection failures > threshold
```

A database alert should normally route to the database/platform owner.

---

# 64. Redis-Related Alerts

Potential signals:

- memory utilization
- evictions
- connected clients
- latency
- replication health
- command errors

A Redis alert should distinguish:

```text
cache degradation
```

from:

```text
critical dependency failure
```

---

# 65. CI/CD Alerting

CI/CD is part of production reliability.

Alert on:

- failed production deployment
- repeated pipeline failures
- security scan failure
- image push failure
- GitOps update failure
- Argo CD sync failure

Example workflow:

```text
Developer
   |
   v
CI
   |
   +--> Build
   +--> Test
   +--> Security scans
   |
   v
ECR
   |
   v
GitOps update
   |
   v
Argo CD
   |
   v
EKS
```

---

# 66. Argo CD Alerts

Important conditions:

```text
Application OutOfSync
Application Degraded
Application SyncFailed
Application Health Unknown
```

A production deployment should be observable from:

```text
Git commit
   |
   v
Argo CD
   |
   v
Sync
   |
   v
Kubernetes
   |
   v
Application health
```

---

# 67. GitOps Drift Alert

Drift means:

```text
Git desired state
        !=
Kubernetes actual state
```

In a GitOps environment this is important.

Example:

```text
Argo CD:
OutOfSync
```

Potential causes:

- manual kubectl change
- failed reconciliation
- invalid manifest
- controller issue
- admission rejection

---

# 68. Security Alerts

Production security alerting may include:

```text
critical vulnerability detected
unauthorized IAM activity
unexpected privileged Kubernetes operation
repeated authentication failures
suspicious API activity
image vulnerability introduced
policy violation
```

Security alerts should route to the security team when required.

---

# 69. Alert Routing by Environment

Use environment labels:

```yaml
environment: prod
```

```yaml
environment: qa
```

```yaml
environment: dev
```

Production should have the strongest notification policy.

Example:

```text
prod critical -> on-call
prod warning  -> team
qa critical   -> engineering
dev warning   -> dashboard/channel
```

---

# 70. Team-Based Routing

Example:

```yaml
team: platform
```

```yaml
team: checkout
```

```yaml
team: security
```

Routing:

```text
team=platform
       |
       v
Platform on-call

team=checkout
       |
       v
Checkout on-call

team=security
       |
       v
Security response
```

---

# 71. Severity + Environment Routing

A practical route matcher:

```yaml
- matchers:
    - severity="critical"
    - environment="prod"
  receiver: prod-critical
```

This is stronger than routing solely by severity.

A critical alert in development should not necessarily wake the production on-call engineer.

---

# 72. Email Notifications

Email works well for:

- warning alerts
- reports
- non-urgent operational alerts

It may be insufficient for:

- immediate production outage
- high-severity customer impact
- security incidents

For those, use an on-call/incident escalation system.

---

# 73. Slack-Style Integrations

A team-chat integration is useful for:

- warning alerts
- incident collaboration
- deployment notifications
- operational awareness

Typical flow:

```text
Alertmanager
     |
     v
Chat webhook
     |
     v
#production-alerts
```

Do not expose webhook credentials directly in Git.

Use a Kubernetes Secret or external secret-management mechanism.

---

# 74. PagerDuty-Style Escalation

For critical incidents:

```text
Alertmanager
     |
     v
Incident platform
     |
     v
Primary on-call
     |
     | no acknowledgement
     v
Secondary on-call
     |
     | no acknowledgement
     v
Incident commander / escalation
```

The actual service integration should be configured using the organization's credentials and incident-management platform.

---

# 75. On-Call Model

A production on-call process should define:

```text
Primary
Secondary
Escalation manager
Service owner
Platform owner
Security escalation
```

The alert should tell the responder:

```text
What happened?
Where?
Impact?
Owner?
Runbook?
Dashboard?
Recent deployment?
```

---

# 76. Alert Notification Content

A useful notification:

```text
CRITICAL: CheckoutHighErrorRate

Environment: prod
Cluster: roboshop-prod
Service: checkout
Team: checkout
Impact: Checkout requests are failing

Current error ratio: 12.4%
Threshold: 5%
Duration: 8m

Runbook:
<runbook>

Dashboard:
<dashboard>

Recent deployment:
<deployment information>
```

---

# 77. Alert Deduplication Strategy

Deduplicate using stable identity labels.

Avoid labels that change every second.

Bad:

```yaml
request_id: ...
timestamp: ...
```

These produce separate alert identities.

Good:

```yaml
cluster
namespace
deployment
service
severity
```

---

# 78. Cardinality Considerations

Do not put high-cardinality dimensions into alert labels unnecessarily.

Examples of dangerous labels:

```text
request_id
user_id
transaction_id
session_id
full URL
```

Prometheus is a time-series system and high cardinality can become expensive.

---

# 79. Alert Noise

Common causes:

- thresholds too low
- no `for`
- duplicate rules
- alerts without ownership
- poor grouping
- alerting on symptoms and causes independently
- alerts for expected behavior
- alerts on development environments
- unstable metrics

---

# 80. Alert Quality Review

For every alert ask:

```text
Is it actionable?
Who owns it?
What does it mean?
What is the impact?
How urgent is it?
Does it page?
Is there a runbook?
Is there a dashboard?
Can it flap?
Can it storm?
Can it be inhibited?
```

---

# 81. Production Alert Naming

Use consistent names.

Examples:

```text
KubernetesNodeNotReady
KubernetesNodeMemoryPressure
ContainerOOMKilled
PodCrashLooping
DeploymentReplicasUnavailable
ApplicationHighErrorRate
ApplicationHighLatency
CheckoutSLOErrorBudgetBurn
ALBUnhealthyTargets
ArgoCDApplicationDegraded
```

Avoid vague names such as:

```text
ProblemDetected
ServerBad
SomethingWrong
```

---

# 82. Production Alert Metadata Standard

Recommended labels:

```yaml
labels:
  severity: critical
  team: checkout
  environment: prod
  cluster: roboshop-prod
  service: checkout
  category: availability
```

Recommended annotations:

```yaml
annotations:
  summary: "Checkout error rate is high"
  description: "..."
  runbook_url: "..."
  dashboard_url: "..."
```

---

# 83. Alertmanager Route Example — More Granular

```yaml
route:
  receiver: default

  group_by:
    - alertname
    - cluster
    - environment
    - team

  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h

  routes:

    - matchers:
        - environment="prod"
        - severity="critical"
        - team="security"
      receiver: security-critical

    - matchers:
        - environment="prod"
        - severity="critical"
        - team="platform"
      receiver: platform-critical

    - matchers:
        - environment="prod"
        - severity="critical"
      receiver: production-critical

    - matchers:
        - environment="prod"
        - severity="warning"
      receiver: production-warning

    - matchers:
        - environment=~"dev|qa"
      receiver: nonproduction
```

Order and route behavior should be tested carefully, especially when using nested routes and `continue`.

---

# 84. Inhibition Design

Example:

```yaml
inhibit_rules:

  - source_matchers:
      - alertname="KubernetesNodeNotReady"
        severity="critical"

    target_matchers:
      - severity="warning"

    equal:
      - cluster
      - node

  - source_matchers:
      - alertname="ApplicationHighErrorRate"
        severity="critical"

    target_matchers:
      - severity="warning"

    equal:
      - cluster
      - service
```

The purpose is to reduce duplicate noise, not hide incidents.

---

# 85. Testing Alert Rules

Before deploying:

```bash
promtool check rules alerts.yaml
```

For YAML/config validation:

```bash
promtool check config prometheus.yml
```

For Alertmanager configuration:

```bash
amtool check-config alertmanager.yml
```

Exact command availability depends on installed versions.

---

# 86. Kubernetes Manifest Validation

Validate manifests before Git commit:

```bash
kubectl apply --dry-run=server -f prometheus-rules.yaml
```

For a GitOps workflow, validate generated manifests as part of CI.

---

# 87. Testing a PrometheusRule

Check CRD:

```bash
kubectl get prometheusrules -n monitoring
```

Describe:

```bash
kubectl describe prometheusrule roboshop-production-alerts -n monitoring
```

Check Prometheus targets/rules through the Prometheus UI.

---

# 88. Testing Alertmanager

Check:

```bash
kubectl get pods -n monitoring
```

Logs:

```bash
kubectl logs -n monitoring \
  <alertmanager-pod>
```

Check Alertmanager UI:

```text
Alerts
Silences
Status
Configuration
```

---

# 89. Safe Alert Test

Do not intentionally break production just to test an alert.

Use one of:

- staging
- controlled synthetic alert
- temporary test rule
- test receiver
- maintenance window

Example temporary rule:

```yaml
- alert: AlertingPipelineTest
  expr: vector(1)
  for: 1m
  labels:
    severity: warning
    team: platform
    environment: qa
  annotations:
    summary: "Alerting pipeline test"
    description: "Synthetic alert used to validate notification routing."
```

---

# 90. Production Test Process

```text
Create test rule
       |
       v
Git commit
       |
       v
CI validation
       |
       v
GitOps repository
       |
       v
Argo CD
       |
       v
Prometheus
       |
       v
Alertmanager
       |
       v
Test receiver
       |
       v
Confirm notification
       |
       v
Remove test rule
```

This validates the complete path.

---

# 91. Alert Troubleshooting Decision Tree

If an alert does not arrive:

```text
Is Prometheus evaluating the rule?
       |
      No
       |
       v
Check rule loading
       |
      Yes
       |
       v
Is alert firing?
       |
      No
       |
       v
Check PromQL
       |
      Yes
       |
       v
Did Alertmanager receive it?
       |
      No
       |
       v
Check Prometheus -> Alertmanager connectivity
       |
      Yes
       |
       v
Was it routed?
       |
      No
       |
       v
Check matchers/routes
       |
      Yes
       |
       v
Was notification delivered?
       |
      No
       |
       v
Check receiver/webhook/email
```

---

# 92. Prometheus Rule Not Loading

Check:

```bash
kubectl get prometheusrules -n monitoring
```

Check Prometheus operator:

```bash
kubectl get pods -n monitoring
```

Logs:

```bash
kubectl logs -n monitoring \
  deployment/prometheus-operator
```

Check labels.

A very common problem is:

```text
PrometheusRule exists
BUT
Prometheus selector does not select it.
```

---

# 93. Alert Rule Exists but Does Not Fire

Test PromQL directly.

Possible causes:

- metric does not exist
- wrong label
- threshold too high
- wrong namespace
- metric name differs
- `for` duration has not elapsed
- data scrape is missing

Example:

```promql
up
```

If:

```text
up = 0
```

the target is down.

---

# 94. Alert Fires but No Notification

Check:

```text
Prometheus
   |
   v
Alertmanager
```

Then inspect Alertmanager:

```text
Alert received?
Routing matched?
Inhibition active?
Silence active?
Receiver valid?
Notification delivery successful?
```

---

# 95. Alertmanager Notification Failure

Possible causes:

- invalid SMTP settings
- invalid webhook
- expired integration
- DNS issue
- network policy
- egress restriction
- TLS failure
- credentials failure

Check logs:

```bash
kubectl logs -n monitoring \
  <alertmanager-pod>
```

---

# 96. Alert Is Too Noisy

Ask:

```text
Is the threshold too sensitive?
```

Then:

```text
Add for duration
```

or:

```text
Use rate/ratio
```

or:

```text
Use SLO
```

or:

```text
Group alerts
```

or:

```text
Inhibit dependent alerts
```

Do not simply silence the alert permanently.

---

# 97. Alert Flapping Troubleshooting

Check metric:

```promql
<expression>
```

over a longer period.

If it repeatedly crosses:

```text
threshold
```

consider:

```text
hysteresis
longer for
different threshold
moving average
rate calculation
```

Prometheus alert rules do not provide classic hysteresis directly in the same way as some monitoring systems, so expression design and duration are important.

---

# 98. Alert Storm Troubleshooting

When hundreds of alerts fire:

1. Identify earliest alert.
2. Find common cluster/node/service labels.
3. Identify root cause.
4. Check inhibition.
5. Check grouping.
6. Temporarily silence only when necessary.
7. Fix root cause.
8. Improve alert rules afterward.

Do not start acknowledging every alert independently without identifying the common cause.

---

# 99. Example Incident — Checkout Error Rate

Alert:

```text
ApplicationHighErrorRate
```

Notification:

```text
CRITICAL
checkout
prod
5xx = 18%
```

Responder actions:

```bash
kubectl get pods -n roboshop
```

```bash
kubectl get deployment checkout -n roboshop
```

```bash
kubectl logs -n roboshop \
  deployment/checkout \
  --tail=200
```

Check recent deployment:

```bash
kubectl rollout history deployment/checkout -n roboshop
```

Check Argo CD application.

Then inspect dependencies:

```text
checkout
  |
  +--> cart
  +--> payment
  +--> database
```

---

# 100. Example Root Cause — Bad Deployment

Timeline:

```text
10:00 deployment starts
10:03 new pods ready
10:05 error rate rises
10:06 alert fires
10:07 on-call investigates
10:10 rollback
10:12 errors return to baseline
```

Alerting succeeded because:

- metric detected customer impact
- severity was critical
- team ownership was clear
- runbook was available
- rollback could be performed

---

# 101. Example Incident — Node Failure

Alert:

```text
KubernetesNodeNotReady
```

Then:

```text
Node unavailable
   |
   +--> pods rescheduled
   |
   +--> capacity decreases
   |
   +--> latency may increase
```

Responder checks:

```bash
kubectl get nodes
kubectl describe node <node>
kubectl get pods -A -o wide
```

AWS:

```bash
aws ec2 describe-instances ...
```

EKS:

```bash
aws eks describe-nodegroup ...
```

Check whether the Auto Scaling Group replaces the node.

---

# 102. Example Incident — OOMKilled

Alert:

```text
ContainerOOMKilled
```

Investigation:

```bash
kubectl describe pod <pod> -n roboshop
```

Check:

```text
requests
limits
memory usage
recent release
traffic
```

Then inspect application behavior.

Possible immediate mitigation:

- rollback release
- scale replicas
- increase memory limit if justified

Long-term:

- identify leak
- optimize memory
- load test
- tune resource requests/limits

---

# 103. Example Incident — ALB Unhealthy Targets

Alert:

```text
ALB target health degradation
```

Flow:

```text
Internet
   |
   v
ALB
   |
   v
Target Group
   |
   v
Kubernetes Service
   |
   v
Pod
```

Check:

```bash
kubectl get ingress -A
kubectl get svc -A
kubectl get endpoints -A
kubectl get pods -A
```

Possible causes:

- wrong target port
- readiness probe failure
- security group
- application not listening
- service selector mismatch
- target registration issue

---

# 104. Example Incident — Argo CD OutOfSync

Alert:

```text
ArgoCDApplicationOutOfSync
```

Check:

```text
Argo CD application
Git commit
manifest differences
sync status
health status
```

If someone changed Kubernetes manually:

```text
Git = desired state
Cluster = changed state
```

Argo CD may restore the desired state.

The correct response depends on whether the manual change was authorized.

---

# 105. Alerting for Deployments

Production deployment alerting should cover:

```text
Deployment started
Deployment failed
Deployment degraded
Deployment succeeded
Rollback triggered
Argo CD sync failed
Argo CD health degraded
```

Not every event needs paging.

For example:

```text
successful deployment -> information
failed production deployment -> warning/critical depending on impact
customer-facing degradation -> critical
```

---

# 106. Alerting and Rollbacks

Alerting should detect the consequences of bad deployments.

Example:

```text
New image
   |
   v
EKS
   |
   v
Error rate increases
   |
   v
Alert
   |
   v
On-call
   |
   v
Rollback
```

This connects alerting with the later `29-Rollback-and-Recovery.md` chapter.

---

# 107. Alerting and GitOps

The desired state lives in Git.

Therefore alerting should help detect:

```text
Git commit
   |
   v
Argo CD sync
   |
   v
Kubernetes deployment
   |
   v
Application health
```

A deployment is not complete simply because Argo CD says:

```text
Synced
```

The application should also be:

```text
Healthy
```

---

# 108. Alerting and ELK

Prometheus handles metrics.

ELK handles logs.

Example:

```text
Prometheus:
error rate = 15%

Alertmanager:
critical alert

ELK:
database connection refused
```

The metric alert tells the responder:

> Something is wrong.

The logs help answer:

> Why?

---

# 109. Alert Investigation Using Grafana + ELK

A responder can follow:

```text
Alert
 |
 v
Grafana dashboard
 |
 +--> error rate
 +--> latency
 +--> CPU
 +--> memory
 |
 v
ELK
 |
 +--> application errors
 +--> stack traces
 +--> dependency failures
 |
 v
Kubernetes
 |
 +--> pods
 +--> deployment
 +--> events
```

---

# 110. Production Alerting Architecture — Full

```text
                         Internet
                            |
                            v
                       AWS ALB
                            |
                            v
                    Kubernetes / EKS
                            |
          +-----------------+------------------+
          |                 |                  |
          v                 v                  v
     Applications       Kubernetes         Platform
          |              Metrics              |
          |                 |                  |
          |                 v                  |
          |             Prometheus <-----------+
          |                 |
          |          +------+------+
          |          |             |
          |          v             v
          |      Alert Rules   Recording Rules
          |          |
          |          v
          |      Alertmanager
          |          |
          |    +-----+------+----------------+
          |    |            |                |
          |    v            v                v
          |  Email        Chat         Incident Platform
          |                               |
          |                               v
          |                            On-call
          |
          +------ Logs ------> ELK

Grafana
   |
   +--> Prometheus dashboards
   +--> Alert investigation
   +--> SLO dashboards
```

---

# 111. RoboShop Production Alert Flow

```text
Developer
    |
    v
Git
    |
    v
CI Pipeline
    |
    +--> Maven / Node / Python build
    +--> Tests
    +--> SonarQube
    +--> Trivy
    +--> Veracode
    +--> Container image
    |
    v
ECR
    |
    v
GitOps Repository
    |
    v
Argo CD
    |
    v
EKS
    |
    +--> Frontend
    +--> Cart
    +--> Catalog
    +--> User
    +--> Payment
    +--> Shipping
    +--> Checkout
    +--> Databases / dependencies
    |
    +--> ALB
    |
    +--> Prometheus
    +--> Grafana
    +--> ELK
    |
    v
Alerting
    |
    +--> Platform
    +--> Application
    +--> Security
    +--> On-call
```

---

# 112. Real Production Incident Walkthrough

## Scenario

A new checkout release is deployed.

Five minutes later:

```text
HTTP 5xx = 14%
```

Prometheus evaluates:

```promql
5xx ratio > 5%
```

The condition remains true for five minutes.

Alert changes:

```text
Pending
   |
   v
Firing
```

Prometheus sends the alert to Alertmanager.

Alertmanager:

```text
environment=prod
severity=critical
team=checkout
```

matches:

```text
prod-critical
```

The on-call engineer receives a notification.

---

# 113. Incident Response Actions

Responder:

```bash
kubectl get pods -n roboshop
```

Finds:

```text
checkout pods restarted
```

Then:

```bash
kubectl logs deployment/checkout \
  -n roboshop \
  --tail=200
```

Finds:

```text
connection refused to payment
```

Checks:

```bash
kubectl get pods -n roboshop -l app=payment
```

Payment service is degraded.

The responder checks the payment deployment and recent GitOps change.

---

# 114. Root Cause

Suppose the payment service was deployed with an incorrect environment variable.

The failure chain becomes:

```text
Bad Git commit
     |
     v
CI passed
     |
     v
GitOps updated
     |
     v
Argo CD synced
     |
     v
Payment degraded
     |
     v
Checkout errors
     |
     v
Prometheus alert
     |
     v
Alertmanager
     |
     v
On-call
```

This demonstrates why alerting must be integrated with the complete DevOps lifecycle.

---

# 115. Alert Response Workflow

```text
1. Receive alert
       |
2. Acknowledge
       |
3. Validate
       |
4. Determine customer impact
       |
5. Identify blast radius
       |
6. Check recent changes
       |
7. Check dependencies
       |
8. Mitigate
       |
9. Confirm recovery
       |
10. Resolve alert
       |
11. Document incident
       |
12. Improve monitoring
```

---

# 116. Alert Acknowledgement vs Resolution

Acknowledgement means:

> Someone is working on it.

Resolution means:

> The underlying condition has recovered or the incident is otherwise operationally resolved.

Do not resolve an alert merely because someone acknowledged it.

---

# 117. Alert Escalation

Example:

```text
T+0
Primary on-call paged

T+5
No acknowledgement -> secondary

T+15
No resolution -> service owner

T+30
Major incident -> incident commander
```

Actual timings depend on the organization's policy.

---

# 118. Major Incident Alerts

Critical customer-impacting alerts may trigger:

```text
incident creation
on-call paging
incident channel
incident commander
communications
status-page process
executive escalation
```

Alerting itself does not replace incident management.

It starts the response.

---

# 119. SLO Alert Example

Suppose checkout has:

```text
SLO = 99.9%
```

The service experiences:

```text
5xx ratio = 0.5%
```

This is technically above the permitted error rate.

An SLO-oriented alert can identify that error budget is being consumed.

A more mature system evaluates:

```text
current error rate
+
historical window
+
remaining error budget
+
burn rate
```

---

# 120. Why CPU-Only Alerting Is Not Enough

Imagine:

```text
CPU = 30%
Memory = 40%
Disk = 50%
```

Everything looks healthy.

But:

```text
HTTP 5xx = 20%
```

The application is broken.

Conversely:

```text
CPU = 95%
```

does not automatically mean customers are impacted if the service is designed for high CPU utilization.

Therefore production alerting should prioritize:

```text
customer impact
service objectives
system saturation
```

rather than arbitrary infrastructure thresholds.

---

# 121. Availability vs Resource Alerting

Recommended priority:

```text
Customer impact
    >
Service health
    >
Capacity
    >
Infrastructure utilization
```

This does not mean resource alerts are unimportant.

It means they should be interpreted in operational context.

---

# 122. Capacity Alerts

Capacity alerts should detect future risk.

Examples:

```text
disk < 15%
node allocatable CPU < threshold
memory headroom low
pod capacity exhausted
EBS volume nearing limit
database storage nearing limit
```

Capacity alerts should usually fire before an outage.

---

# 123. Capacity Planning

If nodes consistently operate at:

```text
CPU 85%
memory 90%
```

the correct action may be:

- scale nodes
- tune requests
- improve autoscaling
- optimize workloads

Do not wait until:

```text
pods Pending
```

to discover capacity problems.

---

# 124. Alerting with HPA

HPA may scale based on CPU/memory/custom metrics.

Alert on:

```text
HPA at max replicas
```

because this may indicate the application has reached its scaling ceiling.

Concept:

```text
Traffic increases
     |
     v
HPA scales
     |
     v
maxReplicas reached
     |
     v
load continues
     |
     v
latency increases
```

The important alert is not merely:

```text
CPU high
```

but:

```text
application cannot scale further
```

---

# 125. Alerting with Cluster Autoscaler / Node Scaling

A production platform should monitor:

```text
Pending pods
Node capacity
Autoscaler activity
Failed scale-up
```

A pod remaining Pending for a sustained duration may indicate cluster capacity or scheduling constraints.

---

# 126. Alerting for Jobs

Production jobs should be monitored for:

```text
failed jobs
stuck jobs
missed schedules
excessive duration
```

For example:

```promql
kube_job_status_failed > 0
```

The exact expression should account for job lifecycle and expected retries.

---

# 127. Alerting for Certificates

Certificate expiry can cause complete outages.

Monitor:

```text
certificate expiration
```

Example policy:

```text
warning:
< 30 days

critical:
< 7 days
```

The exact thresholds depend on the certificate management process.

---

# 128. Alerting for DNS

DNS failure can make a completely healthy application unreachable.

Potential checks:

```text
DNS resolution failure
latency
record correctness
external endpoint availability
```

Synthetic monitoring can complement Prometheus.

---

# 129. Synthetic Monitoring

Synthetic monitoring tests the service from the outside.

Example:

```text
GET https://shop.example.com
```

Then validate:

```text
DNS
TLS
HTTP
latency
expected content
```

This is useful because internal Kubernetes metrics can look healthy while the external user path is broken.

---

# 130. Blackbox-Style Alerting

A blackbox exporter-style architecture can be:

```text
Prometheus
    |
    v
Blackbox probe
    |
    v
Internet / ALB
    |
    v
Application
```

Useful for:

- HTTP availability
- TLS
- DNS
- TCP
- ICMP where appropriate

This complements internal metrics.

---

# 131. External vs Internal Monitoring

Internal:

```text
Pod metrics
Node metrics
Application metrics
```

External:

```text
DNS
ALB
TLS
HTTP
```

Both are valuable.

---

# 132. Alerting Dependency Hierarchy

Consider:

```text
Node
 |
 v
Pod
 |
 v
Service
 |
 v
ALB
 |
 v
Customer
```

A lower-level failure can create multiple symptoms.

Alert design should attempt to identify the highest useful root-cause signal.

---

# 133. Root Cause vs Symptom Alerts

Example:

```text
NodeNotReady
PodPending
ApplicationLatencyHigh
ApplicationErrorRateHigh
```

If all originate from a node failure, NodeNotReady may be the root cause.

However, do not inhibit customer-impact alerts too aggressively.

The goal is:

```text
less noise
without
hiding impact
```

---

# 134. Alert Runbook Structure

A runbook should contain:

```text
Title
Purpose
Severity
Owner
Symptoms
Impact
Initial checks
Investigation
Common causes
Commands
Mitigation
Rollback
Escalation
Recovery validation
Post-incident actions
```

---

# 135. Example Runbook — Pod CrashLoop

```text
1. Confirm alert.

2. Find pod:
kubectl get pods -A

3. Describe:
kubectl describe pod <pod> -n <namespace>

4. Logs:
kubectl logs <pod> -n <namespace>

5. Previous logs:
kubectl logs <pod> -n <namespace> --previous

6. Check events:
kubectl get events -n <namespace>

7. Check deployment:
kubectl describe deployment <deployment> -n <namespace>

8. Check recent deployment:
Argo CD / Git history

9. Mitigate:
rollback or fix dependency

10. Verify:
pod stable
error rate normal
latency normal
```

---

# 136. Alerting Security

Protect:

- Prometheus
- Alertmanager
- Grafana
- Kubernetes APIs

Use:

- authentication
- RBAC
- network restrictions
- TLS where appropriate
- secret management
- least privilege

Do not expose Alertmanager publicly without appropriate controls.

---

# 137. Alertmanager Secrets

Never commit:

```yaml
api_url: https://hooks.example.com/secret-token
```

or:

```yaml
api_key: abc123
```

to Git.

Instead:

```text
Kubernetes Secret
       |
       v
Alertmanager deployment
```

or:

```text
External Secrets
       |
       v
Kubernetes Secret
       |
       v
Alertmanager
```

---

# 138. GitOps and Secrets

The GitOps repository can contain:

```text
alertmanager-config.yaml
```

but sensitive values should be referenced indirectly.

Example conceptual structure:

```yaml
api_url_file: /etc/alertmanager/secrets/chat-webhook/url
```

The secret is mounted separately.

---

# 139. Alerting HA Checklist

Production:

- Prometheus redundancy where required
- Alertmanager HA
- durable configuration
- tested notification integrations
- monitoring of monitoring
- Watchdog/dead-man's-switch
- backup of configuration
- GitOps management
- documented runbooks

---

# 140. Alerting DR

Alerting configuration should be recoverable from Git.

Store:

```text
PrometheusRule
Alertmanager configuration
Grafana dashboards
recording rules
routing configuration
runbooks
```

in the appropriate repositories.

If the cluster is rebuilt:

```text
Terraform
   |
   v
EKS
   |
   v
Argo CD
   |
   v
Monitoring stack
   |
   v
Alerting restored
```

---

# 141. Production Alert Configuration Repository

A possible structure:

```text
monitoring/
├── prometheus/
│   ├── rules/
│   │   ├── infrastructure.yaml
│   │   ├── kubernetes.yaml
│   │   ├── applications.yaml
│   │   └── slo.yaml
│   └── recording-rules.yaml
│
├── alertmanager/
│   ├── config.yaml
│   └── routing.yaml
│
├── grafana/
│   └── dashboards/
│
└── runbooks/
    ├── node-not-ready.md
    ├── pod-crashloop.md
    ├── application-errors.md
    └── slo-burn.md
```

---

# 142. CI Validation for Alert Rules

The CI pipeline should validate:

```text
YAML syntax
Prometheus rule syntax
Alertmanager syntax
Kubernetes schema
policy
security
```

Example:

```bash
yamllint .
promtool check rules monitoring/prometheus/rules/*.yaml
amtool check-config monitoring/alertmanager/config.yaml
kubectl apply --dry-run=server ...
```

The exact pipeline depends on repository architecture.

---

# 143. Code Review for Alerts

Every alert change should be reviewed for:

- correctness
- threshold
- severity
- ownership
- runbook
- routing
- cardinality
- false positives
- false negatives
- production impact

An alert rule is production code.

---

# 144. Alert Tuning Process

Start with:

```text
reasonable threshold
```

Then observe:

```text
alert frequency
false positives
missed incidents
operator feedback
```

Tune:

```text
threshold
for duration
PromQL
severity
routing
grouping
```

Repeat.

---

# 145. Alert Review Metrics

Track:

```text
alerts per day
pages per week
false-positive rate
acknowledgement time
resolution time
repeat alerts
noisy alerts
silence count
```

A very high alert volume is often a monitoring quality problem.

---

# 146. MTTD

Mean Time To Detect.

Example:

```text
Failure:
10:00

Alert:
10:03

MTTD:
3 minutes
```

---

# 147. MTTA

Mean Time To Acknowledge.

```text
Alert:
10:03

Engineer acknowledges:
10:05

MTTA:
2 minutes
```

---

# 148. MTTR

Mean Time To Recovery.

```text
Failure:
10:00

Recovered:
10:15

MTTR:
15 minutes
```

Alerting directly influences MTTD and indirectly influences MTTR.

---

# 149. Alerting Anti-Patterns

Avoid:

```text
CPU > 50%
```

without context.

Avoid:

```text
every pod restart -> page
```

if normal.

Avoid:

```text
same alert every minute
```

Avoid:

```text
critical alerts without owner
```

Avoid:

```text
alerts without runbooks
```

Avoid:

```text
permanent silences
```

Avoid:

```text
credentials in Git
```

Avoid:

```text
thousands of alerts during one incident
```

---

# 150. Production Alerting Best Practices

1. Alert on actionable conditions.
2. Prefer service impact and SLO signals.
3. Use `for` to prevent transient pages.
4. Group related alerts.
5. Use inhibition carefully.
6. Route by severity and ownership.
7. Separate production from non-production.
8. Provide runbooks.
9. Provide dashboards.
10. Test alert delivery.
11. Monitor the monitoring system.
12. Keep secrets outside Git.
13. Review alert quality regularly.
14. Remove obsolete alerts.
15. Document escalation policies.
16. Tune thresholds using production data.
17. Avoid high-cardinality alert labels.
18. Use recording rules for repeated calculations.
19. Treat alert rules as code.
20. Integrate alerts with incident response.

---

# 151. Production Readiness Checklist

## Prometheus

- [ ] Prometheus deployed redundantly where required
- [ ] Scrape targets healthy
- [ ] Recording rules validated
- [ ] Alert rules validated
- [ ] Rule selectors correct
- [ ] Retention appropriate

## Alertmanager

- [ ] HA configured
- [ ] Routes tested
- [ ] Receivers tested
- [ ] Inhibition tested
- [ ] Silences controlled
- [ ] Secrets protected
- [ ] Escalation tested

## Kubernetes

- [ ] Node alerts
- [ ] Pod alerts
- [ ] deployment alerts
- [ ] OOM alerts
- [ ] storage alerts
- [ ] capacity alerts

## Application

- [ ] availability
- [ ] errors
- [ ] latency
- [ ] traffic
- [ ] saturation
- [ ] SLO

## AWS

- [ ] ALB monitoring
- [ ] EKS health
- [ ] node health
- [ ] capacity
- [ ] relevant CloudWatch metrics

## Operations

- [ ] on-call ownership
- [ ] runbooks
- [ ] escalation
- [ ] alert testing
- [ ] incident process
- [ ] postmortems

---

# 152. Practical Command Reference

## Kubernetes

```bash
kubectl get nodes
kubectl describe node <node>
kubectl get pods -A
kubectl get pods -A -o wide
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get events -A --sort-by=.lastTimestamp
kubectl get deployments -A
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout history deployment/<name> -n <namespace>
```

---

## Monitoring

```bash
kubectl get prometheusrules -n monitoring
kubectl get pods -n monitoring
kubectl logs -n monitoring <prometheus-pod>
kubectl logs -n monitoring <alertmanager-pod>
```

---

## AWS

```bash
aws eks describe-cluster --name roboshop-prod
aws eks list-nodegroups --cluster-name roboshop-prod
aws eks describe-nodegroup \
  --cluster-name roboshop-prod \
  --nodegroup-name application
```

---

## Prometheus

```bash
promtool check rules alerts.yaml
promtool check config prometheus.yml
```

---

## Alertmanager

```bash
amtool check-config alertmanager.yml
```

---

# 153. Example Production PrometheusRule — SLO Burn

A simplified but production-oriented multi-window concept:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: checkout-slo-alerts
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
    environment: prod
spec:
  groups:
    - name: checkout.slo
      interval: 30s
      rules:

        - alert: CheckoutSLOFastBurn
          expr: |
            (
              sum(rate(http_requests_total{
                service="checkout",
                environment="prod",
                status=~"5.."
              }[5m]))
              /
              sum(rate(http_requests_total{
                service="checkout",
                environment="prod"
              }[5m]))
            ) > 0.05
          for: 5m
          labels:
            severity: critical
            team: checkout
            environment: prod
            category: slo
          annotations:
            summary: "Checkout is rapidly consuming error budget"
            description: |
              Checkout error ratio has exceeded the fast-burn threshold.
            runbook_url: "https://runbooks.example.internal/slo/checkout-fast-burn"

        - alert: CheckoutSLOSlowBurn
          expr: |
            (
              sum(rate(http_requests_total{
                service="checkout",
                environment="prod",
                status=~"5.."
              }[1h]))
              /
              sum(rate(http_requests_total{
                service="checkout",
                environment="prod"
              }[1h]))
            ) > 0.001
          for: 30m
          labels:
            severity: warning
            team: checkout
            environment: prod
            category: slo
          annotations:
            summary: "Checkout is slowly consuming error budget"
            description: |
              Checkout has experienced sustained elevated errors
              over the long observation window.
            runbook_url: "https://runbooks.example.internal/slo/checkout-slow-burn"
```

The thresholds above are examples. Real burn-rate thresholds should be derived from the organization's SLO and alerting policy rather than copied blindly.

---

# 154. Why SLO Alerts Are Better Than Arbitrary Thresholds

Consider:

```text
CPU = 80%
```

Is that bad?

Not necessarily.

Consider:

```text
Checkout successful request ratio = 98%
SLO = 99.9%
```

This is clearly a service-quality problem.

Therefore:

```text
SLO-based alerting
```

connects technical signals to business/customer outcomes.

---

# 155. Alerting and High Availability

Alerting itself is a production service.

If the application is highly available but:

```text
Prometheus = single instance
Alertmanager = single instance
```

then observability becomes a weak point.

Design monitoring with the same reliability mindset:

```text
HA infrastructure
+
HA monitoring
+
HA notification path
```

---

# 156. Monitoring the Monitoring Stack

Monitor:

```text
Prometheus availability
Prometheus target health
Prometheus rule evaluation failures
Alertmanager availability
Alertmanager notification failures
Grafana availability
ELK health
storage capacity
```

Useful principle:

> You need to know when your monitoring system stops monitoring.

---

# 157. Alert Rule Evaluation Failures

A rule can fail because:

- invalid PromQL
- missing metric
- unexpected labels
- operator/configuration problem

Prometheus exposes internal metrics that can help identify rule evaluation issues.

Alerting on monitoring-system failures is important because otherwise a broken rule can silently stop detecting incidents.

---

# 158. Alertmanager Queue / Notification Health

Monitor:

```text
notification failures
notification latency
pending notifications
configuration errors
```

If Alertmanager cannot send notifications, production incidents may go undetected operationally.

---

# 159. ELK Alerting Considerations

ELK is primarily the centralized logging platform in this capstone.

Use logs for:

- forensic context
- exception analysis
- security investigations
- application error patterns

Where possible:

```text
log -> metric
```

can be preferable for stable operational conditions.

For example, rather than paging directly on every:

```text
ERROR
```

event, expose:

```text
application_error_total
```

and alert on the error rate.

---

# 160. Alerting from Logs vs Metrics

Metrics are better for:

```text
rate
ratio
latency
resource utilization
availability
```

Logs are better for:

```text
specific exception
rare security event
detailed root cause
audit evidence
```

A mature observability design uses both.

---

# 161. Alerting During Deployments

Avoid alerting on expected temporary conditions during controlled changes without considering the deployment process.

Example:

```text
rolling deployment
```

may temporarily create:

```text
old pods terminating
new pods starting
```

Alerts should account for normal rollout behavior.

However, if rollout becomes stuck:

```text
deployment unavailable for 15m
```

then alert.

---

# 162. Deployment-Aware Alerting

Useful conditions:

```text
rollout stalled
new replica unavailable
old replica remains unexpectedly
readiness failures
new version causes elevated 5xx
```

The last signal is especially valuable because it captures customer impact.

---

# 163. Alerting and Canary Releases

If canary deployment is used:

```text
90% old version
10% new version
```

monitor:

```text
canary error rate
canary latency
canary saturation
```

Compare:

```text
canary
vs
baseline
```

before promoting the new version.

---

# 164. Alerting and Blue/Green

For blue/green:

```text
Blue = current
Green = new
```

Monitor Green before traffic switch.

After switch:

```text
error rate
latency
availability
```

should remain healthy.

---

# 165. Alerting During Incident Investigation

The alert is the starting point, not the conclusion.

For:

```text
ApplicationHighErrorRate
```

check:

```text
1. Is the alert real?
2. Is the impact global?
3. Which service?
4. Which endpoints?
5. Which pods?
6. Recent deployments?
7. Dependencies?
8. Infrastructure?
9. Network?
10. Database?
```

---

# 166. Production Troubleshooting Pattern

Use:

```text
SYMPTOM
   |
   v
MEASURE
   |
   v
CORRELATE
   |
   v
ISOLATE
   |
   v
ROOT CAUSE
   |
   v
MITIGATE
   |
   v
VERIFY
   |
   v
PREVENT
```

This pattern will be used heavily in `27-Production-Troubleshooting.md`.

---

# 167. Alert Correlation

Correlate:

```text
timestamp
service
cluster
node
deployment
version
environment
```

Example:

```text
10:05 deployment v42
10:06 CPU increase
10:07 error increase
10:08 latency increase
10:10 alert
```

This makes deployment correlation easier.

---

# 168. Alert Context and Version Labels

Where appropriate, expose:

```text
version
service
environment
cluster
namespace
```

This can help identify whether a particular release is causing problems.

Do not introduce unnecessary high-cardinality dimensions.

---

# 169. Alert Annotation Example with Version

```yaml
annotations:
  summary: "Checkout error rate increased"
  description: |
    Checkout 5xx ratio is above threshold.
    Environment: {{ $labels.environment }}
    Cluster: {{ $labels.cluster }}
    Service: {{ $labels.service }}
  runbook_url: "https://runbooks.example.internal/checkout/errors"
```

Keep templates robust against missing labels.

---

# 170. Alert Lifecycle During Recovery

When the metric returns to normal:

```text
Prometheus:
condition false

Alert:
resolved

Alertmanager:
sends resolved notification if configured

On-call:
confirms recovery
```

Do not assume:

```text
alert resolved = incident fully understood
```

The technical condition may recover while the underlying issue remains unexplained.

---

# 171. Post-Incident Alert Review

After a major incident ask:

```text
Did we detect it?
Was detection fast enough?
Was the right team paged?
Was severity correct?
Was the alert noisy?
Did we have enough context?
Was the runbook useful?
Could we detect the problem earlier?
Could we detect the root cause rather than the symptom?
```

---

# 172. Alert Quality Scorecard

A useful internal scorecard:

| Area | Question |
|---|---|
| Detection | Did it fire? |
| Accuracy | Was it a real problem? |
| Actionability | Could an engineer act? |
| Ownership | Was the team clear? |
| Context | Was enough information provided? |
| Routing | Did the right person receive it? |
| Noise | Was it excessive? |
| Recovery | Did it resolve correctly? |
| Runbook | Was the procedure available? |
| Prevention | Can future detection improve? |

---

# 173. Senior DevOps Perspective

A senior DevOps engineer should not say:

> "We have Prometheus, so monitoring is done."

A stronger answer is:

> "Prometheus collects and evaluates metrics, while Alertmanager handles routing, grouping, deduplication, inhibition and notification. We design alerts around customer impact, service health and SLOs, then supplement them with infrastructure, Kubernetes, AWS and application signals. Every production page has ownership, severity, runbook context and an escalation path."

---

# 174. Interview Question — What Is Alerting?

**Answer:**

Alerting is the mechanism that converts observability signals into actionable operational notifications. In our production EKS environment, Prometheus evaluates metric-based rules and sends firing alerts to Alertmanager. Alertmanager groups, deduplicates, inhibits and routes those alerts based on severity, environment and ownership. Critical production alerts can page the on-call engineer, while warnings can go to team notifications.

---

# 175. Interview Question — Monitoring vs Alerting?

**Answer:**

Monitoring provides visibility into system behavior through metrics, logs and dashboards. Alerting identifies conditions that require action. I do not alert on every abnormal metric. I design alerts around actionable conditions such as customer-impacting error rates, SLO violations, unavailable replicas, node failures, OOMKills and capacity risks.

---

# 176. Interview Question — Why Use Alertmanager?

**Answer:**

Prometheus evaluates the alert conditions, but Alertmanager is responsible for notification management. It handles grouping, deduplication, routing, silences and inhibition. This prevents every firing alert from independently paging engineers and allows us to route critical production alerts to the correct on-call team.

---

# 177. Interview Question — What Is Inhibition?

**Answer:**

Inhibition suppresses lower-priority alerts when a higher-priority related alert is firing. For example, if an EKS node is NotReady, several pods on that node may become unhealthy. Instead of paging independently for every warning, we can inhibit related lower-severity alerts while still preserving the root-cause critical alert. I use inhibition carefully so it does not hide actual customer impact.

---

# 178. Interview Question — How Do You Reduce Alert Noise?

**Answer:**

I first identify whether the alert is actionable. Then I tune the PromQL expression, use an appropriate `for` duration, group related alerts, use inhibition for dependent symptoms, separate warning from critical severity, and route by ownership. I also review alert frequency and false positives. Permanent silencing is not considered a real fix for noisy alerting.

---

# 179. Interview Question — How Do You Alert on Kubernetes?

**Answer:**

I monitor node health, CPU, memory, disk, pod restarts, OOMKilled containers, pending pods, deployment replica availability, failed jobs and application-level service health. I use kube-state-metrics and node metrics with Prometheus. For EKS-specific infrastructure, I also correlate Kubernetes signals with AWS metrics such as ALB target health and node-group/EC2 health.

---

# 180. Interview Question — How Do You Handle an Alert Storm?

**Answer:**

I first identify the common root cause rather than responding to each alert individually. I check Alertmanager grouping and inhibition, identify common cluster/node/service labels, and determine whether one infrastructure failure is producing many downstream symptoms. If necessary, I create a temporary scoped silence while resolving the incident. After recovery, I correct the alerting rules so the same storm does not recur.

---

# 181. Interview Question — How Do You Design SLO Alerts?

**Answer:**

I start with the SLI and SLO, calculate the error budget, and alert on error-budget burn rather than arbitrary infrastructure thresholds. I normally use different windows or burn-rate policies to detect both rapid outages and slower degradation. The goal is to page when the service is consuming its reliability budget at a rate that threatens the SLO.

---

# 182. Interview Question — What Is a Good Production Alert?

**Answer:**

A good production alert is actionable, owned, prioritized and contextual. It should identify the service, environment and impact, provide a meaningful threshold or SLO condition, include a runbook and dashboard where appropriate, route to the correct team, and avoid unnecessary notification noise.

---

# 183. Interview Question — How Do You Test Alerting?

**Answer:**

I validate Prometheus rules using `promtool`, validate Kubernetes manifests with server-side dry runs where possible, inspect Prometheus rule loading, verify the alert reaches Alertmanager, and test the receiver in a non-production or controlled environment. I also maintain a synthetic monitoring heartbeat such as a Watchdog/dead-man's-switch so that loss of the alerting pipeline itself can be detected.

---

# 184. Interview Question — What Happens When Prometheus Detects an Alert?

**Answer:**

Prometheus evaluates the PromQL expression. If the expression remains true for the configured `for` duration, the alert enters the firing state. Prometheus sends the alert to Alertmanager. Alertmanager then groups, deduplicates, applies inhibition and routing rules, and sends the notification to the configured receiver.

---

# 185. Interview Question — How Would You Alert on ALB Failures?

**Answer:**

I would combine AWS load-balancer metrics with Kubernetes and application metrics. I would monitor unhealthy target count, target response time and relevant 4xx/5xx metrics. If targets are unhealthy, I would trace the path from ALB to target group, Kubernetes ingress/service/endpoints and finally the pods. This avoids treating the ALB as an isolated component.

---

# 186. Interview Question — How Do You Monitor the Monitoring System?

**Answer:**

I monitor Prometheus availability, scrape failures, rule evaluation problems, Alertmanager availability and notification failures. I also use a Watchdog/dead-man's-switch concept to detect when the alert pipeline stops producing the expected heartbeat. Monitoring infrastructure needs its own reliability controls because an outage in monitoring can otherwise create a false sense of safety.

---

# 187. Senior Production Scenario

### Question

Production checkout is returning 10% 5xx. Prometheus has fired an alert. What do you do?

### Strong answer

```text
1. Acknowledge the incident.
2. Confirm the alert and customer impact.
3. Check Grafana for error-rate and latency trends.
4. Check the deployment timeline.
5. Check checkout pods and readiness.
6. Inspect application logs in ELK.
7. Check checkout dependencies.
8. Check ALB and service endpoints.
9. If a recent release correlates strongly with the incident,
   follow the rollback procedure.
10. Verify recovery through metrics and external availability.
11. Resolve the alert only after confirming stability.
12. Document root cause and improve alerting/runbooks if needed.
```

---

# 188. Senior Production Scenario — Node Failure

### Question

One EKS node is NotReady. What do you check?

### Answer

```text
kubectl get nodes
kubectl describe node <node>
kubectl get pods -A -o wide
kubectl get events -A --sort-by=.lastTimestamp
```

Then AWS:

```text
EC2 instance health
Auto Scaling Group
EKS node group
subnet capacity
security groups
```

I determine whether workloads were rescheduled and whether capacity remains sufficient. If the node is part of an autoscaled node group, I verify replacement behavior rather than immediately performing manual changes.

---

# 189. Senior Production Scenario — Alert Not Received

### Question

Prometheus shows a firing alert but the engineer receives no notification.

### Answer

I trace the pipeline:

```text
Prometheus rule
      |
      v
Prometheus firing alert
      |
      v
Alertmanager received?
      |
      v
Route matched?
      |
      v
Silenced?
      |
      v
Inhibited?
      |
      v
Receiver configured?
      |
      v
Notification delivery?
```

I inspect Prometheus and Alertmanager status/logs and validate the receiver integration.

---

# 190. Senior Production Scenario — Too Many Alerts

### Question

During a node outage you receive 500 alerts.

### Answer

I would identify the root-cause alert and correlate common labels. Then I would review Alertmanager grouping and inhibition. I would also determine whether the alerts represent real independent failures or downstream symptoms. After the incident, I would redesign the alert hierarchy to reduce noise without hiding important customer-impact signals.

---

# 191. Production Alerting Golden Rules

```text
ALERT ON ACTION
       |
       v
PROTECT SLOs
       |
       v
PAGE ONLY WHEN NECESSARY
       |
       v
ROUTE TO OWNER
       |
       v
PROVIDE CONTEXT
       |
       v
LINK RUNBOOK
       |
       v
REDUCE DUPLICATES
       |
       v
TEST THE PIPELINE
       |
       v
IMPROVE AFTER INCIDENTS
```

---

# 192. Final Production Alerting Architecture

```text
                         +----------------------+
                         |      Developers      |
                         +----------+-----------+
                                    |
                                    v
                              Git / GitHub
                                    |
                                    v
                              CI/CD Pipeline
                                    |
                  +-----------------+-----------------+
                  |                 |                 |
                  v                 v                 v
               Build             Security         Image
               Test               Scans            ECR
                  |                 |                 |
                  +-----------------+-----------------+
                                    |
                                    v
                              GitOps Repository
                                    |
                                    v
                                  Argo CD
                                    |
                                    v
                            AWS EKS Production
                                    |
          +-------------------------+--------------------------+
          |                         |                          |
          v                         v                          v
     Applications             Kubernetes                  AWS ALB
          |                         |                          |
          |                         v                          |
          |                    kube-state                      |
          |                    metrics                          |
          |                         |                          |
          +-------------------------+--------------------------+
                                    |
                                    v
                               Prometheus
                                    |
                    +---------------+----------------+
                    |                                |
                    v                                v
              Recording Rules                  Alert Rules
                                                     |
                                                     v
                                               Alertmanager
                                                     |
                       +-----------------------------+--------------------+
                       |                             |                    |
                       v                             v                    v
                    Email                     Team Chat             Incident System
                       |                             |                    |
                       +-----------------------------+--------------------+
                                                     |
                                                     v
                                                  On-call
                                                     |
                                                     v
                                             Incident Response
                                                     |
                    +--------------------------------+----------------+
                    |                                 |                |
                    v                                 v                v
                 Grafana                             ELK          Kubernetes/AWS
                    |                                 |                |
                    +---------------------------------+----------------+
                                                      |
                                                      v
                                                 Root Cause
                                                      |
                                                      v
                                             Recovery / Rollback
                                                      |
                                                      v
                                              Postmortem / Tuning
```

---

# 193. Final Summary

The production alerting design for this capstone is based on a clear separation of responsibilities:

```text
Prometheus
    |
    +--> scrape metrics
    +--> evaluate PromQL
    +--> recording rules
    +--> alert rules
    |
    v
Alertmanager
    |
    +--> grouping
    +--> deduplication
    +--> inhibition
    +--> silences
    +--> routing
    +--> notifications
    |
    v
Operations
```

Grafana provides visualization and investigation.

ELK provides centralized log analysis and forensic context.

AWS and EKS signals provide infrastructure context.

Argo CD provides GitOps deployment state.

CI/CD provides build, test and security signals.

The complete production alerting model is therefore:

```text
Customer / Service
        |
        v
Application
        |
        v
Kubernetes / EKS
        |
        +--> Metrics --> Prometheus --> Alertmanager --> On-call
        |
        +--> Logs ----> ELK -------------------------> Investigation
        |
        +--> State ---> Argo CD ---------------------> Deployment context
        |
        +--> AWS -----> CloudWatch / ALB ------------> Infrastructure context
```

The most important principle is:

> **A production alert is not merely a threshold. It is an operational contract between the monitoring system and the engineer responsible for restoring service.**

Every critical alert should answer:

```text
What happened?
Where did it happen?
Who owns it?
How serious is it?
What is the likely impact?
How do I investigate it?
How do I mitigate it?
Where is the runbook?
```

When those questions are answered consistently, alerting becomes a reliability system rather than a notification system.

---

# 194. Chapter Completion Checklist

```text
[✓] Alerting fundamentals
[✓] Monitoring vs alerting
[✓] Alert lifecycle
[✓] Alert sources
[✓] Metrics alerts
[✓] Log-related alerting
[✓] Infrastructure alerts
[✓] Kubernetes alerts
[✓] EKS alerts
[✓] Application alerts
[✓] Availability
[✓] Latency
[✓] Error rate
[✓] Resource utilization
[✓] Capacity
[✓] Security
[✓] Deployment alerts
[✓] SLI/SLO/SLA
[✓] Golden signals
[✓] Prometheus alerting
[✓] PromQL
[✓] PrometheusRule
[✓] Alertmanager
[✓] Routing
[✓] Receivers
[✓] Inhibition
[✓] Silences
[✓] Severity
[✓] Labels
[✓] Annotations
[✓] Runbooks
[✓] Ownership
[✓] On-call
[✓] Email
[✓] Team-chat integration
[✓] Incident escalation
[✓] Deduplication
[✓] Grouping
[✓] Alert storms
[✓] Flapping
[✓] Dead-man's-switch
[✓] Recording rules
[✓] Alert evaluation
[✓] Alertmanager HA
[✓] Production architecture
[✓] Node alerts
[✓] Pod alerts
[✓] Deployment alerts
[✓] OOMKilled
[✓] CPU/memory/disk
[✓] Network considerations
[✓] API server considerations
[✓] ALB alerts
[✓] SLO alerts
[✓] Database alerts
[✓] CI/CD alerts
[✓] Argo CD alerts
[✓] GitOps drift
[✓] Security alerts
[✓] Incident escalation
[✓] Alert response workflow
[✓] Troubleshooting
[✓] Production examples
[✓] Prometheus YAML
[✓] Alertmanager YAML
[✓] Routing
[✓] Inhibition
[✓] Environment routing
[✓] Team routing
[✓] Critical/warning strategy
[✓] Testing
[✓] Validation
[✓] Production best practices
[✓] Senior interview questions
```

---