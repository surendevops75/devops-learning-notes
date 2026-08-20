# GitLab CI/CD Fundamentals

> Production-oriented fundamentals for GitLab CI/CD. This file builds the mental model required before advanced `.gitlab-ci.yml` configuration, runners, variables, artifacts, Docker, AWS, EKS, Terraform, ArgoCD, DevSecOps, and production pipeline design.

---

## 1. What Is GitLab CI/CD?

GitLab CI/CD is GitLab's automation capability for continuously:

- validating code
- building applications
- running tests
- performing security checks
- packaging artifacts
- publishing container images
- deploying applications
- verifying deployments

Basic lifecycle:

```text
Code
 ↓
Commit
 ↓
Pipeline
 ↓
Build
 ↓
Test
 ↓
Security
 ↓
Package
 ↓
Deploy
 ↓
Verify
```

The pipeline configuration is commonly stored in:

```text
.gitlab-ci.yml
```

---

## 2. CI vs CD

### Continuous Integration

CI focuses on frequently integrating changes and automatically validating them.

```text
Commit
 ↓
Build
 ↓
Test
 ↓
Quality
 ↓
Security
```

### Continuous Delivery

Continuous Delivery keeps software ready for deployment.

```text
Validated Artifact
 ↓
Release
 ↓
Deployable Environment
```

### Continuous Deployment

Continuous Deployment automatically promotes approved changes to production according to policy.

```text
Commit
 ↓
CI
 ↓
Security
 ↓
Artifact
 ↓
Automatic Production Deployment
```

Not every organization uses full continuous deployment.

---

## 3. Why CI/CD Matters for DevOps

Without CI/CD:

```text
Developer
   ↓
Manual Build
   ↓
Manual Test
   ↓
Manual Copy
   ↓
Manual Deployment
```

This introduces:

- human error
- inconsistent environments
- poor traceability
- slow releases
- difficult rollback

With CI/CD:

```text
Git
 ↓
Automated Validation
 ↓
Immutable Artifact
 ↓
Controlled Deployment
 ↓
Automated Verification
```

---

## 4. GitLab CI/CD Architecture

Simplified architecture:

```text
                 GitLab
                   │
          ┌────────┴────────┐
          │                 │
       Repository        Pipeline
                              │
                              ▼
                         GitLab Runner
                              │
                              ▼
                         Execute Job
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                 Artifacts           Logs
```

GitLab coordinates the workflow while Runners execute jobs.

---

## 5. Pipeline

A pipeline is the complete automated workflow triggered by an event.

Examples of triggers:

- push
- Merge Request
- tag
- schedule
- API
- manual action
- parent pipeline
- other GitLab events

Example:

```text
Pipeline
├── validate
├── build
├── test
├── security
├── package
├── deploy
└── verify
```

---

## 6. Job

A job is one unit of execution.

Example:

```yaml
unit_test:
  stage: test
  script:
    - pytest
```

The Runner executes the job.

A job can have:

- image
- stage
- script
- variables
- artifacts
- cache
- rules
- dependencies
- needs
- environment
- retry/timeout behavior

---

## 7. Stage

A stage groups jobs logically.

Example:

```yaml
stages:
  - build
  - test
  - deploy
```

If jobs are not otherwise optimized with dependency relationships:

```text
build
  ↓
test
  ↓
deploy
```

Multiple jobs in the same stage can execute concurrently.

---

## 8. Basic Pipeline

```yaml
stages:
  - build
  - test

build:
  stage: build
  script:
    - echo "Build"

test:
  stage: test
  script:
    - echo "Test"
```

Flow:

```text
build
  ↓
test
```

---

## 9. Production Pipeline Stages

A DevSecOps pipeline may use:

```yaml
stages:
  - validate
  - build
  - test
  - security
  - package
  - deploy
  - verify
```

Conceptually:

```text
Validate
   ↓
Build
   ↓
Test
   ↓
Security
   ↓
Package
   ↓
Deploy
   ↓
Verify
```

The exact number of stages should reflect the workflow, not an arbitrary template.

---

## 10. Script

The `script` section defines commands executed by the Runner.

Example:

```yaml
build:
  script:
    - mvn clean package
```

For Python:

```yaml
test:
  script:
    - pip install -r requirements.txt
    - pytest
```

For Terraform:

```yaml
terraform_validate:
  script:
    - terraform init -backend=false
    - terraform validate
```

