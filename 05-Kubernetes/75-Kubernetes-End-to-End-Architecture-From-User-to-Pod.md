# Kubernetes End-to-End Architecture: From User to Pod

## Introduction

Most Kubernetes engineers know:

```text
API Server

Scheduler

etcd

kubelet

kube-proxy
```

---

But many engineers struggle to explain:

```text
How A Request Actually Travels

From User

To Running Pod
```

---

This file combines:

```text
Control Plane

Worker Nodes

Networking

Storage

Services
```

into one complete flow.

---

# High Level Architecture

```text
User

↓

kubectl

↓

API Server

↓

Authentication

↓

Authorization

↓

etcd

↓

Controller Manager

↓

ReplicaSet

↓

Pod Object

↓

Scheduler

↓

Node Selection

↓

kubelet

↓

Container Runtime

↓

VPC CNI

↓

Pod Running
```

---

# Production Example

User Deploys

```text
Catalogue Service
```

using:

```bash
kubectl apply -f deployment.yaml
```

---

Goal

```text
Run 3 Catalogue Pods
```

---

# Step 1 - User Executes kubectl

Command

```bash
kubectl apply -f deployment.yaml
```

---

kubectl Reads

```text
~/.kube/config
```

---

Contains

```text
Cluster Endpoint

Authentication Information

Certificates
```

---

kubectl Sends Request To

```text
API Server
```

---

# Step 2 - API Server Authentication

Question

```text
Who Are You?
```

---

EKS Example

```text
IAM User

IAM Role
```

---

API Server Verifies

```text
Valid Identity
```

---

# Example

User

```text
surendra
```

---

Authentication

```text
SUCCESS
```

---

# Step 3 - Authorization

Question

```text
What Are You Allowed To Do?
```

---

Checks

```text
Role

RoleBinding

ClusterRole

ClusterRoleBinding
```

---

Example

```text
Can Create Deployment?
```

---

Result

```text
Allowed
```

---

# Step 4 - Request Validation

API Server Verifies

```text
YAML Syntax

Resource Type

API Version
```

---

Example

```yaml
kind: Deployment
```

---

Invalid YAML

↓

```text
Rejected
```

---

# Step 5 - Store Desired State

API Server Stores

```text
Deployment

Replicas = 3
```

inside:

```text
etcd
```

---

Important

```text
No Pod Exists Yet
```

---

Only Desired State Exists.

---

# Step 6 - Deployment Controller

Controller Watches

```text
etcd
```

continuously.

---

Sees

```text
Desired = 3

Actual = 0
```

---

Action

```text
Create ReplicaSet
```

---

# Step 7 - ReplicaSet Controller

ReplicaSet Sees

```text
Desired = 3

Actual = 0
```

---

Creates

```text
Pod-1

Pod-2

Pod-3
```

---

Current Status

```text
Pending
```

---

Reason

```text
No Node Assigned
```

---

# Step 8 - Scheduler

Scheduler Watches

```text
Pending Pods
```

---

Available Nodes

```text
Node-1

Node-2

Node-3
```

---

# Scheduler Evaluation

Checks

```text
CPU

Memory

Node Selector

Node Affinity

Pod Affinity

Pod AntiAffinity

Taints

Tolerations

TopologySpreadConstraints
```

---

# Scheduling Decision

Example

```text
Pod-1 → Node-1

Pod-2 → Node-2

Pod-3 → Node-3
```

---

Scheduler Updates

```text
nodeName
```

field.

---

Example

```text
nodeName=node-2
```

---

# Important

Scheduler Does NOT

```text
Create Containers
```

---

Scheduler Only

```text
Selects Node
```

---

# Step 9 - kubelet Takes Over

Every Worker Node Runs

```text
kubelet
```

---

Node-1 kubelet notices:

```text
Pod Assigned To Me
```

---

Action

```text
Start Pod Creation
```

---

# Step 10 - Container Runtime

kubelet Communicates With

```text
containerd
```

---

Runtime Performs

```text
Pull Image

Create Container

Start Container
```

---

Example

```text
catalogue:v5
```

---

Image Source

```text
Amazon ECR
```

---

# Step 11 - VPC CNI

Container Exists.

---

Needs Networking.

---

VPC CNI Assigns

```text
Pod IP
```

