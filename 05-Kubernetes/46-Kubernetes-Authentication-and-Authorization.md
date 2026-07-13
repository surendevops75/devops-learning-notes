# Kubernetes Authentication and Authorization

## Introduction

Security is one of the most important aspects of Kubernetes.

---

Whenever someone executes:

```bash
kubectl get pods
```

Kubernetes must answer two questions:

```text
Who Are You?

What Are You Allowed To Do?
```

---

These two questions are handled by:

```text
Authentication

Authorization
```

---

Authentication checks:

```text
Identity
```

---

Authorization checks:

```text
Permissions
```

---

# Real World Example

Suppose:

```text
Suresh
```

executes:

```bash
kubectl get pods
```

---

Kubernetes asks:

```text
Who Is Suresh?
```

---

If identity is valid:

```text
Authentication Success
```

---

Then Kubernetes asks:

```text
Can Suresh Read Pods?
```

---

If permission exists:

```text
Authorization Success
```

---

Request succeeds.

---

# Authentication vs Authorization

## Authentication

```text
Who Are You?
```

---

Examples

```text
IAM User

IAM Role

Certificate

Service Account
```

---

## Authorization

```text
What Can You Do?
```

---

Examples

```text
Read Pods

Create Deployments

Delete Services
```

---

Easy Interview Answer

```text
Authentication verifies identity.

Authorization verifies permissions.
```

---

# Request Flow

When a user runs:

```bash
kubectl get pods
```

---

Flow

```text
User
  │

  ▼

kubectl
  │

  ▼

kubeconfig
  │

  ▼

EKS API Server
  │

  ▼

Authentication
  │

  ▼

Authorization
  │

  ▼

Response
```

---

# What is Authentication?

Authentication verifies:

```text
User Identity
```

---

Examples

```text
Are You Suresh?

Are You Ramesh?

Are You Team Lead?
```

---

Without Authentication

```text
Anyone Can Access Cluster
```

---

Security Risk.

---

# Authentication Methods

Kubernetes supports:

```text
Certificates

OIDC

Service Accounts

Cloud Provider IAM
```

---

For EKS

```text
AWS IAM
```

is most common.

---

# Authentication in Amazon EKS

EKS integrates with:

```text
AWS IAM
```

---

Examples

```text
IAM User

IAM Role
```

---

When a user accesses cluster:

```text
EKS Calls IAM
```

to verify identity.

---

# Example Authentication Flow

User

```text
Suresh
```

---

Runs

```bash
kubectl get pods
```

---

kubectl sends request.

---

EKS checks:

```text
Is Suresh Valid?
```

---

IAM verifies:

```text
User Exists

Credentials Valid
```

---

Result

```text
Authentication Success
```

---

# What is kubeconfig?

kubectl uses:

```text
kubeconfig
```

to connect to cluster.

---

Default Location

```text
~/.kube/config
```

---

Contains

```text
Cluster Information

User Information

Authentication Details
```

---

View Config

```bash
cat ~/.kube/config
```

---

# Request Without kubeconfig

User Executes

```bash
kubectl get pods
```

---

Without kubeconfig

---

Result

```text
Cluster Not Reachable
```

---

# Authorization

After Authentication succeeds:

```text
Authorization Starts
```

---

Kubernetes asks:

```text
What Permissions Does User Have?
```

---

Examples

```text
Read Pods

Create Pods

Delete Pods

Update Deployments
```

---

# Authorization Decision

User

```text
Suresh
```

---

Permission

```text
Read Pods
```

---

Command

```bash
kubectl get pods
```

---

Result

```text
Allowed
```

---

Command

```bash
kubectl delete deployment nginx
```

---

Permission Missing

---

Result

```text
Forbidden
```

---

# Authentication and Authorization Together

```text
User
 │

 ▼

Authentication

Who Are You?
 │

 ▼

Authorization

What Can You Do?
 │

 ▼

Response
```

---

# Production Example

Organization

```text
Roboshop
```

---

Users

```text
Trainee

Junior Engineer

Senior Engineer

Team Lead
```

