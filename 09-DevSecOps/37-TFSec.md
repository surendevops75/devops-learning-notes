# TFSec

## Introduction

TFSec is an Infrastructure as Code (IaC) security scanner specifically designed for Terraform. It analyzes Terraform configurations to identify security misconfigurations before infrastructure is provisioned.

Originally developed by Aqua Security, TFSec is widely used in DevSecOps pipelines to enforce Infrastructure as Code security and cloud security best practices.

Although many of its capabilities are now integrated into Trivy, TFSec is still commonly found in existing enterprise environments and legacy CI/CD pipelines.

---

# Why Companies Use TFSec

Terraform automates cloud infrastructure, but a single insecure configuration can expose production environments.

TFSec detects these issues during development and CI/CD, allowing teams to fix them before deployment.

## Benefits

- Terraform security scanning
- AWS security policy validation
- Azure security policy validation
- Google Cloud security validation
- Kubernetes Terraform module scanning
- Fast static analysis
- CI/CD integration
- Shift-left security
- Compliance validation
- Developer-friendly reports

---

# TFSec vs Checkov

| Feature | TFSec | Checkov |
|----------|--------|----------|
| Terraform | ✓ | ✓ |
| Terraform Plan | ✗ | ✓ |
| Kubernetes YAML | ✗ | ✓ |
| Dockerfile | ✗ | ✓ |
| GitHub Actions | ✗ | ✓ |
| CloudFormation | ✗ | ✓ |
| Azure ARM | ✗ | ✓ |
| Helm Charts | ✗ | ✓ |
| Speed | Very Fast | Fast |
| Primary Focus | Terraform | Multi-IaC |

TFSec specializes in Terraform, whereas Checkov supports multiple Infrastructure as Code frameworks.

---

# Where TFSec Fits in DevSecOps

TFSec validates Terraform configurations before infrastructure provisioning.

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

Terraform Validate

↓

TFSec Scan

↓

Security Validation

↓

Terraform Apply

↓

AWS Infrastructure

↓

Application Deployment
```

Terraform should only be applied after passing security validation.

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
             Terraform Source
                     │
                     ▼
                TFSec Scan
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        AWS       Azure       Google Cloud
                     │
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

Terraform Validate

↓

TFSec

↓

Security Approval

↓

Terraform Apply

↓

Amazon Web Services

↓

Amazon EKS
```

---

# Prerequisites

| Component | Version |
|------------|----------|
| Ubuntu | 22.04 LTS |
| Terraform | Latest |
| Git | Latest |
| Docker | Latest |
| Jenkins | Latest |
| GitHub Actions | Supported |

---

# Installation Methods

TFSec can be installed using:

- Binary
- Homebrew
- Chocolatey
- Docker
- GitHub Actions
- CI/CD Pipelines

---

# Install on Ubuntu

Download the latest binary.

```bash
curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash
```

Verify installation.

```bash
tfsec --version
```

Example output.

```text
v1.x.x
```

---

# Install Using Docker

Pull the official image.

```bash
docker pull aquasec/tfsec
```

Run a scan.

```bash
docker run --rm \
-v $(pwd):/src \
aquasec/tfsec \
/src
```

---

# Install on Jenkins Agent

Connect to the Jenkins build agent.

```bash
ssh jenkins@agent01
```

Install TFSec.

```bash
curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash
```

Verify.

```bash
tfsec --version
```

---

# Install on GitHub Actions Runner

GitHub-hosted runners can install TFSec during workflow execution.

Example.

```yaml
- name: Install TFSec

  run: |

    curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash
```

Self-hosted runners should have TFSec pre-installed and updated regularly.

---

# Verify Installation

Run a scan against the current Terraform project.

```bash
tfsec .
```

Example output.

```text
Terraform Files

↓

Security Checks

↓

Passed

↓

Failed

↓

Summary
```

---

# First Terraform Scan

Project structure.

```text
terraform/

├── provider.tf

├── main.tf

├── variables.tf

├── outputs.tf

└── terraform.tfvars
```

Run the scan.

```bash
tfsec terraform/
```

Example output.

```text
Results

Passed : 96

Failed : 4

Warnings : 2
```

Each finding includes:

- Rule ID
- Severity
- File name
- Resource
- Line number
- Remediation guidance

This allows engineers to remediate infrastructure security issues before Terraform provisions any cloud resources.

---

