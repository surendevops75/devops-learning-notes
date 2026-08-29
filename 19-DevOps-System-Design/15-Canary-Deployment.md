# 19-DevOps-System-Design
# 15-Canary-Deployment

## 1. Purpose

This is a production-grade Canary Deployment design guide for DevOps,
AWS, Kubernetes, EKS, microservices, CI/CD, GitOps, service mesh,
observability, security and large-scale production platforms.

A canary deployment introduces a new version to a small, controlled portion
of production traffic before increasing exposure.

Basic model:

```text
                 Production Traffic
                        |
                 Traffic Controller
                    /          \
                 BLUE          CANARY
                99%             1%
                                |
                              vNext
```

Then:

```text
99/1
 |
95/5
 |
90/10
 |
75/25
 |
50/50
 |
100/0
```

The percentages are examples only.

The central objective is:

```text
Detect release problems while blast radius is still small.
```

A mature canary system does not simply shift percentages. It continuously
answers:

```text
Is the canary healthy?
Is it better than baseline?
Is customer experience degrading?
Are business transactions healthy?
Should traffic increase?
Should the rollout pause?
Should it rollback?
```

---

# PART I — CANARY FOUNDATIONS

## 2. What Is Canary Deployment?

Canary deployment releases a new version to a limited production audience
while the previous version continues serving the majority of traffic.

Example:

```text
BLUE  -> 95%
GREEN -> 5%
```

The new version is observed before full promotion.

---

## 3. Why Canary?

Primary objectives:

```text
reduce blast radius
validate production behavior
detect regressions
validate performance
validate dependencies
enable controlled rollback
```

---

## 4. Canary vs Blue-Green

Blue-green:

```text
BLUE  -> 100%
GREEN -> 0%

switch

BLUE  -> 0%
GREEN -> 100%
```

Canary:

```text
BLUE  -> 99%
GREEN -> 1%
```

then progressively increases exposure.

---

## 5. Canary vs Rolling

Rolling deployment changes instances gradually.

Canary usually introduces an explicitly controlled traffic population and
measures the candidate against a baseline.

---

# PART II — CANARY PRINCIPLES

## 6. Small Blast Radius

Start with:

```text
1%
```

or another percentage appropriate to the workload.

Do not assume a fixed percentage is safe for every system.

---

## 7. Production Reality

Staging cannot reproduce every:

```text
customer
request
dependency
data pattern
traffic spike
network condition
```

Canary provides production evidence.

---

## 8. Statistical Confidence

A 1% canary with only a few requests may not provide meaningful evidence.

You need enough:

```text
traffic
duration
transactions
```

to make a useful decision.

---

# PART III — CANARY DESIGN

## 9. Reference Architecture

```text
Users
  |
Load Balancer / Mesh
  |
Traffic Splitter
  |
+-------------------+
|                   |
Baseline           Canary
v1                 v2
|                   |
+---------+---------+
          |
      Dependencies
          |
      DB / Cache / Queue
          |
     Observability
          |
   Automated Analysis
          |
Promotion / Rollback
```

---

## 10. Main Components

A mature canary system has:

```text
candidate workload
traffic controller
metrics
baseline
analysis engine
promotion policy
rollback mechanism
```

---

# PART IV — TRAFFIC SPLITTING

## 11. Weight-Based Routing

Example:

```text
v1 -> 95
v2 -> 5
```

Weights can be changed automatically.

---

## 12. Header-Based Routing

Example:

```text
X-Canary: true
```

routes selected requests to canary.

Useful for:

```text
internal users
QA
employees
synthetic clients
```

---

## 13. Cookie-Based Routing

A cookie can keep a user consistently assigned to canary.

This is useful when testing user journeys.

---

## 14. User-Based Routing

Examples:

```text
customer cohort
tenant
account
region
```

Canary can target specific populations.

---

# PART V — RANDOM CANARY

## 15. Random Percentage

Traffic is distributed approximately according to weights.

Example:

```text
100,000 requests
5% canary
```

Expected canary volume is approximately:

```text
5,000 requests
```

Actual distribution can vary.

---

# PART VI — STICKY CANARY

## 16. User Stickiness

Without stickiness:

```text
request 1 -> canary
request 2 -> baseline
```

This can make user-level testing difficult.

With deterministic routing:

```text
user A -> canary
user A -> canary
user A -> canary
```

This is useful for workflows.

---

# PART VII — CANARY SELECTION

## 17. Who Gets Canary?

Possible strategies:

```text
random users
internal users
low-risk tenants
specific region
specific account
specific device
specific API route
```

Choose based on risk.

---

# PART VIII — RISK-BASED CANARY

## 18. High-Risk Customers

Do not necessarily expose critical enterprise customers first.

Start with:

```text
low-risk population
```

then expand.

---

# PART IX — REGIONAL CANARY

## 19. Region-Based

```text
Region A -> canary
Region B -> baseline
Region C -> baseline
```

Useful for large global platforms.

---

# PART X — CELL CANARY

## 20. Cell Architecture

```text
Global
 |
+-- Cell A -> canary
+-- Cell B -> baseline
+-- Cell C -> baseline
```

Failure is isolated to one cell.

---

# PART XI — KUBERNETES

## 21. Basic Kubernetes Model

```text
Deployment baseline
Deployment canary
```

Example:

```text
checkout-v1
checkout-v2
```

---

## 22. Service Routing

