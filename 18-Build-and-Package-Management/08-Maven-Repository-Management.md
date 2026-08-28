# 18-Build-and-Package-Management
# 08-Maven-Repository-Management

## 1. Purpose

Maven repository management is the foundation of reliable dependency
consumption and artifact distribution in an enterprise DevOps
environment.

A repository is not simply a place where JAR files are stored.

It is part of the software supply chain:

```text
Developer / CI
      |
      v
Maven
      |
      v
Repository Layer
      |
      +--> dependency resolution
      +--> plugin resolution
      +--> artifact publishing
      +--> metadata
      +--> access control
      +--> caching
      +--> security
      |
      v
Build / Release
```

This file covers Maven repository fundamentals, repository types,
local/remote/internal repositories, Maven settings, mirrors,
authentication, Artifactory/Nexus architecture, dependency resolution,
metadata, snapshots, releases, repository layouts, CI/CD integration,
security, caching, troubleshooting, HA, backup, governance and
production architecture.

---

# PART I — REPOSITORY FUNDAMENTALS

## 2. What Is a Maven Repository?

A Maven repository is a location from which Maven can retrieve or to
which Maven can publish artifacts and metadata.

Artifacts can include:

```text
JAR
WAR
POM
sources JAR
javadoc JAR
plugins
plugin dependencies
metadata
```

---

## 3. Repository Responsibilities

A production repository platform commonly provides:

```text
storage
artifact retrieval
artifact publishing
metadata
authentication
authorization
caching
auditing
retention
replication
```

---

## 4. Why Repositories Matter to DevOps

Without reliable repository management:

```text
builds become dependent on public internet
CI becomes less reproducible
dependency downloads become slower
supply-chain risk increases
artifact governance becomes difficult
```

---

# PART II — TYPES OF MAVEN REPOSITORIES

## 5. Local Repository

The local Maven repository normally exists under:

```text
~/.m2/repository
```

It stores artifacts downloaded or installed locally.

---

## 6. Remote Repository

A remote repository is accessed over a network.

Examples include:

```text
company repository
Maven Central
vendor repository
```

---

## 7. Internal Repository

An enterprise repository is controlled by the organization.

Example:

```text
https://repo.company.example
```

The actual endpoint is organization-specific.

---

## 8. Local Repository in Repository Manager

Do not confuse:

```text
Maven local repository
```

with:

```text
Artifactory/Nexus local repository
```

They are different concepts.

```text
Developer ~/.m2
        |
        v
Enterprise repository manager
        |
        v
Stored artifacts
```

---

# PART III — REPOSITORY MANAGER

## 9. What Is a Repository Manager?

A repository manager provides a controlled interface for artifacts.

Typical capabilities:

```text
local repositories
remote repositories
virtual/group repositories
access control
caching
search
metadata
retention
```

Examples include enterprise platforms such as Artifactory and Nexus.

---

## 10. Why Use a Repository Manager?

Instead of:

```text
CI -> Internet -> Public repository
```

prefer:

```text
CI
 |
v
Enterprise Repository
 |
+--> internal artifacts
+--> approved external artifacts
```

---

# PART IV — LOCAL MAVEN REPOSITORY

## 11. Default Location

Typical:

```text
~/.m2/repository
```

---

## 12. Example Layout

```text
~/.m2/repository/
└── com/
    └── company/
        └── payment-core/
            └── 1.2.0/
                ├── payment-core-1.2.0.jar
                ├── payment-core-1.2.0.pom
                └── metadata...
```

---

## 13. Coordinates to Path

Given:

```text
com.company:payment-core:1.2.0
```

Maven maps it conceptually to:

```text
com/company/payment-core/1.2.0/
```

---

# PART V — REMOTE REPOSITORY

## 14. Remote Dependency

Example:

```text
Application
 |
v
Maven
 |
v
Internal Repository
 |
v
Remote Cache / Approved Upstream
```

---

## 15. Remote Cache

A repository manager can cache an upstream artifact.

Example:

```text
First request
    |
    v
Repository Manager
    |
    v
Public Upstream
    |
    v
Cache

Later requests
    |
    v
Repository Manager
    |
    v
Cached artifact
```

---

## 16. Benefits

