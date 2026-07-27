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