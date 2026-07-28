# DevSecOps in GitHub Actions

## Introduction

GitHub Actions is GitHub's native Continuous Integration and Continuous Delivery (CI/CD) platform that automates software builds, testing, security validation, deployments, and release workflows directly from GitHub repositories.

DevSecOps extends GitHub Actions by integrating automated security checks into every workflow, ensuring vulnerabilities are detected before applications are deployed.

Instead of running security after development, GitHub Actions executes security checks automatically with every push, pull request, and merge.

---

# Why Companies Use GitHub Actions for DevSecOps

GitHub Actions enables development and security teams to build secure software directly inside GitHub without requiring a separate CI platform.

## Benefits

- Native GitHub Integration
- Automated CI/CD
- Shift-Left Security
- Secure Software Supply Chain
- Policy Enforcement
- Infrastructure Automation
- Secret Management
- Enterprise Scalability
- GitOps Integration
- Marketplace Integrations

---

# Jenkins vs GitHub Actions

| Feature | Jenkins | GitHub Actions |
|----------|----------|----------------|
| Installation | Self-hosted | Managed by GitHub |
| Pipeline Definition | Jenkinsfile | YAML Workflow |
| Build Agents | Jenkins Agents | GitHub Runners |
| Marketplace | Plugins | GitHub Marketplace |
| Secret Management | Credentials Store | GitHub Secrets |
| Scaling | Manual | Automatic |
| GitHub Integration | Plugin | Native |
| Maintenance | High | Low |

Both platforms support enterprise DevSecOps pipelines.

---

# DevOps vs DevSecOps Workflow

## Traditional DevOps

```text
Developer

↓

Git Push

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

Security occurs after deployment.

---

## DevSecOps

```text
Developer

↓

Git Push

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Dependency Scan

↓

Secret Scan

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

Security becomes part of every workflow.

---

# Where GitHub Actions Fits in DevSecOps

```text
Developer

↓

Feature Branch

↓

Git Push

↓

Pull Request

↓

GitHub Actions

↓

Build

↓

Security Pipeline

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

GitHub Actions orchestrates automated security validation before deployment.

---

# Enterprise DevSecOps Architecture

```text
                  Developers
                       │
                       ▼
                GitHub Repository
                       │
                 Push / Pull Request
                       │
                       ▼
               GitHub Actions Runner
                       │
      ┌────────────────┼────────────────┐
      ▼                ▼                ▼
 SonarQube      Dependency Check    Gitleaks
      │                │                │
      ▼                ▼                ▼
  Checkov           TFSec          Docker Build
      │                │                │
      └────────────────┼────────────────┘
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
                  Amazon ECR
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

Git Push

↓

Pull Request

↓

Code Review

↓

Merge

↓

GitHub Actions Trigger

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

Amazon ECR

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

Every workflow includes multiple automated security gates.

---

# Why Security Is Added at Every Stage

| Stage | Security Tool |
|---------|---------------|
| Source Code | SonarQube |
| Dependencies | OWASP Dependency-Check |
| Secrets | Gitleaks |
| Terraform | Checkov |
| Terraform | TFSec |
| Container | Trivy |
| Supply Chain | SBOM |
| Image Integrity | Cosign |
| Deployment | ArgoCD |
| Runtime | Falco / Aqua / Prisma |

Each stage protects a different part of the software supply chain.

---

# GitHub Actions Workflow Structure

```text
.github/

└── workflows/

    ├── ci.yml

    ├── security.yml

    ├── deploy.yml

    └── release.yml
```

Keeping workflows separated improves readability and maintenance.

---

# Prerequisites

| Component | Purpose |
|-----------|----------|
| GitHub Repository | Source Code |
| GitHub Actions | CI/CD |
| Docker | Container Build |
| Kubernetes | Deployment Platform |
| Amazon EKS | Container Orchestration |
| Helm | Kubernetes Package Manager |
| SonarQube | Code Analysis |
| OWASP Dependency-Check | Dependency Security |
| Gitleaks | Secret Detection |
| Checkov | IaC Security |
| TFSec | Terraform Security |
| Trivy | Container Security |
| Cosign | Image Signing |
| ArgoCD | GitOps Deployment |

---

# GitHub Actions Runners

GitHub Actions workflows execute on runners.

Runner types.

| Runner | Description |
|----------|-------------|
| GitHub Hosted | Managed by GitHub |
| Self-Hosted | Managed by Organization |
| Kubernetes Runner | Autoscaling Kubernetes Runner |

Enterprise environments often use self-hosted or Kubernetes-based runners for security and scalability.

---

# GitHub Events

Common workflow triggers.

| Event | Purpose |
|---------|----------|
| push | Code Push |
| pull_request | Pull Request Validation |
| workflow_dispatch | Manual Execution |
| release | Release Pipeline |
| schedule | Scheduled Jobs |

Security validation should run on both `push` and `pull_request` events.

---

# First Workflow

Example.

```yaml
name: CI

on:

  push:

    branches:

      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - name: Build

        run: mvn clean package
```

This workflow automatically builds the application whenever code is pushed to the `main` branch.

---

