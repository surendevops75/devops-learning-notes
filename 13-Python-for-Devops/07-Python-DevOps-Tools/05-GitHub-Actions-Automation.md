# GitHub Actions Automation with Python

## 1. Overview

GitHub Actions is a CI/CD and workflow automation platform integrated with GitHub.

Python can automate GitHub Actions workflows and support the complete software delivery lifecycle:

- Triggering workflows
- Passing workflow inputs
- Monitoring workflow runs
- Reading workflow status
- Downloading artifacts
- Managing releases
- Updating repositories
- Creating pull requests
- Managing branches and tags
- Updating GitOps repositories
- Calling GitHub REST APIs
- Calling GitHub GraphQL APIs where appropriate
- Validating repositories
- Generating reports
- Integrating AWS
- Integrating Docker and ECR
- Integrating Terraform
- Integrating Kubernetes and ArgoCD
- Implementing deployment gates
- Automating release workflows
- Troubleshooting failed workflows

The production principle is:

> **GitHub Actions should remain the workflow execution platform, while Python provides reusable automation, API integration, validation, policy enforcement, and release orchestration.**

A typical production flow is:

```text
Developer
   |
   v
GitHub
   |
   v
GitHub Actions
   |
   +-- Python validation
   +-- Tests
   +-- SonarQube
   +-- Trivy
   +-- Docker
   +-- Terraform
   |
   v
ECR
   |
   v
GitOps Repository
   |
   v
ArgoCD
   |
   v
EKS
```

---

# 2. GitHub Actions Architecture

```text
                         GitHub
                           |
              +------------+------------+
              |                         |
              v                         v
        Source Repository         GitOps Repository
              |                         |
              v                         ^
        GitHub Actions                 |
              |                         |
      +-------+-------+                 |
      |       |       |                 |
      v       v       v                 |
   Python   Docker  Terraform           |
      |       |       |                 |
      +-------+-------+                 |
              |                         |
              v                         |
             AWS -----------------------+
              |
              v
             EKS
```

Python may operate:

```text
Inside a GitHub Actions runner
```

or:

```text
Externally through GitHub APIs
```

---

# 3. GitHub Actions Components

A GitHub Actions workflow generally contains:

```text
Workflow
 |
 +-- Trigger
 |
 +-- Jobs
       |
       +-- Steps
             |
             +-- Actions
             +-- Shell commands
             +-- Python scripts
```

Example:

```yaml
name: CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Run Python
        run: python3 scripts/validate.py
```

---

# 4. Workflow Triggers

Common triggers include:

```text
push
pull_request
workflow_dispatch
schedule
workflow_call
release
workflow_run
repository_dispatch
```

Example:

```yaml
on:
  pull_request:
    branches:
      - main
```

The trigger should match the intended lifecycle.

---

# 5. Manual Workflow Trigger

For deployment workflows:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Environment
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod
```

Python can validate the input again.

Never assume a workflow input is safe merely because GitHub UI provides a dropdown.

---

# 6. Python Inside GitHub Actions

Simple pattern:

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Setup Python
    uses: actions/setup-python@v5
    with:
      python-version: "3.12"

  - name: Install dependencies
    run: |
      python -m pip install --upgrade pip
      pip install -r requirements.txt

  - name: Run automation
    run: |
      python scripts/automation.py
```

Python becomes a reusable automation layer inside the workflow.

---

# 7. Python Arguments

GitHub Actions:

```yaml
- name: Run deployment validation
  run: |
    python scripts/validate.py \
      --environment "${{ inputs.environment }}"
```

Python:

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument(
    "--environment",
    required=True
)

args = parser.parse_args()
```

Always validate the allowed values.

---

# 8. GitHub Environment Variables

GitHub Actions provides environment variables such as:

```text
GITHUB_REPOSITORY
GITHUB_REF
GITHUB_SHA
GITHUB_RUN_ID
GITHUB_RUN_NUMBER
GITHUB_WORKSPACE
GITHUB_ACTOR
GITHUB_EVENT_NAME
```

Python:

```python
import os

repository = os.getenv(
    "GITHUB_REPOSITORY"
)

commit = os.getenv(
    "GITHUB_SHA"
)

run_id = os.getenv(
    "GITHUB_RUN_ID"
)
```

These values are useful for release metadata.

---

# 9. GitHub Context

GitHub Actions expressions can access:

```text
github
env
vars
secrets
inputs
needs
steps
matrix
runner
```

Example:

```yaml
run: |
  echo "Commit: $GITHUB_SHA"
```

Do not print sensitive contexts.

---

# 10. Python Release Metadata

Python can construct:

```python
import os

release = {
    "repository": os.getenv(
        "GITHUB_REPOSITORY"
    ),
    "commit": os.getenv(
        "GITHUB_SHA"
    ),
    "run_id": os.getenv(
        "GITHUB_RUN_ID"
    ),
    "actor": os.getenv(
        "GITHUB_ACTOR"
    )
}
```

This metadata can be attached to:

```text
Docker labels
Reports
Artifacts
Deployment records
Logs
```

---

# 11. GitHub REST API

GitHub exposes REST APIs for:

```text
Repositories
Branches
Commits
Pull requests
Issues
Actions
Artifacts
Releases
Deployments
Environments
Packages
```

Python can use:

```python
import requests
```

or an approved GitHub SDK.

For production, use:

```text
Authentication
Timeouts
Retries
Pagination
Rate-limit handling
Error classification
```

---

# 12. GitHub API Authentication

Preferred authentication depends on the automation use case.

Possible mechanisms include:

```text
GitHub App installation token
Fine-grained personal access token
Workflow-provided token
OIDC for cloud identity
```

For organizational automation, GitHub Apps are often preferable when fine-grained repository-level permissions and machine identity are required.

Never hardcode tokens.

---

# 13. `GITHUB_TOKEN`

GitHub Actions can provide:

```text
GITHUB_TOKEN
```

through the workflow environment.

Example:

```yaml
permissions:
  contents: read
