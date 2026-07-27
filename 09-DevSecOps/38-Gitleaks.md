# Gitleaks

## Introduction

Gitleaks is an open-source secret detection tool used to identify sensitive information such as API keys, passwords, access tokens, SSH keys, cloud credentials, and certificates within Git repositories.

It helps organizations prevent accidental exposure of secrets by scanning source code, Git history, commits, branches, and pull requests before code reaches production.

Gitleaks is widely adopted in DevSecOps pipelines as part of Shift-Left Security to stop secret leaks early in the software development lifecycle.

---

# Why Companies Use Gitleaks

One leaked credential can lead to:

- Cloud account compromise
- Data breaches
- Unauthorized infrastructure access
- Supply chain attacks
- Compliance violations

Gitleaks detects secrets before they become security incidents.

## Benefits

- Git repository scanning
- Git history scanning
- Commit scanning
- Pull Request scanning
- Secret detection
- API key detection
- Cloud credential detection
- CI/CD integration
- Custom detection rules
- Compliance support

---

# What Can Gitleaks Detect?

| Secret Type | Supported |
|--------------|-----------|
| AWS Access Keys | ✓ |
| AWS Secret Keys | ✓ |
| Azure Credentials | ✓ |
| GCP Service Account Keys | ✓ |
| GitHub Tokens | ✓ |
| GitLab Tokens | ✓ |
| Docker Hub Tokens | ✓ |
| Slack Tokens | ✓ |
| Kubernetes Secrets | ✓ |
| SSH Private Keys | ✓ |
| RSA Keys | ✓ |
| JWT Tokens | ✓ |
| API Keys | ✓ |
| Passwords | ✓ |
| Database Credentials | ✓ |

---

# Where Gitleaks Fits in DevSecOps

Secrets should be detected before the application is built or deployed.

```text
Developer

↓

Git Commit

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

Checkout Source

↓

Gitleaks Scan

↓

PASS / FAIL

↓

Build

↓

Security Scans

↓

Deploy
```

A pipeline should stop immediately if secrets are detected.

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
              Gitleaks Scan
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
     Source Code   Git History   Commits
                      │
                      ▼
             Secret Detection
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

Pull Request

↓

Jenkins

↓

Checkout

↓

Gitleaks

↓

Secret Validation

↓

Build

↓

Deploy

↓

Production
```

No build should proceed when exposed secrets are detected.

---

# Common Secret Types

Examples of secrets frequently found in repositories.

```text
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

GITHUB_TOKEN

DOCKER_PASSWORD

DATABASE_PASSWORD

JWT_SECRET

PRIVATE_KEY

KUBECONFIG

SSH_PRIVATE_KEY

SLACK_WEBHOOK
```

These values should never be committed to Git.

---

# Prerequisites

| Component | Version |
|------------|----------|
| Ubuntu | 22.04 LTS |
| Git | Latest |
| Docker | Latest |
| Jenkins | Latest |
| GitHub Actions | Supported |
| GitLab CI | Supported |

---

# Installation Methods

Gitleaks can be installed using:

- Binary
- Homebrew
- Docker
- GitHub Actions
- Jenkins
- GitLab CI

---

# Install on Ubuntu

Download the latest release.

```bash
wget https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks_linux_x64.tar.gz
```

Extract.

```bash
tar -xzf gitleaks_linux_x64.tar.gz
```

Move binary.

```bash
sudo mv gitleaks /usr/local/bin/
```

Verify.

```bash
gitleaks version
```

---

# Install Using Docker

Pull the image.

```bash
docker pull zricethezav/gitleaks:latest
```

Run a scan.

```bash
docker run --rm \
-v $(pwd):/repo \
zricethezav/gitleaks:latest \
detect \
--source=/repo
```

---

# Install on Jenkins Agent

Connect to the Jenkins agent.

```bash
ssh jenkins@agent01
```

Verify installation.

```bash
gitleaks version
```

Install if necessary.

```bash
sudo mv gitleaks /usr/local/bin/
```

---

# Install on GitHub Actions Runner

GitHub-hosted runners can install Gitleaks during workflow execution.

Example.

```yaml
- name: Install Gitleaks

  run: |

    wget https://github.com/gitleaks/gitleaks/releases/latest/download/gitleaks_linux_x64.tar.gz

    tar -xzf gitleaks_linux_x64.tar.gz

    sudo mv gitleaks /usr/local/bin/
```

Self-hosted runners should have Gitleaks pre-installed and updated regularly.

---

# Verify Installation

Check the installed version.

```bash
gitleaks version
```

Example output.

```text
Version

↓

8.x.x
```

---

# First Repository Scan

Scan the current repository.

```bash
gitleaks detect \
--source .
```

Example output.

```text
Repository

↓

Scanning Files

↓

Secrets Found

↓

Report Generated
```

Example summary.

```text
Files Scanned : 243

Commits Scanned : 1,582

Secrets Found : 2

Result : FAILED
```

Every detected secret includes:

- Rule ID
- Secret type
- File name
- Commit ID
- Line number
- Severity
- Remediation guidance

This enables developers to remove or rotate exposed credentials before merging code into the main branch.

---

# Scanning Source Code

Gitleaks scans source code repositories for exposed secrets before code is built or deployed.

Project structure.

```text
application/

