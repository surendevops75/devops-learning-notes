# 19-DevOps-System-Design
# 26-Blast-Radius-Reduction

## 1. Purpose

Blast-radius reduction is the practice of designing systems so that a single
mistake, failure, compromise, capacity event, or deployment affects the
smallest possible portion of production.

Core objective:

```text
Failure
   |
Contain
   |
Limit Scope
   |
Protect Healthy Systems
   |
Recover
   |
Learn
```

A mature DevOps architecture does not assume failures will never happen.
It assumes failures happen and makes them local, bounded, observable and
recoverable.

---

# PART I — FUNDAMENTALS

## 2. What Is Blast Radius?

Blast radius is the maximum portion of a system that can be affected by one
event.

Examples:

```text
bad pod
bad deployment
bad IAM policy
bad DNS record
bad Terraform apply
compromised credential
database failure
AZ outage
region outage
```

---

## 3. Blast Radius Formula

A useful conceptual model is:

```text
Blast Radius =
Affected Resources
x
Dependency Reach
x
Duration
x
Customer Criticality
```

This is not a literal mathematical production metric. It is a design model
for evaluating risk.

---

## 4. Small Blast Radius

Example:

```text
Canary deployment
 |
5% traffic
 |
failure
 |
rollback
```

Only a small customer population is affected.

---

## 5. Large Blast Radius

Example:

```text
Production
 |
global deployment
 |
bad configuration
 |
100% services affected
```

---

# PART II — BLAST-RADIUS OBJECTIVES

## 6. Primary Objectives

A resilient platform should minimize:

```text
number of affected customers
number of affected workloads
number of affected failure domains
duration of impact
amount of corrupted state
number of compromised resources
```

---

## 7. Four Control Dimensions

Blast radius can be reduced through:

```text
Isolation
Authorization
Progressive Change
Resource Limits
```

---

# PART III — ISOLATION

## 8. Infrastructure Isolation

Separate workloads by:

```text
node
AZ
cluster
account
region
```

when justified.

---

## 9. Logical Isolation

Use:

```text
namespace
service account
security group
network policy
IAM role
```

for logical boundaries.

---

## 10. Physical/Infrastructure Isolation

Logical isolation is not equivalent to physical isolation.

For high-risk workloads, stronger boundaries may be required.

---

# PART IV — KUBERNETES BLAST RADIUS

## 11. Pod Placement

Avoid:

```text
all replicas -> one node
```

Use:

```text
topology spread
anti-affinity
```

where appropriate.

---

## 12. Namespace Boundaries

Namespaces can provide:

```text
RBAC boundary
resource quota
network policy scope
organizational boundary
```

They are not a complete security or physical isolation boundary.

---

## 13. ResourceQuota

Prevent one team or workload from consuming unlimited namespace resources.

---

## 14. LimitRange

Provide defaults and constraints for container resources.

---

## 15. Priority Classes

Critical workloads can be prioritized during resource pressure.

---

## 16. PodDisruptionBudget

PDB can protect availability during voluntary disruptions.

It does not guarantee protection from all involuntary failures.

---

# PART V — NETWORK BLAST RADIUS

## 17. Network Segmentation

Use:

```text
VPC
subnets
security groups
network policies
firewalls
```

to control connectivity.

---

## 18. Default Deny

For sensitive Kubernetes workloads:

```text
default deny
 |
explicit allow
```

can reduce accidental lateral connectivity.

---

## 19. Security Group Segmentation

Prefer service-specific security groups over one universal production group.

---

## 20. Network Policy

Example conceptual model:

```text
frontend -> backend
backend  -> database
frontend -X-> database
```

---

# PART VI — IAM BLAST RADIUS

## 21. Least Privilege

A compromised identity should have the smallest possible permission set.

---

## 22. Shared Role Risk

Avoid:

```text
one role
 |
all environments
 |
all services
```

---

## 23. Environment Separation

Prefer:

```text
dev role
staging role
production role
```

with appropriate permissions.

---

## 24. Service-Specific Roles

Use workload identity so each service receives only required access.

---

## 25. SCP Guardrails

AWS Organizations service control policies can provide higher-level guardrails
across accounts.

---

# PART VII — CREDENTIAL BLAST RADIUS

## 26. Credential Scope

Reduce:

```text
permissions
lifetime
resource access
environment scope
```

---

## 27. Short-Lived Credentials

Prefer temporary credentials over long-lived static credentials when
supported.

---

## 28. Secret Rotation

Design rotation so one credential does not control the entire production
estate.

---

# PART VIII — AWS ACCOUNT BLAST RADIUS

## 29. Account Isolation

Accounts can isolate:

```text
permissions
billing
quotas
resources
security boundaries
```

---

## 30. Production Accounts

Large organizations often separate production workloads by business or risk
boundary.

---

## 31. Security Account

Central security functions should be protected from ordinary application
administration.

