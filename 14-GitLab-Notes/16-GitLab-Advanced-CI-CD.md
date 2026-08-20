# 13-GitLab — 16 GitLab Advanced CI/CD

> Production-oriented guide to advanced GitLab CI/CD design, including DAG pipelines, `needs`, rules, workflow control, reusable templates, child and multi-project pipelines, dynamic pipelines, matrix jobs, parallelism, artifacts, caching, environments, deployment strategies, GitOps integration, security gates, performance optimization, release orchestration, failure recovery, and senior DevOps interview scenarios.

---

## 1. What Is Advanced CI/CD?

Advanced CI/CD focuses on making pipelines:

```text
Fast
Reliable
Secure
Reusable
Scalable
Observable
Maintainable
```

---

## 2. Basic Pipeline Limitation

A simple pipeline may execute:

```text
build
 ↓
test
 ↓
scan
 ↓
deploy
```

Each stage may wait for every previous job even when there are no real dependencies.

---

## 3. DAG Pipeline

A Directed Acyclic Graph allows jobs to run according to dependencies instead of only stage order.

```text
Build
 ├── Unit Test
 ├── SAST
 └── SCA
       │
       ▼
   Package
       │
       ▼
   Deploy
```

---

## 4. Why DAG Pipelines?

Benefits:

- shorter pipeline duration
- better parallelism
- explicit dependencies
- faster developer feedback

---

## 5. `needs`

`needs` defines a job dependency.

Example:

```yaml
test:
  stage: test
  needs:
    - build
```

The job can start as soon as its dependency is ready.

---

## 6. Stage Order vs `needs`

Traditional:

```text
All build jobs
 ↓
All test jobs
 ↓
All security jobs
```

DAG:

```text
build-api → test-api → deploy-api
build-ui  → test-ui  → deploy-ui
```

---

## 7. `needs: []`

A job with an empty `needs` list can start without waiting for earlier-stage jobs.

Use this carefully.

---

## 8. DAG Example

```yaml
build:
  stage: build
  script:
    - ./build.sh

unit-test:
  stage: test
  needs:
    - build
  script:
    - ./test.sh

lint:
  stage: test
  needs: []
  script:
    - ./lint.sh
```

---

## 9. DAG Design Principle

Only create dependencies when there is an actual dependency.

Unnecessary dependencies slow the pipeline.

---

## 10. Pipeline Critical Path

Pipeline duration is strongly influenced by the longest dependency path.

```text
A → B → C → D
```

If unrelated jobs can run in parallel, remove unnecessary waits.

---

## 11. Pipeline Parallelism

Example:

```text
              ┌── Unit Test
Build ────────┼── SAST
              ├── SCA
              └── Lint
```

Then:

```text
All required checks
 ↓
Package
```

---

## 12. Rules

`rules` controls whether a job is created and how it behaves.

Example:

```yaml
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
```

---

## 13. Rules vs Only/Except

Modern GitLab pipelines generally favor `rules` for expressive conditions.

They can evaluate:

```text
branch
tag
pipeline source
variables
file changes
```

---

## 14. Merge Request Rules

Example:

```yaml
rules:
  - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

This allows MR-specific pipelines.

---

## 15. Branch Rules

Example:

```yaml
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
```

Run a job only for the main branch.

---

## 16. Tag Rules

Example:

```yaml
rules:
  - if: '$CI_COMMIT_TAG'
```

Useful for release pipelines.

---

## 17. Scheduled Pipeline Rules

Example:

```yaml
rules:
  - if: '$CI_PIPELINE_SOURCE == "schedule"'
```

Useful for:

```text
security scans
dependency checks
maintenance
```

---

## 18. Manual Job Rule

Example:

```yaml
rules:
  - if: '$CI_COMMIT_BRANCH == "main"'
    when: manual
```

Manual production approval can be implemented with protected environments and appropriate permissions.

---

## 19. `when`

Common values include:

```text
on_success
manual
delayed
always
never
```

Use deliberately.

---

## 20. `allow_failure`

This controls whether a job failure blocks the pipeline.

Use carefully for security and production jobs.

---

## 21. Mandatory Security Job

Avoid:

```yaml
security_scan:
  allow_failure: true
```

for a mandatory security gate unless policy explicitly requires it.

---

## 22. `workflow: rules`

`workflow: rules` controls whether the entire pipeline is created.

This is different from job-level `rules`.

---

## 23. Workflow Control

Example:

```yaml
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH'
```

---

## 24. Duplicate Pipeline Problem

A push to a branch with an open MR can potentially trigger multiple pipeline types depending on configuration.

Use workflow rules to control this.

---

## 25. MR Pipeline Strategy

A common strategy:

```text
MR
 ↓
