# Disaster-Recovery-Design

## 1. Purpose

This is a production-grade Disaster Recovery (DR) design guide for AWS,
Kubernetes, EKS, DevOps platforms, applications, databases, networking,
security, CI/CD, GitOps and observability.

Disaster Recovery is not simply:

```text
take backups
```

A real DR architecture answers:

```text
What constitutes a disaster?
What must be recovered?
What is the RTO?
What is the RPO?
Where is the recovery environment?
How is data recovered?
How is traffic redirected?
How are identities recovered?
How are secrets recovered?
How are artifacts recovered?
How is Kubernetes rebuilt?
How are dependencies recovered?
Who declares disaster?
Who controls failover?
How is recovery validated?
How is failback performed?
How is the DR plan tested?
```

The central principle is:

```text
A DR plan is only credible when recovery has been repeatedly tested.
```

---

# PART I — DR FOUNDATIONS

## 2. What Is Disaster Recovery?

Disaster Recovery is the set of architecture, processes, automation and
operational procedures used to restore critical services after a major
failure.

Possible disasters include:

```text
region outage
account compromise
cluster destruction
data corruption
ransomware
accidental deletion
major networking failure
control-plane failure
deployment catastrophe
dependency outage
human error
```

---

## 3. DR vs High Availability

HA generally addresses expected component failures:

```text
pod
node
AZ
service instance
```

DR addresses larger recovery events:

```text
cluster
account
region
data
organization
```

The exact boundary is business-specific.

---

## 4. DR vs Backup

Backup:

```text
copy data
```

DR:

```text
restore complete service
```

A backup can exist while the application remains unrecoverable.

---

## 5. DR vs Business Continuity

Business continuity is broader:

```text
people
process
technology
facilities
suppliers
communication
```

DR is the technology recovery component of the overall continuity strategy.

---

# PART II — BUSINESS REQUIREMENTS

## 6. Business Impact Analysis

Identify:

```text
critical services
dependencies
maximum tolerable outage
data sensitivity
financial impact
customer impact
regulatory impact
```

---

## 7. Criticality Classes

Example:

```text
Tier 0 -> mission critical
Tier 1 -> critical
Tier 2 -> important
Tier 3 -> non-critical
```

Each tier can have different:

```text
RTO
RPO
recovery priority
backup frequency
DR cost
```

---

## 8. RTO

Recovery Time Objective:

```text
maximum acceptable recovery time
```

Example:

```text
RTO = 15 minutes
```

The design must realistically recover the service within that window.

---

## 9. RPO

Recovery Point Objective:

```text
maximum acceptable data loss measured in time
```

Example:

```text
RPO = 5 minutes
```

This requires a recovery point no older than the agreed boundary.

---

## 10. MTD

Maximum Tolerable Downtime defines the business limit beyond which the
service outage becomes unacceptable.

RTO should normally be smaller than the business's maximum tolerable
downtime.

---

# PART III — RECOVERY STRATEGIES

## 11. Backup and Restore

```text
Primary
 |
backup
 |
DR
 |
restore
 |
application
```

Lowest ongoing cost, usually slowest recovery.

---

## 12. Pilot Light

Keep minimal core components ready:

```text
data
network
identity
minimal infrastructure
```

Application capacity is created during recovery.

---

## 13. Warm Standby

Maintain a reduced production environment:

```text
Primary -> 100%
DR      -> 20%
```

During disaster:

```text
scale DR
 |
deploy/activate
 |
traffic
```

---

## 14. Hot Standby

Maintain substantial capacity:

```text
Primary -> 100%
DR      -> ready
```

Fast recovery but higher cost.

---

## 15. Active-Active

Both environments serve production:

```text
Users
 |
Global Traffic
 |
+---------+---------+
|                   |
Region A           Region B
```

This can provide rapid failover but creates significant data and operational
complexity.

---

# PART IV — RECOVERY STRATEGY SELECTION

## 16. RTO-Driven Selection

Rough model:

```text
hours
 -> backup/restore may work

tens of minutes
 -> warm standby

minutes/seconds
 -> hot standby or active-active
```

