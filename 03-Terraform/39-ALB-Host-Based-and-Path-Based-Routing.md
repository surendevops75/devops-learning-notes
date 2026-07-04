# ALB Host-Based and Path-Based Routing

## Introduction

Modern applications usually consist of multiple services.

Example:

```text
Frontend

Catalogue

User

Cart

Shipping

Payment
```

Users access a single application, but internally requests must be routed to the correct service.

AWS Application Load Balancer (ALB) provides:

```text
Host-Based Routing

Path-Based Routing
```

to solve this problem.

---

# Load Balancer Components

A typical ALB architecture contains:

```text
Load Balancer

Listener

Rule

Target Group

Instances
```

---

# Request Flow

```text
User
   ↓
Load Balancer
   ↓
Listener
   ↓
Rule
   ↓
Target Group
   ↓
Instance
```

---

# What is a Listener?

A Listener checks incoming requests.

Examples:

```text
Port 80 (HTTP)

Port 443 (HTTPS)
```

---

# Example

```text
http://catalogue.backend-alb-dev.daws86s.fun
```

Listener:

```text
80
```

receives the request.

---

# What is a Rule?

A Rule decides:

```text
Where Traffic Should Go
```

---

# Example

```text
catalogue.backend-alb-dev.daws86s.fun
```

↓

```text
Catalogue Target Group
```

---

# What is a Target Group?

A Target Group is a collection of backend servers.

Example:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

---

# Health Checks

ALB continuously checks backend health.

Example:

```bash
curl http://catalogue.backend-alb-dev.daws86s.fun/health
```

Expected:

```text
Healthy Response
```

---

# Why Health Checks?

Suppose:

```text
Catalogue-2 Crashes
```

ALB detects failure and removes it from traffic.

---

# Host-Based Routing

Host-based routing uses:

```text
Hostname
```

to determine the destination.

---

# Example

Request:

```text
catalogue.backend-alb-dev.daws86s.fun
```

↓

```text
Catalogue Target Group
```

---

Request:

```text
user.backend-alb-dev.daws86s.fun
```

↓

```text
User Target Group
```

---

Request:

```text
shipping.backend-alb-dev.daws86s.fun
```

↓

```text
Shipping Target Group
```

---

# Host-Based Routing Flow

```text
Host Name
      ↓
Listener Rule
      ↓
Target Group
```

---

# Production Example

Rule:

```text
catalogue.backend-alb-dev.daws86s.fun
```

↓

```text
Catalogue TG
```

---

Rule:

```text
cart.backend-alb-dev.daws86s.fun
```

↓

```text
Cart TG
```

---

# Wildcard Host Routing

Example:

```text
*.backend-alb-dev.daws86s.fun
```

Matches:

```text
catalogue.backend-alb-dev.daws86s.fun

user.backend-alb-dev.daws86s.fun

shipping.backend-alb-dev.daws86s.fun
```

---

# What is Path-Based Routing?

Path-based routing uses:

```text
URL Path
```

to determine the destination.

---

# Example

Request:

```text
/api/catalogue
```

↓

```text
Catalogue Target Group
```

---

Request:

```text
/api/user
```

↓

```text
User Target Group
```

---

Request:

```text
/api/cart
```

↓

```text
Cart Target Group
```

---

# Path-Based Routing Flow

```text
URL Path
      ↓
Listener Rule
      ↓
Target Group
```

---

# Example

```text
/api/catalogue/products
```

↓

```text
Catalogue Service
```

---

# Host Name vs Context Path

Trainer Example:

```text
icicibank.com
```

---

# Host-Based Routing Example

```text
netbanking.icicibank.com
```

Here:

```text
netbanking
```

is part of:

```text
Hostname
```

---

Request:

```text
netbanking.icicibank.com/user/239733457/dashboard
```

Routing Decision:

```text
Hostname
```

---

# Path-Based Routing Example

```text
icicibank.com/netbanking
```

Here:

```text
/netbanking
```

is:

```text
Context Path
```

---

Request:

```text
icicibank.com/netbanking/user/239733457/dashboard
```

Routing Decision:

```text
URL Path
```

---

