# Trivy

## Introduction

Trivy is an open-source security scanner developed by Aqua Security that helps identify vulnerabilities, misconfigurations, exposed secrets, licenses, and supply chain risks across container images, filesystems, Git repositories, Infrastructure as Code (IaC), Kubernetes clusters, and Software Bill of Materials (SBOMs).

Unlike traditional vulnerability scanners, Trivy is lightweight, fast, and designed for DevSecOps pipelines, making it one of the most widely adopted security scanners in cloud-native environments.

---

# Why Companies Use Trivy

Modern applications consist of multiple components:

- Application source code
- Base container images
- Operating system packages
- Third-party libraries
- Kubernetes manifests
- Terraform infrastructure
- Helm charts

Any of these components can introduce vulnerabilities.

Trivy enables organizations to detect security issues before software reaches production.

### Benefits

- Detect OS vulnerabilities
- Detect application dependency vulnerabilities
- Scan Docker images
- Scan Kubernetes clusters
- Scan Terraform & Kubernetes YAML
- Generate SBOMs
- Detect exposed secrets
- Scan licenses
- Integrate easily with CI/CD pipelines

---

# Where Trivy Fits in DevSecOps

Trivy operates after the application has been built but before artifacts are published or deployed.

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

Unit Tests

↓

SonarQube Analysis

↓

Quality Gate

↓

Build Docker Image

↓

Trivy Scan

↓

Generate SBOM

↓

Image Signing

↓

Push to Registry

↓

GitOps

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

If Trivy detects HIGH or CRITICAL vulnerabilities, the pipeline should fail immediately.

---

# What Can Trivy Scan?

Trivy supports multiple scan targets.

| Scan Type | Description |
|-----------|-------------|
| Container Image | Scan Docker/OCI images |
| Filesystem | Scan local directories |
| Git Repository | Scan source repositories |
| Kubernetes | Scan cluster resources |
| Terraform | Scan Infrastructure as Code |
| Kubernetes YAML | Scan manifests before deployment |
| Helm Charts | Scan Helm templates |
| VM Filesystem | Scan virtual machines |
| SBOM | Generate and analyze SBOM |
| Secret Scan | Detect exposed secrets |
| License Scan | Identify software licenses |

---

# Enterprise Architecture

```text
                   Developer
                        │
                        ▼
                GitHub Repository
                        │
                        ▼
             Jenkins / GitHub Actions
                        │
                        ▼
                  Build Docker Image
                        │
                        ▼
                      Trivy
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
 Image Scan      Secret Scan     IaC Scan
        │              │              │
        └──────────────┼──────────────┘
                       ▼
                 Generate SBOM
                       │
                       ▼
                 Cosign Sign Image
                       │
                       ▼
                Amazon ECR / ACR
                       │
                       ▼
                     ArgoCD
                       │
                       ▼
                  Amazon EKS
```

---

# Production Architecture

```text
Developer

↓

GitHub Enterprise

↓

Jenkins Controller

↓

Jenkins Agent

↓

Docker Build

↓

Trivy

↓

SBOM

↓

Cosign

↓

Amazon ECR

↓

ArgoCD

↓

Amazon EKS
```

---

# Prerequisites

Before installing Trivy, verify the following.

| Component | Version |
|-----------|----------|
| Ubuntu | 22.04 LTS |
| Docker | Latest |
| Git | Latest |
| Kubernetes | 1.28+ Recommended |
| Helm | Optional |
| Jenkins | Recommended |
| GitHub Actions | Supported |

---

# Supported Operating Systems

Trivy supports:

- Linux
- Windows
- macOS
- Docker
- Kubernetes
- GitHub Actions
- GitLab CI
- Azure DevOps

---

# Installation Methods

Production environments generally use one of the following deployment models.

## Option 1 — Native Installation

```text
Ubuntu Server

↓

Install Trivy

↓

Update Database

↓

Ready to Scan
```

Recommended for Jenkins agents and Linux servers.

---

## Option 2 — Docker

```text
Docker Host

↓

Run Trivy Container

↓

Scan Images
```

Suitable for containerized CI/CD environments.

---

## Option 3 — Kubernetes

```text
Amazon EKS

↓

Trivy Operator

↓

Cluster Security Reports
```

Recommended for production Kubernetes clusters.

---

# Install Trivy on Ubuntu

