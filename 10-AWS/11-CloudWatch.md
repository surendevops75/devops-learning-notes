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

# Metric Filters

Metric Filters convert log data into CloudWatch metrics.

Instead of monitoring only infrastructure metrics, CloudWatch can monitor specific log patterns.

Example

Application Log

```text
ERROR Database Connection Failed
```

Metric Filter

```text
Pattern

↓

ERROR

↓

CloudWatch Metric

↓

Alarm
```

Common Uses

- Failed Logins
- Exception Count
- HTTP 500 Errors
- Database Errors
- Security Events

---

# CloudWatch Logs Insights

CloudWatch Logs Insights is a query service used to analyze logs.

Instead of downloading log files, engineers can search logs directly.

Workflow

```text
CloudWatch Logs

↓

Logs Insights

↓

Query

↓

Results
```

Example Query

```sql
fields @timestamp, @message

| filter @message like /ERROR/

| sort @timestamp desc

| limit 20
```

Another Example

Find HTTP 500 Errors

```sql
fields @timestamp,@message

| filter status=500

| stats count() by bin(5m)
```

Useful For

- Production troubleshooting
- Security investigations
- Performance analysis
- Root Cause Analysis

---

# EventBridge Integration

CloudWatch Events are now part of Amazon EventBridge.

CloudWatch publishes events that EventBridge can process.

Workflow

```text
AWS Event

↓

EventBridge

↓

Rule

↓

Lambda

↓

SNS

↓

Email
```

Example Events

- EC2 State Change
- Auto Scaling Activity
- CodePipeline Execution
- ECR Image Push
- ECS Deployment
- Scheduled Jobs

---

# SNS Notifications

CloudWatch Alarms commonly send notifications using Amazon SNS.

Architecture

```text
CloudWatch Alarm

↓

SNS Topic

↓

Email

SMS

Lambda

Slack

Webhook
```

Typical Workflow

```text
CPU > 80%

↓

Alarm

↓

SNS

↓

DevOps Team
```

---

# EC2 Monitoring

CloudWatch monitors EC2 instances.

Default Metrics

- CPU Utilization
- Network In
- Network Out
- Disk Read Bytes
- Disk Write Bytes
- Status Checks

Additional Metrics (CloudWatch Agent)

- Memory Utilization
- Disk Usage
- Swap Usage
- File System Usage
- Running Processes

Architecture

```text
EC2

↓

CloudWatch Agent

↓

CloudWatch

↓

Dashboard
```

---

# Elastic Load Balancer Monitoring

CloudWatch automatically monitors ALBs and NLBs.

Important Metrics

- Request Count
- Target Response Time
- Healthy Hosts
- Unhealthy Hosts
- HTTP 4XX Errors
- HTTP 5XX Errors
- Active Connections

Workflow

```text
Application Load Balancer

↓

CloudWatch

↓

Alarm

↓

SNS
```

---

# Auto Scaling Monitoring

CloudWatch works closely with Auto Scaling.

Example

```text
CPU

85%

↓

CloudWatch Alarm

↓

Auto Scaling

↓

Launch EC2
```

Useful Metrics

- Group Desired Capacity
- Group InService Instances
- Group Pending Instances
- Group Terminating Instances

---

# Amazon EKS Monitoring

CloudWatch can monitor:

- Worker Nodes
- Pods (via Container Insights)
- Cluster Logs
- API Server
- Scheduler
- Controller Manager

Architecture

```text
Amazon EKS

↓

Container Insights

↓

CloudWatch

↓

Dashboard

↓

Alarm
```

Container Insights provides:

- Pod CPU
- Pod Memory
- Node CPU
- Node Memory
- Network Usage
- Disk Usage

---

# Enterprise Production Architecture

```text
                 AWS Infrastructure

 EC2   ALB   RDS   EKS   Lambda   ECS

      │     │     │      │      │

      └─────┼─────┼──────┼──────┘

          CloudWatch Metrics

                 │

          CloudWatch Logs

                 │

         CloudWatch Dashboard

                 │

        CloudWatch Alarms

                 │

             EventBridge

                 │

                 SNS

                 │

     Email / Slack / Lambda / PagerDuty
```

---

# AWS Console Walkthrough

1. Open CloudWatch Console
2. Navigate to Metrics
3. Select Namespace
4. Choose Metric
5. Create Dashboard
6. Create Alarm
7. Configure SNS Notification
8. Review Alarm State
9. View Logs
10. Query Logs Insights

---

# AWS CLI

List Metrics

```bash
aws cloudwatch list-metrics
```

Get Metric Statistics

```bash
aws cloudwatch get-metric-statistics \
--namespace AWS/EC2 \
--metric-name CPUUtilization \
--statistics Average
```

Create Alarm

```bash
aws cloudwatch put-metric-alarm \
--alarm-name HighCPU \
--metric-name CPUUtilization \
--namespace AWS/EC2 \
--statistic Average \
--threshold 80 \
--comparison-operator GreaterThanThreshold
```

Describe Alarms

```bash
aws cloudwatch describe-alarms
```

List Log Groups

```bash
aws logs describe-log-groups
```

List Log Streams

```bash
aws logs describe-log-streams \
--log-group-name /application/logs
```

---

# Terraform

Dashboard

```hcl
resource "aws_cloudwatch_dashboard" "operations" {

  dashboard_name = "operations-dashboard"

  dashboard_body = jsonencode({

    widgets = []

  })

}
```

CPU Alarm

```hcl
resource "aws_cloudwatch_metric_alarm" "cpu" {

  alarm_name = "HighCPU"

  namespace = "AWS/EC2"

  metric_name = "CPUUtilization"

  threshold = 80

  comparison_operator = "GreaterThanThreshold"

  evaluation_periods = 2

  statistic = "Average"

}
```

