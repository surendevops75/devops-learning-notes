# 17-JFrog-Artifactory
# 20-ECR-vs-Artifactory

## 1. Purpose

This file provides a production-oriented comparison of Amazon Elastic
Container Registry (Amazon ECR) and JFrog Artifactory.

The goal is not to declare one product universally better. The correct
choice depends on:

```text
cloud strategy
application architecture
package ecosystem
multi-cloud requirements
security model
CI/CD architecture
Kubernetes platform
cost model
compliance
operational maturity
```

This file covers:

- ECR fundamentals
- Artifactory fundamentals
- architecture comparison
- repository models
- Docker/OCI images
- package management
- authentication
- AWS IAM
- Artifactory RBAC
- workload identity
- GitHub/GitLab/Jenkins
- EKS
- multi-account AWS
- multi-region
- multi-cloud
- image scanning
- replication
- retention
- lifecycle
- performance
- availability
- disaster recovery
- cost considerations
- migration
- production decision framework
- real-world scenarios
- interview preparation
- production checklist

---

# PART I — ECR FUNDAMENTALS

## 2. What Is Amazon ECR?

Amazon Elastic Container Registry is AWS's managed container registry
service for storing and distributing container images.

It integrates naturally with:

```text
ECS
EKS
EC2
AWS IAM
AWS Organizations
AWS CloudTrail
AWS security services
```

---

## 3. ECR Architecture

Basic model:

```text
Developer / CI
      |
      v
     ECR
      |
      v
EKS / ECS / EC2
```

---

## 4. ECR Repository

An ECR repository stores container images.

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service
```

---

## 5. ECR Registry

An AWS account and region provide the registry namespace.

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

---

# PART II — ARTIFACTORY FUNDAMENTALS

## 6. What Is JFrog Artifactory?

Artifactory is an enterprise universal artifact repository.

It can manage:

```text
Docker/OCI
Maven
Gradle
NPM
PyPI
Helm
NuGet
Generic artifacts
```

---

## 7. Artifactory Architecture

```text
CI/CD
  |
  v
Artifactory
  |
  +--> Docker
  +--> Maven
  +--> NPM
  +--> PyPI
  +--> Helm
  |
  v
Kubernetes
```

---

## 8. Artifactory Repository Types

```text
Local
Remote
Virtual
```

---

## 9. ECR vs Artifactory — Core Difference

Simplified:

```text
ECR:
AWS-native container registry

Artifactory:
Enterprise universal artifact platform
```

This distinction is central to the architecture decision.

---

# PART III — FEATURE COMPARISON

## 10. High-Level Comparison

| Capability | ECR | Artifactory |
|---|---|---|
| Docker/OCI | Yes | Yes |
| Maven | No | Yes |
| NPM | No | Yes |
| PyPI | No | Yes |
| Helm | OCI support | Yes |
| AWS integration | Excellent | Good |
| Multi-cloud | Possible | Strong |
| Universal artifact repository | No | Yes |
| IAM integration | Excellent | Strong RBAC |
| AWS-native operations | Excellent | External platform |
| Remote repositories | Different model | Core feature |
| Virtual repositories | No direct equivalent | Yes |
| Build Info | Limited registry context | Strong |
| EKS integration | Native | Strong |
| Multi-account AWS | Strong | Strong |
| Enterprise package governance | Container-focused | Broad |

Feature availability and exact behavior can change with product
versions and licensing.

---

# PART IV — WHEN ECR IS THE BETTER CHOICE

## 11. AWS-Centric Organization

If the organization is heavily AWS-focused:

```text
EKS
ECS
EC2
IAM
VPC
CloudTrail
```

ECR is often operationally attractive.

---

## 12. Simple Container Architecture

If the organization only needs:

```text
Docker/OCI images
```

a dedicated cloud-native registry can reduce platform complexity.

---

## 13. EKS-First Platform

Example:

```text
GitHub Actions
      |
      v
AWS IAM/OIDC
      |
      v
