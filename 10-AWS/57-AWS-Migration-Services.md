# AWS Migration Services

---

# Introduction

AWS Migration Services are a collection of tools and services that help organizations migrate applications, databases, servers, storage, and workloads from on-premises environments or other cloud providers to AWS with minimal downtime.

Cloud migration involves moving infrastructure, applications, and data while maintaining business continuity. AWS provides specialized migration services that simplify assessment, planning, migration, replication, and modernization.

AWS Migration Services include

- AWS Application Migration Service (MGN)
- AWS Database Migration Service (DMS)
- AWS Migration Hub
- AWS Migration Hub Strategy Recommendations
- AWS Migration Evaluator
- AWS DataSync
- AWS Transfer Family
- AWS Snow Family
- AWS Application Discovery Service

These services support rehosting, replatforming, refactoring, and modernization strategies.

---

# What are AWS Migration Services?

AWS Migration Services help organizations migrate workloads to AWS.

They support

- Server Migration
- Database Migration
- File Migration
- Storage Migration
- Migration Tracking
- Migration Assessment

Workflow

```text
On-Premises

↓

Assessment

↓

Migration

↓

AWS

↓

Optimization
```

---

# Why AWS Migration Services?

Without Migration Services

```text
Manual Migration

↓

Long Downtime

↓

Migration Errors

↓

Business Risk
```

Problems

- Manual Processes
- Downtime
- Data Loss Risk
- Slow Migration
- Limited Visibility

With AWS Migration Services

```text
Assessment

↓

Automated Migration

↓

Validation

↓

Production on AWS
```

---

# Real World Problem Statement

A company plans to migrate

- 500 Virtual Machines
- 120 Databases
- 80 TB File Storage
- Legacy Applications

Requirements

- Minimal Downtime
- Central Tracking
- Data Integrity
- Secure Migration
- Migration Reporting

AWS Migration Services simplify the migration journey.

---

# Enterprise Architecture

```text
On-Premises

Servers

Databases

Storage

        │

        ▼

AWS Migration Services

        │

 ┌────────┼────────────┐

 │        │            │

MGN      DMS      DataSync

 │        │            │

Migration Hub

        │

        ▼

AWS Cloud
```

---

# Core Migration Services

AWS provides

- Application Migration Service (MGN)
- Database Migration Service (DMS)
- Migration Hub
- DataSync
- Snow Family
- Transfer Family
- Migration Evaluator
- Application Discovery Service
- Strategy Recommendations

---

# AWS Application Migration Service (MGN)

AWS MGN performs lift-and-shift server migrations.

Supports

- Physical Servers
- Virtual Machines
- Cloud Servers

Workflow

```text
Source Server

↓

Continuous Replication

↓

AWS

↓

Launch EC2

↓

Cutover
```

Benefits

- Minimal Downtime
- Automated Replication
- Easy Cutover

---

# MGN Features

- Continuous Block-Level Replication
- Non-Disruptive Migration
- Automated Test Instances
- Cutover Automation
- Recovery Options

---

# Supported Sources

Examples

- VMware
- Hyper-V
- Physical Servers
- Other Cloud Providers

---

# AWS Database Migration Service (DMS)

AWS DMS migrates databases to AWS with minimal downtime.

Supports

- Homogeneous Migration
- Heterogeneous Migration

Examples

```text
Oracle

↓

Amazon RDS Oracle

------------

SQL Server

↓

Amazon Aurora PostgreSQL
```

---

# DMS Components

- Replication Instance
- Source Endpoint
- Target Endpoint
- Migration Task

Architecture

```text
Source Database

↓

Replication Instance

↓

Target Database
```

---

# Homogeneous Migration

Same database engine.

Example

```text
Oracle

↓

Oracle RDS
```

Usually faster and simpler.

---

# Heterogeneous Migration

Different database engines.

Example

```text
SQL Server

↓

Aurora PostgreSQL
```

Typically uses AWS Schema Conversion Tool (SCT).

---

# AWS Schema Conversion Tool (SCT)

AWS SCT converts

- Database Schema
- Stored Procedures
- Views
- Functions

Required for many heterogeneous migrations.

---

# AWS Migration Hub

Migration Hub provides centralized migration tracking.

Features

- Migration Progress
- Server Inventory
- Dashboard
- Status Tracking

Supports multiple AWS migration services.

---

# Migration Hub Dashboard

Displays

- Applications
- Servers
- Migration Status
- Progress Reports

Provides centralized visibility.

---

# Summary

AWS Migration Services provide a comprehensive suite of tools for migrating servers, databases, applications, and storage to AWS. Services such as AWS Application Migration Service (MGN), Database Migration Service (DMS), Schema Conversion Tool (SCT), and Migration Hub simplify migration planning, execution, and tracking while minimizing downtime and operational risk.

