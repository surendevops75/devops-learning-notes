# GitLab-Artifactory-Integration

## 1. Purpose

This file provides a production-oriented guide to integrating GitLab CI/CD with JFrog Artifactory.

The objective is to build a secure and traceable pipeline that can:

```text
checkout source
build
test
scan
resolve dependencies
publish artifacts
publish Build Info
promote releases
deploy
rollback
audit
```

Core architecture:

```text
Developer
   |
   v
GitLab Repository
   |
   v
GitLab Runner
   |
   +--> Build
   +--> Test
   +--> Scan
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

# PART I — GITLAB + ARTIFACTORY FUNDAMENTALS

## 2. What Is GitLab CI/CD?

GitLab CI/CD automates:

```text
build
test
security
package
release
deploy
```

Pipeline configuration is commonly stored in:

```text
.gitlab-ci.yml
```

---

## 3. What Is Artifactory?

Artifactory is the centralized artifact repository.

It can manage:

```text
Maven
Gradle
NPM
PyPI
Docker/OCI
Helm
Generic packages
```

---

## 4. Why Integrate GitLab with Artifactory?

GitLab provides:

```text
source control
merge requests
pipeline orchestration
runners
environment controls
```

Artifactory provides:

```text
artifact storage
dependency management
repository federation/proxying
RBAC
Build Info
promotion
retention
```

---

## 5. Basic Pipeline

```text
GitLab
  |
  v
Pipeline
  |
  +--> Build
  +--> Test
  +--> Scan
  |
  v
Artifactory
  |
  +--> Publish
  +--> Build Info
  +--> Promote
  |
  v
Deployment
```

---

# PART II — REPOSITORY ARCHITECTURE

## 6. Dependency Repository

Recommended:

```text
GitLab Runner
      |
      v
Artifactory Virtual
      |
      +--> Local
      |
      +--> Remote
```

This centralizes package resolution.

---

## 7. Publication Repository

Production pipelines publish to local repositories:

```text
GitLab Runner
      |
      v
Artifactory Local
```

Examples:

```text
maven-local
npm-local
pypi-local
docker-local
helm-local
```

---

## 8. Repository Separation

Use repository boundaries for:

```text
development
release
team
application
package type
```

according to enterprise governance.

---

# PART III — AUTHENTICATION

## 9. Authentication vs Authorization

Authentication:

```text
Who is calling?
```

Authorization:

```text
What may that identity do?
```

Example:

```text
Authentication succeeds
+
DEPLOY permission missing
=
403
```

---

## 10. GitLab Runner Identity

A runner should use a dedicated Artifactory machine identity.

Example:

```text
svc-gitlab-payment
```

---

## 11. Least Privilege

Example:

```text
READ:
maven-virtual
docker-virtual

DEPLOY:
payment-maven-local
payment-docker-local

DELETE:
NO

ADMIN:
NO
```

---

## 12. Never Use Admin Credentials

Avoid:

```text
GitLab Runner
    |
    v
Artifactory Admin
```

A compromised runner could then have excessive access.

---

# PART IV — GITLAB CI VARIABLES

## 13. GitLab CI/CD Variables

Store sensitive values in:

```text
GitLab CI/CD Variables
```

Examples:

```text
ARTIFACTORY_URL
ARTIFACTORY_USER
ARTIFACTORY_TOKEN
```

---

## 14. Protected Variables

Production credentials should normally be:

```text
Protected
```

so they are available only to trusted branches/tags according to the GitLab security model.

---

## 15. Masked Variables

Secrets should be:

```text
Masked
```

where supported by GitLab's variable validation requirements.

---

## 16. Environment Scope

Separate:

```text
dev
stage
prod
```

credentials when different trust boundaries exist.

---

## 17. Do Not Hardcode Secrets

Bad:

```yaml
variables:
  ARTIFACTORY_TOKEN: "my-secret-token"
```

Good:

```yaml
script:
  - ./publish.sh
