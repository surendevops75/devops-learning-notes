# Artifact-Publishing

## 1. Purpose

Artifact publishing is the controlled process of taking a validated build
output and making it available to consumers, deployment systems, and
promotion workflows.

Production model:

```text
Git
 |
v
CI
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
Validate Artifact
 |
v
Publish
 |
v
Artifact Repository
 |
v
Promote
 |
+--> DEV
+--> STAGE
+--> PROD
```

The central principle is:

```text
Build once -> publish once -> promote the same artifact
```

An artifact can be:

```text
Maven JAR/WAR
npm package
Python wheel/sdist
Docker image
Helm chart
ZIP/TAR
binary
library
CLI package
```

This file covers repository design, package coordinates, authentication,
permissions, metadata, checksums, immutability, Maven, npm, Python,
Docker, Helm, Artifactory, CI publishing, Jenkins, GitHub Actions,
GitLab, Kubernetes consumption, promotion, snapshots, release
repositories, retention, provenance, SBOM, signing, security, failure
handling, troubleshooting, production architecture, runbooks,
interview preparation, senior scenarios, and golden rules.

---

# PART I — ARTIFACT FUNDAMENTALS

## 2. What Is an Artifact?

An artifact is a generated software output that can be stored,
distributed, tested, deployed, or consumed.

Examples:

```text
payment-api-1.4.2.jar
payment-client-1.4.2.tgz
payment_client-1.4.2-py3-none-any.whl
payment-api:1.4.2
payments-1.4.2.tgz
```

---

## 3. Artifact vs Source Code

Source:

```text
Git
```

Artifact:

```text
build output
```

Relationship:

```text
Source commit
    |
    v
Build
    |
    v
Artifact
```

An artifact should be traceable to its source.

---

## 4. Why Publish Artifacts?

Publishing provides:

```text
central distribution
version control
immutability
traceability
promotion
rollback
dependency management
```

---

# PART II — ARTIFACT IDENTITY

## 5. Artifact Coordinates

A package needs a stable identity.

Maven:

```text
groupId:artifactId:version
```

Example:

```text
com.company.payment:payment-api:1.4.2
```

npm:

```text
@company/payment-client@1.4.2
```

Python:

```text
payment-client 1.4.2
```

Container:

```text
registry/company/payment-api:1.4.2
```

---

## 6. Unique Version

A published release version should be unique.

Bad:

```text
1.4.2 -> artifact A
1.4.2 -> artifact B
```

Good:

```text
1.4.2 -> one immutable release
1.4.3 -> next release
```

---

# PART III — IMMUTABILITY

## 7. Immutable Artifact

Once a production release is published:

```text
1.4.2
```

its contents should not silently change.

---

## 8. Why Immutability Matters

It provides:

```text
reproducibility
rollback
auditability
trust
```

---

## 9. Mutable vs Immutable

Bad:

```text
app:latest
```

as the sole production identity.

Better:

```text
app:1.4.2
```

and ideally:

```text
app@sha256:<digest>
```

for deployment identity.

---

# PART IV — REPOSITORY TYPES

## 10. Common Repository Types

```text
local
remote
virtual
```

Local stores internally produced artifacts.

Remote caches upstream repositories.

Virtual provides one logical endpoint across repositories.

---

## 11. Development Repository

Example:

```text
libs-dev
```

Used for development artifacts.

---

## 12. Release Repository

Example:

```text
libs-release
```

Used for stable release artifacts.

---

## 13. Snapshot Repository

Example:

```text
libs-snapshot
```

Used for mutable development versions where the ecosystem supports them.

---

# PART V — ARTIFACTORY

## 14. Artifactory Model

```text
CI
 |
v
Artifactory
 |
+--> Maven
+--> npm
+--> PyPI
+--> Docker
+--> Helm
```

A single enterprise platform can centralize artifact governance across
package ecosystems.

---

## 15. Virtual Repository

Concept:

```text
Developer/CI
     |
     v
virtual-repo
     |
 +---+---+
 |       |
 v       v
local   remote
```

This simplifies client configuration.

---

# PART VI — PUBLISHING LIFECYCLE

## 16. Standard Lifecycle

```text
Source
 |
v
Build
 |
v
Validate
 |
v
Package
 |
v
Publish
 |
v
Scan/verify
 |
v
Promote
 |
v
Consume/deploy
```

Security scanning may occur before and/or after publishing depending on
the architecture.

---

# PART VII — PRE-PUBLISH VALIDATION

## 17. Validate Before Publish

