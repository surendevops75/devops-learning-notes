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

# Scanning Terraform Projects

TFSec performs static security analysis on Terraform code before infrastructure is provisioned.

Project structure.

```text
terraform/

├── provider.tf

├── variables.tf

├── outputs.tf

├── main.tf

├── vpc.tf

├── eks.tf

└── rds.tf
```

Run the scan.

```bash
tfsec terraform/
```

Workflow.

```text
Terraform Files

↓

Parse Resources

↓

Load Security Rules

↓

Evaluate Resources

↓

Generate Findings
```

---

# Scanning Multiple Terraform Projects

Enterprise repositories often contain multiple Terraform projects.

Example.

```text
Infrastructure

├── networking/

├── eks/

├── monitoring/

├── security/

└── databases/
```

Run.

```bash
tfsec infrastructure/
```

Each project is analyzed independently.

---

# Scanning Terraform Modules

Modules should be scanned before publishing them to a module registry.

Example.

```text
terraform

├── modules

│   ├── vpc

│   ├── eks

│   └── iam

└── environments
```

Run.

```bash
tfsec terraform/modules/
```

Workflow.

```text
Terraform Module

↓

Security Scan

↓

PASS / FAIL

↓

Publish
```

---

# Example 1 - Public S3 Bucket

## Insecure Configuration

```hcl
resource "aws_s3_bucket" "logs" {

  bucket = "company-logs"

}
```

Finding.

```text
AWS S3 Bucket

↓

Public Access Controls Missing
```

---

## Secure Configuration

```hcl
resource "aws_s3_bucket_public_access_block" "logs" {

  bucket = aws_s3_bucket.logs.id

  block_public_acls       = true

  block_public_policy     = true

  ignore_public_acls      = true

  restrict_public_buckets = true

}
```

---

# Example 2 - Unencrypted EBS Volume

## Insecure Configuration

```hcl
resource "aws_ebs_volume" "database" {

  availability_zone = "us-east-1a"

  size = 100

}
```

Finding.

```text
Encryption Disabled
```

---

## Secure Configuration

```hcl
resource "aws_ebs_volume" "database" {

  availability_zone = "us-east-1a"

  size = 100

  encrypted = true

}
```

---

# Example 3 - Security Group Allows SSH from Anywhere

## Insecure Configuration

```hcl
resource "aws_security_group" "ssh" {

  ingress {

    from_port = 22

    to_port = 22

    protocol = "tcp"

    cidr_blocks = ["0.0.0.0/0"]

  }

}
```

Finding.

```text
SSH Accessible from Internet
```

---

## Secure Configuration

```hcl
resource "aws_security_group" "ssh" {

  ingress {

    from_port = 22

    to_port = 22

    protocol = "tcp"

    cidr_blocks = ["10.0.0.0/16"]

  }

}
```

---

# Example 4 - Public RDS Instance

## Insecure Configuration

```hcl
resource "aws_db_instance" "database" {

  publicly_accessible = true

}
```

Finding.

```text
Database Publicly Accessible
```

---

## Secure Configuration

```hcl
resource "aws_db_instance" "database" {

  publicly_accessible = false

}
```

---

# Example 5 - Unencrypted S3 Bucket

## Insecure Configuration

```hcl
resource "aws_s3_bucket" "backup" {

  bucket = "company-backup"

}
```

Finding.

```text
Server Side Encryption Not Enabled
```

---

## Secure Configuration

```hcl
resource "aws_s3_bucket_server_side_encryption_configuration" "backup" {

  bucket = aws_s3_bucket.backup.id

  rule {

    apply_server_side_encryption_by_default {

      sse_algorithm = "AES256"

    }

  }

}
```

---

# Example 6 - IAM Policy with Wildcards

## Insecure Configuration

```json
{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Effect": "Allow",

      "Action": "*",

      "Resource": "*"

    }

  ]

}
```

Finding.

```text
IAM Wildcard Permissions
```

---

## Secure Configuration

```json
{

  "Version": "2012-10-17",

  "Statement": [

    {

      "Effect": "Allow",

      "Action": [

        "s3:GetObject"

      ],

      "Resource": [

        "arn:aws:s3:::company-data/*"

      ]

    }

  ]

}
```

---

# Example 7 - Unencrypted RDS Storage

## Insecure Configuration

```hcl
resource "aws_db_instance" "mysql" {

  storage_encrypted = false

}
```

Finding.

```text
Database Storage Encryption Disabled
```

---

## Secure Configuration

```hcl
resource "aws_db_instance" "mysql" {

  storage_encrypted = true

}
```

---

# Example 8 - EKS Cluster Logging Disabled

## Insecure Configuration

```hcl
resource "aws_eks_cluster" "eks" {

  name = "production"

}
```

Finding.

```text
EKS Control Plane Logging Disabled
```

---

## Secure Configuration

```hcl
resource "aws_eks_cluster" "eks" {

  enabled_cluster_log_types = [

    "api",

    "audit",

    "authenticator"

  ]

}
```

---

# Scan Workflow

```text
Terraform Project

↓

TFSec

↓

Security Rules

↓

Resource Evaluation

↓

PASS / FAIL

↓

Security Report
```

---

# Severity Workflow

```text
Critical

↓

High

↓

Medium

↓

Low

↓

Informational
```

High and Critical findings should block production deployments.

---

# Enterprise Best Practices

- Scan every Terraform project before merge.
- Validate reusable Terraform modules before publishing.
- Fix High and Critical findings immediately.
- Encrypt storage resources by default.
- Restrict network access using least privilege.
- Enable encryption for databases and storage.
- Protect cloud resources with secure IAM policies.
- Enable audit logging for managed cloud services.
- Integrate TFSec into every Infrastructure as Code pipeline.
- Store reports for compliance and security audits.

