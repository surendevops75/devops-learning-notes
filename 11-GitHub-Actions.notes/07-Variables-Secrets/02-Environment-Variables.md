# GitHub Actions Environment Variables

Environment variables are key-value pairs made available to processes running inside a GitHub Actions workflow.

They are commonly used for:

```text
Application Configuration
Build Configuration
Deployment Configuration
Tool Configuration
Environment Selection
Version Information
```

Example:

```yaml
env:
  APP_NAME: catalogue
  ENVIRONMENT: qa
```

Then:

```yaml
run: |
  echo "Application: $APP_NAME"
  echo "Environment: $ENVIRONMENT"
```

---

# Environment Variable Flow

```text
Workflow
   |
   ↓
Environment Variable
   |
   ↓
Job
   |
   ↓
Step
   |
   ↓
Process / Command
```

Example:

```text
APP_NAME=catalogue
        |
        ↓
Docker Build
        |
        ↓
catalogue:<commit-sha>
```

---

# Defining Environment Variables

Environment variables are defined with:

```yaml
env:
```

Example:

```yaml
env:
  APP_NAME: catalogue
  AWS_REGION: ap-south-1
```

---

# Workflow-Level Environment Variables

A workflow-level `env` variable can be available to jobs and steps in the workflow unless overridden at a more specific scope.

Example:

```yaml
name: CI

on:
  push:

env:
  APP_NAME: catalogue
  AWS_REGION: ap-south-1

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Build
        run: |
          echo "Application: $APP_NAME"
          echo "Region: $AWS_REGION"
```

---

# Job-Level Environment Variables

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    env:
      APP_NAME: catalogue
      BUILD_TYPE: release

    steps:

      - name: Build
        run: |
          echo "Application: $APP_NAME"
          echo "Build type: $BUILD_TYPE"
```

These variables apply to the steps in that job.

---

# Step-Level Environment Variables

Example:

```yaml
steps:

  - name: Build
    env:
      BUILD_TYPE: release
    run: |
      echo "Build type: $BUILD_TYPE"
```

The variable is limited to that step.

---

# Environment Variable Scope

Think of the hierarchy as:

```text
Workflow
   |
   ↓
Job
   |
   ↓
Step
```

The narrower scope can override a broader value.

Example:

```yaml
env:
  ENVIRONMENT: dev

jobs:

  deploy:

    env:
      ENVIRONMENT: qa

    steps:

      - name: Deploy
        env:
          ENVIRONMENT: production
        run: |
          echo "$ENVIRONMENT"
```

Result:

```text
production
```

---

# Environment Variable Precedence

Conceptually:

```text
Workflow-level
      ↓
Job-level
      ↓
Step-level
```

If the same name exists at all three levels:

```text
Step
  ↓
Job
  ↓
Workflow
```

the most specific value is used for the step.

---

# Static Environment Variables

Example:

```yaml
env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: catalogue
  NAMESPACE: catalogue
```

These are useful when values do not need to change during workflow execution.

---

# Dynamic Environment Variables

Environment variables can be populated from GitHub contexts.

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
```

Now:

```text
IMAGE_TAG
    ↓
Current Commit SHA
```

---

# Branch-Based Environment Variable

Example:

```yaml
env:
  BRANCH_NAME: ${{ github.ref_name }}
```

Then:

```yaml
run: |
  echo "Branch: $BRANCH_NAME"
```

Possible value:

```text
main
```

or:

```text
feature/login
```

---

# Commit-Based Environment Variable

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
```

Useful for immutable application versions:

```text
Docker Image
    ↓
catalogue:<commit-sha>
```

This provides traceability between:

```text
Git Commit
Docker Image
Deployment
```

---

# Environment Name

Example:

```yaml
env:
  DEPLOYMENT_ENV: production
```

Then:

```yaml
run: |
  echo "Deploying to $DEPLOYMENT_ENV"
```

For reusable workflows, environment selection is often better represented using workflow inputs.

---

# Repository Variables as Environment Variables

A repository configuration variable can be assigned to `env`.

Example:

```yaml
env:
  AWS_REGION: ${{ vars.AWS_REGION }}
```

Then:

```yaml
run: |
  echo "Region: $AWS_REGION"
```

This separates:

```text
Configuration Storage
```

from:

```text
Workflow Execution Environment
```

---

# Environment Configuration Variables

A GitHub Environment can have configuration variables.

Example environments:

```text
qa
uat
production
```

Each can have different configuration.

Conceptually:

```text
QA
 └── API_URL=qa.example

