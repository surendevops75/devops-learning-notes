# 19-DevOps-System-Design
# 08-Multi-Account-AWS-Architecture

## 1. Purpose

This file is a deep production-oriented guide to designing AWS environments
using multiple accounts as security, operational, billing, and failure
boundaries.

The central architecture principle is:

```text
AWS Organization
 |
Organizational Units
 |
AWS Accounts
 |
VPCs / Shared Services / Workloads
 |
EKS / EC2 / Databases / Applications
```

A mature multi-account strategy answers:

```text
Why do we need separate accounts?
What belongs in each account?
Who can access each account?
How do accounts communicate?
Where does networking live?
Where does security live?
Where do logs live?
How does CI/CD cross accounts?
How does GitOps reach production?
How are workloads authenticated?
How are guardrails enforced?
How is cost allocated?
How is an account recovered?
What happens if one account is compromised?
```

---

# PART I — FOUNDATIONS

## 2. What Is an AWS Account Boundary?

An AWS account provides an administrative and security boundary containing:

```text
IAM resources
service quotas
billing attribution
VPCs
resources
logs
policies
```

A multi-account architecture uses those boundaries deliberately.

It is stronger than simply creating multiple VPCs inside one account.

---

## 3. Why Multiple Accounts?

Common reasons:

```text
security isolation
blast-radius reduction
billing separation
compliance
organizational ownership
environment separation
production protection
service quota isolation
independent administration
```

Example:

```text
Organization
 |
+-- Security
+-- Log Archive
+-- Network
+-- Shared Services
+-- Dev
+-- Stage
+-- Prod
```

---

## 4. Account vs VPC vs Cluster

These are different boundaries:

```text
Account
 |
+-- VPC
      |
      +-- EKS
            |
            +-- Namespace
                  |
                  +-- Pod
```

Each layer has different isolation properties.

Do not assume:

```text
namespace = account
VPC = account
cluster = account
```

They solve different problems.

---

# PART II — AWS ORGANIZATIONS

## 5. AWS Organization

The organization provides centralized governance across member accounts.

Conceptually:

```text
Management Account
 |
AWS Organization
 |
+-- Security OU
+-- Infrastructure OU
+-- Workloads OU
+-- Sandbox OU
```

Keep the management account tightly protected and avoid using it for
ordinary workloads.

---

## 6. Organizational Units

OUs group accounts for policy and governance.

Example:

```text
Root
 |
+-- Security
|    +-- Security Account
|    +-- Log Archive
|
+-- Infrastructure
|    +-- Network
|    +-- Shared Services
|
+-- Workloads
     +-- Dev
     +-- Stage
     +-- Production
```

Design OUs around governance requirements rather than merely application
names.

---

## 7. OU Design Principle

A good OU structure allows:

```text
common policies
common security controls
common logging requirements
common compliance rules
```

Avoid creating hundreds of OUs simply to mirror organizational charts.

---

# PART III — MANAGEMENT ACCOUNT

## 8. Management Account

The organization management account is highly privileged.

Protect it with:

```text
strong MFA
central identity
minimal users
break-glass process
audit logging
strict operational procedures
```

Do not deploy normal production workloads into it.

---

## 9. Break-Glass

Emergency access should be:

```text
rare
controlled
audited
time-limited
tested
```

Flow:

```text
Incident
 |
authorization
 |
temporary privileged access
 |
mitigation
 |
audit
 |
access removal
```

---

# PART IV — SECURITY ACCOUNTS

## 10. Security Account

A dedicated security account can host centralized security tooling and
security operations.

Potential responsibilities:

```text
security findings
configuration monitoring
threat detection
security dashboards
incident workflows
```

---

## 11. Log Archive Account

Centralized immutable or protected log storage can be separated from
workload accounts.

Conceptually:

```text
Workload Accounts
 |
logs
 |
Log Archive Account
 |
protected storage
```

This prevents a compromised workload account from easily deleting all
central evidence.

---

## 12. Audit Account

An audit/security function may require read-only visibility into many
accounts.

Prefer:

