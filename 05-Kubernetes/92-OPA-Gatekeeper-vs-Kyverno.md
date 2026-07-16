# OPA Gatekeeper vs Kyverno

## Introduction

As Kubernetes adoption grew, organizations faced a major challenge:

```text
How To Enforce Organizational Policies?
```

---

Examples

```text
Every Pod Must Have CPU Limits

Every Namespace Must Have Labels

Privileged Containers Not Allowed

Only Approved Container Registries Allowed
```

---

Without Policy Enforcement

```text
Every Team Follows Different Standards
```

---

Result

```text
Security Risks

Compliance Issues

Operational Problems
```

---

Need:

```text
Policy As Code
```

---

This led to tools like:

```text
OPA Gatekeeper

Kyverno
```

---

# Why OPA Gatekeeper Was Created?

Kubernetes RBAC Controls

```text
Who Can Do Something
```

---

But RBAC Cannot Control

```text
What Can Be Created
```

---

Example

Developer Has Permission

```text
Create Deployments
```

---

RBAC Allows It.

---

But Company Policy Says

```text
All Deployments Must Have Resource Limits
```

---

RBAC Cannot Enforce This.

---

OPA Solves This Problem.

---

# What Is OPA?

OPA stands for:

```text
Open Policy Agent
```

---

OPA is a general purpose:

```text
Policy Engine
```

---

Not Limited To Kubernetes.

---

Can Be Used For

```text
Kubernetes

Terraform

APIs

CI/CD

Cloud Security
```

---

# What Is Gatekeeper?

Gatekeeper is a Kubernetes implementation of:

```text
OPA
```

---

It integrates with Kubernetes using:

```text
Admission Controllers
```

---

Flow

```text
kubectl apply

↓

API Server

↓

Gatekeeper

↓

Allow / Reject
```

---

# OPA Architecture

```text
User

↓

API Server

↓

Gatekeeper

↓

OPA Policy

↓

Allow / Reject
```

---

# Example Requirement

Company Policy

```text
Every Deployment

Must Have CPU Limits
```

---

Developer Creates

```yaml
resources: {}
```

---

Gatekeeper

```text
Rejects Deployment
```

---

# OPA Uses Rego

Policies Written In

```text
Rego Language
```

---

Example

```rego
package kubernetes

deny[msg] {

  input.review.kind.kind == "Pod"

  not input.review.object.spec.containers[_].resources.limits

  msg := "CPU limits required"
}
```

---

# Problem With Rego

Many Kubernetes Engineers Know:

```text
YAML

Helm

Terraform
```

---

But Not

```text
Rego
```

---

Learning Curve Exists.

---

# Why Kyverno Was Created?

Many Teams Wanted

```text
Policy As Code
```

---

Without Learning

```text
Rego
```

---

Kyverno Solves This.

---

Policies Written In

```yaml
YAML
```

---

Much Easier For Kubernetes Teams.

---

# What Is Kyverno?

Kyverno is a Kubernetes-native policy engine.

---

Uses

```text
YAML Policies
```

instead of:

```text
Rego
```

---

Designed Specifically For Kubernetes.

---

# Kyverno Architecture

```text
User

↓

API Server

↓

Kyverno

↓

Allow / Reject
```

---

Very Similar To Gatekeeper.

---

# Kyverno Capabilities

## Validate

Check Rules.

---

Example

```text
Require Labels
```

---

## Mutate

Modify Resources.

---

Example

```text
Add Missing Labels
```

Automatically.

---

## Generate

Create Resources Automatically.

---

Example

```text
Create Network Policy

When Namespace Created
```

---

# Validation Example

Requirement

```text
Every Namespace

Must Have Owner Label
```

---

Kyverno Policy

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy

metadata:
  name: require-owner-label

spec:

  validationFailureAction: Enforce

  rules:

  - name: check-owner

    match:

      resources:

        kinds:

        - Namespace

    validate:

      message: "Owner label required"

      pattern:

        metadata:

          labels:

            owner: "?*"
```

---

# Mutation Example

Developer Creates

```yaml
labels: {}
```

---

Kyverno Automatically Adds

```yaml
environment: production
```

---

No Manual Work Needed.

---

# Generate Example

New Namespace Created.

---

Kyverno Automatically Creates

```text
Network Policy

Resource Quota

