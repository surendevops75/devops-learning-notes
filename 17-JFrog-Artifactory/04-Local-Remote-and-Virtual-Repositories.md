# 17-JFrog-Artifactory
# 04-Local-Remote-and-Virtual-Repositories

## 1. Purpose

This file goes deeper than the previous repository-type overview and focuses on how Local, Remote and Virtual repositories are actually used together in production DevOps workflows.

The central architecture is:

```text
                     CONSUMERS
                         |
                         v
                  VIRTUAL REPOSITORY
                    /           \
                   /             \
                  v               v
             LOCAL             REMOTE
               |                  |
               v                  v
        Internal Artifacts    External Upstream
                                  |
                                  v
                                Cache
```

The objective is to understand repository configuration, dependency resolution, publishing, permissions, caching, lifecycle, security, troubleshooting and production design.

---

# PART I — LOCAL REPOSITORIES

## 2. Local Repository Definition

A Local Repository is the repository where artifacts owned and published by the organization are managed.

Typical examples:

```text
maven-local
npm-local
pypi-local
docker-local
helm-local
```

Example:

```text
Jenkins
   ↓
Build
   ↓
payment-service-4.2.0.jar
   ↓
maven-local
```

---

## 3. Local Repository Responsibilities

A local repository can provide:

```text
artifact storage
artifact publication
version management
metadata
access control
retention
promotion workflow
auditability
```

It becomes the authoritative location for internally produced artifacts.

---

## 4. Local Repository Ownership

The organization owns the artifact lifecycle.

```text
Create
 ↓
Build
 ↓
Publish
 ↓
Validate
 ↓
Scan
 ↓
Promote
 ↓
Consume
 ↓
Retain
 ↓
Archive/Delete
```

---

## 5. Local Repository Example

Suppose a company has:

```text
payment-service
order-service
customer-service
```

A Maven repository may contain:

```text
maven-local/
 ├── com/company/payment/payment-service/
 ├── com/company/order/order-service/
 └── com/company/customer/customer-service/
```

The exact physical path is package-format dependent.

---

## 6. Local Repository Publishing

Typical CI flow:

```text
Git
 ↓
Jenkins
 ↓
mvn clean test
 ↓
mvn package
 ↓
Security Scan
 ↓
mvn deploy
 ↓
maven-local
```

The CI identity requires appropriate deploy permissions.

---

## 7. Local Repository Read Access

Consumers can retrieve artifacts:

```text
CI
Developers
Deployment systems
Kubernetes workloads
Argo CD
Other services
```

Only the required read access should be granted.

---

## 8. Local Repository Write Access

Write access should be restricted.

Typical:

```text
Developer → No production deploy
CI → Deploy
Release automation → Controlled promotion
Admin → Administrative operations
```

---

## 9. Local Repository Delete Access

Delete access is highly sensitive.

A deletion can affect:

```text
rollback
reproducibility
production deployment
dependency resolution
incident recovery
```

Restrict delete permissions.

---

## 10. Release Repository

A production-oriented local repository may contain release artifacts.

Example:

```text
maven-release-local
```

Conceptually:

```text
Build
 ↓
Validation
 ↓
Approved Artifact
 ↓
Release Repository
```

Whether separate release repositories are appropriate depends on organizational governance.

---

## 11. Development Repository

Some organizations separate development artifacts:

```text
maven-dev-local
```

This can contain:

```text
SNAPSHOT
development builds
temporary integration artifacts
```

Retention should normally be more aggressive than for production releases.

---

## 12. Snapshot Artifacts

Example:

```text
payment-service-4.3.0-SNAPSHOT
```

Snapshots are development artifacts.

They should not be treated as production releases.

---

## 13. Release Artifacts

Example:

```text
payment-service-4.3.0
```

Production release artifacts should normally be immutable.

---

## 14. Local Repository Immutability

A strong production rule:

```text
4.3.0 → one artifact content
```

Do not silently replace it with another build.

This supports:

```text
reproducibility
audit
rollback
incident investigation
```

---

## 15. Local Repository Promotion

A typical lifecycle:

```text
Development
 ↓
Test
 ↓
Security Scan
 ↓
Approval
 ↓
Production
```

The same artifact should move through the lifecycle rather than being rebuilt.

---

## 16. Build Once, Promote Many

Recommended:

```text
Source
 ↓
CI Build #721
 ↓
artifact digest/checksum
 ↓
Test
 ↓
Staging
 ↓
Production
```

Avoid:

```text
Build Dev
Build Stage
Build Prod
```

because the outputs may differ.

---

## 17. Local Repository and Build Info

A build can be associated with:

```text
Git commit
CI job
build number
dependencies
artifact
timestamp
```

This gives traceability.

---

## 18. Local Repository and CI Identity

Example:

```text
jenkins-payment-publisher
```

Permissions:

```text
read:
  maven-virtual

deploy:
  payment-maven-local

delete:
  none
```

This is safer than a global administrator token.

---

## 19. Local Repository Naming

Good:

```text
payment-maven-local
platform-maven-local
shared-maven-local
```

Bad:

```text
repo1
newrepo
testfinal
abc
```

Names should communicate purpose.

---

## 20. Local Repository Ownership Metadata

Document:

```text
owner
team
package format
purpose
retention
security policy
consumers
CI pipeline
```

---

# PART II — REMOTE REPOSITORIES

## 21. Remote Repository Definition

A Remote Repository provides controlled access to an external package source and can cache retrieved content.

Example:

```text
maven-central-remote
```

Flow:

```text
CI
 ↓
Artifactory Remote
 ↓
Maven Central
```

---

## 22. Remote Repository as a Trust Boundary

The remote repository becomes the controlled boundary between:

```text
External ecosystem
        |
        v
Artifactory
        |
        v
Internal organization
```

This is important for software supply-chain governance.

---

## 23. Remote Repository Benefits

```text
centralized dependency access
cache
faster builds
reduced internet traffic
upstream governance
centralized authentication
security scanning
auditing
```

---

## 24. Remote Repository Cache

First request:

```text
Developer
 ↓
Artifactory
 ↓
Upstream
 ↓
Artifact
 ↓
Artifactory Cache
 ↓
Developer
```

Later request:

```text
Developer
 ↓
Artifactory
 ↓
Cached Artifact
```

---

## 25. Cache Hit

A cache hit occurs when Artifactory can satisfy a request from locally available cached content.

Benefits:

```text
lower latency
less upstream traffic
greater resilience
```

---

## 26. Cache Miss

A cache miss may require:

```text
Artifactory
 ↓
Upstream
```

If the upstream is unavailable:

```text
Cache Miss
 +
Upstream Down
 =
Request Failure
```

---

## 27. Cache Does Not Mean Full Mirror

Important:

```text
Remote Repository ≠ Complete copy of upstream
```

Only content retrieved and retained according to configuration/policy may be available locally.

---

## 28. Remote Repository Upstream

Examples:

```text
Maven Central
npm registry
PyPI
Docker registry
approved vendor registry
```

The upstream must be explicitly configured.

---

## 29. Approved Upstream Strategy

Enterprise model:

```text
External Registry
       ↓
Security Review
       ↓
Approved Remote Repository
       ↓
Virtual Repository
       ↓
Consumers
```

---

## 30. Direct Internet Dependency Anti-Pattern

Bad:

```text
Jenkins
  ├── Maven Central
  ├── npm
  ├── PyPI
  ├── Docker Hub
  └── Random registry
```

Better:

```text
Jenkins
   ↓
Artifactory Virtual
   ↓
Approved Remote Repositories
```

---

## 31. Remote Repository Authentication

There can be two separate credential paths.

Consumer authentication:

```text
CI → Artifactory
```

Upstream authentication:

```text
Artifactory → External Registry
```

Do not confuse these.

---

## 32. Remote Upstream Credentials

If an upstream requires authentication:

```text
Artifactory
 ↓
Secure upstream credentials
 ↓
Vendor registry
```

Credentials should be:

```text
scoped
protected
rotated
audited
```

---

## 33. Remote Repository Security

Controls may include:

```text
approved upstream
HTTPS
authentication
access control
package scanning
policy checks
audit logging
```

---

## 34. Remote Repository Availability

Possible states:

```text
Artifactory healthy
Upstream healthy
Cache healthy
```

or:

```text
Artifactory healthy
Upstream unavailable
Cache contains artifact
```

The second scenario may still work for cached dependencies.

---

## 35. Remote Repository Upstream Outage

If dependency is cached:

```text
Consumer
 ↓
Remote Cache
```

If dependency is not cached:

```text
Consumer
 ↓
Remote
 ↓
Upstream unavailable
 ↓
Failure
```

