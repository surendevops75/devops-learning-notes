# Kubernetes Control Plane Architecture

## Introduction

Kubernetes consists of two major parts:

```text
Control Plane

Worker Nodes
```

---

The Control Plane is:

```text
Brain Of Kubernetes
```

---

It is responsible for:

```text
Receiving Requests

Storing Cluster State

Scheduling Pods

Maintaining Desired State

Managing Cluster Health
```

---

Without Control Plane:

```text
Pods Cannot Be Created

Pods Cannot Be Scheduled

Cluster Cannot Function
```

---

# High Level Architecture

```text
User

↓

kubectl

↓

API Server

↓

etcd

↓

Scheduler

↓

Controller Manager

↓

Worker Nodes
```

---

# What Is The Control Plane?

Control Plane Components:

```text
API Server

Scheduler

Controller Manager

etcd
```

---

Each Component Has A Specific Responsibility.

---

# Control Plane Architecture

```text
                API Server
                     │
       ┌─────────────┼─────────────┐
       │             │             │
       ▼             ▼             ▼
   Scheduler    Controller      etcd
                  Manager
```

---

# API Server

## What Is API Server?

API Server is:

```text
Entry Point

Of Kubernetes
```

---

Every Request First Reaches:

```text
API Server
```

---

Examples

```bash
kubectl get pods

kubectl apply -f app.yaml

kubectl delete pod nginx
```

---

Flow

```text
User

↓

API Server
```

---

# Responsibilities Of API Server

```text
Authentication

Authorization

Admission Control

Validation

Communication

State Updates
```

---

# Authentication

API Server Verifies:

```text
Who Are You?
```

---

EKS Example

```text
IAM User

IAM Role

aws-auth
```

---

Example

```text
surendra
```

---

API Server Validates:

```text
Valid User?
```

---

# Authorization

After Authentication

API Server Checks:

```text
What Can You Do?
```

---

Uses

```text
Role

RoleBinding

ClusterRole

ClusterRoleBinding
```

---

Example

```text
Can Surendra

Create Deployment?
```

---

# Request Validation

API Server Validates:

```text
YAML Syntax

Required Fields

API Version

Resource Type
```

---

Example

```yaml
kind: Deployment
```

---

Invalid Resource

↓

```text
Request Rejected
```

---

# Admission Controllers

Admission Controllers Modify Or Validate Requests.

---

Examples

```text
Resource Quotas

Security Policies

Default Values
```

---

Flow

```text
Request

↓

Authentication

↓

Authorization

↓

Admission Controllers

↓

etcd
```

---

# Why API Server Is Important?

Without API Server:

```text
No Communication

With Kubernetes
```

---

# etcd

## What Is etcd?

etcd is:

```text
Distributed Key Value Database
```

---

Purpose

```text
Store Cluster State
```

---

Everything In Kubernetes Is Stored In:

```text
etcd
```

---

Examples

```text
Pods

Deployments

Secrets

ConfigMaps

Services

Nodes
```

---

# Example

Deployment Created

```text
catalogue

replicas=3
```

---

Stored In

```text
etcd
```

---

# Why etcd Is Important?

Without etcd:

```text
Cluster State Lost
```

---

etcd Is Considered:

```text
Single Source Of Truth
```

---

# Example

Desired State

```text
3 Pods
```

---

Stored In

```text
etcd
```

---

Controller Uses:

```text
etcd
```

to verify desired state.

---

# Scheduler

## What Is Scheduler?

Scheduler Decides:

```text
Which Node

Should Run Pod
```

---

Scheduler Does NOT:

```text
Create Containers
```

---

Scheduler Only Selects:

```text
Best Node
```

---

# Scheduler Workflow

Pod Created

↓

```text
Pending
```

---

Reason

```text
No Node Assigned
```

---

Scheduler Starts Evaluation.

---

# Scheduler Checks

```text
Available Nodes

CPU

Memory

Node Selectors

Node Affinity

Pod Affinity

Pod AntiAffinity

Taints

Tolerations

TopologySpreadConstraints
```

---

# Example

Cluster

```text
Node-1

Node-2

Node-3
```

---

Scheduler Decision

```text
Pod-1 → Node-1

Pod-2 → Node-2

Pod-3 → Node-3
```

---

# Hard Constraints

Examples

```text
NodeSelector

Required Affinity

Required AntiAffinity
```

---

If Not Satisfied

```text
Pod Remains Pending
```

---

# Scheduler Output

Updates

```text
nodeName
```

field in Pod Spec.

---

Example

```text
nodeName=node-2
```

---

# Controller Manager

## What Is Controller Manager?

Controller Manager Runs:

```text
Multiple Controllers
```

---

Controllers Continuously Watch:

```text
Desired State

vs

Actual State
```

---

# Controller Philosophy

```text
Observe

↓

Compare

↓

Fix
```

---

# Deployment Controller

Example

```text
Deployment

Replicas = 3
```

---

Actual

```text
0 Pods
```

---

Controller Creates

```text
ReplicaSet
```

---

# ReplicaSet Controller

Desired

```text
3 Pods
```

---

Actual

```text
2 Pods
```

---

Action

```text
Create 1 Pod
```

---

# Node Controller

Responsible For

