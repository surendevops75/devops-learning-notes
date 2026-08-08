# GitHub Actions Steps

A **step** is an individual task inside a GitHub Actions job.

Steps are executed sequentially within a job.

A step can:

- Run a shell command
- Execute a script
- Use a GitHub Action
- Set environment variables
- Generate outputs
- Upload artifacts
- Download artifacts
- Perform deployment operations

Basic structure:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Install Dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Test
        run: npm test
```

Execution order:

```text
Checkout
   |
   ↓
Install Dependencies
   |
   ↓
Build
   |
   ↓
Test
```

---

# Step Structure

A step can commonly use:

```yaml
- name: Step Name
  uses: action/name@version
```

or:

```yaml
- name: Step Name
  run: command
```

Example:

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Build
    run: mvn clean package
```

---

# Step Name

The `name` field provides a readable description of the step.

Example:

```yaml
- name: Run Unit Tests
  run: mvn test
```

The GitHub Actions UI displays:

```text
Run Unit Tests
```

Use descriptive names.

Good:

```yaml
- name: Run Unit Tests
```

Better than:

```yaml
- name: Test
```

for a complex production workflow where several types of tests exist.

---

# run

The `run` keyword executes a shell command.

Example:

```yaml
- name: Display Version
  run: java -version
```

Multiple commands:

```yaml
- name: Build Application
  run: |
    mvn clean
    mvn package
```

Execution:

```text
mvn clean
   |
   ↓
mvn package
```

If a command fails, the step normally fails.

---

# Multi-Line Commands

Use the `|` syntax for multiple commands.

Example:

```yaml
- name: Prepare Application
  run: |
    echo "Installing dependencies"
    npm install
    echo "Building application"
    npm run build
```

This keeps related commands inside one step.

---

# uses

The `uses` keyword runs a reusable GitHub Action.

Example:

```yaml
- name: Checkout Code
  uses: actions/checkout@v4
```

Another example:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: temurin
```

Actions provide reusable functionality without requiring you to write all the implementation yourself.

---

# run vs uses

The basic difference:

```text
run
 |
 └── Execute commands
```

```text
uses
 |
 └── Execute a reusable Action
```

Example:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

```yaml
- name: Build
  run: mvn clean package
```

The distinction becomes important when designing maintainable workflows.

A separate detailed comparison is covered in:

```text
03-Run-vs-Uses.md
```

---

# Step Execution Order

Steps inside the same job execute sequentially.

Example:

```yaml
steps:

  - name: Step 1
    run: echo "First"

  - name: Step 2
    run: echo "Second"

  - name: Step 3
    run: echo "Third"
```

Execution:

```text
Step 1
  |
  ↓
Step 2
  |
  ↓
Step 3
```

The next step normally starts only after the previous step finishes successfully.

---

# Step Failure

Suppose:

```yaml
steps:

  - name: Build
    run: mvn clean package

  - name: Test
    run: mvn test

  - name: Deploy
    run: ./deploy.sh
```

If Build fails:

```text
Build
  |
  ↓
FAILED
  |
  X
Test
  X
Deploy
```

By default, subsequent steps do not continue after a failure.

---

# Step Conditions

A step can use an `if` condition.

Example:

```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main'
  run: ./deploy.sh
```

The step executes only when the condition is true.

---

# Step Condition Example

Example:

```yaml
steps:

  - name: Build
    run: mvn clean package

  - name: Deploy
    if: github.ref == 'refs/heads/main'
    run: ./deploy.sh
```

The build runs normally.

Deployment runs only for `main`.

For production, branch conditions should not be the only deployment protection.

---

# Always Run a Step

GitHub Actions provides status-check functions such as:

```text
success()
failure()
always()
cancelled()
```

Example:

```yaml
- name: Cleanup
  if: ${{ always() }}
  run: ./cleanup.sh
```

This can be useful for cleanup or diagnostic operations.

Use `always()` carefully in production workflows so that it does not accidentally perform sensitive operations after a failure.

---

# Run Only After Failure

A step can run only when a previous step fails.

Example:

```yaml
- name: Failure Diagnostics
  if: ${{ failure() }}
  run: ./collect-diagnostics.sh
```

Execution:

```text
Build
  |
  ↓
Failure
  |
  ↓
Diagnostics
```

This is useful for troubleshooting.

---

# Run Only After Success

A step can explicitly require successful execution.

Example:

```yaml
- name: Deploy
  if: ${{ success() }}
  run: ./deploy.sh
```

This is useful when you want the condition to be explicit.

---

# Run When Cancelled

Example:

```yaml
- name: Cancellation Cleanup
  if: ${{ cancelled() }}
  run: ./cleanup.sh
