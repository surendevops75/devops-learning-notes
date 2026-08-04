# AWS Cost Optimization

---

# Introduction

AWS Cost Optimization is the practice of reducing cloud costs while maintaining performance, reliability, security, and business requirements.

Goals:

- Reduce unnecessary spending
- Improve resource utilization
- Increase operational efficiency
- Build cost-aware architectures
- Implement FinOps practices

---

# AWS Pricing Models

## On-Demand

- Pay only for usage
- No long-term commitment
- Best for unpredictable workloads

---

## Reserved Instances (RI)

- 1-year or 3-year commitment
- Significant discounts
- Best for predictable workloads

---

## Savings Plans

- Flexible pricing model
- Commitment based on hourly spend
- Covers multiple compute services

---

## Spot Instances

- Up to 90% discount
- Uses spare AWS capacity
- Can be interrupted

---

## Dedicated Hosts

- Physical server dedicated to one customer
- License compliance
- Highest cost

---

# AWS Pricing Factors

Compute pricing depends on:

- Instance type
- Instance family
- Operating system
- Region
- Usage duration
- Storage
- Data transfer

---

# Shared Responsibility for Cost

AWS manages:

- Infrastructure
- Hardware
- Networking
- Facilities

Customer manages:

- Resource sizing
- Idle resources
- Scaling
- Scheduling
- Architecture
- Cost governance

---

# AWS Free Tier

Includes examples such as:

- EC2 free hours
- S3 storage
- Lambda requests
- DynamoDB storage
- CloudWatch metrics

Always verify current Free Tier limits before planning workloads.

---

# AWS Billing Dashboard

Provides visibility into:

- Current month charges
- Forecasted costs
- Service-wise spending
- Region-wise spending
- Linked accounts
- Savings Plans utilization

---

# Cost Allocation Tags

Example

```text
Environment = Production

Project = Payments

Application = API

Team = DevOps

Owner = Platform

BusinessUnit = Finance
```

---

# Tag Categories

Mandatory Tags

- Environment
- Project
- Owner
- CostCenter
- Department

Optional Tags

- Application
- Compliance
- Backup
- Team

---

# Untagged Resources

Problems

- No chargeback
- Difficult reporting
- Unknown ownership
- Cost leakage

---

# AWS Organizations Billing

Features

- Consolidated Billing
- Linked Accounts
- Cost Allocation
- Savings Sharing
- Reserved Instance Sharing

---

# Cost Allocation Workflow

```text
AWS Resources

↓

Apply Tags

↓

Cost Allocation Tags

↓

Billing Reports

↓

Cost Explorer

↓

Business Reports
```

---

# Cost Categories

Examples

```text
Production

Development

Testing

Security

Shared Services

Networking

Monitoring
```

---

# AWS Billing Alerts

Example Workflow

```text
Billing

↓

CloudWatch Alarm

↓

SNS

↓

Email

↓

Slack

↓

Operations Team
```

---

# Enable Billing Alerts

```text
Billing Preferences

↓

Receive Billing Alerts

↓

CloudWatch

↓

SNS Notification
```

---

# Monthly Cost Review

Review

- EC2
- EBS
- RDS
- S3
- NAT Gateway
- Elastic IP
- Load Balancers
- Data Transfer
- CloudFront
- Lambda

---

# Common Cost Mistakes

- Leaving EC2 running
- Unused EBS volumes
- Idle Load Balancers
- Idle NAT Gateways
- Public snapshots
- Unused Elastic IPs
- Large CloudWatch retention
- Overprovisioned databases
- Missing lifecycle policies
- Duplicate backups

---

# Cost Optimization Lifecycle

```text
Provision

↓

Monitor

↓

Analyze

↓

Optimize

↓

Automate

↓

Govern

↓

Repeat
```

---

# Cost Governance

Includes

- Budgets
- Policies
- Resource tagging
- Chargeback
- Showback
- Approval workflows
- Periodic reviews

---

# FinOps Principles

- Visibility
- Accountability
- Optimization
- Continuous improvement
- Collaboration
- Business alignment

---

# Key AWS Cost Tools

- AWS Billing Console
- Cost Explorer
- AWS Budgets
- Cost & Usage Report (CUR)
- Cost Anomaly Detection
- Trusted Advisor
- Compute Optimizer
- Resource Groups
- AWS Organizations

---

# Cost Optimization Checklist

Daily

- Review alarms
- Check anomalies
- Verify production resources

Weekly

- Remove idle resources
- Review EC2 utilization
- Review storage growth

Monthly

- Review billing trends
- Purchase Savings Plans
- Review Reserved Instances
- Optimize architecture
- Review tagging compliance

---

# Best Practices

- Tag every resource.
- Use consolidated billing.
- Enable billing alerts.
- Review monthly spending.
- Right-size infrastructure.
- Remove idle resources quickly.
- Implement FinOps governance.
- Monitor costs continuously.
- Use AWS-native cost management tools.
- Automate repetitive cost optimization tasks.

---

# Summary

This section introduced AWS Cost Optimization fundamentals, pricing models, billing concepts, tagging strategies, cost governance, FinOps principles, and the core AWS cost management tools. These concepts form the foundation for building a cost-efficient and well-governed AWS environment.

---