---

## 32. Logging Account

Centralized audit logs should be protected from application accounts.

---

# PART IX — REGION BLAST RADIUS

## 33. Regional Isolation

A region outage can affect every service in that region.

For critical workloads:

```text
Region A
 +
Region B
```

may reduce regional blast radius.

---

## 34. Multi-Region Trade-Off

Multi-region increases:

```text
cost
complexity
data replication challenges
operational burden
```

Use it when business requirements justify it.

---

# PART X — DEPLOYMENT BLAST RADIUS

## 35. Global Deployment

Highest-risk pattern:

```text
commit
 |
all production
 |
100%
```

---

## 36. Progressive Deployment

Safer:

```text
canary
 |
small percentage
 |
observe
 |
expand
```

---

## 37. Canary

Example:

```text
1%
5%
10%
25%
50%
100%
```

Stop immediately if health indicators degrade.

---

## 38. Blue-Green

```text
Blue = current
Green = candidate
```

Switch traffic only after validation.

---

# PART XI — CHANGE BLAST RADIUS

## 39. Change Scope

Limit each production change to:

```text
one service
one cluster
one region
one account
```

where possible.

---

## 40. Change Windows

For high-risk changes:

```text
low traffic
high staffing
rollback ready
```

---

## 41. Change Freeze

Freeze unrelated changes during severe incidents when appropriate.

---

# PART XII — CONFIGURATION BLAST RADIUS

## 42. Global Configuration

One bad configuration value can affect every service.

Avoid uncontrolled global configuration.

---

## 43. Configuration Validation

Validate:

```text
syntax
schema
allowed values
security
dependencies
```

before rollout.

---

## 44. Configuration Canary

Roll configuration to:

```text
one instance
 |
one zone
 |
one cluster
 |
global
```

where practical.

---

# PART XIII — DATABASE BLAST RADIUS

## 45. Shared Database

A single shared database can connect many services to one failure domain.

---

## 46. Database Isolation

Possible approaches:

```text
database
schema
role
cluster
```

depending on requirements.

---

## 47. Connection Pool Limits

Prevent one service from consuming all database connections.

---

## 48. Query Blast Radius

One inefficient query can consume significant database resources.

Controls:

```text
timeouts
indexes
query limits
connection pools
resource controls
```

---

# PART XIV — MIGRATION BLAST RADIUS

## 49. Database Migration

Risky migration:

```text
large table
 |
blocking operation
 |
all application traffic
```

---

## 50. Safer Migration

Use:

```text
expand
 |
migrate
 |
validate
 |
contract
```

---

## 51. Backward Compatibility

Application and schema versions should remain compatible during staged
deployment.

---

# PART XV — CACHE BLAST RADIUS

## 52. Cache Dependency

If an application requires cache availability for every request, cache failure
can become a full outage.

---

## 53. Cache Degradation

Where possible:

```text
cache unavailable
 |
slower database-backed path
```

rather than:

```text
cache unavailable
 |
complete outage
```

---

# PART XVI — QUEUE BLAST RADIUS

## 54. Queue Isolation

Separate critical and noncritical workloads where appropriate.

---

## 55. Consumer Failure

If consumers fail:

```text
queue absorbs backlog
```

instead of immediately overwhelming downstream services.

---

## 56. DLQ

Dead-letter queues isolate repeatedly failing messages.

---

# PART XVII — RETRY BLAST RADIUS

## 57. Retry Amplification

Bad retry design:

```text
1 request
 |
5 retries
 |
dependency
```

At scale this can multiply load.

---

## 58. Retry Controls

Use:

```text
bounded attempts
exponential backoff
jitter
timeouts
```

---

# PART XVIII — TIMEOUT BLAST RADIUS

## 59. Timeout

Without timeouts:

```text
slow dependency
 |
threads blocked
 |
connection pools exhausted
 |
service outage
```

---

# PART XIX — CIRCUIT BREAKER

## 60. Circuit Breaker

```text
normal
 |
failure threshold
 |
open
 |
cooldown
 |
half-open
```

Protects the calling service from an unhealthy dependency.

---

# PART XX — BULKHEAD

## 61. Bulkhead

Separate resources:

```text
checkout
search
reports
```

so a failure in one workload does not exhaust the resources of another.

---

# PART XXI — LOAD SHEDDING

## 62. Load Shedding

During overload:

```text
critical traffic -> allowed
optional traffic -> rejected/degraded
```

---

# PART XXII — RATE LIMITING

## 63. Rate Limits

Apply limits at:

```text
edge
API gateway
service
database
queue consumer
```

where appropriate.

---

# PART XXIII — TRAFFIC MANAGEMENT

## 64. Traffic Shifting

Shift traffic:

```text
100% A
 |
90/10
 |
50/50
 |
0/100
```

with health validation.

---

# PART XXIV — DNS BLAST RADIUS

## 65. DNS

