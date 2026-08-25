# 18-GitOps-Observability.md

# GitOps Observability

## 1. Purpose

GitOps observability is the practice of monitoring the complete GitOps delivery system rather than monitoring only the application.

A production platform must answer:

```text
Is Git healthy?
Is Argo CD healthy?
Is reconciliation working?
Are Applications synchronized?
Are resources healthy?
Are ApplicationSets generating correctly?
Are repositories reachable?
Are target clusters reachable?
Are deployments progressing?
Are rollouts healthy?
Are application metrics healthy?
Are users experiencing errors?
```

For the user's production-oriented stack:

```text
Git
 |
 v
Jenkins / GitHub Actions
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
Argo Rollouts
 |
 v
EKS
 |
 +--> Prometheus
 +--> Grafana
 +--> ELK
 |
 v
Users
```

Observability must cover every important layer.

---

# 2. Observability vs Monitoring

Monitoring asks:

```text
Is something wrong?
```

Observability asks:

```text
Why is it wrong?
```

Monitoring commonly uses:

```text
metrics
alerts
dashboards
```

Observability combines:

```text
metrics
logs
events
health states
system relationships
deployment information
```

For GitOps, all of these are important.

---

# 3. Three Major Signals

The classic observability signals are:

```text
Metrics
Logs
Traces
```

The user's established stack is:

```text
Prometheus -> Metrics
Grafana    -> Visualization
ELK        -> Logs
```

The current environment does not rely on Jaeger/OpenTelemetry, so this file focuses on Prometheus, Grafana and ELK.

---

# 4. GitOps Adds Another Signal

GitOps systems expose declarative state:

```text
Desired state
Actual state
```

This creates useful operational signals such as:

```text
Synced
OutOfSync
Healthy
Degraded
Progressing
Missing
Unknown
```

These are not traditional infrastructure metrics, but they are critical GitOps health indicators.

---

# 5. GitOps Observability Architecture

```text
                       Git
                        |
                        v
                    Argo CD
                        |
             +----------+----------+
             |                     |
             v                     v
       Argo CD Metrics        Kubernetes
             |                     |
             v                     v
         Prometheus            Workloads
             |                     |
             v                     v
          Grafana              App Metrics
                                   |
                                   v
                                  ELK
```

---

# 6. Observability Layers

A production GitOps platform should observe:

```text
Layer 1  Git
Layer 2  CI/CD
Layer 3  Argo CD
Layer 4  ApplicationSet
Layer 5  Kubernetes API
Layer 6  Argo Rollouts
Layer 7  EKS infrastructure
Layer 8  Application workloads
Layer 9  ALB
Layer 10 Logs
```

---

# 7. Git Health

Git is the source of truth.

Observe:

```text
repository availability
authentication
clone/fetch failures
webhook delivery
commit frequency
PR failures
branch protection
```

If Git is unavailable:

```text
existing workloads may continue running
```

but:

```text
new desired state cannot be retrieved
```

---

# 8. Git Availability Is Different From Cluster Availability

If Git is down:

```text
EKS may continue serving users
```

If EKS is down:

```text
applications may become unavailable
```

Therefore:

```text
Git outage != application outage
```

but Git outage can prevent deployments and reconciliation.

---

# 9. Argo CD Observability

Argo CD exposes operational information for:

```text
API Server
Application Controller
Repo Server
ApplicationSet Controller
Redis
Applications
Repositories
Clusters
```

The exact metric names and labels can vary by Argo CD release.

Always validate metric names against the deployed version.

---

# 10. Argo CD Components to Monitor

```text
                    Argo CD
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
     API Server     Repo Server   Application
                                     Controller
        |              |              |
        +--------------+--------------+
                       |
                     Redis
                       |
                ApplicationSet
                  Controller
```

---

# 11. API Server Observability

Monitor:

```text
request rate
request latency
HTTP errors
authentication failures
resource usage
availability
```

Potential symptoms:

```text
UI slow
CLI slow
API requests failing
login failures
```

---

# 12. Application Controller Observability

The Application Controller is one of the most important components.

Monitor:

```text
reconciliation activity
queue depth
processing latency
errors
resource consumption
```

Symptoms of controller problems:

```text
Applications remain OutOfSync
health does not update
reconciliation is delayed
```

---

# 13. Repo Server Observability

Repo Server performs repository-related work such as:

```text
repository access
manifest generation
Helm rendering
Kustomize rendering
plugin execution
```

Monitor:

```text
request rate
latency
errors
CPU
memory
repository failures
manifest generation failures
```

---

# 14. Redis Observability

Redis is used internally by Argo CD.

Monitor:

```text
availability
memory
connections
CPU
latency
pod restarts
```

Do not assume Redis problems are harmless.

A Redis failure can contribute to Argo CD operational degradation depending on the deployment architecture and version.

---

# 15. ApplicationSet Controller Observability

Monitor:

```text
generator errors
reconciliation
Application creation failures
Application deletion failures
API errors
```

Symptoms:

```text
expected Applications are not generated
Applications are stale
cluster changes are not reflected
```

---

# 16. Kubernetes API Observability

Argo CD depends heavily on the Kubernetes API.

If Kubernetes API access fails:

```text
Argo CD cannot reliably compare or apply state
```

Monitor:

```text
API latency
API errors
request saturation
authentication
authorization
control-plane health
```

---

# 17. EKS Control Plane

AWS EKS control-plane observability should include the AWS-supported control-plane logging and metrics available for the chosen configuration.

Common areas include:

```text
API activity
authentication
audit events
scheduler behavior
controller behavior
```

Enable only the logs and retention required by security and operational policy.

---

# 18. Kubernetes Workload Observability

For each workload observe:

```text
Pod availability
restart count
CPU
memory
requests
limits
HPA status
replica count
readiness
liveness
events
```

---

# 19. GitOps Application Health

Argo CD Application health is a critical operational signal.

Important states include:

```text
Healthy
Progressing
Degraded
Suspended
Missing
Unknown
```

Exact health behavior can vary by resource type.

---

# 20. Sync Status

Important sync states:

```text
Synced
OutOfSync
Unknown
```

A production dashboard should make OutOfSync Applications visible.

---

# 21. Why OutOfSync Is Important

OutOfSync can mean:

```text
legitimate pending deployment
manual drift
controller mutation
ignored field difference
failed synchronization
```

Therefore:

```text
OutOfSync != automatically an incident
```

But production OutOfSync should have ownership and expected-duration rules.

---

# 22. Desired vs Actual State

GitOps observability should visualize:

```text
Git desired state
        |
        v
Argo CD desired state
        |
        v
Kubernetes actual state
```

Any difference can become an operational signal.

---

# 23. Drift

Example:

Git:

```yaml
replicas: 6
```

Cluster:

```yaml
replicas: 8
```

Argo CD can detect:

```text
OutOfSync
```

If self-heal is enabled:

```text
actual -> desired
```

---

# 24. Drift Monitoring

Useful metrics:

```text
number of OutOfSync Applications
number of Degraded Applications
number of failed syncs
number of sync operations
sync duration
```

---

# 25. Drift Is Not Always Bad

Some Kubernetes resources have fields that controllers mutate.

Examples:

```text
status
generated fields
controller-owned fields
```

Therefore configure ignore differences carefully.

---

# 26. Alerting on Drift

A useful alert could be:

```text
Production Application OutOfSync
for > 15 minutes
```

rather than:

```text
OutOfSync for 5 seconds
```

This reduces false alerts during normal deployments.

---

# 27. Production Alert Philosophy

Alerts should indicate:

```text
action required
```

not:

```text
interesting information
```

Examples:

Good:

```text
Production app Degraded for 10m
```

Poor:

```text
Application changed state
```

---

# 28. Argo CD Metrics to Collect

Depending on version/configuration, look for metrics covering:

```text
Applications
sync
health
reconciliation
API requests
repository operations
manifest generation
controller queues
```

Do not hard-code dashboard queries without verifying the metric names exposed by the installed version.

---

# 29. Discovering Argo CD Metrics

First identify Services:

```bash
kubectl get svc -n argocd
```

Then inspect Argo CD component configuration and ServiceMonitors/PodMonitors if using Prometheus Operator.

For direct metrics endpoints, follow the deployed Argo CD version documentation and configuration.

---

# 30. Prometheus Integration

If using Prometheus Operator, a production platform may use:

```text
ServiceMonitor
```

or:

```text
PodMonitor
```

depending on the deployment.

Conceptually:

```text
Argo CD
  |
  | /metrics
  v
Prometheus
  |
  v
Grafana
```

---

# 31. ServiceMonitor Example

