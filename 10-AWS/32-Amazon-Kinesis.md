# Amazon Kinesis

---

# Introduction

Amazon Kinesis is a fully managed, highly scalable real-time data streaming platform that enables organizations to collect, process, analyze, and deliver massive amounts of streaming data with low latency.

Unlike traditional batch processing systems, Kinesis processes data continuously as it arrives, making it ideal for applications that require near real-time analytics, monitoring, fraud detection, log processing, IoT data ingestion, clickstream analysis, and operational intelligence.

Amazon Kinesis integrates with:

- Amazon EC2
- Amazon ECS
- Amazon EKS
- AWS Lambda
- Amazon S3
- Amazon Redshift
- Amazon OpenSearch Service
- Amazon DynamoDB
- Amazon EMR
- AWS Glue
- Amazon CloudWatch
- AWS IAM
- Amazon EventBridge

Kinesis is one of AWS's core services for building real-time data processing pipelines.

---

# What is Amazon Kinesis?

Amazon Kinesis is a streaming data platform.

Instead of storing data and processing it later,

applications process data continuously as it arrives.

Workflow

```text
Producers

↓

Amazon Kinesis

↓

Consumers

↓

Analytics
```

---

# Why Amazon Kinesis?

Without Kinesis

```text
Application

↓

Store Data

↓

Batch Processing

↓

Reports
```

Problems

- High Latency
- Delayed Insights
- Batch Dependency
- Slow Decision Making

With Amazon Kinesis

```text
Application

↓

Streaming Data

↓

Kinesis

↓

Real-Time Processing
```

Organizations receive insights within seconds.

---

# Real World Problem Statement

A global e-commerce company generates

- Website Clicks
- Customer Searches
- Orders
- Payment Events
- Application Logs
- IoT Device Data

Requirements

- Real-Time Analytics
- Fraud Detection
- Monitoring
- Dashboards
- Machine Learning

Amazon Kinesis enables continuous processing of streaming events.

---

# Enterprise Architecture

```text
Applications

EC2  ECS  EKS  IoT  Mobile

             │

             ▼

        Amazon Kinesis

             │

     Stream Processing

             │

 ┌───────────┼────────────┐

 │           │            │

Lambda    OpenSearch   Redshift

 │           │            │

Alerts    Dashboards   Analytics
```

---

# Kinesis Family

Amazon Kinesis consists of

- Kinesis Data Streams
- Kinesis Data Firehose
- Kinesis Data Analytics
- Kinesis Video Streams

Each service solves different streaming problems.

---

# Kinesis Data Streams

Provides real-time streaming data ingestion.

Features

- Low Latency
- High Throughput
- Multiple Consumers
- Replay Capability

Best for custom streaming applications.

---

# Kinesis Data Firehose

Automatically delivers streaming data to destinations.

Supports

- Amazon S3
- Redshift
- OpenSearch
- Splunk

No infrastructure management required.

---

# Kinesis Data Analytics

Processes streaming data using SQL or Apache Flink.

Supports

- Streaming SQL
- Stateful Processing
- Window Functions

Useful for real-time analytics.

---

# Kinesis Video Streams

Streams video data.

Use Cases

- CCTV
- Smart Cameras
- Video Analytics
- Machine Learning
- IoT Devices

---

# Data Stream

A Data Stream is a sequence of continuously arriving records.

Workflow

```text
Producer

↓

Data Stream

↓

Consumers
```

---

# Producer

Producers send data into Kinesis.

Examples

- EC2
- Lambda
- Mobile Apps
- IoT Devices
- Applications
- Log Agents

---

# Consumer

Consumers process records.

Examples

- Lambda
- Kinesis Client Library
- Analytics
- Custom Applications

---

# Record

A Record contains

- Data
- Partition Key
- Sequence Number

Example

```json
{
 "User":"John",
 "Page":"Checkout"
}
```

---

# Partition Key

Partition Keys determine

which shard stores a record.

Example

```text
Customer-101

↓

Shard 1
```

Same key always routes to the same shard.

