# Commit SHA

A Git commit SHA is the unique identifier assigned to a Git commit.

In DevOps and CI/CD, the commit SHA is extremely important because it provides a precise way to identify exactly which source code was used to build, test, release, and deploy an application.

A typical production traceability chain is:

```text
Developer
    |
    ↓
Commit
    |
    ↓
Commit SHA
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
Docker Image
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

# 1. What Is a Commit?

A commit represents a recorded change in a Git repository.

Example:

```bash
git commit -m "Add payment timeout handling"
```

Git creates a commit containing information such as:

```text
Author
Committer
Commit message
Timestamp
Parent commit
Repository tree
Commit metadata
```

The commit receives a unique SHA.

---

# 2. What Is a Commit SHA?

A commit SHA is the identifier Git uses to identify a specific commit.

Example:

```text
8a92f31d4f7c...
```

A full SHA is typically represented as a hexadecimal hash.

Example:

```text
8a92f31d4f7c1234567890abcdef1234567890ab
```

You will often see an abbreviated form:

```text
8a92f31
```

---

# 3. Full SHA vs Short SHA

Full SHA:

```text
8a92f31d4f7c1234567890abcdef1234567890ab
```

Short SHA:

```text
8a92f31
```

The short SHA is convenient for:

```text
Logs
Terminal output
Docker tags
Monitoring
Release messages
```

For automation and precise identification, use the full commit SHA where appropriate.

---

# 4. Get Current Commit SHA

Use:

```bash
git rev-parse HEAD
```

Example:

```text
8a92f31d4f7c1234567890abcdef1234567890ab
```

---

# 5. Get Short Commit SHA

```bash
git rev-parse --short HEAD
```

Example:

```text
8a92f31
```

---

# 6. Get Commit SHA with Git Log

```bash
git log -1
```

Example:

```text
commit 8a92f31d4f7c1234567890abcdef1234567890ab
Author: Developer
Date: ...

    Add payment timeout handling
```

---

# 7. One-Line Git Log

```bash
git log --oneline -1
```

Example:

```text
8a92f31 Add payment timeout handling
```

---

# 8. Get Previous Commit SHA

```bash
git rev-parse HEAD~1
```

Two commits before:

```bash
git rev-parse HEAD~2
```

---

# 9. Get SHA of a Specific Branch

```bash
git rev-parse main
```

Remote branch:

```bash
git rev-parse origin/main
```

---

# 10. Get SHA of Remote Main

First:

```bash
git fetch origin
```

Then:

```bash
git rev-parse origin/main
```

This gives the commit currently referenced by the remote-tracking branch.

---

# 11. Compare Branches Using SHAs

Example:

```bash
git rev-parse main
git rev-parse feature/payment
```

You can compare the resulting SHAs to determine whether the branches currently point to the same commit.

---

# 12. Commit SHA and GitHub

GitHub displays commit SHAs for commits.

Example:

```text
Commit:
8a92f31d4f7c...
```

A commit SHA allows you to identify a specific source state.

This is useful for:

```text
Code Review
Debugging
Releases
CI/CD
Auditing
Rollback
Incident Investigation
```

---

# 13. Commit SHA and Pull Request

A Pull Request can contain multiple commits.

Example:

```text
PR #125
 |
 ├── abc111
 ├── def222
 └── ghi333
```

After squash merging:

```text
main
 |
 └── xyz999
```

The final merged commit may have a different SHA.

Therefore, production traceability should record the final commit that was actually released.

---

# 14. Commit SHA and GitHub Actions

GitHub Actions provides commit information through contexts.

Example:

```yaml
${{ github.sha }}
```

This represents the commit SHA associated with the workflow event.

---

# 15. Print Commit SHA in GitHub Actions

```yaml
steps:

  - name: Show Commit SHA
    run: |
      echo "Commit SHA: ${{ github.sha }}"
```

---

# 16. Store Commit SHA in Environment Variable

```yaml
env:
  COMMIT_SHA: ${{ github.sha }}
```

Then:

```yaml
steps:

  - name: Display SHA
    run: |
      echo "Commit SHA: ${COMMIT_SHA}"
