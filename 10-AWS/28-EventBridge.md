# Amazon EventBridge

---

# Introduction

Amazon EventBridge is a fully managed serverless event bus service that enables applications to communicate using events. It allows AWS services, SaaS applications, and custom applications to exchange information asynchronously without being tightly coupled.

In traditional architectures, applications communicate directly with each other. As the number of services grows, managing these integrations becomes increasingly complex. EventBridge simplifies this by acting as a central event router.

Amazon EventBridge integrates with:

- AWS Lambda
- Amazon EC2
- Amazon ECS
- Amazon EKS
- Amazon S3
- Amazon SNS
- Amazon SQS
- AWS Step Functions
- AWS Batch
- AWS Systems Manager
- AWS Config
- AWS CloudTrail
- AWS Backup
- AWS CodePipeline
- AWS CodeBuild
- AWS Organizations
- SaaS Applications

EventBridge is one of the most important services for event-driven architectures in AWS.

---

# What is Amazon EventBridge?

Amazon EventBridge is an event bus.

Applications publish events.

Other applications consume those events.

Workflow

```text
Application

↓

EventBridge

↓

Target Services
```

The producer does not know who consumes the event.

---

# Why EventBridge?

Without EventBridge

```text
Application A

↓

Application B

↓

Application C

↓

Application D
```

Problems

- Tight Coupling
- Complex Integrations
- Difficult Maintenance
- Limited Scalability

With EventBridge

```text
Applications

↓

EventBridge

↓

Rules

↓

Targets
```

Applications become loosely coupled.

---

# Real World Problem Statement

An e-commerce application generates events such as

- Order Created
- Payment Completed
- Inventory Updated
- Shipment Created
- Customer Registered

Each event should trigger multiple downstream systems.

Instead of integrating every service individually,

EventBridge distributes events automatically.

---

# Enterprise Architecture

```text
Applications

EC2   ECS   Lambda   EKS   SaaS

           │

           ▼

     Amazon EventBridge

           │

      Event Rules

           │

 ┌─────────┼─────────┐

 │         │         │

Lambda    SNS      SQS

 │         │         │

StepFn  CloudWatch  API
```

---

# Core Components

Amazon EventBridge consists of

- Event Bus
- Events
- Event Sources
- Event Rules
- Targets
- Event Patterns
- Schedules
- Archives
- Replay
- Schema Registry

---

# Event

An Event is a JSON document representing something that happened.

Example

```text
Order Created

↓

Event Generated

↓

EventBridge
```

Events contain

- Source
- Detail-Type
- Time
- Region
- Account
- Payload

---

# Event Bus

An Event Bus receives events.

Types

- Default Event Bus
- Custom Event Bus
- Partner Event Bus

Every AWS account contains one Default Event Bus.

---

# Default Event Bus

Receives events from AWS services.

Example

```text
EC2

↓

Default Event Bus

↓

Rules
```

---

# Custom Event Bus

Applications publish custom events.

Example

```text
Inventory Application

↓

Custom Event Bus

↓

Consumers
```

Useful for microservices.

---

# Partner Event Bus

Receives events from SaaS providers.

Examples

- Datadog
- Zendesk
- Auth0
- PagerDuty
- Stripe

---

# Event Sources

Event sources include

- AWS Services
- Custom Applications
- SaaS Providers

Examples

- EC2
- S3
- CloudTrail
- Config
- Backup
- Lambda
- CodePipeline

---

# Event Structure

Typical Event

```text
Source

↓

Detail Type

↓

Timestamp

↓

Payload
```

Every event follows JSON format.

---

# Event Patterns

Rules filter events using patterns.

Example

```text
Source

↓

aws.ec2

↓

State Change

↓

Running
```

Only matching events trigger targets.

---

# Rules

Rules determine

- Which events match
- Which targets execute

Workflow

```text
Incoming Event

↓

Rule

↓

Target
```

---

# Rule Matching

Example

```text
EC2 State

↓

Stopped

↓

SNS Notification
```

