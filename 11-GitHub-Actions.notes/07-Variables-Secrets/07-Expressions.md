# GitHub Actions Expressions

Expressions allow GitHub Actions workflows to dynamically evaluate values and make decisions.

They are commonly used for:

```text
Conditions
Variables
Inputs
Contexts
Outputs
Job Control
Dynamic Configuration
```

Basic syntax:

```yaml
${{ expression }}
```

Example:

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

---

# Why Expressions Matter

Without expressions, workflows would contain many hardcoded values.

Instead of:

```yaml
run: deploy-production.sh
```

only when manually decided elsewhere, expressions allow the workflow to determine:

```text
Which branch?
Which environment?
Which event?
Which result?
Which input?
Which output?
```

Example:

```yaml
if: ${{ inputs.environment == 'production' }}
```

---

# Expression Syntax

Basic:

```yaml
${{ github.sha }}
```

Comparison:

```yaml
${{ github.ref == 'refs/heads/main' }}
```

Boolean logic:

```yaml
${{
  github.ref == 'refs/heads/main' &&
  needs.test.result == 'success'
}}
```

---

# Expressions and Contexts

Expressions commonly access contexts:

```yaml
${{ github.repository }}
${{ github.sha }}
${{ inputs.environment }}
${{ vars.AWS_REGION }}
${{ secrets.API_TOKEN }}
${{ matrix.service }}
${{ needs.build.outputs.image-tag }}
${{ steps.version.outputs.value }}
```

---

# Expression Locations

Expressions can be used in many workflow fields, including:

```text
if
env
with
name
runs-on
environment
outputs
strategy
```

Context availability depends on where the expression is evaluated.

---

# String Comparison

Example:

```yaml
if: ${{ inputs.environment == 'production' }}
```

Another:

```yaml
if: ${{ github.ref_name == 'main' }}
```

Not equal:

```yaml
if: ${{ inputs.environment != 'production' }}
```

---

# Boolean Operators

Common operators:

```text
&&   AND
||   OR
!    NOT
```

Example:

```yaml
if: >
  ${{
    github.ref_name == 'main' &&
    github.event_name == 'push'
  }}
```

---

# AND Operator

Example:

```yaml
if: ${{ github.ref_name == 'main' && github.event_name == 'push' }}
```

Both conditions must be true.

```text
Condition A → TRUE
Condition B → TRUE

Result → TRUE
```

---

# OR Operator

Example:

```yaml
if: >
  ${{
    github.ref_name == 'main' ||
    github.ref_name == 'release'
  }}
```

At least one condition must be true.

---

# NOT Operator

Example:

```yaml
if: ${{ !cancelled() }}
```

This means:

```text
Workflow has not been cancelled
```

---

# Parentheses

Use parentheses to make complex logic clear.

Example:

```yaml
if: >
  ${{
    (github.ref_name == 'main' || github.ref_name == 'release') &&
    needs.test.result == 'success'
  }}
```

---

# Equality

Example:

```yaml
${{ inputs.environment == 'qa' }}
```

Example:

```yaml
${{ needs.security.result == 'success' }}
```

---

# Inequality

Example:

```yaml
${{ inputs.environment != 'production' }}
```

Useful when different behavior is required for production.

---

# Comparing Outputs

Example:

```yaml
if: ${{ needs.security.outputs.passed == 'true' }}
```

The output is compared with the expected value.

---

# Comparing Inputs

Example:

```yaml
if: ${{ inputs.run-tests == true }}
```

For boolean workflow inputs, use the appropriate boolean value rather than treating it as an arbitrary string.

---

# Comparing Matrix Values

Example:

```yaml
if: ${{ matrix.service == 'catalogue' }}
```

This can be useful when one matrix entry requires special handling.

---

# Comparing Branches

Example:

```yaml
if: ${{ github.ref_name == 'main' }}
```

Or:

```yaml
if: ${{ github.ref == 'refs/heads/main' }}
```

---

# Comparing Events

