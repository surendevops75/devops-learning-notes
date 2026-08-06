# Kubernetes Enterprise Handbook

# Chapter 1 - Kubernetes Architecture Fundamentals

Modern applications require

- High Availability
- Scalability
- Self-Healing
- Zero Downtime Deployments
- Automated Rollbacks
- Infrastructure Automation

Managing hundreds or thousands of containers manually is impossible.

Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, networking, and lifecycle management of containerized applications.

Today Kubernetes is the de facto standard for container orchestration and is used by organizations ranging from startups to Fortune 500 enterprises.

---

# What is Kubernetes?

Kubernetes is a container orchestration platform that manages

- Containers
- Scheduling
- Networking
- Scaling
- Service Discovery
- Load Balancing
- Self Healing
- Rolling Updates

Instead of managing individual Docker containers,

Kubernetes manages the desired state of an application.

---

# Why Kubernetes?

Without Kubernetes

```text
Developer

↓

Docker Container

↓

Manual Deployment

↓

Manual Scaling

↓

Manual Recovery
```

Problems

- Manual operations
- Difficult scaling
- No self healing
- Downtime
- Complex networking

---

With Kubernetes

```text
Developer

↓

Docker Image

↓

Kubernetes

↓

Scheduling

↓

Scaling

↓

Self Healing

↓

Production
```

Everything becomes automated.

---

# Kubernetes Features

Kubernetes provides

- Container Scheduling
- Auto Healing
- Horizontal Scaling
- Rolling Updates
- Rollbacks
- Service Discovery
- Load Balancing
- Secret Management
- Storage Orchestration

---

# Kubernetes Architecture

A Kubernetes cluster consists of

```text
Cluster

├── Control Plane

└── Worker Nodes
```

The Control Plane manages the cluster.

Worker Nodes run applications.

---

# High-Level Architecture

```text
Users

↓

Ingress / Load Balancer

↓

Services

↓

Pods

↓

Worker Nodes

↓

Control Plane
```

---

# Cluster Components

Major components include

Control Plane

- API Server
- etcd
- Scheduler
- Controller Manager
- Cloud Controller Manager

Worker Node

- kubelet
- kube-proxy
- Container Runtime

---

# Control Plane

The Control Plane makes all cluster decisions.

Responsibilities

- Scheduling
- Cluster State
- API Management
- Scaling Decisions
- Node Monitoring

Think of it as the brain of Kubernetes.

---

# Worker Node

Worker Nodes execute workloads.

Each worker node contains

```text
Worker Node

├── kubelet

├── kube-proxy

├── Container Runtime

└── Pods
```

---

# Kubernetes API Server

The API Server is the entry point for all Kubernetes operations.

Every command

```bash
kubectl apply

kubectl get

kubectl delete
```

communicates with the API Server.

---

# API Server Responsibilities

- Authentication
- Authorization
- Admission Control
- Resource Validation
- REST API
- Cluster Communication

All components communicate through the API Server.

---

# etcd

etcd is Kubernetes' distributed key-value database.

It stores

- Cluster State
- Configuration
- Secrets
- Deployments
- Services
- Nodes

Architecture

```text
API Server

↓

etcd

↓

Cluster State
```

Without etcd,

the cluster loses its state information.

---

# Scheduler

The Scheduler decides

where Pods should run.

Workflow

```text
New Pod

↓

Scheduler

↓

Best Worker Node
```

Scheduling decisions consider

- CPU
- Memory
- Affinity
- Taints
- Node Labels
- Resource Availability

---

# Controller Manager

Controllers continuously compare

Desired State

vs

Current State.

Example

Desired

```text
3 Pods
```

Current

```text
2 Pods
```

Controller creates one additional Pod.

---

# Common Controllers

- Deployment Controller
- ReplicaSet Controller
- Node Controller
- Job Controller
- Endpoint Controller

Each controller manages a specific resource.

---

# Cloud Controller Manager

Used in cloud environments.

Integrates Kubernetes with

- AWS
- Azure
- GCP

Responsibilities include

- Load Balancers
- Volumes
- Node Management
- Routes

Amazon EKS uses the AWS Cloud Controller Manager.

---

# kubelet

Every worker node runs kubelet.

Responsibilities

- Register Node
- Monitor Pods
- Pull Images
- Start Containers
- Report Health

Workflow

```text
API Server

↓

kubelet

↓

Container Runtime

↓

Pods
```

---

# kube-proxy

kube-proxy manages networking.

Responsibilities

