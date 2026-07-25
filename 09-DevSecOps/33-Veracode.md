# Veracode

## Introduction

Veracode is a cloud-native Application Security (AppSec) platform that helps organizations identify, prioritize, and remediate security vulnerabilities throughout the Software Development Life Cycle (SDLC).

Unlike standalone SAST tools, Veracode provides multiple security testing capabilities including Static Application Security Testing (SAST), Software Composition Analysis (SCA), Dynamic Application Security Testing (DAST), Container Security, API Security, Secrets Detection, and Policy Management through a single platform.

Veracode is commonly adopted by enterprises that require centralized security governance, compliance reporting, developer remediation guidance, and security policy enforcement.

---

# Why Companies Use Veracode

Modern applications contain security risks beyond source code.

Organizations need visibility into:

- Custom application code
- Third-party libraries
- Open-source dependencies
- Web applications
- APIs
- Containers
- Secrets
- Compliance status

Veracode provides centralized application security management across all these areas.

## Benefits

- Enterprise SAST
- Software Composition Analysis (SCA)
- Dynamic Application Security Testing (DAST)
- API Security Testing
- Container Security
- Secrets Detection
- Security Policy Enforcement
- Compliance Reporting
- Developer Fix Guidance
- CI/CD Integration

---

# Where Veracode Fits in DevSecOps

Veracode performs multiple security checks throughout the software delivery pipeline.

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

Build Application

↓

Unit Tests

↓

SonarQube

↓

Package Application

↓

Veracode Static Analysis

↓

Software Composition Analysis

↓

Policy Evaluation

↓

Docker Build

↓

Trivy Image Scan

↓

SBOM Generation

↓

Image Signing

↓

Artifact Repository

↓

GitOps

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

Veracode complements SonarQube by providing enterprise-grade application security testing, governance, and compliance.

---

# Veracode Security Services

| Service | Purpose |
|----------|----------|
| Static Analysis | Source and binary code analysis |
| SCA | Open-source dependency analysis |
| Dynamic Analysis | Runtime web application testing |
| Container Security | Container vulnerability scanning |
| Secrets Detection | Detect exposed credentials |
| API Security | API vulnerability assessment |
| Policy Management | Enterprise security governance |
| Reporting | Compliance and executive dashboards |

---

# Enterprise Architecture

```text
                  Developers
                       │
                       ▼
               GitHub Enterprise
                       │
                       ▼
           Jenkins / GitHub Actions
                       │
                       ▼
                Build Application
                       │
                       ▼
                 Upload Artifact
                       │
                       ▼
               Veracode Platform
      ┌────────────┼─────────────┐
      ▼            ▼             ▼
   SAST         SCA          Secrets
      │            │             │
      └────────────┼─────────────┘
                   ▼
             Policy Evaluation
                   │
          PASS ────┴──── FAIL
             │              │
             ▼              ▼
      Continue CI/CD   Stop Pipeline
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

Build Artifact

↓

Veracode Upload

↓

Veracode Cloud

↓

Policy Evaluation

↓

Docker Build

↓

Trivy

↓

Amazon ECR

↓

ArgoCD

↓

Amazon EKS
```

---

# Prerequisites

Before integrating Veracode, ensure the following components are available.

| Component | Version |
|------------|----------|
| Java | 17+ |
| Maven / Gradle | Latest |
| Jenkins | Latest |
| GitHub Actions | Supported |
| Git | Latest |
| Internet Access | Required |
| Veracode Account | Required |

---

# Veracode Platform Overview

Unlike self-hosted security scanners, Veracode is delivered as Software-as-a-Service (SaaS).

```text
Organization

↓

Applications

↓

Security Policies

↓

Scans

↓

Reports

↓

Developers

↓

Security Team
```

All security findings are stored centrally within the Veracode platform.

---

# Account Setup

After purchasing or provisioning a Veracode tenant:

1. Sign in to the Veracode Portal.
2. Create your organization.
3. Configure security policies.
4. Add applications.
5. Invite users.
6. Configure SSO if required.

---

# Create an Application Profile

Navigate to:

```text
Applications

↓

Create Application
```

Example

```text
Application Name

Payment Service

Business Unit

Digital Banking

Criticality

High
```

Application Profiles organize scan results and security policies.

---

# Generate API Credentials

Veracode integrations use API credentials instead of usernames and passwords.

Navigate to:

```text
User Profile

↓

API Credentials

↓

Generate Credentials
```

Example

```text
API ID

xxxxxxxxxxxxxxxx

API Secret

xxxxxxxxxxxxxxxx
```

Store both values securely.

---

# Authentication Methods

Supported authentication methods include:

- API Credentials
- Single Sign-On (SAML)
- OpenID Connect (OIDC)
- Enterprise Identity Provider

CI/CD pipelines should always authenticate using API credentials stored in a secure secret manager.

---

# Install Veracode CLI

Linux installation.

Download the latest CLI package from the Veracode Developer Portal and install it according to your operating system.

Verify installation.

```bash
veracode version
```

Expected output.

```text
Veracode CLI x.x.x
```

---

# Configure API Credentials

Set environment variables.

```bash
export VERACODE_API_KEY_ID=xxxxxxxx

export VERACODE_API_KEY_SECRET=xxxxxxxx
```

Verify.

```bash
env | grep VERACODE
```

Never hardcode API credentials in repositories or pipeline definitions.

---

# Install Java API Wrapper

Download the Java API Wrapper.

```bash
wget https://repo1.maven.org/maven2/com/veracode/vosp/api/wrappers/vosp-api-wrappers-java/latest/vosp-api-wrappers-java.jar
```

Verify.

```bash
ls
```

Example.

```text
vosp-api-wrappers-java.jar
```

The wrapper enables automation from CI/CD pipelines.

---

# Install Pipeline Scan

Download the Pipeline Scan package from the Veracode platform.

Example.

```text
pipeline-scan.jar
```

Verify.

```bash
java -jar pipeline-scan.jar --help
```

Pipeline Scan enables fast security analysis during CI/CD without waiting for a full policy scan.

---

# Configure Credentials File

Instead of environment variables, credentials can also be stored in a configuration file.

Example.

```text
~/.veracode/credentials
```

Configuration.

```ini
[default]

veracode_api_key_id = YOUR_API_KEY_ID

veracode_api_key_secret = YOUR_API_KEY_SECRET
```

Restrict file permissions.

```bash
chmod 600 ~/.veracode/credentials
```

---

# Verify Connectivity

Verify authentication.

```bash
veracode applications list
```

Expected output.

```text
Payment Service

Inventory Service

User Service
```

This confirms that the CLI can successfully communicate with the Veracode platform.

---

# First Pipeline Scan

Run a pipeline scan against the application package.

```bash
java -jar pipeline-scan.jar \
--file target/payment-service.jar
```

Example output.

```text
Policy Passed

High Severity : 0

Medium Severity : 2

Low Severity : 5
```

If policy violations are detected, the pipeline should fail and require remediation before continuing.

---

# Veracode Configuration

Veracode can be configured using:

- API Credentials
- Credentials File
- Environment Variables
- CI/CD Secrets
- Policy Configuration

A centralized configuration ensures consistent security scanning across development, testing, and production environments.

---

# Configuration Priority

When multiple authentication methods are available, Veracode uses the following priority.

```text
Environment Variables

↓

Credentials File

↓

Interactive Login (CLI)

↓

Default Configuration
```

---

# API Authentication

Veracode APIs use API Key authentication instead of username/password authentication.

Environment variables.

```bash
export VERACODE_API_KEY_ID=xxxxxxxxxxxxxxxx

export VERACODE_API_KEY_SECRET=xxxxxxxxxxxxxxxx
```

Credentials file.

