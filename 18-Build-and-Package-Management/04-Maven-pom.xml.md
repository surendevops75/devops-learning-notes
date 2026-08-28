# 18-Build-and-Package-Management
# 04-Maven-pom.xml

## 1. Purpose

`pom.xml` is the central configuration file of a Maven project.

For production DevOps work, understanding the POM means understanding
how Maven knows:

```text
what the project is
what it depends on
how it is built
which plugins execute
which Java version is used
which repositories are used
which artifact is produced
where artifacts are published
how profiles alter behavior
how parent configuration is inherited
how modules are assembled
```

A POM should be treated as production build infrastructure, not merely
as an XML file containing dependency entries.

---

# PART I — POM FUNDAMENTALS

## 2. What Does POM Mean?

POM means:

```text
Project Object Model
```

Maven reads the POM and builds an effective model used during the
build.

Conceptually:

```text
pom.xml
   |
   v
Maven Model
   |
   +--> Project identity
   +--> Dependencies
   +--> Build
   +--> Plugins
   +--> Repositories
   +--> Profiles
   |
   v
Maven Lifecycle
```

---

## 3. Minimal POM

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

## 4. modelVersion

Typical:

```xml
<modelVersion>4.0.0</modelVersion>
```

This identifies the Maven POM model version.

It is not the version of the application.

---

# PART II — PROJECT IDENTITY

## 5. groupId

Example:

```xml
<groupId>com.company.payments</groupId>
```

It identifies the organization or logical namespace.

---

## 6. artifactId

Example:

```xml
<artifactId>payment-service</artifactId>
```

It identifies the project artifact.

---

## 7. version

Example:

```xml
<version>4.2.1</version>
```

It identifies the project version.

---

## 8. Coordinates

Together:

```text
groupId
artifactId
version
```

form the primary Maven coordinates.

Example:

```text
com.company.payments:payment-service:4.2.1
```

---

## 9. packaging

Example:

```xml
<packaging>jar</packaging>
```

Common values:

```text
jar
war
pom
```

If omitted, Maven commonly uses `jar` as the default.

---

# PART III — PARENT POM

## 10. What Is a Parent?

A POM can inherit configuration from another POM.

Example:

```xml
<parent>
    <groupId>com.company.platform</groupId>
    <artifactId>company-parent</artifactId>
    <version>5.0.0</version>
</parent>
```

---

## 11. Why Use a Parent?

Centralize:

```text
plugin versions
dependency management
properties
repositories
common build configuration
```

---

## 12. Enterprise Parent

Example architecture:

```text
company-parent
 |
+--> Java version
+--> Compiler configuration
+--> Test configuration
+--> Approved plugins
+--> Dependency management
 |
 +--> service-a
 +--> service-b
 +--> service-c
```

This reduces duplication.

---

## 13. Parent Version

Parent versions should be controlled.

Avoid:

```text
unknown dynamic parent
```

Use an explicit, tested version.

---

# PART IV — INHERITANCE

## 14. What Can Be Inherited?

Depending on Maven configuration, child POMs can inherit:

```text
properties
dependencyManagement
dependencies
plugin configuration
plugin management
repositories
build configuration
```

Not every element behaves identically.

---

## 15. Why Inheritance Matters

A child POM may appear small:

```xml
<artifactId>payment-service</artifactId>
```

because important configuration exists in the parent.

When troubleshooting, inspect the effective POM.

---

# PART V — EFFECTIVE POM

## 16. What Is Effective POM?

The effective POM is the resulting model after Maven combines relevant
configuration.

Use:

```bash
mvn help:effective-pom
```

---

## 17. Why It Matters

If a build behaves unexpectedly:

```text
pom.xml
    +
parent
    +
profiles
    +
plugin defaults
    |
    v
effective POM
```

The effective POM can reveal inherited configuration.

---

# PART VI — PROPERTIES

## 18. Maven Properties

Example:

```xml
<properties>
    <java.version>21</java.version>
    <maven.compiler.release>21</maven.compiler.release>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>
```

