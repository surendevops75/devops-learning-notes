# AWS Trusted Advisor

---

# Introduction

AWS Trusted Advisor is a cloud optimization service that continuously analyzes AWS environments and provides recommendations to improve cost optimization, security, fault tolerance, performance, service limits, and operational excellence.

As AWS environments grow, organizations often deploy unused resources, misconfigure security settings, exceed service quotas, or miss opportunities to optimize infrastructure. AWS Trusted Advisor automatically evaluates AWS resources against AWS best practices and provides actionable recommendations.

AWS Trusted Advisor integrates with:

- Amazon EC2
- Amazon S3
- Amazon RDS
- Elastic Load Balancer (ALB/NLB)
- Amazon EBS
- Amazon CloudFront
- AWS IAM
- AWS Organizations
- Amazon CloudWatch
- Amazon EventBridge
- AWS Support Center

Trusted Advisor helps organizations reduce costs, improve security, increase reliability, and maintain operational best practices.

---

# What is AWS Trusted Advisor?

AWS Trusted Advisor continuously checks AWS resources and recommends improvements.

It evaluates

- Cost Optimization
- Security
- Fault Tolerance
- Performance
- Service Limits
- Operational Excellence

Workflow

```text
AWS Resources

↓

Trusted Advisor

↓

Best Practice Analysis

↓

Recommendations

↓

Optimization
```

---

# Why Trusted Advisor?

Without Trusted Advisor

```text
AWS Resources

↓

Manual Review

↓

Hidden Issues

↓

Higher Cost
```

Problems

- Idle Resources
- Security Risks
- Service Limit Issues
- Manual Optimization
- Performance Bottlenecks

With Trusted Advisor

```text
AWS Resources

↓

Trusted Advisor

↓

Recommendations

↓

Improved Environment
```

---

# Real World Problem Statement

An enterprise manages

- 2,000 EC2 Instances
- 500 RDS Databases
- Thousands of EBS Volumes
- Hundreds of Load Balancers
- Multiple AWS Accounts

Requirements

- Reduce Costs
- Improve Security
- Increase Availability
- Monitor Service Limits
- Follow AWS Best Practices

Trusted Advisor continuously evaluates the environment.

---

# Enterprise Architecture

```text
AWS Resources

EC2  RDS  EBS  IAM  S3

          │

          ▼

 AWS Trusted Advisor

          │

 Best Practice Checks

          │

 ┌────────┼──────────┐

 │        │          │

Dashboard EventBridge CloudWatch

 │        │          │

Email  Lambda  Operations
```

---

# Core Components

AWS Trusted Advisor consists of

- Cost Optimization Checks
- Security Checks
- Fault Tolerance Checks
- Performance Checks
- Service Limit Checks
- Operational Excellence Checks
- Recommendations
- Refresh Status
- Priority Alerts
- Organization View

---

# Cost Optimization

Trusted Advisor identifies unused or underutilized resources.

Examples

- Idle EC2 Instances
- Unattached EBS Volumes
- Idle Load Balancers
- Underutilized RDS Instances
- Idle Elastic IP Addresses

Helps reduce AWS costs.

---

# Security Checks

Security recommendations include

- Root Account MFA
- IAM Access Keys Rotation
- Public S3 Buckets
- Security Group Rules
- IAM Best Practices
- Exposed Resources

Improves security posture.

---

# Fault Tolerance Checks

Trusted Advisor verifies infrastructure resilience.

Examples

- Multi-AZ RDS
- Auto Scaling Configuration
- Backup Configuration
- Load Balancer Health
- EBS Snapshots

Helps improve availability.

---

# Performance Checks

Performance recommendations include

- High CPU Utilization
- Underutilized Instances
- Optimized Load Balancers
- Network Performance

Improves application responsiveness.

---

# Service Limits

Trusted Advisor monitors AWS service quotas.

Examples

- EC2 Limits
- VPC Limits
- EBS Limits
- Elastic IP Limits
- IAM Limits

Provides early warning before limits are reached.

---

# Operational Excellence

Operational Excellence checks include

- Resource Tagging
- Best Practice Compliance
- Configuration Recommendations
- Resource Organization

Supports governance.

---

# Check Status

Each check has a status.

Possible values

- Green (No Action Required)
- Yellow (Warning)
- Red (Action Required)

Helps prioritize remediation.

---

# Refresh Checks

Trusted Advisor checks can be refreshed manually or automatically depending on the check type.

Regular refresh ensures current recommendations.

---

# AWS Organizations Integration

Trusted Advisor supports organization-wide visibility.

Benefits

- Central Dashboard
- Multi-Account Recommendations
- Enterprise Governance

---

# CloudWatch Integration

Monitor

- Check Results
- Resource Health
- Service Quotas
- Optimization Trends

---

# EventBridge Integration

Trusted Advisor events trigger automation.

Workflow

