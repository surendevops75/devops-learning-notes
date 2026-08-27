# 17-JFrog-Artifactory
# 14-Build-Info

## 1. Purpose

This file covers JFrog Artifactory Build Info from fundamentals through production DevOps, CI/CD, security, traceability and release-management practices.

It covers:

- Build Info fundamentals
- Why Build Info matters
- Build identity
- Build name and build number
- CI integration
- Artifacts
- Dependencies
- Environment information
- Source control metadata
- Build modules
- Published artifacts
- Dependency traceability
- Jenkins integration
- GitHub Actions integration
- GitLab CI integration
- Docker builds
- Maven builds
- NPM builds
- Gradle concepts
- Helm and Kubernetes relationships
- Promotion
- Release management
- Build retention
- Security and supply-chain visibility
- Reproducibility
- Production architecture
- Troubleshooting
- Real-world scenarios
- Interview preparation

---

# PART I — BUILD INFO FUNDAMENTALS

## 2. What Is Build Info?

JFrog Build Info is metadata that describes how an artifact was produced.

Conceptually:

```text
Source
 ↓
CI Build
 ↓
Build Info
 ├── artifacts
 ├── dependencies
 ├── environment
 ├── source
 └── build metadata
```

It connects the build process with the artifacts stored in Artifactory.

---

## 3. Why Build Info Matters

Without Build Info:

```text
Artifact
 ↓
What build created this?
Unknown
```

With Build Info:

```text
Artifact
 ↓
Build
 ↓
Source
Dependencies
Environment
Pipeline
```

This improves:

```text
traceability
auditability
release management
incident response
dependency analysis
promotion
```

---

## 4. Build Info Is Not the Artifact

Important distinction:

```text
Artifact
→ actual package/image/chart

Build Info
→ metadata describing how it was built
```

Example:

```text
payment-service-4.2.1.jar
```

plus:

```text
Build:
payment-service
Number:
721
Commit:
abc1234
Dependencies:
...
```

---

# PART II — BUILD IDENTITY

## 5. Build Name

A build name identifies the logical build pipeline/application.

Example:

```text
payment-service
```

---

## 6. Build Number

A build number identifies one execution of that build.

Example:

```text
721
```

---

## 7. Build Name + Number

Together:

```text
payment-service
+
721
```

identify a specific build record.

---

## 8. Example

```text
Build Name:
payment-service

Build Number:
721

Version:
4.2.1
```

The build number and artifact version do not have to be identical.

---

## 9. Build URL

Build metadata may include a link to the CI build.

Example concept:

```text
Jenkins build:
https://jenkins.example/job/payment/721
```

The exact URL depends on the CI system.

---

# PART III — BUILD INFO CONTENT

## 10. Artifacts

Build Info can associate artifacts produced by the build.

Example:

```text
payment-service-4.2.1.jar
```

---

## 11. Dependencies

Build Info can capture dependencies used by a build.

Example:

```text
Spring
Jackson
Netty
```

The exact dependency collection depends on the build technology and integration.

---

## 12. Environment Information

Build metadata may include information such as:

```text
CI system
build environment
selected environment variables
tool versions
```

Avoid capturing secrets.

---

## 13. Source Control

Build Info can associate source information such as:

```text
Git repository
branch
commit
tag
```

---

## 14. Build Properties

Additional metadata can describe:

```text
team
application
release
environment
pipeline
```

Use a controlled naming policy.

---

# PART IV — BUILD MODULES

## 15. What Is a Build Module?

A build may consist of multiple modules.

Example:

```text
payment-platform
 ├── payment-api
 ├── payment-service
 └── payment-worker
```

---

## 16. Multi-Module Build

A single CI build can produce:

```text
payment-api-4.2.1.jar
payment-service-4.2.1.jar
payment-worker-4.2.1.jar
```

Build Info can provide relationships between the build and these outputs.

---

## 17. Why Modules Matter

Modules help with:

```text
artifact traceability
dependency relationships
release visibility
large repository management
```

---

# PART V — BUILD INFO LIFECYCLE

## 18. Build Info Lifecycle

Typical flow:

```text
CI starts
 ↓
Collect build data
 ↓
Build
 ↓
Publish artifacts
 ↓
Publish Build Info
 ↓
Promote/release
```

---

## 19. Build Started

