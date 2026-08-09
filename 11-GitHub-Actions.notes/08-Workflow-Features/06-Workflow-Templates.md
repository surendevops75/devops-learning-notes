# GitHub Actions Workflow Templates

Workflow templates are predefined GitHub Actions workflow files that help teams quickly create standardized workflows for repositories.

They are useful when an organization wants developers to start with an approved CI/CD structure instead of creating workflows from scratch.

---

# Why Workflow Templates Matter

Without templates:

```text
Developer
   |
   ↓
Create workflow from scratch
   |
   ├── Choose actions
   ├── Configure permissions
   ├── Configure runners
   ├── Configure security
   └── Configure build/test
```

Different developers may create completely different workflows.

With templates:

```text
Platform Team
      |
      ↓
Approved Workflow Templates
      |
      ├── Node.js CI
      ├── Java CI
      ├── Docker Build
      ├── Terraform
      └── Security Scan
             |
             ↓
        Application Teams
```

---

# Workflow Template vs Reusable Workflow

These are different.

## Workflow Template

Helps create a workflow in a repository.

```text
Template
   ↓
New Repository
   ↓
Workflow file
```

The resulting workflow becomes part of the repository.

## Reusable Workflow

Runs centrally when called.

```text
Application Workflow
       |
       ↓
Reusable Workflow
       |
       ↓
Jobs
```

---

# Simple Difference

Remember:

```text
Template
→ Creates workflow structure

Reusable Workflow
→ Reuses workflow execution
```

---

# Workflow Template Location

Organization-level workflow templates are commonly maintained in a special repository:

```text
.github
```

A common structure is:

```text
.github/
│
└── workflow-templates/
    ├── nodejs-ci.yml
    ├── java-ci.yml
    ├── terraform.yml
    └── docker-build.yml
```

Template metadata can also be provided using:

```text
.properties.json
```

files.

---

# Basic Workflow Template

Example:

```yaml
name: Node.js CI

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

permissions:
  contents: read

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: npm

      - name: Install
        run: npm ci

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

This can serve as a starting point for Node.js repositories.

---

# Template Metadata

A workflow template can have metadata describing the template.

Example:

```text
nodejs-ci.properties.json
```

Conceptually:

```json
{
  "name": "Node.js CI",
  "description": "Standard Node.js CI workflow",
  "iconName": "nodejs",
  "categories": [
    "Node.js",
    "Continuous integration"
  ]
}
```

The metadata helps GitHub present the template appropriately when users choose workflows.

---

# Why Metadata Matters

Metadata can communicate:

```text
Template Name
Description
Icon
Categories
```

This makes it easier for developers to identify the correct template.

---

# Organization-Level Templates

An organization can provide standardized templates to its repositories.

Example:

```text
Organization
      |
      ↓
.github Repository
      |
      ↓
workflow-templates
      |
 ┌────┼─────────────┐
 ↓    ↓             ↓
Node Java        Terraform
 ↓    ↓             ↓
Repositories
```

---

# Platform Team Model

A platform team can maintain:

```text
Node.js Template
Java Template
Python Template
Terraform Template
Docker Template
Security Template
```

Application teams can start with the appropriate template.

---

# Why Organizations Use Templates

Templates help standardize:

```text
CI Configuration
Permissions
Approved Actions
Build Tools
Testing
Security Checks
Naming
Basic Pipeline Structure
```

---

# Template Governance

For enterprise environments:

```text
Platform Team
      |
      ↓
Workflow Templates
      |
      ↓
Application Teams
```

The platform team can define a baseline.

For example:

```text
Every new service
   ↓
Uses standard CI template
   ↓
Tests
   ↓
Security
   ↓
Build
```

---

# Security Baseline

A template can establish secure defaults.

Example:

```yaml
permissions:
  contents: read
```

This is better than leaving permissions unnecessarily broad.

---

# Production Template Principle

Start with:

```yaml
permissions:
  contents: read
```

Then add only the permissions actually required.

For AWS OIDC workflows:

```yaml
permissions:
  contents: read
  id-token: write
```

Only use `id-token: write` when the workflow actually requires OIDC authentication.

---

# Template for Docker CI

Example:

```yaml
name: Docker CI

