# EKS OIDC and IRSA

## Introduction

In the previous chapter we learned:

```text
Service Accounts

Pod Identity

RBAC
```

---

A Service Account provides:

```text
Identity
```

for a Pod.

---

But many production applications need access to:

```text
AWS Secrets Manager

Parameter Store

S3

DynamoDB

SNS

SQS
```

---

Question:

```text
How Can A Pod

Access AWS Services

Securely?
```

---

Old Solution

```text
Store AWS Access Key

Store AWS Secret Key

Inside Pod
```

---

Problems

```text
Security Risk

Credential Leakage

Difficult Rotation

Poor Auditing
```

---

Modern Solution

```text
IRSA

IAM Roles For Service Accounts
```

---

# What is IRSA?

IRSA stands for:

```text
IAM Roles For Service Accounts
```

---

IRSA allows:

```text
Kubernetes Pods

To Assume IAM Roles
```

---

Without storing:

```text
AWS Access Keys

AWS Secret Keys
```

inside containers.

---

Easy Interview Answer

```text
IRSA allows Kubernetes Pods to securely access AWS services using IAM Roles mapped to Service Accounts.
```

---

# Why Do We Need IRSA?

Application

```text
Catalogue
```

needs:

```text
Database Password
```

from:

```text
AWS Secrets Manager
```

---

Without IRSA

```text
Store AWS Keys

Inside Pod
```

---

Bad Practice.

---

With IRSA

```text
Pod

↓

Service Account

↓

IAM Role

↓

Secrets Manager
```

---

Much More Secure.

---

# What is OIDC?

OIDC stands for:

```text
OpenID Connect
```

---

OIDC is an:

```text
Identity Layer

On Top Of OAuth 2.0
```

---

In EKS:

```text
OIDC

Connects Kubernetes

With AWS IAM
```

---

Easy Interview Answer

```text
OIDC allows AWS IAM to trust Kubernetes Service Accounts.
```

---

# Why Is OIDC Required?

Without OIDC

```text
AWS Cannot Trust

Kubernetes Service Accounts
```

---

IRSA cannot work.

---

Need Trust Relationship.

---

OIDC provides:

```text
Trust Between

EKS

And

IAM
```

---

# IRSA Architecture

```text
Pod
 │

 ▼

Service Account
 │

 ▼

OIDC Provider
 │

 ▼

IAM Role
 │

 ▼

Temporary Credentials
 │

 ▼

AWS Service
```

---

# Real Production Example

Application

```text
Catalogue Service
```

Needs

```text
MySQL Password
```

from:

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

Database Credentials
```

---

# Step 1

Verify OIDC Provider

Command

```bash
aws eks describe-cluster \
--name roboshop-dev \
--query "cluster.identity.oidc.issuer"
```

---

Example Output

```text
https://oidc.eks.us-east-1.amazonaws.com/id/ABC123
```

---

# Step 2

Associate OIDC Provider

Command

```bash
eksctl utils associate-iam-oidc-provider \
--cluster roboshop-dev \
--approve
```

---

Purpose

```text
Create Trust

Between EKS

And IAM
```

---

# Verify OIDC Provider

Command

```bash
aws iam list-open-id-connect-providers
```

---

Example Output

```text
OIDC Provider ARN
```

---

# What Happens Internally?

AWS creates:

```text
OIDC Provider
```

---

IAM can now trust:

```text
Kubernetes Service Accounts
```

---

# Step 3

Create IAM Policy

Example

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

Purpose

```text
Read Secret

From Secrets Manager
```

---

# Step 4

Create IAM Service Account

Command

```bash
eksctl create iamserviceaccount \
--cluster roboshop-dev \
--name secret-reader \
--namespace roboshop \
--attach-policy-arn arn:aws:iam::123456789:policy/RoboShopMySQLSecretReader \
--approve
```

---

This command creates:

```text
Service Account

IAM Role

Trust Policy

IAM Policy Attachment
```

---

# Verify Service Account

```bash
kubectl get sa \
-n roboshop
```

---

Output

```text
secret-reader
```

---

# Verify IAM Role

```bash
aws iam list-roles
```

---

Role created automatically.

---

# What Is Trust Policy?

Trust policy allows:

```text
Specific Service Account

To Assume IAM Role
```

---

Example

```text
secret-reader

↓

IAM Role
```

---

No other Pod can use it.

---

# IAM Trust Relationship

```text
IAM Role

Trusts

OIDC Provider

+

Service Account
```

---

This is the foundation of IRSA.

---

# Pod Configuration

Deployment

```yaml
spec:

  serviceAccountName: secret-reader

  containers:

  - name: catalogue

    image: catalogue:1.0
```

---

Important Line

```yaml
serviceAccountName: secret-reader
```

---

Meaning

```text
Use IRSA Enabled

Service Account
```

---

# Runtime Authentication Flow

Pod Starts

```text
Catalogue Pod
```

↓

Uses

```text
secret-reader
```

↓

Generates

```text
OIDC Token
```

↓

AWS IAM Validates

```text
OIDC Identity
```

↓

IAM Role Assumed

↓

Temporary Credentials Issued

↓

Access AWS Service

---

# Why Temporary Credentials?

AWS generates:

```text
Short Lived Credentials
```

---

Instead Of

```text
Permanent Access Keys
```

---

Security Benefit

```text
Credentials Expire Automatically
```

---

# Accessing AWS Secrets Manager

Application

```text
Catalogue
```

---

Uses AWS SDK

---

SDK automatically:

```text
Uses IAM Role

