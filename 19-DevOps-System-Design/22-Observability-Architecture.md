# 19-DevOps-System-Design
# 22-Observability-Architecture

## 1. Purpose

Observability Architecture defines how a production platform collects,
correlates, stores, analyzes and acts on telemetry from applications,
Kubernetes, AWS infrastructure, CI/CD systems and business-critical services.

The goal is not simply to collect more telemetry.

The goal is to answer:

```text
What is happening?
Why is it happening?
Who is affected?
Where did it start?
How large is the blast radius?
What changed?
What should we do next?
```

Reference architecture:

```text
Applications
Kubernetes
AWS
Databases
Load Balancers
CI/CD
       |
       v
+-------------------------+
| Telemetry Collection    |
| Metrics | Logs | Traces |
+------------+------------+
             |
             v
+-------------------------+
| OpenTelemetry / Agents  |
+------------+------------+
             |
       +-----+-----+
       |           |
    Metrics     Logs/Traces
       |           |
       v           v
 Prometheus     ELK / Jaeger
       |           |
       +-----+-----+
             |
             v
          Grafana
             |
       +-----+-----+
       |           |
     Alerts      SLOs
       |           |
       +-----+-----+
             |
             v
       Incident Response
```

---

# PART I — OBSERVABILITY FOUNDATIONS

## 2. Monitoring vs Observability

Monitoring generally asks:

```text
Is this known condition unhealthy?
```

Observability asks:

```text
Can we understand the internal state of the system from its external
telemetry?
```

Monitoring is part of observability.

---

## 3. Three Pillars

The traditional model includes:

```text
metrics
logs
traces
```

Modern observability also commonly includes:

```text
profiles
events
continuous testing
business telemetry
```

---

## 4. Metrics

Metrics are numerical measurements over time.

Examples:

```text
request_rate
error_rate
cpu_usage
memory_usage
latency
queue_depth
```

---

## 5. Logs

Logs describe discrete events.

Example:

```text
timestamp
service
level
message
request_id
trace_id
```

---

## 6. Traces

A trace follows a request across distributed components.

```text
User
 |
API
 |
Service A
 |
Service B
 |
Database
```

---

# PART II — OBSERVABILITY DESIGN

## 7. Design Questions

Before selecting tools determine:

```text
What must be observed?
Who consumes telemetry?
What decisions depend on it?
How quickly must data arrive?
How long must it be retained?
What is the acceptable cost?
```

---

## 8. Golden Signals

The classic service-level signals are:

```text
latency
traffic
errors
saturation
```

Use them as a starting point rather than the entire observability strategy.

---

## 9. RED Method

For request-driven services:

```text
Rate
Errors
Duration
```

---

## 10. USE Method

For infrastructure:

```text
Utilization
Saturation
Errors
```

---

# PART III — OBSERVABILITY ARCHITECTURE

## 11. Collection

```text
application
 |
SDK / agent
 |
collector
 |
backend
```

---

## 12. Processing

Telemetry may require:

```text
filtering
sampling
enrichment
redaction
aggregation
routing
```

---

## 13. Storage

Choose storage based on:

```text
query pattern
retention
scale
latency
cost
```

---

# PART IV — PROMETHEUS

## 14. Prometheus

Prometheus is commonly used for time-series metrics collection, storage and
querying.

Architecture:

```text
Targets
 |
Service Discovery
 |
Prometheus
 |
TSDB
 |
PromQL
 |
Grafana / Alertmanager
```

---

## 15. Pull Model

Prometheus commonly scrapes metrics endpoints.

```text
Prometheus
 |
GET /metrics
 |
Application
```

---

## 16. Scrape Configuration

Concept:

```yaml
scrape_configs:
  - job_name: application
    static_configs:
      - targets:
          - app:8080
```

Production environments normally use service discovery rather than static
targets.

---

# PART V — PROMETHEUS SERVICE DISCOVERY

## 17. Kubernetes Discovery

Prometheus can discover:

```text
pods
services
nodes
endpoints
```

using Kubernetes APIs.

---

## 18. Labels

Prometheus labels identify dimensions.

Example:

```text
http_requests_total{
  service="payments",
  method="GET",
  status="200"
}
```

---

# PART VI — CARDINALITY

## 19. Cardinality

Cardinality is the number of unique label combinations.

Dangerous label:

```text
user_id
```

because millions of users can produce millions of time series.

---

## 20. Avoid Unbounded Labels

Do not use:

```text
request_id
user_id
session_id
raw_url
```

as high-cardinality metric labels unless there is a very deliberate design.

---

# PART VII — PROMQL

## 21. Query

Example:

```promql
rate(http_requests_total[5m])
```

---

## 22. Error Rate

Concept:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m]))
```

---

## 23. Aggregation

Use:

```promql
sum by (service)
```

to analyze service-level behavior.

---

# PART VIII — GRAFANA

## 24. Grafana

Grafana provides dashboards and visualization across observability data
sources.

Typical flow:

```text
Prometheus
 |
Grafana
 |
Dashboard
```

---

## 25. Dashboard Design

A production dashboard should answer:

```text
Is the service healthy?
What changed?
Which dependency is failing?
Are users affected?
```

---

# PART IX — DASHBOARD HIERARCHY

## 26. Executive

```text
availability
error budget
major incidents
```

---

## 27. Service

```text
traffic
errors
latency
saturation
dependencies
```

---

## 28. Infrastructure

```text
CPU
memory
disk
network
node health
```

---

# PART X — ALERTMANAGER

## 29. Alerting

Prometheus can evaluate alerting rules and route alerts through an alert
manager.

---

## 30. Alert Routing

```text
alert
 |
labels
 |
route
 |
team
 |
