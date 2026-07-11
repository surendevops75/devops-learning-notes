# Labels vs Annotations

## Introduction

Both Labels and Annotations are:

```text
Key Value Pairs
```

attached to Kubernetes resources.

---

Example

```yaml
metadata:

  labels:
    app: catalogue

  annotations:
    owner: platform-team
```

---

Because both look similar, many beginners get confused.

---

Question:

```text
When Should We Use Labels?

When Should We Use Annotations?
```

---

This is one of the most common Kubernetes interview questions.

---

# Similarities

Both:

```text
Store Metadata

Are Key Value Pairs

Exist Under Metadata Section

Can Be Added To Most Kubernetes Resources
```

---

Example

```yaml
metadata:

  labels:
    app: catalogue

  annotations:
    owner: devops-team
```

---

# Fundamental Difference

Labels are used for:

```text
Identification

Grouping

Selection

Filtering
```

---

Annotations are used for:

```text
Additional Information

Documentation

External Tool Configuration
```

---

Think of it like:

```text
Labels
    ↓
Used By Kubernetes

Annotations
    ↓
Used By Humans And Tools
```

---

# Labels

Purpose

```text
Identify Resources
```

---

Example

```yaml
labels:
  app: catalogue
  env: prod
```

---

Used by:

```text
Services

Deployments

ReplicaSets

Network Policies
```

---

Question

```text
Which Pods Belong To Catalogue?
```

---

Answer

```yaml
app: catalogue
```

---

# Annotations

Purpose

```text
Store Metadata
```

---

Example

```yaml
annotations:
  owner: platform-team
  build-number: "120"
```

---

Used by:

```text
Humans

Prometheus

ALB Controller

ExternalDNS

Cert Manager
```

---

Question

```text
Who Owns This Application?
```

---

Answer

```yaml
owner: platform-team
```

---

# Selection Capability

Labels support:

```text
Selectors
```

---

Example

```yaml
selector:
  app: catalogue
```

---

Kubernetes finds matching resources.

---

Annotations:

```text
Cannot Be Used
As Selectors
```

---

Example

```yaml
annotations:
  owner: platform-team
```

---

Service cannot select resources using annotations.

---

# Filtering Resources

Labels

```bash
kubectl get pods -l app=catalogue
```

---

Works because labels support filtering.

---

Annotations

```text
Cannot Be Filtered
Using Label Selectors
```

---

# Kubernetes Internal Usage

Labels are heavily used internally.

---

Examples

```text
Services

Deployments

ReplicaSets

StatefulSets
```

---

Without labels:

```text
Many Kubernetes Resources
Cannot Function Properly
```

---

Annotations are optional.

---

Used mainly by:

```text
External Controllers

Monitoring Systems

Humans
```

---

# Resource Identification

Labels

```yaml
labels:
  app: catalogue
```

---

Clearly identifies:

```text
Application Name
```

---

Annotations

```yaml
annotations:
  owner: platform-team
```

---

Provides:

```text
Additional Information
```

---

Not identity.

---

# Character Limits

Labels have stricter rules.

---

Example

```yaml
labels:
  app: catalogue
```

---

Reason

```text
Labels Are Indexed
```

---

Used frequently for lookups.

---

Annotations allow:

```text
Much Larger Values
```

---

Useful for:

```text
Build Data

Documentation

Configuration
```

---

# Special Characters

Labels have restrictions.

---

Allowed

```text
Letters

Numbers

-

_

.
```

---

Annotations are more flexible.

---

Can contain:

```text
URLs

JSON

Long Text

Special Characters
```

---

# Real Production Example

Catalogue Application

---

Labels

```yaml
labels:
  app: catalogue
  env: prod
  tier: backend
```

---

Annotations

```yaml
annotations:
  owner: platform-team
  build-number: "120"
  git-commit: abc123
```

---

Purpose

Labels

```text
Resource Selection
```

---

Annotations

```text
Operational Metadata
```

---

# Service Example

Pod

```yaml
labels:
  app: catalogue
```

---

Service

```yaml
selector:
  app: catalogue
```

---

Works.

---

Using Annotation

```yaml
annotations:
  app: catalogue
```

---

Service cannot use this.

---

Fails.

---

# Prometheus Example

Bad Practice

```yaml
labels:
  prometheus.io/scrape: "true"
```

---

Good Practice

```yaml
annotations:
  prometheus.io/scrape: "true"
```