---

# Sequence Number

Every record receives a unique sequence number.

Used for

- Ordering
- Checkpointing
- Replay

---

# Shard

A Shard is the basic throughput unit.

Each shard supports

Write

```
1 MB/sec
```

Read

```
2 MB/sec
```

More shards provide higher throughput.

---

# Stream Capacity

Capacity depends on

Number of Shards

Example

```text
10 Shards

↓

10 MB/sec Write

↓

20 MB/sec Read
```

---

# Resharding

Shards can be

- Split
- Merged

Architecture

```text
Shard

↓

Split

↓

Shard A

Shard B
```

Supports scaling.

---

# Scaling Streams

Increase shards when

- Write Throughput High
- Read Throughput High
- Consumer Count Increases

---

# Retention Period

Data remains available for replay.

Default

```
24 Hours
```

Maximum

```
365 Days
```

---

# Replay

Consumers can reread previous records.

Example

```text
Yesterday's Events

↓

Replay

↓

Analytics
```

Useful for recovery.

---

# Enhanced Fan-Out

Provides dedicated throughput for each consumer.

Benefits

- Lower Latency
- Independent Consumers
- Better Performance

---

# Shared Throughput

Multiple consumers share shard bandwidth.

Suitable for smaller workloads.

---

# Producer Libraries

AWS provides

- Kinesis Producer Library (KPL)
- AWS SDKs
- REST API

These simplify publishing records.

---

# Consumer Libraries

AWS provides

- Kinesis Client Library (KCL)

Supports

- Checkpointing
- Load Balancing
- Failover

---

# Checkpointing

Consumers store processing progress.

Workflow

```text
Read Record

↓

Process

↓

Checkpoint

↓

Next Record
```

Prevents duplicate processing.

---

# Stream Encryption

Supports server-side encryption using AWS KMS.

Benefits

- Data Protection
- Compliance
- Secure Streaming

---

# Access Control

Amazon Kinesis integrates with

- IAM
- KMS
- CloudTrail

Supports fine-grained permissions.

---

# Monitoring

CloudWatch Metrics

- Incoming Records
- Incoming Bytes
- Read Throughput
- Write Throughput
- Iterator Age
- Throttled Records

---

# Summary

Amazon Kinesis is AWS's real-time streaming platform for ingesting, processing, and analyzing continuous streams of data. Services such as Kinesis Data Streams, Data Firehose, Data Analytics, and Video Streams enable organizations to build scalable, low-latency streaming architectures for analytics, monitoring, IoT, fraud detection, and operational intelligence.

---

# Kinesis Data Firehose

Amazon Kinesis Data Firehose is a fully managed delivery service that automatically captures, transforms, and loads streaming data into storage and analytics services.

Unlike Kinesis Data Streams, Firehose does not require shard management.

Supported Destinations

- Amazon S3
- Amazon Redshift
- Amazon OpenSearch Service
- Splunk
- HTTP Endpoints

Architecture

```text
Producers

↓

Firehose

↓

Transformation

↓

Destination
```

---

# Firehose Buffering

Firehose buffers data before delivery.

Buffer Conditions

- Buffer Size
- Buffer Time

Example

```text
5 MB

or

300 Seconds

↓

Deliver
```

Reduces API calls and improves efficiency.

---

# Firehose Data Transformation

Firehose integrates with AWS Lambda.

Workflow

```text
Streaming Data

↓

Lambda

↓

Transform

↓

Destination
```

Common transformations

- JSON Formatting
- Filtering
- Data Enrichment
- Compression

---

# Firehose Data Compression

Supported formats

- GZIP
- ZIP
- Snappy
- Hadoop Snappy

Benefits

- Lower Storage Cost
- Faster Analytics

---

# Kinesis Data Analytics

Kinesis Data Analytics processes streaming data in real time.

Supports

- SQL
- Apache Flink

Use Cases

- Fraud Detection
- Live Dashboards
- Operational Metrics
- IoT Analytics

---

# Streaming SQL

Example

