# GitOps-CI-CD-Integration

## 1. Purpose

This file explains how GitOps integrates with CI/CD in a production DevOps and DevSecOps environment.

The central principle is:

```text
CI builds, tests, scans and publishes artifacts.

GitOps CD changes desired state in Git.

Argo CD reconciles Git state into Kubernetes.
```

For the RoboShop platform, the target architecture is:

```text
Developer
   |
   v
Application Git
   |
   v
Jenkins / GitHub Actions
   |
   +--> Build
   +--> Unit Tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker Image
   |
   v
Amazon ECR
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
AWS EKS
```

This file covers:

- CI vs CD
- CI/CD vs GitOps
- Jenkins integration
- GitHub Actions integration
- GitOps repository update patterns
- Image tags and digests
- Promotion
- PR-based deployments
- Webhooks
- Argo CD refresh
- Security boundaries
- Credentials
- ECR
- Helm
- Kustomize
- ApplicationSets
- Production pipelines
- Approvals
- Rollbacks
- Failure handling
- RoboShop implementation
- Troubleshooting
- Interview questions

---

# 2. CI and CD Before GitOps

Traditional CI/CD often looks like:

```text
Developer
   |
   v
Git
   |
   v
CI/CD Server
   |
   v
kubectl / Helm
   |
   v
Kubernetes
```

The CI/CD server may directly possess Kubernetes credentials.

GitOps changes this model.

---

# 3. GitOps CI/CD Model

GitOps introduces Git as the deployment source of truth:

```text
Developer
   |
   v
Application Git
   |
   v
CI
   |
   v
Image Registry
   |
   v
GitOps Git
   |
   v
Argo CD
   |
   v
Kubernetes
```

CI does not need broad production cluster access.

---

# 4. CI Responsibility

CI should generally handle:

```text
Source checkout
Build
Unit tests
Static analysis
Dependency checks
Security scanning
Container build
Container scanning
Image publishing
Artifact metadata
Promotion automation
```

---

# 5. GitOps CD Responsibility

GitOps CD handles:

```text
Desired state
Deployment
Reconciliation
Drift detection
Health monitoring
Sync
Rollback support
Kubernetes resource lifecycle
```

Argo CD is responsible for continuously reconciling the desired state.

---

# 6. The Most Important Boundary

The boundary is:

```text
CI
 |
 | produces artifact
 v
ECR
 |
 | promotes desired version
 v
GitOps
 |
 | reconciles
 v
Argo CD
 |
 v
EKS
```

CI should not need to execute:

```bash
kubectl apply
```

against production.

---

# 7. Why Separate CI and CD?

Separation provides:

```text
Least privilege
Better auditability
Clear ownership
Reduced cluster credentials
Independent deployment reconciliation
Git-based rollback
```

---

# 8. Traditional Pipeline vs GitOps

### Traditional

```text
Jenkins
   |
   +--> build
   +--> test
   +--> scan
   +--> docker push
   +--> kubectl apply
```

### GitOps

```text
Jenkins
   |
   +--> build
   +--> test
   +--> scan
   +--> docker push
   +--> update GitOps repository
                         |
                         v
                      Argo CD
                         |
                         v
                        EKS
```

---

# 9. CI Should Not Be the Kubernetes Controller

A CI job is usually:

```text
event-driven
temporary
pipeline-oriented
```

Argo CD is:

```text
continuous
state-aware
reconciliation-oriented
```

Therefore they have different responsibilities.

---

# 10. Why Reconciliation Matters

Suppose CI deploys successfully.

Later someone executes:

```bash
kubectl scale deployment/cart --replicas=1
```

The cluster now differs from Git.

Argo CD detects:

```text
Desired: 3
Live:    1
```

and can reconcile it depending on sync/self-heal policy.

A CI pipeline that already finished would not necessarily notice.

---

# 11. Source Repositories

A production architecture commonly has:

```text
Application Git
```

and:

```text
GitOps Git
```

Example:

```text
github.com/company/roboshop-cart
github.com/company/roboshop-gitops
```

---

# 12. Application Repository

Example:

```text
roboshop-cart/
├── src/
├── tests/
├── Dockerfile
├── pom.xml
├── Jenkinsfile
└── README.md
```

This repository changes when application code changes.

---

# 13. GitOps Repository

Example:

```text
roboshop-gitops/
├── applications/
├── applicationsets/
├── projects/
├── charts/
├── environments/
└── platform/
```

This repository changes when deployment desired state changes.

---

# 14. Separation of Ownership

Application team:

```text
source code
tests
Dockerfile
application behavior
```

Platform/GitOps team:

```text
Argo CD
EKS
GitOps structure
deployment policy
environment governance
```

Shared ownership can exist for service-specific Helm values.

---

# 15. Complete RoboShop Flow

```text
Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   +--> Maven/npm build
   +--> Unit tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker build
   |
   v
Trivy image scan
   |
   v
ECR
   |
   v
GitOps update
   |
   v
Argo CD
   |
   v
EKS
   |
   v
ALB
   |
   v
Users
```

---

# 16. CI Pipeline Stages

A production pipeline can contain:

```text
Checkout
   |
Build
   |
Unit Test
   |
SAST
   |
Dependency/SCA checks
   |
Docker Build
   |
Container Scan
   |
Publish ECR
   |
Update GitOps
   |
Wait/validate DEV
   |
Promotion
```

The exact security stages depend on the tools and policies used.

---

# 17. Build Stage

For Java:

```bash
mvn clean package
```

For Node.js:

```bash
npm ci
npm test
```

For Python:

```bash
pip install -r requirements.txt
pytest
```

Use the actual build commands for each RoboShop service.

---

# 18. Unit Test Stage

CI should fail when tests fail.

Example:

```bash
mvn test
```

Do not publish a production candidate if required tests fail.

---

# 19. SonarQube Stage

SonarQube can be integrated into CI for:

```text
Code quality
Static analysis
Security findings
Quality gates
```

Example conceptual Maven command:

```bash
mvn clean verify sonar:sonar \
  -Dsonar.host.url=$SONAR_HOST_URL \
  -Dsonar.token=$SONAR_TOKEN
```

Store the token in the CI credential system.

Never hard-code it.

---

# 20. Trivy Stage

Trivy can scan:

```text
Filesystem
Dependencies
Container images
Kubernetes configuration
```

Example:

```bash
trivy fs .
```

Container:

```bash
trivy image "$IMAGE"
```

Production severity thresholds should be defined by security policy.

---

# 21. Veracode Stage

Veracode integration depends on the organization's selected Veracode product and CI integration.

The pipeline should:

```text
package artifact
   |
   v
security analysis
   |
   v
policy result
```

A failed mandatory security policy should block promotion.

---

# 22. Docker Build

Example:

```bash
docker build \
  -t "$ECR_REPOSITORY:$IMAGE_TAG" \
  .
```

Use immutable build identifiers.

Example:

```text
git-a1b2c3d
```

---

# 23. ECR Login

For AWS ECR:

```bash
aws ecr get-login-password \
  --region "$AWS_REGION" |
docker login \
  --username AWS \
  --password-stdin \
  "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"
```

The CI role should have only the ECR permissions required.

---

# 24. Push Image

```bash
docker push "$ECR_REPOSITORY:$IMAGE_TAG"
```

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart:git-a1b2c3d
```

---

# 25. Capture Image Digest

After pushing, retrieve the digest:

```bash
aws ecr describe-images \
  --repository-name roboshop/cart \
  --image-ids imageTag="$IMAGE_TAG"
```

A digest looks like:

```text
sha256:abcdef...
```

Use the digest for stronger artifact identity.

---

# 26. Tag Strategy

Recommended:

```text
git-<short-commit>
```

Example:

```text
git-a1b2c3d
```

Avoid:

```text
latest
```

for production deployment.

---

# 27. Why latest Is Dangerous

Suppose GitOps contains:

```text
cart:latest
```

The registry may later point `latest` to another image.

Git did not change.

The deployed artifact changed.

That weakens reproducibility.

---

# 28. Immutable Artifact Model

Preferred:

```text
cart:git-a1b2c3d
```

or:

```text
cart@sha256:...
```

Then:

```text
Git commit -> exact artifact
```

---

# 29. GitOps Repository Update

After ECR push:

```text
CI
 |
 v
Update image reference
 |
 v
GitOps commit/PR
```

Example:

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  tag: git-a1b2c3d
```

---

# 30. Two Main GitOps Update Models

### Model A: Direct commit

```text
CI -> GitOps commit -> Argo CD
```

### Model B: Pull request

```text
CI -> GitOps PR -> review -> merge -> Argo CD
```

PR-based promotion is generally stronger for controlled environments.

---

# 31. Direct Commit to DEV

Direct commit can be appropriate for DEV if:

```text
repository is protected
automation account is restricted
changes are limited to DEV paths
```

---

# 32. PR-Based Production Promotion

Production should commonly use:

```text
CI
 |
 v
Promotion PR
 |
 v
Automated checks
 |
 v
Required reviewers
 |
 v
Merge
 |
 v
Argo CD
```

---

# 33. Why PR-Based Promotion?

It provides:

```text
Human approval
Diff review
Security checks
Audit trail
Release discussion
Rollback reference
```

---

# 34. Promotion PR Content

A useful PR should show:

```text
Service: cart
Environment: production
Old image: git-998877
New image: git-a1b2c3d
Digest: sha256:...
Source commit: a1b2c3d
CI build: #842
Security: passed
QA: passed
```

---

# 35. GitOps Commit Example

```text
chore(cart): promote git-a1b2c3d to dev
```

QA:

```text
chore(cart): promote git-a1b2c3d to qa
```

Production:

```text
chore(cart): promote git-a1b2c3d to prod
```

Clear commit messages improve auditability.

---

# 36. Avoid Giant GitOps Commits

Bad:

```text
update all 40 services
```

Better:

```text
promote cart
promote user
promote payment
```

when operationally practical.

Focused changes simplify rollback.

---

# 37. GitOps PR Automation

CI can use:

```text
GitHub API
GitLab API
git CLI
```

to create a PR.

The CI identity should have only the repository permissions it needs.

---

# 38. CI Credentials

Do not store:

```text
GitHub personal access token
AWS secret
Sonar token
Veracode credentials
```

inside the Jenkinsfile.

Use:

```text
Jenkins Credentials
GitHub OIDC
AWS IAM role
Secret Manager
Vault
```

where supported.

---

# 39. Jenkins AWS Authentication

Prefer short-lived AWS credentials.

For example:

```text
Jenkins
   |
   v
OIDC / approved federation
   |
   v
AWS IAM role
   |
   v
ECR
```

