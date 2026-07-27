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

# Repository Management

Repositories are the foundation of Artifactory.

Every artifact is uploaded, stored, and downloaded through a repository.

```text
Application

↓

Build

↓

Repository

↓

Deployment
```

Repositories should be designed based on package type and environment.

---

# Repository Naming Convention

A common enterprise naming convention.

```text
<package>-<environment>-<type>
```

Examples.

```text
docker-dev-local

docker-prod-local

maven-release-local

helm-stage-local

terraform-prod-local

npm-dev-local
```

Using a consistent naming standard simplifies automation and administration.

---

# Local Repository Configuration

Local repositories store artifacts built within the organization.

Navigate to:

```text
Administration

↓

Repositories

↓

Local

↓

Create Repository
```

Example.

```text
Repository Name

docker-dev-local
```

Package Type.

```text
Docker
```

Repository Layout.

```text
Simple Default
```

---

# Local Repository Workflow

```text
Developer

↓

Build

↓

Docker Image

↓

Push

↓

Local Repository

↓

Deployment
```

---

# Remote Repository Configuration

Remote repositories proxy external package repositories.

Examples:

- Docker Hub
- Maven Central
- npm Registry
- PyPI
- Helm Hub

Navigate to:

```text
Repositories

↓

Remote

↓

Create
```

Example.

```text
Repository

dockerhub-remote
```

URL.

```text
https://registry-1.docker.io/
```

Benefits.

- Local cache
- Faster downloads
- Reduced internet traffic
- Improved reliability

---

# Remote Repository Architecture

```text
Docker Client

↓

Artifactory

↓

Docker Hub

↓

Cache

↓

Future Requests

↓

Served Locally
```

---

# Virtual Repository Configuration

Virtual repositories expose multiple repositories through one endpoint.

Example.

```text
docker-all
```

Members.

```text
docker-dev-local

docker-prod-local

dockerhub-remote
```

Clients connect using only one URL.

---

# Virtual Repository Architecture

```text
Docker Client

↓

Virtual Repository

        │

 ┌──────┼────────┐

 ▼      ▼        ▼

Local Remote Local

        │

        ▼

Artifacts
```

---

# Repository Permissions

Repository permissions define who can:

- Read
- Deploy
- Delete
- Annotate
- Manage

Typical permission model.

```text
Developers

↓

Read

Deploy

No Delete
```

```text
Release Engineers

↓

Read

Deploy

Delete
```

```text
Administrators

↓

Full Access
```

---

# Repository Layouts

Repository layouts define directory structures.

Examples.

```text
Maven

groupId

artifactId

version
```

```text
Docker

repository

tag
```

```text
Helm

chart

version
```

---

# Docker Repository

Create a Docker repository.

```text
docker-dev-local
```

Docker Registry URL.

```text
artifactory.company.com/docker-dev-local
```

Push example.

```bash
docker login artifactory.company.com

docker tag payment:v1 \
artifactory.company.com/docker-dev-local/payment:v1

docker push \
artifactory.company.com/docker-dev-local/payment:v1
```

---

# Maven Repository

Example.

```text
maven-release-local
```

Upload.

```bash
mvn deploy
```

Artifacts stored.

```text
payment-service

↓

1.0.0

↓

payment-service-1.0.0.jar
```

---

# Helm Repository

Repository.

```text
helm-prod-local
```

Push chart.

```bash
helm package payment

curl \
-u user:password \
-T payment-1.0.0.tgz \
https://artifactory.company.com/artifactory/helm-prod-local/
```

---

# Terraform Module Repository

Repository.

```text
terraform-prod-local
```

Store:

- Network modules
- VPC modules
- EKS modules
- IAM modules

Workflow.

```text
Terraform Module

↓

Version

↓

Artifactory

↓

Terraform Apply
```

---

# Generic Repository

Generic repositories store files not belonging to a package manager.

Examples.

- ZIP files
- ISO images
- Binary files
- Shell scripts
- Configuration files

---

# User Management