---

## 11. Before Script

`before_script` can define commands that execute before the job's main script.

Example:

```yaml
before_script:
  - echo "Preparing environment"
```

Use carefully.

Avoid placing unrelated global commands that make every job slower.

---

## 12. After Script

`after_script` can perform cleanup or post-job tasks.

Possible uses:

- cleanup temporary files
- collect diagnostic information
- finalize local state

Do not assume cleanup always means successful deployment.

The job's success/failure semantics should remain clear.

---

## 13. CI Configuration File

The main file is:

```text
.gitlab-ci.yml
```

It can define:

```text
stages
jobs
variables
workflow
rules
artifacts
cache
needs
dependencies
environments
images
services
```

Advanced syntax will be covered later.

---

## 14. YAML Indentation

GitLab CI uses YAML.

Correct:

```yaml
build:
  stage: build
  script:
    - echo "Build"
```

Incorrect indentation can cause pipeline configuration errors.

Always validate CI configuration before relying on it.

---

## 15. Pipeline Trigger — Push

A push can trigger a pipeline.

```text
git push
   ↓
GitLab
   ↓
Pipeline
```

This is the most common basic trigger.

---

## 16. Pipeline Trigger — Merge Request

An MR can trigger validation.

```text
Merge Request
      ↓
CI
      ↓
Test
      ↓
Security
```

This provides feedback before merging into the target branch.

---

## 17. Pipeline Trigger — Tag

A tag can trigger release workflows.

```text
v1.2.0
  ↓
Release Pipeline
  ↓
Build
  ↓
Package
  ↓
Publish
```

Tags are useful for release identity.

---

## 18. Pipeline Trigger — Schedule

Scheduled pipelines can run at defined times.

Examples:

- nightly tests
- security scans
- dependency checks
- infrastructure validation
- periodic cleanup

Do not use schedules for operations that require event-driven correctness when a more appropriate trigger exists.

---

## 19. Pipeline Trigger — Manual

A job can require a manual action.

Concept:

```text
Build
 ↓
Test
 ↓
Security
 ↓
Staging
 ↓
Manual Approval
 ↓
Production
```

Manual deployment should still require successful validation.

---

## 20. Pipeline Trigger — API

Pipelines can also be started through API automation.

Concept:

```text
External System
      ↓
GitLab API
      ↓
Pipeline
```

Secure the triggering identity and validate all supplied inputs.

---

## 21. Pipeline Source Matters

A pipeline can originate from different sources.

Examples include:

```text
push
merge_request_event
schedule
web
api
trigger
pipeline
```

This distinction is important for security.

For example:

```text
MR pipeline
```

should not automatically receive the same production permissions as:

```text
protected main pipeline
```

---

## 22. CI Job Execution

Conceptually:

```text
GitLab
  ↓
Find eligible job
  ↓
Select Runner
  ↓
Prepare environment
  ↓
Clone/fetch source
  ↓
Execute script
  ↓
Collect logs/artifacts
  ↓
Return status
```

---

## 23. Runner Selection

A pipeline job needs an eligible Runner.

Runner selection can depend on:

- runner availability
- tags
- protected status
- project/group association
- executor capability

Example:

```yaml
deploy_production:
  tags:
    - production
  script:
    - ./deploy.sh
```

Do not use a privileged runner unnecessarily.

---

## 24. Runner Executor and Job Image

A Docker-based runner can use a job image:

```yaml
image: python:3.12
```

Then:

```yaml
test:
  script:
    - python --version
    - pytest
```

This improves runtime consistency.

---

## 25. Job Services

Jobs may need supporting services.

Concept:

```text
Job
 ├── Application
 └── Service dependency
```

Examples can include databases or other test dependencies.

Production pipelines should avoid unnecessary service startup because it increases complexity and runtime.

---

## 26. Environment Variables

Example:

```yaml
variables:
  APP_ENV: "test"
```

Job:

```yaml
test:
  script:
    - echo "$APP_ENV"
```

Variables should separate configuration from code.

---

## 27. Variable Precedence

GitLab can provide variables at different scopes.

Conceptually:

```text
Instance/group
      ↓
Project
      ↓
Pipeline
      ↓
Job
```

The exact precedence depends on GitLab variable type/source.

Do not rely on memory for security-critical precedence; validate the actual configuration and documentation.