ECR
      |
      v
EKS
```

This is a natural AWS architecture.

---

# PART V — WHEN ARTIFACTORY IS THE BETTER CHOICE

## 14. Multiple Package Ecosystems

If the organization manages:

```text
Maven
NPM
PyPI
Docker
Helm
```

a universal repository can simplify governance.

---

## 15. Multi-Cloud

Example:

```text
AWS EKS
Azure AKS
Google GKE
On-prem Kubernetes
```

Artifactory can provide a common artifact platform.

---

## 16. Enterprise Artifact Governance

If teams need:

```text
common repository strategy
remote caching
virtual repositories
artifact promotion
Build Info
broad package governance
```

Artifactory can be a stronger platform choice.

---

# PART VI — AUTHENTICATION

## 17. ECR Authentication

ECR integrates with AWS IAM.

Typical pattern:

```text
CI identity
   |
   v
AWS IAM
   |
   v
ECR
```

---

## 18. ECR Login

A common AWS CLI pattern is:

```bash
aws ecr get-login-password --region ap-south-1 \
  | docker login \
      --username AWS \
      --password-stdin \
      123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

The AWS identity must have appropriate ECR permissions.

---

## 19. Artifactory Authentication

Artifactory can use:

```text
access tokens
service identities
RBAC
external identity integrations
federated authentication
```

The exact implementation depends on the organization's JFrog Platform
and identity architecture.

---

# PART VII — IAM VS RBAC

## 20. ECR Authorization

ECR commonly relies on:

```text
AWS IAM policies
+
repository policies where applicable
```

---

## 21. Artifactory Authorization

Artifactory commonly uses:

```text
users
groups
projects
permission targets
repositories
tokens
```

---

## 22. Example ECR Permissions

CI push requires permissions such as the relevant ECR upload actions.

Runtime pull requires the appropriate ECR read actions.

Do not grant:

```text
AdministratorAccess
```

just to make image pushes work.

---

## 23. Example Artifactory Permissions

```text
CI:
DEPLOY payment-docker-local

Runtime:
READ payment-docker-virtual

No:
DELETE
ADMIN
```

---

# PART VIII — EKS INTEGRATION

## 24. EKS + ECR

ECR is tightly integrated with AWS.

Conceptually:

```text
EKS
 |
 v
AWS IAM
 |
 v
ECR
```

Depending on the EKS setup, node or workload identities can be used
to authorize image pulls.

---

## 25. EKS + Artifactory

```text
EKS
 |
 v
Network
 |
 v
Artifactory
```

The cluster needs:

```text
DNS
network connectivity
TLS trust
registry authentication
READ permission
```

---

## 26. Operational Difference

With ECR:

```text
AWS networking + IAM
```

is the primary ecosystem.

With Artifactory:

```text
AWS networking
+
external registry platform
+
Artifactory identity/RBAC
```

must be considered.

---

# PART IX — MULTI-ACCOUNT AWS

## 27. ECR Across Accounts

A common enterprise model:

```text
Shared Services Account
        |
        v
ECR
        |
        v
Application Accounts
```

Repository policies and IAM roles can support controlled cross-account
access.

---

## 28. EKS Pulling From Another Account

Concept:

```text
EKS Account
   |
   v
IAM
   |
   v
Cross-account ECR access
   |
   v
ECR
```

Use narrowly scoped permissions.

---

## 29. Artifactory Across AWS Accounts

```text
Account A EKS
      |
      v
      Artifactory
      ^
      |
Account B EKS
```

The registry is external to the AWS account boundary.

This can simplify common-registry designs but introduces external
network and identity dependencies.

---

# PART X — MULTI-REGION

## 30. ECR Multi-Region

AWS provides replication capabilities for ECR.

Concept:

```text
ECR Region A
     |
     v
ECR Region B
```

This can support regional application deployment.

---

## 31. Why Replicate?

Benefits:

```text
lower pull latency
regional availability
disaster recovery
reduced cross-region dependency
```