```text
central identity
+
cross-account read-only roles
```

rather than distributing long-lived credentials.

---

# PART V — NETWORK ACCOUNT

## 13. Central Network Account

A network account may manage:

```text
Transit Gateway
central routing
Direct Connect
VPN
shared DNS
network inspection
egress architecture
```

Example:

```text
             Network Account
                    |
              Transit Gateway
             /       |       \
          Dev       Stage     Prod
          VPC        VPC       VPC
```

---

## 14. Centralized vs Decentralized Networking

### Centralized

```text
Network Account
 |
all shared connectivity
```

Benefits:

```text
central control
standardization
```

Risks:

```text
central dependency
organizational bottleneck
```

### Decentralized

Each account owns more networking.

Benefits:

```text
autonomy
smaller ownership boundary
```

Costs:

```text
duplication
routing complexity
```

Hybrid models are common.

---

# PART VI — SHARED SERVICES

## 15. Shared Services Account

Potential services:

```text
artifact infrastructure
CI runners
internal developer platform
DNS services
monitoring aggregation
shared tooling
```

Do not place every service into one account without considering its
security and failure implications.

---

# PART VII — PRODUCTION ACCOUNTS

## 16. Production Isolation

Production should generally have stronger boundaries than development.

Example:

```text
Dev Account
 |
Dev EKS

Stage Account
 |
Stage EKS

Prod Account
 |
Prod EKS
```

A compromised development account should not automatically provide access
to production.

---

# PART VIII — ACCOUNT NAMING

## 17. Naming

Use predictable names:

```text
org-prod-payments
org-prod-platform
org-stage
org-dev
org-security
org-network
```

Actual naming standards should support automation and ownership.

---

# PART IX — ACCOUNT METADATA

## 18. Account Registry

Maintain:

```text
account ID
account name
OU
environment
owner
business unit
region
network model
security tier
lifecycle state
```

Automate account metadata where possible.

---

# PART X — SERVICE CONTROL POLICIES

## 19. SCP Fundamentals

Service Control Policies provide organization-level guardrails.

Important principle:

```text
SCP does not grant permissions.
```

It defines maximum available permissions for affected principals/accounts
when applicable.

---

## 20. SCP Examples

Possible guardrails:

```text
deny leaving organization
deny disabling security services
deny deleting central logs
deny unsupported regions
deny dangerous actions outside approved roles
```

Test SCPs carefully.

A bad deny policy can break production.

---

## 21. Region Restriction

Organizations may restrict resource creation to approved regions.

But exclusions may be needed for global AWS services and organization
operations.

Never deploy a region restriction without testing the complete AWS service
dependency chain.

---

## 22. SCP Rollout

Safe process:

```text
design
 |
simulate
 |
audit
 |
sandbox
 |
non-production
 |
production
```

Do not apply a broad deny to every account as the first implementation step.

---

# PART XI — IDENTITY

## 23. Central Identity

Preferred model:

```text
Identity Provider
 |
AWS IAM Identity Center
 |
Permission Set
 |
Account
```

Users receive appropriate access without permanent IAM user credentials.

---

## 24. Permission Sets

Examples:

```text
ReadOnly
Developer
PowerUser
PlatformAdmin
SecurityAudit
Billing
```

Keep production permissions separate from development permissions.

---

## 25. Cross-Account Role Assumption

Concept:

```text
User / CI
 |
identity
 |
AssumeRole
 |
Target Account
 |
AWS API
```

Use trust policies to restrict who can assume the role.

---

## 26. CI Cross-Account Access

Preferred:

```text
CI identity
 |
short-lived AWS credentials
 |
assume deployment role
 |
target account
```

Avoid:

```text
static access key stored in CI
```

---

# PART XII — EKS ACROSS ACCOUNTS

## 27. Account-Isolated EKS

Example:

```text
Dev Account
 |
EKS Dev

Stage Account
 |
EKS Stage

Prod Account
 |
EKS Prod
```

This provides strong environment separation.

---

## 28. Production Platform Account

For very large organizations:

```text
Prod Platform Account
 |
EKS Platform Cluster

Prod Application Accounts
 |
external AWS services
```

But application placement should follow workload and ownership needs.

---

# PART XIII — CROSS-ACCOUNT IAM

## 29. Deployment Role

Example:

```text
CI
 |
AssumeRole
 |
ProdDeploymentRole
 |
EKS / AWS APIs
```

Permissions should include only required actions.

---

## 30. Role Chaining

Avoid unnecessarily long chains:

```text
CI -> Role A -> Role B -> Role C -> Role D
```

Every additional hop complicates:

```text
debugging
auditing
expiration
trust relationships
```

Keep the identity path understandable.

---

# PART XIV — CROSS-ACCOUNT NETWORKING

## 31. Transit Gateway

Example:

```text
              Network Account
                     |
                Transit Gateway
              /       |       \
          Dev VPC  Stage VPC  Prod VPC
```

Control route propagation and attachments carefully.

---

## 32. VPC Peering

Peering can connect accounts/VPCs directly.

Advantages:

```text
simple for small numbers
```

Disadvantages:

```text
mesh complexity
```

For large networks, Transit Gateway may provide a more scalable topology.

---

## 33. PrivateLink

PrivateLink is useful when one account exposes a service to another without
requiring broad network connectivity.

```text
Provider Account
 |
Endpoint Service
 |
PrivateLink
 |
Consumer Account
```

---

# PART XV — DNS

## 34. Central DNS

A centralized DNS architecture may use:

```text
Network Account
 |
Route 53 Resolver
 |
VPCs
```

Cross-account DNS sharing can be managed through appropriate AWS mechanisms.

---

## 35. DNS Failure

A shared DNS service can become a common dependency.

Design:

```text
redundancy
monitoring
fallback
controlled change
```

---

# PART XVI — EGRESS

## 36. Central Egress

Example:

```text
Workload VPC
 |
Transit Gateway
 |
Inspection / Egress VPC
 |
NAT / Firewall
 |
Internet
```

Benefits:

```text
central inspection
central policy
```

Costs:

```text
latency
data processing
routing complexity
central failure risk
```

---

## 37. Distributed Egress

Each workload VPC has:

```text
NAT
firewall
endpoints
```

Benefits:

```text
independence
smaller blast radius
```

Costs:

```text
more duplicated infrastructure
```

---

## 38. VPC Endpoints

Use private connectivity to AWS services where appropriate:

```text
ECR
S3
STS
Secrets Manager
CloudWatch
```

This can reduce unnecessary internet/NAT dependency.

---

# PART XVII — SHARED SECURITY

## 39. Central Security Controls

Potential architecture:

```text
Organization
 |
Security services
 |
all accounts
```

Use centralized findings and account-level enforcement.

---

## 40. Security Boundaries

A compromised account should have limited ability to affect:

```text
other accounts
central logs
security tooling
production
```

Cross-account trust is a major security consideration.

---

# PART XVIII — LOGGING

## 41. Central CloudTrail

Conceptually:

```text
All Accounts
 |
CloudTrail
 |
Central Log Archive
```

Protect central logs from workload administrators.

---

## 42. Central Application Logs

For Kubernetes:

```text
EKS clusters
 |
log collectors
 |
central ingestion
 |
storage/search
```

Keep enough local telemetry to diagnose regional or central-service
failures.

---

# PART XIX — OBSERVABILITY

## 43. Multi-Account Observability

Track:

```text
account
cluster
region
namespace
application
team
```

This enables fleet-wide analysis.

---

# PART XX — COST MANAGEMENT

## 44. Cost Allocation

Account separation naturally improves cost attribution.

Track:

```text
account
environment
business unit
team
service
cluster
```

Use tagging standards where supported.

---

## 45. Cost Guardrails

Prevent:

```text
unapproved expensive regions
unexpected GPU capacity
uncontrolled resources
forgotten test accounts
```

Combine organization policy with budgets and monitoring.

---

# PART XXI — CI/CD

## 46. Cross-Account Pipeline

