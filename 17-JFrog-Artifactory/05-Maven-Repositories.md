# 17-JFrog-Artifactory
# 05-Maven-Repositories

## 1. Purpose

This file covers Maven repositories in JFrog Artifactory from a DevOps and production perspective.

It covers:

- Maven repository architecture
- Maven coordinates
- POM files
- Local, remote and virtual Maven repositories
- Maven settings
- Authentication
- Publishing artifacts
- Dependency resolution
- SNAPSHOT and release repositories
- CI/CD integration
- Jenkins/GitHub Actions/GitLab concepts
- Build Info
- Artifact promotion
- Repository permissions
- Maven metadata
- Checksums
- Dependency caching
- Security
- Troubleshooting
- Production architecture
- Real-world scenarios
- Interview preparation

---

# PART I — MAVEN FUNDAMENTALS

## 2. What Is Maven?

Apache Maven is a build and dependency-management tool commonly used for Java applications.

It can:

```text
compile
test
package
publish
resolve dependencies
manage plugins
```

A typical Maven lifecycle is:

```text
validate
 ↓
compile
 ↓
test
 ↓
package
 ↓
verify
 ↓
install
 ↓
deploy
```

---

## 3. Maven and Artifactory

Maven can use Artifactory as:

```text
Dependency source
Artifact publication target
Repository proxy
Artifact management platform
```

Typical architecture:

```text
Maven
  |
  +---- Dependency Read
  |          |
  |          v
  |      maven-virtual
  |
  +---- Artifact Publish
             |
             v
        maven-local
```

---

## 4. Why Use Artifactory for Maven?

Without Artifactory:

```text
Every CI job
    ↓
Maven Central / external repositories
```

With Artifactory:

```text
Every CI job
    ↓
Artifactory
    ↓
Approved remote repositories
```

Benefits:

```text
centralized dependencies
caching
internal artifact storage
access control
auditability
promotion
traceability
reduced external dependency
```

---

## 5. Maven Artifact Coordinates

A Maven artifact is commonly identified by:

```text
groupId
artifactId
version
packaging
classifier
```

Example:

```text
groupId:
com.company.payment

artifactId:
payment-service

version:
4.2.1

packaging:
jar
```

---

## 6. Maven Coordinate Identity

The basic coordinate is:

```text
com.company.payment:
payment-service:
4.2.1
```

This identifies the artifact version within the Maven ecosystem.

---

## 7. Maven Repository Path

Conceptually:

```text
groupId:
com.company.payment

becomes:

com/company/payment
```

Then:

```text
payment-service
4.2.1
```

may result in a layout similar to:

```text
com/company/payment/
  payment-service/
    4.2.1/
      payment-service-4.2.1.jar
      payment-service-4.2.1.pom
```

The exact storage representation is managed by Artifactory.

---

## 8. POM File

The `pom.xml` is the central Maven project configuration file.

It can define:

```text
project coordinates
dependencies
plugins
repositories
build configuration
properties
profiles
distribution management
```

---

## 9. Example POM

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company.payment</groupId>
    <artifactId>payment-service</artifactId>
    <version>4.2.1</version>

    <packaging>jar</packaging>

    <dependencies>
        ...
    </dependencies>
</project>
```

---

## 10. Maven Dependencies

Example:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <version>...</version>
</dependency>
```

Maven resolves the dependency from configured repositories.

---

## 11. Maven Local Repository on Developer Machine

Maven also has a local filesystem cache, commonly:

```text
~/.m2/repository
```

This is different from an Artifactory local repository.

Important distinction:

```text
~/.m2/repository
    =
developer/CI machine cache

Artifactory maven-local
    =
server-side organization repository
```

---

## 12. Maven Resolution Flow

Conceptually:

```text
Maven
 ↓
Local ~/.m2 cache
 ↓
Configured Artifactory endpoint
 ↓
Virtual Repository
 ↓
Local / Remote
 ↓
Upstream if required
```

If the dependency is already in `~/.m2`, Maven may not contact Artifactory for that dependency.

---

## 13. Maven Remote Repository

A remote Artifactory Maven repository can proxy an upstream such as Maven Central.

```text
Maven
 ↓
maven-virtual
 ↓
maven-central-remote
 ↓
Maven Central
```

---

## 14. Maven Local Repository

Internal company artifacts are commonly published to:

```text
maven-local
```

Example:

```text
payment-service-4.2.1.jar
```

---

## 15. Maven Virtual Repository

Consumers can use:

```text
maven-virtual
```

which may aggregate:

```text
maven-local
maven-central-remote
approved-third-party-remote
```

---

# PART II — MAVEN REPOSITORY ARCHITECTURE

## 16. Recommended Maven Architecture

```text
                    Maven Clients
                         |
                         v
                   maven-virtual
                    /          \
                   /            \
                  v              v
           maven-local     maven-central-remote
                                 |
                                 v
                            Maven Central
```

---

## 17. CI Dependency Resolution

```text
Jenkins
   |
   v
Maven
   |
   v
maven-virtual
   |
   +--> internal artifacts
   |
   +--> cached external dependencies
```

---

## 18. CI Artifact Publication

```text
Jenkins
  |
  v
mvn deploy
  |
  v
maven-local
  |
  v
payment-service-4.2.1.jar
```

---

## 19. Why Separate Read and Publish Paths?

This provides a clean security model:

```text
Read:
maven-virtual

Write:
maven-local
```

CI does not need write access to every repository.

---

## 20. Repository Naming

Typical:

```text
maven-local
maven-central-remote
maven-virtual
```

For multiple domains:

```text
payments-maven-local
platform-maven-local
```

Keep naming standardized.

---

## 21. Maven Release Repository

A release repository can contain:

```text
4.1.0
4.2.0
4.2.1
```

Release versions should be treated as immutable.

---

## 22. Maven Snapshot Repository

A development repository can contain:

```text
4.3.0-SNAPSHOT
```

SNAPSHOTs are mutable development versions.

Do not use them as production release identifiers.

---

## 23. Release vs Snapshot

| Property | Release | SNAPSHOT |
|---|---|---|
| Example | 4.2.1 | 4.3.0-SNAPSHOT |
| Production use | Yes | Generally no |
| Mutability | Should be immutable | Can change |
| Lifecycle | Long | Shorter |
| Retention | Longer | More aggressive cleanup |

---

## 24. Why Separate SNAPSHOT and Release Repositories?

A separation can provide:

```text
different permissions
different retention
different lifecycle
different promotion controls
```

It is an architectural choice, not a universal requirement.

---

## 25. Maven Metadata

Maven uses metadata to understand available versions and artifact information.

Examples include:

```text
maven-metadata.xml
```

Metadata is especially relevant to:

```text
SNAPSHOT resolution
version discovery
repository operations
```

---

## 26. SNAPSHOT Metadata

A SNAPSHOT may be resolved to timestamped/build-specific content.

Conceptually:

```text
4.3.0-SNAPSHOT
       ↓
timestamp/build metadata
       ↓
specific artifact
```

This is one reason SNAPSHOT repositories should be treated differently from immutable release repositories.

---

## 27. Maven Checksums

Maven artifacts can have checksums such as:

```text
SHA-1
SHA-256
```

Checksums help verify artifact integrity.

Conceptually:

```text
Artifact
   ↓
Checksum
   ↓
Integrity verification
```

---

## 28. Why Checksums Matter

A checksum can help detect:

```text
corruption
unexpected content
transfer problems
tampering
```

It does not by itself prove that an artifact is trustworthy.

---

## 29. Maven POM + JAR

A Maven artifact commonly consists of:

```text
payment-service-4.2.1.jar
payment-service-4.2.1.pom
```

Additional files may include:

```text
sources
javadoc
checksums
metadata
```

---

# PART III — MAVEN SETTINGS

## 30. settings.xml

Maven commonly uses:

```text
~/.m2/settings.xml
```

for user-specific configuration.

It can contain:

```text
servers
mirrors
profiles
repositories
pluginRepositories
```

Sensitive credentials should be handled securely.

---

## 31. Maven Server Configuration

Conceptually:

```xml
<servers>
  <server>
    <id>company-artifactory</id>
    <username>...</username>
    <password>...</password>
  </server>
</servers>
```

The exact credential mechanism should follow the organization's security policy.

---

## 32. Never Hardcode Secrets in pom.xml

Avoid:

```xml
<password>production-password</password>
```

inside source-controlled files.

Better:

```text
CI secret
 ↓
Maven settings
 ↓
Artifactory
```

---

## 33. Maven Mirror