A normal Kubernetes Service can expose both versions, but percentage-based
routing requires an additional traffic-control mechanism.

Possible mechanisms:

```text
Ingress
service mesh
Gateway API
Argo Rollouts
cloud load balancer
```

---

# PART XII — ARGO ROLLOUTS

## 23. Canary Controller

Argo Rollouts can manage progressive delivery patterns for Kubernetes.

Concept:

```text
stable
 |
canary
 |
setWeight
 |
pause
 |
analysis
 |
promote
```

---

## 24. Example Concept

```yaml
strategy:
  canary:
    steps:
      - setWeight: 5
      - pause: {}
      - setWeight: 25
      - pause: {}
      - setWeight: 50
      - pause: {}
```

The actual configuration must match the chosen traffic provider.

---

# PART XIII — ANALYSIS

## 25. Automated Analysis

A mature canary uses metrics to decide:

```text
continue
pause
rollback
```

---

## 26. Metrics

Common signals:

```text
error rate
latency
throughput
CPU
memory
restarts
saturation
dependency errors
```

---

# PART XIV — GOLDEN SIGNALS

## 27. Traffic

Measure:

```text
requests/sec
```

---

## 28. Errors

Measure:

```text
5xx
4xx where meaningful
application exceptions
failed business transactions
```

---

## 29. Latency

Measure:

```text
P50
P95
P99
```

P99 often reveals tail latency regressions that averages hide.

---

## 30. Saturation

Measure:

```text
CPU
memory
connections
queue depth
IOPS
network
```

---

# PART XV — BUSINESS METRICS

## 31. Technical Metrics Are Not Enough

Example:

```text
HTTP 200 -> healthy
```

but:

```text
checkout success -> down
```

The canary is not healthy from a business perspective.

---

## 32. Business Metrics

Possible:

```text
login success
checkout success
payment success
order creation
search success
conversion
```

---

# PART XVI — BASELINE COMPARISON

## 33. Canary vs Baseline

Do not evaluate canary only against a fixed threshold.

Compare:

```text
canary error rate
vs
baseline error rate
```

Example:

```text
baseline = 0.4%
canary   = 2.0%
```

This is a strong warning even if an absolute threshold is 3%.

---

# PART XVII — RELATIVE METRICS

## 34. Error Ratio

Concept:

```text
canary_error / baseline_error
```

Example:

```text
2.0 / 0.4 = 5x
```

The candidate is performing substantially worse.

---

# PART XVIII — LATENCY COMPARISON

## 35. Tail Latency

Example:

```text
baseline P99 = 250 ms
canary P99   = 700 ms
```

The release may have a performance regression.

---

# PART XIX — SAMPLE SIZE

## 36. Small Sample Problem

Suppose canary receives:

```text
20 requests
```

and two fail.

That is:

```text
10%
```

but the estimate has enormous uncertainty.

Wait for enough meaningful traffic before making some decisions.

---

# PART XX — ANALYSIS WINDOW

## 37. Window

A canary may require:

```text
minimum requests
+
minimum duration
```

before analysis.

---

# PART XXI — PROMOTION

## 38. Promotion Conditions

Example:

```text
canary error ratio < threshold
AND
P99 regression < threshold
AND
business success > threshold
AND
minimum request count reached
```

---

# PART XXII — ROLLBACK

## 39. Rollback

If analysis fails:

```text
Canary
 |
traffic -> 0%
 |
Baseline -> 100%
```

---

# PART XXIII — PAUSE

## 40. Pause

Use a pause when:

```text
traffic stage reached
 |
collect data
 |
human review
```

Useful for high-risk changes.

---

# PART XXIV — AUTOMATED ROLLBACK

## 41. Automated

```text
5% canary
 |
analysis
 |
failure
 |
rollback
```

Fast automated rollback reduces blast radius.

---

# PART XXV — ROLLBACK SAFETY

## 42. Database Compatibility

Rollback can fail if:

```text
canary changed schema incompatibly
```

Use:

```text
expand-contract
```

where possible.

---

# PART XXVI — DATABASE MIGRATION

## 43. Safe Pattern

```text
expand
 |
deploy compatible code
 |
canary
 |
promote
 |
contract later
```

---

# PART XXVII — SHARED DATABASE

## 44. Shared DB

```text
Baseline
   |
   +------ DB
   |
Canary
```

Both versions must understand the same schema.

---

# PART XXVIII — SEPARATE DATABASE

## 45. Separate DB

```text
Baseline -> DB A
Canary   -> DB B
```

This gives stronger isolation but requires:

```text
data replication
consistency
cutover
```

---

# PART XXIX — CACHE

## 46. Cache Compatibility

Canary may encounter baseline cache entries.

Use:

```text
versioned keys
backward-compatible serialization
```

where required.

---

# PART XXX — SESSIONS

## 47. Session Routing

If canary users need consistent behavior:

```text
user
 |
consistent assignment
 |
canary
```

Use appropriate routing/session design.

---

# PART XXXI — QUEUES

## 48. Canary Workers

Example:

```text
Queue
 |
+-- baseline workers
+-- canary workers
```

Both consumers must safely process the same message format unless routing
isolates message populations.

---

# PART XXXII — CRON

## 49. Duplicate Jobs

Do not allow:

```text
baseline cron
+
canary cron
```

to execute the same production task unintentionally.

---

# PART XXXIII — IDEMPOTENCY

## 50. Event Safety

