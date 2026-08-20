# GitLab Advanced Pipelines

> Production-oriented guide to advanced GitLab pipeline engineering: reusable architecture, DAG execution, dynamic pipelines, parent-child and multi-project orchestration, monorepos, microservices, conditional execution, parallelism, matrix jobs, pipeline components, deployment serialization, artifact flow, release orchestration, security controls, performance engineering, failure recovery, and senior DevOps interview scenarios.

---

## 1. Advanced Pipeline Engineering

An advanced pipeline should optimize:

```text
Speed
Reliability
Security
Reusability
Traceability
Scalability
Recovery
```

---

## 2. Pipeline as a Product

Treat CI/CD as an internal platform.

It needs:

```text
standards
documentation
versioning
monitoring
support
security
```

---

## 3. Pipeline Architecture

A production pipeline commonly has:

```text
Source
 ↓
Validation
 ↓
Build
 ↓
Security
 ↓
Package
 ↓
Publish
 ↓
Deploy
 ↓
Verify
```

---

## 4. Pipeline Design Before YAML

Before writing YAML define:

```text
inputs
outputs
dependencies
security boundaries
environments
failure behavior
```

---

## 5. Pipeline Dependency Graph

Example:

```text
Lint ───────┐
Unit Test ──┼──→ Build ──→ Scan ──→ Publish
SAST ───────┤
SCA ────────┘
```

---

## 6. Critical Path

Pipeline time is largely determined by the longest dependency chain.

Reduce unnecessary serial steps.

---

## 7. DAG vs Stages

Stages are useful for organization.

DAG dependencies are useful for execution efficiency.

Use both where appropriate.

---

## 8. `needs`

Example:

```yaml
package:
  needs:
    - build
```

The job does not need to wait for unrelated jobs.

---

## 9. Artifact Dependency

When a downstream job needs build output, explicitly model the artifact dependency.

---

## 10. `needs` and Artifacts

Conceptually:

```text
build
 └── artifact
       ↓
test
```

This makes data flow explicit.

---

## 11. Parallel Security

Security checks can often run concurrently:

```text
SAST
SCA
Secret Detection
IaC Scan
```

---

## 12. Security Dependency

If image scanning requires the built image:

```text
Build
 ↓
Image Scan
```

Do not attempt to scan before the artifact exists.

---

## 13. Conditional Jobs

Use conditions for:

```text
branch
tag
MR
changed paths
pipeline source
environment
variables
```

---

## 14. `rules`

Example:

```yaml
rules:
  - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

---

## 15. `rules:changes`

Example:

```yaml
rules:
  - changes:
      - services/orders/**/*
```

---

## 16. `rules:exists`

Use file existence to control jobs when pipeline behavior depends on repository structure.

---

## 17. Rule Evaluation

Order rules carefully.

A broad rule placed before a specific rule can change expected behavior.

---

## 18. `when: never`

Use explicit exclusions where necessary.

Example:

```yaml
rules:
  - if: '$CI_PIPELINE_SOURCE == "schedule"'
    when: never
```

---

## 19. Workflow Rules

Use:

```yaml
workflow:
  rules:
```

to control pipeline creation globally.

---

## 20. Duplicate Pipelines

Prevent unnecessary:

```text
push pipeline
+
MR pipeline
```

running for the same change unless both are intentionally required.

---

## 21. Pipeline Sources

Common sources include:

```text
push
merge_request_event
schedule
web
pipeline
trigger
parent_pipeline
```

Design rules around the actual source.

---

## 22. MR Pipeline

A merge request pipeline should normally validate:

```text
tests
lint
security
build
```

without deploying production.

---

## 23. Main Branch Pipeline

Main can trigger:

```text
build
publish
Dev deployment
```

depending on the release model.

---

## 24. Tag Pipeline

Tags are useful for:

```text
release
production promotion
versioned artifacts
```

---

## 25. Scheduled Pipeline

Schedules are useful for:

```text
dependency scans
container rescans
maintenance
nightly integration tests
```

---

## 26. Manual Pipeline

Manual pipeline execution can support:

```text
emergency validation
release
maintenance
reprocessing
```

Restrict permissions appropriately.

---

## 27. Pipeline Variables

Variables can influence behavior.

Never allow untrusted users to override security-critical variables without controls.

---

## 28. Variable Precedence

Understand GitLab variable precedence before designing complex configurations.

Do not assume a variable value comes from only one location.

---

## 29. Protected Variables

Sensitive values should be restricted to protected refs/environments as appropriate.

---

## 30. Masked Variables

Masking helps reduce accidental log exposure but does not protect against malicious jobs that can use the secret.

---

## 31. Pipeline Inputs

Validate:

```text
environment
version
region
action
```

before using them in deployment commands.

---

## 32. Input Validation

Avoid directly embedding user-controlled values into shell commands.

---

## 33. Shell Injection

Risky:

```bash
eval "$COMMAND"
```

Avoid dynamic shell execution unless the input is strictly controlled.

---

## 34. Safe Shell Handling

Prefer:

```bash
command "$VARIABLE"
```

rather than constructing arbitrary shell expressions.

---

## 35. `extends`

Reusable job configuration:

```yaml
.base:
  image: alpine:latest
  before_script:
    - echo "prepare"