Validation pipeline

main
 ↓
Build/package/deploy pipeline
```

Avoid redundant pipelines.

---

## 26. Branch Pipeline Strategy

For feature branches:

```text
lint
test
security
```

For main:

```text
full validation
build
publish
deploy
```

---

## 27. Release Pipeline

Tags can trigger:

```text
release
 ↓
production deployment
```

This is useful when releases are explicitly versioned.

---

## 28. Semantic Versioning

Example:

```text
v1.4.2
```

Typically:

```text
MAJOR.MINOR.PATCH
```

Use a consistent release strategy.

---

## 29. Commit SHA Versioning

Example:

```text
user-service:4f9c2e1
```

This provides direct source traceability.

---

## 30. Hybrid Versioning

A production system may use:

```text
release tag
+
commit SHA
+
image digest
```

for human readability and immutable traceability.

---

## 31. Reusable Jobs

Use YAML anchors, hidden jobs, `extends`, or included templates where appropriate.

Goal:

```text
define once
reuse safely
```

---

## 32. Hidden Job Template

Example:

```yaml
.test-template:
  script:
    - ./run-tests.sh
```

Jobs can extend this template.

---

## 33. `extends`

Example:

```yaml
unit-test:
  extends: .test-template
```

Useful for standardized behavior.

---

## 34. YAML Anchors

YAML anchors can reuse configuration.

Use them for simple local reuse.

For larger organizations, centrally managed templates may be easier to govern.

---

## 35. Include

GitLab can include CI configuration from other files/projects depending on configuration.

Example:

```yaml
include:
  - local: ci/security.yml
```

---

## 36. Local Includes

Keep reusable configuration in the same repository.

Useful for:

```text
security.yml
docker.yml
terraform.yml
deploy.yml
```

---

## 37. Project Includes

A central platform repository can provide shared templates to multiple projects.

---

## 38. Central CI Templates

Example:

```text
platform-ci/
├── security.yml
├── docker.yml
├── terraform.yml
└── kubernetes.yml
```

Applications consume approved templates.

---

## 39. Template Governance

Central templates should be:

```text
versioned
tested
reviewed
documented
```

---

## 40. Template Version Pinning

Avoid unexpectedly changing every project when a shared template changes.

Pin approved versions when appropriate.

---

## 41. CI Template Supply Chain

A malicious shared template can compromise many pipelines.

Protect:

```text
template repository
release process
maintainer access
```

---

## 42. Componentization

Separate pipeline concerns:

```text
build
test
security
package
deploy
```

This makes complex pipelines easier to maintain.

---

## 43. Parent-Child Pipelines

A parent pipeline can trigger a child pipeline.

Concept:

```text
Parent
 ├── frontend child
 ├── backend child
 └── infrastructure child
```

---

## 44. Why Child Pipelines?

Useful for:

```text
monorepos
large applications
team ownership
microservices
```

---

## 45. Parent Pipeline

The parent determines which child pipelines are required.

Example:

```text
Change frontend
 ↓
frontend pipeline

Change backend
 ↓
backend pipeline
```

---

## 46. Dynamic Child Pipeline

A job can generate a CI configuration dynamically and trigger it.

Useful when pipeline structure depends on repository contents.

---

## 47. Dynamic Pipeline Risk

Generated configuration is executable code.

Review:

```text
generation logic
inputs
permissions
secrets
```

---

## 48. Multi-Project Pipelines

A GitLab project can trigger another project's pipeline.

Example:

```text
Application
 ↓
GitOps repository
 ↓
Deployment pipeline
```

---

## 49. Multi-Project Pipeline Use Case

Example:

```text
Terraform project
 ↓
Infrastructure complete
 ↓
Application project
```

Trigger only after required infrastructure changes succeed.

---

## 50. Pipeline Dependencies

Cross-project dependencies should be explicit.

Document:

```text
producer
consumer
artifact
trigger
```

---

## 51. Microservices Pipeline Architecture

```text
GitLab
 │
 ├── User
 ├── Cart
 ├── Orders
 ├── Payment
 └── Inventory
```

Each service can have an independent pipeline.

---

## 52. Monorepo Challenge

A monorepo may contain:

```text
services/
├── user
├── cart
├── orders
└── payment
```

A change to one service should not always rebuild everything.

---

## 53. Changes-Based Pipeline

Use `rules:changes` to detect affected paths.

Concept:

```text
services/user/**
```

triggers the user pipeline.

---

## 54. Changes-Based Optimization

Example:

```yaml
rules:
  - changes:
      - services/user/**/*
