# Docker Swarm and Container Orchestration

## Introduction

Docker is excellent for:

```text
Building Images

Running Containers

Managing Single Host Deployments
```

---

However, as applications grow:

```text
More Containers

More Servers

More Traffic

High Availability Requirements
```

---

Managing containers manually becomes difficult.

---

This problem is solved using:

```text
Container Orchestration
```

---

# What is Container Orchestration?

Container Orchestration is the process of:

```text
Deploying Containers

Managing Containers

Scaling Containers

Monitoring Containers

Recovering Failed Containers
```

automatically.

---

Without orchestration:

```text
Manual Deployment

Manual Scaling

Manual Recovery

Manual Load Balancing
```

---

With orchestration:

```text
Automated Operations
```

---

# Why Docker Alone Is Not Enough?

Suppose:

```text
Frontend

Catalogue

User

Cart

Shipping

Payment
```

running on one server.

---

Everything works fine.

---

Now traffic increases.

Need:

```text
3 Frontend Containers

3 Catalogue Containers

2 Cart Containers
```

---

Questions:

```text
Which Server Should Run Them?

What If A Container Fails?

How To Scale?

How To Update Containers?

How To Balance Traffic?
```

---

Docker alone does not solve these problems efficiently.

---

# Problems with Docker Standalone

## No Automatic Scheduling

You manually decide:

```text
Which Server

Which Container
```

---

## No Self Healing

Container crashes.

---

Result

```text
Application Down
```

until manually restarted.

---

## Manual Scaling

Need:

```text
10 Containers
```

---

Must start them manually.

---

## Manual Load Balancing

Traffic distribution must be configured separately.

---

## Multi-Host Networking Complexity

Containers running on different servers need communication.

---

Docker standalone has limitations.

---

# What is Docker Swarm?

Docker Swarm is Docker's native:

```text
Container Orchestration Platform
```

---

It converts multiple Docker hosts into:

```text
Single Cluster
```

---

Swarm provides:

```text
High Availability

Scaling

Self Healing

Load Balancing

Service Discovery
```

---

# Docker Swarm Architecture

Components:

```text
Manager Nodes

Worker Nodes
```

---

Architecture

```text
Manager Node
       ↓

Worker Node 1

Worker Node 2

Worker Node 3
```

---

# Manager Node

Manager node controls:

```text
Cluster State

Scheduling

Service Deployment

Scaling

Monitoring
```

---

Responsibilities:

```text
Accept Commands

Manage Cluster

Assign Work
```

---

# Worker Node

Worker nodes run:

```text
Containers

Tasks

Services
```

---

Workers execute instructions from:

```text
Manager Node
```

---

# Swarm Cluster

Example

```text
Manager
      ↓

Worker-1

Worker-2

Worker-3
```

---

Users interact only with:

```text
Manager Node
```

---

# Initialize Swarm

Create Swarm

```bash
docker swarm init
```

---

Output

```text
Swarm Initialized
```

---

Manager node created.

---

# Join Worker Node

Example

```bash
docker swarm join \
--token TOKEN \
MANAGER-IP:2377
```

---

Worker joins cluster.

---

# View Nodes

```bash
docker node ls
```

---

Example Output

```text
Manager

Worker-1

Worker-2
```

---

# What is a Service?

In Swarm:

```text
Containers Are Managed
As Services
```

---

Instead of:

```bash
docker run nginx
```

---

Create Service

```bash
docker service create nginx
```

---

Benefits

```text
Scaling

Self Healing

Rolling Updates
```

---

# Service Example

```bash
docker service create \
--name frontend \
nginx
```

---

Swarm creates:

```text
Frontend Service
```

---

# Tasks

Service creates:

```text
Tasks
```

---

Each task corresponds to:

```text
Container Instance
```

---

Architecture

```text
Service
      ↓

Task
      ↓

Container
```

---

# Scaling Services

Suppose:

```text
Frontend Service
```

currently has:

```text
1 Replica
```

---

Scale

```bash
docker service scale frontend=5
```

---

Result

```text
Frontend-1

Frontend-2

Frontend-3

Frontend-4

Frontend-5
```

---

Automatically distributed across nodes.

---

# Self Healing

Container crashes.

---

Swarm detects:

```text
Replica Missing
```

---

Automatically starts:

```text
New Container
```

---

No manual intervention required.

---

# High Availability

Node failure.

---

Swarm detects:

```text
Node Unavailable
```

---

Containers restarted on:

```text
Healthy Nodes
```