Check:

```text
version
package contents
dependencies
tests
security
metadata
artifact size
repository target
```

---

## 18. Prevent Accidental Publishing

Only release jobs should normally have artifact write permission.

Pull request validation jobs should generally be read-only.

---

# PART VIII — MAVEN PUBLISHING

## 19. Maven Deploy

Typical command:

```bash
./mvnw -B deploy
```

This publishes project artifacts to the configured Maven repository.

---

## 20. Maven Distribution Management

A Maven project can define repository targets through its build
configuration.

Keep repository URLs and credentials controlled through secure CI
configuration.

---

## 21. Maven Settings

Credentials commonly belong in Maven settings rather than source code.

Concept:

```text
settings.xml
 |
v
server credentials
 |
v
repository
```

Never commit plaintext production credentials.

---

# PART IX — MAVEN POM

## 22. Coordinates

Example:

```xml
<groupId>com.company.payment</groupId>
<artifactId>payment-api</artifactId>
<version>1.4.2</version>
```

These coordinates uniquely identify the package within the repository.

---

# PART X — MAVEN SNAPSHOT

## 23. Snapshot Publishing

Example:

```text
1.5.0-SNAPSHOT
```

Use for active development when snapshot semantics are appropriate.

Do not use snapshots as immutable production releases.

---

# PART XI — MAVEN RELEASE

## 24. Release

Example:

```text
1.5.0
```

Release artifacts should be immutable.

---

# PART XII — NPM PUBLISHING

## 25. npm

Typical command:

```bash
npm publish
```

For private repositories, configure the approved registry.

---

## 26. npm Package Metadata

Verify:

```text
name
version
files
dependencies
peerDependencies
engines
license
```

---

## 27. npm Pack Inspection

Before publishing:

```bash
npm pack --dry-run
```

This helps inspect which files will be included.

---

# PART XIII — NPM PRIVATE REGISTRY

## 28. Registry Configuration

Concept:

```text
.npmrc
 |
v
private registry
 |
v
authentication
```

Do not commit long-lived publishing tokens.

---

# PART XIV — PYTHON PUBLISHING

## 29. Build Python Package

Typical:

```bash
python -m build
```

Output:

```text
dist/
 |
+--> package-1.4.2-py3-none-any.whl
+--> package-1.4.2.tar.gz
```

---

## 30. Validate Python Package

Useful checks include:

```text
metadata
wheel contents
dependencies
version
README
license
```

---

## 31. Publish

A Python package can be uploaded to an approved package repository
using the organization's publishing mechanism.

Do not expose repository credentials in source.

---

# PART XV — WHEELS VS SDIST

## 32. Wheel

A wheel is a built Python distribution.

Advantages can include:

```text
faster installation
prebuilt native components
less compilation
```

---

## 33. Source Distribution

An sdist contains source/package build material.

Consumers may need to build it locally.

---

# PART XVI — DOCKER PUBLISHING

## 34. Build Image

Example:

```bash
docker build -t payment-api:1.4.2 .
```

---

## 35. Tag Registry

Example:

```bash
docker tag payment-api:1.4.2 registry.company.com/payment-api:1.4.2
```

---

## 36. Push

Example:

```bash
docker push registry.company.com/payment-api:1.4.2
```

Use the organization's approved registry and authentication method.

---

# PART XVII — IMAGE DIGEST

## 37. Digest

A container registry assigns an immutable content digest.

Concept:

```text
tag
 |
v
image
 |
v
digest
```

Production deployment should record the digest.

---

# PART XVIII — HELM PUBLISHING

## 38. Helm

Package a chart:

```bash
helm package ./chart
```

Output:

```text
payment-chart-1.4.2.tgz
```

Publish it to the organization's approved OCI or chart repository.

---

# PART XIX — ARTIFACT METADATA

## 39. Metadata

Useful metadata:

```text
name
version
source commit
branch/tag
build number
builder
timestamp
dependencies
checksum
SBOM
provenance
```

---

# PART XX — BUILD INFO

## 40. Build Information

Build information connects:

```text
build
 |
+--> source
+--> dependencies
+--> environment
+--> artifacts
```

This is valuable for incident response and audit.

---

# PART XXI — CHECKSUMS

## 41. Checksums

A checksum can identify exact content.

Example concept:

```text
artifact
 |
v
SHA-256
 |
v
release record
```

Do not use a filename alone to establish artifact identity.

---

# PART XXII — CONTENT VALIDATION

## 42. Artifact Validation

