# GitHub Actions Variables

Variables are values that can be defined once and reused throughout a GitHub Actions workflow.

They are useful for storing configuration values that are:

```text
Non-sensitive
Reusable
Environment-specific
Workflow-specific
Job-specific
```

Examples:

```text
Application Name
AWS Region
ECR Repository
Helm Chart
Namespace
Environment
Terraform Directory
Docker Image Name
```

---

# Why Use Variables?

Without variables:

```yaml
run: docker build -t catalogue:production .
```

The value is hardcoded.

With variables:

```yaml
env:
  APP_NAME: catalogue
  ENVIRONMENT: production
```

Then:

```yaml
run: |
  docker build -t "$APP_NAME:$ENVIRONMENT" .
```

Benefits:

```text
Less duplication
Easier maintenance
Reusable configuration
Cleaner workflows
Environment flexibility
```

---

# Variables vs Secrets

Important distinction:

### Variables

Used for:

```text
Non-sensitive configuration
```

Examples:

```text
AWS_REGION
APP_NAME
ENVIRONMENT
ECR_REPOSITORY
```

### Secrets

Used for:

```text
Sensitive credentials
```

Examples:

```text
API_TOKEN
PASSWORD
PRIVATE_KEY
```

Never store sensitive credentials as normal variables.

---

# Variable Scope

GitHub Actions variables can exist at different levels.

Conceptually:

```text
Repository
    |
    └── Workflow
          |
          ├── Job
          │    |
          │    └── Step
          |
          └── Job
```

The more specific scope can provide a value for that execution context.

---

# Workflow-Level Environment Variables

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

      - name: Display Configuration
        run: |
          echo "Application: $APP_NAME"
          echo "Region: $AWS_REGION"
```

The variables are available to the workflow's jobs and steps unless overridden.

---

# Job-Level Variables

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    env:
      APP_NAME: catalogue
      ENVIRONMENT: qa

    steps:

      - name: Display
        run: |
          echo "Application: $APP_NAME"
          echo "Environment: $ENVIRONMENT"
```

The variables apply to that job's steps.

---

# Step-Level Variables

Example:

```yaml
steps:

  - name: Build
    env:
      BUILD_TYPE: release
    run: |
      echo "Build type: $BUILD_TYPE"
```

The variable applies only to that step.

---

# Scope Example

```text
Workflow
  |
  └── APP_NAME=catalogue
        |
        ├── Build Job
        │     |
        │     └── Step
        |
        └── Test Job
              |
              └── Step
```

A job-specific variable is narrower:

```text
Workflow
  |
  ├── Build Job
  │     └── APP_NAME=catalogue
  |
  └── Test Job
```

---

# Precedence

When the same variable name is defined at multiple levels, the more specific scope can override the broader scope for that step.

Conceptually:

```text
Workflow
   ↓
Job
   ↓
Step
```

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
          ENVIRONMENT: uat
        run: echo "$ENVIRONMENT"
```

The step sees:

```text
uat
```

---

# `env` Variables

A common way to define environment variables is:

```yaml
env:
  APP_NAME: catalogue
  VERSION: ${{ github.sha }}
```

Then:

```yaml
run: |
  echo "$APP_NAME"
  echo "$VERSION"
```

---

# Static Variables

Example:

```yaml
env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: catalogue
  HELM_RELEASE: catalogue
```

These are configuration values, not secrets.

---

# Dynamic Variables

Variables can also be assigned from GitHub context or expressions.

Example:

```yaml
env:
  VERSION: ${{ github.sha }}
```

Now:

```text
VERSION
    ↓
Current commit SHA
```

---

# Repository Variables

Repository-level variables can be configured in GitHub repository settings.

They can be useful for values shared across workflows.

Example conceptual values:

```text
AWS_REGION
ECR_REGISTRY
DEFAULT_BRANCH
```

They can be referenced using the `vars` context.

Example:

```yaml
run: echo "${{ vars.AWS_REGION }}"
```

---

# Organization Variables

Organizations can define variables shared across repositories.

Conceptually:

```text
Organization
     |
     ├── Repository A
     ├── Repository B
     ├── Repository C
     └── Repository D