---

## 28. Secret Variables

Secrets should be:

- masked where supported
- protected where appropriate
- scoped to required environments
- short-lived when possible
- rotated
- excluded from logs

Never:

```yaml
AWS_SECRET_ACCESS_KEY: "secret"
```

inside the repository.

---

## 29. Workload Identity

For AWS integrations, prefer short-lived federation/OIDC where supported.

Concept:

```text
GitLab Job
    ↓
OIDC Identity
    ↓
AWS STS
    ↓
Short-lived Credentials
    ↓
ECR / AWS API
```

Benefits:

- no long-lived access keys
- reduced credential leakage risk
- clearer trust boundaries

---

## 30. Stages vs Dependencies

Traditional stage flow:

```text
build
 ↓
test
 ↓
deploy
```

can make independent jobs wait unnecessarily.

Dependency-aware execution can reduce pipeline time.

Example concept:

```text
build-a ───→ deploy-a
build-b ───→ deploy-b
```

rather than forcing:

```text
build-a
build-b
   ↓
all tests
   ↓
all deploys
```

The `needs` keyword is central to advanced dependency graphs.

---

## 31. Parallel Jobs

Independent jobs can run in parallel.

Example:

```text
          ┌── Unit Tests
Build ────┼── Lint
          └── Security
```

Benefits:

- shorter pipeline
- better runner utilization

Trade-off:

- more concurrent resource consumption

---

## 32. Pipeline Optimization

Measure:

```text
Queue time
Job startup
Build time
Test time
Security scan time
Artifact transfer
Deployment time
```

Do not optimize blindly.

Typical improvements:

- caching
- parallel jobs
- `needs`
- selective execution
- reusable images
- dependency optimization

---

## 33. Pipeline Failure

A failed job should provide:

```text
Job name
Stage
Exit code
Logs
Artifact/report
Commit SHA
Pipeline ID
Environment
```

Avoid generic:

```text
Pipeline failed
```

without identifying the failing boundary.

---

## 34. Exit Codes

Shell commands return exit codes.

Typically:

```text
0 = success
non-zero = failure
```

Example:

```bash
pytest
echo $?
```

A production pipeline should ensure critical commands fail the job when they fail.

Do not hide errors with careless shell constructs.

---

## 35. Command Failure Handling

Bad:

```bash
command_that_can_fail || true
```

This may convert a real failure into success.

Use only when failure is intentionally non-blocking and document why.

---

## 36. Security Gate Failure

Suppose Trivy finds a policy violation:

```text
Build
 ↓
Trivy
 ↓
FAIL
```

The pipeline should normally stop promotion.

Do not simply add:

```bash
trivy ... || true
```

to bypass a mandatory security gate.

---

## 37. Test Failure

Expected behavior:

```text
Build
 ↓
Tests
 ↓
FAIL
 ↓
No production deployment
```

A deployment job should not execute after a mandatory test stage fails.

---

## 38. Build Failure

Example:

```text
Maven Build
   ↓
Compilation failure
   ↓
Pipeline stops
```

Do not publish an invalid application artifact.

---

## 39. Artifact Flow

Example:

```text
Build
 ↓
Artifact
 ↓
Test / Package / Deploy
```

Artifact identity should be deterministic.

For containers:

```text
Source SHA
 ↓
Image
 ↓
Digest
```

---

## 40. Artifact vs Container Image

An artifact may be:

```text
app.jar
binary
test report
Terraform plan
security report
```

A container image is a deployable packaged application/runtime.

The image may be stored in:

```text
GitLab Container Registry
```

or:

```text
Amazon ECR
```

depending on architecture.

---

## 41. Docker Build in GitLab

Concept:

```text
GitLab Runner
     ↓
Docker Build
     ↓
Image
     ↓
Security Scan
     ↓
Registry
```

Production image should use immutable identification.

---

## 42. ECR Publishing

Example high-level flow:

```text
GitLab CI
   ↓
AWS Identity
   ↓
ECR Login
   ↓
Docker Build
   ↓
Docker Push
   ↓
Digest
```

Never print cloud credentials.

---

## 43. Security Pipeline

Your DevSecOps workflow can be:

```text
Build
 ↓
Unit Test
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Policy
 ↓
Artifact
```

Each security tool has a defined purpose.

---

## 44. SonarQube

SonarQube can provide code-quality and static-analysis findings.