UAT
 └── API_URL=uat.example

PROD
 └── API_URL=prod.example
```

---

# Selecting an Environment

Example:

```yaml
jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          echo "Production deployment"
```

The job is associated with the `production` GitHub Environment.

---

# Environment Variables vs GitHub Environments

These are different concepts.

### Environment variable

```yaml
env:
  APP_NAME: catalogue
```

### GitHub Environment

```yaml
environment:
  name: production
```

A GitHub Environment can provide:

```text
Environment Variables
Environment Secrets
Protection Rules
Deployment Controls
```

---

# Environment-Specific Configuration

For your environment promotion model:

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

Each environment can have configuration such as:

```text
Namespace
API URL
AWS Region
Cluster
ECR Repository
Helm Values
```

---

# Example Environment Configuration

```text
QA
 ├── NAMESPACE=qa
 ├── CLUSTER_NAME=qa-cluster
 └── API_URL=https://qa.example

UAT
 ├── NAMESPACE=uat
 ├── CLUSTER_NAME=uat-cluster
 └── API_URL=https://uat.example

PROD
 ├── NAMESPACE=production
 ├── CLUSTER_NAME=prod-cluster
 └── API_URL=https://example
```

---

# Environment Variables and Secrets

Environment variables:

```text
Non-sensitive
```

Secrets:

```text
Sensitive
```

Example:

```text
APP_NAME          → Variable
AWS_REGION        → Variable
IMAGE_TAG         → Variable
JIRA_API_TOKEN    → Secret
DATABASE_PASSWORD → Secret
```

Never put credentials into normal environment variables stored in workflow source.

---

# Secrets as Environment Variables

A secret can be made available to a process through `env`.

Example:

```yaml
steps:

  - name: Deploy
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}
    run: |
      ./deploy.sh
```

The process receives:

```text
API_TOKEN
```

without placing the secret directly in the command.

---

# Never Print Secrets

Bad:

```yaml
run: |
  echo "$API_TOKEN"
```

Never intentionally print credentials.

Good:

```yaml
run: |
  ./deploy.sh
```

where the deployment tool consumes the secret without exposing it.

---

# Environment Variable Naming

Use descriptive names:

```text
APP_NAME
AWS_REGION
IMAGE_TAG
ECR_REPOSITORY
NAMESPACE
HELM_RELEASE
DEPLOYMENT_ENV
```

Avoid:

```text
A
X
VALUE
TEMP
ABC
```

---

# Naming Convention

A common convention:

```text
UPPERCASE_WITH_UNDERSCORES
```

Example:

```text
APPLICATION_NAME
DOCKER_IMAGE
DEPLOYMENT_ENVIRONMENT
```

Use a consistent convention across your organization.

---

# Environment Variables in Bash

Example:

```yaml
env:
  APP_NAME: catalogue

steps:

  - name: Display
    shell: bash
    run: |
      echo "Application: $APP_NAME"
```

Bash accesses environment variables using:

```text
$VARIABLE_NAME
```

or:

```text
${VARIABLE_NAME}
```

---

# Environment Variables in PowerShell

Example:

```yaml
env:
  APP_NAME: catalogue

steps:

  - name: Display
    shell: pwsh
    run: |
      Write-Host "Application: $env:APP_NAME"
```

PowerShell uses:

```text
$env:VARIABLE_NAME
```

---

# Environment Variables in Python

Example:

```yaml
env:
  APP_NAME: catalogue

steps:

  - name: Read Variable
    run: |
      python -c "import os; print(os.environ['APP_NAME'])"
```

Python reads environment variables through the operating system environment.

---

# Environment Variables in Node.js

Example:

```yaml
env:
  APP_NAME: catalogue

steps:

  - name: Read Variable
    run: |
      node -e "console.log(process.env.APP_NAME)"
```

---

# Environment Variables in Java

Java applications can access environment variables through the standard Java environment APIs.

Example:

```text
System.getenv("APP_NAME")
```

This allows workflow configuration to be passed into the application.

---

# Environment Variables in Docker

Example:

```yaml
env:
  IMAGE_NAME: catalogue
  IMAGE_TAG: ${{ github.sha }}

steps:

  - name: Build
    run: |
      docker build \
        -t "$IMAGE_NAME:$IMAGE_TAG" \
        .
```

---

# Environment Variables in Helm

Example:

```yaml
env:
  RELEASE_NAME: catalogue
  NAMESPACE: production
  IMAGE_TAG: ${{ github.sha }}

