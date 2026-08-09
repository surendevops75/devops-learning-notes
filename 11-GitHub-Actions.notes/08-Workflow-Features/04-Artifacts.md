# GitHub Actions Artifacts

Artifacts are files or directories produced by a workflow that you want to store, download, share between jobs, or retain after the workflow finishes.

Typical artifacts include:

```text
Build Packages
Test Reports
Coverage Reports
JUnit Reports
Security Reports
Logs
Terraform Plans
Deployment Manifests
Application Binaries
```

Artifacts are different from caches.

```text
Cache
  ↓
Speed up future workflow runs

Artifact
  ↓
Preserve and transfer workflow outputs
```

---

# Why Artifacts Matter

A GitHub Actions runner is temporary.

Example:

```text
Job
 ↓
Build
 ↓
Runner
 ↓
Job finishes
 ↓
Runner is removed
```

Files created inside the runner do not automatically become available after the job finishes.

Artifacts allow important files to be persisted.

---

# Basic Artifact Upload

Use:

```yaml
- name: Upload Artifact
  uses: actions/upload-artifact@v4
  with:
    name: build-output
    path: dist/
```

This uploads the contents of:

```text
dist/
```

as an artifact named:

```text
build-output
```

---

# Basic Artifact Download

Use:

```yaml
- name: Download Artifact
  uses: actions/download-artifact@v4
  with:
    name: build-output
```

The artifact can then be used by another job.

---

# Artifact Flow

```text
Build Job
    |
    ↓
Generate Files
    |
    ↓
Upload Artifact
    |
    ↓
GitHub Artifact Storage
    |
    ↓
Download Artifact
    |
    ↓
Deployment Job
```

---

# Why Use Artifacts Between Jobs?

Each GitHub Actions job normally runs on its own runner.

Example:

```text
Build Job
   |
   └── Runner A

Test Job
   |
   └── Runner B

Deploy Job
   |
   └── Runner C
```

Files created in Runner A are not automatically present on Runner B or C.

Artifacts provide a way to transfer those files.

---

# Build → Deploy

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Build
        run: |
          npm ci
          npm run build

      - name: Upload Build
        uses: actions/upload-artifact@v4
        with:
          name: application-build
          path: dist/

  deploy:

    needs: build
    runs-on: ubuntu-latest

    steps:

      - name: Download Build
        uses: actions/download-artifact@v4
        with:
          name: application-build

      - name: Deploy
        run: |
          ./deploy.sh
```

Flow:

```text
Build
  ↓
Upload
  ↓
Artifact
  ↓
Download
  ↓
Deploy
```

---

# Artifact Name

Example:

```yaml
name: application-build
```

The artifact name should clearly identify what it contains.

Good:

```text
catalogue-build
test-results
terraform-plan
security-report
deployment-manifest
```

Avoid ambiguous names such as:

```text
output
file
data
```

---

# Artifact Path

Example:

```yaml
path: dist/
```

You can also upload individual files:

```yaml
path: app.jar
```

Or multiple paths:

```yaml
path: |
  dist/
  reports/
```

---

# Multiple Artifacts

Example:

```yaml
- name: Upload Application
  uses: actions/upload-artifact@v4
  with:
    name: application
    path: dist/

- name: Upload Test Reports
  uses: actions/upload-artifact@v4
  with:
    name: test-reports
    path: reports/
```

The workflow produces:

```text
application
test-reports
```

---

# Artifact Structure

Example:

```text
Workflow Run
│
├── application
│   └── dist/
│
├── test-reports
│   └── reports/
│
└── security-reports
    └── security/
```

This makes troubleshooting and auditing easier.

---

# Artifact Retention

Artifacts do not need to be retained forever.

You can configure retention:

```yaml
- name: Upload Artifact
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: reports/
    retention-days: 7