Log Group

```hcl
resource "aws_cloudwatch_log_group" "application" {

  name = "/application/logs"

  retention_in_days = 30

}
```

---

# CloudFormation

```yaml
Resources:

  CPUAlarm:

    Type: AWS::CloudWatch::Alarm

    Properties:

      AlarmName: HighCPU

      Namespace: AWS/EC2

      MetricName: CPUUtilization

      Threshold: 80
```

---

# Python (Boto3)

List Metrics

```python
import boto3

cloudwatch = boto3.client("cloudwatch")

response = cloudwatch.list_metrics()

print(response)
```

Create Alarm

```python
cloudwatch.put_metric_alarm(

    AlarmName="HighCPU",

    MetricName="CPUUtilization",

    Namespace="AWS/EC2",

    Threshold=80,

    ComparisonOperator="GreaterThanThreshold",

    EvaluationPeriods=2,

    Statistic="Average"

)
```

---

# Best Practices

- Enable Detailed Monitoring for production EC2 instances
- Install the CloudWatch Agent for OS-level metrics
- Create dashboards for critical applications
- Configure alarms for CPU, memory, disk, and network usage
- Store logs in structured formats (JSON where possible)
- Set appropriate log retention policies
- Use Composite Alarms to reduce alert fatigue
- Monitor Auto Scaling and ELB health
- Use Logs Insights for troubleshooting
- Integrate alarms with SNS and incident management systems

---

# Common Mistakes

- Relying only on CPU metrics
- Not monitoring memory utilization
- Keeping logs forever without retention policies
- Creating too many unnecessary alarms
- Ignoring alarm thresholds
- Not enabling detailed monitoring
- Missing application logs
- Using overly broad Logs Insights queries

---

# Troubleshooting

## Alarm Never Triggers

Verify:

- Metric exists
- Correct namespace
- Threshold
- Evaluation period
- Alarm state

---

## Missing Memory Metrics

Check:

- CloudWatch Agent
- IAM Role
- Agent Configuration
- Agent Status

---

## Logs Not Appearing

Verify:

- CloudWatch Agent
- IAM Permissions
- Log Group
- Log Stream
- Network Connectivity

---

## Dashboard Shows No Data

Check:

- Correct Region
- Namespace
- Metric Name
- Time Range
- Resource Availability

---

## Auto Scaling Not Triggered

Verify:

- Alarm State
- Scaling Policy
- Auto Scaling Group
- CloudWatch Metric
- Desired/Maximum Capacity

---

# Interview Questions

### Basic

1. What is Amazon CloudWatch?
2. Why is CloudWatch used?
3. What are CloudWatch Metrics?
4. What is a Namespace?
5. What are Dimensions?
6. Difference between Standard and Detailed Monitoring?
7. What is a CloudWatch Alarm?
8. What are Alarm States?
9. What is a Dashboard?
10. What are CloudWatch Logs?

### Intermediate

11. What is a Log Group?
12. What is a Log Stream?
13. What is a Metric Filter?
14. Explain Logs Insights.
15. What is the CloudWatch Agent?
16. How do you monitor memory utilization?
17. Explain Composite Alarms.
18. How does CloudWatch integrate with SNS?
19. How does CloudWatch integrate with Auto Scaling?
20. Explain EventBridge integration.

### Advanced

21. How does CloudWatch monitor EKS?
22. Difference between Metrics and Logs?
23. How would you troubleshoot missing metrics?
24. How do you monitor application logs?
25. Explain CloudWatch architecture.
26. How do you reduce alert fatigue?
27. How do you monitor microservices?
28. Explain custom metrics.
29. Design a monitoring solution for a production application.
30. How would you perform root cause analysis using CloudWatch?

---

# Scenario-Based Questions

### Scenario 1

A production EC2 instance is slow, but CPU utilization is only 20%.

How would you investigate?

---

### Scenario 2

Your CloudWatch Alarm never enters the ALARM state even though the application is overloaded.

What would you check?

---

### Scenario 3

Application logs are not appearing in CloudWatch Logs.

How would you troubleshoot?

---

### Scenario 4

A deployment causes HTTP 500 errors.

How would you identify the root cause using CloudWatch?

---

### Scenario 5

Auto Scaling is not launching new instances despite high traffic.

How does CloudWatch help identify the issue?

---

### Scenario 6

A company wants notifications sent to Slack whenever CPU exceeds 85%.

How would you design the solution?

---

### Scenario 7

Operations teams complain about too many alerts.

How can Composite Alarms improve the situation?

---

### Scenario 8

An Amazon EKS cluster experiences intermittent pod failures.

Which CloudWatch features would you use to investigate?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Metrics | Performance Data |
| Namespace | Metric Category |
| Dimensions | Metric Attributes |
| Alarm | Threshold Monitoring |
| Dashboard | Visualization |
| Logs | Application/System Logs |
| Log Group | Collection of Log Streams |
| Log Stream | Logs from a Single Source |
| Metric Filter | Convert Logs to Metrics |
| Logs Insights | Query Logs |
| EventBridge | Event Routing |
| SNS | Notifications |
| CloudWatch Agent | OS-Level Metrics |

---

# Summary

Amazon CloudWatch is AWS's centralized monitoring and observability platform. It collects metrics, logs, and events from AWS resources and applications, enabling teams to monitor infrastructure, visualize performance, automate responses, and troubleshoot issues.

In production environments, combine CloudWatch Metrics, Logs, Dashboards, Alarms, EventBridge, SNS, and the CloudWatch Agent to build a comprehensive monitoring solution. Monitor infrastructure, operating systems, applications, and business metrics together to achieve proactive operations, faster incident response, and higher application reliability.