```

Python:

```python
import os

token = os.environ[
    "GITHUB_TOKEN"
]
```

Use the minimum permissions needed.

---

# 14. Workflow Permissions

A strong workflow explicitly defines permissions.

Example:

```yaml
permissions:
  contents: read
```

For a workflow that needs to modify repository contents:

```yaml
permissions:
  contents: write
```

Do not grant:

```yaml
permissions: write-all
```

unless there is a justified and documented requirement.

---

# 15. GitHub App Authentication

A GitHub App can provide:

```text
Machine identity
Repository-scoped permissions
Short-lived installation tokens
Centralized management
```

Architecture:

```text
Python
 |
 v
GitHub App
 |
 v
Installation Token
 |
 v
GitHub API
```

This is preferable to sharing a personal account credential for organization-wide automation.

---

# 16. Basic GitHub API Request

```python
import os
import requests

token = os.environ["GITHUB_TOKEN"]

headers = {
    "Authorization": f"Bearer {token}",
    "Accept": "application/vnd.github+json"
}

response = requests.get(
    "https://api.github.com/repos/ORG/REPO",
    headers=headers,
    timeout=20
)

response.raise_for_status()

repo = response.json()

print(repo["full_name"])
```

Do not print the token.

---

# 17. API URL Construction

Avoid unsafe string handling when values originate from untrusted input.

Validate:

```text
Organization
Repository
Branch
Workflow filename
```

before constructing API paths.

---

# 18. Trigger a GitHub Actions Workflow

GitHub provides a workflow dispatch API.

Conceptually:

```text
Python
 |
 v
POST workflow_dispatch
 |
 v
GitHub Actions
 |
 v
Workflow run
```

Example:

```python
url = (
    "https://api.github.com/repos/"
    f"{owner}/{repo}/actions/workflows/"
    f"{workflow_id}/dispatches"
)

payload = {
    "ref": "main",
    "inputs": {
        "environment": "staging",
        "image_tag": "git-abc123"
    }
}

response = requests.post(
    url,
    headers=headers,
    json=payload,
    timeout=20
)

response.raise_for_status()
```

---

# 19. Trigger Validation

Before triggering:

```text
Repository exists
Workflow exists
Workflow enabled
Branch/ref exists
Input values valid
Caller authorized
```

This prevents avoidable workflow failures.

---

# 20. Workflow Run Discovery

After triggering a workflow, the API may not immediately return the exact run object.

A robust approach is:

```text
Trigger workflow
 |
 v
Wait briefly
 |
 v
List recent workflow runs
 |
 v
Find matching run
```

Match using:

```text
Workflow
Branch/ref
Commit SHA
Created time
Inputs where available
Release ID
```

Do not simply take:

```text
latest run
```

because another user or automation may start a run concurrently.

---

# 21. Workflow Run Status

GitHub Actions run data includes status/conclusion information.

Conceptually:

```text
status:
queued
in_progress
completed
```

Conclusion may include:

```text
success
failure
cancelled
skipped
timed_out
```

Python should distinguish:

```text
Still running
```

from:

```text
Completed successfully
```

and:

```text
Completed unsuccessfully
```

---

# 22. Polling Workflow Runs

```python
import time


deadline = time.time() + 1800

while time.time() < deadline:

    response = requests.get(
        run_url,
        headers=headers,
        timeout=20
    )

    response.raise_for_status()

    data = response.json()

    if data["status"] == "completed":
        conclusion = data[
            "conclusion"
        ]
        break

    time.sleep(10)
else:
    raise TimeoutError(
        "Workflow timed out"
    )
```

Always use bounded polling.

---

# 23. Why Polling Needs a Timeout

Bad:

```python
while True:
    ...
```

A GitHub Actions runner can become:

```text
Queued
Stuck
Offline
Waiting for approval
```

A Python orchestrator should eventually stop waiting and report the condition.

---

# 24. GitHub Actions Queue Delays

A workflow can remain queued because:

```text
No available runner
Concurrency group
Environment approval
Repository capacity
Hosted runner availability
Self-hosted runner offline
```

Python should distinguish:

```text
Queue delay
```

from:

```text
Workflow execution failure
```

---

# 25. Self-Hosted Runner Architecture

```text
GitHub
 |
 v
Actions Controller
 |
 v
Self-hosted Runner
 |
 +-- Python
 +-- Docker
 +-- Terraform
 +-- AWS CLI
```

Self-hosted runners require careful security controls because workflows execute code on the runner.

---

# 26. Self-Hosted Runner Security

Consider:

```text
Runner isolation
Ephemeral runners
Network access
Credential exposure
Workspace cleanup
Software updates
Untrusted pull requests
Privilege level
```

Never allow untrusted code to access powerful production credentials.

---

# 27. Ephemeral Runners

Architecture:

```text
Workflow
   |
   v
Runner provisioned
   |
   v
Build
   |
   v
