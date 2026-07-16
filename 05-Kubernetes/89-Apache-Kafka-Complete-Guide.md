# Apache Kafka - Complete Guide

## Introduction

One of the biggest challenges in modern distributed systems is:

```text
How Services Communicate Reliably?
```

---

Before Kafka, applications usually communicated directly.

Example:

```text
Order Service

↓

Payment Service

↓

Notification Service
```

---

This creates:

```text
Tight Coupling

Dependency Issues

Scaling Problems
```

---

# Why Kafka Was Created?

Imagine:

```text
Customer Places Order
```

---

Flow

```text
Order Service

↓

Payment Service
```

---

Problem

```text
Payment Service Down
```

---

Result

```text
Order Creation Fails
```

---

Another Problem

```text
1 Million Events

Per Day
```

---

Direct communication becomes difficult.

---

Need:

```text
Reliable

Scalable

Asynchronous

Communication
```

---

Kafka Solves This Problem.

---

# Real Production Example

Traditional Architecture

```text
Order Service

↓

Payment Service

↓

Notification Service
```

---

Kafka Architecture

```text
Order Service

↓

Kafka

↓

Payment Service

Notification Service

Analytics Service
```

---

If Notification Service Fails

```text
Order Processing Continues
```

---

This is called:

```text
Loose Coupling
```

---

# What Is Kafka?

Kafka is a:

```text
Distributed Event Streaming Platform
```

---

Used For:

```text
Messaging

Event Streaming

Data Pipelines

Log Processing

Real-Time Analytics
```

---

# Core Components

Kafka consists of:

```text
Producer

Topic

Partition

Broker

Consumer

Consumer Group
```

---

# Kafka Architecture

```text
Producer

↓

Topic

↓

Partition

↓

Broker

↓

Consumer Group

↓

Consumers
```

---

# Producer

A Producer sends messages to Kafka.

---

Example

```text
Order Service
```

publishes

```text
Order Created
```

event.

---

Flow

```text
Order Service

↓

Kafka Producer

↓

Orders Topic
```

---

# Topic

A Topic is a logical category where messages are stored.

---

Examples

```text
orders

payments

notifications

audit-events
```

---

Think Of Topic Like

```text
Database Table
```

---

Messages are stored inside Topics.

---

# Partitions

Most Important Kafka Concept.

---

A Topic is divided into:

```text
Partitions
```

---

Example

Topic

```text
orders
```

---

Partitions

```text
Partition-0

Partition-1

Partition-2
```

---

Why?

Because partitions provide:

```text
Scalability

Parallel Processing

High Throughput
```

---

# Without Partitions

```text
One Queue

One Consumer
```

---

Slow.

---

# With Partitions

```text
Partition-0

Partition-1

Partition-2
```

---

Multiple Consumers Process Messages.

---

Faster.

---

# Brokers

Kafka Server = Broker

---

Example

```text
Broker-1

Broker-2

Broker-3
```

---

All Topics And Partitions Are Stored Across Brokers.

---

# Production Example

```text
Broker-1

Partition-0
```

---

```text
Broker-2

Partition-1
```

---

```text
Broker-3

Partition-2
```

---

Distributes Load.

---

# Replication

Production Kafka Uses Replication.

---

Example

```text
Partition-0

Leader → Broker-1

Follower → Broker-2

Follower → Broker-3
```

---

# Why Replication?

Suppose

```text
Broker-1 Fails
```

---

Kafka Promotes

```text
Broker-2
```

as Leader.

---

Result

```text
No Data Loss
```

---

High Availability.

---

# Producer Flow

Producer Sends Message

```text
Order Created
```

---

Kafka Stores

```text
Topic → Partition
```

---

Message Written To Leader.

---

Followers Replicate Data.

---

Flow

```text
Producer

↓

Leader Partition

↓

Follower Replicas
```

---

# Consumers

Consumers Read Messages.

---

Example

```text
Payment Service
```

consumes:

```text
Order Events
```

---

Flow

```text
Kafka

↓

Payment Service
```

---

# Consumer Groups

One Of The Most Asked Interview Topics.

---

Suppose

Topic

