# 18-Build-and-Package-Management
# 05-Maven-Lifecycle

## 1. Purpose

Maven's lifecycle is the execution model that turns project
configuration into a repeatable build process.

A DevOps engineer should understand not only the command:

```bash
mvn clean package
```

but also:

```text
which lifecycle is being invoked
which phase is being requested
which earlier phases execute
which plugins are bound
which goals execute
which artifact is produced
what happens in CI
```

This file covers Maven lifecycles, phases, goals, plugin bindings,
execution order, clean/default/site lifecycles, packaging behavior,
profiles, CI/CD, production practices, troubleshooting and interview
scenarios.

---

# PART I — LIFECYCLE FUNDAMENTALS

## 2. What Is a Maven Lifecycle?

A Maven lifecycle is a defined sequence of build phases.

Conceptually:

```text
Lifecycle
 |
+--> Phase 1
+--> Phase 2
+--> Phase 3
+--> ...
```

Maven provides standard lifecycle models so projects can use consistent
build commands.

---

## 3. Maven Standard Lifecycles

Maven has three built-in lifecycles:

```text
clean
default
site
```

They serve different purposes.

---

## 4. clean Lifecycle

The clean lifecycle is concerned with removing generated build output.

Common command:

```bash
mvn clean
```

Typical result:

```text
target/
```

is removed.

---

## 5. default Lifecycle

The default lifecycle is the primary lifecycle for:

```text
validation
compilation
testing
packaging
verification
installation
deployment
```

Common commands:

```bash
mvn test
mvn package
mvn verify
mvn install
mvn deploy
```

---

## 6. site Lifecycle

The site lifecycle is used for project documentation and reporting.

Common command:

```bash
mvn site
```

It is separate from the normal application packaging lifecycle.

---

# PART II — PHASES

## 7. What Is a Phase?

A phase is a lifecycle step.

Examples:

```text
validate
compile
test
package
verify
install
deploy
```

When you request a later phase, Maven normally executes the required
earlier phases in that lifecycle.

---

## 8. Phase Execution

Example:

```bash
mvn package
```

Conceptually:

```text
validate
   |
compile
   |
test
   |
package
```

The exact work at each phase depends on the project packaging and
plugin bindings.

---

## 9. Later Phase Means Earlier Phases

If you execute:

```bash
mvn verify
```

Maven executes the earlier required default lifecycle phases before
`verify`.

This is why a single command can represent a complete build path.

---

# PART III — PHASE VS GOAL

## 10. What Is a Goal?

A goal is a specific task provided by a Maven plugin.

Example:

```text
maven-compiler-plugin:compile
```

contains:

```text
plugin = maven-compiler-plugin
goal   = compile
```

---

## 11. Phase vs Goal

Important distinction:

```text
Phase
 |
lifecycle step

Goal
 |
plugin task
```

A lifecycle phase can have one or more plugin goals bound to it.

---

## 12. Example

Conceptually:

```text
compile phase
      |
      v
maven-compiler-plugin:compile
```

Testing:

```text
test phase
      |
      v
maven-surefire-plugin:test
```

The exact plugin and execution can depend on packaging and project
configuration.

---

# PART IV — PLUGIN BINDINGS

## 13. What Is a Plugin Binding?

Maven can bind plugin goals to lifecycle phases.

Example concept:

```text
compile phase
 |
v
compiler:compile
```

---

## 14. Why Bindings Matter

A command such as:

```bash
mvn compile
```

does not mean Maven has a hard-coded universal compiler command.

The lifecycle maps phases to appropriate plugin goals.

---

## 15. Packaging Controls Defaults

Packaging affects which standard plugin goals are bound.

For example:

```text
jar
war
pom
```

can result in different lifecycle behavior.

---

# PART V — DEFAULT LIFECYCLE PHASES

## 16. Important Phases

A practical list includes:

```text
validate
initialize
generate-sources
process-sources
generate-resources
process-resources
compile
process-classes
generate-test-sources
process-test-sources
generate-test-resources
process-test-resources
test-compile
process-test-classes
test
prepare-package
package
pre-integration-test
integration-test
post-integration-test
verify
install
deploy
```

Not every project performs meaningful work in every phase.

---

## 17. validate

Checks whether the project is correct and required information is
available.

---

## 18. initialize

Used for early build initialization.

---

## 19. generate-sources

Can generate source code before compilation.

---

## 20. process-sources

Processes project source files.

---

## 21. generate-resources