on:
  push:
  pull_request:

permissions:
  contents: read

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Docker Build
        run: |
          docker build -t application:${GITHUB_SHA} .
```

This gives developers a basic container CI workflow.

---

# Template for Java

Example:

```yaml
name: Java CI

on:
  push:
  pull_request:

permissions:
  contents: read

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'
          cache: maven

      - name: Build
        run: |
          mvn -B verify
```

---

# Template for Terraform

Example:

```yaml
name: Terraform CI

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:

  terraform:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3

      - name: Terraform Init
        run: |
          terraform init

      - name: Terraform Format
        run: |
          terraform fmt -check

      - name: Terraform Validate
        run: |
          terraform validate

      - name: Terraform Plan
        run: |
          terraform plan
```

---

# Template for DevSecOps

A stronger enterprise template can include:

```text
Checkout
   ↓
Dependency Installation
   ↓
Unit Tests
   ↓
SonarQube
   ↓
Build
   ↓
Docker
   ↓
Trivy
   ↓
Veracode
   ↓
Artifact / ECR
```

The exact security controls should match organizational policy.

---

# DevSecOps Template

Conceptual structure:

```yaml
jobs:

  test:
    ...

  security:
    needs: test
    ...

  build:
    needs: security
    ...

  container-scan:
    needs: build
    ...

  publish:
    needs: container-scan
    ...
```

This gives new repositories a consistent baseline.

---

# Template and Reusable Workflow Together

These two features can work together.

Example:

```text
Workflow Template
       |
       ↓
Application Repository
       |
       ↓
Calls Reusable Workflow
       |
       ↓
Central CI/CD Logic
```

This is a powerful enterprise pattern.

---

# Example Architecture

```text
Organization
│
├── Workflow Templates
│      |
│      └── application-ci.yml
│
└── Platform Workflows
       |
       ├── reusable-build.yml
       ├── reusable-security.yml
       ├── reusable-docker.yml
       └── reusable-deploy.yml
```

Application repository:

```text
catalogue
│
└── .github/
    └── workflows/
        └── ci.yml
```

The template creates the initial:

```text
ci.yml
```

which then calls centralized reusable workflows.

---

# Template + Reusable Workflow Flow

```text
New Repository
      |
      ↓
Select Template
      |
      ↓
Workflow Created
      |
      ↓
Calls Reusable Workflow
      |
      ↓
Centralized Build/Security/Deploy
```

---

# Why Combine Them?

Templates solve:

```text
How do we get started?
```

Reusable workflows solve:

```text
How do we centralize execution?
```

Together:

```text
Template
+
Reusable Workflow
=
Standardized + Centralized CI/CD
```

---

# Template and Microservices

For a microservices organization:

```text
New Service
   |
   ↓
Select Standard Template
   |
   ↓
Node/Java CI
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
```

This avoids every team designing CI/CD independently.

---

# Template for Node.js Microservice

Example:

```text
New Service
   |
   ↓
Node.js Template
   |
   ↓
Checkout
   ↓
Node Setup
   ↓
npm Cache
   ↓
npm ci
   ↓
Tests
   ↓
Build
```

The repository can then evolve its workflow as needed.

---

# Template for Java Microservice

```text
New Service
   |
   ↓
Java Template
   |
   ↓
Checkout
   ↓
JDK Setup
   ↓
Maven Cache
   ↓
mvn verify
```

---

# Template for Infrastructure Repository

```text
Infrastructure Repository
        |
        ↓
Terraform Template
        |
        ↓
fmt
validate
security
plan
```

Production apply can be handled separately through protected environments and approval workflows.

---

# Template for Containerized Applications

```text
Application
    |
    ↓
Docker Template
    |
    ├── Build
    ├── Test
    ├── Scan
    └── Push
```

Production image publishing should use the organization's approved registry and immutable release process.

---

# Template and GitOps

A template can create a workflow that updates a GitOps repository.

Example:

```text
Application Repository
       |
       ↓
Build
       |
       ↓
ECR
       |
       ↓
Update GitOps
       |
       ↓
ArgoCD
       |
       ↓