Before promotion verify:

```text
expected name
expected version
expected checksum
expected dependencies
expected file structure
```

---

# PART XXIII — ARTIFACT SIGNING

## 43. Signing

Signing can establish that an artifact was produced or approved by a
trusted identity.

Potentially sign:

```text
container image
package
release metadata
provenance
```

---

# PART XXIV — SBOM

## 44. SBOM Association

```text
Artifact v1.4.2
 |
v
SBOM
 |
+--> dependency A
+--> dependency B
+--> dependency C
```

Keep the SBOM associated with the exact artifact.

---

# PART XXV — PROVENANCE

## 45. Provenance

Provenance answers:

```text
What source produced this artifact?
Which builder?
Which workflow?
Which dependencies?
```

---

# PART XXVI — CI PUBLISHING

## 46. Generic CI Flow

```text
Pull Request
 |
v
Validation only
 |
v
Merge
 |
v
Release workflow
 |
v
Publish
```

This separates untrusted validation from privileged publishing.

---

# PART XXVII — GITHUB ACTIONS

## 47. Example Structure

```yaml
name: Publish

on:
  push:
    tags:
      - 'v*'

permissions:
  contents: read

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build
        run: ./mvnw -B clean verify

      - name: Publish
        run: ./mvnw -B deploy
```

In a real production workflow, configure the required package repository
credentials or federated identity securely.

---

# PART XXVIII — JENKINS

## 48. Jenkins Pipeline

Concept:

```text
Checkout
 |
Build
 |
Test
 |
Scan
 |
Publish
 |
Promote
```

Store repository credentials in Jenkins credentials management.

---

# PART XXIX — GITLAB CI

## 49. GitLab

A typical pipeline can separate:

```text
build
test
security
publish
deploy
```

Only the appropriate release job should have artifact write access.

---

# PART XXX — PUBLISHING AUTHENTICATION

## 50. Authentication

Possible mechanisms:

```text
username/password
access token
API key
OIDC/federated identity
workload identity
```

Prefer short-lived credentials where supported.

---

# PART XXXI — AUTHORIZATION

## 51. Authorization

Separate:

```text
read
write
delete
admin
```

permissions.

A build job normally needs only the permissions required to publish its
specific package.

---

# PART XXXII — REPOSITORY PERMISSIONS

## 52. Least Privilege

Example:

```text
PR CI -> read
release CI -> publish
developers -> consume
repository admin -> administration
```

---

# PART XXXIII — DELETE PERMISSIONS

## 53. Delete

Production publishing identities should normally not have broad delete
permissions.

Deletion can destroy rollback capability.

---

# PART XXXIV — RELEASE REPOSITORY DESIGN

## 54. Example

```text
company-maven-local
company-npm-local
company-pypi-local
company-docker-local
company-helm-local
```

Naming should be standardized.

---

# PART XXXV — REPOSITORY NAMING

## 55. Naming Policy

A repository naming convention can encode:

```text
technology
lifecycle
environment
ownership
```

Keep names stable because changing them can affect many consumers.

---

# PART XXXVI — VIRTUAL REPOSITORY

## 56. Consumer Endpoint

Instead of configuring multiple endpoints:

```text
Maven
npm
pip
```

consumers can use an approved virtual repository where appropriate.

---

# PART XXXVII — REMOTE CACHE

## 57. Upstream Dependencies

A remote repository can cache public dependencies.

Benefits:

```text
availability
speed
central governance
reduced external traffic
```

---

# PART XXXVIII — DEPENDENCY PROXY

## 58. Enterprise Pattern

```text
CI
 |
v
Artifactory virtual
 |
+--> local company artifacts
+--> cached public artifacts
```

This provides centralized dependency access.

---

# PART XXXIX — PUBLISH VS PROMOTE

## 59. Difference

Publish:

```text
make artifact available in repository
```

Promote:

```text
move/change artifact lifecycle state without rebuilding
```

---

## 60. Preferred Model

```text
Build
 |
v
Publish candidate
 |
v
Validate
 |
v
Promote
```

---

# PART XL — REPOSITORY PROMOTION

## 61. Promotion

Concept:

```text
candidate
   |
   v
validated release
   |
   v
production-consumable
```

Promotion implementation varies by repository platform.

---

# PART XLI — BUILD ONCE

## 62. Build Once

```text
Commit
 |
v
Build
 |
v
Artifact A
 |
+--> DEV
+--> STAGE
+--> PROD
```

Artifact A should remain unchanged.

---

