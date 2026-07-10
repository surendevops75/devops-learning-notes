# Why Kubernetes Is Needed

## Introduction

Docker solved many problems in application deployment.

It introduced:

```text
Containerization

Portability

Faster Deployments

Efficient Resource Usage
```

---

However, as organizations started running:

```text
Hundreds Of Containers

Thousands Of Containers

Multiple Servers

Microservices Architectures
```

new challenges emerged.

---

These challenges led to the rise of:

```text
Kubernetes
```

---

# What is Kubernetes?

Kubernetes is:

```text
An Open Source Container Orchestration Platform
```

used to:

```text
Deploy Containers

Manage Containers

Scale Containers

Monitor Containers

Recover Failed Containers
```

automatically.

---

Originally developed by:

```text
Google
```

and currently maintained by:

```text
CNCF
(Cloud Native Computing Foundation)
```

---

# Problem 1: Auto Scaling

Suppose:

```text
100 Users
```

Application works normally.

---

Suddenly:

```text
10,000 Users
```

start accessing the application.

---

Need:

```text
More Frontend Containers

More API Containers

More Backend Containers
```

---

Docker:

```text
Manual Scaling
```

---

Kubernetes:

```text
Automatic Scaling
```

---

Architecture

```text
Traffic Increases
        ↓

CPU Usage Increases
        ↓

Kubernetes Detects
        ↓

New Pods Created
```

---

# Problem 2: Self Healing

Suppose:

```text
Catalogue Service Crashes
```

---

Docker:

```text
Manual Intervention Required
```

---

Kubernetes:

```text
Detect Failure
      ↓

Create New Pod
      ↓

Restore Desired State
```

---

Application remains available.

---

# Problem 3: Load Balancing

Suppose:

```text
Frontend-1

Frontend-2

Frontend-3
```

---

Question:

```text
How Is Traffic Distributed?
```

---

Kubernetes provides:

```text
Built-in Load Balancing
```

---

Architecture

```text
User Requests
        ↓

Kubernetes Service
        ↓

Pod-1

Pod-2

Pod-3
```

---

Traffic automatically distributed.

---

# Problem 4: Multiple Host Management

Imagine:

```text
100 Servers

500 Containers

1000 Pods
```

---

Questions:

```text
Which Server Runs Which Application?

How Are Resources Allocated?

How Are Failures Handled?
```

---

Kubernetes automatically schedules workloads.

---

Architecture

```text
Application
       ↓

Kubernetes Scheduler
       ↓

Best Available Node
```

---

# Problem 5: Desired State Management

Suppose:

```text
Need 5 Frontend Pods
```

---

Current State

```text
5 Pods Running
```

---

One Pod Crashes.

---

Current State

```text
4 Pods Running
```

---

Kubernetes detects:

```text
Current State != Desired State
```

---

Automatically creates:

```text
1 New Pod
```

---

Result

```text
5 Pods Running Again
```

---

# Problem 6: Service Discovery

Microservices communicate with each other.

Example

```text
Frontend
     ↓

Catalogue
     ↓

MongoDB
```

---

IP addresses constantly change.

---

Kubernetes provides:

```text
Service Discovery
```

---

Applications communicate using:

```text
Service Names
```

instead of IP addresses.

---

Example

```text
catalogue

mongodb

redis
```

---

# Problem 7: Secret Management

Applications require:

```text
Database Passwords

API Keys

Certificates

Tokens
```

---

Bad Practice

```text
Hardcoding Secrets
```

---

Kubernetes provides:

```text
Secrets
```

---

Benefits

```text
Secure Storage

Controlled Access

Centralized Management
```

---

# Problem 8: Configuration Management

Applications require:

```text
URLs

Hostnames

Ports

Feature Flags
```

---

Kubernetes provides:

```text
ConfigMaps
```

---

Benefits

```text
External Configuration

Environment Independence

Easy Updates
```

---

# Problem 9: Rolling Updates

Current Version

```text
Frontend:v1
```

---

Deploy

```text
Frontend:v2
```

---

Traditional Deployment

```text
Stop Old Version
      ↓

Start New Version
```

---

Downtime occurs.

---

