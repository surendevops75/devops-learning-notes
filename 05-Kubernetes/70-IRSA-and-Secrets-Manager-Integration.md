# IRSA and AWS Secrets Manager Integration

## Introduction

Applications running inside Kubernetes often need access to:

```text
Database Passwords

API Keys

Tokens

Certificates

Third Party Credentials
```

---

Traditionally teams stored secrets as:

```text
Kubernetes Secrets
```

or

```text
Environment Variables
```

---

Modern EKS environments prefer:

```text
AWS Secrets Manager

+

IRSA
```

---

Reason

```text
Better Security

Centralized Management

IAM Based Access Control

Secret Rotation
```

---

# What Problem Are We Solving?

Application:

```text
Catalogue Service
```

---

Needs

```text
MySQL Password
```

---

Bad Approach

```text
Store Password

Inside Image
```

---

Problems

```text
Security Risk

Hard To Rotate

Image Rebuild Required
```

---

Another Bad Approach

```text
Store Plain Password

Inside Manifest
```

---

Problems

```text
Visible To Users

Difficult To Manage
```

---

Better Approach

```text
Store Secret

In AWS Secrets Manager
```

---

# What Is IRSA?

IRSA

```text
IAM Roles For Service Accounts
```

---

IRSA allows:

```text
Pods

To Assume

IAM Roles
```

---

Without Using

```text
Access Keys

Secret Keys
```

---

# Traditional Authentication

```text
Application

↓

AWS Access Key

↓

AWS Secret Key

↓

AWS Service
```

---

Problems

```text
Credential Rotation

Security Risk

Credential Leakage
```

---

# IRSA Authentication

```text
Pod

↓

Service Account

↓

IAM Role

↓

AWS Service
```

---

Benefits

```text
No Access Keys

No Secret Keys

Temporary Credentials

Better Security
```

---

# High Level Architecture

```text
Pod

↓

Service Account

↓

IAM Role

↓

Secrets Manager

↓

Secret Retrieved
```

---

# Components Required

```text
OIDC Provider

IAM Policy

IAM Role

Service Account

Secrets Manager Secret
```

---

# Step 1 - Configure OIDC Provider

## What Is OIDC?

OIDC

```text
OpenID Connect
```

---

Used By Kubernetes To:

```text
Authenticate

With External Identity Providers
```

---

EKS Uses OIDC For:

```text
IRSA
```

---

# Why OIDC Is Required?

Without OIDC

```text
Kubernetes

Cannot Assume

IAM Roles
```

---

Without OIDC

```text
IRSA Will Not Work
```

---

# Verify OIDC

Command

```bash
aws eks describe-cluster \
--name roboshop-dev \
--query cluster.identity.oidc.issuer
```

---

# Create OIDC Provider

Command

```bash
eksctl utils associate-iam-oidc-provider \
--cluster roboshop-dev \
--approve
```

---

Result

```text
OIDC Provider Created
```

---

# Step 2 - Store Secret In Secrets Manager

Example Secret

```text
roboshop/dev/mysql_password
```

---

Value

```json
{
  "mysql-password": "MySQL@123"
}
```

---

# Why Secrets Manager?

Benefits

```text
Encryption

Versioning

Rotation

Audit Logs

Central Management
```

---

# Step 3 - Create IAM Policy

The Pod Needs Permission To Read Secrets.

---

Example Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "*"
    }
  ]
}
```

---

# Principle Of Least Privilege

Avoid

```text
All Secrets Access
```

---

Prefer

```text
Specific Secret Access
```

---

Example

```text
roboshop/dev/mysql_password
```

Only.

---

# Step 4 - Create Service Account

Command

```bash
eksctl create iamserviceaccount \
--cluster roboshop-dev \
--name secret-mysql-reader \
--namespace roboshop \
--attach-policy-arn arn:aws:iam::123456789012:policy/MySQLSecretReader \
--approve
```

---

What Happens?

eksctl Creates

```text
IAM Role

IAM Role Trust Policy

Service Account

IAM Policy Attachment
```

Automatically.

---

# Result

Service Account

```text
secret-mysql-reader
```

---

IAM Role

```text
Created Automatically
```

---

# Verify Service Account

Command

```bash
kubectl get sa -n roboshop
```

---

Expected

```text
secret-mysql-reader
```

---

# Verify Annotation

Command

```bash
kubectl describe sa secret-mysql-reader
```

---

Expected

```text
eks.amazonaws.com/role-arn
```

---

# Step 5 - Use Service Account In Pod

Example

```yaml
spec:

  serviceAccountName: secret-mysql-reader
```

---

Meaning

```text
Pod Uses IRSA
```

---

# Production Flow

```text
Pod Starts

↓

Uses Service Account

↓

Assumes IAM Role

↓

Gets Temporary Credentials

↓

Reads Secret

↓

Starts Application
```

---

# Production Example

Application

```text
Catalogue
```

---

Needs

```text
MySQL Password
```

---

Stored In

```text
AWS Secrets Manager
```

---

Flow

```text
Catalogue Pod

↓

Service Account

↓

IAM Role

↓

Secrets Manager

↓

Password Retrieved
```

---

# Secret Retrieval Command

Inside Pod

```bash
aws secretsmanager get-secret-value \
--secret-id roboshop/dev/mysql_password \
--query SecretString \
--output text
```

---

Example Output

```json
{
  "mysql-password":"MySQL@123"
}
```

---

# Real Production Scenario

Application Startup

```text
Pod Starts

↓

Init Container Executes

↓

Read MySQL Password

