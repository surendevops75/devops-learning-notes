# Multi-Region-Architecture

## 1. Purpose

This file is a production-grade guide to designing AWS and Kubernetes
platforms across multiple geographic regions.

Multi-region architecture is not simply:

```text
Region A -> EKS
Region B -> EKS
```

A real multi-region design must define:

```text
traffic
compute
data
identity
secrets
artifacts
DNS
networking
observability
deployment
failure detection
failover
recovery
cost
compliance
```

The fundamental question is:

```text
What exactly must survive a complete regional failure?
```

Everything else follows from that requirement.

---

# PART I — FOUNDATIONS

## 2. What Is Multi-Region Architecture?

A multi-region architecture operates workloads or recovery capability in
more than one AWS region.

Example:

```text
                    Global Users
                         |
                  Global Traffic
                    Management
                    /          \
                   /            \
             Region A          Region B
                |                 |
              EKS A             EKS B
                |                 |
             Apps A            Apps B
                |                 |
              Data <----------> Data
```

---

## 3. Why Multi-Region?

Common reasons:

```text
regional disaster recovery
low latency
data residency
business continuity
global availability
capacity
maintenance independence
```

Do not choose multi-region merely because it sounds more resilient.

It adds:

```text
data replication
traffic management
operational complexity
cost
deployment complexity
observability complexity
```

---

## 4. Multi-AZ vs Multi-Region

Multi-AZ protects primarily against:

```text
Availability Zone failure
```

Multi-region protects against:

```text
regional failure
```

A good progression is:

```text
single AZ
      |
multi-AZ
      |
multi-cluster
      |
multi-region
```

Do not skip foundational HA and assume regions solve everything.

---

# PART II — REQUIREMENTS

## 5. Start With Business Requirements

Ask:

```text
What is the RTO?
What is the RPO?
How many users?
Which countries?
What latency target?
Can traffic be active in both regions?
Can data be replicated?
What consistency is required?
What services must survive?
What services may be restored later?
```

---

## 6. RTO

RTO means:

```text
maximum acceptable time to restore service
```

Examples:

```text
RTO = 5 minutes
RTO = 30 minutes
RTO = 4 hours
```

The architecture changes significantly depending on the target.

---

## 7. RPO

RPO means:

```text
maximum acceptable amount of data loss
```

Examples:

```text
RPO = near zero
RPO = 5 minutes
RPO = 1 hour
```

RPO determines replication requirements.

---

# PART III — TOPOLOGIES

## 8. Active-Active

```text
Users
 |
Global Traffic
 |
+---------+---------+
|                   |
Region A           Region B
|                   |
EKS A              EKS B
|                   |
Apps                Apps
```

Both regions serve production traffic.

Benefits:

```text
fast failover
resource utilization
regional latency
```

Costs:

```text
data consistency
deployment coordination
traffic complexity
```

---

## 9. Active-Passive

```text
Primary
 |
Region A
 |
failure
 |
Region B
 |
Recovery
```

Secondary region may be:

```text
cold
warm
hot
```

The required RTO determines how much capacity remains ready.

---

## 10. Warm Standby

```text
Region A
 |
100% traffic

Region B
 |
minimal capacity
```

During failure:

```text
scale Region B
 |
traffic shift
```

RTO depends on:

```text
node startup
application startup
data recovery
traffic propagation
```

---

## 11. Pilot Light

Keep only essential infrastructure/data capability ready.

```text
Primary
 |
full application

DR
 |
critical data + minimal infrastructure
```

Lower cost but slower recovery.

---

## 12. Cold DR

```text
Primary
 |
failure
 |
provision DR infrastructure
 |
restore data
 |
deploy
 |
traffic
```

Lowest ongoing cost but longest recovery time.

---

# PART IV — GLOBAL TRAFFIC

## 13. Global Traffic Architecture

```text
User
 |
DNS / Global Traffic
 |
+----------------------+
|                      |
Region A              Region B
|                      |
Load Balancer         Load Balancer
|                      |
EKS                    EKS
```

Traffic can be controlled through AWS global DNS and traffic-management
services according to the selected design.

---

## 14. Route 53 Failover

Concept:

```text
Primary endpoint
 |
health check
 |
healthy -> traffic

unhealthy
 |
secondary endpoint
```

Consider:

```text
health-check semantics
TTL
resolver caching
application warm-up
```

DNS failover is not instantaneous.

---

## 15. Weighted Routing

Example:

```text
Region A -> 90%
Region B -> 10%
```

