# SonarQube

## Introduction

SonarQube is an enterprise Static Application Security Testing (SAST) platform used to continuously inspect source code for security vulnerabilities, bugs, code smells, duplicate code, and maintainability issues.

In production environments, SonarQube is integrated into CI/CD pipelines to enforce Quality Gates and prevent insecure or low-quality code from reaching production.

---

# Why Companies Use SonarQube

Large organizations deploy hundreds of applications every day.

Manual code reviews alone cannot identify every security issue.

SonarQube automates code quality and security validation before deployment.

Benefits include:

- Detect security vulnerabilities early
- Improve code quality
- Reduce technical debt
- Enforce coding standards
- Block insecure releases
- Generate compliance reports
- Improve developer productivity

---

# Where SonarQube Fits in DevSecOps

SonarQube is one stage of a complete DevSecOps pipeline.

```text
Developer

↓

Feature Branch

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

Code Coverage

↓

SonarQube Analysis

↓

Quality Gate

↓

Container Build

↓

Trivy Scan

↓

SBOM

↓

Cosign

↓

Amazon ECR

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

SonarQube is executed immediately after unit testing and before container image creation.

If the Quality Gate fails, the pipeline stops immediately.

---

# Enterprise Architecture

```text
                        Developers
                             │
                             ▼
                        GitHub Repository
                             │
                             ▼
                    Jenkins / GitHub Actions
                             │
                             ▼
                     Sonar Scanner (CLI/Maven)
                             │
                             ▼
                     SonarQube Server
                    ┌──────────┴──────────┐
                    ▼                     ▼
             PostgreSQL Database     Quality Profiles
                    │                     │
                    └──────────┬──────────┘
                               ▼
                         Quality Gate
                               │
                      PASS ─────┴───── FAIL
                        │               │
                        ▼               ▼
                Continue Pipeline   Stop Pipeline
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

Build Agent

↓

Sonar Scanner

↓

SonarQube Server

↓

PostgreSQL

↓

Quality Gate

↓

Docker Build

↓

Trivy Scan

↓

Amazon ECR

↓

ArgoCD

↓

Amazon EKS
```

---

# Prerequisites

Before installing SonarQube, ensure the following components are available.

| Component | Version |
|------------|----------|
| Ubuntu Server | 22.04 LTS |
| Java | OpenJDK 17 |
| PostgreSQL | 15+ |
| RAM | Minimum 4 GB (8 GB Recommended) |
| CPU | 2 vCPU Minimum |
| Docker (Optional) | Latest |
| Jenkins | Recommended |
| Git | Latest |

---

# Network Ports

| Port | Purpose |
|------|----------|
| 9000 | SonarQube Web UI |
| 5432 | PostgreSQL |
| 22 | SSH |
| 80/443 | Reverse Proxy |

---

# SonarQube Deployment Options

Production environments generally use one of the following deployment models.

## Option 1

Dedicated Virtual Machine

```text
Ubuntu Server

↓

PostgreSQL

↓

SonarQube

↓

Nginx

↓

Developers
```

Recommended for medium-sized organizations.

---

## Option 2

Docker

```text
Docker Host

↓

PostgreSQL Container

↓

SonarQube Container

↓

Reverse Proxy
```

Suitable for small and medium environments.

---

## Option 3

Kubernetes

```text
Amazon EKS

↓

SonarQube Pod

↓

Persistent Volume

↓

PostgreSQL

↓

Ingress

↓

Developers
```

Recommended for enterprise-scale deployments.

---

# Production Installation (Ubuntu)

## Step 1 — Update Server

```bash
sudo apt update

sudo apt upgrade -y
```

---

## Step 2 — Install Java

```bash
sudo apt install openjdk-17-jdk -y
```

Verify installation

```bash
java -version
```

Expected output

```text
openjdk version "17"
```

---

## Step 3 — Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib -y
```

Verify

```bash
sudo systemctl status postgresql
```

Status should be

```text
active (running)
```

---

## Step 4 — Create SonarQube Database

Login

```bash
sudo -u postgres psql
```

Create database

```sql
CREATE DATABASE sonarqube;
```

Create user

```sql
CREATE USER sonar WITH ENCRYPTED PASSWORD 'StrongPassword';
```

Grant permissions

```sql
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
```

Exit

```sql
\q
```

---

# Download SonarQube

Download the latest Community or Enterprise Edition from SonarSource.

