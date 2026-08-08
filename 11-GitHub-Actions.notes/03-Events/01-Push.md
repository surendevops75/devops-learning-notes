# Push Event

The `push` event is one of the most commonly used GitHub Actions triggers.

It starts a workflow when commits or other changes are pushed to a GitHub repository.

The `push` event is commonly used for:

- Continuous Integration
- Build pipelines
- Automated testing
- Security scanning
- Docker image creation
- Artifact publishing
- Deployment pipelines

---

# Basic Syntax

```yaml
name: Push Workflow

on:
  push:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build
        run: echo "Building application"
```

Whenever code is pushed to the repository, the workflow starts.

---

# Execution Flow

```text
Developer

↓

git push

↓

GitHub Repository

↓

push Event

↓

GitHub Actions Workflow

↓

Runner

↓

Job

↓

Steps
```

---

# Push to Any Branch

The following workflow runs whenever code is pushed to any branch.

```yaml
on:
  push:
```

Example:

```text
feature/catalogue

↓

git push

↓

Workflow Runs
```

```text
develop

↓

git push

↓

Workflow Runs
```

```text
main

↓

git push

↓

Workflow Runs
```

This can generate a large number of workflow executions in an active repository.

---

# Push to a Specific Branch

You can restrict the workflow to specific branches.

```yaml
on:
  push:
    branches:
      - main
```

Now the workflow runs only when code is pushed to `main`.

```text
feature branch

↓

Push

↓

No Workflow
```

```text
main

↓

Push

↓

Workflow Runs
```

---

# Multiple Branches

```yaml
on:
  push:
    branches:
      - main
      - develop
      - release/*
```

This workflow runs for:

- `main`
- `develop`
- `release/1.0`
- `release/2.0`

---

# Feature Branch Example

A common enterprise branching strategy is:

```text
main
  │
  ├── feature/catalogue
  ├── feature/cart
  ├── feature/payment
  └── feature/inventory
```

Developers work on short-lived feature branches.

When code is pushed:

```text
feature/catalogue

↓

push

↓

CI Workflow

↓

Build

↓

Unit Tests

↓

Security Scan
```

This allows developers to receive fast feedback before creating or merging a Pull Request.

---

# Main Branch Production Workflow

A production-oriented workflow can restrict execution to `main`.

```yaml
name: Main Branch CI

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package

      - name: Test
        run: mvn test
```

Execution:

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Code Review

↓

Merge to main

↓

push Event

↓

CI Workflow

↓

Build

↓

Test
```

---

# Push with Tags

The `push` event can also trigger workflows when tags are pushed.

```yaml
on:
  push:
    tags:
      - 'v*'
```

Example:

```bash
git tag v1.0.0

git push origin v1.0.0
```

The workflow starts because the tag matches:

```text
v*
```

---

# Production Release Workflow

Tags are commonly used to identify release versions.

```text
Developer

↓

Merge to main

↓

Release Preparation

↓

Create v1.5.0

↓

Push Tag

↓

GitHub Actions

↓

Build Release

↓

Security Scan

↓

Publish Artifact

↓

Create Release
```

Example:

```yaml
name: Release Build

on:
  push:
    tags:
      - 'v*'

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package

      - name: Publish
        run: echo "Publishing release"
```

---

# Branch and Tag Filters

You can configure both branch and tag filters.

```yaml
on:
  push:
    branches:
      - main
    tags:
      - 'v*'
```

This allows the workflow to respond to the configured branch and tag push events.

---

# Path Filters

The `push` event can also be restricted based on which files changed.

Example:

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'pom.xml'
```

The workflow runs when changes affect the specified paths.

---

# Why Use Path Filters?

Large repositories may contain multiple independent components.

For example:

```text
Repository

├── catalogue/

├── cart/

├── payment/

└── infrastructure/
```

A catalogue workflow does not necessarily need to run when only documentation changes.

Path filtering can reduce unnecessary workflow executions.

---

# Enterprise Microservices Example

Suppose a repository contains:

```text
services/

├── catalogue/

├── cart/

├── payment/

└── inventory/
```

A catalogue-specific workflow can use:

```yaml
on:
  push:
    paths:
      - 'services/catalogue/**'
```

Execution:

```text
Change catalogue

↓

Push

↓

Catalogue Workflow

↓

Build

↓

Test

↓

Docker Build
```

A change only to `payment` does not match the catalogue path.

---

# Ignoring Paths

You can exclude paths using `paths-ignore`.

```yaml
on:
  push:
    paths-ignore:
      - 'docs/**'
      - '*.md'
```

This prevents the workflow from running for changes that only affect those paths.

---

# Ignoring Branches

You can also use `branches-ignore`.

```yaml
on:
  push:
    branches-ignore:
      - 'feature/**'
```

This prevents the workflow from running for pushes to feature branches.

---

# Enterprise CI Strategy

