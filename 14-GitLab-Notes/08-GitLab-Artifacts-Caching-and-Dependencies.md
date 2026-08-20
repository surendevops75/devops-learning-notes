# GitLab Artifacts, Caching and Dependencies

> Production-oriented guide to GitLab CI/CD artifacts, cache, job dependencies, `needs`, reports, package flow, retention, security, performance, troubleshooting, and practical DevOps patterns with Docker, Terraform, AWS/ECR, Kubernetes/EKS, and GitOps.

---

## 1. Why Artifacts Matter

CI jobs often produce files required by later jobs.

Examples:

```text
Compiled JAR
Test reports
Coverage reports
Terraform plan
Security reports
Generated manifests
Helm packages
```

Artifacts provide a controlled way to preserve job outputs.

---

## 2. Basic Artifact Flow

```text
Build
 ↓
artifact
 ↓
Test/Package/Deploy
```

Example:

```yaml
build:
  script:
    - mvn package
  artifacts:
    paths:
      - target/app.jar
```

---

## 3. Artifact vs Cache

The most important distinction:

```text
Artifact
→ Job output/evidence

Cache
→ Performance optimization
```

Do not use cache as the only mechanism for delivering production artifacts.

---

## 4. Artifact Lifecycle

Typical:

```text
Job executes
 ↓
Files generated
 ↓
Runner uploads artifacts
 ↓
GitLab stores artifacts
 ↓
Downstream job downloads
 ↓
Artifact expires according to policy
```

---

## 5. Artifact Paths

Example:

```yaml
artifacts:
  paths:
    - target/app.jar
```

Be precise.

Avoid:

```yaml
paths:
  - .
```

because it can upload unnecessary or sensitive files.

---

## 6. Multiple Artifact Paths

Example:

```yaml
artifacts:
  paths:
    - target/app.jar
    - reports/
    - manifests/
```

Only include files that downstream jobs or humans actually need.

---

## 7. Artifact Name

Artifacts can be given a useful name.

Concept:

```yaml
artifacts:
  name: "build-$CI_COMMIT_SHORT_SHA"
```

This improves traceability.

---

## 8. Commit-Based Artifact Identity

A strong naming pattern:

```text
application-<commit-sha>
```

Example:

```text
user-service-a81c92f
```

This connects:

```text
Git commit
 ↓
Build
 ↓
Artifact
```

---

## 9. Artifact Expiration

Temporary artifacts should expire.

Concept:

```yaml
artifacts:
  expire_in: 7 days
```

Retention should reflect:

- debugging needs
- audit requirements
- compliance
- storage cost

---

## 10. Permanent Release Artifacts

Release artifacts may require longer retention.

Examples:

```text
release binary
SBOM
security evidence
deployment manifest
```

Use organizational retention policy rather than keeping everything forever.

---

## 11. Artifact Storage Cost

Artifacts consume storage.

A pipeline that uploads:

```text
500 MB
```

on every commit can create significant storage growth.

Optimize:

```text
artifact size
+
retention
+
frequency
```

---

## 12. Artifact Download

Downstream jobs may retrieve artifacts from upstream jobs.

Concept:

```text
build
 ↓
artifact
 ↓
test
```

Use explicit dependencies when the pipeline becomes complex.

---

## 13. `needs` and Artifacts

`needs` can define which upstream jobs a job depends on.

Concept:

```text
build-a ──→ test-a
build-b ──→ test-b
```

This can avoid unnecessary stage-wide waiting.

---

## 14. Stage Dependencies

Traditional model:

```text
build stage
    ↓
test stage
    ↓
deploy stage
```

Every job in the next stage may wait for the required previous stage behavior.

This is simple but can be slower.

---

## 15. DAG Pipelines

A dependency graph can look like:

```text
build-user ──→ test-user ──→ package-user
build-cart ──→ test-cart ──→ package-cart
build-order ─→ test-order
```

This is useful for microservices and monorepos.

---

## 16. Why `needs` Improves Speed

Suppose:

```text
Build A = 2 min
Build B = 10 min
Test A = 2 min
```

With stage-only sequencing:

```text
Build A + Build B
      ↓
Test A
```

Test A may wait for Build B unnecessarily.

With explicit dependency:

```text
Build A → Test A
Build B
```

Test A can start earlier.

---

## 17. Dependency Graph Design

Create dependencies based on actual data flow.

Bad:

```text
Every job needs every job
```

Good:

```text
Only jobs requiring the output depend on it.
```

This keeps pipelines fast and understandable.

---

## 18. Artifact Dependencies

Example:

```text
compile
  ↓
app.jar
  ↓
package-image
```

The image job consumes the compiled application.

---

## 19. Artifact and Docker Image

A common pipeline:

```text
Compile
 ↓
JAR artifact
 ↓
Docker build
 ↓
Container image
 ↓
ECR
```

The artifact becomes input to the image build.

---

## 20. Artifact and ECR

After building an image:

```text
Docker image
 ↓
ECR
 ↓
Image digest
```

The container registry becomes the durable artifact store for container deployment.

GitLab artifacts can hold supporting evidence, but the image itself can live in ECR.

---

## 21. Immutable Artifact Principle

Build once:

```text
Commit
 ↓
Build
 ↓
Artifact
 ↓
Test
 ↓
Scan
 ↓
Release
```

Do not rebuild different binaries for each environment.

---

## 22. Promotion Model

Recommended:

```text
Build once
 ↓
Dev
 ↓
Staging
 ↓
Production
```

The artifact remains unchanged.

Only environment configuration changes.

---

## 23. Artifact Integrity

A production artifact should be traceable to:

```text
Git commit
+
Pipeline
+
Job
+
Build environment
```

For container images, record:

```text
Image tag
+
Image digest
```

---

## 24. Artifact Digest

Tags can move:

```text
app:latest
```

A digest is immutable:

```text
sha256:<digest>
```

Production deployment should preferably identify the exact artifact digest.

---

## 25. Artifact Provenance

Useful metadata:

```text
Repository
Commit SHA
Pipeline ID
Build timestamp
Builder version
Base image
Dependency versions
Security scan results
```

This helps incident investigation.

---

## 26. Test Reports

CI can produce test reports.

Example concept:

```text
tests/
└── junit.xml
```

The exact GitLab report configuration depends on the test framework and report format.

---

## 27. JUnit Reports

Common flow:

```text
Test
 ↓
JUnit XML
 ↓
GitLab test reporting
```

This allows failures to be visible without reading raw logs.

---

## 28. Coverage Reports

Coverage artifacts can include:

```text
coverage.xml
coverage.html
```

Use coverage as a quality signal, not as the only measure of test quality.

---

## 29. Security Reports

Security jobs may generate reports for:

- SAST
- dependency scanning
- secret detection
- container scanning
- DAST

The report format must match GitLab's supported report integration.

---

## 30. Terraform Plan Artifact

A Terraform job may generate:

```text
tfplan
```

or human-readable plan output.

Treat Terraform plans as potentially sensitive.

Restrict access and retention.

---

## 31. Terraform State Is Not an Artifact

Do not casually upload:

```text
terraform.tfstate
```

as a CI artifact.

State may contain:

- infrastructure metadata
- sensitive values
- resource details

Use a secure remote backend.

---

## 32. Helm Package Artifact

A Helm pipeline can produce:

```text
app-1.2.3.tgz
```

Then:

```text
Validation
 ↓
Package
 ↓
Registry
```

Depending on architecture, the chart can be published to a suitable OCI registry.

---

## 33. Kubernetes Manifest Artifact

CI may generate:

```text
rendered-manifests/
```

Useful for:

- review
- debugging
- audit
- deployment evidence

But generated manifests may contain sensitive values if templating is unsafe.

---

## 34. Artifact Security Review

Before publishing an artifact, ask:

```text
Does it contain credentials?
Tokens?
Private keys?
Secrets?
Internal URLs?
Sensitive Terraform data?
```

Do not assume an artifact is harmless because it is generated by CI.

---

## 35. Artifact Access Control

Production artifacts may contain sensitive information.

Restrict:

```text
Who can view?
Who can download?
How long is it retained?
```

Apply least privilege.

---

## 36. Artifact Exposure Through Logs

Artifacts are not the only output channel.

A job can expose secrets through:

```text
logs
reports
test output
debug files
```

Review the entire job output surface.

---

## 37. Artifact Upload Failure

Symptoms:

```text
Job commands succeed
 ↓
Artifact upload fails
 ↓
Job may be marked failed
```

Check:

- path exists
- permissions
- storage
- network
- artifact size
- GitLab availability

---

## 38. Artifact Path Does Not Exist

Example:

```yaml
paths:
  - target/app.jar
```

but build actually creates:

```text
build/app.jar
```

The artifact will not be available as expected.

Verify the filesystem before upload.

---

## 39. Artifact Permission Problem

The Runner process may not be able to read a generated file.

Check:

```bash
ls -l target/
```

and ownership/permissions.

Avoid broad permission changes such as:

```bash
chmod -R 777 .
```

---

## 40. Artifact Too Large

Large artifacts increase:

```text
upload time
storage
download time
network usage
```

Consider:

- compressing useful files
- excluding unnecessary files
- storing binaries in a package/container registry
- reducing retention

---

## 41. Artifact Retention Strategy

Example:

```text
Feature branch
 → short retention

Main branch
 → moderate retention

Release
 → long retention
```

Use actual organizational requirements to determine durations.

---

## 42. Cache Purpose

Cache speeds repeated work.

Typical cache targets:

```text
Maven dependencies
npm cache
pip cache
Terraform plugin cache
Docker build cache
```

Cache should never be treated as authoritative application state.

---

## 43. Cache Example — Python

Concept:

```yaml
cache:
  paths:
    - .cache/pip/
```

This can reduce repeated dependency downloads.

---

## 44. Cache Example — Maven

Typical cache target:

```text
.m2/repository
```

This can significantly reduce build time.

---

## 45. Cache Example — Node.js

Possible cache:

```text
.npm/
```

or package-manager-specific caches.

Do not blindly cache:

```text
node_modules
```

unless the dependency and platform strategy supports it safely.

---

## 46. Cache Example — Terraform

Terraform provider/plugin caches can reduce initialization time.

Use a controlled cache strategy rather than sharing arbitrary `.terraform` state across incompatible jobs.

---

## 47. Cache Key

A cache key determines which jobs share cache content.

Good cache keys should account for:

```text
project
dependency state
platform
trust boundary
```

---

## 48. Dependency-Lock-Based Cache

A strong pattern is to invalidate dependency caches when lock files change.

Examples:

```text
package-lock.json
requirements.lock
poetry.lock
pom dependency changes
```

Concept:

```text
Lock file changes
 ↓
New cache key
 ↓
Fresh dependencies
```

---

## 49. Branch Cache

A branch-aware cache can reduce interference.

Concept:

```text
feature-a → cache-a
feature-b → cache-b
main      → cache-main
```

But too many unique keys can reduce cache hit rate.

Balance isolation and reuse.

---

## 50. Protected vs Unprotected Cache

Do not allow untrusted jobs to poison a cache used by trusted jobs.

Example attack:

```text
MR
 ↓
writes malicious cache
 ↓
main pipeline restores it
```

Separate trust boundaries.

---

## 51. Cache Poisoning

Cache poisoning means an attacker modifies cached content so another job consumes it.

Potential consequences:

```text
malicious dependency
malicious binary
modified build tool
```

Treat shared caches as an attack surface.

---

## 52. Cache vs Dependency Artifact

If a downstream job requires an exact file from a specific build:

```text
Use artifact
```

If a job wants a reusable speed optimization:

```text
Use cache
```

This distinction is critical.

---

## 53. Cache Miss

A cache miss should usually mean:

```text
Job becomes slower
```

not:

```text
Job becomes incorrect
```

If correctness depends on cache presence, the design is likely wrong.

---

## 54. Cache Restore Failure

Troubleshoot:

```text
Cache key
 ↓
Runner configuration
 ↓
Cache backend
 ↓
Object storage
 ↓
Permissions
 ↓
Network
```

Do not confuse a cache miss with a job failure.

---

## 55. Distributed Cache

With multiple Runners:

```text
Runner A
Runner B
Runner C
```

local disk caches may not be shared.

A distributed cache backend can provide consistent access where configured.

---

## 56. Why Distributed Cache Helps

Without shared cache:

```text
Job 1 → Runner A → download dependencies
Job 2 → Runner B → download again
```

With shared cache:

```text
Job 1 → shared cache
Job 2 → shared cache
```

This is especially useful with autoscaling Runners.

---

## 57. Cache on Ephemeral Kubernetes Runners

An ephemeral Pod may disappear after the job.

Therefore:

```text
Local cache
 ↓
Destroyed
```

Use remote/distributed caching when the performance benefit justifies it.

---

## 58. S3-Backed Cache Concept

In AWS environments:

```text
Runner
 ↓
Cache
 ↓
S3
```

This can provide shared cache storage for Runner fleets.

Secure the bucket with:

- least-privilege IAM
- encryption
- lifecycle policies
- restricted access

---

## 59. Cache Lifecycle

Cache storage should have lifecycle management.

Example:

```text
Old cache
 ↓
Expiration
 ↓
Deletion
```

Do not allow cache storage to grow without bounds.

---

## 60. Cache Storage Security

Protect cache storage from:

- unauthorized read
- unauthorized write
- public access
- cross-environment contamination

Use separate prefixes/buckets according to trust requirements.

---

## 61. Dependency Definition

A dependency is something a job needs to function.

Examples:

```text
Build → compiler/dependencies
Test → built application
Docker → JAR artifact
Deploy → image digest
```

Make dependency relationships explicit.

---

## 62. Dependency Graph

Example:

```text
source
  │
  ├── unit-test
  │
  └── build
       │
       ▼
   docker-build
       │
       ▼
   image-scan
       │
       ▼
      push
       │
       ▼
   gitops-update
```