notification
```

---

## 31. Alert Grouping

Group related alerts to avoid notification storms.

---

# PART XI — ALERT QUALITY

## 32. Good Alert

A good alert is:

```text
actionable
specific
owned
time-sensitive
```

---

## 33. Bad Alert

Avoid alerts for every small metric fluctuation.

---

# PART XII — ALERT FATIGUE

## 34. Problem

Too many alerts cause:

```text
ignored alerts
slow response
operator burnout
```

---

## 35. Solution

Use:

```text
SLO alerts
symptom-based alerts
dependency correlation
severity
deduplication
```

---

# PART XIII — SLO

## 36. Service Level Objective

Example:

```text
99.9% successful requests
```

over a defined period.

---

## 37. SLI

An SLI measures the actual service behavior.

Example:

```text
successful requests / total requests
```

---

## 38. SLA

An SLA is a business or contractual commitment.

Do not confuse SLA with SLO.

---

# PART XIV — ERROR BUDGET

## 39. Error Budget

If SLO is:

```text
99.9%
```

the permitted failure budget is approximately:

```text
0.1%
```

---

## 40. Error Budget Policy

When the budget is exhausted:

```text
slow risky releases
increase reliability work
```

depending on organizational policy.

---

# PART XV — BURN RATE

## 41. Burn Rate

Burn rate measures how quickly the error budget is being consumed.

Fast burn:

```text
high risk
```

Slow burn:

```text
long-term degradation
```

---

# PART XVI — LOGGING

## 42. Structured Logs

Prefer machine-readable logs:

```json
{
  "timestamp": "...",
  "level": "ERROR",
  "service": "payments",
  "request_id": "...",
  "trace_id": "...",
  "message": "database timeout"
}
```

---

## 43. Log Levels

Typical:

```text
DEBUG
INFO
WARN
ERROR
```

Use production levels deliberately.

---

# PART XVII — ELK

## 44. ELK

Common architecture:

```text
Application
 |
Log Collector
 |
Logstash
 |
Elasticsearch
 |
Kibana
```

---

## 45. Elasticsearch

Provides indexing and search for log data.

---

## 46. Logstash

Can provide:

```text
collection
parsing
filtering
enrichment
routing
```

---

## 47. Kibana

Provides visualization and exploration of Elasticsearch data.

---

# PART XVIII — LOG COLLECTION

## 48. Kubernetes

A common pattern:

```text
Pod stdout
 |
Node log
 |
Agent
 |
Log backend
```

---

## 49. DaemonSet Collector

A node-level collector can gather logs from workloads running on that node.

---

# PART XIX — LOG PIPELINE

## 50. Production Pipeline

```text
container
 |
node
 |
collector
 |
buffer
 |
processor
 |
Elasticsearch
 |
Kibana
```

---

# PART XX — LOG PARSING

## 51. Parsing

Convert raw messages into fields:

```text
timestamp
service
status
latency
request_id
trace_id
```

---

# PART XXI — LOG RETENTION

## 52. Retention

Retention should depend on:

```text
operational need
security
compliance
cost
```

---

## 53. Hot / Warm / Cold

Concept:

```text
Hot
 |
Warm
 |
Cold
 |
Delete
```

---

# PART XXII — LOG COST

## 54. Control Volume

Reduce unnecessary cost using:

```text
sampling
filtering
compression
retention policies
```

---

# PART XXIII — LOG SECURITY

## 55. Redaction

Never intentionally log:

```text
password
token
private key
secret
```

---

# PART XXIV — PII

## 56. Sensitive Data

Define what sensitive data must be masked or removed before storage.

---

# PART XXV — TRACE CONTEXT

## 57. Correlation

Use:

```text
trace_id
span_id
request_id
```

to connect logs, traces and requests.

---

# PART XXVI — OPENTELEMETRY

## 58. OpenTelemetry

OpenTelemetry provides vendor-neutral instrumentation, telemetry APIs/SDKs
and collector components.

---

## 59. Architecture

```text
Application
 |
OTel SDK
 |
OTel Collector
 |
+---------+---------+
|         |         |
Metrics   Logs     Traces
```

---

# PART XXVII — OTEL COLLECTOR

## 60. Collector

The collector can provide:

```text
receive
process
filter
transform
batch
export
```

---

## 61. Collector Pipelines

Concept:

```yaml
receivers:
  otlp:

processors:
  batch:

exporters:
  otlp:

service:
  pipelines:
    traces:
      receivers: [otlp]
      processors: [batch]
      exporters: [otlp]
```

---

# PART XXVIII — OTEL AGENT VS COLLECTOR

## 62. Deployment

Common patterns:

```text
DaemonSet
Deployment
Gateway
```

---

## 63. DaemonSet

Useful for node-local telemetry collection.

---

## 64. Gateway

Useful for centralized processing and routing.

---

# PART XXIX — TRACE SAMPLING

## 65. Sampling

Tracing every request can become expensive at high traffic.

Options:

```text
head sampling
tail sampling
probabilistic sampling
```

---

## 66. Tail Sampling

Tail sampling makes decisions after observing trace information.

This can preserve traces for:

```text
errors
slow requests
specific services
```

---

# PART XXX — JAEGER

## 67. Jaeger

Jaeger is a distributed tracing platform commonly used to collect and explore
trace data.

Architecture:

```text
Application
 |
OpenTelemetry
 |
Collector
 |
Jaeger
 |
UI
```

---

# PART XXXI — DISTRIBUTED TRACING

## 68. Trace

Example:

```text
trace
 |
+-- API span
    |
    +-- auth span
    |
    +-- payment span
         |
         +-- database span
```

---

# PART XXXII — SPAN DESIGN

## 69. Span Attributes

Useful:

```text
service.name
http.method
http.route
http.status_code
db.system
```

Avoid sensitive or unbounded values.

---

# PART XXXIII — TRACE PROPAGATION

## 70. Context

Trace context should propagate across service boundaries.

```text
Service A
 |
HTTP headers
 |
Service B
```

---

# PART XXXIV — TRACE SAMPLING STRATEGY

## 71. Production

Sample:

```text
normal traffic -> lower percentage
errors -> retain
slow requests -> retain
critical transactions -> retain
```

---

# PART XXXV — METRICS + LOGS + TRACES

## 72. Correlation

```text
Metric spike
 |
Grafana
 |
service
 |
trace
 |
slow span
 |
log
 |
database timeout
```

This is the core value of unified observability.

---

# PART XXXVI — EXEMPLARS

## 73. Exemplars

Exemplars can connect a metric observation to a trace.

Concept:

```text
latency metric
 |
trace ID
 |
