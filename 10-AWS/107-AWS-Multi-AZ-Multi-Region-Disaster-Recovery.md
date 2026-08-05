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

# Chapter 2 - AWS Availability Zones (AZ), Regions & Global Infrastructure (Deep Dive)

To design highly available AWS architectures, you must first understand AWS Global Infrastructure.

Many interview questions begin with

> "How does AWS achieve High Availability?"

The answer starts with

- Regions
- Availability Zones
- Edge Locations
- AWS Global Backbone

These components work together to provide resilient cloud infrastructure.

---

# AWS Global Infrastructure

AWS has one of the world's largest cloud infrastructures.

It consists of

```text
AWS Global Infrastructure

├── Regions

├── Availability Zones

├── Local Zones

├── Wavelength Zones

├── Edge Locations

└── Regional Edge Caches
```

Each component serves a different purpose.

---

# What is an AWS Region?

A Region is a physical geographic area where AWS operates cloud infrastructure.

Examples

- Mumbai
- Singapore
- Frankfurt
- London
- Virginia
- Oregon
- Sydney

Each Region is completely isolated from every other Region.

---

# Region Architecture

```text
AWS Region

├── Availability Zone A

├── Availability Zone B

├── Availability Zone C
```

Every Region contains multiple Availability Zones.

---

# Characteristics of an AWS Region

- Physically isolated
- Independent networking
- Independent power
- Independent control plane
- Independent services
- Multiple Availability Zones

A Region is considered an independent deployment boundary.

---

# Region Isolation

Suppose

```text
Mumbai Region
```

experiences a major outage.

It does **not** affect

```text
Singapore Region

London Region

Virginia Region
```

This isolation is why Multi-Region Disaster Recovery is possible.

---

# What is an Availability Zone?

An Availability Zone (AZ) is one or more physically separate data centers within a Region.

Each Availability Zone has

- Independent Power
- Independent Cooling
- Independent Networking
- Independent Physical Security

---

# Availability Zone Architecture

```text
Mumbai Region

├── AZ-A

│   ├── Data Center

│   └── Data Center

├── AZ-B

│   ├── Data Center

│   └── Data Center

└── AZ-C

    ├── Data Center

    └── Data Center
```

Each AZ can contain multiple data centers.

---

# Why Multiple Availability Zones?

Suppose an application runs only in

```text
AZ-A
```

If AZ-A fails,

```text
Application

↓

Unavailable
```

Instead,

deploy across multiple AZs.

```text
ALB

↓

AZ-A

↓

EC2

────────────

AZ-B

↓

EC2
```

If AZ-A fails,

AZ-B continues serving users.

---

# Availability Zone Connectivity

Availability Zones inside a Region are connected using

```text
High-Speed

Low-Latency

Private Fiber
```

Benefits

- Fast Replication
- Database Synchronization
- Application Communication

Applications can communicate between AZs with minimal latency.

---

# Region vs Availability Zone

| Region | Availability Zone |
|---------|-------------------|
| Geographic Area | Data Center Group |
| Multiple AZs | One or More Data Centers |
| Independent | Part of a Region |
| Disaster Recovery | High Availability |

---

# AWS Global Backbone

AWS operates its own private global network.

```text
Mumbai

↓

AWS Backbone

↓

Singapore

↓

Frankfurt

↓

Virginia
```

Traffic between Regions travels over AWS infrastructure,

not the public Internet.

---

# Benefits of AWS Backbone

- Lower Latency
- Higher Reliability
- Improved Security
- Better Performance
- Faster Replication

---

# Edge Locations

Edge Locations are different from Regions.

They are used primarily for

- Amazon CloudFront
- Route 53
- AWS Shield
- AWS WAF

Architecture

```text
Users

↓

Edge Location

↓

AWS Region
```

Content is served closer to users.

---

# Regional Edge Cache

Regional Edge Caches sit between

```text
Edge Location

↓

AWS Region
```

They improve cache efficiency for CloudFront.

---

# Local Zones

Local Zones bring AWS services closer to users.

Architecture

```text
Users

↓

Local Zone

↓

Parent Region
```

Use cases

- Gaming
- Video Editing
- Real-Time Analytics
- Media Production

---

# Wavelength Zones

Wavelength Zones integrate AWS with 5G networks.

Architecture

```text
Mobile Device

↓

5G Network

↓

Wavelength Zone

↓

AWS Region
```

Designed for ultra-low latency applications.

---

# Availability Zone Failure

Suppose

```text
AZ-A

↓

Power Failure
```

Application architecture

```text
ALB

↓

AZ-A

EC2

────────────

AZ-B

EC2
```

ALB health checks fail.

Traffic automatically moves to AZ-B.

Users experience little or no downtime.

---

# Region Failure

Suppose

```text
Mumbai

↓

Unavailable
```

Multi-Region architecture

```text
Route 53

↓

Singapore

↓

Application Available
```

Traffic is redirected to the healthy Region.

---

# Data Replication

Within a Region

```text
AZ-A

↓

AZ-B

↓

AZ-C
```

Data replication is fast due to low latency.

Across Regions

```text
Mumbai

↓

Singapore
```

Replication is asynchronous in most AWS services unless specifically configured otherwise.

---

# Choosing a Region

Factors

- Customer Location
- Compliance
- Latency
- AWS Service Availability
- Cost
- Disaster Recovery Requirements

Example

Indian customers often choose

```text
Mumbai
```

for lower latency.

---

# Enterprise Architecture

```text
Users

↓

Amazon Route 53

↓

Mumbai Region

├── AZ-A

├── AZ-B

└── AZ-C

↓

Replication

↓

Singapore Region

├── AZ-A

├── AZ-B

└── AZ-C
```

This architecture provides both High Availability and Disaster Recovery.

---

# Best Practices

- Deploy production workloads across multiple Availability Zones.
- Use Multi-Region for Disaster Recovery.
- Choose Regions close to users.
- Understand data residency requirements.
- Test Region failover regularly.
- Monitor Availability Zone health.

---

# Common Mistakes

- Deploying everything in one Availability Zone.
- Assuming Availability Zones are separate Regions.
- Confusing Edge Locations with Regions.
- Ignoring Region-specific service availability.
- No Multi-Region Disaster Recovery strategy.

---

# Interview Questions

## Basic

- What is an AWS Region?
- What is an Availability Zone?
- Region vs Availability Zone.

## Intermediate

- Why are Availability Zones connected with low latency?
- What is the AWS Global Backbone?
- What are Edge Locations?
- Local Zones vs Wavelength Zones.

## Advanced