Only matching events execute actions.

---

# Targets

Supported targets include

- Lambda
- SNS
- SQS
- Step Functions
- ECS Tasks
- Batch Jobs
- API Destination
- CloudWatch Logs
- Kinesis

One rule can invoke multiple targets.

---

# Fan-Out Architecture

Example

```text
Order Created

↓

EventBridge

↓

Lambda

↓

SNS

↓

SQS

↓

Analytics
```

Multiple consumers receive the same event.

---

# Scheduled Rules

EventBridge replaces traditional CloudWatch Events scheduling.

Example

```text
Every Day

↓

02:00 AM

↓

Lambda Backup
```

Supports

- Cron
- Rate Expressions

---

# Cron Expressions

Example

```text
cron(0 2 * * ? *)
```

Runs daily at 2 AM.

---

# Rate Expressions

Example

```text
rate(5 minutes)
```

Runs every five minutes.

---

# Event Archive

Archives store events.

Benefits

- Troubleshooting
- Compliance
- Replay

---

# Event Replay

Replay previously archived events.

Workflow

```text
Archive

↓

Replay

↓

EventBridge

↓

Applications
```

Useful for testing and recovery.

---

# Schema Registry

Stores event schemas.

Benefits

- Documentation
- Validation
- Code Generation

Supports multiple programming languages.

---

# Schema Discovery

Automatically detects event formats.

Useful for

- Developers
- Integrations
- API Documentation

---

# Event Filtering

Rules filter using

- Source
- Detail Type
- Resource ARN
- Account
- Region
- Payload

Reduces unnecessary processing.

---

# Event Delivery

EventBridge guarantees

- Durable Delivery
- At-Least-Once Delivery
- Automatic Retry

Failed deliveries can use Dead Letter Queues.

---

# Retry Policy

If target execution fails,

EventBridge retries automatically.

Example

```text
Target Failure

↓

Retry

↓

Success
```

---

# Dead Letter Queue (DLQ)

Failed events can be sent to Amazon SQS.

Workflow

```text
Target Failure

↓

Retry Failed

↓

Dead Letter Queue
```

Prevents event loss.

---

# EventBridge Pipes

Pipes connect event producers directly to consumers.

Supports

- Filtering
- Enrichment
- Transformation

Architecture

```text
Source

↓

Pipe

↓

Filter

↓

Transform

↓

Target
```

---

# EventBridge Scheduler

Dedicated scheduling service.

Can invoke

- Lambda
- ECS
- Step Functions
- API Calls
- EventBridge

More scalable than traditional cron jobs.

---

# Cross-Account Events

Events can be shared across AWS accounts.

Architecture

```text
Account A

↓

EventBridge

↓

Account B

↓

Lambda
```

Useful for enterprise environments.

---

# Cross-Region Events

Events can be routed across Regions.

Supports disaster recovery and global architectures.

---

# AWS Service Integrations

Common integrations

- AWS Config
- CloudTrail
- Backup
- Security Hub
- GuardDuty
- Lambda
- ECS
- Step Functions
- SNS
- SQS

---

# AWS CLI

Create Rule

```bash
aws events put-rule \
--name ec2-state-rule
```

List Rules

```bash
aws events list-rules
```

Put Target

```bash
aws events put-targets \
--rule ec2-state-rule
```

---

# Terraform

```hcl
resource "aws_cloudwatch_event_rule" "ec2" {

  name = "ec2-state-change"

}
```

---

# CloudFormation

```yaml
Resources:

  EventRule:

    Type: AWS::Events::Rule

    Properties:

      Name: ec2-state-change
```

---

# Python (Boto3)

```python
import boto3

events = boto3.client("events")

response = events.list_rules()

print(response)
```

---

# Enterprise Production Architecture

```text
             AWS Services & Applications

 EC2  ECS  Lambda  Config  Backup  CloudTrail

                    │

             Amazon EventBridge

                    │

             Event Filtering Rules

       ┌────────────┼────────────┐

       │            │            │

    Lambda         SNS          SQS

       │            │            │

 Step Functions  CloudWatch   API Gateway

                    │

             Operations Team
```

