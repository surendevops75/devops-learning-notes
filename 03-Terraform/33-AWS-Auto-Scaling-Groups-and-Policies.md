# AWS Auto Scaling Groups and Policies

## Introduction

Applications do not receive the same amount of traffic all the time.

Traffic may:

```text
Increase

Decrease

Spike Suddenly
```

If infrastructure remains fixed:

```text
Performance Issues

Resource Wastage

Higher Costs
```

may occur.

AWS provides:

```text
Auto Scaling Groups (ASG)
```

to automatically adjust the number of servers.

---

# Real World Analogy

Suppose a company has:

```text
10 Employees
```

Each employee works:

```text
6 Hours Per Day
```

Workload is normal.

---

# Increased Workload

Suddenly:

```text
Each Employee

↓

12 Hours Per Day
```

This indicates:

```text
Workload Increased
```

Management decides:

```text
Recruit More Employees
```

---

# Reduced Workload

Suppose employees are working:

```text
4 Hours Per Day
```

This indicates:

```text
Resources Are Underutilized
```

Management may:

```text
Release Some Employees
```

to reduce costs.

---

# AWS Equivalent

Employees:

```text
EC2 Instances
```

Recruit New Employees:

```text
Scale Out
```

Release Employees:

```text
Scale In
```

Management:

```text
Auto Scaling Group
```

---

# What is an Auto Scaling Group?

An Auto Scaling Group is a service that automatically manages EC2 instances.

Responsibilities:

```text
Launch New Instances

Terminate Instances

Maintain Desired Capacity

Replace Failed Instances
```

---

# Auto Scaling Components

```text
Launch Template

Auto Scaling Group

Scaling Policy

Target Group
```

---

# Launch Template

Defines:

```text
AMI

Instance Type

Security Group

IAM Role

Subnet
```

ASG uses Launch Template to create instances.

---

# Auto Scaling Group

Controls:

```text
Minimum Instances

Desired Instances

Maximum Instances
```

---

# Example

```text
Minimum = 2

Desired = 4

Maximum = 10
```

Meaning:

```text
Always Keep At Least 2

Normally Run 4

Never Exceed 10
```

---

# Desired Capacity

Desired capacity represents:

```text
Number Of Running Instances
```

Example:

```text
Desired = 3
```

ASG ensures:

```text
3 Instances Running
```

---

# Self Healing Capability

Suppose:

```text
3 Instances Running
```

One instance crashes.

Result:

```text
2 Instances Running
```

ASG detects:

```text
Desired Capacity Not Met
```

Action:

```text
Launch New Instance
```

---

# Production Example

Catalogue Service:

```text
Desired Capacity = 3
```

Instances:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

Catalogue-2 crashes.

ASG automatically creates:

```text
Catalogue-4
```

to maintain capacity.

---

# Scaling Policies

Scaling policies determine:

```text
When To Scale

How Much To Scale
```

---

# Scale Out

Scale Out means:

```text
Add More Instances
```

Example:

```text
CPU > 75%
```

Action:

```text
Launch New Instance
```

---

# Scale In

Scale In means:

```text
Remove Instances
```

Example:

```text
CPU < 75%
```

or another defined threshold.

Action:

```text
Terminate Unnecessary Instances
```

---

# Production Policy Example

```text
Average CPU > 75%

↓

Add 1 Instance
```

---

```text
Average CPU < 30%

↓

Remove 1 Instance
```

---

# Auto Scaling Workflow

```text
Traffic Increases
        ↓
CPU Increases
        ↓
Scaling Policy Triggered
        ↓
Launch Template Used
        ↓
New Instance Created
```

---

# Traffic Reduction Workflow

```text
Traffic Drops
       ↓
CPU Drops
       ↓
Scaling Policy Triggered
       ↓
Instance Removed
```

---

# Relationship with Target Groups

Auto Scaling instances are attached to:

```text
Target Group
```

---

# Example

Target Group:

```text
Catalogue Target Group
```

Current Members:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

---

ASG Creates:

```text
Catalogue-4
```

Automatically Added To:

```text
Catalogue Target Group
```

---

# Complete Production Flow

```text
User
   ↓
ALB
   ↓
Target Group
   ↓
ASG Instances
```

---

# Production Example

Request:

```text
catalogue.backend-alb-dev.daws86s.fun
```

Flow:

