# Amazon DynamoDB

---

# Introduction

Amazon DynamoDB is a fully managed, serverless, highly available NoSQL database service that provides single-digit millisecond performance at any scale. It is designed to handle millions of requests per second while automatically scaling storage and throughput without requiring infrastructure management.

Unlike traditional relational databases that use tables with joins and fixed schemas, DynamoDB stores data as key-value pairs and documents, making it ideal for modern cloud-native applications requiring high scalability and low latency.

Amazon DynamoDB integrates with:

- AWS Lambda
- Amazon API Gateway
- Amazon EC2
- Amazon ECS
- Amazon EKS
- Amazon S3
- Amazon Kinesis
- AWS AppSync
- Amazon CloudWatch
- AWS IAM
- AWS Backup
- Amazon EventBridge

DynamoDB is widely used for web applications, gaming, IoT, mobile backends, real-time analytics, and serverless architectures.

---

# What is Amazon DynamoDB?

Amazon DynamoDB is a NoSQL database.

Instead of using relational tables with joins,

DynamoDB stores data using

- Key-Value
- Document

models.

Workflow

```text
Application

↓

DynamoDB

↓

Millisecond Response
```

---

# Why DynamoDB?

Without DynamoDB

```text
Application

↓

Relational Database

↓

Scaling Challenges

↓

Performance Bottlenecks
```

Problems

- Manual Scaling
- Server Management
- High Latency
- Capacity Planning

With DynamoDB

```text
Application

↓

DynamoDB

↓

Automatic Scaling

↓

Low Latency
```

---

# Real World Problem Statement

A social media platform stores

- User Profiles
- Posts
- Comments
- Likes
- Followers
- Notifications

Requirements

- Millions of Users
- Millisecond Response
- Automatic Scaling
- High Availability

Amazon DynamoDB satisfies these requirements.

---

# Enterprise Architecture

```text
Users

↓

API Gateway

↓

AWS Lambda

↓

Amazon DynamoDB

↓

DynamoDB Streams

↓

Lambda

↓

Notifications
```

---

# Core Components

Amazon DynamoDB consists of

- Table
- Item
- Attribute
- Primary Key
- Partition Key
- Sort Key
- Secondary Index
- Streams
- Global Tables
- DAX

---

# Table

A Table stores data.

Example

```text
Users Table

Orders Table

Products Table
```

Unlike relational databases,

tables are schema flexible.

---

# Item

An Item is equivalent to a row.

Example

```json
{
 "UserId":"101",
 "Name":"John",
 "Country":"India"
}
```

Each Item is uniquely identified.

---

# Attribute

Attributes are equivalent to columns.

Example

```text
UserId

Name

Email

Age

Country
```

Different items may contain different attributes.

---

# Primary Key

Every DynamoDB table requires a Primary Key.

Types

- Partition Key
- Composite Key

---

# Partition Key

Uses one attribute.

Example

```text
UserId

↓

101
```

The partition key determines where data is stored.

---

# Composite Key

Consists of

- Partition Key
- Sort Key

Example

```text
CustomerId

+

OrderId
```

Supports one-to-many relationships.

---

# Partition

Data is distributed across partitions.

Architecture

```text
Table

↓

Partition A

Partition B

Partition C
```

AWS automatically manages partitions.

---

# Sort Key

Sort Keys organize items within a partition.

Example

```text
Customer101

↓

Order1

↓

Order2

↓

Order3
```

Useful for range queries.

---

# Data Types

Supported types

- String
- Number
- Boolean
- Binary
- List
- Map
- Set
- Null

---

# Schema Flexibility

Items can contain different attributes.

Example

```text
User A

Name

Email

----------------

User B

Name

Phone

Address
```

No schema migrations required.

---

# Read Capacity Units (RCU)

RCUs determine read throughput.

Higher RCUs

↓

More Reads Per Second

---

# Write Capacity Units (WCU)

WCUs determine write throughput.

Higher WCUs

↓

More Writes Per Second

---

# Capacity Modes

Two options

- Provisioned
- On-Demand

---

# Provisioned Capacity

Specify

- Read Capacity
- Write Capacity

Suitable for predictable workloads.

---

# On-Demand Capacity

Automatically scales based on demand.

Ideal for

- Unknown Traffic
- Variable Workloads
- Startups

---

# Auto Scaling

Provisioned capacity supports Auto Scaling.

Workflow

```text
Traffic Increase

↓

Auto Scaling

↓

Additional Capacity
```

---

# Consistency Models

DynamoDB supports

- Eventually Consistent
- Strongly Consistent

---

# Eventually Consistent Read

Provides lower latency.

Data becomes consistent shortly after updates.

Default option.

---

# Strongly Consistent Read

Returns the latest committed data.

Useful for financial and transactional applications.

---

# Secondary Indexes

Indexes improve query flexibility.

Types

- Global Secondary Index (GSI)
- Local Secondary Index (LSI)

---

# Global Secondary Index (GSI)

Uses different partition and sort keys.

Allows additional query patterns.

Example

```text
Email

↓

Lookup User
```

---

# Local Secondary Index (LSI)

Uses the same partition key,

but a different sort key.

Useful for alternate sorting.

---

# Query Operation

Query retrieves items using keys.

Example

```text
Customer101

↓

All Orders
```

Efficient operation.

---

# Scan Operation

Scan examines the entire table.

Workflow

```text
Entire Table

↓

Scan

↓

Results
```

Less efficient than Query.

---

# Point Lookup

Retrieve a single item using its Primary Key.

Fastest DynamoDB operation.

---

# Batch Operations

Supported

- Batch Read
- Batch Write

Improves performance.

---

# Conditional Writes

Write only when conditions are satisfied.

Example

```text
Balance > 100

↓

Update
```

Useful for concurrency control.

---

# Transactions

Supports ACID transactions.

Operations

- Put
- Update
- Delete
- Condition Check

Multiple operations execute atomically.

---

# Time To Live (TTL)

Automatically removes expired items.

Example

```text
OTP

↓

Expires

↓

Deleted Automatically
```

Useful for temporary data.

---

# Monitoring

CloudWatch Metrics

- Read Capacity
- Write Capacity
- Throttled Requests
- Latency
- Consumed Capacity

---

# Summary

Amazon DynamoDB is a fully managed NoSQL database that provides single-digit millisecond latency, automatic scaling, and high availability. Core concepts such as Tables, Items, Attributes, Primary Keys, Partition Keys, Sort Keys, Secondary Indexes, Capacity Modes, Transactions, and TTL make DynamoDB an ideal database for cloud-native, serverless, and high-scale applications.

---

