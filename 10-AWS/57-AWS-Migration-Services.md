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