trace
```

---

# PART XXXVII — BUSINESS OBSERVABILITY

## 74. Business Metrics

Technical telemetry should be connected to:

```text
orders
payments
signups
revenue
```

when appropriate.

---

# PART XXXVIII — CUSTOMER IMPACT

## 75. User-Centric Monitoring

A CPU alert is less meaningful than:

```text
checkout success rate dropped
```

when designing user-impact observability.

---

# PART XXXIX — DEPENDENCY OBSERVABILITY

## 76. Dependencies

Track:

```text
database
cache
queue
external API
DNS
```

---

# PART XL — DATABASE OBSERVABILITY

## 77. Database

Monitor:

```text
latency
connections
locks
CPU
IO
storage
replication
errors
```

---

# PART XLI — KUBERNETES OBSERVABILITY

## 78. Cluster

Observe:

```text
nodes
pods
containers
API server
scheduler
controller manager
```

where applicable.

---

# PART XLII — POD HEALTH

## 79. Pod Signals

```text
restart count
OOMKilled
readiness
liveness
CPU
memory
network
```

---

# PART XLIII — NODE HEALTH

## 80. Node Signals

```text
CPU
memory
disk pressure
PID pressure
network
filesystem
```

---

# PART XLIV — EKS OBSERVABILITY

## 81. AWS

Include appropriate:

```text
CloudWatch
AWS service metrics
EKS control-plane telemetry
VPC telemetry
load balancer metrics
```

---

# PART XLV — CONTROL PLANE OBSERVABILITY

## 82. Kubernetes API

Monitor:

```text
request latency
request rate
errors
API saturation
```

---

# PART XLVI — HPA OBSERVABILITY

## 83. Autoscaling

Observe:

```text
current replicas
desired replicas
CPU
memory
custom metrics
scaling events
```

---

# PART XLVII — KEDA

## 84. Event Scaling

For event-driven workloads monitor:

```text
queue depth
trigger state
desired replicas
scaling latency
```

---

# PART XLVIII — SERVICE MESH

## 85. Mesh Telemetry

A service mesh may provide:

```text
request metrics
mTLS telemetry
service topology
distributed tracing
```

---

# PART XLIX — OBSERVABILITY OF OBSERVABILITY

## 86. Telemetry Pipeline

Monitor the monitoring system itself:

```text
collector queue
dropped spans
dropped logs
scrape failures
ingestion latency
backend errors
```

---

# PART L — PROMETHEUS HIGH AVAILABILITY

## 87. HA

A single Prometheus instance may become a failure point.

Use:

```text
replicas
remote storage
federation
remote write
```

as appropriate.

---

# PART LI — LONG-TERM METRICS

## 88. Remote Storage

Prometheus can use remote-write integrations for long-term metric storage.

---

# PART LII — THANOS / MIMIR / OTHER SYSTEMS

## 89. Long-Term Architecture

At large scale, systems such as Thanos or Grafana Mimir can provide scalable
long-term metric architectures.

Selection should consider:

```text
scale
query
HA
retention
cost
operations
```

---

# PART LIII — PROMETHEUS FEDERATION

## 90. Federation

Useful for aggregating selected metrics across environments or Prometheus
instances.

---

# PART LIV — MULTI-CLUSTER OBSERVABILITY

## 91. Fleet

```text
Cluster A
 |
Cluster B
 |
Cluster C
 |
Central Observability
```

---

# PART LV — MULTI-ACCOUNT

## 92. AWS

Centralize security and operational telemetry while maintaining appropriate
account boundaries.

---

# PART LVI — MULTI-REGION

## 93. Regional Architecture

```text
Region A
 |
Local telemetry
 |
Regional backend
 |
Global view

Region B
 |
Local telemetry
 |
Regional backend
 |
Global view
```

---

# PART LVII — OBSERVABILITY HA

## 94. Failure Domains

Separate:

```text
collection
processing
storage
visualization
alerting
```

failure domains where required.

---

# PART LVIII — ALERTING HA

## 95. Critical Alerts

Do not depend on a single alerting path for critical incidents.

Use appropriate redundant notification channels.

---

# PART LIX — TELEMETRY LOSS

## 96. Loss

Some telemetry can be sampled or dropped intentionally.

Critical audit/security telemetry may require stronger delivery guarantees.

---

# PART LX — BACKPRESSURE

## 97. Telemetry Backpressure

When backend capacity falls:

```text
application
 |
collector
 |
queue
 |
backend
```

must avoid uncontrolled memory growth.

---

# PART LXI — BATCHING

## 98. Batch

Batching can reduce network and backend overhead.

Trade-off:

```text
throughput
vs
latency
```

---

# PART LXII — COMPRESSION

## 99. Compression

Compression can reduce network and storage cost but increases CPU usage.

---

# PART LXIII — SAMPLING

## 100. Sampling

Use different policies for:

```text
metrics
logs
traces
```

because their cost and diagnostic value differ.

---

# PART LXIV — METRIC RETENTION

## 101. Retention

High-resolution metrics may require shorter retention than aggregated metrics.

---

# PART LXV — LOG RETENTION

## 102. Retention

Use tiering where long-term retention is required.

---

# PART LXVI — TRACE RETENTION

## 103. Retention

Keep high-value traces longer where useful:

```text
errors
slow requests
critical transactions
```

---

# PART LXVII — OBSERVABILITY COST

## 104. Cost Model

Total cost includes:

```text
collection
network
processing
indexing
storage
query
egress
retention
```

---

# PART LXVIII — COST CONTROL

## 105. Controls

```text
sampling
filtering
aggregation
retention
compression
cardinality control
```

---

# PART LXIX — CARDINALITY MANAGEMENT

## 106. Metrics

Control:

```text
label count
label values
series count
```

---

# PART LXX — LOG CARDINALITY

## 107. Logs

Unbounded fields can make search and indexing expensive.

---

# PART LXXI — TRACE CARDINALITY

## 108. Spans

Avoid excessive unique attributes.

---

# PART LXXII — OBSERVABILITY SECURITY

## 109. Access Control

Not every engineer should see:

```text
all logs
all customer data
all traces
```

---

# PART LXXIII — DATA REDACTION

## 110. Redaction

Apply controls before sensitive data reaches long-term storage.

---

# PART LXXIV — MULTI-TENANT OBSERVABILITY

## 111. Isolation

Tenant A should not automatically see Tenant B telemetry.

Use:

```text
tenant labels
authorization
separate storage
```

where required.

---

# PART LXXV — OBSERVABILITY RBAC

## 112. Grafana

Control:

```text
dashboard access
data source access
folder access
administration
```

---

# PART LXXVI — OBSERVABILITY INCIDENT

## 113. Pipeline Failure

If telemetry disappears:

```text
detect
 |
identify collector/backend
 |
restore telemetry
 |
validate
 |
investigate data loss
```

---

# PART LXXVII — SLO ALERTING

## 114. Principle

Prefer alerts based on customer-impacting SLOs over infrastructure symptoms
when possible.

---

# PART LXXVIII — SYMPTOM VS CAUSE

## 115. Example

Bad:

```text
CPU > 80%
```

Better:

```text
checkout latency SLO is burning rapidly
```

CPU can still be useful for diagnosis.

---

# PART LXXIX — DEPENDENCY CORRELATION

## 116. Example

```text
Checkout error rate
 |
Payment service
 |
Database latency
 |
