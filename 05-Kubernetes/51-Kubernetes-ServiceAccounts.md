# Kubernetes Service Accounts

## Introduction

In Kubernetes we have two types of identities:

```text
Human Identities

Non-Human Identities
```

---

Examples

Human Identities

```text
DevOps Engineers

Developers

Platform Team

Administrators
```

---

These users access Kubernetes using:

```text
kubectl

IAM Users

IAM Roles

Certificates
```

---

Non-Human Identities

```text
Pods

Applications

Controllers

Operators
```

---

These workloads need an identity too.

---

Kubernetes provides:

```text
Service Accounts
```

for workloads.

---

# What is a Service Account?

A Service Account is:

```text
A Kubernetes Identity

Used By Pods
```

---

Easy Interview Answer

```text
A Service Account is a non-human account used by Pods to interact with Kubernetes resources and external services.
```

---

# Human User vs Service Account

Human User

```text
Surendra

Suresh

John

Robert
```

---

Uses

```text
kubectl

AWS CLI

Kubernetes Dashboard
```

---

Service Account

```text
catalogue-sa

payment-sa

secret-reader
```

---

Used By

```text
Pods

Deployments

StatefulSets
```

---

# Why Do Pods Need Identity?

Suppose:

```text
Catalogue Pod
```

needs access to:

```text
ConfigMap

Secret
```

---

How does Kubernetes know:

```text
Who Is Making Request?
```

---

Answer

```text
Service Account
```

---

# Real Production Example

Application

```text
Catalogue Service
```

---

Needs

```text
ConfigMap

Secret

Database Credentials
```

---

Flow

```text
Catalogue Pod
       │

       ▼

Service Account
       │

       ▼

Kubernetes API
       │

       ▼

ConfigMap
```

---

Without Service Account

```text
No Identity
```

---

# Default Service Account

Every Namespace gets:

```text
default
```

Service Account automatically.

---

Verify

```bash
kubectl get sa
```

---

Output

```text
NAME

default
```

---

# Service Account Creation

Create

```yaml
apiVersion: v1

kind: ServiceAccount

metadata:

  name: catalogue-sa

  namespace: roboshop
```

---

Apply

```bash
kubectl apply -f serviceaccount.yaml
```

---

Verify

```bash
kubectl get sa -n roboshop
```

---

Output

```text
catalogue-sa
```

---

# Using Service Account in Pod

Creating Service Account alone is not enough.

---

Pod must use it.

---

Example

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: nginx

spec:

  serviceAccountName: catalogue-sa

  containers:

  - name: nginx

    image: nginx
```

---

Important Line

```yaml
serviceAccountName: catalogue-sa
```

---

Meaning

```text
Run Pod

Using catalogue-sa
```

---

# Deployment Example

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

      serviceAccountName: catalogue-sa

      containers:

      - name: catalogue

        image: catalogue:1.0
```

---

# Verify Service Account Used By Pod

Describe Pod

```bash
kubectl describe pod pod-name
```

---

Output

```text
Service Account:
catalogue-sa
```

---

# Service Account Token

When Pod starts:

```text
Kubernetes Generates

A Service Account Token
```

---

Token is mounted into Pod.

---

Purpose

```text
Authentication
```

---

# Verify Token

Login to Pod

```bash
kubectl exec -it pod-name -- bash
```

---

Check

```bash
ls /var/run/secrets/kubernetes.io/serviceaccount
```

---

Output

```text
ca.crt

namespace

token
```

---

# What Is Token Used For?

When Pod calls:

```text
Kubernetes API
```

---

Token proves:

```text
Pod Identity
```

---

Flow

```text
Pod

↓

Token

↓

Kubernetes API

↓

Authentication
```

---

# Service Account Authentication Flow

```text
Pod
 │

 ▼

Service Account
 │

 ▼

Token
 │

 ▼

API Server
 │

 ▼

Authentication
```

---

# Service Account Without Permissions

Important Concept.

---

Service Account only provides:

```text
Identity
```

---

It does NOT automatically provide:

```text
Permissions
```

---

Need

```text
Role

RoleBinding
```

---

# Production Example

Service Account

```text
catalogue-sa
```

---

Needs

```text
Read ConfigMaps
```

---

Create Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: Role

metadata:

  name: configmap-reader

rules:

- apiGroups: [""]

  resources:

  - configmaps

  verbs:

  - get

  - list
```

---

Create RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1

kind: RoleBinding

metadata:

  name: configmap-reader-binding

subjects:

- kind: ServiceAccount

  name: catalogue-sa

roleRef:

  kind: Role

  name: configmap-reader

  apiGroup: rbac.authorization.k8s.io
```

---

Result

```text
Catalogue Pod

Can Read ConfigMaps
```

---

# Service Account vs Role

Service Account

```text
Who Are You?
```

---

Role

```text
What Can You Do?
```

---

RoleBinding

```text
Connect Identity

To Permissions
```

---

# Service Account vs User