---

## 36. Remote Repository Rate Limiting

Public registries may limit requests.

Artifactory caching can reduce repeated upstream requests.

Architecture:

```text
100 CI jobs
     ↓
Artifactory
     ↓
Cached Dependency
```

instead of:

```text
100 CI jobs
     ↓
Public Registry
```

---

## 37. Remote Repository Supply-Chain Risk

External packages can introduce:

```text
vulnerabilities
malicious code
license issues
typosquatting
dependency confusion
```

Use governance and scanning.

---

## 38. Dependency Confusion

Suppose an internal application expects:

```text
company-payment-client
```

An attacker could publish a package with the same or similar name to a public registry.

A controlled repository architecture should ensure internal namespaces resolve from trusted internal sources.

---

## 39. Internal Namespace Strategy

Examples:

Maven:

```text
com.company.*
```

NPM:

```text
@company/*
```

Python:

```text
company_*
```

The exact convention should be standardized by the organization.

---

## 40. Remote Repository Retention

Remote cache retention should consider:

```text
storage cost
build frequency
upstream reliability
recovery requirements
```

Do not remove cached content without understanding build dependencies.

---

# PART III — VIRTUAL REPOSITORIES

## 41. Virtual Repository Definition

A Virtual Repository is a consumer-facing aggregation endpoint over multiple repositories.

Example:

```text
maven-virtual
    |
    +-- maven-local
    |
    +-- maven-central-remote
```

---

## 42. Why Virtual Repositories Are Important

Consumers get:

```text
one endpoint
```

instead of:

```text
many endpoints
```

This reduces configuration complexity.

---

## 43. Virtual Repository Consumer Flow

```text
Developer
   |
Jenkins
   |
GitHub Actions
   |
GitLab
   |
   v
Virtual Repository
```

---

## 44. Virtual Repository Resolution

Conceptually:

```text
Request
 ↓
Virtual
 ↓
Configured local repositories
 ↓
Configured remote repositories
 ↓
Upstream when needed
```

Actual resolution order and behavior depend on package format and repository configuration.

---

## 45. Virtual Repository and Client Configuration

Maven example:

```text
https://artifactory.company.com/artifactory/maven-virtual/
```

The exact URL format depends on the deployment.

Developers should configure the virtual endpoint rather than every underlying repository.

---

## 46. Virtual Repository Simplifies Migration

Suppose:

```text
maven-central-remote
```

is replaced by:

```text
approved-maven-remote
```

If consumers use:

```text
maven-virtual
```

the underlying change may be possible without changing every client configuration.

This is one of the major architectural benefits.

---

## 47. Virtual Repository Stability

Because many consumers depend on it:

```text
virtual repository = platform API
```

Changes should go through testing and change management.

---

## 48. Virtual Repository Change Risk

Adding an upstream can affect:

```text
dependency resolution
security
package source
build reproducibility
```

Example:

```text
Before:
virtual → local + Maven Central

After:
virtual → local + Maven Central + vendor repo
```

The new vendor repository may change where a dependency is obtained.

---

## 49. Virtual Repository and Dependency Precedence

Resolution behavior depends on Artifactory/package-manager configuration.

Never assume:

```text
first listed repository
=
always selected
```

Verify behavior for the package ecosystem and exact configuration.

---

## 50. Virtual Repository Security

A virtual repository can expose repositories with different trust levels.

Therefore verify:

```text
which repositories are included
who can read
which upstreams are reachable
what packages are exposed
```

---

## 51. Virtual Repository Permission Model

Example:

```text
Developer
 ↓
maven-virtual
 ↓
read
```

They may not have:

```text
deploy
delete
admin
```

on the virtual endpoint.

---

## 52. Publishing Through Virtual Repository

Do not design a pipeline around the assumption that a virtual repository is a normal publication target.

Use the appropriate local repository for publishing according to package format and Artifactory configuration.

---

## 53. Virtual Repository for Dependencies

A common pattern:

```text
CI
 ↓
maven-virtual
 ↓
dependencies
```

Publishing:

```text
CI
 ↓
maven-local
 ↓
application artifact
```

---

## 54. Complete Java Flow

```text
                 Maven Build
                     |
          +----------+----------+
          |                     |
          v                     v
   Dependency Read       Artifact Publish
          |                     |
          v                     v
   maven-virtual          maven-local
      /      \                  |
     /        \                 v
 local       remote        Application JAR
              |
              v
         Maven Central
```

