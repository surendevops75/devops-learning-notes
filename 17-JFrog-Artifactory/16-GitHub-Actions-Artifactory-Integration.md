# 17-JFrog-Artifactory
# 16-GitHub-Actions-Artifactory-Integration

## 1. Purpose

This file provides a deep, production-oriented guide to integrating GitHub Actions with JFrog Artifactory.

The goal is not merely to publish an artifact from a workflow. A production implementation must provide:

```text
secure authentication
least-privilege authorization
dependency management
reproducible builds
artifact immutability
Build Info
security scanning
promotion
release governance
rollback
auditability
```

The architecture covered here is:

```text
Developer
   |
   v
GitHub Repository
   |
   v
GitHub Actions
   |
   +--> Checkout
   +--> Build
   +--> Test
   +--> Security Scan
   +--> Package
   |
   v
JFrog Artifactory
   |
   +--> Dependencies
   +--> Artifacts
   +--> Build Info
   +--> Promotion
   |
   v
Kubernetes / EKS
```

---

# PART I — GITHUB ACTIONS + ARTIFACTORY FUNDAMENTALS

## 2. What Is GitHub Actions?

GitHub Actions is GitHub's CI/CD automation platform.

A workflow is defined in:

```text
.github/workflows/*.yml
```

A workflow can:

```text
checkout source
install tools
build
test
scan
publish artifacts
deploy applications
```

---

## 3. What Is Artifactory?

JFrog Artifactory is an enterprise artifact repository.

It can store and distribute:

```text
Maven
Gradle
NPM
PyPI
Docker/OCI
Helm
NuGet
Generic artifacts
```

---

## 4. Why Integrate Them?

GitHub provides:

```text
source control
pull requests
workflow automation
release events
```

Artifactory provides:

```text
artifact storage
dependency proxying
repository management
access control
artifact metadata
Build Info
promotion
retention
```

Together:

```text
GitHub
   |
   v
Actions
   |
   v
Build
   |
   v
Artifactory
```

---

## 5. Basic Production Flow

```text
Pull Request
    |
    v
Build + Test
    |
    v
Security Checks
    |
    v
Merge
    |
    v
Release Tag
    |
    v
Build
    |
    v
Publish Artifact
    |
    v
Build Info
    |
    v
Promotion
    |
    v
Deploy
```

---

# PART II — CORE ARCHITECTURE

## 6. Dependency Architecture

Do not allow every GitHub Actions runner to retrieve packages randomly from the public internet when enterprise governance requires a controlled source.

Preferred:

```text
GitHub Actions Runner
        |
        v
Artifactory Virtual Repository
        |
   +----+----+
   |         |
Local      Remote
   |         |
Internal   Approved
Packages   External
```

---

## 7. Artifact Publication Architecture

```text
GitHub Actions
       |
       v
Build
       |
       v
Security Scan
       |
       v
Artifactory Local Repository
```

Examples:

```text
maven-local
npm-local
pypi-local
docker-local
helm-local
```

Use the organization's actual repository naming standard.

---

## 8. Build Info Architecture

```text
Git commit
    |
    v
GitHub Actions run
    |
    +--> dependencies
    |
    +--> artifacts
    |
    +--> build metadata
    |
    v
JFrog Build Info
```

---

# PART III — AUTHENTICATION OPTIONS

## 9. Authentication Is Separate from Authorization

Authentication asks:

```text
Who is the workflow?
```

Authorization asks:

```text
What may that workflow do?
```

A valid GitHub Actions credential can still receive:

```text
403 Forbidden
```

if Artifactory permissions are insufficient.

---

## 10. Credential Options

Common patterns include:

```text
Artifactory access token
username/password
OIDC-based federation where supported by the organization's JFrog
architecture
```

The exact supported authentication mechanisms depend on the deployed JFrog Platform version and identity architecture.

---

## 11. Preferred Machine Identity

Use:

```text
GitHub Actions
      |
      v
Dedicated service identity
      |
      v
Scoped credential
      |
      v
Artifactory
```

Do not use:

```text
GitHub Actions
      |
      v
Artifactory administrator
```

---

# PART IV — ARTIFACTORY ACCESS TOKEN

## 12. Why Use an Access Token?

A machine token can be:

```text
scoped
rotated
revoked
separated from human credentials
```

---

## 13. Token Ownership

Create an identity such as:

```text
svc-gha-payment
```

rather than using a human account.

---

