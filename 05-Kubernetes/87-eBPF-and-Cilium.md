# eBPF and Cilium

## Introduction

Kubernetes networking looks simple from the outside.

A pod communicates with:

```text
Another Pod

Services

Ingress

External APIs
```

---

But underneath Kubernetes networking can become very complex.

---

Large production clusters may contain:

```text
Thousands Of Pods

Thousands Of Services

Millions Of Network Connections
```

---

Managing this traffic efficiently becomes difficult.

---

# Why eBPF Was Created?

Before eBPF, Linux networking heavily relied on:

```text
iptables
```

---

Network Flow

```text
Packet

↓

iptables Rules

↓

Service

↓

Pod
```

---

For small environments:

```text
Works Fine
```

---

For large environments:

```text
Thousands Of Rules

High CPU Usage

Complex Troubleshooting
```

---

# Real Production Problem

Suppose:

```text
1000 Pods

500 Services
```

---

Every Service Creates

```text
iptables Rules
```

---

Result

```text
Huge Rule Tables
```

---

Problems

```text
Slower Packet Processing

High CPU Usage

Long Troubleshooting Time
```

---

# What Is eBPF?

eBPF stands for:

```text
Extended Berkeley Packet Filter
```

---

eBPF allows programs to run directly inside:

```text
Linux Kernel
```

---

Without modifying kernel source code.

---

Think Of eBPF As

```text
Custom Logic

Running Inside

Linux Kernel
```

---

# Traditional Networking

```text
Packet

↓

iptables

↓

kube-proxy

↓

Pod
```

---

# eBPF Networking

```text
Packet

↓

eBPF

↓

Pod
```

---

Fewer Steps.

---

Better Performance.

---

# Why Is eBPF Powerful?

Because it can observe:

```text
Network Traffic

System Calls

Security Events

Performance Metrics
```

---

Directly From Kernel Space.

---

# What Is Cilium?

Cilium is a Kubernetes networking platform built on:

```text
eBPF
```

---

It acts as:

```text
CNI

Network Security Layer

Observability Platform
```

---

In many environments Cilium can replace:

```text
VPC CNI Features

kube-proxy

Traditional Network Policies
```

---

# Problem Before Cilium

Traditional Kubernetes Networking Uses

```text
iptables
```

---

Problems

```text
Large Rule Sets

Performance Issues

Difficult Debugging
```

---

Limited Visibility

```text
Who Is Talking To Whom?
```

---

Hard To Answer.

---

# What Problem Cilium Solves?

Provides:

```text
Faster Networking

Better Security

Better Visibility

Simplified Operations
```

---

Using eBPF.

---

# Cilium Architecture

```text
Pod

↓

Cilium Agent

↓

eBPF Programs

↓

Linux Kernel

↓

Network
```

---

# Cilium Components

## Cilium Agent

Runs On Every Node.

---

Responsible For

```text
Policy Enforcement

Networking

Observability
```

---

## eBPF Programs

Loaded Into

```text
Linux Kernel
```

---

Responsible For

```text
Traffic Processing
```

---

## Hubble

Observability Platform.

---

Shows

```text
Network Flows

DNS Requests

Dropped Packets

Connections
```

---

# Traditional Service Flow

```text
Client Pod

↓

Service

↓

iptables

↓

Backend Pod
```

---

# Cilium Service Flow

```text
Client Pod

↓

eBPF

↓

Backend Pod
```

---

Much Faster.

---

# Installing Cilium

Helm

```bash
helm repo add cilium https://helm.cilium.io

helm repo update

helm install cilium cilium/cilium \
  --namespace kube-system
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

# Network Policy Example

## Requirement

Only

```text
Catalogue Service
```

should access:

```text
MongoDB
```

---

All Others Blocked.

---

# Cilium Network Policy

```yaml
apiVersion: cilium.io/v2
kind: CiliumNetworkPolicy

metadata:
  name: mongodb-policy

spec:

  endpointSelector:

    matchLabels:

      app: mongodb

  ingress:

  - fromEndpoints:

    - matchLabels:

        app: catalogue
```

---

Result

```text
Catalogue → MongoDB

Allowed
```

---

```text
User → MongoDB

Blocked
```

---

# Observability With Hubble

Traditional Question

```text
Why Is My Pod Not Reaching MongoDB?
```

---

Difficult To Debug.

---

With Hubble

```text
Source Pod

↓

Destination Pod

↓

Allowed / Denied

↓

Protocol

↓

Latency
```

---

Visible Immediately.

---

# Production Example

Roboshop

```text
Catalogue

User

Cart

Shipping

MongoDB

Redis
```

---

Need

```text
Network Visibility

Security

