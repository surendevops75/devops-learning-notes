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

