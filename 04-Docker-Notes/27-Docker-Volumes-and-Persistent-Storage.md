# Docker Volumes and Persistent Storage

## Introduction

Containers are designed to be:

```text
Lightweight

Portable

Ephemeral
```

---

Question:

```text
What Happens When
A Container Is Deleted?
```

---

By default:

```text
Container Data Is Lost
```

---

Example

```text
MongoDB Container
      ↓

Store Customer Data
      ↓

Container Deleted
      ↓

Data Lost
```

---

This is a major problem for:

```text
Databases

Message Queues

Caches

Applications Storing Files
```

---

Docker solves this using:

```text
Volumes
```

---

# What is a Volume?

A volume is:

```text
Persistent Storage

Managed By Docker
```

---

Volumes exist:

```text
Outside Container Lifecycle
```

---

Meaning

```text
Container Deleted
      ↓

Volume Remains
      ↓

Data Remains
```

---

# Why Volumes Are Required?

Without Volume

```text
Container
      ↓

Data Written
      ↓

Container Removed
      ↓

Data Lost
```

---

With Volume

```text
Container
      ↓

Data Written
      ↓

Volume
      ↓

Container Removed
      ↓

Data Preserved
```

---

# Container Filesystem

Every container has:

```text
Writable Layer
```

---

Example

```text
MongoDB Container
```

stores data inside container.

---

Delete Container

```bash
docker rm -f mongodb
```

---

Data disappears.

---

Reason

```text
Writable Layer Deleted
```

---

# Persistent Storage Architecture

```text
Container
      ↓

Volume
      ↓

Host Storage
```

---

Container uses volume.

---

Volume stores data permanently.

---

# Types of Docker Volumes

Docker commonly uses:

```text
Anonymous Volumes

Named Volumes
```

---

# Anonymous Volumes

Docker creates volume automatically.

---

Example

```bash
docker run -d \
-v /data/db \
mongo
```

---

Docker generates:

```text
Random Volume Name
```

---

Example

```text
2e7f8c5d9...
```

---

Problem

```text
Hard To Manage

Hard To Identify
```

---

# Named Volumes

Recommended approach.

---

Create Volume

```bash
docker volume create mongo-data
```

---

Run Container

```bash
docker run -d \
-v mongo-data:/data/db \
mongo
```

---

Meaning

```text
Volume Name

mongo-data
```

---

Data stored inside:

```text
mongo-data
```

volume.

---

# Volume Lifecycle

Create Volume

```bash
docker volume create mongo-data
```

---

Verify

```bash
docker volume ls
```

---

Output

```text
mongo-data
```

---

Inspect Volume

```bash
docker volume inspect mongo-data
```

---

Shows

```text
Mount Point

Driver

Metadata
```

---

# Docker Managed Volumes

Docker handles:

```text
Creation

Storage

Cleanup

Management
```

---

Benefits

```text
Simple Administration

Persistent Data
```

---

# MongoDB Example

Without Volume

```bash
docker run -d \
--name mongodb \
mongo
```

---

Store Data

---

Delete Container

```bash
docker rm -f mongodb
```

---

Result

```text
Database Lost
```

---

# MongoDB With Volume

```bash
docker volume create mongo-data
```

---

Run

```bash
docker run -d \
--name mongodb \
-v mongo-data:/data/db \
mongo
```

---

Store Data

---

Delete Container

```bash
docker rm -f mongodb
```

---

Create Again

```bash
docker run -d \
--name mongodb \
-v mongo-data:/data/db \
mongo
```

---

Result

```text
Data Still Exists
```

---

# MySQL Example

```bash
docker volume create mysql-data
```

---

Run

```bash
docker run -d \
--name mysql \
-v mysql-data:/var/lib/mysql \
mysql
```

---

Database files stored in:

```text
mysql-data
```

volume.

---

# RabbitMQ Example

RabbitMQ stores:

```text
Queues

Messages

Metadata
```

---

Run

```bash
docker volume create rabbitmq-data
```

---

```bash
docker run -d \
--name rabbitmq \
-v rabbitmq-data:/var/lib/rabbitmq \
rabbitmq:3
```