Actual feasibility must be validated through testing.

---

## 17. Cost Trade-Off

Generally:

```text
backup/restore
      |
pilot light
      |
warm standby
      |
hot standby
      |
active-active
```

Cost tends to increase as recovery speed and operational readiness increase.

---

# PART V — DISASTER DECLARATION

## 18. Who Declares Disaster?

Define explicit authority:

```text
incident commander
business owner
technical lead
security lead
```

Avoid ambiguous decision-making during an outage.

---

## 19. Disaster Criteria

Possible triggers:

```text
region unavailable
critical data corruption
security compromise
recovery environment required
RTO at risk
```

---

## 20. False Failover

Automatic failover can be dangerous if the primary is merely degraded.

Possible outcome:

```text
Primary active
Secondary active
 |
split brain
```

Define safe failover conditions.

---

# PART VI — DR ARCHITECTURE

## 21. Reference

```text
                         Users
                           |
                     Global Traffic
                           |
              +------------+------------+
              |                         |
           Primary                     DR
           Region                    Region
              |                         |
             EKS                       EKS
              |                         |
           Services                  Services
              |                         |
            Data <----------------> Data
              |
          Backups
              |
        Recovery Storage
```

---

# PART VII — AWS MULTI-REGION DR

## 22. Regional Recovery

Primary:

```text
Region A
```

Recovery:

```text
Region B
```

Keep the recovery region sufficiently independent.

---

## 23. Region Selection

Evaluate:

```text
AWS service availability
latency
data residency
cost
capacity
quotas
compliance
network connectivity
```

---

# PART VIII — INFRASTRUCTURE AS CODE

## 24. Rebuildable Infrastructure

Use:

```text
Terraform
CloudFormation
other declarative IaC
```

The recovery environment should not depend entirely on manual console
operations.

---

## 25. State Management

Protect IaC state:

```text
state storage
locking
versioning
backup
access control
```

A destroyed or corrupted state file can delay recovery.

---

# PART IX — AWS ACCOUNT DR

## 26. Account-Level Disaster

A production account can become unavailable because of:

```text
security compromise
misconfiguration
billing issue
identity failure
resource destruction
```

Consider separate recovery accounts and organizational controls.

---

## 27. Recovery Account

Possible structure:

```text
Management
 |
+-- Security
+-- Log Archive
+-- Production
+-- DR/Recovery
```

Keep recovery permissions tightly controlled.

---

# PART X — EKS DR

## 28. Rebuild EKS

Possible flow:

```text
Terraform
 |
VPC
 |
EKS
 |
node capacity
 |
addons
 |
IAM
 |
GitOps
 |
applications
```

---

## 29. EKS Configuration

Recover:

```text
cluster configuration
node groups
security groups
IAM roles
OIDC/workload identity
addons
ingress
storage
policies
```

---

## 30. Cluster Addons

Typical:

```text
CoreDNS
VPC CNI
kube-proxy
metrics
ingress
external-dns
cert-manager
CSI drivers
observability agents
```

The exact set depends on the platform.

---

# PART XI — GITOPS RECOVERY

## 31. GitOps as Recovery Mechanism

If application configuration is stored in Git:

```text
new cluster
 |
Argo CD
 |
Git
 |
desired state
 |
applications
```

This significantly reduces manual recovery work.

---

## 32. Git Repository DR

Protect:

```text
source code
manifests
Helm charts
Terraform
policies
pipelines
documentation
```

A GitOps strategy fails as a recovery mechanism if its source of truth is
not itself recoverable.

---

# PART XII — DATABASE DR

## 33. Database Recovery

Options:

```text
backup restore
replication
cross-region replica
managed global replication
```

Choose according to:

```text
RTO
RPO
consistency
cost
```

---

## 34. Database Backup

Protect:

```text
automated backups
manual snapshots
point-in-time recovery
cross-region copies
```

---

## 35. Point-in-Time Recovery

Useful for:

```text
accidental deletion
bad deployment
corruption
operator error
```

---

## 36. Logical vs Physical Recovery

Logical:

```text
dump/export
```