A ServiceMonitor is only an example because the actual Argo CD Services and metric ports vary by deployment.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: argocd-server
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  namespaceSelector:
    matchNames:
      - argocd
  selector:
    matchLabels:
      app.kubernetes.io/name: argocd-server
  endpoints:
    - port: metrics
      interval: 30s
      path: /metrics
```

Validate labels and port names against the installed Argo CD manifests.

---

# 32. Why ServiceMonitor Labels Matter

Prometheus Operator may select ServiceMonitors using labels.

If the label does not match:

```text
ServiceMonitor exists
```

but:

```text
Prometheus does not discover it
```

This is a common troubleshooting issue.

---

# 33. Prometheus Targets

Check targets in Prometheus.

Look for:

```text
UP
DOWN
```

If Argo CD target is DOWN:

```text
check Service
check endpoint
check port
check network
check TLS
check ServiceMonitor selector
```

---

# 34. Kubernetes Discovery

Prometheus commonly discovers targets through Kubernetes APIs.

Therefore Prometheus itself requires:

```text
RBAC
network connectivity
API access
```

---

# 35. Prometheus Query Examples

The exact metric names depend on the Argo CD version.

Useful conceptual queries include:

```promql
count(...)
```

for counting resources,

```promql
rate(...[5m])
```

for request rates,

and:

```promql
histogram_quantile(...)
```

for latency distributions where histogram metrics are available.

---

# 36. Never Assume Metric Names

A common production mistake is copying an old dashboard query.

Argo CD versions can change:

```text
metric names
labels
metric semantics
```

Always inspect:

```text
/metrics
```

and validate the query.

---

# 37. Grafana

Grafana provides the visualization layer.

A production GitOps dashboard should contain:

```text
Argo CD health
sync status
reconciliation
repositories
clusters
ApplicationSets
Rollouts
Kubernetes
applications
ALB
```

---

# 38. Executive GitOps Dashboard

A high-level dashboard:

```text
Production Applications
Healthy: 42
Degraded: 1
OutOfSync: 2

Clusters
Healthy: 3
Unreachable: 0

Sync Failures
Last 1h: 2

Rollouts
Running: 3
Aborted: 1
```

This gives operators immediate context.

---

# 39. Argo CD Dashboard

Suggested panels:

```text
Applications by Health
Applications by Sync Status
Sync Operations
Sync Failures
Reconciliation Duration
Repository Errors
Cluster Connection Status
ApplicationSet Errors
```

---

# 40. Application Health Panel

Visualize:

```text
Healthy
Progressing
Degraded
Missing
Unknown
```

Use environment filters:

```text
dev
qa
prod
```

---

# 41. Sync Status Panel

Visualize:

```text
Synced
OutOfSync
Unknown
```

Production should have a clear OutOfSync view.

---

# 42. Sync Failure Panel

Track:

```text
failed sync count
failure rate
affected applications
```

A spike may indicate:

```text
bad Git commit
Helm rendering issue
RBAC problem
API failure
dependency issue
```

---

# 43. Reconciliation Latency

A reconciliation delay can mean:

```text
controller overloaded
API slow
repository slow
large application
manifest generation expensive
network issue
```

Monitor both:

```text
average
tail latency
```

---

# 44. Repository Observability

Track:

```text
repo fetch failures
authentication failures
manifest generation failures
timeouts
```

---

# 45. Private Repository Failure

Symptoms:

```text
Application becomes Unknown/OutOfSync
manifest cannot be generated
repo-server logs show authentication errors
```

Check:

```bash
argocd repo list
```

and repository credentials.

---

# 46. Cluster Connection Observability

Run:

```bash
argocd cluster list
```

Watch for:

```text
Successful
Failed
Unknown
```

A target cluster becoming unreachable is a high-priority signal for centralized Argo CD.

---

# 47. Multi-Cluster Dashboard

Example:

```text
Cluster       Environment    Connection
eks-dev       dev            Healthy
eks-qa        qa             Healthy
eks-prod-a    prod           Healthy
eks-prod-b    prod           Failed
```

This is more useful than only monitoring the Argo CD server.

---

# 48. ApplicationSet Observability

Monitor:

```text
expected Applications
actual Applications
generator errors
template errors
cluster discovery
Git discovery
```

---

# 49. ApplicationSet Failure Example

Expected:

```text
10 clusters
```

Generated:

```text
8 Applications
```

This should be detectable.

Possible causes:

```text
cluster label missing
generator error
template error
RBAC
duplicate name
invalid destination
```

---

# 50. Kubernetes Events

Events are extremely useful for deployment troubleshooting:

```bash
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

Events can show:

```text
FailedScheduling
FailedMount
BackOff
Unhealthy
Pulling
Pulled
```

---

# 51. Events Are Short-Lived

Kubernetes Events are not a long-term audit store.

For historical investigation use:

```text
ELK
Cloud/AWS logging
Kubernetes audit logs
```

as appropriate.

---

# 52. Logs

The user's logging stack is:

```text
ELK
```

Use it for:

```text
Argo CD component logs
Kubernetes application logs
ALB access logs where configured
application exceptions
deployment failures
```

---

# 53. Argo CD Logs

Useful commands:

```bash
kubectl logs -n argocd deploy/argocd-server
kubectl logs -n argocd deploy/argocd-repo-server
kubectl logs -n argocd deploy/argocd-applicationset-controller
```

Application Controller may be deployed as a StatefulSet or another resource depending on Argo CD version/manifests, so first inspect:

```bash
kubectl get pods -n argocd
```

---

# 54. Application Controller Logs

Look for:

```text
reconciliation errors
API failures
permission errors
resource comparison errors
```

---

# 55. Repo Server Logs

Look for:

```text
Git clone failure
SSH authentication
HTTPS authentication
Helm rendering
Kustomize errors
plugin failures
timeouts
```

---

# 56. ApplicationSet Controller Logs

Look for:

```text
generator failures
template errors
API errors
Application creation/deletion failures
```

---

# 57. ELK Pipeline

Conceptual architecture:

```text
Kubernetes Pods
      |
      v
Log collector
      |
      v
Logstash
      |
      v
Elasticsearch
      |
      v
Kibana
```

The exact collector implementation depends on the platform.

---

# 58. GitOps Log Fields

Useful structured fields:

```text
timestamp
cluster
environment
namespace
application
component
revision
resource
severity
message
```

---

# 59. Structured Logging

Prefer:

```json
{
  "level": "error",
  "component": "repo-server",
  "application": "cart",
  "environment": "prod",
  "message": "manifest generation failed"
}
```

over unstructured text when possible.

---

# 60. Correlation

When an incident occurs:

```text
Git commit
   |
   v
Argo CD sync
   |
   v
Rollout
   |
   v
Pod logs
   |
   v
Application metrics
```

Operators should be able to connect these events.

---

# 61. Deployment Correlation

A useful deployment record includes:

```text
Git SHA
image digest
environment
Argo CD Application
Argo CD revision
Rollout revision
timestamp
```

---

# 62. Image Digest

Use:

```text
sha256:...
```

instead of relying only on:

```text
v1.2.3
```

This provides stronger artifact identity.

---

# 63. Application Version Label

Example:

```yaml
labels:
  app.kubernetes.io/name: cart
  app.kubernetes.io/version: "2.4.1"
  app.kubernetes.io/part-of: roboshop
```

These labels improve operational filtering.

---

# 64. Prometheus Application Metrics

For RoboShop services, examples include:

```text
http_requests_total
http_request_duration_seconds
application_errors_total
```

Actual instrumentation names must match the application implementation.

---

# 65. Golden Signals

Use the four golden signals:

```text
Latency
Traffic
Errors
Saturation
```

---

# 66. Latency

Track:

```text
p50
p95
p99
```

p95/p99 are particularly useful for identifying tail latency.

---

# 67. Traffic

Track:

```text
requests/sec
requests/min
active requests
```

Traffic context is important when interpreting error rates.

---

# 68. Errors

Track:

```text
HTTP 4xx
HTTP 5xx
application exceptions
business failures
```

Distinguish client errors from server errors.

---

# 69. Saturation

Track:

```text
CPU
memory
disk
network
connection pools
thread pools
database capacity
queue depth
```

---

# 70. Kubernetes Saturation

Useful signals:

```text
Pod CPU
Pod memory
node CPU
node memory
ephemeral storage
Pending Pods
```

---

# 71. EKS Node Monitoring

Monitor:

```text
node readiness
CPU
memory
disk pressure
PID pressure
network
Pod capacity
```

---

# 72. Pending Pods

A spike in Pending Pods can indicate:

```text
insufficient CPU
insufficient memory
taints
affinity
topology constraints
node capacity
```

Use:

```bash
kubectl describe pod <pod> -n <namespace>
```

---

# 73. CrashLoopBackOff

Monitor restart counts and Pod health.

