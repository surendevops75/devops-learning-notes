# Docker Networking Fundamentals

## Introduction

Containers are isolated by default.

Example:

```text
Frontend Container

Catalogue Container

Cart Container

RabbitMQ Container

MySQL Container
```

---

Question:

```text
How Do Containers
Communicate With Each Other?
```

---

Docker solves this using:

```text
Docker Networking
```

---

Docker networking is one of the most important concepts in:

```text
Microservices

Kubernetes

EKS

Container Platforms
```

---

# Why Networking Is Required?

Consider Roboshop.

Architecture

```text
Frontend
    ↓

Catalogue
    ↓

MongoDB
```

---

Frontend must call:

```text
Catalogue API
```

---

Catalogue must connect to:

```text
MongoDB
```

---

Without networking:

```text
Containers Cannot Talk
To Each Other
```

---

# Docker Network Architecture

When Docker starts:

```text
Docker Engine
```

creates:

```text
Virtual Network Components
```

inside the host.

---

Architecture

```text
Linux Server
      ↓

Docker Engine
      ↓

Docker Networks
      ↓

Containers
```

---

# What Happens When Container Starts?

Example

```bash
docker run nginx
```

---

Docker allocates:

```text
Container ID

Virtual Interface

IP Address

Routing Rules
```

---

Container becomes:

```text
Reachable
```

inside Docker network.

---

# Network Drivers

Docker supports:

```text
Bridge

Host

None

Overlay

Macvlan
```

---

Most Common

```text
Bridge

Host
```

---

We'll cover each in separate files.

---

# Understanding Network Isolation

Suppose

```text
Container A

Container B
```

---

Both run on same server.

---

By default:

```text
Separate Filesystems

Separate Processes

Separate Network Namespaces
```

---

Container A cannot simply access:

```text
localhost
```

of Container B.

---

Reason

```text
Each Container Has
Its Own Network Stack
```

---

# Network Namespace

Every container receives:

```text
Own

IP Address

Routing Table

Network Interface
```

---

Example

Container A

```text
172.17.0.2
```

---

Container B

```text
172.17.0.3
```

---

Even though:

```text
Same Host
```

---

# Container IP Address

Command

```bash
docker inspect container-id
```

---

Shows

```text
IP Address

Gateway

Network Details
```

---

Example

```text
172.17.0.2
```

---

Assigned by Docker.

---

# Port Mapping

Containers have:

```text
Internal Ports
```

---

Example

Nginx

```text
80
```

---

Host cannot access automatically.

---

Need:

```bash
docker run -p 80:80 nginx
```

---

Meaning

```text
Host Port 80
      ↓
Container Port 80
```

---

# Port Mapping Flow

```text
Browser
     ↓

Host:80
     ↓

Docker NAT
     ↓

Container:80
```

---

# Example

Run Nginx

```bash
docker run -d -p 80:80 nginx
```

---

Access

```text
http://SERVER-IP
```

---

Response

```text
Nginx Welcome Page
```

---

# Why Port Mapping Is Needed?

Container Network

```text
Private
```

---

External Users

```text
Cannot Reach Container
```

directly.

---

Port mapping exposes:

```text
Container Service
```

to outside world.

---

# Multiple Containers

Example

```text
Nginx
Port 80

Apache
Port 80
```

---

Cannot use:

```bash
-p 80:80
```

for both.

---

Use

```bash
docker run -p 80:80 nginx

docker run -p 8080:80 httpd
```

---

Result

```text
Host 80
    ↓
Nginx

Host 8080
    ↓
Apache
```

---

# Docker Network Commands

List Networks

```bash
docker network ls
```

---

Output

```text
bridge

host

none
```

---

Inspect Network

```bash
docker network inspect bridge
```

---

Shows

```text
Subnet

Gateway

Connected Containers
```

---

# Container Connectivity

Example

```text
Frontend
     ↓

Catalogue
     ↓

RabbitMQ
```

---

Containers communicate using:

```text
Docker Networks
```

---

Without proper network:

```text
Communication Fails
```

---

# Real Production Example

Roboshop

```text
Frontend

Catalogue

User

Cart

Shipping

RabbitMQ

MongoDB
```

---

All services require:

```text
Network Communication
```

---

Otherwise:

```text
Microservices Cannot Function
```

---

# Networking Layers

Container

↓

Virtual Interface

↓

Docker Network

↓

Host Network

↓

Internet

---

Architecture

```text
Container
      ↓

veth Interface
      ↓

Docker Network
      ↓

Host NIC
      ↓

Internet
```

---

# Common Networking Problems

## Application Not Accessible

Check

```bash
docker ps
```

---

Verify

```text
Port Mapping
```

---

Example

```bash
-p 80:80
```

---

Missing mapping:

```text
Container Running

Application Not Reachable
```

---

## Wrong Port

Example

```bash
docker run -p 8080:80 nginx
```

---

Access

```text
SERVER-IP:80
```

---

Fails.

---

Correct

```text
SERVER-IP:8080
```

---

# Production Troubleshooting

Check Container

```bash
docker ps
```

---

Inspect Network

```bash
docker inspect container-id
```

---

Inspect Docker Network

```bash
docker network inspect bridge
```

---

Verify Listening Ports

```bash
netstat -lntp

ss -lntp
```

---

# Common Interview Questions

## Why Is Docker Networking Required?

### Short Answer

Docker networking enables communication between containers and external systems.

### Detailed Explanation

Containers are isolated by default. Docker networking provides connectivity between services and users.

### Production Example

Frontend communicating with Catalogue service.

### Follow-Up Questions

- How do containers communicate?
- What is network isolation?

---

## Why Does Every Container Have Its Own IP?

### Short Answer

Each container receives its own network namespace.

### Detailed Explanation

Containers have isolated networking stacks including interfaces, routing tables, and IP addresses.

### Production Example

172.17.0.2 and 172.17.0.3 assigned to different containers.

### Follow-Up Questions

- How does Docker allocate IPs?
- Can containers share IPs?

---

## What Is Port Mapping?

### Short Answer

Port mapping exposes container ports to the host.

### Detailed Explanation

Docker forwards traffic from a host port to a container port using NAT.

### Production Example

-p 80:80

### Follow-Up Questions

- Why is port mapping required?
- Can multiple containers use the same host port?

---

## How Do You View Docker Networks?

### Short Answer

Using docker network ls.

### Detailed Explanation

Docker maintains virtual networks that containers use for communication.

### Production Example

bridge, host, and none networks.

### Follow-Up Questions

- How do you inspect a network?
- What is the default network?

---

# Key Takeaways

```text
Containers are network isolated by default.

Each container receives its own IP address.

Docker networking enables container communication.

Port mapping exposes container services externally.

Docker provides multiple network drivers.

Bridge and Host networks are the most common.

docker network ls lists available networks.

docker network inspect shows network details.

Networking is the foundation of microservice communication.

Docker networking concepts directly relate to Kubernetes networking.
```