Example:

```yaml
if: ${{ github.event_name == 'pull_request' }}
```

Another:

```yaml
if: ${{ github.event_name == 'workflow_dispatch' }}
```

---

# Production Condition

Example:

```yaml
if: >
  ${{
    github.ref_name == 'main' &&
    inputs.environment == 'production'
  }}
```

This ensures both conditions are satisfied before the step runs.

Remember:

```text
Condition ≠ Authorization
```

Use GitHub Environment protection for production authorization.

---

# `contains()`

`contains()` checks whether a value contains another value.

Example:

```yaml
if: ${{ contains(github.ref, 'release/') }}
```

Possible matching references:

```text
refs/heads/release/v1
refs/heads/release/v2
```

---

# `startsWith()`

Example:

```yaml
if: ${{ startsWith(github.ref, 'refs/tags/') }}
```

Useful for detecting tag references.

---

# `endsWith()`

Example:

```yaml
if: ${{ endsWith(github.ref_name, '-release') }}
```

---

# `format()`

`format()` can construct strings.

Example:

```yaml
env:
  IMAGE_NAME: ${{ format('{0}/{1}', github.repository_owner, 'catalogue') }}
```

This can be useful for generating dynamic names.

---

# `join()`

`join()` can combine values from arrays.

Example concept:

```yaml
${{ join(matrix.service, ',') }}
```

Use it when working with supported array values.

---

# `toJSON()`

`toJSON()` converts supported structured data into JSON.

Example:

```yaml
run: |
  echo '${{ toJSON(github.event) }}'
```

Be careful with:

```text
Large payloads
Sensitive information
Logs
Untrusted data
```

Do not dump sensitive contexts into logs.

---

# `fromJSON()`

`fromJSON()` converts JSON into a value that expressions can work with.

It can be useful for:

```text
Dynamic Matrix
Numbers
Structured Configuration
```

---

# Dynamic Matrix with JSON

Conceptually:

```yaml
strategy:
  matrix: ${{ fromJSON(needs.prepare.outputs.matrix) }}
```

A previous job can generate the matrix definition.

Example flow:

```text
Prepare
  |
  ↓
JSON Output
  |
  ↓
fromJSON()
  |
  ↓
Matrix
```

---

# Dynamic Matrix Example

```yaml
jobs:

  prepare:

    runs-on: ubuntu-latest

    outputs:
      matrix: ${{ steps.matrix.outputs.value }}

    steps:

      - name: Create Matrix
        id: matrix
        run: |
          echo 'value={"service":["user","catalogue","cart"]}' >> "$GITHUB_OUTPUT"

  test:

    needs: prepare

    runs-on: ubuntu-latest

    strategy:
      matrix: ${{ fromJSON(needs.prepare.outputs.matrix) }}

    steps:

      - name: Test
        run: |
          echo "Testing ${{ matrix.service }}"
```

---

# Status Functions

GitHub Actions provides status functions such as:

```text
success()
failure()
cancelled()
always()
```

These are commonly used in expressions.

---

# `success()`

Example:

```yaml
if: ${{ success() }}
```

Runs when the relevant preceding execution state is successful.

---

# `failure()`

Example:

```yaml
if: ${{ failure() }}
```

Useful for:

```text
Failure Notification
Diagnostics
Incident Handling
```

---

# `cancelled()`

Example:

```yaml
if: ${{ cancelled() }}
```

Useful when workflow execution was cancelled.

---

# `always()`

Example:

```yaml
if: ${{ always() }}
```

Useful for cleanup or reporting that should run regardless of earlier outcomes.

Use carefully for destructive operations.

---

# `success()` vs Job Result

Example:

```yaml
if: ${{ needs.build.result == 'success' }}
```

This explicitly checks the result of a particular job.

Whereas:

```yaml
if: ${{ success() }}
```

uses the workflow status function semantics for the current execution context.

---

# `failure()` for Notifications

Example:

```yaml
- name: Failure Notification
  if: ${{ failure() }}
  run: |
    echo "Pipeline failed"
```

A real implementation could call:

```text
Slack
Email
JIRA
Incident Management
```

without exposing credentials.

---

# Combining Status and Context

Example:

```yaml
if: >
  ${{
    failure() &&
    github.ref_name == 'main'
  }}
```

This can trigger special failure handling for the main branch.

---

# `needs` and Expressions

Example:

```yaml
if: ${{ needs.build.result == 'success' }}
```

Another:

```yaml
if: ${{ needs.security.outputs.passed == 'true' }}
```

---

# Multiple `needs` Conditions

Example:

```yaml
if: >
  ${{
    needs.build.result == 'success' &&
    needs.security.result == 'success' &&
    needs.test.result == 'success'
  }}
```

All required jobs must succeed.

---

# Production Gate Expression

A production job may use:

```yaml
if: >
  ${{
    needs.build.result == 'success' &&
    needs.security.result == 'success' &&
    needs.e2e.result == 'success'
  }}
```

Then:

```yaml
environment:
  name: production
```

The expression controls whether the job runs; the environment provides production protection.

---

# JIRA Approval Expression

Example:

```yaml
if: ${{ needs.jira.outputs.approved == 'true' }}
```

Flow:

```text
JIRA Validation
      |
      ↓
approved=true
      |
      ↓
Expression
      |
      ↓
Production Job
```

The actual approval should be obtained from the JIRA API and combined with proper production controls.

---

# SHA Validation Expression

Example:

```yaml
if: >
  ${{
    needs.validation.outputs.approved_sha ==
    github.sha
  }}
```

This can help ensure the workflow promotes the expected commit.

---

# Environment Expression

Example:

```yaml
environment:
  name: ${{ inputs.environment }}
```

This dynamically selects the GitHub Environment based on a controlled workflow input.

---

# Conditional Deployment

Example:

```yaml
- name: Deploy QA
  if: ${{ inputs.environment == 'qa' }}
  run: |
    ./deploy-qa.sh
```

And:

```yaml
- name: Deploy Production
  if: ${{ inputs.environment == 'production' }}
  run: |
    ./deploy-prod.sh
```

For larger systems, reusable workflows or deployment Actions may be cleaner than maintaining many conditional steps.

---

# Better Environment Mapping

Instead of:

```yaml
run: ./deploy-${{ inputs.environment }}.sh
```

use controlled logic.

Example:

```bash
case "$ENVIRONMENT" in
  qa)
    ./deploy-qa.sh
    ;;
  uat)
    ./deploy-uat.sh
    ;;
  production)
    ./deploy-prod.sh
    ;;
  *)
    echo "Invalid environment"
    exit 1
    ;;
esac
```

---

# Expressions and Variables

Example:

```yaml
env:
  REGION: ${{ vars.AWS_REGION }}
```

Then:

```yaml
if: ${{ vars.AWS_REGION == 'ap-south-1' }}
```

Use variables for configuration rather than secrets.

---

# Expressions and Secrets

Example:

```yaml
env:
  TOKEN: ${{ secrets.API_TOKEN }}
```

Avoid using secrets in conditions unless the specific comparison is appropriate and secure.

Never print:

```yaml
${{ secrets.API_TOKEN }}
```

---

# Expressions and Inputs

Example:

```yaml
if: ${{ inputs.environment == 'production' }}
```

Another:

```yaml
env:
  VERSION: ${{ inputs.version }}
```

---

# Expressions and Step Outputs

Example:

```yaml
if: ${{ steps.scan.outputs.status == 'passed' }}
```

---

# Expressions and Job Outputs

Example:

```yaml
if: ${{ needs.scan.outputs.status == 'passed' }}
```

---

# Expressions and Matrix

Example:

```yaml
if: ${{ matrix.service != 'notification' }}
```

This can exclude special handling from a matrix job.

---

# Expressions and Runner

Example:

```yaml
if: ${{ runner.os == 'Linux' }}
```

