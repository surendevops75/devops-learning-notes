# AWS VPC CNI and Pod Networking in EKS

## Introduction

Networking is one of the most important components in Kubernetes.

Every Pod requires:

```text
IP Address

Network Connectivity

Service Communication

Internet Access
```

---

In EKS, networking is provided by:

```text
AWS VPC CNI
```

---

Unlike many Kubernetes distributions:

```text
Pod IPs

Come Directly

From VPC CIDR
```

---

# What Is VPC CNI?

VPC CNI stands for:

```text
Virtual Private Cloud

Container Network Interface
```

---

It is responsible for:

```text
Pod Networking

IP Assignment

Network Connectivity

Pod To Pod Communication
```

---

Addon Name

```text
aws-node
```

---

Namespace

```text
kube-system
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

Example

```text
aws-node-xxxxx
```

---

# Why Is VPC CNI Needed?

When Kubernetes creates a Pod:

```text
Pod Needs IP Address
```

---

Question:

```text
Who Assigns IP?
```

---

Answer:

```text
VPC CNI
```

---

# Traditional Networking

Some Kubernetes distributions use:

```text
Overlay Networks

Calico

Flannel
```

---

Pods Receive:

```text
Virtual IPs
```

---

Traffic Requires:

```text
Encapsulation

Overlay Routing
```

---

# EKS Networking

Pods Receive:

```text
Real VPC IPs
```

---

Example

VPC CIDR

```text
10.0.0.0/16
```

---

Node

```text
10.0.1.100
```

---

Pod

```text
10.0.1.25
```

---

Both Belong To:

```text
Same VPC
```

---

# High Level Architecture

```text
VPC

↓

Subnet

↓

Node

↓

ENI

↓

Secondary IPs

↓

Pods
```

---

# What Is ENI?

ENI

```text
Elastic Network Interface
```

---

Think Of ENI As:

```text
Network Card
```

---

Laptop Example

```text
Ethernet

WiFi
```

---

AWS Equivalent

```text
ENI
```

---

Every EC2 Instance Has:

```text
Primary ENI

Secondary ENIs
```

---

Each ENI Can Hold:

```text
Multiple IP Addresses
```

---

# Pod IP Allocation

VPC CNI Uses:

```text
Secondary IPs

From ENIs
```

---

Flow

```text
Create Pod

↓

Request IP

↓

VPC CNI

↓

Assign Secondary IP

↓

Pod Starts
```

---

# Example

Node

```text
10.0.1.100
```

---

ENI

```text
eni-123
```

---

Secondary IPs

```text
10.0.1.21

10.0.1.22

10.0.1.23

10.0.1.24
```

---

Pods

```text
Pod-1 → 10.0.1.21

Pod-2 → 10.0.1.22

Pod-3 → 10.0.1.23

Pod-4 → 10.0.1.24
```

---

# Why Instance Type Matters?

Different EC2 Types Support:

```text
Different Number Of ENIs

Different Number Of IPs
```

---

Therefore:

```text
Different Pod Limits
```

---

# Example: t3.medium

Supports

```text
3 ENIs
```

---

Each ENI

```text
6 IPs
```

---

Total

```text
18 IPs
```

---

One IP Reserved For Node

```text
18 - 1
```

---

Maximum

```text
17 Pod IPs
```

---

# Example: m5.xlarge

Supports More ENIs

More Secondary IPs

---

Approximate Capacity

```text
60 IPs
```

---

One Reserved For Node

```text
60 - 1
```

---

Maximum

```text
59 Pod IPs
```

---

# Why Pod Limit Exists?

Pods Need:

```text
Unique IP
```

---

No Available IP

↓

```text
Pod Cannot Start
```

---

Status

```text
Pending
```

---

# Production Example

Node

```text
t3.medium
```

---

Current Pods

```text
17
```

---

New Deployment

```text
5 Pods
```

---

Result

```text
Insufficient IPs
```

---

Scheduler Tries:

```text
Other Nodes
```

---

If No Nodes Available

```text
Pending
```

---

# Pod To Pod Communication

Example

```text
Catalogue Pod

↓

MongoDB Pod
```

---

Traffic Flow

```text
Catalogue

↓

VPC Network

↓

MongoDB
```

---

No Overlay Network Required.

---

# Service Communication

Example

```text
User Pod

↓

MongoDB Service

↓

MongoDB Pod
```

---

Flow

```text
User Pod

↓

Service

↓

EndpointSlice

↓

MongoDB Pod
```

---

# Cross Node Communication

Node-1

```text
Catalogue
```

---

Node-2

```text
MongoDB
```

---

Flow

```text
Catalogue

↓

VPC Routing

↓

MongoDB
```

---

Works Because:

```text
All Pod IPs

