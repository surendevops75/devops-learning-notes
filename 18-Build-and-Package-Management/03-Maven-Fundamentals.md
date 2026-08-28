# 18-Build-and-Package-Management
# 03-Maven-Fundamentals

## 1. Purpose

Apache Maven is one of the most widely used build and dependency
management tools in Java and JVM-based enterprise environments.

For a DevOps engineer, Maven knowledge goes beyond knowing:

```bash
mvn clean package
```

A production engineer should understand:

```text
Maven installation
Maven Wrapper
POM
coordinates
repositories
dependencies
plugins
lifecycle
profiles
properties
multi-module builds
testing
packaging
artifact publishing
CI/CD
Artifactory
Jenkins
GitHub Actions
security
caching
troubleshooting
```

This file establishes the foundation before the deeper files on
`pom.xml`, lifecycle, dependencies/plugins, multi-module projects and
CI/CD integration.

---

# PART I — MAVEN FUNDAMENTALS

## 2. What Is Maven?

Maven is a build automation and dependency management tool primarily
used for Java projects.

It can automate:

```text
dependency resolution
compilation
testing
packaging
verification
installation
deployment
```

A simplified flow:

```text
Source
  |
  v
Maven
  |
  +--> Dependencies
  +--> Compile
  +--> Test
  +--> Package
  |
  v
Artifact
```

---

## 3. Why Maven Is Important in DevOps

Maven provides a standardized build process.

Instead of every Java project inventing commands:

```text
compile.sh
build.sh
package.sh
deploy.sh
```

Maven provides a common lifecycle and project model.

This is valuable in enterprise CI/CD because build behavior becomes
more predictable and easier to standardize.

---

## 4. Maven's Core Responsibilities

Maven commonly handles:

```text
Project configuration
Dependency management
Build lifecycle
Plugin execution
Testing
Packaging
Artifact installation
Artifact deployment
```

---

# PART II — MAVEN PROJECT STRUCTURE

## 5. Standard Maven Layout

A conventional Maven project:

```text
my-app/
├── pom.xml
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   └── test/
│       ├── java/
│       └── resources/
└── target/
```

---

## 6. src/main/java

Production Java source code:

```text
src/main/java/
```

Example:

```text
src/main/java/com/company/payment/PaymentService.java
```

---

## 7. src/main/resources

Application resources:

```text
src/main/resources/
```

Examples:

```text
application.properties
application.yml
templates/
messages/
```

Do not place secrets in source-controlled resources.

---

## 8. src/test/java

Test source:

```text
src/test/java/
```

Examples:

```text
unit tests
integration tests
```

---

## 9. src/test/resources

Test-only resources:

```text
src/test/resources/
```

---

## 10. target/

Maven normally generates build outputs under:

```text
target/
```

Examples:

```text
target/classes/
target/test-classes/
target/surefire-reports/
target/app.jar
```

`target/` should normally not be committed to Git.

---

# PART III — POM

## 11. What Is pom.xml?

POM means:

```text
Project Object Model
```

The POM defines important project information and build configuration.

Typical elements include:

```text
groupId
artifactId
version
packaging
properties
dependencies
dependencyManagement
build
plugins
profiles
repositories
distributionManagement
```

The next file covers `pom.xml` in much greater depth.

---

## 12. Minimal POM

Example:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
         http://maven.apache.org/POM/4.0.0
         https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company</groupId>
    <artifactId>payment-service</artifactId>
    <version>1.0.0</version>

