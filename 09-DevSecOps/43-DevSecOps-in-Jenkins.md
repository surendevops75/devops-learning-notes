# DevSecOps in Jenkins

## Introduction

Jenkins is one of the most widely adopted Continuous Integration and Continuous Delivery (CI/CD) platforms used to automate software builds, testing, security validation, deployments, and release management.

DevSecOps extends Jenkins by integrating automated security checks throughout the CI/CD pipeline, ensuring vulnerabilities are detected and remediated before software reaches production.

Instead of treating security as a final review, Jenkins executes security controls alongside every build, making security an integral part of software delivery.

---

# Why Companies Use Jenkins for DevSecOps

Enterprise organizations require automated, repeatable, and secure delivery pipelines.

Jenkins enables security testing to become part of every code change.

## Benefits

- Automated CI/CD
- Shift-Left Security
- Secure Software Supply Chain
- Policy Enforcement
- Infrastructure Automation
- Security Gate Automation
- Compliance Validation
- Artifact Traceability
- GitOps Integration
- Enterprise Scalability

---

# DevOps vs DevSecOps Pipeline

## Traditional DevOps

```text
Developer

↓

Git Push

↓

Build

↓

Test

↓

Docker Build

↓

Deploy

↓

Production

↓

Security Testing
```

Security occurs after deployment.

---

## DevSecOps

```text
Developer

↓

Git Push

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Dependency Scan

↓

Secret Scan

↓

IaC Scan

↓

Container Scan

↓

SBOM

↓

Image Signing

↓

Deploy

↓

Production
```

Security is integrated into every pipeline stage.

---

# Where Jenkins Fits in DevSecOps

```text
Developer

↓

Feature Branch

↓

Git Push

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

Security Pipeline

↓

Artifact Repository

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

Jenkins orchestrates the complete software delivery workflow.

---

# Enterprise DevSecOps Architecture

```text
                 Developers
                      │
                      ▼
              GitHub Enterprise
                      │
                Webhook Trigger
                      │
                      ▼
                  Jenkins
                      │
      ┌───────────────┼────────────────┐
      ▼               ▼                ▼
 Source Analysis  Dependency Scan  Secret Scan
      │               │                │
      ▼               ▼                ▼
 SonarQube     OWASP Dependency   Gitleaks
                     Check
      │               │                │
      └───────────────┼────────────────┘
                      ▼
               Docker Build
                      │
                      ▼
               Container Scan
                      │
                      ▼
                  Trivy Scan
                      │
                      ▼
                  Generate SBOM
                      │
                      ▼
                 Sign Container
                      │
                      ▼
                Amazon ECR
                      │
                      ▼
                  GitOps Repo
                      │
                      ▼
                    ArgoCD
                      │
                      ▼
                  Amazon EKS
                      │
                      ▼
               Runtime Security
          Falco / Aqua / Prisma
```

---

# Enterprise Production Pipeline

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

Jenkins Trigger

↓

Checkout

↓

Build

↓

Unit Tests

↓

Code Coverage

↓

SonarQube

↓

OWASP Dependency-Check

↓

Gitleaks

↓

Checkov

↓

TFSec

↓

Docker Build

↓

Trivy

↓

SBOM

↓

Cosign Image Signing

↓

Amazon ECR

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Falco Runtime Monitoring

↓

Production
```

Every deployment passes multiple automated security gates.

---

# Why Security Is Added at Every Stage

| Stage | Security Control |
|---------|------------------|
| Source Code | SonarQube |
| Dependencies | OWASP Dependency-Check |
| Secrets | Gitleaks |
| Terraform | Checkov |
| Terraform | TFSec |
| Container | Trivy |
| Supply Chain | SBOM |
| Image Integrity | Cosign |
| Deployment | ArgoCD |
| Runtime | Falco / Aqua / Prisma |

Security becomes continuous rather than periodic.

---

# Prerequisites

Before building an enterprise DevSecOps pipeline, ensure the following components are available.

| Component | Purpose |
|-----------|----------|
| Jenkins LTS | CI/CD Platform |
| GitHub | Source Control |
| Docker | Container Build |
| Kubernetes | Deployment Platform |
| Amazon EKS | Container Orchestration |
| Helm | Kubernetes Package Manager |
| SonarQube | Code Quality |
| OWASP Dependency-Check | Dependency Security |
| Gitleaks | Secret Detection |
| Checkov | IaC Security |
| TFSec | Terraform Security |
| Trivy | Container Security |
| Cosign | Image Signing |
| ArgoCD | GitOps Deployment |

---

# Jenkins Installation

Ubuntu installation.

```bash
sudo apt update

sudo apt install openjdk-21-jdk -y

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
/usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/ | \
sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update

sudo apt install jenkins -y
```

---

# Verify Jenkins Installation

```bash
sudo systemctl status jenkins

sudo systemctl enable jenkins

sudo systemctl start jenkins
```

Verify the service.

```bash
systemctl status jenkins
```

Expected output.

```text
Active: active (running)
```

---

# Jenkins Directory Structure

```text
/var/lib/jenkins/

├── jobs/

├── workspace/

├── plugins/

├── secrets/

├── nodes/

├── logs/

└── users/
```

Understanding the directory structure helps with troubleshooting and backup strategies.

---

# Unlock Jenkins

Retrieve the initial administrator password.

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Use this password to complete the web-based setup wizard.

---

# Install Recommended Plugins

Core plugins for enterprise DevSecOps.

- Git
- Pipeline
- Docker Pipeline
- Credentials
- Kubernetes
- Blue Ocean
- GitHub
- SSH Agent
- Configuration as Code
- Role Strategy
- Matrix Authorization
- Job DSL

These plugins provide the foundation for secure and scalable CI/CD pipelines.