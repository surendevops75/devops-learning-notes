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

---

# Logging in Amazon EKS

Centralized logging enables operations and security teams to troubleshoot applications and investigate incidents.

Production clusters should never rely on local container logs.

---

# Enterprise Logging Architecture

```text
Pods

↓

Container Logs

↓

Fluent Bit

↓

Amazon CloudWatch Logs

↓

Amazon OpenSearch / ELK

↓

Operations Team
```

Logs should be retained according to organizational compliance requirements.

---

# Components of Logging

| Component | Purpose |
|-----------|----------|
| Fluent Bit | Log Collection |
| Amazon CloudWatch Logs | Central Log Storage |
| Amazon OpenSearch / ELK | Log Analysis |
| Kibana | Visualization |
| Alerting | Incident Notification |

---

# Fluent Bit

Fluent Bit is deployed as a DaemonSet to collect logs from every worker node.

```text
Node

├── Pod A Logs

├── Pod B Logs

├── Pod C Logs

↓

Fluent Bit

↓

CloudWatch Logs
```

It forwards container logs to centralized logging platforms.

---

# Kubernetes Audit Logging

Audit Logs capture every request made to the Kubernetes API Server.

Events include:

- Authentication
- Authorization
- Resource Creation
- Resource Updates
- Resource Deletion
- RBAC Changes

Audit Logs are essential for security investigations.

---

# Audit Log Flow

```text
User

↓

Amazon EKS API Server

↓

Audit Logs

↓

CloudWatch Logs

↓

SIEM

↓

Security Team
```

Audit logs should be protected from unauthorized modification.

---

# Monitoring Amazon EKS

Continuous monitoring improves availability, reliability, and security.

Recommended monitoring stack:

- Prometheus
- Grafana
- Alertmanager

---

# Monitoring Architecture

```text
Pods

↓

Metrics

↓

Prometheus

↓

Alertmanager

↓

Grafana

↓

Operations Team
```

Monitoring provides visibility into cluster health and workload performance.

---

# Important Metrics

Infrastructure Metrics

- CPU Utilization
- Memory Usage
- Disk Usage
- Node Availability

Application Metrics

- Request Rate
- Response Time
- Error Rate
- Active Sessions

Kubernetes Metrics

- Pod Restarts
- Pending Pods
- Failed Pods
- Deployment Status

---

# Alerting Workflow

```text
Metric Threshold

↓

Prometheus

↓

Alertmanager

↓

Email / Slack / PagerDuty

↓

Operations Team
```

Alerts should prioritize actionable production issues.

---

# Runtime Security

Runtime security detects malicious activity after workloads are deployed.

Common runtime threats include:

- Reverse Shell
- Privilege Escalation
- Crypto Mining
- Unexpected Process Execution
- File Tampering

---

# Runtime Security Architecture

```text
Application

↓

Container

↓

Falco

↓

Security Events

↓

SOC Team
```

Runtime monitoring complements preventive security controls.

---

# Amazon GuardDuty for EKS

Amazon GuardDuty continuously analyzes EKS audit logs to identify suspicious activities.

Examples.

- Anonymous API Calls
- Credential Misuse
- Privilege Escalation
- Suspicious Cluster Activity
- Malware Indicators

GuardDuty provides managed threat detection for AWS environments.

---

# Runtime Detection Flow

```text
Amazon EKS

↓

Audit Logs

↓

Amazon GuardDuty

↓

Threat Detection

↓

Security Alert

↓

Security Team
```

GuardDuty helps identify threats across AWS and Kubernetes.

---

# Compliance

Enterprise EKS environments should align with industry security frameworks.

Common standards include:

- CIS Kubernetes Benchmark
- CIS AWS Foundations Benchmark
- NIST Cybersecurity Framework
- PCI DSS
- ISO 27001
- SOC 2

Compliance should be continuously validated.

---

# Compliance Architecture

```text
Infrastructure

↓

Security Controls

↓

Continuous Assessment

↓

Compliance Reports

↓

Audit
```

Automated compliance reduces operational overhead.

---

# Vulnerability Management

Security should be validated throughout the software lifecycle.

```text
Source Code

↓

SonarQube

↓

OWASP Dependency-Check

↓

Gitleaks

↓

Docker Build

↓

Trivy

↓

Amazon ECR

↓

Amazon EKS
```