```text
Orders
```

has

```text
3 Partitions
```

---

Consumers

```text
Consumer-1

Consumer-2

Consumer-3
```

---

Kafka Distributes Partitions.

---

Example

```text
Consumer-1 → Partition-0

Consumer-2 → Partition-1

Consumer-3 → Partition-2
```

---

Result

```text
Parallel Processing
```

---

# Why Consumer Groups?

Without Consumer Groups

```text
One Consumer
```

Processes Everything.

---

With Consumer Groups

```text
Multiple Consumers

Share Work
```

---

# Offsets

Kafka Stores Messages Sequentially.

---

Example

```text
Offset 0

Offset 1

Offset 2

Offset 3
```

---

Consumer Tracks:

```text
Current Position
```

using Offset.

---

Example

```text
Offset = 100
```

means

```text
100 Messages Already Read
```

---

# Why Offsets Matter?

Suppose Consumer Crashes.

---

Restart.

---

Consumer Continues From

```text
Last Offset
```

---

No Data Loss.

---

# Complete Message Flow

Customer Places Order.

---

Flow

```text
Order Service

↓

Kafka Producer

↓

Orders Topic

↓

Partition

↓

Broker

↓

Consumer Group

↓

Payment Service
```

---

After Payment

```text
Payment Event

↓

Kafka

↓

Notification Service
```

---

# Kafka Retention

Kafka Does Not Immediately Delete Messages.

---

Messages Can Be Retained For

```text
Hours

Days

Weeks
```

---

Example

```text
7 Days
```

---

Consumers Can Re-read Messages.

---

# Why Kafka Is Fast?

Reasons:

```text
Sequential Disk Writes

Partitions

Batch Processing

Zero Copy

Replication Design
```

---

Kafka is designed for:

```text
Millions Of Messages Per Second
```

---

# Kafka In Kubernetes

Kafka is Stateful.

---

Uses

```text
StatefulSet
```

---

Because

```text
Persistent Identity

Persistent Storage
```

are required.

---

# Architecture

```text
Kafka Broker

↓

Persistent Volume
```

---

If Pod Restarts

```text
Data Remains
```

---

# ZooKeeper (Old Architecture)

Older Kafka Versions

```text
Kafka

↓

ZooKeeper
```

---

ZooKeeper Managed

```text
Leader Election

Metadata

Cluster Coordination
```

---

# KRaft (New Architecture)

Modern Kafka Uses:

```text
KRaft
```

---

No ZooKeeper Required.

---

Benefits

```text
Simpler

Faster

Less Infrastructure
```

---

# Real Production Use Cases

## Order Processing

```text
Orders

↓

Kafka

↓

Payment
```

---

## Payment Processing

```text
Payments

↓

Kafka

↓

Notifications
```

---

## Log Aggregation

```text
Applications

↓

Kafka

↓

ELK
```

---

## Audit Events

```text
User Login

↓

Kafka

↓

SIEM
```

---

## Clickstream Analytics

```text
User Clicks

↓

Kafka

↓

Analytics Platform
```

---

# Kafka vs RabbitMQ

## Kafka

Best For

```text
Event Streaming

Large Scale Data

Analytics

High Throughput
```

---

## RabbitMQ

Best For

```text
Task Queues

Traditional Messaging

Request Processing
```

---

# Kafka Monitoring

Monitor

```text
Broker Health

Consumer Lag

Partitions

Disk Usage

Replication Status
```

---

# What Is Consumer Lag?

Very Important Production Metric.

---

Producer

```text
10000 Messages
```

---

Consumer Read

```text
9000 Messages
```

---

Lag

```text
1000 Messages
```

---

Large Lag Means

```text
Consumers Are Slow
```

---

# Common Failure Scenarios

## Broker Failure

Replication Handles It.

---

Follower Becomes Leader.

---

## Consumer Failure

Messages Stay In Kafka.

---

Consumer Restarts.

---

Reads From Last Offset.

---

## Producer Failure

Consumers Continue Reading Existing Messages.

---

## Disk Full

Serious Production Problem.

---

Result

```text
Kafka Cannot Store Messages
```

