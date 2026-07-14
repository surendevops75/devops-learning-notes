# AWS Load Balancer Controller

## Introduction

In previous chapters we learned:

```text
Ingress

Ingress Controller

Host Based Routing

Path Based Routing
```

---

In EKS, an Ingress resource alone cannot expose applications.

---

We need:

```text
AWS Load Balancer Controller
```

---

This controller reads:

```text
Ingress Resources
```

and creates:

```text
Application Load Balancers (ALB)
```

inside AWS.

---

# What is AWS Load Balancer Controller?

AWS Load Balancer Controller is a Kubernetes controller that:

```text
Monitors Ingress Resources

Creates ALBs

Creates Target Groups

Updates Listener Rules

Registers Pods
```

automatically.

---

Easy Interview Answer

```text
AWS Load Balancer Controller is an EKS controller that creates and manages AWS Application Load Balancers based on Kubernetes Ingress resources.
```

---

# Why Do We Need It?

Suppose we create:

```yaml
kind: Ingress
```

---

Question

```text
Who Creates The ALB?
```

---

Answer

```text
AWS Load Balancer Controller
```

---

Without Controller

```text
Ingress Exists

ALB Does Not Exist
```

---

Traffic Fails.

---

# Evolution of AWS Load Balancers

## Classic Load Balancer (CLB)

Old Generation.

---

Supports

```text
Layer 4

Layer 7
```

---

AWS no longer recommends CLB for new workloads.

---

# Network Load Balancer (NLB)

Supports

```text
Layer 4
```

---

Very Fast.

---

Used for:

```text
TCP

UDP

High Performance Traffic
```

---

# Application Load Balancer (ALB)

Supports

```text
Layer 7
```

---

Can inspect:

```text
Host Headers

Paths

HTTP Methods

Cookies
```

---

Perfect for:

```text
Ingress
```

---

# Why ALB For Kubernetes?

Ingress requires:

```text
Host Based Routing

Path Based Routing
```

---

ALB supports both.

---

Example

```text
catalogue.roboshop.com

↓

Catalogue Pods
```

---

Example

```text
cart.roboshop.com

↓

Cart Pods
```

---

Single ALB.

---

# Architecture

```text
User
 │

 ▼

ALB
 │

 ▼

Listener
 │

 ▼

Rules
 │

 ▼

Target Groups
 │

 ▼

Pods
```

---

# Important Components

## Listener

A Listener waits for requests.

---

Examples

```text
HTTP 80

HTTPS 443
```

---

Architecture

```text
User

↓

Listener
```

---

# Rules

Rules determine:

```text
Where Traffic Goes
```

---

Examples

```text
Host

Path
```

---

# Target Group

Target Group contains:

```text
Application Targets
```

---

Examples

```text
Pods

EC2 Instances
```

---

# Complete Request Flow

```text
User

↓

ALB

↓

Listener

↓

Rule

↓

Target Group

↓

Pod
```

---

# Real Example

Request

```text
catalogue.roboshop.com
```

---

Flow

```text
ALB

↓

Listener

↓

Host Rule

↓

Catalogue Target Group

↓

Catalogue Pods
```

---

# Another Example

Request

```text
cart.roboshop.com
```

---

Flow

```text
ALB

↓

Listener

↓

Host Rule

↓

Cart Target Group

↓

Cart Pods
```

---

# Banking Example

Request

```text
netbanking.icici.com
```

---

Flow

```text
ALB

↓

Listener

↓

Rule

↓

NetBanking TG

↓

Pods
```

---

Request

```text
corporate.icici.com
```

---

Flow

```text
ALB

↓

Listener

↓

Rule

↓

Corporate TG

↓

Pods
```

---

# Controller Responsibilities

AWS Load Balancer Controller automatically:

```text
Creates ALB

Creates Target Groups

Creates Listener Rules

Registers Pods

Updates Health Checks
```

---

No manual AWS work.

---

# Installation Prerequisites

Before installing:

```text
EKS Cluster

OIDC Provider

IAM Permissions
```

must exist.

---

# Verify OIDC

```bash
aws iam list-open-id-connect-providers
```

---

Required for:

```text
IRSA
```

---

# Create IAM Policy

AWS provides:

```text
AWS Load Balancer Controller Policy
```

---

Controller requires permissions to:

```text
Create ALBs

Create Target Groups

Modify Listeners

Manage Security Groups
```

---

# Create Service Account

Usually created through:

```bash
eksctl create iamserviceaccount
```

---

Purpose

```text
Allow Controller

To Call AWS APIs
```

---

# Install Using Helm

Add Repository

```bash
helm repo add eks https://aws.github.io/eks-charts
```

---

Update

```bash
helm repo update
```

---

Install

```bash
helm install aws-load-balancer-controller \
eks/aws-load-balancer-controller \
-n kube-system
```

