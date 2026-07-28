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

