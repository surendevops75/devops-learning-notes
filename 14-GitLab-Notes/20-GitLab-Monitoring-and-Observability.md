# GitLab Monitoring and Observability

> Production-oriented guide to monitoring GitLab itself and the complete CI/CD platform: GitLab application health, runners, pipelines, jobs, APIs, webhooks, repositories, artifacts, registries, Kubernetes/EKS deployments, ArgoCD, Prometheus, Grafana, ELK, logs, metrics, alerting, SLOs, capacity planning, troubleshooting and senior DevOps interview scenarios.

---

## 1. What Is GitLab Observability?

GitLab observability means understanding the health and behavior of:

```text
GitLab
Runners
CI/CD
APIs
Webhooks
Artifacts
Container Registry
AWS
Kubernetes
ArgoCD
Applications
```

---

## 2. Why GitLab Observability Matters

A CI/CD platform is production infrastructure.

If GitLab fails:

```text
code delivery
security validation
deployments
releases
```

may stop.

---

## 3. Three Pillars

Observability commonly uses:

```text
Metrics
Logs
Traces
```

For the user's DevOps stack, the primary monitoring/logging tools are:

```text
Prometheus
Grafana
ELK Stack
```

---

## 4. Metrics

Metrics answer:

```text
How much?
How often?
How long?
How many?
```

Examples:

```text
pipeline duration
runner utilization
API latency
CPU
memory
```

---

## 5. Logs

Logs answer:

```text
What happened?
What error occurred?
What context was available?
```

---

## 6. Traces

Distributed tracing can answer:

```text
Where did a request spend time?
Which service caused latency?
```

Tracing is a separate observability capability and is not required for every GitLab monitoring setup.

---

## 7. GitLab Monitoring Layers

Monitor at multiple layers:

```text
Infrastructure
 ↓
GitLab application
 ↓
API
 ↓
Runner
 ↓
Pipeline
 ↓
Deployment
 ↓
Kubernetes
 ↓
Application
```

---

## 8. Infrastructure Layer

Monitor:

```text
CPU
memory
disk
network
filesystem
load
```

---

## 9. GitLab Application Layer

Monitor:

```text
request rate
latency
errors
background jobs
database
cache
```

---

## 10. CI/CD Layer

Monitor:

```text
pipeline success
pipeline duration
queue time
job failures
runner capacity
```

---

## 11. Deployment Layer

Monitor:

```text
deployment frequency
deployment failures
rollbacks
environment health
```

---

## 12. Kubernetes Layer

Monitor:

```text
Pod health
Node health
CPU
memory
restarts
events
```

---

## 13. Application Layer

Monitor:

```text
availability
latency
error rate
business health
```

---

## 14. Prometheus

Prometheus is commonly used for metrics collection and alerting.

Concept:

```text
Targets
 ↓
Prometheus
 ↓
PromQL
 ↓
Grafana
```

---

## 15. Grafana

Grafana visualizes metrics and dashboards.

Typical dashboards:

```text
GitLab
Runners
Kubernetes
Applications
AWS infrastructure
```

---

## 16. ELK Stack

The ELK stack provides centralized log management:

```text
Elasticsearch
Logstash
Kibana
```

---

## 17. ELK Flow

Concept:

```text
Logs
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

---

## 18. Metrics vs Logs

Metrics:

```text
CPU = 80%
```

Logs:

```text
deployment failed: image pull error
```

Use both.

---

## 19. Golden Signals

Four common service signals:

```text
Latency
Traffic
Errors
Saturation
```

---

## 20. Latency

For GitLab:

```text
API response time
pipeline scheduling delay
job startup time
```

---

## 21. Traffic

Examples:

```text
HTTP requests
API requests
webhook events
pipeline volume
```

---

## 22. Errors

Track:

```text
HTTP 4xx
HTTP 5xx
failed jobs
failed pipelines
deployment failures
```

---

## 23. Saturation

Examples:

```text
CPU
memory
disk
runner slots
database connections
```

---

## 24. GitLab Availability

Define what availability means.

Example:

```text
users can access GitLab
API works
CI can execute
```

---

## 25. GitLab SLI

A Service Level Indicator measures actual performance.

Examples:

```text
successful API requests / total requests
```

---

## 26. GitLab SLO

An SLO defines the target.

Example:

```text
99.9% successful API availability
```

Actual target should be based on organizational requirements.

---

## 27. Error Budget

If the SLO is 99.9%, the remaining budget represents allowable unavailability/error within the defined measurement period.

---

## 28. Why Error Budgets Matter

They help balance:

```text
reliability
vs
delivery speed
```

---

## 29. Pipeline SLI

Possible pipeline SLI:

```text
successful pipelines / total pipelines
```

---

## 30. Pipeline SLO

Example:

```text
95% of normal pipelines complete successfully
```

The exact target depends on pipeline type and organization.

---

## 31. Pipeline Duration

Measure:

```text
P50
P90
P95
P99
```

rather than only averages.

---

## 32. Why Percentiles?

An average can hide slow outliers.

P95 shows the experience of slower executions.

---

## 33. Queue Time

Measure:

```text
job created
 ↓
