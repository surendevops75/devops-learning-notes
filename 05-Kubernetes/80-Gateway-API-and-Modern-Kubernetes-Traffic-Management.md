# Gateway API and Modern Kubernetes Traffic Management

## Introduction

For many years Kubernetes used:

```text
Service

↓

Ingress

↓

Ingress Controller
```

for exposing applications.

---

Example

```text
Internet

↓

ALB

↓

Ingress

↓

Service

↓

Pod
```

---

Ingress solved many problems but had limitations.

---

Modern Kubernetes introduces:

```text
Gateway API
```

which is the next generation of:

```text
Ingress
```

---

# Evolution of Kubernetes Networking

## Phase 1

Only Service

```text
User

↓

NodePort

↓

Service

↓

Pod
```

---

Problems

```text
No Host Routing

No TLS Management

No Advanced Traffic Policies
```

---

## Phase 2

Ingress

```text
User

↓

ALB

↓

Ingress

↓

Service

↓

Pod
```

---

Benefits

```text
Host Based Routing

Path Based Routing

TLS Support
```

---

Problems

```text
Vendor Specific Annotations

Limited Traffic Policies

Difficult Multi-Team Management
```

---

## Phase 3

Gateway API

```text
User

↓

Gateway

↓

HTTPRoute

↓

Service

↓

Pod
```

---

Benefits

```text
Standardized

Role Separation

Advanced Routing

Traffic Splitting

Canary Support

Blue-Green Support
```

---

# Why Ingress Needed Replacement?

Traditional Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
```

---

Everything Exists In:

```text
Single Resource
```

---

Responsibilities Mixed Together

```text
Load Balancer

TLS

Routing

Traffic Policies

Security
```

---

Example

```yaml
alb.ingress.kubernetes.io/scheme

alb.ingress.kubernetes.io/group.name

alb.ingress.kubernetes.io/target-type
```

---

NGINX Uses

```yaml
nginx.ingress.kubernetes.io/rewrite-target
```

---

Problem

```text
Every Controller

Uses Different Annotations
```

---

No Standardization.

---

# What Is Gateway API?

Gateway API is a Kubernetes networking framework that separates:

```text
Infrastructure

Routing

Traffic Policies
```

into independent resources.

---

Think Of It As

```text
Ingress 2.0
```

---

# Gateway API Components

Gateway API consists of:

```text
GatewayClass

Gateway

HTTPRoute

TCPRoute

GRPCRoute
```

---

Most Common Components

```text
GatewayClass

Gateway

HTTPRoute
```

---

# Gateway API Architecture

```text
Internet

↓

Load Balancer

↓

Gateway

↓

HTTPRoute

↓

Service

↓

Pod
```

---

# Component 1: GatewayClass

## What Is GatewayClass?

GatewayClass defines:

```text
Which Controller

Will Implement Gateway
```

---

Similar To

```text
IngressClass
```

---

Examples

```text
AWS Gateway Controller

NGINX Gateway Controller

Istio Gateway Controller
```

---

Architecture

```text
GatewayClass

↓

Gateway Controller
```

---

Example

```yaml
kind: GatewayClass
```

---

# Component 2: Gateway

## What Is Gateway?

Gateway represents:

```text
Load Balancer
```

---

Examples

```text
ALB

NLB

NGINX Gateway
```

---

Gateway Handles

```text
Listeners

TLS

Ports

Protocols
```

---

Example

```text
HTTPS

Port 443
```

---

Architecture

```text
Gateway

↓

ALB
```

---

# Component 3: HTTPRoute

## What Is HTTPRoute?

HTTPRoute contains:

```text
Routing Rules
```

---

Example

```text
catalogue.roboshop.com

↓

Catalogue Service
```

---

Another Example

```text
cart.roboshop.com

↓

Cart Service
```

---

Architecture

```text
Gateway

↓

HTTPRoute

↓

Service
```

---

# Traditional Ingress Example

Ingress

```yaml
kind: Ingress
```

---

Contains

```text
Host Rules

TLS

Routing

Annotations
```

all together.

---

Example

```text
catalogue.roboshop.com

↓

Catalogue Service
```

---

# Gateway API Equivalent

Gateway

```text
Owns

Load Balancer
```

---

HTTPRoute

```text
Owns

Routing Rules
```

---

Result

```text
Clean Separation
```

---

# Production Example

Roboshop Application

Services

```text
Catalogue

User

Cart

Shipping

Payment
```

---

Traditional Ingress

```text
One Large Ingress File
```

---

Managed By

```text
Platform Team

Security Team

Application Team
```

---

Problem

```text
Multiple Teams

Editing Same Resource
```

---

# Gateway API Solution

Platform Team Creates

```text
Gateway
```

---

Application Teams Create

```text
HTTPRoutes
```

---

Result

```text
Independent Management
```

---

# Production Architecture

```text
Internet

↓

ALB

↓

Gateway

├── Catalogue Route

├── User Route

├── Cart Route

├── Payment Route

↓

Services

↓

Pods
```

---

# Host-Based Routing

Example

```text
catalogue.roboshop.com

↓

Catalogue Service
```

---

Example

```text
user.roboshop.com

↓

User Service
```

---

Example

```text
cart.roboshop.com

↓