</project>
```

---

# PART IV — MAVEN COORDINATES

## 13. Maven Coordinates

An artifact is commonly identified by:

```text
groupId
artifactId
version
```

Example:

```text
com.company
payment-service
1.0.0
```

---

## 14. groupId

Identifies the organization or logical group.

Example:

```xml
<groupId>com.company.payments</groupId>
```

---

## 15. artifactId

Identifies the project/artifact.

Example:

```xml
<artifactId>payment-service</artifactId>
```

---

## 16. version

Identifies the artifact version.

Example:

```xml
<version>4.2.1</version>
```

---

## 17. Packaging

Common values include:

```text
jar
war
pom
```

Example:

```xml
<packaging>jar</packaging>
```

If omitted, Maven's default packaging is commonly `jar`.

---

# PART V — MAVEN REPOSITORIES

## 18. Repository Concept

Maven resolves dependencies from repositories.

Common sources:

```text
local repository
remote repository
Maven Central
enterprise repository
```

---

## 19. Local Repository

Maven maintains a local repository, commonly under:

```text
~/.m2/repository
```

It stores downloaded artifacts and metadata.

---

## 20. Example Local Repository

Concept:

```text
~/.m2/repository/
└── org/
    └── example/
        └── library/
            └── 1.2.3/
                ├── library-1.2.3.jar
                └── library-1.2.3.pom
```

---

## 21. Why Local Repository Exists

It reduces repeated downloads:

```text
First build
   |
   v
Download dependency
   |
   v
~/.m2/repository
```

Next build may reuse it.

---

## 22. Remote Repository

A remote repository provides artifacts that are not already available
locally.

Examples:

```text
Maven Central
Artifactory
Nexus
```

---

## 23. Enterprise Maven Repository

Typical enterprise architecture:

```text
Developer / CI
       |
       v
Artifactory Maven Virtual
       |
       +--> Internal Local
       |
       +--> Approved Remote
```

---

# PART VI — MAVEN SETTINGS

## 24. settings.xml

Maven user-specific and environment-specific configuration can be
provided through:

```text
~/.m2/settings.xml
```

A CI environment can also provide its own Maven settings.

---

## 25. What Belongs in settings.xml?

Common examples:

```text
mirrors
servers
credentials references
profiles
repositories
plugin repositories
```

Avoid storing plaintext credentials where secure alternatives exist.

---

## 26. settings.xml vs pom.xml

General principle:

```text
pom.xml
 |
project configuration

settings.xml
 |
user/machine/CI configuration
```

Project-wide build configuration should normally be reproducible from
source control rather than hidden in an engineer's laptop settings.

---

# PART VII — MAVEN WRAPPER

## 27. What Is Maven Wrapper?

Maven Wrapper lets a project invoke Maven through wrapper scripts.

Common files:

```text
mvnw
mvnw.cmd
.mvn/wrapper/
```

---

## 28. Example

Linux/macOS:

```bash
./mvnw clean verify
```

Windows:

```powershell
mvnw.cmd clean verify
```

---

## 29. Why Use Maven Wrapper?

Benefits:

```text
consistent Maven version
less dependence on host installation
better CI reproducibility
simpler developer onboarding
```

---

# PART VIII — MAVEN LIFECYCLE

## 30. What Is a Lifecycle?

Maven defines standard build lifecycles.

Important lifecycle names include:

```text
clean
default
site
```

The default lifecycle is commonly the one used for compiling,
testing and packaging.

---

## 31. Common Default Lifecycle Phases

Important phases include:

```text
validate
compile
test
package
verify
install
deploy
```

The exact execution depends on project packaging and plugin
configuration.

---

## 32. Phase Ordering

A later phase normally causes earlier required phases in the same
lifecycle to execute.

For example:

```bash
mvn package
```

runs the necessary earlier default lifecycle phases before packaging.

---

## 33. Example

```text
mvn package
 |
 +--> validate
 +--> compile
 +--> test
 +--> package
```

---

## 34. verify

Example:

```bash
mvn verify
```

`verify` is useful when the project has verification checks configured
after packaging.

---

## 35. install

Example:

```bash
mvn install
```

It installs the artifact into the local Maven repository.

Typical flow:

```text
Build
 |
v
target/artifact.jar
 |