A global DNS mistake can affect all customers.

Use:

```text
change review
health checks
staged routing
```

---

## 66. TTL

TTL influences how quickly routing changes propagate through caches.

---

# PART XXV — CERTIFICATE BLAST RADIUS

## 67. Shared Certificate

One expired shared certificate can affect multiple services.

Prefer service-appropriate certificate boundaries.

---

# PART XXVI — SECRETS BLAST RADIUS

## 68. Shared Secret

Avoid one credential being embedded into many unrelated services.

---

# PART XXVII — STORAGE BLAST RADIUS

## 69. Shared Storage

Shared storage can turn one failure into many workload failures.

---

## 70. Storage Quotas

Use quotas where possible to prevent runaway workloads from consuming all
storage.

---

# PART XXVIII — LOGGING BLAST RADIUS

## 71. Logging

A logging failure should not normally stop the application.

Use:

```text
buffering
asynchronous shipping
bounded queues
```

where appropriate.

---

# PART XXIX — OBSERVABILITY BLAST RADIUS

## 72. Telemetry

A noisy service should not overwhelm the entire observability platform.

Control:

```text
cardinality
sampling
retention
ingestion
```

---

# PART XXX — PROMETHEUS

## 73. Metrics

High-cardinality labels can create excessive memory and storage consumption.

Avoid unbounded labels such as arbitrary request IDs.

---

# PART XXXI — LOGGING

## 74. Log Volume

A runaway application can generate enormous log volume.

Controls:

```text
sampling
rate limits
retention
log levels
```

---

# PART XXXII — TRACING

## 75. Trace Sampling

Use appropriate sampling to prevent trace volume from overwhelming storage.

---

# PART XXXIII — CI/CD BLAST RADIUS

## 76. Central Pipeline

A single broken pipeline can block every team.

Separate:

```text
build
test
artifact
deployment
```

and maintain operational fallbacks.

---

# PART XXXIV — BUILD BLAST RADIUS

## 77. Shared Runner

One overloaded runner pool can delay all builds.

Use:

```text
runner pools
quotas
priority
autoscaling
```

---

# PART XXXV — ARTIFACT BLAST RADIUS

## 78. Artifact Registry

Protect artifacts from:

```text
accidental deletion
malicious modification
retention errors
```

---

# PART XXXVI — GITOPS BLAST RADIUS

## 79. GitOps

A bad manifest merged into the main branch can propagate widely.

Controls:

```text
PR review
policy checks
environment promotion
progressive sync
```

---

# PART XXXVII — TERRAFORM BLAST RADIUS

## 80. Infrastructure Change

Avoid applying unreviewed destructive plans directly to production.

---

## 81. Terraform Controls

Use:

```text
plan
review
policy
approval
state protection
staged apply
```

---

# PART XXXVIII — IAM CHANGE BLAST RADIUS

## 82. Policy Change

A global IAM policy change can break many workloads.

Test policy changes against representative workloads.

---

# PART XXXIX — NETWORK CHANGE BLAST RADIUS

## 83. Route Change

A bad route can disconnect large portions of production.

Use staged changes and explicit rollback.

---

# PART XL — FIREWALL CHANGE

## 84. Security Rules

Avoid modifying broad production firewall rules when a narrower rule is
sufficient.

---

# PART XLI — RESOURCE EXHAUSTION

## 85. Resource Blast Radius

Shared resource exhaustion includes:

```text
CPU
memory
disk
network
connections
IPs
```

---

## 86. Quotas

Quotas contain runaway consumption.

---

# PART XLII — IP EXHAUSTION

## 87. EKS IP Capacity

Pod IP exhaustion can prevent new pods from starting.

Plan:

```text
subnet capacity
secondary IP capacity
CNI behavior
```

---

# PART XLIII — NODE CAPACITY

## 88. Node Pools

Separate capacity for:

```text
critical
general
batch
spot
```

where appropriate.

---

# PART XLIV — SPOT BLAST RADIUS

## 89. Spot

Do not put all critical capacity on Spot.

Use appropriate diversification and interruption handling.

---

# PART XLV — AUTOSCALING

## 90. Autoscaler Failure

Autoscaling itself can create risk if poorly configured.

Controls:

```text
min
max
cooldown
metrics
```

---

# PART XLVI — SCALING STORM

## 91. Scaling Storm

Bad autoscaling can rapidly create:

```text
too many pods
too many connections
too much cost
```

---

# PART XLVII — CONNECTION BLAST RADIUS

## 92. Pooling

Every service should have appropriate connection limits.

---

# PART XLVIII — THREAD EXHAUSTION

## 93. Threads

One slow dependency can consume all application threads.

Use:

```text
timeouts
bulkheads
bounded pools
```

---

# PART XLIX — MEMORY LEAK

## 94. Memory

Contain memory growth through:

```text
limits
restart policies
autoscaling
profiling
```

---

