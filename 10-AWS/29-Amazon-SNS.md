# Amazon Simple Notification Service (Amazon SNS)

---

# Introduction

Amazon Simple Notification Service (Amazon SNS) is a fully managed, highly available, serverless messaging service that enables applications, AWS services, and users to communicate using a publish-subscribe (Pub/Sub) messaging model.

In modern cloud architectures, applications often need to notify multiple systems when an event occurs. Instead of creating direct integrations between every application, Amazon SNS provides a central messaging service that distributes notifications to multiple subscribers simultaneously.

Amazon SNS integrates with:

- AWS Lambda
- Amazon SQS
- Amazon EventBridge
- Amazon CloudWatch
- AWS Config
- AWS Backup
- AWS Systems Manager
- Amazon EC2
- Amazon S3
- AWS CodePipeline
- AWS CodeBuild
- Amazon API Gateway
- AWS Step Functions
- Mobile Push Notifications
- Email
- SMS

SNS is one of the foundational messaging services used in event-driven and serverless architectures.

---

# What is Amazon SNS?

Amazon SNS is a Publish/Subscribe messaging service.

One application publishes a message.

Multiple subscribers receive the same message.

Workflow

```text
Publisher

↓

SNS Topic

↓

Subscribers
```

The publisher does not need to know who receives the message.

---

# Why Amazon SNS?

Without SNS

```text
Application

↓

Email Service

↓

SMS Service

↓

Lambda

↓

Monitoring
```

Problems

- Tight Coupling
- Multiple Integrations
- Difficult Maintenance
- Poor Scalability

With SNS

```text
Application

↓

SNS Topic

↓

Email

SMS

Lambda

SQS

HTTP Endpoint
```

One message reaches multiple subscribers simultaneously.

---

# Real World Problem Statement

An e-commerce platform generates an event whenever an order is placed.

The following systems must be notified:

- Inventory Service
- Billing Service
- Shipping Service
- Customer Notification
- Analytics Platform

Instead of integrating directly with every service,

the application publishes one SNS message.

SNS distributes it automatically.

---

# Enterprise Architecture

```text
Applications

EC2  Lambda  ECS  EKS

        │

        ▼

     Amazon SNS

        │

 ┌──────┼──────────┬─────────┬─────────┐

 │      │          │         │

Email  SMS      Lambda     SQS

 │      │          │         │

Users  Mobile  Processing  Queue

               │

         CloudWatch
```

---

# Core Components

Amazon SNS consists of

- Topics
- Publishers
- Subscribers
- Subscriptions
- Messages
- Message Attributes
- Message Filtering
- Delivery Policies
- Dead Letter Queues
- FIFO Topics

---

# Topic

A Topic is the communication channel.

Publishers send messages to Topics.

Subscribers receive messages from Topics.

Architecture

```text
Publisher

↓

Topic

↓

Subscribers
```

---

# Topic Types

SNS supports

- Standard Topics
- FIFO Topics

---

# Standard Topic

Characteristics

- High Throughput
- Best-Effort Ordering
- At-Least-Once Delivery
- Low Latency

Recommended for most applications.

---

# FIFO Topic

FIFO Topics guarantee

- Message Ordering
- Exactly-Once Delivery

Useful for

- Banking
- Financial Systems
- Order Processing

---

# Publisher

A Publisher sends messages to an SNS Topic.

Examples

- EC2
- Lambda
- EventBridge
- CloudWatch
- Applications
- AWS Backup

---

# Subscriber

Subscribers receive SNS messages.

Supported Subscribers

- Email
- SMS
- Lambda
- Amazon SQS
- HTTP
- HTTPS
- Mobile Push

---

# Subscription

A Subscription connects a Topic to a Subscriber.

Example

```text
Topic

↓

Email

↓

Notification Delivered
```

---

# Publish-Subscribe Model

Architecture

```text
Publisher

↓

SNS Topic

↓

Subscriber A

Subscriber B

Subscriber C

Subscriber D
```

Every subscriber receives a copy.

---

# Fan-Out Pattern

SNS enables Fan-Out messaging.

Example

```text
Order Created

↓

SNS

↓

Billing

Inventory

Shipping

Analytics
```

All services process the event independently.

---

# Message

A Message is the payload sent through SNS.

Example

```json
{
  "OrderId":"1001",
  "Status":"Created"
}
```