v
~/.m2/repository
```

---

## 36. deploy

Example:

```bash
mvn deploy
```

It publishes an artifact to the configured remote deployment
repository when the project is configured for deployment.

---

# PART IX — CLEAN

## 37. clean Lifecycle

The clean lifecycle removes generated build output.

```bash
mvn clean
```

Typically this removes:

```text
target/
```

---

## 38. clean package

Common CI command:

```bash
mvn clean package
```

This ensures the build starts from a clean generated-output state.

---

# PART X — DEPENDENCIES

## 39. Declaring Dependency

Example:

```xml
<dependencies>
    <dependency>
        <groupId>org.example</groupId>
        <artifactId>example-library</artifactId>
        <version>1.2.3</version>
    </dependency>
</dependencies>
```

---

## 40. Dependency Resolution

Maven determines:

```text
direct dependencies
transitive dependencies
versions
repositories
```

---

## 41. Dependency Tree

Use:

```bash
mvn dependency:tree
```

This is one of the most useful Maven troubleshooting commands.

---

# PART XI — MAVEN PLUGINS

## 42. What Is a Maven Plugin?

Maven performs much of its build work through plugins.

Examples include plugins for:

```text
compiler
testing
packaging
dependency analysis
code quality
publishing
```

---

## 43. Compiler Plugin

Conceptually:

```text
Java source
 |
v
maven-compiler-plugin
 |
v
.class
```

---

## 44. Surefire

The Maven Surefire Plugin is commonly used for unit-test execution.

Reports are commonly found under:

```text
target/surefire-reports/
```

---

## 45. Failsafe

The Maven Failsafe Plugin is commonly used for integration-test
execution.

The exact test naming/configuration depends on project setup.

---

# PART XII — MAVEN BUILD OUTPUT

## 46. JAR

Typical:

```text
target/payment-service-1.0.0.jar
```

---

## 47. WAR

Web applications may produce:

```text
target/payment-service-1.0.0.war
```

---

## 48. POM Packaging

A POM-packaged project can act as:

```text
parent POM
BOM
aggregator
```

depending on configuration.

---

# PART XIII — MAVEN PROFILES

## 49. What Is a Profile?

A profile allows conditional build configuration.

Examples:

```text
dev
test
prod
ci
```

---

## 50. Example

Concept:

```xml
<profiles>
    <profile>
        <id>dev</id>
        ...
    </profile>
</profiles>
```

Activate with:

```bash
mvn -Pdev package
```

---

## 51. Profile Warning

Do not use Maven profiles as a substitute for secure runtime
configuration.

Avoid embedding production secrets in profiles.

---

# PART XIV — PROPERTIES

## 52. Maven Properties

Example:

```xml
<properties>
    <java.version>21</java.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

Properties can centralize configuration values.

---

## 53. Property Usage

Example:

```xml
<version>${some.version}</version>
```

Use clear naming and avoid creating an unreadable web of indirection.

---

# PART XV — JAVA VERSION

## 54. Compiler Configuration

A project should explicitly define the intended Java release where
appropriate.

Modern Maven compiler configuration can use a release setting such as:

```xml
<properties>
    <maven.compiler.release>21</maven.compiler.release>
</properties>
```

The actual JDK used by CI must also support the selected release.

---

## 55. Runtime vs Compile JDK

Do not confuse:

```text
JDK used to build
```

with:

```text
JRE/JDK/runtime used to run
```

Compatibility must be tested across both.

---

# PART XVI — MAVEN COMMANDS

## 56. Basic Commands

```bash
mvn validate
mvn compile
mvn test
mvn package
mvn verify
mvn install
mvn deploy
```

---

## 57. Clean Commands

```bash
mvn clean
mvn clean test
mvn clean package
mvn clean verify
```

---

## 58. Skip Tests

Maven projects commonly distinguish between:

```bash
-DskipTests
```

and:

```bash
-Dmaven.test.skip=true
```

They are not identical.

Use test skipping only when there is a documented reason.

---

# PART XVII — MAVEN CI

## 59. Basic Jenkins Flow

```text
Git
 |
v
Jenkins
 |
v
Checkout
 |
v
./mvnw clean verify
 |
v
Artifact
 |
v
Repository
```

---

## 60. CI Best Practice

Prefer:

```bash
./mvnw -B clean verify
```

when the project uses Maven Wrapper.

---

## 61. CI Dependency Cache

A CI runner can cache Maven dependencies, commonly based around:

```text
~/.m2/repository
```

The cache strategy should be designed so dependency changes invalidate
stale cache state.

---

# PART XVIII — MAVEN + ARTIFACTORY

## 62. Enterprise Flow

```text
Maven
 |
v
Artifactory Virtual
 |
+--> Internal artifacts
+--> Approved external dependencies
```

---

## 63. Publishing

```text
Maven Build
 |
v
JAR
 |
v
Artifactory Local Repository
```

Use appropriate release/snapshot repositories according to the
organization's release policy.

---

## 64. Authentication

CI should use:

```text
service identity
scoped token
```

rather than an administrator account.

---

# PART XIX — SNAPSHOTS

## 65. Snapshot Concept

Snapshot versions represent development versions that may change.

Example:

```text
1.5.0-SNAPSHOT
```

They should generally not be treated as immutable production releases.

---

## 66. Release Version

Example:

```text
1.5.0
```

A release repository should enforce the organization's immutability
policy.

---

# PART XX — MAVEN INSTALL VS DEPLOY

## 67. install

```bash
mvn install
```

publishes the artifact to the local repository:

```text
~/.m2/repository
```

---

## 68. deploy

```bash
mvn deploy
```

publishes the artifact to the configured remote repository.

---

## 69. Production Rule

Do not confuse:

```text
install
```

with:

```text
release to enterprise artifact repository
```

The latter generally requires:

```text
deploy
```

or another controlled publishing workflow.

---

# PART XXI — MAVEN SETTINGS AND CREDENTIALS

## 70. Server Configuration

Concept:

```xml
<server>
    <id>company-releases</id>
    <username>...</username>
    <password>...</password>
</server>
```

The credential values should be injected securely rather than
committed into source control.

---

## 71. Repository ID Matching

The repository/server IDs used by Maven configuration must match as
required by the Maven setup.

A mismatch can produce authentication or publishing failures.

---

# PART XXII — MIRRORS

## 72. Maven Mirror

Organizations can configure Maven to use an internal mirror.

Concept:

```text
Maven
 |
v
Corporate Mirror
 |
+--> Internal
+--> Approved External
```

This can improve:

```text
control
speed
availability
security
```

---

# PART XXIII — BUILD REPRODUCIBILITY

## 73. Reproducible Maven Build

Control:

```text
JDK
Maven
dependencies
plugins
repositories
build configuration
environment
```

---

## 74. Maven Wrapper

Use:

```bash
./mvnw
```

to reduce Maven-version drift.

---

## 75. Dependency Versions

Centralize dependency versions where practical using:

```text
dependencyManagement
parent POM
BOM
properties
```

---

# PART XXIV — BOM

## 76. What Is a BOM?

A Bill of Materials can centrally define compatible versions of a group
of dependencies.

Concept:

```text
BOM
 |
+--> dependency A 1.x
+--> dependency B 2.x
+--> dependency C 3.x
```

---

## 77. Why BOM?

It helps maintain a tested set of compatible dependency versions.

---

# PART XXV — MULTI-MODULE FOUNDATION

## 78. Multi-Module Project

Example:

```text
payment-platform/
├── pom.xml
├── payment-api/
├── payment-core/
├── payment-db/
└── payment-app/
```

---

## 79. Parent vs Aggregator

A Maven POM can provide:

```text
parent configuration
```

and/or:

```text
module aggregation
```

These concepts are related but not identical.

The multi-module file covers this in depth.

---

# PART XXVI — TESTING

## 80. Unit Tests

Typically execute during:

```text
test
```

---

## 81. Integration Tests

May run during later verification phases depending on plugin
configuration.

---

## 82. CI Quality Gate

Example:

```text
Compile
 |
v
Unit Tests
 |
v
Integration Tests
 |
v
Security
 |
v
Package
```

---

# PART XXVII — STATIC ANALYSIS