# Comparison

## Host-Based Routing

Example:

```text
catalogue.backend-alb-dev.daws86s.fun
```

Uses:

```text
Hostname
```

---

## Path-Based Routing

Example:

```text
/api/catalogue
```

Uses:

```text
URL Path
```

---

# Real Microservices Example

## Host-Based

```text
catalogue.company.com

user.company.com

cart.company.com
```

---

## Path-Based

```text
company.com/catalogue

company.com/user

company.com/cart
```

---

# Which One is Better?

Depends on requirements.

---

# Host-Based Routing Advantages

```text
Clean Separation

Independent Domains

Better Service Isolation
```

---

# Path-Based Routing Advantages

```text
Single Domain

Simple DNS Management

Easy Client Integration
```

---

# Complete Production Architecture

```text
User
   ↓
ALB
   ↓
Listener (80/443)
   ↓
Rules
   ↓
Target Groups
   ↓
Application Instances
```

---

# Roboshop Example

Request:

```text
catalogue.backend-alb-dev.daws86s.fun
```

↓

```text
ALB
```

↓

```text
Catalogue Listener Rule
```

↓

```text
Catalogue Target Group
```

↓

```text
Catalogue Instances
```

---

# AWS CLI Example

Terminate Instance:

```bash
aws ec2 terminate-instances --instance-ids i-xxxxxxxxxxxxxxxxx
```

Purpose:

```text
Terminate EC2 Instance
```

---

# Production Scenario

Suppose:

```text
Catalogue Instance Failed
```

ASG creates:

```text
New Catalogue Instance
```

ALB automatically registers it through the Target Group.

No manual intervention required.

---

# Common Interview Questions

## What is a Listener in ALB?

### Short Answer

A Listener checks incoming traffic on a specific port and forwards requests based on rules.

### Detailed Explanation

Listeners receive HTTP or HTTPS traffic and evaluate routing rules before forwarding requests to target groups.

### Production Example

Port 443 listener handling production HTTPS traffic.

### Follow-Up Questions

- Can ALB have multiple listeners?
- Difference between HTTP and HTTPS listeners?

---

## What is Host-Based Routing?

### Short Answer

Host-based routing routes traffic using the hostname.

### Detailed Explanation

ALB evaluates the Host header in the request and forwards traffic to the matching target group.

### Production Example

catalogue.backend-alb-dev.daws86s.fun routed to Catalogue Target Group.

### Follow-Up Questions

- What is a wildcard host rule?
- When is host-based routing preferred?

---

## What is Path-Based Routing?

### Short Answer

Path-based routing routes traffic using the URL path.

### Detailed Explanation

ALB evaluates the request path and forwards traffic to the appropriate target group.

### Production Example

/api/catalogue routed to Catalogue service.

### Follow-Up Questions

- What is a context path?
- When is path-based routing preferred?

---

## Difference Between Host-Based and Path-Based Routing?

### Short Answer

Host-based routing uses hostnames, path-based routing uses URL paths.

### Detailed Explanation

Host-based routing evaluates domains or subdomains, whereas path-based routing evaluates the path portion of the URL.

### Production Example

catalogue.company.com vs company.com/catalogue

### Follow-Up Questions

- Which routing method is more common in microservices?
- Can both methods be used together?

---

## What is a Target Group?

### Short Answer

A Target Group is a collection of backend servers that receive traffic from the Load Balancer.

### Detailed Explanation

Target Groups perform health checks and distribute traffic to healthy instances.

### Production Example

Catalogue Target Group containing multiple Catalogue instances.

### Follow-Up Questions

- What happens when a health check fails?
- Can a Target Group contain Auto Scaling instances?

---

# Key Takeaways

```text
ALB routes traffic to backend services.

Listeners receive incoming requests.

Rules determine where traffic should go.

Target Groups contain backend servers.

Health Checks ensure traffic reaches only healthy instances.

Host-Based Routing uses hostnames.

Path-Based Routing uses URL paths.

Wildcard host rules can match multiple subdomains.

Both routing methods are heavily used in microservices architectures.

Understanding ALB routing is a very common AWS and DevOps interview topic.
```