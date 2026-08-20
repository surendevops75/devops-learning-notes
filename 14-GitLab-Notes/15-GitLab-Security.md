# GitLab Security

> Production-oriented GitLab security guide covering identity, authentication, authorization, repository protection, branch and merge-request controls, CI/CD security, runners, variables, tokens, OIDC, supply-chain security, artifacts, registries, webhooks, APIs, auditability, compliance, incident response, and senior DevOps/DevSecOps interview scenarios.

---

## 1. GitLab Security Overview

GitLab security must protect:

```text
Source code
CI/CD pipelines
Credentials
Artifacts
Container images
Infrastructure code
Production environments
Users
Integrations
```

---

## 2. GitLab Security Layers

A practical model:

```text
Identity
 ↓
Access Control
 ↓
Repository Security
 ↓
CI/CD Security
 ↓
Artifact Security
 ↓
Infrastructure Security
 ↓
Monitoring/Audit
 ↓
Incident Response
```

---

## 3. Authentication vs Authorization

### Authentication

Answers:

```text
Who are you?
```

### Authorization

Answers:

```text
What are you allowed to do?
```

Both must be configured correctly.

---

## 4. Identity Security

Use organizational identity controls such as:

```text
SSO
MFA
centralized user lifecycle
strong password policy
```

where supported by the GitLab edition/setup.

---

## 5. MFA

Multi-factor authentication adds another verification factor.

Typical:

```text
Password
+
Authenticator/security factor
```

MFA reduces account takeover risk.

---

## 6. SSO

Single Sign-On allows users to authenticate through a centralized identity provider.

Benefits:

```text
central access management
offboarding
MFA
group management
audit
```

---

## 7. SSO Offboarding

When an employee leaves:

```text
Disable identity-provider account
 ↓
GitLab access removed
```

This is safer than manually tracking accounts across systems.

---

## 8. GitLab User Lifecycle

A mature lifecycle:

```text
Join
 ↓
Provision
 ↓
Assign group/project access
 ↓
Review
 ↓
Change role
 ↓
Offboard
```

---

## 9. Least Privilege

Grant only the permissions required for the user's job.

Avoid:

```text
Everyone = Owner
```

---

## 10. GitLab Roles

GitLab provides project/group roles with different levels of permissions.

Typical roles include:

```text
Guest
Reporter
Developer
Maintainer
Owner
```

Exact capabilities depend on GitLab version/edition.

---

## 11. Developer Role

A developer generally needs to:

```text
clone
push
create branches
create merge requests
run pipelines
```

without necessarily managing project security settings.

---

## 12. Maintainer Role

Maintainers typically manage much more of the project, including CI/CD and project configuration.

Do not grant this role unnecessarily.

---

## 13. Owner Role

Owner-level access should be highly restricted because it can provide broad administrative control over groups/projects.

---

## 14. Group-Level Access

Group membership can provide access across multiple projects.

Use groups to standardize:

```text
team access
permissions
policies
```

---

## 15. Project-Level Access

Use project-level permissions for narrower access.

Example:

```text
Developer → application project
Platform → infrastructure project
Security → security project
```

---

## 16. Protected Branches

Protect critical branches such as:

```text
main
production
release/*
```

Protection should prevent unauthorized direct changes.

---

## 17. Why Protect Main?

Without protection:

```text
Developer
 ↓
push directly
 ↓
production branch
```

With protection:

```text
Developer
 ↓
Merge Request
 ↓
Review
 ↓
Pipeline
 ↓
Merge
```

---

## 18. Direct Push Restriction

For production branches, restrict who can push directly.

Prefer:

```text
Merge Request
+
approval
+
passing pipeline
```

---

## 19. Merge Request Security

Review:

```text
application code
Dockerfile
.gitlab-ci.yml
Terraform
Kubernetes manifests
dependencies
secrets
```

A CI configuration change can be as sensitive as application code.

---

## 20. CODEOWNERS

CODEOWNERS can define required reviewers for important paths.

Example:

```text
.gitlab-ci.yml        @platform-team
terraform/            @cloud-team
k8s/prod/             @platform-team
security/             @security-team
```

---

## 21. Why CODEOWNERS Matters

It ensures specialized changes reach the appropriate reviewers.

Examples:

```text
IAM change → cloud/security
production manifests → platform
pipeline change → DevOps
```

---

## 22. Merge Approval Rules

High-risk changes can require additional approvals.

Examples:

```text
Production deployment
IAM changes
Security policy changes
Runner changes
```

---

## 23. Approval Separation

For sensitive environments:

```text
Developer creates MR
 ↓
Application review
 ↓
Security/platform review
 ↓
Production approval
```

This creates separation of duties.

---

## 24. Pipeline Success Requirement

