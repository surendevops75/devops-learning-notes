# Kubernetes Labels

## Introduction

As Kubernetes clusters grow, there can be:

```text
Hundreds Of Pods

Hundreds Of Services

Hundreds Of Deployments
```

---

Question:

```text
How Do We Identify Resources?

How Do We Group Resources?

How Do Services Find Pods?
```

---

Answer:

```text
Labels
```

---

Labels are one of the most important concepts in Kubernetes because many resources depend on them.

Examples:

```text
Services

Deployments

ReplicaSets

Network Policies
```

---

# What are Labels?

Labels are key-value pairs attached to Kubernetes resources.

---

Example

```yaml
labels:
  app: catalogue
```

---

Labels help Kubernetes:

```text
Identify Resources

Group Resources

Filter Resources

Select Resources
```

---

Think of Labels as:

```text
Tags For Kubernetes Objects
```

---

# Why Do We Need Labels?

Suppose we have:

```text
Catalogue Pods

User Pods

Cart Pods

Shipping Pods
```

---

Question:

```text
How Does Service Know

Which Pods Belong To Catalogue?
```

---

Answer:

```text
Labels
```

---

Example

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

Service automatically finds matching Pods.

---

# Label Structure

Labels follow:

```yaml
key: value
```

---

Example

```yaml
app: frontend
```

---

Multiple Labels

```yaml
labels:
  app: catalogue
  env: prod
  tier: backend
```

---

# Common Labels

Application

```yaml
app: catalogue
```

---

Environment

```yaml
env: dev
```

---

Tier

```yaml
tier: backend
```

---

Version

```yaml
version: v1
```

---

Team

```yaml
team: platform
```

---

# Pod With Labels

Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: catalogue

  labels:
    app: catalogue
    env: prod

spec:
  containers:
  - name: catalogue
    image: nginx
```

---

Create Pod

```bash
kubectl apply -f pod.yaml
```

---

# Viewing Labels

View Labels

```bash
kubectl get pods --show-labels
```

---

Example Output

```text
NAME         LABELS

catalogue    app=catalogue,env=prod
```

---

# Filtering Resources Using Labels

Suppose:

```text
100 Pods
```

---

Need only:

```text
Catalogue Pods
```

---

Command

```bash
kubectl get pods -l app=catalogue
```

---

Output

```text
catalogue-1

catalogue-2
```

---

# Multiple Label Filtering

Example

```bash
kubectl get pods \
-l app=catalogue,env=prod
```

---

Result

```text
Only Production Catalogue Pods
```

---

# Label Selectors

Labels become powerful through:

```text
Selectors
```

---

Selectors help Kubernetes locate resources.

---

Example

```yaml
selector:
  app: catalogue
```

---

Kubernetes finds:

```text
All Pods

Having

app=catalogue
```

---

# Equality Based Selectors

Example

```bash
kubectl get pods \
-l app=catalogue
```

---

Result

```text
Only Matching Pods
```

---

Another Example

```bash
kubectl get pods \
-l env=prod
```

---

Returns:

```text
Production Pods
```

---

# Set Based Selectors

Example

```bash
kubectl get pods \
-l 'env in (dev,qa)'
```

---

Returns

```text
Dev Pods

QA Pods
```

---

Another Example

```bash
kubectl get pods \
-l 'env notin (prod)'
```

---

Returns

```text
Non Production Pods
```

---

# How Services Use Labels

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

Architecture

```text
Service
     ↓

Label Selector
     ↓

Matching Pods
```

---

Without labels:

```text
Service Cannot Find Pods
```

---

# How Deployments Use Labels

Deployment creates Pods.

---

Deployment

```yaml
selector:
  matchLabels:
    app: catalogue
```

---

Pod

```yaml
labels:
  app: catalogue
```

---

Relationship

```text
Deployment
      ↓

Selector
      ↓

Pods
```

---

# MatchLabels

Most commonly used selector.

---

Example

```yaml
selector:
  matchLabels:
    app: catalogue