A Maven mirror can redirect repository requests to a centralized endpoint.

Enterprise pattern:

```text
Maven
 ↓
Artifactory Maven Virtual
```

This can reduce direct access to public repositories.

---

## 34. Mirror Configuration Concept

Conceptually:

```xml
<mirrors>
  <mirror>
    <id>company-artifactory</id>
    <url>https://artifactory.company.com/artifactory/maven-virtual/</url>
    <mirrorOf>*</mirrorOf>
  </mirror>
</mirrors>
```

The exact configuration should be validated against the organization's Maven repository strategy.

---

## 35. Why Use a Mirror?

Benefits:

```text
centralized dependency management
reduced direct internet access
consistent repository behavior
caching
governance
```

---

## 36. Maven Profiles

Profiles can configure environment-specific settings.

However, avoid using profiles to hide major differences in production artifact identity.

Prefer:

```text
same artifact
controlled configuration
```

rather than rebuilding different artifacts for each environment.

---

## 37. Distribution Management

Maven can define where artifacts are published.

Conceptually:

```xml
<distributionManagement>
  <repository>
    <id>company-releases</id>
    <url>...</url>
  </repository>

  <snapshotRepository>
    <id>company-snapshots</id>
    <url>...</url>
  </snapshotRepository>
</distributionManagement>
```

---

## 38. Release Publication

For a release version:

```text
mvn deploy
   ↓
release repository
```

---

## 39. Snapshot Publication

For:

```text
4.3.0-SNAPSHOT
```

Maven may publish to a snapshot repository.

---

## 40. CI Maven Configuration

A production pipeline should inject:

```text
repository URL
credentials/token
settings.xml
```

through secure CI mechanisms.

---

# PART IV — ARTIFACT PUBLICATION

## 41. Maven Install vs Deploy

Important distinction:

```text
mvn install
```

places the artifact into the machine's local Maven repository.

```text
mvn deploy
```

publishes the artifact to a remote repository configured for deployment.

---

## 42. Install Flow

```text
Source
 ↓
Build
 ↓
Package
 ↓
mvn install
 ↓
~/.m2/repository
```

This is not the same as publishing to Artifactory.

---

## 43. Deploy Flow

```text
Source
 ↓
Build
 ↓
Package
 ↓
mvn deploy
 ↓
Artifactory
```

---

## 44. CI Build Example

```bash
mvn clean verify
```

Then:

```bash
mvn deploy
```

A production pipeline should add:

```text
tests
security scans
quality gates
artifact validation
```

before release publication.

---

## 45. Maven CI Pipeline

```text
Checkout
   ↓
Compile
   ↓
Unit Tests
   ↓
Static Analysis
   ↓
Package
   ↓
Dependency/Security Scan
   ↓
Publish Artifact
   ↓
Build Info
   ↓
Promotion
```

---

## 46. Build Once

The preferred flow:

```text
Build artifact
      ↓
Test
      ↓
Store artifact
      ↓
Promote same artifact
      ↓
Production
```

Do not rebuild after approval unless there is a documented reason.

---

## 47. Artifact Coordinates Must Remain Stable

Once:

```text
com.company.payment:payment-service:4.2.1
```

is released, do not silently replace it.

---

## 48. Maven Artifact Promotion

Conceptually:

```text
maven-dev-local
       ↓
validated
       ↓
release
       ↓
production consumption
```

The exact promotion implementation depends on Artifactory configuration and organizational workflow.

---

## 49. Build Info

Build Info can capture:

```text
build name
build number
source control
dependencies
artifacts
environment
modules
timestamps
```

This improves traceability.

---

## 50. Build Provenance

A production artifact should answer:

```text
Which commit produced it?
Which pipeline produced it?
Which dependencies were used?
Which build number?
Which artifact version?
Which promotion path?
```

---

# PART V — MAVEN SECURITY

## 51. Maven Authentication

Possible enterprise mechanisms include:

```text
username/token
access token
OIDC/SSO-integrated workflows
CI-managed credentials
```

Use the mechanism supported by the environment.

---

## 52. CI Service Account

Example:

```text
jenkins-payment-maven
```

Permissions:

```text
READ  maven-virtual
DEPLOY payment-maven-local
DELETE none
ADMIN none
```

---

## 53. Token Rotation

Recommended flow:

```text
Create new token
 ↓
Update CI
 ↓
Test build
 ↓
Monitor
 ↓
Revoke old token
```

---

## 54. Least Privilege

A Maven build should not need:

```text
Artifactory admin
```

to run.

Grant only:

```text
dependency read
artifact deploy
```

where necessary.

---

## 55. Maven Dependency Security

Dependencies should be evaluated for:

```text
known vulnerabilities
license issues
malicious packages
unapproved versions
transitive dependencies
```

Artifactory can participate in a broader supply-chain security process.

---

## 56. Dependency Pinning

Avoid uncontrolled dependency versions.

Prefer:

```xml
<version>6.1.4</version>
```

rather than relying on ambiguous version ranges where reproducibility matters.

---

## 57. Transitive Dependencies

Maven dependencies can bring additional dependencies.

Example:

```text
Application
 ↓
Library A
 ↓
Library B
 ↓
Library C
```

Security scanning should account for transitive dependencies.

---

## 58. Dependency Tree

Use:

```bash
mvn dependency:tree
```

to inspect the dependency graph.

Useful for:

```text
conflicts
unexpected dependencies
vulnerable versions
```

---

## 59. Dependency Conflict

Example:

```text
Library A → log4j 2.x
Library B → log4j another version
```

Maven's dependency mediation can choose one version.

Verify the actual resolved dependency.

---

## 60. Effective POM

Use:

```bash
mvn help:effective-pom
```

to understand inherited and resolved Maven configuration.

Useful for troubleshooting:

```text
repositories
plugins
properties
profiles
parent configuration
```

---

# PART VI — MAVEN + JENKINS

## 61. Jenkins Architecture

```text
Developer
 ↓
Git
 ↓
Jenkins
 ↓
Maven
 ↓
Artifactory
```

---

## 62. Jenkins Dependency Read

```text
Jenkins
 ↓
Maven
 ↓
maven-virtual
 ↓
dependencies
```

---

## 63. Jenkins Artifact Publish

```text
Jenkins
 ↓
mvn deploy
 ↓
maven-local
```

---

## 64. Jenkins Credentials

Store credentials in:

```text
Jenkins Credentials
```

Then inject them into the build.

Do not hardcode:

```text
username
password
token
```

in Jenkinsfiles.

---

## 65. Jenkins Pipeline Concept

```groovy
pipeline {
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('Publish') {
            steps {
                sh 'mvn deploy'
            }
        }
    }
}
```

Production pipelines should include security and approval controls appropriate to the organization.

---

## 66. Jenkins + settings.xml

A secure pipeline can generate or provide a Maven settings file dynamically.

Conceptually:

```text
Jenkins Secret
      ↓
settings.xml
      ↓
Maven
      ↓
Artifactory
```

---

# PART VII — MAVEN + GITHUB ACTIONS

## 67. GitHub Actions Flow

```text
GitHub
 ↓
GitHub Actions
 ↓
Maven
 ↓
Artifactory
```

---

## 68. GitHub Actions Authentication

Use:

```text
GitHub Secrets
```

or an approved workload identity mechanism.

Do not commit Artifactory credentials into:

```text
pom.xml
settings.xml
workflow YAML
```

---

## 69. GitHub Actions Pipeline

Conceptually:

```yaml
steps:
  - checkout
  - setup-java
  - configure-maven
  - mvn clean verify
  - security scan
  - mvn deploy
```

---

# PART VIII — MAVEN + GITLAB

## 70. GitLab CI Flow

```text
GitLab
 ↓
GitLab Runner
 ↓
Maven
 ↓
Artifactory
```

---

## 71. GitLab Secret Handling

Use:

```text
CI/CD variables
protected variables
masked variables
```

according to the organization's security model.

---

## 72. GitLab Pipeline

Conceptually:

```yaml
build:
  script:
    - mvn clean verify

publish:
  script:
    - mvn deploy
```

Add:

```text
security
quality
approval
promotion
```

as appropriate.

---

# PART IX — MAVEN TROUBLESHOOTING

## 73. Troubleshooting Layers

Use:

```text
1. DNS
2. TCP
3. TLS
4. HTTP
5. Authentication
6. Authorization
7. Maven configuration
8. Repository configuration
9. Artifact coordinates
10. Upstream/cache
```

---

## 74. Maven 401

Likely:

```text
missing credentials
expired token
wrong credentials
wrong server ID
```