Require CI to pass before merging important branches.

Example:

```text
Tests
SAST
SCA
Secret Scan
Container Scan
IaC Scan
```

---

## 25. Security Gate

A security gate should define:

```text
what is blocked
what is allowed
who can override
how exceptions are documented
```

---

## 26. Security Override

A security gate override should not be an informal:

```text
just click skip
```

Instead require:

```text
reason
owner
approval
expiry
```

where organizational policy requires it.

---

## 27. CI/CD as Attack Surface

`.gitlab-ci.yml` executes commands.

Therefore:

```text
CI configuration
=
executable infrastructure
```

Treat it as highly sensitive.

---

## 28. Pipeline Script Injection

Dangerous pattern:

```bash
sh -c "deploy $VARIABLE"
```

when the variable can be controlled by untrusted input.

Validate and safely handle inputs.

---

## 29. Merge Request Variables

Do not blindly trust:

```text
MR title
branch name
commit message
user input
```

inside shell commands.

---

## 30. Untrusted Forks

Pipelines from forks may run code you do not control.

Be extremely careful about exposing:

```text
production secrets
cloud credentials
protected variables
```

to untrusted fork pipelines.

---

## 31. Protected Variables

Sensitive variables should be protected so they are available only to authorized protected refs/environments according to GitLab configuration.

---

## 32. Masked Variables

Mask secrets in CI logs where supported.

However:

```text
masked
≠
secure if exposed to malicious script
```

A job that can read a secret can potentially exfiltrate it.

---

## 33. Environment-Scoped Variables

Use different credentials/configuration for:

```text
dev
stage
prod
```

This reduces accidental cross-environment access.

---

## 34. Never Echo Secrets

Avoid:

```bash
echo "$AWS_SECRET_ACCESS_KEY"
```

Even if the variable is masked, do not intentionally print secrets.

---

## 35. Debug Logging Risk

Avoid enabling verbose debugging around sensitive commands.

For example, shell tracing can expose command arguments.

---

## 36. Shell Tracing

Be careful with:

```bash
set -x
```

when commands contain credentials or tokens.

---

## 37. CI Token

GitLab provides job-related credentials/tokens for automation.

Treat them as secrets.

Use the minimum scope necessary.

---

## 38. Job Token Permissions

Where GitLab configuration allows it, restrict what CI job tokens can access.

Avoid unnecessarily broad cross-project access.

---

## 39. Personal Access Tokens

Personal access tokens should not be the default identity for automation.

Problems include:

```text
employee leaves
token remains
token has excessive scope
rotation is forgotten
```

Prefer machine identities or job/OIDC mechanisms where appropriate.

---

## 40. Token Scopes

Grant only required scopes.

Avoid broad scopes such as full API access when a narrower capability is enough.

---

## 41. Token Expiration

Use expiration dates for tokens where possible.

Long-lived credentials increase exposure.

---

## 42. Token Rotation

Rotation process:

```text
Create new
 ↓
Test
 ↓
Switch automation
 ↓
Revoke old
```

---

## 43. Deploy Tokens

Deploy tokens can be useful for machine access to repositories/packages depending on the use case.

Limit:

```text
scope
projects
expiration
```

---

## 44. Deploy Keys

SSH deploy keys can provide repository access.

Use read-only access where write access is not required.

---

## 45. Read-Only Git Access

ArgoCD generally only needs to read GitOps repositories.

Therefore prefer:

```text
read-only repository credential
```

when the architecture permits it.

---

## 46. CI Write Access

A GitLab job updating a GitOps repository may need write permission.

That permission should be limited to:

```text
specific repository
specific branch/workflow
```

where possible.

---

## 47. OIDC

OIDC provides short-lived identity tokens.

Concept:

```text
GitLab Job
 ↓
OIDC
 ↓
Cloud STS
 ↓
Temporary credentials
```

---

## 48. GitLab OIDC + AWS

For AWS:

```text
GitLab
 ↓
OIDC token
 ↓
AWS STS AssumeRoleWithWebIdentity
 ↓
Temporary IAM credentials
```

This avoids storing static AWS keys.

---

## 49. OIDC Trust Policy

AWS IAM should restrict who can assume the role.

Possible conditions can include:

```text
GitLab project
branch/ref
environment
audience
```

Use exact claims supported by the configured integration.

---

## 50. Separate AWS Roles

Use separate roles:

```text
Terraform Dev
Terraform Stage
Terraform Prod
```

rather than one unrestricted role.

---

## 51. Deployment Role

If CI performs AWS deployment operations, use a dedicated deployment role.

Avoid:

```text
AdministratorAccess
```

for normal pipelines.

---

## 52. Terraform Role

Terraform should use a role appropriate to the resources it manages.

