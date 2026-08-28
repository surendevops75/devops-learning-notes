# Artifactory-Security

## 1. Purpose

This file provides a production-oriented security guide for JFrog
Artifactory.

The objective is to secure the complete artifact supply chain:

```text
Developer
   |
   v
Git
   |
   v
CI/CD
   |
   v
Artifactory
   |
   +--> Dependencies
   +--> Build Artifacts
   +--> Container Images
   +--> Helm Charts
   |
   v
Kubernetes / EKS
   |
   v
Production
```

Security must protect:

```text
identity
authentication
authorization
repositories
artifacts
dependencies
build pipelines
credentials
network
storage
audit logs
promotion
production deployments
```

---

# PART I — SECURITY FUNDAMENTALS

## 2. Why Artifactory Security Matters

Artifactory is part of the software supply chain.

If an attacker compromises it, they may be able to:

```text
steal packages
modify packages
publish malicious artifacts
replace dependencies
steal credentials
poison CI/CD
compromise production
```

Therefore registry security is not merely infrastructure security.

It is:

```text
Supply Chain Security
```

---

## 3. Security Layers

A mature design uses multiple layers:

```text
Identity
   |
Authentication
   |
Authorization
   |
Network
   |
Repository Controls
   |
Artifact Security
   |
CI/CD Security
   |
Audit
   |
Detection
   |
Response
```

---

## 4. Defense in Depth

Never depend on one control.

Example:

```text
TLS
+
MFA
+
least privilege
+
network restriction
+
artifact scanning
+
immutable releases
+
audit
```

---

# PART II — IDENTITY

## 5. Human vs Machine Identity

Separate:

```text
Human users
```

from:

```text
CI/CD service identities
```

and:

```text
Application/runtime identities
```

---

## 6. Human Identity

Examples:

```text
developer
platform engineer
security engineer
release manager
administrator
```

---

## 7. Machine Identity

Examples:

```text
svc-jenkins-payment
svc-github-payment
svc-gitlab-orders
```

---

## 8. Runtime Identity

Examples:

```text
EKS workload
Kubernetes node
deployment service
```

A runtime workload normally needs:

```text
READ
```

rather than:

```text
DEPLOY
DELETE
ADMIN
```

---

# PART III — LEAST PRIVILEGE

## 9. Principle

Give each identity only the permissions it needs.

Example:

```text
CI:
READ dependencies
DEPLOY application repository

Runtime:
READ production repository

Security:
READ + scan metadata

Admin:
platform administration
```

---

## 10. Bad Design

```text
All developers
      |
      v
Admin
      |
      v
Artifactory
```

This creates excessive blast radius.

---

## 11. Better Design

```text
Developer
   |
   v
READ

CI
   |
   v
READ + DEPLOY

Runtime
   |
   v
READ
```

---

# PART IV — RBAC

## 12. Role-Based Access Control

RBAC maps identities to permissions.

Concept:

```text
User
 ↓
Group
 ↓
Role
 ↓
Permission
```

---

## 13. Groups

Prefer group-based authorization over manually assigning large numbers
of permissions to individual users.

Example:

```text
artifactory-developers
artifactory-release-engineers
artifactory-security
artifactory-admins
```

---

## 14. Separation of Duties

Separate:

```text
development
release approval
security
administration
```

where required.

---

# PART V — PROJECTS AND REPOSITORY PERMISSIONS

## 15. Repository-Level Isolation

Example:

```text
payment-docker-local
orders-docker-local
shared-maven-virtual
```

---

## 16. Permission Targets

Use permission scopes that restrict:

```text
repositories
paths
users/groups
operations
```

---

## 17. Example

Payment CI:

```text
READ:
shared-maven-virtual

DEPLOY:
payment-maven-local
```

It should not automatically have:

```text
DEPLOY:
orders-maven-local
```

---

# PART VI — ADMINISTRATOR SECURITY

## 18. Admin Accounts

Administrative identities have very high blast radius.

Use them only for:

```text
configuration
user management
platform operations
security administration
```

---

## 19. Do Not Use Admin for CI

Never design:

```text
Jenkins
 ↓
Artifactory admin
```

Instead:

```text
Jenkins
 ↓
scoped service identity
```

---

## 20. Break-Glass Access

Maintain a controlled emergency administrative process.

Concept:

```text
Normal operations
 ↓
No admin

Emergency
 ↓
Break-glass
 ↓
Approval
 ↓
Audit
```

---

# PART VII — AUTHENTICATION

## 21. Authentication Methods

Depending on deployment and identity architecture, Artifactory can
support mechanisms such as:

```text
username/password
access tokens
SSO/federated identity
external identity providers
```

Prefer modern, scoped authentication mechanisms.

---

## 22. Password Security

For human users:

```text
strong password
MFA
SSO where appropriate
rotation according to policy
```

---

## 23. Access Tokens

Tokens are useful for:

```text
CI/CD
automation
service identities
API access
```

---

## 24. Token Scope

A token should be limited by:

```text
identity
permissions
repositories
operations
expiration
```

where supported by the deployment.

---

# PART VIII — TOKEN MANAGEMENT

## 25. Token Lifecycle

```text
Create
 ↓
Use
 ↓
Monitor
 ↓
Rotate
 ↓
Revoke
```

---

## 26. Token Expiration

Prefer:

```text
short-lived
or
appropriately bounded
```

credentials where the architecture permits.

---

## 27. Token Rotation

Production rotation:

```text
Create new token
 ↓
Update consumers
 ↓
Validate
 ↓
Revoke old token
 ↓
Audit
```

---

## 28. Leaked Token

If a token leaks:

```text
1. Revoke/rotate immediately
2. Identify identity
3. Review audit logs
4. Determine accessed repositories
5. Check artifact modifications
6. Check production deployments
7. Correct exposure
8. Document incident
```

---

# PART IX — MFA AND SSO

## 29. MFA

MFA should protect high-value human identities, especially:

```text
administrators
security users
release approvers
```

---

## 30. SSO

Enterprise SSO can centralize:

```text
authentication
MFA
user lifecycle
offboarding
identity governance
```

---

## 31. Offboarding

When an employee leaves:

```text
disable identity
 ↓
remove group membership
 ↓
revoke tokens
 ↓
review active sessions
 ↓
audit privileged access
```

---

# PART X — CI/CD SECURITY

## 32. CI Is a High-Risk Trust Boundary

CI can execute code.

If compromised, it may access:

```text
source
credentials
artifacts
cloud accounts
production
```

---

## 33. CI Identity

Use:

```text
dedicated service identity
```

with:

```text
minimum repository permissions
```

---

## 34. Pull Request Security

Do not expose production credentials to untrusted PR execution.

Especially protect against:

```text
fork pipelines
malicious pull requests
untrusted branches
```

---

## 35. Build Environment

Secure:

```text
runner
workspace
network
credentials
dependencies
toolchain
```

---

# PART XI — SECRET MANAGEMENT

## 36. Never Store Secrets in Git

Never commit:

```text
ARTIFACTORY_TOKEN
password
private key
cloud credential
```

---

## 37. Approved Secret Stores

Examples:

```text
GitHub Secrets
GitLab protected variables
Jenkins Credentials
AWS Secrets Manager
HashiCorp Vault
Kubernetes external secret systems
```

Use the organization's approved system.

---

## 38. Secret Exposure in Logs

Bad:

```bash
echo "$ARTIFACTORY_TOKEN"
```

---

## 39. Shell Safety

Avoid commands that accidentally expose secrets:

```bash
set -x
```

when sensitive commands are executed.

---

# PART XII — NETWORK SECURITY

## 40. TLS

Always protect registry communication with TLS.

```text
Client
  |
 HTTPS
  |
Artifactory
```

---

## 41. Certificate Validation

Verify:

```text
hostname
certificate chain
expiration
trust
```

---

## 42. Never Disable TLS Verification

Do not use insecure workarounds such as:

```text
--insecure
skip TLS verification
```

as permanent production fixes.

---

# PART XIII — NETWORK SEGMENTATION

## 43. Private Artifactory

Where architecture permits:

```text
private network
```

is preferable to unnecessary public exposure.

---

## 44. Firewall

Restrict:

```text
source
destination
port
protocol
```