```

The exact maximum retention available depends on the GitHub plan and repository/organization policy.

---

# Why Retention Matters

Artifacts consume storage.

If every workflow stores:

```text
Large Build
Large Logs
Large Reports
```

forever, storage usage can grow quickly.

Use retention appropriate to the artifact's purpose.

---

# Artifact Retention Strategy

Example:

```text
PR Test Reports
    → Short retention

Build Artifacts
    → Moderate retention

Production Release Evidence
    → Longer retention if required by policy
```

Retention should follow organizational requirements.

---

# Artifact vs Cache

This is one of the most important interview topics.

### Cache

```text
Purpose:
Performance
```

Examples:

```text
npm cache
Maven dependencies
Docker build cache
Terraform providers
```

### Artifact

```text
Purpose:
Persist and transfer outputs
```

Examples:

```text
JAR
ZIP
Reports
Terraform plan
Logs
```

---

# Artifact vs Docker Registry

For containerized applications:

```text
Artifact
  ↓
Workflow output / reports / packages

ECR
  ↓
Container image registry
```

Do not use GitHub Actions artifacts as a replacement for a production container registry when ECR is your designated image repository.

---

# Artifact vs Git

Use Git for:

```text
Source Code
Workflow Files
Terraform Code
Helm Charts
GitOps Manifests
Configuration
```

Use artifacts for generated outputs:

```text
Build Package
Reports
Logs
Test Results
Generated Files
```

---

# Artifact vs Terraform State

Never confuse:

```text
Artifact
```

with:

```text
Terraform State
```

Terraform state should remain in the configured remote backend.

Example:

```text
Terraform
   |
   ↓
S3 Remote State
```

An artifact may contain:

```text
Terraform Plan
```

but it should not replace the Terraform state backend.

---

# Terraform Plan as Artifact

A production pipeline can create a plan:

```bash
terraform plan -out=tfplan
```

Then upload it:

```yaml
- name: Upload Terraform Plan
  uses: actions/upload-artifact@v4
  with:
    name: terraform-plan
    path: tfplan
```

The artifact can then be passed to a later job.

---

# Why Store Terraform Plan?

It can provide:

```text
Plan Review
Auditability
Job-to-Job Transfer
Promotion Evidence
```

However, treat Terraform plan files as potentially sensitive because they can contain infrastructure information and, depending on configuration/provider behavior, sensitive values.

Protect access accordingly.

---

# Terraform Plan → Apply

Conceptual flow:

```text
Terraform Code
      |
      ↓
terraform plan
      |
      ↓
Upload Plan
      |
      ↓
Approval
      |
      ↓
Download Plan
      |
      ↓
terraform apply
```

---

# Important Terraform Principle

Do not generate a new plan after approval and assume it is identical to the reviewed plan.

A safer promotion design is:

```text
Generate Plan
     |
     ↓
Review / Approval
     |
     ↓
Use the Approved Plan
```

provided the plan remains valid for the intended state and deployment process.

---

# Test Reports as Artifacts

Example:

```yaml
- name: Run Tests
  run: |
    mvn test

- name: Upload Test Reports
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: test-reports
    path: |
      target/surefire-reports/
```

Using:

```yaml
if: ${{ always() }}
```

allows report collection even when the test step fails, subject to the job still being able to execute the upload step.

---

# Why Upload Reports After Failure?

Suppose:

```text
Tests
  |
  ↓
Failure
```

You still need:

```text
Test Reports
Logs
Screenshots
Diagnostics
```

to understand the failure.

---

# Test Evidence

Example:

```text
Test Job
   |
   ├── Unit Tests
   ├── Integration Tests
   └── E2E Tests
          |
          ↓
      Test Reports
          |
          ↓
       Artifact
```

---

# JUnit Reports

Example:

```yaml
- name: Upload JUnit Reports
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: junit-results
    path: '**/junit*.xml'
```

The exact file pattern depends on the testing framework.

---

# Coverage Reports

Example:

```yaml
- name: Upload Coverage
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: coverage-report
    path: coverage/
