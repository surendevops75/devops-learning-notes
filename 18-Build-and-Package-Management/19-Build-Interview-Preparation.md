# 18-Build-and-Package-Management
# 19-Build-Interview-Preparation

## 1. Purpose

This file is a production-oriented interview preparation guide for
Build and Package Management.

The goal is not to memorize definitions.

A strong DevOps engineer should be able to explain:

```text
what the technology does
why it is needed
how it is configured
how it behaves in production
how it fails
how it is secured
how it integrates with CI/CD
how artifacts are promoted
how incidents are handled
```

Core technologies and concepts:

```text
Build Systems
Dependency Management
Maven
pom.xml
Maven Lifecycle
Maven Dependencies and Plugins
Multi-Module Maven
Repository Management
Jenkins
GitHub Actions
npm
Node.js
Python Package Management
Build Caching
Versioning
Release Management
Artifact Publishing
Build Security
Production Build Pipelines
Troubleshooting
Artifactory
Docker
Helm
Kubernetes
GitOps
Argo CD
```

---

# PART I — INTERVIEW ANSWER FRAMEWORK

## 2. Answer Structure

For most production questions use:

```text
Definition
   |
   v
How I use it
   |
   v
Production considerations
   |
   v
Security
   |
   v
Failure handling
```

---

## 3. Example

Question:

```text
How do you manage Maven builds?
```

Strong answer:

```text
I use Maven as the Java build and dependency-management system. I keep
the project configuration in pom.xml, control dependency and plugin
versions, use a private repository such as Artifactory for internal and
approved external dependencies, run clean verify in CI, execute security
and quality gates, and publish immutable release artifacts. For
production releases I build once and promote the same artifact across
environments.
```

---

# PART II — BUILD FUNDAMENTALS

## 4. What Is a Build System?

Answer:

```text
A build system automates compilation, dependency resolution, testing,
packaging and related validation required to turn source code into a
deployable artifact.
```

---

## 5. Why Do We Need Build Automation?

Answer:

```text
Manual builds are inconsistent and difficult to reproduce. Automation
provides repeatability, standardization, traceability and integration
with CI/CD.
```

---

## 6. What Is an Artifact?

Answer:

```text
An artifact is a generated software output such as a JAR, npm package,
Python wheel, Docker image, Helm chart or binary that can be stored and
consumed by another system.
```

---

## 7. Artifact vs Source

Answer:

```text
Source code is the input to the build. The artifact is the validated
output. The artifact should be traceable back to the exact source
commit and build that produced it.
```

---

## 8. Build Once, Promote Many

Answer:

```text
I build the artifact once, validate it, publish it immutably and
promote the same artifact through development, staging and production.
I avoid rebuilding separately for every environment because that can
produce different binaries.
```

---

# PART III — MAVEN

## 9. What Is Maven?

Answer:

```text
Maven is a Java build and dependency-management tool based around a
standard project model and lifecycle. It handles compilation,
testing, packaging, dependency resolution and plugin execution.
```

---

## 10. What Is pom.xml?

Answer:

```text
pom.xml is Maven's project configuration file. It defines project
coordinates, dependencies, plugins, build configuration, repositories
and other project metadata.
```

---

## 11. Maven Coordinates

Answer:

```text
Maven commonly identifies an artifact using groupId, artifactId and
version. For example:
com.company.payment:payment-api:1.5.0
```

---

## 12. What Is groupId?

Answer:

```text
groupId identifies the logical organization or namespace of the
artifact.
```

---

## 13. What Is artifactId?

Answer:

```text
artifactId identifies the specific project or package within the
group.
```

---

## 14. What Is version?

Answer:

```text
The version identifies the release or development state of the
artifact.
```

---

## 15. Maven Lifecycle

Answer:

```text
Maven provides standard lifecycles and phases. A common sequence is
validate, compile, test, package, verify and install/deploy depending
on the required workflow.
```

---

## 16. package vs install vs deploy

Answer:

```text
package creates the distributable artifact.
install places it into the local Maven repository.
deploy publishes it to the configured remote repository.
```

---

## 17. What Does clean Do?

Answer:

```text
clean removes previous build output, commonly the target directory,
so the next build starts from a clean state.
```

---

## 18. What Does verify Do?

Answer:

```text
verify runs the validation phases through integration-level checks
configured for the project. It is commonly useful in CI before
publication.
```

