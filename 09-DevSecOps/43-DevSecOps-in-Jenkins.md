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

---

# Required Jenkins Plugins

Enterprise DevSecOps pipelines require additional plugins beyond the default installation.

| Plugin | Purpose |
|---------|----------|
| Git | Source Code Management |
| GitHub | GitHub Integration |
| Pipeline | Jenkins Pipelines |
| Docker Pipeline | Docker Build Support |
| Credentials Binding | Secure Credential Management |
| Kubernetes | Kubernetes Integration |
| Blue Ocean | Pipeline Visualization |
| SSH Agent | SSH Authentication |
| Configuration as Code | Jenkins Automation |
| Role Strategy | RBAC |
| Matrix Authorization | Permission Management |

---

# Install Security Tools

Install the required security tools on the Jenkins server.

```text
SonarQube

OWASP Dependency-Check

Gitleaks

Checkov

TFSec

Trivy

Cosign

Docker

kubectl

Helm
```

All tools should be available in the Jenkins agent PATH.

---

# Jenkins Credentials Management

Never store passwords inside Jenkinsfiles.

Store credentials securely in Jenkins Credentials Store.

Examples.

```text
GitHub Token

Docker Registry Credentials

AWS Access Keys

SonarQube Token

Cosign Private Key

SSH Private Key

Kubeconfig

GitHub PAT
```

---

# Credential Types

| Type | Example |
|------|----------|
| Username & Password | Docker Registry |
| Secret Text | API Token |
| Secret File | kubeconfig |
| SSH Username with Key | Git Access |
| Certificate | TLS Authentication |

Use the minimum required credential type.

---

# Folder Structure

Enterprise Jenkins organization.

```text
Jenkins

├── Platform

├── Applications

├── Infrastructure

├── Security

├── Shared Libraries

└── Templates
```

Folders simplify permission management.

---

# Shared Libraries

Shared Libraries eliminate duplicate pipeline code.

```text
shared-library

├── vars

├── src

├── resources

└── README.md
```

Examples.

- Docker Build
- Trivy Scan
- Sonar Scan
- Terraform Apply
- Kubernetes Deploy

---

# Jenkins Agent Architecture

Large organizations use distributed build agents.

```text
Developer

↓

Jenkins Controller

↓

Linux Agent

↓

Docker Build

↓

Security Scan

↓

Publish Artifact
```

Agents improve scalability and workload isolation.

---

# Kubernetes-Based Jenkins Agents

Modern Jenkins environments dynamically provision agents.

```text
Jenkins Controller

↓

Kubernetes Plugin

↓

Temporary Build Pod

↓

Pipeline Execution

↓

Pod Deleted
```

Benefits.

- Auto Scaling
- Isolation
- Faster Builds
- Lower Infrastructure Cost

---

# Recommended Agent Images

Separate agents for different workloads.

| Agent | Purpose |
|---------|----------|
| Maven | Java Builds |
| Node.js | Frontend Builds |
| Python | Python Projects |
| Docker | Image Builds |
| Terraform | Infrastructure |
| Security | DevSecOps Tools |

Specialized agents reduce build complexity.

---

# Source Code Checkout

Example.

```groovy
stage('Checkout') {

    steps {

        git branch: 'main',

        url: 'https://github.com/company/payment-service.git'

    }

}
```

This stage retrieves the latest application source code.

---

# Build Stage

Java example.

```groovy
stage('Build') {

    steps {

        sh 'mvn clean package'

    }

}
```

Build failures should stop the pipeline immediately.

---

# Unit Testing

Example.

```groovy
stage('Unit Tests') {

    steps {

        sh 'mvn test'

    }

}
```

Unit tests validate application functionality before security scanning.

---

# Code Coverage

Example.

```groovy
stage('Coverage') {

    steps {

        sh 'mvn jacoco:report'

    }

}
```

Coverage reports identify untested code.

---

# SonarQube Integration

Pipeline stage.

