# OWASP ZAP

## Introduction

OWASP ZAP (Zed Attack Proxy) is one of the world's most widely used open-source Dynamic Application Security Testing (DAST) tools.

Unlike SAST tools that analyze source code, OWASP ZAP tests running applications by sending HTTP requests and identifying vulnerabilities that exist during runtime.

It helps organizations discover security issues before applications reach production by continuously testing web applications, REST APIs, and microservices during the CI/CD pipeline.

---

# Why Companies Use OWASP ZAP

Static code analysis cannot detect every security vulnerability.

Many vulnerabilities only appear when an application is running.

OWASP ZAP helps identify these runtime security issues.

## Benefits

- Dynamic Application Security Testing (DAST)
- REST API security testing
- Web application security testing
- Authentication testing
- Session testing
- Security regression testing
- CI/CD integration
- Automated vulnerability detection
- OWASP Top 10 coverage
- Production security validation

---

# SAST vs DAST

| Feature | SAST | DAST |
|----------|------|------|
| Source Code Required | ✓ | ✗ |
| Running Application Required | ✗ | ✓ |
| Detects Runtime Issues | ✗ | ✓ |
| Detects Code Quality | ✓ | ✗ |
| Tests APIs | Limited | ✓ |
| Tests Authentication | ✗ | ✓ |
| Tests HTTP Responses | ✗ | ✓ |
| Pipeline Stage | Build | Post Deployment |

---

# Where OWASP ZAP Fits in DevSecOps

DAST should execute after the application has been deployed to a testing environment.

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

Build

↓

Unit Testing

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

Deploy to Test Environment

↓

OWASP ZAP Scan

↓

Security Validation

↓

Production
```

Applications should only be promoted after passing runtime security testing.

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
              Build Application
                       │
                       ▼
            Deploy Test Environment
                       │
                       ▼
                 OWASP ZAP
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
    Web UI         REST APIs     GraphQL APIs
                       │
                       ▼
             Vulnerability Report
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

Build

↓

Docker Image

↓

Amazon ECR

↓

Deploy to Amazon EKS (Testing)

↓

OWASP ZAP

↓

Security Approval

↓

Production
```

---

# What Can OWASP ZAP Detect?

| Vulnerability | Supported |
|---------------|-----------|
| SQL Injection | ✓ |
| Cross-Site Scripting (XSS) | ✓ |
| Cross-Site Request Forgery (CSRF) | ✓ |
| Command Injection | ✓ |
| Directory Traversal | ✓ |
| Missing Security Headers | ✓ |
| Cookie Security Issues | ✓ |
| Information Disclosure | ✓ |
| Session Management Issues | ✓ |
| Authentication Weaknesses | ✓ |

---

# Scan Types

OWASP ZAP supports multiple scan modes.

| Scan Type | Purpose |
|------------|----------|
| Passive Scan | Analyze traffic without attacking the application |
| Active Scan | Actively test for vulnerabilities |
| Spider | Discover application pages |
| AJAX Spider | Discover JavaScript-driven applications |
| API Scan | Test REST and OpenAPI endpoints |
| Baseline Scan | Quick CI/CD security validation |

---

# Prerequisites

| Component | Version |
|------------|----------|
| Ubuntu | 22.04 LTS |
| Docker | Latest |
| Java | 17+ |
| Git | Latest |
| Jenkins | Latest |
| Kubernetes | Supported |

---

# Installation Methods

OWASP ZAP can be installed using:

- Docker
- Linux Package
- Windows Installer
- macOS
- Jenkins
- GitHub Actions
- Kubernetes

Docker is the preferred approach for CI/CD pipelines.

---

# Install Using Docker

Pull the official image.

```bash
docker pull ghcr.io/zaproxy/zaproxy:stable
```

Verify.

```bash
docker run --rm \
ghcr.io/zaproxy/zaproxy:stable \
zap.sh -version
```

---

# Install on Ubuntu

Download the latest release.

```bash
wget https://github.com/zaproxy/zaproxy/releases/latest/download/ZAP_latest_linux.tar.gz
```

Extract.

```bash
tar -xzf ZAP_latest_linux.tar.gz
```

Start ZAP.

```bash
cd ZAP*

./zap.sh
```

---

# Install on Jenkins Agent

Verify Docker.

```bash
docker --version
```

Pull the ZAP image.

```bash
docker pull ghcr.io/zaproxy/zaproxy:stable
```

Verify.

```bash
docker run --rm \
ghcr.io/zaproxy/zaproxy:stable \
zap.sh -version
```

---

# Install on GitHub Actions Runner

GitHub-hosted runners can execute OWASP ZAP directly from the official Docker image.

Example.

```yaml
- name: Pull OWASP ZAP

  run: |

    docker pull ghcr.io/zaproxy/zaproxy:stable
```

Self-hosted runners should cache the Docker image to reduce pipeline execution time.

---

# Verify Installation

Run.

```bash
docker run --rm \
ghcr.io/zaproxy/zaproxy:stable \
zap.sh -version
```

Example output.

```text
OWASP ZAP

↓

Version 2.x.x
```

---

# First Baseline Scan

Run a baseline scan against a test application.

```bash
docker run --rm \
-t ghcr.io/zaproxy/zaproxy:stable \
zap-baseline.py \
-t https://example.com
```

Example workflow.

```text
Target URL

↓

Spider

↓

Passive Scan

↓

Generate Report

↓

PASS / FAIL
```

