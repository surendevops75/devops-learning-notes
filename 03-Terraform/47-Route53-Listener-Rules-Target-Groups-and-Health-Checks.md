# Route53, Listener Rules, Target Groups and Health Checks

## Introduction

Modern applications require intelligent traffic routing.

A request should reach:

```text
Correct Service

Healthy Server

Correct Environment
```

AWS provides this functionality using:

```text
Route53

Load Balancer Listeners

Listener Rules

Target Groups

Health Checks
```

These components work together to ensure high availability and proper traffic routing.

---

# Request Flow Overview

```text
User
  ↓
Route53
  ↓
Load Balancer
  ↓
Listener
  ↓
Rule Evaluation
  ↓
Target Group
  ↓
Healthy Instance
```

---

# What is Route53?

Route53 is AWS's:

```text
Managed DNS Service
```

Its job is to convert:

```text
Domain Name
```

into:

```text
IP Address

or

Load Balancer DNS Name
```

---

# Example

User opens:

```text
https://roboshop-dev.daws86s.fun
```

---

Browser asks:

```text
Where Is roboshop-dev.daws86s.fun?
```

---

Route53 responds:

```text
frontend-alb.amazonaws.com
```

---

# Route53 Flow

```text
User
  ↓
DNS Query
  ↓
Route53
  ↓
Frontend ALB
```

---

# Frontend DNS Record

Example:

```text
roboshop-dev.daws86s.fun
```

↓

```text
Frontend ALB
```

---

# Backend Wildcard Record

Example:

```text
*.backend-alb-dev.daws86s.fun
```

↓

```text
Backend ALB
```

---

# Why Use Wildcard Records?

Without wildcard:

```text
catalogue.backend-alb-dev.daws86s.fun

user.backend-alb-dev.daws86s.fun

cart.backend-alb-dev.daws86s.fun

shipping.backend-alb-dev.daws86s.fun
```

Need separate records.

---

With wildcard:

```text
*.backend-alb-dev.daws86s.fun
```

One record handles all services.

---

# Load Balancer Listener

A Listener waits for incoming traffic.

Examples:

```text
HTTP 80

HTTPS 443
```

---

# Frontend Listener

```text
HTTPS
```

Port:

```text
443
```

---

# Backend Listener

```text
HTTP
```

Port:

```text
80
```

---

# What Happens After Listener Receives Traffic?

Listener evaluates:

```text
Rules
```

---

# Listener Rules

Rules determine:

```text
Where Traffic Should Go
```

---

# Example

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

# Another Example

Request:

```text
cart.backend-alb-dev.daws86s.fun
```

↓

```text
Cart Target Group
```

---

# Rule Evaluation Flow

```text
Request
   ↓
Listener
   ↓
Rule Matching
   ↓
Target Group
```

---

# What is a Target Group?

A Target Group is:

```text
Collection Of Backend Servers
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

# Cart Target Group

Contains:

```text
Cart-1

Cart-2

Cart-3
```

---

# Why Use Target Groups?

Benefits:

```text
Load Distribution

Health Checks

Auto Scaling Integration

High Availability
```

---

# Traffic Distribution

Incoming requests are distributed across:

```text
Healthy Instances
```

inside the target group.

---

# Health Checks

Health Checks determine:

```text
Is Service Healthy?
```

---

# Why Are Health Checks Required?

Suppose:

```text
Catalogue-2
```

crashes.

Without health checks:

```text
Traffic Still Sent
```

Result:

```text
Application Errors
```

---

# Solution

Load Balancer continuously checks:

```text
Health Endpoint
```

---

# Example

```text
http://catalogue.backend-alb-dev.daws86s.fun/health
```

---

# Health Check Flow

```text
ALB
  ↓
Health Endpoint
  ↓
Success/Failure
```

---

# Healthy Instance

Example Response:

```text
200 OK
```

---

Result:

```text
Instance Remains Active
```

---

# Unhealthy Instance

Example:

```text
500 Error