Limit Range
```

---

Inside Namespace.

---

# OPA vs Kyverno

## Policy Language

OPA

```text
Rego
```

---

Kyverno

```text
YAML
```

---

## Learning Curve

OPA

```text
Higher
```

---

Kyverno

```text
Lower
```

---

## Kubernetes Native

OPA

```text
Generic Policy Engine
```

---

Kyverno

```text
Built Specifically For Kubernetes
```

---

## Mutation Support

OPA

```text
Limited
```

---

Kyverno

```text
Strong
```

---

## Generate Resources

OPA

```text
No
```

---

Kyverno

```text
Yes
```

---

# Real Production Example

Security Team Requirement

```text
No Containers

From Docker Hub
```

---

Allowed

```text
ECR
```

Only.

---

Developer Creates

```yaml
image: nginx
```

---

Policy Engine

```text
Rejects Deployment
```

---

# Production Example 2

Company Policy

```text
All Pods Need

CPU Limits

Memory Limits
```

---

Without Policy Engine

```text
Developers Forget
```

---

With OPA/Kyverno

```text
Deployment Rejected
```

---

# Production Example 3

Namespace Creation

---

Automatically Create

```text
Resource Quota

Network Policy

Limit Range
```

---

Kyverno Can Do This Automatically.

---

# OPA In Enterprise Environments

Common In

```text
Banks

Government

Large Enterprises
```

---

Reason

```text
Compliance Requirements
```

---

# Kyverno In Platform Teams

Common In

```text
Modern Kubernetes Platforms
```

---

Reason

```text
Easy To Learn

Easy To Maintain
```

---

# Troubleshooting Commands

Gatekeeper Pods

```bash
kubectl get pods -n gatekeeper-system
```

---

Kyverno Pods

```bash
kubectl get pods -n kyverno
```

---

Policies

```bash
kubectl get constrainttemplates
```

---

Kyverno Policies

```bash
kubectl get clusterpolicy
```

---

Events

```bash
kubectl get events
```

---

# Common Interview Questions

## Why Were OPA Gatekeeper And Kyverno Created?

### Short Answer

OPA Gatekeeper and Kyverno were created to enforce Kubernetes policies and governance rules automatically.

### Detailed Explanation

RBAC controls who can perform actions, but it cannot enforce organizational standards. OPA and Kyverno ensure resources comply with security, compliance, and operational requirements before entering the cluster.

### Production Example

```text
Deployment

↓

No Resource Limits

↓

Policy Engine

↓

Rejected
```

### Follow-Up Questions

- Why is RBAC insufficient?
- Where do policies run?
- How do they integrate with Kubernetes?

---

## Difference Between OPA Gatekeeper And Kyverno?

### Short Answer

OPA uses Rego policies while Kyverno uses YAML policies.

### Detailed Explanation

OPA is a general-purpose policy engine with powerful capabilities. Kyverno is Kubernetes-native and easier for Kubernetes teams because policies are written in YAML.

### Production Example

```text
OPA

↓

Rego Policy
```

```text
Kyverno

↓

YAML Policy
```

### Follow-Up Questions

- Which one is easier to learn?
- Which one supports mutation?
- Which one supports resource generation?

---

## What Is Policy As Code?

### Short Answer

Policy As Code means defining governance and compliance requirements using version-controlled code.

### Detailed Explanation

Instead of manual reviews, policies are automated and enforced consistently across all environments.

### Production Example

```text
Require Labels

↓

Policy

↓

Automatic Enforcement
```

### Follow-Up Questions

- Why use Policy As Code?
- How does it improve security?
- How does GitOps use Policy As Code?

---

## When Would You Choose Kyverno Over OPA?

### Short Answer

Choose Kyverno when Kubernetes-native YAML policies are preferred and teams want simpler policy management.

### Detailed Explanation

Kyverno is easier for Kubernetes administrators because it uses familiar YAML syntax and supports validation, mutation, and generation of resources.

### Production Example

```text
Namespace Created

↓

Kyverno

↓

Network Policy Generated
```

### Follow-Up Questions

- What are Kyverno advantages?
- Can Kyverno replace OPA?
- Which is more popular today?

---

# Key Takeaways

```text
OPA Gatekeeper and Kyverno enforce Kubernetes policies.

They integrate through Admission Controllers.

OPA uses Rego language.

Kyverno uses YAML policies.

Kyverno supports validation, mutation, and resource generation.

Policy engines improve security, governance, and compliance.

RBAC controls who can act, while policy engines control what can be created.

Many modern Kubernetes platforms use Kyverno due to its simplicity.

OPA remains popular in highly regulated enterprise environments.
```