From IRSA
```

---

No credentials required.

---

# Accessing Parameter Store

Application

```text
Payment Service
```

---

Needs

```text
API Key
```

---

Stored In

```text
AWS Parameter Store
```

---

Flow

```text
Payment Pod

↓

Service Account

↓

IAM Role

↓

Parameter Store
```

---

# Accessing S3

Application

```text
Backup Job
```

---

Needs

```text
S3 Upload Access
```

---

Flow

```text
Backup Pod

↓

Service Account

↓

IAM Role

↓

S3 Bucket
```

---

# Production Benefits

## No Static Credentials

```text
More Secure
```

---

## Least Privilege

```text
One Service Account

One IAM Role
```

---

Only required permissions granted.

---

## Easy Rotation

```text
AWS Manages Credentials
```

---

No redeployment required.

---

## Better Auditing

CloudTrail records:

```text
Who Accessed

Which Resource

When
```

---

# Service Account vs IRSA

Service Account

```text
Kubernetes Identity
```

---

IRSA

```text
AWS IAM Integration
```

---

Service Account

```text
Works Inside Kubernetes
```

---

IRSA

```text
Works With AWS Services
```

---

# Common Problems

## Access Denied

Error

```text
AccessDeniedException
```

---

Cause

```text
IAM Policy Missing
```

---

Verify

```text
Required AWS Permissions
```

---

# OIDC Missing

Cause

```text
OIDC Provider Not Associated
```

---

Verify

```bash
aws iam list-open-id-connect-providers
```

---

# Wrong Service Account

Cause

```text
Deployment Using

Default Service Account
```

---

Verify

```bash
kubectl describe pod pod-name
```

---

Check

```text
Service Account
```

field.

---

# Trust Policy Failure

Cause

```text
IAM Role

Does Not Trust

Service Account
```

---

Result

```text
AssumeRole Failure
```

---

# Troubleshooting

Verify OIDC

```bash
aws iam list-open-id-connect-providers
```

---

Verify Service Account

```bash
kubectl get sa -n roboshop
```

---

Describe Service Account

```bash
kubectl describe sa secret-reader \
-n roboshop
```

---

Describe Pod

```bash
kubectl describe pod pod-name
```

---

Verify IAM Role

```bash
aws iam get-role \
--role-name role-name
```

---

Check AWS Permissions

```bash
aws iam list-attached-role-policies \
--role-name role-name
```

---

# Common Interview Questions

## What is IRSA?

### Short Answer

IRSA stands for IAM Roles for Service Accounts.

### Detailed Explanation

IRSA allows Kubernetes Pods to assume IAM Roles using Service Accounts without storing AWS credentials inside containers.

### Production Example

Catalogue Pods use IRSA to access Secrets Manager.

### Follow-Up Questions

- Why is IRSA needed?
- Which AWS services can be accessed?
- How does IRSA improve security?

---

## Why Is OIDC Required?

### Short Answer

OIDC allows AWS IAM to trust Kubernetes Service Accounts.

### Detailed Explanation

Without OIDC there is no trust relationship between Kubernetes and IAM.

### Production Example

The roboshop-dev cluster uses OIDC to allow Pods to assume IAM Roles.

### Follow-Up Questions

- What happens if OIDC is missing?
- How do you verify OIDC?
- Which command creates OIDC provider?

---

## What Does eksctl create iamserviceaccount Do?

### Short Answer

It creates a Service Account integrated with an IAM Role.

### Detailed Explanation

eksctl automatically creates the Service Account, IAM Role, trust relationship, and policy attachments.

### Production Example

The secret-reader Service Account receives access to AWS Secrets Manager.

### Follow-Up Questions

- Does it create IAM Roles?
- Does it attach policies?
- Can existing Service Accounts be used?

---

## Why Not Store AWS Access Keys In Pods?

### Short Answer

Static credentials are insecure.

### Detailed Explanation

Access Keys can be leaked, rotated poorly, and create security risks.

### Production Example

IRSA eliminates the need to store AWS credentials in application containers.

### Follow-Up Questions

- What replaces access keys?
- How are credentials generated?
- What is least privilege?

---

## How Have You Used IRSA In Production?

### Short Answer

We used IRSA to provide secure access from EKS Pods to AWS Secrets Manager.

### Detailed Explanation

Applications used Service Accounts mapped to IAM Roles. Secrets were retrieved dynamically during runtime without storing credentials in Kubernetes manifests.

### Production Example

Catalogue → Service Account → IAM Role → Secrets Manager → MySQL Credentials.

### Follow-Up Questions

- Why not Kubernetes Secrets?
- How is access controlled?
- What security benefits does IRSA provide?

---

# Key Takeaways

```text
OIDC enables trust between EKS and IAM.

IRSA stands for IAM Roles for Service Accounts.

IRSA eliminates the need for AWS Access Keys in Pods.

Service Accounts provide Kubernetes identity.

IAM Roles provide AWS permissions.

Temporary credentials improve security.

Secrets Manager, Parameter Store, and S3 are common IRSA use cases.

OIDC is mandatory for IRSA.

IRSA is the recommended way for EKS Pods to access AWS services.

IRSA is one of the most important EKS security features.
```