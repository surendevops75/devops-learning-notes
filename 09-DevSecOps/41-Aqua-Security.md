# Aqua Security

## Introduction

Aqua Security is an enterprise Cloud Native Application Protection Platform (CNAPP) that provides end-to-end security for containers, Kubernetes, serverless workloads, Infrastructure as Code (IaC), software supply chains, and cloud environments.

Unlike standalone security tools, Aqua Security combines vulnerability management, runtime protection, compliance monitoring, image assurance, malware detection, secret scanning, and policy enforcement into a single platform.

Large enterprises use Aqua Security to secure cloud-native applications throughout the entire Software Development Life Cycle (SDLC), from source code to production runtime.

---

# Why Companies Use Aqua Security

Modern applications require multiple security controls across development, deployment, and runtime.

Instead of deploying separate tools for every stage, Aqua Security provides a unified platform for securing the entire cloud-native ecosystem.

## Benefits

- Container Image Security
- Kubernetes Security
- Runtime Protection
- Infrastructure as Code Security
- Malware Detection
- Secret Detection
- Software Supply Chain Security
- Compliance Monitoring
- Admission Control
- Risk-Based Prioritization

---

# Aqua Security vs Open Source Tools

| Capability | Aqua Security | Open Source Tools |
|------------|---------------|-------------------|
| Container Scanning | ✓ | Trivy |
| Runtime Protection | ✓ | Falco |
| IaC Security | ✓ | Checkov / TFSec |
| Secret Detection | ✓ | Gitleaks |
| Malware Detection | ✓ | Limited |
| Admission Control | ✓ | Partial |
| Compliance | ✓ | Partial |
| Policy Management | ✓ | Limited |
| Central Dashboard | ✓ | ✗ |
| Enterprise Support | ✓ | ✗ |

Aqua Security consolidates multiple security capabilities into a single enterprise platform.

---

# Where Aqua Security Fits in DevSecOps

Aqua Security protects workloads throughout the software delivery lifecycle.

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

Checkout

↓

Build

↓

SonarQube

↓

OWASP Dependency-Check

↓

Container Build

↓

Aqua Image Scan

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

Aqua Runtime Protection

↓

Production
```

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
             Build Container Image
                       │
                       ▼
             Aqua Image Scanner
                       │
              Policy Evaluation
                       │
                       ▼
                Artifact Registry
                       │
                       ▼
                  Amazon EKS
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
     Aqua Enforcer Aqua Enforcer Aqua Enforcer
          │            │            │
          └────────────┼────────────┘
                       ▼
                 Aqua Console
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      SIEM        Prometheus     Slack
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

Aqua Image Scan

↓

Amazon ECR

↓

ArgoCD

↓

Amazon EKS

↓

Aqua Runtime Protection

↓

Security Dashboard

↓

SOC Team
```

---

# Aqua Security Components

| Component | Purpose |
|------------|----------|
| Aqua Console | Central management |
| Aqua Enforcer | Runtime protection |
| Scanner | Image vulnerability scanning |
| Gateway | Admission control |
| CyberCenter | Threat intelligence |
| Trivy Integration | Open-source scanning |

---

# Security Coverage

Aqua Security protects multiple cloud-native assets.

```text
Source Code

↓

Dependencies

↓

Containers

↓

Images

↓

Registries

↓

Kubernetes

↓

Serverless

↓

Runtime

↓

Cloud Infrastructure
```

---

# What Aqua Security Can Detect

| Security Area | Supported |
|---------------|-----------|
| Container Vulnerabilities | ✓ |
| Malware | ✓ |
| Secrets | ✓ |
| Misconfigurations | ✓ |
| Privilege Escalation | ✓ |
| Container Escape | ✓ |
| Compliance Violations | ✓ |
| Runtime Threats | ✓ |
| Image Drift | ✓ |
| Suspicious Processes | ✓ |

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

Aqua Security can be deployed using:

- Helm
- Kubernetes Manifests
- Docker
- Amazon EKS
- AKS
- GKE