```

A common organization variable might be:

```text
AWS_REGION
```

if appropriate for the organization's workflows.

---

# Environment Variables vs Configuration Variables

Do not confuse:

```text
env
```

with:

```text
vars
```

`env` defines environment variables for workflow execution.

`vars` accesses GitHub configuration variables.

Example:

```yaml
env:
  APP_NAME: catalogue
```

Access:

```text
$APP_NAME
```

Whereas:

```yaml
run: echo "${{ vars.APP_NAME }}"
```

accesses a GitHub configuration variable.

---

# Configuration Variables

GitHub configuration variables can be defined at supported levels such as:

```text
Organization
Repository
Environment
```

They are intended for non-sensitive configuration.

---

# `vars` Context

Example:

```yaml
name: Example

on:
  workflow_dispatch:

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Display Region
        run: |
          echo "Region: ${{ vars.AWS_REGION }}"
```

---

# Environment Configuration Variables

A GitHub Environment can have its own configuration variables.

For example:

```text
qa
uat
production
```

Each environment can have different configuration.

Conceptually:

```text
qa
 └── API_URL=qa.example

uat
 └── API_URL=uat.example

production
 └── API_URL=prod.example
```

---

# Environment-Specific Configuration

Example:

```yaml
jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Display URL
        run: |
          echo "${{ vars.API_URL }}"
```

The value comes from the selected environment's configuration.

---

# Variables and Environments

For your deployment model:

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

Environment-specific variables can provide:

```text
Namespace
Cluster Name
ECR Repository
Helm Values
Application URL
Deployment Configuration
```

---

# Production Example

```yaml
jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    env:
      IMAGE_TAG: ${{ github.sha }}

    steps:

      - name: Deploy
        run: |
          echo "Environment: production"
          echo "Image: $IMAGE_TAG"
          echo "Region: ${{ vars.AWS_REGION }}"
```

---

# Variables and GitHub Context

Variables can be combined with contexts.

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
  BRANCH: ${{ github.ref_name }}
```

Then:

```yaml
run: |
  echo "Branch: $BRANCH"
  echo "Image: $IMAGE_TAG"
```

---

# Variables and Expressions

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
```

Expressions are evaluated by GitHub Actions.

Then the resulting value becomes available to the step.

---

# Variables in `with`

Some Actions expect inputs through `with`, not `env`.

Example:

```yaml
- name: Setup Node
  uses: actions/setup-node@v4
  with:
    node-version: ${{ vars.NODE_VERSION }}
```

This is different from:

```yaml
env:
  NODE_VERSION: '22'
```

---

# `env` vs `with`

### `env`

Provides environment variables:

```yaml
env:
  APP_NAME: catalogue
```

### `with`

Provides inputs to an Action:

```yaml
with:
  node-version: '22'
```

Do not assume an Action's input automatically becomes an environment variable.

---

# Variables in Shell Commands

Example:

```yaml
env:
  APP_NAME: catalogue
  VERSION: v1.0.0

steps:

  - name: Build
    run: |
      echo "Building $APP_NAME"
      echo "Version $VERSION"
```

---

# Variables in PowerShell

On Windows runners:

```yaml
env:
  APP_NAME: catalogue

steps:

  - name: Display
    shell: pwsh
    run: |
      Write-Host "Application: $env:APP_NAME"
```

Shell syntax depends on the shell being used.

---

# Variables in Python

Example:

```yaml
env:
  APP_NAME: catalogue

steps:

  - name: Script
    run: |
      python -c "import os; print(os.environ['APP_NAME'])"
```

---

# Variables in Java

Example:

```yaml
env:
  APP_ENV: qa

steps:

  - name: Run
    run: |
      mvn test
```

The Java application can access the environment variable through the standard environment APIs.

---

# Variables in Docker

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

# Variables for ECR

Example:

```yaml
env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: catalogue
  IMAGE_TAG: ${{ github.sha }}
```

Then:

```yaml
run: |
  echo "Repository: $ECR_REPOSITORY"
  echo "Tag: $IMAGE_TAG"
```

---

# Variables for Kubernetes

Example:

```yaml
env:
  NAMESPACE: catalogue
  IMAGE_TAG: ${{ github.sha }}
```

Then:

```yaml
run: |
  kubectl -n "$NAMESPACE" get pods
```

---

# Variables for Helm

Example:

```yaml
env:
  RELEASE_NAME: catalogue
  NAMESPACE: catalogue
  IMAGE_TAG: ${{ github.sha }}