Troubleshoot:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> --previous -n roboshop
```

---

# 74. OOMKilled

Observe:

```text
container memory
restart count
OOMKilled state
```

Check:

```bash
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
```

---

# 75. Probe Failures

Monitor:

```text
readiness failures
liveness failures
startup failures
```

A probe failure can make a deployment look like a GitOps problem when the actual issue is application health.

---

# 76. HPA Observability

Monitor:

```text
desired replicas
current replicas
CPU utilization
memory utilization
scaling events
```

Commands:

```bash
kubectl get hpa -n roboshop
kubectl describe hpa <name> -n roboshop
```

---

# 77. ALB Observability

For AWS ALB, monitor appropriate AWS metrics such as:

```text
RequestCount
HTTPCode_ELB_5XX_Count
HTTPCode_Target_5XX_Count
TargetResponseTime
HealthyHostCount
UnHealthyHostCount
```

The exact metric dimensions depend on the AWS resource and monitoring configuration.

---

# 78. ALB vs Application Errors

Distinguish:

```text
ALB-generated error
```

from:

```text
target-generated error
```

For example:

```text
HTTPCode_ELB_5XX_Count
```

and:

```text
HTTPCode_Target_5XX_Count
```

can help identify where failures originate.

---

# 79. ALB Latency

Use target response latency to determine whether:

```text
ALB
```

or:

```text
application
```

is contributing to slow responses.

---

# 80. ALB Health

If:

```text
HealthyHostCount decreases
```

investigate:

```text
Pod readiness
Service endpoints
target port
health check
security groups
```

---

# 81. ECR Observability

Monitor deployment-related ECR problems:

```text
image pull failures
repository access
image availability
tag/digest correctness
```

Kubernetes symptoms include:

```text
ImagePullBackOff
ErrImagePull
```

---

# 82. Image Pull Troubleshooting

```bash
kubectl describe pod <pod> -n roboshop
```

Check:

```text
image reference
ECR permissions
node IAM role
network
image existence
```

---

# 83. IAM and GitOps Observability

Monitor failures involving:

```text
EKS cluster access
ECR access
AWS Load Balancer Controller
Secrets Manager
S3
```

Use least privilege and audit AWS API activity through the organization's approved AWS logging setup.

---

# 84. Kubernetes Audit

Kubernetes audit logs can help answer:

```text
Who changed this?
What API operation occurred?
When?
Which resource?
```

This is especially valuable during:

```text
break-glass access
manual changes
security incidents
```

---

# 85. Git Audit

Git history answers:

```text
Who changed the manifest?
What changed?
When?
Which PR?
Which commit?
```

Git is therefore a major audit source in GitOps.

---

# 86. GitOps Audit Chain

```text
Developer
 |
 v
PR
 |
 v
Review
 |
 v
Merge
 |
 v
Argo CD
 |
 v
Kubernetes
 |
 v
Application
```

Each stage should have traceable evidence.

---

# 87. Alerting Architecture

```text
Prometheus
    |
    v
Alertmanager
    |
    +--> Email
    +--> Slack
    +--> PagerDuty
    +--> Incident platform
```

The exact notification system depends on organizational tooling.

---

# 88. Alert Categories

### Platform

```text
Argo CD unavailable
Repo Server failing
Controller overloaded
```

### GitOps

```text
Production OutOfSync
Sync failure
Cluster unreachable
```

### Kubernetes

```text
Pod crash
Node pressure
Pending Pods
```

### Application

```text
5xx
latency
business failure
```

---

# 89. Severity

Example:

```text
P1:
production unavailable

P2:
major degradation

P3:
limited impact

P4:
informational
```

Map severity to actual organizational incident policy.

---

# 90. Alert: Production Degraded

Conceptual PromQL:

```promql
# Use the exact Argo CD health metric exposed by
# your installed version.
```

The important point is to alert on:

```text
production Application
+
Degraded
+
sustained duration
```

rather than relying on an outdated metric name.

---

# 91. Alert: OutOfSync

A production alert can be:

```text
Application OutOfSync for > 15m
```

Exceptions may include:

```text
approved maintenance
planned rollout
known controller-owned difference
```

---

# 92. Alert: Cluster Unreachable

Centralized Argo CD should alert if:

```text
prod EKS cluster
```

cannot be reached.

This can affect:

```text
deployment
drift correction
health reporting
```

---

# 93. Alert: Sync Failure

Alert when:

```text
sync fails repeatedly
```

rather than on a single transient error.

---

# 94. Alert: Repo Server Failure

Alert when:

```text
Repo Server unavailable
```

or:

```text
manifest generation failure rate
```

exceeds an operational threshold.

---

# 95. Alert: Controller Backlog

A sustained controller queue or reconciliation delay can indicate:

```text
controller overload
API saturation
large application count
slow resources
```

---

# 96. Alert: ApplicationSet Failure

Alert when:

```text
expected generated Applications
```

are missing or generator errors persist.

---

# 97. Dashboard: Platform Health

Suggested panels:

```text
Argo CD API status
Application Controller status
Repo Server status
ApplicationSet Controller status
Redis status
Prometheus status
Grafana status
ELK status
```

---

# 98. Dashboard: GitOps Health

```text
Synced Applications
OutOfSync Applications
Degraded Applications
Failed Syncs
Cluster Connections
Repository Connections
ApplicationSet Errors
```

---

# 99. Dashboard: Kubernetes Health

```text
Nodes Ready
Pending Pods
CrashLoopBackOff
OOMKilled
CPU
Memory
HPA
PDB
```

---

# 100. Dashboard: Application Health

For RoboShop:

```text
user
catalog
cart
order
payment
inventory
notification
```

Each should show:

```text
availability
latency
error rate
restarts
```

---

# 101. Dashboard: ALB

```text
requests
5xx
target 5xx
target latency
healthy targets
unhealthy targets
```

---

# 102. Dashboard: Deployment

```text
deployments today
failed deployments
rollouts active
rollouts aborted
sync duration
deployment frequency
```

---

# 103. Deployment Frequency

Track:

```text
deployments/week
```

This is a DORA-style delivery metric.

---

# 104. Lead Time for Changes

Measure:

```text
commit
   |
   v
production
```

This helps evaluate GitOps delivery speed.

---

# 105. Change Failure Rate

Track:

```text
failed deployments / total deployments
```

A failed deployment can be:

```text
rollback
abort
production incident
```

Define the organization's measurement consistently.

---

# 106. Mean Time to Restore

Measure:

```text
incident start
      |
      v
service restored
```

GitOps and automated rollback can reduce recovery time.

---

# 107. DORA Metrics

A production GitOps organization can monitor:

```text
Deployment Frequency
Lead Time for Changes
Change Failure Rate
Time to Restore Service
```

Progressive delivery can improve change failure impact and recovery when correctly implemented.

---

# 108. GitOps SLO

An internal GitOps platform can define SLOs such as:

```text
99.9% Argo CD availability
99% successful sync operations
<5 minute reconciliation delay
<10 minute production drift detection
```

These are examples, not universal targets.

---

# 109. Reconciliation SLO

Example:

```text
95% of approved Git changes
reconciled within 5 minutes
```

Measure from:

```text
Git revision available
```

to:

```text
cluster desired state applied
```

---

# 110. Drift SLO

Example:

```text
production drift detected within 5 minutes
```

Then:

```text
self-healed within 10 minutes
```

if self-heal is enabled.

---

# 111. Observability Data Retention

Decide retention for:

```text
Prometheus metrics
Grafana dashboards
ELK logs
Kubernetes audit logs
Git history
Argo CD events
```

Retention should balance:

```text
incident investigation
compliance
cost
```

---

# 112. Prometheus Retention

Metrics retention depends on:

```text
disk
remote storage
scrape volume
cardinality
retention period
```

Do not retain everything forever without capacity planning.

---

# 113. Elasticsearch Retention

Logs can become expensive quickly.

Use:

```text
Index Lifecycle Management
retention policy
hot/warm/cold tiers
```

where appropriate.

---

# 114. Log Volume Control

Avoid logging:

```text
debug-level everything
```

in production indefinitely.

Use appropriate log levels.

---

# 115. High Cardinality

High-cardinality Prometheus labels can cause:

```text
high memory
slow queries
large TSDB
```

Review labels regularly.

---

# 116. Metrics Naming

Use consistent names.

Examples:

```text
http_requests_total
http_request_duration_seconds
```

Follow the application's instrumentation conventions.

---

# 117. Metric Labels

Useful:

```text
service
environment
version
status
method
route
```

Be careful with:

```text
user_id
request_id
session_id
```

because of cardinality.

---

# 118. Route Cardinality

Do not create unique labels for every dynamic URL.

Bad:

```text
/path/user/12345
/path/user/12346
```

Prefer:

```text
/path/user/{id}
```

or normalized route labels.

---

# 119. Application Log Levels

Typical:

```text
ERROR
WARN
INFO
DEBUG
```

Production default is often:

```text
INFO
```

with temporary targeted debugging during incidents.

---

# 120. Sensitive Data

Do not send secrets into:

```text
Prometheus labels
logs
Git
Grafana annotations
```

Avoid logging:

```text
passwords
tokens
API keys
session credentials
```

---

# 121. Secret Leakage Incident

If a secret appears in logs:

```text
1. revoke/rotate secret
2. investigate exposure
3. remove future logging
4. assess retention/access
5. document incident
```

Do not assume deleting the current log removes historical copies.

---

# 122. Observability Access Control

Grafana and Kibana can contain:

```text
production logs
internal architecture
user data
security events
```

Use:

```text
SSO
RBAC
least privilege
audit
```

---

# 123. Argo CD RBAC and Observability

Different teams may need different visibility.

Example:

```text
Developers:
dev/qa

