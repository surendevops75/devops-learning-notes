# Introduction to Helm

## Introduction

Managing Kubernetes applications using raw YAML files becomes difficult as applications grow.

---

A simple application may contain:

```text
Deployment

Service

ConfigMap

Secret

Ingress

HPA

PVC
```

---

Managing all these files individually can become challenging.

---

Kubernetes provides:

```text
Helm
```

to simplify application deployment and management.

---

# What is Helm?

Helm is:

```text
Package Manager

For Kubernetes
```

---

Just like Linux has:

```text
DNF

YUM

APT
```

---

Kubernetes has:

```text
Helm
```

---

Easy Interview Answer

```text
DNF is package manager for Linux.

Helm is package manager for Kubernetes.
```

---

# Why Helm?

Without Helm

```text
deployment.yaml

service.yaml

configmap.yaml

secret.yaml

ingress.yaml
```

---

Need to manage:

```text
Multiple Files
```

---

With Helm

```text
Single Package
```

manages everything.

---

# Real World Example

Linux

```bash
dnf install nginx -y
```

---

What happens?

```text
Downloads Package

Installs Dependencies

Installs Software
```

---

Similarly

```bash
helm install nginx nginx-chart
```

---

Helm

```text
Creates Deployment

Creates Service

Creates ConfigMap

Creates Secrets
```

automatically.

---

# What Problems Does Helm Solve?

## Problem 1

Managing Large YAML Files

---

Solution

```text
Templates
```

---

## Problem 2

Different Environments

```text
Dev

QA

Production
```

---

Solution

```text
Values Files
```

---

## Problem 3

Version Management

---

Solution

```text
Helm Releases
```

---

## Problem 4

Rollback

---

Solution

```text
Helm Rollback
```

---

# Helm Architecture

```text
User
  │

  ▼

Helm CLI
  │

  ▼

Kubernetes API Server
  │

  ▼

Cluster Resources
```

---

Helm uses:

```text
kubectl Config
```

to connect to cluster.

---

# Helm Components

Helm consists of:

```text
Charts

Repositories

Releases

Values
```

---

# What is a Chart?

A Chart is:

```text
Package

Of Kubernetes Resources
```

---

Contains:

```text
Deployment

Service

ConfigMap

Secret

Ingress
```

---

Think of Chart as:

```text
Application Package
```

---

# What is a Repository?

Repository stores:

```text
Charts
```

---

Similar to:

```text
GitHub

Docker Hub

Artifact Repository
```

---

Examples

```text
Bitnami

Prometheus Community

AWS Charts
```

---

# What is a Release?

When a Chart is installed:

```text
Release Created
```

---

Example

```bash
helm install nginx .
```

---

Release Name

```text
nginx
```

---

Release represents:

```text
Running Application
```

inside cluster.

---

# What are Values?

Values are:

```text
Configuration Inputs
```

for Helm Charts.

---

Example

```text
Image Version

Replica Count

CPU Limits

Memory Limits
```

---

Instead of changing YAML files directly:

```text
Update Values
```

---

# Helm Installation

Install Script

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-4

chmod 700 get_helm.sh

./get_helm.sh
```

---

Verify

```bash
helm version
```

---

Expected Output

```text
Version Information
```

---

# Helm Repository

Add Repository

```bash
helm repo add aws-ebs-csi-driver \
https://kubernetes-sigs.github.io/aws-ebs-csi-driver
```

---

Update Repositories

```bash
helm repo update
```

---

# Why Repositories?

Instead of manually creating:

```text
Deployments

Services

RBAC

Controllers
```

---

Helm can install:

```text
Prebuilt Solutions
```

---

# Example - EBS CSI Driver

Without Helm

```text
Download YAML

Review YAML

Apply YAML
```

---

With Helm

```bash
helm install
```

---

Much simpler.

---

# Installing AWS EBS CSI Driver

```bash
helm upgrade --install aws-ebs-csi-driver \
--namespace kube-system \
aws-ebs-csi-driver/aws-ebs-csi-driver
```

---

Helm automatically creates:

```text
Deployments

DaemonSets

RBAC

Service Accounts
```

required by EBS Driver.

---

# Helm Workflow

```text
Create Chart
       │

       ▼

