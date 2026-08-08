# Pull Request Event

The `pull_request` event triggers a GitHub Actions workflow when activity occurs on a Pull Request (PR).

Pull Requests are a critical part of enterprise development because they provide a controlled process for:

- Code review
- Automated testing
- Code quality checks
- Security scanning
- Branch protection
- Merge approval

A common enterprise flow is:

```text
Developer

↓

Feature Branch

↓

Push Code

↓

Pull Request

↓

GitHub Actions

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Security Scan

↓

Code Review

↓

Approval

↓

Merge
```

---

# What is a Pull Request?

A Pull Request is a request to merge changes from one branch into another branch.

Example:

```text
feature/catalogue

        ↓

    Pull Request

        ↓

main
```

The Pull Request provides a controlled checkpoint before changes reach the target branch.

---

# Why Use Pull Request Workflows?

A Pull Request workflow allows the team to validate code before merging it.

Typical checks include:

- Compilation
- Unit tests
- Code quality
- Security scanning
- Dependency scanning
- Docker build validation
- Terraform validation
- Policy checks

The goal is to prevent defective or insecure code from being merged.

---

# Basic Syntax

```yaml
name: Pull Request CI

on:
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package

      - name: Test
        run: mvn test
```

---

# Pull Request Lifecycle

```text
Developer

↓

Create Feature Branch

↓

Develop

↓

Push Changes

↓

Create Pull Request

↓

GitHub Actions

↓

Build

↓

Test

↓

Security Scan

↓

Code Review

↓

Approval

↓

Merge
```

---

# Pull Request Activity Types

The `pull_request` event supports different activity types.

Common examples include:

- `opened`
- `synchronize`
- `reopened`
- `closed`

The `synchronize` activity is especially important because it occurs when new commits are pushed to the branch associated with an existing Pull Request.

Example:

```text
Create PR

↓

CI Runs

↓

Developer Fixes Code

↓

Push New Commit

↓

synchronize Event

↓

CI Runs Again
```

This ensures that the latest changes are validated.

---

# Trigger When Pull Request Is Opened

```yaml
on:
  pull_request:
    types:
      - opened
```

The workflow runs when a Pull Request is opened.

---

# Trigger When Pull Request Is Updated

```yaml
on:
  pull_request:
    types:
      - synchronize
```

The workflow runs when new commits are pushed to the Pull Request branch.

---

# Trigger on Opened and Synchronize

```yaml
on:
  pull_request:
    types:
      - opened
      - synchronize
```

This is useful for continuous validation during Pull Request development.

---

# Pull Request Target Branch

You can restrict the workflow to Pull Requests targeting a specific branch.

```yaml
on:
  pull_request:
    branches:
      - main
```

This means the workflow runs when a Pull Request targets `main`.

Example:

```text
feature/catalogue

↓

Pull Request

↓

main

↓

Workflow Runs
```

But:

```text
feature/catalogue

↓

Pull Request

↓

develop

↓

Workflow Does Not Run
```

---

# Multiple Target Branches

```yaml
on:
  pull_request:
    branches:
      - main
      - develop
      - release/*
```

This allows Pull Requests targeting those branches to trigger the workflow.

---

# Path Filters

Pull Request workflows can also be limited to specific paths.

```yaml
on:
  pull_request:
    paths:
      - 'services/catalogue/**'
```

This is useful for monorepositories.

Example:

```text
Repository

├── services/

│   ├── catalogue/

│   ├── cart/

│   └── payment/

└── infrastructure/
```

A catalogue Pull Request can trigger only the catalogue validation workflow.

---

# Pull Request CI for Microservices

```yaml
name: Catalogue PR Validation

on:
  pull_request:
    branches:
      - main
    paths:
      - 'services/catalogue/**'

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package

      - name: Unit Tests
        run: mvn test
```

Execution:

```text
Developer

↓

Modify Catalogue

↓

Push

↓

Pull Request to main

↓

Catalogue PR Workflow

↓

Build

↓

Tests

↓

Result
```

