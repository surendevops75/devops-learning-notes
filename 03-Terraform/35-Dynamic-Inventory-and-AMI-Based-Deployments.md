# Dynamic Inventory and AMI Based Deployments

## Introduction

In production environments, applications often run on multiple servers managed by Auto Scaling Groups.

Example:

```text
Catalogue-1

Catalogue-2

Catalogue-3

...

Catalogue-10
```

When a new application version is released, all servers must be updated.

There are two common approaches:

```text
1. Upgrade Existing Servers

2. AMI Based Deployment
```

---

# Problem with Static Inventory

Suppose Ansible inventory contains:

```ini
[catalogue]

catalogue-ip1

catalogue-ip2

catalogue-ip3
```

This works when servers are static.

---

# Challenges in Production

When Auto Scaling is enabled:

```text
Instances Are Created

Instances Are Terminated

IP Addresses Change
```

Maintaining inventory manually becomes difficult.

---

# What is Dynamic Inventory?

Dynamic Inventory allows Ansible to fetch server details automatically from cloud providers such as AWS.

Instead of:

```text
Manually Maintaining IP Addresses
```

Ansible queries AWS APIs and generates inventory dynamically.

---

# Dynamic Inventory Workflow

```text
Ansible
    ↓
AWS API
    ↓
Discover Instances
    ↓
Generate Inventory
    ↓
Execute Playbook
```

---

# Production Example

Current Instances:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

Auto Scaling creates:

```text
Catalogue-4

Catalogue-5
```

Dynamic Inventory automatically discovers them.

No manual inventory updates are required.

---

# Application Upgrade Strategy

Suppose:

```text
10 Catalogue Servers
```

are running in production.

A new application version is released.

---

# Approach 1 - Upgrade Existing Servers

Process:

```text
Get Instance IP Addresses
        ↓
Connect To Servers
        ↓
Deploy New Application Version
```

Inventory Example:

```ini
[catalogue]

catalogue-ip1

catalogue-ip2

catalogue-ip3
```

---

# Advantages

```text
Simple

Quick To Implement
```

---

# Disadvantages

```text
Configuration Drift

Different Server States

Difficult Rollback

Higher Operational Risk
```

---

# Approach 2 - AMI Based Deployment

Preferred production approach.

Process:

```text
Create Instance
       ↓
Configure Using Ansible
       ↓
Validate Application
       ↓
Stop Instance
       ↓
Create AMI
       ↓
Update Launch Template
       ↓
Refresh Auto Scaling Group
```

---

# Why AMI Based Deployments?

Benefits:

```text
Immutable Infrastructure

Consistent Servers

Easy Rollbacks

Predictable Deployments
```

---

# Base AMI

Organizations usually maintain a base image.

Example:

```text
devops-practice
```

Contains:

```text
Operating System

Common Packages

Security Configurations

Monitoring Agents
```

---

# Creating a Custom AMI

Step 1

```text
Create Catalogue Instance
```

---

Step 2

```text
Configure Using Ansible
```

Examples:

```text
Install Java

Deploy Catalogue Application

Configure Service
```

---

Step 3

```text
Validate Application
```

Ensure:

```text
Application Working

Health Checks Passing
```

---

Step 4

```text
Stop Instance
```

---

Step 5

```text
Create AMI
```

Example:

```text
catalogue-v1
```

---

# Launch Template

Launch Template defines:

```text
How New Instances Should Be Created
```

---

# Launch Template Configuration

Contains:

```text
Instance Type

Subnet

Security Group

AMI

Storage

IAM Role
```

---

# Example

```text
AMI = catalogue-v1

Instance Type = t3.micro

Subnet = private-subnet

Security Group = catalogue-sg
```

---

# Target Group

Target Group contains application servers.

Example:

```text
Catalogue Target Group
```

Members:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

---

# Auto Scaling Group

Auto Scaling Group uses:

```text
Launch Template
```

to create instances automatically.

---

# Flow

```text
Launch Template
       ↓
Auto Scaling Group
       ↓
EC2 Instances
       ↓
Target Group
```

---

# ASG Policy

Auto Scaling policies define:

```text
When To Add Servers

When To Remove Servers
```

---

# Scale Out Example

Condition:

```text
Average CPU > 75%
```

Action:

```text
Launch New Instance
```

---

# Scale In Example

Condition:

```text
Average CPU < 30%
```

Action:

```text
Terminate Instance
```

---

# Listener Rule

Application traffic reaches the Load Balancer first.

Example:

```text
catalogue.backend-alb-dev.daws86s.fun
```

---

# Routing Rule

```text
catalogue.backend-alb-dev.daws86s.fun
                 ↓
Catalogue Target Group
```

---

# Complete Production Architecture

```text
User
   ↓
Frontend
   ↓
Backend ALB
   ↓
Listener Rule
   ↓
Catalogue Target Group
   ↓
Auto Scaling Group
   ↓
Catalogue Instances
```

---

# End-to-End Deployment Pipeline

```text
Create Catalogue Instance
        ↓
Configure Using Ansible
        ↓
Stop Instance
        ↓
Create AMI
        ↓
Update Launch Template
        ↓
Auto Scaling Group
        ↓
Target Group
        ↓
Listener Rule
        ↓
Production Traffic
```

---

# Common Interview Questions

## What is Dynamic Inventory?

### Short Answer

Dynamic Inventory automatically discovers servers from AWS and generates inventory dynamically.

### Detailed Explanation

Instead of manually maintaining IP addresses, Ansible queries AWS APIs to identify running instances and build inventory automatically.

### Production Example

Auto Scaling creates new Catalogue servers which are automatically discovered by Ansible.

### Follow-Up Questions

- Why is static inventory difficult in cloud environments?
- What cloud providers support dynamic inventory?

---

## Why Are AMI Based Deployments Preferred?

### Short Answer

They provide consistent and repeatable deployments.

### Detailed Explanation

Applications and configurations are baked into an AMI, ensuring every server is identical.

### Production Example

Catalogue-v2 AMI used to replace all existing Catalogue instances through Auto Scaling.

### Follow-Up Questions

- What is immutable infrastructure?
- Why avoid configuring production servers manually?

---

## What Does a Launch Template Contain?

### Short Answer

A Launch Template contains instance creation settings.

### Detailed Explanation

It stores AMI, instance type, subnet, security group, IAM role, and storage configuration required to launch EC2 instances.

### Production Example

Catalogue Launch Template using catalogue-v1 AMI and catalogue security group.

### Follow-Up Questions

- Can Launch Templates be versioned?
- Can multiple ASGs use the same Launch Template?

---

## Explain the Complete Deployment Flow.

### Short Answer

Create → Configure → AMI → Launch Template → ASG → Target Group → ALB.

### Detailed Explanation

A configured EC2 instance is converted into an AMI. The AMI is referenced by a Launch Template, which is used by Auto Scaling Groups to launch instances that serve traffic through a Load Balancer.

### Production Example

Catalogue application deployed using AMI-based Auto Scaling architecture.

### Follow-Up Questions

- Where does Ansible fit into this flow?
- How are upgrades performed?

---

# Key Takeaways

```text
Dynamic Inventory automatically discovers cloud instances.

Static inventory is difficult to manage in Auto Scaling environments.

AMI based deployments support immutable infrastructure.

Applications are configured once and reused through AMIs.

Launch Templates define instance creation settings.

Auto Scaling Groups launch instances automatically.

Target Groups receive instances created by ASG.

Listener Rules route requests to the correct Target Group.

AMI based deployments provide consistency, scalability, and easy rollbacks.

This architecture is commonly used in production microservices environments.
```