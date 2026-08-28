# 18-Build-and-Package-Management
# 06-Maven-Dependencies-and-Plugins

## 1. Purpose

Maven dependencies and plugins are two of the most important parts of a
real enterprise Maven build.

A dependency supplies software required by the application or tests.

A plugin supplies executable build behavior.

The distinction is critical:

```text
Dependency
    |
    v
Application functionality

Plugin
    |
    v
Build functionality
```

A production DevOps engineer should understand:

```text
dependency declaration
transitive dependencies
dependency scopes
dependency mediation
dependencyManagement
BOM
exclusions
optional dependencies
plugin declaration
plugin versions
plugin goals
plugin executions
pluginManagement
lifecycle bindings
plugin configuration
security
repository resolution
caching
CI/CD
Artifactory
troubleshooting
```

---

# PART I — DEPENDENCY FUNDAMENTALS

## 2. What Is a Maven Dependency?

A Maven dependency is an artifact required by a project.

Example:

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>payment-client</artifactId>
    <version>2.4.1</version>
</dependency>
```

The project can use classes and functionality supplied by the artifact.

---

## 3. Maven Coordinates

A dependency is commonly identified by:

```text
groupId
artifactId
version
```

Example:

```text
com.company
payment-client
2.4.1
```

Together:

```text
com.company:payment-client:2.4.1
```

---

## 4. Additional Dependency Attributes

A dependency can also specify:

```text
scope
optional
classifier
type
exclusions
```

These attributes affect how Maven resolves and uses the dependency.

---

# PART II — DIRECT DEPENDENCIES

## 5. Direct Dependency

If the application explicitly declares:

```text
A
```

then A is a direct dependency.

```text
Application
    |
    +--> A
```

---

## 6. Why Direct Dependencies Should Be Explicit

If application code imports classes from A, declaring A directly makes
the dependency contract clear.

Do not rely on another library's transitive dependency merely because
the current version happens to expose it.

---

# PART III — TRANSITIVE DEPENDENCIES

## 7. Transitive Dependency

Suppose:

```text
Application -> A
A -> B
```

Then:

```text
Application
 |
 +--> A
      |
      +--> B
```

B is transitive.

---

## 8. Why Transitive Dependencies Matter

Transitive dependencies can introduce:

```text
security vulnerabilities
version conflicts
licensing obligations
runtime incompatibilities
```

Therefore dependency analysis must include the resolved graph.

---

# PART IV — DEPENDENCY GRAPH

## 9. Example

```text
Application
 |
+--> A 1.0
|     |
|     +--> C 2.0
|
+--> B 3.0
      |
      +--> C 1.0
      |
      +--> D 4.0
```

Maven must resolve the competing versions of C.

---

## 10. Dependency Tree

Use:

```bash
mvn dependency:tree
```

Example:

```text
com.company:payment-service
+- com.company:A:1.0
|  \- org.example:C:2.0
\- com.company:B:3.0
   \- org.example:C:1.0
```

This is one of the first commands to use during dependency
troubleshooting.

---

# PART V — DEPENDENCY MEDIATION

## 11. Why Version Conflicts Occur

Different dependency paths can request different versions.

```text
A -> C 1.0
B -> C 2.0
```

The resolved graph must choose a version according to Maven's dependency
mediation rules.

---

## 12. Do Not Guess the Selected Version

Never assume the POM declaration tells the whole story.

Inspect:

```bash
mvn dependency:tree
```

and, when useful:

```bash
mvn help:effective-pom
```

---

# PART VI — DEPENDENCY SCOPES

## 13. Common Scopes

Maven commonly supports:

```text
compile
provided
runtime
test
system
```

Use them intentionally.

---

## 14. compile

Compile scope is the normal default.

It is generally available to:

```text
compile
test
runtime
```

according to Maven's dependency classpath semantics.

---

## 15. provided

A provided dependency is expected to be supplied by the execution
environment.

Typical concept:

```text
Application
 |
