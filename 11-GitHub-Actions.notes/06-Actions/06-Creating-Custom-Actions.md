# Creating Custom GitHub Actions

Custom GitHub Actions allow organizations and teams to build reusable automation for their own requirements.

Instead of repeating the same workflow logic across repositories, you can create an internal Action and reuse it.

Conceptually:

```text
Application Repositories
        |
        ├── Service A
        ├── Service B
        ├── Service C
        └── Service D
                |
                ↓
        Custom GitHub Action
                |
        ├── Build
        ├── Security
        ├── Validation
        ├── Deployment
        └── Notifications
```

---

# Why Create Custom Actions?

Create a custom Action when you have repeated logic that should be standardized.

Common examples:

```text
JIRA validation
Deployment validation
Docker build
Security scanning
Terraform validation
Helm validation
GitOps updates
Release automation
Notification
Change-request validation
Environment promotion
```

---

# Types of Custom Actions

GitHub Actions supports different implementation approaches.

```text
Custom Actions
     |
     ├── Composite Actions
     ├── Docker Actions
     └── JavaScript Actions
```

Choose the implementation based on the requirement.

---

# Composite Action

Use when you want to combine:

```text
Shell Commands
+
Existing GitHub Actions
```

Example:

```yaml
runs:
  using: composite
```

Good for:

```text
Build
Test
Validation
Standard setup
Repeated workflow steps
```

---

# Docker Action

Use when you want to package the execution environment.

```yaml
runs:
  using: docker
  image: Dockerfile
```

Good for:

```text
Specialized tooling
Custom dependencies
Custom runtime
Security scanners
Containerized utilities
```

---

# JavaScript Action

Use when you need programmatic logic.

```yaml
runs:
  using: node24
  main: dist/index.js
```

Good for:

```text
API integration
GitHub API
JIRA API
Complex validation
Data processing
Automation
```

---

# Choosing the Action Type

```text
Need reusable workflow steps?
          |
          ↓
    Composite Action

Need custom runtime?
          |
          ↓
      Docker Action

Need complex programmatic logic?
          |
          ↓
    JavaScript Action
```

Use the simplest implementation that satisfies the requirement.

---

# Custom Action Repository

A custom Action can have its own repository.

Example:

```text
company/platform-actions/
│
├── docker-build/
├── terraform-validation/
├── security-scan/
├── jira-validation/
└── deployment-gate/
```

This creates a centralized reusable Action platform.

---

# Multiple Actions in One Repository

An organization can maintain multiple Actions in one repository.

Example:

```text
platform-actions/
│
├── actions/
│   ├── docker-build/
│   │   └── action.yml
│   │
│   ├── terraform-validation/
│   │   └── action.yml
│   │
│   ├── security-scan/
│   │   └── action.yml
│   │
│   └── jira-validation/
│       └── action.yml
│
└── README.md
```

This is useful for platform teams.

---

# Separate Repository per Action

Another approach:

```text
company/
├── docker-build-action
├── terraform-action
├── security-action
└── jira-validation-action
```

Advantages:

```text
Independent lifecycle
Independent versioning
Independent ownership
Smaller repositories
```

Trade-off:

```text
More repositories to manage
```

---

# Central Platform Actions Repository

For an enterprise platform:

```text
company/platform-actions
        |
        ├── Build
        ├── Security
        ├── Cloud
        ├── Infrastructure
        ├── Deployment
        └── Governance
```

Application teams consume approved Actions.

---

# Action Directory

Every Action needs an Action metadata file.

Example:

```text
my-action/
└── action.yml
```

For a larger JavaScript Action:

```text
my-action/
├── action.yml
├── package.json
├── src/
├── dist/
├── tests/
└── README.md
```

---

# `action.yml`

The `action.yml` file defines:

```text
Name
Description
Inputs
Outputs
Runtime
Entry Point
```

Example:

```yaml
name: Docker Build

description: Build and scan Docker images

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
    - name: Build
      shell: bash
      run: |
        docker build \
          -t "${{ inputs.image-name }}:${{ inputs.image-tag }}" \
          .
```

---

# Action Inputs

Inputs make Actions reusable.

Without inputs:

```text
catalogue only
```

With inputs:

```text
catalogue
user
cart
orders
payment
inventory
```

Example:

