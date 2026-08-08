# Run vs Uses

In GitHub Actions, steps generally execute work in two main ways:

```text
run
 |
 └── Execute shell commands

uses
 |
 └── Execute a reusable GitHub Action
```

Understanding the difference between `run` and `uses` is essential when writing GitHub Actions workflows.

---

# 1. run

The `run` keyword executes commands directly on the runner.

Example:

```yaml
- name: Build Application
  run: mvn clean package
```

The runner executes:

```text
mvn clean package
```

Another example:

```yaml
- name: Run Tests
  run: mvn test
```

---

# Multiple Commands with run

Use `|` when multiple commands need to execute in the same step.

```yaml
- name: Build Application
  run: |
    mvn clean
    mvn package
    mvn test
```

Execution:

```text
mvn clean
   |
   ↓
mvn package
   |
   ↓
mvn test
```

---

# Shell Commands

`run` is useful for:

- Linux commands
- Bash scripts
- Maven
- npm
- Terraform
- Docker
- Helm
- kubectl
- Python scripts
- Custom shell scripts

Example:

```yaml
- name: Check Kubernetes
  run: kubectl get pods -A
```

Example:

```yaml
- name: Terraform Plan
  run: terraform plan
```

Example:

```yaml
- name: Docker Build
  run: docker build -t catalogue:${GITHUB_SHA} .
```

---

# 2. uses

The `uses` keyword executes a reusable GitHub Action.

Example:

```yaml
- name: Checkout Code
  uses: actions/checkout@v4
```

Instead of manually writing all the logic required to download repository contents, the workflow uses an existing Action.

---

# Common uses Examples

Checkout:

```yaml
- name: Checkout Code
  uses: actions/checkout@v4
```

Java setup:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: temurin
```

Node.js setup:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '22'
```

Python setup:

```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.12'
```

Terraform setup:

```yaml
- name: Setup Terraform
  uses: hashicorp/setup-terraform@v3
```

Artifact upload:

```yaml
- name: Upload Artifact
  uses: actions/upload-artifact@v4
  with:
    name: application
    path: target/application.jar
```

---

# Basic Difference

The simplest way to remember:

```text
run = command

uses = action
```

Example:

```yaml
- name: Checkout
  uses: actions/checkout@v4

- name: Build
  run: mvn clean package
```

Here:

```text
Checkout → uses
Build    → run
```

---

# run vs uses

| `run` | `uses` |
|---|---|
| Executes a command | Executes a reusable Action |
| Uses shell | Uses an Action implementation |
| Good for custom commands | Good for reusable functionality |
| Useful for scripts | Useful for standard automation |
| Example: `mvn test` | Example: `actions/checkout@v4` |

---

# When to Use run

Use `run` when you already know the command you need to execute.

Examples:

```yaml
- name: Run Unit Tests
  run: mvn test
```

```yaml
- name: Terraform Validate
  run: terraform validate
```

```yaml
- name: Docker Build
  run: docker build -t catalogue:${GITHUB_SHA} .
```

```yaml
- name: Helm Deployment
  run: |
    helm upgrade --install catalogue ./helm/catalogue \
      --namespace catalogue \
      --set image.tag="${IMAGE_TAG}"
```

---

# When to Use uses

Use `uses` when a suitable reusable Action already provides the functionality.

Examples:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
```

```yaml
- name: Upload Artifact
  uses: actions/upload-artifact@v4
```

```yaml
- name: Download Artifact
  uses: actions/download-artifact@v4
