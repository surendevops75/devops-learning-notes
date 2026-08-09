# GitHub Actions Reusable Workflows

Reusable workflows allow you to define a workflow once and call it from other workflows.

They are useful for standardizing CI/CD across:

```text
Multiple Repositories
Multiple Microservices
Multiple Environments
Multiple Teams
Multiple Applications
```

Instead of duplicating the same workflow everywhere:

```text
Repository A
 └── Build + Security + Deploy

Repository B
 └── Build + Security + Deploy

Repository C
 └── Build + Security + Deploy
```

you can centralize the workflow:

```text
Reusable Workflow
       |
 ┌─────┼─────┐
 ↓     ↓     ↓
Repo A Repo B Repo C
```

---

# Why Reusable Workflows Matter

Without reusable workflows:

```text
Service A
 └── 300 lines YAML

Service B
 └── 300 lines YAML

Service C
 └── 300 lines YAML
```

Problems:

```text
Duplication
Maintenance
Inconsistent Security
Different Deployment Logic
Harder Updates
```

With reusable workflows:

```text
Reusable CI Workflow
        |
        ├── Service A
        ├── Service B
        └── Service C
```

One central workflow can provide common logic.

---

# Reusable Workflow vs Composite Action

These are different concepts.

### Reusable Workflow

Reuses:

```text
Entire workflow
Jobs
Steps
Job dependencies
Runners
Environments
Permissions
Secrets
Inputs
Outputs
```

### Composite Action

Reuses:

```text
A collection of steps
```

Conceptually:

```text
Reusable Workflow
    ↓
Multiple Jobs
    ↓
Multiple Steps
```

while:

```text
Composite Action
    ↓
Multiple Steps
```

---

# Basic Reusable Workflow

A reusable workflow uses:

```yaml
on:
  workflow_call:
```

Example:

```yaml
name: Reusable Build

on:
  workflow_call:

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: |
          ./build.sh
```

This workflow can be called by another workflow.

---

# Calling a Reusable Workflow

Example:

```yaml
jobs:

  build:

    uses: ./.github/workflows/reusable-build.yml
```

Flow:

```text
Caller Workflow
      |
      ↓
Reusable Workflow
      |
      ↓
Build Job
```

---

# Repository Structure

A common structure:

```text
.github/
└── workflows/
    ├── ci.yml
    ├── deploy.yml
    ├── reusable-build.yml
    ├── reusable-security.yml
    └── reusable-deploy.yml
```

The reusable workflows are stored under:

```text
.github/workflows/
```

---

# Local Reusable Workflow

Example:

```yaml
jobs:

  build:

    uses: ./.github/workflows/reusable-build.yml
```

This means the reusable workflow exists in the same repository.

---

# Reusable Workflow from Another Repository

You can call a reusable workflow from another repository.

Example:

```yaml
jobs:

  build:

    uses: my-org/platform-workflows/.github/workflows/build.yml@v1
```

The reference includes:

```text
Repository
+
Workflow Path
+
Version Reference
```

---

# Version Pinning

Prefer a controlled version:

```yaml
uses: my-org/platform-workflows/.github/workflows/build.yml@v1
```

or a specific immutable reference where appropriate.

Avoid blindly depending on a moving branch for critical production workflow logic.

---

# Why Version Pinning Matters

Suppose:

```text
Application A
Application B
Application C
```

all use:

```yaml
@main
```

If the shared workflow changes:

```text
Shared Workflow
      |
      ↓
All applications
```

could immediately receive the change.

Versioning provides controlled adoption:

```text
v1
 ↓
Applications using v1

v2
 ↓
Applications migrate when ready
```

---

# Reusable Workflow Inputs

Reusable workflows can accept inputs.

Example:

```yaml
name: Reusable Build

on:
  workflow_call:

    inputs:

      node-version:
        required: true
        type: string

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}

      - name: Build
        run: |
          npm ci
          npm run build
```

---

# Calling with Inputs

```yaml
jobs:

  build:

    uses: ./.github/workflows/reusable-build.yml

    with:
      node-version: '20'
```

Flow:

```text
Caller
  |
  └── node-version = 20
          |
          ↓
Reusable Workflow
          |
          ↓
Node 20
```

---

# Input Types

Reusable workflow inputs support explicit types such as:

```text
string
boolean
number
```

Example:

```yaml
inputs:

  environment:
    required: true
    type: string

  run-tests:
    required: true
    type: boolean

  replicas:
    required: false
    type: number
```

---

# Boolean Input

Reusable workflow:

```yaml
inputs:

  run-tests:
    required: false
    default: true
    type: boolean
```

Caller:

```yaml
with:
  run-tests: true
```

Then:

```yaml
if: ${{ inputs.run-tests }}
```

---

# String Input

```yaml
inputs:

  service:
    required: true
    type: string
```

Caller:

```yaml
with:
  service: catalogue
```

---

# Number Input

```yaml
inputs:

  replicas:
    required: false
    default: 2
    type: number
```

Caller:

```yaml
with:
  replicas: 3
```

---

# Default Inputs

Example:

```yaml
inputs:

  environment:
    required: false
    default: qa
    type: string
```

If the caller does not provide the input:

```text
environment = qa
```

---

# Required Inputs

Example:

```yaml
inputs:

  service:
    required: true
    type: string
```

The caller must provide:

```yaml
with:
  service: catalogue
```

---

# Secrets in Reusable Workflows

Reusable workflows can receive secrets.

Example:

```yaml
on:
  workflow_call:

    secrets:

      deployment-token:
        required: true
```

Caller:

```yaml
jobs:

  deploy:

    uses: ./.github/workflows/reusable-deploy.yml

    secrets:
      deployment-token: ${{ secrets.DEPLOYMENT_TOKEN }}
```

---

# `secrets: inherit`

When appropriate, a caller can pass secrets using:

```yaml
secrets: inherit
```

Example:

```yaml
jobs:

  deploy:

    uses: ./.github/workflows/reusable-deploy.yml

    secrets: inherit
```

Use this carefully.

Do not automatically expose every available secret to a reusable workflow unless there is a clear trust boundary and need.

---

# Secret Principle

Prefer:

```text
Only required secrets
```

instead of:

```text
All repository secrets
```

when practical.

For AWS deployments, prefer:

```text
OIDC
+
IAM Role
```

instead of long-lived AWS credentials.

---

# Permissions

Reusable workflows can define permissions.

Example:

```yaml
permissions:
  contents: read
```

For AWS OIDC:

```yaml
permissions:
  contents: read
  id-token: write
```

Use least privilege.

---

# Why Permissions Matter

Reusable workflows can become centralized security boundaries.

If many repositories call the same workflow:

```text
Caller Repositories
        |
        ↓
Reusable Workflow
        |
        ↓
Cloud / Deployment
```

A permission mistake can affect many applications.

---

# Reusable Workflow Outputs

Reusable workflows can expose outputs.

Example:

```yaml
on:
  workflow_call:

    outputs:

      image-tag:
        description: Built image tag
        value: ${{ jobs.build.outputs.image-tag }}
```

The job provides the output:

```yaml
jobs:

  build:

    outputs:
      image-tag: ${{ steps.image.outputs.tag }}

    steps:

      - name: Set Image Tag
        id: image
        run: |
          echo "tag=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"
```

---

# Consuming Workflow Output

Caller:

```yaml
jobs:

  build:

    uses: ./.github/workflows/reusable-build.yml

  deploy:

    needs: build

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
        run: |
          echo "Deploying $IMAGE_TAG"
```

Flow:

```text
Reusable Build
      |
      ↓
Image Tag
      |
      ↓
Caller
      |
      ↓
Deploy
```

---

# Outputs vs Artifacts

Use outputs for:

```text
Small values
Image Tag
Image Digest
Status
Version
Identifier
```

Use artifacts for:

```text
Reports
Build Files
Logs
Terraform Plan
Packages
Large Files
```

---

# Reusable Workflow Architecture

```text
Caller Workflow
      |
      ↓
Reusable Build
      |
      ↓
Reusable Security
      |
      ↓
Reusable Deploy
```

This creates centralized CI/CD capabilities.

---

# Production CI/CD Architecture

For a microservices platform:

```text
Service Repository
       |
       ↓
Reusable Build
       |
       ↓
Reusable Security
       |
       ↓
Reusable Test
       |
       ↓
Reusable Image Build
       |
       ↓
Reusable Deployment
```

Each application uses the same organizational standards.

---

# Reusable Build Workflow

