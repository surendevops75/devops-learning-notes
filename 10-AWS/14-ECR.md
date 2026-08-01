# Amazon Elastic Container Registry (Amazon ECR)

---

# Introduction

Amazon Elastic Container Registry (Amazon ECR) is a fully managed container image registry provided by AWS. It enables developers and DevOps teams to securely store, manage, scan, version, and distribute Docker and OCI-compatible container images.

Modern applications are increasingly containerized using Docker and orchestrated by platforms such as Amazon EKS, Amazon ECS, and Kubernetes. These container images require a secure and scalable repository where they can be stored and accessed by deployment platforms.

Amazon ECR eliminates the operational overhead of managing a private Docker registry while providing enterprise-grade security, scalability, and integration with the AWS ecosystem.

---

# What is Amazon ECR?

Amazon ECR is a private container image registry.

Instead of storing source code, ECR stores container images.

Example

```text
Docker Build

↓

Docker Image

↓

Amazon ECR

↓

Amazon EKS

↓

Running Pods
```

ECR supports:

- Docker Images
- OCI Images
- OCI Artifacts
- Multi-Architecture Images

---

# Why Amazon ECR?

Imagine managing Docker images on your own server.

Challenges include:

- Installing Docker Registry
- Storage Management
- Authentication
- Authorization
- High Availability
- Image Replication
- Security Scanning
- Backup
- Maintenance

Amazon ECR manages all of these automatically.

---

# Real-World Problem

A company develops 150 microservices.

Every day:

- Hundreds of Docker images are built
- CI/CD pipelines push new versions
- Kubernetes clusters pull images
- Security teams scan images
- Old images must be cleaned automatically

Amazon ECR centralizes image management securely and efficiently.

---

# Enterprise Architecture

```text
Developer

↓

GitHub

↓

Jenkins / GitHub Actions

↓

Docker Build

↓

Amazon ECR

↓

Image Scan

↓

Amazon EKS / ECS

↓

Running Containers
```

---

# Internal Working

```text
Application Code

↓

Docker Build

↓

Docker Image

↓

Image Tag

↓

Amazon ECR Repository

↓

Kubernetes Pulls Image

↓

Container Starts
```

---

# Core Components

Amazon ECR consists of:

- Registry
- Repository
- Image
- Image Tag
- Image Digest
- Repository Policy
- Lifecycle Policy
- Image Scanning
- Image Replication
- Authentication

---

# Registry

Every AWS account automatically gets a private container registry.

Example

```
123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

This registry contains multiple repositories.

---

# Repository

Repositories organize container images.

Example

```text
Registry

├── frontend

├── backend

├── payment

├── user

└── notification
```

Each repository stores different versions of an application.

---

# Docker Images

A Docker image contains everything required to run an application.

Example

```text
Application

+

Runtime

+

Libraries

+

Dependencies

+

Configuration

↓

Docker Image
```

---

# Image Tags

Tags identify image versions.

Example

```text
frontend:latest

frontend:v1.0

frontend:v1.1

frontend:v2.0

frontend:production
```

Best Practice

Never deploy using `latest` in production.

Use immutable version tags.

---

# Image Digest

Every image also receives a unique SHA256 digest.

Example

```text
frontend@sha256:a78f9b3d...
```

Benefits

- Immutable
- Unique
- Prevents accidental image replacement

Production deployments often reference image digests instead of tags.

---

# Authentication

Amazon ECR uses IAM authentication.

Workflow

```text
IAM User

↓

AWS CLI Login

↓

Authentication Token

↓

Docker Login

↓

Push / Pull Images
```

Authentication tokens expire after 12 hours.

---

# Private Repository

Private repositories are accessible only to authorized IAM users and roles.

Suitable for:

- Enterprise Applications
- Internal Microservices
- Financial Systems
- Healthcare Applications

---

# Public Repository

Amazon ECR also supports public repositories.

Suitable for:

- Open Source Projects
- Community Images
- Public SDKs
- Shared Base Images

---

# Repository Policies

Repository Policies define who can:

- Push Images
- Pull Images
- Delete Images
- Describe Images

Example

```text
Developer Role

↓

Push Images

------------------

EKS Role

↓

Pull Images
```

---

# Image Scanning

Amazon ECR can automatically scan container images for vulnerabilities.

Scans identify:

- Critical CVEs
- High Severity CVEs
- Medium Severity CVEs
- Low Severity CVEs

Architecture

```text
Docker Push

↓

Amazon ECR

↓

Security Scan

↓

Report
```

---

# Basic Scanning

Uses AWS native vulnerability scanning.

Provides:

- CVE List
- Severity
- Package Information

---

# Enhanced Scanning

Enhanced Scanning integrates with Amazon Inspector.

Benefits

- Continuous Scanning
- Better CVE Detection
- Runtime Awareness
- Improved Security Reporting

Recommended for production environments.

---

# Image Replication

Images can automatically replicate to other AWS Regions.

Architecture

```text
Mumbai Repository

↓

Automatic Replication

↓

Singapore Repository

↓

Virginia Repository
```

Benefits

- Disaster Recovery
- Lower Latency
- Regional Availability

---

# Cross-Account Replication

Images can also replicate across AWS accounts.

Example

```text
Development Account

↓

Replication

↓

Production Account
```

Useful for enterprise CI/CD pipelines.

---

# Lifecycle Policies

Lifecycle Policies automatically remove old images.

Example Rule

```text
Keep

Last 20 Images

↓

