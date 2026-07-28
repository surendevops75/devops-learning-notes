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

