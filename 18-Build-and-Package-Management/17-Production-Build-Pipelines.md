# Production-Build-Pipelines

## 1. Purpose

A production build pipeline is an automated, controlled path that turns
source code into a tested, secure, traceable, deployable artifact.

Reference lifecycle:

```text
Developer
   |
   v
Git / Pull Request
   |
   v
Code Review
   |
   v
CI Validation
   |
   +--> Secret Scan
   +--> SAST
   +--> Dependency Scan
   |
   v
Build
   |
   +--> Unit Tests
   +--> Integration Tests
   |
   v
Package
   |
   +--> SBOM
   +--> Security Scan
   +--> Provenance
   |
   v
Artifact Repository
   |
   v
Promotion
   |
   +--> DEV
   +--> STAGE
   +--> PROD
```

The production principle is:

```text
Automate the repeatable work.
Gate the risky work.
Record the evidence.
Build once.
Promote the same artifact.
```

---

# PART I — PRODUCTION PIPELINE FUNDAMENTALS

## 2. What Is a Production Build Pipeline?

A production build pipeline is a controlled CI/CD workflow designed to
reliably produce software for production consumption.

It should provide:

```text
repeatability
security
traceability
quality
speed
recovery
auditability
```

---

## 3. Build Pipeline vs Deployment Pipeline

Build pipeline:

```text
source -> build -> test -> package -> publish
```

Deployment pipeline:

```text
artifact -> environment -> validate -> promote -> deploy
```

They can exist in one CI/CD platform but should remain conceptually
separate.

---

## 4. Production Pipeline Goals

A mature pipeline should:

```text
fail fast
produce deterministic outputs
protect credentials
validate dependencies
publish immutable artifacts
support rollback
provide audit evidence
```

---

# PART II — PIPELINE DESIGN

## 5. Pipeline Stages

Typical stages:

```text
Checkout
Prepare
Lint
Secret Scan
Build
Unit Test
Integration Test
SAST
SCA
Package
SBOM
Artifact Scan
Publish
Promote
Deploy
Verify
```

Not every project requires every stage.

---

## 6. Stage Ordering

Fast checks should normally run early.

Example:

```text
Checkout
 |
v
Lint + Secret Scan
 |
v
Compile
 |
v
Unit Test
 |
v
SCA/SAST
 |
v
Integration Test
 |
v
Package
 |
v
Publish
```

Expensive tests can run after inexpensive validation.

---

## 7. Fail Fast

If source does not compile:

```text
stop
```

Do not spend resources running long deployment tests.

---

# PART III — SOURCE CONTROL

## 8. Git as Source of Truth

Production builds should begin from a known Git revision.

Record:

```text
commit SHA
tag
branch
repository
```

---

## 9. Release Build

A release should preferably build from:

```text
protected tag
```

or another controlled immutable source reference.

---

## 10. Clean Checkout

Build from a clean workspace.

Avoid accidental dependence on:

```text
developer files
previous build output
unknown local configuration
stale cache
```

---

# PART IV — BRANCH STRATEGY

## 11. Main Branch

Typical model:

```text
feature
   |
   v
PR
   |
   v
main
   |
   v
release
```

Protect the main branch.

---

## 12. Release Branch

Use release branches when the organization needs:

```text
stabilization
parallel maintenance
multiple supported releases
```

Do not create release branches without a real operational need.

---

# PART V — PULL REQUEST PIPELINE

## 13. PR Pipeline

A PR pipeline commonly runs:

```text
checkout
lint
secret scan
build
unit test
SAST
dependency scan
```

It should normally not publish production artifacts.

---

## 14. PR Security

Treat PR code as potentially untrusted.

Do not expose:

```text
production credentials
signing keys
deployment tokens
repository admin credentials
```

to arbitrary PR execution.

---

# PART VI — RELEASE TRIGGER

## 15. Release Triggers

Possible triggers:

```text
Git tag
manual approval
release event
merge to protected branch
scheduled release
```