Useful for:

```text
canary
migration
regional testing
capacity validation
```

---

## 16. Latency Routing

Users can be directed toward a region based on expected network latency.

But:

```text
network latency != total application latency
```

Application dependencies and database location also matter.

---

## 17. Geolocation Routing

Useful for:

```text
data residency
regional users
business requirements
```

Do not use geography as a substitute for a legal/compliance analysis.

---

## 18. Anycast / Global Accelerator

For workloads requiring faster global traffic steering, a global network
front door can provide a more controlled path than DNS-only failover.

Evaluate:

```text
static entry points
health-based routing
failover speed
application protocol
cost
```

---

# PART V — REGION DESIGN

## 19. Symmetric Regions

```text
Region A
 |
EKS
Ingress
Observability
Applications

Region B
 |
EKS
Ingress
Observability
Applications
```

Advantages:

```text
easy failover
consistent operations
```

Cost:

```text
duplicate infrastructure
```

---

## 20. Asymmetric Regions

```text
Primary Region
 |
full capacity

DR Region
 |
reduced capacity
```

Cost efficient but requires scaling during recovery.

---

## 21. Region Pair Selection

Consider:

```text
service availability
distance
latency
compliance
data residency
pricing
capacity
quotas
operational expertise
```

Do not assume every AWS service is available identically in every region.

---

# PART VI — EKS MULTI-REGION

## 22. Regional EKS Clusters

Recommended conceptual model:

```text
Region A
 |
VPC
 |
EKS A
 |
apps

Region B
 |
VPC
 |
EKS B
 |
apps
```

Keep clusters operationally independent enough to survive regional
management-plane isolation.

---

## 23. EKS Control Plane Independence

Each regional cluster has its own:

```text
Kubernetes API
controllers
scheduler
cluster addons
node capacity
GitOps reconciliation
```

Do not make one regional cluster a mandatory runtime dependency for another.

---

## 24. Cluster Fleet

```text
Global Platform
 |
+-- EKS us-east
+-- EKS eu-west
+-- EKS ap-south
```

Use standardized cluster modules and fleet metadata.

---

# PART VII — NETWORKING

## 25. Regional VPCs

Example:

```text
Region A VPC
10.10.0.0/16

Region B VPC
10.20.0.0/16
```

Avoid overlapping CIDRs when direct inter-region connectivity may be needed.

---

## 26. Inter-Region Connectivity

Possible approaches include:

```text
Transit Gateway peering
VPC peering
Cloud WAN
private application endpoints
```

Choose according to scale and connectivity requirements.

---

## 27. Cross-Region Application Calls

Avoid unnecessary synchronous dependencies:

```text
Region A service
 |
network
 |
Region B service
```

Regional failures can turn this into:

```text
latency
timeouts
retry storms
```

Prefer local dependencies when possible.

---

## 28. Regional Cell Architecture

```text
Global
 |
+-- Region A Cell
|    +-- EKS
|    +-- Data
|
+-- Region B Cell
     +-- EKS
     +-- Data
```

A regional cell should minimize dependencies on another region.

---

# PART VIII — DATA

## 29. Data Is the Hardest Part

Compute can often be recreated:

```text
Terraform
+
container image
+
GitOps
```

Data cannot always be recreated.

Therefore:

```text
multi-region compute
```

is much easier than:

```text
multi-region state
```

---

## 30. Data Classification

Classify workloads:

```text
stateless
stateful but reconstructable
stateful critical
```

Then design replication appropriately.

---

## 31. Active-Passive Database

```text
Region A
 |
Primary DB
 |
replication
 |
Region B
 |
Standby DB
```

Failover must include:

```text
application endpoint
credentials
DNS
connection strings
```

---

## 32. Active-Active Database

```text
Region A DB <----> Region B DB
```

Requires a consistency/conflict strategy.

Potential problems:

```text
write conflicts
replication lag
split brain
conflict resolution
```

Do not choose active-active simply to sound highly available.

---

## 33. RDS Multi-Region

For relational workloads, evaluate supported cross-region replication
features and their actual consistency/failover characteristics.

Architecture:

```text
Primary Region
 |
DB
 |
cross-region replication
 |
DR Region
 |
DB
```

---

## 34. DynamoDB Global Tables

For workloads suited to DynamoDB, multi-region replicated tables can provide
regional data access and multi-region write capability according to the
selected configuration.

Still design for:

```text
conflicts
latency
cost
capacity
application semantics
```

---