steps:

  - name: Deploy
    run: |
      helm upgrade \
        "$RELEASE_NAME" \
        ./helm/catalogue \
        --namespace "$NAMESPACE" \
        --set image.tag="$IMAGE_TAG"
```

---

# Environment Variables in Kubernetes

Example:

```yaml
env:
  NAMESPACE: production

steps:

  - name: Check Pods
    run: |
      kubectl get pods -n "$NAMESPACE"
```

---

# Environment Variables in Terraform

Example:

```yaml
env:
  TF_IN_AUTOMATION: true
  TF_INPUT: false

steps:

  - name: Terraform Validate
    run: |
      terraform validate
```

Environment variables can be used to control tool behavior.

---

# Environment Variables for AWS

Example:

```yaml
env:
  AWS_REGION: ap-south-1
```

Then:

```yaml
run: |
  aws sts get-caller-identity
```

For AWS authentication, do not store long-lived access keys in environment variables.

Prefer OIDC where supported.

---

# OIDC and Environment Variables

A secure pattern:

```text
GitHub Actions
      |
      ↓
OIDC
      |
      ↓
AWS IAM Role
      |
      ↓
Temporary Credentials
      |
      ↓
AWS CLI / Application
```

Configuration:

```text
AWS_REGION → Variable
AWS Role → Controlled configuration
AWS Credentials → Temporary OIDC credentials
```

---

# Environment Variables and ECR

Example:

```yaml
env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: catalogue
  IMAGE_TAG: ${{ github.sha }}
```

Then:

```text
AWS Region
    +
ECR Repository
    +
Image Tag
```

can identify the target image.

---

# Environment Variables and GitOps

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
  SERVICE_NAME: catalogue
```

Flow:

```text
Commit SHA
   |
   ↓
IMAGE_TAG
   |
   ↓
Docker Image
   |
   ↓
ECR
   |
   ↓
GitOps Manifest
   |
   ↓
ArgoCD
   |
   ↓
EKS
```

---

# Environment Variables and Microservices

For a microservices platform:

```text
user
catalogue
cart
orders
payment
inventory
notification
```

Common variables might include:

```text
SERVICE_NAME
IMAGE_TAG
ECR_REPOSITORY
NAMESPACE
ENVIRONMENT
```

---

# Service-Specific Variables

Example:

```yaml
env:
  SERVICE_NAME: catalogue
  ECR_REPOSITORY: catalogue
  NAMESPACE: catalogue
```

The same workflow pattern can be reused for another service:

```text
SERVICE_NAME=payment
ECR_REPOSITORY=payment
NAMESPACE=payment
```

---

# Environment Variables and Matrix

Example:

```yaml
jobs:

  test:

    strategy:
      matrix:
        node:
          - '20'
          - '22'

    runs-on: ubuntu-latest

    env:
      NODE_VERSION: ${{ matrix.node }}

    steps:

      - name: Display Version
        run: |
          echo "Node: $NODE_VERSION"
```

Each matrix job gets its own value.

---

# Environment Variables and Matrix Environments

Example:

```yaml
strategy:
  matrix:
    environment:
      - qa
      - uat
```

Then:

```yaml
env:
  DEPLOYMENT_ENV: ${{ matrix.environment }}
```

The workflow can execute separate jobs for:

```text
qa
uat
```

---

# Environment Variables and `workflow_dispatch`

Manual workflows can receive inputs.

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Deployment environment
        required: true
        type: choice
        options:
          - qa
          - uat
          - production
```

Then:

```yaml
env:
  DEPLOYMENT_ENV: ${{ inputs.environment }}
```

---

# Manual Production Deployment

Example:

```text
User
 |
 ↓
workflow_dispatch
 |
 ↓
environment = production
 |
 ↓
Production Environment
 |
 ↓
Validation
 |
 ↓
Deployment
```

This is useful for controlled production deployments.

---

# Environment Variable Validation

Never blindly trust manually supplied environment values.

Example:

```bash
case "$DEPLOYMENT_ENV" in
  qa|uat|production)
    echo "Valid environment"
    ;;
  *)
    echo "Invalid deployment environment"
    exit 1
    ;;
esac
```

---

# Production Environment Guard

For production:

```text
Input
  |
  ↓
Validate
  |
  ↓
Production Environment
  |
  ↓
Approval / Protection
  |
  ↓
Deploy
```

Do not rely only on an environment variable named:

```text
production
```

Use GitHub Environment protection mechanisms for actual production controls.

---

# Environment Variables and Approvals

Conceptually:

```text
workflow_dispatch
      |
      ↓