---

## 55. Complete Docker Flow

```text
                  Docker Build
                       |
              +--------+--------+
              |                 |
              v                 v
       Base Image Pull      App Image Push
              |                 |
              v                 v
      docker-virtual      docker-local
          /       \
         /         \
docker-local    docker-remote
                    |
                    v
               Upstream
```

---

## 56. Complete NPM Flow

```text
npm install
     |
     v
npm-virtual
   /      \
  /        \
local      remote
             |
             v
          npm registry
```

Publishing:

```text
npm publish
     ↓
npm-local
```

---

## 57. Complete Python Flow

```text
pip install
     |
     v
pypi-virtual
   /      \
local     remote
            |
            v
           PyPI
```

Publishing:

```text
python package build
        ↓
pypi-local
```

---

## 58. Complete Helm Flow

```text
helm pull
   ↓
helm-virtual
 /         \
local      remote
```

Publishing:

```text
helm package
    ↓
helm-local
```

---

# PART IV — COMBINING THE THREE

## 59. Standard Enterprise Pattern

```text
                     CONSUMERS
                         |
                         v
                    VIRTUAL
                   /       \
                  /         \
                 v           v
              LOCAL        REMOTE
                |             |
                v             v
           Internal        Upstream
           Artifacts        Cache
```

---

## 60. Standard CI Pattern

```text
Git
 ↓
CI
 ↓
Dependency Resolution
 ↓
Virtual
 ↓
Local + Remote
 ↓
Build
 ↓
Security Scan
 ↓
Local
 ↓
Promotion
 ↓
Production
```

---

## 61. Standard Kubernetes Pattern

```text
Deployment
 ↓
Kubernetes
 ↓
Container Registry Endpoint
 ↓
Virtual or Local
 ↓
Image
```

Runtime should normally have read-only access.

---

## 62. Standard GitOps Pattern

```text
Git
 ↓
Argo CD
 ↓
Kubernetes
 ↓
Image
 ↓
Artifactory
```

If Helm is used:

```text
Argo CD
 ↓
Helm Repository
 ↓
Artifactory
```

---

## 63. Production Trust Model

```text
External
   ↓
REMOTE
   ↓
Security / Policy
   ↓
VIRTUAL
   ↓
Internal Consumers
   ↓
LOCAL
   ↓
Validated Release
   ↓
Production
```

---

## 64. Separation of Responsibilities

```text
Remote
  → external dependencies

Virtual
  → dependency consumption interface

Local
  → internal artifact ownership
```

This is the most important conceptual separation.

---

## 65. Repository Permissions by Type

Recommended baseline:

```text
LOCAL:
  CI → deploy
  consumers → read

REMOTE:
  consumers → read
  Artifactory → upstream access

VIRTUAL:
  consumers → read
```

Administrative permissions remain separate.

---

## 66. Repository Permission Example

```text
payment-ci
 |
 +-- READ  maven-virtual
 |
 +-- DEPLOY payment-maven-local
 |
 +-- DELETE none
 |
 +-- ADMIN none
```

---

## 67. Runtime Permission Example

```text
eks-payment-runtime
 |
 +-- READ docker-virtual
 |
 +-- DEPLOY none
 |
 +-- DELETE none
 |
 +-- ADMIN none
```

---

## 68. Release Automation Permission Example

```text
release-bot
 |
 +-- READ release source
 +-- PROMOTION controlled
 +-- DELETE only if explicitly required
 +-- ADMIN none
```

---

# PART V — PRODUCTION GOVERNANCE

## 69. Repository Request Process

A production repository request should capture:

```text
repository name
type
package format
owner
purpose
consumers
upstream
permissions
retention
security
backup
DR
```

---

## 70. Repository Review

Before approval:

```text
Is repository required?
Is naming correct?
Is type correct?
Is owner defined?
Are permissions least privilege?
Is upstream approved?
Is retention defined?
Is monitoring enabled?
```

---

## 71. Repository Lifecycle Management

```text
Request
 ↓
Design
 ↓
Approval
 ↓
Create
 ↓
Configure
 ↓
Onboard
 ↓
Monitor
 ↓
Review
 ↓
Retire
```

---

## 72. Repository Retirement

Before deletion:

```text
Search consumers
 ↓
Check CI
 ↓
Check deployments
 ↓
Check dependencies
 ↓
Migrate
 ↓
Monitor
 ↓
Retire
```

Never delete a production repository just because it appears inactive.

---

## 73. Retention by Repository Type

Typical strategy:

```text
Development Local
 → shorter retention

Production Local
 → long retention

Remote Cache
 → policy-based retention

Virtual
 → no independent artifact ownership
```

Actual retention periods should be based on business requirements.

---

## 74. Cleanup

Cleanup policies should consider:

```text
age
version
release status
usage
environment
storage cost
```

Avoid deleting artifacts required for rollback.

---

## 75. Repository Storage Monitoring

Track:

```text
repository size
artifact count
growth rate
cache growth
large artifacts
```

---

## 76. Repository Traffic Monitoring

Track:

```text
download rate
upload rate
request rate
error rate
latency
```

---

## 77. Repository Security Monitoring

Track:

```text
failed authentication
permission failures
unexpected uploads
deletions
admin changes
remote repository changes
```

---

## 78. Remote Upstream Monitoring

Track:

```text
upstream failures
cache misses
latency
authentication failures
TLS failures
```

---

## 79. Virtual Repository Monitoring

Track:

```text
consumer requests
404 rate
403 rate
dependency resolution failures
underlying repository failures
```

---

## 80. Local Repository Monitoring

Track:

```text
upload failures
download failures
storage
deletions
publication rate
```

---

# PART VI — TROUBLESHOOTING

## 81. Troubleshooting Model

Use:

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
Repository Type
 ↓
Artifact
 ↓
Upstream
```

---

## 82. 401

Usually indicates authentication problems.

Check:

```text
credentials
token
expiration
client configuration
```

---

## 83. 403

Usually indicates authorization or policy problems.

Check:

```text
permission target
repository permission
token scope
project access
```

---

## 84. 404

Check:

```text
repository
artifact path
version
virtual repository membership
upstream
```

---

## 85. 409

May indicate an artifact/version conflict or another repository operation conflict.

Check:

```text
existing artifact
deployment policy
client behavior
repository configuration
```

---

## 86. 5xx

Investigate:

```text
Artifactory node
database
storage
network
resource saturation
recent changes
```

---

## 87. Maven Dependency Failure

Check:

```text
Maven settings
repository URL
virtual repository
credentials
coordinates
remote upstream
cache
```

---

## 88. Docker Pull Failure

Check:

```text
registry hostname
image name
tag/digest
DNS
network
authentication
repository permission
```

---

## 89. Docker Push Failure

Check:

```text
login
repository
deploy permission
image tag
TLS
network
```

---

## 90. NPM Failure

Check:

```text
.npmrc
registry
scope
credentials
package version
virtual repository
```

---

## 91. Pip Failure

Check:

```text
index-url
trusted certificates
credentials
package name
version
virtual repository
```

---

## 92. Helm Failure

Check:

```text
repository URL
chart
version
authentication
TLS
repository type
```

---

## 93. CI Dependency Failure

Ask:

```text
Can CI reach Artifactory?
Can CI authenticate?
Can it read virtual?
Does virtual include required repository?
Is artifact cached?
Is upstream healthy?
```

---

## 94. CI Publish Failure

Ask:

```text
Can CI reach Artifactory?
Can CI authenticate?
Does service identity have deploy permission?
Is target local repository correct?
Is version allowed?
```

---

## 95. Runtime Image Failure

Ask:

```text
Does DNS resolve?
Can node reach registry?
Is authentication valid?
Does image exist?
Does identity have read access?
```

---

# PART VII — PRODUCTION SCENARIOS

## 96. Scenario — Maven Central Outage

Architecture:

```text
CI
 ↓
maven-virtual
 ↓
maven-central-remote
 ↓
Cache
```

If dependency exists in cache:

```text
Build may continue
```

If not:

```text
Build may fail
```

Mitigation:

```text
pre-cache critical dependencies
control upstreams
maintain reliable repository infrastructure
```

---

## 97. Scenario — CI Cannot Upload

Flow:

```text
CI
 ↓
Artifactory
 ↓
maven-local
```

Check:

```text
DNS
TLS
authentication
permission
repository
version
storage
```

---

## 98. Scenario — Kubernetes Cannot Pull

Flow:

```text
EKS
 ↓
DNS
 ↓
Network
 ↓
Registry
 ↓