This is a practical production pipeline graph.

---

## 63. Job Dependency vs Application Dependency

Do not confuse:

```text
CI job dependency
```

with:

```text
Application runtime dependency
```

Example:

```text
CI:
docker-build depends on build artifact

Runtime:
orders depends on payment service
```

These are different concepts.

---

## 64. `needs` Design

Use:

```text
needs
```

when a job requires a specific upstream job and should not wait for unrelated jobs.

Avoid building a dependency graph that is unnecessarily complex.

---

## 65. Dependency Failure Propagation

Example:

```text
Build
 ↓ FAIL
Test
 ↓ SKIPPED
Deploy
 ↓ SKIPPED
```

This is desirable for mandatory dependencies.

Do not allow deployment when the required artifact was not successfully validated.

---

## 66. Optional Dependencies

Some workflows have optional jobs.

Example:

```text
Documentation generation
```

may not be required for deployment.

Do not make every optional task a hard deployment dependency.

---

## 67. Parallel Dependencies

Microservices can build independently:

```text
User ──→ test ──→ image
Cart ──→ test ──→ image
Order ─→ test ──→ image
```

This can significantly reduce pipeline duration.

---

## 68. Monorepo Dependency Graph

Example:

```text
services/
├── user/
├── cart/
├── order/
└── payment/
```

A change in:

```text
services/user/
```

should not necessarily rebuild every service.

Use:

```text
rules:changes
```

and dependency-aware pipeline design.

---

## 69. Shared Library Changes

Suppose:

```text
shared/
```

changes.

Then:

```text
shared
 ↓
User
Cart
Order
Payment
```

may all need rebuilding.

This is where dependency mapping becomes important.

---

## 70. Build Once, Test Multiple

Example:

```text
Build artifact
   ↓
Unit tests
   ↓
Security
   ↓
Integration tests
```

Reuse the exact artifact instead of rebuilding.

---

## 71. Why Rebuilding Is Dangerous

If you build twice:

```text
Build A
 ↓
Test

Build B
 ↓
Deploy
```

A and B may differ due to:

- dependency updates
- base image changes
- package repository changes
- timestamps
- tool versions

Build once whenever practical.

---

## 72. Immutable Container Promotion

Recommended:

```text
Build image
 ↓
Scan image
 ↓
Push ECR
 ↓
Record digest
 ↓
Promote digest
```

Do not rebuild for staging and production.

---

## 73. Artifact Registry vs GitLab Artifact

For large production binaries:

```text
Package/Container Registry
```

is often better than CI artifacts.

Examples:

```text
ECR
JFrog Artifactory
GitLab Package Registry
```

Use the appropriate durable artifact store.

---

## 74. Artifact Naming

Use predictable naming:

```text
<application>-<version>
```

or:

```text
<application>-<commit-sha>
```

For container images:

```text
<repository>:<commit>
```

plus digest tracking.

---

## 75. Artifact Versioning

Avoid ambiguous:

```text
latest
```

as the only production identity.

Prefer:

```text
1.4.2
```

or:

```text
commit SHA
```

and record the immutable digest.

---

## 76. Release Artifact

A release artifact should be associated with:

```text
Version
Commit
Pipeline
Security evidence
Build metadata
```

This makes rollback easier.

---

## 77. Artifact Promotion

Promotion should not mutate the artifact.

Example:

```text
ECR image digest
        │
        ├── Dev
        ├── Staging
        └── Production
```

Only deployment configuration changes.

---

## 78. Artifact Retention and Compliance

Some organizations require:

```text
Release evidence
Security reports
Deployment records
```

to be retained for a specific period.

Follow organizational policy.

Do not choose retention only based on convenience.

---

## 79. Cache and Compliance

Caches should generally not be treated as compliance evidence.

Caches can be:

```text
replaced
expired
deleted
reused
```

Use durable artifacts or approved evidence stores for compliance.

---

## 80. Artifact Encryption

Sensitive artifact storage should use encryption at rest where supported.

Also protect:

```text
transport
access
retention
credentials
```

Encryption does not replace access control.

---

## 81. Artifact Access Review

Ask:

```text
Who can download?
Can external contributors download?
Can production artifacts be accessed by development projects?
How long are artifacts retained?
```

Use least privilege.

---

## 82. Artifact Download Performance

Large artifacts can slow pipelines.

Optimize:

```text
artifact size
compression
parallel jobs
registry usage
```

If an artifact is hundreds of MB and only needed by deployment infrastructure, a package/container registry may be more appropriate.

---

## 83. Cache Hit Rate

Monitor:

```text
cache hit
cache miss
restore duration
upload duration
```

If cache overhead exceeds the savings:

```text
Remove or redesign cache
```