```

---

Meaning

```text
Select Pods

Where

app=catalogue
```

---

# Real Production Example

Catalogue Deployment

```yaml
labels:
  app: catalogue
  env: prod
  tier: backend
```

---

User Deployment

```yaml
labels:
  app: user
  env: prod
  tier: backend
```

---

Frontend Deployment

```yaml
labels:
  app: frontend
  env: prod
  tier: frontend
```

---

Benefits

```text
Easy Filtering

Easy Troubleshooting

Easy Service Mapping
```

---

# Label Strategy Best Practices

## Always Use app

Good

```yaml
app: catalogue
```

---

Bad

```yaml
service: catalogue
```

---

## Use Environment Labels

```yaml
env: dev

env: qa

env: prod
```

---

Useful for:

```text
Filtering

Monitoring

Troubleshooting
```

---

## Use Tier Labels

```yaml
tier: frontend

tier: backend

tier: database
```

---

Helps organize workloads.

---

## Keep Labels Consistent

Good

```yaml
app: catalogue
```

everywhere.

---

Bad

```yaml
app: catalogue

application: catalogue

service: catalogue
```

---

# Label Limitations

Labels have restrictions.

---

Key and Value Length:

```text
Limited
```

---

Allowed Characters

```text
Letters

Numbers

-

_

.
```

---

Labels should remain:

```text
Short

Meaningful
```

---

Not intended for:

```text
Large Text

Documentation

Build Information
```

---

# Troubleshooting Labels

View Labels

```bash
kubectl get pods --show-labels
```

---

Describe Pod

```bash
kubectl describe pod catalogue
```

---

Check Service Selector

```bash
kubectl describe svc catalogue
```

---

Common Problem

```text
Service Not Routing Traffic
```

---

Cause

```text
Label Mismatch
```

---

Example

Pod

```yaml
labels:
  app: catalogue
```

---

Service

```yaml
selector:
  app: catalog
```

---

Result

```text
No Matching Pods
```

---

Service Fails
```

---

# Production Troubleshooting Scenario

Issue

```text
Catalogue Service Not Working
```

---

Step 1

```bash
kubectl get pods --show-labels
```

---

Step 2

Verify

```text
app=catalogue
```

exists.

---

Step 3

```bash
kubectl describe svc catalogue
```

---

Step 4

Verify selector matches pod labels.

---

Most service connectivity issues involve:

```text
Wrong Labels

Wrong Selectors
```

---

# Common Interview Questions

## What are Labels?

### Short Answer

Labels are key-value pairs used to identify and group Kubernetes resources.

### Detailed Explanation

Labels help Kubernetes organize resources and allow Services, Deployments, and other objects to locate matching resources.

### Production Example

Service selecting catalogue Pods using app=catalogue.

### Follow-Up Questions

- What are selectors?
- Why are labels important?

---

## Why Are Labels Important?

### Short Answer

Labels enable resource selection and grouping.

### Detailed Explanation

Without labels, Services, Deployments, and ReplicaSets cannot identify the resources they manage.

### Production Example

Frontend service locating frontend pods.

### Follow-Up Questions

- Can Services work without labels?
- How do Deployments use labels?

---

## What is matchLabels?

### Short Answer

matchLabels is a selector used to find resources with specific labels.

### Example

```yaml
selector:
  matchLabels:
    app: catalogue
```

### Follow-Up Questions

- What is the difference between labels and selectors?
- What are set-based selectors?

---

# Key Takeaways

```text
Labels are key-value pairs attached to Kubernetes resources.

Labels are used for identification and grouping.

Services use labels to locate Pods.

Deployments use labels to manage Pods.

Selectors work using labels.

matchLabels is the most common selector type.

Labels should be short and meaningful.

Consistent labeling simplifies operations.

Label mismatches are a common production issue.

Labels are one of the most important concepts in Kubernetes.
```