```

with the token supplied securely by GitLab.

---

# PART V — GITLAB OIDC / FEDERATED IDENTITY

## 18. Why Federation?

Long-lived credentials create:

```text
storage risk
rotation burden
blast radius
```

Federation can enable short-lived credentials where the target identity architecture supports it.

---

## 19. Generic Federation Flow

```text
GitLab Job
   |
   v
OIDC Identity
   |
   v
Trusted Identity Provider
   |
   v
Short-Lived Credential
   |
   v
Artifactory
```

The exact implementation depends on the deployed GitLab and JFrog identity capabilities.

---

## 20. Trust Conditions

Trust should be restricted using appropriate identity claims such as:

```text
project
group
branch
tag
environment
job
```

Never create an unrestricted trust relationship.

---

# PART VI — BASIC GITLAB PIPELINE

## 21. Example

```yaml
stages:
  - build
  - test
  - scan
  - publish

build:
  stage: build
  image: maven:3.9-eclipse-temurin-21
  script:
    - mvn clean package

test:
  stage: test
  image: maven:3.9-eclipse-temurin-21
  script:
    - mvn test

publish:
  stage: publish
  script:
    - ./scripts/publish.sh
```

Production pipelines should pin trusted images/actions according to organizational supply-chain policy.

---

# PART VII — GITLAB RUNNERS

## 22. What Is a Runner?

A runner executes GitLab CI jobs.

It may be:

```text
GitLab-hosted
self-hosted
Kubernetes-based
ephemeral
```

---

## 23. GitLab Runner + Artifactory

```text
GitLab
  |
  v
Runner
  |
  v
Artifactory
```

---

## 24. Self-Hosted Runner

Useful when Artifactory is private:

```text
GitLab
  |
  v
Private Runner
  |
  v
Private Artifactory
```

---

## 25. Runner Security

Control:

```text
runner registration
network access
workspace
credentials
executor
software versions
```

---

## 26. Ephemeral Runner

Preferred for sensitive workloads where practical:

```text
Job
 ↓
Runner created
 ↓
Build
 ↓
Publish
 ↓
Runner destroyed
```

This reduces persistence.

---

# PART VIII — MAVEN

## 27. Maven Dependency Resolution

```text
GitLab Runner
      |
      v
Maven
      |
      v
Artifactory maven-virtual
```

---

## 28. Maven Publish

```text
mvn deploy
     |
     v
maven-local
```

---

## 29. settings.xml

Use a secure settings configuration.

Concept:

```xml
<server>
  <id>company-artifactory</id>
  <username>${env.ARTIFACTORY_USER}</username>
  <password>${env.ARTIFACTORY_TOKEN}</password>
</server>
```

Do not commit real credentials.

---

## 30. Maven Release

Example:

```text
payment-service-4.2.1.jar
```

Release artifacts should be immutable.

---

# PART IX — GRADLE

## 31. Gradle Flow

```text
GitLab
 ↓
Gradle
 ↓
Artifactory
```

---

## 32. Dependency Resolution

```text
Gradle
 ↓
maven-virtual
```

---

## 33. Publication

```text
Gradle
 ↓
maven-local
```

Use dedicated deployment permissions.

---

# PART X — NPM

## 34. NPM Dependency Flow

```text
npm ci
 ↓
Artifactory npm-virtual
```

---

## 35. NPM Publish

```bash
npm publish
```

Target the approved Artifactory NPM repository.

---

## 36. Temporary .npmrc

A pipeline may generate `.npmrc` dynamically.

Example:

```text
registry=https://artifactory.example.com/artifactory/api/npm/npm-virtual/
```

Authentication must come from secure CI variables.

---

# PART XI — PYPI

## 37. Python Build

```bash
python -m build
```

---

## 38. Python Package

Example:

```text
dist/
 ├── payment-4.2.1.tar.gz
 └── payment-4.2.1-py3-none-any.whl
```

---

## 39. Python Publication

```text
GitLab Runner
 ↓
PyPI local
```

Use approved publishing tooling and scoped credentials.

---

# PART XII — DOCKER

## 40. Docker Flow

```text
GitLab Runner
      |
      v
docker build
      |
      v
Security Scan
      |
      v
docker push
      |
      v