```

This reduces unnecessary jobs.

---

## 55. Changes-Based Risk

Shared files can affect multiple services.

Examples:

```text
Docker base
shared library
CI template
Terraform module
```

Include dependent pipelines where necessary.

---

## 56. Dependency Graph for Monorepo

Maintain an explicit understanding:

```text
shared library
 ↓
user
cart
orders
```

A shared change may require broader testing.

---

## 57. Matrix Jobs

Matrix jobs allow the same job to run across combinations of variables.

Useful for:

```text
Python versions
Node versions
AWS regions
service variants
```

---

## 58. Matrix Testing

Concept:

```text
Python 3.11
Python 3.12
Python 3.13
```

with the same test suite.

---

## 59. Matrix Risk

Matrix size grows quickly.

Example:

```text
3 Python versions
×
4 OS variants
×
2 databases
=
24 jobs
```

Use only meaningful combinations.

---

## 60. Parallel Jobs

Use parallel execution for independent tasks.

Examples:

```text
unit tests
lint
SAST
SCA
```

---

## 61. Parallelism vs Concurrency

### Parallelism

Multiple jobs execute at the same time.

### Concurrency control

Limits simultaneous execution of competing jobs.

---

## 62. Resource Groups

Resource groups can serialize deployments to the same environment/resource.

Concept:

```text
prod deployment A
 ↓
prod deployment B waits
```

---

## 63. Deployment Race Condition

Without serialization:

```text
Pipeline A → Prod
Pipeline B → Prod
```

Both can modify the same environment.

---

## 64. Environment Locking

Use protected environments and resource/concurrency controls to prevent conflicting production deployments.

---

## 65. Cancel Older Pipelines

For feature branches, canceling obsolete pipelines can save resources.

But production/release pipelines may require different behavior.

---

## 66. Interruptible Jobs

Jobs that can safely be canceled may be configured as interruptible.

Do not use this for jobs where cancellation can leave infrastructure inconsistent.

---

## 67. Terraform Concurrency

Never allow multiple uncontrolled Terraform applies against the same state.

Serialize production infrastructure changes.

---

## 68. Terraform Pipeline Pattern

```text
fmt
 ↓
validate
 ↓
security scan
 ↓
plan
 ↓
approval
 ↓
apply
```

---

## 69. Plan Artifact

A Terraform plan can be stored as a controlled artifact.

Concept:

```text
plan
 ↓
review
 ↓
approved apply
```

Ensure the plan is handled securely.

---

## 70. Infrastructure Pipeline Approval

Production Terraform apply should require:

```text
successful plan
+
review
+
authorized execution
```

---

## 71. GitLab Environment

An environment represents a deployment target such as:

```text
dev
stage
production
```

---

## 72. Environment Variables

Environment-specific variables allow:

```text
DEV_ACCOUNT
STAGE_ACCOUNT
PROD_ACCOUNT
```

with appropriate protection.

---

## 73. Protected Environment

Production deployment should be restricted to authorized users/groups.

---

## 74. Environment URL

A GitLab environment can expose a deployment URL.

Example:

```text
https://app.example.com
```

This improves navigation from CI to the deployed service.

---

## 75. Review Apps

Review Apps can create temporary environments for merge requests.

Concept:

```text
MR
 ↓
Build
 ↓
Deploy temporary environment
 ↓
Review
 ↓
Destroy
```

---

## 76. Review App Security

Never expose production secrets to review environments.

Use isolated:

```text
database
credentials
namespace
cloud role
```

---

## 77. Ephemeral Environments

Ephemeral environments are useful for:

```text
integration testing
DAST
manual QA
feature review
```

---

## 78. Environment Cleanup

Temporary environments should have automatic cleanup.

Otherwise:

```text
MRs
 ↓
hundreds of environments
 ↓
cost/resource growth
```

---

## 79. Deployment Strategies

Common strategies:

```text
Rolling
Blue/Green
Canary
Recreate
```

---

## 80. Rolling Deployment

Gradually replaces old Pods.

Good default for many Kubernetes workloads.

---

## 81. Blue/Green

Two environments:

```text
Blue = current
Green = new
```

Traffic switches after validation.

---

## 82. Canary

A small percentage receives the new version first.

Example:

```text
95% old
5% new
```

Then increase traffic based on health.

---

## 83. Progressive Delivery

```text
Deploy
 ↓
Observe
 ↓
Increase traffic
 ↓
Observe
 ↓