```yaml
inputs:

  service-name:
    description: Microservice name
    required: true
```

---

# Input Defaults

Inputs can have defaults.

Example:

```yaml
inputs:

  environment:
    description: Target environment
    required: false
    default: qa
```

If the caller doesn't specify:

```text
environment = qa
```

---

# Input Validation

Do not blindly trust inputs.

Example:

```bash
case "$ENVIRONMENT" in
  dev|qa|sit|uat|production)
    echo "Valid environment"
    ;;
  *)
    echo "Invalid environment"
    exit 1
    ;;
esac
```

This is particularly important for deployment Actions.

---

# Outputs

Custom Actions can return values.

Example:

```yaml
outputs:

  image:
    description: Generated image reference
```

The Action can return:

```text
image
version
commit
deployment-status
validation-result
```

---

# Action Interface

Think of an Action as a reusable function.

```text
Input
  |
  ↓
Action
  |
  ↓
Output
```

Example:

```text
JIRA Ticket
Environment
SHA
     |
     ↓
JIRA Validation Action
     |
     ↓
Validation Result
```

---

# Good Action Interface

A good Action should have:

```text
Clear Inputs
Clear Outputs
Predictable Behavior
Minimal Side Effects
Documented Requirements
```

Avoid hidden dependencies.

---

# Example Custom Docker Build Action

```yaml
name: Docker Build

description: Build a microservice Docker image

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

# Calling the Action

```yaml
- name: Build Image
  uses: company/platform-actions/docker-build@v1
  with:
    image-name: catalogue
    image-tag: ${{ github.sha }}
```

---

# Custom Security Action

A platform team could standardize security scanning.

Example:

```text
Security Action
      |
      ├── SonarQube
      ├── Trivy
      └── Veracode
```

The consuming workflow could simply call:

```yaml
- name: Security Scan
  uses: company/platform-actions/security-scan@v1
```

---

# Security Policy

A custom security Action can enforce:

```text
Critical vulnerabilities
High vulnerabilities
Quality gates
Secret detection
Policy violations
```

Example:

```text
Critical > 0
      |
      ↓
Workflow fails
```

---

# Custom Terraform Action

A platform team can create:

```text
terraform-validation
```

that performs:

```text
terraform fmt
terraform init
terraform validate
```

Example:

```yaml
- name: Terraform Validation
  uses: company/platform-actions/terraform-validation@v1
  with:
    working-directory: infrastructure/
```

---

# Custom Helm Action

A reusable Helm validation Action:

```text
helm lint
helm template
```

Example:

```yaml
- name: Helm Validation
  uses: company/platform-actions/helm-validation@v1
  with:
    chart-path: ./helm/catalogue
```

---

# Custom Kubernetes Action

A custom Action can standardize Kubernetes validation.

Example:

```text
Manifest
   |
   ↓
Validation
   |
   ├── kubectl
   ├── Helm
   └── Policy
```

Be careful when turning validation Actions into privileged deployment Actions.

---

# Custom JIRA Validation Action

This is especially useful for controlled production deployments.

Inputs:

```text
JIRA Ticket
Environment
Version
```

Action:

```text
JIRA API
   |
   ├── Ticket exists?
   ├── Approved?
   ├── Correct project?
   ├── Correct component?
   └── Deployment window?
```

Output:

```text
validation-result
```

---

# Production JIRA Gate

```text
Production Deployment
        |
        ↓
JIRA Validation Action
        |
        ├── Ticket
        ├── Approval
        ├── Change Request
        ├── Deployment Window
        └── Version
        |
        ↓
PASS?
   /      \
 NO        YES
 |          |
Stop      Deploy
```

This is a strong example of a custom JavaScript Action.

---

# Change Request Validation

A custom Action can validate:

```text
JIRA ticket
Project
Component
Version / SHA
Approvals
Scan results
Testing results
Deployment window
Rollback plan
```

Example:

```text
CR
 |
 ├── Approved?
 ├── Correct version?
 ├── Testing complete?
 ├── Security checks passed?
 └── Deployment window valid?
```

---

# Production Deployment Gate

A realistic workflow:

```text
UAT Deployment
      |
      ↓
E2E Tests
      |
      ↓
Success?
      |
     YES
      |
      ↓
Production Gate
      |
      ↓
JIRA Validation
      |
      ↓
