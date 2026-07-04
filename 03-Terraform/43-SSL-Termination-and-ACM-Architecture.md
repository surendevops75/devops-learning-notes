# SSL Termination and ACM Architecture

## Introduction

In the previous topic, we learned:

```text
HTTPS

SSL/TLS

Certificates

Public Keys

Private Keys
```

Now the question is:

```text
Where Should HTTPS Be Implemented?
```

In AWS production environments, HTTPS is usually terminated at:

```text
Application Load Balancer (ALB)
```

This is called:

```text
SSL Termination
```

---

# What is SSL Termination?

SSL Termination means:

```text
ALB Handles HTTPS Communication
```

instead of backend servers.

---

# Without SSL Termination

Every server must:

```text
Store Certificates

Store Private Keys

Handle Encryption

Handle Decryption
```

---

# Problems

```text
Certificate Management Complexity

Higher CPU Usage

Difficult Maintenance
```

---

# Solution

Move SSL processing to:

```text
Frontend Load Balancer
```

---

# SSL Termination Flow

```text
Client
    ↓ HTTPS
Frontend ALB
    ↓ HTTP/HTTPS
Backend Services
```

---

# What Happens?

Client communication:

```text
Encrypted
```

---

Load Balancer:

```text
Decrypts Traffic
```

---

Backend receives:

```text
Normal Traffic
```

or

```text
Encrypted Traffic
```

depending on architecture.

---

# Why Use SSL Termination?

Benefits:

```text
Centralized Certificate Management

Simplified Operations

Reduced Server Load

Easier Renewals
```

---

# Domain Requirement

HTTPS requires:

```text
Valid Domain
```

Example:

```text
daws86s.fun
```

---

# Environment Examples

Development:

```text
roboshop-dev.daws86s.fun
```

---

Production:

```text
daws86s.fun
```

---

# Why Domain is Mandatory?

Certificates are issued against:

```text
Domain Names
```

---

Example:

```text
daws86s.fun
```

Certificate belongs to:

```text
daws86s.fun
```

---

# AWS Certificate Manager (ACM)

AWS provides:

```text
ACM
```

Meaning:

```text
AWS Certificate Manager
```

---

# Purpose of ACM

Manage:

```text
SSL Certificates

Renewals

Validation
```

---

# Benefits

```text
No Manual Renewal

AWS Managed

Easy Integration

Highly Available
```

---

# ACM Workflow

```text
Request Certificate
        ↓
Validate Domain
        ↓
Certificate Issued
        ↓
Attach To ALB
```

---

# Domain Validation

AWS verifies:

```text
Do You Own The Domain?
```

---

# Validation Methods

```text
DNS Validation

Email Validation
```

---

# Preferred Method

```text
DNS Validation
```

because it is:

```text
Automatic

Reliable

Production Friendly
```

---

# ACM Certificate Example

Domain:

```text
daws86s.fun
```

---

Certificate Covers:

```text
daws86s.fun
```

---

Wildcard Example:

```text
*.daws86s.fun
```

---

Covers:

```text
catalogue.daws86s.fun

user.daws86s.fun

cart.daws86s.fun
```

---

# Route53 Integration

Route53 stores:

```text
DNS Records
```

---

Example:

```text
daws86s.fun
```

↓

```text
Frontend ALB
```

---

# Request Flow

```text
Browser
    ↓
Route53
    ↓
Frontend ALB
```

---

# HTTPS Listener

ALB Listener:

```text
443
```

receives HTTPS traffic.

---

Example:

```text
https://daws86s.fun
```

---

# Listener Configuration

Contains:

```text
Port 443

HTTPS Protocol

ACM Certificate
```

---

# Listener Rule Processing

Request:

```text
https://daws86s.fun
```

↓

```text
ALB Listener
```

↓

```text
Listener Rule
```

↓

```text
Target Group
```

---

# Production Example

Request:

```text
https://daws86s.fun
```

---

Flow:

```text
User
   ↓
Route53
   ↓
Frontend ALB
   ↓
HTTPS Listener 443
   ↓
Certificate Validation
   ↓
Listener Rule
   ↓
Frontend Target Group
   ↓
Frontend Instances
```

---

# Microservices Example

Frontend:

```text
https://daws86s.fun
```

---

Frontend calls:

```text
catalogue.backend-alb-dev.daws86s.fun

user.backend-alb-dev.daws86s.fun

cart.backend-alb-dev.daws86s.fun
```

