# Artifactory-Fundamentals

## 1. Purpose

JFrog Artifactory is an artifact repository and package-management platform used to store, manage, proxy, secure and distribute software artifacts across the software delivery lifecycle.

For DevOps, Artifactory commonly sits between:

```text
Source Code
    ↓
CI/CD
    ↓
Build
    ↓
Artifactory
    ↓
Deployment
    ↓
Kubernetes / VM / Serverless / Other Runtime
```

This file establishes the fundamentals required before moving into Artifactory architecture, repository types, package-specific repositories, CI/CD integrations, security, HA, DR, troubleshooting and production projects.

---

## 2. What Is an Artifact?

An artifact is a build output or package that can be stored and consumed by another process.

Examples:

```text
JAR
WAR
NPM package
Python wheel
Python source distribution
Docker image
Helm chart
ZIP
TAR
RPM
DEB
```

Example:

```text
Application source
      ↓
     Build
      ↓
application-1.4.2.jar
```

The JAR is the artifact.

---

## 3. What Is an Artifact Repository?

An artifact repository is a centralized system for storing and distributing build artifacts.

Instead of every developer or CI runner independently obtaining dependencies from the public internet:

```text
Developer/CI
     ↓
Public Internet
     ↓
Maven Central / npm / PyPI
```

an organization can use:

```text
Developer/CI
     ↓
Artifactory
     ↓
Approved upstream repositories
```

This provides greater control over:

```text
availability
security
versioning
provenance
access
caching
retention
```

---

## 4. Why Do Organizations Need Artifactory?

Without centralized artifact management:

```text
CI
├── downloads dependency A
├── downloads dependency B
├── downloads dependency C
└── builds application
```

Problems can include:

```text
internet dependency
uncontrolled versions
dependency inconsistency
slow builds
limited traceability
difficult rollback
security concerns
```

Artifactory provides a controlled artifact layer.

---

## 5. Artifactory in a DevOps Lifecycle

A common lifecycle is:

```text
Developer
   ↓
Git
   ↓
CI Pipeline
   ↓
Build
   ↓
Test
   ↓
Security Scan
   ↓
Publish Artifact
   ↓
Artifactory
   ↓
Deploy
   ↓
Kubernetes / VM / Serverless
```

Artifactory is therefore part of the software supply chain.

---

## 6. Git vs Artifactory

Git primarily stores source code and its history.

Artifactory primarily stores build outputs and packages.

```text
Git
 ↓
Source Code
```

versus:

```text
Artifactory
 ↓
Build Artifacts
```

Example:

```text
Git:
payment-service source

Artifactory:
payment-service-2.7.1.jar
```

---

## 7. Core Artifactory Concepts

Know these concepts before going deeper:

```text
Artifact
Repository
Local Repository
Remote Repository
Virtual Repository
Repository Key
Package Format
Artifact Coordinates
Metadata
Build Info
User
Group
Permission
Access Token
```

---

## 8. What Is a Repository?

A repository is a logical location where artifacts are stored or accessed.

Common categories:

```text
Local
Remote
Virtual
```

---

## 9. Local Repository

A local repository stores artifacts owned by the organization.

Example:

```text
company-maven-local
```

A CI pipeline may publish:

```text
payment-service-1.0.0.jar
```

into this repository.

---

## 10. Remote Repository

A remote repository proxies and caches artifacts from an external repository.

Example:

```text
Artifactory
    ↓
Maven Central
```

When a dependency is requested:

```text
CI
 ↓
Artifactory Remote
 ↓
Maven Central
```

Artifactory can cache the retrieved artifact for future requests.

---

## 11. Virtual Repository

A virtual repository provides one logical endpoint over multiple repositories.

Example:

```text
maven-virtual
   ├── company-maven-local
   ├── maven-central-remote
   └── approved-third-party-repo
```

Consumers use:

```text
maven-virtual
```

instead of knowing every underlying repository.

---

## 12. Why Virtual Repositories Matter

They simplify dependency configuration.

Without a virtual repository:

```text
Application
 → internal repository
 → Maven Central
 → third-party repository
```

With a virtual repository:

```text
Application
       ↓
maven-virtual
       ↓
Artifactory
       ├── local
       ├── remote
       └── other approved repositories
```

