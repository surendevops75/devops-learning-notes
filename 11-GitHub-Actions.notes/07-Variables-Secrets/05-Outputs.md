# GitHub Actions Outputs

Outputs allow a step or job to produce a value that can be consumed by later steps or jobs.

They are useful when one part of a workflow generates information that another part needs.

Typical examples:

```text
Image Tag
Image Digest
Commit SHA
Build Version
Artifact Name
Terraform Output
Deployment URL
Release ID
```

The basic flow is:

```text
Step
  |
  ↓
Output
  |
  ↓
Later Step
```

For cross-job communication:

```text
Job A
  |
  ↓
Job Output
  |
  ↓
Job B
```

---

# Why Use Outputs?

Consider a build job that generates an image tag:

```text
Build
  |
  ↓
IMAGE_TAG=8a92f31
```

The deployment job needs that exact value.

Instead of calculating the value again:

```text
Build Job
   |
   ↓
Output: image-tag
   |
   ↓
Deploy Job
```

This creates a clear data flow.

---

# Outputs vs Environment Variables

### Environment Variable

Used to provide configuration to a process:

```yaml
env:
  APP_NAME: catalogue
```

### Output

Used to pass generated data:

```text
Build
 ↓
Generated Image Tag
 ↓
Deploy
```

Simple rule:

```text
Configuration → env / vars
Generated value → outputs
Sensitive value → secrets
```

---

# Step Outputs

A step must have an `id` if another step needs to reference its outputs.

Example:

```yaml
steps:

  - name: Generate Version
    id: version
    run: |
      echo "version=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"

  - name: Display Version
    run: |
      echo "Version: ${{ steps.version.outputs.version }}"
```

---

# `$GITHUB_OUTPUT`

GitHub Actions provides the special environment variable:

```text
GITHUB_OUTPUT
```

Write a step output to this file.

Example:

```bash
echo "version=1.0.0" >> "$GITHUB_OUTPUT"
```

Then the output can be referenced using:

```text
steps.<step_id>.outputs.<output_name>
```

---

# Output Syntax

Example:

```yaml
- name: Generate Version
  id: version
  run: |
    echo "version=1.2.3" >> "$GITHUB_OUTPUT"
```

Consume it:

```yaml
- name: Display
  run: |
    echo "${{ steps.version.outputs.version }}"
```

Structure:

```text
steps
  └── version
       └── outputs
            └── version
```

---

# Step ID

Example:

```yaml
id: build
```

The ID becomes part of the output reference.

Example:

```yaml
${{ steps.build.outputs.image }}
```

Without the correct step ID, the output cannot be referenced this way.

---

# Basic Example

```yaml
name: Outputs Demo

on:
  push:

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Generate Version
        id: version
        run: |
          VERSION="${GITHUB_SHA::7}"
          echo "version=$VERSION" >> "$GITHUB_OUTPUT"

      - name: Display Version
        run: |
          echo "Version: ${{ steps.version.outputs.version }}"
```

---

# Generating an Image Tag

For containerized applications:

```yaml
- name: Generate Image Tag
  id: image
  run: |
    IMAGE_TAG="${GITHUB_SHA::7}"
    echo "tag=$IMAGE_TAG" >> "$GITHUB_OUTPUT"
```

Use it:

```yaml
- name: Build Image
  run: |
    docker build \
      -t catalogue:${{ steps.image.outputs.tag }} \
      .
```

---

# Image Digest Output

After pushing an image, the workflow may need the immutable image digest.

Conceptually:

```text
Docker Build
    |
    ↓
Push to ECR
    |
    ↓
Image Digest
    |
    ↓
Output
```

Example:

```yaml
- name: Set Image Digest
  id: image
  run: |
    echo "digest=sha256:..." >> "$GITHUB_OUTPUT"
```

Later:

```yaml
run: |
  echo "Digest: ${{ steps.image.outputs.digest }}"
```

---

# Multiple Step Outputs

A single step can produce multiple outputs.

Example:

```yaml
- name: Build Metadata
  id: metadata
  run: |
    echo "version=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"
    echo "service=catalogue" >> "$GITHUB_OUTPUT"
    echo "environment=qa" >> "$GITHUB_OUTPUT"
```

Consume:

```yaml
run: |
  echo "Version: ${{ steps.metadata.outputs.version }}"
  echo "Service: ${{ steps.metadata.outputs.service }}"
  echo "Environment: ${{ steps.metadata.outputs.environment }}"
```

---

# Output Names

Use clear names:

```text
version
image-tag
image-digest
artifact-name
deployment-url
```

Avoid:

```text
x
data
value
result
```

Clear output names make workflows easier to maintain.

---

# Job Outputs

Job outputs allow values generated in one job to be consumed by another job.

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.image.outputs.tag }}

    steps:

      - name: Generate Image Tag
        id: image
        run: |
          echo "tag=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"

  deploy:

    needs: build

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          echo "Deploying ${{ needs.build.outputs.image-tag }}"
```

---

# Job Output Flow

```text
Build Job
    |
    ├── Step
    │    └── image.tag
    |
    ↓
Job Output
    |
    ↓
needs.build.outputs.image-tag
    |
    ↓
Deploy Job
```

---

# `needs`

The downstream job needs to declare the upstream job using:

```yaml
needs: build
```

Then it can access:

```yaml
${{ needs.build.outputs.image-tag }}
```

Without the dependency, the intended data flow is not established.

---

# Multiple Job Dependencies

Example:

```yaml
jobs:

  build:
    ...

  security:
    ...

  deploy:
    needs:
      - build
      - security
```

The deploy job can use outputs from the jobs it depends on.

---

# Multiple Job Outputs

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.meta.outputs.tag }}
      image-digest: ${{ steps.meta.outputs.digest }}

    steps:

      - name: Generate Metadata
        id: meta
        run: |
          echo "tag=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"
          echo "digest=sha256:example" >> "$GITHUB_OUTPUT"
```

Later:

```yaml
run: |
  echo "Tag: ${{ needs.build.outputs.image-tag }}"
  echo "Digest: ${{ needs.build.outputs.image-digest }}"
```

---

# Production Build-to-Deploy Flow

```text
Build
  |
  ├── Compile
  ├── Test
  ├── SonarQube
  ├── Trivy
  ├── Veracode
  └── Docker Build
          |
          ↓
      Push to ECR
          |
          ↓
      Image Digest
          |
          ↓
       Job Output
          |
          ↓
        UAT
          |
          ↓
      E2E Tests
          |
          ↓
       Production
```

Outputs provide the connection between these stages.

---

# Immutable Image Output

A production workflow should preferably pass an immutable identifier.

Example:

```text
Image Digest:
sha256:abc123...
```

instead of:

```text
latest
```

This provides stronger traceability.

---

# SHA Output

A commit SHA can be exposed as an output:

```yaml
- name: Set Commit SHA
  id: commit
  run: |
    echo "sha=$GITHUB_SHA" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
echo "${{ steps.commit.outputs.sha }}"
```

---

# Short SHA Output

Example:

```yaml
- name: Generate Short SHA
  id: version
  run: |
    echo "sha=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"
```

Result:

```text
8a92f31
```

Short SHAs are convenient for display and tagging, but use the full SHA where uniqueness and verification are important.

---

# Build Version Output

Example:

```yaml
- name: Generate Version
  id: version
  run: |
    VERSION="1.4.${GITHUB_RUN_NUMBER}"
    echo "version=$VERSION" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
run: |
  echo "Build version: ${{ steps.version.outputs.version }}"
```

---

# Artifact Name Output

Example:

```yaml
- name: Generate Artifact Name
  id: artifact
  run: |
    echo "name=catalogue-${GITHUB_SHA::7}.tar.gz" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
run: |
  echo "${{ steps.artifact.outputs.name }}"
```

---

# Deployment URL Output

A deployment step can produce a URL:

```yaml
- name: Deploy
  id: deploy
  run: |
    echo "url=https://qa.example.com" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
- name: Display URL
  run: |
    echo "Application: ${{ steps.deploy.outputs.url }}"
```