Artifactory Docker
```

---

## 41. Docker Authentication

Example:

```bash
docker login artifactory.example.com
```

Use GitLab-provided secret variables rather than literal credentials.

---

## 42. Docker Build

```bash
docker build \
  -t artifactory.example.com/docker-local/payment-service:4.2.1 \
  .
```

---

## 43. Docker Push

```bash
docker push \
  artifactory.example.com/docker-local/payment-service:4.2.1
```

---

## 44. Docker Digest

Capture:

```text
sha256:...
```

Production deployments should use an immutable identity.

---

# PART XIII — HELM

## 45. Helm Validation

```bash
helm lint ./chart
helm template ./chart
```

---

## 46. Helm Package

```bash
helm package ./chart
```

---

## 47. Helm Publication

Publish to the approved Artifactory Helm or OCI repository.

---

## 48. Version Relationship

Example:

```text
Chart:
1.4.0

Application:
4.2.1

Image:
4.2.1
```

The pipeline should preserve traceability.

---

# PART XIV — JFROG CLI

## 49. Why JFrog CLI?

JFrog CLI can automate:

```text
upload
download
search
build-info
promotion
```

---

## 50. GitLab CLI Configuration

The pipeline should configure the CLI using secure CI variables or the organization's approved authentication method.

Do not echo secrets.

---

## 51. Upload Example

Conceptually:

```bash
jf rt upload \
  "dist/*.jar" \
  "maven-local/com/company/payment/"
```

---

## 52. Download Example

Conceptually:

```bash
jf rt download \
  "maven-virtual/com/company/payment/*"
```

Use exact repository paths and filters in production.

---

# PART XV — BUILD INFO

## 53. Build Info

Build Info connects:

```text
GitLab pipeline
+
source
+
dependencies
+
artifacts
```

---

## 54. Build Name

Example:

```text
payment-service
```

---

## 55. Build Number

Possible source:

```text
CI_PIPELINE_ID
```

Use a consistent enterprise convention.

---

## 56. Useful GitLab Metadata

Examples include:

```text
CI_COMMIT_SHA
CI_COMMIT_REF_NAME
CI_COMMIT_TAG
CI_PIPELINE_ID
CI_JOB_ID
CI_PROJECT_PATH
```

Only publish approved non-sensitive metadata.

---

# PART XVI — BUILD ONCE, PROMOTE MANY

## 57. Correct Pattern

```text
GitLab
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Build Info
 ↓
Promote
 ↓
Stage
 ↓
Production
```

---

## 58. Wrong Pattern

```text
Build DEV
 ↓
Rebuild STAGE
 ↓
Rebuild PROD
```

---

## 59. Why Rebuild Is Dangerous

Build inputs can change:

```text
dependencies
base image
compiler
package repository
environment
toolchain
```

Therefore the production bytes may differ from the tested bytes.

---

# PART XVII — RELEASE TAGS

## 60. Tag-Driven Release

Example:

```text
v4.2.1
```

Flow:

```text
Git tag
 ↓
GitLab pipeline
 ↓
Build
 ↓
Scan
 ↓
Publish
```

---

## 61. Version Validation

Before publishing:

```text
extract version
 ↓
validate
 ↓
check existing artifact
 ↓
publish only if valid
```

---

## 62. Immutable Releases

Never silently overwrite:

```text
4.2.1
```

with a different binary.

Use a new release version for corrected content.

---

# PART XVIII — GITLAB ENVIRONMENTS

## 63. Environment Model

Example:

```text
dev
stage
prod
```

---

## 64. Production Environment

Production should be protected using GitLab's environment and deployment controls where applicable.

Possible controls:

```text
protected environment
protected branches/tags
deployment approvals
restricted users
```

---

## 65. Promotion

```text
Stage
 ↓
Approval
 ↓
Artifactory Promotion
 ↓
Production
```

---

# PART XIX — SECURITY SCANNING

## 66. Pipeline Security

Recommended stages:

```text
Build
 ↓
Unit Test
 ↓
SAST
 ↓
Dependency Scan
 ↓
Container Scan
 ↓