Canary systems should tolerate:

```text
duplicate event
retry
replay
```

where appropriate.

---

# PART XXXIV — EKS CAPACITY

## 51. Canary Capacity

If baseline has:

```text
100 pods
```

canary might start with:

```text
5 pods
```

but actual capacity must account for:

```text
request distribution
pod limits
autoscaling
failure
```

---

# PART XXXV — HPA

## 52. HPA

HPA can scale canary independently.

Be careful with low traffic:

```text
small traffic
 |
unstable metric
 |
incorrect scaling
```

---

# PART XXXVI — KARPENTER / NODE AUTOSCALING

## 53. Node Capacity

Canary rollout can cause a sudden scheduling requirement.

Node provisioning time must be considered in rollout timing.

---

# PART XXXVII — POD STARTUP

## 54. Cold Start

Canary may take time to become production-ready because of:

```text
image pull
JVM warm-up
cache loading
connection pools
model loading
```

---

# PART XXXVIII — READINESS

## 55. Readiness Probe

Readiness should prevent traffic before the application is capable of handling
requests.

But readiness alone does not prove business health.

---

# PART XXXIX — LIVENESS

## 56. Liveness

Liveness should identify a process that must be restarted.

Do not use overly aggressive probes that cause restart loops.

---

# PART XL — STARTUP PROBE

## 57. Startup

Slow-starting applications may need startup probes so liveness checks do not
restart them prematurely.

---

# PART XLI — NETWORKING

## 58. Traffic Path

Example:

```text
Route 53
 |
ALB
 |
Ingress
 |
Service Mesh
 |
Baseline / Canary
```

Understand exactly where the split occurs.

---

# PART XLII — ALB

## 59. ALB Weighting

ALB listener rules and target groups can be used for controlled traffic
patterns depending on architecture.

Validate actual distribution and connection behavior.

---

# PART XLIII — ROUTE 53

## 60. DNS Canary

DNS-based canary:

```text
DNS
 |
95% baseline
5% canary
```

But DNS routing is coarse compared with per-request traffic splitting and
is affected by caching.

---

# PART XLIV — SERVICE MESH

## 61. Mesh

A service mesh can provide:

```text
weighted routing
header routing
subset routing
retry
timeout
telemetry
```

Use carefully because mesh complexity can become significant.

---

# PART XLV — GATEWAY API

## 62. Gateway Routing

Gateway APIs can represent more explicit traffic-routing policy than simple
Service selectors.

---

# PART XLVI — ARGO CD

## 63. GitOps

Example:

```text
Git
 |
Argo CD
 |
Argo Rollouts
 |
EKS
```

Git stores desired deployment configuration while the rollout controller
handles progressive execution.

---

# PART XLVII — GITOPS PROMOTION

## 64. Promotion

Possible state:

```text
candidate
 |
5%
 |
25%
 |
50%
 |
100%
```

Promotion state should be auditable.

---

# PART XLVIII — CI/CD

## 65. Pipeline

```text
commit
 |
build
 |
test
 |
security
 |
artifact
 |
deploy canary
 |
analysis
 |
promotion
```

---

# PART XLIX — BUILD ONCE

## 66. Immutable Artifact

Use:

```text
build once
scan once
sign
promote
```

Do not rebuild a different binary between canary and full production.

---

# PART L — SECURITY

## 67. Canary Security

Canary must use production-equivalent:

```text
IAM
RBAC
network policy
TLS
WAF
security groups
secrets
```

---

# PART LI — SECURITY REGRESSION

## 68. Canary Security Testing

Validate:

```text
authorization
authentication
input validation
headers
TLS
dependency security
```

---

# PART LII — SECRET ROTATION

## 69. Secret Compatibility

If canary expects a new credential:

```text
provision
 |
validate
 |
canary
 |
promote
```

Avoid breaking baseline during transition.

---

# PART LIII — OBSERVABILITY

## 70. Canary Labels

Add dimensions such as:

```text
version
deployment
environment
region
```

This allows comparison.

---

# PART LIV — METRIC CARDINALITY

## 71. Cardinality

Avoid labels with unbounded values such as:

```text
user_id
request_id
```

on high-volume metrics.

Canary analysis depends on reliable observability.

---

# PART LV — LOGGING

## 72. Logs

Separate or filter:

```text
baseline logs
canary logs
```

so regressions are visible.

---

# PART LVI — TRACING

## 73. Distributed Tracing

Trace canary requests across:

```text
gateway
service
database
external API
```

This can reveal dependency regressions.

---

# PART LVII — ERROR BUDGET

## 74. Canary and SLO

Canary analysis can compare impact against:

```text
SLO
error budget
```

Do not burn a large portion of the error budget merely to complete a release.

---

# PART LVIII — SLO GATES

## 75. Gate

Example:

```text
if canary P99 > baseline P99 * 1.2:
    rollback
```

This is conceptual; real thresholds must be workload-specific.

---

# PART LIX — MULTI-METRIC ANALYSIS

## 76. Avoid One-Metric Decisions

A release can have:

```text
low CPU
```

but:

```text
high latency
```

Use multiple meaningful signals.

---

# PART LX — CORRELATED FAILURES

## 77. Dependency Failure

If both baseline and canary suddenly fail because:

```text
database outage
```

do not automatically blame canary.

Compare control and treatment.

---

# PART LXI — BASELINE HEALTH

## 78. Baseline Must Be Healthy

