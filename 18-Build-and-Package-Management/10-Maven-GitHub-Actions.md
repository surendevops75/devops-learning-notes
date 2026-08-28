# 18-Build-and-Package-Management
# 10-Maven-GitHub-Actions

## 1. Purpose

GitHub Actions can provide the CI/CD automation layer around a Maven
project.

The production model is:

```text
GitHub
  |
  v
Pull Request / Push / Tag
  |
  v
GitHub Actions
  |
  +--> checkout
  +--> JDK
  +--> Maven Wrapper
  +--> dependency resolution
  +--> compile
  +--> test
  +--> verify
  +--> security
  +--> package
  +--> publish
  |
  v
Artifactory / Maven Repository
  |
  v
Container Registry
  |
  v
Deployment / GitOps
```

This file covers GitHub Actions fundamentals, workflow structure,
events, jobs, runners, Maven Wrapper, JDK setup, caching, secrets,
environments, artifacts, Maven repositories, Artifactory, tests,
quality gates, security, matrix builds, reusable workflows, composite
actions, concurrency, permissions, releases, containers, Kubernetes,
OIDC concepts, troubleshooting, production architecture and interview
preparation.

---

# PART I — GITHUB ACTIONS FUNDAMENTALS

## 2. What Is GitHub Actions?

GitHub Actions is GitHub's automation platform for executing workflows
based on repository events or schedules.

A workflow can:

```text
build
test
scan
package
publish
deploy
```

---

## 3. Maven's Role

Maven performs the Java project build.

GitHub Actions orchestrates the workflow.

```text
GitHub Actions
       |
       v
./mvnw -B clean verify
       |
       v
Maven Lifecycle
```

---

## 4. Separation of Responsibility

```text
GitHub Actions
 |
CI/CD orchestration

Maven
 |
Java build lifecycle

Artifactory
 |
Maven artifact repository

Container Registry
 |
OCI images

Kubernetes/GitOps
 |
runtime delivery
```

---

# PART II — WORKFLOW FILE

## 5. Workflow Location

GitHub Actions workflows are normally stored under:

```text
.github/workflows/
```

Example:

```text
.github/
└── workflows/
    └── maven-ci.yml
```

---

## 6. Minimal Workflow

Example:

```yaml
name: Maven CI

on:
  push:
  pull_request:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'

      - name: Build
        run: ./mvnw -B clean verify
```

Pin action versions according to your organization's security and
maintenance policy.

---

# PART III — WORKFLOW STRUCTURE

## 7. Main Components

A workflow commonly contains:

```text
name
on
permissions
env
jobs
```

---

## 8. Job

A job runs on a runner.

Concept:

```text
Workflow
 |
+--> Job A
 |
+--> Job B
```

Jobs can run independently or depend on each other.

---

## 9. Step

A job contains steps.

```text
Job
 |
+--> checkout
+--> setup JDK
+--> cache
+--> Maven
+--> reports
```

---

# PART IV — EVENTS

## 10. Pull Request

Typical:

```yaml
on:
  pull_request:
```

Useful for validation before merging.

---

## 11. Push

```yaml
on:
  push:
```

Can run CI after code changes.

---

## 12. Tag

Release workflows commonly respond to tags.

Concept:

```text
Git tag
 |
v
Release workflow
```

---

## 13. Manual Dispatch

A workflow can also support manually initiated runs.

Use manual inputs carefully and validate them.

---

# PART V — BRANCH STRATEGY

## 14. Pull Request

```text
Developer
 |
v
PR
 |
v
Maven verify
 |
+--> tests
+--> security
+--> quality
```

---

## 15. Main Branch

After merge:

```text
main
 |
v
CI
 |
v
validated artifact
```

---

## 16. Release Tag

```text
v1.4.0
 |
v
Release workflow
 |
v
publish
```

The exact release model depends on the organization.

---

# PART VI — RUNNERS

## 17. GitHub-Hosted Runner

GitHub can provide managed runners.

Example:

```yaml
runs-on: ubuntu-latest
```

---

## 18. Self-Hosted Runner

Organizations can operate their own runners.

Concept:

```text
GitHub
 |
v
Self-hosted Runner
 |
v
Maven
```

---

## 19. Runner Security

Treat runners as privileged build infrastructure.

Consider:

```text
network access
credentials
filesystem
Docker socket
cache
persistence
```

---

# PART VII — EPHEMERAL RUNNERS

