# 19-DevOps-System-Design
# 14-Blue-Green-Deployment

## 1. Purpose

This file is a deep production-oriented guide to Blue-Green Deployment for
DevOps, AWS, Kubernetes, EKS, microservices, databases, networking,
observability, GitOps, security and release engineering.

Blue-Green Deployment is a release strategy in which two separately
deployable environments represent two versions of an application:

```text
BLUE  = current production version
GREEN = candidate/new production version
```

Traffic is deliberately shifted between them.

The important production questions are:

```text
How are blue and green isolated?
How is green validated before traffic?
How is traffic switched?
How quickly can we roll back?
Can both versions coexist?
Is the database backward compatible?
How are sessions handled?
How are queues handled?
How are background jobs handled?
How are secrets/configuration synchronized?
How is observability compared?
What happens if green fails after receiving traffic?
How is failback performed?
```

Blue-green is not merely:

```text
create another Deployment
```

It is an end-to-end release architecture.

---

# PART I — FOUNDATIONS

## 2. Definition

A simplified architecture:

```text
                 Traffic
                    |
             Traffic Switch
              /           \
           BLUE           GREEN
         Version N       Version N+1
```

Only one environment normally receives primary production traffic at a
time, although controlled traffic splitting can be introduced for validation.

---

## 3. Why Blue-Green?

Primary goals:

```text
fast deployment
low-risk cutover
fast rollback
production-like validation
```

---

## 4. Main Advantage

Rollback can be:

```text
GREEN -> BLUE
```

instead of rebuilding the previous release.

This can dramatically reduce recovery time.

---

## 5. Main Cost

You may temporarily need:

```text
BLUE capacity
+
GREEN capacity
```

Therefore blue-green can require approximately double application capacity
during the transition.

---

# PART II — BLUE-GREEN VS OTHER STRATEGIES

## 6. Rolling Deployment

```text
old
old
old

new
old
old

new
new
old

new
new
new
```

Advantages:

```text
lower infrastructure overhead
```

Risk:

```text
old/new versions coexist
```

---

## 7. Blue-Green

```text
BLUE  -> 100%
GREEN -> 0%

switch

BLUE  -> 0%
GREEN -> 100%
```

Rollback:

```text
GREEN -> 0%
BLUE  -> 100%
```

---

## 8. Canary

Canary normally uses partial traffic:

```text
BLUE  -> 95%
GREEN -> 5%
```

Blue-green traditionally performs a complete cutover, although hybrid
patterns are possible.

---

## 9. Recreate

```text
stop old
 |
deploy new
```

This can introduce downtime and has slower rollback.

---

# PART III — REQUIREMENTS

## 10. Define Release Objective

Before implementing blue-green define:

```text
availability target
deployment frequency
rollback target
traffic cutover method
validation criteria
database strategy
```

---

## 11. Rollback Requirement

Example:

```text
rollback < 2 minutes
```

Then the rollback mechanism must already be prepared.

---

# PART IV — ARCHITECTURE

## 12. Basic AWS Architecture

```text
                    Users
                      |
                    Route 53
                      |
                     ALB
                  /       \
             BLUE TG     GREEN TG
                |            |
             EKS BLUE      EKS GREEN
                |            |
             Pods N       Pods N+1
```

Traffic can be moved by changing the ALB target-group routing strategy.

---

## 13. Separate Target Groups

A common ALB pattern:

```text
Listener
 |
+-- Blue Target Group
|
+-- Green Target Group
```

This permits controlled switching without rebuilding the load balancer.

---

# PART V — KUBERNETES

## 14. Two Deployments

Example conceptual model:

```text
app-blue
app-green
```

Each can have independent:

```text
Deployment
Service
ConfigMap
Secret references
HPA
PDB
```

---

## 15. Kubernetes Service Switching

Pattern:

```text
Service
 |
selector: version=blue
 |
Blue pods
```

Switch:

```text
selector: version=green
```

The Service remains stable while the backend changes.

---

## 16. Important Caveat

Kubernetes Service selector switching can be fast, but traffic propagation,
connection behavior and application readiness still need validation.

---

# PART VI — LABELING

## 17. Labels

Example:

```yaml
app: checkout
version: blue
```

Green:

```yaml
app: checkout
version: green
```

Avoid ambiguous selectors.

---

# PART VII — NAMESPACES

## 18. Namespace Isolation

Possible:

```text
checkout-blue
checkout-green
```

or:

```text
production
 |
blue resources
green resources
```