Concept:

```text
Source
 ↓
Build/Test
 ↓
SonarQube
 ↓
Quality Gate
```

A failing quality gate can block promotion according to policy.

---

## 45. Trivy

Trivy can scan areas such as:

- container images
- filesystem
- dependencies
- configuration depending on usage

Concept:

```text
Image
 ↓
Trivy
 ↓
Vulnerability Findings
 ↓
Policy
```

---

## 46. Veracode

Veracode can provide application security analysis depending on the selected product/workflow.

Concept:

```text
Build Artifact
 ↓
Veracode
 ↓
Security Result
 ↓
Policy Gate
```

The exact scan type should match the application and security policy.

---

## 47. Deployment Job

A deployment job should be restricted.

Concept:

```text
Build
 ↓
Test
 ↓
Security
 ↓
Artifact
 ↓
Protected Deployment
```

Deployment credentials should not be available to every job.

---

## 48. GitOps Deployment

For your architecture:

```text
GitLab CI
   ↓
Build + Scan
   ↓
ECR
   ↓
Update GitOps
   ↓
ArgoCD
   ↓
EKS
```

GitLab is the CI/orchestration layer; ArgoCD is the Kubernetes reconciliation layer.

---

## 49. Deployment Verification

After ArgoCD sync:

```text
ArgoCD Sync
    ↓
Rollout
    ↓
Pod Ready
    ↓
Service Endpoints
    ↓
ALB Health
    ↓
Smoke Test
```

A successful GitLab job alone is not enough.

---

## 50. Smoke Tests

A smoke test validates critical functionality.

Example:

```bash
curl -f https://example.com/health
```

Potential checks:

- HTTP status
- response body
- application version
- authentication
- critical API

Do not make smoke tests excessively broad.

---

## 51. Health Checks

Deployment verification should distinguish:

```text
Infrastructure healthy
Application healthy
Business behavior healthy
```

All three may not be equivalent.

---

## 52. Deployment Failure

If deployment fails:

```text
Stop promotion
 ↓
Collect evidence
 ↓
Identify revision
 ↓
Check Kubernetes events/logs
 ↓
Check ArgoCD
 ↓
Determine rollback safety
 ↓
Rollback/revert if appropriate
 ↓
Verify
```

Do not blindly retry production deployment.

---

## 53. Manual Deployment Approval

Concept:

```text
CI
 ↓
Staging
 ↓
Validation
 ↓
Manual Approval
 ↓
Production
```

The approval should correspond to:

- exact artifact
- target environment
- security state
- deployment revision

---

## 54. Build Once, Promote Many

Recommended:

```text
Source
 ↓
Build once
 ↓
Image digest
 ↓
Dev
 ↓
Staging
 ↓
Production
```

Avoid rebuilding different artifacts per environment.

---

## 55. Immutable Artifact

An immutable image is identified by digest:

```text
sha256:abcdef...
```

This gives strong traceability:

```text
Commit SHA
 ↓
Image Digest
 ↓
Deployment
```

---

## 56. Mutable Tags

A tag such as:

```text
latest
```

can point to different image content over time.

For production:

```text
version tag + digest
```

is safer.

Even better, record the exact digest used by the deployment.

---

## 57. Environment Promotion

Typical:

```text
Development
    ↓
Staging
    ↓
Production
```

Promotion should preserve the same artifact.

Environment-specific configuration should be separate from the immutable application artifact.

---

## 58. CI/CD and Terraform

A common infrastructure pipeline:

```text
MR
 ↓
fmt
 ↓
validate
 ↓
plan
 ↓
review
 ↓
merge
 ↓
protected apply
```

Terraform state remains managed by Terraform; GitLab orchestrates the workflow.

---

## 59. CI/CD and Kubernetes

For direct Kubernetes workflows:

```text
Build
 ↓
ECR
 ↓
Helm/kubectl
 ↓
EKS
```

For GitOps:

```text
Build
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

When ArgoCD owns production, prefer the second model.

---

## 60. CI/CD and Docker

Typical pipeline:

```text
Checkout
 ↓
Build application
 ↓
Test
 ↓
Docker build
 ↓
Scan image
 ↓
Push registry
```

Use a trusted and reproducible builder environment.

---

## 61. CI/CD and Maven

Example:

```bash
mvn clean verify
```

Pipeline:

```text
Checkout
 ↓