Example:

```yaml
name: Reusable Build

on:
  workflow_call:

    inputs:

      service:
        required: true
        type: string

      node-version:
        required: false
        default: '20'
        type: string

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: npm

      - name: Install
        run: |
          npm ci

      - name: Test
        run: |
          npm test

      - name: Build
        run: |
          npm run build
```

---

# Caller Workflow

```yaml
name: Catalogue CI

on:
  push:
    branches:
      - main

jobs:

  build:

    uses: ./.github/workflows/reusable-build.yml

    with:
      service: catalogue
      node-version: '20'
```

---

# Reusable Security Workflow

Example:

```yaml
name: Reusable Security

on:
  workflow_call:

    inputs:

      image:
        required: true
        type: string

jobs:

  security:

    runs-on: ubuntu-latest

    steps:

      - name: Trivy Scan
        env:
          IMAGE: ${{ inputs.image }}
        run: |
          trivy image "$IMAGE"
```

---

# Security Workflow Caller

```yaml
jobs:

  security:

    needs: build

    uses: ./.github/workflows/reusable-security.yml

    with:
      image: ${{ needs.build.outputs.image }}
```

---

# Reusable Deployment Workflow

Example:

```yaml
name: Reusable Deployment

on:
  workflow_call:

    inputs:

      service:
        required: true
        type: string

      environment:
        required: true
        type: string

      image:
        required: true
        type: string

jobs:

  deploy:

    runs-on: ubuntu-latest

    environment:
      name: ${{ inputs.environment }}

    steps:

      - name: Deploy
        env:
          SERVICE: ${{ inputs.service }}
          IMAGE: ${{ inputs.image }}
        run: |
          echo "Deploying $SERVICE"
          echo "Image: $IMAGE"
```

---

# Reusable Deployment Caller

```yaml
jobs:

  deploy:

    needs:
      - build
      - security

    uses: ./.github/workflows/reusable-deploy.yml

    with:
      service: catalogue
      environment: production
      image: ${{ needs.build.outputs.image }}
```

---

# Reusable Workflow + Environment

Production deployments can use:

```yaml
environment:
  name: ${{ inputs.environment }}
```

This allows the caller to specify:

```text
qa
uat
production
```

However, production authorization should still be enforced through GitHub Environment protection and appropriate permissions.

---

# Reusable Workflow + Concurrency

A reusable deployment workflow can define concurrency.

Example:

```yaml
jobs:

  deploy:

    concurrency:
      group: deploy-${{ inputs.service }}-${{ inputs.environment }}
      cancel-in-progress: false

    environment:
      name: ${{ inputs.environment }}

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

This creates consistent deployment serialization.

---

# Reusable Workflow + Timeout

Example:

```yaml
jobs:

  deploy:

    timeout-minutes: 30

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

Centralizing the timeout helps standardize operational behavior.

---

# Reusable Workflow + Permissions

Example:

```yaml
permissions:
  contents: read
```

For AWS OIDC:

```yaml
permissions:
  contents: read
  id-token: write
```

Avoid excessive permissions.

---

# Reusable Workflow + OIDC

Production AWS deployment:

```text
Caller
  |
  ↓
Reusable Deployment Workflow
  |
  ↓
GitHub OIDC
  |
  ↓
AWS IAM Role
  |
  ↓
EKS
```

This allows centralized AWS deployment logic without distributing long-lived AWS credentials.

---

# Reusable Workflow + EKS

Example architecture:

```text
Microservice Repository
       |
       ↓
Reusable Build
       |
       ↓
Reusable Security
       |
       ↓
ECR
       |
       ↓
Reusable Deployment
       |
       ↓
Helm / GitOps
       |
       ↓
ArgoCD
       |
       ↓
EKS
```

---

# Reusable Workflow + Helm

Example:

```yaml
- name: Helm Deploy
  env:
    SERVICE: ${{ inputs.service }}
    IMAGE_TAG: ${{ inputs.image-tag }}
  run: |
    helm upgrade --install "$SERVICE" \
      "./helm/$SERVICE" \
      --namespace production \
      --wait \
      --timeout 15m \
      --atomic \
      --set image.tag="$IMAGE_TAG"
```

A real production implementation should validate all deployment inputs and ensure the image reference is immutable/trusted.

---

