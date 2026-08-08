# GitHub Actions Contexts

Contexts are collections of information that GitHub makes available to workflows.

They allow workflows to access information about:

```text
Workflow
Repository
Event
Commit
Branch
Pull Request
Actor
Runner
Environment
Jobs
Steps
Inputs
Secrets
Variables
```

Contexts are commonly used with expressions:

```yaml
${{ github.sha }}
${{ github.ref }}
${{ vars.AWS_REGION }}
${{ secrets.API_TOKEN }}
${{ inputs.environment }}
```

---

# Why Contexts Matter

Contexts allow workflows to dynamically understand their execution environment.

Instead of hardcoding:

```yaml
IMAGE_TAG: abc123
```

you can use:

```yaml
IMAGE_TAG: ${{ github.sha }}
```

This makes the workflow dynamic and reusable.

---

# Context Architecture

```text
GitHub Actions
      |
      ├── github
      ├── env
      ├── vars
      ├── secrets
      ├── inputs
      ├── runner
      ├── job
      ├── jobs
      └── steps
```

Different contexts provide different information.

---

# Expression Syntax

Contexts are normally accessed using:

```yaml
${{ context.property }}
```

Example:

```yaml
${{ github.repository }}
```

Another example:

```yaml
${{ github.sha }}
```

---

# Common Contexts

Important contexts include:

```text
github
env
vars
secrets
inputs
runner
job
jobs
steps
strategy
matrix
needs
```

Not every context is available in every location.

Always use a context according to where GitHub Actions makes it available.

---

# `github` Context

The `github` context contains information about the workflow run and GitHub event.

Examples:

```yaml
${{ github.repository }}
${{ github.sha }}
${{ github.ref }}
${{ github.ref_name }}
${{ github.actor }}
${{ github.event_name }}
```

---

# Repository Name

Example:

```yaml
run: |
  echo "Repository: ${{ github.repository }}"
```

Possible result:

```text
company/catalogue
```

The value contains:

```text
OWNER/REPOSITORY
```

---

# Repository Owner

Example:

```yaml
run: |
  echo "Owner: ${{ github.repository_owner }}"
```

---

# Commit SHA

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
```

This gives the commit SHA associated with the workflow run.

Useful for:

```text
Docker Tags
Traceability
Deployment
Auditing
Rollback
```

---

# Branch Reference

Example:

```yaml
run: |
  echo "Ref: ${{ github.ref }}"
```

Possible value:

```text
refs/heads/main
```

---

# Branch Name

Example:

```yaml
run: |
  echo "Branch: ${{ github.ref_name }}"
```

Possible values:

```text
main
develop
feature/catalogue
```

---

# Tag Name

For a tag workflow:

```yaml
run: |
  echo "Tag: ${{ github.ref_name }}"
```

Possible:

```text
v1.2.0
```

---

# Event Name

Example:

```yaml
run: |
  echo "Event: ${{ github.event_name }}"
```

Possible values:

```text
push
pull_request
workflow_dispatch
schedule
workflow_call
```

---

# Actor

Example:

```yaml
run: |
  echo "Triggered by: ${{ github.actor }}"
```

This identifies the GitHub user or automation actor associated with the event.

---

# Workflow Name

Example:

```yaml
run: |
  echo "Workflow: ${{ github.workflow }}"
```

Useful for logging and diagnostics.

---

# Run ID

Example:

```yaml
run: |
  echo "Run ID: ${{ github.run_id }}"
```

Useful for:

```text
Traceability
Deployment Records
Debugging
API Calls
```

---

# Run Number

Example:

```yaml
run: |
  echo "Run: ${{ github.run_number }}"
```

Useful for generating human-readable build versions.

Example:

```text
1
2
3
4
```

---

# Run Attempt

Example:

```yaml
run: |
  echo "Attempt: ${{ github.run_attempt }}"
```

Useful when a workflow is re-run.

---

# Workflow URL

A workflow run can be associated with its GitHub Actions run URL using the available GitHub context values.

This is useful for:

```text
JIRA
Deployment Records
Notifications
Audit Trails
```

---

# `event` Context

The GitHub event payload is available through:

```yaml
github.event
```

Example:

```yaml
run: |
  echo '${{ github.event }}'