v
API
 |
v
Application Server provides implementation
```

---

## 16. runtime

A runtime dependency is required when the application runs but may not
be required to compile the application source.

---

## 17. test

Example:

```xml
<scope>test</scope>
```

Used only for test-related classpaths.

Examples:

```text
JUnit
test utilities
mocking frameworks
```

---

## 18. system

System scope is a special case that refers to an explicitly supplied
local file path.

It should generally be avoided in modern enterprise builds because it
reduces portability and reproducibility.

---

# PART VII — OPTIONAL DEPENDENCIES

## 19. optional

Example:

```xml
<optional>true</optional>
```

Optional dependencies have different transitive propagation semantics
for consumers.

Use them only when the library's API and dependency model actually
requires optional behavior.

---

# PART VIII — EXCLUSIONS

## 20. Excluding a Dependency

Example:

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>library-a</artifactId>
    <version>1.0</version>

    <exclusions>
        <exclusion>
            <groupId>org.example</groupId>
            <artifactId>library-b</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

---

## 21. Why Exclude?

Possible reasons:

```text
conflicting implementation
unwanted transitive dependency
security issue
runtime-provided implementation
duplicate library
```

---

## 22. Exclusion Risk

An exclusion can cause:

```text
ClassNotFoundException
NoClassDefFoundError
NoSuchMethodError
```

Always run relevant tests after exclusions.

---

# PART IX — DEPENDENCY MANAGEMENT

## 23. dependencyManagement

Example:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>library-a</artifactId>
            <version>2.0.0</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

This centralizes dependency information.

---

## 24. dependencies vs dependencyManagement

Remember:

```text
dependencies
    |
declares usage

dependencyManagement
    |
manages dependency information
```

An entry in `dependencyManagement` does not by itself add the
dependency to the project's classpath.

---

# PART X — BOM

## 25. Bill of Materials

A BOM manages a compatible collection of dependency versions.

Example concept:

```text
Company BOM
 |
+--> library A 1.x
+--> library B 2.x
+--> library C 3.x
```

---

## 26. Importing a BOM

Example:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>example-bom</artifactId>
            <version>5.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

## 27. BOM Benefits

```text
consistent versions
tested combinations
centralized upgrades
less duplication
```

---

# PART XI — VERSION PROPERTIES

## 28. Centralized Version

Example:

```xml
<properties>
    <library-a.version>2.0.0</library-a.version>
</properties>
```

Then:

```xml
<version>${library-a.version}</version>
```

---

## 29. Benefits

```text
easy upgrades
centralized review
less duplication
```

Do not create so many properties that the POM becomes difficult to
follow.

---

# PART XII — DEPENDENCY VERSION STRATEGY

## 30. Exact Versions

Prefer controlled explicit versions for production dependencies.

Example:

```text
2.4.1
```

---

## 31. Uncontrolled Version Ranges

Avoid dependency specifications that allow unexpected upgrades unless
there is a deliberate reason and the project has tested the behavior.

---

## 32. Release Management

A dependency upgrade should follow:

```text
identify
 |
v
review
 |
v
upgrade
 |
v
test
 |
v
scan
 |
v
release
```

---

# PART XIII — DEPENDENCY SECURITY

## 33. Direct Dependency Vulnerability

```text
Application
 |
v
Library A
 |
v
CVE
```

The application directly owns the dependency.

---

## 34. Transitive Vulnerability

```text
Application
 |
v
Framework
 |
v
Library A
 |
v
CVE
```

The application may still be affected.

---

## 35. Response

```text
identify
 |
v
determine exposure
 |
v
find fixed version
 |
v
test
 |
v
release
```

---

# PART XIV — DEPENDENCY CONFUSION

## 36. Attack Model

An attacker may publish a public package with a name similar to an
internal package.

Example:

```text
Internal:
com.company:security-utils

