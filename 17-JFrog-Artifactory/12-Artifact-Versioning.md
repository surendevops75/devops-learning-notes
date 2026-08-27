# 17-JFrog-Artifactory
# 12-Artifact-Versioning

## 1. Purpose

This file covers artifact versioning in JFrog Artifactory from fundamentals through production DevOps practices.

It covers:

- Artifact identity
- Versioning fundamentals
- Semantic Versioning
- Maven versions
- NPM versions
- PyPI versions
- Docker tags and digests
- Helm chart versions
- Snapshot and release versions
- Immutable artifacts
- Build numbers
- Git commit traceability
- Artifact promotion
- Build once and promote many
- CI/CD version generation
- Jenkins
- GitHub Actions
- GitLab CI
- Kubernetes
- Release management
- Rollback
- Retention
- Artifact lifecycle
- Versioning security
- Reproducibility
- Production architecture
- Troubleshooting
- Real-world scenarios
- Interview preparation

---

# PART I — VERSIONING FUNDAMENTALS

## 2. What Is Artifact Versioning?

Artifact versioning gives a unique identity to a software artifact.

Examples:

```text
payment-service-4.2.1.jar
payment-client-4.2.1.whl
payment-service:4.2.1
payment-service-1.4.0.tgz
```

Versioning allows teams to answer:

```text
What was released?
When?
From which source?
Which build created it?
Which production deployment uses it?
Can we roll back?
```

---

## 3. Why Versioning Matters

Good versioning provides:

```text
traceability
reproducibility
rollback
auditability
dependency management
release clarity
```

---

## 4. Artifact Identity

A production artifact should be identifiable by more than a friendly name.

Strong traceability can include:

```text
artifact name
version
Git commit
build number
pipeline
timestamp
digest/checksum
environment
release metadata
```

---

## 5. Example

```text
payment-service
version: 4.2.1
commit: abc1234
build: 721
Docker digest: sha256:...
```

This allows operations teams to trace a deployed artifact back to its source and build.

---

# PART II — SEMANTIC VERSIONING

## 6. Semantic Versioning

Semantic Versioning commonly uses:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
4.2.1
```

---

## 7. MAJOR

A major version generally represents incompatible API or behavior changes.

Example:

```text
4.x
→
5.x
```

---

## 8. MINOR

A minor version generally adds backward-compatible functionality.

Example:

```text
4.2.0
→
4.3.0
```

---

## 9. PATCH

A patch version generally contains backward-compatible fixes.

Example:

```text
4.2.1
→
4.2.2
```

---

## 10. SemVer Is a Policy

Semantic Versioning is useful, but teams must define how it applies to their software.

For example:

```text
breaking API
→ major

new backward-compatible feature
→ minor

bug/security fix
→ patch
```

---

# PART III — MAVEN VERSIONING

## 11. Maven Artifact

Example:

```text
groupId:
com.company.payment

artifactId:
payment-service

version:
4.2.1
```

Coordinates:

```text
com.company.payment:payment-service:4.2.1
```

---

## 12. Maven Release

Example:

```text
4.2.1
```

A release version should represent immutable artifact content.

---

## 13. Maven Snapshot

Example:

```text
4.2.1-SNAPSHOT
```

Snapshots represent ongoing development.

They should not be treated as immutable production releases.

---

## 14. Snapshot vs Release

```text
SNAPSHOT
→ development

4.2.1
→ release
```

---

## 15. Why Avoid SNAPSHOT in Production?

Because snapshot content can change.

This can cause:

```text
non-reproducible builds
unexpected behavior
difficult rollback
audit problems
```

---

## 16. Maven Versioning Flow

```text
Developer
 ↓
4.2.1-SNAPSHOT
 ↓
CI validation
 ↓
Release preparation
 ↓
4.2.1
 ↓
Artifactory
```

---

# PART IV — NPM VERSIONING

## 17. NPM Version

Example:

```text
4.2.1
```

Package metadata can define:

```json
{
  "name": "@company/payment-client",
  "version": "4.2.1"
}
```

---

## 18. NPM Semantic Versioning

NPM commonly uses SemVer concepts.

Examples:

```text
4.2.1
4.3.0
5.0.0
```

---

## 19. NPM Pre-Releases

Examples:

```text
4.3.0-alpha.1
4.3.0-beta.1
4.3.0-rc.1
```

Use controlled release policies.

---

## 20. NPM Version Mutation

Avoid republishing different content under the same production version.

Bad:

```text
4.2.1
→ content A