## 20. Why Ephemeral?

A clean runner per job reduces:

```text
state leakage
cross-job contamination
credential residue
```

---

## 21. Persistent Runner Risk

Long-lived runners may accumulate:

```text
old Maven caches
workspaces
credentials
temporary files
tools
```

Clean them or use ephemeral runners where practical.

---

# PART VIII — JDK SETUP

## 22. setup-java

Typical:

```yaml
- uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '21'
```

---

## 23. Version Consistency

Control:

```text
JDK
Maven
compiler plugin
project release
```

Example:

```xml
<release>21</release>
```

---

# PART IX — MAVEN WRAPPER

## 24. Use the Wrapper

Prefer:

```bash
./mvnw
```

instead of:

```bash
mvn
```

when the repository maintains the Maven Wrapper.

---

## 25. Why?

Benefits:

```text
Maven version consistency
developer/CI parity
reproducibility
```

---

# PART X — BASIC MAVEN CI

## 26. Build

```yaml
- name: Build
  run: ./mvnw -B clean verify
```

---

## 27. Why Batch Mode?

```bash
-B
```

is appropriate for non-interactive CI.

---

# PART XI — MAVEN CACHE

## 28. setup-java Cache

GitHub's Java setup action can support Maven dependency caching.

Concept:

```yaml
with:
  cache: maven
```

---

## 29. Benefits

```text
faster builds
less repository traffic
lower build latency
```

---

## 30. Cache Risks

```text
stale cache
corrupt cache
incorrect cache keys
cross-environment assumptions
```

---

# PART XII — COLD CACHE

## 31. Why Test Cold Cache?

A warm cache can hide:

```text
repository outage
missing dependency
broken mirror
incorrect repository configuration
```

Periodically validate clean-cache builds.

---

# PART XIII — MAVEN SETTINGS

## 32. settings.xml

Enterprise builds may need:

```text
mirror
repositories
pluginRepositories
server IDs
profiles
```

---

## 33. Do Not Commit Secrets

Never store repository passwords in:

```text
pom.xml
settings.xml committed to Git
workflow YAML
```

Use GitHub secrets, environment-specific credentials or workload
identity mechanisms.

---

# PART XIV — GITHUB SECRETS

## 34. Secrets

Secrets can be supplied to workflows through GitHub's secret
mechanisms.

Concept:

```yaml
env:
  REPO_TOKEN: ${{ secrets.REPO_TOKEN }}
```

Avoid printing secrets.

---

## 35. Secret Scope

Prefer:

```text
repository
environment
organization
```

scope appropriate to the use case.

---

# PART XV — ENVIRONMENTS

## 36. GitHub Environments

Environments can represent:

```text
development
staging
production
```

They can provide environment-specific controls and secrets.

---

## 37. Production Approval

A production environment can be protected by required reviewers or
other repository governance controls.

Concept:

```text
Build
 |
v
Stage
 |
v
Production Approval
 |
v
Production
```

---

# PART XVI — PERMISSIONS

## 38. Workflow Permissions

Use least privilege.

Concept:

```yaml
permissions:
  contents: read
```

Only request additional permissions when needed.

---

## 39. Why?

If a workflow is compromised, excessive permissions increase blast
radius.

---

# PART XVII — OIDC

## 40. OIDC Concept

GitHub Actions can use OpenID Connect to obtain short-lived cloud or
external identity credentials when the target platform supports it.

Concept:

```text
GitHub Actions
 |
v
OIDC Identity Token
 |
v
Cloud/External Identity Provider
 |
v
Temporary Credentials
```

---

## 41. Benefits

```text
less long-lived secret storage
short-lived credentials
federated identity
```

---

## 42. OIDC Permissions

Workflows commonly need:

```yaml
permissions:
  id-token: write
  contents: read
```

Only grant permissions required by the workflow.

---

# PART XVIII — ARTIFACTORY AUTHENTICATION

## 43. Maven Repository

Concept:

```text
GitHub Actions
 |
v
Maven
 |
v
Artifactory
```

---

## 44. Read Access

PR builds normally need repository read access.

---

## 45. Publish Access

Release workflows need publish permissions.

Separate:

```text
reader
publisher
administrator
```

identities.

---

# PART XIX — MAVEN DEPLOY

## 46. Publish

Example:

```bash
./mvnw -B deploy
```

The POM must define appropriate distribution management and the
workflow must provide matching credentials.

---