# PART L — CPU

## 95. CPU

Prevent one workload from consuming all node CPU through appropriate resource
requests, limits and workload isolation.

---

# PART LI — DISK

## 96. Disk

Use:

```text
quotas
retention
monitoring
automatic cleanup
```

---

# PART LII — SECURITY BLAST RADIUS

## 97. Compromised Workload

If one pod is compromised:

```text
network policy
RBAC
IAM
secrets isolation
```

should limit lateral movement.

---

# PART LIII — LATERAL MOVEMENT

## 98. Reduce

Avoid unrestricted:

```text
pod -> pod
pod -> metadata
pod -> database
pod -> management APIs
```

connectivity.

---

# PART LIV — METADATA ACCESS

## 99. Cloud Credentials

Workloads should receive only the cloud permissions they need.

---

# PART LV — SUPPLY CHAIN

## 100. Container Image

A compromised image can spread across every deployment using that image.

Controls:

```text
image scanning
signing
trusted registries
promotion
SBOM
```

---

# PART LVI — IMAGE PROMOTION

## 101. Environment Promotion

Prefer:

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

rather than rebuilding different artifacts for each environment.

---

# PART LVII — PACKAGE BLAST RADIUS

## 102. Dependency

A compromised dependency can enter many services.

Use:

```text
lock files
dependency scanning
approved repositories
```

---

# PART LVIII — SECRET SCANNING

## 103. Git

Prevent secrets from reaching source control.

---

# PART LIX — BRANCH PROTECTION

## 104. Protected Branches

Require appropriate:

```text
review
checks
status
```

before production-affecting changes.

---

# PART LX — FEATURE BLAST RADIUS

## 105. Feature Flags

A feature flag can reduce the blast radius of a risky feature.

---

# PART LXI — TENANT BLAST RADIUS

## 106. Multi-Tenant

A single tenant should not be able to exhaust shared resources.

Use:

```text
quotas
rate limits
isolation
priority
```

---

# PART LXII — CUSTOMER BLAST RADIUS

## 107. Customer Segmentation

Where appropriate, isolate:

```text
enterprise
standard
internal
```

traffic paths or resources.

---

# PART LXIII — DATA BLAST RADIUS

## 108. Data Access

Limit data access by:

```text
tenant
service
role
environment
```

---

# PART LXIV — BACKUP BLAST RADIUS

## 109. Backup Deletion

Protect backups from the same credentials that operate production.

---

## 110. Immutable Backup

Use immutable or protected backups where business requirements justify them.

---

# PART LXV — DR BLAST RADIUS

## 111. DR

DR should not depend entirely on the failed environment.

---

# PART LXVI — MULTI-ACCOUNT DR

## 112. Cross-Account Recovery

A separate recovery account can reduce the risk of a compromised primary
account destroying both production and recovery assets.

---

# PART LXVII — MULTI-REGION DR

## 113. Regional Recovery

Use:

```text
replication
backup
IaC
DNS
secrets
observability
```

in the recovery design.

---

# PART LXVIII — INCIDENT BLAST RADIUS

## 114. Incident Command

During an incident:

```text
identify
 |
contain
 |
protect healthy systems
 |
recover
```

---

# PART LXIX — CHANGE CORRELATION

## 115. Recent Changes

Always correlate:

```text
deployment
configuration
IAM
network
database
```

with incident start time.

---

# PART LXX — AUTOMATION BLAST RADIUS

## 116. Automation

Automation can reduce operational toil but can also multiply mistakes.

---

## 117. Guarded Automation

Require:

```text
scope
preconditions
limits
validation
rollback
```

---

# PART LXXI — AUTO-REMEDIATION

## 118. Safe Example

```text
pod unhealthy
 |
restart pod
 |
health check
```

---

## 119. Dangerous Example

```text
any alert
 |
restart entire production
```

Avoid broad destructive remediation.

---

# PART LXXII — RUNBOOK AUTOMATION

## 120. Runbook

Automate only steps that are well understood and reversible.

---

# PART LXXIII — RATE-LIMITED AUTOMATION

## 121. Limits

Automation should have:

```text
maximum actions
maximum resources
time window
abort condition
```

---

# PART LXXIV — HUMAN APPROVAL

## 122. High-Risk Actions

Require human approval for destructive or high-blast-radius changes when
appropriate.

---

# PART LXXV — CHANGE APPROVAL

## 123. Risk-Based Approval

Low-risk:

```text
automated
```

High-risk:

```text
review
approval
staged rollout
```

---

# PART LXXVI — FAILURE DOMAIN + BLAST RADIUS

## 124. Relationship

Failure-domain design answers:

```text
Where can failure spread?
```

Blast-radius reduction answers:

```text
How do we stop it?
```

---

# PART LXXVII — DEFENSE IN DEPTH

## 125. Multiple Controls

Example:

```text
IAM
+
network policy
+
security group
+
namespace
+
account boundary
```