# PART XLII — ENVIRONMENT CONFIGURATION

## 63. Configuration

Avoid rebuilding because:

```text
DEV URL
STAGE URL
PROD URL
```

changed.

Use environment-specific configuration through the approved deployment
mechanism.

---

# PART XLIII — ARTIFACT RETENTION

## 64. Retention

Retention should account for:

```text
rollback
support
compliance
storage cost
```

---

## 65. Active Version Protection

Do not delete artifacts that are still deployed or needed for rollback.

---

# PART XLIV — SNAPSHOT RETENTION

## 66. Snapshots

Snapshots may have shorter retention because they represent development
state.

Define cleanup rules carefully.

---

# PART XLV — RELEASE RETENTION

## 67. Releases

Stable production artifacts should have retention aligned with support
and recovery requirements.

---

# PART XLVI — ARTIFACT STORAGE

## 68. Storage Growth

Monitor:

```text
repository size
artifact count
large files
retention cleanup
Docker layers
```

---

# PART XLVII — LARGE ARTIFACTS

## 69. Large Files

Large artifacts can affect:

```text
upload
download
storage
backup
deployment
```

Optimize package contents without removing required runtime components.

---

# PART XLVIII — PACKAGE CONTENT

## 70. Avoid Unnecessary Files

Do not publish:

```text
.git
local secrets
temporary logs
build caches
developer files
```

---

# PART XLIX — npm FILES

## 71. npm

Use package inclusion/exclusion configuration and inspect:

```bash
npm pack --dry-run
```

before release.

---

# PART L — PYTHON FILES

## 72. Python

Inspect wheel and sdist contents before publication.

Ensure:

```text
license
README
metadata
package modules
```

are correct.

---

# PART LI — MAVEN FILES

## 73. Maven

Validate generated:

```text
JAR
POM
sources
javadocs
checksums
```

according to the project's release requirements.

---

# PART LII — CONTAINER CONTENT

## 74. Container

Check:

```text
base image
runtime files
licenses
application
configuration
user
permissions
```

Do not include secrets.

---

# PART LIII — ARTIFACT SECURITY

## 75. Security Gates

Before release:

```text
SAST
dependency scan
secret scan
container scan
license checks
```

Use the controls required by the organization.

---

# PART LIV — MALICIOUS ARTIFACT

## 76. Incident

If an artifact is suspected compromised:

```text
stop promotion
 |
v
quarantine
 |
v
identify consumers
 |
v
audit source/build
 |
v
revoke credentials if needed
 |
v
rebuild trusted artifact
```

---

# PART LV — PUBLISH FAILURE

## 77. Failure Categories

```text
authentication
authorization
network
repository
duplicate version
metadata
checksum
storage
```

---

# PART LVI — AUTHENTICATION FAILURE

## 78. Diagnose

Check:

```text
credential validity
credential scope
registry URL
secret injection
token expiration
```

Never print secrets into CI logs.

---

# PART LVII — AUTHORIZATION FAILURE

## 79. Diagnose

Check:

```text
repository permissions
package scope
identity
project permissions
repository policy
```

---

# PART LVIII — DUPLICATE VERSION

## 80. Diagnose

If:

```text
1.4.2 already exists
```

do not overwrite it.

Create a new version if the release content changed.

---

# PART LIX — NETWORK FAILURE

## 81. Diagnose

Check:

```text
DNS
TLS
proxy
firewall
registry availability
timeouts
```

---

# PART LX — CHECKSUM FAILURE

## 82. Diagnose

Check:

```text
corrupt local cache
network interruption
repository integrity
artifact generation
```

Rebuild or re-download from trusted sources as appropriate.

---

# PART LXI — REPOSITORY OUTAGE

## 83. Impact

Publishing may fail while existing consumers can continue to use cached
or already available artifacts.

Design recovery procedures around repository availability.

---

# PART LXII — HIGH AVAILABILITY

## 84. Repository HA

For critical enterprise repositories consider:

```text
redundant nodes
database availability
storage redundancy
load balancing
backup
disaster recovery
```

Exact architecture depends on the repository platform.

---

# PART LXIII — DISASTER RECOVERY

## 85. Recovery

Protect:

```text
artifacts
metadata
repository configuration
security configuration
database
storage
```

Test restoration.

---

# PART LXIV — BACKUP

## 86. Backup

Backups should support:

```text
point-in-time recovery
artifact recovery
metadata recovery
configuration recovery
```

---

# PART LXV — DR TEST

## 87. Test