```

---

# 17. Short SHA in GitHub Actions

```yaml
steps:

  - name: Get Short SHA
    run: |
      SHORT_SHA="${GITHUB_SHA::7}"
      echo "Short SHA: ${SHORT_SHA}"
```

Example:

```text
8a92f31
```

---

# 18. GITHUB_SHA

GitHub Actions provides:

```text
GITHUB_SHA
```

as a default environment variable.

Example:

```yaml
steps:

  - name: Display SHA
    run: |
      echo "SHA: $GITHUB_SHA"
```

---

# 19. GitHub Context vs Environment Variable

GitHub Actions provides:

```yaml
${{ github.sha }}
```

and:

```bash
$GITHUB_SHA
```

Both can be used to access the workflow's commit SHA, depending on where the value is needed.

Example:

```yaml
env:
  COMMIT_SHA: ${{ github.sha }}
```

Then:

```bash
echo "$COMMIT_SHA"
```

---

# 20. Commit SHA in CI

A CI pipeline can record:

```text
Commit SHA
Branch
Pull Request
Build Number
Test Result
Security Result
Artifact
```

Example:

```text
Commit:
8a92f31

Build:
#245

Result:
SUCCESS
```

---

# 21. Commit SHA in Build Metadata

A build can expose the commit SHA.

Example:

```bash
echo "BUILD_COMMIT_SHA=${GITHUB_SHA}"
```

This can become part of:

```text
Build Metadata
Artifact Metadata
Release Metadata
Deployment Metadata
```

---

# 22. Docker Image Tag Using Commit SHA

A common CI/CD pattern is:

```text
Application
    |
    ↓
Commit SHA
    |
    ↓
Docker Image Tag
```

Example:

```bash
IMAGE_TAG="${GITHUB_SHA}"
```

Then:

```bash
docker build \
  -t "catalogue:${IMAGE_TAG}" \
  .
```

---

# 23. Short SHA Docker Tag

You can use:

```bash
SHORT_SHA="${GITHUB_SHA::7}"
```

Then:

```bash
docker build \
  -t "catalogue:${SHORT_SHA}" \
  .
```

Example:

```text
catalogue:8a92f31
```

For production traceability, consider storing the full SHA and/or immutable image digest alongside the human-friendly tag.

---

# 24. Docker Image Flow

```text
Git Commit
    |
    ↓
SHA: 8a92f31
    |
    ↓
Docker Build
    |
    ↓
catalogue:8a92f31
    |
    ↓
ECR
```

This makes it easier to determine which source revision produced an image.

---

# 25. Commit SHA and ECR

Example:

```text
Repository:
catalogue

Commit:
8a92f31

Image:
catalogue:8a92f31
```

The deployment system can associate the deployed image with the source commit.

---

# 26. Image Digest vs Commit SHA

These are different identifiers.

### Commit SHA

Identifies:

```text
Source Code
```

### Image Digest

Identifies:

```text
Container Image Content
```

Example:

```text
Commit:
8a92f31

Image Digest:
sha256:abc123...
```

A production pipeline should ideally maintain the relationship:

```text
Commit SHA
     |
     ↓
Image Digest
     |
     ↓
Deployment
```

---

# 27. Why Image Digest Matters

A Docker tag can potentially be moved.

Example:

```text
catalogue:latest
```

could point to different images over time.

An image digest is content-addressed.

Example:

```text
sha256:abc123...
```

This provides stronger immutability.

---

# 28. Recommended Production Identity

A production deployment should ideally record:

```text
Repository
Commit SHA
Build ID
Image Tag
Image Digest
Deployment ID
Environment
```

Example:

```text
Repository:
catalogue

Commit:
8a92f31

Build:
#245

Image:
catalogue:8a92f31

Digest:
sha256:abc123...

Environment:
production
```

---

# 29. Commit SHA and GitOps

Suppose the application build creates:

```text
catalogue:8a92f31
```

The GitOps repository is updated:

```yaml
image:
  repository: <ECR_REPOSITORY>
  tag: 8a92f31