## 14. Token Permissions

Example:

```text
READ:
maven-virtual
npm-virtual
docker-virtual

DEPLOY:
payment-maven-local
payment-npm-local
payment-docker-local

DELETE:
NO

ADMIN:
NO
```

---

## 15. Token Storage

Store the credential in:

```text
GitHub Actions Secrets
```

or an approved external secret mechanism.

Never:

```text
commit token to repository
write token into workflow YAML
print token in logs
```

---

# PART V — GITHUB ACTIONS SECRETS

## 16. Repository Secrets

A simple pattern:

```text
ARTIFACTORY_URL
ARTIFACTORY_TOKEN
```

---

## 17. Environment Secrets

For stronger separation:

```text
dev
stage
prod
```

Each environment can have different protected secrets and approval policies.

---

## 18. Organization Secrets

For shared enterprise configuration, organization-level secrets can reduce duplication.

However, access should be restricted to only repositories that require them.

---

## 19. Secret Scope

Avoid exposing production credentials to:

```text
pull_request from untrusted fork
feature branch
untrusted reusable workflow
```

unless the security model explicitly supports it.

---

# PART VI — GITHUB OIDC AND FEDERATED IDENTITY

## 20. Why OIDC Matters

GitHub Actions can issue an OIDC identity token to a workflow.

The broader pattern is:

```text
GitHub Actions
      |
      | OIDC identity
      v
Trusted identity provider
      |
      v
Short-lived credentials
```

---

## 21. Long-Lived vs Federated Credentials

Long-lived:

```text
GitHub
 ↓
Static token
 ↓
Artifactory
```

Federated:

```text
GitHub
 ↓
OIDC
 ↓
Identity federation
 ↓
Short-lived credential
```

Federation can reduce the exposure associated with long-lived static credentials when supported by the target architecture.

---

## 22. OIDC Security

Do not trust every GitHub workflow automatically.

Use conditions based on appropriate claims such as:

```text
repository
organization
branch
environment
workflow
```

The exact claims and trust configuration depend on the identity integration.

---

# PART VII — GITHUB ENVIRONMENTS

## 23. Why Use Environments?

Production credentials should be separated from development credentials.

Example:

```text
dev
stage
prod
```

---

## 24. Production Environment

A production GitHub Environment can enforce controls such as:

```text
required reviewers
restricted branches/tags
environment-specific secrets
```

---

## 25. Production Flow

```text
Release Tag
    |
    v
GitHub Actions
    |
    v
Production Environment
    |
    v
Approval
    |
    v
Artifactory Promotion
```

---

# PART VIII — WORKFLOW SECURITY

## 26. pull_request Security

A pull request workflow can execute code from the proposed change.

Therefore:

```text
Do not expose high-value production credentials unnecessarily.
```

---

## 27. Fork Pull Requests

Fork workflows are especially sensitive.

Avoid exposing:

```text
production Artifactory token
cloud administrator credentials
deployment credentials
```

to untrusted fork execution.

---

## 28. permissions Block

Use minimal GitHub token permissions.

Example:

```yaml
permissions:
  contents: read
```

Add permissions only when required.

---

## 29. Workflow-Level Permissions

Prefer:

```yaml
permissions:
  contents: read
```

and tighten further at job level when appropriate.

---

# PART IX — BASIC WORKFLOW STRUCTURE

## 30. Example

```yaml
name: Build and Publish

on:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Build
        run: mvn clean verify
```

The exact action versions should be reviewed periodically and pinned according to the organization's supply-chain policy.

---

# PART X — MAVEN + ARTIFACTORY

## 31. Maven Architecture

```text
GitHub Actions
      |
      v
Maven
      |
      v
Artifactory Virtual
      |
      +--> Local
      +--> Remote
```

---

## 32. Maven Dependency Resolution

Configure Maven to resolve dependencies through:

```text
maven-virtual
```

rather than directly accessing arbitrary external repositories.

---

## 33. Maven Publish

```text
GitHub Actions
      |
      v
mvn deploy
      |
      v
maven-local
```

---

## 34. Maven settings.xml

A production workflow should generate or provide settings securely.

Example pattern:

```xml
<servers>
  <server>
    <id>company-artifactory</id>
    <username>${env.ARTIFACTORY_USER}</username>
    <password>${env.ARTIFACTORY_TOKEN}</password>
  </server>
</servers>
```

Do not commit a file containing real credentials.

---