environment = production
      |
      ↓
GitHub Environment
      |
      ↓
Required Approval
      |
      ↓
Deployment
```

The environment provides the governance boundary.

---

# Environment Variables and JIRA

Example:

```yaml
env:
  JIRA_PROJECT: PLATFORM
  JIRA_COMPONENT: catalogue
  DEPLOYMENT_ENV: production
```

These are configuration values.

A credential such as:

```text
JIRA_API_TOKEN
```

should be a secret.

---

# Production Change Request

Example:

```text
JIRA Ticket
      |
      ↓
JavaScript Validation Action
      |
      ├── Status
      ├── Approval
      ├── Window
      └── Version
      |
      ↓
Production Environment
      |
      ↓
Deploy
```

Environment variables can provide non-sensitive configuration to the validation process.

---

# Environment Variables and Deployment Windows

Example:

```text
DEPLOYMENT_ENV
DEPLOYMENT_WINDOW
TIMEZONE
```

can represent configuration.

However, deployment authorization should be validated by trusted systems rather than trusting user-controlled values.

---

# Environment Variables and Rollback

Example:

```yaml
env:
  RELEASE_NAME: catalogue
  NAMESPACE: production
  IMAGE_TAG: ${{ github.sha }}
```

A rollback process can identify:

```text
Current Version
Previous Version
Release Name
Namespace
```

The actual rollback operation should be controlled and auditable.

---

# Environment Variables and Helm Rollback

Example concept:

```text
Production
    |
    ↓
Helm Release
    |
    ↓
Failure
    |
    ↓
Rollback
```

Environment variables can provide:

```text
RELEASE_NAME
NAMESPACE
```

while the deployment system determines the rollback revision.

---

# Environment Variables and Reusable Workflows

Reusable workflow:

```yaml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
```

Then:

```yaml
env:
  DEPLOYMENT_ENV: ${{ inputs.environment }}
```

Caller:

```yaml
jobs:

  deploy:

    uses: company/platform-workflows/deploy.yml@v1

    with:
      environment: production
```

---

# Environment Variables and Custom Actions

Custom Actions can accept inputs:

```yaml
with:
  environment: production
```

Inside a workflow:

```yaml
env:
  DEPLOYMENT_ENV: ${{ inputs.environment }}
```

Choose the mechanism based on the interface you are designing.

---

# `env` vs `vars`

Important distinction:

### `env`

Defines an environment variable:

```yaml
env:
  APP_NAME: catalogue
```

Access:

```text
$APP_NAME
```

### `vars`

Reads a GitHub configuration variable:

```yaml
${{ vars.APP_NAME }}
```

You can combine them:

```yaml
env:
  APP_NAME: ${{ vars.APP_NAME }}
```

Then:

```text
$APP_NAME
```

---

# `env` vs `inputs`

### `env`

Execution-time environment variable:

```yaml
env:
  ENVIRONMENT: qa
```

### `inputs`

Explicit interface to a workflow or Action:

```yaml
with:
  environment: qa
```

For reusable components, inputs are generally clearer than hidden environment dependencies.

---

# `env` vs `outputs`

### `env`

Configuration provided to a process.

### `outputs`

Data produced by a step or job.

Example:

```text
Build Step
   |
   ↓
Output: image-tag
   |
   ↓
Deploy Job
```

---

# Environment Variables and Job Outputs

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.build.outputs.image-tag }}

    steps:

      - name: Build
        id: build
        run: |
          echo "image-tag=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"
```

A later job can consume:

```yaml
${{ needs.build.outputs.image-tag }}
```

---

# Environment Variables Are Not Automatically Shared Across Jobs

A variable defined inside one job does not automatically become an environment variable in another job.

Example:

```text
Job A
  |
  └── APP_VERSION
```

does not automatically mean:

```text
Job B
  └── APP_VERSION
```

For cross-job data, use:

```text
Job Outputs
Artifacts
Other supported mechanisms
```

---

# Cross-Job Example

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.version.outputs.image-tag }}

    steps:

      - name: Generate Version
        id: version
        run: |
          echo "image-tag=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"

  deploy:

    needs: build

    runs-on: ubuntu-latest

    env:
      IMAGE_TAG: ${{ needs.build.outputs.image-tag }}

    steps:

      - name: Deploy
        run: |
          echo "Deploying $IMAGE_TAG"
```

---

# Environment Variables and Artifacts

Use artifacts when the value is part of a file or build output.

Example:

```text
Build
  |
  ↓