Helm is the recommended installation method for enterprise Kubernetes clusters.

---

# Add Aqua Helm Repository

```bash
helm repo add aqua https://helm.aquasec.com

helm repo update
```

---

# Install Aqua Console

```bash
helm install aqua-console aqua/console \
--namespace aqua \
--create-namespace
```

---

# Install Aqua Enforcer

```bash
helm install aqua-enforcer aqua/enforcer \
--namespace aqua
```

The Enforcer runs on Kubernetes nodes and provides runtime protection.

---

# Verify Installation

Verify the deployed components.

```bash
kubectl get pods -n aqua
```

Example output.

```text
aqua-console-xxxxx

Running

aqua-enforcer-xxxxx

Running
```

---

# Verify Runtime Protection

Check the Enforcer DaemonSet.

```bash
kubectl get daemonset -n aqua
```

Expected output.

```text
NAME

aqua-enforcer

DESIRED

3

CURRENT

3

READY

3
```

Every Kubernetes worker node should run one Aqua Enforcer instance.

---

# First Image Scan

Scan a container image before deployment.

```text
Container Image

↓

Aqua Scanner

↓

Vulnerability Analysis

↓

Policy Evaluation

↓

Security Report

↓

PASS / FAIL
```

The image is evaluated against enterprise security policies before it is pushed to the container registry

---

# Configuration

Aqua Security provides centralized policy management for securing containers, Kubernetes clusters, registries, and cloud-native workloads.

Configuration typically includes:

- Image Scanning Policies
- Runtime Policies
- Admission Control
- Compliance Policies
- Registry Integration
- Kubernetes Integration
- Notifications
- Role-Based Access Control (RBAC)

A centralized configuration ensures consistent security across all environments.

---

# Aqua Components Configuration

```text
Aqua Console

↓

Security Policies

├── Image Policies

├── Runtime Policies

├── Compliance Policies

├── Admission Policies

└── Notifications
```

All security policies are managed through the Aqua Console.

---

# Configuration Files

Example deployment.

```text
values.yaml

console-values.yaml

enforcer-values.yaml

scanner-values.yaml
```

Helm values files should be stored in version control.

---

# Configure Aqua Console

Example.

```bash
helm upgrade aqua-console aqua/console \
-f console-values.yaml \
-n aqua
```

Verify.

```bash
kubectl get pods -n aqua
```

---

# Configure Aqua Enforcer

Example.

```bash
helm upgrade aqua-enforcer aqua/enforcer \
-f enforcer-values.yaml \
-n aqua
```

The Enforcer automatically begins monitoring workloads after deployment.

---

# Image Scanning Policies

Image scanning policies determine which vulnerabilities block deployments.

Typical policy.

```text
Critical

↓

Block Deployment

High

↓

Block Deployment

Medium

↓

Warning

Low

↓

Informational
```

Production environments should fail deployments containing Critical vulnerabilities.

---

# Runtime Policies

Runtime policies define the expected behaviour of running containers.

Examples.

- Allowed Processes
- Allowed Network Connections
- File Access
- Package Installation
- Privilege Escalation
- Shell Execution

Workflow.

```text
Running Container

↓

Runtime Event

↓

Policy Evaluation

↓

Allow / Alert / Block
```

---

# Admission Control

Admission Control validates Kubernetes workloads before they are created.

```text
kubectl apply

↓

Admission Controller

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

Admission policies help prevent insecure workloads from entering the cluster.

---

# Registry Integration

Aqua can continuously monitor container registries.

Supported registries include:

- Amazon ECR
- Azure Container Registry
- Google Artifact Registry
- Docker Hub
- JFrog Artifactory
- Harbor

Workflow.

```text
Container Registry

↓

New Image

↓

Automatic Scan

↓

Policy Evaluation

↓

Report
```

---

# Kubernetes Integration

Aqua integrates directly with Kubernetes.

```text
Kubernetes API

↓

Aqua Console

↓

Enforcer

↓