Maven Build
 ↓
Unit Tests
 ↓
SonarQube
 ↓
Package
```

Do not skip tests merely to speed up a production pipeline unless an approved workflow explicitly separates them.

---

## 62. CI/CD and Python

Example:

```bash
python -m venv .venv
pip install -r requirements.txt
pytest
```

Production pipelines should use controlled dependency versions and preferably a standardized build image.

---

## 63. CI/CD and Node.js

Example:

```bash
npm ci
npm test
npm run build
```

`npm ci` is commonly preferred in CI for reproducible installation from the lockfile.

---

## 64. Dependency Caching

Common caches:

```text
Maven → .m2
npm → npm cache
Python → pip cache
Terraform → plugin cache
```

Caching reduces download time.

But cache should not be blindly trusted as a release artifact.

---

## 65. Cache Poisoning Risk

If untrusted jobs can write to a shared cache that trusted jobs later consume, malicious content can potentially cross trust boundaries.

Therefore:

- scope caches
- avoid sharing sensitive caches
- use trusted runners
- understand cache keys
- separate untrusted and trusted workflows

---

## 66. Pipeline Timeouts

Every production-sensitive job should have a reasonable timeout.

A hung command should not consume a runner indefinitely.

Examples of causes:

- network request
- Terraform operation
- Docker build
- Kubernetes API call
- external deployment system

---

## 67. Retry Strategy

Some jobs may be retryable:

- transient registry issue
- temporary API error
- infrastructure provisioning timeout

Do not retry:

- deterministic compile error
- failing unit test
- security policy violation
- authorization failure

Classify failures before retrying.

---

## 68. Retry Storm

Bad design:

```text
Job
 ↓ retry
API
 ↓ fail
Job
 ↓ retry
API
 ↓ fail
```

Across many jobs this can overload a dependency.

Use:

- bounded retries
- exponential backoff
- jitter
- request budgets
- circuit-breaking where appropriate

---

## 69. Pipeline Idempotency

A production pipeline should be safe to retry where possible.

Examples:

```text
Push same artifact
Update same GitOps value
Create existing resource
```

should not cause unexpected duplicate side effects.

---

## 70. GitOps Idempotency

Suppose CI runs twice.

Bad:

```text
Run 1 → commit
Run 2 → duplicate/noisy commit
```

Better:

```text
Run 1 → desired state changes → commit
Run 2 → desired state already correct → no unnecessary mutation
```

This keeps Git history clean.

---

## 71. Pipeline Concurrency

Two pipelines may deploy the same environment simultaneously.

Risk:

```text
Pipeline A
     ↓
Production

Pipeline B
     ↓
Production
```

Possible controls:

- resource/environment locking
- serialized deployment jobs
- protected environments
- deployment policies
- concurrency controls

---

## 72. Race Condition Example

Pipeline A:

```text
image v10
```

Pipeline B:

```text
image v11
```

If A finishes after B, production may accidentally return to v10.

Use ordering/concurrency controls and verify the final desired state.

---

## 73. Deployment Lock

Concept:

```text
Production
   │
   └── Deployment lock
          │
          ├── Pipeline A
          └── Pipeline B waits
```

This reduces concurrent production mutation.

---

## 74. Pipeline Cancellation

If a newer commit supersedes an older one, obsolete pipelines may be canceled according to policy.

Example:

```text
Commit A → pipeline running
Commit B → newer change
             ↓
        Cancel obsolete A
```

Be careful with cancellation of pipelines already performing production changes.

---

## 75. Auto-Cancel Redundant Pipelines

Useful for development branches where many commits arrive quickly.

Potential benefit:

```text
Commit 1 → canceled
Commit 2 → canceled
Commit 3 → latest validation
```

Production deployment pipelines require more conservative handling.

---

## 76. Pipeline Artifact Retention

Artifacts consume storage.

Define retention appropriate to:

- debugging
- compliance
- release evidence
- cost

Important production artifacts may require longer retention than temporary CI output.

---

## 77. Pipeline Logs

Logs should answer:

```text
What ran?
Where?
For which commit?
Which environment?
What failed?
How long did it take?
What artifact was produced?
```

Include safe correlation IDs.

Do not log secrets.

---

## 78. Correlation ID

Example:

```text
release_id=rel-2026-0012
commit=abc123
pipeline=4567
artifact=sha256:...
environment=staging
```

The same identifiers can be included in:

- CI logs
- GitOps commits
- deployment records
- application logs

---

## 79. Auditability

A production pipeline should allow investigators to answer:

```text
Who changed it?
What changed?
Which pipeline ran?
Which artifact was produced?
Who approved it?
Where was it deployed?
What revision is running?
```

This is essential during incidents and audits.

---

## 80. GitLab CI/CD Security Boundary

A good model:

```text
Untrusted Source
      ↓
