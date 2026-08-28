# 18-Build-and-Package-Management
# 09-Maven-Jenkins-Integration

## 1. Purpose

Maven and Jenkins are commonly combined to implement enterprise Java
CI/CD pipelines.

The important production model is not simply:

```text
Jenkins -> mvn package
```

A mature integration looks more like:

```text
Git
 |
v
Jenkins
 |
+--> checkout
+--> toolchain
+--> credentials
+--> Maven Wrapper
+--> dependency resolution
+--> compile
+--> unit tests
+--> integration tests
+--> quality
+--> security
+--> package
+--> publish
+--> deploy/promote
 |
v
Artifact Repository
```

This file covers Jenkins fundamentals, Maven integration, Pipeline
design, Declarative and Scripted Pipeline concepts, Maven Wrapper,
JDK/tool management, settings.xml, Artifactory, credentials, caching,
parallelism, agents, Docker, Kubernetes, testing, quality gates,
security, artifact promotion, releases, rollback, troubleshooting,
production architecture and interview preparation.

---

# PART I — FUNDAMENTALS

## 2. What Is Jenkins?

Jenkins is an automation server commonly used to implement CI/CD
pipelines.

It can:

```text
checkout source
run builds
run tests
execute security scans
publish artifacts
build images
deploy applications
```

---

## 3. What Is Maven's Role?

Maven manages the Java build.

Jenkins manages pipeline orchestration.

```text
Jenkins
   |
   +--> invokes Maven
             |
             +--> dependency resolution
             +--> compile
             +--> test
             +--> package
             +--> deploy
```

---

## 4. Separation of Responsibility

A useful model:

```text
Jenkins
 |
pipeline orchestration

Maven
 |
project build lifecycle

Artifactory
 |
artifact/dependency repository

Kubernetes
 |
runtime orchestration
```

---

# PART II — BASIC JENKINS MAVEN PIPELINE

## 5. Minimal Declarative Pipeline

Example:

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh './mvnw -B clean verify'
            }
        }
    }
}
```

This is intentionally simple.

Production pipelines usually add:

```text
credentials
caching
reports
quality
security
artifact publishing
notifications
```

---

# PART III — MAVEN WRAPPER

## 6. Why Maven Wrapper?

Use:

```bash
./mvnw
```

instead of assuming the Jenkins agent has the correct Maven version.

The wrapper improves:

```text
version consistency
developer/CI parity
reproducibility
```

---

## 7. Production Rule

Prefer:

```groovy
sh './mvnw -B clean verify'
```

over:

```groovy
sh 'mvn clean verify'
```

when the project maintains a Maven Wrapper.

---

# PART IV — JDK MANAGEMENT

## 8. Maven Requires a JDK

The Jenkins agent needs a compatible Java runtime/toolchain.

Example:

```text
JDK 21
 |
v
Maven
 |
v
Build
```

---

## 9. Version Alignment

Control:

```text
JDK
Maven
compiler plugin
project release
```

Example:

```xml
<release>21</release>
```

The Jenkins environment must support it.

---

# PART V — JENKINS AGENTS

## 10. Agent

A Jenkins agent executes pipeline steps.

Example:

```text
Jenkins Controller
       |
       +--> Agent A
       |
       +--> Agent B
       |
       +--> Agent C
```

---

## 11. Agent Requirements

A Maven agent may need:

```text
JDK
Git
network access
Maven Wrapper support
Docker if required
scanner tools if required
```

---

## 12. Ephemeral Agents

Modern pipelines often use ephemeral agents.

```text
Job
 |
v
New Agent
 |
v
Build
 |
v
Destroy Agent
```

Benefits:

```text
clean environment
less state leakage
better isolation
```

---

# PART VI — JENKINS CONTROLLER VS AGENT

## 13. Controller

The Jenkins controller orchestrates jobs and pipeline scheduling.

Avoid using it as a general-purpose build server in production unless
the architecture explicitly requires it.

---

## 14. Agent

The agent executes build commands.

```text
Controller
    |
    v
