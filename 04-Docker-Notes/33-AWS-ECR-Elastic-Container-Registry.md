# AWS ECR (Elastic Container Registry)

## Introduction

Building Docker images is only the first step.

To deploy containers in:

```text
EC2

ECS

EKS

Kubernetes
```

images must be stored in a centralized repository.

---

AWS provides:

```text
Amazon ECR
```

---

ECR stands for:

```text
Elastic Container Registry
```

---

# What is Amazon ECR?

Amazon ECR is:

```text
AWS Managed Docker Registry
```

used to:

```text
Store Docker Images

Version Images

Manage Image Access

Integrate With AWS Services
```

---

# Why ECR?

Suppose:

```text
Developer Machine
```

builds image:

```bash
docker build -t catalogue:v1 .
```

---

Production EKS Cluster:

```text
Cannot Access Local Images
```

---

Need:

```text
Central Image Repository
```

---

Solution:

```text
Amazon ECR
```

---

# ECR Workflow

```text
Developer / Jenkins
          ↓

Build Docker Image
          ↓

Push To ECR
          ↓

ECR Repository
          ↓

EKS / ECS
          ↓

Pull Image
          ↓

Run Container
```

---

# ECR Benefits

```text
Fully Managed

Highly Available

Secure

AWS IAM Integration

Image Scanning

Lifecycle Policies

EKS Integration
```

---

# Public vs Private ECR

## Private ECR

Used for:

```text
Production Applications

Internal Services

Enterprise Workloads
```

---

Access Controlled By:

```text
IAM Policies
```

---

## Public ECR

Used for:

```text
Publicly Available Images
```

---

Anyone can pull images.

---

# Creating an ECR Repository

AWS Console

```text
ECR
     ↓

Repositories
     ↓

Create Repository
```

---

Example

```text
catalogue
```

---

Repository URI

```text
123456789012.dkr.ecr.us-east-1.amazonaws.com/catalogue
```

---

# ECR Repository Structure

```text
AWS Account
        ↓

Region
        ↓

Repository
        ↓

Image
        ↓

Tag
```

---

Example

```text
123456789012.dkr.ecr.us-east-1.amazonaws.com/catalogue:v1
```

---

# Authenticate Docker To ECR

Docker must authenticate before pushing images.

---

Command

```bash
aws ecr get-login-password \
--region us-east-1 \
| docker login \
--username AWS \
--password-stdin \
123456789012.dkr.ecr.us-east-1.amazonaws.com
```

---

Result

```text
Login Succeeded
```

---

# Build Image

```bash
docker build -t catalogue:v1 .
```

---

Verify

```bash
docker images
```

---

# Tag Image For ECR

Format

```text
ACCOUNT-ID.dkr.ecr.REGION.amazonaws.com/REPO:TAG
```

---

Example

```bash
docker tag \
catalogue:v1 \
123456789012.dkr.ecr.us-east-1.amazonaws.com/catalogue:v1
```

---

# Push Image To ECR

Command

```bash
docker push \
123456789012.dkr.ecr.us-east-1.amazonaws.com/catalogue:v1
```

---

Flow

```text
Local Image
      ↓

ECR Repository
```

---

# Verify Image

AWS Console

```text
ECR
      ↓

Repository
      ↓

Images
```

---

Example

```text
catalogue:v1
```

visible.

---

# Pull Image From ECR

On another server:

Authenticate

```bash
aws ecr get-login-password \
--region us-east-1 \
| docker login \
--username AWS \
--password-stdin \
123456789012.dkr.ecr.us-east-1.amazonaws.com
```

---

Pull

```bash
docker pull \
123456789012.dkr.ecr.us-east-1.amazonaws.com/catalogue:v1
```

---

Run

```bash
docker run -d \
123456789012.dkr.ecr.us-east-1.amazonaws.com/catalogue:v1
```

---

# ECR and EKS

One of the most common production architectures.

---

Architecture

```text
Developer
      ↓

GitHub
      ↓

Jenkins
      ↓

Build Image
      ↓

Push To ECR
      ↓

EKS
      ↓

Pull Image
      ↓

Pod Running
```

---

# IAM Permissions

Users need:

```text
ecr:GetAuthorizationToken

ecr:BatchCheckLayerAvailability

ecr:GetDownloadUrlForLayer

ecr:BatchGetImage

ecr:PutImage
```

---

Without permissions:

