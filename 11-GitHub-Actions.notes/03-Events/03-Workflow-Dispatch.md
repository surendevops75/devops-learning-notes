# Workflow Dispatch

The `workflow_dispatch` event allows a GitHub Actions workflow to be triggered **manually** from the GitHub Actions interface.

This is especially important for enterprise environments where certain operations should not happen automatically.

Typical use cases include:

- Production deployments
- Rollbacks
- Manual releases
- Environment deployments
- Database maintenance
- Infrastructure operations
- Emergency fixes
- Operational tasks

---

# Why Use workflow_dispatch?

A `push` event is automatic.

A `workflow_dispatch` event requires a user to intentionally start the workflow.

```text
Automatic

git push

↓

Workflow Starts
```

Compared with:

```text
Manual

Developer / DevOps Engineer

↓

Select Workflow

↓

Provide Inputs

↓

Click Run Workflow

↓

Workflow Starts
```

This makes `workflow_dispatch` particularly useful for controlled production operations.

---

# Basic Syntax

```yaml
name: Manual Workflow

on:
  workflow_dispatch:

jobs:
  hello:
    runs-on: ubuntu-latest

    steps:
      - name: Run Manual Job
        run: echo "Workflow started manually"
```

The workflow appears in the **Actions** section of the GitHub repository.

A user can select the workflow and manually start it.

---

# Execution Flow

```text
User

↓

GitHub Actions

↓

Select Workflow

↓

Click Run Workflow

↓

Runner Allocated

↓

Job Starts

↓

Steps Execute

↓

Workflow Completed
```

---

# Manual Workflow Inputs

One of the most useful features of `workflow_dispatch` is accepting input from the user.

Example:

```yaml
name: Deploy Application

on:
  workflow_dispatch:
    inputs:
      environment:
        description: "Select environment"
        required: true
        type: choice
        options:
          - qa
          - sit
          - uat
          - prod
```

When the user starts the workflow, GitHub displays an environment selection.

```text
Run Workflow

Environment:

[ QA  ▼ ]

```

---

# Common Enterprise Inputs

Based on your deployment process, manual workflows can accept values such as:

- Environment
- JIRA ticket number
- Commit SHA
- Version
- Deployment type
- Rollback version

Example:

```yaml
on:
  workflow_dispatch:
    inputs:

      environment:
        description: "Deployment Environment"
        required: true
        type: choice
        options:
          - qa
          - sit
          - uat
          - prod

      jira_ticket:
        description: "JIRA Change Request"
        required: true
        type: string

      version:
        description: "Commit SHA or release version"
        required: true
        type: string
```

---

# Input Types

GitHub Actions supports different input types for manual workflows.

Common types include:

- `string`
- `choice`
- `boolean`
- `environment`

---

# String Input

A string input allows the user to enter text.

```yaml
jira_ticket:
  description: "JIRA Ticket Number"
  required: true
  type: string
```

Example input:

```text
CR-12345
```

---

# Choice Input

A choice input allows the user to select from predefined options.

```yaml
environment:
  description: "Environment"
  required: true
  type: choice
  options:
    - qa
    - sit
    - uat
    - prod
```

This prevents users from entering unsupported environment names.

---

# Boolean Input

A Boolean input allows the user to select true or false.

Example:

```yaml
skip_tests:
  description: "Skip tests"
  required: true
  default: false
  type: boolean
```

For production workflows, bypass options such as this should be carefully controlled.

---

# Environment Input

A workflow can request a GitHub Environment.

```yaml
environment:
  description: "Deployment Environment"
  required: true
  type: environment
```

This can be combined with protected environments and required reviewers.

---

# Using Inputs

Inputs can be accessed through the `inputs` context.

Example:

```yaml
- name: Display Environment
  run: echo "Deploying to ${{ inputs.environment }}"
```

If the user selected:

```text
prod
```

the command receives:

```text
Deploying to prod
```

---

# Complete Manual Deployment Example

```yaml
name: Application Deployment

on:
  workflow_dispatch:
    inputs:

      environment:
        description: "Deployment Environment"
        required: true
        type: choice
        options:
          - qa
          - sit
          - uat
          - prod

      version:
        description: "Version / Commit SHA"
        required: true
        type: string

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Display Deployment Details
        run: |
          echo "Environment: ${{ inputs.environment }}"
          echo "Version: ${{ inputs.version }}"

      - name: Deploy
        run: |
          echo "Deploying ${{ inputs.version }}"
          echo "Target: ${{ inputs.environment }}"
```