Choose according to operational and policy requirements.

---

# PART VIII — EKS DESIGN

## 19. Same Cluster

Option:

```text
EKS
 |
+-- blue
+-- green
```

Advantages:

```text
lower infrastructure cost
simpler shared networking
```

Risks:

```text
shared cluster failure domain
resource contention
```

---

## 20. Separate Clusters

```text
EKS Blue
EKS Green
```

Advantages:

```text
stronger isolation
```

Costs:

```text
higher cost
more operations
```

---

# PART IX — NODE CAPACITY

## 21. Double Capacity

If blue uses:

```text
10 nodes
```

green may require:

```text
10 additional nodes
```

Plan for:

```text
20 nodes
+
failure headroom
```

---

## 22. Scheduling

Do not allow green to starve blue.

Use:

```text
requests
quotas
priority
node pools
taints/tolerations
```

where appropriate.

---

# PART X — PREPARATION

## 23. Blue-Green Release Flow

```text
Build
 |
Test
 |
Publish
 |
Deploy GREEN
 |
Wait
 |
Health checks
 |
Smoke tests
 |
Production validation
 |
Switch traffic
 |
Observe
 |
Retain BLUE
 |
Retire BLUE
```

---

# PART XI — BUILD

## 24. Immutable Artifact

Build once:

```text
source
 |
CI
 |
image@digest
```

Deploy the same artifact to green.

Do not rebuild separately for production.

---

# PART XII — IMAGE TAGS

## 25. Prefer Digest

Use:

```text
repository/image@sha256:digest
```

rather than:

```text
latest
```

for production release identity.

---

# PART XIII — CONFIGURATION

## 26. Configuration Parity

Blue and green should have intentionally comparable:

```text
environment variables
config
secrets
feature flags
network policies
resource limits
```

Differences should be deliberate and documented.

---

# PART XIV — SECRETS

## 27. Secret Compatibility

Green may require:

```text
new secret
new API key
new certificate
```

Provision these before cutover.

---

# PART XV — DATABASE

## 28. Database Is the Hard Problem

Application rollback is easy only when the database remains compatible.

Bad migration:

```text
BLUE -> schema V1
GREEN -> schema V2
```

If V2 removes fields required by V1:

```text
rollback -> failure
```

---

# PART XVI — EXPAND-CONTRACT

## 29. Safe Schema Migration

Use:

```text
Expand
 |
deploy compatible code
 |
migrate data
 |
switch
 |
Contract later
```

---

## 30. Expand

Add new schema elements without breaking blue.

Example:

```text
add column
```

Blue continues working.

---

## 31. Contract

Remove obsolete elements only after old code is no longer required.

---

# PART XVII — DATABASE ROLLBACK

## 32. Application Rollback vs Data Rollback

These are different:

```text
application rollback
!=
database rollback
```

Database rollback may require:

```text
migration reversal
PITR
reconciliation
```

---

# PART XVIII — TRANSACTIONS

## 33. Long Transactions

A deployment cutover can interact with:

```text
long-running transactions
locks
connection pools
```

Plan graceful behavior.

---

# PART XIX — CONNECTION POOLS

## 34. Green Connection Storm

When green starts:

```text
100 pods
x
50 DB connections
=
5,000 connections
```

This can overwhelm the database before green receives meaningful traffic.

Use controlled initialization.

---

# PART XX — CACHE

## 35. Cache Compatibility

Blue and green may share cache.

Problems:

```text
different serialization
different key format
different schema
```

Use versioned cache keys where necessary.

---

# PART XXI — SESSIONS

## 36. Session Strategy

If sessions are local:

```text
user -> blue
```

then after switching:

```text
user -> green
```

the session may disappear.

Use:

```text
shared session store
stateless tokens
```

where appropriate.

---

# PART XXII — QUEUES

## 37. Background Jobs

Blue-green becomes more complicated when both versions can consume the same
queue.

Example:

```text
Blue worker
Green worker
 |
same queue
```

They must be compatible with the same message schema.

---

# PART XXIII — WORKER STRATEGY

## 38. Worker Cutover

Options:

```text
stop blue workers
 |
start green workers
```

or:

```text
run both with compatible consumers
```

Choose according to processing semantics.

---

# PART XXIV — CRON JOBS

## 39. Duplicate Cron Risk

If both environments run:

```text
daily billing job
```

you can execute it twice.

Use:

```text
single scheduler
distributed lock
leader election
job ownership
```

---