Avoid long-lived IAM access keys on build agents.

---

# 40. GitHub Actions AWS Authentication

Prefer:

```text
GitHub Actions OIDC
        |
        v
AWS IAM role
        |
        v
ECR
```

instead of:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

stored as long-lived secrets.

---

# 41. GitHub Actions OIDC Concept

```text
GitHub workflow
      |
      v
OIDC token
      |
      v
AWS STS
      |
      v
AssumeRoleWithWebIdentity
      |
      v
Temporary credentials
```

The IAM trust policy should restrict:

```text
repository
branch/tag
environment
organization
```

as appropriate.

---

# 42. Jenkins GitOps Authentication

If Jenkins must modify the GitOps repository, use a dedicated machine identity.

Prefer:

```text
GitHub App
```

or an appropriately scoped automation credential.

Avoid using a developer's personal token.

---

# 43. GitOps Repository Write Permissions

The CI automation should ideally be allowed to modify only:

```text
environments/dev/
```

for automatic DEV updates.

Production changes should require a PR and review.

---

# 44. Environment-Specific Automation Permissions

Example:

```text
CI bot
  |
  +--> DEV: write
  |
  +--> QA: PR
  |
  +--> PROD: PR only
```

This creates a security boundary.

---

# 45. GitOps Repository Branch Model

A practical approach:

```text
main
```

contains all desired state.

Promotion changes:

```text
environments/dev
environments/qa
environments/prod
```

Long-lived branches are optional.

---

# 46. GitOps Branch Protection

Protect:

```text
main
```

with:

```text
PR required
required checks
CODEOWNERS
no force push
```

Production paths should have stronger review rules.

---

# 47. CODEOWNERS Example

```text
/environments/dev/     @devops-team
/environments/qa/      @release-team
/environments/prod/    @platform-team @release-team
/applicationsets/      @platform-team
/projects/              @platform-team
```

This is a policy example.

---

# 48. CI/CD Environment Variables

Example:

```bash
AWS_REGION=ap-south-1
ECR_REPOSITORY=roboshop/cart
IMAGE_TAG=git-a1b2c3d
GITOPS_REPO=company/roboshop-gitops
ENVIRONMENT=dev
```

Sensitive values should not be stored as ordinary source-controlled variables.

---

# 49. GitHub Actions Workflow Structure

Example:

```yaml
name: Cart CI

on:
  push:
    branches:
      - main

permissions:
  contents: read
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ap-south-1

      - name: Build
        run: mvn clean package

      - name: Build image
        run: |
          docker build \
            -t "$ECR_REGISTRY/roboshop/cart:${GITHUB_SHA::7}" .

      - name: Login to ECR
        run: |
          aws ecr get-login-password --region ap-south-1 |
          docker login \
            --username AWS \
            --password-stdin "$ECR_REGISTRY"

      - name: Push image
        run: |
          docker push \
            "$ECR_REGISTRY/roboshop/cart:${GITHUB_SHA::7}"
```

Pin actions according to your organization's supply-chain policy rather than blindly trusting mutable references.

---

# 50. GitOps Update Job

Conceptual GitHub Actions job:

```yaml
update-gitops:
  needs: build
  runs-on: ubuntu-latest

  steps:
    - name: Checkout GitOps repository
      uses: actions/checkout@v4
      with:
        repository: company/roboshop-gitops
        token: ${{ secrets.GITOPS_TOKEN }}

    - name: Update image
      run: |
        sed -i \
          "s/tag:.*/tag: git-${GITHUB_SHA::7}/" \
          environments/dev/values/cart.yaml

    - name: Commit
      run: |
        git config user.name "gitops-bot"
        git config user.email "gitops-bot@example.com"
        git add environments/dev/values/cart.yaml
        git commit -m \
          "chore(cart): promote ${GITHUB_SHA::7} to dev"
        git push
```

In production, prefer structured YAML editing or a dedicated promotion tool over fragile `sed` substitutions.

---

# 51. Why sed Can Be Dangerous

This:

```bash
sed -i "s/tag:.*/tag: ..."
```

can accidentally modify the wrong YAML field.

Better approaches include:

```text
yq
Helm-aware tooling
purpose-built release automation
```

---

# 52. yq Example

Conceptually:

```bash
yq -i \
  '.image.tag = "git-a1b2c3d"' \
  environments/dev/values/cart.yaml
```

This targets the intended YAML field.

---

# 53. GitOps Commit Validation

After modification:

```bash
git diff --check
```

Then:

```bash
helm template
```

or:

```bash
kustomize build
```

before pushing.

---

# 54. Promotion PR Validation

A PR pipeline should validate:

```text
YAML syntax
Helm rendering
Kustomize rendering
Policy
Security
Image existence
Required metadata
```

---

# 55. Image Existence Check

Before promotion:

```bash
aws ecr describe-images \
  --repository-name roboshop/cart \
  --image-ids imageTag=git-a1b2c3d
```

This prevents Git from referencing a missing artifact.

---

# 56. GitOps and ECR Race Condition

Potential problem:

```text
GitOps updated
   |
   v
Argo CD sync
   |
   v
Image not yet available
```

Avoid this by:

```text
push image first
verify registry
then update GitOps
```

---

# 57. Deployment Ordering

Correct:

```text
Build
 |
Scan
 |
Push ECR
 |
Verify ECR
 |
Update GitOps
 |
Argo CD
```

Incorrect:

```text
Update GitOps
 |
Argo CD
 |
Image push later
```

---

# 58. Argo CD Refresh

Argo CD can detect repository changes through its normal repository refresh/reconciliation behavior.

Webhooks can make refresh faster.

---

# 59. Webhook Model

```text
Git provider
    |
    | webhook
    v
Argo CD API
    |
    v
Repository refresh
    |
    v
Application comparison
    |
    v
Sync
```

The webhook accelerates awareness; it does not replace reconciliation.

---

# 60. Webhook Security

Protect webhook endpoints using:

```text
signature validation
TLS
secret/token validation
network controls
rate limiting where appropriate
```

Do not accept arbitrary unauthenticated deployment triggers.

---

# 61. Webhook vs Polling

### Polling

```text
Argo CD periodically checks Git
```

### Webhook

```text
Git immediately tells Argo CD
```

Webhooks can reduce detection latency.

Argo CD still needs reconciliation because webhooks can be:

```text
lost
delayed
duplicated
```

---

# 62. Why Reconciliation Still Matters

Suppose webhook fails.

Without reconciliation:

```text
Git changed
cluster remains old
```

With periodic reconciliation:

```text
Argo CD eventually discovers the change
```

---

# 63. Application Refresh

Commands:

```bash
argocd app get roboshop-dev
```

and:

```bash
argocd app refresh roboshop-dev
```

can be used to inspect or request refresh depending on CLI version.

---

# 64. Sync Is Different From Refresh

Important:

```text
Refresh
```

means:

```text
recalculate desired/live state
```

while:

```text
Sync
```

means:

```text
apply desired state to cluster
```

An Application can be:

```text
OutOfSync
```

after refresh without automatically deploying if automated sync is disabled.

---

# 65. GitOps CI Trigger Sequence

```text
Git push
 |
 v
CI
 |
 v
Build/Test/Scan
 |
 v
ECR
 |
 v
GitOps commit
 |
 v
Git webhook
 |
 v
Argo CD refresh
 |
 v
OutOfSync
 |
 v
Automated sync
 |
 v
Kubernetes
```

---

# 66. CI Failure Handling

If:

```text
unit test fails
```

stop.

If:

```text
SonarQube gate fails
```

stop.

If:

```text
Trivy policy fails
```

stop.

If:

```text
ECR push fails
```

stop.

Do not update GitOps.

---

# 67. GitOps Update Failure

If:

```text
ECR push succeeds
```

but:

```text
GitOps update fails
```

the image remains in ECR but is not deployed.

This is safe.

A retry can update GitOps later.

---

# 68. Argo CD Deployment Failure

If GitOps update succeeds but deployment fails:

```text
Git remains the desired state
Argo CD reports health/sync failure
```

Investigate Kubernetes rather than rebuilding the image unnecessarily.

---

# 69. Failed Deployment Workflow

```text
GitOps updated
   |
   v
Argo CD sync
   |
   v
Deployment
   |
   v
CrashLoopBackOff
```

Check:

```bash
argocd app get roboshop-dev
kubectl get pods -n roboshop-dev
kubectl describe pod <pod> -n roboshop-dev
kubectl logs <pod> -n roboshop-dev
```

---

# 70. CI vs Argo CD Logs

CI logs answer:

```text
Was the artifact built correctly?
```

Argo CD logs answer:

```text
Was desired state rendered/applied?
```

Kubernetes logs answer:

```text
Is the workload running correctly?
```

This separation speeds troubleshooting.

---

# 71. Repository Authentication Failure

Argo CD error can indicate:

```text
invalid credentials
repository unavailable
SSH host key issue
TLS problem
permission denied
```

Check:

```bash
argocd repo list
```

and repository configuration.

---

# 72. GitOps Repository Authentication

Use:

```text
SSH deploy key
GitHub App
HTTPS token
```

depending on organization policy.

Prefer short-lived or narrowly scoped credentials where possible.

---

# 73. Private Git Repository

Production Argo CD commonly reads:

```text
private Git repository
```

The repository credential should allow:

```text
read access
```

unless Argo CD itself needs write access, which is normally unnecessary.

---

# 74. Important Security Principle

Argo CD usually needs:

```text
READ Git
```

CI automation may need:

```text
WRITE GitOps
```

Do not give Argo CD write access to Git just because CI has it.

---

# 75. ECR Permissions

CI needs image publishing permissions such as:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:InitiateLayerUpload
ecr:UploadLayerPart
ecr:CompleteLayerUpload
ecr:PutImage
```

Scope repository access as tightly as possible.

The exact policy should reflect the build architecture.

---

# 76. EKS Runtime ECR Access

The Kubernetes nodes or workload identity need permission to pull images from ECR.

Do not give application pods broad:

```text
ecr:* 
```

permissions.

Use the AWS-supported EKS image pull mechanism and least privilege.

---

# 77. CI Should Not Need EKS Admin

A well-designed GitOps pipeline can have:

```text
CI -> ECR
CI -> GitOps Git
Argo CD -> EKS
```

CI does not need:

```text
cluster-admin
```

---

# 78. Argo CD Cluster Access

Argo CD needs credentials to the target cluster.

For centralized Argo CD:

```text
Argo CD management cluster
       |
       +--> EKS DEV
       +--> EKS QA
       +--> EKS PROD