Kubernetes provides:

```text
Rolling Updates
```

---

Process

```text
Pod-v1
      ↓

Pod-v2
      ↓

Traffic Shift
```

---

No downtime.

---

# Problem 10: Rollbacks

Deployment fails.

---

Example

```text
Frontend:v2
```

contains bug.

---

Kubernetes can:

```text
Rollback
```

to:

```text
Frontend:v1
```

---

Quick recovery.

---

# Problem 11: High Availability

Node Failure

---

Without Kubernetes

```text
Application Down
```

---

With Kubernetes

```text
Node Failure
      ↓

Pods Recreated
      ↓

Application Available
```

---

Improved reliability.

---

# Problem 12: Cloud Integration

Modern applications run on:

```text
AWS

Azure

Google Cloud
```

---

Kubernetes integrates with:

```text
EKS

AKS

GKE
```

---

Cloud-native operations become easier.

---

# Kubernetes Architecture at a High Level

```text
Users
      ↓

Kubernetes Cluster
      ↓

Pods
      ↓

Containers
      ↓

Applications
```

---

# Kubernetes Features

```text
Auto Scaling

Self Healing

Load Balancing

Service Discovery

Secrets Management

Configuration Management

Rolling Updates

Rollbacks

High Availability

Cloud Integration
```

---

# Docker vs Kubernetes

| Feature | Docker | Kubernetes |
|----------|----------|----------|
| Run Containers | Yes | Yes |
| Auto Scaling | No | Yes |
| Self Healing | No | Yes |
| Load Balancing | Limited | Yes |
| Service Discovery | Limited | Yes |
| Secrets Management | Basic | Advanced |
| Rolling Updates | Manual | Automated |
| Rollbacks | Manual | Automated |
| Multi Host Management | No | Yes |
| High Availability | Limited | Yes |

---

# Real Production Example

Roboshop Application

```text
Frontend

Catalogue

User

Cart

Shipping

Payment

MongoDB

RabbitMQ
```

---

Requirements

```text
Scale During Peak Traffic

Recover Failed Pods

Load Balance Requests

Secure Secrets

Perform Zero-Downtime Deployments
```

---

Kubernetes handles all these automatically.

---

# Common Interview Questions

## Why is Kubernetes Needed?

### Short Answer

Kubernetes is needed to manage containerized applications at scale.

### Detailed Explanation

Docker provides containerization, while Kubernetes provides orchestration, automation, scaling, recovery, networking, and high availability.

### Production Example

Managing hundreds of microservices across multiple nodes in EKS.

### Follow-Up Questions

- What problems does Kubernetes solve?
- Why is Docker alone not enough?

---

## What is Self Healing?

### Short Answer

Automatically recreating failed pods.

### Detailed Explanation

Kubernetes continuously monitors workloads and restores the desired state.

### Production Example

Replacing a crashed catalogue pod.

### Follow-Up Questions

- Which Kubernetes component performs this?
- How does Kubernetes detect failures?

---

## What is Auto Scaling?

### Short Answer

Automatically increasing or decreasing pods based on resource usage.

### Detailed Explanation

Kubernetes can scale applications based on CPU, memory, or custom metrics.

### Production Example

Scaling frontend pods during high traffic.

### Follow-Up Questions

- What is HPA?
- Which metrics are used?

---

## What is Desired State?

### Short Answer

The state defined by the user that Kubernetes continuously maintains.

### Detailed Explanation

Kubernetes compares actual state with desired state and takes corrective actions when differences exist.

### Production Example

Maintaining five frontend pods even after failures.

### Follow-Up Questions

- Which component maintains desired state?
- What happens if a pod crashes?

---

# Key Takeaways

```text
Kubernetes is a container orchestration platform.

Docker provides containerization, Kubernetes provides orchestration.

Kubernetes supports auto scaling.

Kubernetes provides self healing.

Kubernetes includes built-in load balancing.

Kubernetes supports service discovery.

Kubernetes manages secrets and configuration.

Rolling updates reduce deployment downtime.

Rollbacks enable fast recovery.

Kubernetes maintains desired state automatically.

Kubernetes is the industry standard for running containerized applications at scale.
```