Example:

```text
CI runners
   |
   v
TCP 443
   |
   v
Artifactory
```

---

## 45. Administrative Access

Separate:

```text
user access
```

from:

```text
administrative access
```

Use restricted network paths for administration where possible.

---

# PART XIV — REPOSITORY SECURITY

## 46. Repository Types

Security policies should account for:

```text
local
remote
virtual
```

---

## 47. Local Repository

Internal artifacts should normally be protected from unauthorized:

```text
deploy
delete
overwrite
```

---

## 48. Remote Repository

Remote repositories can introduce external software into the
organization.

Therefore control:

```text
approved upstream
cache policy
security scanning
dependency governance
```

---

## 49. Virtual Repository

A virtual repository can become a controlled dependency gateway.

Use it to centralize:

```text
approved package sources
```

---

# PART XV — DEPENDENCY SECURITY

## 50. Dependency Risk

External dependencies can contain:

```text
vulnerabilities
malware
license issues
typosquatting
compromised maintainers
```

---

## 51. Dependency Flow

Preferred:

```text
Application
   |
   v
Artifactory Virtual
   |
   v
Approved Remote
   |
   v
External Registry
```

---

## 52. Dependency Pinning

Prefer:

```text
specific version
```

rather than:

```text
latest
```

---

## 53. Lock Files

Use package-manager lock files where appropriate:

```text
package-lock.json
poetry.lock
requirements lock strategy
Gradle dependency locking
Maven dependency management
```

---

# PART XVI — ARTIFACT INTEGRITY

## 54. Immutability

Production artifacts should be immutable.

Example:

```text
4.2.1
```

should not silently become a different binary.

---

## 55. Why Immutability Matters

It protects:

```text
reproducibility
rollback
audit
forensics
trust
```

---

## 56. Build Once

Preferred:

```text
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Promote
```

Avoid:

```text
Build DEV
 ↓
Rebuild STAGE
 ↓
Rebuild PROD
```

---

# PART XVII — ARTIFACT SCANNING

## 57. Security Scanning

Scan artifacts/images for:

```text
CVEs
malware
secrets
licenses
policy violations
```

Capabilities depend on the JFrog security products and licensing
enabled in the environment.

---

## 58. Quality Gates

Example:

```text
Critical vulnerability
 ↓
BLOCK promotion
```

---

## 59. Risk-Based Policy

Not every vulnerability has the same risk.

Consider:

```text
severity
exploitability
runtime exposure
internet exposure
compensating controls
application usage
```

---

# PART XVIII — CONTAINER SECURITY

## 60. Base Images

Use:

```text
approved
maintained
scanned
```

base images.

---

## 61. Minimal Images

Reduce:

```text
packages
shell tools
debug utilities
```

when they are not required.

---

## 62. Run as Non-Root

Example:

```dockerfile
USER 10001
```

---

## 63. No Secrets in Images

Never include:

```text
API keys
AWS credentials
database passwords
private keys
```

inside container layers.

---

# PART XIX — IMAGE SIGNING

## 64. Why Sign Images?

Signing can establish:

```text
publisher identity
artifact integrity
trust
```

---

## 65. Verification

Production deployment can require:

```text
approved registry
+
approved signer
+
valid signature
```

The exact signing implementation should follow the organization's
chosen tooling.

---

# PART XX — SBOM

## 66. Software Bill of Materials

SBOM identifies software components.

Example:

```text
payment-service
 |
 +--> Java
 +--> Spring
 +--> Jackson
 +--> OS packages
```

---

## 67. Why SBOM Matters

If a vulnerability is discovered:

```text
CVE
 ↓
Affected library
 ↓
Find images/artifacts containing it
 ↓
Prioritize remediation
```

---

# PART XXI — MALICIOUS PACKAGE DEFENSE

## 68. Supply Chain Attack

Example:

```text
External package
 ↓
Compromised maintainer
 ↓
Malicious release
 ↓
Artifactory caches it
 ↓
CI downloads it
 ↓
Production compromise
```

---

## 69. Controls

Use:

```text
approved repositories
dependency scanning
version pinning
lock files
artifact scanning
review
monitoring
```

---