```

This can be useful for cleanup activities after a workflow is cancelled.

---

# Environment Variables

Steps can use environment variables.

Example:

```yaml
- name: Display Environment
  env:
    ENVIRONMENT: qa
  run: echo "Deploying to $ENVIRONMENT"
```

Execution:

```text
Step
 |
 ├── ENVIRONMENT=qa
 |
 └── Command
```

---

# Job-Level vs Step-Level Environment Variables

Job-level:

```yaml
jobs:

  deploy:

    env:
      ENVIRONMENT: production

    steps:

      - name: Step 1
        run: echo "$ENVIRONMENT"

      - name: Step 2
        run: echo "$ENVIRONMENT"
```

Both steps can access the variable.

Step-level:

```yaml
steps:

  - name: QA Deployment
    env:
      ENVIRONMENT: qa
    run: ./deploy.sh
```

Only that step receives the variable.

---

# Repository Variables

GitHub Actions variables can be accessed using the `vars` context.

Example:

```yaml
- name: Display Environment
  run: echo "${{ vars.ENVIRONMENT }}"
```

Variables are appropriate for non-sensitive configuration.

Do not use ordinary variables for secrets.

---

# Secrets

Secrets are accessed through the `secrets` context.

Example:

```yaml
- name: Deploy
  env:
    TOKEN: ${{ secrets.DEPLOYMENT_TOKEN }}
  run: ./deploy.sh
```

Never hardcode:

```yaml
TOKEN: my-secret-token
```

Use secure secret management.

---

# Step Outputs

A step can create an output.

Example:

```yaml
- name: Get Version
  id: version
  run: echo "version=v1.5.0" >> "$GITHUB_OUTPUT"
```

The step has:

```text
id = version
```

The output can be referenced as:

```yaml
${{ steps.version.outputs.version }}
```

---

# Step ID

The `id` field gives a step a unique identifier.

Example:

```yaml
- name: Get Version
  id: version
  run: echo "version=v1.5.0" >> "$GITHUB_OUTPUT"
```

Later:

```yaml
- name: Display Version
  run: echo "${{ steps.version.outputs.version }}"
```

Architecture:

```text
Get Version
    |
    ↓
id: version
    |
    ↓
output: version
    |
    ↓
Display Version
```

---

# Step Outputs in Production

Step outputs are useful for values such as:

- Image tag
- Commit SHA
- Version
- Artifact name
- Deployment ID
- Terraform outputs

Example:

```yaml
- name: Generate Image Tag
  id: image
  run: |
    TAG="${GITHUB_SHA::7}"
    echo "tag=$TAG" >> "$GITHUB_OUTPUT"

- name: Display Image Tag
  run: echo "${{ steps.image.outputs.tag }}"
```

---

# Shell

A step can specify the shell.

Example:

```yaml
- name: Linux Command
  shell: bash
  run: |
    echo "Hello"
```

PowerShell:

```yaml
- name: Windows Command
  shell: pwsh
  run: |
    Write-Host "Hello"
```

The shell should match the runner operating system and script requirements.

---

# Working Directory

A step can specify its working directory.

Example:

```yaml
- name: Build Backend
  working-directory: backend
  run: mvn clean package
```

Directory structure:

```text
repository/
├── backend/
└── frontend/
```

The command executes from:

```text
backend/
```

---

# Timeout for a Step

Steps can have timeouts.

Example:

```yaml
- name: Integration Tests
  timeout-minutes: 15
  run: ./run-integration-tests.sh
```

This prevents a specific operation from hanging indefinitely.

---

# Continue on Error

A step can be configured to continue after failure.

Example:

```yaml
- name: Optional Report
  continue-on-error: true
  run: ./generate-report.sh
```

The workflow can continue even if this step fails.

Use this carefully.

Do not use `continue-on-error` for mandatory:

- Security scans
- Production validation
- Deployment gates
- Approval checks
- Critical tests

unless the failure is intentionally non-blocking.

---

# Production Security Gate

Bad:

```yaml
- name: Trivy Scan
  continue-on-error: true
  run: trivy image "$IMAGE"
```

If Trivy is a mandatory security gate, allowing the workflow to continue may permit a vulnerable image to reach production.

Better:

```yaml
- name: Trivy Scan
  run: trivy image "$IMAGE"
```

The scan should fail the pipeline when the configured security policy is violated.

---

# Upload Artifacts

A step can use an action to upload an artifact.

Example:

```yaml
- name: Upload Build Artifact
  uses: actions/upload-artifact@v4
  with:
    name: application
    path: target/application.jar
```

Flow:

```text
Build
  |
  ↓
application.jar
  |
  ↓
Upload Artifact
```

---

# Download Artifacts

Another job can download the artifact.

Example:

```yaml
- name: Download Artifact
  uses: actions/download-artifact@v4
  with:
    name: application
