# EKS IAM Integration and aws-auth ConfigMap

## Introduction

In previous chapters we learned:

```text
Authentication

Authorization

RBAC

Role

RoleBinding

ClusterRole

ClusterRoleBinding
```

---

Now the important question is:

```text
How Does EKS Know

Who The User Is?
```

---

The answer is:

```text
AWS IAM
```

---

Amazon EKS uses:

```text
IAM

For Authentication
```

and

```text
RBAC

For Authorization
```

---

Easy Interview Answer

```text
Authentication → IAM

Authorization → RBAC
```

---

# Real Production Scenario

New Employee

```text
Suresh
```

joins the team.

---

Requirement

```text
Access EKS Cluster
```

---

Questions

```text
Who Is Suresh?

Can He Access Cluster?

What Actions Can He Perform?
```

---

EKS solves this using:

```text
IAM

+

aws-auth ConfigMap

+

RBAC
```

---

# Authentication vs Authorization

Authentication

```text
Who Are You?
```

---

Handled By

```text
AWS IAM
```

---

Authorization

```text
What Can You Do?
```

---

Handled By

```text
Kubernetes RBAC
```

---

# EKS Authentication Flow

Developer

```text
Suresh
```

---

Runs

```bash
kubectl get pods
```

---

Flow

```text
kubectl
    │

    ▼

kubeconfig
    │

    ▼

EKS API Server
    │

    ▼

AWS IAM
(Authentication)
    │

    ▼

aws-auth ConfigMap
    │

    ▼

RBAC
(Authorization)
    │

    ▼

Response
```

---

# Step 1 - Create IAM User

Create IAM User

```text
suresh
```

---

AWS Console

```text
IAM

↓

Users

↓

Create User
```

---

Purpose

```text
Identity Creation
```

---

# Step 2 - Grant EKS Describe Access

Without this permission:

```text
User Cannot Connect
```

to cluster.

---

Example Policy

```json
{
  "Version": "2012-10-17",

  "Statement": [

    {
      "Effect": "Allow",

      "Action": [

        "eks:DescribeCluster"

      ],

      "Resource": "*"
    }
  ]
}
```

---

Purpose

```text
Cluster Discovery
```

---

# Why DescribeCluster Permission?

When user runs:

```bash
kubectl get pods
```

---

kubectl needs:

```text
Cluster Endpoint

Certificate

Cluster Information
```

---

EKS provides this through:

```text
DescribeCluster API
```

---

# Verify IAM Identity

Command

```bash
aws sts get-caller-identity
```

---

Example Output

```json
{
  "UserId": "AIDAEXAMPLE",

  "Account": "123456789",

  "Arn": "arn:aws:iam::123456789:user/suresh"
}
```

---

# Step 3 - Update kubeconfig

Command

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name roboshop-dev
```

---

Purpose

```text
Configure kubectl

To Access EKS
```

---

# What Happens Internally?

AWS CLI calls:

```text
DescribeCluster
```

---

Downloads

```text
Cluster Endpoint

Certificate Authority

Authentication Data
```

---

Stores inside:

```text
~/.kube/config
```

---

# Step 4 - aws-auth ConfigMap

This is the most important EKS component.

---

aws-auth acts as:

```text
Bridge Between

IAM

And

Kubernetes
```

---

Without aws-auth

```text
IAM User Exists

But Kubernetes Doesn't Know User
```

---

# What is aws-auth?

A Kubernetes ConfigMap.

---

Located In

```text
kube-system Namespace
```

---

View

```bash
kubectl get configmap aws-auth \
-n kube-system
```

---

Describe

```bash
kubectl describe configmap aws-auth \
-n kube-system
```

---

# Why Do We Need aws-auth?

IAM understands:

```text
Users

Roles
```

---

Kubernetes understands:

```text
Users

Groups
```

---

aws-auth maps:

```text
IAM Identity

↓

Kubernetes Identity
```

---

# Example aws-auth Mapping

```yaml
mapUsers:

- userarn: arn:aws:iam::123456789:user/suresh

  username: suresh

  groups:

  - developers
```

---

Meaning

```text
IAM User

↓

Kubernetes User
```

---

# Authentication Result

IAM verifies:

```text
User Exists
```

---

aws-auth maps:

```text
IAM User

↓

Kubernetes User
```

---

Authentication Success.

---

# Step 5 - Create RBAC Rules

Now Kubernetes knows:

```text
Suresh
```

---

Need permissions.

---

Create Role

```yaml
kind: Role
```

---

Create RoleBinding

```yaml
kind: RoleBinding
```

---

Grant access.

---

# Complete Production Flow

Requirement

```text
Give Suresh Access

To roboshop-dev Namespace
```

---

Steps

```text
1. Create IAM User

2. Add EKS Describe Access

3. Update kubeconfig

4. Add User To aws-auth

5. Create Role