Use the organization's controlled release model.

---

## 16. Tag Trigger

Example concept:

```text
v1.5.0
 |
v
release workflow
 |
v
build
 |
v
publish
```

---

# PART VII — VERSIONING

## 17. Version Source

Choose one authoritative version source.

Possible models:

```text
Git tag
pom.xml
package.json
pyproject.toml
release configuration
```

Avoid conflicting manually maintained versions.

---

## 18. Version Validation

Before publishing:

```text
version exists?
version unique?
tag matches version?
artifact metadata matches?
```

---

# PART VIII — BUILD ENVIRONMENT

## 19. Standard Build Environment

Control:

```text
OS
JDK
Maven
Node
npm
Python
pip
Docker/BuildKit
compiler
```

---

## 20. Toolchain Versioning

Example:

```text
Java 21
Maven 3.x
Node 22
Python 3.x
```

Exact versions should be defined by project requirements.

---

## 21. Toolchain Drift

Uncontrolled tool upgrades can produce different artifacts.

Use:

```text
pinned tool versions
versioned build images
controlled runners
```

---

# PART IX — DEPENDENCY MANAGEMENT

## 22. Dependency Restore

Use controlled repositories.

Example:

```text
CI
 |
v
Artifactory virtual repository
 |
+--> internal packages
+--> approved cached dependencies
```

---

## 23. Dependency Locking

Use appropriate locking:

```text
package-lock.json
poetry.lock
constraints
Maven dependency management
```

---

## 24. Dependency Cache

Caching improves performance but should not change correctness.

```text
cache miss -> build still works
cache hit  -> build is faster
```

---

# PART X — BUILD CACHING

## 25. Cache Layers

Potential caches:

```text
Maven ~/.m2
npm cache
pip cache
Docker layers
Gradle cache
CI workspace cache
```

---

## 26. Cache Security

Never cache:

```text
secrets
private keys
production tokens
sensitive environment files
```

---

# PART XI — BUILD

## 27. Maven

Typical production validation:

```bash
./mvnw -B clean verify
```

Publishing can occur only after required gates pass.

---

## 28. npm

Typical:

```bash
npm ci
npm test
npm run build
```

Use `npm ci` when a lockfile-based reproducible install is intended.

---

## 29. Python

Typical:

```bash
python -m pip install -r requirements.txt
python -m pytest
python -m build
```

Use the project's controlled dependency mechanism.

---

# PART XII — TESTING

## 30. Unit Tests

Unit tests validate individual components.

Run early because they are generally fast.

---

## 31. Integration Tests

Validate interactions:

```text
application
 |
+--> database
+--> queue
+--> service
```

---

## 32. Contract Tests

Useful for service/API compatibility.

Example:

```text
consumer contract
 |
v
provider validation
```

---

## 33. End-to-End Tests

Validate complete business flows.

They are often more expensive and slower.

---

# PART XIII — TEST STRATEGY

## 34. Test Pyramid

Concept:

```text
       E2E
      /   \
 Integration
   /       \
   Unit Tests
```

More fast tests, fewer expensive tests.

---

## 35. Flaky Tests

Flaky tests reduce trust.

Track:

```text
test
failure
rerun result
history
```

Do not permanently hide flaky tests with unlimited retries.

---

# PART XIV — QUALITY GATES

## 36. Quality Gate

Examples:

```text
tests pass
coverage threshold
no critical security findings
lint pass
quality threshold
```

---

## 37. Coverage

Coverage is useful but is not proof of correctness.

A high coverage percentage can still contain poor tests.

---

# PART XV — STATIC ANALYSIS

## 38. SAST

Use static analysis for source-level security issues.

Pipeline:

```text
source
 |
v
SAST
 |
v
findings
 |
v
policy
```

---

# PART XVI — DEPENDENCY SCANNING

## 39. SCA

Scan:

```text
direct dependencies
transitive dependencies
runtime dependencies
build dependencies
```

