# AWS CloudWatch

---

# Introduction

Amazon CloudWatch is a fully managed monitoring, observability, logging, and operational intelligence service that collects and monitors metrics, logs, events, and traces from AWS resources, applications, and on-premises environments.

Modern cloud applications require continuous monitoring to maintain availability, performance, reliability, and operational excellence. Amazon CloudWatch provides real-time visibility into infrastructure, applications, and AWS services by collecting metrics, logs, alarms, dashboards, and events.

CloudWatch integrates with:

- Amazon EC2
- Amazon EKS
- Amazon ECS
- AWS Lambda
- Amazon RDS
- Elastic Load Balancer (ALB/NLB)
- Amazon S3
- AWS CloudTrail
- AWS Config
- Amazon EventBridge
- AWS Systems Manager
- AWS X-Ray
- Amazon SNS

CloudWatch is one of the most widely used AWS services for monitoring production environments.

---

# What is Amazon CloudWatch?

Amazon CloudWatch continuously monitors AWS resources and applications.

It collects

- Metrics
- Logs
- Events
- Alarms
- Dashboards

Workflow

```text
AWS Resources

↓

CloudWatch

↓

Metrics & Logs

↓

Alarms

↓

Notifications
```

---

# Why CloudWatch?

Without CloudWatch

```text
Application

↓

Failure

↓

Users Report Issue

↓

Investigation
```

Problems

- No Visibility
- Slow Incident Detection
- Manual Troubleshooting
- No Alerting
- Poor Performance Monitoring

With CloudWatch

```text
Application

↓

CloudWatch

↓

Alert

↓

Immediate Response
```

---

# Real World Problem Statement

A company manages

- 1,500 EC2 Instances
- 250 RDS Databases
- Hundreds of Lambda Functions
- Kubernetes Clusters
- Application Load Balancers

Requirements

- Infrastructure Monitoring
- Application Monitoring
- Log Management
- Automated Alerts
- Dashboards

CloudWatch provides centralized observability.

---

# Enterprise Architecture

```text
EC2

Lambda

RDS

EKS

ALB

      │

      ▼

Amazon CloudWatch

      │

Metrics Logs Events

      │

 ┌────┼─────────┐

 │    │         │

Alarms Dashboards SNS
```

---

# Core Components

Amazon CloudWatch consists of

- Metrics
- Namespaces
- Dimensions
- Alarms
- Dashboards
- Logs
- Log Groups
- Log Streams
- Events
- Metric Filters

---

# Metrics

Metrics measure resource performance over time.

Examples

- CPU Utilization
- Memory Usage
- Network Traffic
- Disk Activity
- Request Count

Metrics are stored as time-series data.

---

# Standard Metrics

AWS automatically publishes metrics for many services.

Examples

EC2

- CPUUtilization
- NetworkIn
- NetworkOut
- DiskReadOps

RDS

- CPUUtilization
- FreeStorageSpace
- DatabaseConnections

Lambda

- Invocations
- Errors
- Duration
- Throttles

---

# Custom Metrics

Applications can publish their own metrics.

Examples

- Orders Processed
- Active Users
- Login Failures
- Transactions Per Minute

Useful for business monitoring.

---

# Namespace

Metrics are organized into namespaces.

Examples

```text
AWS/EC2

AWS/RDS

AWS/Lambda

Custom/Application
```

---

# Dimensions

Dimensions identify a specific resource.

Example

```text
Metric

↓

CPUUtilization

↓

InstanceId=i-123456
```

One metric can have multiple dimensions.

---

# Metric Resolution

CloudWatch supports

- Standard Resolution (1 Minute)
- High Resolution (1 Second)

High-resolution metrics enable faster alerting.

---

# Statistics

CloudWatch calculates

- Average
- Sum
- Minimum
- Maximum
- Sample Count

Useful for analysis and dashboards.

---

# CloudWatch Alarms

Alarms monitor metrics and perform actions when thresholds are crossed.

Workflow

```text
Metric

↓

Threshold

↓

Alarm

↓

SNS Notification
```

---

# Alarm States

Possible states

- OK
- ALARM
- INSUFFICIENT_DATA

State changes trigger actions.

---

# Alarm Actions

CloudWatch Alarms can

- Send SNS Notifications
- Stop EC2 Instances
- Terminate EC2 Instances
- Recover EC2 Instances
- Trigger Auto Scaling
- Invoke Lambda Functions

---

# Composite Alarms

Composite Alarms combine multiple alarms.

Example

```text
High CPU

AND

High Memory

↓

Critical Alarm
```

Reduces unnecessary alerts.

---

# Dashboards

Dashboards provide visual monitoring.

Widgets include

- Line Charts
- Number Widgets
- Bar Charts
- Pie Charts
- Text Widgets

Supports multiple AWS Regions.

---

# Cross-Region Dashboards

Monitor resources from multiple Regions.

Architecture

```text
Mumbai

Virginia

Singapore

↓

CloudWatch Dashboard
```

---

# Summary

Amazon CloudWatch is a fully managed monitoring and observability service that collects metrics, logs, events, and operational data from AWS resources and applications. Features such as Metrics, Namespaces, Dimensions, Alarms, Dashboards, Composite Alarms, and Custom Metrics enable organizations to monitor infrastructure health, improve application reliability, and automate operational responses across enterprise AWS environments.

---

