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

# Baseline Scan

A Baseline Scan performs passive security testing without attacking the application.

It is safe to run on every CI/CD pipeline.

Run.

```bash
docker run --rm \
-t ghcr.io/zaproxy/zaproxy:stable \
zap-baseline.py \
-t https://test.example.com
```

Workflow.

```text
Target URL

↓

Spider

↓

Passive Analysis

↓

Security Report

↓

PASS / FAIL
```

---

# Active Scan

An Active Scan attempts to identify exploitable vulnerabilities by sending attack payloads to the application.

Run.

```bash
docker run --rm \
-t ghcr.io/zaproxy/zaproxy:stable \
zap-full-scan.py \
-t https://test.example.com
```

Workflow.

```text
Deploy Test Environment

↓

Spider

↓

Active Scan

↓

Vulnerability Detection

↓

Security Report
```

Active scans should never be executed against production systems unless explicitly authorised.

---

# API Security Testing

Modern applications expose REST APIs that must be validated.

Example OpenAPI scan.

```bash
docker run --rm \
-v $(pwd):/zap/wrk \
-t ghcr.io/zaproxy/zaproxy:stable \
zap-api-scan.py \
-f openapi \
-t /zap/wrk/openapi.yaml
```

Workflow.

```text
OpenAPI Specification

↓

Import API

↓

Discover Endpoints

↓

Security Tests

↓

Report
```

---

# Authenticated Application Scanning

Enterprise applications usually require authentication.

```text
Login Page

↓

Authentication

↓

Session Cookie

↓

Authenticated User

↓

Security Scan
```

Authenticated scanning validates pages accessible only after login.

---

# Session Management Testing

OWASP ZAP validates session handling.

Checks include:

- Session fixation
- Session timeout
- Cookie attributes
- Secure cookies
- HttpOnly cookies
- SameSite cookies

Workflow.

```text
Login

↓

Session Cookie

↓

Request Validation

↓

Security Analysis
```

---

# Security Header Validation

OWASP ZAP verifies recommended HTTP security headers.

Common headers.

| Header | Purpose |
|----------|----------|
| Content-Security-Policy | Prevent XSS |
| Strict-Transport-Security | Force HTTPS |
| X-Frame-Options | Prevent Clickjacking |
| X-Content-Type-Options | MIME Protection |
| Referrer-Policy | Referrer Control |
| Permissions-Policy | Browser Feature Restrictions |

Missing headers generate security alerts.

---

# OWASP Top 10 Coverage

OWASP ZAP helps detect many OWASP Top 10 vulnerabilities.

| Vulnerability | Detection |
|---------------|-----------|
| Broken Access Control | Partial |
| Cryptographic Failures | Partial |
| Injection | ✓ |
| Insecure Design | Partial |
| Security Misconfiguration | ✓ |
| Vulnerable Components | Partial |
| Authentication Failures | ✓ |
| Software Integrity Failures | Partial |
| Logging & Monitoring Issues | Partial |
| Server-Side Request Forgery | Partial |

Some vulnerabilities require manual verification.

---

# Running Against Kubernetes Applications

Example deployment flow.

```text
Developer

↓

Docker Image

↓

Amazon ECR

↓

ArgoCD

↓

Amazon EKS

↓

Ingress

↓

OWASP ZAP

↓

Security Report
```

The scan targets the application's external endpoint exposed through the Ingress or Load Balancer.

---

# Running Against Microservices

Example architecture.

```text
Frontend

↓

API Gateway

↓

User Service

↓

Order Service

↓

Payment Service

↓

Inventory Service
```

Each externally accessible API should be tested independently.

---

# Running Against Internal APIs

Internal APIs can be tested inside the Kubernetes cluster.

Example.

```text
OWASP ZAP Pod

↓

ClusterIP Service

↓

Internal API

↓

Security Scan
```

This approach validates services that are not exposed publicly.

---

# Running OWASP ZAP in Docker

