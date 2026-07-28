# DevSecOps in GitLab

## Introduction

GitLab is a complete DevSecOps platform that combines Source Code Management (SCM), CI/CD, security testing, package management, GitOps, and monitoring into a single application.

Unlike traditional CI/CD tools, GitLab provides many security capabilities out of the box, allowing organizations to implement DevSecOps without integrating numerous third-party products.

DevSecOps in GitLab ensures security is integrated into every phase of the Software Development Life Cycle (SDLC).

---

# Why Companies Use GitLab for DevSecOps

GitLab provides an end-to-end software delivery platform with built-in security capabilities.

## Benefits

- Single DevSecOps Platform
- Integrated Source Control
- Built-in CI/CD
- Built-in Security Testing
- Merge Request Validation
- Container Registry
- Package Registry
- GitOps Integration
- Kubernetes Integration
- Enterprise Governance

---

# GitHub Actions vs GitLab CI/CD

| Feature | GitHub Actions | GitLab CI/CD |
|----------|----------------|--------------|
| Repository | GitHub | GitLab |
| Pipeline File | workflow.yml | .gitlab-ci.yml |
| Runner | GitHub Runner | GitLab Runner |
| Container Registry | External / GitHub | Built-in |
| Package Registry | External | Built-in |
| Security Dashboard | Limited | Built-in |
| Merge Requests | GitHub PR | GitLab MR |
| Auto DevOps | No | Yes |
| GitOps Support | Yes | Yes |

Both platforms support enterprise DevSecOps workflows.

---

# DevOps vs DevSecOps

## Traditional DevOps

```text
Developer

↓

Commit

↓

Build

↓

Test

↓

Deploy

↓

Production

↓

Security Testing
```

Security occurs late in the release cycle.

---

## DevSecOps

```text
Developer

↓

Commit

↓

Build

↓

Unit Tests

↓

SAST

↓

Dependency Scan

↓

Secret Detection

↓

IaC Scan

↓

Container Scan

↓

SBOM

↓

Image Signing

↓

Deploy

↓

Production
```

Security is integrated throughout the pipeline.

---

# Where GitLab Fits in DevSecOps

```text
Developer

↓

GitLab Repository

↓

Merge Request

↓

GitLab CI/CD

↓

Security Validation

↓

Container Registry

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

GitLab orchestrates secure software delivery from commit to production.

---

# Enterprise DevSecOps Architecture

```text
                  Developers
                       │
                       ▼
               GitLab Repository
                       │
               Merge Request / Push
                       │
                       ▼
                GitLab CI/CD Runner
                       │
      ┌────────────────┼─────────────────┐
      ▼                ▼                 ▼
  SonarQube      Dependency Check    Gitleaks
      │                │                 │
      ▼                ▼                 ▼
  Checkov           TFSec          Docker Build
      │                │                 │
      └────────────────┼─────────────────┘
                       ▼
                  Trivy Scan
                       │
                       ▼
                 Generate SBOM
                       │
                       ▼
                 Cosign Signing
                       │
                       ▼
            GitLab Container Registry
                       │
                       ▼
                GitOps Repository
                       │
                       ▼
                    ArgoCD
                       │
                       ▼
                  Amazon EKS
                       │
                       ▼
        Falco / Aqua / Prisma Runtime
```

---

# Enterprise Production Pipeline

```text
Developer

↓

Feature Branch

↓

Commit

↓

Push

↓

Merge Request

↓

Code Review

↓

Merge

↓

GitLab Pipeline

↓

Checkout

↓

Build

↓

Unit Tests

↓

Coverage

↓

SonarQube

↓

OWASP Dependency-Check

↓

Gitleaks

↓

Checkov

↓

TFSec

↓

Docker Build

↓

Trivy

↓

SBOM

↓

Cosign Image Signing

↓

GitLab Container Registry

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Falco Runtime Monitoring

↓

Production
```

Every pipeline stage validates security before deployment.

---

# GitLab CI/CD Components

| Component | Purpose |
|-----------|---------|
| Repository | Source Code |
| Merge Request | Code Review |
| Pipeline | CI/CD Workflow |
| Runner | Job Execution |
| Container Registry | Image Storage |
| Package Registry | Package Storage |
| Security Dashboard | Security Findings |
| Environments | Deployment Targets |

---

# GitLab Pipeline Stages

```text
.gitlab-ci.yml

