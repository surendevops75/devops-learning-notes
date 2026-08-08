# GitHub Actions Inputs

Inputs allow values to be provided to workflows, reusable workflows, and custom Actions.

They make automation configurable instead of hardcoding values.

Typical inputs:

```text
Environment
Version
Service Name
Branch
Terraform Directory
Helm Chart
Deployment Type
JIRA Ticket
Image Tag
```

Example:

```text
User
  |
  ↓
Input
  |
  ↓
Workflow
  |
  ↓
Deployment
```

---

# Why Use Inputs?

Without inputs:

```yaml
run: |
  ./deploy.sh production
```

The workflow is tied to one environment.

With inputs:

```yaml
environment: production
```

the same workflow can support:

```text
qa
uat
production
```

---

# Types of Inputs

GitHub Actions inputs are commonly used with:

```text
workflow_dispatch
workflow_call
Custom Actions
```

These solve different problems.

---

# `workflow_dispatch` Inputs

`workflow_dispatch` allows a user to manually trigger a workflow and provide inputs.

Example:

```yaml
name: Deployment

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

---

# Manual Workflow Flow

```text
GitHub UI
   |
   ↓
Run workflow
   |
   ↓
Select Inputs
   |
   ↓
Workflow
   |
   ↓
Validation
   |
   ↓
Deployment
```

---

# String Input

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: Image version
        required: true
        type: string
```

User provides:

```text
version = abc123
```

Then:

```yaml
run: |
  echo "Deploying ${{ inputs.version }}"
```

---

# Choice Input

Use `choice` when only predefined values should be accepted.

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Environment
        required: true
        type: choice
        options:
          - qa
          - uat
          - production
```

This prevents arbitrary environment names from being entered through the UI.

---

# Boolean Input

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      run-tests:
        description: Run tests
        required: true
        type: boolean
```

Then:

```yaml
if: ${{ inputs.run-tests }}
```

Boolean inputs are useful for optional workflow behavior.

---

# Environment Input

GitHub Actions supports the `environment` input type for manual workflows.

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Deployment environment
        required: true
        type: environment
```

This allows the workflow to select from GitHub Environments.

---

# Production Environment Input

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: Deployment environment
        required: true
        type: environment
```

Then the job can use:

```yaml
jobs:

  deploy:

    environment:
      name: ${{ inputs.environment }}
```

This connects the user-selected environment to the GitHub Environment.

---

# `workflow_call` Inputs

Reusable workflows can define inputs using:

```yaml
on:
  workflow_call:
```

Example:

```yaml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
```

The caller provides the value.

---

# Reusable Workflow Flow

```text
Application Workflow
        |
        ↓
Reusable Workflow
        |
        ↓
Input
        |
        ↓
Build / Test / Deploy
```

---

# Caller Workflow

Example:

```yaml
jobs:

  deploy:

    uses: company/platform-workflows/deploy.yml@v1

    with:
      environment: production
```

The reusable workflow receives:

```text
environment = production
```

---

# Reusable Workflow Input Types

For `workflow_call`, inputs can use supported types such as:

```text
string
boolean
number
```

Example:

```yaml
on:
  workflow_call:
    inputs:

      environment:
        required: true
        type: string

      replicas:
        required: false
        type: number

      run-tests:
        required: false
        type: boolean
```

---

# Required Inputs

Example:

```yaml
inputs:

  environment:
    required: true
    type: string
```

The caller must provide the value.

---

# Optional Inputs

Example:

```yaml
inputs:

  environment:
    required: false
    type: string
    default: qa
```

If the caller doesn't provide the value:

```text
environment = qa
```

---

# Input Defaults

Example:

```yaml
on:
  workflow_call:
    inputs:
      environment:
        required: false
        type: string
        default: qa
```

This is useful when one environment is the normal default.

---

# Custom Action Inputs

Custom Actions define inputs in:

```text
action.yml
```

Example:

```yaml
inputs:

  environment:
    description: Target environment
    required: true

  version:
    description: Application version
    required: true
```

---

# Calling a Custom Action

Example:

```yaml
- name: Deploy
  uses: company/platform-actions/deploy@v1
  with:
    environment: production
    version: ${{ github.sha }}
```

---

# Input Flow for Custom Actions