```

Each destination should have controlled permissions.

---

# 79. Production Security Boundary

```text
Developer
   |
   v
Application Git
   |
   v
CI
   |
   +--> ECR
   |
   +--> GitOps PR
           |
           v
        Reviewer
           |
           v
        Git main
           |
           v
        Argo CD
           |
           v
        EKS PROD
```

This is much safer than:

```text
Developer -> kubectl -> PROD
```

---

# 80. Jenkins Pipeline Example

Production-style skeleton:

```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REGISTRY = '123456789012.dkr.ecr.ap-south-1.amazonaws.com'
        IMAGE_REPO = 'roboshop/cart'
        IMAGE_TAG = "git-${env.GIT_COMMIT.take(7)}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube') {
            steps {
                sh '''
                  mvn sonar:sonar \
                    -Dsonar.host.url="$SONAR_HOST_URL" \
                    -Dsonar.token="$SONAR_TOKEN"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                  docker build \
                    -t "$ECR_REGISTRY/$IMAGE_REPO:$IMAGE_TAG" .
                '''
            }
        }

        stage('Image Scan') {
            steps {
                sh '''
                  trivy image \
                    "$ECR_REGISTRY/$IMAGE_REPO:$IMAGE_TAG"
                '''
            }
        }

        stage('Push ECR') {
            steps {
                sh '''
                  aws ecr get-login-password \
                    --region "$AWS_REGION" |
                  docker login \
                    --username AWS \
                    --password-stdin "$ECR_REGISTRY"

                  docker push \
                    "$ECR_REGISTRY/$IMAGE_REPO:$IMAGE_TAG"
                '''
            }
        }

        stage('Update GitOps') {
            steps {
                sh '''
                  ./scripts/update-gitops.sh \
                    "$IMAGE_TAG"
                '''
            }
        }
    }
}
```

Use Jenkins credentials and federated AWS authentication rather than embedding secrets.

---

# 81. Jenkins GitOps Script

A production script can:

```text
clone GitOps repo
checkout branch
update YAML
validate YAML
render Helm
run policy checks
commit
push branch
create PR
```

---

# 82. GitOps Update Script Flow

```bash
git clone "$GITOPS_REPO"
cd roboshop-gitops

git checkout -b "promote-cart-${IMAGE_TAG}"

yq -i \
  ".image.tag = \"${IMAGE_TAG}\"" \
  environments/dev/values/cart.yaml

helm template cart charts/roboshop \
  -f environments/dev/values/cart.yaml >/tmp/cart.yaml

git diff --check
git add .
git commit -m "chore(cart): promote ${IMAGE_TAG} to dev"
git push origin "promote-cart-${IMAGE_TAG}"
```

The PR creation can then be performed through the Git provider's approved automation.

---

# 83. Production PR Automation

A robust pipeline:

```text
CI
 |
 v
Create branch
 |
 v
Update values
 |
 v
Validate
 |
 v
Push branch
 |
 v
Create PR
 |
 v
Required checks
 |
 v
Review
 |
 v
Merge
 |
 v
Argo CD
```

---

# 84. GitHub Actions PR Promotion

Conceptual workflow:

```yaml
name: Promote to Production

on:
  workflow_dispatch:
    inputs:
      image_tag:
        description: "Immutable image tag"
        required: true

permissions:
  contents: write
  pull-requests: write

jobs:
  promote:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Update production value
        run: |
          yq -i \
            '.image.tag = "${{ inputs.image_tag }}"' \
            environments/prod/values/cart.yaml

      - name: Validate
        run: |
          git diff --check
          helm template cart charts/roboshop \
            -f environments/prod/values/cart.yaml

      - name: Commit
        run: |
          git config user.name "release-bot"
          git config user.email "release-bot@example.com"
          git checkout -b \
            "promote-cart-${{ inputs.image_tag }}"
          git add .
          git commit -m \
            "chore(cart): promote ${{ inputs.image_tag }} to prod"
          git push origin HEAD
```

Production workflows should add appropriate review and environment protection.

---

# 85. GitHub Environments

GitHub environments can add controls such as:

```text
production
```

with:

```text
required reviewers
deployment protection rules
environment-scoped secrets
```

This can complement GitOps PR controls.

---

# 86. Jenkins Approval

Jenkins can also implement an approval gate:

```groovy
input message: 'Approve production promotion?'
```

However, the preferred location of approval should align with the organization's GitOps governance model.

A Git PR approval is often more auditable because the approval is attached directly to the desired-state change.

---

# 87. Avoid Double Approval Complexity

Do not unnecessarily create:

```text
Jenkins approval
+
GitHub approval
+
Argo CD manual approval
```

unless policy requires it.

Too many gates can slow releases without improving control.

---

# 88. GitOps and Change Management

A production change can be associated with:

```text
ticket
PR
commit
image
Argo CD revision
```

Example:

```text
CHG-1024
  |
  v
PR #842
  |
  v
Git commit a1b2c3d
  |
  v
image digest sha256:...
  |
  v
Argo CD revision
```

---

# 89. Notifications

Argo CD notifications can alert:

```text
sync succeeded
sync failed
application degraded
application out of sync
```

Send to approved destinations such as:

```text
Slack
email
webhook
incident system
```

The exact integration depends on the environment.

---

# 90. CI Notifications

CI should notify on:

```text
build failure
security failure
image push failure
promotion PR creation
promotion PR failure
```

Argo CD should notify on:

```text
deployment failure
health degradation
sync failure
```

---

# 91. Ownership During Failure

### Build failure

```text
Application/CI team
```

### Security failure

```text
Security/Application team
```

### GitOps rendering failure

```text
Platform/GitOps team
```

### Kubernetes runtime failure

```text
Application + Platform
```

Clear ownership improves incident response.

---

# 92. Argo CD Application Health

After sync:

```bash
argocd app get roboshop-dev
```

Review:

```text
Sync Status
Health Status
Revision
Resources
```

A successful sync does not necessarily mean the application is healthy.

---

# 93. Sync Success vs Health Success

Important distinction:

```text
Sync = desired resources were applied/reconciled
```

while:

```text
Healthy = resources report acceptable health
```

Example:

```text
Sync: Synced
Health: Degraded
```

can happen if:

```text
Deployment exists
but pods are failing
```

---

# 94. CI Should Validate Deployment Health

After DEV deployment:

```text
Wait for Argo CD
 |
 v
Check sync
 |
 v
Check health
 |
 v
Run smoke tests
```

Only then promote.

---

# 95. Argo CD CLI Validation

Example:

```bash
argocd app wait roboshop-dev \
  --sync \
  --health
```

The exact flags available depend on the installed CLI version.

---

# 96. Smoke Testing

After deployment:

```bash
curl -f https://dev.example.com/health
```

For RoboShop, validate representative endpoints rather than relying only on Kubernetes health.

---

# 97. Smoke Test Failure

If:

```text
Argo CD Healthy
```

but:

```text
application smoke test fails
```

the deployment can still be considered unsuccessful.

Kubernetes health and business health are different layers.

---

# 98. Promotion Gate

A strong gate is:

```text
Argo CD Healthy
+
smoke tests passed
+
security checks passed
```

then:

```text
promote QA
```

---

# 99. QA Gate

Example:

```text
DEV deployment
    |
    v
Smoke tests
    |
    v
Integration tests
    |
    v
Performance/security checks
    |
    v
QA promotion
```

---

# 100. Production Gate

Example:

```text
QA validated
   |
   v
Release PR
   |
   v
Review
   |
   v
Merge
   |
   v
Argo CD PROD
   |
   v
Health checks
   |
   v
Monitoring
```

---

# 101. GitOps and Helm Release

Argo CD can render Helm charts.

GitOps defines:

```text
chart version
values
target cluster
namespace
sync policy
```

Argo CD performs the deployment.

---

# 102. Helm Chart Versioning

Example:

```yaml
apiVersion: v2
name: roboshop
version: 1.5.0
appVersion: 2.4.1
```

Do not confuse:

```text
chart version
```

with:

```text
application version
```

---

# 103. Chart Promotion

A release may require:

```text
chart change
+
image change
```

or only:

```text
image change
```

Keep these changes explicit.

---

# 104. ApplicationSet + CI

CI should generally update:

```text
input values
```

not generate arbitrary Application manifests.

For example:

```text
CI:
image tag -> values file

ApplicationSet:
generates Application

Argo CD:
deploys Application
```

This separates concerns.

---

# 105. Example ApplicationSet Architecture

```text
GitOps
 |
 +--> ApplicationSet
 |       |
 |       +--> cart-dev
 |       +--> cart-qa
 |       +--> cart-prod
 |
 +--> values/dev
 +--> values/qa
 +--> values/prod
```

---

# 106. CI Updates Values

```text
cart-dev:
image.tag = git-a1b2c3d
```

ApplicationSet remains unchanged.

This is much cleaner than CI modifying:

```text
ApplicationSet
Application
Deployment
Service
```

for every release.

---

# 107. GitOps Repository Structure for CI

```text
roboshop-gitops/
├── applicationsets/
│   └── roboshop.yaml
│
├── projects/
│   ├── dev.yaml
│   ├── qa.yaml
│   └── prod.yaml
│
├── charts/
│   └── roboshop/
│
└── environments/
    ├── dev/
    │   └── values/
    ├── qa/
    │   └── values/
    └── prod/
        └── values/
```

---

# 108. CI Change Surface

CI should normally touch only:

```text
environments/<env>/values/<service>.yaml
```

for an image-only release.

This reduces accidental changes.

---

# 109. Production GitOps PR Example

```diff
 image:
   repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
-  tag: git-998877
+  tag: git-a1b2c3d
```

A tiny diff is easier to review.

---

# 110. Image Digest Promotion

Example:

```diff
 image:
   repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
-  digest: sha256:111...
+  digest: sha256:222...
```

This is highly deterministic.

---

# 111. Image Promotion vs Rebuild

Bad:

```text
DEV build -> image A
QA rebuild -> image B
PROD rebuild -> image C
```

Preferred:

```text
one build
   |
   v
image A
   |
   +--> DEV
   +--> QA
   +--> PROD
```

---

# 112. Environment-Specific Docker Builds

Avoid changing:

```text
Dockerfile
```

based on:

```text
DEV
QA
PROD
```

unless there is a strong architectural reason.

Environment differences belong primarily in runtime configuration.

---

# 113. Docker Image Configuration

The same image can consume:

```text
environment variables
ConfigMaps
Secrets
AWS services
```

to behave appropriately per environment.

---

# 114. GitOps and ConfigMaps

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cart-config
data:
  ENVIRONMENT: prod
  LOG_LEVEL: info
```