```

Then:

```text
GitOps PR
    |
    ↓
Merge
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 30. GitOps Traceability

A strong traceability chain is:

```text
JIRA Ticket
     |
     ↓
Pull Request
     |
     ↓
Commit SHA
     |
     ↓
GitHub Actions
     |
     ↓
Docker Image
     |
     ↓
ECR
     |
     ↓
GitOps Commit
     |
     ↓
ArgoCD
     |
     ↓
EKS
```

This is extremely useful during production incidents.

---

# 31. Commit SHA and ArgoCD

ArgoCD manages the desired state stored in Git.

The GitOps repository commit can identify the configuration change that caused a deployment.

Example:

```text
GitOps Commit:
123abc

Application Image:
catalogue:8a92f31
```

Therefore:

```text
GitOps SHA
    ↓
Deployment Configuration

Application SHA
    ↓
Application Source
```

Both may need to be tracked.

---

# 32. Application SHA vs GitOps SHA

These are different.

### Application SHA

```text
8a92f31
```

Identifies application source.

### GitOps SHA

```text
123abc
```

Identifies deployment configuration.

Relationship:

```text
Application SHA
       |
       ↓
Docker Image
       |
       ↓
GitOps Commit
       |
       ↓
ArgoCD
       |
       ↓
EKS
```

---

# 33. Commit SHA and Terraform

Terraform changes should also be traceable to Git commits.

Example:

```text
Terraform PR
     |
     ↓
Commit SHA
     |
     ↓
terraform plan
     |
     ↓
Review
     |
     ↓
Merge
     |
     ↓
terraform apply
```

The infrastructure change can then be associated with the commit that introduced it.

---

# 34. Commit SHA and Kubernetes

For Kubernetes manifests:

```text
Commit
 ↓
SHA
 ↓
Manifest Change
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

This makes it easier to identify which Git change modified the desired state.

---

# 35. Commit SHA and Helm

Example:

```text
Git Commit
   |
   ↓
Helm Chart Change
   |
   ↓
Commit SHA
   |
   ↓
GitOps
   |
   ↓
ArgoCD
```

You can also include application version information in chart metadata where appropriate.

---

# 36. Commit SHA in Helm Values

Example:

```yaml
image:
  repository: <ECR_REPOSITORY>
  tag: "8a92f31"
```

This creates a direct association between:

```text
Deployment
 ↓
Image Tag
 ↓
Commit
```

For stronger immutability, deployments can reference image digests.

---

# 37. Commit SHA and Release

A release should be traceable to a commit.

Example:

```text
v2.5.0
 |
 ↓
Commit 8a92f31
```

This allows teams to answer:

```text
What code is in this release?
```

---

# 38. Git Tags

A release can use a Git tag:

```bash
git tag v2.5.0
```

Push:

```bash
git push origin v2.5.0
```

Then:

```text
v2.5.0
   |
   ↓
Commit SHA
   |
   ↓
Build
   |
   ↓
Artifact
```

---

# 39. Commit SHA and Release Tag

A tag points to a Git object.

You can inspect it:

```bash
git rev-list -n 1 v2.5.0
```

Example:

```text
8a92f31d4f7c...
```

This tells you which commit the release tag references.

---

# 40. Commit SHA and Rollback

Suppose production is running:

```text
Version 3
Commit:
8a92f31
```

Previous version:

```text
Version 2
Commit:
7b81e20
```

A rollback can target the previously validated artifact or GitOps state.

Conceptually:

```text
Production
   |
   ↓
Version 3
   |
   ↓
Problem
   |
   ↓
Rollback
   |
   ↓
Version 2
```

The SHA helps identify the exact source version.

---

# 41. GitOps Rollback Using Commit History

Example:

```text
GitOps:

A → B → C
        ↑
      Bad deployment
```

Revert:

```text
A → B → C → D
              ↑
         Revert C
```

ArgoCD observes the resulting desired state and reconciles it.

---

# 42. Commit SHA During Incident Response

Production incident:

```text
Users report errors
        |
        ↓
Check deployed image
        |
        ↓
Find image digest
        |
        ↓