Validation
      ↓
Security
      ↓
Trusted Artifact
      ↓
Protected Environment
      ↓
Production
```

The further a job is from trusted state, the less privilege it should receive.

---

## 81. Untrusted Merge Request

MR pipeline:

```text
Code
 ↓
Tests
 ↓
Security
```

Avoid giving it:

```text
AWS production credentials
Kubernetes admin
production deploy token
```

unless the workflow has explicit, safe isolation and policy.

---

## 82. Protected Variables

Production variables should only be available where appropriate.

Concept:

```text
Protected branch/environment
       ↓
Deployment job
       ↓
Production credentials
```

A developer should not be able to modify arbitrary pipeline code and automatically gain production credentials.

---

## 83. Secret Masking Limitations

Masked variables reduce accidental log exposure but are not a complete security boundary.

Malicious or unsafe code can still:

- exfiltrate secrets
- send requests
- write files
- alter commands

Therefore:

```text
Masking
+
Protection
+
Isolation
+
Least privilege
```

are all needed.

---

## 84. Runner Compromise

If a runner is compromised:

```text
Runner
 ↓
Job execution
 ↓
Potential credential exposure
```

Response:

1. isolate runner
2. stop affected jobs
3. revoke/rotate credentials
4. inspect logs
5. inspect cloud/GitLab audit records
6. rebuild trusted runner
7. investigate affected artifacts
8. restore secure workflow

---

## 85. GitLab CI/CD and DevSecOps

Security gates should be integrated:

```text
Source
 ↓
SAST
 ↓
SCA
 ↓
Secrets
 ↓
Build
 ↓
Container Scan
 ↓
Policy
 ↓
Artifact
```

Do not move all security checks to a separate post-deployment process.

---

## 86. Quality Gates

A quality gate can block progression.

Examples:

```text
SonarQube quality gate failed
        ↓
Pipeline blocked
```

or:

```text
Trivy critical finding
        ↓
Pipeline blocked
```

Policy should be explicit.

---

## 87. Pipeline Policy

A mature policy can state:

```text
Critical security finding
 → block

Required test failure
 → block

Lint warning
 → warn or block according to policy

Infrastructure plan
 → review required

Production deployment
 → protected approval
```

---

## 88. Pipeline Failure Classification

Useful categories:

```text
VALIDATION_FAILED
BUILD_FAILED
TEST_FAILED
SECURITY_FAILED
ARTIFACT_FAILED
AUTH_FAILED
INFRASTRUCTURE_FAILED
DEPLOYMENT_FAILED
VERIFICATION_FAILED
TIMEOUT
CANCELED
```

Classification improves incident response.

---

## 89. Example End-to-End Pipeline

```text
                    GitLab
                       │
                       ▼
                 Merge Request
                       │
                       ▼
                   Validate
                       │
                       ▼
                    Build
                       │
                       ▼
                     Test
                       │
              ┌────────┼────────┐
              ▼        ▼        ▼
          SonarQube  Trivy  Veracode
              └────────┼────────┘
                       ▼
                 Docker Build
                       │
                       ▼
                      ECR
                       │
                       ▼
                GitOps Repository
                       │
                       ▼
                    ArgoCD
                       │
                       ▼
                     EKS
                       │
                       ▼
                 ALB Ingress
                       │
                       ▼
                 Smoke Testing
