# Composite Actions

A **composite action** allows you to combine multiple workflow steps into a reusable Action.

Instead of repeating the same sequence of steps across multiple workflows, you can package those steps into a composite Action and call it using `uses:`.

Conceptually:

```text
Without Composite Action

Workflow A
 ├── Checkout
 ├── Setup
 ├── Build
 └── Test

Workflow B
 ├── Checkout
 ├── Setup
 ├── Build
 └── Test

Workflow C
 ├── Checkout
 ├── Setup
 ├── Build
 └── Test
```

With a composite Action:

```text
                 Composite Action
                /       |       \
               ↓        ↓        ↓
         Workflow A  Workflow B  Workflow C
```

---

# Why Composite Actions?

Composite Actions are useful when multiple workflows repeatedly execute the same sequence of commands.

Example:

```text
Checkout
Setup Java
Configure Maven
Run Tests
```

Instead of duplicating these steps everywhere:

```text
Workflow A → repeated steps
Workflow B → repeated steps
Workflow C → repeated steps
```

Create:

```text
Java Build Composite Action
```

Then reuse it.

---

# Basic Structure

A repository can contain:

```text
repository/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
└── .github/
    └── actions/
        └── java-build/
            └── action.yml
```

The important file is:

```text
action.yml
```

It defines the composite Action.

---

# `action.yml`

Basic example:

```yaml
name: Java Build

description: Build and test a Java application

runs:
  using: composite

  steps:

    - name: Setup Java
      uses: actions/setup-java@v4
      with:
        distribution: temurin
        java-version: '21'

    - name: Build
      shell: bash
      run: mvn clean package

    - name: Test
      shell: bash
      run: mvn test
```

The key part is:

```yaml
runs:
  using: composite
```

---

# Calling a Composite Action

Suppose the Action is located at:

```text
.github/actions/java-build/action.yml
```

Workflow:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Java Build
        uses: ./.github/actions/java-build
```

The workflow now reuses the complete sequence.

---

# Composite Action Flow

```text
Workflow
   |
   ↓
uses: ./.github/actions/java-build
   |
   ↓
Composite Action
   |
   ├── Setup Java
   ├── Maven Build
   └── Maven Test
```

---

# Composite Actions Can Contain Multiple Steps

Example:

```yaml
runs:
  using: composite

  steps:

    - name: Step 1
      shell: bash
      run: echo "Step 1"

    - name: Step 2
      shell: bash
      run: echo "Step 2"

    - name: Step 3
      shell: bash
      run: echo "Step 3"
```

This allows several commands and Actions to be packaged together.

---

# Composite Action with `run`

Example:

```yaml
runs:
  using: composite

  steps:

    - name: Display Information
      shell: bash
      run: |
        echo "Repository: $GITHUB_REPOSITORY"
        echo "Commit: $GITHUB_SHA"
```

Important:

For composite Actions, specify the shell for `run` steps.

Example:

```yaml
shell: bash
```

---

# Composite Action with Existing Actions

Composite Actions can use other Actions.

Example:

```yaml
runs:
  using: composite

  steps:

    - name: Setup Node
      uses: actions/setup-node@v4
      with:
        node-version: '22'

    - name: Install
      shell: bash
      run: npm ci
```

So a composite Action can combine:

```text
Existing Actions
+
Shell Commands
```

---

# Inputs

Composite Actions can accept inputs.

Example:

```yaml
name: Application Build

inputs:

  java-version:
    description: Java version
    required: true
    default: '21'

runs:
  using: composite

  steps:

    - name: Setup Java
      uses: actions/setup-java@v4
      with:
        distribution: temurin
        java-version: ${{ inputs.java-version }}

    - name: Build
      shell: bash
      run: mvn clean package
```

---

# Calling with Inputs

Workflow:

```yaml
- name: Build Application
  uses: ./.github/actions/java-build
  with:
    java-version: '21'
