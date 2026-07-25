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