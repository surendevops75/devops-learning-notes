# Admission Controllers

## Introduction

When a user runs:

```bash
kubectl apply -f deployment.yaml
```

many engineers think:

```text
kubectl

↓

API Server

↓

ETCD
```

---

But that is not the complete flow.

---

Actual Flow

```text
kubectl

↓

Authentication

↓

Authorization

↓

Admission Controllers

↓

ETCD
```

---

Before Kubernetes stores any object, it passes through:

```text
Admission Controllers
```

---

# Why Admission Controllers Were Created?

Suppose a developer deploys:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:

  containers:

  - name: nginx

    image: nginx

    securityContext:

      privileged: true
```

---

Problem

```text
Privileged Container

Can Access Host Resources

Security Risk
```

---

Another Example

Developer Creates

```yaml
resources: {}
```

---

No Limits.

---

Result

```text
One Pod Consumes Entire Node
```

---

Need A Mechanism To

```text
Validate

Modify

Reject
```

requests before they enter the cluster.

---

Admission Controllers solve this problem.

---

# What Are Admission Controllers?

Admission Controllers are plugins that intercept requests after:

```text
Authentication

Authorization
```

but before:

```text
Object Stored In ETCD
```

---

They can:

```text
Validate Requests

Modify Requests

Reject Requests
```

---

# Kubernetes API Request Flow

```text
kubectl apply

↓

API Server

↓

Authentication

↓

Authorization

↓

Admission Controllers

↓

ETCD
```

---

# Example Flow

Developer Creates Pod

---

Request Reaches

```text
API Server
```

---

Admission Controller Checks

```text
Security Policies

Labels

Namespaces

Resource Limits
```

---

Result

```text
Allow

or

Reject
```

---

# Types Of Admission Controllers

There are two categories.

---

## Mutating Admission Controllers

Can

```text
Modify Requests
```

before storing.

---

Example

Developer Creates

```yaml
metadata:

  labels: {}
```

---

Admission Controller Automatically Adds

```yaml
environment: production
```

---

Request Modified.

---

Stored In ETCD.

---

# Validating Admission Controllers

Can

```text
Approve

Reject
```

requests.

---

Cannot Modify.

---

Example

```yaml
privileged: true
```

---

Policy Says

```text
Privileged Containers Not Allowed
```

---

Result

```text
Request Rejected
```

---

# Admission Controller Workflow

```text
Request

↓

Mutating Admission Controller

↓

Request Modified

↓

Validating Admission Controller

↓

Approved

↓

ETCD
```

---

# Built-In Admission Controllers

Kubernetes ships with many admission controllers.

---

# NamespaceLifecycle

Prevents

```text
Objects Created

In Deleted Namespace
```

---

# LimitRanger

Automatically Enforces

```text
CPU Limits

Memory Limits
```

---

Example

```yaml
cpu: 500m

memory: 512Mi
```

---

# ResourceQuota

Controls Resource Usage.

---

Example

Namespace Limited To

```text
20 Pods

10 CPUs

20GB Memory
```

---

Prevent Resource Exhaustion.

---

# DefaultStorageClass

Automatically Assigns

```text
Storage Class
```

if user doesn't specify one.

---

# ServiceAccount

Automatically Assigns

```text
Default Service Account
```

to Pods.

---

# PodSecurity Admission Controller

Enforces

```text
Pod Security Standards
```

---

Example

Reject

```text
Root Containers

Privileged Containers
```

---

# MutatingWebhookConfiguration

Allows External Systems To

```text
Modify Requests
```

---

Example

```text
Istio Sidecar Injection
```

---

Flow

```text
Pod Created

↓

Mutating Webhook

↓

Envoy Sidecar Added

↓

Pod Stored
```

---

# ValidatingWebhookConfiguration

Allows External Systems To

```text
Validate Requests
```

---

Examples

```text
OPA Gatekeeper

Kyverno
```

---

Can Reject Requests.

---

# Real Production Example

Company Policy

```text
Every Deployment

Must Have

CPU Limits

Memory Limits
```

---

Developer Creates

```yaml
resources: {}
```

---

Admission Controller

```text
Rejects Deployment
```

---

Deployment Never Created.

---

# Example: Namespace Enforcement

Company Has

```text
dev

qa

prod
```

Namespaces.

---

Policy

```text
All Deployments

Must Have