---

# Message Attributes

Messages may contain metadata.

Examples

- Environment
- Priority
- Region
- Department

Useful for filtering.

---

# Message Filtering

Subscribers receive only relevant messages.

Example

```text
Priority

↓

High

↓

Operations Queue

----------------

Priority

↓

Low

↓

Analytics Queue
```

Improves efficiency.

---

# Filter Policies

Example

```text
Department

↓

Finance

↓

Finance Queue

----------------

Department

↓

HR

↓

HR Queue
```

Each subscriber processes only matching messages.

---

# Message Delivery

SNS guarantees

- Durable Storage
- At-Least-Once Delivery
- Automatic Retry

---

# Retry Policy

If delivery fails,

SNS retries automatically.

Workflow

```text
Publish

↓

Delivery Failed

↓

Retry

↓

Success
```

---

# Dead Letter Queue (DLQ)

Failed messages can be stored in Amazon SQS.

Architecture

```text
SNS

↓

Delivery Failed

↓

Dead Letter Queue
```

Prevents message loss.

---

# Delivery Status

SNS tracks delivery success for

- SMS
- Mobile Push
- HTTP
- Lambda

Useful for monitoring.

---

# Email Notifications

Workflow

```text
Application

↓

SNS

↓

Email

↓

User
```

Common for operational alerts.

---

# SMS Notifications

Workflow

```text
SNS

↓

SMS

↓

Mobile Phone
```

Useful for critical alerts.

---

# Mobile Push Notifications

Supports

- Android
- iOS
- Fire OS

---

# HTTP/HTTPS Endpoints

SNS can invoke REST APIs.

Architecture

```text
SNS

↓

HTTPS Endpoint

↓

Application
```

---

# Lambda Integration

SNS directly invokes Lambda.

Workflow

```text
SNS

↓

Lambda

↓

Business Logic
```

---

# Amazon SQS Integration

SNS and SQS are commonly used together.

Architecture

```text
Publisher

↓

SNS

↓

Multiple SQS Queues

↓

Consumers
```

Provides scalable asynchronous processing.

---

# EventBridge Integration

EventBridge can publish events to SNS.

Example

```text
EC2 Stopped

↓

EventBridge

↓

SNS

↓

Operations Team
```

---

# CloudWatch Integration

CloudWatch alarms publish notifications to SNS.

Workflow

```text
CPU > 80%

↓

Alarm

↓

SNS

↓

Email
```

---

# AWS Config Integration

Compliance violations trigger SNS notifications.

---

# AWS Backup Integration

Backup failures automatically notify administrators.

---

# Security

SNS integrates with

- IAM
- KMS
- VPC Endpoints
- CloudTrail

Supports encryption and auditing.

---

# Server-Side Encryption

SNS Topics support AWS KMS encryption.

Workflow

```text
Publisher

↓

Encrypted Topic

↓

Subscribers
```

---

# Access Policies

SNS Topic Policies control access.

Example

Allow

- EventBridge

Deny

- External Accounts

---

# AWS CLI

Create Topic

```bash
aws sns create-topic \
--name production-alerts
```

List Topics

```bash
aws sns list-topics
```

Publish Message

```bash
aws sns publish \
--topic-arn <topic-arn> \
--message "Deployment Completed"
```

Subscribe

```bash
aws sns subscribe \
--topic-arn <topic-arn> \
--protocol email \
--notification-endpoint admin@company.com
```

---

# Terraform

```hcl
resource "aws_sns_topic" "alerts" {

  name = "production-alerts"

}
```

Subscription

```hcl
resource "aws_sns_topic_subscription" "email" {

  topic_arn = aws_sns_topic.alerts.arn

  protocol = "email"

  endpoint = "admin@company.com"

}
```

---

# CloudFormation

```yaml
Resources:

  AlertTopic:

    Type: AWS::SNS::Topic

    Properties:

      TopicName: production-alerts
```

---

# Python (Boto3)

```python
import boto3

sns = boto3.client("sns")

response = sns.list_topics()

print(response)
```

Publish

```python
sns.publish(

    TopicArn="arn:aws:sns:region:account:alerts",

    Message="Deployment Successful"

)
```

---

# Enterprise Production Architecture

