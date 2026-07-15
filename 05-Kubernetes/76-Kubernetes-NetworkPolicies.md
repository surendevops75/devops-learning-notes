# Kubernetes Network Policies

## Introduction

By default Kubernetes networking follows:

```text
Allow All
```

---

Meaning:

```text
Any Pod

Can Communicate

With Any Other Pod
```

---

Example

```text
Catalogue Pod

↓

MongoDB Pod
```

Allowed.

---

Example

```text
User Pod

↓

MongoDB Pod
```

Also Allowed.

---

This creates a security problem.

---

# Real Production Problem

Application

```text
Catalogue
```

needs access to:

```text
MongoDB
```

---

Application

```text
User
```

should NOT access:

```text
MongoDB
```

---

Without Network Policies

```text
All Pods

Can Talk To All Pods
```

---

Security Risk.

---

# What Is A Network Policy?

Network Policy is a Kubernetes resource that controls:

```text
Pod Communication
```

---

It acts like:

```text
Firewall

For Pods
```

---

Network Policy Controls:

```text
Ingress Traffic

Egress Traffic
```

---

# Network Policy Architecture

```text
Pod

↓

Network Policy

↓

Allow / Deny

↓

Destination Pod
```

---

# Important Requirement

Network Policies work only if:

```text
Network Plugin Supports Them
```

---

Examples

```text
Calico

Cilium

AWS VPC CNI + Network Policy Support
```

---

Without Support

```text
Policies Ignored
```

---

# Default Kubernetes Behavior

Without Network Policy

```text
Pod-A

↓

Pod-B
```

Allowed.

---

```text
Pod-C

↓

Pod-B
```

Allowed.

---

Everything Is Open.

---

# Example Application

Namespace

```text
roboshop
```

---

Pods

```text
Catalogue

User

Cart

MongoDB
```

---

Requirement

```text
Catalogue

Can Access MongoDB
```

---

Requirement

```text
User

Cannot Access MongoDB
```

---

# Types Of Network Policies

## Ingress Policy

Controls:

```text
Incoming Traffic
```

---

Example

```text
Who Can Reach MongoDB?
```

---

# Egress Policy

Controls:

```text
Outgoing Traffic
```

---

Example

```text
Which Services

Can Catalogue Access?
```

---

# Default Deny Policy

Most Production Clusters Start With:

```text
Deny All
```

---

Then Explicitly Allow Required Traffic.

---

# Default Deny Example

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: default-deny

spec:
  podSelector: {}

  policyTypes:
  - Ingress
```

---

Meaning

```text
No Incoming Traffic

Allowed
```

---

# Allow Catalogue To MongoDB

Architecture

```text
Catalogue

↓

MongoDB
```

Allowed.

---

All Others

```text
Denied
```

---

# Example Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: mongodb-policy

spec:

  podSelector:
    matchLabels:
      app: mongodb

  policyTypes:
  - Ingress

  ingress:

  - from:

    - podSelector:
        matchLabels:
          app: catalogue
```

---

Result

Allowed

```text
Catalogue

↓

MongoDB
```

---

Denied

```text
User

↓

MongoDB
```

---

Denied

```text
Cart

↓

MongoDB
```

---

# Namespace Isolation

Production Requirement

```text
Dev Namespace

Must Not Access

Prod Namespace
```

---

Network Policies Can Enforce:

```text
Namespace Isolation
```

---

Architecture

```text
dev

❌

prod
```

---

# Namespace Selector Example

```yaml
namespaceSelector:

  matchLabels:
    environment: production
```

---

# Egress Policy Example

Requirement

```text
Catalogue

Can Access

MongoDB
```

---

But Cannot Access:

```text
Internet

Other Databases
```

---

Policy

```text
Restrict Outbound Traffic
```

---

# Production Example

Microservices

```text
Catalogue

Cart

User

Shipping

Payment

MongoDB
```

---

Desired Flow

```text
Catalogue

↓

MongoDB
```

Allowed.

---

```text
User

↓

MongoDB
```

Denied.

---

```text
Payment

↓

Catalogue
```

Allowed.

---

Everything Else

```text
Denied
```

---

# Banking Application Example

Services

```text
Frontend

API

Database
```

---

Required

```text
Frontend

↓

API
```

Allowed.

---

```text
API

↓

Database
```

Allowed.

---

```text
Frontend

↓

Database
```

