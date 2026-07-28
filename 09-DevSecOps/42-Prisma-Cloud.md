# Prisma Cloud

## Introduction

Prisma Cloud is Palo Alto Networks' Cloud Native Application Protection Platform (CNAPP) that provides comprehensive security across cloud infrastructure, containers, Kubernetes, serverless workloads, Infrastructure as Code (IaC), software supply chains, identities, and runtime environments.

Prisma Cloud enables organizations to secure cloud-native applications from development through production using a unified platform for posture management, vulnerability management, runtime protection, compliance, and threat detection.

Large enterprises use Prisma Cloud to secure multi-cloud environments across AWS, Azure, Google Cloud Platform, and Kubernetes.

---

# Why Companies Use Prisma Cloud

Modern cloud environments require continuous visibility into infrastructure, workloads, identities, and applications.

Prisma Cloud provides centralized cloud security across the entire application lifecycle.

## Benefits

- Cloud Security Posture Management (CSPM)
- Cloud Workload Protection (CWPP)
- Container Security
- Kubernetes Security
- Infrastructure as Code Security
- Runtime Protection
- Vulnerability Management
- Identity Security
- Compliance Monitoring
- Software Supply Chain Security

---

# Prisma Cloud vs Traditional Security

| Capability | Prisma Cloud | Traditional Security |
|------------|--------------|----------------------|
| Multi-Cloud Security | ✓ | Limited |
| Kubernetes Security | ✓ | Limited |
| Runtime Protection | ✓ | Limited |
| IaC Security | ✓ | ✗ |
| Compliance Monitoring | ✓ | Partial |
| Cloud Misconfiguration Detection | ✓ | ✗ |
| Identity Security | ✓ | Partial |
| Container Security | ✓ | Partial |
| Central Dashboard | ✓ | Partial |
| Policy Automation | ✓ | Limited |

Prisma Cloud provides cloud-native security instead of relying solely on traditional perimeter security.

---

# Where Prisma Cloud Fits in DevSecOps

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

CI Pipeline

↓

Checkout

↓

Build

↓

SonarQube

↓

OWASP Dependency-Check

↓

Docker Build

↓

Prisma Image Scan

↓

Policy Validation

↓

Artifact Repository

↓

GitOps

↓

ArgoCD

↓

Amazon EKS

↓

Prisma Runtime Protection

↓

Production
```

Prisma Cloud secures workloads before deployment and continuously protects them during runtime.

---

# Enterprise Architecture

```text
                    Developers
                         │
                         ▼
                  Git Repository
                         │
                         ▼
        Jenkins / GitHub Actions / GitLab
                         │
                         ▼
                 Build Container Image
                         │
                         ▼
                 Prisma Image Scan
                         │
                  Policy Evaluation
                         │
                         ▼
                Amazon ECR / ACR / GAR
                         │
                         ▼
                     Kubernetes
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
     Prisma Defender Prisma Defender Prisma Defender
          │              │              │
          └──────────────┼──────────────┘
                         ▼
                  Prisma Console
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
    SIEM            Prometheus        Slack
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

Docker Build

↓

Prisma Image Scan

↓

Amazon ECR

↓

GitOps

↓

ArgoCD

↓

Amazon EKS

↓

Prisma Defender

↓

Prisma Console

↓

SOC Team
```

---

# Prisma Cloud Components

| Component | Purpose |
|------------|----------|
| Prisma Console | Central management platform |
| Prisma Defender | Runtime protection |
| Image Scanner | Container image security |
| Compliance Engine | Compliance monitoring |
| Admission Controller | Kubernetes policy enforcement |
| Threat Intelligence | Threat detection |
| Identity Security | IAM risk analysis |

---

# Security Coverage

Prisma Cloud protects cloud-native environments across multiple layers.

```text
Source Code

↓

Infrastructure as Code

↓

Container Images

↓

Registries

↓

Kubernetes

↓

Virtual Machines

↓

Serverless

↓

Cloud Infrastructure

↓