Separate:

```text
infrastructure provisioning
application deployment
```

where practical.

---

## 53. Break-Glass AWS Access

Emergency administrative access should be:

```text
restricted
audited
temporary
strongly authenticated
```

---

## 54. Runner Security

GitLab runners execute CI jobs.

A malicious job can potentially access:

```text
filesystem
network
environment variables
credentials
Docker socket
```

depending on runner architecture.

---

## 55. Shared Runner Risk

Shared runners can increase cross-project risk if isolation is inadequate.

Production-sensitive workloads should use trusted runner architecture.

---

## 56. Protected Runner

Restrict production deployment jobs to approved runners where appropriate.

---

## 57. Ephemeral Runners

Ephemeral runners reduce persistence between jobs.

Concept:

```text
Job
 ↓
Temporary runner
 ↓
Destroy
```

---

## 58. Runner Executor

Common execution models include:

```text
Docker
Kubernetes
Shell
```

Each has different security characteristics.

---

## 59. Shell Runner Risk

Shell runners execute commands directly on the host.

A malicious pipeline can potentially compromise the host.

Use only where the trust model is appropriate.

---

## 60. Docker Socket Risk

Mounting:

```text
/var/run/docker.sock
```

can provide powerful access to the Docker host.

Avoid it when possible.

---

## 61. Docker-in-Docker

Docker-in-Docker can be used for container builds, but runner isolation and privilege requirements must be carefully considered.

---

## 62. Rootless Builds

Where supported by the toolchain, rootless/containerized build approaches can reduce privilege requirements.

---

## 63. Runner Network Security

Restrict runner network access to required destinations:

```text
GitLab
ECR
Artifactory
AWS APIs
approved dependencies
```

---

## 64. Runner Egress

Unrestricted outbound network access increases the ability of malicious CI code to exfiltrate data.

Use egress controls where practical.

---

## 65. Runner Patching

Keep runner:

```text
OS
executor
container runtime
GitLab Runner
```

patched.

---

## 66. Runner Registration Tokens/Credentials

Protect runner registration credentials.

Compromised runner registration can allow unauthorized job execution.

---

## 67. Runner Tags

Use tags to route sensitive jobs:

```yaml
tags:
  - production
  - trusted
```

Tags are routing controls, not a complete security boundary.

---

## 68. Artifact Security

CI artifacts can contain:

```text
build outputs
test reports
logs
SBOM
configuration
```

Protect artifacts that contain sensitive information.

---

## 69. Artifact Exposure

Do not upload:

```text
.env
private keys
AWS credentials
production dumps
```

as public or broadly accessible artifacts.

---

## 70. Artifact Expiration

Use appropriate artifact retention.

Reduce unnecessary exposure of sensitive historical artifacts.

---

## 71. Cache Security

Caches may contain:

```text
dependencies
build files
credentials accidentally written to disk
```

Do not cache sensitive files.

---

## 72. Cache Poisoning

A malicious or compromised pipeline can potentially influence cached content.

Use trusted cache keys and isolation appropriate to the project.

---

## 73. Dependency Cache

Cache package dependencies for speed, but verify package integrity through lock files and trusted registries.

---

## 74. Container Registry Security

GitLab Container Registry or ECR should have:

```text
access controls
retention
scanning
audit
```

---

## 75. Registry Push Permissions

Only trusted pipelines/users should push production images.

---

## 76. Registry Delete Permissions

Delete permissions should be restricted.

Accidental image deletion can break rollback.

---

## 77. Immutable Tags

If supported by the registry workflow, prevent overwriting production tags.

Prefer:

```text
commit SHA
release version
digest
```

---

## 78. `latest` Tag

Avoid relying on:

```text
latest
```

for production deployment.

It hides version changes and weakens rollback traceability.

---

## 79. Image Promotion

Promote:

```text
same digest
```

rather than rebuilding for each environment.

---

## 80. GitLab Package Registry

Package registries may contain:

```text
Maven
npm
PyPI
```

artifacts.

Restrict package publication and consumption appropriately.

---

## 81. Package Registry Access

Use:

```text
read-only credentials
```

for consumers that do not need publication access.

---

## 82. Dependency Proxy

A dependency proxy can help control and cache external container dependencies.

It can also reduce repeated external pulls.

---

## 83. External Dependency Risk

External package registries can become:

```text
availability risk
supply-chain risk
malicious package risk
```

Use trusted sources and pin versions.

---

## 84. Package Locking

Lock dependencies to reproducible versions.

This helps ensure:

```text
CI today
```

uses the same dependency versions as:

```text
CI tomorrow
```

unless intentionally updated.

---

## 85. Dependency Update Automation

Automated dependency updates should still run:

