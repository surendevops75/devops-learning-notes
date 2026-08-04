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

# AWS Budgets

---

# Introduction

AWS Budgets helps monitor AWS spending, usage, Reserved Instances (RI), and Savings Plans against defined limits.

It supports:

- Cost Budgets
- Usage Budgets
- Reservation Budgets
- Savings Plans Budgets

---

# Budget Workflow

```text
Create Budget

↓

Monitor Spending

↓

Compare Against Threshold

↓

Send Notification

↓

Take Action

↓

Reduce Costs
```

---

# Budget Types

## Cost Budget

Tracks total AWS spending.

Example

```text
Monthly Budget

$2,000
```

---

## Usage Budget

Tracks service usage instead of cost.

Example

```text
EC2 Running Hours

500 Hours
```

---

## Reserved Instance Budget

Tracks

- RI Utilization
- RI Coverage

---

## Savings Plans Budget

Tracks

- Savings Plan Coverage
- Savings Plan Utilization

---

# Budget Periods

Supported periods

- Daily
- Monthly
- Quarterly
- Annually

---

# Budget Thresholds

Example

```text
50%

↓

80%

↓

90%

↓

100%

↓

110%
```

---

# Budget Notifications

Notifications can be sent using

- Email
- Amazon SNS

---

# Budget Notification Workflow

```text
Budget

↓

Threshold Reached

↓

SNS

↓

Email

↓

Operations Team
```

---

# Budget Actions

AWS Budgets Actions can automatically perform actions when thresholds are exceeded.

Examples

- Stop EC2 instances
- Apply IAM policy
- Restrict new resource creation
- Notify administrators

---

# Budget Example

```text
Monthly Budget

$1000

Alert

80%

Critical

100%
```

---

# Multi-Account Budgets

Using AWS Organizations

```text
Management Account

↓

Linked Accounts

↓

Department Budgets

↓

Team Budgets
```

---

# Budget Reports

Review

- Actual Cost
- Forecast Cost
- Remaining Budget
- Percentage Used

---

# Cost & Usage Report (CUR)

---

# Introduction

Cost & Usage Report (CUR) is the most detailed AWS billing report.

Contains

- Resource-level costs
- Usage data
- Pricing information
- Discounts
- Savings Plans
- Reserved Instances
- Tags

---

# CUR Workflow

```text
AWS Billing

↓

CUR

↓

Amazon S3

↓

AWS Glue

↓

Amazon Athena

↓

QuickSight Dashboard
```

---

# CUR Storage

Best practice

```text
Amazon S3

↓

Partitioned Data

↓

Glue Catalog

↓

Athena Queries
```

---

# CUR Data Includes

- Resource IDs
- Service names
- Usage types
- Pricing
- Tags
- Regions
- Accounts
- Discounts
- Credits

---

# CUR Delivery

Formats

- Parquet (Recommended)
- CSV

Compression

- GZIP

---

# AWS Glue Integration

Workflow

```text
CUR

↓

Glue Crawler

↓

Glue Data Catalog

↓

Athena
```

---

# Athena Analysis

Example questions

- Which EC2 instances cost the most?
- Which team spends the most?
- Which AWS Region has the highest cost?
- Which S3 buckets generate the most storage charges?

---

# Amazon QuickSight

Dashboard Examples

- Monthly spending
- Team spending
- Cost trends
- Resource utilization
- Department reports

---

# Cost Allocation Tags in CUR

Example

```text
Environment

Project

Owner

CostCenter

Application

BusinessUnit
```

---

# CUR Retention

Recommended

- Store CUR in versioned S3 buckets
- Apply lifecycle policies
- Archive old reports to Glacier

---

# Cost Anomaly Detection

---

# Introduction

AWS Cost Anomaly Detection uses machine learning to identify unusual spending patterns automatically.

---

# Workflow

```text
AWS Billing

↓

Machine Learning

↓

Anomaly Detected

↓

Notification

↓

Investigation

↓

Optimization
```

---

# Anomaly Monitor Types

Supported monitors

- AWS Services
- Linked Accounts
- Cost Categories
- Cost Allocation Tags

---

# Service Monitor Example

```text
Amazon EC2

Amazon S3

Amazon RDS

Amazon Lambda
```

---