```

Then:

```yaml
run: |
  helm upgrade \
    "$RELEASE_NAME" \
    ./helm/catalogue \
    --namespace "$NAMESPACE" \
    --set image.tag="$IMAGE_TAG"
```

---

# Variables for Terraform

Example:

```yaml
env:
  TF_IN_AUTOMATION: true
  TF_INPUT: false
```

These can control Terraform behavior during automation.

---

# Boolean Values

Be careful with YAML types.

Example:

```yaml
env:
  DEBUG: false
```

Environment variables are ultimately represented as strings when passed to processes.

If your application expects:

```text
"false"
```

handle the value accordingly.

---

# Numeric Values

Example:

```yaml
env:
  RETRY_COUNT: '3'
```

Explicitly quoting values can make the intended string representation clear.

---

# Special Characters

Be careful with values containing:

```text
:
#
$
"
'
spaces
```

Example:

```yaml
env:
  MESSAGE: "Build completed successfully"
```

Use appropriate YAML quoting.

---

# Variable Naming

Use clear names:

```text
APP_NAME
AWS_REGION
IMAGE_TAG
ECR_REPOSITORY
NAMESPACE
ENVIRONMENT
```

Avoid unclear names:

```text
X
TEMP
ABC
VALUE1
```

---

# Naming Convention

A common convention is:

```text
UPPERCASE_WITH_UNDERSCORES
```

Example:

```text
APPLICATION_NAME
DOCKER_IMAGE
DEPLOYMENT_ENVIRONMENT
```

Consistency is more important than any single naming convention.

---

# Variable Reuse

Bad:

```yaml
run: docker build -t catalogue:${{ github.sha }} .
```

repeated many times.

Better:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
```

Then:

```yaml
run: docker build -t catalogue:$IMAGE_TAG .
```

---

# Variables for DRY Workflows

Variables help reduce duplication.

Example:

```yaml
env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: catalogue
  IMAGE_TAG: ${{ github.sha }}
```

Then multiple steps can reuse them.

---

# Avoid Excessive Variables

Do not create variables for every small value.

Bad:

```text
COMMAND1
COMMAND2
VALUE1
VALUE2
NAME1
NAME2
```

Variables should improve readability rather than hide the workflow.

---

# Variables and Security

Variables are not a replacement for secrets.

Never store:

```text
Password
API Token
Private Key
AWS Secret Access Key
Database Credential
```

as ordinary variables.

Use GitHub Secrets or an appropriate external secret-management system.

---

# Variable Visibility

A normal variable may appear in logs if you print it.

Example:

```yaml
run: echo "$APP_NAME"
```

This is fine for non-sensitive values.

Do not assume variables are automatically masked.

---

# Variable Precedence Example

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

because the step-level value is more specific.

---

# Variables and Matrix Jobs

Variables can work with matrix configurations.

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
  ENVIRONMENT: ${{ matrix.environment }}
```

The jobs receive:

```text
qa
uat
```

---

# Matrix Example

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

      - name: Display
        run: |
          echo "Node: $NODE_VERSION"
```

---

# Variables and Job Outputs

Variables and outputs serve different purposes.

```text
Variable
   |
   ↓
Configuration

Output
   |
   ↓
Value produced by a step/job
```

Example:

```text
IMAGE_TAG
```

can be generated by one job and passed as a job output.

---

# Do Not Confuse `env`, `vars`, and `outputs`

### `env`

Environment variable:

```yaml
env:
  APP_NAME: catalogue
```

### `vars`

GitHub configuration variable:

```yaml
${{ vars.APP_NAME }}
```

### `outputs`

Value produced by a step/job:

```yaml
${{ steps.build.outputs.image }}
```

---

# Production Variable Strategy

A good production design separates:

```text
Code
Configuration
Secrets
```

Example:

```text
Code
 ↓
Git Repository

Configuration
 ↓
GitHub Variables / Environment Variables

Secrets
 ↓
GitHub Secrets / External Secret Manager
```

---

# Production Example

```text
Application
      |
      ├── APP_NAME
      ├── IMAGE_TAG
      ├── AWS_REGION
      └── NAMESPACE
```

Sensitive:

```text
      ├── AWS credentials
      ├── JIRA token
      └── Database password
```

