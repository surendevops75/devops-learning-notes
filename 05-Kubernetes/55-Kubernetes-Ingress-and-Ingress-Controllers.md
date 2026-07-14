# Kubernetes Ingress and Ingress Controllers

## Introduction

In Kubernetes, applications need to be accessed by users.

---

Example

```text
User
  │
  ▼

Web Application
```

---

Applications run inside:

```text
Pods
```

---

Pods are private.

---

Users cannot directly access Pods.

---

Kubernetes provides:

```text
Service
```

to expose applications.

---

Question:

```text
How Do We Expose

Multiple Applications

Efficiently?
```

---

Answer:

```text
Ingress
```

---

# Recap

Application

```text
Catalogue
```

runs inside:

```text
Pod
```

---

Pod IP

```text
Dynamic
```

---

Problem

```text
Pod IP Changes
```

---

Solution

```text
Service
```

---

# Service Types

Kubernetes provides:

```text
ClusterIP

NodePort

LoadBalancer
```

---

# ClusterIP

Purpose

```text
Internal Communication
```

---

Example

```text
Catalogue

↓

MongoDB
```

---

Within cluster only.

---

# NodePort

Purpose

```text
Expose Application

Through Worker Nodes
```

---

Flow

```text
User

↓

NodeIP:Port

↓

Service

↓

Pod
```

---

Problems

```text
Need Node IP

Need Port Number

Not User Friendly
```

---

# LoadBalancer Service

Purpose

```text
Expose Application

To Internet
```

---

Example

```yaml
type: LoadBalancer
```

---

In AWS

```text
AWS Load Balancer
```

gets created automatically.

---

# Problem With LoadBalancer Services

Suppose we have:

```text
Cart

Catalogue

User

Payment
```

---

Each Service

```yaml
type: LoadBalancer
```

---

Result

```text
4 AWS Load Balancers
```

---

Architecture

```text
LB-1 → Cart

LB-2 → Catalogue

LB-3 → User

LB-4 → Payment
```

---

Problems

```text
High Cost

Difficult Management

Too Many Public Endpoints
```

---

# Real Production Requirement

Applications

```text
Cart

Catalogue

User

Payment
```

---

Requirement

```text
Single Entry Point
```

---

Solution

```text
Ingress
```

---

# What is Ingress?

Ingress is a Kubernetes resource.

---

Purpose

```text
Manage External Traffic
```

---

Ingress provides:

```text
Routing Rules
```

---

Based on:

```text
Host

Path
```

---

Easy Interview Answer

```text
Ingress is a Kubernetes resource used to route external traffic to services inside the cluster.
```

---

# Ingress Architecture

```text
User
  │
  ▼

Ingress
  │
  ▼

Service
  │
  ▼

Pods
```

---

# Real Example

User Opens

```text
catalogue.roboshop.com
```

---

Ingress

↓

Catalogue Service

↓

Catalogue Pods

---

# Another Example

User Opens

```text
cart.roboshop.com
```

---

Ingress

↓

Cart Service

↓

Cart Pods

---

# Single Entry Point

Without Ingress

```text
LB-1

LB-2

LB-3

LB-4
```

---

With Ingress

```text
Single Load Balancer

↓

Ingress Rules

↓

Multiple Applications
```

---

# Important Concept

Ingress is only:

```text
Rule Definition
```

---

Ingress does not process traffic.

---

Question

```text
Who Reads Ingress Rules?
```

---

Answer

```text
Ingress Controller
```

---

# What is an Ingress Controller?

Ingress Controller is a software component.

---

Purpose

```text
Reads Ingress Rules

Routes Traffic
```

---

Without Ingress Controller

```text
Ingress Does Nothing
```

---

Very Important Interview Question.

---

# Ingress Resource vs Ingress Controller

Ingress

```text
Rules
```

---

Ingress Controller

```text
Implementation
```

---

Example

```text
Traffic Rules
```

defined in:

```yaml
kind: Ingress
```

---

Controller executes them.

---

# Real World Analogy

Ingress

```text
Traffic Rules Board
```

---

Ingress Controller

```text
Traffic Police
```

---

Rules alone do nothing.

---

Need someone to enforce them.

---

# Popular Ingress Controllers

## NGINX Ingress Controller

Most popular.

---

Runs inside cluster.

---

# AWS Load Balancer Controller

Used in EKS.

---

Creates

```text
Application Load Balancer
```

---

# HAProxy Ingress

Alternative solution.

---

# Traefik

Cloud Native ingress controller.

---

# Traffic Flow

```text
User

↓

DNS

↓

Load Balancer

↓

Ingress Controller

↓

Service

↓

Pods
```

---

# Example Flow

User

```text
catalogue.roboshop.com
```

↓

Load Balancer

↓

Ingress Controller

↓

Catalogue Service

↓

Catalogue Pods

---

# Why Not Use NodePort?

Problems

```text
Need Node IP

Need Port Number

Poor User Experience
```

---

Example

```text
10.0.1.10:31567
```

---

Not user friendly.

---