Runtime
```

---

# What Prisma Cloud Can Detect

| Security Area | Supported |
|---------------|-----------|
| Container Vulnerabilities | ✓ |
| Cloud Misconfigurations | ✓ |
| Malware | ✓ |
| Secrets | ✓ |
| Kubernetes Risks | ✓ |
| Runtime Threats | ✓ |
| IAM Risks | ✓ |
| Compliance Violations | ✓ |
| Supply Chain Risks | ✓ |
| Network Anomalies | ✓ |

---

# Supported Cloud Platforms

Prisma Cloud supports enterprise multi-cloud environments.

- Amazon Web Services (AWS)
- Microsoft Azure
- Google Cloud Platform (GCP)
- Kubernetes
- OpenShift
- VMware

---

# Prerequisites

| Component | Version |
|------------|----------|
| Kubernetes | 1.28+ |
| Helm | Latest |
| Docker | Latest |
| Amazon EKS | Supported |
| AKS | Supported |
| GKE | Supported |

---

# Installation Methods

Prisma Cloud can be deployed using:

- Helm
- Kubernetes Manifests
- SaaS Console
- Self-Hosted Console
- Docker
- Amazon EKS
- Azure AKS
- Google GKE

The SaaS Console is commonly used for enterprise deployments.

---

# Install Prisma Defender Using Helm

```bash
helm repo add prisma https://charts.prismacloud.io

helm repo update
```

Deploy the Defender.

```bash
helm install prisma-defender prisma/defender \
--namespace prisma \
--create-namespace
```

---

# Verify Installation

```bash
kubectl get pods -n prisma
```

Example output.

```text
prisma-console-xxxxx

Running

prisma-defender-xxxxx

Running
```

---

# Verify Defender Deployment

```bash
kubectl get daemonset -n prisma
```

Expected output.

```text
NAME

prisma-defender

DESIRED

3

CURRENT

3

READY

3
```

Every Kubernetes worker node should run one Prisma Defender instance.

---

# First Container Image Scan

```text
Docker Image

↓

Prisma Scanner

↓

Vulnerability Analysis

↓

Compliance Checks

↓

Policy Evaluation

↓

PASS / FAIL
```

Container images should be scanned before being pushed to the registry to prevent vulnerable workloads from reaching production.

---

# Configuration

Prisma Cloud provides centralized security policy management across cloud infrastructure, containers, Kubernetes, identities, and runtime workloads.

Configuration typically includes:

- Image Scanning Policies
- Runtime Policies
- Cloud Security Policies
- Compliance Policies
- Admission Control
- Registry Integration
- Kubernetes Integration
- Identity Policies
- Notifications

A centralized configuration ensures consistent security across all cloud environments.

---

# Prisma Cloud Architecture

```text
Prisma Console

↓

Security Policies

├── Image Policies

├── Runtime Policies

├── Cloud Policies

├── Compliance Policies

├── IAM Policies

└── Notifications
```

All policies are managed from the Prisma Console.

---

# Configuration Files

Example deployment.

```text
values.yaml

console-values.yaml

defender-values.yaml

scanner-values.yaml
```

Store configuration files in Git and manage them through GitOps.

---

# Configure Prisma Console

Example.

```bash
helm upgrade prisma-console prisma/console \
-f console-values.yaml \
-n prisma
```

Verify.

```bash
kubectl get pods -n prisma
```

---

# Configure Prisma Defender

Example.

```bash
helm upgrade prisma-defender prisma/defender \
-f defender-values.yaml \
-n prisma
```

The Defender automatically protects workloads after deployment.

---

# Container Image Policies

Image policies determine which images are allowed into production.

Example.

```text
Critical Vulnerabilities

↓

Reject

High Vulnerabilities

↓

Reject

Medium Vulnerabilities

↓

Warning

Low Vulnerabilities

↓

Allow
```

Production environments should reject images containing Critical vulnerabilities.

---

# Runtime Policies

Runtime policies monitor application behaviour after deployment.

Examples.

- Process Execution
- Network Connections
- File Access
- Package Installation
- Privilege Escalation
- Container Escape
- Reverse Shell

Workflow.

```text
Runtime Event

↓

Policy Evaluation

↓

Allow

or

Alert

or

Block
```

---

# Admission Controller

Prisma Cloud validates Kubernetes workloads before deployment.

```text
kubectl apply

