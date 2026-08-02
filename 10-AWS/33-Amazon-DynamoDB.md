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

# DynamoDB Streams

DynamoDB Streams capture every change made to items in a table.

Supported Events

- INSERT
- MODIFY
- REMOVE

Architecture

```text
Application

↓

DynamoDB

↓

DynamoDB Streams

↓

Lambda

↓

Business Logic
```

Streams enable event-driven architectures.

---

# Stream Records

Each stream record contains

- Keys
- Old Image
- New Image
- Event Type
- Timestamp

Useful for auditing and replication.

---

# Stream View Types

Available options

- Keys Only
- New Image
- Old Image
- New and Old Images

Choose based on application requirements.

---

# Lambda Integration

Lambda automatically processes stream records.

Workflow

```text
Table Update

↓

DynamoDB Stream

↓

Lambda

↓

Notification
```

Common use cases

- Audit Logging
- Notifications
- Data Synchronization
- Analytics

---

# Global Tables

Global Tables replicate DynamoDB tables across multiple AWS Regions.

Architecture

```text
Mumbai

↔

Singapore

↔

Virginia

↔

London
```

Benefits

- Global Availability
- Disaster Recovery
- Low Latency
- Multi-Region Writes

---

# Multi-Region Architecture

```text
Users

↓

Nearest Region

↓

Global Table

↓

Automatic Replication

↓

Other Regions
```

Applications read and write locally.

---

# Conflict Resolution

Global Tables use

Last Writer Wins

to resolve update conflicts.

Applications should avoid simultaneous updates to the same item from multiple Regions.

---

# DynamoDB Accelerator (DAX)

DAX is an in-memory cache for DynamoDB.

Benefits

- Microsecond Latency
- Reduced Read Load
- Automatic Cache Management

Architecture

```text
Application

↓

DAX

↓

DynamoDB
```

---

# Cache Workflow

```text
Read Request

↓

DAX

↓

Cache Hit

↓

Response

------------

Cache Miss

↓

DynamoDB

↓

Cache Updated
```

---

# Backup and Restore

DynamoDB supports

- On-Demand Backup
- Point-in-Time Recovery (PITR)

No downtime required.

---

# On-Demand Backup

Create a backup at any time.

Workflow

```text
Table

↓

Backup

↓

Restore Later
```

Useful before major deployments.

---

# Point-in-Time Recovery (PITR)

PITR continuously backs up the table.

Recovery Window

```
35 Days
```

Allows restoration to any second within the retention period.

---

# Export to Amazon S3

DynamoDB tables can be exported directly to Amazon S3.

Benefits

- Analytics
- Archiving
- Compliance
- Data Lake Integration

---

# Import from Amazon S3

Bulk data can be imported into DynamoDB from Amazon S3.

Useful for

- Data Migration
- Initial Table Loading
- Disaster Recovery

---

# Security

DynamoDB integrates with

- IAM
- AWS KMS
- CloudTrail
- VPC Endpoints

Supports enterprise security requirements.

---

# Encryption

Server-side encryption protects table data.

Encryption Options

- AWS Owned Key
- AWS Managed Key
- Customer Managed KMS Key

---

# IAM Integration

IAM controls permissions such as

- Read Items
- Write Items
- Delete Items
- Create Tables
- Update Tables

Supports least-privilege access.

---

# EventBridge Integration

Table events can trigger EventBridge workflows through Lambda or Streams.

Example

```text
Order Updated

↓

Stream

↓

Lambda

↓

EventBridge

↓

Notification
```

---

# Amazon S3 Integration

Export data for

- Backup
- Analytics
- Machine Learning
- Compliance

---

# Kinesis Integration

DynamoDB Streams integrate with Kinesis.

Architecture

```text
DynamoDB

↓

Streams

↓

Kinesis

↓

Analytics
```

Useful for real-time processing.

---

# CloudWatch Integration

Monitor

- Read Capacity
- Write Capacity
- Throttled Requests
- Latency
- Errors
- Successful Requests

Configure alarms for operational visibility.

---

# Auto Scaling Monitoring

CloudWatch metrics automatically adjust

- RCUs
- WCUs

based on workload.

---

# AWS CLI

Create Table

```bash
aws dynamodb create-table \
--table-name Users
```

List Tables

```bash
aws dynamodb list-tables
```

Describe Table

```bash
aws dynamodb describe-table \
--table-name Users
```

Put Item

```bash
aws dynamodb put-item \
--table-name Users \
--item file://user.json
```

Query Items

```bash
aws dynamodb query \
--table-name Users
```

---

# Terraform

```hcl
resource "aws_dynamodb_table" "users" {

  name         = "Users"

  billing_mode = "PAY_PER_REQUEST"

  hash_key     = "UserId"

  attribute {

    name = "UserId"

    type = "S"

  }

}
```

---

# CloudFormation

```yaml
Resources:

  UsersTable:

    Type: AWS::DynamoDB::Table

    Properties:

      TableName: Users

      BillingMode: PAY_PER_REQUEST
```