```sql
SELECT COUNT(*)

FROM STREAM

GROUP BY WINDOW
```

Processes records continuously.

---

# Apache Flink

Apache Flink enables

- Stateful Processing
- Event Time Processing
- Windowing
- Complex Event Processing

Suitable for enterprise streaming applications.

---

# Window Processing

Examples

- Sliding Window
- Tumbling Window
- Session Window

Architecture

```text
Incoming Events

↓

Window

↓

Aggregation

↓

Output
```

---

# Kinesis Video Streams

Designed for

- Video Streaming
- CCTV
- Smart Cameras
- Machine Vision
- Surveillance

Supports real-time video ingestion.

---

# Lambda Integration

Lambda automatically processes stream records.

Workflow

```text
Kinesis Stream

↓

Lambda

↓

Processing

↓

Database
```

Serverless event processing.

---

# Amazon S3 Integration

Store streaming data in Amazon S3.

Architecture

```text
Kinesis

↓

Firehose

↓

Amazon S3

↓

Data Lake
```

Useful for long-term storage.

---

# Amazon Redshift Integration

Stream analytics data into Redshift.

Architecture

```text
Kinesis Firehose

↓

Amazon S3

↓

COPY

↓

Amazon Redshift
```

Supports real-time business intelligence.

---

# OpenSearch Integration

Send streaming logs directly to OpenSearch.

Workflow

```text
Logs

↓

Firehose

↓

OpenSearch

↓

Dashboards
```

Useful for observability.

---

# CloudWatch Integration

Monitor

- Incoming Records
- Incoming Bytes
- Write Throughput
- Read Throughput
- Iterator Age
- Put Record Success
- Get Record Success

CloudWatch alarms detect streaming issues.

---

# EventBridge Integration

Stream events trigger downstream automation.

Example

```text
Kinesis Event

↓

EventBridge

↓

Lambda

↓

Notification
```

---

# Security

Amazon Kinesis integrates with

- IAM
- AWS KMS
- CloudTrail
- VPC Endpoints

Supports encryption

- At Rest
- In Transit

---

# High Availability

Kinesis replicates data across multiple Availability Zones.

Benefits

- High Durability
- Fault Tolerance
- Automatic Recovery

---

# Scaling Best Practices

Scale by

- Increasing Shards
- Enhanced Fan-Out
- Parallel Consumers

Monitor CloudWatch metrics continuously.

---

# AWS CLI

Create Stream

```bash
aws kinesis create-stream \
--stream-name orders \
--shard-count 2
```

List Streams

```bash
aws kinesis list-streams
```

Describe Stream

```bash
aws kinesis describe-stream \
--stream-name orders
```

Put Record

```bash
aws kinesis put-record \
--stream-name orders \
--partition-key customer1 \
--data "Order Created"
```

---

# Terraform

```hcl
resource "aws_kinesis_stream" "orders" {

  name = "orders"

  shard_count = 2

  retention_period = 24

}
```

---

# CloudFormation

```yaml
Resources:

  OrderStream:

    Type: AWS::Kinesis::Stream

    Properties:

      Name: orders

      ShardCount: 2
```

---

# Python (Boto3)

```python
import boto3

kinesis = boto3.client("kinesis")

response = kinesis.list_streams()

print(response)
```

Put Record

```python
kinesis.put_record(

    StreamName="orders",

    Data="Order Created",

    PartitionKey="customer1"

)
```

---

# Enterprise Production Architecture

```text
           Applications & Devices

 EC2  ECS  EKS  Mobile  IoT  Logs

                  │

                  ▼

        Amazon Kinesis Data Streams

                  │

        ┌─────────┼─────────┐

        │         │         │

     Lambda   Firehose   Analytics

        │         │         │

 OpenSearch   Amazon S3  Redshift

        │         │         │

 Dashboards  Data Lake   BI Reports

                  │

      CloudWatch • EventBridge
```

---

# Best Practices