If baseline is already failing:

```text
canary vs broken baseline
```

comparison becomes difficult.

---

# PART LXII — ANALYSIS CONTAMINATION

## 79. Shared Dependencies

Both versions may share:

```text
database
cache
queue
external API
```

A dependency issue can affect both.

Canary analysis must account for shared failure domains.

---

# PART LXIII — CANARY DATA

## 80. Data Distribution

A 5% random request split does not guarantee:

```text
5% of enterprise customers
5% of high-value transactions
```

Understand traffic composition.

---

# PART LXIV — COHORT BIAS

## 81. Example

Suppose canary receives mostly:

```text
mobile traffic
```

while baseline receives:

```text
desktop traffic
```

Performance comparisons can be biased.

---

# PART LXV — CONTROL GROUP

## 82. Control

Baseline is the control group.

Keep enough baseline traffic during analysis to establish a useful comparison.

---

# PART LXVI — STATISTICAL THINKING

## 83. Noise

Metrics naturally fluctuate.

Avoid rollback because of one random spike unless the risk warrants immediate
action.

---

# PART LXVII — CONFIDENCE

## 84. Evidence

Canary promotion should depend on:

```text
sample size
duration
effect size
business impact
```

not merely:

```text
one green dashboard
```

---

# PART LXVIII — PROMOTION SPEED

## 85. Slow vs Fast

Fast:

```text
5%
 |
25%
 |
50%
 |
100%
```

Slow:

```text
1%
 |
5%
 |
10%
 |
25%
 |
50%
 |
100%
```

Higher-risk changes generally benefit from more cautious progression.

---

# PART LXIX — ROLLBACK SPEED

## 86. Rollback Target

Define:

```text
rollback < X minutes
```

and test it.

---

# PART LXX — TRAFFIC DRAIN

## 87. Existing Connections

Changing weights may not immediately terminate:

```text
WebSockets
gRPC
long HTTP requests
```

Plan draining and reconnection.

---

# PART LXXI — RATE LIMITING

## 88. Canary Rate Limits

Ensure canary traffic does not accidentally trigger:

```text
API rate limits
database limits
third-party quotas
```

---

# PART LXXII — EXTERNAL APIS

## 89. Third-Party Calls

A canary can multiply external API traffic if:

```text
retry behavior
fan-out
```

changes.

Monitor external dependencies.

---

# PART LXXIII — RETRIES

## 90. Retry Storm

Bad canary:

```text
failure
 |
retry
 |
retry
 |
retry
```

can amplify load.

Use bounded retries and backoff.

---

# PART LXXIV — TIMEOUTS

## 91. Timeout Changes

A longer timeout can improve success rate while dramatically increasing
resource consumption.

Monitor:

```text
latency
connections
thread pools
```

---

# PART LXXV — CIRCUIT BREAKERS

## 92. Circuit Breaking

Canary may need compatible:

```text
timeout
retry
circuit breaker
```

behavior.

---

# PART LXXVI — FEATURE FLAGS

## 93. Canary + Feature Flags

Deployment:

```text
code available
```

Feature:

```text
disabled
```

Then:

```text
enable for canary
```

This allows more controlled experimentation.

---

# PART LXXVII — EXPERIMENTATION

## 94. Canary as Experiment

Track:

```text
hypothesis
population
metric
duration
decision
```

This prevents subjective promotion.

---

# PART LXXVIII — BUSINESS SAFETY

## 95. Irreversible Actions

Be careful when canary performs:

```text
payments
deletions
financial transactions
external writes
```

Use safe test data and controlled cohorts.

---

# PART LXXIX — BACKGROUND JOBS

## 96. Worker Canary

Canary worker percentage should be based on:

```text
queue throughput
processing time
message compatibility
```

not only HTTP traffic.

---

# PART LXXX — CRON OWNERSHIP

## 97. Scheduler

Use:

```text
single active scheduler
```

or another explicit ownership mechanism.

---

# PART LXXXI — STORAGE

## 98. File Writes

If canary writes files differently:

```text
baseline reader
canary writer
```

compatibility must be validated.

---

# PART LXXXII — API CONTRACTS

## 99. Backward Compatibility

Canary changes should preserve API contracts during the transition.

---

# PART LXXXIII — EVENT CONTRACTS

## 100. Event Compatibility

New producers and old consumers may coexist.

Use additive changes where possible.

---

# PART LXXXIV — MULTI-SERVICE CANARY

## 101. Dependency Chain

If:

```text
Frontend canary
 |
API baseline
```

test compatibility.

If:

```text
API canary
 |
Database baseline
```

same principle.

---

# PART LXXXV — CASCADING CANARIES

## 102. Service-by-Service

For microservices:

```text
Service A -> 5%
 |
Service B -> baseline
```

Later:

```text
Service B -> 5%
```

Coordinate dependencies carefully.

---

# PART LXXXVI — SERVICE MESH CANARY

## 103. Subset Routing

Concept:

```text
Service
 |
+-- stable subset
+-- canary subset
```

Route using:

```text
weight
headers
cookies
```

---

# PART LXXXVII — MULTI-CLUSTER

## 104. Cluster Canary

```text
Cluster A -> canary
Cluster B -> stable
```

Useful when cluster-level isolation is required.

---

# PART LXXXVIII — MULTI-REGION

## 105. Region Canary

```text
Region A -> canary
Region B -> stable
```

Validate:

```text
latency
business traffic
data replication
```

