# 18-Build-and-Package-Management
# 18-Build-Troubleshooting

## 1. Purpose

Build troubleshooting is the systematic process of identifying,
isolating, correcting, and preventing failures in build, package,
artifact, and CI/CD workflows.

Production troubleshooting should follow:

```text
Observe
   |
   v
Classify
   |
   v
Reproduce
   |
   v
Isolate
   |
   v
Validate
   |
   v
Fix
   |
   v
Prevent recurrence
```

Never begin by randomly changing multiple things.

The objective is to identify:

```text
what failed
where it failed
why it failed
what changed
what is affected
how to recover safely
how to prevent recurrence
```

---

# PART I — TROUBLESHOOTING FUNDAMENTALS

## 2. Build Failure Categories

Most failures fall into:

```text
source
dependency
toolchain
configuration
code
test
security
network
repository
authentication
authorization
resource
artifact
deployment
environment
```

Classify first.

---

## 3. First Five Questions

Ask:

```text
1. What exactly failed?
2. Which stage failed?
3. Did it ever work?
4. What changed?
5. Can it be reproduced?
```

These questions prevent premature fixes.

---

## 4. Failure Timeline

Establish:

```text
last known good build
        |
        v
change
        |
        v
first failed build
        |
        v
current state
```

The change between the last good and first bad build is often the
highest-value investigation area.

---

# PART II — PIPELINE OBSERVABILITY

## 5. Pipeline Evidence

Collect:

```text
job logs
stage duration
exit code
commit SHA
build number
runner
tool versions
dependency versions
artifact version
environment
```

---

## 6. Exit Codes

A non-zero exit code means the command failed, but the exit code alone
may not identify the root cause.

Read the surrounding logs.

---

## 7. Log Levels

Useful levels:

```text
normal
verbose
debug
trace
```

Use increased verbosity temporarily and protect secrets.

---

# PART III — REPRODUCTION

## 8. Reproduce Locally

If possible:

```text
CI failure
 |
v
same source
 |
v
same tool versions
 |
v
same command
```

But remember that local success does not prove CI correctness.

---

## 9. Reproduce in Clean Environment

Best reproduction:

```text
clean runner
clean workspace
controlled dependencies
same command
```

---

# PART IV — BUILD ENVIRONMENT

## 10. Tool Version Mismatch

Example:

```text
local Java 21
CI Java 17
```

The build may behave differently.

Check:

```bash
java -version
mvn -version
node --version
npm --version
python --version
```

---

## 11. Environment Drift

Symptoms:

```text
works locally
fails in CI
```

Check:

```text
OS
JDK
Node
Python
Maven
npm
compiler
environment variables
```

---

# PART V — CLEAN BUILD

## 12. Clean Build

Maven:

```bash
./mvnw clean verify
```

npm:

```bash
rm -rf node_modules
npm ci
npm test
```

Python environments should be recreated using the project's defined
dependency mechanism.

---

# PART VI — SOURCE FAILURES

## 13. Compile Failure

Check:

```text
syntax
imports
API compatibility
compiler version
generated source
dependency version
```

---

## 14. Generated Source Failure

If code is generated during build:

```text
generator
 |
v
generated source
 |
v
compiler
```

Check whether generation completed successfully.

---

# PART VII — DEPENDENCY FAILURES

## 15. Dependency Not Found

Symptoms:

```text
Could not resolve dependency
package not found
module not found
```

Check:

```text
repository URL
package name
version
credentials
network
repository availability
```

---

## 16. Wrong Dependency Version

Inspect:

```text
direct dependency
transitive dependency
lockfile
dependency management
```

---

# PART VIII — MAVEN TROUBLESHOOTING

## 17. Maven Dependency Tree

Useful command:

```bash
./mvnw dependency:tree
```

It helps identify transitive dependency resolution.

---

## 18. Maven Effective Configuration

When configuration is confusing, inspect the effective Maven model and
settings rather than guessing.

---

## 19. Maven Cache

Local Maven cache:

```text
~/.m2/repository
```

A corrupted local dependency can cause unexpected failures.

---

## 20. Maven Cache Recovery