---

## 19. Why Use Properties?

Centralize values:

```text
version
Java release
plugin configuration
encoding
dependency versions
```

---

## 20. Property Reference

Example:

```xml
<version>${some.version}</version>
```

This avoids repeating the same value.

---

## 21. Property Naming

Prefer clear names:

```text
<java.version>
<spring.version>
<maven.compiler.release>
```

Avoid cryptic names.

---

# PART VII — DEPENDENCIES

## 22. Dependency Block

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

## 23. Dependency Coordinates

A dependency normally identifies:

```text
groupId
artifactId
version
```

Additional attributes can control:

```text
scope
optional
classifier
type
exclusions
```

---

# PART VIII — DEPENDENCY SCOPE

## 24. Common Scopes

```text
compile
provided
runtime
test
system
```

Use scopes deliberately.

---

## 25. compile

A compile dependency is generally available to:

```text
compile
test
runtime
```

and participates in the normal application dependency model.

---

## 26. provided

A provided dependency is expected to be supplied by the runtime or
container.

Typical examples can include APIs supplied by an application server.

---

## 27. runtime

Runtime dependency is needed during execution but may not be required
to compile application source.

Example concept:

```text
JDBC API
+
database driver at runtime
```

---

## 28. test

Test-only dependency:

```xml
<scope>test</scope>
```

It should not normally become a production runtime dependency.

---

# PART IX — OPTIONAL DEPENDENCIES

## 29. optional

Concept:

```xml
<optional>true</optional>
```

An optional dependency does not automatically propagate to consumers
in the same way as a normal transitive dependency.

Use this feature only when the dependency semantics actually require it.

---

# PART X — EXCLUSIONS

## 30. Exclusion

Example:

```xml
<exclusions>
    <exclusion>
        <groupId>org.example</groupId>
        <artifactId>old-library</artifactId>
    </exclusion>
</exclusions>
```

---

## 31. Why Exclude?

Common reasons:

```text
conflicting version
unwanted transitive dependency
security issue
runtime-provided dependency
```

---

## 32. Exclusion Risk

Removing a dependency can cause:

```text
ClassNotFoundException
NoClassDefFoundError
NoSuchMethodError
```

Always validate the runtime.

---

# PART XI — DEPENDENCY MANAGEMENT

## 33. dependencyManagement

Example:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>example-library</artifactId>
            <version>1.2.3</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

---

## 34. dependencyManagement vs dependencies

Important distinction:

```text
dependencies
 |
declares dependency usage

dependencyManagement
 |
centralizes dependency information
```

A dependency-management entry does not by itself mean the project
uses that dependency.

---

## 35. Child Usage

Parent:

```xml
<dependencyManagement>
    ...
</dependencyManagement>
```

Child:

```xml
<dependency>
    <groupId>org.example</groupId>
    <artifactId>example-library</artifactId>
</dependency>
```

The child can inherit the managed version.

---

# PART XII — BOM

## 36. Bill of Materials

A BOM is a POM designed to centrally manage compatible dependency
versions.

---

## 37. Importing a BOM

Concept:

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

## 38. Why BOMs?

A BOM can provide:

```text
compatible dependency versions
centralized upgrades
less version duplication
```

---

# PART XIII — BUILD SECTION

## 39. build

The build section configures build behavior.

Example:

```xml
<build>
    <plugins>
        ...
    </plugins>
</build>
```

---

## 40. Common Build Areas

```text
directory configuration
resources
testResources
plugins
pluginManagement
finalName
```

---

# PART XIV — PLUGINS

## 41. Plugin

Maven plugins perform build tasks.

Example:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>...</version>
</plugin>
```

---

## 42. Why Plugin Versions Matter

A plugin is executable build logic.

Changing it can change:

```text
compilation
packaging
testing
verification
```

Therefore plugin versions should be controlled.

---

# PART XV — pluginManagement

## 43. pluginManagement

`pluginManagement` centralizes plugin configuration and versions for
plugins used by the project hierarchy.

Concept:

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

## 44. pluginManagement vs plugins

Important distinction:

```text
pluginManagement
 |