Delete Older Images
```

Benefits

- Reduced Storage Cost
- Cleaner Repository
- Automatic Cleanup

---

# Pull Through Cache

Amazon ECR supports pull-through caching.

Instead of downloading repeatedly from Docker Hub:

```text
Docker Hub

↓

Amazon ECR Cache

↓

Kubernetes
```

Benefits

- Faster Pulls
- Reduced Rate Limits
- Improved Availability

---

# Image Encryption

Images stored in Amazon ECR are encrypted at rest.

Supports:

- AWS Managed Keys
- Customer Managed KMS Keys

Recommended

Use customer-managed KMS keys for production environments requiring strict compliance.

---

# Image Versioning

Typical production repositories contain multiple versions.

Example

```text
v1.0

v1.1

v1.2

v2.0

v2.1
```

Deployment systems choose the required version.

---

# AWS Console Walkthrough

1. Open Amazon ECR
2. Create Repository
3. Configure Scanning
4. Configure Encryption
5. Configure Lifecycle Policy
6. Push Docker Image
7. Verify Image
8. View Scan Results
9. Copy Repository URI
10. Deploy Image

---

# AWS CLI

Create Repository

```bash
aws ecr create-repository \
--repository-name frontend
```

Login

```bash
aws ecr get-login-password \
| docker login \
--username AWS \
--password-stdin \
123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

List Images

```bash
aws ecr list-images \
--repository-name frontend
```

Delete Image

```bash
aws ecr batch-delete-image \
--repository-name frontend \
--image-ids imageTag=v1.0
```

---

# Terraform

```hcl
resource "aws_ecr_repository" "frontend" {

  name = "frontend"

  image_scanning_configuration {

    scan_on_push = true

  }

  image_tag_mutability = "IMMUTABLE"

}
```

Lifecycle Policy

```hcl
resource "aws_ecr_lifecycle_policy" "cleanup" {

  repository = aws_ecr_repository.frontend.name

  policy = jsonencode({

    rules = []

  })

}
```

---

# CloudFormation

```yaml
Resources:

  Repository:

    Type: AWS::ECR::Repository

    Properties:

      RepositoryName: frontend

      ImageScanningConfiguration:

        ScanOnPush: true
```

---

# Python (Boto3)

```python
import boto3

ecr = boto3.client("ecr")

response = ecr.describe_repositories()

print(response)
```

---

# Best Practices

- Enable image scanning
- Use immutable tags
- Avoid using `latest` in production
- Configure lifecycle policies
- Enable encryption with KMS
- Grant least-privilege IAM permissions
- Replicate images across Regions for DR
- Regularly remove unused images
- Integrate ECR with CI/CD pipelines
- Scan images before deployment

---

# Common Mistakes

- Using mutable tags
- Skipping vulnerability scanning
- Keeping unused images forever
- Hardcoding registry credentials
- Using public repositories for sensitive applications
- Granting overly broad IAM permissions
- Ignoring critical CVEs

---

# Troubleshooting

## Docker Push Fails

Check:

- IAM Permissions
- Docker Login
- Repository Exists
- Authentication Token

---

## Image Pull Fails in EKS

Verify:

- Node IAM Role
- Repository Policy
- Image URI
- Repository Region

---

## Image Scan Not Running

Check:

- Scan on Push
- Enhanced Scanning Configuration
- Inspector Integration

---

## Repository Not Accessible

Verify:

- Repository Policy
- IAM Policy
- Cross-Account Permissions

---

# Interview Questions

1. What is Amazon ECR?
2. Difference between ECR and Docker Hub?
3. What is a repository?
4. What is an image tag?
5. What is an image digest?
6. Difference between mutable and immutable tags?
7. Explain image scanning.
8. What is Enhanced Scanning?
9. Explain lifecycle policies.
10. What is pull-through cache?
11. How does EKS authenticate with ECR?
12. Explain cross-region replication.
13. Explain cross-account replication.
14. How would you secure an ECR repository?
15. Why should production deployments avoid the `latest` tag?

---

# Scenario-Based Questions

### Scenario 1

Your EKS cluster cannot pull images from ECR.

How would you troubleshoot?

---

### Scenario 2

Security discovers critical vulnerabilities in production images.

How would you prevent deployment of vulnerable images?

---

### Scenario 3

Storage costs increase because thousands of old images remain.

How would you optimize the repository?

---

### Scenario 4

Your company deploys applications across three AWS Regions.

How would you design ECR for high availability?

---

### Scenario 5

A deployment unexpectedly uses an older image even though a new version was pushed.

How would immutable tags and image digests help prevent this?

---

# Cheat Sheet

| Feature | Amazon ECR |
|---------|------------|
| Service Type | Container Registry |
| Stores | Docker & OCI Images |
| Authentication | IAM |
| Encryption | AWS KMS |
| Vulnerability Scanning | Yes |
| Lifecycle Policies | Yes |
| Cross-Region Replication | Yes |
| Cross-Account Replication | Yes |
| Pull Through Cache | Yes |
| Integration | EKS, ECS, Lambda |

---

# Summary

Amazon Elastic Container Registry (Amazon ECR) is a secure, scalable, and fully managed container image registry for Docker and OCI artifacts. It integrates seamlessly with AWS services such as Amazon EKS, Amazon ECS, and CI/CD pipelines, enabling organizations to store, scan, version, replicate, and deploy container images efficiently.

For production environments, enable enhanced vulnerability scanning, use immutable image tags, enforce least-privilege IAM policies, configure lifecycle policies for automatic cleanup, encrypt repositories with AWS KMS, and replicate critical repositories across Regions for disaster recovery.