Example.

```bash
docker run --rm \

-v $(pwd):/zap/wrk \

-t ghcr.io/zaproxy/zaproxy:stable \

zap-baseline.py \

-t https://test.example.com
```

Docker provides a consistent execution environment across development, testing, and CI/CD systems.

---

# Running OWASP ZAP in Kubernetes

Example Job.

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: zap-scan

spec:
  template:
    spec:
      restartPolicy: Never

      containers:

      - name: zap

        image: ghcr.io/zaproxy/zaproxy:stable

        command:

        - zap-baseline.py

        - -t

        - https://test.example.com
```

Kubernetes Jobs allow automated security testing during deployment validation.

---

# Integrating with Jenkins

Pipeline workflow.

```text
Checkout

↓

Build

↓

Deploy Test Environment

↓

OWASP ZAP

↓

Generate Report

↓

Archive Report

↓

PASS / FAIL
```

Jenkins should archive generated reports as build artifacts.

---

# Integrating with GitHub Actions

Workflow.

```text
Git Push

↓

GitHub Actions

↓

Build

↓

Deploy Test Environment

↓

OWASP ZAP

↓

Upload Report

↓

PASS / FAIL
```

Security reports can be uploaded as workflow artifacts.

---

# Integrating with GitLab CI

Pipeline.

```text
Git Commit

↓

GitLab Runner

↓

Deploy Test Environment

↓

OWASP ZAP

↓

Security Report

↓

Pipeline Result
```

Security jobs should execute after deployment to the testing environment.

---

# Enterprise Best Practices

- Scan every deployment in the testing environment.
- Execute baseline scans on every pull request.
- Run active scans during release validation.
- Use authenticated scans for protected applications.
- Scan REST APIs using OpenAPI specifications.
- Validate all Internet-facing applications.
- Store reports as pipeline artifacts.
- Review High and Critical findings before deployment.
- Keep OWASP ZAP Docker images updated.
- Combine DAST with SAST, SCA, IaC, and container security tools for comprehensive DevSecOps coverage.

---

# Jenkins Integration

OWASP ZAP is commonly executed after deploying the application to a test environment.

Enterprise pipeline.

```text
Developer

↓

Git Push

↓

Jenkins

↓

Checkout

↓

Build

↓

Docker Build

↓

Deploy to Test

↓

OWASP ZAP

↓

Generate Report

↓

Archive Report

↓

Production Approval
```

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                checkout scm

            }

        }

        stage('Build') {

            steps {

                sh 'mvn clean package'

            }

        }

        stage('Deploy Test') {

            steps {

                sh './deploy-test.sh'

            }

        }

        stage('OWASP ZAP Scan') {

            steps {

                sh '''

                docker run --rm \
                -v $(pwd):/zap/wrk \
                -t ghcr.io/zaproxy/zaproxy:stable \
                zap-baseline.py \
                -t https://test.example.com \
                -r zap-report.html \
                -J zap-report.json

                '''

            }

        }

    }

    post {

        always {

            archiveArtifacts 'zap-report.*'

        }

    }

}
```

---

# GitHub Actions Integration

Enterprise workflow.

```text
Git Push

↓

GitHub Actions

↓

Build

↓

Deploy Test

↓

OWASP ZAP

↓

Upload Reports

↓

Security Review
```

---

# Production GitHub Actions Workflow

```yaml
name: OWASP-ZAP

on:

  pull_request:

  push:

    branches:

      - main

jobs:

  zap:

    runs-on: ubuntu-latest

    steps:

    - uses: actions/checkout@v4

    - name: Build

      run: mvn clean package

    - name: Deploy Test

      run: ./deploy-test.sh

    - name: OWASP ZAP Scan

      run: |

        docker run --rm \
        -v $(pwd):/zap/wrk \
        -t ghcr.io/zaproxy/zaproxy:stable \
        zap-baseline.py \
        -t https://test.example.com \
        -r zap-report.html \
        -J zap-report.json

    - name: Upload Reports

      uses: actions/upload-artifact@v4

      with:

        name: zap-report

        path: |

          zap-report.html

          zap-report.json
```