defines managed configuration

plugins
 |
activates plugin usage
```

A plugin generally needs to be referenced in the effective build to
execute.

---

# PART XVI — COMPILER CONFIGURATION

## 45. Java Release

Modern Maven projects can use:

```xml
<properties>
    <maven.compiler.release>21</maven.compiler.release>
</properties>
```

---

## 46. Build JDK

The CI runner must have a JDK capable of compiling for the selected
release.

Example:

```text
POM -> release 21
CI -> JDK 21
```

---

# PART XVII — RESOURCES

## 47. Resources

Example:

```xml
<resources>
    <resource>
        <directory>src/main/resources</directory>
    </resource>
</resources>
```

Resources are files packaged alongside application classes according to
the configured build.

---

## 48. Filtering

Maven can support resource filtering.

Be careful with:

```text
secrets
binary files
configuration
```

Do not accidentally inject secrets into artifacts.

---

# PART XVIII — TEST RESOURCES

## 49. Test Resources

Example:

```xml
<testResources>
    <testResource>
        <directory>src/test/resources</directory>
    </testResource>
</testResources>
```

Used for test-specific resources.

---

# PART XIX — FINAL NAME

## 50. finalName

Example:

```xml
<build>
    <finalName>payment-service</finalName>
</build>
```

This controls the generated artifact base name.

Use predictable naming, but avoid hiding version identity if your
release process relies on Maven coordinates.

---

# PART XX — PROFILES

## 51. Profile

Profiles allow conditional configuration.

Example:

```xml
<profiles>
    <profile>
        <id>ci</id>
        ...
    </profile>
</profiles>
```

---

## 52. Activation

Example:

```bash
mvn -Pci verify
```

Profiles can also be activated through configured conditions.

---

## 53. Profile Anti-Pattern

Avoid:

```text
prod profile contains passwords
```

Instead:

```text
POM
 |
build configuration

Secret Manager
 |
runtime secret
```

---

# PART XXI — REPOSITORIES

## 54. repositories

A POM can declare dependency repositories.

Concept:

```xml
<repositories>
    <repository>
        <id>company-virtual</id>
        <url>https://repo.example.com/maven/virtual</url>
    </repository>
</repositories>
```

---

## 55. Enterprise Recommendation

For enterprise environments, prefer a controlled internal repository
architecture rather than allowing every build to reach arbitrary public
repositories.

---

# PART XXII — PLUGIN REPOSITORIES

## 56. pluginRepositories

Maven can define repositories for plugin resolution.

Concept:

```xml
<pluginRepositories>
    <pluginRepository>
        <id>company-plugins</id>
        <url>https://repo.example.com/maven/plugins</url>
    </pluginRepository>
</pluginRepositories>
```

---

## 57. Security

Plugin repositories are important because plugins execute during builds.

Treat them as supply-chain boundaries.

---

# PART XXIII — DISTRIBUTION MANAGEMENT

## 58. distributionManagement

This section can define where artifacts are deployed.

Concept:

```xml
<distributionManagement>
    <repository>
        <id>releases</id>
        <url>https://repo.example.com/releases</url>
    </repository>

    <snapshotRepository>
        <id>snapshots</id>
        <url>https://repo.example.com/snapshots</url>
    </snapshotRepository>
</distributionManagement>
```

---

## 59. Repository ID and Credentials

The server credentials configured in Maven settings should correspond
to the repository ID expected by the deployment configuration.

---

# PART XXIV — SCM

## 60. scm

The POM can describe source-control information.

Concept:

```xml
<scm>
    <connection>...</connection>
    <developerConnection>...</developerConnection>
    <url>...</url>