```

---

# Security Reports

Security tools can produce reports.

Example:

```text
SonarQube
Trivy
Veracode
```

Reports can be uploaded when they are appropriate to retain.

Example:

```yaml
- name: Upload Security Report
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: security-report
    path: reports/security/
```

Do not upload credentials or sensitive scanner configuration.

---

# Logs as Artifacts

For complex troubleshooting:

```yaml
- name: Collect Logs
  if: ${{ always() }}
  run: |
    mkdir -p logs
    kubectl logs deployment/catalogue -n production > logs/catalogue.log

- name: Upload Logs
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: deployment-logs
    path: logs/
```

Make sure the logs do not contain secrets or sensitive information.

---

# Kubernetes Diagnostics

Production deployment troubleshooting may collect:

```text
Pod Description
Events
Pod Logs
Deployment Status
Helm Status
```

Example:

```bash
kubectl get pods -n production > logs/pods.txt

kubectl get events \
  -n production \
  --sort-by=.lastTimestamp > logs/events.txt
```

Then:

```yaml
- name: Upload Diagnostics
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: kubernetes-diagnostics
    path: logs/
```

---

# Artifact Download to Custom Path

Example:

```yaml
- name: Download Build
  uses: actions/download-artifact@v4
  with:
    name: application-build
    path: ./release
```

The files are placed under:

```text
./release
```

---

# Download All Artifacts

You can download all artifacts from a workflow run.

Example:

```yaml
- name: Download Artifacts
  uses: actions/download-artifact@v4
```

Use this only when you actually need all artifacts.

For production workflows, downloading only the required artifact is generally clearer.

---

# Download Specific Artifact

Recommended:

```yaml
- name: Download Application
  uses: actions/download-artifact@v4
  with:
    name: application-build
    path: ./release
```

This makes the workflow dependency explicit.

---

# Artifact Naming by Service

For microservices:

```text
user-build
catalogue-build
cart-build
orders-build
payment-build
inventory-build
notification-build
```

This can help when multiple services are built in the same workflow.

---

# Artifact Naming by Version

You can include version information in artifact names.

Example:

```yaml
name: catalogue-${{ github.sha }}
```

This provides traceability.

However, avoid creating unnecessarily many duplicate artifacts if the workflow already provides immutable run/commit metadata.

---

# Artifact and Commit SHA

A strong traceability pattern is:

```text
Git Commit SHA
       |
       ↓
Build
       |
       ↓
Artifact
       |
       ↓
Container Image
       |
       ↓
Deployment
```

Example:

```text
SHA:
8a92f31...
```

Use the immutable SHA as part of release metadata.

---

# Artifact and Container Image

For your EKS platform:

```text
Source
  |
  ↓
Git SHA
  |
  ↓
Docker Build
  |
  ↓
ECR
  |
  ↓
Image Digest
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

A build artifact may be useful for reports and deployment evidence, but ECR remains the container image registry.

---

# Build Once, Promote Many

A production-grade principle:

```text
Build Once
    |
    ↓
Test
    |
    ↓
Security
    |
    ↓
Promote Same Artifact
```

Do not rebuild the application separately for:

```text
QA
UAT
Production
```

if your release process is designed around immutable artifacts.

---

# Artifact Promotion

Example:

```text
Build
  |
  ↓
Artifact
  |
  ├── QA
  |
  ├── UAT
  |
  └── Production
```

The same immutable release output should be promoted.

---

# Artifact Integrity

For sensitive release workflows, consider verifying:

```text
Artifact Name
Artifact Source
Commit SHA
Digest
Expected Version
```

Do not trust an artifact merely because its filename looks correct.

---

# Artifact Security

Artifacts may contain:

```text
Source Code
Configuration
Build Output
Logs
Reports
```

Some of these may contain sensitive information.

Before uploading:

```text
Check contents
Remove secrets
Remove tokens
Remove private keys
Remove credentials
```

---

# Do Not Upload `.env`

Avoid:

```yaml
path: .env
```

if the file contains:

```text
Passwords
API Keys
AWS Credentials
Database Credentials
Tokens
```

---

