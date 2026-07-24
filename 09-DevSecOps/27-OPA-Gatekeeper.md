# OPA Gatekeeper

## Introduction

OPA (Open Policy Agent) Gatekeeper is a Kubernetes-native policy enforcement solution that uses the Open Policy Agent (OPA) to validate Kubernetes resources before they are admitted into the cluster.

It allows organizations to enforce security, compliance, and governance policies consistently across Kubernetes environments.

In DevSecOps, OPA Gatekeeper is widely used to prevent insecure workloads from reaching production.

---

## Why Do We Need OPA Gatekeeper?

Developers can unintentionally deploy workloads that violate security standards.

Without OPA Gatekeeper:

- Privileged containers may run
- Root containers can be deployed
- Resource limits may be missing
- Images from untrusted registries may be used
- Compliance violations reach production

OPA Gatekeeper automatically blocks deployments that violate organizational policies.

---

## What is OPA Gatekeeper?

OPA Gatekeeper is an Admission Controller built on Open Policy Agent.

It validates Kubernetes resources against policies written in **Rego**, OPA's policy language.

It can enforce policies for:

- Pods
- Deployments
- Services
- Ingresses
- Namespaces
- Persistent Volumes
- Custom Resources

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

OPA Gatekeeper

↓

Evaluate Rego Policy

↓

Allow or Reject

↓

Store in etcd

↓

Deploy Resource
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

OPA Gatekeeper

↓

Constraint

↓

ConstraintTemplate

↓

Rego Policy

↓

etcd

↓

Worker Node
```

---

## Workflow

```text
Write Kubernetes Manifest

↓

Commit to Git

↓

CI/CD Pipeline

↓

Deploy

↓

OPA Gatekeeper

↓

Policy Validation

↓

Approved

↓

Production
```

---

# OPA Gatekeeper Components

## Open Policy Agent (OPA)

OPA is a general-purpose policy engine that evaluates requests using Rego policies.

---

## Gatekeeper

Gatekeeper integrates OPA with Kubernetes Admission Controllers.

It intercepts resource creation requests before they reach the cluster.

---

## ConstraintTemplate

Defines reusable policy logic.

Contains:

- Rego Policy
- Parameters
- Validation Schema

```text
ConstraintTemplate

↓

Reusable Policy

↓

Multiple Constraints
```

---

## Constraint

Applies a ConstraintTemplate to specific Kubernetes resources.

Example

```text
Constraint

↓

Apply Policy

↓

Pods

↓

Deployments
```

---

## Rego Policy

Rego is OPA's declarative policy language.

Example policy objectives:

- Block root containers
- Require labels
- Restrict namespaces
- Enforce image registries

---

# Request Validation Flow

```text
Deployment Request

↓

API Server

↓

OPA Gatekeeper

↓

Load Constraint

↓

Evaluate Rego Policy

↓

Compliant?

↓

Yes

↓

Deploy

OR

No

↓

Reject
```

---

# Common Policy Examples

## Block Root Containers

```text
Container

↓

runAsUser = 0

↓

OPA Policy

↓

Rejected
```

---

## Require Resource Limits

```text
Pod

↓

CPU Limit Missing

↓

Rejected
```

---

## Allow Trusted Registries Only

```text
docker.io/random/image

↓

OPA Policy

↓

Rejected
```

---

## Require Labels

```text
Deployment

↓

Owner Label Missing

↓

Rejected
```

---

## Restrict Privileged Containers

```text
Privileged = true

↓

OPA Policy

↓

Rejected
```

---

# Example Enforcement Architecture

```text
Developer

↓

GitHub

↓

GitHub Actions

↓

Deploy Manifest

↓

OPA Gatekeeper

↓

Validate Policies

↓

Approved

↓

Amazon EKS
```

---

# Advantages of OPA Gatekeeper

- Kubernetes-native
- Uses Admission Controllers
- Centralized policy management
- GitOps-friendly
- Version-controlled policies
- Supports auditing
- Highly scalable
- Reusable policies through ConstraintTemplates

---

# Comparison

| Feature | OPA Gatekeeper | Kyverno |
|----------|----------------|----------|
| Policy Language | Rego | YAML |
| Admission Control | Yes | Yes |
| Mutation Support | Limited | Native |
| Kubernetes Native | Yes | Yes |
| Learning Curve | Higher | Lower |

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

Push Registry

↓

Deploy Kubernetes

↓

OPA Gatekeeper

↓

Policy Validation

↓

Production
```

OPA Gatekeeper acts as the final policy enforcement layer before deployment.

---

## Production Workflow