Find image tag
        |
        ↓
Find commit SHA
        |
        ↓
Find PR
        |
        ↓
Find JIRA ticket
```

This dramatically reduces investigation time.

---

# 43. Incident Investigation Example

Suppose:

```text
Production Image:
catalogue:8a92f31
```

Find the source:

```text
8a92f31
 ↓
Commit
 ↓
PR #245
 ↓
JIRA DEV-1234
```

Then inspect:

```text
What changed?
Who reviewed it?
What tests ran?
What security checks passed?
What deployment occurred?
```

---

# 44. Commit SHA and Auditability

Commit SHA provides a strong technical identifier for source state.

An audit trail can look like:

```text
JIRA
 ↓
PR
 ↓
Commit SHA
 ↓
CI Run
 ↓
Artifact
 ↓
Deployment
 ↓
Environment
```

This is useful for enterprise DevOps.

---

# 45. Commit SHA and Build Numbers

A build system can record:

```text
Build #245
Commit 8a92f31
```

Example:

```text
Build:
245

Commit:
8a92f31
```

This allows you to correlate:

```text
GitHub Actions
+
Artifact Repository
+
Deployment System
```

---

# 46. GitHub Actions Build Metadata

Example:

```yaml
name: Build

on:
  push:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Metadata
        run: |
          echo "Repository: $GITHUB_REPOSITORY"
          echo "Commit SHA: $GITHUB_SHA"
          echo "Branch: $GITHUB_REF_NAME"
          echo "Run ID: $GITHUB_RUN_ID"
          echo "Run Number: $GITHUB_RUN_NUMBER"
```

---

# 47. Commit SHA in Docker Labels

You can include source metadata in a container image.

Example:

```dockerfile
ARG GIT_COMMIT_SHA

LABEL org.opencontainers.image.revision="${GIT_COMMIT_SHA}"
```

Build:

```bash
docker build \
  --build-arg GIT_COMMIT_SHA="${GITHUB_SHA}" \
  -t "catalogue:${GITHUB_SHA}" \
  .
```

This allows the image metadata to identify the source revision.

---

# 48. OCI Image Metadata

Container images can include metadata such as:

```text
Source
Revision
Version
Created
Description
```

Example:

```text
org.opencontainers.image.revision
```

This can connect:

```text
Container
 ↓
Commit SHA
```

---

# 49. Verify Image Metadata

Depending on your container tooling, inspect the image metadata.

Conceptually:

```bash
docker inspect <image>
```

Look for labels such as:

```text
org.opencontainers.image.revision
```

---

# 50. Commit SHA and SBOM

Software Bill of Materials can be associated with:

```text
Application
Version
Commit
Image
```

Example:

```text
Commit:
8a92f31

Image:
catalogue:8a92f31

SBOM:
catalogue-sbom.json
```

This strengthens software supply-chain traceability.

---

# 51. Commit SHA and Security Scanning

Security tools can scan an artifact produced from a specific commit.

Example:

```text
Commit
 ↓
Build
 ↓
Docker Image
 ↓
Trivy
 ↓
Security Result
```

Record:

```text
Commit SHA
Image Digest
Scan Result
```

---

# 52. Commit SHA and SonarQube

Code quality analysis can be associated with the source revision being analyzed.

Conceptually:

```text
Commit
 ↓
CI
 ↓
SonarQube
 ↓
Quality Gate
```

This allows teams to understand which code state produced the quality result.

---

# 53. Commit SHA and Veracode

Similarly:

```text
Commit
 ↓
Build
 ↓
Veracode
 ↓
Security Result
```

The source revision and build artifact should be traceable.

---

# 54. Commit SHA and Dependabot

A dependency update may arrive through a PR.

Flow:

```text
Dependabot
 ↓
PR
 ↓
CI
 ↓
Security Checks
 ↓
Merge
 ↓
Commit SHA
 ↓
Build
```

The resulting commit identifies the source state after the dependency change.

---

# 55. Commit SHA and Pull Request Review

A PR can have multiple commits:

```text
PR #250