Runtime Protection
```

The Console manages policies while Enforcers enforce them on every node.

---

# Compliance Policies

Compliance policies verify adherence to industry standards.

Examples.

- CIS Kubernetes Benchmark
- CIS Docker Benchmark
- PCI DSS
- HIPAA
- NIST
- SOC 2

Workflow.

```text
Cluster

↓

Compliance Scan

↓

Violations

↓

Compliance Report
```

---

# Runtime Profiles

Runtime Profiles learn normal container behaviour.

```text
Container

↓

Learning Mode

↓

Observed Processes

↓

Runtime Profile

↓

Enforcement
```

Profiles reduce false positives by defining expected behaviour.

---

# Malware Protection

Aqua scans workloads for malware.

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

Known malware signatures are identified before deployment.

---

# Secret Detection

Aqua detects secrets stored inside container images.

Examples.

```text
AWS Keys

Azure Keys

Database Passwords

GitHub Tokens

Private Keys

JWT Secrets
```

Secrets should never be included in production container images.

---

# Notifications

Aqua supports multiple notification channels.

```text
Aqua Console

↓

Alert

├── Email

├── Slack

├── Webhook

├── SIEM

└── Microsoft Teams
```

Security teams should receive Critical alerts immediately.

---

# Role-Based Access Control (RBAC)

Enterprise deployments should restrict access based on job responsibilities.

| Role | Permissions |
|------|-------------|
| Administrator | Full Access |
| Security Team | Policy Management |
| DevOps Engineer | Image Scans |
| Developer | View Reports |
| Auditor | Read Only |

Apply the principle of least privilege to every role.

---

# Authentication

Aqua supports enterprise authentication providers.

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

Aqua Console
```

Enterprise authentication simplifies user management and auditing.

---

# Audit Logging

Every important action is recorded.

Examples.

```text
User Login

Policy Change

Image Scan

Deployment Decision

Runtime Alert

Compliance Scan
```

Audit logs should be retained according to organizational compliance requirements.

---

# Enterprise Best Practices

- Store Helm values files in version control.
- Block deployments with Critical vulnerabilities.
- Enable Admission Control for production clusters.
- Scan all connected container registries automatically.
- Create Runtime Profiles for production workloads.
- Enable compliance scanning for Kubernetes clusters.
- Integrate enterprise authentication providers.
- Forward alerts to SIEM and collaboration platforms.
- Review RBAC permissions regularly.
- Audit policy changes and administrative activities.

===

# Container Image Scanning

Aqua Security scans container images before they are deployed.

Workflow.

```text
Developer

↓

Docker Build

↓

Container Image

↓

Aqua Scanner

↓

Vulnerability Analysis

↓

Policy Evaluation

↓

PASS / FAIL
```

Image scanning prevents vulnerable containers from reaching production.

---

# Image Assurance Policies

Image Assurance enforces security policies before deployment.

Example policy.

```text
Critical Vulnerabilities

↓

Block

High Vulnerabilities

↓

Block

Medium Vulnerabilities

↓

Warn

Low Vulnerabilities

↓

Allow
```

Only compliant images should be promoted.

---

# Image Scan Workflow

```text
Docker Image

↓

Operating System Packages

↓

Application Libraries

↓

Language Dependencies

↓

Secrets

↓

Malware

↓

Compliance Checks

↓

Security Report
```

Aqua performs multiple security checks in a single scan.

---

# Registry Scanning

Container registries are continuously monitored.

```text
Developer

↓

Push Image

↓

Amazon ECR

↓

Automatic Aqua Scan

↓

Security Report
```

Supported registries.

- Amazon ECR
- Azure Container Registry
- Google Artifact Registry
- Harbor
- JFrog Artifactory
- Docker Hub

---

# Vulnerability Detection

Aqua identifies vulnerabilities across multiple layers.

Examples.

```text
Operating System Packages

↓

Ubuntu

↓

OpenSSL

↓

glibc

↓

curl
```