- Use appropriate partition keys
- Enable KMS encryption
- Monitor shard utilization
- Use Enhanced Fan-Out for multiple consumers
- Enable CloudWatch alarms
- Archive data to S3
- Compress Firehose deliveries
- Design idempotent consumers
- Monitor Iterator Age
- Scale shards proactively

---

# Common Mistakes

- Hot partition keys
- Too few shards
- Ignoring throttling
- No checkpointing
- Large record sizes
- Missing encryption
- No monitoring
- Hardcoded shard assumptions
- Ignoring retry logic
- Poor consumer scaling

---

# Troubleshooting

## Write Throughput Exceeded

Check

- Number of Shards
- Partition Key Distribution
- Producer Rate

---

## High Iterator Age

Verify

- Consumer Speed
- Processing Time
- Number of Consumers

---

## Consumer Not Receiving Records

Check

- Stream Status
- IAM Permissions
- Shard Iterator
- Consumer Application

---

## Firehose Delivery Failed

Verify

- Destination Permissions
- Buffer Settings
- Lambda Transformation
- S3 Bucket Policy

---

## Uneven Shard Utilization

Check

- Partition Keys
- Record Distribution
- Producer Logic

---

# Interview Questions

## Basic

1. What is Amazon Kinesis?
2. Kinesis Data Streams vs Firehose?
3. What is a Shard?
4. What is a Record?
5. What is a Partition Key?
6. What is Enhanced Fan-Out?
7. What is Firehose?
8. What is Kinesis Data Analytics?
9. What is Kinesis Video Streams?
10. What is Stream Retention?

---

## Intermediate

11. Explain shard scaling.
12. Explain checkpointing.
13. Explain replay capability.
14. Explain Firehose buffering.
15. Explain Lambda integration.
16. Explain Redshift integration.
17. Explain OpenSearch integration.
18. Explain stream encryption.
19. Explain KCL.
20. Explain CloudWatch monitoring.

---

## Advanced

21. Design a real-time analytics platform.
22. Design clickstream analytics using Kinesis.
23. Explain shard splitting and merging.
24. Design fraud detection architecture.
25. Kinesis vs Kafka?
26. Kinesis vs SQS?
27. How would you process millions of events per second?
28. Explain event ordering.
29. Design enterprise streaming architecture.
30. Best practices for production Kinesis deployments.

---

# Production Scenarios

### Scenario 1

A website generates millions of click events every hour.

How would Amazon Kinesis process this data in real time?

---

### Scenario 2

Your consumers are falling behind and Iterator Age continues increasing.

How would you troubleshoot and scale the solution?

---

### Scenario 3

Your analytics team needs all streaming data stored in Amazon S3.

How would Firehose simplify the architecture?

---

### Scenario 4

Fraud detection requires processing transactions within seconds.

Which Kinesis services would you use?

---

### Scenario 5

A single shard reaches its throughput limit.

How would you resolve the issue?

---

### Scenario 6

Your enterprise wants live dashboards from application logs.

How would Kinesis integrate with OpenSearch and CloudWatch?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Data Streams | Real-Time Streaming |
| Firehose | Managed Data Delivery |
| Data Analytics | SQL/Flink Processing |
| Video Streams | Video Ingestion |
| Shard | Throughput Unit |
| Record | Stream Data |
| Partition Key | Record Distribution |
| Sequence Number | Record Ordering |
| Enhanced Fan-Out | Dedicated Consumer Throughput |
| KCL | Consumer Library |
| Firehose Buffer | Batch Delivery |
| Replay | Reprocess Stream Data |

---

# Summary

Amazon Kinesis is AWS's fully managed real-time streaming platform that enables organizations to ingest, process, analyze, and deliver massive volumes of streaming data with low latency. Services such as Kinesis Data Streams, Data Firehose, Data Analytics, and Video Streams support use cases including clickstream analytics, IoT, fraud detection, log processing, and operational monitoring. Combined with Lambda, S3, Redshift, OpenSearch, CloudWatch, and EventBridge, Amazon Kinesis provides a scalable, highly available, and enterprise-ready foundation for modern real-time data architectures.