Check:

```text
settings.xml
server ID
CI secrets
token
```

---

## 75. Maven 403

Likely:

```text
authenticated
but unauthorized
```

Check:

```text
permission target
repository permissions
project access
deploy rights
```

---

## 76. Maven 404

Check:

```text
groupId
artifactId
version
repository URL
virtual repository
artifact existence
```

---

## 77. Maven 409

Check:

```text
release immutability
existing artifact
deployment policy
duplicate publication
```

---

## 78. Maven Timeout

Check:

```text
DNS
network
proxy
load balancer
Artifactory health
upstream
```

---

## 79. TLS Failure

Typical symptoms:

```text
PKIX path building failed
certificate errors
handshake failures
```

Check:

```text
certificate chain
hostname
truststore
expiration
proxy interception
```

---

## 80. Maven Cannot Resolve Dependency

Use:

```bash
mvn dependency:tree
```

and:

```bash
mvn -U clean verify
```

where appropriate.

Also inspect:

```text
settings.xml
effective POM
repository URL
Artifactory logs
```

---

## 81. SNAPSHOT Not Updating

Check:

```text
update policy
local Maven cache
snapshot metadata
repository configuration
```

A developer's `~/.m2` cache can make an old dependency appear to persist.

---

## 82. Force Dependency Update

Common Maven option:

```bash
mvn -U clean verify
```

`-U` requests checks for updated releases and snapshots.

Use it intentionally; it can increase repository traffic.

---

## 83. Artifact Published but Cannot Be Downloaded

Check:

```text
artifact path
repository
permissions
virtual repository inclusion
version
```

---

## 84. Artifact Upload Succeeds but Build Fails

Possible causes:

```text
publication target is correct
but consumer endpoint does not include repository
```

Example:

```text
Published:
maven-local

Consumer:
maven-virtual

Virtual does not include:
maven-local
```

---

## 85. Maven Plugin Resolution Failure

Maven also downloads build plugins.

Do not only configure dependencies.

Check:

```text
pluginRepositories
mirror
virtual repository
remote repository
```

---

## 86. Maven Central Works Locally but CI Fails

Possible reason:

```text
developer ~/.m2 cache
```

contains dependencies that CI does not have.

CI may reveal repository configuration problems.

---

## 87. CI Works but Production Build Fails

Possible differences:

```text
credentials
settings.xml
network
repository URL
JDK
Maven version
dependency cache
```

Standardize build environments.

---

# PART X — PRODUCTION MAVEN ARCHITECTURE

## 88. Recommended Production Pattern

```text
                       Developers / CI
                              |
                              v
                       maven-virtual
                         /        \
                        /          \
                       v            v
                maven-local   maven-remote
                                  |
                                  v
                            Maven Central
```

---

## 89. Production CI Pattern

```text
Git
 ↓
CI
 ↓
Maven
 ↓
Virtual
 ↓
Dependencies
 ↓
Build
 ↓
Tests
 ↓
Security
 ↓
Publish
 ↓
Local
 ↓
Promotion
 ↓
Production
```

---

## 90. Production Security Pattern

```text
                 CI Identity
                     |
          +----------+----------+
          |                     |
       READ                  DEPLOY
          |                     |
          v                     v
    maven-virtual         maven-local
          |
       Remote
          |
      Upstream
```

---

## 91. Production Artifact Lifecycle

```text
Source Commit
    ↓
CI Build
    ↓
Maven Artifact
    ↓
Artifactory Local
    ↓
Security Validation
    ↓
Approval
    ↓
Promotion
    ↓
Production
```

---

## 92. Production Immutability

Release:

```text
4.2.1
```

should map to one approved artifact.

Rollback:

```text
4.2.0
```

should remain available according to retention policy.

---

## 93. Maven Repository Capacity

Plan for:

```text
artifact count
artifact size
dependency cache
SNAPSHOT growth
release retention
CI concurrency
backup size
```

---

## 94. Maven Performance

Important factors:

```text
network latency
Artifactory latency
storage I/O
database health
Maven cache
dependency graph size
artifact size
CI concurrency
```

---

## 95. Build Burst

Suppose:

```text
300 Jenkins jobs
```

start together.

They may request:

```text
thousands of dependencies
plugins
POM metadata
artifacts
```

Artifactory and upstream capacity must handle the burst.