abc111
def222
ghi333
```

After review:

```text
Squash Merge
```

Final main commit:

```text
xyz999
```

For release traceability:

```text
Production
 ↓
xyz999
 ↓
PR #250
```

---

# 56. Squash Merge and SHA

Before:

```text
Feature:
A → B → C
```

After squash:

```text
main:
A → S
```

The final SHA:

```text
S
```

is different from:

```text
A
B
C
```

Therefore the release pipeline should record the SHA of the final source state being released.

---

# 57. Rebase and SHA

Before:

```text
D = abc123
```

After rebase:

```text
D' = xyz789
```

The SHA changes.

Therefore:

```text
Rebase
 ↓
New SHA
 ↓
CI Again
```

---

# 58. Merge Commit and SHA

A merge commit creates another commit:

```text
Feature
 ↓
Merge
 ↓
Merge Commit
```

Example:

```text
Merge SHA:
123abc
```

The merge commit can represent the integrated state of the target branch.

---

# 59. Choosing the Production SHA

Depending on your merge strategy:

### Squash

Use:

```text
Squash / merged main SHA
```

### Merge Commit

Use:

```text
Merge commit SHA
```

### Rebase

Use:

```text
Final validated commit on main
```

The key principle is:

```text
Release the exact source state that was validated.
```

---

# 60. Do Not Assume HEAD Is Production

Example:

```text
main
 |
 ├── Commit A
 ├── Commit B
 └── Commit C
```

Production may still run:

```text
Commit B
```

because:

```text
Commit C
```

has not been deployed yet.

Therefore:

```text
Git HEAD
```

and:

```text
Currently Deployed Commit
```

are not necessarily the same.

---

# 61. Production Deployment Record

Record:

```text
Environment:
production

Application:
catalogue

Commit:
8a92f31

Image:
catalogue:8a92f31

Image Digest:
sha256:...

GitOps Commit:
123abc

Deployment Time:
...
```

This provides strong deployment traceability.

---

# 62. Environment Comparison

You might have:

```text
QA:
Commit A

SIT:
Commit B

UAT:
Commit B

Production:
Commit C
```

This helps identify differences between environments.

---

# 63. Promote the Same Artifact

A mature deployment model prefers:

```text
Build Once
     |
     ↓
Scan Once
     |
     ↓
Promote Same Artifact
```

Instead of:

```text
QA Build
 ↓
Production Rebuild
```

Why?

Because rebuilding can produce a different artifact.

---

# 64. Recommended Promotion Flow

```text
Commit SHA
    |
    ↓
Build
    |
    ↓
Docker Image
    |
    ↓
Security Scan
    |
    ↓
ECR
    |
    ↓
QA
    |
    ↓
SIT
    |
    ↓
UAT
    |
    ↓
Production
```

The same immutable image should ideally be promoted.

---

# 65. Commit SHA and Environment Promotion

Example:

```text
Commit:
8a92f31

Image:
catalogue:8a92f31

QA
 ↓
SIT
 ↓
UAT
 ↓
Production
```

The application source identity remains consistent.

---

# 66. Commit SHA and Rollback

A rollback should target a known-good artifact.

Example:

```text
Current:
8a92f31

Previous:
7b81e20
```

Rollback:

```text
8a92f31
   ↓
7b81e20
```

This is safer than guessing which version was previously deployed.

---

# 67. Commit SHA and Canary Deployment

Example:

```text
Current:
7b81e20

Canary:
8a92f31
```

Traffic:

```text
95% → 7b81e20
5%  → 8a92f31
```

If metrics are healthy:

```text
8a92f31
   ↓
100%
```

If unhealthy:

```text
8a92f31
   ↓
Rollback
```

---

# 68. Commit SHA and Blue-Green Deployment

Example:

```text
Blue:
7b81e20

Green:
8a92f31
```

Traffic initially goes to Blue.

After validation:

```text
Blue → Green
```

If Green fails:

```text
Green → Blue
```

The SHA identifies exactly what is running in each environment.

---

# 69. Commit SHA and Rolling Deployment

Example:

```text
Pod 1 → 8a92f31
Pod 2 → 8a92f31
Pod 3 → 7b81e20
```

During a rollout, multiple versions may temporarily exist.

Tracking image identity is therefore important.

---

# 70. Commit SHA and Kubernetes

You can expose version information as labels.

Example:

```yaml
metadata:
  labels:
    app: catalogue
    version: "8a92f31"