```text
tests
security scans
compatibility checks
```

before merging.

---

## 86. Dependency Review

For a new package, review:

```text
maintainer
source
license
activity
known vulnerabilities
transitive dependencies
```

---

## 87. Supply Chain Attack

A supply-chain attack can target:

```text
source
dependency
CI template
runner
container
registry
GitOps repository
```

---

## 88. Software Supply Chain

```text
Developer
 ↓
Git
 ↓
CI
 ↓
Dependencies
 ↓
Build
 ↓
Artifact
 ↓
Registry
 ↓
GitOps
 ↓
Cluster
```

Every stage needs security controls.

---

## 89. SLSA Concepts

Supply-chain security frameworks such as SLSA focus on:

```text
provenance
build integrity
artifact traceability
builder trust
```

Apply only the controls appropriate to organizational maturity.

---

## 90. Build Provenance

Record:

```text
source revision
builder
build time
dependencies
artifact digest
```

---

## 91. Artifact Signing

Signing can provide evidence that an artifact was produced by a trusted process.

The deployment platform/admission layer must also verify the signature for enforcement.

---

## 92. Cosign

Cosign is commonly used in OCI image signing workflows.

Concept:

```text
Build
 ↓
Sign
 ↓
Registry
 ↓
Verify
 ↓
Deploy
```

---

## 93. Admission Verification

A Kubernetes admission control can enforce trusted images.

Concept:

```text
Pod request
 ↓
Admission policy
 ↓
Signature verified?
 ↓
Allow / Reject
```

---

## 94. Git Commit Signing

Commit signing can improve confidence about author identity and repository integrity.

It does not replace branch protection or CI security.

---

## 95. GitLab Auditability

Track:

```text
login
permission changes
repository changes
pipeline activity
deployments
```

using available GitLab audit capabilities.

---

## 96. Audit Logs

Audit records help answer:

```text
Who changed it?
When?
What changed?
From where?
```

---

## 97. Audit Retention

Retention should satisfy:

```text
incident response
security investigations
compliance
```

requirements.

---

## 98. Log Access

Restrict access to audit/security logs.

Logs may contain sensitive operational information.

---

## 99. Webhook Security

GitLab webhooks can notify:

```text
ArgoCD
Slack
security systems
external automation
```

Protect webhook secrets and endpoints.

---

## 100. Webhook Secret

Validate webhook authenticity using the configured secret/signature mechanism.

Never treat any incoming webhook as trusted merely because it claims to come from GitLab.

---

## 101. Webhook Endpoint

Protect the endpoint with:

```text
TLS
authentication/signature validation
rate limiting
network controls
logging
```

---

## 102. API Security

GitLab APIs should use:

```text
least-privilege tokens
HTTPS
rate controls
access control
```

---

## 103. API Token Rotation

Use:

```text
short-lived
scoped
rotatable
```

credentials wherever practical.

---

## 104. GitLab API Automation

Automation should use a dedicated identity where possible.

Avoid tying critical automation to an individual's account.

---

## 105. API Abuse

Monitor for:

```text
unusual token usage
mass repository access
unexpected project changes
```

---

## 106. GitLab Integrations

Third-party integrations can receive access to:

```text
repositories
webhooks
pipelines
issues
```

Review integrations periodically.

---

## 107. OAuth Applications

Review OAuth applications and remove unnecessary access.

A compromised OAuth application can become an access path into GitLab.

---

## 108. Group Access Tokens

Group-level tokens can affect multiple projects.

Use only when necessary and restrict scopes/expiration.

---

## 109. Project Access Tokens

Project access tokens are useful for project-scoped automation.

Prefer them over personal tokens when appropriate.

---

## 110. Token Ownership

Every machine credential should have:

```text
purpose
owner
scope
expiration
rotation process
```

---

## 111. Credential Inventory

Maintain an inventory of:

```text
PATs
deploy tokens
deploy keys
project tokens
group tokens
cloud roles
webhook secrets
runner credentials
```

---

## 112. Credential Sprawl

Credential sprawl occurs when many unmanaged credentials exist.

Reduce it through:

```text
OIDC
central identity
short-lived tokens
machine identities
rotation
```

---

## 113. Protected Environments

Production deployment jobs should target protected environments where supported.

This helps restrict who can trigger sensitive deployments.

---

## 114. Environment Deployment Permissions

Define:

```text
who can deploy dev
who can deploy stage
who can deploy prod
```

---

## 115. Manual Production Jobs

A manual production job should still require:

```text
approved source
passed security gates
authorized user
controlled runner
```

Manual does not mean insecure.

---

## 116. Deployment Approval

For critical workloads:

```text
CI
 ↓
Security checks
 ↓
Approval
 ↓
Production deployment
```