Agent
    |
    v
./mvnw
```

---

# PART VII — WORKSPACE

## 15. Jenkins Workspace

A job gets a workspace:

```text
$WORKSPACE
```

Source code is checked out there.

Typical flow:

```text
workspace
 |
+--> pom.xml
+--> src/
+--> .mvn/
+--> mvnw
```

---

## 16. Workspace Cleanup

Stale workspace data can cause confusing builds.

For important jobs:

```text
clean workspace
 |
v
checkout
 |
v
build
```

Ephemeral agents naturally reduce this risk.

---

# PART VIII — CHECKOUT

## 17. Source Checkout

Typical:

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

---

## 18. Checkout Traceability

Record:

```text
commit SHA
branch/tag
build number
repository
```

This is important for release traceability.

---

# PART IX — MAVEN SETTINGS

## 19. settings.xml

Enterprise builds often require Maven settings for:

```text
mirror
repository
plugin repository
authentication
profiles
```

---

## 20. Do Not Commit Secrets

Never place production credentials directly in:

```text
pom.xml
settings.xml in Git
Jenkinsfile
```

Use Jenkins credentials or an equivalent secret-management system.

---

# PART X — JENKINS CREDENTIALS

## 21. Credential Types

Depending on environment, Jenkins may manage:

```text
username/password
secret text
SSH credentials
certificates
cloud credentials
tokens
```

---

## 22. Maven Repository Token

Concept:

```text
Jenkins Credential
        |
        v
Maven settings
        |
        v
Artifactory
```

---

## 23. Least Privilege

PR builds generally need:

```text
read
```

Release publishing needs:

```text
write/deploy
```

Do not give every pipeline administrative repository permissions.

---

# PART XI — MAVEN + ARTIFACTORY

## 24. Dependency Flow

```text
Jenkins
 |
v
Maven
 |
v
Artifactory Virtual
 |
+--> internal
+--> approved remote
```

---

## 25. Publish Flow

```text
Jenkins
 |
v
Maven deploy
 |
v
Artifactory Release Repository
```

---

# PART XII — DISTRIBUTION MANAGEMENT

## 26. Example

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

The actual URL is environment-specific.

---

## 27. Server ID Matching

The repository ID must align with the credentials configuration.

Concept:

```text
POM
 |
+--> company-releases
          |
          v
settings.xml
 |
+--> server id = company-releases
```

---

# PART XIII — BUILD PIPELINE

## 28. Standard Production Flow

```text
Checkout
 |
v
Environment Validation
 |
v
Dependency Resolution
 |
v
Compile
 |
v
Unit Test
 |
v
Integration Test
 |
v
Quality
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

# PART XIV — CLEAN BUILD

## 29. clean verify

Typical:

```bash
./mvnw -B clean verify
```

This is a strong general CI validation command.

---

## 30. Why verify?

`verify` allows configured verification and integration-test activities
to run after packaging.

The exact behavior depends on project configuration.

---

# PART XV — TESTING

## 31. Unit Tests

Typical Maven phase:

```text
test
```

Jenkins should publish test results.

---

## 32. Integration Tests

Common lifecycle:

```text
pre-integration-test
integration-test
post-integration-test
verify
```

Projects commonly use Failsafe for this workflow.

---

## 33. Test Reports

Typical:

```text
target/surefire-reports/
target/failsafe-reports/
```

Configure Jenkins to collect the appropriate JUnit-compatible reports.

---

# PART XVI — QUALITY GATES

## 34. Quality

A pipeline may include:

```text
static analysis
code coverage
quality gate
```

Example flow:

```text
Build
 |
v
Test
 |
v
Quality Gate
 |
+--> PASS -> continue
|
+--> FAIL -> stop
```

---

# PART XVII — SECURITY

## 35. Dependency Security

Scan:

```text
direct dependencies
transitive dependencies
```

---

## 36. Build Security

Also consider:

```text
Maven plugins
JDK
base build image
Jenkins plugins
agent image
```

