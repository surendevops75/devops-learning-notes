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

---

# Configuration

OWASP Dependency-Check can be configured using:

- Command-line arguments
- Configuration properties
- Maven plugin
- Gradle plugin
- Environment variables

Using a centralized configuration ensures consistent dependency scanning across all CI/CD pipelines.

---

# Configuration Priority

When multiple configuration methods are used, Dependency-Check follows this order.

```text
Command Line Arguments

↓

Plugin Configuration

↓

dependency-check.properties

↓

Default Configuration
```

---

# dependency-check.properties

Create a configuration file.

```bash
mkdir -p ~/.dependency-check

vi ~/.dependency-check/dependency-check.properties
```

Example configuration.

```properties
autoUpdate=true

format=HTML,JSON

failBuildOnCVSS=7

nvd.api.key=YOUR_NVD_API_KEY

data.directory=/opt/dependency-check/data

suppression.file=dependency-check-suppressions.xml
```

---

# Common Configuration Options

| Property | Purpose |
|----------|----------|
| autoUpdate | Automatically update vulnerability database |
| format | Output report format |
| failBuildOnCVSS | Pipeline failure threshold |
| nvd.api.key | NVD API authentication |
| data.directory | Local vulnerability database |
| suppression.file | Ignore approved false positives |

---

# NVD API Key

The National Vulnerability Database (NVD) provides API access for downloading vulnerability information.

Without an API key:

- Slower updates
- Rate limiting
- Longer CI/CD execution

Generate an API key from the NVD website and configure it securely.

Example.

```properties
nvd.api.key=xxxxxxxxxxxxxxxxxxxxxxxx
```

---

# Store API Keys Securely

Never store API keys inside:

- Git repositories
- Source code
- Jenkinsfiles
- Dockerfiles
- Terraform files

Use secure secret management.

Examples:

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

Dependency-Check

↓

NVD API
```

---

# Database Configuration

Dependency-Check downloads vulnerability data locally.

Default structure.

```text
dependency-check-data/

├── cache/

├── nvdcve/

├── jsrepository/

└── reports/
```

Use a shared database location on dedicated build agents to reduce repeated downloads.

Example.

```properties
data.directory=/opt/dependency-check/data
```

---

# Database Update Workflow

```text
NVD

↓

Download Updates

↓

Local Database

↓

Dependency Scan

↓

Generate Report
```

Production recommendation:

Update the database once daily rather than during every pipeline execution.

---

# Proxy Configuration

Organizations using outbound proxy servers should configure proxy settings.

Example.

```bash
export HTTP_PROXY=http://proxy.company.com:8080

export HTTPS_PROXY=http://proxy.company.com:8080

export NO_PROXY=localhost,127.0.0.1
```

Verify.

```bash
env | grep PROXY
```

---

# Suppression Files

Some findings may be accepted temporarily after security review.

Create a suppression file.

```bash
vi dependency-check-suppressions.xml
```

Example.

```xml
<?xml version="1.0"?>

<suppressions>

    <suppress>

        <cve>CVE-2024-12345</cve>

    </suppress>

</suppressions>
```

Run.

```bash
dependency-check.sh \
--suppression dependency-check-suppressions.xml \
--scan .
```

Suppress vulnerabilities only after formal approval from the security team.

---

# False Positives

Occasionally a dependency may be incorrectly matched with a CVE.

Workflow.

```text
Security Finding

↓

Developer Review

↓

Security Validation

↓

Approved Suppression

↓

Future Scans Ignore Finding
```

All suppressions should be documented and reviewed periodically.

---

# CVSS Severity Threshold

Dependency-Check uses CVSS scores to measure vulnerability severity.

| CVSS Score | Severity |
|------------|----------|
| 0.1–3.9 | Low |
| 4.0–6.9 | Medium |
| 7.0–8.9 | High |
| 9.0–10.0 | Critical |

Example.

```bash
dependency-check.sh \
--failOnCVSS 7
```

Pipeline behavior.

```text
CVSS < 7

↓

Pipeline Continues
```

```text
CVSS ≥ 7

↓