runner starts job
```

This reveals runner capacity problems.

---

## 34. Execution Time

Measure:

```text
runner starts
 ↓
job completes
```

---

## 35. Total Pipeline Time

```text
queue
+
execution
+
dependency waiting
```

Analyze each component separately.

---

## 36. Pipeline Critical Path

DAG pipelines reduce unnecessary waiting.

Monitor critical-path jobs.

---

## 37. Runner Utilization

Track:

```text
active jobs
idle runners
queued jobs
CPU
memory
```

---

## 38. Runner Queue

A growing queue can indicate:

```text
insufficient runners
slow jobs
runner failure
tag mismatch
```

---

## 39. Runner Availability

Monitor:

```text
online/offline
healthy/unhealthy
job pickup time
```

---

## 40. Runner Failure Rate

Track failed jobs by runner.

A single unhealthy runner may cause repeated failures.

---

## 41. Runner Version

Track runner versions and upgrade status.

Outdated runners can create:

```text
compatibility
security
performance
```

issues.

---

## 42. Runner Tags

Monitor whether jobs are stuck because no runner matches the required tags.

---

## 43. Protected Runners

Track production/deployment runners separately.

---

## 44. Runner Capacity Planning

Estimate:

```text
average concurrent jobs
peak concurrent jobs
job duration
required runner capacity
```

---

## 45. Autoscaling Runners

Kubernetes-based runners can dynamically create execution Pods.

Monitor:

```text
scale-up latency
Pod startup time
job queue
node capacity
```

---

## 46. Runner Startup Latency

If autoscaling is slow:

```text
job queue increases
```

Even if total runner capacity is sufficient.

---

## 47. GitLab API Monitoring

Track:

```text
request rate
latency
status codes
5xx
429
```

---

## 48. API Error Rate

A sudden increase in:

```text
5xx
```

may indicate GitLab infrastructure problems.

A sudden increase in:

```text
401/403
```

may indicate authentication/authorization changes.

---

## 49. API Rate Limits

Track:

```text
429 responses
```

to identify excessive automation.

---

## 50. API Latency

Monitor:

```text
P50
P95
P99
```

for important API endpoints.

---

## 51. Webhook Monitoring

Track:

```text
event count
delivery success
delivery latency
failure count
retry count
```

---

## 52. Webhook Failure

A webhook failure can silently break:

```text
automation
notifications
deployment workflows
```

---

## 53. Webhook Queue

If asynchronous:

```text
queue depth
oldest event age
worker processing time
```

are important.

---

## 54. Webhook Duplicate Events

Monitor duplicate processing.

Unexpected duplicates may indicate:

```text
retry behavior
consumer failure
deduplication issue
```

---

## 55. GitLab Background Jobs

Background jobs can affect:

```text
notifications
repository operations
CI processing
maintenance
```

Monitor queue depth and processing latency where metrics are available.

---

## 56. Database Monitoring

GitLab depends heavily on database health.

Monitor:

```text
CPU
memory
connections
latency
locks
storage
replication
```

---

## 57. Database Connection Saturation

If connections are exhausted:

```text
requests fail
background work slows
```

---

## 58. Database Query Latency

Slow queries can cause:

```text
API latency
background job delay
pipeline UI slowness
```

---

## 59. Database Storage

Monitor:

```text
free space
growth rate
IOPS
latency
```

---

## 60. Database Backups

Monitor:

```text
backup success
backup duration
backup age
restore tests
```

---

## 61. Redis/Cache Layer

Where GitLab uses caching components, monitor:

```text
memory
connections
latency
evictions
availability
```

---

## 62. Cache Saturation

Cache problems can increase:

```text
database load
application latency
```

---

## 63. Object Storage

GitLab may use object storage for suitable data such as artifacts or uploads depending on configuration.

Monitor:

```text
availability
latency
storage
errors
```

---

## 64. Artifact Storage

Track:

```text
artifact volume
retention
storage growth
upload failures
download failures
```

---

## 65. Artifact Expiration

Use retention policies to control storage growth.

---

## 66. Container Registry

Monitor:

```text
image push
image pull
storage
request errors
latency
```

---

## 67. Registry Storage Growth

Old images and tags can consume significant storage.

Use controlled cleanup policies.

---

## 68. Image Pull Failure

Track deployment failures caused by:

```text
image not found
authentication
network
registry outage
```

---

## 69. ECR Monitoring

For AWS-based environments monitor:

```text
image push/pull
repository access
storage
authentication
```

through appropriate AWS monitoring sources.

---

## 70. GitLab Disk Usage

Monitor:

```text
filesystem
logs
repositories
artifacts
registry
backups
```

---

## 71. Disk Full Incident

Symptoms:

```text
Git operations fail
logs stop writing
database errors
GitLab becomes unstable
```

---

## 72. Disk Alert Thresholds

Use multiple levels:

```text
warning
critical
```

and consider growth rate, not only current percentage.

---

## 73. Inode Usage

A filesystem can run out of inodes even when free disk space remains.

Monitor both:

```text
disk %
inode %
```

---

## 74. CPU Saturation

High CPU can increase:

```text
API latency
background job delay
pipeline operations
```

---

## 75. Memory Pressure

Memory pressure can cause:

```text
OOM
swapping
latency
process restarts
```

---

## 76. Swap

Monitor swap behavior where applicable.

Unexpected swap activity can indicate memory pressure.

---

## 77. Network Saturation

Monitor:

```text
throughput
packet errors
connections
latency
```

---

## 78. Load Balancer

For GitLab behind a load balancer monitor:

```text
requests
latency
5xx
target health
connections
```

---

## 79. Reverse Proxy

Monitor:

```text
upstream latency
connection errors
TLS errors
5xx
```

---

## 80. TLS Monitoring

Track:

```text
certificate expiry
TLS handshake errors
```

---

## 81. Certificate Expiration

Create proactive alerts before expiration.

---

## 82. DNS Monitoring

Check:

```text
resolution
latency
correct records
```

---

## 83. GitLab Health Endpoint

Use supported health/readiness endpoints where available.

Health checks should verify the dependency level appropriate for the purpose.

---

## 84. Liveness vs Readiness

Liveness:

```text
process should restart?
```

Readiness:

```text
should receive traffic?
```

---

## 85. GitLab Container Health

For containerized GitLab components, monitor:

```text
container status
restart count
resource usage
health checks
```

---

## 86. Kubernetes GitLab Deployment

If GitLab is deployed on Kubernetes, monitor:

```text
Pods
Deployments
StatefulSets
PVCs
Services
Ingress
```

---

## 87. Kubernetes Pod Restarts

Frequent restarts may indicate:

```text
OOMKilled
probe failure
application crash
node issue
```

---

## 88. Kubernetes Events

Use:

```bash
kubectl get events --sort-by=.lastTimestamp
```

during troubleshooting.

---

## 89. Node Health

Monitor:

```text
CPU
memory
disk pressure
PID pressure
network
```

---

## 90. PVC Monitoring

Persistent volume problems can break:

```text
repositories
database
artifacts
```

depending on architecture.

---

## 91. Kubernetes Ingress

Monitor:

```text
5xx
latency
TLS
backend health
```

---

## 92. GitLab Application Logs

Centralize logs into ELK where appropriate.

---

## 93. Log Levels

Typical:

```text
DEBUG
INFO
WARN
ERROR
```

Production should avoid excessive DEBUG logging.

---

## 94. Structured Logs

Prefer fields such as:

```text
timestamp
level
service
request_id
status
duration
```

---

## 95. Request ID

A request identifier helps correlate:

```text
load balancer
GitLab
backend
database
```

logs.

---

## 96. Log Correlation

Example:

```text
request_id=abc123
```

appears across related logs.

---

## 97. ELK Index Strategy

Organize logs logically:

```text
gitlab
runner
proxy
kubernetes
application
```

depending on the logging architecture.

---

## 98. Log Retention

Balance:

```text
troubleshooting
audit
storage cost
privacy
```

---

## 99. Sensitive Logs

Never allow:

```text
passwords
tokens
private keys
secret values
```

into centralized logs.

---

## 100. Log Scrubbing

Use filtering/redaction where required.

---

## 101. Kibana Investigation

A typical investigation:

```text
time range
 ↓