# Account Monitor Example

```text
Production

Development

Testing
```

---

# Tag-Based Monitor

```text
Environment=Production

Project=Payments

Team=Platform
```

---

# Cost Category Monitor

Example

```text
Infrastructure

Applications

Security

Networking
```

---

# Anomaly Subscription

Notification methods

- Email
- Amazon SNS

---

# Alert Workflow

```text
Unexpected Cost

↓

Cost Anomaly Detection

↓

SNS

↓

Email

↓

Cloud Team
```

---

# Investigation Workflow

```text
Alert

↓

Cost Explorer

↓

Identify Service

↓

Identify Resource

↓

Optimize

↓

Verify Savings
```

---

# Common Anomalies

- Large EC2 deployment
- Unused NAT Gateway
- Excessive data transfer
- Snapshot growth
- Unexpected Lambda invocations
- Large S3 uploads
- High CloudWatch ingestion
- Overprovisioned databases

---

# Monthly FinOps Reports

Include

- Monthly spend
- Forecast
- Budget status
- Cost anomalies
- Savings Plans utilization
- Reserved Instance utilization
- Top cost drivers
- Optimization opportunities

---

# Operational Review

Daily

- Budget alerts
- Cost anomalies
- Billing spikes

Weekly

- Team spending
- Resource growth
- Tag compliance

Monthly

- Executive report
- Optimization plan
- Savings review
- Budget adjustments

---

# Best Practices

- Create budgets for every production account.
- Configure multiple alert thresholds.
- Use Budget Actions for automated governance.
- Enable Cost & Usage Reports in Parquet format.
- Analyze CUR using Athena instead of spreadsheets.
- Build QuickSight dashboards for executives.
- Enable Cost Anomaly Detection for all production accounts.
- Investigate anomalies immediately.
- Apply consistent cost allocation tags.
- Review budgets and forecasts every month.

---

# Summary

This section covered AWS Budgets, Budget Actions, Cost & Usage Reports (CUR), AWS Glue integration, Amazon Athena analysis, QuickSight dashboards, Cost Anomaly Detection, anomaly monitors, and FinOps reporting workflows. Together, these services provide comprehensive visibility into AWS spending, automated alerting, and data-driven cost optimization.

---

# Savings Plans

---

# Introduction

AWS Savings Plans provide flexible pricing based on a committed hourly spend over a 1-year or 3-year term.

Benefits

- Lower compute costs
- Flexible usage
- Automatic discount application
- Simpler than Reserved Instances

---

# Savings Plans Workflow

```text
Analyze Usage

↓

Purchase Savings Plan

↓

Hourly Commitment

↓

Automatic Discounts

↓

Reduced AWS Bill
```

---

# Savings Plans Types

## Compute Savings Plans

Supports

- Amazon EC2
- AWS Lambda
- AWS Fargate

Benefits

- Maximum flexibility
- Works across instance families
- Works across regions
- Supports operating system changes

---

## EC2 Instance Savings Plans

Applies to

- Specific instance family
- Specific AWS Region

Provides higher discounts than Compute Savings Plans.

---

## SageMaker Savings Plans

Supports

- SageMaker Training
- SageMaker Inference
- SageMaker Processing

---

# Savings Plans Terms

Available commitments

- 1 Year
- 3 Years

Payment options

- No Upfront
- Partial Upfront
- All Upfront

---

# Savings Plans Coverage

Measures

```text
Eligible Usage

↓

Covered Usage

↓

Coverage %
```

Higher coverage generally results in greater savings.

---

# Savings Plans Utilization

Measures

```text
Purchased Commitment

↓

Actual Usage

↓

Utilization %
```

Unused commitment represents lost savings opportunities.

---

# Savings Plans Recommendation Workflow

```text
Historical Usage

↓

AWS Recommendation

↓

Review

↓

Purchase

↓

Monitor Utilization
```

---

# Reserved Instances (RI)

---

# Introduction

Reserved Instances provide discounted pricing for predictable EC2 workloads.

---

# RI Types

## Standard Reserved Instances

Benefits

- Highest discount
- Fixed configuration
- Limited flexibility

---

## Convertible Reserved Instances

Benefits

- Exchange supported
- Flexible
- Lower discount than Standard RI

---

