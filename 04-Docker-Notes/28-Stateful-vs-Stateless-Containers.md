# Stateful vs Stateless Containers

## Introduction

One of the most important concepts in:

```text
Docker

Kubernetes

EKS

Microservices
```

is understanding the difference between:

```text
Stateful Applications

Stateless Applications
```

---

This concept directly impacts:

```text
Storage Design

Scaling Strategy

Backup Strategy

Disaster Recovery

High Availability
```

---

# What is State?

State means:

```text
Data

Session Information

Messages

Transactions

Files
```

that an application needs to retain.

---

Example

```text
Customer Records

Orders

Products

Payment Details

Messages
```

---

If this information is important after restart:

```text
Application Has State
```

---

# What is a Stateful Application?

A Stateful application stores:

```text
Business Data

User Data

Messages

Transactions
```

that must survive:

```text
Restart

Upgrade

Container Recreation

Server Failure
```

---

Examples

```text
MongoDB

MySQL

PostgreSQL

Redis

RabbitMQ
```

---

# Stateful Architecture

```text
Application
      ↓

Container
      ↓

Persistent Volume
      ↓

Storage
```

---

If container dies:

```text
Data Still Exists
```

---

# MongoDB Example

Store Customer Data

```json
{
  "name": "John",
  "email": "john@gmail.com"
}
```

---

Container Restart

---

Expected Result

```text
Data Must Remain
```

---

Therefore

```text
MongoDB Is Stateful
```

---

# MySQL Example

Store Orders

```text
OrderID

CustomerID

Products

Payments
```

---

Database Restart

---

Orders should remain.

---

Therefore

```text
MySQL Is Stateful
```

---

# RabbitMQ Example

Stores

```text
Queues

Messages

Exchanges
```

---

Container Restart

---

Messages should survive.

---

Therefore

```text
RabbitMQ Is Stateful
```

---

# Redis Example

Stores

```text
Cache

Snapshots

Session Data
```

---

When persistence is enabled:

```text
Redis Is Stateful
```

---

# Stateful Application Characteristics

```text
Requires Storage

Data Persistence Required

Needs Backup Strategy

Needs Disaster Recovery

Scaling Is More Complex
```

---

# What is a Stateless Application?

Stateless applications store:

```text
Code Only
```

---

They do NOT permanently store:

```text
Business Data

User Data

Transactions
```

---

If container dies:

```text
Replace Container
```

---

Application continues to work.

---

# Stateless Examples

Roboshop

```text
Frontend

Catalogue

User

Cart

Shipping

Payment
```

---

These services primarily contain:

```text
Application Code

Business Logic

APIs
```

---

Data is stored elsewhere.

---

# Stateless Architecture

```text
Application
      ↓

Database
```

---

Application stores nothing locally.

---

Database stores everything.

---

# Frontend Example

Frontend serves:

```text
HTML

CSS

JavaScript
```

---

Container Restart

---

Result

```text
Start New Container
```

---

No data loss.

---

Therefore

```text
Frontend Is Stateless
```

---

# Catalogue Service Example

Stores:

```text
Business Logic
```

---

Product Data stored in:

```text
MongoDB
```

---

Container Restart

---

Start another instance.

---

Application works normally.

---

Therefore

```text
Catalogue Is Stateless
```

---

# Why Stateless Is Preferred?

Stateless applications are easier to:

```text
Deploy

Scale

Replace

Upgrade
```

---

Example

```text
Catalogue
```

---

Current

```text
1 Instance
```

---

Scale

```text
5 Instances
```

---

No data synchronization needed.

---

# Scaling Stateful Applications

Suppose

```text
MySQL
```

---

Scale

```text
5 Instances
```

---

Questions arise:

```text
Who Owns Data?

Replication?

Synchronization?

Consistency?
```

---

Much more complex.

---

# Scaling Stateless Applications

Suppose

```text
Catalogue
```

---

Scale

```text
1 → 10 Pods
```

---

No issues.

---

All instances serve requests.

---

This is why microservices are usually:

```text
Stateless
```

---

# Docker Examples

## Stateless Container

```bash
docker run frontend:v1
```

---

Delete

```bash
docker rm -f frontend
```

---

Create Again

```bash
docker run frontend:v1
```

---

No problem.

