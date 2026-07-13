# Kubernetes RBAC - Role and RoleBinding

## Introduction

After Authentication and Authorization, Kubernetes needs a mechanism to control:

```text
Who Can Access What?

What Actions Can Be Performed?

Which Namespace Can Be Accessed?
```

---

This is handled through:

```text
RBAC

Role Based Access Control
```

---

RBAC is the most commonly used authorization mechanism in Kubernetes.

---

# What is RBAC?

RBAC stands for:

```text
Role Based Access Control
```

---

RBAC controls:

```text
Users

Groups

Service Accounts
```

and defines:

```text
What Resources

What Actions
```

they can perform.

---

# Real World Example

Organization

```text
Trainee

Junior Engineer

Senior Engineer

Team Lead
```

---

Permissions

```text
Trainee
    ↓
Read Only

Junior Engineer
    ↓
Create + Read

Senior Engineer
    ↓
Create + Read + Update

Team Lead
    ↓
Full Access
```

---

RBAC allows us to implement this model.

---

# RBAC Components

Main Components

```text
Role

RoleBinding
```

---

Later we will learn:

```text
ClusterRole

ClusterRoleBinding
```

---

# RBAC Terminology

RBAC is built using:

```text
Resources

Actions
```

---

# Resources (Nouns)

Resources are Kubernetes objects.

---

Examples

```text
Pods

Services

ConfigMaps

Deployments

Secrets
```

---

Think of them as:

```text
Nouns
```

---

# Actions (Verbs)

Operations performed on resources.

---

Examples

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

Think of them as:

```text
Verbs
```

---

# Easy Interview Answer

```text
Resources = Nouns

Actions = Verbs
```

---

# API Groups

Resources belong to API Groups.

---

Examples

```text
Pods

Services

ConfigMaps
```

belong to:

```text
Core API Group
```

---

Deployments belong to:

```text
apps
```

API Group.

---

# Example

Deployment

```yaml
apiVersion: apps/v1
```

---

Pods

```yaml
apiVersion: v1
```

---

# RBAC Questions

RBAC always answers:

```text
Which API Group?

Which Resource?

Which Action?
```

---

Example

```text
apps

deployments

get
```

---

Meaning

```text
Read Deployments
```

---

# What is a Role?

Role is:

```text
Namespace Level Permission
```

---

Role defines:

```text
Resources

Actions
```

allowed within a namespace.

---

Role DOES NOT grant access.

---

Role only defines permissions.

---

# Role Example

Requirement

```text
Allow Reading Pods
```

in namespace:

```text
roboshop-dev
```

---

Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: Role

metadata:

  name: pod-reader

  namespace: roboshop-dev

rules:

- apiGroups: [""]

  resources: ["pods"]

  verbs:

  - get

  - list

  - watch
```

---

Meaning

```text
Read Pods

Inside roboshop-dev
```

---

# Understanding Role Fields

API Group

```yaml
apiGroups: [""]
```

---

Core Kubernetes Resources.

---

Resources

```yaml
resources:
- pods
```

---

Target Resource.

---

Verbs

```yaml
verbs:

- get

- list

- watch
```

---

Allowed Actions.

---

# What is RoleBinding?

RoleBinding connects:

```text
User

↓

Role
```

---

Without RoleBinding

```text
Role Exists

Permission Not Granted
```

---

# RoleBinding Architecture

```text
User

↓

RoleBinding

↓

Role

↓

Permissions
```

---

# Example RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

  name: pod-reader-binding

  namespace: roboshop-dev

subjects:

- kind: User

  name: suresh

roleRef:

  kind: Role

  name: pod-reader

  apiGroup: rbac.authorization.k8s.io
```

---

Meaning

```text
Suresh

Can Read Pods

In roboshop-dev
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
View Pods

View Services

View Deployments
```

---

Solution

```text
Create Role

Create RoleBinding
```

---

# Trainee Access

Requirement

```text
Read Only
```

---

Resources

```text
Pods

Services

Deployments
```

---

Verbs

```text
get

list

watch
```

---

# Junior Engineer Access

