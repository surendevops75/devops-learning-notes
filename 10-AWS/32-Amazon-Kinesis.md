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

