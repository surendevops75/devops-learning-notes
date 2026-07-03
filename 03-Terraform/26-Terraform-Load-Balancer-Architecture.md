# Terraform Load Balancer Architecture

## Introduction

In production environments, users should not directly access EC2 instances.

Instead, traffic should flow through:

```text
Load Balancer
```

Benefits:

```text
High Availability

Traffic Distribution

Health Monitoring

SSL Termination

Host Based Routing

Path Based Routing
```

---

# Organization Analogy

To understand Load Balancer easily:

```text
Delivery Manager → Load Balancer

Manager → Listener

Team Lead → Rule

Team → Target Group

Team Member → Instance

Availability Check → Health Check
```

---

# Real World Example

Client Requirement:

```text
Enable WhatsApp Banking
```

Flow:

```text
Client
   ↓
Business Analyst
   ↓
Delivery Manager
   ↓
Manager
   ↓
Lead
   ↓
Team
   ↓
Team Member
```

---

# AWS Equivalent

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

# What is a Load Balancer?

A Load Balancer receives requests and forwards them to backend servers.

Responsibilities:

```text
Traffic Distribution

High Availability

Routing

Health Monitoring
```

---

# Load Balancer Components

```text
Load Balancer

Listener

Rule

Target Group

Health Check

Instance
```

---

# Frontend Load Balancer

Frontend applications are exposed to users through a public ALB.

Example:

```text
https://daws86s-dev.fun
```

Frontend ALB listens on:

```text
443
```

Protocol:

```text
HTTPS
```

---

# Frontend Traffic Flow

```text
User
   ↓
Frontend ALB
   ↓
Listener 443
   ↓
Rule
   ↓
Frontend Target Group
   ↓
Frontend Instance
```

---

# Frontend ALB DNS

AWS Generated DNS:

```text
frontend-alb-123456.us-east-1.elb.amazonaws.com
```

Route53 Alias Record:

```text
frontend-alb.daws86s.fun
```

or

```text
daws86s-dev.fun
```

points to:

```text
Frontend ALB
```

---

# What is a Listener?

Listener means:

```text
Whom Are We Listening To?
```

A Listener waits for incoming traffic on a specific port.

Examples:

```text
80

443

8080
```

---

# Listener Examples

HTTP:

```text
http://daws86s.fun
```

Port:

```text
80
```

---

HTTPS:

```text
https://daws86s.fun
```

Port:

```text
443
```

---

Custom Port:

```text
http://daws86s.fun:90
```

Port:

```text
90
```

---

# Backend Load Balancer

Backend services are generally accessed internally.

Example:

```text
backend-alb-dev.daws86s.fun
```

Listener:

```text
8080
```

Purpose:

```text
Route Requests To Backend Services
```

---

# What is a Rule?

Rules determine where traffic should go.

Example:

```text
catalogue.backend-alb-dev.daws86s.fun
```

Rule:

```text
Forward To Catalogue Target Group
```

---

Example:

```text
shipping.backend-alb-dev.daws86s.fun
```

Rule:

```text
Forward To Shipping Target Group
```

---

# Host Based Routing

Routing traffic based on hostname.

Example:

```text
catalogue.backend-alb-dev.daws86s.fun
```

↓

```text
Catalogue Target Group
```

---

Example:

```text
user.backend-alb-dev.daws86s.fun
```

↓

```text
User Target Group
```

---

Example:

```text
payment.backend-alb-dev.daws86s.fun
```

↓

```text
Payment Target Group
```

---

# Rule Flow

```text
Request
   ↓
Listener
   ↓
Rule Match
   ↓
Target Group
```

---

# What is a Target Group?

A Target Group is a collection of backend servers.

Example:

```text
Catalogue Target Group
```

Contains:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

---

# Team Analogy

```text
Team
```

contains:

```text
Member-1

Member-2

Member-3
```

AWS Equivalent:

```text
Target Group
```

contains:

```text
Instance-1

Instance-2

Instance-3
```

---

# What is a Health Check?

Health Check verifies application availability.

Example:

```text
http://catalogue.daws86s.fun:8080/health
```