```ini
[default]

veracode_api_key_id=xxxxxxxxxxxxxxxx

veracode_api_key_secret=xxxxxxxxxxxxxxxx
```

Verify authentication.

```bash
veracode applications list
```

---

# Enterprise Credential Management

Never store API credentials inside:

- Source code
- Git repositories
- Jenkinsfiles
- Docker images
- Terraform variables

Store credentials securely using:

- Jenkins Credentials
- GitHub Secrets
- GitLab CI Variables
- AWS Secrets Manager
- Azure Key Vault
- HashiCorp Vault

Architecture.

```text
Developer

↓

Secret Manager

↓

CI/CD Pipeline

↓

Veracode API

↓

Security Scan
```

---

# Pipeline Scan Configuration

Pipeline Scan is optimized for CI/CD.

Example.

```bash
java -jar pipeline-scan.jar \
--file target/payment-service.jar \
--fail_on_severity="Very High,High"
```

Behavior.

```text
Application Build

↓

Upload Binary

↓

Security Scan

↓

Policy Evaluation

↓

PASS

↓

Continue Pipeline

OR

FAIL

↓

Pipeline Stops
```

---

# Policy Configuration

Security policies define the minimum security requirements that applications must satisfy before deployment.

Example policy.

| Rule | Value |
|------|-------|
| Very High Severity | 0 |
| High Severity | 0 |
| Medium Severity | Review Required |
| Low Severity | Allowed |
| CWE Coverage | Enabled |

Example workflow.

```text
Pipeline Scan

↓

Policy Evaluation

↓

PASS

↓

Deployment

OR

FAIL

↓

Developer Remediation
```

---

# Sandbox Configuration

Sandboxes provide isolated environments for testing without affecting production results.

Example.

```text
Payment Service

├── Production

├── QA Sandbox

├── Development Sandbox

└── Feature Sandbox
```

Benefits.

- Test security fixes
- Validate new features
- Separate development from production
- Prevent false production reports

---

# Applications

Each application should have its own profile.

Example.

```text
Organization

│

├── Payment Service

├── Inventory Service

├── User Service

├── Order Service

└── Notification Service
```

Benefits.

- Separate reporting
- Individual policies
- Easier auditing
- Team ownership

---

# Teams

Applications should be grouped by business ownership.

Example.

```text
Organization

│

├── Digital Banking

├── Platform Engineering

├── Mobile Team

├── Security Team

└── Shared Services
```

This simplifies permission management and reporting.

---

# User Management

Typical enterprise users include:

- Security Administrator
- Security Engineer
- Development Lead
- Developer
- Auditor
- Compliance Officer
- CI/CD Service Account

Avoid sharing user accounts.

Each user should have an individual identity.

---

# Role-Based Access Control (RBAC)

Example permission model.

| Role | Responsibilities |
|------|------------------|
| Administrator | Manage platform, users, policies, integrations |
| Security Team | Review findings, approve mitigations, manage scans |
| Development Lead | Manage application scans and remediation |
| Developer | View findings and fix vulnerabilities |
| Auditor | Read-only access to reports and compliance data |
| CI Service Account | Execute automated scans only |

Apply the Principle of Least Privilege.

---

# Single Sign-On (SSO)

Large organizations integrate Veracode with corporate identity providers.

Supported options include:

- SAML
- OIDC
- Azure AD
- Okta
- Ping Identity

Authentication flow.

```text
Developer

↓

Corporate Identity Provider

↓

Single Sign-On

↓

Veracode Platform
```

Benefits.

- Centralized authentication
- Multi-Factor Authentication
- User lifecycle management
- Reduced password management

---

# Proxy Configuration

Some enterprise environments require outbound traffic through a proxy.

Example.

```bash
export HTTPS_PROXY=http://proxy.company.com:8080

export HTTP_PROXY=http://proxy.company.com:8080

export NO_PROXY=localhost,127.0.0.1
```

