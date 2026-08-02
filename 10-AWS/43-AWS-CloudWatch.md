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

# CloudWatch Logs

Amazon CloudWatch Logs collects, stores, monitors, and analyzes log data from AWS resources and applications.

Sources include

- Amazon EC2
- AWS Lambda
- Amazon ECS
- Amazon EKS
- Amazon API Gateway
- AWS CloudTrail
- Custom Applications

Benefits

- Centralized Logging
- Real-Time Monitoring
- Troubleshooting
- Compliance

---

# Log Groups

A Log Group is a container for log streams.

Examples

```text
/application/web

/aws/lambda/payment-service

/aws/eks/cluster

/aws/ec2/system
```

Retention policies are configured at the Log Group level.

---

# Log Streams

A Log Stream contains log events from a single source.

Example

```text
Log Group

↓

/application/web

↓

Log Stream

↓

i-0123456789abcdef
```

Each EC2 instance or Lambda function typically creates its own log stream.

---

# Log Events

Each log event contains

- Timestamp
- Log Message
- Source

Example

```text
2026-08-02 10:15:32

Application Started
```

---

# CloudWatch Agent

The CloudWatch Agent collects

- Memory Usage
- Disk Usage
- Swap Usage
- Disk I/O
- Application Logs
- Custom Metrics

Unlike default EC2 metrics, the agent provides operating system–level visibility.

---

# CloudWatch Logs Insights

CloudWatch Logs Insights allows SQL-like querying of log data.

Example Query

```sql
fields @timestamp, @message

| filter @message like /ERROR/

| sort @timestamp desc

| limit 20
```

Useful for troubleshooting application issues.

---

# Metric Filters

Metric Filters convert log patterns into CloudWatch metrics.

Example

```text
ERROR

↓

Metric Filter

↓

ErrorCount Metric

↓

Alarm
```

Useful for monitoring application errors.

---

# Contributor Insights

Contributor Insights analyzes high-cardinality log data.

Provides

- Top Contributors
- Traffic Sources
- API Consumers
- Error Sources

Useful for identifying hotspots.

---

# CloudWatch Synthetics

CloudWatch Synthetics creates canaries that simulate user behavior.

Example

```text
Synthetic User

↓

Open Website

↓

Login

↓

Checkout

↓

Success / Failure
```

Supports proactive monitoring.

---

# Canary

A Canary performs scheduled tests.

Examples

- Website Availability
- API Health
- Login Workflow
- Payment Flow

Helps detect issues before users experience them.

---

# Application Insights

CloudWatch Application Insights automatically detects application problems.

Supports

- EC2 Applications
- .NET Applications
- Java Applications
- SQL Server
- SAP Workloads

Provides recommended remediation actions.

---

# Anomaly Detection

CloudWatch uses machine learning to detect unusual metric behavior.

Workflow

```text
Historical Metrics

↓

Machine Learning

↓

Expected Range

↓

Anomaly Alert
```

No static threshold is required.

---

# EventBridge Integration

CloudWatch Alarms generate EventBridge events.

Example

```text
CPU > 90%

↓

Alarm

↓

EventBridge

↓

Lambda

↓

Scale EC2
```

---

# Auto Scaling Integration

CloudWatch Alarms can trigger Auto Scaling policies.

Example

```text
CPU > 70%

↓

Scale Out

------------

CPU < 30%

↓

Scale In
```

---

# Amazon SNS Integration

CloudWatch Alarms notify administrators using SNS.

Notification methods

- Email
- SMS
- HTTP Endpoint
- Lambda

---

# CloudTrail Integration

CloudTrail logs can be streamed to CloudWatch Logs for real-time monitoring.

Useful for

- Security Monitoring
- IAM Activity
- API Tracking

---

# AWS CLI

List Metrics

```bash
aws cloudwatch list-metrics
```

Get Metric Statistics

```bash
aws cloudwatch get-metric-statistics
```

Create Alarm

```bash
aws cloudwatch put-metric-alarm \
--alarm-name HighCPU
```

Describe Alarms

```bash
aws cloudwatch describe-alarms
```

---

# Terraform

```hcl
resource "aws_cloudwatch_metric_alarm" "high_cpu" {

  alarm_name          = "HighCPU"

  comparison_operator = "GreaterThanThreshold"

  threshold           = 80

  evaluation_periods  = 2

  metric_name         = "CPUUtilization"

  namespace           = "AWS/EC2"

  statistic           = "Average"

  period              = 300

}
```

---

# CloudFormation

```yaml
Resources:

  HighCPUAlarm:

    Type: AWS::CloudWatch::Alarm

    Properties:

      AlarmName: HighCPU

      MetricName: CPUUtilization

      Namespace: AWS/EC2

      Threshold: 80
```

---

# Python (Boto3)

```python
import boto3

cloudwatch = boto3.client("cloudwatch")

response = cloudwatch.describe_alarms()

print(response)
```

Put Custom Metric

```python
cloudwatch.put_metric_data(

    Namespace="Application",

    MetricData=[

        {

            "MetricName":"OrdersProcessed",

            "Value":100

        }

    ]

)
```