Operations:
all environments

Security:
audit/security views
```

Do not give unnecessary write permissions merely for dashboard access.

---

# 124. Multi-Cluster Observability

Centralized dashboards should include:

```text
cluster
account
region
environment
```

Example:

```text
prod
 |
 +-- account A / ap-south-1
 +-- account B / ap-south-1
 +-- account C / ap-southeast-1
```

---

# 125. Cluster Labels

Use consistent labels:

```text
environment=prod
region=ap-south-1
account=production
team=platform
```

These labels improve filtering.

---

# 126. Environment Separation

Dashboard filters:

```text
environment=dev
environment=qa
environment=prod
```

Never mix production and development alerts accidentally.

---

# 127. Namespace Observability

Use namespace labels:

```text
namespace=roboshop
```

Then filter:

```text
service=cart
namespace=roboshop
environment=prod
```

---

# 128. GitOps Application Labels

Useful Application metadata:

```yaml
metadata:
  labels:
    environment: prod
    team: roboshop
    criticality: high
```

These can improve operational organization.

---

# 129. Grafana Variables

Useful dashboard variables:

```text
$cluster
$environment
$namespace
$application
$service
$version
```

This allows one dashboard to serve multiple environments.

---

# 130. Production Grafana Layout

```text
Row 1:
Platform availability

Row 2:
GitOps health

Row 3:
Cluster health

Row 4:
Application health

Row 5:
ALB

Row 6:
Deployment/Rollout

Row 7:
Logs
```

---

# 131. Observability During Deployment

Before deployment:

```text
baseline metrics
```

During deployment:

```text
canary metrics
```

After deployment:

```text
post-release comparison
```

---

# 132. Deployment Annotations

Grafana can display deployment markers.

Conceptually:

```text
10:00  v2.4.1 deployed
10:05  error rate rises
10:08  rollout aborted
```

This makes incident correlation easier.

---

# 133. Git Commit as Deployment Marker

A deployment event can reference:

```text
Git SHA
```

This helps connect:

```text
metric anomaly
```

to:

```text
specific code/config revision
```

---

# 134. Rollout Marker

Progressive delivery should expose:

```text
stable version
canary version
current weight
analysis status
```

on dashboards where practical.

---

# 135. Observability for Argo Rollouts

Monitor:

```text
Rollout phase
current step
analysis runs
ReplicaSets
Pod readiness
abort count
promotion count
```

---

# 136. Rollout Failure Rate

A useful platform KPI:

```text
aborted rollouts / total rollouts
```

Break down by:

```text
service
team
environment
reason
```

---

# 137. Rollout Duration

Track:

```text
start -> complete
```

A sudden increase can indicate:

```text
slow startup
slow analysis
capacity
ALB routing
controller issues
```

---

# 138. Deployment Health vs Application Health

A deployment can be:

```text
successful
```

while the application is:

```text
degraded
```

Therefore observe both.

---

# 139. GitOps Health Model

Think in four states:

```text
Desired state
Sync state
Resource health
User experience
```

Example:

```text
Git = v2
Argo CD = Synced
Pods = Healthy
Users = 5xx
```

The platform is synchronized but the application is unhealthy.

---

# 140. Important Interview Concept

```text
Synced != Healthy
```

and:

```text
Healthy != User experience is perfect
```

These states measure different things.

---

# 141. OutOfSync But Healthy

Example:

```text
Git desired = 6 replicas
cluster actual = 8
```

Application may still be healthy.

But it is:

```text
OutOfSync
```

because actual state differs from desired state.

---

# 142. Synced But Degraded

Example:

```text
Git image = v2
Argo CD synced
Pod crashes
```

The Application can be:

```text
Synced
Degraded
```

This distinction is frequently tested in interviews.

---

# 143. Unknown Health

Unknown can indicate:

```text
resource type not recognized
custom health unavailable
API problem
```

Investigate rather than assuming failure.

---

# 144. Custom Health Checks

Argo CD can support custom health logic for resource types.

Use this when the default health assessment does not accurately represent application readiness.

Avoid unnecessarily complex custom health scripts.

---

# 145. Health Check Design

A good health check answers:

```text
Can this resource safely be considered operational?
```

It should not simply check:

```text
object exists
```

---

# 146. Application Health Dependencies

A parent application can contain:

```text
Deployment
Service
Ingress
ConfigMap
Secret
HPA
```

Health should be interpreted at the application level as well as resource level.

---

# 147. Ingress Observability

Monitor:

```text
Ingress existence
ALB status
target health
TLS
routing
```

Commands:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress <name> -n roboshop
```

---

# 148. Service Observability

Check:

```bash
kubectl get svc -n roboshop
kubectl get endpoints -n roboshop
```

No endpoints can explain:

```text
ALB unhealthy
connection failures
5xx
```

---

# 149. EndpointSlice

Modern Kubernetes uses EndpointSlice resources.

Check:

```bash
kubectl get endpointslice -n roboshop
```

This can be useful when troubleshooting Service routing.

---

# 150. Network Observability

When traffic fails:

```text
ALB
 |
 v
Service
 |
 v
Pod
```

inspect each layer.

Possible causes:

```text
security group
network policy
service selector
target port
readiness
DNS
```

---

# 151. DNS Observability

Test from a Pod:

```bash
kubectl exec -it <pod> -n roboshop -- nslookup cart
```

or use an appropriate DNS utility available in the container.

---

# 152. NetworkPolicy

If NetworkPolicies are used:

```text
Argo CD
Prometheus
application
```

may require explicitly allowed traffic.

A NetworkPolicy can make a component appear unhealthy even when the Pod itself is running.

---

# 153. Prometheus NetworkPolicy

If Prometheus cannot scrape:

```text
allow Prometheus -> metrics endpoint
```

using the organization's NetworkPolicy model.

---

# 154. Grafana Data Source Failure

If dashboards show:

```text
No data
```

check:

```text
Prometheus datasource
credentials
URL
network
TLS
query
time range
```

---

# 155. ELK Data Source Failure

If Kibana shows no logs:

```text
collector
Logstash
Elasticsearch
index
pipeline
retention
```

must be checked.

---

# 156. Observability Stack Failure

If:

```text
Prometheus down
```

you lose metrics.

If:

```text
ELK down
```

you lose convenient log search.

If:

```text
Grafana down
```

raw Prometheus may still be available.

Design monitoring so the observability platform itself is monitored.

---

# 157. Monitoring the Monitoring Stack

Observe:

```text
Prometheus
Grafana
Elasticsearch
Logstash
collectors
Alertmanager
```

Otherwise a monitoring outage can remain invisible.

---

# 158. Prometheus Self-Monitoring

Useful areas:

```text
scrape health
TSDB size
ingestion
query latency
rule evaluation
target count
```

---

# 159. Elasticsearch Self-Monitoring

Monitor:

```text
cluster health
heap
disk
shards
ingestion
query latency
```

---

# 160. Logstash Self-Monitoring

Monitor:

```text
events in
events out
pipeline errors
queue
CPU
memory
```

---

# 161. Grafana Self-Monitoring

Monitor:

```text
availability
request latency
datasource errors
resource usage
```

---

# 162. Alertmanager Self-Monitoring

Monitor:

```text
alert delivery
notification failures
silences
queue
```

---

# 163. Alert Fatigue

Too many alerts cause:

```text
ignored alerts
missed incidents
operator burnout
```

Use:

```text
severity
duration
grouping
deduplication
maintenance windows
```

---

# 164. Alert Grouping

Group related failures:

```text
Argo CD outage
```

rather than sending:

```text
100 individual Application alerts
```

when the root cause is the platform itself.

---

# 165. Alert Suppression

During planned maintenance:

```text
silence expected alerts
```

but ensure:

```text
silence has owner
silence has expiration
```