---

# Enterprise Pull Request Pipeline

A production-grade Pull Request workflow can contain multiple validation stages.

```text
Pull Request

↓

Checkout

↓

Build

↓

Unit Tests

↓

Code Coverage

↓

SonarQube

↓

Dependency Scan

↓

Trivy

↓

Terraform Validate

↓

Policy Check

↓

Result
```

Only after successful validation should the Pull Request become eligible for merging.

---

# Pull Request Checks

GitHub Actions results appear as status checks on the Pull Request.

Example:

```text
Pull Request

├── Build              ✓
├── Unit Tests         ✓
├── SonarQube          ✓
├── Security Scan      ✓
└── Terraform Check   ✓
```

If a required check fails:

```text
Security Scan          ✗

↓

Merge Blocked
```

This creates a quality gate before code reaches the protected branch.

---

# Branch Protection

Pull Request workflows become more powerful when combined with branch protection.

Example:

```text
Developer

↓

Pull Request

↓

Required Checks

├── Build ✓
├── Tests ✓
├── SonarQube ✓
└── Security ✓

↓

Required Review

↓

Merge
```

This prevents developers from bypassing mandatory validation.

---

# Pull Request and Commit SHA

Each commit has a unique SHA.

Example:

```text
Commit

↓

a83f91c...
```

GitHub Actions can access the commit information through GitHub contexts.

The exact commit SHA that was validated should be traceable throughout the CI/CD process.

This is especially important when the resulting artifact is promoted toward production.

---

# Merge Strategy

Your development process can use different merge strategies.

## Merge

A merge can create an additional merge commit.

```text
A --- B --- C
       \
        D --- E

           ↓

A --- B --- C ------- M
       \             /
        D --- E ----
```

The merge commit records the integration of the branches.

---

# Rebase

Rebase moves commits onto a new base.

```text
Before:

A --- B --- C
       \
        D --- E

After:

A --- B --- C --- D' --- E'
```

The rebased commits receive new commit IDs.

Therefore:

```text
Commit ID Changed

↓

Commit Identity Changed
```

This does not necessarily mean that the application behavior changed, but the Git commit identity is different.

For production traceability, record the exact commit SHA that was validated and ultimately deployed.

---

# Pull Request vs Push

The two events have different purposes.

| `pull_request` | `push` |
|---|---|
| Validates Pull Requests | Validates pushed commits |
| Commonly used before merge | Commonly used after push |
| Supports code review workflows | Commonly used for CI |
| Works with branch protection | Can build artifacts |
| Helps prevent bad code from merging | Can trigger downstream pipelines |

Typical enterprise flow:

```text
Feature Branch

↓

Push

↓

Pull Request

↓

PR Validation

↓

Review

↓

Merge

↓

Push to main

↓

Full CI
```

---

# Production Deployment Strategy

A Pull Request should generally validate code rather than directly deploy production.

Recommended flow:

```text
Pull Request

↓

Build

↓

Test

↓

Security Scan

↓

Approval

↓

Merge

↓

main

↓

Build Immutable Artifact

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

This creates a clear separation between code validation and production promotion.

---

# Enterprise Pull Request Workflow

```yaml
name: Enterprise PR Validation

on:
  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:

  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package

      - name: Unit Tests
        run: mvn test

  security:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Security Scan
        run: echo "Run security scanner"
```

Build and Security can execute independently.

```text
             Pull Request
                  |
          ┌───────┴────────┐
          ↓                ↓
       Build           Security
          ↓                ↓
        Tests            Scan
          └───────┬────────┘
                  ↓
             PR Checks
                  ↓
               Review
                  ↓
                Merge
```

---

# Enterprise Production Workflow

A mature enterprise workflow separates Pull Request validation from deployment.

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

PR Validation

├── Build
├── Unit Tests
├── SonarQube
├── Trivy
└── Terraform Validate

↓

Code Review

↓

Merge to main

↓

Push Event

↓

Full CI

↓

Docker Image

↓

Artifact

↓

QA

↓

SIT

↓

UAT

↓

JIRA Change Request

↓

Production Approval

↓

Production Deployment
```

