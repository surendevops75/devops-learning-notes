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

---

# Authentication in Amazon EKS

Amazon EKS uses AWS Identity and Access Management (IAM) for authentication and Kubernetes RBAC for authorization.

Authentication verifies who the user is before granting access to the cluster.

---

# Authentication Flow

```text
Developer

↓

AWS IAM User / IAM Role

↓

AWS STS Token

↓

Amazon EKS API Server

↓

Authentication

↓

RBAC Authorization

↓

Kubernetes Resources
```

Authentication and authorization work together to secure cluster access.

---

# Authentication Methods

| Method | Purpose |
|---------|----------|
| IAM User | Individual administrator access |
| IAM Role | Production workloads |
| AWS SSO | Enterprise user authentication |
| IAM Roles for Service Accounts (IRSA) | Pod authentication |
| kubeconfig | Cluster access configuration |

IAM Roles are recommended over IAM Users for production.

---

# Amazon EKS IAM Architecture

```text
AWS IAM

↓

IAM User / Role

↓

STS Token

↓

Amazon EKS

↓

Kubernetes API

↓

RBAC
```

IAM controls authentication while RBAC controls permissions.

---

# Authorization using RBAC

After authentication, Kubernetes RBAC determines what actions the user or application can perform.

RBAC implements the principle of least privilege.

---

# RBAC Components

```text
User / Service Account

↓

Role

↓

RoleBinding

↓

Namespace Resources
```

Cluster-wide permissions use ClusterRole and ClusterRoleBinding.

---

# RBAC Objects

| Object | Purpose |
|---------|----------|
| Role | Namespace permissions |
| ClusterRole | Cluster-wide permissions |
| RoleBinding | Assign Role |
| ClusterRoleBinding | Assign ClusterRole |
| Service Account | Identity for Pods |

---

# Example Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: Role

metadata:

  name: pod-reader

rules:

- apiGroups: [""]

  resources: ["pods"]

  verbs:

  - get

  - list

  - watch
```

The Role grants read-only access to Pods within a namespace.

---

# Example RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

  name: developer-binding

subjects:

- kind: User

  name: developer

roleRef:

  kind: Role

  name: pod-reader

  apiGroup: rbac.authorization.k8s.io
```

The RoleBinding associates the Role with a user.

---

# IAM Roles for Service Accounts (IRSA)

IRSA allows Kubernetes Pods to securely access AWS services using IAM Roles.

Pods no longer require long-lived AWS credentials.

---

# IRSA Architecture

```text
Pod

↓

Service Account

↓

OIDC Provider

↓

IAM Role

↓

Temporary AWS Credentials

↓

AWS Service
```

IRSA is the recommended authentication mechanism for AWS services.

---

# Services Commonly Accessed Using IRSA

- Amazon S3
- Amazon DynamoDB
- Amazon SQS
- Amazon SNS
- AWS Secrets Manager
- AWS Systems Manager Parameter Store
- Amazon CloudWatch

Each application should use its own IAM Role.

---

# Service Account Example

```yaml
apiVersion: v1

kind: ServiceAccount

metadata:

  name: payment-sa

  annotations:

    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/payment-role
```

The annotation links the Service Account to an IAM Role.

---

# Secure Access to AWS Services

```text
Application

↓

Pod

↓

Service Account

↓

IAM Role

↓

Temporary Credentials

↓

Amazon S3
```

Temporary credentials eliminate the need for embedded AWS keys.

---

# AWS Secrets Manager Integration

Sensitive information should be stored outside Kubernetes whenever possible.

Examples include:

- Database Passwords
- API Keys
- Tokens
- Certificates
- Encryption Keys

AWS Secrets Manager provides secure storage and automatic rotation.

---

# Secret Retrieval Flow

```text
Application

↓

Pod

↓

IRSA

↓

AWS Secrets Manager

↓

Retrieve Secret

↓

Application
```

Applications retrieve secrets at runtime instead of storing them in container images.

---

# Secure kubeconfig

The kubeconfig file contains cluster access information.

Protect it by:

- Restricting file permissions
- Avoiding source control
- Using short-lived credentials
- Rotating credentials regularly

---

# Multi-Account Access

Large enterprises commonly separate environments across AWS accounts.

```text
AWS Organization

├── Development Account

├── Testing Account

├── Staging Account

└── Production Account
```

This reduces the blast radius of security incidents.

---

# Enterprise Access Architecture

```text
Developer

↓

AWS IAM Identity Center

↓

IAM Role

↓

Amazon EKS

↓

RBAC

↓

Namespace

↓

Application
```

Access is centrally managed while permissions remain granular.

---

# Enterprise Best Practices

- Use IAM Roles instead of IAM Users whenever possible.
- Implement IAM Roles for Service Accounts (IRSA).
- Never store AWS Access Keys inside Pods.
- Grant least-privilege IAM permissions.
- Separate development, staging, and production accounts.
- Protect kubeconfig files from unauthorized access.
- Rotate IAM credentials regularly.
- Use AWS Secrets Manager for sensitive data.
- Audit IAM and RBAC permissions periodically.
- Review unused roles and bindings regularly.

---

# Amazon VPC Integration