---

# Jenkins Integration

TFSec is commonly integrated into Jenkins pipelines to prevent insecure infrastructure from being provisioned.

## Enterprise Architecture

```text
Developer

↓

Git Push

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Checkout

↓

Terraform Format

↓

Terraform Validate

↓

TFSec Scan

↓

PASS / FAIL

↓

Terraform Plan

↓

Terraform Apply

↓

AWS Infrastructure
```

Only validated Terraform code should be deployed.

---

# Jenkins Prerequisites

Install TFSec on the Jenkins agent.

```bash
tfsec --version
```

Install Terraform.

```bash
terraform version
```

Store Jenkins credentials.

```text
AWS Credentials

Terraform Backend Credentials

GitHub Access Token
```

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/company/terraform.git'

            }

        }

        stage('Terraform Format') {

            steps {

                sh 'terraform fmt -check'

            }

        }

        stage('Terraform Validate') {

            steps {

                sh 'terraform init'

                sh 'terraform validate'

            }

        }

        stage('TFSec Scan') {

            steps {

                sh 'tfsec terraform/'

            }

        }

        stage('Terraform Plan') {

            steps {

                sh 'terraform plan'

            }

        }

    }

}
```

---

# GitHub Actions Integration

Store repository secrets.

```text
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY
```

Workflow.

```text
GitHub

↓

Workflow Trigger

↓

Checkout

↓

Terraform Validate

↓

TFSec

↓

PASS / FAIL
```

---

# Production GitHub Actions Workflow

```yaml
name: Terraform Security

on:

  pull_request:

  push:

    branches:

      - main

jobs:

  tfsec:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - name: Install TFSec

        run: |

          curl -s https://raw.githubusercontent.com/aquasecurity/tfsec/master/scripts/install_linux.sh | bash

      - name: Terraform Init

        run: terraform init

      - name: Terraform Validate

        run: terraform validate

      - name: Run TFSec

        run: tfsec terraform/
```

---

# GitLab CI Integration

```yaml
stages:

  - security

tfsec:

  stage: security

  image: aquasec/tfsec

  script:

    - tfsec terraform/

  allow_failure: false
```

---

# Pull Request Validation

Every Pull Request should trigger a Terraform security scan.

```text
Developer

↓

Pull Request

↓

Terraform Validation

↓

TFSec

↓

PASS / FAIL

↓

Merge
```

Infrastructure should never be merged without passing security validation.

---

# Monorepository Scanning

Many enterprise repositories contain multiple Terraform projects.

Example.

```text
Repository

├── networking/

├── eks/

├── databases/

├── monitoring/

└── security/
```

Run.

```bash
tfsec .
```

Or scan an individual project.

```bash
tfsec networking/
```

---

# Scanning Selected Directories

Scan multiple directories.

```bash
tfsec terraform/networking

tfsec terraform/eks

tfsec terraform/database
```

Useful when different teams manage separate infrastructure components.

---

# Ignoring Directories

Exclude downloaded providers and temporary files.

```bash
tfsec \
--exclude-path .terraform .
```

Multiple exclusions.

```bash
tfsec \
--exclude-path .terraform \
--exclude-path vendor .
```

---

# Custom Checks

Organizations can extend TFSec using custom Rego policies.

Example structure.

```text
custom-policies/

├── s3.rego

├── iam.rego

└── eks.rego
```

Run.

```bash
tfsec \
--rego-policy-dir custom-policies \
terraform/
```

Custom policies help enforce internal security standards.

---

# Compliance Validation

TFSec helps organizations align Terraform configurations with security best practices.

Examples.

- CIS Benchmarks
- PCI DSS
- SOC 2
- HIPAA
- ISO 27001
- NIST

Workflow.

```text
Terraform

↓

TFSec

↓

Compliance Rules

↓

Compliance Report
```

---

# Exit Codes

Exit codes allow CI/CD pipelines to determine success or failure.

| Exit Code | Meaning |
|-----------|---------|
| 0 | Scan Passed |
| 1 | Security Findings Detected |
| >1 | Execution Error |

Production pipelines should fail when security findings are detected.

---

# Reporting

Generate JSON.

```bash
tfsec \
--format json .
```

Generate SARIF.

```bash
tfsec \
--format sarif .
```

Generate JUnit.

```bash
tfsec \
--format junit .
```

Reports should be archived as CI/CD artifacts.

---

# Report Workflow

```text
Terraform Code

↓

TFSec

↓

JSON

↓

SARIF

↓

JUnit

↓

Security Dashboard
```

---

# GitHub Code Scanning Integration

Generate a SARIF report.

```bash
tfsec \
--format sarif \
--out tfsec-results.sarif .
```

Workflow.

```text
TFSec

↓

SARIF Report

↓

GitHub Code Scanning

↓

Security Alerts

↓

Developer
```

Developers can review findings directly in pull requests.

---

# Enterprise DevSecOps Pipeline

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

CI Trigger

↓

Checkout

↓

Terraform Format

↓

Terraform Validate

↓

TFSec Scan

↓

Policy Validation

↓

Terraform Plan

↓

Terraform Apply

↓

AWS Infrastructure

↓

Deploy Applications
```

---

# Enterprise Best Practices

- Run TFSec on every pull request.
- Scan all Terraform modules before publishing.
- Fail production pipelines on High and Critical findings.
- Integrate TFSec with Jenkins, GitHub Actions, and GitLab CI.
- Generate SARIF reports for GitHub Security.
- Archive reports for compliance audits.
- Keep custom security policies under version control.
- Exclude only trusted directories from scanning.
- Regularly update TFSec to use the latest security rules.
- Combine TFSec with Terraform validation, Checkov, Trivy, and cloud security reviews for comprehensive Infrastructure as Code security.

---

