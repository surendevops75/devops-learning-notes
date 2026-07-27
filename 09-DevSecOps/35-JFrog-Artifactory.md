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

# Maven Repository Integration

Artifactory acts as the enterprise Maven repository for storing and distributing Java artifacts.

## Architecture

```text
Developer

↓

Maven Build

↓

mvn deploy

↓

Artifactory

↓

Developers

↓

CI/CD

↓

Production
```

---

# Configure Maven

Configure `settings.xml`.

```xml
<servers>

    <server>

        <id>artifactory</id>

        <username>build-user</username>

        <password>ACCESS_TOKEN</password>

    </server>

</servers>
```

Configure `pom.xml`.

```xml
<distributionManagement>

    <repository>

        <id>artifactory</id>

        <url>

https://artifactory.company.com/artifactory/maven-release-local

        </url>

    </repository>

</distributionManagement>
```

Deploy artifact.

```bash
mvn clean deploy
```

---

# Gradle Repository Integration

Example configuration.

```groovy
repositories {

    maven {

        url "https://artifactory.company.com/artifactory/maven-release-local"

        credentials {

            username = "build-user"

            password = System.getenv("ARTIFACTORY_TOKEN")

        }

    }

}
```

Publish.

```bash
gradle publish
```

---

# npm Repository Integration

Configure npm.

```bash
npm config set registry \
https://artifactory.company.com/artifactory/api/npm/npm-local/
```

Authenticate.

```bash
npm login
```

Publish.

```bash
npm publish
```

Workflow.

```text
Node Application

↓

npm publish

↓

Artifactory

↓

Development Teams
```

---

# Docker Registry Integration

Artifactory can function as a private Docker Registry.

Login.

```bash
docker login artifactory.company.com
```

Tag image.

```bash
docker tag payment:v1 \
artifactory.company.com/docker-dev-local/payment:v1
```

Push.

```bash
docker push \
artifactory.company.com/docker-dev-local/payment:v1
```

Pull.

```bash
docker pull \
artifactory.company.com/docker-dev-local/payment:v1
```

---

# Docker Repository Architecture

```text
Docker Build

↓

Tag Image

↓

Push

↓

Artifactory

↓

Pull

↓

Kubernetes
```

---

# Helm Repository Integration

Package chart.

```bash
helm package payment
```

Upload.

```bash
curl \
-u build-user:TOKEN \
-T payment-1.0.0.tgz \
https://artifactory.company.com/artifactory/helm-prod-local/
```

Add repository.

```bash
helm repo add company \
https://artifactory.company.com/artifactory/helm-prod-local
```

Deploy.

```bash
helm install payment company/payment
```

---

# Terraform Module Repository

Artifactory supports version-controlled Terraform modules.

Example.

```text
terraform-network

↓

Version 1.0.0

↓

Artifactory

↓

Terraform Apply
```

Terraform configuration.

```hcl
module "vpc" {

  source = "artifactory.company.com/network/aws"

  version = "1.0.0"

}
```

---

# Generic Repository Usage

Store deployment assets such as:

- SQL scripts
- Shell scripts
- Configuration files
- Certificates
- Binary packages
- Release documentation

Example upload.

```bash
curl \
-u build-user:TOKEN \
-T install.sh \
https://artifactory.company.com/artifactory/generic-local/
```

---

# Build Information

Artifactory stores build metadata for every CI/CD execution.

Typical metadata includes:

- Build number
- Build timestamp
- Git commit
- Branch
- Build URL
- Environment variables
- Published artifacts
- Dependencies

Architecture.

```text
Build

↓

Artifacts

↓

Metadata

↓

Artifactory

↓

Audit
```

Build information improves traceability and simplifies troubleshooting.

---

# Artifact Promotion

Enterprise deployments promote the same immutable artifact through multiple environments.

```text
Build

↓

Development

↓

QA

↓

Staging

↓

Production
```

No rebuilds occur between environments.

---

# Promotion Workflow

```text
Application

↓

Build

↓

Security Validation

↓

Artifactory

↓

Promote

↓

Production Repository

↓

Deploy
```

This ensures consistency across all environments.

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

Build

↓

Security Scans