```

Flow:

```text
Build Job
    |
    ↓
Upload Artifact
    |
    ↓
Deployment Job
    |
    ↓
Download Artifact
```

---

# Checkout Step

Most workflows need repository source code.

Use:

```yaml
- name: Checkout Code
  uses: actions/checkout@v4
```

This makes repository content available to subsequent steps.

Typical flow:

```text
Checkout
   |
   ↓
Build
   |
   ↓
Test
```

---

# Setup Language Runtime

GitHub Actions provides official setup actions for common languages.

Example Java:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: temurin
```

Node.js:

```yaml
- name: Setup Node.js
  uses: actions/setup-node@v4
  with:
    node-version: '22'
```

Python:

```yaml
- name: Setup Python
  uses: actions/setup-python@v5
  with:
    python-version: '3.12'
```

---

# Production CI Step Sequence

A typical Java application job may look like:

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

  - name: Unit Tests
    run: mvn test

  - name: Package
    run: mvn package
```

Execution:

```text
Checkout
   |
   ↓
Setup Java
   |
   ↓
Build
   |
   ↓
Unit Tests
   |
   ↓
Package
```

---

# DevSecOps Step Sequence

A job can contain security-related steps.

Example:

```text
Checkout
   |
   ↓
Build
   |
   ↓
Unit Tests
   |
   ↓
SonarQube
   |
   ↓
Docker Build
   |
   ↓
Trivy
   |
   ↓
Push Image
```

A production implementation should configure the security tools so that mandatory policy violations fail the appropriate gate.

---

# Docker Build Steps

Example:

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Build Docker Image
    run: docker build -t catalogue:${GITHUB_SHA} .

  - name: Scan Image
    run: trivy image catalogue:${GITHUB_SHA}
```

Flow:

```text
Source
   |
   ↓
Docker Build
   |
   ↓
Image
   |
   ↓
Trivy
   |
   ↓
Push
```

---

# Helm Deployment Steps

A Kubernetes deployment may contain:

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Helm Upgrade
    run: |
      helm upgrade --install catalogue ./helm/catalogue \
        --namespace catalogue \
        --create-namespace \
        --set image.tag="${IMAGE_TAG}"
```

Production deployments should additionally validate:

- Kubernetes context
- Namespace
- Image version
- Helm values
- Deployment status
- Health checks
- Rollback strategy

---

# Terraform Steps

A Terraform job may use:

```yaml
steps:

  - name: Checkout
    uses: actions/checkout@v4

  - name: Setup Terraform
    uses: hashicorp/setup-terraform@v3

  - name: Terraform Init
    run: terraform init

  - name: Terraform Validate
    run: terraform validate

  - name: Terraform Plan
    run: terraform plan
```

Production flow:

```text
Checkout
   |
   ↓
Terraform Init
   |
   ↓
Validate
   |
   ↓
Plan
   |
   ↓
Approval
   |
   ↓
Apply
```

---

# Production Step Design

A production step should have:

```text
Clear Purpose
     |
     ↓
Controlled Input
     |
     ↓
Least Privilege
     |
     ↓
Secure Secrets
     |
     ↓
Expected Failure Behavior
     |
     ↓
Useful Logs
```

Avoid large steps containing unrelated operations.

Bad:

```yaml
- name: Everything
  run: |
    build
    test
    scan
    deploy
    cleanup
    notify
```

Prefer:

```yaml
- name: Build
  run: ...

- name: Test
  run: ...

- name: Security Scan
  run: ...

- name: Deploy
  run: ...

- name: Smoke Test
  run: ...
```

This makes troubleshooting easier.

---

# Step Logging

Good step names and commands make failures easier to identify.

Good:

```yaml
- name: Validate Terraform Configuration
  run: terraform validate
```

Better than:

```yaml
- name: Run
  run: terraform validate
```

Logs should provide enough information to diagnose failures without exposing secrets.

---

# Never Print Secrets

Avoid:

```yaml
- name: Debug
  run: echo "${{ secrets.DEPLOYMENT_TOKEN }}"
```

Never intentionally print:

- Passwords
- API tokens
- Access keys
- Private keys
- Database credentials

Use secure secret handling.

---

# Step Failure Diagnostics

A diagnostic step can run after failure.

Example:

```yaml
- name: Collect Diagnostics
  if: ${{ failure() }}
  run: |
    kubectl get pods -A
    kubectl get events -A
```

This can be useful for Kubernetes deployment troubleshooting.

Be careful not to print sensitive configuration.

---

# Production Kubernetes Example

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

  - name: Wait for Rollout
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

Execution:

```text
Deploy
   |
   ↓