---

## 13. Repository Naming

Organizations should establish predictable naming conventions.

Example:

```text
maven-local
maven-remote
maven-virtual

npm-local
npm-remote
npm-virtual

docker-local
docker-remote
docker-virtual

helm-local
helm-remote
helm-virtual
```

Avoid creating unnecessary repository fragmentation.

---

## 14. Artifact Coordinates

Many package ecosystems identify artifacts using coordinates.

Maven example:

```text
groupId
artifactId
version
```

Example:

```text
com.company.payment
payment-service
1.4.2
```

Conceptually:

```text
com/company/payment/payment-service/1.4.2/
```

---

## 15. Artifact Versioning

A version identifies a particular artifact release.

Example:

```text
payment-service:1.0.0
payment-service:1.1.0
payment-service:2.0.0
```

Production deployments should reference known immutable versions where possible.

---

## 16. Immutable Artifacts

An immutable artifact should not be silently replaced after publication.

Bad practice:

```text
payment-service:1.4.2
```

is published today and replaced tomorrow with different content.

Better:

```text
1.4.2
```

always represents the same build.

---

## 17. Why Immutability Matters

It supports:

```text
reproducible deployments
auditing
rollback
incident investigation
software supply-chain security
```

---

## 18. Artifact Promotion

A common enterprise model is:

```text
Build
 ↓
Test
 ↓
Security Scan
 ↓
Approved
 ↓
Promote
 ↓
Production
```

The same tested artifact should move through environments instead of being rebuilt for each environment.

---

## 19. Build Once, Promote Many

Recommended:

```text
Source
 ↓
Build once
 ↓
artifact 7f81...
 ↓
test
 ↓
staging
 ↓
production
```

Avoid:

```text
Build dev
Build staging
Build production
```

because separate builds can produce different outputs.

---

## 20. Artifact Metadata

Useful metadata can include:

```text
build number
source revision
branch
commit
build timestamp
CI system
dependencies
environment
release information
```

---

## 21. What Is Build Info?

Build Info describes how an artifact was produced.

It can connect:

```text
source
build
dependencies
artifacts
environment
CI job
```

Example:

```text
Git commit
   ↓
Jenkins build #481
   ↓
payment-service-2.3.1.jar
   ↓
Production
```

Build Info becomes especially important in CI/CD integration.

---

## 22. Why Build Traceability Matters

Suppose production has:

```text
payment-service:2.3.1
```

A good artifact process should allow the team to determine:

```text
Which source commit?
Which pipeline?
Which dependencies?
Which build?
Who published it?
When?
Which deployment?
```

---

## 23. Artifact Provenance

Provenance describes where an artifact came from and how it was produced.

A useful chain is:

```text
Developer
 ↓
Git commit
 ↓
CI build
 ↓
Tests
 ↓
Security scans
 ↓
Artifact
 ↓
Deployment
```

---

## 24. Dependency Management

Applications depend on external packages.

Example:

```text
Application
 ├── framework
 ├── logging library
 ├── HTTP client
 └── database driver
```

Artifactory can provide controlled access to these dependencies.

---

## 25. Dependency Caching

Remote repositories can cache upstream dependencies.

First build:

```text
CI
 ↓
Artifactory
 ↓
Maven Central
 ↓
Dependency
```

Later build:

```text
CI
 ↓
Artifactory cache
 ↓
Dependency
```

Benefits:

```text
faster builds
reduced internet dependency
better resilience
centralized control
```

---

## 26. Why Caching Improves Reliability

If an upstream public repository becomes temporarily unavailable, already-cached dependencies may still be available locally.

Caching does not mean every dependency is available forever; retention and repository policies must be designed intentionally.

---

## 27. Package Formats

Common DevOps examples include:

```text
Maven
Gradle
NPM
PyPI
Docker/OCI
Helm
RPM
Debian
NuGet
Go
Generic
```

The exact supported formats and features depend on the Artifactory version and edition.

---

## 28. Maven Artifact Example

```text
payment-service-2.1.0.jar
```

Coordinates:

```text
groupId:
com.company.payment

artifactId:
payment-service

version:
2.1.0
```

---

## 29. NPM Artifact Example

```text
@company/payment-client
```

Version:

```text
3.2.0
```

---