- Design a highly available architecture using three Availability Zones.
- Explain how AWS achieves high availability using Regions and Availability Zones.
- Design a global application serving users from multiple continents while minimizing latency and ensuring disaster recovery.

---

# Chapter 3 - High Availability vs Fault Tolerance vs Disaster Recovery (Deep Dive)

One of the most commonly misunderstood AWS architecture topics is the difference between

- High Availability (HA)
- Fault Tolerance (FT)
- Disaster Recovery (DR)

Many engineers use these terms interchangeably, but in enterprise architecture they represent different design goals.

Understanding these concepts is essential for AWS Solutions Architect, DevOps Engineer, Platform Engineer, and Cloud Architect interviews.

---

# Relationship Between HA, FT & DR

```text
Business Continuity

        │

        ▼

 ┌─────────────────────┐
 │ High Availability   │
 └─────────────────────┘
            │
            ▼
 ┌─────────────────────┐
 │ Fault Tolerance     │
 └─────────────────────┘
            │
            ▼
 ┌─────────────────────┐
 │ Disaster Recovery   │
 └─────────────────────┘
```

Each solves a different type of failure.

---

# High Availability (HA)

High Availability means

> The application continues running with minimal interruption when individual infrastructure components fail.

Example

```text
Application Load Balancer

        │

 ┌──────┴──────┐

EC2-A       EC2-B

AZ-A        AZ-B
```

If

```text
EC2-A

↓

Fails
```

Traffic automatically shifts to

```text
EC2-B
```

Users continue using the application.

---

# Characteristics of High Availability

- Automatic failover
- Redundant resources
- Health checks
- Multiple Availability Zones
- Load balancing
- Small recovery time

---

# Typical AWS Services for HA

- Elastic Load Balancer
- Auto Scaling Groups
- Amazon RDS Multi-AZ
- Amazon EFS
- Amazon ECS
- Amazon EKS
- Route 53 Health Checks

---

# Example

An e-commerce application

```text
Users

↓

ALB

↓

EC2

AZ-A

────────────

EC2

AZ-B
```

If one EC2 instance fails,

ALB automatically routes traffic to healthy instances.

---

# Fault Tolerance (FT)

Fault Tolerance means

> The system continues operating with **zero interruption** despite failures.

Unlike High Availability,

there is no noticeable downtime.

---

# Fault Tolerant Architecture

```text
Application

↓

Server-A

↓

Failure

↓

Server-B

↓

Immediately Continues
```

Everything is duplicated.

---

# Characteristics

- No downtime
- No data loss
- Continuous processing
- Complete redundancy
- Immediate failover

---

# Where Fault Tolerance is Used

- Air Traffic Control
- Banking Core Systems
- Stock Exchanges
- Healthcare Systems
- Space Missions
- Nuclear Systems

These workloads cannot tolerate downtime.

---

# AWS Example

```text
Users

↓

Route53

↓

ALB

↓

Multiple AZs

↓

Synchronous Database Replication

↓

Multiple Storage Copies
```

Every component has redundancy.

---

# Why Fault Tolerance is Expensive

Everything exists twice.

Example

```text
Primary Server

↓

Running

Backup Server

↓

Also Running
```

Both consume infrastructure continuously.

---

# Disaster Recovery (DR)

Disaster Recovery focuses on recovering from

major failures.

Examples

- Entire Region Failure
- Earthquake
- Flood
- Cyber Attack
- Large-scale Power Failure
- Human Error

---

# Disaster Recovery Architecture

```text
Primary Region

↓

Mumbai

↓

Replication

↓

Singapore

↓

Standby
```

If Mumbai fails,

applications recover in Singapore.

---

# Disaster Recovery Characteristics

- Backup
- Replication
- Recovery Plans
- Automated Recovery
- Regional Failover
- Business Continuity

---

# High Availability vs Disaster Recovery

High Availability protects against

```text
EC2 Failure

AZ Failure

Storage Failure
```

Disaster Recovery protects against

```text
Entire Region Failure

Large Disaster

Data Corruption
```

---

# Fault Tolerance vs High Availability

High Availability

```text
Failure

↓

Small Recovery Time
```

Fault Tolerance

```text
Failure

↓

No Recovery Time

↓

Application Continues
```

---

# Recovery Time

Example

High Availability

```text
Failure

↓

20 Seconds

↓

Application Back
```

Fault Tolerance

```text
Failure

↓

0 Seconds

↓

Application Running
```

---

# Business Example

Online Shopping

Acceptable

```text
30 Seconds

Downtime
```

Suitable

```text
High Availability
```

---

Stock Exchange

Not Acceptable

```text
1 Second

Downtime
```

Suitable

```text
Fault Tolerance
```

---

Bank

Acceptable

```text
Region Failure

↓

Recover

↓

15 Minutes
```

Suitable

```text
Disaster Recovery
```

---

# Failure Types

```text
Application Crash

↓

HA

──────────────────

Server Failure

↓

HA

──────────────────

Availability Zone Failure

↓

HA

──────────────────

Region Failure

↓

DR

──────────────────

Natural Disaster

↓

DR

──────────────────

Disk Failure

↓

FT / HA

──────────────────

Power Failure

↓

HA / FT
```

---

# Enterprise Example

Payment Platform

```text
Users

↓

Route53

↓

ALB

↓

Mumbai

AZ-A

AZ-B

↓

Replication

↓

Singapore

↓

Standby
```

Failure

```text
EC2

↓

ALB Handles

──────────────

AZ

↓

Multi-AZ

──────────────

Region

↓

Disaster Recovery
```

Every failure has a predefined recovery mechanism.

---

# Decision Matrix

| Requirement | Solution |
|-------------|----------|
| Server Failure | High Availability |
| AZ Failure | Multi-AZ |
| Region Failure | Disaster Recovery |
| Zero Downtime | Fault Tolerance |
| Low Cost | High Availability |
| Mission Critical | Fault Tolerance |
| Regional Backup | Disaster Recovery |

---

# Enterprise Design Strategy

Large organizations combine all three.

```text
Application

↓

High Availability

↓

Fault Tolerant Components

↓

Disaster Recovery

↓

Business Continuity
```

No single strategy is sufficient for enterprise systems.

---

# Best Practices

- Design for failure.
- Deploy across multiple Availability Zones.
- Implement automated failover.
- Use Multi-Region Disaster Recovery for critical workloads.
- Define RTO and RPO before architecture design.
- Test recovery procedures regularly.
- Automate infrastructure provisioning.

---

# Common Mistakes

- Assuming High Availability protects against Region failure.
- Calling every redundant system Fault Tolerant.
- Never testing Disaster Recovery.
- Designing only for server failures.
- Ignoring business recovery objectives.
- Depending on manual recovery.

