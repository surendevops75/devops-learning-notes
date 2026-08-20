# 13-GitLab — 05 GitLab CI/CD Configuration

> Production-oriented guide to writing, structuring, validating, securing, and optimizing `.gitlab-ci.yml`. This file moves from CI/CD fundamentals into practical configuration patterns used with Jenkins/GitHub Actions-style workflows, Docker, AWS/ECR, Terraform, Kubernetes/EKS, ArgoCD, SonarQube, Trivy, Veracode, and production deployments.

---

## 1. Purpose of `.gitlab-ci.yml`

`.gitlab-ci.yml` is the primary configuration file for GitLab CI/CD.

It defines:

- pipeline workflow
- stages
- jobs
- commands
- images
- services
- variables
- rules
- dependencies
- artifacts
- caches
- environments
- deployment behavior

Typical repository:

```text
application/
├── src/
├── tests/
├── Dockerfile
├── helm/
├── README.md
└── .gitlab-ci.yml
```

---

## 2. Basic Configuration

Minimal example:

```yaml
stages:
  - test

test:
  stage: test
  script:
    - echo "Running tests"
```

Flow:

```text
Git Push
   ↓
Pipeline
   ↓
test job
   ↓
Runner
   ↓
Command
```

---

## 3. YAML Structure

A GitLab CI file normally contains top-level configuration and job definitions.

Example:

```yaml
stages:
  - build
  - test

variables:
  APP_ENV: "test"

build:
  stage: build
  script:
    - echo "Build"

test:
  stage: test
  script:
    - echo "Test"
```

---

## 4. Stages

Define stages explicitly:

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

Stage order expresses the high-level pipeline lifecycle.

---

## 5. Jobs

Each job performs a unit of work.

```yaml
build_application:
  stage: build
  script:
    - mvn clean package
```

Job names should be descriptive:

```text
terraform_validate
unit_tests
docker_build
container_scan
deploy_staging
smoke_test
```

Avoid:

```text
job1
test2
run
```

---

## 6. Job Naming

Good:

```yaml
terraform_plan:
```

Better than:

```yaml
job:
```

A useful job name makes pipeline failures easier to understand.

---

## 7. Script

The `script` section is required for normal jobs.

```yaml
unit_test:
  stage: test
  script:
    - pytest
```

Multiple commands:

```yaml
unit_test:
  stage: test
  script:
    - pip install -r requirements.txt
    - pytest
    - coverage report
```

---

## 8. Before Script

Use `before_script` for shared setup.

Example:

```yaml
before_script:
  - python --version
  - pip install -r requirements.txt
```

Global setup should be lightweight.

If only one job needs the commands, put them in that job.

---

## 9. After Script

`after_script` can perform cleanup or diagnostics.

Example:

```yaml
after_script:
  - echo "Job finished"
```

Do not use cleanup commands that accidentally hide important failure evidence.

---

## 10. Default Configuration

Common job defaults can be centralized.

Concept:

```yaml
default:
  image: python:3.12
  before_script:
    - python --version
```

This reduces repetition.

Do not use global defaults when jobs require substantially different execution environments.

---

## 11. Job Image

For container-based runners:

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

The image should be trusted and preferably pinned according to organizational policy.

---

## 12. Job-Specific Image

Different jobs can use different images.

```yaml
python_tests:
  image: python:3.12
  script:
    - pytest

terraform_validate:
  image: hashicorp/terraform:<approved-version>
  script:
    - terraform validate
```

This is useful for heterogeneous pipelines.

---

## 13. Image Selection Security

Do not blindly use arbitrary public images.

Review:

- publisher
- provenance
- vulnerability status
- version
- package contents
- update policy

A CI image executes commands with the job's available permissions.

---

## 14. Services

Services provide supporting containers to jobs where supported by the Runner/executor.

Concept:

```text
Job Container
      │
      └── Service Container
```

Example use:

```text
Application tests
       ↓
PostgreSQL service
```

Use services only when needed.

---

## 15. Variables

Example:

```yaml
variables:
  APP_ENV: "test"
  REGION: "ap-south-1"
```

Access:

```bash
echo "$APP_ENV"
```

Do not store secrets in plain YAML.

---

## 16. Variable Expansion

Shell variables can be referenced:

```bash
echo "$CI_COMMIT_SHA"
echo "$CI_COMMIT_REF_NAME"
```

GitLab also provides predefined CI/CD variables.

Useful categories include:

- commit information
- project information
- pipeline information
- branch/tag information
- job information

---

## 17. Predefined Variables

Examples:

```text
CI_COMMIT_SHA
CI_COMMIT_SHORT_SHA
CI_COMMIT_BRANCH
CI_COMMIT_TAG
CI_PIPELINE_ID
CI_JOB_ID
CI_PROJECT_PATH
CI_PROJECT_DIR
```

Use predefined variables for traceability rather than hard-coding values.

---

## 18. Build Identity

A useful image tag can include the commit SHA:

```bash
IMAGE_TAG="$CI_COMMIT_SHORT_SHA"
```

Example:

```text
user-service:a81c92f
```

For production, also record the resulting image digest.

---

## 19. Source of Truth for Image Identity

Strong model:

```text
CI_COMMIT_SHA
      ↓
Build
      ↓
Image
      ↓
Image Digest
      ↓
GitOps
      ↓
ArgoCD
```

The digest is the strongest artifact identity.

---

## 20. Rules

`rules` control whether and how jobs are included.

Concept:

```yaml
deploy:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

Rules are central to safe GitLab pipelines.

---

## 21. Branch-Based Rules

Example:

```yaml
deploy_staging:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```

This means the job is intended for main branch execution.

It is not a replacement for protected environments.

---

## 22. Tag-Based Rules

Example:

```yaml
release:
  rules:
    - if: '$CI_COMMIT_TAG'
```

This can create a release workflow:

```text
Tag
 ↓
Build
 ↓
Security
 ↓
Publish
```

---

## 23. Merge Request Rules

Concept:

```yaml
unit_tests:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
```

This is useful for pre-merge validation.

---

## 24. Pipeline Source Rules

Example:

```yaml
security:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_PIPELINE_SOURCE == "push"'
```

Always understand what event caused the pipeline.

---

## 25. Workflow Rules

`workflow: rules` can control whether a pipeline itself is created.

Concept:

```yaml
workflow:
  rules:
    - if: '$CI_PIPELINE_SOURCE == "merge_request_event"'
    - if: '$CI_COMMIT_BRANCH'
```

This is different from job-level `rules`.

### Important distinction

```text
workflow:rules
     ↓
Should a pipeline exist?

job:rules
     ↓
Should this job exist in that pipeline?
```

---

## 26. Avoid Duplicate Pipelines

Poor configuration can create:

```text
Push pipeline
+
Merge Request pipeline
```

for the same change.

This can double:

- runner consumption
- build time
- security scanning
- artifact production

Use `workflow: rules` and job rules intentionally.

---

## 27. Rules with Changes

For monorepos:

```yaml
user_tests:
  rules:
    - changes:
        - services/user/**/*
```

This can prevent unrelated jobs from running.

Example:

```text
services/user/
   ↓
user pipeline

services/cart/
   ↓
cart pipeline
```

---

## 28. Rules with Variables

Concept:

```yaml
deploy:
  rules:
    - if: '$DEPLOY_ENABLED == "true"'
```

Do not use user-controlled variables as the only security boundary for production deployment.

---

## 29. Rules and Protected Branches

Better:

```text
rules
+
protected branch
+
protected environment
+
restricted credentials
```

Do not assume:

```yaml
if: '$CI_COMMIT_BRANCH == "production"'
```

alone secures production.

---

## 30. Only / Except

Older GitLab CI configurations may use:

```yaml
only:
except:
```

Modern configurations generally favor:

```yaml
rules:
```

When maintaining existing repositories, understand the existing behavior before migrating.

---

## 31. Dependencies

Jobs may depend on outputs from earlier jobs.

Concept:

```text
build
 ↓
artifact
 ↓
test
```

Use explicit relationships where useful.

---

## 32. `needs`

`needs` can create a dependency graph that allows jobs to start earlier.

Concept:

```text
build-a ───→ test-a
build-b ───→ test-b
```

instead of forcing all jobs to wait for the entire previous stage.

This can significantly reduce pipeline duration.

---

## 33. Stage vs `needs`

Without dependency optimization:

```text
Stage 1
  ↓
Stage 2
  ↓
Stage 3
```

With `needs`:

```text
build-a ─→ test-a ─→ deploy-a
build-b ─→ test-b
```

Use dependency graphs deliberately.

---

## 34. Artifacts

Example:

```yaml
build:
  script:
    - mvn package
  artifacts:
    paths:
      - target/app.jar
```

Artifact:

```text
Build
 ↓
