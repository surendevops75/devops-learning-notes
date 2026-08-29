# 19-DevOps-System-Design
# 07-Multi-Cluster-Architecture

## 1. Purpose

This file is a production-focused guide to designing, operating, securing,
scaling, and recovering Kubernetes environments distributed across multiple
clusters.

The central principle is:

```text
Multiple clusters are an architectural boundary,
not merely multiple Kubernetes API endpoints.
```

A serious multi-cluster design must answer:

```text
Why do we need multiple clusters?
Where should workloads run?
Who controls each cluster?
How are clusters provisioned?
How are applications promoted?
How do clusters communicate?
How is identity managed?
How is traffic routed?
How are secrets distributed?
How are policies enforced?
How are upgrades performed?
What happens when one cluster fails?
What happens when an entire region fails?
How is blast radius limited?
How is cost controlled?
```

---

# PART I — FOUNDATIONS

## 2. What Is Multi-Cluster Architecture?

A multi-cluster platform contains two or more independent Kubernetes
clusters operated as part of one platform.

Example:

```text
                    Platform Control
                          |
                    GitOps / Fleet
                          |
          +---------------+---------------+
          |               |               |
       Cluster A       Cluster B       Cluster C
          |               |               |
        Prod-A          Prod-B           DR
```

Clusters may be:

```text
same region
different regions
same AWS account
different AWS accounts
same environment
different environments
```

The architecture should be driven by isolation, scale, availability,
compliance, and operational requirements.

---

## 3. Why Multiple Clusters?

Common reasons:

```text
failure isolation
security isolation
compliance
regional deployment
organizational boundaries
upgrade independence
resource isolation
scale
team autonomy
business-unit separation
```

Do not create clusters simply because Kubernetes supports them.

Every additional cluster creates:

```text
control-plane lifecycle
node lifecycle
networking
DNS
IAM
observability
security
GitOps
upgrade
backup
cost
```

operational overhead.

---

## 4. Single Cluster vs Multi-Cluster

### Single Cluster

Advantages:

```text
lower operational overhead
shared platform
simpler networking
simpler observability
better resource pooling
```

Disadvantages:

```text
larger blast radius
shared failure domain
harder isolation
upgrade affects more workloads
resource contention
```

### Multi-Cluster

Advantages:

```text
failure isolation
independent upgrades
security boundaries
regional distribution
capacity partitioning
```

Disadvantages:

```text
higher cost
higher operational complexity
cross-cluster networking
duplicate addons
duplicate observability
fleet management
```

---

## 5. Decision Framework

Choose multiple clusters when the benefit of isolation exceeds the
operational cost.

A useful decision model:

```text
business criticality
+
security requirement
+
scale
+
failure-domain requirement
+
compliance
+
upgrade independence
```

against:

```text
operational complexity
+
cost
+
network complexity
+
platform staffing
```

---

# PART II — CLUSTER TOPOLOGIES

## 6. Environment-Based Clusters

Example:

```text
dev-cluster
stage-cluster
prod-cluster
```

Simple and common.

---

## 7. Regional Clusters

```text
Region A
 |
prod-a

Region B
 |
prod-b
```

Useful for:

```text
latency
regional resilience
data residency
traffic localization
```

---

## 8. Cell-Based Architecture

A cell is an independently operating slice of the platform.

```text
Global
 |
+-- Cell A
|    |
|    +-- Cluster
|
+-- Cell B
|    |
|    +-- Cluster
|
+-- Cell C
     |
     +-- Cluster
```

Cell architecture reduces blast radius.

If Cell B fails:

```text
Cell A -> healthy
Cell B -> failed
Cell C -> healthy
```

rather than affecting the entire fleet.

---

## 9. Active-Active

```text
Users
 |
Global traffic
 |
+--------+--------+
|                 |
Cluster A       Cluster B
|                 |
App               App
```

Both clusters serve production traffic.

Benefits:

```text
better utilization
lower failover latency
continuous resilience
```

Costs:

```text
data consistency complexity
traffic management
deployment coordination
```

---

## 10. Active-Passive