Caching is not automatically beneficial.

---

## 84. Cache Key Too Broad

Example:

```text
global-cache
```

Risks:

```text
incompatible dependencies
cross-project contamination
cache poisoning
hard-to-debug failures
```

Use meaningful isolation.

---

## 85. Cache Key Too Narrow

Example:

```text
unique key for every commit
```

Result:

```text
almost every job misses cache
```

This wastes storage and provides little benefit.

---

## 86. Dependency Locking

Use lock files where supported:

```text
package-lock.json
poetry.lock
requirements lock
Maven dependency management
```

This improves reproducibility.

---

## 87. Reproducible Dependencies

A production build should minimize:

```text
floating dependency versions
floating base images
uncontrolled external downloads
```

Pin versions where practical.

---

## 88. Dependency Supply Chain

CI dependencies include:

```text
Base image
Build tool
Package manager
Libraries
CI templates
Runner
External scripts
```

All are part of the software supply chain.

---

## 89. Remote CI Templates as Dependencies

If `.gitlab-ci.yml` includes a remote/shared template:

```text
Pipeline
 ↓
External template
```

A template change can change pipeline behavior.

Version and review shared CI dependencies.

---

## 90. Artifact Supply Chain

Example:

```text
Source
 ↓
Build
 ↓
Artifact
 ↓
Security scan
 ↓
Registry
 ↓
Deployment
```

At each step maintain traceability.

---

## 91. SBOM as Artifact

A build can produce:

```text
SBOM
```

which documents dependencies/components.

Flow:

```text
Build
 ↓
SBOM
 ↓
Security analysis
 ↓
Release evidence
```

Retain SBOM according to organizational policy.

---

## 92. Vulnerability Report as Artifact

Security tools may generate:

```text
scan-report.json
```

or another supported format.

Store only what is needed and protect sensitive information.

---

## 93. Trivy Output

A container pipeline may run:

```bash
trivy image "$IMAGE"
```

The scan can generate machine-readable output for reporting.

A failed security threshold should stop promotion when policy requires it.

---

## 94. SonarQube Report Flow

Concept:

```text
Source
 ↓
SonarQube analysis
 ↓
Quality Gate
 ↓
Build promotion
```

The analysis result is evidence, not a replacement for tests.

---

## 95. Veracode Dependency

If Veracode is part of the pipeline:

```text
Build/package
 ↓
Veracode analysis
 ↓
Policy decision
 ↓
Promotion
```

Treat scanner credentials as protected variables or use supported identity mechanisms.

---

## 96. Artifact and Security Gate

Strong model:

```text
Build
 ↓
Artifact
 ↓
Security
 ↓
Approved
 ↓
Publish/promote
```

Avoid publishing a production-trusted artifact before required security gates unless the registry is explicitly a quarantine stage.

---

## 97. Dependency Failure Scenario

Suppose:

```text
Build
 ↓
artifact missing
 ↓
Docker job starts
```

The Docker job should fail clearly rather than silently rebuilding from source.

This preserves build-once semantics.

---

## 98. Dependency Failure Troubleshooting

Check:

```text
Producer job status
 ↓
Artifact path
 ↓
Artifact retention
 ↓
needs/dependencies
 ↓
Job rules
 ↓
Runner
```

---

## 99. Artifact Not Available in Downstream Job

Possible causes:

- upstream job did not produce it
- wrong path
- artifact expired
- incorrect dependency
- `needs` configuration
- job not included
- permissions/storage problem

Diagnose the pipeline graph first.

---

## 100. Artifact Exists but Is Old

If a downstream job uses the wrong artifact:

```text
Check dependency source
 ↓
Check pipeline ID
 ↓
Check artifact name
 ↓
Check job relationship
```

Never blindly use an artifact from another pipeline.

---

## 101. Cross-Pipeline Artifacts

Cross-pipeline artifact consumption requires deliberate configuration and access.

Validate:

```text
project
pipeline
job
artifact
permissions
```

Do not assume same-name artifacts are interchangeable.

---

## 102. Artifact Expired Before Deployment

This is a pipeline design problem.

Options:

```text
Short pipeline → short retention
Long promotion lifecycle → durable registry
```

For production promotion, store the artifact in an appropriate durable registry.

---

## 103. Cache Causes Intermittent Failure

Symptoms:

```text
Pipeline sometimes passes
Pipeline sometimes fails
Clearing cache fixes it
```

Potential causes:

- race condition
- stale dependencies
- shared cache poisoning
- incompatible platform cache
- incomplete cache key

Redesign the cache rather than accepting intermittent behavior.

---

## 104. Concurrent Cache Writes

Multiple jobs may write the same cache.

Potential result:

```text
Job A ─┐
       ├──→ shared cache
Job B ─┘
```