# PART XXV — EVENT PROCESSING

## 40. Idempotency

During deployment:

```text
event
 |
blue
green
```

Duplicate processing may occur.

Use idempotent consumers where required.

---

# PART XXVI — TRAFFIC SWITCH

## 41. Switching Mechanisms

Possible:

```text
ALB listener
target group
Route 53
service selector
ingress
service mesh
API gateway
global traffic manager
```

---

# PART XXVII — ALB SWITCH

## 42. ALB

Concept:

```text
Listener
 |
target group
 |
BLUE
```

Change:

```text
Listener
 |
target group
 |
GREEN
```

---

# PART XXVIII — ROUTE 53 SWITCH

## 43. DNS

```text
app.example
 |
Route 53
 |
BLUE
```

Switch:

```text
app.example
 |
Route 53
 |
GREEN
```

DNS caching means the transition is not necessarily instantaneous.

---

# PART XXIX — SERVICE SWITCH

## 44. Kubernetes Service

```text
Service
 |
version=blue
```

then:

```text
version=green
```

This avoids changing clients.

---

# PART XXX — GLOBAL TRAFFIC

## 45. Multi-Region Blue-Green

```text
Global
 |
+-- Region A
|    +-- Blue
|    +-- Green
|
+-- Region B
     +-- Blue
     +-- Green
```

Complexity grows quickly.

Use only when the release strategy justifies it.

---

# PART XXXI — VALIDATION

## 46. Green Validation

Validate:

```text
pod readiness
application health
database connectivity
cache
queue
external APIs
TLS
DNS
```

---

## 47. Smoke Tests

Examples:

```text
login
GET product
create cart
checkout test
```

---

# PART XXXII — SYNTHETIC TESTING

## 48. Production-Like Tests

A green environment should be validated with representative traffic patterns
without accidentally creating real business side effects.

Use:

```text
test accounts
synthetic transactions
sandbox integrations
```

where appropriate.

---

# PART XXXIII — SHADOW TRAFFIC

## 49. Shadow Traffic

Copy production requests to green without using green responses for users.

Useful for:

```text
compatibility
performance
error comparison
```

Protect against:

```text
side effects
duplicate writes
sensitive data exposure
```

---

# PART XXXIV — CANARY + BLUE-GREEN

## 50. Hybrid

Start:

```text
Blue -> 100%
Green -> 0%
```

Then:

```text
Blue -> 95%
Green -> 5%
```

If healthy:

```text
50/50
```

Then:

```text
Green -> 100%
```

This combines isolation with progressive validation.

---

# PART XXXV — OBSERVABILITY

## 51. Compare Blue and Green

Monitor separately:

```text
RPS
error rate
P50
P95
P99
CPU
memory
DB latency
external API errors
```

---

# PART XXXVI — GOLDEN SIGNALS

## 52. Green Health

Watch:

```text
latency
traffic
errors
saturation
```

Compare with blue.

---

# PART XXXVII — BUSINESS METRICS

## 53. Technical Health Is Not Enough

Monitor:

```text
checkout success
payment success
order creation
login success
conversion
```

A service can have:

```text
HTTP 200
```

while business behavior is broken.

---

# PART XXXVIII — AUTOMATED GATES

## 54. Promotion Gate

Example:

```text
error rate < threshold
AND
P99 < threshold
AND
business success > threshold
```

Only then permit cutover.

---

# PART XXXIX — ROLLBACK

## 55. Fast Rollback

```text
GREEN
 |
failure
 |
switch
 |
BLUE
```

Rollback should not require:

```text
new build
new deployment
manual reconstruction
```

---

# PART XL — ROLLBACK TRIGGERS

## 56. Automatic Triggers

Possible:

```text
5xx spike
P99 latency spike
business transaction failure
health-check failure
dependency failure
```

Use conservative thresholds to avoid oscillation.

---

# PART XLI — ROLLBACK OSCILLATION

## 57. Flapping

Bad:

```text
blue -> green
green -> blue
blue -> green
```

Use:

```text
stabilization
cooldown
manual approval
clear thresholds
```

---

# PART XLII — CONNECTION DRAIN

## 58. Blue Drain

Before retiring blue:

```text
stop new traffic
 |
connection draining
 |
active requests finish
 |
terminate
```

Long-lived connections require special handling.

---

# PART XLIII — WEBSOCKETS

## 59. Long-Lived Connections

Switching traffic does not instantly move existing WebSocket connections.

Plan:

```text
drain
reconnect
session continuity
```

---

# PART XLIV — GRPC

## 60. Streaming

Long-lived gRPC streams can remain connected to blue after traffic moves.

Define graceful draining.

---

# PART XLV — FEATURE FLAGS

## 61. Feature Flags

Separate:

```text
code deployment
```

from:

```text
feature activation
```

This can reduce release risk.

---

# PART XLVI — RELEASE CONTROL

## 62. Recommended Sequence

```text
Deploy code
 |
Validate infrastructure
 |
Validate green
 |
Enable feature
 |
Observe
 |
Increase exposure
```

---

# PART XLVII — SECURITY

## 63. Security Parity

Green must have equivalent:

```text
WAF
network policy
IAM
security groups
TLS
logging
audit
```

---

# PART XLVIII — RBAC

## 64. Kubernetes RBAC

Ensure green has required:

```text
ServiceAccount
Role
RoleBinding
```

without accidentally granting additional permissions.

---

# PART XLIX — NETWORK POLICY

## 65. NetworkPolicy

Validate green connectivity to:

```text
database
cache
queue
external APIs
```

while preserving intended isolation.

---

# PART L — CERTIFICATES

## 66. TLS

Green must support:

```text
certificate
SNI
hostnames
TLS versions
```

before cutover.

---

# PART LI — CONFIG DRIFT

## 67. Blue-Green Drift

Bad:

```text
Blue -> correct
Green -> outdated config
```

Use declarative configuration and automated parity checks.

---

# PART LII — GITOPS

## 68. GitOps Flow

```text
Git
 |
Argo CD
 |
Green manifests
 |
EKS
 |
Validation
 |
Traffic switch
```

---

# PART LIII — PROMOTION

## 69. GitOps Promotion

Possible repository model:

```text
dev
 |
staging
 |
production-green
 |
production-active
```

Promotion should be traceable.

---

# PART LIV — ARGO ROLLBACK

## 70. Git Revert

A GitOps rollback can restore the previous desired state.

But:

```text
Git rollback
!=
database rollback
```

Always evaluate stateful dependencies.

---

# PART LV — CI/CD PIPELINE

## 71. Pipeline

```text
Commit
 |
Build
 |
Unit tests
 |
Security
 |
Image
 |
Deploy Green
 |
Smoke
 |
Approval
 |
Switch
 |
Observe
```

---

# PART LVI — APPROVALS

## 72. Production Approval

Use approval gates when business risk requires them.

Automation should handle predictable checks.

---

# PART LVII — CAPACITY

## 73. Capacity Planning

Blue-green requires capacity for:

```text
blue
+
green
+
surge
+
failure
```

---

# PART LVIII — COST

## 74. Cost

Temporary double capacity can affect:

```text
EC2
EKS
database connections
load balancer
observability
network
```

---

# PART LIX — DATABASE CAPACITY

## 75. Shared Database

If both versions share the database:

```text
Blue
 |
+-- DB
 |
Green
```

database load may increase during validation.

---

# PART LX — SEPARATE DATABASE

## 76. Separate Database

```text
Blue -> DB Blue
Green -> DB Green
```

This provides isolation but creates:

```text
data synchronization
migration
cutover
```

complexity.

---

# PART LXI — DATA MIGRATION

## 77. Migration Strategies

Possible:

```text
dual write
CDC
replication
offline migration
expand-contract
```

Choose based on consistency and scale.

---

# PART LXII — DUAL WRITE

## 78. Dual Write Risk

```text
write A -> DB1
write B -> DB2
```

One write can succeed while the other fails.

Use carefully.

---

# PART LXIII — CDC

## 79. Change Data Capture

```text
Primary DB
 |
CDC
 |
Target DB
```

Useful for migration/cutover scenarios but requires operational monitoring.

---

# PART LXIV — STATEFUL SERVICES

## 80. Stateful Blue-Green

Harder for:

```text
databases
queues
file stores
distributed locks
```

Prefer blue-green at the application layer where state cannot be duplicated
safely.

---

# PART LXV — CACHE WARMING

## 81. Warm Green

Before traffic:

```text
populate cache
 |
validate hit ratio
 |
switch
```

This avoids an immediate database load spike.

---

# PART LXVI — WARM-UP

## 82. Application Warm-Up

Green may need:

```text
JVM JIT
connection pool
cache
model loading
DNS
TLS
```

Do not cut traffic immediately after pod readiness.

---

# PART LXVII — READINESS