# Do Not Upload AWS Credentials

Never upload:

```text
~/.aws/credentials
```

as an artifact.

Use:

```text
GitHub OIDC
AWS IAM
Environment protection
```

for secure authentication.

---

# Artifact and OIDC

Correct:

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
AWS
```

Do not create an artifact containing long-lived AWS credentials.

---

# Artifact Access

Artifacts can contain sensitive operational information.

Control access through:

```text
Repository permissions
Organization policies
Workflow permissions
Environment controls
Artifact retention
```

Use the minimum access necessary.

---

# Artifact Compression

Artifacts are generally packaged for transfer/storage.

You do not always need to manually compress them.

If your output is already compressed:

```text
.zip
.tar.gz
.jar
```

additional compression may provide little benefit.

---

# Large Artifacts

Avoid unnecessarily large artifacts.

Examples:

```text
node_modules/
.git/
Docker image tarballs
Large dependency caches
Temporary files
```

unless there is a specific reason to retain them.

---

# Better Artifact Selection

Instead of:

```yaml
path: .
```

prefer:

```yaml
path: |
  dist/
  reports/
```

Upload only what is required.

---

# Artifact Retention Strategy

Example:

```text
PR reports
   → 7 days

CI build artifacts
   → 7–30 days

Release evidence
   → Policy-driven

Production diagnostics
   → Short retention unless required
```

Use your organization's compliance and operational requirements to determine actual retention.

---

# Artifact Storage Considerations

Artifacts consume GitHub storage.

Large or long-retained artifacts can increase storage usage.

Optimize:

```text
Artifact size
Retention
Number of artifacts
Duplicate artifacts
```

---

# Artifact and Caching

Do not confuse:

```text
Cache
```

with:

```text
Artifact
```

Example:

```text
Maven Dependencies
      ↓
Cache

JAR Build Output
      ↓
Artifact

Docker Image
      ↓
ECR

Terraform State
      ↓
S3 Backend
```

---

# Artifact and Reusable Workflows

Reusable workflows can produce artifacts.

Example:

```text
Application Workflow
       |
       ↓
Reusable Build Workflow
       |
       ↓
Build Artifact
       |
       ↓
Application Workflow
```

This allows centralized CI logic.

---

# Artifact in Matrix Builds

Suppose:

```yaml
strategy:
  matrix:
    service:
      - user
      - catalogue
      - cart
```

Each matrix job can upload its own artifact.

Example:

```yaml
- name: Upload Build
  uses: actions/upload-artifact@v4
  with:
    name: ${{ matrix.service }}-build
    path: dist/
```

Result:

```text
user-build
catalogue-build
cart-build
```

---

# Matrix Artifact Collision

Avoid using the same artifact name for multiple matrix jobs when you intend to keep separate outputs.

Bad:

```yaml
name: build
```

for every matrix job.

Better:

```yaml
name: ${{ matrix.service }}-build
```

---

# Matrix Architecture

```text
                 Build
                   |
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      User     Catalogue     Cart
        |          |          |
        ↓          ↓          ↓
      Artifact   Artifact   Artifact
```

---

# Artifact Merge

Sometimes multiple jobs need to contribute files to one logical artifact.

Design this carefully.

Example:

```text
Service A → report A
Service B → report B
Service C → report C
```

You may either:

```text
Keep separate artifacts
```

or:

```text
Aggregate them in a final job
```

Separate artifacts often provide clearer ownership and debugging.

---

# Artifact Aggregation

Example architecture:

```text
Test A
  ↓
Report A
  ↓
Artifact

Test B
  ↓
Report B
  ↓
Artifact

Test C
  ↓
Report C
  ↓
Artifact

      ↓

Aggregate Job
      ↓
Combined Report
```

---

# Artifact Download in Aggregation Job

Conceptually:

```yaml
- name: Download Artifacts
  uses: actions/download-artifact@v4
  with:
    path: reports/
```

Then:

```yaml
- name: Generate Summary
  run: |
    ./generate-summary.sh