---

# 166. Incident Correlation

Example:

```text
10:00 Git commit
10:02 Argo CD sync
10:03 Rollout canary
10:05 5xx rises
10:06 Analysis fails
10:07 Rollout abort
```

A good observability system lets you reconstruct this timeline quickly.

---

# 167. GitOps Incident Timeline

Capture:

```text
commit
PR
Argo revision
sync
rollout
metric anomaly
alert
rollback
recovery
```

---

# 168. Production Postmortem

After an incident ask:

```text
What changed?
What detected it?
How long did detection take?
Why did progressive delivery continue/stop?
Was rollback successful?
Could the signal be improved?
```

---

# 169. Observability Runbook

For a production degradation:

```text
1. Check Grafana.
2. Check Argo CD Application.
3. Check Sync status.
4. Check Health status.
5. Check Rollout.
6. Check ALB.
7. Check Pods.
8. Check events.
9. Check Prometheus.
10. Check ELK.
11. Identify root cause.
12. Restore service.
13. Reconcile Git.
```

---

# 170. First 5 Minutes of an Incident

### Minute 1

```text
Is production actually impacted?
```

### Minute 2

```text
What changed?
```

### Minute 3

```text
Is there an active rollout?
```

### Minute 4

```text
Are errors isolated to canary?
```

### Minute 5

```text
Abort/rollback if policy requires.
```

---

# 171. Production Observability Decision Tree

```text
User reports errors
       |
       v
Check Grafana
       |
       +--> no metric change
       |       |
       |       v
       |    logs/network
       |
       +--> 5xx increased
               |
               v
          Check rollout
               |
          +----+----+
          |         |
       Canary     Stable
          |         |
          v         v
       abort      investigate
```

---

# 172. GitOps Troubleshooting Decision Tree

```text
Deployment missing
       |
       v
Is Application generated?
       |
      no
       |
ApplicationSet
       |
      yes
       |
       v
Is Application Synced?
       |
      no
       |
Repo / render / sync
       |
      yes
       |
       v
Is Application Healthy?
       |
      no
       |
Kubernetes / workload
```

---

# 173. ApplicationSet Troubleshooting

Check:

```bash
kubectl get applications -n argocd
kubectl get applicationsets -n argocd
kubectl describe applicationset <name> -n argocd
```

Then inspect controller logs.

---

# 174. Repository Troubleshooting

```bash
argocd repo list
```

If repository is not accessible:

```text
credential
SSH key
known host
HTTPS token
network
repository URL
```

---

# 175. Cluster Troubleshooting

```bash
argocd cluster list
```

If failed:

```text
credentials
RBAC
API endpoint
network
AWS authentication
cluster status
```

---

# 176. Application Troubleshooting

```bash
argocd app get <app>
argocd app diff <app>
```

Then:

```bash
kubectl get applications -n argocd
kubectl describe application <app> -n argocd
```

---

# 177. Sync Troubleshooting

```bash
argocd app sync <app>
```

If it fails:

```text
read the exact resource error
```

Do not repeatedly retry without understanding the failure.

---

# 178. Resource-Level Troubleshooting

If a Deployment is unhealthy:

```bash
kubectl describe deployment <name> -n <namespace>
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
```

---

# 179. Event-Based Troubleshooting

```bash
kubectl get events -n <namespace> --sort-by=.lastTimestamp
```

Look for:

```text
FailedMount
FailedScheduling
BackOff
Unhealthy
Failed
```

---

# 180. Observability for Sync Waves

If sync waves are used:

```text
wave -2
platform
wave -1
namespace/config
wave 0
application
wave 1
post-deployment
```

Monitor:

```text
wave completion
hook failure
application health
```

---

# 181. Observability for Hooks

Track:

```text
PreSync
Sync
PostSync
SyncFail
```

A failed hook can block application progression.

---

# 182. Hook Logs

Inspect:

```bash
kubectl get jobs -n roboshop
kubectl describe job <job> -n roboshop
kubectl logs job/<job> -n roboshop
```

---

# 183. Database Migration Observability

For migration Jobs monitor:

```text
job success
duration
failure
database locks
migration version
```

Never hide migration errors behind a successful Argo CD sync.

---

# 184. GitOps and Database Health

Argo CD can report:

```text
Job Synced
```

while the application may still fail because:

```text
migration incomplete
```

Application-level health must therefore include meaningful database readiness where appropriate.

---

# 185. Production Readiness Dashboard

Before allowing automated promotion:

```text
Argo CD = Healthy
Repo Server = Healthy
Prometheus = Healthy
ALB = Healthy
Application = Healthy
No active incidents
```

---

# 186. Progressive Delivery Gate

A production release should not promote when:

```text
monitoring unavailable
```

or:

```text
cluster capacity critically low
```

or:

```text
dependency outage
```

if those conditions affect release confidence.

---

# 187. Observability and Change Windows

During maintenance:

```text
expected alerts
```

should be known.

Document:

```text
start
end
owner
affected systems
rollback
```

---

# 188. GitOps Observability Ownership

Define owners:

```text
Platform team:
Argo CD, EKS, Prometheus

Application team:
service health, business metrics

Security:
audit, access

SRE/Operations:
incident response
```

---

# 189. Production Runbook Metadata

Every runbook should contain:

```text
Purpose
Symptoms
Checks
Commands
Expected results
Escalation
Rollback
Recovery
Post-incident action
```

---

# 190. Example Runbook: Application OutOfSync

```text
Symptom:
Production Application OutOfSync.

1. argocd app get <app>
2. argocd app diff <app>
3. Identify changed resource.
4. Determine whether change is expected.
5. Check recent Git commit.
6. Check controller-owned fields.
7. Check ignoreDifferences.
8. If unintended drift:
   sync/self-heal according to policy.
9. Verify Synced.
10. Record incident if manual change occurred.
```

---

# 191. Example Runbook: Application Degraded

```text
1. argocd app get <app>
2. Identify unhealthy resource.
3. kubectl get pods
4. kubectl describe pod
5. kubectl logs
6. kubectl get events
7. Check Prometheus.
8. Check Grafana.
9. Check ELK.
10. Check ALB.
11. Restore application.
12. Verify Argo CD health.
```

---

# 192. Example Runbook: Cluster Unreachable

```text
1. argocd cluster list
2. Check EKS status.
3. Check API endpoint.
4. Check authentication.
5. Check network path.
6. Check IAM/RBAC.
7. Check cluster credentials.
8. Restore connectivity.
9. Confirm Applications recover.
```

---

# 193. Example Runbook: Repo Server Failure

```text
1. kubectl get pods -n argocd
2. Check repo-server status.
3. Check repo-server logs.
4. Check Git connectivity.
5. Check credentials.
6. Check memory/CPU.
7. Restart only according to approved procedure.
8. Verify manifest generation.
9. Verify Application reconciliation.
```

---

# 194. Example Runbook: Prometheus Scrape Failure

```text
1. Check Prometheus target.
2. Check ServiceMonitor.
3. Check Service.
4. Check endpoint.
5. Check port.
6. Check NetworkPolicy.
7. Check TLS/auth.
8. Check Prometheus logs.
9. Validate query.
```

---

# 195. Example Runbook: ELK Logs Missing

```text
1. Check collector.
2. Check Logstash.
3. Check Elasticsearch.
4. Check index.
5. Check ingestion.
6. Check retention.
7. Verify Kibana query/time range.
```

---

# 196. GitOps Observability Security

Protect:

```text
metrics
logs
dashboards
audit logs
Git metadata
cluster metadata
```

because they can reveal:

```text
architecture
service names
versions
internal endpoints
security events
```

---

# 197. Least Privilege

Observability components should receive only the access required for:

```text
scraping
reading
querying
```

Do not grant broad cluster-admin privileges simply because a monitoring component needs Kubernetes discovery.

---

# 198. Network Segmentation

Consider separate access paths for:

```text
users -> Grafana
Grafana -> Prometheus
Prometheus -> targets
Kibana -> Elasticsearch
Argo CD -> Kubernetes
```

---

# 199. Production Security Boundary

```text
Internet
   |
   v
ALB
   |
   v
Application

Separate management plane:

Operator
   |
   v
SSO
   |
   v
Grafana / Argo CD / Kibana
```

Do not expose management interfaces unnecessarily.

---

# 200. SSO

Use organizational SSO for:

```text
Argo CD
Grafana
Kibana
```

where supported.

Central identity improves:

```text
access control
offboarding
audit
MFA
```

---

# 201. Auditability

Record:

```text
dashboard access
Argo CD actions
Git changes
Kubernetes changes
AWS actions
```

according to security requirements.

---

# 202. Observability Cost

Metrics and logs have different cost profiles.

High volume:

```text
logs
```

can become expensive.