EKS
```

The template provides the initial workflow structure.

The actual deployment logic can live in a reusable workflow.

---

# Template and ArgoCD

A standard application template could provide:

```text
Build
Security
Image Push
GitOps Update
```

Then:

```text
ArgoCD
```

handles:

```text
Synchronization
Drift Detection
Deployment
Health
```

---

# Template and Helm

A template can standardize the expected repository structure:

```text
service/
├── src/
├── Dockerfile
├── helm/
│   └── service/
└── .github/
    └── workflows/
```

This gives application teams a predictable starting point.

---

# Template and ECR

A container template can establish:

```text
Image Name
Tagging Convention
Registry
Security Scan
Push Policy
```

For example:

```text
service: catalogue

image:
ECR/catalogue

tag:
Git SHA
```

Prefer immutable identifiers such as commit SHA or digest for release traceability.

---

# Template and Git SHA

A production-friendly template can establish:

```text
Git Commit
    |
    ↓
Docker Image Tag
    |
    ↓
Image Digest
```

Example:

```text
catalogue:8a92f31
```

Then use the digest for stronger immutability.

---

# Template and Security

A standard template should avoid:

```text
Untrusted Actions
Broad Permissions
Hardcoded Credentials
Long-Lived Cloud Keys
Secret Logging
Unsafe Shell Construction
```

Instead:

```text
Approved Actions
Least Privilege
OIDC
Input Validation
Secure Defaults
```

---

# Template Security Governance

The platform team should review:

```text
Action Versions
Permissions
Secrets
Runner Selection
Third-Party Actions
Shell Commands
Cloud Authentication
Artifact Handling
```

before making a template broadly available.

---

# Template and Action Pinning

Third-party Actions should be controlled according to your organization's security policy.

For critical workflows, consider immutable references such as commit SHAs where practical.

Example concept:

```yaml
uses: actions/checkout@<approved-immutable-reference>
```

This reduces the risk of unexpected upstream changes.

---

# Template Update Problem

A key difference from reusable workflows:

Once a template creates a workflow in a repository:

```text
Template
   ↓
Workflow copied into repository
```

future changes to the template do not automatically mean every existing workflow is updated.

This is an important distinction.

---

# Template vs Centralized Workflow

### Template

```text
Create once
↓
Copy into repository
↓
Repository owns resulting workflow
```

### Reusable Workflow

```text
Central workflow
↓
Repositories call it
↓
Central implementation can be updated/versioned
```

---

# Template Maintenance

If the platform team improves a template:

```text
Template v2
```

existing repositories may need to update their workflows manually or through an automated migration process.

This is why templates are best used for:

```text
Initial scaffolding
```

while reusable workflows are often better for:

```text
Centralized ongoing standards
```

---

# Template Versioning

You can communicate versions through naming/documentation:

```text
Node CI Template v1
Node CI Template v2
```

However, remember that once copied into a repository, the resulting workflow is independent of future template changes.

---

# Template Migration

Example:

```text
Old Template
     |
     ↓
Existing Repositories
     |
     ↓
Migration
     |
     ↓
New Standard
```

For large organizations, migration can be automated through pull requests.

---

# Template + Dependabot

If your template creates workflows using Actions:

```text
actions/checkout
actions/setup-node
actions/upload-artifact
```

those references need ongoing maintenance.

Dependabot can help keep GitHub Actions dependencies updated where configured.

---

# Template + Security Updates

If a security issue affects a workflow Action:

```text
Shared Template
```

does not automatically update already-created workflow files.

Existing repositories must be updated.

This is another reason centralized reusable workflows can be valuable.

---

# Template Repository Structure

Example:

```text
.github/
│
└── workflow-templates/
    │
    ├── nodejs-ci.yml
    ├── nodejs-ci.properties.json
    │
    ├── java-ci.yml
    ├── java-ci.properties.json
    │
    ├── terraform.yml
    ├── terraform.properties.json
    │
    ├── docker.yml
    └── docker.properties.json
