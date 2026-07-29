# DevSecOps for Kubernetes

## Introduction

Kubernetes has become the standard platform for running containerized applications. While it provides scalability and resilience, it also introduces new security challenges across the control plane, worker nodes, containers, networking, storage, and workloads.

DevSecOps for Kubernetes integrates security controls into the entire Kubernetes lifecycle—from infrastructure provisioning and application deployment to runtime monitoring and compliance.

---

# Why Kubernetes Security Matters

A Kubernetes cluster contains multiple components that can become attack targets if not properly secured.

Examples include:

- Kubernetes API Server
- etcd Database
- Worker Nodes
- Pods
- Containers
- Secrets
- Service Accounts
- Container Images
- Network Policies

A compromise in any of these components can affect the entire cluster.

---

# Kubernetes Attack Surface

```text
                 Internet
                     │
                     ▼
             Ingress Controller
                     │
                     ▼
                Kubernetes API
                     │
      ┌──────────────┼──────────────┐
      ▼              ▼              ▼
   Worker 1       Worker 2       Worker 3
      │              │              │
      ▼              ▼              ▼
   Pod A          Pod B          Pod C
      │              │              │
      ▼              ▼              ▼
 Containers     Containers     Containers
```

Every layer must be protected using defense-in-depth.

---

# DevOps vs DevSecOps for Kubernetes

## Traditional Kubernetes Deployment

```text
Developer

↓

Docker Build

↓

Push Image

↓

Deploy

↓

Production

↓

Security Scan
```

Security occurs after deployment.

---

## Secure Kubernetes Deployment

```text
Developer

↓

Source Code Scan

↓

Dependency Scan

↓

Secret Scan

↓

Docker Build

↓

Image Scan

↓

SBOM

↓

Image Signing

↓

GitOps

↓

Admission Validation

↓

Deploy

↓

Runtime Security

↓

Production
```

Security is enforced throughout the deployment lifecycle.

---

# Enterprise Kubernetes DevSecOps Architecture

```text
Developer

↓

Git Repository

↓

CI Pipeline

↓

SonarQube

↓

Dependency Check

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

Cosign

↓

Amazon ECR

↓

GitOps Repository

↓

ArgoCD

↓

Admission Controller

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

Multiple security controls protect every deployment stage.

---

# Kubernetes Security Layers

```text
Application Security

↓

Container Security

↓

Pod Security

↓

Namespace Security

↓

Network Security

↓

Node Security

↓

Cluster Security

↓

Cloud Security
```

Security should be implemented at every layer.

---

# Kubernetes Shared Responsibility

```text
Cloud Provider

├── Physical Infrastructure

├── Networking

├── Compute

└── Managed Control Plane

Customer

├── Applications

├── Containers

├── RBAC

├── Secrets

├── Policies

├── Images

└── Workloads
```

Understanding ownership is critical for securing managed Kubernetes platforms.

---

# Enterprise Kubernetes Security Pipeline

```text
Developer

↓

Git Push

↓

CI Pipeline

↓

Source Code Scan

↓

Secret Detection

↓

IaC Validation

↓

Container Build

↓

Container Scan

↓

Image Signing

↓

Container Registry

↓

GitOps

↓

Admission Controller

↓

Deploy

↓

Runtime Monitoring
```

Every stage introduces additional security validation.

---

# Kubernetes Security Objectives

Enterprise Kubernetes security focuses on:

- Protecting workloads
- Preventing privilege escalation
- Securing cluster communication
- Protecting sensitive data
- Enforcing deployment policies
- Detecting runtime threats
- Meeting compliance requirements

---

# Core Kubernetes Security Components

| Component | Purpose |
|-----------|---------|
| RBAC | Authorization |
| Service Accounts | Pod Identity |
| Secrets | Sensitive Data |
| ConfigMaps | Configuration |
| Network Policies | Network Isolation |
| Pod Security Admission | Pod Security Enforcement |
| Admission Controllers | Policy Enforcement |
| Audit Logs | Compliance |
| Image Registry | Trusted Images |
| Runtime Security | Threat Detection |

---

# Kubernetes Security Lifecycle

```text
Plan

↓

Develop

↓

Build

↓

Scan

↓

Sign

↓

Store

↓

Deploy

↓

Validate

↓

Monitor

↓