deploy:
  extends: .base
```

Pin images appropriately in production.

---

## 36. YAML Anchors

Useful for local repetition.

Do not make configuration so abstract that troubleshooting becomes difficult.

---

## 37. Includes

Split complex pipelines:

```text
.gitlab-ci.yml
ci/build.yml
ci/test.yml
ci/security.yml
ci/deploy.yml
```

---

## 38. Pipeline Modularity

A large monolithic YAML file becomes difficult to:

```text
review
debug
reuse
govern
```

---

## 39. Central Templates

Platform teams can provide:

```text
security template
Docker template
Terraform template
Kubernetes template
```

---

## 40. Template Versioning

Version templates to avoid unexpected breaking changes.

---

## 41. Template Testing

Before releasing a central template:

```text
lint
test
run sample pipelines
validate security behavior
```

---

## 42. Template Security

Protect template repositories.

A malicious template can compromise every consuming project.

---

## 43. Pipeline Components

Reusable CI/CD components can standardize repeated automation.

Use governed versions and documented inputs/outputs.

---

## 44. Component Interface

A reusable component should define:

```text
inputs
outputs
variables
artifacts
failure behavior
```

---

## 45. Parent-Child Pipeline

Example:

```text
Parent
 ├── frontend child
 ├── backend child
 └── infrastructure child
```

---

## 46. Child Pipeline Trigger

A parent can trigger a child pipeline after required preparation.

---

## 47. Parent Pipeline Ownership

The parent should coordinate high-level workflow.

Child pipelines should own domain-specific logic.

---

## 48. Child Pipeline Artifacts

Define how artifacts/results move between parent and child pipelines.

---

## 49. Dynamic Child Pipeline

Generate configuration based on repository content.

Example:

```text
Detect changed services
 ↓
Generate CI configuration
 ↓
Trigger child pipelines
```

---

## 50. Dynamic Pipeline Security

Generated configuration is executable.

Validate generated output before execution where possible.

---

## 51. Monorepo Architecture

Example:

```text
repo/
├── services/
│   ├── user
│   ├── cart
│   ├── orders
│   └── payment
├── shared/
└── infrastructure/
```

---

## 52. Monorepo Problem

A change to:

```text
services/user
```

should not necessarily run every service pipeline.

---

## 53. Path-Based Execution

Use:

```yaml
rules:
  - changes:
      - services/user/**/*
