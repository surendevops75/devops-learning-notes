# Kubernetes Groups and RBAC Best Practices

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

So far we have been assigning permissions to:

```text
Individual Users
```

---

Example

```text
Suresh

John

Rahim

Robert
```

---

In small environments this works.

---

But in production:

```text
Hundreds Of Engineers
```

may access Kubernetes.

---

Managing permissions individually becomes difficult.

---

Solution

```text
Groups
```

---

# What is a Group?

A Group is:

```text
Collection Of Users
```

---

Example

```text
Trainee Team
```

contains:

```text
John

Rahim

Robert
```

---

Instead of assigning permissions to:

```text
John

Rahim

Robert
```

individually,

---

we assign permissions to:

```text
Trainee Group
```

---

# Why Groups?

Without Groups

```text
John
   ↓

RoleBinding

Rahim
   ↓

RoleBinding

Robert
   ↓

RoleBinding
```

---

Many RoleBindings.

---

Difficult To Manage.

---

With Groups

```text
John
Rahim
Robert

      ↓

Trainee Group

      ↓

RoleBinding
```

---

Single RoleBinding.

---

# Real Production Example

Organization

```text
Roboshop
```

---

Teams

```text
Trainee

Developer

QA

Operations

Platform Team
```

---

Each team receives:

```text
Different Permissions
```

---

# Group Based Access Model

```text
Users

↓

Groups

↓

RoleBinding

↓

Role

↓

Permissions
```

---

# Example Group Structure

```text
Trainee

├── John

├── Rahim

└── Robert
```

---

```text
Developer

├── Suresh

├── Ravi

└── Kumar
```

---

```text
Operations

├── David

├── James

└── Steve
```

---

# Why Companies Prefer Groups?

Benefits

```text
Easy Management

Centralized Access

Faster Onboarding

Faster Offboarding
```

---

# User Onboarding

New Employee

```text
Vijay
```

joins trainee team.

---

Without Groups

```text
Create RoleBinding
```

---

With Groups

```text
Add User To Group
```

---

Access Granted Automatically.

---

# User Offboarding

Employee Leaves.

---

Without Groups

```text
Remove RoleBinding

Remove Policies

Remove Bindings
```

---

With Groups

```text
Remove User

From Group
```

---

Done.

---

# RBAC Components

RBAC uses:

```text
Users

Groups

Service Accounts
```

as identities.

---

RoleBinding can bind:

```text
User

Group

Service Account
```

---

# RoleBinding To User

Example

```yaml
subjects:

- kind: User

  name: suresh
```

---

Access assigned directly.

---

# RoleBinding To Group

Example

```yaml
subjects:

- kind: Group

  name: trainee
```

---

Access assigned to entire group.

---

# Production Scenario

Requirement

```text
Trainees

Can View Resources

Cannot Modify Resources
```

---

Resources

```text
Pods

Services

Deployments
```

---

Permissions

```text
get

list

watch
```

---

# Create Read Only Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: Role

metadata:

  name: trainee-reader

  namespace: roboshop-dev

rules:

- apiGroups: [""]

  resources:

  - pods

  - services

  verbs:

  - get

  - list

  - watch
```

---

# Create Group RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

  name: trainee-binding

  namespace: roboshop-dev

subjects:

- kind: Group

  name: trainee

roleRef:

  kind: Role

  name: trainee-reader

  apiGroup: rbac.authorization.k8s.io
```

---

Result

```text
All Users

Inside trainee Group

Get Read Access
```

---

# Access Flow

```text
John

↓

Authentication

↓

Group Membership

↓

RoleBinding

↓

Role

↓

Read Pods
```

---

# Common Production Groups

## Trainee

Permissions

```text
Read Only
```

---

Actions

```text
get

list

watch
```

---

# Developer

Permissions

```text
Create

Read

Update
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

# QA Team

Permissions

```text
Read

Restart Pods
```

---

Limited access.

---

# Operations Team

Permissions

```text
Namespace Admin
```

---

Can manage deployments.

---

# Platform Team

Permissions

```text
Cluster Administration
```

---

Uses

```text
ClusterRole

ClusterRoleBinding
```

---

# How Groups Work In EKS?

Remember:

```text
Authentication

Handled By IAM
```

---

Authentication succeeds.

---

Then

```text
aws-auth ConfigMap
```

maps users.

---

Example

```yaml
mapUsers:

- userarn: arn:aws:iam::123456789:user/john

  username: john

  groups:

  - trainee
```

---

Meaning

```text
IAM User

↓

Kubernetes Group
```

---

# Production Example

Users

```text
John

Rahim

Robert
```

---

All belong to:

```text
trainee
```

---

Only one RoleBinding required.

---

Benefits

```text
Less Management

Better Security
```

---

# Bad Practice

Creating permissions for:

```text
Every User Individually
```

---

Problems

```text
Hard To Audit

Hard To Manage

Too Many Bindings
```

---

# Good Practice

Create:

```text
Groups
```

---

Assign:

```text
Users To Groups
```

---

Assign:

```text
Permissions To Groups
```

---

# Production Access Strategy

```text
IAM User

↓

Kubernetes Group

↓

RoleBinding

↓

Role

↓

Permissions
```

---

# Least Privilege Principle

Golden Rule

```text
Grant Only

Required Permissions
```

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

# Namespace Isolation

Roboshop Team

```text
roboshop-dev
```

---

Amazon Team

```text
amazon-dev
```

---

Separate Groups

---

Separate RoleBindings

---

No Cross Access.

---

# Common Problems

## User Cannot Access Resources

Cause

```text
Group Missing
```

---

Verify

```text
aws-auth ConfigMap
```

---

# Forbidden Error

Cause

```text
Role Missing

RoleBinding Missing
```

---

Authentication succeeded.

---

Authorization failed.

---

# Wrong Group

User

```text
John
```

---

Expected Group

```text
trainee
```

---

Actual Group

```text
developer
```

---

Result

```text
Unexpected Permissions
```

---

# Troubleshooting

View RoleBindings

```bash
kubectl get rolebinding -A
```

---

Describe RoleBinding

```bash
kubectl describe rolebinding trainee-binding
```

---

View aws-auth

```bash
kubectl get configmap aws-auth \
-n kube-system -o yaml
```

---

Check Permission

```bash
kubectl auth can-i get pods
```

---

Verify Current Identity

```bash
aws sts get-caller-identity
```

---

# Production Best Practices

## Use Groups

Instead Of

```text
User Based Permissions
```

---

## Follow Least Privilege

Grant Minimum Access.

---

## Separate Teams

```text
Developers

QA

Operations

Platform Team
```

---

## Audit Regularly

Review

```text
Roles

RoleBindings

Groups
```

---

## Avoid Cluster Admin

Do not assign:

```text
cluster-admin
```

to every user.

---

# Common Interview Questions

## What is a Group in Kubernetes?

### Short Answer

A Group is a collection of users that share the same permissions.

### Detailed Explanation

Groups simplify permission management by allowing access to be assigned once and inherited by all members.

### Production Example

John, Rahim, and Robert belong to the trainee group.

### Follow-Up Questions

- Why use groups?
- How are groups mapped in EKS?
- Can RoleBindings reference groups?

---

## Why Use Groups Instead of Users?

### Short Answer

Groups simplify RBAC management.

### Detailed Explanation

Permissions are assigned once to the group instead of individually to every user.

### Production Example

A trainee group receives read-only access through a single RoleBinding.

### Follow-Up Questions

- What happens when a new user joins?
- How is offboarding simplified?
- What are auditing benefits?

---

## How Are Groups Mapped in EKS?

### Short Answer

Using aws-auth ConfigMap.

### Detailed Explanation

IAM users are mapped to Kubernetes groups through aws-auth.

### Production Example

John's IAM user is mapped to the trainee Kubernetes group.

### Follow-Up Questions

- Where is aws-auth located?
- What happens if mapping is missing?
- How do you verify group membership?

---

## What is Least Privilege?

### Short Answer

Grant only the permissions required to perform a task.

### Detailed Explanation

Least privilege reduces accidental changes and security risks.

### Production Example

Trainees receive read-only access instead of admin access.

### Follow-Up Questions

- Why is least privilege important?
- What security risks does it prevent?
- Is it a production best practice?

---

# Key Takeaways

```text
Groups simplify RBAC management.

Groups are collections of users.

RoleBindings can reference Groups.

Groups reduce the number of RoleBindings.

EKS maps IAM users to Kubernetes groups through aws-auth.

Groups simplify onboarding and offboarding.

Least privilege should always be followed.

Use group-based access instead of user-based access whenever possible.

Groups are heavily used in enterprise Kubernetes environments.
```