---

Example

```text
10.0.1.25
```

---

IP Comes From

```text
VPC CIDR
```

---

Pod Status

```text
Running
```

---

# Step 12 - kubelet Updates API Server

kubelet Reports

```text
Pod Running
```

---

API Server Updates

```text
etcd
```

---

Now

```bash
kubectl get pods
```

shows:

```text
Running
```

---

# Complete Pod Creation Flow

```text
User

↓

kubectl apply

↓

API Server

↓

Authentication

↓

Authorization

↓

Validation

↓

etcd

↓

Deployment Controller

↓

ReplicaSet Controller

↓

Pod Object

↓

Scheduler

↓

Node Selection

↓

kubelet

↓

containerd

↓

VPC CNI

↓

Pod Running
```

---

# Service Creation Flow

User Creates

```yaml
kind: Service
```

---

API Server Stores

```text
Service Object
```

---

Inside

```text
etcd
```

---

# EndpointSlice Controller

Discovers

```text
Pod IPs
```

---

Example

```text
10.0.1.25

10.0.2.18

10.0.3.11
```

---

Creates

```text
EndpointSlice
```

---

# Service Architecture

```text
Service

↓

EndpointSlice

↓

Pod IPs
```

---

# kube-proxy

kube-proxy Watches

```text
Services

EndpointSlices
```

---

Creates Rules

```text
ClusterIP

↓

Backend Pods
```

---

# Service Traffic Flow

```text
Catalogue Service

↓

kube-proxy

↓

Catalogue Pod
```

---

# DNS Resolution Flow

Applications Usually Don't Use:

```text
IP Addresses
```

---

They Use

```text
Service Names
```

---

Example

```text
mongodb
```

---

# CoreDNS

Application Requests

```text
mongodb
```

---

CoreDNS Resolves

```text
mongodb

↓

ClusterIP
```

---

Traffic Continues

```text
ClusterIP

↓

Pod
```

---

# DNS Architecture

```text
Application

↓

CoreDNS

↓

Service

↓

ClusterIP

↓

Pod
```

---

# Ingress Flow

External User

```text
https://catalogue.company.com
```

---

Request Hits

```text
ALB
```

---

Then

```text
Ingress
```

---

Then

```text
Service
```

---

Then

```text
Pod
```

---

# Complete Ingress Flow

```text
User

↓

Route53

↓

ALB

↓

Ingress

↓

Service

↓

EndpointSlice

↓

Pod
```

---

# Storage Flow

Application Needs Storage

---

Creates

```text
PVC
```

---

PVC Uses

```text
StorageClass
```

---

StorageClass Calls

```text
EBS CSI Driver
```

---

Creates

```text
EBS Volume
```

---

# Storage Architecture

```text
PVC

↓

StorageClass

↓

EBS CSI

↓

EBS Volume

↓

Pod
```

---

# Secrets Flow

Application Needs

```text
MySQL Password
```

---

Uses

```text
IRSA
```

---

Flow

```text
Pod

↓

Service Account

↓

IAM Role

↓

Secrets Manager

↓

Secret Retrieved
```

---

# Complete Application Flow

Catalogue Pod Starts

↓

```text
IRSA
```

↓

```text
Secrets Manager
```

↓

```text
Read Password
```

↓

```text
Connect MongoDB
```

↓

```text
Application Ready
```

---

# Complete Kubernetes Architecture

```text
User

↓

kubectl

↓

API Server

↓

Authentication

↓

Authorization

↓

etcd

↓

Deployment Controller

↓

ReplicaSet Controller

↓

Scheduler

↓

Node Selection

↓

kubelet

↓

containerd

↓

VPC CNI

↓

Pod Running

↓

CoreDNS

↓

Service Discovery

↓

kube-proxy

↓

Service Routing

↓

Application Ready
```

---

# Common Production Problems

## Authentication Failure

Symptoms

```text
Unauthorized
```

---

Check

```text
IAM

RBAC

aws-auth
```

---

# Pods Pending

Cause

```text
Scheduler Cannot Find Node
```

---

Verify

```bash
kubectl describe pod
```

---

# ImagePullBackOff

Cause

```text
Container Runtime

Cannot Pull Image
```

---

# DNS Failure

Cause

```text
CoreDNS Problem
```