---

# Troubleshooting Commands

List Topics

```bash
kafka-topics.sh --list
```

---

Describe Topic

```bash
kafka-topics.sh --describe --topic orders
```

---

Consumer Groups

```bash
kafka-consumer-groups.sh --describe
```

---

Check Offsets

```bash
kafka-consumer-groups.sh --describe \
--group payment-group
```

---

# Common Interview Questions

## What Problem Does Kafka Solve?

### Short Answer

Kafka solves reliable, scalable, asynchronous communication between distributed systems.

### Detailed Explanation

Direct service-to-service communication creates tight coupling and failure dependencies. Kafka acts as a durable event streaming platform that decouples producers and consumers.

### Production Example

```text
Order Service

↓

Kafka

↓

Payment Service

Notification Service

Analytics Service
```

### Follow-Up Questions

- Why not use REST APIs?
- What is asynchronous communication?
- What is event streaming?
- Why is Kafka popular?

---

## What Is A Topic?

### Short Answer

A Topic is a logical category used to store messages.

### Detailed Explanation

Producers write messages to Topics, and consumers read messages from Topics. Topics are divided into partitions for scalability.

### Production Example

```text
orders

payments

notifications
```

### Follow-Up Questions

- Can Topics have multiple partitions?
- Who creates Topics?
- How are messages stored?

---

## What Is A Partition?

### Short Answer

A Partition is a subset of a Topic that enables parallel processing and scalability.

### Detailed Explanation

Kafka distributes messages across partitions, allowing multiple consumers to process data simultaneously.

### Production Example

```text
Orders Topic

↓

Partition-0

Partition-1

Partition-2
```

### Follow-Up Questions

- Why are partitions important?
- How many partitions should we create?
- What happens when partitions increase?

---

## What Is A Consumer Group?

### Short Answer

A Consumer Group is a set of consumers working together to process messages from a Topic.

### Detailed Explanation

Kafka distributes partitions among consumers within the same group, enabling horizontal scaling.

### Production Example

```text
Consumer-1 → Partition-0

Consumer-2 → Partition-1

Consumer-3 → Partition-2
```

### Follow-Up Questions

- Can two consumers read the same partition?
- What happens if a consumer crashes?
- What is rebalancing?

---

## What Is Offset?

### Short Answer

Offset represents a consumer's position within a partition.

### Detailed Explanation

Kafka stores messages sequentially, and consumers track their progress using offsets.

### Production Example

```text
Offset = 100

↓

100 Messages Read
```

### Follow-Up Questions

- Where are offsets stored?
- What happens after restart?
- Can offsets be reset?

---

## How Does Kafka Achieve High Availability?

### Short Answer

Kafka uses partition replication across multiple brokers.

### Detailed Explanation

Each partition has a leader and follower replicas. If the leader fails, a follower is promoted.

### Production Example

```text
Leader → Broker-1

Follower → Broker-2

Follower → Broker-3
```

### Follow-Up Questions

- What is ISR?
- What is leader election?
- What happens during broker failure?

---

## Kafka vs RabbitMQ?

### Short Answer

Kafka is optimized for event streaming and high throughput, while RabbitMQ is optimized for traditional messaging and task queues.

### Detailed Explanation

Kafka retains messages and supports replay, while RabbitMQ focuses on immediate delivery and acknowledgment-based processing.

### Production Example

```text
Kafka

↓

Order Events

Analytics

Streaming
```

```text
RabbitMQ

↓

Background Tasks

Email Queue

Job Queue
```

### Follow-Up Questions

- Which is better for microservices?
- Which is better for analytics?
- Which supports replay?

---

# Key Takeaways

```text
Kafka is a distributed event streaming platform.

Kafka solves asynchronous communication problems.

Topics store messages.

Partitions provide scalability and parallel processing.

Brokers are Kafka servers.

Replication provides high availability.

Consumer Groups provide horizontal scaling.

Offsets track consumer progress.

Kafka is widely used for event streaming, analytics, logging, and microservices.

Modern Kafka uses KRaft instead of ZooKeeper.

Kafka is one of the most important technologies in modern distributed systems.
```