High cardinality:

```text
metrics
```

can become expensive.

Plan:

```text
retention
sampling
aggregation
index lifecycle
```

---

# 203. Cost Optimization

For Prometheus:

```text
reduce unnecessary scrape targets
control cardinality
appropriate retention
remote storage when justified
```

For ELK:

```text
reduce noisy logs
compression
retention policies
tiered storage
```

---

# 204. Observability Capacity Planning

As application count grows:

```text
10 apps
  |
  v
100 apps
  |
  v
500 apps
```

the platform must scale:

```text
Argo CD
Prometheus
Grafana
Elasticsearch
Logstash
```

---

# 205. Argo CD Scaling

Scale components according to workload.

Potential scaling areas:

```text
Application Controller
Repo Server
API Server
ApplicationSet Controller
Redis architecture
```

Do not scale blindly; identify the bottleneck first.

---

# 206. Repo Server Scaling

Repo Server can become a bottleneck when there are many:

```text
Applications
repositories
Helm charts
Kustomize builds
plugins
```

Monitor CPU, memory and generation latency.

---

# 207. Application Controller Scaling

The controller can become busy with:

```text
large application count
many resources
frequent changes
many clusters
```

Observe reconciliation performance before tuning.

---

# 208. Prometheus Scaling

At scale consider:

```text
sharding
remote write
long-term storage
federation
```

depending on architecture.

---

# 209. Elasticsearch Scaling

At scale consider:

```text
data nodes
master nodes
shards
hot/warm/cold architecture
retention
```

---

# 210. Observability Failure Domains

Separate:

```text
application failure
cluster failure
GitOps failure
monitoring failure
logging failure
```

Do not assume they are the same incident.

---

# 211. Example: Argo CD Down

If Argo CD is unavailable:

```text
existing Pods may continue serving
```

but:

```text
new deployments
drift correction
health updates
```

may be affected.

This is a control-plane incident, not necessarily an immediate application outage.

---

# 212. Example: Prometheus Down

Application may continue serving users.

But:

```text
alerts
dashboards
automated progressive analysis
```

can be impaired.

Therefore monitoring itself is part of production reliability.

---

# 213. Example: ELK Down

Application may continue serving users.

But:

```text
log investigation
```

is impaired.

Have alternative emergency diagnostics such as:

```bash
kubectl logs
```

---

# 214. Example: Git Down

Existing workload may continue.

But:

```text
new GitOps changes
```

cannot be pulled.

This demonstrates why runtime availability and deployment-plane availability should be measured separately.

---

# 215. Observability DR

Document how to recover:

```text
Prometheus
Grafana
ELK
Argo CD
```

from backups/infrastructure-as-code where applicable.

---

# 216. Backup Dashboards

Dashboards should be version-controlled when practical.

Do not rely only on manually created Grafana UI configuration.

---

# 217. GitOps for Observability

Observability configuration itself can be GitOps-managed:

```text
ServiceMonitors
PrometheusRules
Grafana dashboards
Alerting configuration
```

This provides:

```text
review
versioning
rollback
audit
```

---

# 218. Observability Repository

Example:

```text
observability/
├── prometheus/
│   ├── servicemonitors/
│   └── rules/
├── grafana/
│   └── dashboards/
├── elk/
│   ├── pipelines/
│   └── policies/
└── argocd/
    └── applications/
```

---

# 219. GitOps-Managed PrometheusRule

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: gitops-production-alerts
  namespace: monitoring
spec:
  groups:
    - name: gitops.rules
      rules:
        - alert: ProductionApplicationDegraded
          expr: |
            # Replace with the exact Argo CD health metric
            # exposed by the installed version.
            <ARGOCD_HEALTH_EXPRESSION>
          for: 10m
          labels:
            severity: critical
            environment: prod
          annotations:
            summary: "Production Argo CD Application is degraded"
            description: "A production Application has remained unhealthy."
```

The placeholder is intentional: validate the actual metric exposed by your Argo CD version before deploying the rule.

---

# 220. Why Avoid Hard-Coded Vendor Queries

Production observability configuration must be version-aware.

A query copied from:

```text
old dashboard
```

may fail after:

```text
Argo CD upgrade
Prometheus upgrade
label changes
```

Always test.

---

# 221. Dashboard as Code

Store dashboards as:

```text
JSON
YAML/provisioning
ConfigMaps
Grafana dashboard resources
```

according to your chosen Grafana deployment model.

---

# 222. Grafana Dashboard Review

Treat dashboards like code:

```text
PR
review
test
deploy
```

This prevents undocumented production dashboard changes.

---

# 223. Alert Rule Review

Every critical alert should have:

```text
owner
runbook
severity
threshold
duration
reason
```

---

# 224. Runbook Link

An alert should ideally point operators to:

```text
runbook
dashboard
Application
cluster
```

so response is faster.

---

# 225. Observability Naming Convention

Example:

```text
prod-argocd-health
prod-roboshop-cart
prod-eks-cluster
prod-alb
```

Consistent naming helps operations.

---

# 226. Application Criticality

Label services:

```text
criticality=critical
criticality=high
criticality=medium
criticality=low
```

Then alert thresholds can be appropriate to business importance.

---

# 227. Critical RoboShop Services

Potentially higher criticality:

```text
payment
orders
user/authentication
inventory
```

because failures can affect core business workflows.

---

# 228. Lower-Criticality Example

A notification component may tolerate:

```text
temporary queue backlog
```

better than:

```text
payment failure
```

Alerting should reflect business impact.

---

# 229. Business Metrics

Technical metrics:

```text
CPU
memory
5xx
latency
```

Business metrics:

```text
orders/min
payment success
checkout completion
```

Business metrics are often the best evidence of actual user impact.

---

# 230. GitOps and Business Observability

A deployment can be:

```text
technically healthy
```

but:

```text
orders decreased
```

This should trigger investigation.

---

# 231. Canary Business Analysis

For payment:

```text
canary payment success
```

should be compared against:

```text
stable payment success
```

where traffic and sample size permit meaningful comparison.

---

# 232. Synthetic Monitoring

Synthetic tests can periodically verify:

```text
login
catalog
cart
checkout
```

This is useful when organic traffic is low.

---

# 233. Synthetic + Canary

A powerful pattern:

```text
5% canary
 |
 v
synthetic transaction
 +
Prometheus metrics
```

This combines:

```text
real traffic
+
controlled validation
```

---

# 234. Synthetic Failure

If synthetic checkout fails:

```text
abort or stop promotion
```

if the release policy defines it as a blocking signal.

---

# 235. Observability and Progressive Delivery

The relationship is:

```text
Progressive delivery
       |
       v
needs evidence
       |
       v
Observability
       |
       +--> Metrics
       +--> Logs
       +--> Events
       +--> Business signals
```

Without observability, progressive delivery becomes gradual deployment rather than evidence-based delivery.

---

# 236. Observability and GitOps

GitOps gives:

```text
desired state
```

Observability gives:

```text
evidence of actual behavior
```

Together:

```text
Git = what should happen
Kubernetes = what is running
Observability = how it behaves
```

---

# 237. Complete Mental Model

```text
                  Git
                   |
                   v
                Argo CD
                   |
          desired state
                   |
                   v
                EKS
                   |
        +----------+----------+
        |          |          |
        v          v          v
      ALB       Pods       Services
        |          |
        |          v
        |      Prometheus
        |          |
        v          v
      Users     Grafana
                   |
                   v
                  ELK
```

---

# 238. Production GitOps Observability Stack

```text
Git
 |
 +--> CI status
 |
 v
Argo CD
 |
 +--> Applications
 +--> ApplicationSets
 +--> Clusters
 +--> Repositories
 |
 v
Kubernetes / EKS
 |
 +--> Pods
 +--> Services
 +--> Ingress / ALB
 +--> HPA
 +--> Nodes
 |
 +--> Prometheus
 |      |
 |      v
 |    Grafana
 |
 +--> ELK
```

---

# 239. Recommended RoboShop Dashboard

Create one dashboard with:

```text
Environment selector
Cluster selector
Application selector
```

Then:

```text
Argo CD sync
Argo CD health
Rollout state
ALB traffic
HTTP errors
Latency
CPU
Memory
Pod restarts
Logs
```

---

# 240. Production Dashboard Example

```text
================================================
ROBOSHOP PRODUCTION GITOPS
================================================

Argo CD
Synced: 8
OutOfSync: 0
Degraded: 0

Rollouts
Running: 1
Aborted: 0

ALB
Requests: 1,250/s
Target 5xx: 0.15%
P95: 210ms

Kubernetes
Pending Pods: 0
CrashLoop: 0
OOMKilled: 0

Business
Orders/min: 420
Payment Success: 99.8%
================================================
```

---

# 241. Production Alert Example

```text
ALERT:
Production cart Application degraded

