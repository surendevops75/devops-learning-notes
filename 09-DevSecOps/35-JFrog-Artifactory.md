# JFrog Artifactory

## Introduction

JFrog Artifactory is an enterprise artifact repository manager that securely stores, manages, versions, and distributes software artifacts throughout the Software Development Lifecycle (SDLC).

It acts as the central repository for build artifacts, Docker images, Helm charts, Terraform modules, Maven packages, npm packages, and many other package formats.

Artifactory ensures that only trusted, versioned, and approved artifacts are promoted through development, testing, staging, and production environments.

---

# Why Companies Use JFrog Artifactory

Modern software development generates thousands of build artifacts every day.

Without a centralized repository, managing versions, security, and deployments becomes difficult.

Artifactory provides a single source of truth for all software artifacts.

## Benefits

- Centralized artifact storage
- Artifact versioning
- Build promotion
- Secure artifact distribution
- High availability
- Access control
- CI/CD integration
- Replication across regions
- Metadata management
- Software supply chain security

---

# What is an Artifact Repository?

An artifact repository stores software packages generated during the build process.

Examples include:

- Java JAR files
- WAR files
- Docker images
- Helm charts
- Terraform modules
- npm packages
- Python packages
- Generic binaries

Instead of rebuilding software every time, deployment pipelines retrieve trusted artifacts directly from Artifactory.

---

# Where Artifactory Fits in DevSecOps

Artifactory stores only artifacts that have successfully passed security and quality validation.

```text
Developer

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Build

↓

Unit Tests

↓

SonarQube

↓

OWASP Dependency-Check

↓

Veracode

↓

Docker Build

↓

Trivy Scan

↓

SBOM Generation

↓

Cosign Image Signing

↓

JFrog Artifactory

↓

Amazon ECR

↓

GitOps

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

Artifactory becomes the trusted artifact repository before deployment.

---

# Why Store Artifacts Instead of Source Code?

Source code represents the application.

Artifacts represent the deployable software.

```text
Source Code

↓

Build

↓

Artifact

↓

Store in Artifactory

↓

Deploy Multiple Times
```

Advantages:

- Faster deployments
- Immutable releases
- Rollback support
- Version consistency
- Compliance tracking

---

# Enterprise Architecture

```text
                  Developers
                       │
                       ▼
               GitHub / GitLab
                       │
                       ▼
          Jenkins / GitHub Actions
                       │
                       ▼
                Build Application
                       │
                       ▼
              Security Validation
                       │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
 SonarQube      Veracode         Trivy Scan
      │               │                │
      └───────────────┼────────────────┘
                      ▼
                Build Artifact
                      │
                      ▼
             JFrog Artifactory
                      │
        ┌─────────────┼─────────────┐
        ▼             ▼             ▼
   Development      Staging     Production
                      │
                      ▼
                 Deployments
```

---

# Production Architecture

```text
                    Developers

                         │

                         ▼

                  GitHub Enterprise

                         │

                         ▼

                     Jenkins

                         │

                         ▼

                  Build Agents

                         │

                         ▼

              Security Scanning

                         │

                         ▼

                JFrog Artifactory

              ┌────────┼────────┐

              ▼        ▼        ▼

          Maven     Docker     Helm

              │        │        │

              └────────┼────────┘

                       ▼

                  Amazon ECR

                       ▼

                    ArgoCD

                       ▼

                  Amazon EKS
```

---

# Repository Types

Artifactory provides three repository types.

| Repository | Purpose |
|------------|----------|
| Local | Store internally built artifacts |
| Remote | Cache external repositories |
| Virtual | Unified access to multiple repositories |

---

# Local Repository

A Local Repository stores artifacts produced within the organization.

Examples:

- Company Docker images
- Internal Maven packages
- Private Helm charts
- Terraform modules

```text
CI Pipeline

↓

Build Artifact

↓

Local Repository

↓

Deployment
```

---

# Remote Repository

A Remote Repository proxies public repositories.

Examples:

- Docker Hub
- Maven Central
- npm Registry
- PyPI
- Helm Hub

Benefits:

- Local caching
- Faster downloads
- Reduced internet dependency
- Improved availability

---

# Virtual Repository

A Virtual Repository combines multiple repositories into one logical endpoint.

```text
Virtual Repository

        │

 ┌──────┼──────┐

 ▼      ▼      ▼

Local Remote Local

        │

        ▼

Single URL
```

Applications interact with one repository instead of multiple endpoints.

---

# Supported Package Formats

Artifactory supports numerous package ecosystems.

| Package Type | Supported |
|--------------|-----------|
| Maven | ✓ |
| Gradle | ✓ |
| Docker | ✓ |
| Helm | ✓ |
| npm | ✓ |
| NuGet | ✓ |
| PyPI | ✓ |
| Go | ✓ |
| Conan | ✓ |
| Terraform Modules | ✓ |
| Generic Files | ✓ |

---

# Prerequisites

Before installing Artifactory, ensure the following components are available.

| Component | Version |
|------------|----------|
| Ubuntu | 22.04 LTS |
| Docker | Latest |
| Docker Compose | Latest |
| PostgreSQL | Supported Version |
| Nginx | Latest |
| Java | Bundled |
| DNS | Recommended |
| SSL Certificate | Production |

---

# Installation Methods

Artifactory can be deployed using:

- Linux Installation
- Docker
- Docker Compose
- Kubernetes
- OpenShift
- Helm Chart

Production deployments commonly use Docker Compose or Kubernetes with an external PostgreSQL database.

---

# Install on Ubuntu (Docker)

Create directories.

```bash
sudo mkdir -p /opt/artifactory

cd /opt/artifactory
```

Create persistent storage.

```bash
mkdir data

mkdir logs

mkdir backup
```

---

# Create Docker Compose File

```yaml
version: "3.8"

services:

  artifactory:

    image: releases-docker.jfrog.io/jfrog/artifactory-oss:latest

    container_name: artifactory

    ports:

      - "8082:8082"

      - "8081:8081"

    volumes:

      - ./data:/var/opt/jfrog/artifactory

    restart: unless-stopped
```

Start Artifactory.

```bash
docker compose up -d
```

Verify.

```bash
docker ps
```

---

# Access Artifactory

Open the browser.

```text
http://SERVER-IP:8082
```

Example.

```text
http://192.168.1.100:8082
```

The initial setup wizard will appear after startup.

---

# Install on Kubernetes

Deploy using the official Helm chart.

```bash
helm repo add jfrog https://charts.jfrog.io

helm repo update
```

Install.

```bash
helm install artifactory jfrog/artifactory
```

Verify.

```bash
kubectl get pods
```

Expected output.

```text
artifactory-0      Running
```

---

# Initial Login

Default administrator account.

```text
Username

admin
```

```text
Password

password
```

**Important:** Change the default administrator password immediately after the first login and create separate administrator accounts for operational use.

---

# Verify Installation

Check the system status.

```text
Administration

↓

System

↓

System Health
```

Verify:

- Repository service
- Database connection
- Storage
- License status
- Node health

All services should report a healthy status before onboarding CI/CD pipelines.

---

# Create Your First Repository

Navigate to:

```text
Administration

↓

Repositories

↓

Create Repository
```

Choose a repository type.

Example.

```text
Local Repository

↓

Docker

↓

docker-dev-local
```

Artifactory is now ready to receive build artifacts from enterprise CI/CD pipelines.

---