Public malicious artifact:
com.company:security-utils
```

A poorly configured repository strategy can create supply-chain risk.

---

## 37. Prevention

Use:

```text
internal repositories
approved mirrors
namespace controls
repository policies
```

---

# PART XV — REPOSITORY RESOLUTION

## 38. Resolution Flow

```text
POM
 |
v
Local ~/.m2
 |
+--> found --> use cache
 |
+--> missing
       |
       v
Internal Repository
       |
       +--> local
       +--> remote cache
       |
       v
Approved Upstream
```

---

## 39. Why Internal Repository?

Benefits:

```text
caching
availability
audit
security
centralized access
```

---

# PART XVI — ARTIFACTORY

## 40. Maven + Artifactory

Typical enterprise architecture:

```text
Maven
 |
v
Artifactory Virtual
 |
+--> Company Local
+--> Approved Remote
```

---

## 41. Dependency Consumption

CI can use a stable internal endpoint:

```text
https://repo.company.example/maven/virtual
```

The actual URL is organization-specific.

---

## 42. Publishing

```text
Maven
 |
v
package
 |
v
deploy
 |
v
Artifactory Release Repository
```

---

# PART XVII — MAVEN PLUGINS

## 43. What Is a Plugin?

A Maven plugin provides executable build goals.

Examples:

```text
maven-compiler-plugin
maven-surefire-plugin
maven-failsafe-plugin
maven-jar-plugin
maven-war-plugin
```

---

## 44. Dependency vs Plugin

```text
Dependency
 |
application/test classpath

Plugin
 |
Maven build execution
```

Plugins are part of the build supply chain.

---

# PART XVIII — PLUGIN COORDINATES

## 45. Plugin Example

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>...</version>
</plugin>
```

---

## 46. Plugin Version

Plugin versions should be explicitly controlled for important builds.

Do not rely on accidental versions inherited from an uncontrolled
environment.

---

# PART XIX — PLUGIN GOALS

## 47. Goal

A plugin exposes goals.

Example:

```text
maven-compiler-plugin:compile
```

means:

```text
plugin = maven-compiler-plugin
goal   = compile
```

---

## 48. Direct Goal Invocation

Example:

```bash
mvn dependency:tree
```

This invokes a plugin goal directly.

---

# PART XX — LIFECYCLE BINDINGS

## 49. Binding

A plugin goal can be bound to a lifecycle phase.

Example:

```text
compile phase
    |
    v
compiler:compile
```

---

## 50. Why Bind Goals?

This allows standard lifecycle commands to trigger the required
plugin behavior.

Example:

```bash
mvn compile
```

can execute the compiler plugin's compile goal through lifecycle
binding.

---

# PART XXI — PLUGIN EXECUTIONS

## 51. execution

Example:

```xml
<executions>
    <execution>
        <id>generate-sources</id>
        <phase>generate-sources</phase>
        <goals>
            <goal>generate</goal>
        </goals>
    </execution>
</executions>
```

---

## 52. Execution ID

Use meaningful IDs:

```text
generate-openapi
integration-tests
frontend-build
```

This makes effective configuration easier to understand.

---

# PART XXII — PLUGIN CONFIGURATION

## 53. configuration

Example:

```xml
<configuration>
    <release>21</release>
</configuration>
```

Configuration is plugin-specific.

Do not assume an option from one plugin works in another.

---

# PART XXIII — PLUGINMANAGEMENT

## 54. What Is pluginManagement?

It centrally manages plugin configuration and versions for a POM
hierarchy.

Example:

```xml
<pluginManagement>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>...</version>
        </plugin>
    </plugins>
</pluginManagement>
```

---

## 55. pluginManagement vs plugins

Remember:

```text
pluginManagement
 |
managed configuration

plugins
 |
actual plugin usage
```

A plugin normally needs to be referenced in the build to execute.

---

# PART XXIV — COMPILER PLUGIN

## 56. Purpose

The compiler plugin compiles Java source.

Concept:

```text
.java
 |
v
compiler plugin
 |
v
.class
```

---

## 57. Java Release

Example:

```xml
<configuration>
    <release>21</release>
</configuration>
```

The CI JDK must support the selected release.

---

# PART XXV — SUREFIRE

## 58. Surefire

The Maven Surefire Plugin is commonly used for unit tests.

Typical lifecycle location:

```text
test
```

---

## 59. Reports

Common output:

```text
target/surefire-reports/
```

Use these reports for CI test diagnostics.

---

# PART XXVI — FAILSAFE

## 60. Failsafe

The Maven Failsafe Plugin is commonly used for integration tests.

Typical lifecycle relationship:

```text
pre-integration-test
integration-test
post-integration-test
verify
```

---

## 61. Why Failsafe?

It supports integration-test workflows where test infrastructure needs
to be prepared and cleaned up around the test phase.

---

# PART XXVII — JAR PLUGIN

## 62. JAR Plugin

The Maven JAR Plugin creates JAR artifacts.

Concept:

```text
classes
 |
v
jar plugin
 |
v
application.jar
```

---

# PART XXVIII — WAR PLUGIN

## 63. WAR Plugin

Used for projects packaged as:

```xml
<packaging>war</packaging>
```

It produces a WAR artifact according to project configuration.

---

# PART XXIX — SOURCE AND JAVADOC PLUGINS

## 64. Source/Javadoc Artifacts

Projects may generate:

```text
sources.jar
javadoc.jar
```

These are often useful when publishing reusable libraries.

---

# PART XXX — DEPENDENCY PLUGIN

## 65. Dependency Plugin

Useful goals can inspect or analyze dependencies.

Example:

```bash
mvn dependency:tree
```

---

## 66. Dependency Analysis

Use dependency tooling to identify:

```text
unused dependencies
undeclared dependencies
transitive dependencies
```

Do not remove a dependency solely because one static analysis result
says it is unused; verify runtime behavior and project conventions.

---

# PART XXXI — PLUGIN SECURITY

## 67. Plugins Execute Code

A Maven plugin can execute code on the build agent.

Therefore:

```text
plugin source
plugin version
plugin configuration
```

are security-sensitive.

---

## 68. Plugin Repository

Prefer trusted plugin repositories and controlled mirrors.

---

## 69. Plugin Updates

Test plugin upgrades like application dependency upgrades.

A plugin upgrade can change:

```text
compiler behavior
test behavior
packaging
resource processing
```

---

# PART XXXII — PLUGIN DEPENDENCIES

## 70. Plugins Can Have Dependencies

A plugin itself can depend on other libraries.

Therefore the build supply chain can include:

```text
Application dependencies
+
Plugin dependencies
```

---

## 71. Enterprise Security

A strong security model scans both where tooling supports it.

---

# PART XXXIII — MAVEN RESOLUTION CACHE

## 72. Local Cache

Maven stores downloaded dependencies and metadata under:

```text
~/.m2/repository
```

---

## 73. CI Cache

A CI runner may cache:

```text
~/.m2/repository
```

to reduce network downloads.

---

## 74. Cache Key

A useful cache key should account for relevant inputs such as:

```text
OS
JDK
Maven
POM
dependency lock/configuration
```

A common practical approach includes a hash of relevant POM files.

---

# PART XXXIV — SNAPSHOTS

## 75. Snapshot Dependency

Example:

```text
1.5.0-SNAPSHOT
```

Snapshot artifacts represent development versions and may change.

---

## 76. Production Rule

Avoid depending on mutable snapshots in production release paths.

Prefer:

```text
1.5.0
```

for a release artifact.

---

# PART XXXV — DEPENDENCY OVERRIDES

## 77. Why Override?

Suppose:

```text
A -> vulnerable C 1.0
```

and:

```text
C 1.1
```

is a compatible fixed release.

Dependency management can sometimes be used to select the approved
version.

---

## 78. Validate Overrides

Run:

```text
unit tests
integration tests
application startup
critical business flows
```

before release.

---

# PART XXXVI — RUNTIME CONFLICTS