```

Do not blindly print the entire event payload because it may contain large or sensitive information.

---

# Event-Specific Information

Different events provide different data.

For example:

```text
push
 └── commits

pull_request
 └── pull request information

workflow_dispatch
 └── inputs

issues
 └── issue information
```

Access the appropriate fields for the event.

---

# Pull Request Context

For a pull request workflow:

```yaml
run: |
  echo "PR Number: ${{ github.event.pull_request.number }}"
```

Other information can include:

```text
PR Title
PR Author
Base Branch
Head Branch
Labels
Changed Files
```

depending on the event payload.

---

# Pull Request Base Branch

Conceptually:

```yaml
${{ github.event.pull_request.base.ref }}
```

Example:

```text
main
```

---

# Pull Request Head Branch

Conceptually:

```yaml
${{ github.event.pull_request.head.ref }}
```

Example:

```text
feature/catalogue
```

---

# Push Event

A push event can provide commit information.

Example:

```yaml
run: |
  echo "SHA: ${{ github.sha }}"
  echo "Branch: ${{ github.ref_name }}"
```

---

# Manual Workflow Context

For:

```yaml
workflow_dispatch:
```

manual inputs are available through:

```yaml
${{ inputs.environment }}
```

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
          - production
```

Then:

```yaml
run: |
  echo "Environment: ${{ inputs.environment }}"
```

---

# `inputs` Context

The `inputs` context contains values supplied to:

```text
workflow_dispatch
workflow_call
```

Example:

```yaml
${{ inputs.environment }}
```

---

# Input Example

```yaml
on:
  workflow_dispatch:
    inputs:
      version:
        required: true
        type: string

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          echo "Version: ${{ inputs.version }}"
```

---

# `vars` Context

The `vars` context accesses GitHub configuration variables.

Example:

```yaml
run: |
  echo "Region: ${{ vars.AWS_REGION }}"
```

Useful for:

```text
Repository Configuration
Environment Configuration
Organization Configuration
```

---

# `secrets` Context

The `secrets` context accesses secrets.

Example:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

Remember:

```text
Secrets are sensitive.
```

Do not print them.

---

# `env` Context

The `env` context contains environment variables defined in the workflow.

Example:

```yaml
env:
  APP_NAME: catalogue

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Display
        run: |
          echo "${{ env.APP_NAME }}"
```

---

# `env` vs Shell Environment Variable

These are related but have different syntax.

Expression:

```yaml
${{ env.APP_NAME }}
```

Shell:

```bash
$APP_NAME
```

Example:

```yaml
env:
  APP_NAME: catalogue

steps:

  - name: Display
    run: |
      echo "${{ env.APP_NAME }}"
      echo "$APP_NAME"
```

---

# `runner` Context

The `runner` context provides information about the runner executing the job.

Examples include:

```yaml
${{ runner.os }}
${{ runner.arch }}
${{ runner.name }}
```

---

# Runner OS

Example:

```yaml
run: |
  echo "OS: ${{ runner.os }}"
```

Possible:

```text
Linux
Windows
macOS
```

---

# Runner Architecture

Example:

```yaml
run: |
  echo "Architecture: ${{ runner.arch }}"
```

Useful when workflows need architecture-specific behavior.

---

# Runner Name

Example:

```yaml
run: |
  echo "Runner: ${{ runner.name }}"
```

Useful for troubleshooting self-hosted runners.

---

# Runner Temporary Directory

The runner context provides paths useful for workflow execution.

Use GitHub-provided path variables/context rather than hardcoding runner filesystem locations.

---

# `job` Context

The `job` context provides information about the current job.

It can be useful for:

```text
Status
Container
Services
```

depending on where the context is used.

---

# Job Status

GitHub Actions provides status functions such as:

```text
success()
failure()
cancelled()
always()
```

Example:

```yaml
if: ${{ failure() }}
```

This can be used for failure-handling logic.

---

# `steps` Context

The `steps` context provides information about steps that have run.

It is commonly used to access step outputs.

Example:

```yaml
- name: Generate Version
  id: version
  run: |
    echo "value=1.2.3" >> "$GITHUB_OUTPUT"

- name: Display
  run: |
    echo "${{ steps.version.outputs.value }}"
```

---

# Step Output Structure