app.jar
 ↓
Later job
```

---

## 35. Artifact Expiration

Temporary artifacts should not live forever.

Concept:

```yaml
artifacts:
  expire_in: 7 days
```

Important release artifacts may require different retention.

Retention should balance:

```text
Debugging
+
Audit
+
Compliance
+
Cost
```

---

## 36. Artifact Reports

Artifacts can also represent reports.

Examples:

- test reports
- coverage reports
- security reports
- Terraform plan output

The exact report syntax depends on report type and GitLab configuration.

---

## 37. Artifact Paths

Keep artifact paths precise.

Bad:

```yaml
paths:
  - .
```

This may upload unnecessary or sensitive files.

Better:

```yaml
paths:
  - target/app.jar
  - reports/
```

---

## 38. Cache

Cache is intended to speed repeated work.

Example concept:

```yaml
cache:
  paths:
    - .cache/
```

Typical use:

```text
Dependencies
 ↓
Cache
 ↓
Future pipeline
```

---

## 39. Artifact vs Cache

| Feature | Artifact | Cache |
|---|---|---|
| Purpose | Job output | Speed repeated work |
| Release evidence | Yes | No |
| Guaranteed availability | Depends on configuration | Not guaranteed |
| Typical use | Binary/report | Dependencies |
| Source of truth | Can be | Should not be |

Never treat cache as your only production artifact.

---

## 40. Cache Keys

Cache keys determine cache grouping.

Poor:

```text
one global cache
```

Better concept:

```text
project + dependency state + branch/trust boundary
```

The exact key should match the workload.

---

## 41. Cache Security

Do not share sensitive caches across untrusted and trusted jobs.

Potential risk:

```text
Untrusted job
 ↓
Writes malicious cache
 ↓
Trusted job restores cache
```

Separate trust boundaries.

---

## 42. YAML Anchors

YAML anchors can reduce repetition.

Example:

```yaml
.test_template: &test_template
  stage: test
  script:
    - pytest

unit_test:
  <<: *test_template
```

Use sparingly.

Excessive YAML indirection can make pipelines difficult to understand.

---

## 43. Hidden Jobs

A job name beginning with `.` can be used as a reusable configuration template.

Concept:

```yaml
.python_template:
  image: python:3.12
  before_script:
    - pip install -r requirements.txt
```

Then extend/reuse it.

---

## 44. `extends`

Example:

```yaml
.python_job:
  image: python:3.12
  before_script:
    - pip install -r requirements.txt

unit_test:
  extends: .python_job
  script:
    - pytest
```

This is generally clearer than deeply nested YAML anchors.

---

## 45. Reusable CI Templates

Large organizations may centralize reusable pipeline templates:

```text
platform-ci/
├── python.yml
├── docker.yml
├── terraform.yml
└── security.yml
```

Projects consume approved templates.

Benefits:

- consistency
- security policy
- reduced duplication
- easier updates

Risk:

- template changes can affect many projects

Use versioning and controlled rollout.

---

## 46. Includes

GitLab supports including configuration from reusable sources.

Concept:

```yaml
include:
  - local: '/ci/security.yml'
```

Other include mechanisms may reference remote/project/template sources depending on configuration.

Always validate the source and version.

---

## 47. Include Security

A pipeline can change dramatically when an included template changes.

For shared templates:

```text
Version
 ↓
Review
 ↓
Test
 ↓
Controlled rollout
```

Avoid blindly consuming an untrusted mutable external configuration.

---

## 48. Parent-Child Pipelines

Large systems can split pipelines.

Concept:

```text
Parent Pipeline
      │
 ┌────┼────┐
 ▼    ▼    ▼
App  Infra Security
```

Useful for:

- monorepos
- platform pipelines
- large organizations

---

## 49. Multi-Project Pipelines

One project can trigger or coordinate work in another project.

Example:

```text
Application Repository
       ↓
Build
       ↓
GitOps Repository
       ↓
Deployment Pipeline
```

Use controlled tokens/permissions.

---

## 50. Pipeline Components

Reusable CI/CD components can standardize common workflows.

Examples:

```text
Python test component
Docker build component
Terraform validation component
Security scanning component
```

Treat components as production dependencies.

---

## 51. Terraform CI Configuration

Example conceptual:

```yaml
terraform_validate:
  stage: validate
  script:
    - terraform fmt -check
    - terraform init -backend=false
    - terraform validate