## 79. NoSuchMethodError

Typical investigation:

```bash
mvn dependency:tree
```

Then compare:

```text
compile graph
packaged dependencies
runtime classpath
```

---

## 80. ClassNotFoundException

Investigate:

```text
scope
exclusion
packaging
runtime dependency
container image
```

---

## 81. NoClassDefFoundError

This may indicate a missing class at runtime.

Inspect:

```text
dependency graph
artifact contents
runtime classpath
```

---

# PART XXXVII — DEPENDENCY CONVERGENCE

## 82. Version Convergence

A healthy dependency graph avoids unnecessary multiple versions of the
same library.

Example:

```text
Bad:
C 1.0
C 1.1
C 2.0
```

Prefer a tested consistent version where compatible.

---

# PART XXXVIII — DEPENDENCY EXCLUSIONS VS MANAGEMENT

## 83. Exclusion

Use when a transitive dependency should not be brought in.

```text
A -> B
```

Exclude B.

---

## 84. Dependency Management

Use when you want to control the selected version:

```text
A -> B 1.0
C -> B 2.0
```

and a tested version needs to be selected.

Do not use exclusions when simple version management is the correct
solution.

---

# PART XXXIX — DEPENDENCY TREE COMMANDS

## 85. Basic

```bash
mvn dependency:tree
```

---

## 86. Verbose Tree

Maven's dependency plugin supports additional options for filtering and
verbose dependency analysis.

Use the plugin documentation for the exact syntax supported by the
version used by your project.

---

# PART XL — EFFECTIVE POM

## 87. Effective POM

Use:

```bash
mvn help:effective-pom
```

It helps reveal:

```text
inherited dependencies
plugin configuration
plugin management
profiles
properties
```

---

# PART XLI — EFFECTIVE SETTINGS

## 88. Effective Settings

Use:

```bash
mvn help:effective-settings
```

Useful for diagnosing:

```text
mirrors
servers
profiles
repositories
```

---

# PART XLII — REPOSITORY AUTHENTICATION

## 89. settings.xml

Credentials are commonly referenced through Maven's server
configuration.

Concept:

```xml
<server>
    <id>company-releases</id>
    <username>...</username>
    <password>...</password>
</server>
```

Use secure CI secret injection.

---

## 90. Never Commit Credentials

Bad:

```xml
<password>production-password</password>
```

Use:

```text
CI secret store
service identity
short-lived/scoped credentials
```

where supported.

---

# PART XLIII — MIRRORS

## 91. Corporate Mirror

Example architecture:

```text
Maven
 |
v
Corporate Mirror
 |
+--> Internal
+--> Approved External
```

This creates a controlled dependency entry point.

---

# PART XLIV — OFFLINE MODE

## 92. Offline

Example:

```bash
mvn -o test
```

Maven will avoid remote repository access.

This only works if required artifacts are already locally available.

---

# PART XLV — FORCE UPDATE

## 93. Update Resolution

Maven provides options such as:

```bash
-U
```

to force checking for updated snapshots/releases according to Maven's
resolution behavior.

Use it carefully because it can increase network traffic and reduce
the benefit of caches.

---

# PART XLVI — DEPENDENCY CACHING FAILURE

## 94. Corrupt Local Cache

Symptoms:

```text
unexpected parsing failure
checksum issue
corrupt artifact
```

A controlled cache cleanup and re-download can be appropriate after
confirming the repository copy is valid.

---

## 95. CI Cache Problem

If all builds on one runner fail:

```text
runner cache
```

may be relevant.

If all runners fail:

```text
internal repository
network
DNS
TLS
```

become stronger suspects.

---

# PART XLVII — BUILD REPRODUCIBILITY

## 96. Control

For reproducible builds control:

```text
Maven version
JDK
dependencies
plugins
repositories
profiles
build configuration
```

---

## 97. Maven Wrapper

Use:

```bash
./mvnw
```

to control Maven version at project level.

---