---

Authentication

```text
IAM User
```

confirms identity.

---

Authorization

```text
RBAC
```

defines permissions.

---

# Common Authentication Failures

## Invalid Credentials

Cause

```text
Wrong IAM Credentials
```

---

Result

```text
Authentication Failed
```

---

## Missing kubeconfig

Cause

```text
No Cluster Configuration
```

---

Result

```text
Cannot Connect To Cluster
```

---

## Expired Credentials

Cause

```text
Expired Token
```

---

Result

```text
Unauthorized
```

---

# Common Authorization Failures

## Forbidden Error

Example

```bash
kubectl delete pod nginx
```

---

Result

```text
Forbidden
```

---

Cause

```text
Permission Missing
```

---

Authentication succeeded.

---

Authorization failed.

---

# Authentication Error vs Authorization Error

Authentication Error

```text
You Are Unknown
```

---

Authorization Error

```text
You Are Known

But Not Allowed
```

---

# Production EKS Flow

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

RBAC
(Authorization)
      │

      ▼

Response
```

---

# Why Both Are Required?

Without Authentication

```text
Anyone Can Access Cluster
```

---

Without Authorization

```text
Everyone Becomes Admin
```

---

Both are mandatory.

---

# Production Scenario

Requirement

```text
Give Suresh Access
```

---

Steps

```text
Create IAM User

Provide Cluster Access

Map User To Cluster

Create RBAC Rules
```

---

Then

```text
Authentication

+

Authorization
```

work together.

---

# Troubleshooting

Check Cluster Access

```bash
kubectl get nodes
```

---

Check Current User

```bash
aws sts get-caller-identity
```

---

Verify kubeconfig

```bash
kubectl config view
```

---

Update kubeconfig

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name roboshop-dev
```

---

# Common Interview Questions

## What is Authentication?

### Short Answer

Authentication verifies user identity.

### Detailed Explanation

Authentication determines who is making the request before any permissions are evaluated.

### Production Example

EKS uses IAM to verify a developer's identity.

### Follow-Up Questions

- Which service performs authentication in EKS?
- What is kubeconfig?
- What happens if credentials are invalid?

---

## What is Authorization?

### Short Answer

Authorization determines what actions a user can perform.

### Detailed Explanation

After authentication succeeds, Kubernetes checks permissions using RBAC.

### Production Example

A user may read Pods but cannot delete Deployments.

### Follow-Up Questions

- Which component handles authorization?
- What happens if permissions are missing?
- What is a Forbidden error?

---

## Difference Between Authentication and Authorization?

### Short Answer

Authentication verifies identity while authorization verifies permissions.

### Detailed Explanation

Authentication answers "Who are you?" and authorization answers "What can you do?"

### Production Example

IAM verifies the user and RBAC verifies access rights.

### Follow-Up Questions

- Which comes first?
- Can authorization happen without authentication?
- What services handle each in EKS?

---

## How Does Authentication Work In EKS?

### Short Answer

EKS uses AWS IAM for authentication.

### Detailed Explanation

Requests are sent through kubeconfig to EKS, which validates the user using IAM.

### Production Example

A developer uses IAM credentials to access the cluster.

### Follow-Up Questions

- What is aws-auth?
- What is kubeconfig?
- How does IAM integrate with EKS?

---

## What Causes Forbidden Errors?

### Short Answer

Authorization failure.

### Detailed Explanation

The user is authenticated successfully but lacks required permissions.

### Production Example

A trainee can view Pods but cannot delete Deployments.

### Follow-Up Questions

- Is authentication successful?
- Which component denies access?
- How is access granted?

---

# Key Takeaways

```text
Authentication verifies identity.

Authorization verifies permissions.

Authentication happens before authorization.

EKS uses AWS IAM for authentication.

RBAC handles authorization.

kubectl uses kubeconfig to connect to the cluster.

Authentication failure means user is unknown.

Authorization failure means user is known but not allowed.

Both authentication and authorization are required for secure Kubernetes access.

Understanding these concepts is essential before learning RBAC.
```