---

# PART IV — MAVEN DEPENDENCIES

## 19. Dependency

Answer:

```text
A dependency is an external library required by the application or
build. Maven resolves it using configured repositories and the
dependency graph.
```

---

## 20. Transitive Dependency

Answer:

```text
A transitive dependency is a dependency brought in by another
dependency rather than being declared directly by the application.
```

---

## 21. How Do You Find Dependency Conflicts?

Answer:

```text
I inspect the dependency tree, identify which paths introduce the
different versions and then use dependency management or exclusions
only when I understand the compatibility impact.
```

Example:

```bash
./mvnw dependency:tree
```

---

## 22. What Is Dependency Management?

Answer:

```text
Dependency management centralizes version and dependency behavior so
multi-module projects can use consistent versions.
```

---

## 23. Why Avoid Random Exclusions?

Answer:

```text
An exclusion can hide a required transitive dependency and create a
runtime failure. I use exclusions only after understanding the graph
and validating the application.
```

---

# PART V — MAVEN PLUGINS

## 24. What Is a Maven Plugin?

Answer:

```text
A Maven plugin provides executable build functionality such as
compilation, testing, packaging, code generation or publishing.
```

---

## 25. Are Maven Plugins Security-Sensitive?

Answer:

```text
Yes. Plugins execute during the build, so they are part of the build
supply chain. I control their versions and repository sources.
```

---

# PART VI — MULTI-MODULE MAVEN

## 26. Why Multi-Module?

Answer:

```text
A multi-module Maven project allows related modules to share build
configuration and dependency management while producing separate
artifacts.
```

---

## 27. Example

```text
parent
 |
+--> common
+--> payment-api
+--> payment-client
+--> payment-worker
```

---

## 28. Parent POM

Answer:

```text
The parent can centralize properties, dependency management, plugin
configuration and common build behavior.
```

---

# PART VII — MAVEN REPOSITORIES

## 29. Local Repository

Answer:

```text
The local Maven repository stores downloaded dependencies and locally
installed artifacts, commonly under ~/.m2/repository.
```

---

## 30. Remote Repository

Answer:

```text
A remote repository stores artifacts accessible over the network.
Enterprise environments often use Artifactory or another repository
manager.
```

---

## 31. Virtual Repository

Answer:

```text
A virtual repository provides one logical endpoint over selected local
and remote repositories, simplifying client configuration and enabling
central governance.
```

---

# PART VIII — ARTIFACTORY

## 32. Why Artifactory?

Answer:

```text
Artifactory provides centralized artifact storage and dependency
distribution across ecosystems such as Maven, npm, PyPI, Docker and
Helm. It also supports access control, metadata, lifecycle management
and integration with CI/CD.
```

---

## 33. Local vs Remote vs Virtual

Answer:

```text
Local stores internally produced artifacts.
Remote proxies or caches upstream repositories.
Virtual provides a unified consumer endpoint across configured
repositories.
```

---

## 34. How Do You Secure Artifactory?

Answer:

```text
I use RBAC and least privilege, separate read/write permissions,
protect release repositories, restrict delete access, use secure
credentials, audit access and enforce artifact immutability.
```

---

# PART IX — NPM AND NODE.JS

## 35. What Is npm?

Answer:

```text
npm is the package manager commonly used in the Node.js ecosystem. It
resolves dependencies, runs package scripts and publishes packages.
```

---

## 36. npm install vs npm ci

Answer:

```text
npm install can update or modify dependency resolution and lockfile
state. npm ci is designed for clean CI installation using the lockfile
and is generally preferred for reproducible CI builds.
```

---

## 37. package-lock.json

Answer:

```text
package-lock.json records the resolved dependency tree so CI and other
developers can reproduce the intended dependency versions.
```

---

## 38. npm Publishing

Answer:

```text
I validate package metadata and contents, authenticate to the approved
private registry, publish a unique version and verify the package.
```

---

# PART X — PYTHON PACKAGE MANAGEMENT

## 39. Common Python Package Outputs

Answer:

```text
A Python project may produce a wheel and/or source distribution.
```

---

## 40. Build Python Package

Example:

```bash
python -m build
```

---

## 41. Wheel vs sdist

Answer:

```text
A wheel is a built distribution intended to simplify installation,
while an sdist contains source/build material and may require a build
step on the consumer side.
```