# PART XXII — REMOTE REPOSITORY SECURITY

## 70. Upstream Control

Do not allow arbitrary external sources when governance requires
approved sources.

---

## 71. Remote Cache

A remote repository can cache external packages.

Benefits:

```text
availability
speed
central governance
reduced external traffic
```

But cached malicious content can also spread.

Therefore scanning and policy remain important.

---

# PART XXIII — AUDIT LOGGING

## 72. Audit

Record security-relevant actions such as:

```text
login
repository access
artifact upload
artifact download
delete
permission change
user change
token change
```

Exact audit events depend on platform/version and configuration.

---

## 73. Why Audit Matters

During an incident:

```text
Who?
What?
When?
Where?
Which artifact?
Which repository?
```

---

## 74. Audit Retention

Retain logs according to:

```text
security policy
compliance
incident response
legal requirements
```

---

# PART XXIV — MONITORING

## 75. Security Monitoring

Monitor:

```text
authentication failures
403 spikes
unexpected uploads
unexpected deletes
privileged changes
token activity
repository anomalies
```

---

## 76. Baseline

Normal:

```text
CI publishes known repositories
```

Anomaly:

```text
developer account uploads production image
```

---

# PART XXV — ALERTING

## 77. High-Value Alerts

Examples:

```text
admin login anomaly
mass artifact deletion
new admin user
permission escalation
unexpected repository creation
credential failures
suspicious package upload
```

---

# PART XXVI — ARTIFACT PROMOTION SECURITY

## 78. Promotion

Production promotion should require:

```text
artifact validation
security checks
approval
```

where policy requires.

---

## 79. Promote Exact Artifact

Use:

```text
artifact version
digest
Build Info
```

Do not promote:

```text
latest
```

---

## 80. Separation of Duties

Possible model:

```text
Developer
 ↓
Build

Security
 ↓
Scan

Release Approver
 ↓
Promotion

Production
 ↓
Deploy
```

---

# PART XXVII — KUBERNETES SECURITY

## 81. Runtime Pull Permissions

Kubernetes workloads usually need:

```text
READ
```

not:

```text
DEPLOY
DELETE
```

---

## 82. imagePullSecrets

Protect registry secrets.

Use:

```text
namespace isolation
ServiceAccounts
external secret management
```

where appropriate.

---

## 83. Approved Registry

Use admission controls where possible to prevent:

```text
random public image
```

from entering production.

---

# PART XXVIII — EKS SECURITY

## 84. EKS + Artifactory

Security layers:

```text
EKS
 |
Network
 |
TLS
 |
Registry authentication
 |
Artifactory RBAC
 |
Image security
```

---

## 85. Separate CI and Runtime

```text
CI:
DEPLOY

EKS:
READ
```

This is an important least-privilege boundary.

---

# PART XXIX — GITHUB ACTIONS SECURITY

## 86. GitHub Actions

Protect:

```text
ARTIFACTORY_TOKEN
```

using:

```text
Secrets
Environments
protected release workflows
```

---

## 87. Forks

Do not expose production Artifactory credentials to untrusted fork
execution.

---

## 88. OIDC

Where supported by the identity architecture, consider federated
identity to reduce long-lived credentials.

---

# PART XXX — GITLAB SECURITY

## 89. GitLab

Use:

```text
protected variables
masked variables
protected branches
protected tags
protected environments
```

---

## 90. Runner Security

A compromised runner may expose:

```text
source
credentials
artifacts
cloud credentials
```

Use:

```text
ephemeral runners
isolation
patching
network restrictions
```

where practical.

---

# PART XXXI — JENKINS SECURITY

## 91. Jenkins

Protect:

```text
Jenkins controller
agents
credentials
plugins
pipelines
shared libraries
```

---

## 92. Credentials

Use:

```text
Jenkins Credentials Store
```

rather than pipeline source code.

---

## 93. Agent Isolation

Use isolated or ephemeral agents for high-risk workloads where
practical.

---

# PART XXXII — CI SUPPLY CHAIN

## 94. Pipeline Dependencies

Secure:

```text
GitHub Actions
GitLab templates
Jenkins plugins
JFrog CLI
Docker images
build tools
package managers
```

---