Denied.

---

# Zero Trust Networking

Modern Kubernetes Security Uses:

```text
Zero Trust
```

---

Principle

```text
Trust Nothing

Allow Explicitly
```

---

Network Policies Help Implement:

```text
Zero Trust
```

---

# Network Policy Flow

```text
Request

↓

Source Pod

↓

Network Policy

↓

Allow?

↓

Destination Pod
```

---

# Common Production Mistakes

## No Default Deny

Problem

```text
Entire Cluster Open
```

---

Risk

```text
Lateral Movement
```

---

# Missing DNS Access

Policy Applied

↓

Pods Cannot Resolve DNS

---

Cause

```text
CoreDNS Not Allowed
```

---

Symptoms

```text
DNS Failures
```

---

# Overly Restrictive Policy

Symptoms

```text
Application Timeout
```

---

Cause

```text
Required Traffic Blocked
```

---

# Network Policy Not Working

Cause

```text
CNI Does Not Support

Network Policies
```

---

Verify

```text
Calico

Cilium

AWS VPC CNI NP Support
```

---

# Troubleshooting Commands

View Policies

```bash
kubectl get networkpolicy
```

---

Describe Policy

```bash
kubectl describe networkpolicy
```

---

View Pods

```bash
kubectl get pods --show-labels
```

---

Test Connectivity

```bash
kubectl exec -it pod-name -- sh
```

---

Inside Pod

```bash
curl service-name
```

---

DNS Test

```bash
nslookup mongodb
```

---

# Production Architecture

```text
Internet

↓

Ingress

↓

Frontend

↓

API

↓

Database
```

---

Allowed

```text
Frontend

↓

API
```

---

Allowed

```text
API

↓

Database
```

---

Denied

```text
Frontend

↓

Database
```

---

Denied

```text
Random Pod

↓

Database
```

---

# Common Interview Questions

## What Is A Network Policy?

### Short Answer

A Network Policy is a Kubernetes resource that controls Pod-to-Pod communication.

### Detailed Explanation

Network Policies act as firewalls for Pods by controlling ingress and egress traffic based on labels, namespaces, and ports.

### Production Example

```text
Catalogue

↓

MongoDB
```

Allowed.

```text
User

↓

MongoDB
```

Denied.

### Follow-Up Questions

- Does Kubernetes deny traffic by default?
- Which CNI plugins support Network Policies?
- What is ingress vs egress?
- How do policies select Pods?

---

## Why Are Network Policies Important?

### Short Answer

They prevent unauthorized communication between workloads.

### Detailed Explanation

Without Network Policies, all Pods can communicate with each other. Policies enforce least-privilege networking.

### Production Example

```text
Frontend

↓

Database
```

Denied.

### Follow-Up Questions

- What is Zero Trust Networking?
- How do policies improve security?
- What risks exist without policies?
- Can they isolate namespaces?

---

## What Is The Difference Between Ingress And Egress Policies?

### Short Answer

Ingress controls incoming traffic while Egress controls outgoing traffic.

### Detailed Explanation

Ingress defines who can reach a Pod. Egress defines where a Pod can connect.

### Production Example

```text
Ingress

Catalogue

↓

MongoDB
```

```text
Egress

Catalogue

↓

External API
```

### Follow-Up Questions

- Can both be used together?
- Which is more commonly forgotten?
- How do you troubleshoot egress failures?
- What happens if policyTypes is omitted?

---

## What Is A Default Deny Policy?

### Short Answer

A policy that blocks all traffic unless explicitly allowed.

### Detailed Explanation

Default deny policies are commonly used in production to implement Zero Trust security models.

### Production Example

```text
All Traffic

Denied

↓

Explicit Allow Rules Added
```

### Follow-Up Questions

- Why start with deny-all?
- What traffic should be allowed first?
- How does DNS fit into deny-all?
- Can deny-all break applications?

---

# Key Takeaways

```text
By default Kubernetes allows Pod-to-Pod communication.

Network Policies act as Pod firewalls.

Policies control ingress and egress traffic.

Production clusters usually start with default deny policies.

Network Policies are essential for Zero Trust security.

Policies use labels and namespaces for traffic control.

Network Policies require CNI support.

Namespace isolation can be enforced using policies.

Always test DNS access after applying policies.

Network Policies are a common Kubernetes security interview topic.
```