```

The value:

```text
21
```

is passed into:

```yaml
${{ inputs.java-version }}
```

---

# Why Inputs Matter

Inputs make composite Actions reusable.

Without inputs:

```text
Java 17 only
```

With inputs:

```text
Java 17
Java 21
Java 22
```

The same Action can support different requirements.

---

# Multiple Inputs

Example:

```yaml
inputs:

  java-version:
    description: Java version
    required: true

  build-command:
    description: Maven build command
    required: false
    default: mvn clean package
```

Usage:

```yaml
- name: Build
  uses: ./.github/actions/java-build
  with:
    java-version: '21'
    build-command: mvn clean verify
```

---

# Input Validation

Composite Actions can use inputs, but validation should be handled deliberately.

Example:

```yaml
- name: Validate Environment
  shell: bash
  run: |
    case "${{ inputs.environment }}" in
      dev|qa|uat|production)
        echo "Valid environment"
        ;;
      *)
        echo "Invalid environment"
        exit 1
        ;;
    esac
```

This prevents unsupported values from being silently accepted.

---

# Outputs

Composite Actions can expose outputs.

Example:

```yaml
name: Build Image

outputs:

  image-tag:
    description: Generated image tag
    value: ${{ steps.build.outputs.image-tag }}

runs:
  using: composite

  steps:

    - name: Build
      id: build
      shell: bash
      run: |
        TAG="${GITHUB_SHA}"
        echo "image-tag=$TAG" >> "$GITHUB_OUTPUT"
```

The Action exposes:

```text
image-tag
```

---

# Consuming Composite Action Output

Workflow:

```yaml
- name: Build Image
  id: image
  uses: ./.github/actions/build-image

- name: Display Tag
  run: |
    echo "Image Tag: ${{ steps.image.outputs.image-tag }}"
```

Flow:

```text
Composite Action
      |
      ↓
Output
      |
      ↓
Workflow
      |
      ↓
Next Step
```

---

# Inputs vs Outputs

### Inputs

Information going **into** the Action.

```text
Workflow
   |
   ↓
Input
   |
   ↓
Composite Action
```

### Outputs

Information coming **out of** the Action.

```text
Composite Action
   |
   ↓
Output
   |
   ↓
Workflow
```

---

# Environment Variables

Composite Actions can use environment variables.

Example:

```yaml
- name: Build
  shell: bash
  env:
    APP_ENV: ${{ inputs.environment }}
  run: |
    echo "Environment: $APP_ENV"
```

Prefer explicit inputs and environment variables instead of hidden dependencies.

---

# GitHub Context

Composite Actions can access GitHub context through expressions.

Example:

```yaml
- name: Display Commit
  shell: bash
  run: |
    echo "Commit: ${{ github.sha }}"
```

Common context values:

```text
github.repository
github.sha
github.ref
github.actor
github.event_name
```

---

# Composite Action Directory

A recommended structure:

```text
.github/
│
├── actions/
│   ├── java-build/
│   │   └── action.yml
│   │
│   ├── docker-build/
│   │   └── action.yml
│   │
│   └── security-scan/
│       └── action.yml
│
└── workflows/
    ├── ci.yml
    └── cd.yml
```

---

# Composite Action for Docker Build

Example:

```yaml
name: Docker Build

inputs:

  image-name:
    description: Docker image name
    required: true

  image-tag:
    description: Docker image tag
    required: true

runs:
  using: composite

  steps:

    - name: Build Docker Image
      shell: bash
      run: |
        docker build \
          -t "${{ inputs.image-name }}:${{ inputs.image-tag }}" \
          .
```

---

# Calling Docker Build Action

```yaml
- name: Docker Build
  uses: ./.github/actions/docker-build
  with:
    image-name: catalogue
    image-tag: ${{ github.sha }}
```

---

# Docker Build with Security Scan

A more useful internal Action could standardize:

```text
Docker Build
      |
      ↓
Image Scan
      |
      ↓
Tag
```

Example concept:

```yaml
runs:
  using: composite

  steps:

    - name: Docker Build
      shell: bash
      run: |
        docker build \
          -t "${{ inputs.image-name }}:${{ inputs.image-tag }}" .

    - name: Trivy Scan
      shell: bash
      run: |
        trivy image \
          --severity HIGH,CRITICAL \
          --exit-code 1 \
          "${{ inputs.image-name }}:${{ inputs.image-tag }}"