Commit Validation
      |
      ↓
Approval Validation
      |
      ↓
Deployment Window
      |
      ↓
Production Deployment
```

---

# Commit SHA Validation

A custom Action can validate that the version being deployed matches the approved SHA.

Conceptually:

```text
Approved SHA
     |
     ↓
Actual Artifact SHA
     |
     ↓
Compare
     |
  ┌──┴──┐
 SAME  DIFFERENT
  |        |
 PASS      FAIL
```

This improves deployment traceability.

---

# Artifact Traceability

Production pipeline:

```text
Git Commit
    |
    ↓
Build
    |
    ↓
Artifact
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
Deployment
```

A custom Action can verify metadata at important stages.

---

# Custom GitOps Action

A GitOps Action can:

```text
Update image tag
Validate manifest
Commit
Push
```

Flow:

```text
Build
  |
  ↓
ECR
  |
  ↓
GitOps Action
  |
  ├── Update SHA
  ├── Commit
  └── Push
  |
  ↓
ArgoCD
  |
  ↓
EKS
```

---

# GitOps Security

The GitOps update job may need:

```yaml
permissions:
  contents: write
```

Do not give this permission to unrelated jobs.

Use job-level permissions where practical.

---

# Custom Release Action

A release Action can standardize:

```text
Version
Tag
Release Notes
GitHub Release
Artifact Metadata
```

Example:

```text
Merge
  |
  ↓
Release Action
  |
  ├── Determine version
  ├── Create tag
  └── Create release
```

---

# Custom Notification Action

A notification Action can standardize:

```text
Slack
Teams
Email
JIRA
ServiceNow
```

Example:

```text
Deployment
   |
   ↓
Notification Action
   |
   ↓
Team Channel
```

---

# Custom Environment Promotion Action

Example environments:

```text
DEV
 ↓
QA
 ↓
SIT
 ↓
UAT
 ↓
PROD
```

A promotion Action can validate:

```text
Previous deployment
Test result
Approval
Commit SHA
Security status
```

---

# Reusable Platform Workflow

Custom Actions can be combined with reusable workflows.

Example:

```text
Reusable Workflow
        |
        ├── Build Action
        ├── Security Action
        ├── Docker Action
        └── Deployment Action
```

This creates a standardized CI/CD platform.

---

# Custom Action vs Reusable Workflow

### Custom Action

Packages:

```text
Steps / Logic
```

Example:

```yaml
uses: company/platform-actions/security-scan@v1
```

### Reusable Workflow

Packages:

```text
Jobs
Triggers
Permissions
Environments
Deployment Flow
```

Example:

```yaml
uses: company/platform-workflows/service-ci.yml@v1
```

Use them together when appropriate.

---

# Action Versioning

Never make consumers depend on uncontrolled changes.

Example:

```yaml
uses: company/platform-actions/docker-build@v1
```

Then:

```text
v1.0.0
v1.1.0
v1.1.1
```

Breaking changes:

```text
v2.0.0
```

---

# Major Version Strategy

A platform team can maintain:

```text
v1
v2
```

Consumers choose:

```yaml
uses: company/platform-actions/security-scan@v1
```

or:

```yaml
uses: company/platform-actions/security-scan@v2
```

This allows controlled migration.

---

# SHA Pinning

For stronger supply-chain control:

```yaml
uses: company/platform-actions/security-scan@<commit-sha>
```

Benefits:

```text
Immutable Reference
Predictable Code
Supply-Chain Protection
```

Trade-off:

```text
Requires controlled update process
```

---

# Action Release Process

A production Action should follow:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
Review
      |
      ↓
Tests
      |
      ↓
Security Scan
      |
      ↓
Merge
      |
      ↓
Release
      |
      ↓
Version Tag
```

---

# Testing Custom Actions

Test:

```text
Valid Input
Invalid Input
Missing Input
Output
Failure
Permissions
API Failure
Network Failure
Security
Runner Compatibility
```

---

# Unit Testing

Especially useful for JavaScript Actions.

Example:

```text
Input
 ↓
Function
 ↓
Expected Output
```

Test:

```text
Valid JIRA response
Invalid JIRA response
Missing ticket
API failure
Approval failure
```

---

# Integration Testing

Run the Action inside GitHub Actions.

Example:

```text
Pull Request
      |
      ↓
Test Workflow
      |
      ↓
Custom Action
      |
      ↓
GitHub / JIRA / Cloud
```

This catches issues that unit tests may not detect.

---

# Action CI Pipeline

A custom JavaScript Action repository can have:

```yaml
name: Action CI

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '22'
          cache: npm

      - name: Install Dependencies
        run: npm ci

      - name: Test
        run: npm test

      - name: Build
        run: npm run build

      - name: Verify Distribution
        run: git diff --exit-code -- dist/
```

Use a Node version compatible with the Action runtime and project requirements.

---

# Security Pipeline

A production Action repository should include:

```text
Lint
 ↓
Unit Tests
 ↓
Dependency Scan
 ↓
Build
 ↓
Integration Tests
 ↓
Action Test
 ↓
Review
 ↓
Release
```

---

# Dependency Management

For JavaScript Actions:

```text
package.json
package-lock.json
```

Keep dependencies controlled.

Process:

```text
Dependency Update
       |
       ↓
Tests
       |
       ↓
Security Scan
       |
       ↓
Review
       |
       ↓
Release
```

---

# Custom Action Supply Chain

```text
Source Code
     |
     ↓
Dependencies
     |
     ↓
Build
     |
     ↓
Distribution
     |
     ↓
Release
     |
     ↓
Consumer Workflows
```

Every stage should be controlled.

---

# Security Review

Before approving a custom Action:

```text
Source Code
Dependencies
Permissions
Secrets
Network
Inputs
Outputs
Runner
External APIs
```

Review all of them.

---

# Least Privilege

Example:

```yaml
permissions:
  contents: read
```

For GitOps update:

```yaml
permissions:
  contents: write
```

For cloud OIDC:

```yaml
permissions:
  contents: read
  id-token: write
```

Only grant the permissions actually required.

---

# Secret Management

Never hardcode:

```text
AWS keys
JIRA passwords
API tokens
GitHub tokens
Database passwords
```

Use:

```text
GitHub Secrets
OIDC
Environment Secrets
External Secret Managers
```

where appropriate.

---

# OIDC

For cloud deployments:

```text
GitHub Actions
      |
      ↓
OIDC
      |
      ↓
AWS IAM
      |
      ↓
Temporary Credentials
```

This avoids long-lived cloud credentials in repository configuration.

---

# Custom Action and AWS

Example:

```text
Custom Action
      |
      ↓
AWS Authentication
      |
      ↓
IAM Role
      |
      ↓
AWS Resource
```

Use narrowly scoped IAM permissions.

---

# Custom Action and ECR

Example:

```text
Docker Build
      |
      ↓
Security Scan
      |
      ↓
AWS OIDC
      |
      ↓
ECR Login
      |
      ↓
Push Image
```

---

# Custom Action and EKS

Direct deployment:

```text
Custom Action
      |
      ↓
AWS / Kubernetes
      |
      ↓
EKS
```

GitOps deployment:

```text
Custom Action
      |
      ↓
Git Manifest
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

The GitOps model can reduce direct cluster permissions in CI.

---

# Self-Hosted Runner Security

If a custom Action executes on a self-hosted runner:

```text
Custom Action
      |
      ↓
Self-Hosted Runner
      |
      ↓
Private Infrastructure
```

The Action must be trusted.

Use:

```text
Runner Groups
Labels
Ephemeral Runners
Network Controls
Least Privilege
Action Review
```

---

# ARC

With Actions Runner Controller:

```text
GitHub
   |
   ↓
ARC
   |
   ↓
Ephemeral Runner
   |
   ↓
Custom Action
   |
   ↓
Build / Scan / Deploy
```

This can improve runner isolation and lifecycle management.

---

# Production Runner Separation

Use separate runners where necessary:

```text
CI Runners
    |
    ├── Build
    └── Test

Security Runners
    |
    └── Scanning

Production Runners
    |
    └── Deployment
```

Do not unnecessarily give CI workloads access to production infrastructure.

---

# Custom Action for DevSecOps

A standardized pipeline:

```text
Checkout
   |
   ↓
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
Docker Build
   |
   ↓
Container Scan
   |
   ↓