```groovy
stage('SonarQube Scan') {

    steps {

        sh 'mvn sonar:sonar'

    }

}
```

Static code analysis detects quality issues and security vulnerabilities.

---

# Sonar Quality Gate

Quality Gate blocks insecure builds.

```text
Code Analysis

↓

Quality Gate

↓

Pass?

    │

┌───┴────┐

▼        ▼

Yes      No

│         │

Continue  Fail Pipeline
```

Quality Gates should be mandatory for production branches.

---

# Enterprise Best Practices

- Use Jenkins LTS releases.
- Install only required plugins.
- Keep plugins updated.
- Store credentials in Jenkins Credentials Store.
- Use Kubernetes-based build agents.
- Create reusable Shared Libraries.
- Separate Controller and Agents.
- Use Role-Based Access Control.
- Restrict administrator access.
- Back up Jenkins configuration regularly.

===

# OWASP Dependency-Check Integration

OWASP Dependency-Check identifies vulnerable third-party libraries used by the application.

Pipeline stage.

```groovy
stage('Dependency Scan') {

    steps {

        sh '''

        dependency-check.sh \
        --scan . \
        --format HTML \
        --out reports

        '''

    }

}
```

Builds should fail if Critical vulnerabilities are detected.

---

# Gitleaks Integration

Gitleaks detects secrets before code is deployed.

Pipeline stage.

```groovy
stage('Secret Scan') {

    steps {

        sh '''

        gitleaks detect \
        --source . \
        --report-format json \
        --report-path gitleaks-report.json

        '''

    }

}
```

Typical secrets.

- AWS Keys
- Azure Keys
- GitHub Tokens
- Database Passwords
- Private Keys

---

# Checkov Integration

Checkov scans Infrastructure as Code.

Pipeline stage.

```groovy
stage('Checkov Scan') {

    steps {

        sh '''

        checkov \
        -d . \
        -o cli

        '''

    }

}
```

Resources are validated before infrastructure deployment.

---

# TFSec Integration

TFSec validates Terraform security.

Pipeline stage.

```groovy
stage('TFSec Scan') {

    steps {

        sh '''

        tfsec .

        '''

    }

}
```

Terraform misconfigurations are detected before deployment.

---

# Docker Build Stage

Container images are built only after source code passes security validation.

```groovy
stage('Docker Build') {

    steps {

        sh '''

        docker build \
        -t payment-service:${BUILD_NUMBER} .

        '''

    }

}
```

---

# Trivy Integration

Trivy scans container images for vulnerabilities.

Pipeline stage.

```groovy
stage('Trivy Scan') {

    steps {

        sh '''

        trivy image \
        --exit-code 1 \
        payment-service:${BUILD_NUMBER}

        '''

    }

}
```

The pipeline should stop if High or Critical vulnerabilities are detected.

---

# SBOM Generation

Generate a Software Bill of Materials after the image scan.

```groovy
stage('Generate SBOM') {

    steps {

        sh '''

        trivy image \
        --format cyclonedx \
        --output sbom.json \
        payment-service:${BUILD_NUMBER}

        '''

    }

}
```

The SBOM should be archived with the build artifacts.

---

# Image Signing

Digitally sign container images before publishing.

```groovy
stage('Image Signing') {

    steps {

        sh '''

        cosign sign \
        payment-service:${BUILD_NUMBER}

        '''

    }

}
```

Signed images help protect the software supply chain.

---

# Push Image to Amazon ECR

Publish approved images.

```groovy
stage('Push Image') {

    steps {

        sh '''

        docker push \
        payment-service:${BUILD_NUMBER}

        '''

    }

}
```

Only signed and approved images should be pushed.

---

# Update GitOps Repository

Instead of deploying directly, update the GitOps repository.

```text
Jenkins

↓

Update Image Tag

↓

Git Commit

↓

Push

↓

GitOps Repository
```

Git becomes the deployment source of truth.

---

# GitOps Commit

Example.

