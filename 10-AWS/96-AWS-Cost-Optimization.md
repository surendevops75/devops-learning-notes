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