```text
Primary Cluster
      |
      | failure
      v
Secondary Cluster
```

The secondary cluster may have:

```text
warm capacity
partial capacity
full capacity
```

depending on RTO requirements.

---

# PART III — CONTROL PLANE STRATEGY

## 11. Centralized GitOps

One management cluster can operate multiple target clusters.

```text
                Argo CD
                   |
        +----------+----------+
        |          |          |
     Cluster A  Cluster B  Cluster C
```

Benefits:

```text
centralized policy
centralized application visibility
simple developer experience
fleet-level control
```

Risks:

```text
management-plane failure
credential concentration
larger blast radius
network dependency
```

---

## 12. Distributed GitOps

Each cluster runs its own GitOps controller.

```text
Git
 |
+-- Cluster A Argo
+-- Cluster B Argo
+-- Cluster C Argo
```

Benefits:

```text
local autonomy
smaller control-plane blast radius
clusters can reconcile independently
```

Costs:

```text
more management
duplicate components
fleet visibility complexity
```

---

## 13. Hybrid GitOps

A common enterprise approach:

```text
Central platform
 |
fleet configuration
 |
+-- cluster-local Argo
+-- cluster-local addons
```

Central Git controls desired configuration while each cluster performs
local reconciliation.

---

## 14. Choosing Central vs Distributed

Centralized is attractive when:

```text
cluster count is moderate
network connectivity is reliable
central control is required
```

Distributed is attractive when:

```text
clusters must survive management-plane isolation
regions are independent
security boundaries are strong
```

Hybrid designs often provide the best balance.

---

# PART IV — FLEET MANAGEMENT

## 15. Cluster Inventory

Maintain metadata such as:

```yaml
cluster: prod-eu-01
region: eu-west
environment: production
tier: critical
team: payments
```

This metadata can drive:

```text
ApplicationSets
policies
monitoring
deployment waves
```

---

## 16. Cluster Labels

Use consistent labels:

```text
environment=production
region=us-east
compliance=regulated
tier=critical
```

Avoid uncontrolled label naming across teams.

---

## 17. Cluster Registry

The platform should know:

```text
cluster name
region
account
environment
Kubernetes version
node strategy
GitOps endpoint
observability endpoint
owner
lifecycle status
```

---

# PART V — BOOTSTRAPPING

## 18. Cluster Bootstrap

A repeatable flow:

```text
Terraform
 |
VPC
 |
EKS
 |
IAM
 |
node capacity
 |
base addons
 |
GitOps
 |
platform addons
 |
policies
 |
observability
 |
applications
```

The bootstrap process should be reproducible from source control.

---

## 19. Cluster Bootstrap Repository

Example:

```text
clusters/
 |
+-- prod-us-east-1/
|    +-- addons/
|    +-- policies/
|    +-- apps/
|
+-- prod-eu-west-1/
|    +-- addons/
|    +-- policies/
|    +-- apps/
```

Avoid manually configuring production clusters differently.

---

## 20. Cluster Drift

Drift occurs when:

```text
desired state != actual state
```

Examples:

```text
manual RBAC change
manual Deployment change
addon version changed manually
node configuration changed
```

GitOps should detect and, where appropriate, correct drift.

---

# PART VI — APPLICATION PLACEMENT

## 21. Placement Criteria

Decide where an application runs based on:

```text
latency
data residency
capacity
security
dependencies
availability
cost
```

---

## 22. Stateless Applications

Stateless workloads are easiest to move:

```text
Cluster A
 |
failure
 |
Cluster B
 |
same image
same configuration
```

provided dependencies are available.

---

## 23. Stateful Applications

Stateful workloads require:

```text
data replication
storage strategy
consistency model
backup
restore
failover
```

Do not assume moving pods between clusters moves their data.

---

# PART VII — TRAFFIC MANAGEMENT

## 24. Global Traffic

Typical architecture:

```text
User
 |
Global DNS / traffic manager
 |
+---------------+---------------+
|                               |
Region A                       Region B
|                               |
Cluster A                       Cluster B
```

---

## 25. DNS Failover