---

## 32. Artifactory Multi-Region

Artifactory deployments can be architected for distributed operation,
replication and high availability depending on the JFrog edition and
architecture.

Example:

```text
Region A
 Artifactory
     |
 replication
     |
Region B
 Artifactory
```

Exact topology should be designed against current JFrog capabilities.

---

# PART XI — MULTI-CLOUD

## 33. ECR Multi-Cloud

ECR can still be consumed outside AWS, but the organization must manage:

```text
network
authentication
egress
registry access
```

---

## 34. Artifactory Multi-Cloud

A common pattern:

```text
                  Artifactory
                 /     |      \
                /      |       \
              EKS     AKS      GKE
```

This provides one enterprise artifact platform.

---

## 35. Trade-Off

Artifactory:

```text
common platform
+
central governance
```

but:

```text
additional platform
+
licensing
+
operations
```

ECR:

```text
AWS-native
+
managed
```

but:

```text
AWS-centric
```

---

# PART XII — PACKAGE ECOSYSTEM

## 36. ECR

Primary focus:

```text
container images
```

---

## 37. Artifactory

Can manage:

```text
container images
Java artifacts
JavaScript packages
Python packages
Helm
NuGet
generic binaries
```

---

## 38. Enterprise Example

Application:

```text
Java service
```

May produce:

```text
payment-service.jar
Docker image
Helm chart
SBOM
```

Artifactory can centralize these artifact types.

ECR primarily stores the resulting container image.

---

# PART XIII — REMOTE DEPENDENCY MANAGEMENT

## 39. Artifactory Remote Repository

Example:

```text
Artifactory
    |
    v
Docker Hub / External Registry
```

It can cache external images according to repository policy.

---

## 40. ECR Alternative

AWS provides other services and mechanisms for dependency/security
management, but ECR itself is primarily a container registry rather
than a universal remote repository manager.

---

# PART XIV — VIRTUAL REPOSITORIES

## 41. Artifactory Virtual

Example:

```text
docker-virtual
 |
 +--> internal
 |
 +--> external cache
```

Consumers use one endpoint.

---

## 42. Why This Matters

Applications do not need to know every underlying repository.

Benefits:

```text
central policy
simpler configuration
controlled dependency access
repository abstraction
```

---

# PART XV — IMAGE SCANNING

## 43. ECR Scanning

ECR supports image scanning capabilities integrated into AWS security
workflows.

Organizations should configure the scanning mode and vulnerability
response policy appropriate to their AWS environment.

---

## 44. Artifactory Scanning

JFrog environments can integrate image/artifact security scanning and
policy controls depending on the deployed JFrog security products and
licensing.

---

## 45. Security Comparison

The important question is not simply:

```text
Which scanner is better?
```

Instead evaluate:

```text
coverage
update frequency
policy controls
SBOM
CI integration
admission integration
reporting
workflow
```

---

# PART XVI — IMAGE IMMUTABILITY

## 46. ECR

Use immutable image tags where the repository policy supports the
desired release model.

---

## 47. Artifactory

Use repository/version policies that prevent accidental overwrites.

---

## 48. Best Practice

Regardless of registry:

```text
Build once
 ↓
Immutable image
 ↓
Digest
 ↓
Promote/deploy
```

---

# PART XVII — BUILD INFO

## 49. Artifactory Build Info

Artifactory provides strong Build Info capabilities.

It can associate:

```text
Git commit
build
dependencies
artifacts
environment metadata
```

---

## 50. ECR Metadata

ECR provides image and repository metadata, and AWS services can
provide additional build/deployment traceability.

However, the model is different from Artifactory's universal Build Info
approach.

---

## 51. Traceability

Artifactory:

```text
Production image
 ↓
Build Info
 ↓
CI
 ↓
Git
```

ECR:

```text
Production image
 ↓
ECR metadata
 ↓
CI/CD metadata
 ↓
Git
```

A mature AWS platform can still achieve end-to-end traceability.