Physical:

```text
snapshot/storage recovery
```

Trade-offs include recovery speed and portability.

---

# PART XIII — DATABASE CORRUPTION

## 37. Corruption Is Different From Outage

If corrupted data is replicated immediately:

```text
Primary
 |
corruption
 |
replica
```

the replica may also contain corrupted data.

Therefore backups and recovery points are essential.

---

# PART XIV — S3 DR

## 38. Object Recovery

Use:

```text
versioning
replication
backup
retention
```

where business requirements justify them.

---

## 39. Delete Protection

Versioning can provide recovery options after accidental deletion.

Validate delete-marker and replication behavior.

---

# PART XV — EBS DR

## 40. Volume Recovery

```text
snapshot
 |
copy/recovery
 |
new region
 |
new volume
 |
application
```

Do not assume EBS volumes automatically exist in another region.

---

# PART XVI — CONTAINER REGISTRY DR

## 41. Images

Critical production images must remain recoverable.

Use:

```text
replication
retention
immutable tags/digests
regional availability
```

---

## 42. Image Digest

Prefer recovery using:

```text
image@sha256:...
```

rather than mutable tags.

---

# PART XVII — SECRETS DR

## 43. Secret Recovery

Recover:

```text
application secrets
database credentials
certificates
API keys
encryption metadata
```

Never store raw production secrets inside normal Git repositories.

---

## 44. Secret Rotation

DR must account for:

```text
current secret
previous secret
rotation schedule
regional copies
application reload
```

---

# PART XVIII — KMS DR

## 45. Encryption Keys

Recovery requires access to the keys protecting:

```text
database
S3
backups
secrets
volumes
```

A backup is useless if its encryption key cannot be recovered.

---

# PART XIX — IAM DR

## 46. Identity Recovery

Recover:

```text
roles
policies
trust relationships
workload identities
automation identities
break-glass access
```

---

## 47. Break-Glass Access

Maintain tightly controlled emergency access:

```text
MFA
auditing
limited permissions
dual control where appropriate
```

Test it.

---

# PART XX — NETWORK DR

## 48. VPC Recovery

Recover:

```text
VPC
subnets
routes
security groups
NACLs
endpoints
NAT
load balancers
DNS
```

---

## 49. CIDR Planning

Pre-plan recovery CIDRs.

Avoid accidental overlap with:

```text
on-premises
other regions
shared services
```

---

# PART XXI — DNS DR

## 50. Traffic Failover

```text
Primary endpoint
 |
health check
 |
failure
 |
DR endpoint
```

Test real user behavior, not only DNS record changes.

---

## 51. TTL

Low TTL can improve steering responsiveness but does not guarantee
instantaneous failover because recursive resolvers and clients may cache
responses.

---

# PART XXII — APPLICATION RECOVERY

## 52. Recovery Order

Example:

```text
1. Network
2. Identity
3. Security
4. Data
5. Cluster
6. Platform addons
7. Application
8. Traffic
9. Validation
```

The exact sequence depends on architecture.

---

## 53. Dependency Graph

Model:

```text
Application
 |
API
 |
Database
 |
Secrets
 |
KMS
```

Recovery order should follow dependency relationships.

---

# PART XXIII — PRIORITY-BASED RECOVERY

## 54. Tiered Recovery

Example:

```text
Tier 0:
identity + network + database

Tier 1:
checkout + authentication

Tier 2:
search + recommendations

Tier 3:
analytics + batch
```

Recover critical functions first.

---

# PART XXIV — RECOVERY AUTOMATION

## 55. Automation

Automate:

```text
infrastructure
DNS
cluster
applications
secrets
validation
```

Keep manual approval for high-risk actions.

---

# PART XXV — RUNBOOK AUTOMATION

## 56. Recovery Pipeline

```text
Declare DR
 |
Provision
 |
Restore
 |
Deploy
 |
Validate
 |
Traffic
```

Every stage should produce evidence.

---

# PART XXVI — RECOVERY VALIDATION

## 57. Infrastructure Validation

Check:

```text
VPC
routes
security groups
IAM
nodes
```