Every release should pass vulnerability assessment before deployment.

---

# Disaster Recovery

Production clusters should include disaster recovery planning.

Recommendations:

- Multi-AZ Deployment
- Automated Backups
- Infrastructure as Code
- GitOps Recovery
- Restore Testing

Recovery procedures should be tested regularly.

---

# High Availability Architecture

```text
Internet

↓

Application Load Balancer

↓

Amazon EKS Control Plane

↓

Node Group (AZ-1)

↓

Node Group (AZ-2)

↓

Node Group (AZ-3)

↓

Applications
```

Distributing workloads across multiple Availability Zones improves resilience.

---

# Enterprise Best Practices

- Centralize logs using Fluent Bit and CloudWatch Logs.
- Enable Kubernetes Audit Logging.
- Monitor workloads with Prometheus and Grafana.
- Configure Alertmanager for production alerts.
- Enable Amazon GuardDuty for EKS threat detection.
- Scan every release before deployment.
- Continuously monitor compliance status.
- Deploy clusters across multiple Availability Zones.
- Test backup and disaster recovery procedures regularly.
- Integrate monitoring and security alerts with the organization's incident response process.

---

# Common Mistakes

## Mistake 1

### Running Worker Nodes in Public Subnets

**Problem**

Worker Nodes are deployed with public IP addresses.

```text
Internet

↓

Worker Node
```

**Impact**

- Increased attack surface
- Direct internet exposure
- Higher security risk

**Recommendation**

Deploy Worker Nodes only in private subnets.

---

## Mistake 2

### Using AWS Access Keys Inside Pods

**Problem**

Applications use static AWS credentials.

```text
Pod

↓

AWS Access Key

↓

AWS Service
```

**Impact**

- Credential leakage
- Long-lived secrets
- Difficult credential rotation

**Recommendation**

Use IAM Roles for Service Accounts (IRSA).

---

## Mistake 3

### Granting Excessive IAM Permissions

**Problem**

IAM Roles use wildcard permissions.

```json
{
  "Action": "*",
  "Resource": "*"
}
```

**Impact**

- Privilege escalation
- Larger blast radius
- Compliance violations

**Recommendation**

Follow the Principle of Least Privilege.

---

## Mistake 4

### Storing Secrets in Kubernetes Manifests

**Problem**

Database passwords and API keys are stored directly in YAML files.

**Impact**

- Secret exposure
- Git repository leaks
- Compliance failures

**Recommendation**

Store secrets in AWS Secrets Manager and access them using IRSA.

---

## Mistake 5

### No Runtime Monitoring

**Problem**

The cluster has no runtime threat detection.

**Impact**

- Delayed attack detection
- Limited incident visibility
- Increased recovery time

**Recommendation**

Deploy Falco and enable Amazon GuardDuty for EKS.

---

# Troubleshooting

## Scenario 1

### Pods Cannot Pull Images from Amazon ECR

**Symptoms**

```text
ImagePullBackOff
```

**Possible Causes**

- Incorrect image name
- Missing IAM permissions
- ECR authentication issue
- Repository does not exist

**Resolution**

```bash
kubectl describe pod payment

kubectl get events
```

Verify:

- ECR repository
- Node IAM Role
- Image tag
- Network connectivity

---

## Scenario 2

### Pod Cannot Access Amazon S3

**Symptoms**

Application receives Access Denied errors.

**Possible Causes**

- Missing IRSA configuration
- Incorrect IAM Role
- Missing S3 permissions

**Resolution**

```bash
kubectl describe serviceaccount payment-sa
```

Verify:

- IAM Role annotation
- OIDC provider
- IAM Policy
- Trust relationship

---

## Scenario 3

### Application Load Balancer Not Created

**Symptoms**

Ingress remains pending.

**Possible Causes**

- AWS Load Balancer Controller not installed
- Missing IAM permissions
- Incorrect IngressClass

**Resolution**

```bash
kubectl get ingress

kubectl describe ingress
```

Review controller logs and IAM configuration.

---

## Scenario 4

### Worker Nodes Fail to Join the Cluster

**Symptoms**

Nodes remain in the NotReady state.

**Possible Causes**