## 30. Python Artifact Example

```text
payment_client-2.1.0-py3-none-any.whl
```

---

## 31. Docker Artifact Example

```text
company/payment-service:2.1.0
```

A digest identifies immutable image content:

```text
sha256:...
```

Production systems should prefer immutable digests when practical.

---

## 32. Helm Artifact Example

```text
payment-service-1.8.0.tgz
```

Helm charts can be stored and distributed through supported Artifactory Helm repository configurations.

---

## 33. Artifactory in CI

Typical pipeline:

```text
Checkout
 ↓
Build
 ↓
Unit Tests
 ↓
Security Scan
 ↓
Publish Artifact
 ↓
Record Build Info
 ↓
Promote
```

---

## 34. Artifactory in CD

Typical deployment:

```text
Artifactory
 ↓
Approved artifact
 ↓
Deployment system
 ↓
Kubernetes
```

---

## 35. Artifactory With Jenkins

Common flow:

```text
Jenkins
 ↓
Build
 ↓
Artifactory
 ↓
Build Info
 ↓
Artifact promotion
 ↓
Deployment
```

Detailed integration is covered in:

```text
15-Jenkins-Artifactory-Integration.md
```

---

## 36. Artifactory With GitHub Actions

Common flow:

```text
GitHub
 ↓
GitHub Actions
 ↓
Build
 ↓
Artifactory
 ↓
Deployment
```

Authentication should use secure machine credentials, tokens or supported federated mechanisms.

---

## 37. Artifactory With GitLab

Typical flow:

```text
GitLab
 ↓
GitLab CI
 ↓
Build
 ↓
Artifactory
 ↓
Deployment
```

---

## 38. Artifactory With Docker

Typical workflow:

```text
Docker Build
 ↓
Tag
 ↓
Push
 ↓
Artifactory Docker Repository
 ↓
Kubernetes
```

---

## 39. Artifactory With Kubernetes

Kubernetes workloads can consume images from Artifactory.

Example:

```yaml
spec:
  containers:
    - name: payment
      image: registry.company.example/payment-service:2.1.0
```

Authentication can use Kubernetes image pull secrets or an appropriate registry integration.

---

## 40. Artifactory and ECR

Both can provide container registry capabilities, but they solve somewhat different platform needs.

Artifactory can centralize multiple package ecosystems.

ECR is AWS-native and integrates closely with AWS services.

A later file will compare them in detail:

```text
20-ECR-vs-Artifactory.md
```

---

## 41. Authentication vs Authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to do?
```

---

## 42. RBAC

Role-Based Access Control assigns permissions through roles/groups.

Example:

```text
Developer
 ↓
read virtual repositories
deploy local artifacts

Release Manager
 ↓
promote artifacts

Administrator
 ↓
manage repositories/security
```

Use least privilege.

---

## 43. Repository Permissions

A user or service may need capabilities such as:

```text
read
deploy/cache
delete
annotate
administer
```

Exact permissions depend on Artifactory version and edition.

---

## 44. Why Avoid Broad Admin Access?

Broad access can allow:

```text
artifact deletion
repository modification
credential changes
security changes
supply-chain compromise
```

Production users should receive only required permissions.

---

## 45. Authentication Methods

Enterprise environments can integrate Artifactory with identity systems.

Possible approaches include:

```text
username/password
access tokens
SSO
LDAP
SAML
OIDC/federated identity
```

Exact availability depends on deployment and edition.

---

## 46. Access Tokens

Access tokens can provide scoped and time-limited authentication for automation.

CI systems should prefer machine credentials designed for automation rather than personal passwords.

---

## 47. CI Credential Best Practice

Avoid:

```text
hard-coded passwords
tokens in Git
tokens in Dockerfiles
tokens in shell history
```

Use:

```text
CI secret store
short-lived credentials where supported
scoped tokens
rotation
audit
```

---

## 48. Artifact Security

Artifact security should cover:

```text
who published it
what source created it
what dependencies it contains
whether it was scanned
where it was promoted
who consumed it
```

---

## 49. Software Supply Chain

Artifactory can be part of a larger supply-chain security architecture:

```text
Developer
 ↓
Git
 ↓
CI
 ↓
Dependency Resolution
 ↓
Security Scan
 ↓