```text
Workflow
   |
   ↓
with:
   |
   ├── environment
   └── version
   |
   ↓
Custom Action
```

---

# `with` Keyword

`with` passes inputs to an Action or reusable workflow.

Example:

```yaml
- name: Setup Node
  uses: actions/setup-node@v4
  with:
    node-version: '22'
```

Do not confuse:

```yaml
with:
```

with:

```yaml
env:
```

---

# `with` vs `env`

### `with`

Provides an input to an Action:

```yaml
with:
  node-version: '22'
```

### `env`

Provides an environment variable:

```yaml
env:
  NODE_VERSION: '22'
```

The Action only receives `with` values that it explicitly defines as inputs.

---

# Input Validation

Inputs should be validated before performing important operations.

Example:

```bash
case "$ENVIRONMENT" in
  qa|uat|production)
    echo "Valid environment"
    ;;
  *)
    echo "Invalid environment"
    exit 1
    ;;
esac
```

---

# Why Validate Inputs?

Inputs can influence:

```text
Deployment
Infrastructure
Cloud Resources
Production Systems
APIs
Git Operations
```

Invalid input can cause:

```text
Deployment Failure
Wrong Environment
Unexpected Resource Changes
Security Issues
```

---

# Environment Validation

Bad:

```yaml
run: |
  ./deploy.sh ${{ inputs.environment }}
```

without validation.

Better:

```bash
case "$DEPLOYMENT_ENV" in
  qa|uat|production)
    ./deploy.sh "$DEPLOYMENT_ENV"
    ;;
  *)
    echo "Invalid environment"
    exit 1
    ;;
esac
```

---

# Version Input

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        description: Version to deploy
        required: true
        type: string
```

Then:

```yaml
env:
  IMAGE_TAG: ${{ inputs.version }}
```

---

# Immutable Versioning

For production, prefer immutable identifiers such as:

```text
Git SHA
Image Digest
Release Version
```

Example:

```text
IMAGE_TAG = 8a92f31...
```

rather than:

```text
latest
```

---

# Service Input

For a multi-microservice platform:

```yaml
on:
  workflow_dispatch:
    inputs:
      service:
        description: Service to deploy
        required: true
        type: choice
        options:
          - user
          - catalogue
          - cart
          - orders
          - payment
          - inventory
          - notification
```

This allows controlled service selection.

---

# Deployment Type Input

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      deployment-type:
        description: Deployment strategy
        required: true
        type: choice
        options:
          - rolling
          - canary
          - blue-green
```

The deployment logic can then select the appropriate strategy.

---

# JIRA Ticket Input

For production deployment:

```yaml
on:
  workflow_dispatch:
    inputs:
      jira-ticket:
        description: Approved JIRA ticket
        required: true
        type: string
```

Then:

```yaml
env:
  JIRA_TICKET: ${{ inputs.jira-ticket }}
```

---

# Production Input Set

A controlled production workflow might request:

```text
Environment
JIRA Ticket
Version / SHA
Service
Deployment Strategy
```

Example:

```text
Environment → production
JIRA Ticket → PROJ-1234
Version → 8a92f31
Service → catalogue
Strategy → rolling
```

---

# Production Deployment Flow

```text
User
 |
 ↓
Manual Trigger
 |
 ├── Environment
 ├── JIRA Ticket
 ├── Version
 └── Service
 |
 ↓
Validate Inputs
 |
 ↓
JIRA Validation
 |
 ↓
SHA Validation
 |
 ↓
Production Environment
 |
 ↓
Approval
 |
 ↓
Deploy
```

---

# Input Validation and JIRA

A production workflow should not assume:

```text
JIRA Ticket = Approved
```

because a user entered it.

Instead:

```text
Input
  |
  ↓
JIRA API
  |
  ├── Ticket exists?
  ├── Correct project?
  ├── Correct component?
  ├── Approved?
  ├── Correct version?
  └── Deployment window?
  |
  ↓
PASS / FAIL
```

---

# Input Validation and Commit SHA

Example:

```text
Requested Version
      |
      ↓
Compare with approved SHA
      |
      ↓
Same?
  /     \
YES      NO
 |        |
PASS      FAIL
```

This protects production from deploying an unapproved version.

---