Environment: prod
Cluster: eks-prod-a
Application: cart

Check:
Argo CD
Grafana
ELK
Rollout
```

---

# 242. Production Deployment Marker

```text
2026-08-19 14:00
cart v2.4.1
Git SHA: abc123
Image: sha256:...
Canary: 5%
```

Then:

```text
14:05 analysis PASS
14:10 25%
14:20 analysis PASS
14:30 50%
14:45 analysis PASS
15:00 100%
```

---

# 243. Incident Timeline Example

```text
15:00 release v2.4.1
15:05 canary 5%
15:10 5xx rises
15:11 alert
15:12 AnalysisRun fails
15:12 rollout aborts
15:14 stable verified
15:20 root cause identified
15:40 fix committed
```

This is the operational value of progressive delivery plus observability.

---

# 244. Production Metrics Review

Weekly review:

```text
deployment frequency
lead time
failed deployments
rollout aborts
drift incidents
sync failures
MTTR
application SLOs
```

---

# 245. GitOps Platform Review

Monthly:

```text
Argo CD resource count
cluster count
ApplicationSet count
repo count
controller utilization
Repo Server utilization
Prometheus cardinality
ELK storage
alert quality
```

---

# 246. Observability Capacity Review

Ask:

```text
Are scrape intervals appropriate?
Are there unnecessary metrics?
Are logs too noisy?
Is Elasticsearch storage sufficient?
Are Argo CD controllers scaling?
Are dashboards still useful?
```

---

# 247. Upgrade Observability

Before upgrading Argo CD:

```text
capture current metrics
capture dashboards
validate metric names
test alert rules
test dashboards
test Application health
```

---

# 248. Upgrade Failure Scenario

After Argo CD upgrade:

```text
dashboard shows no data
```

Possible reason:

```text
metric name changed
label changed
```

Therefore observability should be tested as part of platform upgrades.

---

# 249. Upgrade Runbook

```text
1. Record current version.
2. Review release notes.
3. Check metric changes.
4. Test in non-prod.
5. Validate dashboards.
6. Validate alerts.
7. Upgrade.
8. Confirm components.
9. Confirm Applications.
10. Confirm metrics.
```

---

# 250. Disaster Recovery Observability

After rebuilding:

```text
Argo CD healthy
Prometheus scraping
Grafana data source healthy
ELK ingestion healthy
Applications synced
Applications healthy
ALB healthy
```

must all be validated.

---

# 251. Observability Backup

Back up:

```text
Grafana dashboards
Prometheus rules
Alerting configuration
Argo CD configuration
ELK configuration
GitOps manifests
```

where supported by the deployment architecture.

---

# 252. Observability Testing

Test intentionally:

```text
break repository access
break cluster access
break Prometheus scrape
break Pod
break ALB target
fail canary
```

Then confirm alerts and runbooks.

---

# 253. Alert Testing

An alert that has never been tested is not a reliable control.

Use:

```text
staging
synthetic failures
controlled tests
```

rather than waiting for production incidents.

---

# 254. Observability Definition of Done

A production service is not fully ready until:

```text
metrics exist
logs exist
dashboards exist
alerts exist
runbook exists
Git revision traceable
deployment visible
business health visible
```

---

# 255. GitOps Definition of Done

A GitOps deployment is not fully observable until:

```text
Git change traceable
Argo CD sync traceable
resource health visible
rollout visible
application metrics visible
logs searchable
failure alerts configured
```

---

# 256. Interview: What Do You Monitor in Argo CD?

### Answer

> I monitor Argo CD component availability, Application sync and health status, reconciliation latency, sync failures, repository and cluster connectivity, ApplicationSet errors, and resource utilization. I also connect these signals to Kubernetes, ALB and application metrics.

---

# 257. Interview: Synced vs Healthy?

### Answer

> Synced means the cluster matches the desired state from Git. Healthy means the resources are considered operational according to Argo CD health evaluation. An Application can be Synced but Degraded.

---

# 258. Interview: How Do You Monitor OutOfSync?

### Answer

> I expose Argo CD Application sync state to Prometheus, visualize it in Grafana, and alert on sustained production OutOfSync conditions while excluding expected deployment windows and controller-owned differences.

---

# 259. Interview: How Do You Troubleshoot a Degraded Application?

### Answer

> I first use `argocd app get` to identify the unhealthy resource, then inspect the Kubernetes resource, Pods, events, logs and metrics. For external access I check the ALB and Service endpoints. I use Grafana for metrics and ELK for detailed logs.

---

# 260. Interview: How Do You Monitor Multi-Cluster Argo CD?

### Answer

> I monitor Argo CD cluster connection status, Application health and sync status per cluster, and then monitor each EKS cluster's Kubernetes, ALB and application health. I use labels for environment, region and account so production failures can be isolated quickly.

---

# 261. Interview: Why Prometheus for Argo CD?

### Answer

> Prometheus provides time-series metrics that can be aggregated, alerted on and visualized in Grafana. It is useful for tracking Argo CD operations as well as Kubernetes and application behavior.

---

# 262. Interview: Why ELK if Prometheus Exists?

### Answer

> Prometheus is primarily metrics-oriented. ELK provides detailed searchable logs, which are critical for root-cause analysis. For example, a Prometheus alert can show increased 5xx errors while ELK can reveal the exact application exception.

---

# 263. Interview: Why Grafana?

### Answer

> Grafana provides a common visualization layer for Prometheus metrics and helps correlate platform, Kubernetes, ALB, rollout and application signals in one operational view.

---

# 264. Interview: What Are the Four Golden Signals?

### Answer

> Latency, traffic, errors and saturation. I use them as a baseline for service observability and then add business-specific metrics.

---

# 265. Interview: How Do You Monitor a Canary?

### Answer

> I compare canary and stable metrics for error rate, latency, traffic and relevant business KPIs. Prometheus provides the metrics, Grafana provides visualization, and Argo Rollouts can use Prometheus analysis to automatically promote or abort.

---

# 266. Interview: What If Metrics Show Healthy but Users Report Failures?

### Answer

> I investigate whether the metric is measuring the correct user path. I check ALB behavior, logs, business metrics, synthetic checks and dependency health. A technically healthy Pod does not guarantee a healthy user experience.

---

# 267. Interview: How Do You Prevent False Alerts?

### Answer

> I use appropriate thresholds, sustained durations, sufficient sample sizes, environment filtering, alert grouping and maintenance silences. I also review alerts after incidents to improve signal quality.

---

# 268. Interview: What Is High Cardinality?

### Answer

> High cardinality means a metric has a very large number of unique label combinations. It can increase Prometheus memory usage and query cost. I avoid labels such as unbounded user IDs or request IDs.

---

# 269. Interview: How Do You Monitor ALB?

### Answer

> I monitor request count, ALB and target 5xx, target response time, healthy target count and unhealthy target count. I correlate these with Kubernetes Pod readiness and application metrics.

---

# 270. Interview: How Do You Correlate a Git Change to an Incident?

### Answer

> I identify the Git commit and image digest, check when Argo CD synchronized that revision, inspect the Rollout timeline, correlate it with Grafana metrics and then use ELK to inspect application logs around the same timestamp.

---

# 271. Interview: What Happens if Argo CD Is Down?

### Answer

> Existing workloads can generally continue serving traffic because Argo CD is the deployment control plane rather than the runtime data plane. However, new synchronization, drift correction and health updates can be affected. I monitor Argo CD separately from application availability.

---

# 272. Interview: What Happens if Prometheus Is Down?

### Answer

> Applications may continue serving, but metrics, alerting and automated progressive-delivery analysis can be impaired. Critical releases should avoid blindly promoting when the required health evidence is unavailable.

---

# 273. Interview: What Happens if Git Is Down?

### Answer

> Existing workloads can continue running, but Argo CD cannot reliably obtain new desired state. Once Git access is restored, reconciliation can continue.

---

# 274. Interview: How Do You Monitor ApplicationSet?

### Answer

> I monitor ApplicationSet controller health and generator errors, compare expected and generated Applications, and inspect the ApplicationSet resource and controller logs when Applications are missing.

---

# 275. Interview: How Do You Monitor Repo Server?

### Answer

> I monitor availability, CPU and memory, repository operation failures, manifest generation latency and errors. During failures I inspect repo-server logs and validate repository credentials and Helm/Kustomize rendering.

---

# 276. Interview: What Should a GitOps Dashboard Contain?

### Answer

> At minimum: Application sync and health, cluster connectivity, repository health, sync failures, reconciliation latency, Rollout status, Kubernetes health, ALB health and application golden signals.

---

# 277. Interview: How Do You Monitor Production Drift?

### Answer

> I track sustained OutOfSync Applications and correlate them with Git changes and manual cluster activity. For unexpected drift, I use Argo CD diff and Kubernetes audit information to identify the source.

---

# 278. Interview: How Do You Handle Observability During an Argo CD Upgrade?

### Answer

> I validate metric names, labels, dashboards and alert rules in a non-production environment before upgrading production. After the upgrade I verify component health, Application reconciliation and Prometheus scraping.

---

# 279. Interview: How Do You Design Observability for 100+ Applications?

### Answer

> I use consistent labels, centralized dashboards with variables, application-level metrics, environment and cluster dimensions, standardized alert rules, and platform-level aggregation. I also monitor Argo CD controller and Repo Server capacity.

---

# 280. Interview: What Is an Observability SLO for GitOps?

### Answer

> It can be a measurable reliability target for the GitOps control plane, such as the percentage of approved changes reconciled within a defined time or the availability of Argo CD. The SLO should reflect business and operational requirements.

---

# 281. Senior Scenario: All Applications Become OutOfSync

Investigate:

```text
Argo CD controller
Kubernetes API
repository revision
controller mutations
CRD changes
```

Do not manually sync 100 applications before identifying the common cause.

---

# 282. Senior Scenario: All Metrics Disappear

Investigate:

```text
Prometheus
ServiceMonitors
network
targets
storage
```

This is likely a monitoring-platform incident rather than an application incident.

---

# 283. Senior Scenario: Only One Service Has No Metrics

Check:

```text
ServiceMonitor
PodMonitor
labels
metrics endpoint
port
application instrumentation
```

---

# 284. Senior Scenario: Logs Missing After Deployment

Compare:

```text
old Pods
new Pods
collector
labels
index
```

A new version may have changed logging format or labels.

---

# 285. Senior Scenario: Dashboard Says Healthy but ALB Says Unhealthy

Trace:

```text
Pod
 |
 v