## 95. Pin Trusted Tools

Where organizational policy requires it, pin:

```text
action versions
container images
tool versions
```

and verify provenance.

---

# PART XXXIII — ARTIFACT DELETION SECURITY

## 96. Delete Permission

DELETE should be highly restricted.

---

## 97. Why?

An attacker with delete permission can cause:

```text
outage
failed deployments
rollback failure
supply-chain disruption
```

---

## 98. Soft/Controlled Cleanup

Use lifecycle automation instead of giving developers manual delete
access.

---

# PART XXXIV — BACKUP SECURITY

## 99. Backup

Back up critical:

```text
configuration
metadata
database
filestore
security configuration
```

according to the selected Artifactory architecture.

---

## 100. Backup Protection

Backups must also be protected against:

```text
unauthorized access
deletion
ransomware
tampering
```

---

# PART XXXV — DISASTER RECOVERY

## 101. DR Security

DR must preserve:

```text
identity
permissions
repositories
artifacts
metadata
certificates
DNS
```

---

## 102. DR Testing

Test:

```text
restore
 ↓
authenticate
 ↓
pull
 ↓
publish if required
 ↓
deploy
```

---

# PART XXXVI — HIGH AVAILABILITY SECURITY

## 103. HA

HA improves availability but increases the importance of securing:

```text
nodes
database
storage
load balancer
internal communication
```

---

## 104. Internal Network

Do not expose internal Artifactory components unnecessarily.

Use:

```text
segmentation
firewalls
private subnets
```

as appropriate.

---

# PART XXXVII — INCIDENT RESPONSE

## 105. Security Incident

Example:

```text
Malicious artifact discovered
```

Immediate steps:

```text
Identify artifact
 ↓
Block promotion
 ↓
Identify consumers
 ↓
Quarantine
 ↓
Audit
 ↓
Rotate credentials if needed
 ↓
Remove compromised artifact
 ↓
Deploy clean version
```

---

## 106. Credential Compromise

```text
Revoke
 ↓
Audit
 ↓
Identify access
 ↓
Rotate
 ↓
Validate
```

---

## 107. Mass Deletion

```text
Stop automation
 ↓
Identify actor
 ↓
Preserve logs
 ↓
Assess impact
 ↓
Restore
 ↓
Review permissions
```

---

# PART XXXVIII — PRODUCTION SECURITY ARCHITECTURE

## 108. Enterprise Model

```text
                 Identity Provider
                       |
                       v
                Human / Machine
                       |
                       v
                 Authentication
                       |
                       v
                   Artifactory
                 /     |      \
                /      |       \
             Local   Remote   Virtual
                |       |        |
                +-------+--------+
                        |
                        v
                  Security Scan
                        |
                        v
                     Build Info
                        |
                        v
                    Promotion
                        |
                        v
                 Kubernetes / EKS
```

---

## 109. Security Boundaries

```text
Internet
   |
   X
Private Network
   |
   v
Artifactory
   |
   +--> CI
   |
   +--> Kubernetes
```

---

# PART XXXIX — SECURITY CHECKLIST

## 110. Identity

```text
[ ] human and machine identities separated
[ ] SSO where appropriate
[ ] MFA for privileged users
[ ] service identities
[ ] offboarding
```

---

## 111. Authorization

```text
[ ] least privilege
[ ] groups
[ ] permission targets
[ ] project isolation
[ ] no unnecessary DELETE
[ ] no unnecessary ADMIN
```

---

## 112. Credentials

```text
[ ] tokens scoped
[ ] tokens rotated
[ ] secrets not in Git
[ ] secrets not in logs
[ ] CI secrets protected
[ ] runtime credentials separated
```

---

## 113. Network

```text
[ ] TLS
[ ] certificate validation
[ ] private access where appropriate
[ ] firewall
[ ] segmentation
[ ] restricted administration
```

---

## 114. Artifacts

```text
[ ] immutable releases
[ ] vulnerability scanning
[ ] SBOM
[ ] signing where required
[ ] approved repositories
[ ] dependency controls
```

---

## 115. Operations

```text
[ ] audit logging
[ ] monitoring
[ ] alerting
[ ] backup
[ ] DR
[ ] incident response
```