```

This can standardize security gates across services.

---

# DevSecOps Composite Action

For a DevSecOps platform:

```text
Composite Action
 |
 ├── Build
 ├── SonarQube
 ├── Trivy
 └── Veracode
```

Application workflows can reuse the standardized implementation.

---

# Microservices Example

Suppose the platform contains:

```text
user
catalogue
cart
orders
payment
inventory
notification
```

Each service needs:

```text
Checkout
Build
Test
Security Scan
Docker Build
```

Without reuse:

```text
Service 1 → duplicate logic
Service 2 → duplicate logic
Service 3 → duplicate logic
...
```

With composite Actions:

```text
                   Composite Actions
                          |
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
      User            Catalogue             Cart
       ↓                  ↓                  ↓
    Workflow           Workflow           Workflow
```

---

# Standardized Build Action

Example:

```text
.github/actions/java-service-build/
└── action.yml
```

It can standardize:

```text
Java Setup
Maven
Tests
SonarQube
Artifact
```

Application workflow:

```yaml
- name: Standard Java Build
  uses: ./.github/actions/java-service-build
  with:
    java-version: '21'
```

---

# Composite Action for Terraform

Example:

```yaml
name: Terraform Validation

inputs:

  working-directory:
    description: Terraform directory
    required: true

runs:
  using: composite

  steps:

    - name: Terraform Format
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: terraform fmt -check

    - name: Terraform Init
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: terraform init -backend=false

    - name: Terraform Validate
      shell: bash
      working-directory: ${{ inputs.working-directory }}
      run: terraform validate
```

---

# Calling Terraform Composite Action

```yaml
- name: Terraform Validation
  uses: ./.github/actions/terraform-validation
  with:
    working-directory: infrastructure/
```

This makes Terraform validation consistent across repositories.

---

# Composite Action for Helm

Example concept:

```yaml
name: Helm Validation

inputs:

  chart-path:
    description: Helm chart path
    required: true

runs:
  using: composite

  steps:

    - name: Helm Lint
      shell: bash
      run: |
        helm lint "${{ inputs.chart-path }}"

    - name: Helm Template
      shell: bash
      run: |
        helm template test "${{ inputs.chart-path }}"
```

---

# Composite Action for Kubernetes

Example:

```yaml
name: Kubernetes Validation

inputs:

  namespace:
    description: Kubernetes namespace
    required: true

runs:
  using: composite

  steps:

    - name: Check Deployment
      shell: bash
      run: |
        kubectl get deployments \
          -n "${{ inputs.namespace }}"
```

Be careful with deployment permissions when turning validation Actions into deployment Actions.

---

# Composite Action for GitOps

A GitOps helper could standardize:

```text
Update image tag
Commit
Push
```

Conceptually:

```text
Build Image
     |
     ↓
Push ECR
     |
     ↓
Composite Action
     |
     ├── Update manifest
     ├── Commit
     └── Push
     |
     ↓
ArgoCD
```

---

# Composite Action and GitOps Security

If a composite Action pushes to Git:

```text
Action
   |
   ↓
GitHub Repository
```

It may need:

```yaml
permissions:
  contents: write
```

Only grant this to the specific job that needs it.

Do not give write access to the entire workflow unnecessarily.

---

# Composite Actions and Secrets

Composite Actions should avoid requiring secrets unless necessary.

Bad design:

```text
Composite Action
   |
   └── Hardcoded credential
```

Never hardcode:

```text
AWS keys
Passwords
Tokens
API keys
```

---

# Passing Secrets

Secrets can be passed through environment variables or inputs when required.

Example:

```yaml
- name: Deploy
  uses: ./.github/actions/deploy
  env:
    DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

The Action should use the secret only for the required operation.

---

# Composite Action and OIDC

For cloud authentication, prefer OIDC where supported rather than passing long-lived credentials.