---

## 117. Production Branch + Environment

Protect both:

```text
Git branch
+
deployment environment
```

Defense in depth is stronger than either alone.

---

## 118. GitLab Security Architecture

```text
Users
 │
 ▼
SSO/MFA
 │
 ▼
Groups
 │
 ▼
Projects
 │
 ├── Protected branches
 ├── MR approvals
 ├── CODEOWNERS
 └── Protected environments
        │
        ▼
       CI
        │
 ├── Secure variables
 ├── Trusted runners
 └── OIDC
        │
        ▼
      AWS/ECR
```

---

## 119. GitLab + ArgoCD Security Boundary

Recommended:

```text
GitLab
 ↓
GitOps repository
 ↓
ArgoCD
 ↓
EKS
```

GitLab does not need broad persistent Kubernetes credentials just to trigger GitOps.

---

## 120. GitOps Repository Protection

Production GitOps repositories should have:

```text
protected branches
CODEOWNERS
MR approvals
security scanning
restricted write access
auditability
```

---

## 121. GitOps Malicious Commit

If an attacker changes:

```yaml
image: malicious/image
```

ArgoCD may deploy it if GitOps controls are bypassed.

Therefore GitOps repository protection is a production security boundary.

---

## 122. GitOps Review

Review changes to:

```text
image
RBAC
ServiceAccount
Ingress
NetworkPolicy
Secrets references
resources
```

---

## 123. Dangerous Kubernetes Manifest

Example:

```yaml
securityContext:
  privileged: true
```

This should trigger security review.

---

## 124. Dangerous RBAC

Example:

```yaml
verbs:
  - "*"
resources:
  - "*"
```

Avoid wildcard permissions unless there is a documented requirement.

---

## 125. Dangerous ServiceAccount

An application should not receive cluster-admin access simply because it needs access to one AWS resource.

Use AWS workload identity with least privilege.

---

## 126. Dangerous NetworkPolicy

A policy allowing:

```text
0.0.0.0/0
```

or all namespace communication may defeat isolation goals.

---

## 127. Public Exposure

Review:

```text
LoadBalancer
Ingress
Security Groups
WAF
DNS
```

before exposing services publicly.

---

## 128. AWS Security Group

Avoid unnecessarily broad:

```text
0.0.0.0/0
```

especially for administrative ports such as SSH.

---

## 129. SSH Security

Prefer:

```text
SSM
bastion controls
private networking
VPN
```

over exposing SSH publicly where possible.

---

## 130. S3 Security

Review:

```text
public access
bucket policy
encryption
versioning
logging
```

---

## 131. RDS Security

Review:

```text
private subnet
security group
encryption
backup
credentials
```

---

## 132. EKS API Security

Consider:

```text
private endpoint
restricted public endpoint
authorized networks
RBAC
identity provider
```

according to requirements.

---

## 133. Kubernetes API Access

Do not give every engineer unrestricted access to:

```text
secrets
pods/exec
cluster roles
nodes
```

---

## 134. `kubectl exec` Risk

`pods/exec` can provide shell access inside workloads.

Restrict it in production according to operational requirements.

---

## 135. Kubernetes Secret Access

The permission:

```text
get secrets
```

can expose credentials.

Treat it as a sensitive permission.

---

## 136. ClusterRole vs Role

### Role

Namespace-scoped.

### ClusterRole

Can define cluster-wide permissions.

Use the narrowest scope possible.

---

## 137. RoleBinding vs ClusterRoleBinding

Prefer namespace-scoped bindings when cluster-wide access is unnecessary.

---

## 138. Default Service Account

Do not assume the default service account is appropriate for production applications.

Create dedicated service accounts.

---

## 139. Pod Identity Principle

Application permissions should be:

```text
application-specific
minimum required
short-lived where possible
```

---

## 140. Secrets in CI

Use GitLab protected/masked variables or external secret systems for CI secrets.

Prefer OIDC for cloud authentication where available.

---

## 141. Secrets in Docker Build

Build-time secrets should not be embedded into final image layers.

Use secure build mechanisms supported by the build system.

---

## 142. Secrets in Logs

Check logs for:

```text
tokens
passwords
connection strings
authorization headers
```

Redact sensitive values.

---

## 143. Security Testing Environment

Do not run destructive security tests against production unless specifically authorized.

Use:

```text
dev
test
staging
ephemeral environment
```

for aggressive tests.

---

## 144. Ephemeral Security Environment

A pipeline can create:

```text
temporary environment
 ↓
deploy
 ↓
DAST
 ↓
destroy
```

This provides safer application security testing.

---

## 145. Security Scan Artifacts

Store reports such as:

```text
SAST report
dependency report
Trivy report
SBOM
DAST report
```