---

## 40. Vulnerability Policy

Example:

```text
Critical -> block
High -> block/review
Medium -> review
Low -> track
```

Actual thresholds depend on organizational risk.

---

# PART XVII — SECRET SCANNING

## 41. Secret Gate

Run secret scanning before artifact publication.

Detect:

```text
API tokens
cloud keys
private keys
passwords
registry credentials
```

---

# PART XVIII — CONTAINER BUILD

## 42. Container Pipeline

```text
Build source
 |
v
Dockerfile
 |
v
Image
 |
v
Image scan
 |
v
SBOM
 |
v
Sign/attest
 |
v
Registry
```

---

## 43. Docker Build Context

Keep context small.

Use:

```text
.dockerignore
```

to exclude:

```text
.git
node_modules
logs
credentials
local files
```

---

# PART XIX — IMAGE TAGGING

## 44. Tag

Use a release version:

```text
payment-api:1.5.0
```

---

## 45. Digest

Record:

```text
sha256:...
```

Production should use immutable image identity.

---

# PART XX — ARTIFACT PACKAGING

## 46. Package

Examples:

```text
JAR
WAR
TGZ
WHL
SDIST
Docker image
Helm chart
ZIP
```

---

## 47. Artifact Inspection

Before publication verify:

```text
name
version
contents
dependencies
metadata
size
```

---

# PART XXI — ARTIFACT REPOSITORY

## 48. Publish

```text
CI
 |
v
Artifact
 |
v
Artifactory
```

---

## 49. Repository Permissions

Release CI should have only the required write permissions.

Developers and PR jobs should generally not have broad release write
access.

---

# PART XXII — IMMUTABILITY

## 50. Immutable Release

Example:

```text
payment-api-1.5.0.jar
```

Once consumed, do not replace its contents.

---

## 51. Duplicate Version

If publication returns:

```text
version already exists
```

stop.

Do not force overwrite.

---

# PART XXIII — BUILD ONCE PROMOTE MANY

## 52. Model

```text
Build
 |
v
Artifact A
 |
+--> DEV
+--> STAGE
+--> PROD
```

---

## 53. Why?

The artifact tested in stage should be the artifact deployed in
production.

---

# PART XXIV — ENVIRONMENT CONFIGURATION

## 54. Configuration

Do not rebuild because environment values differ.

Use:

```text
environment variables
configuration service
Kubernetes ConfigMap
Kubernetes Secret
deployment values
```

according to architecture.

---

# PART XXV — PIPELINE SECURITY

## 55. Least Privilege

Pipeline permissions should be minimal.

Example:

```text
PR -> read
build -> read dependencies
release -> publish
deploy -> deployment permission
```

---

## 56. Protected Environment

Production deployment should use a protected environment where
appropriate.

---

# PART XXVI — CREDENTIAL MANAGEMENT

## 57. Secret Store

Use:

```text
CI secret manager
Vault
cloud secret manager
OIDC
workload identity
```

---

## 58. OIDC

Preferred model where supported:

```text
CI
 |
v
OIDC token
 |
v
cloud trust
 |
v
temporary credentials
```

---

# PART XXVII — RUNNERS

## 59. Ephemeral Runner

```text
job
 |
v
new runner
 |
v
build
 |
v
destroy
```

This reduces persistence and cross-job contamination.

---

## 60. Persistent Runner

If persistent runners are required:

```text
harden
patch
clean workspace
restrict access
monitor
```

---

# PART XXVIII — NETWORK

## 61. Build Network

Allow only required access:

```text
Git
Artifactory
security services
required APIs
```

---

## 62. Egress

Restrict unnecessary external traffic in sensitive builds.

---

# PART XXIX — PARALLELISM

## 63. Parallel Jobs

Independent work can run concurrently:

```text
             +--> unit tests
Build -------+--> SAST
             +--> SCA
             +--> lint
```

This reduces pipeline duration.

---

