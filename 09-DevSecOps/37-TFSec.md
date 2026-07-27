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

# Configuration

TFSec supports multiple configuration methods to standardize security scanning across enterprise environments.

Configuration can be managed using:

- Command-line arguments
- Configuration files
- Environment variables
- CI/CD pipeline configuration

Using a centralized configuration ensures consistent security policy enforcement across all Terraform projects.

---

# Configuration Priority

When multiple configuration methods are used, TFSec follows this order.

```text
Command Line Options

↓

Environment Variables

↓

.tfsec.yml

↓

Default Settings
```

---

# TFSec Configuration File

Create a configuration file.

```bash
vi .tfsec.yml
```

Example.

```yaml
minimum_severity: HIGH

exclude:

  - AWS006

  - AWS017

exclude_paths:

  - .terraform/

  - modules/

format: default
```

Run.

```bash
tfsec .
```

---

# Common Configuration Options

| Option | Purpose |
|----------|----------|
| minimum_severity | Report findings above selected severity |
| exclude | Skip specific rules |
| exclude_paths | Ignore directories |
| format | Output report format |
| soft_fail | Continue pipeline |
| verbose | Detailed output |

---

# Severity Levels

TFSec categorizes findings based on risk.

```text
Critical

↓

High

↓

Medium

↓

Low
```

Example.

```bash
tfsec \
--minimum-severity HIGH .
```

Only High and Critical findings will be reported.

---

# Excluding Rules

Sometimes a rule may not apply to a specific project.

Example.

```bash
tfsec \
--exclude AWS017 .
```

Exclude multiple rules.

```bash
tfsec \
--exclude AWS017 \
--exclude AWS006 .
```

Every excluded rule should be reviewed and approved by the security team.

---

# Inline Ignore

A specific resource can ignore a rule using an inline comment.

Example.

```hcl
#tfsec:ignore:AWS017

resource "aws_security_group" "example" {

  name = "demo"

}
```

Inline ignores should include a documented business justification.

---

# Excluding Directories

Terraform downloads provider modules into directories that usually do not require scanning.

Example.

```bash
tfsec \
--exclude-path .terraform .
```

Multiple directories.

```bash
tfsec \
--exclude-path .terraform \
--exclude-path vendor .
```

---

# Output Formats

TFSec supports multiple report formats.

| Format | Purpose |
|----------|----------|
| Default | Console |
| JSON | Automation |
| CSV | Reporting |
| JUnit | CI/CD |
| SARIF | GitHub Security |
| Checkstyle | Static Analysis |

Example.

```bash
tfsec \
--format json .
```

Generate SARIF.

```bash
tfsec \
--format sarif .
```

Generate multiple reports.

```bash
tfsec \
--format json \
--out reports/tfsec .
```

---

# Report Directory

Generate reports in a dedicated directory.

```bash
mkdir reports
```

Example.

```bash
tfsec \
--format json \
--out reports/tfsec .
```

Generated files.

```text
reports/

├── tfsec.json

├── tfsec.sarif

└── tfsec.junit
```

---

# Soft Fail

Soft Fail allows the pipeline to continue even when security findings are detected.

Example.

```bash
tfsec \
--soft-fail .
```

Workflow.

```text
Terraform

↓

TFSec

↓

Findings

↓

Report

↓

Pipeline Continues
```

Recommended only for development or proof-of-concept environments.

---

# Hard Fail

Production pipelines should stop when High or Critical issues are detected.

```bash
tfsec .
```

Workflow.

```text
Terraform

↓

TFSec

↓

Policy Violation

↓

Pipeline Stops
```

---

# Verbose Output

Enable detailed logging.

```bash
tfsec \
--verbose .
```

Useful for troubleshooting scans and understanding policy evaluations.

---

# Environment Variables

Common environment variables.

```bash
export TFSEC_LOG_LEVEL=debug

export TFSEC_CONFIG_FILE=.tfsec.yml
```

Verify.

```bash
env | grep TFSEC
```

---

# Scanning Specific Directories

Run TFSec against a Terraform project.

```bash
tfsec terraform/
```

Scan another project.

```bash
tfsec infrastructure/
```

This is useful when multiple Terraform projects exist within a monorepository.

---

# Scanning a Single Module

Project structure.

```text
terraform/

├── networking/

├── eks/

├── iam/

└── rds/
```

Scan only the EKS module.

```bash
tfsec terraform/eks
```

---

# Configuration Workflow

```text
Terraform Project

↓

Read Configuration

↓

Load Rules

↓

Apply Exclusions

↓

Evaluate Resources

↓

Generate Report
```

---

# Enterprise Best Practices

- Store `.tfsec.yml` in version control.
- Standardize configuration across all Terraform repositories.
- Fail production pipelines on High and Critical findings.
- Use inline ignores only with documented approval.
- Exclude only trusted directories such as `.terraform`.
- Generate SARIF reports for GitHub Security integration.
- Archive JSON reports for compliance audits.
- Keep TFSec updated with the latest security rules.
- Review excluded rules during security audits.
- Use consistent severity thresholds across all CI/CD pipelines.

---

