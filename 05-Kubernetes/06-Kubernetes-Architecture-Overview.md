# Kubernetes Architecture Overview

## Introduction

Kubernetes is a container orchestration platform used to manage containerized applications at scale.

It provides:

```text
Auto Scaling

Self Healing

Load Balancing

Service Discovery

Rolling Updates

High Availability
```

---

To provide these capabilities, Kubernetes follows a distributed architecture.

Understanding Kubernetes architecture is essential for:

```text
Cluster Administration

Troubleshooting

Production Support

Interview Preparation
```

---

# Kubernetes Cluster Architecture

A Kubernetes cluster consists of:

```text
Control Plane

Worker Nodes
```

---

Architecture

```text
+----------------------+
|    Control Plane     |
+----------+-----------+
           |
           |
  --------------------
  |        |         |
  v        v         v

+------+ +------+ +------+
|Node1 | |Node2 | |Node3 |
+------+ +------+ +------+

Worker Nodes
```

---

# Components of Kubernetes

```text
Control Plane
    |
    |-- API Server
    |-- ETCD
    |-- Scheduler
    |-- Controller Manager

Worker Nodes
    |
    |-- Kubelet
    |-- Kube Proxy
    |-- Container Runtime
    |-- Pods
```

---

# What is Control Plane?

Control Plane is the brain of Kubernetes.

---

Responsibilities:

```text
Cluster Management

Scheduling

Decision Making

Desired State Management

Cluster Monitoring
```

---

Without Control Plane:

```text
Cluster Cannot Function
```

---

# Control Plane Components

```text
API Server

ETCD

Scheduler

Controller Manager
```

---

# API Server

## What is API Server?

API Server is the entry point of Kubernetes.

---

All requests go through:

```text
Kubernetes API Server
```

---

Architecture

```text
kubectl
     ↓

API Server
```

---

Example

```bash
kubectl get pods
```

---

Flow

```text
kubectl
      ↓

API Server
      ↓

Return Response
```

---

Responsibilities

```text
Authentication

Authorization

Validation

API Processing
```

---

Think of API Server as:

```text
Front Door Of Kubernetes
```

---

# ETCD

## What is ETCD?

ETCD is Kubernetes' database.

---

Stores:

```text
Cluster State

Pods

Nodes

Deployments

Secrets

ConfigMaps
```

---

Architecture

```text
API Server
      ↓

ETCD
```

---

Example

If a pod is created:

```text
Pod Information
```

stored in:

```text
ETCD
```

---

Important

```text
ETCD Is The Source Of Truth
```

---

If ETCD is lost:

```text
Cluster State Lost
```

---

# Scheduler

## What is Scheduler?

Scheduler decides:

```text
Which Node Runs Which Pod
```

---

Example

Cluster

```text
Node-1

Node-2

Node-3
```

---

New Pod Created.

---

Scheduler evaluates:

```text
CPU

Memory

Node Availability

Constraints
```

---

Selects:

```text
Best Node
```

---

Architecture

```text
Pod Request
      ↓

Scheduler
      ↓

Node Selection
```

---

# Controller Manager

## What is Controller Manager?

Controller Manager ensures:

```text
Actual State

Matches

Desired State
```

---

Example

Desired State

```text
3 Pods
```

---

Current State

```text
2 Pods
```

---

Controller detects mismatch.

---

Action

```text
Create Missing Pod
```

---

Responsibilities

```text
Node Controller

Deployment Controller

ReplicaSet Controller

Job Controller
```

---

# Worker Node

Worker Nodes execute workloads.

---

Responsibilities

```text
Run Pods

Run Containers

Process Application Traffic
```

---

Architecture

```text
Worker Node
     |
     |-- Kubelet
     |-- Kube Proxy
     |-- Container Runtime
     |-- Pods
```

---

# Kubelet

## What is Kubelet?

Kubelet is an agent running on every worker node.

---

Responsibilities

```text
Receive Instructions

Create Pods

Monitor Pods

Report Status
```

---

Communication

```text
Control Plane
      ↓

Kubelet
      ↓

Pods
```

---

Example

Scheduler assigns pod.

---

Kubelet:

```text
Creates Pod

Starts Containers

Monitors Health
```

---

# Kube Proxy

## What is Kube Proxy?

Kube Proxy manages:

```text
Networking

Traffic Routing

Service Connectivity
```

---

Responsibilities

```text
Pod Communication

Service Communication

Load Balancing
```

---

Example

```text
User Request
      ↓

Service
      ↓

Pod
```

---