---

Application remains available.

---

# Service Discovery

Services communicate using:

```text
Service Names
```

---

Example

```text
frontend

catalogue

mongodb
```

---

No need to track IP addresses.

---

# Load Balancing

Swarm provides:

```text
Built-in Load Balancing
```

---

Traffic distributed across:

```text
Multiple Replicas
```

---

Example

```text
Request
    ↓

Frontend Service
    ↓

Replica-1

Replica-2

Replica-3
```

---

# Overlay Network

Bridge network works on:

```text
Single Host
```

---

Swarm introduces:

```text
Overlay Network
```

---

Purpose

```text
Multi-Host Container Communication
```

---

Architecture

```text
Node-1
  ↓

Overlay Network
  ↓

Node-2
```

---

Containers communicate across servers.

---

# Rolling Updates

Traditional Update

```text
Stop Old Version

Start New Version
```

---

Downtime occurs.

---

Swarm provides:

```text
Rolling Updates
```

---

Process

```text
Old Container
      ↓

New Container
      ↓

Traffic Shift
```

---

No downtime.

---

# Rollback

Deployment failure.

---

Swarm can:

```text
Rollback
```

to previous version.

---

Benefits

```text
Safer Deployments
```

---

# Service Replicas

Example

```bash
docker service create \
--replicas 3 \
nginx
```

---

Result

```text
3 Running Containers
```

managed automatically.

---

# Common Swarm Commands

Initialize Cluster

```bash
docker swarm init
```

---

Join Worker

```bash
docker swarm join
```

---

List Nodes

```bash
docker node ls
```

---

Create Service

```bash
docker service create
```

---

List Services

```bash
docker service ls
```

---

Scale Service

```bash
docker service scale
```

---

View Tasks

```bash
docker service ps
```

---

# Real Production Example

E-Commerce Application

```text
Frontend
      ↓

Catalogue
      ↓

MongoDB
```

---

Requirements

```text
Multiple Servers

High Availability

Scaling

Load Balancing
```

---

Swarm provides:

```text
Cluster Management

Service Discovery

Automatic Recovery
```

---

# Benefits of Docker Swarm

```text
Easy Setup

Docker Native

Simple Architecture

Built-in Load Balancing

Service Discovery

Scaling

Self Healing
```

---

# Limitations of Docker Swarm

```text
Smaller Ecosystem

Fewer Features

Limited Community Adoption

Less Flexible Than Kubernetes
```

---

Today most enterprises prefer:

```text
Kubernetes
```

---

But Swarm is still useful for:

```text
Learning Orchestration

Small Clusters

Interview Discussions
```

---

# Common Interview Questions

## What is Docker Swarm?

### Short Answer

Docker Swarm is Docker's native container orchestration platform.

### Detailed Explanation

It manages multiple Docker hosts as a single cluster and provides scaling, self-healing, and service discovery.

### Production Example

Running frontend services across multiple servers.

### Follow-Up Questions

- What are manager and worker nodes?
- How does Swarm perform load balancing?

---

## Why Do We Need Docker Swarm?

### Short Answer

To manage containers across multiple servers.

### Detailed Explanation

Swarm automates deployment, scaling, recovery, and networking for containerized applications.

### Production Example

Scaling frontend replicas across worker nodes.

### Follow-Up Questions

- What problems does Swarm solve?
- How does Swarm compare to Docker standalone?

---

## What is a Service in Docker Swarm?

### Short Answer

A service defines the desired state of containers.

### Detailed Explanation

Swarm manages containers through services and ensures the required number of replicas are running.

### Production Example

Frontend service with five replicas.

### Follow-Up Questions

- What is a task?
- How are replicas managed?

---

## What is Self Healing?

### Short Answer

Automatically recreating failed containers.

### Detailed Explanation

Swarm continuously monitors services and restores missing replicas.

### Production Example

Replacing a crashed frontend container automatically.

### Follow-Up Questions

- How does Swarm detect failures?
- Is manual intervention required?

---

# Key Takeaways

```text
Container orchestration automates container management.

Docker standalone has limitations for large-scale deployments.

Docker Swarm is Docker's native orchestration platform.

Swarm clusters consist of manager and worker nodes.

Services are the primary deployment unit in Swarm.

Tasks represent container instances.

Swarm provides scaling and self-healing.

Overlay networks enable multi-host communication.

Built-in load balancing distributes traffic across replicas.

Swarm introduced many concepts later expanded by Kubernetes.
```