Artifact
  |
  ↓
Deploy
```

Do not try to put large data into environment variables.

---

# Environment Variables and Files

For complex configuration:

```text
Environment Variable
       |
       ↓
Generate Configuration
       |
       ↓
config.yaml
       |
       ↓
Application
```

Keep secrets protected when generating configuration files.

---

# Environment Variables and Configuration Files

Example:

```yaml
env:
  APP_ENV: production
  API_URL: https://api.example
```

Then:

```bash
cat > config.env <<EOF
APP_ENV=$APP_ENV
API_URL=$API_URL
EOF
```

Be careful if any values can contain secrets or untrusted input.

---

# Multiline Environment Variables

Environment variables can contain multiline values, but this should be used carefully.

For complex structured content, consider:

```text
Files
Artifacts
Secrets
Configuration Stores
```

instead of very large environment variables.

---

# Environment Variable Size

Do not treat environment variables as a general-purpose storage mechanism.

They are intended for relatively small configuration values.

For larger data:

```text
Artifact
File
Object Storage
Configuration Service
```

may be more appropriate.

---

# Environment Variable Expansion

Example:

```yaml
env:
  APP_NAME: catalogue
  IMAGE_TAG: ${{ github.sha }}

steps:

  - name: Build
    run: |
      docker build -t "$APP_NAME:$IMAGE_TAG" .
```

GitHub evaluates expressions such as:

```text
${{ github.sha }}
```

before the command executes.

The shell then expands:

```text
$APP_NAME
$IMAGE_TAG
```

---

# Expressions and Environment Variables

Example:

```yaml
env:
  IS_MAIN: ${{ github.ref == 'refs/heads/main' }}
```

Then:

```yaml
run: |
  echo "Main branch: $IS_MAIN"
```

Expressions are covered in more detail in:

```text
07-Expressions.md
```

---

# Conditional Use

Environment variables can be used in workflow logic through expressions.

Example:

```yaml
env:
  DEPLOY_ENV: production

steps:

  - name: Production Step
    if: ${{ env.DEPLOY_ENV == 'production' }}
    run: |
      echo "Production deployment"
```

---

# Avoid Unsafe Production Conditions

Do not rely solely on:

```yaml
if: env.DEPLOY_ENV == 'production'
```

for security.

Production authorization should use:

```text
GitHub Environment
+
Permissions
+
Approvals
+
Change Management
```

where required.

---

# Environment Variables and Conditions

Example:

```yaml
if: ${{ env.DEPLOY_ENV == 'production' }}
```

This controls workflow execution.

However:

```text
Condition
≠
Authorization
```

Use proper environment protection for production.

---

# Environment Variables and Security Boundaries

A strong architecture:

```text
Configuration
    ↓
Variables

Sensitive Data
    ↓
Secrets

Cloud Identity
    ↓
OIDC

Production Authorization
    ↓
GitHub Environment
```

Each mechanism has a different purpose.

---

# Production Deployment Architecture

```text
Developer
   |
   ↓
GitHub Workflow
   |
   ├── Variables
   ├── Secrets
   ├── OIDC
   └── Environment
          |
          ↓
     Production Gate
          |
          ↓
      Deployment
```

---

# Example: Production Workflow

```yaml
name: Production Deployment

on:
  workflow_dispatch:
    inputs:
      version:
        description: Image version
        required: true
        type: string

permissions:
  contents: read
  id-token: write

jobs:

  deploy:

    environment:
      name: production

    runs-on:
      - self-hosted
      - linux
      - production

    env:
      AWS_REGION: ${{ vars.AWS_REGION }}
      IMAGE_TAG: ${{ inputs.version }}
      NAMESPACE: ${{ vars.NAMESPACE }}

    steps:

      - name: Display Deployment
        run: |
          echo "Region: $AWS_REGION"
          echo "Image: $IMAGE_TAG"
          echo "Namespace: $NAMESPACE"

      - name: Deploy
        run: |
          echo "Deploying application"
```

---

# Production Variable Strategy

Use:

```text
Repository Variables
        |
        ↓
Common Configuration

Environment Variables
        |
        ↓
Environment Configuration

Secrets
        |
        ↓
Sensitive Credentials

OIDC
        |
        ↓