├── src/

├── config/

├── scripts/

├── Dockerfile

├── Jenkinsfile

└── .github/
```

Run.

```bash
gitleaks detect \
--source .
```

Workflow.

```text
Repository

↓

Read Files

↓

Apply Detection Rules

↓

Secrets Found

↓

Security Report
```

---

# Scanning Git History

Secrets removed from the latest code may still exist in previous commits.

Scan complete Git history.

```bash
gitleaks git
```

Workflow.

```text
Repository

↓

Git History

↓

All Commits

↓

Secret Detection

↓

Report
```

Git history should always be scanned before open-sourcing a repository.

---

# Scanning a Specific Commit

Scan a specific commit range.

```bash
gitleaks git \
--log-opts="-1"
```

Last five commits.

```bash
gitleaks git \
--log-opts="-5"
```

Specific branch.

```bash
gitleaks git \
--log-opts="main"
```

---

# Scanning Pull Requests

Every Pull Request should be scanned before merging.

```text
Developer

↓

Pull Request

↓

Gitleaks

↓

Secret Detection

↓

PASS / FAIL

↓

Merge
```

This prevents accidental exposure of secrets in the main branch.

---

# Pre-Commit Scanning

Developers can prevent secrets from being committed using Git hooks.

Install pre-commit.

```bash
pip install pre-commit
```

Example configuration.

```yaml
repos:

- repo: https://github.com/gitleaks/gitleaks

  rev: v8.24.0

  hooks:

    - id: gitleaks
```

Install hooks.

```bash
pre-commit install
```

Workflow.

```text
Developer

↓

Git Commit

↓

Pre-Commit Hook

↓

Gitleaks

↓

PASS

↓

Commit
```

---

# Scanning Dockerfiles

Secrets are sometimes hardcoded in Dockerfiles.

Example.

```Dockerfile
ENV AWS_SECRET_ACCESS_KEY=ABC123456789
```

Run.

```bash
gitleaks detect \
--source .
```

Finding.

```text
AWS Secret Key

↓

FAILED
```

Use build-time secrets instead of embedding credentials.

---

# Scanning Kubernetes Manifests

Secrets should never be stored directly in Kubernetes YAML files.

Example.

```yaml
data:

  password: admin123
```

Finding.

```text
Potential Password

↓

FAILED
```

Use Kubernetes Secrets or an external secret management solution.

---

# Scanning Terraform Projects

Terraform code frequently contains provider credentials.

Project.

```text
terraform/

├── provider.tf

├── main.tf

├── variables.tf

└── terraform.tfvars
```

Run.

```bash
gitleaks detect \
--source terraform/
```

Typical findings.

- AWS Access Keys
- Azure Credentials
- Google Cloud Keys
- Terraform Cloud Tokens

---

# Scanning Environment Files

Environment files commonly contain sensitive information.

Example.

```text
.env

.env.production

.env.development
```

Run.

```bash
gitleaks detect \
--source .
```

Possible findings.

```text
DATABASE_PASSWORD

JWT_SECRET

API_KEY
```

Environment files containing secrets should not be committed.

---

# Scanning Shell Scripts

Example.

```bash
export AWS_SECRET_ACCESS_KEY=ABC123
```

Run.

```bash
gitleaks detect \
--source scripts/
```

Finding.

```text
AWS Secret Key

↓

FAILED
```

---

# Scanning Configuration Files

Supported examples.

```text
application.yml

application.properties

config.json

settings.xml

values.yaml
```

Hardcoded passwords and tokens should be removed before committing configuration files.

---

# Scanning Monorepositories

Enterprise repositories often contain multiple applications.

Example.

```text
Repository

├── backend/

├── frontend/

├── terraform/

├── kubernetes/

├── scripts/

└── shared/
```

Run.

```bash
gitleaks detect \
--source .
```

Every project inside the repository is scanned.

---

# Secret Detection Workflow

```text
Repository

↓

Files

↓

Regular Expressions

↓

Entropy Analysis

↓

Secret Validation

↓

PASS / FAIL
```

---

# Types of Findings

Examples.

| Secret | Example |
|----------|----------|
| AWS Key | AKIA... |
| GitHub Token | ghp_xxxxx |
| GitLab Token | glpat_xxxxx |
| Slack Token | xoxb-xxxxx |
| JWT | eyJhbGciOi... |
| SSH Private Key | -----BEGIN OPENSSH PRIVATE KEY----- |
| RSA Private Key | -----BEGIN RSA PRIVATE KEY----- |
| API Key | api_xxxxxxxxx |

---

# Enterprise Best Practices

- Scan every commit before merging.
- Enable pre-commit hooks for all developers.
- Scan Git history before making repositories public.
- Never store secrets in Terraform files.
- Never commit `.env` files containing production credentials.
- Use secret management solutions instead of hardcoded values.
- Rotate exposed credentials immediately.
- Scan monorepositories from the repository root.
- Include Gitleaks in every CI/CD pipeline.
- Archive scan reports for compliance and auditing.

---