---

# Interview Questions

## Basic

- What is High Availability?
- What is Fault Tolerance?
- What is Disaster Recovery?

## Intermediate

- High Availability vs Fault Tolerance.
- High Availability vs Disaster Recovery.
- Why is Fault Tolerance more expensive?

## Advanced

- Design a banking application requiring zero downtime and multi-region disaster recovery.
- Explain how AWS services provide High Availability across Availability Zones.
- A company requires an RTO of 15 minutes and an RPO of 5 minutes. Which AWS architecture would you recommend and why?

---

# Chapter 4 - Recovery Time Objective (RTO), Recovery Point Objective (RPO) & Business Continuity

Every Disaster Recovery (DR) strategy starts with two business questions:

1. **How quickly should the application recover?**
2. **How much data can the business afford to lose?**

The answers define the entire DR architecture.

These two metrics are called

- **Recovery Time Objective (RTO)**
- **Recovery Point Objective (RPO)**

Every enterprise application has different RTO and RPO requirements based on business impact.

---

# Business Continuity

Business Continuity (BC) ensures that an organization can continue operating during and after a disruption.

It includes

- Disaster Recovery
- Backup Strategy
- Incident Response
- Communication Plan
- Recovery Procedures
- Business Operations
- Infrastructure Recovery

Architecture

```text
Business Continuity

├── Disaster Recovery

├── Backup

├── Recovery Plan

├── Incident Response

├── Monitoring

└── Testing
```

Disaster Recovery is one part of Business Continuity.

---

# What is RTO?

Recovery Time Objective (RTO) defines

> **The maximum acceptable time required to restore an application after a failure.**

Example

```text
Application Failure

↓

Recover

↓

20 Minutes
```

If the business defines

```text
RTO = 20 Minutes
```

the application must be restored within 20 minutes.

---

# RTO Timeline

```text
Application Failure

↓

Service Unavailable

↓

Recovery Activities

↓

Application Restored

<----- RTO ----->
```

The recovery process must finish before the RTO expires.

---

# Real Example

Online Shopping Website

Business Requirement

```text
Maximum Downtime

15 Minutes
```

Therefore

```text
RTO = 15 Minutes
```

Any recovery taking longer violates business requirements.

---

# What is RPO?

Recovery Point Objective (RPO) defines

> **The maximum acceptable amount of data loss during a disaster.**

Example

```text
Database Backup

↓

10:00 AM

Failure

↓

10:04 AM

Recovery
```

Maximum data loss

```text
4 Minutes
```

---

# RPO Timeline

```text
Backup

↓

New Transactions

↓

Failure

↓

Recovery

<--- RPO --->
```

The time between the last recoverable copy and the failure represents potential data loss.

---

# Real Example

Banking System

Requirement

```text
Maximum Data Loss

30 Seconds
```

Therefore

```text
RPO = 30 Seconds
```

The backup or replication strategy must ensure no more than 30 seconds of data can be lost.

---

# Understanding RTO

Suppose an application crashes.

```text
09:00

↓

Failure

↓

09:20

↓

Recovered
```

Recovery Time

```text
20 Minutes
```

If

```text
RTO

30 Minutes
```

Requirement is satisfied.

---

# Understanding RPO

Suppose

```text
Last Replication

09:55

↓

Failure

10:00
```

Possible Data Loss

```text
5 Minutes
```

If

```text
RPO

5 Minutes
```

Requirement is met.

---

# RTO vs RPO

| RTO | RPO |
|-----|-----|
| Recovery Time | Data Loss |
| Service Availability | Data Protection |
| Minutes/Hours | Seconds/Minutes/Hours |
| Focuses on Downtime | Focuses on Lost Data |

This is one of the most frequently asked interview questions.

---

# Low RTO

Example

```text
RTO

5 Minutes
```

Requires

- Automated Recovery
- Warm Standby
- Hot Standby
- Load Balancing
- Automation

---

# High RTO

Example

```text
RTO

24 Hours
```

Suitable for

- Internal Applications
- Reporting Systems
- Development Environments

---

# Low RPO

Example

```text
RPO

30 Seconds
```

Requires

- Continuous Replication
- Database Replication
- Streaming Replication
- Real-Time Synchronization

---

# High RPO

Example

```text
RPO

24 Hours
```

Suitable for

- Archive Systems
- Historical Reports
- Non-Critical Applications

---

# Different Business Requirements

## Banking

```text
RTO

5 Minutes

RPO

0–30 Seconds
```

---

## E-Commerce

```text
RTO

15 Minutes

RPO

5 Minutes
```

---

## Internal HR Portal

```text
RTO

4 Hours

RPO

1 Hour
```

---

## Development Environment

```text
RTO

24 Hours

RPO

24 Hours
```

---

# Disaster Recovery Strategies Based on RTO/RPO

| Strategy | RTO | RPO | Cost |
|----------|-----|-----|------|
| Backup & Restore | High | High | Low |
| Pilot Light | Medium | Medium | Medium |
| Warm Standby | Low | Low | High |
| Multi-Site Active-Active | Very Low | Near Zero | Very High |

We will explore each strategy in detail in the next chapter.

---

# Cost vs Recovery

```text
Lower Cost

↓

Higher RTO

Higher RPO

────────────────────────

Higher Cost

↓

Lower RTO

Lower RPO
```

Better recovery generally requires more infrastructure.

---

# Enterprise Banking Example

Architecture

```text
Mumbai

↓

Primary Database

↓

Real-Time Replication

↓

Singapore

↓

Standby Database
```

Business Requirement

```text
RTO

5 Minutes

RPO

30 Seconds
```

This requires continuous replication and automated failover.

---

# E-Commerce Example

```text
Users

↓

Mumbai

↓

Orders

↓

Replication

↓

Singapore

↓

Standby
```

Business Requirement

```text
RTO

15 Minutes

RPO

5 Minutes
```

Acceptable because a small amount of order data can be recreated if necessary.

---

# Healthcare Example

Electronic Medical Records

Requirements

```text
RTO

10 Minutes

RPO

1 Minute
```

Patient records cannot tolerate significant data loss.

---

# Determining RTO & RPO

Before designing a DR solution, ask

- How much downtime is acceptable?
- How much data loss is acceptable?
- What is the financial impact?
- What are the compliance requirements?
- What are customer expectations?

These business answers drive the technical architecture.

---

# Enterprise Architecture

```text
Users

↓

Application

↓

Primary Region

↓

Continuous Replication

↓

Secondary Region

↓

Automatic Recovery
```

The replication method and failover design are selected based on the required RTO and RPO.

---