This can be managed through Git.

Do not store passwords here.

---

# 115. GitOps and Secrets

Use:

```text
AWS Secrets Manager
External Secrets Operator
```

or another approved secret system.

Argo CD should deploy secret references rather than plaintext credentials.

---

# 116. Production Secret Flow

```text
GitOps
 |
 v
ExternalSecret
 |
 v
External Secrets Operator
 |
 v
AWS Secrets Manager
 |
 v
Kubernetes Secret
 |
 v
Pod
```

---

# 117. CI Does Not Need Production Secrets

A good architecture avoids:

```text
CI -> read PROD database password
```

CI usually only needs:

```text
ECR publish permission
GitOps repository permission
test environment access
```

depending on pipeline requirements.

---

# 118. GitOps and AWS ALB

For RoboShop:

```text
Application
   |
   v
Kubernetes Service
   |
   v
Ingress
   |
   v
AWS Load Balancer Controller
   |
   v
ALB
```

GitOps manages:

```text
Ingress manifest
annotations
host/path
TLS references
```

Terraform may manage:

```text
VPC
subnets
IAM
EKS
```

Ownership must be clearly defined.

---

# 119. CI Does Not Provision EKS

Infrastructure should normally be managed separately:

```text
Terraform
   |
   v
AWS/EKS infrastructure
```

while:

```text
Argo CD
   |
   v
Kubernetes application/platform resources
```

---

# 120. Terraform and Argo CD Boundary

Terraform:

```text
VPC
EKS
IAM
security groups
subnets
ECR
AWS infrastructure
```

Argo CD:

```text
Namespace
Deployment
Service
Ingress
ConfigMap
HPA
PDB
Application
```

Do not have both continuously managing the same Kubernetes resource unless intentionally designed.

---

# 121. CI + Terraform + Argo CD

A mature architecture can be:

```text
Infrastructure Git
      |
      v
Terraform CI/CD
      |
      v
AWS/EKS

Application Git
      |
      v
CI
      |
      v
ECR

GitOps Git
      |
      v
Argo CD
      |
      v
Kubernetes
```

---

# 122. Infrastructure Pipeline

```text
Terraform Git
 |
 v
terraform fmt
 |
 v
terraform validate
 |
 v
terraform plan
 |
 v
approval
 |
 v
terraform apply
```

Argo CD should not replace Terraform for AWS infrastructure provisioning.

---

# 123. Application Pipeline

```text
Application Git
 |
 v
Build
 |
 v
Test
 |
 v
Security
 |
 v
Image
 |
 v
ECR
 |
 v
GitOps
 |
 v
Argo CD
```

---

# 124. Why Two Pipelines?

Because:

```text
Infrastructure lifecycle
```

and:

```text
Application lifecycle
```

have different change frequencies and risks.

Application releases may occur:

```text
many times per day
```

while infrastructure changes may be:

```text
less frequent
```

---

# 125. Production Pipeline Separation

Recommended:

```text
Pipeline A:
Terraform infrastructure

Pipeline B:
Application CI

GitOps:
Application CD
```

---

# 126. GitOps and Platform Applications

Argo CD can deploy:

```text
AWS Load Balancer Controller
External Secrets Operator
Prometheus
Grafana
ELK components
```

if those are intentionally managed through GitOps.

---

# 127. Platform/Application Ordering

Example:

```text
Infrastructure
      |
      v
EKS
      |
      v
Platform components
      |
      v
Ingress/Secrets/Observability
      |
      v
Application
```

Use appropriate dependency management such as sync waves where needed.

---

# 128. Platform Bootstrapping

A bootstrap flow can be:

```text
Terraform
 |
 v
EKS
 |
 v
Argo CD
 |
 v
platform-root
 |
 +--> ingress
 +--> external-secrets
 +--> monitoring
 +--> applications
```

This leads into the App of Apps pattern covered elsewhere.

---

# 129. CI and Bootstrap

CI should not repeatedly bootstrap Argo CD for every application deployment.

Bootstrap is:

```text
platform lifecycle
```

Application release is:

```text
application lifecycle
```

---

# 130. GitOps Event Flow

```text
Developer pushes source
        |
        v
GitHub webhook
        |
        v
Jenkins/GitHub Actions
        |
        v
Build/Test/Security
        |
        v
ECR
        |
        v
GitOps update
        |
        v
Git provider
        |
        v
Argo CD webhook/refresh
        |
        v
Desired state calculation
        |
        v
Kubernetes API
        |
        v
Pods
```

---

# 131. Failure Boundary: Source Git

If source Git is unavailable:

```text
new CI runs may fail
```

Existing workloads generally continue.

---

# 132. Failure Boundary: CI

If CI is down:

```text
new artifacts cannot be built
```

Existing GitOps deployments continue to operate.

---

# 133. Failure Boundary: GitOps Git

If GitOps Git is unavailable:

```text
new desired-state changes cannot be committed
```

Existing workloads generally continue.

---

# 134. Failure Boundary: Argo CD

If Argo CD is unavailable:

```text
existing Kubernetes workloads generally continue
```

but:

```text
new reconciliation
deployment
drift correction
```

can be delayed.

---

# 135. Failure Boundary: Kubernetes

If EKS is unavailable:

```text
application unavailable
```

depending on architecture.

GitOps can help rebuild/reconcile after infrastructure recovery.

---

# 136. Failure Boundary: ECR

If ECR is unavailable:

```text
new image pulls
```

can fail.

Existing running containers may continue until they need replacement.

This is why:

```text
image retention
registry resilience
cross-region/cross-account strategy
```

can matter.

---

# 137. Failure Boundary: AWS Secrets Manager

If workloads need secrets and the secret provider is unavailable:

```text
new secret retrieval
```

may fail.

Existing Kubernetes Secrets may continue depending on the integration design.

---

# 138. CI Retry Strategy

Retry transient failures:

```text
network timeout
ECR transient error
Git API temporary failure
```

Do not blindly retry:

```text
security policy failure
unit test failure
invalid YAML
```

---

# 139. Argo CD Retry Strategy

Argo CD Application retry can help transient deployment errors.

Use bounded retries and investigate persistent failures.

Do not use retries to hide:

```text
invalid manifests
RBAC problems
bad images
broken probes
```

---

# 140. Promotion Failure

Suppose QA validation fails.

Do not promote to PROD.

Instead:

```text
QA remains on previous version
```

while the new image can be investigated.

---

# 141. GitOps Rollback After Failed Promotion

If production promotion was merged accidentally:

```text
revert Git commit
```

Then:

```text
Argo CD reconciles previous desired state
```

---

# 142. Emergency Rollback

If application impact is severe:

```bash
argocd app history roboshop-prod
argocd app rollback roboshop-prod <revision>
```

Then immediately ensure Git desired state reflects the recovered version.

---

# 143. Rollback Does Not Fix Bad Git

If Git still declares:

```text
bad version
```

and you manually rollback the cluster:

```text
Argo CD may redeploy the bad version.
```

Therefore:

```text
emergency rollback
+
Git correction
```

is essential.

---

# 144. GitOps Drift After Manual kubectl

Example:

```bash
kubectl set image deployment/cart \
  cart=...:hotfix \
  -n roboshop
```

Git still says:

```text
old image
```

Argo CD sees:

```text
drift
```

and may restore Git state.

---

# 145. Emergency Hotfix Strategy

A better approach:

```text
Create emergency GitOps PR
```

if time allows.

If not:

```text
manual emergency change
        |
        v
restore service
        |
        v
immediately record in Git
```

---

# 146. Production Deployment Strategy

For low-risk services:

```text
automatic sync
```

may be acceptable.

For high-risk production services:

```text
PR
+
approval
+
manual sync
```

may be appropriate.

---

# 147. Service-Level Deployment Policy

Not every RoboShop service has identical risk.

For example:

```text
frontend
cart
catalogue
```

may have different release requirements than:

```text
payment
orders
```

Use service-level risk classification where useful.

---

# 148. Payment Service Example

A payment release may require:

```text
Security checks
QA validation
manual production approval
controlled rollout
enhanced monitoring
```

This is a business-risk decision, not only a Kubernetes decision.

---

# 149. CI Artifact Metadata

Generate metadata:

```json
{
  "service": "cart",
  "sourceCommit": "a1b2c3d",
  "imageTag": "git-a1b2c3d",
  "imageDigest": "sha256:...",
  "build": "842"
}
```

Store it in an approved artifact system if required.

---

# 150. SBOM

For supply-chain security, CI can generate an SBOM.

Example tooling may include:

```text
Trivy
Syft
```

depending on organizational standards.

The SBOM can be associated with:

```text
image digest
```

---

# 151. Artifact Provenance

Strong release metadata can connect:

```text
source
 |
 v
build
 |
 v
artifact
 |
 v
GitOps
 |
 v
deployment
```

This is increasingly important for software supply-chain security.

---

# 152. Signed Images

A mature platform can use image signing.

Conceptually:

```text
CI
 |
 v
Build image
 |
 v
Sign image
 |
 v
ECR
 |
 v
Admission policy
 |
 v
EKS
```

Tools such as Cosign may be used where approved.

Argo CD itself is not a replacement for admission policy.

---

# 153. Git Commit Signing

Organizations may also sign Git commits/tags.

This can strengthen:

```text
source authenticity
release traceability
```

---

# 154. GitOps Admission Controls

Kubernetes admission controls can enforce:

```text
signed images
approved registries
required securityContext
resource limits
disallowed capabilities
```

Examples include:

```text
Kyverno
OPA Gatekeeper
```

if used by the organization.

---

# 155. CI vs Admission Policy

CI:

```text
detect before deployment
```

Admission:

```text
enforce at cluster boundary
```

Using both provides defense in depth.

---

# 156. GitOps Deployment Policy

A production deployment may pass:

```text
CI security
```

but fail:

```text
cluster admission
```

This is intentional.

The cluster should not trust CI blindly.

---

# 157. Argo CD and Admission Failure

If Kubernetes rejects a resource:

```text
Argo CD sync fails
```

Investigate:

```text
kubectl events
admission controller logs
resource manifest
policy violation
```

---

# 158. Troubleshooting Checklist

When a GitOps deployment fails:

```text
1. Did CI pass?
2. Was image pushed?
3. Does image exist?
4. Did GitOps update?
5. Did Argo CD refresh?
6. Is Application OutOfSync?
7. Did sync fail?
8. Is Application Degraded?
9. Did Kubernetes reject resource?
10. Are pods healthy?
```

---