# PART XXX — DEPENDENCY BETWEEN JOBS

## 64. Dependency Graph

Example:

```text
Build
 |
+--> Unit
+--> SAST
+--> SCA
 |
 v
Package
 |
 v
Publish
```

Do not publish until required gates pass.

---

# PART XXXI — ARTIFACT PASSING

## 65. CI Artifact Transfer

When passing build outputs between jobs:

```text
Build Job
 |
v
CI Artifact
 |
v
Test/Package Job
```

Use trusted artifact mechanisms and verify integrity.

---

# PART XXXII — WORKSPACE SECURITY

## 66. Clean Workspace

Avoid:

```text
previous output
unknown files
developer configuration
secret files
```

---

# PART XXXIII — RELEASE PIPELINE

## 67. Reference

```text
Tag
 |
v
Checkout
 |
v
Validate Version
 |
v
Build
 |
v
Unit Test
 |
v
Integration Test
 |
v
Security
 |
v
Package
 |
v
SBOM/Provenance
 |
v
Publish
 |
v
Promote
```

---

# PART XXXIV — GITHUB ACTIONS

## 68. Production Structure

```yaml
name: Production Build

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: ./mvnw -B clean verify

      - name: Package
        run: ./mvnw -B package
```

In production, add the required security, publishing, environment
protection and credential controls.

---

# PART XXXV — JENKINS

## 69. Jenkinsfile

Typical:

```text
pipeline {
  stages {
    Checkout
    Build
    Test
    Security
    Package
    Publish
  }
}
```

Credentials belong in Jenkins credential management.

---

# PART XXXVI — GITLAB CI

## 70. Stages

Typical:

```yaml
stages:
  - validate
  - build
  - test
  - security
  - package
  - publish
```

---

# PART XXXVII — ARTIFACTORY INTEGRATION

## 71. Flow

```text
CI
 |
v
Build Info
 |
v
Artifact
 |
v
Artifactory
 |
v
Promotion
```

---

## 72. Build Info

Record:

```text
build number
source revision
dependencies
artifacts
```

---

# PART XXXVIII — BUILD PROVENANCE

## 73. Provenance

A production artifact should be traceable:

```text
artifact
 |
v
build
 |
v
commit
```

---

# PART XXXIX — SBOM

## 74. SBOM

Generate an SBOM for releases where required.

It supports rapid vulnerability impact analysis.

---

# PART XL — SIGNING

## 75. Signing

```text
Artifact
 |
v
Sign
 |
v
Repository
 |
v
Verify
```

Protect signing keys separately from ordinary CI credentials.

---

# PART XLI — POLICY AS CODE

## 76. Example

Policies can enforce:

```text
no critical CVEs
approved registry only
signed artifact required
version immutable
production requires approval
```

---

# PART XLII — RELEASE APPROVAL

## 77. Approval

Production approval can be:

```text
manual
automated policy
change-management
```

depending on risk and organization.

---

# PART XLIII — PROGRESSIVE DEPLOYMENT

## 78. Canary

```text
1%
 |
v
10%
 |
v
50%
 |
v
100%
```

Monitor health at each stage.

---

## 79. Blue/Green

```text
Blue = current
Green = new

traffic
  |
  +--> Blue

validation

traffic
  |
  +--> Green
```

---

# PART XLIV — KUBERNETES

## 80. Kubernetes Release

```text
Artifact
 |
v
Container Registry
 |
v
GitOps Repository
 |
v
Argo CD
 |
v
Kubernetes
```

---

## 81. Deployment Identity

Prefer:

```yaml
image: registry.company.com/payment-api@sha256:...
```

for immutable production identity.

---

# PART XLV — HEALTH VALIDATION

## 82. Post-Deploy

Validate:

```text
pod readiness
pod restarts
HTTP errors
latency
CPU
memory
business metrics
```

---

# PART XLVI — ROLLBACK

## 83. Rollback

```text
v1.5.0
 |
failure
 |
v1.4.3
```