---

# AWS Migration Evaluator

AWS Migration Evaluator helps organizations estimate the cost, savings, and business value of migrating workloads to AWS.

Features

- Current Infrastructure Assessment
- Cost Analysis
- AWS Pricing Estimates
- Total Cost of Ownership (TCO)
- Business Case Reports

Workflow

```text
On-Premises Infrastructure

↓

Migration Evaluator

↓

Cost Analysis

↓

AWS Cost Estimate

↓

Migration Report
```

Supports executive planning and budgeting.

---

# AWS Application Discovery Service

AWS Application Discovery Service collects information about on-premises infrastructure.

Discovers

- Servers
- Applications
- Dependencies
- CPU Usage
- Memory Usage
- Network Connections

Benefits

- Dependency Mapping
- Migration Planning
- Resource Inventory

---

# Discovery Workflow

```text
On-Premises Servers

↓

Discovery Agent

↓

AWS Discovery Service

↓

Migration Planning
```

Provides accurate migration readiness information.

---

# Migration Hub Strategy Recommendations

Migration Hub Strategy Recommendations analyzes workloads and recommends migration approaches.

Supports strategies such as

- Rehost
- Replatform
- Refactor
- Repurchase
- Retire
- Retain

Helps organizations choose the best migration path.

---

# The 6 Rs of Migration

```text
Application

↓

Assessment

↓

Rehost

Replatform

Refactor

Repurchase

Retire

Retain
```

Explanation

- Rehost – Lift and Shift
- Replatform – Minor Optimization
- Refactor – Redesign Application
- Repurchase – Replace with SaaS
- Retire – Remove Unused Application
- Retain – Keep On-Premises

---

# AWS DataSync

AWS DataSync accelerates online data transfer between on-premises storage and AWS.

Supports

- NFS
- SMB
- Amazon S3
- Amazon EFS
- Amazon FSx

Workflow

```text
On-Premises Storage

↓

AWS DataSync Agent

↓

Amazon S3 / EFS / FSx
```

Benefits

- Fast Data Transfer
- Data Validation
- Encryption
- Scheduling

---

# AWS Transfer Family

AWS Transfer Family provides managed file transfer services.

Protocols

- SFTP
- FTPS
- FTP

Storage Targets

- Amazon S3
- Amazon EFS

Use Cases

- B2B File Exchange
- Secure Partner Uploads
- Legacy File Transfer

---

# AWS Snow Family

AWS Snow Family transfers large volumes of data using physical devices.

Services

- Snowcone
- Snowball Edge
- Snowmobile

Suitable for

- Petabyte-Scale Migration
- Limited Network Bandwidth
- Disaster Recovery

---

# Snow Family Selection

| Service | Use Case |
|----------|----------|
| Snowcone | Small Edge Deployments |
| Snowball Edge | Large Data Migration |
| Snowmobile | Exabyte-Scale Data Transfer |

---

# Choosing the Right Migration Service

| Requirement | AWS Service |
|-------------|-------------|
| Server Migration | AWS MGN |
| Database Migration | AWS DMS |
| Schema Conversion | AWS SCT |
| Migration Tracking | Migration Hub |
| Cost Assessment | Migration Evaluator |
| Dependency Discovery | Application Discovery Service |
| Online File Transfer | DataSync |
| Secure File Transfer | Transfer Family |
| Offline Large Data Transfer | Snow Family |

---

# AWS CLI

Migration Hub

```bash
aws migrationhub list-progress-update-streams
```

DataSync

```bash
aws datasync list-tasks
```

DMS

```bash
aws dms describe-replication-tasks
```

MGN

```bash
aws mgn describe-source-servers
```

---

# Terraform

Example (AWS DataSync)

```hcl
resource "aws_datasync_location_s3" "backup" {

  s3_bucket_arn = aws_s3_bucket.backup.arn

}
```

---

# CloudFormation

```yaml
Resources:

  DataSyncTask:

    Type: AWS::DataSync::Task
```

---

# Python (Boto3)

```python
import boto3

mgn = boto3.client("mgn")

response = mgn.describe_source_servers()

print(response)
```

---

# Enterprise Production Architecture

```text
              On-Premises Environment

 Servers • Databases • File Storage

                    │

        ┌───────────┼────────────┬────────────┐

        │           │            │            │

       MGN         DMS       DataSync    Snow Family

        │           │            │            │

        └───────────┼────────────┴────────────┘

                    │

            AWS Migration Hub

                    │

           Amazon EC2 • RDS • S3

                    │

           Monitoring & Validation
```

---

# Best Practices

