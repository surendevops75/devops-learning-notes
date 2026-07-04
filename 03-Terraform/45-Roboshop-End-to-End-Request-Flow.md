# Roboshop End-to-End Request Flow

## Introduction

In a microservices architecture, a single user request may travel through multiple components before a response is returned.

Examples:

```text
Route53

Frontend Load Balancer

Frontend Application

Backend Load Balancer

Catalogue Service

MongoDB
```

Understanding the complete request flow is important for:

```text
Troubleshooting

Debugging

Performance Analysis

Production Support

DevOps Interviews
```

---

# Roboshop Architecture Overview

```text
User
  ↓
Route53
  ↓
Frontend ALB
  ↓
Frontend Target Group
  ↓
Frontend Application (Nginx)
  ↓
Backend ALB
  ↓
Backend Target Group
  ↓
Catalogue Service
  ↓
MongoDB
```

---

# User Accesses Application

Example:

```text
https://roboshop-dev.daws86s.fun
```

---

# Step 1 - Route53 Resolution

Browser sends DNS request.

Route53 contains:

```text
roboshop-dev.daws86s.fun
```

↓

```text
Frontend ALB DNS Name
```

---

# Route53 Record

Example:

```text
roboshop-dev.daws86s.fun
```

↓

```text
frontend-alb.amazonaws.com
```

---

# Request Flow

```text
User
 ↓
Route53
 ↓
Frontend ALB
```

---

# Step 2 - Frontend ALB Receives Request

Frontend ALB listens on:

```text
HTTPS 443
```

---

# Listener

```text
443
```

Protocol:

```text
HTTPS
```

---

# SSL Termination

Frontend ALB contains:

```text
ACM Certificate
```

---

Example:

```text
roboshop-dev.daws86s.fun
```

Certificate validation happens here.

---

# Request Flow

```text
User
 ↓ HTTPS
Frontend ALB
```

---

# Step 3 - Listener Rule Evaluation

ALB evaluates listener rules.

Example Rule:

```text
roboshop-dev.daws86s.fun
```

↓

```text
Frontend Target Group
```

---

# Step 4 - Target Group

Target Group contains:

```text
Frontend-1

Frontend-2

Frontend-3
```

---

# Health Check Validation

Before sending traffic:

ALB verifies:

```text
Frontend Healthy?
```

---

Example:

```text
Health Check Success
```

Traffic forwarded.

---

Example:

```text
Health Check Failed
```

Traffic blocked.

---

# Request Reaches Frontend

Frontend service is typically:

```text
Nginx
```

---

Request:

```text
https://roboshop-dev.daws86s.fun
```

↓

```text
Frontend Nginx
```

---

# Frontend Serves Static Content

Examples:

```text
HTML

CSS

JavaScript
```

---

# User Clicks Categories

Request:

```text
https://roboshop-dev.daws86s.fun/api/catalogue/categories
```

---

# What Happens Next?

Frontend itself does not contain catalogue data.

It forwards the request.

---

# Nginx Reverse Proxy

Frontend Nginx contains:

```text
nginx.conf
```

---

Inside nginx.conf:

```text
/api/catalogue
```

↓

```text
catalogue.backend-alb-dev.daws86s.fun
```

---

# Request Flow

```text
Frontend
 ↓
Backend ALB
```

---

# Backend ALB Architecture

Route53 contains:

```text
*.backend-alb-dev.daws86s.fun
```

↓

```text
Backend ALB
```

---

Examples:

```text
catalogue.backend-alb-dev.daws86s.fun

user.backend-alb-dev.daws86s.fun

cart.backend-alb-dev.daws86s.fun
```

---

# Backend ALB Listener

Backend ALB listens on:

```text
HTTP 80
```

---

Request

```text
catalogue.backend-alb-dev.daws86s.fun
```

arrives at Backend ALB.

---

# Rule Evaluation

Backend ALB evaluates:

```text
catalogue.*
```

---

Matching Rule:

```text
Catalogue Rule
```

↓

```text
Catalogue Target Group
```

---

# Catalogue Target Group

Contains:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

---

# Health Check