## 83. Readiness Is Necessary But Not Sufficient

A pod can be:

```text
Ready
```

while:

```text
database latency
```

is unacceptable.

Use service-level validation.

---

# PART LXVIII — LOAD TESTING

## 84. Green Load Test

Test:

```text
normal load
peak load
expected concurrency
```

before switching when feasible.

---

# PART LXIX — PERFORMANCE REGRESSION

## 85. Compare

```text
Blue P99 = 180ms
Green P99 = 700ms
```

Even if both are technically healthy, green may be a regression.

---

# PART LXX — DEPLOYMENT SAFETY

## 86. Freeze

During final cutover:

```text
no unrelated infrastructure changes
```

Reduce variables during diagnosis.

---

# PART LXXI — INCIDENT DURING CUTOVER

## 87. Procedure

```text
stop rollout
 |
stabilize
 |
switch back if necessary
 |
investigate
```

Do not continue deployment while the system is unstable.

---

# PART LXXII — FAILBACK

## 88. Keep Blue

Do not immediately delete blue.

Keep it until:

```text
green stable
rollback window passed
data compatibility confirmed
```

---

# PART LXXIII — BLUE RETIREMENT

## 89. Cleanup

After validation:

```text
remove old pods
release capacity
archive deployment
```

Retain rollback metadata and artifacts according to policy.

---

# PART LXXIV — MULTI-SERVICE RELEASES

## 90. Coordinated Releases

If:

```text
Service A
Service B
```

must change together, blue-green may need compatibility sequencing.

Prefer:

```text
backward-compatible API
```

where possible.

---

# PART LXXV — API VERSIONING

## 91. Versioned API

```text
/v1
/v2
```

can allow blue and green to coexist safely.

---

# PART LXXVI — MESSAGE VERSIONING

## 92. Event Schema

Use backward-compatible event changes:

```text
old consumer
+
new consumer
```

must understand the transition period.

---

# PART LXXVII — DEPENDENCY COMPATIBILITY

## 93. Compatibility Matrix

For each release:

```text
Green
 |
DB
 |
Cache
 |
Queue
 |
External APIs
```

validate compatibility.

---

# PART LXXVIII — THIRD-PARTY SERVICES

## 94. External APIs

Green may use:

```text
new endpoint
new API version
new credentials
```

Validate before cutover.

---

# PART LXXIX — OBSERVABILITY RELEASE

## 95. Telemetry

Green must emit compatible:

```text
metrics
logs
traces
```

Do not introduce uncontrolled metric cardinality during deployment.

---

# PART LXXX — AUDIT

## 96. Release Evidence

Record:

```text
commit
image digest
deployment time
approver
validation results
traffic switch
rollback
```

---

# PART LXXXI — SECURITY SCANNING

## 97. Artifact Security

Before green:

```text
dependency scan
image scan
SBOM
signature/provenance
policy checks
```

where required.

---

# PART LXXXII — SUPPLY CHAIN

## 98. Immutable Release

Promote the same verified artifact:

```text
build once
 |
scan
 |
sign
 |
promote
 |
deploy
```

---

# PART LXXXIII — BLUE-GREEN WITH EKS INGRESS

## 99. Ingress

Concept:

```text
Ingress
 |
blue service
```

switch:

```text
Ingress
 |
green service
```

Use declarative GitOps management.

---

# PART LXXXIV — SERVICE MESH

## 100. Mesh Traffic

A service mesh can provide:

```text
traffic splitting
header routing
weight routing
```

This enables blue-green/canary hybrids.

---

# PART LXXXV — GLOBAL DEPLOYMENT

## 101. Regional Waves

For multi-region:

```text
Region A -> green
validate
Region B -> green
validate
Region C -> green
```

This reduces global blast radius.

---

# PART LXXXVI — FAILURE DOMAINS

## 102. Never Cut Over Everything Blindly

For large platforms:

```text
cell 1
 |
validate
 |
cell 2
 |
validate
```

Progressive deployment reduces risk.

---

# PART LXXXVII — ROLLBACK DATA

## 103. Rollback Readiness

Keep:

```text
blue artifact
blue configuration
blue manifests
blue routing
```

available.

---

# PART LXXXVIII — AUTOMATED ROLLBACK

## 104. Controller

Concept:

```text
deploy green
 |
observe
 |
SLO breach?
 |
yes -> blue
no  -> continue
```

Automation should have safeguards.

---

# PART LXXXIX — ROLLBACK SAFETY