## 35. Maven Repository Configuration

Conceptually:

```xml
<repository>
  <id>company-artifactory</id>
  <url>https://artifactory.example.com/artifactory/maven-virtual</url>
</repository>
```

Use the actual enterprise endpoint in production.

---

## 36. Maven Release

Example:

```text
4.2.1
```

A production release should be immutable.

---

## 37. Maven Snapshot

Example:

```text
4.2.1-SNAPSHOT
```

Snapshots should be governed separately from production releases.

---

# PART XI — NPM + ARTIFACTORY

## 38. NPM Dependency Flow

```text
npm ci
   |
   v
Artifactory npm-virtual
```

---

## 39. NPM Publish Flow

```text
npm pack
   |
   v
npm publish
   |
   v
npm-local
```

---

## 40. .npmrc

A workflow can create a temporary `.npmrc`.

Example concept:

```text
registry=https://artifactory.example.com/artifactory/api/npm/npm-virtual/
```

Credentials should be injected securely.

---

## 41. Do Not Commit Tokens

Bad:

```text
//registry.example.com/:_authToken=xxxxx
```

inside Git.

Good:

```text
GitHub Secret
 ↓
temporary workflow configuration
```

---

# PART XII — PYPI + ARTIFACTORY

## 42. Python Dependency Flow

```text
pip install
   |
   v
Artifactory PyPI virtual
```

---

## 43. Build

```bash
python -m build
```

---

## 44. Publish

Use the organization's approved Python publishing tool and Artifactory endpoint.

Conceptually:

```text
dist/
 ├── package-4.2.1.tar.gz
 └── package-4.2.1-py3-none-any.whl
```

---

# PART XIII — DOCKER + ARTIFACTORY

## 45. Docker Architecture

```text
GitHub Actions
      |
      v
docker build
      |
      v
Container Scan
      |
      v
docker push
      |
      v
Artifactory Docker Repository
```

---

## 46. Docker Authentication

Use a dedicated service identity with only the required registry permissions.

---

## 47. Docker Build

```bash
docker build \
  -t artifactory.example.com/docker-local/payment-service:4.2.1 \
  .
```

---

## 48. Docker Push

```bash
docker push \
  artifactory.example.com/docker-local/payment-service:4.2.1
```

---

## 49. Digest

After publishing, record:

```text
sha256:...
```

Production deployment should preferably use an immutable release reference and/or verified digest.

---

## 50. Build Metadata

Connect:

```text
Git commit
workflow run
image tag
image digest
Build Info
```

---

# PART XIV — HELM + ARTIFACTORY

## 51. Helm Validation

```bash
helm lint ./chart
helm template ./chart
```

---

## 52. Helm Packaging

```bash
helm package ./chart
```

---

## 53. Helm Repository

The workflow publishes the chart to the organization's approved Artifactory Helm or OCI repository.

---

## 54. Chart/Application Version

Example:

```text
Chart:
1.4.0

Application:
4.2.1
```

Keep the relationship explicit.

---

# PART XV — JFROG CLI IN GITHUB ACTIONS

## 55. Why Use JFrog CLI?

JFrog CLI can simplify:

```text
artifact upload
artifact download
search
build-info collection
build-info publication
promotion
```

---

## 56. CLI Setup

Use the official JFrog-provided GitHub integration/action or CLI installation approach appropriate for the organization's current toolchain.

Do not copy unverified third-party actions into production workflows.

---

## 57. Example Upload Concept

```bash
jf rt upload \
  "dist/*.jar" \
  "maven-local/com/company/payment/"
```

Use repository-specific path conventions.

---

## 58. Build Name and Number

Example:

```text
Build name:
payment-service

Build number:
${GITHUB_RUN_NUMBER}
```

A stronger production identifier can also incorporate repository/release context according to the organization's Build Info policy.

---

# PART XVI — BUILD INFO IN GITHUB ACTIONS

## 59. Build Info Flow

```text
GitHub Actions
      |
      +--> Source
      +--> Dependencies
      +--> Artifacts
      |
      v
JFrog Build Info
```

---

## 60. Useful Metadata

Capture:

```text
repository
commit SHA
tag
branch
workflow
run number
artifact version
```

---

## 61. Avoid Secrets

Do not publish:

```text
ARTIFACTORY_TOKEN
AWS_SECRET_ACCESS_KEY
DATABASE_PASSWORD
PRIVATE_KEY
```

as build environment metadata.

---