Rollout Check
   |
   ↓
Smoke Test
   |
   ├── Success → Complete
   |
   └── Failure → Diagnostics
```

---

# Step Reuse

If the same group of steps is repeated across many workflows, consider:

- Reusable workflows
- Composite actions

For example:

```text
Checkout
Setup Java
Cache
Build
Test
```

If this sequence appears in many repositories, centralize it rather than copying it everywhere.

---

# Step vs Job

A Job:

```text
runs on a runner
```

A Step:

```text
runs inside a job
```

Example:

```text
Workflow
   |
   └── Job
        |
        ├── Checkout Step
        ├── Build Step
        ├── Test Step
        └── Deploy Step
```

---

# Steps and Job Isolation

Steps within the same job share the same runner environment.

Example:

```text
Step 1
   |
   ↓
Create file
   |
   ↓
Step 2
   |
   ↓
Read file
```

The file can normally be accessed by the next step because both steps execute within the same job environment.

This differs from separate jobs.

```text
Job A
  |
  └── Runner A

Job B
  |
  └── Runner B
```

For Job A → Job B file transfer, use artifacts or another external mechanism.

---

# Production Best Practices

- Give every important step a meaningful name.
- Keep steps focused on one responsibility.
- Use `uses` for reusable actions.
- Use `run` for commands and scripts.
- Use `id` when a step must expose outputs.
- Use `if` for controlled conditional execution.
- Use `failure()` for diagnostics.
- Use `always()` carefully for cleanup.
- Set timeouts for long-running operations.
- Use `working-directory` when appropriate.
- Use explicit shells when required.
- Keep secrets out of logs.
- Avoid unnecessary `continue-on-error`.
- Upload important build artifacts.
- Download artifacts explicitly in later jobs.
- Make security gates fail when policy requirements are violated.
- Keep production deployment steps separate and auditable.
- Use reusable workflows or composite actions when step sequences are repeatedly duplicated.

---

# Common Mistakes

- Putting the entire pipeline into one step.
- Using `continue-on-error` for mandatory security checks.
- Printing secrets in logs.
- Forgetting to checkout the repository.
- Assuming steps from different jobs share the filesystem.
- Using unclear step names.
- Hardcoding credentials.
- Running deployment without validation.
- Not collecting useful diagnostics after failures.
- Making one step responsible for unrelated operations.
- Repeating identical step sequences across many repositories.

---

# Summary

A step is an individual operation inside a GitHub Actions job.

The two fundamental ways to execute work are:

```text
run
 |
 └── Execute commands
```

and:

```text
uses
 |
 └── Execute reusable Actions
```

Steps execute sequentially within a job:

```text
Checkout
   |
   ↓
Setup
   |
   ↓
Build
   |
   ↓
Test
   |
   ↓
Security
   |
   ↓
Deploy
```

Steps can also use:

```text
if
env
id
with
working-directory
shell
timeout-minutes
continue-on-error
```

For production-grade workflows, steps should be:

```text
Focused
Secure
Auditable
Failure-aware
Easy to troubleshoot
```

---

# Interview Questions

## Basic

1. What is a step in GitHub Actions?
2. How are steps different from jobs?
3. What is the purpose of `run`?
4. What is the purpose of `uses`?
5. Do steps execute sequentially?
6. What is the purpose of the `name` field?
7. What is the purpose of `id`?

## Intermediate

8. How do you execute multiple commands in one step?
9. How do you conditionally execute a step?
10. How do you run a step only after a failure?
11. What is `always()`?
12. What is `continue-on-error`?
13. How do you pass environment variables to a step?
14. How do you pass secrets to a step?
15. How do you create and consume step outputs?
16. How do you specify a working directory?
17. How do you select a shell?

## Advanced

18. Design a production Java CI job using checkout, Java setup, Maven build, unit tests, SonarQube, Docker build, and Trivy.
19. Design a Kubernetes deployment job that deploys with Helm, waits for rollout completion, runs smoke tests, and collects diagnostics if deployment fails.
20. A security scan fails but the deployment continues. Explain how you would investigate the step configuration and failure handling.
21. A step needs a value generated by an earlier step. Explain how you would use `id` and `$GITHUB_OUTPUT`.
22. A deployment fails and you need Kubernetes diagnostics automatically. Design the appropriate failure-handling steps.
23. Explain the difference between sharing files between steps in the same job and sharing files between different jobs.
24. A team has copied the same 10 steps into 30 repositories. Explain how you would reduce duplication using reusable workflows or composite actions.
25. Design production-grade steps for a Docker build, image security scan, ECR push, Helm deployment, rollout validation, and smoke testing.
26. Explain why `continue-on-error` should generally not be used for mandatory security and production validation steps.