↓

Publish Artifact

↓

Artifactory

↓

Deployment
```

Store credentials using Jenkins Credentials.

```text
ARTIFACTORY_URL

ARTIFACTORY_USER

ARTIFACTORY_TOKEN
```

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    environment {

        ARTIFACTORY_URL = 'https://artifactory.company.com'

        ARTIFACTORY_USER = credentials('artifactory-user')

        ARTIFACTORY_TOKEN = credentials('artifactory-token')

    }

    stages {

        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/company/payment-service.git'

            }

        }

        stage('Build') {

            steps {

                sh 'mvn clean package'

            }

        }

        stage('Upload Artifact') {

            steps {

                sh '''

                curl \
                -u $ARTIFACTORY_USER:$ARTIFACTORY_TOKEN \
                -T target/payment-service.jar \
                $ARTIFACTORY_URL/artifactory/maven-release-local/

                '''

            }

        }

    }

}
```

---

# GitHub Actions Integration

Store repository secrets.

```text
ARTIFACTORY_URL

ARTIFACTORY_USERNAME

ARTIFACTORY_TOKEN
```

---

# Production GitHub Actions Workflow

```yaml
name: Publish Artifact

on:

  push:

    branches:

      - main

jobs:

  publish:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - name: Build

        run: mvn clean package

      - name: Upload Artifact

        env:

          URL: ${{ secrets.ARTIFACTORY_URL }}

          USERNAME: ${{ secrets.ARTIFACTORY_USERNAME }}

          TOKEN: ${{ secrets.ARTIFACTORY_TOKEN }}

        run: |

          curl \
          -u $USERNAME:$TOKEN \
          -T target/payment-service.jar \
          $URL/artifactory/maven-release-local/
```

---

# GitLab CI Integration

```yaml
publish_artifact:

  image: eclipse-temurin:17

  script:

    - mvn clean package

    - curl \
      -u $ARTIFACTORY_USER:$ARTIFACTORY_TOKEN \
      -T target/payment-service.jar \
      $ARTIFACTORY_URL/artifactory/maven-release-local/

  variables:

    ARTIFACTORY_URL: https://artifactory.company.com
```

---

# Artifactory REST API

Artifactory provides REST APIs for automation.

Retrieve repository information.

```bash
curl \
-u build-user:TOKEN \
https://artifactory.company.com/artifactory/api/repositories
```

Upload an artifact.

```bash
curl \
-u build-user:TOKEN \
-T app.jar \
https://artifactory.company.com/artifactory/libs-release-local/
```

Download an artifact.

```bash
curl \
-u build-user:TOKEN \
-O \
https://artifactory.company.com/artifactory/libs-release-local/app.jar
```

---

# Integrating with Amazon ECR

For containerized workloads, Artifactory can serve as the internal artifact repository before publishing approved images to Amazon ECR.

```text
Docker Build

↓

Trivy Scan

↓

Cosign Sign

↓

Artifactory

↓

Promote

↓

Amazon ECR

↓

Amazon EKS
```

This approach ensures that only validated and approved images are promoted to the container registry used for deployments.

---

# Enterprise Best Practices

- Store only immutable release artifacts.
- Promote artifacts instead of rebuilding them.
- Use access tokens instead of passwords.
- Enable build-info publishing for traceability.
- Separate repositories by package type and environment.
- Automate uploads through CI/CD pipelines.
- Scan artifacts before publishing them.
- Apply retention policies to remove obsolete artifacts.
- Enable audit logging for all repository operations.
- Protect all repository endpoints with HTTPS.

----

# Enterprise DevSecOps Pipeline

Artifactory serves as the trusted artifact repository in an enterprise DevSecOps pipeline.

Only artifacts that successfully pass quality and security validation should be stored and promoted.

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
               Checkout Source Code
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
          OWASP Dependency-Check
                         │
                         ▼
              Veracode Pipeline Scan
                         │
                         ▼
               Build Docker Image
                         │
                         ▼
                Trivy Image Scan
                         │
                         ▼
                 Generate SBOM
                         │
                         ▼
              Cosign Image Signing
                         │
                         ▼
             Publish to Artifactory
                         │
                         ▼
            Artifact Promotion
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

