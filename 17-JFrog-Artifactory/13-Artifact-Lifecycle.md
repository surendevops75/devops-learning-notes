# Artifact-Lifecycle

## 1. Purpose

This file covers the complete software artifact lifecycle in JFrog Artifactory, from creation through consumption, promotion, retention and final cleanup.

It covers:

- Artifact lifecycle fundamentals
- Artifact states
- Build and publication
- Metadata
- Validation
- Security scanning
- Promotion
- Release
- Consumption
- Production deployment
- Rollback
- Retention
- Cleanup
- Archival
- Deletion
- Repository lifecycle
- Maven
- NPM
- PyPI
- Docker/OCI
- Helm
- Build Info
- Jenkins
- GitHub Actions
- GitLab CI
- Kubernetes
- GitOps
- Production governance
- Compliance
- Disaster recovery
- Troubleshooting
- Real-world scenarios
- Interview preparation

---

# PART I — ARTIFACT LIFECYCLE FUNDAMENTALS

## 2. What Is an Artifact Lifecycle?

An artifact lifecycle describes how software moves from creation to retirement.

Typical flow:

```text
Source
 ↓
Build
 ↓
Test
 ↓
Publish
 ↓
Validate
 ↓
Scan
 ↓
Promote
 ↓
Release
 ↓
Deploy
 ↓
Consume
 ↓
Retain
 ↓
Archive/Delete
```

---

## 3. Why Lifecycle Management Matters

Without lifecycle governance:

```text
repositories grow indefinitely
old artifacts accumulate
vulnerable artifacts remain available
storage costs increase
rollback becomes confusing
ownership becomes unclear
```

---

## 4. Lifecycle Objectives

A production lifecycle should provide:

```text
traceability
security
immutability
reproducibility
controlled promotion
availability
retention
cleanup
auditability
```

---

## 5. Artifact States

A useful logical model is:

```text
CREATED
  ↓
VALIDATED
  ↓
SCANNED
  ↓
PUBLISHED
  ↓
PROMOTED
  ↓
RELEASED
  ↓
DEPLOYED
  ↓
RETIRED
  ↓
ARCHIVED / DELETED
```

These are lifecycle concepts; exact Artifactory repository properties and automation can vary by implementation.

---

# PART II — ARTIFACT CREATION

## 6. Source Code

Everything starts with source:

```text
Git repository
```

Example:

```text
payment-service
```

---

## 7. Commit

Example:

```text
Git commit:
abc1234
```

The commit represents the source state used by the build.

---

## 8. Build

CI compiles/packages the application.

Examples:

```text
Maven → JAR
NPM → package
Python → wheel/sdist
Docker → image
Helm → chart
```

---

## 9. Build Metadata

Capture:

```text
Git commit
branch/tag
build number
builder
timestamp
dependencies
artifact version
```

---

# PART III — ARTIFACT PUBLICATION

## 10. Publish

After successful build and validation:

```text
CI
 ↓
Artifactory
 ↓
Repository
```

---

## 11. Repository Selection

Examples:

```text
Maven
→ maven-local

Docker
→ docker-local

Helm
→ helm-local
```

Development and production artifacts should have clearly defined repository boundaries.

---

## 12. Artifact Identity

An artifact should be identifiable by:

```text
name
version
repository
checksum/digest
build information
```

---

## 13. Immutability

Release artifacts should be immutable.

Example:

```text
payment-service:4.2.1
```

should continue to represent the same content.

---

# PART IV — VALIDATION

## 14. Artifact Validation

Before promotion:

```text
package validation
unit tests
integration tests
security scanning
quality checks
policy checks
```

---

## 15. Maven Validation

Example:

```bash
mvn test
mvn verify
```

---

## 16. NPM Validation

Example:

```bash
npm test
npm pack
```

---

## 17. Python Validation

Example:

```bash
pytest
python -m build
```

---

## 18. Docker Validation

Example:

```bash
docker build .
```

Then:

```text
image scan
configuration checks
runtime tests
```

---

## 19. Helm Validation

Example:

```bash
helm lint ./chart
helm template ./chart
```

Then scan the rendered Kubernetes manifests.

---

# PART V — SECURITY SCANNING

