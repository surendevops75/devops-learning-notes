# Rolling Deployments and Instance Refresh

## Introduction

Applications continuously receive:

```text
New Features

Bug Fixes

Security Updates

Performance Improvements
```

When a new version is released, it must be deployed without causing downtime.

---

# Traditional Deployment Problem

Suppose:

```text
10 Catalogue Servers
```

are currently serving production traffic.

To deploy a new version:

```text
Stop All Servers

Deploy New Version

Start All Servers
```

Problems:

```text
Downtime

User Impact

Business Loss
```

---

# Better Approach

Use:

```text
Rolling Deployment
```

---

# What is a Rolling Deployment?

A Rolling Deployment replaces old servers gradually.

Instead of replacing:

```text
10 Servers Together
```

replace:

```text
One Server At A Time
```

---

# Example

Current Environment:

```text
Catalogue-1

Catalogue-2

Catalogue-3

Catalogue-4

Catalogue-5

Catalogue-6

Catalogue-7

Catalogue-8

Catalogue-9

Catalogue-10
```

Current Version:

```text
catalogue-v1
```

---

# New Release

New Version:

```text
catalogue-v2
```

New AMI:

```text
catalogue-v2-ami
```

---

# Rolling Deployment Process

Step 1

```text
Create New Server
```

using:

```text
catalogue-v2-ami
```

---

Step 2

Health Check Passes.

```text
Healthy
```

---

Step 3

Remove One Old Server.

---

Step 4

Repeat Process.

---

# Result

Old:

```text
10 Servers
```

↓

New:

```text
10 Servers
```

No downtime.

---

# Visual Flow

```text
Create New Server
        ↓
Health Check
        ↓
Add To Target Group
        ↓
Remove Old Server
        ↓
Repeat
```

---

# Production Example

Current:

```text
Catalogue-1 (v1)

Catalogue-2 (v1)

Catalogue-3 (v1)
```

---

Create:

```text
Catalogue-4 (v2)
```

---

After Health Check:

```text
Remove Catalogue-1
```

---

Repeat Until:

```text
All Servers Running v2
```

---

# Why Health Checks Matter?

Suppose:

```text
New Version Has Issues
```

Without Health Checks:

```text
Bad Server Receives Traffic
```

---

With Health Checks:

```text
Server Not Healthy
```

↓

```text
Do Not Route Traffic
```

---

# Target Group Role

Target Group contains:

```text
Running Instances
```

Example:

```text
Catalogue Target Group
```

---

Old Deployment

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

---

During Rolling Deployment

```text
Catalogue-1

Catalogue-2

Catalogue-3

Catalogue-4
```

---

After Refresh

```text
Catalogue-2

Catalogue-3

Catalogue-4
```

---

# Complete Production Architecture

```text
User
   ↓
ALB
   ↓
Catalogue Target Group
   ↓
Auto Scaling Instances
```

---

# Deployment Workflow

```text
Create New AMI
        ↓
Update Launch Template
        ↓
Trigger Instance Refresh
        ↓
ASG Starts Replacing Servers
        ↓
Traffic Continues Normally
```

---

# Instance Refresh

AWS Auto Scaling provides:

```text
Instance Refresh
```

feature.

Purpose:

```text
Replace Existing Instances

With New Launch Template Version
```

---

# Why Instance Refresh?

Suppose:

```text
Launch Template Updated
```

with:

```text
New AMI
```

Existing instances:

```text
Still Running Old Version
```

---

Instance Refresh ensures:

```text
All Servers Eventually Run

New Version
```

---

# Production Example

Current Launch Template:

```text
catalogue-v1
```

---

Updated Launch Template:

```text
catalogue-v2
```

---

Instance Refresh:

```text
Replace Old Instances

One By One
```

---

Final Result:

```text
All Instances Running catalogue-v2
```

---

# Deployment Strategies

## Recreate Deployment

```text
Destroy All

Create New
```

Problems:

```text
Downtime
```

---

## Rolling Deployment

```text
Replace Gradually
```

Benefits:

```text
No Downtime

Safer Deployment
```

---

# Blue-Green Deployment

Two environments:

```text
Blue

Green
```

---

Traffic:

```text
Blue
```

Current Production.

---

Deploy:

```text
Green
```

New Version.

---

Switch Traffic:

```text
Blue → Green
```

---

Benefits:

```text
Fast Rollback

Minimal Risk
```

---

# Production Scenario

Current:

```text
10 Catalogue Servers

Version v1
```

New Release:

```text
Version v2
```

Process:

```text
Create New AMI
       ↓
Update Launch Template
       ↓
Start Instance Refresh
       ↓
Replace Instances One By One
       ↓
All Servers Running v2
```

---

# Benefits of Rolling Deployment

```text
No Downtime

Gradual Rollout

Easy Monitoring

Reduced Risk

Improved Availability
```

---

# Common Interview Questions

## What is a Rolling Deployment?

### Short Answer

A Rolling Deployment gradually replaces old servers with new servers.

### Detailed Explanation

Instead of updating all servers at once, instances are replaced in batches or one by one, reducing deployment risk and avoiding downtime.

### Production Example

Replacing ten Catalogue servers running v1 with ten servers running v2.

### Follow-Up Questions

- Why is rolling deployment preferred?
- What are the alternatives?

---

## What is Instance Refresh?

### Short Answer

Instance Refresh replaces existing Auto Scaling instances using the latest Launch Template version.

### Detailed Explanation

When Launch Templates are updated, existing instances continue using old configurations. Instance Refresh gradually replaces them with new instances.

### Production Example

Replacing all Catalogue v1 instances after updating the Launch Template to use catalogue-v2 AMI.

### Follow-Up Questions

- Does Instance Refresh cause downtime?
- How does AWS perform replacement?

---

## Why Create a New AMI for Every Release?

### Short Answer

To ensure all new servers are launched with the latest application version.

### Detailed Explanation

The application and configuration are baked into the AMI, making deployments consistent and repeatable.

### Production Example

catalogue-v2 AMI contains the latest application release and dependencies.

### Follow-Up Questions

- What are immutable deployments?
- Why not update servers manually?

---

## How Does ALB Help During Rolling Deployments?

### Short Answer

ALB sends traffic only to healthy instances.

### Detailed Explanation

Health checks ensure newly launched instances receive traffic only after they become healthy.

### Production Example

Catalogue-v2 instance joins the Target Group only after passing health checks.

### Follow-Up Questions

- What happens if health checks fail?
- Does ALB automatically remove unhealthy targets?

---

## Difference Between Rolling Deployment and Blue-Green Deployment?

### Short Answer

Rolling Deployment replaces servers gradually, while Blue-Green Deployment switches traffic between two environments.

### Detailed Explanation

Rolling deployments are resource-efficient, whereas Blue-Green deployments provide faster rollback and stronger isolation.

### Production Example

Rolling Deployment replaces one Catalogue server at a time. Blue-Green deployment runs two complete environments and switches traffic after validation.

### Follow-Up Questions

- Which deployment strategy is safer?
- Which strategy consumes more resources?

---

# Key Takeaways

```text
Applications require regular upgrades and releases.

Rolling Deployment replaces servers gradually.

Rolling Deployments minimize downtime.

Health Checks protect users from unhealthy deployments.

Target Groups automatically manage healthy instances.

Launch Templates define instance configuration.

Instance Refresh updates Auto Scaling instances to the latest version.

AMI-based deployments improve consistency.

Blue-Green Deployment provides fast rollback capabilities.

Rolling Deployment is one of the most commonly used production deployment strategies.
```