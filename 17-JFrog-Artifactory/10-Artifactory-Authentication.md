# Artifactory-Authentication

## 1. Purpose

This file covers authentication and identity management for JFrog Artifactory from fundamentals through production DevOps environments.

It covers:

- Authentication vs authorization
- Artifactory identity model
- Users
- Groups
- Service accounts
- Access tokens
- API keys and legacy considerations
- Username/password authentication
- SSO and identity providers
- LDAP concepts
- OAuth/OIDC concepts
- SAML concepts
- CI/CD authentication
- Jenkins
- GitHub Actions
- GitLab CI
- Kubernetes
- Docker
- Maven
- NPM
- PyPI
- Helm
- Token scopes
- Token expiration
- Token rotation
- Credential management
- Least privilege
- Authentication troubleshooting
- Audit and monitoring
- Production security architecture
- Incident response
- Real-world scenarios
- Interview preparation

---

# PART I — AUTHENTICATION FUNDAMENTALS

## 2. What Is Authentication?

Authentication answers:

```text
Who are you?
```

Examples:

```text
username + password
access token
SSO identity
OIDC identity
service identity
```

---

## 3. What Is Authorization?

Authorization answers:

```text
What are you allowed to do?
```

Examples:

```text
READ repository
DEPLOY artifact
DELETE artifact
ADMINISTER system
```

---

## 4. Authentication vs Authorization

```text
Client
  |
  v
Authentication
  |
  v
Identity established
  |
  v
Authorization
  |
  v
Permission evaluated
  |
  v
Repository operation
```

A successful login does not automatically mean the user can access every repository.

---

## 5. Example

A CI service account may successfully authenticate:

```text
Authenticated:
yes
```

but receive:

```text
403 Forbidden
```

because:

```text
Authorized:
no
```

---

## 6. Why Authentication Matters in Artifactory

Artifactory stores critical software artifacts:

```text
Docker images
Maven packages
NPM packages
Python packages
Helm charts
generic binaries
```

A compromised identity can potentially:

```text
download artifacts
publish malicious artifacts
delete artifacts
modify configuration
access sensitive repositories
```

Therefore authentication is a critical supply-chain control.

---

# PART II — ARTIFACTORY IDENTITY MODEL

## 7. Common Identity Types

Artifactory environments can have:

```text
human users
groups
service identities
automation identities
external directory users
SSO identities
```

---

## 8. Human User

Example:

```text
developer01
```

A human user may need:

```text
READ development repositories
```

but not:

```text
ADMIN
DELETE production artifacts
```

---

## 9. Group

Groups aggregate users.

Example:

```text
developers
release-engineers
platform-admins
security-team
```

Permissions can then be assigned to groups instead of individual users.

---

## 10. Why Use Groups?

Benefits:

```text
centralized management
consistent permissions
simpler onboarding
simpler offboarding
less permission drift
```

---

## 11. Service Identity

Automation should use dedicated identities.

Examples:

```text
jenkins-payment
github-actions-platform
gitlab-release
argocd-prod
```

Avoid using a human administrator account for automation.

---

## 12. Why Separate Service Identities?

It improves:

```text
auditability
least privilege
credential rotation
incident investigation
ownership
```

---

## 13. Shared Credentials Risk

Bad pattern:

```text
All CI jobs
    ↓
one admin Artifactory account
```

Problems:

```text
no clear attribution
large blast radius
difficult rotation
excessive privileges
```

Better:

```text
Pipeline A → service identity A
Pipeline B → service identity B
GitOps → service identity C
```

---

# PART III — AUTHENTICATION METHODS

## 14. Common Methods

Depending on deployment and version, Artifactory can support authentication mechanisms such as:

```text
username/password
access tokens
API keys in legacy environments
SSO
SAML
LDAP
OAuth/OIDC integrations
external identity providers
```

The exact available mechanisms depend on the Artifactory/JFrog Platform configuration.

---

## 15. Username and Password

Basic concept:

```text
username
+
password
```

This is simple but generally less desirable for long-lived automation.

---

## 16. Password Risk

Passwords can be:

```text
reused
shared
exposed
hard to rotate
hard to attribute
```

For automation, prefer short-lived or scoped machine credentials where supported.

---

## 17. Access Token

An access token is a credential representing an authenticated identity with defined scope and lifecycle.

Conceptually:

```text
Identity
   ↓
Token
   ↓
Artifactory
   ↓
Permission evaluation
```

---

## 18. Why Prefer Tokens for Automation?

Benefits can include:

```text
scoping
expiration
rotation
revocation
reduced password exposure
better automation handling
```

---

## 19. Token Lifetime

A token should have an appropriate lifetime.

Avoid:

```text
never-expiring production token
```

when the platform and workflow support controlled expiration.

---

## 20. Short-Lived vs Long-Lived Credentials

Short-lived:

```text
lower exposure window
```

Long-lived:

```text
easier operationally
higher compromise risk
```

Use the shortest practical lifetime that does not create unnecessary operational failure.

---

# PART IV — ACCESS TOKENS

## 21. Token Scope

A token should provide only the access required.

Conceptually:

```text
Token
 ↓
Repository permissions
 ↓
Allowed operations
```

---

## 22. Read-Only Token

Example:

```text
CI dependency consumer
```

Needs:

```text
READ
```

only.

---

## 23. Deploy Token

Example:

```text
CI package publisher
```

Needs:

```text
READ
DEPLOY
```

to approved repositories.

---

## 24. Administrative Token

An administrator may require broader permissions.

However:

```text
ADMIN token
```

should never be used as the default CI credential.

---

## 25. Token Blast Radius

Suppose a token has:

```text
READ every repository
DEPLOY every repository
DELETE every repository
```

If compromised:

```text
entire Artifactory environment
```

may be affected.

Use scoped identities.

---

## 26. Token Expiration

Define:

```text
creation date
expiration
owner
purpose
scope
rotation procedure
```

---

## 27. Token Rotation

Recommended:

```text
Create replacement
      ↓
Configure new token
      ↓
Test
      ↓
Monitor
      ↓
Revoke old token
```

---

## 28. Zero-Downtime Rotation

For critical automation:

```text
Old token active
       +
New token active
       ↓
Switch client
       ↓
Validate
       ↓
Revoke old
```

This avoids unnecessary deployment interruption.

---

# PART V — SSO

## 29. What Is SSO?

Single Sign-On allows users to authenticate through a centralized identity provider.

Conceptually:

```text
User
 ↓
Identity Provider
 ↓
Authentication
 ↓
Artifactory
```

---

## 30. SSO Benefits

```text
centralized authentication
MFA integration
central identity lifecycle
simpler offboarding
central audit
reduced password handling
```

---

## 31. SSO Architecture

```text
              Identity Provider
                     |
                     v
                 SSO Login
                     |
                     v
                 Artifactory
                     |
                     v
               Authorization
```

---

## 32. SAML

SAML is commonly used for enterprise web SSO.

Conceptually:

```text
Browser
 ↓
Identity Provider
 ↓
SAML Assertion
 ↓
Artifactory
```

---

## 33. OIDC

OIDC builds authentication on OAuth 2.0 concepts.

Conceptually:

```text
Client
 ↓
Identity Provider
 ↓
OIDC Token
 ↓
Application
```

OIDC is commonly used for modern workload and application identity patterns, depending on platform support.

---

# PART VI — LDAP AND DIRECTORY SERVICES

## 34. LDAP

LDAP can integrate Artifactory authentication with an enterprise directory.

Conceptually:

```text
User
 ↓
Artifactory
 ↓
LDAP
 ↓
Enterprise Directory
```

---

## 35. LDAP Benefits

```text
central users
central groups
existing enterprise identity
simpler user lifecycle
```

---

## 36. LDAP Group Mapping

Example:

```text
LDAP Group:
devops-engineers
```

mapped to:

```text
Artifactory Group:
developers
```

This can simplify repository permissions.

---

## 37. LDAP Failure

If LDAP becomes unavailable, authentication behavior depends on the configured architecture and cached/session state.

Monitor:

```text
LDAP connectivity
TLS
bind credentials
directory availability
```

