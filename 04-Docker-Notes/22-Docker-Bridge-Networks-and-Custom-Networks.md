# Docker Bridge Networks and Custom Networks

## Introduction

One of the most important Docker networking concepts is:

```text
Bridge Network
```

---

Docker containers need to communicate with:

```text
Other Containers

Databases

Message Queues

Microservices
```

---

Docker provides:

```text
Bridge Networks
```

to enable this communication.

---

# What is a Bridge Network?

A bridge network is:

```text
A Virtual Network

Created By Docker

Inside The Host Server
```

---

When Docker is installed:

```text
Docker Creates

docker0
```

bridge interface automatically.

---

Verify

```bash
ip addr
```

---

Output

```text
docker0
```

---

Architecture

```text
Host Server
      ↓
docker0 Bridge
      ↓
Containers
```

---

# Default Bridge Network

Check Available Networks

```bash
docker network ls
```

---

Output

```text
NETWORK ID     NAME      DRIVER

xxxxx          bridge    bridge
xxxxx          host      host
xxxxx          none      null
```

---

Docker automatically connects containers to:

```text
bridge
```

network if no network is specified.

---

Example

```bash
docker run -d nginx
```

---

Container joins:

```text
Default Bridge Network
```

---

# How Default Bridge Works?

Example

```text
Container A
172.17.0.2

Container B
172.17.0.3
```

---

Both connected through:

```text
docker0
```

---

Architecture

```text
Container A
172.17.0.2
      ↓

docker0 Bridge
      ↓

Container B
172.17.0.3
```

---

Communication possible using:

```text
IP Addresses
```

---

# Problem with Default Bridge

Suppose:

```text
RabbitMQ

Catalogue

Cart
```

containers running.

---

RabbitMQ

```text
172.17.0.2
```

---

Catalogue

```text
172.17.0.3
```

---

Application Configuration

```text
RABBITMQ_HOST=172.17.0.2
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

# Production Issue

Default bridge network does not provide reliable:

```text
Container Name Resolution
```

for modern microservice communication.

---

Using IP addresses:

```text
Not Recommended
```

---

# Solution

Create Custom Bridge Network.

---

# Custom Bridge Network

Create

```bash
docker network create roboshop
```

---

Verify

```bash
docker network ls
```

---

Output

```text
bridge

host

none

roboshop
```

---

Docker creates:

```text
Dedicated Network
```

for applications.

---

# Benefits of Custom Networks

```text
Automatic DNS Resolution

Container Discovery

Better Isolation

Microservice Friendly
```

---

# Running Containers in Custom Network

Example

```bash
docker run -d \
--name rabbitmq \
--network roboshop \
rabbitmq:3
```

---

Container joins:

```text
roboshop Network
```

---

# RabbitMQ Example

Command

```bash
docker run -d \
--name rabbitmq \
--network roboshop \
-e RABBITMQ_DEFAULT_USER=roboshop \
-e RABBITMQ_DEFAULT_PASS=roboshop123 \
rabbitmq:3
```

---

Breakdown

```text
-d
→ Detached Mode

--name rabbitmq
→ Container Name

--network roboshop
→ Attach To Custom Network

-e
→ Environment Variables

rabbitmq:3
→ RabbitMQ Image
```

---

# Why Container Name Matters?

Suppose:

```text
Catalogue Service
```

needs RabbitMQ.

---

Instead of

```text
172.18.0.5
```

use

```text
rabbitmq
```

---

Application Configuration

```text
RABBITMQ_HOST=rabbitmq
```

---

Docker DNS resolves:

```text
rabbitmq
```

↓

Container IP

Automatically

---

# Container Communication Flow

```text
Catalogue
      ↓

rabbitmq
(Container Name)
      ↓

Docker DNS
      ↓

RabbitMQ Container
```

---

No IP addresses required.

---

# Verify Network Details

Inspect Network

```bash
docker network inspect roboshop
```

---

Shows

```text
Subnet

Gateway