↓

API Server

↓

Admission Controller

↓

Security Validation

↓

Approved?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Deploy     Reject
```

Only compliant workloads should enter the Kubernetes cluster.

---

# Registry Integration

Prisma Cloud continuously scans enterprise container registries.

Supported registries.

- Amazon ECR
- Azure Container Registry
- Google Artifact Registry
- Harbor
- Docker Hub
- JFrog Artifactory

Workflow.

```text
Container Registry

↓

New Image

↓

Automatic Scan

↓

Security Report
```

---

# Kubernetes Integration

Prisma integrates directly with Kubernetes clusters.

```text
Kubernetes API

↓

Prisma Console

↓

Prisma Defender

↓

Runtime Protection
```

All nodes remain continuously protected.

---

# Cloud Account Integration

Prisma Cloud connects directly to cloud providers.

Supported platforms.

- AWS
- Azure
- Google Cloud Platform

Architecture.

```text
Cloud Account

↓

Prisma Cloud

↓

Inventory

↓

Risk Analysis

↓

Security Dashboard
```

Cloud resources are continuously evaluated for security risks.

---

# Identity Security

Prisma Cloud analyses cloud identities and permissions.

Examples.

- IAM Users
- IAM Roles
- Service Accounts
- Azure AD Roles
- Google IAM Roles

Workflow.

```text
Cloud Identity

↓

Permission Analysis

↓

Risk Detection

↓

Recommendation
```

Excessive permissions are identified automatically.

---

# Compliance Policies

Compliance policies validate cloud resources against industry standards.

Supported frameworks.

- CIS AWS Benchmark
- CIS Azure Benchmark
- CIS Kubernetes Benchmark
- PCI DSS
- HIPAA
- NIST
- SOC 2
- ISO 27001

Workflow.

```text
Cloud Resources

↓

Compliance Scan

↓

Violations

↓

Compliance Report
```

---

# Vulnerability Management

Prisma continuously tracks vulnerabilities.

Workflow.

```text
Container Image

↓

Operating System Packages

↓

Application Dependencies

↓

Vulnerability Database

↓

Risk Assessment

↓

Security Report
```

Risk scores help prioritize remediation.

---

# Secret Detection

Prisma scans workloads for exposed credentials.

Examples.

```text
AWS Keys

Azure Keys

Private Keys

JWT Secrets

Database Passwords

GitHub Tokens
```

Secrets should never be stored inside container images or source repositories.

---

# Malware Detection

Container images are analysed for malicious software.

Workflow.

```text
Container Image

↓

Malware Scan

↓

Threat Intelligence

↓

Detection

↓

Alert
```

Images containing malware should be blocked immediately.

---

# Notifications

Prisma Cloud supports enterprise notification platforms.

```text
Prisma Console

↓

Alert

├── Email

├── Slack

├── Microsoft Teams

├── Webhook

└── SIEM
```

Critical alerts should be delivered immediately.

---

# Role-Based Access Control (RBAC)

Enterprise environments should separate responsibilities.

| Role | Permissions |
|------|-------------|
| Administrator | Full Access |
| Security Team | Policy Management |
| DevOps Engineer | Image Scans |
| Developer | View Findings |
| Auditor | Read Only |

Apply the principle of least privilege.

---

# Authentication

Prisma Cloud supports enterprise identity providers.

Examples.

- LDAP
- Active Directory
- SAML
- OAuth
- OpenID Connect

Workflow.

```text
User

↓

Identity Provider

↓

Authentication

↓

Prisma Console
```

Centralized authentication improves governance and auditing.

---

# Audit Logging

Every administrative activity is recorded.

Examples.

```text
User Login

Policy Update

Image Scan

Compliance Scan

Deployment Decision

Runtime Alert
```

Audit logs should be retained according to organizational compliance requirements.

---

# Enterprise Best Practices

- Store Helm values files in Git repositories.
- Block deployments containing Critical vulnerabilities.
- Enable Admission Controller for production clusters.
- Continuously scan connected registries.
- Enable runtime protection on every Kubernetes node.
- Connect all cloud accounts for posture management.
- Monitor IAM risks continuously.
- Enable compliance monitoring for every cloud environment.
- Forward alerts to SIEM platforms.
- Review RBAC permissions regularly.
- Audit administrative actions and policy changes.

---

# Container Image Scanning

Prisma Cloud scans container images before deployment to identify vulnerabilities, malware, secrets, and compliance violations.

Workflow.

```text
Developer