```bash
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-<version>.zip
```

Extract

```bash
sudo unzip sonarqube-*.zip -d /opt
```

Rename

```bash
sudo mv /opt/sonarqube-* /opt/sonarqube
```

---

# Create SonarQube User

```bash
sudo groupadd sonar

sudo useradd -r -g sonar -d /opt/sonarqube sonar

sudo chown -R sonar:sonar /opt/sonarqube
```

---

# Configure Database Connection

Edit

```bash
sudo vi /opt/sonarqube/conf/sonar.properties
```

Configure

```properties
sonar.jdbc.username=sonar

sonar.jdbc.password=StrongPassword

sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonarqube
```

---

# Configure Java

```bash
sudo vi /opt/sonarqube/conf/wrapper.conf
```

Set Java Home if required.

---

# Linux Kernel Configuration

Production servers require Elasticsearch kernel settings.

Edit

```bash
sudo vi /etc/sysctl.conf
```

Add

```text
vm.max_map_count=524288

fs.file-max=131072
```

Apply

```bash
sudo sysctl -p
```

---

# Configure Limits

```bash
sudo vi /etc/security/limits.conf
```

Add

```text
sonar soft nofile 131072
sonar hard nofile 131072
sonar soft nproc 8192
sonar hard nproc 8192
```

---

# Configure SonarQube as a Service

Create a systemd service so SonarQube starts automatically after server reboot.

Create the service file.

```bash
sudo vi /etc/systemd/system/sonarqube.service
```

Add the following configuration.

```ini
[Unit]
Description=SonarQube Service
After=network.target postgresql.service

[Service]
Type=forking

User=sonar
Group=sonar

ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop

Restart=always

LimitNOFILE=131072
LimitNPROC=8192

[Install]
WantedBy=multi-user.target
```

Reload systemd.

```bash
sudo systemctl daemon-reload
```

Enable the service.

```bash
sudo systemctl enable sonarqube
```

Start SonarQube.

```bash
sudo systemctl start sonarqube
```

Verify status.

```bash
sudo systemctl status sonarqube
```

Expected output

```text
active (running)
```

---

# Configure Nginx Reverse Proxy

In production, SonarQube should not be exposed directly on port **9000**.

Recommended Architecture

```text
Internet

↓

HTTPS :443

↓

Nginx

↓

SonarQube :9000
```

Install Nginx.

```bash
sudo apt install nginx -y
```

Create configuration.

```bash
sudo vi /etc/nginx/sites-available/sonarqube
```

Example configuration.

```nginx
server {

    listen 80;

    server_name sonar.company.com;

    location / {

        proxy_pass http://127.0.0.1:9000;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    }

}
```

Enable configuration.

```bash
sudo ln -s /etc/nginx/sites-available/sonarqube /etc/nginx/sites-enabled/
```

Test configuration.

```bash
sudo nginx -t
```

Restart Nginx.

```bash
sudo systemctl restart nginx
```

---

# Enable HTTPS

For production deployments, always use HTTPS.

Example using Let's Encrypt.

```bash
sudo apt install certbot python3-certbot-nginx
```

Generate certificate.

```bash
sudo certbot --nginx
```

Now SonarQube becomes available at

```text
https://sonar.company.com
```

---

# First Login

Open

```text
https://sonar.company.com
```

Default credentials

```text
Username

admin

Password

admin
```

Immediately change the default password.

---

# Create Your First Project

Navigate to

```text
Projects

↓

Create Project
```

Example

```text
Project Name

payment-service

Project Key

payment-service
```

---

# Generate Project Token

Navigate

```text
Project

↓

Administration

↓

Security

↓

Generate Token
```

Example

```text
payment-service-token
```

Save the token securely.

It will later be used by Jenkins or GitHub Actions.

---

# Create Quality Profile

Navigate

```text
Quality Profiles
```

Select language.

Example

```text
Java
```

Copy the built-in profile.

```text
Sonar Way

↓

Copy

↓

Company Java Profile
```

Customize rules according to organizational standards.

---

# Configure Quality Gate

Navigate

```text
Quality Gates
```

Example production Quality Gate.

| Metric | Condition |
|---------|-----------|
| Bugs | 0 Blocker |
| Vulnerabilities | 0 Critical |
| Security Hotspots | Reviewed |
| Coverage | >80% |
| Duplicate Code | <3% |

Pipeline flow.