Can generate resources required later in the build.

---

## 22. process-resources

Processes and copies resources into the build output.

---

## 23. compile

Compiles main source code.

Typical output:

```text
target/classes/
```

---

## 24. process-classes

Processes compiled classes when configured.

---

## 25. generate-test-sources

Can generate test source code.

---

## 26. process-test-sources

Processes test source files.

---

## 27. generate-test-resources

Can generate test resources.

---

## 28. process-test-resources

Processes test resources.

---

## 29. test-compile

Compiles test source.

Typical output:

```text
target/test-classes/
```

---

## 30. process-test-classes

Processes compiled test classes when configured.

---

## 31. test

Runs unit tests according to configured test plugins.

Typical reporting directory:

```text
target/surefire-reports/
```

---

## 32. prepare-package

Runs actions needed before packaging.

---

## 33. package

Creates the distributable artifact.

Examples:

```text
JAR
WAR
```

---

## 34. pre-integration-test

Prepares the environment for integration testing.

---

## 35. integration-test

Runs integration-test activity.

This phase is commonly associated with the Maven Failsafe Plugin in
projects configured for integration testing.

---

## 36. post-integration-test

Cleans up or shuts down resources used by integration tests.

---

## 37. verify

Runs checks that validate the package and project before installation
or deployment.

---

## 38. install

Installs the artifact into the local Maven repository.

```text
target/artifact.jar
        |
        v
~/.m2/repository
```

---

## 39. deploy

Publishes the artifact to the configured remote repository.

```text
artifact
   |
   v
remote repository
```

---

# PART VI — CLEAN LIFECYCLE

## 40. Clean Phases

The clean lifecycle has its own phases:

```text
pre-clean
clean
post-clean
```

---

## 41. clean

Common command:

```bash
mvn clean
```

The normal purpose is to remove generated build output.

---

## 42. clean package

A common CI command:

```bash
mvn clean package
```

This invokes the clean lifecycle and then the default lifecycle up to
`package`.

---

## 43. Why Clean Builds?

A clean build helps avoid stale generated files.

Useful when:

```text
source changed
dependencies changed
plugins changed
generated sources changed
previous build state is suspicious
```

---

# PART VII — COMMANDS

## 44. validate

```bash
mvn validate
```

---

## 45. compile

```bash
mvn compile
```

---

## 46. test

```bash
mvn test
```

---

## 47. package

```bash
mvn package
```

---

## 48. verify

```bash
mvn verify
```

---

## 49. install

```bash
mvn install
```

---

## 50. deploy

```bash
mvn deploy
```

---

## 51. Clean

```bash
mvn clean
```

---

## 52. Common CI Command

```bash
./mvnw -B clean verify
```

`-B` requests batch mode, which is useful in non-interactive CI.

---

# PART VIII — PACKAGE-SPECIFIC LIFECYCLE

## 53. JAR Packaging

For:

```xml
<packaging>jar</packaging>
```

the default lifecycle includes standard Java compilation, testing and
JAR packaging behavior through Maven's standard plugin bindings.

---

## 54. WAR Packaging

For:

```xml
<packaging>war</packaging>
```

the lifecycle uses WAR-oriented packaging behavior.

---

## 55. POM Packaging

For:

```xml
<packaging>pom</packaging>
```

the project commonly serves as:

```text
parent
aggregator
BOM
```

depending on configuration.

---

# PART IX — LIFECYCLE EXECUTION MODEL

## 56. Example

Command:

```bash
mvn package
```

Execution concept:

```text
validate
   |
initialize
   |
generate sources/resources
   |
process sources/resources
   |
compile
   |
test-compile
   |
test
   |
prepare-package
   |
package
```

Not every project configures work in every phase.

---

## 57. verify

```bash
mvn verify
```

extends the execution:

```text
...
package
 |
pre-integration-test
 |
integration-test
 |
post-integration-test
 |
verify
```

when those phases have relevant configured goals.

---

# PART X — INTEGRATION TESTING

## 58. Why Separate Integration Tests?

Integration tests can require:

```text
database
message broker
external service
container
network
```

They are often more expensive than unit tests.

---

## 59. Typical Model

```text
Unit Tests
 |
v
Package
 |
v
Start Test Environment
 |
v
Integration Tests
 |
v
Cleanup
 |
v
Verify
```

---

## 60. Failsafe

Projects commonly configure:

```text
maven-failsafe-plugin
```

for integration testing.