---

# Terraform Outputs

Terraform can produce outputs:

```text
Terraform
   |
   ↓
terraform output
   |
   ↓
GitHub Actions
   |
   ↓
Job Output
```

Example concept:

```bash
terraform output -raw alb_dns_name
```

The workflow can capture the value and expose it as an output.

---

# Terraform Output Example

```yaml
- name: Terraform Output
  id: terraform
  run: |
    ALB_DNS=$(terraform output -raw alb_dns_name)
    echo "alb_dns=$ALB_DNS" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
run: |
  echo "ALB: ${{ steps.terraform.outputs.alb_dns }}"
```

---

# Outputs and Kubernetes

A deployment step can produce:

```text
Namespace
Service
Deployment
Endpoint
Version
```

Example:

```yaml
- name: Get Deployment Version
  id: deploy
  run: |
    VERSION=$(kubectl -n catalogue get deployment catalogue \
      -o jsonpath='{.spec.template.spec.containers[0].image}')

    echo "image=$VERSION" >> "$GITHUB_OUTPUT"
```

---

# Outputs and Helm

Example:

```yaml
- name: Helm Release
  id: helm
  run: |
    REVISION=$(helm history catalogue -n production -o json | jq -r '.[-1].revision')
    echo "revision=$REVISION" >> "$GITHUB_OUTPUT"
```

Later:

```yaml
run: |
  echo "Helm revision: ${{ steps.helm.outputs.revision }}"
```

---

# Outputs and Rollback

Outputs can capture deployment metadata:

```text
Release Name
Revision
Image Tag
Image Digest
Commit SHA
```

Example:

```text
Deploy
  |
  ├── Image Digest
  ├── Helm Revision
  └── Commit SHA
          |
          ↓
      Outputs
          |
          ↓
   Deployment Record
```

This helps with traceability and rollback decisions.

---

# Outputs and JIRA

A validation Action can produce:

```text
ticket-status
approved
change-window
approved-version
```

Example:

```yaml
- name: Validate JIRA
  id: jira
  run: |
    echo "approved=true" >> "$GITHUB_OUTPUT"
    echo "status=Approved" >> "$GITHUB_OUTPUT"
```

Later:

```yaml
if: ${{ steps.jira.outputs.approved == 'true' }}
```

The real production implementation should obtain these values from the JIRA API rather than trusting hardcoded output values.

---

# JIRA Production Gate

Conceptually:

```text
JIRA Ticket
    |
    ↓
JIRA API
    |
    ├── Exists
    ├── Approved
    ├── Correct Version
    └── Deployment Window
    |
    ↓
Outputs
    |
    ↓
Production Gate
```

---

# Outputs and Security Checks

Security tools can produce results used by later stages.

Example:

```text
Trivy
  |
  ↓
Critical Vulnerabilities
  |
  ↓
Output
  |
  ↓
Deployment Decision
```

Example:

```yaml
- name: Security Check
  id: security
  run: |
    echo "passed=true" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
if: ${{ steps.security.outputs.passed == 'true' }}
```

The actual value should come from the scanner's verified result.

---

# Outputs and Test Results

Example:

```text
E2E Tests
    |
    ↓
Result
    |
    ↓
Output
    |
    ↓
Production Job
```

However, for robust workflows, job success/failure and test reporting mechanisms should be used rather than relying only on string outputs.

---

# Output as Boolean

Example:

```yaml
echo "passed=true" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
if: ${{ steps.test.outputs.passed == 'true' }}
```

Be careful:

```text
"true"
```

is an output value represented through the workflow expression system; validate the exact behavior expected by the expression.

---

# Output as Number

Example:

```yaml
echo "count=5" >> "$GITHUB_OUTPUT"
```

Then:

```yaml
echo "${{ steps.scan.outputs.count }}"
```

When using numeric comparisons, use expressions appropriately.

---

# Output as JSON

Outputs can carry structured data when necessary.

Example:

```yaml
echo 'metadata={"service":"catalogue","version":"1.2.0"}' >> "$GITHUB_OUTPUT"
```

Then:

```yaml
echo '${{ steps.meta.outputs.metadata }}'
```

For complex data, carefully consider size, quoting, escaping, and security.

---

# Multiline Outputs

Multiline values can be written using the GitHub Actions delimiter syntax.

Example:

```bash
{
  echo 'summary<<EOF'
  echo 'Build completed'
  echo 'Tests passed'
  echo 'Security scan passed'
  echo 'EOF'
} >> "$GITHUB_OUTPUT"
```

Then:

```yaml
echo "${{ steps.build.outputs.summary }}"
```

Use multiline outputs only when necessary.

---

# Output Encoding

Be careful when output values contain:

```text
Newlines
Quotes
Special Characters
Shell Characters
JSON
```

Use GitHub's supported output file syntax rather than manually constructing unsafe strings.

---

# Output Injection

Never blindly place untrusted content into outputs and later execute it as shell code.

Dangerous flow:

```text
Untrusted Input
    |
    ↓
Output
    |
    ↓
Shell Command
```

Treat outputs containing user-controlled data as untrusted.

---

# Safe Output Usage

Prefer:

```yaml
env:
  VERSION: ${{ steps.build.outputs.version }}

run: |
  ./deploy.sh "$VERSION"
```

and validate the value.

Avoid building arbitrary shell syntax from outputs.

---

# Outputs and Secrets

Do not use outputs as a mechanism for sharing secrets unnecessarily.

Bad design:

```text
Secret
  |
  ↓
Step Output
  |
  ↓
Many Jobs
```

This increases exposure.

Use secrets directly where required or use an appropriate secret-management system.

---

# Secret Output Risk

If a step produces:

```text
password=...
```

as an output, downstream jobs may gain access to sensitive data.

Avoid this architecture.

Prefer:

```text
Secret Manager
      |
      ↓
Specific Job
```

---

# Outputs and Job Dependencies

Example:

```yaml
jobs:

  build:

    outputs:
      image: ${{ steps.image.outputs.name }}

  test:
    needs: build

  deploy:
    needs:
      - build
      - test
```

The deploy job can consume:

```yaml
${{ needs.build.outputs.image }}
```

---

# Outputs and Conditional Jobs

Example:

```yaml
jobs:

  validate:

    outputs:
      approved: ${{ steps.check.outputs.approved }}

    steps:

      - name: Check
        id: check
        run: |
          echo "approved=true" >> "$GITHUB_OUTPUT"

  deploy:

    needs: validate

    if: ${{ needs.validate.outputs.approved == 'true' }}

    steps:

      - name: Deploy
        run: |
          echo "Deploying"
```

---

# Production Approval Pattern

A production workflow can use:

```text
Validation Job
      |
      ↓
JIRA Output
      |
      ↓
Security Output
      |
      ↓
Test Output
      |
      ↓
Production Job
```

However, actual authorization should be enforced with GitHub Environment protection and appropriate permissions, not merely with output strings.

---

# Outputs and `needs`

`needs` provides access to job outputs.

Example:

```yaml
${{ needs.build.outputs.image-tag }}
```

Structure:

```text
needs
  └── build
       └── outputs
            └── image-tag
```

---

# Multiple Jobs Producing Outputs

Example:

```text
Build Job
   └── image-tag

Security Job
   └── scan-status

Test Job
   └── test-status

Deploy Job
   |
   ├── needs.build.outputs.image-tag
   ├── needs.security.outputs.scan-status
   └── needs.test.outputs.test-status
```

This creates explicit dependencies.

---

# Production CI/CD Example

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.image.outputs.tag }}

    steps:

      - name: Build Image
        id: image
        run: |
          TAG="${GITHUB_SHA::7}"
          echo "tag=$TAG" >> "$GITHUB_OUTPUT"

  security:

    needs: build
    runs-on: ubuntu-latest

    outputs:
      passed: ${{ steps.scan.outputs.passed }}

    steps:

      - name: Security Scan
        id: scan
        run: |
          echo "passed=true" >> "$GITHUB_OUTPUT"

  deploy:

    needs:
      - build
      - security

    if: ${{ needs.security.outputs.passed == 'true' }}

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
        run: |
          echo "Deploying catalogue:$IMAGE_TAG"