If the cache contents are not safely shareable, isolate keys.

---

## 105. Dependency Race Condition

Example:

```text
Job A generates file
Job B assumes file exists
```

without explicit dependency.

Correct:

```text
Job A
 ↓
artifact
 ↓
Job B
```

The pipeline graph should express actual data dependencies.

---

## 106. Cache Race vs Artifact

If correctness depends on one job producing a file:

```text
Artifact
```

not:

```text
Cache
```

This is one of the most important CI design rules.

---

## 107. Parallel Test Dependencies

Example:

```text
build
 ├── unit-test
 ├── integration-test
 └── security-test
```

All can consume the same immutable artifact if appropriate.

This reduces repeated builds.

---

## 108. Parallel Microservice Builds

For a microservices platform:

```text
User build
Cart build
Order build
Payment build
Inventory build
Notification build
```

can run independently.

Then:

```text
individual security
 ↓
individual image
 ↓
GitOps update
```

This improves scalability.

---

## 109. Monorepo Selective Artifacts

If only one service changes:

```text
services/user/
```

build:

```text
user artifact
```

instead of rebuilding the whole platform.

Combine:

```text
rules:changes
+
needs
+
service-specific artifacts
```

---

## 110. Artifact Naming in Monorepo

Use service-specific names:

```text
user-$CI_COMMIT_SHORT_SHA
cart-$CI_COMMIT_SHORT_SHA
order-$CI_COMMIT_SHORT_SHA
```

This avoids ambiguity.

---

## 111. Dependency Graph for Microservices

Example:

```text
              source
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
      user     cart     order
       │        │        │
       ▼        ▼        ▼
     tests    tests    tests
       │        │        │
       ▼        ▼        ▼
     image    image    image
       │        │        │
       └────────┼────────┘
                ▼
           GitOps update
```

This is a scalable CI design.

---

## 112. Artifact and GitOps

The GitOps repository should normally reference:

```text
image tag
```

and preferably:

```text
image digest
```

The actual binary remains in the registry.

---

## 113. Artifact Digest Update

Example conceptual:

```text
ECR:
user-service@sha256:abc123...
```

GitOps:

```yaml
image:
  repository: <ecr-repository>
  digest: sha256:abc123...
```

ArgoCD reconciles the desired state.

---

## 114. Cache Does Not Belong in GitOps

Do not commit:

```text
CI cache
```

to the GitOps repository.

GitOps should contain declarative desired configuration, not ephemeral CI performance data.

---

## 115. Cache Does Not Replace Registry

Bad:

```text
Build
 ↓
Cache
 ↓
Production
```

Better:

```text
Build
 ↓
Registry
 ↓
Production
```

Cache accelerates builds; registry stores deployable artifacts.

---

## 116. Artifact Does Not Replace Source Control

Artifact:

```text
compiled output
```

Git:

```text
source + configuration
```

Both have different roles.

---

## 117. Artifact Metadata

Attach useful metadata where possible:

```text
commit SHA
version
pipeline ID
build timestamp
branch/tag
security status
```

This helps trace production incidents back to source.

---

## 118. Release Manifest

A release manifest can describe:

```text
Application
Version
Commit
Image digest
Helm version
Environment
Security status
```

Example:

```text
release:
  version: 1.8.0
  commit: a81c92f
  image: sha256:...
```

---

## 119. Artifact Promotion Approval

Production promotion can require:

```text
Security gate
+
manual approval
+
protected environment
```

The artifact should remain unchanged during approval.

---

## 120. Deployment Rollback

Rollback should reference:

```text
previous known-good artifact
```

not:

```text
rebuild old source
```

This makes recovery more deterministic.

---

## 121. Artifact Retention for Rollback

If rollback artifacts are deleted too early:

```text
Incident
 ↓
Need previous artifact
 ↓
Artifact unavailable
```

Keep enough durable release history for operational recovery.

---

## 122. Cache Retention for Rollback

Cache is irrelevant to rollback.

Rollback needs:

```text
known-good deployable artifact
```

not:

```text
old dependency cache
```

---

## 123. Production Artifact Storage Strategy

For your AWS/EKS environment:

```text
GitLab
  ↓
CI
  ↓
ECR
  ↓
Image digest
  ↓
GitOps
  ↓
ArgoCD
  ↓
EKS
```

Use GitLab artifacts for:

```text
reports
evidence
temporary build outputs
```

and ECR for deployable container images.

---

## 124. Artifact and JFrog Artifactory

If Artifactory is used:

```text
CI
 ↓
Build package
 ↓
Artifactory
 ↓
Promotion
```

It can store packages such as:

```text
Maven
npm
Python
Docker/OCI
```

depending on the configured repositories.

