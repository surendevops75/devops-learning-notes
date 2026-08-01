# Amazon EC2 Auto Scaling

---

# Introduction

Amazon EC2 Auto Scaling is a fully managed AWS service that automatically adjusts the number of EC2 instances based on application demand.

Instead of manually launching or terminating EC2 instances during traffic fluctuations, Auto Scaling continuously monitors application health and CloudWatch metrics to ensure the desired number of healthy instances are always running.

Auto Scaling helps organizations build applications that are:

- Highly Available
- Fault Tolerant
- Scalable
- Cost Optimized
- Self-Healing

It is one of the most important services used in production AWS environments.

---

# What is Auto Scaling?

Amazon EC2 Auto Scaling automatically launches or terminates EC2 instances based on predefined scaling policies.

Instead of keeping a fixed number of servers running all the time, Auto Scaling adjusts capacity dynamically according to workload demand.

Example

```text
Morning Traffic

↓

2 EC2 Instances

↓

Afternoon Traffic

↓

8 EC2 Instances

↓

Night Traffic

↓

2 EC2 Instances
```

This ensures applications remain responsive while reducing infrastructure costs.

---

# Why Auto Scaling?

Imagine an online shopping application hosted on two EC2 instances.

Normal Days

```text
Users

↓

ALB

↓

EC2-1

EC2-2
```

During a flash sale:

```text
200 Users

↓

20,000 Users
```

Problems:

- High CPU Utilization
- Slow Response Time
- Request Timeouts
- Application Crash

Auto Scaling detects increased demand and automatically launches additional EC2 instances.

```text
Users

↓

ALB

↓

EC2-1

EC2-2

EC2-3

EC2-4

EC2-5
```

Once traffic decreases, unnecessary instances are automatically terminated.

---

# Real-World Problem

An e-commerce company experiences:

- Normal Traffic: 5,000 users/hour
- Festival Traffic: 500,000 users/hour

Requirements:

- No downtime
- Automatic scaling
- Cost optimization
- Zero manual intervention
- High availability

Amazon EC2 Auto Scaling fulfills all these requirements.

---

# Enterprise Architecture

```text
                         Internet

                             │

                        Amazon Route53

                             │

                 Application Load Balancer

                             │

                  Auto Scaling Group (ASG)

        ┌───────────────────┼───────────────────┐

        │                   │                   │

    EC2 Instance       EC2 Instance       EC2 Instance

        │                   │                   │

        └───────────────────┼───────────────────┘

                     CloudWatch Metrics

                             │

                   Scaling Policies

                             │

                Launch / Terminate Instances
```

---

# Internal Working

The Auto Scaling workflow consists of continuous monitoring and automated actions.

```text
CloudWatch Metrics

↓

Scaling Policy Evaluation

↓

Auto Scaling Group

↓

Launch Template

↓

Launch or Terminate EC2

↓

Register with ALB

↓

Serve Traffic
```

This cycle repeats continuously.

---

# Core Components

Amazon EC2 Auto Scaling consists of:

- Launch Template
- Launch Configuration (Legacy)
- Auto Scaling Group
- Scaling Policies
- CloudWatch Alarms
- Health Checks
- Lifecycle Hooks
- Warm Pools
- Instance Refresh

---

# Launch Template

A Launch Template defines how new EC2 instances should be created.

It contains:

- Amazon Machine Image (AMI)
- Instance Type
- Key Pair
- Security Groups
- IAM Role
- User Data
- Storage Configuration
- Network Configuration

Think of a Launch Template as a reusable blueprint for launching EC2 instances.

---

# Why Launch Templates?

Without Launch Templates:

Every EC2 launch requires manual configuration.

With Launch Templates:

```text
Scaling Event

↓

Launch Template

↓

Automatically Create EC2
```

This ensures consistency across all instances.

---

# Launch Template Components

Typical configuration includes:

- AMI ID
- Instance Type
- IAM Instance Profile
- Security Groups
- Root Volume
- User Data Script
- Monitoring
- Tags

---

# Launch Configuration (Legacy)

Launch Configurations were the original mechanism for defining EC2 launch settings.

Limitations:

- Cannot be modified
- Fewer features
- No versioning
- No Mixed Instance support

AWS recommends using Launch Templates for all new deployments.

---

# Auto Scaling Group (ASG)

An Auto Scaling Group manages a collection of EC2 instances.

Responsibilities:

- Launch instances
- Terminate instances
- Replace unhealthy instances
- Maintain desired capacity
- Distribute instances across Availability Zones

Architecture

```text
Auto Scaling Group

↓

Launch Template

↓

EC2-1

EC2-2

EC2-3
```

---

# Desired Capacity

Desired Capacity defines how many EC2 instances should currently be running.