Navigate.

```text
Administration

↓

Identity

↓

Users
```

Each employee should receive an individual account.

Avoid shared administrator accounts.

---

# Groups

Typical enterprise groups.

```text
developers

qa

release-engineers

security

platform-team

administrators
```

Assign permissions to groups instead of individual users.

---

# Role-Based Access Control (RBAC)

Recommended RBAC model.

| Role | Permissions |
|------|-------------|
| Developer | Read, Deploy |
| QA | Read |
| Release Engineer | Read, Deploy, Delete |
| Security | Read, Audit |
| Administrator | Full Control |

Least privilege should always be enforced.

---

# LDAP Integration

Large organizations integrate Artifactory with LDAP or Active Directory.

Architecture.

```text
Employee

↓

Artifactory

↓

LDAP

↓

Authentication

↓

Access Granted
```

Benefits.

- Centralized identity
- Automatic onboarding
- Automatic offboarding
- Group synchronization

---

# OpenID Connect (OIDC)

Artifactory supports authentication through enterprise identity providers.

Examples.

- Okta
- Azure AD
- Keycloak
- Google Workspace

Architecture.

```text
User

↓

Identity Provider

↓

OIDC Token

↓

Artifactory

↓

Login
```

---

# SAML Authentication

SAML is commonly used in enterprise Single Sign-On environments.

```text
User

↓

Identity Provider

↓

SAML Assertion

↓

Artifactory

↓

Authenticated
```

---

# Access Tokens

Automation should authenticate using access tokens instead of passwords.

Create token.

```text
Administration

↓

Identity

↓

Access Tokens
```

Example.

```text
Token Name

jenkins-token
```

Use in automation.

```bash
curl \
-H "Authorization: Bearer TOKEN" \
https://artifactory.company.com
```

---

# Service Accounts

CI/CD pipelines should use dedicated service accounts.

Example.

```text
jenkins-service

github-actions-service

gitlab-runner-service

argocd-service
```

Each account should have only the permissions required for its function.

---

# Backup Strategy

Enterprise backup should include:

- Repository data
- Metadata
- Configuration
- Security settings
- Access tokens
- Database

Recommended schedule.

```text
Daily Incremental

↓

Weekly Full Backup

↓

Monthly Archive
```

---

# High Availability (HA)

Enterprise deployments commonly use multiple Artifactory nodes.

Architecture.

```text
          Load Balancer

                │

      ┌─────────┴─────────┐

      ▼                   ▼

Artifactory-1      Artifactory-2

      │                   │

      └─────────┬─────────┘

                ▼

          Shared Storage

                ▼

          PostgreSQL
```

Benefits.

- High availability
- Fault tolerance
- Horizontal scaling
- Zero-downtime maintenance

---

# Storage Management

Monitor storage usage regularly.

View storage.

```text
Administration

↓

Storage

↓

Storage Summary
```

Monitor:

- Disk utilization
- Repository size
- Growth rate
- Largest repositories

---

# Cleanup Policies

Unused artifacts consume storage and increase backup time.

Automate cleanup.

Examples.

- Delete snapshots older than 30 days.
- Delete unused Docker images.
- Remove obsolete Helm charts.
- Remove expired npm packages.
- Archive inactive repositories.

---

# Repository Replication

Replication synchronizes repositories across multiple sites.

```text
Primary Site

↓

Replication

↓

Secondary Site

↓

Disaster Recovery
```

Benefits.

- Disaster recovery
- Multi-region deployments
- Reduced latency
- Business continuity

---

# Enterprise Best Practices

- Separate development, staging, and production repositories.
- Use virtual repositories for client access.
- Integrate LDAP or OIDC for centralized authentication.
- Enforce RBAC using groups.
- Use service accounts for automation.
- Store credentials as access tokens instead of passwords.
- Enable regular backups and replication.
- Monitor storage growth and apply cleanup policies.
- Protect Artifactory with HTTPS and a reverse proxy.
- Audit repository access and permission changes regularly.

---