Deploy the previous known-good artifact.

---

## 84. Rollback Conditions

Define thresholds before deployment:

```text
error rate
latency
availability
business failure
```

---

# PART XLVII — DATABASE

## 85. Database Migration

A pipeline must treat schema changes carefully.

Prefer:

```text
expand
 |
v
compatible application
 |
v
migrate
 |
v
contract
```

---

# PART XLVIII — RELEASE EVIDENCE

## 86. Evidence

Store:

```text
commit
build ID
version
artifact digest
test results
security results
SBOM
approval
deployment
```

---

# PART XLIX — OBSERVABILITY

## 87. Pipeline Metrics

Track:

```text
duration
queue time
failure rate
success rate
deployment frequency
rollback rate
```

---

# PART L — PIPELINE PERFORMANCE

## 88. Bottleneck Analysis

Measure each stage:

```text
checkout
dependency restore
compile
tests
security
package
upload
```

Optimize the slowest significant stage first.

---

# PART LI — CACHING

## 89. Safe Caching

Use caching for:

```text
dependencies
Docker layers
tool downloads
```

Do not allow cache behavior to bypass security validation.

---

# PART LII — TEST PARALLELIZATION

## 90. Parallel Tests

Split independent suites:

```text
unit-1
unit-2
integration-1
integration-2
```

Aggregate results before publication.

---

# PART LIII — RETRIES

## 91. Retry

Retry transient infrastructure failures.

Do not blindly retry deterministic test failures.

---

# PART LIV — TIMEOUTS

## 92. Timeouts

Every external operation should have sensible timeouts.

Prevent:

```text
hung job
resource exhaustion
indefinite pipeline
```

---

# PART LV — CONCURRENCY

## 93. Release Concurrency

Prevent two pipelines from publishing the same release version.

Use:

```text
release locks
concurrency groups
unique versions
```

---

# PART LVI — CANCEL OBSOLETE BUILDS

## 94. Superseded Builds

For branch CI, cancel obsolete builds when safe.

Do not cancel an already approved production release without explicit
policy.

---

# PART LVII — ARTIFACT RETENTION

## 95. Retention

Retain production artifacts according to:

```text
rollback
support
compliance
DR
```

---

# PART LVIII — LOG RETENTION

## 96. Logs

Retain sufficient logs for:

```text
audit
troubleshooting
incident response
```

Protect logs from unauthorized access.

---

# PART LIX — AUDIT

## 97. Audit Trail

Answer:

```text
who
what
when
where
which artifact
which source
which approval
```

---

# PART LX — FAILURE HANDLING

## 98. Build Failure

```text
stop
 |
v
capture logs
 |
v
identify cause
 |
v
fix
 |
v
rerun
```

---

# PART LXI — SECURITY FAILURE

## 99. Security Gate Failure

Do not bypass automatically.

Determine:

```text
finding
severity
affected component
policy
exception
```

---

# PART LXII — PUBLISH FAILURE

## 100. Publish Failure

Check:

```text
authentication
authorization
repository
network
version
checksum
storage
```

---

# PART LXIII — DEPLOYMENT FAILURE

## 101. Deployment Failure

Separate:

```text
build problem
artifact problem
deployment problem
application problem
infrastructure problem
```

Verify artifact identity first.

---

# PART LXIV — INCIDENT RESPONSE

## 102. Pipeline Compromise

If CI is compromised:

```text
stop releases
 |
v
revoke credentials
 |
v
audit workflow
 |
v
inspect artifacts
 |
v
rebuild trusted
 |
v
restore secure pipeline
```

---

# PART LXV — PRODUCTION REFERENCE ARCHITECTURE

## 103. End-to-End