```

This demonstrates:

```text
Build
  ↓
Image Output
  ↓
Deploy

Security
  ↓
Approval Output
  ↓
Deploy
```

---

# Build → UAT → E2E → Production

A production pipeline can use outputs to pass deployment metadata:

```text
BUILD
  |
  └── image-digest
          |
          ↓
UAT DEPLOY
  |
  └── deployment-revision
          |
          ↓
E2E TEST
  |
  └── test-result
          |
          ↓
PROD
```

This provides a traceable deployment chain.

---

# Example Production Flow

```yaml
jobs:

  build:

    outputs:
      image-digest: ${{ steps.image.outputs.digest }}

  uat:

    needs: build

  e2e:

    needs: uat

  production:

    needs:
      - build
      - e2e

    environment:
      name: production

    steps:

      - name: Deploy
        env:
          IMAGE_DIGEST: ${{ needs.build.outputs.image-digest }}
        run: |
          echo "Deploying $IMAGE_DIGEST"
```

---

# Outputs and GitOps

A GitOps pipeline may produce:

```text
image-tag
image-digest
git-commit
manifest-version
```

Then:

```text
Build
 ↓
Output
 ↓
Update GitOps Repository
 ↓
ArgoCD
 ↓
EKS
```

The output represents the exact artifact to promote.

---

# GitOps Production Example

```text
Application Commit
      |
      ↓
Build Image
      |
      ↓
ECR
      |
      ↓
Image Digest
      |
      ↓
GitHub Actions Output
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

---

# Outputs and Artifactory

If JFrog Artifactory is used:

```text
Build
  |
  ↓
Artifact
  |
  ↓
Artifactory
  |
  ↓
Artifact URL
  |
  ↓
Output
  |
  ↓
Deployment
```

Example output:

```text
artifact-url
```

---

# Outputs and Terraform

A Terraform workflow may produce:

```text
VPC ID
EKS Cluster Name
ALB DNS
ECR Repository
RDS Endpoint
```

These can be captured as outputs when later workflow stages need them.

Example:

```text
Terraform
   |
   ├── EKS Cluster
   ├── ECR
   └── ALB
        |
        ↓
      Outputs
        |
        ↓
 Application Deployment
```

---

# Outputs and Infrastructure Provisioning

Production infrastructure pipeline:

```text
Terraform
    |
    ↓
Infrastructure Outputs
    |
    ├── Cluster
    ├── Registry
    └── Load Balancer
    |
    ↓
Application Deployment
```

Use explicit outputs instead of scraping logs.

---

# Why Outputs Are Better Than Logs

Bad:

```text
Build logs contain:
IMAGE_TAG=abc123
```

and another script searches the logs.

Better:

```text
Build Job
   |
   ↓
Explicit Output
   |
   ↓
Deploy Job
```

Outputs provide structured workflow communication.

---

# Outputs vs Artifacts

### Outputs

Best for:

```text
Small values
Metadata
Identifiers
Status
URLs
Version
Digest
```

### Artifacts

Best for:

```text
Build Packages
Reports
Test Results
Large Files
Deployment Bundles
```

---

# Outputs vs Cache

Cache is intended for:

```text
Reusable Dependencies
Build Cache
Package Cache
```

Do not use cache as a general communication mechanism between jobs.

---

# Outputs vs Variables

Use:

```text
Variables
```

for configuration.

Use:

```text
Outputs
```

for generated values.

Example:

```text
AWS_REGION
    ↓
Variable

IMAGE_DIGEST
    ↓
Output
```

---

# Output Naming Convention

Recommended:

```text
image-tag
image-digest
artifact-name
deployment-url
release-version
helm-revision
commit-sha
```

Use names that clearly describe the data.

---

