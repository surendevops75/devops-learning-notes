# AWS Service Quotas

---

# Introduction

AWS Service Quotas is a fully managed service that helps organizations view, monitor, request, and manage AWS service quotas (previously called service limits) across AWS accounts and Regions.

Every AWS service has default quotas to protect AWS infrastructure and ensure fair resource usage. As organizations grow, these quotas may become bottlenecks that prevent deployments or scaling.

AWS Service Quotas provides centralized visibility into quota usage and allows organizations to request quota increases before limits impact production workloads.

AWS Service Quotas integrates with

- Amazon EC2
- Amazon VPC
- Amazon EBS
- Amazon RDS
- AWS Lambda
- Amazon EKS
- Amazon ECS
- AWS Organizations
- Amazon CloudWatch
- Amazon EventBridge
- AWS Trusted Advisor
- AWS Support

It is an essential governance and capacity planning service for enterprise AWS environments.

---

# What is AWS Service Quotas?

AWS Service Quotas manages AWS service limits.

It helps organizations

- View Service Quotas
- Monitor Usage
- Request Increases
- Prevent Deployment Failures
- Improve Capacity Planning

Workflow

```text
AWS Resources

↓

Current Usage

↓

Service Quotas

↓

Monitoring

↓

Quota Increase
```

---

# Why Service Quotas?

Without Service Quotas

```text
Application Growth

↓

Quota Reached

↓

Deployment Failure

↓

Production Impact
```

Problems

- Deployment Failures
- Scaling Issues
- Manual Monitoring
- Unexpected Limits
- Business Downtime

With Service Quotas

```text
AWS Resources

↓

Quota Monitoring

↓

Alerts

↓

Quota Increase

↓

Successful Scaling
```

---

# Real World Problem Statement

An enterprise operates

- 1,500 EC2 Instances
- Hundreds of VPCs
- Thousands of EBS Volumes
- Multi-Region Infrastructure

Requirements

- Capacity Planning
- Quota Monitoring
- Automated Alerts
- Fast Quota Requests

AWS Service Quotas provides centralized quota management.

---

# Enterprise Architecture

```text
AWS Resources

EC2  VPC  EBS  Lambda

          │

          ▼

AWS Service Quotas

          │

Quota Monitoring

          │

 ┌────────┼──────────┐

 │        │          │

CloudWatch EventBridge AWS Support

 │        │          │

Alerts Automation Quota Increase
```

---

# Core Components

AWS Service Quotas consists of

- Service Quotas
- Applied Quotas
- Default Quotas
- Usage Metrics
- Quota Requests
- CloudWatch Integration
- EventBridge Integration
- Organization Support
- Templates
- Notifications

---

# Default Quotas

Default Quotas are the initial limits assigned by AWS.

Examples

- EC2 Instances
- VPCs
- EBS Volumes
- Elastic IPs

Some quotas are adjustable, while others are fixed.

---

# Applied Quotas

Applied Quotas are the active quota values after approved quota increase requests.

Example

```text
Default EC2 Quota

↓

1,000 Instances

↓

Requested Increase

↓

5,000 Instances
```

---

# Adjustable vs Non-Adjustable Quotas

Adjustable

- EC2 Instance Limits
- Elastic IP Addresses
- VPC Limits

Non-Adjustable

- Certain AWS service internal limits
- Service-specific architectural constraints

Always verify whether a quota supports increases.

---

# Quota Usage

Service Quotas displays

- Current Usage
- Maximum Quota
- Utilization Percentage

Example

```text
Quota

1000 EC2 Instances

↓

Current Usage

920

↓

92% Utilized
```

---

# Quota Increase Requests

Organizations can request increases for supported quotas.

Workflow

```text
Current Usage

↓

Quota Limit

↓

Request Increase

↓

AWS Review

↓

Approved
```

---

# Quota Templates

Quota Templates automatically request approved quota values for newly created AWS accounts in an organization.

Benefits

- Consistent Capacity
- Standardized Limits
- Reduced Manual Work

Useful with AWS Organizations.

---

# AWS Organizations Integration

Organizations support centralized quota management.

Benefits

- Standard Quotas
- Multi-Account Visibility
- Automatic Templates

---

# CloudWatch Integration

CloudWatch monitors quota utilization.

Examples

- EC2 Usage
- VPC Usage
- Lambda Concurrency

CloudWatch alarms notify administrators before limits are reached.

---

# EventBridge Integration

EventBridge automates quota-related workflows.

Example

```text
Quota Reaches 90%

↓

EventBridge

↓

Lambda

↓

SNS

↓

Operations Team
```

---

# Trusted Advisor Integration

Trusted Advisor identifies service quota risks and recommends quota increases when appropriate.

Together, Trusted Advisor and Service Quotas help prevent capacity-related issues.

---

# AWS Support Integration

Quota increase requests are processed through AWS Support for supported services.

Response time depends on

- AWS Service
- Support Plan
- Requested Quota

---

# AWS CLI

List Services

```bash
aws service-quotas list-services
```

List Service Quotas

```bash
aws service-quotas list-service-quotas \
--service-code ec2
```

Request Quota Increase

```bash
aws service-quotas request-service-quota-increase \
--service-code ec2 \
--quota-code L-1216C47A \
--desired-value 5000
```

---

# Terraform

