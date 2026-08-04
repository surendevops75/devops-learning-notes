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

# AWS Trusted Advisor

---

# Introduction

AWS Trusted Advisor analyzes AWS resources and provides recommendations to improve:

- Cost Optimization
- Performance
- Security
- Fault Tolerance
- Service Limits
- Operational Excellence

---

# Trusted Advisor Workflow

```text
AWS Resources

↓

Trusted Advisor

↓

Recommendations

↓

Review

↓

Implement Fixes

↓

Reduce Cost
```

---

# Cost Optimization Checks

Examples

- Idle EC2 Instances
- Underutilized EC2
- Low Utilization EBS Volumes
- Idle Load Balancers
- Idle NAT Gateways
- Unassociated Elastic IPs
- Amazon RDS Idle Instances
- S3 Incomplete Multipart Uploads

---

# Performance Checks

Review

- High CPU
- Network throughput
- EBS performance
- Load Balancer health

---

# Security Checks

Review

- IAM usage
- Security Groups
- MFA
- Public buckets
- Root account

---

# Fault Tolerance Checks

Examples

- Multi-AZ databases
- Auto Scaling
- Backup configuration
- Health checks

---

# Service Limits

Review

- EC2 limits
- VPC limits
- EBS limits
- IAM limits
- Lambda limits

---

# Operational Excellence

Review

- Monitoring
- Logging
- Automation
- Governance

---

# Trusted Advisor Priority

```text
High

↓

Medium

↓

Low
```

---

# Weekly Review

Review

- New recommendations
- Fixed recommendations
- Cost savings
- Resource growth

---

# AWS Compute Optimizer

---

# Introduction

AWS Compute Optimizer analyzes resource utilization and recommends optimal configurations using machine learning.

Supports

- EC2
- Auto Scaling Groups
- EBS
- Lambda
- ECS Services

---

# Compute Optimizer Workflow

```text
CloudWatch Metrics

↓

Machine Learning

↓

Recommendation

↓

Review

↓

Resize

↓

Save Money
```

---

# EC2 Recommendations

Review

- CPU utilization
- Memory utilization
- Network throughput
- Disk throughput

Example

```text
Current

m5.2xlarge

↓

Recommended

m7i.large
```

---

# Auto Scaling Recommendations

Optimize

- Minimum capacity
- Maximum capacity
- Desired capacity

---

# EBS Recommendations

Review

- Volume type
- IOPS
- Throughput
- Capacity

Example

```text
gp2

↓

gp3
```

---

# Lambda Recommendations

Review

- Memory allocation
- Execution duration
- Cost
- Performance

---

# ECS Recommendations

Review

- CPU
- Memory
- Task sizing

---

# Recommendation Categories

- Over-provisioned
- Under-provisioned
- Optimized

---

# Savings Estimate

Example

```text
Current Cost

↓

Recommended Cost

↓

Monthly Savings
```

---

# AWS Resource Explorer

---

# Benefits

- Find AWS resources
- Search across Regions
- Resource inventory
- Governance support

---

# Resource Groups

Examples

```text
Production

Development

Networking

Databases

Security
```

---

# Tag-Based Resource Groups

Example

```text
Environment=Production

Project=Payments
```

---

# AWS Health Dashboard

---

# Review

- Scheduled maintenance
- Service events
- Regional issues
- Resource health

---

# AWS Personal Health Dashboard

Provides

- Account-specific events
- Maintenance notifications
- Service disruptions

---

# Optimization Workflow

```text
Trusted Advisor

↓

Compute Optimizer

↓

Resource Explorer

↓

Review Findings

↓

Optimize Resources

↓

Measure Savings
```

---

# Operational Review

Daily

- Health events
- Critical recommendations

---

Weekly

- Trusted Advisor findings
- Compute Optimizer recommendations
- Resource inventory

---

Monthly

- Resource cleanup
- Cost savings report
- Governance review

---

# Enterprise Governance

Include

- Resource ownership
- Tag compliance
- Budget compliance
- Cost allocation
- Optimization review
- Executive reporting

---

# Automated Optimization

Examples

- Stop idle EC2 instances
- Delete unattached EBS volumes
- Remove unused Elastic IPs
- Archive old snapshots
- Delete old ECR images

---

