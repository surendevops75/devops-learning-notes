# Production RBAC Patterns and Troubleshooting

## Introduction

In previous chapters we learned:

```text
Authentication

Authorization

Role

RoleBinding

ClusterRole

ClusterRoleBinding

IAM

aws-auth ConfigMap
```

---

Now let's understand:

```text
How RBAC Is Used

In Real Production Environments
```

---

Most companies do NOT give:

```text
Cluster Admin

To Everyone
```

---

Instead they follow:

```text
Least Privilege Principle
```

---

Meaning

```text
Give Only Required Access

Nothing More
```

---

# Why RBAC Is Important?

Imagine:

```text
100 Engineers

Using Same Cluster
```

---

Without RBAC

```text
Anyone Can Delete Production
```

---

Example

```bash
kubectl delete namespace roboshop-prod
```

---

Result

```text
Production Outage
```

---

RBAC prevents this.

---

# Organization Structure

Example Company

```text
Trainee

Junior Engineer

Senior Engineer

Team Lead

Platform Team
```

---

Each role requires:

```text
Different Permissions
```

---

# Trainee Access Pattern

Requirement

```text
View Resources
```

---

Allowed

```text
Pods

Services

Deployments
```

---

Actions

```text
get

list

watch
```

---

Not Allowed

```text
create

update

delete
```

---

Role Example

```yaml
verbs:

- get

- list

- watch
```

---

# Junior Engineer Access Pattern

Requirement

```text
Create Resources

View Resources
```

---

Actions

```text
create

get

list

watch
```

---

Not Allowed

```text
delete
```

---

# Senior Engineer Access Pattern

Requirement

```text
Manage Applications
```

---

Actions

```text
create

get

list

watch

update

patch
```

---

# Team Lead Access Pattern

Requirement

```text
Full Namespace Control
```

---

Actions

```text
create

get

list

watch

update

patch

delete
```

---

# Platform Team Access Pattern

Requirement

```text
Cluster Administration
```

---

Resources

```text
Nodes

StorageClasses

Namespaces

ClusterRoles
```

---

Access

```text
ClusterRole

ClusterRoleBinding
```

---

# Real Production Scenario

Project

```text
Roboshop
```

---

Namespace

```text
roboshop-dev
```

---

Developer

```text
Suresh
```

---

Requirement

```text
Read Pods

Read Services

Read Deployments
```

---

Solution

```text
Role

+

RoleBinding
```

---

# Access Request Flow

Developer Raises Request

---

Example

```text
Need Access To

roboshop-dev Namespace
```

---

Approval Flow

```text
Manager Approval

↓

Platform Team

↓

RBAC Creation

↓

Access Verification
```

---

# Namespace Isolation

Project A

```text
roboshop-dev
```

---

Project B

```text
amazon-dev
```

---

Developer From Roboshop

Should Not Access

```text
amazon-dev
```

---

Solution

```text
Namespace Specific Roles
```

---

# Read Only Access Pattern

Very Common.

---

Used For

```text
Auditors

Managers

Trainees

Support Teams
```

---

Permissions

```text
View Everything

Modify Nothing
```

---

# Production Deployment Team

Requirement

```text
Deploy Applications
```

---

Permissions

```text
Deployments

Services

ConfigMaps
```

---

Not Allowed

```text
Cluster Configuration
```

---

# Database Team Access

Requirement

```text
Access Database Namespace
```

---

Example

```text
mongodb-prod

mysql-prod
```

---

Permissions

```text
Limited To Database Namespaces
```

---

# Common Security Mistakes

## Giving cluster-admin To Everyone

Bad Practice

---

Result

```text
Security Risk

Accidental Deletion

Privilege Escalation
```

---

# Using Wildcards

Bad Example

```yaml
resources:

- "*"

verbs:

- "*"
```

---

Meaning

```text
Everything

On Everything
```

---

Avoid whenever possible.

---

# No Namespace Isolation

Bad Practice

---

Developer Can Access

```text
Every Namespace
```

---

Result

```text
Cross Team Access
```

---

# Principle Of Least Privilege

Golden Rule

---

Give

```text
Minimum Permissions
```

---

Required To Perform Work.

---

Example

Need

```text
Read Pods
```

---

Grant

```text
get

list

watch
```

---

Do Not Grant

```text
delete
```

---

# Production Troubleshooting

## Forbidden Error

Most Common RBAC Issue.

---

Example

```bash
kubectl delete pod nginx
```

---

Output

```text
Forbidden
```

---

Meaning

```text
Authentication Success

Authorization Failed
```

---

# How To Troubleshoot?

First Question

```text
Can User Perform Action?
```

---

Check

```bash
kubectl auth can-i delete pod
```

---

Result

```text
yes

or

no
```

---

# Check Role

```bash
kubectl get role
```

---

Describe

