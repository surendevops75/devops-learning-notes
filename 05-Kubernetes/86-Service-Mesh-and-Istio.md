# Service Mesh and Istio

## Introduction

As Kubernetes adoption increased, applications evolved from:

```text
Monolithic Applications
```

to

```text
Microservices
```

---

Example

```text
Frontend

↓

User Service

↓

Catalogue Service

↓

Cart Service

↓

Payment Service

↓

Shipping Service
```

---

Now hundreds of services communicate with each other.

---

# Problem Before Service Mesh

Suppose:

```text
User Service

↓

Payment Service
```

communicate over HTTP.

---

Business Requirement 1

```text
Encrypt All Traffic
```

---

Without Service Mesh

Developers must implement:

```text
TLS

Certificates

Authentication
```

inside application code.

---

# Business Requirement 2

Need:

```text
Retries
```

---

Example

```text
Payment Service

↓

Temporary Failure
```

---

Need automatic retry.

---

Again developers write code.

---

# Business Requirement 3

Need

```text
Canary Deployment
```

Example

```text
90% → v1

10% → v2
```

---

Without Service Mesh

Application code changes required.

---

# Business Requirement 4

Need

```text
Request Tracing

Latency Monitoring

Traffic Visibility
```

for every service.

---

Without Service Mesh

Every team implements monitoring differently.

---

# Problem Summary

Microservices Need

```text
Security

Traffic Management

Observability

Reliability
```

---

But these concerns should not be implemented repeatedly inside every application.

---

# What Is A Service Mesh?

A Service Mesh is an infrastructure layer that manages:

```text
Service To Service Communication
```

---

Instead Of

```text
Application Handling Networking
```

---

We Move Networking Logic To

```text
Service Mesh
```

---

# What Is Istio?

Istio is the most popular Service Mesh for Kubernetes.

---

It provides:

```text
mTLS

Traffic Splitting

Canary Deployments

Blue-Green Deployments

Retries

Circuit Breaking

Observability

Tracing
```

---

Without changing application code.

---

# Main Goal

Move

```text
Networking Logic
```

from

```text
Application Code
```

to

```text
Infrastructure Layer
```

---

# Istio Architecture

Traditional Kubernetes

```text
Application

↓

Application
```

---

Istio

```text
Application

↓

Envoy Sidecar

↓

Envoy Sidecar

↓

Application
```

---

# What Is Envoy?

Envoy is a high-performance proxy.

---

Every Pod gets:

```text
Application Container

+

Envoy Sidecar
```

---

# Sidecar Pattern

Example

```text
Catalogue Pod

├── Catalogue Container

└── Envoy Sidecar
```

---

```text
User Pod

├── User Container

└── Envoy Sidecar
```

---

Traffic

```text
User

↓

Envoy

↓

Envoy

↓

Catalogue
```

---

# Why Sidecars?

Because Istio can:

```text
Observe Traffic

Encrypt Traffic

Control Traffic
```

without modifying applications.

---

# Sidecar Injection

Enable Namespace

```yaml
apiVersion: v1
kind: Namespace

metadata:

  name: roboshop

  labels:

    istio-injection: enabled
```

---

New Pods Automatically Receive

```text
Envoy Sidecar
```

---

# Istio Components

## Istiod

Control Plane.

---

Responsible For

```text
Configuration

Certificates

Policies

Service Discovery
```

---

## Envoy

Data Plane.

---

Responsible For

```text
Traffic Processing
```

---

# Architecture

```text
Istiod

↓

Envoy Sidecars

↓

Application Pods
```

---

# mTLS

## Problem

Service Communication

```text
Plain Text
```

---

Traffic Can Be Intercepted.

---

# Solution

Mutual TLS

```text
Encrypted

Authenticated
```

Communication.

---

Flow

```text
Envoy

↓

Encrypted Traffic

↓

Envoy
```

---

# Production Example

Banking Application

```text
Account Service

↓

Payment Service
```

---

Traffic Must Be

```text
Encrypted
```

---

Istio Provides

```text
Automatic mTLS
```

---

# Traffic Splitting

One Of Istio's Most Popular Features.

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

Need

```text
90% Traffic → v1

10% Traffic → v2
```

---

# DestinationRule

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule

metadata:
  name: catalogue

spec:

  host: catalogue

  subsets:

  - name: v1

    labels:
      version: v1

  - name: v2

    labels:
      version: v2
```

---

# VirtualService

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService

metadata:
  name: catalogue

spec:

  hosts:
  - catalogue

  http:

  - route:

    - destination:
        host: catalogue
        subset: v1
      weight: 90

    - destination:
        host: catalogue
        subset: v2
      weight: 10
```

---

Result

```text
100 Requests

↓

90 → v1

10 → v2
```

---

# Blue-Green Deployment

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

Traffic

```text
100% Blue
```

---

Testing

```text
90% Blue

10% Green
```

---

After Validation

```text
100% Green
```

---

Rollback

```text
100% Blue
```

---

Without redeploying applications.

---

# Retries