# Inputs and Secrets

Do not use inputs to collect secrets through normal workflow interfaces.

Bad design:

```text
Input:
password
```

Use:

```text
GitHub Secret
```

instead.

---

# Input vs Secret

### Input

```text
JIRA Ticket
Environment
Version
Service
```

### Secret

```text
JIRA API Token
Database Password
Private Key
API Credential
```

---

# Input vs Variable

### Input

Provided by:

```text
User
Caller Workflow
Action Consumer
```

### Variable

Provided as:

```text
Configuration
```

Example:

```text
Input:
environment = production

Variable:
AWS_REGION = ap-south-1
```

---

# Input vs Output

### Input

Data entering a component:

```text
Workflow → Action
```

### Output

Data leaving a component:

```text
Action → Workflow
```

---

# Input vs Environment Variable

Example:

```yaml
with:
  environment: production
```

This is an explicit interface.

Whereas:

```yaml
env:
  ENVIRONMENT: production
```

creates an environment variable.

For reusable components, explicit inputs are generally easier to understand and document.

---

# Inputs and Expressions

Inputs can be used in expressions.

Example:

```yaml
if: ${{ inputs.environment == 'production' }}
```

Or:

```yaml
env:
  IMAGE_TAG: ${{ inputs.version }}
```

---

# Inputs and Conditions

Example:

```yaml
- name: Production Deployment
  if: ${{ inputs.environment == 'production' }}
  run: |
    ./deploy-prod.sh
```

Remember that a condition controls execution; it is not a replacement for production authorization.

---

# Inputs and Matrix

Inputs can be combined with matrix strategies.

Example:

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        required: true
        type: choice
        options:
          - qa
          - uat
```

Then:

```yaml
strategy:
  matrix:
    service:
      - user
      - catalogue
      - cart
```

The workflow can deploy selected environment services according to the design.

---

# Inputs and Environment Variables

Example:

```yaml
env:
  DEPLOYMENT_ENV: ${{ inputs.environment }}
  IMAGE_TAG: ${{ inputs.version }}
```

Then shell commands can use:

```text
$DEPLOYMENT_ENV
$IMAGE_TAG
```

This is useful when multiple commands need the same input.

---

# Inputs and Reusable Workflow

Caller:

```yaml
jobs:

  deploy:

    uses: company/platform-workflows/deploy.yml@v1

    with:
      environment: production
      version: ${{ github.sha }}
```

Reusable workflow:

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
```

---

# Inputs and Custom Action

`action.yml`:

```yaml
name: Deploy

inputs:

  environment:
    description: Target environment
    required: true

  version:
    description: Application version
    required: true

runs:
  using: composite

  steps:

    - name: Deploy
      shell: bash
      run: |
        echo "Environment: ${{ inputs.environment }}"
        echo "Version: ${{ inputs.version }}"
```

---

# Custom Action Input Design

A good input should have:

```text
Clear Name
Description
Required/Optional
Default if appropriate
Expected Format
Validation
```

Example:

```yaml
inputs:

  environment:
    description: Deployment environment
    required: true

  version:
    description: Immutable application version or commit SHA
    required: true
```

---

# Input Naming

Use:

```text
environment
version
service-name
jira-ticket
image-tag
namespace
```

Avoid:

```text
x
value
data
input1
```

Clear names make Actions easier to consume.

---

# Input Defaults

Use defaults carefully.

Good:

```yaml
inputs:

  run-tests:
    required: false
    default: true
```

Potentially dangerous:

```yaml
inputs:

  environment:
    required: false
    default: production
```

Never accidentally make production the default unless there is a strong reason and appropriate protection.

---

# Safe Defaults

Prefer:

```text
qa
```

or:

```text
dry-run
```

for destructive workflows when appropriate.

For production:

```text
Explicit selection
+
Validation
+
Approval
```

is safer.

---

# Dry Run Input

A deployment workflow can provide:

```yaml
on:
  workflow_dispatch:
    inputs:
      dry-run:
        description: Validate without deploying
        required: true
        type: boolean
```

Then:

```yaml
if: ${{ !inputs.dry-run }}
```

can control deployment execution.

---

# Dry Run Flow