---

## 37. Secrets

Secrets should be injected at runtime.

Do not expose them through:

```text
echo
command arguments
logs
artifacts
```

---

# PART XVIII — PIPELINE STAGES

## 38. Example Structure

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh './mvnw -B clean verify'
            }
        }

        stage('Package') {
            steps {
                sh './mvnw -B package'
            }
        }
    }
}
```

In practice, avoid unnecessarily repeating lifecycle work. A single
well-designed build can often produce the artifact that later stages
consume.

---

# PART XIX — BUILD ONCE

## 39. Anti-Pattern

```text
Build
 |
v
DEV
 |
rebuild
 |
v
STAGE
 |
rebuild
 |
v
PROD
```

---

## 40. Better

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

---

# PART XX — ARTIFACT PUBLISHING

## 41. Release

Example:

```bash
./mvnw -B deploy
```

Only run this after required quality and security gates.

---

## 42. Snapshot

Development builds may publish:

```text
1.4.0-SNAPSHOT
```

to a snapshot repository.

---

## 43. Release

Production releases should use immutable release coordinates:

```text
1.4.0
```

---

# PART XXI — CONDITIONAL PUBLISHING

## 44. Branch Strategy

For example:

```text
Pull Request
 |
v
verify
 |
X
no release publish
```

Release:

```text
release/tag
 |
v
verify
 |
v
deploy
```

The exact branching strategy depends on organizational workflow.

---

# PART XXII — JENKINS PARAMETERS

## 45. Parameters

Possible pipeline inputs:

```text
environment
version
release flag
feature flag
```

Avoid allowing arbitrary user-controlled commands.

---

# PART XXIII — ENVIRONMENT VARIABLES

## 46. Variables

Common examples:

```text
JAVA_HOME
MAVEN_OPTS
WORKSPACE
BUILD_NUMBER
BUILD_URL
```

Do not depend on undocumented agent-specific variables.

---

# PART XXIV — MAVEN OPTIONS

## 47. Batch Mode

Use:

```bash
-B
```

for CI.

---

## 48. Debug

Use only when required:

```bash
-X
```

It can produce very large logs and may expose sensitive diagnostic
information depending on configuration.

---

## 49. Quiet

```bash
-q
```

should be used carefully because excessive suppression can make
troubleshooting harder.

---

# PART XXV — MAVEN CACHE

## 50. ~/.m2/repository

Maven stores downloaded dependencies locally.

Jenkins can cache this directory.

---

## 51. Cache Benefits

```text
faster builds
less network traffic
less repository load
```

---

## 52. Cache Risks

```text
stale data
corrupted artifacts
cross-job contamination
disk growth
```

---

## 53. Ephemeral Agent Cache

A controlled persistent cache can provide speed while preserving
ephemeral build workspaces.

---

# PART XXVI — CACHE DESIGN

## 54. Cache Key

Consider:

```text
OS
JDK
Maven
POM hash
dependency configuration
```

---

## 55. Cold Cache Testing

Periodically validate builds without relying on a warm cache.

This helps reveal:

```text
hidden dependencies
unavailable artifacts
broken repository configuration
```

---

# PART XXVII — PARALLELIZATION

## 56. Maven Threads

Example:

```bash
./mvnw -T 4 clean verify
```

---

## 57. Jenkins Parallel Stages

Independent activities can sometimes run in parallel:

```text
             +--> Security
Build ------>|
             +--> Documentation