```text
Developer

↓

Deploy Application

↓

API Server

↓

OPA Gatekeeper

↓

Evaluate Rego Policy

↓

Approved

↓

Pod Running
```

---

## Production Example

A developer deploys a Pod running as the root user.

```text
kubectl apply

↓

OPA Gatekeeper

↓

runAsUser = 0

↓

Policy Violation

↓

Deployment Rejected

↓

Developer Updates Manifest

↓

Deployment Approved
```

This prevents insecure containers from entering the cluster.

---

## Best Practices

- Store Rego policies in Git
- Create reusable ConstraintTemplates
- Test policies in staging
- Enable audit mode
- Restrict privileged containers
- Require resource requests and limits
- Enforce trusted image registries
- Require mandatory labels
- Version control policy changes
- Continuously review policy effectiveness

---

## Common Mistakes

- Writing overly complex Rego policies
- Applying policies directly to production
- Ignoring audit results
- Using cluster-admin for testing
- Forgetting to version policies
- Not documenting policy intent
- Blocking Kubernetes system namespaces
- Creating duplicate constraints

---

# Common Troubleshooting

## Issue 1

### Deployment Rejected Unexpectedly

**Cause**

A Rego policy was violated.

**Resolution**

```text
Review Error

↓

Identify Constraint

↓

Fix Manifest

↓

Deploy Again
```

---

## Issue 2

### Constraint Not Applied

**Cause**

ConstraintTemplate is missing or incorrectly configured.

**Resolution**

```text
Verify Template

↓

Apply ConstraintTemplate

↓

Apply Constraint

↓

Retry
```

---

## Issue 3

### Rego Syntax Error

**Cause**

Invalid Rego policy syntax.

**Resolution**

```text
Review Policy

↓

Validate Syntax

↓

Apply Updated Policy
```

---

## Issue 4

### OPA Gatekeeper Pods Not Running

**Cause**

Controller deployment failed.

**Resolution**

```text
Check Pods

↓

Review Logs

↓

Restart Controller

↓

Verify Status
```

---

## Issue 5

### Audit Reports Missing Violations

**Cause**

Audit functionality is disabled.

**Resolution**

```text
Enable Audit

↓

Restart Gatekeeper

↓

Generate Report

↓

Review Violations
```

---

# Production Interview Questions

## Question 1

### What is OPA Gatekeeper?

OPA Gatekeeper is a Kubernetes Admission Controller that uses Open Policy Agent to enforce security and compliance policies before resources are created.

---

## Question 2

### What is Rego?

Rego is the declarative policy language used by Open Policy Agent to define policy rules and validation logic.

---

## Question 3

### What is the difference between a ConstraintTemplate and a Constraint?

A ConstraintTemplate defines reusable policy logic, while a Constraint applies that policy to specific Kubernetes resources.

---

## Question 4

### Where does OPA Gatekeeper operate in Kubernetes?

It operates as a Validating Admission Controller, intercepting requests after authentication and authorization but before resources are stored in etcd.

---

## Question 5

### What types of policies can OPA Gatekeeper enforce?

It can enforce policies such as blocking privileged containers, requiring resource limits, enforcing labels, restricting image registries, and validating security contexts.

---

## Question 6

### How does OPA Gatekeeper integrate with CI/CD?

CI/CD pipelines deploy manifests to Kubernetes, where Gatekeeper validates them before allowing deployment into the cluster.

---

## Question 7

### What is audit mode in OPA Gatekeeper?

Audit mode continuously scans existing Kubernetes resources and reports policy violations without blocking running workloads.

---

## Question 8

### What is the difference between OPA Gatekeeper and Kyverno?

OPA Gatekeeper uses the Rego language and focuses on policy evaluation, while Kyverno uses native Kubernetes YAML policies and includes built-in mutation capabilities.

---

## Question 9

### Can OPA Gatekeeper enforce image registry restrictions?

Yes. Policies can restrict deployments to approved container registries such as Amazon ECR, Azure ACR, or private Harbor registries.

---

## Question 10

### Why is OPA Gatekeeper important in DevSecOps?

OPA Gatekeeper automates Kubernetes policy enforcement, prevents insecure deployments, supports compliance, and strengthens cloud-native security.

---

# Key Takeaways

- OPA Gatekeeper is a Kubernetes policy enforcement solution built on Open Policy Agent.
- It validates Kubernetes resources before they are admitted into the cluster.
- Policies are written in Rego and managed as code.
- ConstraintTemplates define reusable policy logic, while Constraints apply policies to resources.
- OPA Gatekeeper is a key component of production-grade Kubernetes governance and DevSecOps.