Artifactory
 ↓
Deployment
```

---

## 50. Artifact Scanning

Security processes may inspect:

```text
vulnerabilities
licenses
malicious packages
policy violations
secrets
```

Exact scanning capabilities depend on the JFrog products and edition in use.

---

## 51. Remote Repository Governance

Do not blindly proxy every public repository.

Instead:

```text
Approved upstream
        ↓
Artifactory Remote
        ↓
Controlled consumers
```

This improves governance.

---

## 52. Why Centralize Dependencies?

Benefits include:

```text
consistent versions
faster builds
auditability
security controls
availability
central governance
```

---

## 53. Dependency Pinning

Prefer explicit versions.

Bad:

```text
latest
```

Better:

```text
2.7.1
```

For containers:

```text
image@sha256:<digest>
```

where practical.

---

## 54. Reproducible Builds

A reproducible build should produce the same intended output from the same controlled inputs.

Important inputs include:

```text
source revision
dependency versions
build tooling
base image
configuration
```

Artifactory helps control dependency retrieval but does not alone guarantee reproducibility.

---

## 55. Snapshot vs Release

Typical Maven concept:

```text
1.4.0-SNAPSHOT
```

represents development output.

A release:

```text
1.4.0
```

should normally be treated as stable and immutable.

---

## 56. Docker Tags vs Digests

Tag:

```text
payment-service:2.1.0
```

Digest:

```text
payment-service@sha256:...
```

Tags are human-friendly references; digests identify exact content.

---

## 57. Why Tags Can Be Dangerous

If a mutable tag changes:

```text
payment-service:latest
```

the same deployment reference can point to different content later.

Use controlled versioning and immutable references for production.

---

## 58. Artifact Retention

Retention controls how long artifacts remain available.

Policies may consider:

```text
release status
age
usage
environment
storage cost
compliance
```

Do not delete artifacts blindly.

---

## 59. Cleanup Policies

A mature repository strategy defines cleanup rules for:

```text
old snapshots
temporary builds
unused images
development artifacts
cache content
```

Production release artifacts should have stronger retention guarantees.

---

## 60. Artifact Lifecycle

A practical lifecycle is:

```text
Created
 ↓
Uploaded
 ↓
Validated
 ↓
Scanned
 ↓
Approved
 ↓
Promoted
 ↓
Consumed
 ↓
Retained
 ↓
Archived/Deleted according to policy
```

---

## 61. Rollback

If production currently runs:

```text
payment-service:3.4.2
```

and release `3.4.3` fails:

```text
rollback
 ↓
3.4.2
```

The old artifact must remain available according to retention policy.

---

## 62. Storage Architecture

At a conceptual level:

```text
Consumers
   ↓
Artifactory
   ↓
Repository layer
   ↓
Artifact metadata + binary storage
```

Exact storage architecture depends on deployment type and configuration.

---

## 63. Deployment Models

Artifactory can be consumed through JFrog-managed/cloud offerings or deployed in environments controlled by the organization, depending on available product offerings.

Production architecture should consider:

```text
availability
storage
networking
security
backup
DR
scaling
```

---

## 64. Self-Hosted Considerations

For self-managed deployments consider:

```text
compute
database
object/file storage
load balancer
TLS
DNS
backup
monitoring
upgrade process
```

Exact components depend on supported Artifactory architecture/version.

---

## 65. Cloud Offering Considerations

For managed offerings, the provider handles portions of platform operations, but the customer still owns:

```text
repository design
access control
artifact governance
CI/CD integration
retention
security policies
consumer configuration
```

---

## 66. High Availability Fundamentals

Enterprise environments may require high availability.

Conceptually:

```text
              Load Balancer
                    |
          +---------+---------+
          |                   |
      Artifactory-1      Artifactory-2
          |                   |
          +---------+---------+
                    |
              Shared/managed
              persistence