Runner destroyed
```

Benefits:

```text
Clean environment
Reduced persistence
Less cross-job contamination
Better isolation
```

This is especially useful for sensitive CI workloads.

---

# 28. GitHub Actions Environments

Use environments:

```text
dev
staging
production
```

Production can have:

```text
Required reviewers
Environment secrets
Deployment protection rules
```

Python should complement these controls rather than trying to replace them.

---

# 29. Production Deployment Gate

A strong architecture:

```text
Workflow
 |
 v
Build
 |
 v
Test
 |
 v
Security
 |
 v
Image
 |
 v
Production Environment
 |
 v
Required Approval
 |
 v
Deploy
```

This is safer than relying on a Python `if` statement alone.

---

# 30. GitHub Actions Concurrency

Concurrency prevents overlapping deployments.

Example:

```yaml
concurrency:
  group: production-deployment
  cancel-in-progress: false
```

This can prevent:

```text
Deployment A
```

and:

```text
Deployment B
```

from modifying production simultaneously.

The policy should match the deployment model.

---

# 31. Python and Idempotency

Suppose Python triggers:

```text
production deployment
```

and receives a timeout.

The workflow may already be running.

Do not blindly trigger again.

Instead:

```text
Check workflow runs
 |
 v
Match release ID / commit
 |
 v
Reuse existing run
```

This prevents duplicate deployments.

---

# 32. Release ID

Create a unique release identifier:

```text
release-2026-08-18-001
```

Pass it through:

```text
Python
 |
 v
GitHub Actions input
 |
 v
Docker metadata
 |
 v
GitOps commit
 |
 v
ArgoCD
 |
 v
EKS
```

This makes cross-system correlation easier.

---

# 33. Workflow Inputs

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod

      image_tag:
        required: true
        type: string
```

Python can trigger:

```text
environment=staging
image_tag=git-a1b2c3
```

Validate both before use.

---

# 34. Input Validation in Python

```python
allowed = {
    "dev",
    "staging",
    "prod"
}

if environment not in allowed:
    raise ValueError(
        "Invalid environment"
    )
```

For image tags:

```python
import re

if not re.fullmatch(
    r"[A-Za-z0-9._-]+",
    image_tag
):
    raise ValueError(
        "Invalid image tag"
    )
```

Never pass raw user input directly into shell commands.

---

# 35. Shell Injection

Bad:

```python
import os

os.system(
    f"docker build -t {image_tag} ."
)
```

Better:

```python
import subprocess

subprocess.run(
    [
        "docker",
        "build",
        "-t",
        image_tag,
        "."
    ],
    check=True
)
```

Use the same principle for:

```text
terraform
kubectl
aws
git
helm
```

---

# 36. GitHub Repository API

Python can retrieve repository metadata:

```text
Default branch
Visibility
Topics
Archived state
Permissions
Security settings
```

This can support compliance checks.

Example:

```python
response = requests.get(
    repo_url,
    headers=headers,
    timeout=20
)

response.raise_for_status()

data = response.json()

print(
    data["default_branch"]
)
```

---

# 37. Branch Protection Validation

Production repositories should protect important branches.

Python can validate:

```text
main
release/*
```

for required controls such as:

```text
Pull request review
Status checks
Conversation resolution
Force-push restrictions
```

The exact API fields depend on repository configuration and GitHub API version.

---

# 38. Pull Request Automation

Python can:

```text
Create PR
Comment on PR
Add labels
Request reviewers
Check status
Merge PR
```

Example workflow:

```text
Build image
 |
 v
Update GitOps repository
 |
 v
Create PR
 |
 v
CI
 |
 v
Review
 |
 v
Merge
 |
 v
ArgoCD
```

This is safer than directly pushing production changes in many GitOps workflows.

---

# 39. GitOps Repository Update

Suppose application image changes:

```text
image:
  repository: ECR_REPO
  tag: old
```

Python can update:

```text
tag: git-abc123
```

in the GitOps repository.

Then:

```text
Commit
 |
 v
Pull Request
 |
 v
Review
 |
 v
Merge
 |
 v
ArgoCD
```

This preserves GitOps auditability.

---

# 40. Do Not Mix GitOps Ownership

Avoid:

```text
Python directly modifies EKS
```

when ArgoCD owns the application.

Prefer:

```text
Python
 |
 v
GitOps repository
 |
 v
ArgoCD
 |
 v
EKS
```

---

# 41. GitHub Actions + Docker

Typical workflow:

```text
Checkout
 |
 v
Python validation
 |
 v
Docker build
 |
 v
Trivy scan
 |
 v
ECR login
 |
 v
Push image
 |
 v
Capture digest
```

Python can orchestrate or validate each step.

---

# 42. Immutable Docker Tagging

Use:

```text
git-<short-sha>
```

Example:

```text
git-a1b2c3d
```

Better for production:

```text
image digest
```

because tags can theoretically be moved.

---

# 43. Capture Image Digest

After push:

```text
ECR
 |
 v
sha256:...
```

Store:

```text
Git SHA
Image tag
Image digest
Workflow run
```

Then promote the same artifact.

---

# 44. Build Once, Promote Same Artifact

Production principle:

```text
Build
 |
 v
Test
 |
 v
Scan
 |
 v
Publish
 |
 v
Promote
```

Do not rebuild the image separately for:

```text
staging
production
```

if the release model expects immutable artifact promotion.

---

# 45. GitHub Actions + AWS

For AWS deployments, prefer:

```text
GitHub OIDC
```

rather than long-lived access keys where supported.

Architecture:

```text
GitHub Actions
 |
 v
OIDC
 |
 v
AWS IAM Role
 |
 v
AWS
```

Python can verify:

```text
Account ID
Region
Identity
```

before sensitive operations.

---

# 46. AWS Account Validation

```python
import boto3

sts = boto3.client(
    "sts",
    region_name="ap-south-1"
)

identity = (
    sts.get_caller_identity()
)

actual_account = identity[
    "Account"
]

if actual_account != expected_account:
    raise RuntimeError(
        "Wrong AWS account"
    )
```

This is an important production safety gate.

---

# 47. GitHub OIDC Trust

AWS IAM trust should restrict:

```text
GitHub organization
Repository
Branch/environment
Workflow context
```

Do not allow any repository in an organization to assume a highly privileged production role unless that is explicitly intended.

---

# 48. Least-Privilege AWS Role

A production deployment role should have only the permissions required by the workflow.

Avoid:

```text
AdministratorAccess
```

for routine CI/CD.

For example:

```text
ECR push
S3 access
EKS describe
specific infrastructure permissions
```

depending on the workflow.

---

# 49. GitHub Actions + Terraform

Architecture:

```text
GitHub Actions
 |
 v
Python preflight
 |
 +-- AWS identity
 +-- Region
 +-- Environment
 |
 v
Terraform
 |
 v
Plan
 |
 v
Policy
 |
 v
Approval
 |
 v
Apply
```

This directly extends the Terraform automation patterns from:

```text
03-Terraform-Automation.md
```

---

# 50. Terraform Plan Artifact

A production workflow can:

```text
Generate plan
 |
 v
Save plan artifact
 |
 v
Policy analysis
 |
 v
Approval
 |
 v
Apply approved plan
```

Treat the plan as sensitive infrastructure data.

Use controlled artifact retention.

---

# 51. GitHub Actions + Jenkins

Some organizations operate both.

Possible migration architecture:

```text
GitHub
 |
 v
GitHub Actions
 |
 v
Python
 |
 v
Jenkins
 |
 v
Legacy pipeline
```

Python can bridge systems while workloads migrate.

But avoid building a permanent maze of nested CI/CD systems.

Define clear ownership.

---

# 52. Triggering Jenkins from GitHub Actions

Python can:

```text
Trigger Jenkins
Poll build
Read result
```

GitHub Actions then continues only if Jenkins succeeds.

Architecture:

```text
GitHub Actions
 |
 v
Python
 |
 v
Jenkins
 |
 v
Build
 |
 v
Result
 |
 v
GitHub Actions
```

This is useful during CI migration.

---

# 53. GitHub Actions + ArgoCD

A clean GitOps flow:

```text
GitHub application repo
 |
 v
GitHub Actions
 |
 +-- Build
 +-- Test
 +-- Scan
 +-- Push ECR
 |
 v
GitOps repo
 |
 v
ArgoCD
 |
 v
EKS
```

Python can update the GitOps repository and validate the release metadata.

---

# 54. GitHub Actions + Kubernetes

GitHub Actions can run:

```text
kubectl
helm
argocd CLI
```

But if ArgoCD is the deployment authority, the preferred pattern is:

```text
Git change
 |
 v
ArgoCD reconciliation
```

rather than direct `kubectl apply`.

---

# 55. Deployment Verification

After GitOps update:

```text
GitHub Actions
 |
 v
Wait / poll
 |
 v
ArgoCD
 |
 v
EKS
 |
 v
Application health
```

Python can verify:

```text
ArgoCD health
Deployment replicas
Pod status
Application endpoint
```

depending on the architecture.

---

# 56. GitHub Actions Job Outputs

Jobs can expose outputs:

```yaml
outputs:
  image_digest: ${{ steps.build.outputs.digest }}
```

Downstream jobs can consume them.

Python can also generate:

```text
release.json
```

and upload it as an artifact.

---

# 57. Python Release Manifest

Example:

```json
{
  "repository": "org/payment",
  "commit": "abc123",
  "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment",
  "digest": "sha256:...",
  "workflow_run": "987654321",
  "environment": "staging"
}
```

This creates a traceable release record.

---

# 58. GitHub Actions Artifacts

Artifacts can store:

```text
Test reports
Security reports
Terraform plans
Deployment manifests
Release metadata
Logs
```

Do not upload:

```text
Secrets
Private keys
Unredacted credentials
Sensitive state
```

unless there is a specifically secured and justified process.

---

# 59. Artifact Retention

Artifact retention should match:

```text
Compliance
Debugging needs
Release lifecycle
Storage cost
```

Production workflows should avoid indefinite artifact retention by default.

---

# 60. Downloading Artifacts with Python

Python can use the GitHub API to retrieve artifact metadata.

Then:

```text
Artifact ID
 |
 v
Download URL
 |
 v
Archive
 |
 v
Extract / validate
```

Verify:

```text
Expected artifact
Checksum where available
Version
Size
```

---

# 61. GitHub Releases

Python can automate releases:

```text
Create tag
Create release
Generate release notes
Upload assets
Publish release
```

Typical flow:

```text
Merge
 |
 v
Build
 |
 v
Test
 |
 v
Tag
 |
 v
Release
```

Avoid automatically releasing unverified artifacts.

---

# 62. Semantic Versioning