---

# PART VII — MFA

## 38. Multi-Factor Authentication

MFA requires more than one authentication factor.

Example:

```text
Password
+
Authenticator
```

---

## 39. Why MFA Matters

MFA reduces risk from:

```text
stolen passwords
phishing
credential reuse
```

---

## 40. Human vs Machine Authentication

Important distinction:

```text
Human:
SSO + MFA

Automation:
token/workload identity
```

Do not force interactive MFA into unattended CI jobs unless the architecture explicitly supports it.

---

# PART VIII — CI/CD AUTHENTICATION

## 41. CI Authentication Architecture

```text
CI Pipeline
    |
    v
Secret / Identity Provider
    |
    v
Service Identity
    |
    v
Artifactory
```

---

## 42. Jenkins

Example:

```text
Jenkins
 ↓
Jenkins Credentials
 ↓
Artifactory Token
 ↓
Artifactory
```

---

## 43. GitHub Actions

Example:

```text
GitHub Actions
 ↓
GitHub Secret / Federation
 ↓
Artifactory
```

Use the organization's approved identity mechanism.

---

## 44. GitLab CI

Example:

```text
GitLab Runner
 ↓
CI/CD Secret
 ↓
Artifactory
```

---

## 45. CI Read Identity

A build that only downloads dependencies should receive:

```text
READ
```

to:

```text
maven-virtual
npm-virtual
pypi-virtual
docker-virtual
helm-virtual
```

as required.

---

## 46. CI Publish Identity

A package publisher may receive:

```text
READ
+
DEPLOY
```

to a specific repository.

Example:

```text
payment-ci
 ↓
DEPLOY
 ↓
payment-maven-local
```

---

## 47. Separate Build and Release Identities

A stronger architecture can use:

```text
build identity
    ↓
READ dependencies

release identity
    ↓
DEPLOY approved artifact
```

This reduces privilege during ordinary builds.

---

# PART IX — AUTHENTICATION FOR DIFFERENT REPOSITORY TYPES

## 48. Maven

Maven commonly gets credentials through:

```text
settings.xml
CI secret
token
```

---

## 49. NPM

NPM commonly uses:

```text
.npmrc
registry
token
```

Secrets should be injected securely.

---

## 50. PyPI

Python tooling can use:

```text
pip configuration
environment
Twine credentials
token
```

---

## 51. Docker

Docker commonly uses:

```bash
docker login
```

with credentials supplied securely.

---

## 52. Helm

Traditional Helm repositories and OCI registries can use authentication mechanisms supported by the repository and Helm version.

For OCI:

```bash
helm registry login
```

can be used.

---

## 53. Kubernetes

Kubernetes workloads may use:

```text
imagePullSecrets
service accounts
workload identity
```

depending on registry architecture.

The Pods normally need image pull access, not chart publication access.

---

# PART X — CREDENTIAL MANAGEMENT

## 54. Central Secret Store

CI credentials should preferably be stored in:

```text
Jenkins Credentials
GitHub Secrets
GitLab CI Variables
Vault
Cloud Secret Manager
```

or an equivalent enterprise secret-management platform.

---

## 55. Secret Injection

Preferred:

```text
Secret Store
 ↓
CI runtime
 ↓
Temporary environment/configuration
 ↓
Artifactory
```

Avoid:

```text
Git repository
 ↓
credential
```

---

## 56. Environment Variables

Environment variables can reduce source-code exposure, but they are not automatically secure.

Risk:

```text
debug logs
process inspection
accidental output
```

Mask secrets and limit exposure.

---

## 57. Configuration Files

A temporary configuration file can be generated during CI.

Example:

```text
CI Secret
 ↓
temporary settings.xml
 ↓
Maven
 ↓
Artifactory
```

Delete it after use where appropriate.

---

## 58. Secret Masking

CI systems should mask:

```text
tokens
passwords
API keys
```

from logs.

But never assume masking is perfect.

Avoid printing secrets in commands.

---

# PART XI — SERVICE ACCOUNTS

## 59. Dedicated Service Account

Example:

```text
svc-payment-build
```

Purpose:

```text
Build payment application
```

Permissions:

```text
READ approved repositories
DEPLOY payment repository
```

---

## 60. Naming Convention

Use a consistent naming strategy:

```text
svc-<application>-<purpose>
```

Examples:

```text
svc-payment-build
svc-platform-release
svc-gitops-prod
```

---

## 61. Ownership

Every service identity should have:

```text
owner
team
purpose
repositories
expiration/rotation policy
```

---

## 62. Dormant Accounts

Regularly identify:

```text
unused users
unused tokens
old service accounts
former integrations
```

Disable or revoke them according to policy.

---

# PART XII — LEAST PRIVILEGE

## 63. Principle

Grant:

```text
minimum permissions
```

required to perform the task.

---

## 64. Example

Bad:

```text
Jenkins
 ↓
Artifactory Admin
```

Better:

```text
Jenkins
 ↓
READ virtual repositories
DEPLOY application local repository
```

---

## 65. Delete Permission

Delete access is powerful.

Avoid granting:

```text
DELETE
```

to standard build pipelines unless explicitly required.

---

## 66. Repository-Level Isolation

Example:

```text
payment-ci
 ↓
payment-maven-local
```

not:

```text
payment-ci
 ↓
all-maven-repositories
```

---

# PART XIII — AUTHENTICATION + RBAC

## 67. Authentication Is Not RBAC

Authentication:

```text
Who?
```

RBAC/permissions:

```text
What can they do?
```

Both must be designed together.

---

## 68. Example

```text
User authenticates successfully
        ↓
Group membership
        ↓
Permission target
        ↓
Repository permission
```

---

## 69. Permission Evaluation

A request may depend on:

```text
identity
group
token
repository
path
operation
project
```

The exact authorization model depends on Artifactory configuration.

---

# PART XIV — AUTHENTICATION AUDITING

## 70. What to Audit

Monitor:

```text
successful logins
failed logins
token creation
token revocation
permission changes
user creation
user deletion
group changes
repository access
administrative operations
```

---

## 71. Failed Authentication

Repeated failures can indicate:

```text
wrong CI secret
expired credential
credential rotation issue
brute force
compromised identity
```

---

## 72. Unexpected Token Use

If a token is used from an unexpected location:

```text
investigate
validate owner
check activity
rotate/revoke if necessary
```

---

## 73. Audit Correlation

Correlate:

```text
Artifactory logs
CI logs
identity-provider logs
Kubernetes audit
cloud logs
```

to reconstruct incidents.

---

# PART XV — AUTHENTICATION TROUBLESHOOTING

## 74. Troubleshooting Model

Use:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
Authentication
 ↓
Authorization
 ↓
Repository
 ↓
Artifact
```

---

## 75. HTTP 401

Usually:

```text
authentication failure
```

Check:

```text
credential
token
expiration
registry URL
username
CI secret
```

---

## 76. HTTP 403

Usually:

```text
authenticated
but unauthorized
```

Check:

```text
permission target
repository permission
group membership
token scope
project access
```

---

## 77. HTTP 404

Can indicate:

```text
wrong repository
wrong artifact
wrong path
artifact not available
virtual repository configuration
```

Do not automatically assume authentication is correct or incorrect from a 404 without checking the complete request and server behavior.

---

## 78. Docker Login Failure

Check:

```bash
docker login <registry>
```

Then inspect:

```text
registry hostname
credentials
token
TLS
network
```

---

## 79. Maven 401

Check:

```text
settings.xml
server ID
token
registry URL
CI secret
```

---

## 80. NPM 401

Check:

```text
.npmrc
registry
_authToken
CI secret
token expiration
```

---

## 81. PyPI 401

Check:

```text
pip index
Twine configuration
credentials
token
environment
```

---

## 82. Helm Authentication Failure

Check:

```text
Helm repository configuration
OCI login
token
repository URL
CI credentials
```

---

## 83. Kubernetes Image Pull Authentication

If:

```text
ImagePullBackOff
```

check:

```text
imagePullSecret
ServiceAccount
registry URL
token
repository permission
```

---

# PART XVI — TLS AND CERTIFICATES

## 84. Why TLS Matters

Authentication credentials travel through network connections.

Use:

```text
HTTPS
TLS
```

for secure communication.

---

## 85. Certificate Validation

Clients verify:

```text
certificate chain
hostname
validity period
trust
```

---

## 86. Internal CA

Enterprise environments may use:

```text
internal CA
```

Clients must trust the appropriate CA chain.

---

## 87. Certificate Rotation

Plan:

```text
New certificate
 ↓