Useful when workflow behavior differs by operating system.

---

# Expression Truthiness

Expressions evaluate values as true/false depending on their type and value.

Be careful when comparing:

```text
true
"true"
false
"false"
0
"0"
```

Do not assume strings and booleans are identical.

---

# Boolean Input

If:

```yaml
type: boolean
```

then use:

```yaml
if: ${{ inputs.run-tests }}
```

rather than unnecessarily converting it into a string.

---

# Output Values

Outputs are commonly represented as strings.

Therefore:

```yaml
if: ${{ steps.test.outputs.passed == 'true' }}
```

may be appropriate when the output was written as:

```bash
echo "passed=true" >> "$GITHUB_OUTPUT"
```

---

# Expression Precedence

When combining multiple operators:

```text
!
&&
||
```

use parentheses where clarity matters.

Example:

```yaml
if: >
  ${{
    (github.ref_name == 'main' || github.ref_name == 'release') &&
    needs.test.result == 'success'
  }}
```

This is easier to reason about than a long ungrouped expression.

---

# Expression Formatting

For simple conditions:

```yaml
if: ${{ github.ref_name == 'main' }}
```

For complex conditions:

```yaml
if: >
  ${{
    github.ref_name == 'main' &&
    needs.build.result == 'success' &&
    needs.security.result == 'success'
  }}
```

Readable expressions are easier to maintain.

---

# Expressions in `if`

GitHub Actions supports expression evaluation in `if`.

Example:

```yaml
if: ${{ github.event_name == 'pull_request' }}
```

For `if`, GitHub Actions also supports expression syntax in contexts where the expression is expected.

Keep the explicit `${{ }}` form when it improves clarity.

---

# Expressions in `env`

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
  SERVICE: ${{ inputs.service }}
  REGION: ${{ vars.AWS_REGION }}
```

---

# Expressions in `with`

Example:

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: ${{ vars.NODE_VERSION }}
```

---

# Expressions in `runs-on`

Example:

```yaml
runs-on: ${{ vars.RUNNER_LABEL }}
```

Only use trusted configuration for privileged runner selection.

---

# Expressions in Outputs

A job output can reference a step output:

```yaml
outputs:
  image-tag: ${{ steps.image.outputs.tag }}
```

---

# Expression Data Flow

```text
Context
   |
   ↓
Expression
   |
   ↓
Condition / Value
   |
   ↓
Workflow Behavior
```

Example:

```text
github.ref
   |
   ↓
Expression
   |
   ↓
main?
   |
   ↓
Production Path
```

---

# Expression Security

Expressions can become dangerous when dynamic values are inserted into shell commands.

Potentially untrusted sources:

```text
PR Title
Issue Title
Branch Name
Commit Message
Workflow Input
External API Data
```

Treat them as data.

---

# Unsafe Pattern

Avoid:

```yaml
run: |
  echo "Deploying ${{ github.event.pull_request.title }}"
```

when the value may contain shell-sensitive content.

Safer:

```yaml
env:
  PR_TITLE: ${{ github.event.pull_request.title }}

run: |
  printf '%s\n' "$PR_TITLE"
```

---

# Command Injection

The dangerous pattern is:

```text
Untrusted Data
      |
      ↓
Expression
      |
      ↓
Shell Command
```

Safe principle:

```text
Do not let data become code.
```

---

# Expression Security with Inputs

Bad:

```yaml
run: |
  kubectl apply -f ${{ inputs.file }}
```

Better:

```yaml
env:
  FILE: ${{ inputs.file }}

run: |
  kubectl apply -f "$FILE"
```

And validate that the file path is allowed.

---

# Expression Security with Branch Names

Branch names can contain unexpected characters.

Avoid constructing shell syntax directly from:

```yaml
${{ github.ref_name }}
```

Instead:

```yaml
env:
  BRANCH_NAME: ${{ github.ref_name }}

run: |
  printf '%s\n' "$BRANCH_NAME"
```

