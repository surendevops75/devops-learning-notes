# Amazon Relational Database Service (Amazon RDS)

---

# Introduction

Amazon Relational Database Service (Amazon RDS) is a fully managed relational database service provided by AWS. It simplifies the process of setting up, operating, scaling, securing, and maintaining relational databases in the cloud.

Instead of spending time installing database software, configuring replication, performing backups, applying patches, monitoring servers, and planning disaster recovery, AWS automates these operational tasks so engineers can focus on building applications.

Amazon RDS is designed for production workloads that require:

- High Availability
- Automatic Backups
- Disaster Recovery
- Security
- Performance
- Scalability
- Monitoring

Today, thousands of enterprise applications use Amazon RDS as their primary relational database platform.

---

# What is Amazon RDS?

Amazon RDS is a managed database service that allows you to create relational databases without managing the underlying infrastructure.

AWS manages:

- Operating System
- Database Installation
- Software Patching
- Hardware Maintenance
- Automatic Backups
- Monitoring
- Failover
- Storage Scaling

You only manage:

- Database Schema
- Tables
- Queries
- Users
- Permissions
- Application Data

---

# Why Amazon RDS?

Imagine hosting MySQL on an EC2 instance.

You are responsible for:

- Installing MySQL
- Configuring replication
- Taking backups
- Monitoring CPU
- Monitoring storage
- Updating OS
- Applying database patches
- Recovering after failures
- Scaling storage
- Configuring High Availability

This requires significant operational effort.

With Amazon RDS:

```text
Application

↓

Amazon RDS

↓

AWS Manages Everything Else
```

Developers focus on applications instead of database administration.

---

# Real-World Problem

A banking application serves millions of customers.

Requirements:

- Zero downtime
- Automated backups
- High availability
- Disaster recovery
- Secure encryption
- Read scaling
- Automatic failover

Managing this manually is complex.

Amazon RDS provides these capabilities as managed features.

---

# Enterprise Architecture

```text
                    Internet

                        │

                  Application Load Balancer

                        │

                  EC2 / Amazon EKS

                        │

                Amazon RDS Proxy (Optional)

                        │

              ┌───────────────────────────┐

              │      Writer Instance      │

              │        Multi-AZ           │

              └─────────────┬─────────────┘

                            │

                 Synchronous Replication

                            │

              ┌───────────────────────────┐

              │     Standby Instance      │

              └───────────────────────────┘

                            │

                Automated Backups

                            │

                           S3
```

---

# Internal Working

When an application sends a database request:

```text
Application

↓

DNS Endpoint

↓

Amazon RDS

↓

Database Engine

↓

Storage Layer

↓

Response
```

If Multi-AZ is enabled:

```text
Write Request

↓

Primary Database

↓

Synchronous Replication

↓

Standby Database

↓

Acknowledgement

↓

Application
```

This ensures data durability before confirming the transaction.

---

# Core Components

Amazon RDS consists of:

- DB Instance
- Database Engine
- Storage
- DB Subnet Group
- Parameter Group
- Option Group
- Security Group
- Multi-AZ
- Read Replica
- Backups
- Snapshots
- Monitoring
- Encryption

---

# Supported Database Engines

Amazon RDS supports multiple database engines.

| Engine | Typical Use Case |
|----------|----------------|
| MySQL | Web Applications |
| PostgreSQL | Enterprise Applications |
| MariaDB | Open Source Workloads |
| Oracle | Enterprise ERP |
| Microsoft SQL Server | Windows Applications |
| Amazon Aurora | High Performance Cloud Databases |

Each engine supports different features, licensing models, and performance characteristics.

---

# Choosing the Right Database Engine

| Requirement | Recommended Engine |
|-------------|-------------------|
| Open Source | PostgreSQL |
| WordPress | MySQL |
| Enterprise ERP | Oracle |
| Microsoft Ecosystem | SQL Server |
| High Performance | Aurora |

---

# DB Instance

A DB Instance is the compute resource that runs your database.

It includes:

- CPU
- Memory
- Storage
- Database Engine
- Network
- Security

Example

```text
Application

↓

DB Endpoint

↓

RDS Instance
```

Unlike EC2, you do not log in to the operating system.

---

# DB Instance Classes

Examples

| Class | Purpose |
|--------|---------|
| db.t3.micro | Development |
| db.t3.small | Testing |
| db.m6i.large | Production |
| db.r6g.large | Memory Intensive |
| db.x2iedn | Enterprise Databases |