Authentication
 ↓
Docker repository
```

Check each layer.

---

## 99. Scenario — Virtual Repository Changed

After configuration change:

```text
Build failures
```

Possible reason:

```text
dependency resolution changed
```

Response:

```text
review change
compare previous configuration
test dependency source
rollback if necessary
```

---

## 100. Scenario — Wrong Artifact Version

If deployment uses:

```text
payment-service:latest
```

and content changes unexpectedly, reproducibility suffers.

Better:

```text
payment-service:4.2.1
```

or immutable digest.

---

## 101. Scenario — Remote Repository Compromised

If upstream is suspected malicious:

```text
Stop affected dependency flow
 ↓
Identify affected artifacts
 ↓
Review builds
 ↓
Scan
 ↓
Quarantine/deny
 ↓
Rebuild from trusted source
 ↓
Investigate provenance
```

---

## 102. Scenario — Repository Storage Full

Response:

```text
Protect availability
 ↓
Identify largest consumers
 ↓
Review retention
 ↓
Remove only approved disposable data
 ↓
Expand capacity
 ↓
Add stronger alerts
```

---

## 103. Scenario — Repository Needs Rename

Do not immediately rename and break clients.

Instead:

```text
Inventory consumers
 ↓
Create target
 ↓
Migrate
 ↓
Update clients
 ↓
Validate
 ↓
Retire old
```

---

## 104. Scenario — Need New External Package Source

Process:

```text
Request
 ↓
Security review
 ↓
Business approval
 ↓
Create Remote
 ↓
Add to Virtual
 ↓
Test
 ↓
Monitor
```

---

# PART VIII — INTERVIEW PREPARATION

## 105. Explain Local, Remote and Virtual

Strong answer:

```text
Local repositories store artifacts owned by the organization. Remote
repositories represent external package sources and provide proxy/cache
behavior. Virtual repositories aggregate multiple local and remote
repositories behind one consumer-facing endpoint.
```

---

## 106. Why Use a Virtual Repository?

Strong answer:

```text
It gives developers and CI a stable endpoint and hides repository
topology. We can change underlying local or remote repositories with
less client reconfiguration while maintaining centralized governance.
```

---

## 107. What Is the Main Difference Between Local and Remote?

Strong answer:

```text
Local is organization-owned content and is normally the publication
target for internal builds. Remote represents external content and
can cache artifacts retrieved from an upstream repository.
```

---

## 108. What Happens During a Remote Cache Miss?

Strong answer:

```text
Artifactory attempts to retrieve the artifact from the configured
upstream. If the upstream is unavailable or the artifact cannot be
retrieved, the consumer request may fail.
```

---

## 109. Is a Remote Repository a Backup?

Strong answer:

```text
No. A remote repository cache is not a replacement for backup. Backup
must cover the required artifact data, metadata and configuration
needed for recovery.
```

---

## 110. Is a Virtual Repository a Storage Location?

Strong answer:

```text
It is primarily an aggregation and consumer-access layer. Artifact
publication should normally target the appropriate local repository.
```

---

## 111. How Would You Design Repository Permissions?

Strong answer:

```text
I start with the minimum operation required. Runtime consumers get
read-only access. CI gets read access to dependency repositories and
deploy access only to its designated local repository. Release
automation receives controlled promotion permissions. Administrative
permissions remain separate.
```

---

## 112. How Would You Prevent Dependency Confusion?

Strong answer:

```text
I use internal namespaces, approved upstreams, controlled remote
repositories, virtual repositories, dependency scanning and
restricted direct access to public registries.
```

---

## 113. How Would You Design Repositories for 100+ Teams?

Strong answer:

```text
I would avoid creating repositories blindly per team. I would define
standard repository patterns by package format and trust/lifecycle
boundary, use projects and permission targets for isolation, provide
shared virtual repositories and assign clear ownership and retention
policies.
```

---

## 114. How Would You Design Repositories for EKS?

Strong answer:

```text
I would provide a controlled Docker/OCI repository endpoint and give
EKS workloads read-only access. I would ensure DNS, network, TLS and
authentication are reliable and capacity-plan for deployment and
autoscaling image-pull bursts.
```

---

## 115. How Would You Handle Multiple Upstreams?

Strong answer:

```text
I would create separate approved remote repositories and expose only
the required ones through virtual repositories. I would document
trust, ownership and security for each upstream and test dependency
resolution before production changes.
```

---

## 116. How Would You Handle Production Repository Changes?

Strong answer:

```text
I treat repository changes as platform changes. I identify consumers,
assess impact, test the change, implement it through change
management, monitor the result and keep a rollback path.
```

---

## 117. What Is Your Golden Repository Model?

Strong answer:

```text
For dependencies, consumers use virtual repositories. Virtual
repositories expose approved local and remote repositories. Remote
repositories provide controlled external dependency access and
caching. Local repositories store organization-owned build outputs.
CI publishes to local repositories and consumes dependencies through
virtual repositories. Production runtimes receive read-only access
to the required artifact repository.
```

---

# PART IX — REFERENCE ARCHITECTURES

## 118. Basic

```text
Developer
   |
   v