Complete
```

---

## 84. GitLab + ArgoCD Deployment Strategy

GitLab:

```text
build/test/security
```

GitOps:

```text
desired deployment state
```

ArgoCD:

```text
reconciliation
```

Progressive delivery tooling:

```text
traffic/rollout analysis
```

---

## 85. Deployment Verification

After deployment:

```text
Pod status
Deployment status
Service endpoints
Ingress
Smoke test
Metrics
Logs
```

---

## 86. Automatic Rollback

Automatic rollback can be useful but risky.

Define:

```text
rollback signal
threshold
scope
recovery process
```

before enabling it.

---

## 87. Smoke Tests

Example:

```bash
curl -f https://service.example.com/health
```

Follow with a critical API test.

---

## 88. API Integration Test

Example:

```text
POST /login
GET /products
POST /cart
```

Use non-production test data.

---

## 89. Contract Testing

Microservices can use contract tests to validate:

```text
consumer expectations
provider API
```

before deployment.

---

## 90. Pipeline Quality

A strong pipeline tests:

```text
code
artifact
infrastructure
deployment
runtime
```

---

## 91. Build Once

Avoid:

```text
Build Dev
Build Stage
Build Prod
```

Prefer:

```text
Build once
 ↓
Promote same artifact
```

---

## 92. Artifact Promotion

```text
Build
 ↓
Test
 ↓
Scan
 ↓
ECR
 ↓
Dev
 ↓
Stage
 ↓
Prod
```

---

## 93. Artifact Immutability

Use:

```text
digest
commit SHA
release version
```

to identify artifacts.

---

## 94. GitOps Promotion

Example:

```text
Dev values
 ↓
Stage values
 ↓
Prod MR
```

The image digest remains unchanged.

---

## 95. Release Orchestration

A release may coordinate:

```text
application
database
infrastructure
configuration
security approval
```

---

## 96. Release Pipeline

Concept:

```text
Validate
 ↓
Build
 ↓
Scan
 ↓
Publish
 ↓
Promote
 ↓
Deploy
 ↓
Verify
```

---

## 97. Deployment Freeze

During a freeze:

```text
normal production deployment
        ↓
       blocked
```

Emergency deployment follows a separate controlled process.

---

## 98. Scheduled Security Pipeline

Run periodic:

```text
dependency scans
container scans
IaC scans
secret scans
```

because new vulnerabilities can appear without source changes.

---

## 99. Dependency Monitoring

A package can become vulnerable after deployment.

Therefore:

```text
daily/periodic scanning
```

is valuable.

---

## 100. Container Rescanning

Existing images should be rescanned periodically because vulnerability databases change.

---

## 101. CI Failure Classification

Differentiate:

```text
code failure
test failure
security failure
infrastructure failure
runner failure
external dependency failure
```

---

## 102. Retry Strategy

Retry only transient failures.

Examples:

```text
temporary network error
registry timeout
external API timeout
```

Do not blindly retry:

```text
unit test failure
security violation
invalid Terraform
```

---

## 103. `retry`

GitLab can retry certain failed jobs.

Configure retry rules carefully so genuine failures are not hidden.

---

## 104. Idempotency

Deployment scripts should be safe to run repeatedly.

Example:

```text
ensure resource exists
```

rather than:

```text
create resource blindly
```

---

## 105. Idempotent Infrastructure

Terraform is designed around desired state.

Use Terraform rather than ad-hoc shell scripts for resources it owns.

---

## 106. Idempotent Kubernetes

Kubernetes manifests applied through ArgoCD are declarative.

The desired state can be repeatedly reconciled.

---

## 107. Pipeline Failure Recovery

A failed pipeline should leave systems in a known state.

Design:

```text
transaction-like stages
clear ownership
rollback path
```

---

## 108. Partial Deployment

If service A succeeds but service B fails:

```text
A = new
B = old
```

Determine whether this mixed state is safe.

---

## 109. Microservice Independence

Independent deployment is an advantage of microservices.

But shared contracts must remain compatible.

---

## 110. Database Compatibility

Deployments should avoid breaking:

```text
old application
new application
```

during rolling upgrades.

---

## 111. Expand and Contract

Database migration pattern:

```text
Expand
 ↓
Deploy compatible app
 ↓
Migrate
 ↓
Contract
```

---

## 112. Pipeline Notifications

Notify teams on:

```text
failed deployment
security gate
production success
rollback
```

Avoid excessive notification noise.

---

## 113. ChatOps

Pipeline systems can integrate with collaboration platforms for:

```text
deployment notifications
approval
incident workflows
```

Use secure integration credentials.

---

## 114. Deployment Audit

Record:

```text
who
what
when
which artifact
which environment
which revision
```

---

## 115. Release Notes

Generate release notes from:

```text
commits
merge requests
issues
security changes
```

---

## 116. Semantic Release

Automated versioning can derive versions from commit conventions.

Use only when the organization follows the required commit discipline.

---

## 117. Git Tags

A release tag can represent:

```text
approved source revision
```

and trigger release pipelines.

---

## 118. Release Candidate

A release candidate can move through:

```text
test
stage
validation
production approval
```

before final promotion.

---

## 119. Canary Approval

For high-risk releases:

```text
Deploy 5%
 ↓