# PART XVII — RELEASE VERSIONING

## 62. Git Tag Driven Release

Example:

```text
v4.2.1
```

Flow:

```text
Tag
 ↓
Workflow
 ↓
Extract version
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
```

---

## 63. Version Validation

Before publishing:

```text
validate version
 ↓
check repository
 ↓
ensure version does not already exist
```

---

## 64. Immutable Release

If:

```text
4.2.1
```

already exists, do not silently overwrite it.

Create a new version when a corrected release is required.

---

# PART XVIII — BUILD ONCE, PROMOTE MANY

## 65. Production Pattern

```text
Release Tag
    |
    v
GitHub Actions
    |
    v
Build
    |
    v
Test
    |
    v
Scan
    |
    v
Artifactory
    |
    v
Build Info
    |
    v
Promotion
    |
    +--> Stage
    |
    +--> Production
```

---

## 66. Why Promote Instead of Rebuild?

Rebuilding can produce:

```text
different dependencies
different base image
different timestamps
different compiler behavior
different artifact bytes
```

Promotion keeps the tested artifact intact.

---

## 67. Promotion Gate

Production promotion may require:

```text
security passed
tests passed
approval
change record
artifact version validated
```

---

# PART XIX — GITHUB RELEASES

## 68. GitHub Release

A GitHub Release can represent:

```text
v4.2.1
```

It should be associated with a Git tag.

---

## 69. Release Workflow

```text
Merge
 ↓
Create Tag
 ↓
GitHub Release
 ↓
Actions
 ↓
Build
 ↓
Artifactory
```

---

## 70. Release Artifact Separation

GitHub Release assets and Artifactory packages serve different purposes.

Artifactory should remain the controlled enterprise package source when it is the organization's artifact system of record.

---

# PART XX — ARTIFACT PROMOTION

## 71. Promotion Concept

```text
Candidate Artifact
       |
       v
Validated
       |
       v
Approved
       |
       v
Production
```

---

## 72. Promotion Identity

Promotion should operate on:

```text
specific artifact/version
```

not:

```text
latest
```

---

## 73. Docker Promotion

Track:

```text
image tag
image digest
Build Info
```

---

# PART XXI — SECURITY SCANNING

## 74. Dependency Scanning

Check:

```text
direct dependencies
transitive dependencies
```

---

## 75. Container Scanning

Check:

```text
OS packages
application libraries
configuration
secrets
known CVEs
```

---

## 76. Secret Scanning

Use GitHub security tooling and/or approved enterprise scanning before publication.

---

## 77. Quality Gate

Example:

```text
Critical vulnerability
 ↓
BLOCK
```

Exact thresholds must come from security policy.

---

# PART XXII — REUSABLE WORKFLOWS

## 78. Why Reusable Workflows?

Large organizations may have:

```text
100+ repositories
```

Repeating Artifactory logic in every repository creates drift.

---

## 79. Central Workflow

Conceptually:

```text
Application Repo
       |
       v
Reusable Workflow
       |
       +--> Authenticate
       +--> Build
       +--> Scan
       +--> Publish
       +--> Build Info
```

---

## 80. Benefits

```text
standardization
central security fixes
consistent permissions
less YAML duplication
faster onboarding
```

---

# PART XXIII — COMPOSITE ACTIONS

## 81. Composite Action

Reusable logic can also be packaged into a composite action.

Example conceptual tasks:

```text
configure Artifactory
configure package manager
publish artifact
publish Build Info
```

---

## 82. Security

Pin internal reusable actions to trusted references.

Avoid blindly consuming:

```text
third-party@main
```

for security-sensitive operations.

---

# PART XXIV — SELF-HOSTED RUNNERS

## 83. Why Self-Hosted?

Self-hosted runners may be used when organizations require:

```text
private network access
custom tooling
specialized hardware
internal Artifactory connectivity
```

---

## 84. Self-Hosted Architecture

```text
GitHub
 ↓
Self-hosted Runner
 ↓
Private Network
 ↓
Artifactory
```

---

## 85. Security Requirements

Control:

```text
runner registration
network access
workspace cleanup
credential exposure
software updates
runner isolation
```

---

## 86. Ephemeral Runners

Preferred for high-security workloads where practical:

```text
Job starts
 ↓
Runner created
 ↓
Job executes
 ↓
Runner destroyed
```

This reduces persistence between jobs.

---

# PART XXV — PRIVATE ARTIFACTORY NETWORKING