```text
lower latency
reduced public traffic
better availability
centralized control
```

---

# PART VI — VIRTUAL / GROUP REPOSITORY

## 17. Virtual Repository

A virtual repository presents multiple repositories through one logical
endpoint.

Concept:

```text
                Maven
                  |
                  v
             Virtual Repo
              /         \
             v           v
          Local        Remote
```

---

## 18. Why Virtual Repositories?

Developers and CI can use one URL while the repository manager handles
the underlying sources.

Benefits:

```text
simpler configuration
centralized governance
easy migration
controlled upstream access
```

---

# PART VII — MAVEN SETTINGS

## 19. settings.xml

Maven settings commonly configure machine/user-specific behavior.

Common locations include:

```text
~/.m2/settings.xml
```

and Maven installation-level settings.

---

## 20. Typical settings Responsibilities

```text
mirrors
servers
profiles
repositories
pluginRepositories
authentication
```

---

## 21. POM vs settings.xml

Use the POM primarily for project configuration.

Use settings for environment/user-specific configuration.

```text
POM
 |
project configuration

settings.xml
 |
machine / user / CI configuration
```

---

# PART VIII — MIRRORS

## 22. Mirror

A mirror redirects repository access through another repository.

Concept:

```text
Maven
 |
v
Mirror
 |
v
Enterprise Repository
```

---

## 23. Enterprise Mirror

Typical architecture:

```text
Developer
    |
CI  |
    v
Corporate Maven Mirror
    |
    +--> Internal Local
    |
    +--> Approved Remote
```

---

## 24. Why Mirrors?

Benefits:

```text
centralized access
caching
security policy
availability
auditability
```

---

# PART IX — MIRROR CONFIGURATION

## 25. Conceptual Example

```xml
<mirrors>
    <mirror>
        <id>company-mirror</id>
        <mirrorOf>*</mirrorOf>
        <url>https://repo.company.example/maven/virtual</url>
    </mirror>
</mirrors>
```

The exact mirror policy should be designed according to enterprise
requirements.

---

## 26. mirrorOf

A broad mirror such as:

```text
*
```

can route repository access through the corporate repository.

Understand exceptions and mirror matching rules before deploying a
global configuration.

---

# PART X — AUTHENTICATION

## 27. Repository Authentication

Maven can use credentials associated with a server ID.

Concept:

```xml
<server>
    <id>company-releases</id>
    <username>...</username>
    <password>...</password>
</server>
```

---

## 28. Server ID

The server ID must correspond to the repository configuration that needs
the credentials.

Example:

```text
distributionManagement
        |
        +--> id: company-releases

settings.xml
        |
        +--> server id: company-releases
```

---

## 29. CI Authentication

Prefer:

```text
CI secret store
service identity
short-lived/scoped token
```

over long-lived human passwords.

---

# PART XI — AUTHORIZATION

## 30. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

---

## 31. Example

```text
CI identity
 |
+--> read dependencies
+--> deploy releases
+--> no repository administration
```

Use least privilege.

---

# PART XII — DEPENDENCY RESOLUTION

## 32. Resolution Flow

Conceptually:

```text
POM
 |
v
Local Repository
 |
+--> artifact available?
|       |
|       +--> yes -> use
|
+--> no
      |
      v
Configured Repository
      |
      v
Artifact
```

The exact repository selection behavior depends on Maven settings,
POM repositories and mirror configuration.

---

## 33. Resolution Failure

If Maven cannot resolve a required dependency:

```text
compile
   |
   X
```

The build cannot normally continue successfully.

---

# PART XIII — REPOSITORY PRIORITY

## 34. Do Not Guess

When a repository issue occurs, inspect:

```text
effective POM
effective settings
mirror configuration
repository definitions
```

Useful commands:

```bash
mvn help:effective-pom
mvn help:effective-settings
```

---

# PART XIV — ARTIFACT LAYOUT

## 35. Maven Repository Layout

Maven uses a predictable path structure based on coordinates.

Concept:

```text
groupId
   |
artifactId
   |
version
   |
artifact
```

---

## 36. Example

Coordinates:

```text
com.company:payment-api:3.0.0
```

Conceptual path:

```text
com/company/payment-api/3.0.0/
```

---

# PART XV — POM ARTIFACT