</scm>
```

This helps tooling understand the source repository.

---

# PART XXV — LICENSE

## 61. licenses

Projects can declare licensing information.

This is useful for:

```text
metadata
distribution
compliance
```

---

# PART XXVI — DEVELOPERS AND CONTRIBUTORS

## 62. Metadata

A POM can contain:

```text
developers
contributors
organization
url
description
```

These fields are generally metadata rather than build-critical
configuration.

---

# PART XXVII — MULTI-MODULE POM

## 63. modules

An aggregator POM can list modules.

Example:

```xml
<modules>
    <module>payment-api</module>
    <module>payment-core</module>
    <module>payment-app</module>
</modules>
```

---

## 64. Directory

```text
payment-platform/
├── pom.xml
├── payment-api/
├── payment-core/
└── payment-app/
```

---

## 65. Aggregation

The root POM can execute builds across modules.

```text
Root
 |
+--> API
+--> Core
+--> App
```

---

# PART XXVIII — PARENT VS AGGREGATOR

## 66. Important Distinction

Parent:

```text
inherit configuration
```

Aggregator:

```text
build modules together
```

One POM can serve both roles, but the concepts are different.

---

## 67. Production Implication

When troubleshooting a multi-module project, determine:

```text
which POM is parent?
which POM is aggregator?
which configuration is inherited?
```

---

# PART XXIX — MODULE VERSION MANAGEMENT

## 68. Central Version

A parent can centralize project properties and dependency versions.

Example:

```xml
<properties>
    <jackson.version>...</jackson.version>
</properties>
```

---

## 69. Avoid Version Duplication

Bad:

```text
module-a -> library 1.2
module-b -> library 1.2
module-c -> library 1.3
```

Better:

```text
parent
 |
v
managed version
```

---

# PART XXX — PLUGIN CONFIGURATION

## 70. Compiler Plugin

Concept:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>...</version>
    <configuration>
        <release>21</release>
    </configuration>
</plugin>
```

---

## 71. Surefire

Concept:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>...</version>
</plugin>
```

Used commonly for unit tests.

---

## 72. Failsafe

Used commonly for integration tests.

Typical execution is configured so integration tests run in an
appropriate later lifecycle phase.

---

# PART XXXI — JAR PACKAGING

## 73. Standard JAR

```xml
<packaging>jar</packaging>
```

Output:

```text
target/payment-service-1.0.0.jar
```

---

## 74. Executable JAR

Some applications require additional plugin configuration to create
an executable/fat/uber JAR or another runtime-specific layout.

Do not assume every JAR is executable with:

```bash
java -jar
```

---

# PART XXXII — WAR PACKAGING

## 75. WAR

Example:

```xml
<packaging>war</packaging>
```

Used for web application deployment models that expect WAR artifacts.

---

# PART XXXIII — POM PACKAGING

## 76. POM Packaging

```xml
<packaging>pom</packaging>
```

Common uses:

```text
parent POM
aggregator POM
BOM
```

---

# PART XXXIV — VERSION PROPERTIES

## 77. Dependency Version Property

Example:

```xml
<properties>
    <example.version>1.2.3</example.version>
</properties>
```

Then:

```xml
<version>${example.version}</version>
```

---

## 78. Benefits

```text
centralized upgrades
less duplication
easy review
```

---

# PART XXXV — ENVIRONMENT CONFIGURATION

## 79. POM Is Not a Secret Store

Do not place:

```text
DB_PASSWORD
AWS_SECRET_ACCESS_KEY
ARTIFACTORY_PASSWORD
```

in the POM.

---

## 80. Better Model

```text
POM
 |
build configuration

CI Secret Store
 |
credentials

Runtime Secret Manager
 |
application secrets
```

---

# PART XXXVI — SETTINGS VS POM

## 81. POM

Generally project-level:

```text
dependencies
plugins
build lifecycle configuration
project metadata
```

---

## 82. settings.xml

Generally user/machine/CI-specific:

```text
mirrors
servers
credentials
environment-specific settings
```

Do not rely on undocumented developer-local settings for a production
build.

---

# PART XXXVII — MAVEN MIRROR

## 83. Mirror

Corporate Maven setup may redirect repository access:

```text
Developer / CI
 |