---

# PART XVIII — CI/CD INTEGRATION

## 52. GitHub Actions + ECR

Common architecture:

```text
GitHub Actions
      |
      v
AWS OIDC
      |
      v
IAM Role
      |
      v
ECR
```

This can avoid long-lived AWS access keys in GitHub.

---

## 53. GitHub Actions + Artifactory

```text
GitHub Actions
      |
      v
Artifactory identity
      |
      v
Artifactory
```

The organization can use scoped tokens or an appropriate federation
model.

---

## 54. GitLab + ECR

```text
GitLab Runner
      |
      v
AWS identity
      |
      v
ECR
```

---

## 55. GitLab + Artifactory

```text
GitLab Runner
      |
      v
Artifactory service identity
      |
      v
Artifactory
```

---

## 56. Jenkins + ECR

Jenkins can assume an AWS role or use an approved AWS credential
mechanism.

---

## 57. Jenkins + Artifactory

Jenkins can use:

```text
Artifactory credentials
JFrog CLI
JFrog integrations
```

depending on the enterprise implementation.

---

# PART XIX — KUBERNETES DEPLOYMENT

## 58. ECR Deployment

```yaml
containers:
  - name: payment
    image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment-service:4.2.1
```

---

## 59. Artifactory Deployment

```yaml
containers:
  - name: payment
    image: artifactory.example.com/docker-local/payment-service:4.2.1
```

---

## 60. Digest Pinning

ECR:

```text
...amazonaws.com/payment-service@sha256:...
```

Artifactory:

```text
artifactory.example.com/docker-local/payment-service@sha256:...
```

The concept is identical.

---

# PART XX — GITOPS

## 61. ECR + GitOps

```text
CI
 ↓
ECR
 ↓
Git desired state
 ↓
Argo CD
 ↓
EKS
```

---

## 62. Artifactory + GitOps

```text
CI
 ↓
Artifactory
 ↓
Git desired state
 ↓
Argo CD
 ↓
EKS
```

---

## 63. Important Principle

GitOps does not require Artifactory.

GitOps requires:

```text
immutable artifact
+
declarative desired state
+
reconciliation
```

---

# PART XXI — PERFORMANCE

## 64. ECR Performance

For AWS workloads, ECR can provide a naturally integrated path:

```text
EKS
 ↓
AWS network
 ↓
ECR
```

---

## 65. Artifactory Performance

Performance depends on:

```text
Artifactory deployment
network path
storage
caching
replication
registry location
image size
```

---

## 66. Large Images

For both systems:

```text
large image
 ↓
more storage
 ↓
more network
 ↓
slower deployment
```

Optimize image size regardless of registry.

---

# PART XXII — AVAILABILITY

## 67. ECR

As an AWS managed service, ECR removes much of the registry server
management burden.

---

## 68. Artifactory

Artifactory availability depends on the chosen JFrog deployment model,
edition and architecture.

Production deployments may require:

```text
HA
database availability
storage availability
load balancing
replication
backup
DR
```

---

## 69. Operational Difference

ECR:

```text
less registry infrastructure management
```

Artifactory:

```text
more platform responsibility
```

but potentially:

```text
more enterprise artifact capabilities
```

---

# PART XXIII — DISASTER RECOVERY

## 70. ECR DR

Design around:

```text
multi-region replication
AWS account strategy
repository recovery
deployment references
```

---

## 71. Artifactory DR

Consider:

```text
database
filestore
configuration
replication
backup
DNS
credentials
```

---

## 72. DR Test

Do not only document DR.

Test:

```text
registry unavailable
 ↓
switch/restore
 ↓
pull image
 ↓
deploy
 ↓
validate
```

---

# PART XXIV — RETENTION

## 73. ECR Lifecycle Policies

ECR supports lifecycle policies to expire images according to defined
rules.

Examples:

```text
remove old untagged images
retain last N development images
retain production releases longer
```

---

## 74. Artifactory Retention

