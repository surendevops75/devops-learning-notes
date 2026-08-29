# Maven-Multi-Module-Projects

## 1. Purpose

Maven multi-module projects allow a large application or platform to
be organized into multiple Maven projects that are built together as a
reactor.

A production DevOps engineer must understand more than:

```bash
mvn clean install
```

The important concepts are:

```text
parent POM
aggregator POM
module
reactor
inter-module dependency
build order
inheritance
dependencyManagement
pluginManagement
BOM
module isolation
parallel builds
selective builds
incremental builds
CI optimization
artifact publishing
version management
release strategy
troubleshooting
```

This file covers multi-module Maven architecture from fundamentals to
enterprise production patterns.

---

# PART I — MULTI-MODULE FUNDAMENTALS

## 2. What Is a Multi-Module Maven Project?

A multi-module Maven project contains multiple Maven projects managed
from a higher-level POM.

Example:

```text
payment-platform/
├── pom.xml
├── payment-api/
│   └── pom.xml
├── payment-core/
│   └── pom.xml
├── payment-db/
│   └── pom.xml
└── payment-app/
    └── pom.xml
```

The root POM coordinates the modules.

---

## 3. Why Use Multi-Module Projects?

Benefits include:

```text
shared configuration
logical separation
dependency relationships
consistent versions
single build command
centralized CI
```

---

## 4. Typical Enterprise Structure

```text
platform
 |
+--> api
+--> domain
+--> persistence
+--> application
+--> integration
```

Each module has its own:

```text
pom.xml
src/
target/
```

---

# PART II — ROOT POM

## 5. Root POM

Example:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="
           http://maven.apache.org/POM/4.0.0
           https://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>com.company.payment</groupId>
    <artifactId>payment-platform</artifactId>
    <version>1.0.0</version>

    <packaging>pom</packaging>

    <modules>
        <module>payment-api</module>
        <module>payment-core</module>
        <module>payment-db</module>
        <module>payment-app</module>
    </modules>
</project>
```

---

## 6. Why packaging=pom?

A root multi-module aggregator commonly uses:

```xml
<packaging>pom</packaging>
```

because it is coordinating modules rather than producing an application
JAR.

---

# PART III — MODULE POM

## 7. Module POM

Example:

```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.company.payment</groupId>
        <artifactId>payment-platform</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>payment-core</artifactId>

</project>
```

The module can inherit configuration from the parent.

---

# PART IV — PARENT VS AGGREGATOR

## 8. Parent

A parent provides inheritance.

```text
Parent
 |
+--> properties
+--> dependencyManagement
+--> pluginManagement
+--> common configuration
```

---

## 9. Aggregator

An aggregator lists modules:

```xml
<modules>
    <module>payment-api</module>
    <module>payment-core</module>
</modules>
```

It allows Maven to coordinate the build.

---

## 10. One POM Can Do Both

A root POM commonly acts as:

```text
parent
+
aggregator
```

But the concepts are different.

---

## 11. Why Distinguish Them?

A POM can be a parent without aggregating modules.

A POM can aggregate modules without being their parent.

This distinction is important when designing reusable enterprise build
hierarchies.

---

# PART V — INHERITANCE

## 12. Inherited Properties

Parent:

```xml
<properties>
    <java.version>21</java.version>
</properties>
```

Child can use the inherited property.

---

## 13. Inherited Dependency Management

Parent:

```xml
<dependencyManagement>
    ...
</dependencyManagement>
```

Children can use managed versions without duplicating them.

---

## 14. Inherited Plugin Management

Parent:

```xml
<pluginManagement>
    ...
</pluginManagement>
```

Children can use centrally managed plugin versions/configuration.

---

# PART VI — MODULE DEPENDENCIES

## 15. Example

```text
payment-app
     |
     v
payment-core
     |
     v
payment-api
```

The application module can declare:

```xml
<dependency>
    <groupId>com.company.payment</groupId>
    <artifactId>payment-core</artifactId>
    <version>${project.version}</version>