## Step 1 — Update Packages

```bash
sudo apt update

sudo apt upgrade -y
```

---

## Step 2 — Install Required Packages

```bash
sudo apt install wget apt-transport-https gnupg lsb-release -y
```

---

## Step 3 — Add Aqua Security Repository

```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
```

Add repository.

```bash
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/trivy.list
```

---

## Step 4 — Install Trivy

```bash
sudo apt update

sudo apt install trivy -y
```

---

## Step 5 — Verify Installation

```bash
trivy --version
```

Expected output

```text
Version: x.x.x
```

---

# Install Using Docker

Pull the official image.

```bash
docker pull aquasec/trivy:latest
```

Verify.

```bash
docker images
```

Run.

```bash
docker run --rm aquasec/trivy image nginx:latest
```

---

# Install on Jenkins Agent

SSH into the Jenkins build agent.

```bash
ssh jenkins@agent01
```

Install using the Ubuntu steps above.

Verify.

```bash
trivy --version
```

The Jenkins pipeline can now execute Trivy scans locally on the build agent.

---

# Install on GitHub Actions Runner

GitHub-hosted runners can install Trivy dynamically.

Example.

```yaml
- name: Install Trivy
  run: |
    sudo apt-get update
    sudo apt-get install -y trivy
```

For self-hosted runners, install Trivy once and keep it updated.

---

# Update the Vulnerability Database

Trivy downloads a vulnerability database before performing scans.

Update manually.

```bash
trivy image --download-db-only
```

Production recommendation:

Schedule a daily update using cron.

Example.

```text
0 2 * * * trivy image --download-db-only
```

This keeps vulnerability definitions current without delaying CI/CD pipelines.

---

# Verify Database

```bash
trivy image alpine:latest
```

The first scan downloads and validates the database if required.

---

# First Image Scan

Scan the latest NGINX image.

```bash
trivy image nginx:latest
```

Example output.

```text
Target: nginx:latest

Total: 3 (HIGH:2, CRITICAL:1)
```

---

# Scan Local Filesystem

```bash
trivy fs .
```

This scans the current project directory for:

- Vulnerabilities
- Secrets
- Misconfigurations

---

# Scan a Git Repository

```bash
trivy repo https://github.com/company/payment-service
```

Useful for scanning repositories before cloning into CI/CD environments.

---

# Trivy Configuration

Trivy can be configured using:

- Command-line arguments
- Environment variables
- Configuration file (`trivy.yaml`)

Using a configuration file provides consistency across local development, CI/CD pipelines, and production environments.

---

# Configuration Priority

When multiple configuration methods are used, Trivy follows this order.

```text
Command Line Arguments

↓

Environment Variables

↓

trivy.yaml

↓

Default Values
```

---

# Configuration File

Create a configuration file.

```bash
mkdir -p ~/.trivy

vi ~/.trivy/trivy.yaml
```

Example configuration.

```yaml
cache:
  dir: ~/.cache/trivy

db:
  auto-update: true

scan:
  security-checks:
    - vuln
    - secret
    - config

severity:
  - HIGH
  - CRITICAL

exit-code: 1

format: table

ignore-unfixed: false
```

---

# Common Configuration Options

| Setting | Purpose |
|----------|----------|
| cache.dir | Location of Trivy cache |
| db.auto-update | Automatically update vulnerability database |
| security-checks | Enable vulnerability, secret, config scanning |
| severity | Vulnerability levels to report |
| exit-code | Pipeline exit status |
| format | Output format |
| ignore-unfixed | Ignore vulnerabilities without fixes |

---

# Registry Authentication

Private container registries require authentication before Trivy can pull and scan images.

Supported registries include:

- Amazon ECR
- Azure Container Registry (ACR)
- Docker Hub
- Google Artifact Registry (GAR)
- GitHub Container Registry (GHCR)
- Harbor
- JFrog Artifactory

---

# Amazon ECR Authentication

Authenticate using AWS CLI.

```bash
aws ecr get-login-password \
| docker login \
--username AWS \
--password-stdin \
123456789012.dkr.ecr.us-east-1.amazonaws.com
```

Scan an image.

```bash
trivy image \
123456789012.dkr.ecr.us-east-1.amazonaws.com/payment-service:1.0
```

---

# Azure Container Registry (ACR)

Login.

```bash
az acr login --name companyregistry
```

