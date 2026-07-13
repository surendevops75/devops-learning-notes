# Kubernetes ClusterRole and ClusterRoleBinding

## Introduction

In the previous chapter we learned:

```text
Role

RoleBinding
```

---

Role provides:

```text
Namespace Level Access
```

---

Example

```text
roboshop-dev
```

---

A Role created in:

```text
roboshop-dev
```

cannot grant permissions in:

```text
roboshop-prod
```

---

Sometimes users need:

```text
Cluster Wide Access
```

---

Examples

```text
View Nodes

View Namespaces

View Storage Classes

View Cluster Information
```

---

For these requirements Kubernetes provides:

```text
ClusterRole

ClusterRoleBinding
```

---

# Why Role Is Not Enough?

Suppose user:

```text
Suresh
```

needs access to:

```text
All Namespaces
```

---

Role works only inside:

```text
Single Namespace
```

---

Creating Roles everywhere becomes difficult.

---

Solution

```text
ClusterRole
```

---

# Role vs ClusterRole

## Role

```text
Namespace Scoped
```

---

Examples

```text
Pods

Services

Deployments

ConfigMaps
```

inside one namespace.

---

## ClusterRole

```text
Cluster Scoped
```

---

Examples

```text
Nodes

Namespaces

StorageClasses

PersistentVolumes
```

---

Can also grant access across:

```text
All Namespaces
```

---

# What is ClusterRole?

ClusterRole defines:

```text
Resources

Actions
```

for:

```text
Entire Cluster
```

---

Just like Role:

```text
Defines Permissions
```

---

But ClusterRole is:

```text
Cluster Scoped
```

---

# ClusterRole Does Not Grant Access

Important

---

ClusterRole only defines:

```text
Permissions
```

---

Need:

```text
ClusterRoleBinding
```

to grant access.

---

# ClusterRole Example

Requirement

```text
Read All Nodes
```

---

YAML

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: ClusterRole

metadata:

  name: node-reader

rules:

- apiGroups: [""]

  resources:

  - nodes

  verbs:

  - get

  - list

  - watch
```

---

Meaning

```text
Read Nodes

Across Cluster
```

---

# Understanding ClusterRole

API Group

```yaml
apiGroups: [""]
```

---

Resources

```yaml
resources:
- nodes
```

---

Actions

```yaml
verbs:
- get
- list
- watch
```

---

# What is ClusterRoleBinding?

ClusterRoleBinding connects:

```text
User

↓

ClusterRole
```

---

Without ClusterRoleBinding

```text
Permission Exists

Access Not Granted
```

---

# ClusterRoleBinding Architecture

```text
User

↓

ClusterRoleBinding

↓

ClusterRole

↓

Permissions
```

---

# ClusterRoleBinding Example

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: ClusterRoleBinding

metadata:

  name: node-reader-binding

subjects:

- kind: User

  name: suresh

roleRef:

  kind: ClusterRole

  name: node-reader

  apiGroup: rbac.authorization.k8s.io
```

---

Meaning

```text
Suresh

Can Read Nodes
```

---

# Cluster Resources

Common Cluster Resources

```text
Nodes

Namespaces

PersistentVolumes

StorageClasses

ClusterRoles

ClusterRoleBindings
```

---

These resources are not tied to a namespace.

---

# Production Example

Requirement

```text
Operations Team

Needs Read Access

To Entire Cluster
```

---

Access Needed

```text
Nodes

Namespaces

Storage Classes

Persistent Volumes
```

---

Solution

```text
ClusterRole

+

ClusterRoleBinding
```

---

# Read Only Cluster Access

Requirement

```text
Cluster Visibility

No Modification
```

---

ClusterRole

```yaml
verbs:

- get

- list

- watch
```

---

Benefits

```text
Safe

Auditable

Least Privilege
```

---

# Namespace Admin vs Cluster Admin

Namespace Admin

```text
Role

RoleBinding
```

---

Scope

```text
Single Namespace
```

---

Cluster Admin

```text
ClusterRole

ClusterRoleBinding
```

---

Scope

```text
Entire Cluster
```

---

# Real Production Scenario

Requirement

```text
Suresh

Needs Cluster Describe Access
```

---

Access Needed

```text
Nodes

Namespaces

PVs

Storage Classes
```

---

Solution

```text
ClusterRole
```

with:

```text
get

list

watch
```

permissions.

---

Bind using:

```text
ClusterRoleBinding
```

---

# Built-In ClusterRoles

Kubernetes provides built-in roles.

---

Examples

```text
cluster-admin

admin

edit

view
```

---

# cluster-admin

Highest Privilege.

---

Can perform:

```text
All Actions

On All Resources
```

---

Equivalent to:

```text
Super Administrator
```

---

# view

Read Only Access.

---

Can:

```text
Get

List

Watch
```

---

Cannot

```text
Create

Update

Delete
```

---

# Using Built-In View Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: ClusterRoleBinding

metadata:

  name: suresh-view

subjects:

- kind: User

  name: suresh