# Regional Reserved Instances

Benefits

- Capacity flexibility
- Size flexibility
- Availability Zone flexibility

---

# Zonal Reserved Instances

Benefits

- Capacity reservation
- Fixed Availability Zone
- Fixed capacity

---

# RI Terms

Available

- 1 Year
- 3 Years

Payment options

- No Upfront
- Partial Upfront
- All Upfront

---

# RI Coverage

Review

- Covered hours
- Remaining On-Demand usage
- Uncovered workloads

---

# RI Utilization

Review

- Utilized reservations
- Idle reservations
- Expiring reservations

---

# Reserved Instance Recommendation

```text
Historical Usage

↓

AWS Recommendation

↓

Review

↓

Purchase RI

↓

Monitor Usage
```

---

# Savings Plans vs Reserved Instances

| Feature | Savings Plans | Reserved Instances |
|----------|---------------|-------------------|
| Flexibility | High | Medium |
| Instance Family Change | Yes (Compute SP) | Limited |
| Region Change | Compute SP Only | Regional RI Only |
| Capacity Reservation | No | Zonal RI |
| Discount | High | Highest |

---

# Spot Instances

---

# Introduction

Spot Instances use spare AWS capacity and can provide discounts of up to 90% compared to On-Demand pricing.

---

# Best Use Cases

- Batch processing
- CI/CD builds
- Kubernetes worker nodes
- Big data processing
- Rendering
- Machine Learning training
- Fault-tolerant applications

---

# Avoid Spot For

- Critical databases
- Stateful applications
- Long-running critical jobs
- Single-instance production workloads

---

# Spot Interruption

AWS may reclaim Spot capacity with a two-minute interruption notice.

Workflow

```text
Spot Instance

↓

Interruption Notice

↓

Drain Workload

↓

Launch Replacement

↓

Continue Processing
```

---

# Spot Fleet

Provides

- Multiple instance types
- Multiple Availability Zones
- Lowest cost selection
- Automatic diversification

---

# EC2 Fleet

Supports

- On-Demand
- Reserved
- Spot

Example

```text
20%

On-Demand

↓

80%

Spot
```

---

# Mixed Instance Policy

Example

```text
Auto Scaling Group

↓

On-Demand Base Capacity

↓

Spot Capacity

↓

Multiple Instance Types
```

---

# Capacity Rebalancing

Automatically launches replacement Spot Instances before interruption occurs.

---

# Spot Allocation Strategies

Available strategies

- Price Capacity Optimized
- Capacity Optimized
- Lowest Price

Production recommendation

- Capacity Optimized
- Price Capacity Optimized

---

# Spot Cost Optimization Workflow

```text
Identify Fault-Tolerant Workloads

↓

Launch Spot Instances

↓

Monitor Interruptions

↓

Auto Scaling

↓

Reduce Costs
```

---

# Purchase Recommendation Workflow

```text
Analyze Usage

↓

Review Cost Explorer

↓

Savings Plans Recommendation

↓

RI Recommendation

↓

Spot Opportunity

↓

Optimize Architecture
```

---

# Example Workload Strategy

Production API

```text
Savings Plans
```

---

Development Environment

```text
Spot Instances
```

---

Batch Processing

```text
Spot Fleet
```

---

Database

```text
Reserved Instances
```

---

Lambda

```text
Compute Savings Plans
```

---

# Decision Matrix

| Workload | Recommended Option |
|----------|--------------------|
| Predictable EC2 | Reserved Instances |
| Flexible Compute | Compute Savings Plans |
| Lambda | Compute Savings Plans |
| Fargate | Compute Savings Plans |
| Temporary Batch Jobs | Spot Instances |
| CI/CD Agents | Spot Instances |
| Development | Spot Instances |
| Production Database | Reserved Instances |

---

# Common Mistakes

- Purchasing Savings Plans without usage analysis.
- Buying Standard RIs for changing workloads.
- Running production databases entirely on Spot Instances.
- Ignoring RI expiration.
- Low Savings Plans utilization.
- Running all workloads On-Demand.
- Not diversifying Spot instance types.
- Ignoring interruption handling.
- Not reviewing recommendations regularly.

---

# Review Schedule

Weekly

- Spot interruptions
- Spot utilization
- Savings Plans utilization