## 35. S3 Cross-Region Replication

Object replication:

```text
Bucket A
 |
CRR
 |
Bucket B
```

Validate:

```text
replication status
versioning
permissions
encryption
delete behavior
RPO
```

---

## 36. EBS and Region Failure

EBS volumes are region/AZ scoped resources.

For DR:

```text
snapshot
 |
copy / recovery
 |
new region
 |
new volume
```

Do not assume an EKS pod using EBS can simply start in another region.

---

# PART IX — OBJECTS AND ARTIFACTS

## 37. Container Registry

A multi-region platform needs reliable image availability.

Options:

```text
regional registries
replication
cross-region pulls
```

Prefer:

```text
same immutable image
same digest
```

across regions.

---

## 38. Registry Failure

If registry becomes unavailable:

```text
existing pods
 |
may continue

new pod
 |
image pull
 |
may fail
```

Therefore retain critical images and test regional recovery.

---

# PART X — SECRETS

## 39. Regional Secrets

A workload in Region A should have appropriate access to Region A secrets.

If Region B must operate independently:

```text
Region B
 |
regional secret
```

rather than requiring every runtime request to cross regions.

---

## 40. Secret Replication

```text
Source
 |
controlled replication
 |
Region A secret
Region B secret
```

Validate rotation propagation.

---

# PART XI — IDENTITY

## 41. Regional Workload Identity

Each regional cluster should have its own workload identity configuration.

Example:

```text
EKS A
 |
ServiceAccount
 |
IAM role A

EKS B
 |
ServiceAccount
 |
IAM role B
```

This limits compromise scope.

---

# PART XII — DEPLOYMENT

## 42. Multi-Region Deployment

Basic:

```text
Build once
 |
Artifact
 |
Deploy Region A
 |
Validate
 |
Deploy Region B
```

This reduces simultaneous failure risk.

---

## 43. Active-Active Deployment

For active-active:

```text
Region A -> new version
Region B -> old version
```

Ensure backward compatibility.

---

## 44. Database Migration

Use:

```text
expand
 |
compatible application
 |
migrate
 |
validate
 |
contract
```

Never depend on instant rollback for destructive schema changes.

---

# PART XIII — GITOPS

## 45. Regional GitOps

Example:

```text
Git
 |
clusters/
 |
+-- us-east
+-- eu-west
```

Argo CD or another GitOps controller reconciles each regional cluster.

---

## 46. Regional Overlays

Common base:

```text
application base
```

Regional differences:

```text
replicas
domain
storage
region-specific endpoints
```

Keep differences minimal.

---

## 47. Global Rollout

Use:

```text
Region A
 |
observe
 |
Region B
```

For more regions:

```text
Canary
 |
Wave 1
 |
Wave 2
 |
Wave 3
```

---

# PART XIV — OBSERVABILITY

## 48. Regional Observability

Each region should have enough telemetry to diagnose local failures.

```text
Region A
 |
metrics/logs/traces
 |
regional buffer
 |
central aggregation
```

---

## 49. Global Dashboard

Show:

```text
Region
Cluster
Application
Traffic
Error rate
Latency
Capacity
Deployment
Data replication
```

---

## 50. Regional Health

Monitor:

```text
API availability
node health
pod health
load balancer health
DNS
CNI
storage
database replication
queue lag
```

---

# PART XV — FAILOVER

## 51. Failover Chain

```text
Regional failure
 |
health detection
 |
traffic removal
 |
secondary activation
 |
application health
 |
data validation
 |
traffic restoration
```

---

## 52. Failover Must Be Automated Carefully

Fully automatic failover can create:

```text
false-positive failover
split brain
traffic oscillation
```

Therefore define:

```text
automatic conditions
manual approval
recovery procedure
```

based on business criticality.

---

## 53. Failback

After primary recovery:

```text
secondary active
 |
primary repaired
 |
data synchronization
 |
validation
 |
controlled traffic shift
 |
primary active
```

Do not immediately fail back because the primary is technically online.

---

# PART XVI — SPLIT BRAIN

## 54. Split Brain

Dangerous scenario:

```text
Region A thinks it is primary
Region B thinks it is primary
```

Potential consequences:

```text
conflicting writes
duplicate processing
data corruption
```

Design explicit leadership/ownership semantics for stateful systems.

---

# PART XVII — QUEUES

## 55. Multi-Region Queue

Prefer regional processing:

```text
Region A queue -> Region A workers
Region B queue -> Region B workers
```

Cross-region queue dependencies should be justified.