Respond
```

Security continues after deployment through continuous monitoring.

---

# Security Controls by SDLC Phase

| SDLC Phase | Security Control |
|------------|------------------|
| Planning | Security Requirements |
| Development | Secure Coding |
| Build | SAST |
| Package | Image Scanning |
| Release | Image Signing |
| Deploy | Admission Policies |
| Runtime | Falco |
| Operations | Monitoring & Auditing |

Every SDLC phase contributes to Kubernetes security.

---

# Prerequisites

| Component | Purpose |
|-----------|----------|
| Kubernetes Cluster | Container Platform |
| Amazon EKS | Managed Kubernetes |
| Docker | Container Runtime |
| Helm | Package Manager |
| kubectl | Cluster Management |
| ArgoCD | GitOps |
| SonarQube | SAST |
| Trivy | Image Scanning |
| Cosign | Image Signing |
| Falco | Runtime Security |
| Prometheus | Monitoring |
| Grafana | Dashboards |

---

# Enterprise Security Principles

Successful Kubernetes security is based on the following principles.

- Least Privilege
- Zero Trust
- Defense in Depth
- Immutable Infrastructure
- GitOps
- Policy as Code
- Continuous Monitoring
- Continuous Compliance
- Secure Software Supply Chain
- Automated Security Validation

These principles form the foundation of an enterprise-grade Kubernetes DevSecOps strategy.

---

# Kubernetes Authentication

Authentication verifies the identity of users, applications, and services before allowing access to the Kubernetes API Server.

Supported authentication methods.

- X.509 Certificates
- Service Accounts
- OpenID Connect (OIDC)
- IAM Authentication
- Webhook Authentication

---

# Authentication Flow

```text
User

↓

kubectl

↓

Authentication

↓

API Server

↓

Authenticated?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

RBAC      Reject
```

Authentication always occurs before authorization.

---

# Kubernetes Authorization

After authentication, Kubernetes determines what actions the identity can perform.

Authorization methods.

- RBAC
- ABAC
- Webhook Authorization

RBAC is the recommended authorization mechanism.

---

# RBAC Architecture

```text
User

↓

RoleBinding

↓

Role

↓

Permissions

↓

Namespace Resources
```

RBAC limits access based on the principle of least privilege.

---

# RBAC Components

| Component | Purpose |
|-----------|---------|
| Role | Namespace Permissions |
| ClusterRole | Cluster-wide Permissions |
| RoleBinding | Assign Role |
| ClusterRoleBinding | Assign ClusterRole |

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

This Role provides read-only access to Pods.

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

RoleBindings associate identities with Roles.

---

# Service Accounts

Pods use Service Accounts instead of user accounts.

```text
Application Pod

↓

Service Account

↓

API Server

↓

Cluster Resources
```

Each workload should use a dedicated Service Account.

---

# Create Service Account

Example.

```yaml
apiVersion: v1

kind: ServiceAccount

metadata:

  name: payment-service
```

Avoid using the default Service Account for production workloads.

---

# Least Privilege Access

```text
Application

↓

Dedicated Service Account

↓

Minimal RBAC Permissions

↓

Required Resources
```

Grant only the permissions required by the application.

---

# Kubernetes Secrets

Secrets store sensitive information.

Examples.

- Database Passwords
- API Tokens
- Certificates
- SSH Keys
- OAuth Tokens

Secrets should never be stored inside container images.

---

# Secret Example

```yaml
apiVersion: v1

kind: Secret

metadata:

  name: database-secret

type: Opaque

stringData:

  username: admin

  password: StrongPassword123
```

Use external secret management for production whenever possible.

---

# Secret Access Flow

```text
Secret

↓

API Server

↓

Pod

↓

Application
```

Only authorized Pods should be allowed to access Secrets.

---

# ConfigMaps

ConfigMaps store non-sensitive application configuration.

Example.

```yaml
apiVersion: v1

kind: ConfigMap

metadata:

  name: application-config

data:

  LOG_LEVEL: INFO

  REGION: ap-south-1
```

Separate configuration from application code.

---

# Namespace Isolation

Namespaces isolate workloads inside the cluster.

Example.

```text
Cluster

├── development

├── testing

├── staging

└── production
```

Use separate namespaces for different environments.

---

# Resource Quotas

Resource Quotas prevent resource exhaustion.

Example.

```yaml
apiVersion: v1

kind: ResourceQuota

metadata:

  name: production-quota

spec:

  hard:

    pods: "100"

    requests.cpu: "50"

    requests.memory: 100Gi
```

Resource Quotas improve cluster stability.

---

# Limit Ranges

LimitRanges define default resource requests and limits.

Example.

```yaml
apiVersion: v1

kind: LimitRange

metadata:

  name: resource-limits

spec:

  limits:

  - default:

      cpu: "500m"

      memory: "512Mi"

    defaultRequest:

      cpu: "250m"

      memory: "256Mi"

    type: Container