```

Only parallelize work that has no unsafe shared-state dependency.

---

# PART XXVIII — TEST PARALLELIZATION

## 58. Unit Tests

Test parallelization can reduce duration when tests are isolated.

Watch for:

```text
shared database
shared files
shared ports
global state
```

---

# PART XXIX — TIMEOUTS

## 59. Pipeline Timeout

Use timeouts to prevent stuck jobs from consuming agents indefinitely.

Concept:

```groovy
options {
    timeout(time: 30, unit: 'MINUTES')
}
```

The correct timeout should be based on observed build duration and
reasonable operational limits.

---

# PART XXX — RETRIES

## 60. Retry Carefully

Transient infrastructure failures may justify limited retry behavior.

Do not blindly retry deterministic test failures.

---

## 61. Good Retry Candidates

```text
temporary network failure
transient repository failure
ephemeral infrastructure issue
```

---

## 62. Bad Retry Candidates

```text
compilation error
unit test failure
security gate failure
invalid POM
```

---

# PART XXXI — DOCKER AGENTS

## 63. Containerized Agent

Example concept:

```text
Jenkins
 |
v
Docker Agent
 |
+--> JDK
+--> Maven prerequisites
+--> Git
 |
v
Maven Build
```

---

## 64. Benefits

```text
consistent environment
easy versioning
isolated dependencies
```

---

# PART XXXII — KUBERNETES AGENTS

## 65. Dynamic Agents

Enterprise Jenkins can provision temporary build pods.

```text
Jenkins
 |
v
Kubernetes
 |
v
Ephemeral Pod
 |
v
Maven
```

---

## 66. Benefits

```text
elasticity
isolation
resource control
clean environments
```

---

# PART XXXIII — RESOURCE LIMITS

## 67. Maven Build Resources

A Java build may consume significant:

```text
CPU
memory
disk
network
```

Kubernetes/Jenkins agents should have appropriate requests and limits
according to the workload.

---

# PART XXXIV — MAVEN_OPTS

## 68. JVM Memory

Example:

```bash
MAVEN_OPTS="-Xmx2g"
```

Do not blindly increase memory.

Measure:

```text
heap
GC
CPU
container limits
```

---

# PART XXXV — WORKSPACE DISK

## 69. Disk Usage

Large builds can generate:

```text
target/
reports/
logs/
dependency cache
```

Monitor workspace and agent disk.

---

# PART XXXVI — PIPELINE SECURITY

## 70. Jenkinsfile Review

Pipeline code is production automation code.

Review:

```text
shell commands
credentials
URLs
artifact paths
deployment commands
```

---

## 71. Script Approval

Jenkins security controls may require approval for certain scripted
operations.

Do not disable security controls simply to make a pipeline work.

---

# PART XXXVII — CREDENTIAL BINDING

## 72. Credential Usage

Concept:

```groovy
withCredentials([
    usernamePassword(
        credentialsId: 'repo-creds',
        usernameVariable: 'REPO_USER',
        passwordVariable: 'REPO_PASS'
    )
]) {
    sh './mvnw -B deploy'
}
```

Use Jenkins-supported secret handling and avoid printing variables.

---

# PART XXXVIII — SECRET LEAK PREVENTION

## 73. Avoid

```bash
echo $REPO_PASS
```

---

## 74. Avoid Credentials in Command Lines

Command lines may become visible in process lists or logs depending on
the environment.

Prefer secure files/environment injection mechanisms supported by the
CI system and repository tooling.

---

# PART XXXIX — SETTINGS FILE GENERATION

## 75. CI Settings

A pipeline can provide a controlled Maven settings file containing:

```text
mirror
repositories
server IDs
profiles
```

Credentials should be supplied securely rather than hard-coded.

---

# PART XL — ARTIFACTORY PUBLISHING PIPELINE

## 76. Example

```text
Checkout
 |
v
./mvnw -B clean verify
 |
v
Security
 |
v
./mvnw -B deploy
 |
v
Artifactory
```

---

# PART XLI — BUILD INFO / PROVENANCE

## 77. Build Metadata

Enterprise artifact repositories can associate build information with
published artifacts.

Useful metadata includes:

```text
Jenkins job
build number
Git commit
module
dependencies
environment
```

---

## 78. Why?

This enables:

```text
traceability
audit
incident investigation
release tracking
```

---

# PART XLII — VERSIONING

## 79. CI Version

Avoid using arbitrary mutable versions for production artifacts.

Example:

```text
1.8.0
```

or an organization-approved release versioning model.

---

## 80. Build Number

Jenkins build number is useful metadata but should not automatically
replace application semantic versioning.

---

# PART XLIII — MULTI-MODULE MAVEN

## 81. Reactor Build

```text
Jenkins
 |