General Recommendation

- **t-series** → Development
- **m-series** → Balanced workloads
- **r-series** → Memory-intensive databases

---

# Storage Types

Amazon RDS supports three storage options.

| Storage | Best For |
|----------|----------|
| General Purpose SSD (gp3) | Most Workloads |
| Provisioned IOPS SSD | High Performance OLTP |
| Magnetic | Legacy Workloads |

---

# General Purpose SSD (gp3)

Most commonly used storage type.

Benefits:

- Cost effective
- Consistent performance
- Independent scaling of storage and IOPS
- Suitable for production

Recommended for:

- Web Applications
- APIs
- Business Applications

---

# Provisioned IOPS SSD

Designed for I/O intensive workloads.

Examples:

- Banking
- ERP
- Financial Systems
- Large Databases

Advantages

- Very low latency
- High throughput
- Predictable performance

---

# Magnetic Storage

Legacy storage option.

Disadvantages

- Low performance
- Higher latency
- Not recommended for new deployments

---

# Storage Autoscaling

RDS can automatically increase storage when utilization reaches configured thresholds.

Workflow

```text
Storage Usage

↓

Threshold Reached

↓

Automatic Storage Expansion

↓

Database Continues Running
```

Benefits

- Prevents storage exhaustion
- No downtime
- Reduced administrative effort

---

# DB Subnet Group

A DB Subnet Group defines the subnets where RDS instances can be deployed.

Typical Design

```text
VPC

├── Private Subnet A

├── Private Subnet B

└── Private Subnet C

↓

DB Subnet Group

↓

Amazon RDS
```

Best Practice

Always deploy RDS in private subnets.

---

# Security Groups

Security Groups control network access to the database.

Example

Inbound Rule

| Source | Port |
|----------|------|
| Application SG | 3306 |

Do NOT allow:

```
0.0.0.0/0
```

for production databases.

---

# Parameter Groups

Parameter Groups contain database engine configuration parameters.

Examples

- max_connections
- innodb_buffer_pool_size
- log_bin
- work_mem
- shared_buffers

Benefits

- Centralized configuration
- Consistent settings
- Easy tuning

Some parameters require a database reboot.

---

# Option Groups

Option Groups enable additional database features.

Examples

Oracle

- Oracle Enterprise Manager
- Transparent Data Encryption

SQL Server

- Native Backup
- SQL Server Audit

Not all engines require Option Groups.

---

# Multi-AZ Deployment

Multi-AZ provides High Availability.

Architecture

```text
Primary Database

↓

Synchronous Replication

↓

Standby Database
```

Characteristics

- Automatic failover
- No manual intervention
- Same endpoint
- High availability
- Disaster recovery within a Region

The standby database is **not used for read traffic**.

---

# Multi-AZ Failover

Failure Scenario

```text
Primary Fails

↓

AWS Detects Failure

↓

Standby Promoted

↓

DNS Updated

↓

Application Reconnects
```

Typical failover completes within a few minutes, depending on the engine and workload.

---

# Read Replicas

Read Replicas improve read performance.

Architecture

```text
Application

          │

     Write Requests

          │

      Primary RDS

          │

 Asynchronous Replication

          │

Read Replica 1

Read Replica 2

Read Replica 3
```

Applications send:

- Writes → Primary
- Reads → Replicas

---

# Multi-AZ vs Read Replica

| Multi-AZ | Read Replica |
|-----------|--------------|
| High Availability | Read Scaling |
| Synchronous Replication | Asynchronous Replication |
| Automatic Failover | Manual Promotion |
| Same Endpoint | Different Endpoint |
| Standby Not Readable | Replica Readable |

This is one of the most common AWS interview questions.

---

# Encryption

Amazon RDS supports encryption at rest using AWS KMS.

Encrypted components:

- Database Storage
- Automated Backups
- Manual Snapshots
- Read Replicas
- Transaction Logs

Benefits

- Compliance
- Secure storage
- Data protection

Production Recommendation:

Always enable encryption when creating a production database.

---

# IAM Database Authentication

Instead of database passwords, supported engines can authenticate using IAM.

Workflow

```text
IAM User

↓

Temporary Authentication Token

↓

Amazon RDS

↓

Database Access
```

Benefits

- No hardcoded passwords
- Temporary credentials
- Better security
- Centralized access management