```text
Sonar Analysis

↓

Quality Gate

↓

PASS

↓

Continue Deployment

OR

FAIL

↓

Pipeline Stops
```

---

# Authentication Methods

Enterprise deployments rarely use only local users.

Supported authentication.

```text
SonarQube

↓

Local Authentication

LDAP

GitHub OAuth

GitLab OAuth

Azure AD

SAML

OIDC
```

---

# LDAP Integration

Common in enterprise environments.

```text
Developer

↓

Corporate Active Directory

↓

LDAP

↓

SonarQube
```

Example configuration.

```properties
sonar.security.realm=LDAP

ldap.url=ldap://ldap.company.com

ldap.bindDn=cn=admin,dc=company,dc=com

ldap.bindPassword=********
```

Benefits

- Centralized authentication
- No local passwords
- Automatic onboarding
- Easier user management

---

# OAuth / OIDC Authentication

Many organizations integrate SonarQube with GitHub or Azure AD.

```text
Developer

↓

GitHub Login

↓

OAuth

↓

SonarQube
```

Advantages

- Single Sign-On (SSO)
- MFA support
- Central identity management

---

# Users and Groups

Instead of assigning permissions directly to users, companies manage access through groups.

Example.

```text
Users

↓

Groups

↓

Permissions
```

Recommended groups.

```text
Administrators

Security Team

Developers

QA Team

Auditors

CI Service Accounts
```

---

# Production RBAC

Example permission model.

| Role | Responsibilities |
|------|------------------|
| System Administrator | Full administration, plugins, upgrades, users, global settings |
| Project Administrator | Manage project settings, quality profiles, branches, permissions |
| Security Team | Review vulnerabilities, hotspots, security reports |
| Developer | Execute analyses, view issues, resolve assigned findings |
| QA Team | View quality reports and coverage metrics |
| Auditor | Read-only access for compliance and audit reviews |
| CI Service Account | Execute pipeline scans using project token only |

**Best Practice:** Never use the default `admin` account in CI/CD pipelines. Create a dedicated **service account** with the minimum required permissions and use project-specific tokens stored in your CI/CD secret manager.

---

# API Tokens

Instead of username/password authentication, CI/CD pipelines should use API tokens.

Example

```text
Jenkins

↓

Credentials Manager

↓

SonarQube Token

↓

Pipeline Authentication
```

Never hardcode tokens in pipeline scripts or repositories.

Store them in:

- Jenkins Credentials
- GitHub Secrets
- GitLab CI Variables
- Azure Key Vault
- HashiCorp Vault
- AWS Secrets Manager

---

# Jenkins Integration

Most enterprise organizations integrate SonarQube with Jenkins to perform automated code quality analysis during every build.

## Architecture

```text
Developer

↓

Git Push

↓

GitHub Repository

↓

Webhook

↓

Jenkins Pipeline

↓

Build

↓

Unit Test

↓

SonarQube Analysis

↓

Quality Gate

↓

Continue Pipeline
```

---

# Install SonarQube Plugin in Jenkins

Navigate to

```text
Manage Jenkins

↓

Plugins

↓

Available Plugins

↓

Search

↓

SonarQube Scanner
```

Install

```text
SonarQube Scanner
```

Restart Jenkins.

---

# Configure SonarQube Server

Navigate

```text
Manage Jenkins

↓

System

↓

SonarQube Servers

↓

Add SonarQube
```

Example

| Field | Value |
|--------|-------|
| Name | SonarQube |
| Server URL | https://sonar.company.com |
| Server Authentication Token | Jenkins Credential |

---

# Create SonarQube Token

SonarQube

```text
Administration

↓

Security

↓

Users

↓

Tokens

↓

Generate Token
```

Example

```text
jenkins-sonarqube-token
```

Copy the token.

---

# Store Token in Jenkins

Navigate

```text
Manage Jenkins

↓

Credentials

↓

Global Credentials

↓

Add Credentials
```

Kind

```text
Secret Text
```

ID

```text
sonarqube-token
```

Paste the generated token.

Never store tokens directly inside Jenkinsfiles.

---

# Configure Sonar Scanner

Navigate

```text
Manage Jenkins

↓

Global Tool Configuration

↓

SonarQube Scanner

↓

Add Scanner
```

Example

```text
Name

SonarScanner
```

Install Automatically

✔ Enabled

---

# Project Configuration

Every application requires a Sonar configuration.

