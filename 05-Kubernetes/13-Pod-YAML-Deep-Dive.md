# Pod YAML Deep Dive

## Introduction

In Kubernetes, resources are typically created using:

```text
YAML Manifests
```

---

Examples:

```text
Pods

Services

Deployments

ConfigMaps

Secrets

Ingress
```

---

A YAML file describes:

```text
Desired State
```

that Kubernetes should maintain.

---

Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx

spec:
  containers:
  - name: nginx
    image: nginx
```

---

# Why YAML?

Instead of creating resources manually every time:

```bash
kubectl run nginx
```

we define infrastructure as code.

---

Benefits

```text
Version Control

Repeatability

Automation

GitOps Compatibility
```

---

# Anatomy of a Pod YAML

Example

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: nginx

spec:
  containers:
  - name: nginx
    image: nginx
```

---

Main Sections

```text
apiVersion

kind

metadata

spec
```

---

# apiVersion

Defines:

```text
Which Kubernetes API
Should Process The Resource
```

---

Example

```yaml
apiVersion: v1
```

---

Different resources use different versions.

---

Examples

```yaml
apiVersion: v1
```

Pods

---

```yaml
apiVersion: apps/v1
```

Deployments

ReplicaSets

StatefulSets

---

# kind

Defines:

```text
What Resource
You Are Creating
```

---

Example

```yaml
kind: Pod
```

---

Other Examples

```yaml
kind: Service
```

---

```yaml
kind: Deployment
```

---

```yaml
kind: ConfigMap
```

---

# Metadata

Metadata contains:

```text
Identity Information
```

for the resource.

---

Example

```yaml
metadata:
  name: nginx
```

---

Metadata commonly contains:

```text
Name

Namespace

Labels

Annotations
```

---

# Name

Every resource needs a name.

---

Example

```yaml
metadata:
  name: catalogue
```

---

Used to identify the resource.

---

View

```bash
kubectl get pods
```

---

Output

```text
catalogue
```

---

# Namespace

Namespace determines:

```text
Where Resource Is Created
```

---

Example

```yaml
metadata:
  namespace: roboshop
```

---

Resource created inside:

```text
roboshop Namespace
```

---

Equivalent Command

```bash
kubectl get pods -n roboshop
```

---

# Labels

Labels identify resources.

---

Example

```yaml
labels:
  project: roboshop
  env: dev
```

---

Used by:

```text
Services

Deployments

ReplicaSets
```

---

Purpose

```text
Selection

Filtering

Grouping
```

---

# Annotations

Annotations store metadata.

---

Example

```yaml
annotations:
  jenkins.url: "https://jenkins.example.com/build/6"
```

---

Used for:

```text
Build Tracking

Documentation

External Integrations
```

---

# Spec

Spec defines:

```text
Desired State
```

of the resource.

---

Example

```yaml
spec:
```

---

Everything Kubernetes should create lives inside spec.

---

# Containers Section

Every Pod must contain:

```text
At Least One Container
```

---

Example

```yaml
containers:
- name: nginx
  image: nginx
```

---

Structure

```text
Pod
 │
 └── Container
```

---

# Container Name

Example

```yaml
name: nginx
```

---

Purpose

```text
Container Identification
```

---

Useful for:

```text
Logs

Debugging

Monitoring
```

---

# Container Image

Defines:

```text
Which Image To Run
```

---

Example

```yaml
image: nginx
```

---

Examples

```yaml
image: nginx
```

---

```yaml
image: redis
```

---

```yaml
image: amazonlinux
```

---

# Image Pull Process

Kubernetes asks:

```text
Container Runtime
```

to pull image.

---

Flow

```text
Pod Created
      ↓

Image Pulled
      ↓

Container Started
```

---

# Environment Variables

Applications need configuration.

---

Example

```yaml
env:
- name: COURSE
  value: kubernetes
```

---

Application receives:

```text
COURSE=kubernetes
```

---

Inside Container

```bash
echo $COURSE
```

---

Output

```text
kubernetes
```

---

# Multiple Environment Variables

Example

```yaml
env:
- name: ENV
  value: dev

- name: APP
  value: catalogue
```

---

Useful for:

```text
Configuration

Feature Flags

Application Settings
```

---

# Resources Section

Used to control:

```text
CPU

Memory
```

consumption.

---

Example

```yaml
resources:
```

---

Contains:

```text
Requests

Limits
```

---

# Requests