according to retention policy.

---

## 146. Security Report Integrity

A report should identify:

```text
application
commit
image digest
scanner
scanner version
timestamp
```

where possible.

---

## 147. Scan Reproducibility

To reproduce a finding, preserve:

```text
source revision
image digest
scanner version
configuration
```

---

## 148. Security Tool Drift

A pipeline may change behavior when scanner versions change.

Control versions of:

```text
scanner
CI template
policy bundle
```

where reproducibility matters.

---

## 149. Security Baseline Updates

Security baselines must evolve when:

```text
new CVEs
new attack techniques
new AWS services
new Kubernetes versions
```

appear.

---

## 150. Vulnerability Intelligence

Security teams should correlate scanner results with:

```text
active exploitation
asset exposure
business criticality
```

rather than relying only on CVSS.

---

## 151. Internet-Facing Priority

A vulnerability on:

```text
public ALB
```

may deserve faster remediation than the same vulnerability on:

```text
isolated internal service
```

depending on exploitability.

---

## 152. Critical Business Service

For a critical service:

```text
payment
authentication
orders
```

security findings may have higher operational priority.

---

## 153. Risk-Based Security

The goal is not:

```text
zero scanner findings
```

The goal is:

```text
acceptable and controlled risk
```

---

## 154. Security Debt Review

Schedule periodic review of:

```text
open findings
exceptions
expired exceptions
old dependencies
old images
```

---

## 155. Security Exception Expiry

When an exception expires:

```text
Reassess
 ↓
Fix
or
renew with approval
```

Do not automatically renew forever.

---

## 156. Security Incident Classification

Classify incidents by:

```text
credential compromise
supply-chain attack
vulnerability exploitation
unauthorized deployment
repository compromise
runner compromise
```

---

## 157. GitLab Account Compromise

Response:

```text
Disable account/session
 ↓
Reset credentials
 ↓
Revoke tokens
 ↓
Review recent activity
 ↓
Check repositories/pipelines
 ↓
Check cloud activity
```

---

## 158. Compromised Personal Token

Immediately:

```text
revoke
rotate dependent credentials
review API activity
identify affected resources
```

---

## 159. Compromised Project Token

Scope the investigation to:

```text
project
repository
packages
pipelines
deployments
```

Then rotate/revoke.

---

## 160. Compromised Runner

Investigate:

```text
jobs executed
variables accessed
network connections
artifacts
cloud API calls
```

Rebuild/replace the runner from a trusted image.

---

## 161. Compromised ECR Credentials

Rotate or invalidate the affected identity and inspect:

```text
pushes
deletes
image changes
repository policy changes
```

---

## 162. Unauthorized GitOps Change

Check:

```text
Git commit author
MR
approval
pipeline
ArgoCD sync
```

Then restore known-good desired state.

---

## 163. Unauthorized Production Deployment

Correlate:

```text
GitLab
ArgoCD
Kubernetes
AWS
```

timestamps and identities.

---

## 164. Security Timeline

Example:

```text
09:01 token created
09:03 suspicious Git push
09:04 pipeline started
09:05 ECR image pushed
09:06 GitOps changed
09:07 ArgoCD synced
```

A timeline helps establish blast radius.

---

## 165. Security Post-Incident Actions

After containment:

```text
Root cause
 ↓
Control gap
 ↓
New guardrail
 ↓
Test
 ↓
Document
```

---

## 166. Security Architecture Review

Periodically review:

```text
users
roles
tokens
runners
repositories
integrations
CI templates
AWS trust policies
ArgoCD access
```

---

## 167. Access Review

Perform periodic access reviews:

```text
Who has access?
Why?
Is it still needed?
Is the role excessive?
```

---

## 168. Dormant Access

Remove access that is:

```text
unused
expired
unjustified
```

---

## 169. Group Membership Review

Group membership can silently grant access to many projects.

Review it regularly.

---

## 170. External Collaborators

Review external users carefully.

Ensure they cannot access:

```text
production secrets
internal repositories
security configuration
```

unless explicitly required.

---

## 171. Repository Visibility

Choose appropriate visibility:

```text
Private
Internal
Public
```

based on information sensitivity.

---

## 172. Public Repository Risk

Before making a repository public, check:

```text
secrets
internal URLs
architecture details
credentials
CI configuration
dependencies
```

---

## 173. Secret Scanning Before Public Release

Run a full secret scan before publishing a repository.

---

## 174. README Security

Avoid documenting:

```text
real credentials
private IPs
sensitive endpoints
internal tokens
```

---

## 175. Sample Configuration

Use:

```text
.example
template
placeholder
```

files rather than real secrets.

---

## 176. GitLab Security Checklist — Identity