Problem

```text
Temporary Network Failure
```

---

Without Istio

Developers implement retries.

---

With Istio

```yaml
retries:

  attempts: 3

  perTryTimeout: 2s
```

---

Automatically.

---

# Circuit Breaking

Problem

```text
Payment Service Failing
```

---

Without Protection

```text
All Requests Fail
```

---

With Circuit Breaker

```text
Stop Sending Traffic
```

to unhealthy service.

---

Protects system.

---

# Observability

Istio Automatically Provides

```text
Request Metrics

Latency

Error Rate

Traffic Volume
```

---

Example

```text
Catalogue

↓

500 Requests/sec

↓

50 ms Latency
```

---

Visible without application changes.

---

# Distributed Tracing

Request

```text
Frontend

↓

User

↓

Cart

↓

Payment
```

---

Need

```text
End-To-End Trace
```

---

Istio supports:

```text
Jaeger

Zipkin

Grafana Tempo
```

---

# Real Production Example

E-Commerce Application

```text
Frontend

↓

Catalogue

↓

Cart

↓

Payment

↓

Shipping
```

---

Need

```text
Canary Releases

mTLS

Tracing

Observability
```

---

Istio Provides All.

---

# Installation

Download Istio

```bash
curl -L https://istio.io/downloadIstio | sh
```

---

Install

```bash
istioctl install
```

---

Verify

```bash
kubectl get pods -n istio-system
```

---

# Common Production Problems

## Sidecar Missing

Check

```bash
kubectl get pods
```

---

Verify

```text
Envoy Container Present
```

---

## Traffic Not Routing

Verify

```bash
kubectl get virtualservice
```

---

## mTLS Failures

Verify

```bash
kubectl get peerauthentication
```

---

## Configuration Issues

Check

```bash
istioctl analyze
```

---

# Troubleshooting Commands

Virtual Services

```bash
kubectl get virtualservice
```

---

Destination Rules

```bash
kubectl get destinationrule
```

---

Pods

```bash
kubectl get pods
```

---

Proxy Status

```bash
istioctl proxy-status
```

---

Analyze

```bash
istioctl analyze
```

---

# Common Interview Questions

## Why Was Istio Created?

### Short Answer

Istio was created to solve microservice communication challenges such as security, traffic management, observability, and reliability.

### Detailed Explanation

As organizations moved from monolithic applications to microservices, every service needed encryption, retries, traffic control, monitoring, and tracing. Implementing these features in every application created duplication and inconsistency. Istio moved these concerns into infrastructure.

### Production Example

```text
User Service

↓

Payment Service

↓

Encrypted

Monitored

Retried
```

without application code changes.

### Follow-Up Questions

- Why is Service Mesh needed?
- What problems does Istio solve?
- Why not implement this in code?
- What alternatives exist?

---

## What Is A Service Mesh?

### Short Answer

A Service Mesh is an infrastructure layer that manages service-to-service communication.

### Detailed Explanation

A Service Mesh inserts proxies between services and controls networking, security, and observability centrally.

### Production Example

```text
Application

↓

Envoy

↓

Envoy

↓

Application
```

### Follow-Up Questions

- What is a sidecar?
- What is Envoy?
- What is the control plane?
- What is the data plane?

---

## What Is The Difference Between Istiod And Envoy?

### Short Answer

Istiod is the control plane while Envoy is the data plane.

### Detailed Explanation

Istiod manages configuration, certificates, and policies. Envoy proxies handle actual traffic between services.

### Production Example

```text
Istiod

↓

Configure Envoy

↓

Process Traffic
```

### Follow-Up Questions

- Can Envoy run without Istiod?
- What happens if Istiod goes down?
- How does Envoy get configuration?

---

## How Does Istio Implement Canary Deployments?

### Short Answer

Using VirtualService and DestinationRule resources.

### Detailed Explanation

Istio can split traffic between multiple versions of an application without modifying application code.

### Production Example

```text
90% → v1

10% → v2
```

### Follow-Up Questions

- What is DestinationRule?
- What is VirtualService?
- How do you rollback?
- How do you monitor success?

---

## What Is mTLS?

### Short Answer

Mutual TLS encrypts and authenticates service-to-service communication.

### Detailed Explanation

Both client and server verify each other's identity and establish encrypted communication automatically through Envoy sidecars.

### Production Example

```text
Catalogue

↓

Encrypted Traffic

↓

Payment
```

### Follow-Up Questions

- Why is mTLS important?
- How are certificates managed?
- What happens when certificates expire?
- Is application code modified?

---

# Key Takeaways

```text
Istio is the most popular Kubernetes Service Mesh.

It solves security, traffic management, observability, and reliability problems.

Envoy sidecars intercept all service-to-service traffic.

Istiod is the control plane.

Envoy is the data plane.

Istio provides mTLS automatically.

Istio supports canary and blue-green deployments.

Istio provides retries, circuit breaking, and traffic management.

Istio enables distributed tracing and observability.

Service Mesh becomes valuable as microservice complexity increases.
```