# Production Service Accounts and IRSA Using Secrets Manager

## Introduction

In previous chapters we learned:

```text
Service Accounts

OIDC

IRSA
```

---

Now let's understand a real production use case.

---

Most applications need:

```text
Database Credentials

API Tokens

Certificates

Application Secrets
```

---

Question:

```text
Where Should We Store Them?
```

---

Bad Options

```text
Inside Source Code

Inside Docker Images

Inside Kubernetes Manifests
```

---

Good Option

```text
AWS Secrets Manager
```

---

Question:

```text
How Can A Pod

Securely Access

AWS Secrets Manager?
```

---

Answer

```text
Service Account

+

IRSA

+

IAM Role
```

---

# Production Scenario

Application

```text
Catalogue Service
```

---

Needs

```text
MySQL Username

MySQL Password
```

---

Database

```text
Amazon RDS MySQL
```

---

Requirement

```text
Securely Retrieve Credentials

At Runtime
```

---

# Traditional Approach

Store Password

Inside Deployment YAML

---

Example

```yaml
env:

- name: DB_USERNAME

  value: catalogue

- name: DB_PASSWORD

  value: catalogue@123
```

---

Problems

```text
Visible In Git

Visible In CI/CD

Visible In Manifests

Security Risk
```

---

# Kubernetes Secret Approach

```yaml
kind: Secret
```

---

Better than plaintext.

---

But still

```text
Stored Inside Cluster

Visible To Cluster Admins

Manual Rotation
```

---

# Production Approach

Store Secret In

```text
AWS Secrets Manager
```

---

Secret Name

```text
roboshop/mysql
```

---

Secret Content

```json
{
  "username":"catalogue",
  "password":"catalogue@123"
}
```

---

# High Level Architecture

```text
Catalogue Pod
        │

        ▼

Service Account
(secret-reader)
        │

        ▼

IAM Role
        │

        ▼

AWS Secrets Manager
        │

        ▼

Database Credentials
        │

        ▼

MySQL Database
```

---

# Complete Authentication Flow

```text
Catalogue Pod
      │

      ▼

Service Account Token
      │

      ▼

OIDC Provider
      │

      ▼

IAM Role
      │

      ▼

Temporary AWS Credentials
      │

      ▼

Secrets Manager
      │

      ▼

Secret Value
```

---

# Step 1 - Create Secret

AWS Console

```text
Secrets Manager

↓

Store New Secret
```

---

Name

```text
roboshop/mysql
```

---

Value

```json
{
  "username":"catalogue",
  "password":"catalogue@123"
}
```

---

# Step 2 - Create IAM Policy

Policy

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

# Step 3 - Associate OIDC Provider

Command

```bash
eksctl utils associate-iam-oidc-provider \
--cluster roboshop-dev \
--approve
```

---

Purpose

```text
Trust Relationship

Between EKS

And IAM
```

---

Without OIDC

```text
IRSA Cannot Work
```

---

# Step 4 - Create IAM Service Account

Command

```bash
eksctl create iamserviceaccount \
--cluster roboshop-dev \
--name secret-reader \
--namespace roboshop \
--attach-policy-arn arn:aws:iam::123456789012:policy/RoboShopMySQLSecretReader \
--approve
```

---

What Gets Created?

```text
Service Account

IAM Role

IAM Trust Policy

Policy Attachment
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

# Step 5 - Configure Deployment

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: catalogue

spec:

  replicas: 2

  selector:

    matchLabels:

      app: catalogue

  template:

    metadata:

      labels:

        app: catalogue

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
Run Pod

Using IRSA Enabled

Service Account
```

---

# Runtime Flow

Application Starts

```text
Catalogue Pod
```

↓

Uses

```text
secret-reader
```

↓

Assumes

```text
IAM Role
```

↓

AWS Generates

```text
Temporary Credentials
```

↓

Application Calls

```text
Secrets Manager
```

↓

Reads

```text
username

password
```

↓

Connects To

```text
MySQL
```

---

# How Does AWS Know The Pod?

AWS doesn't trust Pods.

---

AWS trusts:

```text
IAM Roles
```

---

IAM Role trusts:

```text
OIDC Provider
```

---

OIDC validates:

```text
Service Account Token
```

---

Authentication succeeds.

---

# Why Temporary Credentials?

Old Method

```text
Access Key

Secret Key
```

---

Stored Inside

```text
Pod

Container

Git Repository
```

---

