# Least Privilege

## Introduction

The Principle of Least Privilege (PoLP) is a security concept that ensures users, applications, services, and systems are granted only the minimum permissions required to perform their tasks.

In DevSecOps, Least Privilege minimizes the attack surface, limits the impact of compromised accounts, and reduces the risk of accidental or malicious actions.

---

## Why Do We Need Least Privilege?

Modern cloud environments contain numerous users, applications, containers, and automation tools.

Without Least Privilege:

- Users receive excessive permissions
- Attackers gain unrestricted access after compromising an account
- Accidental deletion of resources becomes possible
- Insider threats become more dangerous
- Compliance requirements become difficult to achieve

Applying Least Privilege significantly improves the security posture of an organization.

---

## What is Least Privilege?

Least Privilege means granting only the permissions necessary to complete a specific task—and nothing more.

Instead of giving broad administrative access, permissions should be narrowly scoped.

Example:

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

↓

Read Secrets
```

Only the required permissions are granted.

---

# How Least Privilege Works

```text
User Requests Access

↓

Verify Identity

↓

Determine Job Role

↓

Assign Minimum Permissions

↓

Perform Required Task

↓

Access Ends
```

---

# Where Least Privilege Should Be Applied

## Users

Grant users only the permissions required for their responsibilities.

Example

```text
Developer

↓

Deploy Applications

↓

Read Logs

↓

No Database Access
```

---

## Applications

Applications should only access the services they require.

Example

```text
Application

↓

Read Database

↓

Read Secrets

↓

No IAM Administration
```

---

## CI/CD Pipelines

Pipelines should receive temporary permissions only during deployment.

Example

```text
Pipeline

↓

Assume IAM Role

↓

Deploy Resources

↓

Permissions Expire
```

---

## Containers

Containers should not run as root.

Instead

```text
Container

↓

Non-Root User

↓

Limited Linux Capabilities

↓

Restricted Filesystem
```

---

## Kubernetes

Use RBAC to limit Kubernetes access.

Example

```text
Developer

↓

Namespace Access

↓

Deploy Pods

↓

View Logs

↓

Cannot Modify Cluster Roles
```

---

## Cloud Resources

Grant access only to required services.

Example

```text
Application

↓

Read S3

↓

Read Secrets

↓

No EC2 Administration
```

---

# Least Privilege Architecture

```text
Users

↓

Authentication

↓

IAM

↓

Least Privilege Policies

↓

Applications

↓

Cloud Resources
```

---

# Benefits of Least Privilege

- Reduces attack surface
- Prevents privilege escalation
- Limits insider threats
- Reduces accidental changes
- Improves compliance
- Simplifies auditing
- Protects sensitive resources
- Minimizes damage from compromised accounts

---

# Privilege Escalation Example

Without Least Privilege

```text
Developer Account

↓

Administrator Access

↓

Account Compromised

↓

Entire AWS Environment Compromised
```

With Least Privilege

```text
Developer Account

↓

Limited Permissions

↓

Account Compromised

↓

Only Limited Resources Affected
```

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

CI/CD Pipeline

↓

Temporary IAM Role

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

Each stage receives only the permissions required for that specific task.

---

## Production Workflow

```text
User Login

↓

Authentication

↓

IAM Policy Evaluation

↓

Least Privilege Granted

↓

Perform Task

↓

Access Logged
```

---

## Production Example

A Jenkins pipeline deploys applications to Amazon EKS.

Instead of providing AdministratorAccess:

```text
Jenkins

↓

Assume Deployment Role

↓

Deploy to EKS

↓

Update Deployment

↓

Role Expires
```

The pipeline cannot modify unrelated AWS resources.

---

## Best Practices

- Always follow Least Privilege
- Use IAM Roles instead of long-lived credentials
- Grant permissions based on job roles
- Review permissions regularly
- Remove unused permissions
- Enable MFA
- Audit privileged accounts
- Use temporary credentials
- Separate production and development permissions
- Automate permission reviews

---

## Common Mistakes

- Granting AdministratorAccess to everyone
- Sharing privileged accounts
- Never reviewing permissions
- Using root accounts for daily work
- Ignoring inactive users
- Hardcoding privileged credentials
- Giving applications unnecessary permissions
- Running containers as root

---

# Common Troubleshooting

## Issue 1

### User Receives "Access Denied"

**Cause**

Required permission is missing.

**Resolution**

```text
Review IAM Policy

↓

Identify Required Action

↓

Grant Minimum Permission

↓

Retry
```

---

## Issue 2

### Application Cannot Read Secrets

**Cause**

Application role lacks permission.

**Resolution**

```text
Review IAM Role

↓

Grant Read Secret Permission

↓

Restart Application

↓

Verify Access
```

---

## Issue 3

### CI/CD Pipeline Deployment Fails

**Cause**

Deployment role does not have required permissions.

**Resolution**

```text
Review Deployment Role

↓

Update IAM Policy

↓

Run Pipeline

↓

Verify Deployment
```

---

## Issue 4

### Kubernetes User Cannot Deploy Resources

**Cause**

RBAC permissions are insufficient.

**Resolution**

```text
Review Role

↓

Review RoleBinding

↓

Grant Required Permission

↓

Retry Deployment
```

---

## Issue 5

### Security Audit Finds Excessive Permissions

**Cause**

Users accumulated unnecessary privileges over time.

**Resolution**

```text
Review Permissions

↓

Remove Unused Access

↓

Apply Least Privilege

↓

Perform Audit Again
```

---

# Production Interview Questions

## Question 1

### What is the Principle of Least Privilege?

Least Privilege is the practice of granting users, applications, and services only the minimum permissions required to perform their tasks.

---

## Question 2

### Why is Least Privilege important in DevSecOps?

It reduces the attack surface, limits privilege escalation, minimizes insider threats, and improves overall security.

---

## Question 3

### How does Least Privilege improve cloud security?

By preventing identities from accessing unnecessary resources, reducing the impact of compromised credentials.

---

## Question 4

### How is Least Privilege implemented in AWS?

Using IAM Policies, IAM Roles, permission boundaries, resource-based policies, and temporary credentials.

---

## Question 5

### Why should CI/CD pipelines use temporary credentials?

Temporary credentials automatically expire, reducing the risk of credential theft and unauthorized access.

---

## Question 6

### How does Kubernetes implement Least Privilege?

Through RBAC, Service Accounts, Network Policies, Pod Security Standards, and namespace isolation.

---

## Question 7

### Why should containers avoid running as the root user?

Running as root increases the impact of container compromise and may allow attackers to gain elevated privileges on the host.

---

## Question 8

### How often should permissions be reviewed?

Permissions should be reviewed regularly, after organizational changes, and during security audits to remove unnecessary access.

---

## Question 9

### What is privilege escalation?

Privilege escalation occurs when a user or attacker gains permissions beyond those originally granted, allowing unauthorized actions.

---

## Question 10

### How does Least Privilege integrate into DevSecOps?

Least Privilege is applied across developers, CI/CD pipelines, cloud resources, Kubernetes clusters, applications, and runtime environments to ensure every component has only the permissions required to perform its function.

---

# Key Takeaways

- Least Privilege grants only the minimum permissions required.
- It significantly reduces the attack surface and limits privilege escalation.
- Apply Least Privilege to users, applications, containers, Kubernetes, and CI/CD pipelines.
- Use IAM Roles, RBAC, temporary credentials, and regular permission reviews.
- Least Privilege is one of the most fundamental security principles in every DevSecOps environment.