```

Plan:

```yaml
terraform_plan:
  stage: test
  script:
    - terraform init
    - terraform plan
```

Production apply should be protected.

---

## 52. Terraform Apply

Concept:

```text
Merge
 ↓
Plan
 ↓
Approval
 ↓
Protected Apply
```

Do not expose unrestricted `terraform apply` to arbitrary branches.

---

## 53. Terraform Plan Artifact

A controlled pipeline may preserve plan output for review.

Concept:

```text
terraform plan
      ↓
plan file/report
      ↓
Review
      ↓
Apply
```

Be careful with sensitive information in plan artifacts.

Protect and restrict their retention.

---

## 54. Docker CI Configuration

Typical:

```text
Build application
 ↓
Run tests
 ↓
Build image
 ↓
Scan image
 ↓
Push image
```

Do not push before required validation unless the architecture intentionally supports a quarantine registry.

---

## 55. Docker Build Context

Large build contexts slow pipelines.

Use:

```text
.dockerignore
```

Example:

```dockerignore
.git
.gitlab
node_modules
.venv
*.log
terraform.tfstate
```

This improves build speed and reduces accidental inclusion.

---

## 56. Docker Image Tag

Use commit identity:

```bash
IMAGE_TAG="$CI_COMMIT_SHORT_SHA"
```

Then:

```bash
docker build -t "$IMAGE:$IMAGE_TAG" .
```

Push:

```bash
docker push "$IMAGE:$IMAGE_TAG"
```

Then retrieve/record the digest.

---

## 57. GitLab Registry vs ECR

GitLab can provide a container registry.

Your AWS architecture may use:

```text
GitLab CI
   ↓
ECR
   ↓
EKS
```

Use ECR when AWS-native registry integration and IAM controls fit the architecture.

---

## 58. AWS Authentication

Prefer:

```text
GitLab CI
 ↓
OIDC
 ↓
AWS STS
 ↓
Role
 ↓
ECR
```

Avoid long-lived access keys embedded in CI configuration.

---

## 59. Kubernetes Validation

For Helm/Kubernetes configuration:

```text
helm lint
helm template
Kubernetes schema validation
```

where appropriate.

The goal is to catch errors before deployment.

---

## 60. GitOps Update Job

Concept:

```text
Build image
 ↓
Get digest
 ↓
Update GitOps repository
 ↓
Commit
 ↓
Push
```

The job should verify:

```text
Correct repository
Correct branch
Correct application
Correct environment
Correct digest
```

---

## 61. ArgoCD Boundary

GitLab should not compete with ArgoCD for Kubernetes ownership.

Recommended:

```text
GitLab CI
 → Build
 → Test
 → Security
 → Publish
 → Update GitOps

ArgoCD
 → Reconcile
 → Sync
 → Health
```

This keeps responsibilities clear.

---

## 62. Environment Job

Example concept:

```yaml
deploy_staging:
  stage: deploy
  environment:
    name: staging
  script:
    - ./deploy-staging.sh
```

Environment metadata improves deployment visibility.

---

## 63. Production Environment

Concept:

```yaml
deploy_production:
  stage: deploy
  environment:
    name: production
  script:
    - ./deploy-production.sh
```

The environment should be protected in GitLab.

---

## 64. Manual Production Deployment

Concept:

```yaml
deploy_production:
  when: manual
```

Manual execution should be combined with:

- protected environment
- authorized users
- successful pipeline
- immutable artifact

---

## 65. Rules + Manual Deployment

A production job can be restricted to main:

```yaml
deploy_production:
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: manual
```

Again:

> Branch rules decide job inclusion; protected environments decide who is allowed to deploy.

---

## 66. Deployment Verification Job

Example:

```yaml
smoke_test:
  stage: verify
  script:
    - curl -f https://example.com/health
```

For Kubernetes/GitOps, verification can query application endpoints and deployment status.

---

## 67. Failure After Deployment

A production pipeline should make failure visible:

```text
Deploy
 ↓
Verify
 ↓
FAIL
```

Do not report success merely because the deployment command returned zero.

---

## 68. Rollback Job

A rollback mechanism can be prepared:

```text
Previous known-good revision
        ↓
Rollback
        ↓