RDS connection saturation
```

---

# PART LXXX — CHANGE CORRELATION

## 117. Deployments

Connect telemetry to:

```text
deployment
commit
version
configuration change
```

---

# PART LXXXI — RELEASE OBSERVABILITY

## 118. Deployment

During rollout compare:

```text
old version
vs
new version
```

for:

```text
errors
latency
resource usage
```

---

# PART LXXXII — CANARY OBSERVABILITY

## 119. Canary

```text
canary traffic
 |
metrics
 |
traces
 |
errors
 |
decision
```

---

# PART LXXXIII — AUTOMATED ROLLBACK

## 120. Rollback

If defined SLO conditions fail:

```text
deployment
 |
analysis
 |
rollback
```

---

# PART LXXXIV — BLUE-GREEN OBSERVABILITY

## 121. Compare

Observe both environments before switching traffic.

---

# PART LXXXV — INCIDENT RESPONSE

## 122. Observability During Incident

First determine:

```text
customer impact
start time
affected services
recent changes
dependency health
```

---

# PART LXXXVI — INCIDENT TIMELINE

## 123. Timeline

Use:

```text
deployments
alerts
metrics
logs
traces
events
```

to reconstruct the incident.

---

# PART LXXXVII — CORRELATION ID

## 124. Standard

Every important request should have a correlation mechanism.

---

# PART LXXXVIII — REQUEST ID VS TRACE ID

## 125. Difference

Request ID identifies an application request.

Trace ID identifies a distributed trace.

They can be correlated.

---

# PART LXXXIX — LOG SCHEMA

## 126. Standard Fields

Recommended baseline:

```text
timestamp
level
service.name
environment
host
pod
namespace
request_id
trace_id
span_id
message
```

---

# PART XC — METRIC NAMING

## 127. Convention

Use consistent names and units.

Examples:

```text
http_requests_total
http_request_duration_seconds
```

---

# PART XCI — UNITS

## 128. Consistency

Prefer standard units:

```text
seconds
bytes
bytes_per_second
```

---

# PART XCII — LABEL DESIGN

## 129. Good Labels

```text
service
environment
region
status
method
```

---

# PART XCIII — BAD LABELS

## 130. Avoid

```text
user_id
session_id
random_uuid
full_url
```

when unbounded.

---

# PART XCIV — ALERT LABELS

## 131. Labels

Use labels for:

```text
severity
service
team
environment
```

to route alerts.

---

# PART XCV — ALERT ANNOTATIONS

## 132. Runbook

Alerts should link conceptually to:

```text
runbook
dashboard
service owner
```

---

# PART XCVI — RUNBOOK ALERT

## 133. Example

```text
Alert:
Checkout SLO burn rate high

Owner:
Payments team

Dashboard:
Checkout overview

Runbook:
Checkout incident procedure
```

---

# PART XCVII — ON-CALL

## 134. Ownership

Every paging alert should have an accountable team.

---

# PART XCVIII — ALERT PRIORITY

## 135. Severity

Example:

```text
P1 -> immediate customer impact
P2 -> major degradation
P3 -> limited impact
```

Exact definitions should be organizationally standardized.

---

# PART XCIX — NOISE REDUCTION

## 136. Suppression

Suppress dependent alerts when a known root cause explains them.

---

# PART C — ALERT DEDUPLICATION

## 137. Example

One database outage may trigger:

```text
50 services
 |
50 alerts
```

Deduplicate or group when appropriate.

---

# PART CI — RECORDING RULES

## 138. Prometheus

Precompute expensive frequently used queries with recording rules.

---

# PART CII — QUERY PERFORMANCE

## 139. Observability Backend

Optimize:

```text
query range
aggregation
index
retention
```

---

# PART CIII — GRAFANA VARIABLES

## 140. Dashboards

Use controlled variables:

```text
cluster
namespace
service
environment
```

Avoid dashboards that generate expensive uncontrolled queries.

---

# PART CIV — GOLDEN SIGNAL DASHBOARD

## 141. Example

```text
Service: payments

Traffic:       12k req/s
Errors:        0.08%
P95 latency:   180 ms
Saturation:    62%

Dependencies:
DB      healthy
Redis   healthy
Queue   warning
```

---

# PART CV — SERVICE TOPOLOGY

## 142. Map

Visualize:

```text
frontend
 |
API
 |
payments
 |
database
```

and identify dependency health.

---

# PART CVI — SERVICE CATALOG

## 143. Ownership

Observability should connect telemetry to:

```text
service
team
repository
environment
runbook
```

---

# PART CVII — PLATFORM OBSERVABILITY

## 144. Platform

Observe:

```text
CI/CD
GitOps
Kubernetes
Terraform
registries
secret systems
```

---

# PART CVIII — CI/CD OBSERVABILITY

## 145. Pipeline Metrics

Track:

```text
build duration
queue time
failure rate
deployment frequency
lead time
```

---

# PART CIX — GITOPS OBSERVABILITY

## 146. Argo CD

Track:

```text
sync status
health
sync failures
drift
deployment duration
```

---

# PART CX — TERRAFORM OBSERVABILITY

## 147. IaC

Track:

```text
plan failures
apply failures
duration
drift
```

---

# PART CXI — OBSERVABILITY QUALITY

## 148. Telemetry Quality

Telemetry should be:

```text
accurate
timely
complete enough
consistent
actionable
```

---

# PART CXII — MISSING TELEMETRY

## 149. Unknown Unknowns

A blank dashboard is not proof of health.

Distinguish:

```text
zero
```

from:

```text
no data
```

---

# PART CXIII — ABSENT VS ZERO

## 150. Critical

An absent metric may mean:

```text
application down
scrape broken
instrumentation broken
```

not necessarily:

```text
zero activity
```

---

# PART CXIV — OBSERVABILITY TESTING

## 151. Test

Test:

```text
metric generation
log generation
trace propagation
alert firing
dashboard queries
```

---

# PART CXV — CHAOS

## 152. Failure Testing

Inject:

```text
latency
errors
pod failures
node failures
database failures
```

and verify observability detects them.

---

# PART CXVI — SYNTHETIC MONITORING

## 153. Synthetic

Generate controlled transactions:

```text
login
search
checkout
```

to verify user paths.

---

# PART CXVII — BLACK-BOX VS WHITE-BOX

## 154. Black-Box

Observe system externally.

---

## 155. White-Box

Observe internal metrics and telemetry.

Use both.

---

# PART CXVIII — UPTIME MONITORING

## 156. External

An external probe can detect:

```text
DNS failure
network failure
application outage
```

even when internal monitoring fails.

---

# PART CXIX — OBSERVABILITY DEPENDENCY

## 157. Independence

Avoid making the observability stack depend entirely on the production
application it monitors.

---

# PART CXX — OUT-OF-BAND MONITORING

## 158. Principle

Critical monitoring should have enough independence to observe a production
failure.

---

# PART CXXI — SECURITY OBSERVABILITY

## 159. Security Signals

Include:

```text
authentication failures
IAM changes
secret access
network anomalies
privilege escalation
```

---

# PART CXXII — AUDIT VS OBSERVABILITY

## 160. Difference

Operational telemetry optimizes diagnosis.

Audit telemetry may require stronger:

```text
integrity
retention
access controls
```

---

# PART CXXIII — COMPLIANCE

## 161. Retention

Compliance requirements can dictate telemetry retention and access.

---

# PART CXXIV — PII

## 162. Privacy

Observability systems can become accidental data warehouses.

Collect only what is needed.

---

# PART CXXV — DATA MASKING

## 163. Example

Mask:

```text
email
phone
payment data
tokens
```

when required.

---

# PART CXXVI — OBSERVABILITY RBAC

## 164. Access

Separate:

```text
viewer
operator
administrator
security
```

permissions.

---

# PART CXXVII — MULTI-TEAM

## 165. Ownership

Teams should see their services while platform teams maintain shared
observability infrastructure.

---

# PART CXXVIII — CENTRAL PLATFORM

## 166. Platform

Provide:

```text
standard collectors
dashboards
alerts
SDKs
logging formats
```

---

# PART CXXIX — GOLDEN DASHBOARDS

## 167. Standard

Every production service should have a baseline dashboard.

---

# PART CXXX — GOLDEN ALERTS

## 168. Standard

Every production service should have baseline:

```text
availability
error rate
latency
saturation
```

alerts appropriate to its workload.

---

# PART CXXXI — OBSERVABILITY MATURITY

## 169. Levels

```text
0 -> logs only
1 -> basic metrics
2 -> dashboards and alerts
3 -> distributed tracing
4 -> SLO-driven
5 -> correlated and automated
```

---

# PART CXXXII — AUTOMATED REMEDIATION

## 170. Observability

Some alerts can trigger controlled automation:

```text
alert
 |