Install
 ↓
Validate clients
 ↓
Monitor
 ↓
Remove old certificate
```

---

## 88. Do Not Disable TLS Verification

Avoid production workarounds such as:

```text
insecure
skip certificate verification
trust any certificate
```

These weaken authentication security.

---

# PART XVII — SSO AND MACHINE USERS

## 89. Human Authentication

Preferred pattern:

```text
User
 ↓
SSO
 ↓
MFA
 ↓
Artifactory
```

---

## 90. Machine Authentication

Preferred pattern:

```text
CI
 ↓
Machine identity
 ↓
Scoped token/workload identity
 ↓
Artifactory
```

Do not use interactive human credentials for unattended automation.

---

# PART XVIII — KUBERNETES / GITOPS AUTHENTICATION

## 91. Argo CD Authentication

A GitOps controller may need credentials to access:

```text
Git
+
Artifactory
```

These are separate trust relationships.

---

## 92. Argo CD + Artifactory

Conceptually:

```text
Argo CD
   |
   v
Artifactory Helm/OCI
   |
   v
Chart
```

Use a dedicated identity:

```text
svc-argocd-prod
```

with appropriate read permissions.

---

## 93. Kubernetes Runtime

Pods usually need:

```text
Docker/OCI image READ
```

not:

```text
Helm chart READ
```

because the chart has already been rendered/applied.

---

## 94. Separation of Credentials

Prefer:

```text
Argo CD credential
→ Helm repository

Kubernetes pull credential
→ Docker repository
```

rather than one credential with broad access to everything.

---

# PART XIX — INCIDENT RESPONSE

## 95. Compromised Token

Immediate steps:

```text
Identify token
 ↓
Revoke token
 ↓
Identify owner
 ↓
Review audit logs
 ↓
Identify accessed repositories
 ↓
Identify published/deleted artifacts
 ↓
Rotate related credentials
 ↓
Assess impact
```

---

## 96. Compromised CI Account

Response:

```text
Disable account/token
 ↓
Stop affected pipelines
 ↓
Review builds
 ↓
Review artifact changes
 ↓
Check production deployments
 ↓
Rotate credentials
 ↓
Rebuild trusted artifacts
```

---

## 97. Malicious Artifact Published

Flow:

```text
Detect
 ↓
Quarantine/block
 ↓
Identify artifact consumers
 ↓
Identify affected deployments
 ↓
Revoke compromised credentials
 ↓
Remove malicious version from approved flow
 ↓
Publish trusted version
 ↓
Redeploy
 ↓
Audit
```

---

# PART XX — PRODUCTION AUTHENTICATION ARCHITECTURE

## 98. Enterprise Architecture

```text
                  Identity Provider
                         |
                         v
                 Human SSO + MFA
                         |
                         v
                    Artifactory
                         |
               +---------+---------+
               |                   |
            Groups             Service IDs
               |                   |
               v                   v
            RBAC             Scoped Tokens
                                   |
                                   v
                              CI / GitOps
```

---

## 99. CI Architecture

```text
CI
 |
 v
Secret / Identity System
 |
 v
Service Identity
 |
 v
Scoped Token
 |
 v
Artifactory
 |
 +--> READ virtual
 |
 +--> DEPLOY specific local
```

---

## 100. Production Separation

```text
Human
→ SSO + MFA

CI
→ service identity + scoped token

GitOps
→ dedicated read identity

Kubernetes
→ image pull identity
```

This reduces credential blast radius.

---

# PART XXI — PRODUCTION CREDENTIAL LIFECYCLE

## 101. Lifecycle

```text
Request
 ↓
Approval
 ↓