Example directory

```text
payment-service/

├── pom.xml
├── Jenkinsfile
├── sonar-project.properties
├── src/
└── target/
```

---

# sonar-project.properties

```properties
sonar.projectKey=payment-service

sonar.projectName=Payment Service

sonar.projectVersion=1.0

sonar.sources=src

sonar.java.binaries=target

sonar.sourceEncoding=UTF-8

sonar.host.url=https://sonar.company.com

sonar.token=${SONAR_TOKEN}
```

---

# Maven Integration

Run analysis

```bash
mvn clean verify sonar:sonar
```

Pipeline

```text
Checkout

↓

Build

↓

Unit Tests

↓

Maven Sonar Plugin

↓

Upload Report

↓

Quality Gate
```

---

# Gradle Integration

Example

```bash
./gradlew sonarqube
```

Example configuration

```groovy
plugins {

    id "org.sonarqube"

}
```

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    tools {

        maven 'Maven3'

    }

    environment {

        SCANNER_HOME = tool 'SonarScanner'

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

        stage('Unit Test') {

            steps {

                sh 'mvn test'

            }

        }

        stage('SonarQube Analysis') {

            steps {

                withSonarQubeEnv('SonarQube') {

                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=payment-service \
                    -Dsonar.projectName="Payment Service"
                    '''

                }

            }

        }

        stage('Quality Gate') {

            steps {

                timeout(time:5, unit:'MINUTES') {

                    waitForQualityGate abortPipeline: true

                }

            }

        }

    }

}
```

---

# How Quality Gate Works

```text
Source Code

↓

Sonar Scanner

↓

Upload Results

↓

SonarQube Server

↓

Evaluate Rules

↓

Quality Gate

↓

PASS

↓

Next Pipeline Stage

OR

FAIL

↓

Pipeline Stops
```

---

# GitHub Actions Integration

Store the following GitHub Secrets.

```text
SONAR_TOKEN

SONAR_HOST_URL
```

---

# Production GitHub Actions Workflow

```yaml
name: SonarQube Analysis

on:

  push:

    branches:

      - main

jobs:

  sonar:

    runs-on: ubuntu-latest

    steps:

    - uses: actions/checkout@v4

    - name: Setup Java

      uses: actions/setup-java@v4

      with:

        distribution: temurin

        java-version: 17

    - name: Cache Maven

      uses: actions/cache@v4

      with:

        path: ~/.m2

        key: maven

    - name: Build

      run: mvn clean verify

    - name: SonarQube Scan

      env:

        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

      run: |

        mvn sonar:sonar \
        -Dsonar.host.url=${{ secrets.SONAR_HOST_URL }} \
        -Dsonar.login=${{ secrets.SONAR_TOKEN }}
```

---

# GitLab CI Integration

Example

```yaml
sonarqube-check:

  image: maven:3.9

  script:

    - mvn clean verify sonar:sonar
```

---

# Multi-Branch Analysis

Enterprise teams rarely analyze only the main branch.

Example

```text
main

develop

release

feature/payment

feature/login

hotfix
```

SonarQube tracks quality independently for every branch.

---

# Pull Request Analysis

Every Pull Request is scanned before merge.

```text
Developer

↓

Pull Request

↓

Jenkins

↓

SonarQube

↓

Quality Gate

↓

Approve Merge
```

This prevents low-quality or vulnerable code from entering the main branch.

---

# Branch Protection Workflow

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

SonarQube

↓

Quality Gate

↓

PASS

↓

Reviewer Approval

↓

Merge

↓

Deployment
```

---

# Enterprise Best Practices

- Analyze every Pull Request.
- Never bypass the Quality Gate.
- Store authentication tokens in a secure secret manager.
- Use dedicated service accounts for CI/CD.
- Configure separate Quality Profiles for different languages.
- Enforce minimum code coverage thresholds.
- Regularly review Security Hotspots.
- Keep SonarQube and plugins updated.
- Monitor analysis performance and database health.
- Back up the SonarQube database regularly.

---

# Enterprise DevSecOps Pipeline

The following pipeline represents how SonarQube is integrated into a production-grade DevSecOps workflow.

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
                 Pull Request (PR)
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
                 Checkout Source
                         │
                         ▼
                  Install Dependencies
                         │
                         ▼
                  Compile Application
                         │
                         ▼
                     Unit Tests
                         │
                         ▼
                 Code Coverage (JaCoCo)
                         │
                         ▼
                SonarQube Code Analysis
                         │
                         ▼
                    Quality Gate
               ┌─────────┴─────────┐
               │                   │
             PASS                FAIL
               │                   │
               │             Stop Pipeline
               │                   │
               ▼                   ▼
        Software Composition   Developer Fix
        Analysis (SCA)              │
               │                   │
               ▼                   │
        Secret Scanning            │
               │                   │
               ▼                   │
        Build Docker Image         │
               │                   │
               ▼                   │
      Trivy Container Scan         │
               │                   │
               ▼                   │
          Generate SBOM            │
               │                   │
               ▼                   │
        Cosign Image Signing       │
               │                   │
               ▼                   │
        Push to Amazon ECR         │
               │                   │
               ▼                   │
      Update GitOps Repository     │
               │                   │
               ▼                   │
             ArgoCD Sync           │
               │                   │
               ▼                   │
        Deploy to Amazon EKS       │
               │
               ▼
           Smoke Tests
               │
               ▼
          Production
               │
               ▼
         Prometheus Metrics
               │
               ▼
        Grafana Dashboards
               │
               ▼
            ELK Logging
```

---

# Reports & Dashboards

SonarQube generates comprehensive reports that help developers, security engineers, and managers understand code quality and security.

## Project Dashboard

Displays

- Overall Quality Gate Status
- Bugs
- Vulnerabilities
- Code Smells
- Security Hotspots
- Technical Debt
- Code Coverage
- Duplicate Code
- Reliability Rating
- Maintainability Rating

---

## Security Dashboard

Security teams primarily monitor:

```text
Critical Vulnerabilities

↓

High Vulnerabilities

↓

Security Hotspots

↓

Reviewed Hotspots

↓

Resolved Issues
```

---

## Technical Debt Dashboard

Provides visibility into:

- Estimated remediation time
- Maintainability rating
- Code smells
- Duplicate code
- Long-term improvement trends

---

## Pull Request Dashboard

Developers can review:

- New bugs
- New vulnerabilities
- Coverage changes
- Quality Gate status
- Inline issue comments

---

# Enterprise Best Practices

## Installation

- Use PostgreSQL instead of the embedded database.
- Deploy SonarQube behind an Nginx reverse proxy.
- Always enable HTTPS.
- Use DNS instead of IP addresses.
- Schedule regular database backups.

---

## Authentication

- Integrate with LDAP or Azure AD.
- Enable Single Sign-On (SSO).
- Disable default administrator credentials.
- Enforce Multi-Factor Authentication (through the identity provider).

---

## Security

- Store API tokens in Jenkins Credentials, GitHub Secrets, or a secrets manager.
- Never hardcode tokens in repositories.
- Rotate tokens periodically.
- Restrict administrative access using RBAC.
- Audit user activity regularly.

---

## CI/CD

- Scan every Pull Request.
- Block deployments when the Quality Gate fails.
- Enforce minimum code coverage.
- Review Security Hotspots before release.
- Run SonarQube before Docker image creation.

---

## Operations

- Monitor CPU, memory, and database usage.
- Upgrade SonarQube regularly.
- Keep language plugins up to date.
- Archive inactive projects.
- Monitor Elasticsearch health.

---

# Common Mistakes

## Mistake 1

Ignoring Quality Gate failures.

Result

```text
Vulnerable Code

↓

Production

↓

Security Incident
```

---

## Mistake 2

Using the administrator token in CI/CD.

Correct approach

```text
Jenkins

↓

Dedicated Service Account

↓

Project Token
```

---

## Mistake 3

Skipping Pull Request analysis.

Result

```text
Unreviewed Code

↓

Merged

↓

Production Issues
```

---

## Mistake 4

Scanning only before releases.

Correct approach

```text
Every Commit

↓

Every Pull Request

↓

Every Build
```

---

## Mistake 5

Ignoring Security Hotspots.

Remember:

A Security Hotspot requires manual review before deciding whether it is a true vulnerability or acceptable in context.

---

# Common Troubleshooting

## Issue 1

### Quality Gate Failed

**Cause**

Critical vulnerabilities, blocker bugs, or insufficient test coverage.

**Resolution**

```text
Pipeline Failed

↓

Open Sonar Dashboard

↓

Review Findings

↓

Fix Code

↓

Commit Changes

↓

Pipeline Re-runs

↓

Quality Gate Passed
```

---

## Issue 2

### Sonar Scanner Authentication Failed

**Cause**

Expired or incorrect authentication token.

**Resolution**

```text
Generate New Token

↓

Update Jenkins Credentials

↓

Restart Pipeline
```

---

## Issue 3

### Coverage Report Not Displayed

**Cause**

JaCoCo report was not generated or not referenced correctly.

**Resolution**

```text
Run Unit Tests

↓

Generate JaCoCo Report

↓

Configure sonar.coverage.jacoco.xmlReportPaths

↓

Run Sonar Analysis Again
```

---

## Issue 4

### SonarQube Server Cannot Connect to PostgreSQL

**Cause**

Database service is unavailable or JDBC configuration is incorrect.

**Resolution**

```text
Check PostgreSQL

↓

Verify JDBC URL

↓

Verify Username

↓

Verify Password

↓

Restart SonarQube
```

---

## Issue 5

### Analysis Takes Too Long

**Cause**

Large repositories or unnecessary files included in the scan.

**Resolution**

```text
Exclude Generated Files

↓

Exclude Build Artifacts

↓

Optimize Scanner Configuration

↓

Re-run Analysis
```

---

# Production Interview Questions

## Question 1

### Explain how SonarQube fits into your DevSecOps pipeline.

**Answer**

SonarQube runs after unit testing and code coverage generation. It performs static code analysis, uploads results to the SonarQube server, evaluates the Quality Gate, and blocks the pipeline if quality or security requirements are not met. Only code that passes the Quality Gate proceeds to container build and deployment.

---

## Question 2

### Why should SonarQube run before Docker image creation?

**Answer**

Running SonarQube before building the container prevents insecure or low-quality code from being packaged into images, saving build time and reducing downstream risk.

---

## Question 3

### How do you authenticate Jenkins with SonarQube?

**Answer**

Generate a project token in SonarQube, store it securely in Jenkins Credentials, configure the SonarQube server in Jenkins, and use `withSonarQubeEnv` within the Jenkins pipeline.

---

## Question 4

### What is a Quality Gate?

**Answer**

A Quality Gate is a set of predefined quality conditions, such as zero critical vulnerabilities and minimum code coverage. The pipeline continues only if all conditions are satisfied.

---

## Question 5

### What is the difference between a Quality Profile and a Quality Gate?

**Answer**

A Quality Profile defines the analysis rules applied to the code, while a Quality Gate evaluates the analysis results against defined thresholds to determine whether the build passes.

---

## Question 6

### How do you secure SonarQube in production?

**Answer**

Use HTTPS, integrate with LDAP or SSO, enforce RBAC, use dedicated service accounts, store tokens securely, and regularly update SonarQube and its plugins.

---

## Question 7

### How do you analyze Pull Requests?

**Answer**

Configure the CI pipeline to trigger SonarQube analysis on every Pull Request. Developers review the Quality Gate results before the PR is approved and merged.

---

## Question 8

### How do you manage SonarQube permissions?

**Answer**

Assign permissions through groups and roles. Developers receive analysis permissions, Security teams review findings, Auditors have read-only access, and CI uses dedicated service accounts with minimal privileges.

---

## Question 9

### How do you troubleshoot a failed Quality Gate?

**Answer**

Open the SonarQube dashboard, identify blocker issues or failed conditions, fix the source code, commit the changes, and rerun the pipeline until the Quality Gate passes.

---

## Question 10

### Which reports do development and security teams commonly use?

**Answer**

Teams monitor project dashboards, security reports, technical debt metrics, code coverage trends, duplicate code statistics, and Pull Request analysis to improve code quality and security continuously.

---

# Key Takeaways

- SonarQube is an enterprise SAST platform integrated into CI/CD pipelines.
- Install SonarQube using PostgreSQL and expose it securely through an HTTPS reverse proxy.
- Integrate with LDAP, OAuth, or SSO for centralized authentication.
- Use RBAC and dedicated CI service accounts with project-specific tokens.
- Configure Quality Profiles and Quality Gates to enforce coding and security standards.
- Integrate SonarQube with Jenkins, GitHub Actions, or GitLab CI to automate analysis.
- Analyze every Pull Request and block deployments when the Quality Gate fails.
- Run SonarQube before building container images to stop insecure code early.
- Continuously monitor reports, review Security Hotspots, and maintain the SonarQube platform.