## 83. Quality Tools

Maven builds can integrate with tools for:

```text
code quality
style
SAST
dependency analysis
coverage
```

Examples in enterprise environments may include:

```text
SonarQube
Checkstyle
SpotBugs
PMD
```

The exact toolset is organization-specific.

---

# PART XXVIII — MAVEN SECURITY

## 84. Dependency Security

Scan:

```text
direct dependencies
transitive dependencies
```

---

## 85. Build Plugin Security

Plugins execute during the build and should also be treated as supply
chain dependencies.

Use:

```text
trusted plugin sources
controlled versions
security review
```

---

## 86. Secrets

Never commit:

```text
Artifactory password
cloud credentials
signing key
database password
```

---

# PART XXIX — MAVEN TROUBLESHOOTING

## 87. Dependency Resolution Failure

Run:

```bash
mvn dependency:tree
```

Then check:

```text
repository
version
network
TLS
credentials
artifact existence
```

---

## 88. Dependency Not Found

Check:

```text
groupId
artifactId
version
repository
mirror
```

---

## 89. 401

Likely:

```text
authentication
credential
token
```

---

## 90. 403

Likely:

```text
authorization
repository permission
path permission
```

---

## 91. 404

Likely:

```text
wrong coordinates
missing artifact
wrong repository
```

---

## 92. Compilation Failure

Check:

```text
JDK
Maven
compiler release
dependency versions
source code
```

---

## 93. NoSuchMethodError

This commonly suggests a runtime dependency mismatch.

Investigate:

```bash
mvn dependency:tree
```

and compare runtime libraries.

---

## 94. ClassNotFoundException

Check:

```text
missing runtime dependency
scope
packaging
dependency exclusion
```

---

# PART XXX — PERFORMANCE

## 95. Slow Dependency Downloads

Check:

```text
repository latency
network
proxy
Maven mirror
cache
artifact size
```

---

## 96. Slow Compilation

Check:

```text
CPU
memory
source size
annotation processing
compiler configuration
```

---

## 97. Slow Tests

Measure:

```text
unit tests
integration tests
external calls
database setup
container startup
```

Then parallelize or optimize where safe.

---

# PART XXXI — MAVEN DEBUGGING

## 98. Verbose Output

Useful option:

```bash
mvn -X
```

It produces extensive debug output.

Do not paste debug logs containing secrets into public channels.

---

## 99. Effective POM

Useful command:

```bash
mvn help:effective-pom
```

This helps investigate inherited and effective configuration.

---

## 100. Effective Settings

Useful command:

```bash
mvn help:effective-settings
```

Use it to understand effective Maven settings.

---

# PART XXXII — MAVEN OFFLINE MODE

## 101. Offline

Example:

```bash
mvn -o test
```

This tells Maven to work offline.

It succeeds only if required artifacts and metadata are already
available locally.

---

# PART XXXIII — MAVEN CI FAILURE PATTERNS

## 102. Works Locally, Fails in CI

Investigate:

```text
JDK
Maven version
Maven settings
environment
credentials
repository
network
cache
```

---

## 103. Works on One Runner, Fails on Another

Likely:

```text
runner toolchain
local cache
disk
environment
network
```

---

## 104. All Runners Fail

Investigate shared infrastructure:

```text
Artifactory
DNS
network
identity
upstream repository
```

---

# PART XXXIV — MAVEN RELEASE ARCHITECTURE

## 105. Production Flow

```text
Git Tag
 |
v
CI
 |
v
./mvnw clean verify
 |
v
Security
 |
v
Package
 |
v
Deploy to Release Repository
 |
v
Artifact
 |
v
Promotion
 |
v
Production
```

---

## 106. Artifact Traceability

Track:

```text
groupId
artifactId
version
Git SHA
CI build
dependencies
repository
deployment
```

---

# PART XXXV — MAVEN WITH CONTAINERS

## 107. Java Container Build

```text
Git
 |
v
Maven
 |
v
JAR
 |
v
Container Image
 |
v
Registry
 |
v
Kubernetes
```

