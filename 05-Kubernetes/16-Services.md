# Kubernetes Services

## Introduction

Pods are the smallest deployable units in Kubernetes.

However Pods have a major problem.

---

# Problem With Pods

Every Pod gets an IP address.

Example

```text
Catalogue Pod

10.244.1.10
```

---

Cart Service calls:

```text
10.244.1.10
```

---

Everything works.

---

Now Pod crashes.

Kubernetes creates a new Pod.

---

New Pod

```text
10.244.2.15
```

---

Problem

```text
Old IP No Longer Exists
```

---

Applications fail to communicate.

---

Pod IPs are:

```text
Ephemeral

Temporary

Dynamic
```

---

We need a stable endpoint.

---

Solution

```text
Services
```

---

# What is a Service?

A Service is an abstraction layer that provides:

```text
Stable IP

Stable DNS

Load Balancing

Service Discovery
```

for Pods.

---

Think of Service as:

```text
Permanent Address

For Temporary Pods
```

---

# Why Do We Need Services?

Services solve:

```text
Pod IP Changes

Service Discovery

Load Balancing

Application Communication
```

---

Without Service

```text
Cart
 ↓
10.244.1.10
```

---

Pod changes.

Application breaks.

---

With Service

```text
Cart
 ↓
catalogue
 ↓
Pods
```

---

Pods can change.

Service remains same.

---

# Service Architecture

```text
Cart Service
       ↓

Catalogue Service
       ↓

Pod-1

Pod-2

Pod-3
```

---

Service acts as:

```text
Load Balancer
```

---

# Labels and Services

Services use:

```text
Labels
```

to find Pods.

---

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

Result

```text
Service Automatically
Finds Matching Pods
```

---

# Service YAML

Example

```yaml
apiVersion: v1
kind: Service

metadata:
  name: catalogue

spec:
  selector:
    app: catalogue

  ports:
  - port: 8080
    targetPort: 8080
```

---

Create Service

```bash
kubectl apply -f service.yaml
```

---

Verify

```bash
kubectl get svc
```

---

# Types of Services

Kubernetes supports:

```text
ClusterIP

NodePort

LoadBalancer
```

---

# ClusterIP

Default Service Type.

---

Used for:

```text
Internal Communication
```

inside cluster.

---

Architecture

```text
Cart
 ↓

Catalogue Service
 ↓

Catalogue Pods
```

---

Traffic remains inside cluster.

---

Example

```yaml
spec:
  type: ClusterIP
```

---

Or simply:

```yaml
spec:
```

because ClusterIP is default.

---

# ClusterIP Example

Catalogue Service

```yaml
apiVersion: v1
kind: Service

metadata:
  name: catalogue

spec:

  selector:
    app: catalogue

  ports:
  - port: 8080
    targetPort: 8080
```

---

Access

```text
Only From Cluster
```

---

Good For

```text
Catalogue

User

Cart

Shipping

Payment
```

---

# ClusterIP Flow

```text
Cart Pod
      ↓

Catalogue Service
      ↓

Catalogue Pods
```

---

Most commonly used Service type.

---

# NodePort

Used to expose application outside cluster.

---

Architecture

```text
Internet
    ↓

Node IP
    ↓

NodePort
    ↓

Pods
```

---

Example

```yaml
spec:
  type: NodePort
```

---

Port Range

```text
30000 - 32767
```

---

Example

```yaml
ports:

- port: 80

  targetPort: 8080

  nodePort: 30080
```

---

Access

```text
http://NodeIP:30080
```

---

Example

```text
http://54.20.10.5:30080
```

---

# NodePort Flow

```text
User
 ↓

NodeIP:30080
 ↓

Service
 ↓

Pods
```

---

Advantages

```text
Simple

Easy To Test
```

---

Disadvantages

```text
Not Production Friendly

Port Management Required

Limited Scalability
```

---

# LoadBalancer

Most common production Service type.

---

Used in:

```text
AWS

Azure

GCP
```

---

Example

```yaml
spec:
  type: LoadBalancer
```

---

Architecture

```text
Internet
      ↓

AWS Load Balancer
      ↓

Service
      ↓

Pods
```

---

Benefits

```text
External Access

High Availability

Cloud Integration
```

---

# LoadBalancer Example