Architecture:

```text
Developer
 |
Git
 |
CI
 |
Build/Test/Scan
 |
Artifact Registry
 |
Assume Deployment Role
 |
Target Account
 |
EKS
```

Prefer immutable artifacts.

---

## 47. GitOps Cross-Account

Safer model:

```text
CI
 |
artifact
 |
GitOps repository
 |
Argo CD
 |
Target EKS
```

The CI system does not need broad direct production cluster credentials.

---

# PART XXII — ARTIFACTS

## 48. Central Registry

Possible model:

```text
Shared Registry Account
 |
container registry
 |
+-- Dev
+-- Stage
+-- Prod
```

Consider:

```text
cross-account pulls
regional replication
permissions
retention
```

---

# PART XXIII — SECRETS

## 49. Secrets Architecture

Prefer:

```text
AWS Secrets Manager
 |
account/region scoped access
 |
workload identity
 |
application
```

Avoid making every workload capable of reading every account's secrets.

---

# PART XXIV — ENCRYPTION

## 50. KMS

Separate keys by security boundary where required.

Examples:

```text
prod key
security key
log archive key
application key
```

Be careful with cross-account KMS policies.

---

## 51. Cross-Account KMS

Access typically requires compatible permissions in both the identity and
key policies.

Test encryption workflows before production migration.

---

# PART XXV — S3

## 52. Central Buckets

Central security/logging buckets should have:

```text
restricted writes
restricted deletes
encryption
versioning
lifecycle
monitoring
```

Where supported and appropriate, use controls that make deletion harder.

---

# PART XXVI — DATABASES

## 53. Database Account Boundary

A database can remain in:

```text
application account
```

or:

```text
data account
```

depending on organizational architecture.

---

## 54. Cross-Account Database Access

Prefer private connectivity:

```text
EKS VPC
 |
private routing
 |
database VPC
 |
database
```

Avoid exposing databases publicly merely to solve account connectivity.

---

# PART XXVII — ACCOUNT PROVISIONING

## 55. Account Factory

New account process:

```text
request
 |
approval
 |
create account
 |
place OU
 |
baseline SCP
 |
identity
 |
logging
 |
security
 |
network
 |
budget
 |
ready
```

Automate this process.

---

# PART XXVIII — ACCOUNT BASELINE

## 56. Baseline Components

Every workload account may receive:

```text
CloudTrail
security services
logging
IAM baseline
budgets
tagging
SCP
config
network baseline
```

Exact controls depend on organizational requirements.

---

# PART XXIX — ACCOUNT DECOMMISSION

## 57. Decommissioning

Process:

```text
inventory
 |
backup
 |
remove dependencies
 |
remove IAM access
 |
delete workloads
 |
remove network attachments
 |
validate no active resources
 |
close account
```

Never close an account simply because its name suggests it is unused.

---

# PART XXX — FAILURE DOMAINS

## 58. Account Failure

If one account is compromised:

```text
Account A
 |
failure
X
Account B
 |
should remain operational
```

This depends on strong identity and network isolation.

---

## 59. Shared Dependency Failure

If all accounts depend on:

```text
central DNS
central egress
central registry
central GitOps
```

that service can become a fleet-wide failure domain.

Design redundancy and fallback where business requirements justify it.

---

# PART XXXI — SECURITY INCIDENT

## 60. Compromised Workload Account

Response:

```text
detect
 |
isolate account/network
 |
revoke compromised identities
 |
preserve evidence
 |
rotate secrets
 |
validate cross-account trusts
 |
rebuild affected workloads
 |
restore from trusted source
```

---

## 61. Compromised Cross-Account Role

Check:

```text
trust policy
permissions policy
CloudTrail
session history
dependent accounts
KMS access
S3 access
EKS access
```

Then revoke or restrict the trust path.

---

# PART XXXII — DISASTER RECOVERY

## 62. Account-Level DR

If an account becomes unusable:

```text
Infrastructure as Code
 |
new account / recovery account
 |
VPC
 |
EKS
 |
IAM
 |
addons
 |
GitOps
 |
applications
 |
data restore
```