later

4.2.1
→ content B
```

Prefer:

```text
4.2.1
→ immutable

4.2.2
→ new content
```

---

# PART V — PYPI VERSIONING

## 21. Python Package Version

Example:

```text
4.2.1
```

A Python distribution might be:

```text
payment_client-4.2.1-py3-none-any.whl
```

---

## 22. Python Pre-Releases

Examples:

```text
4.3.0a1
4.3.0b1
4.3.0rc1
```

---

## 23. Python Development Releases

Example:

```text
4.3.0.dev1
```

Use development versions only where appropriate.

---

## 24. PyPI Release Immutability

Once a production version is approved:

```text
4.2.1
```

do not replace it with unrelated content.

---

# PART VI — DOCKER VERSIONING

## 25. Docker Tag

Example:

```text
payment-service:4.2.1
```

Tags are human-readable references.

---

## 26. Tag Mutability

A tag can potentially move:

```text
4.2.1
→ digest A

later

4.2.1
→ digest B
```

Therefore tags should be protected by repository immutability policies where supported.

---

## 27. Docker Digest

Example:

```text
payment-service@sha256:abcdef...
```

A digest identifies image content.

---

## 28. Tag vs Digest

```text
Tag
→ human-friendly release reference

Digest
→ content identity
```

For high-assurance production deployment, use a verified immutable image digest where practical.

---

## 29. Avoid latest

Avoid:

```text
payment-service:latest
```

as the only production identity.

Prefer:

```text
payment-service:4.2.1
```

and/or:

```text
payment-service@sha256:...
```

---

# PART VII — HELM VERSIONING

## 30. Helm Chart Version

Example:

```yaml
version: 1.4.0
```

This identifies the chart package.

---

## 31. Helm appVersion

Example:

```yaml
appVersion: "4.2.1"
```

This identifies the application version represented by the chart.

---

## 32. Chart Version vs Application Version

Example:

```text
Helm Chart:
1.4.0

Application:
4.2.1
```

The chart can change without changing the application image.

---

## 33. Chart-Only Change

Example:

```text
Chart:
1.4.0
→
1.4.1

Application:
4.2.1
```

Possible reasons:

```text
probe changes
resource changes
securityContext
Ingress
HPA
RBAC
deployment configuration
```

---

## 34. Application-Only Change

Example:

```text
Chart:
1.4.0

Application:
4.2.1
→
4.2.2
```

The chart may remain unchanged if the values/configuration support the new image.

---

# PART VIII — BUILD NUMBERS

## 35. Build Number

CI systems often assign build numbers.

Example:

```text
Build #721
```

---

## 36. Build Number as Traceability

A build number can map:

```text
Git commit
 ↓
CI build 721
 ↓
artifact 4.2.1
```

---

## 37. Should Build Number Be the Version?

Not always.

A common strategy is:

```text
Business release version:
4.2.1

CI metadata:
build 721
```

This separates product versioning from pipeline execution numbering.

---

## 38. Example

```text
Version:
4.2.1

Build:
721

Commit:
abc1234
```

---

# PART IX — GIT COMMIT TRACEABILITY

## 39. Git Commit

Example:

```text
abc123456789
```

A commit identifies source state.

---

## 40. Version + Commit

A Docker tag might be:

```text
4.2.1-abc1234
```

This provides convenient source traceability.

---

## 41. Git SHA Labels

Images can also include metadata labels such as:

```text
org.opencontainers.image.revision
```

where supported by the build process.

---

## 42. Source-to-Artifact Chain

Strong traceability:

```text
Git commit
 ↓
CI pipeline
 ↓
Build number
 ↓
Artifact version
 ↓
Checksum/digest
 ↓