---

## 56. Message Duplication

During failover:

```text
message
 |
retry
 |
processed twice
```

Use:

```text
idempotency
deduplication
transactional design
```

---

# PART XVIII — CACHING

## 57. Regional Cache

```text
Region A -> cache A
Region B -> cache B
```

Usually preferable to forcing every cache request across regions.

Treat cache as rebuildable when possible.

---

# PART XIX — GLOBAL RATE LIMITING

## 58. Rate Limits

Global traffic may hit:

```text
Region A
Region B
```

Ensure rate limits are designed for:

```text
global
regional
tenant
user
```

scope.

---

# PART XX — CERTIFICATES

## 59. Regional Certificates

Ensure certificates and certificate automation exist in every required
region.

A certificate available in one regional workflow does not automatically
solve every regional endpoint requirement.

---

# PART XXI — DNS

## 60. DNS TTL

Low TTL can improve failover responsiveness but increases DNS query volume.

High TTL:

```text
lower query volume
slower failover
```

Choose deliberately.

---

## 61. DNS Failover Testing

Test:

```text
healthy primary
 |
failure
 |
secondary
```

Measure actual time until meaningful user traffic reaches the secondary.

---

# PART XXII — SECURITY

## 62. Regional Security Boundaries

Do not assume regions are security boundaries.

Use:

```text
account
IAM
VPC
security groups
NetworkPolicy
RBAC
admission
```

---

## 63. Cross-Region IAM

Prefer regional workloads to use appropriate local identities.

Avoid broad policies such as:

```text
all regions -> all resources
```

unless genuinely required.

---

# PART XXIII — DR

## 64. DR Levels

### Level 1

```text
backup only
```

### Level 2

```text
backup + infrastructure code
```

### Level 3

```text
warm standby
```

### Level 4

```text
active-active
```

Choose based on RTO/RPO.

---

## 65. DR Readiness

A DR region is not ready if:

```text
images unavailable
secrets unavailable
DNS unavailable
IAM unavailable
data stale
GitOps unavailable
```

---

# PART XXIV — DR TESTING

## 66. Regional Game Day

Test:

```text
disable primary traffic
 |
secondary activation
 |
data validation
 |
application validation
 |
user traffic
```

Record:

```text
detection time
failover time
RTO
RPO
errors
manual steps
```

---

# PART XXV — COST

## 67. Multi-Region Cost

Costs can include:

```text
duplicate compute
duplicate load balancers
replication
cross-region data transfer
observability
storage
NAT
```

Active-active can be expensive because both regions run substantial
capacity continuously.

---

# PART XXVI — CAPACITY

## 68. DR Capacity

If secondary normally runs at 20% capacity:

```text
primary failure
 |
traffic becomes 100%
```

The DR region must be able to scale rapidly enough to meet RTO.

---

# PART XXVII — GLOBAL CAPACITY

## 69. Capacity Model

For active-active:

```text
Region A -> 50%
Region B -> 50%
```

After Region A failure:

```text
Region B -> 100%
```

Therefore Region B needs enough spare capacity.

---

# PART XXVIII — PLATFORM FAILURE

## 70. Shared Global Dependencies

Dangerous architecture:

```text
All Regions
 |
single global dependency
 |
failure
 |
all regions affected
```

Examples:

```text
single secret endpoint
single GitOps controller
single DNS dependency
single observability gateway
```

Evaluate every shared dependency.

---

# PART XXIX — MULTI-REGION GITOPS

## 71. Management Plane

Possible:

```text
Central Git
 |
Regional Argo A
Regional Argo B
```

This provides:

```text
central desired state
regional reconciliation
```

---

# PART XXX — CENTRAL VS REGIONAL CONTROL

## 72. Central Control

Benefits:

```text
single view
simple governance
```

Risk:

```text
central dependency
```

---

## 73. Regional Control

Benefits:

```text
regional autonomy
failure isolation
```

Costs:

```text
more components
more operations
```

---

# PART XXXI — APPLICATION DESIGN

## 74. Stateless First

The easiest multi-region service:

```text
immutable image
stateless pods
regional dependencies
```

The hardest:

```text
stateful application
strong consistency
cross-region writes
```

---

# PART XXXII — RESILIENCE

## 75. Timeouts

Cross-region calls should use conservative:

```text
timeouts
retries
backoff
```

---

## 76. Retry Storm

Bad:

```text
Region A
 |
Region B unavailable
 |
retry x100
 |
Region B overload
```