Temporary Cloud Credentials
```

---

# Common Mistakes

### 1. Storing secrets as environment variables in source

Never commit credentials.

### 2. Printing environment variables blindly

A variable could contain sensitive information.

### 3. Using one global environment configuration for every environment

Environment-specific configuration should be isolated where necessary.

### 4. Treating environment variables as authorization

A variable saying:

```text
production
```

does not make a deployment authorized.

### 5. Sharing job data through `env`

Use job outputs for cross-job values.

### 6. Putting large data into environment variables

Use files or artifacts instead.

### 7. Mixing `vars`, `env`, and `inputs`

Understand their different purposes.

---

# Best Practices

- Use environment variables for runtime configuration.
- Keep scopes as narrow as practical.
- Use workflow-level variables only for truly global configuration.
- Use job-level variables for job-specific configuration.
- Use step-level variables for temporary configuration.
- Use GitHub configuration variables for reusable non-sensitive values.
- Use GitHub Environments for environment-specific configuration and protection.
- Use secrets for sensitive values.
- Prefer OIDC for cloud authentication where supported.
- Use inputs for explicit reusable workflow/Action interfaces.
- Use outputs for values produced during execution.
- Validate manual deployment inputs.
- Avoid printing sensitive environment variables.
- Keep production authorization separate from configuration.
- Document important environment variables.
- Use consistent naming conventions.

---

# Production-Level Environment Strategy

```text
                         GitHub
                           |
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
       Variables       Environments       Secrets
          |                |                |
          ↓                ↓                ↓
      Non-sensitive    QA/UAT/PROD       Sensitive
      Configuration     + Protection     Credentials
          |                |                |
          └────────────────┼────────────────┘
                           ↓
                       Workflow
                           |
                 ┌─────────┴─────────┐
                 ↓                   ↓
               OIDC             Deployment
                 |                   |
                 ↓                   ↓
              AWS IAM               EKS
```

---

# Key Takeaways

Environment variables are primarily used to provide configuration to workflow processes.

Remember:

```text
env
 ↓
Runtime Environment Variable

vars
 ↓
GitHub Configuration Variable

inputs
 ↓
Workflow / Action Input

outputs
 ↓
Generated Execution Data

secrets
 ↓
Sensitive Data

OIDC
 ↓
Short-Lived Cloud Identity

environment
 ↓
Deployment Environment + Protection
```

The key principle:

```text
Use environment variables for configuration,
not as a replacement for secrets, outputs, or authorization controls.
```

---

# Interview Questions

## Basic

1. What are environment variables in GitHub Actions?
2. How do you define an environment variable?
3. What is the difference between workflow-level and job-level environment variables?
4. What is the difference between job-level and step-level environment variables?
5. How do you access an environment variable in Bash?
6. How do you access one in PowerShell?
7. How do you access one from Python?
8. How do you access one from Node.js?
9. How do you use an environment variable in Docker commands?
10. How do you use environment variables with Helm?

## Intermediate

11. How does environment variable precedence work?
12. What is the difference between `env` and `vars`?
13. What is the difference between `env` and `inputs`?
14. What is the difference between `env` and `outputs`?
15. Can an environment variable automatically pass from one job to another?
16. How do you pass values between jobs?
17. How do you use environment variables with matrix jobs?
18. How do you use environment variables with `workflow_dispatch`?
19. How do you use environment-specific variables?
20. How do you use secrets as environment variables?
21. How do you prevent secrets from appearing in logs?
22. How do you validate environment input values?
23. How do you use environment variables with reusable workflows?
24. How do you use environment variables with custom Actions?

## Advanced / Production

25. Design an environment-variable strategy for DEV, QA, SIT, UAT, and PROD.
26. How would you separate configuration, secrets, identity, and authorization?
27. How would you use environment variables with AWS OIDC?
28. How would you configure environment-specific ECR repositories?
29. How would you configure environment-specific Kubernetes namespaces?
30. How would you pass an immutable image SHA from build to production deployment?
31. How would you use environment variables in a GitOps workflow with ECR, ArgoCD, and EKS?
32. How would you design production protection so changing an environment variable cannot bypass deployment controls?
33. How would you use GitHub Environments with environment variables and environment secrets?
34. How would you design environment variables for a microservices platform?
35. How would you prevent QA configuration from being accidentally used in production?
36. How would you audit organization and repository environment variables?
37. How would you handle a secret accidentally exposed through an environment variable?
38. How would you design a production workflow using `env`, `vars`, `inputs`, `outputs`, `secrets`, OIDC, and GitHub Environments together?
39. How would you troubleshoot a variable that has the expected value in one job but a different value in another?
40. Explain the complete lifecycle of a deployment configuration value from GitHub configuration through the workflow, runner, Docker image, Helm deployment, and EKS.