# 159. CI Failure Commands

Inspect Jenkins/GitHub Actions logs.

Verify:

```bash
git rev-parse HEAD
docker images
aws ecr describe-images ...
```

---

# 160. GitOps Repository Failure Commands

```bash
git status
git log --oneline -10
git diff
```

Validate:

```bash
helm template ...
kustomize build ...
```

---

# 161. Argo CD Failure Commands

```bash
argocd app get roboshop-dev
argocd app diff roboshop-dev
argocd app history roboshop-dev
argocd repo list
```

---

# 162. Kubernetes Failure Commands

```bash
kubectl get applications -n argocd
kubectl get pods -n roboshop-dev
kubectl get events -n roboshop-dev --sort-by=.lastTimestamp
kubectl describe deployment cart -n roboshop-dev
kubectl describe pod <pod> -n roboshop-dev
kubectl logs <pod> -n roboshop-dev
```

---

# 163. Failure: GitOps Updated but Argo CD Did Not Notice

Check:

```text
repository connectivity
revision
webhook
refresh interval
Argo CD repo-server
```

Run:

```bash
argocd app refresh roboshop-dev
```

Then:

```bash
argocd app get roboshop-dev
```

---

# 164. Failure: Argo CD Sees Change but Sync Is Disabled

Check:

```text
syncPolicy
automated
```

If manual:

```bash
argocd app sync roboshop-dev
```

---

# 165. Failure: Sync Succeeds but Pod Fails

Check:

```bash
kubectl get pods -n roboshop-dev
kubectl describe pod <pod> -n roboshop-dev
kubectl logs <pod> -n roboshop-dev
```

Likely causes:

```text
bad configuration
missing secret
wrong image
probe failure
resource issue
dependency failure
```

---

# 166. Failure: ImagePullBackOff

Check:

```bash
kubectl describe pod <pod> -n roboshop-dev
```

Look for:

```text
image name
ECR permissions
node IAM
network
registry availability
```

Verify image:

```bash
aws ecr describe-images \
  --repository-name roboshop/cart \
  --image-ids imageTag=git-a1b2c3d
```

---

# 167. Failure: Helm Rendering Error

Test locally:

```bash
helm lint charts/roboshop
```

Then:

```bash
helm template cart charts/roboshop \
  -f environments/prod/values/cart.yaml
```

Check:

```text
missing values
invalid template
wrong indentation
wrong type
```

---

# 168. Failure: GitOps PR Merged but Wrong Image

Check:

```text
values file
Application source path
ApplicationSet template
Helm values precedence
```

The image in the GitOps file may not be the value actually consumed by the chart.

---

# 169. Failure: Wrong Values File

Helm may use:

```yaml
valueFiles:
  - values/prod/cart.yaml
```

Verify that:

```text
path exists
file is correct
Application points to correct chart
```

---

# 170. Failure: CI Updates Wrong Environment

Check:

```text
ENVIRONMENT variable
GitOps path
branch
workflow input
promotion logic
```

Use explicit environment names.

Avoid deriving:

```text
prod
```

from unreliable branch assumptions.

---

# 171. Safer Promotion Input

Instead of:

```text
if branch == main -> prod
```

prefer explicit workflow controls:

```text
environment=dev
environment=qa
environment=prod
```

combined with repository protections.

---

# 172. GitHub Actions Production Environment

Example:

```yaml
jobs:
  promote:
    environment:
      name: production
```

This can connect the workflow to protected environment controls.

---

# 173. Jenkins Production Parameter

Example:

```groovy
parameters {
    choice(
        name: 'ENVIRONMENT',
        choices: ['dev', 'qa', 'prod'],
        description: 'Deployment environment'
    )
}
```

Production should still require policy-based approval.

---

# 174. Avoid Unsafe Dynamic Paths

Dangerous:

```bash
rm -rf environments/$ENVIRONMENT/*
```

if input is not validated.

Prefer an allowlist:

```text
dev
qa
prod
```

---

# 175. Pipeline Input Validation

Validate:

```bash
case "$ENVIRONMENT" in
  dev|qa|prod)
    ;;
  *)
    echo "Invalid environment"
    exit 1
    ;;
esac
```

---

# 176. GitOps Automation Identity

Create dedicated identities:

```text
ci-dev-bot
release-bot
argocd-readonly
```

rather than sharing:

```text
admin
```

credentials.

---

# 177. Principle of Least Privilege

CI:

```text
ECR push
GitOps PR
```

Argo CD:

```text
Git read
Kubernetes deployment permissions
```

Developer:

```text
source repository
DEV access
```

Production approver:

```text
review/approve
```

---

# 178. No Shared Admin Credentials

Avoid:

```text
Jenkins -> cluster-admin
Argo CD -> Git write
Developer -> production admin
```

These violate separation of duties.

---

# 179. Audit Logs

Capture:

```text
CI build logs
Git PR history
Argo CD application history
Kubernetes audit logs where enabled
AWS CloudTrail for AWS API activity
```

The user's application monitoring stack can focus on:

```text
Prometheus
Grafana
ELK
```

while platform audit systems serve a different purpose.

---

# 180. GitOps Observability

Monitor:

```text
Argo CD applications
sync failures
health degradation
reconciliation latency
repo-server errors
application-controller errors
```

Export metrics to the organization's monitoring stack where supported.

---

# 181. Prometheus Integration

Argo CD exposes metrics that can be scraped by Prometheus depending on the deployed configuration.

Useful categories include:

```text
application health
sync status
controller activity
repository operations
```

Build alerts around service impact, not every transient event.

---

# 182. Grafana Dashboard

A useful GitOps dashboard can show:

```text
Applications
Synced
OutOfSync
Healthy
Degraded
Sync failures
Repository errors
Controller activity
```

---

# 183. ELK Integration

Logs can be centralized for:

```text
Argo CD
Kubernetes
Ingress
application
CI agents
```

Then correlate:

```text
deployment
+
application errors
```

---

# 184. Deployment Correlation

Example:

```text
10:05 GitOps commit
10:06 Argo CD sync
10:07 new pods
10:08 HTTP 5xx spike
10:09 rollback
```

Centralized logs and metrics make this timeline visible.

---

# 185. Deployment Marker

A useful practice is to expose deployment metadata:

```text
version
commit
image digest
```

in:

```text
application metrics
health endpoint
logs
```

This helps correlate releases with incidents.

---

# 186. Example Application Version

ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cart-version
data:
  VERSION: "v2.4.1"
  GIT_COMMIT: "a1b2c3d"
```

Do not store secrets here.

---

# 187. Production Change Timeline

```text
Git commit
   |
   v
CI build
   |
   v
Security
   |
   v
ECR
   |
   v
GitOps PR
   |
   v
Approval
   |
   v
Argo CD
   |
   v
EKS
   |
   v
Prometheus/Grafana/ELK
```

This is the complete traceability chain.

---

# 188. GitOps with Jenkins: Recommended Model

```text
Jenkins
  |
  +--> Build
  +--> Test
  +--> SonarQube
  +--> Trivy
  +--> Veracode
  +--> Docker build
  +--> ECR
  |
  +--> GitOps PR
             |
             v
          Argo CD
             |
             v
            EKS
```

---

# 189. GitOps with GitHub Actions: Recommended Model

```text
GitHub
 |
 v
Actions
 |
 +--> Build
 +--> Test
 +--> Security
 +--> ECR
 |
 v
GitOps PR
 |
 v
Argo CD
 |
 v
EKS
```

---

# 190. Jenkins vs GitHub Actions

Both can perform CI responsibilities.

The GitOps architecture does not fundamentally change:

```text
CI -> artifact -> GitOps -> Argo CD -> Kubernetes
```

Only the CI engine changes.

---

# 191. CI Tool Choice

Jenkins:

```text
high customization
large plugin ecosystem
existing enterprise installations
```

GitHub Actions:

```text
native GitHub integration
workflow-as-code
OIDC integration
easy repository-local automation
```

Use the platform already approved by the organization.

---

# 192. What CI Should Never Own

CI should not be the source of truth for:

```text
desired replicas
Ingress
Service
HPA
deployment strategy
production configuration
```

Those should live in GitOps configuration.

---

# 193. What GitOps Should Not Own

GitOps should not replace:

```text
application compilation
unit testing
source scanning
container building
```

Those are CI responsibilities.

---

# 194. What Argo CD Should Not Do

Argo CD should not be used as:

```text
application build server
```

It deploys artifacts and desired configuration.

---

# 195. GitOps and CI/CD Terminology

Traditional:

```text
CI = build/test
CD = deploy
```

GitOps:

```text
CI = artifact lifecycle
GitOps repository = desired deployment state
Argo CD = continuous delivery/reconciliation
Kubernetes = runtime
```

---

# 196. GitOps Pull-Based CD

Argo CD pulls:

```text
desired state from Git
```

rather than CI pushing commands into Kubernetes.

This reduces inbound deployment access from CI.

---

# 197. Push vs Pull

### Push

```text
CI
 |
 | credentials
 v
Kubernetes
```

### Pull

```text
Git
 |
 v
Argo CD inside/connected to cluster
 |
 v
Kubernetes
```

The pull model is a key GitOps security advantage.

---

# 198. Central Argo CD

In multi-cluster environments:

```text
Git
 |
 v
Central Argo CD
 |
 +--> DEV EKS
 +--> QA EKS
 +--> PROD EKS
```

CI updates GitOps.

Argo CD controls deployment.

---

# 199. CI and Multi-Cluster

CI does not need:

```text
DEV cluster credentials
QA cluster credentials
PROD cluster credentials
```

if the architecture is fully GitOps-based.

It only needs:

```text
artifact registry access
GitOps repository access
```

---

# 200. Blast Radius Reduction

If CI credentials are compromised:

Traditional:

```text
CI credential
   |
   +--> production cluster
```

Potentially catastrophic.

GitOps:

```text
CI credential
   |
   +--> GitOps repository