# Output Documentation

For reusable workflows, document:

```text
Output Name
Description
Format
Producer
Consumer
Security Sensitivity
```

Example:

| Output | Description | Example |
|---|---|---|
| image-tag | Immutable image tag | 8a92f31 |
| image-digest | Container digest | sha256:... |
| artifact-name | Build artifact | app.tar.gz |
| deployment-url | Application URL | https://... |

---

# Output Lifecycle

```text
Generate
   |
   ↓
Set
   |
   ↓
Expose
   |
   ↓
Consume
   |
   ↓
Validate
   |
   ↓
Deploy
```

---

# Output Failure Handling

If a step fails before producing an expected output:

```text
Output may be unavailable
```

Downstream jobs should not assume that a value exists.

Use:

```text
Job Dependencies
Conditions
Validation
Failure Handling
```

---

# Validate Critical Outputs

For production:

```text
Output Image Digest
        |
        ↓
Validate Format
        |
        ↓
Check ECR
        |
        ↓
Check Approved SHA
        |
        ↓
Deploy
```

Do not blindly deploy a value merely because a previous step produced it.

---

# Output and Approval

A validation job may produce:

```text
approved=true
```

But:

```text
Output = Data
```

not:

```text
Output = Security Boundary
```

Production approval should be implemented using proper GitHub Environment protection and organizational change controls.

---

# Outputs and Deployment Records

Capture useful metadata:

```text
Commit SHA
Image Digest
Environment
JIRA Ticket
Deployment Time
Helm Revision
Workflow Run
```

Outputs can help build this metadata.

---

# Production Traceability

```text
Git Commit
    |
    ↓
Build
    |
    ↓
Image Digest
    |
    ↓
UAT
    |
    ↓
E2E
    |
    ↓
JIRA Approval
    |
    ↓
Production
```

Every stage should be able to identify the exact artifact being promoted.

---

# Common Mistakes

### 1. Using logs instead of outputs

Use structured outputs for workflow data.

### 2. Forgetting the step ID

Outputs require the correct step reference.

### 3. Forgetting `needs`

Cross-job outputs require the appropriate job dependency.

### 4. Passing secrets through outputs

Avoid unnecessary secret propagation.

### 5. Using mutable tags

Prefer immutable identifiers.

### 6. Trusting output values blindly

Validate important production outputs.

### 7. Passing huge data through outputs

Use artifacts or appropriate storage.

### 8. Using outputs as authorization

Outputs are data, not an authorization boundary.

---

# Best Practices

- Use outputs for generated workflow data.
- Give every producing step a clear `id`.
- Use `$GITHUB_OUTPUT` to create step outputs.
- Use job outputs for cross-job communication.
- Use `needs` to establish job dependencies.
- Pass immutable image tags or digests through outputs.
- Validate critical production outputs.
- Avoid passing secrets through outputs.
- Use artifacts for large files.
- Use explicit output names.
- Document reusable workflow outputs.
- Keep outputs small and focused.
- Use outputs for deployment traceability.
- Prefer structured data over log scraping.
- Do not treat outputs as authorization controls.

---

# Production-Level Output Architecture