Publish
```

---

## 67. Critical Vulnerability

Example policy:

```text
Critical
 ↓
BLOCK
```

Use organizational risk policy for actual thresholds.

---

## 68. Secret Scanning

Scan:

```text
Git repository
dependencies
build output
container
```

according to enterprise tooling.

---

# PART XX — GITLAB SECURITY VARIABLES

## 69. Protected Variable

Production credential should not be available to arbitrary branches.

Concept:

```text
protected tag/branch
      |
      v
production variable
```

---

## 70. Masked Variable

Prevent accidental display in logs.

Still avoid:

```bash
echo "$ARTIFACTORY_TOKEN"
```

---

# PART XXI — REUSABLE GITLAB COMPONENTS

## 71. Include Templates

Large organizations can centralize pipeline logic.

Example:

```yaml
include:
  - project: platform/ci-templates
    file: /artifactory-publish.yml
```

---

## 72. Shared Pipeline Template

Conceptually:

```text
Application Repo
      |
      v
Central CI Template
      |
      +--> Authenticate
      +--> Build
      +--> Scan
      +--> Publish
      +--> Build Info
      +--> Promote
```

---

## 73. Benefits

```text
standardization
central security fixes
less duplication
consistent Artifactory usage
```

---

# PART XXII — CHILD AND PARENT PIPELINES

## 74. Parent Pipeline

Can orchestrate:

```text
build
security
release
deployment
```

---

## 75. Child Pipeline

Can isolate:

```text
application build
container build
deployment
```

---

## 76. Artifact Handoff

Use immutable identifiers:

```text
version
digest
artifact path
Build Info
```

Do not pass:

```text
latest
```

as a production artifact identity.

---

# PART XXIII — PARALLEL JOBS

## 77. Parallel Build

Example:

```text
             +--> Maven
             |
GitLab ------+--> NPM
             |
             +--> Docker
```

---

## 78. Shared Release Identity

All outputs should be associated with:

```text
Git commit
pipeline ID
release version
```

---

## 79. Collision Prevention

Avoid two pipelines publishing the same immutable release version.

Use:

```text
release tags
version validation
protected release workflows
```

---

# PART XXIV — KUBERNETES RUNNERS

## 80. Kubernetes Executor

GitLab Runner can create jobs as Kubernetes workloads.

Architecture:

```text
GitLab
  |
  v
Runner
  |
  v
Kubernetes
  |
  +--> Job Pod
          |
          v
      Artifactory
```

---

## 81. EKS

Example:

```text
AWS EKS
 |
 +--> GitLab Runner
 |
 +--> Job Pods
```

Job pods access Artifactory through approved network paths.

---

## 82. Ephemeral Job Pods

After the job:

```text
Pod destroyed
```

This reduces persistent workspace risk.

---

# PART XXV — PRIVATE ARTIFACTORY

## 83. Private Network

```text
GitLab
  |
  v
Private Runner
  |
  v
Private Artifactory
```

---

## 84. DNS

Runner must resolve:

```text
artifactory.company.internal
```

---

## 85. TLS

Verify:

```text
certificate
hostname
trust chain
expiration
```

Never permanently disable certificate verification to bypass a problem.

---

## 86. Firewall

Permit only required traffic:

```text
Runner
 ↓
443
 ↓
Artifactory
```

Avoid broad network access.

---

# PART XXVI — AWS + GITLAB + ARTIFACTORY

## 87. Example

```text
GitLab
  |
  v
GitLab Runner
  |
  +----> Artifactory
  |
  +----> AWS
             |
             v
            EKS
```

AWS credentials and Artifactory credentials should be independently scoped.

---

## 88. EKS Deployment

```text
GitLab Pipeline
      |
      v
Artifactory
      |
      v
Docker Image
      |
      v
EKS
```

---

## 89. Runtime Credentials

Do not automatically reuse:

```text
GitLab CI Artifactory credentials
```

as Kubernetes runtime credentials.

Runtime access should have its own identity model.

---

# PART XXVII — GITLAB DEPLOYMENT STRATEGY

## 90. Continuous Delivery

Pipeline:

```text
Commit
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Stage
```

Production may require approval.

---

## 91. Continuous Deployment

If policy permits:

```text
Commit
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Promote
 ↓