↓

Store In Shared Volume

↓

Main Container Starts

↓

Connect To Database
```

---

# Why Combine Init Containers With IRSA?

Benefits

```text
No Hardcoded Secrets

No Kubernetes Secrets

No Access Keys

Secure Startup
```

---

# Production Architecture

```text
AWS Secrets Manager

↓

IAM Policy

↓

IAM Role

↓

Service Account

↓

Init Container

↓

Shared Volume

↓

Application Container
```

---

# Complete Production Example

## Secret

```text
roboshop/dev/mysql_password
```

---

## Policy

```text
Allow

GetSecretValue
```

---

## Service Account

```text
secret-mysql-reader
```

---

## Pod

```yaml
serviceAccountName: secret-mysql-reader
```

---

## Result

```text
Application Reads Password

Without Access Keys
```

---

# Benefits Of IRSA

```text
Improved Security

No Static Credentials

Least Privilege Access

Easy Auditing

Centralized Permissions
```

---

# Common Production Problems

## OIDC Not Configured

Symptoms

```text
Access Denied
```

---

Verify

```bash
aws eks describe-cluster
```

---

# Missing IAM Policy

Symptoms

```text
AccessDeniedException
```

---

Verify

```text
Policy Attached To Role
```

---

# Wrong Service Account

Symptoms

```text
Secret Retrieval Failure
```

---

Verify

```yaml
serviceAccountName
```

---

# Wrong Secret Name

Symptoms

```text
Resource Not Found
```

---

Verify

```bash
aws secretsmanager list-secrets
```

---

# Trust Relationship Issue

Symptoms

```text
AssumeRole Failure
```

---

Verify

```text
IAM Role Trust Policy
```

---

# Troubleshooting Commands

Verify OIDC

```bash
aws eks describe-cluster \
--name roboshop-dev
```

---

Verify Service Account

```bash
kubectl get sa -A
```

---

Verify Annotations

```bash
kubectl describe sa secret-mysql-reader
```

---

Verify Secret Access

```bash
aws secretsmanager get-secret-value \
--secret-id roboshop/dev/mysql_password
```

---

Verify Pod

```bash
kubectl describe pod <pod-name>
```

---

# Common Interview Questions

## What Is IRSA?

### Short Answer

IRSA stands for IAM Roles for Service Accounts and allows Kubernetes Pods to securely access AWS services using IAM roles.

### Detailed Explanation

IRSA eliminates the need for AWS access keys inside Pods. Instead, Pods assume IAM roles through Kubernetes service accounts and OIDC integration.

### Production Example

```text
Pod

↓

Service Account

↓

IAM Role

↓

Secrets Manager
```

### Follow-Up Questions

- Why is IRSA more secure?
- What role does OIDC play?
- How does the Pod get credentials?
- What AWS services can use IRSA?

---

## Why Is OIDC Required For IRSA?

### Short Answer

OIDC allows EKS to establish trust between Kubernetes service accounts and AWS IAM roles.

### Detailed Explanation

Without OIDC, IAM cannot verify Kubernetes identities. IRSA depends on OIDC trust relationships to issue temporary credentials.

### Production Example

```bash
eksctl utils associate-iam-oidc-provider \
--cluster roboshop-dev \
--approve
```

### Follow-Up Questions

- What happens if OIDC is missing?
- How do you verify OIDC?
- Can IRSA work without OIDC?
- What is OpenID Connect?

---

## How Does A Pod Access AWS Secrets Manager?

### Short Answer

Using a service account associated with an IAM role that has permission to read secrets.

### Detailed Explanation

The Pod assumes the IAM role through IRSA and retrieves the secret from AWS Secrets Manager without storing credentials locally.

### Production Example

```text
Pod

↓

IAM Role

↓

Secrets Manager

↓

Database Password
```

### Follow-Up Questions

- Which permission is required?
- How do you restrict access?
- How do you audit access?
- Can multiple Pods use the same role?

---

## Why Use AWS Secrets Manager Instead Of Kubernetes Secrets?

### Short Answer

AWS Secrets Manager provides stronger security, rotation, auditing, and centralized management.

### Detailed Explanation

Kubernetes Secrets are base64 encoded and managed inside the cluster. Secrets Manager provides encryption, versioning, IAM controls, and secret rotation capabilities.

### Production Example

```text
MySQL Password

Stored In

AWS Secrets Manager
```

### Follow-Up Questions

- Are Kubernetes Secrets encrypted?
- How does rotation work?
- Which option is more secure?
- How do you manage secret versions?

---

## What Is The Principle Of Least Privilege?

### Short Answer

Grant only the minimum permissions required.

### Detailed Explanation

Instead of allowing access to all secrets, IAM policies should allow access only to the specific secret required by the application.

### Production Example

Allow:

```text
roboshop/dev/mysql_password
```

Avoid:

```text
All Secrets
```

### Follow-Up Questions

- Why is least privilege important?
- How do you implement it?
- What risks exist with broad permissions?
- How do you audit permissions?

---

# Key Takeaways

```text
IRSA stands for IAM Roles for Service Accounts.

IRSA eliminates the need for AWS access keys inside Pods.

OIDC is required for IRSA to function.

Pods assume IAM roles using Service Accounts.

AWS Secrets Manager provides secure secret storage.

IAM Policies control which secrets can be accessed.

Least Privilege should always be followed.

Init Containers are commonly used to fetch secrets before application startup.

IRSA improves security, auditing, and credential management.

Modern EKS environments prefer IRSA over static AWS credentials.
```