```

---

# Combining run and uses

A production job normally uses both.

Example:

```yaml
steps:

  - name: Checkout Code
    uses: actions/checkout@v4

  - name: Setup Java
    uses: actions/setup-java@v4
    with:
      java-version: '21'
      distribution: temurin

  - name: Build
    run: mvn clean package

  - name: Run Unit Tests
    run: mvn test

  - name: Upload Artifact
    uses: actions/upload-artifact@v4
    with:
      name: application
      path: target/*.jar
```

Architecture:

```text
uses
  |
  └── Checkout

uses
  |
  └── Setup Java

run
  |
  └── Maven Build

run
  |
  └── Maven Test

uses
  |
  └── Upload Artifact
```

---

# Complete Java CI Example

```yaml
name: Java CI

on:
  push:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: temurin
          cache: maven

      - name: Build Application
        run: mvn clean package

      - name: Run Unit Tests
        run: mvn test

      - name: Upload Artifact
        uses: actions/upload-artifact@v4
        with:
          name: application
          path: target/*.jar
```

This workflow demonstrates the normal combination:

```text
uses → setup

run → build

run → test

uses → artifact
```

---

# Action Inputs with with

Actions can receive configuration through `with`.

Example:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: temurin
```

Here:

```text
uses
  |
  ↓
actions/setup-java@v4
  |
  ├── java-version
  └── distribution
```

`with` is commonly used to configure an Action.

---

# run with Environment Variables

A `run` step can receive environment variables.

```yaml
- name: Build
  env:
    ENVIRONMENT: qa
  run: |
    echo "Environment: $ENVIRONMENT"
    mvn clean package
```

The shell command can access:

```text
$ENVIRONMENT
```

---

# uses with Secrets

Actions can also receive values from secrets when supported by the Action.

Example:

```yaml
- name: Deploy
  uses: example/deploy-action@v1
  with:
    token: ${{ secrets.DEPLOYMENT_TOKEN }}
```

Secrets should never be hardcoded.

---

# run with Secrets

Commands can receive secrets through environment variables.

```yaml
- name: Deploy
  env:
    DEPLOYMENT_TOKEN: ${{ secrets.DEPLOYMENT_TOKEN }}
  run: ./deploy.sh
```

The script can access:

```text
$DEPLOYMENT_TOKEN
```

Never print the secret.

---

# Production Security

When using either `run` or `uses`, apply least privilege.

For example:

```yaml
permissions:
  contents: read
```

Only grant additional permissions when required.

Avoid giving every step access to:

- Repository write permissions
- Deployment credentials
- Cloud credentials
- Production secrets

---

# Trusted Actions

Actions are executable code.

Therefore, do not blindly use random Actions from the marketplace.

Before using an Action, review:

- Repository
- Maintainer
- Source code
- Version
- Permissions
- Security history
- Required inputs
- Required secrets

For enterprise environments, approved Actions should preferably be standardized.

---

# Action Versioning

Example:

```yaml
uses: actions/checkout@v4
```

The reference:

```text
v4
```

identifies the Action version line.

For highly controlled production environments, organizations may choose to pin Actions to immutable commit SHAs.

Example pattern:

```yaml
uses: actions/checkout@<commit-sha>
```

This provides stronger protection against an unexpected moving reference.

The trade-off is increased maintenance because SHA updates must be managed intentionally.

---

# Why Version Pinning Matters

Suppose a workflow uses:

```yaml
uses: some-org/some-action@main
```

The `main` branch can change.

A future change could affect your pipeline unexpectedly.

A versioned reference:

```yaml
uses: some-org/some-action@v1
```

provides more predictable behavior.

An immutable SHA provides even stronger reproducibility.

Enterprise strategy:

```text
Development
    |
    ↓
Test Action Version
    |
    ↓
Security Review
    |
    ↓
Approved Version
    |
    ↓
Production
```

---

# run and Scripts

Instead of putting a large script directly into YAML:

```yaml
- name: Deploy
  run: |
    command1
    command2
    command3
    command4
    command5
    command6
    command7
```

you can maintain a script in the repository:

```text
scripts/
└── deploy.sh
```

Then:

```yaml
- name: Deploy
  run: ./scripts/deploy.sh
```

Advantages:

- Easier testing
- Better version control
- Cleaner workflow
- Reusable scripts
- Easier local execution

---

# Example Repository Structure

```text
repository/

├── .github/
│   └── workflows/
│       └── ci.yml
│
├── scripts/
│   ├── build.sh
│   ├── test.sh
│   └── deploy.sh
│
├── Dockerfile
├── pom.xml
└── README.md
```

Workflow:

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Build
    run: ./scripts/build.sh

  - name: Test
    run: ./scripts/test.sh

  - name: Deploy
    run: ./scripts/deploy.sh
```

---

# run for Existing DevOps Tools

`run` is ideal when you already know the CLI command.

Examples:

## Maven

```yaml
- name: Build
  run: mvn clean package
```

## Terraform

```yaml
- name: Terraform Plan
  run: terraform plan
```

## Docker

```yaml
- name: Docker Build
  run: docker build -t catalogue:${GITHUB_SHA} .
```

## Helm

```yaml
- name: Helm Upgrade
  run: |
    helm upgrade --install catalogue ./helm/catalogue \
      --namespace catalogue \
      --set image.tag="${IMAGE_TAG}"
```

## kubectl

```yaml
- name: Kubernetes Status
  run: kubectl get pods -A
```

## Bash

```yaml
- name: Run Script
  run: ./scripts/deploy.sh
```

---

# uses for Standardized Functionality

`uses` is ideal when the functionality has already been packaged as an Action.

Common examples:

```text
Checkout
   ↓
actions/checkout

Java Setup
   ↓
actions/setup-java

Node Setup
   ↓
actions/setup-node

Python Setup
   ↓
actions/setup-python

Terraform Setup
   ↓
hashicorp/setup-terraform

Artifact Upload
   ↓
actions/upload-artifact

Artifact Download
   ↓
actions/download-artifact
```

---

# Enterprise DevSecOps Example

A production CI workflow may look like:

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Setup Java
    uses: actions/setup-java@v4
    with:
      java-version: '21'
      distribution: temurin

  - name: Build
    run: mvn clean package

  - name: Unit Tests
    run: mvn test

  - name: SonarQube Scan
    run: ./scripts/sonarqube.sh

  - name: Docker Build
    run: docker build -t catalogue:${GITHUB_SHA} .

  - name: Trivy Scan
    run: trivy image catalogue:${GITHUB_SHA}

  - name: Push Image
    run: ./scripts/push-image.sh
```

Here:

```text
uses
  |
  ├── Checkout
  └── Java Setup

run
  |
  ├── Build
  ├── Tests
  ├── SonarQube
  ├── Docker Build
  ├── Trivy
  └── Push
```

---

# Production Deployment Example

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Deploy with Helm
    run: |
      helm upgrade --install catalogue ./helm/catalogue \
        --namespace catalogue \
        --create-namespace \
        --set image.tag="${IMAGE_TAG}"

  - name: Verify Rollout
    run: |
      kubectl rollout status deployment/catalogue \
        -n catalogue \
        --timeout=5m

  - name: Smoke Test
    run: ./scripts/smoke-test.sh

  - name: Collect Diagnostics
    if: ${{ failure() }}
    run: |
      kubectl get pods -n catalogue
      kubectl get events -n catalogue
```

This demonstrates that production workflows commonly combine reusable Actions with direct commands.

---

# Production Rule

Do not choose `uses` simply because an Action exists.

Ask:

```text
Can an approved Action safely perform this operation?

        |
        ├── YES → uses
        |
        └── NO → run / approved script
```

For organization-specific deployment logic, a controlled script or approved internal Action may be preferable.

---

# Custom Actions

Organizations can create their own Actions.

Example concept:

```text
Company Platform Repository

        |
        ↓
Internal Deployment Action

        |
        ↓
Application Repositories
```

Application workflow:

```yaml
- name: Deploy Application
  uses: company/platform-deploy-action@v1
```

This can standardize deployment behavior across many repositories.

---

# Composite Actions

Composite Actions can package multiple shell commands and Actions into a reusable step.

Conceptually:

```text
Composite Action

├── Checkout
├── Setup
├── Build
└── Test
```

Then another workflow can use it as:

```yaml
- name: Standard CI
  uses: company/standard-ci@v1
```

Composite Actions are different from reusable workflows.

A reusable workflow can contain jobs.

A composite Action is used as a step.

---

# run vs Composite Action vs Reusable Workflow

```text
run
 |
 └── Execute command

Composite Action
 |
 └── Reuse multiple steps

Reusable Workflow
 |
 └── Reuse complete jobs/workflow logic
```

Example:

```text
run
 ↓
mvn test
```

```text
Composite Action
 ↓
Setup Java
 ↓
Cache
 ↓
Build
 ↓
Test
```

```text
Reusable Workflow
 ↓
Build Job
 ↓
Test Job
 ↓
Security Job
 ↓
Artifact Job
```

---

# Troubleshooting run

If a `run` step fails, check:

```text
1. Command syntax
2. Tool installation
3. PATH
4. Working directory
5. Environment variables
6. Permissions
7. Input files
8. Secrets
9. Runner OS
10. External dependencies
```

Example:

```text
mvn: command not found
```

Possible cause:

```text
Java / Maven environment not configured
```

Solution:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
```

and ensure Maven is available or use the appropriate build setup.

---

# Troubleshooting uses

If an Action fails, check:

```text
1. Action repository
2. Action version
3. Inputs
4. Permissions
5. Secrets
6. Runner compatibility
7. Action logs
8. Action source code
9. Version compatibility
```

Example:

```yaml
uses: actions/checkout@v4
```

If the step fails, inspect the Action output and configuration rather than assuming the shell command is the problem.

---

# Security Troubleshooting

If an Action requires unexpected permissions:

```text
Action
  |
  ↓
Required Permission
  |
  ↓
Workflow Permission
```

Review whether the permission is genuinely necessary.

Avoid:

```yaml
permissions: write-all
```

unless there is a clearly justified organizational requirement.

Prefer:

```yaml
permissions:
  contents: read
```

and grant additional permissions only where required.

---

# Common Mistakes

## Mistake 1 - Using run for Everything

Example:

```yaml
- name: Checkout
  run: git clone ...
```

If an approved checkout Action is available, use:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

This provides maintained, reusable functionality.

---

## Mistake 2 - Using Random Actions

Do not blindly use:

```yaml
uses: unknown-user/random-action@main
```

Review the Action first.

---

## Mistake 3 - Hardcoding Secrets

Bad:

```yaml
run: ./deploy.sh --token=my-secret
```

Use:

```yaml
env:
  TOKEN: ${{ secrets.DEPLOYMENT_TOKEN }}
```

---

## Mistake 4 - Using Moving Branches for Critical Actions

Avoid relying on:

```yaml
uses: company/deploy-action@main
```

for critical production workflows unless the organization's governance explicitly accepts this model.

Prefer an approved version or immutable reference.

---

## Mistake 5 - One Giant run Step

Avoid:

```yaml
- name: Pipeline
  run: |
    build
    test
    scan
    deploy
    rollback
    notify
```

Separate responsibilities into clear steps and jobs.

---

# Best Practices

- Use `run` for commands and scripts.
- Use `uses` for approved reusable Actions.
- Use official or organization-approved Actions where possible.
- Review third-party Actions before adoption.
- Pin critical Actions appropriately.
- Use least-privilege permissions.
- Never hardcode secrets.
- Use `with` for Action configuration.
- Use scripts for complex organization-specific commands.
- Keep steps focused.
- Separate production deployment from validation where appropriate.
- Use reusable workflows or composite Actions to eliminate repeated logic.
- Test Actions and scripts before production adoption.

---

# Quick Decision Guide

Use this mental model:

```text
Do I need to execute a shell command?

        |
       YES
        ↓
       run
```

```text
Do I need functionality provided by a reusable Action?

        |
       YES
        ↓
       uses
```

```text
Do I need to reuse multiple steps?

        |
       YES
        ↓
Composite Action
```

```text
Do I need to reuse multiple jobs?

        |
       YES
        ↓
Reusable Workflow
```

---

# Summary

The two primary step execution mechanisms are:

```text
run
```

and:

```text
uses
```

`run` executes commands:

```yaml
- name: Build
  run: mvn clean package
```

`uses` executes a reusable Action:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

Production workflows commonly combine both:

```text
uses
  |
  ├── Checkout
  ├── Setup Runtime
  └── Upload Artifact

run
  |
  ├── Build
  ├── Test
  ├── Security Scan
  └── Deploy
```

The key principle is:

```text
Use approved Actions for reusable functionality.
Use run for commands and organization-specific automation.
```

---

# Interview Questions

## Basic

1. What is the difference between `run` and `uses`?
2. When would you use `run`?
3. When would you use `uses`?
4. Give three examples of Actions commonly used with `uses`.
5. Can a workflow use both `run` and `uses`?

## Intermediate

6. What is the purpose of `with`?
7. How do you pass environment variables to a `run` step?
8. How do you pass secrets to a step?
9. How do you execute multiple shell commands in a `run` step?
10. What is the difference between a custom shell script and a reusable Action?
11. What is a composite Action?
12. What is the difference between a composite Action and a reusable workflow?

## Advanced

13. Design a production Java CI workflow that combines `uses` for checkout/runtime setup and `run` for Maven build, testing, security scanning, and deployment.
14. How would you evaluate whether a third-party Action is safe to use in an enterprise production pipeline?
15. Why is pinning or versioning Actions important for production?
16. A third-party Action suddenly changes behavior and breaks production CI. How would you investigate and prevent this in the future?
17. Explain how you would design an enterprise-approved internal Action for standardized Kubernetes deployments.
18. A team has duplicated 15 shell commands across 50 repositories. Explain when you would use a composite Action versus a reusable workflow.
19. Explain how least-privilege permissions apply to Actions used through `uses`.
20. Design a production GitHub Actions pipeline using `run`, `uses`, artifacts, security gates, Helm deployment, rollout validation, and failure diagnostics.