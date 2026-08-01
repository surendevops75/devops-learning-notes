# Amazon CloudWatch

---

# Introduction

Amazon CloudWatch is AWS's fully managed monitoring and observability service that helps you monitor AWS resources, applications, infrastructure, and services in real time.

CloudWatch collects metrics, logs, events, and traces from AWS resources and applications, enabling engineers to monitor system health, detect anomalies, troubleshoot issues, automate responses, and maintain high availability.

CloudWatch is one of the most important services in AWS because almost every production workload relies on monitoring and alerting.

---

# What is Amazon CloudWatch?

Amazon CloudWatch is a centralized monitoring service.

It continuously collects operational data from AWS resources and applications.

CloudWatch can monitor:

- EC2 Instances
- Elastic Load Balancers
- Auto Scaling Groups
- EBS Volumes
- RDS Databases
- Lambda Functions
- ECS
- EKS
- S3
- CloudFront
- API Gateway
- Custom Applications

CloudWatch provides visibility into resource utilization, application performance, system health, and operational events.

---

# Why Do We Need CloudWatch?

Imagine a production application.

```text
Users

↓

Application

↓

EC2
```

One day:

- CPU reaches 100%
- Memory becomes full
- Disk space runs out
- Application crashes

Without monitoring:

Administrators discover the issue only after customers complain.

With CloudWatch:

```text
CPU > 80%

↓

Alarm

↓

SNS Notification

↓

Auto Scaling

↓

Launch New Instance
```

Problems are detected before they affect users.

---

# Real-World Problem

A banking application receives millions of transactions every day.

Requirements:

- Monitor infrastructure
- Detect failures immediately
- Notify engineers
- Automatically scale
- Collect logs
- Analyze performance
- Maintain compliance

Amazon CloudWatch provides these capabilities from a single monitoring platform.

---

# Enterprise Architecture

```text
                      AWS Resources

      EC2   RDS   ALB   Lambda   EKS   ECS

                 │    │    │    │    │

                 └────┼────┼────┼────┘

                      Amazon CloudWatch

          ┌──────────────┼──────────────┐

          │              │              │

       Metrics         Logs         Dashboards

          │              │              │

          └──────────────┼──────────────┘

                  Alarms & EventBridge

                         │

                        SNS

                         │

                 Email / Slack / Lambda
```

---

# Internal Working

CloudWatch continuously collects monitoring data.

Workflow

```text
AWS Resource

↓

Metric Collection

↓

CloudWatch

↓

Store Metrics

↓

Alarm Evaluation

↓

SNS / Auto Scaling

↓

Administrator
```

This process repeats continuously.

---

# Core Components

Amazon CloudWatch consists of:

- Metrics
- Namespaces
- Dimensions
- Alarms
- Dashboards
- Logs
- Log Groups
- Log Streams
- Metric Filters
- Logs Insights
- Events (EventBridge)
- CloudWatch Agent

---

# Metrics

Metrics are numerical values representing the performance of AWS resources over time.

Examples:

- CPU Utilization
- Network In
- Network Out
- Disk Read Operations
- Disk Write Operations
- Request Count
- Latency

CloudWatch automatically stores metrics in time-series format.

---

# Metric Example

```text
Time

↓

10:00 → CPU 22%

10:05 → CPU 35%

10:10 → CPU 81%

10:15 → CPU 91%
```

CloudWatch visualizes these values as graphs.

---

# Namespaces

A Namespace groups related metrics.

AWS Services provide predefined namespaces.

Examples

| Service | Namespace |
|----------|-----------|
| EC2 | AWS/EC2 |
| ELB | AWS/ApplicationELB |
| Auto Scaling | AWS/AutoScaling |
| Lambda | AWS/Lambda |
| RDS | AWS/RDS |
| EBS | AWS/EBS |

Custom applications can create their own namespaces.

Example

```
Company/Application
```

---

# Dimensions

Dimensions provide additional information about a metric.

Example

Metric

```
CPUUtilization
```

Dimension

```
InstanceId

↓

i-0123456789
```

One metric can have multiple dimensions.

Examples:

- InstanceId
- AutoScalingGroupName
- LoadBalancer
- TargetGroup
- AvailabilityZone

---

# Standard Monitoring