```text
Input
 |
 ├── dry-run=true
 |       |
 |       ↓
 |    Validate
 |       |
 |       ↓
 |     Stop
 |
 └── dry-run=false
         |
         ↓
      Validate
         |
         ↓
      Deploy
```

---

# Production Approval

A `dry-run` boolean should not be treated as a production authorization mechanism.

Production should use:

```text
GitHub Environment
Required Reviewers
Change Management
Least Privilege
```

where required.

---

# Input Format Validation

Examples:

```text
Environment → qa|uat|production
SHA → expected commit format
JIRA Ticket → project-number
Service → approved service list
Version → approved version format
```

Validate before execution.

---

# JIRA Ticket Format

Example validation:

```bash
if [[ ! "$JIRA_TICKET" =~ ^[A-Z]+-[0-9]+$ ]]; then
  echo "Invalid JIRA ticket format"
  exit 1
fi
```

Format validation only checks syntax.

It does not prove:

```text
Ticket exists
Ticket is approved
Ticket belongs to correct project
```

Those require API validation.

---

# SHA Validation

A SHA input should be validated against trusted sources.

Conceptually:

```text
Input SHA
    |
    ↓
GitHub
    |
    ↓
Commit Exists?
    |
    ↓
Approved?
```

---

# Service Validation

Example:

```bash
case "$SERVICE" in
  user|catalogue|cart|orders|payment|inventory|notification)
    echo "Valid service"
    ;;
  *)
    echo "Invalid service"
    exit 1
    ;;
esac
```

---

# Environment Validation

Example:

```bash
case "$ENVIRONMENT" in
  qa|sit|uat|production)
    echo "Valid environment"
    ;;
  *)
    echo "Invalid environment"
    exit 1
    ;;
esac
```

---

# Inputs and Security

Inputs are untrusted data.

Treat them carefully.

Potentially dangerous input:

```text
Shell command
File path
Git reference
Cloud resource
URL
Deployment target
```

Never blindly interpolate untrusted values into shell commands.

---

# Command Injection Risk

Bad pattern:

```yaml
run: |
  ./deploy.sh ${{ inputs.environment }}
```

If the input is not properly controlled, shell interpretation can become dangerous.

Safer pattern:

```yaml
env:
  DEPLOYMENT_ENV: ${{ inputs.environment }}

run: |
  ./deploy.sh "$DEPLOYMENT_ENV"
```

And validate the allowed values first.

---

# Input Validation + Quoting

Use:

```bash
"$VARIABLE"
```

instead of:

```bash
$VARIABLE
```

when passing variables to commands where shell word splitting or special characters could cause problems.

---

# Untrusted Pull Request Inputs

Be especially careful when workflow values originate from untrusted pull-request content.

Do not combine:

```text
Untrusted Input
+
Privileged Credentials
+
Privileged Runner
```

without a strong security boundary.

---

# Production Input Security

For production deployment:

```text
User Input
    |
    ↓
Syntax Validation
    |
    ↓
Business Validation
    |
    ↓
Security Validation
    |
    ↓
Approval
    |
    ↓
Deployment
```

---

# Input Auditability

Production inputs should be visible in deployment records where appropriate:

```text
Who
What
When
Environment
Version
JIRA Ticket
Service
```

This improves traceability.

---

# Deployment Input Example

```text
Requested by:
Surendra

Environment:
production

Service:
catalogue

Version:
8a92f31

JIRA:
PROJ-1234

Strategy:
rolling
```

The workflow can use these values for validation and deployment.

---

# Input and Deployment Window

Example:

```text
Input:
environment=production

        |
        ↓

JIRA API
        |
        ↓

Approved CR?
        |
        ↓

Current Time
        |
        ↓

Deployment Window?
        |
        ↓

PASS
```

The workflow should calculate/verify the actual deployment window rather than trusting a user-provided time.

---

# Input and Rollback

A production deployment may accept:

```text
version
```

and use the immutable version for deployment.

Rollback should use a known-good version or Helm revision according to the platform's rollback design.

---

# Input and Helm

Example:

```yaml
env:
  RELEASE_NAME: catalogue
  NAMESPACE: production
  IMAGE_TAG: ${{ inputs.version }}

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

# Input and ECR

Example:

```yaml
env:
  IMAGE_TAG: ${{ inputs.version }}