Deploy
```

---

## 92. Production Approval

Approval should apply to:

```text
specific release
specific artifact
specific digest
```

not a mutable reference.

---

# PART XXVIII — ROLLBACK

## 93. Rollback

Example:

```text
Current:
4.2.2

Known Good:
4.2.1
```

---

## 94. Rollback Flow

```text
Select known-good version
 ↓
Verify artifact
 ↓
Deploy
 ↓
Smoke test
 ↓
Monitor
```

---

## 95. Digest Rollback

For Docker:

```text
sha256:known-good
```

is stronger than:

```text
latest
```

---

# PART XXIX — TROUBLESHOOTING

## 96. 401 Unauthorized

Check:

```text
CI variable
token
token expiration
identity
Artifactory URL
```

---

## 97. 403 Forbidden

Check:

```text
permission target
repository
READ
DEPLOY
DELETE
project permissions
```

---

## 98. GitLab Variable Missing

Check:

```text
protected variable
branch/tag
environment scope
variable name
pipeline source
```

---

## 99. Fork Pipeline Cannot Access Secret

This may be intentional.

Check:

```text
fork security model
protected variables
pipeline source
```

Do not simply expose the production secret.

---

## 100. Docker Push Failed

Check:

```text
docker login
registry path
token
DEPLOY permission
repository
TLS
DNS
```

---

## 101. Maven Dependency Download Failed

Check:

```text
settings.xml
virtual repository
credentials
network
repository availability
```

---

## 102. NPM Publish Failed

Check:

```text
.npmrc
registry
token
repository permissions
package version
```

---

## 103. Build Info Missing

Check:

```text
build name
pipeline ID
JFrog CLI
publication command
identity permission
```

---

## 104. Artifactory Host Unreachable

Check:

```bash
nslookup artifactory.example.com
curl -vk https://artifactory.example.com/
```

Then investigate:

```text
DNS
routing
firewall
proxy
TLS
```

---

# PART XXX — REAL-WORLD SCENARIOS

## 105. Scenario — Production Token Available to Feature Branch

Problem:

```text
feature branch
 ↓
production token
```

Fix:

```text
protect variable
 ↓
restrict production environment
 ↓
allow release tags only
 ↓
review pipeline
```

---

## 106. Scenario — Fork Job Attempts Artifact Publication

Response:

```text
Do not expose production credentials
 ↓
separate PR pipeline
 ↓
trusted release pipeline
 ↓
audit
```

---

## 107. Scenario — Same Version Published Twice

Example:

```text
4.2.1
```

contains different bytes.

Fix:

```text
enforce immutability
 ↓
fail duplicate publication
 ↓
create corrected version
```

---

## 108. Scenario — Production Differs from Stage

Check:

```text
artifact digest
Build Info
pipeline ID
deployment configuration
```

If digests differ:

```text
rebuild occurred
```

or a different artifact was selected.

---

## 109. Scenario — Runner Compromised

Response:

```text
Stop runner
 ↓
Revoke Artifactory credentials
 ↓
Review GitLab audit
 ↓
Review Artifactory audit
 ↓
Identify artifacts
 ↓
Check production deployments
 ↓
Rotate credentials
```

---

## 110. Scenario — Private Artifactory Cannot Be Reached

Do not immediately make Artifactory public.

Investigate:

```text
private runner
DNS
routing
security group
firewall
proxy
TLS
```

---

# PART XXXI — ENTERPRISE ARCHITECTURE

## 111. Enterprise Model

```text
                    GitLab
                      |
             +--------+--------+
             |                 |
        Application Repos   CI Templates
             |                 |
             +--------+--------+
                      |
                      v
                 GitLab Runner
                      |
          +-----------+-----------+
          |           |           |
        Build        Test        Scan
          |           |           |
          +-----------+-----------+
                      |
                      v
                 Artifactory
               /      |       \
            Maven    Docker    Helm
                      |
                      v
                     EKS