Scan.

```bash
trivy image companyregistry.azurecr.io/payment-service:v1
```

---

# Docker Hub Authentication

```bash
docker login
```

Scan.

```bash
trivy image company/payment-service:latest
```

---

# GitHub Container Registry (GHCR)

Authenticate.

```bash
echo $GITHUB_TOKEN | docker login ghcr.io \
-u USERNAME \
--password-stdin
```

Scan.

```bash
trivy image ghcr.io/company/payment-service:latest
```

---

# Authentication Flow

```text
Trivy

↓

Registry Authentication

↓

Download Image

↓

Extract Layers

↓

Analyze Packages

↓

Generate Report
```

---

# Using Environment Variables

Credentials should never be hardcoded.

Example.

```bash
export AWS_ACCESS_KEY_ID=xxxxxxxx

export AWS_SECRET_ACCESS_KEY=xxxxxxxx

export AWS_DEFAULT_REGION=us-east-1
```

CI/CD platforms should inject credentials securely.

Examples:

- Jenkins Credentials
- GitHub Secrets
- GitLab CI Variables
- AWS Secrets Manager
- Azure Key Vault
- HashiCorp Vault

---

# Proxy Configuration

Many enterprise environments use outbound proxy servers.

Example.

```bash
export HTTP_PROXY=http://proxy.company.com:8080

export HTTPS_PROXY=http://proxy.company.com:8080

export NO_PROXY=localhost,127.0.0.1
```

Verify.

```bash
env | grep PROXY
```

---

# Cache Management

Trivy caches:

- Vulnerability Database
- Java Database
- Scan Results

Default location.

```text
~/.cache/trivy
```

Clear cache.

```bash
trivy clean --all
```

View cache size.

```bash
du -sh ~/.cache/trivy
```

---

# Updating Cache

Download the latest database.

```bash
trivy image --download-db-only
```

Download Java database.

```bash
trivy image --download-java-db-only
```

Production recommendation:

Schedule automatic database updates during non-business hours to reduce CI/CD execution time.

---

# Ignore File

Some vulnerabilities may be accepted temporarily.

Create an ignore file.

```bash
vi .trivyignore
```

Example.

```text
CVE-2024-12345

CVE-2024-56789
```

Run.

```bash
trivy image \
--ignorefile .trivyignore \
payment-service:latest
```

Use ignore files only after formal security approval.

---

# Severity Filtering

Scan only HIGH and CRITICAL vulnerabilities.

```bash
trivy image \
--severity HIGH,CRITICAL \
payment-service:latest
```

Include MEDIUM severity.

```bash
trivy image \
--severity MEDIUM,HIGH,CRITICAL \
payment-service:latest
```

---

# Exit Codes

Exit codes determine whether the CI/CD pipeline continues.

Example.

```bash
trivy image \
--exit-code 1 \
--severity HIGH,CRITICAL \
payment-service:latest
```

Behavior.

```text
No HIGH/CRITICAL

↓

Exit Code 0

↓

Pipeline Continues
```

```text
HIGH/CRITICAL Found

↓

Exit Code 1

↓

Pipeline Fails
```

---

# Secret Scanning

Trivy detects exposed secrets such as:

- AWS Access Keys
- AWS Secret Keys
- GitHub Tokens
- GitLab Tokens
- Private Keys
- API Keys
- Passwords

Run.

```bash
trivy fs \
--scanners secret .
```

Example output.

```text
HIGH

AWS Secret Key

Location

config/application.yml
```

---

# License Scanning

Organizations often restrict software licenses.

Scan licenses.

```bash
trivy fs \
--scanners license .
```

Example.

```text
MIT

Apache-2.0

GPL-3.0
```

This helps ensure compliance with organizational licensing policies.

---

# Kubernetes RBAC

When scanning Kubernetes clusters, Trivy requires appropriate permissions.

Example architecture.

```text
Trivy Operator

↓

Service Account

↓

ClusterRole

↓

ClusterRoleBinding

↓

Kubernetes API
```

Example Service Account.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: trivy
  namespace: trivy-system
```

Example ClusterRole.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: trivy-role
rules:
- apiGroups: [""]
  resources:
  - pods
  - namespaces
  - nodes
  - services
  verbs:
  - get
  - list
  - watch
```

Example ClusterRoleBinding.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: trivy-binding
subjects:
- kind: ServiceAccount
  name: trivy
  namespace: trivy-system