Observe
 ↓
Approve
 ↓
50%
 ↓
Observe
 ↓
100%
```

---

## 120. Automated Canary Analysis

Evaluate:

```text
error rate
latency
availability
business metrics
```

before increasing traffic.

---

## 121. Pipeline Metrics

Track:

```text
pipeline duration
queue time
failure rate
deployment frequency
rollback rate
```

---

## 122. DORA Metrics

Common delivery metrics include:

```text
Deployment Frequency
Lead Time for Changes
Change Failure Rate
Time to Restore Service
```

Use them to improve engineering flow.

---

## 123. Pipeline Duration

Break down:

```text
queue
build
test
security
package
deploy
verification
```

to find bottlenecks.

---

## 124. Queue Time

Long queue time can indicate:

```text
insufficient runners
runner tags too restrictive
resource shortage
```

---

## 125. Runner Capacity

Scale runners based on:

```text
job concurrency
CPU
memory
queue time
```

---

## 126. Kubernetes Runner Autoscaling

Kubernetes-based runners can create job execution Pods dynamically.

This can improve elasticity.

---

## 127. Runner Autoscaling Security

Dynamic runners must still use:

```text
trusted images
network controls
least privilege
ephemeral lifecycle
```

---

## 128. Pipeline Caching

Cache:

```text
dependencies
build outputs
```

when safe.

---

## 129. Cache vs Artifact

### Cache

Optimization for future jobs/pipelines.

### Artifact

Output intentionally passed/stored between jobs.

---

## 130. Cache Security

Never use cache as a secure secret store.

---

## 131. Artifact Dependency

A job can consume an upstream artifact when there is an explicit dependency.

---

## 132. Artifact Size

Large artifacts slow pipelines.

Store large immutable binaries in an appropriate artifact/object repository where practical.

---

## 133. Dependency Proxy

Use a dependency proxy where it improves:

```text
pull performance
availability
external dependency control
```

---

## 134. BuildKit

Modern container builds can use BuildKit capabilities for:

```text
efficient builds
caching
secure build secrets
multi-platform builds
```

---

## 135. Build Secret

A build secret should be mounted only for the build step and should not become part of the final image.

---

## 136. Multi-Platform Build

Build images for:

```text
amd64
arm64
```

when the deployment platform requires both.

---

## 137. EKS Architecture Awareness

If nodes use:

```text
amd64
```

ensure the image supports that architecture.

---

## 138. Image Manifest

A multi-platform image can reference architecture-specific images through a manifest.

---

## 139. Build Matrix for Architecture

Example:

```text
amd64 build
arm64 build
      ↓
manifest
      ↓
registry
```

---

## 140. Pipeline Artifacts for Terraform

Possible artifacts:

```text
terraform plan
security report
policy report
```

Protect sensitive plan files.

---

## 141. Terraform Plan Security

Terraform plan can reveal:

```text
resource configuration
sensitive metadata
infrastructure details
```

Restrict access.

---

## 142. Terraform Apply Authorization

Production apply should be limited to:

```text
approved branch
approved environment
authorized user
trusted runner
```

---

## 143. Infrastructure Pipeline Separation

Separate:

```text
plan
approval
apply
```

for high-risk environments.

---

## 144. GitLab + Terraform + OIDC

```text
GitLab Job
 ↓
OIDC
 ↓
AWS STS
 ↓
Terraform
 ↓
AWS
```

No static AWS keys are required.

---

## 145. GitLab + ArgoCD + OIDC

A GitOps deployment can avoid giving GitLab direct long-lived cluster credentials:

```text
GitLab
 ↓
GitOps commit
 ↓
ArgoCD
 ↓
EKS
```

---

## 146. Pipeline Security Boundary

Treat:

```text
GitLab
runner
GitOps
ArgoCD
AWS
EKS
```

as separate trust boundaries.

---

## 147. Pipeline Approval Boundary

Production should have an explicit authorization point.

Example:

```text
Automated checks
 ↓
Human approval
 ↓
Production
```

---

## 148. Automatic Deployment in Dev

Development can use:

```text
commit
 ↓
CI
 ↓
GitOps
 ↓
