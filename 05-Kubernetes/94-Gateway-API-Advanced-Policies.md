# Gateway API Advanced Policies

## Introduction

For many years Kubernetes used:

```text
Service

↓

Ingress

↓

Application
```

---

Ingress solved basic problems:

```text
Host Based Routing

Path Based Routing

TLS Termination
```

---

Example

```text
shop.example.com

↓

Frontend
```

---

```text
shop.example.com/api

↓

Backend
```

---

This worked well initially.

---

# Why Gateway API Was Created?

As Kubernetes adoption increased, organizations needed:

```text
Traffic Splitting

Traffic Mirroring

Cross Namespace Routing

Retries

Timeouts

Gateway Ownership

Multi Team Access
```

---

Ingress became difficult to extend.

---

Different Ingress Controllers implemented features differently.

---

Example

```text
NGINX Ingress

ALB Ingress

Traefik Ingress
```

---

Same YAML

```text
Different Behavior
```

---

Need

```text
Standard Kubernetes API
```

---

This led to:

```text
Gateway API
```

---

# What Is Gateway API?

Gateway API is the next generation replacement for Ingress.

---

Instead Of

```text
Ingress
```

Gateway API introduces:

```text
GatewayClass

Gateway

HTTPRoute

GRPCRoute

TCPRoute

TLSRoute
```

---

Provides:

```text
More Flexibility

More Security

More Traffic Controls
```

---

# Gateway API Architecture

Traditional

```text
User

↓

Ingress

↓

Service

↓

Pods
```

---

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

Pods
```

---

# Core Components

## GatewayClass

Defines

```text
Which Gateway Controller
```

implements Gateway.

---

Example

```yaml
kind: GatewayClass

metadata:

  name: alb
```

---

Think Of It Like

```text
IngressClass
```

---

# Gateway

Represents

```text
Load Balancer

Traffic Entry Point
```

---

Example

```yaml
kind: Gateway
```

---

Equivalent To

```text
ALB

NLB

NGINX
```

---

# HTTPRoute

Defines

```text
Routing Rules
```

---

Example

```text
/api

↓

Backend Service
```

---

# Example Architecture

```text
Internet

↓

Gateway

↓

HTTPRoute

↓

Catalogue Service

↓

Pods
```

---

# Basic Gateway Example

Gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1

kind: Gateway

metadata:

  name: roboshop

spec:

  gatewayClassName: alb

  listeners:

  - name: http

    port: 80

    protocol: HTTP
```

---

# HTTPRoute Example

```yaml
apiVersion: gateway.networking.k8s.io/v1

kind: HTTPRoute

metadata:

  name: catalogue

spec:

  parentRefs:

  - name: roboshop

  rules:

  - matches:

    - path:

        type: PathPrefix

        value: /catalogue

    backendRefs:

    - name: catalogue

      port: 8080
```

---

# Problem Gateway API Solves

Suppose

```text
Platform Team

Owns Gateway
```

---

Application Team

Owns Routes.

---

Ingress Cannot Cleanly Separate Ownership.

---

Gateway API Supports

```text
Multi Team Ownership
```

---

# Cross Namespace Routing

Gateway

```text
Namespace A
```

---

Routes

```text
Namespace B
```

---

Possible With Gateway API.

---

Very Useful In Enterprises.

---

# Advanced Policy 1

Traffic Splitting

---

Current Version

```text
v1
```

---

New Version

```text
v2
```

---

Need

```text
90% → v1

10% → v2
```

---

Gateway API Supports

```yaml
backendRefs:

- name: catalogue-v1
  weight: 90

- name: catalogue-v2
  weight: 10
```

---

Result

```text
Canary Deployment
```

---

# Production Example

100 Requests

---

Traffic

```text
90 Requests → v1

10 Requests → v2
```

---

Observe Behavior.

---

Gradually Increase.

---

# Advanced Policy 2

Blue-Green Deployment

Current

```text
Blue
```

---

New

```text
Green
```

---

Initial

```text
100% Blue
```

---

Testing

```text
95% Blue

5% Green
```

---

Final

```text
100% Green
```

---

Rollback

```text
100% Blue
```

---

Without Redeployment.

---

# Advanced Policy 3

Traffic Mirroring

Very Popular In Production.

---

Problem

Need To Test New Version.

---

Without Affecting Users.

---

Traffic

```text
User Request

↓

v1
```

---

Copy Traffic To

```text
v2
```

---

User Only Receives

```text
v1 Response
```

---

But v2 Processes Same Requests.

---

Useful For

```text
Performance Testing

Migration Testing
```

---

# Example

```text
100 Requests

↓

Production Service

↓

Mirror To New Service
```

---

# Advanced Policy 4

Header Based Routing

Route Based On

```text
Headers
```

---

Example

```text
X-Environment=beta
```

---

Route To

```text
Beta Version
```

---

Others

```text
Production Version
```

---

# Production Example

Internal Employees

```text
Beta Version
```

---

Customers

```text
Stable Version
```

---

# Advanced Policy 5

Path Based Routing

Example

```text
/api

↓

Backend
```

---

```text
/admin

↓

Admin Service
```

---

```text
/payment

↓

Payment Service
```

---

# Advanced Policy 6