- Service Networking
- Load Balancing
- Packet Forwarding

It enables communication between Pods and Services.

---

# Container Runtime

The container runtime executes containers.

Examples

- containerd
- CRI-O

(Docker Engine is no longer the default runtime in modern Kubernetes.)

---

# Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod contains

- One or More Containers
- Shared Network
- Shared Storage

Example

```text
Pod

├── Application Container

└── Sidecar Container
```

---

# Node

A Node is a worker machine.

It can be

- Physical Server
- Virtual Machine
- Cloud Instance

Each node hosts multiple Pods.

---

# Cluster

A Cluster is a collection of nodes managed by a Control Plane.

```text
Cluster

├── Control Plane

├── Worker Node 1

├── Worker Node 2

└── Worker Node 3
```

---

# Kubernetes Workflow

```text
Developer

↓

kubectl apply

↓

API Server

↓

Scheduler

↓

Worker Node

↓

Pod Created
```

---

# Desired State

Kubernetes works using declarative configuration.

Example

You declare

```text
Replicas = 3
```

Kubernetes ensures

three Pods are always running.

---

# Self Healing

Suppose

```text
Pod

↓

Crash
```

Controller detects failure

↓

Creates New Pod

Applications remain available.

---

# Horizontal Scaling

Instead of

```text
1 Pod
```

Kubernetes scales to

```text
5 Pods

↓

10 Pods

↓

20 Pods
```

based on demand.

---

# Rolling Updates

Deployment

```text
Version 1

↓

Version 2

↓

Zero Downtime
```

Pods are replaced gradually.

---

# Rollback

If deployment fails

```text
Version 2

↓

Rollback

↓

Version 1
```

Recovery is automatic.

---

# Enterprise Architecture

```text
Users

↓

Route53

↓

Application Load Balancer

↓

Ingress Controller

↓

Kubernetes Service

↓

Pods

↓

Amazon RDS

↓

Amazon S3
```

This architecture supports highly available enterprise applications.

---

# Banking Example

```text
Customers

↓

ALB

↓

Payment Service Pods

↓

Aurora Database

↓

SNS Notifications
```

Each component scales independently.

---

# Kubernetes vs Docker

| Docker | Kubernetes |
|----------|------------|
| Container Runtime | Container Orchestrator |
| Single Host | Cluster |
| Manual Scaling | Auto Scaling |
| Manual Recovery | Self Healing |
| Manual Networking | Built-in Networking |

---

# Kubernetes vs Docker Swarm

| Kubernetes | Docker Swarm |
|-------------|--------------|
| Enterprise Standard | Simpler |
| Rich Ecosystem | Smaller Ecosystem |
| Highly Scalable | Medium Scale |
| Advanced Scheduling | Basic Scheduling |
| Large Community | Smaller Community |

---

# Benefits

- High Availability
- Automatic Recovery
- Horizontal Scaling
- Rolling Updates
- Infrastructure Automation
- Efficient Resource Utilization
- Cloud Portability
- Declarative Management

---

# Best Practices

- Keep Control Plane highly available.
- Use multiple Worker Nodes across Availability Zones.
- Deploy stateless workloads whenever possible.
- Monitor cluster health continuously.
- Secure the API Server.
- Backup etcd regularly.
- Use namespaces to isolate workloads.
- Follow the principle of least privilege.

---

# Common Mistakes

- Running production on a single node.
- Exposing the API Server publicly without protection.
- Ignoring etcd backups.
- Running everything in the default namespace.
- Overcommitting CPU and memory.
- Ignoring resource requests and limits.
- Treating Pods as virtual machines.

---

# Interview Questions

## Basic

- What is Kubernetes?
- Why do we need Kubernetes?
- Explain Kubernetes architecture.
- What are the Control Plane components?
- What is a Worker Node?
- What is a Pod?

## Intermediate

- Explain the responsibilities of the API Server.
- What is etcd?
- How does the Scheduler select a node?
- kubelet vs kube-proxy.
- How does Kubernetes achieve self healing?

## Advanced

- Design a highly available Kubernetes cluster for an enterprise banking platform running across multiple Availability Zones.
- Explain how the Control Plane components interact during a deployment.
- A production Kubernetes cluster experiences a worker node failure during peak traffic. Explain how Kubernetes maintains application availability, schedules replacement Pods, updates Service endpoints, and restores the desired state while minimizing customer impact.

---

# Chapter 2 - Pods, ReplicaSets & Deployments (Deep Dive)

Containers are the building blocks of applications,