# Example Automation Workflow

```text
CloudWatch Event

↓

Lambda

↓

Check Resource

↓

Delete or Stop

↓

Notify Team
```

---

# Resource Cleanup Checklist

Compute

- Idle EC2
- Old AMIs
- Auto Scaling review

---

Storage

- Unused EBS
- Old snapshots
- S3 lifecycle

---

Networking

- Idle NAT Gateway
- Elastic IP
- Load Balancer

---

Containers

- Old ECR images
- Idle ECS services
- Unused EKS clusters

---

Monitoring

- Old log groups
- Custom metrics
- Dashboards

---

# KPIs

Track

- Monthly cloud spend
- Cost per application
- Savings achieved
- Resource utilization
- Idle resource count
- Tag compliance
- Budget adherence
- Recommendation completion rate

---

# Common Findings

- Oversized EC2 instances
- Detached EBS volumes
- Idle Elastic IPs
- Low-utilization Load Balancers
- Large CloudWatch log retention
- Old snapshots
- Unused AMIs
- Underutilized ECS services

---

# Best Practices

- Review Trusted Advisor recommendations weekly.
- Review Compute Optimizer recommendations monthly.
- Enable enhanced infrastructure metrics where supported.
- Use consistent resource tagging for governance.
- Automate cleanup of unused resources.
- Review AWS Health Dashboard regularly.
- Prioritize high-impact optimization recommendations.
- Track realized savings after implementing changes.
- Measure optimization KPIs over time.
- Make optimization reviews part of regular operational processes.

---

# Summary

This section covered AWS Trusted Advisor, Compute Optimizer, Resource Explorer, Resource Groups, AWS Health Dashboard, automated optimization workflows, governance practices, resource cleanup strategies, and optimization KPIs. These services help identify waste, improve resource utilization, automate remediation, and maintain a cost-efficient AWS environment.

---

# Introduction to FinOps

---

# What is FinOps?

FinOps (Financial Operations) is the practice of bringing engineering, finance, and business teams together to maximize the business value of cloud spending.

Goals

- Cost visibility
- Accountability
- Continuous optimization
- Business alignment
- Data-driven decisions

---

# FinOps Lifecycle

```text
Inform

↓

Optimize

↓

Operate

↓

Repeat
```

---

# FinOps Teams

Engineering

- Optimize resources
- Improve utilization
- Reduce waste

---

Finance

- Budget planning
- Forecasting
- Cost reporting

---

Business

- ROI
- Product profitability
- Cost ownership

---

Cloud Operations

- Governance
- Automation
- Monitoring

---

# Cloud Cost Governance

Includes

- Resource ownership
- Budget controls
- Cost policies
- Approval workflows
- Continuous reviews

---

# Cost Allocation

Purpose

- Identify resource owners
- Allocate cloud expenses
- Enable business reporting

---

# Cost Allocation Tags

Mandatory

```text
Environment

Project

Application

Owner

Department

BusinessUnit

CostCenter
```

---

Optional

```text
Compliance

Backup

ManagedBy

Region

SupportLevel
```

---

# Tagging Workflow

```text
Provision Resource

↓

Apply Tags

↓

Billing Data

↓

Cost Reports

↓

Chargeback

↓

Optimization
```

---

# Tag Governance

Rules

- Every resource must be tagged.
- Use standardized tag names.
- Enforce tagging using IAM or SCPs.
- Review tag compliance regularly.

---

# Cost Categories

Examples

```text
Infrastructure

Applications

Security

Networking

Monitoring

Shared Services

Development

Production
```

---

# Chargeback

Definition

Chargeback bills business units for their actual cloud usage.

Example

```text
Engineering

↓

$4,500

Finance

↓

$1,200

Marketing

↓

$900
```

---

# Benefits

- Accountability
- Budget ownership
- Cost awareness
- Responsible cloud usage

---

# Showback

Definition

Showback reports costs without billing departments directly.

Example

```text
Monthly Report

↓

Engineering

↓

$5,000

↓

Information Only
```

---

# Chargeback vs Showback

| Feature | Chargeback | Showback |
|----------|------------|----------|
| Billing | Yes | No |
| Accountability | High | Medium |
| Financial Impact | Direct | Informational |
| Budget Ownership | Yes | Partial |

---