Python can calculate or validate:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
2.4.1
```

But versioning policy should come from the project/release process.

Do not create conflicting version sources.

---

# 63. Git Tags

Production release metadata may include:

```text
v2.4.1
```

plus:

```text
Git SHA
Docker digest
Workflow run
```

The tag identifies the release; the digest identifies the exact artifact.

---

# 64. GitHub API Pagination

Many GitHub API endpoints paginate results.

Bad:

```python
response.json()
```

assuming all results are returned.

For large repositories/orgs, implement:

```text
page
per_page
next page
```

or use a library that handles pagination.

---

# 65. Rate Limits

GitHub APIs have rate limits.

Python should inspect response headers where useful.

Handle:

```text
429
403 with rate-limit context
```

with appropriate backoff.

Do not aggressively poll GitHub Actions every second.

---

# 66. ETag / Conditional Requests

For frequently queried resources, use:

```text
ETag
If-None-Match
```

where supported.

This can reduce unnecessary API traffic.

---

# 67. API Retry Strategy

Retry:

```text
Network timeout
Temporary connection error
5xx
Rate limiting after honoring retry guidance
```

Do not blindly retry:

```text
401
403
404
Invalid input
Permission denied
```

Use bounded retries.

---

# 68. GitHub API Error Handling

Example:

```python
response = requests.get(
    url,
    headers=headers,
    timeout=20
)

if response.status_code == 404:
    raise RuntimeError(
        "Repository or resource not found"
    )

if response.status_code == 403:
    raise RuntimeError(
        "Permission denied or rate limited"
    )

response.raise_for_status()
```

In production, parse structured error details where useful.

---

# 69. GitHub Actions Failure Classification

Possible failures:

```text
WORKFLOW_NOT_FOUND
INVALID_INPUT
PERMISSION_DENIED
RUN_FAILED
RUN_CANCELLED
RUN_TIMEOUT
RUNNER_UNAVAILABLE
ARTIFACT_FAILURE
DEPLOYMENT_FAILURE
```

Python can map each to an operational action.

---

# 70. Retry vs Stop

Example policy:

```text
Runner unavailable -> retry
API timeout -> retry
5xx -> retry

Invalid workflow input -> stop
Permission denied -> stop
Security scan failure -> stop
Production approval rejected -> stop
```

Never use retry as a substitute for diagnosis.

---

# 71. Concurrency and Deployment Safety

Suppose:

```text
Run A -> production deployment
Run B -> production deployment
```

Without controls, they may race.

Use:

```yaml
concurrency:
  group: prod
  cancel-in-progress: false
```

or another documented deployment policy.

---

# 72. Cancel-in-Progress

For CI builds:

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

may be appropriate.

For production deployments:

```text
cancel-in-progress: false
```

is often safer because cancelling an active deployment can leave systems in an intermediate state.

The exact policy depends on the deployment mechanism.

---

# 73. Pull Request Automation

Python can inspect PRs for:

```text
Changed files
Labels
Review status
Checks
Approvals
Mergeability
```

Then enforce organization-specific rules.

Example:

```text
Terraform files changed
 |
 v
