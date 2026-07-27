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

---

# Scanning Terraform

Terraform is the most common Infrastructure as Code framework scanned by Checkov.

Example project.

```text
terraform/

├── main.tf

├── variables.tf

├── outputs.tf

├── provider.tf

└── terraform.tfvars
```

Run the scan.

```bash
checkov \
-d terraform/
```

Workflow.

```text
Terraform Files

↓

Parse Resources

↓

Evaluate Policies

↓

PASS / FAIL
```

---

# Example Terraform Finding

Example insecure S3 bucket.

```hcl
resource "aws_s3_bucket" "logs" {

  bucket = "company-logs"

}
```

Checkov output.

```text
FAILED

CKV_AWS_21

Ensure S3 Bucket Versioning is Enabled
```

Recommended fix.

```hcl
resource "aws_s3_bucket_versioning" "logs" {

  bucket = aws_s3_bucket.logs.id

  versioning_configuration {

    status = "Enabled"

  }

}
```

---

# Scanning Terraform Plan

Scanning a Terraform Plan provides more accurate results because variables have already been evaluated.

Generate the plan.

```bash
terraform plan \
-out=tfplan.binary
```

Convert the plan.

```bash
terraform show \
-json tfplan.binary \
> tfplan.json
```

Run Checkov.

```bash
checkov \
-f tfplan.json
```

Workflow.

```text
Terraform Code

↓

Terraform Plan

↓

JSON Plan

↓

Checkov

↓

Policy Evaluation
```

---

# Scanning Kubernetes Manifests

Checkov validates Kubernetes YAML files against security best practices.

Example project.

```text
kubernetes/

├── deployment.yaml

├── service.yaml

├── ingress.yaml

└── namespace.yaml
```

Run.

```bash
checkov \
-d kubernetes/
```

---

# Example Kubernetes Finding

Example deployment.

```yaml
containers:

- name: payment

  image: payment:v1
```

Checkov output.

```text
FAILED

CKV_K8S_20

Container Should Not Run as Root
```

Recommended fix.

```yaml
securityContext:

  runAsNonRoot: true

  runAsUser: 1000
```

---

# Common Kubernetes Policies

| Policy | Description |
|----------|-------------|
| Run as Non-Root | Prevent root containers |
| Read Only Root Filesystem | Prevent filesystem modifications |
| Resource Limits | Prevent resource exhaustion |
| Image Pull Policy | Use secure image pull behavior |
| Privileged Containers | Prevent privileged execution |
| Host Network | Restrict host networking |
| Capabilities | Drop unnecessary Linux capabilities |

---

# Scanning Helm Charts

Helm charts are rendered before policy evaluation.

Project.

```text
helm/

├── Chart.yaml

├── values.yaml

├── templates/
```

Run.

```bash
checkov \
-d helm/
```

Workflow.

```text
Helm Chart

↓

Render Templates

↓

Generated YAML

↓

Policy Scan
```

---

# Scanning Dockerfiles

Checkov identifies Dockerfile security issues.

Example.

```Dockerfile
FROM ubuntu:latest

USER root

RUN apt update
```

Run.

```bash
checkov \
-d .
```

Possible findings.

```text
Latest Tag Used

Running as Root

Missing HEALTHCHECK

Large Base Image
```

Recommended Dockerfile.

```Dockerfile
FROM eclipse-temurin:21-jre

RUN adduser \
--system appuser

USER appuser

HEALTHCHECK CMD curl \
-f http://localhost:8080 || exit 1
```

---

# Scanning GitHub Actions

Checkov validates GitHub Actions workflows for security risks.

Project.

```text
.github/

└── workflows/

    └── ci.yml
```

Run.

```bash
checkov \
-d .
```

Common findings.

- Unpinned GitHub Actions
- Insecure secrets usage
- Overly permissive permissions
- Untrusted third-party actions

---

# Scanning GitLab CI

Checkov also scans GitLab CI configuration.

Project.

```text
.gitlab-ci.yml
```

Run.

```bash
checkov \
-d .
```

Checks include:

- Protected variables
- Image security
- Pipeline permissions
- Secret handling

---

# Scanning CloudFormation

CloudFormation templates are fully supported.

Example.

```bash
checkov \
-d cloudformation/
```

Example resources.

```text
VPC

EC2

S3

IAM

Lambda

CloudFront
```

---

# Scanning Azure ARM Templates

Run.

```bash
checkov \
-d arm/
```

Supported resources include:

- Virtual Machines
- Storage Accounts
- Key Vault
- Networking
- AKS
- SQL Database

---

# Scanning Azure Bicep

Run.

```bash
checkov \
-d bicep/
```

Workflow.

```text
Bicep

↓

Compile

↓

Policy Scan

↓

Results
```

---

# Scanning Multiple Frameworks

Enterprise repositories often contain multiple Infrastructure as Code technologies.

Example.

```bash
checkov \
-d . \
--framework \
terraform,kubernetes,dockerfile,github_actions
```

Workflow.

```text
Repository

├── Terraform

├── Kubernetes

├── Dockerfile

├── GitHub Actions

└── Helm

        │

        ▼

     Checkov

        │

        ▼

 Security Report
```

---

# Enterprise Best Practices