workflow
 |
restart / scale / rollback
 |
verify
```

Use guardrails to avoid remediation loops.

---

# PART CXXXIII — ALERT TO INCIDENT

## 171. Workflow

```text
signal
 |
alert
 |
deduplicate
 |
route
 |
incident
 |
investigation
 |
resolution
```

---

# PART CXXXIV — INCIDENT CORRELATION

## 172. Correlate

Join:

```text
metric anomaly
+
deployment
+
trace
+
log
+
infrastructure event
```

---

# PART CXXXV — CHANGE EVENTS

## 173. Deployment Marker

Put deployment events on service dashboards.

---

# PART CXXXVI — RELEASE MARKERS

## 174. Example

```text
14:00 -> version 2.8.1
14:05 -> error rate rises
14:10 -> latency SLO burns
```

This dramatically reduces investigation time.

---

# PART CXXXVII — OBSERVABILITY FOR CANARY

## 175. Analysis

Compare:

```text
baseline
vs
canary
```

using the same signals.

---

# PART CXXXVIII — AUTOMATED ANALYSIS

## 176. Progressive Delivery

Use metrics to decide:

```text
promote
pause
rollback
```

---

# PART CXXXIX — OBSERVABILITY ARCHITECTURE FOR EKS

## 177. Reference

```text
Pods
 |
OpenTelemetry / Prometheus
 |
Collectors
 |
+------------------+
|                  |
Metrics            Logs/Traces
|                  |
Prometheus         ELK / Jaeger
|                  |
+--------+---------+
         |
       Grafana
         |
    Alerting / SLO
```

---

# PART CXL — EKS NODE TELEMETRY

## 178. Node

Collect:

```text
CPU
memory
filesystem
network
container runtime
```

---

# PART CXLI — EKS CONTROL PLANE

## 179. Control Plane

Observe relevant:

```text
API requests
authentication
authorization
scheduler behavior
controller behavior
```

---

# PART CXLII — AWS TELEMETRY

## 180. Services

Integrate relevant telemetry from:

```text
ALB
NLB
RDS
ElastiCache
SQS
SNS
Lambda
EKS
VPC
```

---

# PART CXLIII — OBSERVABILITY DATA FLOW

## 181. Production

```text
Workload
 |
Agent
 |
Collector
 |
Buffer
 |
Processor
 |
Backend
 |
Grafana
 |
Alert
 |
On-call
```

---

# PART CXLIV — COLLECTOR SCALING

## 182. Scale

Scale collectors based on:

```text
telemetry rate
CPU
memory
queue depth
export latency
```

---

# PART CXLV — COLLECTOR HA

## 183. HA

Use multiple collector instances.

Avoid a single telemetry gateway for critical environments.

---

# PART CXLVI — QUEUES

## 184. Buffer

Queues can absorb temporary backend slowdowns.

But queues do not solve indefinite backend outages.

---

# PART CXLVII — DROPPED TELEMETRY

## 185. Monitor

Alert on:

```text
dropped spans
dropped logs
failed exports
scrape failures
```

---

# PART CXLVIII — BACKEND FAILURE

## 186. Strategy

```text
backend unavailable
 |
buffer
 |
retry
 |
backpressure
 |