</dependency>
```

The exact version strategy should match the project's parent and
release design.

---

## 16. Reactor Awareness

When modules are part of the same Maven reactor, Maven can build
required upstream modules in the correct order.

---

# PART VII — REACTOR

## 17. What Is the Maven Reactor?

The reactor is Maven's mechanism for building a set of projects together.

Concept:

```text
Root
 |
+--> A
+--> B
+--> C
```

Maven determines an appropriate build order based on module
relationships.

---

## 18. Dependency-Based Order

Suppose:

```text
A -> B
```

Then B must be built before A when both participate in the reactor.

---

## 19. Graph

```text
A
|
v
B
|
v
C
```

Build order:

```text
C
 |
v
B
 |
v
A
```

---

# PART VIII — MODULE BUILD ORDER

## 20. Why Build Order Matters

If module A requires module B:

```text
A
 |
v
B
```

B's artifact/classes must be available for the relevant build work in A.

---

## 21. Incorrect Assumption

Do not assume directory order alone determines build order.

Maven considers reactor relationships.

---

# PART IX — MULTI-MODULE COMMANDS

## 22. Build All

From the root:

```bash
mvn clean verify
```

This can build the complete reactor.

---

## 23. Install All

```bash
mvn clean install
```

This builds and installs reactor artifacts into the local repository.

---

## 24. Deploy All

```bash
mvn clean deploy
```

This can publish the configured artifacts to a remote repository.

Use carefully in CI because it may publish every module.

---

# PART X — SELECTIVE BUILDS

## 25. -pl

Maven supports project-list selection.

Example:

```bash
mvn -pl payment-app verify
```

This selects the specified module/project.

---

## 26. -am

`-am` means:

```text
also make required projects
```

Example:

```bash
mvn -pl payment-app -am verify
```

This can build the selected module and its required reactor projects.

---

## 27. -amd

`-amd` means:

```text
also make dependents
```

It can be useful when investigating changes that affect downstream
modules.

---

# PART XI — SELECTIVE CI BUILDS

## 28. Why Selective Builds?

Large repositories may contain:

```text
100+ modules
```

Building everything for every change can waste:

```text
time
CPU
memory
CI minutes
```

---

## 29. Selective Strategy

```text
Changed module
 |
v
Required upstream modules
 |
v
Tests
 |
v
Package
```

---

## 30. Do Not Over-Optimize

A selective build must still include all required dependencies and
quality gates.

Incorrect dependency detection can produce false confidence.

---

# PART XII — MULTI-MODULE VERSIONING

## 31. Shared Version

Common design:

```text
platform 1.0.0
 |
+--> api 1.0.0
+--> core 1.0.0
+--> app 1.0.0
```

---

## 32. Why Shared Version?

Benefits:

```text
simple release coordination
consistent artifact set
easy traceability
```

---

## 33. Independent Versions

Another model:

```text
api 2.1.0
core 5.4.0
app 8.2.0
```

Useful when modules have independent release lifecycles.

---

## 34. Choose Deliberately

Shared versions are simpler for tightly coupled applications.

Independent versions can be useful for reusable libraries.

Do not mix models without clear release governance.

---

# PART XIII — INTERNAL MODULE DEPENDENCY

## 35. Example

```text
payment-app
 |
v
payment-service
 |
v
payment-core
```

The POM can express these relationships.

---

## 36. Dependency Graph

```text
payment-app
 |
+--> payment-service
      |
      +--> payment-core
```

Maven reactor ordering follows these relationships.

---

# PART XIV — CYCLIC DEPENDENCIES

## 37. Bad Graph

```text
A
 |
v
B
 |
v
A
```

This is a cycle.

---

## 38. Why Cycles Are Bad

They create:

```text
build-order problems
architecture coupling
maintenance difficulty
```

---

## 39. Fixing Cycles

Possible approaches:

```text
extract common module
invert dependency
use interface module
redesign boundaries
```

Example:

```text
A -> common <- B
```

instead of:

```text
A <-> B
```

---

# PART XV — MODULE BOUNDARIES

## 40. Good Boundary

```text
api
 |
public contracts

