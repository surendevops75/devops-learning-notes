# OWASP Dependency-Check

## Introduction

OWASP Dependency-Check is an open-source Software Composition Analysis (SCA) tool that identifies publicly disclosed vulnerabilities in third-party libraries used by applications.

It compares project dependencies against the National Vulnerability Database (NVD) and other vulnerability data sources to detect known Common Vulnerabilities and Exposures (CVEs).

Dependency-Check is widely used in DevSecOps pipelines to prevent applications from being released with vulnerable open-source components.

---

# Why Companies Use OWASP Dependency-Check

Modern applications depend heavily on third-party libraries.

A single vulnerable dependency can expose an application to severe security risks.

Dependency-Check helps organizations detect these vulnerabilities early during the build process.

## Benefits

- Detect vulnerable open-source libraries
- Identify CVEs
- Generate detailed security reports
- Support multiple programming languages
- Integrate with CI/CD
- Support compliance requirements
- Reduce software supply chain risks
- Automate dependency security checks

---

# Where OWASP Dependency-Check Fits in DevSecOps

Dependency-Check runs immediately after the application is built and before packaging or deployment.

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

Compile Application

↓

Unit Tests

↓

SonarQube Analysis

↓

Dependency-Check Scan

↓

Policy Decision

↓

Package Application

↓

Veracode

↓

Docker Build

↓

Trivy Image Scan

↓

SBOM

↓

Cosign

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

If vulnerable dependencies exceed organizational thresholds, the pipeline should stop immediately.

---

# What Can Dependency-Check Scan?

Dependency-Check supports multiple project types.

| Project Type | Supported |
|--------------|-----------|
| Java (Maven) | ✓ |
| Java (Gradle) | ✓ |
| .NET | ✓ |
| Node.js | ✓ |
| Python | ✓ |
| Ruby | ✓ |
| Go | Partial |
| PHP | ✓ |
| Standalone CLI | ✓ |

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
                 Build Application
                         │
                         ▼
             Dependency-Check Scan
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
   Project Files   Dependency List     Lock Files
        │                │                │
        └────────────────┼────────────────┘
                         ▼
               Vulnerability Database
                         │
                         ▼
                    CVE Matching
                         │
                         ▼
                  Security Report
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

Build Agent

↓

Dependency-Check

↓

NVD Database

↓

Security Report

↓

Veracode

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

Before installing Dependency-Check, ensure the following components are available.

| Component | Version |
|------------|----------|
| Ubuntu | 22.04 LTS |
| Java | 17+ |
| Maven | Latest |
| Gradle | Latest |
| Jenkins | Latest |
| Git | Latest |
| Internet Access | Required |

---

# Supported Data Sources

Dependency-Check retrieves vulnerability information from multiple sources.

Primary sources include:

- National Vulnerability Database (NVD)
- CVE Database
- CISA Known Exploited Vulnerabilities (KEV)
- OSS Index (Optional)

The NVD is the primary source for CVE information.

---

# Installation Methods

Dependency-Check can be installed using:

- Standalone CLI
- Docker
- Maven Plugin
- Gradle Plugin
- Jenkins Plugin

---

# Install on Ubuntu

## Step 1 — Update Packages

```bash
sudo apt update

sudo apt upgrade -y
```

---

## Step 2 — Install Java

```bash
sudo apt install openjdk-17-jdk -y
```

Verify.

```bash
java -version
```

Expected output.

```text
openjdk version "17"
```

---

## Step 3 — Download Dependency-Check

Download the latest release from the official OWASP GitHub repository.

Example.

```bash
wget https://github.com/dependency-check/DependencyCheck/releases/download/v12.x.x/dependency-check-12.x.x-release.zip
```

Extract.

```bash
unzip dependency-check-*.zip
```

Rename.

```bash
mv dependency-check dependency-check-cli
```

---

## Step 4 — Add to PATH

```bash
export PATH=$PATH:$HOME/dependency-check-cli/bin
```

Verify installation.

```bash
dependency-check.sh --version
```

Expected output.

```text
Dependency-Check 12.x.x
```

---

# Install Using Docker

Pull the official image.

```bash
docker pull owasp/dependency-check:latest
```

Verify.

```bash
docker images
```

Example scan.

```bash
docker run --rm \
-v $(pwd):/src \
owasp/dependency-check:latest \
--scan /src
```

---

# Install on Jenkins Agent

SSH into the Jenkins build agent.

```bash
ssh jenkins@agent01
```

Install Dependency-Check using the Ubuntu installation steps.

Verify.

```bash
dependency-check.sh --version
```

The Jenkins pipeline can now perform dependency analysis during builds.

---

# GitHub Actions Runners

GitHub-hosted runners can install Dependency-Check dynamically.

Example.

```yaml
- name: Install Dependency-Check

  run: |

    wget https://github.com/dependency-check/DependencyCheck/releases/latest/download/dependency-check.zip

    unzip dependency-check.zip
```

For self-hosted runners, install Dependency-Check once and keep it updated.

---

# National Vulnerability Database (NVD)

Dependency-Check downloads vulnerability data from the National Vulnerability Database.

```text
Application Dependencies

↓

Dependency-Check

↓

NVD Database

↓

CVE Matching

↓

Security Report
```

The first execution downloads the latest vulnerability database.

Subsequent scans update only changed records.

---

# Update the Vulnerability Database

Download the latest CVE database.

```bash
dependency-check.sh --updateonly
```

Production recommendation:

Schedule a daily update.

Example.

```text
0 1 * * * dependency-check.sh --updateonly
```

This keeps scan results accurate while reducing CI/CD execution time.

---

# Verify Installation

Run a scan against the current project.

```bash
dependency-check.sh \
--scan .
```

Example output.

```text
Analysis Started

↓

Dependency Collection

↓

CVE Matching

↓

Report Generated
```

---

# First Dependency Scan

Scan a Maven project.

```bash
dependency-check.sh \
--project payment-service \
--scan .
```

Example output.

```text
Dependencies Scanned : 152

Vulnerabilities Found : 5

High : 1

Medium : 3

Low : 1
```

The generated report helps developers identify vulnerable libraries before the application proceeds to the next stages of the DevSecOps pipeline.