Deployment
```

---

# PART X — SNAPSHOT AND RELEASE STRATEGY

## 43. Development Artifacts

Examples:

```text
4.3.0-SNAPSHOT
4.3.0-dev
4.3.0-alpha.1
```

Use clear repository and lifecycle policies.

---

## 44. Release Artifacts

Examples:

```text
4.2.1
```

Production releases should be immutable.

---

## 45. Development Repository

Example:

```text
maven-snapshot-local
```

or equivalent organizational repository.

---

## 46. Release Repository

Example:

```text
maven-release-local
```

---

## 47. Separation

```text
Development
 ↓
snapshot repository

Production
 ↓
release repository
```

This makes lifecycle boundaries easier to enforce.

---

# PART XI — VERSION GENERATION

## 48. Manual Versioning

Developer sets:

```text
4.2.1
```

Simple but prone to:

```text
human error
duplicate versions
incorrect increments
```

---

## 49. CI-Generated Version

CI can calculate:

```text
version
```

from:

```text
Git tag
release metadata
branch
build configuration
```

---

## 50. Git Tag as Release Trigger

Example:

```text
Git tag:
v4.2.1
```

Flow:

```text
Git tag
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Publish 4.2.1
```

---

## 51. Release Branch

Example:

```text
release/4.2
```

The exact branching strategy depends on the team's development model.

---

# PART XII — VERSION COLLISION

## 52. What Is a Version Collision?

A version collision occurs when multiple builds attempt to publish the same release identity.

Example:

```text
Build A → 4.2.1
Build B → 4.2.1
```

---

## 53. Why Collision Is Dangerous

If overwrite is allowed:

```text
4.2.1
→ artifact A

later

4.2.1
→ artifact B
```

Production traceability is broken.

---

## 54. Prevent Collisions

Use:

```text
immutable release repositories
unique version generation
CI release locks
Git tags
repository deployment policies
```

---

# PART XIII — IMMUTABILITY

## 55. What Is Artifact Immutability?

An immutable artifact cannot be silently replaced after release.

---

## 56. Why Immutability Matters

It guarantees:

```text
version X
→ same content
```

over time.

---

## 57. Benefits

```text
reproducible deployments
safe rollback
auditability
incident investigation
dependency consistency
```

---

## 58. Mutable Development Artifacts

Development artifacts may have different policies.

For example:

```text
SNAPSHOT
```

may be updated.

But:

```text
4.2.1
```

should normally remain immutable.

---

# PART XIV — ARTIFACT PROMOTION

## 59. Build Once, Promote Many

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
 ↓
Stage
 ↓
Production
```

---

## 60. Why Not Rebuild?

If each environment rebuilds:

```text
DEV build
STAGE build
PROD build
```

the artifacts may differ.

Instead:

```text
One artifact
 ↓
multiple environments
```

---

## 61. Promotion Identity

The promoted artifact should retain:

```text
same version
same checksum/digest
same content
```

---

## 62. Example

```text
payment-service:4.2.1

DEV
 ↓
STAGE
 ↓
PROD
```

The content remains the same.

---

# PART XV — ARTIFACTORY BUILD INFO

## 63. Build Info

JFrog Build Info can associate build metadata with artifacts and dependencies.

Conceptually:

```text
CI Build
 ↓
Build Info
 ├── source
 ├── dependencies
 ├── artifacts
 └── environment metadata
```

---

## 64. Why Build Info Matters

It improves:

```text
traceability
release visibility
dependency analysis
audit
promotion
incident investigation
```

---

## 65. Build-to-Artifact Relationship

Example:

```text
Build:
payment-service #721

Artifacts:
payment-service-4.2.1.jar
payment-service:4.2.1
payment-service-1.4.0.tgz
```

The exact artifact set depends on the pipeline.

---

# PART XVI — JENKINS VERSIONING

## 66. Jenkins Release Flow

```text
Git Tag
 ↓
Jenkins
 ↓
Version
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Publish
 ↓
Artifactory
```

---

## 67. Jenkins Environment

The version can be passed through:

```text
environment variable
build parameter
Git tag
release metadata
```

---

## 68. Maven Example

Conceptually:

```bash
mvn versions:set -DnewVersion=4.2.1
mvn clean deploy
```

Use the organization's approved release tooling and avoid mutating source files unnecessarily.

---

## 69. Docker Example