Verification
```

Do not build a rollback job that blindly reverses arbitrary state.

---

## 69. Retry Configuration

Some GitLab jobs can be configured to retry.

Use retries for:

```text
transient failures
```

not:

```text
code failures
security failures
policy failures
```

Bound retries.

---

## 70. Timeout Configuration

Set job timeouts appropriate to workload.

Examples:

```text
Unit test → short
Docker build → moderate
Terraform plan → moderate
Deployment → controlled
```

Avoid extremely long global timeouts.

---

## 71. Interruptible Jobs

Development validation jobs may be safely interruptible when a newer commit supersedes them.

Production mutation jobs require much more careful treatment.

Never make a deployment interruptible without understanding partial-change behavior.

---

## 72. Resource Groups

When multiple pipelines can deploy the same environment, resource serialization can prevent conflicting operations.

Concept:

```text
Production resource
      ↓
Pipeline A runs
      ↓
Pipeline B waits
```

Useful for deployment race prevention.

---

## 73. Parallel Matrix Jobs

Testing multiple versions/configurations can use matrix-style execution.

Concept:

```text
Python 3.11
Python 3.12
Python 3.13
```

or:

```text
region-a
region-b
region-c
```

Use matrix testing where compatibility matters.

---

## 74. Matrix Cost

More parallel jobs mean:

```text
Higher runner usage
+
Higher pipeline cost
+
More logs
```

Use only the matrix dimensions that provide meaningful confidence.

---

## 75. Conditional Security Jobs

Security checks may run:

```text
Every MR
Every main build
Nightly
Release
```

The correct frequency depends on:

- risk
- scan duration
- application type
- policy
- release model

Critical controls should not be accidentally limited to an infrequent schedule.

---

## 76. Scheduled Security Scan

A scheduled pipeline can complement normal CI:

```text
Nightly
 ↓
Dependency scan
 ↓
Container scan
 ↓
Report
```

Useful because new vulnerabilities can appear after the original build.

---

## 77. Dependency Updates

A dependency update pipeline can:

```text
Detect update
 ↓
Create branch/MR
 ↓
Run tests
 ↓
Security
 ↓
