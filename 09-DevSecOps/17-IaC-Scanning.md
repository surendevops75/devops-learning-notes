# Infrastructure as Code (IaC) Scanning

## Introduction

Infrastructure as Code (IaC) Scanning is the process of analyzing Infrastructure as Code templates to identify security vulnerabilities, misconfigurations, and compliance violations before infrastructure is provisioned.

IaC Scanning helps organizations detect security issues early in the DevSecOps lifecycle, reducing the risk of deploying insecure cloud infrastructure.

---

## Why Do We Need IaC Scanning?

Infrastructure is increasingly created using tools like Terraform, CloudFormation, ARM Templates, and Kubernetes manifests.

Without IaC Scanning:

- Public cloud resources may be exposed
- Storage buckets become publicly accessible
- Security groups allow unrestricted access
- Encryption may be disabled
- Compliance violations reach production

IaC Scanning ensures infrastructure is secure before deployment.

---

## What is IaC Scanning?

IaC Scanning analyzes infrastructure definition files instead of deployed infrastructure.

It validates configurations against:

- Security Best Practices
- Compliance Standards
- Organizational Policies
- Cloud Security Guidelines

Unlike runtime scanning, IaC Scanning detects problems before resources are created.

---

## How It Works

```text
Developer Creates IaC

↓

Git Commit

↓

IaC Scanner

↓

Analyze Templates

↓

Check Security Rules

↓

Generate Report

↓

Fix Issues

↓

Deploy Infrastructure
```

---

## Architecture

```text
Developer

↓

Git Repository

↓

CI/CD Pipeline

↓

IaC Scanner

↓

Security Policies

↓

Security Report

↓

Terraform Apply

↓

Cloud Infrastructure
```

---

## Workflow

```text
Write Terraform Code

↓

Commit Code

↓

Run IaC Scan

↓

Review Findings

↓

Fix Issues

↓

Approve Changes

↓

Deploy Infrastructure
```

---

# Supported Infrastructure

## Terraform

Example

```text
main.tf

↓

Variables

↓

Modules

↓

IaC Scan
```

---

## AWS CloudFormation

Example

```text
Template.yaml

↓

IaC Scanner

↓

Security Validation
```

---

## Kubernetes Manifests

Example

```text
deployment.yaml

↓

IaC Scan

↓

Security Report
```

---

## Helm Charts

Example

```text
Helm Templates

↓

Render Templates

↓

IaC Scan
```

---

# Common Security Issues Detected

## Public S3 Bucket

Example

```text
S3 Bucket

↓

Public Read = True

↓

Security Risk
```

---

## Open Security Groups

Example

```text
Security Group

↓

0.0.0.0/0

↓

SSH Port 22

↓

Critical Finding
```

---

## Missing Encryption

Example

```text
EBS Volume

↓

Encryption Disabled

↓

Compliance Failure
```

---

## Hardcoded Secrets

Example

```text
Terraform Variables

↓

AWS Secret Key

↓

Critical Finding
```

---

## Privileged Kubernetes Containers

Example

```text
Container

↓

Privileged = true

↓

Security Risk
```

---

# Popular IaC Scanning Tools

| Tool | Purpose |
|------|---------|
| Checkov | Terraform, Kubernetes, CloudFormation scanning |
| Tfsec | Terraform security scanning |
| Terrascan | Multi-cloud IaC scanning |
| KICS | Infrastructure security analysis |
| Trivy | Kubernetes & IaC scanning |
| Prisma Cloud | Enterprise IaC security |

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

Pull Request

↓

IaC Scan

↓

Security Report

↓

Fix Findings

↓

Code Review

↓

Terraform Plan

↓

Terraform Apply

↓

Infrastructure Deployment
```

The deployment should be blocked if critical security issues are detected.

---

## Production Workflow

```text
Developer

↓

Write Infrastructure Code

↓

Commit Changes

↓

IaC Scan

↓

Security Approval

↓

Terraform Apply

↓

Cloud Resources Created
```

---

## Production Example

A Terraform configuration creates an EC2 Security Group.

```text
Terraform Code

↓

Security Group

↓

Ingress

↓

0.0.0.0/0

↓

Checkov Scan

↓

Critical Finding

↓

Developer Restricts CIDR

↓

Pipeline Passes