# Reusable Workflow + GitOps

Instead of deploying directly:

```text
Reusable Deployment
       |
       ↓
Update GitOps Repository
       |
       ↓
ArgoCD
       |
       ↓
EKS
```

This fits a GitOps architecture.

---

# GitOps Reusable Workflow

Example concept:

```yaml
jobs:

  update-gitops:

    steps:

      - name: Update Manifest
        run: |
          ./update-image.sh \
            "${{ inputs.service }}" \
            "${{ inputs.image }}"
```

The actual implementation should use a controlled Git identity, branch strategy, permissions, and commit validation.

---

# Reusable Workflow + DevSecOps

Centralize security:

```text
Reusable Security Workflow
       |
       ├── SonarQube
       ├── Trivy
       └── Veracode
```

Then:

```text
Service A → Security
Service B → Security
Service C → Security
```

All services follow the same security baseline.

---

# Why Centralized Security Matters

Without reusable workflows:

```text
Service A
 └── Trivy configuration A

Service B
 └── Trivy configuration B

Service C
 └── Trivy configuration C
```

Possible result:

```text
Different policies
Different thresholds
Different tool versions
Different configurations
```

Reusable workflows reduce this drift.

---

# Security Gate

Example:

```text
Build
  |
  ↓
SonarQube
  |
  ↓
Trivy
  |
  ↓
Veracode
  |
  ↓
Promotion
```

The reusable workflow should return clear success/failure signals.

---

# Reusable Workflow + Outputs

Example:

```text
Build Workflow
    |
    ├── image-tag
    ├── image-digest
    └── artifact-version
```

Then:

```text
Security Workflow
    |
    └── scan result

Deployment Workflow
    |
    └── consumes approved image
```

---

# Image Digest Output

For production, an image digest is preferable to a mutable tag.

Example:

```text
sha256:abc123...
```

Reusable workflow output:

```yaml
outputs:

  image-digest:
    value: ${{ jobs.build.outputs.image-digest }}
```

Caller:

```yaml
${{ needs.build.outputs.image-digest }}
```

---

# Why Digest Matters

Tag:

```text
catalogue:latest
```

can move.

Digest:

```text
sha256:abc123...
```

identifies a specific image.

Production deployment should prefer immutable references.

---

# Reusable Workflow + Artifact

A reusable workflow can upload an artifact:

```yaml
- name: Upload Report
  uses: actions/upload-artifact@v4
  with:
    name: security-report
    path: reports/
```

The caller can use the workflow's outputs for small values and artifacts for files.

---

# Reusable Workflow + Matrix

Caller can use a matrix with reusable workflows in appropriate job-level patterns.

Conceptually:

```text
Matrix
 ├── user
 ├── catalogue
 ├── cart
 └── orders
       |
       ↓
Reusable Build
```

This allows the same workflow logic to process multiple services.

---

# Microservices Architecture

Example:

```yaml
strategy:
  matrix:
    service:
      - user
      - catalogue
      - cart
      - orders
```

Each service can call the same reusable build/deployment logic where the workflow design supports it.

---

# Reusable Workflow + Inputs Validation

Never blindly trust inputs.

Example:

```yaml
inputs:

  environment:
    required: true
    type: string
```

Then validate:

```bash
case "$ENVIRONMENT" in
  qa|uat|production)
    ;;
  *)
    echo "Invalid environment"
    exit 1
    ;;
esac
```

This is especially important for privileged deployment workflows.

---

# Service Input Validation

Example:

```bash
case "$SERVICE" in
  user|catalogue|cart|orders|payment|inventory|notification)
    ;;
  *)
    echo "Unsupported service"
    exit 1
    ;;
esac
```

For larger platforms, prefer controlled configuration or allowlists rather than constructing arbitrary paths or commands from untrusted input.

---

# Reusable Workflow Security Boundary

A reusable workflow may have access to:

```text
Secrets
Cloud Credentials
Production Environment
Deployment Permissions
```

Therefore:

```text
Reusable Workflow
        |
        ↓
Security Boundary
```

Treat changes to shared workflows as high-impact changes.

---

# Trust Model

```text
Application Repository
        |
        ↓
Reusable Workflow
        |
        ↓
Production Infrastructure
```

If the reusable workflow is compromised:

```text
Many repositories
        |
        ↓
Potential impact
```