service
 ↓
error
 ↓
request ID
 ↓
related events
```

---

## 102. Prometheus Scraping

Prometheus collects metrics from configured targets.

Concept:

```text
Target
 ↓
/metrics
 ↓
Prometheus
```

---

## 103. Exporters

Exporters expose metrics from systems that do not natively expose Prometheus metrics.

Examples:

```text
node exporter
database exporter
```

Use only the exporters appropriate to the environment.

---

## 104. Node Exporter

Provides host metrics such as:

```text
CPU
memory
disk
network
```

---

## 105. Kubernetes Metrics

Kubernetes monitoring commonly uses metrics from:

```text
kubelet
kube-state-metrics
node exporter
```

depending on the architecture.

---

## 106. kube-state-metrics

Provides Kubernetes object-state metrics.

Examples:

```text
Deployment replicas
Pod state
DaemonSet state
```

---

## 107. PromQL

PromQL queries Prometheus metrics.

Concept:

```promql
rate(http_requests_total[5m])
```

---

## 108. Request Rate

Example:

```promql
sum(rate(http_requests_total[5m]))
```

Metric names vary by application/exporter.

---

## 109. Error Rate

Concept:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

Adapt labels to the actual metric schema.

---

## 110. CPU Usage

Use appropriate node/container metrics to calculate CPU consumption.

---

## 111. Memory Usage

Monitor:

```text
working set
RSS
container memory
node memory
```

according to the metric source.

---

## 112. Pod Restart Rate

A rising restart rate is an important Kubernetes signal.

---

## 113. OOMKilled Detection

Correlate:

```text
container termination reason
+
memory metrics
+
logs
```

---

## 114. Disk Alert

Concept:

```promql
(node_filesystem_avail_bytes /
 node_filesystem_size_bytes) < threshold
