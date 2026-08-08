# Workflow Call Event

The `workflow_call` event allows one GitHub Actions workflow to call another workflow and reuse its jobs and logic.

It is primarily used to create **reusable workflows**.

Reusable workflows are extremely useful in enterprise environments because multiple repositories often need the same CI/CD process.

Instead of duplicating the same workflow in every repository, an organization can define the workflow once and allow other workflows to call it.

---

# Why Use workflow_call?

Without reusable workflows:

```text
Repository A

↓

Copy CI Workflow


Repository B

↓

Copy CI Workflow


Repository C

↓

Copy CI Workflow
```

This creates:

- Duplicate code
- Maintenance overhead
- Inconsistent pipelines
- Difficult upgrades

With `workflow_call`:

```text
Central Reusable Workflow

        ↑
        |
 ┌──────┼──────┐
 |      |      |
 ↓      ↓      ↓
Repo A Repo B Repo C
```

One workflow can be maintained centrally and reused by multiple repositories.

---

# Basic Syntax

A reusable workflow must declare:

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
      - name: Build Application
        run: echo "Building application"
```

This workflow is designed to be called by another workflow.

---

# Calling a Reusable Workflow

A calling workflow uses:

```yaml
uses:
```

Example:

```yaml
name: Application CI

on:
  push:
    branches:
      - main

jobs:
  build:
    uses: organization/reusable-workflows/.github/workflows/build.yml@main
```

Execution:

```text
Application Repository

↓

Push

↓

Application CI

↓

Reusable Build Workflow

↓

Build
```

---

# Workflow Architecture

A reusable workflow separates:

```text
Calling Workflow

↓

Application-Specific Configuration

↓

Reusable Workflow

↓

Common CI/CD Logic
```

This allows organizations to standardize pipelines.

---

# Local Reusable Workflow

A repository can also call a reusable workflow from the same repository.

Example structure:

```text
.github/

└── workflows/

      ci.yml

      reusable-build.yml
```

Calling workflow:

```yaml
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
```

This is useful when a repository has multiple workflows that share common logic.

---

# Reusable Workflow Inputs

Reusable workflows can accept inputs.

Example:

```yaml
name: Reusable Deployment

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Display Environment
        run: echo "Deploying to ${{ inputs.environment }}"
```

The calling workflow can pass the environment.

---

# Calling with Inputs

```yaml
jobs:
  deploy:
    uses: ./.github/workflows/reusable-deployment.yml

    with:
      environment: prod
```

Execution:

```text
Calling Workflow

↓

environment = prod

↓

Reusable Workflow

↓

Deploy to prod
```

---

# Multiple Inputs

A reusable workflow can accept multiple inputs.

Example:

```yaml
on:
  workflow_call:
    inputs:

      environment:
        required: true
        type: string

      version:
        required: true
        type: string

      application:
        required: true
        type: string
```

The caller provides:

```yaml
with:
  environment: prod
  version: a83f91c
  application: catalogue
```

---

# Input Types

Reusable workflows support typed inputs.

Common types include:

- `string`
- `boolean`
- `number`

Example:

```yaml
on:
  workflow_call:
    inputs:
      run_tests:
        required: true
        type: boolean
```

---

# Secrets

Reusable workflows can also receive secrets.

Example:

```yaml
on:
  workflow_call:
    secrets:
      deployment_token:
        required: true
```

The calling workflow passes the secret.

```yaml
jobs:
  deploy:
    uses: ./.github/workflows/reusable-deployment.yml

    secrets:
      deployment_token: ${{ secrets.DEPLOYMENT_TOKEN }}
```

Secrets should never be hardcoded.

---

# Secrets Inheritance

In appropriate organizational designs, a calling workflow can pass available secrets to a reusable workflow using:

```yaml
secrets: inherit
```

Example:

```yaml
jobs:
  deploy:
    uses: organization/platform/.github/workflows/deploy.yml@main

    secrets: inherit
```

This should be used carefully because it can expose more secrets than the reusable workflow actually needs.

Prefer explicitly passing only the secrets required by the reusable workflow.

---

# Enterprise Reusable CI Workflow

A company may create a centralized CI workflow:

```text
Organization

└── Platform Workflows

      ├── Java Build
      ├── Node Build
      ├── Security Scan
      ├── Docker Build
      └── Terraform Validation
```

Application repositories call these workflows.

```text
Catalogue Repository
        |
        ↓
Reusable Java CI

Cart Repository
        |
        ↓
Reusable Java CI

Payment Repository
        |
        ↓
Reusable Java CI
```

---

# Enterprise Standardization

Reusable workflows are useful when an organization wants every application to follow the same standards.

Example:

```text
Every Application

↓

Build

↓