- Scan every Infrastructure as Code change before merging.
- Scan Terraform Plans for production deployments.
- Validate Kubernetes manifests before applying them.
- Scan Helm charts after rendering.
- Use secure Docker base images.
- Pin GitHub Actions to specific versions.
- Protect GitLab pipelines with secure variables.
- Scan all supported frameworks within monorepositories.
- Fix High and Critical findings before deployment.
- Integrate scans into every pull request and CI/CD pipeline.

---

# Jenkins Integration

Checkov is commonly integrated into Jenkins pipelines to prevent insecure infrastructure from being provisioned.

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

Terraform Validate

↓

Checkov Scan

↓

PASS / FAIL

↓

Terraform Apply

↓

AWS
```

Only validated Infrastructure as Code should proceed to deployment.

---

# Jenkins Prerequisites

Install on Jenkins agent.

```bash
pip3 install checkov
```

Verify.

```bash
checkov --version
```

Store Jenkins credentials.

```text
AWS Credentials

Terraform Backend Credentials

Bridgecrew API Key (Optional)
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

        stage('Terraform Validate') {

            steps {

                sh 'terraform init'

                sh 'terraform validate'

            }

        }

        stage('Checkov Scan') {

            steps {

                sh '''

                checkov \
                -d . \
                --framework terraform

                '''

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

Store secrets.

```text
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

BC_API_KEY (Optional)
```

Workflow.

```text
GitHub

↓

Workflow Trigger

↓

Checkout

↓

Install Checkov

↓

Scan

↓

PASS / FAIL
```

---

# Production GitHub Actions Workflow

```yaml
name: Checkov Scan

on:

  pull_request:

  push:

    branches:

      - main

jobs:

  checkov:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - name: Install Checkov

        run: pip install checkov

      - name: Run Checkov

        run: |

          checkov \
          -d . \
          --framework terraform
```

---

# GitLab CI Integration

Example.

```yaml
stages:

  - security

checkov:

  stage: security

  image: python:3.12

  before_script:

    - pip install checkov

  script:

    - checkov -d .

  allow_failure: false
```

---

# Pull Request Validation

Every Pull Request should trigger Checkov automatically.

```text
Developer

↓

Pull Request

↓

Checkov

↓

Security Report

↓

PASS / FAIL

↓

Merge
```

Infrastructure code should never be merged without security validation.

---

# Scanning Monorepositories

Enterprise repositories often contain multiple applications and infrastructure directories.

Example.

```text
Repository

├── terraform/

├── kubernetes/

├── docker/

├── helm/

├── applications/

└── github-actions/
```

Run.

```bash
checkov \
-d .
```

Or target only Terraform.

```bash
checkov \
-d terraform/
```

---

# Scanning Specific Files

Scan a single file.

```bash
checkov \
-f main.tf
```

Scan multiple files.

```bash
checkov \
-f main.tf \
-f eks.tf
```

---

# Scanning Multiple Directories

```bash
checkov \
-d terraform/

checkov \
-d kubernetes/
```

Useful for repositories with separate infrastructure projects.

---

# Ignoring Directories

Skip unnecessary directories.

Example.

```bash
checkov \
-d . \
--skip-path .terraform
```

Multiple directories.

```bash
checkov \
-d . \
--skip-path .terraform \
--skip-path vendor
```

---

# Custom Policies

Organizations can write custom policies for internal security requirements.

Example structure.

```text
custom-policies/

├── s3_policy.yaml

├── iam_policy.yaml

└── eks_policy.yaml
```

Run.

```bash
checkov \
-d . \
--external-checks-dir custom-policies/
```

---

# Compliance Frameworks

Checkov includes policies aligned with common compliance standards.

Examples.

- CIS Benchmark
- AWS Well-Architected Framework
- PCI DSS
- SOC 2
- HIPAA
- NIST
- ISO 27001

Workflow.

```text
Infrastructure

↓

Checkov

↓

Compliance Policies

↓

Compliance Report
```

---

# Exit Codes

Exit codes help CI/CD pipelines determine success or failure.

| Exit Code | Meaning |
|-----------|---------|
| 0 | Scan Passed |
| 1 | Policy Violations Found |
| >1 | Execution Error |

Pipelines should stop when policy violations are detected.

---

# Reporting

Generate JSON.

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

Generate JUnit XML.

```bash
checkov \
-d . \
-o junitxml
```

Reports can be archived as CI/CD artifacts for auditing and compliance.

---

# Report Workflow

```text
Infrastructure Code

↓

Checkov

↓

JSON

↓

SARIF

↓

JUnit XML

↓

Security Dashboard
```

---

# Integrating with GitHub Code Scanning

Generate a SARIF report.

```bash
checkov \
-d . \
-o sarif
```

Upload the SARIF report using GitHub Actions.

```text
Checkov

↓

SARIF Report

↓

GitHub Security

↓

Code Scanning Alerts
```

Developers can review findings directly within pull requests.

---

# Enterprise Pipeline

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

Terraform Validate

↓

Checkov Scan

↓

Policy Validation

↓

Terraform Plan

↓

Terraform Apply

↓

Infrastructure Created

↓

Application Deployment
```

---

# Enterprise Best Practices

- Run Checkov on every pull request.
- Scan Terraform Plans before production deployments.
- Fail production pipelines on High and Critical findings.
- Keep custom security policies in version control.
- Scan monorepositories at the repository root.
- Generate SARIF reports for GitHub Security.
- Archive scan reports for compliance audits.
- Keep Checkov updated with the latest policy definitions.
- Review skipped checks periodically with the security team.
- Combine Checkov with Terraform validation and security scanning for layered protection.

---