v
./mvnw -B clean verify
 |
v
Maven Reactor
 |
+--> api
+--> core
+--> service
+--> app
```

---

## 82. Selective Build

For large repositories:

```bash
./mvnw -pl payment-app -am verify
```

can focus on the selected project and required reactor projects.

Validate the impact-analysis strategy before making it the only CI gate.

---

# PART XLIV — PIPELINE FOR MULTI-MODULE

## 83. Build All

```bash
./mvnw -B clean verify
```

---

## 84. Package

```bash
./mvnw -B package
```

---

## 85. Publish

```bash
./mvnw -B deploy
```

Do not unnecessarily rebuild between these operations.

---

# PART XLV — QUALITY + SECURITY FLOW

## 86. Recommended

```text
Checkout
 |
v
Dependency Resolve
 |
v
Compile
 |
v
Unit Test
 |
v
Integration Test
 |
v
Quality
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

The exact ordering can be optimized if the same quality controls remain
effective.

---

# PART XLVI — CONTAINER BUILD

## 87. Application Flow

```text
Maven
 |
v
JAR
 |
v
Container Build
 |
v
Image Registry
```

---

## 88. Do Not Rebuild Unnecessarily

If Maven already produced a validated artifact, the container stage
should consume that exact artifact.

---

# PART XLVII — KUBERNETES DEPLOYMENT

## 89. Deployment Flow

```text
Maven
 |
v
JAR
 |
v
Image
 |
v
Registry
 |
v
Deployment
```

The deployment mechanism may be Jenkins, GitOps or another approved
delivery process.

---

# PART XLVIII — RELEASE GATES

## 90. Example

```text
Build
 |
v
Tests PASS
 |
v
Quality PASS
 |
v
Security PASS
 |
v
Artifact Published
```

---

# PART XLIX — APPROVALS

## 91. Production Approval

Production promotion can require an approval gate depending on
organizational policy.

Example:

```text
STAGE
 |
v
Approval
 |
v
PROD
```

---

# PART L — ROLLBACK

## 92. Artifact-Based Rollback

Prefer rollback to a known-good artifact/image.

```text
Current
 |
X
Previous Known Good
 |
v
Deploy
```

---

## 93. Do Not Rebuild During Rollback

A rollback should normally reuse the previously validated artifact.

---

# PART LI — FAILURE HANDLING

## 94. Post Actions

Jenkins can run post-build actions for:

```text
always
success
failure
unstable
aborted
```

Use them for:

```text
reports
cleanup
notifications
```

---

# PART LII — TEST REPORTING

## 95. Publish Reports

Jenkins should retain relevant test results so engineers can diagnose
failures without rerunning the entire pipeline.

---

# PART LIII — LOGGING

## 96. Build Logs

Include useful information:

```text
commit
JDK
Maven version
module
phase
result
artifact version
```

Do not expose:

```text
passwords
tokens
private keys
```

---

# PART LIV — NOTIFICATIONS

## 97. Failure Notification

Notify appropriate owners when:

```text
build fails
security gate fails
release fails
deployment fails
```

Avoid noisy notifications for every transient event.

---

# PART LV — OBSERVABILITY

## 98. Pipeline Metrics

Track:

```text
queue time
agent provisioning time
checkout time
dependency resolution
compile
tests
security
package
publish
deployment
```

---

## 99. Build Health

Useful metrics:

```text
success rate
failure rate
mean duration
queue time
flaky tests
```

---

# PART LVI — FLAKY TESTS

## 100. Detect

Track tests that:

```text
pass sometimes
fail sometimes
```

---

## 101. Do Not Hide

Do not use unlimited retries to conceal flaky tests.

Treat flakiness as a reliability problem.

---

# PART LVII — REPOSITORY FAILURE

## 102. Maven Cannot Download

