# AWS Multi-AZ, Multi-Region & Disaster Recovery

# Chapter 1 - High Availability, Fault Tolerance & Disaster Recovery Fundamentals

Every production system is designed with one question in mind:

> **"What happens if something fails?"**

Failures are inevitable.

Servers fail.

Disks fail.

Availability Zones fail.

Entire AWS Regions can become unavailable.

Enterprise architects design systems assuming failures will occur, not hoping they won't.

This philosophy is called **Design for Failure**.

---

# Why High Availability Matters

Imagine an online banking application.

If the application becomes unavailable for just 10 minutes,

it may result in

- Millions of dollars in lost transactions
- Customer dissatisfaction
- Regulatory penalties
- Reputation damage

Therefore, production applications must continue operating even during failures.

---

# Core Concepts

Enterprise resiliency is built on three major concepts.

```text
High Availability

↓

Fault Tolerance

↓

Disaster Recovery
```

Although these terms are related, they are not the same.

---

# What is High Availability (HA)?

High Availability means an application continues operating even if one or more components fail.

Example

```text
Load Balancer

↓

EC2-A

EC2-B
```

If EC2-A fails,

traffic automatically moves to EC2-B.

The application remains available.

---

# High Availability Characteristics

- Redundant Infrastructure
- Automatic Failover
- Minimal Downtime
- Multiple Availability Zones
- Health Checks
- Load Balancing

---

# What is Fault Tolerance?

Fault Tolerance means the system continues operating **without interruption**, even when components fail.

Example

```text
Application

↓

Server-1

↓

Failure

↓

Server-2

↓

No Downtime
```

Fault-tolerant systems are typically more expensive because redundancy exists everywhere.

---

# High Availability vs Fault Tolerance

| High Availability | Fault Tolerance |
|-------------------|-----------------|
| Small Downtime Possible | No Downtime |
| Automatic Recovery | Continuous Operation |
| Lower Cost | Higher Cost |
| Most AWS Applications | Mission-Critical Systems |

Most enterprise AWS workloads are designed for High Availability rather than full Fault Tolerance.

---

# What is Disaster Recovery (DR)?

Disaster Recovery is the ability to restore applications after a major disaster.

Disasters include

- Region Failure
- Data Center Fire
- Flood
- Cyber Attack
- Data Corruption
- Power Failure

Example

```text
Mumbai Region

↓

Unavailable

↓

Failover

↓

Singapore Region
```

Business operations continue from another location.

---

# Disaster Recovery Goals

Every DR strategy focuses on

- Business Continuity
- Minimal Data Loss
- Fast Recovery
- Automated Recovery
- Predictable Recovery Process

---

# Types of Failures

Enterprise architectures prepare for multiple failure levels.

```text
Application Failure

↓

Server Failure

↓

Rack Failure

↓

Availability Zone Failure

↓

Region Failure

↓

Global Disaster
```

Each failure requires a different recovery strategy.

---

# AWS Global Infrastructure

AWS infrastructure consists of

```text
Region

↓

Availability Zones

↓

Data Centers
```

Example

```text
Mumbai Region

├── AZ-A

├── AZ-B

└── AZ-C
```

Each Availability Zone is physically separated.

---

# What is an Availability Zone?

An Availability Zone (AZ) is one or more physically separate data centers within an AWS Region.

Characteristics

- Independent Power
- Independent Cooling
- Independent Networking
- High-Speed Interconnect

AZ failures are isolated from each other.

---

# What is an AWS Region?

A Region is a geographic location containing multiple Availability Zones.

Example

```text
Asia Pacific (Mumbai)

↓

AZ-A

AZ-B

AZ-C
```

Regions are isolated from one another.

---

# Multi-AZ Architecture

A Multi-AZ deployment distributes resources across Availability Zones within the same Region.

Example

```text
Application Load Balancer

↓

AZ-A

↓

EC2

────────────

AZ-B

↓

EC2
```

If one Availability Zone fails,

the application continues running.

---

# Multi-Region Architecture

A Multi-Region deployment distributes workloads across multiple AWS Regions.

Example

```text
Mumbai

↓

Production

────────────

Singapore

↓

Disaster Recovery
```

If an entire Region fails,

traffic can move to another Region.

---

# High Availability vs Disaster Recovery

```text
High Availability

↓

Availability Zone Failure

────────────────────────

Disaster Recovery

↓

Region Failure
```

HA protects against local failures.

DR protects against large-scale disasters.

---

# Design for Failure

AWS Well-Architected Framework recommends

```text
Assume Everything Can Fail
```

Instead of building systems that never fail,

build systems that recover automatically.

---

# Business Continuity

Business Continuity is broader than Disaster Recovery.

It includes

- Recovery Planning
- Communication
- Operations
- Personnel
- Infrastructure
- Applications

Disaster Recovery is one component of Business Continuity.

---

# Recovery Objectives

Two metrics determine Disaster Recovery success.

```text
RTO

Recovery Time Objective

↓

How Quickly Must We Recover?

────────────────────────

RPO

Recovery Point Objective

↓

How Much Data Can We Lose?
```

These concepts are explored in detail in later chapters.

---

# Example

Suppose an e-commerce platform defines

```text
RTO

30 Minutes

RPO

5 Minutes
```

Meaning

- Services must recover within 30 minutes.
- At most 5 minutes of data can be lost.

---

# Enterprise Banking Example

Architecture

```text
Users

↓

Route 53

↓

Mumbai

↓

Primary

↓

Replication

↓

Singapore

↓

Standby
```

If Mumbai becomes unavailable,

Route 53 redirects users to Singapore.

Business continues with minimal interruption.

---

# Typical Enterprise Workloads

Applications commonly designed for HA and DR

- Banking
- Healthcare
- E-commerce
- ERP
- Insurance
- Government
- Manufacturing
- SaaS Platforms

---

# Benefits

- Reduced Downtime
- Better Customer Experience
- Compliance
- Improved Reliability
- Business Continuity
- Faster Recovery
- Higher Availability

---

# Best Practices

- Design for failure.
- Use multiple Availability Zones.
- Automate recovery.
- Test disaster recovery plans regularly.
- Monitor health continuously.
- Replicate critical data.
- Define RTO and RPO before designing architecture.

---

# Common Mistakes

- Assuming a single Availability Zone is sufficient.
- Confusing High Availability with Disaster Recovery.
- Never testing failover.
- No documented recovery procedures.
- Depending on manual recovery.
- Ignoring business recovery requirements.

---

# Interview Questions

## Basic

- What is High Availability?
- What is Fault Tolerance?
- What is Disaster Recovery?
- Difference between Availability Zone and Region.

## Intermediate

- High Availability vs Disaster Recovery.
- Fault Tolerance vs High Availability.
- Explain Multi-AZ architecture.
- Explain Multi-Region architecture.

## Advanced

- Design a highly available architecture for an online banking platform.
- Explain how you would design an application to survive an Availability Zone failure and an entire AWS Region failure.
- Design an enterprise Disaster Recovery strategy for a mission-critical application with strict business continuity requirements.

---