environment Label
```

---

Developer Forgets Label.

---

Admission Controller

```text
Rejects Request
```

---

# Example: Security Enforcement

Developer Deploys

```yaml
privileged: true
```

---

Policy

```text
Privileged Containers Forbidden
```

---

Result

```text
Deployment Rejected
```

---

# Admission Controllers In EKS

EKS Uses Many Built-In Admission Controllers.

---

Examples

```text
NamespaceLifecycle

LimitRanger

ResourceQuota

ServiceAccount

PodSecurity
```

---

Additionally

```text
OPA Gatekeeper

Kyverno

Istio
```

can register custom webhooks.

---

# Why OPA And Kyverno Need Admission Controllers?

OPA And Kyverno Do Not Directly Control Kubernetes.

---

Instead

```text
API Server

↓

Admission Webhook

↓

OPA/Kyverno

↓

Allow / Reject
```

---

This is how policy enforcement works.

---

# Production Architecture

```text
kubectl apply

↓

API Server

↓

Authentication

↓

Authorization

↓

Mutating Admission Controller

↓

Validating Admission Controller

↓

ETCD
```

---

# Common Production Problems

## Problem 1

Admission Webhook Down.

---

Result

```text
Deployment Fails
```

---

Reason

```text
API Server

Cannot Reach Webhook
```

---

# Problem 2

Wrong Policy

---

Valid Applications

```text
Rejected
```

---

# Problem 3

Slow Admission Webhooks

---

Result

```text
Slow Deployments
```

---

# Troubleshooting Commands

Admission Webhooks

```bash
kubectl get validatingwebhookconfigurations
```

---

Mutating Webhooks

```bash
kubectl get mutatingwebhookconfigurations
```

---

Describe

```bash
kubectl describe validatingwebhookconfiguration
```

---

Events

```bash
kubectl get events
```

---

API Server Logs

```bash
kubectl logs kube-apiserver
```

(Self Managed Clusters)

---

# Common Interview Questions

## What Are Admission Controllers?

### Short Answer

Admission Controllers are Kubernetes components that intercept API requests before objects are stored in ETCD.

### Detailed Explanation

After authentication and authorization, every Kubernetes request passes through admission controllers. They can validate, modify, or reject requests based on security, compliance, and operational requirements.

### Production Example

```text
kubectl apply

↓

Admission Controller

↓

Allow / Reject

↓

ETCD
```

### Follow-Up Questions

- Where do Admission Controllers run?
- When are they executed?
- Can they modify requests?
- Can they reject requests?

---

## Difference Between Authentication, Authorization And Admission Controllers?

### Short Answer

Authentication verifies identity, authorization checks permissions, and admission controllers validate or modify requests.

### Detailed Explanation

Kubernetes first identifies the user, then checks what actions they are allowed to perform, and finally validates whether the request complies with cluster policies.

### Production Example

```text
User Login

↓

Authentication

↓

Authorization

↓

Admission Controller

↓

ETCD
```

### Follow-Up Questions

- Which happens first?
- Can Admission Controllers replace RBAC?
- Where does OPA fit?

---

## Difference Between Mutating And Validating Admission Controllers?

### Short Answer

Mutating Admission Controllers modify requests, while Validating Admission Controllers approve or reject requests.

### Detailed Explanation

Mutating controllers can automatically add labels, annotations, sidecars, or defaults. Validating controllers ensure requests comply with policies before they are persisted.

### Production Example

```text
Pod Created

↓

Mutating

(Add Sidecar)

↓

Validating

(Check Security)

↓

Stored
```

### Follow-Up Questions

- Which executes first?
- Can validating modify requests?
- How does Istio use mutating webhooks?

---

## How Do OPA Gatekeeper And Kyverno Work?

### Short Answer

OPA Gatekeeper and Kyverno use validating admission webhooks to enforce policies.

### Detailed Explanation

When users create Kubernetes resources, API Server sends the request to OPA or Kyverno through admission webhooks. Based on policy definitions, the request is either approved or rejected.

### Production Example

```text
Deployment

↓

OPA Policy Check

↓

Reject

(No Resource Limits)
```

### Follow-Up Questions

- Is OPA a built-in admission controller?
- Is Kyverno a built-in admission controller?
- Which one is easier to manage?

---

# Key Takeaways

```text
Admission Controllers intercept requests before they reach ETCD.

They run after authentication and authorization.

They can validate, modify, or reject requests.

Mutating Admission Controllers modify requests.

Validating Admission Controllers approve or reject requests.

OPA Gatekeeper and Kyverno depend on admission webhooks.

Admission Controllers are critical for security, governance, and compliance.

Many production Kubernetes platforms use admission controllers extensively.
```