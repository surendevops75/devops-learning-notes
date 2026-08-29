# Build-Caching-and-Optimization

## 1. Purpose

Build performance directly affects developer productivity, CI cost, deployment frequency and release reliability.

Production optimization must preserve:

```text
Fast + Reproducible + Secure + Correct + Observable
```

A typical enterprise flow is:

```text
Git -> CI Runner -> Tool Setup -> Dependency Restore -> Cache -> Compile -> Test -> Security -> Package -> Publish
                                                                                                      |
                                                                                                      v
                                                                                             Artifact Repository
```

This file covers build-performance fundamentals, baselines, bottleneck analysis, dependency caching, Maven, npm, Python, Docker layer caching, remote caches, cache invalidation, cache security, parallelism, incremental builds, test optimization, monorepos, GitHub Actions, Jenkins, Kubernetes runners, repository performance, observability, capacity planning, troubleshooting, production architecture and interview preparation.

---

# PART I — BUILD PERFORMANCE FUNDAMENTALS

## 2. What Is Build Optimization?

Build optimization means reducing unnecessary:

```text
time
CPU
memory
network traffic
storage
runner minutes
```

while maintaining correctness, security, reproducibility and traceability.

## 3. Build Time Components

Example:

```text
Checkout       20s
Tool setup     10s
Dependencies   90s
Compile        60s
Tests          180s
Security       60s
Package        20s
Publish        30s
```

Do not optimize a 10-second stage while ignoring a 180-second stage.

## 4. First Rule

```text
Measure before optimizing.
```

---

# PART II — BASELINE AND BOTTLENECKS

## 5. Establish a Baseline

Record:

```text
average duration
p50
p90
p95
failure rate
queue time
runner startup
cache hit rate
dependency download time
test duration
```

## 6. Bottleneck Analysis

Common bottlenecks:

```text
dependency download
compilation
tests
Docker build
security scanning
artifact upload
runner startup
```

## 7. Build Timeline

Visualize:

```text
| checkout | install | compile | test | scan | publish |
```

Compare successful and failed runs.

## 8. Queue Time

Total developer wait time is:

```text
queue + startup + execution
```

A fast build is still slow for developers if runners wait ten minutes for capacity.

---

# PART III — CACHE FUNDAMENTALS

## 9. What Is a Build Cache?

A cache stores reusable build inputs or outputs.

```text
First Build -> Download/Build -> Cache
Next Build  -> Cache Hit     -> Reuse
```

## 10. Cache Benefits

```text
lower latency
lower network usage
lower CI cost
higher developer productivity
```

## 11. Cache Is Not Source of Truth

Authoritative sources remain:

```text
source repository
artifact repository
package registry
```

A cache should be disposable.

## 12. Cache Hit

```text
Requested data -> Cache -> FOUND -> Reuse
```

## 13. Cache Miss

```text
Requested data -> Cache -> NOT FOUND -> Download/Build -> Store
```

---

# PART IV — CACHE KEYS AND INVALIDATION

## 14. Cache Key

A cache key should represent the inputs that materially affect the cached data.

Typical inputs:

```text
OS
architecture
tool version
dependency lockfile hash
build configuration
```

## 15. Bad Key

```text
node-cache
```

This can mix unrelated dependency states.

## 16. Better Concept

```text
node-20-linux-package-lock-hash
```

The exact syntax depends on the CI platform.

## 17. Cache Invalidation

Invalidate when relevant inputs change:

```text
lockfile
compiler version
Node version
Python version
Maven configuration
OS
architecture
```

## 18. Classic Failure

```text
Old cache -> stale/incompatible dependency -> confusing build failure
```

Always have a clean-cache fallback.

---

# PART V — DEPENDENCY CACHING

## 19. Common Targets

```text
Maven ~/.m2/repository
npm package cache
pip package cache
Gradle caches
```

## 20. Dependency Cache Model

```text
CI -> Cache -> hit
          \-> miss -> Repository
```

## 21. Maven Cache

Maven commonly stores downloaded artifacts under:

```text
~/.m2/repository
```

Caching this location can significantly reduce dependency download time.

## 22. Maven Cache Inputs

Consider:

```text
OS
JDK
Maven version
pom.xml
dependency management
settings
```