If corruption is suspected:

```text
identify affected dependency
 |
v
remove only affected local cache
 |
v
rerun build
```

Do not routinely delete the entire cache without evidence.

---

# PART IX — NPM TROUBLESHOOTING

## 21. npm Dependency Failure

Check:

```bash
node --version
npm --version
npm ci
```

Then inspect:

```text
package-lock.json
registry
package version
peer dependencies
```

---

## 22. npm Peer Dependency

Errors may result from incompatible peer dependencies.

Check:

```text
package versions
peerDependencies
dependency graph
```

Do not use force options as the default production fix.

---

# PART X — PYTHON TROUBLESHOOTING

## 23. Python Module Not Found

Check:

```bash
python --version
python -m pip --version
python -m pip list
```

Ensure pip and Python refer to the intended environment.

---

## 24. Virtual Environment

Confirm:

```text
correct interpreter
correct virtualenv
correct dependency set
```

---

# PART XI — PACKAGE BUILD FAILURES

## 25. Maven Package

Check:

```text
pom.xml
plugin versions
resources
compiler
tests
```

---

## 26. npm Build

Check:

```text
scripts
Node version
environment variables
dependencies
bundler configuration
```

---

## 27. Python Build

Check:

```text
pyproject.toml
build backend
package metadata
Python version
wheel configuration
```

---

# PART XII — TEST FAILURES

## 28. Unit Test Failure

Determine:

```text
code regression
test defect
environment issue
dependency change
timing issue
```

---

## 29. Flaky Test

If a test:

```text
passes
fails
passes
```

investigate:

```text
timing
parallelism
shared state
external service
randomness
test order
```

Do not simply add unlimited retries.

---

# PART XIII — INTEGRATION TESTS

## 30. Integration Failure

Check dependent services:

```text
database
queue
API
DNS
credentials
network
schema
```

---

## 31. Service Availability

Confirm:

```text
service running
port open
DNS resolves
TLS works
credentials valid
```

---

# PART XIV — NETWORK TROUBLESHOOTING

## 32. Repository Connection

Check:

```text
DNS
TCP
TLS
proxy
firewall
routing
```

---

## 33. DNS

Symptoms:

```text
Could not resolve host
```

Check:

```bash
nslookup repository.company.com
```

or the approved DNS diagnostic available in the runner.

---

## 34. TLS

Symptoms:

```text
certificate error
SSL handshake
PKIX path
```

Check:

```text
certificate chain
trusted CA
hostname
system time
proxy interception
```

---

# PART XV — PROXY

## 35. Proxy Failure

Check:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

and CI runner configuration.

Do not expose credentials embedded in proxy URLs.

---

# PART XVI — ARTIFACTORY TROUBLESHOOTING

## 36. Repository Failure

Check:

```text
repository URL
repository exists
package type
permissions
authentication
storage
health
```

---

## 37. 401 Unauthorized

Usually investigate:

```text
credential
token
authentication endpoint
token expiration
```

---

## 38. 403 Forbidden

Usually investigate:

```text
authenticated identity
repository permission
path permission
deployment permission
```

---

## 39. 404 Not Found

Possible causes:

```text
wrong repository
wrong coordinates
wrong version
package not published
incorrect virtual repository
```

---

# PART XVII — PUBLISHING FAILURES

## 40. Duplicate Artifact

Example:

```text
1.5.0 already exists
```

Do not overwrite a production release.

Use a new version if content changed.

---

## 41. Upload Timeout

Check:

```text
artifact size
network
proxy
repository health
runner resources
```

---

## 42. Checksum Failure

Check:

```text
artifact generation
transfer
local cache
repository integrity
```

---

# PART XVIII — DOCKER TROUBLESHOOTING

## 43. Docker Build Failure

Check:

```text
Dockerfile
build context
base image
network
BuildKit
permissions
disk
```

---

## 44. Docker Build Context

A huge context can slow builds.

Use:

```text
.dockerignore
```

---

## 45. Base Image Pull Failure

Check:

```text
registry
credentials
network
image name
tag/digest
rate limits
```

---