```text
steps
  |
  └── version
       |
       └── outputs
            |
            └── value
```

Reference:

```yaml
${{ steps.version.outputs.value }}
```

---

# Step Outcome

A step can have an outcome.

Conceptually:

```yaml
${{ steps.test.outcome }}
```

Possible values include states such as:

```text
success
failure
cancelled
skipped
```

This can help distinguish what happened to a step.

---

# Step Conclusion

The `steps` context also exposes conclusion information.

Be careful to distinguish:

```text
outcome
conclusion
```

when working with steps using:

```text
continue-on-error
```

---

# `needs` Context

The `needs` context provides information about jobs that the current job depends on.

Example:

```yaml
jobs:

  build:
    ...

  deploy:

    needs: build

    steps:

      - name: Display
        run: |
          echo "${{ needs.build.result }}"
```

---

# Job Output Through `needs`

Example:

```yaml
jobs:

  build:

    outputs:
      image-tag: ${{ steps.image.outputs.tag }}

  deploy:

    needs: build

    steps:

      - name: Deploy
        run: |
          echo "${{ needs.build.outputs.image-tag }}"
```

---

# `needs.result`

Example:

```yaml
${{ needs.build.result }}
```

Possible result states include:

```text
success
failure
cancelled
skipped
```

This can be used in workflow conditions.

---

# Multiple `needs`

Example:

```yaml
jobs:

  build:
    ...

  security:
    ...

  test:
    ...

  deploy:

    needs:
      - build
      - security
      - test
```

The deploy job can inspect the relevant upstream job results and outputs.

---

# `strategy` Context

The `strategy` context contains information about the job strategy.

It is useful with:

```text
Matrix
Fail-fast
Max Parallel
```

---

# `matrix` Context

The `matrix` context provides the current matrix values.

Example:

```yaml
strategy:
  matrix:
    node:
      - '20'
      - '22'
```

Use:

```yaml
${{ matrix.node }}
```

---

# Matrix Example

```yaml
jobs:

  test:

    runs-on: ubuntu-latest

    strategy:
      matrix:
        node:
          - '20'
          - '22'

    steps:

      - name: Display Version
        run: |
          echo "Node: ${{ matrix.node }}"
```

Each matrix job gets its own value.

---

# Matrix Environment

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
environment:
  name: ${{ matrix.environment }}
```

This can be useful for controlled multi-environment workflows, subject to your deployment design.

---

# `jobs` Context

The `jobs` context is primarily relevant to reusable workflows and certain workflow-level interactions.

It can provide information about jobs and their outputs in supported contexts.

Do not assume every context is available everywhere.

---

# Context Availability

One of the most important concepts:

```text
A context is not automatically available in every part of a workflow.
```

For example:

```text
Some contexts are available in jobs
Some in steps
Some in expressions
Some in reusable workflows
```

Always use the context according to GitHub Actions expression/context availability rules.

---

# Contexts in `if`

Contexts are commonly used in conditions.

Example:

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

Another:

```yaml
if: ${{ github.event_name == 'pull_request' }}
```

Another:

```yaml
if: ${{ inputs.environment == 'production' }}
```

---

# Contexts in `env`

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
  AWS_REGION: ${{ vars.AWS_REGION }}
  ENVIRONMENT: ${{ inputs.environment }}
```

This creates runtime environment variables from contexts.

---

# Contexts in `with`

Example:

```yaml
- name: Setup Node
  uses: actions/setup-node@v4
  with:
    node-version: ${{ vars.NODE_VERSION }}
```

---

# Contexts in `runs-on`

Example:

```yaml
runs-on: ${{ vars.RUNNER_LABEL }}
```

Use carefully.

For production, avoid allowing untrusted input to arbitrarily select privileged runners.

---

# Contexts in `environment`

Example:

```yaml
environment:
  name: ${{ inputs.environment }}
```

This is useful for controlled deployment workflows.

---

# Contexts in Job Names

Example:

```yaml
name: Deploy ${{ inputs.environment }}
```

Dynamic names can improve workflow readability.

---

# Contexts in Step Names

Example:

```yaml
- name: Deploy ${{ inputs.environment }}
  run: |
    echo "Deploying"
```

Useful for clearer workflow logs.

---

# Contexts and Expressions

Contexts are frequently combined with operators.