---

# GitLab CI Integration

Example pipeline.

```yaml
stages:

  - build

  - deploy

  - security

zap:

  stage: security

  image: ghcr.io/zaproxy/zaproxy:stable

  script:

    - zap-baseline.py
      -t https://test.example.com
      -r report.html
      -J report.json

  artifacts:

    paths:

      - report.html

      - report.json
```

---

# Scan Policies

Production environments generally use multiple scan policies depending on the application.

```text
Baseline Policy

↓

Passive Scan

↓

Authentication Policy

↓

API Policy

↓

Active Scan Policy

↓

Final Security Report
```

Separate policies reduce scan duration and improve accuracy.

---

# REST API Security Testing

OWASP ZAP supports automated REST API testing using OpenAPI definitions.

```text
OpenAPI File

↓

Import Endpoints

↓

Generate Requests

↓

Attack Rules

↓

Security Report
```

Example.

```bash
zap-api-scan.py \
-f openapi \
-t openapi.yaml \
-r api-report.html
```

---

# GraphQL Security Testing

GraphQL endpoints can also be tested.

Workflow.

```text
GraphQL Endpoint

↓

Schema Discovery

↓

Security Testing

↓

Report
```

Example.

```bash
zap-api-scan.py \
-f graphql \
-t https://test.example.com/graphql
```

---

# Authentication Testing

Authenticated scans validate areas that anonymous users cannot access.

Typical authentication flow.

```text
Login Page

↓

Username

↓

Password

↓

Session Cookie

↓

Authenticated Scan
```

Supported authentication methods include:

- Form Login
- Basic Authentication
- OAuth
- JWT
- API Keys
- Session Cookies

---

# False Positive Management

Not every alert represents a genuine vulnerability.

Review process.

```text
Security Alert

↓

Developer Review

↓

Security Team Review

↓

Verified

↓

Fix

or

Risk Accepted
```

Every accepted risk should be documented.

---

# Risk Levels

OWASP ZAP categorizes findings by severity.

| Risk | Recommended Action |
|------|--------------------|
| Informational | Review |
| Low | Fix when practical |
| Medium | Resolve before release |
| High | Block deployment |
| Critical | Immediate remediation |

Production deployments should not proceed with unresolved High or Critical findings.

---

# Report Analysis

Typical report contents.

```text
Security Report

├── Alert Summary

├── Risk Levels

├── Affected URLs

├── Vulnerability Details

├── Evidence

├── Recommended Fix

└── References
```

Reports should be reviewed during release approval.

---

# Security Gates

OWASP ZAP is typically configured as a quality gate.

```text
Build

↓

Deploy Test

↓

OWASP ZAP

↓

High Risk Found?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Fail      Continue

Pipeline   Deployment
```

---

# Enterprise DevSecOps Pipeline

```text
Developer

↓

Feature Branch

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Trigger

↓

Checkout

↓

Build

↓

Unit Tests

↓

Coverage

↓

SonarQube

↓

OWASP Dependency-Check

↓

Veracode

↓

Docker Build

↓

Trivy

↓

SBOM

↓

Image Signing

↓

Artifact Repository

↓

Deploy Test Environment

↓

OWASP ZAP

↓

Security Approval

↓

GitOps

↓

ArgoCD

↓

Amazon EKS

↓

Smoke Test

↓

Production
```

OWASP ZAP provides runtime security validation immediately before production deployment.

---

# Enterprise Best Practices

- Perform baseline scans on every pull request.
- Execute active scans before production releases.
- Scan authenticated application areas.
- Test REST APIs using OpenAPI specifications.
- Test GraphQL endpoints where applicable.
- Review all High and Critical alerts.
- Maintain separate scan policies for different applications.
- Archive reports for compliance and audits.
- Integrate OWASP ZAP with Jenkins, GitHub Actions, and GitLab CI.
- Combine OWASP ZAP with SonarQube, Veracode, Trivy, Gitleaks, Checkov, and other DevSecOps tools for complete application security.

