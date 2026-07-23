# Admission Controllers

## Introduction

Admission Controllers are Kubernetes components that intercept API requests after authentication and authorization but before objects are stored in etcd.

They validate, modify, or reject Kubernetes resources to enforce security, compliance, and organizational policies automatically.

In DevSecOps, Admission Controllers act as the final security checkpoint before workloads are deployed into the cluster.

---

## Why Do We Need Admission Controllers?

Even if developers follow best practices, insecure Kubernetes manifests can still be deployed.

Without Admission Controllers:

- Containers may run as root
- Privileged Pods can be created
- Images from untrusted registries may be deployed
- Resource limits may be missing
- Security policies become inconsistent

Admission Controllers ensure every deployment complies with security standards.

---

## What are Admission Controllers?

Admission Controllers are plugins that inspect API requests before Kubernetes stores objects.

They can:

- Validate Requests
- Modify Requests
- Reject Requests
- Enforce Organizational Policies
- Improve Cluster Security

They work after authentication and authorization but before persistence.

---

## How It Works

```text
kubectl apply

↓

API Server

↓

Authentication

↓

Authorization

↓

Admission Controller

↓

Validate / Mutate

↓

Accept or Reject

↓

Store in etcd

↓

Pod Created
```

---

## Architecture

```text
Developer

↓

kubectl apply

↓

Kubernetes API Server

↓

Authentication

↓

Authorization

↓

Admission Controller

↓

etcd

↓

Scheduler

↓

Worker Node

↓

Running Pod
```

---

## Workflow

```text
Developer Creates Manifest

↓

kubectl apply

↓

API Server

↓

Admission Controller

↓

Policy Validation

↓

Approved

↓

Resource Created
```

---

# Types of Admission Controllers

## Mutating Admission Controller

Modifies requests before they are stored.

Examples

- Add Labels
- Inject Sidecars
- Set Default Values
- Add Resource Limits

Example

```text
Deployment

↓

Mutating Controller

↓

Inject Istio Sidecar

↓

Store Resource
```

---

## Validating Admission Controller

Validates requests and either accepts or rejects them.

Examples

- Block Root Containers
- Require Resource Limits
- Enforce Labels
- Restrict Privileged Pods

Example

```text
Deployment

↓

Validate Policy

↓

Violation Found

↓

Deployment Rejected
```

---

# Admission Controller Flow

```text
API Request

↓

Authentication

↓

Authorization

↓

Mutating Admission

↓

Validating Admission

↓

Store Object

↓

Scheduler

↓

Pod Running
```

---

# Common Built-in Admission Controllers

| Admission Controller | Purpose |
|----------------------|---------|
| NamespaceLifecycle | Validates namespace operations |
| LimitRanger | Applies default resource limits |
| ResourceQuota | Enforces namespace quotas |
| PodSecurity | Enforces Pod Security Standards |
| DefaultStorageClass | Assigns default storage class |
| MutatingAdmissionWebhook | Calls external mutation webhook |
| ValidatingAdmissionWebhook | Calls external validation webhook |

---

# Common Security Policies

## Block Root Containers

```text
Pod

↓

runAsUser = 0

↓

Admission Controller

↓

Rejected
```

---

## Require Resource Limits

```text
Pod

↓

No CPU Limit

↓

Admission Controller

↓

Rejected
```

---

## Allow Trusted Images Only

```text
Docker Image

↓

Docker Hub

↓

Untrusted

↓

Rejected
```

---

## Require Labels

```text
Deployment

↓

Missing Owner Label

↓

Rejected
```

---

## Block Privileged Containers

```text
Pod

↓

Privileged = true

↓

Rejected
```

---

# Popular Policy Engines

| Tool | Purpose |
|------|---------|
| Kyverno | Kubernetes-native policy engine |
| OPA Gatekeeper | Policy enforcement using Rego |
| Kubewarden | Policy enforcement with WebAssembly |
| Pod Security Admission | Built-in Pod security enforcement |

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

SAST

↓

SCA

↓

IaC Scan

↓

Container Scan

↓

Build Image

↓

Image Signing

↓

Push Registry

↓

Deploy to Kubernetes

↓

Admission Controller

↓

Policy Validation

↓

Production
```

Admission Controllers act as the final security gate before workloads enter the cluster.

---

## Production Workflow

```text
Developer

↓