A common convention is:

```text
pre-integration-test
integration-test
post-integration-test
verify
```

The exact configuration and test naming must be checked in the POM.

---

# PART XI — SUREFIRE VS FAILSAFE

## 61. Surefire

Commonly used for unit tests during:

```text
test
```

---

## 62. Failsafe

Commonly used for integration tests around:

```text
integration-test
verify
```

---

## 63. Why This Separation?

If integration infrastructure fails, lifecycle cleanup can still be
configured around the integration-test sequence.

---

# PART XII — SKIPPING TESTS

## 64. -DskipTests

Common command:

```bash
mvn package -DskipTests
```

This generally skips running tests while still allowing test
compilation.

---

## 65. maven.test.skip

Example:

```bash
mvn package -Dmaven.test.skip=true
```

This can skip test compilation and execution.

These options are not equivalent.

---

## 66. Production Rule

Do not skip tests simply to make a failing pipeline green.

Use documented exceptions only.

---

# PART XIII — LIFECYCLE AND PROFILES

## 67. Profile

A profile can alter configuration used during lifecycle execution.

Example:

```bash
mvn -Pci verify
```

---

## 68. Profile Example

```xml
<profile>
    <id>ci</id>
    <properties>
        <some.setting>true</some.setting>
    </properties>
</profile>
```

---

## 69. Profile Risk

Too many profiles can cause:

```text
unexpected behavior
environment drift
hard-to-reproduce builds
```

---

# PART XIV — LIFECYCLE AND PLUGINS

## 70. Plugin Execution

Example concept:

```text
compile
 |
v
compiler:compile
```

---

## 71. Explicit Plugin Execution

A plugin can be configured with:

```xml
<executions>
    <execution>
        <id>generate-code</id>
        <phase>generate-sources</phase>
        <goals>
            <goal>generate</goal>
        </goals>
    </execution>
</executions>
```

This means the goal is associated with a lifecycle phase.

---

## 72. Execution ID

The execution ID helps distinguish multiple configured executions
of a plugin.

Use meaningful IDs:

```text
generate-openapi
integration-tests
frontend-build
```

---

# PART XV — MULTIPLE GOALS

## 73. Multiple Goals

A plugin can have multiple executions.

Concept:

```text
Plugin
 |
+--> execution A
|      |
|      +--> phase
|      +--> goal
|
+--> execution B
       |
       +--> phase
       +--> goal
```

---

# PART XVI — DIRECT GOAL INVOCATION

## 74. Direct Plugin Goal

You can invoke a goal directly:

```bash
mvn dependency:tree
```

This is different from:

```bash
mvn test
```

because `dependency:tree` directly invokes a plugin goal rather than
requesting a standard lifecycle phase.

---

## 75. Why Direct Goals Matter?

Useful for:

```text
troubleshooting
report generation
dependency inspection
specialized build tasks
```

---

# PART XVII — PHASE + GOAL

## 76. Mental Model

Remember:

```text
mvn package
```

means:

```text
request lifecycle phase
```

while:

```text
mvn dependency:tree
```

means:

```text
invoke plugin goal
```

---

# PART XVIII — LIFECYCLE AND DEPENDENCIES

## 77. Dependency Resolution

Before compilation Maven must obtain required dependencies.

Concept:

```text
POM
 |
v
Resolve dependencies
 |
v
compile
```

---

## 78. Dependency Failure

If a dependency cannot be resolved:

```text
compile
   X
```

and later phases normally do not complete.

---

# PART XIX — LIFECYCLE AND ARTIFACTS

## 79. package

For a JAR:

```text
package
 |
v
target/application.jar
```

---

## 80. install

```text
target/application.jar
 |
v
~/.m2/repository
```

---

## 81. deploy

```text
target/application.jar
 |
v
Artifactory / Nexus / approved repository
```

---

# PART XX — LIFECYCLE AND CI/CD

## 82. Pull Request Pipeline

A typical PR pipeline:

```text
Checkout
 |
v
./mvnw -B clean verify
 |
+--> compile
+--> unit tests
+--> integration tests where configured
+--> quality
+--> security
 |
v
Result
```

---

## 83. Release Pipeline

```text
Git Tag
 |
v
Clean Build
 |
v
Verify
 |
v
Security
 |
v
Package
 |
v
Publish
 |
v
Promotion
```

---

## 84. Build Once

Do not do:

```text
DEV build
STAGE rebuild
PROD rebuild
```