Require infrastructure review
```

---

# 74. Changed Files Detection

Python can query PR files and detect:

```text
*.tf
Dockerfile
Helm charts
Kubernetes manifests
Jenkinsfile
.github/workflows/*
```

Then route specialized checks.

Example:

```text
Terraform changed
 -> Terraform validation

Dockerfile changed
 -> Trivy/image validation

Workflow changed
 -> CI security review
```

---

# 75. Dynamic Validation

A Python preflight layer can decide:

```text
What changed?
```

then run:

```text
Relevant tests only
```

This can improve CI efficiency.

However, avoid skipping mandatory security and regression tests solely because a change appears small.

---

# 76. Monorepo Automation

For a monorepo:

```text
services/
  user/
  cart/
  payment/
  inventory/
```

Python can identify changed services.

Example:

```text
git diff
 |
 v
Python
 |
 +-- user changed
 +-- payment changed
 |
 v
Run targeted workflows
```

This can reduce unnecessary builds.

---

# 77. Matrix Workflows

GitHub Actions supports matrix jobs.

Example:

```yaml
strategy:
  matrix:
    python:
      - "3.11"
      - "3.12"
```

Python scripts should remain deterministic across matrix entries.

---

# 78. Python Matrix Testing

A CI workflow may test:

```text
Python 3.11
Python 3.12
```

or application variants.

Use:

```text
pytest
coverage
linting
security checks
```

The exact matrix should match production runtime support.

---

# 79. GitHub Actions Cache

Caching can improve performance for:

```text
pip
Docker layers
Terraform providers
Node dependencies
```

But cache keys should include relevant version/platform information.

Never cache secrets.

---

# 80. Python Dependency Cache

Typical cache inputs:

```text
Python version
requirements hash
OS
Architecture
```

A dependency lock/hash change should invalidate the cache.

---

# 81. Security: Third-Party Actions

A workflow may use:

```yaml
uses: some/action@...
```

Third-party actions execute code in your CI environment.

Prefer:

```text
Trusted actions
Pinned versions
Commit SHA pinning where organizational policy requires it
Minimal permissions
```

Review supply-chain risk.

---

# 82. Workflow Supply Chain Security

Protect:

```text
Actions
Dependencies
Docker base images
Python packages
Terraform providers
GitHub Actions runners
```

A compromised CI dependency can compromise:

```text
AWS credentials
ECR
Production infrastructure
GitHub repositories
```

---

# 83. Python Dependency Security

Use:

```text
Pinned dependencies
Dependency scanning
Trusted package indexes
Lock files where appropriate
```

Avoid:

```text
pip install arbitrary package
```

from untrusted sources in production workflows.

---

# 84. Secret Scanning

Your DevSecOps pipeline can include:

```text
Secret scanning
SAST
SCA
Trivy
Veracode
```

Python can aggregate results.

A security failure should block the appropriate promotion stage.

---

# 85. SonarQube Integration

Workflow:

```text
Checkout
 |
 v
Build/Test
 |
 v
SonarQube
 |
 v
Quality Gate
 |
 v
Continue
```

Python can query SonarQube status if an external orchestration layer needs it.

---

# 86. Trivy Integration

Workflow:

```text
Docker build
 |
 v
Trivy image scan
 |
 v
Policy
 |
 v
Push
```

Prefer scanning before production promotion.

Python can parse scanner output and classify:

```text
CRITICAL
HIGH
MEDIUM
LOW
```

according to policy.

---

# 87. Veracode Integration

If your organization uses Veracode:

```text
Build artifact
 |
 v
Veracode scan
 |
 v
Policy
 |
 v
Release
```

Python can retrieve scan status through the approved API/integration.

Do not expose security credentials.

---

# 88. Terraform Security in GitHub Actions

Workflow:

```text
Terraform code
 |
 v
fmt
 |
 v
validate
 |
 v
IaC security scan
 |
 v
plan
 |
 v
Python policy
 |
 v
approval
```

This aligns with the DevSecOps workflow already used in your notes.

---

# 89. GitHub Actions + Prometheus/Grafana

GitHub-hosted workflows are not normally your application's monitoring system.

But your automation platform can export:

```text
workflow duration
deployment count
deployment failures
release frequency
```

to an observability system.

Useful metrics:

```text
deployment_success_total
deployment_failure_total
workflow_duration_seconds
release_duration_seconds
```

---

# 90. GitHub Actions + ELK

Python can generate structured release logs:

```json
{
  "service": "payment",
  "environment": "prod",
  "commit": "abc123",
  "workflow": "deploy",
  "result": "success"
}
```

These logs can be centralized into ELK.

Do not include:

```text
tokens
secrets
credentials
private keys
```

---

# 91. Correlation ID

Use:

```text
release_id
```

across:

```text
GitHub
Python
Docker
ECR
Terraform
GitOps
ArgoCD
EKS
```

Example:

```text
release-2026-08-18-001
```

This is extremely useful during production incidents.

---

# 92. Deployment Timeline

Python can create:

```text
09:00 Build started
09:03 Tests passed
09:05 Trivy passed
09:06 Image pushed
09:07 GitOps PR created
09:10 PR merged
09:11 ArgoCD synced
09:13 Pods healthy
```

This gives an end-to-end release timeline.

---

# 93. GitHub Actions Logs

Logs should be:

```text
Structured
Useful
Searchable
Sanitized
```

Avoid excessive logging.

Never use:

```python
print(os.environ)
```

in CI.

---

# 94. Debugging Workflow Failures

Start with:

```text
1. Workflow run
2. Failed job
3. Failed step
4. Step logs
5. Runner
6. Credentials
7. External dependencies
8. Recent code/config changes
```

Then classify:

```text
Code failure
Infrastructure failure
CI platform failure
Credential failure
Dependency failure
```

---

# 95. Runner Disk Full

For self-hosted runners:

```bash
df -h
du -sh
```

Investigate:

```text
Docker layers
Workspaces
Caches
Artifacts
Temporary files
```

Use ephemeral runners and controlled cleanup where possible.

---

# 96. Runner Memory Pressure

Check:

```text
RAM
CPU
Concurrent jobs
Docker processes
Build tools
Terraform
Node/Python workloads
```

If builds are too heavy:

```text
Increase runner resources
Reduce concurrency
Split jobs
Optimize build
Use appropriate runner class
```

---

# 97. Workflow Permission Failure

Symptoms:

```text
403
Resource not accessible by integration
Permission denied
```

Check:

```text
permissions:
GITHUB_TOKEN scope
GitHub App permissions
Repository access
Environment protection
Branch rules
```

Do not immediately grant broad write access.

---

# 98. OIDC Failure

If AWS OIDC fails:

```text
Check workflow permissions
Check IAM trust policy
Check repository
Check branch/environment condition
Check audience
Check AWS role ARN
```

The first question should be:

```text
Which identity is GitHub actually presenting?
```

---

# 99. GitOps PR Automation Failure

If Python cannot create/update the GitOps PR:

```text
Check GitHub token/App permissions
Check repository
Check branch
Check existing PR
Check merge conflicts
Check branch protection
```

Do not bypass branch protection automatically.

---

# 100. ArgoCD Sync Failure

GitHub Actions may be successful while ArgoCD fails.

Check:

```text
GitOps commit
ArgoCD application
Sync status
Health status
Kubernetes events
Image availability
Manifest errors
```

The deployment authority remains ArgoCD.

---

# 101. ImagePullBackOff After GitHub Actions Deployment

Check:

```text
ECR image exists
Image digest correct
EKS node IAM permissions
ECR authentication
Image architecture
Repository URI
Tag/digest
```

A successful Docker push does not guarantee Kubernetes can pull the image.

---

# 102. Wrong Image Deployed

Possible causes:

```text
Mutable tag
GitOps update failed
Wrong repository
Wrong environment
Wrong digest
ArgoCD revision mismatch
```

Production should use:

```text
Git SHA
+
immutable digest
```

where practical.

---

# 103. Rollback

If application deployment fails:

```text
Known-good GitOps revision
 |
 v
ArgoCD
 |
 v
EKS
```

or revert the GitOps commit.

Do not rebuild the old application unless necessary.

The goal is to redeploy the exact known-good artifact.

---

# 104. GitHub Actions Production Checklist

```text
[ ] Workflows stored in Git
[ ] Workflow permissions explicitly defined
[ ] Minimal GITHUB_TOKEN permissions
[ ] GitHub App considered for external automation
[ ] OIDC used for AWS where appropriate
[ ] Long-lived cloud keys avoided
[ ] Environment protection configured
[ ] Production approvals configured
[ ] Concurrency configured
[ ] Inputs validated
[ ] Shell injection prevented
[ ] Third-party actions reviewed
[ ] Dependencies pinned/controlled
[ ] Secrets never printed
[ ] Self-hosted runners isolated
[ ] Ephemeral runners considered
[ ] Runner disk monitored
[ ] Runner memory monitored
[ ] API polling bounded
[ ] API retries use backoff
[ ] Duplicate workflows prevented
[ ] Release IDs used
[ ] Docker image digest captured
[ ] Trivy/security gates enabled
[ ] Terraform plan reviewed
[ ] GitOps ownership preserved
[ ] ArgoCD remains deployment authority
[ ] Post-deployment verification enabled
[ ] Rollback strategy documented
[ ] Artifacts retained appropriately
[ ] Sensitive artifacts protected
[ ] Audit trail available
[ ] Workflow failures observable
```

---

# 105. Interview Questions

## Q1. Why use Python with GitHub Actions?

GitHub Actions provides workflow orchestration.

Python is useful for:

```text
API integration
Complex validation
Docker automation
Terraform orchestration
AWS validation
GitOps updates
Release reporting
```

I avoid using Python to duplicate native GitHub Actions functionality unnecessarily.

---

## Q2. How do you trigger a GitHub Actions workflow from Python?

I use the GitHub Actions workflow dispatch API.

The process is:

```text
Authenticate
 |
 v
Validate repository/workflow/ref
 |
 v
POST workflow_dispatch
 |
 v
Identify the resulting run
 |
 v
Poll/reconcile status
 |
 v
Return result
```

I use a release ID or commit SHA to avoid confusing the run with another concurrent execution.

---

## Q3. How do you authenticate Python to GitHub?

Depending on the use case:

```text
GITHUB_TOKEN
GitHub App installation token
Fine-grained token
```

For organization-level automation, I generally prefer a GitHub App because it provides a dedicated machine identity and fine-grained permissions.

---

## Q4. How do you authenticate GitHub Actions to AWS?

I prefer GitHub OIDC with an AWS IAM role instead of long-lived AWS access keys.

The IAM trust policy should restrict:

```text
Repository
Organization
Branch/environment
Workflow context
```

as appropriate.

---

## Q5. How do you prevent duplicate workflow runs?

I use:

```text
GitHub concurrency
Release IDs
Commit SHA
Workflow run reconciliation
```

If a trigger request times out, I check existing runs before retrying.

---

## Q6. How do you secure a GitHub Actions workflow?

I use:

```text
Least-privilege permissions
OIDC
Environment approvals
Protected branches
Trusted/pinned actions
Secret protection
Runner isolation
Dependency scanning
Concurrency
```

---

## Q7. How would you deploy an application to EKS using GitHub Actions and ArgoCD?

I would use:

```text
GitHub Actions
 -> build
 -> test
 -> scan
 -> push ECR
 -> update GitOps repository
 -> ArgoCD
 -> EKS
```

ArgoCD remains the Kubernetes deployment authority.

---

## Q8. Why not use `kubectl apply` directly from GitHub Actions?

If ArgoCD owns the application, direct `kubectl apply` creates two competing sources of desired state.

I prefer:

```text
Git change
 -> ArgoCD
 -> Kubernetes
```

---

## Q9. What happens if the GitHub API returns 429?

I would respect the rate-limit information and retry with bounded backoff.

I would also reduce unnecessary polling.

---

## Q10. How do you troubleshoot a GitHub Actions job stuck in queued state?

I would check:

```text
Runner availability
Runner labels
Concurrency
Environment approvals
Repository capacity
Self-hosted runner status
```

I would distinguish queue delay from actual workflow failure.

---

# 106. Scenario-Based Interview Questions

## Scenario 1 — GitHub Workflow Trigger API Timed Out

### Strong Answer

I would not trigger the workflow again immediately.

I would:

```text
Check recent workflow runs
Match repository
Match workflow
Match ref
Match commit
Match release ID
```

If the original run exists, continue monitoring it.

---

## Scenario 2 — GitHub Actions Has AWS Admin Access

### Strong Answer

I would treat this as a security issue.

I would migrate to:

```text
OIDC
+
least-privilege IAM role
```

and restrict the trust policy to the intended repository/environment.

---

## Scenario 3 — Production Workflow Can Be Started by Any Developer

### Strong Answer

I would use:

```text
GitHub environment protection
Required reviewers
Branch protection
Deployment permissions
```

Python validation is useful, but authorization should be enforced at the platform level.

---

## Scenario 4 — Two Production Deployments Start Simultaneously

### Strong Answer

I would use GitHub Actions concurrency:

```yaml
concurrency:
  group: production-deployment
  cancel-in-progress: false
```

Then investigate the release process to ensure only one deployment can modify production at a time.

---

## Scenario 5 — Workflow SUCCESS but Pods Are Unhealthy

### Strong Answer

I would separate CI success from application health.

I would verify:

```text
GitOps revision
ArgoCD sync
Kubernetes rollout
Pod status
Events
Application health
```

If required, roll back the known-good GitOps revision.

---

## Scenario 6 — Self-Hosted Runner Is Compromised

### Strong Answer

I would:

```text
Isolate runner
Revoke exposed credentials
Rotate secrets
Investigate logs
Check persistence
Rebuild runner
Review workflows
Audit permissions
```

For sensitive workloads, I would move toward ephemeral isolated runners.

---

## Scenario 7 — Trivy Finds a Critical Vulnerability

### Strong Answer

The workflow should stop before production promotion according to policy.

I would:

```text
Identify vulnerability
Check exploitability
Check fixed version
Update dependency/base image
Rebuild
Rescan
```

I would not simply ignore the scan to make the pipeline green unless there is an approved exception process.

---

## Scenario 8 — GitOps Repository Has a Merge Conflict

### Strong Answer

Python should not blindly overwrite the repository.

I would:

```text
Fetch latest branch
Detect conflict
Stop automation
Notify owner
Resolve through normal review process
```

For automated changes, the update operation should be designed to minimize conflicting concurrent writes.

---

## Scenario 9 — Wrong ECR Image Is Deployed

### Strong Answer

I would compare:

```text
Git SHA
Image tag
Image digest
GitOps commit
ArgoCD revision
Running image
```

This establishes where the mismatch occurred.

---

## Scenario 10 — GitHub API Works Locally but Fails in Actions

### Strong Answer

I would compare:

```text
Token
Permissions
Repository
Environment
API URL
Network
GitHub App installation
GITHUB_TOKEN scope
```

The most likely issue is often different authentication or permissions between local and CI environments.

---

# 107. Senior-Level GitHub Actions Automation Thinking

A basic question is:

```text
How do I run a Python script in GitHub Actions?
```

A senior-level question is:

```text
How do I create a reliable release automation system
that can authenticate securely, trigger workflows,
avoid duplicate executions, publish immutable artifacts,
update GitOps, deploy through ArgoCD, and verify EKS?
```

The production flow becomes:

```text
Developer
   |
   v
GitHub
   |
   v
Workflow
   |
   v
Python Validation
   |
   v
Build
   |
   v
Test + Security
   |
   v
Docker
   |
   v
ECR
   |
   v
GitOps PR
   |
   v
Review
   |
   v
Merge
   |
   v
ArgoCD
   |
   v
EKS
   |
   v
Health Verification
   |
   v
Release Report
```

---

# 108. Final Production Architecture

```text
                           GitHub
                             |
              +--------------+--------------+
              |                             |
              v                             v
       Application Repo                GitOps Repo
              |                             |
              v                             ^
       GitHub Actions                      |
              |                             |
        +-----+-----+                       |
        |           |                       |
        v           v                       |
      Python     Security                   |
        |           |                       |
        v           v                       |
     Docker      SonarQube                  |
        |        Trivy/                    |
        |        Veracode                   |
        v                                   |
       ECR                                  |
        |                                   |
        +----------------+                  |
                         |                  |
                         v                  |
                    Release Artifact        |
                         |                  |
                         v                  |
                    GitOps Update ---------+
                         |
                         v
                       ArgoCD
                         |
                         v
                        EKS
                         |
            +------------+------------+
            |                         |
            v                         v
        Prometheus                   ELK
            |                         |
            v                         v
         Grafana                   Kibana
```

---

# 109. Key Takeaways

### 1. GitHub Actions is the CI/CD execution platform

Python should extend it rather than duplicate it.

### 2. Use secure machine identities

Prefer:

```text
GitHub App
OIDC
Short-lived credentials
```

over long-lived secrets.

### 3. Use least privilege

```text
GITHUB_TOKEN
AWS IAM
GitHub App
```

should all have minimum required permissions.

### 4. Build once, promote the same artifact

Use:

```text
Git SHA
+
Image digest
```

to identify releases.

### 5. Protect production

Use:

```text
Environment approvals
Branch protection
Concurrency
OIDC
IAM
```

### 6. Design for API failure

Expect:

```text
Timeout
429
5xx
Runner delay
Duplicate trigger
```

### 7. Maintain GitOps boundaries

```text
GitHub Actions -> build/release
Python -> automation/integration
ArgoCD -> Kubernetes desired state
EKS -> runtime
```

### 8. Verify deployment

A successful workflow does not necessarily mean a healthy application.

### 9. Correlate everything

Use:

```text
Release ID
Git SHA
Workflow run
Image digest
GitOps revision
ArgoCD revision
```

### 10. Think beyond the workflow file

The goal is not:

```text
GitHub Actions -> Python
```

The goal is:

```text
Validate
 -> Build
 -> Secure
 -> Publish
 -> Promote
 -> Deploy
 -> Verify
 -> Report
```

---

# 110. Final Mental Model

Remember Python + GitHub Actions as:

```text
                       GitHub
                          |
                          v
                   GitHub Actions
                          |
                          v
                       Python
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
       Docker         Terraform          AWS
          |               |               |
          v               v               v
         ECR              AWS          Validation
          |               |
          +-------+-------+
                  |
                  v
             GitOps Repo
                  |
                  v
                ArgoCD
                  |
                  v
                 EKS
                  |
        +---------+---------+
        |                   |
        v                   v
    Prometheus             ELK
        |                   |
        v                   v
     Grafana              Kibana
```

The production principle is:

> **Use GitHub Actions as the CI/CD execution layer, Python for reliable automation and integration, secure AWS identity through OIDC, Docker/ECR for immutable artifacts, Terraform for infrastructure, GitOps for application desired state, ArgoCD for Kubernetes reconciliation, and observability to verify the complete delivery lifecycle.**