---

Reason

```text
Prometheus Reads Annotations
```

---

# AWS ALB Example

Bad Practice

```yaml
labels:
  alb.ingress.kubernetes.io/scheme: internet-facing
```

---

Good Practice

```yaml
annotations:
  alb.ingress.kubernetes.io/scheme: internet-facing
```

---

Reason

```text
AWS Load Balancer Controller
Reads Annotations
```

---

# Build Information Example

Bad Practice

```yaml
labels:
  build-number: "120"
```

---

Good Practice

```yaml
annotations:
  build-number: "120"
```

---

Reason

```text
Build Number Is Metadata
```

---

# Decision Flow

Question:

```text
Do I Need Resource Selection?
```

---

Yes

```text
Use Labels
```

---

No

```text
Use Annotations
```

---

Question:

```text
Will A Service Or Deployment
Use This Value?
```

---

Yes

```text
Use Labels
```

---

No

```text
Use Annotations
```

---

# Common Mistakes

## Mistake 1

Using annotations for Service selectors.

---

Wrong

```yaml
annotations:
  app: catalogue
```

---

Correct

```yaml
labels:
  app: catalogue
```

---

## Mistake 2

Storing build information in labels.

---

Wrong

```yaml
labels:
  build-number: "100"
```

---

Correct

```yaml
annotations:
  build-number: "100"
```

---

## Mistake 3

Using labels for ALB configuration.

---

Wrong

```yaml
labels:
  alb.ingress.kubernetes.io/scheme: internet-facing
```

---

Correct

```yaml
annotations:
  alb.ingress.kubernetes.io/scheme: internet-facing
```

---

# Comparison Table

| Feature | Labels | Annotations |
|----------|----------|----------|
| Purpose | Identification | Metadata |
| Resource Selection | Yes | No |
| Filtering | Yes | No |
| Grouping | Yes | No |
| Service Selectors | Yes | No |
| Deployment Selectors | Yes | No |
| Indexed | Yes | No |
| Large Values | Limited | Yes |
| Build Information | No | Yes |
| Prometheus Configuration | No | Yes |
| ALB Configuration | No | Yes |
| External Tool Integration | Limited | Yes |

---

# Production Scenario

Suppose:

```text
Catalogue Service Not Working
```

---

First Check

```bash
kubectl get pods --show-labels
```

---

Verify

```yaml
app: catalogue
```

---

Because:

```text
Service Depends On Labels
```

---

Suppose:

```text
ALB Not Created
```

---

Check

```bash
kubectl describe ingress ingress-name
```

---

Verify

```yaml
annotations:
  alb.ingress.kubernetes.io/scheme
```

---

Because:

```text
ALB Controller Depends On Annotations
```

---

# Interview Shortcut

Remember:

```text
Labels
    ↓

Select Resources
```

---

```text
Annotations
    ↓

Store Information
```

---

# Common Interview Questions

## What is the Difference Between Labels and Annotations?

### Short Answer

Labels are used for identification and selection, while annotations are used for metadata and external integrations.

### Detailed Explanation

Labels help Kubernetes locate and manage resources. Annotations store additional information used by humans and external systems.

### Production Example

Services use labels to find Pods, while ALB Controller uses annotations to configure load balancers.

### Follow-Up Questions

- Can annotations be used as selectors?
- Why are labels indexed?

---

## Can Services Use Annotations?

### Short Answer

No.

### Detailed Explanation

Services use label selectors to identify matching Pods.

### Production Example

Service selecting Pods using app=catalogue label.

### Follow-Up Questions

- What happens if labels don't match?
- Can Deployments use annotations?

---

## When Should You Use Annotations?

### Short Answer

When storing metadata or configuring external tools.

### Detailed Explanation

Annotations are commonly used for Prometheus, ALB Controller, ExternalDNS, and Cert Manager.

### Production Example

Creating an internet-facing ALB using annotations.

### Follow-Up Questions

- Which AWS services use annotations?
- Which monitoring tools use annotations?

---

# Key Takeaways

```text
Labels and Annotations are both key-value pairs.

Labels are used for identification and selection.

Annotations are used for metadata and integrations.

Services and Deployments depend on labels.

Prometheus depends on annotations.

AWS Load Balancer Controller depends on annotations.

Labels are indexed and optimized for lookups.

Annotations support larger values and richer metadata.

Use Labels for resource selection.

Use Annotations for additional information.
```