```

Tune threshold and exclude irrelevant filesystems.

---

## 115. Alert Rules

Prometheus alert rules convert conditions into alerts.

---

## 116. Warning Alert

Example concept:

```text
disk > 80%
```

---

## 117. Critical Alert

Example concept:

```text
disk > 90%
```

Thresholds should reflect actual risk and filesystem growth.

---

## 118. Alert Duration

Use a condition duration where appropriate to avoid alerts from short spikes.

---

## 119. Alert Flapping

An alert that repeatedly:

```text
fires
resolves
fires
```

creates noise.

Tune:

```text
threshold
duration
hysteresis
```

where supported.

---

## 120. Alert Severity

Example:

```text
info
warning
critical
```

---

## 121. Alert Labels

Useful labels:

```text
service
environment
severity
team
region
```

---

## 122. Alert Routing

Route alerts by:

```text
team
severity
environment
service
```

---

## 123. Production Alerts

Production alerts should be:

```text
actionable
specific
owned
```

---

## 124. Alert Fatigue

Too many low-value alerts cause engineers to ignore important alerts.

---

## 125. Symptom vs Cause

Prefer alerting on user-impacting symptoms where practical.

Example:

```text
availability down
```

may be more actionable than:

```text
CPU high
```

alone.

---

## 126. Multi-Condition Alerts

Correlate:

```text
high error rate
+
high latency
```

when it improves signal quality.

---

## 127. SLO-Based Alerting

Alert when the service is consuming its error budget too quickly rather than relying only on infrastructure thresholds.

---

## 128. GitLab Pipeline Dashboard

A useful dashboard includes:

```text
pipeline success rate
pipeline duration
queue time
failed jobs
```

---

## 129. Runner Dashboard

Include:

```text
active runners
queued jobs
job duration
CPU
memory
offline runners
```

---

## 130. API Dashboard

Include:

```text
request rate
P95 latency
5xx
4xx
429
```

---

## 131. Infrastructure Dashboard

Include:

```text
CPU
memory
disk
network
load
```

---

## 132. Registry Dashboard

Include:

```text
push failures
pull failures
storage
latency
```

---

## 133. Kubernetes Dashboard

Include:

```text
nodes
Pods
restarts
CPU
memory
pending Pods
events
```

---

## 134. Deployment Dashboard

Include:

```text
deployments
success
failure
rollback
current version
```

---

## 135. GitOps Dashboard

Include:

```text
ArgoCD sync
health
out-of-sync applications
failed syncs
```

---

## 136. Application Dashboard

Include:

```text
traffic
errors
latency
saturation
business indicators
```

---

## 137. Dashboard Hierarchy

Use:

```text
Executive/Platform
 ↓
Environment
 ↓
Service
 ↓