Example:

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

Operators can include comparisons such as:

```text
==
!=
&&
||
!
```

Expressions are covered in greater detail in:

```text
07-Expressions.md
```

---

# Contexts and String Functions

GitHub Actions expressions support functions.

Examples include:

```text
contains()
startsWith()
endsWith()
format()
join()
toJSON()
fromJSON()
```

Use only where appropriate.

---

# `toJSON()`

`toJSON()` can help inspect structured context data.

Example:

```yaml
run: |
  echo '${{ toJSON(github.event) }}'
```

Be careful:

```text
Event payloads can be large.
Some contexts can contain sensitive information.
```

Do not dump secrets or sensitive data into logs.

---

# Debugging Contexts

For troubleshooting, print only the relevant non-sensitive fields.

Example:

```yaml
run: |
  echo "Repository: ${{ github.repository }}"
  echo "Branch: ${{ github.ref_name }}"
  echo "SHA: ${{ github.sha }}"
  echo "Event: ${{ github.event_name }}"
```

Avoid dumping entire contexts in production workflows.

---

# Context Debugging

Useful debugging information:

```text
Repository
Branch
Commit SHA
Event
Actor
Workflow
Run ID
Runner OS
Runner Architecture
```

This can help identify why a workflow behaved differently.

---

# Contexts and Branch Protection

Example:

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

This can control when a job executes.

But:

```text
Condition
≠
Branch Protection
```

Use GitHub repository protection rules for actual branch governance.

---

# Contexts and Production

A production workflow can use:

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

but this alone should not authorize production deployment.

Use:

```text
GitHub Environment
Required Reviewers
Permissions
Change Management
```

as appropriate.

---

# Contexts and Pull Requests

Example:

```yaml
if: ${{ github.event_name == 'pull_request' }}
```

This allows a step to run only for pull-request workflows.

---

# Contexts and Push

Example:

```yaml
if: ${{ github.event_name == 'push' }}
```

Useful for distinguishing CI behavior.

---

# Contexts and Tags

Example:

```yaml
if: startsWith(github.ref, 'refs/tags/')
```

This can be useful for release workflows.

---

# Contexts and Main Branch

Example:

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

Typical use:

```text
Feature Branch
    ↓
CI

Main
    ↓
Build / Release
```

---

# Contexts and Commit SHA

A strong deployment pattern:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
```

Then:

```text
Git Commit
    ↓
Docker Image
    ↓
ECR
    ↓
Deployment
```

The same SHA provides traceability.

---

# Contexts and Repository

Example:

```yaml
env:
  REPOSITORY: ${{ github.repository }}
```

Useful when calling GitHub APIs.

Example:

```text
OWNER/REPOSITORY
```

---

# GitHub API Example

A workflow may construct a GitHub API endpoint using:

```yaml
${{ github.repository }}
```

Example concept:

```bash
curl \
  "https://api.github.com/repos/${GITHUB_REPOSITORY}/..."
```

Use appropriate authentication and permissions.

---

# Dependabot API Example

Your notes include this pattern:

```bash
curl -s \
  -H "Authorization: Bearer $TOKEN" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${GITHUB_REPOSITORY}/dependabot/alerts?severity=critical&state=open"
```

Then:

```bash
| jq '. | length'
```

can count returned alerts.

Keep the authentication token protected.

---

# Contexts and Security

Some contexts can contain information supplied by users or external systems.

Treat values such as:

```text
Pull Request Title
Issue Title
Commit Message
Branch Name
User Input
Event Payload
```

as potentially untrusted.

Do not directly inject untrusted values into shell commands.

---

# Dangerous Pattern

Avoid:

```yaml
run: |
  echo "${{ github.event.pull_request.title }}"
```

when the value is going to be interpreted by a shell in a security-sensitive way.

Prefer passing data through environment variables and treating it as data:

```yaml
env:
  PR_TITLE: ${{ github.event.pull_request.title }}

run: |
  printf '%s\n' "$PR_TITLE"