Virtual
 /   \
Local Remote
        |
        v
      Upstream
```

---

## 119. CI/CD

```text
                 Git
                  |
                  v
                 CI
              /     \
             /       \
       Dependency    Publish
          Read          |
            |            v
            v         Local
         Virtual         |
         /     \         v
      Local   Remote   Artifact
                |
                v
             Upstream
```

---

## 120. Kubernetes

```text
GitOps
  |
  v
Kubernetes
  |
  v
Registry Endpoint
  |
  v
Artifactory
  |
  v
Docker/OCI Local
```

---

## 121. Enterprise

```text
                         Users / CI
                             |
                           DNS
                             |
                     Load Balancer
                             |
                       Artifactory
                             |
                +------------+------------+
                |                         |
            Virtual                    Local
                |                         |
          +-----+-----+                   |
          |           |                   |
        Local       Remote                |
                      |                   |
                      v                   |
                  Upstream                |
                                          |
                                   Release Artifact
                                          |
                                          v
                                      Production
```

---

## 122. Final Repository Selection Rules

```text
INTERNAL BUILD OUTPUT
→ LOCAL

EXTERNAL DEPENDENCY
→ REMOTE

MULTIPLE SOURCES FOR CONSUMERS
→ VIRTUAL

CI DEPENDENCY DOWNLOAD
→ VIRTUAL

CI ARTIFACT PUBLISH
→ LOCAL

KUBERNETES IMAGE PULL
→ LOCAL or approved VIRTUAL

DEVELOPER DEPENDENCY ACCESS
→ VIRTUAL

EXTERNAL REGISTRY ACCESS
→ REMOTE

PRODUCTION RELEASE
→ CONTROLLED LOCAL / RELEASE WORKFLOW
```

---

## 123. Production Checklist

```text
LOCAL
[ ] Correct package format
[ ] Correct owner
[ ] CI publish identity
[ ] Immutable release policy
[ ] Retention
[ ] Promotion
[ ] Delete restrictions

REMOTE
[ ] Approved upstream
[ ] HTTPS
[ ] Credentials if required
[ ] Cache policy
[ ] Security scanning
[ ] Upstream monitoring

VIRTUAL
[ ] Required local repositories
[ ] Required remote repositories
[ ] Correct resolution behavior
[ ] Stable endpoint
[ ] Consumer permissions
[ ] Change management

GLOBAL
[ ] TLS
[ ] RBAC
[ ] Audit
[ ] Monitoring
[ ] Capacity
[ ] Backup
[ ] DR
[ ] Documentation
```

---

## 124. Golden Rules

```text
1. Local owns internal artifacts.
2. Remote controls external dependency access.
3. Virtual simplifies consumer access.
4. Publish internal artifacts to the correct local repository.
5. Consume dependencies through virtual repositories where appropriate.
6. Treat virtual configuration as a production interface.
7. Do not treat remote cache as backup.
8. Restrict remote upstreams.
9. Use organization-controlled namespaces.
10. Use least-privilege service identities.
11. Keep runtime access read-only whenever possible.
12. Protect delete and administrative permissions.
13. Define retention before repository growth becomes a problem.
14. Monitor cache, storage and traffic.
15. Capacity-plan for CI and Kubernetes bursts.
16. Test repository changes before production rollout.
17. Preserve immutable production releases.
18. Document every production repository.
19. Assign owners.
20. Design the repository topology around security and lifecycle,
    not arbitrary team boundaries.
21. Validate exact behavior against the deployed Artifactory release,
    edition and package ecosystem.
```

---

# END OF 04-Local-Remote-and-Virtual-Repositories.md