# Best Practices

- Define RTO and RPO before designing the solution.
- Align recovery objectives with business requirements.
- Use automation to reduce recovery time.
- Replicate critical data continuously.
- Test Disaster Recovery regularly.
- Review RTO and RPO after major application changes.
- Document recovery procedures.

---

# Common Mistakes

- Designing DR without business requirements.
- Confusing RTO with RPO.
- Assuming backups alone provide low RPO.
- Never measuring actual recovery time.
- Not testing whether recovery objectives can actually be achieved.
- Applying the same RTO/RPO to every application.

---

# Interview Questions

## Basic

- What is RTO?
- What is RPO?
- RTO vs RPO.

## Intermediate

- Why is RPO important for databases?
- How do business requirements influence RTO and RPO?
- Why do lower RTO and RPO generally increase costs?

## Advanced

- A banking application requires an RTO of 5 minutes and an RPO of 30 seconds. Design a Disaster Recovery architecture that satisfies these objectives.
- Compare the DR requirements of an e-commerce platform, a healthcare system, and a development environment, explaining how their RTO and RPO values influence the overall architecture.
- Your company currently restores applications in 2 hours after a disaster, but the business now requires recovery within 15 minutes. Explain the architectural changes you would recommend.

---

# Chapter 5 - Disaster Recovery Strategies (Backup & Restore, Pilot Light, Warm Standby & Multi-Site Active-Active)

Not every application requires the same Disaster Recovery (DR) architecture.

A development environment can tolerate hours of downtime, while an online banking system may require recovery within minutes.

AWS provides multiple Disaster Recovery strategies to balance

- Cost
- Recovery Time (RTO)
- Recovery Point (RPO)
- Business Requirements

Choosing the correct strategy is one of the most important architectural decisions.

---

# AWS Disaster Recovery Strategies

AWS recommends four primary Disaster Recovery strategies.

```text
Backup & Restore

↓

Pilot Light

↓

Warm Standby

↓

Multi-Site Active-Active
```

As we move down the list,

- Cost increases
- Availability improves
- Recovery time decreases

---

# Disaster Recovery Comparison

| Strategy | Cost | RTO | RPO |
|----------|------|------|------|
| Backup & Restore | Low | High | High |
| Pilot Light | Medium | Medium | Medium |
| Warm Standby | High | Low | Low |
| Active-Active | Very High | Near Zero | Near Zero |

---

# Strategy 1 - Backup & Restore

This is the simplest Disaster Recovery strategy.

Applications run only in the primary Region.

Backups are stored separately.

Architecture

```text
Primary Region

↓

Application

↓

Backup

↓

Amazon S3

↓

Disaster

↓

Restore
```

Infrastructure is created only after a disaster occurs.

---

# Backup & Restore Workflow

```text
Application

↓

Amazon RDS

↓

Backup

↓

Amazon S3

↓

Failure

↓

Restore

↓

Application Running
```

Recovery requires rebuilding infrastructure.

---

# Characteristics

- Lowest Cost
- Longest Recovery Time
- Infrastructure created after failure
- Suitable for non-critical workloads

---

# Example

Development Environment

```text
EC2

↓

Daily Backup

↓

Amazon S3

↓

Restore When Needed
```

Losing several hours is acceptable.

---

# Advantages

- Very inexpensive
- Simple architecture
- Minimal infrastructure
- Easy to maintain

---

# Disadvantages

- High RTO
- High RPO
- Manual recovery
- Long application downtime

---

# Suitable Workloads

- Development
- Testing
- Internal Applications
- Archive Systems

---

# Strategy 2 - Pilot Light

Pilot Light means

Only the critical core infrastructure is always running.

Everything else starts only during a disaster.

---

# Pilot Light Architecture

```text
Primary Region

↓

Production

↓

Continuous Database Replication

↓

Secondary Region

↓

Database Running

↓

Application Servers

Stopped
```

The database is already available.

Application servers start only during failover.

---

# Recovery Workflow

```text
Primary Failure

↓

Launch EC2

↓

Attach ALB

↓

Connect Database

↓

Application Available
```

Recovery is much faster than Backup & Restore.

---

# Characteristics

- Medium Cost
- Medium Recovery Time
- Database Always Running
- Infrastructure Created During Disaster

---

# Example

E-Commerce Platform

```text
Primary

↓

EC2

↓

Database

↓

Replication

↓

Secondary

↓

Database Only
```

Application servers launch only after disaster.

---

# Advantages

- Lower Cost
- Faster Recovery
- Continuous Data Replication
- Smaller Secondary Environment

---

# Disadvantages

- Recovery still requires automation
- Infrastructure provisioning during disaster
- Higher operational complexity

---

# Suitable Workloads

- Medium Business Applications
- Internal Business Systems
- Moderate Availability Requirements

---

# Strategy 3 - Warm Standby

Warm Standby means

A fully functional but smaller version of production runs continuously.

Architecture

```text
Primary Region

↓

Large Production

↓

Replication

↓

Secondary Region

↓

Small Production
```

Applications already run in both Regions.

Secondary resources scale up after failover.

---

# Warm Standby Workflow

```text
Primary Failure

↓

Route53 Failover

↓

Scale Auto Scaling Group

↓

Users Continue
```

Recovery is much faster.

---

# Characteristics

- Low RTO
- Low RPO
- Higher Cost
- Continuous Operations

---

# Example

Banking Portal

```text
Mumbai

↓

10 EC2

↓

Replication

↓

Singapore

↓

2 EC2
```

During failure

```text
2 EC2

↓

Auto Scaling

↓

10 EC2
```

---

# Advantages

- Fast Recovery
- Continuous Testing
- Lower Risk
- Easier Failover

---

# Disadvantages

- Higher Infrastructure Cost
- Continuous Resource Consumption

---

# Suitable Workloads

- Banking
- Insurance
- Healthcare
- Enterprise SaaS

---

# Strategy 4 - Multi-Site Active-Active

This is the most advanced Disaster Recovery strategy.

Applications run simultaneously in multiple Regions.

Architecture

```text
Users

↓

Route53

├── Mumbai

└── Singapore
```

Both Regions actively serve traffic.

---

# Active-Active Workflow

Normal Operation

```text
Users

↓

Mumbai

Singapore
```

Failure

```text
Mumbai

↓

Unavailable

↓

Singapore

↓

100% Traffic
```

No infrastructure needs to start.

---

# Characteristics

- Near Zero RTO
- Near Zero RPO
- Highest Cost
- Highest Availability

---

# Enterprise Banking Example

```text
Route53

↓

Mumbai

↓

Active

──────────────

Singapore

↓

Active
```

