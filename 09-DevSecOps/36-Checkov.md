# Checkov

## Introduction

Checkov is an open-source Infrastructure as Code (IaC) security scanner developed by Bridgecrew (now part of Palo Alto Networks).

It scans Infrastructure as Code templates, Kubernetes manifests, Dockerfiles, CI/CD pipelines, and cloud configurations to detect security and compliance issues before deployment.

Checkov enables organizations to shift security left by identifying misconfigurations during development and CI/CD rather than after infrastructure is provisioned.

---

# Why Companies Use Checkov

Infrastructure is increasingly defined as code.

A single insecure Terraform resource or Kubernetes manifest can expose an entire production environment.

Checkov identifies these security issues before infrastructure reaches production.

## Benefits

- Infrastructure as Code security scanning
- Kubernetes manifest scanning
- Dockerfile security validation
- GitHub Actions workflow scanning
- CloudFormation security analysis
- Azure ARM template scanning
- Bicep template scanning
- Terraform Plan scanning
- Policy-as-Code support
- CI/CD integration

---

# Where Checkov Fits in DevSecOps

Checkov validates infrastructure before provisioning cloud resources.

```text
Developer

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Pipeline

↓

Build Application

↓

Terraform Validation

↓

Checkov Scan

↓

Policy Evaluation

↓

Terraform Apply

↓

Infrastructure Provisioned

↓

Application Deployment
```

Infrastructure should never be provisioned before passing security validation.

---

# What Can Checkov Scan?

| Resource | Supported |
|-----------|-----------|
| Terraform | ✓ |
| Terraform Plan | ✓ |
| Kubernetes YAML | ✓ |
| Helm Charts | ✓ |
| Dockerfile | ✓ |
| GitHub Actions | ✓ |
| GitLab CI | ✓ |
| Azure ARM | ✓ |
| Azure Bicep | ✓ |
| CloudFormation | ✓ |
| Serverless Framework | ✓ |
| Kubernetes Policies | ✓ |

---

# Enterprise Architecture

```text
                  Developers
                       │
                       ▼
                 Git Repository
                       │
                       ▼
            Jenkins / GitHub Actions
                       │
                       ▼
              Infrastructure Code
                       │
                       ▼
                  Checkov Scan
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
     Terraform     Kubernetes    Dockerfile
          │            │            │
          └────────────┼────────────┘
                       ▼
               Security Policies
                       │
                  PASS / FAIL
```

---

# Production Architecture

```text
Developer

↓

GitHub Enterprise

↓

Jenkins

↓

Terraform Repository

↓

Checkov

↓

Policy Validation

↓

Terraform Apply

↓

AWS Infrastructure

↓

Amazon EKS
```

---

# Prerequisites

| Component | Version |
|------------|----------|
| Ubuntu | 22.04 LTS |
| Python | 3.10+ |
| pip | Latest |
| Git | Latest |
| Docker | Latest |
| Terraform | Latest |
| Kubernetes CLI | Latest |

---

# Installation Methods

Checkov can be installed using:

- Python (pip)
- Docker
- GitHub Actions
- Pre-commit Hook
- CI/CD Pipelines

---

# Install Using pip

Install Checkov.

```bash
pip install checkov
```

Verify installation.

```bash
checkov --version
```

Example output.

```text
Checkov 3.x.x
```

---

# Install Using Docker

Pull the official image.

```bash
docker pull bridgecrew/checkov:latest
```

Run a scan.

```bash
docker run --rm \
-v $(pwd):/tf \
bridgecrew/checkov \
-d /tf
```

---

# Install on Ubuntu

Install Python.

```bash
sudo apt update

sudo apt install python3-pip -y
```

Install Checkov.

```bash
pip3 install checkov
```

Verify.

```bash
checkov --version
```

---

# Install on Jenkins Agent

SSH into the build agent.

```bash
ssh jenkins@agent01
```

Install.

```bash
pip3 install checkov
```

Verify.

```bash
checkov --version
```

---

# Install on GitHub Actions Runner

GitHub-hosted runners can install Checkov during workflow execution.

Example.

```yaml
- name: Install Checkov

  run: |

    pip install checkov
```

Self-hosted runners should have Checkov pre-installed and updated regularly.

---

# Verify Installation

Run a basic scan against the current Terraform project.

```bash
checkov -d .
```

Example output.

```text
Terraform Files

↓

Policies Evaluated

↓

Passed Checks

↓

Failed Checks

↓

Summary Report
```

---

# First Terraform Scan

Scan an Infrastructure as Code project.

```bash
checkov \
-d .
```

Example output.

```text
Terraform Files : 18

Checks Passed : 124

Checks Failed : 3

Skipped : 2
```

Each failed check includes:

- Check ID
- Severity
- Resource
- File name
- Line number
- Remediation guidance

This allows developers to fix infrastructure security issues before any resources are created in the cloud.