↓

Stages

├── Build

├── Test

├── Security

├── Package

└── Deploy
```

Each stage groups related jobs for easier pipeline management.

---

# Prerequisites

| Component | Purpose |
|-----------|----------|
| GitLab | Source Control |
| GitLab Runner | Job Execution |
| Docker | Container Build |
| Kubernetes | Container Platform |
| Amazon EKS | Kubernetes Cluster |
| SonarQube | Code Analysis |
| OWASP Dependency-Check | Dependency Security |
| Gitleaks | Secret Detection |
| Checkov | IaC Security |
| TFSec | Terraform Security |
| Trivy | Container Security |
| Cosign | Image Signing |
| ArgoCD | GitOps Deployment |

---

# GitLab Runners

GitLab pipelines execute on GitLab Runners.

Runner types.

| Runner | Description |
|---------|-------------|
| Shared Runner | Managed by GitLab |
| Group Runner | Shared within a Group |
| Project Runner | Dedicated to a Project |
| Self-Hosted Runner | Managed by Organization |

Enterprise environments commonly use dedicated self-hosted runners.

---

# GitLab Events

Common pipeline triggers.

| Event | Purpose |
|--------|----------|
| Push | Commit Validation |
| Merge Request | Security Validation |
| Tag | Release Pipeline |
| Schedule | Scheduled Jobs |
| Manual | On-demand Execution |

Security validation should execute for every Merge Request.

---

# First GitLab Pipeline

Example.

```yaml
stages:

  - build

build:

  stage: build

  script:

    - mvn clean package
```

The pipeline automatically builds the application after every commit.

---

# Repository Structure

A well-structured repository simplifies CI/CD and security workflow management.

```text
project/

├── .gitlab-ci.yml

├── src/

├── terraform/

├── kubernetes/

├── Dockerfile

├── pom.xml

└── README.md
```

Keep infrastructure, application, and deployment manifests organized.

---

# GitLab Variables

Sensitive information should never be stored inside the pipeline file.

Examples.

```text
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

SONAR_TOKEN

DOCKER_USERNAME

DOCKER_PASSWORD

COSIGN_PRIVATE_KEY

KUBECONFIG

ARGOCD_TOKEN
```

GitLab encrypts CI/CD variables and injects them securely into jobs.

---

# Environment Variables

Example.

```yaml
variables:

  IMAGE_NAME: payment-service

  REGISTRY: registry.gitlab.com/company/project
```

Variables simplify pipeline maintenance and reduce duplication.

---

# Protected Variables

Protected variables are only available to protected branches and tags.

Examples.

```text
main