---

# PART LXXXIX — DATA REPLICATION

## 106. Canary and Replicas

If canary reads from a replica:

```text
replica lag
```

can create misleading behavior.

Monitor replication.

---

# PART XC — CACHE WARMING

## 107. Warm Before Traffic

For expensive caches:

```text
deploy canary
 |
warm cache
 |
validate
 |
traffic
```

---

# PART XCI — LOAD TEST

## 108. Pre-Canary Load

Use realistic load testing before production exposure where feasible.

---

# PART XCII — CHAOS

## 109. Failure Testing

Canary should survive expected:

```text
pod failure
node failure
dependency latency
network errors
```

according to reliability objectives.

---

# PART XCIII — INCIDENT DURING CANARY

## 110. Response

```text
pause
 |
assess
 |
rollback if needed
 |
protect customer impact
 |
investigate
```

---

# PART XCIV — FALSE POSITIVES

## 111. Alert Noise

If canary analysis rolls back constantly due to noisy metrics:

```text
deployment velocity
```

will suffer.

Tune:

```text
threshold
window
sample size
baseline comparison
```

---

# PART XCV — FALSE NEGATIVES

## 112. Missed Failure

A canary can appear healthy while missing:

```text
rare customer workflow
rare data shape
```

Use targeted cohorts and synthetic tests.

---

# PART XCVI — ROLLOUT PAUSE

## 113. Manual Pause

Useful during:

```text
business peak
market event
migration
high-risk release
```

---

# PART XCVII — RELEASE FREEZE

## 114. Freeze Windows

Avoid canary releases during periods when:

```text
incident
major campaign
holiday peak
```

unless explicitly required.

---

# PART XCVIII — COST

## 115. Canary Cost

Canary requires additional:

```text
compute
observability
analysis
traffic-control
```

Cost is usually lower than permanent duplicate environments but still exists.

---

# PART XCIX — SECURITY OF ROLLOUT CONTROLLER

## 116. Controller Permissions

The rollout controller may need permissions to:

```text
change replicas
modify traffic
read metrics
```

Grant only required permissions.

---

# PART C — AUDIT

## 117. Release Audit

Record:

```text
commit
image digest
canary start
traffic weights
analysis result
promotion
rollback
operator
```

---

# PART CI — PRODUCTION DASHBOARD

## 118. Dashboard

Show side-by-side:

```text
baseline
canary
```

for:

```text
traffic
errors
latency
saturation
business KPIs
```

---

# PART CII — ALERTS

## 119. Canary Alerts

Examples:

```text
canary error rate high
canary P99 regression
canary restart rate high
canary business failure
canary dependency failures
```

---

# PART CIII — DEPLOYMENT STATES

## 120. State Machine

Concept:

```text
Pending
 |
Progressing
 |
Paused
 |
Analyzing
 |
Promoted
```

Failure:

```text
Analyzing
 |
Failed
 |
Rolled Back
```

---

# PART CIV — CANARY STATE

## 121. Desired State

Git can define:

```text
image
steps
weights
analysis
```

The controller executes the progressive process.

---

# PART CV — RECOVERY

## 122. Controller Failure

If the rollout controller fails, traffic should remain in a safe state.

Avoid designs where controller failure automatically creates uncontrolled traffic.

---

# PART CVI — LOAD BALANCER FAILURE

## 123. Traffic Layer Failure

Traffic switching depends on:

```text
ALB
Ingress
mesh
DNS
```

These are part of the canary failure domain.

---

# PART CVII — OBSERVABILITY FAILURE

## 124. Metrics Unavailable

If analysis cannot obtain reliable metrics:

```text
pause
```

rather than blindly promote.

---

# PART CVIII — BASELINE DISAPPEARS

## 125. Control Loss

If baseline is removed too early:

```text
rollback
```

may no longer be immediate.

Keep stable capacity until promotion confidence is established.

---

# PART CIX — CANARY CLEANUP

## 126. After Promotion

```text
canary -> stable
old baseline -> previous
```

Then:

```text
remove old workload
```

after rollback policy allows.

---

# PART CX — LARGE FLEET

## 127. Fleet Strategy

For hundreds of clusters:

```text
global
 |
wave 1
 |
wave 2
 |
wave 3
 |
wave N
```

Use progressive rollout across failure domains.

---

# PART CXI — REGIONAL WAVES

## 128. Example

```text
Region A -> 5%
validate
Region B -> 5%
validate
Region C -> 5%
```

Then expand each region.

---

# PART CXII — TENANT CANARY

## 129. SaaS

Canary can target:

```text
tenant cohort
```

This can provide controlled business exposure.

---

# PART CXIII — INTERNAL CANARY

## 130. Employees First

```text
internal users -> canary
external users -> stable
```

Useful for UX and compatibility validation.

---

# PART CXIV — API CANARY

## 131. Route-Specific

Canary only:

```text
/new-api
```

while keeping:

```text
/legacy-api
```

stable.

Useful for high-risk endpoints.

---

# PART CXV — MOBILE CLIENTS

## 132. Compatibility

Mobile clients may remain on old versions for months.

Backend canary must preserve compatibility with supported client versions.

---

# PART CXVI — DATABASE VERSIONING

## 133. Compatibility Matrix

Maintain:

```text
client
service
schema
event
```

compatibility during the canary window.

---

# PART CXVII — MIGRATIONS

## 134. Migration Rule