Retries

Problem

```text
Temporary Network Failure
```

---

Gateway API Can Retry.

---

Flow

```text
Request

↓

Failure

↓

Retry

↓

Success
```

---

Improves Reliability.

---

# Advanced Policy 7

Timeouts

Prevent

```text
Long Running Requests
```

---

Example

```text
Request Timeout

30 Seconds
```

---

After Timeout

```text
Request Terminated
```

---

Protects System Resources.

---

# Advanced Policy 8

Rate Limiting

Prevent

```text
Abuse

DDoS

Excessive Requests
```

---

Example

```text
100 Requests

Per Minute
```

---

Additional Requests

```text
Rejected
```

---

# Gateway API vs Ingress

## Ingress

```text
Simple

Limited

Basic Routing
```

---

## Gateway API

```text
Traffic Splitting

Mirroring

Advanced Routing

Multi Team Support

Policy Framework
```

---

# Gateway API vs Istio

Important Interview Topic.

---

Gateway API

```text
North-South Traffic
```

---

Traffic From

```text
Internet

↓

Cluster
```

---

Istio

```text
East-West Traffic
```

---

Traffic Between

```text
Microservices
```

---

Many Companies Use Both.

---

# Real Production Example

EKS

```text
Internet

↓

AWS Load Balancer Controller

↓

Gateway

↓

HTTPRoute

↓

Catalogue

↓

Pods
```

---

Canary Deployment

```text
95% → v1

5% → v2
```

Managed Through Gateway API.

---

# Common Production Problems

## Route Not Attached

Check

```bash
kubectl get httproute
```

---

## Gateway Not Ready

Check

```bash
kubectl get gateway
```

---

## Wrong Backend Service

Traffic Fails.

---

Verify

```bash
kubectl get svc
```

---

## Gateway Controller Missing

Gateway Never Becomes Ready.

---

# Troubleshooting Commands

Gateways

```bash
kubectl get gateway
```

---

Gateway Classes

```bash
kubectl get gatewayclass
```

---

Routes

```bash
kubectl get httproute
```

---

Describe Gateway

```bash
kubectl describe gateway
```

---

Describe Route

```bash
kubectl describe httproute
```

---

# Common Interview Questions

## Why Was Gateway API Created?

### Short Answer

Gateway API was created to overcome Ingress limitations and provide advanced traffic management capabilities.

### Detailed Explanation

Ingress provided basic routing but lacked flexibility for advanced use cases such as traffic splitting, mirroring, retries, and multi-team ownership. Gateway API introduces standardized, extensible networking APIs.

### Production Example

```text
Canary Deployment

95% → v1

5% → v2
```

using HTTPRoute.

### Follow-Up Questions

- What are Ingress limitations?
- Is Gateway API replacing Ingress?
- Which controllers support Gateway API?

---

## What Is The Difference Between Gateway And HTTPRoute?

### Short Answer

Gateway provides the traffic entry point while HTTPRoute defines routing rules.

### Detailed Explanation

Gateway represents infrastructure such as load balancers. HTTPRoute defines how traffic should be routed to backend services.

### Production Example

```text
Gateway

↓

HTTPRoute

↓

Service

↓

Pods
```

### Follow-Up Questions

- Can multiple routes use one Gateway?
- Can routes exist in different namespaces?
- Who owns Gateway vs Routes?

---

## How Does Gateway API Support Canary Deployments?

### Short Answer

Using weighted backend references.

### Detailed Explanation

Traffic can be split between multiple service versions by assigning different weights.

### Production Example

```text
90% → v1

10% → v2
```

### Follow-Up Questions

- How do you rollback?
- How do you monitor canary success?
- How is this different from Istio?

---

## Gateway API vs Ingress?

### Short Answer

Gateway API is more flexible, extensible, and powerful than Ingress.

### Detailed Explanation

Gateway API supports advanced routing, traffic management, policy attachment, and multi-team ownership models that Ingress was not designed to handle.

### Production Example

```text
Traffic Mirroring

Traffic Splitting

Retries

Rate Limiting
```

### Follow-Up Questions

- Will Ingress be deprecated?
- Which should be used for new projects?
- Does EKS support Gateway API?

---

## Gateway API vs Istio?

### Short Answer

Gateway API primarily manages north-south traffic, while Istio manages east-west service-to-service traffic.

### Detailed Explanation

Gateway API controls traffic entering the cluster, whereas Istio focuses on communication between internal services.

### Production Example

```text
Internet

↓

Gateway API

↓

Frontend
```

```text
Frontend

↓

Istio

↓

Payment Service
```

### Follow-Up Questions

- Can they be used together?
- Which one handles mTLS?
- Which one handles service mesh features?

---

# Key Takeaways

```text
Gateway API is the next-generation replacement for Ingress.

GatewayClass defines the controller implementation.

Gateway defines traffic entry points.

HTTPRoute defines routing rules.

Gateway API supports canary deployments and blue-green deployments.

Gateway API supports traffic mirroring.

Gateway API supports retries, timeouts, and advanced routing.

Gateway API improves multi-team ownership.

Gateway API is becoming the future of Kubernetes north-south networking.

Many modern Kubernetes platforms are adopting Gateway API.
```