core
 |
business logic

db
 |
persistence

app
 |
application assembly
```

---

## 41. Bad Boundary

A module should not become a random collection of unrelated classes.

Keep module responsibility clear.

---

# PART XVI — DEPENDENCY MANAGEMENT

## 42. Parent dependencyManagement

Centralize common versions:

```xml
<dependencyManagement>
    <dependencies>
        ...
    </dependencies>
</dependencyManagement>
```

---

## 43. Module Dependency

Child:

```xml
<dependencies>
    <dependency>
        <groupId>com.company.payment</groupId>
        <artifactId>payment-core</artifactId>
    </dependency>
</dependencies>
```

The version can be inherited/managed when appropriate.

---

# PART XVII — BOM

## 44. Enterprise BOM

A platform can use:

```text
company-bom
 |
+--> logging
+--> HTTP client
+--> JSON
+--> testing
```

Modules import or inherit the approved dependency versions.

---

# PART XVIII — PLUGIN MANAGEMENT

## 45. Parent pluginManagement

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

## 46. Child Plugin

The child can declare plugin usage while inheriting managed version and
configuration.

---

# PART XIX — COMMON PLUGINS

## 47. Compiler

```text
maven-compiler-plugin
```

---

## 48. Surefire

```text
maven-surefire-plugin
```

---

## 49. Failsafe

```text
maven-failsafe-plugin
```

---

## 50. JAR

```text
maven-jar-plugin
```

---

## 51. Source/Javadoc

Useful for reusable library publishing.

---

# PART XX — BUILD ORDER AND PLUGINS

## 52. Module-Specific Plugins

A module may have additional plugins.

Example:

```text
api
 |
generate code

core
 |
compile/test

app
 |
package executable
```

The parent can standardize versions while modules define their
specific behavior.

---

# PART XXI — MULTI-MODULE TESTING

## 53. Unit Tests

Each module can have:

```text
src/test/java
```

and its own tests.

---

## 54. Integration Tests

An application module may execute broader integration tests.

```text
module tests
 |
v
integration environment
 |
v
application tests
```

---

## 55. Test Strategy

```text
Module unit tests
       |
       v
Integration tests
       |
       v
End-to-end tests
```

Do not put all expensive testing into every module if it creates
unnecessary CI cost.

---

# PART XXII — MODULE ARTIFACTS

## 56. Library Modules

Example:

```text
payment-core-1.0.0.jar
```

---

## 57. Application Module

Example:

```text
payment-app-1.0.0.jar
```

---

## 58. Parent/Root

Often:

```text
payment-platform-1.0.0.pom
```

---

# PART XXIII — ARTIFACT PUBLISHING

## 59. Publish Every Module?

Not always.

Some modules may be:

```text
internal-only
application-only
reusable libraries
parent/BOM metadata
```

Publishing strategy should match the artifact lifecycle.

---

## 60. Artifactory

Example:

```text
Maven Reactor
 |
+--> api.jar
+--> core.jar
+--> app.jar
+--> platform.pom
 |
v
Artifactory
```

---

# PART XXIV — SNAPSHOTS

## 61. Snapshot Reactor

Development:

```text
1.2.0-SNAPSHOT
```

Modules can reference compatible snapshot versions during development.

---

## 62. Production

Prefer release versions:

```text
1.2.0
```

and immutable repository artifacts.

---

# PART XXV — CI PIPELINE

## 63. Standard

```text
Checkout
 |
v
Maven Wrapper
 |
v
Clean
 |
v
Resolve dependencies
 |
v
Compile
 |
v
Unit tests
 |
v
Integration tests
 |
v
Security
 |
v
Package
 |