```

---

# Artifact and Deployment

A deployment job can consume a build artifact.

Example:

```text
Build
 ↓
Artifact
 ↓
Approval
 ↓
Deploy
```

This helps separate:

```text
Build process
```

from:

```text
Deployment process
```

---

# Production Artifact Promotion

Example:

```text
Build
   |
   ↓
Artifact
   |
   ↓
Security
   |
   ↓
UAT
   |
   ↓
Approval
   |
   ↓
Production
```

The release process should preserve traceability between:

```text
Commit
Artifact
Image
Deployment
```

---

# Artifact and Change Request

For a production change request:

```text
JIRA Ticket
     |
     ↓
Commit SHA
     |
     ↓
Build Artifact
     |
     ↓
Security Results
     |
     ↓
Testing Results
     |
     ↓
Approval
     |
     ↓
Production
```

Artifacts can help preserve evidence such as:

```text
Test Reports
Security Reports
Deployment Metadata
```

---

# Artifact and JIRA

Example release evidence:

```text
JIRA:
PROJ-1234

Commit:
8a92f31

Image:
catalogue:8a92f31

Test:
PASSED

Security:
PASSED

Environment:
production
```

Do not put secrets or sensitive credentials into the evidence artifact.

---

# Artifact and DevSecOps

A production pipeline may produce:

```text
Build Artifact
Test Report
Coverage Report
SonarQube Report
Trivy Report
Veracode Report
Deployment Evidence
```

Then:

```text
Approval
   ↓
Production
```

---

# Artifact and Security Gates

Important distinction:

```text
Artifact
```

stores evidence.

It does not itself decide:

```text
PASS
```

The security tool or validation job should produce the result.

Example:

```text
Trivy
  ↓
Scan
  ↓
PASS
  ↓
Output
  ↓
Production Gate
```

Artifact:

```text
Trivy Report
```

provides evidence.

---

# Artifact and Outputs

Artifacts and job outputs are also different.

### Job Output

Best for:

```text
Small values
Flags
Identifiers
Status
```

Example:

```text
approved=true
image_digest=sha256:...
```

### Artifact

Best for:

```text
Files
Reports
Packages
Logs
Plans
```

---

# Outputs vs Artifacts

```text
Job Output
   ↓
Small piece of data
```

```text
Artifact
   ↓
File or directory
```

Example:

```text
image_digest
   → Job Output

terraform plan
   → Artifact
```

---

# Artifact and Environment

Production deployment may use:

```text
Artifact
   ↓
Validation
   ↓
GitHub Environment
   ↓
Approval
   ↓
Deploy
```

Environment protection controls the promotion process.

---

# Artifact and Concurrency

Concurrency can protect artifact-producing deployment workflows.

Example:

```text
Build
 ↓
Artifact
 ↓
Production Deployment
      |
      ↓
Concurrency
```

The artifact itself does not provide deployment serialization.

---

# Artifact and Timeout

Timeout protects workflow execution:

```text
timeout-minutes
```

Artifact preserves output:

```text
upload-artifact
```

They solve different problems.

---

# Artifact After Failure

A useful pattern:

```yaml
- name: Upload Diagnostics
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: diagnostics
    path: logs/
```

This can preserve information after failures.

---

# Failure Diagnostics

For Kubernetes deployment failures:

```text
kubectl get pods
kubectl describe pod
kubectl get events
kubectl logs
helm status
helm history
```

Save the results:

```text
logs/
```

Then upload:

```text
kubernetes-diagnostics
```

---

# Production Deployment Diagnostics

Example:

```yaml
- name: Collect Diagnostics
  if: ${{ always() }}
  run: |
    mkdir -p diagnostics

    kubectl get pods -n production \
      > diagnostics/pods.txt

    kubectl get events -n production \
      --sort-by=.lastTimestamp \
      > diagnostics/events.txt

    helm status catalogue -n production \
      > diagnostics/helm-status.txt

    helm history catalogue -n production \
      > diagnostics/helm-history.txt