ArgoCD auto-sync
```

to provide fast feedback.

---

## 149. Automatic Deployment in Production

It can be used when:

```text
tests are strong
security gates are strong
monitoring is mature
rollback is tested
business risk is acceptable
```

---

## 150. Manual Production Deployment

Manual approval is not a substitute for:

```text
testing
security
observability
rollback
```

---

## 151. Production Deployment Checklist

```text
[ ] CI passed
[ ] Security passed
[ ] Image approved
[ ] Digest recorded
[ ] GitOps approved
[ ] Change window valid
[ ] Monitoring ready
[ ] Rollback known
[ ] Owner available
```

---

## 152. Deployment Verification Checklist

```text
[ ] ArgoCD Synced
[ ] ArgoCD Healthy
[ ] Pods Ready
[ ] Deployment Available
[ ] Ingress healthy
[ ] Smoke test passed
[ ] Error rate normal
[ ] Latency normal
[ ] Logs normal
```

---

## 153. Pipeline Observability

Monitor the pipeline itself.

Important:

```text
failed jobs
queue time
duration
runner utilization
deployment failures
security failures
```

---

## 154. Pipeline Logs

Logs should be:

```text
useful
structured
secret-free
retained appropriately
```

---

## 155. Structured CI Logs

Example:

```text
INFO build_start commit=4f9c2e1
INFO image_push digest=sha256:...
INFO promotion environment=stage
```

Avoid logging credentials.

---

## 156. Deployment Metadata

Attach:

```text
commit
pipeline
image digest
environment
```

to deployment metadata where practical.

---

## 157. Production Correlation

Example:

```text
GitLab pipeline 1234
 ↓
GitOps commit abc123
 ↓
ArgoCD sync
 ↓
EKS rollout
 ↓
Prometheus alert
```

This enables fast troubleshooting.

---

## 158. Pipeline Incident

If CI suddenly becomes slow:

```text
Check queue
 ↓
Check runners
 ↓
Check registry
 ↓
Check dependencies
 ↓
Check recent pipeline changes
```

---

## 159. Runner Incident

If many jobs fail:

```text
Check runner health
 ↓
executor
 ↓
network
 ↓
permissions
 ↓
container runtime
```

---

## 160. Registry Incident

If image pushes fail:

```text
Check ECR/GitLab registry
 ↓
credentials
 ↓
network
 ↓
repository
 ↓
permissions
```

---

## 161. GitOps Incident

If promotion succeeds but deployment does not:

```text
Check GitOps commit
 ↓
ArgoCD source
 ↓
ArgoCD sync
 ↓
Kubernetes events
```

---

## 162. Kubernetes Incident

If deployment syncs but service is unhealthy:

```text
Pods
 ↓
events
 ↓
probes
 ↓
resources
 ↓
application logs
 ↓
dependencies
```

---

## 163. Retry External APIs

Use controlled retries for:

```text
registry
artifact repository
cloud APIs
Git API
```

with exponential backoff where appropriate.

---

## 164. Retry Storm

Too many retries can increase load.

Use:

```text
bounded retries
backoff
jitter
```

---

## 165. Timeout Design

Every external operation should have an appropriate timeout.

Avoid indefinite waits.

---

## 166. Pipeline Timeouts

Configure reasonable job/pipeline timeouts.

This prevents stuck jobs consuming runner capacity indefinitely.

---

## 167. Manual Intervention

Manual jobs should be used for meaningful decisions:

```text
production approval
security exception
release promotion
```

not to compensate for poor automation.

---

## 168. DR Pipeline

Disaster recovery may require:

```text
Terraform
 ↓
EKS
 ↓
ArgoCD
 ↓
Applications
```

with tested automation.

---

## 169. Backup CI Configuration

Keep:

```text
.gitlab-ci.yml
templates
scripts
Terraform
GitOps
```

in version control.

---

## 170. CI/CD Disaster Recovery

Ensure the organization can recreate:

```text
runners
credentials
repositories
templates
deployment configuration
```

after a major failure.

---

## 171. GitLab Backup Considerations

GitLab itself contains critical source and CI configuration.

Use the organization's supported backup/DR strategy for GitLab infrastructure.

---

## 172. GitLab DR

Document:

```text
RPO
RTO
backup
restore
validation
```

---

## 173. RPO

Recovery Point Objective:

```text
How much data can be lost?
```

---

## 174. RTO

Recovery Time Objective:

```text
How quickly must service be restored?
```

---

## 175. Pipeline DR Test

Do not assume backups work.

Test:

```text
restore
pipeline execution
artifact access
deployment
```

---

## 176. Production Change Management

For high-risk changes:

```text
MR
 ↓
Review
 ↓
Security
 ↓
Approval
 ↓
Deployment
 ↓
Verification
```

---

## 177. Change Failure Rate

Track how often deployments cause:

```text
rollback
incident
hotfix
```

---

## 178. Lead Time

Measure:

```text
first code change
 ↓