Amazon EKS runs inside an Amazon Virtual Private Cloud (VPC), providing network isolation for Kubernetes clusters.

The VPC forms the network foundation for worker nodes and applications.

---

# Amazon EKS Networking Architecture

```text
Internet

↓

Application Load Balancer

↓

Amazon VPC

├── Public Subnets

│      │

│      └── NAT Gateway

│

└── Private Subnets

       │

       ├── Worker Nodes

       │      │

       │      └── Pods

       │

       └── Worker Nodes

              │

              └── Pods
```

Production workloads should run in private subnets.

---

# Private and Public Subnets

| Subnet | Purpose |
|---------|----------|
| Public Subnet | Load Balancers, NAT Gateway |
| Private Subnet | Worker Nodes and Pods |

Worker Nodes should not have public IP addresses in production.

---

# Security Groups

Security Groups act as virtual firewalls for EKS resources.

They control inbound and outbound traffic.

Examples.

- Control Plane Security Group
- Worker Node Security Group
- Load Balancer Security Group

---

# Security Group Architecture

```text
Internet

↓

Application Load Balancer

↓

Security Group

↓

Worker Nodes

↓

Pods
```

Only required ports should be opened.

---

# Network ACLs

Network ACLs provide subnet-level firewall protection.

```text
Amazon VPC

↓

Subnet

↓

Network ACL

↓

Worker Nodes
```

Security Groups and Network ACLs provide layered network security.

---

# Kubernetes Network Policies

Network Policies control communication between Pods.

Without Network Policies, Pods can communicate freely.

---

# Network Policy Flow

```text
Frontend Pod

↓

Allowed

↓

Backend Pod

↓

Allowed

↓

Database

✖

Other Pods
```

Only approved communication paths should be allowed.

---

# Default Deny Strategy

Begin with a default-deny policy.

```text
All Traffic

↓

Blocked

↓

Explicit Rules

↓

Allowed Traffic
```

This follows the principle of least privilege.

---

# Ingress Security

Ingress Controllers expose applications securely.

Recommended controls.

- HTTPS Only
- TLS Certificates
- AWS WAF
- Rate Limiting
- Authentication
- Access Logging

---

# TLS Architecture

```text
Client

↓

HTTPS

↓

Application Load Balancer

↓

TLS Termination

↓

Ingress

↓

Application
```

Encrypt all traffic entering the cluster.

---

# AWS WAF Integration

AWS WAF protects internet-facing applications.

Common protections.

- SQL Injection
- Cross-Site Scripting
- IP Reputation
- Rate Limiting
- Bot Protection

AWS WAF should be associated with the Application Load Balancer.

---

# Secrets Management

Sensitive information should never be stored in container images or Git repositories.

Examples.

- Database Passwords
- API Keys
- Tokens
- Certificates
- Encryption Keys

---

# AWS Secrets Manager Architecture

```text
Application

↓

Pod

↓

IRSA

↓

AWS Secrets Manager

↓

Retrieve Secret

↓

Application
```

Secrets are retrieved securely during application runtime.

---

# AWS Parameter Store

AWS Systems Manager Parameter Store is suitable for storing application configuration and non-sensitive parameters.

Examples.

- Configuration Values
- URLs
- Feature Flags
- Environment Variables

Sensitive values can also be encrypted using AWS KMS.

---

# Encryption at Rest

Secrets stored in Kubernetes should be encrypted using AWS Key Management Service (KMS).

```text
Secret

↓

AWS KMS

↓

Encrypted Secret

↓

etcd
```

Encryption protects sensitive data stored in the cluster.

---

# Pod Security Admission

Pod Security Admission enforces security standards before Pods are created.

Recommended profile.

```text
Restricted
```

Restricted provides the strongest default security for production workloads.

---

# Secure Security Context

Production Pods should include a SecurityContext.

Example.

```yaml
securityContext:

  runAsNonRoot: true

  allowPrivilegeEscalation: false

  readOnlyRootFilesystem: true

  capabilities:

    drop:

    - ALL
```

Containers should operate with the minimum required privileges.

---

# Image Security

Deploy only trusted container images.

Production pipeline.

```text
Docker Build

↓

Trivy Scan

↓

Generate SBOM

↓

Cosign Sign

↓

Amazon ECR

↓

Admission Verification

↓

Deploy
```

Unsigned or vulnerable images should never reach production.

---

# Runtime Security

Runtime security detects suspicious behaviour after workloads are deployed.

Common tools.

- Falco
- Amazon GuardDuty for EKS

Runtime monitoring complements preventive security controls.

---

# Runtime Monitoring Flow

```text
Application

↓

Container

↓

Falco

↓

Security Alert

↓

SOC Team
```

Runtime events should be integrated with enterprise alerting platforms.

---

# Enterprise Best Practices

- Deploy worker nodes only in private subnets.
- Restrict Security Group rules to required ports.
- Apply default-deny Network Policies.
- Encrypt all traffic using TLS.
- Protect internet-facing applications with AWS WAF.
- Store secrets in AWS Secrets Manager.
- Encrypt Kubernetes Secrets using AWS KMS.
- Enforce the Restricted Pod Security profile.
- Scan, sign, and verify every container image.
- Enable runtime monitoring using Falco and GuardDuty for EKS.