# PART XLVIII — CI PIPELINE

## 98. Dependency + Plugin Flow

```text
Checkout
 |
v
Maven Wrapper
 |
v
Resolve Dependencies
 |
v
Resolve Plugins
 |
v
Compile
 |
v
Test
 |
v
Package
 |
v
Scan
 |
v
Publish
```

---

## 99. Failure Gate

```text
Dependency resolution FAIL
        |
        X
No compile

Test FAIL
        |
        X
No publish

Security FAIL
        |
        X
No production promotion
```

---

# PART XLIX — JENKINS

## 100. Jenkins

Example:

```groovy
stage('Build') {
    steps {
        sh './mvnw -B clean verify'
    }
}
```

---

## 101. Publish

```groovy
stage('Publish') {
    steps {
        sh './mvnw -B deploy'
    }
}
```

Use release repository policy appropriate to the artifact version.

---

# PART L — GITHUB ACTIONS

## 102. Typical Flow

```text
Checkout
 |
v
Setup JDK
 |
v
Maven Wrapper
 |
v
clean verify
 |
v
security
 |
v
deploy
```

---

# PART LI — ARTIFACTORY + PLUGINS

## 103. Plugin Resolution

A corporate repository can also provide controlled access to plugin
artifacts.

Concept:

```text
Maven
 |
v
Internal Repository
 |
+--> Application dependencies
+--> Plugin dependencies
```

---

## 104. Why?

Benefits:

```text
cache
availability
governance
audit
```

---

# PART LII — SECURITY SCANNING

## 105. Scan Dependency Graph

Inspect:

```text
direct dependencies
transitive dependencies
```

---

## 106. Scan Build Supply Chain

Where tooling supports it, also inspect:

```text
plugins
plugin dependencies
build images
JDK/base tooling
```

---

## 107. SBOM

A build can generate an SBOM describing application components.

Concept:

```text
JAR
 |
v
SBOM
 |
+--> dependency A
+--> dependency B
+--> dependency C
```

---

# PART LIII — LICENSES

## 108. License Review

Dependency management should account for organizational license
policies.

Possible workflow:

```text
Dependency
 |
v
License Scan
 |
+--> approved
+--> review
+--> blocked
```

---

# PART LIV — PRODUCTION UPGRADE

## 109. Dependency Upgrade

```text
Current
 |
v
New version
 |
v
Compatibility
 |
v
Security
 |
v
Tests
 |
v
Artifact
```

---

## 110. Plugin Upgrade

```text
Current plugin
 |
v
New plugin
 |
v
Build validation
 |
v
Test
 |
v
Release
```

---

# PART LV — PRODUCTION INCIDENTS

## 111. Incident: Runtime NoSuchMethodError

Possible cause:

```text
dependency version mismatch
```

Investigation:

```bash
mvn dependency:tree
```

Then inspect the actual packaged artifact.

---

## 112. Incident: Security Scanner Reports Transitive CVE

Process:

```text
CVE
 |
v
dependency path
 |
v
fixed version
 |
v
compatibility
 |
v
tests
 |
v
release
```

---

## 113. Incident: Build Suddenly Cannot Resolve Plugin

Check:

```text
plugin coordinates
plugin repository
mirror
DNS
TLS
authentication
Artifactory
```

---

## 114. Incident: Plugin Upgrade Breaks Build

Compare:

```text
plugin version
effective POM
parent POM
JDK
Maven version
plugin configuration
```

---

## 115. Incident: Local Build Works but CI Fails

Compare:

```text
JDK
Maven
settings.xml
repository
credentials
cache
environment
```

---

# PART LVI — PRODUCTION ARCHITECTURE

## 116. Enterprise Reference