- name: Upload Diagnostics
  if: ${{ always() }}
  uses: actions/upload-artifact@v4
  with:
    name: production-diagnostics
    path: diagnostics/
```

Before using this in production, ensure the captured output does not expose secrets or sensitive data.

---

# Artifact Naming Strategy

Recommended pattern:

```text
<service>-<purpose>-<version>
```

Example:

```text
catalogue-build-8a92f31
catalogue-test-report-8a92f31
catalogue-security-report-8a92f31
```

Alternatively, use a stable artifact name within a workflow and rely on the workflow run/commit metadata for version traceability.

---

# Artifact Traceability

A release should be traceable:

```text
Git SHA
   ↓
Workflow Run
   ↓
Artifact
   ↓
Image Digest
   ↓
GitOps Commit
   ↓
ArgoCD
   ↓
EKS
```

This is important for:

```text
Auditing
Rollback
Troubleshooting
Incident Response
Compliance
```

---

# Artifact and Rollback

Artifacts can help with rollback evidence.

Example:

```text
Current Version:
8a92f31

Previous Version:
5bd23aa
```

The actual rollback mechanism for your application may instead use:

```text
ECR image
Helm revision
GitOps commit
```

The artifact is supporting evidence, not necessarily the rollback mechanism.

---

# Helm Rollback

For Helm:

```bash
helm history catalogue -n production
```

Then a controlled rollback may use:

```bash
helm rollback catalogue <REVISION> -n production
```

Artifacts can preserve deployment diagnostics and release metadata around that operation.

---

# GitOps Rollback

With ArgoCD:

```text
Git Commit
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

Rollback should generally be handled through the GitOps process and repository history according to the team's GitOps strategy.

Artifacts can preserve:

```text
Deployment Evidence
Test Results
Security Reports
```

---

# Artifact Lifecycle

```text
Create
  ↓
Upload
  ↓
Store
  ↓
Download
  ↓
Use
  ↓
Expire
```

Design each stage intentionally.

---

# Artifact Checklist

```text
☐ Correct artifact path
☐ Meaningful artifact name
☐ Required files only
☐ No secrets
☐ No credentials
☐ Appropriate retention
☐ Clear traceability
☐ Artifact used for correct purpose
☐ Large files minimized
☐ Matrix artifacts uniquely named
☐ Failure diagnostics preserved
☐ Sensitive reports access-controlled
```

---

# Common Mistakes

### 1. Uploading the entire workspace

Creates huge artifacts.

### 2. Uploading secrets

Major security risk.

### 3. Using artifacts as cache

Wrong storage model.

### 4. Using artifacts as Terraform state

Incorrect and unsafe.

### 5. Using artifacts as ECR replacement

Container images should remain in the designated registry.

### 6. No retention policy

Storage grows unnecessarily.

### 7. Same artifact name for matrix jobs

Can make outputs difficult to distinguish.

### 8. No traceability

You should know which commit produced the artifact.

### 9. Uploading only on success

You may lose valuable failure diagnostics.

### 10. Treating artifact as approval

Artifact storage does not equal deployment authorization.

---

# Best Practices

- Upload only required files.
- Use clear artifact names.
- Use appropriate retention periods.
- Preserve test and security evidence.
- Upload diagnostics after failures when useful.
- Never upload secrets.
- Do not use artifacts as Terraform state.
- Do not use artifacts as a container registry.
- Use immutable version metadata.
- Keep artifacts traceable to the Git commit.
- Use separate artifacts for logically different outputs.
- Give matrix artifacts unique names when needed.
- Protect sensitive reports.
- Keep production promotion based on validated immutable artifacts.
- Use artifacts as evidence and transfer mechanisms, not authorization mechanisms.

---

# Production-Level CI/CD Architecture

