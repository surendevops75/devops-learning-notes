# Blue-Green Deployment Strategy

## Introduction

One of the biggest challenges in production environments is:

```text
Deploying Changes

Without Downtime
```

---

Users expect:

```text
24/7 Availability
```

---

Organizations cannot afford:

```text
Application Downtime

Customer Impact

Revenue Loss
```

---

To solve this problem:

```text
Blue-Green Deployment
```

is used.

---

# What is Blue-Green Deployment?

Blue-Green Deployment is a:

```text
Zero Downtime Deployment Strategy
```

---

Two identical environments exist:

```text
Blue Environment

Green Environment
```

---

One environment serves traffic.

---

The other environment is prepared for:

```text
Testing

Validation

Deployment
```

---

# Why Blue-Green?

Traditional Deployment

```text
Stop Old Version

Deploy New Version

Start Application
```

---

Problem

```text
Downtime
```

---

Blue-Green Deployment

```text
Blue Running

Green Prepared

Switch Traffic

Remove Blue
```

---

Result

```text
Near Zero Downtime
```

---

# Understanding Blue and Green

## Blue Environment

Current Production.

---

Example

```text
Catalogue v4.0

Running In Production
```

---

Users access:

```text
Blue
```

---

## Green Environment

New Version.

---

Example

```text
Catalogue v5.0
```

---

Prepared and tested.

---

Not receiving production traffic initially.

---

# Basic Architecture

Before Deployment

```text
Users

↓

Blue Environment

v4.0
```

---

Green Exists

```text
Green

v5.0
```

---

No Traffic.

---

# Traffic Switch

After Validation

```text
Users

↓

Green Environment

v5.0
```

---

Blue becomes standby.

---

# Real Production Example

Current Application

```text
Catalogue v4.0
```

---

Running On

```text
Blue
```

---

New Version

```text
Catalogue v5.0
```

---

Deploy To

```text
Green
```

---

Validate

```text
Login

Search

Checkout

Payment
```

---

Traffic Switch

```text
Blue

↓

Green
```

---

Upgrade Complete.

---

# Blue-Green Workflow

```text
Blue Running

↓

Create Green

↓

Deploy New Version

↓

Validate

↓

Switch Traffic

↓

Monitor

↓

Delete Blue
```

---

# Key Characteristic

Both environments have:

```text
Same Capacity

Same Infrastructure

Same Configuration
```

---

Example

Blue

```text
5 Servers
```

---

Green

```text
5 Servers
```

---

Total

```text
10 Servers
```

---

Temporary increase in cost.

---

# Blue-Green in EKS

Blue

```text
Node Group A

Kubernetes 1.32
```

---

Green

```text
Node Group B

Kubernetes 1.33
```

---

Workloads migrate:

```text
Blue NG

↓

Green NG
```

---

# Blue-Green for Application Upgrades

Current

```text
Catalogue v4.0
```

---

New Version

```text
Catalogue v5.0
```

---

Deploy

```text
Green
```

---

Validate

```text
Smoke Testing

Regression Testing
```

---

Switch Traffic.

---

# Blue-Green for Platform Upgrades

Current

```text
EKS 1.32
```

---

New Environment

```text
EKS 1.33
```

---

Move Workloads.

---

Retire Old Environment.

---

# Blue-Green for Database Upgrades

Current

```text
MySQL 8.0
```

---

Green

```text
MySQL 8.4
```

---

Replicate Data.

---

Validate.

---

Switch Applications.

---

# Traffic Switching Methods

Traffic can be switched using:

```text
Load Balancer

DNS

Ingress

Service Selector
```

---

# Load Balancer Switch

Before

```text
ALB

↓

Blue
```

---

After

```text
ALB

↓

Green
```

---

# DNS Switch

Before

```text
app.company.com

↓

Blue
```

---

After

```text
app.company.com

↓

Green
```

---

# Kubernetes Service Switch

Before

```text
Service

↓

Blue Pods
```

---

After

```text
Service

↓

Green Pods
```

---

# Validation Before Traffic Switch

Check:

```text
Pods

Services

Ingress

Storage

Database Connectivity
```

---

Verify:

```text
Application Functionality
```

---

# Smoke Testing

Basic Validation

```text
Login

Search

API Calls
```

---

# Performance Testing

Check:

```text
Latency

Response Time

Errors
```

---

# Rollback Advantage

Suppose Green fails.

---

Current State

```text
Green Active
```

---

Issue Found

```text
Application Crash
```

---

Rollback