```

Still validate and handle untrusted content appropriately.

---

# Context Injection

Context injection can happen when attacker-controlled values are inserted into commands.

Potential sources:

```text
PR Title
Issue Title
Branch Name
Commit Message
Workflow Input
External API Response
```

Safe principle:

```text
Treat dynamic context values as data,
not executable code.
```

---

# Contexts and Secrets

Never expose:

```yaml
${{ secrets.API_TOKEN }}
```

through debugging.

Bad:

```yaml
run: |
  echo '${{ toJSON(secrets) }}'
```

Do not dump secret contexts.

---

# Contexts and OIDC

GitHub context information can be used when designing cloud identity trust policies.

For example:

```text
Repository
Branch
Environment
```

can be part of the identity model.

Conceptually:

```text
GitHub Workflow
      |
      ↓
OIDC Token
      |
      ↓
Cloud Trust Policy
      |
      ↓
IAM Role
```

The trust policy should be restrictive.

---

# Contexts and Environment Protection

Example:

```yaml
environment:
  name: production
```

combined with:

```yaml
${{ inputs.environment }}
```

can connect workflow inputs to a GitHub Environment.

But the environment should provide the actual protection boundary.

---

# Production Context Architecture

```text
                    GitHub Workflow
                           |
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
      github             inputs             vars
        |                  |                  |
        ↓                  ↓                  ↓
 Commit / Event       User Input        Configuration
        |                  |                  |
        └──────────────────┼──────────────────┘
                           ↓
                      Validation
                           |
                           ↓
                    GitHub Environment
                           |
                           ↓
                        Deploy
```

---

# Contexts in a DevSecOps Pipeline

```text
github.sha
    |
    ↓
Image Tag

github.repository
    |
    ↓
ECR / GitHub API

github.event_name
    |
    ↓
Workflow Behavior

inputs.environment
    |
    ↓
Deployment Environment

vars.AWS_REGION
    |
    ↓
AWS Configuration

secrets.JIRA_API_TOKEN
    |
    ↓
JIRA Validation

needs.build.outputs.image-digest
    |
    ↓
Production Promotion
```

---

# Contexts and Your Production Workflow

For your production flow:

```text
workflow_dispatch
      |
      ↓
inputs
 ├── JIRA Ticket
 ├── Version / SHA
 └── Environment
      |
      ↓
github
 ├── Repository
 ├── Commit
 └── Actor
      |
      ↓
JIRA Validation
      |
      ↓
Security / Testing
      |
      ↓
needs outputs
      |
      ↓
Production Environment
      |
      ↓
Deployment
```

---

# Contexts and Microservices

For a microservices platform:

```text
github.repository
        ↓
Repository

github.sha
        ↓
Version

inputs.service
        ↓
Selected Service

inputs.environment
        ↓
Target Environment

vars.AWS_REGION
        ↓
AWS Configuration

needs.build.outputs.image-digest
        ↓
Exact Artifact
```

---

# Contexts and GitOps

```text
github.sha
    |
    ↓
Build Image
    |
    ↓
ECR
    |
    ↓
Output Image Digest
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

Contexts and outputs make the workflow dynamic and traceable.

---

# Contexts and Reusable Workflows

A reusable workflow can receive:

```text
inputs
secrets
```

and access other supported contexts.

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

---

# Contexts and Custom Actions

A custom Action commonly uses:

```text
inputs
env
runner
github
```

depending on its implementation.

Example:

```yaml
with:
  environment: production
```

The Action reads:

```text
inputs.environment
```

---

# Contexts and Composite Actions

Composite Actions commonly receive values through:

```yaml
with:
```

Example:

```yaml
- uses: ./actions/deploy
  with:
    environment: production
    version: ${{ github.sha }}
```

This creates an explicit interface.

---

# Contexts and Matrix Deployments

Example:

```yaml
strategy:
  matrix:
    service:
      - user
      - catalogue
      - cart
```

Then:

```yaml
env:
  SERVICE_NAME: ${{ matrix.service }}
```

Each matrix job receives the correct service.

---

# Contexts and Runner Selection

Example:

```yaml
runs-on: ${{ vars.RUNNER_LABEL }}
```

Use only trusted configuration for privileged runner selection.

Do not allow arbitrary external input to select a production runner.

---

# Contexts and Job Status

Example:

```yaml
- name: Failure Notification
  if: ${{ failure() }}
  run: |
    echo "A previous step failed"
```

Status functions are especially useful for:

```text
Notifications
Cleanup
Diagnostics
Rollback Logic
```

