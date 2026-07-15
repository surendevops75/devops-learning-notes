# Kubernetes Node Components and Add-ons

## Introduction

After the Control Plane decides:

```text
Where A Pod Should Run
```

the responsibility moves to:

```text
Worker Nodes
```

---

Worker Nodes are responsible for:

```text
Running Pods

Managing Containers

Networking

Storage Access

Node Health Reporting
```

---

Without Worker Nodes:

```text
Applications Cannot Run
```

---

# Kubernetes Architecture

```text
Control Plane

↓

Scheduler

↓

Worker Node

↓

kubelet

↓

Container Runtime

↓

Pod Running
```

---

# What Exists Inside A Worker Node?

Every Kubernetes Worker Node contains:

```text
kubelet

kube-proxy

Container Runtime
```

---

Additionally clusters use add-ons:

```text
CoreDNS

VPC CNI

Metrics Server

EBS CSI

EFS CSI

Pod Identity Agent
```

---

# Worker Node Architecture

```text
Worker Node

├── kubelet

├── kube-proxy

├── containerd

├── VPC CNI

├── CoreDNS

├── Metrics Server

├── EBS CSI

├── EFS CSI

└── Pod Identity Agent
```

---

# kubelet

## What Is kubelet?

kubelet is:

```text
Node Agent
```

running on every worker node.

---

Think of kubelet as:

```text
Manager Of The Node
```

---

# Responsibilities Of kubelet

```text
Communicate With API Server

Watch Assigned Pods

Create Pods

Monitor Containers

Report Node Health
```

---

# Example

Scheduler Decides:

```text
Catalogue Pod

↓

Node-2
```

---

API Server Updates:

```text
nodeName=Node-2
```

---

Node-2 kubelet notices:

```text
Pod Assigned To Me
```

---

kubelet starts Pod creation process.

---

# kubelet Workflow

```text
API Server

↓

Pod Assigned

↓

kubelet Reads Pod Spec

↓

Container Runtime

↓

Create Container

↓

Pod Running
```

---

# Node Health Reporting

kubelet periodically reports:

```text
Node Ready

Node NotReady

Node Resources

Pod Status
```

to API Server.

---

# If kubelet Stops?

Result

```text
Node Becomes NotReady
```

---

Applications may eventually be rescheduled.

---

# Verify kubelet

```bash
systemctl status kubelet
```

---

# Container Runtime

## What Is Container Runtime?

Container Runtime is responsible for:

```text
Pulling Images

Creating Containers

Starting Containers

Stopping Containers
```

---

# Examples

```text
containerd

CRI-O
```

---

EKS Default

```text
containerd
```

---

# Container Runtime Workflow

Pod Assigned

↓

```text
kubelet
```

↓

Container Runtime

↓

```text
Pull Image
```

↓

```text
Create Container
```

↓

```text
Start Container
```

---

# Example

Image

```text
catalogue:v5
```

---

Runtime Pulls

```text
ECR

Docker Hub

Private Registry
```

---

Then Creates:

```text
Running Container
```

---

# If Image Pull Fails?

Pod Status

```text
ImagePullBackOff
```

---

Common Causes

```text
Wrong Image

Registry Access Issue

Network Issue
```

---

# kube-proxy

## What Is kube-proxy?

kube-proxy provides:

```text
Service Networking
```

---

Purpose

```text
Service To Pod Communication
```

---

Without kube-proxy:

```text
Service Cannot Reach Pods
```

---

# Example

Service

```text
catalogue
```

---

ClusterIP

```text
172.20.1.10
```

---

Pods

```text
10.0.1.25

10.0.2.15

10.0.3.11
```

---

kube-proxy creates rules:

```text
ClusterIP

↓

Pod IPs
```

---

# kube-proxy Workflow

```text
Service

↓

EndpointSlice

↓

kube-proxy

↓

iptables/ipvs Rules

↓

Pod
```

---

# Verify kube-proxy

```bash
kubectl get pods -n kube-system
```

---

Output

```text
kube-proxy
```

---

# CoreDNS

## What Is CoreDNS?

CoreDNS provides:

```text
DNS Resolution
```

inside Kubernetes.

---

Without CoreDNS

```text
Pods Cannot Resolve Services
```

---

# Example

Application

```text
catalogue
```

---

Needs Database

```text
mongodb
```

---

Instead Of

```text
10.0.1.25
```

application uses:

```text
mongodb
```

---

CoreDNS Resolves:

```text
mongodb

↓

ClusterIP
```

---

# CoreDNS Workflow

```text
Application

↓

DNS Query

↓

CoreDNS

↓

Service IP

↓

Database
```

---

# Verify CoreDNS

```bash
kubectl get pods -n kube-system
```

---

Output

```text
coredns
```

---

# AWS VPC CNI

## What Is VPC CNI?

Provides:

```text
Pod Networking
```

in EKS.

---

Responsibilities

```text
Assign Pod IPs

Configure Networking

Manage ENIs
```

---

# Workflow

```text
Pod Created

↓

VPC CNI

↓

Assign IP

↓

Pod Running
```

---

Example

```text
10.0.1.25
```

assigned from:

```text
VPC CIDR
```

---

# Verify

```bash
kubectl get pods -n kube-system
```

---

Output

```text
aws-node
```

---

# Metrics Server

## What Is Metrics Server?

Provides:

```text
CPU Metrics

Memory Metrics
```

for Kubernetes.

---

Used By

```text
HPA

kubectl top
```

---

# Example

```bash
kubectl top pods
```

---

Output

```text
CPU

Memory
```

---

# Without Metrics Server

