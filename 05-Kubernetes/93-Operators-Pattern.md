# Operators Pattern

## Introduction

Kubernetes is excellent at managing:

```text
Pods

Deployments

Services

StatefulSets
```

---

But many real-world applications need much more than that.

Examples:

```text
MongoDB

Kafka

PostgreSQL

Redis

Prometheus
```

---

These applications require:

```text
Installation

Configuration

Scaling

Backup

Restore

Upgrades

Failover
```

---

Traditionally this work is done by:

```text
Database Administrator

System Administrator
```

---

Need automation.

---

This led to:

```text
Operators Pattern
```

---

# Why Operators Were Created?

Suppose we want to run:

```text
MongoDB Cluster
```

inside Kubernetes.

---

Creating only a StatefulSet is not enough.

Need:

```text
Persistent Volumes

Replica Configuration

Backup Jobs

Failover

Monitoring

Version Upgrades
```

---

Without Operators

Engineers perform these tasks manually.

---

Problems

```text
Human Errors

Slow Recovery

Complex Operations
```

---

Need Kubernetes To Manage Applications Like A Human Administrator.

---

This is exactly what Operators do.

---

# What Is An Operator?

An Operator is a Kubernetes application that extends Kubernetes capabilities.

---

Think Of Operator As:

```text
Automated Administrator
```

running inside Kubernetes.

---

Operator continuously watches:

```text
Application State
```

and ensures:

```text
Desired State

=

Actual State
```

---

# Real World Analogy

Traditional Admin

```text
Database Down

↓

Admin Logs In

↓

Fixes Problem
```

---

Operator

```text
Database Down

↓

Operator Detects

↓

Operator Fixes
```

Automatically.

---

# How Operators Work?

Operators use:

```text
CRD

+

Controller
```

---

# CRD

CRD means:

```text
Custom Resource Definition
```

---

Allows Kubernetes To Understand New Resource Types.

---

Normally Kubernetes Knows:

```text
Pod

Deployment

Service
```

---

Operator Adds:

```text
MongoDB

Kafka

PostgreSQL
```

---

as Kubernetes Resources.

---

# Example

Without Operator

```yaml
kind: StatefulSet
```

---

With Operator

```yaml
kind: MongoDB
```

---

Much Simpler.

---

# Controller

Controller Watches Resources.

---

Example

```text
MongoDB Resource Created
```

---

Controller Creates

```text
StatefulSet

PVC

Services

Secrets
```

Automatically.

---

# Operator Architecture

```text
MongoDB CR

↓

Operator

↓

StatefulSet

↓

Pods

↓

PVC
```

---

# Custom Resource Definition (CRD)

Example

```yaml
apiVersion: mongodb.com/v1

kind: MongoDB

metadata:

  name: prod-db

spec:

  replicas: 3

  version: "7.0"
```

---

Kubernetes Normally Doesn't Understand:

```text
MongoDB
```

---

CRD Teaches Kubernetes About It.

---

# Operator Control Loop

Operator Continuously Watches.

---

Example

Desired

```text
3 MongoDB Pods
```

---

Actual

```text
2 MongoDB Pods
```

---

Operator Detects Difference.

---

Creates Missing Pod.

---

Flow

```text
Desired State

↓

Operator

↓

Actual State
```

---

# Production Example

Company Runs

```text
MongoDB Replica Set
```

---

Requirement

```text
3 Nodes
```

---

One Node Crashes.

---

Operator Detects

```text
Replica Missing
```

---

Automatically Creates New Pod.

---

No Manual Intervention.

---

# MongoDB Operator Example

User Creates

```yaml
kind: MongoDB
```

---

Operator Creates

```text
StatefulSet

Services

PVCs

Secrets
```

---

Configures

```text
Replication

Failover
```

Automatically.

---

# Kafka Operator Example

Instead Of

```text
Manually Creating

Kafka Brokers

PVCs

Services
```

---

Create

```yaml
kind: Kafka
```

---

Operator Creates Everything.

---

Popular Example

```text
Strimzi Kafka Operator
```

---

# Prometheus Operator

One Of The Most Common Operators.

---

Without Operator

Need To Manage

```text
Prometheus

AlertManager

ServiceMonitors
```

Manually.

---

With Operator

Create

```yaml
kind: Prometheus
```

---

Everything Managed Automatically.

---

# PostgreSQL Operator

Operator Handles

```text
Replication

Backups

Failover

Upgrades
```

---

Example Operators

```text
CloudNativePG

CrunchyData
```

---

# Operator Lifecycle

## Installation

Install Operator.

---

Operator Registers

```text
CRDs
```

---

## Create Custom Resource

Example

```yaml
kind: Kafka
```

---

## Operator Watches

Detects Resource.

---

## Operator Creates Components

```text
Pods

Services

PVCs
```

---

## Continuous Monitoring

Operator Ensures Desired State.

---

# Production Benefits