Therefore protect:

```text
Workflow Repository
Branch
Tags
Review Process
Permissions
Secrets
OIDC Trust
```

---

# Pin Shared Workflow Versions

Example:

```yaml
uses: my-org/platform-workflows/.github/workflows/deploy.yml@v1
```

Better than blindly using:

```yaml
uses: my-org/platform-workflows/.github/workflows/deploy.yml@main
```

for critical production workflows.

For higher assurance, use an immutable commit SHA where your organizational workflow supports that approach.

---

# Reusable Workflow Repository

Enterprise structure:

```text
platform-workflows/
│
└── .github/
    └── workflows/
        ├── reusable-build.yml
        ├── reusable-security.yml
        ├── reusable-test.yml
        ├── reusable-docker.yml
        ├── reusable-terraform.yml
        └── reusable-deploy.yml
```

---

# Application Repository

Example:

```text
catalogue/
│
├── src/
├── Dockerfile
├── helm/
└── .github/
    └── workflows/
        └── ci.yml
```

Application workflow:

```yaml
jobs:

  build:

    uses: my-org/platform-workflows/.github/workflows/reusable-build.yml@v1
```

---

# Central Platform Model

```text
                 Platform Workflows
                        |
        ┌───────────────┼───────────────┐
        ↓               ↓               ↓
      Build          Security         Deploy
        |               |               |
        └───────────────┼───────────────┘
                        ↓
                 Application Repos
              ┌─────────┼─────────┐
              ↓         ↓         ↓
          Catalogue    Orders    Payment
```

---

# Reusable Workflow Versioning

Example:

```text
v1
v2
v3
```

Migration:

```text
Applications
   |
   ├── v1
   ├── v1
   └── v2
```

Teams can migrate gradually.

---

# Breaking Changes

Suppose:

```text
v1:
input = image-tag
```

v2 changes it to:

```text
input = image-digest
```

Do not silently break every consumer.

Create a new version:

```text
v1 → existing consumers
v2 → migrated consumers
```

---

# Reusable Workflow Documentation

Each reusable workflow should document:

```text
Purpose
Inputs
Outputs
Secrets
Permissions
Environment
Expected Runner
Dependencies
Version
Failure Behavior
Security Requirements
```

Example:

```text
Reusable Deploy Workflow

Inputs:
service
environment
image-digest

Secrets:
none

Permissions:
contents: read
id-token: write

Output:
deployment-status
```

---

# Reusable Workflow Interface

Think of a reusable workflow like an API.

```text
Inputs
   ↓
Reusable Workflow
   ↓
Outputs
```

For example:

```text
Input:
service=catalogue

Input:
environment=production

Input:
image-digest=sha256:abc...

        ↓

Reusable Deploy

        ↓

Output:
status=success
```

---

# Stable Interface

Avoid unnecessary changes to:

```text
Input Names
Input Types
Output Names
Required Secrets
Behavior
```

Treat the interface as a contract.

---

# Reusable Workflow + JIRA

A centralized production workflow can validate:

```text
JIRA Ticket
Change Request
Approval
Deployment Window
Commit SHA
Artifact/Image
```

Example architecture:

```text
Caller
  |
  ↓
Reusable Production Workflow
  |
  ├── JIRA Validation
  ├── SHA Validation
  ├── Security Validation
  ├── Test Validation
  └── Deployment
```

This provides consistent production controls.

---

# JIRA Validation Output

Example:

```text
approved=true
```

Then:

```yaml
if: ${{ needs.validation.outputs.approved == 'true' }}
```

The caller or reusable workflow can use this output to control promotion.

---

# Reusable Workflow + Change Management

Production:

```text
Input JIRA Ticket
      |
      ↓
Validate Change Request
      |
      ↓
Check Approved
      |
      ↓
Check Deployment Window
      |
      ↓
Check Commit SHA
      |
      ↓
Security/Test Results
      |
      ↓
GitHub Environment
      |
      ↓
Approval
      |
      ↓
Deploy
```

---

# Reusable Workflow + Production Environment

Example:

```yaml
jobs:

  deploy:

    environment:
      name: production

    concurrency:
      group: production-${{ inputs.service }}
      cancel-in-progress: false
```

This combines:

```text
Centralized deployment logic
+
Environment protection
+
Deployment serialization
```