## 37. Why Store POM?

A Maven artifact normally has metadata describing:

```text
coordinates
dependencies
parent
packaging
```

The POM itself is an important repository artifact.

---

# PART XVI — METADATA

## 38. Maven Metadata

Repository metadata can help Maven understand versions and snapshot
information.

Examples include:

```text
maven-metadata.xml
```

---

## 39. Metadata Importance

Metadata can support:

```text
version discovery
snapshot resolution
repository operations
```

Corrupt or inconsistent metadata can cause confusing resolution
behavior.

---

# PART XVII — RELEASE REPOSITORY

## 40. Release Artifact

Example:

```text
1.4.2
```

A release repository should normally treat released artifacts as
immutable.

---

## 41. Release Principle

```text
1.4.2
 |
v
published
 |
v
do not silently replace
```

This protects reproducibility.

---

# PART XVIII — SNAPSHOT REPOSITORY

## 42. Snapshot

Example:

```text
1.5.0-SNAPSHOT
```

Snapshot artifacts are intended for development and can change.

---

## 43. Snapshot vs Release

```text
SNAPSHOT
 |
mutable development artifact

RELEASE
 |
immutable production artifact
```

---

# PART XIX — SNAPSHOT MANAGEMENT

## 44. Snapshot Cleanup

A repository should have retention policies for snapshots.

Example:

```text
keep recent N builds
remove older snapshots
```

The exact policy should match recovery and development requirements.

---

# PART XX — ARTIFACT PUBLISHING

## 45. Maven Deploy

Typical:

```bash
mvn deploy
```

The project must have appropriate remote repository configuration.

---

## 46. Distribution Management

Example:

```xml
<distributionManagement>
    <repository>
        <id>company-releases</id>
        <url>https://repo.company.example/releases</url>
    </repository>

    <snapshotRepository>
        <id>company-snapshots</id>
        <url>https://repo.company.example/snapshots</url>
    </snapshotRepository>
</distributionManagement>
```

---

# PART XXI — RELEASE PIPELINE

## 47. Production Flow

```text
Git
 |
v
CI
 |
v
Maven
 |
v
clean verify
 |
v
Security
 |
v
Package
 |
v
Deploy
 |
v
Release Repository
```

---

# PART XXII — BUILD ONCE

## 48. Artifact Promotion

Prefer:

```text
Build
 |
v
Artifact
 |
v
Repository
 |
+--> DEV
+--> STAGE
+--> PROD
```

rather than:

```text
DEV build
STAGE build
PROD build
```

---

# PART XXIII — ARTIFACT TRACEABILITY

## 49. Traceability

Associate:

```text
Git commit
build number
Maven version
artifact checksum
SBOM
container image
deployment
```

---

## 50. Why?

During an incident you should be able to answer:

```text
Which source produced this artifact?
Which dependencies were included?
Which build created it?
Where is it deployed?
```

---

# PART XXIV — CHECKSUMS AND INTEGRITY

## 51. Artifact Integrity

Repository ecosystems use checksums and metadata to help validate
artifact integrity.

Do not ignore checksum errors.

---

## 52. Checksum Failure

Possible causes:

```text
corrupt download
proxy issue
repository inconsistency
tampered artifact
incomplete transfer
```

Investigate before bypassing validation.

---

# PART XXV — REPOSITORY SECURITY

## 53. HTTPS

Use encrypted repository communication.

Avoid insecure plain HTTP in production unless there is a controlled,
explicit reason and compensating architecture.

---

## 54. Least Privilege

Separate identities:

```text
developer
CI reader
CI publisher
repository administrator
```

---

## 55. Read vs Write

Most builds require:

```text
read
```

Only publishing pipelines should normally require:

```text
write
```

---

# PART XXVI — CI REPOSITORY IDENTITY

## 56. CI Service Account

Use a dedicated identity.

Example:

```text
jenkins-maven-publisher
```

or equivalent workload identity.

Avoid using a developer's personal repository account.

---

# PART XXVII — TOKEN ROTATION

## 57. Credential Rotation

Repository credentials should support:

```text
rotation
revocation
expiry
auditing
```

---

# PART XXVIII — REPOSITORY AVAILABILITY

## 58. Repository Is Build Infrastructure

