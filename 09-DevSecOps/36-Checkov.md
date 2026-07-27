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

---

# Configuration

Checkov supports multiple configuration methods for enterprise deployments.

Configuration can be managed using:

- Command-line arguments
- Configuration files
- Environment variables
- CI/CD pipeline configuration
- Bridgecrew Platform integration

Using a centralized configuration ensures consistent policy enforcement across all repositories.

---

# Configuration Priority

When multiple configuration methods are used, Checkov follows this order.

```text
Command Line Arguments

↓

Environment Variables

↓

.checkov.yaml

↓

Default Settings
```

---

# Checkov Configuration File

Create a configuration file.

```bash
vi .checkov.yaml
```

Example.

```yaml
framework:

  - terraform

  - kubernetes

  - dockerfile

download-external-modules: true

quiet: false

compact: false

output:

  - cli

  - json

soft-fail: false
```

Run.

```bash
checkov -d .
```

---

# Common Configuration Options

| Option | Purpose |
|----------|----------|
| framework | Scan specific resource types |
| skip-check | Skip specific policies |
| check | Run selected policies |
| output | Report format |
| quiet | Reduce console output |
| soft-fail | Continue pipeline on failures |
| compact | Compact report output |

---

# Framework Selection

Instead of scanning everything, organizations often scan only the required frameworks.

Example.

```bash
checkov \
-d . \
--framework terraform
```

Multiple frameworks.

```bash
checkov \
-d . \
--framework terraform,kubernetes,dockerfile
```

---

# Selecting Specific Policies

Run only selected security checks.

Example.

```bash
checkov \
-d . \
--check CKV_AWS_20
```

Multiple checks.

```bash
checkov \
-d . \
--check CKV_AWS_20,CKV_AWS_21
```

Useful for validating compliance requirements.

---

# Skipping Policies

Sometimes a policy may not apply to a particular environment.

Example.

```bash
checkov \
-d . \
--skip-check CKV_AWS_20
```

Multiple checks.

```bash
checkov \
-d . \
--skip-check CKV_AWS_20,CKV_K8S_13
```

Skipped checks should always be documented and approved by the security team.

---

# Skip Check Annotation

Checks can also be skipped inside Infrastructure as Code files.

Terraform example.

```hcl
#checkov:skip=CKV_AWS_20:Public access required for testing

resource "aws_s3_bucket" "demo" {

  bucket = "company-demo"

}
```

Use inline skips only after security review.

---

# Severity Filtering

Organizations often prioritize High and Critical findings.

Example.

```bash
checkov \
-d . \
--check MEDIUM,HIGH,CRITICAL
```

Typical workflow.

```text
Critical

↓

High

↓

Medium

↓

Low

↓

Passed
```

---

# Output Formats

Checkov supports multiple report formats.

| Format | Purpose |
|---------|----------|
| CLI | Console output |
| JSON | Automation |
| SARIF | GitHub Security |
| JUnit XML | CI/CD |
| CycloneDX | SBOM |
| GitLab SAST | GitLab Security |

Example.

```bash
checkov \
-d . \
-o json
```

Generate SARIF.

```bash
checkov \
-d . \
-o sarif
```

Generate multiple reports.

```bash
checkov \
-d . \
-o cli \
-o json \
-o sarif
```

---

# Report Directory

Specify the output directory.

```bash
checkov \
-d . \
--output-file-path reports/
```

Example.

```text
reports/

├── checkov.json

├── checkov.sarif

├── checkov.xml
```

---

# Environment Variables

Common environment variables.

```bash
export CHECKOV_EXPERIMENTAL=true

export LOG_LEVEL=INFO
```

View.

```bash
env | grep CHECKOV
```

---

# External Module Download

Many Terraform projects use remote modules.

Enable module downloads.

```bash
checkov \
-d . \
--download-external-modules true
```

Workflow.

```text
Terraform

↓

External Modules

↓

Download

↓

Policy Scan

↓

Results
```

---

# Variable Files

Terraform variable files can be supplied during scanning.

Example.

```bash
checkov \
-d . \
--var-file production.tfvars
```

This improves policy accuracy.

---

# Soft Fail

Soft Fail allows the pipeline to continue even when findings exist.

Example.

```bash
checkov \
-d . \
--soft-fail
```

Workflow.

```text
Scan

↓

Findings

↓

Report Generated

↓

Pipeline Continues
```

Recommended only for development environments.

---

# Hard Fail

Production pipelines should stop when policy violations are detected.

```bash
checkov \
-d . \
```

Workflow.

```text
Scan

↓

Policy Failure

↓

Pipeline Stops
```

---

# Bridgecrew Platform Integration

Organizations using the Bridgecrew platform can upload scan results centrally.

```bash
checkov \
-d . \
--bc-api-key YOUR_API_KEY
```

Benefits.

- Centralized dashboards
- Compliance reporting
- Policy management
- Risk tracking

---

# Policy Management

Policies can be:

- Built-in
- Custom
- Organization-specific

Architecture.

```text
Terraform

↓

Checkov

↓

Security Policies

↓

PASS / FAIL
```

---

# Enterprise Best Practices

- Store configuration in version control.
- Scan only required frameworks.
- Review skipped policies regularly.
- Use SARIF reports for GitHub integration.
- Fail production builds on policy violations.
- Use Soft Fail only in development.
- Scan external Terraform modules.
- Store reports for auditing.
- Keep Checkov updated to the latest stable release.
- Standardize configuration across all repositories.