---

# Reusable Workflow + Timeout

Example:

```yaml
jobs:

  deploy:

    timeout-minutes: 30
```

Centralizing this prevents each application from choosing arbitrary deployment timeout behavior.

---

# Reusable Workflow + Failure Handling

Example:

```yaml
- name: Collect Diagnostics
  if: ${{ failure() }}
  run: |
    ./collect-diagnostics.sh
```

Or:

```yaml
- name: Upload Diagnostics
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: deployment-diagnostics
    path: diagnostics/
```

---

# Reusable Workflow + Rollback

For Helm:

```text
Deploy
   |
   ↓
Health Check
   |
 ┌─┴─┐
 ↓   ↓
OK  FAIL
 |    |
 ↓    ↓
Done Rollback
```

A centralized deployment workflow can standardize this behavior.

---

# Reusable Workflow + GitOps Rollback

For ArgoCD:

```text
GitOps Repository
       |
       ↓
ArgoCD
       |
       ↓
EKS
       |
       ↓
Health Failure
       |
       ↓
GitOps Rollback / Revert
```

The exact rollback strategy should match your GitOps operating model.

---

# Reusable Workflow + Terraform

A centralized Terraform workflow can standardize:

```text
Format
Validate
Security Scan
Plan
Approval
Apply
```

Example:

```text
Reusable Terraform Workflow
          |
          ├── terraform fmt
          ├── terraform validate
          ├── security
          ├── plan
          └── apply
```

---

# Terraform Production Workflow

```text
Terraform Plan
      |
      ↓
Artifact
      |
      ↓
Review
      |
      ↓
Reusable Apply
      |
      ↓
Terraform State
      |
      ↓
AWS Infrastructure
```

Concurrency should protect the target environment/state.

---

# Reusable Workflow + Docker

Centralized Docker workflow:

```text
Checkout
   ↓
Buildx
   ↓
Build Cache
   ↓
Security
   ↓
Push
   ↓
ECR
   ↓
Image Digest
```

Output:

```text
image-digest
```

---

# Reusable Docker Workflow

Example interface:

```text
Inputs:
service
image-name
dockerfile
context
```

Outputs:

```text
image-tag
image-digest
```

This allows all microservices to follow a common image-building process.

---

# Reusable Workflow + DevSecOps

Recommended centralized flow:

```text
Build
  ↓
Unit Tests
  ↓
SonarQube
  ↓
Docker Build
  ↓
Trivy
  ↓
Veracode
  ↓
ECR
  ↓
Promotion
```

Every service receives the same baseline controls.

---

# Enterprise DevSecOps Architecture

```text
                Application Repositories
                         |
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
     Catalogue         Orders           Payment
        |                |                |
        └────────────────┼────────────────┘
                         ↓
                Platform Workflows
                         |
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
     Build            Security           Deploy
       |                 |                 |
       ↓                 ↓                 ↓
     Docker          SonarQube           Helm
                       Trivy             GitOps
                     Veracode             |
                                          ↓
                                        ArgoCD
                                          |
                                          ↓
                                         EKS
```

---

# Reusable Workflow vs Workflow Template

These solve different problems.

### Reusable Workflow

Used at runtime:

```text
Caller
  ↓
Reusable Workflow
```

### Workflow Template

Used to help create a new workflow:

```text
Template
   ↓
New Repository
   ↓
Workflow copied/created
```

A template is not the same as a dynamically called reusable workflow.

---

# Reusable Workflow vs Composite Action

### Reusable Workflow

```text
Workflow
 ├── Job A
 ├── Job B
 └── Job C
```

### Composite Action

```text
Action
 ├── Step A
 ├── Step B
 └── Step C
```

Use a reusable workflow when you need to standardize jobs and pipeline structure.

Use a composite Action when you want to package reusable step logic.

---

# Reusable Workflow vs Script

### Script

```text
bash deploy.sh
```

Good for:

```text
Command Logic
```

### Composite Action

```text
Reusable step sequence
```

### Reusable Workflow

```text
Reusable CI/CD stage or process
```

Choose the correct abstraction.

---

# Production Design Principle

Do not put everything into one giant reusable workflow.

Bad:

```text
One Workflow
 ├── Build
 ├── Test
 ├── Security
 ├── Terraform
 ├── Docker
 ├── Deploy
 ├── Rollback
 └── Notifications
```

Better:

```text
Reusable Build
Reusable Security
Reusable Test
Reusable Docker
Reusable Terraform
Reusable Deploy
```

Compose them where appropriate.

---

# Reusable Workflow Composition

Conceptually:

```text
Application
    |
    ├── Build Workflow
    |
    ├── Security Workflow
    |
    ├── Test Workflow
    |
    └── Deploy Workflow
```

This keeps responsibilities clearer.

---

# Reusable Workflow Governance

For enterprise environments:

```text
Platform Team
    |
    ↓
Reusable Workflows
    |
    ↓
Application Teams
```

Platform team manages:

```text
Security Baseline
Runner Configuration
Tool Versions
Deployment Standards
Cloud Authentication
Production Controls
```

Application teams provide:

```text
Application-specific inputs
```

---

# Governance Benefits

Centralized workflows can enforce:

```text
Required Security Scans
Approved Actions
Approved Runners
OIDC
Least Privilege
Artifact Standards
Deployment Controls
```

---

# Governance Risk

Centralization also creates blast radius.

If a shared workflow has a bug:

```text
Shared Workflow
      |
      ↓
100 Repositories
      |
      ↓
Potential impact
```

Therefore use:

```text
Versioning
Testing
Review
Release Process
Rollback
```

for reusable workflow changes.

---

# Testing Reusable Workflows

Before promoting a shared workflow:

```text
Development
   ↓
Test Repository
   ↓
Integration Tests
   ↓
Security Review
   ↓
Version Release
   ↓
Application Adoption
```

Do not test major shared workflow changes directly in all production repositories.

---

# Reusable Workflow Release Process

```text
Change
  |
  ↓
Pull Request
  |
  ↓
Tests
  |
  ↓
Security
  |
  ↓
Review
  |
  ↓
Version
  |
  ↓
Release
  |
  ↓
Consumers Upgrade
```

---

# Reusable Workflow Security Checklist

```text
☐ Least privilege permissions
☐ Minimal secrets
☐ OIDC instead of long-lived cloud credentials
☐ Trusted runner
☐ Input validation
☐ Output validation
☐ Secure action versions
☐ Protected workflow repository
☐ Versioned releases
☐ Reviewed changes
☐ No secret logging
☐ No arbitrary command injection
☐ Production environment protection
```

---

# Input Injection Risk

Avoid constructing commands directly from untrusted inputs.

Risky pattern:

```yaml
run: |
  ./deploy.sh ${{ inputs.service }}
```

If the input is not trusted or validated, command construction can become dangerous.

Prefer:

```yaml
env:
  SERVICE: ${{ inputs.service }}

run: |
  ./deploy.sh "$SERVICE"
```

and validate:

```bash
case "$SERVICE" in
  user|catalogue|cart|orders)
    ;;
  *)
    echo "Invalid service"
    exit 1
    ;;
esac
```

---

# Reusable Workflow and Untrusted PRs

Be especially careful when a reusable workflow has:

```text
Production Permissions
Secrets
OIDC
Self-hosted Runners
```

and can be triggered by untrusted pull-request code.

Do not create a path where:

```text
Untrusted PR
      ↓
Privileged Reusable Workflow
      ↓
Production Access
```

---

# Reusable Workflow Permissions

A caller's permissions and the reusable workflow's requirements must be designed together.

For example:

```yaml
permissions:
  contents: read
  id-token: write
```

Only grant:

```text
id-token: write
```

where AWS/OIDC authentication is actually required.

---

# Production Caller Example

```yaml
name: Catalogue Production

on:
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:

  deploy:

    uses: my-org/platform-workflows/.github/workflows/deploy.yml@v1

    with:
      service: catalogue
      environment: production
      image-digest: sha256:abc123...

    secrets: inherit
```

For stronger security, pass only the secrets actually required instead of inheriting all secrets.

---

# Reusable Workflow Inputs and Secrets

A clear interface might be:

```text
Inputs:
service
environment
image-digest
jira-ticket

Secrets:
none

Permissions:
contents: read
id-token: write

Outputs:
deployment-status
```

This is cleaner than exposing unnecessary internal details.

---

# Production Promotion Interface

Example:

```text
INPUTS
 ├── service
 ├── environment
 ├── image-digest
 └── jira-ticket

VALIDATION
 ├── JIRA
 ├── SHA
 ├── Security
 └── Tests

AUTHORIZATION
 └── GitHub Environment

DEPLOYMENT
 ├── Helm
 └── GitOps

OUTPUT
 └── deployment-status
```

---

# Key Takeaways

Reusable workflows allow you to standardize complete workflow logic.

Basic trigger:

```yaml
on:
  workflow_call:
```

Caller:

```yaml
jobs:

  build:

    uses: ./.github/workflows/reusable-build.yml
```

Inputs:

```yaml
with:
  service: catalogue
```

Secrets:

```yaml
secrets:
  deployment-token: ${{ secrets.DEPLOYMENT_TOKEN }}
```

Outputs:

```yaml
needs.build.outputs.image-digest
```

Reusable workflows are ideal for:

```text
Build
Testing
Security
Docker
Terraform
Deployment
GitOps
Production Controls
```

For your DevSecOps architecture:

```text
Application
     |
     ↓
Reusable Build
     |
     ↓
Reusable Security
     |
     ↓
Reusable Docker
     |
     ↓
ECR
     |
     ↓
Reusable Deployment
     |
     ↓
GitOps / Helm
     |
     ↓
ArgoCD
     |
     ↓
EKS
```

The most important production principle is:

```text
Centralize common standards,
but keep application-specific configuration as inputs.
```

Treat reusable workflows as **shared infrastructure and security components**, not merely YAML shortcuts.

---

# Interview Questions

## Basic

1. What is a reusable workflow in GitHub Actions?
2. How do you create a reusable workflow?
3. What is `workflow_call`?
4. How do you call a reusable workflow?
5. What is the difference between a reusable workflow and a composite Action?
6. What are inputs in reusable workflows?
7. What are outputs in reusable workflows?
8. How do you pass secrets to a reusable workflow?
9. What is `secrets: inherit`?
10. Where are reusable workflows normally stored?

## Intermediate

11. How do you pass a string input?
12. How do you pass a boolean input?
13. How do you pass a number input?
14. How do you define required inputs?
15. How do you define default inputs?
16. How do you return an image digest from a reusable workflow?
17. How do you consume reusable workflow outputs?
18. How do you use reusable workflows with GitHub Environments?
19. How do you use reusable workflows with concurrency?
20. How do you use reusable workflows with timeouts?
21. How would you centralize SonarQube, Trivy, and Veracode scanning?
22. How would you centralize Docker builds?
23. How would you centralize Terraform plan/apply?
24. How would you centralize Kubernetes deployment?
25. How do reusable workflows improve consistency across microservices?

## Advanced / Production

26. Design a reusable workflow architecture for a microservices platform.
27. How would you build a centralized DevSecOps workflow for multiple repositories?
28. How would you securely pass AWS authentication to a reusable deployment workflow?
29. Why is OIDC preferable to long-lived AWS credentials?
30. How would you protect a reusable workflow that has production deployment permissions?
31. How would you version reusable workflows?
32. Why should critical workflows avoid blindly referencing `@main`?
33. How would you handle breaking changes in a shared workflow?
34. How would you test a reusable workflow before releasing it to hundreds of repositories?
35. How would you prevent a compromised reusable workflow from affecting all consuming repositories?
36. How would you validate service and environment inputs?
37. How would you prevent command injection through reusable workflow inputs?
38. How would you protect reusable workflows from untrusted pull requests?
39. How would you combine reusable workflows with GitHub Environment approvals?
40. How would you combine reusable workflows with concurrency and timeout controls?
41. How would you design a reusable Terraform workflow with plan, artifact, approval, concurrency, and apply?
42. How would you design a reusable Helm deployment workflow with timeout, health checks, and rollback?
43. How would you design a reusable GitOps workflow that updates manifests and lets ArgoCD deploy to EKS?
44. How would you return an immutable ECR image digest from a reusable Docker workflow?
45. Design an enterprise-grade reusable GitHub Actions platform covering Build, Maven/npm caching, Docker, SonarQube, Trivy, Veracode, ECR, Terraform, JIRA change validation, GitHub Environments, OIDC, Helm, ArgoCD, EKS, concurrency, timeout, rollback, versioning, and security governance.