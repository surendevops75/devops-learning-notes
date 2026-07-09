# Container-to-Container Communication

## Introduction

In real-world applications:

```text
Frontend

Catalogue

Cart

User

Shipping

Payment

RabbitMQ

Redis

MongoDB

MySQL
```

all run as separate containers.

---

Question:

```text
How Do Containers
Communicate With Each Other?
```

---

Docker Networking provides:

```text
Container-to-Container Communication
```

---

This is the foundation of:

```text
Microservices

Docker Compose

Kubernetes

EKS
```

---

# Communication Example

Roboshop Architecture

```text
Frontend
    ↓

Catalogue
    ↓

MongoDB
```

---

Frontend sends request:

```text
/api/catalogue/products
```

---

Catalogue processes request.

---

Catalogue fetches data from:

```text
MongoDB
```

---

Response returns to:

```text
Frontend
```

---

# Container Communication Requirement

Suppose:

```text
Frontend Container

Catalogue Container
```

---

Frontend must know:

```text
Catalogue Location
```

---

Without networking:

```text
Communication Fails
```

---

# Traditional VM Approach

Applications communicate using:

```text
Server IP

DNS Name
```

---

Example

```text
catalogue.company.com
```

---

Docker uses:

```text
Container Names
```

inside custom networks.

---

# Custom Network

Create Network

```bash
docker network create roboshop
```

---

Run Containers

```bash
docker run -d \
--name catalogue \
--network roboshop \
catalogue:v1
```

---

```bash
docker run -d \
--name frontend \
--network roboshop \
frontend:v1
```

---

Both join:

```text
roboshop
```

network.

---

# Docker DNS

Docker provides:

```text
Built-in DNS
```

inside custom networks.

---

Container Name

```text
catalogue
```

automatically resolves to:

```text
Container IP
```

---

No manual DNS required.

---

# Example

Frontend Configuration

```text
CATALOGUE_HOST=catalogue
```

---

Frontend sends request to:

```text
http://catalogue:8080
```

---

Docker resolves:

```text
catalogue
```

↓

Container IP

Automatically

---

# Communication Flow

```text
Frontend
      ↓

catalogue
(Container Name)
      ↓

Docker DNS
      ↓

Catalogue Container
```

---

# Catalogue and MongoDB

Run MongoDB

```bash
docker run -d \
--name mongodb \
--network roboshop \
mongo
```

---

Run Catalogue

```bash
docker run -d \
--name catalogue \
--network roboshop \
-e MONGO_HOST=mongodb \
catalogue:v1
```

---

Catalogue connects to:

```text
mongodb
```

instead of:

```text
172.x.x.x
```

---

# RabbitMQ Example

Run RabbitMQ

```bash
docker run -d \
--name rabbitmq \
--network roboshop \
-e RABBITMQ_DEFAULT_USER=roboshop \
-e RABBITMQ_DEFAULT_PASS=roboshop123 \
rabbitmq:3
```

---

Application Configuration

```text
RABBITMQ_HOST=rabbitmq
```

---

Communication

```text
Cart
      ↓

rabbitmq
      ↓

RabbitMQ Container
```

---

# Redis Example

Run Redis

```bash
docker run -d \
--name redis \
--network roboshop \
redis
```

---

Cart Service

```text
REDIS_HOST=redis
```

---

Connection

```text
Cart
    ↓

redis
```

---

# MySQL Example

Run MySQL

```bash
docker run -d \
--name mysql \
--network roboshop \
mysql
```

---

Shipping Service

```text
MYSQL_HOST=mysql
```

---

Communication

```text
Shipping
      ↓

mysql
```

---

# Why Use Container Names?

Bad

```text
172.18.0.2
```

---

Problem

```text
Container Restart
```

↓

IP Changes

↓

Application Breaks

---

Good

```text
rabbitmq

mongodb

redis

mysql
```

---

Benefits

```text
Stable

Readable

Production Friendly
```

---

# Internal Container Ports

Suppose

```text
MongoDB
```

listens on:

```text
27017
```

---

Catalogue connects:

```text
mongodb:27017
```

---

RabbitMQ

```text
5672
```

---

Application

```text
rabbitmq:5672
```

---

Redis

```text
6379
```

---

Application

```text
redis:6379
```

---