---

# Health Check Flow

```text
ALB
  ↓
Health Check URL
  ↓
Application Response
```

---

Successful Response:

```text
200 OK
```

Status:

```text
Healthy
```

---

Failed Response:

```text
Connection Refused

500 Error

Timeout
```

Status:

```text
Unhealthy
```

---

# Consecutive Failures

Suppose ALB checks every:

```text
5 Seconds
```

and receives:

```text
Failure

Failure

Failure
```

Result:

```text
Target Marked Unhealthy
```

Traffic stops.

---

# Production Example

Health Check URL:

```text
http://catalogue.daws86s.fun:8080/health
```

Result:

```text
200 OK
```

Target Status:

```text
Healthy
```

---

If application crashes:

```text
500 Internal Server Error
```

ALB removes instance from rotation.

---

# Complete Backend Flow

Request:

```text
catalogue.backend-alb-dev.daws86s.fun
```

Flow:

```text
Backend ALB
      ↓
Listener 8080
      ↓
Rule
      ↓
Catalogue Target Group
      ↓
Healthy Catalogue Instance
```

---

# Complete Roboshop Architecture

```text
Internet
    ↓
Frontend ALB
    ↓
Frontend
    ↓
Backend ALB
    ↓
Catalogue
    ↓
MongoDB
```

---

# Production Scenario

User opens:

```text
https://daws86s-dev.fun
```

Flow:

```text
Frontend ALB
      ↓
Frontend Service
      ↓
Backend ALB
      ↓
Catalogue Service
      ↓
MongoDB
```

All components remain isolated and scalable.

---

# ALB Benefits

```text
High Availability

Auto Scaling Support

SSL Offloading

Health Monitoring

Host Based Routing

Traffic Distribution
```

---

# Common Interview Questions

## What is a Load Balancer?

### Short Answer

A Load Balancer distributes traffic across multiple backend servers.

### Detailed Explanation

It improves availability, fault tolerance, and scalability by routing requests only to healthy instances.

### Production Example

Frontend traffic distributed across multiple frontend EC2 instances.

### Follow-Up Questions

- Why use ALB instead of direct EC2 access?
- What are the types of AWS Load Balancers?

---

## What is a Listener?

### Short Answer

A Listener waits for requests on a specific port.

### Detailed Explanation

Listeners receive incoming traffic and evaluate rules before forwarding requests.

### Production Example

Frontend ALB listening on:

```text
443
```

Backend ALB listening on:

```text
8080
```

### Follow-Up Questions

- Can ALB have multiple listeners?
- Difference between 80 and 443?

---

## What is a Target Group?

### Short Answer

A Target Group is a collection of backend instances.

### Detailed Explanation

Load Balancer forwards traffic to target groups based on listener rules.

### Production Example

Catalogue Target Group containing multiple Catalogue EC2 instances.

### Follow-Up Questions

- Can one ALB have multiple target groups?
- Can a target group contain Auto Scaling instances?

---

## What is a Health Check?

### Short Answer

A Health Check verifies application availability.

### Detailed Explanation

ALB periodically checks configured endpoints and removes unhealthy targets from traffic routing.

### Production Example

```text
http://catalogue.daws86s.fun:8080/health
```

returns:

```text
200 OK
```

### Follow-Up Questions

- What happens if health checks fail?
- Can health check paths be customized?

---

## What is Host Based Routing?

### Short Answer

Routing traffic based on hostname.

### Detailed Explanation

ALB evaluates hostname rules and forwards traffic to the correct target group.

### Production Example

```text
catalogue.backend-alb-dev.daws86s.fun
```

↓

```text
Catalogue Target Group
```

### Follow-Up Questions

- Difference between host-based and path-based routing?
- Which routing strategy is commonly used in microservices?

---

# Key Takeaways

```text
ALB receives incoming requests.

Listeners wait on specific ports.

Rules decide where traffic should go.

Target Groups contain backend instances.

Health Checks determine availability.

Frontend ALB is public.

Backend ALB is private.

Host-based routing is commonly used in microservices.

ALB improves availability, scalability, and reliability.
```