Unit Test

↓

SonarQube

↓

Trivy

↓

Artifact

↓

Standardized Result
```

Instead of allowing every team to create a completely different pipeline, the organization provides a common reusable workflow.

---

# Enterprise DevSecOps Workflow

A reusable workflow can standardize security checks.

```text
Application Repository

↓

Reusable CI

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Trivy

↓

Veracode

↓

Artifact
```

This is especially useful when security controls must be consistently applied across many repositories.

---

# Enterprise Docker Workflow

A reusable workflow can standardize container builds.

```text
Application

↓

Reusable Docker Workflow

↓

Docker Build

↓

Image Scan

↓

Tag

↓

Push to ECR
```

The calling repository only supplies the required inputs.

---

# Enterprise Deployment Workflow

A reusable deployment workflow can standardize environment promotion.

```text
Application

↓

Reusable Deployment Workflow

↓

Environment Input

↓

Version Input

↓

Validation

↓

Helm Deployment

↓

Health Check
```

Multiple services can use the same deployment logic.

---

# Production Deployment Architecture

A mature enterprise architecture can look like:

```text
Application Repository

↓

Application Workflow

↓

Reusable CI Workflow

↓

Build

↓

Test

↓

Security Scan

↓

Docker Image

↓

ECR

↓

Reusable Deployment Workflow

↓

QA

↓

SIT

↓

UAT

↓

Production Approval

↓

Production
```

This separates application-specific configuration from organization-wide CI/CD standards.

---

# Reusable Workflow vs Composite Action

These are related but different.

| Reusable Workflow | Composite Action |
|---|---|
| Reuses complete jobs/workflows | Reuses steps |
| Can contain multiple jobs | Runs as a step |
| Defined using `workflow_call` | Defined using `action.yml` |
| Can be called by another workflow | Used inside a workflow step |
| Useful for complete CI/CD processes | Useful for reusable command sequences |

Example:

```text
Reusable Workflow

↓

Build Job

↓

Test Job

↓

Security Job
```

Whereas:

```text
Composite Action

↓

Checkout

↓

Install Dependencies

↓

Run Command
```

---

# workflow_call vs repository_dispatch

These events have different purposes.

### workflow_call

Used for:

```text
Workflow

↓

Reusable Workflow
```

### repository_dispatch

Used for:

```text
External System

↓

GitHub API

↓

Workflow
```

Use `workflow_call` when the caller is another GitHub Actions workflow.

Use `repository_dispatch` when an external system needs to trigger a workflow.

---

# Reusable Workflow Versioning

When calling a reusable workflow from another repository, specify a reference.

Example:

```yaml
uses: organization/platform/.github/workflows/deploy.yml@v1
```

Possible references include:

```text
main

v1

v1.2.0

commit SHA
```

For production environments, immutable references such as a specific commit SHA provide stronger reproducibility than a moving branch reference.

---

# Why Version Reusable Workflows?

Suppose 100 repositories use:

```text
deploy.yml@main
```

A change to the reusable workflow can affect all repositories.

Versioning allows teams to control adoption.

Example:

```text
v1

↓

Existing Applications


v2

↓

New Applications
```

Teams can migrate gradually.

---

# Enterprise Platform Repository

A centralized platform repository might contain:

```text
platform-workflows/

├── java-ci.yml

├── node-ci.yml

├── docker-build.yml

├── security-scan.yml

├── terraform.yml

├── helm-deploy.yml

└── production-deploy.yml
```

Application repositories consume these workflows.

---

# Production Example

Reusable CI workflow:

```yaml
name: Reusable Java CI

on:
  workflow_call:
    inputs:
      java-version:
        required: true
        type: string

jobs:

  build:
    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.java-version }}
          distribution: temurin

      - name: Build
        run: mvn clean package

      - name: Test
        run: mvn test
```

Calling workflow:

```yaml
name: Catalogue CI

on:
  push:
    branches:
      - main

jobs:

  ci:
    uses: organization/platform-workflows/.github/workflows/java-ci.yml@v1

    with:
      java-version: '21'
```

Execution:

```text
Catalogue Repository

↓

Catalogue CI

↓

Reusable Java CI

↓

Setup Java

↓

Build

↓

Test
```

---

# Production Deployment Example

Reusable deployment workflow:

```yaml
name: Reusable Deployment

on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string

      version:
        required: true
        type: string

jobs:

  deploy:
    runs-on: self-hosted

    environment: ${{ inputs.environment }}

    steps:

      - name: Deploy
        run: |
          echo "Application: catalogue"
          echo "Environment: ${{ inputs.environment }}"
          echo "Version: ${{ inputs.version }}"

      - name: Health Check
        run: echo "Running health check"