Belong To VPC
```

---

# How VPC CNI Works Internally

Pod Created

↓

Scheduler Selects Node

↓

kubelet Creates Pod

↓

VPC CNI Allocates IP

↓

Network Interface Attached

↓

Pod Running

---

Architecture

```text
Scheduler

↓

Node

↓

kubelet

↓

VPC CNI

↓

ENI

↓

Secondary IP

↓

Pod
```

---

# Verify VPC CNI

Check Addon

```bash
kubectl get daemonset -n kube-system
```

---

Output

```text
aws-node
```

---

Check Pods

```bash
kubectl get pods -n kube-system
```

---

# View Node IP Capacity

Command

```bash
kubectl describe node
```

---

Useful For

```text
Pod Capacity

Allocatable Resources
```

---

# Common Production Problems

## IP Exhaustion

Symptoms

```text
Pods Pending
```

---

Reason

```text
No Available Pod IPs
```

---

Verify

```bash
kubectl describe pod
```

---

Events

```text
Failed To Assign IP
```

---

# VPC CNI Crash

Symptoms

```text
Pods Not Getting IPs
```

---

Check

```bash
kubectl get pods -n kube-system
```

---

Verify

```text
aws-node
```

---

# Wrong Subnet Size

Small Subnet

```text
/28
```

---

May Run Out Of:

```text
IP Addresses
```

---

# Node Scaling Issues

Problem

```text
More Pods

Than Available IPs
```

---

Solution

```text
Add More Nodes

Larger Instance Types

Larger Subnets
```

---

# Production Architecture

```text
VPC

↓

Subnet

↓

Node

↓

ENI

↓

Secondary IP

↓

VPC CNI

↓

Pod
```

---

# Common Interview Questions

## What Is AWS VPC CNI?

### Short Answer

AWS VPC CNI is the networking plugin used by EKS to assign VPC IP addresses directly to Pods.

### Detailed Explanation

Unlike overlay networking solutions, VPC CNI allocates real VPC IP addresses to Pods using secondary IPs attached to ENIs. This enables native AWS networking without encapsulation.

### Production Example

```text
VPC

10.0.0.0/16

↓

Pod

10.0.1.25
```

### Follow-Up Questions

- What is CNI?
- How does VPC CNI work?
- Where does the Pod IP come from?
- What addon provides VPC CNI?

---

## What Is An ENI?

### Short Answer

An ENI is an Elastic Network Interface attached to an EC2 instance.

### Detailed Explanation

ENIs act as network adapters for EC2 instances and can contain multiple secondary IP addresses. VPC CNI uses these secondary IPs for Pods.

### Production Example

```text
Node

↓

ENI

↓

Secondary IPs

↓

Pods
```

### Follow-Up Questions

- How many ENIs can a node have?
- What are secondary IPs?
- Why are ENIs important in EKS?
- How do you check ENIs?

---

## Why Does Instance Type Affect Pod Count?

### Short Answer

Different instance types support different numbers of ENIs and IP addresses.

### Detailed Explanation

The maximum number of Pods on a node depends on how many IP addresses AWS can allocate through ENIs.

### Production Example

```text
t3.medium

17 Pods
```

```text
m5.xlarge

59 Pods
```

### Follow-Up Questions

- How do you calculate max Pods?
- Why can't Pods share IPs?
- What happens when IPs run out?
- How do you increase Pod density?

---

## What Happens When A Pod Is Created?

### Short Answer

VPC CNI allocates an IP address from available ENIs and assigns it to the Pod.

### Detailed Explanation

After the scheduler selects a node and kubelet creates the Pod, VPC CNI allocates a secondary IP address and configures networking.

### Production Example

```text
Scheduler

↓

Node

↓

kubelet

↓

VPC CNI

↓

Pod IP

↓

Running
```

### Follow-Up Questions

- Which component assigns Pod IPs?
- What happens if no IP is available?
- How does kubelet interact with CNI?
- What is the role of ENIs?

---

## How Do You Troubleshoot IP Exhaustion?

### Short Answer

Check available IPs, subnet size, node capacity, and VPC CNI health.

### Detailed Explanation

Pod creation can fail when ENIs have no available secondary IPs. Investigate node limits, subnet capacity, and VPC CNI status.

### Production Example

Symptoms

```text
Pods Pending
```

Root Cause

```text
No Available Pod IPs
```

### Follow-Up Questions

- How do you check Pod limits?
- How do you scale nodes?
- What logs should be checked?
- How do subnet sizes affect EKS?
```

# Key Takeaways

```text
VPC CNI provides networking for EKS Pods.

Pods receive real VPC IP addresses.

ENIs provide secondary IPs used by Pods.

Instance type determines maximum Pod density.

Pod limits are based on ENI and IP capacity.

VPC CNI runs as aws-node in kube-system.

Pod-to-Pod communication uses native VPC networking.

IP exhaustion is a common production issue.

Subnet size directly impacts Pod scalability.

Understanding VPC CNI is critical for EKS networking troubleshooting.
```