AWS Service Quotas currently has limited Terraform support.

Example

```hcl
resource "aws_servicequotas_service_quota" "ec2" {

  service_code = "ec2"

  quota_code   = "L-1216C47A"

  value        = 5000

}
```

---

# CloudFormation

AWS CloudFormation has limited native support for Service Quotas.

Organizations typically manage quota requests using the AWS CLI or SDKs.

---

# Python (Boto3)

```python
import boto3

sq = boto3.client("service-quotas")

response = sq.list_service_quotas(

    ServiceCode="ec2"

)

print(response)
```

---

# Enterprise Production Architecture

```text
         AWS Resources

 EC2  VPC  Lambda  RDS  EKS

              │

              ▼

      AWS Service Quotas

              │

 Usage Metrics • Quota Templates

              │

 ┌────────────┼─────────────┐

 │            │             │

CloudWatch EventBridge AWS Support

 │            │             │

Alerts Automation Quota Requests
```

---

# Best Practices

- Monitor quota utilization regularly
- Create CloudWatch alarms for critical quotas
- Request increases before reaching limits
- Use quota templates with AWS Organizations
- Review service quotas during architecture planning
- Monitor growth trends
- Document approved quota increases
- Test scaling against quota limits
- Enable EventBridge automation
- Coordinate quota planning with application teams
- Review Trusted Advisor recommendations
- Include quota reviews in production readiness checks

---

# Common Mistakes

- Discovering quota limits during production deployments
- Ignoring CloudWatch quota alarms
- Waiting until quotas are fully consumed
- No quota planning for new Regions
- Manual quota tracking
- No organization-wide quota standards
- Ignoring Trusted Advisor warnings
- Not documenting quota increases
- Assuming all quotas are adjustable
- No capacity planning process

---

# Troubleshooting

## Quota Increase Request Pending

Check

- AWS Support Case
- Requested Value
- Service Eligibility
- Support Plan

---

## Deployment Failed Due to Quota

Verify

- Current Usage
- Applied Quota
- AWS Region
- Resource Type

---

## CloudWatch Alarm Not Triggering

Check

- Metric Configuration
- Alarm Threshold
- Evaluation Period
- Service Metrics

---

## Quota Template Not Applied

Verify

- AWS Organizations
- Template Configuration
- New Account Status

---

## Service Quota Missing

Check

- AWS Region
- Supported Service
- IAM Permissions

---

# Interview Questions

## Basic

1. What is AWS Service Quotas?
2. What are Service Quotas?
3. What is the difference between Default and Applied Quotas?
4. Which quotas can be increased?
5. How do you request a quota increase?
6. What are Quota Templates?
7. How does CloudWatch integrate with Service Quotas?
8. How does EventBridge integrate with Service Quotas?
9. How does AWS Organizations support quota management?
10. What is the role of AWS Support?

---

## Intermediate

11. Explain quota monitoring.
12. Explain quota templates.
13. Explain CloudWatch quota alarms.
14. Explain EventBridge automation.
15. Explain organization-wide quota governance.
16. Explain quota planning.
17. Explain Trusted Advisor integration.
18. Explain regional quotas.
19. Explain capacity planning.
20. Explain production quota management.

---

## Advanced

21. Design enterprise quota governance.
22. How would you prevent EC2 quota failures during scaling?
23. Explain Service Quotas vs Trusted Advisor.
24. Design automated quota monitoring.
25. Explain quota management across AWS Organizations.
26. Design quota planning for global applications.
27. Explain operational best practices.
28. Design capacity planning workflows.
29. Explain quota monitoring architecture.
30. Best practices for AWS Service Quotas.

---

# Production Scenarios

### Scenario 1

Your Auto Scaling Group cannot launch additional EC2 instances because the account quota has been reached.

How would AWS Service Quotas help resolve this issue?

---

### Scenario 2

Your organization creates new AWS accounts every week.

How would Quota Templates ensure consistent quota values?

---

### Scenario 3

Leadership wants alerts whenever EC2 utilization exceeds 85% of the account quota.

How would CloudWatch and Service Quotas implement this?

---

### Scenario 4

A production deployment fails because the VPC quota has been exceeded.

How would you troubleshoot and prevent this issue?

---

### Scenario 5

An enterprise expands into five new AWS Regions.

How would you validate that all required service quotas are sufficient before deployment?

---

### Scenario 6

Operations teams need centralized quota governance across hundreds of AWS accounts.

How would AWS Organizations support this requirement?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Default Quota | Initial AWS Limit |
| Applied Quota | Active Approved Limit |
| Quota Usage | Current Resource Consumption |
| Quota Request | Increase Service Limit |
| Quota Template | Standardize Limits for New Accounts |
| CloudWatch | Quota Monitoring |
| EventBridge | Automation |
| AWS Organizations | Multi-Account Governance |
| AWS Support | Quota Increase Approval |
| Trusted Advisor | Quota Recommendations |

---

# Summary

AWS Service Quotas is a centralized service for managing AWS service limits across accounts and Regions. Features such as quota monitoring, applied quotas, quota increase requests, quota templates, CloudWatch integration, EventBridge automation, AWS Organizations support, and AWS Support integration enable enterprises to proactively manage capacity, prevent deployment failures, standardize quota management, and support large-scale AWS environments.