↓

Docker Build

↓

Container Image

↓

Prisma Scanner

↓

Security Analysis

↓

Policy Evaluation

↓

PASS / FAIL
```

Only compliant images should be promoted to production.

---

# Image Assurance Policies

Image Assurance validates container images against enterprise security policies.

Example.

```text
Critical Vulnerabilities

↓

Reject

High Vulnerabilities

↓

Reject

Medium Vulnerabilities

↓

Warning

Low Vulnerabilities

↓

Allow
```

Policy enforcement prevents insecure images from reaching Kubernetes.

---

# Image Scan Workflow

```text
Container Image

↓

Operating System Packages

↓

Application Dependencies

↓

Secrets

↓

Malware

↓

Compliance Checks

↓

Risk Score

↓

Security Report
```

Prisma Cloud performs multiple security checks in a single scan.

---

# Registry Scanning

Container registries are monitored continuously.

```text
Developer

↓

Push Image

↓

Amazon ECR

↓

Automatic Prisma Scan

↓

Security Report
```

Supported registries.

- Amazon ECR
- Azure Container Registry
- Google Artifact Registry
- Harbor
- Docker Hub
- JFrog Artifactory

---

# Vulnerability Detection

Prisma Cloud detects vulnerabilities across operating systems and application packages.

Examples.

```text
Ubuntu

↓

OpenSSL

↓

glibc

↓

curl
```

```text
Java

↓

Spring Boot

↓

Node.js

↓

Python

↓

Go

↓

.NET
```

All packages are compared against continuously updated vulnerability databases.

---

# Malware Detection

Container images are scanned for malicious software before deployment.

Workflow.

```text
Container Image

↓

Malware Scan

↓

Threat Intelligence

↓

Detection

↓

Alert
```

Known malware signatures are identified automatically.

---

# Secret Detection

Prisma detects secrets embedded inside images.

Examples.

```text
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

DATABASE_PASSWORD

JWT_SECRET

SSH_PRIVATE_KEY

GITHUB_TOKEN
```

Secrets should always be managed through secure secret management solutions.

---

# Software Supply Chain Security

Prisma Cloud protects the software supply chain from development through deployment.

```text
Source Code

↓

Dependencies

↓

Container Build

↓

Image Scan

↓

Policy Validation

↓

Container Registry

↓

Deployment
```

Supply chain security reduces the risk of compromised software entering production.

---

# Runtime Protection

Prisma Defender continuously monitors running workloads.

Workflow.

```text
Running Container

↓

Process Activity

↓

Network Activity

↓

File Activity

↓

Runtime Policy

↓

Alert
```

Runtime monitoring continues throughout the application's lifecycle.

---

# Runtime Behaviour Monitoring

Prisma Cloud observes workload behaviour in real time.

Examples.

- Shell execution
- Privilege escalation
- File modifications
- Unexpected processes
- Suspicious network traffic

Workflow.

```text
Runtime Event

↓

Policy Engine

↓

Allow

or

Alert

or

Block
```

---

# Container Drift Detection

Production containers should remain identical to the approved image.

Workflow.

```text
Approved Image

↓

Running Container

↓

Unexpected Change

↓

Drift Detected

↓

Alert
```

Examples.

- New binaries
- Installed packages
- Modified configuration
- Permission changes

---

# File Integrity Monitoring

Critical system and application files are monitored continuously.

Examples.

```text
/etc/passwd

/etc/shadow

/root/.ssh

/etc/kubernetes

/app/config
```

Workflow.

```text
File Change

↓

Integrity Check

↓

Alert
```

---

# Process Monitoring

Prisma Defender monitors processes executed inside containers.

Expected processes.

```text
java

node

python

nginx

dotnet
```

Unexpected or unauthorized processes generate alerts.

---

# Network Protection

Prisma Cloud monitors container network activity.

Examples.

```text
Container