roleRef:

  kind: ClusterRole

  name: view

  apiGroup: rbac.authorization.k8s.io
```

---

Meaning

```text
Suresh

Gets Read Only Access
```

---

# Verify Permissions

Check

```bash
kubectl auth can-i get nodes
```

---

Result

```text
yes
```

or

```text
no
```

---

Check Delete Permission

```bash
kubectl auth can-i delete nodes
```

---

Result

```text
Denied
```

for read-only users.

---

# Role vs ClusterRole

| Feature | Role | ClusterRole |
|----------|----------|----------|
| Scope | Namespace | Cluster |
| Resources | Namespace Resources | Cluster Resources |
| Binding | RoleBinding | ClusterRoleBinding |
| Example | Pods | Nodes |
| Access Range | Single Namespace | Entire Cluster |

---

# RoleBinding Can Use ClusterRole

Interesting Interview Question.

---

RoleBinding can reference:

```text
ClusterRole
```

---

Result

```text
ClusterRole Permissions

Limited To Namespace
```

---

Example

```text
Use Built-In View Role

Only In roboshop-dev
```

---

# Common Problems

## ClusterRole Exists But Access Not Working

Cause

```text
ClusterRoleBinding Missing
```

---

Need

```text
ClusterRoleBinding
```

---

## Forbidden Error

Cause

```text
Missing Verb
```

---

Example

```text
Delete Not Granted
```

---

Result

```text
Forbidden
```

---

## Too Much Access

Cause

```text
cluster-admin Assigned
```

to everyone.

---

Risk

```text
Security Violation
```

---

# Production Best Practices

## Principle Of Least Privilege

Grant only:

```text
Required Permissions
```

---

## Avoid cluster-admin

Use only when necessary.

---

## Use Read Only Roles

For:

```text
Auditors

Developers

Trainees
```

---

## Separate Teams

```text
Developers

Operations

Security
```

should have different permissions.

---

# Troubleshooting

View ClusterRoles

```bash
kubectl get clusterroles
```

---

Describe ClusterRole

```bash
kubectl describe clusterrole role-name
```

---

View ClusterRoleBindings

```bash
kubectl get clusterrolebindings
```

---

Describe Binding

```bash
kubectl describe clusterrolebinding binding-name
```

---

Check Permission

```bash
kubectl auth can-i get nodes
```

---

# Common Interview Questions

## What is a ClusterRole?

### Short Answer

A ClusterRole defines cluster-wide permissions.

### Detailed Explanation

ClusterRoles define actions on cluster resources such as Nodes, Namespaces, and StorageClasses.

### Production Example

Operations engineers receive read access to Nodes and Namespaces.

### Follow-Up Questions

- Is ClusterRole namespace scoped?
- Can ClusterRole access nodes?
- Does ClusterRole grant access?

---

## What is a ClusterRoleBinding?

### Short Answer

A ClusterRoleBinding grants ClusterRole permissions to users, groups, or service accounts.

### Detailed Explanation

ClusterRoleBinding connects identities to ClusterRoles.

### Production Example

Suresh receives cluster-wide read access using ClusterRoleBinding.

### Follow-Up Questions

- Can ClusterRole work without ClusterRoleBinding?
- What subjects can be bound?
- Is ClusterRoleBinding cluster scoped?

---

## Difference Between Role and ClusterRole?

### Short Answer

Role is namespace scoped while ClusterRole is cluster scoped.

### Detailed Explanation

Roles manage namespace resources and ClusterRoles manage cluster-wide resources.

### Production Example

Role grants Pod access in roboshop-dev while ClusterRole grants Node access across the cluster.

### Follow-Up Questions

- Which one accesses Nodes?
- Which one accesses Pods?
- Which one uses ClusterRoleBinding?

---

## What is cluster-admin?

### Short Answer

cluster-admin is the highest privilege ClusterRole in Kubernetes.

### Detailed Explanation

It provides full administrative access to all resources in the cluster.

### Production Example

Platform administrators use cluster-admin for cluster management.

### Follow-Up Questions

- Should everyone get cluster-admin?
- What are the risks?
- What alternatives exist?

---

## Can RoleBinding Reference ClusterRole?

### Short Answer

Yes.

### Detailed Explanation

A RoleBinding can reference a ClusterRole and restrict those permissions to a specific namespace.

### Production Example

The built-in view ClusterRole is granted only in roboshop-dev namespace.

### Follow-Up Questions

- Why would you do this?
- Does it become cluster-wide access?
- Is ClusterRoleBinding required?

---

# Key Takeaways

```text
ClusterRole provides cluster-wide permissions.

ClusterRoleBinding grants ClusterRole permissions.

Cluster resources include Nodes, Namespaces, PVs, and StorageClasses.

ClusterRole does not grant access by itself.

ClusterRoleBinding connects users to ClusterRoles.

cluster-admin provides full access.

Role is namespace scoped.

ClusterRole is cluster scoped.

RoleBinding can reference ClusterRole.

Always follow least privilege principles.
```