Instance/Pod
```

---

## 138. Dashboard Design

Avoid putting every metric on one dashboard.

Use focused dashboards.

---

## 139. Dashboard Time Range

Make common ranges easy:

```text
15m
1h
6h
24h
7d
```

---

## 140. Deployment Markers

Mark deployments on dashboards.

This makes correlation easier:

```text
deployment
 ↓
metric change
```

---

## 141. Change Correlation

When errors increase after deployment:

```text
check release
 ↓
check image digest
 ↓
check logs
 ↓
compare metrics
```

---

## 142. GitLab Release Correlation

Track:

```text
commit
pipeline
release
deployment
```

as a connected chain.

---

## 143. Observability Metadata

Include:

```text
environment
service
version
commit
region
```

in metrics/logs where appropriate.

---

## 144. High Cardinality

Avoid unbounded labels such as:

```text
user_id
request_id
full URL
```

in Prometheus metrics.

These can cause memory and storage problems.

---

## 145. Cardinality

Cardinality is the number of unique label combinations.

---

## 146. High Cardinality Incident

Symptoms:

```text
Prometheus memory growth
slow queries
large storage
```

---

## 147. Metric Naming

Use consistent names and units.

Examples:

```text
_seconds
_bytes
_total
```

according to metric conventions.

---

## 148. Counter

Counters increase over time.

Examples:

```text
requests_total
errors_total
```

---

## 149. Gauge

Gauges can increase or decrease.

Examples:

```text
memory_usage
active_jobs
queue_depth
```

---

## 150. Histogram

Histograms help measure distributions such as:

```text
request latency
job duration
```

---

## 151. Summary

Summaries can provide quantile information depending on implementation.

Understand the tradeoffs before choosing histogram vs summary.

---

## 152. Rate

For counters, calculate change over time using functions such as:

```promql
rate()
```

---

## 153. Increase

Use functions such as:

```promql
increase()
```

to estimate counter growth over a period.

---

## 154. Recording Rules

Precompute expensive PromQL queries.

Useful for:

```text
dashboards
alerts
high-volume queries
```

---

## 155. Alert Rule Testing

Validate alert expressions before production rollout.

---

## 156. Prometheus Retention

Choose retention based on:

```text
storage
query needs
compliance
capacity
```

---

## 157. Grafana Variables

Use dashboard variables for:

```text
environment
service
namespace
cluster
```

---

## 158. Grafana Dashboard Reuse

Create reusable dashboards with variables instead of duplicating dashboards per environment.

---

## 159. Grafana Alerting

Grafana can provide visualization and alerting capabilities depending on the architecture.

Keep alert ownership clear when multiple alerting systems exist.

---

## 160. ELK Ingestion

Monitor:

```text
events received
processing failures
queue/backpressure
Elasticsearch indexing
```

---

## 161. Logstash Backpressure

If Elasticsearch slows:

```text
Logstash queues grow
```

Monitor queue depth and throughput.

---

## 162. Elasticsearch Health

Monitor:

```text
cluster health
nodes
disk
heap
shards
indexing
search latency
```

---

## 163. Elasticsearch Disk Watermarks

Low disk space can affect shard allocation and indexing.

Monitor disk proactively.

---

## 164. Elasticsearch Heap

High heap pressure can cause:

```text
GC
latency
instability
```

---

## 165. Kibana Health

Monitor:

```text
availability
response time
backend connectivity
```

---

## 166. Log Search Performance

Slow queries can result from:

```text
large time ranges
high-cardinality fields
large indices
```

---

## 167. Log Index Lifecycle

Use an appropriate retention/lifecycle strategy.

---

## 168. Log Sampling

For extremely high-volume non-critical logs, sampling can reduce cost if operationally acceptable.

---

## 169. Log Aggregation During Incident

Start with:

```text
time
environment
service
severity
request ID
```

---

## 170. Incident Timeline

Build a timeline:

```text
10:00 deployment
10:02 errors rise
10:04 alert
10:06 rollback
10:08 recovery
```

---

## 171. Deployment Marker

Deployment timestamps should be available to responders.

---

## 172. Change Failure Correlation

Compare:

```text
deployment time
+
error spike
```

before concluding the deployment caused the issue.

---

## 173. Baseline

Know normal:

```text
traffic
latency
CPU
memory
pipeline duration
```

---

## 174. Anomaly Detection

A metric is more useful when compared with a baseline.

---

## 175. Capacity Planning

Use historical trends:

```text
CPU
memory
storage
runner jobs
API traffic
```

---

## 176. Runner Capacity Forecast

Estimate future concurrency.

---

## 177. Storage Forecast

Monitor:

```text
daily growth
monthly growth
retention
```

---

## 178. GitLab Upgrade Monitoring

Before upgrade:

```text
backup
capacity
compatibility
```

After upgrade:

```text
errors
latency
jobs
runners
API
```

---

## 179. Upgrade Canary

Test upgrades in a controlled environment before broad rollout where the architecture permits.

---

## 180. Post-Upgrade Validation

Validate:

```text
Git operations
MR
CI
runner
API
artifact
registry
deployment
```

---

## 181. Runner Upgrade

After runner upgrades verify:

```text
job pickup
Docker
Kubernetes executor
cache
artifacts
```

---

## 182. Monitoring the Monitoring Stack

Monitor:

```text
Prometheus
Grafana
Elasticsearch
Logstash
```

itself.

---

## 183. Prometheus Self-Monitoring

Track:

```text
scrape failures
target down
query latency
memory
storage
```

---

## 184. Target Down Alert

Alert when critical monitoring targets are unavailable.

---

## 185. Scrape Failure

Investigate:

```text
network
authentication
endpoint
TLS
target health
```

---

## 186. Grafana Availability

If Grafana is unavailable, metrics may still exist in Prometheus, but dashboards and alert workflows can be affected depending on architecture.

---

## 187. Elasticsearch Backup

Ensure recovery strategy exists for required log data.

---

## 188. Observability DR

Test:

```text
restore dashboards
restore metrics where required
restore logs
restore alerting
```

---

## 189. Monitoring Blind Spot

A system can be healthy while monitoring is broken.

Monitor the monitoring stack independently.

---

## 190. Synthetic Monitoring

Synthetic checks can validate:

```text
GitLab URL
login path
API
critical workflow
```

where safe and authorized.

---

## 191. CI Synthetic Test

A scheduled pipeline can test:

```text
repository access
pipeline trigger
runner availability
artifact upload
```

without changing production workloads.

---

## 192. Deployment Synthetic Test

A controlled environment can periodically validate:

```text
GitOps
ArgoCD
EKS
application health
```

---

## 193. Monitoring Security

Protect:

```text
Prometheus
Grafana
Kibana
```

with authentication and appropriate network access.

---

## 194. Grafana RBAC

Restrict dashboard and data-source access according to team needs.

---

## 195. Elasticsearch Security

Protect:

```text
indices
credentials
API access
```

---

## 196. Prometheus Security

Do not expose Prometheus administrative endpoints publicly.

---

## 197. Secrets in Monitoring

Do not put secret values into:

```text
metrics
labels
logs
dashboards
alerts
```

---

## 198. Alert Notification Security

Notification channels may contain:

```text
service names
URLs
incident details
```

Protect them appropriately.

---

## 199. Observability Cost

Monitor the cost of:

```text
metrics
logs
storage
retention
queries
```

---

## 200. Metric Cardinality Cost

High cardinality increases:

```text
RAM
disk
query cost
```

---

## 201. Log Volume Cost

Reduce unnecessary logs through:

```text
log levels
sampling
retention
filtering
```

---

## 202. Dashboard Query Cost

Avoid dashboards with dozens of expensive queries refreshing every few seconds.

---

## 203. Alert Query Cost

Alert queries should be efficient and tested.

---

## 204. Observability Ownership

Define owners for:

```text
metrics
logs
dashboards
alerts
on-call
```

---

## 205. Alert Runbook

Each critical alert should link to a runbook where possible.

---

## 206. Runbook Content

Include:

```text
meaning
impact
first checks
commands
dashboards
rollback
escalation
```

---

## 207. Alert Example

```text
GitLabRunnerQueueHigh
```

Meaning:

```text
CI jobs are waiting for execution.
```

First checks:

```text
runner status
runner capacity
job tags
runner errors
```

---

## 208. API Latency Alert

Example condition:

```text
P95 API latency above baseline
```

Investigate:

```text
application
database
network
load
```

---

## 209. Pipeline Failure Alert

Alert on production deployment pipeline failures rather than every development pipeline failure.

---

## 210. Deployment Failure Alert

A production deployment failure should include:

```text
service
environment
version
pipeline
failure stage
```

---

## 211. ArgoCD Alert

Useful signals:

```text
OutOfSync
Degraded
SyncFailed
```

depending on ArgoCD monitoring setup.

---

## 212. Kubernetes Alert

Examples:

```text
PodCrashLooping
PodPending
NodeNotReady
HighMemory
HighCPU
```

---

## 213. Application Alert

Examples:

```text
High5xxRate
HighLatency
LowAvailability
```

---

## 214. Alert Correlation

Combine signals:

```text
Deployment
+
ArgoCD sync
+
Pod restart
+
5xx increase
```

This can accelerate incident analysis.

---

## 215. Incident Response Flow

```text
Alert
 ↓