The baseline scan performs passive security analysis without attempting to exploit vulnerabilities, making it suitable for execution during every CI/CD pipeline.

---

# Configuration

OWASP ZAP can be configured for different environments ranging from local development to enterprise CI/CD pipelines.

Configuration can include:

- Scan policies
- Authentication
- Contexts
- Users
- API configuration
- Report generation
- Proxy settings
- Exclusions

Proper configuration reduces false positives and improves scan accuracy.

---

# Configuration Components

| Component | Purpose |
|-----------|----------|
| Context | Defines application scope |
| User | Authenticated scanning |
| Scan Policy | Controls attack rules |
| Authentication | Login configuration |
| Exclude Rules | Ignore specific URLs |
| Reports | Generate security findings |
| API | Automation support |

---

# Configuration Priority

```text
Command Line

↓

Automation Framework

↓

Context File

↓

Default Configuration
```

---

# Context

A Context defines which applications and URLs should be tested.

Example.

```text
Context

↓

example.com

↓

/api/*

↓

/login

↓

/dashboard
```

Everything outside the context is ignored.

---

# Creating a Context

Start ZAP.

```bash
./zap.sh
```

Navigate.

```text
File

↓

New Context

↓

Add URLs

↓

Save
```

Contexts improve scan performance by limiting unnecessary requests.

---

# Authentication

Many enterprise applications require login before testing.

Authentication types include:

- Form-Based Authentication
- HTTP Authentication
- OAuth
- JWT
- Session Cookies

Workflow.

```text
User

↓

Login

↓

Authentication

↓

Session

↓

Authenticated Scan
```

---

# User Management

Applications can have multiple users.

Example.

```text
Admin

Developer

Operator

ReadOnly
```

Separate users help validate role-based access control (RBAC).

---

# Scan Policies

Scan policies define which vulnerability checks should be executed.

Common policies.

- SQL Injection
- Cross-Site Scripting
- Command Injection
- Path Traversal
- Server Misconfiguration
- Information Disclosure

Example.

```text
Policy

↓

Attack Rules

↓

Enabled

↓

Run Scan
```

---

# Passive Scan

Passive scanning observes HTTP traffic without modifying requests.

Workflow.

```text
Browser

↓

HTTP Request

↓

Proxy

↓

Passive Analysis

↓

Report
```

Passive scans are safe to run continuously.

---

# Active Scan

Active scanning attempts to exploit vulnerabilities.

Workflow.

```text
Spider

↓

Attack Rules

↓

Target Application

↓

Response Analysis

↓

Security Report
```

Active scans should only be executed against non-production environments.

---

# Spider Scan

Spider discovers application pages automatically.

Run.

```bash
zap.sh -cmd \
-spider https://example.com
```

Workflow.

```text
Home Page

↓

Links

↓

Forms

↓

New Pages

↓

Complete Site Map
```

---

# AJAX Spider

Modern web applications often rely on JavaScript.

AJAX Spider executes JavaScript before crawling.

Run.

```bash
zap.sh -cmd \
-ajaxSpider https://example.com
```

Useful for:

- React
- Angular
- Vue
- Single Page Applications

---

# API Scanning

OWASP ZAP supports REST API testing.

Example.

```bash
zap-api-scan.py \
-f openapi \
-t openapi.yaml
```

Supported specifications.

- OpenAPI
- Swagger
- GraphQL
- SOAP

---

# Proxy Configuration

OWASP ZAP operates as an intercepting proxy.

Default proxy.

```text
Host

↓

localhost

↓

Port

↓

8080
```

Applications route traffic through ZAP for analysis.

---

# Excluding URLs

Some URLs should not be scanned.

Examples.

```text
/logout

/health

/metrics

/static

/images
```

Excluding these paths reduces unnecessary requests and avoids session termination.

---

# API Configuration

Enable API access.

Example.

```bash
zap.sh \
-config api.disablekey=true
```

Production environments should use API keys instead of disabling authentication.

---

# Report Formats

OWASP ZAP supports multiple report formats.

| Format | Purpose |
|----------|----------|
| HTML | Human-readable report |
| JSON | Automation |
| XML | Integration |
| Markdown | Documentation |
| SARIF | GitHub Security |

---

# Generate HTML Report

Example.

```bash
zap-baseline.py \
-t https://example.com \
-r report.html
```

---

# Generate JSON Report

Example.

```bash
zap-baseline.py \
-t https://example.com \
-J report.json
```

---

# Generate Markdown Report

Example.

```bash
zap-baseline.py \
-t https://example.com \
-w report.md
```

---

# Report Directory

```text
reports/

├── report.html

├── report.json

├── report.xml

├── report.md

└── report.sarif
```

Reports should be stored as CI/CD artifacts for auditing and compliance.

---

# Exit Codes

OWASP ZAP returns exit codes that pipelines can use to determine success or failure.

| Exit Code | Meaning |
|-----------|---------|
| 0 | No Alerts |
| 1 | At Least One Failure |
| 2 | Warnings Present |
| 3 | Execution Error |

Production pipelines should fail when High or Critical vulnerabilities are detected.

---

# Enterprise Best Practices

- Run passive scans on every build.
- Execute active scans only in isolated test environments.
- Define contexts for every application.
- Use authenticated scanning for protected applications.
- Exclude logout and health-check endpoints.
- Generate HTML and JSON reports for every pipeline.
- Archive reports for compliance audits.
- Integrate ZAP with CI/CD automation.
- Keep scan policies updated.
- Review and remediate High-risk findings before deployment.

---