release/*

production
```

Production credentials should always be configured as protected variables.

---

# GitLab Environments

GitLab environments represent deployment targets.

Examples.

```text
Development

Testing

Staging

Production
```

Each deployment updates the corresponding environment.

---

# GitLab Environment Workflow

```text
Commit

↓

Pipeline

↓

Deploy

↓

Environment

↓

Development

↓

Testing

↓

Production
```

Environments provide deployment visibility and history.

---

# GitLab Runners

GitLab Runners execute pipeline jobs.

```text
Developer

↓

GitLab

↓

GitLab Runner

↓

Build

↓

Security

↓

Deploy
```

Self-hosted runners are recommended for enterprise workloads.

---

# Docker Executor

The Docker executor creates isolated build environments.

```text
GitLab Runner

↓

Docker Container

↓

Pipeline Job

↓

Container Removed
```

Every job starts with a clean execution environment.

---

# Kubernetes Executor

GitLab Runner can launch temporary Kubernetes Pods.

```text
GitLab Runner

↓

Kubernetes

↓

Runner Pod

↓

Pipeline

↓

Pod Deleted
```

Ephemeral Pods improve scalability and security.

---

# Merge Request Validation

Security validation should occur before merging code.

```text
Developer

↓

Merge Request

↓

Pipeline

↓

Security Validation

↓

Approval

↓

Merge
```

Only validated code should reach the default branch.

---

# Required Pipeline Checks

Typical mandatory jobs.

```text
Build

Unit Tests

SonarQube

Dependency Check

Gitleaks

Checkov

TFSec

Trivy
```

Merge Requests should be blocked if any required job fails.

---

# Build Stage

Example.

```yaml
build:

  stage: build

  script:

    - mvn clean package
```

Compilation errors should immediately stop the pipeline.

---

# Unit Testing

Example.

```yaml
unit-test:

  stage: test

  script:

    - mvn test
```

Execute unit tests before packaging or deployment.

---

# Code Coverage

Example.

```yaml
coverage:

  stage: test

  script:

    - mvn jacoco:report
```

Coverage reports help identify untested application code.

---

# SonarQube Integration

Example.

```yaml
sonarqube:

  stage: security

  script:

    - mvn sonar:sonar \
      -Dsonar.login=$SONAR_TOKEN
```

SonarQube performs static application security testing and code quality analysis.

---

# SonarQube Quality Gate

Every production pipeline should enforce the Quality Gate.

```text
Source Code

↓

SonarQube

↓

Quality Gate

↓

Passed?

     │

┌────┴─────┐

▼          ▼

Yes         No

│            │

Continue   Stop Pipeline
```

Applications failing the Quality Gate should not proceed.

---

# Pipeline Artifacts

Artifacts allow jobs to share generated files.

Example.

```yaml
artifacts:

  paths:

    - target/

    - reports/
```

Artifacts preserve build outputs and security reports.

---

# Pipeline Dependencies

Jobs can consume artifacts from earlier stages.

Example.

```yaml
dependencies:

  - build
```

Dependencies reduce unnecessary rebuilding.

---

# Pipeline Cache

Caching speeds up repeated executions.

Example.

```yaml
cache:

  paths:

    - .m2/repository
```

Package caches significantly reduce build duration.

---

# Enterprise Best Practices

- Store all sensitive values as protected CI/CD variables.
- Use dedicated self-hosted runners for production workloads.
- Execute pipelines for every Merge Request.
- Enforce SonarQube Quality Gates.
- Cache dependencies to improve pipeline speed.
- Use pipeline artifacts for reports and build outputs.
- Deploy using Kubernetes-based runners when possible.
- Protect production branches and environments.
- Separate build, test, security, and deployment stages.
- Continuously update GitLab Runner versions.

---

# OWASP Dependency-Check Integration

OWASP Dependency-Check scans application dependencies for known vulnerabilities.

Example.

```yaml
dependency-check:

  stage: security

  script:

    - dependency-check.sh \
      --scan . \
      --format HTML \
      --out reports
```

Fail the pipeline if High or Critical vulnerabilities are detected.

---

# Gitleaks Integration

Gitleaks scans the repository for exposed secrets.

Example.

```yaml
gitleaks:

  stage: security

  script:

    - gitleaks detect \
      --source . \
      --report-format json \
      --report-path gitleaks-report.json
```

Typical secrets detected.

- AWS Access Keys
- GitHub Tokens
- Azure Credentials
- SSH Private Keys
- API Keys
- Database Passwords

---

# Checkov Integration

Checkov validates Infrastructure as Code security.

Example.

```yaml
checkov:

  stage: security

  script:

    - checkov -d .
```

Checkov scans Terraform, Kubernetes manifests, Dockerfiles, and cloud configurations.

---

# TFSec Integration

TFSec scans Terraform code for security misconfigurations.

Example.

```yaml
tfsec:

  stage: security

  script:

    - tfsec .
```

Terraform resources should be validated before infrastructure deployment.

---

# Docker Build

Build the container image only after source code passes security validation.

Example.

```yaml
docker-build:

  stage: package

  script:

    - docker build \
      -t payment-service:$CI_PIPELINE_ID .
```

---

# Trivy Integration

Trivy scans container images for vulnerabilities.

Example.

```yaml
trivy:

  stage: security

  script:

    - trivy image \
      --exit-code 1 \
      payment-service:$CI_PIPELINE_ID
```

Stop the pipeline when High or Critical vulnerabilities are found.

---

# SBOM Generation

Generate a Software Bill of Materials after the container scan.

Example.

```yaml
sbom:

  stage: security

  script:

    - trivy image \
      --format cyclonedx \
      --output sbom.json \
      payment-service:$CI_PIPELINE_ID
```

Archive the SBOM for compliance and auditing.

---

# Cosign Image Signing

Digitally sign container images before publishing.

Example.

```yaml
cosign:

  stage: package

  script:

    - cosign sign \
      payment-service:$CI_PIPELINE_ID
```

Only signed images should be released.

---

# Login to GitLab Container Registry

Authenticate before pushing images.

Example.

```yaml
registry-login:

  stage: package

  script:

    - docker login \
      -u $CI_REGISTRY_USER \
      -p $CI_REGISTRY_PASSWORD \
      $CI_REGISTRY
```

GitLab automatically provides registry authentication variables.

---

# Push Container Image

Push verified container images to the GitLab Container Registry.

Example.

```yaml
push-image:

  stage: package

  script:

    - docker push \
      $CI_REGISTRY_IMAGE:$CI_PIPELINE_ID
```

Publish only scanned and signed images.

---

# Update GitOps Repository

GitLab CI updates deployment manifests instead of deploying directly.

```text
GitLab Pipeline

↓

Build Success

↓

Update Image Tag

↓

Commit Changes

↓

Push

↓

GitOps Repository
```

Git remains the deployment source of truth.

---

# Update Deployment Manifest

Example.

```yaml
update-gitops:

  stage: deploy

  script:

    - sed -i "s/tag:.*/tag: $CI_PIPELINE_ID/" deployment.yaml

    - git add .

    - git commit -m "Update image"

    - git push