# Do We Need Port Mapping?

For container-to-container communication:

```text
No
```

---

Example

```bash
docker run \
--name mongodb \
--network roboshop \
mongo
```

---

No

```bash
-p
```

required.

---

Reason

```text
Communication Happens
Inside Docker Network
```

---

# When Port Mapping Is Required?

External Access

Example

```text
Browser

Postman

Users
```

---

Need

```bash
-p 8080:8080
```

---

Internal Communication

```text
No Port Mapping Required
```

---

# Architecture

```text
Frontend
     ↓

Catalogue
     ↓

MongoDB
```

---

Docker Resolution

```text
catalogue
     ↓

Container IP
```

---

MongoDB Resolution

```text
mongodb
     ↓

Container IP
```

---

# Communication Testing

Access Container

```bash
docker exec -it frontend bash
```

---

Ping Service

```bash
ping catalogue
```

---

Result

```text
Catalogue IP Returned
```

---

# Test Using Curl

```bash
curl http://catalogue:8080/health
```

---

Expected

```text
Healthy Response
```

---

# Test TCP Port

```bash
nc -zv rabbitmq 5672
```

---

Result

```text
Connection Successful
```

---

# Test Database Port

MongoDB

```bash
nc -zv mongodb 27017
```

---

MySQL

```bash
nc -zv mysql 3306
```

---

Redis

```bash
nc -zv redis 6379
```

---

# Troubleshooting Communication Issues

## Check Network

```bash
docker network ls
```

---

Verify Network

```bash
docker network inspect roboshop
```

---

Check Connected Containers

```text
frontend

catalogue

mongodb

rabbitmq
```

---

# Check DNS Resolution

Inside Container

```bash
ping rabbitmq
```

---

Fails?

---

Possible Cause

```text
Not In Same Network
```

---

# Check Container Status

```bash
docker ps
```

---

Verify

```text
Container Running
```

---

# Check Logs

```bash
docker logs catalogue
```

---

Look For

```text
Connection Refused

Timeout

Host Not Found
```

---

# Real Production Example

Roboshop

```text
Frontend
      ↓

Catalogue
      ↓

MongoDB

Cart
      ↓

Redis

User
      ↓

MongoDB

Shipping
      ↓

MySQL

Payment
      ↓

RabbitMQ
```

---

All services communicate using:

```text
Container Names
```

inside:

```text
roboshop Network
```

---

# Common Interview Questions

## How Do Containers Communicate?

### Short Answer

Containers communicate through Docker networks using container names and ports.

### Detailed Explanation

Docker provides built-in DNS resolution that converts container names into IP addresses.

### Production Example

Catalogue connecting to MongoDB using mongodb hostname.

### Follow-Up Questions

- Do we need static IPs?
- How does Docker DNS work?

---

## Why Use Container Names Instead of IP Addresses?

### Short Answer

Container IPs can change after restart.

### Detailed Explanation

Docker DNS provides stable service discovery using container names.

### Production Example

rabbitmq instead of 172.18.0.5.

### Follow-Up Questions

- What happens when container restarts?
- How is name resolution performed?

---

## Is Port Mapping Required Between Containers?

### Short Answer

No.

### Detailed Explanation

Containers communicate directly inside the Docker network using internal ports.

### Production Example

Catalogue connecting to mongodb:27017 without -p.

### Follow-Up Questions

- When is -p required?
- What is the difference between internal and external access?

---

## How Do You Troubleshoot Container Connectivity?

### Short Answer

Check networks, DNS resolution, container status, and listening ports.

### Detailed Explanation

Use docker network inspect, ping, curl, nc, and docker logs to identify issues.

### Production Example

Testing RabbitMQ connectivity using nc -zv rabbitmq 5672.

### Follow-Up Questions

- How do you verify Docker DNS?
- How do you identify network membership?

---

# Key Takeaways

```text
Containers communicate through Docker networks.

Custom bridge networks provide DNS-based service discovery.

Container names should be used instead of IP addresses.

Docker automatically resolves container names to IP addresses.

Port mapping is not required for internal container communication.

Services communicate using hostname:port format.

docker network inspect helps troubleshoot connectivity issues.

ping, curl, and nc are useful networking troubleshooting tools.

Container-to-container communication is the foundation of microservice architecture.
```