v
Corporate Mirror
 |
+--> Internal
+--> Approved External
```

This improves:

```text
control
caching
availability
security
```

---

# PART XXXVIII — POM AND ARTIFACTORY

## 84. Dependency Flow

```text
pom.xml
 |
v
Artifactory Virtual
 |
+--> Local
+--> Remote
```

---

## 85. Publishing Flow

```text
Maven
 |
v
Package
 |
v
Artifactory Release Repository
```

---

## 86. CI Credentials

Use:

```text
scoped token
service account
```

rather than a human admin credential.

---

# PART XXXIX — POM AND CI

## 87. Jenkins

Example:

```bash
./mvnw -B clean verify
```

Then:

```text
publish artifact
```

---

## 88. GitHub Actions

Typical sequence:

```text
checkout
 |
setup JDK
 |
Maven Wrapper
 |
verify
 |
scan
 |
publish
```

---

# PART XL — POM REPRODUCIBILITY

## 89. Control

A reproducible POM strategy controls:

```text
parent version
dependency versions
plugin versions
JDK
Maven version
repositories
profiles
```

---

## 90. Hidden Configuration

Avoid builds depending on:

```text
developer ~/.m2 settings
unknown local repository
uncontrolled environment variables
manual plugin installations
```

---

# PART XLI — POM SECURITY

## 91. Plugin Security

Plugins execute code.

Control:

```text
plugin source
plugin version
plugin configuration
```

---

## 92. Dependency Security

Scan:

```text
direct
transitive
```

dependencies.

---

## 93. Repository Security

Use:

```text
HTTPS
authentication
authorization
approved repositories
```

---

# PART XLII — POM TROUBLESHOOTING

## 94. Unexpected Configuration

Run:

```bash
mvn help:effective-pom
```

---

## 95. Dependency Conflict

Run:

```bash
mvn dependency:tree
```

---

## 96. Effective Settings

Run:

```bash
mvn help:effective-settings
```

---

## 97. Debug Build

Use:

```bash
mvn -X
```

when normal logs are insufficient.

---

## 98. Repository Failure

Check:

```text
POM repository
settings mirror
server credentials
DNS
TLS
network
artifact existence
```

---

# PART XLIII — COMMON POM MISTAKES

## 99. Hard-Coded Secrets

Bad:

```xml
<password>secret</password>
```

Never commit this.

---

## 100. Uncontrolled Versions

Bad:

```text
many modules
many different dependency versions
```

Use centralized management where appropriate.

---

## 101. Unpinned Plugins

Build behavior can change when plugin versions change.

Control important plugin versions.

---

## 102. Too Many Profiles

Excessive profiles can create:

```text
configuration complexity
unexpected activation
environment drift
```

---

## 103. Giant Parent POM

A parent that controls everything can become difficult to understand.

Keep centralized configuration purposeful.

---

# PART XLIV — PRODUCTION POM DESIGN

## 104. Recommended Structure

A maintainable enterprise POM commonly separates:

```text
Project Metadata
Parent
Properties
Dependencies
Dependency Management
Build
Plugin Management
Plugins
Profiles
Repository/Publishing Metadata
```

The exact order can vary; consistency is more important than a single
mandatory layout.

---

## 105. Example Enterprise POM

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
           http://maven.apache.org/POM/4.0.0
           https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.company.platform</groupId>
        <artifactId>company-parent</artifactId>
        <version>5.0.0</version>
    </parent>

    <groupId>com.company.payments</groupId>
    <artifactId>payment-service</artifactId>
    <version>4.2.1</version>

    <properties>
        <java.version>21</java.version>
        <maven.compiler.release>21</maven.compiler.release>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.example</groupId>
            <artifactId>example-library</artifactId>
            <version>1.2.3</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- approved, version-controlled plugins -->
        </plugins>
    </build>

</project>
```

The actual enterprise parent, dependency and plugin choices should be
based on the organization's approved platform.

---

# PART XLV — PRODUCTION POM ARCHITECTURE

## 106. Layered Model