```bash
docker build \
  -t artifactory.company.com/docker-local/payment-service:4.2.1 \
  .

docker push \
  artifactory.company.com/docker-local/payment-service:4.2.1
```

---

# PART XVII — GITHUB ACTIONS VERSIONING

## 70. Git Tag

A release can be triggered from:

```text
v4.2.1
```

---

## 71. GitHub Actions Flow

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

## 72. GitHub SHA

For development builds:

```text
payment-service:sha-abc1234
```

For release:

```text
payment-service:4.2.1
```

This separates development identity from release identity.

---

# PART XVIII — GITLAB CI VERSIONING

## 73. GitLab Pipeline

```text
Tag
 ↓
Pipeline
 ↓
VERSION
 ↓
Build
 ↓
Publish
```

---

## 74. GitLab Commit SHA

Development image:

```text
payment-service:$CI_COMMIT_SHA
```

Release:

```text
payment-service:4.2.1
```

---

# PART XIX — KUBERNETES VERSIONING

## 75. Kubernetes Deployment

Example:

```yaml
image:
  repository: artifactory.company.com/docker-local/payment-service
  tag: "4.2.1"
```

---

## 76. Digest Deployment

Higher-assurance pattern:

```yaml
image: artifactory.company.com/docker-local/payment-service@sha256:...
```

This pins the deployed content.

---

## 77. Kubernetes Rollout

Flow:

```text
Chart version
+
Image version/digest
+
Values
 ↓
Kubernetes Deployment
```

---

## 78. Kubernetes Rollback

If:

```text
4.2.2
```

fails:

```text
rollback
 ↓
4.2.1
```

The old immutable artifact must still be available.

---

# PART XX — VERSIONING SECURITY

## 79. Supply Chain

Versioning is a security control because attackers may attempt:

```text
artifact replacement
version confusion
dependency substitution
tag mutation
```

---

## 80. Immutable Release

Protect:

```text
4.2.1
```

from overwrite.

---

## 81. Digest Verification

For container images:

```text
tag
+
digest
```

can strengthen deployment verification.

---

## 82. Dependency Pinning

Production applications should use a controlled dependency resolution strategy.

Examples:

```text
exact versions
constraints
lockfiles
approved dependency ranges
```

---

## 83. Dependency Confusion

An attacker may publish a public package with a name similar to an internal package.

Controls include:

```text
internal naming
approved repositories
virtual repositories
dependency scanning
version governance
```

---

# PART XXI — VERSIONING TROUBLESHOOTING

## 84. Duplicate Version

Symptom:

```text
Version already exists
```

Check:

```text
release process
Git tag
previous publication
repository immutability
CI concurrency
```

---

## 85. Wrong Artifact Version

Check:

```text
Git tag
CI variables
build configuration
package metadata
Docker tag
Helm values
```

---

## 86. Docker Tag Points to Unexpected Image

Check:

```text
tag mutation
push history
repository audit
digest
CI pipeline
```

---

## 87. Helm Chart Version Mismatch

Check:

```text
Chart.yaml
packaged chart
Artifactory version
deployment command
```

---

## 88. Maven Version Mismatch

Check:

```text
pom.xml
effective POM
CI version parameter
parent version
dependency version
```

---

## 89. NPM Version Mismatch

Check:

```text
package.json
published package metadata
lockfile
CI release version
```

---

## 90. Python Version Mismatch

Check:

```text
pyproject.toml
build metadata
wheel filename
sdist
CI version
```

---

# PART XXII — RELEASE MANAGEMENT

## 91. Release Candidate

Example:

```text
4.3.0-rc.1
```

Use this for controlled validation before final release where appropriate.

---

## 92. Final Release

Example:

```text
4.3.0
```

Once published:

```text
immutable
```

---

## 93. Hotfix

Suppose production runs:

```text
4.2.1
```

A bug is discovered.

Release:

```text
4.2.2
```

Do not overwrite:

```text
4.2.1
```

---

## 94. Rollback

If:

```text
4.2.2
```

fails:

```text
rollback to 4.2.1
```

---

## 95. Release Notes

Each production version should have traceable information such as:

```text
version
changes
Git commit
build
dependencies
security status
deployment status
```

---

# PART XXIII — RETENTION

## 96. Version Retention