Conceptually:

```text
Workflow
   |
   ↓
OIDC
   |
   ↓
Cloud IAM
   |
   ↓
Temporary Credentials
```

The composite Action can then execute cloud commands within that authenticated context.

---

# Composite Action Security

A composite Action executes commands in the caller's workflow environment.

Therefore:

```text
Composite Action
       |
       ↓
Runner
       |
       ↓
Workflow Permissions
       |
       ↓
Secrets / Infrastructure
```

Treat internal composite Actions as trusted code.

---

# Internal Composite Action Trust

If your organization maintains:

```text
company/platform-actions
```

then changes to these Actions can affect many repositories.

Therefore:

```text
Code Review
+
Branch Protection
+
Testing
+
Security Review
+
Versioning
```

are important.

---

# Versioning Internal Composite Actions

For an Action stored in the same repository:

```yaml
uses: ./.github/actions/build
```

the workflow uses the current repository revision.

For a centralized Action repository:

```yaml
uses: company/platform-actions/build@v1
```

you can manage versions independently.

---

# Centralized Action Repository

Enterprise structure:

```text
company/
 |
 └── platform-actions/
      |
      ├── java-build
      ├── docker-build
      ├── terraform
      ├── security-scan
      └── deploy
```

Application repositories consume them.

---

# Monorepo vs Central Action Repository

### Local repository

```text
Application
 |
 └── .github/actions/
```

Advantages:

```text
Simple
Close to application
Easy development
```

### Central repository

```text
platform-actions
```

Advantages:

```text
Centralized
Reusable
Standardized
Versioned
```

Trade-off:

```text
Central dependency management required
```

---

# Composite Action vs Script

A shell script:

```text
build.sh
```

can package commands.

A composite Action:

```text
action.yml
```

can package:

```text
Shell Commands
+
Other Actions
+
Inputs
+
Outputs
```

Use a composite Action when workflow-level reuse is valuable.

---

# Composite Action vs Docker Action

### Composite Action

Runs steps directly on the runner.

```text
Runner
 |
 ├── Shell
 ├── Existing Actions
 └── Tools
```

### Docker Action

Runs the Action inside a Docker container.

```text
Runner
 |
 └── Docker Container
       |
       └── Action
```

Docker Actions are covered in:

```text
04-Docker-Actions.md
```

---

# Composite Action vs JavaScript Action

### Composite

```text
YAML
+
Shell
+
Existing Actions
```

### JavaScript

```text
Node.js
+
JavaScript
```

JavaScript Actions are covered in:

```text
05-JavaScript-Actions.md
```

---

# When to Use Composite Actions

Good use cases:

```text
Repeated build process
Repeated test setup
Standard security scan
Common Docker build
Common Terraform validation
Common Helm validation
Standard repository setup
Common deployment preparation
```

---

# When Not to Use Composite Actions

Avoid creating a composite Action for:

```text
One simple command
One workflow-specific operation
Tiny logic that is never reused
Highly specialized application logic
```

Example:

```yaml
- run: npm test
```

does not necessarily need a custom Action.

---

# Composite Action Design Principle

A good Action should have:

```text
Clear Purpose
Small Scope
Stable Interface
Documented Inputs
Documented Outputs
Predictable Behavior
Minimal Permissions
Good Error Handling
```

---

# Bad Composite Action

Example concept:

```text
mega-deployment-action
 |
 ├── Build
 ├── Test
 ├── Scan
 ├── Terraform
 ├── Docker
 ├── Kubernetes
 ├── Database
 ├── Notifications
 └── Production Deployment
```

This becomes difficult to:

```text
Understand
Test
Version
Debug
Reuse
```

---

# Better Design

Split responsibilities:

```text
java-build
docker-build
security-scan
terraform-validation
helm-validation
deployment
```

Then compose them in the workflow.

---

# Composite Action Error Handling

Example:

```yaml
- name: Security Scan
  shell: bash
  run: |
    trivy fs \
      --severity HIGH,CRITICAL \
      --exit-code 1 \
      .
```