```

You can also use standard Kubernetes labeling conventions appropriate to your organization's deployment tooling.

---

# 71. Commit SHA in Deployment Metadata

Example:

```yaml
metadata:
  annotations:
    app.example.com/commit-sha: "8a92f31"
```

This can help during troubleshooting.

---

# 72. Query Deployment Metadata

Example:

```bash
kubectl get deployment catalogue -o yaml
```

Then inspect:

```text
annotations
labels
image
```

The goal is to identify:

```text
What version is deployed?
```

---

# 73. Verify Running Image

Example:

```bash
kubectl get pods -l app=catalogue \
  -o jsonpath='{.items[*].spec.containers[*].image}'
```

You may get:

```text
123456789.dkr.ecr.region.amazonaws.com/catalogue:8a92f31
```

This gives you the image tag currently configured on the pods.

For stronger verification, also consider the image digest.

---

# 74. Commit SHA and Kubernetes Incident

Incident:

```text
High latency
```

Check:

```text
Pod
 ↓
Image
 ↓
Tag
 ↓
Commit SHA
 ↓
PR
 ↓
Recent Changes
```

This can quickly narrow the investigation.

---

# 75. Commit SHA and Monitoring

Monitoring dashboards can expose:

```text
Application Version
Commit SHA
Image
Environment
```

Example:

```text
Service:
catalogue

Version:
8a92f31

Environment:
production
```

This allows operators to correlate:

```text
Deployment
 ↓
Version
 ↓
Metrics
```

---

# 76. Commit SHA and Logs

Applications can include version information in logs.

Example:

```text
service=catalogue
version=8a92f31
environment=production
```

Then during an incident:

```text
Error
 ↓
Version
 ↓
Commit
```

---

# 77. Commit SHA and ELK

In an ELK-based observability stack, useful fields can include:

```text
service
environment
version
commit_sha
```

Example:

```text
service: payment
environment: production
commit_sha: 8a92f31
```

This allows log filtering by deployment version.

---

# 78. Commit SHA and Prometheus

Applications can expose a version metric.

Conceptually:

```text
application_info{
  version="8a92f31"
}
```

This can help correlate:

```text
Version
+
Metrics
```

Use labels carefully to avoid excessive cardinality.

---

# 79. Commit SHA and Grafana

A Grafana dashboard can show:

```text
Service
Version
Environment
Deployment
```

Example:

```text
Payment Service
Version: 8a92f31
Environment: Production
```

This makes release correlation easier.

---

# 80. Commit SHA and DevSecOps

A mature DevSecOps pipeline maintains traceability:

```text
Commit
 ↓
Build
 ↓
SAST
 ↓
SCA
 ↓
Container Scan
 ↓
Artifact
 ↓
Deployment
```

Examples:

```text
SonarQube
Trivy
Veracode
Dependabot
```

The commit SHA connects these stages.

---

# 81. Commit SHA and Security Gate

Example:

```text
Commit:
8a92f31
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
Security Gate
```

If the security gate fails:

```text
Deployment blocked
```

---

# 82. Commit SHA and Software Supply Chain

A secure supply chain should answer:

```text
Where did this artifact come from?
```

Example:

```text
Commit SHA
    ↓
Build
    ↓
Image
    ↓
SBOM
    ↓
Security Scan
    ↓
Registry
    ↓
Deployment
```

This is part of software supply-chain security.

---

# 83. Commit SHA and Build Provenance

Build provenance describes where an artifact originated.

Conceptually:

```text
Source Repository
       ↓
Commit SHA
       ↓
Build System
       ↓