# Docker Image Lifecycle

Docker images progress through multiple stages before reaching production.

```text
Dockerfile

↓

Docker Build

↓

Security Scan

↓

Image Signing

↓

Artifactory

↓

Promotion

↓

Amazon ECR

↓

Kubernetes
```

Images should never be rebuilt after security approval.

---

# Helm Chart Lifecycle

Helm charts should also be versioned and promoted.

```text
Helm Chart

↓

Package

↓

Artifactory

↓

Promotion

↓

ArgoCD

↓

Amazon EKS
```

Example.

```bash
helm package payment

helm push payment-1.0.0.tgz \
oci://artifactory.company.com/helm-prod-local
```

---

# Repository Promotion Strategy

Instead of rebuilding software for each environment, the same artifact is promoted.

```text
Development Repository

↓

QA Repository

↓

Staging Repository

↓

Production Repository
```

Benefits:

- Immutable releases
- Consistent deployments
- Faster rollbacks
- Auditability

---

# Environment Promotion Flow

```text
Build

↓

docker-dev-local

↓

QA Approval

↓

docker-qa-local

↓

Business Approval

↓

docker-stage-local

↓

Production Approval

↓

docker-prod-local
```

Every promotion should be recorded for auditing purposes.

---

# Immutable Artifacts

An artifact should never be modified after publication.

```text
Artifact v1.0

↓

Published

↓

Read Only

↓

Promoted

↓

Production
```

If changes are required, create a new version instead of overwriting an existing one.

---

# Build Information

Artifactory stores metadata for every build.

Example.

```text
Build Number

↓

Git Commit

↓

Branch

↓

Build Time

↓

Artifact Version

↓

Stored Together
```

Benefits.

- Full traceability
- Easy rollback
- Release auditing
- Compliance reporting

---

# Artifact Signing

Artifacts should be digitally signed before publication.

Example workflow.

```text
Application

↓

Build

↓

Cosign

↓

Digital Signature

↓

Artifactory
```

Verification.

```text
Deployment

↓

Verify Signature

↓

Trusted

↓

Deploy
```

Unsigned artifacts should not be promoted to production repositories.

---

# JFrog Xray Integration

JFrog Xray continuously scans artifacts stored in Artifactory for vulnerabilities and license issues.

Architecture.

```text
Artifact

↓

Artifactory

↓

JFrog Xray

↓

Security Analysis

↓

Policy Evaluation

↓

PASS / FAIL
```

Typical checks.

- CVEs
- License violations
- Malware
- Security policies
- Dependency analysis

---

# Release Workflow

```text
Developer

↓

Build

↓

Security Validation

↓

Publish

↓

Artifactory

↓

Approval

↓

Promotion

↓

Deploy
```

Only approved releases should be promoted beyond the development repository.

---

# Disaster Recovery

Enterprise environments should replicate Artifactory repositories to a disaster recovery site.

```text
Primary Artifactory

↓

Replication

↓

Secondary Artifactory

↓

Disaster Recovery
```

Regularly test failover procedures to ensure business continuity.

---

# Monitoring

Monitor the following metrics.

- Repository growth
- Storage utilization
- Upload failures
- Download failures
- Authentication failures
- Replication status
- Node health
- API response times

Example workflow.

```text
Artifactory

↓

Prometheus

↓

Grafana

↓

Alerts
```

---

# Common Mistakes

## Mistake 1

Publishing artifacts before security validation.

Correct workflow.

```text
Build

↓

Security Scan

↓

Publish

↓

Deploy
```

---

## Mistake 2

Using mutable tags.

Avoid.

```text
latest
```

Prefer.

```text
payment:v1.0.0

payment:v1.0.1
```

Immutable version tags simplify rollbacks and auditing.

---

## Mistake 3

Using administrator credentials in CI/CD.

Correct approach.

```text
Jenkins

↓

Service Account

↓

Access Token

↓

Artifactory
```

---

## Mistake 4

Mixing development and production repositories.

Correct approach.

```text
Development

↓

QA

↓

Stage

↓

Production
```

Separate repositories reduce deployment risk.

---

## Mistake 5