---

## 125. Artifact Repository Security

Use:

```text
authentication
authorization
TLS
repository separation
retention
scanning
audit
```

Avoid anonymous write access.

---

## 126. Dependency Proxy

A dependency proxy can reduce repeated external downloads and improve supply-chain control where available.

Concept:

```text
CI
 ↓
Proxy/cache
 ↓
External registry
```

This can improve:

- speed
- availability
- dependency control

---

## 127. Dependency Availability

External package registries can fail.

A cache/proxy can provide resilience:

```text
Public registry outage
 ↓
Internal cache
 ↓
Build continues
```

But ensure cached packages are trusted and correctly versioned.

---

## 128. Dependency Pinning

Avoid:

```text
latest
```

for build dependencies.

Prefer controlled versions:

```text
Python 3.12.x
Terraform approved version
Trivy approved version
```

Pin according to your patching strategy.

---

## 129. Base Image Dependencies

Container builds depend on:

```text
Base image
```

Track:

```text
image version
digest
vulnerability status
```

A base image update can change application output.

---

## 130. Dependency Update Pipeline

Automate:

```text
Dependency update
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
MR
```

Do not blindly deploy dependency updates.

---

## 131. Artifact SBOM Correlation

For a release:

```text
Image digest
 ↓
SBOM
 ↓
Vulnerability report
 ↓
Release
```

This provides stronger supply-chain traceability.

---

## 132. Artifact Signing

Where organizational tooling supports it:

```text
Build
 ↓
Sign artifact/image
 ↓
Verify signature
 ↓
Deploy
```

Signing provides an additional integrity control.

---

## 133. Signature vs Digest

Digest answers:

```text
What exact content is this?
```

Signature answers:

```text
Who/what authorized this content?
```

Together they provide stronger supply-chain assurance.

---

## 134. Dependency Trust

Do not trust:

```text
artifact
cache
template
image
package
```

merely because CI produced or downloaded it.

Establish:

```text
source
integrity
provenance
policy
```

---

## 135. Production Incident — Malicious Artifact

If a compromised artifact is detected:

```text
Stop promotion
 ↓
Identify affected digest
 ↓
Quarantine/remove according to registry policy
 ↓
Identify pipelines/releases using it
 ↓
Rebuild from trusted source
 ↓
Scan/sign
 ↓
Redeploy
```

Also investigate the supply-chain entry point.

---

## 136. Production Incident — Cache Poisoning

Response:

```text
Stop affected pipelines
 ↓
Invalidate cache
 ↓
Inspect Runner/job
 ↓
Review untrusted pipeline access
 ↓
Rotate credentials if exposed
 ↓
Redesign cache isolation
```

---

## 137. Production Incident — Artifact Storage Unavailable

If GitLab artifact storage is unavailable:

```text
Determine affected jobs
 ↓
Check registry availability
 ↓
Avoid unnecessary rebuilds
 ↓
Restore storage path
 ↓
Verify artifact integrity
```

Durable production artifacts should have an appropriate registry strategy.

---

## 138. Production Incident — Wrong Artifact Deployed

Check:

```text
Git commit
 ↓
Pipeline ID
 ↓
Job
 ↓
Artifact/image tag
 ↓
Image digest
 ↓
GitOps commit
 ↓
ArgoCD revision
```

This traceability chain should identify where the wrong artifact entered the deployment path.

---

## 139. Senior Interview — Artifact vs Cache

> Artifacts are job outputs that need to be preserved or passed to downstream jobs. Cache is primarily a performance optimization for reusable data such as dependencies. I never make production correctness depend on cache availability.

---

## 140. Senior Interview — Why Build Once?

> Rebuilding for every environment can produce different binaries because dependencies, base images, tools, or external repositories may change. I prefer building once, testing and scanning that artifact, publishing it immutably, and promoting the same artifact through environments.

---

## 141. Senior Interview — How Do You Optimize a Slow Pipeline?

> I measure job duration and queue time first. Then I use parallelization, `needs`, dependency caching, selective execution with `rules:changes`, smaller artifacts, efficient Docker contexts, and appropriate Runner capacity. I avoid sacrificing required tests or security controls.

---

## 142. Senior Interview — Why Not Use `latest`?

> `latest` is mutable and does not uniquely identify what was deployed. I prefer immutable versioning or commit-based tags and record the image digest for production traceability.

---

## 143. Senior Interview — What Should Be Stored in GitLab Artifacts?

> Temporary build outputs, test reports, coverage, security evidence, generated manifests, and other job outputs. Large deployable binaries or container images are often better stored in a package/container registry.

---

## 144. Senior Interview — How Do You Protect Artifacts?