---

# PART XI — BUILD CACHING

## 42. Why Cache?

Answer:

```text
Caching reduces repeated dependency downloads and expensive build work,
which can significantly reduce CI duration.
```

---

## 43. Can a Build Depend on Cache?

Answer:

```text
No. Cache should accelerate a correct build. A clean cold-cache build
should still work.
```

---

## 44. Cache Security

Answer:

```text
I never cache secrets and I scope caches so untrusted jobs cannot
poison artifacts or consume sensitive cached state.
```

---

# PART XII — VERSIONING

## 45. Semantic Versioning

Answer:

```text
Semantic Versioning commonly uses MAJOR.MINOR.PATCH. Major indicates
breaking changes, minor indicates compatible functionality additions
and patch indicates compatible fixes.
```

---

## 46. Why Unique Release Versions?

Answer:

```text
A release version should identify one immutable artifact. If the
contents change, I publish a new version instead of replacing the old
one.
```

---

## 47. Snapshot Version

Answer:

```text
A snapshot represents development state and can have mutable semantics.
I do not use snapshots as immutable production releases.
```

---

# PART XIII — ARTIFACT PUBLISHING

## 48. What Is Artifact Publishing?

Answer:

```text
Artifact publishing is the process of taking a validated build output,
assigning a stable identity and storing it in an approved repository
for consumption or promotion.
```

---

## 49. Publishing Flow

```text
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
SBOM/Provenance
 |
v
Publish
 |
v
Promote
```

---

## 50. What If Artifact Version Already Exists?

Answer:

```text
I do not overwrite a production artifact. I verify whether it was
already published and create a new version if the artifact content
needs to change.
```

---

# PART XIV — BUILD SECURITY

## 51. What Is Build Security?

Answer:

```text
Build security protects the complete software supply chain from source,
dependencies and build infrastructure through artifact publication
and deployment.
```

---

## 52. How Do You Secure CI Credentials?

Answer:

```text
I use dedicated identities, least privilege, secure secret stores and
short-lived credentials where possible. For cloud access I prefer
OIDC or workload identity over long-lived static credentials.
```

---

## 53. How Do You Secure PR Builds?

Answer:

```text
I treat PR code as potentially untrusted. PR workflows use restricted
permissions and do not receive production credentials, signing keys or
artifact administration access.
```

---

## 54. What Is Dependency Confusion?

Answer:

```text
Dependency confusion occurs when a malicious public package is selected
instead of an intended internal package. I mitigate it with private
repositories, namespaces, controlled registries, dependency pinning and
source policy.
```

---

## 55. Why SBOM?

Answer:

```text
An SBOM gives us a component inventory for an artifact. During a
vulnerability incident it allows us to quickly identify which releases
contain an affected component.
```

---

## 56. Why Provenance?

Answer:

```text
Provenance records how an artifact was built, including source, builder
and relevant inputs. It supports traceability and supply-chain trust.
```

---

## 57. Why Sign Artifacts?

Answer:

```text
Signing allows consumers or deployment systems to verify that an
artifact was produced or approved by a trusted identity and was not
altered after signing.
```

---

# PART XV — PRODUCTION BUILD PIPELINES

## 58. Explain Your Production Pipeline

Answer:

```text
A typical pipeline starts from a protected source revision. CI performs
linting, secret scanning, compilation, unit tests, SAST and dependency
checks. The application is packaged, scanned and associated with SBOM
and provenance information. The immutable artifact is published to
Artifactory and promoted through environments without rebuilding.
Production deployment is protected by approvals, health checks and
rollback procedures.
```

---

## 59. Pipeline Stages

```text
Checkout
 |
v
Validate
 |
v
Build
 |
v
Unit Test
 |
v
SAST/SCA
 |
v
Integration Test
 |
v
Package
 |
v
SBOM
 |
v
Publish
 |
v
Promote
```

---

## 60. Why Fail Fast?

Answer:

```text
Fast validation prevents expensive downstream stages from consuming
resources when a basic condition such as compilation, linting or secret
scanning has already failed.
```

---

# PART XVI — PIPELINE SECURITY

## 61. Least Privilege

Answer:

```text
Each job receives only the permissions it needs. A PR job may only read
source and dependencies, while a trusted release job receives the
specific repository write permission required for publication.
```

---

## 62. Ephemeral Runners

Answer:

```text
Ephemeral runners are created for a job and destroyed afterward. They
reduce persistent workspace contamination and cross-job state.
```

---

## 63. Persistent Runners

Answer:

```text
If persistent runners are required, I harden and patch them, isolate
workspaces, clean state, restrict network access and monitor them.
```

---

# PART XVII — PIPELINE PERFORMANCE

## 64. Pipeline Is Slow

Answer:

```text
I measure queue time and each pipeline stage first. Then I optimize
the dominant bottleneck using dependency caching, parallel jobs,
test parallelization, appropriate runners or repository mirrors. I
do not remove mandatory security controls simply to make the pipeline
faster.
```

---

## 65. Parallel Jobs

Example:

```text
             +--> Unit tests
Build -------+--> SAST
             +--> SCA
             +--> Lint
```

Aggregate required results before publication.

---

## 66. Retry Policy

Answer:

```text
I retry transient infrastructure failures where appropriate, but I do
not blindly retry deterministic test or compilation failures because
that hides real problems.
```

---

# PART XVIII — TROUBLESHOOTING

## 67. Build Works Locally but Fails in CI

Answer:

```text
I compare tool versions, operating system, dependencies, environment
variables, credentials, network access, workspace state and caches. I
then reproduce the issue using the CI toolchain and a clean workspace.
```

---

## 68. Maven Dependency Not Found

Answer:

```text
I check coordinates, repository configuration, authentication,
authorization, DNS/TLS/network connectivity, repository health and
local cache state.
```

---

## 69. Artifactory 401

Answer:

```text
I investigate authentication: credential validity, token expiration,
identity and registry endpoint.
```

---

## 70. Artifactory 403

Answer:

```text
I investigate authorization: the authenticated identity, repository
permissions, path permissions and requested operation.
```

---

## 71. Artifactory 404

Answer:

```text
I verify repository name, artifact coordinates, version, package path
and whether the artifact exists or is available through the selected
virtual repository.
```

---

## 72. Artifactory 409

Answer:

```text
I check for a publication conflict, usually an existing artifact or
concurrent release using the same identity.
```

---

## 73. Artifactory 429

Answer:

```text
I investigate rate limiting and uncontrolled concurrency, then use
approved caching or repository infrastructure rather than bypassing
the enterprise repository.
```

---

# PART XIX — DOCKER BUILD INTERVIEW

## 74. Docker Build Failure

Answer:

```text
I check the Dockerfile, build context, base image, registry access,
BuildKit output and runner resources. I also check disk space and
network connectivity.
```

---

## 75. Docker Image Identity

Answer:

```text
I use a unique versioned tag and record the immutable digest. For
production deployment I prefer the digest as the deployment identity.
```

---

## 76. Image Is Too Large

Answer:

```text
I inspect the build context, unnecessary files, base image and layer
structure. I use multi-stage builds and .dockerignore where
appropriate without removing required runtime components.
```

---

# PART XX — HELM

## 77. What Is Helm?

Answer:

```text
Helm is a Kubernetes package manager and templating system used to
package and deploy Kubernetes resources as versioned charts.
```

---

## 78. Helm Troubleshooting

Answer:

```text
I validate Chart.yaml, values, templates and dependencies and render
the chart before deployment using helm template where appropriate.
```

---

# PART XXI — KUBERNETES

## 79. ImagePullBackOff

Answer:

```text
I verify image name, tag or digest, registry availability,
authentication, imagePullSecrets or workload identity and network
access.
```

---

## 80. CrashLoopBackOff

Answer:

```text
I treat it primarily as a runtime problem. I inspect pod events,
container logs, configuration, probes, resource limits and dependent
services.
```

---

# PART XXII — GITOPS

## 81. Build to GitOps