---

# Python (Boto3)

Create Table

```python
import boto3

dynamodb = boto3.client("dynamodb")

response = dynamodb.list_tables()

print(response)
```

Put Item

```python
table = boto3.resource("dynamodb").Table("Users")

table.put_item(

    Item={

        "UserId":"101",

        "Name":"John"

    }

)
```

---

# Enterprise Production Architecture

```text
                Users

                  │

             API Gateway

                  │

               AWS Lambda

                  │

             Amazon DynamoDB

      ┌───────────┼───────────┐

      │           │           │

   DAX Cache   Streams    Global Tables

      │           │           │

 CloudWatch   Lambda     Multi-Region

      │           │           │

 Amazon S3  EventBridge  Analytics
```

---

# Best Practices

- Design efficient partition keys
- Use On-Demand capacity for unpredictable workloads
- Use GSIs only when necessary
- Avoid Scan operations
- Enable Auto Scaling
- Enable PITR
- Encrypt tables with KMS
- Enable CloudWatch alarms
- Use DAX for read-heavy workloads
- Use Global Tables for global applications
- Monitor throttling continuously
- Design idempotent write operations

---

# Common Mistakes

- Choosing poor partition keys
- Overusing Scan
- Creating unnecessary GSIs
- Ignoring throttling
- Disabling backups
- Not enabling PITR
- Large hot partitions
- Hardcoding capacity values
- No monitoring
- No disaster recovery strategy

---

# Troubleshooting

## Throttling Errors

Check

- Read Capacity
- Write Capacity
- Auto Scaling
- Hot Partitions

---

## High Latency

Verify

- DAX
- Network
- Query Design
- Scan Operations

---

## Uneven Traffic Distribution

Check

- Partition Key
- Access Patterns
- Hot Keys

---

## Stream Not Triggering

Verify

- Streams Enabled
- Lambda Permissions
- Event Source Mapping

---

## Global Table Replication Delay

Check

- Region Health
- Network
- Write Conflicts
- CloudWatch Metrics

---

# Interview Questions

## Basic

1. What is Amazon DynamoDB?
2. SQL vs NoSQL?
3. What is a Partition Key?
4. What is a Sort Key?
5. What is an Item?
6. What is an Attribute?
7. What is a GSI?
8. What is an LSI?
9. Provisioned vs On-Demand?
10. What is DynamoDB Streams?

---

## Intermediate

11. Explain Global Tables.
12. Explain DAX.
13. Explain PITR.
14. Explain DynamoDB Streams.
15. Explain Query vs Scan.
16. Explain RCU and WCU.
17. Explain Auto Scaling.
18. Explain TTL.
19. Explain Transactions.
20. Explain Secondary Indexes.

---

## Advanced

21. Design a global e-commerce database using DynamoDB.
22. How would you avoid hot partitions?
23. Explain partition key selection strategy.
24. DynamoDB vs RDS?
25. DynamoDB vs Cassandra?
26. Explain Global Table conflict resolution.
27. Design a serverless backend using DynamoDB.
28. How would you optimize read performance?
29. Explain DynamoDB security architecture.
30. Best practices for enterprise DynamoDB deployments.

---

# Production Scenarios

### Scenario 1

A gaming application serves millions of users worldwide.

How would Global Tables improve user experience?

---

### Scenario 2

A table experiences frequent throttling during peak traffic.

How would you investigate and resolve the issue?

---

### Scenario 3

Your application performs many repeated reads.

How would DAX reduce database latency?

---

### Scenario 4

A user accidentally deletes important records.

How would Point-in-Time Recovery restore the data?

---

### Scenario 5

You need to trigger downstream processing whenever an order is updated.

How would DynamoDB Streams integrate with Lambda?

---

### Scenario 6

Your organization wants to export production data into a data lake.

How would DynamoDB integrate with Amazon S3?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Table | Data Container |
| Item | Record |
| Attribute | Field |
| Partition Key | Data Distribution |
| Sort Key | Ordered Access |
| GSI | Alternate Query Pattern |
| LSI | Alternate Sort Order |
| Streams | Change Data Capture |
| Global Tables | Multi-Region Replication |
| DAX | In-Memory Cache |
| PITR | Point-in-Time Recovery |
| TTL | Automatic Item Expiration |
| On-Demand | Automatic Capacity |
| Provisioned | Fixed Capacity |

---

# Summary

Amazon DynamoDB is a fully managed, serverless NoSQL database designed for high performance, low latency, and automatic scalability. Features such as Global Secondary Indexes, Local Secondary Indexes, Streams, Global Tables, DynamoDB Accelerator (DAX), Point-in-Time Recovery (PITR), Auto Scaling, and server-side encryption enable organizations to build resilient, globally distributed, and highly scalable applications. When integrated with Lambda, API Gateway, Kinesis, EventBridge, CloudWatch, and Amazon S3, DynamoDB serves as a core data platform for modern cloud-native and serverless architectures.