If the repository is unavailable:

```text
many pipelines
     |
     v
dependency resolution failure
```

Therefore repository availability is part of CI reliability.

---

## 59. Local Cache Limitation

A local Maven cache can reduce impact but cannot guarantee every
required artifact is available.

---

# PART XXIX — HIGH AVAILABILITY

## 60. HA Concept

Production repository architecture may use:

```text
Load Balancer
      |
      v
+-----+-----+
|           |
v           v
Repo A    Repo B
|           |
+-----+-----+
      |
      v
Shared / replicated storage
```

The exact topology depends on the repository platform.

---

## 61. Database

Some enterprise repository platforms rely on databases and storage
systems that also require availability planning.

---

# PART XXX — BACKUP

## 62. Backup

Back up critical repository data according to business requirements.

Consider:

```text
artifact storage
metadata
database
configuration
security configuration
```

---

## 63. Restore Testing

A backup is not sufficient.

Perform restore tests.

```text
Backup
 |
v
Restore
 |
v
Validate artifacts
 |
v
Validate metadata
 |
v
Validate publishing/reading
```

---

# PART XXXI — DISASTER RECOVERY

## 64. DR Architecture

```text
Primary Repository
        |
        v
Replication / Backup
        |
        v
DR Repository
```

---

## 65. RPO and RTO

Define:

```text
RPO = acceptable data loss
RTO = acceptable recovery time
```

Repository DR should have explicit targets.

---

# PART XXXII — REPOSITORY RETENTION

## 66. Why Retention?

Repositories can grow rapidly.

Retention helps control:

```text
storage
cost
search performance
operational complexity
```

---

## 67. Retention Categories

Different policies for:

```text
release
snapshot
temporary
build artifacts
```

---

# PART XXXIII — DO NOT DELETE RELEASES CARELESSLY

## 68. Release Retention

Before deleting old releases consider:

```text
running applications
rollback requirements
audit requirements
compliance
dependency consumers
```

---

# PART XXXIV — REPOSITORY POLICIES

## 69. Governance

Enterprise repository policy can define:

```text
approved upstreams
artifact naming
release immutability
snapshot retention
permissions
security scans
promotion
```

---

# PART XXXV — REMOTE REPOSITORY GOVERNANCE

## 70. Approved Upstreams

Do not allow unrestricted external repository access from every CI
runner.

Prefer:

```text
Approved upstreams
       |
       v
Repository manager
       |
       v
CI
```

---

# PART XXXVI — SUPPLY CHAIN

## 71. Supply-Chain Model

```text
Public Dependency
 |
v
Approved Remote
 |
v
Enterprise Cache
 |
v
Maven Build
 |
v
Application Artifact
 |
v
Production
```

---

## 72. Risk Points

```text
malicious package
compromised upstream
dependency confusion
credential theft
repository compromise
build compromise
```

---

# PART XXXVII — ARTIFACT PROMOTION

## 73. Promotion

A validated artifact can move logically through environments:

```text
candidate
   |
   v
staging
   |
   v
production
```

The artifact itself should remain unchanged.

---

# PART XXXVIII — MAVEN REPOSITORY + ARTIFACTORY

## 74. Reference

```text
                    Maven
                      |
                      v
               Artifactory Virtual
                  /          \
                 v            v
       Company Local      Approved Remote
             |                  |
             v                  v
       Internal JARs       Cached External
```

---

## 75. Publishing

```text
Maven deploy
      |
      v
Artifactory Local Release
```

---

# PART XXXIX — MAVEN REPOSITORY + NEXUS

## 76. General Architecture

The same general principles apply to other repository managers:

```text
Maven
 |
v
Repository Manager
 |
+--> internal
+--> external proxy
+--> group/virtual
```

The configuration syntax and product capabilities differ.

---

# PART XL — LOCAL CACHE TROUBLESHOOTING

## 77. Corrupted Artifact

Symptoms:

```text
checksum error
zip error
invalid POM
```

Investigate the local cached copy and repository copy.

---

## 78. Controlled Cleanup

Do not blindly delete all of:

```text
~/.m2/repository
```

on every failure.

Target the affected artifact when possible.

---

# PART XLI — DEPENDENCY NOT FOUND

## 79. Checklist

