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

