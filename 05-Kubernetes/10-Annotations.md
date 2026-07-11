# Kubernetes Annotations

## Introduction

Kubernetes resources often need additional information beyond identification and selection.

Examples:

```text
Build Information

Owner Details

Monitoring Configuration

Load Balancer Configuration

DNS Configuration
```

---

Question:

```text
Where Should We Store
Additional Metadata?
```

---

Answer:

```text
Annotations
```

---

Annotations are one of the most widely used features in production Kubernetes environments.

Especially in:

```text
AWS EKS

Prometheus

Ingress Controllers

ExternalDNS

Cert Manager
```

---

# What are Annotations?

Annotations are:

```text
Key Value Pairs
```

attached to Kubernetes resources.

---

Example

```yaml
annotations:
  owner: platform-team
```

---

Annotations provide:

```text
Metadata

Documentation

Configuration

Integration Information
```

---

Think of annotations as:

```text
Notes Attached To Kubernetes Resources
```

---

# Why Do We Need Annotations?

Suppose we have:

```text
Catalogue Service
```

---

Need to store:

```text
Owner Team

Build Number

Git Commit

Monitoring Configuration
```

---

Labels are not suitable for this.

---

Annotations are designed for:

```text
Additional Information
```

---

# Annotation Structure

Format

```yaml
annotations:
  key: value
```

---

Example

```yaml
annotations:
  owner: devops-team
```

---

Multiple Annotations

```yaml
annotations:
  owner: devops-team
  build-number: "120"
  git-commit: abc123
```

---

# Basic Annotation Example

Pod

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: catalogue

  annotations:
    owner: platform-team
    build-number: "100"

spec:
  containers:
  - name: catalogue
    image: nginx
```

---

# Viewing Annotations

Describe Resource

```bash
kubectl describe pod catalogue
```

---

Output

```text
Annotations:
  owner: platform-team
  build-number: 100
```

---

# Common Annotation Use Cases

## Ownership Information

```yaml
annotations:
  owner: platform-team
```

---

## Documentation

```yaml
annotations:
  description: Catalogue Service
```

---

## Build Tracking

```yaml
annotations:
  build-number: "245"
```

---

## Git Commit Tracking

```yaml
annotations:
  git-commit: a12bc34
```

---

## Deployment Information

```yaml
annotations:
  deployed-by: jenkins
```

---

# Production Example

```yaml
annotations:
  owner: platform-team
  deployed-by: argocd
  build-number: "120"
  git-commit: abc123
```

---

Useful for:

```text
Auditing

Troubleshooting

Change Tracking
```

---

# Annotations in AWS EKS

Annotations become extremely important in EKS.

---

Many AWS integrations work using:

```text
Annotations
```

---

Examples

```text
AWS Load Balancer Controller

ExternalDNS

Cert Manager
```

---

# AWS Load Balancer Controller

Suppose we create an Ingress.

---

Question

```text
Should ALB Be Internal?

Or Internet Facing?
```

---

Annotation

```yaml
annotations:
  alb.ingress.kubernetes.io/scheme: internet-facing
```

---

AWS Load Balancer Controller reads:

```text
Annotation
```

and creates:

```text
Internet Facing ALB
```

---

# Internal ALB Example

```yaml
annotations:
  alb.ingress.kubernetes.io/scheme: internal
```

---

Result

```text
Private ALB
```

---

# Target Type Annotation

Example

```yaml
annotations:
  alb.ingress.kubernetes.io/target-type: ip
```

---

Meaning

```text
Register Pods Directly
```

with ALB.

---

Common in EKS.

---

# SSL Certificate Annotation

Example

```yaml
annotations:
  alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:xxxx
```

---

Result

```text
HTTPS Enabled
```

on ALB.

---

# Prometheus Annotations

Prometheus uses annotations for monitoring.

---

Example

```yaml
annotations:
  prometheus.io/scrape: "true"
```

---

Meaning

```text
Monitor This Application
```

---

Specify Port

```yaml
annotations:
  prometheus.io/port: "8080"
```

---

Specify Path

```yaml
annotations:
  prometheus.io/path: /metrics
