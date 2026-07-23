# IAM Best Practices

## Introduction

Identity and Access Management (IAM) is the practice of controlling who can access cloud resources, applications, and services, and what actions they are allowed to perform.

In DevSecOps, IAM ensures that users, applications, services, and automation tools have only the permissions they need to perform their tasks securely.

---

## Why Do We Need IAM?

Every organization has multiple users, applications, CI/CD pipelines, and cloud resources.

Without proper IAM:

- Unauthorized users gain access
- Excessive permissions increase security risks
- Sensitive resources become exposed
- Insider threats become difficult to control
- Compliance requirements cannot be met

IAM provides secure authentication and authorization across the organization.

---

## What is IAM?

IAM consists of two fundamental concepts:

- Authentication – Verifying identity
- Authorization – Determining permissions

Example:

```text
Developer Login

↓

Identity Verified

↓

Permissions Checked

↓

Access Granted
```

---

# IAM Components

## Users

Human identities.

Examples

- DevOps Engineers
- Developers
- Security Engineers
- Database Administrators

---

## Groups

A collection of users with similar permissions.

Example

```text
Developers Group

├── User A

├── User B

└── User C
```

---

## Roles

Roles provide temporary permissions to users, applications, or cloud services.

Examples

- EC2 IAM Role
- EKS IAM Role
- Lambda Execution Role

---

## Policies

Policies define what actions are allowed or denied.

Example

```text
Developer

↓

IAM Policy

↓

Allow

├── Read S3

├── Deploy EKS

└── Read Secrets

↓

Deny

└── Delete Production Database
```

---

## Permissions

Permissions define which actions can be performed.

Examples

- Read
- Write
- Delete
- List
- Execute

---

# Authentication vs Authorization

## Authentication

Verifies identity.

Examples

- Username & Password
- MFA
- SSH Keys
- SSO

---

## Authorization

Determines what authenticated users can do.

Example

```text
Login Successful

↓

Check IAM Policy

↓

Allow Resource Access
```

---

# Principle of Least Privilege

Every identity should receive only the minimum permissions required.

Example

Instead of

```text
Developer

↓

Administrator Access
```

Use

```text
Developer

↓

Read Repository

↓

Deploy Application

↓

View Logs
```

Only the required permissions are granted.

---

# Multi-Factor Authentication (MFA)

MFA requires multiple authentication factors before access is granted.

Example

```text
Username

↓

Password

↓

OTP

↓

Access Granted
```

Benefits

- Prevents account compromise
- Protects privileged accounts
- Meets compliance requirements

---

# Role-Based Access Control (RBAC)

Permissions are assigned to roles instead of individual users.

```text
Developer Role

↓

Deploy Applications

↓

Read Logs

↓

View Resources
```

Benefits

- Easier administration
- Consistent permissions
- Reduced human error

---

# Temporary Credentials

Avoid long-lived credentials.

Instead use:

```text
Application

↓

IAM Role

↓

Temporary Token

↓

AWS Resources
```

Benefits

- Reduced credential exposure
- Automatic expiration
- Better security

---

# IAM Architecture

```text
Users

↓

Authentication

↓

IAM

↓

Authorization

↓

Cloud Resources

├── EC2

├── EKS

├── S3

├── RDS

└── Secrets Manager
```

---

# DevSecOps Pipeline Integration

```text
Developer

↓

Git Commit

↓

CI/CD Pipeline

↓

IAM Authentication

↓

Temporary Credentials

↓

Terraform

↓

Deploy Infrastructure

↓

Kubernetes

↓

Application

↓

Production
```

IAM ensures secure authentication throughout the pipeline without exposing credentials.

---

## Production Workflow

```text
Developer

↓

Login

↓

MFA

↓

IAM Verification

↓

Assume Role

↓

Temporary Credentials

↓

Deploy Resources
```

---

## Production Example

A GitHub Actions pipeline deploys infrastructure to AWS.

Instead of storing AWS Access Keys:

```text
GitHub Actions

↓

OIDC Authentication

↓

Assume IAM Role

↓

Temporary Credentials

↓

Terraform Apply

↓

Deploy Infrastructure
```

No long-lived credentials are stored in the repository.

---

## Best Practices

- Enable MFA for every privileged account
- Follow Least Privilege
- Use IAM Roles instead of access keys
- Rotate credentials regularly
- Use temporary credentials
- Review IAM policies periodically
- Remove unused users
- Monitor IAM activities
- Enable audit logging
- Use Single Sign-On where possible

---

## Common Mistakes

- Granting AdministratorAccess unnecessarily
- Sharing IAM users
- Disabling MFA
- Using root account daily
- Hardcoding access keys
- Never reviewing IAM policies
- Creating overly permissive roles
- Forgetting to remove inactive users

---

# Common Troubleshooting

## Issue 1

### User Cannot Access AWS Resources

**Cause**

Missing IAM permissions.

**Resolution**

```text
Verify User

↓

Review IAM Policy

↓

Grant Required Permission

↓

Retry Access
```

---

## Issue 2

### CI/CD Pipeline Authentication Fails

**Cause**

Incorrect IAM Role configuration.

**Resolution**

```text
Verify Trust Policy

↓

Verify Role

↓

Assume Role

↓

Run Pipeline
```

---

## Issue 3

### Application Receives Access Denied

**Cause**

Service lacks required IAM Role permissions.

**Resolution**

```text
Review IAM Role

↓

Update Policy

↓

Restart Application

↓

Verify Access
```

---

## Issue 4

### Compromised Access Keys

**Cause**

Access keys were exposed publicly.

**Resolution**

```text
Disable Keys

↓

Generate New Keys

↓

Update Applications

↓

Monitor Activity
```

---

## Issue 5

### Excessive Permissions Found During Audit

**Cause**

Policies violate Least Privilege.

**Resolution**

```text
Review Policies

↓

Remove Unused Permissions

↓

Apply Least Privilege

↓

Audit Again
```

---

# Production Interview Questions

## Question 1

### What is IAM?

IAM is the framework used to manage identities and control access to cloud resources through authentication and authorization.

---

## Question 2

### What is the difference between Authentication and Authorization?

Authentication verifies identity, while Authorization determines what actions an authenticated identity is allowed to perform.

---

## Question 3

### What is the Principle of Least Privilege?

It is the practice of granting only the minimum permissions required to perform a specific task, reducing the attack surface.

---

## Question 4

### Why should IAM Roles be preferred over Access Keys?

IAM Roles provide temporary credentials that automatically expire, reducing the risk of credential leakage and improving security.

---

## Question 5

### What is MFA?

Multi-Factor Authentication requires users to provide multiple forms of verification before access is granted, improving account security.

---

## Question 6

### What is RBAC?

Role-Based Access Control assigns permissions to roles instead of individual users, simplifying permission management and improving consistency.

---

## Question 7

### Why should the root account not be used for daily operations?

The root account has unrestricted access. Using it regularly increases the risk of accidental changes and security breaches.

---

## Question 8

### How can IAM improve CI/CD security?

IAM enables pipelines to authenticate using temporary credentials and roles instead of storing permanent secrets in repositories.

---

## Question 9

### How often should IAM policies be reviewed?

IAM policies should be reviewed regularly, especially after organizational changes, security incidents, or compliance audits.

---

## Question 10

### How does IAM support DevSecOps?

IAM secures users, applications, automation tools, and cloud resources by enforcing authentication, authorization, Least Privilege, temporary credentials, and continuous access monitoring throughout the software delivery lifecycle.

---

# Key Takeaways

- IAM controls who can access resources and what actions they can perform.
- Authentication verifies identity, while Authorization determines permissions.
- Always follow the Principle of Least Privilege.
- Use IAM Roles and temporary credentials instead of long-lived access keys.
- Strong IAM practices are essential for building secure and compliant DevSecOps environments.