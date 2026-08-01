# AWS Lambda

---

# Introduction

AWS Lambda is a fully managed serverless compute service that allows you to run code without provisioning or managing servers.

Instead of launching EC2 instances or managing Kubernetes clusters, you simply upload your code, configure a trigger, and AWS automatically executes your application.

AWS Lambda automatically handles:

- Infrastructure Provisioning
- Server Management
- Operating System Patching
- High Availability
- Auto Scaling
- Fault Tolerance

Developers only focus on writing business logic.

Lambda is one of the core building blocks of modern serverless architectures.

---

# What is AWS Lambda?

AWS Lambda is an event-driven serverless compute service.

Code runs only when an event occurs.

Examples of events

- API Request
- S3 File Upload
- DynamoDB Update
- SQS Message
- SNS Notification
- EventBridge Schedule
- CloudWatch Alarm

Workflow

```text
Event

↓

AWS Lambda

↓

Execute Function

↓

Return Response
```

---

# Why Lambda?

Traditional Approach

```text
Users

↓

EC2

↓

Application

↓

Always Running
```

Problems

- Pay even when idle
- Server management
- Scaling complexity
- Maintenance

Lambda Approach

```text
Event

↓

Lambda Function

↓

Execute

↓

Stop
```

Advantages

- No Servers
- Automatic Scaling
- Pay Per Execution
- Highly Available

---

# Real World Problem

An e-commerce company needs to:

- Resize uploaded images
- Send order confirmation emails
- Process payments
- Generate invoices
- Process logs

Running EC2 instances 24/7 is expensive.

AWS Lambda executes code only when required.

---

# Lambda Architecture

```text
User

↓

API Gateway

↓

AWS Lambda

↓

Amazon RDS

↓

Response
```

Another Example

```text
Amazon S3

↓

Image Upload

↓

Lambda

↓

Image Resize

↓

Store Thumbnail
```

---

# Core Components

AWS Lambda consists of:

- Function
- Event Source
- Trigger
- Runtime
- Execution Environment
- IAM Role
- Layers
- Environment Variables
- Destination
- Dead Letter Queue

---

# Lambda Function

A Lambda Function is the application code.

Supported Languages

- Python
- Node.js
- Java
- Go
- .NET
- Ruby
- Custom Runtime

---

# Runtime

The Runtime provides the execution environment.

Example

```text
Python Function

↓

Python Runtime

↓

Execution
```

AWS manages runtime updates.

---

# Event Sources

Lambda supports hundreds of event sources.

Common AWS Services

- API Gateway
- S3
- SNS
- SQS
- EventBridge
- DynamoDB Streams
- Kinesis
- CloudWatch
- ECR
- CodeCommit

---

# Invocation Types

Three invocation models exist.

### Synchronous

```text
Client

↓

Lambda

↓

Immediate Response
```

Example

API Gateway

---

### Asynchronous

```text
Event

↓

Lambda

↓

Response Later
```

Examples

- SNS
- EventBridge
- S3

---

### Poll-Based

Lambda polls services continuously.

Examples

- Amazon SQS
- Kinesis
- DynamoDB Streams

---

# Execution Environment

Execution Flow

```text
Event

↓

Initialize Runtime

↓

Load Function

↓

Execute Code

↓

Return Response
```

AWS may reuse the execution environment for future requests.

---

# Cold Start

A Cold Start occurs when AWS creates a new execution environment.

Workflow

```text
Request

↓

Create Environment

↓

Initialize Runtime

↓

Execute Function
```

Cold starts increase latency.

---

# Warm Start

If Lambda reuses an existing environment,

execution is much faster.

```text
Request

↓

Existing Environment

↓

Execute
```

---

# Memory Configuration

Lambda supports configurable memory.

Example

- 128 MB
- 512 MB
- 1024 MB
- 2048 MB
- 4096 MB
- 10240 MB

Increasing memory also increases available CPU.

---

# Timeout

Maximum execution time

```
15 Minutes
```

Functions exceeding the timeout are terminated.

---

# Environment Variables

Used for configuration.

Examples

- Database Endpoint
- API URL
- Bucket Name
- Region

Avoid storing passwords.

Use Secrets Manager instead.

---

# Lambda Layers

Layers share common code.

Example

```text
Layer

↓

Python Libraries

↓

Multiple Functions
```

Benefits