```

for targeted jobs.

---

## 54. Shared Code

If:

```text
shared/**
```

changes, trigger all dependent services.

---

## 55. Dependency Map

Maintain a dependency map:

```text
shared
 ├── user
 ├── cart
 ├── orders
 └── payment
```

---

## 56. Monorepo Pipeline

```text
Detect Changes
      │
 ┌────┼─────┐
 ▼    ▼     ▼
User Cart Orders
 │    │      │
 ▼    ▼      ▼
Tests Tests Tests
```

---

## 57. Microservices Pipeline

Each service can own:

```text
Dockerfile
tests
CI
Helm
```

while common standards remain centralized.

---

## 58. Service Template

Example common pipeline:

```text
test
build
scan
publish
```

Each service provides its own variables.

---

## 59. Matrix Jobs

Matrix execution can test combinations:

```text
Python 3.11
Python 3.12
Python 3.13
```

---

## 60. Matrix Explosion

Avoid unnecessarily large matrices.

Calculate:

```text
versions × OS × database
```

before enabling combinations.

---

## 61. Parallel Jobs

Independent jobs should run simultaneously where runner capacity allows.

---

## 62. Runner Capacity

More parallelism requires more runner capacity.

Optimize both:

```text
pipeline graph
runner infrastructure
```

---

## 63. Concurrency

Concurrency controls how many jobs run simultaneously.

It is different from pipeline dependency design.

---

## 64. Resource Groups

Use resource groups for shared deployment targets.

Example:

```text
production
```

ensures competing production deployments are serialized.

---

## 65. Deployment Race

Without concurrency control:

```text
Pipeline A → Prod
Pipeline B → Prod
```

could produce unpredictable final state.

---

## 66. Environment Lock

Production environments should have controlled deployment concurrency.

---

## 67. Interruptible Jobs

Safe-to-cancel jobs can be interrupted when newer commits supersede them.

Useful for:

```text
feature branch tests
MR validation
```

---

## 68. Non-Interruptible Jobs

Avoid interrupting:

```text
database migration
production deployment
destructive infrastructure operation
```

unless the operation is explicitly designed for interruption.

---

## 69. Retry

Use retries only for transient failures.

---

## 70. Retryable Failures

Examples:

```text
temporary network timeout
registry unavailable
external API 503
```

---

## 71. Non-Retryable Failures

Examples:

```text
unit test failure
compile error
security policy violation
invalid Terraform
```

---

## 72. Exponential Backoff

Retries should use increasing delays.

Concept:

```text
1s
2s
4s
8s
```

with a reasonable maximum.

---

## 73. Jitter

Randomized delay can prevent many clients from retrying simultaneously.

---

## 74. Timeouts

Configure:

```text
job timeout
HTTP timeout
deployment timeout
```

for external operations.

---

## 75. Stuck Job

A job that hangs indefinitely consumes runner capacity.

Use reasonable timeouts.

---

## 76. Artifact Flow

Typical:

```text
Build
 ↓ artifact
Test
 ↓ artifact
Package
 ↓ artifact
Publish
```

---

## 77. Artifact Retention

Use retention policies appropriate to:

```text
development
release
production
security reports
```

---

## 78. Artifact Security

Do not publish:

```text
private keys
.env
credentials
production dumps
```

as artifacts.

---

## 79. Cache vs Artifact

Cache:

```text
performance optimization
```

Artifact:

```text
intentional build output
```

---

## 80. Cache Key

A good cache key should reflect dependency changes.

Example concept:

```text
lockfile hash
```

---

## 81. Cache Poisoning

Do not allow untrusted branches to influence trusted production build inputs through unsafe shared caches.

---

## 82. Dependency Cache

Useful for:

```text
Maven
npm
pip
Terraform providers
```

where appropriate.

---

## 83. Docker Build Cache

Build cache can significantly reduce image build time.

Use secure cache sources.

---

## 84. BuildKit Cache

BuildKit supports efficient cache mechanisms and modern build features.

---

## 85. Build Once

Preferred:

```text
source
 ↓
build
 ↓
scan
 ↓
publish
```

Then promote.

---

## 86. Rebuild Risk

Rebuilding for production may introduce:

```text
different dependency
different base image
different toolchain
```

---

## 87. Image Digest

Use:

```text
sha256:...
```

to identify the exact OCI image.

---

## 88. Registry Promotion

Promote the same digest between logical environments.

---

## 89. Security Gate

Block promotion when mandatory security policy fails.

---

## 90. Security Exception

If an exception is allowed, require:

```text
reason
owner
approval
expiry
```

---

## 91. SAST

Run source analysis early.

---

## 92. SCA

Scan direct and transitive dependencies.

---

## 93. Secret Detection

Detect:

```text
AWS keys
tokens
passwords
private keys
```

before merge.

---

## 94. Container Scan

Use Trivy or approved tooling to scan the image.

---

## 95. IaC Scan

Scan:

```text
Terraform
Kubernetes
Helm
Docker
```

for security issues.

---

## 96. DAST

Run DAST against an appropriate deployed environment.

Avoid unauthorized destructive tests against production.

---

## 97. Security Parallelization

```text
              ┌── SAST
Build ────────┼── SCA
              ├── Secret Scan
              └── IaC
```

---

## 98. DAST Dependency

DAST requires an application environment.

```text
Deploy Test
 ↓
DAST
```

---

## 99. Test Pyramid

Pipeline tests can include:

```text
Unit
Integration
Contract
End-to-End
DAST
```

Balance confidence against execution time.

---

## 100. Fast Feedback

Run cheap checks early:

```text
lint
unit tests
syntax
```

before expensive tests.

---

## 101. Expensive Tests

Examples:

```text
full integration
browser tests
load tests
DAST
```

Run where they provide the most value.

---

## 102. Test Parallelization

Split large suites:

```text
test shard 1
test shard 2
test shard 3
```

---

## 103. Test Sharding

Divide tests across parallel jobs.

Useful for large suites.

---

## 104. Sharding Risk

Ensure test isolation.

Shared state can cause flaky results.

---

## 105. Flaky Tests

A flaky test passes/fails without a meaningful code change.

Track and fix it rather than hiding it with retries.

---

## 106. Test Retry

Limited retries can reduce noise while investigation continues.

Do not treat retries as the permanent solution to flakiness.

---

## 107. Pipeline Quality Gate

Example:

```text
Unit Tests ✓
Integration ✓
SAST ✓
SCA ✓
Trivy ✓
```

then:

```text
Publish
```

---

## 108. Production Promotion Gate

Example:

```text
Stage validation ✓
Security ✓
Smoke tests ✓
Approval ✓
```

then:

```text
Production
```

---

## 109. Environment Dependencies

Model:

```text
Build
 ↓
Dev
 ↓
Stage
 ↓
Prod
```

explicitly when required.

---

## 110. Cross-Project Pipeline

Example:

```text
Application CI
 ↓
GitOps update
 ↓
Deployment project
```

---

## 111. Pipeline Trigger Token

Trigger credentials should be:

```text
scoped
rotated
protected
```

---

## 112. Pipeline API

Use the GitLab API for controlled automation such as:

```text
trigger pipeline
query status
create release
update variables
```

Use least-privilege credentials.

---

## 113. API Automation Identity

Avoid using a developer's personal token for critical automation.

Prefer:

```text
project token
group token
job token
OIDC
```

depending on the operation.

---

## 114. Pipeline Status Automation

Automation can wait for:

```text
running
pending
success
failed
canceled
```

and act accordingly.

---

## 115. Trigger Chain

Avoid excessively long chains:

```text
A → B → C → D → E
```

They become difficult to troubleshoot.

---

## 116. Pipeline Orchestration

Keep orchestration responsibilities clear.

Example:

```text
Parent = coordination
Child = domain execution
```

---

## 117. Pipeline Failure Propagation

Define whether downstream pipelines should run when upstream pipelines fail.

Production flows should fail closed for mandatory checks.

---

## 118. Fail Fast

Fail quickly on:

```text
syntax
configuration
missing variable
security policy
```

---

## 119. Fail Safely

A production deployment failure should not silently continue to the next stage.

---

## 120. Manual Recovery

Manual intervention should be explicit and auditable.

---

## 121. Deployment Retry

Retrying a failed deployment is safe only when the deployment is idempotent and the failure is understood.

---

## 122. Idempotency

A deployment can be repeated without causing unintended duplicate state.

---

## 123. Kubernetes Deployment Idempotency

Declarative manifests are naturally suited to repeated reconciliation.

---

## 124. Terraform Idempotency

Terraform compares desired and current state rather than blindly creating resources.

---

## 125. Database Idempotency

Migration systems should track applied migrations and avoid re-running destructive operations.

---

## 126. Release Pipeline

```text
Validate
 ↓
Build
 ↓
Scan
 ↓
Publish
 ↓
Deploy
 ↓
Verify
 ↓
Release
```

---

## 127. Release Metadata

Record:

```text
version
commit
pipeline ID
image digest
environment
```

---

## 128. Release Tag

Use immutable release tags according to organizational convention.

---

## 129. Release Candidate

A release candidate is an artifact/configuration combination intended for final validation.

---

## 130. Promotion Without Rebuild

```text
RC digest
 ↓
Stage
 ↓
Prod
```

---

## 131. Canary Pipeline

```text
Build
 ↓
Deploy 5%
 ↓
Observe
 ↓
Deploy 25%
 ↓
Observe
 ↓
Deploy 100%
```

---

## 132. Canary Failure

If:

```text
error rate ↑
latency ↑
```

stop rollout and restore previous traffic.

---

## 133. Blue/Green Pipeline

```text
Build
 ↓
Deploy Green
 ↓
Test
 ↓
Switch traffic
 ↓
Observe
```

---

## 134. Rolling Pipeline

Kubernetes gradually replaces Pods.

Configure:

```text
maxUnavailable
maxSurge
```

based on availability requirements.

---

## 135. Deployment Verification

Check:

```text
ArgoCD
Kubernetes
ALB
application
Prometheus
ELK
```

---

## 136. Smoke Test

Example:

```bash
curl -f https://app.example.com/health
```

---

## 137. Functional Smoke Test

Validate critical behavior:

```text
login
catalog
cart
order
```

where appropriate.

---

## 138. Post-Deployment Monitoring

Watch:

```text
5xx
latency
CPU
memory
restarts
business metrics
```

---

## 139. Automatic Rollback

Only automate rollback when:

```text
signal is reliable
threshold is defined
rollback is safe
```

---

## 140. Rollback Source of Truth

In GitOps, prefer reverting desired state rather than making ad-hoc cluster changes.

---

## 141. Production Rollback

```text
Identify previous digest
 ↓
Git revert
 ↓
ArgoCD sync
 ↓
Health validation
```

---

## 142. Pipeline Observability

Measure:

```text
duration
queue time
success rate
failure rate
runner utilization
```

---

## 143. DORA Metrics

Track:

```text
Deployment Frequency
Lead Time for Changes
Change Failure Rate
Time to Restore Service
```

---

## 144. Pipeline Reliability

A fast pipeline that frequently fails is not successful.

Optimize:

```text
speed
AND
reliability
```

---

## 145. Pipeline Failure Categories

Classify:

```text
code
test
security
runner
network
registry
AWS
Kubernetes
GitOps
```

---

## 146. Runner Failure

If all jobs fail:

```text
check runner
check executor
check network
check image
check permissions
```

---

## 147. Registry Failure

If image push fails:

```text
check credentials
check ECR
check repository
check network
check image size
```

---

## 148. AWS Failure

Check:

```text
OIDC
IAM
STS
region
service limits
```

---

## 149. ArgoCD Failure

Check:

```text
Git revision
repository access
manifest rendering
sync status
health
```

---

## 150. Kubernetes Failure

Check:

```text
Pods
Events
Services
Ingress
Probes
Resources
```

---

## 151. Pipeline Debugging Order

```text
Pipeline status
 ↓
Failed job
 ↓
Job logs
 ↓
Variables/config
 ↓
Runner
 ↓
External dependency
```

---

## 152. Log Analysis

Search for:

```text
ERROR
FAILED
timeout
denied
unauthorized
not found
```

---

## 153. CI Debugging

Do not immediately rerun.

First determine:

```text
why it failed
```

A rerun can hide a flaky infrastructure problem.

---

## 154. Reproducibility

A good pipeline should produce the same result from the same:

```text
source
dependencies
build environment
configuration
```

---

## 155. Dependency Pinning

Pin:

```text
base images
package versions
CI templates
scanner versions
Terraform providers
```

where reproducibility is required.

---

## 156. Floating Tags

Avoid relying on:

```text
latest
```

for critical build dependencies.

---

## 157. Container Base Image

Use approved base images.

Scan them regularly.

---

## 158. Secure Build Environment

Build runners should be:

```text
patched
isolated
minimal
trusted
```

---

## 159. CI Runner Network

Restrict outbound traffic where practical.

Allow only required dependencies.

---

## 160. Build Environment Secrets

Prefer short-lived credentials.

Avoid writing credentials to disk unnecessarily.

---

## 161. OIDC Build Authentication

```text
GitLab
 ↓
OIDC
 ↓
AWS STS
 ↓
temporary credentials
```

---

## 162. Production Deployment Credential

Only the job/environment that needs production access should receive it.

---

## 163. Fork Protection

Do not expose production secrets to untrusted fork pipelines.

---

## 164. Protected Branch

Production deployment should originate from an approved branch/tag/revision.

---

## 165. Protected Environment

Production environment access should be independently restricted.

---

## 166. Defense in Depth

```text
Branch protection
+
MR approval
+
CI tests
+
Security scans
+
Protected variables
+
OIDC
+
Protected environment
+
GitOps
```

---

## 167. Pipeline Security Review

Review:

```text
.gitlab-ci.yml
includes
scripts
Dockerfiles
variables
runners
deploy commands
```

---

## 168. CI Script Review

Treat deployment scripts as production code.

They can:

```text
delete resources
change infrastructure
deploy applications
```

---

## 169. Destructive Commands

Be careful with:

```bash
rm -rf
terraform destroy
kubectl delete
```

Add explicit environment safeguards.

---

## 170. Environment Guard

Example concept:

```bash
if [ "$ENVIRONMENT" != "production" ]; then
  ...
fi
```

For destructive operations, use stronger explicit confirmation mechanisms rather than relying on a single variable check.

---

## 171. Production Confirmation

Production destructive actions should require deliberate authorization.

---

## 172. Terraform Destroy

Never make production destroy an accidental default pipeline action.

---

## 173. Kubernetes Delete

Restrict destructive permissions.

---

## 174. Database Drop

Production database destruction should be extremely restricted and separated from normal deployment automation.

---

## 175. Pipeline Permissions

Use the narrowest:

```text
GitLab role
token
AWS IAM role
Kubernetes RBAC
```

possible.

---

## 176. Pipeline Secret Rotation

Automate rotation where practical.

---

## 177. Credential Inventory

Maintain:

```text
token
purpose
owner
scope
expiry
```

---

## 178. Security Audit

Periodically review:

```text
protected branches
variables
runners
tokens
integrations
environments
```

---

## 179. Pipeline Compliance

Organizations may require:

```text
mandatory scans
approval
audit
retention
deployment records
```

---

## 180. Compliance Evidence

Store evidence such as:

```text
MR
approval
pipeline result
security scan
deployment record
```

according to policy.

---

## 181. Pipeline Retention

Balance retention with:

```text
audit requirements
storage cost
privacy
```

---

## 182. Pipeline Notifications

Notify on:

```text
production failure
security gate
rollback
successful release
```

---

## 183. Alert Fatigue

Do not notify on every minor development failure.

Prioritize actionable events.

---

## 184. ChatOps Security

Integrations should use:

```text
scoped credentials
webhook validation
TLS
```

---

## 185. Deployment Approval Through Chat

If used, ensure approval identity is authenticated and auditable.

---

## 186. Pipeline Documentation

Document:

```text
what
why
inputs
outputs
failure behavior
rollback
owner
```

---

## 187. Pipeline Ownership

Every critical pipeline should have an owner.

---

## 188. Platform Team

A platform team can maintain:

```text
runner infrastructure
templates
security standards
deployment framework
```

---

## 189. Application Team

Application teams own:

```text
tests
service build
application configuration
service health
```

---

## 190. Shared Responsibility

CI/CD security is shared between:

```text
platform
security
application
cloud
```

teams.

---

## 191. Pipeline Change Review

Changes to:

```text
deployment
security
runner
AWS
Terraform
GitOps
```

should receive appropriate review.

---

## 192. Version-Controlled Pipeline

Never keep critical deployment logic only in a manually configured CI server.

Store it in Git.

---

## 193. Pipeline Portability

Use standard scripts where practical:

```text
scripts/build.sh
scripts/test.sh
scripts/deploy.sh
```

rather than embedding every detail in YAML.

---

## 194. Script Testing

Test scripts independently where practical.

---

## 195. Bash Strict Mode

A common pattern is:

```bash
set -euo pipefail
```

Understand its behavior before using it blindly.

---

## 196. Error Handling

Fail with meaningful messages:

```bash
echo "ERROR: image digest is missing" >&2
exit 1
```

---

## 197. Secret-Safe Errors

Never include:

```text
password
token
secret
```

in error messages.

---

## 198. Structured Pipeline Output

Use clear log markers:

```text
[BUILD]
[TEST]
[SECURITY]
[DEPLOY]
[VERIFY]
```

---

## 199. Pipeline Timing

Record durations of major stages.

This helps identify bottlenecks.

---

## 200. Pipeline Optimization Process

```text
Measure
 ↓
Identify bottleneck
 ↓
Change
 ↓
Measure again
```

Do not optimize based on assumptions.

---

## 201. Queue Optimization

If queue time dominates:

```text
add runners
scale runners
remove unnecessary tags
```

---

## 202. Build Optimization

Use:

```text
dependency cache
Docker cache
smaller build context
multi-stage builds
```

---

## 203. Test Optimization

Use:

```text
parallelism
sharding
targeted tests
changed-path execution
```

---

## 204. Security Optimization

Use:

```text
parallel scans
incremental checks
scheduled deep scans
shared scanner caches
```

without weakening mandatory gates.

---

## 205. Deployment Optimization

Use:

```text
GitOps
immutable artifacts
automated validation
progressive delivery
```

---

## 206. Pipeline Cost Optimization

Track:

```text
runner minutes
storage
network
build compute
```

---

## 207. Expensive Jobs

Identify jobs that consume the most:

```text
time
CPU
memory
```

---

## 208. Pipeline Waste

Examples:

```text
build unchanged service
run tests twice
rebuild same image
run identical scans sequentially
```

---

## 209. Pipeline Efficiency

Target:

```text
less waiting
less duplication
same confidence
```

---

## 210. Advanced Pipeline Architecture

```text
                           GitLab
                              │
                         Workflow Rules
                              │
                              ▼
                         Merge Request
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
               Lint         Unit         Security
                 │            │       ┌────┼─────┐
                 │            │       ▼    ▼     ▼
                 │            │     SAST  SCA  Secret
                 └────────────┼────────────┘
                              ▼
                            Build
                              │
                              ▼
                         Container Scan
                              │
                              ▼
                              ECR
                              │
                        Immutable Digest
                              │
                              ▼
                         GitOps Repo
                              │
                              ▼
                            ArgoCD
                              │
                              ▼
                             EKS
                              │
                              ▼
                   Smoke / Health Validation
                              │
                              ▼
                  Prometheus / Grafana / ELK
```

---

## 211. Advanced Microservices Architecture

```text
                         Parent Pipeline
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
     User Child            Cart Child          Orders Child
          │                    │                    │
       Build/Test            Build/Test           Build/Test
          │                    │                    │
        Scan                  Scan                 Scan
          │                    │                    │
         ECR                  ECR                  ECR
          │                    │                    │
          └────────────────────┼────────────────────┘
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

## 212. Advanced Monorepo Architecture

```text
                         Commit
                           │
                           ▼
                     Change Detection
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
          Service A      Service B     Terraform
             │             │             │
          Child          Child         Child
          Pipeline       Pipeline      Pipeline
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                       Validation
                           │
                           ▼
                         Release
```

---

## 213. Advanced Production Flow

```text
Feature
 ↓
MR
 ↓
Workflow rules
 ↓
DAG validation
 ↓
Parallel tests/security
 ↓
Build
 ↓
Immutable image
 ↓
ECR
 ↓
Dev
 ↓
Integration
 ↓
Stage
 ↓
Approval
 ↓
Prod
 ↓
Canary/Rolling
 ↓
Verification
 ↓
Observability
 ↓
Promote / Rollback
```

---

## 214. Senior Interview — What Makes a Pipeline Advanced?

> An advanced pipeline models real dependencies, executes independent work in parallel, uses reusable components, supports child or multi-project workflows, promotes immutable artifacts, protects production, integrates security and observability, and provides reliable recovery.

---

## 215. Senior Interview — How Do You Optimize a Large Pipeline?

> I measure queue time and job duration first. Then I remove unnecessary dependencies, use DAG execution, parallelize independent checks, cache safe dependencies, use path-based execution, shard expensive tests, and eliminate duplicate work.

---

## 216. Senior Interview — How Do You Design a Monorepo Pipeline?

> I detect changed paths, map shared dependencies, generate or trigger only the necessary child pipelines, and ensure shared-library or infrastructure changes trigger the required broader validation.

---

## 217. Senior Interview — How Do You Avoid Duplicate Pipelines?

> I use `workflow: rules` to define which pipeline sources should create pipelines, and job-level `rules` to determine which jobs belong in each pipeline type.

---

## 218. Senior Interview — When Would You Use Child Pipelines?

> I use child pipelines when a repository contains logically independent domains such as multiple microservices, frontend/backend components, or infrastructure. They keep ownership and pipeline logic modular.

---

## 219. Senior Interview — When Would You Use Multi-Project Pipelines?

> I use them when different repositories own different lifecycle responsibilities, for example an application repository producing an artifact and a GitOps/deployment repository managing environment promotion.

---

## 220. Senior Interview — How Do You Prevent Production Deployment Races?

> I use protected environments and deployment concurrency controls such as resource groups. This prevents two pipelines from modifying the same production target simultaneously.

---

## 221. Senior Interview — How Do You Handle Flaky Tests?

> I identify and track the flaky test, isolate the cause, and fix it. A limited retry may reduce temporary pipeline noise, but I do not use retries as a permanent substitute for fixing test instability.

---

## 222. Senior Interview — How Do You Handle Transient Pipeline Failures?

> I classify the error first. Network, registry and temporary cloud API failures may use bounded retries with exponential backoff and jitter. Code, security and configuration failures should fail immediately.

---

## 223. Senior Interview — How Do You Secure Dynamic Pipelines?

> I treat generated CI configuration as executable code. I control who can modify the generator, validate generated configuration, protect production variables, restrict runners, and avoid exposing secrets to untrusted pipelines.

---

## 224. Senior Interview — How Do You Handle a Large Microservices Platform?

> I use standardized service templates, independent child pipelines, changes-based execution, immutable ECR images, centralized security controls, GitOps deployment and shared observability. Each service can move independently while platform standards remain consistent.

---

## 225. Senior Interview — How Do You Make Production Releases Safe?

> I build once, scan the immutable artifact, promote the same digest, protect the production environment, require appropriate approval, deploy progressively when needed, verify health, monitor metrics/logs and maintain a tested rollback path.

---

## 226. Senior Interview — How Do You Troubleshoot a Slow Pipeline?

> I first separate queue time from execution time. Then I identify the longest jobs, inspect runner capacity, examine dependency and Docker caching, find unnecessary serial dependencies, and check whether duplicate jobs are running.

---

## 227. Senior Interview — How Do You Secure Shared CI Templates?

> I restrict template repository access, require review, version templates, test them before release, pin versions where appropriate, and treat template changes as supply-chain-sensitive because one malicious change can affect many projects.

---

## 228. Senior Interview — How Do You Handle Production Rollback?

> I identify the previous known-good artifact and desired state, revert the GitOps configuration, allow ArgoCD to reconcile, and validate the application through health checks, smoke tests and observability.

---

## 229. Senior Interview — How Do You Design CI/CD for Terraform and EKS?

> I separate infrastructure and application responsibilities where appropriate. Terraform performs controlled infrastructure changes using OIDC and protected environments, while application CI builds and scans images and GitOps with ArgoCD manages Kubernetes desired state.

---

## 230. Senior Interview — How Do You Improve Pipeline Reliability?

> I use idempotent jobs, bounded retries for transient failures, explicit timeouts, deterministic dependencies, immutable artifacts, isolated runners, clear failure classification and tested rollback/recovery procedures.

---

## 231. Senior Interview — What Is Your Advanced Pipeline Mental Model?

> I think of the pipeline as a dependency graph and security boundary. Every job has inputs, outputs, permissions and failure behavior. Independent work runs in parallel, sensitive operations are protected, artifacts are immutable, and production changes are observable and reversible.

---

## 232. Final Advanced Pipeline Checklist

```text
[ ] Pipeline architecture
[ ] DAG
[ ] needs
[ ] workflow rules
[ ] job rules
[ ] changes
[ ] reusable templates
[ ] includes
[ ] components
[ ] child pipelines
[ ] dynamic pipelines
[ ] multi-project pipelines
[ ] monorepo strategy
[ ] matrix jobs
[ ] test sharding
[ ] resource groups
[ ] interruptible jobs
[ ] retries
[ ] timeouts
[ ] artifacts
[ ] cache
[ ] immutable images
[ ] ECR
[ ] security gates
[ ] OIDC
[ ] protected variables
[ ] protected environments
[ ] GitOps
[ ] ArgoCD
[ ] deployment verification
[ ] rollback
[ ] observability
[ ] DORA metrics
[ ] disaster recovery
```

---

## 233. Final Mental Model

```text
                    ADVANCED PIPELINE

                         SOURCE
                           │
                           ▼
                    WORKFLOW CONTROL
                           │
                           ▼
                    DEPENDENCY GRAPH
                           │
          ┌────────────────┼────────────────┐
          ▼                ▼                ▼
        TEST            SECURITY          LINT
          │                │                │
          └────────────────┼────────────────┘
                           ▼
                          BUILD
                           │
                           ▼
                       IMMUTABLE
                         ARTIFACT
                           │
                           ▼
                         ECR
                           │
                           ▼
                        GITOPS
                           │
                           ▼
                        ARGOCD
                           │
                           ▼
                          EKS
                           │
                           ▼
                       VERIFY
                           │
                           ▼
                       OBSERVE
                           │
                           ▼
                   PROMOTE / ROLLBACK
```

> **Core principle:** Advanced GitLab pipelines should not become complex for the sake of complexity. The goal is to make delivery faster and safer by modeling dependencies accurately, reusing proven automation, eliminating duplicate work, protecting execution boundaries, promoting immutable artifacts, and making every production change observable and recoverable.

---