```

Calling workflow:

```yaml
jobs:

  deploy:
    uses: organization/platform-workflows/.github/workflows/deploy.yml@v1

    with:
      environment: production
      version: a83f91c
```

---

# Production Governance

Reusable workflows can enforce organizational standards.

For example:

```text
Every Production Deployment

↓

Reusable Workflow

↓

JIRA Validation

↓

Commit Validation

↓

CI Status

↓

Security Results

↓

Testing Results

↓

Approval

↓

Deployment

↓

Smoke Test
```

Application teams cannot accidentally omit required steps if the organization centralizes these controls appropriately.

---

# Important Security Principle

Do not assume reusable workflows are automatically secure.

Review:

- Workflow source
- Version reference
- Permissions
- Secrets
- Inputs
- Runner type
- Environment protection

A reusable workflow with excessive permissions can create a large security impact because many repositories may consume it.

---

# Permissions

Reusable workflows should use least-privilege permissions.

Example:

```yaml
permissions:
  contents: read
```

Additional permissions should only be granted when required.

---

# Production Troubleshooting

## Scenario 1 - Reusable Workflow Cannot Be Called

Check:

```text
1. workflow_call is defined.
2. Workflow file path is correct.
3. Repository name is correct.
4. Reference is valid.
5. Caller has access to the workflow.
```

---

## Scenario 2 - Input Is Missing

Check:

```text
Caller

↓

with:

↓

Input Name

↓

Reusable Workflow

↓

inputs.<name>
```

The input name must match exactly.

---

## Scenario 3 - Secret Is Not Available

Check:

```text
Caller

↓

secrets:

↓

Reusable Workflow

↓

Secret Definition
```

Confirm that the required secret was passed correctly.

---

## Scenario 4 - Reusable Workflow Change Breaks Multiple Repositories

This is why versioning is important.

If repositories use:

```text
deploy.yml@main
```

a change to the reusable workflow can affect many consumers.

A safer strategy is:

```text
deploy.yml@v1
```

or an immutable commit SHA.

---

## Scenario 5 - Production Deployment Uses Wrong Workflow Version

Trace:

```text
Application Repository

↓

uses:

↓

Reusable Workflow Reference

↓

Workflow Version

↓

Deployment Logic
```

Verify the exact reusable workflow reference used by the deployment.

---

# Best Practices

- Use reusable workflows to eliminate duplicate CI/CD logic.
- Centralize enterprise standards.
- Keep reusable workflows focused.
- Define explicit inputs.
- Pass only required secrets.
- Use least-privilege permissions.
- Version reusable workflows.
- Prefer immutable references for production.
- Test reusable workflows before broad adoption.
- Document inputs and expected behavior.

---

# Common Mistakes

- Copying the same workflow into many repositories.
- Using a moving `main` reference for critical production workflows.
- Passing unnecessary secrets.
- Giving reusable workflows excessive permissions.
- Not validating inputs.
- Mixing unrelated responsibilities into one reusable workflow.
- Making breaking changes without versioning.
- Not testing the reusable workflow against representative repositories.

---

# Summary

The `workflow_call` event enables GitHub Actions workflows to be reused by other workflows.

It is one of the most important features for enterprise CI/CD standardization.

The basic architecture is:

```text
Application Workflow

↓

workflow_call

↓

Reusable Workflow

↓

Standardized CI/CD Logic
```

Reusable workflows can standardize:

- Builds
- Tests
- Security scanning
- Docker image creation
- Terraform
- Helm deployments
- Production controls

A strong enterprise architecture uses reusable workflows to centralize common standards while keeping application-specific configuration inside individual repositories.

---

# Interview Questions

## Basic

1. What is `workflow_call`?
2. What is a reusable workflow?
3. How does one workflow call another workflow?
4. Why are reusable workflows useful?
5. What is the purpose of `with`?

## Intermediate

6. How do you pass inputs to a reusable workflow?
7. How do you pass secrets?
8. What is the difference between reusable workflows and composite actions?
9. What is the difference between `workflow_call` and `repository_dispatch`?
10. Why should reusable workflows be versioned?

## Advanced

11. Design a centralized enterprise CI workflow that can be reused by Java microservices across 100 repositories.
12. Design a reusable production deployment workflow that accepts environment and version inputs while enforcing JIRA validation, security checks, approvals, and health checks.
13. A platform team changes a reusable deployment workflow and unexpectedly breaks 50 applications. Explain how versioning and immutable references could have prevented the issue.
14. Design a reusable DevSecOps workflow that standardizes SonarQube, Trivy, Veracode, artifact creation, and deployment controls across multiple application repositories.
15. Explain how you would securely design reusable workflows so that application repositories receive only the permissions and secrets required for their specific deployment.