Deleting release artifacts.

Release artifacts should remain available for:

- Rollbacks
- Audits
- Compliance
- Disaster recovery

---

# Common Troubleshooting

## Issue 1

### Upload Failed

**Cause**

Invalid credentials or insufficient permissions.

**Resolution**

```text
Verify Access Token

↓

Check Repository Permissions

↓

Retry Upload
```

---

## Issue 2

### Repository Out of Space

**Cause**

Storage capacity exhausted.

**Resolution**

- Expand storage.
- Apply cleanup policies.
- Archive old artifacts.
- Remove obsolete snapshots.

---

## Issue 3

### Docker Push Denied

**Cause**

Incorrect repository URL or authentication failure.

**Resolution**

```text
Verify Docker Login

↓

Verify Repository Name

↓

Retry Push
```

---

## Issue 4

### Replication Failure

**Cause**

Network connectivity or repository configuration issues.

**Resolution**

```text
Verify Network

↓

Check Replication Settings

↓

Restart Synchronization
```

---

## Issue 5

### Slow Artifact Downloads

**Cause**

Network latency or overloaded storage.

**Resolution**

- Enable remote caching.
- Verify storage performance.
- Scale Artifactory nodes.
- Use geographically distributed replication.

---

# Production Interview Questions

## Question 1

### What is JFrog Artifactory?

**Answer**

JFrog Artifactory is an enterprise artifact repository manager used to securely store, manage, version, and distribute software artifacts such as Docker images, Maven packages, Helm charts, Terraform modules, and other build outputs.

---

## Question 2

### What is the difference between Local, Remote, and Virtual repositories?

**Answer**

- **Local Repository** stores internally built artifacts.
- **Remote Repository** caches external package repositories.
- **Virtual Repository** provides a single endpoint that aggregates multiple repositories.

---

## Question 3

### Why should artifacts be immutable?

**Answer**

Immutable artifacts ensure that the exact binary tested in development is the same binary deployed to production, improving consistency, traceability, and rollback capabilities.

---

## Question 4

### Why is Artifactory placed after security scanning?

**Answer**

Only artifacts that have passed quality checks, vulnerability scans, and policy validation should be stored and promoted. This prevents insecure artifacts from entering deployment pipelines.

---

## Question 5

### What is artifact promotion?

**Answer**

Artifact promotion moves the same validated artifact between environments (Development, QA, Staging, Production) without rebuilding it.

---

## Question 6

### Why use access tokens instead of passwords?

**Answer**

Access tokens are easier to rotate, can be scoped to specific permissions, reduce credential exposure, and are better suited for automation.

---

## Question 7

### How does Artifactory integrate with CI/CD?

**Answer**

CI/CD pipelines publish build artifacts to Artifactory after successful builds and security scans. Deployment pipelines retrieve approved artifacts from Artifactory instead of rebuilding them.

---

## Question 8

### What is JFrog Xray?

**Answer**

JFrog Xray is a security scanning solution that integrates with Artifactory to identify vulnerabilities, license issues, and policy violations in stored artifacts.

---

## Question 9

### Why separate repositories for Development and Production?

**Answer**

Separate repositories prevent accidental deployments, simplify access control, and support controlled promotion of approved artifacts through the software delivery lifecycle.

---

## Question 10

### What are the enterprise best practices for Artifactory?

**Answer**

Use immutable artifacts, separate repositories by environment, secure automation with service accounts and access tokens, enable replication and backups, monitor repository health, integrate with security scanners, and promote artifacts instead of rebuilding them.

---

# Key Takeaways

- JFrog Artifactory is the enterprise source of truth for software artifacts.
- Store only validated and security-approved artifacts.
- Promote immutable artifacts through Development, QA, Staging, and Production.
- Integrate Artifactory with Jenkins, GitHub Actions, GitLab CI, and deployment pipelines.
- Secure automation using service accounts and access tokens.
- Monitor storage, replication, and repository health continuously.
- Integrate JFrog Xray for continuous vulnerability and license scanning.
- Combine Artifactory with SonarQube, OWASP Dependency-Check, Veracode, Trivy, SBOM generation, and Cosign to build a secure software supply chain.