# Why Not Use Multiple LoadBalancers?

Problems

```text
Cost

Management

Security
```

---

Production environments prefer:

```text
Single Entry Point
```

---

# Basic Ingress YAML

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: roboshop

spec:

  rules:

  - host: catalogue.roboshop.com

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: catalogue

            port:

              number: 8080
```

---

Meaning

```text
catalogue.roboshop.com

↓

Catalogue Service
```

---

# Important Components

Host

```yaml
host:
```

---

Defines

```text
Domain Name
```

---

Path

```yaml
path:
```

---

Defines

```text
URL Path
```

---

Backend

```yaml
backend:
```

---

Defines

```text
Target Service
```

---

# Ingress Supports

```text
Host Based Routing

Path Based Routing
```

---

Covered in next chapter.

---

# Kubernetes Networking Flow

```text
Ingress

↓

Service

↓

Pod
```

---

Ingress never directly connects to Pod.

---

Service is mandatory.

---

# Can Ingress Route To Pod?

No.

---

Ingress routes to:

```text
Service
```

---

Service routes to:

```text
Pod
```

---

# Production Example

Roboshop

Applications

```text
Catalogue

Cart

User

Payment
```

---

Traffic

```text
catalogue.roboshop.com

↓

Catalogue Service

↓

Catalogue Pods
```

---

Traffic

```text
cart.roboshop.com

↓

Cart Service

↓

Cart Pods
```

---

Single Load Balancer.

---

# Advantages of Ingress

## Cost Optimization

Single Load Balancer.

---

## Centralized Routing

One entry point.

---

## Better Security

Fewer exposed endpoints.

---

## Easier Management

One place for routing rules.

---

# Common Problems

## Ingress Created But Not Working

Cause

```text
Ingress Controller Missing
```

---

Most common issue.

---

# DNS Not Resolving

Cause

```text
DNS Record Missing
```

---

Verify

```text
Route53

DNS Provider
```

---

# Service Not Found

Cause

```text
Wrong Service Name
```

---

Check

```bash
kubectl get svc
```

---

# Pod Not Reachable

Cause

```text
Service Selector Issue
```

---

Check

```bash
kubectl describe svc service-name
```

---

# Troubleshooting

View Ingress

```bash
kubectl get ingress
```

---

Describe Ingress

```bash
kubectl describe ingress ingress-name
```

---

View Services

```bash
kubectl get svc
```

---

View Endpoints

```bash
kubectl get endpoints
```

---

Check Pods

```bash
kubectl get pods
```

---

# Common Interview Questions

## What is Ingress?

### Short Answer

Ingress is a Kubernetes resource used to route external traffic to services.

### Detailed Explanation

Ingress defines host-based and path-based routing rules for applications running inside Kubernetes.

### Production Example

catalogue.roboshop.com routes to Catalogue Service.

### Follow-Up Questions

- Does Ingress expose Pods directly?
- Does Ingress require a Service?
- Can Ingress work without a controller?

---

## What is an Ingress Controller?

### Short Answer

An Ingress Controller implements and enforces Ingress rules.

### Detailed Explanation

Ingress resources only define rules. The Ingress Controller reads those rules and routes traffic accordingly.

### Production Example

AWS Load Balancer Controller creates an ALB and applies Ingress rules.

### Follow-Up Questions

- What happens if no controller exists?
- Which ingress controllers have you used?
- Is ALB itself an ingress controller?

---

## Difference Between Ingress and Service?

### Short Answer

Service exposes Pods, while Ingress routes external traffic to Services.

### Detailed Explanation

Ingress sits in front of Services and provides centralized routing.

### Production Example

Ingress routes catalogue.roboshop.com to Catalogue Service.

### Follow-Up Questions

- Can Ingress route directly to Pods?
- Does Service require Ingress?
- Which comes first in traffic flow?

---

## Why Use Ingress?

### Short Answer

Ingress provides a single entry point for multiple applications.

### Detailed Explanation

Instead of creating multiple LoadBalancers, one Ingress can route traffic to many services.

### Production Example

Cart, Catalogue, User, and Payment share a single ALB.

### Follow-Up Questions

- What cost benefits exist?
- What routing methods are supported?
- What is host-based routing?

---

## Can Ingress Work Without an Ingress Controller?

### Short Answer

No.

### Detailed Explanation

Ingress only defines routing rules. An Ingress Controller is required to enforce them.

### Production Example

Creating an Ingress resource without AWS Load Balancer Controller results in no traffic routing.

### Follow-Up Questions

- What does the controller do?
- Which controllers are common?
- How do you verify controller installation?

---

# Key Takeaways

```text
Ingress is a Kubernetes resource.

Ingress routes external traffic to Services.

Ingress does not route directly to Pods.

Ingress requires an Ingress Controller.

Ingress Controllers read and implement routing rules.

Ingress reduces the number of LoadBalancers.

Ingress provides a single entry point.

AWS Load Balancer Controller is commonly used in EKS.

Host-based and path-based routing are the two primary routing methods.

Ingress is one of the most important Kubernetes networking concepts.
```