Monthly

- RI utilization
- Coverage reports
- Purchase recommendations
- Expiring commitments

Quarterly

- Re-evaluate purchasing strategy
- Optimize commitments
- Review architecture changes

---

# Best Practices

- Purchase Compute Savings Plans for flexible production workloads.
- Use EC2 Instance Savings Plans for stable workloads in a single Region.
- Purchase Reserved Instances only for predictable long-term workloads.
- Diversify Spot workloads across multiple instance families and Availability Zones.
- Enable Capacity Rebalancing for Spot Auto Scaling Groups.
- Review Savings Plans and RI utilization monthly.
- Mix On-Demand and Spot Instances using Auto Scaling Groups.
- Follow AWS purchase recommendations instead of guessing commitment levels.
- Continuously monitor commitment coverage and utilization.
- Match purchasing strategy to workload characteristics.

---

# Summary

This section covered AWS Savings Plans, Reserved Instances, Spot Instances, Spot Fleet, EC2 Fleet, Capacity Rebalancing, mixed instance policies, purchasing recommendations, workload decision matrices, and production optimization strategies. These pricing models are fundamental for reducing AWS compute costs while maintaining application performance and availability.

---

# Amazon EC2 Cost Optimization

---

# Right-Sizing

Review

- CPU utilization
- Memory utilization
- Network throughput
- Disk utilization

Example

```text
m5.2xlarge

↓

m6i.large

↓

Lower Cost
```

---

# Stop Idle Instances

Review

- Development
- Testing
- Sandbox
- Training

Automate shutdown during non-business hours.

---

# Auto Scaling

Benefits

- Launch instances only when needed
- Remove idle capacity
- Reduce On-Demand costs

---

# Mixed Instance Auto Scaling

Example

```text
20%

On-Demand

↓

80%

Spot
```

---

# Use Latest Generation Instances

Example

```text
m5

↓

m7i

↓

Better Price/Performance
```

---

# Graviton Migration

Example

```text
x86

↓

AWS Graviton

↓

Lower Compute Cost
```

---

# Elastic IP Optimization

Remove

- Unused Elastic IPs
- Detached Elastic IPs

---

# Dedicated Hosts Review

Only use when required for

- License compliance
- Regulatory requirements

---

# Amazon EBS Cost Optimization

---

# gp2 to gp3

Example

```text
gp2

↓

gp3

↓

Lower Cost
```

---

# Delete Unused Volumes

Review

- Available volumes
- Detached volumes

---

# Delete Old Snapshots

Review

- Obsolete backups
- Duplicate snapshots

---

# Snapshot Lifecycle

Workflow

```text
Snapshot

↓

Retention

↓

Archive

↓

Delete
```

---

# Right-Size Volumes

Avoid

- Overprovisioned storage
- Unused IOPS

---

# Amazon S3 Cost Optimization

---

# Lifecycle Policies

Workflow

```text
Standard

↓

Standard-IA

↓

Glacier Instant Retrieval

↓

Glacier Flexible Retrieval

↓

Deep Archive
```

---

# Intelligent-Tiering

Ideal for

- Unknown access patterns
- Frequently changing workloads

---

# Delete Incomplete Multipart Uploads

Automatically remove abandoned uploads.

---

# Remove Old Object Versions

Review

- Versioned buckets
- Obsolete versions

---

# Compress Data

Examples

- GZIP
- Parquet
- ORC

---

# Amazon RDS Cost Optimization

---

# Right-Size Instances

Review

- CPU
- Memory
- Storage
- Connections

---

# Multi-AZ Review

Enable only for

- Production
- Business-critical databases

---

# Storage Autoscaling

Enable automatic storage growth.

---

# Delete Old Snapshots

Keep only required backups.

---

# Reserved Instances

Recommended for

- Production databases
- Predictable workloads

---

# Aurora Optimization

---

# Serverless v2

Ideal for

- Variable workloads
- Development
- Event-driven applications

---

# Reader Instances

Add replicas only when needed.

---

# DynamoDB Cost Optimization

---

# Billing Mode

Choose

- On-Demand
- Provisioned

Based on workload characteristics.

---

# Auto Scaling

Enable for provisioned capacity.

---

# TTL

Automatically remove expired items.