controlled drop
```

Define priorities for what must be retained.

---

# PART CXLIX — PRIORITY

## 187. Telemetry Priority

Example:

```text
security audit -> highest
critical errors -> high
traces -> medium
debug logs -> low
```

Exact policy depends on requirements.

---

# PART CL — STORAGE ARCHITECTURE

## 188. Separate Workloads

Consider separating:

```text
metrics
logs
traces
```

storage characteristics.

---

# PART CLI — ELASTICSEARCH SCALING

## 189. Scale

Consider:

```text
data nodes
master-eligible nodes
ingest
shards
replicas
storage
```

---

# PART CLII — SHARDING

## 190. Elasticsearch

Poor shard design can cause:

```text
hot shards
slow queries
resource waste
```

---

# PART CLIII — INDEX LIFECYCLE

## 191. ILM

Use lifecycle policies for:

```text
hot
warm
cold
delete
```

---

# PART CLIV — JAEGER SCALING

## 192. Traces

At scale consider:

```text
collector
storage
sampling
retention
query capacity
```

---

# PART CLV — QUERY GOVERNANCE

## 193. Observability Queries

Protect backend capacity from expensive uncontrolled queries.

---

# PART CLVI — DASHBOARD GOVERNANCE

## 194. Dashboards

Avoid dashboards with dozens of expensive queries refreshing every few
seconds.

---

# PART CLVII — OBSERVABILITY DR

## 195. Recovery

Define:

```text
configuration backup
dashboard backup
alert rules
recording rules
collector configuration
```

---

# PART CLVIII — OBSERVABILITY BACKUP

## 196. GitOps

Store observability configuration in Git where practical.

---

# PART CLIX — DASHBOARD AS CODE

## 197. Principle

Manage:

```text
dashboards
alerts
recording rules
collector config
```

as code where possible.

---

# PART CLX — VERSION CONTROL

## 198. Benefits

Versioning provides:

```text
review
rollback
audit
reproducibility
```

---

# PART CLXI — OBSERVABILITY TEST ENVIRONMENT

## 199. Test

Validate:

```text
collector config
PromQL
alert rules
dashboard queries
```

before production.

---

# PART CLXII — ALERT UNIT TESTING

## 200. Rules

Test alert expressions against known time-series scenarios.

---

# PART CLXIII — RECORDING RULE TESTING

## 201. Queries

Validate recording rules before relying on them for SLOs.

---

# PART CLXIV — OBSERVABILITY SECURITY

## 202. Threat Model

Threats include:

```text
telemetry injection
log poisoning
credential leakage
unauthorized access
data exfiltration
```

---

# PART CLXV — LOG INJECTION

## 203. Protection

Structure and sanitize user-controlled fields appropriately.

---

# PART CLXVI — TELEMETRY INJECTION

## 204. Protection

Validate telemetry sources and authentication where required.

---

# PART CLXVII — OBSERVABILITY CREDENTIALS

## 205. Access

Collectors should have only the permissions required to:

```text
read metrics
read logs
export telemetry
```

---

# PART CLXVIII — SECRET SAFETY

## 206. Never

Do not include:

```text
passwords
tokens
private keys
```

in telemetry.

---

# PART CLXIX — INCIDENT FORENSICS

## 207. Timeline

Use:

```text
logs
metrics
traces
deployments
cloud events
Kubernetes events
```

to reconstruct incidents.

---

# PART CLXX — ROOT CAUSE ANALYSIS

## 208. RCA

Do not stop at:

```text
CPU was high
```

Find:

```text
why CPU increased
what changed
why safeguards failed
what customers experienced
```

---

# PART CLXXI — FIVE WHYS

## 209. Example

```text
Checkout failed
 -> DB latency increased
 -> connection pool saturated
 -> deployment increased query count
 -> missing query optimization
```

---

# PART CLXXII — DEPENDENCY MAP

## 210. Architecture

Maintain service dependency visibility.

---

# PART CLXXIII — SERVICE OWNERSHIP

## 211. Catalog

For every production service:

```text
owner
repository
on-call
dashboard
runbook
SLO
```

---

# PART CLXXIV — OBSERVABILITY CONTRACT

## 212. Standard

Platform can define a minimum telemetry contract:

```text
metrics endpoint
structured logs
trace propagation
health endpoints
service metadata
```

---

# PART CLXXV — HEALTH ENDPOINTS

## 213. Liveness

Liveness answers:

```text
Should this process be restarted?
```

---

## 214. Readiness

Readiness answers:

```text
Should this instance receive traffic?
```

Do not use them interchangeably.

---

# PART CLXXVI — STARTUP PROBE

## 215. Startup

Use startup probes when initialization can take longer than normal health
checks.

---

# PART CLXXVII — OBSERVABILITY ANTI-PATTERNS

## 216. Anti-Pattern

```text
collect everything forever
```

This creates:

```text
cost
noise
privacy risk
query complexity
```

---

## 217. Anti-Pattern

```text
alert on every metric
```

Creates alert fatigue.

---

## 218. Anti-Pattern

```text
CPU dashboard only
```

does not demonstrate application health.

---

## 219. Anti-Pattern

```text
logs without correlation IDs
```

makes distributed troubleshooting harder.

---

## 220. Anti-Pattern

```text
tracing without sampling strategy
```

can become prohibitively expensive.

---

# PART CLXXVII — SENIOR SYSTEM DESIGN

## 221. Design Global Observability Platform

```text
Region A                 Region B
   |                        |
Collectors              Collectors
   |                        |
   +-----------+------------+
               |
        Central / Global
        Observability
               |
      +--------+--------+
      |        |        |
   Metrics    Logs    Traces
      |        |        |
      +--------+--------+
               |
            Grafana
               |
        SLO / Alerts
               |
           On-Call
```

---

## 222. Design EKS Observability

Requirements:

```text
100+ clusters
multi-account
multi-region
high availability
central dashboards
tenant isolation
cost control
```

Architecture:

```text
EKS
 |
Node / Pod Collectors
 |
OTel / Prometheus
 |
Regional Gateway
 |
Central Backends
 |
Grafana
```

---

## 223. Design 1 Million Requests/Second

At this scale:

```text
do not blindly trace 100%
```

Use:

```text
sampling
tail sampling
aggregation
cardinality control
regional collection
```

---

## 224. Design Large-Scale Logs

```text
Applications
 |
Node Collectors
 |
Buffer
 |
Processors
 |
Elasticsearch
 |
ILM
 |
Kibana
```

Add:

```text
filtering
compression
retention
shard strategy
```

---

## 225. Design Distributed Tracing

```text
Applications
 |
OTel SDK
 |
Regional Collectors
 |
Tail Sampling
 |
Jaeger / Trace Backend
 |
UI
```

---

## 226. Design SLO Platform

```text
SLI
 |
Prometheus
 |
Recording Rules
 |
SLO Calculation
 |
Burn Rate
 |
Alert
 |
Incident
```

---

## 227. Design Observability for Canary

```text
Deployment
 |
Canary
 |
Metrics
 |
Traces
 |
Logs
 |
SLO Analysis
 |
Promote / Pause / Rollback
```

---

## 228. Design Observability During Outage

Prioritize:

```text
customer impact
availability
error rate
latency
recent changes
dependencies
```

Then drill down.

---

# PART CLXXVIII — INTERVIEW ANSWERS

## 229. What Is Observability?

Answer:

```text
Observability is the ability to understand internal system behavior from
telemetry such as metrics, logs and traces.

A mature platform correlates these signals with deployments, infrastructure
events and service ownership so engineers can move from symptom to root cause.
```

---

## 230. Monitoring vs Observability?

Answer:

```text
Monitoring generally detects known failure conditions.

Observability enables investigation of unknown or unexpected behavior by
providing rich, correlated telemetry.
```

---

## 231. Why Prometheus?

Answer:

```text
Prometheus provides a strong time-series metric model, service discovery and
PromQL, making it well suited for Kubernetes and cloud-native monitoring.