```text
Green

↓

Blue
```

---

Fast Recovery.

---

# Why Rollback Is Easy?

Old Environment Still Exists.

---

No Need To:

```text
Redeploy

Rebuild

Reconfigure
```

---

Simply switch traffic.

---

# Advantages

## Zero Downtime

Users continue accessing applications.

---

## Easy Rollback

Switch traffic back.

---

## Safe Testing

Validate before production.

---

## Reduced Risk

Production impact minimized.

---

# Disadvantages

## Double Infrastructure Cost

Blue

+

Green

---

Need duplicate resources.

---

## Database Challenges

Applications easy.

---

Databases harder.

---

Need:

```text
Replication

Migration Planning
```

---

## Operational Complexity

Requires:

```text
Planning

Validation

Monitoring
```

---

# Production Example

E-Commerce Application

Blue

```text
Frontend v4

Catalogue v4

Cart v4
```

---

Green

```text
Frontend v5

Catalogue v5

Cart v5
```

---

Testing Completed.

---

Traffic Shifted.

---

Users now access:

```text
Green
```

---

# Banking Example

Blue

```text
NetBanking v10
```

---

Green

```text
NetBanking v11
```

---

Validation

```text
Balance Check

Fund Transfer

Statements
```

---

Traffic Switch.

---

# Common Problems

## Green Not Fully Tested

Result

```text
Production Issues
```

---

Always validate.

---

# Database Incompatibility

Application upgraded.

---

Database not upgraded.

---

Failures occur.

---

# Configuration Drift

Blue

```text
Config A
```

---

Green

```text
Config B
```

---

Unexpected behavior.

---

# Capacity Planning Issues

Blue

```text
5 Nodes
```

---

Green

```text
2 Nodes
```

---

Insufficient capacity.

---

# Monitoring After Switch

Monitor:

```text
CPU

Memory

Errors

Latency

Logs
```

---

# Success Criteria

```text
No Errors

Healthy Pods

Healthy ALB

Successful Transactions
```

---

# Common Interview Questions

## What is Blue-Green Deployment?

### Short Answer

Blue-Green Deployment is a zero-downtime deployment strategy using two identical environments.

### Detailed Explanation

One environment serves production traffic while the other is used for deploying and validating changes. Traffic is switched after successful testing.

### Production Example

Catalogue v4 running on Blue while Catalogue v5 is deployed and tested on Green.

### Follow-Up Questions

- Why is it called Blue-Green?
- How is traffic switched?
- What are the rollback benefits?

---

## Why Use Blue-Green Deployment?

### Short Answer

To minimize downtime and deployment risk.

### Detailed Explanation

Blue-Green allows validation before production traffic is switched.

### Production Example

EKS upgrade from 1.32 to 1.33 using Blue and Green node groups.

### Follow-Up Questions

- How does rollback work?
- Is downtime eliminated completely?
- What are the costs?

---

## What Are the Advantages of Blue-Green?

### Short Answer

Zero downtime, easy rollback, and safer deployments.

### Detailed Explanation

The old environment remains available until the new environment is validated.

### Production Example

Rollback from Green to Blue after discovering application errors.

### Follow-Up Questions

- Why is rollback fast?
- What risks still exist?
- How is validation performed?

---

## What Are the Disadvantages of Blue-Green?

### Short Answer

Higher infrastructure cost and operational complexity.

### Detailed Explanation

Both environments run simultaneously during deployment.

### Production Example

Maintaining two EKS node groups with identical capacity during upgrades.

### Follow-Up Questions

- How much extra capacity is needed?
- What database challenges exist?
- When should Blue-Green not be used?

---

## Have You Used Blue-Green Deployment in Production?

### Short Answer

Yes, for EKS upgrades and application releases.

### Detailed Explanation

We created a Green environment, validated workloads, migrated traffic, and then decommissioned the Blue environment after successful verification.

### Production Example

EKS cluster upgrade from 1.32 to 1.33 using Blue and Green node groups.

### Follow-Up Questions

- How did you validate Green?
- How did you move traffic?
- What rollback plan existed?

---

# Key Takeaways

```text
Blue-Green Deployment is a zero-downtime deployment strategy.

Two identical environments exist.

Blue = Current Production.

Green = New Version.

Traffic is switched after validation.

Rollback is fast because Blue remains available.

Used for application upgrades, EKS upgrades, platform upgrades, and database upgrades.

Provides safer releases with minimal customer impact.

Requires additional infrastructure during deployment.

One of the most common production deployment strategies.
```