EC2 provides Standard Monitoring by default.

Characteristics:

- 5-minute interval
- Lower cost
- Basic visibility

Example

```
CPU

↓

Every

5 Minutes
```

---

# Detailed Monitoring

Detailed Monitoring provides metrics every minute.

Characteristics:

- 1-minute interval
- Faster alarm response
- Better visibility

Recommended for production environments.

---

# Custom Metrics

CloudWatch can store application-specific metrics.

Examples:

- Active Users
- Orders Processed
- Login Failures
- Queue Length
- Business Transactions
- Memory Utilization

Example

```text
Application

↓

Custom Metric

↓

CloudWatch

↓

Dashboard
```

---

# CloudWatch Agent

The CloudWatch Agent collects operating system metrics that AWS does not collect by default.

Examples

- Memory Usage
- Disk Usage
- Swap Usage
- Running Processes
- File System Utilization

Architecture

```text
Linux Server

↓

CloudWatch Agent

↓

CloudWatch
```

Without the agent, EC2 does not publish memory utilization metrics.

---

# Installing CloudWatch Agent

Amazon Linux

```bash
sudo yum install amazon-cloudwatch-agent -y
```

Ubuntu

```bash
sudo apt install amazon-cloudwatch-agent
```

Start Agent

```bash
sudo systemctl start amazon-cloudwatch-agent
```

Enable Agent

```bash
sudo systemctl enable amazon-cloudwatch-agent
```

---

# CloudWatch Alarms

CloudWatch Alarms monitor metrics and perform actions when thresholds are crossed.

Example

```text
CPU > 80%

↓

Alarm

↓

SNS

↓

Email

↓

Administrator
```

Alarms can also trigger:

- Auto Scaling
- Lambda
- Systems Manager
- EventBridge

---

# Alarm States

CloudWatch Alarms have three possible states.

| State | Meaning |
|--------|----------|
| OK | Metric within threshold |
| ALARM | Threshold exceeded |
| INSUFFICIENT_DATA | Not enough data available |

Workflow

```text
CPU = 35%

↓

OK

----------------

CPU = 90%

↓

ALARM

----------------

No Metrics

↓

INSUFFICIENT_DATA
```

---

# Alarm Evaluation

Example

Threshold

```
CPU > 80%
```

Evaluation

```text
78%

↓

No Alarm

----------------

82%

↓

Alarm

----------------

45%

↓

Alarm Cleared
```

---

# Composite Alarms

Composite Alarms combine multiple alarms into a single alarm.

Example

```text
CPU Alarm

+

Memory Alarm

↓

Composite Alarm

↓

SNS Notification
```

Benefits

- Reduced alert noise
- Better incident management
- Fewer false positives

---

# Dashboards

CloudWatch Dashboards provide a centralized monitoring view.

Example Dashboard

```text
CPU Utilization

Network Traffic

Memory Usage

Request Count

Error Rate

Latency
```

Operations teams monitor dashboards continuously.

---

# Dashboard Widgets

Supported widgets include:

- Line Graph
- Number
- Stacked Area
- Bar Chart
- Pie Chart
- Text Widget

---

# CloudWatch Logs

CloudWatch Logs stores application and system logs.

Sources include:

- EC2
- Lambda
- ECS
- EKS
- API Gateway
- CloudTrail
- Custom Applications

Example

```text
Application

↓

Log File

↓

CloudWatch Logs

↓

Search
```

---

# Log Groups

A Log Group is a collection of related log streams.

Example

```
/aws/ec2/application

/aws/lambda/payment

/aws/eks/cluster

/application/logs
```

Retention policies can be configured for each Log Group.

---

# Log Streams

Each Log Group contains one or more Log Streams.

Example

```text
Log Group

↓

EC2-1

↓

Log Stream

----------------

EC2-2

↓

Log Stream

----------------

EC2-3

↓

Log Stream
```

This makes it easier to isolate logs from individual resources.

---

# Log Retention

CloudWatch allows configurable retention periods.

Examples

- 1 Day
- 7 Days
- 30 Days
- 90 Days
- 1 Year
- 3 Years
- Never Expire

Production recommendation:

Set retention policies according to compliance and business requirements instead of keeping logs indefinitely.

---