CI identifies:

```text
build name
build number
source revision
```

---

## 20. Dependencies Collected

The build system resolves:

```text
Maven dependencies
NPM dependencies
Python dependencies
base images
other artifacts
```

depending on the integration.

---

## 21. Artifacts Published

Examples:

```text
JAR
NPM package
Docker image
Helm chart
```

---

## 22. Build Info Published

After the build:

```text
Build Info
 ↓
Artifactory
```

The build record can then be used for traceability and release operations.

---

# PART VI — JFROG CLI

## 23. JFrog CLI

JFrog CLI is commonly used to interact with JFrog products and automate artifact/build operations.

Conceptually:

```text
CI
 ↓
JFrog CLI
 ↓
Artifactory
```

---

## 24. Configure JFrog CLI

A CI environment should authenticate using:

```text
service identity
+
scoped token
```

Do not hardcode credentials.

---

## 25. Build Environment

JFrog tooling can be configured to associate operations with:

```text
build name
build number
```

---

## 26. Collect Environment Data

Build information may include selected environment variables.

Important:

```text
Do not collect secrets.
```

Avoid exposing:

```text
passwords
tokens
private keys
cloud credentials
```

---

# PART VII — MAVEN + BUILD INFO

## 27. Maven Build

Typical flow:

```text
Git
 ↓
Maven
 ↓
Dependencies
 ↓
Compile/Test
 ↓
Package
 ↓
Artifactory
```

---

## 28. Maven Artifact

Example:

```text
payment-service-4.2.1.jar
```

Build Info associates the artifact with the build.

---

## 29. Maven Dependencies

Example:

```text
payment-service
 ├── spring
 ├── jackson
 └── netty
```

This dependency graph is useful during vulnerability analysis.

---

## 30. Maven Production Traceability

Example:

```text
Git:
abc1234

Build:
payment-service #721

Artifact:
payment-service-4.2.1.jar
```

---

# PART VIII — GRADLE + BUILD INFO

## 31. Gradle

Gradle builds can also participate in JFrog build-information workflows.

Example:

```text
Gradle
 ↓
Build
 ↓
Artifacts
 ↓
Artifactory
 ↓
Build Info
```

---

## 32. Multi-Project Gradle

Example:

```text
platform
 ├── api
 ├── service
 └── worker
```

Build Info can help connect produced artifacts to the build.

---

# PART IX — NPM + BUILD INFO

## 33. NPM Build

Flow:

```text
Git
 ↓
npm ci
 ↓
npm test
 ↓
npm pack
 ↓
Publish
```

---

## 34. NPM Artifact

Example:

```text
payment-client-4.2.1.tgz
```

---

## 35. NPM Dependencies

Build metadata can associate dependencies with the build where supported by the chosen JFrog tooling and integration.

---

# PART X — DOCKER + BUILD INFO

## 36. Docker Build

Example:

```bash
docker build \
  -t artifactory.company.com/docker-local/payment-service:4.2.1 \
  .
```

---

## 37. Docker Push

```bash
docker push \
  artifactory.company.com/docker-local/payment-service:4.2.1
```

---

## 38. Docker Build Info

A production build record should connect:

```text
Docker image
+
build number
+
Git commit
+
base image/dependencies
```

where supported by the integration.

---

## 39. Image Digest

Example:

```text
sha256:abcdef...
```

Track the digest because it provides stronger content identity than a mutable tag.

---

# PART XI — HELM + BUILD INFO

## 40. Helm Build

Example:

```bash
helm lint ./payment-service
helm package ./payment-service
```

Output:

```text
payment-service-1.4.0.tgz
```

---

## 41. Helm Build Metadata

Connect:

```text
chart version
appVersion
Git commit
CI build
```

to the release process.

---

## 42. Helm + Docker Relationship

Example:

```text
Helm:
1.4.0

Application:
4.2.1

Docker:
4.2.1
sha256:...
```

Build/release metadata should make these relationships traceable.

---

# PART XII — JENKINS INTEGRATION

## 43. Jenkins Architecture

```text
Git
 ↓
Jenkins
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
```

---

## 44. Jenkins Build Identity

Example:

```text
Job:
payment-service

Build:
721
```

Use stable build naming.

---

## 45. Jenkins Metadata

Useful metadata:

```text
BUILD_NUMBER
GIT_COMMIT
GIT_BRANCH
BUILD_URL
```

Do not publish sensitive environment variables into Build Info.

---

## 46. Jenkins + Artifactory

Conceptually:

```text
Jenkins
 ↓
JFrog integration/CLI
 ↓
Artifactory
```

The exact plugin and configuration depend on the Jenkins/JFrog versions deployed.

---

# PART XIII — GITHUB ACTIONS

## 47. GitHub Actions Flow

```text
GitHub
 ↓
Actions
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
```

---

## 48. GitHub Commit

Use:

```text
GITHUB_SHA
```

or equivalent workflow metadata to associate the build with source.

---

## 49. GitHub Release

Example:

```text
v4.2.1
```

The release tag can provide the product version while the workflow run number provides CI execution identity.

---

# PART XIV — GITLAB CI

## 50. GitLab Flow

```text
GitLab
 ↓
Runner
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
```

---

## 51. GitLab Metadata

Useful values may include:

```text
CI_COMMIT_SHA
CI_COMMIT_REF_NAME
CI_PIPELINE_ID
CI_JOB_ID
```

Use only the metadata required by the build-information policy.

---

# PART XV — BUILD INFO + DEPENDENCIES

## 52. Dependency Graph

Conceptually:

```text
Application
 |
 +-- Dependency A
 |
 +-- Dependency B
 |
 +-- Dependency C
```

---

## 53. Why Dependency Tracking Matters

Suppose:

```text
Dependency B
```

has a critical vulnerability.

You need to determine:

```text
Which builds use B?
Which artifacts contain B?
Which production deployments are affected?
```

---

## 54. Incident Response

Build Info can help answer:

```text
Which build created this artifact?
What dependencies were used?
Which source commit created it?
Where was it published?
```

---

# PART XVI — BUILD INFO + SECURITY

## 55. Software Supply Chain

Build Info supports supply-chain visibility by connecting:

```text
source
+
build
+
dependencies
+
artifacts
```

---

## 56. Dependency Vulnerability

Flow:

```text
CVE detected
 ↓
Find affected dependency
 ↓
Find builds containing dependency
 ↓
Find artifacts
 ↓
Find deployments
 ↓
Remediate
```

---

## 57. Provenance

Build provenance describes:

```text
where artifact came from
how it was built
which source produced it
```

Build Info can contribute important provenance metadata.

---

# PART XVII — BUILD INFO + PROMOTION

## 58. Promotion

A build can be associated with artifacts that are promoted through environments.

Conceptually:

```text
Build 721
 ↓
Artifact 4.2.1
 ↓
DEV
 ↓
STAGE
 ↓
PROD
```

---

## 59. Why Promote by Build?

Build-based promotion helps ensure:

```text
exact tested artifact
```

moves through environments.

---

## 60. Build Promotion

Avoid:

```text
Rebuild for STAGE
Rebuild for PROD
```

Prefer:

```text
Build once
 ↓
Promote
```

---

# PART XVIII — RELEASE TRACEABILITY

## 61. Release Record

A release can be represented by:

```text
Application:
payment-service

Version:
4.2.1

Build:
721

Commit:
abc1234

Docker Digest:
sha256:...

Helm:
1.4.0
```

---

## 62. Production Question

During an incident:

```text
What is running?
```

Answer should be traceable to:

```text
artifact
digest
build
commit
release
```

---

# PART XIX — BUILD INFO RETENTION

## 63. Why Retain Build Info?

For:

```text
audit
incident response
release history
dependency analysis
support
compliance
```

---

## 64. Build Info vs Artifact Retention

They are related but different.

```text
Artifact
→ actual binary/image/package

Build Info
→ metadata
```

Deleting one may affect traceability of the other.

---

## 65. Retention Strategy

Retain Build Info for at least the period required to support:

```text
artifact support
production rollback
audit
security investigation
```

---

# PART XX — BUILD INFO SECURITY

## 66. Do Not Store Secrets

Never intentionally capture:

```text
passwords
tokens
private keys
cloud secrets
database passwords
```

---

## 67. Environment Variable Risk

CI environments may contain:

```text
AWS_SECRET_ACCESS_KEY
ARTIFACTORY_TOKEN
DB_PASSWORD
```

Do not blindly publish every environment variable into build metadata.