---

# Expressions and API Calls

When constructing API requests:

```text
github.repository
```

can be useful.

Example:

```bash
curl \
  "https://api.github.com/repos/${GITHUB_REPOSITORY}/..."
```

Use authentication through secure mechanisms and validate API responses.

---

# Expression and Dependabot Example

For a security workflow:

```text
github.repository
      |
      ↓
GitHub API
      |
      ↓
Dependabot Alerts
      |
      ↓
jq
      |
      ↓
Critical Count
```

Then a condition can determine whether the workflow should continue.

---

# Security Gate Example

Conceptually:

```yaml
if: ${{ steps.dependabot.outputs.critical_count == '0' }}
```

Or use a numeric comparison after converting data appropriately.

The scanner/API result should be trusted only after validating the API response.

---

# Expressions and DevSecOps

Typical pipeline:

```text
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
Test
  |
  ↓
Expression-Based Gate
  |
  ↓
UAT
```

Expressions can decide whether downstream steps/jobs execute based on trusted results.

---

# Expression-Based Security Gate

Example:

```yaml
if: >
  ${{
    needs.sonarqube.result == 'success' &&
    needs.trivy.result == 'success' &&
    needs.veracode.result == 'success'
  }}
```

This creates an explicit gate.

---

# Expressions and E2E Tests

```yaml
if: ${{ needs.e2e.result == 'success' }}
```

Production deployment should happen only after required tests succeed.

---

# Expressions and GitOps

Example:

```yaml
if: >
  ${{
    github.ref_name == 'main' &&
    needs.security.result == 'success' &&
    needs.e2e.result == 'success'
  }}
```

Then:

```text
Update GitOps Repository
        |
        ↓
ArgoCD
        |
        ↓
EKS
```

---

# Production Promotion Expression

Example:

```yaml
if: >
  ${{
    github.ref_name == 'main' &&
    needs.build.result == 'success' &&
    needs.security.result == 'success' &&
    needs.e2e.result == 'success' &&
    needs.jira.outputs.approved == 'true'
  }}
```

Then:

```yaml
environment:
  name: production
```

This provides a logical gate, while environment protection provides the actual approval boundary.

---

# Expressions and Rollback

A workflow may use conditions to select rollback behavior.

Example concept:

```text
Deployment
   |
   ↓
Failure
   |
   ↓
failure()
   |
   ↓
Rollback Logic
```

However, rollback should be designed carefully and should not blindly execute destructive operations.

---

# Expression-Based Notification

Example:

```yaml
- name: Notify Failure
  if: ${{ failure() }}
  run: |
    ./notify.sh
```

Useful for:

```text
Slack
JIRA
Incident Management
Email
```

---

# Expression-Based Cleanup

Example:

```yaml
- name: Cleanup
  if: ${{ always() }}
  run: |
    ./cleanup.sh
```

Do not use `always()` for cleanup that could accidentally delete important production resources.

---

# Expressions and Reusable Workflows

Caller:

```yaml
jobs:

  deploy:

    uses: company/platform-workflows/deploy.yml@v1

    with:
      environment: production
      version: ${{ github.sha }}
```

The reusable workflow can use the supplied inputs.

---

# Expressions and Custom Actions

Example:

```yaml
- uses: company/deploy-action@v1
  with:
    environment: ${{ inputs.environment }}
    version: ${{ github.sha }}
```

The workflow dynamically supplies values to the Action.

---

# Expressions and Composite Actions

Example:

```yaml
- uses: ./actions/deploy
  with:
    service: ${{ matrix.service }}
    version: ${{ github.sha }}
```

This allows reusable logic to work across multiple services.

---

# Expressions and Matrix

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
  SERVICE: ${{ matrix.service }}
```

The same workflow runs for each service.

---

# Dynamic Service Deployment

```text
Matrix
 |
 ├── user
 ├── catalogue
 ├── cart
 ├── orders
 ├── payment
 ├── inventory
 └── notification
        |
        ↓