## 105. Do Not Roll Back Blindly

Before rollback:

```text
is data compatible?
are writes still occurring?
is blue healthy?
```

---

# PART XC — PRODUCTION EXAMPLE

## 106. E-Commerce

```text
Users
 |
ALB
 |
+-----------+
|           |
BLUE       GREEN
v1.8       v1.9
 |           |
+-----+-----+
      |
    DB
```

Deploy green:

```text
v1.9
```

Validate:

```text
login
cart
checkout
```

Switch:

```text
ALB -> GREEN
```

Keep blue.

---

# PART XCI — FAILURE EXAMPLE

## 107. Green Database Regression

After cutover:

```text
Green
 |
DB latency
 |
P99 rises
```

Action:

```text
switch back
 |
Blue
```

Then investigate without increasing customer impact.

---

# PART XCII — PRODUCTION CHECKLIST

## 108. Before Deployment

```text
[ ] artifact immutable
[ ] security checks passed
[ ] database compatibility checked
[ ] config ready
[ ] secrets ready
[ ] capacity available
[ ] rollback artifact available
[ ] observability ready
```

## 109. Green Validation

```text
[ ] readiness
[ ] health
[ ] smoke
[ ] database
[ ] cache
[ ] queue
[ ] external APIs
[ ] performance
[ ] business transactions
```

## 110. Cutover

```text
[ ] approval
[ ] traffic switch
[ ] monitor
[ ] rollback criteria active
```

## 111. After Cutover

```text
[ ] business metrics healthy
[ ] no error regression
[ ] latency healthy
[ ] capacity healthy
[ ] blue retained
[ ] rollback window observed
```

---

# PART XCIII — SENIOR SYSTEM-DESIGN SCENARIOS

## 112. Design Blue-Green for 10,000 RPS

Requirements:

```text
10,000 RPS
99.99%
rollback < 2 minutes
```

Design:

```text
ALB
 |
blue/green target groups
 |
EKS
 |
HA database
```

Need:

```text
double application capacity
database compatibility
automated validation
fast target-group switch
```

---

## 113. Design Blue-Green for Stateful Database

Prefer:

```text
expand-contract
```

rather than destructive schema changes.

Application rollback must remain compatible with the expanded schema.

---

## 114. Design Blue-Green for WebSockets

Need:

```text
connection draining
reconnect strategy
session continuity
```

---

## 115. Design Blue-Green for Queue Workers

Need:

```text
message compatibility
idempotency
worker ownership
duplicate processing strategy
```

---

## 116. Design Multi-Region Blue-Green

Use:

```text
regional rollout
 |
validation
 |
traffic
 |
next region
```

Avoid changing every region simultaneously.

---

## 117. Design Blue-Green With GitOps

Flow:

```text
Git
 |
CI
 |
image
 |
production-green
 |
Argo CD
 |
validation
 |
promotion
 |
production-active
```

---

# PART XCIV — INTERVIEW FRAMEWORK

## 118. Senior Answer

Use:

```text
1. Define blue and green boundaries.
2. Define traffic switch.
3. Define capacity requirements.
4. Deploy immutable artifact.
5. Validate green.
6. Verify dependency compatibility.
7. Address database migration.
8. Address sessions and queues.
9. Define cutover.
10. Define rollback.
11. Keep old environment available.
12. Monitor technical and business SLOs.
13. Define cleanup.
14. Explain cost and trade-offs.
```

---

# PART XCV — PRODUCTION RUNBOOK

## 119. Deployment

```text
1. Freeze unrelated changes.
2. Verify artifact.
3. Verify configuration.
4. Verify capacity.
5. Deploy green.
6. Wait for readiness.
7. Run health checks.
8. Run smoke tests.
9. Run business validation.
10. Compare blue/green metrics.
11. Obtain approval if required.
12. Switch traffic.
13. Monitor.
14. Trigger rollback if criteria are breached.
15. Keep blue until rollback window expires.
16. Clean up after stability.
```

---

## 120. Rollback

```text
1. Confirm rollback trigger.
2. Stop further promotion.
3. Verify blue health.
4. Check data compatibility.
5. Switch traffic to blue.
6. Drain green.
7. Monitor blue.
8. Preserve green evidence.
9. Investigate root cause.
10. Fix and retest before another promotion.
```

---

# PART XCVI — 250 PRODUCTION GOLDEN RULES

## 121. Rules 1–50