## 46. Docker Disk Full

Check:

```bash
docker system df
df -h
```

Clean only according to runner policy.

---

# PART XIX — CONTAINER RUNTIME FAILURES

## 47. Image Starts Locally but Not in Kubernetes

Check:

```text
architecture
environment variables
secrets
ports
filesystem permissions
user
health checks
```

---

## 48. Wrong Architecture

Example:

```text
amd64 image
 |
v
arm64 node
```

Verify image platform support.

---

# PART XX — HELM TROUBLESHOOTING

## 49. Helm Package Failure

Check:

```text
Chart.yaml
templates
values
dependencies
syntax
```

---

## 50. Helm Template

Render locally:

```bash
helm template release ./chart
```

This can identify template rendering problems before deployment.

---

# PART XXI — SECURITY FAILURES

## 51. Secret Scan Failure

Do not immediately bypass.

Determine:

```text
real secret
false positive
location
exposure
```

---

## 52. Real Secret Found

```text
stop
 |
v
revoke
 |
v
rotate
 |
v
audit
 |
v
remove exposure
```

---

## 53. SAST Failure

Inspect:

```text
rule
file
line
data flow
severity
```

---

## 54. SCA Failure

Identify:

```text
package
version
CVE
severity
runtime usage
fixed version
```

---

# PART XXII — CONTAINER SECURITY

## 55. Image Scan Failure

Check:

```text
base OS
OS package
application dependency
severity
fixed version
```

---

## 56. No Fixed Version

If no fixed version exists:

```text
assess exploitability
mitigate
document exception
monitor upstream
```

---

# PART XXIII — CI RUNNER

## 57. Runner Offline

Check:

```text
runner service
network
registration
capacity
resource exhaustion
```

---

## 58. Runner Disk Full

Check:

```bash
df -h
du -sh <workspace>
```

Then clean according to runner policy.

---

## 59. Runner Memory Pressure

Symptoms:

```text
process killed
OOM
slow builds
random failures
```

Check:

```text
memory
parallel jobs
build process
container limits
```

---

# PART XXIV — CPU

## 60. High CPU

Determine whether CPU is consumed by:

```text
compiler
tests
Docker build
security scanner
dependency processing
```

Scale or optimize only after identifying the bottleneck.

---

# PART XXV — TIMEOUTS

## 61. Build Timeout

Check stage duration.

Possible causes:

```text
dependency download
test hang
network
scanner
deadlock
resource starvation
```

---

# PART XXVI — CONCURRENCY

## 62. Concurrent Release

Two builds may attempt:

```text
same version
same repository
same deployment
```

Use release concurrency controls.

---

# PART XXVII — CACHE TROUBLESHOOTING

## 63. Cache Corruption

Symptoms:

```text
random dependency errors
inconsistent builds
checksum mismatch
```

Compare:

```text
cache hit
cache miss
clean build
```

---

## 64. Cache Is Required for Success

This indicates an unhealthy build.

A clean build should succeed without an optimization cache.

---

# PART XXVIII — ARTIFACT TROUBLESHOOTING

## 65. Artifact Missing

Check:

```text
build output
package path
publication logs
repository
version
permissions
```

---

## 66. Wrong Artifact

Compare:

```text
version
checksum
digest
build ID
commit SHA
```

---

# PART XXIX — SOURCE-TO-ARTIFACT TRACEABILITY

## 67. Trace

```text
Production
 |
v
artifact digest
 |
v
repository
 |
v
build ID
 |
v
commit SHA
```

If this chain is missing, improve build metadata.

---

# PART XXX — PROVENANCE TROUBLESHOOTING

## 68. Missing Provenance

Check:

```text
build metadata generation
CI permissions
attestation step
artifact association
```

---

# PART XXXI — SBOM TROUBLESHOOTING

## 69. Missing SBOM

Check:

```text
generation tool
pipeline step
artifact association
storage
publication
```

---

# PART XXXII — SIGNING TROUBLESHOOTING

## 70. Signature Failure

Check:

```text
signing identity
key availability
artifact digest
certificate
trust store
verification policy
```

---