Expression
        |
        ↓
Deploy Selected Service
```

---

# Expression and Environment Mapping

Instead of:

```yaml
environment:
  name: ${{ inputs.environment }}
```

with arbitrary input, validate the environment first.

For example:

```text
qa
uat
production
```

Then map it to trusted deployment configuration.

---

# Expression and Cloud Authentication

Example:

```yaml
permissions:
  id-token: write
  contents: read
```

Then use:

```text
GitHub identity
      |
      ↓
OIDC
      |
      ↓
AWS IAM
```

The IAM trust policy should restrict:

```text
Repository
Branch
Environment
```

as appropriate.

---

# Expression and AWS Region

Example:

```yaml
env:
  AWS_REGION: ${{ vars.AWS_REGION }}
```

This avoids hardcoding region values throughout the workflow.

---

# Expression and ECR

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}
  AWS_REGION: ${{ vars.AWS_REGION }}
```

Flow:

```text
github.sha
   ↓
Image Tag
   ↓
ECR
   ↓
Image Digest
```

---

# Expression and Helm

Example:

```yaml
env:
  IMAGE_TAG: ${{ github.sha }}

run: |
  helm upgrade catalogue ./helm/catalogue \
    --set image.tag="$IMAGE_TAG"
```

The immutable commit SHA identifies the version.

---

# Expression and ArgoCD

GitOps workflow:

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
Update GitOps Manifest
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

Expressions provide the dynamic values required to move information through the workflow.

---

# Expression and JIRA

Example:

```yaml
env:
  JIRA_TICKET: ${{ inputs.jira-ticket }}
```

Then:

```text
JIRA Ticket
   |
   ↓
JIRA API
   |
   ↓
Validation
   |
   ↓
Output
   |
   ↓
Expression
   |
   ↓
Production Gate
```

---

# Expression and Deployment Window

Conceptually:

```text
JIRA
 |
 └── Deployment Window
          |
          ↓
Current Time
          |
          ↓
Validation
          |
          ↓
Output
          |
          ↓
Expression
```

Do not rely on a user-provided timestamp as proof of an approved deployment window.

---

# Expressions and Change Request

Production gate:

```text
CR Approved
   AND
SHA Approved
   AND
Security Passed
   AND
E2E Passed
   AND
Deployment Window Valid
```

Expression:

```yaml
if: >
  ${{
    needs.cr.result == 'success' &&
    needs.security.result == 'success' &&
    needs.e2e.result == 'success'
  }}
```

The actual change-control validation should come from the trusted systems.

---

# Expression Readability

Bad:

```yaml
if: ${{ github.ref_name == 'main' && needs.a.result == 'success' && needs.b.result == 'success' && needs.c.result == 'success' && inputs.environment == 'production' }}
```

Better:

```yaml
if: >
  ${{
    github.ref_name == 'main' &&
    inputs.environment == 'production' &&
    needs.build.result == 'success' &&
    needs.security.result == 'success' &&
    needs.e2e.result == 'success'
  }}
```

Readable conditions are easier to review.

---

# Avoid Overly Complex Expressions

If an expression becomes difficult to understand:

```text
Move validation into a dedicated job
      |
      ↓
Produce clear outputs
      |
      ↓
Use simple production condition
```

Example:

```text
Validation Job
 ├── JIRA
 ├── SHA
 ├── Security
 └── Tests
       |
       ↓
approved=true
       |
       ↓
Production
```

---

# Better Production Architecture

Instead of one giant expression:

```text
Production
  |
  └── 20 conditions
```

use:

```text
JIRA Validation
      |
      ↓
Security Validation
      |
      ↓
Test Validation
      |
      ↓
Promotion Gate
      |
      ↓
Production Environment
```

Each stage has a clear responsibility.

---

# Expressions and Outputs

Expressions connect outputs to later stages.

Example:

```text
Build
  |
  └── image-digest
          |
          ↓
      Expression
          |
          ↓
Production
```

Reference:

```yaml
${{ needs.build.outputs.image-digest }}
```

---

# Expressions and Status

Useful combination:

```yaml
if: >
  ${{
    failure() &&
    github.ref_name == 'main'
  }}
```

This can trigger a main-branch failure notification.

---

# Expressions and Auditability

Use expressions to construct metadata:

```yaml
env:
  REPOSITORY: ${{ github.repository }}
  SHA: ${{ github.sha }}
  RUN_ID: ${{ github.run_id }}
  ACTOR: ${{ github.actor }}
```

This can be sent to an approved deployment/audit system.

---

# Expression Security Checklist

```text
☐ Treat dynamic values as untrusted
☐ Validate workflow inputs
☐ Validate external API responses
☐ Avoid direct shell interpolation
☐ Use environment variables for shell data
☐ Quote shell variables
☐ Do not print secrets
☐ Do not dump entire contexts
☐ Restrict production conditions
☐ Use GitHub Environment protection
☐ Use least-privilege permissions
☐ Restrict OIDC trust
☐ Avoid arbitrary runner selection
```

---

# Common Mistakes

### 1. Confusing strings and booleans

```text
"true"
```

and:

```text
true
```

are not conceptually the same type.

### 2. Giant expressions

Move complex logic into validation jobs.

### 3. Using expressions as authorization

Conditions do not replace approvals.

### 4. Directly interpolating untrusted data

This can create command injection risks.

### 5. Dumping contexts

Avoid unnecessary debug output.

### 6. Using mutable references

Prefer immutable SHA/image digest values.

### 7. No validation

Do not assume an input is safe.

---

# Best Practices

- Use expressions to make workflows dynamic.
- Keep expressions readable.
- Use contexts rather than hardcoding information.
- Use `&&`, `||`, and `!` carefully.
- Use parentheses for complex conditions.
- Use status functions for failure/cleanup behavior.
- Use `contains()`, `startsWith()`, and `endsWith()` where appropriate.
- Use `fromJSON()` for controlled dynamic matrices.
- Use outputs for generated data.
- Validate critical production values.
- Treat user-controlled values as untrusted.
- Avoid direct shell interpolation of dynamic values.
- Use environment variables for shell input.
- Keep authorization separate from workflow logic.
- Use GitHub Environments for production protection.

---

# Production-Level Expression Architecture

```text
                       GitHub Event
                            |
                            ↓
                        Contexts
                            |
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
           github         inputs         vars
              |             |             |
              └─────────────┼─────────────┘
                            ↓
                       Validation
                            |
              ┌─────────────┼─────────────┐
              ↓             ↓             ↓
            JIRA         Security        Tests
              |             |             |
              └─────────────┼─────────────┘
                            ↓
                         Outputs
                            |
                            ↓
                       Expressions
                            |
                            ↓
                  Production Environment
                            |
                            ↓
                         Approval
                            |
                            ↓
                          ArgoCD
                            |
                            ↓
                           EKS
```

---

# Complete Production Example

```yaml
name: Production Deployment

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

      version:
        description: Version / commit SHA
        required: true
        type: string

permissions:
  contents: read
  id-token: write

jobs:

  validate:

    runs-on: ubuntu-latest

    outputs:
      approved: ${{ steps.check.outputs.approved }}

    steps:

      - name: Validate
        id: check
        env:
          ENVIRONMENT: ${{ inputs.environment }}
          VERSION: ${{ inputs.version }}
        run: |
          if [[ "$ENVIRONMENT" != "qa" &&
                "$ENVIRONMENT" != "uat" &&
                "$ENVIRONMENT" != "production" ]]; then
            echo "Invalid environment"
            exit 1
          fi

          if [[ -z "$VERSION" ]]; then
            echo "Version is required"
            exit 1
          fi

          echo "approved=true" >> "$GITHUB_OUTPUT"

  production:

    needs: validate

    if: >
      ${{
        needs.validate.result == 'success' &&
        needs.validate.outputs.approved == 'true' &&
        inputs.environment == 'production'
      }}

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          VERSION: ${{ inputs.version }}
        run: |
          echo "Deploying version: $VERSION"
```