# Business Unit Reporting

Example

```text
Finance

↓

Engineering

↓

Marketing

↓

HR

↓

Operations
```

---

# Project-Level Reporting

Example

```text
Payments

Authentication

Analytics

Platform

DevOps
```

---

# Environment Reporting

Track separately

- Production
- Development
- Testing
- Staging
- Sandbox

---

# Cost Ownership

Every resource should have

- Owner
- Team
- Department
- Business Unit
- Project

---

# Budget Governance

Review

- Monthly budgets
- Department budgets
- Team budgets
- Project budgets

---

# Approval Workflow

```text
Engineer

↓

Resource Request

↓

Manager Approval

↓

Provision

↓

Tag Resources

↓

Monitor Costs
```

---

# Resource Ownership Matrix

| Resource | Owner |
|----------|-------|
| EC2 | DevOps |
| RDS | DBA |
| EKS | Platform Team |
| S3 | Application Team |
| CloudFront | Networking Team |
| Lambda | Development Team |

---

# Executive Dashboard

Display

- Monthly spend
- Forecast
- Budget status
- Top services
- Top departments
- Savings achieved
- Cost anomalies
- Optimization opportunities

---

# FinOps KPIs

Track

- Total cloud spend
- Cost per application
- Cost per customer
- Cost per deployment
- Budget variance
- Savings achieved
- Resource utilization
- Idle resource percentage
- Tag compliance
- RI utilization
- Savings Plans utilization

---

# Governance Reviews

Daily

- Cost anomalies
- Budget alerts

---

Weekly

- Resource ownership
- New resources
- Untagged resources

---

Monthly

- Executive reporting
- Optimization review
- Forecast updates
- Budget adjustments

---

Quarterly

- FinOps maturity assessment
- Architecture review
- Purchasing strategy
- Governance updates

---

# Enterprise Governance Workflow

```text
Provision

↓

Tag Resources

↓

Allocate Costs

↓

Generate Reports

↓

Review KPIs

↓

Optimize Resources

↓

Improve Governance
```

---

# FinOps Maturity

Level 1

```text
Basic Visibility
```

---

Level 2

```text
Budget Management
```

---

Level 3

```text
Chargeback

Automation

Optimization
```

---

Level 4

```text
Predictive Optimization

Business Integration
```

---

# Common Governance Issues

- Missing tags
- Unknown resource owners
- Budget overruns
- Idle infrastructure
- Duplicate environments
- Lack of accountability
- No optimization reviews
- Poor reporting

---

# Enterprise FinOps Checklist

## Visibility

- Cost Explorer enabled
- CUR enabled
- Executive dashboards
- Budget reports

---

## Governance

- Tag policy
- Budget policy
- Approval workflow
- Resource ownership

---

## Optimization

- Monthly reviews
- Savings Plans review
- RI review
- Idle resource cleanup

---

## Accountability

- Department reports
- Project reports
- Cost ownership
- KPI tracking

---

# Best Practices

- Assign an owner to every AWS resource.
- Standardize mandatory cost allocation tags.
- Review tag compliance weekly.
- Use Showback before implementing Chargeback.
- Separate costs by environment, project, and department.
- Create executive dashboards for cloud spending.
- Measure FinOps KPIs consistently.
- Establish monthly optimization review meetings.
- Combine engineering and finance reviews for cloud costs.
- Continuously improve governance processes.

---

# Summary

This section covered FinOps principles, chargeback, showback, cost allocation, tagging governance, budget governance, cost ownership, executive reporting, FinOps KPIs, governance workflows, and enterprise cost management frameworks. These practices help organizations build financial accountability, optimize cloud spending, and align engineering decisions with business objectives.

---

# Cost Optimization Automation

---

# Introduction

Automation reduces cloud costs by automatically detecting, scheduling, optimizing, and removing unnecessary resources.

Benefits

- Lower operational costs
- Reduced manual effort
- Consistent governance
- Faster remediation
- Continuous optimization

---

# Automation Workflow

```text
Monitor

↓

Detect Waste

↓

Trigger Automation

↓

Optimize Resource

↓

Notify Team

↓

Verify Savings
```

---

# Amazon EventBridge Scheduling

---

# Common Schedules

Examples