---

# PART XL — INTERVIEW PREPARATION

## 116. How Do You Secure Artifactory?

Answer:

```text
I use defense in depth: SSO and MFA for humans, dedicated service
identities for automation, least-privilege RBAC and permission
targets, scoped tokens, TLS, network restrictions, immutable
artifacts, vulnerability scanning, audit logging, monitoring and
tested backup/DR.
```

---

## 117. How Do You Secure CI Access?

Answer:

```text
I create a dedicated service identity with only the repositories and
operations required by that pipeline. Credentials are stored in the
CI secret mechanism, protected from untrusted PRs and rotated
regularly. Where supported, I evaluate short-lived federated
credentials.
```

---

## 118. Why Is Artifactory a Supply-Chain Security Boundary?

Answer:

```text
It is the point through which internal builds can obtain dependencies
and through which production systems can obtain artifacts. If the
repository is compromised, malicious content can propagate into CI and
production, so registry security directly affects software supply
chain security.
```

---

## 119. How Do You Handle a Leaked Token?

Answer:

```text
I immediately revoke or rotate the token, identify its scope, review
Artifactory audit logs, determine which repositories and artifacts
were accessed or modified, inspect downstream deployments and then
correct the original secret exposure.
```

---

## 120. How Do You Prevent Developers From Deleting Production Images?

Answer:

```text
I separate READ, DEPLOY and DELETE permissions, restrict DELETE to a
small operational group or controlled automation, enforce immutable
release policies and use lifecycle automation rather than broad
developer deletion access.
```

---

## 121. How Do You Secure Dependencies?

Answer:

```text
I route dependency resolution through approved Artifactory virtual
and remote repositories, pin versions, use lock files where
applicable, scan dependencies, enforce policy gates and monitor for
new vulnerabilities.
```

---

## 122. How Do You Secure Kubernetes Image Pulls?

Answer:

```text
I use a dedicated runtime identity with READ-only access, protected
registry credentials or an approved workload identity mechanism, TLS,
network restrictions and admission controls where appropriate. I
also use immutable image versions and digest tracking.
```

---

## 123. What Happens if Artifactory Is Compromised?

Answer:

```text
I treat it as a supply-chain incident. I isolate the affected
systems, preserve audit evidence, identify compromised artifacts and
credentials, block promotion, rotate credentials, determine affected
deployments, restore trusted artifacts and redeploy clean versions.
```

---

# PART XLI — GOLDEN RULES

## 124. Rules

```text
1. Treat Artifactory as a critical software supply-chain system.

2. Separate human, CI and runtime identities.

3. Apply least privilege everywhere.

4. Never use administrator credentials for normal CI/CD.

5. Restrict DELETE permissions aggressively.

6. Protect privileged accounts with MFA and strong authentication.

7. Use SSO where appropriate.

8. Prefer scoped access tokens for machine automation.

9. Rotate and revoke credentials regularly.

10. Never commit credentials to Git.

11. Never expose secrets in CI logs.

12. Protect production credentials from untrusted pull requests.

13. Use TLS and validate certificates.

14. Keep Artifactory private when practical.

15. Restrict network access to required paths.

16. Separate local, remote and virtual repository responsibilities.

17. Control external dependencies.

18. Scan artifacts and dependencies according to security policy.

19. Use immutable production artifacts.

20. Build once and promote the exact artifact.

21. Track image digests.

22. Use SBOM and signing where required.

23. Separate CI DEPLOY permissions from Kubernetes READ permissions.

24. Use admission controls to enforce approved registries where
    appropriate.

25. Monitor authentication failures and privileged activity.

26. Alert on unexpected artifact uploads and deletions.

27. Protect backups from unauthorized access and deletion.

28. Test disaster recovery.

29. Treat compromised runners and leaked tokens as supply-chain
    security incidents.

30. Maintain an incident response process for malicious artifacts.

31. Correlate Git commit, CI build, Build Info, artifact and
    production deployment.

32. Review permissions periodically.

33. Remove unused identities and tokens.

34. Review third-party CI tooling and integrations.

35. Security is a continuous process, not a one-time Artifactory
    configuration.
```

---