## 23. Maven Cache Risks

```text
corrupt local repository
stale SNAPSHOT
incorrect mirror
cross-run contamination
```

## 24. Maven Repository vs CI Cache

```text
CI Cache       -> fast reusable acceleration
Artifactory    -> durable enterprise repository
```

Do not replace the artifact repository with an ephemeral cache.

## 25. SNAPSHOT Caching

Snapshots are mutable development versions. Aggressive caching can produce stale results. Use appropriate repository and Maven update policies.

## 26. npm Cache

npm maintains a package cache that can accelerate `npm ci`.

Good inputs include:

```text
OS
Node version
lockfile hash
package-manager version
```

## 27. node_modules Cache

Caching `node_modules` may be fast but risks:

```text
native module mismatch
OS mismatch
Node ABI mismatch
stale state
```

Prefer clean lockfile-driven installation unless measurements justify installed-state caching.

## 28. Python Cache

pip can cache downloaded distributions. Useful inputs include:

```text
OS
Python version
architecture
requirements/lock hash
package-manager version
```

Do not make correctness depend on a cached virtual environment.

---

# PART VI — DOCKER LAYER CACHING

## 29. Docker Layers

Docker builds can reuse unchanged layers.

```text
Layer 1
Layer 2
Layer 3
Layer 4
```

## 30. Poor Layer Ordering

```dockerfile
COPY . .
RUN npm ci
```

A source change may invalidate dependency installation.

## 31. Better Ordering

```dockerfile
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build
```

Stable dependency inputs are separated from frequently changing source.

## 32. Python Example

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

## 33. Maven Example

Separate stable POM/dependency inputs from application source when designing the Docker build.

## 34. BuildKit

BuildKit can provide efficient cache handling and parallel build execution. Do not persist secrets in build layers.

## 35. .dockerignore

Keep build context small:

```text
.git
node_modules
.venv
target
coverage
```

Choose entries according to the build.

---

# PART VII — REMOTE CACHE AND EPHEMERAL RUNNERS

## 36. Remote Cache

```text
Runner A -> Remote Cache <- Runner B
```

Useful for ephemeral runners, large teams, monorepos and expensive builds.

## 37. Ephemeral Runner Problem

```text
new runner -> empty cache -> download everything
```

## 38. Ephemeral Runner Solution

```text
new runner -> remote cache -> reuse
```

This combines isolation with performance.

## 39. Persistent Runner Tradeoff

Warm caches can be fast, but persistent state can cause:

```text
state contamination
security exposure
disk growth
stale dependencies
```

---

# PART VIII — CACHE SECURITY

## 40. Cache Poisoning

A malicious or incorrect workflow can potentially write unwanted data into a shared cache.

## 41. Controls

Use:

```text
trusted cache writers
scoped keys
branch trust boundaries
isolated caches
immutable artifacts
```

## 42. Untrusted Pull Requests

Do not allow untrusted PR workflows to poison caches consumed by privileged release workflows.

## 43. Incident Response

```text
disable/isolate cache
-> identify writers
-> audit workflow
-> rotate credentials if needed
-> rebuild clean
-> restore trusted cache
```

---

# PART IX — PARALLELISM

## 44. Sequential

```text
Build -> Test -> Security
```

## 45. Parallel Where Independent

```text
             +-> Unit Tests
Build -------+-> Lint
             +-> Security
```

## 46. Parallelism Limit

Too much parallelism can cause:

```text
CPU contention
memory pressure
network saturation
registry overload
runner exhaustion
```

## 47. Job Dependencies

Use dependencies so publishing waits for required gates:

```text
build
 |
 +-> test
 +-> security
 +-> quality
       |
       v
     publish
```

---

# PART X — TEST OPTIMIZATION

## 48. Tests Often Dominate

Example:

```text
Install  2m
Compile  1m
Tests    8m
Scan     2m
```

## 49. Test Categories

```text
unit
integration
contract
end-to-end
performance
```

## 50. Parallel Tests

Parallelize safe independent suites when resource capacity exists.

## 51. Flaky Tests

Flaky tests create:

```text
retries -> longer builds -> less confidence
```

Fix the root cause rather than hiding failures with unlimited retries.