## 20. Why Scan Artifacts?

Artifacts may contain:

```text
known vulnerabilities
malicious dependencies
secrets
unsafe configuration
license issues
```

---

## 21. Dependency Scanning

Scan:

```text
direct dependencies
transitive dependencies
base images
package dependencies
Helm dependencies
```

---

## 22. Container Scanning

For Docker images inspect:

```text
OS packages
application libraries
configuration
secrets
known CVEs
```

---

## 23. Helm Security

Scan rendered manifests for:

```text
privileged containers
hostNetwork
hostPath
unsafe capabilities
missing securityContext
public exposure
weak RBAC
```

---

## 24. Scan Before Promotion

Preferred:

```text
Build
 ↓
Scan
 ↓
Publish/Promote
```

or, depending on the platform workflow:

```text
Build
 ↓
Publish to quarantine/non-release location
 ↓
Scan
 ↓
Promote
```

The important requirement is that unapproved artifacts do not enter the trusted production release flow.

---

# PART VI — ARTIFACT QUALITY

## 25. Quality Gates

Examples:

```text
tests pass
code quality threshold
security threshold
license policy
artifact metadata complete
```

---

## 26. Failed Quality Gate

If validation fails:

```text
Do not promote
```

Instead:

```text
fix
 ↓
rebuild
 ↓
validate again
```

---

## 27. Artifact Quarantine

Organizations may use a quarantine concept for artifacts that:

```text
need scanning
need approval
have policy violations
are awaiting investigation
```

The exact implementation can use repositories, properties, lifecycle policies or promotion workflows.

---

# PART VII — ARTIFACT PROMOTION

## 28. What Is Promotion?

Promotion moves an artifact from one trust stage to another.

Example:

```text
development
 ↓
staging
 ↓
production
```

---

## 29. Build Once, Promote Many

Preferred architecture:

```text
Build once
 ↓
Artifact
 ↓
Test
 ↓
Promote
 ↓
DEV
 ↓
STAGE
 ↓
PROD
```

---

## 30. Why Not Rebuild Per Environment?

Rebuilding can produce:

```text
different binaries
different dependencies
different timestamps
different build metadata
different image layers
```

The tested artifact may no longer be the deployed artifact.

---

## 31. Promotion Integrity

The promoted artifact should preserve:

```text
same content
same checksum
same digest
same release identity
```

---

# PART VIII — RELEASE

## 32. Release Artifact

A release artifact is an approved artifact intended for stable consumption.

Example:

```text
payment-service-4.2.1
```

---

## 33. Release Criteria

Possible requirements:

```text
tests passed
security scan passed
approval completed
version valid
metadata complete
artifact immutable
```

---

## 34. Release Approval

Production release may require:

```text
automated gates
manual approval
change ticket
security approval
business approval
```

depending on organizational policy.

---

# PART IX — DEPLOYMENT

## 35. Deployment

Deployment consumes the approved artifact.

Example:

```text
Artifactory
 ↓
Kubernetes
```

---

## 36. Kubernetes Deployment

Example:

```yaml
image:
  repository: artifactory.company.com/docker-local/payment-service
  tag: "4.2.1"
```

---

## 37. Digest-Based Deployment

For stronger content pinning:

```yaml
image: artifactory.company.com/docker-local/payment-service@sha256:...
```

---

## 38. Helm Deployment

Example:

```bash
helm upgrade --install payment \
  company/payment-service \
  --version 1.4.0 \
  -f values-prod.yaml
```

---

# PART X — CONSUMPTION

## 39. Consumers

Consumers may include:

```text
developers
CI systems
Kubernetes
applications
other teams
external approved systems
```

---

## 40. Consumer Access

Consumers should have only:

```text
READ
```

unless they need to publish or administer.

---

## 41. Virtual Repositories

A virtual repository can provide:

```text
one consumer endpoint
```

while aggregating:

```text
local
+
remote
```

repositories.

---

# PART XI — PRODUCTION LIFECYCLE

## 42. Production Flow

```text
Git
 ↓
CI
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
Production
 ↓
Monitor
```

---

## 43. Production Monitoring

Monitor:

```text
deployment health
application health
artifact availability
repository health
image pull failures
release errors
```

---

## 44. Production Rollback

If release:

```text
4.2.2
```

fails:

```text
rollback
 ↓
4.2.1
```

The previous artifact must still be available.

---

# PART XII — RETENTION

## 45. What Is Retention?

Retention defines how long artifacts remain available.

---

## 46. Why Retain Artifacts?

For:

```text
rollback
support
audit
compliance
reproducibility
incident investigation
```

---

## 47. Development Retention

Development artifacts may have shorter retention.

Examples:

```text
feature builds
nightly builds
temporary snapshots
```

---

## 48. Production Retention

Production releases should be retained according to:

```text
rollback requirements
support policy
business requirements
compliance
```

---

## 49. Retention Is Not the Same as Deletion

Retention defines:

```text
how long
```

Deletion is the actual removal operation.

---

# PART XIII — CLEANUP

## 50. Why Cleanup?

Without cleanup:

```text
storage grows
repository performance can suffer
backup size increases
management becomes difficult
```

---

## 51. Safe Cleanup

Before deletion:

```text
Identify artifact
 ↓
Check age
 ↓
Check deployment status
 ↓
Check rollback requirements
 ↓
Check retention policy
 ↓
Delete if eligible
```

---

## 52. Never Blindly Delete

Avoid deleting:

```text
currently deployed version
approved rollback target
compliance-required version
actively consumed dependency
```

---

## 53. Development Cleanup

Possible candidates:

```text
old snapshots
old feature builds
failed test artifacts
temporary packages
```

Only delete according to documented policy.

---

# PART XIV — ARCHIVAL

## 54. What Is Archival?

Archival moves or preserves artifacts for long-term retention outside the active operational set.

---

## 55. Why Archive?

For:

```text
compliance
historical reference
long-term support
audit
legal/business requirements
```

---

## 56. Active vs Archive

```text
Active repository
 ↓
supported releases
```

while:

```text
Archive
 ↓
historical releases
```

---

## 57. Archive Verification

Archived artifacts should remain:

```text
identifiable
retrievable
integrity-verified
documented
```

---

# PART XV — DELETION

## 58. Artifact Deletion

Deletion should be controlled.

Possible controls:

```text
role restriction
approval
retention policy
audit
automation
```

---

## 59. Delete Permission

Do not give deletion permissions broadly.

Typical:

```text
Developer
→ NO

CI
→ NO

Release
→ usually NO

Platform administrator
→ controlled
```

---

## 60. Deletion Audit

Record:

```text
who
what
when
where
why
```

where supported by the platform and operational logging.

---

# PART XVI — ARTIFACT LIFECYCLE BY TYPE

## 61. Maven Lifecycle

```text
SNAPSHOT
 ↓
Validate
 ↓
Release
 ↓
Promote
 ↓
Consume
 ↓
Retain
 ↓
Cleanup
```

---

## 62. NPM Lifecycle

```text
development
 ↓
test
 ↓
publish
 ↓
scan
 ↓
release
 ↓
consume
 ↓
retire
```

---

## 63. PyPI Lifecycle

```text
build
 ↓
validate
 ↓
publish
 ↓
scan
 ↓
release
 ↓
consume
 ↓
retain
```

---

## 64. Docker Lifecycle

```text
Source
 ↓
Build image
 ↓
Scan
 ↓
Push
 ↓
Promote
 ↓
Deploy
 ↓
Retain
```

---

## 65. Helm Lifecycle

```text
Chart
 ↓
Lint
 ↓
Render
 ↓
Scan
 ↓
Package
 ↓
Publish
 ↓
Promote
 ↓
Deploy
 ↓
Retain
```

---

# PART XVII — ARTIFACT PROPERTIES AND METADATA

## 66. Metadata

Useful metadata can include:

```text
team
application
environment
release
commit
build
security status
owner
```

---

## 67. Why Metadata Matters

Metadata helps with:

```text
search
automation
promotion
retention
audit
ownership
```

---

## 68. Example

```text
application=payment
team=payments
release=4.2.1
build=721
commit=abc1234
```

---

## 69. Metadata Governance

Do not allow arbitrary uncontrolled metadata.