Review
```

Automation should not blindly merge dependency changes into production.

---

## 78. Branch-Specific Configuration

Example:

```text
feature/*
 → test

main
 → test + package

tag
 → release

production
 → protected deployment
```

Keep branch logic readable.

---

## 79. Avoid Complex Rule Spaghetti

Bad:

```text
many overlapping if/changes/variables rules
```

This creates:

- unexpected jobs
- duplicate pipelines
- security gaps
- difficult troubleshooting

Prefer clear, documented conditions.

---

## 80. YAML Validation

Before committing major changes:

```text
Validate YAML
 ↓
Validate GitLab CI configuration
 ↓
Run test pipeline
```

Do not rely only on a text editor's YAML highlighting.

---

## 81. Pipeline Linting

Use GitLab's available CI configuration validation mechanisms to catch:

- syntax errors
- invalid keywords
- invalid combinations
- malformed rules

For production repositories, validate CI changes before merging.

---

## 82. CI Configuration Testing

Treat CI as code.

Workflow:

```text
Edit .gitlab-ci.yml
 ↓
Validate
 ↓
MR
 ↓
Pipeline
 ↓
Review
 ↓
Merge
```

Avoid making production CI changes directly in the web UI without review.

---

## 83. Debugging CI Variables

Safe debugging:

```bash
echo "Environment: $APP_ENV"
echo "Commit: $CI_COMMIT_SHORT_SHA"
echo "Project: $CI_PROJECT_PATH"
```

Never print secrets.

For a secret, verify presence without revealing its value, for example by checking whether it is set rather than printing it.

---

## 84. Shell Debugging

Temporary shell debugging may use:

```bash
set -x
```

But be careful:

```text
set -x
+
secret-bearing commands
```

can expose sensitive values.

Disable it before sensitive operations.

---

## 85. Shell Strictness

A controlled shell may use:

```bash
set -euo pipefail
```

Meaning broadly:

- `-e` → fail on unhandled command errors
- `-u` → catch unset variables
- `pipefail` → propagate pipeline failures

Test scripts carefully because strict mode can expose assumptions.

---

## 86. Error Messages

Prefer explicit errors:

```bash
test -n "$IMAGE_TAG" || {
  echo "IMAGE_TAG is missing"
  exit 1
}
```

This is easier to troubleshoot than a generic failure.

---

## 87. Validate Required Inputs

Deployment jobs should validate:

```text
AWS_ACCOUNT
AWS_REGION
CLUSTER_NAME
IMAGE_DIGEST
ENVIRONMENT
```

before changing production.

Example concept:

```bash
test -n "$CLUSTER_NAME" || exit 1
```

---

## 88. Prevent Wrong AWS Account

A production deployment should verify AWS identity.

Concept:

```bash
aws sts get-caller-identity
```

Then validate:

```text
Expected account
+
Expected role
```

Do not assume the role is correct because the environment variable says `prod`.

---

## 89. Prevent Wrong EKS Cluster

Before deployment:

```text
Expected AWS account
+
Expected region
+
Expected EKS cluster
```

Then verify Kubernetes context.

Never deploy based solely on a developer's local kubeconfig when the pipeline is intended to be controlled.

---

## 90. Kubernetes Context Validation

Concept:

```bash
kubectl config current-context
```

Then verify the expected target before any direct Kubernetes operation.

In GitOps, ArgoCD should normally own the production Kubernetes mutation path.

---

## 91. Production Safety Guard

A deployment script can validate:

```text
ENVIRONMENT == production
AWS account == expected
AWS region == expected
cluster == expected
artifact == expected
```

Only then proceed.

---

## 92. Terraform Environment Guard

Terraform pipelines should ensure:

```text
correct backend
correct workspace/directory
correct account
correct region
correct variable set
```

Do not trust only a variable called:

```text
ENV=production
```

---

## 93. Secret Exposure Through Artifacts

Artifacts can accidentally contain:

```text
.env
credentials
Terraform plans
logs
temporary files
```

Review artifact paths carefully.

Use restricted retention and access for sensitive reports.

---

## 94. Secret Exposure Through Logs

Common mistakes:

```bash
echo "$TOKEN"
env
set
curl -H "Authorization: Bearer $TOKEN" ...
```

Be careful with commands that print:

- environment
- headers
- command lines
- configuration

---

## 95. Secret Exposure Through Docker

Do not pass secrets into Docker builds casually.

A secret used during image build can become part of:

- layers
- build logs
- cached content

Use approved secret-aware build mechanisms where necessary.

---

## 96. Production Configuration Pattern

Separate:

```text
Application artifact
```

from:

```text
Environment configuration
```

Example:

```text
Image
 ↓
Same artifact
 ↓
Dev configuration
 ↓
Staging configuration
 ↓
Production configuration
```

Do not bake environment-specific secrets into the image.

---

## 97. CI/CD for Your DevSecOps Stack

A practical pipeline:

```text
GitLab
 ↓
Maven / npm build
 ↓
Unit tests
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Docker build
 ↓
ECR
 ↓
GitOps update
 ↓
ArgoCD
 ↓
EKS
```

This should be the baseline mental model for your interview answers.

---

## 98. Complete Example — Application Pipeline

```yaml
stages:
  - validate
  - build
  - test
  - security
  - package

variables:
  IMAGE_TAG: "$CI_COMMIT_SHORT_SHA"

validate:
  stage: validate
  script:
    - echo "Validate source"

build:
  stage: build
  script:
    - echo "Build application"

test:
  stage: test
  script:
    - echo "Run tests"

security:
  stage: security
  script:
    - echo "Run security checks"

package:
  stage: package
  script:
    - echo "Build immutable package"
```

This is intentionally simplified.

---

## 99. Example — Python Pipeline

```yaml
stages:
  - test

python_tests:
  image: python:3.12
  stage: test
  script:
    - python --version
    - pip install -r requirements.txt
    - pytest
```

Production improvements would include:

- dependency locking
- caching
- reports
- security scanning
- controlled image
- test coverage
- reproducible environment

---

## 100. Example — Terraform Pipeline

```yaml
stages:
  - validate
  - plan

terraform_validate:
  image: hashicorp/terraform:<approved-version>
  stage: validate
  script:
    - terraform fmt -check
    - terraform init -backend=false
    - terraform validate

terraform_plan:
  image: hashicorp/terraform:<approved-version>
  stage: plan
  script:
    - terraform init
    - terraform plan
```

Production apply should be a separately controlled job.

---

## 101. Example — Docker Pipeline

```yaml
stages:
  - test
  - package

test:
  stage: test
  script:
    - ./run-tests.sh

docker_build:
  stage: package
  script:
    - docker build -t "$IMAGE:$CI_COMMIT_SHORT_SHA" .
```

Actual Docker execution depends on Runner architecture.

---

## 102. Example — GitOps Pipeline

Concept:

```yaml
update_gitops:
  stage: package
  script:
    - ./update-gitops.sh
```

The script should:

```text
Validate target
 ↓
Update exact image digest
 ↓
Review diff
 ↓
Commit
 ↓
Push
```

---

## 103. Avoid Hard-Coded Branch Names Everywhere

Bad:

```text
many jobs each independently check main
```

Prefer centralized workflow rules and reusable configuration.

This reduces inconsistent behavior.

---

## 104. Configuration DRY Principle

Avoid repeating:

```text
image
before_script
variables
rules
cache
```

Use:

- defaults
- templates
- extends
- includes

But do not over-abstract.

Readable CI is more valuable than clever CI.

---

## 105. Configuration Documentation

Document unusual behavior:

```yaml
# Production deployment is restricted to protected main.
# ArgoCD owns Kubernetes reconciliation.
# This job only updates the GitOps repository.
```

Documentation should explain **why**, not merely restate the command.

---

## 106. Pipeline Versioning

Changes to CI templates should be versioned when practical:

```text
v1
v2
```

Consumers can upgrade deliberately.

This reduces unexpected organization-wide breakage.

---

## 107. Breaking Template Change

Suppose shared template changes:

```text
Docker build logic
```

and 100 projects consume it.

Do not immediately deploy the change everywhere.

Use:

```text
Test
 ↓
Pilot projects
 ↓
Observe
 ↓
Expand
```

---

## 108. Production Pipeline Change Process

For critical CI changes:

```text
Feature branch
 ↓
CI configuration change
 ↓
Validation
 ↓
MR
 ↓
Review
 ↓
Security
 ↓
Pilot
 ↓
Merge
 ↓
Monitor
```

Treat pipeline infrastructure like application infrastructure.

---

## 109. Pipeline Observability

Measure:

```text
Pipeline success rate
Pipeline duration
Queue time
Runner utilization
Job failure rate
Deployment frequency
Deployment success
Rollback rate
Security failure rate
```

Use Prometheus/Grafana/ELK where integrated with the platform.

---

## 110. Pipeline Logs and ELK

Concept:

```text
GitLab / Runner logs
       ↓
Log collection
       ↓
ELK
       ↓
Search / dashboards
```

Useful for:

- recurring failures
- deployment incidents
- runner problems
- authentication errors

Never forward secrets into centralized logs.

---

## 111. Pipeline Metrics and Prometheus

Concept:

```text
CI/CD Metrics
      ↓
Prometheus
      ↓
Grafana
```

Track operational indicators rather than collecting every possible metric.

---

## 112. Pipeline SLO Concepts

Useful service-level indicators:

```text
Pipeline success rate
Median pipeline duration
P95 pipeline duration
Deployment success rate
Rollback frequency
```

These can help identify CI platform degradation.

---

## 113. Pipeline Cost Optimization

Reduce unnecessary work through:

```text
rules
changes
needs
parallelization
cache
efficient Docker context
dependency caching
right-sized runners
```

Do not optimize by removing critical tests/security.

---

## 114. Runner Capacity Planning

Pipeline throughput depends on:

```text
Job arrival rate
+
Job duration
+
Runner count
+
Runner capacity
```

If jobs queue for long periods:

```text
Increase capacity
or
Reduce execution time
or
Improve scheduling
```

---

## 115. Production Incident — Configuration Error

Suppose a YAML change causes every pipeline to fail.

Response:

```text
Identify commit
 ↓
Confirm scope
 ↓
Revert CI change
 ↓
Validate
 ↓
Restore pipeline
 ↓
Investigate root cause
```

Do not repeatedly modify production CI without restoring a known-good baseline.

---

## 116. Production Incident — Wrong Rule

A security job unexpectedly stops running.

Check:

```text
workflow rules
job rules
pipeline source
branch
changes
variables
include/template version
```

Then verify that the security gate is actually mandatory for the target workflow.

---

## 117. Production Incident — Production Job Appears in MR

Potential risk:

```text
Untrusted MR
 ↓
Production job visible/eligible
```

Check:

```text
rules
environment protection
protected variables
runner protection
pipeline source
```

The job should not gain production access merely because it is visible in the pipeline configuration.

---

## 118. Production Incident — Artifact Missing

Check:

```text
Artifact path
Job status
Artifact expiration
Job dependency
needs/dependencies
Runner upload
Storage
```

Do not rebuild blindly until determining whether the original artifact is recoverable.

---

## 119. Production Incident — Job Cannot Download Artifact

Check:

```text
Producer job
 ↓
Artifact exists?
 ↓
Consumer dependency
 ↓
needs/dependencies
 ↓
Artifact retention
 ↓
Access
```

A successful producer job does not automatically mean every consumer has access to its artifacts under every dependency configuration.

---

## 120. Production Incident — Cache Corruption

Symptoms:

```text
Previously passing job
 ↓
Suddenly fails
 ↓
Clear cache
 ↓
Job passes
```

Root cause may be:

- stale dependency
- incompatible cache key
- corrupted cache
- trust-boundary issue

Fix the cache design instead of repeatedly clearing it.

---

## 121. Senior Interview — `rules` vs `workflow:rules`

> `workflow:rules` controls whether GitLab creates a pipeline. Job-level `rules` control whether a particular job is included and how it behaves. I use workflow rules to prevent duplicate/unwanted pipelines and job rules to control individual job execution.

---

## 122. Senior Interview — `needs` vs Stages

> Stages provide the high-level ordering model. `needs` creates explicit job dependencies and can allow downstream jobs to start without waiting for every job in the previous stage, reducing pipeline duration.

---

## 123. Senior Interview — How Do You Secure `.gitlab-ci.yml`?

> Protect the repository and branches, review CI changes, restrict production variables, use trusted runners, prefer short-lived credentials, protect environments, validate includes/templates, and ensure untrusted MR pipelines cannot access production privileges.

---

## 124. Senior Interview — How Do You Avoid Duplicate Pipelines?

> I examine the pipeline sources and use `workflow: rules` to control which pipeline types should be created, then use job-level `rules` for individual execution. I test push and MR behavior explicitly.

---

## 125. Senior Interview — How Do You Optimize a Slow Pipeline?

> I measure queue time and job duration first. Then I use dependency graphs, parallel jobs, caching where safe, selective `changes` rules, efficient build contexts, optimized dependency installation, and appropriate Runner capacity. I never remove mandatory security or correctness checks simply for speed.

---

## 126. Senior Interview — How Do You Prevent Wrong-Environment Deployment?

> I verify the protected environment, AWS account, region, EKS cluster, deployment identity, artifact digest, and GitOps target before deployment. I do not rely solely on an environment variable such as `ENV=prod`.

---

## 127. Senior Interview — Why Is CI Configuration Production Code?

> Because CI configuration determines what commands execute, which credentials are available, which runners are used, what security checks run, which artifacts are created, and whether deployment occurs. A malicious or incorrect CI change can therefore affect the entire production system.

---

## 128. Senior Interview — What Makes a Pipeline Production-Ready?

> It must be reproducible, secure, observable, idempotent, traceable, protected, reasonably fast, failure-aware, and recoverable. It should produce immutable artifacts, use least-privilege credentials, control production concurrency, verify deployments, and have a tested rollback strategy.

---

## 129. Final Configuration Checklist

```text
[ ] Valid YAML
[ ] Clear stages
[ ] Descriptive jobs
[ ] Trusted images
[ ] Appropriate Runner
[ ] Correct variables
[ ] No hard-coded secrets
[ ] Protected variables
[ ] Workflow rules
[ ] Job rules
[ ] No duplicate pipelines
[ ] Correct MR behavior
[ ] Appropriate needs/dependencies
[ ] Artifacts defined
[ ] Cache scoped correctly
[ ] Security scans
[ ] Immutable artifact identity
[ ] Protected environments
[ ] Production approval
[ ] Timeouts
[ ] Bounded retries
[ ] Concurrency control
[ ] Deployment verification
[ ] Rollback
[ ] Logs
[ ] Metrics
[ ] Auditability
```

---

## 130. Final Mental Model

```text
.gitlab-ci.yml
      │
      ▼
Workflow Rules
      │
      ▼
Pipeline
      │
      ├── Validate
      ├── Build
      ├── Test
      ├── Security
      ├── Package
      ├── Deploy
      └── Verify
             │
             ▼
       Immutable Artifact
             │
             ▼
       Protected Environment
             │
             ▼
           GitOps
             │
             ▼
           ArgoCD
             │
             ▼
             EKS
```

The key principle:

> **A good `.gitlab-ci.yml` is not simply a list of shell commands. It is executable delivery policy: it defines when work runs, what is trusted, which checks are mandatory, which artifacts are produced, who can deploy, and how production changes are verified and recovered.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md ✓
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

**Next: `06-GitLab-Runners.md`**