## 52. Test Selection

Monorepos may use affected tests, but dependency analysis must be reliable.

## 53. Nightly Validation

A possible model is:

```text
PR       -> targeted validation
main     -> full validation
nightly  -> exhaustive validation
```

Use only when it matches application risk and release requirements.

---

# PART XI — INCREMENTAL BUILDS AND MONOREPOS

## 54. Incremental Build

Only rebuild changed components where safe.

Benefits:

```text
shorter builds
less CPU
less network
```

Risk:

```text
incorrect dependency analysis -> stale output
```

## 55. Monorepo Example

```text
repo
 |
 +-> service-a
 +-> service-b
 +-> service-c
 +-> shared-lib
```

## 56. Dependency Graph

```text
shared-lib -> service-a
shared-lib -> service-b
```

A shared library change may affect both services.

## 57. Maven Multi-Module

```text
parent
 |
 +-> module-a
 +-> module-b
 +-> module-c
```

Example:

```bash
./mvnw -pl module-c -am verify
```

Use selective builds only with correct dependency analysis.

## 58. npm Workspaces

```text
root
 |
 +-> package-a
 +-> package-b
 +-> package-c
```

Use workspace-aware dependency and test commands.

## 59. Python Monorepos

Use the selected Python tool's supported project/workspace model. Do not bypass package relationships merely for speed.

---

# PART XII — TOOLCHAIN OPTIMIZATION

## 60. Standardize Versions

Control:

```text
JDK
Maven
Node
npm
Python
pip
build backend
Docker/BuildKit
```

Different versions can materially affect performance and output.

## 61. Compiler Optimization

Potential techniques:

```text
incremental compilation
parallel compilation
appropriate compiler flags
```

Measure CPU and memory impact.

## 62. Node Build Optimization

Common bottlenecks:

```text
bundling
minification
TypeScript
source maps
large dependency graph
```

Do not disable source maps blindly if operations need them.

## 63. Python Optimization

Common bottlenecks:

```text
dependency compilation
tests
package build
Docker build
```

Prefer compatible wheels where practical.

## 64. Maven Optimization

Potential bottlenecks:

```text
dependency resolution
compiler
tests
plugins
integration tests
```

Profile before changing build behavior.

---

# PART XIII — RUNNER SIZING AND COST

## 65. CPU

More CPU can help compilation, bundling and parallel tests.

## 66. Memory

More memory can help large Java/TypeScript builds and test suites.

## 67. Network

Faster network helps dependency downloads, artifact uploads and image pulls.

## 68. Right-Sizing

Compare cost and duration. For example:

```text
10 min x 8 CPU
vs
6 min x 16 CPU
```

The cheaper choice depends on CI pricing and workload utilization.

## 69. Capacity Planning

Track:

```text
average concurrency
peak concurrency
runner startup time
CPU demand
memory demand
registry throughput
```

## 70. Autoscaling

Autoscaling must consider downstream limits. Creating more runners does not help if the artifact repository becomes saturated.

---

# PART XIV — CI CACHE DESIGN

## 71. Cache Scope

Possible scopes:

```text
repository
branch
organization
runner
workflow
```

Use the smallest safe scope that provides useful reuse.

## 72. Shared Cache Tradeoff

Sharing improves reuse but can increase:

```text
collision risk
security risk
invalidation complexity
```

## 73. Exact vs Fallback Keys

Prefer exact matches. Older fallback data should be treated as potentially stale.

---

# PART XV — ARTIFACTS AND REPOSITORIES

## 74. Dependency Cache vs Artifact

Dependency cache:

```text
downloaded inputs
```

Artifact:

```text
validated build output
```

## 75. Enterprise Artifact Repository

Use a durable repository such as Artifactory, Nexus or an appropriate cloud artifact registry.

## 76. Build Once

```text
Build -> Validated Artifact -> DEV -> STAGE -> PROD
```

Do not rebuild separately for every environment.

## 77. Artifact Size

Large artifacts can dominate upload time. Measure:

```text
size
compression
number of files
network
retention
```

---

# PART XVI — SECURITY SCANNING PERFORMANCE

## 78. Scan Optimization

Possible optimizations:

```text
incremental scans
cached vulnerability data
parallel independent scans
```

Never weaken required security coverage merely for speed.

## 79. Quality Gates

Independent checks can run in parallel:

```text
Build -> lint
      -> SAST
      -> dependency scan
```

Publishing waits for required gates.

## 80. Fail Fast

If compilation fails, expensive downstream stages may be unnecessary. Use fail-fast logic only when required diagnostics and checks remain available.

---

# PART XVII — GITHUB ACTIONS

## 81. Node Cache

Example:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: npm
```

## 82. Maven Cache

Example:

```yaml
- uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '21'
    cache: maven
```

## 83. Python Cache

Example:

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.12'
    cache: pip
```

Use action versions and permissions according to organizational security policy.

## 84. Job Design

```text
build
 |
 +-> test
 +-> security
 +-> lint
       |
       v
    publish
```

---

# PART XVIII — JENKINS

## 85. Jenkins Cache Options

```text
persistent agents
shared storage
repository mirrors
external caches
Docker layer caches
```

## 86. Persistent Agent Risk

Warm local caches are convenient, but state can become stale or contaminated. Clean and monitor them.

## 87. Jenkins + Ephemeral Kubernetes

```text
Jenkins -> Kubernetes -> Build Pod -> Cache/Repository -> Pod deleted
```

External caches can retain performance benefits.

---

# PART XIX — KUBERNETES CI RUNNERS

## 88. Ephemeral Build Pods

```text
Build Pod
 |
 +-> source
 +-> tools
 +-> cache
 |
 v
Build
 |
 v
Pod deleted
```

## 89. Persistent Volume Cache

```text
Build Pod -> PVC -> Dependency Cache
```

This can be useful but requires lifecycle and isolation controls.

## 90. Remote Cache

```text
Build Pod -> Remote Cache -> Object/Artifact Storage
```

This preserves ephemeral execution while retaining reusable cache data.

---

# PART XX — REGISTRY PERFORMANCE

## 91. Enterprise Mirror

```text
CI -> Corporate Repository -> Approved Upstreams
```

Benefits:

```text
lower latency
availability
central governance
reduced internet dependence
```

## 92. Artifactory Performance

Consider:

```text
geographic placement
remote repository caching
network latency
storage performance
repository design
```

## 93. Virtual Repositories

A virtual endpoint can simplify developer and CI configuration while providing controlled routing.

## 94. Network Diagnostics

Measure:

```text
DNS
TLS
connection setup
download
upload
registry response time
```

---

# PART XXI — BUILD OBSERVABILITY

## 95. Metrics

Track:

```text
total duration
queue time
CPU
memory
network
cache hits
cache misses
dependency download time
test duration
artifact upload
```

## 96. Build SLO

Example:

```text
95% of PR builds < 10 minutes
```

Use percentiles because averages can hide poor experiences.

## 97. Useful Alerts

```text
build p95 increased
cache hit rate dropped
registry latency increased
runner capacity exhausted
failure rate increased
```

---

# PART XXII — TROUBLESHOOTING

## 98. Build Suddenly Becomes Slow

Check:

```text
cache hit rate
dependency downloads
tests
CPU
memory
network
registry
runner queue
```

## 99. Maven Is Slow

Compare warm and cold cache runs. Inspect dependency resolution, plugins and test phases.

## 100. npm Is Slow

Check:

```text
registry
cache
lockfile
native modules
Node version
```

## 101. Python Is Slow

Check:

```text
index
wheel availability
cache
native compilation
Python version
```

## 102. Docker Is Slow

Check:

```text
context size
layer invalidation
base image pulls
dependency installation
build cache
registry
```

## 103. Cache Corruption

Symptoms:

```text
random compile failures
checksum errors
missing files
inconsistent builds
```

Response:

```text
cold-cache reproduction -> identify key -> invalidate affected cache -> rebuild
```

## 104. Local Fast, CI Slow

Compare:

```text
cache availability
runner resources
registry latency
network
tool versions
```

## 105. Optimization Introduced Bugs

The optimization is unsuccessful. Compare warm/cold results, restore missing validation and fix dependency analysis.

---

# PART XXIII — PRODUCTION ARCHITECTURE