- Perform discovery before migration
- Classify applications using the 6 Rs
- Test migrations before production cutover
- Validate application dependencies
- Use continuous replication for critical workloads
- Convert schemas using AWS SCT when required
- Monitor migration progress with Migration Hub
- Use DataSync for recurring file transfers
- Use Snow Family for very large datasets
- Validate migrated applications before cutover
- Document rollback procedures
- Monitor post-migration performance

---

# Common Mistakes

- Migrating without dependency analysis
- Choosing the wrong migration strategy
- Skipping migration testing
- Ignoring schema conversion requirements
- No rollback plan
- Migrating during peak business hours
- Forgetting application validation
- Underestimating bandwidth requirements
- Poor migration documentation
- Ignoring post-migration optimization

---

# Troubleshooting

## Server Replication Failed

Check

- Replication Agent
- Network Connectivity
- IAM Permissions
- Firewall Rules

---

## Database Migration Slow

Verify

- Replication Instance Size
- Network Latency
- Database Performance
- Migration Task Configuration

---

## DataSync Task Failed

Check

- Agent Status
- Source Permissions
- Destination Permissions
- Network Connectivity

---

## Snow Device Delayed

Verify

- Shipping Status
- Job Status
- Import Task
- Device Unlock Code

---

## Migration Hub Missing Resources

Check

- Connected Migration Tools
- Discovery Status
- IAM Permissions

---

# Interview Questions

## Basic

1. What are AWS Migration Services?
2. What is AWS Application Migration Service (MGN)?
3. What is AWS Database Migration Service (DMS)?
4. What is AWS Migration Hub?
5. What is AWS Schema Conversion Tool (SCT)?
6. What is AWS DataSync?
7. What is AWS Transfer Family?
8. What is AWS Snow Family?
9. What is AWS Migration Evaluator?
10. What is AWS Application Discovery Service?

---

## Intermediate

11. Explain the 6 Rs of migration.
12. Explain homogeneous vs heterogeneous database migration.
13. Explain Migration Hub Strategy Recommendations.
14. Explain DataSync architecture.
15. Explain Transfer Family.
16. Explain Snow Family use cases.
17. Explain migration dependency analysis.
18. Explain migration assessment.
19. Explain migration cutover planning.
20. Explain post-migration validation.

---

## Advanced

21. Design an enterprise migration strategy.
22. How would you migrate 1,000 servers to AWS?
23. Explain MGN vs DMS.
24. Design a database migration with minimal downtime.
25. Explain DataSync vs Transfer Family.
26. Design petabyte-scale migration architecture.
27. Explain migration governance.
28. Design rollback strategies for production migrations.
29. Explain operational best practices for AWS migrations.
30. Best practices for enterprise cloud migrations.

---

# Production Scenarios

### Scenario 1

A company wants to migrate 600 VMware virtual machines to AWS with minimal downtime.

Which AWS migration service would you choose and why?

---

### Scenario 2

An Oracle database must be migrated to Amazon Aurora PostgreSQL.

Which AWS services are required, and what role does each service play?

---

### Scenario 3

An organization needs to transfer 300 TB of files from its data center to Amazon S3 over a slow WAN connection.

Would you recommend AWS DataSync or AWS Snow Family? Explain your decision.

---

### Scenario 4

Leadership requests an estimate of migration costs before approving the project.

Which AWS service would you use to generate a business case?

---

### Scenario 5

A company wants to understand application dependencies before migration.

Which AWS service would you use, and what information would it collect?

---

### Scenario 6

A migration program includes hundreds of servers, databases, and file systems.

How would AWS Migration Hub help track progress across multiple migration services?

---

# Cheat Sheet

| Service | Purpose |
|---------|---------|
| AWS MGN | Server Migration |
| AWS DMS | Database Migration |
| AWS SCT | Schema Conversion |
| Migration Hub | Migration Tracking |
| Migration Evaluator | Cost & TCO Analysis |
| Application Discovery Service | Dependency Discovery |
| AWS DataSync | Online Data Transfer |
| AWS Transfer Family | Managed FTP/SFTP/FTPS |
| AWS Snow Family | Offline Large Data Transfer |
| Strategy Recommendations | Migration Planning |

---

# Summary

AWS Migration Services provide a comprehensive migration portfolio for moving servers, databases, storage, and applications to AWS. Services such as AWS Application Migration Service (MGN), Database Migration Service (DMS), Schema Conversion Tool (SCT), Migration Hub, Migration Evaluator, Application Discovery Service, DataSync, Transfer Family, and Snow Family enable organizations to assess, plan, execute, and optimize cloud migrations while minimizing downtime, reducing risk, and supporting enterprise-scale transformation initiatives.