No single control should be assumed perfect.

---

# PART LXXVIII — ZERO TRUST

## 126. Principle

Do not assume:

```text
inside network = trusted
```

Authorize based on identity and required access.

---

# PART LXXIX — PROGRESSIVE DELIVERY

## 127. End-to-End

```text
code
 |
test
 |
scan
 |
artifact
 |
canary
 |
observe
 |
expand
```

---

# PART LXXX — OBSERVABILITY GATES

## 128. Promotion

Use:

```text
error rate
latency
availability
business metrics
```

as promotion gates.

---

# PART LXXXI — AUTOMATED ROLLBACK

## 129. Rollback

If measurable health criteria fail:

```text
stop rollout
 |
rollback
 |
validate
```

---

# PART LXXXII — BUSINESS METRICS

## 130. Technical Metrics

Do not rely only on:

```text
CPU
memory
```

Include:

```text
checkout success
payment success
orders
login success
```

---

# PART LXXXIII — CUSTOMER SEGMENTS

## 131. Blast Radius

Monitor impact by:

```text
region
tenant
endpoint
customer tier
device
version
```

---

# PART LXXXIV — DEPLOYMENT VERSION

## 132. Correlation

Always make application version observable.

---

# PART LXXXV — REGION-AWARE DEPLOYMENT

## 133. Deployment

Deploy:

```text
Region A
 |
observe
 |
Region B
```

rather than all regions simultaneously when appropriate.

---

# PART LXXXVI — CLUSTER-AWARE DEPLOYMENT

## 134. Multi-Cluster

Progressive rollout:

```text
cluster A
 |
cluster B
 |
remaining
```

---

# PART LXXXVII — AZ-AWARE DEPLOYMENT

## 135. Availability Zones

For high-risk infrastructure changes, avoid changing every AZ simultaneously
when operationally feasible.

---

# PART LXXXVIII — DATABASE SAFETY

## 136. Migration

Separate schema migration from application behavior where necessary.

---

# PART LXXXIX — API VERSIONING

## 137. Compatibility

Use versioning or backward-compatible contracts to reduce deployment
coordination blast radius.

---

# PART XC — DEPENDENCY VERSIONING

## 138. Libraries

Pin or constrain dependency versions appropriately and test upgrades before
global adoption.

---

# PART XCI — CONTAINER BASE IMAGES

## 139. Base Image

A common vulnerable base image can affect many services.

Use:

```text
approved images
scanning
controlled promotion
```

---

# PART XCII — SUPPLY-CHAIN PROMOTION

## 140. Promotion

Promote trusted artifacts instead of allowing every environment to build
independently.

---

# PART XCIII — SECURITY PATCH

## 141. Patch Blast Radius

Test patches on:

```text
canary
noncritical
representative workloads
```

before broad deployment.

---

# PART XCIV — KERNEL/AMI

## 142. AMI Rollout

Use staged node-group replacement.

---

# PART XCV — KUBERNETES VERSION

## 143. Upgrade

Do not upgrade every critical cluster simultaneously unless the architecture
explicitly supports that risk.

---

# PART XCVI — PLATFORM BLAST RADIUS

## 144. Internal Developer Platform

A platform team can accidentally affect hundreds of services through one
shared template.

Use:

```text
versioning
canary
backward compatibility
```

---

# PART XCVII — GOLDEN PATH

## 145. Templates

Templates should provide safe defaults:

```text
resources
probes
PDB
security
observability
```

---

# PART XCVIII — PLATFORM VERSIONING

## 146. Version

Allow controlled migration between platform template versions.

---

# PART XCIX — API GATEWAY

## 147. Edge Protection

Use:

```text
authentication
rate limiting
WAF
quotas
```

to limit incoming blast radius.

---

# PART C — WAF

## 148. Security

WAF can reduce application-layer attack impact.

---

# PART CI — DDoS

## 149. Traffic Attack

Use appropriate:

```text
CDN
WAF
rate limits
autoscaling
provider protections
```

---

# PART CII — COST BLAST RADIUS

## 150. Cost

A runaway workload can create financial blast radius.

Use:

```text
budgets
quotas
autoscaling limits
anomaly detection
```

---

# PART CIII — RESOURCE LIMITS

## 151. Limits

Set maximums for:

```text
instances
pods
nodes
storage
requests
```

where appropriate.

---

# PART CIV — COST GUARDRAILS

## 152. Guardrails

Do not allow an automated scaling loop to grow without bounded limits.

---

# PART CV — INCIDENT COST

## 153. Emergency Changes

Emergency scaling should still have:

```text
upper bounds
owner
review
```

---

# PART CVI — OBSERVABILITY COST

## 154. Telemetry

High-cardinality telemetry can itself create operational and financial
blast radius.

---

# PART CVII — DATA RETENTION

## 155. Retention

Keep only the retention required for:

```text
operations
security
compliance
debugging
```

---

# PART CVIII — LOGGING TENANCY

## 156. Isolation

Separate critical security logs from ordinary application logs where required.

---

# PART CIX — SECURITY LOG PROTECTION

## 157. Audit

Application administrators should not be able to erase their own audit trail.

---

# PART CX — ACCOUNT COMPROMISE

## 158. Containment

If one production account is compromised:

```text
isolate
 |
revoke
 |
protect backups
 |
investigate
 |
recover
```

---

# PART CXI — REGION COMPROMISE

## 159. Recovery

A secondary region should have appropriate independent credentials and
recovery controls.

---

# PART CXII — CREDENTIAL COMPROMISE

## 160. Response

```text
identify
 |
disable
 |
rotate
 |
audit
 |
validate
```

---

# PART CXIII — DATA EXFILTRATION

## 161. Limit

Reduce data-access blast radius through:

```text
least privilege
network controls
encryption
data segmentation
```

---

# PART CXIV — INCIDENT COMMUNICATION

## 162. Scope

Communication should clearly distinguish:

```text
confirmed impact
suspected impact
```

---

# PART CXV — POSTMORTEM

## 163. Blast-Radius Review

After incidents ask:

```text
Why did the failure spread?
Which boundary failed?
Which boundary was missing?
Why did safeguards not stop propagation?
```

---

# PART CXVI — RESILIENCE TESTING

## 164. Game Day

Test:

```text
bad deployment
bad IAM
bad network rule
database failure
node failure
AZ failure
region failure
```

---

# PART CXVII — CHAOS

## 165. Controlled Failure

Chaos experiments should validate containment boundaries.

---

# PART CXVIII — BLAST-RADIUS SCORECARD

## 166. Example

| Dimension | Low Risk | High Risk |
|---|---|---|
| Deployment | canary | global |
| Credentials | service-specific | shared |
| Network | segmented | flat |
| Database | isolated | universal |
| Account | separated | shared |
| Region | redundant | single |
| Automation | bounded | unrestricted |
| Monitoring | independent | shared |
| Recovery | tested | assumed |

---

# PART CXIX — DESIGN REVIEW

## 167. Questions

```text
What happens if this deployment is wrong?
What happens if this credential leaks?
What happens if this node dies?
What happens if this AZ fails?
What happens if this account is compromised?
What happens if this region disappears?
```

---

# PART CXX — GOLDEN ARCHITECTURE

## 168. Reference

```text
                     Global Traffic
                           |
                      CDN / WAF
                           |
                    Global Routing
                           |
             +-------------+-------------+
             |                           |
          Region A                    Region B
             |                           |
       +-----+-----+               +-----+-----+
       |     |     |               |     |     |
      AZ1   AZ2   AZ3             AZ1   AZ2   AZ3
       |     |     |               |     |     |
     Nodes Nodes Nodes           Nodes Nodes Nodes
       |     |     |               |     |     |
      Pods  Pods  Pods            Pods  Pods  Pods
       |     |     |               |     |     |
       +-----+-----+               +-----+-----+
             |                           |
          Data Layer                 Replication
             |
       Observability
             |
      Independent Monitoring
```

---

# PART CXXI — SENIOR SCENARIOS

## 169. Bad Deployment

Question:

```text
How do you prevent one bad deployment from affecting 100% of customers?
```

Answer:

```text
Use canary deployment, automated health gates, business metrics, progressive
traffic expansion and automated or operator-approved rollback.
```

---

## 170. Bad IAM Policy

Question:

```text
How do you prevent one IAM mistake from breaking all environments?
```

Answer:

```text
Separate accounts and roles, use least privilege, policy validation, SCP
guardrails and staged policy deployment.
```

---

## 171. Compromised Pod

Question:

```text
How do you prevent lateral movement?
```

Answer:

```text
Use workload identity, least-privilege RBAC, network policies, isolated
secrets, restricted metadata access and service-specific credentials.
```

---

## 172. AZ Failure

Question:

```text
How do you keep the service available?
```

Answer:

```text
Distribute replicas across AZs, maintain surviving capacity, use resilient
load balancing and ensure stateful dependencies have appropriate HA.
```

---

## 173. Region Failure

Question:

```text
How do you reduce regional blast radius?
```

Answer:

```text
Use multi-region architecture where justified, independent recovery
capability, data replication, tested DR, global traffic management and
validated RTO/RPO.
```

---

## 174. Terraform Disaster

Question:

```text
How do you prevent one Terraform mistake from destroying production?
```

Answer:

```text
Use pull requests, plan review, policy-as-code, protected state, approvals,
scoped changes and staged applies.
```

---

## 175. Global Configuration Error

Question:

```text
How do you safely deploy global configuration?
```

Answer:

```text
Validate schema, deploy to a small scope first, monitor customer and system
metrics, then progressively expand.
```

---