```

Every container should have resource requests and limits.

---

# Enterprise Best Practices

- Use RBAC instead of broad administrator access.
- Create dedicated Service Accounts for every workload.
- Never use the default Service Account in production.
- Store sensitive information in Secrets, not ConfigMaps.
- Use external secret managers for production environments.
- Separate workloads using Namespaces.
- Configure ResourceQuotas for every namespace.
- Configure LimitRanges to prevent resource abuse.
- Apply least-privilege access across the cluster.
- Audit RBAC permissions regularly.

---

# Pod Security Admission

Pod Security Admission (PSA) enforces security standards before Pods are created.

Security levels.

- Privileged
- Baseline
- Restricted

Restricted is recommended for production workloads.

---

# Pod Security Admission Flow

```text
Deployment

↓

API Server

↓

Pod Security Admission

↓

Policy Validation

↓

Allowed?

      │

 ┌────┴─────┐

 ▼          ▼

Yes         No

 │           │

Create     Reject
```

Every Pod is validated before scheduling.

---

# Namespace Security Labels

Enable Pod Security Admission using namespace labels.

Example.

```yaml
apiVersion: v1

kind: Namespace

metadata:

  name: production

  labels:

    pod-security.kubernetes.io/enforce: restricted
```

The restricted profile enforces strong security defaults.

---

# Secure Security Context

Every production container should define a SecurityContext.

Example.

```yaml
securityContext:

  runAsNonRoot: true

  runAsUser: 1000

  allowPrivilegeEscalation: false

  readOnlyRootFilesystem: true

  capabilities:

    drop:

      - ALL
```

These settings significantly reduce container privileges.

---

# Secure Pod Example

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: payment

spec:

  securityContext:

    runAsNonRoot: true

  containers:

  - name: payment

    image: company/payment:v1

    securityContext:

      allowPrivilegeEscalation: false

      readOnlyRootFilesystem: true

      capabilities:

        drop:

        - ALL
```

Secure defaults should be applied to every workload.

---

# Linux Capabilities

Containers should only receive the capabilities they require.

```text
Default Linux

↓

Many Capabilities

↓

Container

↓

Drop Unused Capabilities

↓

Minimal Privileges
```

Dropping unnecessary capabilities limits attack opportunities.

---

# Privileged Containers

Avoid running privileged containers.

```yaml
securityContext:

  privileged: false
```

Privileged containers can access host resources and should only be used when absolutely necessary.

---

# Running as Root

Avoid running applications as the root user.

```yaml
securityContext:

  runAsNonRoot: true

  runAsUser: 1000
```

Running as a non-root user limits the impact of container compromise.

---

# Read-Only Root Filesystem

Production containers should use read-only root filesystems whenever possible.

Example.

```yaml
securityContext:

  readOnlyRootFilesystem: true
```

Attackers cannot easily modify application files.

---

# Seccomp Profiles

Seccomp restricts Linux system calls.

```yaml
securityContext:

  seccompProfile:

    type: RuntimeDefault
```

The RuntimeDefault profile is recommended for production.

---

# AppArmor

AppArmor limits application capabilities at the operating system level.

```text
Application

↓

AppArmor Profile

↓

Allowed Operations

↓

Linux Kernel
```

Use AppArmor where supported by the Kubernetes distribution.

---

# SELinux

SELinux provides mandatory access control for containers.

```text
Container

↓

SELinux Policy

↓

Kernel

↓

Protected Resources
```

SELinux improves workload isolation on supported Linux distributions.

---

# Network Policies

Network Policies control Pod-to-Pod communication.

Without policies, every Pod can communicate with every other Pod.

---

# Default Communication

```text
Pod A

↔

Pod B

↔

Pod C

↔

Database
```

This unrestricted communication increases the attack surface.

---

# Network Policy Architecture

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

All Other Pods
```

Only approved communication paths should be allowed.

---

# Network Policy Example

```yaml
apiVersion: networking.k8s.io/v1

kind: NetworkPolicy

metadata:

  name: backend-policy

spec:

  podSelector:

    matchLabels:

      app: backend

  policyTypes:

  - Ingress
```

Start with a default-deny policy and explicitly allow required traffic.

---

# Ingress Security

Secure external traffic entering the cluster.

Recommendations.

- HTTPS Only
- TLS Certificates
- WAF Integration
- Rate Limiting
- Authentication
- Authorization

Ingress should terminate TLS using trusted certificates.

---

# Enterprise Best Practices

- Enforce the Restricted Pod Security Admission profile.
- Never run privileged containers unless required.
- Always run containers as non-root users.
- Disable privilege escalation.
- Drop unnecessary Linux capabilities.
- Enable read-only root filesystems.
- Use RuntimeDefault Seccomp profiles.
- Implement Network Policies for every namespace.
- Encrypt all external communication using TLS.
- Review security policies during every deployment.

---