```

---

## 90. Production Pipeline Design Principles

A good pipeline should be:

### Repeatable

Same source and configuration produce predictable results.

### Idempotent

Retries do not create unexpected side effects.

### Secure

Least privilege and protected credentials.

### Observable

Logs, metrics, status, and audit evidence.

### Fast enough

Parallelize safe independent work.

### Deterministic

Use pinned dependencies and immutable artifacts.

### Recoverable

Failures have known remediation/rollback paths.

---

## 91. CI/CD Anti-Patterns

### Anti-pattern 1

Deploying directly from every feature branch.

### Anti-pattern 2

Long-lived cloud access keys in CI.

### Anti-pattern 3

`latest` as the only image identity.

### Anti-pattern 4

`|| true` around mandatory security/tests.

### Anti-pattern 5

No timeout.

### Anti-pattern 6

Unlimited retries.

### Anti-pattern 7

Privileged shared runner.

### Anti-pattern 8

Every job has production credentials.

### Anti-pattern 9

Rebuilding different artifacts for each environment.

### Anti-pattern 10

Deploying successfully but never verifying application health.

---

## 92. Troubleshooting — Pipeline Never Starts

Check:

```text
.gitlab-ci.yml syntax
 ↓
workflow rules
 ↓
job rules
 ↓
pipeline source
 ↓
branch
 ↓
project permissions
 ↓
Runner availability
```

A pipeline that never starts is different from a pipeline whose job fails.

---

## 93. Troubleshooting — Job Stuck

Check:

1. Runner online status.
2. Runner tags.
3. Protected runner restrictions.
4. Project/group association.
5. Executor availability.
6. Runner capacity.
7. Runner registration/configuration.

Common cause:

```text
Job requires tag X
Runner does not have tag X
```

---

## 94. Troubleshooting — Job Pending

Typical causes:

```text
No matching Runner
Runner offline
Runner busy
Protected runner mismatch
Tag mismatch
Runner scope mismatch
```

Do not change the pipeline blindly before checking Runner configuration.

---

## 95. Troubleshooting — Job Fails Immediately

Check:

```text
Image
Shell
Working directory
Environment variables
Dependency installation
Runner executor
Permissions
```

Compare local runtime with CI runtime.

---

## 96. Troubleshooting — Authentication Failure

For AWS:

```text
Identity
 ↓
Role
 ↓
Trust policy
 ↓
Permission
 ↓
Region/account
```

For GitLab:

```text
Token
 ↓
Scope
 ↓
Project access
 ↓
Protected status
```

Do not solve authorization failures by granting AdministratorAccess.

---

## 97. Troubleshooting — Docker Push Failure

Check:

```text
Registry URL
AWS/GitLab authentication
Repository existence
Image tag
Network
Permissions
Region/account
```

For ECR, verify the AWS identity and repository.

---

## 98. Troubleshooting — Terraform Job Failure

Check:

```text
Terraform version
Provider versions
Backend
Credentials
State lock
Variables
Working directory
Plan
```

If apply timed out, inspect actual infrastructure/state before rerunning.

---

## 99. Troubleshooting — ArgoCD Deployment Failure

Check:

```text
GitOps commit
 ↓
ArgoCD application
 ↓
Sync status
 ↓
Health status
 ↓
Kubernetes events
 ↓
Pod logs
 ↓
Service/endpoints
 ↓
Ingress/ALB
```

Do not assume ArgoCD sync success equals application success.

---

## 100. Troubleshooting — Security Gate Failed

Identify:

```text
Tool
 ↓
Finding
 ↓
Severity
 ↓
Affected component
 ↓
Policy
```

Then:

```text
Remediate
 ↓
Rebuild
 ↓
Rescan
 ↓
Verify
```

Use an approved exception process only when justified.

---

## 101. Troubleshooting — Pipeline Timeout

Determine where time was spent:

```text
Queue
 ↓
Runner startup
 ↓
Dependency download
 ↓
Build
 ↓
Scan
 ↓
Artifact upload
 ↓
Deployment
```

Then optimize the actual bottleneck.

---

## 102. Troubleshooting — Pipeline Suddenly Slower

Compare:

```text
Current vs previous pipeline
```

Look for:

- dependency changes
- runner capacity
- cache misses
- security scan changes
- network latency
- Docker build context
- artifact size
- external service latency

Measure before changing architecture.

---

## 103. Troubleshooting — Wrong Version Deployed

Trace:

```text
Git commit
 ↓
Pipeline
 ↓
Artifact
 ↓
Image digest
 ↓
GitOps commit
 ↓
ArgoCD revision
 ↓