```

ArgoCD automatically detects the new image version.

---

# ArgoCD Deployment

Deployment flow.

```text
GitOps Repository

↓

ArgoCD

↓

Compare Desired State

↓

Synchronize

↓

Amazon EKS

↓

Production
```

GitLab CI should update Git only, not deploy directly to Kubernetes.

---

# Deployment Validation

Validate the deployment after synchronization.

Example.

```yaml
validation:

  stage: deploy

  script:

    - kubectl get deployments

    - kubectl get pods
```

Production pipelines should also execute smoke tests.

---

# Security Reports

Generate reports from every security tool.

```text
Security Reports

├── SonarQube

├── Dependency Check

├── Gitleaks

├── Checkov

├── TFSec

├── Trivy

├── SBOM

└── Pipeline Logs
```

Store reports for compliance and future investigations.

---

# Upload Security Reports

Example.

```yaml
artifacts:

  paths:

    - reports/

    - sbom.json
```

Artifacts remain available after the pipeline completes.

---

# Security Gate Workflow

Every security stage validates the application before deployment.

```text
Checkout

↓

Build

↓

Unit Tests

↓

Security Scans

↓

Passed?

     │

┌────┴─────┐

▼          ▼

Yes         No

│            │

Continue   Stop Pipeline
```

Critical vulnerabilities should immediately stop the pipeline.

---

# Parallel Security Jobs

GitLab CI supports running independent security jobs simultaneously.

```text
Build

↓

Parallel Jobs

├── SonarQube

├── Dependency Check

├── Gitleaks

├── Checkov

├── TFSec

↓

Results

↓

Docker Build
```

Parallel execution reduces total pipeline duration.

---

# Enterprise Best Practices

- Run source code scans before building container images.
- Execute security jobs in parallel whenever possible.
- Stop pipelines on High and Critical vulnerabilities.
- Generate an SBOM for every production image.
- Sign all production images using Cosign.
- Push only verified images to the registry.
- Archive security reports and SBOMs.
- Use GitOps instead of direct Kubernetes deployments.
- Validate deployments after ArgoCD synchronization.
- Keep security tools and GitLab Runners up to date.

---

# Complete Production GitLab CI/CD Pipeline

The following `.gitlab-ci.yml` demonstrates an enterprise DevSecOps pipeline integrating build, testing, security validation, containerization, image signing, and GitOps.

```yaml
stages:

  - build

  - test

  - security

  - package

  - deploy