This provides traceability from the original Pull Request through production.

---

# Production Troubleshooting

## Scenario 1 - Pull Request Workflow Does Not Run

Check:

```text
1. Workflow exists in .github/workflows/
2. pull_request event is configured
3. Target branch matches the branch filter
4. Path filters do not exclude the change
5. Workflow is enabled
6. YAML syntax is valid
```

---

## Scenario 2 - Workflow Runs but Required Checks Are Missing

Review:

```text
Workflow

↓

Jobs

↓

Runner

↓

Permissions

↓

Step Execution
```

Check the workflow run logs to identify where execution stopped.

---

## Scenario 3 - Pull Request Cannot Be Merged

Review required checks.

```text
Build                 ✓
Unit Tests            ✓
SonarQube             ✓
Security Scan         ✗
```

The failed required check must be investigated and resolved before merging.

---

## Scenario 4 - Pull Request Workflow Runs Repeatedly

A common cause is the `synchronize` activity.

Every new commit pushed to an open Pull Request can trigger another workflow execution.

Review:

- Event configuration
- Branch filters
- Path filters
- Duplicate workflows

---

## Scenario 5 - Different Commit Is Deployed to Production

Verify:

```text
Pull Request Commit SHA

↓

CI Validation SHA

↓

Built Artifact SHA

↓

Deployment SHA
```

The deployment process should maintain traceability between the validated source and the artifact deployed to production.

---

# Best Practices

- Use Pull Request workflows for pre-merge validation.
- Protect `main` and other critical branches.
- Require successful CI checks before merging.
- Use path filters in large monorepositories.
- Keep PR pipelines fast enough for developer feedback.
- Run security checks before merge.
- Record the commit SHA used for validation.
- Keep production deployment separate from PR validation.
- Use required reviewers for sensitive repositories.
- Use least-privilege workflow permissions.

---

# Common Mistakes

- Deploying production directly from a Pull Request.
- Not protecting the main branch.
- Running expensive workflows unnecessarily.
- Ignoring `synchronize` activity.
- Allowing failed security checks to be bypassed.
- Not requiring successful status checks.
- Using broad workflows for unrelated monorepo services.
- Losing traceability between the validated commit and production artifact.

---

# Summary

The `pull_request` event is primarily used to validate code before it is merged.

A strong enterprise Pull Request workflow should provide:

- Build validation
- Automated testing
- Code quality checks
- Security scanning
- Infrastructure validation
- Required status checks
- Code review

The Pull Request should establish confidence that the change is safe to merge.

Production deployment should normally happen through a separate controlled promotion process after the code is merged and validated.

---

# Interview Questions

## Basic

1. What is the `pull_request` event?
2. When does a Pull Request workflow execute?
3. What is the difference between `push` and `pull_request`?
4. What is the `synchronize` Pull Request activity?
5. How do you trigger a workflow only for Pull Requests targeting `main`?

## Intermediate

6. How do Pull Request workflows help CI/CD?
7. How do you prevent a Pull Request from being merged when tests fail?
8. How do branch protection rules work with GitHub Actions?
9. What are Pull Request path filters?
10. Why should production deployment usually be separated from Pull Request validation?

## Advanced

11. Design an enterprise Pull Request pipeline that performs build, unit testing, SonarQube, Trivy, Terraform validation, and policy checks before allowing a merge.
12. A monorepository contains multiple microservices and every Pull Request triggers all service pipelines. How would you optimize the workflow?
13. A developer pushes five commits to an open Pull Request and the CI pipeline runs five times. Explain why this happens and how you would optimize the process.
14. A required security check passes for a Pull Request, but the production deployment later uses a different commit SHA. Explain the risk and design a workflow that guarantees the exact validated commit is the one promoted to production.