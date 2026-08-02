# Amazon Simple Queue Service (Amazon SQS)

---

# Introduction

Amazon Simple Queue Service (Amazon SQS) is a fully managed, highly scalable, secure, and reliable message queuing service that enables applications, microservices, and distributed systems to communicate asynchronously.

Instead of applications communicating directly with each other, messages are stored in a queue until they are processed by consumers. This decouples application components, improves scalability, increases fault tolerance, and enhances system reliability.

Amazon SQS integrates with:

- AWS Lambda
- Amazon EC2
- Amazon ECS
- Amazon EKS
- Amazon SNS
- Amazon EventBridge
- AWS Step Functions
- AWS Batch
- API Gateway
- Amazon CloudWatch
- AWS IAM
- AWS KMS

Amazon SQS is one of the most widely used messaging services in enterprise cloud architectures.

---

# What is Amazon SQS?

Amazon SQS is a managed message queue.

Instead of sending requests directly,

applications place messages into a queue.

Consumers process those messages independently.

Workflow

```text
Producer

↓

Amazon SQS

↓

Consumer
```

Applications become loosely coupled.

---

# Why Amazon SQS?

Without SQS

```text
Application A

↓

Application B

↓

Application C
```

Problems

- Tight Coupling
- Failures Affect Entire System
- Poor Scalability
- Difficult Maintenance

With Amazon SQS

```text
Producer

↓

Queue

↓

Consumer
```

Applications communicate asynchronously.

---

# Real World Problem Statement

An online shopping platform receives thousands of orders every minute.

Each order requires

- Payment Processing
- Inventory Update
- Invoice Generation
- Email Notification
- Shipping

Instead of processing everything immediately,

orders are stored in Amazon SQS.

Workers process messages independently.

---

# Enterprise Architecture

```text
Users

↓

Application

↓

Amazon SQS

↓

Worker Nodes

↓

Database

↓

Email Service
```

---

# Core Components

Amazon SQS consists of

- Queue
- Message
- Producer
- Consumer
- Visibility Timeout
- Long Polling
- Dead Letter Queue
- Message Retention
- Delay Queue
- FIFO Queue
- Standard Queue

---

# Queue

A Queue stores messages.

Workflow

```text
Producer

↓

Queue

↓

Consumer
```

Messages remain in the queue until processed.

---

# Message

A Message contains

- Message Body
- Message Attributes
- Message ID
- Receipt Handle
- Timestamp

Example

```json
{
  "OrderId":"1001",
  "Customer":"John",
  "Amount":"500"
}
```

---

# Producer

A Producer sends messages to the queue.

Examples

- EC2
- Lambda
- API Gateway
- EventBridge
- Applications

---

# Consumer

Consumers read messages from the queue.

Examples

- Lambda
- ECS
- EC2
- Kubernetes Pods
- Worker Applications

---

# Standard Queue

Standard Queues provide

- Nearly Unlimited Throughput
- At-Least-Once Delivery
- Best-Effort Ordering

Suitable for most applications.

---

# FIFO Queue

FIFO means

First In

First Out

Features

- Exactly Once Processing
- Strict Message Ordering

Useful for

- Banking
- Financial Transactions
- Inventory
- Order Processing

---

# Standard vs FIFO

| Standard Queue | FIFO Queue |
|---------------|------------|
| Unlimited Throughput | Limited Throughput |
| Best Effort Ordering | Strict Ordering |
| At Least Once | Exactly Once |
| High Performance | Ordered Processing |

---

# Message Lifecycle

```text
Producer

↓

Queue

↓

Consumer

↓

Delete Message
```

A message exists until deleted or expired.

---

# Visibility Timeout

After a consumer receives a message,

the message becomes temporarily invisible.

Workflow

```text
Consumer

↓

Message Hidden

↓

Processing

↓

Delete Message
```

If processing fails,

the message becomes visible again.

---

# Why Visibility Timeout?

Without Visibility Timeout

```text
Consumer A

↓

Message

↓

Consumer B

↓

Duplicate Processing
```

Visibility Timeout prevents multiple consumers from processing the same message simultaneously.

---

# Message Retention Period

Messages remain in the queue even if they are not processed.

Retention Period

- Minimum

1 Minute

- Maximum

14 Days

Default

4 Days

---

# Delay Queue

Delay Queues postpone message delivery.

Example

```text
Producer

↓

Queue

↓

Delay

↓

Consumer
```

Useful for scheduled processing.

---

# Message Delay

Individual messages can also have delays.

Example

```text
Order Created

↓

Delay 10 Minutes

↓

Processing Starts
```

---

# Long Polling

Long Polling waits for messages before returning.

Without Long Polling

```text
Consumer

↓

No Message

↓

Repeated Requests
```

With Long Polling

```text
Consumer

↓

Wait

↓

Message Arrives

↓

Receive
```

Benefits

- Reduced Cost
- Lower API Calls
- Better Performance

---

# Short Polling

Short Polling immediately returns available messages.

May result in

- Empty Responses
- Higher Costs

---

# Message Size

Maximum Message Size

```
256 KB
```

Larger payloads typically use

Amazon S3

with SQS references.

---

# Batch Operations

SQS supports batch processing.

Operations

- Send
- Receive
- Delete

Benefits

- Reduced API Calls
- Lower Cost
- Better Performance

---

# Message Attributes

Messages may contain metadata.

Examples

- Priority
- Department
- Region
- Environment

Useful for processing decisions.

---

# Queue Attributes

Common attributes

- Visibility Timeout
- Delay Seconds
- Retention Period
- Receive Wait Time
- Maximum Message Size

---

# Dead Letter Queue (DLQ)

Messages that repeatedly fail move to a Dead Letter Queue.

Workflow

```text
Main Queue

↓

Processing Failed

↓

Retry

↓

Maximum Retries

↓

Dead Letter Queue
```

DLQs simplify troubleshooting.

---

# Redrive Policy

Defines

- Maximum Receive Count
- Target DLQ

Example

```text
Retry

5 Times

↓

DLQ
```

---

# Exactly Once Processing

Supported only by FIFO Queues.

Benefits

- No Duplicate Processing
- Ordered Execution

---

# At Least Once Delivery

Standard Queues may deliver messages more than once.

Applications should be idempotent.

---

# Message Ordering

FIFO Queues preserve message order.

Example

```text
Order 1

↓

Order 2

↓

Order 3
```

Messages are processed sequentially.

---

# Deduplication

FIFO Queues support message deduplication.

Duplicate messages are automatically ignored.

---

# Message Group ID

FIFO Queues process messages within a Message Group sequentially.

Example

```text
Customer A

↓

Messages

↓

Ordered Processing
```

Different groups can process concurrently.

---

# Queue Encryption

Amazon SQS supports server-side encryption.

Encryption options

- AWS Managed Key
- Customer Managed KMS Key

---

# Access Control

SQS integrates with

- IAM
- Resource Policies
- KMS

Provides secure access management.

---

# Monitoring

CloudWatch Metrics

- Queue Depth
- Messages Sent
- Messages Received
- Messages Deleted
- Oldest Message Age

Useful for capacity planning.

---

# Summary

Amazon SQS is a fully managed message queuing service that enables asynchronous communication between distributed applications. Core concepts such as Standard Queues, FIFO Queues, Visibility Timeout, Long Polling, Message Retention, Delay Queues, Dead Letter Queues, Deduplication, and Message Groups provide the foundation for building scalable, fault-tolerant, and loosely coupled cloud architectures.

---