---

# Global Tables Review

Deploy only when cross-region replication is required.

---

# Lambda Cost Optimization

---

# Memory Tuning

Review

- Duration
- Memory
- CPU allocation

---

# Provisioned Concurrency

Use only for latency-sensitive workloads.

---

# Remove Unused Functions

Delete

- Old versions
- Unused aliases
- Deprecated functions

---

# Optimize Package Size

Reduce deployment package size to improve cold starts.

---

# Amazon EKS Cost Optimization

---

# Managed Node Groups

Use managed node groups to simplify operations.

---

# Cluster Autoscaler

Automatically scale worker nodes.

---

# Karpenter

Launch right-sized instances dynamically.

---

# Spot Worker Nodes

Ideal for

- CI/CD
- Batch jobs
- Stateless applications

---

# Remove Idle Clusters

Delete unused development clusters.

---

# Amazon ECS Cost Optimization

---

# Fargate Spot

Use for interruptible workloads.

---

# Right-Size Tasks

Review

- CPU
- Memory

---

# Service Auto Scaling

Scale tasks automatically based on demand.

---

# CloudFront Cost Optimization

---

# Cache Optimization

Increase cache hit ratio.

Benefits

- Lower origin requests
- Lower bandwidth cost

---

# Compression

Enable

- GZIP
- Brotli

---

# Origin Optimization

Reduce unnecessary origin fetches.

---

# AWS Networking Cost Optimization

---

# NAT Gateway

Reduce NAT traffic using

- Gateway Endpoints
- Interface Endpoints

---

# Data Transfer

Review

- Cross-AZ traffic
- Cross-region traffic
- Internet egress

---

# VPC Endpoints

Use

- S3 Gateway Endpoint
- DynamoDB Gateway Endpoint
- Interface Endpoints

To reduce NAT Gateway charges.

---

# Load Balancers

Delete unused

- ALBs
- NLBs
- CLBs

---

# CloudWatch Cost Optimization

---

# Log Retention

Configure

- 7 Days
- 30 Days
- 90 Days

Avoid indefinite retention.

---

# Custom Metrics

Delete unused metrics.

---

# Log Filtering

Reduce unnecessary log ingestion.

---

# Container Image Optimization

Review

- Old ECR images
- Untagged images
- Unused repositories

Apply lifecycle policies.

---

# Backup Optimization

Review

- Backup frequency
- Snapshot retention
- Cross-region copies

---

# Common Service-Level Waste

- Idle EC2 instances
- Detached EBS volumes
- Old EBS snapshots
- Unused Elastic IPs
- Idle NAT Gateways
- Idle Load Balancers
- Old Lambda versions
- Unused ECR images
- Large CloudWatch log retention
- Orphaned EKS clusters

---

# Optimization Workflow

```text
Monitor

↓

Identify Waste

↓

Analyze Usage

↓

Right-Size

↓

Automate

↓

Review Monthly
```

---

# Monthly Service Review Checklist

EC2

- Idle instances
- Right sizing
- Savings Plans coverage

---

Storage

- EBS
- S3
- Snapshots

---

Database

- RDS
- Aurora
- DynamoDB

---

Networking

- NAT Gateway
- Elastic IP
- Data Transfer

---

Containers

- ECS
- EKS
- ECR

---

Monitoring

- CloudWatch
- Logs
- Metrics

---

# Best Practices

- Right-size compute resources regularly.
- Upgrade EBS volumes from gp2 to gp3 where appropriate.
- Use S3 lifecycle policies for archival.
- Enable Intelligent-Tiering for unpredictable access patterns.
- Purchase Reserved Instances for long-running databases.
- Use Cluster Autoscaler or Karpenter for EKS.
- Reduce NAT Gateway traffic with VPC Endpoints.
- Configure CloudWatch log retention policies.
- Apply ECR lifecycle policies to remove old images.
- Review service utilization every month.

---

# Summary

This section covered service-level cost optimization techniques for Amazon EC2, EBS, S3, RDS, Aurora, DynamoDB, Lambda, EKS, ECS, CloudFront, networking, CloudWatch, ECR, and backup strategies. These practices help eliminate waste, improve utilization, and significantly reduce AWS operational costs while maintaining performance and reliability.

---