Pod image
```

This is more reliable than checking only the image tag.

---

## 104. Senior Interview Question — What Is CI/CD?

> CI/CD is the automated process of validating source changes, producing trusted artifacts, and delivering them through controlled environments. CI emphasizes integration and validation; CD extends this toward repeatable release and deployment.

---

## 105. Senior Interview Question — What Happens When a Developer Pushes Code?

Strong answer:

> GitLab receives the push, evaluates pipeline/workflow rules, creates the applicable pipeline, assigns eligible jobs to Runners, executes validation/build/test/security stages, produces artifacts, and—if the workflow permits—promotes an immutable artifact through protected deployment stages. In a GitOps architecture, ArgoCD then reconciles the Kubernetes desired state.

---

## 106. Senior Interview Question — How Do You Design a Production Pipeline?

> I start with source validation, build, tests, security gates, immutable artifact creation, controlled promotion, protected deployment credentials, deployment verification, observability, and rollback. I also design for idempotency, bounded retries, timeouts, concurrency control, and auditability.

---

## 107. Senior Interview Question — Why Use Immutable Artifacts?

> An immutable artifact gives a stable identity to the exact content that was tested and approved. This improves traceability and prevents a mutable tag from silently pointing to different content.

---

## 108. Senior Interview Question — Why Use OIDC?

> OIDC can provide short-lived federated cloud credentials without storing long-lived access keys in GitLab. I still restrict the trust policy and permissions to the intended repository, environment, and workload.

---

## 109. Senior Interview Question — Why Not Put All Jobs on One Runner?

> Different jobs have different trust and resource requirements. A single highly privileged runner increases blast radius. I prefer appropriate isolation, restricted runners, and least privilege.

---

## 110. Senior Interview Question — Why Does a Pipeline Passing Not Mean Deployment Success?

> A pipeline job can successfully execute its commands while the application remains unhealthy. Deployment success requires verification of the target state, rollout, Pod readiness, Service endpoints, ingress/load-balancer health, and application smoke tests.

---

## 111. Senior Interview Scenario — Production Pipeline Fails After Image Push

Answer structure:

```text
Image already published
        ↓
Do not rebuild blindly
        ↓
Verify image digest
        ↓
Identify failed stage
        ↓
Inspect GitOps state
        ↓
Determine whether deployment occurred
        ↓
Recover from the correct checkpoint
```

The artifact may already be valid even if a later stage failed.

---

## 112. Senior Interview Scenario — Two Pipelines Deploy Simultaneously

> I would prevent conflicting production mutations using environment/deployment concurrency controls, identify which revision should win, and verify the final GitOps desired state and running image. I would not simply rerun the older pipeline.

---

## 113. Senior Interview Scenario — Security Scan Is Slow

> First measure whether the scan is actually the bottleneck. Then consider caching where safe, scanning only relevant artifacts, parallelizing independent checks, optimizing the scanner configuration, and retaining required security coverage. I would not disable critical security checks simply for speed.

---

## 114. Senior Interview Scenario — Runner Is Compromised

> I would isolate the runner, stop affected workloads, rotate credentials, inspect GitLab/cloud audit logs, determine affected artifacts, rebuild the runner from a trusted image, and review why the runner had excessive access.

---

## 115. Production Readiness Checklist

```text
[ ] CI configuration reviewed
[ ] Protected branches
[ ] Protected environments
[ ] Trusted runners
[ ] Runner isolation
[ ] Least-privilege credentials
[ ] OIDC/short-lived credentials where supported
[ ] Build reproducibility
[ ] Unit/integration tests
[ ] SonarQube
[ ] Trivy
[ ] Veracode where applicable
[ ] Immutable artifacts
[ ] ECR/registry controls
[ ] GitOps boundary
[ ] ArgoCD integration
[ ] Deployment verification
[ ] Smoke tests
[ ] Timeouts
[ ] Retry limits
[ ] Concurrency controls
[ ] Auditability
[ ] Rollback
[ ] Pipeline monitoring
```

---

## 116. Final Mental Model

Remember GitLab CI/CD as:

```text
              GitLab
                 │
                 ▼
             Pipeline
                 │
        ┌────────┼────────┐
        ▼        ▼        ▼
      Build     Test    Security
        │        │        │
        └────────┼────────┘
                 ▼
             Artifact
                 │
                 ▼
          Protected Deploy
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
             Verification
                 │
                 ▼
            Observability
```

The key principle:

> **CI/CD is not just running commands automatically. A production pipeline is a controlled system that converts an approved source change into a tested, secured, traceable, immutable artifact and then verifies that the intended production state was actually achieved.**

---