Connection Refused

Timeout
```

---

Result:

```text
Removed From Traffic
```

---

# Traffic Routing Example

User opens:

```text
https://roboshop-dev.daws86s.fun/api/catalogue/categories
```

---

Flow

```text
User
  ↓
Route53
  ↓
Frontend ALB
  ↓
Frontend Nginx
  ↓
catalogue.backend-alb-dev.daws86s.fun
  ↓
Backend ALB
  ↓
Catalogue Rule
  ↓
Catalogue TG
  ↓
Healthy Catalogue Instance
```

---

# Health Check Benefits

```text
Automatic Failure Detection

Automatic Traffic Removal

Improved Availability

Reduced Downtime
```

---

# Production Scenario

Suppose:

```text
Catalogue-2
```

fails.

Health Check:

```text
Failed
```

---

ALB Action:

```text
Remove Catalogue-2
```

---

Remaining:

```text
Catalogue-1

Catalogue-3
```

continue serving traffic.

---

# Auto Scaling Integration

Suppose:

```text
Catalogue-2
```

fails permanently.

---

ASG creates:

```text
Catalogue-4
```

---

Health Check:

```text
Passed
```

---

Target Group:

```text
Automatically Registers
```

new instance.

---

# Complete Production Architecture

```text
User
  ↓
Route53
  ↓
Frontend ALB
  ↓
HTTPS Listener
  ↓
Frontend TG
  ↓
Frontend
  ↓
Backend ALB
  ↓
HTTP Listener
  ↓
Listener Rule
  ↓
Target Group
  ↓
Healthy Instance
```

---

# Common Interview Questions

## What is Route53?

### Short Answer

Route53 is AWS's managed DNS service.

### Detailed Explanation

It resolves domain names to AWS resources such as Load Balancers and EC2 instances.

### Production Example

roboshop-dev.daws86s.fun pointing to Frontend ALB.

### Follow-Up Questions

- What is DNS?
- What is a Hosted Zone?

---

## What is a Listener?

### Short Answer

A Listener waits for incoming traffic on a specific port.

### Detailed Explanation

Listeners receive requests and evaluate rules before forwarding traffic to target groups.

### Production Example

Frontend ALB listener on port 443.

### Follow-Up Questions

- Can ALB have multiple listeners?
- Difference between HTTP and HTTPS listeners?

---

## What is a Listener Rule?

### Short Answer

A Listener Rule determines where traffic should be forwarded.

### Detailed Explanation

Rules evaluate hostnames or paths and route requests to appropriate target groups.

### Production Example

catalogue.backend-alb-dev.daws86s.fun routed to Catalogue TG.

### Follow-Up Questions

- What is host-based routing?
- What is path-based routing?

---

## What is a Target Group?

### Short Answer

A Target Group is a collection of backend servers.

### Detailed Explanation

Load Balancers distribute traffic across healthy instances within target groups.

### Production Example

Catalogue Target Group containing multiple Catalogue servers.

### Follow-Up Questions

- Can ASGs register automatically?
- How are health checks configured?

---

## What is a Health Check?

### Short Answer

A Health Check determines whether an instance is healthy and capable of serving traffic.

### Detailed Explanation

ALB continuously monitors health endpoints and removes unhealthy instances from traffic.

### Production Example

Catalogue service health endpoint returning HTTP 200.

### Follow-Up Questions

- What happens when a health check fails?
- What are common health check endpoints?

---

# Key Takeaways

```text
Route53 resolves domains to AWS resources.

Listeners receive traffic on specific ports.

Listener Rules determine traffic routing.

Target Groups contain backend servers.

Health Checks verify service availability.

Unhealthy instances are automatically removed from traffic.

Wildcard DNS records simplify backend service routing.

Target Groups integrate with Auto Scaling Groups.

These components form the foundation of highly available AWS architectures.

Understanding these concepts is essential for AWS and DevOps interviews.
```