# PART XXXIII — RELEASE VERSIONING

## 71. Version Mismatch

Example:

```text
Git tag: v1.5.0
POM: 1.4.9
```

Stop publication.

The release metadata must be corrected before publishing.

---

# PART XXXIV — BUILD CONFIGURATION

## 72. Environment Variables

Check:

```text
missing
wrong value
wrong scope
masked incorrectly
```

Do not print sensitive values.

---

## 73. Configuration Drift

Compare:

```text
working build
failed build
```

environment and configuration.

---

# PART XXXV — JAVA TROUBLESHOOTING

## 74. Java Version

Check:

```bash
java -version
javac -version
./mvnw -version
```

Ensure the Maven runtime and compiler use the intended JDK.

---

## 75. Java Memory

Possible errors:

```text
OutOfMemoryError
GC overhead
forked process killed
```

Investigate:

```text
heap
runner memory
parallelism
test JVMs
```

---

# PART XXXVI — NODE TROUBLESHOOTING

## 76. Node Version

Check:

```bash
node --version
npm --version
```

Use the version declared by the project.

---

## 77. Native npm Modules

Failures may depend on:

```text
OS
Node ABI
compiler
Python
system libraries
```

---

# PART XXXVII — PYTHON TROUBLESHOOTING

## 78. Python Version

Check:

```bash
python --version
python -m pip --version
```

---

## 79. Native Python Dependency

Check:

```text
compiler
OS packages
Python ABI
wheel availability
```

---

# PART XXXVIII — PACKAGE MANAGER REGISTRY

## 80. Wrong Registry

Verify:

```text
Maven repository
npm registry
PyPI index
Docker registry
Helm registry
```

---

# PART XXXIX — AUTHENTICATION

## 81. Credential Debugging

Do not print credentials.

Instead inspect:

```text
identity
credential existence
expiration
scope
target repository
```

---

# PART XL — AUTHORIZATION

## 82. Permission Debugging

Check:

```text
user/service identity
repository
path
operation
policy
```

---

# PART XLI — CI SECRET INJECTION

## 83. Secret Missing

Possible causes:

```text
wrong environment
protected variable
wrong branch
permission
secret name
environment scope
```

---

# PART XLII — GITHUB ACTIONS

## 84. Workflow Failure

Check:

```text
workflow trigger
permissions
runner
action version
secrets
environment protection
```

---

## 85. Actions Permissions

A workflow can fail because its token does not have the required
permission.

Grant only the specific permission required.

---

# PART XLIII — JENKINS

## 86. Jenkins Failure

Check:

```text
agent
workspace
credentials
tool installation
pipeline syntax
plugins
network
```

---

## 87. Jenkins Agent

If only one agent fails:

```text
compare healthy agent
```

Do not immediately change application code.

---

# PART XLIV — GITLAB CI

## 88. GitLab Runner

Check:

```text
runner tags
runner status
variables
protected variables
image
services
```

---

# PART XLV — DEPLOYMENT HANDOFF

## 89. Build-to-Deployment

If deployment receives the wrong artifact:

```text
build artifact
 |
v
CI artifact transfer
 |
v
repository
 |
v
deployment manifest
```

verify every boundary.

---

# PART XLVI — KUBERNETES

## 90. ImagePullBackOff

Check:

```text
image name
tag/digest
registry
credentials
network
```

---

## 91. CrashLoopBackOff

This is usually a runtime problem rather than a build problem.

Check:

```bash
kubectl logs
kubectl describe pod
```

and application configuration.

---

# PART XLVII — GITOPS

## 92. Argo CD Shows Wrong Version

Check:

```text
Git commit
manifest
image tag/digest
Argo CD sync
Kubernetes object
```

---

# PART XLVIII — ROLLBACK

## 93. Rollback Failure

Check:

```text
previous artifact exists
artifact immutable
database compatibility
configuration compatibility
deployment history
```

---

# PART XLIX — DATABASE

## 94. Migration Failure

Check:

```text
schema version
migration ordering
permissions
locking
backward compatibility
```

Do not assume application rollback automatically reverses database
changes.

---

# PART L — INTERMITTENT FAILURES