## Automation

No Manual Work.

---

## Self Healing

Automatic Recovery.

---

## Standardization

Same Deployment Pattern.

---

## Easier Upgrades

Operator Manages Version Changes.

---

## Easier Backups

Built-in Automation.

---

# Real Production Example

Banking Application

---

Need

```text
PostgreSQL Cluster
```

---

Requirements

```text
HA

Backups

Failover
```

---

Operator Provides

```text
Automatic Management
```

---

# Operators In EKS

Very Common.

---

Examples

```text
AWS Load Balancer Controller

ExternalDNS

Cert Manager

Prometheus Operator

Strimzi Kafka Operator

MongoDB Operator
```

---

Many Engineers Use Operators Daily Without Realizing It.

---

# AWS Load Balancer Controller Is An Operator

You Create

```yaml
kind: Ingress
```

---

Controller Creates

```text
ALB

Target Groups

Listeners
```

Automatically.

---

This follows Operator Pattern.

---

# Cert Manager Is An Operator

Create

```yaml
kind: Certificate
```

---

Operator Creates

```text
TLS Certificates
```

Automatically.

---

# Common Production Problems

## CRD Missing

Operator Installed Incorrectly.

---

Result

```text
Unknown Resource Type
```

---

## Controller Down

Operator Stops Reconciling.

---

Resources Continue Running.

---

But Automation Stops.

---

## Version Mismatch

Operator Version

↓

Application Version

Incompatible.

---

Need Careful Upgrades.

---

# Troubleshooting Commands

Check CRDs

```bash
kubectl get crd
```

---

Check Operator Pods

```bash
kubectl get pods -A
```

---

Check Custom Resources

```bash
kubectl get mongodb
```

Example.

---

Describe Resource

```bash
kubectl describe mongodb prod-db
```

---

Operator Logs

```bash
kubectl logs deployment/operator
```

---

# Common Interview Questions

## What Is An Operator?

### Short Answer

An Operator is a Kubernetes extension that automates application lifecycle management using CRDs and controllers.

### Detailed Explanation

Operators encode operational knowledge into software. They continuously watch application state and automatically perform tasks such as deployment, scaling, backup, failover, and upgrades.

### Production Example

```text
MongoDB Pod Fails

↓

Operator Detects

↓

Operator Recreates Pod
```

### Follow-Up Questions

- What problem do Operators solve?
- Why not use StatefulSets directly?
- What components make up an Operator?

---

## What Is A CRD?

### Short Answer

A CRD (Custom Resource Definition) extends Kubernetes with new resource types.

### Detailed Explanation

CRDs allow Kubernetes to manage resources beyond built-in objects such as Pods and Deployments.

### Production Example

```text
kind: MongoDB

kind: Kafka

kind: Prometheus
```

### Follow-Up Questions

- How does Kubernetes learn new resource types?
- Are CRDs namespaced?
- Who creates CRDs?

---

## How Does An Operator Work?

### Short Answer

Operators use a controller loop to continuously compare desired state and actual state.

### Detailed Explanation

When differences are detected, the Operator takes corrective action automatically.

### Production Example

```text
Desired

3 Replicas

↓

Actual

2 Replicas

↓

Operator Creates 1 Replica
```

### Follow-Up Questions

- What is reconciliation?
- How frequently does reconciliation occur?
- Can Operators perform upgrades?

---

## Why Are Operators Important In Production?

### Short Answer

Operators automate complex operational tasks and reduce manual effort.

### Detailed Explanation

Applications such as databases and messaging systems require backups, failover, upgrades, and scaling. Operators automate these processes and improve reliability.

### Production Example

```text
Primary Database Fails

↓

Operator Promotes Replica

↓

Application Continues
```

### Follow-Up Questions

- Which Operators have you used?
- What is Prometheus Operator?
- What is Strimzi?
- Is AWS Load Balancer Controller an Operator?

---

## Difference Between Controller And Operator?

### Short Answer

Every Operator contains controllers, but not every controller is an Operator.

### Detailed Explanation

Operators extend Kubernetes with domain-specific operational knowledge and custom resources, while controllers simply reconcile Kubernetes resources.

### Production Example

```text
Deployment Controller

↓

Built-In Controller
```

```text
MongoDB Operator

↓

Controller + CRD + Business Logic
```

### Follow-Up Questions

- What additional capabilities do Operators provide?
- Why are CRDs important?
- Can Operators manage external resources?

---

# Key Takeaways

```text
Operators automate application lifecycle management.

Operators use CRDs and controllers.

CRDs extend Kubernetes with new resource types.

Operators continuously reconcile desired state and actual state.

Operators automate backups, failover, upgrades, and scaling.

MongoDB, Kafka, PostgreSQL, and Prometheus commonly use Operators.

AWS Load Balancer Controller and Cert Manager follow the Operator pattern.

Operators are one of the most important concepts in modern Kubernetes platforms.
```