Backend ALB validates:

```text
Catalogue Healthy?
```

---

Example:

```text
Health Check Passed
```

Traffic forwarded.

---

# Request Reaches Catalogue Service

Catalogue application receives:

```text
/api/catalogue/categories
```

---

Catalogue now needs product data.

---

# Database Communication

Catalogue connects to:

```text
MongoDB
```

---

Request Flow

```text
Catalogue
 ↓
MongoDB
```

---

MongoDB returns:

```text
Category Data

Product Data
```

---

# Response Flow

```text
MongoDB
 ↓
Catalogue
 ↓
Backend ALB
 ↓
Frontend
 ↓
User Browser
```

---

# Cart Example

User Action:

```text
Add Product To Cart
```

---

Request:

```text
https://roboshop-dev.daws86s.fun/api/cart/add/siva/HPTD/1
```

---

Request Breakdown

```text
User = siva

Product = HPTD

Quantity = 1
```

---

# Request Flow

```text
User
 ↓
Route53
 ↓
Frontend ALB
 ↓
Frontend
 ↓
Backend ALB
 ↓
Cart Service
 ↓
Redis / Database
 ↓
Response
```

---

# Complete Catalogue Flow

```text
User
 ↓
Route53
 ↓
Frontend ALB
 ↓
Frontend TG
 ↓
Frontend Nginx
 ↓
Backend ALB
 ↓
Catalogue Rule
 ↓
Catalogue TG
 ↓
Catalogue Service
 ↓
MongoDB
 ↓
Response
```

---

# Production Troubleshooting

Suppose:

```text
Categories Not Loading
```

---

Check:

```text
Frontend Health
```

---

Check:

```text
Backend ALB Rule
```

---

Check:

```text
Catalogue Health Check
```

---

Check:

```text
MongoDB Connectivity
```

---

# Debugging Commands

Frontend:

```bash
curl http://catalogue.backend-alb-dev.daws86s.fun/health
```

---

Catalogue:

```bash
systemctl status catalogue
```

---

MongoDB Connectivity:

```bash
nc -zv mongodb-host 27017
```

---

# Why Is This Architecture Used?

Benefits:

```text
Microservices

Independent Scaling

High Availability

Fault Isolation

Centralized Routing
```

---

# Common Interview Questions

## Explain the Request Flow in Roboshop.

### Short Answer

User → Route53 → Frontend ALB → Frontend → Backend ALB → Catalogue Service → MongoDB → Response.

### Detailed Explanation

A user request first reaches Route53, then the Frontend ALB. Frontend Nginx forwards API requests to Backend ALB, which routes them to the appropriate microservice and database.

### Production Example

/api/catalogue/categories request reaching MongoDB through Catalogue service.

### Follow-Up Questions

- Why use two load balancers?
- Why not directly expose catalogue service?

---

## Why Does Frontend Call Backend ALB Instead of Services Directly?

### Short Answer

Backend ALB provides centralized routing and service discovery.

### Detailed Explanation

Frontend applications only need to know Backend ALB DNS names. Backend ALB handles routing to the correct microservice.

### Production Example

catalogue.backend-alb-dev.daws86s.fun routed to Catalogue Target Group.

### Follow-Up Questions

- What happens when catalogue instances scale?
- Does Nginx configuration change?

---

## What Happens If Catalogue Service Fails?

### Short Answer

Backend ALB health checks fail and traffic is removed from unhealthy instances.

### Detailed Explanation

ALB continuously monitors service health and forwards traffic only to healthy instances.

### Production Example

Catalogue-2 fails and is removed from the target group automatically.

### Follow-Up Questions

- What performs the health checks?
- What happens if all instances fail?

---

# Key Takeaways

```text
User requests first reach Route53.

Frontend ALB handles HTTPS traffic.

Frontend Nginx serves static content and proxies API requests.

Backend ALB routes requests to microservices.

Listener rules determine target groups.

Health checks ensure traffic reaches healthy instances.

Catalogue service retrieves data from MongoDB.

Responses travel back through the same architecture.

Understanding request flow is critical for production troubleshooting and DevOps interviews.
```