## 95. Random Build Failures

Look for:

```text
race conditions
shared state
network instability
resource contention
dependency availability
flaky tests
cache behavior
```

---

# PART LI — RACE CONDITIONS

## 96. Parallel Jobs

A race can occur when jobs modify:

```text
same file
same cache
same database
same artifact version
same temporary resource
```

Use isolated resources.

---

# PART LII — DEPENDENCY AVAILABILITY

## 97. External Registry Failure

If external dependency access fails:

```text
check enterprise cache
check mirror
check repository health
```

Avoid uncontrolled bypasses.

---

# PART LIII — RATE LIMITING

## 98. Registry Rate Limit

Symptoms:

```text
429 Too Many Requests
```

Solutions may include:

```text
enterprise caching
request reduction
controlled concurrency
approved mirror
```

---

# PART LIV — STORAGE

## 99. Repository Storage Full

Impact:

```text
publish failure
metadata failure
build failure
```

Response:

```text
check growth
protect releases
clean eligible artifacts
expand storage
```

---

# PART LV — DISK CLEANUP

## 100. Runner Cleanup

Remove only approved disposable data:

```text
old workspaces
temporary files
unused container layers
```

Do not delete active artifacts blindly.

---

# PART LVI — PIPELINE PERFORMANCE

## 101. Slow Build

Measure:

```text
queue
checkout
dependency restore
compile
tests
security
package
upload
```

---

## 102. Slow Dependency Restore

Investigate:

```text
network
registry
cache
dependency count
parallelism
```

---

# PART LVII — SLOW TESTS

## 103. Test Optimization

Identify:

```text
slowest tests
test setup
external dependencies
parallelization opportunities
```

---

# PART LVIII — SLOW DOCKER BUILD

## 104. Docker Optimization

Review:

```text
build context
layer ordering
dependency installation
cache usage
base image
```

---

# PART LIX — SLOW SECURITY SCAN

## 105. Scan Optimization

Use:

```text
incremental scanning
parallel scanning
cached databases
appropriate scanner configuration
```

Do not disable mandatory security gates merely to reduce duration.

---

# PART LX — PRODUCTION INCIDENT PROCESS

## 106. Incident Steps

```text
1. Detect.
2. Confirm.
3. Assess impact.
4. Contain.
5. Preserve evidence.
6. Identify root cause.
7. Recover.
8. Validate.
9. Document.
10. Prevent recurrence.
```

---

# PART LXI — ROOT CAUSE ANALYSIS

## 107. Five Whys

Example:

```text
Why did deployment fail?
 -> artifact missing.

Why missing?
 -> publish job failed.

Why?
 -> repository permission changed.

Why?
 -> RBAC update removed path permission.

Why?
 -> permission change was not tested in staging.
```

The corrective action is broader than simply rerunning the build.

---

# PART LXII — CORRECTIVE ACTION

## 108. Good Fix

A good fix addresses:

```text
immediate problem
underlying cause
systemic prevention
```

---

# PART LXIII — POST-INCIDENT

## 109. Review

Record:

```text
timeline
impact
root cause
detection
recovery
actions
owner
deadline
```

---

# PART LXIV — PRODUCTION TROUBLESHOOTING RUNBOOK

## 110. First Response

```text
[ ] Identify failed pipeline/build.
[ ] Record build ID.
[ ] Record commit SHA.
[ ] Identify failed stage.
[ ] Capture error.
[ ] Check recent changes.
[ ] Compare last successful build.
[ ] Check runner health.
[ ] Check repository health.
[ ] Check dependency availability.
[ ] Check security gates.
```

---

# PART LXV — BUILD RUNBOOK

## 111. Clean Reproduction

```text
[ ] same source
[ ] same toolchain
[ ] clean workspace
[ ] controlled dependencies
[ ] same command
[ ] same runner class
```

---

# PART LXVI — PUBLISH RUNBOOK

## 112. Artifact Publication

```text
[ ] version unique
[ ] artifact generated
[ ] tests passed
[ ] security passed
[ ] SBOM generated
[ ] provenance recorded
[ ] repository available
[ ] credentials valid
[ ] permission valid
[ ] artifact immutable
```