```

An attacker may still cause deployment through malicious Git changes, but cluster API credentials are not directly exposed to CI.

Repository protection, reviews, signed commits and policy controls remain essential.

---

# 201. GitOps Repository as Security Boundary

Protect:

```text
main
prod paths
ApplicationSets
Projects
cluster definitions
```

because modifying these can alter deployment behavior across many clusters.

---

# 202. ApplicationSet Security

A malicious ApplicationSet change could:

```text
generate applications
target another cluster
deploy privileged resources
```

Therefore ApplicationSet files require strong review.

---

# 203. Project Security

A malicious AppProject change could broaden:

```text
repositories
clusters
namespaces
resource permissions
```

Therefore Projects should be platform-owned and tightly protected.

---

# 204. Production CI Pipeline Checklist

```text
[ ] Checkout
[ ] Build
[ ] Unit tests
[ ] SonarQube
[ ] Dependency/security checks
[ ] Docker build
[ ] Trivy image scan
[ ] Veracode where required
[ ] ECR push
[ ] Verify image
[ ] Generate metadata
[ ] Update GitOps
[ ] Validate manifests
[ ] Create PR
[ ] Approval
[ ] Argo CD deployment
[ ] Health validation
[ ] Smoke tests
[ ] Monitoring
```

---

# 205. Production GitOps Checklist

```text
[ ] Separate application and GitOps repos
[ ] Immutable image
[ ] ECR
[ ] Environment values
[ ] Argo CD ApplicationSet
[ ] Projects
[ ] RBAC
[ ] CODEOWNERS
[ ] Branch protection
[ ] Production approval
[ ] Secrets externalized
[ ] Monitoring
[ ] Notifications
[ ] Rollback
[ ] DR
```

---

# 206. RoboShop End-to-End Example

Suppose:

```text
cart service commit = a1b2c3d
```

CI creates:

```text
cart:git-a1b2c3d
```

Pushes:

```text
ECR
```

Then updates:

```text
environments/dev/values/cart.yaml
```

Argo CD sees:

```text
OutOfSync
```

and syncs.

After DEV validation:

```text
QA values
```

are changed to the same tag/digest.

After QA validation:

```text
PROD values
```

are changed.

Argo CD deploys the exact same artifact.

---

# 207. RoboShop Service Example

For:

```text
cart
```

the pipeline is:

```text
roboshop-cart Git
       |
       v
Jenkins/GitHub Actions
       |
       +--> Maven test
       +--> SonarQube
       +--> Trivy
       +--> Veracode
       |
       v
Docker
       |
       v
ECR
       |
       v
roboshop-gitops
       |
       v
Argo CD
       |
       v
EKS
```

---

# 208. RoboShop Microservices

The same pattern can be used for:

```text
frontend
user
catalogue
cart
shipping
payment
dispatch
```

Each service can have:

```text
independent CI
independent image
independent release
```

while Argo CD provides consistent CD.

---

# 209. Service-Level GitOps Values

Example:

```text
environments/prod/values/
├── frontend.yaml
├── user.yaml
├── catalogue.yaml
├── cart.yaml
├── payment.yaml
├── shipping.yaml
└── dispatch.yaml
```

---

# 210. Independent Release

Cart can move:

```text
v2.4.1
```

while payment remains:

```text
v2.3.7
```

GitOps supports this independently.

---

# 211. Release Coordination

If multiple services must be released together:

```text
one promotion PR
```

can update several values files.

Example:

```text
cart
payment
orders
```

must use compatible versions.

---

# 212. Microservice Dependency Risk

A new:

```text
orders
```

version may depend on a new:

```text
payment
```

API.

CI should test compatibility before production promotion.

GitOps only deploys the approved desired state.

---

# 213. Contract Testing

For microservices, CI can use:

```text
API contract tests
integration tests
```

before promotion.

This prevents GitOps from being blamed for application incompatibility.

---

# 214. Database Migration Integration

CI should package migrations appropriately.

Deployment sequence may be:

```text
migration
 |
 v
application
```

or:

```text
backward-compatible app
 |
 v
migration
 |
 v
new app behavior
```

Use Argo CD sync waves/hooks carefully if database jobs are managed through GitOps.

---

# 215. Migration Safety

Avoid a release where:

```text
new app requires new schema immediately
```

and rollback would restore an old app that cannot understand the new schema.

Use compatible migration design.

---

# 216. CI Artifact Retention

Retain enough ECR images for:

```text
rollback
incident investigation
DR
compliance
```

Do not let aggressive lifecycle policies delete the only rollback image.

---

# 217. GitOps Repository Retention

Git history should retain:

```text
production promotion commits
```

Do not squash away all useful deployment history without a separate audit mechanism.

---

# 218. Release Branches

For long-lived release stabilization, a branch may be useful:

```text
release/2.4
```

However, avoid unnecessary branching complexity.

Use the simplest model that satisfies release governance.

---

# 219. Hotfix Branch

Emergency application fix:

```text
hotfix/cart-payment-timeout
```

After validation:

```text
main
```

and then:

```text
GitOps promotion
```

---

# 220. Hotfix GitOps

Do not bypass GitOps simply because it is a hotfix.

Preferred:

```text
hotfix artifact
   |
   v
security validation
   |
   v
GitOps production PR
   |
   v
approval
   |
   v
Argo CD
```

---

# 221. Emergency Direct kubectl

If absolutely necessary:

```text
restore service immediately
```

then:

```text
document
commit equivalent Git state
verify Argo CD
```

The exception should not become the normal deployment process.

---

# 222. CI/CD Metrics

Useful metrics include:

```text
Deployment frequency
Lead time for changes
Change failure rate
Mean time to recovery
```

These are useful DevOps delivery metrics.

---

# 223. GitOps-Specific Metrics

Track:

```text
Sync success rate
Sync duration
OutOfSync applications
Degraded applications
Deployment failures
Rollback count
Reconciliation latency
```

---

# 224. DORA Metrics + GitOps

GitOps can improve:

```text
Deployment frequency
Lead time
Change traceability
Recovery
```

if the process is well-designed.

GitOps alone does not automatically improve engineering performance.

---

# 225. Pipeline Duration Optimization

Avoid unnecessary:

```text
full rebuild
full scans
duplicate tests
```

but do not remove security controls solely for speed.

Use:

```text
caching
parallel jobs
artifact reuse
incremental tests
```

where appropriate.

---

# 226. Parallel CI Stages

After checkout:

```text
Build
Test
SAST
SCA
```

may be parallelized where dependencies permit.

Then:

```text
Docker build
```

after required checks.

---

# 227. Security Gate Ordering

Example:

```text
Checkout
 |
 +--> Unit tests
 +--> SonarQube
 +--> Dependency scan
 |
 v
Docker build
 |
 v
Trivy
 |
 v
ECR
 |
 v
GitOps
```

Exact ordering depends on the organization's security strategy.

---

# 228. Build Cache Security

Do not blindly trust cached artifacts.

Use:

```text
trusted runners
controlled registries
pinned dependencies
verified base images
```

where required.

---

# 229. Base Image Management

Use approved base images.

Example:

```text
Amazon Linux
Eclipse Temurin
Node
Python
```

depending on application requirements.

Track:

```text
base image version
vulnerabilities
updates
```

---

# 230. Base Image Pipeline

A platform team can maintain:

```text
approved base image
      |
      v
scan
      |
      v
publish
      |
      v
application builds
```

This is part of supply-chain governance.

---

# 231. GitOps and Dependency Updates

Application dependency updates should follow:

```text
source PR
CI
security
artifact
GitOps promotion
```

Do not directly modify production containers.

---

# 232. GitOps and Configuration Review

Review:

```text
replicas
resources
securityContext
image
Ingress
environment variables
HPA
PDB
```

not just:

```text
image tag
```

---

# 233. Configuration Review Example

If a PR changes:

```yaml
replicas: 3
```

to:

```yaml
replicas: 30
```

the reviewer should ask:

```text
Why?
```

GitOps makes such changes visible.

---

# 234. Resource Abuse Prevention

Use:

```text
ResourceQuota
LimitRange
Policy
RBAC
```

to prevent accidental oversized workloads.

---

# 235. GitOps Policy as Code

Policy engines can enforce:

```text
required resources
non-root containers
approved registries
no privileged containers
required labels
```

This can run:

```text
CI
```

and/or:

```text
Kubernetes admission
```

---

# 236. Argo CD Diff + Policy

A production pipeline can:

```text
render
 |
 v
policy check
 |
 v
Git PR
 |
 v
Argo CD diff
 |
 v
sync
```

Defense in depth.

---

# 237. Application Deployment Dependencies

RoboShop may have dependencies such as:

```text
database
cache
message broker
```

Do not rely on Kubernetes startup ordering alone.

Applications should handle dependency readiness correctly.

---

# 238. Argo CD Sync Waves

For GitOps-managed resources:

```text
Namespace
   |
wave 0

Secret/ExternalSecret
   |
wave 1

ConfigMap
   |
wave 1

Deployment
   |
wave 2

Ingress
   |
wave 3
```

Use waves only when dependency ordering is genuinely needed.

---

# 239. Hooks and CI

Argo CD hooks can run:

```text
PreSync
Sync
PostSync
SyncFail
```

A migration job may be a candidate for:

```text
PreSync
```

but database migrations require careful idempotency and rollback design.

---

# 240. CI vs Argo CD Hooks

CI can run:

```text
build-time tests
```

Argo CD hooks can run:

```text
cluster deployment-time tasks
```

Do not duplicate expensive work unnecessarily.

---

# 241. Production Hook Safety

Hooks should be:

```text
idempotent
observable
timeout-controlled
owned
reviewed
```

A failed hook should produce a clear deployment failure.

---

# 242. Sync Wave Failure

If:

```text
wave 1
```

fails, later waves may not proceed as expected.

Check:

```text
hook status
resource health
events
Argo CD operation details
```

---

# 243. Webhook Failure

If Git webhook stops working:

```text
Argo CD may still refresh periodically
```

but detection may be delayed.

Monitor webhook health externally where possible.

---

# 244. Repository Polling Failure

If repo-server cannot access Git:

```text
new Git state cannot be rendered
```

Check:

```text
DNS
network
credentials
SSH
TLS
proxy
Git provider
```

---

# 245. GitOps Repository Merge Conflict

If multiple automation jobs update the same file:

```text
promotion collision
```

can occur.

Solutions:

```text
service-specific files
queue promotions
rebase branch
retry carefully
```

---

# 246. Avoid Concurrent Writes

If two pipelines update:

```text
environments/dev/values.yaml
```

simultaneously:

```text
last writer wins
```

can lose a release.

Prefer:

```text
one file per service
```

where practical.

---

# 247. Promotion Queue

For high-volume repositories:

```text
CI events
 |
 v
promotion queue
 |
 v