---

## 68. Selective Collection

Prefer:

```text
approved metadata
```

instead of:

```text
all environment variables
```

---

# PART XXI — PRODUCTION ARCHITECTURE

## 69. Complete Architecture

```text
                         Git
                          |
                          v
                         CI
                          |
                 +--------+--------+
                 |        |        |
               Build     Test     Scan
                 |        |        |
                 +--------+--------+
                          |
                          v
                    Artifactory
                     /        \
                    /          \
             Artifacts       Build Info
                  |               |
                  +-------+-------+
                          |
                          v
                      Promotion
                          |
                          v
                    Kubernetes
```

---

## 70. Multi-Artifact Build

```text
Build #721
    |
    +--> Maven JAR 4.2.1
    |
    +--> Docker Image 4.2.1
    |
    +--> Helm Chart 1.4.0
```

All should be traceable to the same release/build context where appropriate.

---

## 71. Production Traceability

```text
Production Pod
 ↓
Image digest
 ↓
Docker image
 ↓
Build #721
 ↓
Git commit abc1234
 ↓
Release v4.2.1
```

---

# PART XXII — KUBERNETES AND GITOPS

## 72. Kubernetes

Kubernetes consumes the resulting image:

```text
Kubernetes
 ↓
Artifactory Docker Repository
 ↓
Image
```

---

## 73. GitOps

```text
Git
 ↓
Argo CD
 ↓
Helm
 ↓
Artifactory
 ↓
Kubernetes
```

Build Info can remain the traceability layer connecting the artifact to the CI build.

---

## 74. Multi-Cluster

Example:

```text
Build #721
 ↓
Artifact 4.2.1
 ↓
EKS-Dev
 ↓
EKS-Stage
 ↓
EKS-Prod
```

The same artifact should be promoted rather than rebuilt.

---

# PART XXIII — TROUBLESHOOTING BUILD INFO

## 75. Artifact Exists but Build Info Missing

Check:

```text
CI integration
JFrog CLI configuration
build name
build number
publish-build-info step
permissions
```

---

## 76. Wrong Build Number

Check:

```text
CI environment
build configuration
pipeline variables
parallel jobs
manual overrides
```

---

## 77. Wrong Git Commit

Check:

```text
checkout stage
detached HEAD
merge commit
tag
CI source metadata
```

---

## 78. Missing Dependencies

Check:

```text
dependency collection
build tool integration
JFrog CLI configuration
dependency resolution path
```

---

## 79. Secrets Appeared in Build Info

Immediate:

```text
Treat secret as exposed
 ↓
Rotate/revoke
 ↓
Remove future collection
 ↓
Review access
 ↓
Audit
```

Do not rely on deleting metadata alone as a complete security response.

---

## 80. Build Info References Wrong Artifact

Check:

```text
build name
build number
repository
artifact version
pipeline concurrency
publication step
```

---

# PART XXIV — REAL-WORLD SCENARIOS

## 81. Scenario — Find Who Built a Production Image

Use:

```text
image tag/digest
 ↓
Artifactory artifact
 ↓
Build Info
 ↓
CI build
 ↓
Git commit
```

---

## 82. Scenario — Critical CVE Found

Question:

```text
Which production applications use this dependency?
```

Use build/dependency relationships to identify:

```text
affected builds
affected artifacts
affected releases
```

Then verify actual production deployment state separately.

---

## 83. Scenario — Wrong Image Deployed

Trace:

```text
Kubernetes deployment
 ↓
image digest
 ↓
Artifactory
 ↓
Build Info
 ↓
CI pipeline
 ↓
Git commit
```

---

## 84. Scenario — Two Artifacts Have Same Application Version

Use:

```text
checksum/digest
build number
Git commit
Build Info
```

to determine whether they are actually identical.

---

## 85. Scenario — Pipeline Rebuilt for Production

Problem:

```text
STAGE artifact
≠
PROD artifact
```

Fix:

```text
build once
 ↓
publish
 ↓
promote
```

---

## 86. Scenario — Build Info Contains Secrets

Response:

```text
Revoke/rotate secret
 ↓
Identify exposure
 ↓
Fix collection rules
 ↓
Audit access
 ↓
Review historical metadata
```

---

# PART XXV — INTERVIEW PREPARATION

## 87. What Is JFrog Build Info?