ECR
```

Custom Actions can package repeated portions of this pipeline.

---

# Microservices Platform

For services:

```text
User
Product
Cart
Orders
Payment
Inventory
Notification
```

Standardize:

```text
Build
Test
Security
Docker
Push
Deployment
```

through reusable platform Actions.

---

# Example Microservice Workflow

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    permissions:
      contents: read

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Standard Build
        uses: company/platform-actions/java-build@v1
        with:
          java-version: '21'

      - name: Security Scan
        uses: company/platform-actions/security-scan@v1

      - name: Docker Build
        uses: company/platform-actions/docker-build@v1
        with:
          image-name: catalogue
          image-tag: ${{ github.sha }}
```

---

# Production Promotion

A production workflow can use:

```text
QA
 |
 ↓
SIT
 |
 ↓
UAT
 |
 ↓
E2E Tests
 |
 ↓
Production Gate
 |
 ↓
Production
```

Custom validation Actions can enforce promotion requirements.

---

# Production Gate Example

```text
Input
 ├── JIRA Ticket
 └── SHA

      |
      ↓

JIRA Validation
      |
      ↓
Commit Validation
      |
      ↓
Approval Validation
      |
      ↓
Deployment Window
      |
      ↓
Rollback Plan
      |
      ↓
Production Deploy
```

---

# Rollback

Production deployments should have a rollback strategy.

For Helm-based deployments:

```text
Deployment
    |
    ↓
Failure
    |
    ↓
Helm Rollback
```

A custom Action can validate that rollback information exists before deployment.

---

# Deployment Safety

A custom deployment Action should not blindly deploy.

It should verify:

```text
Correct environment
Correct version
Approved change
Required tests
Security checks
Deployment window
Rollback plan
```

---

# Action Documentation

Every production Action should document:

```text
Name
Purpose
Inputs
Outputs
Example
Required Permissions
Secrets
Supported Runners
Dependencies
Failure Conditions
Version
Owner
Security Considerations
```

---

# Example Documentation

```markdown
# JIRA Deployment Validation

## Purpose

Validate production change requirements.

## Inputs

- jira-ticket
- version
- environment

## Outputs

- validation-result

## Required Permissions

contents: read

## Secrets

JIRA_API_TOKEN

## Example

uses: company/platform-actions/jira-validation@v1
```

---

# Action Ownership

Every shared Action should have an owner.

Example:

```text
docker-build
    |
    └── Platform Engineering

security-scan
    |
    └── DevSecOps

jira-validation
    |
    └── Release Engineering

terraform-validation
    |
    └── Cloud Platform
```

---

# Action Governance

Enterprise governance can define:

```text
Approved Actions
Version Policy
Security Review
Ownership
Release Process
Dependency Policy
Permission Policy
Update Process
Deprecation Process
```

---

# Action Inventory

Maintain an inventory:

| Action | Purpose | Owner | Version | Security |
|---|---|---|---|---|
| java-build | Java build | Platform | v1 | Approved |
| docker-build | Docker build | Platform | v1 | Approved |
| security-scan | Security | DevSecOps | v1 | Approved |
| jira-validation | CR validation | Release | v1 | Approved |

---

# Action Deprecation

When an Action is no longer supported:

```text
Active
  |
  ↓
Deprecated
  |
  ↓
Migration Period
  |
  ↓
Retired
```

Consumers should be informed before removal.

---

# Breaking Changes

Example:

```text
v1
```

expects:

```text
environment
```

while:

```text
v2
```

expects:

```text
target-environment
```

Do not silently change v1.

Release:

```text
v2
```

and provide migration instructions.

---

# Action Update Strategy

```text
New Version
    |
    ↓
Test
    |
    ↓
Security Review
    |
    ↓
Canary Repository
    |
    ↓
Selected Applications
    |
    ↓
All Applications
```

This reduces risk for organization-wide updates.

---

# Canary Strategy

Instead of immediately upgrading:

```text
200 repositories
```

test first:

```text
Platform Action
      |
      ↓
5 repositories
      |
      ↓
25 repositories
      |
      ↓
100 repositories
      |
      ↓
200 repositories
```

This is useful for high-impact shared Actions.

---

# Monitoring Shared Actions

Track:

```text
Failure Rate
Execution Time
Version Adoption
Security Alerts
Dependency Status
Consumer Errors
```

A shared Action is a platform component and should be operated accordingly.

---

# Custom Action Incident

Suppose a custom Action is compromised.