serialized GitOps updates
```

can prevent conflicting writes.

---

# 248. Monorepo Scaling

One GitOps repository may contain:

```text
100+ services
```

Use:

```text
clear ownership
directory boundaries
CODEOWNERS
ApplicationSets
small files
```

---

# 249. Repository Performance

Large repositories can increase:

```text
clone time
render time
review complexity
```

Argo CD architecture should be sized appropriately.

---

# 250. GitOps Repository Design Principle

A good repository should answer quickly:

```text
Where is DEV config?
Where is QA config?
Where is PROD config?
Where is this service's image?
Where is its ApplicationSet?
Who owns it?
```

---

# 251. Production Deployment Runbook

### Before release

```text
CI passed
Security passed
QA passed
Image exists
Promotion PR reviewed
```

### During release

```text
Merge
Argo CD refresh
Sync
Health check
Smoke test
```

### After release

```text
Prometheus
Grafana
ELK
business metrics
```

Monitor for regressions.

---

# 252. Production Rollback Runbook

```text
1. Confirm impact.
2. Identify deployed revision.
3. Identify previous known-good revision.
4. Revert GitOps PR or use emergency rollback.
5. Verify Argo CD.
6. Verify pods.
7. Verify application endpoint.
8. Monitor metrics/logs.
9. Record incident.
10. Fix Git desired state if emergency rollback was used.
```

---

# 253. CI Incident Runbook

If CI is failing:

```text
Check source
Check dependency
Check runner
Check security tool
Check registry
Check credentials
```

Do not manually deploy around CI without recording the exception.

---

# 254. GitOps Incident Runbook

If Argo CD deployment fails:

```text
Check Application
Check diff
Check operation
Check events
Check resource health
Check repo-server
Check controller
```

---

# 255. Production Change Runbook

```text
PR created
 |
 v
Automated validation
 |
 v
Security checks
 |
 v
Reviewer approval
 |
 v
Merge
 |
 v
Argo CD
 |
 v
Sync
 |
 v
Health
 |
 v
Smoke tests
 |
 v
Observe
```

---

# 256. What Happens If CI Is Compromised?

Potential attacker action:

```text
build malicious image
```

Controls:

```text
code review
branch protection
security scans
image signing
ECR controls
admission policies
GitOps review
```

---

# 257. What Happens If GitOps Bot Is Compromised?

Potential attacker action:

```text
modify GitOps repository
```

Controls:

```text
scoped token
PR-only production access
CODEOWNERS
branch protection
signed commits
audit
alerts
```

---

# 258. What Happens If Argo CD Is Compromised?

Potential impact:

```text
cluster resources
```

Controls:

```text
RBAC
least privilege
network controls
separate Projects
management cluster security
SSO
audit
HA
backup
```

---

# 259. Defense in Depth

The complete security chain:

```text
Developer review
      |
      v
CI security
      |
      v
ECR
      |
      v
GitOps review
      |
      v
Argo CD RBAC
      |
      v
Kubernetes RBAC
      |
      v
Admission policy
      |
      v
Runtime monitoring
```

---

# 260. Production Architecture

```text
                     Developer
                         |
                         v
                  Application Git
                         |
                         v
              Jenkins / GitHub Actions
                         |
       +-----------------+------------------+
       |                 |                  |
       v                 v                  v
    SonarQube          Trivy             Veracode
       |                 |                  |
       +-----------------+------------------+
                         |
                         v
                    Docker Image
                         |
                         v
                       ECR
                         |
                         v
                    GitOps Repo
                         |
                         v
                  Central Argo CD
                 /        |        \
                /         |         \
               v          v          v
            EKS DEV    EKS QA     EKS PROD
               |          |          |
              ALB        ALB        ALB
```

---

# 261. Trust Boundaries

### Boundary 1

```text
Developer -> Application Git
```

Protected by:

```text
branch protection
PR
review
```

### Boundary 2

```text
CI -> ECR
```

Protected by:

```text
IAM
OIDC
least privilege
```

### Boundary 3

```text
CI -> GitOps
```

Protected by:

```text
scoped Git identity
PR
```

### Boundary 4

```text
Argo CD -> EKS
```

Protected by:

```text
Kubernetes RBAC
cluster credentials
Projects
```

---

# 262. Why This Architecture Is Production-Friendly

It provides:

```text
No direct CI-to-prod Kubernetes credentials
Immutable artifacts
Git-based desired state
Automated reconciliation
Environment isolation
Approval gates
Rollback
Auditability
Scalability
```

---

# 263. Common Anti-Patterns

### Anti-pattern 1

```text
Jenkins -> kubectl apply
```

for all environments.

### Anti-pattern 2

```text
latest
```

in production.

### Anti-pattern 3

```text
Different rebuild per environment
```

### Anti-pattern 4

```text
One cluster-admin token shared everywhere
```

### Anti-pattern 5

```text
Plaintext secrets in Git
```

### Anti-pattern 6

```text
CI modifying ApplicationSets automatically
```

without controls.

---

# 264. Anti-Pattern: GitOps Repository as Build Repository

Do not put:

```text
source code
Docker build
production values
```

into one uncontrolled repository simply because it is convenient.

Separation usually gives better ownership.

---

# 265. Anti-Pattern: Direct Production Push

Avoid:

```text
git push main
```

to production from an unreviewed CI job.

Use:

```text
PR
review
merge
Argo CD
```

---

# 266. Anti-Pattern: Environment-Specific Images

Avoid:

```text
cart-dev
cart-qa
cart-prod
```

when these are rebuilt independently.

Use:

```text
one immutable image
```

promoted through environments.

---

# 267. Anti-Pattern: Huge Values File

Avoid:

```text
5,000-line values.yaml
```

containing every environment and service.

Prefer:

```text
common values
service values
environment values
```

with clear ownership.

---

# 268. Anti-Pattern: Manual kubectl as Normal Process

If operators routinely run:

```bash
kubectl edit
kubectl patch
kubectl set image
```

in production, GitOps is losing its purpose.

Manual access should be emergency/diagnostic rather than the standard deployment mechanism.

---

# 269. Anti-Pattern: Ignoring Argo CD Health

Do not treat:

```text
Synced
```

as equivalent to:

```text
Healthy
```

Always check both.

---

# 270. Anti-Pattern: No Rollback Artifact

If ECR deletes the previous image immediately after deployment:

```text
rollback may fail
```

Keep a suitable retention window.

---

# 271. Anti-Pattern: No DR Test

Having:

```text
GitOps repository
```

does not prove recovery.

Actually test:

```text
cluster recreation
Argo CD installation
cluster registration
secret restoration
application reconciliation
```

---

# 272. Advanced Pattern: Promotion Repository

Some organizations separate:

```text
application configuration
```

from:

```text
environment promotion metadata
```

This can be useful at very large scale but adds complexity.

Use only when the organization needs it.

---

# 273. Advanced Pattern: Release Manifest

A release manifest can represent:

```text
service
version
image digest
environment
approval
```

This can become the source for automated promotion.

---

# 274. Advanced Pattern: GitOps Environment Lock

A production environment can have:

```text
freeze=true
```

or a sync window during incident response.

This prevents uncontrolled automated synchronization.

---

# 275. Advanced Pattern: Fleet Promotion

For many clusters:

```text
wave1
wave2
wave3
```

can be used.

Example:

```text
Wave 1:
1 production cluster

Wave 2:
20%

Wave 3:
remaining fleet
```

---

# 276. Advanced Pattern: Cluster Labels

Labels:

```text
environment=prod
region=ap-south-1
wave=1
role=primary
```

can drive ApplicationSet selection.

This is powerful and must be governed carefully.

---

# 277. Advanced Pattern: Multi-Region Promotion

Example:

```text
PROD AP-SOUTH-1
      |
      v
validation
      |
      v
PROD AP-SOUTHEAST-1
```

The same artifact can be deployed across regions.

---

# 278. Regional Configuration

Keep environment/region-specific values limited to:

```text
region
AWS endpoints
DNS
capacity
availability requirements
```

Do not fork the entire application configuration unnecessarily.

---

# 279. Multi-Account Promotion

```text
DEV AWS account
      |
      v
QA AWS account
      |
      v
PROD AWS account
```

Argo CD can manage target clusters while Git remains the desired-state source.

IAM boundaries should prevent unintended cross-account access.

---

# 280. ECR Cross-Account Consideration

Options include:

```text
central ECR
cross-account repository access
ECR replication
account-local ECR
```

Choose based on:

```text
security
network
latency
compliance
ownership
```

---

# 281. CI ECR Strategy

A common model:

```text
Build account
   |
   v
ECR
   |
   +--> DEV
   +--> QA
   +--> PROD
```

Another:

```text
DEV ECR
QA ECR
PROD ECR
```

The key is to preserve:

```text
artifact identity
```

across promotion.

---

# 282. Do Not Rebuild for Cross-Account Promotion

If moving an image between registries:

```text
copy/promote same digest
```

rather than:

```text
rebuild
```

where the architecture permits.

---

# 283. ECR Replication

ECR replication can help distribute images across accounts/regions.

The exact setup should be managed as infrastructure and tested.

---

# 284. GitOps and ECR Lifecycle

Define lifecycle policies that balance:

```text
storage cost
rollback requirements
DR
compliance
```

Do not delete production rollback candidates too aggressively.

---

# 285. Production Release Freeze

During freeze:

```text
CI can continue building
```

but:

```text
PROD promotion is blocked
```

This allows engineering work to continue without deploying it.

---

# 286. Decoupling Build and Release

This is an important GitOps principle.

Build:

```text
when code changes
```

Release:

```text
when environment is ready
```

They do not have to happen simultaneously.

---

# 287. Release Later

An image built on:

```text
August 19
```

can be promoted on:

```text
August 25
```

without rebuilding.

That is possible because the artifact is immutable.

---

# 288. Promotion by Digest

Example:

```text
sha256:abc...
```

can move:

```text
DEV -> QA -> PROD
```

without changing the artifact.

---

# 289. GitOps Release History

Git history becomes:

```text
Aug 19:
cart digest abc -> DEV

Aug 20:
cart digest abc -> QA

Aug 21:
cart digest abc -> PROD
```

This is an excellent audit trail.

---

# 290. Production Incident Reconstruction

Suppose an incident occurs.

Investigate:

```text
1. Argo CD deployed revision?
2. GitOps commit?
3. Image digest?
4. CI build?
5. Source commit?
6. Security scan?
7. PR approval?
8. Kubernetes events?
9. Prometheus metrics?
10. ELK logs?
```

This creates an end-to-end chain.

---

# 291. Deployment Failure Example

```text
CI passed
       |
       v
ECR push passed
       |
       v
GitOps PR passed
       |
       v
Argo CD sync passed
       |
       v
Pods CrashLoopBackOff
```

Root cause is likely runtime/application/configuration rather than GitOps transport.

---

# 292. GitOps Rendering Failure Example

```text
CI passed
       |
       v
GitOps changed
       |
       v
Argo CD
       |
       X