---

Symptoms

```text
Cannot Resolve Service Name
```

---

# Service Not Working

Cause

```text
kube-proxy Issue
```

---

# Storage Failure

Cause

```text
CSI Driver Issue
```

---

# Secret Retrieval Failure

Cause

```text
IRSA

IAM Policy

Secrets Manager
```

---

# Troubleshooting Commands

Pods

```bash
kubectl get pods -A
```

---

Services

```bash
kubectl get svc
```

---

Endpoints

```bash
kubectl get endpointslices
```

---

DNS

```bash
kubectl get pods -n kube-system
```

---

Nodes

```bash
kubectl get nodes
```

---

Events

```bash
kubectl get events
```

---

# Common Interview Questions

## Explain Kubernetes Architecture From User To Pod

### Short Answer

A user request reaches the API Server, gets stored in etcd, controllers create Pods, Scheduler selects nodes, kubelet creates containers, VPC CNI assigns networking, and the Pod becomes Running.

### Detailed Explanation

Kubernetes follows a control loop model. The Control Plane receives and stores desired state, controllers create resources, Scheduler chooses nodes, and worker node components run the workload.

### Production Example

```text
User

↓

API Server

↓

etcd

↓

Controllers

↓

Scheduler

↓

kubelet

↓

containerd

↓

VPC CNI

↓

Running Pod
```

### Follow-Up Questions

- What is the role of Scheduler?
- Who creates containers?
- Who assigns Pod IPs?
- Where is desired state stored?

---

## What Happens When kubectl apply Is Executed?

### Short Answer

The request reaches API Server, gets validated, stored in etcd, and processed by controllers.

### Detailed Explanation

The Deployment object is persisted in etcd, controllers create ReplicaSets and Pods, and the Scheduler eventually places Pods on worker nodes.

### Production Example

```bash
kubectl apply -f deployment.yaml
```

↓

```text
API Server

↓

etcd

↓

Controllers

↓

Pods
```

### Follow-Up Questions

- What validates YAML?
- What happens after storage?
- Does Scheduler create Pods?
- Where is Deployment stored?

---

## What Happens When A Service Is Created?

### Short Answer

EndpointSlice Controller discovers Pods and kube-proxy configures routing.

### Detailed Explanation

Services provide stable access to dynamic Pods. EndpointSlices track Pod IPs while kube-proxy creates networking rules.

### Production Example

```text
Service

↓

EndpointSlice

↓

kube-proxy

↓

Pod
```

### Follow-Up Questions

- What is EndpointSlice?
- What is ClusterIP?
- How does Service find Pods?
- Who creates EndpointSlices?

---

## How Does DNS Work In Kubernetes?

### Short Answer

CoreDNS resolves service names into ClusterIP addresses.

### Detailed Explanation

Applications communicate using service names instead of IP addresses. CoreDNS provides service discovery.

### Production Example

```text
mongodb

↓

CoreDNS

↓

ClusterIP

↓

Pod
```

### Follow-Up Questions

- What happens if CoreDNS fails?
- How do Pods resolve names?
- What records does CoreDNS create?
- How do you troubleshoot DNS?

---

## Explain External Traffic Flow To A Pod

### Short Answer

Traffic flows through Route53, ALB, Ingress, Service, and finally reaches Pods.

### Detailed Explanation

Ingress provides Layer-7 routing while Services provide internal load balancing to backend Pods.

### Production Example

```text
User

↓

Route53

↓

ALB

↓

Ingress

↓

Service

↓

Pod
```

### Follow-Up Questions

- What is Ingress?
- Why use ALB?
- What is a Target Group?
- How is traffic routed?
```

---

# Key Takeaways

```text
API Server is the entry point of Kubernetes.

etcd stores desired and actual cluster state.

Controllers create resources based on desired state.

Scheduler selects nodes for Pods.

kubelet manages Pods on worker nodes.

containerd creates and runs containers.

VPC CNI assigns Pod IP addresses.

CoreDNS provides service discovery.

kube-proxy enables Service networking.

EndpointSlices connect Services to Pods.

Ingress manages external traffic.

CSI Drivers provide persistent storage.

IRSA enables secure AWS access.

Understanding the complete user-to-pod flow is essential for production troubleshooting and Kubernetes interviews.
```