```bash
git clone https://github.com/company/gitops.git

cd gitops

sed -i "s/tag:.*/tag: ${BUILD_NUMBER}/" deployment.yaml

git add .

git commit -m "Update image"

git push
```

ArgoCD automatically detects the change.

---

# ArgoCD Deployment

Deployment workflow.

```text
GitOps Repository

↓

ArgoCD

↓

Compare Desired State

↓

Sync

↓

Amazon EKS

↓

Production
```

No direct deployment from Jenkins is required.

---

# Post Deployment Validation

Validate the deployment.

Pipeline stage.

```groovy
stage('Validation') {

    steps {

        sh '''

        kubectl get pods

        kubectl get deployments

        '''

    }

}
```

Smoke tests should execute before pipeline completion.

---

# Security Reports

Generate reports from every security tool.

```text
Build Reports

├── SonarQube

├── Dependency Check

├── Gitleaks

├── Checkov

├── TFSec

├── Trivy

├── SBOM

└── Build Logs
```

Reports should be archived for auditing.

---

# Artifact Archiving

Archive reports after every build.

Example.

```groovy
post {

    always {

        archiveArtifacts artifacts: 'reports/**'

    }

}
```

Historical reports assist with audits and investigations.

---

# Pipeline Failure Strategy

Every security stage should stop the pipeline when Critical findings exist.

```text
Pipeline

↓

Security Stage

↓

Critical Found?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Stop      Continue
```

Early failure reduces deployment risk.

---

# Parallel Security Scanning

Independent security checks can execute simultaneously.

```text
Build

↓

Parallel Stage

├── SonarQube

├── Dependency Check

├── Gitleaks

├── Checkov

└── TFSec

↓

Results

↓

Continue
```

Parallel execution significantly reduces pipeline duration.

---

# Enterprise Best Practices

- Execute security scans before Docker image creation whenever possible.
- Run independent scans in parallel to reduce build time.
- Fail builds on Critical vulnerabilities.
- Generate an SBOM for every production image.
- Sign every production container image.
- Store reports for compliance and audits.
- Use GitOps instead of direct deployments.
- Never bypass security gates.
- Automate deployment validation after every release.
- Continuously update security scanning tools.

===

# Complete Production Jenkinsfile

The following Jenkinsfile demonstrates an enterprise DevSecOps pipeline with multiple security gates.

```groovy
pipeline {

    agent any

    environment {

        IMAGE = "company/payment-service:${BUILD_NUMBER}"

    }

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

        stage('Unit Tests') {

            steps {

                sh 'mvn test'

            }

        }

        stage('Coverage') {

            steps {

                sh 'mvn jacoco:report'

            }

        }

        stage('Security Scans') {

            parallel {

                stage('SonarQube') {

                    steps {

                        sh 'mvn sonar:sonar'

                    }

                }

                stage('Dependency Check') {

                    steps {

                        sh 'dependency-check.sh --scan .'

                    }

                }

                stage('Gitleaks') {

                    steps {

                        sh 'gitleaks detect --source .'

                    }

                }

                stage('Checkov') {

                    steps {

                        sh 'checkov -d .'

                    }

                }

                stage('TFSec') {

                    steps {

                        sh 'tfsec .'

                    }

                }

            }

        }

        stage('Docker Build') {

            steps {

                sh 'docker build -t $IMAGE .'

            }

        }

        stage('Trivy Scan') {

            steps {

                sh 'trivy image --exit-code 1 $IMAGE'

            }

        }

        stage('Generate SBOM') {

            steps {

                sh 'trivy image --format cyclonedx --output sbom.json $IMAGE'

            }

        }

        stage('Image Signing') {

            steps {

                sh 'cosign sign $IMAGE'

            }

        }

        stage('Push Image') {

            steps {

                sh 'docker push $IMAGE'

            }

        }

    }

}
```

---

# Security Gate Workflow

Every stage validates the application before allowing the pipeline to continue.