roleRef:
  kind: ClusterRole
  name: trivy-role
  apiGroup: rbac.authorization.k8s.io
```

---

# Enterprise Best Practices

- Use a centralized `trivy.yaml` across all projects.
- Store registry credentials in a secrets manager.
- Scan only authenticated images from trusted registries.
- Fail the pipeline for HIGH and CRITICAL vulnerabilities.
- Keep the vulnerability database updated daily.
- Review ignored CVEs periodically and remove them once fixes are available.
- Use dedicated Kubernetes service accounts with least-privilege RBAC.
- Clean the cache periodically on build agents to avoid stale data.
- Integrate secret and license scanning into every CI/CD pipeline.
- Generate machine-readable reports (SARIF, JSON) for security dashboards.

---

# Filesystem Scanning

Filesystem scanning analyzes application source code, configuration files, dependencies, secrets, licenses, and Infrastructure as Code (IaC) before building a container image.

## Architecture

```text
Developer

↓

Application Source Code

↓

Trivy Filesystem Scan

↓

Vulnerabilities

Secrets

Misconfigurations

Licenses

↓

Security Report
```

Run a filesystem scan.

```bash
trivy fs .
```

Scan a specific directory.

```bash
trivy fs /opt/projects/payment-service
```

Scan only vulnerabilities.

```bash
trivy fs \
--scanners vuln .
```

---

# Git Repository Scanning

Trivy can scan Git repositories without requiring a local clone.

```bash
trivy repo https://github.com/company/payment-service
```

Private repositories require authentication.

Example.

```bash
export GITHUB_TOKEN=<token>

trivy repo https://github.com/company/payment-service
```

Repository scanning detects:

- Vulnerable dependencies
- Secrets
- Misconfigurations
- License issues

---

# Docker Image Scanning

Container image scanning is one of Trivy's most common production use cases.

## Architecture

```text
Docker Build

↓

Docker Image

↓

Trivy

↓

Extract Layers

↓

Operating System Packages

↓

Application Dependencies

↓

Vulnerability Database

↓

Security Report
```

Scan a local image.

```bash
trivy image payment-service:latest
```

Scan a remote image.

```bash
trivy image nginx:latest
```

Scan only HIGH and CRITICAL vulnerabilities.

```bash
trivy image \
--severity HIGH,CRITICAL \
payment-service:latest
```

Ignore unfixed vulnerabilities.

```bash
trivy image \
--ignore-unfixed \
payment-service:latest
```

Exit with failure when vulnerabilities are found.

```bash
trivy image \
--exit-code 1 \
--severity HIGH,CRITICAL \
payment-service:latest
```

---

# Kubernetes Scanning

Trivy supports scanning Kubernetes clusters and Kubernetes manifests.

## Scan Kubernetes Cluster

```bash
trivy k8s cluster
```

## Scan a Namespace

```bash
trivy k8s namespace production
```

## Scan Pods

```bash
trivy k8s pod payment-service
```

---

# Kubernetes Manifest Scanning

Scan deployment manifests before deployment.

```bash
trivy config k8s/
```

Scan a single YAML file.

```bash
trivy config deployment.yaml
```

Trivy detects:

- Privileged containers
- Missing resource limits
- Containers running as root
- Missing securityContext
- Insecure capabilities
- Insecure hostPath mounts

---

# Terraform Scanning

Trivy scans Infrastructure as Code before infrastructure provisioning.

```text
Terraform Code

↓

Trivy Config Scan

↓

Security Checks

↓

Deployment
```

Scan Terraform files.

```bash
trivy config terraform/
```

Example findings.

```text
HIGH

S3 Bucket Public Access

HIGH

Security Group Open to 0.0.0.0/0

MEDIUM

Encryption Disabled
```

---

# Helm Chart Scanning

Scan Helm charts.

```bash
trivy config helm-chart/
```

This identifies insecure Kubernetes configurations before deployment.

---

# Secret Scanning Example

Example application.

```text
payment-service/

├── app.py
├── config.yml
├── secrets.env
└── Dockerfile
```

Run.

```bash
trivy fs \
--scanners secret .
```

Possible output.

```text
HIGH

AWS Secret Access Key

File