```

---

## 112. Team Isolation

```text
Team A
 ↓
svc-gitlab-team-a
 ↓
team-a repositories

Team B
 ↓
svc-gitlab-team-b
 ↓
team-b repositories
```

Both can use common read-only virtual repositories.

---

## 113. Central CI Governance

```text
GitLab Group
   |
   v
Central CI Templates
   |
   +--> Authentication
   +--> Security
   +--> Artifactory
   +--> Build Info
   +--> Promotion
```

---

# PART XXXII — AUDIT AND OBSERVABILITY

## 114. Pipeline Audit

Track:

```text
project
pipeline ID
commit
user
runner
artifact
release
```

---

## 115. Artifactory Audit

Track:

```text
identity
repository
artifact
operation
timestamp
```

---

## 116. Correlation

During an incident:

```text
Production Image
 ↓
Artifact
 ↓
Build Info
 ↓
GitLab Pipeline
 ↓
Commit
 ↓
Merge Request
```

---

# PART XXXIII — PERFORMANCE AND RELIABILITY

## 117. Dependency Caching

Artifactory remote repositories can cache external dependencies according to repository configuration.

Benefits:

```text
faster builds
less external dependency
better availability
central governance
```

---

## 118. Parallel Jobs

Use parallelism for independent stages.

Avoid unnecessary parallel jobs that:

```text
publish same version
modify same artifact
race for release
```

---

## 119. Retry Strategy

Retry only transient failures:

```text
network timeout
temporary service unavailable
```

Do not repeatedly retry:

```text
401
403
invalid version
policy failure
```

---

# PART XXXIV — SECURITY BEST PRACTICES

## 120. GitLab

```text
[ ] protected branches
[ ] protected tags
[ ] protected variables
[ ] minimal job permissions
[ ] reviewed CI templates
```

---

## 121. Runner

```text
[ ] isolated
[ ] patched
[ ] ephemeral where practical
[ ] restricted network
[ ] workspace cleanup
```

---

## 122. Artifactory

```text
[ ] service identity
[ ] least privilege
[ ] repository boundaries
[ ] immutable releases
[ ] audit
```

---

## 123. Network

```text
[ ] TLS
[ ] DNS
[ ] private connectivity
[ ] firewall
[ ] restricted ports
```

---

# PART XXXV — INTERVIEW PREPARATION

## 124. How Do You Integrate GitLab CI with Artifactory?

Answer:

```text
I configure GitLab Runner jobs to authenticate to Artifactory using a
dedicated least-privilege service identity. Dependencies are resolved
through Artifactory virtual repositories, artifacts are published to
local repositories, Build Info is recorded and approved artifacts are
promoted through environments without rebuilding.
```

---

## 125. How Do You Secure GitLab CI Credentials?

Answer:

```text
I store credentials in GitLab protected and masked CI/CD variables or
use an approved federated identity mechanism. Production credentials
are restricted to protected branches, tags or environments and are
never hardcoded in the repository.
```

---

## 126. Why Use Protected Variables?

Answer:

```text
They prevent sensitive credentials from being exposed to arbitrary
branches or pipeline contexts. Production credentials should be
available only to trusted release workflows.
```

---

## 127. How Do You Handle Fork Pipelines?

Answer:

```text
I treat fork pipelines as untrusted execution. I do not expose
production Artifactory credentials to them. Pull request validation
is separated from trusted release workflows.
```

---

## 128. How Do You Publish Docker Images?

Answer:

```text
The GitLab Runner authenticates to the Artifactory Docker registry,
builds and scans the image, pushes it to the approved local
repository and records the resulting digest and Build Info.
```

---

## 129. How Do You Implement Build Once, Promote Many?

Answer:

```text
The pipeline builds the artifact once, validates and scans it,
publishes it to Artifactory and records Build Info. The exact
artifact or image digest is then promoted through staging and
production instead of rebuilding.
```

---

## 130. GitLab Gets 403 From Artifactory. What Do You Check?

Answer:

```text
I check the service identity, Artifactory permission target, target
repository, project access and whether the requested operation is
READ, DEPLOY or DELETE.
```

---

## 131. GitLab Gets 401. What Do You Check?

Answer:

```text
I verify that the CI variable is available to the current pipeline,
the token is valid and not expired, the Artifactory URL is correct and
the workflow is using the intended identity.
```

---

## 132. How Do You Troubleshoot Private Artifactory Access?

Answer:

```text
I verify DNS resolution, routing, proxy configuration, firewall rules,
TLS certificate trust and runner network placement. If Artifactory
is private, I use a runner with approved private connectivity rather
than making the repository publicly accessible.
```

---

## 133. How Do You Trace a Production Image to Git?

Answer:

```text
I start with the production image digest, identify the Artifactory
artifact and Build Info, map it to the GitLab pipeline and then to
the exact Git commit and release tag.
```

---

## 134. How Do You Design GitLab CI for Hundreds of Repositories?

Answer:

```text
I centralize common Artifactory authentication, build, security,
publication and promotion logic in reviewed CI templates or
components. Each application retains controlled repository-specific
permissions while common standards are managed centrally.
```

---

# PART XXXVI — PRODUCTION CHECKLIST

## 135. GitLab

```text
[ ] protected branches
[ ] protected tags
[ ] protected variables
[ ] environment controls
[ ] reviewed templates
[ ] minimal permissions
```

---

## 136. Authentication

```text
[ ] dedicated service identity
[ ] scoped token
[ ] rotation
[ ] federation evaluated
[ ] no hardcoded credentials
```

---

## 137. Artifactory

```text
[ ] virtual repositories for dependencies
[ ] local repositories for publication
[ ] immutable versions
[ ] Build Info
[ ] retention
[ ] audit
```

---

## 138. Pipeline

```text
[ ] build
[ ] test
[ ] security scan
[ ] publish
[ ] Build Info
[ ] promotion
[ ] deployment
[ ] rollback
```

---

## 139. Network

```text
[ ] DNS
[ ] TLS
[ ] routing
[ ] firewall
[ ] proxy
[ ] private connectivity
```

---

## 140. Kubernetes

```text
[ ] image digest tracked
[ ] runtime identity separated
[ ] rollback version retained
[ ] deployment verified
```

---

# PART XXXVII — GOLDEN RULES

## 141. Rules

```text
1. GitLab orchestrates CI/CD; Artifactory manages enterprise artifacts.