```

---

# Template Naming

Use clear names:

```text
nodejs-ci.yml
java-ci.yml
terraform-ci.yml
docker-ci.yml
python-ci.yml
```

Avoid:

```text
workflow1.yml
default.yml
test.yml
new.yml
```

Names should communicate the purpose.

---

# Template Categories

Useful categories include:

```text
CI
CD
Security
Infrastructure
Docker
Kubernetes
Language
Testing
```

Metadata can help organize templates for users.

---

# Template Documentation

Every enterprise template should document:

```text
Purpose
Supported Languages
Required Inputs
Expected Repository Structure
Secrets
Permissions
Runner
Security Controls
Deployment Model
Ownership
Version
Support Process
```

---

# Template Ownership

Example:

```text
Platform Engineering
      |
      ↓
Workflow Template
      |
      ↓
Application Teams
```

Clearly identify who owns the template.

---

# Template Change Management

Treat production templates as code.

Use:

```text
Pull Request
Code Review
Testing
Security Review
Versioning
Release Notes
```

before changing a template.

---

# Template Testing

Test templates using representative repositories.

Example:

```text
Template
   |
   ├── Node Test Repository
   ├── Java Test Repository
   └── Terraform Test Repository
```

Verify:

```text
Build
Tests
Security
Artifacts
Permissions
```

---

# Template and Self-Hosted Runners

A template can specify runner labels according to the organization's design.

Example:

```yaml
runs-on: ubuntu-latest
```

or:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
```

Be careful when templates select self-hosted runners.

---

# Self-Hosted Runner Security

Do not send untrusted pull-request code to privileged self-hosted runners without a carefully designed isolation and trust model.

Potential risk:

```text
Untrusted PR
   ↓
Self-hosted Runner
   ↓
Credentials / Network
```

Templates should use safe defaults.

---

# Template and Environment

A deployment template may use:

```yaml
environment:
  name: production
```

or:

```yaml
environment:
  name: ${{ inputs.environment }}
```

Production environments should have appropriate:

```text
Protection Rules
Approvals
Secrets
Deployment Controls
```

---

# Template and Production

A production template should not simply:

```text
Build
 ↓
Deploy Production
```

A stronger workflow is:

```text
Build
 ↓
Test
 ↓
Security
 ↓
UAT
 ↓
Change Validation
 ↓
Approval
 ↓
Production
```

---

# Production Workflow Template

Conceptually:

```text
                    Production Template
                           |
                           ↓
                       Validation
                           |
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
       JIRA/CR          Security          Tests
          |                |                |
          └────────────────┼────────────────┘
                           ↓
                    GitHub Environment
                           |
                           ↓
                       Approval
                           |
                           ↓
                       Deployment
                           |
                           ↓
                       Verification
                           |
                     ┌─────┴─────┐
                     ↓           ↓
                   PASS        FAIL
                     |           |
                     ↓           ↓
                 Complete    Rollback
```

---

# Template and JIRA

A production template can establish the expected input:

```text
JIRA Ticket
```

Then the workflow can validate:

```text
Ticket Exists
Status Approved
Deployment Window
Project
Component
Version
```

The exact API implementation should live in a reusable workflow or script rather than duplicating complex logic across repositories.

---

# Template and Change Request

A production workflow can require:

```text
JIRA Ticket
Change Request
Approvals
Deployment Window
Commit SHA
Security Results
Testing Results
Rollback Plan
```

This aligns with a controlled production release process.

---

# Template and Rollback

Template can establish that deployments must include:

```text
Health Verification
Rollback Strategy
Diagnostics
```

For Helm:

```bash
helm upgrade --install \
  --wait \
  --timeout 15m \
  --atomic
```

For GitOps:

```text
Revert GitOps Change
       ↓
ArgoCD
       ↓
EKS
```

The exact rollback mechanism depends on the deployment architecture.

---

# Template and Artifacts

A production template can standardize:

```text
Test Reports
Security Reports
Deployment Evidence
Diagnostics
```

Example:

```yaml
- name: Upload Test Results
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: reports/
```

---

# Template and Caching

A standard template can enable supported package-manager caching.

Node:

```yaml
cache: npm
```

Java:

```yaml
cache: maven
```

This provides a consistent baseline.

---

# Template and Timeout

A template can define a reasonable job timeout:

```yaml
timeout-minutes: 20
```

Production deployment templates can use a larger timeout based on actual deployment duration.

---

# Template and Concurrency

Deployment templates should consider concurrency.

Example:

```yaml
concurrency:
  group: production-deploy
  cancel-in-progress: false
```