---

## 108. Multi-Stage Docker Build

Concept:

```text
Build Stage
 |
+--> JDK
+--> Maven
 |
v
JAR
 |
v
Runtime Stage
 |
+--> smaller runtime
 |
v
Image
```

This can reduce final image size.

---

# PART XXXVI — MAVEN AND KUBERNETES

## 109. Responsibility Separation

```text
Maven:
build application

Registry:
store image

GitOps:
declare desired version

Kubernetes:
run application
```

---

# PART XXXVII — MAVEN BUILD SECURITY PIPELINE

## 110. Secure Flow

```text
Checkout
 |
v
Maven Wrapper
 |
v
Dependency Resolution
 |
v
SCA
 |
v
Compile
 |
v
Test
 |
v
SAST
 |
v
Package
 |
v
Artifact Scan
 |
v
Publish
```

---

# PART XXXVIII — MAVEN BEST PRACTICES

## 111. Practices

```text
Use Maven Wrapper.
Pin important toolchain versions.
Control dependency versions.
Use a corporate artifact repository.
Use clean CI workspaces.
Do not commit target/.
Do not commit credentials.
Use immutable release versions.
Inspect dependency trees during conflicts.
Use dependency scanning.
Keep parent/BOM management understandable.
```

---

# PART XXXIX — PRODUCTION ARCHITECTURE

## 112. Enterprise Reference

```text
Developer
   |
   v
Git
   |
   v
CI
   |
   +--> Maven Wrapper
   |
   +--> Internal Maven Mirror
   |
   +--> Tests
   |
   +--> Security
   |
   v
Package
   |
   v
Artifactory
   |
   v
Container Build
   |
   v
Container Registry
   |
   v
GitOps
   |
   v
Kubernetes
```

---

# PART XL — PRODUCTION CHECKLIST

## 113. Project

```text
[ ] standard Maven structure
[ ] pom.xml reviewed
[ ] Maven Wrapper
[ ] Java version defined
```

## 114. Dependencies

```text
[ ] versions controlled
[ ] transitive graph reviewed
[ ] security scanning
[ ] approved repositories
```

## 115. Build

```text
[ ] compile
[ ] unit tests
[ ] integration tests
[ ] static analysis
[ ] package
```

## 116. CI

```text
[ ] clean workspace
[ ] reproducible toolchain
[ ] secure credentials
[ ] dependency cache
[ ] logs
```

## 117. Artifact

```text
[ ] versioned
[ ] immutable
[ ] scanned
[ ] traceable
[ ] published
```

---

# PART XLI — INTERVIEW PREPARATION

## 118. What Is Maven?

Answer:

```text
Maven is a Java build automation and dependency management tool. It
provides a standard project model, dependency resolution, build
lifecycles and plugin-based build execution, and can package and
publish artifacts.
```

## 119. What Is pom.xml?

Answer:

```text
pom.xml is the Maven Project Object Model. It defines project
coordinates and can define dependencies, dependency management,
plugins, build configuration, profiles, repositories and publishing
configuration.
```

## 120. What Is the Maven Lifecycle?

Answer:

```text
Maven provides standard build lifecycles. The default lifecycle
contains phases such as validate, compile, test, package, verify,
install and deploy. Running a later phase normally executes the
required earlier phases.
```

## 121. install vs deploy?

Answer:

```text
install puts the artifact into the local Maven repository. deploy
publishes it to the configured remote repository.
```

## 122. Why Maven Wrapper?

Answer:

```text
It helps ensure developers and CI use the Maven version expected by
the project, reducing toolchain drift and improving reproducibility.
```

## 123. How Do You Troubleshoot a Dependency Conflict?

Answer:

```text
I start with mvn dependency:tree, identify the paths introducing
the conflicting dependency, determine the selected version and then
use dependency management or another controlled solution. I run
tests afterward because dependency changes can cause runtime
incompatibilities.
```

## 124. How Do You Secure Maven Builds?

Answer:

```text
I use approved repositories, controlled dependency versions,
dependency and plugin scanning, protected CI credentials, clean
build agents, immutable artifacts and provenance tracking.
```

## 125. How Do You Make Maven Builds Reproducible?

Answer:

```text
I use Maven Wrapper, control the JDK, manage dependency versions,
use internal repositories, keep project configuration in source
control and avoid uncontrolled external inputs.
```

## 126. What Happens During mvn package?

Answer:

```text
Maven executes the required earlier phases of the default lifecycle,
including validation, compilation and testing according to project
configuration, and then packages the application into its configured
artifact format.
```

---

# PART XLII — SENIOR SCENARIOS

## 127. Maven Builds Suddenly Fail Across All Teams

Answer:

```text
I would first establish whether the failure is common to all
projects. If it is, I would investigate shared dependencies such as
the internal repository, DNS, network, authentication, certificates
or upstream repositories rather than changing individual POM files.
```

## 128. One Team's Maven Build Is Slow

Answer:

```text
I would compare dependency download time, compilation time, test time
and repository latency with the baseline. I would inspect the local
cache and dependency graph, then optimize the measured bottleneck
rather than adding infrastructure blindly.
```

## 129. Production Runtime Has NoSuchMethodError

Answer:

```text
I would suspect a runtime dependency mismatch. I would inspect the
resolved Maven dependency tree, identify duplicate or incompatible
versions, compare the packaged artifact with the runtime classpath,
then correct dependency management and test the resulting artifact.
```

## 130. Security Scanner Finds a Vulnerable Transitive Dependency

Answer:

```text
I identify which direct dependency introduced it, check whether a
fixed version exists, evaluate compatibility, use dependency
management or upgrade the parent dependency, run tests and security
scans, and publish a new artifact if required.
```

## 131. Maven Works Locally but Fails in Jenkins

Answer:

```text
I compare the JDK and Maven versions, Maven settings, credentials,
repository configuration, network path, environment variables and
local cache assumptions. I reproduce the Jenkins environment rather
than changing dependencies blindly.
```

---

# PART XLIII — GOLDEN RULES

## 132. Rules

```text
1. Understand Maven as a build platform, not just a command.

2. Know the standard Maven project structure.

3. Understand groupId, artifactId and version.

4. Keep pom.xml understandable.

5. Use Maven Wrapper for controlled Maven versions.

6. Control the JDK used by CI.

7. Understand lifecycle phases.

8. Know clean, test, package, verify, install and deploy.

9. Understand install vs deploy.

10. Use dependency:tree during dependency investigations.

11. Understand direct and transitive dependencies.

12. Control dependency versions.

13. Use dependencyManagement or BOMs appropriately.

14. Treat plugins as supply-chain dependencies.

15. Use approved repositories.

16. Prefer enterprise repository mirrors/virtual repositories where
    organizational architecture requires them.

17. Do not commit credentials.

18. Do not use administrator credentials in CI.

19. Use scoped service identities.

20. Keep release artifacts immutable.

21. Do not use SNAPSHOT artifacts as production releases.

22. Keep target/ out of source control.

23. Use clean CI workspaces.

24. Cache dependencies carefully.

25. Design cache keys around dependency/toolchain inputs.

26. Scan direct and transitive dependencies.

27. Scan build plugins where supported.

28. Test dependency upgrades.

29. Do not solve every dependency conflict with exclusions.

30. Investigate runtime errors against the actual dependency graph.

31. Separate build-time configuration from runtime secrets.

32. Do not use Maven profiles as a secret store.

33. Record build provenance.

34. Build once and promote the same artifact.

35. Package the artifact before creating the production container.

36. Keep CI and deployment responsibilities separate.

37. Monitor build duration and failure rates.

38. Troubleshoot shared infrastructure when many builds fail together.

39. Reproduce CI failures in equivalent environments.

40. Validate exact Maven/JDK/plugin behavior for the versions actually
    deployed in production.
```

---

# END OF 03-Maven-Fundamentals.md