# AWS Cost Explorer

---

# Introduction

AWS Cost Explorer is a cost analysis tool that helps visualize, analyze, forecast, and optimize AWS spending.

Features include:

- Historical cost analysis
- Forecasting
- Cost breakdowns
- Usage analysis
- Savings Plans analysis
- Reserved Instance analysis

---

# Cost Explorer Workflow

```text
AWS Billing Data

↓

Cost Explorer

↓

Analyze Trends

↓

Identify Cost Drivers

↓

Optimize Resources

↓

Monitor Savings
```

---

# Cost Explorer Dashboard

Provides visibility into:

- Monthly cost
- Daily cost
- Forecasted cost
- Service-wise spending
- Region-wise spending
- Linked account spending
- Usage trends

---

# Cost Views

## Daily

Used for:

- Detecting spikes
- Daily monitoring

---

## Monthly

Used for:

- Budget planning
- Long-term analysis

---

## Hourly

Useful for:

- Investigating incidents
- Temporary workloads

---

# Group By Options

Group costs by:

- Service
- Region
- Linked Account
- Availability Zone
- Instance Type
- Usage Type
- Cost Allocation Tag
- API Operation

---

# Filter Options

Common filters

- AWS Service
- Region
- Usage Type
- Linked Account
- Purchase Option
- Instance Type
- Tag
- Cost Category

---

# Service-wise Analysis

Example

```text
Amazon EC2

Amazon S3

Amazon RDS

AWS Lambda

Amazon EKS

Amazon ECS

CloudFront

CloudWatch
```

---

# Region-wise Analysis

Example

```text
ap-south-1

us-east-1

eu-west-1

ap-southeast-1
```

---

# Linked Account Analysis

Useful for

- Chargeback
- Team billing
- Department billing
- Business unit reporting

---

# Tag-Based Analysis

Example

```text
Project = Payments

Environment = Production

Owner = Platform

Department = Finance
```

---

# Purchase Option Analysis

Analyze spending for

- On-Demand
- Reserved Instances
- Savings Plans
- Spot Instances

---

# EC2 Cost Analysis

Review

- Instance families
- Instance types
- Running hours
- Idle instances
- Utilization

---

# Storage Cost Analysis

Review

- S3
- EBS
- EFS
- FSx
- Snapshots

---

# Database Cost Analysis

Review

- RDS
- Aurora
- DynamoDB
- ElastiCache

---

# Networking Cost Analysis

Review

- NAT Gateway

- Elastic IP

- Data Transfer

- Transit Gateway

- Load Balancers

- CloudFront

---

# Forecasting

Forecasts estimate future spending based on historical usage patterns.

Example

```text
Current Month

↓

Historical Trend

↓

Forecast

↓

Budget Planning
```

---

# Cost Trends

Example

```text
January

↓

February

↓

March

↓

April

↓

Increasing Storage Costs
```

---

# Cost Spike Investigation

Workflow

```text
Unexpected Bill

↓

Identify Service

↓

Identify Region

↓

Identify Resource

↓

Check Recent Changes

↓

Optimize
```

---

# Analyze Top Services

Typical review order

1. EC2
2. EBS
3. RDS
4. S3
5. NAT Gateway
6. Data Transfer
7. EKS
8. CloudFront

---

# Analyze Resource Growth

Review

- Running instances
- Storage usage
- Database growth
- Snapshot count
- Load Balancers
- Elastic IPs

---

# Monthly Review Checklist

Review

- Highest-cost services
- Highest-cost regions
- Unused resources
- Idle resources
- New services
- Storage growth
- Data transfer charges

---

# Savings Plans Analysis

Review

- Coverage %
- Utilization %
- Hourly commitment
- Estimated savings
- Expiration dates

---

# Reserved Instance Analysis

Review

- Coverage
- Utilization
- Expiration
- Family compatibility

---

# Visualization Types

Available views

- Line Chart
- Bar Chart
- Stacked Bar
- Table

---

# Export Reports

Supported formats

- CSV
- Billing reports
- Cost & Usage Reports (CUR)

---

# Practical Investigation Example

```text
Monthly Cost Increased

↓

Filter by Service

↓

Amazon EC2

↓

Group by Instance Type

↓

Identify New Instances

↓

Terminate Idle Resources

↓

Validate Savings
```

---

# Cost Review Frequency

Daily

- Billing spikes
- Anomalies
- Production costs

Weekly

- Service growth
- Resource utilization
- Idle resources

Monthly

- Budgets
- Savings Plans
- Reserved Instances
- Long-term trends

---

# Best Practices

- Review Cost Explorer weekly.
- Group costs using cost allocation tags.
- Investigate sudden cost increases immediately.
- Review region-wise spending regularly.
- Track storage growth every month.
- Analyze EC2 usage before purchasing Savings Plans.
- Review Reserved Instance utilization.
- Export reports for business analysis.
- Compare forecasts with budgets.
- Make Cost Explorer reviews part of operational governance.

---

# Summary

This section covered AWS Cost Explorer, cost visualization, service-wise and region-wise analysis, filtering and grouping, forecasting, Savings Plans analysis, Reserved Instance analysis, cost investigations, and reporting. These capabilities help identify spending trends, optimize cloud resources, and improve financial governance.

---