secrets.env
```

---

# Misconfiguration Scanning

Trivy detects common security misconfigurations.

Examples include:

- Public S3 buckets
- Unencrypted storage
- Privileged containers
- Containers running as root
- Missing security contexts
- Excessive Linux capabilities
- Open security groups

Example.

```bash
trivy config infrastructure/
```

---

# SBOM Generation

Software Bill of Materials (SBOM) provides an inventory of software components.

## Architecture

```text
Docker Image

↓

Trivy

↓

Package Discovery

↓

Generate SBOM

↓

Compliance

Security

Auditing
```

Generate CycloneDX SBOM.

```bash
trivy image \
--format cyclonedx \
-o sbom.json \
payment-service:latest
```

Generate SPDX SBOM.

```bash
trivy image \
--format spdx-json \
-o sbom-spdx.json \
payment-service:latest
```

Benefits:

- Supply chain visibility
- License compliance
- Dependency inventory
- Faster vulnerability response

---

# Report Formats

Trivy supports multiple output formats.

| Format | Use Case |
|---------|----------|
| Table | Human-readable reports |
| JSON | Automation |
| SARIF | GitHub Security |
| CycloneDX | SBOM |
| SPDX | SBOM |
| Template | Custom reporting |

Example JSON report.

```bash
trivy image \
--format json \
-o report.json \
payment-service:latest
```

Generate SARIF report.

```bash
trivy image \
--format sarif \
-o trivy-results.sarif \
payment-service:latest
```

---

# Jenkins Integration

## Architecture

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Build Image

↓

Trivy Scan

↓

PASS

↓

Push Image

OR

FAIL

↓

Stop Pipeline
```

Install Trivy on every Jenkins build agent.

Verify.

```bash
trivy --version
```

Store registry credentials securely using Jenkins Credentials.

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/company/payment-service.git'

            }

        }

        stage('Build Image') {

            steps {

                sh 'docker build -t payment-service:latest .'

            }

        }

        stage('Trivy Scan') {

            steps {

                sh '''
                trivy image \
                --severity HIGH,CRITICAL \
                --exit-code 1 \
                payment-service:latest
                '''
            }

        }

        stage('Push Image') {

            steps {

                sh 'docker push payment-service:latest'

            }

        }

    }

}
```

---

# GitHub Actions Integration

Store the following secrets.

```text
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

AWS_REGION
```

If using a private registry, also configure the required registry credentials.

---

# Production GitHub Actions Workflow

```yaml
name: Trivy Scan

on:

  push:

    branches:

      - main

jobs:

  security:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - name: Build Image

        run: docker build -t payment-service:latest .

      - name: Run Trivy

        uses: aquasecurity/trivy-action@0.28.0

        with:
          image-ref: payment-service:latest
          format: table
          exit-code: '1'
          severity: HIGH,CRITICAL
```

---

# GitLab CI Integration

```yaml
trivy_scan:

  image: aquasec/trivy:latest

  script:

    - trivy image \
      --severity HIGH,CRITICAL \
      --exit-code 1 \
      payment-service:latest
```

---

# Recommended Scan Order

```text
Filesystem Scan

↓

IaC Scan

↓

Secret Scan

↓

Build Docker Image

↓

Container Image Scan

↓

Generate SBOM

↓

Image Signing

↓

Push Container Registry

↓

Deploy to Kubernetes
```

---

# Enterprise DevSecOps Pipeline

The following workflow shows how Trivy is integrated into a production DevSecOps pipeline.

```text
                    Developer
                         │
                         ▼
                  Feature Branch
                         │
                         ▼
                     Git Push
                         │
                         ▼
                  Pull Request
                         │
                         ▼
                  Code Review
                         │
                         ▼
                  Merge to Main
                         │
                         ▼
              Jenkins / GitHub Actions
                         │
                         ▼
                   Checkout Code
                         │
                         ▼
                 Compile Application
                         │
                         ▼
                    Unit Testing
                         │
                         ▼
                SonarQube Analysis
                         │
                         ▼
                  Quality Gate
                         │
                         ▼
                Build Docker Image
                         │
                         ▼
                 Trivy Image Scan
                         │
                         ▼
              HIGH / CRITICAL Check
                  ┌────────┴────────┐
                  │                 │
                PASS              FAIL
                  │                 │
                  ▼                 ▼
            Generate SBOM     Stop Pipeline
                  │
                  ▼
          Cosign Image Signing
                  │
                  ▼
            Push to Amazon ECR
                  │
                  ▼
        Update GitOps Repository
                  │
                  ▼
               ArgoCD Sync
                  │
                  ▼
            Deploy to Amazon EKS
                  │
                  ▼
             Smoke Testing
                  │
                  ▼
              Production