Traffic routing handled by:

```text
Kube Proxy
```

---

# Container Runtime

## What is Container Runtime?

Container Runtime executes containers.

---

Examples

```text
containerd

CRI-O
```

---

Earlier Kubernetes used:

```text
Docker
```

---

Current versions commonly use:

```text
containerd
```

---

Responsibilities

```text
Pull Images

Create Containers

Run Containers

Stop Containers
```

---

# Pods

## What is a Pod?

Pod is the smallest deployable unit in Kubernetes.

---

A Pod contains:

```text
One Or More Containers
```

---

Architecture

```text
Pod
 |
 |-- Container
```

---

Example

```text
Frontend Pod
```

contains:

```text
Frontend Container
```

---

# Complete Kubernetes Request Flow

Suppose user executes:

```bash
kubectl apply -f deployment.yaml
```

---

Step 1

```text
kubectl Sends Request
```

---

Step 2

```text
API Server Receives Request
```

---

Step 3

```text
Store Desired State In ETCD
```

---

Step 4

```text
Scheduler Selects Node
```

---

Step 5

```text
Kubelet Creates Pod
```

---

Step 6

```text
Container Runtime Starts Container
```

---

Final Result

```text
Application Running
```

---

# Complete Architecture Diagram

```text
                    kubectl
                        |
                        v

                +---------------+
                |  API Server   |
                +-------+-------+
                        |
        --------------------------------
        |              |               |
        v              v               v

     ETCD        Scheduler     Controller Manager

                        |
                        |
        ---------------------------------
        |                               |
        v                               v

    Worker Node 1                 Worker Node 2

    +-----------+                 +-----------+
    | Kubelet   |                 | Kubelet   |
    | KubeProxy |                 | KubeProxy |
    | Runtime   |                 | Runtime   |
    | Pods      |                 | Pods      |
    +-----------+                 +-----------+
```

---

# Kubernetes Components Summary

| Component | Purpose |
|------------|----------|
| API Server | Entry Point |
| ETCD | Cluster Database |
| Scheduler | Assign Pods To Nodes |
| Controller Manager | Maintains Desired State |
| Kubelet | Node Agent |
| Kube Proxy | Networking |
| Container Runtime | Runs Containers |
| Pod | Smallest Deployable Unit |

---

# Production Example (EKS)

Architecture

```text
AWS EKS
      |
      |-- Control Plane (Managed By AWS)
      |
      |-- Worker Nodes
              |
              |-- Kubelet
              |-- Kube Proxy
              |-- Pods
```

---

As DevOps engineers we usually manage:

```text
Worker Nodes

Pods

Deployments

Services
```

---

AWS manages:

```text
API Server

ETCD

Scheduler

Controller Manager
```

---

# Common Interview Questions

## What are the main components of Kubernetes Architecture?

### Short Answer

Control Plane and Worker Nodes.

### Detailed Explanation

Control Plane manages the cluster, while Worker Nodes run application workloads.

### Production Example

AWS EKS managed control plane with EC2 worker nodes.

### Follow-Up Questions

- What is API Server?
- What is ETCD?

---

## What is API Server?

### Short Answer

API Server is the entry point of Kubernetes.

### Detailed Explanation

All requests from kubectl, controllers, and nodes pass through the API Server.

### Production Example

Creating deployments using kubectl apply.

### Follow-Up Questions

- Does kubectl talk directly to ETCD?
- How does authentication happen?

---

## What is ETCD?

### Short Answer

ETCD is Kubernetes' key-value database.

### Detailed Explanation

It stores the complete cluster state and acts as the source of truth.

### Production Example

Storing deployment, pod, and secret information.

### Follow-Up Questions

- Why is ETCD important?
- What happens if ETCD is unavailable?

---

## What is Kubelet?

### Short Answer

Kubelet is the agent running on every worker node.

### Detailed Explanation

It receives instructions from the control plane and manages pod lifecycle.

### Production Example

Starting containers on EKS worker nodes.

### Follow-Up Questions

- Does Kubelet create pods?
- How does Kubelet communicate with API Server?

---

# Key Takeaways

```text
Kubernetes consists of Control Plane and Worker Nodes.

API Server is the entry point to Kubernetes.

ETCD stores the cluster state.

Scheduler assigns pods to nodes.

Controller Manager maintains desired state.

Kubelet manages pods on worker nodes.

Kube Proxy handles networking.

Container Runtime runs containers.

Pods are the smallest deployable unit.

Understanding Kubernetes architecture is critical for administration and troubleshooting.
```