---

# Environment Selection

Your enterprise deployment process can use:

```text
workflow_dispatch

↓

Select Environment

├── QA
├── SIT
├── UAT
└── PROD
```

The same workflow can therefore be reused for multiple environments.

---

# Production Deployment Pattern

For production, the workflow should not blindly deploy after receiving inputs.

A controlled workflow should validate the request first.

```text
workflow_dispatch

↓

Input JIRA Ticket

↓

Input Commit SHA

↓

Select PROD

↓

Validate JIRA

↓

Validate Commit

↓

Validate Deployment Window

↓

Validate Required Checks

↓

Production Approval

↓

Deploy
```

This is much safer than:

```text
workflow_dispatch

↓

Deploy PROD
```

---

# JIRA Validation

Your notes describe a production process where the workflow receives a JIRA ticket number and calls the JIRA API to validate the change request.

The conceptual flow is:

```text
User

↓

Enter JIRA Ticket

↓

GitHub Actions

↓

Call JIRA API

↓

Read Ticket Details

↓

Check Status

↓

Check Deployment Window

↓

Continue / Stop
```

For example:

```text
JIRA Ticket

↓

Status = Approved?

↓

YES → Continue

NO  → Stop
```

The exact JIRA API implementation depends on the organization's JIRA setup and authentication method.

---

# Commit SHA Validation

The production workflow can also require a specific commit SHA.

```text
User Input

↓

Commit SHA

↓

Validate Commit

↓

Check CI Status

↓

Approved?

↓

Continue
```

This provides traceability between:

```text
Source Code

↓

CI Validation

↓

Artifact

↓

Production Deployment
```

---

# Production Deployment Window

Your notes include a deployment-window validation.

The workflow can conceptually perform:

```text
Input JIRA Ticket

↓

Retrieve Change Details

↓

Read Deployment Window

↓

Current Time

↓

Inside Approved Window?

       /       \
     YES        NO
      |          |
      ↓          ↓
   Deploy       Stop
```

This prevents accidental production deployments outside the approved change window.

---

# Enterprise Production Workflow

A production deployment workflow based on your deployment process can look like this:

```text
workflow_dispatch

↓

Input

├── JIRA Ticket
├── Commit SHA
└── Environment

↓

JIRA API Validation

↓

Check Change Request

↓

Check Approval Status

↓

Check Deployment Window

↓

Check Commit Status

↓

Check Scan Results

↓

Check Testing Results

↓

Production Approval

↓

Deploy

↓

Smoke Tests

↓

Developer Sanity Checks

↓

Monitoring
```

This creates multiple control points before production deployment.

---

# QA / SIT / UAT Promotion

Manual deployment can also be used for environment promotion.

```text
Build Artifact

↓

workflow_dispatch

↓

QA

↓

Smoke Tests

↓

SIT

↓

Integration Tests

↓

UAT

↓

E2E Tests

↓

Production Approval

↓

PROD
```

The same artifact should be promoted between environments rather than rebuilding the application for every environment.

---

# Rollback Workflow

`workflow_dispatch` is also useful for controlled rollback.

Example:

```yaml
name: Production Rollback

on:
  workflow_dispatch:
    inputs:
      version:
        description: "Version to rollback to"
        required: true
        type: string

jobs:
  rollback:
    runs-on: self-hosted

    steps:
      - name: Rollback
        run: |
          echo "Rolling back to ${{ inputs.version }}"
```

Execution:

```text
Production Incident

↓

Select Rollback Workflow

↓

Enter Known-Good Version

↓

Approval

↓

Helm Rollback

↓

Health Check

↓

Monitoring
```

Your existing production process uses Helm automatic rollbacks, but a manual rollback workflow can still be useful for controlled operational recovery.

---

# Concurrency for Production

Manual workflows can also use concurrency to prevent multiple production deployments from running simultaneously.

Example:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

Execution:

```text
Deployment A

↓

Production Lock

↓

Deploying
```

Meanwhile:

```text
Deployment B

↓

Waiting
```

This prevents competing production deployments.

---

# Permissions

Production workflows should use restrictive permissions.

Example:

```yaml
permissions:
  contents: read
```

Additional permissions should be granted only when required.

---

# Production Example

A simplified enterprise workflow:

```yaml
name: Production Deployment

on:
  workflow_dispatch:
    inputs:

      jira_ticket:
        description: "JIRA Change Request"
        required: true
        type: string

      version:
        description: "Commit SHA"
        required: true
        type: string

      environment:
        description: "Environment"
        required: true
        type: environment

permissions:
  contents: read

concurrency:
  group: production
  cancel-in-progress: false

jobs:

  validate:

    runs-on: ubuntu-latest

    steps:

      - name: Validate Request
        run: |
          echo "JIRA: ${{ inputs.jira_ticket }}"
          echo "Version: ${{ inputs.version }}"
          echo "Environment: ${{ inputs.environment }}"

      - name: Validate Commit
        run: echo "Checking commit status"

      - name: Validate Deployment Window
        run: echo "Checking deployment window"

  deploy:

    needs: validate

    runs-on: self-hosted

    environment: ${{ inputs.environment }}

    steps:

      - name: Deploy
        run: |
          echo "Deploying version ${{ inputs.version }}"

      - name: Smoke Test
        run: echo "Running smoke tests"
```

This example demonstrates an important enterprise pattern:

```text
Manual Trigger

↓

Inputs

↓

Validation

↓

Approval / Environment Protection

↓

Deployment

↓

Smoke Test
```

---

# Production Troubleshooting

## Scenario 1 - workflow_dispatch Does Not Appear

Check:

```text
1. Workflow file exists.
2. Workflow is inside .github/workflows/.
3. YAML syntax is valid.
4. Workflow is present on the selected branch.
5. Workflow is enabled.
```

---

## Scenario 2 - Input Is Empty

Check:

```text
1. Input name.
2. Required setting.
3. Input context.
4. Workflow syntax.
```

Use:

```yaml
${{ inputs.environment }}
```

instead of using an incorrect input name.

---

## Scenario 3 - Wrong Environment Selected

Use a `choice` or `environment` input instead of allowing arbitrary environment strings.

Prefer:

```yaml
type: choice

options:
  - qa
  - sit
  - uat
  - prod
```

This reduces input mistakes.

---

## Scenario 4 - Two Production Deployments Start Together

Use concurrency:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

This ensures that only one production deployment can execute at a time.

---

# Best Practices

- Use `workflow_dispatch` for controlled operational workflows.
- Use typed inputs where possible.
- Use environment protection for production.
- Validate JIRA/change requests before deployment.
- Validate the exact commit SHA being deployed.
- Validate the deployment window.
- Use concurrency for production deployments.
- Keep rollback workflows available.
- Require appropriate approvals.
- Use least-privilege permissions.

---

# Common Mistakes

- Allowing arbitrary production input values.
- Deploying immediately without validation.
- Storing credentials directly in workflow inputs.
- Allowing multiple production deployments simultaneously.
- Not validating the commit SHA.
- Not checking the change request.
- Not enforcing production approvals.
- Using manual deployment without a rollback strategy.

---

# Summary

`workflow_dispatch` provides a controlled way to start GitHub Actions workflows manually.

It becomes especially powerful when combined with:

- Inputs
- Environments
- Approvals
- Concurrency
- Commit SHA validation
- JIRA change requests
- Deployment-window checks
- Smoke tests
- Rollback workflows

For enterprise production deployments, a strong pattern is:

```text
workflow_dispatch

↓

Inputs

↓

JIRA Validation

↓

Commit Validation

↓

Deployment Window

↓

Approval

↓

Deploy

↓

Smoke Tests

↓

Monitoring
```

This provides controlled, traceable, and auditable production deployment.

---

# Interview Questions

## Basic

1. What is `workflow_dispatch`?
2. How do you manually trigger a GitHub Actions workflow?
3. What are workflow inputs?
4. What input types can be used?
5. How do you access workflow inputs?

## Intermediate

6. Why is `workflow_dispatch` useful for production deployments?
7. How do you restrict users to predefined environments?
8. How can concurrency prevent multiple production deployments?
9. How can GitHub Environments be combined with manual workflows?
10. Why should the commit SHA be validated before production deployment?

## Advanced

11. Design a production deployment workflow that accepts a JIRA ticket, commit SHA, and environment as inputs.
12. Explain how you would validate a JIRA change request before allowing production deployment.
13. Design a rollback workflow using `workflow_dispatch` and a known-good application version.
14. A developer manually starts a production workflow with the wrong commit SHA. How would you design validation to prevent the deployment?
15. Design an enterprise `workflow_dispatch` process that validates the JIRA approval, deployment window, CI status, security scan results, testing results, and production approval before deploying.