v
Publish
```

---

# PART XXVI — PARALLEL BUILDS

## 64. Parallel Reactor

Maven supports parallel build execution options.

Example:

```bash
mvn -T 4 clean verify
```

or thread-count based options appropriate to the build.

---

## 65. Why Parallelize?

Potential benefits:

```text
shorter CI time
better CPU utilization
```

---

## 66. Risks

Poorly designed builds can expose:

```text
race conditions
resource contention
non-thread-safe plugins
test environment conflicts
```

Validate plugin and project compatibility before using aggressive
parallelism.

---

# PART XXVII — CI RESOURCE MANAGEMENT

## 67. Parallel Build Cost

More threads require:

```text
CPU
memory
disk
network
```

---

## 68. Production Rule

Do not choose:

```bash
-T 32
```

just because the runner has 32 CPUs.

Benchmark first.

---

# PART XXVIII — MAVEN CACHE

## 69. Shared Cache

CI can cache:

```text
~/.m2/repository
```

---

## 70. Cache Key

Include relevant inputs:

```text
OS
JDK
Maven
POM hashes
```

---

## 71. Cache Risk

A stale cache can produce confusing behavior.

When diagnosing:

```text
works with cache
fails without cache
```

or vice versa, test with a clean repository.

---

# PART XXIX — REACTOR VS LOCAL REPOSITORY

## 72. Reactor Dependency

When modules are built together, Maven can use reactor outputs rather
than requiring each module to be separately installed first.

---

## 73. Important Distinction

You do not normally need:

```bash
mvn install
```

between every module in the same reactor build.

The reactor coordinates inter-module build relationships.

---

# PART XXX — COMMON ANTI-PATTERN

## 74. Running Install Per Module

Bad CI pattern:

```text
cd module-a
mvn install

cd module-b
mvn install

cd module-c
mvn install
```

This creates unnecessary complexity.

Prefer a correctly configured reactor build.

---

# PART XXXI — MODULE RELEASE

## 75. Release Sequence

```text
Source Tag
 |
v
Clean Reactor Build
 |
v
Tests
 |
v
Security
 |
v
Package
 |
v
Publish
```

---

# PART XXXII — BUILD ONCE

## 76. Artifact Promotion

```text
Reactor Build
 |
v
Artifacts
 |
v
Artifactory
 |
+--> DEV
+--> STAGE
+--> PROD
```

Avoid rebuilding the same source independently per environment.

---

# PART XXXIII — CHANGE IMPACT

## 77. Core Module Change

If:

```text
core
 |
+--> service
      |
      +--> app
```

then a core change can affect:

```text
service
app
```

---

## 78. API Change

A public API change can affect many consumers.

Use module dependency graphs to assess impact.

---

# PART XXXIV — API MODULE

## 79. Why Separate API?

A dedicated API module can contain:

```text
interfaces
DTOs
events
contracts
```

This can reduce accidental dependency on implementation internals.

---

# PART XXXV — CORE MODULE

## 80. Core

Contains:

```text
business logic
domain model
core services
```

Avoid making core depend on infrastructure modules unnecessarily.

---

# PART XXXVI — INFRASTRUCTURE MODULE

## 81. Persistence

Example:

```text
payment-db
 |
database adapters
repositories
migrations
```

Keep infrastructure boundaries explicit.

---

# PART XXXVII — APPLICATION MODULE

## 82. Application

Common role:

```text
assemble modules
configure runtime
produce executable artifact
```

---

# PART XXXVIII — ARCHITECTURE

## 83. Example

```text
                 payment-app
                      |
          +-----------+-----------+
          |                       |
          v                       v
    payment-service        payment-integration
          |
          v
     payment-core
          |
          v
     payment-api
```

Infrastructure:

```text
payment-service
       |
       v
   payment-db
```

The exact architecture depends on the application.

---

# PART XXXIX — DEPENDENCY DIRECTION

## 84. Preferred Direction

Example:

```text
API
 ^
 |
CORE
 ^
 |
SERVICE
 ^
 |
