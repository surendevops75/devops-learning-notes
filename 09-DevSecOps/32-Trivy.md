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