Guaranteed Resources.

---

Example

```yaml
requests:
  cpu: "50m"
  memory: "64Mi"
```

---

Meaning

```text
Scheduler Guarantees
These Resources
```

---

# Limits

Maximum Allowed Resources.

---

Example

```yaml
limits:
  cpu: "100m"
  memory: "128Mi"
```

---

Meaning

```text
Container Cannot Exceed
These Values
```

---

# Resource Example

```yaml
resources:
  requests:
    cpu: "50m"
    memory: "64Mi"

  limits:
    cpu: "100m"
    memory: "128Mi"
```

---

Result

```text
Guaranteed:
50m CPU
64Mi Memory

Maximum:
100m CPU
128Mi Memory
```

---

# Complete Production Example

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: catalogue

  namespace: roboshop

  labels:
    app: catalogue
    env: dev

  annotations:
    jenkins.url: "https://jenkins.example.com/build/6"

spec:

  containers:

  - name: catalogue

    image: nginx

    resources:

      requests:
        cpu: "50m"
        memory: "64Mi"

      limits:
        cpu: "100m"
        memory: "128Mi"

    env:

    - name: COURSE
      value: kubernetes
```

---

# Creating Pod

Apply YAML

```bash
kubectl apply -f pod.yaml
```

---

Verify

```bash
kubectl get pods
```

---

# Viewing Pod Details

View Pods

```bash
kubectl get pods
```

---

Detailed Information

```bash
kubectl describe pod pod-name
```

---

Useful Information

```text
Events

Labels

Annotations

Resources

Image Details
```

---

# Accessing Pod

Open Shell

```bash
kubectl exec -it pod-name -- bash
```

---

Example

```bash
kubectl exec -it catalogue -- bash
```

---

Useful For

```text
Debugging

Configuration Validation

Network Testing
```

---

# Common YAML Mistakes

## Wrong Indentation

Wrong

```yaml
metadata:
name: nginx
```

---

Correct

```yaml
metadata:
  name: nginx
```

---

YAML is indentation-sensitive.

---

## Missing apiVersion

Wrong

```yaml
kind: Pod
```

---

Correct

```yaml
apiVersion: v1
kind: Pod
```

---

## Missing Container Name

Wrong

```yaml
containers:
- image: nginx
```

---

Correct

```yaml
containers:
- name: nginx
  image: nginx
```

---

# Pod Creation Flow

```text
YAML
  ↓

kubectl apply
  ↓

API Server
  ↓

ETCD
  ↓

Scheduler
  ↓

Node Selection
  ↓

Kubelet
  ↓

Container Runtime
  ↓

Running Pod
```

---

# Production Troubleshooting

Issue

```text
Pod Not Starting
```

---

Step 1

```bash
kubectl get pods
```

---

Step 2

```bash
kubectl describe pod pod-name
```

---

Step 3

```bash
kubectl logs pod-name
```

---

Step 4

```bash
kubectl exec -it pod-name -- bash
```

---

Common Causes

```text
Wrong Image

Wrong Configuration

Resource Constraints

Missing Environment Variables
```

---

# Common Interview Questions

## What are the main sections of a Pod YAML?

### Short Answer

```text
apiVersion

kind

metadata

spec
```

### Follow-Up Questions

- What is metadata?
- What is spec?

---

## What is apiVersion?

### Short Answer

Defines which Kubernetes API processes the resource.

### Example

```yaml
apiVersion: v1
```

### Follow-Up Questions

- Why do Deployments use apps/v1?
- What resources use v1?

---

## What is Spec?

### Short Answer

Spec defines the desired state of the resource.

### Detailed Explanation

It tells Kubernetes what should be created and maintained.

### Follow-Up Questions

- What happens if actual state differs?
- Which component maintains desired state?

---

## What is the purpose of metadata?

### Short Answer

Metadata stores identification information.

### Examples

```text
Name

Namespace

Labels

Annotations
```

### Follow-Up Questions

- Why are labels important?
- What are annotations?

---

# Key Takeaways

```text
YAML is the primary way to define Kubernetes resources.

Every resource contains apiVersion, kind, metadata, and spec.

Metadata identifies resources.

Spec defines desired state.

Containers are defined inside spec.

Environment variables provide configuration.

Resources control CPU and memory usage.

Labels and annotations are part of metadata.

kubectl apply creates resources from YAML.

Understanding YAML structure is essential for all Kubernetes resources.
```