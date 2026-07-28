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