Keep enough versions for:

```text
rollback
support
audit
compliance
incident investigation
```

---

## 97. Development Retention

Development versions can often have shorter retention.

Examples:

```text
feature builds
temporary snapshots
nightly builds
```

---

## 98. Production Retention

Production releases should remain available according to:

```text
support policy
rollback requirements
compliance
business requirements
```

---

## 99. Cleanup Rule

Never blindly delete:

```text
latest production version
rollback target
currently deployed artifact
```

---

# PART XXIV — PRODUCTION ARCHITECTURE

## 100. Versioning Architecture

```text
Git
 ↓
Release Tag
 ↓
CI
 ↓
Version Generation
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Publish Immutable Artifact
 ↓
Build Info
 ↓
Promotion
 ↓
Deployment
```

---

## 101. Multi-Artifact Release

A single application release may create:

```text
Maven JAR
Docker Image
Helm Chart
```

Example:

```text
Application:
4.2.1

Docker:
4.2.1

Helm:
1.4.0
appVersion: 4.2.1

Maven:
4.2.1
```

---

## 102. Traceability Matrix

```text
Git:
abc1234

Build:
721

Maven:
payment-service-4.2.1.jar

Docker:
payment-service:4.2.1
digest sha256:...

Helm:
payment-service-1.4.0.tgz

Deployment:
production
```

---

## 103. Enterprise Flow

```text
Developer
   |
   v
Git Tag v4.2.1
   |
   v
CI Pipeline
   |
   +--> Maven 4.2.1
   |
   +--> Docker 4.2.1
   |
   +--> Helm 1.4.0
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
   v
Kubernetes
```

---

# PART XXV — REAL-WORLD SCENARIOS

## 104. Scenario — Same Version Published Twice

Problem:

```text
4.2.1
```

was published twice with different content.

Response:

```text
Stop release
 ↓
Identify both artifacts
 ↓
Audit publisher
 ↓
Determine which was deployed
 ↓
Enforce immutability
 ↓
Create corrected version
```

---

## 105. Scenario — Production Uses latest

Problem:

```text
latest
```

does not identify exact content.

Response:

```text
Identify current digest
 ↓
Create immutable release
 ↓
Update deployment
 ↓
Use version/digest
 ↓
Document migration
```

---

## 106. Scenario — Rollback Artifact Deleted

Problem:

```text
4.2.1
```

is needed for rollback but no longer exists.

Prevention:

```text
production retention
rollback policy
artifact protection
```

Response:

```text
Recover from backup if possible
or
rebuild from a reproducible source only when equivalence can be
demonstrated
```

Prefer preserving the original artifact.

---

## 107. Scenario — Build Number and Version Disagree

Example:

```text
Build #721
Version 4.2.0
```

but expected:

```text
4.2.1
```

Investigate:

```text
release tag
CI variables
version calculation
branch
previous release
```

---

## 108. Scenario — Helm and Docker Versions Misaligned

Example:

```text
Helm:
1.4.0

Image:
4.1.0
```

Determine whether this is intentional.

The chart may legitimately reference an older or newer application version, but production release policy should make the relationship explicit.

---

## 109. Scenario — Vulnerable Version in Production

Flow:

```text
Identify deployed version
 ↓
Identify vulnerability
 ↓
Find fixed version
 ↓
Build
 ↓
Scan
 ↓
Publish
 ↓
Promote
 ↓
Deploy
 ↓
Verify
```

---

# PART XXVI — INTERVIEW PREPARATION

## 110. Why Is Artifact Versioning Important?

Answer:

```text
It provides a stable identity for software artifacts and enables
traceability, reproducibility, rollback, dependency management and
auditability.
```

---

## 111. What Is Semantic Versioning?

Answer:

```text
Semantic Versioning commonly uses MAJOR.MINOR.PATCH, where major
indicates incompatible changes, minor represents backward-compatible
features and patch represents backward-compatible fixes.
```

---

## 112. Why Should Production Artifacts Be Immutable?

Answer:

```text
If the same version can contain different content, deployments become
non-reproducible and rollback and auditing become unreliable.
Immutability guarantees that a released version continues to identify
the same artifact content.
```

---

## 113. Tag vs Digest?