Cart Service
```

---

# Path-Based Routing

Example

```text
api.roboshop.com/catalogue

↓

Catalogue Service
```

---

Example

```text
api.roboshop.com/cart

↓

Cart Service
```

---

# Traffic Splitting

One of the biggest advantages of Gateway API.

---

Current Version

```text
Catalogue-v1
```

---

New Version

```text
Catalogue-v2
```

---

Traffic Policy

```text
90%

↓

v1
```

---

```text
10%

↓

v2
```

---

Used For

```text
Canary Deployments
```

---

# Blue-Green Deployments

Current

```text
Catalogue Blue
```

---

New

```text
Catalogue Green
```

---

Traffic

```text
100%

↓

Blue
```

---

Testing

```text
90%

↓

Blue
```

```text
10%

↓

Green
```

---

Production

```text
100%

↓

Green
```

---

Rollback

```text
100%

↓

Blue
```

---

Very Easy With Gateway API.

---

# Header-Based Routing

Example

```text
User-Agent = Mobile
```

---

Route To

```text
Mobile Backend
```

---

Example

```text
User-Agent = Desktop
```

---

Route To

```text
Desktop Backend
```

---

# Multi-Tenant Architecture

Platform Team

```text
Gateway
```

---

Team-A

```text
HTTPRoute
```

---

Team-B

```text
HTTPRoute
```

---

Team-C

```text
HTTPRoute
```

---

Result

```text
Isolation

Delegation

Governance
```

---

# Gateway API vs Ingress

## Ingress

```text
Simple

Mature

Widely Used

Limited Flexibility
```

---

## Gateway API

```text
Advanced

Modular

Role-Oriented

Future Standard
```

---

# Current EKS Reality

Today Most EKS Clusters Still Use

```text
AWS Load Balancer Controller

+

Ingress
```

---

Reason

```text
Stable

Battle Tested

Well Understood
```

---

However New Kubernetes Networking Designs Are Moving Toward

```text
Gateway API
```

---

# Production Use Cases

## Canary Deployments

```text
90% v1

10% v2
```

---

## Blue-Green Deployments

```text
Blue

↓

Green
```

---

## Traffic Splitting

```text
Different Versions

Different Traffic Percentages
```

---

## Multi-Team Platforms

```text
Shared Gateway

Multiple Routes
```

---

## Service Mesh Integrations

```text
Istio

Linkerd

Consul
```

---

# Common Production Problems

## Route Not Working

Check

```text
HTTPRoute
```

---

Verify

```text
Backend Service
```

---

## Gateway Not Ready

Verify

```text
Gateway Controller
```

---

## TLS Issues

Verify

```text
Certificates

Listeners
```

---

## Traffic Not Reaching Pods

Verify

```text
Service

Endpoints

Routes
```

---

# Troubleshooting Commands

View Gateways

```bash
kubectl get gateways
```

---

Describe Gateway

```bash
kubectl describe gateway
```

---

View HTTPRoutes

```bash
kubectl get httproutes
```

---

Describe Route

```bash
kubectl describe httproute
```

---

View Services

```bash
kubectl get svc
```

---

View Endpoints

```bash
kubectl get endpointslices
```

---

# Common Interview Questions

## What Is Gateway API?

### Short Answer

Gateway API is the next-generation Kubernetes networking API designed to replace many Ingress use cases.

### Detailed Explanation

It separates infrastructure, routing, and traffic management into independent resources, enabling advanced traffic control and multi-team operations.

### Production Example

```text
Gateway

↓

HTTPRoute

↓

Service

↓

Pod
```

### Follow-Up Questions

- Why was Ingress insufficient?
- What is GatewayClass?
- What is HTTPRoute?
- How does Gateway API improve multi-tenancy?

---

## Why Is Gateway API Better Than Ingress?

### Short Answer

Gateway API provides better separation of concerns, standardization, and advanced traffic management.

### Detailed Explanation

Ingress combines multiple responsibilities into one object. Gateway API separates them into GatewayClass, Gateway, and Routes.

### Production Example

```text
Platform Team

↓

Gateway

Application Team

↓

HTTPRoute
```

### Follow-Up Questions

- Can Gateway API replace Ingress?
- Does AWS support Gateway API?
- What traffic policies are supported?
- What are the benefits for large organizations?

---

## Explain GatewayClass, Gateway, and HTTPRoute

### Short Answer

GatewayClass defines the controller, Gateway defines the load balancer, and HTTPRoute defines routing rules.

### Detailed Explanation

These three resources work together to provide modular traffic management.

### Production Example

```text
GatewayClass

↓

Gateway

↓

HTTPRoute

↓

Service

↓

Pod
```

### Follow-Up Questions

- Which resource creates the load balancer?
- Which resource defines routing?
- What is the role of the controller?
- How do teams share Gateways?

---

# Key Takeaways

```text
Gateway API is the next evolution of Kubernetes ingress.

Gateway API separates infrastructure and routing concerns.

GatewayClass defines the implementation controller.

Gateway represents the load balancer.

HTTPRoute defines traffic routing rules.

Gateway API supports canary deployments.

Gateway API supports blue-green deployments.

Gateway API supports traffic splitting.

Gateway API enables multi-team management.

Ingress remains common today, but Gateway API is becoming the future standard for Kubernetes traffic management.
```