```text
                         Developer
                             |
                             v
                         Git / PR
                             |
                    +--------+--------+
                    |                 |
                    v                 v
               Code Review       Security Scan
                    |                 |
                    +--------+--------+
                             |
                             v
                            CI
                             |
        +--------------------+--------------------+
        |                    |                    |
        v                    v                    v
      Build               SAST/SCA          Secret Scan
        |                    |                    |
        +--------------------+--------------------+
                             |
                             v
                          Test
                             |
                             v
                         Package
                             |
                  +----------+----------+
                  |          |          |
                  v          v          v
                SBOM     Provenance   Sign
                  |          |          |
                  +----------+----------+
                             |
                             v
                         Artifactory
                             |
                             v
                         Promotion
                             |
             +---------------+---------------+
             |               |               |
             v               v               v
            DEV            STAGE            PROD
                                             |
                                             v
                                        Monitoring
                                             |
                                      +------+------+
                                      |             |
                                      v             v
                                   Healthy       Rollback
```

---

# PART LXVI — PRODUCTION CHECKLIST

## 104. Source

```text
[ ] protected branch
[ ] reviewed code
[ ] exact commit
[ ] protected release tag
```

## 105. Build

```text
[ ] clean environment
[ ] controlled tool versions
[ ] controlled dependencies
[ ] reproducible configuration
```

## 106. Security

```text
[ ] secret scan
[ ] SAST
[ ] SCA
[ ] container scan
[ ] vulnerability policy
[ ] exception process
```

## 107. Artifact

```text
[ ] correct version
[ ] correct contents
[ ] immutable
[ ] checksum/digest
[ ] SBOM
[ ] provenance
[ ] signing where required
```

## 108. Repository

```text
[ ] correct repository
[ ] correct permissions
[ ] audit enabled
[ ] retention configured
```

## 109. Deployment

```text
[ ] same artifact promoted
[ ] approval
[ ] health checks
[ ] monitoring
[ ] rollback ready
```

---

# PART LXVII — INTERVIEW PREPARATION

## 110. Explain Your Production Build Pipeline

Answer:

```text
I start from a protected source revision. The CI pipeline validates the
code with linting, secret scanning, unit tests, SAST and dependency
checks. It then builds and packages the application, generates required
SBOM/provenance information, scans and publishes an immutable artifact
to Artifactory. The same artifact is promoted through environments
without rebuilding, and production deployment is protected by
approval, health validation and rollback procedures.
```

## 111. How Do You Make Builds Reproducible?

Answer:

```text
I control toolchain versions, dependencies, build configuration and
source revision. I use lockfiles or equivalent dependency controls,
clean build environments and controlled repositories. Caches accelerate
the process but are not required for correctness.
```

## 112. How Do You Secure a Production Pipeline?

Answer:

```text
I use protected branches and tags, least-privilege CI identities,
restricted PR permissions, secure secret management, isolated runners,
dependency and source scanning, immutable artifacts, provenance,
SBOMs, signing where required and protected production environments.
```

## 113. How Do You Reduce Pipeline Duration?

Answer:

```text
I measure stage duration first, then optimize the dominant bottlenecks
using dependency caching, Docker layer caching, parallel independent
jobs, test parallelization, appropriate runner sizing and incremental
strategies while preserving mandatory quality and security gates.
```

## 114. What Does Build Once Promote Many Mean?

Answer:

```text
The artifact is built and validated once, stored immutably, and the
same artifact is promoted from development to staging and production.
We do not rebuild separately for each environment.
```

## 115. How Do You Handle a Failed Production Release?

Answer:

```text
I first assess impact and verify the exact deployed artifact. If the
previous artifact is compatible, I roll back to the known-good immutable
version. If rollback is unsafe because of schema or compatibility
constraints, I use a controlled forward fix.
```

---

# PART LXVIII — SENIOR SCENARIOS

## 116. Pipeline Takes 45 Minutes

Answer:

```text
I collect timing for queue, checkout, dependency restore, build,
testing, security scans, packaging and publishing. I identify the
largest bottleneck, then use safe caching, parallelism or test
optimization. I avoid simply removing security or test gates.
```

## 117. Two Release Pipelines Start Together