production deployment
```

This identifies delivery bottlenecks.

---

## 179. Deployment Frequency

Measure successful production deployments over time.

Do not optimize frequency at the expense of reliability.

---

## 180. Time to Restore

Measure:

```text
incident
 ↓
service restored
```

---

## 181. CI/CD Maturity

### Level 1

```text
Manual builds
```

### Level 2

```text
Basic CI
```

### Level 3

```text
Automated CI/CD
```

### Level 4

```text
GitOps + security + observability
```

### Level 5

```text
Progressive delivery + policy + automated recovery
```

---

## 182. Advanced CI/CD Architecture

```text
Developer
   │
   ▼
 GitLab MR
   │
   ▼
Workflow Rules
   │
   ▼
DAG Pipeline
   │
   ├── Test
   ├── SAST
   ├── SCA
   ├── Secrets
   └── IaC
          │
          ▼
        Build
          │
          ▼
       Trivy/SBOM
          │
          ▼
         ECR
          │
          ▼
     GitOps Update
          │
          ▼
        ArgoCD
          │
          ▼
         EKS
          │
          ▼
      Verification
          │
          ▼
   Prometheus/Grafana/ELK
```

---

## 183. Advanced Pipeline Design Principles

```text
Fast feedback
+
Explicit dependencies
+
Immutable artifacts
+
Least privilege
+
Security gates
+
Controlled production
+
Observability
+
Rollback
```

---

## 184. Senior Interview — How Do You Make GitLab Pipelines Faster?

> I first measure queue time and job duration, then use DAG dependencies and `needs` to parallelize independent jobs, cache safe dependencies, use changes-based execution for monorepos, reuse templates, and optimize expensive security/build jobs without removing mandatory controls.

---

## 185. Senior Interview — What Is a DAG Pipeline?

> A DAG pipeline uses explicit job dependencies instead of forcing all jobs to wait for the entire previous stage. With `needs`, independent jobs can execute in parallel, reducing the pipeline's critical path.

---

## 186. Senior Interview — Why Use `workflow: rules`?

> `workflow: rules` controls whether the pipeline itself is created. I use it to avoid duplicate pipelines, for example when both push and merge-request events could otherwise create redundant pipelines.

---

## 187. Senior Interview — How Do You Handle a Monorepo?

> I use path-based rules to identify affected services, run only relevant jobs, and use child pipelines where the repository becomes large. I also account for shared libraries and common CI changes so dependent services are not accidentally skipped.

---

## 188. Senior Interview — Why Use Child Pipelines?

> Child pipelines allow a large pipeline to be decomposed into independently managed workflows, such as frontend, backend and infrastructure pipelines. This improves maintainability and can reduce unnecessary execution.

---

## 189. Senior Interview — What Is a Multi-Project Pipeline?

> It allows one GitLab project to trigger another project's pipeline. I use it when responsibilities are separated, such as application CI triggering a controlled GitOps or deployment workflow.

---

## 190. Senior Interview — How Do You Prevent Two Production Deployments Running Together?

> I use protected environments and concurrency controls such as resource groups so deployments targeting the same production environment are serialized.

---

## 191. Senior Interview — How Do You Handle Pipeline Failures?

> I classify the failure first. Code/test failures should be fixed rather than retried blindly. Transient infrastructure or network failures can use bounded retries with backoff. Production deployment failures require state validation and a rollback/recovery procedure.

---

## 192. Senior Interview — What Is Build Once, Deploy Many?

> It means building one immutable artifact, validating it, and promoting the same artifact through Dev, Stage and Production. This prevents environment-specific rebuilds from producing different binaries.

---

## 193. Senior Interview — How Do You Secure a Production Pipeline?

> I use protected branches/environments, MR approvals, security gates, protected variables, OIDC for AWS, trusted runners, least-privilege roles, immutable artifacts, controlled deployment permissions and complete auditability.

---

## 194. Senior Interview — How Do You Handle Terraform in an Advanced Pipeline?

> I run formatting, validation, security checks and plan first. Production apply is separated and protected, uses a trusted runner and short-lived AWS credentials, and is serialized to prevent concurrent state changes.

---

## 195. Senior Interview — How Do You Integrate GitLab With ArgoCD?

> GitLab performs CI and publishes the immutable image. After security gates pass, CI updates the GitOps repository with the image digest. ArgoCD watches that repository and reconciles the desired state into EKS. GitLab does not need broad direct Kubernetes deployment credentials.

---

## 196. Senior Interview — How Would You Design a Microservices CI/CD Platform?

> I would use independent service pipelines, shared governed CI templates, changes-based execution, container security scanning, immutable ECR artifacts, GitOps-based deployment, ArgoCD, protected production environments, and centralized observability. Shared libraries and platform changes would trigger appropriate dependent validation.

---

## 197. Senior Interview — How Do You Implement Zero-Downtime Deployment?

> I use Kubernetes rolling deployments with readiness/startup probes, appropriate resource settings, PodDisruptionBudgets where needed, backward-compatible application/database changes, and post-deployment health validation. For higher-risk releases I can use canary or blue-green strategies.

---

## 198. Senior Interview — How Do You Roll Back?

> In a GitOps model I normally revert the GitOps change to a known-good image digest and let ArgoCD reconcile it. The rollback is validated through application health, smoke tests and monitoring.

---

## 199. Senior Interview — How Do You Handle Database Changes?

> I use backward-compatible expand-and-contract migrations. The schema is expanded first, the application is deployed in a compatible state, data is migrated, and obsolete schema is removed later. This avoids breaking rolling deployments.

---

## 200. Senior Interview — How Do You Prevent Secrets From Reaching Pipelines?

> I use secret detection, protected and masked variables, environment scoping, external secret management and OIDC. I also prevent untrusted fork pipelines from accessing production credentials.

---

## 201. Senior Interview — What Would You Monitor in CI/CD?

> I monitor pipeline duration, queue time, failure rate, runner utilization, security gate failures, deployment frequency, change failure rate, rollback frequency and time to restore service.

---

## 202. Senior Interview — How Do You Optimize Security Without Making CI Too Slow?

> I parallelize independent security checks, use safe dependency caching, reuse templates, run changes-based checks where appropriate, and reserve deeper DAST or scheduled scans for suitable environments. I never remove mandatory production security controls just to reduce pipeline time.

---

## 203. Senior Interview — What Makes a Pipeline Production-Grade?

> A production-grade pipeline is reproducible, secure, observable, idempotent, fast enough for developers, protected against unauthorized changes, uses immutable artifacts, has controlled production access, and provides a tested rollback and disaster-recovery path.

---

## 204. Final Advanced CI/CD Checklist

```text
[ ] workflow rules
[ ] job rules
[ ] DAG / needs
[ ] parallel jobs
[ ] changes-based execution
[ ] reusable templates
[ ] template versioning
[ ] child pipelines
[ ] multi-project pipelines
[ ] matrix jobs where useful
[ ] protected environments
[ ] deployment serialization
[ ] immutable artifacts
[ ] security gates
[ ] OIDC
[ ] trusted runners
[ ] artifact controls
[ ] cache controls
[ ] GitOps
[ ] smoke tests
[ ] rollback
[ ] monitoring
[ ] DORA metrics
[ ] DR
```

---

## 205. Complete Production Pipeline

```text
Developer
   │
   ▼