Create identity
 ↓
Assign least privilege
 ↓
Generate credential
 ↓
Store securely
 ↓
Deploy
 ↓
Monitor
 ↓
Rotate
 ↓
Revoke
```

---

## 102. Credential Inventory

Maintain:

```text
identity
owner
purpose
repositories
permissions
creation date
expiration
rotation date
last use
```

---

## 103. Access Review

Perform periodic reviews:

```text
Who has access?
What repositories?
Which operations?
Is access still needed?
```

---

## 104. Offboarding

When a user leaves or changes role:

```text
Disable identity
 ↓
Remove group membership
 ↓
Revoke tokens
 ↓
Review service ownership
```

---

# PART XXII — REAL-WORLD SCENARIOS

## 105. Scenario — Jenkins Suddenly Gets 401

Likely:

```text
token expired
credential rotated
secret changed
wrong server/registry configuration
```

Response:

```text
Validate token
Check secret
Check server URL
Test authentication
Review rotation history
```

---

## 106. Scenario — Jenkins Gets 403 After Credential Rotation

Likely:

```text
new token authenticates
but lacks old permissions
```

Response:

```text
Compare scopes
Compare repository permissions
Validate service identity
```

---

## 107. Scenario — Developer Can Download but Cannot Publish

This may be intentional.

Example:

```text
Developer:
READ

CI Release:
READ + DEPLOY
```

Use CI for controlled publication.

---

## 108. Scenario — GitOps Can Pull Charts but Cannot Push

This is usually correct.

GitOps should generally need:

```text
READ
```

not:

```text
DEPLOY
```

---

## 109. Scenario — One Token Is Used by 100 Pipelines

Risk:

```text
large blast radius
poor attribution
difficult rotation
```

Improve:

```text
separate service identities
or
scoped identities by team/application
```

---

## 110. Scenario — Token Appears in Logs

Immediate:

```text
Assume exposed
 ↓
Revoke/rotate
 ↓
Review logs
 ↓
Check access
 ↓
Fix masking
 ↓
Prevent recurrence
```

---

## 111. Scenario — User Has Excessive Permissions

Response:

```text
Review actual requirements
 ↓
Remove unnecessary permissions
 ↓
Use group/RBAC design
 ↓
Test
 ↓
