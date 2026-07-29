# DevSecOps for Amazon EKS

## Introduction

Amazon Elastic Kubernetes Service (EKS) is AWS's managed Kubernetes service that simplifies Kubernetes cluster management by operating the control plane while allowing organizations to manage worker nodes and workloads.

DevSecOps for Amazon EKS integrates security controls across AWS infrastructure, Kubernetes, CI/CD, GitOps, networking, runtime monitoring, and compliance to deliver secure production workloads.

---

# Why Companies Use Amazon EKS

Amazon EKS provides a highly available, scalable, and managed Kubernetes platform for enterprise applications.

## Benefits

- Managed Kubernetes Control Plane
- High Availability
- Automatic Control Plane Updates
- AWS IAM Integration
- Amazon VPC Networking
- Native AWS Security Services
- Multi-AZ Deployment
- GitOps Integration
- Enterprise Scalability
- Compliance Support

---

# Amazon EKS Architecture

```text
                    Internet
                        │
                        ▼
                 AWS Load Balancer
                        │
                        ▼
                 Amazon VPC
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
      Public AZ      Private AZ     Private AZ
         │              │              │
         ▼              ▼              ▼
     EKS Nodes      EKS Nodes      EKS Nodes
         │              │              │
         ▼              ▼              ▼
       Pods           Pods           Pods
```

High availability is achieved by distributing workloads across multiple Availability Zones.

---

# Enterprise DevSecOps Architecture

```text
Developer

↓

Git Repository

↓

Jenkins / GitHub Actions / GitLab

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

Generate SBOM

↓

Cosign

↓

Amazon ECR

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Falco

↓

Prometheus

↓

Grafana

↓

Production
```

Every stage contributes to a secure software delivery pipeline.

---

# Amazon EKS Shared Responsibility

```text
AWS

├── Physical Data Centers

├── Networking

├── Control Plane

├── Hypervisor

└── Managed Services

Customer

├── Worker Nodes

├── IAM Policies

├── Applications

├── Containers

├── RBAC

├── Secrets

├── Network Policies

└── Monitoring
```

Both AWS and customers have security responsibilities.

---

# Amazon EKS Security Layers

```text
Application Security

↓

Container Security

↓

Pod Security

↓

Node Security

↓

Cluster Security

↓

Network Security

↓

IAM Security

↓

AWS Infrastructure Security
```

Each layer requires dedicated security controls.

---

# Enterprise Production Pipeline

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

Security Validation

↓

Amazon ECR

↓

GitOps Repository

↓

ArgoCD

↓

Amazon EKS

↓

Runtime Monitoring

↓

Production
```

Security validation occurs before deployment to EKS.

---

# Amazon EKS Components

| Component | Purpose |
|-----------|----------|
| EKS Control Plane | Kubernetes Management |
| Worker Nodes | Run Containers |
| Amazon ECR | Container Registry |
| IAM | Authentication |
| VPC | Network Isolation |
| Security Groups | Firewall Rules |
| Application Load Balancer | External Access |
| ArgoCD | GitOps Deployment |
| CloudTrail | API Auditing |
| Prometheus | Monitoring |

---

# Typical Production Architecture

```text
Internet

↓

Application Load Balancer

↓

Ingress Controller

↓

Service

↓

Deployment

↓

Pods

↓

Amazon RDS
```

Application traffic flows through multiple security layers before reaching workloads.

---

# Security Objectives

Enterprise EKS security focuses on:

- Secure Authentication
- Least Privilege Access
- Secure Networking
- Secure Container Images
- Runtime Protection
- Continuous Monitoring
- Compliance
- Secure Software Supply Chain

---

# Prerequisites

| Component | Purpose |
|-----------|----------|
| AWS Account | Cloud Platform |
| Amazon EKS | Kubernetes Cluster |
| Amazon ECR | Image Registry |
| kubectl | Cluster Management |
| Helm | Package Management |
| ArgoCD | GitOps |
| SonarQube | SAST |
| Trivy | Container Security |
| Cosign | Image Signing |
| Falco | Runtime Security |
| Prometheus | Monitoring |
| Grafana | Dashboards |

---

# Enterprise Security Principles

Amazon EKS security should follow these principles.

- Zero Trust
- Least Privilege
- Defense in Depth
- GitOps
- Immutable Infrastructure
- Policy as Code
- Continuous Compliance
- Secure Software Supply Chain
- Runtime Protection
- Continuous Monitoring

These principles form the foundation of enterprise-grade Amazon EKS security.