---

# `always()`

Example:

```yaml
- name: Cleanup
  if: ${{ always() }}
  run: |
    ./cleanup.sh
```

This can be useful when cleanup should run regardless of earlier step outcomes.

Be careful with cleanup operations that could destroy evidence or important diagnostic data.

---

# `failure()`

Example:

```yaml
- name: Failure Notification
  if: ${{ failure() }}
  run: |
    echo "Pipeline failed"
```

Useful for:

```text
Notifications
Incident Handling
Diagnostics
```

---

# `cancelled()`

Example:

```yaml
- name: Cancellation Handler
  if: ${{ cancelled() }}
  run: |
    echo "Workflow was cancelled"
```

---

# `success()`

Example:

```yaml
- name: Success Notification
  if: ${{ success() }}
  run: |
    echo "Pipeline succeeded"
```

---

# Contexts and Conditional Deployment

Example:

```yaml
if: >
  ${{
    github.ref == 'refs/heads/main' &&
    needs.tests.result == 'success'
  }}
```

This combines:

```text
GitHub Context
+
Needs Context
```

---

# Contexts and Production Gate

A production job might use:

```yaml
if: >
  ${{
    github.ref == 'refs/heads/main' &&
    needs.e2e.result == 'success'
  }}

environment:
  name: production
```

The `if` condition controls execution.

The environment provides the production protection boundary.

---

# Contexts and Change Management

Example:

```text
inputs.jira-ticket
       |
       ↓
JIRA API
       |
       ↓
Validation Output
       |
       ↓
needs.jira.outputs.approved
       |
       ↓
Production Environment
```

This provides an explicit data flow.

---

# Contexts and Audit

Useful values for deployment records:

```text
github.actor
github.repository
github.workflow
github.run_id
github.run_number
github.sha
github.ref
inputs.environment
inputs.jira-ticket
```

These help answer:

```text
Who deployed?
What was deployed?
Where?
When?
From which commit?
Under which change request?
```

---

# Production Deployment Record

Example:

```text
Repository:
company/catalogue

Commit:
8a92f31...

Actor:
developer

Environment:
production

JIRA:
PROJ-1234

Workflow:
Production Deployment

Run:
1234
```

This creates strong traceability.

---

# Common Context Mistakes

### 1. Using the wrong context

Example:

```text
github.sha
```

when you actually need:

```text
inputs.version
```

Understand where the value comes from.

### 2. Assuming all contexts are available everywhere

Context availability depends on workflow location.

### 3. Dumping entire contexts

This can expose unnecessary information.

### 4. Trusting user-controlled context values

Treat them as untrusted.

### 5. Using conditions as authorization

A condition is not a security boundary.

### 6. Using arbitrary input for runner selection

This can create a privilege escalation risk.

---

# Best Practices

- Learn the purpose of each context.
- Use the narrowest relevant context.
- Use `github.sha` for immutable commit identification.
- Use `inputs` for explicit workflow inputs.
- Use `vars` for non-sensitive configuration.
- Use `secrets` for sensitive values.
- Use `steps` for step outputs.
- Use `needs` for cross-job outputs and results.
- Use `matrix` for matrix values.
- Use `runner` for runner information.
- Treat event data as potentially untrusted.
- Never dump secrets or sensitive contexts.
- Validate dynamic values.
- Quote shell variables.
- Use GitHub Environments for production protection.
- Use OIDC with restrictive trust policies.
- Keep audit metadata such as SHA, actor, run ID, and JIRA ticket.

---

# Context Cheat Sheet

| Context | Purpose |
|---|---|
| `github` | Workflow and repository/event information |
| `env` | Environment variables |
| `vars` | Configuration variables |
| `secrets` | Sensitive values |
| `inputs` | Workflow inputs |
| `runner` | Runner information |
| `job` | Current job information |
| `jobs` | Job information in supported reusable-workflow contexts |
| `steps` | Step information and outputs |
| `needs` | Dependencies, results, and job outputs |
| `strategy` | Strategy information |
| `matrix` | Current matrix values |

---

# Important Context Examples

```yaml
${{ github.sha }}
```

Current workflow commit.

```yaml
${{ github.repository }}
```

Repository.

```yaml
${{ github.ref_name }}
```