Artifact
```

The stronger the provenance information, the easier it is to verify the artifact's origin.

---

# 84. Commit SHA and CI/CD Security

Do not trust only:

```text
Branch Name
```

For example:

```text
main
```

can move forward.

Instead, record:

```text
Exact Commit SHA
```

This identifies the specific source state.

---

# 85. Why `latest` Is Weak for Traceability

Example:

```text
catalogue:latest
```

Today:

```text
→ Commit A
```

Tomorrow:

```text
→ Commit B
```

The tag is mutable.

Better:

```text
catalogue:8a92f31
```

and preferably verify the digest.

---

# 86. Commit SHA as Docker Tag

Example:

```bash
IMAGE="catalogue"
TAG="${GITHUB_SHA}"

docker build \
  -t "${IMAGE}:${TAG}" \
  .
```

Then:

```bash
docker push "${IMAGE}:${TAG}"
```

---

# 87. GitHub Actions Example

```yaml
name: Build Image

on:
  push:
    branches:
      - main

env:
  IMAGE_NAME: catalogue

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build Image
        run: |
          docker build \
            -t "${IMAGE_NAME}:${GITHUB_SHA}" \
            .

      - name: Display Image
        run: |
          echo "${IMAGE_NAME}:${GITHUB_SHA}"
```

---

# 88. ECR Example

Conceptually:

```bash
IMAGE_URI="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/catalogue"

docker build \
  -t "${IMAGE_URI}:${GITHUB_SHA}" \
  .

docker push \
  "${IMAGE_URI}:${GITHUB_SHA}"
```

Production pipelines should also implement:

```text
Authentication
Least Privilege
Security Scanning
Immutable Promotion
```

---

# 89. GitOps Update

Example:

```yaml
image:
  repository: <ECR_REPOSITORY>
  tag: "8a92f31"
```

Then:

```text
GitOps PR
 ↓
Review
 ↓
Merge
 ↓
ArgoCD
```

---

# 90. Commit SHA and Deployment Verification

After deployment:

```text
Expected SHA:
8a92f31
```

Check the running image:

```text
Running:
8a92f31
```

If they differ:

```text
Investigate
```

The expected and actual deployed versions should be traceable.

---

# 91. End-to-End Traceability Example

```text
JIRA:
DEV-1234

       ↓

PR:
#245

       ↓

Commit:
8a92f31

       ↓

GitHub Actions:
Run #1250

       ↓

Docker:
catalogue:8a92f31

       ↓

ECR:
sha256:abc123...

       ↓

GitOps:
Commit 123abc

       ↓

ArgoCD:
Application Sync

       ↓

EKS:
Production
```

This is a strong production traceability model.

---

# 92. Incident Response Workflow

```text
Production Incident
        |
        ↓
Identify Service
        |
        ↓
Identify Running Image
        |
        ↓
Identify Image Digest
        |
        ↓
Identify Commit SHA
        |
        ↓
Identify PR
        |
        ↓
Identify JIRA
        |
        ↓
Review Changes
        |
        ↓
Rollback / Fix
```

---

# 93. Commit SHA and Change Failure Rate

If a deployment causes a failure:

```text
Deployment
 ↓
Commit SHA
 ↓
Incident
```

The organization can associate the failure with the change.

This supports DevOps metrics such as:

```text
Change Failure Rate
```

---

# 94. Commit SHA and Deployment Frequency

Each deployment can record:

```text
Commit SHA
Deployment Time
Environment
Result
```

This helps calculate deployment metrics accurately.

---

# 95. Commit SHA and Time to Restore

During an incident:

```text
Failed Commit
 ↓
Identify Previous Known-Good SHA
 ↓
Rollback
 ↓
Service Restored
```

The exact SHA can reduce ambiguity during recovery.

---

# 96. Production Commit Policy

A strong policy is:

```text
Every production deployment
must identify:

☐ Repository
☐ Commit SHA
☐ Build ID
☐ Artifact
☐ Image Digest
☐ GitOps Commit
☐ Environment
☐ Deployment Time
```

---

# 97. Common Mistakes

### 1. Using Only `latest`

```text
catalogue:latest
```

does not provide strong version traceability.

### 2. Not Recording the Final Merge SHA

Especially important when using:

```text
Squash
Rebase
Merge Commit
```

### 3. Confusing Commit SHA and Image Digest

They identify different objects.

### 4. Assuming Main HEAD Is Production

Production may be running an earlier version.

### 5. Rebuilding During Promotion

This can produce a different artifact.

### 6. Losing GitOps Commit Information

For GitOps, track both:

```text
Application SHA
GitOps SHA
```

### 7. Logging Only the Version Tag

Prefer recording enough information to map:

```text
Tag → Digest → Commit
```

### 8. Not Tracking Rollbacks

Record which known-good version was restored.

---

# 98. Best Practices

```text
☐ Record commit SHA in every build
☐ Use immutable artifact references
☐ Prefer commit-based image tags
☐ Track image digests
☐ Track GitOps commits
☐ Track deployment versions
☐ Maintain JIRA → PR → SHA traceability
☐ Store build metadata
☐ Include source revision in container metadata
☐ Re-run CI after rebasing
☐ Do not rely only on branch names
☐ Do not rely only on latest tags
☐ Record rollback versions
☐ Correlate deployments with monitoring
☐ Maintain audit records
```

---

# 99. Interview Questions

## Basic

1. What is a Git commit SHA?
2. What is the difference between a full SHA and short SHA?
3. How do you get the current commit SHA?
4. What is `git rev-parse HEAD`?
5. What is `GITHUB_SHA`?
6. How do you get the short SHA in GitHub Actions?
7. Why is a commit SHA useful in CI/CD?
8. What is the difference between a branch and a commit SHA?
9. What is the difference between a commit SHA and a Git tag?
10. Does a commit SHA change when a commit is rebased?

## Intermediate

11. How would you use a commit SHA as a Docker image tag?
12. Why is a commit SHA better than `latest` for traceability?
13. What is the difference between commit SHA and image digest?
14. How would you associate an ECR image with a Git commit?
15. How would you record commit information in GitHub Actions?
16. How would you expose commit SHA in a Docker image?
17. How would you use commit SHA in Kubernetes deployment metadata?
18. How would you trace a production pod back to a Git commit?
19. How would you associate a GitOps commit with an application commit?
20. How would you use commit SHA during incident investigation?
21. How does squash merging affect commit SHA?
22. How does rebase affect commit SHA?
23. How does a merge commit affect commit SHA?
24. Why should CI run again after a rebase?
25. How would you use Git tags with commit SHAs?

## Advanced / Production

26. Design a complete source-to-production traceability system using commit SHA.
27. How would you trace JIRA → PR → commit → GitHub Actions → Docker → ECR → GitOps → ArgoCD → EKS?
28. How would you distinguish application SHA from GitOps SHA?
29. How would you maintain immutable artifact promotion across QA, SIT, UAT, and production?
30. Why should you record image digest in addition to commit SHA?
31. How would you identify the exact commit running in production during an incident?
32. How would you design rollback using known-good commit and artifact identities?
33. How would you integrate commit SHA with SonarQube, Trivy, Veracode, and Dependabot?
34. How would you expose application version information through Kubernetes metadata?
35. How would you correlate commit SHA with Prometheus, Grafana, and ELK?
36. How would you implement commit-based Docker tagging in GitHub Actions?
37. How would you ensure the artifact deployed to production is the same artifact tested in QA?
38. How would you prevent mutable tags from causing production traceability problems?
39. How would you track application and infrastructure commits separately?
40. How would you design commit traceability for a Terraform + Kubernetes + ArgoCD environment?
41. A production deployment is failing. You know only the ECR image digest. How would you trace it back to the source commit?
42. A GitOps deployment points to image tag `8a92f31`, but the running pod shows a different digest. How would you investigate?
43. A developer rebased a feature branch after CI passed. Which SHA should be trusted for release and why?
44. Your organization uses squash merges. How would you preserve traceability from individual developer commits to the production commit?
45. Design an enterprise-grade commit-to-production traceability architecture covering GitHub, GitHub Actions, JIRA, SonarQube, Trivy, Veracode, ECR, Terraform, Kubernetes, GitOps, ArgoCD, EKS, observability, rollback, and audit requirements.