```text
Checkout

↓

Build

↓

Tests

↓

Security Scans

↓

Passed?

     │

┌────┴─────┐

▼          ▼

Yes         No

│            │

Continue   Stop Pipeline
```

Each security gate reduces the likelihood of vulnerabilities reaching production.

---

# Security Decision Flow

```text
Source Code

↓

SonarQube

↓

Dependencies

↓

OWASP Dependency-Check

↓

Secrets

↓

Gitleaks

↓

Terraform

↓

Checkov

↓

TFSec

↓

Container

↓

Trivy

↓

Passed?

     │

┌────┴─────┐

▼          ▼

Yes         No

│            │

Deploy      Stop
```

---

# Build Reports

Every execution should generate reports.

```text
reports/

├── sonar/

├── dependency-check/

├── gitleaks/

├── checkov/

├── tfsec/

├── trivy/

├── sbom/

└── build.log
```

Reports should be archived and retained for compliance.

---

# Report Publishing

Example.

```groovy
post {

    always {

        publishHTML(target: [

            reportDir: 'reports',

            reportFiles: 'index.html',

            reportName: 'Security Report'

        ])

    }

}
```

Security reports should be easily accessible from Jenkins.

---

# Notifications

Notify development and security teams after pipeline completion.

Supported platforms.

- Email
- Slack
- Microsoft Teams
- Webhooks

Workflow.

```text
Pipeline

↓

Completed

↓

Notification

↓

Development Team

↓

Security Team
```

Critical failures should trigger immediate notifications.

---

# Slack Notification Example

```groovy
post {

    failure {

        slackSend(

            channel: '#devsecops',

            message: 'Build Failed'

        )

    }

}
```

Security failures should be communicated immediately.

---

# Build Retention

Old builds consume storage.

Example.

```groovy
options {

    buildDiscarder(

        logRotator(

            numToKeepStr: '30'

        )

    )

}
```

Retention policies help manage Jenkins storage efficiently.

---

# Backup Strategy

Back up Jenkins regularly.

Critical data.

```text
Jenkins Home

↓

Jobs

↓

Credentials

↓

Plugins

↓

Shared Libraries

↓

Configuration

↓

Pipeline History
```

Backups should be automated and tested periodically.

---

# High Availability Architecture

Enterprise Jenkins deployments should avoid a single point of failure.

```text
GitHub

↓

Load Balancer

↓

Jenkins Controller

↓

Kubernetes Agents

↓

Amazon EKS
```

Use external storage and automated backups for resilience.

---

# Monitoring Jenkins

Monitor the Jenkins platform continuously.

Recommended metrics.

- CPU Usage
- Memory Usage
- Disk Space
- Queue Length
- Build Duration
- Failed Builds
- Active Agents
- Plugin Health

---

# Monitoring Architecture

```text
Jenkins

↓

Prometheus

↓

Grafana

↓

Operations Dashboard
```

Operations teams can monitor build performance and system health in real time.

---

# Logging

Centralize Jenkins logs for troubleshooting.

```text
Jenkins

↓

Log Collection

↓

Elastic Stack

↓

Search

↓

Analysis
```

Centralized logging simplifies incident investigation.

---

# Security Hardening

Secure Jenkins before onboarding development teams.

Recommendations.

- Enable HTTPS.
- Disable anonymous access.
- Enable CSRF protection.
- Enable RBAC.
- Use least-privilege permissions.
- Rotate credentials regularly.
- Keep plugins updated.
- Restrict Script Console access.
- Enable audit logging.
- Use dedicated build agents.

---

# Enterprise Best Practices

- Separate Jenkins Controllers from build agents.
- Use ephemeral Kubernetes agents.
- Store credentials in Jenkins Credentials Store.
- Execute security scans in parallel.
- Fail builds on Critical vulnerabilities.
- Sign every production container image.
- Store SBOMs with build artifacts.
- Archive all security reports.
- Monitor Jenkins using Prometheus and Grafana.
- Centralize logs using the ELK Stack.
- Back up Jenkins automatically.
- Review plugin updates regularly.

---