Database replication occurs continuously.

Users are automatically routed to the healthy Region.

---

# Advantages

- Extremely High Availability
- Fastest Recovery
- Minimal Data Loss
- Continuous Production Testing

---

# Disadvantages

- Very Expensive
- Complex Architecture
- Data Synchronization Challenges
- Operational Complexity

---

# Suitable Workloads

- Banking
- Stock Trading
- Payment Gateways
- Airline Reservation Systems
- Global SaaS Platforms

---

# Recovery Strategy Comparison

```text
Backup & Restore

↓

Hours

────────────────

Pilot Light

↓

Minutes

────────────────

Warm Standby

↓

Few Minutes

────────────────

Active-Active

↓

Seconds
```

Recovery time improves as investment increases.

---

# Cost Comparison

```text
Lowest

↓

Backup & Restore

↓

Pilot Light

↓

Warm Standby

↓

Active-Active

↓

Highest
```

Organizations choose based on business requirements.

---

# Choosing the Right Strategy

| Business Requirement | Recommended Strategy |
|----------------------|----------------------|
| Development | Backup & Restore |
| Internal Applications | Pilot Light |
| Customer-Facing Applications | Warm Standby |
| Mission-Critical Banking | Multi-Site Active-Active |

---

# Enterprise Architecture Example

```text
Users

↓

Amazon Route53

↓

Mumbai

↓

Application

↓

Amazon Aurora

↓

Cross-Region Replication

↓

Singapore

↓

Warm Standby

↓

Auto Scaling
```

Recovery occurs automatically after a regional failure.

---

# Best Practices

- Choose the DR strategy based on business RTO and RPO.
- Automate infrastructure provisioning using Infrastructure as Code.
- Replicate critical databases continuously.
- Test failover regularly.
- Monitor replication health.
- Keep recovery procedures documented.
- Review DR architecture annually.

---

# Common Mistakes

- Selecting Backup & Restore for mission-critical applications.
- Never testing disaster recovery.
- Ignoring replication lag.
- Keeping outdated backups.
- Depending on manual recovery.
- Assuming backups alone provide business continuity.

---

# Interview Questions

## Basic

- What are the four AWS Disaster Recovery strategies?
- What is Pilot Light?
- What is Warm Standby?

## Intermediate

- Backup & Restore vs Pilot Light.
- Warm Standby vs Multi-Site Active-Active.
- Which DR strategy provides the lowest RTO?

## Advanced

- Design a Disaster Recovery architecture for an online banking platform with an RTO of 5 minutes and an RPO of 30 seconds.
- Explain how you would migrate an organization from Backup & Restore to Warm Standby as business availability requirements increase.
- A global e-commerce company serves customers from multiple continents and requires continuous availability even during an AWS Region outage. Design the complete Multi-Site Active-Active architecture, explaining traffic routing, database replication, failover, and operational considerations.

---

# Chapter 6 - Multi-AZ Architecture (Deep Dive)

High Availability in AWS is primarily achieved through **Multi-AZ Architecture**.

Instead of running an application in a single Availability Zone, enterprise applications are deployed across multiple Availability Zones within the same AWS Region.

This protects applications against

- EC2 Failures
- Rack Failures
- Storage Failures
- Network Failures
- Complete Availability Zone Failures

Multi-AZ is one of the most fundamental concepts in AWS architecture and appears in almost every Solutions Architect interview.

---

# What is Multi-AZ?

Multi-AZ means deploying application components across two or more Availability Zones inside the same AWS Region.

Example

```text
Mumbai Region

├── AZ-A

└── AZ-B
```

Applications are distributed between both AZs.

---

# Single-AZ Architecture

Consider an application deployed only in one Availability Zone.

```text
Users

↓

Application Load Balancer

↓

EC2

↓

AZ-A
```

If AZ-A experiences an outage,

```text
Application

↓

Unavailable
```

This architecture has a Single Point of Failure.

---

# Multi-AZ Architecture

Instead,

deploy instances across multiple AZs.

```text
Users

↓

Application Load Balancer

↓

───────────────

AZ-A

↓

EC2

───────────────

AZ-B

↓

EC2
```

If one Availability Zone fails,

the other continues serving requests.

---

# How Multi-AZ Works

Workflow

```text
Client Request

↓

Application Load Balancer

↓

Health Check

↓

Healthy Instance

↓

Response
```

Traffic is automatically distributed among healthy instances.

---

# Failure Scenario

Normal Operation

```text
ALB

↓

EC2-A

↓

AZ-A

────────────

EC2-B

↓

AZ-B
```

Failure

```text
AZ-A

↓

Unavailable
```

Recovery

```text
ALB

↓

EC2-B

↓

AZ-B
```

Users continue accessing the application.

---

# Components of Multi-AZ Architecture

Typical architecture includes

- Multiple Availability Zones
- Application Load Balancer
- Auto Scaling Group
- Private Subnets
- NAT Gateway
- Databases
- Monitoring

Each component contributes to High Availability.

---

# Multi-AZ with Auto Scaling

Architecture

```text
Application Load Balancer

↓

Auto Scaling Group

↓

AZ-A

↓

EC2

────────────

AZ-B

↓

EC2
```

If an instance fails,

Auto Scaling automatically launches a replacement.

---

# Auto Scaling During Failure

Suppose

```text
EC2-A

↓

Failure
```

Auto Scaling detects

↓

Launch New EC2

↓

Healthy

↓

Added to Load Balancer

Recovery happens automatically.

---

# Health Checks

The Application Load Balancer continuously checks instance health.

Workflow

```text
ALB

↓

Health Check

↓

Healthy

↓

Receive Traffic

────────────

Unhealthy

↓

Remove From Rotation
```

Only healthy instances receive requests.

---

# Private Subnets

Production EC2 instances should usually run in private subnets.

Architecture

```text
Internet

↓

ALB

↓

Private Subnet

↓

EC2
```

Direct Internet access is avoided.

---

# Public vs Private Components

Typical deployment

```text
Public

↓

Application Load Balancer

↓

Private

↓

Application Servers

↓

Private

↓

Database
```

Only the Load Balancer is publicly accessible.

---

# NAT Gateway in Multi-AZ

Private instances may still require outbound Internet access.

Example

```text
Private EC2

↓

NAT Gateway

↓

Internet
```

AWS recommends deploying one NAT Gateway per Availability Zone.

---

# Multi-AZ NAT Architecture

```text
AZ-A

EC2

↓

NAT Gateway-A

──────────────

AZ-B

EC2

↓

NAT Gateway-B
```

This avoids cross-AZ dependency.

---

# Multi-AZ Database