```text
08:00

Start Development Environment

↓

19:00

Stop Development Environment

↓

Weekend

Shutdown Non-Production

↓

Month End

Generate Cost Report
```

---

# AWS Lambda Automation

Common Tasks

- Stop idle EC2 instances
- Start development servers
- Delete unattached EBS volumes
- Remove old snapshots
- Clean ECR repositories
- Delete unused AMIs

---

# Systems Manager Automation

Automate

- Start instances
- Stop instances
- Patch systems
- Run scripts
- Collect inventory
- Restart services

---

# Instance Scheduler

Use Cases

- Development
- QA
- Testing
- Training
- Sandbox

Workflow

```text
Business Hours

↓

Start Instances

↓

Working Hours

↓

Stop Instances

↓

Savings
```

---

# Auto Scaling Schedules

Example

```text
Business Hours

Minimum Capacity = 5

↓

Night

Minimum Capacity = 2

↓

Weekend

Minimum Capacity = 1
```

---

# Lambda Cleanup Automation

Examples

Automatically delete

- Old snapshots
- Detached volumes
- Old AMIs
- Unused ECR images
- Expired backups

---

# Idle Resource Detection

Monitor

- CPU utilization
- Network traffic
- Memory usage
- Disk activity

Identify

- Idle EC2
- Idle RDS
- Idle Load Balancers
- Idle NAT Gateways

---

# CloudWatch Automation

Workflow

```text
CloudWatch Alarm

↓

EventBridge

↓

Lambda

↓

Stop Resource

↓

Notify Team
```

---

# EBS Cleanup

Review

- Available volumes
- Unattached volumes
- Old snapshots

Automation

```text
Identify

↓

Notify

↓

Approval

↓

Delete
```

---

# S3 Lifecycle Automation

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

↓

Delete
```

---

# ECR Lifecycle Automation

Automatically remove

- Untagged images
- Images older than 90 days
- Images exceeding retention limit

---

# CloudWatch Logs Automation

Automatically

- Set retention periods
- Archive logs
- Delete expired log groups

---

# Backup Lifecycle Automation

Automatically

- Create backups
- Archive backups
- Delete expired backups

---

# AWS Backup Automation

Policies

- Daily backup
- Weekly backup
- Monthly backup
- Yearly archive

---

# Resource Expiration

Apply Tags

```text
Owner

Environment

ExpiryDate

Project
```

Workflow

```text
Expiry Date

↓

Lambda Check

↓

Notification

↓

Approval

↓

Delete Resource
```

---

# Budget Automation

Workflow

```text
Budget Threshold

↓

SNS

↓

Lambda

↓

Restrict IAM

↓

Stop Resources
```

---

# Cost Anomaly Automation

Workflow

```text
Cost Anomaly

↓

SNS

↓

Lambda

↓

Create Incident

↓

Notify Teams
```

---

# Auto Scaling Optimization

Automatically

- Increase capacity during peak hours
- Decrease capacity during low demand
- Replace Spot interruptions
- Launch right-sized instances

---

# Karpenter Automation

Workflow

```text
Pending Pods

↓

Karpenter

↓

Launch Right-Sized Node

↓

Run Workload

↓

Terminate Idle Node
```

---

# Spot Automation

Automatically

- Detect interruption notice
- Launch replacement
- Drain Kubernetes nodes
- Resume workload

---

# Intelligent Recommendations

Sources

- Trusted Advisor
- Compute Optimizer
- Cost Explorer
- Cost Anomaly Detection

Workflow

```text
Recommendation

↓

Review

↓

Approve

↓

Automate Change
```

---

# Infrastructure Scheduling

Examples

Development

```text
08:00–19:00
```

---

QA

```text
08:00–18:00
```

---

Training

```text
On Demand
```

---

Production

```text
24 × 7
```

---

# Monthly Cleanup Workflow

```text
Review Resources

↓

Unused Resources

↓

Approval

↓

Delete

↓

Generate Report
```

---

# Cost Optimization Playbook

Step 1

Review Cost Explorer

↓

Step 2

Review Trusted Advisor

↓

Step 3

Review Compute Optimizer

↓

Step 4

Review Budgets

↓

Step 5

Automate Cleanup

↓

Step 6

Measure Savings

---

# Enterprise Automation Pipeline

```text
CloudWatch