Check:

```text
Artifactory
DNS
network
mirror
credentials
cache
```

---

## 103. Maven Cannot Deploy

Check:

```text
distributionManagement
server ID
credentials
permissions
release/snapshot repository
```

---

# PART LVIII — JENKINS FAILURE

## 104. Agent Offline

Check:

```text
agent capacity
network
JVM
Kubernetes pod
container runtime
```

---

## 105. Queue Is Growing

Check:

```text
executor count
agent capacity
resource limits
stuck jobs
```

---

# PART LIX — MAVEN FAILURE

## 106. Compilation Failure

Check:

```text
JDK
Maven
compiler plugin
dependencies
source
```

---

## 107. Test Failure

Check:

```text
test report
environment
test data
dependency
flakiness
```

---

# PART LX — CACHE FAILURE

## 108. Corrupted ~/.m2

Target the affected artifact where possible.

Do not automatically delete the entire Maven repository on every
failure.

---

# PART LXI — JENKINS + MAVEN TROUBLESHOOTING MATRIX

## 109. Error Classification

```text
Dependency  -> repository path
Compile     -> JDK/plugin/source
Test        -> test/environment
Quality     -> quality configuration
Security    -> scanner/policy
Publish     -> repository/auth
Agent       -> Jenkins infrastructure
```

---

# PART LXII — PRODUCTION PIPELINE DESIGN

## 110. Reference

```text
                         Git
                          |
                          v
                       Jenkins
                          |
                  +-------+-------+
                  |               |
                  v               v
             Ephemeral        Credentials
                Agent
                  |
                  v
             Maven Wrapper
                  |
                  v
           Corporate Repository
                  |
                  v
       +----------+----------+
       |                     |
       v                     v
  Dependencies            Plugins
       |                     |
       +----------+----------+
                  |
                  v
               Compile
                  |
                  v
              Unit Test
                  |
                  v
          Integration Test
                  |
                  v
               Quality
                  |
                  v
              Security
                  |
                  v
               Package
                  |
                  v
              Artifact
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
             Deployment
                  |
                  v
             Kubernetes
```

---

# PART LXIII — ENTERPRISE JENKINS ARCHITECTURE

## 111. Controller/Agent Model

```text
                Jenkins Controller
                       |
         +-------------+-------------+
         |             |             |
         v             v             v
      Agent A       Agent B       Agent C
         |             |             |
       Maven         Maven         Maven
         |             |             |
         +-------------+-------------+
                       |
                       v
                Repository Manager
```

---

# PART LXIV — EPHEMERAL KUBERNETES ARCHITECTURE

## 112. Reference

```text
Git
 |
v
Jenkins Controller
 |
v
Kubernetes
 |
+--> Pod Agent
      |
      +--> JDK
      +--> Git
      +--> Maven Wrapper
      |
      v
   Maven Build
      |
      v
Artifactory
```

---

# PART LXV — PRODUCTION SECURITY CHECKLIST

## 113. Jenkins

```text
[ ] RBAC
[ ] protected credentials
[ ] approved plugins
[ ] agent isolation
[ ] pipeline review
[ ] audit logging
```

## 114. Maven

```text
[ ] Maven Wrapper
[ ] controlled JDK
[ ] controlled plugins
[ ] dependency scanning
[ ] trusted repository
```

## 115. Repository

```text
[ ] HTTPS
[ ] least privilege
[ ] release immutability
[ ] snapshot policy
[ ] audit
```

---

# PART LXVI — PRODUCTION PERFORMANCE

## 116. Optimize in Order

Measure:

```text
queue
 |
agent provisioning
 |
checkout
 |
dependency resolution
 |
compile
 |
tests
 |
security
 |
publish
```

Optimize the largest bottleneck first.

---

# PART LXVII — COMMON ANTI-PATTERNS

## 117. Hard-Coded Maven

```groovy
sh 'mvn clean install'
```

when the project expects a different Maven version.

Prefer the wrapper.

---

## 118. Hard-Coded Credentials