```

---

# Report Analysis

Understanding the scan results is as important as running the scan.

Example output.

```text
Target

payment-service:1.0

Total Vulnerabilities

HIGH      : 4

CRITICAL  : 1

MEDIUM    : 12

LOW       : 18
```

Security teams should prioritize fixing:

1. Critical
2. High
3. Medium
4. Low

---

# Vulnerability Report

Typical report sections include:

```text
Package Name

Installed Version

Fixed Version

Severity

CVE ID

Description
```

Example.

| Package | Installed | Fixed | Severity |
|----------|------------|---------|----------|
| openssl | 3.0.2 | 3.0.15 | CRITICAL |
| curl | 7.81 | 7.88 | HIGH |
| glibc | 2.35 | 2.36 | HIGH |

---

# Secret Scan Report

Example.

```text
HIGH

AWS Secret Access Key

File

application.yml

Line

25
```

Immediate action:

- Remove the secret
- Rotate the credential
- Update the secret manager
- Commit the corrected code

---

# Misconfiguration Report

Example findings.

```text
Deployment

↓

Container Running as Root

↓

HIGH
```

```text
Security Group

↓

0.0.0.0/0

↓

HIGH
```

```text
S3 Bucket

↓

Public Access Enabled

↓

CRITICAL
```

---

# SBOM Report

Generated SBOM includes:

- Operating system packages
- Application libraries
- Third-party dependencies
- Package versions
- Licenses

Example.

```text
payment-service

↓

Spring Boot

↓

Log4j

↓

Jackson

↓

OpenSSL

↓

glibc
```

SBOMs are useful for:

- Compliance
- Supply chain audits
- Incident response
- Vulnerability tracking

---

# Integrating with Security Dashboards

Trivy reports can be integrated with:

- GitHub Security
- DefectDojo
- Dependency-Track
- ELK Stack
- SIEM platforms
- Custom dashboards

Typical workflow.

```text
Trivy

↓

JSON/SARIF Report

↓

Security Dashboard

↓

Security Team Review

↓

Remediation
```

---

# Enterprise Best Practices

## Image Security

- Use minimal base images such as Alpine or Distroless where appropriate.
- Remove unused packages from container images.
- Regularly rebuild images with the latest security updates.
- Scan every image before pushing to a registry.

---

## CI/CD

- Run Trivy on every Pull Request.
- Fail the pipeline for HIGH and CRITICAL vulnerabilities.
- Generate SBOMs for every production image.
- Sign container images after successful scans.

---

## Registry Security

- Scan images before pushing to the registry.
- Periodically rescan existing registry images.
- Remove vulnerable image tags.
- Enforce image signing before deployment.

---

## Kubernetes

- Scan manifests before deployment.
- Use Trivy Operator for continuous cluster scanning.
- Restrict Trivy permissions using least-privilege RBAC.
- Review Kubernetes misconfiguration reports regularly.

---

## Operations

- Update the vulnerability database daily.
- Keep Trivy updated to the latest stable version.
- Archive historical reports for auditing.
- Automate report generation for every release.

---

# Common Mistakes

## Mistake 1

Scanning only production images.

Correct approach.

```text
Every Pull Request

↓

Every Build

↓

Every Image

↓

Every Release
```

---

## Mistake 2

Ignoring HIGH vulnerabilities.

```text
HIGH Today

↓

Critical Tomorrow

↓

Production Incident
```

Always investigate HIGH findings before deployment.

---

## Mistake 3

Using outdated vulnerability databases.

```text
Old Database

↓

Missed CVEs

↓

False Sense of Security
```

Update the database daily.

---

## Mistake 4

Scanning only Docker images.

Remember that infrastructure, Kubernetes manifests, and source repositories also require scanning.

---

## Mistake 5

Ignoring secrets detected by Trivy.

Leaked credentials can lead to:

- Unauthorized access
- Data breaches
- Infrastructure compromise

Secrets should be removed immediately and rotated.

---

# Common Troubleshooting

## Issue 1

### Vulnerability Database Download Failed

**Cause**

No internet connectivity or proxy restrictions.

**Resolution**

```text
Check Network