Branch/tag name.

```yaml
${{ inputs.environment }}
```

Workflow input.

```yaml
${{ vars.AWS_REGION }}
```

Configuration variable.

```yaml
${{ secrets.API_TOKEN }}
```

Secret.

```yaml
${{ matrix.service }}
```

Matrix value.

```yaml
${{ needs.build.outputs.image-tag }}
```

Output from a required job.

```yaml
${{ steps.build.outputs.image }}
```

Output from a previous step.

---

# Production Context Flow

```text
                     GitHub Event
                          |
                          ↓
                     github Context
                          |
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       inputs           vars            secrets
          |               |               |
          └───────────────┼───────────────┘
                          ↓
                      Workflow
                          |
              ┌───────────┴───────────┐
              ↓                       ↓
           Build                  Security
              |                       |
              ↓                       ↓
           outputs                 outputs
              |                       |
              └───────────┬───────────┘
                          ↓
                        needs
                          |
                          ↓
                    Production Job
                          |
                     environment
                          |
                          ↓
                       ArgoCD
                          |
                          ↓
                         EKS
```

---

# Key Takeaways

Contexts provide dynamic information to GitHub Actions workflows.

Remember:

```text
github
 ↓
GitHub event / repository information

inputs
 ↓
Workflow inputs

vars
 ↓
Configuration

secrets
 ↓
Sensitive data

env
 ↓
Environment variables

steps
 ↓
Step outputs

needs
 ↓
Job outputs and results

matrix
 ↓
Matrix values

runner
 ↓
Runner information
```

The most important security principle:

```text
Context values can come from users,
events, branches, or external data.

Treat dynamic values as data,
validate them,
and never blindly execute them.
```

For production workflows:

```text
Contexts
   ↓
Inputs
   ↓
Validation
   ↓
Outputs
   ↓
Production Environment
   ↓
Approval
   ↓
Deployment
```

---

# Interview Questions

## Basic

1. What are contexts in GitHub Actions?
2. Why are contexts used?
3. What is the `github` context?
4. What is the `env` context?
5. What is the `vars` context?
6. What is the `secrets` context?
7. What is the `inputs` context?
8. What is the `runner` context?
9. What is the `steps` context?
10. What is the `needs` context?

## Intermediate

11. What is the difference between `github.ref` and `github.ref_name`?
12. How do you get the current commit SHA?
13. How do you get the repository name?
14. How do you access a workflow input?
15. How do you access a repository variable?
16. How do you access a secret?
17. How do you access a step output?
18. How do you access a job output?
19. How do you access matrix values?
20. How do you check which event triggered a workflow?
21. What is the difference between `env` and `vars`?
22. What is the difference between `inputs` and `vars`?
23. What is the difference between `steps` and `needs`?
24. How do you use contexts in an `if` condition?
25. How do you use contexts in `with`?
26. How do you use contexts in environment variables?

## Advanced / Production

27. Design a production deployment workflow using `github`, `inputs`, `vars`, `secrets`, `steps`, and `needs`.
28. How would you use `github.sha` for immutable ECR image tagging?
29. How would you pass the image digest from build to production using contexts and outputs?
30. How would you use `inputs.environment` with GitHub Environments?
31. How would you prevent arbitrary inputs from selecting privileged production runners?
32. How would you secure context values originating from pull requests?
33. Explain context injection and command injection in GitHub Actions.
34. How would you safely handle a pull request title inside a shell command?
35. How would you use contexts in a GitOps workflow with ArgoCD and EKS?
36. How would you use `github.repository` when calling the GitHub API?
37. How would you use `github.event` to process Dependabot alerts?
38. How would you combine `needs` outputs with production environment protection?
39. How would you use contexts to create deployment audit records?
40. How would you use OIDC and GitHub context information to restrict AWS IAM role assumption?
41. How would you design contexts for a multi-microservice platform?
42. How would you prevent secrets from being exposed while debugging contexts?
43. How would you use `failure()`, `success()`, `cancelled()`, and `always()` in production workflows?
44. How would you distinguish workflow conditions from actual production authorization?
45. Design an enterprise-grade GitHub Actions architecture using contexts, inputs, variables, secrets, outputs, reusable workflows, GitHub Environments, OIDC, ArgoCD, and EKS.