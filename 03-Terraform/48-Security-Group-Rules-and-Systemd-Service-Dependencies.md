# Security Group Rules and Systemd Service Dependencies

## Introduction

In a microservices architecture, applications rarely work independently.

Example:

```text
Frontend
   ↓
Catalogue
   ↓
MongoDB
```

---

Another Example:

```text
Cart
   ↓
Redis
```

---

Another Example:

```text
Shipping
   ↓
MySQL
```

---

For applications to communicate successfully, two things must be correct:

```text
Security Group Rules

Service Dependencies
```

Most production outages happen because one of these is misconfigured.

---

# What are Security Groups?

Security Groups act as:

```text
Virtual Firewalls
```

for AWS resources.

---

# Purpose

Control:

```text
Who Can Connect?

Which Port?

Which Protocol?
```

---

# Traffic Types

## Ingress

Incoming Traffic

```text
Outside
   ↓
Server
```

---

Examples:

```text
User → Frontend

Catalogue → MongoDB

Cart → Redis
```

---

## Egress

Outgoing Traffic

```text
Server
   ↓
Outside
```

---

Examples:

```text
Catalogue → MongoDB

Shipping → MySQL

Frontend → Backend ALB
```

---

# Example Architecture

```text
Frontend
   ↓
Catalogue
   ↓
MongoDB
```

---

Communication Ports

```text
Frontend → Catalogue : 8080

Catalogue → MongoDB : 27017
```

---

# Security Group Requirement

Suppose:

```text
Frontend
```

wants to connect to:

```text
Catalogue
```

---

Catalogue listens on:

```text
8080
```

---

Catalogue Security Group must allow:

```text
Port 8080
```

from:

```text
Frontend
```

---

# Wrong Approach

Allow:

```text
10.0.1.15
```

---

Problem:

```text
IP Changes

Auto Scaling

Maintenance
```

---

# Production Best Practice

Allow:

```text
Security Group ID
```

instead of IP address.

---

# Example

Frontend SG:

```text
sg-frontend
```

---

Catalogue SG:

```text
sg-catalogue
```

---

Rule:

```text
Allow Port 8080

Source = sg-frontend
```

---

Meaning:

```text
Any Instance Using sg-frontend

Can Access Catalogue
```

---

# Why SG Referencing?

Benefits:

```text
Dynamic

Auto Scaling Friendly

Easy Maintenance
```

---

# Real Production Example

Suppose:

```text
Frontend-1

Frontend-2

Frontend-3
```

exist today.

---

Tomorrow ASG creates:

```text
Frontend-4

Frontend-5
```

---

Because all use:

```text
sg-frontend
```

no new firewall rules are required.

---

# Common Microservice Ports

MongoDB:

```text
27017
```

---

MySQL:

```text
3306
```

---

Redis:

```text
6379
```

---

RabbitMQ:

```text
5672
```

---

Catalogue:

```text
8080
```

---

# Security Group Communication Example

```text
Frontend SG
      ↓
Catalogue SG
      ↓
MongoDB SG
```

---

Rules:

```text
Frontend SG
    ↓
Allow 8080
    ↓
Catalogue SG
```

---

```text
Catalogue SG
    ↓
Allow 27017
    ↓
MongoDB SG
```

---

# Production Troubleshooting

Suppose:

```text
Catalogue Page Not Loading
```

---

Check:

```text
Frontend → Catalogue Connectivity
```

---

Possible Issue:

```text
Port 8080 Blocked
```

---

Check:

```bash
nc -zv catalogue-host 8080
```

---

Or:

```bash
telnet catalogue-host 8080
```

---

# What are Systemd Services?

Most Linux applications run as:

```text
Systemd Services
```

---

Examples:

```text
catalogue.service

user.service

cart.service

shipping.service
```

---

# Why Systemd?

Provides:

```text
Auto Start

Auto Restart

Logging

Dependency Management
```

---

# Service File Example

```text
/etc/systemd/system/catalogue.service
```

---

# Service Dependencies

Applications usually depend on:

```text
Database

Cache

Message Queue

External Services
```

---

# Example

Catalogue depends on:

```text
MongoDB
```

---

Flow:

```text
Catalogue
   ↓
MongoDB
```

---

# Production Problem

Suppose:

```text
MongoDB Down
```

---

Catalogue Starts First

Result:

```text
Connection Refused
```

---

Application Fails.

---

# Another Example

Shipping Service

depends on:

```text
MySQL
```

---

If MySQL is unavailable:

```text
Shipping Service Fails
```

---

# What Should DevOps Engineers Check?

Verify:

```text
Host URL

Port Number

Connectivity

Service Status
```

---

# Example Validation

MongoDB Host:

```text
mongodb-dev.daws86s.fun
```

---

Port:

```text
27017
```

---

Check:

```bash
nc -zv mongodb-dev.daws86s.fun 27017
```

---

# Catalogue Troubleshooting Flow

Step 1

```bash
systemctl status catalogue
```

---

Step 2

Check Logs

```bash
journalctl -u catalogue
```

---

Step 3

Verify Host

```text
mongodb-dev.daws86s.fun
```

---

Step 4

Verify Port

```text
27017
```

---

Step 5

Test Connectivity

```bash
nc -zv mongodb-dev.daws86s.fun 27017
```

---

# Common Service Dependency Chain

```text
Frontend
   ↓
Catalogue
   ↓
MongoDB
```

---

```text
Cart
   ↓
Redis
```

---

```text
Shipping
   ↓
MySQL
```

---

```text
Payment
   ↓
RabbitMQ
```

---

# Real Production Scenario

Issue:

```text
Catalogue Health Check Failing
```

---

Investigation:

```text
Catalogue Service Running
```

---

But:

```text
MongoDB Port Blocked
```

---

Result:

```text
Catalogue Cannot Connect

Health Check Fails

ALB Removes Instance
```

---

Root Cause:

```text
Security Group Misconfiguration
```

---

# Common Interview Questions

## What is a Security Group?

### Short Answer

A Security Group is a virtual firewall controlling inbound and outbound traffic.

### Detailed Explanation

Security Groups define which resources can communicate and on which ports.

### Production Example

Frontend SG allowed to access Catalogue SG on port 8080.

### Follow-Up Questions

- Difference between SG and NACL?
- What is SG referencing?

---

## Why Use Security Group IDs Instead of IP Addresses?

### Short Answer

Because IP addresses change while Security Groups remain constant.

### Detailed Explanation

Using Security Group references supports Auto Scaling and dynamic infrastructure.

### Production Example

Frontend ASG launching new instances without requiring firewall changes.

### Follow-Up Questions

- Can Security Groups reference other Security Groups?
- Why is this useful in microservices?

---

## What are Systemd Service Dependencies?

### Short Answer

Applications often depend on databases, caches, or other services to function.

### Detailed Explanation

Even if a service starts successfully, it may fail if dependent services are unavailable.

### Production Example

Catalogue service depending on MongoDB.

### Follow-Up Questions

- How do you troubleshoot service startup failures?
- What happens if dependencies are unavailable?

---

## How Would You Troubleshoot Catalogue Service Failure?

### Short Answer

Check service status, logs, connectivity, hostnames, and ports.

### Detailed Explanation

Validate whether the application can communicate with MongoDB and verify security group rules.

### Production Example

MongoDB port 27017 blocked by a Security Group.

### Follow-Up Questions

- Which Linux commands would you use?
- How would you verify database connectivity?

---

# Key Takeaways

```text
Security Groups act as AWS virtual firewalls.

Ingress controls incoming traffic.

Egress controls outgoing traffic.

Security Group referencing is preferred over IP-based rules.

Microservices communicate using specific ports.

Applications often depend on databases and caches.

Systemd manages service startup and monitoring.

Most production issues involve connectivity or dependency failures.

Always verify hostnames, ports, service status, and Security Group rules during troubleshooting.

Understanding SGs and service dependencies is essential for production DevOps work.
```