Example

```
Desired = 3
```

If one instance fails:

```text
Running

↓

2 Instances

↓

Auto Scaling

↓

Launch New Instance

↓

Back to 3
```

This self-healing capability is one of the biggest advantages of Auto Scaling.

---

# Minimum Capacity

Defines the minimum number of instances that must always remain running.

Example

```
Minimum = 2
```

Even if traffic drops to zero:

```
2 EC2 Instances

Remain Running
```

---

# Maximum Capacity

Defines the maximum number of instances Auto Scaling is allowed to launch.

Example

```
Maximum = 10
```

Even during heavy traffic:

```
Never Exceeds

10 EC2 Instances
```

---

# Capacity Relationship

Example

```
Minimum = 2

Desired = 4

Maximum = 10
```

Rules:

- Running instances should be 4
- Never go below 2
- Never exceed 10

---

# Scaling Policies

Scaling Policies determine when Auto Scaling should launch or terminate instances.

AWS supports:

- Target Tracking Scaling
- Step Scaling
- Simple Scaling
- Scheduled Scaling
- Predictive Scaling

---

# Target Tracking Scaling

The most commonly used scaling policy.

You specify a target metric.

Example

```
Target CPU

50%
```

If CPU rises above 50%:

```
Launch EC2
```

If CPU falls below 50%:

```
Terminate EC2
```

AWS automatically calculates how many instances should be added or removed.

Recommended for most production workloads.

---

# Step Scaling

Step Scaling performs different scaling actions depending on metric thresholds.

Example

| CPU Utilization | Action |
|-----------------|--------|
| 60–70% | Add 1 Instance |
| 70–80% | Add 2 Instances |
| Above 80% | Add 4 Instances |

Useful when workloads increase rapidly.

---

# Simple Scaling

Simple Scaling performs one scaling action after a CloudWatch alarm is triggered.

Example

```
CPU > 70%

↓

Launch 1 EC2

↓

Cooldown Period
```

This policy is easier to configure but slower than Target Tracking.

---

# Scheduled Scaling

Scheduled Scaling adjusts capacity at specific times.

Example

```text
Monday

08:00 AM

↓

Increase to 10 Instances

-------------------------

11:00 PM

↓

Reduce to 2 Instances
```

Ideal for predictable workloads.

Examples:

- Office Applications
- Batch Processing
- Business Hours Traffic

---

# Predictive Scaling

Predictive Scaling uses machine learning to forecast future traffic.

Instead of reacting after CPU increases:

```
Traffic Predicted

↓

Launch EC2

↓

Traffic Arrives
```

Benefits

- Faster response
- Better performance
- Reduced latency

Suitable for applications with predictable usage patterns.

---

# Dynamic Scaling

Dynamic Scaling automatically adjusts capacity based on real-time CloudWatch metrics.

Metrics commonly used:

- CPU Utilization
- Memory Utilization (Custom Metric)
- Request Count
- Network In
- Network Out
- ALB Request Count Per Target

Example

```text
CloudWatch

↓

CPU 75%

↓

Scaling Policy

↓

Launch EC2

↓

Register with ALB
```

---

# Cooldown Period

After a scaling activity completes, Auto Scaling waits before performing another scaling action.

Purpose:

- Prevent rapid scaling
- Avoid unnecessary instance launches
- Allow applications time to stabilize

Example

```
Scale Out

↓

Wait

300 Seconds

↓

Evaluate Again
```

---

# Health Checks

Auto Scaling continuously monitors EC2 instance health.

Supports two health check types:

- EC2 Health Check
- ELB Health Check

If an instance becomes unhealthy:

```text
EC2 Failed

↓

Terminate

↓

Launch Replacement

↓

Register with ALB
```

Applications remain available without manual intervention.

---

# EC2 Health Check

Checks whether the EC2 instance itself is healthy.

Examples:

- Instance Status
- System Status
- Hardware Failure
- Operating System Crash

---

# Elastic Load Balancer Health Check

ELB verifies that the application is actually responding.

Example

```
GET /health

↓

HTTP 200

↓

Healthy
```

If the application fails but the EC2 instance is still running, ELB marks it unhealthy and Auto Scaling replaces it.

---

# Lifecycle Hooks

Lifecycle Hooks allow you to pause an instance during launch or termination.

Launch Example

```text
Launch EC2

↓

Wait

↓

Install Software

↓

Configuration

↓

Complete Lifecycle

↓

Register with ALB
```

Termination Example

```text
Terminate Instance

↓

Pause

↓

Upload Logs

↓

Backup Data

↓

Complete

↓

Terminate
```

Lifecycle Hooks are commonly used for:

- Configuration Management
- Log Collection
- Backup
- Notifications
- Monitoring

---