↓

Outbound Connection

↓

Unknown Destination

↓

Policy Evaluation

↓

Alert
```

Typical detections.

- Suspicious outbound traffic
- Communication with malicious IPs
- Lateral movement
- Unauthorized ports
- Unexpected DNS requests

---

# Kubernetes Workload Protection

Prisma Cloud protects Kubernetes resources throughout their lifecycle.

Supported resources.

- Pods
- Deployments
- StatefulSets
- DaemonSets
- Jobs
- CronJobs

Workflow.

```text
Kubernetes Resource

↓

Admission Controller

↓

Security Validation

↓

Deploy

or

Reject
```

---

# Admission Controller Workflow

```text
kubectl apply

↓

Kubernetes API Server

↓

Prisma Admission Controller

↓

Image Verification

↓

Policy Validation

↓

Approved?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Deploy     Reject
```

Only verified workloads are admitted into the cluster.

---

# Cloud Security Posture Management (CSPM)

Prisma Cloud continuously evaluates cloud resources for security risks.

Workflow.

```text
AWS

Azure

Google Cloud

↓

Cloud Resource Discovery

↓

Configuration Analysis

↓

Risk Detection

↓

Recommendations
```

Typical findings.

- Public S3 buckets
- Open security groups
- Unencrypted storage
- Public databases
- Weak IAM policies

---

# Identity Security

Cloud identities are analysed for excessive permissions.

Workflow.

```text
IAM User

↓

Permission Analysis

↓

Risk Score

↓

Recommendation
```

Examples.

- Administrator permissions
- Unused credentials
- Excessive privileges
- Cross-account access
- Dormant accounts

---

# Enterprise Best Practices

- Scan every image before publishing.
- Enable automatic registry scanning.
- Reject images containing Critical vulnerabilities.
- Detect malware and secrets before deployment.
- Enable runtime protection on every Kubernetes node.
- Monitor container drift continuously.
- Enforce Admission Controller policies.
- Continuously monitor cloud posture across AWS, Azure, and GCP.
- Review IAM risks regularly.
- Prioritize remediation based on risk score and business impact.

---

# Jenkins Integration

Prisma Cloud integrates with Jenkins to scan container images before they are published to the container registry.

Enterprise workflow.

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

Prisma Image Scan

↓

Policy Validation

↓

Amazon ECR

↓

Deploy
```

Only images that comply with enterprise security policies should be promoted.

---