APP
```

The actual arrows depend on which module consumes which API. The key
principle is to maintain deliberate dependency direction and avoid
cycles.

---

# PART XL — MODULE OWNERSHIP

## 85. Ownership

For large enterprises, define ownership:

```text
api -> team A
core -> team B
db -> team C
app -> team D
```

Ownership should include:

```text
code
dependencies
security
release
```

---

# PART XLI — VERSION GOVERNANCE

## 86. Parent Version

Control parent versions.

Example:

```text
company-parent 5.2.0
```

---

## 87. Dependency Versions

Centralize through:

```text
dependencyManagement
BOM
properties
```

---

## 88. Plugin Versions

Centralize through:

```text
pluginManagement
```

---

# PART XLII — MULTI-MODULE SECURITY

## 89. Scan Each Module

Security scanning should consider:

```text
module dependencies
transitive dependencies
```

---

## 90. Final Application

Also scan the final application artifact/container because the final
runtime dependency set may differ from individual module assumptions.

---

# PART XLIII — LICENSES

## 91. Multi-Module License Review

A platform may aggregate dependencies from many modules.

Track:

```text
module
dependency
license
version
```

---

# PART XLIV — SBOM

## 92. SBOM Strategy

An organization can generate:

```text
module SBOM
+
application SBOM
```

The application-level SBOM is especially useful for production
inventory.

---

# PART XLV — TROUBLESHOOTING

## 93. Module Not Found

Check:

```text
<modules>
path
directory
pom.xml
```

---

## 94. Dependency Not Found

Check:

```text
groupId
artifactId
version
module relationship
repository
```

---

## 95. Wrong Build Order

Check:

```text
inter-module dependency declaration
aggregator modules
parent
reactor
```

---

## 96. Cyclic Dependency

Look for:

```text
A -> B
B -> A
```

Refactor module boundaries.

---

## 97. One Module Fails

Determine whether dependent modules are failing because of the upstream
failure.

---

# PART XLVI — REACTOR FAILURE

## 98. Example

```text
core FAIL
 |
v
service SKIPPED/FAIL
 |
v
app SKIPPED/FAIL
```

Do not troubleshoot every downstream failure independently until the
root upstream failure is fixed.

---

# PART XLVII — DEBUGGING

## 99. Dependency Tree

```bash
mvn dependency:tree
```

---

## 100. Effective POM

```bash
mvn help:effective-pom
```

---

## 101. Reactor Debugging

Use Maven normal logs and, when necessary:

```bash
mvn -X verify
```

Identify:

```text
module
phase
plugin
goal
```

where failure occurs.

---

# PART XLVIII — SELECTIVE TROUBLESHOOTING

## 102. Build One Module

```bash
mvn -pl payment-core verify
```

---

## 103. Module + Dependencies

```bash
mvn -pl payment-app -am verify
```

This is useful for focused development and CI validation.

---

# PART XLIX — PRODUCTION CI DESIGN

## 104. Monolithic Reactor Pipeline

```text
Git
 |
v
CI
 |
v
Maven Reactor
 |
+--> API
+--> Core
+--> Service
+--> App
 |
v
Security
 |
v
Artifactory
```

---

## 105. Selective Pipeline

```text
Git Diff
 |
v
Impact Analysis
 |
v
Selected Reactor Projects
 |
v
Tests
 |
v
Package
 |
v
Publish
```

Use only when dependency impact analysis is reliable.

---

# PART L — LARGE REPOSITORY DESIGN

## 106. Hundreds of Modules

Consider:

```text
domain boundaries
ownership
build graph
parallelism
CI partitioning
artifact lifecycle
```

---

## 107. Avoid Giant Modules

One huge module can create:

```text
slow builds
poor ownership
high coupling
```

---

## 108. Avoid Excessive Modules

Too many tiny modules create:

```text
POM complexity
dependency management overhead
build graph complexity
```

Use module boundaries for meaningful architecture.

---

# PART LI — MONOREPO VS MULTI-REPO

## 109. Multi-Module + Monorepo

```text
One Git repository
 |
v
Many Maven modules
```

Advantages:

```text
atomic changes
central build
easy cross-module refactoring
```

---

## 110. Multi-Repo

```text
repo-a
repo-b
repo-c
```

Can provide:

```text
independent ownership
independent release cycles
```

But cross-repository dependency updates require stronger release
coordination.

---

# PART LII — MULTI-MODULE ARTIFACTORY STRATEGY

## 111. Internal Libraries

```text
api.jar
core.jar
integration.jar
```

can be published for reuse.

---

## 112. Application Artifact

The application module can produce:

```text
app.jar
```

and then feed:

```text
container build
```

---

# PART LIII — CONTAINERIZATION

## 113. Build Flow

```text
Maven Reactor
 |