Define:

```text
standard fields
allowed values
ownership
automation rules
```

---

# PART XVIII — BUILD INFO AND LIFECYCLE

## 70. Build Info

JFrog Build Info can connect:

```text
source
dependencies
build
artifacts
environment metadata
```

---

## 71. Lifecycle Traceability

Example:

```text
Git commit abc1234
 ↓
Jenkins build 721
 ↓
Docker image 4.2.1
 ↓
Build Info
 ↓
Production
```

---

## 72. Why This Matters During Incidents

If a vulnerability is discovered:

```text
Identify vulnerable artifact
 ↓
Find build
 ↓
Find source commit
 ↓
Find deployed environments
 ↓
Remediate
```

---

# PART XIX — JENKINS LIFECYCLE

## 73. Jenkins Flow

```text
Checkout
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

## 74. Failed Build

If tests fail:

```text
No production promotion
```

---

## 75. Failed Scan

If security policy fails:

```text
Quarantine/block
 ↓
Remediate
 ↓
Rebuild
```

---

## 76. Successful Release

```text
Artifact
 ↓
Build Info
 ↓
Promotion
 ↓
Deployment
```

---

# PART XX — GITHUB ACTIONS LIFECYCLE

## 77. GitHub Flow

```text
Pull Request
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Merge
 ↓
Release Tag
 ↓
Publish
 ↓
Promote
```

---

## 78. Release Tag

Example:

```text
v4.2.1
```

The tag can become the release identity.

---

# PART XXI — GITLAB CI LIFECYCLE

## 79. GitLab Flow

```text
Commit
 ↓
Pipeline
 ↓
Test
 ↓
Scan
 ↓
Package
 ↓
Publish
 ↓
Release
```

---

## 80. Environment Promotion

```text
Dev
 ↓
Stage
 ↓
Prod
```

Use protected environments and approvals where required.

---

# PART XXII — GITOPS LIFECYCLE

## 81. GitOps

Git stores desired deployment state.

```text
Git
 ↓
Argo CD
 ↓
Artifactory
 ↓
Kubernetes
```

---

## 82. Artifact Repository Role

Artifactory provides:

```text
immutable charts
images
packages
release artifacts
```

---

## 83. GitOps Rollback

Rollback can be performed by:

```text
reverting Git desired state
```

or:

```text
selecting a known-good release
```

depending on the GitOps strategy.

---

# PART XXIII — PRODUCTION LIFECYCLE GOVERNANCE

## 84. Ownership

Every artifact should have an owner.

Example:

```text
Application:
payment-service

Owner:
payments-platform
```

---

## 85. Support Status

Track whether an artifact is:

```text
active
supported
deprecated
retired
```

---

## 86. Deprecation

Before retirement:

```text
announce
 ↓
identify consumers
 ↓
provide replacement
 ↓
migrate consumers
 ↓
retire
```

---

## 87. Retired Artifact

A retired artifact should generally no longer be used for new deployments.

It may still be retained for:

```text
rollback
audit
legacy support
```

---

# PART XXIV — SECURITY LIFECYCLE

## 88. Vulnerability Discovered

Example:

```text
payment-service 4.2.1
```

contains a critical vulnerability.

Flow:

```text
Detect
 ↓
Assess
 ↓
Identify affected deployments
 ↓
Build fixed version
 ↓
Scan
 ↓
Publish
 ↓
Promote
 ↓
Deploy
 ↓
Retire vulnerable version
```

---

## 89. Vulnerable Dependency

If a dependency is vulnerable:

```text
Update dependency
 ↓
Rebuild
 ↓
Test
 ↓
Scan
 ↓
Publish new version
```

Do not modify the existing production artifact.

---

## 90. Malicious Artifact

If compromise is suspected:

```text
Quarantine
 ↓
Stop promotion
 ↓
Identify publisher
 ↓
Revoke credentials
 ↓
Identify consumers
 ↓
Assess production
 ↓