If critical vulnerabilities are found:

```text
Exit Code 1
     |
     ↓
Action Fails
     |
     ↓
Workflow Stops
```

This is useful when the scan is intended as a release gate.

---

# Logging

Composite Actions should provide useful logs.

Example:

```yaml
- name: Build
  shell: bash
  run: |
    echo "Starting Maven build..."
    mvn clean package
    echo "Build completed."
```

Avoid printing secrets.

---

# Error Messages

Bad:

```text
Command failed
```

Better:

```text
Terraform validation failed.
Check formatting and configuration in infrastructure/.
```

Good error messages reduce troubleshooting time.

---

# Exit Codes

Composite Actions should use meaningful exit codes.

Example:

```bash
if ! terraform validate; then
  echo "Terraform validation failed."
  exit 1
fi
```

A non-zero exit code generally causes the step to fail.

---

# Production Composite Action

Example:

```yaml
name: Docker Build and Scan

description: Build and scan a Docker image

inputs:

  image-name:
    description: Docker image name
    required: true

  image-tag:
    description: Docker image tag
    required: true

outputs:

  image:
    description: Final image reference
    value: ${{ steps.image.outputs.image }}

runs:
  using: composite

  steps:

    - name: Build Image
      shell: bash
      run: |
        docker build \
          -t "${{ inputs.image-name }}:${{ inputs.image-tag }}" \
          .

    - name: Scan Image
      shell: bash
      run: |
        trivy image \
          --severity HIGH,CRITICAL \
          --exit-code 1 \
          "${{ inputs.image-name }}:${{ inputs.image-tag }}"

    - name: Set Image Output
      id: image
      shell: bash
      run: |
        echo "image=${{ inputs.image-name }}:${{ inputs.image-tag }}" \
          >> "$GITHUB_OUTPUT"
```

---

# Calling Production Composite Action

```yaml
jobs:

  build:

    runs-on:
      - self-hosted
      - linux
      - ci

    permissions:
      contents: read

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build and Scan
        id: image
        uses: ./.github/actions/docker-build
        with:
          image-name: catalogue
          image-tag: ${{ github.sha }}

      - name: Display Image
        run: |
          echo "Built: ${{ steps.image.outputs.image }}"
```

---

# Composite Action in CI/CD

A mature pipeline can look like:

```text
Checkout
   |
   ↓
Composite Build Action
   |
   ├── Setup
   ├── Build
   └── Test
   |
   ↓
Composite Security Action
   |
   ├── SonarQube
   ├── Trivy
   └── Veracode
   |
   ↓
Composite Docker Action
   |
   ├── Build
   └── Scan
   |
   ↓
Push ECR
   |
   ↓
GitOps
   |
   ↓
ArgoCD
   |
   ↓
EKS
```

---

# Composite Actions and Self-Hosted Runners

Composite Actions execute on the runner selected by the workflow.

Example:

```yaml
runs-on:
  - self-hosted
  - linux
  - ci
```

Then:

```yaml
uses: ./.github/actions/docker-build
```

The Action runs on that runner.

---

# Composite Actions and ARC

With ARC:

```text
Workflow
   |
   ↓
ARC Runner
   |
   ↓
Composite Action
   |
   ├── Build
   ├── Scan
   └── Test
```

The Action uses the tools available in the runner environment.

---

# Tool Dependency

A composite Action may require:

```text
Java
Maven
Docker
Terraform
Helm
kubectl
Trivy
```

Document these requirements.

Example:

```text
Requirements:
- Linux
- Bash
- Docker
- Trivy
```

Do not assume every runner has the same tools.

---

# Tool Setup Inside the Action

A composite Action can use setup Actions.

Example:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '21'
```

This reduces dependency on preconfigured runners.

---

# Self-Hosted Runner Consideration

For self-hosted runners, do not blindly assume:

```text
Docker installed
kubectl installed
Helm installed
Terraform installed
```

Either:

```text
Install through a controlled runner image
```

or:

```text
Set up the tools explicitly
```

---

# Composite Action Testing

Before production:

```text
Create test workflow
      |
      ↓