DNS-based failover can direct users away from unhealthy endpoints.

Consider:

```text
TTL
health checks
cache behavior
application warm-up
```

DNS failover is not instantaneous because clients and resolvers may cache
records.

---

## 26. Weighted Routing

Example:

```text
Cluster A -> 90%
Cluster B -> 10%
```

Useful for:

```text
canary
migration
capacity testing
regional rollout
```

---

## 27. Latency Routing

Route users toward a region based on latency where supported and
appropriate.

Do not assume lowest network latency always means lowest application
latency.

---

## 28. Geographic Routing

Use geographic rules when requirements include:

```text
data residency
regional users
legal boundaries
business routing
```

---

# PART VIII — CROSS-CLUSTER NETWORKING

## 29. Do Clusters Need Direct Networking?

Not always.

Prefer:

```text
publicly/internal reachable API
or
application gateway
```

rather than making every pod in every cluster directly routable.

---

## 30. Direct Pod-to-Pod Networking

Possible designs can connect cluster networks, but they increase:

```text
routing complexity
CIDR requirements
security complexity
failure coupling
```

Use only when application requirements justify it.

---

## 31. Service-to-Service Across Clusters

Example:

```text
Cluster A
service-a
 |
gateway
 |
Cluster B
service-b
```

Prefer explicit service boundaries.

---

## 32. Multi-Cluster Service Discovery

Options include:

```text
global DNS
service mesh
gateway
API gateway
private load balancer
```

Choose the simplest solution that meets requirements.

---

# PART IX — AWS NETWORKING

## 33. Multi-Cluster VPC Strategy

Possible:

```text
VPC A -> Cluster A
VPC B -> Cluster B
VPC C -> Cluster C
```

Connect selectively using:

```text
Transit Gateway
PrivateLink
VPC peering
VPN
Direct Connect
```

---

## 34. Transit Gateway

A hub-and-spoke design:

```text
             Transit Gateway
             /      |      \
          VPC-A   VPC-B   VPC-C
```

Benefits:

```text
central routing
scalable connectivity
```

Risks:

```text
central dependency
route complexity
cost
```

---

## 35. PrivateLink

Use when a service should be consumed without exposing the entire network.

Concept:

```text
Provider service
 |
PrivateLink
 |
Consumer VPC
```

This can reduce broad network connectivity.

---

# PART X — IDENTITY

## 36. Human Identity

Centralize human authentication:

```text
SSO / identity provider
 |
AWS access
 |
EKS authentication
 |
RBAC
```

Avoid local permanent credentials.

---

## 37. Workload Identity Across Clusters

Each cluster should have explicit workload identity configuration.

Example:

```text
Cluster A
 |
ServiceAccount
 |
IAM Role A

Cluster B
 |
ServiceAccount
 |
IAM Role B
```

Avoid unintentionally sharing powerful roles between clusters.

---

# PART XI — SECRETS

## 38. Multi-Cluster Secrets

Central secrets can create:

```text
central dependency
large blast radius
```

Cluster-local secret access can provide stronger isolation.

---

## 39. Secret Replication

If secrets must exist in several regions:

```text
source secret
 |
controlled replication
 |
regional secret stores
 |
cluster integration
```

Ensure rotation propagates correctly.

---

# PART XII — SECURITY

## 40. Cluster Security Boundary

A cluster can be a useful isolation boundary, but security must also use:

```text
AWS account
IAM
VPC
RBAC
NetworkPolicy
admission
Pod Security
```

---

## 41. Cluster Compromise

If Cluster A is compromised:

```text
Cluster B
 |
should remain isolated
```

This requires avoiding:

```text
shared admin credentials
shared unrestricted IAM roles
shared secrets
unrestricted network access
```

---

## 42. Blast Radius

Reduce blast radius with:

```text
account boundaries
cluster boundaries
GitOps projects
deployment waves
regional boundaries
identity boundaries
```

---

# PART XIII — RBAC

## 43. Fleet RBAC

Do not automatically give a user admin access to every cluster.

Use:

```text
developer -> namespace in dev
developer -> namespace in stage
developer -> limited production access
platform -> cluster admin
SRE -> controlled production permissions
```

---

# PART XIV — POLICY

## 44. Fleet-Wide Policy

Examples:

```text
approved registry
non-root
resource requests
required labels
no privileged containers
```

Policies should be centrally standardized but safely rolled out.

---

## 45. Policy Rollout

Use:

```text
audit
 |
observe violations
 |
remediate workloads
 |
enforce
```

Do not suddenly enforce a restrictive policy across every cluster without
checking existing workloads.

---

# PART XV — OBSERVABILITY

## 46. Central Observability

```text
Cluster A --\
Cluster B ----> Metrics / Logs / Traces
Cluster C --/
```

Advantages:

```text
fleet visibility
central search
common dashboards
```

---

## 47. Regional Observability

For high-resilience environments:

```text
Cluster
 |
regional telemetry
 |
central aggregation
```

This reduces dependency on a single central endpoint.

---

## 48. Fleet Dashboard

Show:

```text
cluster health
Kubernetes version
node health
pending pods
API errors
CNI health
DNS
deployment status
security policy status
capacity
cost
```

---

# PART XVI — ALERTING

## 49. Cluster-Level Alerts

```text
cluster unavailable
node capacity low
pending pods
CNI errors
DNS failures
storage failures
GitOps sync failures
certificate expiry
```

---

## 50. Fleet-Level Alerts

Examples:

```text
multiple clusters degraded
same addon failing fleet-wide
policy rollout causing errors
central observability unavailable
global traffic failover
```

Fleet alerts should avoid generating hundreds of duplicate notifications.

---

# PART XVII — GITOPS PROMOTION

## 51. Environment Promotion

```text
dev
 |
stage
 |
prod-canary
 |
prod
```

Promotion should move the same artifact or immutable version.

---

## 52. Cluster Waves

Example:

```text
Wave 1 -> dev
Wave 2 -> stage
Wave 3 -> prod cluster A
Wave 4 -> prod cluster B
Wave 5 -> prod cluster C
```

Observe between waves.

---

## 53. Fleet-Wide Bad Commit

If a bad commit reaches many clusters:

```text
stop promotion
 |
identify revision
 |
revert Git
 |
sync affected clusters
 |
verify
```

The ability to stop the wave is a critical safety control.

---

# PART XVIII — DEPLOYMENT STRATEGIES

## 54. Cluster Canary

```text
Fleet
 |
Cluster A -> new version
 |
observe
 |
Cluster B/C -> new version
```

Useful for:

```text
platform changes
base images
operators
networking
```

---

## 55. Application Canary Across Clusters

```text
Cluster A
 |
10% traffic
 |
observe
 |
expand
```

---

# PART XIX — UPGRADES

## 56. Fleet Upgrade

Never upgrade every production cluster simultaneously by default.

Use:

```text
test
 |
dev
 |
stage
 |
production canary
 |
production wave
```

---

## 57. Version Skew

A fleet may temporarily contain:

```text
Cluster A -> Kubernetes N
Cluster B -> Kubernetes N
Cluster C -> Kubernetes N-1
```

Maintain a supported compatibility window.

---

## 58. Addon Fleet Upgrade

Upgrade in dependency order where applicable:

```text
cluster foundation
 |
CNI
 |
CoreDNS
 |
CSI
 |
Ingress
 |
observability
 |
GitOps
```

Always verify current compatibility documentation for exact versions.

---

# PART XX — DISASTER RECOVERY

## 59. Cluster Rebuild

A rebuild should be:

```text
Terraform
 |
VPC
 |
EKS
 |
addons
 |
GitOps
 |
applications
 |
secrets
 |
data
```

The cluster should not depend on undocumented manual steps.

---

## 60. Regional Disaster

Example:

```text
Region A
 |
failure
 |
Global traffic
 |
Region B
 |
EKS
 |
applications
```

But this requires:

```text
data
identity
secrets
images
DNS
observability
```

to be available in Region B.

---

## 61. RTO/RPO

RTO:

```text
maximum acceptable restoration time
```