```

Exact HA architecture depends on the supported product architecture.

---

## 67. Backup Fundamentals

Backups should protect:

```text
artifact data
metadata
configuration
security configuration
database/persistence
```

Backing up only binary files may be insufficient.

---

## 68. Disaster Recovery Fundamentals

DR should define:

```text
RPO
RTO
backup frequency
replication
restore procedure
DNS strategy
credential recovery
dependency recovery
```

---

## 69. RPO

Recovery Point Objective:

```text
How much data loss is acceptable?
```

---

## 70. RTO

Recovery Time Objective:

```text
How quickly must service be restored?
```

---

## 71. Monitoring Artifactory

Monitor:

```text
availability
latency
error rate
storage
repository health
request volume
authentication failures
resource utilization
```

---

## 72. Common Failure Classes

```text
authentication failure
authorization failure
artifact not found
repository misconfiguration
upstream unavailable
storage exhaustion
network failure
TLS failure
CI credential failure
Docker push/pull failure
```

---

## 73. 401 vs 403

Generally:

```text
401 → authentication is missing/invalid
403 → authenticated identity is not authorized
```

Always verify the exact response and configuration.

---

## 74. Artifact Not Found

Investigate:

```text
repository
artifact path
version
package coordinates
permissions
virtual repository routing
remote cache
```

---

## 75. Docker Push Failure

Check:

```text
registry URL
login
token
repository
permissions
TLS
network
image tag
```

---

## 76. Kubernetes Image Pull Failure

Check:

```text
image name
tag/digest
registry DNS
network
imagePullSecret
service account
repository permissions
```

---

## 77. Maven Dependency Download Failure

Check:

```text
repository URL
credentials
coordinates
virtual repository
remote upstream
network
TLS
```

---

## 78. NPM Install Failure

Check:

```text
registry configuration
authentication
package scope
repository
version
network
```

---

## 79. Helm Pull Failure

Check:

```text
repository URL
authentication
chart name
version
repository type
TLS
```

---

## 80. PyPI Install Failure

Check:

```text
index URL
package name
version
authentication
repository
upstream
```

---

## 81. CI Cannot Publish Artifact

Check:

```text
credentials
repository permissions
repository exists
artifact path
network
TLS
token scope
```

---

## 82. Artifactory Is Unreachable

Use layered troubleshooting:

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
```

---

## 83. DNS Troubleshooting

```bash
dig artifactory.example.com
```

Check:

```text
record
TTL
resolver
network path
```

---

## 84. TCP Troubleshooting

```bash
nc -vz artifactory.example.com 443
```

---

## 85. TLS Troubleshooting

```bash
openssl s_client \
  -connect artifactory.example.com:443 \
  -servername artifactory.example.com
```

---

## 86. HTTP Troubleshooting

```bash
curl -vk https://artifactory.example.com/
```

Never expose credentials in command history or shared logs.

---

## 87. Production Security Baseline

```text
[ ] TLS enabled
[ ] SSO/federation where appropriate
[ ] scoped tokens
[ ] least privilege
[ ] admin access restricted
[ ] audit logging
[ ] artifact scanning
[ ] immutable releases
[ ] approved upstreams
[ ] retention policies
```

---

## 88. Production CI/CD Baseline

```text
[ ] Build once
[ ] Test
[ ] Scan
[ ] Publish
[ ] Record provenance
[ ] Promote
[ ] Deploy
[ ] Verify
[ ] Roll back if required
```

---

## 89. Production Architecture Example

```text
                 Developers
                     |
                     v
                    Git
                     |
                     v
                  CI/CD
                     |
          +----------+----------+
          |                     |
      Dependencies          Build Output
          |                     |
          +----------+----------+
                     |
                     v
                Artifactory
          +----------+----------+
          |          |          |
       Maven      Docker      Helm
          |          |          |
          +----------+----------+
                     |
                     v
                 Deployment
                     |
          +----------+----------+
          |                     |
        EKS                    VMs
```

---

## 90. Production Dependency Flow

```text
CI
 ↓
Virtual Repository
 ↓
Remote Cache
 ↓
Approved Upstream
```

Internal packages:

```text
CI
 ↓
Local Repository
```

---

## 91. Production Artifact Flow

```text
Source
 ↓
CI
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Artifactory Local
 ↓
Promotion
 ↓
Production Consumer
```

---

## 92. What Should Not Be Stored in Artifactory?

Do not treat Artifactory as a general-purpose secret store.

Avoid storing:

```text
passwords
private keys
API credentials
production secrets
```

Use an appropriate secrets-management system.

---

## 93. Artifact vs Secret

Artifact:

```text
payment-service-2.4.1.jar
```

Secret:

```text
DATABASE_PASSWORD
```

They have different security and lifecycle requirements.

---

## 94. Artifact vs Configuration

Artifact:

```text
application binary
```

Configuration:

```text
environment-specific settings
```

Keep environment-specific secrets/configuration separate from immutable application binaries where practical.

---

## 95. Artifactory and GitOps

A GitOps system may store deployment manifests in Git while container images and Helm packages live in Artifactory.

Example:

```text
Git
 ↓
Helm values / manifests
          |
          v
      Argo CD
          |
          v
       Kubernetes
          |
          v
    Artifactory image
```

Argo CD manages desired state; Artifactory provides the artifact.

---

## 96. Artifactory as a Reliability Layer

A repository can reduce external dependency during builds by caching artifacts.

However:

```text
cache ≠ complete DR
```

Backups, replication and recovery procedures remain necessary.

---

## 97. Public Dependency Outage Example

If Maven Central becomes unavailable:

```text
CI
 ↓
Artifactory Remote Cache
 ↓
Cached dependency
```

Previously cached dependencies may continue to be served.

---

## 98. Production Repository Full

Symptoms:

```text
uploads fail
system alerts
performance degradation
```

Response:

```text
identify largest repositories
review retention
remove approved temporary content
expand storage
restore headroom
```

Do not delete production artifacts blindly.

---

## 99. Artifactory Outage

Potential impact:

```text
CI dependency resolution
artifact publishing
container pulls
Helm pulls
deployments
```

Therefore production architecture should consider:

```text
HA
backup
DR
caching
monitoring
capacity
```

---

## 100. Production Artifact Corruption

If an artifact does not match expected content:

Investigate:

```text
artifact checksum/digest
repository
upload process
storage
network
consumer cache
```

Do not simply overwrite the release artifact.

---

# 101. Interview Question — What Problem Does Artifactory Solve?

### Strong Answer

```text
Artifactory provides centralized artifact and package management. It
stores build outputs, proxies and caches external dependencies,
controls access, supports package ecosystems and provides a reliable
source for artifacts consumed by CI/CD and runtime platforms.
```

---

# 102. Interview Question — Why Not Download Dependencies Directly From the Internet?

### Strong Answer

```text
Direct internet dependency creates availability, security, governance
and reproducibility concerns. Artifactory provides a controlled
repository boundary, caching, access control and better traceability.
```

---

# 103. Interview Question — Local vs Remote vs Virtual

### Strong Answer

```text
Local stores organization-owned artifacts. Remote proxies and caches
artifacts from external repositories. Virtual provides one logical
endpoint across multiple local and remote repositories.
```

---

# 104. Interview Question — What Is Build Info?

### Strong Answer

```text
Build Info provides metadata about a build and its relationship to
source, dependencies, artifacts and the CI process. It improves
traceability and release visibility.
```

---

# 105. Interview Question — Why Are Immutable Artifacts Important?

### Strong Answer

```text
They ensure a version continues to represent the same content. This
supports reproducibility, reliable rollback, auditing and supply-chain
security.
```

---

# 106. Interview Question — What Is Artifact Promotion?

### Strong Answer

```text
Artifact promotion moves an already-built and validated artifact
through release stages such as testing, staging and production
without rebuilding it.
```

---

# 107. Interview Question — Artifactory vs ECR?

### Strong Answer

```text
ECR is an AWS-native container registry with strong AWS integration.
Artifactory is a broader artifact-management platform that can
centralize multiple package formats such as Maven, NPM, PyPI, Docker
and Helm. The choice depends on the organization's package ecosystem,
AWS dependency, governance and platform requirements.
```

---

# 108. Interview Question — How Do You Secure Artifactory?

### Strong Answer

```text
I use TLS, centralized identity where appropriate, least-privilege
RBAC, scoped automation credentials, controlled repository access,
approved upstreams, artifact scanning, audit logging and immutable
release practices. I separate administrative access from normal
CI/CD access.
```

---

# 109. Interview Question — CI Upload Failure

### Strong Answer

```text
I verify DNS, network and TLS connectivity, then authentication,
authorization and repository existence. I verify the artifact path and
package format, inspect CI and Artifactory logs, and confirm that the
token has the required repository permissions.
```