but Kubernetes does **not** deploy containers directly.

Instead, Kubernetes manages

```text
Container

↓

Pod

↓

ReplicaSet

↓

Deployment
```

Understanding this hierarchy is essential for designing highly available and scalable applications.

---

# Kubernetes Object Hierarchy

```text
Deployment

↓

ReplicaSet

↓

Pod

↓

Container
```

Each layer has a specific responsibility.

---

# What is a Pod?

A Pod is the **smallest deployable unit** in Kubernetes.

It represents one or more tightly coupled containers that share

- Network
- Storage
- Lifecycle

Architecture

```text
Pod

├── Application Container

└── Sidecar Container
```

Pods are ephemeral by design.

---

# Why Pods Instead of Containers?

Without Pods

```text
Application Container

↓

Managed Individually
```

Problems

- Difficult networking
- No shared storage
- No lifecycle management

---

With Pods

```text
Pod

↓

Containers

↓

Shared Network

↓

Shared Storage
```

Containers work together as one unit.

---

# Pod Characteristics

Every Pod has

- One IP Address
- Shared Network Namespace
- Shared Storage Volumes
- One Lifecycle
- One Scheduling Unit

Containers inside a Pod communicate using

```text
localhost
```

---

# Single Container Pod

Most production Pods contain

one container.

Example

```text
Pod

↓

Nginx Container
```

Simple and easy to manage.

---

# Multi-Container Pod

Some applications require

multiple containers.

Example

```text
Pod

├── Java Application

├── Log Collector

└── Monitoring Agent
```

All containers share the same Pod resources.

---

# Sidecar Pattern

A common multi-container pattern.

```text
Pod

├── Main Application

└── Sidecar Container
```

Sidecars perform

- Logging
- Monitoring
- Proxying
- Metrics Collection

without modifying the main application.

---

# Pod Lifecycle

```text
Pending

↓

Running

↓

Succeeded

↓

Failed

↓

Unknown
```

Pods transition through these phases automatically.

---

# Pending State

The Pod has been accepted,

but containers are not yet running.

Possible reasons

- Image Pull
- Scheduling Delay
- Volume Attachment

---

# Running State

The Pod has started successfully.

Applications begin serving traffic.

---

# Succeeded State

All containers completed successfully.

Usually used for

- Jobs
- Batch Processing

---

# Failed State

Containers terminated unexpectedly.

Possible reasons

- Application Crash
- OOMKilled
- Configuration Errors

---

# Unknown State

The Control Plane cannot determine the Pod status.

Usually caused by

- Network Issues
- Node Failures

---

# Pod Scheduling

Workflow

```text
Pod Created

↓

Scheduler

↓

Worker Node

↓

kubelet

↓

Container Runtime

↓

Running Pod
```

Pods are scheduled only once.

---

# Static Pods

Normally,

Pods are managed through the API Server.

Static Pods are managed directly by

```text
kubelet
```

Common use

- Control Plane Components

---

# What is a ReplicaSet?

A ReplicaSet ensures

a specified number of identical Pods are always running.

Example

Desired

```text
3 Pods
```

Current

```text
2 Pods
```

ReplicaSet creates

```text
1 New Pod
```

---

# ReplicaSet Architecture

```text
ReplicaSet

↓

Pod A

↓

Pod B

↓

Pod C
```

If a Pod fails,

ReplicaSet automatically replaces it.

---

# ReplicaSet Responsibilities

- Maintain Replica Count
- Replace Failed Pods
- Monitor Pod Health

ReplicaSets do **not** perform rolling updates.

---

# ReplicaSet Self Healing

Suppose

```text
Pod B

↓

Deleted
```

ReplicaSet detects

```text
Desired = 3

Current = 2
```

Creates

```text
New Pod
```

Application availability remains unchanged.

---

# Scaling ReplicaSets

Example

Before

```text
Replicas = 3
```

After

```text
Replicas = 8
```

ReplicaSet launches

five additional Pods.

---

# What is a Deployment?

Deployments manage

ReplicaSets.

Architecture

```text
Deployment

↓

ReplicaSet

↓

Pods
```

Deployments provide

- Rolling Updates
- Rollbacks
- Scaling
- Version Management

---

# Why Deployments?

Without Deployments

```text
ReplicaSet

↓

Manual Updates
```

Problems

- Downtime
- Difficult Rollbacks

---

With Deployments

```text
Deployment

↓

Rolling Update

↓

Automatic Rollback
```

Updates become automated.

---

# Deployment Workflow