Artifactory retention can be implemented using repository policies,
cleanup jobs and artifact lifecycle mechanisms according to the
organization's configuration.

---

## 75. Production Principle

Never delete an image only because it is old.

Check:

```text
currently deployed?
rollback target?
supported release?
audit requirement?
compliance retention?
```

---

# PART XXV — COST CONSIDERATIONS

## 76. ECR Cost Model

Evaluate:

```text
storage
data transfer
requests
regional architecture
cross-region traffic
```

Use current AWS pricing for an actual financial estimate.

---

## 77. Artifactory Cost Model

Evaluate:

```text
license/subscription
compute
database
storage
network
HA
backup
operations
```

---

## 78. Hidden Operational Cost

For self-managed or managed enterprise Artifactory, account for:

```text
platform engineering
upgrades
monitoring
backup
security
capacity
incident response
```

---

## 79. Cost Decision

Do not compare only:

```text
registry storage price
```

Compare:

```text
total cost of ownership
```

---

# PART XXVI — SECURITY MODEL

## 80. ECR Security

Common AWS controls:

```text
IAM
resource policies
CloudTrail
VPC endpoints where appropriate
AWS security services
encryption
```

---

## 81. Artifactory Security

Common controls:

```text
RBAC
permission targets
access tokens
TLS
audit
repository policies
security scanning
```

---

## 82. Network Security

Both can be secured with:

```text
TLS
private networking
restricted egress
firewalls
DNS
network segmentation
```

---

# PART XXVII — MULTI-ACCOUNT / MULTI-TEAM DESIGN

## 83. ECR Centralized Model

```text
                 Shared ECR
                /    |     \
               /     |      \
             EKS A  EKS B   EKS C
```

---

## 84. ECR Per-Account Model

```text
Account A
  |
  +--> ECR A

Account B
  |
  +--> ECR B
```

This improves account isolation but may increase duplication.

---

## 85. Artifactory Centralized Model

```text
              Artifactory
              /    |     \
          Team A Team B Team C
```

---

## 86. Central Registry Trade-Off

Advantages:

```text
central governance
shared caching
common policies
```

Risks:

```text
central dependency
blast radius
network dependency
```

Use HA/DR and strong isolation.

---

# PART XXVIII — MIGRATION: ECR TO ARTIFACTORY

## 87. Migration Flow

```text
ECR
 ↓
Inventory
 ↓
Select repositories
 ↓
Copy images
 ↓
Validate digests
 ↓
Update manifests
 ↓
Test
 ↓
Cutover
```

---

## 88. Migration Validation

Compare:

```text
repository
image
tag
digest
architecture
metadata
```

---

## 89. Dual-Publish Strategy

During transition:

```text
CI
 |
 +--> ECR
 |
 +--> Artifactory
```

Use this only temporarily and ensure both destinations are governed.

---

# PART XXIX — MIGRATION: ARTIFACTORY TO ECR

## 90. Migration Flow

```text
Artifactory
 ↓
Inventory
 ↓
Select images
 ↓
Copy
 ↓
ECR
 ↓
Validate digest
 ↓
Update deployments
```

---

## 91. Migration Risk

Do not assume:

```text
same tag
=
same image
```

Validate digests.

---

# PART XXX — HYBRID ARCHITECTURE

## 92. ECR + Artifactory

Some organizations use both.

Example:

```text
                 Artifactory
               /      |       \
           Maven     NPM      Docker
                              |
                              v
                         Promotion
                              |
                              v
                             ECR
                              |
                              v
                             EKS
```

---

## 93. Why Hybrid?

Possible reasons:

```text
universal artifact management
+
AWS-native runtime registry
```

---

## 94. Hybrid Complexity

Now there are:

```text
two registries
two authentication models
two lifecycle systems
two audit paths
```

Use hybrid only when the benefits justify the operational complexity.

---

# PART XXXI — PRODUCTION DECISION FRAMEWORK

## 95. Choose ECR If