RPO:

```text
maximum acceptable data loss
```

The number of clusters should be derived partly from these requirements.

---

# PART XXI — DATA ARCHITECTURE

## 62. Shared Database

```text
Cluster A --\
             Database
Cluster B --/
```

Benefits:

```text
central data
```

Risks:

```text
shared failure
cross-region latency
connection storms
```

---

## 63. Regional Database

```text
Region A -> DB A
Region B -> DB B
```

Requires:

```text
replication
conflict strategy
failover
consistency model
```

---

## 64. Database Failover

Application failover is incomplete if:

```text
application moves
but
database does not
```

Design application and data recovery together.

---

# PART XXII — REGISTRY

## 65. Multi-Cluster Image Availability

Ensure every target cluster can retrieve the artifact.

Options include:

```text
regional registries
replication
caching
cross-region access
```

---

## 66. Registry Failure

If the registry is unavailable during a deployment:

```text
existing pods -> may continue
new pods -> may fail image pull
```

Therefore critical images should have appropriate retention and regional
availability.

---

# PART XXIII — COST

## 67. Multi-Cluster Cost

Costs multiply through:

```text
cluster overhead
nodes
load balancers
NAT
observability
storage
data transfer
addons
```

A cluster with 5% utilization may still have significant fixed and
platform costs.

---

## 68. Cost Allocation

Tag and attribute:

```text
account
cluster
namespace
team
application
environment
```

Use cost visibility to identify idle or overprovisioned environments.

---

# PART XXIV — PLATFORM OPERATIONS

## 69. Standardization

Standardize:

```text
cluster module
node strategy
networking
addons
GitOps
observability
security
policies
```

Allow controlled exceptions.

---

## 70. Cluster Lifecycle

Define states:

```text
planned
provisioning
active
maintenance
decommissioning
decommissioned
```

Do not leave forgotten clusters running.

---

# PART XXV — INCIDENT MANAGEMENT

## 71. Cluster Incident

During a cluster outage:

```text
detect
 |
classify
 |
protect users
 |
fail over if designed
 |
stabilize
 |
recover
 |
validate
 |
root cause
```

---

## 72. Fleet Incident

If all clusters experience the same issue:

```text
suspect shared dependency
```

Examples:

```text
bad Git commit
bad policy
shared IAM change
shared DNS
shared registry
shared observability
```

Do not debug each cluster independently before checking the common cause.

---

# PART XXVI — FAILURE SCENARIOS

## 73. Cluster A Lost

```text
Cluster A
 |
lost
 |
traffic removed
 |
Cluster B
 |
serves workload
```

Requirements:

```text
capacity
data
secrets
images
routing
```

---

## 74. GitOps Controller Lost

If central Argo fails:

```text
existing workloads
 |
continue running
```

but:

```text
new reconciliation
 |
may stop
```

Design management-plane failure so it does not immediately become a
workload outage.

---

## 75. Network Partition

```text
Cluster A X Cluster B
```

Applications should have:

```text
timeouts
retries
fallback
circuit breaking
```

Do not design critical synchronous paths that require fragile
cross-cluster communication unless necessary.

---

# PART XXVII — CELL ARCHITECTURE

## 76. Cell Isolation

```text
Global Control
 |
+-- Cell A
|    +-- cluster
|    +-- data
|
+-- Cell B
|    +-- cluster
|    +-- data
|
+-- Cell C
     +-- cluster
     +-- data
```

Each cell can have:

```text
network
observability
deployment boundary
capacity
```

This strongly limits blast radius.

---

# PART XXVIII — PLATFORM ENGINEERING

## 77. Fleet Self-Service

Developer experience:

```text
portal
 |
choose environment
 |
choose region
 |
choose service template
 |
Git repository
 |
CI
 |
artifact
 |
GitOps
 |
selected clusters
```

The developer should not manually configure every target cluster.

---

# PART XXIX — SECURITY SCENARIOS

## 78. Credential Leakage

If a production credential leaks:

```text
identify scope
 |
revoke
 |
rotate
 |
audit
 |
redeploy
```