```text
                    Public Repositories
                   /        |         \
                  v         v          v
               Maven       npm       PyPI
                  \         |         /
                   +--------+--------+
                            |
                            v
                    Enterprise Repository
                            |
                 +----------+----------+
                 |                     |
                 v                     v
              Virtual                Local
                 |
                 v
                CI
                 |
                 v
           Maven Wrapper
                 |
        +--------+--------+
        |                 |
        v                 v
   Dependencies        Plugins
        |                 |
        +--------+--------+
                 |
                 v
              Compile
                 |
                 v
                Test
                 |
                 v
               Verify
                 |
                 v
              Package
                 |
                 v
            Security Scan
                 |
                 v
            Artifactory
                 |
                 v
           Container Build
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

# PART LVII — PRODUCTION CHECKLIST

## 117. Dependencies

```text
[ ] direct dependencies explicit
[ ] transitive graph understood
[ ] versions controlled
[ ] scopes correct
[ ] exclusions justified
[ ] BOM reviewed
[ ] vulnerabilities scanned
[ ] licenses reviewed
```

## 118. Plugins

```text
[ ] plugin versions controlled
[ ] plugin sources trusted
[ ] plugin configuration reviewed
[ ] executions understood
[ ] pluginManagement understood
[ ] plugin upgrades tested
```

## 119. Repository

```text
[ ] internal repository
[ ] controlled mirror
[ ] HTTPS
[ ] least-privilege credentials
[ ] cache
[ ] release immutability
```

## 120. CI

```text
[ ] Maven Wrapper
[ ] JDK controlled
[ ] clean build
[ ] dependency cache
[ ] security gates
[ ] test reports
[ ] artifact traceability
```

---

# PART LVIII — INTERVIEW PREPARATION

## 121. Dependency vs Plugin?

Answer:

```text
A dependency provides application or test libraries. A Maven plugin
provides executable build behavior and exposes goals that can be
bound to lifecycle phases or invoked directly.
```

## 122. What Is a Transitive Dependency?

Answer:

```text
It is a dependency introduced through another dependency. I inspect
the complete resolved graph because transitive dependencies can cause
security, compatibility and licensing issues.
```

## 123. How Do You Find Dependency Conflicts?

Answer:

```text
I use mvn dependency:tree to see which dependency paths introduce the
conflicting artifact and which version Maven resolves. I then use
dependencyManagement, a compatible upgrade or another controlled
solution.
```

## 124. dependencyManagement vs dependencies?

Answer:

```text
dependencies declares actual usage. dependencyManagement centrally
controls dependency information such as versions for the project
hierarchy but does not itself add the dependency to the classpath.
```

## 125. pluginManagement vs plugins?

Answer:

```text
pluginManagement centralizes plugin configuration and versions.
plugins represents plugin usage in the build. A plugin generally must
be used in the effective build for its goals to execute.
```

## 126. Why Are Plugin Versions Important?

Answer:

```text
Plugins execute build logic. A plugin upgrade can change compilation,
testing, packaging or verification behavior, so I treat important
plugin versions as controlled build dependencies.
```

## 127. How Do You Handle a Vulnerable Transitive Dependency?

Answer:

```text
I identify the dependency path, determine whether a fixed version
exists, test an upgrade or controlled version management, run
application tests and security scans, and release a new artifact.
```

## 128. How Do You Secure Maven Plugins?

Answer:

```text
I use trusted plugin repositories, controlled versions, dependency
scanning where supported, restricted CI permissions and controlled
build environments because plugins execute code on the build agent.
```

## 129. Why Use Artifactory?

Answer:

```text
It provides a controlled enterprise endpoint for dependencies and
plugins, supports caching and access control, improves availability
and gives the organization better supply-chain governance.
```

## 130. How Do You Troubleshoot Plugin Resolution?

Answer:

```text
I verify plugin coordinates and version, then check effective POM,
plugin repositories, mirrors, credentials, DNS, TLS, network and the
internal repository.
```

---

# PART LIX — SENIOR-LEVEL SCENARIOS

## 131. Hundreds of Services Have Conflicting Versions

Answer:

```text
I would first inventory the dependency graph and identify high-risk
shared libraries. I would establish approved versions through a
parent/BOM where appropriate, test migration waves and automate
security/update reporting. I would avoid a forced big-bang upgrade.
```

## 132. Security Team Blocks a Transitive CVE

Answer:

```text
I would trace the dependency path, determine exploitability and
whether a fixed compatible version exists. If available, I would
upgrade or manage the version and validate the application. If no
safe upgrade exists, I would document compensating controls and a
risk-owned exception rather than forcing an incompatible version.
```

## 133. Build Plugin Upgrade Breaks 200 Pipelines

Answer:

```text
I would stop broad rollout, compare the effective plugin
configuration and JDK/Maven compatibility, identify the breaking
change, pin the last known-good version and test the new version in
representative projects before a staged rollout.
```

## 134. Artifactory Is Healthy but Maven Cannot Resolve Plugins

Answer:

```text
I would distinguish application dependency resolution from plugin
resolution. I would inspect pluginRepository configuration, mirror
settings, plugin coordinates, repository permissions, credentials,
TLS and the effective settings/POM.
```

## 135. Local Maven Build Uses One Version, CI Another

Answer:

```text
I would inspect dependency:tree and effective-pom, then compare
Maven/JDK versions, settings, profiles, local repository contents and
CI cache behavior. I would eliminate hidden local state and make the
build inputs explicit.
```

## 136. A Developer Adds an Exclusion to Fix a CVE

Answer:

```text
I would not approve the exclusion automatically. I would verify
whether the dependency is actually required at runtime, determine
whether a safe version can be selected through dependency management,
and run tests. Removing a transitive dependency can create a different
production failure.
```

---

# PART LX — GOLDEN RULES

## 137. Rules

```text
1. Understand dependencies and plugins as different concepts.