At large scale it can be combined with remote storage or horizontally scalable
metric systems.
```

---

## 232. Why OpenTelemetry?

Answer:

```text
OpenTelemetry provides vendor-neutral instrumentation and telemetry
collection.

It reduces tight coupling between application instrumentation and a specific
observability backend.
```

---

## 233. Why Use a Collector?

Answer:

```text
A collector centralizes processing such as batching, filtering, enrichment,
sampling and export.

It also prevents every application from needing direct knowledge of every
backend.
```

---

## 234. Why Not Collect Every Trace?

Answer:

```text
At high traffic, 100% tracing can create substantial CPU, network, storage and
query costs.

Sampling can retain high-value traces such as errors and slow requests while
controlling cost.
```

---

## 235. How Do You Reduce Alert Fatigue?

Answer:

```text
Use actionable alerts, SLO-based alerting, grouping, deduplication,
dependency correlation, severity and clear ownership.

Do not page engineers for every infrastructure fluctuation.
```

---

## 236. How Do You Design Observability for EKS?

Answer:

```text
Collect node, pod, application and control-plane telemetry.

Use Prometheus for metrics, OpenTelemetry for vendor-neutral collection and
trace/log pipelines, Grafana for visualization, and appropriate backends for
logs and traces.

Add SLOs, alerting, ownership, cost controls and multi-cluster aggregation.
```

---

# PART CLXXIX — PRODUCTION RUNBOOKS

## 237. High Error Rate

```text
1. Confirm customer impact.
2. Check error-rate trend.
3. Check recent deployments.
4. Check traces.
5. Inspect correlated logs.
6. Check dependencies.
7. Identify blast radius.
8. Roll back or remediate.
9. Verify recovery.
10. Document incident.
```

---

## 238. High Latency

```text
1. Check P50/P95/P99.
2. Identify affected endpoint.
3. Inspect traces.
4. Find slow span.
5. Check database/cache/external dependency.
6. Check resource saturation.
7. Correlate deployment/configuration.
8. Remediate.
9. Verify SLO.
```

---

## 239. Missing Metrics

```text
1. Check target.
2. Check service discovery.
3. Check scrape health.
4. Check exporter.
5. Check network.
6. Check collector.
7. Check Prometheus.
8. Determine whether data is absent or zero.
9. Restore telemetry.
10. Verify alerts.
```

---

## 240. Missing Logs

```text
1. Check pod stdout.
2. Check node log files.
3. Check collector.
4. Check buffer.
5. Check processor.
6. Check backend ingestion.
7. Check storage.
8. Check permissions.
9. Verify recovery.
```

---

## 241. Missing Traces

```text
1. Check instrumentation.
2. Check context propagation.
3. Check collector receiver.
4. Check sampling.
5. Check exporter.
6. Check backend ingestion.
7. Check query filters.
8. Verify trace IDs.
```

---

## 242. Prometheus High Memory

```text
1. Check active series.
2. Check cardinality.
3. Identify high-cardinality labels.
4. Check scrape frequency.
5. Check target count.
6. Reduce unnecessary series.
7. Add recording rules.
8. Scale appropriately.
9. Verify query performance.
```

---

## 243. Elasticsearch High Disk

```text
1. Check index growth.
2. Check retention.
3. Check shard allocation.
4. Check replica count.
5. Check log volume.
6. Apply lifecycle policy.
7. Remove unnecessary data according to policy.
8. Scale storage if required.
```

---

## 244. Alert Storm

```text
1. Identify common root cause.
2. Group alerts.
3. Suppress dependent alerts.
4. Identify primary service.
5. Notify owner.
6. Restore service.
7. Improve alert dependency logic.
```

---

# PART CLXXX — PRODUCTION CHECKLIST

## 245. Application

```text
[ ] structured logs
[ ] metrics
[ ] trace propagation
[ ] health endpoints
[ ] service metadata
[ ] owner
[ ] dashboard
[ ] runbook
[ ] SLO
```

---

## 246. Kubernetes

```text
[ ] node metrics
[ ] pod metrics
[ ] container logs
[ ] Kubernetes events
[ ] API telemetry
[ ] resource metrics
[ ] network telemetry
[ ] cluster dashboards
```

---

## 247. Observability Platform

```text
[ ] HA
[ ] retention
[ ] backup
[ ] RBAC
[ ] cost controls
[ ] cardinality controls
[ ] telemetry pipeline monitoring
[ ] DR
```

---

# PART CLXXXI — 250 PRODUCTION GOLDEN RULES

## 248. Rules 1–50

```text
1. Observability exists to reduce uncertainty.
2. Monitor customer impact.
3. Use metrics, logs and traces together.
4. Add events where useful.
5. Use business telemetry where appropriate.
6. Define observability requirements before selecting tools.
7. Standardize telemetry metadata.
8. Use service ownership.
9. Use correlation IDs.
10. Propagate trace context.
11. Use structured logs.
12. Never log secrets.
13. Redact sensitive data.
14. Control telemetry access.
15. Treat observability data as sensitive.
16. Use Prometheus for appropriate metric workloads.
17. Use PromQL effectively.
18. Control metric cardinality.
19. Avoid unbounded metric labels.
20. Avoid user IDs as metric labels.
21. Avoid request IDs as metric labels.
22. Avoid raw URLs as metric labels.
23. Use recording rules for expensive repeated queries.
24. Use service discovery.
25. Monitor scrape health.
26. Monitor exporter health.
27. Monitor Prometheus itself.
28. Use Grafana dashboards.
29. Design dashboards around questions.
30. Avoid dashboard overload.
31. Show traffic.
32. Show errors.
33. Show latency.
34. Show saturation.
35. Show dependency health.
36. Show recent deployments.
37. Use SLOs.
38. Define SLIs.
39. Distinguish SLA from SLO.
40. Manage error budgets.
41. Monitor burn rate.
42. Prefer actionable alerts.
43. Avoid alerting on every metric.
44. Group related alerts.
45. Deduplicate alerts.
46. Route alerts by ownership.
47. Give alerts clear severity.
48. Link alerts to runbooks.
49. Link alerts to dashboards.
50. Every paging alert must have an owner.
```

## 249. Rules 51–100

```text
51. Use Alertmanager or an equivalent alert-routing system appropriately.
52. Protect against alert storms.
53. Suppress dependent alerts where appropriate.
54. Prefer symptom-based paging.
55. Use infrastructure metrics for diagnosis.
56. Use structured log schemas.
57. Standardize timestamps.
58. Include service identity.
59. Include environment.
60. Include request correlation.
61. Avoid excessive log volume.
62. Avoid DEBUG logging in normal production operation.
63. Define log retention.
64. Use hot/warm/cold tiers when appropriate.
65. Compress telemetry.
66. Filter unnecessary logs.
67. Protect Elasticsearch.
68. Design shard strategy deliberately.
69. Use index lifecycle management.
70. Protect Kibana/Grafana access.
71. Monitor log ingestion.
72. Monitor dropped logs.
73. Monitor storage growth.
74. Monitor query performance.
75. Use OpenTelemetry for vendor-neutral instrumentation where appropriate.
76. Use collectors for processing and routing.
77. Batch telemetry.
78. Compress telemetry when beneficial.
79. Bound collector queues.
80. Monitor collector memory.
81. Monitor collector CPU.
82. Monitor collector export failures.
83. Monitor dropped spans.
84. Monitor dropped logs.
85. Use DaemonSets for node-local collection where appropriate.
86. Use gateways for centralized processing where appropriate.
87. Avoid a single telemetry gateway failure domain.
88. Scale collectors based on telemetry rate.
89. Use tail sampling for high-value trace retention.
90. Retain errors.
91. Retain slow traces.
92. Retain critical transactions.
93. Avoid blindly tracing 100 percent at massive scale.
94. Protect trace storage.
95. Protect trace query capacity.
96. Use Jaeger or an appropriate tracing backend.
97. Correlate traces with logs.
98. Correlate traces with metrics.
99. Correlate telemetry with deployments.
100. Correlate telemetry with infrastructure events.
```

## 250. Rules 101–150

```text
101. Use exemplars where useful.
102. Propagate trace context across services.
103. Use meaningful span attributes.
104. Avoid sensitive span attributes.
105. Avoid unbounded span attributes.
106. Observe databases.
107. Observe caches.
108. Observe queues.
109. Observe external APIs.
110. Observe DNS where important.
111. Observe load balancers.
112. Observe Kubernetes nodes.
113. Observe Kubernetes pods.
114. Observe Kubernetes containers.
115. Observe Kubernetes control-plane behavior.
116. Observe EKS dependencies.
117. Observe AWS services.
118. Observe autoscaling.
119. Observe deployments.
120. Observe GitOps.
121. Observe CI/CD.
122. Observe Terraform operations.
123. Observe registry operations.
124. Observe secret-management dependencies.
125. Observe observability infrastructure itself.
126. Monitor telemetry pipeline latency.
127. Monitor telemetry pipeline loss.
128. Monitor backend availability.
129. Define telemetry priorities.
130. Protect security/audit telemetry.
131. Do not treat audit logs like ordinary debug logs.
132. Protect telemetry integrity.
133. Protect telemetry confidentiality.
134. Define data retention.
135. Define data deletion.
136. Consider data residency.
137. Minimize PII.
138. Mask sensitive fields.
139. Avoid telemetry becoming a data warehouse.
140. Use tenant isolation when required.
141. Use RBAC.
142. Restrict dashboard access.
143. Restrict backend access.
144. Restrict collector credentials.
145. Use least privilege.
146. Use workload identity where supported.
147. Avoid static observability credentials.
148. Encrypt telemetry in transit.
149. Encrypt sensitive telemetry at rest.
150. Audit privileged observability access.
```

## 251. Rules 151–200

```text
151. Design for backend failure.
152. Use bounded buffering.
153. Use retries with backoff.
154. Avoid infinite retry loops.
155. Use backpressure.
156. Define controlled telemetry dropping.
157. Prioritize high-value telemetry.
158. Monitor queue depth.
159. Monitor export latency.
160. Monitor ingestion latency.
161. Design Prometheus HA where required.
162. Consider remote write for long-term metrics.
163. Consider scalable metric backends at large scale.
164. Use federation deliberately.
165. Design multi-cluster observability.
166. Design multi-account observability.
167. Design multi-region observability.
168. Preserve regional failure isolation.
169. Protect global observability dependencies.
170. Maintain observability configuration as code.
171. Version dashboards.
172. Version alert rules.
173. Version recording rules.
174. Version collector configuration.
175. Review observability changes.
176. Test alert rules.
177. Test PromQL.
178. Test dashboards.
179. Test trace propagation.
180. Test log collection.
181. Test telemetry loss scenarios.
182. Test backend outage scenarios.
183. Test collector failure.
184. Test regional failure.
185. Test cluster failure.
186. Test node failure.
187. Test application failure.
188. Test database failure.
189. Test dependency latency.
190. Test alert routing.
191. Test incident notification.
192. Use synthetic monitoring.
193. Use black-box monitoring.
194. Use white-box monitoring.
195. Keep critical monitoring sufficiently independent.
196. Maintain external probes for critical services.
197. Distinguish zero from missing data.
198. Alert on observability blind spots.
199. Measure observability quality.
200. Measure telemetry completeness.
```

## 252. Rules 201–250

```text
201. Measure MTTD.
202. Measure MTTR.
203. Measure alert noise.
204. Measure false positives.
205. Measure dropped telemetry.
206. Measure telemetry cost.
207. Control cardinality.
208. Control retention.
209. Control query cost.
210. Control ingestion volume.
211. Use sampling deliberately.
212. Use aggregation deliberately.
213. Use compression deliberately.
214. Use tiered storage where appropriate.
215. Do not retain everything forever.
216. Do not page on symptoms without customer impact when avoidable.
217. Do not rely only on CPU.
218. Do not rely only on logs.
219. Do not rely only on metrics.
220. Do not rely only on traces.
221. Correlate all relevant signals.
222. Add deployment markers.
223. Add change metadata.
224. Connect services to repositories.
225. Connect services to owners.
226. Connect services to runbooks.
227. Connect services to SLOs.
228. Build golden dashboards.
229. Build golden alerts.
230. Standardize service telemetry contracts.
231. Provide secure developer instrumentation.
232. Provide platform observability templates.
233. Make observability easy for developers.
234. Avoid requiring every team to build its own platform.
235. Centralize common capabilities.
236. Preserve team ownership.
237. Use risk-based retention.
238. Use customer impact as the highest-level signal.
239. During incidents, establish scope before deep diagnosis.
240. During incidents, check recent changes early.
241. During incidents, correlate dependencies.
242. During incidents, protect telemetry availability.
243. Use observability to validate remediation.
244. Use observability to validate deployments.
245. Use observability to validate canaries.
246. Use observability to validate rollbacks.
247. Continuously improve telemetry based on incidents.
248. Remove telemetry that creates cost without value.
249. Observability is successful when engineers can rapidly move from symptom to
     cause to impact to action.
250. The ultimate goal is a scalable, secure, cost-aware and highly available
     observability platform that makes production behavior understandable and
     supports reliable engineering decisions.
```

# END OF 22-Observability-Architecture.md