```text
Application Packages

↓

Java

↓

Node.js

↓

Python

↓

Go

↓

.NET
```

Every package is compared against vulnerability databases.

---

# Malware Detection

Container images are inspected for malware before deployment.

Workflow.

```text
Container Image

↓

File Analysis

↓

Threat Intelligence

↓

Malware Detection

↓

Alert
```

Known malware signatures and suspicious binaries are identified automatically.

---

# Secret Scanning

Images are scanned for embedded credentials.

Examples.

```text
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

DATABASE_PASSWORD

JWT_SECRET

PRIVATE_KEY

GITHUB_TOKEN
```

Embedded secrets should be removed before publishing the image.

---

# Software Supply Chain Security

Aqua validates software supply chain integrity.

```text
Source Code

↓

Dependencies

↓

Container Build

↓

Image Scan

↓

Image Assurance

↓

Registry

↓

Deployment
```

Supply chain validation reduces the risk of compromised software reaching production.

---

# Runtime Protection

Runtime protection continues after deployment.

Workflow.

```text
Container Running

↓

Process Execution

↓

Network Activity

↓

File Access

↓

Runtime Policy

↓

Alert
```

Runtime monitoring complements image scanning.

---

# Runtime Behaviour Monitoring

Aqua observes container activity continuously.

Examples.

- New processes
- Shell execution
- File modifications
- Privilege escalation
- Unexpected network connections

Workflow.

```text
Container

↓

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

Production containers should remain immutable.

Drift detection identifies changes after deployment.

```text
Approved Image

↓

Running Container

↓

Unexpected Package

↓

Drift Detected

↓

Alert
```

Examples.

- New binaries
- Modified configuration
- Installed packages
- Changed permissions

---

# Process Whitelisting

Runtime policies define approved processes.

Example.

```text
Approved

↓

java

↓

nginx

↓

node

↓

python
```

Unexpected processes generate alerts.

---

# File Integrity Monitoring

Critical files are continuously monitored.

Examples.

```text
/etc/passwd

/etc/shadow

/etc/ssl

/app/config

/root/.ssh
```

Workflow.

```text
File Change

↓

Policy Evaluation

↓

Alert
```

---

# Network Protection

Aqua monitors container network activity.

Examples.

```text
Outbound Connection

↓

Unknown IP

↓

Policy Check

↓

Alert
```

Typical events.

- Unexpected Internet access
- Communication with blocked IPs
- Suspicious ports
- Lateral movement

---

# Kubernetes Workload Protection

Aqua secures Kubernetes resources.

Examples.

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

Policy Validation

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

API Server

↓

Aqua Admission Controller

↓

Image Assurance

↓

Security Policy

↓

Approved?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Deploy     Reject
```

Only approved workloads enter the cluster.

---

# Compliance Monitoring

Aqua continuously evaluates Kubernetes environments against compliance frameworks.

Examples.

- CIS Kubernetes Benchmark
- CIS Docker Benchmark
- PCI DSS
- HIPAA
- NIST
- SOC 2

Workflow.

```text
Cluster

↓

Compliance Scan

↓

Violations

↓

Compliance Report
```

---

# Risk-Based Prioritization

Not all vulnerabilities present the same level of risk.

Priority model.

```text
Critical

↓

Immediate Remediation

High

↓

Next Release

Medium

↓

Planned Fix

Low

↓

Monitor
```

Risk prioritization helps teams focus on the most significant issues first.

---

# Enterprise Best Practices

- Scan every container image before publishing.
- Enable automatic registry scanning.
- Block images containing Critical vulnerabilities.
- Detect malware and embedded secrets before deployment.
- Enable runtime protection on every Kubernetes node.
- Monitor container drift continuously.
- Enforce Admission Controller policies.
- Maintain immutable container images.
- Enable continuous compliance monitoring.
- Prioritize remediation based on business risk and vulnerability severity.

---

# Jenkins Integration

Aqua Security integrates with Jenkins to scan container images before they are pushed to the container registry.

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

Aqua Image Scan