Amazon RDS supports Multi-AZ deployments.

Architecture

```text
Primary Database

↓

AZ-A

↓

Synchronous Replication

↓

Standby Database

↓

AZ-B
```

AWS automatically manages replication.

---

# Database Failure

Suppose

```text
Primary Database

↓

Failure
```

AWS automatically promotes

```text
Standby

↓

Primary
```

Applications reconnect with minimal downtime.

---

# Storage High Availability

Amazon EFS automatically stores data across multiple Availability Zones.

Architecture

```text
EC2

↓

Amazon EFS

↓

Multiple AZs
```

Applications continue accessing shared files during an AZ failure.

---

# Multi-AZ Networking

Typical architecture

```text
VPC

├── Public Subnet AZ-A

├── Public Subnet AZ-B

├── Private Subnet AZ-A

└── Private Subnet AZ-B
```

Every Availability Zone contains both public and private resources.

---

# Complete Multi-AZ Web Application

```text
Users

↓

Route53

↓

Application Load Balancer

↓

──────────────

AZ-A

↓

EC2

↓

RDS Primary

──────────────

AZ-B

↓

EC2

↓

RDS Standby
```

This is a common production architecture.

---

# Multi-AZ Kubernetes (Amazon EKS)

Production EKS clusters distribute worker nodes across Availability Zones.

Architecture

```text
Amazon EKS

├── Node Group

│

├── AZ-A

├── AZ-B

└── AZ-C
```

Pods are automatically scheduled across nodes.

---

# EKS Failure

Suppose

```text
AZ-A

↓

Unavailable
```

Kubernetes automatically schedules new Pods on healthy nodes in

```text
AZ-B

AZ-C
```

Applications continue running.

---

# Multi-AZ ECS

Amazon ECS services can also span Availability Zones.

```text
Application Load Balancer

↓

Fargate Task

AZ-A

────────────

Fargate Task

AZ-B
```

Traffic continues even if one task or AZ fails.

---

# Enterprise Banking Example

```text
Customers

↓

Route53

↓

ALB

↓

AZ-A

↓

Payment API

↓

AZ-B

↓

Payment API

↓

RDS Multi-AZ
```

Failure of an entire Availability Zone does not interrupt payment processing.

---

# Benefits

- High Availability
- Automatic Failover
- Better Reliability
- Load Distribution
- Reduced Downtime
- Improved Fault Isolation

---

# Best Practices

- Always deploy production workloads across multiple Availability Zones.
- Use Application Load Balancers.
- Use Auto Scaling Groups.
- Deploy NAT Gateways in each AZ.
- Enable Multi-AZ for RDS.
- Spread workloads evenly across Availability Zones.
- Monitor Availability Zone health.
- Regularly test failover.

---

# Common Mistakes

- Deploying all EC2 instances in one Availability Zone.
- Using only one NAT Gateway for a Multi-AZ architecture.
- Ignoring database redundancy.
- Running all Kubernetes worker nodes in one AZ.
- Not configuring health checks.
- Assuming Multi-AZ protects against Region failure.

---

# Interview Questions

## Basic

- What is Multi-AZ?
- Why is Multi-AZ important?
- What happens if an Availability Zone fails?

## Intermediate

- Explain how an Application Load Balancer supports Multi-AZ deployments.
- Why should NAT Gateways be deployed in each Availability Zone?
- Explain RDS Multi-AZ architecture.

## Advanced

- Design a highly available three-tier web application using Multi-AZ architecture.
- Explain how Amazon EKS achieves High Availability across Availability Zones.
- A production application deployed in two Availability Zones loses one entire AZ. Explain, step by step, how the Application Load Balancer, Auto Scaling Group, EC2 instances, and RDS Multi-AZ work together to keep the application available.

---

# Chapter 7 - Multi-Region Architecture (Deep Dive)

While **Multi-AZ** protects applications against **Availability Zone failures**, it does **not** protect against an entire AWS Region becoming unavailable.

For mission-critical applications such as

- Banking
- Healthcare
- E-Commerce
- Government
- Airline Reservation Systems
- Global SaaS Platforms

organizations deploy applications across multiple AWS Regions.

This architecture is called **Multi-Region Architecture**.

---

# What is Multi-Region?

A Multi-Region architecture deploys applications in two or more AWS Regions.

Example

```text
Primary Region

Mumbai

↓

Secondary Region

Singapore
```

Both Regions contain complete application infrastructure.

---

# Why Multi-Region?

Although AWS Regions are extremely reliable,

an entire Region can become unavailable because of

- Large Network Failures
- Natural Disasters
- Major Infrastructure Issues
- Human Errors
- Regional Service Disruptions

A Multi-Region architecture protects against these failures.

---

# Single-Region Architecture

Example

```text
Users

↓

Route53

↓

Mumbai

↓

Application
```

If Mumbai becomes unavailable,

```text
Application

↓

Unavailable
```

Entire business stops.

---

# Multi-Region Architecture

Instead

```text
Users

↓

Route53

↓

Mumbai

↓

Application

──────────────

Singapore

↓

Application
```

If Mumbai fails,

traffic moves to Singapore.

---

# Components of Multi-Region Architecture

Typical architecture includes

- Route53
- Application Load Balancer
- Auto Scaling
- EC2 / ECS / EKS
- Database Replication
- Amazon S3 Replication
- CloudFront
- Monitoring

Each Region operates independently.

---

# Route53

Amazon Route53 acts as the global traffic manager.

```text
Users

↓

Route53

↓

Mumbai

↓

Singapore
```

Route53 determines which Region should receive traffic.

---

# Route53 Health Checks

Route53 continuously checks application health.

```text
Route53

↓

Health Check

↓

Healthy

↓

Route Traffic

──────────────

Unhealthy

↓

Stop Routing
```

Traffic is automatically redirected.

---

# Failover Routing

Primary Region

```text
Mumbai

↓

Healthy
```

Traffic

↓

Mumbai

If

```text
Mumbai

↓

Unhealthy
```

Route53

↓

Singapore

Users experience minimal disruption.

---

# Active-Passive Multi-Region

One Region serves traffic.

The other waits for disaster.

```text
Users

↓

Route53

↓

Mumbai

Active

────────────

Singapore

Standby
```

Lower cost.

Higher recovery time.

---

# Active-Active Multi-Region

Both Regions serve traffic.

```text
Users

↓

Route53

↓

Mumbai

↓

Users

──────────────

Singapore

↓

Users
```

Benefits

- Better Performance
- Load Sharing
- Higher Availability

---

# Active-Passive vs Active-Active