```text
                    Build Job
                        |
                ┌───────┴────────┐
                ↓                ↓
          Image Tag        Image Digest
                |                |
                └───────┬────────┘
                        ↓
                    Job Outputs
                        |
                        ↓
                    UAT Deploy
                        |
                        ↓
                    E2E Tests
                        |
                        ↓
                Production Gate
                        |
                        ↓
                    Production
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
name: Production Promotion

on:
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image-tag: ${{ steps.meta.outputs.tag }}
      image-digest: ${{ steps.meta.outputs.digest }}

    steps:

      - name: Generate Image Metadata
        id: meta
        run: |
          TAG="${GITHUB_SHA::7}"
          DIGEST="sha256:example"

          echo "tag=$TAG" >> "$GITHUB_OUTPUT"
          echo "digest=$DIGEST" >> "$GITHUB_OUTPUT"

  uat:

    needs: build

    runs-on: ubuntu-latest

    steps:

      - name: Deploy to UAT
        env:
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
          IMAGE_DIGEST: ${{ needs.build.outputs.image-digest }}
        run: |
          echo "Deploying $IMAGE_TAG"
          echo "Digest $IMAGE_DIGEST"

  e2e:

    needs: uat

    runs-on: ubuntu-latest

    outputs:
      passed: ${{ steps.test.outputs.passed }}

    steps:

      - name: E2E Tests
        id: test
        run: |
          echo "Running E2E tests"
          echo "passed=true" >> "$GITHUB_OUTPUT"

  production:

    needs:
      - build
      - e2e

    if: ${{ needs.e2e.outputs.passed == 'true' }}

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Production Deployment
        env:
          IMAGE_TAG: ${{ needs.build.outputs.image-tag }}
          IMAGE_DIGEST: ${{ needs.build.outputs.image-digest }}
        run: |
          echo "Deploying $IMAGE_TAG"
          echo "Immutable digest: $IMAGE_DIGEST"
```

The example demonstrates:

```text
Build
 ↓
Step Outputs
 ↓
Job Outputs
 ↓
UAT
 ↓
E2E Output
 ↓
Production
 ↓
Environment Protection
```

In a real production implementation, the image digest, test result, and approval status must come from actual trusted systems rather than placeholder values.

---

# Key Takeaways

Remember:

```text
Step Output
    ↓
steps.<id>.outputs.<name>
```

For cross-job communication:

```text
Job Output
    ↓
needs.<job>.outputs.<name>
```

Use:

```text
env
 ↓
Configuration

outputs
 ↓
Generated Data

secrets
 ↓
Sensitive Data

artifacts
 ↓
Files / Large Data
```

The most important production principle:

```text
Use outputs to create explicit,
traceable data flow between workflow components.
```

Outputs should carry the exact information required by the next stage, while security boundaries such as production approvals, permissions, and environments should remain separate.

---

# Interview Questions

## Basic

1. What are outputs in GitHub Actions?
2. Why are outputs used?
3. What is `$GITHUB_OUTPUT`?
4. What is a step output?
5. Why does a step need an `id` to expose outputs?
6. How do you reference a step output?
7. What is a job output?
8. How do you reference a job output?
9. What is the difference between outputs and environment variables?
10. What is the difference between outputs and artifacts?

## Intermediate

11. How do you create multiple outputs from one step?
12. How do you pass an output from one job to another?
13. Why is `needs` required for cross-job outputs?
14. How do you use outputs in conditions?
15. How do you generate an image tag as an output?
16. How do you pass an image digest between jobs?
17. How do you capture Terraform outputs in GitHub Actions?
18. How do you pass deployment metadata between jobs?
19. How do outputs work with reusable workflows?
20. How do you create multiline outputs?
21. How do you handle JSON outputs?
22. How do you validate an output before using it?
23. Why should outputs generally not be used to pass secrets?
24. When should you use an artifact instead of an output?

## Advanced / Production

25. Design a Build → UAT → E2E → Production workflow using job outputs.
26. How would you pass an immutable ECR image digest from build to production?
27. How would you validate that the output image actually exists in ECR?
28. How would you connect GitHub Actions outputs with ArgoCD and EKS?
29. How would you use outputs to implement JIRA change-request validation?
30. How would you pass an approved SHA through a multi-stage pipeline?
31. How would you use outputs from SonarQube, Trivy, and Veracode stages?
32. How would you design outputs for a multi-microservice deployment platform?
33. How would you prevent sensitive information from leaking through outputs?
34. How would you handle a missing or empty output in a production workflow?
35. How would you distinguish output data from authorization decisions?
36. How would you use outputs to create deployment traceability?
37. How would you capture Helm release revision as an output?
38. How would you use Terraform outputs during infrastructure provisioning followed by application deployment?
39. How would you pass build metadata to a reusable deployment workflow?
40. Design an enterprise GitHub Actions output strategy covering build metadata, security results, test results, artifact information, deployment metadata, and production promotion.