steps:

  - name: Validate Image
    run: |
      echo "Checking ECR image: $IMAGE_TAG"
```

The workflow should verify that the requested immutable image actually exists before deployment.

---

# Input and ArgoCD

For GitOps:

```text
Input Version
      |
      ↓
Validate ECR Image
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

The input should identify the intended immutable application version.

---

# Input and EKS

Avoid allowing arbitrary cluster names from users in privileged workflows.

Prefer:

```text
Environment
   |
   ↓
Trusted Configuration
   |
   ↓
Cluster
```

instead of:

```text
User Input
   |
   ↓
Arbitrary Cluster
```

---

# Input and Cloud Resources

Never blindly accept arbitrary:

```text
AWS Account
AWS Role
Cluster
Bucket
Security Group
VPC
```

from an untrusted user.

Map controlled inputs to trusted configuration.

---

# Safe Mapping Pattern

Instead of:

```text
Input:
cluster=anything
```

use:

```bash
case "$ENVIRONMENT" in
  qa)
    CLUSTER="qa-cluster"
    ;;
  uat)
    CLUSTER="uat-cluster"
    ;;
  production)
    CLUSTER="prod-cluster"
    ;;
  *)
    exit 1
    ;;
esac
```

The user selects the environment; the workflow determines the trusted infrastructure target.

---

# Inputs and Git Branches

Inputs can select:

```text
branch
tag
SHA
```

But production deployments should prefer immutable references.

Better:

```text
SHA
```

than:

```text
main
```

because a branch can move.

---

# Input and Artifact

A deployment workflow may accept:

```text
artifact version
```

Then:

```text
Validate artifact
      |
      ↓
Deploy
```

Use immutable identifiers where possible.

---

# Input Documentation

Every reusable workflow and custom Action should document:

```text
Input
Description
Required
Type
Default
Allowed Values
Example
Security Considerations
```

Example:

| Input | Type | Required | Example |
|---|---|---|---|
| environment | string | Yes | production |
| version | string | Yes | 8a92f31 |
| dry-run | boolean | No | true |

---

# Good Input Interface

```text
Input
   |
   ├── Clear
   ├── Explicit
   ├── Validated
   ├── Documented
   └── Minimal
```

Avoid hidden configuration dependencies.

---

# Input Interface for Platform Actions

For example:

```text
docker-build
   |
   ├── image-name
   ├── image-tag
   └── dockerfile

security-scan
   |
   ├── image
   └── severity-threshold

jira-validation
   |
   ├── jira-ticket
   ├── version
   └── environment
```

Each Action has a focused interface.

---

# Input Interface for Deployment Action

```text
helm-deploy
   |
   ├── release-name
   ├── chart-path
   ├── namespace
   ├── image-tag
   └── environment
```

The Action should validate values before executing deployment operations.

---

# Inputs and Platform Standardization

For multiple microservices:

```text
user
catalogue
cart
orders
payment
inventory
notification
```

A standardized deployment Action can accept:

```text
service
environment
version
namespace
```

instead of maintaining separate deployment scripts for every service.

---

# Production Platform Workflow