Run on test branch
      |
      ↓
Test Inputs
      |
      ↓
Test Outputs
      |
      ↓
Test Failure Cases
      |
      ↓
Security Review
      |
      ↓
Production
```

---

# Testing Multiple Inputs

If Action supports:

```text
dev
qa
uat
production
```

test all supported values.

Also test:

```text
Invalid input
Missing input
Unexpected input
```

---

# Testing Failure Cases

Test:

```text
Build failure
Scan failure
Network failure
Missing tool
Invalid input
Missing secret
Invalid credentials
```

The Action should fail clearly.

---

# Composite Action Security Checklist

```text
☐ No hardcoded credentials
☐ Minimal permissions
☐ Minimal secrets
☐ Trusted dependencies
☐ Reviewed source code
☐ Input validation
☐ Secure shell handling
☐ No secret logging
☐ Controlled Action versions
☐ Tested failure behavior
```

---

# Shell Injection Consideration

Be careful when using user-controlled inputs in shell commands.

Potentially unsafe pattern:

```yaml
run: |
  ./deploy.sh ${{ inputs.environment }}
```

Validate inputs before using them.

Better:

```yaml
env:
  TARGET_ENV: ${{ inputs.environment }}

run: |
  case "$TARGET_ENV" in
    dev|qa|uat|production)
      ./deploy.sh "$TARGET_ENV"
      ;;
    *)
      echo "Invalid environment"
      exit 1
      ;;
  esac
```

---

# Composite Action Versioning

For organization-wide Actions:

```text
v1
v2
v3
```

Use releases to communicate changes.

Example:

```yaml
uses: company/platform-actions/docker-build@v1
```

A breaking change can become:

```yaml
uses: company/platform-actions/docker-build@v2
```

---

# Backward Compatibility

When changing a widely used Action:

```text
Current:
v1

New:
v2
```

Avoid unexpectedly breaking all consumers.

Use:

```text
Versioning
Documentation
Migration Guide
Testing
Deprecation Period
```

---

# Action Ownership

Every shared Action should have an owner.

Example:

```text
docker-build
    |
    └── Platform Engineering

terraform-validation
    |
    └── Cloud Platform

security-scan
    |
    └── DevSecOps
```

Without ownership, Actions can become abandoned dependencies.

---

# Composite Action Documentation

Each shared Action should document:

```text
Purpose
Usage
Inputs
Outputs
Requirements
Supported Environments
Permissions
Secrets
Examples
Failure Behavior
Version
Owner
```

---

# Example README

```markdown
# Docker Build Action

## Purpose

Build and scan Docker images.

## Inputs

- image-name
- image-tag

## Outputs

- image

## Requirements

- Docker
- Trivy

## Example

uses: company/platform-actions/docker-build@v1
```

---

# Action Governance

For enterprise shared Actions:

```text
Developer
   |
   ↓
Pull Request
   |
   ↓
Code Review
   |
   ↓
Security Review
   |
   ↓
Automated Tests
   |
   ↓
Release
   |
   ↓
Application Teams
```

---

# Common Mistakes

### 1. Creating Actions for everything

Not every command needs an Action.

### 2. Creating a giant Action

Keep responsibilities focused.

### 3. Hardcoding secrets

Never do this.

### 4. Not documenting inputs

Consumers need a stable interface.

### 5. Not validating inputs

Invalid input can cause unexpected behavior.

### 6. Using unpinned third-party Actions

Review dependencies.

### 7. Giving excessive permissions

Use least privilege.

### 8. Not testing failure cases

Production failures become harder to troubleshoot.

### 9. No owner

Shared Actions need maintenance.

### 10. Breaking all consumers with an update

Use versioning.

---

# Best Practices

- Use composite Actions to remove repeated workflow logic.
- Keep each Action focused on a clear responsibility.
- Use inputs to make Actions reusable.
- Use outputs to return useful values.
- Validate important inputs.
- Specify shells explicitly for `run` steps.
- Avoid hardcoded secrets.
- Use least-privilege permissions.
- Review third-party Actions used inside composite Actions.
- Version organization-wide Actions.
- Test Actions before production.
- Document requirements and behavior.
- Assign ownership.
- Maintain backward compatibility.
- Use meaningful error messages.
- Avoid exposing secrets in logs.
- Prefer OIDC for supported cloud authentication.
- Keep production Actions small and auditable.

---

# Composite Action Design Pattern

A good reusable Action:

```text
             Composite Action
                    |
        ┌───────────┼───────────┐
        ↓           ↓           ↓
      Input      Processing    Output
        |           |           |
        ↓           ↓           ↓
 environment     Build       image-tag
 version          Scan