| Active-Passive | Active-Active |
|----------------|---------------|
| One Region Active | Both Regions Active |
| Lower Cost | Higher Cost |
| Disaster Recovery | Continuous Availability |
| Simpler | More Complex |

---

# Global Traffic Flow

```text
Users

↓

DNS

↓

Route53

↓

Nearest Healthy Region

↓

Application
```

Users automatically reach an available Region.

---

# Database Replication

Applications require data replication between Regions.

Example

```text
Primary Database

Mumbai

↓

Replication

↓

Singapore
```

Without replication,

applications cannot recover properly.

---

# Amazon Aurora Global Database

Aurora Global Database replicates data across Regions.

Architecture

```text
Mumbai

↓

Primary Cluster

↓

Replication

↓

Singapore

↓

Read Replica
```

Benefits

- Low Replication Lag
- Fast Failover
- Global Reads

---

# Amazon RDS Cross-Region Read Replica

Example

```text
Primary RDS

↓

Mumbai

↓

Replication

↓

Singapore

↓

Read Replica
```

The replica can be promoted during disaster recovery.

---

# Amazon DynamoDB Global Tables

DynamoDB supports

multi-active databases.

```text
Mumbai

↓

Read + Write

──────────────

Singapore

↓

Read + Write
```

Both Regions remain active.

---

# Amazon S3 Cross-Region Replication

Objects uploaded in one Region automatically replicate.

```text
Mumbai Bucket

↓

Cross Region Replication

↓

Singapore Bucket
```

Useful for

- Backups
- Compliance
- Disaster Recovery

---

# Multi-Region Kubernetes

Amazon EKS clusters can exist in multiple Regions.

```text
Mumbai

↓

Amazon EKS

──────────────

Singapore

↓

Amazon EKS
```

Applications deploy independently in each Region.

---

# Multi-Region ECS

Amazon ECS services can also span Regions.

```text
Route53

↓

Mumbai ECS

──────────────

Singapore ECS
```

Traffic automatically switches during failures.

---

# Multi-Region Microservices

Example

```text
Users

↓

Route53

↓

Mumbai

↓

Payment API

↓

Order API

↓

Notification API

──────────────

Singapore

↓

Payment API

↓

Order API

↓

Notification API
```

Each Region contains the complete application stack.

---

# Cross-Region Networking

Regions communicate over

```text
AWS Global Backbone
```

Benefits

- Private Network
- High Reliability
- Secure Replication
- Lower Latency

---

# Regional Failover

Failure

```text
Mumbai

↓

Unavailable
```

Recovery

```text
Route53

↓

Singapore

↓

Application Available
```

No infrastructure creation is required.

---

# Enterprise Banking Example

```text
Customers

↓

Route53

↓

Mumbai

↓

Internet Banking

↓

Aurora

↓

Cross-Region Replication

↓

Singapore

↓

Standby Banking Platform
```

Banking continues after regional failures.

---

# Global SaaS Example

```text
Users

↓

North America

↓

Virginia

────────────

Europe

↓

Frankfurt

────────────

Asia

↓

Mumbai
```

Users connect to the nearest healthy Region.

---

# Multi-Region Monitoring

Monitor

- Region Health
- Database Replication Lag
- Route53 Health Checks
- Application Health
- API Latency
- Regional Traffic

CloudWatch provides visibility into each Region independently.

---

# Enterprise Architecture

```text
Users

↓

CloudFront

↓

Route53

↓

Mumbai

↓

ALB

↓

Auto Scaling

↓

Aurora

↓

Cross Region Replication

↓

Singapore

↓

ALB

↓

Auto Scaling

↓

Aurora Replica
```

This architecture provides

- High Availability
- Disaster Recovery
- Global Performance
- Business Continuity

---

# Benefits

- Regional Disaster Recovery
- Lower Global Latency
- Better Customer Experience
- Higher Availability
- Global Scaling
- Business Continuity

---

# Best Practices

- Deploy critical workloads across multiple Regions.
- Use Route53 health checks.
- Replicate databases continuously.
- Enable Cross-Region Replication for Amazon S3.
- Automate failover.
- Test Regional Disaster Recovery regularly.
- Monitor replication lag.
- Keep infrastructure identical across Regions.

---

# Common Mistakes

- Assuming Multi-AZ protects against Region failure.
- Not replicating databases.
- Forgetting DNS failover.
- Deploying different application versions in different Regions.
- Never testing regional failover.
- Ignoring replication latency.

---

# Interview Questions

## Basic

- What is Multi-Region architecture?
- Why use Multi-Region instead of Multi-AZ?
- What is Route53 Failover Routing?

## Intermediate

- Active-Passive vs Active-Active Multi-Region.
- Explain Cross-Region Replication.
- How does Aurora Global Database support Disaster Recovery?

## Advanced

- Design a Multi-Region architecture for a global banking application requiring continuous availability.
- Explain how Route53, Aurora Global Database, CloudFront, Auto Scaling, and Cross-Region Replication work together during an AWS Region outage.
- Your company operates in Asia, Europe, and North America with strict Disaster Recovery requirements. Design a complete Multi-Region AWS architecture that minimizes latency, ensures business continuity, and provides automatic failover during regional failures.

---

# Chapter 8 - AWS Disaster Recovery Services & Data Replication

Disaster Recovery is not achieved by a single AWS service.

Instead, AWS provides multiple services that work together to ensure

- Data Protection
- Continuous Replication
- Automated Recovery
- Business Continuity
- Low Recovery Time
- Low Data Loss

A well-designed Disaster Recovery architecture combines compute, storage, networking, databases, DNS, and monitoring.

---

# Disaster Recovery Architecture

```text
Users

↓

Amazon Route53

↓

Primary Region

↓

Applications

↓

Databases

↓

Replication

↓

Secondary Region

↓

Standby Applications

↓

Standby Databases
```

Every layer participates in Disaster Recovery.

---

# AWS Services Used in Disaster Recovery

The most commonly used AWS services are

- Amazon Route53
- Amazon S3
- Amazon EBS
- Amazon RDS
- Amazon Aurora
- DynamoDB
- AWS Backup
- AWS Elastic Disaster Recovery (AWS DRS)
- Amazon EFS
- AWS DataSync
- CloudWatch
- Amazon SNS

Each service solves a different part of Disaster Recovery.

---

# Amazon Route53

Route53 is responsible for

- DNS
- Health Checks
- Traffic Routing
- Regional Failover

Architecture

```text
Users

↓

Route53

↓

Healthy Region
```

If the primary Region fails,

Route53 redirects traffic automatically.

---

# Route53 Health Checks

```text
Application

↓

Health Check

↓

Healthy

↓

Route Traffic

────────────

Unhealthy

↓

Secondary Region
```