```yaml
apiVersion: v1
kind: Service

metadata:
  name: frontend

spec:

  type: LoadBalancer

  selector:
    app: frontend

  ports:

  - port: 80

    targetPort: 8080
```

---

Create Service

```bash
kubectl apply -f service.yaml
```

---

Check

```bash
kubectl get svc
```

---

Output

```text
EXTERNAL-IP
```

appears.

---

# Service Discovery

Every Service automatically gets DNS.

---

Example

Service

```text
catalogue
```

---

DNS

```text
catalogue.default.svc.cluster.local
```

---

Short Form

```text
catalogue
```

---

Applications usually use:

```text
catalogue
```

only.

---

# Real Example

Cart Service

Needs Catalogue.

---

Instead of:

```text
10.244.1.10
```

---

Use

```text
catalogue
```

---

Benefits

```text
Stable

Reliable

Easy To Maintain
```

---

# Endpoints

Service itself does not run application.

---

Service points to:

```text
Endpoints
```

---

Endpoints are:

```text
Actual Pod IPs
```

behind Service.

---

View Endpoints

```bash
kubectl get endpoints
```

---

Example

```text
catalogue

10.244.1.10

10.244.1.15

10.244.2.11
```

---

Service load balances across endpoints.

---

# Service Load Balancing

Suppose:

```text
Catalogue Pod-1

Catalogue Pod-2

Catalogue Pod-3
```

---

Service distributes traffic.

---

Example

```text
Request-1 → Pod-1

Request-2 → Pod-2

Request-3 → Pod-3
```

---

Benefits

```text
High Availability

Scalability
```

---

# Port vs TargetPort

Example

```yaml
ports:

- port: 80

  targetPort: 8080
```

---

Meaning

```text
Service Listens On

80
```

---

Forwards Traffic To

```text
Container Port

8080
```

---

# Service Troubleshooting

View Services

```bash
kubectl get svc
```

---

Describe Service

```bash
kubectl describe svc catalogue
```

---

Check Endpoints

```bash
kubectl get endpoints
```

---

# Common Problem

Service Exists

But Application Not Working.

---

Check

```text
Labels

Selectors
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

Service Fails.

---

# Production Example (EKS)

Roboshop

```text
Frontend
```

Uses

```text
LoadBalancer
```

---

Internal Services

```text
Catalogue

User

Cart

Shipping

Payment
```

Use

```text
ClusterIP
```

---

Architecture

```text
Internet
    ↓

AWS Load Balancer
    ↓

Frontend
    ↓

Catalogue
    ↓

MongoDB
```

---

# Common Interview Questions

## Why Do We Need Services?

### Short Answer

Services provide stable networking for Pods.

### Detailed Explanation

Pod IPs are temporary. Services provide a stable IP, DNS name, and load balancing.

### Production Example

Cart service accessing catalogue service through DNS.

### Follow-Up Questions

- Why are Pod IPs not reliable?
- What is service discovery?

---

## What is ClusterIP?

### Short Answer

ClusterIP exposes applications internally within the cluster.

### Production Example

Catalogue service communicating with cart service.

### Follow-Up Questions

- Is ClusterIP accessible from outside?
- Is ClusterIP the default service type?

---

## What is NodePort?

### Short Answer

NodePort exposes applications using a port on every node.

### Example

```text
NodeIP:30080
```

### Follow-Up Questions

- What is NodePort range?
- Why is NodePort rarely used in production?

---

## What is LoadBalancer?

### Short Answer

LoadBalancer creates a cloud load balancer and exposes applications externally.

### Production Example

Frontend service exposed through AWS Load Balancer.

### Follow-Up Questions

- How does EKS create ALBs/NLBs?
- What is the difference between LoadBalancer and Ingress?

---

## What are Endpoints?

### Short Answer

Endpoints are the actual Pod IPs behind a Service.

### Follow-Up Questions

- How can you view endpoints?
- What happens when endpoints are empty?

---

# Key Takeaways

```text
Pod IPs are ephemeral.

Services provide stable networking.

Services provide DNS-based service discovery.

Services provide load balancing.

ClusterIP is the default service type.

NodePort exposes applications using node ports.

LoadBalancer exposes applications externally.

Services use labels and selectors.

Endpoints represent actual Pod IPs.

Services are one of the most important Kubernetes networking concepts.
```