Prefer:

```text
one artifact
 |
+--> DEV
+--> STAGE
+--> PROD
```

---

# PART XXI — LIFECYCLE + ARTIFACTORY

## 85. Enterprise Flow

```text
Maven
 |
v
Lifecycle
 |
v
Package
 |
v
Artifact
 |
v
Artifactory
```

---

## 86. Dependency Flow

```text
Lifecycle
 |
v
Dependency Resolution
 |
v
Artifactory Virtual
 |
+--> Local
+--> Remote
```

---

## 87. Publishing

A controlled release can use:

```bash
./mvnw -B clean deploy
```

provided the POM, settings and repository credentials are configured
correctly.

---

# PART XXII — LIFECYCLE + JENKINS

## 88. Jenkins Pipeline

Example:

```groovy
stage('Build') {
    steps {
        sh './mvnw -B clean verify'
    }
}
```

---

## 89. Publishing Stage

Concept:

```groovy
stage('Publish') {
    steps {
        sh './mvnw -B deploy'
    }
}
```

The production pipeline should separate validation from release
publishing according to organizational policy.

---

# PART XXIII — LIFECYCLE + GITHUB ACTIONS

## 90. Example Flow

```text
checkout
 |
setup JDK
 |
Maven Wrapper
 |
clean verify
 |
security
 |
publish
```

---

## 91. Cache

A CI workflow can cache Maven dependency data, but cache keys should
include relevant dependency/toolchain inputs.

---

# PART XXIV — LIFECYCLE + CONTAINERS

## 92. Build Flow

```text
Maven
 |
v
package
 |
v
JAR
 |
v
Docker build
 |
v
OCI image
```

---

## 93. Multi-Stage Build

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
+--> runtime
 |
v
Image
```

This can keep build tools out of the final runtime image.

---

# PART XXV — LIFECYCLE + KUBERNETES

## 94. Separation

```text
Maven:
build JAR

Docker:
build image

Registry:
store image

GitOps:
declare image

Kubernetes:
run image
```

---

# PART XXVI — LIFECYCLE PERFORMANCE

## 95. Slow Dependency Resolution

Investigate:

```text
repository latency
DNS
network
Maven mirror
local cache
artifact size
```

---

## 96. Slow Tests

Investigate:

```text
test count
integration dependencies
database setup
container startup
external calls
parallelization
```

---

## 97. Slow Compilation

Investigate:

```text
CPU
memory
annotation processors
generated source
source size
JDK
```

---

## 98. Pipeline Optimization

Possible techniques:

```text
dependency caching
parallel test execution
ephemeral runners
prebuilt CI images
incremental work
```

Measure before changing architecture.

---

# PART XXVII — REPRODUCIBILITY

## 99. Lifecycle Reproducibility

Control:

```text
Maven version
JDK
POM
dependencies
plugins
repositories
profiles
environment
```

---

## 100. Maven Wrapper

Prefer:

```bash
./mvnw
```

when the project maintains a wrapper.

---

## 101. Clean Builds

For release builds:

```bash
./mvnw -B clean verify
```

helps remove stale generated output.

---

# PART XXVIII — LIFECYCLE TROUBLESHOOTING

## 102. Build Stops at compile

Check:

```text
source code
JDK
compiler release
dependency versions
annotation processors
```

---

## 103. Build Stops at test

Check:

```text
unit test failure
environment
test data
dependency
flaky tests
```

---

## 104. Build Stops at integration-test

Check:

```text
database
container
network
service dependency
test environment
```

---

## 105. Build Stops at verify

Check:

```text
integration-test result
quality gates
verification plugin
test reports
```

---

## 106. Deploy Fails

Check:

```text
distributionManagement
repository URL
server ID
credentials
token scope
permissions
version policy
```

---

# PART XXIX — DEBUGGING LIFECYCLE EXECUTION

## 107. Debug Logs

Use:

```bash
mvn -X package
```

when normal output is insufficient.

---

## 108. Effective POM

Use:

```bash
mvn help:effective-pom
```

to determine inherited plugin configuration and lifecycle-related
configuration.

---

## 109. Effective Settings

Use:

```bash
mvn help:effective-settings
```

to investigate:

```text
mirrors
servers
profiles
repository configuration
```

---

# PART XXX — LIFECYCLE AND MULTI-MODULE

## 110. Reactor

In a multi-module Maven build:

```text
root
 |