---

# Best Practices

- Use custom event buses for applications
- Filter events using precise event patterns
- Configure Dead Letter Queues
- Enable event archives
- Use replay for testing
- Use least-privilege IAM policies
- Separate production and development event buses
- Monitor failed event deliveries
- Use EventBridge Scheduler instead of cron servers
- Document schemas using Schema Registry

---

# Common Mistakes

- Sending every event to every target
- No filtering rules
- Ignoring failed deliveries
- Not using DLQs
- Hardcoding integrations
- Using polling instead of events
- No archive configuration
- Large event payloads
- Missing IAM permissions
- Not monitoring event failures

---

# Troubleshooting

## Rule Not Triggering

Check

- Event Pattern
- Rule Enabled
- Event Bus
- Region

---

## Target Not Invoked

Verify

- IAM Permissions
- Target ARN
- Retry Policy
- DLQ Configuration

---

## Event Missing

Check

- Event Source
- Event Bus
- Rule Pattern
- Archive

---

## Cross-Account Event Failed

Verify

- Resource Policy
- IAM Role
- Event Bus Permissions

---

## Scheduler Failed

Check

- Cron Expression
- Target Permissions
- Service Availability

---

# Interview Questions

## Basic

1. What is Amazon EventBridge?
2. What is an Event Bus?
3. Default Event Bus vs Custom Event Bus?
4. What is an Event?
5. What is an Event Rule?
6. What is an Event Pattern?
7. What is a Target?
8. What is Event Replay?
9. What is Schema Registry?
10. What is EventBridge Scheduler?

---

## Intermediate

11. EventBridge vs SNS?
12. EventBridge vs SQS?
13. Explain EventBridge Pipes.
14. Explain Dead Letter Queues.
15. Explain Retry Policies.
16. Explain Event Archive.
17. Explain Cross-Account Events.
18. Explain Scheduled Rules.
19. Explain Event Filtering.
20. Explain EventBridge architecture.

---

## Advanced

21. Design an event-driven architecture.
22. How would you integrate EventBridge with microservices?
23. Explain EventBridge replay.
24. Design enterprise event routing.
25. Explain EventBridge governance.
26. How would you monitor failed events?
27. Explain EventBridge security.
28. Design multi-account event routing.
29. Explain EventBridge Scheduler.
30. Best practices for EventBridge deployments.

---

# Production Scenarios

### Scenario 1

An EC2 instance stops unexpectedly.

How would EventBridge automatically notify the operations team?

---

### Scenario 2

A new object is uploaded to Amazon S3.

How would EventBridge trigger a Lambda function for image processing?

---

### Scenario 3

A compliance violation is detected by AWS Config.

How would EventBridge automate remediation?

---

### Scenario 4

A backup job fails.

How would EventBridge notify administrators immediately?

---

### Scenario 5

Your application generates millions of business events daily.

How would EventBridge distribute events to multiple consumers?

---

### Scenario 6

You need to replay production events in a testing environment.

Which EventBridge features would you use?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Event Bus | Receives Events |
| Event | JSON Payload |
| Event Rule | Filters Events |
| Event Pattern | Matching Criteria |
| Target | Event Destination |
| Archive | Store Events |
| Replay | Reprocess Events |
| Scheduler | Scheduled Execution |
| Pipes | Connect Sources and Targets |
| Schema Registry | Event Documentation |
| DLQ | Store Failed Events |

---

# Summary

Amazon EventBridge is a fully managed event routing service that enables event-driven architectures across AWS services, custom applications, and SaaS providers. By using Event Buses, Rules, Event Patterns, Targets, Archives, Replay, Scheduler, Pipes, and Schema Registry, organizations can build loosely coupled, scalable, and highly reliable systems. EventBridge is a core building block for modern cloud-native applications, automation workflows, and enterprise integration architectures.