---

# Common Vulnerabilities Detected

## SQL Injection

Example.

```text
Login Form

↓

User Input

↓

SQL Query

↓

Database
```

OWASP ZAP attempts SQL injection payloads to determine whether database queries are vulnerable.

---

## Cross-Site Scripting (XSS)

Example.

```text
User Input

↓

Application

↓

Browser

↓

JavaScript Executes
```

ZAP tests whether user input is reflected or stored without proper sanitization.

---

## Cross-Site Request Forgery (CSRF)

Workflow.

```text
Authenticated User

↓

Malicious Website

↓

Unauthorized Request

↓

Application
```

ZAP checks whether applications validate CSRF tokens for state-changing requests.

---

## Command Injection

Workflow.

```text
User Input

↓

Operating System Command

↓

Server

↓

Response
```

Applications should validate and sanitize user input before executing system commands.

---

## Directory Traversal

Workflow.

```text
../../etc/passwd

↓

Application

↓

Filesystem

↓

Sensitive Files
```

ZAP identifies endpoints vulnerable to path traversal attacks.

---

## Missing Security Headers

Example findings.

```text
Missing

↓

Content-Security-Policy

Strict-Transport-Security

X-Frame-Options

X-Content-Type-Options
```

Missing headers weaken browser-side security protections.

---

## Cookie Security Issues

OWASP ZAP validates cookie attributes.

Example.

```text
Cookie

↓

Secure

↓

HttpOnly

↓

SameSite
```

Cookies should include recommended security attributes.

---

# AWS Production Example

Enterprise deployment.

```text
Developer

↓

GitHub

↓

Jenkins

↓

Docker Build

↓

Amazon ECR

↓

Amazon EKS (Testing)

↓

AWS Load Balancer

↓

OWASP ZAP

↓

Security Report
```

Target.

```text
https://test.example.com
```

Only Internet-facing endpoints exposed through the Load Balancer should be scanned.

---

# Azure Production Example

Architecture.

```text
Developer

↓

Azure DevOps

↓

AKS

↓

Application Gateway

↓

OWASP ZAP

↓

Security Report
```

ZAP validates runtime security before production approval.

---

# Google Cloud Production Example

Architecture.

```text
Developer

↓

Cloud Build

↓

Artifact Registry

↓

Google Kubernetes Engine

↓

HTTP Load Balancer

↓

OWASP ZAP

↓

Security Report
```

Runtime validation is performed before deployment promotion.

---

# Microservices Security Testing

Example architecture.

```text
Frontend

↓

API Gateway

↓

User Service

↓

Order Service

↓

Inventory Service

↓

Payment Service
```

Each public endpoint should be tested independently to ensure complete security coverage.

---

# Common Mistakes

## Mistake 1

Scanning production systems with Active Scan.

**Impact**

Service disruption or unintended changes to production workloads.

**Recommendation**

Run Active Scans only in dedicated testing or staging environments.

---

## Mistake 2

Running only passive scans.

**Impact**

Exploitable vulnerabilities may remain undetected.

**Recommendation**

Use passive scans for every build and active scans before releases.

---

## Mistake 3

Ignoring authenticated application areas.

**Impact**

Protected functionality remains untested.

**Recommendation**

Configure authenticated users and contexts for enterprise applications.

---

## Mistake 4

Ignoring High-risk findings.

**Impact**

Applications are released with known vulnerabilities.

**Recommendation**

Treat High and Critical findings as deployment blockers.

---

## Mistake 5

Scanning without defining contexts.

**Impact**

Unrelated URLs may be scanned, increasing scan duration and false positives.

**Recommendation**

Create application-specific contexts and exclusion rules.