---

# 110. Interview Question — Kubernetes Image Pull Failure

### Strong Answer

```text
I verify the image repository, tag or digest, DNS and network
connectivity, registry authentication, imagePullSecrets or workload
identity, repository permissions and the exact Kubernetes event.
```

---

# 111. Interview Question — Production Artifactory Architecture

### Strong Answer

```text
I design for availability, secure network access, TLS, controlled
repositories, least privilege, artifact retention, monitoring,
centralized logging, backups and tested disaster recovery. I also
plan storage growth and CI/CD dependency capacity before production
rollout.
```

---

# 112. Fundamental Troubleshooting Decision Tree

```text
Cannot access Artifactory?
        |
        +-- DNS?
        |     |
        |     +-- No → DNS investigation
        |
        +-- TCP?
        |     |
        |     +-- No → routing/firewall investigation
        |
        +-- TLS?
        |     |
        |     +-- No → certificate/SNI/trust investigation
        |
        +-- HTTP?
        |     |
        |     +-- 401 → authentication
        |     +-- 403 → authorization
        |     +-- 404 → repository/path/artifact
        |
        +-- Package operation?
              |
              +-- Check repository/client configuration
              +-- Check permissions
              +-- Check upstream/cache
```

---

# 113. What You Should Know Before File 02

Before moving to:

```text
02-Artifactory-Architecture.md
```

you should understand:

```text
what Artifactory is
why artifact repositories exist
artifact lifecycle
local repositories
remote repositories
virtual repositories
package formats
versioning
immutability
promotion
Build Info
authentication
RBAC
security
CI/CD role
Kubernetes role
HA/backup/DR fundamentals
```

---

# 114. Final Fundamentals Mental Model

```text
                SOFTWARE SUPPLY CHAIN

Developer
   ↓
Git
   ↓
CI/CD
   ↓
Build
   ↓
Test / Scan
   ↓
ARTIFACTORY
   ├── Local repositories
   ├── Remote repositories
   ├── Virtual repositories
   └── Metadata / Build Info
   ↓
Promotion
   ↓
Deployment
   ↓
Kubernetes / VM / Runtime
```

Artifactory is not simply storage for JAR files.

It is a control point for:

```text
artifact management
dependency management
release management
software supply-chain governance
traceability
security
reliability
```

---

# 115. Final Interview Answer

### "Explain Artifactory in a production DevOps environment."

```text
Artifactory is the centralized artifact and package-management layer
in the software delivery lifecycle. We use it to store internally
built artifacts, proxy and cache approved external dependencies, and
provide controlled repository endpoints for CI/CD and runtime
platforms.

I would typically separate local repositories for organization-owned
artifacts, remote repositories for external dependencies and virtual
repositories for a unified consumer endpoint. I would enforce
authentication, least-privilege permissions, scoped CI credentials,
artifact immutability, retention and security scanning.

In the pipeline, the application is built and tested, the artifact is
published to Artifactory with traceability metadata, and the same
validated artifact is promoted through environments rather than being
rebuilt. For Kubernetes, container images and Helm packages can be
consumed from Artifactory.

For production I also consider HA, storage capacity, monitoring,
backups, disaster recovery and network security because the artifact
repository can become a critical dependency for CI/CD and runtime
scaling.
```

---

# 116. Final Golden Rules

```text
1. Treat artifacts as release assets, not temporary files.
2. Prefer immutable production artifacts.
3. Build once and promote the same artifact.
4. Use virtual repositories to simplify consumers.
5. Control remote repositories and upstream dependencies.
6. Never store CI credentials in Git.
7. Use scoped credentials and least privilege.
8. Separate artifact storage from secret management.
9. Keep production artifacts available for rollback.
10. Define retention before implementing cleanup.
11. Record artifact provenance and build metadata.
12. Monitor storage, availability and errors.
13. Test backup restoration, not only backup creation.
14. Test DR against defined RPO/RTO.
15. Treat Artifactory as part of the software supply chain.
16. Troubleshoot DNS → TCP → TLS → HTTP → authentication →
    authorization → repository/package behavior.
17. Prefer automation and IaC over undocumented manual changes.
18. Protect the artifact repository because compromise can affect
    the software delivery pipeline.
```

---