↓

EventBridge

↓

Lambda

↓

Systems Manager

↓

AWS APIs

↓

Optimization

↓

SNS Notification
```

---

# KPIs

Track

- Resources scheduled
- Resources cleaned
- Monthly savings
- Automation success rate
- Manual interventions
- Resource utilization
- Idle resource count
- Cost reduction percentage

---

# Common Automation Targets

Compute

- EC2
- ECS
- EKS

---

Storage

- EBS
- S3
- Snapshots

---

Database

- RDS
- Aurora

---

Networking

- NAT Gateway
- Elastic IP
- Load Balancer

---

Monitoring

- CloudWatch Logs
- Dashboards
- Metrics

---

# Best Practices

- Automate shutdown of non-production environments.
- Schedule development resources based on business hours.
- Use Lambda for lightweight cleanup tasks.
- Automate snapshot and backup lifecycle management.
- Apply expiration tags to temporary resources.
- Use EventBridge for scheduled automation.
- Integrate Cost Anomaly Detection with automated notifications.
- Review automation success metrics regularly.
- Require approval for destructive actions in production.
- Continuously improve automation based on operational feedback.

---

# Summary

This section covered automation techniques for AWS cost optimization using EventBridge, Lambda, Systems Manager, Auto Scaling, lifecycle policies, cleanup automation, intelligent recommendations, and enterprise scheduling strategies. These automation patterns reduce manual effort, improve governance, and deliver continuous cost savings across AWS environments.

---

# Production Cost Optimization

---

# Introduction

Production cost optimization focuses on reducing AWS spending while maintaining:

- High Availability
- Security
- Performance
- Scalability
- Reliability

The objective is **maximize business value**, not simply minimize cost.

---

# Startup Cost Optimization

Architecture

```text
CloudFront

↓

Application Load Balancer

↓

Auto Scaling Group

↓

RDS

↓

Amazon S3
```

Optimization

- Compute Savings Plans
- Spot Instances for CI/CD
- S3 Lifecycle Policies
- CloudFront caching
- gp3 EBS volumes
- Lambda for scheduled tasks

Result

```text
30–50%

Infrastructure Savings
```

---

# Enterprise Cost Optimization

Architecture

```text
AWS Organizations

↓

Management Account

↓

Shared Services

↓

Development

↓

Testing

↓

Production
```

Optimization

- Consolidated Billing
- Cost Allocation Tags
- Chargeback
- Showback
- CUR
- Budgets
- Savings Plans

---

# Kubernetes (Amazon EKS)

Architecture

```text
Users

↓

ALB

↓

Amazon EKS

↓

Managed Node Groups

↓

RDS
```

Optimization

- Karpenter
- Cluster Autoscaler
- Spot Worker Nodes
- Graviton Instances
- Resource Requests/Limits
- Namespace Quotas
- Remove idle clusters

Example

```text
Before

20 Nodes

↓

After

12 Nodes

↓

40% Lower Compute Cost
```

---

# ECS Cost Optimization

Optimization

- ECS Fargate Spot
- Auto Scaling
- Right-sized Tasks
- Image cleanup
- Service scaling

---

# Lambda Optimization

Review

- Memory allocation
- Duration
- Cold starts
- Package size

Optimization

```text
512 MB

↓

256 MB

↓

Lower Cost
```

---

# Database Optimization

RDS

- Reserved Instances
- Storage Autoscaling
- Right sizing

Aurora

- Reader optimization
- Serverless v2
- Auto Pause (where applicable)

DynamoDB

- Auto Scaling
- TTL
- On-Demand billing

---

# Storage Optimization

Amazon S3

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

Amazon EBS

Optimization

- gp2 → gp3
- Delete unattached volumes
- Archive snapshots

---

Amazon EFS

Optimization

- Lifecycle Management
- Intelligent Tiering

---

# Networking Optimization

Review

- NAT Gateway
- Data Transfer
- Transit Gateway
- Elastic IP
- Load Balancer

Optimization

- Gateway Endpoints
- Interface Endpoints
- CloudFront
- Consolidate ALBs

---

# Logging Optimization

Review

- CloudWatch Logs
- Log Retention
- OpenSearch Storage

Optimization

- Retention Policies
- Archive Logs
- Delete obsolete Log Groups

---

# Multi-Account Cost Management

Architecture

```text
Management