Answer:

```text
I prevent version collisions with release concurrency controls and
unique version allocation. Only one pipeline can claim a release
version. The other must fail safely and use a new version.
```

## 118. Developer Adds Production Secret to Build

Answer:

```text
I stop the release, rotate the secret, inspect repository and CI logs,
remove the secret from source history as required, verify whether the
secret reached artifacts or caches, and correct the secret-management
design.
```

## 119. Security Scan Is Blocking Delivery

Answer:

```text
I inspect the actual finding and policy. If valid, I remediate or use a
formally approved temporary exception with owner, mitigation and expiry.
I do not create blanket permanent bypasses.
```

## 120. Stage and Production Use Different Images

Answer:

```text
I compare immutable image digests and promotion records. If the digests
differ, I stop the rollout and fix artifact promotion. Production must
consume the validated artifact.
```

## 121. Build Runner Is Compromised

Answer:

```text
I isolate or destroy the runner, revoke credentials available to it,
audit source and artifacts, investigate potential exfiltration and
rebuild using trusted infrastructure.
```

## 122. Artifactory Is Unavailable

Answer:

```text
I use the organization's HA/DR or approved repository recovery process.
I do not bypass the artifact governance model by manually copying
unverified artifacts into production.
```

## 123. Critical Vulnerability Found After Release

Answer:

```text
I use SBOM and artifact lineage to identify affected releases and
deployments, assess exposure, patch the dependency, run required
security and functional validation, publish a new immutable version
and deploy it through the controlled pipeline.
```

## 124. Database Migration Breaks Rollback

Answer:

```text
I separate application rollback from database rollback. For future
releases I use backward-compatible expand/contract migrations and
validate migration recovery independently.
```

## 125. Pipeline Uses latest Docker Tag

Answer:

```text
I replace latest as the production identity with a versioned release
and immutable digest. The digest is recorded in deployment metadata
and preferably in the GitOps configuration.
```

## 126. Build Works Only When Cache Exists

Answer:

```text
That indicates an unhealthy dependency on cache state. I run a clean
cold-cache build, identify missing explicit inputs and fix the build so
cache is only an optimization.
```

---

# PART LXIX — GOLDEN RULES

## 127. Rules 1

```text
1. Start production builds from trusted source.
2. Record the exact commit.
3. Protect release tags.
4. Use clean build environments.
5. Control toolchain versions.
6. Control dependencies.
7. Use lockfiles where appropriate.
8. Use trusted repositories.
9. Treat PR code as untrusted.
10. Restrict PR permissions.
11. Never expose production secrets to untrusted PRs.
12. Protect CI workflow files.
13. Review workflow changes.
14. Review third-party actions.
15. Minimize workflow permissions.
16. Use dedicated CI identities.
17. Prefer short-lived credentials.
18. Prefer OIDC where supported.
19. Never commit secrets.
20. Rotate exposed credentials immediately.
```

## 128. Rules 2

```text
21. Scan source for secrets.
22. Use SAST.
23. Use SCA.
24. Scan transitive dependencies.
25. Scan container images.
26. Scan base images.
27. Define vulnerability policy.
28. Do not blindly suppress findings.
29. Use temporary exceptions.
30. Give exceptions owners.
31. Give exceptions expiry dates.
32. Generate SBOMs where required.
33. Preserve provenance.
34. Sign artifacts where required.
35. Protect signing keys.
36. Use KMS/HSM where appropriate.
37. Record artifact checksums.
38. Record image digests.
39. Keep production artifacts immutable.
40. Never overwrite consumed release versions.
```

## 129. Rules 3

```text
41. Build once.
42. Promote the same artifact.
43. Do not rebuild per environment.
44. Keep environment configuration separate.
45. Store artifacts durably.
46. Use Artifactory or approved repository management.
47. Restrict repository write access.
48. Restrict delete access.
49. Protect release repositories.
50. Maintain retention policies.
51. Protect rollback artifacts.
52. Maintain repository backups.
53. Test repository recovery.
54. Use repository HA for critical systems.
55. Use virtual repositories carefully.
56. Control remote dependency sources.
57. Protect internal package namespaces.
58. Prevent dependency confusion.
59. Prevent typosquatting where possible.
60. Review package provenance.
```