---

Messages survive:

```text
Container Restart

Container Recreation
```

---

# Redis Example

Redis can persist:

```text
Cache Data

Snapshots
```

---

Run

```bash
docker volume create redis-data
```

---

```bash
docker run -d \
--name redis \
-v redis-data:/data \
redis
```

---

# Docker Compose Example

```yaml
services:

  mongodb:
    image: mongo

    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```

---

Start

```bash
docker compose up -d
```

---

Docker creates:

```text
mongo-data Volume
```

automatically.

---

# Verify Mounted Volumes

Container

```bash
docker inspect mongodb
```

---

Output

```text
Mounts
```

section.

---

Shows

```text
Volume Name

Container Path
```

---

# Volume Commands

Create

```bash
docker volume create mongo-data
```

---

List

```bash
docker volume ls
```

---

Inspect

```bash
docker volume inspect mongo-data
```

---

Remove

```bash
docker volume rm mongo-data
```

---

Remove Unused Volumes

```bash
docker volume prune
```

---

# Data Flow

```text
Application
      ↓

Container
      ↓

Volume
      ↓

Host Storage
```

---

# Volumes vs Container Storage

| Feature | Container Storage | Volume |
|----------|----------|----------|
| Persistent | No | Yes |
| Survives Container Deletion | No | Yes |
| Managed By Docker | No | Yes |
| Production Usage | No | Yes |

---

# Production Use Cases

## MongoDB

```text
Database Files
```

---

## MySQL

```text
Database Storage
```

---

## RabbitMQ

```text
Messages

Queues
```

---

## Redis

```text
Snapshots

Persistence Files
```

---

# Common Mistakes

## Not Using Volumes

Result

```text
Container Deleted
      ↓

Data Lost
```

---

Very common beginner mistake.

---

## Removing Volumes Accidentally

Command

```bash
docker volume prune
```

---

Danger

```text
Deletes Unused Volumes
```

---

Verify before cleanup.

---

# Production Example

Roboshop

```text
Catalogue
      ↓

MongoDB Volume

Shipping
      ↓

MySQL Volume

Cart
      ↓

Redis Volume

Payment
      ↓

RabbitMQ Volume
```

---

Application containers:

```text
Stateless
```

---

Database containers:

```text
Stateful
```

---

Require persistent storage.

---

# Common Interview Questions

## What is a Docker Volume?

### Short Answer

A Docker-managed persistent storage mechanism that exists independently of containers.

### Detailed Explanation

Volumes allow data to survive container deletion and recreation.

### Production Example

MongoDB storing data in a named volume.

### Follow-Up Questions

- Why are volumes required?
- Where are volumes stored?

---

## Why Are Containers Called Ephemeral?

### Short Answer

Because container data disappears when the container is removed.

### Detailed Explanation

Container writable layers are deleted when containers are deleted.

### Production Example

MongoDB losing data without a volume.

### Follow-Up Questions

- How do you persist data?
- What is a writable layer?

---

## What is the Difference Between Named and Anonymous Volumes?

### Short Answer

Named volumes have user-defined names, while anonymous volumes receive randomly generated names.

### Detailed Explanation

Named volumes are easier to manage and are preferred in production.

### Production Example

mongo-data volume.

### Follow-Up Questions

- Which should be used in production?
- How do you inspect a volume?

---

## How Do You View Docker Volumes?

### Short Answer

Using docker volume ls.

### Detailed Explanation

Docker provides commands to create, inspect, list, and remove volumes.

### Production Example

docker volume inspect mongo-data.

### Follow-Up Questions

- How do you delete volumes?
- How do you identify unused volumes?

---

# Key Takeaways

```text
Containers are ephemeral by default.

Deleting a container deletes its writable layer.

Volumes provide persistent storage.

Volumes exist independently of containers.

Named volumes are preferred in production.

MongoDB, MySQL, Redis, and RabbitMQ commonly use volumes.

Docker manages volume lifecycle.

Volumes can be created, inspected, and removed using Docker commands.

Docker Compose can create and manage volumes automatically.

Persistent storage is essential for stateful applications.
```