# Production Jenkins Pipeline

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

        stage('Docker Build') {

            steps {

                sh 'docker build -t $IMAGE .'

            }

        }

        stage('Prisma Scan') {

            steps {

                sh '''

                twistcli images scan \
                --address https://console.company.com \
                --user admin \
                --password $PASSWORD \
                $IMAGE

                '''

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

# GitHub Actions Integration

Enterprise workflow.

```text
Git Push

↓

GitHub Actions

↓

Build

↓

Docker Build

↓

Prisma Scan

↓

Push Image

↓

Deploy
```

---

# Production GitHub Actions Workflow

```yaml
name: Prisma-Cloud-Scan

on:

  push:

    branches:

      - main

jobs:

  security:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - name: Build Image

        run: |

          docker build -t payment-service:${{ github.sha }} .

      - name: Prisma Scan

        run: |

          twistcli images scan \
          --address https://console.company.com \
          --user admin \
          --password $PASSWORD \
          payment-service:${{ github.sha }}
```

---

# GitLab CI Integration

Example.

```yaml
stages:

  - build

  - security

build:

  stage: build

  script:

    - docker build -t payment-service:$CI_COMMIT_SHA .

prisma-scan:

  stage: security

  script:

    - twistcli images scan \
      --address https://console.company.com \
      --user admin \
      --password $PASSWORD \
      payment-service:$CI_COMMIT_SHA
```

---

# Amazon ECR Integration

Prisma continuously monitors Amazon ECR repositories.

Workflow.

```text
Docker Build

↓

Amazon ECR

↓

Automatic Prisma Scan

↓

Policy Validation

↓

Security Report
```

Images are rescanned whenever vulnerability intelligence is updated.

---

# Azure Container Registry Integration

```text
Docker Build

↓

Azure Container Registry

↓

Automatic Scan

↓

Compliance Report
```

Container images remain continuously monitored after publication.

---

# Google Artifact Registry Integration

```text
Docker Build

↓

Google Artifact Registry

↓

Automatic Scan

↓

Risk Assessment
```

Every newly published image is evaluated automatically.

---

# JFrog Artifactory Integration

Prisma Cloud integrates with enterprise artifact repositories.

Workflow.

```text
Docker Build

↓

JFrog Artifactory

↓

Automatic Scan

↓

Policy Validation

↓

Security Report
```

Registries remain under continuous monitoring.

---

# Kubernetes Admission Controller

The Admission Controller prevents non-compliant workloads from entering Kubernetes.

Workflow.

```text
kubectl apply

↓

API Server

↓

Prisma Admission Controller

↓

Image Verified?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Deploy     Reject
```

Admission control reduces deployment risk.

---

# Runtime Protection

Prisma Defender continuously protects running workloads.

```text
Container

↓

Runtime Event

↓

Prisma Defender

↓

Policy Evaluation

↓

Allow

or

Alert

or

Block
```

Examples.

- Shell execution
- Privilege escalation
- Container escape
- Sensitive file access
- Reverse shell
- Suspicious network activity

---

# Software Bill of Materials (SBOM)

Prisma Cloud generates Software Bill of Materials (SBOM) data.

```text
Container Image

↓

Dependency Discovery

↓

Package Inventory

↓

SBOM

↓

Compliance Report
```

SBOMs improve software supply chain visibility and audit readiness.

---

# Image Signing Verification

Digitally signed images improve software supply chain integrity.

Workflow.

```text
Container Image

↓

Digital Signature

↓

Signature Verification

↓

Approved

↓

Deploy
```

Unsigned or modified images should be rejected.

---

# Compliance Reporting

Prisma Cloud generates enterprise compliance reports.

Typical report.

```text
Compliance Report

├── CIS

├── PCI DSS

├── HIPAA

├── NIST

├── ISO 27001

├── SOC 2

└── Recommendations
```

Reports support regulatory audits and governance.

---

# Security Dashboard

The Prisma Console provides centralized visibility across cloud environments.

Typical dashboard sections.

- Vulnerability Summary
- Runtime Threats
- Cloud Misconfigurations
- Compliance Status
- Container Inventory
- Kubernetes Risks
- Identity Risks
- Malware Detection
- Secret Exposure
- Policy Violations

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

Docker Build

↓

Prisma Image Scan

↓

SBOM

↓

Image Signing

↓

Amazon ECR

↓

Admission Controller

↓

GitOps

↓

ArgoCD

↓

Amazon EKS

↓

Prisma Defender

↓

Runtime Protection

↓

Production
```

Prisma Cloud provides security from source code through runtime.

---

# Enterprise Best Practices

- Scan every image before publishing.
- Continuously monitor container registries.
- Block deployments containing Critical vulnerabilities.
- Enable Admission Controller in production.
- Generate SBOMs for production images.
- Verify image signatures before deployment.
- Enable runtime protection on every Kubernetes node.
- Integrate alerts with SIEM platforms.
- Archive compliance reports for audits.
- Keep Prisma policies and Defender components updated.

===

# AWS Production Architecture

Prisma Cloud provides end-to-end security for AWS workloads from infrastructure provisioning to runtime protection.

Architecture.

```text
Developer

↓

GitHub

↓

Jenkins

↓

Terraform

↓

AWS Infrastructure

↓

Docker Build

↓

Prisma Image Scan

↓

Amazon ECR

↓

Admission Controller

↓

ArgoCD

↓

Amazon EKS

↓

Prisma Defender

↓

Prisma Console

↓

SOC Team
```

Prisma continuously monitors AWS resources and Kubernetes workloads.

---

# Azure Production Architecture

Example deployment.

```text
Developer

↓

Azure DevOps

↓

Terraform

↓

Azure Resources

↓

Azure Container Registry

↓

AKS

↓

Prisma Defender

↓

Prisma Console
```

Azure resources remain continuously monitored for posture, compliance, and runtime threats.

---

# Google Cloud Production Architecture

Example deployment.

```text
Developer

↓

Cloud Build

↓

Terraform

↓

Google Cloud Resources

↓

Artifact Registry

↓

Google Kubernetes Engine

↓

Prisma Defender

↓

Security Dashboard
```

All cloud resources remain under continuous security monitoring.

---

# Cloud Security Posture Management (CSPM)

Prisma Cloud continuously evaluates cloud environments.

```text
AWS

Azure

Google Cloud

↓

Cloud Discovery

↓

Configuration Analysis

↓

Risk Detection

↓

Recommendations
```

Typical findings.

- Public storage buckets
- Open security groups
- Internet-facing databases
- Disabled encryption
- Weak IAM policies
- Unused resources

---

# Cloud Workload Protection Platform (CWPP)

CWPP protects workloads running in cloud environments.

```text
Virtual Machines

↓

Containers

↓

Kubernetes

↓

Serverless

↓

Runtime Protection
```

Workloads remain protected throughout their lifecycle.

---

# Identity Security

Cloud identities are continuously analysed.

Workflow.

```text
IAM User

↓

Role Analysis

↓

Permission Review

↓

Risk Score

↓

Recommendation
```

Typical risks.

- Administrator permissions
- Dormant accounts
- Unused access keys
- Privilege escalation paths
- Excessive permissions

---

# Runtime Threat Detection

Prisma Defender continuously detects suspicious activity.

Examples.

```text
Container Escape

Reverse Shell

Privilege Escalation

Malware

Cryptocurrency Mining

Sensitive File Access
```

Workflow.

```text
Runtime Event

↓

Prisma Defender

↓

Threat Detection

↓

Alert

↓

SOC Team
```

---

# Container Drift Detection

Production containers should remain immutable.

Workflow.

```text
Approved Image

↓

Running Container

↓

Unexpected Modification

↓

Drift Detection

↓

Alert
```

Examples.

- Installed software
- Modified binaries
- New users
- Configuration changes
- File permission changes

---

# Security Dashboard

The Prisma Console provides centralized visibility.

```text
Dashboard

├── Cloud Risks

├── Vulnerabilities

├── Runtime Alerts

├── Compliance

├── Identity Risks

├── Kubernetes Security

├── Malware

├── Secrets

├── Registry Security

└── Policy Violations
```

The dashboard helps security teams prioritize remediation activities.

---

# Incident Response Workflow

Enterprise incident response.

```text
Prisma Alert

↓

SOC Team

↓

Validate Alert

↓

Contain Threat

↓

Collect Evidence

↓

Root Cause Analysis

↓

Remediation

↓

Close Incident
```

Every Critical alert should be investigated immediately.

---

# Common Mistakes

## Mistake 1

Scanning images only before production releases.

**Impact**

Security issues remain undetected during development.

**Recommendation**

Scan every image generated by the CI pipeline.

---

## Mistake 2

Ignoring cloud posture findings.

**Impact**

Misconfigured cloud resources increase the attack surface.

**Recommendation**

Review CSPM findings regularly and remediate High and Critical risks.

---

## Mistake 3

Disabling Admission Controller.

**Impact**

Unverified workloads may be deployed to Kubernetes.

**Recommendation**

Keep Admission Controller enabled for production environments.

---

## Mistake 4

Ignoring runtime alerts.

**Impact**

Compromised workloads may continue running.

**Recommendation**

Forward alerts to SIEM platforms and incident response teams.

---

## Mistake 5

Granting excessive Console permissions.

**Impact**

Unauthorized policy changes become possible.

**Recommendation**

Implement RBAC using the principle of least privilege.

---

# Troubleshooting

## Scenario 1

### Prisma Console Is Not Accessible

**Cause**

Console pod or Service is unavailable.

**Resolution**

```bash
kubectl get pods -n prisma

kubectl get svc -n prisma

kubectl describe pod <console-pod> -n prisma
```

---

## Scenario 2

### Image Scan Failed

**Cause**

Registry authentication or image access failed.

**Resolution**

- Verify registry credentials.
- Confirm image exists.
- Check scanner connectivity.
- Review scan logs.

---

## Scenario 3

### Defender Pods Not Running

**Cause**

DaemonSet deployment failed.

**Resolution**

```bash
kubectl get daemonset -n prisma

kubectl describe daemonset prisma-defender -n prisma
```

Ensure every worker node runs a Defender pod.

---

## Scenario 4

### Admission Controller Rejects Deployments

**Cause**

Image policy validation failed.

**Resolution**

- Review Admission Controller logs.
- Verify image compliance.
- Check image signature.
- Confirm registry connectivity.

---

## Scenario 5

### Excessive Runtime Alerts

**Cause**

Policies are not tuned for the application workload.

**Resolution**

- Review Runtime Policies.
- Tune detection rules.
- Add approved exceptions.
- Validate alerts before suppressing them.

---

# Production Interview Questions

## Question 1

### What is Prisma Cloud?

**Answer**

Prisma Cloud is Palo Alto Networks' Cloud Native Application Protection Platform (CNAPP) that secures cloud infrastructure, containers, Kubernetes, identities, workloads, and software supply chains from development through production.

---

## Question 2

### What is the difference between CSPM and CWPP?

**Answer**

CSPM identifies cloud infrastructure misconfigurations and compliance issues, while CWPP protects running workloads such as virtual machines, containers, Kubernetes clusters, and serverless applications.

---

## Question 3

### What is Prisma Defender?

**Answer**

Prisma Defender is the runtime component that monitors workloads, enforces security policies, detects threats, and reports runtime events to the Prisma Console.

---

## Question 4

### Why is Admission Controller important?

**Answer**

Admission Controller validates Kubernetes workloads before deployment to ensure only compliant images and configurations are admitted into the cluster.

---

## Question 5

### What is Cloud Security Posture Management (CSPM)?

**Answer**

CSPM continuously monitors cloud environments to detect misconfigurations, policy violations, and compliance issues across AWS, Azure, and Google Cloud.

---

## Question 6

### What is Container Drift?

**Answer**

Container Drift occurs when a running container changes from its approved image due to installed packages, modified files, or configuration changes after deployment.

---

## Question 7

### Can Prisma Cloud generate an SBOM?

**Answer**

Yes. Prisma Cloud supports Software Bill of Materials (SBOM) generation for container images, helping organizations understand software dependencies and improve supply chain security.

---

## Question 8

### Which cloud providers are supported?

**Answer**

Prisma Cloud supports AWS, Microsoft Azure, Google Cloud Platform, Kubernetes, OpenShift, and other cloud-native environments.

---

## Question 9

### How does Prisma Cloud integrate into a DevSecOps pipeline?

**Answer**

Prisma Cloud scans container images during CI/CD, validates deployment policies, protects Kubernetes workloads at runtime, continuously monitors cloud posture, and forwards alerts to enterprise monitoring and SIEM platforms.

---

## Question 10

### What are the enterprise best practices for Prisma Cloud?

**Answer**

Scan every image, enable Admission Controller, deploy Prisma Defender on every Kubernetes node, continuously monitor cloud posture, integrate with SIEM platforms, enforce RBAC, generate SBOMs, monitor runtime behaviour, and regularly review compliance findings.

---

# Key Takeaways

- Prisma Cloud is an enterprise CNAPP for securing cloud-native applications.
- Protect cloud infrastructure using CSPM.
- Protect workloads using CWPP and Prisma Defender.
- Scan container images before deployment.
- Enforce Kubernetes Admission Controller policies.
- Continuously monitor runtime threats and container drift.
- Secure identities using IAM risk analysis.
- Integrate Prisma Cloud with Jenkins, GitHub Actions, GitLab CI, Amazon EKS, AKS, GKE, ArgoCD, Amazon ECR, Azure Container Registry, and Google Artifact Registry.
- Forward alerts to SIEM and incident response platforms.
- Combine Prisma Cloud with SonarQube, OWASP Dependency-Check, Veracode, Gitleaks, Checkov, TFSec, Trivy, OWASP ZAP, Falco, Aqua Security, GitOps, and ArgoCD to implement a comprehensive enterprise DevSecOps platform.