↓

Policy Validation

↓

Amazon ECR

↓

Deploy
```

Only images that satisfy enterprise security policies should be published.

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

        stage('Aqua Scan') {

            steps {

                sh '''

                aqua scan \
                --image $IMAGE

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

Aqua Scan

↓

Push Image

↓

Deploy
```

---

# Production GitHub Actions Workflow

```yaml
name: Aqua-Scan

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

    - name: Aqua Scan

      run: |

        aqua scan \
        --image payment-service:${{ github.sha }}
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

aqua-scan:

  stage: security

  script:

    - aqua scan \
      --image payment-service:$CI_COMMIT_SHA
```

---

# Amazon ECR Integration

Aqua continuously monitors Amazon ECR repositories.

Workflow.

```text
Docker Build

↓

Amazon ECR

↓

Automatic Aqua Scan

↓

Policy Evaluation

↓

Security Report
```

New images are scanned automatically after being pushed.

---

# Azure Container Registry Integration

```text
Docker Build

↓

Azure Container Registry

↓

Automatic Scan

↓

Report
```

Registry integration enables continuous monitoring of newly published images.

---

# Google Artifact Registry Integration

```text
Docker Build

↓

Google Artifact Registry

↓

Image Scan

↓

Compliance Validation
```

Images remain under continuous security monitoring after publication.

---

# JFrog Artifactory Integration

Aqua can monitor container images stored in JFrog Artifactory.

Workflow.

```text
Docker Build

↓

JFrog Artifactory

↓

Image Scan

↓

Policy Check

↓

Report
```

Every newly published image is evaluated against enterprise security policies.

---

# Kubernetes Admission Controller

The Admission Controller validates Kubernetes workloads before they are scheduled.

Workflow.

```text
kubectl apply

↓

API Server

↓

Admission Controller

↓

Image Verified?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Deploy     Reject
```

This prevents vulnerable workloads from entering the cluster.

---

# Runtime Policy Enforcement

Runtime policies remain active after deployment.

```text
Container

↓

Runtime Event

↓

Aqua Enforcer

↓

Policy Check

↓

Allow

or

Block

or

Alert
```

Examples include:

- Shell execution
- Privilege escalation
- Sensitive file access
- Unexpected network traffic

---

# Image Signing Verification

Trusted images should be digitally signed before deployment.

Workflow.

```text
Container Image

↓

Digital Signature

↓

Signature Validation

↓

Deployment Approved
```

Unsigned or tampered images should be rejected.

---

# Software Bill of Materials (SBOM)

Aqua generates SBOM data for container images.

```text
Container Image

↓

Package Discovery

↓

Dependency Inventory

↓

SBOM

↓

Compliance Report
```

SBOMs improve software supply chain visibility.

---

# Compliance Reporting

Compliance reports summarize policy violations.

Typical report.

```text
Compliance Report

├── CIS Controls

├── PCI DSS

├── HIPAA

├── NIST

├── SOC 2

└── Recommendations
```

Reports should be archived for audits.

---

# Security Dashboard

The Aqua Console provides centralized visibility.

Dashboard widgets typically include:

- Vulnerabilities by Severity
- Runtime Alerts
- Compliance Status
- Registry Scan Results
- Policy Violations
- Malware Findings
- Secret Detection
- Image Drift
- Kubernetes Risks
- Active Enforcers

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

Aqua Image Scan

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

Aqua Runtime Protection

↓

Production
```

Aqua Security provides protection across both build-time and runtime stages.

---

# Enterprise Best Practices

- Scan every container image before pushing it to a registry.
- Enable continuous registry scanning.
- Integrate image scanning into every CI/CD pipeline.
- Enforce Admission Controller policies in production clusters.
- Verify image signatures before deployment.
- Generate SBOMs for all production images.
- Enable runtime protection on every Kubernetes node.
- Archive compliance and vulnerability reports.
- Integrate alerts with enterprise SIEM platforms.
- Keep Aqua Security components and policies updated regularly.