```text
1. Define blue and green clearly.
2. Treat blue-green as an architecture, not a naming convention.
3. Define rollback requirements first.
4. Keep rollback faster than redeployment where required.
5. Build immutable artifacts.
6. Deploy the same artifact to green.
7. Prefer image digests.
8. Do not use mutable latest tags for release identity.
9. Validate green before cutover.
10. Keep blue healthy during green validation.
11. Plan temporary double capacity.
12. Include failure headroom.
13. Protect blue from green resource contention.
14. Use requests and limits correctly.
15. Use quotas where appropriate.
16. Use dedicated capacity when necessary.
17. Validate node capacity.
18. Validate IP capacity.
19. Validate load-balancer capacity.
20. Validate database capacity.
21. Validate cache capacity.
22. Validate queue capacity.
23. Validate observability capacity.
24. Validate network capacity.
25. Define traffic-switch mechanism.
26. Keep switching simple.
27. Automate traffic switching where safe.
28. Keep rollback path ready.
29. Do not delete blue immediately.
30. Define rollback window.
31. Use health checks.
32. Use readiness checks.
33. Use smoke tests.
34. Use synthetic tests.
35. Use business validation.
36. Compare blue and green metrics.
37. Compare P95.
38. Compare P99.
39. Compare error rate.
40. Compare saturation.
41. Compare business success.
42. Define promotion gates.
43. Define rollback gates.
44. Avoid ambiguous thresholds.
45. Avoid rollback flapping.
46. Use stabilization periods.
47. Use cooldowns where appropriate.
48. Keep evidence of every release.
49. Record artifact digest.
50. Record traffic-switch time.
```

## 122. Rules 51–100

```text
51. Record rollback events.
52. Record approval.
53. Keep deployment history.
54. Keep release manifests.
55. Manage configuration declaratively.
56. Detect configuration drift.
57. Synchronize required secrets.
58. Validate certificates.
59. Validate DNS.
60. Validate network policies.
61. Validate IAM.
62. Validate ServiceAccounts.
63. Validate RBAC.
64. Validate external dependencies.
65. Validate database connectivity.
66. Validate cache compatibility.
67. Validate queue compatibility.
68. Validate event schemas.
69. Validate API compatibility.
70. Use backward-compatible changes.
71. Prefer expand-contract database migrations.
72. Do not perform destructive migrations before rollback safety exists.
73. Separate application rollback from database rollback.
74. Understand schema compatibility.
75. Test migration rollback assumptions.
76. Avoid incompatible cache schemas.
77. Version cache keys when necessary.
78. Avoid local sessions where possible.
79. Use shared session state when required.
80. Plan WebSocket draining.
81. Plan gRPC stream draining.
82. Plan long-running requests.
83. Plan queue consumers.
84. Prevent duplicate workers.
85. Protect scheduled jobs.
86. Prevent duplicate cron execution.
87. Use idempotency.
88. Handle duplicate events.
89. Handle replay.
90. Plan background-job ownership.
91. Warm caches where necessary.
92. Warm application runtimes where necessary.
93. Do not equate pod Ready with production readiness.
94. Test realistic load.
95. Test peak load.
96. Test dependency behavior.
97. Test external API behavior.
98. Test failure during cutover.
99. Test rollback during peak load.
100. Test rollback after data changes.
```

## 123. Rules 101–150

```text
101. Use ALB target groups appropriately.
102. Use stable Kubernetes Services when appropriate.
103. Use Route 53 deliberately.
104. Understand DNS caching.
105. Understand connection persistence.
106. Understand load-balancer draining.
107. Use service mesh traffic controls when justified.
108. Use progressive traffic when risk requires it.
109. Combine blue-green with canary when useful.
110. Use shadow traffic carefully.
111. Prevent shadow traffic side effects.
112. Protect sensitive production data in tests.
113. Use synthetic accounts.
114. Use sandbox third-party integrations.
115. Do not send real payments during tests.
116. Validate authentication.
117. Validate authorization.
118. Validate TLS.
119. Validate WAF.
120. Validate logging.
121. Validate metrics.
122. Validate tracing.
123. Avoid telemetry cardinality explosions.
124. Validate alerting.
125. Validate dashboards.
126. Validate business KPIs.
127. Validate customer-critical workflows.
128. Freeze unrelated changes during cutover.
129. Keep incident command available.
130. Stop promotion when instability appears.
131. Do not continue a rollout during an incident.
132. Roll back when defined criteria are met.
133. Do not roll back blindly after destructive data changes.
134. Verify blue health before rollback.
135. Verify data compatibility before rollback.
136. Preserve green evidence after rollback.
137. Investigate root cause.
138. Retest before promotion.
139. Keep release artifacts available.
140. Keep previous configuration available.
141. Keep previous manifests available.
142. Keep previous routing configuration available.
143. Automate restoration of previous routing.
144. Protect deployment permissions.
145. Use least privilege.
146. Audit production promotion.
147. Separate build and production privileges.
148. Verify supply-chain security.
149. Scan artifacts before deployment.
150. Promote verified artifacts rather than rebuilding.
```