Publish trusted replacement
```

---

# PART XXV — LIFECYCLE TROUBLESHOOTING

## 91. Artifact Cannot Be Promoted

Check:

```text
permission
artifact exists
version
repository
build metadata
promotion policy
security status
```

---

## 92. Artifact Was Deleted Too Early

Investigate:

```text
retention rule
cleanup job
repository policy
manual deletion
automation
```

---

## 93. Production Artifact Missing

Check:

```text
cleanup
retention
repository
replication
backup
promotion
```

---

## 94. Old Artifact Cannot Be Identified

Check:

```text
version
checksum
Build Info
Git commit
CI build
deployment record
```

---

## 95. Promotion Succeeded but Deployment Failed

Separate:

```text
artifact lifecycle
```

from:

```text
deployment lifecycle
```

Check:

```text
Kubernetes
image pull
configuration
permissions
health checks
```

---

## 96. Cleanup Removed Rollback Artifact

Root cause:

```text
retention policy did not understand production dependency
```

Fix:

```text
protect active/rollback versions
improve retention rules
test cleanup automation
```

---

# PART XXVI — PRODUCTION ARCHITECTURE

## 97. Complete Lifecycle

```text
                    Git
                     |
                     v
                    CI
                     |
              +------+------+
              |             |
            Build          Test
              |             |
              +------+------+
                     |
                     v
                   Scan
                     |
                     v
                Artifactory
                     |
                     v
                 Promotion
                     |
          +----------+----------+
          |                     |
         STAGE                 PROD
          |                     |
          +----------+----------+
                     |
                     v
                 Kubernetes
                     |
                     v
                 Monitoring
                     |
                     v
              Retention/Cleanup
```

---

## 98. Enterprise Repository Model

```text
                    Artifactory
                         |
       +-----------------+-----------------+
       |                 |                 |
     Local             Remote            Virtual
       |                 |                 |
 Internal artifacts   External deps     Consumer endpoint
```

---

## 99. Trust Stages

Conceptually:

```text
Untrusted
   ↓
Validated
   ↓
Scanned
   ↓
Approved
   ↓
Released
   ↓
Production
   ↓
Retired
```

---

# PART XXVII — DISASTER RECOVERY

## 100. Lifecycle and DR

Production artifacts must remain recoverable.

Plan for:

```text
repository failure
storage failure
region failure
accidental deletion
corruption
```

---

## 101. Backup

Back up according to the supported Artifactory architecture:

```text
artifact storage
metadata
configuration
database
```

---

## 102. Restore

Test:

```text
artifact retrieval
version lookup
permissions
promotion
CI access
GitOps access
```

---

# PART XXVIII — REAL-WORLD SCENARIOS

## 103. Scenario — Artifact Promotion Blocked

Response:

```text
Check repository
 ↓
Check permissions
 ↓
Check artifact metadata
 ↓
Check security status
 ↓
Check promotion rules
```

---

## 104. Scenario — Production Artifact Accidentally Deleted

Response:

```text
Stop further deletion
 ↓
Identify artifact
 ↓
Check audit
 ↓
Restore from backup if needed
 ↓
Protect production versions
 ↓
Fix cleanup policy
```

---

## 105. Scenario — Vulnerable Image Running in Production

Response:

```text
Identify digest
 ↓
Find source/build
 ↓
Build patched version
 ↓
Scan
 ↓
Publish
 ↓
Promote
 ↓
Deploy
 ↓
Verify old version is no longer running
```

---

## 106. Scenario — Hundreds of Old Docker Images

Response:

```text
Classify versions
 ↓
Identify active deployments
 ↓
Protect supported releases
 ↓
Define retention
 ↓
Delete eligible versions
 ↓
Monitor storage
```

---

## 107. Scenario — Old Maven SNAPSHOTs Consume Storage

Response:

```text
Identify snapshot age
 ↓
Check active consumers
 ↓
Define snapshot retention
 ↓
Clean old snapshots
 ↓