---

## 96. Maven Cache Strategy

Two caches can exist:

```text
CI ~/.m2
        +
Artifactory remote cache
```

They serve different purposes.

---

## 97. CI Cache vs Artifactory Cache

```text
CI ~/.m2
→ local to build environment

Artifactory Remote Cache
→ shared organizational cache
```

The shared cache reduces repeated upstream traffic across many builds.

---

## 98. Maven Repository Disaster Recovery

Recovery must preserve:

```text
artifact files
POMs
metadata
repository configuration
permissions
build information where required
```

Test restores.

---

## 99. Maven Repository Backup

Do not assume:

```text
filesystem copy alone
```

is a complete application backup.

Follow the supported JFrog backup strategy for the deployed architecture.

---

## 100. Maven Repository Monitoring

Monitor:

```text
request rate
latency
HTTP errors
storage
cache
upstream failures
authentication failures
```

---

## 101. Maven Security Monitoring

Look for:

```text
unexpected artifact publication
failed login attempts
mass downloads
deletions
permission changes
unknown clients
```

---

# PART XI — REAL-WORLD SCENARIOS

## 102. Scenario — Maven Central Down

```text
Maven
 ↓
maven-virtual
 ↓
maven-central-remote
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
cache frequently used dependencies
maintain reliable Artifactory
control critical dependencies
```

---

## 103. Scenario — CI Gets 401

Check:

```text
settings.xml
server ID
token
secret injection
```

---

## 104. Scenario — CI Gets 403

Check:

```text
permission target
deploy permission
repository
project
token scope
```

---

## 105. Scenario — CI Gets 404

Check:

```text
repository path
artifact
version
virtual membership
```

---

## 106. Scenario — SNAPSHOT Is Stale

Check:

```text
~/.m2
update policy
snapshot metadata
repository
```

---

## 107. Scenario — Artifact Cannot Be Promoted

Check:

```text
source repository
target repository
permissions
artifact status
release policy
build information
```

---

## 108. Scenario — Build Suddenly Uses Different Dependency

Investigate:

```text
dependency tree
virtual repository configuration
remote repository changes
version ranges
metadata
```

---

## 109. Scenario — Malicious Dependency Detected

Response:

```text
quarantine/deny affected dependency
identify consumers
identify builds
identify deployments
scan artifacts
revoke if necessary
rebuild from trusted sources
document incident
```

---

## 110. Scenario — Maven Repository Storage Full

Response:

```text
identify largest repositories
identify SNAPSHOT/cache growth
review retention
remove approved disposable content
increase capacity
implement forecasting
```

---

# PART XII — INTERVIEW QUESTIONS

## 111. What Is a Maven Repository in Artifactory?

Answer:

```text
It is an Artifactory repository configured for Maven artifacts and
metadata. It can be local for internally produced artifacts, remote
for external Maven sources, or virtual for a unified dependency
endpoint.
```

---

## 112. What Is the Difference Between mvn install and mvn deploy?

Answer:

```text
mvn install places the artifact in the machine's local Maven repository,
usually ~/.m2/repository. mvn deploy publishes the artifact to a
configured remote repository such as an Artifactory Maven local
repository.
```

---

## 113. Why Use a Maven Virtual Repository?

Answer:

```text
It gives developers and CI a single dependency endpoint while
Artifactory manages access to internal and approved external
repositories.
```

---

## 114. Where Do You Publish Internal Maven Artifacts?

Answer:

```text
I publish organization-owned build outputs to an appropriate Maven
local repository. Consumers normally read them through a Maven
virtual repository.
```

---

## 115. Why Separate SNAPSHOT and Release Artifacts?

Answer:

```text
SNAPSHOT artifacts are mutable development versions and usually need
shorter retention and different controls. Release artifacts should
be immutable and retained according to rollback and compliance
requirements.
```

---

## 116. How Do You Secure Maven CI?

Answer:

```text
I use dedicated CI identities, secret-managed credentials, least
privilege, read access to virtual repositories and deploy access only
to designated local repositories. I do not store Artifactory
credentials in source code.
```

---

## 117. How Do You Troubleshoot Maven 401?

Answer:

```text
I check settings.xml, the server ID referenced by the repository,
credential injection, token validity and the actual endpoint being
used by Maven.
```

---

## 118. How Do You Troubleshoot Maven 403?