---

# PART LXVII — RELEASE RUNBOOK

## 113. Release

```text
[ ] exact tag
[ ] correct version
[ ] immutable artifact
[ ] artifact digest
[ ] promotion record
[ ] approval
[ ] deployment health
[ ] rollback version available
```

---

# PART LXVIII — TROUBLESHOOTING MATRIX

## 114. Common Errors

| Error | Likely areas |
|---|---|
| Compilation error | source, JDK, dependency |
| Module not found | dependency, registry |
| 401 | authentication |
| 403 | authorization |
| 404 | repository/path/version |
| 409 | duplicate artifact |
| 429 | rate limit |
| TLS error | certificate, proxy, CA |
| Timeout | network, resource, service |
| OOM | memory, parallelism |
| Disk full | workspace, cache, Docker |
| Test flaky | timing, state, external service |
| Image pull failure | registry, image, credentials |
| Wrong artifact | version, digest, promotion |
| Security gate | CVE, SAST, secret, policy |

---

# PART LXIX — INTERVIEW PREPARATION

## 115. How Do You Troubleshoot a Failed Build?

Answer:

```text
I first identify the failed stage and exact error, then compare it with
the last known-good build and inspect recent changes. I verify the
toolchain, dependencies, environment, runner and repository before
reproducing the issue in a clean environment. Once the root cause is
identified I apply the smallest safe fix, rerun validation and document
the preventive action.
```

## 116. Build Works Locally but Fails in CI. What Do You Check?

Answer:

```text
I compare OS, JDK, Node, Python, package-manager versions, environment
variables, dependency resolution, credentials, network access, workspace
state and caches. I then reproduce using the CI toolchain and a clean
environment.
```

## 117. Maven Dependency Cannot Be Downloaded.

Answer:

```text
I check the dependency coordinates, repository URL, authentication,
authorization, DNS/TLS/network connectivity and repository availability.
Then I inspect dependency resolution and the local Maven cache.
```

## 118. How Do You Troubleshoot npm?

Answer:

```text
I verify Node and npm versions, lockfile state, registry configuration,
package availability and peer dependencies. I prefer npm ci for
lockfile-based CI installs and reproduce from a clean workspace.
```

## 119. How Do You Troubleshoot Python Builds?

Answer:

```text
I verify Python and pip versions, the active environment, dependency
constraints, build backend, native dependencies and package metadata.
I reproduce in a clean virtual environment.
```

## 120. Artifactory Returns 403. What Do You Do?

Answer:

```text
I confirm the authenticated service identity and inspect repository and
path permissions for the requested operation. A 403 generally means
the identity is recognized but is not authorized for that operation or
resource.
```

## 121. Artifact Already Exists. What Do You Do?

Answer:

```text
I do not overwrite it. I verify whether the release is already
published and use a new version if the artifact content needs to
change.
```

## 122. How Do You Troubleshoot a Docker Build?

Answer:

```text
I inspect the Dockerfile, build context, base image, registry access,
BuildKit output, filesystem and runner resources. For slow builds I
measure context transfer, dependency installation and layer cache
behavior.
```

## 123. Pipeline Is Too Slow. How Do You Improve It?

Answer:

```text
I measure each stage first. Then I optimize the largest bottleneck
using safe caching, parallel jobs, test parallelization, dependency
mirrors, appropriate runner sizing and incremental strategies without
removing required security or quality gates.
```

---

# PART LXX — SENIOR PRODUCTION SCENARIOS

## 124. Every Other Build Fails

Answer:

```text
I suspect nondeterminism: race conditions, shared state, flaky tests,
network instability, resource contention or cache corruption. I compare
successful and failed runs and isolate the variable that changes.
```

## 125. Only One Runner Fails

Answer:

```text
I compare that runner against a healthy runner: tool versions, disk,
memory, network, workspace, container runtime and configuration. If
the runner is unhealthy, I replace or rebuild it rather than changing
application code without evidence.
```

## 126. All Builds Suddenly Fail to Download Dependencies