---

# Verify Installation

Pods

```bash
kubectl get pods -n kube-system
```

---

Expected

```text
aws-load-balancer-controller
```

---

# Verify Deployment

```bash
kubectl get deployment \
-n kube-system
```

---

# Controller Watches

```text
Ingress

Services

TargetGroupBindings
```

---

Whenever changes occur:

```text
Controller Reconciles AWS Resources
```

---

# Reconciliation Concept

Desired State

```text
Ingress Exists
```

---

Actual State

```text
No ALB
```

---

Controller Creates ALB.

---

Desired State Matches Actual State.

---

# Production Architecture

```text
User
 │

 ▼

Route53
 │

 ▼

ALB
 │

 ▼

Listener
 │

 ▼

Rule
 │

 ▼

Target Group
 │

 ▼

Kubernetes Pods
```

---

# Benefits

## Automation

No manual ALB creation.

---

## Scalability

Automatically adjusts target registration.

---

## Kubernetes Native

Uses Ingress resources.

---

## Cost Optimization

Single ALB

Multiple Applications.

---

# Common Problems

## ALB Not Created

Cause

```text
Controller Missing
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

## IAM Permission Missing

Cause

```text
Access Denied
```

---

Controller cannot create ALB.

---

# OIDC Missing

Cause

```text
IRSA Failure
```

---

Controller cannot assume IAM Role.

---

# Ingress Not Processed

Cause

```text
Wrong Ingress Class
```

---

Verify

```bash
kubectl describe ingress
```

---

# Target Group Empty

Cause

```text
Pods Not Ready
```

---

Verify

```bash
kubectl get endpoints
```

---

# Troubleshooting

Check Controller Pods

```bash
kubectl get pods -n kube-system
```

---

Check Logs

```bash
kubectl logs deployment/aws-load-balancer-controller \
-n kube-system
```

---

Check Ingress

```bash
kubectl get ingress
```

---

Describe Ingress

```bash
kubectl describe ingress ingress-name
```

---

Check ALB

```bash
aws elbv2 describe-load-balancers
```

---

Check Target Groups

```bash
aws elbv2 describe-target-groups
```

---

# Common Interview Questions

## What is AWS Load Balancer Controller?

### Short Answer

It is an EKS controller that creates and manages ALBs using Kubernetes Ingress resources.

### Detailed Explanation

The controller continuously watches Ingress resources and reconciles AWS resources such as ALBs, listeners, rules, and target groups.

### Production Example

A new Ingress for catalogue.roboshop.com automatically creates listener rules and target groups.

### Follow-Up Questions

- Does it create ALBs automatically?
- Does it require IAM permissions?
- How does it authenticate to AWS?

---

## Why Do We Need AWS Load Balancer Controller?

### Short Answer

Ingress resources alone cannot create AWS load balancers.

### Detailed Explanation

The controller translates Kubernetes resources into AWS infrastructure.

### Production Example

Creating an Ingress results in automatic ALB creation.

### Follow-Up Questions

- What happens if controller is missing?
- Can Ingress work without it?
- What AWS resources are created?

---

## What Resources Does The Controller Create?

### Short Answer

ALBs, listeners, listener rules, target groups, and target registrations.

### Detailed Explanation

The controller converts Kubernetes Ingress configuration into AWS networking resources.

### Production Example

Each application receives its own target group under a shared ALB.

### Follow-Up Questions

- Does it create security groups?
- Does it create Route53 records?
- Does it register Pods automatically?

---

## Why Is ALB Preferred Over CLB?

### Short Answer

ALB supports Layer 7 routing.

### Detailed Explanation

ALB can route traffic using hosts and paths, making it ideal for Kubernetes Ingress.

### Production Example

catalogue.roboshop.com and cart.roboshop.com share one ALB.

### Follow-Up Questions

- What layer does ALB operate on?
- What is host-based routing?
- What is path-based routing?

---

## How Does AWS Load Balancer Controller Authenticate To AWS?

### Short Answer

Using IRSA.

### Detailed Explanation

The controller runs with a Service Account mapped to an IAM Role.

### Production Example

The controller assumes an IAM Role and creates ALBs through AWS APIs.

### Follow-Up Questions

- Why is OIDC required?
- Which IAM permissions are required?
- How do you verify the role?

---

# Key Takeaways

```text
AWS Load Balancer Controller is required for ALB-based Ingress in EKS.

Ingress resources alone cannot create ALBs.

The controller watches Ingress resources.

The controller creates ALBs, listeners, rules, and target groups.

ALB supports host-based and path-based routing.

OIDC and IRSA are required for secure AWS API access.

The controller continuously reconciles desired and actual state.

AWS Load Balancer Controller is the standard ingress solution for EKS.

Understanding ALB → Listener → Rule → Target Group → Pod flow is critical for interviews and production support.
```