For microservices:

```yaml
concurrency:
  group: deploy-${{ inputs.service }}-${{ inputs.environment }}
  cancel-in-progress: false
```

---

# Template and Permissions

A secure default:

```yaml
permissions:
  contents: read
```

Production AWS deployment:

```yaml
permissions:
  contents: read
  id-token: write
```

Only grant additional permissions when required.

---

# Template and Secrets

Avoid:

```yaml
secrets: inherit
```

unless there is a clear reason.

Prefer explicit secret passing when possible.

Example:

```yaml
secrets:
  deployment-token: ${{ secrets.DEPLOYMENT_TOKEN }}
```

---

# Template and OIDC

For AWS:

```text
GitHub Actions
      |
      ↓
OIDC Token
      |
      ↓
AWS IAM Role
      |
      ↓
EKS / ECR / AWS
```

Templates can standardize this pattern.

---

# Template and AWS Trust Policy

The AWS IAM trust relationship should restrict:

```text
Organization
Repository
Branch / Environment
```

according to the security model.

Do not give every repository unrestricted access to the same production role.

---

# Template and Multi-Environment

A standard template can support:

```text
QA
SIT
UAT
Production
```

Example flow:

```text
Commit
  ↓
Build
  ↓
Security
  ↓
QA
  ↓
SIT
  ↓
UAT
  ↓
Production
```

Production should have stronger controls.

---

# Environment Promotion

Example:

```text
QA
 |
 ↓
UAT
 |
 ↓
Production
```

At each stage:

```text
Validation
Testing
Approval
```

can be applied according to policy.

---

# Template and Inputs

A workflow template itself can contain normal workflow inputs only when the resulting workflow is designed accordingly.

For centrally reusable parameters, reusable workflows are usually a better abstraction.

Example:

```text
Template
   ↓
Creates caller workflow

Caller
   ↓
Passes inputs

Reusable Workflow
   ↓
Executes centralized logic
```

---

# Enterprise Pattern

```text
                Organization
                     |
          ┌──────────┴──────────┐
          ↓                     ↓
   Workflow Templates     Reusable Workflows
          |                     |
          ↓                     ↓
 Initial Repository       Central CI/CD Logic
          |                     |
          └──────────┬──────────┘
                     ↓
             Application Teams
                     |
                     ↓
             Production Platform
```

---

# Recommended Use

Use workflow templates for:

```text
New Repository Scaffolding
Standard Starting Workflows
Developer Onboarding
Language-Specific CI
Basic Repository Automation
```

Use reusable workflows for:

```text
Centralized Security
Production Deployment
Cloud Authentication
Terraform Standards
Docker Standards
Organizational CI/CD Policies
```

---

# Key Difference

Remember this interview answer:

```text
Workflow Template:
Provides a starting workflow that can be added to a repository.

Reusable Workflow:
Provides centralized workflow logic that another workflow calls.
```

---

# Production-Level Design

For your DevSecOps environment:

```text
New Microservice
      |
      ↓
Select Node/Java Template
      |
      ↓
Initial CI Workflow
      |
      ↓
Reusable Build
      |
      ↓
Reusable Security
      |
      ├── SonarQube
      ├── Trivy
      └── Veracode
      |
      ↓
Docker Build
      |
      ↓
ECR
      |
      ↓
UAT
      |
      ↓
JIRA / CR Validation
      |
      ↓
GitHub Environment Approval
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

This combines:

```text
Templates
+
Reusable Workflows
+
DevSecOps
+
GitOps
+
Production Governance
```

---

# Best Practices

```text
☐ Keep templates simple
☐ Use clear names
☐ Provide metadata
☐ Use secure default permissions
☐ Avoid hardcoded credentials
☐ Prefer OIDC for AWS
☐ Use approved Actions
☐ Document ownership
☐ Test templates
☐ Review template changes
☐ Version reusable workflows
☐ Use templates for scaffolding
☐ Use reusable workflows for central logic
☐ Validate production inputs
☐ Protect production environments
☐ Use concurrency for deployments
☐ Configure realistic timeouts
☐ Preserve security/test evidence
☐ Avoid privileged self-hosted runners for untrusted code
```

---

# Common Mistakes

### 1. Thinking templates automatically update existing workflows

They do not work like centralized reusable workflows.

### 2. Using templates for all centralized logic

Reusable workflows are usually better for ongoing centralized behavior.

### 3. Giving templates excessive permissions

Start with least privilege.

### 4. Hardcoding secrets

Never do this.

### 5. Using long-lived AWS credentials

Prefer OIDC.

### 6. Using untrusted code on privileged runners

This can create serious security risks.

### 7. No ownership

Someone must maintain the template.

### 8. No testing

A broken template can affect many new repositories.

### 9. No documentation

Developers need to understand what the template provides.

### 10. No migration strategy

Templates evolve, but existing repositories need explicit updates.

---

# Key Takeaways

Workflow templates provide a standardized starting point.

Typical location:

```text
.github/workflow-templates/
```

A template may include:

```text
Workflow YAML
Metadata
```

The most important distinction is:

```text
Workflow Template
→ Initial scaffolding

