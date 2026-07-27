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