Traffic Monitoring
```

---

Cilium + Hubble Provide

```text
End-To-End Visibility
```

---

# Service Mesh Integration

Modern Cilium Can Also Provide:

```text
Service Mesh Features
```

---

Without:

```text
Envoy Sidecars
```

for some use cases.

---

Benefits

```text
Lower Resource Usage

Simpler Architecture
```

---

# Security Use Cases

## Microsegmentation

Example

```text
Catalogue

↓

MongoDB

Allowed
```

---

```text
Frontend

↓

MongoDB

Blocked
```

---

Improves Security.

---

# DNS Visibility

See

```text
Which Pod

Accessed

Which Domain
```

---

Example

```text
Catalogue

↓

api.stripe.com
```

---

Visible In Hubble.

---

# Production Architecture

```text
Pod

↓

Cilium Agent

↓

eBPF

↓

Linux Kernel

↓

Network
```

---

# Real World Use Cases

## Financial Applications

Need

```text
Strong Security
```

---

## SaaS Platforms

Need

```text
Network Visibility
```

---

## Large EKS Clusters

Need

```text
High Performance Networking
```

---

# Common Production Problems

## Cilium Pods Not Running

Check

```bash
kubectl get pods -n kube-system
```

---

## Policy Not Working

Verify

```bash
kubectl get ciliumnetworkpolicy
```

---

## Connectivity Issues

Check

```bash
cilium status
```

---

## Traffic Visibility Missing

Verify

```bash
hubble status
```

---

# Troubleshooting Commands

Cluster Status

```bash
cilium status
```

---

Endpoints

```bash
cilium endpoint list
```

---

Policies

```bash
kubectl get ciliumnetworkpolicy
```

---

Flows

```bash
hubble observe
```

---

DNS Requests

```bash
hubble observe --protocol dns
```

---

# Common Interview Questions

## Why Was eBPF Created?

### Short Answer

eBPF was created to allow programs to run safely inside the Linux kernel for networking, security, and observability use cases.

### Detailed Explanation

Traditional networking relies heavily on iptables and kernel-user space interactions. eBPF enables programmable kernel-level processing, reducing overhead and improving performance while providing deep visibility into system behavior.

### Production Example

```text
Packet

↓

eBPF

↓

Pod
```

instead of

```text
Packet

↓

iptables

↓

Pod
```

### Follow-Up Questions

- Why is eBPF faster?
- Does eBPF replace Linux kernel?
- What other use cases exist?
- Is eBPF Kubernetes-specific?

---

## What Problem Does Cilium Solve?

### Short Answer

Cilium provides high-performance networking, security, and observability using eBPF.

### Detailed Explanation

Traditional Kubernetes networking becomes harder to manage as clusters grow. Cilium replaces many iptables-based operations with eBPF and provides visibility into network communication.

### Production Example

```text
1000 Pods

↓

Network Visibility

↓

Cilium + Hubble
```

### Follow-Up Questions

- Is Cilium a CNI?
- Can it replace kube-proxy?
- Can it enforce network policies?
- How does it improve observability?

---

## What Is Hubble?

### Short Answer

Hubble is the observability component of Cilium.

### Detailed Explanation

Hubble collects flow information from eBPF programs and shows pod-to-pod communication, DNS requests, dropped packets, and latency information.

### Production Example

```text
Catalogue

↓

MongoDB

↓

Allowed

↓

20ms
```

### Follow-Up Questions

- How does Hubble get data?
- What visibility does it provide?
- Is packet capture required?
- How is troubleshooting simplified?

---

## Difference Between Kubernetes Network Policies And Cilium Network Policies?

### Short Answer

Cilium Network Policies provide more advanced controls and visibility than standard Kubernetes Network Policies.

### Detailed Explanation

Standard Network Policies mainly focus on Layer 3 and Layer 4 traffic. Cilium can enforce policies using DNS names, HTTP paths, identities, and other advanced criteria.

### Production Example

```text
Allow

Catalogue

↓

MongoDB
```

Block

```text
Frontend

↓

MongoDB
```

### Follow-Up Questions

- Can both coexist?
- Which one is preferred?
- Does Cilium require eBPF?
- What additional capabilities exist?

---

# Key Takeaways

```text
eBPF allows programmable logic to run inside the Linux kernel.

eBPF improves networking performance and observability.

Cilium is a Kubernetes networking platform built on eBPF.

Cilium provides networking, security, and observability.

Hubble provides network flow visibility.

Cilium Network Policies are more powerful than standard Network Policies.

Cilium is increasingly adopted in large Kubernetes and EKS environments.

eBPF and Cilium are important platform engineering technologies for modern Kubernetes clusters.
```