Answer:

```text
A tag is a human-friendly reference and may be mutable. A digest
identifies container image content. For high-assurance production
deployments I prefer immutable release tags and verified digests.
```

---

## 114. Snapshot vs Release?

Answer:

```text
A snapshot represents ongoing development and may change. A release
version represents an approved immutable artifact intended for stable
consumption.
```

---

## 115. Why Build Once and Promote?

Answer:

```text
Building once ensures the exact artifact tested is the artifact
promoted to staging and production. Rebuilding per environment can
produce different outputs.
```

---

## 116. How Do You Handle Hotfixes?

Answer:

```text
I create a new patch version rather than replacing the existing
production version. For example, 4.2.1 becomes 4.2.2.
```

---

## 117. How Do You Implement Versioning in CI/CD?

Answer:

```text
I derive the release version from a controlled source such as a Git
release tag, validate uniqueness, build the artifact, run tests and
security scans, publish it immutably and associate it with build
metadata.
```

---

## 118. How Do You Trace a Production Image Back to Git?

Answer:

```text
I use the image digest/tag, CI build number, Git commit metadata and
build information. The deployment record maps the production image
to the exact build and source revision.
```

---

## 119. What Happens if an Artifact Version Already Exists?

Answer:

```text
I do not overwrite the release. I investigate the collision, verify
the release source and create a new unique version if a corrected
artifact is required.
```

---

## 120. How Do You Design Versioning for Multiple Artifact Types?

Answer:

```text
I define separate version semantics where appropriate but maintain a
common release identity and traceability. For example, the application
can be 4.2.1, the Docker image can be 4.2.1 and the Helm chart can be
1.4.0 with appVersion 4.2.1.
```

---

# PART XXVII — PRODUCTION CHECKLIST

## 121. Versioning

```text
[ ] versioning policy
[ ] semantic versioning where appropriate
[ ] release version
[ ] pre-release policy
[ ] snapshot policy
[ ] uniqueness
```

---

## 122. Immutability

```text
[ ] release artifacts immutable
[ ] Docker tags protected
[ ] digest tracking
[ ] no production latest
[ ] no release overwrite
```

---

## 123. Traceability

```text
[ ] Git commit
[ ] Git tag
[ ] CI build
[ ] artifact version
[ ] checksum/digest
[ ] Build Info
[ ] deployment record
```

---

## 124. Promotion

```text
[ ] build once
[ ] test
[ ] scan
[ ] publish
[ ] promote
[ ] production
[ ] rollback
```

---

## 125. Retention

```text
[ ] development retention
[ ] production retention
[ ] rollback retention
[ ] cleanup protection
[ ] backup
```

---

# PART XXVIII — GOLDEN RULES

## 126. Rules

```text
1. Every production artifact needs a stable identity.

2. Do not overwrite immutable release versions.

3. Use semantic versioning where it fits the artifact.

4. Keep snapshot/development artifacts separate from production
   releases.

5. Do not use latest as the only production image identity.

6. Track Docker digests for production images.

7. Separate chart version from application version.

8. Keep build numbers as traceability metadata rather than replacing
   meaningful product versions unless that is the defined policy.

9. Connect Git commit, CI build, artifact version and deployment.

10. Validate version uniqueness before publication.

11. Build once and promote the same artifact.

12. Keep rollback versions available.

13. Do not blindly delete production artifacts.

14. Use new patch versions for hotfixes.

15. Protect release repositories from accidental overwrite.

16. Use controlled dependency versions for reproducible builds.

17. Record Build Info and provenance where supported.

18. Treat versioning as both a release-management and supply-chain
    security control.

19. Make multi-artifact version relationships explicit.

20. Document versioning policy for Maven, NPM, PyPI, Docker and Helm.

21. Audit unexpected version mutations.

22. Rebuild only when necessary and understand that a rebuild may not
    produce byte-for-byte identical content.

23. Prefer preserving and promoting the original artifact over
    reconstructing it.

24. Design retention around rollback and support requirements.

25. Validate exact versioning, immutability, promotion and repository
    behavior against the deployed JFrog/Artifactory and package-tool
    versions before production rollout.
```

---

# END OF 12-Artifact-Versioning.md