---

## 58. Application Validation

Check:

```text
health
latency
errors
dependencies
business transactions
```

---

## 59. Data Validation

Check:

```text
replication point
row/object counts
consistency
business invariants
```

Do not rely only on "database is online."

---

# PART XXVII — SYNTHETIC TRANSACTIONS

## 60. Business Validation

Run synthetic:

```text
login
browse
checkout
payment test
order lookup
```

where safe and appropriate.

---

# PART XXVIII — FAILOVER

## 61. Failover Process

```text
1. Detect incident.
2. Confirm scope.
3. Declare DR.
4. Freeze risky changes.
5. Protect data.
6. Activate recovery.
7. Validate data.
8. Validate applications.
9. Shift traffic.
10. Monitor.
```

---

# PART XXIX — FAILBACK

## 62. Failback Is a Separate Project

```text
DR active
 |
repair primary
 |
synchronize data
 |
validate
 |
controlled traffic shift
 |
primary active
```

---

## 63. Do Not Rush Failback

Immediate failback can cause:

```text
second outage
data divergence
rollback complexity
```

---

# PART XXX — DATA DIVERGENCE

## 64. During DR

If DR accepts writes:

```text
Primary data
     X
DR data
```

When primary returns, reconcile:

```text
conflicts
missing writes
duplicate writes
```

Design this before the disaster.

---

# PART XXXI — ACTIVE-ACTIVE DR

## 65. Active-Active

Advantages:

```text
fast recovery
normal resource utilization
regional latency
```

Challenges:

```text
data conflicts
global consistency
deployment coordination
traffic management
cost
```

---

# PART XXXII — ACTIVE-PASSIVE DR

## 66. Active-Passive

Advantages:

```text
simpler
lower cost
clear ownership
```

Challenges:

```text
unused capacity
failover delay
stale data
```

---

# PART XXXIII — PILOT LIGHT

## 67. Pilot Light

Keep:

```text
core data
network
security foundation
```

ready.

Provision application capacity during recovery.

---

# PART XXXIV — WARM STANDBY

## 68. Warm Standby

Maintain:

```text
network
cluster
application
reduced capacity
```

Then:

```text
scale
validate
shift
```

---

# PART XXXV — COLD DR

## 69. Cold DR

Everything is recreated:

```text
IaC
 |
network
 |
cluster
 |
data
 |
application
```

This requires highly reliable automation.

---

# PART XXXVI — SECURITY DISASTER

## 70. Compromised Region

Do not automatically restore compromised artifacts.

Consider:

```text
clean environment
known-good images
known-good infrastructure code
credential rotation
forensic evidence
```

---

## 71. Ransomware

Recovery requires:

```text
isolated backups
immutable recovery points
clean credentials
clean infrastructure
validation
```

---

# PART XXXVII — SUPPLY CHAIN DR

## 72. Compromised Image

Do not blindly restore:

```text
latest image
```

Use:

```text
verified digest
known-good artifact
SBOM
security scan
provenance
```

---

# PART XXXVIII — BACKUP SECURITY

## 73. Backup Isolation

Backups should have:

```text
restricted access
separate permissions
retention protection
encryption
monitoring
```

---

# PART XXXIX — IMMUTABLE BACKUPS

## 74. Immutability

Immutable recovery points reduce risk of:

```text
accidental deletion
malicious deletion
ransomware
```

---

# PART XL — DR TESTING

## 75. DR Test

A real test should:

```text
restore
deploy
route
validate
measure
```

not simply confirm:

```text
backup job succeeded
```

---

# PART XLI — TABLETOP EXERCISE

## 76. Tabletop

Simulate:

```text
region outage
```

without necessarily causing production impact.

Walk through:

```text
roles
decisions
runbooks
communications
```

---

# PART XLII — GAME DAY

## 77. Technical Game Day

Inject failure into a controlled environment.

Measure:

```text
MTTD
MTTR
RTO
RPO
manual effort
```

---

# PART XLIII — RECOVERY METRICS

## 78. Important Metrics

Track:

```text
RTO achieved
RPO achieved
restore duration
data lag
traffic convergence
application startup
manual steps
failure rate
```

---

# PART XLIV — DR COMMUNICATION

## 79. Incident Communication

Define:

```text
technical channel
executive channel
customer communication
status updates
decision log
```

---

# PART XLV — CHANGE MANAGEMENT

## 80. DR Changes

Recovery systems must be updated when production changes.

If production has:

```text
new database
new secret
new service
```

DR must reflect it.

---

# PART XLVI — DR DRIFT

## 81. Configuration Drift

Bad:

```text
Production
new configuration

DR
old configuration
```

At recovery:

```text
DR fails
```

Use automated drift detection.

---

# PART XLVII — DR DOCUMENTATION

## 82. Required Documentation

Maintain:

```text
architecture
dependencies
RTO/RPO
recovery steps
credentials process
contacts
escalation
validation
failback
```

---

# PART XLVIII — DR DEPENDENCY MAP

## 83. Dependency Inventory

For each application:

```text
compute
database
cache
queue
DNS
secrets
KMS
registry
external APIs
```

Assign recovery priority.

---

# PART XLIX — THIRD-PARTY DEPENDENCIES

## 84. External Services

A DR region does not solve:

```text
payment provider outage
email provider outage
identity provider outage
```

Classify third-party dependencies.

---

# PART L — DEGRADED DR

## 85. Partial Service

During disaster:

```text
checkout -> available
search -> degraded
recommendations -> disabled
analytics -> delayed
```

This can produce better business continuity than attempting to restore
everything simultaneously.

---

# PART LI — DR CAPACITY

## 86. Capacity Calculation

If production peak is:

```text
10,000 RPS
```

and DR must support full failover:

```text
DR capacity >= required 10,000 RPS
```

unless the business explicitly accepts reduced service.

---

# PART LII — DR AUTOSCALING

## 87. Autoscaling

Do not rely exclusively on reactive autoscaling for a very strict RTO.

Use:

```text
warm capacity
pre-scaling
reserved capacity
rapid provisioning
```

as appropriate.

---

# PART LIII — DR NETWORK CAPACITY

## 88. Network

Validate:

```text
bandwidth
IP capacity
NAT
load balancer
service endpoints
DNS
```

A DR cluster with insufficient IP space is not a usable DR environment.

---

# PART LIV — DR OBSERVABILITY

## 89. Monitoring Recovery

DR needs:

```text
metrics
logs
traces
alerts
dashboards
```

before traffic is shifted.

---

# PART LV — DR OBSERVABILITY FAILURE

## 90. Do Not Blindly Fail Over

If observability is unavailable:

```text
unknown application health
```

Use alternate validation mechanisms.

---

# PART LVI — DR COST

## 91. Cost Model

Include:

```text
standby compute
storage
replication
network transfer
backup
observability
licenses
testing
```

---

# PART LVII — DR GOVERNANCE

## 92. Ownership

Every recovery component should have an owner.

Example:

```text
network -> platform
database -> data team
application -> service team
security -> security team
```

---

# PART LVIII — DR COMPLIANCE

## 93. Evidence

Keep evidence of:

```text
backup
restore tests
DR tests
access reviews
RTO/RPO results
```

---

# PART LIX — DR AUDIT

## 94. Audit Questions

```text
Can you recover?
When was it last tested?
What was the measured RTO?
What was the measured RPO?
Who owns recovery?
Where are backups?
Are backups immutable?
Can encryption keys be recovered?
```

---

# PART LX — REAL-WORLD SCENARIOS

## 95. Region Failure

```text
Region A
 |
complete outage
 |
Global routing
 |
Region B
 |
application
 |
data
```

Validate every dependency.

---

## 96. Database Corruption

```text
bad deployment
 |
corrupted data
 |
replication
```

Do not simply fail over to the newest replica.

Use:

```text
known-good recovery point
```

---

## 97. Accidental Terraform Destroy

Recovery:

```text
Git
 |
Terraform
 |
network
 |
EKS
 |
GitOps
 |
application
```

Data requires separate restoration.

---

## 98. Account Compromise