| Feature | User | Service Account |
|----------|----------|----------|
| Human | Yes | No |
| Used By | Engineers | Pods |
| Authentication | IAM / Certificates | Token |
| Scope | External | Kubernetes |
| Purpose | Human Access | Workload Access |

---

# Common Production Use Cases

## Access ConfigMaps

```text
Application Reads Configuration
```

---

## Access Secrets

```text
Application Reads Passwords
```

---

## Access Kubernetes API

```text
Custom Controllers

Operators
```

---

## Access External Services

```text
AWS Secrets Manager

Parameter Store

S3
```

---

(Using IRSA, covered in next chapter)

---

# Namespace Scope

Service Accounts are:

```text
Namespace Scoped
```

---

Example

```text
roboshop/catalogue-sa
```

---

Cannot automatically be used in:

```text
amazon namespace
```

---

# List Service Accounts

Current Namespace

```bash
kubectl get sa
```

---

All Namespaces

```bash
kubectl get sa -A
```

---

# Describe Service Account

```bash
kubectl describe sa catalogue-sa
```

---

Useful for troubleshooting.

---

# Common Problems

## Pod Uses Default Service Account

Cause

```text
serviceAccountName

Not Specified
```

---

Result

```text
default Service Account Used
```

---

## Forbidden Error

Cause

```text
Role Missing

RoleBinding Missing
```

---

Service Account authenticated successfully.

---

Authorization failed.

---

## Service Account Not Found

Cause

```text
Wrong Namespace
```

---

Verify

```bash
kubectl get sa -A
```

---

# Security Best Practices

## Avoid Default Service Account

Bad Practice

```text
All Pods

Use default
```

---

Good Practice

```text
One Service Account

Per Application
```

---

## Least Privilege

Grant Only

```text
Required Permissions
```

---

Avoid

```text
Cluster Admin
```

for Pods.

---

## Separate Applications

Example

```text
catalogue-sa

payment-sa

user-sa
```

---

Each gets different permissions.

---

# Troubleshooting

View Service Accounts

```bash
kubectl get sa
```

---

Describe Service Account

```bash
kubectl describe sa catalogue-sa
```

---

Check Pod

```bash
kubectl describe pod pod-name
```

---

Verify Token

```bash
kubectl exec -it pod-name -- ls \
/var/run/secrets/kubernetes.io/serviceaccount
```

---

Verify Permissions

```bash
kubectl auth can-i get configmaps \
--as=system:serviceaccount:roboshop:catalogue-sa
```

---

# Common Interview Questions

## What is a Service Account?

### Short Answer

A Service Account is a Kubernetes identity used by Pods.

### Detailed Explanation

Service Accounts allow workloads to authenticate with Kubernetes and other services.

### Production Example

Catalogue Pods use a Service Account to access ConfigMaps and Secrets.

### Follow-Up Questions

- Is a Service Account a user?
- Can Pods run without Service Accounts?
- What is the default Service Account?

---

## Why Do Pods Need Service Accounts?

### Short Answer

Pods need an identity to authenticate with Kubernetes resources.

### Detailed Explanation

Without an identity Kubernetes cannot determine who is making requests.

### Production Example

Catalogue Pods authenticate before reading ConfigMaps.

### Follow-Up Questions

- How is identity established?
- What is the Service Account token?
- What happens without a Service Account?

---

## Does Service Account Provide Permissions?

### Short Answer

No.

### Detailed Explanation

Service Accounts provide identity only. Permissions come from Roles and RoleBindings.

### Production Example

catalogue-sa needs a Role and RoleBinding before accessing ConfigMaps.

### Follow-Up Questions

- What provides permissions?
- Can Service Account work alone?
- What causes Forbidden errors?

---

## What Is the Default Service Account?

### Short Answer

Every namespace automatically receives a default Service Account.

### Detailed Explanation

Pods use it if no custom Service Account is specified.

### Production Example

A Pod without serviceAccountName automatically uses default.

### Follow-Up Questions

- Should we use default in production?
- How do you verify it?
- Can it be disabled?

---

## How Have You Used Service Accounts In Production?

### Short Answer

We used Service Accounts to provide application identity and controlled access to ConfigMaps, Secrets, and AWS services.

### Detailed Explanation

Each microservice had its own Service Account and RBAC permissions following least privilege principles.

### Production Example

Catalogue Pods used catalogue-sa to access application configuration securely.

### Follow-Up Questions

- Why not use default Service Account?
- How is access controlled?
- What is IRSA?

---

# Key Takeaways

```text
Service Accounts are identities for Pods.

Service Accounts are non-human accounts.

Every namespace gets a default Service Account.

Pods use Service Accounts through serviceAccountName.

Service Account tokens authenticate Pods.

Service Accounts provide identity, not permissions.

Roles and RoleBindings provide permissions.

Service Accounts are namespace scoped.

Avoid using the default Service Account in production.

Use one Service Account per application whenever possible.
```