Use bounded retries and circuit breaking.

---

# PART XXXIII — DATA CONSISTENCY

## 77. Strong Consistency

Cross-region strong consistency usually introduces:

```text
latency
availability trade-offs
complexity
```

Use it only when business requirements require it.

---

## 78. Eventual Consistency

Can improve:

```text
availability
latency
regional independence
```

But applications must tolerate stale data.

---

# PART XXXIV — COMPLIANCE

## 79. Data Residency

Some workloads may require:

```text
EU data -> EU region
India data -> approved India region
```

The architecture must also consider:

```text
logs
backups
replicas
telemetry
support access
```

---

# PART XXXV — FAILURE SCENARIOS

## 80. Region A Lost

```text
Region A
 |
failure
 |
global traffic
 |
Region B
```

Validate:

```text
capacity
data
secrets
images
identity
```

---

## 81. Region A Network Partition

If Region A is partially isolated:

```text
global traffic
 |
may still see endpoint
```

Health checks must represent real application health rather than merely
network reachability.

---

## 82. Database Replication Lag

During failover:

```text
primary failure
 |
replica
 |
lag
 |
RPO impact
```

Measure actual replication lag.

---

## 83. Registry Failure

If image registry is unavailable:

```text
existing workloads -> may continue
new deployments -> image pull failure
```

Regional artifact availability reduces this risk.

---

## 84. DNS Failure

If global DNS is unavailable:

```text
traffic steering
 |
may fail
```

Existing connections may continue while new routing can be affected.

---

# PART XXXVI — SENIOR DESIGN SCENARIOS

## 85. Design Global E-Commerce

Requirements:

```text
global users
high availability
regional latency
```

Architecture:

```text
Global Traffic
 |
+-------------+-------------+
|                           |
Region A                   Region B
EKS                         EKS
|                           |
apps                        apps
|                           |
regional data + replication
```

Use active-active where data semantics support it.

---

## 86. Design 5-Minute RTO

A likely direction:

```text
warm/hot secondary
+
pre-provisioned capacity
+
replicated data
+
automated traffic failover
```

Cold DR is unlikely to satisfy a strict 5-minute target reliably.

---

## 87. Design Near-Zero RPO

Focus on:

```text
replication
write durability
cross-region data architecture
failover correctness
```

Compute duplication alone does not produce near-zero RPO.

---

## 88. Design 100 Clusters Across 3 Regions

Use:

```text
regional fleet
 |
central standards
 |
regional GitOps
 |
cluster labels
 |
deployment waves
 |
central observability
```

Avoid one global control dependency for every runtime operation.

---

# PART XXXVII — PRODUCTION CHECKLIST

## 89. Architecture

```text
[ ] RTO defined
[ ] RPO defined
[ ] active-active/passive selected
[ ] region pair justified
[ ] failure domains documented
```

## 90. Traffic

```text
[ ] global routing
[ ] health checks
[ ] TTL strategy
[ ] failover
[ ] failback
[ ] traffic testing
```

## 91. Data

```text
[ ] replication
[ ] backup
[ ] restore
[ ] consistency model
[ ] replication lag
[ ] failover procedure
```

## 92. Platform

```text
[ ] regional EKS
[ ] regional addons
[ ] GitOps
[ ] workload identity
[ ] secrets
[ ] registry
[ ] observability
```

---

# PART XXXVIII — 150 PRODUCTION GOLDEN RULES

## 93. Rules 1–30

```text
1. Start with RTO and RPO.
2. Define what must survive regional failure.
3. Multi-AZ is not multi-region.
4. Do not choose multi-region without a business reason.
5. Understand the operational cost.
6. Choose active-active only when justified.
7. Choose active-passive when simpler recovery is sufficient.
8. Use warm standby when RTO requires it.
9. Use cold DR when business requirements permit it.
10. Compute is easier to recover than data.
11. Treat data as the hardest dependency.
12. Avoid unnecessary cross-region synchronous calls.
13. Prefer regional dependencies.
14. Keep regional cells independently operable.
15. Plan global traffic routing.
16. Test DNS failover.
17. Understand DNS caching.
18. Use weighted routing deliberately.
19. Use latency routing deliberately.
20. Use geographic routing only for real requirements.
21. Evaluate global traffic services against protocol needs.
22. Use multi-AZ EKS inside each region.
23. Do not depend on another region's control plane for runtime.
24. Standardize regional clusters.
25. Keep cluster-specific differences minimal.
26. Plan non-overlapping CIDRs.
27. Design inter-region connectivity intentionally.
28. Avoid unnecessary network meshes.
29. Protect regional network independence.
30. Document shared global dependencies.
```