## 106. Reference

```text
                              Git
                               |
                               v
                              CI
                               |
                    +----------+----------+
                    |                     |
                    v                     v
             Ephemeral Runner        Build Metadata
                    |
        +-----------+-----------+
        |           |           |
        v           v           v
      Maven        npm         Python
        |           |           |
        v           v           v
      Cache       Cache       Cache
        |           |           |
        +-----------+-----------+
                    |
                    v
             Enterprise Registry
                    |
             Validated Artifact
                    |
                    v
              Container Build
                    |
                    v
             Container Registry
                    |
                    v
               Promotion
                    |
                    v
               Deployment
```

## 107. Enterprise Cache Architecture

```text
Developer/CI
    |
    v
Corporate Repository
    |
+---+---+
|       |
v       v
Local   Remote Cache
Packages Upstreams
    |
    v
CI Cache
    |
    v
Build
```

The exact architecture depends on security, network and platform requirements.

---

# PART XXIV — PRODUCTION CHECKLIST

## 108. Measurement

```text
[ ] baseline exists
[ ] p50 measured
[ ] p95 measured
[ ] queue time measured
[ ] bottlenecks identified
[ ] runner metrics available
```

## 109. Cache

```text
[ ] dependency cache
[ ] correct cache key
[ ] invalidation strategy
[ ] cold-cache fallback
[ ] cache security
```

## 110. Build

```text
[ ] parallelism
[ ] incremental strategy where safe
[ ] test optimization
[ ] artifact optimization
[ ] Docker layer optimization
```

## 111. Infrastructure

```text
[ ] runner capacity
[ ] CPU sizing
[ ] memory sizing
[ ] network capacity
[ ] registry capacity
```

## 112. Security

```text
[ ] cache trust boundaries
[ ] PR isolation
[ ] secret protection
[ ] artifact integrity
[ ] dependency security
```

---

# PART XXV — INTERVIEW PREPARATION

## 113. How Do You Optimize a Slow CI Build?

Answer:

```text
I establish a baseline and break the build into measurable stages. I identify whether dependency resolution, compilation, tests, security scanning, container builds, runner startup or artifact transfer is the bottleneck. I then optimize the largest contributor using caching, parallelism, runner sizing or incremental builds while validating that cache hits and misses produce the same correct result.
```

## 114. What Do You Cache?

Answer:

```text
I commonly cache Maven, npm and pip dependency data and use Docker or build-system caches where appropriate. I treat caches as disposable acceleration rather than authoritative artifact storage.
```

## 115. What Makes a Good Cache Key?

Answer:

```text
It includes inputs that materially affect the cached data, such as operating system, architecture, tool version and dependency lock or configuration state.
```

## 116. Why Not Cache node_modules or Virtual Environments Everywhere?

Answer:

```text
They can contain platform-specific binaries and stale state. Clean lockfile-driven installation is more predictable. I use installed-state caching only when measured and compatible.
```

## 117. How Do You Optimize Docker Builds?

Answer:

```text
I minimize the build context, use .dockerignore, order stable dependency layers before frequently changing source layers and use an appropriate remote build cache for ephemeral runners.
```

## 118. How Do You Optimize Monorepo Builds?

Answer:

```text
I model the dependency graph, identify affected packages, reuse safe caches, parallelize independent work and run the minimum safe validation set. I never skip required dependency-impact testing merely to reduce build time.
```

## 119. How Do You Handle Cache Poisoning?

Answer:

```text
I isolate trust boundaries, restrict cache writers, use scoped keys, prevent untrusted PR workflows from writing privileged release caches and invalidate affected caches when compromise is suspected.
```

## 120. How Do You Measure Build Performance?

Answer:

```text
I track total duration plus queue time, runner startup, dependency installation, cache hit rate, compile time, test time, security scan, artifact transfer and failure rate. I use percentiles such as p95 rather than average alone.
```

## 121. Is More CPU Always Better?

Answer:

```text
No. A build can be network-bound, serial, memory-bound or waiting on tests or external services. I measure first and then right-size resources.
```

---

# PART XXVI — SENIOR-LEVEL SCENARIOS

## 122. Build Time Doubled Overnight

Answer:

```text
I compare the last known-good build with the current run stage by stage. I check dependency changes, cache hit rate, runner image, toolchain versions, registry latency, test duration and security scan duration. I focus first on the changed bottleneck.
```

## 123. Cache Hit Rate Falls From 90% to 10%

Answer:

```text
I inspect recent changes to cache keys, lockfiles, tool versions, operating systems and workflow configuration. I correct unnecessary invalidation and verify that cache misses correctly fall back to the authoritative repository.
```

## 124. CI Execution Is Fast but Developers Wait

Answer:

```text
I inspect queue time and runner provisioning. The problem may be capacity rather than build execution. I review peak concurrency, autoscaling and available runners.
```

## 125. Docker Build Is Slow Only in CI

Answer:

```text
I compare local and CI layer-cache availability, build context size, base-image pulls, registry latency and remote cache configuration. Ephemeral runners often require an external cache.
```

## 126. Shared Cache Is Suspected of Being Poisoned

Answer:

```text
I isolate the cache, identify writers, audit workflow changes, inspect affected artifacts, rotate credentials if required, rebuild from authoritative sources and restore only trusted cache data.
```

## 127. Optimization Reduced Time but Introduced Bugs

Answer:

```text
I treat it as a failed optimization. I compare cold and warm outputs, inspect incremental dependency analysis and test selection, restore missing validation and redesign the optimization around correctness.
```

## 128. 500 Repositories Use One CI Template

Answer:

```text
I standardize common caching, runner, repository and observability patterns through reusable workflows while allowing controlled application-specific configuration. I roll out global changes gradually and monitor performance.
```

## 129. Artifact Repository Becomes the Bottleneck

Answer:

```text
I measure repository latency and throughput, then evaluate remote caching, virtual repositories, network placement, repository capacity and CI concurrency. I avoid uncontrolled mirrors that bypass governance.
```

## 130. Tests Take 80% of the Build

Answer:

```text
I profile the test suite, identify slow and flaky tests, parallelize safe suites, use targeted testing where dependency analysis is reliable and remove redundant execution without skipping required coverage.
```

## 131. Warm Cache Is Fast but Reproducibility Is Unclear

Answer:

```text
I perform warm-cache and cold-cache builds and compare expected outputs. The build must remain correct without the cache. Any difference indicates hidden state or uncontrolled inputs that must be fixed.
```

---

# PART XXVII — GOLDEN RULES

## 132. Rules