Reusable Workflow
→ Centralized execution
```

A strong enterprise pattern is:

```text
Template
   ↓
Application Workflow
   ↓
Reusable Workflow
   ↓
Centralized CI/CD
```

For your DevSecOps environment:

```text
Template
 ↓
Build
 ↓
SonarQube
 ↓
Trivy
 ↓
Veracode
 ↓
Docker
 ↓
ECR
 ↓
JIRA / CR
 ↓
Approval
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

Templates make it easier to start correctly; reusable workflows make it easier to remain consistent.

---

# Interview Questions

## Basic

1. What is a GitHub Actions workflow template?
2. Why are workflow templates used?
3. Where are organization workflow templates stored?
4. What is a `.properties.json` file used for?
5. What is the difference between a workflow template and a reusable workflow?
6. What is the difference between a workflow template and a composite Action?
7. Why should templates have clear names?
8. What information can template metadata provide?
9. How do workflow templates help standardize CI/CD?
10. Who should own enterprise workflow templates?

## Intermediate

11. How would you create a Node.js workflow template?
12. How would you create a Java/Maven workflow template?
13. How would you create a Terraform workflow template?
14. How would you create a Docker workflow template?
15. How would you include security scanning in a template?
16. How would you configure least-privilege permissions in a template?
17. How would you configure npm/Maven caching in templates?
18. How would you include artifacts in a template?
19. How would you configure timeout and concurrency in a deployment template?
20. How would you design a template for multiple environments?
21. How would you use a workflow template together with reusable workflows?
22. Why are reusable workflows better for centralized ongoing standards?
23. How would you standardize CI/CD across multiple microservices?
24. How would you handle template updates?
25. How would you test a workflow template before making it available to developers?

## Advanced / Production

26. Design an enterprise workflow-template architecture for 100+ microservices.
27. How would you combine workflow templates with reusable workflows?
28. How would you centralize SonarQube, Trivy, and Veracode using templates and reusable workflows?
29. How would you design a secure AWS OIDC template?
30. How would you prevent a template from granting excessive permissions?
31. How would you protect templates from untrusted pull requests?
32. How would you handle self-hosted runners in organization templates?
33. How would you prevent a compromised template from spreading insecure practices?
34. How would you version and govern workflow templates?
35. How would you test a template against Node.js, Java, and Terraform repositories?
36. How would you design a production deployment template with JIRA change validation?
37. How would you integrate GitHub Environments and approval gates into a production template?
38. How would you integrate Helm, ArgoCD, and EKS into a standard deployment workflow?
39. How would you combine templates, reusable workflows, ECR, and GitOps?
40. How would you design a template that supports QA, SIT, UAT, and Production?
41. How would you prevent long-lived AWS credentials from being introduced into templates?
42. How would you design artifact, cache, timeout, and concurrency standards in templates?
43. How would you handle a critical security update to an Action used by an existing template?
44. Why might a reusable workflow be preferred over a workflow template for production deployment logic?
45. Design an enterprise-grade GitHub Actions platform using workflow templates for repository scaffolding and reusable workflows for Build, SonarQube, Trivy, Veracode, Docker, ECR, Terraform, JIRA/CR validation, GitHub Environments, OIDC, Helm, ArgoCD, EKS, rollback, security governance, and production deployment.