2. Dependencies affect application/test classpaths.

3. Plugins execute build logic.

4. Treat plugins as part of the software supply chain.

5. Declare application-used direct dependencies explicitly.

6. Understand transitive dependencies.

7. Inspect dependency:tree during conflicts.

8. Control important dependency versions.

9. Understand dependencyManagement.

10. Understand BOM imports.

11. Use scopes intentionally.

12. Avoid system-scoped dependencies unless there is a justified
    exceptional requirement.

13. Use exclusions carefully.

14. Prefer version management over unnecessary exclusions.

15. Test every dependency override.

16. Scan transitive dependencies.

17. Review dependency licenses.

18. Protect against dependency confusion.

19. Use approved internal repositories.

20. Prefer controlled mirrors in enterprise environments.

21. Cache dependencies and plugins appropriately.

22. Control plugin versions.

23. Understand plugin goals.

24. Understand lifecycle bindings.

25. Understand plugin executions.

26. Give executions meaningful IDs.

27. Understand pluginManagement.

28. Remember pluginManagement does not by itself mean the plugin
    executes.

29. Use Maven Wrapper.

30. Control the JDK.

31. Keep Maven configuration reproducible.

32. Keep secrets out of POM files.

33. Use least-privilege CI credentials.

34. Protect plugin repositories.

35. Treat plugin upgrades like dependency upgrades.

36. Use effective-pom when inherited behavior is unclear.

37. Use effective-settings when repository behavior is unclear.

38. Use debug logs carefully.

39. Keep release artifacts immutable.

40. Avoid mutable snapshots in production release paths.

41. Do not publish failed builds.

42. Build once and promote the same artifact.

43. Scan application and build supply chains where tooling supports it.

44. Generate SBOMs where required.

45. Compare actual runtime classpaths when debugging runtime linkage
    failures.

46. Do not assume the POM alone describes the runtime environment.

47. Monitor dependency and plugin resolution failures.

48. If many teams fail simultaneously, investigate shared repository,
    DNS, TLS and network infrastructure.

49. Stage large dependency/plugin upgrades.

50. Validate exact Maven, JDK, plugin and dependency behavior for the
    versions actually used in production.
```

---

# END OF 06-Maven-Dependencies-and-Plugins.md