The first group can be configuration.

The second group must be protected as secrets.

---

# Environment Promotion

Variables can represent environment-specific configuration:

```text
QA
 |
 ├── NAMESPACE=qa
 ├── API_URL=qa.example
 └── ECR_REPOSITORY=catalogue

UAT
 |
 ├── NAMESPACE=uat
 ├── API_URL=uat.example
 └── ECR_REPOSITORY=catalogue

PROD
 |
 ├── NAMESPACE=prod
 ├── API_URL=prod.example
 └── ECR_REPOSITORY=catalogue
```

---

# Environment-Specific Deployment

Example:

```yaml
jobs:

  deploy:

    environment:
      name: ${{ inputs.environment }}

    env:
      IMAGE_TAG: ${{ github.sha }}

    steps:

      - name: Deploy
        run: |
          echo "Environment: ${{ inputs.environment }}"
          echo "Image: $IMAGE_TAG"
```

The environment can supply its own configuration and protection rules.

---

# Production Configuration Pattern

Recommended:

```text
Repository Variables
       |
       ↓
Common Configuration

Environment Variables
       |
       ↓
Environment-specific Configuration

Secrets
       |
       ↓
Sensitive Data

OIDC
       |
       ↓
Temporary Cloud Credentials
```

---

# Variables and OIDC

Do not store cloud credentials in variables.

Use:

```text
AWS_REGION → variable
AWS_ROLE_ARN → variable if appropriate
AWS credentials → OIDC
```

Conceptually:

```text
Variable
  |
  ↓
Role ARN
  |
  ↓
OIDC
  |
  ↓
Temporary AWS Credentials
```

---

# Variables and JIRA

Example:

```text
JIRA_PROJECT
JIRA_COMPONENT
JIRA_BASE_URL
```

can be configuration.

But:

```text
JIRA_API_TOKEN
```

should be treated as a secret.

---

# Variables and CR Process

For a controlled production deployment:

```text
JIRA Ticket
Version / SHA
Environment
Deployment Window
```

can be inputs/configuration.

Credentials used to access JIRA must be protected.

---

# Variables and Rollback

Example:

```yaml
env:
  RELEASE_NAME: catalogue
  NAMESPACE: production
  IMAGE_TAG: ${{ github.sha }}
```

Rollback can use a previous known version.

Example concept:

```text
Current
  ↓
SHA-A

Previous
  ↓
SHA-B
```

A deployment system can use the previous known version according to its rollback strategy.

---

# Variables in Reusable Workflows

Reusable workflows can receive inputs.

Example:

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
  ENVIRONMENT: ${{ inputs.environment }}
```

This allows standardized workflows to remain configurable.

---

# Variables in Reusable Workflows

Caller:

```yaml
jobs:

  deploy:

    uses: company/platform-workflows/deploy.yml@v1

    with:
      environment: production
```

The reusable workflow receives the value as an input.

---

# Variables in Custom Actions

Custom Actions can receive:

```text
inputs
```

rather than relying on hidden environment variables.

Example:

```yaml
with:
  environment: production
```

This makes the Action interface clearer.

---

# Variable Documentation

For important variables document:

```text
Name
Purpose
Scope
Default
Allowed Values
Sensitive?
Owner
```

Example:

| Variable | Purpose | Sensitive |
|---|---|---|
| AWS_REGION | AWS region | No |
| IMAGE_TAG | Image version | No |
| NAMESPACE | Kubernetes namespace | No |
| JIRA_API_TOKEN | JIRA authentication | Yes |

---

# Common Mistakes

### 1. Storing secrets as variables

Never store credentials in normal variables.

### 2. Hardcoding environment configuration

Use appropriate configuration mechanisms.

### 3. Defining the same variable everywhere

Use the appropriate scope.

### 4. Confusing `vars` with `env`

They serve different purposes.

### 5. Confusing `env` with `with`

Action inputs are not the same as environment variables.

### 6. Using unclear names

Prefer descriptive names.

### 7. Printing sensitive values

Never expose secrets in logs.

### 8. Creating too many variables

Only abstract values that improve maintainability.

---

# Best Practices

- Use variables for non-sensitive configuration.
- Use clear naming conventions.
- Keep variable scope as narrow as practical.
- Use repository variables for shared repository configuration.
- Use environment variables for environment-specific configuration.
- Use `env` for workflow/job/step execution variables.
- Use `vars` for GitHub configuration variables.
- Use `inputs` for reusable workflow and Action interfaces.
- Use outputs for values generated during execution.
- Use secrets for sensitive values.
- Prefer OIDC for cloud credentials where supported.
- Validate environment-specific values.
- Document important variables.
- Avoid unnecessary duplication.
- Avoid printing sensitive data.
- Use environment protection for production deployments.

---

# Production-Level Variable Architecture

```text
                    GitHub Repository
                           |
          ┌────────────────┼────────────────┐
          ↓                ↓                ↓
     Variables         Environments        Secrets
          |                |                |
          ↓                ↓                ↓
    Common Config    QA/UAT/PROD       Credentials
          |                |                |
          └────────────────┼────────────────┘
                           ↓
                     GitHub Workflow
                           |
                           ↓
                    Deployment Logic