- Code Reuse
- Smaller Packages
- Easier Maintenance

---

# IAM Execution Role

Every Lambda Function requires an IAM Role.

Example

```text
Lambda

↓

IAM Role

↓

Amazon S3
```

Use least-privilege permissions.

---

# VPC Integration

Lambda can run inside a VPC.

Useful for accessing

- Amazon RDS
- ElastiCache
- Internal APIs

Architecture

```text
Lambda

↓

Private Subnet

↓

Amazon RDS
```

---

# Concurrency

Concurrency defines how many Lambda executions run simultaneously.

Example

```text
100 Requests

↓

100 Concurrent Executions
```

AWS automatically scales concurrency.

---

# Reserved Concurrency

Guarantees capacity for critical functions.

Example

```text
Payment Function

↓

Reserved

200 Concurrent Executions
```

---

# Provisioned Concurrency

Keeps execution environments warm.

Benefits

- Eliminates Cold Starts
- Predictable Latency

Useful for APIs.

---

# Monitoring

Lambda integrates with

- CloudWatch Logs
- CloudWatch Metrics
- AWS X-Ray

Common Metrics

- Invocations
- Duration
- Errors
- Throttles
- Concurrent Executions

---

# Error Handling

Lambda supports

- Automatic Retries
- Dead Letter Queue
- Destinations

Dead Letter Queue

```text
Failed Event

↓

SQS

↓

Investigation
```

---

# AWS CLI

Create Function

```bash
aws lambda create-function \
--function-name image-resizer
```

List Functions

```bash
aws lambda list-functions
```

Invoke Function

```bash
aws lambda invoke \
--function-name image-resizer \
response.json
```

---

# Terraform

```hcl
resource "aws_lambda_function" "resize" {

  function_name = "image-resizer"

  runtime = "python3.12"

  handler = "lambda_function.lambda_handler"

  role = aws_iam_role.lambda.arn

}
```

---

# Best Practices

- Keep functions small
- Follow single responsibility
- Use IAM least privilege
- Store secrets in Secrets Manager
- Enable CloudWatch Logs
- Configure appropriate timeout
- Optimize memory
- Use Layers for shared libraries
- Enable Provisioned Concurrency for latency-sensitive APIs
- Keep deployment packages small

---

# Common Mistakes

- Hardcoding credentials
- Large deployment packages
- Ignoring cold starts
- Excessive timeout values
- No retry strategy
- Missing DLQ
- Running long-running jobs in Lambda

---

# Troubleshooting

## Function Timeout

Check

- External API latency
- Database queries
- Timeout configuration

---

## Access Denied

Verify

- IAM Execution Role
- Resource Policy

---

## High Latency

Check

- Cold Starts
- VPC Configuration
- Provisioned Concurrency

---

## Throttling

Verify

- Concurrent Executions
- Reserved Concurrency
- Account Limits

---

# Interview Questions

1. What is AWS Lambda?
2. What is serverless computing?
3. Explain cold start.
4. Explain warm start.
5. What are Lambda Layers?
6. What is Provisioned Concurrency?
7. What is Reserved Concurrency?
8. Explain Lambda execution lifecycle.
9. How does Lambda integrate with API Gateway?
10. Explain VPC-enabled Lambda.
11. What is a Dead Letter Queue?
12. How do you monitor Lambda?
13. Lambda vs EC2?
14. Lambda vs ECS?
15. Lambda vs EKS?

---

# Scenario Questions

### Scenario 1

Your API has high latency only for the first request after inactivity.

How would you solve it?

---

### Scenario 2

A Lambda function cannot connect to Amazon RDS in a private subnet.

What would you check?

---

### Scenario 3

Thousands of SQS messages arrive simultaneously.

How does Lambda scale?

---

### Scenario 4

A function repeatedly fails while processing events.

How would you prevent data loss?

---

### Scenario 5

An image-processing Lambda has become expensive.

How would you optimize cost and performance?

---

# Summary

AWS Lambda is a fully managed serverless compute service that executes code in response to events without requiring server management. By integrating with API Gateway, S3, SQS, SNS, EventBridge, CloudWatch, IAM, and VPCs, Lambda enables developers to build scalable, event-driven applications while paying only for actual execution time. For production workloads, use least-privilege IAM roles, Secrets Manager, CloudWatch monitoring, appropriate concurrency settings, and dead-letter queues to build secure and resilient serverless applications.