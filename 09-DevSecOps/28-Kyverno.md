# Kyverno

## Introduction

Kyverno is a Kubernetes-native policy engine that allows you to define, validate, mutate, and generate Kubernetes resources using YAML instead of a programming language.

Unlike OPA Gatekeeper, Kyverno does not require learning Rego. It uses familiar Kubernetes YAML syntax, making it easier for Kubernetes administrators and DevOps engineers to implement security and governance policies.

In DevSecOps, Kyverno helps enforce security, compliance, and operational standards before workloads are deployed.

---

## Why Do We Need Kyverno?

Managing Kubernetes clusters manually becomes difficult as the number of applications and teams grows.

Without Kyverno:

- Pods may run as root
- Resource limits may be missing
- Security labels become inconsistent
- Privileged containers may be deployed
- Images from untrusted registries may be used
- Compliance requirements become difficult to enforce

Kyverno automates these checks using Kubernetes-native policies.

---

## What is Kyverno?

Kyverno is a Kubernetes Admission Controller that validates, mutates, generates, and verifies Kubernetes resources.

Kyverno policies are written entirely in YAML and can enforce:

- Security Policies
- Compliance Policies
- Best Practices
- Resource Standards
- Image Verification
- Configuration Consistency

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

Kyverno

↓

Validate / Mutate / Generate

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

Git Repository

↓

CI/CD Pipeline

↓

Kubernetes API Server

↓

Kyverno

↓

ClusterPolicy

↓

Admission Controller

↓

etcd

↓

Worker Node

↓

Running Pods
```

---

## Workflow

```text
Developer Writes Manifest

↓

Commit to Git

↓

CI/CD Pipeline

↓

Deploy Manifest

↓

Kyverno Policy Evaluation

↓

Approved

↓

Production
```

---

# Kyverno Policy Types

## Validate Policy

Checks whether a resource complies with organizational rules.

Example

```text
Deployment

↓

Validate Policy

↓

CPU Limits Missing

↓

Rejected
```

---

## Mutate Policy

Automatically modifies resources before they are created.

Example

```text
Deployment

↓

Mutate Policy

↓

Add Labels

↓

Deploy
```

---

## Generate Policy

Automatically creates additional Kubernetes resources.

Example

```text
Namespace Created

↓

Generate Policy

↓

NetworkPolicy

↓

ResourceQuota

↓

LimitRange
```

---

## VerifyImages Policy

Ensures only trusted and signed container images are deployed.

Example

```text
Container Image

↓

Verify Cosign Signature

↓

Valid

↓

Deploy
```

---

# Kyverno Architecture Flow

```text
Deployment Request

↓

API Server

↓

Kyverno

↓

Validate

↓

Mutate

↓

Verify Image

↓

Generate Resources

↓

Store Resource

↓

Pod Running
```

---

# Common Kyverno Policies

## Require Labels

```text
Deployment

↓

Missing app Label

↓

Rejected
```

---

## Block Root Containers

```text
runAsUser = 0

↓

Rejected
```

---

## Require Resource Limits

```text
Pod

↓

Memory Limit Missing

↓

Rejected
```

---

## Allow Trusted Registries Only

```text
docker.io/random/image

↓

Rejected

↓

Amazon ECR

↓

Allowed
```

---

## Verify Signed Images

```text
Container Image

↓

Cosign Verify

↓

Valid

↓

Deploy
```

---

## Generate Default Network Policy

```text
Namespace

↓

Kyverno

↓

Generate NetworkPolicy

↓

Namespace Protected
```

---

# Common Kyverno Resources

| Resource | Purpose |
|----------|---------|
| ClusterPolicy | Cluster-wide policy |
| Policy | Namespace-specific policy |
| Admission Controller | Intercepts API requests |
| Background Scan | Audits existing resources |
| VerifyImages | Verifies signed images |

---

# OPA Gatekeeper vs Kyverno

| Feature | OPA Gatekeeper | Kyverno |
|----------|----------------|----------|
| Policy Language | Rego | YAML |
| Learning Curve | Higher | Lower |
| Mutation | Limited | Native |
| Resource Generation | No | Yes |
| Image Verification | External | Built-in |
| Kubernetes Native | Yes | Yes |

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

Build

↓

Container Scan

↓

Cosign Sign

↓

Push Registry

↓

Deploy Kubernetes

↓

Kyverno Validation

↓

Production
```

Kyverno ensures only compliant resources enter the Kubernetes cluster.

---