+--> module-a
+--> module-b
+--> module-c
```

Maven builds modules according to their relationships and reactor
ordering.

---

## 111. Module Dependency

If:

```text
module-b -> module-a
```

Maven needs module A available before the relevant work for module B.

---

## 112. Reactor Failure

A failure in an upstream module can prevent dependent modules from
building successfully.

---

# PART XXXI — SELECTIVE MODULE BUILDS

## 113. Large Repository

For large multi-module projects, Maven supports reactor options such
as:

```bash
-pl module-a
-am
```

For example:

```bash
mvn -pl module-b -am test
```

can build the selected module and required reactor dependencies.

Exact project structure should be understood before using selective
builds in CI.

---

# PART XXXII — FAIL-FAST VS RECOVERY

## 114. Failure Strategy

A pipeline should stop when a required quality gate fails.

Example:

```text
compile FAIL
   |
   X
do not publish
```

---

## 115. Never Publish Failed Builds

Production repository:

```text
only validated artifacts
```

---

# PART XXXIII — TEST SKIPPING ANTI-PATTERNS

## 116. Bad

```bash
mvn package -DskipTests
```

only because tests fail.

---

## 117. Better

```text
Investigate failure
 |
v
Fix root cause
 |
v
Run tests
 |
v
Publish
```

---

# PART XXXIV — PRODUCTION LIFECYCLE DESIGN

## 118. PR

```text
clean
 |
verify
```

---

## 119. Release

```text
clean
 |
verify
 |
security
 |
deploy
```

---

## 120. Promotion

```text
Artifact Repository
 |
v
Promote same artifact
```

Do not rebuild solely for environment promotion.

---

# PART XXXV — COMMON MISTAKES

## 121. Mistake: Confusing Phase and Goal

Incorrect mental model:

```text
package = plugin
```

Better:

```text
package = lifecycle phase
```

---

## 122. Mistake: Assuming Every Phase Has Work

A phase can exist even when no relevant plugin goal is bound for a
specific project.

---

## 123. Mistake: Assuming package Means deploy

```bash
mvn package
```

creates the artifact.

It does not automatically publish it to a remote artifact repository.

---

## 124. Mistake: Assuming install Means Remote Publish

```bash
mvn install
```

installs into the local Maven repository.

---

## 125. Mistake: Using deploy for Every CI Build

Publishing every pull-request build to a release repository can create
repository pollution.

Use repository strategy appropriate to:

```text
snapshot
candidate
release
```

artifacts.

---

# PART XXXVI — PRODUCTION ARCHITECTURE

## 126. Reference

```text
                       Git
                        |
                        v
                    CI Trigger
                        |
                        v
                 Ephemeral Runner
                        |
                        v
                  Maven Wrapper
                        |
                        v
               Dependency Resolution
                        |
                        v
                 Artifactory Virtual
                        |
                        v
              Compile -> Test -> Verify
                        |
                        v
                     Package
                        |
                        v
                Security / Quality
                        |
                        v
              Artifactory Release Repo
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

# PART XXXVII — PRODUCTION CHECKLIST

## 127. Lifecycle

```text
[ ] correct lifecycle understood
[ ] correct phase selected
[ ] plugin bindings reviewed
[ ] packaging understood
```

## 128. Build

```text
[ ] Maven Wrapper
[ ] JDK controlled
[ ] dependencies controlled
[ ] tests enabled
[ ] quality gates
```

## 129. CI

```text
[ ] clean workspace
[ ] batch mode
[ ] dependency cache
[ ] protected credentials
[ ] build logs
```

## 130. Artifact

```text
[ ] package generated
[ ] versioned
[ ] scanned
[ ] immutable
[ ] published only after validation
```

## 131. Release

```text
[ ] build once
[ ] promote same artifact
[ ] rollback artifact available
[ ] deployment traceability
```

---

# PART XXXVIII — INTERVIEW PREPARATION

## 132. What Is a Maven Lifecycle?

Answer:

```text
A Maven lifecycle is a predefined sequence of build phases. Maven
provides clean, default and site lifecycles. The default lifecycle is
used for common application build activities such as compilation,
testing, packaging, verification, installation and deployment.
```

## 133. Phase vs Goal?

Answer:

```text
A phase is a step in a Maven lifecycle. A goal is a specific task
provided by a Maven plugin. Plugin goals can be bound to lifecycle
phases.
```

## 134. What Happens During mvn package?

Answer:

```text
Maven executes the required earlier phases of the default lifecycle
and then reaches package, where the configured packaging plugin creates
the project artifact.
```

