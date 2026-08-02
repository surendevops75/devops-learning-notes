# AWS Disaster Recovery (DR)

---

# Introduction

AWS Disaster Recovery (DR) is a collection of strategies, architectures, and AWS services that help organizations recover applications and data after infrastructure failures, natural disasters, cyberattacks, or regional outages while minimizing downtime and data loss.

Business continuity is critical for modern enterprises. Organizations must prepare for unexpected failures by designing resilient architectures and recovery strategies that meet Recovery Time Objective (RTO) and Recovery Point Objective (RPO) requirements.

AWS provides multiple disaster recovery strategies ranging from low-cost backup solutions to fully active multi-region deployments.

AWS Disaster Recovery integrates with

- Amazon EC2
- Amazon EBS
- Amazon RDS
- Amazon S3
- Amazon Route 53
- AWS Backup
- AWS Elastic Disaster Recovery (DRS)
- Amazon CloudFront
- Amazon EFS
- AWS Global Infrastructure

---

# What is Disaster Recovery?

Disaster Recovery is the process of restoring applications, infrastructure, and data after a disaster.

Goals

- Minimize Downtime
- Prevent Data Loss
- Maintain Business Continuity
- Restore Critical Services
- Protect Customer Data

Workflow

```text
Production

↓

Disaster

↓

Recovery Plan

↓

Restore Services

↓

Business Continuity
```

---

# Why Disaster Recovery?

Without DR

```text
Production Failure

↓

Application Offline

↓

Revenue Loss

↓

Business Impact
```

Problems

- Long Downtime
- Data Loss
- Customer Impact
- Financial Loss
- Compliance Issues

With DR

```text
Failure

↓

Automatic Recovery

↓

Minimal Downtime

↓

Business Continuity
```

---

# Real World Problem Statement

An enterprise operates

- Production Applications
- Multi-Tier Architecture
- Critical Databases
- Global Customers

Requirements

- High Availability
- Fast Recovery
- Minimal Data Loss
- Multi-Region Protection

AWS Disaster Recovery provides multiple recovery options.

---

# Enterprise Architecture

```text
Primary Region

EC2

RDS

S3

      │

Replication

      │

      ▼

Secondary Region

Standby Infrastructure

Backups

Monitoring
```

---

# Core Concepts

Disaster Recovery planning includes

- Recovery Time Objective (RTO)
- Recovery Point Objective (RPO)
- Backup Strategy
- Replication
- Failover
- Failback
- Testing
- Monitoring

---

# Recovery Time Objective (RTO)

RTO is the maximum acceptable downtime after a disaster.

Example

```text
Failure

↓

Recovery

↓

30 Minutes
```

RTO = 30 Minutes

Lower RTO requires more investment.

---

# Recovery Point Objective (RPO)

RPO is the maximum acceptable amount of data loss.

Example

```text
Last Backup

↓

Failure

↓

5 Minutes

↓

Recovery
```

RPO = 5 Minutes

Lower RPO requires more frequent replication.

---

# RTO vs RPO

| Metric | Meaning |
|---------|---------|
| RTO | Maximum Downtime |
| RPO | Maximum Data Loss |

---

# Disaster Recovery Strategies

AWS supports four primary DR strategies

- Backup and Restore
- Pilot Light
- Warm Standby
- Multi-Site Active/Active

Each strategy balances cost, complexity, and recovery speed.

---

# Backup and Restore

The simplest and lowest-cost strategy.

Architecture

```text
Production

↓

AWS Backup

↓

Amazon S3

↓

Restore During Disaster
```

Characteristics

- Lowest Cost
- Highest Recovery Time
- Suitable for Non-Critical Applications

---

# Pilot Light

A minimal version of the production environment runs continuously.

Architecture

```text
Production

↓

Continuous Replication

↓

Minimal Infrastructure

↓

Scale During Disaster
```

Benefits

- Lower Cost
- Faster Recovery than Backup
- Good Balance for Many Workloads

---

# Warm Standby

A scaled-down production environment runs continuously in another Region.

Architecture

```text
Primary Region

↓

Replication

↓

Standby Region

↓

Scale Up During Disaster
```

Benefits

- Faster Recovery
- Moderate Cost
- Improved Availability

---

# Multi-Site Active/Active

Applications run simultaneously in multiple Regions.

Architecture

```text
Users

↓

Route 53

↓

Region A

↓

Region B

↓

Both Active
```

Provides

- Lowest RTO
- Lowest RPO
- Highest Availability
- Highest Cost

---

# Summary

AWS Disaster Recovery provides multiple recovery strategies ranging from Backup and Restore to fully active multi-region architectures. By understanding Recovery Time Objective (RTO), Recovery Point Objective (RPO), and selecting the appropriate recovery model, organizations can minimize downtime, reduce data loss, and ensure business continuity during disasters.

---