HPA Cannot Work.

---

# EBS CSI Driver

## What Is EBS CSI?

Provides:

```text
Persistent Storage

Using AWS EBS
```

---

Used For

```text
MySQL

MongoDB

PostgreSQL
```

---

# Storage Workflow

```text
PVC

↓

StorageClass

↓

EBS CSI

↓

EBS Volume
```

---

# Example

Database

```text
MongoDB
```

needs:

```text
Persistent Storage
```

---

EBS CSI creates:

```text
EBS Volume
```

automatically.

---

# EFS CSI Driver

## What Is EFS CSI?

Provides:

```text
Shared Storage
```

across nodes.

---

Used For

```text
Shared Files

Uploads

Logs

Reports
```

---

# Difference

EBS

```text
Single Node
```

---

EFS

```text
Multiple Nodes
```

---

# Example

```text
Pod-1

↓

EFS

↓

Pod-2
```

---

Shared Access.

---

# EKS Pod Identity Agent

## What Is Pod Identity Agent?

Provides:

```text
IRSA

Pod Identity
```

integration.

---

Allows Pods To Access:

```text
Secrets Manager

Parameter Store

S3

DynamoDB
```

---

Without Access Keys.

---

# Example

```text
Pod

↓

Service Account

↓

IAM Role

↓

Secrets Manager
```

---

# Production Add-on Architecture

```text
Application Pod

├── CoreDNS

├── VPC CNI

├── kube-proxy

├── Metrics Server

├── EBS CSI

├── EFS CSI

└── Pod Identity Agent
```

---

# Complete Worker Node Flow

```text
Scheduler

↓

Node Selected

↓

kubelet

↓

Container Runtime

↓

Image Pulled

↓

VPC CNI

↓

Pod IP Assigned

↓

CoreDNS

↓

DNS Resolution

↓

kube-proxy

↓

Service Communication

↓

Pod Running
```

---

# Common Production Problems

## Node NotReady

Cause

```text
kubelet Failure
```

---

Verify

```bash
kubectl get nodes
```

---

# ImagePullBackOff

Cause

```text
Image Not Found

Registry Issue
```

---

# Service Not Working

Cause

```text
kube-proxy Issue
```

---

# DNS Failure

Cause

```text
CoreDNS Issue
```

---

Symptoms

```text
Unable To Resolve Service Names
```

---

# HPA Not Scaling

Cause

```text
Metrics Server Missing
```

---

# PVC Pending

Cause

```text
CSI Driver Issue
```

---

# Troubleshooting Commands

Nodes

```bash
kubectl get nodes
```

---

Node Details

```bash
kubectl describe node
```

---

Pods

```bash
kubectl get pods -A
```

---

Metrics

```bash
kubectl top nodes
```

---

DNS

```bash
kubectl get pods -n kube-system
```

---

Storage

```bash
kubectl get pvc
```

---

# Common Interview Questions

## What Is kubelet?

### Short Answer

kubelet is the node agent responsible for managing Pods on a worker node.

### Detailed Explanation

kubelet communicates with the API Server, watches assigned Pods, instructs the container runtime to create containers, and reports node health.

### Production Example

```text
Scheduler

↓

Node-2

↓

kubelet

↓

Container Created
```

### Follow-Up Questions

- Does kubelet schedule Pods?
- How does kubelet communicate with API Server?
- What happens if kubelet fails?
- How does kubelet work with containerd?

---

## What Is kube-proxy?

### Short Answer

kube-proxy provides Service networking.

### Detailed Explanation

It watches Services and EndpointSlices and creates networking rules to route traffic to backend Pods.

### Production Example

```text
Service

↓

kube-proxy

↓

Pod
```

### Follow-Up Questions

- What is ClusterIP?
- What is EndpointSlice?
- Does kube-proxy load balance?
- What happens if kube-proxy fails?

---

## What Is CoreDNS?

### Short Answer

CoreDNS provides DNS resolution inside Kubernetes.

### Detailed Explanation

Applications use service names instead of IP addresses. CoreDNS translates service names into ClusterIP addresses.

### Production Example

```text
mongodb

↓

10.0.1.25
```

### Follow-Up Questions

- How do Pods resolve service names?
- What happens if CoreDNS fails?
- How do you troubleshoot DNS?
- Where does CoreDNS run?

---

## What Is Container Runtime?

### Short Answer

Container Runtime creates and manages containers.

### Detailed Explanation

It pulls container images, creates containers, and runs application processes.

### Production Example

```text
kubelet

↓

containerd

↓

Container Running
```

### Follow-Up Questions

- What is containerd?
- What replaced Docker in Kubernetes?
- Who pulls images?
- What causes ImagePullBackOff?

---

## What Is EBS CSI Driver?

### Short Answer

EBS CSI Driver provisions EBS volumes for Kubernetes workloads.

### Detailed Explanation

It allows PersistentVolumeClaims to dynamically create EBS volumes in AWS.

### Production Example

```text
PVC

↓

EBS CSI

↓

EBS Volume
```

### Follow-Up Questions

- What is CSI?
- What is dynamic provisioning?
- Which workloads use EBS?
- Why is EBS not shared?

---

# Key Takeaways

```text
Worker Nodes run application workloads.

kubelet is the node agent.

Container Runtime creates and manages containers.

kube-proxy provides Service networking.

CoreDNS provides DNS resolution.

VPC CNI assigns Pod IP addresses.

Metrics Server powers HPA and kubectl top.

EBS CSI provides persistent storage.

EFS CSI provides shared storage.

Pod Identity Agent enables secure AWS access.

All worker node components work together to run applications successfully.
```