Answer:

```text
I check repository health, DNS, TLS, credentials, proxy and network
changes first. If the repository is unavailable, I check the approved
enterprise cache or mirror and follow the outage process.
```

## 127. Only Production Release Fails

Answer:

```text
I compare release-specific permissions, protected environment rules,
tag triggers, version validation, repository write permissions,
signing credentials and approval requirements against a successful
lower-environment pipeline.
```

## 128. Same Version Was Published Twice

Answer:

```text
I investigate how immutability was bypassed, identify whether consumers
received different content, preserve evidence and immediately correct
repository and pipeline permissions.
```

## 129. Production Artifact Cannot Be Reproduced

Answer:

```text
I compare source commit, toolchain, dependency versions, build
environment and generated inputs. Missing provenance or uncontrolled
dependencies usually indicate a reproducibility gap.
```

## 130. Build Fails Only With Cold Cache

Answer:

```text
I treat this as a build defect. Cache should accelerate a build, not
provide missing inputs. I identify undeclared dependencies or generated
files that exist only in the warm workspace.
```

## 131. Security Scan Is the Slowest Stage

Answer:

```text
I measure scanner database access, artifact size, dependency graph and
scanner configuration. I use safe caching, parallelization or
incremental analysis where supported while preserving required gates.
```

## 132. Docker Image Works on AMD64 but Fails on ARM64

Answer:

```text
I inspect image platform manifests and native dependencies. I build or
publish the required multi-platform image and validate each target
architecture.
```

## 133. Artifact Is Correct but Deployment Uses Old Version

Answer:

```text
I trace the artifact identity through the repository, deployment
manifest, GitOps commit, Argo CD synchronization and Kubernetes object.
I compare immutable digests rather than relying on tags alone.
```

## 134. Database Migration Succeeded but Application Rollback Fails

Answer:

```text
I separate schema rollback from application rollback. I check whether
the previous application version is compatible with the new schema and
use backward-compatible expand/contract migrations in future releases.
```

## 135. Build Credentials Are Expired

Answer:

```text
I verify credential expiration and scope, rotate or renew through the
approved secret mechanism, test access without exposing the secret and
prefer short-lived identity mechanisms where possible.
```

## 136. Registry Returns 429

Answer:

```text
I identify whether the issue is external rate limiting or internal
repository capacity. I reduce uncontrolled concurrency and use the
approved enterprise cache or mirror rather than bypassing governance.
```

---

# PART LXXI — GOLDEN RULES

## 137. Rules 1

```text
1. Classify before fixing.
2. Read the exact error.
3. Identify the failed stage.
4. Find the last known-good build.
5. Identify recent changes.
6. Compare good and bad runs.
7. Reproduce cleanly.
8. Change one major variable at a time.
9. Preserve evidence.
10. Record build ID.
11. Record commit SHA.
12. Record runner.
13. Record tool versions.
14. Record dependency versions.
15. Record artifact version.
16. Never guess credentials from logs.
17. Never print secrets.
18. Use controlled debug logging.
19. Protect troubleshooting logs.
20. Fix root cause, not only symptoms.
```

## 138. Rules 2

```text
21. Verify Java version.
22. Verify Maven version.
23. Verify Node version.
24. Verify npm version.
25. Verify Python version.
26. Verify package-manager configuration.
27. Verify repository endpoint.
28. Verify DNS.
29. Verify TLS.
30. Verify proxy.
31. Verify firewall.
32. Verify network route.
33. Verify authentication.
34. Verify authorization.
35. Verify repository health.
36. Verify package coordinates.
37. Verify package version.
38. Verify dependency resolution.
39. Verify transitive dependencies.
40. Verify cache state.
```

## 139. Rules 3

```text
41. Use clean workspaces.
42. Do not assume local success means CI success.
43. Test cold-cache builds.
44. Treat caches as optimization.
45. Never cache secrets.
46. Scope shared caches.
47. Investigate flaky tests.
48. Do not hide failures with unlimited retries.
49. Separate code failure from environment failure.
50. Separate build failure from deployment failure.
51. Inspect Maven dependency trees.
52. Inspect npm lockfiles.
53. Inspect Python environments.
54. Inspect Docker build context.
55. Inspect Helm templates.
56. Verify artifact contents.
57. Verify checksums/digests.
58. Verify source-to-artifact lineage.
59. Verify artifact-to-deployment lineage.
60. Never overwrite production artifacts.
```