## 94. Rules 31–60

```text
31. Define database consistency requirements.
32. Define replication mode.
33. Measure replication lag.
34. Test database failover.
35. Test database failback.
36. Do not assume a replica equals zero-RPO.
37. Do not assume active-active equals conflict-free.
38. Design write ownership.
39. Prevent split brain.
40. Design idempotency.
41. Use immutable container images.
42. Ensure image availability in every required region.
43. Replicate critical artifacts.
44. Retain recovery versions.
45. Replicate required secrets.
46. Avoid runtime dependence on another region's secret store.
47. Use regional workload identity.
48. Limit cross-region IAM.
49. Keep regional certificates available.
50. Monitor certificate renewal.
51. Deploy through progressive regional waves.
52. Build once and promote the same artifact.
53. Test backward compatibility.
54. Use expand/migrate/contract database changes.
55. Never assume application rollback undoes data changes.
56. Use regional GitOps reconciliation.
57. Avoid a single regional controller as a runtime dependency.
58. Protect GitOps repositories.
59. Stop bad deployment waves.
60. Revert bad desired state safely.
```

## 95. Rules 61–90

```text
61. Collect regional telemetry.
62. Maintain global fleet dashboards.
63. Monitor data replication.
64. Monitor traffic distribution.
65. Monitor regional capacity.
66. Monitor DNS health.
67. Monitor load balancer health.
68. Monitor EKS health.
69. Monitor CNI.
70. Monitor storage.
71. Alert on meaningful regional degradation.
72. Avoid duplicate alert storms.
73. Define failover triggers.
74. Decide automatic vs manual failover.
75. Test false-positive behavior.
76. Design failback.
77. Do not fail back immediately after recovery.
78. Synchronize data before failback.
79. Validate application health before traffic shift.
80. Test user-visible failover.
81. Test regional outage.
82. Test regional network partition.
83. Test registry failure.
84. Test DNS failure.
85. Test identity failure.
86. Test secret failure.
87. Test database failure.
88. Test GitOps failure.
89. Test observability failure.
90. Measure actual RTO.
```

## 96. Rules 91–120

```text
91. Measure actual RPO.
92. Test DR regularly.
93. Document manual recovery steps.
94. Automate repeatable recovery.
95. Keep infrastructure as code.
96. Keep platform configuration as code.
97. Keep data recovery procedures separate from application deployment.
98. Protect recovery credentials.
99. Protect recovery data.
100. Verify backup integrity.
101. Validate cross-region encryption.
102. Validate KMS recovery.
103. Validate S3 replication.
104. Validate database replication.
105. Validate artifact replication.
106. Validate secret replication.
107. Validate DNS recovery.
108. Validate IAM recovery.
109. Maintain sufficient DR capacity.
110. Calculate post-failure capacity.
111. Active-active requires spare capacity after one region fails.
112. Warm standby requires rapid scaling.
113. Cold DR requires realistic provisioning time.
114. Include node startup in RTO.
115. Include data recovery in RTO.
116. Include DNS/traffic convergence in RTO.
117. Include application warm-up in RTO.
118. Include human approval time in RTO.
119. Do not claim an RTO shorter than your slowest required recovery step.
120. Test the complete chain.
```

## 97. Rules 121–150

```text
121. Consider data residency.
122. Consider backup residency.
123. Consider log residency.
124. Consider telemetry residency.
125. Consider support access.
126. Minimize global dependencies.
127. Protect global traffic management.
128. Avoid global single points of failure.
129. Use regional observability buffering where appropriate.
130. Use regional egress where appropriate.
131. Avoid cross-region retry storms.
132. Use timeouts.
133. Use bounded retries.
134. Use backoff.
135. Use circuit breaking where appropriate.
136. Use idempotent operations.
137. Design for duplicate messages.
138. Design for stale data where applicable.
139. Define consistency explicitly.
140. Define ownership explicitly.
141. Standardize regional infrastructure.
142. Standardize cluster addons.
143. Standardize security.
144. Standardize GitOps.
145. Standardize observability.
146. Use controlled exceptions.
147. Document architectural trade-offs.
148. Test every resilience claim.
149. Optimize cost only after reliability requirements are understood.
150. A multi-region architecture is successful when a regional failure
     becomes a controlled recovery event rather than an uncontrolled
     business outage.
```
---