```bash
kubectl describe role role-name
```

---

Verify

```text
Resources

Verbs
```

---

# Check RoleBinding

```bash
kubectl get rolebinding
```

---

Describe

```bash
kubectl describe rolebinding rolebinding-name
```

---

Verify

```text
Correct User

Correct Role
```

---

# Check ClusterRole

```bash
kubectl get clusterrole
```

---

Describe

```bash
kubectl describe clusterrole role-name
```

---

# Check ClusterRoleBinding

```bash
kubectl get clusterrolebinding
```

---

Describe

```bash
kubectl describe clusterrolebinding binding-name
```

---

# Verify Namespace

Very Common Issue.

---

Role Exists In

```text
roboshop-dev
```

---

User Accessing

```text
roboshop-prod
```

---

Result

```text
Forbidden
```

---

# Verify Current Context

```bash
kubectl config current-context
```

---

Verify Namespace

```bash
kubectl config view
```

---

# Production Access Verification

Can Read Pods

```bash
kubectl auth can-i get pods
```

---

Can Create Pods

```bash
kubectl auth can-i create pods
```

---

Can Delete Pods

```bash
kubectl auth can-i delete pods
```

---

Namespace Specific

```bash
kubectl auth can-i get pods \
-n roboshop-dev
```

---

# User Onboarding Process

New Employee

```text
Suresh
```

---

Steps

```text
1. Create IAM User/Role

2. Add aws-auth Mapping

3. Create Role

4. Create RoleBinding

5. Verify Access

6. Document Permissions
```

---

# User Offboarding Process

Employee Leaves Organization.

---

Steps

```text
Remove RoleBinding

Remove aws-auth Mapping

Disable IAM User

Audit Access
```

---

# Production Best Practices

## Use Groups

Instead Of

```text
Individual User Bindings
```

---

Example

```text
Developers

QA

Operations
```

---

# Audit Permissions

Regularly Review

```text
Roles

Bindings

IAM Access
```

---

# Separate Environments

Development

```text
dev
```

---

Production

```text
prod
```

---

Different Permissions.

---

# Never Share Admin Accounts

Each User Should Have

```text
Individual Identity
```

---

# Common Interview Questions

## What Is Principle Of Least Privilege?

### Short Answer

Users should receive only the minimum permissions required to perform their job.

### Detailed Explanation

Least privilege reduces security risks and accidental changes.

### Production Example

A trainee receives read-only access instead of cluster-admin access.

### Follow-Up Questions

- Why is least privilege important?
- What problems does it prevent?
- Is it a security best practice?

---

## How Do You Troubleshoot Forbidden Errors?

### Short Answer

Verify RBAC permissions using kubectl auth can-i and inspect Roles and RoleBindings.

### Detailed Explanation

Forbidden indicates authorization failure. Authentication succeeded but RBAC denied the request.

### Production Example

A developer cannot delete Pods because delete permission is not present in the assigned Role.

### Follow-Up Questions

- Is authentication successful?
- Which command verifies permissions?
- Which objects should be checked?

---

## Why Should We Avoid cluster-admin?

### Short Answer

cluster-admin grants unrestricted access to the entire cluster.

### Detailed Explanation

Misuse can result in security incidents, accidental deletions, and privilege escalation.

### Production Example

Developers receive namespace-level permissions instead of cluster-admin access.

### Follow-Up Questions

- Who should receive cluster-admin?
- What risks exist?
- What alternatives exist?

---

## What Is Namespace Isolation?

### Short Answer

Namespace isolation restricts users to specific projects or environments.

### Detailed Explanation

RBAC permissions are limited to the namespace where Roles and RoleBindings exist.

### Production Example

Roboshop developers cannot access Amazon project namespaces.

### Follow-Up Questions

- Why is namespace isolation important?
- Does Role work across namespaces?
- How is access granted?

---

## How Would You Give Suresh Access To roboshop-dev?

### Short Answer

Create IAM access, map user through aws-auth, create Role, and create RoleBinding.

### Detailed Explanation

Authentication is handled through IAM and aws-auth. Authorization is handled through RBAC.

### Production Example

Suresh receives read-only access to Pods, Services, and Deployments in roboshop-dev namespace.

### Follow-Up Questions

- What handles authentication?
- What handles authorization?
- Where is aws-auth located?

---

# Key Takeaways

```text
RBAC should follow least privilege principles.

Different teams require different permissions.

Role and RoleBinding provide namespace access.

ClusterRole and ClusterRoleBinding provide cluster access.

Avoid cluster-admin whenever possible.

Forbidden errors usually indicate RBAC issues.

kubectl auth can-i is the most useful RBAC troubleshooting command.

Namespace isolation is a critical production practice.

User onboarding and offboarding should follow a documented access process.

RBAC is one of the most important Kubernetes security controls.
```