```text
1. Measure before optimizing.
2. Establish a build-performance baseline.
3. Track p50 and p95 duration.
4. Track queue time separately from execution time.
5. Identify the actual bottleneck.
6. Optimize the largest contributor first.
7. Treat caches as disposable acceleration.
8. Never treat a CI cache as the source of truth.
9. Maintain a cold-cache fallback.
10. Warm and cold builds must remain correct.
11. Design cache keys around actual inputs.
12. Include OS where platform differences matter.
13. Include architecture where binary artifacts differ.
14. Include tool versions where compatibility matters.
15. Include lock/constraint state where applicable.
16. Invalidate caches when relevant inputs change.
17. Avoid one generic cache key for unrelated builds.
18. Avoid incompatible dependency-cache sharing.
19. Protect cache trust boundaries.
20. Never let untrusted PRs poison privileged release caches.
21. Restrict cache writers.
22. Maintain cache-corruption recovery.
23. Cache Maven dependencies when appropriate.
24. Cache npm package data when appropriate.
25. Cache pip package data when appropriate.
26. Avoid relying on cached virtual environments.
27. Avoid blindly caching node_modules.
28. Use Docker layer caching appropriately.
29. Keep stable Docker layers before changing source layers.
30. Keep Docker build context small.
31. Use .dockerignore.
32. Do not copy unnecessary files into images.
33. Use remote caches for ephemeral runners where justified.
34. Do not replace enterprise artifact repositories with CI caches.
35. Use durable artifact repositories for release artifacts.
36. Use virtual repositories where appropriate.
37. Control public upstream access.
38. Monitor repository performance.
39. Monitor DNS and network latency.
40. Monitor artifact upload time.
41. Monitor dependency download time.
42. Monitor cache hit rate.
43. Monitor cache miss rate.
44. Monitor runner CPU.
45. Monitor runner memory.
46. Monitor runner queue time.
47. Monitor runner startup time.
48. Right-size runners using measurements.
49. More CPU is not always better.
50. More memory is not always better.
51. Network can be the real bottleneck.
52. Registry capacity can limit CI throughput.
53. Avoid uncontrolled CI concurrency.
54. Parallelize independent stages.
55. Do not parallelize unsafe shared-state work.
56. Avoid runner exhaustion.
57. Avoid registry overload.
58. Optimize tests because they often dominate builds.
59. Profile test duration.
60. Parallelize safe test suites.
61. Fix flaky tests.
62. Do not hide flaky tests with unlimited retries.
63. Use targeted tests only with reliable dependency analysis.
64. Use incremental builds only when correctness is preserved.
65. Understand monorepo dependency graphs.
66. Build affected packages only when safely supported.
67. Optimize Maven reactor builds carefully.
68. Optimize npm workspace builds carefully.
69. Optimize Python multi-package builds carefully.
70. Standardize tool versions.
71. Control JDK versions.
72. Control Maven versions.
73. Control Node versions.
74. Control npm versions.
75. Control Python versions.
76. Control package-manager versions.
77. Control build backend versions.
78. Do not let local developer state influence CI.
79. Use clean build environments.
80. Prefer ephemeral runners when security requires them.
81. Externalize caches for ephemeral runners.
82. Persistent runners provide warm caches but create state risks.
83. Clean persistent runner state.
84. Monitor persistent cache disk usage.
85. Understand Maven SNAPSHOT cache behavior.
86. Understand npm cache behavior.
87. Understand Python package cache behavior.
88. Prefer compatible Python wheels where practical.
89. Standardize native build toolchains.
90. Understand native-module compatibility.
91. Separate build-time and runtime dependencies.
92. Keep runtime container images small.
93. Do not weaken security controls for speed.
94. Parallelize independent security checks when safe.
95. Preserve required quality gates.
96. Fail fast when downstream work is impossible.
97. Do not fail fast in a way that hides required diagnostics.
98. Keep publishing behind required gates.
99. Build once and promote immutable artifacts.
100. Do not rebuild merely for another environment.
101. Optimize artifact sizes.
102. Do not package unnecessary files.
103. Preserve logs and reports needed for diagnosis.
104. Use build profiling tools.
105. Compare cold and warm builds.
106. Compare local and CI builds.
107. Compare successful and failed builds.
108. Investigate changes correlated with regressions.
109. Roll out global optimization changes gradually.
110. Monitor performance after optimization.
111. Define build SLOs where useful.
112. Use p95 for user-experience targets.
113. Treat cache poisoning as a security incident.
114. Rotate credentials when compromise could expose them.
115. Rebuild from authoritative sources after cache compromise.
116. Validate optimized builds for correctness.
117. Never accept a faster but incorrect build.
118. Document cache ownership.
119. Document cache invalidation.
120. Document runner sizing.
121. Document repository dependencies.
122. Document rollback procedures.
123. Test cache recovery.
124. Test cold-cache recovery.
125. Test runner failure recovery.
126. Test registry outage behavior.
127. Test dependency mirror failure behavior.
128. Avoid making builds depend on a single cache.
129. Ensure authoritative repositories remain available.
130. Use redundancy for critical artifact repositories.
131. Keep dependency downloads auditable.
132. Preserve build provenance.
133. Track source-to-artifact relationships.
134. Track workflow-to-artifact relationships.
135. Track container build inputs.
136. Monitor build cost as well as speed.
137. Optimize cost and latency together.
138. Consider developer experience when setting targets.
139. Do not optimize only for average builds.
140. Optimize peak concurrency behavior.
141. Account for autoscaling delays.
142. Account for remote-cache latency.
143. Account for cache-storage cost.
144. Account for repository-storage cost.
145. Account for runner cost.
146. Reassess optimizations when workload changes.
147. Validate cache-hit and cache-miss correctness.
148. Preserve required security and quality coverage.
149. Keep release artifacts immutable.
150. Validate exact CI, runner, cache, registry, build-tool, dependency, container and deployment behavior for the production architecture actually used.
```
---