```

---

# Example Production Workflow

```yaml
name: Production Deployment

on:
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

env:
  AWS_REGION: ${{ vars.AWS_REGION }}
  IMAGE_TAG: ${{ github.sha }}

jobs:

  deploy:

    environment:
      name: production

    runs-on:
      - self-hosted
      - linux
      - production

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Display Configuration
        run: |
          echo "Region: $AWS_REGION"
          echo "Image: $IMAGE_TAG"

      - name: Deploy
        run: |
          echo "Deploying $IMAGE_TAG to production"
```

This demonstrates the separation between:

```text
Configuration
+
GitHub Context
+
Environment
+
Permissions
```

while keeping sensitive credentials out of variables.

---

# Key Takeaways

```text
Variables
=
Reusable non-sensitive configuration
```

Important mechanisms:

```text
env
vars
inputs
outputs
contexts
expressions
```

Security separation:

```text
Configuration → Variables
Sensitive Data → Secrets
Cloud Credentials → OIDC where supported
```

Scope:

```text
Workflow
   ↓
Job
   ↓
Step
```

The key principle:

```text
Use variables to make workflows configurable,
but never use ordinary variables as a substitute for secrets.
```

---

# Interview Questions

## Basic

1. What are variables in GitHub Actions?
2. Why do we use variables?
3. What is the difference between a variable and a secret?
4. What is an environment variable?
5. What is the `env` keyword?
6. What is the `vars` context?
7. What are repository variables?
8. What are environment variables?
9. What is variable scope?
10. How do you access an environment variable in a shell?

## Intermediate

11. What is the difference between workflow-level, job-level, and step-level variables?
12. How does variable precedence work?
13. What is the difference between `env` and `vars`?
14. What is the difference between `env` and `with`?
15. How do you use GitHub context values as variables?
16. How do you use variables with matrix jobs?
17. How do variables work with reusable workflows?
18. How do you pass configuration to a custom Action?
19. When should you use inputs instead of environment variables?
20. When should you use outputs instead of variables?

## Advanced / Production

21. Design a variable strategy for DEV, QA, SIT, UAT, and PROD.
22. How would you separate configuration from secrets?
23. How would you manage AWS configuration without storing AWS credentials?
24. How would you use variables with GitHub OIDC and AWS IAM?
25. How would you manage environment-specific Kubernetes namespaces?
26. How would you manage environment-specific ECR configuration?
27. How would you design variables for a multi-microservice GitHub Actions platform?
28. How would you prevent configuration drift between QA, UAT, and production?
29. How would you manage organization-wide variables?
30. How would you handle a variable that must differ between repositories?
31. How would you securely pass JIRA configuration and credentials?
32. How would you design variables for a production JIRA change-request gate?
33. How would you pass the approved commit SHA through multiple jobs?
34. How would you use variables in a GitOps workflow with ECR, ArgoCD, and EKS?
35. How would you prevent production configuration from accidentally being used in QA?
36. How would you design variable naming and governance standards for an enterprise?
37. What would you do if a developer accidentally stored a password in a normal variable?
38. How would you audit and clean up unused repository and organization variables?
39. How would you design a production workflow that uses variables, environment protection, secrets, and OIDC together?
40. Explain the difference between `env`, `vars`, `inputs`, `outputs`, `secrets`, and contexts in a production GitHub Actions architecture.