Deploy Application

↓

API Server

↓

Admission Controller

↓

Validate Policies

↓

Approved

↓

Pod Scheduled

↓

Running Application
```

---

## Production Example

A developer deploys a Pod without CPU and Memory limits.

```text
Deployment

↓

Admission Controller

↓

Resource Limits Missing

↓

Deployment Rejected

↓

Developer Adds Limits

↓

Deployment Approved
```

The workload is prevented from consuming unlimited cluster resources.

---

## Best Practices

- Enable Validating Admission Controllers
- Use Mutating Controllers for standard configurations
- Block privileged containers
- Require resource requests and limits
- Enforce trusted container registries
- Require securityContext
- Validate image signatures
- Enforce mandatory labels
- Audit rejected requests
- Test policies before production rollout

---

## Common Mistakes

- Disabling Admission Controllers
- Creating overly restrictive policies
- Not testing policies in staging
- Allowing images from any registry
- Ignoring audit logs
- Blocking critical system workloads
- Using outdated policy definitions
- Applying inconsistent policies across clusters

---

# Common Troubleshooting

## Issue 1

### Deployment Rejected Unexpectedly

**Cause**

Admission policy validation failed.

**Resolution**

```text
Review Error

↓

Identify Failed Policy

↓

Update Manifest

↓

Deploy Again
```

---

## Issue 2

### Sidecar Not Injected

**Cause**

Mutating Admission Controller is disabled or misconfigured.

**Resolution**

```text
Verify Webhook

↓

Check Namespace Labels

↓

Restart Controller

↓

Deploy Again
```

---

## Issue 3

### Pod Security Policy Violation

**Cause**

Container runs as root or uses privileged mode.

**Resolution**

```text
Update Security Context

↓

Disable Privileged Mode

↓

Deploy Again
```

---

## Issue 4

### Admission Webhook Timeout

**Cause**

Webhook service is unavailable.

**Resolution**

```text
Check Webhook Service

↓

Verify Network

↓

Restart Controller

↓

Retry Deployment
```

---

## Issue 5

### Trusted Image Policy Failure

**Cause**

Image originates from an unapproved registry.

**Resolution**

```text
Push Image

↓

Approved Registry

↓

Update Manifest

↓

Deploy
```

---

# Production Interview Questions

## Question 1

### What is an Admission Controller in Kubernetes?

An Admission Controller is a Kubernetes component that intercepts API requests after authentication and authorization to validate or modify resources before they are stored.

---

## Question 2

### Why are Admission Controllers important?

They enforce security, compliance, and operational policies, preventing insecure workloads from entering the cluster.

---

## Question 3

### What is the difference between Mutating and Validating Admission Controllers?

Mutating Admission Controllers modify incoming requests, while Validating Admission Controllers either approve or reject requests based on defined policies.

---

## Question 4

### At which stage do Admission Controllers execute?

They execute after authentication and authorization but before Kubernetes stores the object in etcd.

---

## Question 5

### What types of security policies can Admission Controllers enforce?

They can require resource limits, block privileged containers, enforce trusted image registries, validate labels, require security contexts, and verify image signatures.

---

## Question 6

### What is the difference between Admission Controllers and RBAC?

RBAC controls **who** can perform actions, while Admission Controllers control **what** resources are allowed into the cluster.

---

## Question 7

### Which tools are commonly used for Kubernetes policy enforcement?

Kyverno, OPA Gatekeeper, Pod Security Admission, and Kubewarden.

---

## Question 8

### Why should Admission Controllers be integrated into DevSecOps?

They automatically enforce security policies, ensuring only compliant workloads are deployed.

---

## Question 9

### Can Admission Controllers modify Kubernetes resources?

Yes. Mutating Admission Controllers can automatically add labels, inject sidecars, set default values, or modify manifests before they are stored.

---

## Question 10

### What happens if an Admission Controller rejects a request?

The Kubernetes API Server rejects the request, the resource is not stored in etcd, and the workload is never deployed.

---

# Key Takeaways

- Admission Controllers are the final security checkpoint before Kubernetes stores resources.
- They execute after authentication and authorization.
- Mutating Controllers modify resources, while Validating Controllers approve or reject them.
- They enforce security, compliance, and organizational policies automatically.
- Admission Controllers are an essential component of production-grade Kubernetes security in a DevSecOps environment.