Verify.

```bash
env | grep PROXY
```

---

# Scan Modes

Veracode supports multiple scan types.

| Scan | Purpose |
|------|----------|
| Pipeline Scan | Fast CI/CD security checks |
| Static Analysis | Comprehensive binary analysis |
| Software Composition Analysis | Open-source dependency scanning |
| Dynamic Analysis | Runtime web application testing |
| Container Security | Container image assessment |

Typical workflow.

```text
Developer

↓

Pipeline Scan

↓

Merge

↓

Static Analysis

↓

Production Approval
```

---

# Security Policies

Different applications may require different policies.

Example.

```text
Critical Banking Application

↓

Strict Policy

↓

Zero High Severity Findings
```

```text
Internal Utility

↓

Standard Policy

↓

Limited Exceptions Allowed
```

Policy-based governance ensures consistent security requirements across the organization.

---

# Mitigation Workflow

Not every finding can be fixed immediately.

Example process.

```text
Security Finding

↓

Developer Review

↓

Security Team Review

↓

Approve Mitigation

↓

Document Exception

↓

Next Release
```

All mitigations should be documented and periodically reviewed.

---

# Compliance Reporting

Veracode provides reports that support compliance frameworks such as:

- PCI DSS
- ISO 27001
- SOC 2
- NIST
- HIPAA
- OWASP ASVS

Reports help demonstrate secure development practices during audits.

---

# Enterprise Best Practices

- Integrate Veracode into every CI/CD pipeline.
- Use Pipeline Scan for rapid feedback during development.
- Schedule full Static Analysis for release builds.
- Enforce security policies before production deployments.
- Store API credentials in a secure secrets manager.
- Integrate SSO with the corporate identity provider.
- Use RBAC and least-privilege access for all users.
- Separate production and development scans using sandboxes.
- Review mitigations regularly and remove expired exceptions.
- Monitor compliance reports as part of regular security governance.

---

# Static Application Security Testing (SAST)

Static Analysis scans compiled application binaries without executing the application.

Unlike Pipeline Scan, Static Analysis performs a comprehensive security assessment and is typically executed for release builds.

## Architecture

```text
Developer

↓

Build Application

↓

Package Binary

↓

Upload to Veracode

↓

Static Analysis

↓

Security Findings

↓

Policy Evaluation

↓

PASS / FAIL
```

---

# Supported Package Types

Veracode supports numerous application packages.

Examples include:

- Java (.jar, .war, .ear)
- .NET (.dll, .exe)
- JavaScript
- Python
- Go
- PHP
- Ruby
- Android APK
- iOS IPA

---

# Upload an Application

Example using the CLI.

```bash
veracode static scan submit \
--appname "Payment Service" \
--file target/payment-service.jar
```

Workflow.

```text
Package Application

↓

Upload Binary

↓

Static Analysis

↓

Results Available
```

---

# Static Analysis Results

Typical findings include:

- SQL Injection
- Cross-Site Scripting (XSS)
- Command Injection
- Path Traversal
- XML External Entity (XXE)
- Insecure Cryptography
- Authentication Issues
- Authorization Issues

Each finding includes:

- Severity
- CWE ID
- File Name
- Line Number (when available)
- Remediation Guidance

---

# Software Composition Analysis (SCA)

Software Composition Analysis identifies vulnerable third-party libraries and open-source dependencies.

## Architecture

```text
Application

↓

Dependency Analysis

↓

Known CVEs

↓

License Review

↓

Security Report
```

Example scan.

```bash
veracode sca scan
```

SCA identifies:

- Vulnerable packages
- Outdated dependencies
- License violations
- Dependency risk

---

# Dependency Risk Example

```text
Spring Boot

↓

Jackson

↓

Log4j

↓

Known CVE

↓

Upgrade Required
```

Benefits.

- Early detection of vulnerable libraries
- Faster remediation
- Supply chain visibility
- License compliance

---