v
app.jar
 |
v
Docker/OCI build
 |
v
image
 |
v
registry
```

---

## 114. Build Once

Do not compile modules again inside the container if the architecture
already produces a validated application artifact unless there is a
specific reproducibility strategy for container-native builds.

---

# PART LIV — KUBERNETES

## 115. Deployment

```text
Maven
 |
v
JAR
 |
v
Container
 |
v
Registry
 |
v
GitOps
 |
v
Kubernetes
```

Maven owns application build; Kubernetes owns runtime orchestration.

---

# PART LV — PERFORMANCE ENGINEERING

## 116. Measure

Track:

```text
total build time
module build time
test time
dependency resolution
package time
publish time
```

---

## 117. Parallelism

Use:

```bash
mvn -T ...
```

after benchmarking.

---

## 118. Module Graph Optimization

Reduce unnecessary dependencies.

A module that depends on everything creates poor parallelism and high
change impact.

---

## 119. Test Optimization

Separate:

```text
fast unit tests
slower integration tests
expensive end-to-end tests
```

---

# PART LVI — PRODUCTION RELEASE CHECKLIST

## 120. Structure

```text
[ ] root POM
[ ] modules
[ ] clear boundaries
[ ] no cycles
```

## 121. Dependency

```text
[ ] module dependencies explicit
[ ] dependency versions controlled
[ ] BOM/management standardized
[ ] transitive graph reviewed
```

## 122. Plugins

```text
[ ] plugin versions controlled
[ ] plugin management
[ ] module-specific plugins justified
```

## 123. CI

```text
[ ] Maven Wrapper
[ ] clean build
[ ] tests
[ ] security
[ ] cache
[ ] controlled parallelism
```

## 124. Artifacts

```text
[ ] correct module artifacts
[ ] immutable release
[ ] provenance
[ ] repository permissions
```

---

# PART LVII — INTERVIEW PREPARATION

## 125. What Is a Multi-Module Maven Project?

Answer:

```text
It is a Maven build containing multiple related Maven projects that
can be coordinated through an aggregator POM and commonly share
configuration through a parent POM.
```

## 126. Parent vs Aggregator?

Answer:

```text
A parent provides inheritance. An aggregator lists modules and lets
Maven coordinate their reactor build. A root POM can be both.
```

## 127. What Is the Reactor?

Answer:

```text
The reactor is Maven's mechanism for building a set of Maven projects
together while respecting their relationships and build order.
```

## 128. How Does Maven Determine Build Order?

Answer:

```text
It uses reactor project relationships, including inter-module
dependencies, to determine an appropriate build order rather than
simply relying on directory order.
```

## 129. What Does -pl Do?

Answer:

```text
-pl selects specific projects/modules for the Maven reactor build.
```

## 130. What Does -am Do?

Answer:

```text
-am also makes required upstream reactor projects for the selected
project.
```

## 131. What Does -amd Do?

Answer:

```text
-amd also makes projects that depend on the selected project. It can
be useful for assessing downstream impact.
```

## 132. Why Use Multi-Module?

Answer:

```text
It provides logical module boundaries, centralized build
configuration, dependency-aware builds and a consistent way to build
related components.
```

## 133. How Do You Avoid Circular Dependencies?

Answer:

```text
I define clear module boundaries, keep dependency direction deliberate
and extract shared contracts or common functionality into a separate
module when two modules begin depending on each other.
```

## 134. How Do You Optimize a Large Reactor Build?

Answer:

```text
I first measure module and test timings, then use dependency-aware
selective builds, Maven caching, appropriate reactor parallelism and
test optimization. I avoid aggressive parallelism until plugin and
resource compatibility are validated.
```

---

# PART LVIII — SENIOR-LEVEL SCENARIOS

## 135. 300-Module Reactor Takes One Hour

Answer:

```text
I would profile the build by module and phase, identify the critical
path, dependency-resolution overhead, slow tests and resource
contention. Then I would reduce unnecessary dependencies, introduce
safe parallelism, optimize caches and use reliable selective builds
where appropriate.
```

## 136. One Core Module Change Triggers Everything

Answer:

```text
I would inspect the dependency graph and determine whether the module
is genuinely a central dependency or whether boundaries are too broad.
If the dependency is legitimate, I would optimize downstream tests and
build selection. If coupling is accidental, I would refactor module
boundaries.
```

## 137. Reactor Build Works but Individual Module Build Fails

Answer:

```text
I would check whether the module relies on reactor-provided artifacts
or inherited configuration. A standalone module build may require
artifacts to be installed or an appropriate parent/repository setup.
I would make the module's intended standalone behavior explicit.
```

## 138. Parallel Build Is Faster but Occasionally Fails

Answer:

```text
I would look for shared files, non-thread-safe plugins, test ports,
shared databases, generated output or resource contention. I would
identify the race condition and either fix the build or constrain
parallelism rather than accepting nondeterministic CI.
```

## 139. Parent Upgrade Breaks All Modules

Answer:

```text
I would inspect the effective POM and compare inherited dependency,
plugin and lifecycle configuration. I would pin the last known-good
parent if necessary, test the upgrade against representative modules,
and roll it out incrementally.
```

## 140. Only One Module Should Be Released

Answer:

```text
I would determine whether that module is independently versioned and
whether downstream artifacts require synchronized versions. I would
use reactor selection and the repository release strategy to publish
only the intended artifacts while preserving dependency correctness.
```

---

# PART LIX — GOLDEN RULES

## 141. Rules

```text
1. Understand parent and aggregator as separate concepts.