↓

Deploy
```

The insecure security group is fixed before deployment.

---

## Best Practices

- Scan every Infrastructure as Code change
- Integrate scanning into CI/CD
- Fail builds for critical findings
- Encrypt cloud resources by default
- Restrict public network access
- Remove hardcoded secrets
- Use approved Terraform modules
- Continuously update security policies
- Review scan reports regularly
- Combine IaC Scanning with runtime monitoring

---

## Common Mistakes

- Running scans only before production releases
- Ignoring medium and high findings
- Hardcoding cloud credentials
- Allowing unrestricted network access
- Disabling encryption
- Skipping compliance checks
- Ignoring Kubernetes manifests
- Using outdated scanning policies

---

# Common Troubleshooting

## Issue 1

### Pipeline Fails Due to Public Security Group

**Cause**

Terraform allows unrestricted inbound traffic.

**Resolution**

```text
Review Security Group

↓

Restrict CIDR

↓

Re-run Scan

↓

Deploy
```

---

## Issue 2

### Storage Bucket Reported as Public

**Cause**

Bucket ACL or policy allows public access.

**Resolution**

```text
Review Bucket Policy

↓

Disable Public Access

↓

Scan Again

↓

Deploy
```

---

## Issue 3

### Hardcoded Secrets Detected

**Cause**

Credentials were stored inside Terraform files.

**Resolution**

```text
Remove Secrets

↓

Store in Secret Manager

↓

Update Code

↓

Run Scan
```

---

## Issue 4

### Compliance Check Failure

**Cause**

Infrastructure violates organizational security policies.

**Resolution**

```text
Review Findings

↓

Update Configuration

↓

Validate Compliance

↓

Deploy
```

---

## Issue 5

### Kubernetes Manifest Security Failure

**Cause**

Container runs with privileged permissions.

**Resolution**

```text
Review Manifest

↓

Remove Privileged Mode

↓

Run Scan

↓

Deploy
```

---

# Production Interview Questions

## Question 1

### What is Infrastructure as Code (IaC) Scanning?

IaC Scanning is the process of analyzing infrastructure definition files for security vulnerabilities, misconfigurations, and compliance issues before infrastructure is deployed.

---

## Question 2

### Why is IaC Scanning important?

It detects security issues early, preventing insecure infrastructure from being provisioned in cloud environments.

---

## Question 3

### Which Infrastructure as Code technologies can be scanned?

Terraform, CloudFormation, ARM Templates, Kubernetes YAML manifests, Helm Charts, and other infrastructure definition files.

---

## Question 4

### What types of issues can IaC Scanning detect?

Public storage buckets, overly permissive security groups, missing encryption, hardcoded secrets, privileged containers, and compliance violations.

---

## Question 5

### What is the difference between IaC Scanning and Cloud Security Posture Management (CSPM)?

IaC Scanning analyzes infrastructure before deployment, whereas CSPM continuously monitors deployed cloud resources for security risks and misconfigurations.

---

## Question 6

### Which IaC Scanning tools have you used?

Common tools include Checkov, Tfsec, Terrascan, KICS, Trivy, and Prisma Cloud.

---

## Question 7

### At which stage should IaC Scanning be performed?

IaC Scanning should run automatically after infrastructure code is committed and before Terraform Apply or any infrastructure deployment.

---

## Question 8

### Should the CI/CD pipeline fail if critical IaC issues are found?

Yes. Critical findings should block infrastructure deployment until they are resolved to prevent insecure resources from being provisioned.

---

## Question 9

### Can IaC Scanning replace runtime cloud security monitoring?

No. IaC Scanning secures infrastructure before deployment, while runtime monitoring identifies configuration drift and security issues after deployment.

---

## Question 10

### How does IaC Scanning support DevSecOps?

IaC Scanning enforces secure infrastructure configurations, prevents cloud misconfigurations, automates compliance checks, and ensures infrastructure security is validated before deployment.

---

# Key Takeaways

- IaC Scanning secures infrastructure before deployment.
- It detects cloud misconfigurations, compliance issues, and insecure infrastructure definitions.
- Integrate IaC Scanning into every CI/CD pipeline.
- Fail deployments when critical security issues are detected.
- IaC Scanning is an essential DevSecOps practice for building secure cloud infrastructure.