Acknowledge
 ↓
Assess impact
 ↓
Check recent change
 ↓
Inspect metrics
 ↓
Inspect logs
 ↓
Mitigate
 ↓
Verify recovery
 ↓
Root cause
```

---

## 216. Recent Change First

During incidents check:

```text
deployment
configuration
infrastructure
```

changes before assuming a random infrastructure failure.

---

## 217. Metrics First

Metrics quickly show:

```text
when
how much
which environment
which service
```

---

## 218. Logs Second

Logs explain:

```text
what
why
```

after identifying the affected period/service.

---

## 219. Kubernetes Events

Events often reveal:

```text
scheduling
image pull
probe
volume
permission
```

problems.

---

## 220. Incident Correlation

Example:

```text
10:05 deployment
10:06 Pod restarts
10:07 5xx increases
10:08 alert
```

This is strong evidence of correlation, but still validate causality.

---

## 221. Rollback Decision

Rollback when:

```text
impact high
previous version known-good
rollback safe
```

---

## 222. Fix Forward Decision

Fix forward when:

```text
change is small
rollback is risky
root cause understood
```

---

## 223. Post-Incident Observability

After incident ask:

```text
Did we detect quickly?
Was alert actionable?
Did logs contain enough context?
Could we correlate deployment?
```

---

## 224. Observability Improvement

Every incident should produce monitoring improvements when gaps are identified.

---

## 225. Production Monitoring Architecture

```text
                     GitLab
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
           Pipelines            API
              │                   │
              └─────────┬─────────┘
                        ▼
                     Metrics
                        │
                    Prometheus
                        │
                     Grafana
                        │
                  Dashboards/Alerts