Response:

```text
isolate
 |
revoke credentials
 |
preserve evidence
 |
clean recovery account
 |
known-good infrastructure
 |
restore data
```

---

## 99. Ransomware

Use:

```text
isolated backups
 |
clean environment
 |
restore
 |
validate
```

Never assume online backups are safe.

---

# PART LXI — SENIOR SYSTEM-DESIGN SCENARIOS

## 100. Design 15-Minute RTO

Use:

```text
warm/hot standby
replicated data
pre-created infrastructure
automated failover
traffic management
```

Measure each step.

---

## 101. Design 5-Minute RTO

Likely requires:

```text
hot standby
or active-active
```

with:

```text
rapid detection
pre-warmed capacity
fast traffic steering
ready data
```

---

## 102. Design 1-Hour RTO

Could use:

```text
pilot light
backup restore
IaC provisioning
```

depending on workload size and tested recovery duration.

---

## 103. Design Near-Zero RPO

Focus on:

```text
continuous replication
durable writes
replication health
failover correctness
```

---

## 104. Design Large EKS DR

Requirements:

```text
100 clusters
```

Need:

```text
cluster templates
fleet GitOps
regional capacity
artifact replication
secrets
observability
```

---

# PART LXII — INTERVIEW FRAMEWORK

## 105. Senior DR Answer

Use:

```text
1. Define business impact.
2. Define RTO.
3. Define RPO.
4. Classify workloads.
5. Select recovery strategy.
6. Design data recovery.
7. Design infrastructure recovery.
8. Design traffic failover.
9. Design identity/secrets/KMS.
10. Design validation.
11. Design failback.
12. Test the complete process.
13. Explain cost and trade-offs.
```

---

# PART LXIII — PRODUCTION RUNBOOK

## 106. Disaster Declaration

```text
1. Confirm major failure.
2. Determine blast radius.
3. Start incident command.
4. Freeze risky changes.
5. Protect data.
6. Declare DR if criteria are met.
```

---

## 107. Recovery

```text
1. Activate recovery environment.
2. Verify network.
3. Verify identity.
4. Verify encryption.
5. Restore/validate data.
6. Verify cluster.
7. Deploy platform addons.
8. Deploy critical applications.
9. Run synthetic tests.
10. Shift traffic.
11. Monitor.
```

---

## 108. Failback

```text
1. Repair primary.
2. Validate infrastructure.
3. Synchronize data.
4. Validate applications.
5. Test primary.
6. Shift traffic gradually.
7. Monitor.
8. Confirm stability.
9. Keep DR ready.
```

---

# PART LXIV — PRODUCTION CHECKLIST

## 109. Business

```text
[ ] service criticality defined
[ ] RTO defined
[ ] RPO defined
[ ] owners defined
[ ] communication plan
```

## 110. Infrastructure

```text
[ ] IaC
[ ] DR region/account
[ ] network
[ ] IAM
[ ] KMS
[ ] security
```

## 111. Data

```text
[ ] backups
[ ] replication
[ ] restore
[ ] corruption recovery
[ ] immutable recovery points
```

## 112. Kubernetes

```text
[ ] EKS
[ ] node capacity
[ ] addons
[ ] GitOps
[ ] ingress
[ ] storage
```

## 113. Application

```text
[ ] images
[ ] secrets
[ ] configuration
[ ] dependencies
[ ] synthetic tests
```

## 114. Operations

```text
[ ] monitoring
[ ] alerts
[ ] runbooks
[ ] game day
[ ] failover test
[ ] failback test
```

---

# PART LXV — 250 PRODUCTION GOLDEN RULES

## 115. Rules 1–50