```

---

# Enterprise Platform Pattern

```text
                 Platform Actions
                        |
       ┌────────────────┼────────────────┐
       ↓                ↓                ↓
   Java Build       Docker Build    Security Scan
       |                |                |
       └────────────────┼────────────────┘
                        ↓
                Application Workflows
                        |
              ┌─────────┼─────────┐
              ↓         ↓         ↓
            User    Catalogue    Cart
```

This is a common platform-engineering pattern.

---

# Key Takeaways

```text
Composite Action
=
Reusable collection of workflow steps
```

It can contain:

```text
Shell Commands
+
Other Actions
+
Inputs
+
Outputs
```

Basic definition:

```yaml
runs:
  using: composite
```

Basic usage:

```yaml
- name: Build
  uses: ./.github/actions/build
```

For production:

```text
Small
Reusable
Versioned
Tested
Secure
Documented
Owned
```

The key principle is:

```text
Use Composite Actions to standardize repeated workflow logic,
not to hide an unnecessarily complicated pipeline inside one Action.
```

---

# Interview Questions

## Basic

1. What is a Composite Action?
2. Why would you create a Composite Action?
3. How is a Composite Action different from a normal workflow?
4. What does `runs.using: composite` mean?
5. Where is a Composite Action usually defined?
6. How do you call a local Composite Action?
7. Can a Composite Action contain multiple steps?
8. Can a Composite Action use another Action?
9. Can a Composite Action execute shell commands?

## Intermediate

10. How do you pass inputs to a Composite Action?
11. How do you define outputs?
12. How do you consume Composite Action outputs?
13. What is the difference between inputs and outputs?
14. Why should you specify `shell` for `run` steps?
15. How would you create a reusable Docker build Composite Action?
16. How would you create a reusable Terraform validation Action?
17. How would you handle input validation?
18. How would you handle errors inside a Composite Action?
19. What is the difference between a Composite Action and a shell script?
20. What is the difference between a Composite Action and a Docker Action?
21. What is the difference between a Composite Action and a JavaScript Action?
22. When should you avoid creating a Composite Action?

## Advanced / Production

23. Design a reusable Composite Action for building Java microservices.
24. Design a Composite Action for Docker build + Trivy scanning.
25. How would you standardize SonarQube, Trivy, and Veracode through Composite Actions?
26. How would you design a centralized platform-actions repository?
27. How would you version shared Composite Actions used by hundreds of repositories?
28. How would you prevent a breaking change from affecting all consuming repositories?
29. How would you secure a Composite Action that runs on production self-hosted runners?
30. How would you prevent shell injection through Composite Action inputs?
31. How would you handle secrets inside Composite Actions?
32. How would you use OIDC from a Composite Action-based deployment workflow?
33. How would you combine Composite Actions with ARC?
34. How would you combine Composite Actions with GitOps and ArgoCD?
35. How would you design a production Docker build Composite Action that builds, scans, tags, and publishes images to ECR?
36. What security risks exist when a Composite Action uses third-party Actions internally?
37. How would you test a Composite Action before releasing it?
38. What should be included in the documentation of an enterprise Composite Action?
39. How would you troubleshoot a Composite Action that works on GitHub-hosted runners but fails on self-hosted runners?
40. How would you design Composite Actions for a microservices platform while avoiding a single giant "do everything" Action?