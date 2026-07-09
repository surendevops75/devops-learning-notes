# Docker Compose Introduction

## Introduction

As applications grow, running containers individually becomes difficult.

Example:

```text
Frontend

Catalogue

User

Cart

Shipping

Payment

MongoDB

MySQL

Redis

RabbitMQ
```

---

Without Docker Compose:

```bash
docker run ...

docker run ...

docker run ...

docker run ...
```

---

Managing multiple containers manually becomes:

```text
Complex

Time Consuming

Error Prone
```

---

Docker provides:

```text
Docker Compose
```

to solve this problem.

---

# What is Docker Compose?

Docker Compose is:

```text
A YAML Based Tool
```

used to define and manage:

```text
Multiple Containers

Networks

Volumes

Dependencies
```

using a single file.

---

Instead of:

```bash
docker run

docker run

docker run
```

we define everything in:

```text
docker-compose.yml
```

and start all services together.

---

# Why Docker Compose?

Suppose Roboshop contains:

```text
Frontend

Catalogue

User

Cart

MongoDB

RabbitMQ

Redis

MySQL
```

---

Without Compose:

```text
Create Network

Start MongoDB

Start Redis

Start RabbitMQ

Start Catalogue

Start User

Start Cart

Start Frontend
```

---

Many commands.

---

Docker Compose allows:

```bash
docker compose up
```

---

Everything starts automatically.

---

# Traditional Approach

Create Network

```bash
docker network create roboshop
```

---

Run RabbitMQ

```bash
docker run -d \
--name rabbitmq \
--network roboshop \
rabbitmq:3
```

---

Run Catalogue

```bash
docker run -d \
--name catalogue \
--network roboshop \
catalogue:v1
```

---

Run Frontend

```bash
docker run -d \
--name frontend \
--network roboshop \
-p 80:80 \
frontend:v1
```

---

Many commands.

---

# Docker Compose Approach

Single File

```yaml
services:
  rabbitmq:
    image: rabbitmq:3

  catalogue:
    image: catalogue:v1

  frontend:
    image: frontend:v1
```

---

Start Everything

```bash
docker compose up
```

---

# Docker Compose File

Default Name

```text
docker-compose.yml
```

---

Compose reads:

```text
Services

Networks

Volumes

Dependencies
```

from this file.

---

# Basic Structure

```yaml
services:
```

All containers are defined under:

```yaml
services
```

---

Example

```yaml
services:

  nginx:
    image: nginx

  redis:
    image: redis
```

---

Docker creates:

```text
Nginx Container

Redis Container
```

---

# First Compose File

```yaml
services:

  nginx:
    image: nginx
```

---

Start

```bash
docker compose up
```

---

Result

```text
Nginx Container Started
```

---

# Running In Background

```bash
docker compose up -d
```

---

Meaning

```text
Detached Mode
```

---

Similar To

```bash
docker run -d
```

---

# Stopping Containers

```bash
docker compose down
```

---

Docker removes:

```text
Containers

Networks
```

created by compose.

---

# Checking Containers

```bash
docker compose ps
```

---

Shows

```text
Running Services
```

---

# Viewing Logs

All Services

```bash
docker compose logs
```

---

Specific Service

```bash
docker compose logs frontend
```

---

# Service Definition

Example

```yaml
services:

  frontend:
    image: frontend:v1
```

---

Meaning

```text
Create Container

Using frontend:v1 Image
```

---

# Port Mapping

Example

```yaml
services:

  frontend:
    image: frontend:v1

    ports:
      - "80:80"
```

---

Equivalent

```bash
docker run -p 80:80
```

---

# Environment Variables

Example

```yaml
services:

  rabbitmq:
    image: rabbitmq:3

    environment:
      RABBITMQ_DEFAULT_USER: roboshop

      RABBITMQ_DEFAULT_PASS: roboshop123
```

---

Equivalent

```bash
-e RABBITMQ_DEFAULT_USER=roboshop

-e RABBITMQ_DEFAULT_PASS=roboshop123
```

---

# Container Names

Example

```yaml
services:

  frontend:
    container_name: frontend
```

---

Result

```text
Container Name

frontend
```

---

# Dependencies

Applications often depend on:

```text
Databases

Message Queues

Caches
```