Answer:

```text
I verify that authentication succeeded and then inspect repository
permissions, permission targets, project access and token scope.
```

---

## 119. How Do You Troubleshoot Maven 404?

Answer:

```text
I verify groupId, artifactId and version, then check the repository
URL, virtual repository membership and whether the artifact exists in
the underlying repository or upstream.
```

---

## 120. Why Does Maven Work on a Developer Laptop but Fail in CI?

Answer:

```text
The developer may already have dependencies in ~/.m2/repository.
CI may start with an empty cache, exposing incorrect repository,
authentication, mirror or network configuration.
```

---

## 121. How Do You Investigate a Dependency Conflict?

Answer:

```text
I use mvn dependency:tree, inspect the effective POM and determine
which dependency path introduces the conflicting version. Then I
apply the appropriate dependency-management strategy and verify the
resolved graph.
```

---

## 122. How Do You Handle Maven Central Outage?

Answer:

```text
I first determine whether the required dependencies are already
cached in Artifactory. If they are not cached, builds depending on
them may fail. For critical environments I reduce external
dependency risk through approved remotes, caching and controlled
dependency management.
```

---

## 123. What Is the Difference Between Artifactory Cache and ~/.m2?

Answer:

```text
~/.m2 is a local cache on a developer or CI machine. Artifactory's
remote cache is a shared repository-side cache that can serve many
consumers and reduce upstream traffic.
```

---

## 124. How Do You Design Maven Repositories for Large Organizations?

Answer:

```text
I standardize local, remote and virtual repository patterns, use
projects and RBAC for isolation, centralize external dependency
access, use virtual repositories for consumers, define ownership and
retention and avoid creating repositories without a real security or
lifecycle boundary.
```

---

## 125. How Do You Make Maven Builds Reproducible?

Answer:

```text
I use immutable release artifacts, pinned dependency versions,
controlled repositories, consistent Maven/JDK versions, build
provenance and promotion of the same artifact instead of rebuilding
for each environment.
```

---

# PART XIII — PRODUCTION CHECKLIST

## 126. Maven Repository Checklist

```text
REPOSITORIES
[ ] maven-local configured
[ ] maven-remote configured
[ ] maven-virtual configured
[ ] naming standardized
[ ] ownership documented

MAVEN
[ ] settings.xml controlled
[ ] mirror strategy defined
[ ] distributionManagement configured
[ ] dependency versions controlled
[ ] plugin resolution controlled

SECURITY
[ ] CI identity
[ ] token/credential management
[ ] least privilege
[ ] no hardcoded secrets
[ ] dependency scanning
[ ] namespace policy

CI/CD
[ ] build
[ ] test
[ ] scan
[ ] publish
[ ] Build Info
[ ] promotion

OPERATIONS
[ ] monitoring
[ ] logging
[ ] audit
[ ] capacity
[ ] retention
[ ] backup
[ ] restore testing
[ ] DR

RELIABILITY
[ ] upstream governance
[ ] remote cache
[ ] CI burst capacity
[ ] production immutability
[ ] rollback artifacts
```

---

# PART XIV — GOLDEN RULES

## 127. Rules

```text
1. Use Artifactory as the controlled Maven dependency boundary.

2. Use Maven Virtual for consumer dependency resolution where
   appropriate.

3. Publish internal artifacts to Maven Local.

4. Keep release artifacts immutable.

5. Treat SNAPSHOT artifacts as development content.

6. Do not confuse ~/.m2 with Artifactory storage.

7. Do not hardcode Artifactory credentials.

8. Use dedicated CI identities.

9. Apply least privilege.

10. Protect delete and administrative permissions.

11. Control external Maven repositories.

12. Use internal namespaces to reduce dependency confusion risk.

13. Pin important dependency versions.

14. Inspect transitive dependencies.

15. Use Build Info/provenance for production traceability.

16. Build once and promote the same artifact.

17. Monitor Artifactory and upstream dependency health.

18. Plan for CI dependency bursts.

19. Test repository and virtual-repository changes.

20. Backup and restore the complete required repository state.

21. Treat Artifactory Maven repositories as critical CI/CD
    infrastructure.

22. Validate exact syntax and behavior against the deployed Maven,
    Artifactory and JFrog versions before production rollout.
```

---

# END OF 05-Maven-Repositories.md