## 130. Rules 4

```text
61. Run fast checks early.
62. Fail fast.
63. Parallelize independent jobs.
64. Measure pipeline bottlenecks.
65. Optimize the dominant bottleneck.
66. Use dependency caching safely.
67. Use Docker layer caching safely.
68. Never cache secrets.
69. Protect shared caches.
70. Test cold-cache builds.
71. Use sensible timeouts.
72. Retry only transient failures.
73. Do not hide deterministic failures with retries.
74. Cancel obsolete branch builds where safe.
75. Protect approved release builds from accidental cancellation.
76. Prevent concurrent version collisions.
77. Use release locks/concurrency groups.
78. Keep workspaces clean.
79. Prefer ephemeral runners.
80. Harden persistent runners.
```

## 131. Rules 5

```text
81. Restrict build network access.
82. Control egress where required.
83. Do not give runners production admin access.
84. Separate build and deployment permissions.
85. Use protected production environments.
86. Require approval where risk requires it.
87. Preserve separation of duties where appropriate.
88. Record approvals.
89. Record promotion events.
90. Record deployment events.
91. Record test results.
92. Record security results.
93. Record artifact identity.
94. Record source identity.
95. Record build identity.
96. Record dependency state.
97. Preserve audit logs.
98. Protect CI logs.
99. Do not expose secrets in debug output.
100. Treat CI logs as potentially sensitive.
```

## 132. Rules 6

```text
101. Validate artifact contents.
102. Validate package metadata.
103. Validate version uniqueness.
104. Validate tag-to-version mapping.
105. Validate artifact-to-source mapping.
106. Validate artifact-to-deployment mapping.
107. Use immutable image digests.
108. Do not use latest as sole production identity.
109. Use progressive deployment when risk requires it.
110. Monitor canaries.
111. Define rollback thresholds.
112. Test rollback.
113. Retain previous known-good artifacts.
114. Account for database compatibility.
115. Use expand/contract migrations where appropriate.
116. Separate application rollback from schema recovery.
117. Monitor post-deployment health.
118. Monitor business metrics where available.
119. Stop unsafe releases quickly.
120. Preserve evidence during incidents.
```

## 133. Rules 7

```text
121. Revoke compromised credentials.
122. Rotate exposed credentials.
123. Quarantine compromised artifacts.
124. Identify consumers of compromised artifacts.
125. Audit affected deployments.
126. Rebuild from trusted source.
127. Re-establish trust after CI compromise.
128. Rotate signing keys after compromise.
129. Review workflow access after incidents.
130. Review repository access after incidents.
131. Maintain incident runbooks.
132. Maintain emergency release procedures.
133. Maintain vulnerability response procedures.
134. Maintain DR procedures.
135. Test DR.
136. Test artifact recovery.
137. Test runner recovery.
138. Test credential recovery.
139. Test signing-key recovery.
140. Keep production recovery practical.
```

## 134. Rules 8

```text
141. Do not manually bypass the pipeline for convenience.
142. Do not manually copy unverified artifacts.
143. Automate repeatable validation.
144. Automate evidence collection.
145. Keep policies version-controlled.
146. Treat pipeline configuration as production code.
147. Review changes to build scripts.
148. Review Dockerfiles.
149. Review package configuration.
150. Review repository configuration.
151. Review release scripts.
152. Review deployment manifests.
153. Keep build environments standardized.
154. Keep dependency sources controlled.
155. Keep release identity immutable.
156. Keep rollback available.
157. Keep audit evidence.
158. Measure delivery performance.
159. Improve speed without weakening security.
160. Design the pipeline so the secure path is the easiest path.
```
---