```text
                         Git Commit
                             |
                             ↓
                       GitHub Actions
                             |
              ┌──────────────┼──────────────┐
              ↓              ↓              ↓
            Build          Tests         Security
              |              |              |
              ↓              ↓              ↓
         Build Artifact   Test Report   Security Report
              |              |              |
              └──────────────┼──────────────┘
                             ↓
                       Artifact Storage
                             |
                             ↓
                     Immutable Image
                             |
                             ↓
                            ECR
                             |
                             ↓
                       UAT Validation
                             |
                             ↓
                         Approval
                             |
                             ↓
                      GitOps Promotion
                             |
                             ↓
                           ArgoCD
                             |
                             ↓
                            EKS
```

---

# Key Takeaways

Artifacts are used to:

```text
Store workflow outputs
Transfer files between jobs
Preserve test reports
Preserve security reports
Preserve diagnostics
Transfer build packages
Support release evidence
```

The core commands are:

```yaml
uses: actions/upload-artifact@v4
```

and:

```yaml
uses: actions/download-artifact@v4
```

The most important distinction is:

```text
Cache
→ Performance

Artifact
→ Workflow output / transfer / retention

ECR
→ Container images

Git
→ Source and GitOps desired state

S3
→ Terraform state
```

For production:

```text
Build
 ↓
Artifact
 ↓
Security
 ↓
Validation
 ↓
Approval
 ↓
Immutable Image
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

Artifacts provide **traceability and evidence**, but they should not be confused with authentication, authorization, deployment state, or infrastructure state.

---

# Interview Questions

## Basic

1. What are artifacts in GitHub Actions?
2. Why are artifacts needed?
3. What is `actions/upload-artifact`?
4. What is `actions/download-artifact`?
5. Why can't one job directly use files created by another job?
6. How do you upload a directory as an artifact?
7. How do you upload multiple files?
8. How do you download a specific artifact?
9. What is artifact retention?
10. Why should artifacts have meaningful names?

## Intermediate

11. What is the difference between an artifact and a cache?
12. What is the difference between an artifact and a job output?
13. How would you pass a build from one job to another?
14. How would you store test reports?
15. How would you preserve logs after a failed deployment?
16. How would you upload JUnit reports?
17. How would you upload coverage reports?
18. How would you store Terraform plans as artifacts?
19. Why should Terraform state not be stored as an artifact?
20. How would you handle artifacts in matrix builds?
21. How would you name artifacts for multiple microservices?
22. How would you set artifact retention?
23. Why should you upload diagnostics with `always()`?
24. How would you protect sensitive artifact contents?
25. What information should be included for artifact traceability?

## Advanced / Production

26. Design a Build → Test → Security → Artifact → Deploy pipeline.
27. How would you implement Build Once, Promote Many?
28. How would you connect Git SHA, artifact, ECR image, GitOps commit, ArgoCD, and EKS?
29. How would you use artifacts to preserve DevSecOps evidence?
30. How would you store SonarQube, Trivy, and Veracode reports?
31. How would you prevent secrets from being uploaded as artifacts?
32. How would you design artifact retention for production release evidence?
33. How would you handle a large artifact?
34. How would you decide whether something should be a cache, artifact, ECR image, Git file, or Terraform state?
35. How would you use artifacts to transfer a Terraform plan from plan to apply?
36. Why should the approved Terraform plan be used rather than generating a completely new plan after approval?
37. How would you preserve Kubernetes diagnostics when a Helm deployment fails?
38. How would you design artifact handling for a multi-service EKS platform?
39. How would you protect artifact-based workflows from supply-chain risks?
40. How would you ensure an artifact corresponds to the approved Git commit?
41. How would you use artifacts as release evidence for a JIRA change request?
42. How would you design artifact handling for GitHub Actions + ECR + Helm + ArgoCD?
43. How would you handle rollback when the current deployment artifact is unhealthy?
44. How would you design artifact access and retention for a production environment?
45. Design an enterprise-grade GitHub Actions artifact architecture covering build outputs, test reports, security reports, Terraform plans, Kubernetes diagnostics, ECR images, GitOps, ArgoCD, EKS, retention, traceability, and security.