Avoid:

```text
canary starts
 |
destructive migration
 |
rollback required
```

without a safe recovery path.

---

# PART CXVIII — DATA CORRUPTION

## 135. Canary Detection

Monitor:

```text
business invariants
data validation
```

A release can return successful HTTP responses while writing incorrect data.

---

# PART CXIX — SHADOW WRITE

## 136. Shadow Systems

If writes are mirrored for testing:

```text
production write
 |
shadow target
```

protect against unintended side effects.

---

# PART CXX — OBSERVABILITY QUALITY

## 137. Metric Correctness

Canary analysis is only as reliable as:

```text
instrumentation
aggregation
labels
sampling
```

Validate the measurement system.

---

# PART CXXI — SAMPLING

## 138. Trace Sampling

If traces are sampled too aggressively, rare canary failures may disappear.

Use appropriate sampling for high-risk release analysis.

---

# PART CXXII — LOG SAMPLING

## 139. Logs

Avoid aggressive log dropping during canary analysis if it hides important
failure evidence.

---

# PART CXXIII — SLO

## 140. SLO-Based Promotion

Promotion should respect:

```text
availability
latency
business reliability
```

---

# PART CXXIV — ERROR BUDGET

## 141. Budget

If a release consumes excessive error budget:

```text
stop promotion
```

---

# PART CXXV — RELEASE RISK

## 142. Risk Factors

Higher-risk releases include:

```text
database migration
authentication
payment
networking
security
large dependency change
```

Use smaller canary steps and longer analysis windows.

---

# PART CXXVI — LOW-RISK RELEASE

## 143. Low Risk

A documentation-only or isolated UI change may use a simpler rollout.

---

# PART CXXVII — HIGH-RISK RELEASE

## 144. High Risk

For payment/authentication changes:

```text
1%
 |
analysis
 |
5%
 |
analysis
 |
10%
```

with strong business validation.

---

# PART CXXVIII — ROLLOUT TIME

## 145. Total Time

Include:

```text
startup
warm-up
analysis
pause
traffic change
```

Do not define rollout duration only by controller step delays.

---

# PART CXXIX — PRODUCTION EXAMPLE

## 146. E-Commerce API

Initial:

```text
v1 -> 99%
v2 -> 1%
```

Metrics:

```text
v1 error = 0.4%
v2 error = 0.5%
```

Continue.

Next:

```text
v1 -> 95%
v2 -> 5%
```

If:

```text
v2 error = 3%
```

rollback:

```text
v1 -> 100%
v2 -> 0%
```

---

# PART CXXX — PAYMENT EXAMPLE

## 147. Payment Service

Use:

```text
1% internal/test cohort
 |
5%
 |
10%
```

Monitor:

```text
authorization success
declines
timeouts
duplicate charges
latency
```

Never rely only on HTTP status.

---

# PART CXXXI — FAILURE EXAMPLE

## 148. Memory Leak

Canary:

```text
memory steadily increases
```

Baseline:

```text
stable
```

Analysis detects the divergence before full promotion.

---

# PART CXXXII — FAILURE EXAMPLE

## 149. Latency Regression

```text
baseline P99 = 200ms
canary P99 = 900ms
```

Pause/rollback depending on defined policy.

---

# PART CXXXIII — FAILURE EXAMPLE

## 150. Shared Database Outage

Both:

```text
baseline errors ↑
canary errors ↑
```

Do not conclude the canary caused the outage.

---

# PART CXXXIV — FAILURE EXAMPLE

## 151. Rare Workflow

Canary technical metrics look normal, but:

```text
enterprise export failure
```

Use targeted business cohorts and synthetic workflows.

---

# PART CXXXV — SENIOR DESIGN

## 152. Design Canary for 100K RPS

Requirements:

```text
100K RPS
99.99%
rollback < 2 minutes
```

Architecture:

```text
Global Traffic
 |
Regional LB
 |
Mesh / Gateway
 |
Stable + Canary
 |
Observability
 |
Automated Analysis
```

Need:

```text
small initial blast radius
high-volume metrics
capacity headroom
fast traffic reversal
```

---

# PART CXXXVI — SENIOR DESIGN

## 153. Design Canary for EKS

Components:

```text
EKS
 |
Argo Rollouts
 |
Ingress / Mesh
 |
Prometheus
 |
Analysis
 |
Argo CD
```

---

# PART CXXXVII — SENIOR DESIGN

## 154. Design Multi-Region Canary

```text
Global
 |
Region A -> canary
Region B -> stable
Region C -> stable
```

After validation:

```text
Region B -> canary
```

Use regional waves.

---

# PART CXXXVIII — SENIOR DESIGN

## 155. Design Canary With Database Migration

Use:

```text
expand
 |
compatible application
 |
canary
 |
promotion
 |
contract
```

Do not make rollback impossible before canary evidence is collected.

---

# PART CXXXIX — SENIOR DESIGN

## 156. Design Canary for Stateful Workers

Define:

```text
message compatibility
worker percentage
partition ownership
idempotency
duplicate handling
```

---

# PART CXL — SENIOR DESIGN

## 157. Design Canary for WebSockets

Need:

```text
sticky assignment
connection draining
reconnect
session state
```

---

# PART CXLI — SENIOR DESIGN

## 158. Design Canary for SaaS

Use:

```text
tenant cohorts
```

and ensure:

```text
tenant isolation
business metrics
```

are represented in analysis.