Cluster-specific identities reduce blast radius.

---

## 79. Malicious Deployment

Controls:

```text
PR review
CI scanning
artifact signing
GitOps approval
admission policy
runtime monitoring
```

---

# PART XXX — DESIGN SCENARIOS

## 80. Scenario: Two Production Regions

Requirements:

```text
high availability
regional latency
```

Design:

```text
Region A -> EKS A
Region B -> EKS B
 |
global traffic
```

Data architecture must be explicitly designed.

---

## 81. Scenario: Five Production Clusters

Use:

```text
centralized fleet metadata
GitOps
ApplicationSets
deployment waves
common observability
standard addons
cluster-specific overlays
```

Avoid five completely different platform implementations.

---

## 82. Scenario: Regulated Workloads

Use:

```text
dedicated account
dedicated cluster
restricted network
separate IAM
restricted GitOps project
audit logs
data-residency controls
```

Exact controls depend on regulatory requirements.

---

## 83. Scenario: 100 Clusters

At this scale, automation becomes mandatory.

Need:

```text
cluster provisioning automation
fleet inventory
GitOps automation
standard addons
policy automation
central observability
upgrade waves
capacity management
lifecycle automation
```

Manual operations do not scale.

---

## 84. Scenario: 1,000 Clusters

Consider:

```text
cell-based fleet
regional management
hierarchical GitOps
automated lifecycle
standard platform APIs
central policy with local enforcement
fleet health automation
```

Avoid a single central dependency for every cluster operation.

---

# PART XXXI — INTERVIEW DESIGN METHOD

## 85. Step 1 — Clarify

Ask:

```text
How many clusters?
Why multiple clusters?
Same or different regions?
Same or different accounts?
Active-active or active-passive?
What RTO/RPO?
What data model?
What security boundary?
How many applications?
```

---

## 86. Step 2 — Draw

Start:

```text
Users
 |
Traffic Layer
 |
Regions
 |
Clusters
 |
Namespaces
 |
Applications
 |
Data
```

Then add:

```text
GitOps
identity
observability
security
```

---

## 87. Step 3 — Failure Analysis

Explicitly explain:

```text
node failure
AZ failure
cluster failure
region failure
GitOps failure
registry failure
DNS failure
identity failure
database failure
```

---

## 88. Senior-Level Answer

A strong answer says:

```text
I would not choose the number of clusters first. I would derive the
cluster topology from failure isolation, compliance, scale, regional
requirements and upgrade independence. I would then standardize the fleet
through IaC and GitOps while keeping cluster-specific configuration
minimal and explicit.
```

---

# PART XXXII — 150 PRODUCTION GOLDEN RULES

## 89. Rules 1–30

```text
1. Do not create clusters without a business or operational reason.
2. Treat every cluster as an operational product.
3. Define cluster ownership.
4. Define cluster lifecycle.
5. Define failure domains.
6. Define RTO.
7. Define RPO.
8. Define security boundaries.
9. Prefer standardized cluster foundations.
10. Automate cluster provisioning.
11. Use infrastructure as code.
12. Make bootstrap repeatable.
13. Avoid undocumented manual configuration.
14. Keep cluster metadata.
15. Standardize labels.
16. Track Kubernetes versions.
17. Track addon versions.
18. Track cluster owners.
19. Track lifecycle status.
20. Design account boundaries deliberately.
21. Use multiple AZs for production clusters.
22. Plan VPC CIDRs.
23. Plan pod IP capacity.
24. Plan inter-cluster routing.
25. Avoid unnecessary direct pod networking.
26. Prefer explicit service gateways for cross-cluster traffic.
27. Keep network dependencies minimal.
28. Avoid unnecessary shared infrastructure.
29. Reduce blast radius.
30. Design for partial failure.
```

## 90. Rules 31–60