variables:

  IMAGE: $CI_REGISTRY_IMAGE:$CI_PIPELINE_ID

build:

  stage: build

  script:

    - mvn clean package

unit-test:

  stage: test

  script:

    - mvn test

coverage:

  stage: test

  script:

    - mvn jacoco:report

sonarqube:

  stage: security

  script:

    - mvn sonar:sonar

dependency-check:

  stage: security

  script:

    - dependency-check.sh --scan .

gitleaks:

  stage: security

  script:

    - gitleaks detect --source .

checkov:

  stage: security

  script:

    - checkov -d .

tfsec:

  stage: security

  script:

    - tfsec .

docker-build:

  stage: package

  script:

    - docker build -t $IMAGE .

trivy:

  stage: security

  script:

    - trivy image --exit-code 1 $IMAGE

sbom:

  stage: security

  script:

    - trivy image --format cyclonedx --output sbom.json $IMAGE

cosign:

  stage: package

  script:

    - cosign sign $IMAGE

push-image:

  stage: package

  script:

    - docker push $IMAGE
```

---

# Enterprise Pipeline Flow

```text
Developer

↓

Commit

↓

Merge Request

↓

GitLab Pipeline

↓

Checkout

↓

Build

↓

Tests

↓

Security Validation

↓

Container Build

↓

Container Scan

↓

Generate SBOM

↓

Image Signing

↓

GitLab Container Registry

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

Every stage contributes to secure software delivery.

---

# Parallel Security Jobs

GitLab allows multiple jobs to execute simultaneously.

Example.

```yaml
sonarqube:

  stage: security

dependency-check:

  stage: security

gitleaks:

  stage: security

checkov:

  stage: security

tfsec:

  stage: security
```

All jobs inside the same stage execute in parallel whenever possible.

---

# Parallel Execution Architecture

```text
Checkout

↓

Build

↓

Parallel Security Jobs

├── SonarQube

├── Dependency Check

├── Gitleaks

├── Checkov

├── TFSec

↓

Merge Results

↓

Docker Build
```

Running independent scans together significantly reduces pipeline duration.

---

# Upload Build Artifacts

Store generated reports and binaries.

Example.

```yaml
artifacts:

  paths:

    - target/

    - reports/

    - sbom.json
```

Artifacts are retained after pipeline completion.

---

# Dependency Cache

Caching reduces repeated downloads.

Example.

```yaml
cache:

  key: maven-cache

  paths:

    - .m2/repository
```

Dependency caching speeds up subsequent pipeline executions.

---

# Pipeline Rules

Control when jobs execute.

Example.

```yaml
rules:

  - if: '$CI_COMMIT_BRANCH == "main"'
```

Rules reduce unnecessary pipeline executions.

---

# Environment Configuration

Define deployment environments.

Example.

```yaml
environment:

  name: Production
```

GitLab tracks deployment history for every environment.

---

# Protected Environments

Production deployments can require approvals.

```text
Pipeline

↓

Security Passed

↓

Production Environment

↓

Approval

↓

Deployment
```

Protected environments reduce deployment risk.

---

# Manual Deployment

Production deployments are often manual.

Example.

```yaml
deploy-production:

  stage: deploy

  when: manual
```

Manual approval provides an additional security checkpoint.

---

# GitLab Container Registry

Store verified container images.

```text
Pipeline

↓

Docker Build

↓

Trivy

↓

Cosign

↓

Container Registry
```

Only scanned and signed images should be published.

---

# Pipeline Notifications

Notify teams after pipeline execution.

Supported integrations.

- Email
- Slack
- Microsoft Teams
- Webhooks

Workflow.

```text
Pipeline

↓

Completed

↓

Notification

↓

Development Team

↓

Security Team
```

Immediate notifications improve operational awareness.

---

# Failure Strategy

Critical findings should immediately stop deployments.

```text
Pipeline

↓

Security Tool

↓

Critical Findings?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Stop      Continue
```

Failing early reduces security risk.

---

# Monitoring GitLab Pipelines

Monitor pipeline health continuously.