## 135. install vs deploy?

Answer:

```text
install places the artifact in the local Maven repository. deploy
publishes the artifact to the configured remote repository.
```

## 136. clean package?

Answer:

```text
It runs the clean lifecycle and then the default lifecycle through
package. This removes generated output before creating a fresh
artifact.
```

## 137. Why verify Instead of package?

Answer:

```text
verify is useful when the project has checks configured after package,
including integration-test and verification activities. I use the
phase that represents the project's required quality gates.
```

## 138. Surefire vs Failsafe?

Answer:

```text
Surefire is commonly used for unit tests during the test phase.
Failsafe is commonly used for integration testing around the
integration-test and verify phases.
```

## 139. How Do You Debug Lifecycle Behavior?

Answer:

```text
I inspect the POM and effective POM, identify plugin bindings and
executions, run Maven with debug logging when necessary, and inspect
the exact phase where execution stops.
```

---

# PART XXXIX — SENIOR-LEVEL SCENARIOS

## 140. All Maven Builds Fail During Dependency Resolution

Answer:

```text
I would first determine whether the problem is project-specific or
shared. If many projects fail at the same phase, I would investigate
the internal repository, mirror, DNS, TLS, network, authentication and
upstream availability before modifying individual POMs.
```

## 141. Build Takes 30 Minutes Instead of 8

Answer:

```text
I would compare stage timings with the baseline. I would determine
whether dependency resolution, compilation, unit tests, integration
tests, scanning or publishing changed. Then I would optimize the
measured bottleneck using caching, parallelization or infrastructure
changes as appropriate.
```

## 142. Developer Uses package but CI Uses deploy

Answer:

```text
That is not necessarily a problem because the commands have different
purposes. package creates the artifact, while deploy publishes it to a
remote repository. I would ensure the release pipeline publishes only
validated artifacts and uses the correct repository policy.
```

## 143. Integration Tests Leave Containers Running

Answer:

```text
I would inspect the Failsafe and lifecycle configuration around
pre-integration-test, integration-test and post-integration-test.
Cleanup must be reliable even when tests fail so CI runners do not
leak containers, ports or resources.
```

## 144. A Plugin Executes Unexpectedly

Answer:

```text
I would inspect the effective POM, parent POM, pluginManagement,
plugins and executions, then identify which lifecycle phase the goal
is bound to. I would avoid deleting configuration until I understand
where it was inherited from.
```

---

# PART XL — GOLDEN RULES

## 145. Rules

```text
1. Understand lifecycle before memorizing commands.

2. Maven has clean, default and site lifecycles.

3. Distinguish lifecycle phase from plugin goal.

4. A phase may have plugin goals bound to it.

5. Packaging influences default plugin bindings.

6. Later phases execute the required earlier phases.

7. package creates a distributable artifact.

8. install uses the local Maven repository.

9. deploy publishes to a configured remote repository.

10. clean removes generated build output.

11. Use clean builds for release confidence where appropriate.

12. Understand validate, compile, test, package, verify, install and
    deploy.

13. Understand integration-test and verify.

14. Use Surefire commonly for unit tests.

15. Use Failsafe commonly for integration tests.

16. Do not skip tests just to make a pipeline green.

17. Understand the difference between -DskipTests and
    maven.test.skip.

18. Treat plugin versions as controlled build dependencies.

19. Understand plugin executions.

20. Use meaningful execution IDs.

21. Use effective-pom when inherited configuration is unclear.

22. Use dependency:tree when dependency resolution is unclear.

23. Use effective-settings when repository configuration is unclear.

24. Use debug logging carefully.

25. Keep CI builds non-interactive.

26. Prefer Maven Wrapper.

27. Control JDK and Maven versions.

28. Cache dependencies carefully.

29. Do not publish failed artifacts.

30. Separate validation from release publishing.

31. Build once and promote the same artifact.

32. Do not confuse package with deploy.

33. Do not confuse install with remote publishing.

34. Avoid repository pollution from unnecessary CI deployments.

35. Protect repository credentials.

36. Treat plugin execution as part of the software supply chain.

37. Keep integration-test cleanup reliable.

38. Monitor build duration by lifecycle stage.

39. Investigate shared infrastructure when many builds fail at the same
    phase.

40. Validate exact Maven, JDK and plugin behavior for the versions used
    in production.
```

---

# END OF 05-Maven-Lifecycle.md