```text
             AWS Applications

 EC2  ECS  Lambda  EventBridge

              │

              ▼

           Amazon SNS

              │

 ┌────────────┼────────────┬────────────┐

 │            │            │

 Lambda      Email       Amazon SQS

 │            │            │

Automation  Operations   Background Jobs

              │

         CloudWatch

              │

        Operations Team
```

---

# Best Practices

- Use Topics for logical grouping
- Enable KMS encryption
- Use Message Filtering
- Configure Dead Letter Queues
- Monitor failed deliveries
- Use SNS with SQS for fan-out architectures
- Apply least-privilege IAM policies
- Enable CloudTrail logging
- Separate development and production topics
- Monitor CloudWatch metrics

---

# Common Mistakes

- Creating one topic for everything
- No DLQ configuration
- Ignoring failed deliveries
- Sending sensitive data unencrypted
- Missing subscription confirmation
- Not using filter policies
- Tight application coupling
- No monitoring
- Public topic permissions
- Using SNS instead of SQS where persistence is required

---

# Troubleshooting

## Email Not Received

Check

- Subscription Confirmation
- Spam Folder
- Topic ARN
- Delivery Status

---

## Lambda Not Triggered

Verify

- Lambda Permissions
- Subscription
- Topic Policy
- CloudWatch Logs

---

## Message Not Delivered

Check

- Subscriber Status
- Retry Policy
- DLQ
- IAM Permissions

---

## SQS Queue Not Receiving Messages

Verify

- Queue Policy
- SNS Subscription
- Region
- Topic ARN

---

## SMS Delivery Failed

Check

- SMS Preferences
- Destination Number
- Country Restrictions
- Spend Limit

---

# Interview Questions

## Basic

1. What is Amazon SNS?
2. What is Publish/Subscribe?
3. What is an SNS Topic?
4. Standard Topic vs FIFO Topic?
5. What are Subscribers?
6. What is a Subscription?
7. What is Message Filtering?
8. What is a DLQ?
9. SNS vs SQS?
10. SNS vs EventBridge?

---

## Intermediate

11. Explain Fan-Out architecture.
12. Explain Filter Policies.
13. Explain SNS encryption.
14. Explain Retry Policy.
15. Explain SNS with Lambda.
16. Explain SNS with SQS.
17. Explain SNS security.
18. Explain Topic Policies.
19. Explain FIFO Topics.
20. Explain monitoring.

---

## Advanced

21. Design a scalable notification system.
22. Explain enterprise fan-out architecture.
23. How would you notify multiple microservices?
24. Explain SNS security architecture.
25. Design SNS for disaster recovery.
26. Explain message filtering strategy.
27. Explain high availability.
28. Design cross-account notifications.
29. Explain SNS integration with EventBridge.
30. Best practices for production SNS deployments.

---

# Production Scenarios

### Scenario 1

A CloudWatch alarm detects CPU utilization above 90%.

How would SNS notify the operations team?

---

### Scenario 2

A customer places an order.

How would SNS distribute notifications to Billing, Inventory, Shipping, and Analytics services?

---

### Scenario 3

A backup job fails overnight.

How would SNS notify administrators immediately?

---

### Scenario 4

A compliance violation occurs in AWS Config.

How would SNS integrate into the notification workflow?

---

### Scenario 5

Your application sends millions of notifications daily.

How would SNS scale to support this workload?

---

### Scenario 6

Only finance-related events should reach the Finance application.

How would SNS Filter Policies implement this requirement?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Topic | Communication Channel |
| Publisher | Sends Messages |
| Subscriber | Receives Messages |
| Subscription | Connects Topic and Subscriber |
| Standard Topic | High Throughput |
| FIFO Topic | Ordered Delivery |
| Filter Policy | Selective Delivery |
| DLQ | Failed Message Storage |
| KMS | Encryption |
| Topic Policy | Access Control |

---

# Summary

Amazon SNS is a fully managed publish-subscribe messaging service that enables scalable, loosely coupled communication between applications, AWS services, and end users. Using Topics, Subscriptions, Message Filtering, FIFO Topics, Dead Letter Queues, and KMS encryption, SNS delivers reliable notifications to multiple subscribers simultaneously. When integrated with EventBridge, SQS, Lambda, CloudWatch, and AWS Config, SNS becomes a foundational service for building event-driven, highly available, and enterprise-grade cloud architectures.