Never:

```groovy
sh "mvn deploy -Dpassword=${PASSWORD}"
```

if this causes secrets to appear in logs or process arguments.

Use secure credential mechanisms.

---

## 119. Build on Controller

Avoid using the Jenkins controller as a general build machine.

---

## 120. Permanent Dirty Agents

Long-lived agents can accumulate:

```text
old caches
old workspaces
old tools
```

Use controlled lifecycle and cleanup.

---

## 121. Unlimited Retry

Retries can hide deterministic failures.

---

## 122. Publish Before Testing

Never publish a production artifact before required tests and security
gates.

---

# PART LXVIII — INTERVIEW PREPARATION

## 123. How Do Jenkins and Maven Work Together?

Answer:

```text
Jenkins orchestrates the CI/CD pipeline while Maven performs the Java
build lifecycle. Jenkins invokes Maven for dependency resolution,
compilation, testing, packaging and, when appropriate, deployment to
the artifact repository.
```

## 124. Why Use Maven Wrapper in Jenkins?

Answer:

```text
It makes the Maven version part of the project and reduces differences
between developer and CI environments.
```

## 125. How Do You Configure Maven Credentials?

Answer:

```text
I keep repository credentials in Jenkins's protected credential store
and inject them into Maven settings or another supported mechanism.
I never commit credentials to the POM or source repository.
```

## 126. How Do You Publish to Artifactory?

Answer:

```text
I configure the appropriate repository and server ID, provide
least-privilege CI credentials securely, run the required validation
and security gates, and then execute Maven deploy to publish the
validated artifact.
```

## 127. How Do You Optimize Maven Builds in Jenkins?

Answer:

```text
I measure stage and module timings first. Then I optimize dependency
caching, agent provisioning, Maven reactor parallelism, test execution
and selective builds while maintaining correctness and reproducibility.
```

## 128. How Do You Handle Maven Dependency Failures?

Answer:

```text
I inspect the exact Maven error, dependency coordinates, effective
settings, mirror, repository health, credentials, DNS, TLS and cache.
I first determine whether the issue is project-specific or shared
infrastructure.
```

## 129. How Do You Secure Jenkins-Maven Pipelines?

Answer:

```text
I use RBAC, protected credentials, ephemeral agents where practical,
trusted Maven repositories, controlled Maven/plugin versions, security
scanning, reviewed pipeline code and least-privilege publishing
identities.
```

## 130. Why Build Once?

Answer:

```text
Building once produces one tested artifact that can be promoted
through environments. Rebuilding per environment can introduce
different dependencies, timestamps, toolchains or source state and
reduces release reproducibility.
```

---

# PART LXIX — SENIOR-LEVEL SCENARIOS

## 131. Jenkins Builds Suddenly Take Twice as Long

Answer:

```text
I compare pipeline stage timings with the previous baseline and
determine whether the regression is in queueing, agent provisioning,
dependency resolution, compilation, tests, security scanning or
publishing. I then investigate the measured bottleneck rather than
blindly increasing executors or Maven threads.
```

## 132. All Jenkins Jobs Fail at Dependency Download

Answer:

```text
Because the failure affects many jobs, I treat it as shared
infrastructure first. I check Artifactory, mirrors, DNS, TLS, network,
authentication and repository availability before changing project
POMs.
```

## 133. Developer Build Works but Jenkins Fails

Answer:

```text
I compare JDK, Maven, wrapper version, settings.xml, profiles,
repository access, credentials, environment variables and local cache.
I remove hidden local state and reproduce with a clean environment.
```

## 134. Jenkins Publishes a Broken Artifact

Answer:

```text
I stop further promotion, identify which commit and build produced the
artifact, inspect test/security gates and repository provenance, and
prevent promotion of the artifact. I then restore the last known-good
artifact where appropriate and fix the pipeline gate that failed.
```

## 135. Jenkins Agent Runs Out of Memory

Answer:

```text
I inspect Maven heap, test JVMs, parallel threads, container limits
and concurrent builds. I would not simply increase memory without
checking whether parallelism or a memory leak is causing the issue.
```

## 136. Maven Cache Causes Intermittent Builds

Answer:

```text
I compare warm-cache and cold-cache builds, inspect the affected
artifacts and cache keys, and isolate jobs from unsafe shared state.
The cache should improve performance, not become a hidden dependency
for correctness.
```

## 137. Production Deployment Needs Rollback

Answer:

```text
I identify the exact previously validated artifact/image and deploy
that immutable version. I avoid rebuilding the source during rollback
because rollback should restore a known-good release.
```

## 138. A Pipeline Wants to Use Admin Credentials

Answer:

```text
I would reject that design unless there is a documented exceptional
requirement. I would create a dedicated service identity with only the
permissions required for the operation.
```

---

# PART LXX — GOLDEN RULES

## 139. Rules

```text
1. Jenkins orchestrates; Maven builds.

2. Use Maven Wrapper in CI when maintained by the project.

3. Control JDK and Maven versions.

4. Treat Jenkins agents as build infrastructure.

5. Prefer ephemeral agents where practical.

6. Keep build work off the Jenkins controller.

7. Clean or replace dirty workspaces.

8. Use protected credentials.

9. Never commit repository credentials.

10. Use least-privilege CI identities.

11. Separate read and publish permissions.

12. Secure Maven settings.

13. Match Maven server IDs correctly.

14. Use approved internal repositories.

15. Prefer controlled repository mirrors.

16. Use Artifactory or approved repository management for enterprise
    dependency distribution.

17. Run clean verify for appropriate CI validation.

18. Publish only after required gates pass.

19. Understand package versus deploy.

20. Keep release artifacts immutable.

21. Separate snapshots from releases.

22. Build once and promote.

23. Do not rebuild during rollback.

24. Preserve commit-to-artifact traceability.

25. Publish test reports.

26. Monitor flaky tests.

27. Do not use retries to hide deterministic failures.

28. Use timeouts for stuck jobs.

29. Retry only plausible transient failures.

30. Control Maven dependency caching.

31. Test cold-cache builds.

32. Do not blindly delete the entire ~/.m2 repository.

33. Measure build stages before optimizing.

34. Use Maven parallelism carefully.

35. Validate plugin thread safety.

36. Validate test isolation before parallelizing tests.

37. Monitor agent CPU and memory.

38. Monitor workspace disk.

39. Monitor Jenkins queue time.

40. Monitor repository latency.

41. Monitor artifact publishing failures.

42. Treat Jenkinsfiles as production code.

43. Review shell commands in pipelines.

44. Protect secrets from logs.

45. Avoid secrets in command-line arguments.

46. Scan dependencies.

47. Scan build tooling where supported.

48. Maintain approved Jenkins plugins.

49. Use RBAC.

50. Audit administrative actions.

51. Use multi-module reactor builds appropriately.

52. Use -pl and -am carefully for selective builds.

53. Do not sacrifice dependency impact coverage for speed.

54. Consume the exact Maven artifact when building a container.

55. Do not compile the application again unnecessarily in the container
    stage.

56. Separate Maven artifact repositories from container registries.

57. Use artifact provenance/build information where available.

58. Keep build environments reproducible.

59. Maintain backup and recovery for critical CI configuration.

60. Design Jenkins for agent failure.

61. Design repository access for repository failure.

62. Distinguish application failures from shared infrastructure
    failures.

63. Investigate the first meaningful failure, not every downstream
    symptom.

64. Keep release approval separate from routine validation when policy
    requires it.

65. Automate quality and security gates.

66. Preserve logs and reports for incident investigation.

67. Avoid permanent mutable agents.

68. Avoid uncontrolled global Maven configuration.

69. Version build tooling deliberately.

70. Validate exact Jenkins, Maven, JDK, plugin, repository and agent
    behavior for the versions and production architecture actually
    used.
```

---

# END OF 09-Maven-Jenkins-Integration.md