Recommended metrics.

- Pipeline Duration
- Success Rate
- Failure Rate
- Queue Time
- Runner Utilization
- Security Scan Duration
- Deployment Frequency
- Artifact Size

---

# Logging Architecture

```text
GitLab Pipeline

↓

Job Logs

↓

Log Collection

↓

Elastic Stack

↓

Search

↓

Analysis
```

Centralized logging simplifies troubleshooting and compliance.

---

# Security Hardening

Secure GitLab before onboarding development teams.

Recommendations.

- Enable Multi-Factor Authentication.
- Protect production branches.
- Protect production environments.
- Require Merge Requests.
- Enforce Code Reviews.
- Use protected CI/CD variables.
- Restrict Runner access.
- Keep GitLab Runners updated.
- Review pipeline changes through Merge Requests.
- Enable audit logging.

---

# Enterprise Best Practices

- Separate build, test, security, package, and deployment stages.
- Execute security jobs in parallel.
- Generate an SBOM for every production build.
- Sign every production container image.
- Push only verified images to the Container Registry.
- Protect production environments with approvals.
- Archive reports and SBOMs.
- Monitor pipeline performance.
- Centralize logs for auditing.
- Continuously update GitLab and GitLab Runners.

---

# Common Mistakes

## Mistake 1

### Storing Secrets in .gitlab-ci.yml

**Problem**

Sensitive credentials are hardcoded inside the pipeline.

```yaml
variables:

  AWS_ACCESS_KEY_ID: AKIAxxxxxxxx

  AWS_SECRET_ACCESS_KEY: xxxxxxxxxxxxxxxxx
```

**Impact**

- Credential leakage
- Security incidents
- Compliance violations

**Recommendation**

Store secrets as GitLab Protected CI/CD Variables or use cloud identity federation where supported.

---

## Mistake 2

### Deploying Without Security Validation

**Problem**

The application is deployed immediately after the build.

```text
Checkout

↓

Build

↓

Deploy
```

**Impact**

- Vulnerable applications
- Policy violations
- Increased production risk

**Recommendation**

Execute security scans before packaging and deployment.

---

## Mistake 3

### Deploying Directly to Kubernetes

**Problem**

GitLab CI executes Kubernetes deployments directly.

```text
GitLab Pipeline

↓

kubectl apply

↓

Production
```

**Impact**

- Configuration drift
- Difficult rollback
- No GitOps audit trail

**Recommendation**

Update the GitOps repository and allow ArgoCD to synchronize changes.

---

## Mistake 4

### Ignoring Failed Security Jobs

**Problem**

The pipeline continues after Critical vulnerabilities are detected.

**Impact**

- Vulnerable software reaches production
- Security policy violations
- Increased attack surface

**Recommendation**

Fail the pipeline whenever security jobs report High or Critical findings.

---

## Mistake 5

### Sharing Production Runners

**Problem**

Development and production workloads execute on the same runner.

**Impact**

- Security isolation issues
- Resource contention
- Increased operational risk

**Recommendation**

Use dedicated runners for production workloads.

---

# Troubleshooting

## Scenario 1

### Pipeline Does Not Start

**Cause**

The pipeline configuration is invalid.

**Resolution**

Verify:

- `.gitlab-ci.yml` exists in the repository root.
- YAML syntax is correct.
- Pipeline rules allow execution.
- GitLab Runner is online.

Example.

```yaml
stages:

  - build

build:

  stage: build

  script:

    - echo "Pipeline Started"
```

---

## Scenario 2

### SonarQube Scan Fails

**Cause**

Authentication or connectivity issues.

**Resolution**

Verify:

- SONAR_TOKEN variable
- SonarQube URL
- Project Key
- Network connectivity
- Runner access

---

## Scenario 3

### Container Push Fails

**Cause**

Authentication to the GitLab Container Registry fails.

**Resolution**

Verify:

```bash
docker login \
$CI_REGISTRY
```

Check:

- Registry credentials
- Repository permissions
- Image name
- Runner network access

---

## Scenario 4

### Trivy Reports Critical Vulnerabilities

**Cause**

The container image contains vulnerable packages.