↓

Security

↓

Networking

↓

Development

↓

Production
```

Optimization

- Consolidated Billing
- Budget per Account
- Department Reporting
- Cost Categories

---

# Multi-Region Optimization

Review

- Cross-region replication
- Data transfer
- Duplicate resources
- Disaster Recovery

Optimization

- Remove unnecessary replicas
- Right-size standby environments
- Review replication policies

---

# Development Environment Scheduling

Workflow

```text
08:00

Start

↓

19:00

Stop

↓

Weekend Shutdown

↓

Monthly Savings
```

---

# Batch Processing

Recommended

```text
Spot Fleet

↓

Batch Jobs

↓

Automatic Scaling

↓

Lower Cost
```

---

# CI/CD Cost Optimization

Review

- Build frequency
- Build duration
- Runner size
- Artifact retention

Optimization

- Spot runners
- Auto-clean artifacts
- Right-size build agents
- Remove unused pipelines

---

# Monitoring Cost Optimization

Review

- Metrics
- Dashboards
- Logs
- Alarms

Optimization

- Reduce retention
- Remove unused dashboards
- Delete obsolete metrics

---

# Backup Cost Optimization

Review

- Snapshot retention
- Cross-region copies
- Duplicate backups

Optimization

- Lifecycle Policies
- Archive
- Delete expired backups

---

# Monthly Cost Review Workflow

```text
Billing

↓

Cost Explorer

↓

Trusted Advisor

↓

Compute Optimizer

↓

Budgets

↓

Optimization Plan

↓

Implementation
```

---

# Cost Optimization KPIs

Track

- Monthly AWS spend
- Cost per application
- Cost per deployment
- Savings achieved
- Resource utilization
- Idle resources
- Savings Plans coverage
- RI utilization
- Budget variance
- Cost anomalies

---

# ROI Example

Investment

```text
Savings Plan

↓

$10,000 Commitment
```

Savings

```text
$18,000 Annual Savings

↓

Positive ROI
```

---

# Common Architecture Patterns

## Production Web Application

```text
CloudFront

↓

WAF

↓

ALB

↓

Auto Scaling

↓

RDS Multi-AZ

↓

S3
```

---

## Event-Driven Architecture

```text
EventBridge

↓

Lambda

↓

SQS

↓

SNS
```

---

## Container Platform

```text
ALB

↓

Amazon EKS

↓

Spot Nodes

↓

Managed Node Groups

↓

Amazon ECR
```

---

## Analytics Platform

```text
Amazon S3

↓

Glue

↓

Athena

↓

QuickSight
```

---

# Common Cost Optimization Mistakes

- Leaving development environments running 24×7
- Purchasing Savings Plans without usage analysis
- Ignoring Cost Explorer recommendations
- No tagging strategy
- Large CloudWatch log retention
- Idle NAT Gateways
- Overprovisioned RDS instances
- Keeping unattached EBS volumes
- Running everything On-Demand
- Ignoring Spot opportunities

---

# Enterprise Cost Review Checklist

## Daily

- Cost anomalies
- Budget alerts
- Unexpected spikes

---

## Weekly

- Idle resources
- Tag compliance
- Trusted Advisor findings

---

## Monthly

- Savings Plans review
- RI utilization
- Executive reports
- Architecture review
- Cleanup activities

---

## Quarterly

- Purchasing strategy
- FinOps review
- Multi-account governance
- Disaster Recovery cost review

---

# Best Practices

- Optimize based on utilization data, not assumptions.
- Review Cost Explorer and Trusted Advisor regularly.
- Combine Savings Plans with Spot Instances where appropriate.
- Schedule non-production resources.
- Apply lifecycle policies to storage services.
- Track KPIs and measure realized savings.
- Review architecture as workloads evolve.
- Automate repetitive optimization tasks.
- Maintain strong tagging and governance policies.
- Treat cost optimization as a continuous operational process.

---

# Summary

This section presented production cost optimization case studies, architecture patterns, real-world optimization scenarios, ROI examples, operational KPIs, and enterprise review processes. These patterns demonstrate how organizations can systematically reduce AWS costs while maintaining secure, scalable, and highly available production environments.

---