> I restrict access, minimize contents, set retention, avoid secrets, use encryption where supported, and store production artifacts in an appropriate registry with controlled permissions.

---

## 145. Senior Interview — Why Can Cache Be Dangerous?

> Shared caches can become a trust boundary. An untrusted job could potentially poison cached content consumed by a trusted job. I scope cache keys and separate trusted/untrusted workloads where necessary.

---

## 146. Senior Interview — How Do You Handle Terraform Plans?

> I treat plans as potentially sensitive, restrict artifact access and retention, avoid exposing state, and ensure the plan corresponds to the intended environment and code revision.

---

## 147. Senior Interview — How Do You Handle Microservice Pipelines?

> I build services independently where possible, use `rules:changes` to avoid unrelated builds, use explicit dependencies, run tests/security checks in parallel, publish immutable images to ECR, and update the GitOps repository with exact image identities.

---

## 148. Senior Interview — How Does GitOps Change Artifact Handling?

> CI creates and validates the artifact, then records its immutable identity in GitOps. ArgoCD reconciles that desired state into EKS. The GitOps repository does not need to contain the binary itself.

---

## 149. Senior Interview — How Do You Roll Back?

> I roll back to a known-good immutable artifact or GitOps revision rather than rebuilding an older source commit. This reduces uncertainty during incidents.

---

## 150. Senior Interview — What Is the Biggest Artifact Mistake?

> Rebuilding the application separately for each environment. That breaks the build-once/promote model and can cause environment-to-environment differences.

---

## 151. Production Artifact Architecture

```text
                    GitLab
                       │
                       ▼
                   CI Pipeline
                       │
              ┌────────┴────────┐
              ▼                 ▼
          Artifacts           Cache
              │                 │
              │                 └── Dependencies
              │
              ├── Reports
              ├── Evidence
              └── Build outputs
                       │
                       ▼
                 ECR / Registry
                       │
                       ▼
                 Image Digest
                       │
                       ▼
                 GitOps Repo
                       │
                       ▼
                    ArgoCD
                       │
                       ▼
                      EKS
```

---

## 152. Recommended Cache Architecture

```text
                 GitLab Runner
                       │
                       ▼
                Cache Request
                       │
              ┌────────┴────────┐
              ▼                 ▼
          Local cache       Remote cache
                               │
                               ▼
                              S3
                               │
                         Lifecycle policy
```

Use remote caching when ephemeral/autoscaled Runners make local caching ineffective.

---

## 153. Recommended Microservice Dependency Graph

```text
                    Source
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
      User           Cart          Order
        │             │             │
        ▼             ▼             ▼
      Test           Test          Test
        │             │             │
        ▼             ▼             ▼
     Security      Security      Security
        │             │             │
        ▼             ▼             ▼
       ECR           ECR           ECR
        │             │             │
        └─────────────┼─────────────┘
                      ▼
                 GitOps Update
                      │
                      ▼
                    ArgoCD
                      │
                      ▼
                     EKS
```

---

## 154. Production Readiness Checklist

```text
[ ] Artifacts have clear purpose
[ ] Artifact paths are precise
[ ] Artifact retention is defined
[ ] Sensitive artifacts are protected
[ ] Cache is not used as source of truth
[ ] Cache keys are meaningful
[ ] Cache trust boundaries are defined
[ ] Dependency graph is explicit
[ ] `needs` used where beneficial
[ ] Unnecessary dependencies removed
[ ] Build once / promote model
[ ] Image digests recorded
[ ] Registry used for durable deployables
[ ] Test reports retained appropriately
[ ] Security reports handled securely
[ ] Terraform state never uploaded casually
[ ] SBOM retained where required
[ ] Artifact provenance available
[ ] Rollback artifact retained
[ ] Cache lifecycle configured
[ ] Remote cache secured
[ ] Runner capacity monitored
[ ] Pipeline performance measured
[ ] Production artifact access restricted
```

---

## 155. Final Mental Model

```text
                    Git Commit
                        │
                        ▼
                     Build
                        │
               ┌────────┴────────┐
               ▼                 ▼
           Artifact             Cache
               │                 │
               │            Speed only
               ▼
             Test
               │
               ▼
           Security
               │
               ▼
          Publish/Store
               │
        ┌──────┴──────┐
        ▼             ▼
      Reports         ECR
                        │
                        ▼
                   Image Digest
                        │
                        ▼
                   GitOps Repo
                        │
                        ▼
                      ArgoCD
                        │
                        ▼
                       EKS
```

> **Artifacts preserve trusted job outputs; cache accelerates repeatable work; dependencies define the real data flow. A production GitLab pipeline should build once, validate once, publish immutable artifacts, and promote the exact same artifact through environments.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md ✓
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `09-GitLab-Docker-and-Container-Registry.md`**