Account recovery requires more than restoring Kubernetes YAML.

---

# PART XXXIII — REGION STRATEGY

## 63. Account + Region Matrix

Example:

```text
Prod Account
 |
+-- Region A
|    +-- EKS
|
+-- Region B
     +-- EKS
```

A multi-account strategy and multi-region strategy solve different
failure and governance problems.

---

# PART XXXIV — PLATFORM ARCHITECTURE

## 64. Enterprise Reference

```text
AWS Organization
 |
+------------------------------------------------+
|                                                |
Security OU                              Infrastructure OU
|                                        |
+-- Security Account                     +-- Network Account
+-- Log Archive Account                  +-- Shared Services
|                                                |
+------------------------------------------------+
                         |
                  Workloads OU
                         |
          +--------------+--------------+
          |              |              |
        Dev            Stage          Prod
          |              |              |
        VPC              VPC            VPC
          |              |              |
        EKS              EKS            EKS
```

---

# PART XXXV — MULTI-ACCOUNT EKS

## 65. Production EKS Fleet

```text
Prod Account A
 |
EKS Cluster A

Prod Account B
 |
EKS Cluster B

Prod Account C
 |
EKS Cluster C
```

Central platform standards can be applied through GitOps and automation
without giving one account unrestricted control over everything.

---

# PART XXXVI — GOVERNANCE

## 66. Governance Layers

```text
Organization
 |
SCP
 |
Account
 |
IAM
 |
VPC
 |
EKS
 |
Kubernetes RBAC
 |
Application
```

Each layer should enforce the controls appropriate to its scope.

---

# PART XXXVII — TAGGING

## 67. Mandatory Tags

Possible standard:

```text
Environment
Owner
Application
CostCenter
BusinessUnit
ManagedBy
DataClassification
```

Tags support:

```text
cost
inventory
automation
ownership
```

---

# PART XXXVIII — QUOTAS

## 68. AWS Service Quotas

Multi-account architecture also distributes some service quota pressure.

But do not use account multiplication merely as a workaround for poor
capacity planning.

Request quota increases where appropriate.

---

# PART XXXIX — PLATFORM AUTOMATION

## 69. Account Automation

Automate:

```text
account creation
OU placement
baseline
logging
security
network
budgets
identity
```

---

## 70. Terraform Strategy

Possible repository:

```text
terraform/
 |
+-- organization/
+-- accounts/
+-- network/
+-- security/
+-- eks/
```

Use clear ownership boundaries.

---

# PART XL — GITOPS

## 71. GitOps Boundary

Terraform:

```text
accounts
VPC
IAM
EKS
```

GitOps:

```text
Kubernetes applications
addons
policies
namespaces
```

Avoid both tools continuously changing the same resource.

---

# PART XLI — PROGRESSIVE CHANGES

## 72. Organization-Wide Change

Example:

```text
Sandbox
 |
Dev
 |
Stage
 |
Prod Canary
 |
Prod Fleet
```

Use waves for:

```text
SCP
IAM
network
EKS addons
policy
```

---

# PART XLII — SCP INCIDENT

## 73. Bad SCP

Symptoms:

```text
AWS API AccessDenied
```

Response:

```text
identify affected OU/account
 |
identify SCP
 |
compare policy evaluation
 |
remove/revise safely
 |
validate
```

Never debug only IAM when SCPs may be involved.

---

# PART XLIII — IAM DEBUGGING

## 74. AccessDenied Flow

Check:

```text
identity policy
+
resource policy
+
SCP
+
permissions boundary
+
session policy
+
KMS policy
+
trust policy
```

An allowed action can still fail because another policy layer denies it.

---

# PART XLIV — CROSS-ACCOUNT TROUBLESHOOTING

## 75. Network Failure

Check:

```text
route tables
TGW attachment
TGW routes
security groups
NACLs
DNS
firewall
endpoint
```

Trace:

```text
source VPC
 |
routing
 |
destination VPC
 |
service
```

---

