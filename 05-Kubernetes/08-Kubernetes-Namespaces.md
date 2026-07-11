# Kubernetes Namespaces

## Introduction

As Kubernetes clusters grow, multiple teams and applications share the same cluster.

Example:

```text
Roboshop

Banking Application

HR Application

Monitoring Stack
```

---

Question:

```text
How Do We Isolate Resources
Inside The Same Cluster?
```

---

Answer:

```text
Namespaces
```

---

# What is a Namespace?

A Namespace is a logical partition inside a Kubernetes cluster.

It provides:

```text
Resource Isolation

Resource Organization

Access Control

Environment Separation
```

---

Think of a Namespace as:

```text
A Project Space
```

inside a Kubernetes cluster.

---

Example

```text
Kubernetes Cluster
        |
        |-- roboshop
        |-- monitoring
        |-- banking
        |-- hr
```

---

Each namespace contains its own:

```text
Pods

Services

Deployments

Secrets

ConfigMaps
```

---

# Why Do We Need Namespaces?

Suppose:

```text
Team A

Team B

Team C
```

share the same cluster.

---

Without namespaces:

```text
All Resources Mixed Together
```

---

Problems

```text
Naming Conflicts

Security Issues

Difficult Management

Poor Organization
```

---

Namespaces solve these issues.

---

# Real Production Example

Single Cluster

```text
EKS Cluster
```

---

Namespaces

```text
roboshop-dev

roboshop-qa

roboshop-uat

roboshop-prod
```

---

Benefits

```text
Environment Isolation

Easy Management

Cost Optimization
```

---

# Default Namespaces

View Namespaces

```bash
kubectl get namespaces
```

---

Example Output

```text
default

kube-system

kube-public

kube-node-lease
```

---

# default Namespace

If no namespace specified:

```text
Resources Created Here
```

---

Example

```bash
kubectl get pods
```

actually means:

```bash
kubectl get pods -n default
```

---

# kube-system Namespace

Contains Kubernetes system components.

Examples

```text
CoreDNS

Kube Proxy

AWS Load Balancer Controller

Metrics Server
```

---

View Resources

```bash
kubectl get pods -n kube-system
```

---

Never delete resources here unless you know what you are doing.

---

# kube-public Namespace

Contains publicly readable resources.

---

Usually used for:

```text
Cluster Information
```

---

Rarely used in day-to-day operations.

---

# kube-node-lease Namespace

Stores:

```text
Node Heartbeats
```

---

Used internally by Kubernetes.

---

# Creating Namespace

Create Namespace

```bash
kubectl create namespace roboshop
```

---

Verify

```bash
kubectl get namespaces
```

---

Output

```text
roboshop
```

---

# Using Namespace

View Pods

```bash
kubectl get pods -n roboshop
```

---

View Services

```bash
kubectl get svc -n roboshop
```

---

View Deployments

```bash
kubectl get deployments -n roboshop
```

---

# Create Resource In Namespace

Example

```bash
kubectl apply -f deployment.yaml -n roboshop
```

---

Resource created inside:

```text
roboshop Namespace
```

---

# Namespace YAML

Example

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: roboshop
```

---

Create Namespace

```bash
kubectl apply -f namespace.yaml
```

---

# Set Default Namespace

Current Context

```bash
kubectl config current-context
```

---

Set Namespace

```bash
kubectl config set-context \
--current \
--namespace=roboshop
```

---

Now commands automatically use:

```text
roboshop Namespace
```

---

Example

```bash
kubectl get pods
```

instead of:

```bash
kubectl get pods -n roboshop
```

---

# Namespace Isolation

Suppose:

Namespace A

```text
roboshop
```

---

Contains

```text
catalogue

user

cart
```

---

Namespace B

```text
banking
```

---

Contains

```text
account

payment

transaction
```

---

Resources remain isolated.

---

# Can Namespaces Communicate?

Yes.

---

Example

```text
Namespace A
       ↓

Namespace B
```

---

Communication is allowed unless restricted.

---

Using:

```text
Services

Network Policies
```

---

# Namespace-Level Resources

Resources created inside namespaces.

Examples

```text
Pods

Services

Deployments

Secrets

ConfigMaps

ReplicaSets
```

---

Check

```bash
kubectl api-resources
```

---

Look for

```text
NAMESPACED = true
```

---

# Cluster-Level Resources

Resources shared across cluster.

Examples

```text
Nodes

Namespaces

PersistentVolumes

StorageClasses
```

---

Check

```bash
kubectl api-resources
```

---

Look for

```text
NAMESPACED = false
```

---

# kubectl api-resources

Displays all Kubernetes resource types.

---

Command

```bash
kubectl api-resources
```

---

Example Output

```text
pods

services

deployments

nodes

namespaces
```

---

Useful for:

```text
Learning Resources

Troubleshooting

API Discovery
```

---

# Namespace Best Practices

## Separate Environments

Good

```text
dev

qa

uat

prod
```

---

Bad

```text
Everything In default Namespace
```

---

## Use Meaningful Names

Good

```text
roboshop-prod

roboshop-dev
```

---

Bad

```text
test1

temp
```

---

## Avoid Using default

Production applications should use dedicated namespaces.

---

# Real EKS Example

Cluster

```text
roboshop-eks
```

---

Namespaces

```text
roboshop-dev

roboshop-prod

monitoring

argocd
```

---

Benefits

```text
Isolation

Security

Organization
```

---

# Common Interview Questions

## What is a Namespace?

### Short Answer

A Namespace is a logical partition used to isolate Kubernetes resources.

### Detailed Explanation

Namespaces allow multiple teams and applications to share the same cluster while keeping resources organized and isolated.

### Production Example

Separate namespaces for dev, qa, and prod environments.

### Follow-Up Questions

- Why do we need namespaces?
- Are namespaces physical isolation?

---

## How Do You Create a Namespace?

### Short Answer

Using kubectl create namespace.

### Example

```bash
kubectl create namespace roboshop
```

### Follow-Up Questions

- Can namespaces be created using YAML?
- How do you switch namespaces?

---

## What is the Difference Between Namespace-Level and Cluster-Level Resources?

### Short Answer

Namespace-level resources exist inside namespaces, while cluster-level resources are shared across the cluster.

### Examples

Namespace Resources:

```text
Pods

Services

Deployments
```

Cluster Resources:

```text
Nodes

Namespaces

StorageClasses
```

### Follow-Up Questions

- How do you identify them?
- What command shows resource types?

---

# Key Takeaways

```text
Namespaces provide logical isolation.

Namespaces help organize resources.

Multiple teams can share a cluster safely.

Namespaces separate environments like dev, qa, and prod.

default is the default namespace.

kube-system contains Kubernetes system components.

kubectl create namespace creates a namespace.

kubectl api-resources helps identify namespace and cluster resources.

Most production clusters use multiple namespaces.

Namespaces are one of the foundational concepts in Kubernetes.
```