```text
1. Start DR design with business impact.
2. Define RTO.
3. Define RPO.
4. Define MTD.
5. Classify workloads.
6. Assign recovery priority.
7. Identify every dependency.
8. Separate HA from DR.
9. Separate backup from DR.
10. Do not claim DR because backups exist.
11. Choose recovery strategy based on RTO.
12. Choose recovery strategy based on RPO.
13. Validate the strategy through testing.
14. Keep recovery automation versioned.
15. Keep recovery documentation current.
16. Define disaster declaration authority.
17. Define failover criteria.
18. Prevent false failover.
19. Prevent split brain.
20. Protect data before failover.
21. Use independent recovery capacity where required.
22. Avoid one shared failure domain.
23. Select recovery regions deliberately.
24. Validate AWS service availability.
25. Validate regional quotas.
26. Validate regional capacity.
27. Plan non-overlapping CIDRs.
28. Protect Terraform state.
29. Version infrastructure.
30. Rebuild infrastructure declaratively.
31. Do not depend on manual console reconstruction.
32. Separate application recovery from data recovery.
33. Test infrastructure recreation.
34. Test application recreation.
35. Test database recovery.
36. Test network recovery.
37. Test identity recovery.
38. Test secret recovery.
39. Test KMS recovery.
40. Test DNS recovery.
41. Test artifact recovery.
42. Test observability recovery.
43. Test GitOps recovery.
44. Test CI/CD dependencies.
45. Document recovery order.
46. Recover dependencies before consumers.
47. Recover critical services first.
48. Allow non-critical services to remain degraded.
49. Measure actual recovery duration.
50. Never assume the planned RTO is the achieved RTO.
```

## 116. Rules 51–100

```text
51. Use backups appropriate to RPO.
52. Test point-in-time recovery.
53. Test backup integrity.
54. Test restore regularly.
55. Keep recovery points protected.
56. Use immutable backups where justified.
57. Restrict backup access.
58. Encrypt backups.
59. Protect encryption keys.
60. Replicate critical recovery data.
61. Monitor replication lag.
62. Do not assume replicas eliminate corruption.
63. Maintain historical recovery points.
64. Define corruption recovery procedures.
65. Define accidental deletion recovery.
66. Define ransomware recovery.
67. Define operator-error recovery.
68. Keep known-good application artifacts.
69. Prefer immutable image digests.
70. Replicate critical images.
71. Protect package repositories.
72. Recover Helm charts.
73. Recover Terraform modules.
74. Recover Kubernetes manifests.
75. Protect Git repositories.
76. Protect GitOps credentials.
77. Keep GitOps source recoverable.
78. Use GitOps to reduce manual recovery.
79. Rebuild EKS declaratively.
80. Recover cluster addons.
81. Recover node capacity.
82. Recover workload identity.
83. Recover ingress.
84. Recover storage drivers.
85. Recover DNS integration.
86. Recover certificate automation.
87. Recover observability agents.
88. Verify cluster health before applications.
89. Verify networking before workloads.
90. Verify IAM before automation.
91. Verify KMS before encrypted restoration.
92. Verify secrets before application startup.
93. Verify databases before traffic.
94. Verify queues before consumers.
95. Verify caches before relying on them.
96. Treat cache as rebuildable when possible.
97. Validate external dependencies.
98. Document third-party DR limitations.
99. Define degraded-mode behavior.
100. Prioritize critical business transactions.
```

## 117. Rules 101–150

```text
101. Design global traffic failover.
102. Test DNS convergence.
103. Understand TTL effects.
104. Test health-check semantics.
105. Do not use shallow health checks for critical failover.
106. Validate user-visible traffic after failover.
107. Maintain sufficient DR capacity.
108. Include autoscaling delay in RTO.
109. Include image pull time in RTO.
110. Include application startup in RTO.
111. Include database promotion time in RTO.
112. Include DNS convergence in RTO.
113. Include human approval time in RTO.
114. Measure each recovery stage.
115. Automate repetitive recovery.
116. Require approval for destructive operations.
117. Protect break-glass access.
118. Test break-glass access.
119. Audit recovery credentials.
120. Rotate compromised credentials.
121. Isolate compromised environments.
122. Preserve forensic evidence.
123. Do not restore compromised artifacts blindly.
124. Use known-good infrastructure.
125. Validate supply-chain integrity.
126. Validate image provenance.
127. Validate SBOM/security status where applicable.
128. Protect recovery accounts.
129. Limit cross-account permissions.
130. Separate recovery responsibilities.
131. Maintain incident command.
132. Maintain decision logs.
133. Maintain communication channels.
134. Define executive escalation.
135. Define customer communication.
136. Freeze risky deployments during major incidents.
137. Do not make unrelated changes during recovery.
138. Keep recovery procedures simple.
139. Avoid hidden manual dependencies.
140. Test every documented manual step.
141. Remove obsolete runbook steps.
142. Review DR after every major architecture change.
143. Keep production and DR configuration aligned.
144. Detect DR drift.
145. Automate drift detection.
146. Keep regional configuration versioned.
147. Maintain dependency maps.
148. Assign owners.
149. Review RTO/RPO periodically.
150. Keep evidence of DR tests.
```