Monitor
```

---

## 108. Scenario — Production Rollback Impossible

Root causes may include:

```text
artifact deleted
retention too short
repository unavailable
incorrect version tracking
```

Fix:

```text
retain rollback targets
track deployed versions
test rollback
```

---

# PART XXIX — INTERVIEW PREPARATION

## 109. What Is Artifact Lifecycle Management?

Answer:

```text
It is the controlled management of an artifact from creation and
validation through publication, promotion, release, deployment,
consumption, retention and retirement.
```

---

## 110. Why Is Artifact Immutability Important?

Answer:

```text
It ensures that a version always represents the same content. This
provides reproducibility, reliable rollback and auditability.
```

---

## 111. What Is Build Once and Promote Many?

Answer:

```text
It means creating one tested artifact and promoting that exact
artifact through environments instead of rebuilding separately for
each environment.
```

---

## 112. How Do You Handle Vulnerable Artifacts?

Answer:

```text
I identify affected versions and deployments, build a patched
version, scan it, publish and promote the trusted artifact, deploy it
and retire or block the vulnerable version according to policy.
```

---

## 113. How Do You Design Artifact Retention?

Answer:

```text
I retain production and supported rollback versions according to
business, support and compliance requirements, while applying shorter
retention to development artifacts. Cleanup must never remove active
or required rollback artifacts.
```

---

## 114. How Do You Prevent Accidental Deletion?

Answer:

```text
I restrict DELETE permissions, protect production repositories,
implement retention policies, audit deletion events and test cleanup
automation.
```

---

## 115. How Do You Track an Artifact Back to Source?

Answer:

```text
I use artifact version, checksum or digest, Build Info, CI build
number, Git commit/tag and deployment records.
```

---

## 116. What Is the Difference Between Artifact and Deployment Lifecycle?

Answer:

```text
The artifact lifecycle manages the software package itself, while
the deployment lifecycle manages how that package is deployed,
updated, monitored and rolled back in an environment.
```

---

## 117. How Do You Manage Lifecycle Across Maven, Docker and Helm?

Answer:

```text
I define lifecycle policies per artifact type but maintain common
release traceability. Maven packages, Docker images and Helm charts
can have independent version formats while remaining linked to the
same application release and CI build.
```

---

# PART XXX — PRODUCTION CHECKLIST

## 118. Creation

```text
[ ] source commit
[ ] version
[ ] build
[ ] metadata
```

---

## 119. Validation

```text
[ ] tests
[ ] quality
[ ] security scan
[ ] policy checks
```

---

## 120. Publication

```text
[ ] correct repository
[ ] immutable release
[ ] Build Info
[ ] ownership
```

---

## 121. Promotion

```text
[ ] approval
[ ] same artifact
[ ] checksum/digest
[ ] audit
```

---

## 122. Deployment

```text
[ ] version tracked
[ ] digest tracked
[ ] health verification
[ ] rollback target
```

---

## 123. Retention

```text
[ ] production retention
[ ] development retention
[ ] rollback protection
[ ] cleanup policy
```

---

## 124. Retirement

```text
[ ] consumers identified
[ ] replacement available
[ ] deprecation communicated
[ ] archive/delete approved
```

---

# PART XXXI — GOLDEN RULES

## 125. Rules

```text
1. Treat artifacts as controlled production assets.

2. Track every production artifact back to source and build.

3. Keep release artifacts immutable.

4. Build once and promote the same artifact.

5. Validate and scan before trusted production promotion.

6. Separate development, release and production lifecycle policies.

7. Never blindly delete artifacts.

8. Protect active and rollback versions.

9. Keep enough retention for operational recovery.

10. Use metadata and Build Info for traceability.

11. Track Docker digests for production images.

12. Treat vulnerable artifacts as lifecycle events requiring
    assessment and remediation.

13. Restrict DELETE permissions.

14. Automate cleanup only after testing its safety.

15. Define ownership for repositories and artifacts.

16. Define artifact support and retirement states.

17. Archive artifacts when long-term retention is required.

18. Separate artifact lifecycle from deployment lifecycle.

19. Test rollback using real retained artifacts.

20. Test backup and restore.

21. Audit promotion, deletion and administrative lifecycle changes.

22. Use GitOps and controlled promotion for repeatable deployment.

23. Do not rebuild an artifact unnecessarily after it has been tested
    and approved.

24. Make lifecycle rules explicit for Maven, NPM, PyPI, Docker and
    Helm.

25. Validate exact Artifactory lifecycle, retention, promotion,
    properties and cleanup behavior against the deployed JFrog
    version before production rollout.
```

---