```text
Developer

↓

kubectl apply

↓

Deployment

↓

ReplicaSet

↓

Pods
```

The Deployment controller manages the complete lifecycle.

---

# Rolling Updates

Deployments update Pods gradually.

Example

```text
Version 1

↓

Pod Updated

↓

Pod Updated

↓

Pod Updated

↓

Version 2
```

Users continue accessing the application during updates.

---

# Rollback

Suppose

Version 2 fails.

Deployment performs

```text
Version 2

↓

Rollback

↓

Version 1
```

Previous ReplicaSet becomes active again.

---

# Deployment Strategy

Default strategy

```text
RollingUpdate
```

Alternative

```text
Recreate
```

RollingUpdate minimizes downtime.

---

# Rolling Update Parameters

Important settings

- maxUnavailable
- maxSurge

Example

```text
Replicas = 4

maxUnavailable = 1

maxSurge = 1
```

Only one Pod becomes unavailable during deployment.

---

# Deployment Revisions

Every Deployment maintains

revision history.

Example

```text
Revision 1

↓

Revision 2

↓

Revision 3
```

Rollback can target any previous revision.

---

# Scaling Deployments

Scaling a Deployment

```text
Deployment

↓

ReplicaSet

↓

10 Pods
```

Deployment updates the ReplicaSet automatically.

---

# Pod Labels

Labels organize Kubernetes resources.

Example

```text
app=frontend

env=production

team=devops
```

Labels are simple key-value pairs.

---

# Selectors

ReplicaSets and Services locate Pods using

selectors.

Example

```text
Selector

↓

app=frontend
```

Only matching Pods are selected.

---

# Labels and Selectors

```text
Deployment

↓

Selector

↓

ReplicaSet

↓

Matching Pods
```

Incorrect selectors may result in

- No Pods Managed
- Wrong Pods Selected

---

# Enterprise Deployment Flow

```text
GitHub

↓

GitHub Actions

↓

Docker Build

↓

Amazon ECR

↓

Deployment

↓

ReplicaSet

↓

Pods

↓

Production
```

This is a common CI/CD workflow.

---

# Banking Example

```text
Payment Deployment

↓

ReplicaSet

↓

5 Payment Pods

↓

Service

↓

Customers
```

If one Pod fails,

another is created automatically.

---

# Deployment vs ReplicaSet

| Deployment | ReplicaSet |
|------------|------------|
| Manages ReplicaSets | Manages Pods |
| Rolling Updates | No Rolling Updates |
| Rollback Support | No Rollback |
| Version History | No Version History |
| Recommended | Rarely Used Directly |

---

# Pod vs Deployment

| Pod | Deployment |
|-----|------------|
| Smallest Unit | Higher-Level Controller |
| Short-Lived | Long-Term Management |
| Single Instance | Multiple Replicas |
| Manual | Automated |

---

# Benefits

- Automatic Recovery
- Zero-Downtime Updates
- Horizontal Scaling
- Version Control
- Rollback Support
- Declarative Configuration

---

# Best Practices

- Use Deployments instead of standalone Pods.
- Avoid creating ReplicaSets manually.
- Keep Pods stateless whenever possible.
- Use meaningful labels and selectors.
- Configure readiness and liveness probes.
- Set CPU and memory requests/limits.
- Use rolling updates for production deployments.
- Maintain Deployment revision history.

---

# Common Mistakes

- Deploying standalone Pods in production.
- Managing ReplicaSets manually.
- Using incorrect label selectors.
- Running multiple unrelated applications in one Pod.
- Forgetting resource requests and limits.
- Using the Recreate strategy for production unnecessarily.
- Deleting Pods manually instead of updating Deployments.

---

# Interview Questions

## Basic

- What is a Pod?
- Why does Kubernetes use Pods instead of containers?
- What is a ReplicaSet?
- What is a Deployment?

## Intermediate

- Deployment vs ReplicaSet.
- Single-container vs Multi-container Pods.
- Explain the Sidecar pattern.
- How does a ReplicaSet provide self-healing?
- Explain Rolling Updates and Rollbacks.

## Advanced

- Design a zero-downtime deployment strategy for a payment application using Deployments, ReplicaSets, and rolling updates.
- Explain what happens internally when `kubectl apply` updates a Deployment from version 1 to version 2.
- A production Deployment running 50 Pods experiences multiple Pod failures during peak traffic. Explain how Kubernetes maintains availability, creates replacement Pods, updates ReplicaSets, manages Service endpoints, and completes the deployment without impacting customers.

---