```text
Image Push Fails

Image Pull Fails
```

---

# Image Tags in ECR

Examples

```text
catalogue:v1

catalogue:v2

catalogue:latest
```

---

Production Recommendation

```text
Use Version Tags

Avoid latest
```

---

Examples

```text
catalogue:1.0.0

catalogue:1.1.0

catalogue:2.0.0
```

---

# Image Scanning

ECR provides:

```text
Vulnerability Scanning
```

---

Checks:

```text
Operating System Packages

Known CVEs

Security Risks
```

---

Benefits

```text
Improved Security

Compliance

Risk Reduction
```

---

# Lifecycle Policies

Repositories can accumulate:

```text
Hundreds Of Images
```

---

Lifecycle Policies help:

```text
Delete Old Images

Reduce Storage Cost

Improve Management
```

---

Example

```text
Keep Last 20 Images
```

---

Delete older versions automatically.

---

# Cross Account ECR Access

Common Production Scenario

---

Example

```text
Dev Account
      ↓

Build Images
      ↓

Push To Shared ECR
      ↓

Prod Account
      ↓

Pull Images
```

---

Requires:

```text
Repository Policies

IAM Permissions
```

---

# ECR Commands

List Repositories

```bash
aws ecr describe-repositories
```

---

List Images

```bash
aws ecr list-images \
--repository-name catalogue
```

---

Describe Images

```bash
aws ecr describe-images \
--repository-name catalogue
```

---

Delete Image

```bash
aws ecr batch-delete-image \
--repository-name catalogue \
--image-ids imageTag=v1
```

---

# Common ECR Errors

## Access Denied

Example

```text
no basic auth credentials
```

---

Cause

```text
Docker Not Logged In
```

---

Fix

```bash
aws ecr get-login-password
```

---

Authenticate again.

---

## Repository Not Found

Example

```text
repository does not exist
```

---

Cause

```text
Wrong Repository Name
```

---

Verify

```bash
aws ecr describe-repositories
```

---

## Image Pull Failure

Cause

```text
Missing IAM Permissions
```

---

Verify:

```text
EKS Node Role

EC2 Role

User Permissions
```

---

# Production Example

CI/CD Pipeline

```text
GitHub
      ↓

Jenkins Pipeline
      ↓

Build Docker Image
      ↓

Security Scan
      ↓

Push To ECR
      ↓

Update Kubernetes Manifest
      ↓

Deploy To EKS
```

---

# Common Interview Questions

## What is Amazon ECR?

### Short Answer

Amazon ECR is AWS's managed container image registry.

### Detailed Explanation

ECR stores Docker images and integrates with ECS, EKS, IAM, and CI/CD pipelines.

### Production Example

Storing application images for EKS deployments.

### Follow-Up Questions

- How does ECR differ from Docker Hub?
- What services integrate with ECR?

---

## How Do You Push Images To ECR?

### Short Answer

Authenticate, tag the image, and push it.

### Detailed Explanation

Docker must authenticate to ECR using AWS credentials before images can be pushed.

### Production Example

Pushing catalogue:v1 to a private ECR repository.

### Follow-Up Questions

- Why is docker login required?
- How is authentication performed?

---

## Why Is ECR Preferred In AWS?

### Short Answer

Because it integrates directly with AWS services and IAM.

### Detailed Explanation

ECR provides secure image storage, vulnerability scanning, lifecycle management, and seamless EKS integration.

### Production Example

EKS pulling images from a private ECR repository.

### Follow-Up Questions

- What are lifecycle policies?
- What is image scanning?

---

## What IAM Permissions Are Needed For ECR?

### Short Answer

Permissions for authentication, pushing, and pulling images.

### Detailed Explanation

ECR operations rely on IAM policies controlling repository access.

### Production Example

Jenkins pushing images and EKS nodes pulling images.

### Follow-Up Questions

- What happens if permissions are missing?
- How does EKS authenticate to ECR?

---

# Key Takeaways

```text
Amazon ECR is AWS's managed Docker registry.

ECR stores container images securely.

Docker must authenticate before pushing images.

Images are tagged using repository URIs.

EKS commonly pulls images from ECR.

IAM controls access to repositories.

Image scanning helps identify vulnerabilities.

Lifecycle policies automatically remove old images.

ECR integrates tightly with CI/CD pipelines.

ECR is the most common registry used in AWS-based Kubernetes environments.
```