A common approach is to run lightweight validation for feature branches and more comprehensive processing for protected branches.

```text
Feature Branch

↓

Push

↓

Build

↓

Unit Tests

↓

Static Checks
```

Then:

```text
main

↓

Push

↓

Full CI

↓

SonarQube

↓

Trivy

↓

Docker Build

↓

Push Image
```

This keeps developer feedback fast while enforcing stronger validation on the main branch.

---

# Push Event and Commit SHA

Every push identifies a specific commit.

GitHub Actions exposes commit information through GitHub contexts.

For example:

```yaml
- name: Display Commit SHA
  run: echo "${{ github.sha }}"
```

The SHA can be used to identify exactly which source version was tested or built.

---

# Why Commit SHA Matters in Production

For production deployments, using an immutable commit or image reference improves traceability.

Example:

```text
Git Commit SHA

↓

Build

↓

Docker Image

↓

Image Tag / Digest

↓

Deployment

↓

Production
```

This allows the team to determine exactly which version was deployed.

---

# Production Deployment Workflow

A production pipeline may use a push to `main` only to initiate CI.

```text
Push to main

↓

CI

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Trivy

↓

Docker Image

↓

Push Image

↓

Artifact Ready
```

Production deployment can then be controlled separately:

```text
Artifact

↓

UAT

↓

E2E Tests

↓

Change Request

↓

Approval

↓

Deployment Window

↓

Production
```

This is safer than automatically deploying every push to `main`.

---

# Push Event vs Production Deployment

A common enterprise pattern is:

```text
push to feature

↓

CI Validation
```

```text
push to main

↓

Full CI

↓

Build Artifact
```

```text
workflow_dispatch

↓

Production Deployment
```

This separation provides better production control.

---

# Important Consideration

A `push` event represents a repository change.

It does **not** automatically mean that the application should be deployed to production.

A mature enterprise pipeline separates:

```text
Code Change

↓

Validation

↓

Artifact Creation

↓

Environment Promotion

↓

Production Deployment
```

---

# Best Practices

- Use branch filters to avoid unnecessary executions.
- Use path filters for monorepositories when appropriate.
- Use tags for release workflows.
- Keep production deployment separate from ordinary push-based CI when approvals are required.
- Use commit SHA for traceability.
- Keep feature-branch workflows fast.
- Run comprehensive validation on protected branches.

---

# Common Mistakes

- Running expensive pipelines for every repository change.
- Automatically deploying every push to `main`.
- Ignoring path filters in large repositories.
- Using mutable version references for production deployments.
- Not recording the commit SHA used for deployment.
- Mixing CI validation and controlled production deployment in one unrestricted push workflow.

---

# Production Troubleshooting

## Scenario - Workflow Did Not Run After Push

Check:

```text
1. Was the workflow file present?
2. Is the workflow inside .github/workflows/?
3. Does the push branch match the configured branches?
4. Do path filters exclude the changed files?
5. Was the workflow enabled?
6. Was the workflow syntax valid?
```

---

## Scenario - Workflow Runs Too Often

Review:

```text
1. Branch filters
2. Path filters
3. Unnecessary push triggers
4. Duplicate workflows
5. Monorepo structure
```

Then reduce the trigger scope.

---

## Scenario - Production Pipeline Runs Unexpectedly

Check:

```text
Push Event

↓

Branch Filter

↓

Path Filter

↓

Workflow Conditions

↓

Deployment Job Conditions
```

Do not assume that a successful workflow trigger should automatically result in a production deployment.

---

# Summary

The `push` event is one of the fundamental GitHub Actions triggers.

It can be configured using:

- Branch filters
- Tag filters
- Path filters
- `branches-ignore`
- `paths-ignore`

In enterprise environments, the push event is commonly used for Continuous Integration, while production deployments are often controlled through separate workflows, approvals, deployment windows, and manual triggers.

---

# Interview Questions

## Basic

1. What is the `push` event?
2. When does a push workflow execute?
3. How do you trigger a workflow only for `main`?
4. How do you trigger a workflow for tags?
5. What is `github.sha`?

## Intermediate

6. What are branch filters?
7. What are path filters?
8. What is the difference between `paths` and `paths-ignore`?
9. Why should production deployment not always happen directly after a push?
10. How can push events be optimized in a monorepo?

## Advanced

11. Design a GitHub Actions CI workflow that runs for feature branches but performs additional security scanning when code reaches `main`.
12. Design a production release process where a push to `main` creates an immutable artifact but production deployment requires a JIRA change request, approval, and deployment window.
13. A monorepo contains 20 microservices and every push currently triggers all 20 pipelines. How would you redesign the push-event configuration to reduce unnecessary builds?
14. A developer pushes a commit to `main`, but the production deployment starts unexpectedly. Explain how you would investigate the trigger and prevent uncontrolled production deployments.