## 176. Database Migration

Question:

```text
How do you reduce migration blast radius?
```

Answer:

```text
Use backward-compatible expand/migrate/contract patterns, test against
production-like data, monitor locks and latency, and maintain rollback or
forward-recovery plans.
```

---

# PART CXXII — PRODUCTION CHECKLIST

## 177. Deployment

```text
[ ] canary
[ ] health gates
[ ] rollback
[ ] version visibility
[ ] business metrics
```

---

## 178. Kubernetes

```text
[ ] topology spread
[ ] anti-affinity
[ ] resource requests
[ ] quotas
[ ] PDB
[ ] network policies
```

---

## 179. AWS

```text
[ ] multi-AZ
[ ] account separation
[ ] least privilege
[ ] backup
[ ] DR
[ ] cost limits
```

---

## 180. Security

```text
[ ] workload identity
[ ] secret isolation
[ ] image scanning
[ ] artifact signing
[ ] audit logs
[ ] break-glass
```

---

## 181. Automation

```text
[ ] scope
[ ] limits
[ ] preconditions
[ ] validation
[ ] rollback
```

---

# PART CXXIII — 250 PRODUCTION GOLDEN RULES

## 182. Rules 1–50

```text
1. Design every change with a blast-radius limit.
2. Assume every production change can be wrong.
3. Prefer reversible changes.
4. Prefer small changes.
5. Prefer progressive rollout.
6. Use canaries.
7. Use health gates.
8. Monitor business metrics.
9. Automate safe rollback.
10. Keep rollback artifacts available.
11. Do not deploy globally without justification.
12. Separate environments.
13. Separate critical accounts.
14. Separate critical credentials.
15. Use least privilege.
16. Use short-lived credentials.
17. Avoid shared production roles.
18. Avoid shared secrets.
19. Use service-specific identities.
20. Use workload identity.
21. Use network segmentation.
22. Use default-deny where appropriate.
23. Limit lateral movement.
24. Limit metadata access.
25. Use namespace boundaries appropriately.
26. Use resource quotas.
27. Use LimitRange.
28. Use PDB appropriately.
29. Use topology spreading.
30. Use anti-affinity where appropriate.
31. Maintain capacity for failure.
32. Avoid one-node concentration.
33. Avoid one-AZ concentration.
34. Avoid one-cluster concentration for critical systems.
35. Consider multi-region when justified.
36. Protect recovery environments.
37. Protect backups from production credentials.
38. Test backups.
39. Test restore.
40. Test failover.
41. Define RTO.
42. Define RPO.
43. Test RTO.
44. Test RPO.
45. Isolate critical databases.
46. Limit database connections.
47. Control expensive queries.
48. Use safe migrations.
49. Maintain backward compatibility.
50. Avoid global schema changes.
```

## 183. Rules 51–100

```text
51. Use bounded retries.
52. Use exponential backoff.
53. Use jitter.
54. Use timeouts.
55. Use circuit breakers.
56. Use bulkheads.
57. Use rate limits.
58. Use load shedding.
59. Protect critical traffic.
60. Isolate optional features.
61. Use queues for appropriate decoupling.
62. Use DLQs.
63. Prevent retry storms.
64. Prevent connection storms.
65. Prevent scaling storms.
66. Prevent restart storms.
67. Bound autoscaling.
68. Set maximum capacity.
69. Monitor resource exhaustion.
70. Monitor IP exhaustion.
71. Monitor disk exhaustion.
72. Monitor memory pressure.
73. Monitor CPU pressure.
74. Monitor network saturation.
75. Monitor database connections.
76. Monitor queue lag.
77. Monitor dependency latency.
78. Monitor dependency errors.
79. Monitor deployment version.
80. Monitor region.
81. Monitor cluster.
82. Monitor AZ.
83. Segment customer impact.
84. Segment tenant impact.
85. Segment endpoint impact.
86. Segment application-version impact.
87. Correlate changes with incidents.
88. Keep deployment timestamps observable.
89. Keep configuration versions observable.
90. Keep infrastructure versions observable.
91. Review global configuration.
92. Validate configuration before rollout.
93. Stage configuration.
94. Protect DNS changes.
95. Stage traffic changes.
96. Protect certificates.
97. Automate certificate renewal.
98. Protect secrets.
99. Rotate secrets safely.
100. Audit emergency access.
```

## 184. Rules 101–150