```text
Node Health
```

---

Checks

```text
Ready

NotReady

Unreachable
```

---

Example

```text
Node Lost
```

---

Controller Detects

```text
NotReady
```

---

# EndpointSlice Controller

Responsible For

```text
Service Endpoints
```

---

Example

```text
Catalogue Service
```

---

Discovers

```text
Pod IPs
```

---

Creates

```text
EndpointSlice
```

---

# Job Controller

Responsible For

```text
Batch Jobs
```

---

Examples

```text
Database Migration

Cleanup Jobs

Reports
```

---

Status

```text
Completed
```

---

# StatefulSet Controller

Responsible For

```text
Stateful Applications
```

---

Examples

```text
MongoDB

MySQL

Kafka
```

---

# Controller Manager Architecture

```text
Controller Manager

├── Deployment Controller

├── ReplicaSet Controller

├── Node Controller

├── EndpointSlice Controller

├── Job Controller

└── StatefulSet Controller
```

---

# Complete Request Flow

User

```bash
kubectl apply -f deployment.yaml
```

---

Flow

```text
User

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

Pod Created

↓

Scheduler

↓

Node Selected
```

---

# Control Plane Responsibilities Summary

API Server

```text
Receives Requests
```

---

etcd

```text
Stores State
```

---

Scheduler

```text
Selects Nodes
```

---

Controller Manager

```text
Maintains Desired State
```

---

# Common Production Problems

## API Server Unavailable

Symptoms

```text
kubectl Commands Fail
```

---

Verify

```bash
kubectl cluster-info
```

---

# Authentication Failure

Symptoms

```text
Unauthorized
```

---

Verify

```text
IAM

aws-auth

RBAC
```

---

# Authorization Failure

Symptoms

```text
Forbidden
```

---

Verify

```text
Roles

RoleBindings
```

---

# Pods Pending

Symptoms

```text
Pending
```

---

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

# etcd Issues

Symptoms

```text
Cluster Instability
```

---

Impact

```text
State Cannot Be Stored
```

---

# Troubleshooting Commands

Cluster Info

```bash
kubectl cluster-info
```

---

Nodes

```bash
kubectl get nodes
```

---

Pods

```bash
kubectl get pods -A
```

---

Events

```bash
kubectl get events
```

---

Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

# Common Interview Questions

## What Is Kubernetes Control Plane?

### Short Answer

The Control Plane is the brain of Kubernetes that manages cluster operations and maintains desired state.

### Detailed Explanation

It consists of API Server, Scheduler, Controller Manager, and etcd. Together they receive requests, store state, schedule workloads, and continuously maintain cluster health.

### Production Example

```text
User

↓

API Server

↓

etcd

↓

Scheduler

↓

Controller Manager
```

### Follow-Up Questions

- What are control plane components?
- Can worker nodes run without control plane?
- What happens if API Server fails?
- Why is etcd important?

---

## What Is API Server?

### Short Answer

API Server is the entry point for all Kubernetes operations.

### Detailed Explanation

Every request from users, controllers, schedulers, and nodes passes through the API Server. It performs authentication, authorization, validation, and communication.

### Production Example

```bash
kubectl get pods
```

↓

```text
API Server
```

### Follow-Up Questions

- What is authentication?
- What is authorization?
- What are admission controllers?
- Why is API Server critical?

---

## What Is etcd?

### Short Answer

etcd is Kubernetes' distributed key-value database.

### Detailed Explanation

It stores the entire cluster state, including Pods, Deployments, Services, ConfigMaps, Secrets, and Nodes.

### Production Example

```text
Deployment

catalogue

replicas=3
```

stored in:

```text
etcd
```

### Follow-Up Questions

- Why is etcd called source of truth?
- What happens if etcd fails?
- What data is stored in etcd?
- Is etcd a relational database?

---

## What Does Scheduler Do?

### Short Answer

Scheduler decides which node should run a Pod.

### Detailed Explanation

It evaluates available nodes and scheduling constraints before selecting the best node.

### Production Example

```text
Pod-1 → Node-1

Pod-2 → Node-2

Pod-3 → Node-3
```

### Follow-Up Questions

- Does scheduler create containers?
- What is node affinity?
- What is pod anti-affinity?
- Why do Pods remain Pending?

---

## What Does Controller Manager Do?

### Short Answer

Controller Manager runs controllers that continuously maintain desired state.

### Detailed Explanation

Controllers compare desired and actual states and take corrective actions whenever differences are detected.

### Production Example

```text
Desired = 3 Pods

Actual = 2 Pods

↓

Create 1 Pod
```

### Follow-Up Questions

- What are controllers?
- What is ReplicaSet Controller?
- What is Node Controller?
- What is EndpointSlice Controller?

---

# Key Takeaways

```text
Control Plane is the brain of Kubernetes.

API Server is the entry point for all requests.

Authentication and Authorization happen at API Server.

etcd stores the complete cluster state.

Scheduler selects the best node for Pods.

Controller Manager maintains desired state.

Controllers continuously monitor resources.

Pods remain Pending until Scheduler assigns a node.

etcd is the source of truth for Kubernetes.

Understanding the Control Plane is essential for Kubernetes troubleshooting and interviews.
```