```text
Build
 |
v
Scan
 |
v
Registry
 |
v
Image Digest
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

## 82. Why GitOps?

Answer:

```text
GitOps makes desired deployment state version-controlled, reviewable
and auditable. The deployment controller reconciles the cluster toward
the declared state.
```

---

# PART XXIII — ARGO CD

## 83. What Does Argo CD Do?

Answer:

```text
Argo CD is a GitOps continuous delivery controller for Kubernetes. It
observes desired state in Git and reconciles Kubernetes resources to
that state.
```

---

## 84. Build vs Deployment

Answer:

```text
CI builds and validates the artifact. Argo CD should normally deploy
the already-built artifact declared by the GitOps configuration rather
than rebuilding application code.
```

---

# PART XXIV — SCENARIO QUESTIONS

## 85. Production Build Suddenly Fails

Answer:

```text
I identify the failed stage, compare it with the last successful build,
inspect recent changes and verify runner, dependency, repository and
credential health. I reproduce safely before making the fix.
```

---

## 86. All Builds Cannot Download Dependencies

Answer:

```text
I check repository health, DNS, TLS, proxy, credentials and network
changes. I also check the enterprise dependency cache or mirror.
```

---

## 87. One Runner Fails

Answer:

```text
I compare it with a healthy runner for tool versions, disk, memory,
network, workspace and configuration. If the runner is unhealthy I
replace or rebuild it instead of modifying application code without
evidence.
```

---

## 88. Build Fails Only With Cold Cache

Answer:

```text
That indicates an undeclared build dependency or generated state. Cache
should be an optimization, so I reproduce from a clean environment and
fix the missing explicit input.
```

---

## 89. Two Releases Use Same Version

Answer:

```text
I prevent concurrent version allocation with release locks or
concurrency controls. Production versions must be unique and
immutable.
```

---

## 90. Stage and Production Have Different Artifacts

Answer:

```text
I compare checksums or container digests and trace the promotion
history. If the artifacts differ, I stop the rollout and correct the
promotion process.
```

---

## 91. Security Scanner Finds Critical CVE

Answer:

```text
I identify affected artifacts and deployments using dependency
metadata and SBOMs, stop further promotion if policy requires it,
upgrade the dependency, rebuild, retest, rescan and publish a new
immutable release.
```

---

## 92. PR Has Access to Production Secret

Answer:

```text
I treat it as a security incident. I revoke or rotate the credential,
audit access, remove the secret from the untrusted workflow and redesign
the permissions using protected environments and least privilege.
```

---

## 93. Signing Key Is Compromised

Answer:

```text
I revoke trust in the key, rotate it, identify artifacts signed with
it, assess deployed versions and establish a new trusted signing path.
```

---

# PART XXV — REAL-WORLD ARCHITECTURE QUESTIONS

## 94. Design Enterprise Build Platform

Answer:

```text
Git
 |
v
Protected PR
 |
v
CI
 |
+--> Secret Scan
+--> SAST
+--> SCA
 |
v
Build
 |
v
Test
 |
v
Package
 |
+--> SBOM
+--> Provenance
+--> Signing
 |
v
Artifactory
 |
v
Promotion
 |
v
GitOps
 |
v
Argo CD
 |