Most answers are:

```text
AWS-centric
container-only
EKS/ECS heavy
prefer managed AWS service
minimal registry platform operations
```

---

## 96. Choose Artifactory If

Most answers are:

```text
multi-cloud
multi-package
universal artifact repository
central enterprise governance
remote/virtual repository requirements
strong Build Info requirements
```

---

## 97. Choose Both If

There is a clear architectural reason:

```text
Artifactory = enterprise artifact platform
ECR = AWS runtime registry
```

and the organization accepts the additional complexity.

---

# PART XXXII — REAL-WORLD SCENARIOS

## 98. Scenario — AWS-Only Startup

Architecture:

```text
GitHub Actions
 ↓
AWS OIDC
 ↓
ECR
 ↓
EKS
```

Likely choice:

```text
ECR
```

because it minimizes platform complexity.

---

## 99. Scenario — Global Enterprise

Environment:

```text
AWS
Azure
GCP
on-prem
```

Artifacts:

```text
Maven
NPM
PyPI
Docker
Helm
```

Likely choice:

```text
Artifactory
```

or a carefully designed universal artifact platform.

---

## 100. Scenario — Company Already Has Artifactory

If Artifactory already governs:

```text
Maven
NPM
PyPI
Docker
Helm
```

adding ECR only for convenience may create unnecessary duplication.

Evaluate:

```text
ECR-specific benefits
vs
existing Artifactory investment
```

---

## 101. Scenario — EKS Needs Fast AWS-Native Pulls

If the platform is strongly AWS-centric:

```text
EKS
 ↓
ECR
```

can simplify:

```text
identity
networking
operations
```

---

## 102. Scenario — Multiple Cloud Kubernetes Clusters

```text
Artifactory
 /    |    \
EKS  AKS   GKE
```

This can provide a common artifact source.

---

## 103. Scenario — Security Requires Central Artifact Governance

If security requires:

```text
central policy
approved external dependencies
common artifact lifecycle
cross-team audit
```

a universal repository may be preferable.

---

## 104. Scenario — Registry Outage

For either system:

```text
registry outage
 ↓
new image pulls fail
```

Existing Pods may continue running if their images are already local.

Therefore:

```text
HA
replication
caching
DR
```

matter.

---

# PART XXXIII — INTERVIEW PREPARATION

## 105. What Is the Main Difference Between ECR and Artifactory?

Answer:

```text
ECR is an AWS-native managed container registry, while Artifactory is
a broader enterprise artifact platform supporting containers and
multiple package ecosystems. I choose based on whether the
organization needs AWS-native simplicity or universal artifact
governance and multi-platform capabilities.
```

---

## 106. Which Is Better for EKS?

Answer:

```text
There is no universal answer. For an AWS-centric organization where
container images are the primary artifact, ECR provides strong native
integration. If the enterprise already uses Artifactory for multiple
package types or needs a common registry across multiple clouds,
Artifactory may be a better platform.
```

---

## 107. Why Would You Use Artifactory Instead of ECR?

Answer:

```text
I would consider Artifactory when the organization needs a universal
artifact repository, multiple package ecosystems, virtual and remote
repositories, centralized artifact governance, multi-cloud support
and strong Build Info and promotion capabilities.
```

---

## 108. Why Would You Use ECR Instead of Artifactory?

Answer:

```text
For a strongly AWS-centric platform focused primarily on container
images, ECR reduces external platform dependencies and integrates
naturally with AWS IAM, EKS and other AWS services.
```

---

## 109. Can ECR and Artifactory Coexist?

Answer:

```text
Yes. A hybrid architecture can use Artifactory as the enterprise
artifact platform and ECR as an AWS-native runtime registry. However,
I would only use this model when there is a clear business or
technical reason because it introduces additional synchronization,
authentication, lifecycle and audit complexity.
```

---

## 110. How Do You Secure ECR?

Answer:

```text
I use least-privilege IAM roles, repository policies where required,
OIDC for CI where supported, private networking where appropriate,
image scanning, immutable release practices and CloudTrail/audit
controls.
```

---

## 111. How Do You Secure Artifactory?

Answer:

```text
I use dedicated service identities, least-privilege permission
targets, scoped access tokens, TLS, repository isolation, image
scanning, audit logging and controlled network access.
```

---

## 112. Which Is Better for Multi-Cloud?

Answer:

```text
If the requirement is one common artifact platform across AWS, Azure,
GCP and on-prem environments, Artifactory is often a stronger fit.
ECR can still be used in each AWS environment, but it is AWS-specific
and requires additional architecture for a common multi-cloud model.
```

---

## 113. How Do You Migrate ECR to Artifactory?

Answer:

```text
I inventory repositories and images, copy the images, verify tags and
digests, test pulls from Artifactory, update Kubernetes and CI
references, perform a controlled cutover and retain the old registry
until rollback and validation requirements are satisfied.
```

---

## 114. How Do You Compare the Cost?

Answer:

```text
I compare total cost of ownership rather than only storage price. For
ECR I consider storage, transfer and AWS service usage. For
Artifactory I also consider licensing, infrastructure, storage,
database, HA, backup and platform engineering operations.
```

---

# PART XXXIV — PRODUCTION CHECKLIST

## 115. Requirements

```text
[ ] container-only or universal artifacts?
[ ] AWS-only or multi-cloud?
[ ] EKS/ECS?
[ ] package ecosystems?
[ ] compliance requirements?
[ ] Build Info requirement?
[ ] remote/virtual repositories?
```

---

## 116. ECR

```text
[ ] IAM roles
[ ] least privilege
[ ] repository policy
[ ] lifecycle policy
[ ] scanning
[ ] replication
[ ] CloudTrail
[ ] network design
```

---

## 117. Artifactory

```text
[ ] repository design
[ ] RBAC
[ ] permission targets
[ ] access tokens
[ ] virtual repositories
[ ] remote repositories
[ ] Build Info
[ ] HA
[ ] backup/DR
```

---

## 118. Kubernetes

```text
[ ] image source
[ ] digest
[ ] runtime authentication
[ ] network path
[ ] DNS
[ ] TLS
[ ] rollback
```

---

## 119. CI/CD

```text
[ ] OIDC/federation
[ ] protected secrets
[ ] build once
[ ] immutable artifacts
[ ] scan
[ ] promotion
[ ] audit
```

---

# PART XXXV — GOLDEN RULES

## 120. Rules

```text
1. ECR is an AWS-native container registry.

2. Artifactory is a broader universal artifact platform.

3. Do not choose based only on popularity.

4. Start with workload and governance requirements.

5. Use ECR when AWS-native container registry simplicity is the main
   requirement.

6. Use Artifactory when universal artifact management is a core
   requirement.

7. Use Artifactory for multi-cloud standardization when that benefit
   justifies its platform cost.

8. Use both only when there is a clear architectural reason.

9. Do not maintain duplicate registries without a defined ownership
   and synchronization strategy.

10. Use least-privilege identities regardless of registry.

11. Prefer short-lived/federated credentials where supported.

12. Never use administrator credentials for normal CI.

13. Use immutable image versions.

14. Track image digests.

15. Scan images before production promotion.

16. Retain rollback versions.

17. Test registry disaster recovery.

18. Compare total cost of ownership, not just storage cost.

19. For EKS, evaluate native AWS integration versus enterprise
    artifact requirements.

20. For multi-cloud, evaluate network latency, availability and
    centralized governance.

21. Separate CI push permissions from Kubernetes runtime pull
    permissions.

22. Keep audit trails from Git to CI to registry to deployment.

23. Validate actual product features against current AWS and JFrog
    versions before final architecture decisions.

24. The correct registry is the one that satisfies security,
    reliability, operational and business requirements with acceptable
    complexity.
```

---

# END OF 20-ECR-vs-Artifactory.md