Response:

```text
1. Identify affected Action version
2. Identify consuming repositories
3. Stop affected deployments
4. Revoke exposed credentials if necessary
5. Release fixed version
6. Update consumers
7. Review logs
8. Investigate root cause
9. Add preventive controls
```

---

# Supply Chain Incident

```text
Compromised Dependency
        |
        ↓
Custom Action
        |
        ↓
Many Repositories
        |
        ↓
Runner
        |
        ↓
Secrets / Infrastructure
```

Minimize blast radius through:

```text
Least Privilege
SHA Pinning
Short-Lived Credentials
Runner Isolation
Security Scanning
Version Control
```

---

# Action Security Checklist

```text
☐ Source reviewed
☐ Dependencies reviewed
☐ Inputs validated
☐ Secrets protected
☐ Permissions minimized
☐ API access restricted
☐ No secret logging
☐ Version controlled
☐ Security scanning
☐ Tests available
☐ Owner assigned
☐ Documentation available
☐ Release process defined
```

---

# Custom Action Development Lifecycle

```text
Requirement
    |
    ↓
Choose Action Type
    |
    ↓
Design Inputs / Outputs
    |
    ↓
Implement
    |
    ↓
Unit Test
    |
    ↓
Security Review
    |
    ↓
Integration Test
    |
    ↓
Release
    |
    ↓
Version
    |
    ↓
Consumers
    |
    ↓
Monitor
    |
    ↓
Update / Retire
```

---

# Production-Level Architecture

```text
                    Platform Team
                         |
                         ↓
                Custom Action Registry
                         |
        ┌────────────────┼─────────────────┐
        ↓                ↓                 ↓
      Build           Security          Governance
        |                |                 |
        ↓                ↓                 ↓
      Docker           Trivy           JIRA Gate
      Maven            SonarQube        Approval
      Node             Veracode         Window
        |                |                 |
        └────────────────┼─────────────────┘
                         ↓
                 Application Workflows
                         |
              ┌──────────┼──────────┐
              ↓          ↓          ↓
            User      Catalogue     Cart
              |
              ↓
             ECR
              |
              ↓
          GitOps Update
              |
              ↓
            ArgoCD
              |
              ↓
             EKS
```

---

# Recommended Platform Action Set

For a DevOps / DevSecOps platform:

```text
01-java-build
02-node-build
03-python-build
04-docker-build
05-security-scan
06-terraform-validation
07-helm-validation
08-jira-validation
09-deployment-gate
10-gitops-update
11-release
12-notification
```

Keep each Action focused.

---

# Do Not Create One Giant Action

Avoid:

```text
platform-action
 |
 ├── Build
 ├── Test
 ├── SonarQube
 ├── Trivy
 ├── Veracode
 ├── Docker
 ├── Terraform
 ├── Helm
 ├── Kubernetes
 ├── JIRA
 ├── Release
 └── Notification
```

This creates:

```text
High Coupling
Hard Debugging
Difficult Versioning
Large Blast Radius
```

Prefer smaller Actions.

---

# Good Platform Design

```text
java-build
     |
     ↓
security-scan
     |
     ↓
docker-build
     |
     ↓
ecr-publish
     |
     ↓
gitops-update
     |
     ↓
ArgoCD
```

Each Action has a clear responsibility.

---

# Custom Action vs Script

A script:

```text
build.sh
```

is useful for local or workflow-specific logic.

A custom Action:

```text
company/platform-actions/build
```

is useful when the same logic should be:

```text
Versioned
Documented
Tested
Governed
Reused
```

across repositories.

---

# Custom Action vs Reusable Workflow

Use a custom Action for:

```text
Reusable operation
Reusable logic
Reusable step
```

Use a reusable workflow for:

```text
Complete CI/CD process
Multiple jobs
Environment
Approvals
Deployment flow
Permissions
```

---

# Combined Architecture

```text
Reusable Workflow
       |
       ├── Build Action
       ├── Security Action
       ├── Docker Action
       ├── JIRA Action
       └── Deployment Action
```

This is a strong enterprise pattern.

---

# Production Example