Answer:

```text
Build Info is metadata that describes a CI build, including its
artifacts, dependencies and relevant source/build information. It
connects the produced artifact to the process that created it.
```

---

## 88. Why Is Build Info Important?

Answer:

```text
It provides traceability from production artifacts back to the build,
source commit and dependencies. It is especially useful for release
management, security investigations and supply-chain visibility.
```

---

## 89. What Is Build Name vs Build Number?

Answer:

```text
Build name identifies the logical pipeline or application, while
build number identifies a particular execution of that build.
```

---

## 90. How Do You Trace a Docker Image to Git?

Answer:

```text
I start with the deployed image digest, locate the corresponding
artifact and Build Info, identify the CI build and then map that build
to the Git commit or release tag.
```

---

## 91. How Does Build Info Help During a CVE?

Answer:

```text
I can identify which builds contain the vulnerable dependency, map
those builds to artifacts and then determine which releases may be
affected. I then verify actual production deployments and remediate
the vulnerable artifacts.
```

---

## 92. Why Should You Not Capture All Environment Variables?

Answer:

```text
CI environments often contain credentials and secrets. Blindly
capturing all variables can expose sensitive information in build
metadata. I collect only approved non-sensitive metadata.
```

---

## 93. How Does Build Info Support Build Once, Promote Many?

Answer:

```text
It provides the metadata linking the tested artifact to its build,
dependencies and source. The same artifact/build can then be promoted
through environments without rebuilding.
```

---

## 94. How Would You Use Build Info in a Production Environment?

Answer:

```text
I use it for artifact traceability, dependency visibility, release
promotion, incident response, security investigations and audit.
Every production artifact should be traceable to its CI build and
source revision.
```

---

## 95. What If Build Info Is Missing?

Answer:

```text
I verify the CI/JFrog integration, build name and number, artifact
publication and Build Info publication step, then check service
identity permissions and pipeline configuration.
```

---

# PART XXVI — PRODUCTION CHECKLIST

## 96. Build Identity

```text
[ ] build name
[ ] build number
[ ] CI URL
[ ] source commit
[ ] release tag
```

---

## 97. Artifacts

```text
[ ] artifacts recorded
[ ] versions recorded
[ ] repository recorded
[ ] checksums/digests tracked
```

---

## 98. Dependencies

```text
[ ] dependencies collected
[ ] transitive dependencies where supported
[ ] vulnerability traceability
[ ] approved dependency sources
```

---

## 99. Security

```text
[ ] no secrets in Build Info
[ ] selective environment metadata
[ ] least-privilege CI identity
[ ] audit
[ ] token rotation
```

---

## 100. Promotion

```text
[ ] build once
[ ] tested artifact
[ ] Build Info published
[ ] controlled promotion
[ ] production traceability
```

---

## 101. Operations

```text
[ ] Build Info retention
[ ] artifact retention
[ ] incident procedures
[ ] backup/DR
[ ] monitoring
```

---

# PART XXVII — GOLDEN RULES

## 102. Rules

```text
1. Treat Build Info as a core software-supply-chain metadata layer.

2. Connect every production artifact to its CI build.

3. Track source commit and release tag.

4. Track artifact versions and Docker digests.

5. Track dependencies where supported.

6. Use Build Info to improve promotion and release traceability.

7. Build once and promote the same artifact.

8. Do not capture secrets in Build Info.

9. Collect only approved environment metadata.

10. Use dedicated least-privilege CI identities.

11. Retain Build Info for required audit and incident-response
    periods.

12. Use Build Info during vulnerability investigations.

13. Correlate Build Info with actual Kubernetes deployment state.

14. Do not assume Build Info alone proves what is currently running;
    verify the live deployment.

15. Maintain consistent build naming across CI pipelines.

16. Avoid manually overriding build numbers without a documented
    reason.

17. Keep multi-module build relationships traceable.

18. Treat missing Build Info as a traceability gap.

19. Audit Build Info publication and access.

20. Protect production artifacts and their associated metadata.

21. Use Build Info as part of release governance, not as a substitute
    for testing or security scanning.

22. Validate exact JFrog CLI, plugin, build-info schema and CI
    integration behavior against the deployed JFrog/Artifactory
    version before production rollout.
```

---

# END OF 14-Build-Info.md