## 118. Rules 151–200

```text
151. Run tabletop exercises.
152. Run technical game days.
153. Test regional failure.
154. Test account-level recovery.
155. Test database corruption.
156. Test accidental deletion.
157. Test ransomware scenarios.
158. Test registry failure.
159. Test Git failure.
160. Test identity failure.
161. Test secret failure.
162. Test DNS failure.
163. Test observability failure.
164. Test third-party dependency failure.
165. Test failover.
166. Test failback.
167. Test recovery during peak load.
168. Test recovery with realistic data volume.
169. Test recovery with realistic traffic.
170. Measure RTO.
171. Measure RPO.
172. Measure restore duration.
173. Measure replication lag.
174. Measure traffic convergence.
175. Measure manual effort.
176. Record failures found during testing.
177. Track remediation items.
178. Re-test after remediation.
179. Do not treat tabletop completion as technical proof.
180. Do not treat backup success as restore proof.
181. Do not treat infrastructure creation as service recovery.
182. Validate business transactions.
183. Validate data correctness.
184. Validate security controls.
185. Validate monitoring.
186. Validate alerts.
187. Validate logging.
188. Validate audit trails.
189. Validate certificate paths.
190. Validate service discovery.
191. Validate external API connectivity.
192. Validate egress.
193. Validate ingress.
194. Validate load balancing.
195. Validate queue processing.
196. Validate scheduled jobs.
197. Validate batch workloads.
198. Validate cron workloads.
199. Validate asynchronous processing.
200. Keep DR ready after recovery.
```

## 119. Rules 201–250

```text
201. Do not rush failback.
202. Repair primary completely before failback.
203. Synchronize data before failback.
204. Validate primary before traffic shift.
205. Shift traffic gradually when practical.
206. Monitor after failback.
207. Keep DR recovery capability available during stabilization.
208. Define data reconciliation.
209. Define write ownership.
210. Prevent conflicting writes.
211. Define active-active consistency.
212. Define active-passive promotion.
213. Define replica promotion.
214. Define recovery-point selection.
215. Preserve historical recovery points.
216. Protect recovery data from the same disaster.
217. Avoid storing every backup in the same account.
218. Avoid storing every recovery dependency in the same region.
219. Separate critical recovery credentials.
220. Protect KMS access.
221. Protect root/break-glass procedures.
222. Monitor backup deletion.
223. Monitor replication health.
224. Alert on failed backups.
225. Alert on stale replicas.
226. Alert on DR drift.
227. Alert on recovery dependency failure.
228. Review backup retention.
229. Review restore duration.
230. Review DR capacity.
231. Review DR cost.
232. Review third-party contracts.
233. Review compliance requirements.
234. Review data residency.
235. Review log residency.
236. Review backup residency.
237. Review support access.
238. Review recovery account security.
239. Review recovery permissions.
240. Review recovery automation.
241. Review disaster declaration criteria.
242. Review incident communication.
243. Review recovery ownership.
244. Review dependency priorities.
245. Review degraded-mode behavior.
246. Review failure-domain assumptions.
247. Test after architecture migrations.
248. Test after major database changes.
249. Test after major Kubernetes changes.
250. Disaster Recovery is successful only when the organization can restore
     critical business capability within its tested RTO/RPO under a realistic
     failure scenario, with known ownership, recoverable data, working
     infrastructure, validated dependencies and a proven failback path.
```
---