## 124. Rules 151–200

```text
151. Use SBOM where required.
152. Validate image provenance where required.
153. Keep production images recoverable.
154. Protect the registry.
155. Plan registry failure.
156. Plan secret-store failure.
157. Plan database failure during rollout.
158. Plan cache failure during rollout.
159. Plan queue failure during rollout.
160. Plan DNS failure during rollout.
161. Plan observability failure during rollout.
162. Plan node failure during rollout.
163. Plan AZ failure during rollout.
164. Ensure green survives expected failures.
165. Ensure blue survives green resource consumption.
166. Keep enough spare capacity for failover.
167. Understand EKS scheduling behavior.
168. Use topology distribution for critical replicas.
169. Protect critical workloads with PDB where appropriate.
170. Validate HPA behavior.
171. Validate node autoscaling.
172. Avoid autoscaler feedback loops.
173. Include startup time in rollout planning.
174. Include image-pull time.
175. Include cache-warm time.
176. Include database-connection initialization.
177. Include validation time.
178. Include DNS convergence where applicable.
179. Include connection-drain time.
180. Include human approval time.
181. Define total cutover time.
182. Define total rollback time.
183. Measure actual cutover time.
184. Measure actual rollback time.
185. Compare against SLO.
186. Keep release procedures documented.
187. Test runbooks.
188. Test automated gates.
189. Test rollback automation.
190. Test traffic switching.
191. Test database compatibility.
192. Test session continuity.
193. Test queue compatibility.
194. Test cron ownership.
195. Test long-lived connections.
196. Test business workflows.
197. Test multi-service compatibility.
198. Test third-party integrations.
199. Test after major platform changes.
200. Blue-green is credible only when its rollback path has been tested.
```

## 125. Rules 201–250

```text
201. Keep old versions available during rollback windows.
202. Do not optimize cost by deleting rollback capacity too early.
203. Remove blue only after defined confidence criteria.
204. Retain recovery artifacts according to policy.
205. Use feature flags for risky functionality.
206. Separate deployment from feature activation.
207. Gradually activate risky features.
208. Monitor feature-specific metrics.
209. Disable features without necessarily rolling back code when possible.
210. Maintain API compatibility during transition.
211. Maintain event compatibility during transition.
212. Maintain schema compatibility during transition.
213. Avoid synchronized breaking changes.
214. Use versioned APIs where appropriate.
215. Use versioned event schemas where appropriate.
216. Protect external API quotas.
217. Avoid green validation exhausting third-party limits.
218. Avoid green validation generating real business side effects.
219. Protect customer data during testing.
220. Protect production secrets.
221. Keep green security posture equal to blue.
222. Do not bypass security controls for green.
223. Validate network policies.
224. Validate egress controls.
225. Validate ingress controls.
226. Validate encryption.
227. Validate audit trails.
228. Validate compliance controls.
229. Validate observability before traffic.
230. Validate alerts before traffic.
231. Validate capacity before traffic.
232. Validate rollback before traffic.
233. Define explicit ownership for cutover.
234. Define explicit ownership for rollback.
235. Define approval boundaries.
236. Maintain an incident escalation path.
237. Stop rollout on unexplained SLO regression.
238. Prefer preserving customer availability over release completion.
239. Keep changes small enough to diagnose.
240. Use progressive regional rollout for large fleets.
241. Use cells or waves for large platforms.
242. Avoid global simultaneous cutovers unless justified.
243. Monitor each region separately.
244. Monitor each cluster separately.
245. Monitor each service separately.
246. Document every production exception.
247. Review failed deployments.
248. Feed lessons into release automation.
249. Treat blue-green as part of the complete reliability architecture.
250. The objective of blue-green deployment is not simply zero downtime; it is
     controlled release, measurable validation, rapid and safe rollback,
     compatibility across stateful dependencies, and minimal customer impact.
```

# END OF 14-Blue-Green-Deployment.md