```text
101. Protect audit logs.
102. Use centralized security logging.
103. Separate logging from application failure where possible.
104. Prevent log storms.
105. Prevent metric-cardinality explosions.
106. Control trace volume.
107. Control telemetry retention.
108. Protect observability infrastructure.
109. Maintain independent monitoring.
110. Maintain external synthetic checks.
111. Protect CI/CD.
112. Avoid making CI/CD a runtime dependency.
113. Protect artifact registries.
114. Scan artifacts.
115. Sign trusted artifacts where appropriate.
116. Promote immutable artifacts.
117. Avoid rebuilding different artifacts per environment.
118. Protect Git repositories.
119. Protect main branches.
120. Require appropriate reviews.
121. Validate GitOps manifests.
122. Stage GitOps synchronization.
123. Protect Terraform state.
124. Review Terraform plans.
125. Use policy-as-code.
126. Scope Terraform changes.
127. Stage infrastructure changes.
128. Protect network routes.
129. Stage firewall changes.
130. Avoid broad security-rule modifications.
131. Protect IAM policies.
132. Validate IAM changes.
133. Use SCP guardrails.
134. Separate security administration.
135. Separate application administration.
136. Protect root credentials.
137. Protect break-glass procedures.
138. Use JIT access where appropriate.
139. Audit privileged actions.
140. Restrict production access.
141. Reduce tenant blast radius.
142. Use tenant quotas.
143. Use tenant rate limits.
144. Protect shared infrastructure.
145. Control noisy neighbors.
146. Isolate batch workloads.
147. Isolate critical workloads.
148. Use dedicated node pools when justified.
149. Diversify capacity types.
150. Do not depend entirely on Spot for critical workloads.
```

## 185. Rules 151–200

```text
151. Stage AMI changes.
152. Stage Kubernetes upgrades.
153. Stage platform template changes.
154. Version platform APIs.
155. Maintain backward compatibility.
156. Test platform changes with representative workloads.
157. Use golden paths with safe defaults.
158. Provide rollback for platform releases.
159. Control image base versions.
160. Scan dependencies.
161. Lock dependencies appropriately.
162. Protect package repositories.
163. Scan source for secrets.
164. Rotate leaked credentials immediately.
165. Investigate credential exposure.
166. Preserve forensic evidence.
167. Contain compromised workloads.
168. Restrict egress where justified.
169. Restrict east-west traffic.
170. Protect sensitive data.
171. Segment data by tenant where required.
172. Encrypt sensitive data.
173. Limit decryption permissions.
174. Protect KMS administration.
175. Protect backup deletion.
176. Use immutable backups when justified.
177. Keep recovery credentials independent.
178. Keep DR infrastructure sufficiently independent.
179. Test regional recovery.
180. Test account recovery.
181. Test credential recovery.
182. Test certificate recovery.
183. Test DNS recovery.
184. Test database recovery.
185. Test observability recovery.
186. Test CI/CD recovery.
187. Conduct game days.
188. Conduct controlled chaos.
189. Define chaos abort criteria.
190. Limit experiment scope.
191. Measure blast radius during tests.
192. Measure recovery time.
193. Measure customer impact.
194. Review failed assumptions.
195. Fix missing isolation.
196. Fix common-mode failures.
197. Fix correlated failures.
198. Track resilience debt.
199. Track single points of failure.
200. Review architecture after major incidents.
```

## 186. Rules 201–250

```text
201. Treat failure-domain design and blast-radius reduction as related but
     distinct disciplines.
202. Failure-domain design identifies where failure can spread.
203. Blast-radius reduction limits how far that failure can spread.
204. Use multiple independent controls.
205. Do not depend on one safety mechanism.
206. Make critical paths small.
207. Keep optional paths detachable.
208. Make dependencies explicit.
209. Document fallback behavior.
210. Document failure behavior.
211. Document recovery behavior.
212. Test documented behavior.
213. Make incident runbooks executable.
214. Automate repetitive safe actions.
215. Bound every automated action.
216. Require preconditions for automation.
217. Require post-action validation.
218. Stop automation when assumptions fail.
219. Require approval for destructive high-risk actions.
220. Keep emergency changes auditable.
221. Reconcile emergency changes into desired state.
222. Keep GitOps state accurate.
223. Keep Terraform state accurate.
224. Remove configuration drift.
225. Monitor drift.
226. Monitor policy drift.
227. Monitor identity drift.
228. Monitor network drift.
229. Monitor resource drift.
230. Monitor deployment drift.
231. Use progressive promotion.
232. Use environment gates.
233. Use regional gates.
234. Use cluster gates.
235. Use AZ-aware rollout where useful.
236. Protect database migrations.
237. Protect API compatibility.
238. Protect client compatibility.
239. Protect message compatibility.
240. Protect recovery paths.
241. Protect customer-critical workflows.
242. Measure customer impact, not only infrastructure health.
243. Design degraded modes deliberately.
244. Practice degraded modes.
245. Practice failover.
246. Practice rollback.
247. Practice containment.
248. A resilient system makes failures local, bounded and reversible whenever
     possible.
249. The best blast-radius control is a combination of isolation, progressive
     change, least privilege, bounded resources and tested recovery.
250. The ultimate objective is not zero failures; it is preventing one failure
     from becoming an uncontrolled production-wide event.
```

# END OF 26-Blast-Radius-Reduction.md