## Production Workflow

```text
Developer

↓

Deploy Manifest

↓

Kyverno

↓

Validate Policy

↓

Verify Image Signature

↓

Generate Resources

↓

Deploy Application
```

---

## Production Example

A developer deploys a Pod without resource limits and using an unsigned image.

```text
Deployment

↓

Kyverno

↓

Resource Limits Missing

↓

Unsigned Image

↓

Deployment Rejected

↓

Developer Fixes Manifest

↓

Cosign Signs Image

↓

Deployment Approved
```

Kyverno prevents insecure workloads from entering production.

---

## Best Practices

- Store Kyverno policies in Git
- Use ClusterPolicies for organization-wide rules
- Enable background scanning
- Verify signed container images
- Require resource requests and limits
- Enforce non-root containers
- Generate default security resources automatically
- Test policies in non-production clusters
- Review policy violations regularly
- Version control all policy changes

---

## Common Mistakes

- Disabling policy enforcement
- Creating conflicting policies
- Not testing policies before production
- Ignoring audit reports
- Allowing unsigned images
- Using overly broad exceptions
- Forgetting background scans
- Not documenting policy intent

---

# Common Troubleshooting

## Issue 1

### Deployment Rejected by Kyverno

**Cause**

The deployment violates one or more Kyverno policies.

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

### Mutation Not Applied

**Cause**

The Mutate policy is incorrectly configured.

**Resolution**

```text
Review Policy

↓

Validate YAML

↓

Apply Updated Policy

↓

Retry Deployment
```

---

## Issue 3

### Image Verification Failed

**Cause**

The container image is unsigned or has an invalid Cosign signature.

**Resolution**

```text
Sign Image

↓

Push Registry

↓

Verify Signature

↓

Deploy Again
```

---

## Issue 4

### Generated Resources Missing

**Cause**

Generate policy failed or lacks permissions.

**Resolution**

```text
Review Generate Policy

↓

Check RBAC

↓

Apply Policy

↓

Retry
```

---

## Issue 5

### Background Scan Reports No Violations

**Cause**

Background scanning is disabled.

**Resolution**

```text
Enable Background Scan

↓

Restart Kyverno

↓

Run Scan

↓

Review Results
```

---

# Production Interview Questions

## Question 1

### What is Kyverno?

Kyverno is a Kubernetes-native policy engine that validates, mutates, generates, and verifies Kubernetes resources using YAML-based policies.

---

## Question 2

### Why is Kyverno preferred by many Kubernetes administrators?

Kyverno uses native Kubernetes YAML instead of Rego, making it easier to learn and manage for teams already familiar with Kubernetes manifests.

---

## Question 3

### What are the four main types of Kyverno policies?

Validate, Mutate, Generate, and VerifyImages.

---

## Question 4

### What is the purpose of the VerifyImages policy?

It verifies the signatures of container images (for example, those signed with Cosign) before allowing them to run in the cluster.

---

## Question 5

### What is the difference between ClusterPolicy and Policy?

A **ClusterPolicy** applies to the entire Kubernetes cluster, while a **Policy** applies only within a specific namespace.

---

## Question 6

### How does Kyverno support DevSecOps?

Kyverno automates policy enforcement, validates Kubernetes manifests, verifies image signatures, and ensures security and compliance before deployment.

---

## Question 7

### Can Kyverno automatically create Kubernetes resources?

Yes. Using **Generate** policies, Kyverno can automatically create resources such as NetworkPolicies, ResourceQuotas, and LimitRanges.

---

## Question 8

### How is Kyverno integrated into a CI/CD pipeline?

After security scans and image signing, manifests are deployed to Kubernetes, where Kyverno validates policies and verifies image signatures before admitting resources.

---

## Question 9

### What is background scanning in Kyverno?

Background scanning periodically audits existing Kubernetes resources against configured policies to identify compliance violations.

---

## Question 10

### When would you choose Kyverno over OPA Gatekeeper?

Kyverno is a good choice when teams prefer YAML-based policies, need built-in mutation and resource generation, or want native image signature verification without writing Rego policies.

---

# Key Takeaways

- Kyverno is a Kubernetes-native policy engine that uses YAML-based policies.
- It supports validation, mutation, generation, and image verification.
- Kyverno integrates with Admission Controllers to enforce security before deployment.
- It works seamlessly with Cosign for container image signature verification.
- Kyverno simplifies Kubernetes governance and is an essential tool for implementing Policy as Code in DevSecOps.