```yaml
name: Production Deployment

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:

  validate:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Validate Change Request
        uses: company/platform-actions/jira-validation@v1
        with:
          jira-ticket: ${{ inputs.jira-ticket }}
          version: ${{ github.sha }}

  deploy:

    needs: validate

    environment:
      name: production

    permissions:
      contents: read
      id-token: write

    runs-on:
      - self-hosted
      - linux
      - production

    steps:

      - name: Production Deployment
        uses: company/platform-actions/helm-deploy@v1
        with:
          version: ${{ github.sha }}
```

This demonstrates:

```text
Manual trigger
+
Validation
+
Job dependency
+
Production environment
+
OIDC
+
Self-hosted runner
+
Custom Actions
```

---

# Production Workflow with UAT

```text
Build
 |
 ↓
QA
 |
 ↓
SIT
 |
 ↓
UAT
 |
 ↓
E2E Tests
 |
 ↓
Production Gate
 |
 ├── JIRA
 ├── CR
 ├── Approvals
 ├── SHA
 ├── Scan
 └── Deployment Window
 |
 ↓
Production
```

Custom Actions can implement individual validation responsibilities.

---

# Best Practices

- Choose the simplest suitable Action type.
- Keep Actions small and focused.
- Define clear inputs and outputs.
- Validate inputs.
- Avoid hardcoded credentials.
- Use least-privilege permissions.
- Prefer OIDC for supported cloud authentication.
- Version Actions.
- Consider SHA pinning.
- Test Actions independently and inside workflows.
- Scan dependencies.
- Document requirements.
- Assign ownership.
- Use controlled release processes.
- Use canary upgrades for high-impact Actions.
- Monitor shared Action health.
- Maintain an Action inventory.
- Define a deprecation process.
- Keep production deployment Actions auditable.
- Separate validation from deployment where practical.

---

# Interview Questions

## Basic

1. What is a custom GitHub Action?
2. Why would you create a custom Action?
3. What are the different types of custom Actions?
4. How do you choose between Composite, Docker, and JavaScript Actions?
5. What is `action.yml`?
6. What are Action inputs?
7. What are Action outputs?
8. How do you version a custom Action?
9. How do you call a custom Action?
10. What should be documented for a custom Action?

## Intermediate

11. How would you create a reusable Docker build Action?
12. How would you create a Terraform validation Action?
13. How would you create a security scanning Action?
14. How would you create a JIRA validation Action?
15. How would you handle input validation?
16. How would you expose outputs from a custom Action?
17. How would you test a custom Action?
18. What is the difference between a custom Action and a reusable workflow?
19. How would you organize multiple Actions in an enterprise?
20. How would you manage Action dependencies?
21. How would you handle Action version upgrades?
22. Why should custom Actions have owners?

## Advanced / Production

23. Design a centralized enterprise GitHub Actions platform.
24. How would you standardize CI/CD across multiple microservices using custom Actions?
25. Design a production Docker build + security scanning Action.
26. Design a JIRA change-request validation Action for production deployment.
27. How would you validate deployment windows using a custom Action?
28. How would you verify that the deployed SHA matches the approved version?
29. How would you integrate custom Actions with AWS OIDC and IAM?
30. How would you integrate custom Actions with ECR?
31. How would you integrate custom Actions with EKS?
32. How would you integrate custom Actions with GitOps and ArgoCD?
33. How would you secure custom Actions running on self-hosted runners?
34. How would you secure custom Actions running on ARC?
35. How would you design least-privilege permissions for a custom Action?
36. How would you handle secrets in custom Actions?
37. How would you protect custom Actions from supply-chain attacks?
38. How would you respond if a shared Action used by hundreds of repositories were compromised?
39. How would you roll out a breaking Action update across hundreds of repositories?
40. How would you implement canary releases for platform Actions?
41. How would you design a CI pipeline for the Action repository itself?
42. How would you monitor the health and failure rate of shared Actions?
43. How would you design an Action inventory and governance process?
44. How would you separate CI, security, and production deployment Actions?
45. How would you design a production workflow using custom Actions, JIRA approval, ECR, GitOps, ArgoCD, and EKS?
46. When would you choose a custom Action over a reusable workflow?
47. How would you prevent a single giant custom Action from becoming a platform bottleneck?
48. How would you handle deprecation of an enterprise-wide custom Action?
49. How would you secure third-party Actions used inside your custom Actions?
50. Explain how you would build a production-grade custom GitHub Actions platform from development through release, governance, and retirement.