- Security Group configuration
- IAM Role issues
- Bootstrap failure
- Network connectivity

**Resolution**

```bash
kubectl get nodes

kubectl describe node <node-name>
```

Check:

- Worker Node IAM Role
- Security Groups
- Private subnet routing
- Bootstrap logs

---

## Scenario 5

### Pods Cannot Retrieve Secrets

**Symptoms**

Application startup fails because secrets cannot be loaded.

**Possible Causes**

- Missing IAM permissions
- Incorrect Secret name
- IRSA configuration issue
- AWS Secrets Manager access denied

**Resolution**

Verify:

- Service Account annotation
- IAM Policy
- Secret ARN
- Application configuration

---

# Production Interview Questions

## Question 1

### Why is Amazon EKS preferred over self-managed Kubernetes?

**Answer**

Amazon EKS provides a managed control plane, high availability, automatic upgrades, AWS service integration, and reduced operational overhead.

---

## Question 2

### What is IRSA?

**Answer**

IAM Roles for Service Accounts (IRSA) allows Kubernetes Pods to securely access AWS services using temporary IAM credentials instead of static AWS Access Keys.

---

## Question 3

### Why should Worker Nodes be deployed in private subnets?

**Answer**

Private subnets reduce internet exposure, improve network isolation, and strengthen overall cluster security.

---

## Question 4

### What is the purpose of Amazon ECR?

**Answer**

Amazon ECR is a managed container image registry used to securely store, scan, and distribute container images.

---

## Question 5

### How do you secure secrets in Amazon EKS?

**Answer**

Store secrets in AWS Secrets Manager, encrypt them using AWS KMS, and retrieve them securely through IRSA.

---

## Question 6

### How is external traffic secured in Amazon EKS?

**Answer**

External traffic is protected using Application Load Balancers, TLS, AWS WAF, Security Groups, and Kubernetes Ingress resources.

---

## Question 7

### What runtime security tools are commonly used with Amazon EKS?

**Answer**

Falco provides runtime threat detection, while Amazon GuardDuty for EKS analyzes audit logs to detect suspicious cluster activity.

---

## Question 8

### What monitoring stack is commonly used with Amazon EKS?

**Answer**

Prometheus collects metrics, Grafana visualizes dashboards, Alertmanager sends alerts, and CloudWatch provides AWS-native monitoring and logging.

---

## Question 9

### Why is GitOps recommended for Amazon EKS?

**Answer**

GitOps provides version-controlled deployments, automatic reconciliation, easier rollbacks, and improved auditability using tools such as ArgoCD.

---

## Question 10

### What are the core security best practices for Amazon EKS?

**Answer**

Use private subnets, IAM Roles for Service Accounts, least-privilege IAM policies, AWS Secrets Manager, Pod Security Admission, Network Policies, image scanning, runtime monitoring, centralized logging, and continuous compliance validation.

---

# Key Takeaways

- Use Amazon EKS for managed Kubernetes operations.
- Authenticate users with AWS IAM.
- Authorize access using Kubernetes RBAC.
- Use IRSA instead of static AWS credentials.
- Deploy Worker Nodes in private subnets.
- Secure external traffic using ALB, TLS, and AWS WAF.
- Store secrets in AWS Secrets Manager.
- Encrypt sensitive data using AWS KMS.
- Scan, sign, and verify every container image.
- Deploy workloads using GitOps with ArgoCD.
- Monitor clusters using Prometheus, Grafana, and CloudWatch.
- Enable Falco and Amazon GuardDuty for runtime security.
- Centralize logs for auditing and incident response.
- Continuously validate compliance against enterprise standards.

---

# Enterprise Amazon EKS DevSecOps Workflow

```text
Developer

↓

Git Push

↓

Pull Request

↓

Code Review

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

Trivy Scan

↓

Generate SBOM

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

Pod Security Admission

↓

Network Policies

↓

Falco Runtime Security

↓

Amazon GuardDuty

↓

Prometheus

↓

Grafana

↓

CloudWatch Logs

↓

Production
```

This workflow demonstrates a complete enterprise DevSecOps implementation for Amazon EKS, integrating AWS-native security services, Kubernetes security controls, secure CI/CD, GitOps, runtime protection, observability, and continuous compliance.