v
Kubernetes
```

Explain:

```text
RBAC
secrets
runner isolation
repository governance
immutable artifacts
observability
DR
rollback
```

---

## 95. How Would You Support Multiple Languages?

Answer:

```text
I standardize the CI framework while allowing language-specific build
steps. Maven handles Java, npm handles Node.js, Python tooling handles
Python packages and Docker handles container packaging. All outputs
are published to the enterprise repository with consistent security,
metadata, provenance and promotion policies.
```

---

# PART XXVI — PRODUCTION SECURITY SCENARIOS

## 96. Malicious Dependency

Answer:

```text
I stop promotion, identify affected releases and consumers, quarantine
where supported, investigate provenance, produce a trusted replacement
and audit downstream deployments.
```

---

## 97. Compromised CI Runner

Answer:

```text
I isolate or destroy the runner, revoke credentials available to it,
audit source and artifacts, investigate possible exfiltration and
rebuild using clean infrastructure.
```

---

## 98. Repository Compromise

Answer:

```text
I isolate repository access, preserve evidence, identify modified
artifacts, verify trusted copies using checksums/signatures, rotate
credentials and restore trusted artifacts.
```

---

# PART XXVII — BEHAVIORAL QUESTIONS

## 99. Tell Me About a Production Build Issue You Solved

Structure:

```text
Situation
Task
Action
Result
Prevention
```

Example:

```text
A release pipeline started failing during artifact publication.
I compared the failed run with the last successful run and found a
repository permission change. I restored the required least-privilege
publish permission, validated the release in a controlled environment
and added permission validation to the release process. The important
result was not only restoring the release but preventing recurrence.
```

---

## 100. How Do You Handle Pressure During a Release?

Answer:

```text
I first stabilize the situation and establish facts. I avoid
uncontrolled changes, identify whether rollback is available, preserve
evidence and communicate clearly. I prefer a known-good artifact and
controlled rollback over an unverified emergency change.
```

---

## 101. What If a Developer Wants to Bypass a Security Gate?

Answer:

```text
I first understand the business impact and security finding. If the
finding is valid, I follow the formal exception process with an owner,
mitigation and expiry. I do not create an undocumented permanent
bypass.
```

---

# PART XXVIII — SENIOR ENGINEER QUESTIONS

## 102. How Do You Improve Build Reliability?

Answer:

```text
I standardize build environments, control dependencies, use clean
reproducible builds, improve test reliability, monitor pipeline
metrics, reduce shared state and maintain resilient artifact
repositories. I also test recovery paths instead of only the happy path.
```

---

## 103. How Do You Improve Developer Experience?

Answer:

```text
I provide standardized pipeline templates, fast feedback, dependency
caching, clear failure messages, reusable build images, consistent
repository endpoints and self-service documentation while preserving
security controls.
```

---

## 104. How Do You Balance Security and Speed?

Answer:

```text
I optimize around security rather than removing it. I parallelize
independent scans, cache safely, use efficient runners and fail fast.
Mandatory controls remain enforced.
```

---

## 105. How Do You Manage Hundreds of Services?

Answer:

```text
I use standardized CI templates, common artifact conventions,
centralized repository management, policy-as-code, reusable security
checks, consistent observability and automated release workflows.
Teams retain application ownership while platform engineering provides
the secure delivery platform.
```

---

## 106. How Do You Handle Multiple Clusters?

Answer:

```text
I separate artifact creation from cluster deployment. CI publishes the
immutable artifact, GitOps records the desired version or digest and
Argo CD reconciles that state across clusters. Cluster-specific
configuration remains in Git rather than being embedded into the build.
```

---

# PART XXIX — ARCHITECTURE TRADE-OFFS

## 107. Maven vs Gradle

Answer:

```text
Maven emphasizes convention, a standard lifecycle and declarative
configuration. Gradle provides a highly programmable build model and
can offer flexibility and performance advantages. I choose based on
project requirements and existing organizational standards.
```

---

## 108. npm vs Yarn/pnpm

Answer:

```text
The important production concerns are reproducible dependency
resolution, lockfile behavior, registry governance, security and CI
performance. The specific manager depends on project standards.
```

---

## 109. Public Registry vs Private Repository

Answer:

```text
A private enterprise repository provides centralized governance,
caching, auditing, access control and availability. Public registries
remain useful as upstream sources but should be accessed according to
organizational policy.
```

---

## 110. Build Cache vs Artifact Repository

Answer:

```text
A cache is disposable acceleration. An artifact repository stores
validated software outputs that must be traceable, durable and
recoverable.
```

---

# PART XXX — TROUBLESHOOTING RAPID FIRE

## 111. 401?

```text
Authentication.
```

## 112. 403?

```text
Authorization.
```

## 113. 404?

```text
Resource/path/version/repository.
```

## 114. 409?

```text
Conflict, often duplicate publication.
```

## 115. 429?

```text
Rate limiting.
```

## 116. OOM?

```text
Memory/resource investigation.
```

## 117. Disk Full?

```text
Storage/capacity investigation.
```

## 118. TLS Failure?

```text
Certificate, CA, hostname, proxy or time investigation.
```

## 119. Works Locally?

```text
Compare environment and dependencies with CI.
```

## 120. Wrong Production Image?

```text
Compare immutable digest and promotion lineage.
```

---

# PART XXXI — COMMAND CHEAT SHEET

## 121. Maven

```bash
./mvnw -B clean verify
./mvnw dependency:tree
./mvnw -B package
./mvnw -B deploy
```

---

## 122. npm

```bash
node --version
npm --version
npm ci
npm test
npm run build
npm pack --dry-run
npm publish
```

---

## 123. Python

```bash
python --version
python -m pip --version
python -m pip list
python -m build
python -m pytest
```

---

## 124. Docker

```bash
docker build -t app:1.0.0 .
docker images
docker system df
docker push registry/app:1.0.0
```

---

## 125. Helm

```bash
helm lint ./chart
helm template release ./chart
helm package ./chart
```

---

## 126. Kubernetes

```bash
kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl get deployment
```

---

# PART XXXII — PRODUCTION CHECKLIST

## 127. Before Release

```text
[ ] source revision confirmed
[ ] version confirmed
[ ] dependencies controlled
[ ] build environment controlled
[ ] tests passed
[ ] SAST passed
[ ] SCA passed
[ ] secret scan passed
[ ] artifact inspected
[ ] SBOM generated where required
[ ] provenance recorded
[ ] artifact signed where required
[ ] artifact published
[ ] artifact immutable
```

---

## 128. Before Production

```text
[ ] exact artifact confirmed
[ ] digest/checksum confirmed
[ ] promotion recorded
[ ] approval complete
[ ] configuration validated
[ ] migration compatibility checked
[ ] health checks ready
[ ] rollback artifact available
[ ] monitoring ready
```

---

# PART XXXIII — GOLDEN INTERVIEW RULES

## 129. Rules 1

```text
1. Answer from production experience.
2. Explain why, not only what.
3. Mention security.
4. Mention reliability.
5. Mention observability.
6. Mention rollback.
7. Mention failure handling.
8. Use real commands where useful.
9. Avoid claiming tools you have not used.
10. Explain trade-offs honestly.
11. Start with a concise answer.
12. Expand when the interviewer asks.
13. Separate build from deployment.
14. Separate cache from artifact.
15. Separate authentication from authorization.
16. Use immutable artifact identity.
17. Preserve source-to-artifact traceability.
18. Build once and promote.
19. Never overwrite production artifacts.
20. Protect publishing credentials.
```

## 130. Rules 2

```text
21. Treat CI YAML as executable code.
22. Treat build plugins as dependencies.
23. Treat package managers as supply-chain components.
24. Protect PR workflows.
25. Use least privilege.
26. Prefer short-lived credentials.
27. Prefer OIDC where supported.
28. Never commit secrets.
29. Rotate exposed credentials.
30. Use SCA.
31. Use SAST.
32. Use secret scanning.
33. Scan containers.
34. Generate SBOMs.
35. Preserve provenance.
36. Sign artifacts where required.
37. Protect signing keys.
38. Use trusted repositories.
39. Control public dependencies.
40. Prevent dependency confusion.
```

## 131. Rules 3

```text
41. Pin dependencies appropriately.
42. Control toolchain versions.
43. Keep builds reproducible.
44. Test cold-cache builds.
45. Cache safely.
46. Never cache secrets.
47. Prefer ephemeral runners.
48. Harden persistent runners.
49. Restrict runner network access.
50. Monitor runner health.
51. Use Artifactory or approved repository management.
52. Use repository RBAC.
53. Restrict delete permissions.
54. Protect release repositories.
55. Maintain retention.
56. Maintain backups.
57. Test recovery.
58. Monitor repository capacity.
59. Use virtual repositories carefully.
60. Use enterprise dependency mirrors where appropriate.
```

## 132. Rules 4

```text
61. Validate artifact contents.
62. Validate package metadata.
63. Validate version uniqueness.
64. Record checksums.
65. Record image digests.
66. Record build IDs.
67. Record commit SHAs.
68. Record promotion.
69. Record approvals.
70. Record deployment.
71. Fail fast on deterministic failures.
72. Retry transient failures only.
73. Parallelize independent work.
74. Measure bottlenecks.
75. Optimize safely.
76. Do not remove security gates for speed.
77. Protect production environments.
78. Protect release tags.
79. Protect release workflows.
80. Prevent concurrent release collisions.
```

## 133. Rules 5

```text
81. Use known-good rollback artifacts.
82. Test rollback.
83. Monitor deployment health.
84. Monitor business health where possible.
85. Use progressive deployment when appropriate.
86. Account for database compatibility.
87. Use backward-compatible migrations.
88. Separate schema rollback from application rollback.
89. Trace deployment to artifact digest.
90. Trace artifact to build.
91. Trace build to source.
92. Use GitOps for controlled deployment state.
93. Keep deployment configuration in Git where appropriate.
94. Keep application builds independent of cluster-specific configuration.
95. Use Argo CD or an approved controller for Kubernetes reconciliation.
96. Protect GitOps repositories.
97. Protect cluster credentials.
98. Keep production audit evidence.
99. Practice incident response.
100. Design the entire delivery chain for safe recovery.
```

# END OF 19-Build-Interview-Preparation.md