# PART XX — BUILD ONCE

## 47. Correct Model

```text
GitHub Actions
 |
v
Maven Build
 |
v
Validated Artifact
 |
v
Artifactory
 |
+--> DEV
+--> STAGE
+--> PROD
```

---

## 48. Avoid

```text
DEV rebuild
STAGE rebuild
PROD rebuild
```

The same validated artifact should be promoted when practical.

---

# PART XXI — TESTING

## 49. Unit Tests

```bash
./mvnw -B test
```

Often included automatically by:

```bash
./mvnw -B verify
```

depending on the lifecycle configuration.

---

## 50. Integration Tests

Common lifecycle:

```text
pre-integration-test
integration-test
post-integration-test
verify
```

---

# PART XXII — TEST REPORTS

## 51. Surefire Reports

Common:

```text
target/surefire-reports/
```

---

## 52. Failsafe Reports

Common:

```text
target/failsafe-reports/
```

Configure workflow steps to publish appropriate test results.

---

# PART XXIII — QUALITY

## 53. Quality Gate

Example:

```text
Build
 |
v
Tests
 |
v
Quality
 |
+--> PASS
|
+--> FAIL
```

Do not publish production artifacts after a required quality gate
fails.

---

# PART XXIV — SECURITY SCANNING

## 54. Dependency Security

Scan:

```text
direct dependencies
transitive dependencies
```

---

## 55. Build Tooling

Where supported, also assess:

```text
Maven plugins
GitHub Actions
runner image
JDK
build container
```

---

## 56. Action Supply Chain

Third-party GitHub Actions execute code in the workflow environment.

Treat action selection and versioning as supply-chain decisions.

---

# PART XXV — ACTION PINNING

## 57. Version Pinning

Actions can be referenced by:

```text
major version
tag
commit SHA
```

For high-assurance environments, pinning to an immutable commit SHA
provides stronger protection against mutable tag changes.

Balance this with your organization's action-update process.

---

# PART XXVI — DEPENDABOT / UPDATES

## 58. Dependency Updates

Automated update tooling can help identify updates for:

```text
Maven dependencies
GitHub Actions
```

Review and test updates rather than blindly merging them.

---

# PART XXVII — MATRIX BUILDS

## 59. Matrix

A matrix can test combinations.

Example:

```yaml
strategy:
  matrix:
    java: ['17', '21']
```

---

## 60. Use Cases

```text
multiple JDKs
multiple operating systems
multiple Maven profiles
compatibility testing
```

---

## 61. Cost

Matrix builds multiply:

```text
runner minutes
repository downloads
test execution
```

Use only where compatibility coverage provides value.

---

# PART XXVIII — JOB DEPENDENCIES

## 62. needs

Concept:

```yaml
jobs:
  test:
    ...

  publish:
    needs: test
```

This prevents publish from starting until the required job succeeds.

---

# PART XXIX — PARALLEL JOBS

## 63. Example

```text
             +--> Unit Tests
Build ------>|
             +--> Security
```

Only parallelize independent activities.

---

# PART XXX — CONCURRENCY

## 64. Why Concurrency Control?

Multiple workflow runs may target the same branch/environment.

Use concurrency controls where appropriate to prevent obsolete runs from
competing with current releases or deployments.

---

## 65. Example Concept

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

Choose behavior according to release requirements.

---

# PART XXXI — ARTIFACTS

## 66. GitHub Workflow Artifacts

A workflow can upload files for later jobs or investigation.

Examples:

```text
test reports
logs
coverage
build outputs
```

Do not treat temporary workflow artifacts as a replacement for an
enterprise artifact repository for production releases.

---

# PART XXXII — MAVEN ARTIFACT

## 67. Artifact

Example:

```text
target/payment-app-1.4.0.jar
```

---

## 68. Enterprise Repository

```text
GitHub Actions
 |
v
Artifactory
 |
v
payment-app-1.4.0.jar
```

---

# PART XXXIII — MULTI-MODULE MAVEN

## 69. Reactor

```text
GitHub Actions
 |
v
./mvnw -B clean verify
 |
v
Maven Reactor
 |
+--> api
+--> core
+--> service
+--> app
```

---

## 70. Selective Build

Example:

```bash
./mvnw -pl payment-app -am verify
```

Use selective builds only when dependency impact analysis is reliable.

---

# PART XXXIV — MONOREPO

## 71. Monorepo