GitLab Merge Request
   │
   ├── CODEOWNERS
   ├── Approval
   └── Workflow Rules
           │
           ▼
       DAG Pipeline
           │
    ┌──────┼─────────┐
    ▼      ▼         ▼
  Test   Security   Lint
    │      │
    │   ┌──┼──────────┐
    │   ▼  ▼          ▼
    │  SAST SCA     Secrets
    │
    └──────┬─────────┘
           ▼
         Build
           │
           ▼
     Container Scan
           │
           ▼
          SBOM
           │
           ▼
          ECR
           │
      Immutable Digest
           │
           ▼
      GitOps Update
           │
           ▼
        ArgoCD
           │
           ▼
          EKS
           │
      ┌────┼────┐
      ▼    ▼    ▼
    Pods  ALB  Services
           │
           ▼
       Smoke Tests
           │
           ▼
   Prometheus/Grafana/ELK
```

---

## 206. Final Mental Model

```text
                    ADVANCED CI/CD

                        GitLab
                           │
              ┌────────────┴────────────┐
              ▼                         ▼
          Governance                    CI
              │                         │
       ┌──────┼──────┐        ┌────────┼────────┐
       ▼      ▼      ▼        ▼        ▼        ▼
     Rules  Review  RBAC     Build   Test    Security
                                      │
                                      ▼
                                    ECR
                                      │
                                Immutable Artifact
                                      │
                                      ▼
                                   GitOps
                                      │
                                      ▼
                                   ArgoCD
                                      │
                                      ▼
                                     EKS
                                      │
                                      ▼
                                  Observe
                                      │
                                      ▼
                                  Improve
```

> **Core principle:** Advanced GitLab CI/CD is about designing a delivery system rather than writing a long YAML file. The system should minimize unnecessary waiting, make dependencies explicit, reuse governed components, protect credentials and production environments, promote immutable artifacts, integrate GitOps, and provide reliable recovery when something fails.

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
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md ✓
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `17-GitLab-Multi-Environment-Deployments.md`**