2. A root POM can be both parent and aggregator.

3. Use packaging=pom for typical aggregator/parent POMs.

4. Keep module boundaries meaningful.

5. Declare inter-module dependencies explicitly.

6. Understand reactor build order.

7. Do not rely on directory order.

8. Avoid circular dependencies.

9. Use dependencyManagement for centralized versions.

10. Use BOMs where compatible dependency sets need centralized
    management.

11. Use pluginManagement for centralized plugin governance.

12. Control parent versions.

13. Control plugin versions.

14. Control dependency versions.

15. Do not run install separately for every module in a reactor build.

16. Use reactor builds for related modules.

17. Use -pl for targeted project selection.

18. Use -am for required upstream reactor projects.

19. Use -amd for downstream impact scenarios.

20. Validate selective builds before relying on them for release gates.

21. Do not sacrifice coverage for CI speed.

22. Use parallel builds only after measuring compatibility.

23. Monitor CPU, memory, disk and network during parallel builds.

24. Cache Maven dependencies carefully.

25. Include dependency/POM inputs in cache strategy.

26. Keep release artifacts immutable.

27. Do not publish every CI build unnecessarily.

28. Build once and promote the same artifacts.

29. Scan dependencies across modules.

30. Scan the final application artifact/container.

31. Track licenses across the dependency graph.

32. Generate SBOMs where required.

33. Keep build configuration reproducible.

34. Use Maven Wrapper.

35. Control the JDK.

36. Separate fast unit tests from expensive integration tests.

37. Treat plugin execution as part of the build supply chain.

38. Investigate the first failed reactor module before downstream
    failures.

39. Use dependency:tree for dependency issues.

40. Use effective-pom for inherited configuration issues.

41. Use clean builds when stale module output is suspected.

42. Avoid giant modules.

43. Avoid meaningless micro-modules.

44. Define module ownership.

45. Define module release ownership.

46. Assess change impact through the dependency graph.

47. Keep API/contract modules separate when architecture benefits from
    explicit contracts.

48. Do not allow infrastructure dependencies to leak into every module.

49. Use Artifactory or another approved repository for controlled
    artifact distribution.

50. Validate exact Maven, plugin, JDK and repository behavior for the
    versions actually used in production.
```

---