## 140. Rules 4

```text
61. Treat 401 as an authentication investigation.
62. Treat 403 as an authorization investigation.
63. Treat 404 as a resource/path/version investigation.
64. Treat 409 as a conflict investigation.
65. Treat 429 as a rate-limit investigation.
66. Treat TLS failures as trust/connectivity issues.
67. Treat OOM as a resource investigation.
68. Treat disk-full as a capacity investigation.
69. Treat timeout as a stage/resource/network investigation.
70. Check runner health before changing application code.
71. Prefer ephemeral runners.
72. Clean persistent runners.
73. Monitor runner capacity.
74. Monitor repository capacity.
75. Monitor dependency availability.
76. Use approved mirrors.
77. Use enterprise dependency caches.
78. Do not bypass repository security.
79. Do not manually copy unverified artifacts.
80. Preserve immutable release identity.
```

## 141. Rules 5

```text
81. Stop unsafe releases.
82. Quarantine suspicious artifacts.
83. Revoke compromised credentials.
84. Rotate exposed secrets.
85. Audit compromised runners.
86. Audit compromised artifacts.
87. Protect signing keys.
88. Preserve forensic evidence.
89. Use SBOMs for impact analysis.
90. Use provenance for traceability.
91. Use signatures where required.
92. Validate security findings.
93. Do not blindly suppress scanners.
94. Use time-limited exceptions.
95. Give exceptions owners.
96. Give exceptions expiry dates.
97. Keep production approval controls.
98. Protect release tags.
99. Protect release workflows.
100. Protect production credentials.
```

## 142. Rules 6

```text
101. Measure pipeline duration.
102. Measure queue time.
103. Measure test duration.
104. Measure security scan duration.
105. Measure artifact upload duration.
106. Optimize the largest bottleneck.
107. Parallelize independent jobs.
108. Cache safely.
109. Use appropriate runner sizing.
110. Use sensible timeouts.
111. Retry only transient failures.
112. Avoid infinite retries.
113. Prevent release concurrency collisions.
114. Use unique versions.
115. Build once.
116. Promote the same artifact.
117. Verify promotion identity.
118. Use immutable image digests.
119. Keep rollback artifacts.
120. Test rollback.
```

## 143. Rules 7

```text
121. Validate database compatibility.
122. Separate schema and application rollback.
123. Monitor post-deployment health.
124. Use error rate thresholds.
125. Monitor latency.
126. Monitor business health where possible.
127. Use progressive rollout when appropriate.
128. Preserve deployment history.
129. Keep incident timelines.
130. Perform root-cause analysis.
131. Track corrective actions.
132. Assign action owners.
133. Set action deadlines.
134. Test corrective fixes.
135. Update runbooks.
136. Improve monitoring after incidents.
137. Improve automation after incidents.
138. Improve security after incidents.
139. Improve reliability after incidents.
140. Share relevant lessons with engineering teams.
```

## 144. Rules 8

```text
141. Troubleshooting should be evidence-driven.
142. Troubleshooting should be reversible.
143. Troubleshooting should protect production.
144. Troubleshooting should protect secrets.
145. Troubleshooting should preserve artifacts.
146. Troubleshooting should preserve audit evidence.
147. Troubleshooting should distinguish symptom from cause.
148. Troubleshooting should reduce future recurrence.
149. Automate known diagnostic checks.
150. Keep diagnostic scripts version-controlled.
151. Test runbooks periodically.
152. Test repository recovery.
153. Test runner recovery.
154. Test credential recovery.
155. Test artifact recovery.
156. Test DR.
157. Test cold-cache builds.
158. Test release rollback.
159. Test security incident response.
160. Design production builds so failures are observable, diagnosable,
     recoverable and preventable.
```

# END OF 18-Build-Troubleshooting.md