Requirement

```text
Create

Read
```

---

Verbs

```text
create

get

list

watch
```

---

# Senior Engineer Access

Requirement

```text
Create

Read

Update
```

---

Verbs

```text
create

get

list

watch

update

patch
```

---

# Team Lead Access

Requirement

```text
CRUD
```

---

Verbs

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

# Namespace Isolation

Namespace

```text
roboshop-dev
```

---

Role created here.

---

Result

```text
Permissions Limited

To roboshop-dev
```

---

Cannot access:

```text
roboshop-prod
```

---

# Verify Permissions

Check access.

---

Command

```bash
kubectl auth can-i get pods \
--namespace roboshop-dev
```

---

Output

```text
yes
```

or

```text
no
```

---

# Verify Another Action

```bash
kubectl auth can-i delete pods \
--namespace roboshop-dev
```

---

Useful for troubleshooting.

---

# Common Production Roles

Read Only

```text
Developers

Auditors

Trainees
```

---

Developer

```text
Create

Read

Update
```

---

Operations

```text
Full Namespace Access
```

---

# Common Problems

## Role Created But Access Not Working

Cause

```text
RoleBinding Missing
```

---

Role alone is not enough.

---

Need

```text
RoleBinding
```

---

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
Delete Permission Missing
```

---

## Wrong Namespace

Role

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

# Troubleshooting

View Roles

```bash
kubectl get roles -A
```

---

Describe Role

```bash
kubectl describe role role-name
```

---

View RoleBindings

```bash
kubectl get rolebindings -A
```

---

Describe RoleBinding

```bash
kubectl describe rolebinding binding-name
```

---

Check Permission

```bash
kubectl auth can-i get pods
```

---

# Common Interview Questions

## What is RBAC?

### Short Answer

RBAC is Role Based Access Control used to manage permissions in Kubernetes.

### Detailed Explanation

RBAC controls what actions users, groups, and service accounts can perform on Kubernetes resources.

### Production Example

Developers receive read access while Team Leads receive full namespace access.

### Follow-Up Questions

- Why is RBAC needed?
- What resources can RBAC control?
- What objects implement RBAC?

---

## What is a Role?

### Short Answer

A Role defines namespace-level permissions.

### Detailed Explanation

Roles specify which resources and actions are allowed within a namespace.

### Production Example

A Role allows reading Pods in roboshop-dev namespace.

### Follow-Up Questions

- Does Role grant access?
- Is Role namespace scoped?
- What fields exist in a Role?

---

## What is a RoleBinding?

### Short Answer

A RoleBinding assigns a Role to a user, group, or service account.

### Detailed Explanation

RoleBinding connects identities to permissions defined in a Role.

### Production Example

Suresh receives read access through a RoleBinding.

### Follow-Up Questions

- Can RoleBinding bind multiple users?
- Can RoleBinding bind service accounts?
- What happens without RoleBinding?

---

## Difference Between Role and RoleBinding?

### Short Answer

Role defines permissions, RoleBinding grants permissions.

### Detailed Explanation

Role contains rules while RoleBinding connects those rules to identities.

### Production Example

A pod-reader Role exists but users gain access only after RoleBinding is created.

### Follow-Up Questions

- Can Role work without RoleBinding?
- Which one contains users?
- Which one contains verbs?

---

## What Does kubectl auth can-i Do?

### Short Answer

It verifies whether a user can perform a specific action.

### Detailed Explanation

Kubernetes evaluates RBAC permissions and returns yes or no.

### Production Example

Engineers verify access before assigning permissions.

### Follow-Up Questions

- Can it check namespace permissions?
- Can it test delete access?
- Why is it useful?

---

# Key Takeaways

```text
RBAC stands for Role Based Access Control.

Resources are nouns.

Actions are verbs.

Roles define namespace-level permissions.

RoleBindings grant permissions.

Role alone does not provide access.

RoleBinding connects users to roles.

RBAC follows least privilege principles.

kubectl auth can-i is useful for permission verification.

Roles and RoleBindings are the foundation of Kubernetes authorization.
```