This demonstrates:

```text
workflow_dispatch
      ↓
inputs
      ↓
Validation
      ↓
Job Output
      ↓
Expression
      ↓
Production Environment
      ↓
Deployment
```

In a real production workflow, the validation job should perform actual JIRA, artifact, security, testing, and change-management checks rather than treating `approved=true` as sufficient authorization.

---

# Key Takeaways

Expressions are the decision-making and dynamic-value mechanism of GitHub Actions.

Remember:

```text
${{ github.sha }}
```

Context value.

```text
${{ inputs.environment }}
```

Workflow input.

```text
${{ steps.build.outputs.image }}
```

Step output.

```text
${{ needs.build.outputs.image }}
```

Job output.

```text
${{ matrix.service }}
```

Matrix value.

```text
${{ github.ref_name == 'main' }}
```

Comparison.

```text
${{ conditionA && conditionB }}
```

AND.

```text
${{ conditionA || conditionB }}
```

OR.

```text
${{ !condition }}
```

NOT.

The most important production principle:

```text
Use expressions to control workflow behavior,
but do not confuse workflow conditions with
security authorization or production approval.
```

For production deployments:

```text
Expression
    ↓
Logical Gate
    ↓
GitHub Environment
    ↓
Required Approval
    ↓
Deployment
```

---

# Interview Questions

## Basic

1. What are GitHub Actions expressions?
2. What is the `${{ }}` syntax?
3. Where can expressions be used?
4. What is the difference between a context and an expression?
5. What is the `==` operator?
6. What are `&&`, `||`, and `!`?
7. What is `contains()`?
8. What is `startsWith()`?
9. What is `endsWith()`?
10. What is `format()`?
11. What is `toJSON()`?
12. What is `fromJSON()`?
13. What are `success()`, `failure()`, `cancelled()`, and `always()`?

## Intermediate

14. How do you use expressions with workflow inputs?
15. How do you compare a job output in an `if` condition?
16. How do you use expressions with matrix values?
17. How do you check whether a workflow was triggered by a pull request?
18. How do you check whether a workflow is running on the main branch?
19. How do you dynamically select an environment?
20. How do you use expressions with `env`?
21. How do you use expressions with `with`?
22. How do you use `needs` with expressions?
23. How do you create a dynamic matrix using `fromJSON()`?
24. How do you use `failure()` for notifications?
25. What is the difference between `success()` and `needs.<job>.result`?
26. How do you combine multiple conditions?
27. How do you make a complex expression readable?

## Advanced / Production

28. Design a production deployment gate using GitHub Actions expressions.
29. How would you combine JIRA approval, security results, E2E results, and SHA validation in a production condition?
30. How would you pass a validated image digest from Build to Production using expressions and outputs?
31. How would you prevent command injection through workflow inputs?
32. Why is directly interpolating pull-request data into shell commands dangerous?
33. How would you safely use a PR title inside a shell command?
34. How would you use expressions with OIDC and AWS IAM?
35. How would you restrict production deployment to `main` while still supporting manual deployment?
36. How would you combine expressions with GitHub Environment protection?
37. How would you design a dynamic multi-microservice deployment using matrix expressions?
38. How would you use `fromJSON()` to dynamically create a service matrix?
39. How would you implement a DevSecOps gate using SonarQube, Trivy, and Veracode results?
40. How would you use expressions to control ArgoCD/EKS promotion?
41. How would you implement failure notifications without exposing secrets?
42. How would you design rollback conditions using expressions?
43. Why should expressions not be treated as an authorization mechanism?
44. How would you avoid creating one giant production expression?
45. Design an enterprise-grade GitHub Actions workflow using contexts, expressions, inputs, outputs, reusable workflows, OIDC, GitHub Environments, JIRA change control, ArgoCD, and EKS.