6. Create RoleBinding
```

---

Result

```text
Suresh Can Access Cluster
```

---

# EKS Cluster Creator

Interesting Interview Question.

---

Who Gets Access Initially?

---

Answer

```text
Cluster Creator
```

---

When EKS cluster is created:

```text
Creator Automatically Gets

cluster-admin
```

permissions.

---

Reason

```text
Someone Must Administer Cluster
```

---

# IAM Role Example

Instead of IAM User

Many organizations use:

```text
IAM Roles
```

---

Advantages

```text
More Secure

Temporary Credentials

Better Auditing
```

---

Example Mapping

```yaml
mapRoles:

- rolearn: arn:aws:iam::123456789:role/devops-role

  username: devops

  groups:

  - system:masters
```

---

Meaning

```text
IAM Role

↓

Kubernetes Admin
```

---

# system:masters Group

Special Kubernetes Group.

---

Members receive:

```text
Cluster Admin Access
```

---

Equivalent To

```text
cluster-admin
```

---

Use Carefully.

---

# Production Access Pattern

Trainee

```text
IAM User

↓

Read Only Role
```

---

Developer

```text
IAM User

↓

Developer Role
```

---

Senior Engineer

```text
IAM Role

↓

Namespace Admin
```

---

Platform Team

```text
IAM Role

↓

Cluster Admin
```

---

# Common Problems

## Unauthorized

Error

```text
Unauthorized
```

---

Cause

```text
IAM Authentication Failure
```

---

Check

```bash
aws sts get-caller-identity
```

---

# Forbidden

Error

```text
Forbidden
```

---

Cause

```text
RBAC Permission Missing
```

---

Authentication succeeded.

---

Authorization failed.

---

# User Exists But Cannot Access

Cause

```text
Missing aws-auth Mapping
```

---

Verify

```bash
kubectl get configmap aws-auth \
-n kube-system -o yaml
```

---

# kubeconfig Issues

Cause

```text
Wrong Cluster

Wrong Region
```

---

Fix

```bash
aws eks update-kubeconfig
```

---

# Troubleshooting Commands

Verify Identity

```bash
aws sts get-caller-identity
```

---

Update kubeconfig

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name roboshop-dev
```

---

Check Current Context

```bash
kubectl config current-context
```

---

View aws-auth

```bash
kubectl get configmap aws-auth \
-n kube-system -o yaml
```

---

Check Access

```bash
kubectl auth can-i get pods
```

---

# Common Interview Questions

## How Does Authentication Work In EKS?

### Short Answer

EKS uses AWS IAM for authentication.

### Detailed Explanation

When kubectl sends a request, EKS validates the user's IAM identity before allowing access.

### Production Example

Suresh authenticates using IAM credentials before accessing the cluster.

### Follow-Up Questions

- What service performs authentication?
- What permissions are required?
- What happens if credentials are invalid?

---

## What is aws-auth ConfigMap?

### Short Answer

aws-auth maps IAM users and roles to Kubernetes users and groups.

### Detailed Explanation

It acts as the bridge between AWS IAM and Kubernetes RBAC.

### Production Example

An IAM user named suresh is mapped to the developers group.

### Follow-Up Questions

- Where is aws-auth stored?
- Why is it required?
- What happens if mapping is missing?

---

## Why is DescribeCluster Permission Required?

### Short Answer

kubectl needs cluster information before connecting.

### Detailed Explanation

The DescribeCluster API provides endpoint and certificate information required by kubeconfig.

### Production Example

A developer cannot access EKS without eks:DescribeCluster permission.

### Follow-Up Questions

- Which API provides cluster endpoint?
- Does kubectl call DescribeCluster?
- What happens if permission is missing?

---

## Difference Between Unauthorized and Forbidden?

### Short Answer

Unauthorized means authentication failed, while Forbidden means authorization failed.

### Detailed Explanation

Unauthorized indicates IAM or identity issues. Forbidden indicates RBAC permission issues.

### Production Example

Missing IAM permissions cause Unauthorized. Missing RoleBinding causes Forbidden.

### Follow-Up Questions

- Which one relates to IAM?
- Which one relates to RBAC?
- How do you troubleshoot each?

---

## What Access Does the EKS Creator Get?

### Short Answer

The EKS creator automatically receives cluster-admin access.

### Detailed Explanation

The identity that creates the cluster becomes the initial administrator.

### Production Example

The platform engineer who created the cluster can manage all resources immediately after creation.

### Follow-Up Questions

- Can this access be removed?
- Is it granted through RBAC?
- Why is it required?

---

# Key Takeaways

```text
EKS uses IAM for authentication.

RBAC handles authorization.

aws-auth ConfigMap connects IAM and Kubernetes.

DescribeCluster permission is required for cluster access.

kubectl uses kubeconfig to connect to EKS.

Unauthorized indicates authentication failure.

Forbidden indicates authorization failure.

The EKS creator receives cluster-admin access.

IAM Roles are preferred over IAM Users in production.

Understanding aws-auth is essential for EKS administration.
```