# Dynamic Application Security Testing (DAST)

Dynamic Analysis evaluates a running application by simulating real-world attacks.

Unlike SAST, the application must be deployed and accessible.

## Architecture

```text
Running Web Application

↓

Veracode DAST

↓

HTTP Requests

↓

Attack Simulation

↓

Security Findings
```

Typical vulnerabilities detected:

- Cross-Site Scripting (XSS)
- SQL Injection
- Session Management Issues
- Authentication Weaknesses
- CSRF
- Security Header Issues

---

# API Security Testing

Modern applications expose REST and GraphQL APIs.

Veracode validates API endpoints for security vulnerabilities.

Architecture.

```text
API Specification

↓

API Scan

↓

Authentication

↓

Endpoint Testing

↓

Security Report
```

API testing helps identify:

- Broken authentication
- Authorization issues
- Injection vulnerabilities
- Sensitive data exposure
- Improper API configuration

---

# Container Security

Veracode also evaluates container images for security risks.

Typical workflow.

```text
Docker Image

↓

Veracode

↓

Package Analysis

↓

Known CVEs

↓

Policy Evaluation
```

Container analysis includes:

- Operating system packages
- Application libraries
- Vulnerable dependencies
- Configuration issues

---

# Secret Detection

Veracode detects sensitive credentials committed to source code.

Examples include:

- AWS Access Keys
- Azure Credentials
- GitHub Tokens
- API Keys
- Private Keys
- Passwords

Workflow.

```text
Source Code

↓

Secret Detection

↓

Credential Found

↓

Developer Notification
```

Secrets should be removed immediately and rotated.

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

Pipeline Scan

↓

Static Analysis

↓

Policy Evaluation

↓

Continue Pipeline
```

Store Veracode API credentials in Jenkins Credentials.

Required credentials.

```text
VERACODE_API_KEY_ID

VERACODE_API_KEY_SECRET
```

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    environment {

        VERACODE_API_KEY_ID = credentials('veracode-api-id')
        VERACODE_API_KEY_SECRET = credentials('veracode-api-secret')

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

        stage('Pipeline Scan') {

            steps {

                sh '''
                java -jar pipeline-scan.jar \
                --file target/payment-service.jar \
                --fail_on_severity="Very High,High"
                '''
            }

        }

    }

}
```

---

# GitHub Actions Integration

Store the following GitHub Secrets.

```text
VERACODE_API_KEY_ID

VERACODE_API_KEY_SECRET
```

---

# Production GitHub Actions Workflow

```yaml
name: Veracode Pipeline Scan

on:

  push:

    branches:

      - main

jobs:

  veracode:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - name: Build

        run: mvn clean package

      - name: Run Veracode Pipeline Scan

        env:
          VERACODE_API_KEY_ID: ${{ secrets.VERACODE_API_KEY_ID }}
          VERACODE_API_KEY_SECRET: ${{ secrets.VERACODE_API_KEY_SECRET }}

        run: |
          java -jar pipeline-scan.jar \
          --file target/payment-service.jar \
          --fail_on_severity="Very High,High"
```

---

# GitLab CI Integration

```yaml
veracode_scan:

  image: eclipse-temurin:17

  script:

    - mvn clean package

    - java -jar pipeline-scan.jar \
      --file target/payment-service.jar \
      --fail_on_severity="Very High,High"
```

---

# Report Formats

Veracode provides multiple reporting options.

| Report | Purpose |
|---------|----------|
| PDF | Executive reports |
| HTML | Security review |
| JSON | CI/CD automation |
| XML | Tool integration |
| CSV | Compliance and auditing |

---

# Recommended Scan Strategy

```text
Developer

↓

Pipeline Scan

↓

Pull Request

↓

Merge

↓

Build

↓

Static Analysis

↓

Software Composition Analysis

↓

Policy Evaluation

↓

Docker Build

↓

Trivy Image Scan

↓

Deploy
```

---