---

Example

```text
Catalogue
      ↓

MongoDB
```

---

Docker Compose supports:

```yaml
depends_on
```

---

Example

```yaml
services:

  catalogue:
    depends_on:
      - mongodb

  mongodb:
    image: mongo
```

---

Meaning

```text
Start MongoDB First
```

---

# Networks

Compose automatically creates:

```text
Private Network
```

for all services.

---

Example

```text
frontend
      ↓

catalogue
      ↓

mongodb
```

---

Communication happens using:

```text
Service Names
```

---

Example

```text
mongodb

rabbitmq

redis
```

---

No IP addresses needed.

---

# Docker Compose Networking

Example

```yaml
services:

  catalogue:
    image: catalogue:v1

  mongodb:
    image: mongo
```

---

Catalogue can access:

```text
mongodb
```

directly.

---

Docker DNS handles:

```text
Name Resolution
```

automatically.

---

# Roboshop Example

```yaml
services:

  mongodb:
    image: mongo

  rabbitmq:
    image: rabbitmq:3

  catalogue:
    image: catalogue:v1

  frontend:
    image: frontend:v1
```

---

Start Entire Application

```bash
docker compose up -d
```

---

All services start together.

---

# Build Images Using Compose

Instead of:

```text
Prebuilt Images
```

Compose can build images.

---

Example

```yaml
services:

  catalogue:
    build: .
```

---

Equivalent

```bash
docker build .
```

---

# Building Multiple Service Images

In a microservices architecture, each service is usually built independently.

Example

```bash
for i in mongodb mysql catalogue user cart shipping payment frontend
do
    cd $i
    docker build -t $i:v1 .
    cd ..
done
```
---

Compose can then use:

```yaml
services:

  catalogue:
    image: catalogue:v1

  user:
    image: user:v1

  frontend:
    image: frontend:v1
```

---

# Common Commands

Start

```bash
docker compose up
```

---

Start In Background

```bash
docker compose up -d
```

---

Stop

```bash
docker compose down
```

---

View Logs

```bash
docker compose logs
```

---

View Containers

```bash
docker compose ps
```

---

Restart Services

```bash
docker compose restart
```

---

# Architecture

```text
docker-compose.yml
          ↓

Services
          ↓

Networks
          ↓

Containers
          ↓

Application Running
```

---

# Advantages of Docker Compose

```text
Single YAML File

Easy Management

Automatic Networking

Service Discovery

Dependency Handling

Simple Deployment

Developer Friendly
```

---

# Limitations

Docker Compose is mainly used for:

```text
Development

Testing

Small Environments
```

---

Large-scale production deployments usually use:

```text
Kubernetes

EKS

OpenShift
```

---

# Common Interview Questions

## What is Docker Compose?

### Short Answer

Docker Compose is a YAML-based tool used to define and manage multi-container applications.

### Detailed Explanation

It allows multiple containers, networks, and dependencies to be managed through a single configuration file.

### Production Example

Roboshop application with frontend, catalogue, MongoDB, and RabbitMQ.

### Follow-Up Questions

- What file does Compose use?
- How do you start all services?

---

## What is docker-compose.yml?

### Short Answer

A YAML file containing service definitions and configurations.

### Detailed Explanation

Compose reads service, network, volume, and dependency definitions from this file.

### Production Example

Defining frontend, catalogue, and MongoDB services.

### Follow-Up Questions

- What sections can exist in a Compose file?
- How are services defined?

---

## Difference Between docker run and Docker Compose?

### Short Answer

docker run manages individual containers, while Docker Compose manages multiple containers together.

### Detailed Explanation

Compose simplifies multi-container deployments using a single configuration file.

### Production Example

Roboshop deployment using one compose file instead of multiple docker run commands.

### Follow-Up Questions

- When should Compose be used?
- What are Compose advantages?

---

# Key Takeaways

```text
Docker Compose manages multi-container applications.

docker-compose.yml is the default Compose file.

Compose can create containers, networks, and volumes.

docker compose up starts all services.

docker compose down removes services.

Compose automatically provides networking and service discovery.

Services communicate using service names.

depends_on manages startup order.

Compose simplifies application deployment and management.

Docker Compose is widely used in development and testing environments.
```