Health checks automate failover decisions.

---

# Amazon S3

S3 is commonly used for

- Backups
- Snapshots
- Log Storage
- Disaster Recovery Files

Architecture

```text
Application

↓

Amazon S3

↓

Cross Region Replication

↓

Backup Region
```

---

# S3 Cross-Region Replication (CRR)

Objects uploaded in one Region automatically replicate.

```text
Mumbai Bucket

↓

Replication

↓

Singapore Bucket
```

Benefits

- Disaster Recovery
- Compliance
- Backup
- Global Availability

---

# Amazon EBS Snapshots

EC2 storage can be backed up using EBS Snapshots.

Architecture

```text
EC2

↓

EBS Volume

↓

Snapshot

↓

Amazon S3
```

Snapshots can later restore new EBS volumes.

---

# Snapshot Recovery

```text
Snapshot

↓

New EBS Volume

↓

Attach EC2

↓

Application Restored
```

Recovery is much faster than rebuilding manually.

---

# Amazon RDS Multi-AZ

RDS Multi-AZ protects against

Availability Zone failures.

Architecture

```text
Primary Database

↓

Synchronous Replication

↓

Standby Database
```

AWS automatically performs failover.

---

# Amazon RDS Cross-Region Read Replica

For Regional Disaster Recovery,

RDS supports cross-region replication.

```text
Mumbai

↓

Primary RDS

↓

Replication

↓

Singapore

↓

Read Replica
```

During disaster,

the replica can be promoted.

---

# Amazon Aurora Global Database

Aurora Global Database provides

low-latency cross-region replication.

Architecture

```text
Mumbai

↓

Primary Cluster

↓

Global Replication

↓

Singapore

↓

Secondary Cluster
```

Benefits

- Very Low Replication Lag
- Fast Recovery
- Global Reads

---

# Amazon DynamoDB Global Tables

DynamoDB supports active-active replication.

```text
Mumbai

↓

Read + Write

──────────────

Singapore

↓

Read + Write
```

Applications can write in multiple Regions simultaneously.

---

# Amazon EFS

Amazon EFS provides shared storage.

Combined with AWS Backup,

it supports Disaster Recovery for shared file systems.

Architecture

```text
Applications

↓

Amazon EFS

↓

AWS Backup
```

---

# AWS Backup

AWS Backup centralizes backup management.

Supports

- EBS
- EFS
- RDS
- DynamoDB
- FSx
- Storage Gateway

Architecture

```text
AWS Resources

↓

AWS Backup

↓

Backup Vault
```

One service manages backup policies across AWS.

---

# Backup Vault

Backups are stored securely.

```text
AWS Backup

↓

Backup Vault

↓

Recovery
```

Retention policies control how long backups are stored.

---

# AWS Elastic Disaster Recovery (AWS DRS)

AWS Elastic Disaster Recovery continuously replicates servers into AWS.

Architecture

```text
On-Premises

↓

Continuous Replication

↓

AWS

↓

Recovery Instance
```

Suitable for

- Physical Servers
- VMware
- Hyper-V
- EC2

---

# AWS DRS Workflow

```text
Production Server

↓

Continuous Replication

↓

Low-Cost Staging Area

↓

Disaster

↓

Launch Recovery Server
```

Recovery is automated.

---

# AWS DataSync

AWS DataSync transfers large datasets.

Architecture

```text
On-Premises

↓

AWS DataSync

↓

Amazon S3

↓

Amazon EFS

↓

Amazon FSx
```

Useful for

- Disaster Recovery
- Migration
- Backup

---

# Amazon FSx

FSx provides managed file systems.

Examples

- Windows File Server
- Lustre
- NetApp ONTAP
- OpenZFS

FSx backups support Disaster Recovery.

---

# CloudWatch

CloudWatch continuously monitors

- Application Health
- CPU
- Memory
- Database Status
- Replication
- Networking

Problems are detected before complete failures occur.

---

# Amazon SNS

CloudWatch alarms trigger SNS.

```text
CloudWatch

↓

Alarm

↓

SNS

↓

Email

↓

Operations Team
```

Engineers are immediately notified.

---

# Disaster Recovery Workflow

```text
Application Failure

↓

CloudWatch Alarm

↓

SNS Notification

↓

Route53

↓

Secondary Region

↓

Recovery
```

Automation minimizes downtime.

---

# Complete Enterprise DR Architecture

```text
Users

↓

CloudFront

↓

Route53

↓

Mumbai Region

↓

ALB

↓

Auto Scaling

↓

Amazon Aurora

↓

Cross Region Replication

↓

Singapore Region

↓

Standby ALB

↓

Auto Scaling

↓

Aurora Secondary

↓

CloudWatch

↓

SNS
```

This architecture provides

- High Availability
- Regional Disaster Recovery
- Automated Failover
- Continuous Monitoring

---

# Enterprise Banking Example

```text
Customers

↓

Route53

↓

Mumbai

↓

Internet Banking

↓

Aurora

↓

Global Database

↓

Singapore

↓

Standby Banking Platform
```

If Mumbai fails,

Route53 redirects users,

Aurora promotes the secondary database,

and business continues.

---

# Best Practices

- Use Route53 health checks for automated failover.
- Enable Cross-Region Replication for critical data.
- Store backups in multiple Regions.
- Use AWS Backup for centralized backup management.
- Continuously monitor replication health.
- Test recovery procedures regularly.
- Encrypt backups using AWS KMS.
- Automate recovery with Infrastructure as Code.

---

# Common Mistakes

- Keeping backups in the same Region.
- Never testing backup restoration.
- Ignoring replication lag.
- Assuming snapshots alone provide Disaster Recovery.
- No monitoring for backup failures.
- Manual recovery procedures for critical systems.

---

# Interview Questions

## Basic

- What AWS services are commonly used for Disaster Recovery?
- What is AWS Elastic Disaster Recovery (AWS DRS)?
- What is S3 Cross-Region Replication?

## Intermediate

- RDS Multi-AZ vs Cross-Region Read Replica.
- Aurora Global Database vs DynamoDB Global Tables.
- AWS Backup vs EBS Snapshots.

## Advanced

- Design a complete Disaster Recovery solution for a global financial application using Route53, Aurora Global Database, AWS Backup, S3 Cross-Region Replication, CloudWatch, and AWS Elastic Disaster Recovery.
- Explain how AWS DRS performs continuous replication and rapid recovery for on-premises workloads.
- A multinational enterprise requires automated Disaster Recovery with an RTO of 10 minutes and an RPO of less than 1 minute. Explain which AWS services you would combine and why.

---