```text
Critical Recommendation

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

# AWS CLI

Describe Checks

```bash
aws support describe-trusted-advisor-checks
```

Describe Check Result

```bash
aws support describe-trusted-advisor-check-result \
--check-id <check-id>
```

Refresh Check

```bash
aws support refresh-trusted-advisor-check \
--check-id <check-id>
```

---

# Terraform

Trusted Advisor has no dedicated Terraform resource.

It is typically accessed through AWS Support APIs and automation workflows.

---

# CloudFormation

There is currently no native CloudFormation resource for Trusted Advisor.

---

# Python (Boto3)

```python
import boto3

support = boto3.client("support", region_name="us-east-1")

response = support.describe_trusted_advisor_checks(

    language="en"

)

print(response)
```

---

# Enterprise Production Architecture

```text
      AWS Resources

EC2  IAM  RDS  S3  EBS

            │

            ▼

    AWS Trusted Advisor

            │

 Best Practice Evaluation

            │

 ┌──────────┼───────────┐

 │          │           │

Dashboard EventBridge CloudWatch

 │          │           │

SNS      Lambda   Operations Team
```

---

# Best Practices

- Review Trusted Advisor weekly
- Resolve Red recommendations immediately
- Monitor Service Quotas
- Remove idle resources
- Enable MFA for the root account
- Rotate IAM access keys regularly
- Review public S3 buckets
- Use Multi-AZ databases
- Enable organization-wide visibility
- Automate notifications using EventBridge
- Monitor cost optimization opportunities
- Include Trusted Advisor reviews in operational audits

---

# Common Mistakes

- Ignoring Red recommendations
- Never refreshing checks
- Leaving idle EC2 instances running
- Keeping unattached EBS volumes
- Not monitoring service quotas
- Public S3 buckets
- Missing MFA on root account
- Ignoring cost optimization opportunities
- Delayed remediation
- No governance process

---

# Troubleshooting

## Checks Not Refreshing

Check

- AWS Support Plan
- Refresh Availability
- Service Status

---

## Missing Recommendations

Verify

- Supported AWS Services
- AWS Region
- Account Permissions

---

## Service Quota Warning Missing

Check

- Trusted Advisor Refresh
- Service Quota Configuration
- AWS Support Plan

---

## Access Denied

Verify

- IAM Permissions
- AWS Support API Access

---

## EventBridge Automation Failed

Check

- Event Rule
- Lambda Permissions
- SNS Subscription
- Target Configuration

---

# Interview Questions

## Basic

1. What is AWS Trusted Advisor?
2. What problem does Trusted Advisor solve?
3. What are the six Trusted Advisor categories?
4. What is a Trusted Advisor check?
5. Explain Green, Yellow, and Red status.
6. What are Cost Optimization checks?
7. What are Security checks?
8. What are Fault Tolerance checks?
9. What are Service Limit checks?
10. Trusted Advisor vs AWS Config?

---

## Intermediate

11. Explain Performance checks.
12. Explain Operational Excellence checks.
13. Explain Service Quotas monitoring.
14. Explain EventBridge integration.
15. Explain CloudWatch integration.
16. Explain AWS Organizations integration.
17. Explain refresh checks.
18. Explain Support API integration.
19. Explain enterprise optimization.
20. Explain cost governance.

---

## Advanced

21. Design enterprise cloud optimization architecture.
22. How would you reduce AWS costs using Trusted Advisor?
23. Design automated remediation workflows.
24. Explain Trusted Advisor vs Compute Optimizer.
25. Explain governance using Trusted Advisor.
26. Design multi-account optimization.
27. Explain operational best practices.
28. How would you prioritize recommendations?
29. Explain enterprise cost optimization strategy.
30. Best practices for production Trusted Advisor deployments.

---

# Production Scenarios

### Scenario 1

Trusted Advisor reports multiple idle EC2 instances.

How would you validate and safely remove them?

---

### Scenario 2

Your root account does not have MFA enabled.

How would Trusted Advisor identify this issue and why is it critical?

---

### Scenario 3

A production AWS account is approaching its EC2 service quota.

How would Trusted Advisor help prevent deployment failures?

---

### Scenario 4

Several S3 buckets are publicly accessible.

How would Trusted Advisor help identify and prioritize remediation?

---

### Scenario 5

Your finance team asks for ways to reduce monthly AWS costs.

Which Trusted Advisor recommendations would you review first?

---

### Scenario 6

An enterprise manages 600 AWS accounts.

How would Trusted Advisor help maintain organization-wide best practices?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Cost Optimization | Reduce AWS Costs |
| Security Checks | Improve Security |
| Fault Tolerance | Increase Availability |
| Performance | Optimize Resources |
| Service Limits | Monitor Quotas |
| Operational Excellence | Governance |
| Check Status | Green / Yellow / Red |
| EventBridge | Automation |
| CloudWatch | Monitoring |
| AWS Organizations | Multi-Account Visibility |

---

# Summary

AWS Trusted Advisor is a cloud optimization service that continuously evaluates AWS resources against AWS best practices for cost optimization, security, fault tolerance, performance, service limits, and operational excellence. By providing actionable recommendations, organization-wide visibility, EventBridge integration, CloudWatch monitoring, and governance capabilities, Trusted Advisor helps enterprises reduce costs, improve security, enhance reliability, and maintain well-architected AWS environments.