```text
Git Repository
 |
+--> module-a
+--> module-b
+--> module-c
```

A single workflow can coordinate the entire Maven reactor.

---

# PART XXXV — REUSABLE WORKFLOWS

## 72. Why Reusable Workflows?

Large organizations often need consistent CI standards.

Example:

```text
Service A
Service B
Service C
   |
   v
Reusable Maven CI
```

---

## 73. Benefits

```text
standardization
less duplication
centralized updates
consistent security controls
```

---

# PART XXXVI — COMPOSITE ACTIONS

## 74. Composite Action

A composite action can package repeated steps.

Example use:

```text
setup enterprise Maven
 |
configure repository
 |
run standard checks
```

---

# PART XXXVII — STANDARDIZATION

## 75. Enterprise Golden Pipeline

```text
Checkout
 |
v
JDK
 |
v
Maven Wrapper
 |
v
Cache
 |
v
Build
 |
v
Test
 |
v
Security
 |
v
Package
 |
v
Publish
```

---

# PART XXXVIII — RELEASE WORKFLOW

## 76. Release Tag

```text
v1.4.0
 |
v
Release Workflow
 |
v
clean verify
 |
v
security
 |
v
deploy
 |
v
Artifactory
```

---

# PART XXXIX — SNAPSHOT WORKFLOW

## 77. Development

```text
main/development
 |
v
CI
 |
v
verify
 |
v
snapshot publish
```

Snapshot publishing should follow organizational policy.

---

# PART XL — PRODUCTION WORKFLOW

## 78. Production

```text
Tag
 |
v
Build
 |
v
Test
 |
v
Security
 |
v
Publish
 |
v
Approval
 |
v
Promotion
```

---

# PART XLI — CONTAINER BUILD

## 79. Maven Artifact to Image

```text
Maven
 |
v
JAR
 |
v
Docker/BuildKit
 |
v
Image
 |
v
Container Registry
```

---

## 80. Build Once

The container stage should consume the exact validated application
artifact when the pipeline architecture uses a separate Maven build.

---

# PART XLII — CONTAINER REGISTRY

## 81. Different Repositories

```text
Maven Repository
 |
JAR/WAR/POM

Container Registry
 |
OCI Image
```

Do not confuse the two.

---

# PART XLIII — KUBERNETES

## 82. Delivery

```text
GitHub Actions
 |
v
Maven
 |
v
JAR
 |
v
Container Image
 |
v
Registry
 |
v
GitOps
 |
v
Kubernetes
```

---

# PART XLIV — GITOPS

## 83. GitOps Separation

A mature model can separate:

```text
Application CI
 |
build image

GitOps repository
 |
declare image version

CD controller
 |
deploy Kubernetes
```

This reduces direct deployment permissions in the CI workflow.

---

# PART XLV — ENVIRONMENT PROMOTION

## 84. Promotion

```text
Artifact
 |
v
DEV
 |
v
STAGE
 |
v
PROD
```

The artifact should remain unchanged.

---

# PART XLVI — ROLLBACK

## 85. Artifact Rollback

Use a previously validated version.

```text
current 1.5.0
      |
      X
known-good 1.4.2
      |
      v
deploy
```

---

## 86. Do Not Rebuild Rollback

Avoid rebuilding old source during rollback.

Use the existing immutable artifact/image.

---

# PART XLVII — WORKFLOW SECURITY

## 87. Untrusted Pull Requests

Be careful when workflows execute code from untrusted pull-request
branches.

Potential risks include:

```text
malicious build commands
secret exfiltration
dependency manipulation
artifact poisoning
```

---

## 88. Secret Exposure

Do not expose production credentials to untrusted PR jobs.

---

# PART XLVIII — FORK SECURITY

## 89. Forked PRs

Treat fork-originated pull requests as untrusted unless the workflow
design explicitly establishes a safe trust boundary.

Avoid giving powerful write credentials to arbitrary PR execution.

---

# PART XLIX — PULL REQUEST TARGET

## 90. Security Consideration

Workflows that execute privileged actions based on pull-request
content require careful review.

The source of the workflow and the source of the code being executed
must be understood.

---

# PART L — DEPENDENCY SUPPLY CHAIN

## 91. Maven Dependency Flow

```text
GitHub Actions
 |
v
Enterprise Repository
 |
+--> Internal
+--> Approved Remote
 |
v
Maven
```

---

## 92. Dependency Confusion

Prevent through:

```text
repository policy
approved namespaces
internal mirrors
controlled upstreams
```

---

# PART LI — ACTION SUPPLY CHAIN

## 93. Actions

Every external action is executable code.

Review:

```text
publisher
repository
version
permissions
maintenance
```

---

# PART LII — RUNNER SECURITY

## 94. Self-Hosted Runner

If compromised, a persistent runner can expose:

```text
secrets
network access
workspace
cached artifacts
```

Use isolation and lifecycle controls.

---

# PART LIII — SECRET MANAGEMENT

## 95. Good

```text
GitHub Secret
 |
v
Workflow
 |
v
Maven settings
 |
v
Artifactory
```

---

## 96. Better Where Supported

```text
GitHub OIDC
 |
v
External Identity
 |
v
Short-Lived Credential
```

This can reduce long-lived static secrets.

---

# PART LIV — LOG SECURITY

## 97. Do Not Log

```text
tokens
passwords
private keys
cloud credentials
```

---

## 98. Maven Debug Logs

Be cautious with:

```bash
./mvnw -X
```

because verbose logs can expose environment or configuration details.

---

# PART LV — BUILD PROVENANCE

## 99. Traceability

Record:

```text
Git commit
workflow run
workflow file
JDK
Maven
dependencies
artifact version
artifact checksum
container image
deployment
```

---

# PART LVI — SBOM

## 100. SBOM

Generate an SBOM when required.

Concept:

```text
Build
 |
v
SBOM
 |
+--> dependencies
+--> versions
+--> components
```

---

# PART LVII — ATTESTATIONS

## 101. Build Attestation

Where supported, workflow systems can produce provenance or
attestation information.

The goal is to answer:

```text
Where did this artifact come from?
Which workflow produced it?
Which source revision was used?
```

---

# PART LVIII — RELEASE TAG SECURITY

## 102. Protected Releases

Protect release branches/tags according to organizational governance.

Avoid allowing arbitrary users/workflows to publish production
coordinates.

---

# PART LIX — VERSIONING

## 103. Maven Version

Example:

```text
1.4.0
```

---

## 104. Git Tag

Example:

```text
v1.4.0
```

---

## 105. Jenkins/GitHub Run Number

A workflow run number is useful metadata but should not automatically
replace application release versioning.

---

# PART LX — CACHING AND REPRODUCIBILITY

## 106. Cache Principle

Caching should optimize the build, not determine whether the build is
correct.

---

## 107. Reproducible Inputs

Control:

```text
source
Maven
JDK
dependencies
plugins
repository
workflow
runner
```

---

# PART LXI — TIMEOUTS

## 108. Job Timeout

Set reasonable job timeouts for production workflows.

A stuck Maven build should not consume a runner indefinitely.

---

# PART LXII — RETRIES

## 109. Retry Transient Failures

Possible candidates:

```text
temporary repository failure
transient network error
runner provisioning issue
```

---

## 110. Do Not Retry Deterministic Failures

Examples:

```text
compile failure
unit test failure
security gate failure
invalid POM
```

---

# PART LXIII — FAILURE GATES

## 111. Pipeline

```text
Compile
 |
+--> FAIL -> STOP
 |
v
Tests
 |
+--> FAIL -> STOP
 |
v
Security
 |
+--> FAIL -> STOP
 |
v
Publish
```

---

# PART LXIV — JOB DEPENDENCIES

## 112. Example

```text
build
 |
v
test
 |
v
security
 |
v
publish
```

Use `needs` to enforce dependencies.

---

# PART LXV — PARALLEL QUALITY CHECKS

## 113. Example

```text
             +--> Unit Test
             |
Build ------>+--> SAST
             |
             +--> Dependency Scan
```

Then:

```text
all required checks
 |
v
publish
```

---

# PART LXVI — TEST MATRIX

## 114. JDK Matrix

```text
JDK 17
JDK 21
```

Useful for libraries supporting multiple JDKs.

---

# PART LXVII — MULTI-OS MATRIX

## 115. Example

```text
Ubuntu
Windows
macOS
```

Only use if the application actually requires cross-platform
validation.

---

# PART LXVIII — ARTIFACT UPLOAD

## 116. Temporary Artifact

GitHub Actions artifact storage can retain:

```text
logs
reports
debug files
```

---

## 117. Production Artifact

Use an enterprise repository for long-term release artifacts when that
is the organization's artifact-management platform.

---