```text
Backend ALB
      ↓
Catalogue Target Group
      ↓
Auto Scaling Instances
```

---

# High Traffic Scenario

Current Instances:

```text
10 Catalogue Servers
```

Traffic suddenly increases.

CPU:

```text
85%
```

Policy:

```text
CPU > 75%
```

Action:

```text
Launch Additional Servers
```

Result:

```text
Catalogue-11

Catalogue-12

Catalogue-13
```

added automatically.

---
# Application Upgrade Strategies

Suppose:

```text
10 Catalogue Servers
```

are already running.

A new application version is released.

There are two approaches.

---

## Approach 1 - Upgrade Existing Servers

```text
Collect Instance IPs
       ↓
Connect To Every Server
       ↓
Deploy New Version
```

Example:

```ini
[catalogue]

catalogue-ip1

catalogue-ip2

catalogue-ip3
```

---

### Advantages

```text
Simple
```

---

### Disadvantages

```text
Configuration Drift

Manual Risk

Inconsistent Servers
```

---

## Approach 2 - AMI Based Upgrade

```text
Create New Server
       ↓
Configure Using Ansible
       ↓
Create AMI
       ↓
Refresh Auto Scaling Group
```

---

### Advantages

```text
Immutable Infrastructure

Consistent Configuration

Production Friendly
```

This is the preferred enterprise approach.

---

# Low Traffic Scenario

Traffic decreases.

CPU:

```text
20%
```

Policy:

```text
CPU < 30%
```

Action:

```text
Terminate Extra Servers
```

Cost optimized automatically.

---

# Benefits of Auto Scaling

```text
High Availability

Self Healing

Automatic Scaling

Cost Optimization

Reduced Manual Effort
```

---

# Common Interview Questions

## What is an Auto Scaling Group?

### Short Answer

An Auto Scaling Group automatically manages EC2 instances based on configured rules.

### Detailed Explanation

ASG maintains the desired number of instances, replaces failed instances, and scales infrastructure based on workload.

### Production Example

Catalogue service maintaining three running instances even when one server crashes.

### Follow-Up Questions

- What happens if an instance fails?
- Can ASG automatically replace unhealthy instances?

---

## What is the Difference Between Scale Out and Scale In?

### Short Answer

Scale Out adds instances. Scale In removes instances.

### Detailed Explanation

Scale Out occurs when workload increases, while Scale In occurs when workload decreases.

### Production Example

CPU > 75% launches new servers. CPU < 30% removes extra servers.

### Follow-Up Questions

- What metrics can trigger scaling?
- Can scaling happen automatically?

---

## What is Desired Capacity?

### Short Answer

Desired capacity is the number of instances ASG tries to maintain.

### Detailed Explanation

ASG continuously monitors instance count and launches replacements whenever desired capacity is not met.

### Production Example

Desired capacity of three ensures three Catalogue servers are always running.

### Follow-Up Questions

- What is minimum capacity?
- What is maximum capacity?

---

## How Does ASG Work with Load Balancers?

### Short Answer

ASG instances are registered automatically with Target Groups attached to Load Balancers.

### Detailed Explanation

When ASG creates or removes instances, Target Groups are automatically updated, allowing ALB to distribute traffic correctly.

### Production Example

New Catalogue instance created by ASG is automatically added to Catalogue Target Group.

### Follow-Up Questions

- Does ALB need manual updates?
- How does health check integration work?

---

## Why Do We Need Launch Templates with ASG?

### Short Answer

ASG uses Launch Templates to know how to create new instances.

### Detailed Explanation

Launch Templates provide AMI, instance type, security group, IAM role, and subnet configuration required for instance creation.

### Production Example

ASG launches Catalogue servers using a Launch Template containing catalogue-v2 AMI.

### Follow-Up Questions

- Can ASG work without Launch Templates?
- Can Launch Templates use custom AMIs?

---

# Key Takeaways

```text
Auto Scaling Groups automatically manage EC2 instances.

ASG supports self-healing and automatic recovery.

Launch Templates define how new instances are created.

Scale Out adds instances during high traffic.

Scale In removes instances during low traffic.

Desired Capacity determines the number of running instances.

ASG works closely with ALB and Target Groups.

CPU utilization is a common scaling metric.

Auto Scaling improves availability and reduces operational effort.

ASG helps balance performance and cost automatically.
```