```text
groupId
artifactId
version
packaging/type
repository URL
mirror
credentials
artifact existence
network
DNS
TLS
```

---

# PART XLII — 401 UNAUTHORIZED

## 80. Meaning

Usually indicates authentication failure.

Check:

```text
credential
token
server ID
expired secret
```

---

# PART XLIII — 403 FORBIDDEN

## 81. Meaning

Usually indicates the identity is authenticated but not authorized for
the requested operation.

Check:

```text
repository permission
path permission
deploy permission
token scope
```

---

# PART XLIV — 404 NOT FOUND

## 82. Possible Causes

```text
wrong coordinates
wrong repository
artifact not published
mirror configuration
repository routing
```

Do not assume every 404 is a missing artifact.

---

# PART XLV — 409 CONFLICT

## 83. Possible Cause

A repository may reject an operation because of repository policies,
such as attempting to overwrite an immutable release.

---

# PART XLVI — 5XX ERRORS

## 84. Server Failure

Investigate:

```text
repository health
load balancer
database
storage
network
upstream
```

---

# PART XLVII — TLS ERRORS

## 85. TLS Investigation

Check:

```text
certificate
trust store
hostname
proxy
JDK
corporate CA
```

---

# PART XLVIII — DNS

## 86. DNS

If CI cannot resolve:

```text
repo.company.example
```

check:

```text
DNS
routing
network policy
proxy
```

---

# PART XLIX — PROXY

## 87. Corporate Proxy

A CI environment may require proxy configuration.

Consider:

```text
Maven settings
JVM properties
CI runner
repository manager
```

Use one consistent enterprise networking model.

---

# PART L — OFFLINE BUILDS

## 88. Offline Mode

```bash
mvn -o verify
```

Useful when all required artifacts exist locally.

---

## 89. Offline Limitation

If an artifact is absent locally:

```text
offline build
     |
     X
dependency unavailable
```

---

# PART LI — CACHE STRATEGY

## 90. Local Maven Cache

```text
CI Runner
 |
v
~/.m2/repository
```

---

## 91. Repository Manager Cache

```text
CI
 |
v
Enterprise Repository
 |
v
Remote Cache
```

The two caching layers can coexist.

---

# PART LII — CACHE INVALIDATION

## 92. Why Cache Problems Are Difficult

A build may succeed because an old artifact exists in cache even though
the upstream or repository configuration is broken.

Therefore compare:

```text
warm cache
cold cache
```

during troubleshooting.

---

# PART LIII — PRODUCTION CI DESIGN

## 93. CI Reader

For normal build:

```text
CI
 |
v
Read-only repository access
```

---

## 94. CI Publisher

For release:

```text
Release Pipeline
 |
v
Publisher identity
 |
v
Release Repository
```

Separate these permissions where practical.

---

# PART LIV — JENKINS

## 95. Jenkins Flow

```text
Jenkins
 |
v
Maven Wrapper
 |
v
settings.xml
 |
v
Enterprise Repository
 |
v
Dependencies
 |
v
Build
 |
v
Deploy
```

---

## 96. Credentials

Use Jenkins credentials or an equivalent secret mechanism rather than
hard-coded POM credentials.

---

# PART LV — GITHUB ACTIONS

## 97. Workflow

```text
GitHub Actions
 |
v
JDK
 |
v
Maven Wrapper
 |
v
Repository credentials
 |
v
Enterprise Maven Repository
```

---

# PART LVI — MULTI-MODULE REPOSITORY

## 98. Reactor + Repository

```text
Maven Reactor
 |
+--> module A
+--> module B
+--> module C
 |
v
Publish
 |
v
Repository
```

The reactor can satisfy inter-module dependencies without requiring
each module to be separately installed first.

---

# PART LVII — REPOSITORY AND CONTAINERS

## 99. Application Artifact

```text
Maven
 |
v
JAR
 |
v
Docker Build
 |
v
Container Registry
```

The Maven repository and container registry serve different artifact
types.

---

# PART LVIII — REPOSITORY AND KUBERNETES

## 100. Runtime Flow

```text
Maven Repository
 |
v
JAR
 |
v
Container Image
 |
v
Image Registry
 |
v
GitOps
 |
v
Kubernetes
```

---