# PART LXIX — REUSABLE MAVEN WORKFLOW

## 118. Concept

```text
workflow_call
 |
v
Standard Maven CI
 |
+--> JDK
+--> cache
+--> verify
+--> security
+--> reports
```

Service repositories consume the standardized workflow.

---

# PART LXX — ENTERPRISE GOVERNANCE

## 119. Standard Controls

Require:

```text
Maven Wrapper
approved JDK
approved actions
least privilege
dependency scanning
quality gates
artifact repository
```

---

# PART LXXI — OBSERVABILITY

## 120. Metrics

Track:

```text
workflow duration
queue time
runner startup
Maven duration
test duration
security duration
publish duration
failure rate
```

---

# PART LXXII — SLOW BUILD

## 121. Diagnose

Measure:

```text
checkout
JDK setup
cache restore
dependency resolution
compile
tests
security
publish
```

Optimize the largest bottleneck first.

---

# PART LXXIII — REPOSITORY FAILURE

## 122. Dependency Download Failure

Check:

```text
Artifactory
mirror
credentials
DNS
TLS
network
cache
```

---

# PART LXXIV — AUTHENTICATION ERRORS

## 123. 401

Usually investigate:

```text
credential
token
server ID
secret injection
```

---

## 124. 403

Usually investigate:

```text
authorization
repository permissions
token scope
```

---

## 125. 404

Investigate:

```text
coordinates
repository
routing
mirror
artifact existence
```

---

# PART LXXV — MAVEN BUILD FAILURE

## 126. Compilation

Check:

```text
JDK
Maven
compiler plugin
dependencies
source
```

---

## 127. Test

Check:

```text
reports
environment
test data
flaky behavior
```

---

# PART LXXVI — RUNNER FAILURE

## 128. Runner

Check:

```text
runner status
capacity
network
disk
memory
container runtime
```

---

# PART LXXVII — CACHE FAILURE

## 129. Cache

Compare:

```text
warm cache
cold cache
```

Target corrupted entries rather than blindly deleting everything.

---

# PART LXXVIII — CONTAINER FAILURE

## 130. Docker Build

Check:

```text
Dockerfile
build context
artifact path
registry credentials
base image
```

---

# PART LXXIX — KUBERNETES FAILURE

## 131. Deployment

Separate:

```text
CI build
image push
GitOps change
deployment
runtime
```

Do not treat every Kubernetes failure as a Maven failure.

---

# PART LXXX — PRODUCTION ARCHITECTURE

## 132. Reference

```text
                              GitHub
                                 |
                     +-----------+-----------+
                     |                       |
                     v                       v
                    PR                     Tag
                     |                       |
                     +-----------+-----------+
                                 |
                                 v
                         GitHub Actions
                                 |
                         Ephemeral Runner
                                 |
                   +-------------+-------------+
                   |                           |
                   v                           v
              JDK / Maven                Credentials
                   |
                   v
              Maven Wrapper
                   |
                   v
          Enterprise Repository
                   |
          +--------+--------+
          |                 |
          v                 v
     Dependencies        Plugins
          |
          v
        Compile
          |
          v
         Test
          |
          v
       Security
          |
          v
        Package
          |
          v
      Maven Artifact
          |
          v
       Artifactory
          |
          v
     Container Build
          |
          v
   Container Registry
          |
          v
       GitOps Repo
          |
          v
      CD Controller
          |
          v
      Kubernetes
```

---

# PART LXXXI — ENTERPRISE SECURITY ARCHITECTURE

## 133. Identity

```text
GitHub Actions
 |
+--> repository read
+--> artifact read
 |
Release workflow
 |
+--> artifact publish
```

Production deployment credentials should be isolated from ordinary PR
workflows.

---

# PART LXXXII — PRODUCTION CHECKLIST

## 134. GitHub

```text
[ ] branch protection
[ ] protected releases
[ ] workflow review
[ ] approved actions
[ ] least-privilege permissions
```

## 135. Maven

```text
[ ] Maven Wrapper
[ ] controlled JDK
[ ] dependency versions
[ ] plugin versions
[ ] reproducible build
```

## 136. Repository

```text
[ ] Artifactory/Nexus
[ ] virtual repository
[ ] approved upstreams
[ ] read/write separation
[ ] immutable releases
```

## 137. Security

```text
[ ] dependency scan
[ ] action review
[ ] secret protection
[ ] OIDC where appropriate
[ ] runner isolation
[ ] SBOM
```