Security Risk.

---

IRSA Method

```text
Temporary Credentials

Generated Dynamically
```

---

Benefits

```text
Automatically Expire

Automatically Rotate
```

---

# Security Benefits

## No AWS Access Keys

```text
More Secure
```

---

## Least Privilege

Only

```text
Catalogue Pod
```

can access:

```text
roboshop/mysql
```

---

## Easier Rotation

Change Secret

Without Rebuilding Images

Without Updating Deployments
```

---

## Better Auditing

CloudTrail Shows

```text
Who Read Secret

When

Using Which Role
```

---

# Production Example 2

Payment Service

Needs

```text
Payment Gateway Token
```

---

Store In

```text
Secrets Manager
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

Secrets Manager
```

---

# Production Example 3

Inventory Service

Needs

```text
S3 Bucket Access
```

---

Flow

```text
Inventory Pod

↓

Service Account

↓

IAM Role

↓

S3
```

---

No access keys required.

---

# Production Example 4

Shipping Service

Needs

```text
Parameter Store Values
```

---

Flow

```text
Shipping Pod

↓

Service Account

↓

IAM Role

↓

SSM Parameter Store
```

---

# Why Not Use One IAM Role For All Pods?

Bad Practice

```text
One Role

Many Applications
```

---

Problems

```text
Too Much Access

Difficult Auditing

Security Risk
```

---

Good Practice

```text
One Service Account

One IAM Role

One Application
```

---

Example

```text
catalogue-sa

payment-sa

shipping-sa
```

---

Separate Roles.

---

# Common Problems

## AccessDeniedException

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
secretsmanager:GetSecretValue
```

permission.

---

# Secret Not Found

Cause

```text
Wrong Secret Name
```

---

Verify

```bash
aws secretsmanager list-secrets
```

---

# OIDC Missing

Cause

```text
OIDC Provider Not Configured
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
Default Service Account

Being Used
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

# IAM Trust Relationship Failure

Cause

```text
IAM Role

Not Trusting

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

Check Pod

```bash
kubectl describe pod pod-name
```

---

Check IAM Role

```bash
aws iam get-role \
--role-name role-name
```

---

Check Policy

```bash
aws iam list-attached-role-policies \
--role-name role-name
```

---

# Common Interview Questions

## How Have You Used Service Accounts In Production?

### Short Answer

We used Service Accounts with IRSA to securely access AWS Secrets Manager.

### Detailed Explanation

Applications were assigned dedicated Service Accounts mapped to IAM Roles. Database credentials were stored in Secrets Manager and retrieved dynamically at runtime.

### Production Example

Catalogue → Service Account → IAM Role → Secrets Manager → MySQL.

### Follow-Up Questions

- Why not Kubernetes Secrets?
- Why not Access Keys?
- How is auditing performed?

---

## Why Use IRSA Instead Of Access Keys?

### Short Answer

IRSA eliminates static credentials.

### Detailed Explanation

AWS generates temporary credentials dynamically using IAM Roles.

### Production Example

Catalogue Pods accessed Secrets Manager without storing AWS credentials.

### Follow-Up Questions

- What credentials are generated?
- How are they rotated?
- What security benefits exist?

---

## Why Is OIDC Required?

### Short Answer

OIDC enables trust between Kubernetes and IAM.

### Detailed Explanation

Without OIDC, IAM cannot verify Kubernetes Service Accounts.

### Production Example

roboshop-dev cluster uses OIDC to support IRSA.

### Follow-Up Questions

- How do you verify OIDC?
- What command enables OIDC?
- What happens if OIDC is missing?

---

## Why Not Store Passwords In Git?

### Short Answer

Credentials can be exposed.

### Detailed Explanation

Secrets stored in repositories create major security risks.

### Production Example

Database credentials are stored in Secrets Manager instead of Helm values files.

### Follow-Up Questions

- Where should secrets be stored?
- How are secrets rotated?
- How is access controlled?

---

# Key Takeaways

```text
Service Accounts provide Pod identity.

IRSA maps Service Accounts to IAM Roles.

OIDC is mandatory for IRSA.

Secrets Manager is a common production use case.

No AWS Access Keys are stored inside Pods.

Temporary credentials improve security.

One application should have one Service Account and one IAM Role.

CloudTrail provides auditing.

Catalogue → Service Account → IAM Role → Secrets Manager is a common EKS production pattern.

IRSA is one of the most important EKS security features.
```