Pipeline Fails
```

---

# Report Formats

Dependency-Check supports multiple output formats.

| Format | Purpose |
|---------|----------|
| HTML | Human-readable reports |
| JSON | Automation |
| XML | Tool integration |
| CSV | Reporting |
| SARIF | GitHub Security |
| JUNIT | CI/CD testing |

Generate an HTML report.

```bash
dependency-check.sh \
--format HTML \
--scan .
```

Generate multiple reports.

```bash
dependency-check.sh \
--format ALL \
--scan .
```

---

# Report Directory

Specify an output directory.

```bash
dependency-check.sh \
--out reports \
--scan .
```

Example structure.

```text
reports/

├── dependency-check-report.html

├── dependency-check-report.json

├── dependency-check-report.xml

└── dependency-check-report.sarif
```

---

# Cache Management

Dependency-Check stores downloaded vulnerability data locally.

View cache size.

```bash
du -sh /opt/dependency-check/data
```

Clean old cache.

```bash
rm -rf /opt/dependency-check/data/*
```

Rebuild the database.

```bash
dependency-check.sh --updateonly
```

Avoid deleting the cache before every pipeline execution, as this significantly increases scan time.

---

# Scan Performance

Large enterprise applications may contain thousands of dependencies.

Improve performance by:

- Sharing the vulnerability database across build agents.
- Updating the NVD database outside CI/CD.
- Scanning only the required project directory.
- Excluding build artifacts and temporary files.

---

# Enterprise Best Practices

- Use a centralized `dependency-check.properties` file.
- Configure an NVD API key to avoid rate limiting.
- Update the vulnerability database daily.
- Fail builds for High and Critical CVSS scores.
- Store API keys in an enterprise secrets manager.
- Use suppression files only after security approval.
- Review suppressed CVEs regularly.
- Generate SARIF reports for GitHub Security integration.
- Archive reports for compliance and auditing.
- Keep Dependency-Check updated to the latest stable release.

---

# Maven Integration

OWASP Dependency-Check provides an official Maven plugin that scans project dependencies during the Maven build lifecycle.

## Architecture

```text
Developer

↓

Git Push

↓

Jenkins

↓

Maven Build

↓

Dependency-Check Plugin

↓

NVD Database

↓

Security Report

↓

PASS / FAIL
```

---

# Configure Maven Plugin

Add the plugin to your `pom.xml`.

```xml
<build>

    <plugins>

        <plugin>

            <groupId>org.owasp</groupId>

            <artifactId>dependency-check-maven</artifactId>

            <version>12.x.x</version>

            <configuration>

                <failBuildOnCVSS>7</failBuildOnCVSS>

                <formats>

                    <format>HTML</format>

                    <format>JSON</format>

                </formats>

            </configuration>

            <executions>

                <execution>

                    <goals>

                        <goal>check</goal>

                    </goals>

                </execution>

            </executions>

        </plugin>

    </plugins>

</build>
```

---

# Execute Maven Scan

```bash
mvn clean verify
```

Or execute only Dependency-Check.

```bash
mvn org.owasp:dependency-check-maven:check
```

---

# Gradle Integration

Dependency-Check also provides an official Gradle plugin.

Example configuration.

```groovy
plugins {

    id "org.owasp.dependencycheck" version "12.x.x"

}
```

Configure.

```groovy
dependencyCheck {

    failBuildOnCVSS = 7

    formats = ['HTML','JSON']

}
```

Run.

```bash
gradle dependencyCheckAnalyze
```

---

# Ant Integration

Dependency-Check supports Apache Ant builds.

Example.

```xml
<taskdef

resource="dependencycheck.properties"

classpath="dependency-check-ant.jar"

/>

<dependency-check

scanSet="."

format="HTML"

/>
```

Execute.

```bash
ant dependency-check
```

---

# Standalone CLI Scanning

CLI mode is useful for projects that do not use Maven or Gradle.

Example.

```bash
dependency-check.sh \
--project payment-service \
--scan .
```

Specify report location.

```bash
dependency-check.sh \
--scan . \
--out reports
```

Generate all report formats.

```bash
dependency-check.sh \
--scan . \
--format ALL
```

---

# Scanning Multiple Projects

Enterprise repositories often contain multiple services.

Example.

```text
microservices/

├── user-service

├── payment-service

├── cart-service

├── order-service

└── notification-service
```

Scan the entire repository.

```bash
dependency-check.sh \
--scan microservices
```

Each dependency is analysed independently before generating a consolidated report.

---

# Excluding Directories

Exclude unnecessary directories to improve scan speed.

Example.

```bash
dependency-check.sh \
--scan . \
--exclude "**/node_modules/**" \
--exclude "**/target/**"
```

Common exclusions:

- node_modules
- build
- target
- dist
- vendor
- .git

---

# Docker Image Dependency Scan

Dependency-Check primarily scans application dependencies rather than operating system packages.

Typical workflow.

```text
Application Source

↓

Dependency-Check

↓

Docker Build

↓

Trivy Image Scan

↓

Deploy
```

Recommendation:

- Dependency-Check → Application dependencies
- Trivy → Container operating system and application packages

Using both tools provides broader coverage.

---

# SBOM Integration

Dependency-Check complements Software Bill of Materials (SBOM) generation.

Workflow.

```text
Application

↓

Dependency Analysis

↓

Generate SBOM

↓

Verify Components

↓

Compliance
```

SBOMs improve:

- Software supply chain visibility
- Compliance
- Vulnerability management
- License tracking

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

Checkout

↓

Build

↓

Dependency-Check

↓

HTML Report

↓

PASS / FAIL
```

Install the **OWASP Dependency-Check** plugin in Jenkins.

Configure:

- Installation path
- Report directory
- Publisher
- Build failure threshold

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

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

        stage('Dependency Scan') {

            steps {

                sh '''

                dependency-check.sh \
                --scan . \
                --format HTML \
                --format JSON \
                --failOnCVSS 7 \
                --out reports

                '''

            }

        }

    }

    post {

        always {

            archiveArtifacts 'reports/*'

        }

    }

}
```

---

# GitHub Actions Integration

Store any required secrets (such as an NVD API key) in GitHub Secrets.

Example:

```text
NVD_API_KEY
```

---

# Production GitHub Actions Workflow

```yaml
name: Dependency Check

on:

  push:

    branches:

      - main

jobs:

  dependency-check:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - name: Build

        run: mvn clean package

      - name: Dependency Scan

        env:

          NVD_API_KEY: ${{ secrets.NVD_API_KEY }}

        run: |

          dependency-check.sh \
          --scan . \
          --failOnCVSS 7 \
          --format HTML \
          --format JSON \
          --out reports

      - name: Upload Reports

        uses: actions/upload-artifact@v4

        with:

          name: dependency-check-report

          path: reports
```

---

# GitLab CI Integration

```yaml
dependency_check:

  image: eclipse-temurin:17

  script:

    - mvn clean package

    - dependency-check.sh \
      --scan . \
      --failOnCVSS 7 \
      --format HTML \
      --format JSON \
      --out reports

  artifacts:

    paths:

      - reports/
```

---

# Integrating with SonarQube

A common enterprise workflow combines SonarQube and Dependency-Check.

```text
Build

↓

SonarQube

↓

Code Quality Gate

↓

Dependency-Check

↓

Open Source Vulnerabilities

↓

Veracode

↓

Container Build
```

SonarQube analyses source code quality, while Dependency-Check focuses on third-party dependency vulnerabilities.

---

# Integrating with Veracode

Many organisations run both tools together.

```text
Dependency-Check

↓

Known CVEs

↓

Veracode SAST

↓

Application Vulnerabilities

↓

Security Policy

↓

Deploy
```

This combination provides broader application security coverage.

---

# Integrating with Trivy

```text
Dependency-Check

↓

Application Libraries

↓

Docker Build

↓

Trivy

↓

Operating System Packages

↓

Container Security
```

Using both tools helps detect vulnerabilities before and after containerisation.

---

# Enterprise DevSecOps Pipeline

OWASP Dependency-Check becomes part of the software supply chain security stage in an enterprise CI/CD pipeline.

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
          OWASP Dependency-Check Scan
                         │
                  ┌──────┴──────┐
                  │             │
                PASS          FAIL
                  │             │
                  ▼             ▼
             Veracode SAST   Stop Pipeline
                  │
                  ▼
            Docker Image Build
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

# Understanding the Scan Report

A typical report contains:

- Project information
- Dependency inventory
- Vulnerability summary
- CVE identifiers
- CVSS scores
- Vulnerable versions
- Fixed versions
- References
- Report generation time

Example.

```text
Application

↓

152 Dependencies

↓

8 Vulnerabilities

↓

HTML Report

↓

Developer Review
```

---

# Understanding CVSS Scores

Dependency-Check uses the Common Vulnerability Scoring System (CVSS) to measure vulnerability severity.

| CVSS | Severity | Recommended Action |
|------|----------|--------------------|
| 0.1 – 3.9 | Low | Monitor |
| 4.0 – 6.9 | Medium | Plan remediation |
| 7.0 – 8.9 | High | Fix before release |
| 9.0 – 10.0 | Critical | Immediate remediation |

---

# Sample Vulnerability Report

| Dependency | CVE | Severity | Status |
|------------|-----|----------|--------|
| log4j-core | CVE-2021-44228 | Critical | Upgrade Required |
| jackson-databind | CVE-2022-42003 | High | Upgrade Required |
| commons-text | CVE-2022-42889 | High | Upgrade Required |
| spring-core | None | Safe | Continue |

---

# Vulnerability Remediation Workflow

```text
Dependency Scan

↓

Vulnerability Found

↓

Developer Review

↓

Upgrade Dependency

↓

Rebuild Application

↓

Re-scan

↓

PASS
```

Dependency upgrades should be tested before deployment to ensure compatibility.

---

# Managing False Positives

Occasionally, a reported vulnerability may not apply to your application.

Recommended workflow.

```text
Finding

↓

Developer Review

↓

Security Team Validation

↓

Approved

↓

Suppression File

↓

Future Scans
```

Every suppression should include:

- Business justification
- Security approval
- Expiry date
- Review schedule

---

# Dashboards

## Developer Dashboard

Shows:

- Vulnerable libraries
- Dependency tree
- CVSS scores
- Upgrade recommendations

---

## Security Dashboard

Shows:

- Total vulnerable dependencies
- Critical vulnerabilities
- Open remediation tasks
- Policy compliance
- Risk trends

---

## Executive Dashboard

Provides:

- Overall software supply chain risk
- Compliance metrics
- Application security posture
- Remediation progress
- Historical trends

---

# Enterprise Best Practices

## Dependency Management

- Keep dependencies updated.
- Remove unused libraries.
- Pin dependency versions.
- Use trusted package repositories.
- Review dependency changes during code review.

---

## CI/CD

- Scan every Pull Request.
- Scan every release build.
- Fail builds for High and Critical vulnerabilities.
- Archive reports for auditing.
- Integrate reports with security dashboards.

---

## Security

- Enable NVD API authentication.
- Update vulnerability databases daily.
- Use suppression files only after approval.
- Protect CI/CD credentials.
- Monitor newly disclosed CVEs.

---

## Performance

- Share vulnerability databases across build agents.
- Exclude unnecessary directories.
- Scan only required modules.
- Schedule database updates outside build hours.

---

# Common Mistakes

## Mistake 1

Running scans only before production releases.

Correct approach.

```text
Every Pull Request

↓

Dependency Scan

↓

Fix Vulnerabilities

↓

Merge
```

---

## Mistake 2

Ignoring Medium vulnerabilities.

Medium vulnerabilities can become High or Critical as new exploit techniques emerge.

Regular reviews reduce long-term risk.

---

## Mistake 3

Using outdated vulnerability databases.

```text
Old Database

↓

Missed CVEs

↓

Insecure Release
```

Always update the database before relying on scan results.

---

## Mistake 4

Scanning only application code.

Dependency-Check focuses on third-party libraries.

Combine it with:

- SonarQube
- Veracode
- Trivy
- SBOM generation

to achieve comprehensive application security.

---

## Mistake 5

Ignoring transitive dependencies.

Many vulnerabilities originate from indirect dependencies.

```text
Application

↓

Spring Boot

↓

Library A

↓

Library B

↓

Critical CVE
```

Review the full dependency tree rather than only direct dependencies.

---

# Common Troubleshooting

## Issue 1

### NVD Update Failed

**Cause**

Internet connectivity or API rate limiting.

**Resolution**

```text
Verify Network

↓

Verify NVD API Key

↓

Retry Database Update
```

---

## Issue 2

### Scan Takes Too Long

**Cause**

Large projects or first-time database download.

**Resolution**

- Use a shared database.
- Update the database daily.
- Exclude unnecessary directories.

---

## Issue 3

### Build Failed Due to CVSS Threshold

**Cause**

Detected vulnerability exceeds the configured threshold.

**Resolution**

```text
Review Report

↓

Upgrade Dependency

↓

Rebuild

↓

Re-run Scan
```

---

## Issue 4

### False Positive Detected

**Cause**

Incorrect vulnerability matching.

**Resolution**

```text
Security Review

↓

Approve Suppression

↓

Update Suppression File
```

---

## Issue 5

### Report Not Generated

**Cause**

Invalid output directory or scan failure.

**Resolution**

- Verify output directory permissions.
- Check Dependency-Check logs.
- Confirm the scan completed successfully.

---

# Production Interview Questions

## Question 1

### What is OWASP Dependency-Check?

**Answer**

OWASP Dependency-Check is an open-source Software Composition Analysis (SCA) tool that identifies known vulnerabilities in third-party libraries by comparing project dependencies with public vulnerability databases such as the National Vulnerability Database (NVD).

---

## Question 2

### What is the difference between Dependency-Check and Trivy?

**Answer**

Dependency-Check focuses on application dependencies and open-source libraries, while Trivy scans container images, operating system packages, Kubernetes resources, Infrastructure as Code, secrets, and SBOMs.

---

## Question 3

### Why is an NVD API key recommended?

**Answer**

An NVD API key increases update speed, reduces rate limiting, and improves reliability when downloading vulnerability data during automated scans.

---

## Question 4

### What is a CVSS score?

**Answer**

CVSS (Common Vulnerability Scoring System) is a standardized method for measuring the severity of software vulnerabilities, helping organizations prioritize remediation efforts.

---

## Question 5

### Why should builds fail on High or Critical vulnerabilities?

**Answer**

Blocking builds prevents vulnerable software from progressing through the CI/CD pipeline, reducing the risk of deploying exploitable applications.

---

## Question 6

### What are transitive dependencies?

**Answer**

Transitive dependencies are libraries indirectly included through another dependency. They can introduce vulnerabilities even if they are not explicitly declared by the application.

---

## Question 7

### Why are suppression files used?

**Answer**

Suppression files exclude approved false positives or accepted risks from future scans after review and authorization by the security team.

---

## Question 8

### How does Dependency-Check integrate with CI/CD?

**Answer**

It integrates through Maven, Gradle, CLI, Jenkins, GitHub Actions, GitLab CI, and other automation tools to perform dependency scanning during every build.

---

## Question 9

### Can Dependency-Check replace Veracode or SonarQube?

**Answer**

No. Dependency-Check specializes in Software Composition Analysis. SonarQube focuses on code quality and static analysis, while Veracode provides enterprise application security testing and governance. These tools complement each other.

---

## Question 10

### What are the enterprise best practices for Dependency-Check?

**Answer**

Run scans on every Pull Request and release build, maintain an up-to-date vulnerability database, fail builds for High and Critical vulnerabilities, secure API keys, review suppression files regularly, and combine Dependency-Check with SonarQube, Veracode, Trivy, SBOM generation, and image signing.

---

# Key Takeaways

- OWASP Dependency-Check is an enterprise-grade Software Composition Analysis (SCA) tool.
- It detects vulnerable third-party libraries before software is released.
- Integrate scans into every CI/CD pipeline to identify risks early.
- Keep the NVD database updated for accurate vulnerability detection.
- Use CVSS thresholds to automate security policy enforcement.
- Combine Dependency-Check with SonarQube, Veracode, Trivy, SBOM generation, and Cosign for layered software supply chain security.
- Review suppression files regularly and document all accepted risks.
- Continuously monitor dependency updates to reduce long-term exposure to newly disclosed vulnerabilities.