---

# Troubleshooting

## Scenario 1

### Unable to Reach Target

**Cause**

Incorrect URL, DNS issue, or application unavailable.

**Resolution**

```bash
curl https://test.example.com
```

Verify connectivity before running the scan.

---

## Scenario 2

### Authentication Failed

**Cause**

Incorrect login configuration or expired credentials.

**Resolution**

- Verify authentication settings.
- Confirm credentials.
- Check session cookies.
- Validate login workflow.

---

## Scenario 3

### Spider Finds No Pages

**Cause**

Application requires authentication or relies heavily on JavaScript.

**Resolution**

- Configure authentication.
- Use AJAX Spider for Single Page Applications.
- Verify the target URL.

---

## Scenario 4

### Pipeline Fails After Scan

**Cause**

High or Critical vulnerabilities detected.

**Resolution**

- Review the generated report.
- Fix identified issues.
- Re-run the scan.
- Promote only after passing security validation.

---

## Scenario 5

### Scan Duration Is Too Long

**Cause**

Large applications or broad scan scope.

**Resolution**

- Define application contexts.
- Exclude static content.
- Separate API and UI scans.
- Use targeted scan policies.

---

# Production Interview Questions

## Question 1

### What is OWASP ZAP?

**Answer**

OWASP ZAP is an open-source Dynamic Application Security Testing (DAST) tool that identifies runtime security vulnerabilities in web applications and APIs.

---

## Question 2

### What is the difference between SAST and DAST?

**Answer**

SAST analyzes source code without running the application, while DAST tests a running application by sending HTTP requests and analysing responses.

---

## Question 3

### Why is OWASP ZAP executed after deployment?

**Answer**

DAST requires a running application. Therefore, OWASP ZAP is typically executed after deployment to a testing or staging environment.

---

## Question 4

### What is the difference between a Baseline Scan and an Active Scan?

**Answer**

A Baseline Scan performs passive analysis without attacking the application. An Active Scan sends attack payloads to identify exploitable vulnerabilities.

---

## Question 5

### Can OWASP ZAP test REST APIs?

**Answer**

Yes. OWASP ZAP supports REST APIs using OpenAPI, Swagger, GraphQL, and SOAP specifications.

---

## Question 6

### Why should authenticated scanning be used?

**Answer**

Authenticated scanning validates functionality that is accessible only after login, increasing overall security coverage.

---

## Question 7

### Why should Active Scans not be executed in production?

**Answer**

Active Scans intentionally send attack payloads that may affect application availability or modify application state.

---

## Question 8

### How is OWASP ZAP integrated into CI/CD?

**Answer**

OWASP ZAP is executed after deploying the application to a testing environment. Reports are generated, archived, and used as security gates before production deployment.

---

## Question 9

### What report formats does OWASP ZAP support?

**Answer**

OWASP ZAP supports HTML, JSON, XML, Markdown, and SARIF reports for integration with dashboards and CI/CD systems.

---

## Question 10

### What are the enterprise best practices for OWASP ZAP?

**Answer**

Run baseline scans on every build, execute active scans before releases, configure authenticated scans, define application contexts, review High and Critical findings, archive reports, and integrate OWASP ZAP with the broader DevSecOps pipeline.

---

# Key Takeaways

- OWASP ZAP is a leading open-source DAST solution.
- It validates the security of running web applications and APIs.
- Execute baseline scans during every CI/CD pipeline.
- Perform active scans before production releases.
- Test authenticated application areas and APIs.
- Define application contexts to improve scan accuracy.
- Integrate with Jenkins, GitHub Actions, and GitLab CI.
- Use security reports as deployment quality gates.
- Never execute active scans directly against production systems.
- Combine OWASP ZAP with SonarQube, OWASP Dependency-Check, Veracode, Gitleaks, Trivy, Checkov, TFSec, GitOps, ArgoCD, and Amazon EKS to build a complete enterprise DevSecOps platform.