---

# Stateful Container

```bash
docker run mongo
```

---

Delete Container

---

Without volume

```text
Data Lost
```

---

Need

```bash
-v mongo-data:/data/db
```

---

# Kubernetes Perspective

Stateless Applications

```text
Deployment
```

resource used.

---

Examples

```text
Frontend

Catalogue

User

Cart
```

---

# Stateful Applications

Kubernetes uses:

```text
StatefulSet
```

---

Examples

```text
MongoDB

MySQL

RabbitMQ

Redis
```

---

Reason

```text
Stable Identity

Persistent Storage

Ordered Deployment
```

---

# Real Roboshop Architecture

```text
Frontend
      ↓

Catalogue
      ↓

MongoDB
```

---

Classification

```text
Frontend → Stateless

Catalogue → Stateless

MongoDB → Stateful
```

---

Another Example

```text
Shipping
      ↓

MySQL
```

---

Classification

```text
Shipping → Stateless

MySQL → Stateful
```

---

# Backup Requirements

Stateless

```text
Git Repository

Container Image
```

is enough.

---

Stateful

Need

```text
Database Backup

Volume Snapshot

Disaster Recovery Plan
```

---

# Upgrade Strategy

Stateless

```text
Delete Old Container

Create New Container
```

---

Easy.

---

Stateful

```text
Backup Data

Upgrade Carefully

Validate Storage
```

---

More complex.

---

# High Availability

Stateless

```text
Multiple Replicas
```

---

Load Balancer distributes traffic.

---

Stateful

Need

```text
Replication

Clustering

Failover
```

---

Examples

```text
Mongo Replica Set

MySQL Replication

RabbitMQ Cluster
```

---

# Comparison Table

| Feature | Stateless | Stateful |
|----------|------------|------------|
| Stores Data | No | Yes |
| Needs Persistent Storage | No | Yes |
| Easy To Scale | Yes | No |
| Easy To Replace | Yes | No |
| Backup Required | Minimal | Yes |
| Kubernetes Resource | Deployment | StatefulSet |
| Examples | Frontend, Catalogue | MongoDB, MySQL |

---

# Production Examples

## Stateless

```text
Frontend

Catalogue

User

Cart

Shipping

Payment

API Services
```

---

## Stateful

```text
MongoDB

MySQL

Redis

RabbitMQ

PostgreSQL
```

---

# Common Interview Questions

## What is a Stateful Application?

### Short Answer

An application that stores data which must survive restarts and upgrades.

### Detailed Explanation

Stateful applications require persistent storage because business data must remain available after container recreation.

### Production Example

MongoDB storing customer information.

### Follow-Up Questions

- Why do stateful applications need volumes?
- How are they deployed in Kubernetes?

---

## What is a Stateless Application?

### Short Answer

An application that does not permanently store business data.

### Detailed Explanation

Stateless applications store only code and can be recreated without affecting application functionality.

### Production Example

Frontend service serving static content.

### Follow-Up Questions

- Why are stateless applications easier to scale?
- Why are Deployments commonly used?

---

## Why Are Databases Stateful?

### Short Answer

Because business data must survive restarts and failures.

### Detailed Explanation

Deleting database data would impact users and application functionality.

### Production Example

MySQL storing order information.

### Follow-Up Questions

- How is persistence achieved?
- What happens without volumes?

---

## Why Are Microservices Usually Stateless?

### Short Answer

Stateless services are easier to scale and manage.

### Detailed Explanation

Business data is stored in dedicated databases while application containers focus only on processing requests.

### Production Example

Catalogue service storing products in MongoDB.

### Follow-Up Questions

- What are the scaling advantages?
- How does Kubernetes handle stateless applications?

---

# Key Takeaways

```text
State represents business data and application information.

Stateful applications require persistent storage.

MongoDB, MySQL, Redis, and RabbitMQ are stateful applications.

Stateless applications primarily contain code and business logic.

Frontend, Catalogue, User, Cart, Shipping, and Payment are stateless services.

Stateless applications are easier to scale and replace.

Stateful applications require backup and disaster recovery strategies.

Docker volumes provide persistence for stateful containers.

Kubernetes Deployments are typically used for stateless workloads.

Kubernetes StatefulSets are typically used for stateful workloads.
```