Helm template error
```

Root cause:

```text
deployment configuration
```

not necessarily application code.

---

# 293. ECR Failure Example

```text
Build passed
 |
 v
Docker build passed
 |
 X
ECR push failed
```

Do not update GitOps.

---

# 294. Git Failure Example

```text
ECR push passed
 |
 X
GitOps PR failed
```

The image remains unused.

Retry the GitOps promotion safely.

---

# 295. Argo CD Failure Example

```text
GitOps merged
 |
 v
Argo CD
 |
 X
RBAC denied
```

Check:

```text
target cluster credentials
Kubernetes RBAC
Project restrictions
```

---

# 296. Kubernetes Failure Example

```text
Argo CD sync
 |
 v
Deployment
 |
 X
Admission denied
```

Check:

```text
policy engine
securityContext
image policy
resource policy
```

---

# 297. Production Debugging Decision Tree

```text
Deployment failed?
       |
       +--> CI failed?
       |      |
       |      +--> fix CI
       |
       +--> ECR failed?
       |      |
       |      +--> fix registry/auth
       |
       +--> GitOps failed?
       |      |
       |      +--> fix Git/PR/render
       |
       +--> Argo CD failed?
       |      |
       |      +--> inspect Application
       |
       +--> Kubernetes failed?
              |
              +--> inspect pods/events/policies
```

---

# 298. Interview Question: Explain CI + GitOps

### Answer

> CI is responsible for building, testing, scanning and publishing the immutable application artifact. After the artifact is available, CI updates the GitOps repository, ideally through a controlled promotion PR. Argo CD watches the GitOps repository, calculates desired state, and reconciles that state into Kubernetes. This separates artifact creation from deployment and avoids giving CI broad production cluster credentials.

---

# 299. Interview Question: Why Should CI Not Run kubectl Apply?

### Answer

> Direct kubectl access gives CI credentials to the cluster and makes the pipeline responsible for deployment state. With GitOps, CI only publishes the artifact and changes Git. Argo CD performs deployment and reconciliation, which provides better auditability, drift correction and least privilege.

---

# 300. Interview Question: How Do You Promote the Same Image From DEV to PROD?

### Answer

> I build the image once, tag it with an immutable identifier and push it to ECR. DEV, QA and PROD GitOps configurations reference the same image tag or digest. Promotion changes the environment's GitOps reference rather than rebuilding the image.

---

# 301. Interview Question: How Does Argo CD Know CI Changed the Image?

### Answer

> CI updates the GitOps repository. Argo CD detects the Git revision through repository refresh or a webhook-triggered refresh. It compares desired state with live state. If the application is OutOfSync and automated synchronization is enabled, Argo CD applies the desired state.

---

# 302. Interview Question: Does a Git Webhook Deploy the Application?

### Answer

> Not directly. A webhook tells Argo CD that the repository changed and can accelerate refresh. Argo CD then calculates desired state and, depending on sync policy, performs synchronization. The actual deployment is performed through Kubernetes APIs.

---

# 303. Interview Question: What If the Webhook Fails?

### Answer

> Argo CD can still discover repository changes through its normal refresh/reconciliation behavior. The webhook improves responsiveness but should not be treated as the only mechanism for correctness.

---

# 304. Interview Question: How Do You Secure Jenkins in GitOps?

### Answer

> Jenkins should use short-lived AWS credentials or federated IAM roles for ECR, and a dedicated narrowly scoped identity for GitOps repository updates. It should not have cluster-admin access. Production changes should normally go through a protected GitOps PR.

---

# 305. Interview Question: How Do You Secure GitHub Actions?

### Answer

> I prefer GitHub OIDC to assume a narrowly scoped AWS IAM role rather than storing long-lived AWS keys. GitOps repository permissions should be restricted, production changes should use protected environments or PR review, and workflow permissions should follow least privilege.

---

# 306. Interview Question: What Happens if the GitOps Repo Changes but Deployment Does Not?

### Answer

> I check whether Argo CD refreshed the repository, then inspect Application status and diff. If the application is OutOfSync, I determine whether automated sync is disabled or whether synchronization failed. I then inspect Argo CD operation details and Kubernetes events.

---

# 307. Interview Question: What Is the Difference Between Refresh and Sync?

### Answer

> Refresh recalculates the application's desired and live state and discovers changes. Sync actually applies the desired state to the target cluster. An application can be OutOfSync after refresh without being deployed if automated sync is disabled.

---

# 308. Interview Question: Why Use GitOps PRs for Production?

### Answer

> A production PR provides a visible diff, reviewer approval, automated validation and a durable audit trail. It also keeps the desired production state in Git, making rollback and disaster recovery easier.

---

# 309. Interview Question: What If a Production Deployment Fails After Git Merge?

### Answer

> I first determine whether the failure is rendering, synchronization, Kubernetes admission, or application runtime. If the release is unhealthy, I revert the GitOps change to the last known-good image or use an emergency rollback mechanism if necessary. After emergency recovery, I make sure Git reflects the recovered desired state.

---

# 310. Interview Scenario: Jenkins Has Cluster-Admin

### Question

What would you change?

### Answer

I would remove direct cluster-admin access and redesign the flow:

```text
Jenkins -> ECR
Jenkins -> GitOps PR
Argo CD -> EKS
```

This reduces credential blast radius and centralizes deployment control.

---

# 311. Interview Scenario: Production Deploys Without Approval

Investigate:

```text
Application syncPolicy
ApplicationSet
Git branch protection
CI automation
Argo CD RBAC
sync windows
```

Then enforce:

```text
protected GitOps production path
+
required review
+
controlled synchronization
```

---

# 312. Interview Scenario: Same Image But PROD Fails

Answer:

> I verify the image digest first. If the digest is identical, I compare production-specific configuration, secrets, IAM, networking, database connectivity, resource limits, probes and external dependencies.

---

# 313. Interview Scenario: GitOps PR Has 500 Changed Lines

Investigate:

```text
values structure
formatting changes
chart version
generated files
unrelated environment changes
automation behavior
```

A promotion PR should ideally have a focused diff.

---

# 314. Interview Scenario: Two Pipelines Update the Same File

Potential result:

```text
lost update
merge conflict
incorrect image
```

Solutions:

```text
service-specific files
promotion queue
PR-based updates
serialized automation
```

---

# 315. Interview Scenario: ECR Image Missing

If GitOps points to:

```text
git-a1b2c3d
```

but ECR does not contain it:

```text
do not deploy
```

Fix:

```text
restore/publish exact artifact
```

or:

```text
revert GitOps
```

depending on release state.

---

# 316. Interview Scenario: Argo CD Sync Successful but Application Broken

Answer:

> Sync success means desired Kubernetes resources were applied. It does not guarantee business functionality. I check Deployment health, pod logs, probes, Service endpoints, Ingress/ALB behavior, application metrics and ELK logs.

---

# 317. Interview Scenario: Security Scan Passes but Cluster Rejects Image

Possible reasons:

```text
admission policy
image signature
registry allowlist
securityContext
```

The cluster's policy boundary is functioning.

---

# 318. Interview Scenario: Need Emergency Production Hotfix

Answer:

> I prefer a fast-tracked GitOps PR with the emergency artifact and required approval. If a direct cluster change is unavoidable to restore service, I treat it as an emergency exception and immediately reconcile the resulting state back into Git.

---

# 319. Interview Scenario: CI Is Down but Production Is Healthy

Production generally continues because:

```text
CI is not the runtime controller.
```

Existing workloads continue.

New releases cannot be built until CI recovers unless an existing artifact can be promoted through an approved process.

---

# 320. Interview Scenario: Argo CD Is Down

Existing workloads generally continue running.

However:

```text
new deployments
drift correction
reconciliation
```

are affected.

Restore Argo CD and verify reconciliation.

---

# 321. Interview Scenario: GitOps Repo Is Down

Existing workloads generally continue.

New desired-state changes cannot be safely committed until Git is available.

---

# 322. Interview Scenario: ECR Is Down

Existing pods may continue running.

New pod image pulls may fail.

Recovery depends on:

```text
registry availability
node image cache
replica state
multi-region/account design
```

---

# 323. Production CI/CD + GitOps Final Architecture

```text
                        Developer
                            |
                            v
                    Application Git
                            |
                            v
                +----------------------+
                | Jenkins / GitHub     |
                | Actions              |
                +----------------------+
                            |
       +--------------------+--------------------+
       |                    |                    |
       v                    v                    v
   Build/Test           SonarQube             Security
                                              Trivy/Veracode
       |                    |                    |
       +--------------------+--------------------+
                            |
                            v
                       Docker Build
                            |
                            v
                           ECR
                            |
                            v
                    GitOps Repository
                            |
                     PR / Promotion
                            |
                            v
                      Central Argo CD
                       /     |      \
                      /      |       \
                     v       v        v
                 EKS-DEV  EKS-QA   EKS-PROD
                     |       |        |
                    ALB     ALB      ALB
                     |       |        |
                     +-------+--------+
                             |
                           Users
```

---

# 324. Final Production Principles

1. CI builds.
2. CI tests.
3. CI scans.
4. CI publishes.
5. GitOps stores desired deployment state.
6. Argo CD reconciles.
7. Kubernetes runs workloads.
8. ECR stores immutable artifacts.
9. Production changes use controlled Git workflows.
10. CI should not require cluster-admin.
11. Promote the same artifact.
12. Prefer image digests for deterministic deployment.
13. Use PRs for production promotion.
14. Use ApplicationSets for scale.
15. Use Projects for boundaries.
16. Keep secrets out of plaintext Git.
17. Monitor both deployment and runtime health.
18. Keep rollback artifacts.
19. Test disaster recovery.
20. Treat GitOps repository security as production infrastructure security.

---

# 325. One-Minute Interview Explanation

> In my DevOps architecture, CI and CD are separated. Jenkins or GitHub Actions handles source checkout, build, testing, SonarQube, Trivy, Veracode, Docker image creation and publishing to ECR. The pipeline does not directly deploy to EKS. Instead, it updates the GitOps repository with the immutable image tag or digest. For DEV this can be an automated GitOps update, while QA and PROD use controlled promotion PRs and approvals. Argo CD watches the GitOps repository, compares desired state with the live EKS cluster and reconciles the application. This gives us pull-based deployment, drift detection, auditability, rollback and reduced CI-to-cluster credentials. In the RoboShop platform, the same image is promoted from DEV to QA to PROD, while environment-specific configuration is maintained separately in Helm values or Kustomize overlays. Terraform remains responsible for AWS/EKS infrastructure, while Argo CD manages Kubernetes application resources.

---