2. Use dedicated Artifactory identities for GitLab automation.

3. Never use an Artifactory administrator identity for normal CI.

4. Keep credentials in protected CI/CD variables or approved
   federated identity mechanisms.

5. Never hardcode secrets in .gitlab-ci.yml.

6. Never expose production credentials to untrusted fork pipelines.

7. Use protected branches, protected tags and protected environments
   for production release workflows.

8. Use Artifactory virtual repositories for controlled dependency
   resolution.

9. Publish internal artifacts to approved local repositories.

10. Use immutable release versions.

11. Track Docker image digests.

12. Publish Build Info.

13. Connect GitLab pipeline ID, commit SHA, artifact and deployment.

14. Build once and promote the same artifact.

15. Do not rebuild separately for production.

16. Scan dependencies and containers before trusted promotion.

17. Restrict DELETE and ADMIN permissions.

18. Use self-hosted or ephemeral runners when private connectivity or
   isolation requires them.

19. Keep Artifactory private when possible and provide controlled
   network connectivity.

20. Do not disable TLS certificate verification as a permanent fix.

21. Standardize Artifactory integration using reusable GitLab CI
   templates/components.

22. Review shared CI templates as production code.

23. Audit artifact publication, promotion and deletion.

24. Test rollback using retained artifacts.

25. Correlate GitLab audit information with Artifactory audit data.

26. Treat compromised runners and leaked CI credentials as
   supply-chain security incidents.

27. Fail clearly when artifact publication fails.

28. Do not silently ignore failed uploads.

29. Keep CI identities separate from Kubernetes runtime identities.

30. Validate exact GitLab Runner, JFrog CLI, JFrog integration,
   Artifactory and identity-provider behavior against the deployed
   versions before production rollout.
```

---