---

Routing:

```text
ALB
   ↓
Rules
   ↓
Target Groups
   ↓
Services
```

---

# SSL Termination Example

Client:

```text
HTTPS
```

↓

```text
Frontend ALB
```

---

ALB decrypts traffic.

---

ALB forwards request to:

```text
Frontend Service
```

---

Backend services do not need:

```text
Public Certificates
```

---

# Why SSL Termination at ALB?

Imagine:

```text
20 Microservices
```

Without ALB termination:

```text
20 Certificates

20 Private Keys

20 Renewals
```

---

With ALB termination:

```text
1 Certificate

1 Renewal

Centralized Management
```

---

# Port Examples

Default HTTPS:

```text
https://daws86s.fun
```

uses:

```text
443
```

---

Explicit:

```text
https://daws86s.fun:443
```

---

Custom Port:

```text
https://daws86s.fun:445
```

---

# Complete Production Architecture

```text
User
   ↓
HTTPS 443
   ↓
Route53
   ↓
Frontend ALB
   ↓
ACM Certificate
   ↓
SSL Termination
   ↓
Listener Rules
   ↓
Target Groups
   ↓
Application Services
```

---

# Terraform Integration

Repository Example:

```text
terraform-aws-roboshop
```

---

Terraform Typically Creates

```text
ACM Certificate

Route53 Records

ALB

Listeners

Target Groups

Listener Rules
```

---

# Production Workflow

```text
Buy Domain
      ↓
Create Hosted Zone
      ↓
Request ACM Certificate
      ↓
Validate Domain
      ↓
Attach Certificate To ALB
      ↓
Create HTTPS Listener
      ↓
Configure Rules
      ↓
Serve Secure Traffic
```

---

# Benefits of SSL Termination

```text
Centralized Security

Easy Certificate Management

Reduced Backend Load

Simplified Operations

Improved Scalability
```

---

# Common Interview Questions

## What is SSL Termination?

### Short Answer

SSL Termination is the process where the Load Balancer handles HTTPS encryption and decryption.

### Detailed Explanation

The Load Balancer receives encrypted traffic, decrypts it, and forwards requests to backend services.

### Production Example

Frontend ALB handling HTTPS traffic for daws86s.fun.

### Follow-Up Questions

- Why is SSL termination used?
- Where is SSL termination usually implemented?

---

## Why Use ACM?

### Short Answer

ACM simplifies SSL certificate management in AWS.

### Detailed Explanation

ACM automates certificate issuance, validation, and renewal for AWS resources.

### Production Example

ACM certificate attached to a production ALB.

### Follow-Up Questions

- What validation methods does ACM support?
- Does ACM automatically renew certificates?

---

## Why Is a Domain Required for HTTPS?

### Short Answer

Certificates are issued for domain names and used to verify website identity.

### Detailed Explanation

Certificate Authorities validate ownership of domains before issuing certificates.

### Production Example

Certificate issued for daws86s.fun.

### Follow-Up Questions

- Can HTTPS work without a domain?
- Why do browsers validate domains?

---

## Why Is SSL Termination Done at ALB?

### Short Answer

To centralize certificate management and reduce backend complexity.

### Detailed Explanation

Instead of every service managing certificates, ALB handles HTTPS for all services.

### Production Example

One ALB certificate serving multiple microservices.

### Follow-Up Questions

- What are the advantages of ALB termination?
- Can backend services also use HTTPS?

---

## Explain the Complete HTTPS Architecture in AWS.

### Short Answer

Route53 → ALB → ACM Certificate → SSL Termination → Target Group → Application.

### Detailed Explanation

Users access a domain through Route53. Traffic reaches an HTTPS listener on ALB, where ACM certificates are used for SSL termination before requests are routed to backend services.

### Production Example

https://daws86s.fun served securely through an ALB using ACM.

### Follow-Up Questions

- What role does Route53 play?
- What role does ACM play?

---

# Key Takeaways

```text
HTTPS is usually terminated at the Application Load Balancer.

SSL Termination centralizes encryption and decryption.

Domains are required for production HTTPS implementations.

ACM manages SSL certificates in AWS.

Route53 directs domain traffic to ALBs.

HTTPS listeners typically use port 443.

Certificates are attached to ALB listeners.

ALB routes requests using listener rules and target groups.

SSL Termination reduces operational complexity.

This architecture is widely used in production AWS environments.
```