# PART XLV — EKS ACCESS TROUBLESHOOTING

## 76. CI Cannot Reach EKS

Check:

```text
network path
EKS endpoint mode
security groups
authentication
authorization
role trust
RBAC
```

Separate:

```text
cannot reach API
```

from:

```text
reached API but unauthorized
```

---

# PART XLVI — OBSERVABILITY TROUBLESHOOTING

## 77. Logs Missing

Check:

```text
collector
 |
network
 |
credentials
 |
destination
 |
permissions
 |
retention
```

Central observability failures should not silently eliminate all incident
evidence.

---

# PART XLVII — COST ARCHITECTURE

## 78. Account Cost Model

Account-level separation enables:

```text
production cost
development cost
security cost
network cost
shared platform cost
```

Then allocate shared costs using an agreed model.

---

# PART XLVIII — PLATFORM SLOs

## 79. Multi-Account Platform SLOs

Examples:

```text
account provisioning time
baseline completion
deployment success
cross-account connectivity
central logging availability
security finding delivery
```

---

# PART XLIX — INCIDENT DESIGN

## 80. Shared Infrastructure Failure

If Network Account fails:

```text
multiple workload accounts
 |
may lose connectivity
```

Therefore central infrastructure should be:

```text
highly available
carefully changed
strongly monitored
```

---

# PART L — CENTRALIZATION TRADE-OFFS

## 81. Centralize

Good candidates:

```text
governance
security findings
log archive
identity
standards
```

Potentially dangerous to centralize without resilience:

```text
single egress
single DNS path
single registry
single GitOps controller
```

---

# PART LI — DECENTRALIZATION

## 82. Decentralize

Useful for:

```text
critical regional services
high-isolation environments
independent failure domains
```

Costs:

```text
duplication
more operations
higher cost
```

---

# PART LII — CELL-BASED ACCOUNTS

## 83. Account Cells

```text
Cell A
 |
Account A
 |
EKS A

Cell B
 |
Account B
 |
EKS B
```

Each cell has:

```text
network
identity
workloads
observability
deployment boundary
```

Useful for very large platforms.

---

# PART LIII — SECURITY ARCHITECTURE

## 84. Defense in Depth

```text
Organization SCP
 |
Account IAM
 |
VPC controls
 |
Security Groups
 |
NetworkPolicy
 |
Kubernetes RBAC
 |
Pod Security
 |
Admission
 |
Image security
```

---

# PART LIV — DATA CLASSIFICATION

## 85. Data Tiers

Example:

```text
Public
Internal
Confidential
Restricted
```

Use classification to influence:

```text
account
region
encryption
access
logging
retention
```

---

# PART LV — PRODUCTION CHECKLIST

## 86. Organization

```text
[ ] management account protected
[ ] OUs defined
[ ] SCP strategy
[ ] security account
[ ] log archive
[ ] account inventory
```

## 87. Identity

```text
[ ] central SSO
[ ] permission sets
[ ] least privilege
[ ] break-glass
[ ] cross-account roles
[ ] short-lived CI credentials
```

## 88. Networking

```text
[ ] VPC CIDRs
[ ] TGW where appropriate
[ ] routes
[ ] private connectivity
[ ] DNS
[ ] egress
[ ] VPC endpoints
```

## 89. Workloads

```text
[ ] EKS isolation
[ ] GitOps
[ ] workload identity
[ ] secrets
[ ] observability
[ ] backup
[ ] DR
```

---

# PART LVI — SENIOR SYSTEM DESIGN

## 90. Design AWS for 50 Teams

Approach:

```text
1. Clarify compliance.
2. Define account boundaries.
3. Define OU structure.
4. Define identity.
5. Define security.
6. Define central logging.
7. Define network.
8. Define workload accounts.
9. Define EKS strategy.
10. Define CI/CD.
11. Define GitOps.
12. Define observability.
13. Define cost.
14. Define DR.
15. Define governance.
```

---

## 91. Design for Production Isolation

Answer:

```text
I would isolate production at the AWS account boundary rather than
relying only on namespaces or VPCs. I would then layer SCPs, IAM,
network controls, Kubernetes RBAC and workload identity so compromise
of a lower environment does not automatically provide production access.
```

---

## 92. Design Cross-Account CI/CD

Answer:

```text
The pipeline builds and scans an immutable artifact, then either updates
the GitOps desired state or assumes a tightly scoped deployment role.
Production access uses short-lived credentials and a target-account trust
policy rather than static access keys.
```

---

## 93. Design Network Connectivity

Answer:

```text
For a small environment I may use direct VPC connectivity. At larger
scale I would evaluate Transit Gateway for hub-and-spoke routing and
PrivateLink for selected service exposure. I would avoid creating a
full-mesh network unless the requirements justify it.
```

---

## 94. Design Security

Answer:

```text
I would separate security, logging, network and workload accounts, apply
organization-level guardrails, centralize identity and evidence, and
keep cross-account trust minimal. Every shared dependency would be
evaluated as a potential blast-radius boundary.
```

---

## 95. Design Account Recovery

Answer:

```text
I would make account recovery infrastructure-driven. VPC, IAM, EKS and
platform configuration would be reproducible from source control, while
application data and secrets would have independent backup and restore
strategies.
```

---

# PART LVII — FAILURE SCENARIOS

## 96. Dev Account Compromised

Expected boundary:

```text
Dev
 |
compromised
X
Prod
 |
protected
```

Validate:

```text
IAM trust
SCP
network
secrets
CI permissions
GitOps permissions
```

---

## 97. Network Account Failure

Impact can include:

```text
cross-VPC connectivity
central egress
inspection
```

Mitigate through:

```text
multi-AZ network appliances
redundant TGW design
careful route changes
local fallback where appropriate
```

---

## 98. Log Archive Failure

Workload services should continue operating even if central log ingestion
is temporarily unavailable, while local or durable buffering preserves
important evidence where feasible.

---

## 99. Security Account Failure

Security tooling failure should not automatically stop production unless
the security architecture intentionally requires enforcement at that
dependency.

Design fail-open vs fail-closed behavior deliberately.

---

## 100. Management Account Incident

Treat management-account compromise as a critical organization-level
incident.

Protect:

```text
root access
MFA
organization administration
SCP authority
billing
account lifecycle
```

---

# PART LVIII — 150 PRODUCTION GOLDEN RULES

## 101. Rules 1–30

```text
1. Use multiple accounts for deliberate isolation.
2. Do not create accounts without an ownership model.
3. Protect the management account.
4. Do not run ordinary workloads in the management account.
5. Design OUs around governance.
6. Keep security boundaries explicit.
7. Separate production from lower environments.
8. Use account boundaries for strong isolation.
9. Use VPCs for network boundaries.
10. Use clusters for workload/platform boundaries.
11. Use namespaces for Kubernetes tenancy.
12. Do not confuse these boundaries.
13. Maintain an account inventory.
14. Automate account provisioning.
15. Automate account baselines.
16. Automate account decommissioning.
17. Protect break-glass access.
18. Test emergency access.
19. Use centralized identity.
20. Prefer short-lived credentials.
21. Avoid permanent CI keys.
22. Restrict cross-account trust.
23. Keep role chains short.
24. Use least-privilege permission sets.
25. Protect production roles.
26. Use SCPs as guardrails.
27. Remember SCPs do not grant permissions.
28. Test SCPs before fleet-wide rollout.
29. Roll policy changes progressively.
30. Treat a broad SCP deny as a production change.
```

## 102. Rules 31–60