↓

Verify Proxy

↓

Download Database

↓

Retry Scan
```

---

## Issue 2

### Private Registry Authentication Failed

**Cause**

Expired credentials or missing permissions.

**Resolution**

```text
Authenticate Registry

↓

Verify Credentials

↓

Retry Image Scan
```

---

## Issue 3

### Image Not Found

**Cause**

The image was not built or the image tag is incorrect.

**Resolution**

```text
Docker Build

↓

Verify Image Tag

↓

Run Trivy
```

---

## Issue 4

### Pipeline Failed Due to HIGH Vulnerabilities

**Cause**

Detected vulnerabilities exceeded the configured threshold.

**Resolution**

```text
Review Report

↓

Update Dependencies

↓

Rebuild Image

↓

Re-run Scan
```

---

## Issue 5

### Kubernetes Scan Permission Denied

**Cause**

Service Account lacks required RBAC permissions.

**Resolution**

```text
Verify Service Account

↓

Review ClusterRole

↓

Update ClusterRoleBinding

↓

Retry Scan
```

---

# Production Interview Questions

## Question 1

### What is Trivy and why is it used?

**Answer**

Trivy is a security scanner that detects vulnerabilities, secrets, misconfigurations, licenses, and supply chain risks across container images, filesystems, repositories, Infrastructure as Code, and Kubernetes environments.

---

## Question 2

### Where does Trivy fit into a DevSecOps pipeline?

**Answer**

Trivy runs after the Docker image is built but before the image is pushed to the container registry or deployed to Kubernetes, ensuring vulnerable artifacts are blocked early.

---

## Question 3

### What types of scans does Trivy support?

**Answer**

Trivy supports container image scanning, filesystem scanning, repository scanning, Kubernetes scanning, Infrastructure as Code scanning, secret detection, license scanning, and SBOM generation.

---

## Question 4

### Why should Trivy run before pushing images to Amazon ECR?

**Answer**

Running Trivy before pushing images prevents vulnerable images from being stored in the registry and deployed to production.

---

## Question 5

### How do you fail a Jenkins pipeline when vulnerabilities are found?

**Answer**

Use the `--exit-code 1` option with the desired severity level (for example, HIGH and CRITICAL). If matching vulnerabilities are detected, Trivy exits with a non-zero status and Jenkins marks the build as failed.

---

## Question 6

### How do you authenticate Trivy to scan private registries?

**Answer**

Authenticate using the registry's supported mechanism, such as AWS CLI for Amazon ECR, Docker login for Docker Hub, Azure CLI for Azure Container Registry, or GitHub tokens for GHCR.

---

## Question 7

### What is an SBOM and why is it important?

**Answer**

A Software Bill of Materials (SBOM) lists all software components and dependencies in an application. It improves supply chain visibility, compliance, vulnerability management, and incident response.

---

## Question 8

### How do you reduce false positives in Trivy?

**Answer**

Keep the vulnerability database updated, use `.trivyignore` only for approved exceptions, review findings regularly, and rescan after dependency updates.

---

## Question 9

### How do you scan Kubernetes manifests before deployment?

**Answer**

Use `trivy config` to analyze Kubernetes YAML files for insecure configurations before they are applied to the cluster.

---

## Question 10

### What are the key enterprise best practices for Trivy?

**Answer**

Automate scans in CI/CD, update the vulnerability database regularly, fail builds for HIGH and CRITICAL vulnerabilities, generate SBOMs, use least-privilege access, protect registry credentials, and integrate reports with security dashboards.

---

# Key Takeaways

- Trivy is an enterprise security scanner for containers, filesystems, repositories, Kubernetes, Infrastructure as Code, secrets, licenses, and SBOMs.
- Integrate Trivy into CI/CD immediately after building container images and before publishing them.
- Authenticate securely with private registries using managed credentials or secret stores.
- Scan images, source code, Kubernetes manifests, and Terraform configurations as part of every build.
- Generate SBOMs to improve software supply chain visibility and compliance.
- Configure pipelines to fail automatically when HIGH or CRITICAL vulnerabilities are detected.
- Protect Kubernetes environments using Trivy Operator and least-privilege RBAC.
- Keep vulnerability databases updated and review reports continuously to maintain a secure software delivery pipeline.