```text
[ ] SSO where appropriate
[ ] MFA
[ ] Least privilege
[ ] Access review
[ ] Offboarding
[ ] External user review
```

---

## 177. GitLab Security Checklist — Repository

```text
[ ] Protected branches
[ ] MR approvals
[ ] CODEOWNERS
[ ] Secret scanning
[ ] Restricted visibility
[ ] CI configuration review
```

---

## 178. GitLab Security Checklist — CI

```text
[ ] Protected variables
[ ] Masked secrets
[ ] OIDC
[ ] Trusted runners
[ ] Runner isolation
[ ] No shell injection
[ ] Limited job token access
```

---

## 179. GitLab Security Checklist — Artifacts

```text
[ ] Restricted artifacts
[ ] Retention policy
[ ] No secrets
[ ] Immutable production images
[ ] SBOM
[ ] Provenance
```

---

## 180. GitLab Security Checklist — AWS

```text
[ ] OIDC
[ ] IAM least privilege
[ ] Separate environment roles
[ ] No static cloud keys
[ ] Restricted ECR access
[ ] Auditing
```

---

## 181. GitLab Security Checklist — Kubernetes

```text
[ ] RBAC
[ ] NetworkPolicy
[ ] Pod security
[ ] Non-root
[ ] No privileged workloads
[ ] Restricted service accounts
[ ] Secret management
[ ] Image policy
```

---

## 182. GitLab Security Checklist — GitOps

```text
[ ] Protected GitOps repository
[ ] CODEOWNERS
[ ] ArgoCD RBAC
[ ] Project restrictions
[ ] Immutable images
[ ] Controlled sync
[ ] Controlled prune
```

---

## 183. GitLab Security Checklist — Incident Response

```text
[ ] Token revocation process
[ ] Credential rotation
[ ] Runner compromise process
[ ] GitOps rollback
[ ] Cloud audit
[ ] Timeline reconstruction
[ ] Postmortem
```

---

## 184. Senior Interview — How Do You Protect Production Branches?

> I protect the production branch, restrict direct pushes, require merge requests and appropriate approvals, require mandatory CI/security checks, use CODEOWNERS for sensitive paths, and protect the corresponding production deployment environment.

---

## 185. Senior Interview — Why Is `.gitlab-ci.yml` Sensitive?

> Because it is executable code. A malicious CI change can alter build behavior, access protected variables, assume cloud roles, push images, or modify deployment state. Therefore pipeline configuration needs the same review and protection as application code.

---

## 186. Senior Interview — Why Prefer OIDC Over Static AWS Keys?

> OIDC allows GitLab jobs to exchange a short-lived identity token for temporary AWS credentials through STS. It reduces long-lived credential storage and allows the IAM trust policy to restrict which GitLab identities can assume the role.

---

## 187. Senior Interview — How Do You Secure GitLab Runners?

> I use trusted/protected runners for production, isolate workloads, minimize host privileges, restrict network access, patch the runner infrastructure, avoid exposing Docker socket unnecessarily, and prevent untrusted pipelines from accessing production credentials.

---

## 188. Senior Interview — What Happens If a Pipeline Secret Is Exposed?

> I treat it as compromised, revoke or rotate it immediately, identify where it was exposed, investigate access logs, remove the source of exposure, and implement a preventive control. I don't rely only on masked logs because a malicious job can still use an accessible secret.

---

## 189. Senior Interview — How Do You Secure GitOps?

> I protect the GitOps repository with branch protection, CODEOWNERS and approvals, use immutable image digests, restrict ArgoCD RBAC and Projects, secure repository credentials, and limit production sync/delete permissions.

---

## 190. Senior Interview — How Do You Handle a Critical Vulnerability in an EKS Image?

> I identify the exact image digest and affected environments, determine exposure and exploitability, rebuild with the patched dependency/base image, scan it, publish the approved digest to ECR, update GitOps, sync through ArgoCD, and validate the workload.

---

## 191. Senior Interview — How Do You Secure Kubernetes Workloads?

> I use non-root containers, restricted capabilities, `allowPrivilegeEscalation: false`, appropriate seccomp, read-only filesystems where possible, RBAC, NetworkPolicies, dedicated service accounts, external secret management, resource controls and admission policies.

---

## 192. Senior Interview — How Do You Prevent Secret Leakage?

> I use secret detection in GitLab CI, protected/masked variables for CI secrets, AWS Secrets Manager or an approved external secret mechanism for runtime secrets, and never bake credentials into Docker images or Git repositories.

---

## 193. Senior Interview — How Do You Handle Security Exceptions?

> I document the finding, business and technical risk, owner, compensating controls, remediation plan and expiry date. Exceptions require appropriate approval and are reviewed again before expiration.

---