```text
Application Repository
       |
       ↓
Reusable Workflow
       |
       ├── service
       ├── environment
       └── version
       |
       ↓
Custom Actions
       |
       ├── Build
       ├── Security
       ├── JIRA Validation
       └── Deployment
       |
       ↓
ECR
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

# Input Security Checklist

```text
☐ Validate input
☐ Restrict allowed values
☐ Quote shell variables
☐ Avoid command injection
☐ Avoid arbitrary resource selection
☐ Never use inputs for secrets
☐ Do not expose inputs to privileged code unnecessarily
☐ Use immutable versions
☐ Validate production JIRA ticket
☐ Validate approved SHA
☐ Use environment protection
☐ Keep audit information
```

---

# Common Mistakes

### 1. Using secrets as inputs

Use secrets for sensitive values.

### 2. Hardcoding deployment environment

Use controlled inputs when manual selection is required.

### 3. Accepting arbitrary production targets

Map controlled inputs to trusted configuration.

### 4. No input validation

Always validate important inputs.

### 5. Using `latest`

Prefer immutable image tags or digests.

### 6. Passing untrusted input directly into shell commands

Validate and quote values.

### 7. Treating inputs as authorization

Inputs identify what the user wants to do.

They do not authorize the action.

### 8. Making production the default

Require deliberate production selection and protection.

---

# Best Practices

- Use inputs to make workflows reusable.
- Prefer `choice` for controlled environment/service selections.
- Use explicit types.
- Mark critical inputs as required.
- Use safe defaults.
- Validate all deployment-related inputs.
- Treat inputs as untrusted data.
- Quote shell variables.
- Prefer immutable SHAs or image digests.
- Never use inputs to carry secrets.
- Use `with` for Action inputs.
- Use `workflow_call` inputs for reusable workflows.
- Use `workflow_dispatch` inputs for manual workflows.
- Use GitHub Environments for production protection.
- Keep input interfaces small and focused.
- Document every reusable input.
- Audit important production deployment inputs.

---

# Production-Level Input Architecture

```text
                    User / Caller
                         |
                         ↓
                       Inputs
                         |
             ┌───────────┼───────────┐
             ↓           ↓           ↓
        Environment     Version     Service
             |           |           |
             └───────────┼───────────┘
                         ↓
                    Validation
                         |
             ┌───────────┼───────────┐
             ↓           ↓           ↓
          JIRA         SHA         Security
          Check        Check         Check
             |           |           |
             └───────────┼───────────┘
                         ↓
                  GitHub Environment
                         |
                         ↓
                      Approval
                         |
                         ↓
                     Deployment
```

---

# Key Takeaways

Inputs are the interface through which data enters:

```text
Manual Workflows
Reusable Workflows
Custom Actions
```

Remember:

```text
workflow_dispatch
    ↓
Manual User Input

workflow_call
    ↓
Reusable Workflow Input

with
    ↓
Action Input
```

Security principle:

```text
Input
  ≠
Secret

Input
  ≠
Authorization
```

For production:

```text
Input
 ↓
Validate
 ↓
Authorize
 ↓
Deploy
```

The most important principle:

```text
Treat workflow inputs as untrusted data,
validate them strictly,
and never use them as a substitute for secrets or authorization controls.
```

---

# Interview Questions

## Basic

1. What are inputs in GitHub Actions?
2. What is `workflow_dispatch`?
3. What are manual workflow inputs?
4. What is `workflow_call`?
5. What is the `with` keyword?
6. What is the difference between `with` and `env`?
7. What input types are commonly supported?
8. What is a required input?
9. What is a default input?
10. How do you define an input for a custom Action?

## Intermediate

11. What is the difference between `workflow_dispatch` and `workflow_call` inputs?
12. How do you create a choice input?
13. How do you create a boolean input?
14. How do you create an environment input?
15. How do you pass inputs to a reusable workflow?
16. How do you pass inputs to a custom Action?
17. How do you validate inputs?
18. How do you pass an input into an environment variable?
19. How do you use inputs in conditions?
20. How do inputs work with matrix strategies?
21. What is the difference between inputs, variables, secrets, and outputs?
22. How do you safely use an input in a shell command?
23. Why should you avoid using arbitrary user input for cloud resources?

## Advanced / Production

24. Design a production deployment workflow using `workflow_dispatch` inputs.
25. How would you allow users to select QA, UAT, or production safely?
26. How would you validate a production JIRA ticket supplied as an input?
27. How would you validate that the requested SHA is approved?
28. How would you prevent a user from deploying an arbitrary image?
29. How would you prevent command injection through workflow inputs?
30. How would you safely map an environment input to an AWS account and EKS cluster?
31. How would you design inputs for a multi-microservice deployment platform?
32. How would you use inputs with ECR, Helm, ArgoCD, and EKS?
33. How would you design a reusable deployment workflow with environment, service, version, and JIRA ticket inputs?
34. How would you ensure production cannot be deployed merely because the user selected `production`?
35. How would you combine inputs with GitHub Environments and required approvals?
36. How would you design safe defaults for a production workflow?
37. Why should production deployments use immutable SHAs or image digests instead of `latest`?
38. How would you handle untrusted inputs from pull requests?
39. How would you design input validation for a custom JIRA validation Action?
40. How would you design a complete production deployment interface using inputs, variables, secrets, outputs, environments, OIDC, and custom Actions?