## 87. Private Artifactory

If Artifactory is not internet-facing:

```text
GitHub-hosted runner
        |
        X
```

may not be able to reach it directly.

---

## 88. Options

Depending on architecture:

```text
self-hosted runner
private connectivity
proxy
network gateway
approved ingress
```

---

## 89. DNS

Ensure the runner can resolve:

```text
artifactory.company.internal
```

---

## 90. TLS

Use valid certificates.

Verify:

```text
certificate chain
hostname
expiration
trust store
```

Do not disable TLS verification as a permanent production fix.

---

# PART XXVI — AWS + GITHUB ACTIONS + ARTIFACTORY

## 91. Example Architecture

```text
GitHub
  |
  v
GitHub Actions
  |
  +--> OIDC --> AWS
  |
  v
Artifactory
  |
  v
EKS
```

The AWS and Artifactory authentication paths should be independently scoped.

---

## 92. EKS Deployment

```text
GitHub Actions
 ↓
Artifactory
 ↓
Docker Image
 ↓
EKS
```

---

## 93. Runtime Pull

Kubernetes nodes/workloads pull from Artifactory using an appropriate registry authentication design.

The GitHub Actions deployment credential should not automatically become the runtime image-pull credential.

---

# PART XXVII — PRODUCTION SECURITY

## 94. Branch Protection

Production publishing should normally be restricted to:

```text
protected branches
release tags
approved environments
```

---

## 95. Pull Request Restrictions

Do not let untrusted PR code access:

```text
production Artifactory credentials
```

---

## 96. Environment Protection

Use:

```text
production environment
required reviewers
restricted deployment branches/tags
```

where appropriate.

---

## 97. Token Rotation

A mature process:

```text
Create new credential
 ↓
Update GitHub secret
 ↓
Run test workflow
 ↓
Revoke old credential
 ↓
Audit
```

---

# PART XXVIII — FAILURE HANDLING

## 98. Artifactory Unavailable

The workflow should fail clearly.

Do not:

```text
pretend publication succeeded
```

---

## 99. Retry

Transient failures may justify controlled retries.

Example:

```text
network timeout
 ↓
retry
 ↓
success
```

Avoid infinite retries.

---

## 100. Partial Publication

If multiple artifacts are published and one fails:

```text
Identify published artifacts
 ↓
Determine release state
 ↓
Do not create ambiguous production state
```

Use immutable versions and cleanup/promotion policy.

---

# PART XXIX — TROUBLESHOOTING

## 101. 401 Unauthorized

Check:

```text
secret exists
token valid
token expired?
wrong identity?
wrong Artifactory URL?
```

---

## 102. 403 Forbidden

Check:

```text
permission target
repository
READ vs DEPLOY
service identity
project membership
```

---

## 103. DNS Failure

Symptoms:

```text
Could not resolve host
```

Check:

```text
DNS
runner network
private endpoint
proxy
```

---

## 104. TLS Failure

Symptoms:

```text
certificate verify failed
```

Check:

```text
certificate chain
hostname
runner trust store
expiration
```

---

## 105. Docker Push Denied

Check:

```text
docker login
repository path
DEPLOY permission
version policy
token
```

---

## 106. Maven Cannot Download Dependencies

Check:

```text
settings.xml
Artifactory virtual repository
credential
repository configuration
network
```

---

## 107. NPM Authentication Failure

Check:

```text
.npmrc
registry URL
token
repository permissions
```

---

## 108. Build Info Missing

Check:

```text
build name
build number
JFrog CLI
Build Info publication
service identity permissions
```

---

## 109. Workflow Secret Not Available

Check:

```text
repository/environment scope
branch restrictions
fork workflow
secret name
environment approval
```

---

# PART XXX — REAL-WORLD SCENARIOS

## 110. Scenario — PR Can Publish to Production

This is a serious security issue.

Investigate:

```text
workflow permissions
environment protection
secret exposure
Artifactory service identity
branch protection
```

Fix:

```text
remove production secret from PR jobs
restrict publishing to release workflow
restrict Artifactory permissions
```

---

## 111. Scenario — Fork Can Access Artifactory Token

Response:

```text
Revoke/rotate token
 ↓
Audit usage
 ↓
Restrict secrets
 ↓
Separate PR and release workflows
 ↓
Use trusted environments
```

---

## 112. Scenario — Developer Can Push Any Docker Tag

Problem:

```text
broad DEPLOY permission
```

Fix:

```text
restrict repository/path
use release workflow
enforce version policy
```

---

## 113. Scenario — Production Uses Different Build

Problem:

```text
Stage image digest:
A

Production image digest:
B
```

Likely cause:

```text
rebuild during deployment
```

Fix:

```text
build once
promote same digest
```

---

## 114. Scenario — Token Leaked in Workflow Log

Response:

```text
Rotate immediately
 ↓
Inspect GitHub logs
 ↓
Inspect Artifactory audit
 ↓
Identify accessed repositories
 ↓
Review published/deleted artifacts
 ↓
Fix logging
```

---

## 115. Scenario — Artifactory Private but GitHub Runner Is Public

Solution options:

```text
self-hosted runner with private connectivity
```

or an approved secure connectivity architecture.

Do not expose Artifactory publicly merely to simplify CI connectivity without a security review.

---

# PART XXXI — ENTERPRISE ARCHITECTURE

## 116. Large Organization

```text
                 GitHub Enterprise
                        |
              +---------+---------+
              |                   |
        Application Repos     Shared Workflows
              |                   |
              +---------+---------+
                        |
                        v
                 GitHub Actions
                        |
          +-------------+-------------+
          |             |             |
        Build          Scan        Release
          |             |             |
          +-------------+-------------+
                        |
                        v
                   Artifactory
                /       |        \
             Maven    Docker     Helm
                        |
                        v
                       EKS
```

---

## 117. Shared Workflow Model

```text
Application repository
        |
        v
Central reusable workflow
        |
        +--> authentication
        +--> build
        +--> test
        +--> scan
        +--> publish
        +--> Build Info
        +--> promotion
```

---

## 118. Identity Model

```text
Payment Repository
        |
        v
svc-gha-payment
        |
        +--> READ common virtual repos
        |
        +--> DEPLOY payment repos
```

Another team:

```text
Orders Repository
        |
        v
svc-gha-orders
        |
        +--> READ common virtual repos
        |
        +--> DEPLOY orders repos
```

This limits cross-team blast radius.

---

# PART XXXII — OBSERVABILITY AND AUDIT

## 119. Monitor Workflow Failures

Track:

```text
authentication failures
authorization failures
publication failures
promotion failures
deployment failures
```

---

## 120. Artifactory Audit

Correlate:

```text
GitHub workflow
service identity
artifact
repository
timestamp
```

---

## 121. Security Alert

An unexpected artifact publication should trigger investigation:

```text
Who?
Which repository?
Which workflow?
Which commit?
Which artifact?
Which digest?
```

---

# PART XXXIII — ROLLBACK

## 122. Rollback Architecture

```text
Production
   |
   v
Current:
4.2.2
   |
failure
   |
   v
Known Good:
4.2.1
```

---

## 123. Rollback Using Immutable Artifact

Use:

```text
4.2.1
```

or:

```text
sha256:...
```

not:

```text
latest
```

---

## 124. GitOps Rollback

For GitOps:

```text
Git desired state
 ↓
previous known-good version
 ↓
Argo CD
 ↓
Kubernetes
```

The artifact must remain available in Artifactory.

---

# PART XXXIV — PRODUCTION CHECKLIST

## 125. GitHub

```text
[ ] branch protection
[ ] minimal permissions
[ ] protected environments
[ ] required reviewers
[ ] secrets scoped
[ ] reusable workflows reviewed
```

---

## 126. Artifactory

```text
[ ] service identity
[ ] least privilege
[ ] repository boundaries
[ ] immutable releases
[ ] Build Info
[ ] retention
```

---

## 127. Authentication

```text
[ ] token rotation
[ ] OIDC evaluated where appropriate
[ ] no hardcoded credentials
[ ] no secrets in logs
```

---

## 128. Pipeline

```text
[ ] build
[ ] test
[ ] scan
[ ] publish
[ ] Build Info
[ ] promotion
[ ] deployment
```

---

## 129. Network

```text
[ ] DNS
[ ] TLS
[ ] routing
[ ] firewall
[ ] proxy
[ ] private connectivity
```

---

## 130. Kubernetes

```text
[ ] immutable image
[ ] digest tracking
[ ] runtime credentials separate
[ ] rollback tested
```

---

# PART XXXV — INTERVIEW PREPARATION

## 131. How Do You Integrate GitHub Actions with Artifactory?

Answer:

```text
I configure GitHub Actions to authenticate to Artifactory using a
dedicated service identity and scoped credential. The workflow
resolves dependencies from Artifactory virtual repositories, builds
and tests the application, scans it, publishes immutable artifacts to
local repositories, publishes Build Info and promotes the same
artifact through the release environments.
```

---

## 132. How Do You Secure GitHub Actions Secrets?

Answer:

```text
I keep Artifactory credentials in GitHub Secrets or an approved
external secret mechanism, scope them to trusted environments, avoid
exposing them to untrusted pull requests and rotate them regularly.
For suitable architectures I evaluate OIDC/federated identity to
reduce long-lived credentials.
```

---

## 133. Why Should PR Workflows Not Get Production Credentials?

Answer:

```text
A pull request workflow can execute code from the proposed change.
If a malicious change can access production credentials, it could
publish, modify or delete production artifacts. I isolate PR jobs
from production release jobs and secrets.
```

---

## 134. How Do You Implement Build Once and Promote?

Answer:

```text
The release workflow builds the artifact once, tests and scans it,
publishes it to Artifactory and records Build Info. The exact
artifact version or digest is then promoted to staging and production
without rebuilding.
```

---

## 135. How Do You Secure Docker Publishing?

Answer:

```text
I use a dedicated Artifactory service identity with DEPLOY access
only to the required Docker repository, authenticate securely from
GitHub Actions, scan the image before release, use immutable version
tags and capture the image digest.
```

---

## 136. How Do You Handle Artifactory 403?

Answer:

```text
I verify the authenticated identity and then check repository access,
permission targets, project membership and whether the requested
operation is READ or DEPLOY. I also verify that the workflow is using
the intended credential.
```

---

## 137. How Do You Handle a Leaked GitHub Actions Token?

Answer:

```text
I immediately revoke or rotate the credential, inspect Artifactory
audit logs for activity performed by that identity, determine which
repositories and artifacts were accessed, fix the workflow exposure
and verify production integrity.
```

---

## 138. How Do You Connect an Image to a Git Commit?

Answer:

```text
I use the image digest/tag, Build Info, GitHub Actions run metadata
and commit SHA. This creates a chain from the production image to the
exact source revision and CI execution that produced it.
```

---

## 139. How Do You Design GitHub Actions for 100+ Repositories?

Answer:

```text
I centralize common Artifactory logic in reviewed reusable workflows
or composite actions, use repository-specific service identities or
scoped permissions, standardize naming and versioning and keep
production release workflows protected by environments and
approvals.
```

---

# PART XXXVI — GOLDEN RULES

## 140. Rules

```text
1. GitHub Actions should use a dedicated Artifactory identity.

2. Never use an Artifactory administrator credential for normal CI.

3. Keep Artifactory credentials out of Git.

4. Do not expose production credentials to untrusted pull requests.

5. Use GitHub Environment protection for sensitive production flows.

6. Keep workflow permissions minimal.

7. Prefer short-lived/federated credentials where the architecture
   supports them.

8. Rotate long-lived machine credentials.

9. Resolve dependencies through controlled Artifactory virtual
   repositories.

10. Publish artifacts only to approved local repositories.

11. Use immutable release versions.

12. Capture Docker image digests.

13. Publish Build Info.

14. Connect Git commit, workflow run, artifact and deployment.

15. Build once and promote the exact artifact.

16. Do not rebuild for each environment.

17. Scan dependencies and container images before release.

18. Treat critical security findings as release blockers according
    to organizational policy.

19. Use reusable workflows for enterprise standardization.

20. Review third-party GitHub Actions before using them in production.

21. Pin security-sensitive actions according to organizational
    supply-chain policy.

22. Use ephemeral/self-hosted runners where private connectivity or
    isolation requires them.

23. Never disable TLS verification as a permanent troubleshooting fix.

24. Audit artifact publication and promotion.

25. Keep rollback artifacts available.

26. Separate CI credentials from Kubernetes runtime credentials.

27. Do not assume a GitHub workflow's AWS identity should have
    Artifactory administrative access.

28. Fail the pipeline clearly when artifact publication fails.

29. Treat leaked GitHub Actions credentials as supply-chain security
    incidents.

30. Validate exact GitHub Actions, JFrog CLI, JFrog integration,
    Artifactory and identity-provider behavior against the deployed
    versions before production rollout.
```

---

# END OF 16-GitHub-Actions-Artifactory-Integration.md