Readiness
 |
 v
Service
 |
 v
Target registration
 |
 v
ALB health check
```

Health at one layer does not prove health at the next.

---

# 286. Senior Scenario: Canary Analysis Passes but Orders Drop

This indicates:

```text
technical health != business health
```

Add or investigate:

```text
orders/min
checkout success
payment success
```

---

# 287. Senior Scenario: High Error Rate but No Recent Deployment

Investigate:

```text
dependency
database
ALB
traffic spike
node issue
external provider
```

Do not blame GitOps simply because the system uses GitOps.

---

# 288. Senior Scenario: Sync Is Successful but Pods Are Old

Check:

```text
imagePullPolicy
image digest
Deployment/Rollout revision
Pod template hash
Argo CD diff
```

The manifest may have synced but the expected image change may not actually be present.

---

# 289. Senior Scenario: Git Says v2, Cluster Says v1

Check:

```text
Argo CD sync
Application status
target revision
path
Helm values
generated manifest
```

The wrong environment values file is a common source of confusion.

---

# 290. Senior Scenario: Argo CD Synced Wrong Environment

Check:

```text
Application destination
ApplicationSet template
cluster labels
Helm values
Git path
```

This is a high-severity GitOps governance problem.

---

# 291. Senior Scenario: Production Alert Fires During Planned Deployment

Check:

```text
Was the alert expected?
Is deployment window known?
Is the application actually degraded?
```

Do not silence alerts permanently to hide deployment noise.

---

# 292. Senior Scenario: Prometheus Query Is Slow

Investigate:

```text
high cardinality
wide time range
expensive joins
high-frequency metrics
```

Optimize the query and metric design.

---

# 293. Senior Scenario: Elasticsearch Is Filling Disk

Immediate concerns:

```text
disk watermark
index allocation
ingestion
retention
```

Long-term:

```text
ILM
log reduction
capacity planning
```

---

# 294. Senior Scenario: Argo CD Repo Server CPU High

Check:

```text
repository count
Application count
manifest generation
Helm
Kustomize
plugins
large repositories
```

Scale after identifying the workload pattern.

---

# 295. Senior Scenario: Application Controller CPU High

Check:

```text
Application count
resource count
reconciliation frequency
API latency
cluster count
```

Then tune architecture.

---

# 296. Senior Scenario: ApplicationSet Controller High CPU

Check:

```text
generator frequency
Git generator
cluster generator
number of generated Applications
template complexity
```

---

# 297. Production Observability Checklist

```text
[ ] Git health
[ ] CI health
[ ] ECR availability
[ ] Argo CD API
[ ] Application Controller
[ ] Repo Server
[ ] ApplicationSet Controller
[ ] Redis
[ ] Application sync
[ ] Application health
[ ] Cluster connections
[ ] Repository connections
[ ] Kubernetes API
[ ] Nodes
[ ] Pods
[ ] Services
[ ] Ingress
[ ] ALB
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] Alerts
[ ] Audit
[ ] Runbooks
```

---

# 298. Production Observability Architecture

```text
                         Git
                          |
                          v
                  Jenkins / GitHub Actions
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
        +-----------------+------------------+
        |                 |                  |
        v                 v                  v
   Applications     ApplicationSets      Clusters
        |
        v
   Argo Rollouts
        |
        v
       EKS
        |
   +----+----+----------------+
   |         |                |
   v         v                v
  ALB      Pods            Services
   |         |
   |         v
   |     Prometheus
   |         |
   |         v
   |      Grafana
   |
   v
 Users

Pods / Argo CD / Kubernetes
        |
        v
       ELK
```

---

# 299. Final Operational Mental Model

Remember these relationships:

```text
Git
=
desired state

Argo CD
=
reconciliation

Kubernetes
=
runtime state

Prometheus
=
metrics

Grafana
=
visualization

ELK
=
logs

Argo Rollouts
=
progressive delivery

ALB
=
external traffic path
```

---

# 300. Final Principle

A mature GitOps platform does not stop at:

```text
Git commit -> Argo CD -> Kubernetes
```

It continues:

```text
Git commit
   |
   v
Argo CD
   |
   v
Kubernetes
   |
   v
Application
   |
   v
Metrics + Logs + Events
   |
   v
Grafana / ELK
   |
   v
Alerts
   |
   v
Operator action
```

The strongest production model is:

```text
Desired state
+
Reconciliation
+
Health
+
Metrics
+
Logs
+
Alerts
+
Audit
+
Runbooks
```

That is the foundation of observable GitOps.

---

# 301. Final RoboShop Mental Model

For the RoboShop platform:

```text
Developer
   |
   v
Git
   |
   v
Jenkins / GitHub Actions
   |
   +--> Maven / npm / tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker Image
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
   +--> ALB
   +--> Services
   +--> Pods
   |
   +--> Prometheus
   |       |
   |       v
   |     Grafana
   |
   +--> ELK
           |
           v
       Troubleshooting
```

When something goes wrong:

```text
Git
 -> Argo CD
 -> Kubernetes
 -> ALB
 -> Pod
 -> Prometheus
 -> Grafana
 -> ELK
```

trace the request and desired state through every layer.

---

# 302. Interview Summary

Be able to explain:

```text
1. What GitOps observability means.
2. How to monitor Argo CD.
3. How to monitor Applications.
4. Synced vs Healthy.
5. OutOfSync detection.
6. Reconciliation monitoring.
7. Repo Server monitoring.
8. Application Controller monitoring.
9. ApplicationSet monitoring.
10. Multi-cluster monitoring.
11. Prometheus integration.
12. Grafana dashboards.
13. ELK troubleshooting.
14. Kubernetes events.
15. ALB monitoring.
16. EKS monitoring.
17. Golden signals.
18. DORA metrics.
19. GitOps SLOs.
20. Progressive-delivery observability.
21. Canary analysis.
22. Business metrics.
23. High cardinality.
24. Alert design.
25. Incident correlation.
26. Git audit.
27. Kubernetes audit.
28. Disaster recovery.
29. Monitoring the monitoring stack.
30. Production troubleshooting.
```

---

# 303. Final Takeaway

The most important concept is:

> **GitOps tells you what the system should be. Kubernetes tells you what is running. Observability tells you how the system is actually behaving.**

A production-grade GitOps platform needs all three.

```text
              GIT
               |
        Desired State
               |
               v
            Argo CD
               |
        Reconciliation
               |
               v
            EKS
               |
       Actual Runtime
               |
       +-------+-------+
       |               |
       v               v
   Prometheus         ELK
       |               |
       v               v
    Grafana         Logs/Search
       |
       +-------+
               |
               v
            Alerts
               |
               v
            Operators
```

The final goal is not merely:

```text
"Argo CD says Synced."
```

The real production question is:

```text
"Is the desired state deployed,
is the workload healthy,
is traffic healthy,
are users successful,
and can we prove it through metrics and logs?"
```