```text
Corporate Parent
       |
       v
Service Parent
       |
       v
Application POM
       |
       +--> Dependencies
       +--> Plugins
       +--> Build
       |
       v
Artifact
```

This can provide layered governance without putting every detail into
every service POM.

---

# PART XLVI — POM AND VERSIONING

## 107. Release

Example:

```text
payment-service 4.2.1
```

CI should associate:

```text
Git SHA
POM version
artifact
container image
deployment
```

---

## 108. Snapshot

Development:

```text
4.3.0-SNAPSHOT
```

Release:

```text
4.3.0
```

Do not treat mutable snapshots as production release artifacts.

---

# PART XLVII — POM AND REPRODUCIBLE ARTIFACTS

## 109. Build Once

```text
POM
 |
v
Maven Build
 |
v
payment-service-4.2.1.jar
 |
v
Artifact Repository
 |
v
Promotion
```

Avoid rebuilding the same source independently for each environment.

---

# PART XLVIII — INCIDENT SCENARIOS

## 110. Scenario: Dependency Version Unexpected

Symptoms:

```text
POM says A 2.0
runtime contains A 1.8
```

Investigate:

```bash
mvn dependency:tree
```

Then inspect:

```text
parent
dependencyManagement
transitive dependencies
exclusions
packaging
runtime image
```

---

## 111. Scenario: Plugin Behavior Changed

Check:

```text
effective POM
plugin version
parent POM
CI Maven version
JDK
```

---

## 112. Scenario: Local Build Works, CI Fails

Compare:

```text
Maven version
JDK
settings.xml
repository mirror
credentials
environment
local cache
```

---

## 113. Scenario: Artifact Cannot Be Published

Check:

```text
distributionManagement
repository URL
server ID
CI credential
repository permissions
artifact version policy
```

---

## 114. Scenario: Build Suddenly Cannot Resolve Dependencies

Check:

```text
repository availability
mirror
DNS
TLS
credentials
artifact coordinates
cache
```

---

# PART XLIX — POM REVIEW CHECKLIST

## 115. Identity

```text
[ ] groupId
[ ] artifactId
[ ] version
[ ] packaging
```

## 116. Parent

```text
[ ] parent is intentional
[ ] parent version controlled
[ ] inherited configuration understood
```

## 117. Dependencies

```text
[ ] versions controlled
[ ] scopes correct
[ ] transitive dependencies reviewed
[ ] unnecessary dependencies removed
```

## 118. Build

```text
[ ] Java version controlled
[ ] plugin versions controlled
[ ] tests configured
[ ] resources reviewed
```

## 119. Repositories

```text
[ ] approved repository
[ ] HTTPS
[ ] CI credentials protected
[ ] publishing repository correct
```

## 120. Security

```text
[ ] no secrets
[ ] dependency scanning
[ ] plugin governance
[ ] artifact integrity
```

---

# PART L — INTERVIEW PREPARATION

## 121. What Is pom.xml?

Answer:

```text
pom.xml is Maven's Project Object Model. It defines project identity
and can configure dependencies, dependency management, plugins,
build behavior, repositories, profiles, modules and artifact
publishing.
```

## 122. What Are Maven Coordinates?

Answer:

```text
The primary Maven coordinates are groupId, artifactId and version.
They uniquely identify a project artifact within the repository
ecosystem.
```

## 123. dependencies vs dependencyManagement?

Answer:

```text
dependencies declares what the project consumes. dependencyManagement
centralizes dependency information such as versions for the project
hierarchy, but an entry in dependencyManagement does not by itself
mean the project consumes that dependency.
```

## 124. plugins vs pluginManagement?

Answer:

```text
plugins configure plugin usage in the build. pluginManagement provides
centralized plugin versions and configuration that can be inherited
by modules when those plugins are actually used.
```

## 125. Parent vs Aggregator?

Answer:

```text
A parent provides inheritance. An aggregator groups modules so Maven
can build them together. A single POM can perform both roles, but
they solve different problems.
```