```

---

Result

```text
Prometheus Collects Metrics
```

---

# ExternalDNS Annotations

ExternalDNS automates Route53 records.

---

Example

```yaml
annotations:
  external-dns.alpha.kubernetes.io/hostname: catalogue.example.com
```

---

Result

```text
DNS Record Created Automatically
```

---

# Cert Manager Annotations

Used for TLS certificate automation.

---

Example

```yaml
annotations:
  cert-manager.io/cluster-issuer: letsencrypt
```

---

Result

```text
Certificate Generated Automatically
```

---

# NGINX Ingress Annotations

Configure NGINX behavior.

---

Example

```yaml
annotations:
  nginx.ingress.kubernetes.io/rewrite-target: /
```

---

Result

```text
Custom URL Rewriting
```

---

# ArgoCD Annotations

Used in GitOps environments.

---

Example

```yaml
annotations:
  argocd.argoproj.io/sync-wave: "1"
```

---

Used to control:

```text
Deployment Order
```

---

# Custom Annotations

Organizations often create custom annotations.

---

Example

```yaml
annotations:
  application-owner: payments-team
  support-email: support@company.com
```

---

Useful for:

```text
Governance

Auditing

Operations
```

---

# Annotation Best Practices

## Use Meaningful Names

Good

```yaml
owner: platform-team
```

---

Bad

```yaml
x: abc
```

---

## Store Metadata Only

Good

```yaml
build-number: "120"
```

---

Bad

```yaml
app: catalogue
```

---

Use labels for identification.

---

## Use Standard Vendor Keys

Good

```yaml
prometheus.io/scrape
```

---

```yaml
alb.ingress.kubernetes.io/scheme
```

---

Follow official documentation.

---

# Troubleshooting Annotations

View Resource

```bash
kubectl describe ingress ingress-name
```

---

Check

```text
Annotations Section
```

---

Common Problem

```text
ALB Not Created
```

---

Cause

```text
Wrong Annotation
```

---

Example

Wrong

```yaml
alb.ingress.kubernetes.io/scheem: internet-facing
```

---

Correct

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
```

---

# Production Troubleshooting Example

Issue

```text
Prometheus Not Scraping Metrics
```

---

Check

```bash
kubectl describe pod app-pod
```

---

Verify

```yaml
annotations:
  prometheus.io/scrape: "true"
```

exists.

---

If missing:

```text
Prometheus Ignores Pod
```

---

# Real EKS Example

Catalogue Application

```yaml
annotations:
  owner: platform-team

  prometheus.io/scrape: "true"

  prometheus.io/port: "8080"

  build-number: "120"

  git-commit: abc123
```

---

Benefits

```text
Monitoring

Documentation

Auditing

Automation
```

---

# Common Interview Questions

## What are Annotations?

### Short Answer

Annotations are key-value pairs used to store metadata about Kubernetes resources.

### Detailed Explanation

Annotations store information used by humans, tools, and external controllers but are not used for resource selection.

### Production Example

Prometheus and AWS Load Balancer Controller annotations.

### Follow-Up Questions

- Can annotations be used as selectors?
- Why not use labels?

---

## Why Are Annotations Important In EKS?

### Short Answer

Many AWS integrations depend on annotations.

### Detailed Explanation

AWS Load Balancer Controller, ExternalDNS, and Cert Manager use annotations to configure infrastructure behavior.

### Production Example

Creating internet-facing ALBs using annotations.

### Follow-Up Questions

- Which annotation creates an ALB?
- Which annotation enables HTTPS?

---

## How Do You View Annotations?

### Short Answer

Using kubectl describe.

### Example

```bash
kubectl describe pod pod-name
```

### Follow-Up Questions

- Where are annotations stored?
- Can annotations be updated?

---

# Key Takeaways

```text
Annotations are key-value pairs used for metadata.

Annotations are not used for resource selection.

Annotations are heavily used by external tools.

Prometheus uses annotations for monitoring.

AWS Load Balancer Controller uses annotations for ALB configuration.

ExternalDNS uses annotations for Route53 management.

Cert Manager uses annotations for certificate automation.

Annotations are critical in production Kubernetes environments.

Annotations should store metadata, not resource identity.

Understanding annotations is essential for EKS administration.
```