## 194. Senior Interview — How Do You Secure a Terraform Pipeline?

> I use GitLab protected branches and approvals, Terraform validation and plan review, IaC security scanning, least-privilege AWS roles through OIDC, protected production environments and controlled apply permissions.

---

## 195. Senior Interview — How Do You Prevent a Different Artifact From Being Deployed After Scanning?

> I identify the exact image digest during the scan and promote that same digest. The GitOps repository references the immutable digest, so production deploys the artifact that passed the security gate rather than rebuilding or retagging it.

---

## 196. Senior Interview — How Do You Secure Third-Party Dependencies?

> I use trusted registries, lock dependencies, run SCA, review new packages, use private artifact repositories where appropriate, scan transitive dependencies, and maintain a controlled dependency update process.

---

## 197. Senior Interview — What Is Defense in Depth?

> Defense in depth means using multiple independent security controls. For example, I don't rely only on Trivy; I combine source scanning, dependency scanning, secret detection, image scanning, Kubernetes admission policies, IAM least privilege and runtime monitoring.

---

## 198. Senior Interview — What Is the Most Important Security Principle?

> Least privilege combined with strong identity and layered controls. Every user, pipeline, workload and integration should receive only the access required for its purpose.

---

## 199. Senior Interview — How Would You Investigate Unauthorized Production Deployment?

> I correlate GitLab audit activity, merge requests, pipeline identity, GitOps commits, ArgoCD sync history, Kubernetes events and AWS audit logs. I establish the timeline, identify the compromised identity or process, restore known-good state, rotate credentials if necessary, and implement preventive controls.

---

## 200. Senior Interview — What Is Your GitLab Security Model?

> My model is protected identity, least privilege, protected repositories, mandatory review, secure CI, short-lived cloud credentials, trusted runners, immutable artifacts, GitOps protection, Kubernetes controls and continuous auditing. Security is enforced at multiple boundaries rather than through one scanner.

---

## 201. Production Security Architecture

```text
                         Identity Provider
                                │
                             SSO/MFA
                                │
                                ▼
                             GitLab
                                │
             ┌──────────────────┼──────────────────┐
             ▼                  ▼                  ▼
         Protected Repo       CI/CD             Access
             │                  │
             │        ┌─────────┼─────────┐
             │        ▼         ▼         ▼
             │      SAST       SCA      Secrets
             │        │         │         │
             │        └─────────┼─────────┘
             │                  ▼
             │               Build
             │                  │
             │                  ▼
             │               Trivy
             │                  │
             │                  ▼
             │                 ECR
             │                  │
             │             Image Digest
             │                  │
             ▼                  ▼
          GitOps Repo       AWS OIDC
             │                  │
             ▼                  ▼
           ArgoCD          AWS IAM/ST​S
             │
             ▼
            EKS
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
     RBAC  Network  Pod Security
             │
             ▼
          Workloads
             │
       ┌─────┼─────┐
       ▼     ▼     ▼
    Metrics  Logs  Alerts
```

---

## 202. End-to-End Security Flow

```text
1. Developer authenticates with SSO/MFA
2. Developer creates MR
3. Protected repository rules apply
4. CODEOWNERS requests appropriate review
5. CI starts on trusted runner
6. SAST runs
7. SCA runs
8. Secret detection runs
9. Tests run
10. Container is built
11. Trivy scans image
12. IaC is scanned
13. SBOM is generated
14. Security policy evaluates findings
15. Approved image is pushed to ECR
16. Immutable digest is recorded
17. GitOps change is reviewed
18. ArgoCD reconciles desired state
19. EKS admission policies evaluate workload
20. Kubernetes runs the application
21. Prometheus/Grafana/ELK monitor behavior
22. Security incidents trigger response
```

---

## 203. Final Security Mental Model

```text
             TRUST NOTHING
                  │
                  ▼
              VERIFY IDENTITY
                  │
                  ▼
             LEAST PRIVILEGE
                  │
                  ▼
            PROTECT SOURCE
                  │
                  ▼
             SECURE CI/CD
                  │
                  ▼
          SECURE ARTIFACTS
                  │
                  ▼
           SECURE GITOPS
                  │
                  ▼
          SECURE KUBERNETES
                  │
                  ▼
             SECURE AWS
                  │
                  ▼
             MONITOR
                  │
                  ▼
              RESPOND
                  │
                  ▼
             IMPROVE
```

> **Core principle:** GitLab security is not only about protecting the Git repository. A production DevOps security model must protect identity, source code, CI runners, credentials, artifacts, cloud access, GitOps repositories, Kubernetes workloads, and the audit trail connecting every deployment.

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
├── 15-GitLab-Security.md ✓
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `16-GitLab-Advanced-CI-CD.md`**