---

# PART CXLII — SENIOR DESIGN

## 159. Design Global Fleet Canary

For:

```text
1,000 clusters
```

use:

```text
wave 1
wave 2
wave 3
```

with automatic stop on correlated failure.

---

# PART CXLIII — INTERVIEW FRAMEWORK

## 160. Senior Canary Answer

Use:

```text
1. Define blast radius.
2. Define candidate and baseline.
3. Choose traffic split mechanism.
4. Define initial percentage.
5. Define metrics.
6. Define baseline comparison.
7. Define minimum sample/window.
8. Define promotion gates.
9. Define rollback gates.
10. Address database compatibility.
11. Address sessions.
12. Address queues/jobs.
13. Address capacity.
14. Address observability.
15. Address security.
16. Address multi-region behavior.
17. Explain failure scenarios.
18. Explain cost.
```

---

# PART CXLIV — PRODUCTION RUNBOOK

## 161. Before Canary

```text
[ ] artifact immutable
[ ] security checks passed
[ ] database compatibility checked
[ ] configuration ready
[ ] secrets ready
[ ] capacity available
[ ] observability ready
[ ] baseline healthy
[ ] rollback tested
[ ] business metrics defined
```

---

## 162. Start Canary

```text
1. Deploy candidate.
2. Wait for readiness.
3. Validate health.
4. Start small traffic percentage.
5. Confirm actual traffic split.
6. Begin analysis window.
```

---

## 163. Analyze

```text
1. Compare traffic.
2. Compare errors.
3. Compare P99.
4. Compare saturation.
5. Compare dependency behavior.
6. Compare business metrics.
7. Check sample size.
8. Check analysis result.
```

---

## 164. Promote

```text
1. Increase weight.
2. Wait.
3. Analyze.
4. Increase again.
5. Repeat.
6. Reach 100%.
7. Retain previous version for rollback window.
```

---

## 165. Rollback

```text
1. Detect failure.
2. Stop progression.
3. Set canary traffic to zero.
4. Return traffic to baseline.
5. Drain candidate.
6. Preserve evidence.
7. Investigate.
8. Fix.
9. Retest.
```

---

# PART CXLV — 250 PRODUCTION GOLDEN RULES

## 166. Rules 1–50

```text
1. Define the canary objective.
2. Define blast radius.
3. Define baseline.
4. Define candidate.
5. Define traffic mechanism.
6. Start with an intentionally small exposure.
7. Do not assume 1% is universally safe.
8. Ensure enough traffic for useful analysis.
9. Ensure enough duration for useful analysis.
10. Use production evidence.
11. Compare canary with baseline.
12. Measure error rate.
13. Measure latency.
14. Measure P99.
15. Measure saturation.
16. Measure throughput.
17. Measure business success.
18. Measure dependency health.
19. Define promotion gates.
20. Define rollback gates.
21. Automate analysis where reliable.
22. Pause when evidence is insufficient.
23. Roll back on defined severe regressions.
24. Avoid noisy rollback policies.
25. Avoid false confidence.
26. Keep baseline healthy.
27. Keep baseline available for rollback.
28. Do not remove baseline too early.
29. Build once.
30. Promote the same artifact.
31. Use immutable image identity.
32. Validate security before canary.
33. Validate configuration.
34. Validate secrets.
35. Validate certificates.
36. Validate IAM.
37. Validate RBAC.
38. Validate network policies.
39. Validate external dependencies.
40. Validate database compatibility.
41. Validate cache compatibility.
42. Validate queue compatibility.
43. Validate API compatibility.
44. Validate event compatibility.
45. Validate background jobs.
46. Validate cron ownership.
47. Validate sessions.
48. Validate long-lived connections.
49. Validate business workflows.
50. Keep rollback fast.
```

## 167. Rules 51–100

```text
51. Use enough canary capacity.
52. Protect baseline capacity.
53. Include startup time.
54. Include image pull time.
55. Include cache warm-up.
56. Include connection-pool warm-up.
57. Include autoscaling delay.
58. Include analysis duration.
59. Include traffic convergence.
60. Measure real rollout time.
61. Use readiness probes.
62. Use startup probes when required.
63. Use liveness carefully.
64. Do not treat readiness as business health.
65. Use synthetic checks.
66. Use business metrics.
67. Use technical metrics.
68. Use dependency metrics.
69. Use side-by-side dashboards.
70. Label metrics by release version.
71. Avoid unbounded metric cardinality.
72. Protect observability capacity.
73. Validate metric correctness.
74. Validate trace sampling.
75. Validate log visibility.
76. Keep analysis independent enough to detect failure.
77. Monitor shared dependencies.
78. Understand control/treatment contamination.
79. Do not blame canary for shared dependency outages.
80. Compare failure rates.
81. Compare tail latency.
82. Compare resource usage.
83. Compare business outcomes.
84. Use minimum sample counts.
85. Use analysis windows.
86. Use confidence appropriately.
87. Account for natural variance.
88. Avoid one-metric promotion.
89. Avoid one-spike rollback without context when risk permits.
90. Use multiple signals.
91. Use error ratios.
92. Use SLOs.
93. Use error budgets.
94. Stop releases that create unacceptable reliability impact.
95. Use risk-based rollout speed.
96. Use smaller steps for high-risk changes.
97. Use longer observation for high-risk changes.
98. Use targeted cohorts when necessary.
99. Avoid biased cohorts.
100. Ensure the canary represents the risk being tested.
```