## 126. How Do You Troubleshoot an Unexpected Maven Configuration?

Answer:

```text
I inspect the effective POM with mvn help:effective-pom, then trace
the parent hierarchy, profiles, plugin configuration and properties.
For dependency issues I also use mvn dependency:tree.
```

## 127. How Do You Secure a POM?

Answer:

```text
I avoid secrets in source control, use approved repositories, control
dependency and plugin versions, use secure CI credentials, scan
dependencies and treat build plugins as supply-chain dependencies.
```

## 128. How Do You Make a POM Reproducible?

Answer:

```text
I control parent, dependency and plugin versions, use Maven Wrapper,
standardize the JDK and repository configuration, keep build
configuration in source control and avoid hidden developer-local
settings.
```

## 129. What Is a BOM?

Answer:

```text
A BOM is a Maven POM that centrally manages compatible versions of a
set of dependencies. It can be imported through dependencyManagement.
```

---

# PART LI — SENIOR SCENARIOS

## 130. Hundreds of Services Have Different POM Standards

Answer:

```text
I would establish an enterprise parent or approved build templates
for common controls such as Java versions, plugin versions, security
and repository configuration. I would migrate services incrementally
and allow controlled exceptions rather than forcing a risky
big-bang migration.
```

## 131. A Parent POM Upgrade Breaks Many Services

Answer:

```text
I would identify the inherited changes using the effective POM,
compare plugin and dependency versions, reproduce representative
services, roll back or pin the parent if necessary, and introduce the
new parent through tested upgrade waves.
```

## 132. A Dependency Version Is Correct in the POM but Wrong at Runtime

Answer:

```text
I would inspect the resolved dependency tree and packaged artifact,
then check transitive dependencies, dependency management,
exclusions, scopes and the runtime image. I would verify the actual
runtime classpath rather than assuming the POM alone determines it.
```

## 133. Maven Cannot Deploy to Artifactory

Answer:

```text
I would verify distributionManagement, repository URL, server ID,
CI credentials, token scope, repository permissions and whether the
target repository accepts that version type. I would also confirm
that the failure is authentication versus authorization.
```

---

# PART LII — GOLDEN RULES

## 134. Rules

```text
1. Treat pom.xml as production build infrastructure.

2. Understand Maven coordinates.

3. Keep the POM readable.

4. Use an intentional parent hierarchy.

5. Understand inheritance.

6. Distinguish parent from aggregator.

7. Use properties to centralize meaningful configuration.

8. Control dependency versions.

9. Understand dependencyManagement.

10. Understand BOM imports.

11. Use dependency scopes correctly.

12. Do not use exclusions without testing runtime behavior.

13. Control plugin versions.

14. Understand pluginManagement.

15. Do not assume every POM dependency is a runtime dependency.

16. Do not assume every JAR is executable.

17. Keep secrets out of POM files.

18. Keep environment-specific credentials outside source control.

19. Prefer controlled enterprise repositories.

20. Treat plugin repositories as security boundaries.

21. Understand distributionManagement.

22. Use Maven Wrapper.

23. Standardize JDK versions.

24. Inspect effective POM when inherited behavior is unclear.

25. Inspect dependency:tree when dependency behavior is unclear.

26. Use effective-settings when repository configuration is unclear.

27. Use debug logs only when necessary and protect sensitive output.

28. Keep production releases immutable.

29. Do not treat SNAPSHOT artifacts as production releases.

30. Avoid excessive profiles.

31. Avoid giant parent POMs.

32. Build once and promote the same artifact.

33. Track POM version to Git commit and artifact.

34. Scan direct and transitive dependencies.

35. Treat build plugins as supply-chain dependencies.

36. Use least-privilege repository credentials.

37. Separate build configuration from runtime configuration.

38. Validate POM changes through CI.

39. Test parent and dependency upgrades before broad rollout.

40. Validate exact Maven, JDK and plugin behavior for the versions
    actually used in production.
```

---

# END OF 04-Maven-pom.xml.md