```text
31. Choose centralized GitOps deliberately.
32. Choose distributed GitOps deliberately.
33. Consider hybrid GitOps for large fleets.
34. Do not make one management cluster a hidden single point of failure.
35. Protect GitOps credentials.
36. Use least-privilege cluster access.
37. Separate cluster identities.
38. Separate workload identities.
39. Avoid shared powerful IAM roles.
40. Avoid shared permanent credentials.
41. Use SSO for human access.
42. Use RBAC.
43. Use namespace isolation.
44. Use NetworkPolicy.
45. Use admission policy.
46. Roll out policies gradually.
47. Monitor policy violations.
48. Use approved registries.
49. Use immutable artifacts.
50. Prefer image digests.
51. Scan images.
52. Generate SBOM where required.
53. Verify signatures where required.
54. Keep secrets out of Git.
55. Design secret replication.
56. Rotate secrets.
57. Protect break-glass access.
58. Audit privileged actions.
59. Keep clusters independently recoverable.
60. Test recovery.
```

## 91. Rules 61–90

```text
61. Use deployment waves.
62. Canary platform changes.
63. Stop bad waves quickly.
64. Revert bad Git changes.
65. Do not blindly roll back data migrations.
66. Promote immutable versions.
67. Keep environment overlays controlled.
68. Minimize cluster-specific differences.
69. Use cluster labels for placement.
70. Use ApplicationSets where appropriate.
71. Keep fleet configuration discoverable.
72. Standardize observability.
73. Collect cluster health.
74. Collect application health.
75. Centralize logs where practical.
76. Keep regional telemetry paths for resilience.
77. Alert on fleet-level patterns.
78. Avoid duplicate alert storms.
79. Track capacity per cluster.
80. Track capacity across the fleet.
81. Track IP utilization.
82. Track storage.
83. Track cost.
84. Track deployment health.
85. Track version drift.
86. Track addon drift.
87. Track security policy drift.
88. Track configuration drift.
89. Automate drift detection.
90. Reconcile desired state.
```

## 92. Rules 91–120

```text
91. Upgrade clusters in waves.
92. Upgrade non-production first.
93. Canary production upgrades.
94. Maintain supported version skew.
95. Validate addon compatibility.
96. Test CNI upgrades.
97. Test CoreDNS upgrades.
98. Test CSI upgrades.
99. Test ingress upgrades.
100. Test policy changes.
101. Test GitOps changes.
102. Never assume a second cluster equals DR.
103. Replicate required data.
104. Replicate required secrets.
105. Ensure image availability.
106. Ensure DNS recovery.
107. Ensure identity recovery.
108. Ensure observability recovery.
109. Test regional failover.
110. Test cluster failover.
111. Test cluster rebuild.
112. Measure actual RTO.
113. Measure actual RPO.
114. Document external dependencies.
115. Protect shared dependencies.
116. Avoid shared dependencies when isolation requires independence.
117. Use cell architecture for very large fleets.
118. Use regional control where appropriate.
119. Avoid centralizing every failure mode.
120. Keep recovery paths simpler than normal paths.
```

## 93. Rules 121–150

```text
121. Do not use direct cross-cluster networking without a requirement.
122. Use explicit APIs for cross-cluster dependencies.
123. Apply timeouts.
124. Apply bounded retries.
125. Prevent retry storms.
126. Design for network partitions.
127. Design for DNS failure.
128. Design for registry failure.
129. Design for GitOps failure.
130. Design for IAM failure.
131. Design for observability failure.
132. Keep existing workloads running when management systems fail.
133. Separate management-plane and data-plane failure.
134. Avoid fleet-wide risky changes.
135. Use progressive rollout.
136. Use standardized golden paths.
137. Automate team onboarding.
138. Automate cluster onboarding.
139. Automate decommissioning.
140. Remove unused clusters.
141. Control idle capacity.
142. Use cost allocation.
143. Optimize observability retention.
144. Avoid unnecessary load balancers.
145. Avoid unnecessary NAT paths.
146. Use appropriate regional architecture.
147. Explain trade-offs, not only tools.
148. Test every claimed resilience property.
149. Design for failure before optimizing for convenience.
150. A multi-cluster platform is successful when clusters remain isolated
     enough to contain failures while still being standardized enough to
     operate as one platform.
```

# END OF 07-Multi-Cluster-Architecture.md