```text
31. Centralize security evidence.
32. Protect the log archive account.
33. Separate security tooling from workloads.
34. Use a network account where central networking is justified.
35. Do not make central networking an unnecessary single point of failure.
36. Use Transit Gateway when hub-and-spoke scale requires it.
37. Avoid network meshes that do not scale.
38. Use PrivateLink for selected service exposure.
39. Plan CIDRs globally.
40. Avoid overlapping VPC CIDRs.
41. Design DNS centrally only when it improves operations.
42. Protect shared DNS.
43. Design egress deliberately.
44. Avoid a single-AZ central egress path.
45. Use VPC endpoints where appropriate.
46. Control internet egress.
47. Inspect traffic where requirements justify it.
48. Keep databases private.
49. Prefer private cross-account connectivity.
50. Do not expose databases publicly to simplify networking.
51. Separate CI identity from runtime identity.
52. Use target-account deployment roles.
53. Protect trust policies.
54. Audit AssumeRole activity.
55. Restrict production deployment permissions.
56. Prefer GitOps for Kubernetes deployment.
57. Keep infrastructure ownership clear.
58. Avoid Terraform and GitOps ownership conflicts.
59. Use immutable artifacts.
60. Track artifact provenance.
```

## 103. Rules 61–90

```text
61. Scan container images.
62. Use image digests where appropriate.
63. Protect registry access.
64. Plan cross-account image pulls.
65. Plan regional registry availability.
66. Protect secrets by account and workload.
67. Avoid universal secret-reader roles.
68. Use workload identity.
69. Protect KMS policies.
70. Separate keys where security requires it.
71. Test cross-account encryption.
72. Centralize CloudTrail where appropriate.
73. Protect central logs from workload deletion.
74. Monitor log delivery.
75. Keep enough local evidence for outages.
76. Standardize tags.
77. Track owners.
78. Track environments.
79. Track cost centers.
80. Track data classification.
81. Use account-level budgets.
82. Monitor unexpected spend.
83. Control expensive resources.
84. Remove unused accounts.
85. Remove unused VPCs.
86. Remove unused network attachments.
87. Review cross-account trusts.
88. Review stale IAM roles.
89. Review stale permission sets.
90. Review SCP exceptions.
```

## 104. Rules 91–120

```text
91. Treat production account changes as high-risk.
92. Use change waves.
93. Test security policies in non-production.
94. Test network changes progressively.
95. Test identity changes before production.
96. Test account baseline automation.
97. Maintain disaster recovery procedures.
98. Make account rebuild repeatable.
99. Keep infrastructure code versioned.
100. Keep GitOps configuration versioned.
101. Back up application data independently.
102. Back up critical secrets appropriately.
103. Test restoration.
104. Define RTO.
105. Define RPO.
106. Test regional recovery where required.
107. Test account recovery assumptions.
108. Document external dependencies.
109. Document shared dependencies.
110. Minimize shared failure domains.
111. Use cell architecture for very large fleets.
112. Standardize workload account foundations.
113. Standardize EKS account patterns.
114. Standardize CI/CD roles.
115. Standardize logging.
116. Standardize security.
117. Standardize networking.
118. Allow controlled exceptions.
119. Keep exceptions documented.
120. Review exceptions periodically.
```

## 105. Rules 121–150

```text
121. Never assume account separation alone provides complete security.
122. Combine SCP with IAM.
123. Combine IAM with network controls.
124. Combine network controls with Kubernetes policy.
125. Protect management-plane identities.
126. Protect organization administration.
127. Protect central security accounts.
128. Protect central logging.
129. Protect network control planes.
130. Protect shared CI/CD.
131. Protect GitOps repositories.
132. Avoid universal production credentials.
133. Avoid universal cross-account roles.
134. Reduce blast radius of shared services.
135. Evaluate centralization trade-offs.
136. Evaluate decentralization trade-offs.
137. Design for partial failures.
138. Design for account compromise.
139. Design for network partition.
140. Design for central-service outage.
141. Design for region failure.
142. Monitor fleet health.
143. Monitor account security posture.
144. Monitor cost posture.
145. Monitor policy drift.
146. Monitor identity drift.
147. Monitor network drift.
148. Test what the architecture claims to protect.
149. Explain trade-offs during system-design interviews.
150. A mature AWS multi-account platform uses accounts as intentional
     security, governance, operational and failure boundaries while
     preserving enough standardization to operate the organization as
     one platform.
```

# END OF 08-Multi-Account-AWS-Architecture.md