A DR test should verify:

```text
repository restoration
artifact readability
metadata consistency
consumer access
CI publishing
deployment recovery
```

---

# PART LXVI — CONSUMER SIDE

## 88. Consumption

Consumers need:

```text
repository endpoint
credentials
package coordinates
version
```

---

# PART LXVII — KUBERNETES CONSUMPTION

## 89. Image

Kubernetes consumes a container image.

Production example:

```yaml
image: registry.company.com/payment-api@sha256:...
```

This provides immutable image identity.

---

# PART LXVIII — IMAGE PULL AUTH

## 90. Registry Authentication

Kubernetes may use:

```text
imagePullSecrets
workload identity
cloud-native registry integration
```

Use the least-privilege mechanism supported by the environment.

---

# PART LXIX — HELM CONSUMPTION

## 91. Helm

A deployment pipeline can consume a versioned chart:

```text
payment-chart 1.4.2
```

Pin chart versions in controlled production deployments.

---

# PART LXX — DEPENDENCY REPRODUCIBILITY

## 92. Pinning

Production builds should use controlled dependency versions through:

```text
lockfiles
dependency constraints
version ranges with lock resolution
```

as appropriate to the ecosystem.

---

# PART LXXI — RELEASE MANIFEST

## 93. Manifest

A release manifest can record:

```yaml
application: payment-api
version: 1.4.2
commit: abc123
imageDigest: sha256:...
build: 5821
```

This creates a compact release identity.

---

# PART LXXII — PROMOTION MANIFEST

## 94. Promotion

```text
artifact
 |
v
release manifest
 |
+--> DEV
+--> STAGE
+--> PROD
```

---

# PART LXXIII — ARTIFACTORY BUILD INFO

## 95. Build Info

Build information can connect artifacts and dependencies to a build.

Concept:

```text
Build 5821
 |
+--> source
+--> dependencies
+--> artifacts
```

Use it to support traceability and release investigation.

---

# PART LXXIV — CI ARTIFACT VS RELEASE ARTIFACT

## 96. Difference

CI artifact:

```text
temporary validation output
```

Release artifact:

```text
validated, versioned, retained output
```

Do not confuse the two.

---

# PART LXXV — PULL REQUEST ARTIFACTS

## 97. PR Builds

PR builds may produce artifacts for testing.

They should not automatically receive production publishing permissions.

---

# PART LXXVI — BRANCH PUBLISHING

## 98. Branch Builds

Some organizations publish development artifacts from branches.

Use clearly distinguishable versions:

```text
1.5.0-dev.<build>
```

rather than pretending they are stable releases.

---

# PART LXXVII — RELEASE VERSION

## 99. Stable

Stable version:

```text
1.5.0
```

should represent a validated release.

---

# PART LXXVIII — BUILD NUMBER

## 100. Build Number

A CI build number is useful:

```text
Build 5821
```

but should not replace the product/package version.

Use both:

```text
version = 1.5.0
build = 5821
```

---

# PART LXXIX — SOURCE REVISION

## 101. Git SHA

Always preserve the source revision:

```text
commit = abc123...
```

---

# PART LXXX — RELEASE TRACEABILITY

## 102. Full Chain

```text
Git SHA
 |
v
CI build
 |
v
version
 |
v
artifact digest
 |
v
repository
 |
v
deployment
```

This is the target for production traceability.

---

# PART LXXXI — RELEASE APPROVAL

## 103. Approval

Production promotion may require approval.

Record:

```text
approver
timestamp
version
artifact
environment
```

---

# PART LXXXII — SEPARATION OF DUTIES

## 104. Separation

For high-risk systems:

```text
developer
 |
build

approver
 |
promotion
```

This reduces unauthorized release risk.

---

# PART LXXXIII — PUBLISHING PIPELINE DESIGN

## 105. Recommended

```text
PR
 |
v
Read-only validation
 |
v
Merge
 |
v
Release tag
 |
v
Trusted runner
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
Promote
```

---

# PART LXXXIV — RELEASE REPOSITORY ACCESS

## 106. Access

Keep:

```text
developers -> read
CI release -> write
admins -> manage
```

as a starting principle, then refine permissions per project.

---

# PART LXXXV — PROJECT-LEVEL ISOLATION

## 107. Projects

Large organizations may isolate repositories or permissions by:

```text
team
business unit
application
environment
```

This reduces blast radius.

---

# PART LXXXVI — PACKAGE NAMESPACE

## 108. Namespace

Use organizational namespaces:

```text
com.company.*
@company/*
company-*
```

This reduces collisions and clarifies ownership.

---

# PART LXXXVII — OWNERSHIP

## 109. Metadata

Record:

```text
owner
team
support channel
classification
lifecycle
```

where the repository platform supports such metadata.

---

# PART LXXXVIII — LIFECYCLE

## 110. Lifecycle States

Example:

```text
development
candidate
release
deprecated
archived
```

The exact implementation depends on the repository platform.

---

# PART LXXXIX — DEPRECATION

## 111. Package Deprecation

Communicate:

```text
deprecated version
replacement
migration
removal date
```

---

# PART XC — PACKAGE REMOVAL

## 112. Removal

Before deletion:

```text
identify consumers
communicate
verify no active deployments
archive where required
```

Never delete blindly.

---

# PART XCI — RELEASE ROLLBACK

## 113. Artifact-Based Rollback

```text
current
 |
v
failure
 |
v
previous artifact
```

No rebuild should be necessary in the normal rollback path.

---

# PART XCII — CONTAINER ROLLBACK

## 114. Digest

```text
new digest
 |
X
 |
previous digest
 |
v
deployment
```

---

# PART XCIII — PACKAGE ROLLBACK

## 115. Package

Deploy the previous validated package version.

Example:

```text
1.4.3 -> 1.4.2
```

subject to database and compatibility constraints.

---

# PART XCIV — ARTIFACT REPLICATION

## 116. Multi-Region

Critical organizations may replicate artifacts between regions.

Benefits:

```text
availability
lower latency
DR
regional resilience
```

---

# PART XCV — REPLICATION CONSISTENCY

## 117. Validate

Verify:

```text
artifact checksum
metadata
version
digest
```

after replication.

---

# PART XCVI — AIR-GAPPED ENVIRONMENT

## 118. Offline

For restricted environments:

```text
approved build
 |
v
artifact bundle
 |
v
transfer/control process
 |
v
offline repository
 |
v
deployment
```

All artifacts and dependencies must be approved and transferred through
the organization's controlled process.

---

# PART XCVII — MIRRORING

## 119. Dependency Mirror

Maintain approved copies of required upstream dependencies.

This reduces reliance on external availability.

---

# PART XCVIII — SUPPLY CHAIN

## 120. Supply-Chain Controls

Use:

```text
trusted sources
dependency pinning
artifact scanning
SBOM
provenance
signing
least privilege
```

---

# PART XCIX — PRODUCTION ARCHITECTURE

## 121. Reference

```text
                         Git
                          |
                          v
                         CI
                          |
             +------------+------------+
             |            |            |
             v            v            v
           Build        Test       Security
             |            |            |
             +------------+------------+
                          |
                          v
                    Package Artifact
                          |
                          v
                    Validate/Sign
                          |
                          v
                     Artifactory
                          |
                +---------+---------+
                |                   |
                v                   v
           Candidate             Release
                |                   |
                +---------+---------+
                          |
                          v
                       Promote
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
             DEV        STAGE       PROD
```

---

# PART C — CONTAINER PRODUCTION ARCHITECTURE

## 122. Reference