## 168. Rules 101–150

```text
101. Use header routing for controlled users.
102. Use cookie routing for consistent sessions.
103. Use tenant routing when appropriate.
104. Use regional routing when appropriate.
105. Use cell routing for large platforms.
106. Use weighted routing for broad exposure.
107. Understand DNS limitations.
108. Understand load-balancer behavior.
109. Understand connection persistence.
110. Understand service-mesh behavior.
111. Understand retry behavior.
112. Understand timeout behavior.
113. Understand circuit breakers.
114. Prevent retry storms.
115. Prevent canary overload.
116. Protect third-party quotas.
117. Protect database connections.
118. Protect database capacity.
119. Protect cache capacity.
120. Protect queue capacity.
121. Protect node capacity.
122. Protect IP capacity.
123. Protect network capacity.
124. Protect observability capacity.
125. Avoid duplicate cron execution.
126. Avoid duplicate workers when unsafe.
127. Use idempotent processing.
128. Handle duplicate messages.
129. Handle replay.
130. Version event schemas.
131. Prefer backward-compatible APIs.
132. Prefer additive schema changes.
133. Use expand-contract migrations.
134. Do not make rollback impossible prematurely.
135. Separate application rollback from database rollback.
136. Validate data correctness.
137. Monitor business invariants.
138. Protect irreversible operations.
139. Use safe synthetic data.
140. Use controlled external integrations.
141. Do not accidentally charge customers during tests.
142. Do not duplicate external writes.
143. Protect sensitive customer data.
144. Validate authentication.
145. Validate authorization.
146. Validate TLS.
147. Validate WAF behavior.
148. Validate audit logging.
149. Validate security policies.
150. Keep canary security equivalent to production.
```

## 169. Rules 151–200

```text
151. Use GitOps for declarative rollout configuration.
152. Keep promotion auditable.
153. Record commit.
154. Record image digest.
155. Record traffic weights.
156. Record analysis results.
157. Record promotion.
158. Record rollback.
159. Record operator decisions.
160. Protect rollout-controller permissions.
161. Use least privilege.
162. Test controller failure.
163. Test traffic-controller failure.
164. Test observability failure.
165. Test rollback.
166. Test rollback during load.
167. Test rollback after schema expansion.
168. Test rollback with active connections.
169. Test rollback with active queues.
170. Test rollback with feature flags.
171. Keep previous artifact available.
172. Keep previous configuration available.
173. Keep previous routing available.
174. Keep recovery evidence.
175. Use feature flags for risky functionality.
176. Separate deployment from activation.
177. Activate risky features gradually.
178. Monitor feature-specific metrics.
179. Use regional waves.
180. Use cluster waves.
181. Use cell waves.
182. Stop the fleet on correlated failure.
183. Avoid global simultaneous rollout.
184. Protect failure domains.
185. Consider multi-region dependencies.
186. Monitor replication lag.
187. Monitor cross-region behavior.
188. Validate regional business metrics.
189. Validate regional latency.
190. Validate regional traffic.
191. Keep rollback capability across regions.
192. Define regional rollback order.
193. Define global rollback policy.
194. Document ownership.
195. Document escalation.
196. Define manual approval points.
197. Keep runbooks current.
198. Test runbooks.
199. Review failed canaries.
200. Feed failures back into automation.
```

## 170. Rules 201–250

```text
201. Canary is not a substitute for testing.
202. Canary complements unit tests.
203. Canary complements integration tests.
204. Canary complements load tests.
205. Canary complements security testing.
206. Canary complements disaster recovery.
207. Canary is a production risk-control mechanism.
208. Keep staging representative.
209. Do not use canary to discover preventable defects.
210. Validate obvious defects before production.
211. Use progressive exposure.
212. Do not promote because the timer expired.
213. Promote because evidence supports promotion.
214. Do not rollback because a metric is meaningless.
215. Validate metric semantics.
216. Validate aggregation.
217. Validate sampling.
218. Validate labels.
219. Validate baseline selection.
220. Validate cohort composition.
221. Account for traffic seasonality.
222. Account for peak traffic.
223. Account for low-traffic periods.
224. Account for rare workflows.
225. Account for high-value transactions.
226. Account for mobile/legacy clients.
227. Account for asynchronous workloads.
228. Account for long-lived connections.
229. Account for data migrations.
230. Account for third-party services.
231. Account for shared infrastructure.
232. Keep rollout state recoverable.
233. Keep rollout decisions auditable.
234. Keep customer impact minimal.
235. Prefer fast rollback over prolonged diagnosis during severe impact.
236. Preserve evidence after rollback.
237. Investigate before retrying.
238. Fix before re-promoting.
239. Re-test after fixes.
240. Keep rollback windows appropriate.
241. Clean up old versions only after confidence.
242. Balance rollout speed against risk.
243. Balance capacity against cost.
244. Balance automation against operational control.
245. Balance statistical confidence against release velocity.
246. Treat business metrics as first-class release signals.
247. Treat dependency health as first-class release signals.
248. Treat rollback as part of the design, not an emergency afterthought.
249. A successful canary is evidence that a release is safer to promote, not
     proof that every possible production failure has been eliminated.
250. The ultimate goal of canary deployment is controlled exposure, early
     detection, measurable evidence, rapid rollback and minimal blast radius.
```

# END OF 15-Canary-Deployment.md
