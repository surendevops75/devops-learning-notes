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