Audit
```

---

## 112. Scenario — SSO Is Down

Impact depends on architecture.

Investigate:

```text
existing sessions
token-based automation
identity provider availability
fallback behavior
```

Automation should not depend unnecessarily on interactive SSO.

---

# PART XXIII — INTERVIEW PREPARATION

## 113. What Is the Difference Between Authentication and Authorization?

Answer:

```text
Authentication verifies identity, while authorization determines
what that identity is allowed to do. In Artifactory, a user can
authenticate successfully but still receive 403 if repository
permissions do not allow the requested operation.
```

---

## 114. How Do You Authenticate Jenkins to Artifactory?

Answer:

```text
I use a dedicated Jenkins service identity with a scoped token or
approved machine credential. The credential is stored in Jenkins
Credentials and injected at runtime. I grant only the required read
and deploy permissions.
```

---

## 115. Why Should CI Not Use an Admin Account?

Answer:

```text
An admin credential creates excessive privilege and a large blast
radius. If compromised, an attacker could potentially modify or
delete critical repositories. I use dedicated least-privilege service
identities.
```

---

## 116. How Do You Handle Token Rotation?

Answer:

```text
I create the replacement token first, configure the client, validate
the workflow, monitor for successful use and then revoke the old
token. For critical systems I use overlapping validity to avoid
downtime.
```

---

## 117. How Do You Troubleshoot 401 vs 403?

Answer:

```text
401 generally points to authentication problems such as invalid or
expired credentials. 403 generally means authentication succeeded
but the identity lacks authorization for the requested operation.
I still verify the exact endpoint and server behavior before making
the final conclusion.
```

---

## 118. How Do You Secure Artifactory for Developers?

Answer:

```text
I use enterprise SSO and MFA for humans, groups for access
management, least-privilege repository permissions, controlled
production access and regular access reviews.
```

---

## 119. How Do You Secure CI/CD?

Answer:

```text
I use dedicated service identities, scoped credentials, secure CI
secret stores, short practical token lifetimes, rotation, audit
logging and separate read and publish permissions.
```

---

## 120. How Do You Secure GitOps Access?

Answer:

```text
I create a dedicated GitOps identity with read-only access to the
required Helm/OCI repositories. I do not give the controller
unnecessary deploy, delete or administrative privileges.
```

---

## 121. How Do You Handle a Compromised Token?

Answer:

```text
I revoke it immediately, identify the owner and affected systems,
review Artifactory and CI audit logs, determine what repositories and
operations were accessed, rotate related credentials and remediate
any affected artifacts or deployments.
```

---

## 122. Why Use SSO for Humans but Tokens for CI?

Answer:

```text
Humans benefit from interactive SSO and MFA. CI is unattended, so it
needs a machine identity that can authenticate without interactive
login and can be scoped, rotated and audited independently.
```

---

## 123. How Do You Prevent Credential Sprawl?

Answer:

```text
I centralize secrets, standardize service identities, avoid shared
accounts, assign owners, enforce expiration/rotation, remove unused
credentials and perform periodic access reviews.
```

---

## 124. What Should a Production Authentication Design Include?

Answer:

```text
Human SSO/MFA, dedicated service identities, least privilege,
scoped tokens, secure secret storage, rotation, auditing, access
reviews, TLS, incident response and separation between human,
automation, GitOps and runtime credentials.
```

---

# PART XXIV — PRODUCTION CHECKLIST

## 125. Human Access

```text
[ ] SSO
[ ] MFA
[ ] groups
[ ] least privilege
[ ] access reviews
[ ] offboarding
```

---

## 126. Service Identities

```text
[ ] dedicated identity
[ ] owner
[ ] purpose
[ ] scoped permissions
[ ] expiration
[ ] rotation
[ ] last-use tracking
```

---

## 127. CI/CD

```text
[ ] Jenkins credentials
[ ] GitHub secrets/federation
[ ] GitLab variables
[ ] no hardcoded secrets
[ ] read-only build access
[ ] controlled publish access
```

---

## 128. Repository Permissions

```text
[ ] READ
[ ] DEPLOY
[ ] DELETE restricted
[ ] ADMIN restricted
[ ] project isolation
[ ] production separation
```

---

## 129. Security

```text
[ ] TLS
[ ] trusted CA
[ ] token rotation
[ ] secret masking
[ ] audit logging
[ ] failed-login monitoring
[ ] incident response
```

---

## 130. Operations

```text
[ ] credential inventory
[ ] access reviews
[ ] dormant account cleanup
[ ] backup identity configuration where required
[ ] DR
[ ] documented recovery procedures
```

---

# PART XXV — GOLDEN RULES

## 131. Rules

```text
1. Authentication answers "who are you?"

2. Authorization answers "what are you allowed to do?"

3. Never confuse successful authentication with repository access.

4. Use SSO + MFA for human users where supported.

5. Use dedicated service identities for automation.

6. Never use a human administrator account for normal CI/CD.

7. Apply least privilege.

8. Prefer scoped tokens for machine access where appropriate.

9. Avoid permanent credentials when practical.

10. Define token expiration and rotation.

11. Rotate credentials without unnecessary downtime.

12. Store credentials in approved secret-management systems.

13. Never commit credentials to Git.

14. Never print credentials in CI logs.

15. Separate read and publish identities when practical.

16. Restrict DELETE and ADMIN permissions.

17. Give GitOps controllers only the repository access they need.

18. Give Kubernetes workloads only the image-pull access they need.

19. Use TLS and validate certificates.

20. Audit authentication and authorization changes.

21. Monitor failed authentication and unusual token activity.

22. Maintain an inventory of service identities and credentials.

23. Review permissions periodically.

24. Revoke unused and compromised credentials immediately.

25. Design authentication with incident response and blast-radius
    reduction in mind.

26. Validate exact authentication features, token behavior and
    configuration syntax against the deployed JFrog/Artifactory
    version before production rollout.
```

---