GitLab / Runner / Kubernetes / Apps
              │
              ▼
             Logs
              │
           Logstash
              │
        Elasticsearch
              │
            Kibana
```

---

## 226. CI/CD Observability Architecture

```text
Developer
   │
   ▼
GitLab
   │
   ▼
Pipeline
   │
 ┌─┼───────────────┐
 ▼ ▼               ▼
Jobs Security     Build
 │
 ▼
Runner
 │
 ▼
ECR
 │
 ▼
GitOps
 │
 ▼
ArgoCD
 │
 ▼
EKS
 │
 ▼
Application
 │
 ├── Metrics → Prometheus → Grafana
 └── Logs → ELK
```

---

## 227. Production Dashboard Layout

```text
┌────────────────────────────────────┐
│ Availability | Error Rate | Latency│
├────────────────────────────────────┤
│ Traffic      | CPU        | Memory │
├────────────────────────────────────┤
│ Pipelines    | Runners    | Queue  │
├────────────────────────────────────┤
│ Deployments  | ArgoCD     | Pods   │
├────────────────────────────────────┤
│ Recent Errors / Logs               │
└────────────────────────────────────┘
```

---

## 228. Senior Interview — What Do You Monitor in GitLab?

> I monitor GitLab availability, API latency and errors, background work, database health, storage, runner availability, queue time, pipeline success rate, job duration, artifact and registry health, and deployment status.

---

## 229. Senior Interview — How Do You Monitor CI/CD?

> I measure pipeline success rate, P50/P95/P99 duration, queue time, runner utilization, failed jobs, security gate failures, deployment frequency and change failure rate. I use Prometheus/Grafana for metrics and ELK for detailed logs.

---

## 230. Senior Interview — How Do You Troubleshoot a Slow Pipeline?

> I first separate queue time from execution time. Then I inspect runner capacity, job duration, dependency caches, DAG dependencies and duplicate work. If queue time is high I investigate runners; if execution time is high I inspect the slow jobs.

---

## 231. Senior Interview — How Do You Monitor GitLab Runners?

> I monitor online/offline state, job queue, pickup latency, active concurrency, CPU, memory, failure rate and runner version. For Kubernetes runners I also monitor Pod startup and node capacity.

---

## 232. Senior Interview — How Do You Monitor Kubernetes Deployments?

> I monitor ArgoCD sync and health, Deployment rollout status, Pod readiness, restarts, events, CPU/memory, service and ingress health, and application error/latency metrics.

---

## 233. Senior Interview — How Do You Correlate a Deployment With an Incident?

> I compare deployment timestamps and versions with metric and log changes. I identify the exact image digest and GitOps revision, then correlate Pod events, application logs and Prometheus metrics before deciding whether the deployment caused the issue.

---

## 234. Senior Interview — Why Use Both Prometheus and ELK?

> Prometheus is optimized for numeric time-series metrics and alerting, while ELK provides centralized log collection, search and analysis. Metrics tell me when and how much; logs help explain what happened.

---

## 235. Senior Interview — What Is the Difference Between Monitoring and Observability?

> Monitoring tells me whether known conditions are healthy through predefined signals and alerts. Observability gives me enough telemetry and context to investigate unknown failure modes and understand system behavior.

---

## 236. Senior Interview — What Are the Four Golden Signals?

> Latency, traffic, errors and saturation. They provide a practical starting point for understanding service health and user impact.

---

## 237. Senior Interview — How Do You Avoid Alert Fatigue?

> I make alerts actionable, route them to the correct owner, use appropriate durations and thresholds, prioritize user-impacting symptoms, remove duplicate alerts and link critical alerts to runbooks.

---

## 238. Senior Interview — What Would You Alert On for GitLab Runners?

> I would alert on sustained queue growth, critical runner unavailability, abnormal job pickup latency, high failure rates and resource saturation. I would avoid paging on every temporary runner fluctuation.

---

## 239. Senior Interview — How Do You Monitor Elasticsearch?

> I monitor cluster health, node availability, disk usage, heap pressure, indexing rate, search latency, shard behavior and ingestion backpressure. I also monitor retention and storage growth.

---

## 240. Senior Interview — How Do You Troubleshoot Missing Logs?

> I check the application first, then the log collector/Logstash, network, Elasticsearch ingestion, index availability and Kibana query/time range. I determine whether logs were never generated, were lost during transport, or were indexed somewhere unexpected.

---

## 241. Senior Interview — How Do You Troubleshoot Missing Metrics?

> I check the target, endpoint, service discovery, Prometheus scrape status, network/TLS/authentication and the metric query itself. I also verify whether the application actually exposes the metric.

---

## 242. Senior Interview — What Is High Cardinality?

> High cardinality occurs when metrics have too many unique label combinations. For example, using user IDs or request IDs as Prometheus labels can create huge numbers of time series and cause memory and query-performance problems.

---

## 243. Senior Interview — How Do You Monitor Production Releases?

> I mark deployments, track ArgoCD health, Kubernetes rollout status, error rate, latency, traffic, resource usage and application logs. For high-risk releases I use progressive delivery and predefined rollback criteria.

---

## 244. Senior Interview — How Do You Design an Observability Strategy?

> I start from user and service objectives, define SLIs/SLOs, instrument metrics and structured logs, build focused dashboards, create actionable alerts, correlate deployments with telemetry, define runbooks and continuously improve monitoring based on incidents.

---

## 245. Final Monitoring Checklist

```text
[ ] GitLab availability
[ ] API latency
[ ] API errors
[ ] API rate limits
[ ] background jobs
[ ] database
[ ] cache
[ ] storage
[ ] artifacts
[ ] registry
[ ] runners
[ ] queue time
[ ] pipeline duration
[ ] pipeline failures
[ ] deployments
[ ] ArgoCD
[ ] Kubernetes
[ ] application metrics
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] logs
[ ] alerts
[ ] SLOs
[ ] error budgets
[ ] capacity
[ ] security
[ ] DR
```

---

## 246. Final Mental Model

```text
                         OBSERVABILITY

                             Users
                               │
                               ▼
                          Application
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
                 Metrics                 Logs
                    │                     │
                Prometheus              ELK
                    │                     │
                 Grafana               Kibana
                    │                     │
                    └──────────┬──────────┘
                               ▼
                         Correlation
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
       GitLab               ArgoCD                EKS
          │                    │                    │
       Pipeline             Sync                 Pods
          │                    │                    │
       Runners             Deployments          Services
          │                    │                    │
          └────────────────────┼────────────────────┘
                               ▼
                          Alert / Detect
                               │
                               ▼
                         Investigate
                               │
                               ▼
                         Mitigate
                               │
                               ▼
                           Improve
```

> **Core principle:** Production observability is not just collecting dashboards. It connects GitLab pipelines, runners, deployments, ArgoCD, EKS, applications, Prometheus metrics and ELK logs into one operational picture. The goal is to detect user-impacting problems quickly, explain what changed, recover safely and continuously improve the delivery platform.

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md ✓
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `21-GitLab-Production-Troubleshooting.md`**