Store In Repository
       │

       ▼

Install Chart
       │

       ▼

Create Release
       │

       ▼

Deploy Application
```

---

# Helm vs Kubectl

## Kubectl

```text
Apply Individual YAML Files
```

---

Example

```bash
kubectl apply -f deployment.yaml

kubectl apply -f service.yaml
```

---

## Helm

```text
Deploy Entire Application
```

---

Example

```bash
helm install nginx .
```

---

# Common Use Cases

## Monitoring

```text
Prometheus

Grafana
```

---

## Storage

```text
EBS CSI Driver

EFS CSI Driver
```

---

## Ingress

```text
NGINX Ingress

AWS Load Balancer Controller
```

---

## Applications

```text
Jenkins

SonarQube

ArgoCD
```

---

# Production Example

Requirement

```text
Install EBS CSI Driver
```

---

Without Helm

```text
Many YAML Files
```

---

With Helm

```bash
helm install aws-ebs-csi-driver
```

---

Result

```text
Deployment Created

RBAC Created

Service Accounts Created
```

automatically.

---

# Common Problems

## Repository Not Found

Cause

```text
Repository Missing
```

---

Fix

```bash
helm repo add
```

---

## Chart Not Found

Cause

```text
Repository Not Updated
```

---

Fix

```bash
helm repo update
```

---

## Cluster Connection Failure

Cause

```text
Invalid kubeconfig
```

---

Verify

```bash
kubectl get nodes
```

---

# Troubleshooting

Check Helm Version

```bash
helm version
```

---

View Repositories

```bash
helm repo list
```

---

Update Repositories

```bash
helm repo update
```

---

Verify Cluster Access

```bash
kubectl get nodes
```

---

# Common Interview Questions

## What is Helm?

### Short Answer

Helm is the package manager for Kubernetes.

### Detailed Explanation

Helm simplifies deployment and management of Kubernetes applications by packaging resources into reusable charts.

### Production Example

Installing AWS EBS CSI Driver using a Helm chart instead of manually applying multiple YAML files.

### Follow-Up Questions

- What is a Chart?
- What is a Release?
- What is a Repository?

---

## Why Do We Need Helm?

### Short Answer

To simplify Kubernetes application deployment and management.

### Detailed Explanation

Helm reduces YAML duplication, supports templating, versioning, and rollback capabilities.

### Production Example

Managing Prometheus deployment using a Helm chart instead of dozens of YAML manifests.

### Follow-Up Questions

- What problems does Helm solve?
- How does Helm handle environments?
- How does Helm support rollback?

---

## What is a Chart?

### Short Answer

A Chart is a package containing Kubernetes resources.

### Detailed Explanation

Charts bundle Deployments, Services, ConfigMaps, Secrets, and other resources into a reusable package.

### Production Example

The AWS EBS CSI Driver Helm chart contains all resources required for deployment.

### Follow-Up Questions

- What files exist inside a chart?
- Where are charts stored?
- How are charts installed?

---

## What is a Release?

### Short Answer

A Release is a deployed instance of a Helm chart.

### Detailed Explanation

Every Helm installation creates a release that Helm tracks and manages.

### Production Example

Installing nginx chart creates a release named nginx.

### Follow-Up Questions

- Can multiple releases use the same chart?
- How do you view releases?
- How does rollback work?

---

## Helm vs Kubectl?

### Short Answer

Kubectl manages resources individually, while Helm manages applications as packages.

### Detailed Explanation

Helm bundles resources into charts and adds versioning, upgrades, and rollback capabilities.

### Production Example

A DevOps engineer upgrades an application using Helm instead of modifying multiple YAML files manually.

### Follow-Up Questions

- Does Helm replace kubectl?
- Can Helm use existing YAML files?
- Which one supports rollback?

---

# Key Takeaways

```text
Helm is the package manager for Kubernetes.

Helm simplifies Kubernetes application deployment.

Charts are application packages.

Repositories store charts.

Releases represent deployed charts.

Values provide configuration inputs.

Helm supports upgrades and rollbacks.

Helm is widely used in production Kubernetes environments.

EBS CSI Driver, Prometheus, Grafana, Jenkins, and ArgoCD are commonly installed using Helm.
```