# PART LIX — REPOSITORY SECURITY CONTROLS

## 101. Recommended Controls

```text
HTTPS
RBAC
least privilege
token rotation
audit logs
approved upstreams
artifact immutability
security scanning
retention
backup
DR
```

---

# PART LX — REPOSITORY MONITORING

## 102. Monitor

Track:

```text
availability
latency
storage
error rate
download rate
upload rate
cache hit ratio
authentication failures
```

---

## 103. Alerts

Examples:

```text
repository unavailable
storage nearly full
high 5xx rate
authentication failures spike
cache failure rate increases
```

---

# PART LXI — STORAGE

## 104. Capacity Planning

Track:

```text
current usage
growth rate
retention
snapshot growth
backup storage
```

---

## 105. Storage Failure

A full repository filesystem can cause:

```text
publishing failure
metadata failure
application degradation
```

Monitor before reaching capacity.

---

# PART LXII — PRODUCTION ARCHITECTURE

## 106. Enterprise Reference

```text
                         Internet
                            |
                   Approved Upstreams
                            |
                            v
                    +----------------+
                    | Repository     |
                    | Manager        |
                    |                |
                    | Remote Cache   |
                    | Local Releases |
                    | Local Snapshots|
                    | Virtual Repo   |
                    +-------+--------+
                            |
                       Load Balancer
                            |
             +--------------+--------------+
             |                             |
             v                             v
          Repo Node A                   Repo Node B
             |                             |
             +--------------+--------------+
                            |
                     Database / Storage
                            |
                     Backup / DR
                            |
                            v
                        Recovery
```

---

# PART LXIII — PRODUCTION DEPENDENCY FLOW

## 107. Dependency Flow

```text
Developer / CI
      |
      v
Maven
      |
      v
Virtual Repository
      |
      +--> Internal Local
      |
      +--> Approved Remote
              |
              v
          Upstream
```

---

# PART LXIV — PRODUCTION RELEASE FLOW

## 108. Release

```text
Git
 |
v
CI
 |
v
Maven clean verify
 |
v
Security
 |
v
Package
 |
v
Deploy
 |
v
Release Repository
 |
v
Promotion
 |
v
Production
```

---

# PART LXV — BACKUP AND DR CHECKLIST

## 109. Backup

```text
[ ] artifact storage
[ ] database
[ ] metadata
[ ] configuration
[ ] permissions
```

## 110. DR

```text
[ ] RPO defined
[ ] RTO defined
[ ] replication/backup
[ ] restore procedure
[ ] restore testing
[ ] DNS/load balancer plan
[ ] credential recovery
```

---

# PART LXVI — TROUBLESHOOTING PLAYBOOK

## 111. Build Cannot Download Dependency

Step 1:

```text
Read exact error
```

Step 2:

```text
verify coordinates
```

Step 3:

```text
check effective settings
```

Step 4:

```text
check mirror
```

Step 5:

```text
check repository
```

Step 6:

```text
check authentication
```

Step 7:

```text
check network/DNS/TLS
```

Step 8:

```text
test with controlled cache
```

---

## 112. Build Cannot Publish

Check:

```text
distributionManagement
server ID
credentials
permissions
release/snapshot repository
version policy
repository availability
```

---

## 113. Everyone's Builds Fail

Strong suspects:

```text
repository outage
DNS
TLS
network
mirror
authentication service
```

Do not immediately change every project's POM.

---

## 114. Only One Developer Fails

Strong suspects:

```text
local ~/.m2
settings.xml
JDK
proxy
local credentials
local network
```

---

## 115. Only CI Fails

Check:

```text
CI settings
CI secrets
runner network
Maven/JDK
mirror
repository permissions
cache
```

---

# PART LXVII — SECURITY INCIDENT

## 116. Compromised Credential

Response:

```text
revoke credential
 |
v
rotate
 |
v
audit usage
 |
v
inspect published artifacts
 |
v
check repository access
 |
v
validate CI
```

---

## 117. Malicious Artifact

Response:

```text
identify artifact
 |
v
identify consumers
 |
v
block/remove according to policy
 |
v
replace with trusted version
 |
v
rebuild
 |
v
scan
 |
v
redeploy
```

Follow organizational incident-response procedures.

---