**Resolution**

```bash
trivy image payment-service:latest
```

Update:

- Base image
- Operating system packages
- Application dependencies

Rebuild and rescan before deployment.

---

## Scenario 5

### ArgoCD Does Not Deploy

**Cause**

The GitOps repository was not updated correctly.

**Resolution**

Verify:

```bash
argocd app sync payment-service

kubectl get applications -n argocd
```

Check:

- Repository updates
- Image tag changes
- Repository permissions
- ArgoCD synchronization status

---

# Production Interview Questions

## Question 1

### What is GitLab CI/CD?

**Answer**

GitLab CI/CD is GitLab's built-in Continuous Integration and Continuous Delivery platform that automates application building, testing, security scanning, packaging, and deployment using `.gitlab-ci.yml`.

---

## Question 2

### Why is GitLab suitable for DevSecOps?

**Answer**

GitLab combines source control, CI/CD, security testing, package management, container registry, and deployment capabilities into a single platform, simplifying DevSecOps implementation.

---

## Question 3

### What is a GitLab Runner?

**Answer**

A GitLab Runner is an agent that executes pipeline jobs. It can run on virtual machines, physical servers, Docker containers, or Kubernetes clusters.

---

## Question 4

### Why use Protected Variables?

**Answer**

Protected Variables ensure sensitive credentials are only available to protected branches and tags, reducing the risk of credential exposure.

---

## Question 5

### Which security tools are commonly integrated into GitLab CI?

**Answer**

Common integrations include SonarQube, OWASP Dependency-Check, Gitleaks, Checkov, TFSec, Trivy, Cosign, Falco, Aqua Security, Prisma Cloud, and ArgoCD.

---

## Question 6

### Why generate an SBOM?

**Answer**

An SBOM provides a complete inventory of software components, helping organizations identify vulnerable dependencies, improve supply chain visibility, and support compliance requirements.

---

## Question 7

### Why should container images be signed?

**Answer**

Image signing verifies authenticity and integrity, ensuring that only trusted and untampered container images are deployed.

---

## Question 8

### Why is GitOps preferred over direct deployment?

**Answer**

GitOps provides version-controlled deployments, automated reconciliation, easier rollbacks, auditability, and consistent cluster state management.

---

## Question 9

### How can GitLab pipelines be optimized?

**Answer**

Use dependency caching, parallel stages, dedicated runners, reusable templates, pipeline rules, and artifacts to improve performance and scalability.

---

## Question 10

### What are enterprise best practices for GitLab DevSecOps?

**Answer**

Use protected branches, protected variables, dedicated runners, security gates, image signing, SBOM generation, GitOps deployment, centralized logging, and continuous monitoring.

---

# Key Takeaways

- GitLab provides an integrated platform for enterprise DevSecOps.
- Execute security scans throughout the CI/CD pipeline.
- Run SonarQube, OWASP Dependency-Check, Gitleaks, Checkov, TFSec, and Trivy before deployment.
- Generate an SBOM for every production image.
- Sign container images using Cosign.
- Store credentials securely using Protected CI/CD Variables.
- Use GitOps with ArgoCD instead of direct Kubernetes deployments.
- Protect production branches, environments, and runners.
- Archive reports for auditing and compliance.
- Continuously monitor pipeline health and security.

---

# Enterprise DevSecOps Pipeline Summary

```text
Developer

↓

Feature Branch

↓

Commit

↓

Push

↓

Merge Request

↓

Code Review

↓

Merge

↓

GitLab Pipeline

↓

Checkout

↓

Build

↓

Unit Tests

↓

Code Coverage

↓

SonarQube

↓

OWASP Dependency-Check

↓

Gitleaks

↓

Checkov

↓

TFSec

↓

Docker Build

↓

Trivy

↓

Generate SBOM

↓

Cosign Image Signing

↓

GitLab Container Registry

↓

Update GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Falco / Aqua / Prisma Runtime Protection

↓

Production
```

This pipeline represents a complete enterprise DevSecOps implementation using GitLab, integrating secure software development, automated security validation, GitOps-based deployments, and runtime security to deliver scalable, secure, and auditable applications.