Connected Containers
```

---

Example Output

```text
rabbitmq

catalogue

cart
```

---

# Connect Existing Container

Suppose container already running.

---

Attach to network

```bash
docker network connect roboshop catalogue
```

---

Now catalogue joins:

```text
roboshop
```

network.

---

# Disconnect Container

```bash
docker network disconnect roboshop catalogue
```

---

Container removed from network.

---

# Multi-Container Example

RabbitMQ

```bash
docker run -d \
--name rabbitmq \
--network roboshop \
rabbitmq:3
```

---

Catalogue

```bash
docker run -d \
--name catalogue \
--network roboshop \
catalogue:v1
```

---

Cart

```bash
docker run -d \
--name cart \
--network roboshop \
cart:v1
```

---

Communication

```text
catalogue
     ↓

rabbitmq

cart
     ↓

rabbitmq
```

---

Using container names.

---

# Real Roboshop Example

Architecture

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
```

---

All containers connected to:

```text
roboshop
```

network.

---

Services communicate using:

```text
mongodb

redis

mysql

rabbitmq
```

instead of IPs.

---

# Custom Network Advantages

## Service Discovery

```text
Container Names Work
```

---

## Better Isolation

Containers only communicate inside:

```text
Specified Network
```

---

## Easier Configuration

Use

```text
rabbitmq
```

instead of

```text
172.x.x.x
```

---

## Production Friendly

Microservices depend heavily on:

```text
DNS-Based Communication
```

---

# Troubleshooting

List Networks

```bash
docker network ls
```

---

Inspect Network

```bash
docker network inspect roboshop
```

---

Check Container Networks

```bash
docker inspect container-id
```

---

Verify DNS Resolution

```bash
docker exec -it catalogue bash
```

---

Test

```bash
ping rabbitmq
```

---

Result

```text
RabbitMQ IP Returned
```

---

# Default Bridge vs Custom Bridge

| Feature | Default Bridge | Custom Bridge |
|----------|---------------|---------------|
| Created Automatically | Yes | No |
| DNS Resolution | Limited | Yes |
| Container Name Access | Not Reliable | Yes |
| Isolation | Basic | Better |
| Production Usage | Rare | Very Common |

---

# Common Interview Questions

## What is Docker Bridge Network?

### Short Answer

A virtual network that enables communication between Docker containers.

### Detailed Explanation

Docker creates a bridge interface that connects containers running on the same host.

### Production Example

docker0 bridge network.

### Follow-Up Questions

- What is docker0?
- How are IP addresses assigned?

---

## Why Create a Custom Bridge Network?

### Short Answer

To enable DNS-based communication and better isolation.

### Detailed Explanation

Containers can communicate using container names instead of changing IP addresses.

### Production Example

rabbitmq accessible using hostname rabbitmq.

### Follow-Up Questions

- What happens if container IP changes?
- How does Docker DNS work?

---

## How Do Containers Communicate in a Custom Network?

### Short Answer

Using container names.

### Detailed Explanation

Docker provides built-in DNS resolution inside custom bridge networks.

### Production Example

Catalogue connects to RabbitMQ using rabbitmq hostname.

### Follow-Up Questions

- Do we need static IPs?
- How is name resolution performed?

---

## What Does docker network inspect Do?

### Short Answer

Displays network configuration and connected containers.

### Detailed Explanation

Useful for troubleshooting networking issues and verifying container membership.

### Production Example

docker network inspect roboshop

### Follow-Up Questions

- How do you verify connected containers?
- How do you identify network subnets?

---

# Key Takeaways

```text
Docker creates a default bridge network called docker0.

Containers receive private IP addresses.

Using container IPs is not recommended.

Custom bridge networks provide DNS resolution.

Container names can be used for communication.

docker network create creates custom networks.

docker network inspect shows network details.

Custom networks are preferred in production.

Microservices typically communicate using container names rather than IP addresses.

Docker custom bridge networks are the foundation of container service discovery.
```