---

# Enterprise Production Architecture

```text
 EC2  Lambda  RDS  EKS  ECS

             │

             ▼

      Amazon CloudWatch

             │

 Metrics  Logs  Events

             │

 ┌─────────┼────────────┐

 │         │            │

Alarms Dashboards Logs Insights

 │         │            │

SNS  EventBridge Auto Scaling

             │

      Operations Team
```

---

# Best Practices

- Enable CloudWatch Agent on EC2 instances
- Use custom metrics for business KPIs
- Configure meaningful alarm thresholds
- Enable log retention policies
- Use CloudWatch Logs Insights for troubleshooting
- Enable anomaly detection where appropriate
- Integrate alarms with SNS and EventBridge
- Monitor Auto Scaling metrics
- Create centralized dashboards
- Archive logs when required
- Monitor application and infrastructure metrics together
- Review alarms periodically to eliminate noise

---

# Common Mistakes

- Monitoring only CPU utilization
- Ignoring memory and disk metrics
- Using incorrect alarm thresholds
- No log retention policy
- Excessive alarm notifications
- Not monitoring application logs
- Ignoring failed Lambda invocations
- No dashboards
- No anomaly detection
- Missing CloudWatch Agent

---

# Troubleshooting

## Alarm Not Triggering

Check

- Metric Namespace
- Alarm Threshold
- Evaluation Period
- Metric Availability

---

## Logs Not Appearing

Verify

- CloudWatch Agent
- IAM Role
- Log Group
- Network Connectivity

---

## Custom Metric Missing

Check

- Namespace
- Metric Name
- PutMetricData API
- IAM Permissions

---

## Dashboard Not Updating

Verify

- Widget Configuration
- Region
- Metric Source
- Refresh Interval

---

## CloudWatch Agent Not Sending Metrics

Check

- Agent Status
- Configuration File
- IAM Permissions
- Network Access

---

# Interview Questions

## Basic

1. What is Amazon CloudWatch?
2. What are Metrics?
3. What is a Namespace?
4. What are Dimensions?
5. What is a CloudWatch Alarm?
6. What is a Dashboard?
7. What are Log Groups?
8. What are Log Streams?
9. What is CloudWatch Logs?
10. What is the CloudWatch Agent?

---

## Intermediate

11. Explain Metric Filters.
12. Explain Logs Insights.
13. Explain Contributor Insights.
14. Explain CloudWatch Synthetics.
15. Explain Application Insights.
16. Explain Anomaly Detection.
17. Explain EventBridge integration.
18. Explain Auto Scaling integration.
19. Explain CloudTrail integration.
20. Explain custom metrics.

---

## Advanced

21. Design enterprise monitoring architecture.
22. How would you monitor a microservices application?
23. Design centralized logging for multiple AWS accounts.
24. Explain CloudWatch vs Prometheus.
25. Explain CloudWatch vs CloudTrail.
26. Design proactive monitoring using canaries.
27. Explain observability best practices.
28. Design automated remediation using CloudWatch.
29. Explain CloudWatch operational best practices.
30. Best practices for enterprise CloudWatch deployments.

---

# Production Scenarios

### Scenario 1

CPU utilization exceeds 90% on production EC2 instances.

How would CloudWatch automatically scale the environment?

---

### Scenario 2

A payment application generates repeated ERROR logs.

How would Metric Filters and Alarms notify the operations team?

---

### Scenario 3

Your organization wants to monitor application memory usage.

How would the CloudWatch Agent provide this capability?

---

### Scenario 4

A company needs to verify that its website is available every five minutes.

How would CloudWatch Synthetics accomplish this?

---

### Scenario 5

Security teams require alerts whenever CloudTrail records failed console logins.

How would CloudWatch and EventBridge automate this workflow?

---

### Scenario 6

Operations teams need a centralized dashboard displaying EC2, RDS, Lambda, and EKS metrics across Regions.

Which CloudWatch features provide this functionality?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Metrics | Performance Data |
| Namespace | Metric Group |
| Dimensions | Resource Identifier |
| Alarm | Threshold Monitoring |
| Dashboard | Visual Monitoring |
| Log Group | Container for Logs |
| Log Stream | Logs from One Source |
| Logs Insights | Log Query Engine |
| Metric Filter | Convert Logs to Metrics |
| CloudWatch Agent | OS Metrics & Logs |
| Synthetics | Synthetic Monitoring |
| Application Insights | Application Health Analysis |

---

# Summary

Amazon CloudWatch is a fully managed monitoring and observability service that collects metrics, logs, events, and operational data from AWS resources and applications. Features such as CloudWatch Logs, Log Groups, Log Streams, Metric Filters, CloudWatch Agent, Logs Insights, Contributor Insights, Synthetics, Application Insights, Dashboards, Alarms, EventBridge integration, and Auto Scaling integration enable organizations to build comprehensive monitoring, proactive alerting, automated remediation, and operational visibility across enterprise AWS environments.