# PART LXVIII — GOVERNANCE

## 118. Repository Governance

Define:

```text
who can publish
who can delete
who can administer
which upstreams are allowed
which artifacts require scanning
how snapshots are retained
how releases are promoted
```

---

# PART LXIX — PRODUCTION CHECKLIST

## 119. Repository

```text
[ ] enterprise repository manager
[ ] virtual repository
[ ] local repository
[ ] approved remote repositories
[ ] controlled mirrors
```

## 120. Security

```text
[ ] HTTPS
[ ] RBAC
[ ] least privilege
[ ] service identities
[ ] credential rotation
[ ] audit logs
```

## 121. Reliability

```text
[ ] HA
[ ] monitoring
[ ] storage capacity
[ ] backup
[ ] DR
[ ] restore testing
```

## 122. Build

```text
[ ] Maven Wrapper
[ ] controlled settings
[ ] dependency cache
[ ] plugin resolution
[ ] reproducible builds
```

## 123. Release

```text
[ ] release immutability
[ ] snapshot retention
[ ] artifact promotion
[ ] provenance
[ ] SBOM
```

---

# PART LXX — INTERVIEW PREPARATION

## 124. What Is a Maven Repository?

Answer:

```text
A Maven repository stores and serves Maven artifacts and metadata.
It can be local to a developer, remote, or managed by an enterprise
repository platform.
```

## 125. Local Repository vs Remote Repository?

Answer:

```text
The local Maven repository is normally the developer or CI cache under
~/.m2/repository. A remote repository is accessed over the network and
can provide internal artifacts or approved external dependencies.
```

## 126. Why Use Artifactory/Nexus?

Answer:

```text
They provide centralized artifact storage, caching, access control,
repository aggregation, governance and better supply-chain control.
```

## 127. What Is a Virtual Repository?

Answer:

```text
It provides one logical endpoint over multiple underlying local and
remote repositories, simplifying Maven configuration and centralizing
repository policy.
```

## 128. What Is a Maven Mirror?

Answer:

```text
A mirror redirects Maven repository requests to another repository
endpoint. Enterprises commonly use mirrors to route dependency and
plugin access through a controlled internal repository.
```

## 129. How Do You Secure Maven Repository Access?

Answer:

```text
I use HTTPS, RBAC, least-privilege service identities, protected CI
secrets, token rotation, approved upstreams, artifact scanning and
audit logging.
```

## 130. Release vs Snapshot?

Answer:

```text
A release is intended to be immutable and production-consumable.
A snapshot is a development artifact that may change and should have
separate retention and repository policies.
```

## 131. How Do You Troubleshoot a 401?

Answer:

```text
I check credentials, token validity, server ID mapping, secret
injection and authentication configuration.
```

## 132. How Do You Troubleshoot a 403?

Answer:

```text
I check authorization, repository permissions, path permissions and
token scopes because the identity may be authenticated but not
authorized.
```

## 133. How Do You Troubleshoot a 404?

Answer:

```text
I verify coordinates, repository routing, mirror configuration and
whether the artifact exists in the expected repository.
```

## 134. How Do You Troubleshoot Repository-Wide Failures?

Answer:

```text
I check repository health, load balancer, DNS, TLS, network,
authentication services, storage and upstream connectivity before
changing application POMs.
```

---

# PART LXXI — SENIOR-LEVEL SCENARIOS

## 135. Maven Central Is Temporarily Unavailable

Answer:

```text
I would rely on the enterprise repository's cached artifacts where
possible, verify whether required artifacts are already available,
avoid uncontrolled direct access from CI, and investigate repository
availability and upstream cache strategy.
```

## 136. Repository Storage Is 95% Full

Answer:

```text
I would assess growth rate and identify snapshots, temporary artifacts
and retention opportunities. I would expand capacity if necessary and
apply retention policies carefully. I would not delete production
releases without checking rollback, audit and consumer requirements.
```

## 137. All Jenkins Jobs Fail to Download Dependencies

Answer:

```text
Because the failure is broad, I would investigate shared
infrastructure first: repository health, DNS, network, TLS, mirror,
credentials and load balancer. I would not modify hundreds of POMs
until the shared path is proven healthy.
```

## 138. Developer Can Download but Cannot Deploy