```text
Git
 |
v
CI
 |
v
Docker Build
 |
v
Image Scan
 |
v
Registry
 |
v
Immutable Digest
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

---

# PART CI — INTERVIEW PREPARATION

## 123. What Is Artifact Publishing?

Answer:

```text
Artifact publishing is the controlled process of taking a validated
build output, assigning a stable identity, storing it in an approved
repository and making it available for promotion or consumption.
```

## 124. Why Build Once and Promote?

Answer:

```text
It ensures the exact artifact tested in CI is the artifact deployed
through environments. Rebuilding for every environment can produce
different outputs and weakens traceability.
```

## 125. What Is an Immutable Artifact?

Answer:

```text
An immutable artifact cannot silently change after publication. If the
contents need to change, a new version is published.
```

## 126. How Do You Secure Artifact Publishing?

Answer:

```text
I separate read and write permissions, use least-privilege release
identities, protect publishing credentials, isolate untrusted PR jobs,
scan artifacts, preserve provenance, and enforce repository
immutability.
```

## 127. How Do You Publish Maven Artifacts?

Answer:

```text
I configure the Maven repository through controlled build settings,
authenticate securely, run the appropriate deploy lifecycle, verify
coordinates and repository target, and prevent overwriting released
versions.
```

## 128. How Do You Publish npm Packages?

Answer:

```text
I verify package metadata and contents, run npm pack --dry-run when
useful, authenticate to the approved registry, publish the unique
version, and verify the resulting package.
```

## 129. How Do You Publish Python Packages?

Answer:

```text
I build distributions with python -m build, inspect wheel and source
distribution metadata and contents, run tests and security checks, then
publish the version to the approved private repository.
```

## 130. How Do You Publish Docker Images?

Answer:

```text
I build and scan the image, tag it with a unique release version, push
it to the approved registry and record the immutable digest. Production
deployment references the digest rather than relying only on a mutable
tag.
```

## 131. Publish vs Promote?

Answer:

```text
Publishing makes the artifact available in the repository. Promotion
moves the same validated artifact through lifecycle or environment
stages without rebuilding it.
```

## 132. What If the Version Already Exists?

Answer:

```text
I never overwrite an existing production artifact. I determine whether
the version is already consumed and publish a new version if the
artifact contents need to change.
```

## 133. How Do You Troubleshoot Publish Failures?

Answer:

```text
I classify the failure as authentication, authorization, network,
repository policy, duplicate version, metadata, checksum or storage.
Then I inspect the relevant logs and repository state without exposing
credentials.
```

---

# PART CII — SENIOR-LEVEL SCENARIOS

## 134. Release Job Can Publish but PR Job Cannot

Answer:

```text
That is desirable. PR jobs should generally be read-only while trusted
release jobs receive narrowly scoped publish permissions.
```

## 135. Artifact Was Changed After Publishing

Answer:

```text
I treat it as an immutability violation. I identify consumers and
deployments, preserve evidence, determine exposure, publish a correctly
versioned artifact, and fix repository permissions to prevent mutation.
```

## 136. Production Has a Different Image Than Stage

Answer:

```text
I compare immutable digests and promotion records first. If the digest
differs, I stop the rollout and correct the promotion mechanism before
investigating application behavior.
```

## 137. Artifact Repository Is Down During Release

Answer:

```text
I do not bypass repository governance by manually copying artifacts.
I follow the repository outage procedure, use approved redundancy or
recovery paths, and resume publication once artifact integrity and
repository availability are restored.
```

## 138. Registry Storage Is Growing Rapidly

Answer:

```text
I identify repository growth by package type, artifact count and large
files, then review retention, snapshot cleanup, container layer
strategy and unnecessary artifact publication while protecting
rollback requirements.
```

## 139. A Public Dependency Is Unavailable

Answer:

```text
If the dependency was already cached in the enterprise repository,
consumption may continue. Otherwise I use the approved mirror or
controlled recovery process rather than bypassing security controls
with an arbitrary external source.
```

## 140. Artifact Has a Critical Vulnerability

Answer:

```text
I identify affected versions and consumers, stop further promotion,
assess exposure, produce a patched version, scan it, publish it
immutably, redeploy affected systems and document the release lineage.
```

## 141. Publish Credentials Leak

Answer:

```text
I revoke and rotate the credentials, audit repository activity, inspect
published artifacts, assess whether unauthorized changes occurred,
restore trusted artifacts if required, and move toward short-lived or
federated authentication.
```

## 142. Need Multi-Region Artifact Availability

Answer:

```text
I use repository replication or an approved multi-region architecture,
validate checksum and metadata consistency, test failover and ensure
CI and production consumers have deterministic repository behavior.
```

## 143. Need Air-Gapped Production

Answer:

```text
I create a controlled artifact transfer process containing the exact
validated artifacts and required dependencies, verify checksums and
provenance at the boundary, and consume them from the approved offline
repository.
```

## 144. Teams Manually Copy JARs to Servers

Answer:

```text
I replace manual copying with versioned repository publishing and
deployment automation. The deployment should reference an immutable
artifact and preserve source-to-artifact-to-production traceability.
```

---

# PART CIII — PRODUCTION RUNBOOK

## 145. Publish Runbook

```text
1. Confirm source commit.
2. Confirm release version.
3. Confirm version does not already exist.
4. Checkout exact tag/ref.
5. Build in a controlled environment.
6. Run tests.
7. Run security checks.
8. Generate artifact.
9. Inspect artifact contents.
10. Generate checksum/digest.
11. Generate SBOM/provenance where required.
12. Authenticate using least privilege.
13. Publish to the approved repository.
14. Verify repository metadata.
15. Record artifact identity.
16. Promote without rebuilding.
17. Deploy to lower environment.
18. Validate.
19. Approve production promotion if required.
20. Deploy.
21. Monitor.
22. Preserve release evidence.
```

---

# PART CIV — TROUBLESHOOTING MATRIX

| Symptom | First checks |
|---|---|
| 401/403 | credentials, token scope, permissions |
| Duplicate version | repository state, concurrent release |
| Upload timeout | network, proxy, repository health |
| Checksum mismatch | artifact generation, transfer, repository |
| Missing package | coordinates, repository, promotion |
| Wrong Docker image | tag, digest, deployment manifest |
| CI publishes unexpectedly | workflow trigger, permissions |
| Large repository | retention, snapshots, large artifacts |
| Cannot rollback | retention, artifact availability |
| Stage/prod mismatch | artifact digest, promotion logs |

---

# PART CV — GOLDEN RULES

## 146. Rules 1

```text
1. Publish only validated artifacts.
2. Give every release a unique identity.
3. Associate artifacts with source commits.
4. Keep production artifacts immutable.
5. Never overwrite consumed production versions.
6. Build once and promote.
7. Store releases in durable repositories.
8. Treat CI caches as disposable.
9. Record checksums or immutable digests.
10. Record build IDs.
11. Record provenance.
12. Associate SBOMs with exact artifacts.
13. Sign artifacts when required.
14. Validate artifact contents.
15. Validate package metadata.
16. Validate dependencies.
17. Validate repository target.
18. Protect publishing credentials.
19. Use least privilege.
20. Separate read and write permissions.
```

## 147. Rules 2

```text
21. Keep PR jobs away from production publish permissions.
22. Use protected release workflows.
23. Protect release tags.
24. Prefer short-lived credentials where supported.
25. Use OIDC/federation where appropriate.
26. Do not print secrets in logs.
27. Restrict delete permissions.
28. Protect rollback artifacts.
29. Maintain artifact retention.
30. Test artifact recovery.
31. Use repository redundancy for critical systems.
32. Test disaster recovery.
33. Monitor repository capacity.
34. Monitor upload latency.
35. Monitor download latency.
36. Monitor artifact growth.
37. Monitor failed publications.
38. Monitor authentication failures.
39. Monitor authorization failures.
40. Monitor storage growth.
```

## 148. Rules 3

```text
41. Use repository naming standards.
42. Use organizational namespaces.
43. Define local/remote/virtual repository purposes.
44. Separate development and release lifecycle where useful.
45. Treat snapshots as development artifacts.
46. Use immutable release versions.
47. Inspect npm packages before publishing.
48. Inspect Python wheels and sdists.
49. Validate Maven coordinates.
50. Validate container image metadata.
51. Publish Helm charts with controlled versions.
52. Keep environment configuration outside artifacts when appropriate.
53. Never include secrets in packages.
54. Exclude unnecessary files.
55. Keep containers minimal.
56. Scan dependencies.
57. Scan containers.
58. Scan packages where required.
59. Preserve license information.
60. Preserve release metadata.
```

## 149. Rules 4

```text
61. Distinguish publishing from promotion.
62. Promote the same artifact.
63. Do not rebuild between environments.
64. Record promotion events.
65. Record approvers.
66. Record deployment targets.
67. Prefer immutable image digests.
68. Do not rely on latest for production identity.
69. Pin controlled dependency versions.
70. Maintain reproducible release inputs.
71. Use enterprise dependency mirrors where appropriate.
72. Avoid uncontrolled public registry access.
73. Maintain air-gap procedures when required.
74. Verify artifacts across replication.
75. Protect repository administration.
76. Isolate teams when blast radius requires it.
77. Define artifact ownership.
78. Define artifact lifecycle.
79. Define deprecation policy.
80. Identify consumers before deletion.
```

## 150. Rules 5

```text
81. Never delete active rollback artifacts.
82. Maintain emergency publishing procedures.
83. Maintain incident procedures for compromised artifacts.
84. Quarantine suspicious artifacts.
85. Identify consumers of compromised releases.
86. Rotate credentials after suspected compromise.
87. Rebuild from trusted source after compromise.
88. Do not manually copy artifacts around controls.
89. Automate publishing.
90. Automate validation.
91. Automate provenance collection.
92. Automate promotion where safe.
93. Keep production approval where risk requires it.
94. Separate duties for high-risk releases.
95. Keep release evidence.
96. Test rollback.
97. Test repository outage recovery.
98. Test DR.
99. Test cold-cache publishing.
100. Validate the exact artifact publishing, repository, security,
    promotion, deployment, rollback and audit architecture used in
    production.
```
---