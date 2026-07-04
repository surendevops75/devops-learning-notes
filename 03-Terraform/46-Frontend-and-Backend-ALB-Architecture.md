# Frontend and Backend ALB Architecture

## Introduction

In a microservices architecture, exposing every service directly to users is not a good practice.

Examples:

```text
Catalogue

Cart

User

Shipping

Payment
```

If every service is exposed publicly:

```text
Security Risks

Complex Routing

Difficult Management

Increased Attack Surface
```

To solve this problem, Roboshop uses:

```text
Frontend ALB

Backend ALB
```

---

# Why Two Load Balancers?

The architecture separates:

```text
User Traffic

Internal Service Traffic
```

---

# Frontend ALB

Frontend ALB is:

```text
Internet Facing
```

It accepts traffic from:

```text
Users

Browsers

Mobile Applications
```

---

# Frontend ALB DNS

Example:

```text
roboshop-dev.daws86s.fun
```

---

# Traffic Flow

```text
User
  ↓
Route53
  ↓
Frontend ALB
```

---

# Frontend ALB Listener

Protocol:

```text
HTTPS
```

Port:

```text
443
```

---

# Why HTTPS?

Benefits:

```text
Encryption

Authentication

Secure Communication
```

---

# SSL Termination

Frontend ALB contains:

```text
ACM Certificate
```

Example:

```text
roboshop-dev.daws86s.fun
```

---

# Frontend Listener Rule

Rule:

```text
roboshop-dev.daws86s.fun
```

↓

```text
Frontend Target Group
```

---

# Frontend Target Group

Contains:

```text
Frontend-1

Frontend-2

Frontend-3
```

---

# Health Checks

Before forwarding traffic:

ALB verifies:

```text
Frontend Healthy?
```

---

# Frontend Application

Typically:

```text
Nginx
```

---

# Responsibilities of Frontend

```text
Serve HTML

Serve CSS

Serve JavaScript

Reverse Proxy API Requests
```

---

# Example Request

```text
https://roboshop-dev.daws86s.fun
```

---

Served by:

```text
Frontend Nginx
```

---

# API Requests

Example:

```text
https://roboshop-dev.daws86s.fun/api/catalogue/categories
```

---

Frontend cannot process catalogue requests.

Instead:

```text
Nginx Proxies Request
```

to Backend ALB.

---

# Nginx Configuration

Example:

```nginx
location /api/catalogue {
    proxy_pass http://catalogue.backend-alb-dev.daws86s.fun;
}
```

---

# Why Use Nginx?

Benefits:

```text
Reverse Proxy

Static Content Delivery

API Routing

Performance
```

---

# Backend ALB

Backend ALB is:

```text
Private/Internal
```

---

Purpose:

```text
Microservice Routing
```

---

# Backend ALB DNS

Examples:

```text
catalogue.backend-alb-dev.daws86s.fun

user.backend-alb-dev.daws86s.fun

cart.backend-alb-dev.daws86s.fun
```

---

# Route53 Wildcard Record

Example:

```text
*.backend-alb-dev.daws86s.fun
```

↓

```text
Backend ALB
```

---

# Why Wildcard Records?

Instead of creating:

```text
catalogue.backend-alb-dev.daws86s.fun

user.backend-alb-dev.daws86s.fun

cart.backend-alb-dev.daws86s.fun

shipping.backend-alb-dev.daws86s.fun
```

individually,

one record handles all backend services.

---

# Backend ALB Listener

Protocol:

```text
HTTP
```

Port:

```text
80
```

---

# Why HTTP?

Traffic remains inside the VPC.

Communication is:

```text
Internal

Private

Controlled
```

---

# Backend ALB Rule Evaluation

Request:

```text
catalogue.backend-alb-dev.daws86s.fun
```

---

Rule:

```text
catalogue.*
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
cart.backend-alb-dev.daws86s.fun
```

↓

```text
Cart Target Group
```

---

# Backend Target Groups

Each service has its own target group.

Examples:

```text
Catalogue TG

User TG

Cart TG

Shipping TG

Payment TG
```

---

# Why Separate Target Groups?

Benefits:

```text
Independent Scaling

Independent Health Checks

Fault Isolation
```

---

# Catalogue Request Flow

```text
User
  ↓
Frontend ALB
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
```

---

# Database Communication

Catalogue Service:

```text
Reads Product Data
```

from:

```text
MongoDB
```

---

# Complete Request Flow

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
Catalogue TG
  ↓
Catalogue Service
  ↓
MongoDB
```

---

# Why Not Expose Catalogue Directly?

Wrong Architecture:

```text
User
  ↓
Catalogue Service
```

Problems:

```text
Security Risks

No Centralized Routing

No Load Balancing

No Health Checks
```

---

# Benefits of Frontend and Backend ALB Separation

```text
Improved Security

Traffic Isolation

Centralized Routing

Scalability

High Availability
```

---

# Production Example

Frontend Domain:

```text
roboshop-dev.daws86s.fun
```

Publicly accessible.

---

Backend Services:

```text
catalogue.backend-alb-dev.daws86s.fun

user.backend-alb-dev.daws86s.fun

cart.backend-alb-dev.daws86s.fun
```

Used internally by applications.

---

# Common Interview Questions

## Why Use Two Load Balancers?

### Short Answer

To separate user traffic from internal service traffic.

### Detailed Explanation

Frontend ALB handles external HTTPS requests, while Backend ALB manages internal microservice routing.

### Production Example

Users access roboshop-dev.daws86s.fun through Frontend ALB while API requests are routed through Backend ALB.

### Follow-Up Questions

- What are the security benefits?
- Can Backend ALB be internet-facing?

---

## Why Does Nginx Call Backend ALB?

### Short Answer

Backend ALB provides centralized routing for microservices.

### Detailed Explanation

Nginx forwards API requests to Backend ALB, which determines the correct target group and service.

### Production Example

/api/catalogue/categories forwarded to catalogue.backend-alb-dev.daws86s.fun.

### Follow-Up Questions

- Why not call service IPs directly?
- What happens during auto scaling?

---

## Why Use Wildcard DNS Records?

### Short Answer

To simplify DNS management for multiple services.

### Detailed Explanation

A single wildcard record can route requests for multiple backend services to the Backend ALB.

### Production Example

*.backend-alb-dev.daws86s.fun.

### Follow-Up Questions

- What is a wildcard DNS record?
- Can wildcard records be used in production?

---

## Why Is Backend ALB Usually Private?

### Short Answer

Because backend services should not be exposed directly to the internet.

### Detailed Explanation

Internal ALBs provide service routing while reducing the attack surface.

### Production Example

Catalogue and Cart services accessible only within the VPC.

### Follow-Up Questions

- What are internal ALBs?
- How do frontend services access backend ALBs?

---

# Key Takeaways

```text
Frontend ALB handles external HTTPS traffic.

Backend ALB handles internal microservice routing.

Nginx serves static content and proxies API requests.

Backend ALB uses host-based routing.

Wildcard Route53 records simplify DNS management.

Each microservice has its own target group.

Health checks ensure traffic reaches healthy services.

Frontend and Backend ALB separation improves security and scalability.

This architecture is commonly used in production microservices environments.
```