## 138. Release

```text
[ ] build once
[ ] artifact provenance
[ ] approval
[ ] promotion
[ ] rollback
```

---

# PART LXXXIII — INTERVIEW PREPARATION

## 139. How Does GitHub Actions Integrate with Maven?

Answer:

```text
GitHub Actions orchestrates the CI/CD workflow and invokes Maven for
the project lifecycle. I normally use the Maven Wrapper, a controlled
JDK, repository authentication, dependency caching, tests, security
checks and artifact publishing as separate pipeline concerns.
```

## 140. Why Use Maven Wrapper?

Answer:

```text
It makes the Maven version part of the project and reduces differences
between developer and GitHub Actions environments.
```

## 141. How Do You Cache Maven Dependencies?

Answer:

```text
I use the supported Maven cache mechanism, typically through the Java
setup action, and ensure cache inputs are appropriate. I also validate
cold-cache builds so the cache is an optimization rather than a hidden
requirement.
```

## 142. How Do You Publish Maven Artifacts?

Answer:

```text
After required build, test, quality and security gates pass, the
release workflow executes Maven deploy with least-privilege
credentials to the configured artifact repository.
```

## 143. How Do You Secure GitHub Actions?

Answer:

```text
I use least-privilege workflow permissions, protected environments,
approved actions, secure secrets, isolated runners and OIDC or
short-lived credentials where supported.
```

## 144. Why Build Once?

Answer:

```text
One validated artifact can be promoted through environments without
introducing differences caused by rebuilding dependencies, tools or
source. This improves reproducibility and rollback.
```

## 145. What Is OIDC?

Answer:

```text
OIDC allows GitHub Actions to establish federated identity with a
supported external platform and obtain short-lived credentials instead
of storing long-lived static credentials.
```

## 146. How Do You Secure PR Workflows?

Answer:

```text
I treat untrusted PR code as potentially malicious, avoid exposing
production credentials, restrict permissions, review privileged
workflow triggers and separate validation from release operations.
```

---

# PART LXXXIV — SENIOR-LEVEL SCENARIOS

## 147. PR Workflow Is Compromised

Answer:

```text
I would immediately determine what permissions and secrets the
workflow had, revoke or rotate exposed credentials, inspect published
artifacts and workflow changes, and review whether the workflow
allowed untrusted code to access privileged resources. I would then
reduce permissions and strengthen the trust boundary.
```

## 148. GitHub Actions Runner Is Compromised

Answer:

```text
I would isolate the runner, revoke credentials available to it, inspect
workflow and artifact activity, determine lateral access, replace the
runner and investigate persistence. Persistent self-hosted runners
receive particular scrutiny because their state can survive jobs.
```

## 149. Maven Dependencies Suddenly Fail Across Repositories

Answer:

```text
Because the failure is widespread, I would investigate the shared
enterprise repository, mirror, DNS, TLS, network and authentication
path before changing individual Maven POMs.
```

## 150. Release Workflow Publishes Before Security Scan

Answer:

```text
I would treat this as a pipeline control failure. I would make the
security job an explicit prerequisite of publishing, enforce the
dependency with needs, review branch/tag protections and verify that
no alternate workflow can bypass the release gate.
```

## 151. Same Commit Produces Different Artifacts

Answer:

```text
I would compare Maven/JDK versions, dependency resolution, plugin
versions, repository contents, workflow revision, runner image,
environment variables and timestamps or generated content. The goal is
to identify uncontrolled build inputs and make them reproducible.
```

## 152. GitHub Actions Cache Is Corrupted

Answer:

```text
I would test with a cold cache, identify whether the issue is isolated
to one cache key, invalidate or replace the affected cache and verify
that the build succeeds from the authoritative repository.
```

## 153. Production Deployment Must Be Rolled Back

Answer:

```text
I would identify the last known-good immutable image/artifact and
promote or deploy that exact version. I would avoid rebuilding source
during rollback.
```

## 154. Organization Has 500 Maven Repositories

Answer:

```text
I would standardize a reusable Maven workflow, approved JDK versions,
Maven Wrapper usage, repository access, security gates, action
policies and artifact publishing. The reusable workflow should provide
the common controls while allowing controlled application-specific
configuration.
```

## 155. Teams Use Long-Lived Artifactory Passwords

Answer:

```text
I would migrate toward scoped service identities, short-lived tokens
or OIDC/federated authentication where the repository and environment
support it. I would rotate existing credentials and audit their usage.
```

---

# PART LXXXV — GOLDEN RULES

## 156. Rules

```text
1. GitHub Actions orchestrates; Maven builds.

2. Use Maven Wrapper when the project maintains it.

3. Control JDK and Maven versions.

4. Keep workflow files under .github/workflows.

5. Treat workflow code as production automation code.

6. Review workflow changes.

7. Use least-privilege workflow permissions.

8. Default to minimal contents permissions.

9. Grant id-token permission only when OIDC is required.

10. Protect production environments.

11. Separate PR validation from release publishing.

12. Never expose production credentials to untrusted PR code.

13. Treat forked PRs as untrusted by default.

14. Be careful with privileged pull_request_target-style designs.

15. Review every third-party action.

16. Prefer immutable action references for high-assurance environments.

17. Maintain an action update process.

18. Use trusted JDK distributions.

19. Use Maven Wrapper.

20. Run Maven in batch mode in CI.

21. Use clean verify for appropriate CI validation.

22. Publish only after required gates pass.

23. Use repository read access for normal builds.

24. Use separate publishing identities.

25. Use least-privilege Artifactory permissions.

26. Never commit repository credentials.

27. Use GitHub secrets or workload identity appropriately.

28. Prefer short-lived credentials where supported.

29. Understand OIDC trust relationships.

30. Protect self-hosted runners.

31. Prefer ephemeral runners where practical.

32. Do not treat persistent runners as trusted forever.

33. Monitor runner disk, memory and network.

34. Cache Maven dependencies for performance.

35. Treat cache as an optimization, not a source of truth.

36. Test cold-cache builds.

37. Do not blindly delete all caches during troubleshooting.

38. Publish test reports.

39. Separate unit and integration test concerns.

40. Enforce quality gates.

41. Enforce security gates.

42. Scan direct and transitive dependencies.

43. Review Maven plugins as build supply-chain components.

44. Review GitHub Actions as executable supply-chain components.

45. Generate SBOMs where required.

46. Preserve build provenance.

47. Associate commits with workflow runs.

48. Associate workflow runs with artifacts.

49. Associate artifacts with container images.

50. Associate images with deployments.

51. Use build-once/promote strategy.

52. Do not rebuild separately for each environment.

53. Keep release artifacts immutable.

54. Keep snapshots separate from releases.

55. Use protected release tags.

56. Use reusable workflows for common enterprise controls.

57. Use composite actions for repeatable local step groups where
    appropriate.

58. Use matrix builds only when compatibility coverage justifies cost.

59. Use job dependencies to enforce release gates.

60. Parallelize only independent work.

61. Use concurrency controls for overlapping releases/deployments.

62. Use timeouts for stuck jobs.

63. Retry only plausible transient failures.

64. Do not retry deterministic compilation/test/security failures
    indefinitely.

65. Use GitHub workflow artifacts for reports and diagnostics, not as a
    replacement for enterprise artifact repositories.

66. Separate Maven repositories from container registries.

67. Consume the validated Maven artifact when building the container.

68. Separate CI from GitOps deployment when architecture benefits from
    that boundary.

69. Keep production deployment permissions out of ordinary PR jobs.

70. Design rollback around immutable known-good artifacts.

71. Monitor workflow duration and failure rate.

72. Measure before optimizing Maven builds.

73. Investigate shared infrastructure when many repositories fail at
    once.

74. Distinguish repository failures from Maven configuration failures.

75. Distinguish CI failures from Kubernetes runtime failures.

76. Use effective Maven settings/POM diagnostics when repository
    behavior is unclear.

77. Use dependency:tree for dependency problems.

78. Protect logs from secrets.

79. Be cautious with Maven debug output.

80. Maintain branch and environment protections.

81. Standardize enterprise build workflows without preventing required
    application-specific behavior.

82. Validate reusable workflows across representative repositories.

83. Version build tooling deliberately.

84. Maintain documented release procedures.

85. Test disaster recovery and rollback procedures.

86. Review external action permissions.

87. Review workflow-trigger trust boundaries.

88. Do not assume a successful workflow means a secure workflow.

89. Do not assume a cached build proves repository health.

90. Validate exact GitHub Actions, runner, Maven, JDK, plugin,
    repository and deployment behavior for the production architecture
    actually used.
```

---

# END OF 10-Maven-GitHub-Actions.md