Answer:

```text
This strongly suggests a write authorization or publishing
configuration issue. I would compare read and deploy permissions,
distributionManagement, server ID, token scope and target repository
policy.
```

## 139. Release Artifact Was Accidentally Overwritten

Answer:

```text
I would immediately determine which builds and environments consumed
the artifact, preserve evidence, review repository permissions and
restore immutability controls. If the artifact identity is no longer
trustworthy, I would create a new release version rather than silently
replacing the same production coordinate.
```

## 140. Repository Is Down During Production Incident

Answer:

```text
I would first determine whether running workloads are affected or only
new builds/deployments. Running applications may continue because their
runtime artifacts are already deployed. For new releases I would use
the repository DR/failover strategy and restore service according to
the defined RTO.
```

## 141. Cache Hides an Upstream Failure

Answer:

```text
That is one of the benefits of a repository cache, but it can hide
upstream availability problems. I would distinguish warm-cache and
cold-cache behavior and monitor upstream/cache health separately.
```

---

# PART LXXII — GOLDEN RULES

## 142. Rules

```text
1. Treat Maven repositories as critical CI/CD infrastructure.

2. Understand local, remote, local-managed and virtual repositories.

3. Do not confuse ~/.m2/repository with an enterprise repository's
   local repository.

4. Use enterprise repository management for controlled dependency
   access.

5. Prefer a virtual repository for a stable enterprise Maven endpoint.

6. Use approved remote repositories.

7. Control upstream access.

8. Understand Maven settings.xml.

9. Separate project configuration from environment configuration.

10. Understand mirrors.

11. Verify server IDs when authentication fails.

12. Use least-privilege credentials.

13. Never commit repository passwords.

14. Prefer protected CI secrets or workload identities.

15. Rotate credentials.

16. Audit repository access.

17. Use HTTPS.

18. Understand Maven repository layout.

19. Understand Maven metadata.

20. Treat POM files as repository artifacts.

21. Keep release artifacts immutable.

22. Keep snapshots separate from releases.

23. Apply snapshot retention policies.

24. Do not casually delete releases.

25. Use artifact promotion rather than rebuilding.

26. Build once and promote the same artifact.

27. Track Git commit to artifact.

28. Track artifact to container image.

29. Track artifact to deployment.

30. Preserve checksums and provenance.

31. Investigate checksum failures instead of bypassing them.

32. Monitor repository availability.

33. Monitor storage capacity.

34. Monitor latency and error rates.

35. Monitor authentication failures.

36. Monitor cache behavior.

37. Design repository HA for business-critical environments.

38. Back up artifacts and metadata according to requirements.

39. Back up configuration and security configuration.

40. Test restores.

41. Define repository RPO.

42. Define repository RTO.

43. Maintain a DR strategy.

44. Test DR.

45. Use repository governance.

46. Separate developer, CI reader, CI publisher and administrator roles.

47. Do not use human accounts for automated publishing.

48. Use effective-settings when repository behavior is unclear.

49. Use effective-pom when repository definitions are unclear.

50. Use dependency:tree for dependency resolution issues.

51. Distinguish 401 from 403.

52. Treat 404 as a routing/coordinate/repository question, not only a
    missing-artifact question.

53. Investigate 5xx errors at the repository infrastructure layer.

54. Investigate TLS and DNS independently.

55. Understand corporate proxy behavior.

56. Use controlled caches.

57. Test warm-cache and cold-cache scenarios when troubleshooting.

58. Do not delete the entire ~/.m2 repository without reason.

59. Target corrupted artifacts where practical.

60. Separate Maven repositories from container registries.

61. Integrate Maven repositories with CI/CD securely.

62. Protect plugin resolution as well as application dependency
    resolution.

63. Scan dependencies and build supply-chain components where tooling
    supports it.

64. Generate SBOMs where required.

65. Review licenses.

66. Plan capacity before storage becomes critical.

67. Use retention to control repository growth.

68. Do not allow retention policies to destroy required rollback
    artifacts.

69. Treat repository outages as shared infrastructure incidents.

70. Validate exact repository-manager, Maven, JDK and CI behavior for
    the versions and architecture actually used in production.
```

---

# END OF 08-Maven-Repository-Management.md
