# AWS Support Plans

---

# Introduction

AWS Support Plans provide organizations with technical support, architectural guidance, operational best practices, proactive recommendations, and faster response times for AWS workloads.

As organizations move from development to production, the need for reliable support increases. AWS offers multiple support plans designed for individuals, startups, enterprises, and mission-critical production environments.

AWS Support Plans provide access to

- Technical Support Engineers
- Architecture Guidance
- AWS Trusted Advisor
- AWS Health Dashboard
- Infrastructure Event Management
- Concierge Support
- Technical Account Manager (TAM)

Choosing the appropriate support plan helps improve operational efficiency, reduce downtime, and accelerate issue resolution.

---

# What are AWS Support Plans?

AWS Support Plans are subscription-based support services that provide varying levels of technical assistance and operational guidance.

AWS offers five support plans

- Basic Support
- Developer Support
- Business Support
- Enterprise On-Ramp Support
- Enterprise Support

Each plan includes different features, response times, and support channels.

---

# Why AWS Support Plans?

Without AWS Support

```text
Production Issue

↓

Self Troubleshooting

↓

Long Downtime

↓

Business Impact
```

Problems

- Slow Issue Resolution
- Limited Guidance
- No Architectural Reviews
- Operational Risks

With AWS Support

```text
Production Issue

↓

AWS Support

↓

Technical Assistance

↓

Quick Resolution
```

---

# AWS Support Plan Comparison

| Feature | Basic | Developer | Business | Enterprise On-Ramp | Enterprise |
|----------|-------|-----------|----------|--------------------|------------|
| Billing Support | Yes | Yes | Yes | Yes | Yes |
| Technical Support | No | Yes | Yes | Yes | Yes |
| Trusted Advisor Core Checks | Yes | Yes | Yes | Yes | Yes |
| Full Trusted Advisor | No | No | Yes | Yes | Yes |
| AWS Health Dashboard | Personal | Personal | Organizational | Organizational | Organizational |
| Technical Account Manager | No | No | No | Pool of TAMs | Dedicated TAM |
| Concierge Support | No | No | No | Yes | Yes |
| Infrastructure Event Management | No | No | Optional | Included | Included |

---

# Basic Support

Basic Support is included with every AWS account.

Features

- Customer Service
- Billing Support
- Documentation
- Whitepapers
- AWS Health Dashboard
- Trusted Advisor Core Checks

Suitable for

- Learning AWS
- Personal Projects
- Small Test Environments

---

# Developer Support

Developer Support is designed for development and testing environments.

Features

- Email Support
- Business Hours Access
- General Architecture Guidance
- Technical Assistance

Recommended for

- Individual Developers
- Small Development Teams
- Test Workloads

---

# Business Support

Business Support is intended for production workloads.

Features

- 24×7 Technical Support
- Phone Support
- Chat Support
- Full Trusted Advisor
- AWS Health Dashboard
- Third-Party Software Guidance

Suitable for

- Production Applications
- Medium to Large Organizations

---

# Enterprise On-Ramp Support

Enterprise On-Ramp helps organizations scale cloud operations.

Features

- Pool of Technical Account Managers
- Concierge Support
- Architecture Guidance
- Proactive Reviews
- Faster Response Times

Suitable for

- Growing Enterprises
- Critical Business Applications

---

# Enterprise Support

Enterprise Support is the highest AWS support tier.

Features

- Dedicated Technical Account Manager (TAM)
- Infrastructure Event Management
- Operations Reviews
- Architecture Reviews
- Concierge Team
- 24×7 Critical Support
- Proactive Guidance

Suitable for

- Large Enterprises
- Mission-Critical Applications
- Regulated Industries

---

# Technical Account Manager (TAM)

A TAM provides proactive technical guidance.

Responsibilities

- Architecture Reviews
- Operational Reviews
- Best Practices
- Escalation Management
- Cost Optimization Recommendations

Acts as a trusted technical advisor.

---

# Concierge Support

Concierge Support assists with

- Billing Questions
- Account Management
- Licensing
- AWS Marketplace
- Cost Inquiries

Available with Enterprise plans.

---

# Infrastructure Event Management (IEM)

IEM provides operational guidance before and during planned events.

Examples

- Product Launches
- Black Friday Sales
- Large Migrations
- Seasonal Traffic Peaks

Workflow

```text
Planned Event

↓

AWS Support

↓

Readiness Review

↓

Live Monitoring

↓

Post-Event Review
```

---

# AWS Trusted Advisor Integration

Support plans determine Trusted Advisor access.

Core Checks

- Available for all plans

Full Checks

- Business
- Enterprise On-Ramp
- Enterprise

---

# AWS Health Dashboard

Provides visibility into AWS service events affecting your resources.

Information includes

- Scheduled Maintenance
- Service Degradation
- Outages
- Security Notifications

---

# Case Severity Levels

Support cases are categorized by severity.

Levels

- General Guidance
- System Impaired
- Production System Impaired
- Production System Down
- Business-Critical System Down

Higher severity cases receive faster response targets.

---

# Response Time Targets

Typical initial response targets

| Severity | Developer | Business | Enterprise On-Ramp | Enterprise |
|----------|-----------|----------|--------------------|------------|
| General Guidance | < 24 Business Hours | < 24 Hours | < 24 Hours | < 24 Hours |
| System Impaired | N/A | < 12 Hours | < 4 Hours | < 4 Hours |
| Production System Impaired | N/A | < 4 Hours | < 2 Hours | < 1 Hour |
| Business-Critical System Down | N/A | < 1 Hour | < 30 Minutes | < 15 Minutes |

---

# AWS CLI

AWS Support APIs are available for eligible support plans.

Describe Services

```bash
aws support describe-services
```

Describe Severity Levels

```bash
aws support describe-severity-levels
```

Create Support Case

```bash
aws support create-case
```

---

# Terraform

AWS Support Plans are account-level subscriptions and cannot be managed using Terraform resources.

---

# CloudFormation

There are currently no native CloudFormation resources for AWS Support Plans.

---

# Python (Boto3)

```python
import boto3

support = boto3.client("support", region_name="us-east-1")

response = support.describe_services()

print(response)
```

---

# Enterprise Production Architecture

```text
      AWS Workloads

             │

             ▼

     AWS Support Plan

             │

 ┌───────────┼────────────┐

 │           │            │

Support Engineers TAM Concierge

 │           │            │

Architecture Reviews Trusted Advisor

             │

     Business Continuity
```

---

# Best Practices

- Choose a support plan based on workload criticality
- Use Business Support for production workloads
- Schedule architecture reviews regularly
- Review Trusted Advisor recommendations
- Monitor AWS Health Dashboard
- Open support cases early during incidents
- Use Infrastructure Event Management for major launches
- Engage your TAM proactively
- Maintain updated contact information
- Document support case history
- Review operational readiness periodically

---

# Common Mistakes

- Using Basic Support for production systems
- Waiting too long before opening support cases
- Ignoring Trusted Advisor recommendations
- Not reviewing AWS Health notifications
- Underestimating business impact
- Skipping architecture reviews
- Not using TAM guidance
- Missing event readiness planning
- Incomplete support case information
- Delayed escalation

---

# Troubleshooting

## Unable to Create Support Case

Check

- Support Plan Eligibility
- IAM Permissions
- AWS Account Status

---

## Trusted Advisor Features Missing

Verify

- Active Support Plan
- Eligible Checks
- Account Permissions

---

## Slow Case Response

Check

- Selected Severity
- Support Plan
- Case Details

---

## AWS Health Dashboard Empty

Verify

- AWS Region
- Service Impact
- Account Scope

---

## Concierge Support Unavailable

Check

- Support Plan Level
- Account Eligibility

---

# Interview Questions

## Basic

1. What are AWS Support Plans?
2. Name the five AWS Support Plans.
3. What is included in Basic Support?
4. What is Business Support?
5. What is Enterprise Support?
6. What is a Technical Account Manager (TAM)?
7. What is Concierge Support?
8. What is Infrastructure Event Management?
9. What is AWS Health Dashboard?
10. What is AWS Trusted Advisor?

---

## Intermediate

11. Explain the differences between Business and Enterprise Support.
12. Which support plan includes a dedicated TAM?
13. Explain support case severity levels.
14. Explain Infrastructure Event Management.
15. Explain Trusted Advisor access across support plans.
16. Explain AWS Health integration.
17. Explain Concierge Support.
18. Explain response time targets.
19. Explain production support best practices.
20. Explain architecture reviews.

---

## Advanced

21. Design an enterprise AWS support strategy.
22. Which support plan would you recommend for a financial institution and why?
23. Explain the role of a TAM during cloud migration.
24. Design operational readiness for a Black Friday event.
25. Explain Enterprise On-Ramp vs Enterprise Support.
26. How would you handle a production outage using AWS Support?
27. Explain proactive operational guidance.
28. Design support processes for multi-account AWS environments.
29. Explain best practices for enterprise support operations.
30. When should an organization upgrade its AWS Support Plan?

---

# Production Scenarios

### Scenario 1

Your production application experiences a critical outage at midnight.

Which AWS Support plan would provide the fastest assistance?

---

### Scenario 2

Your organization is preparing for a major product launch expecting 10x normal traffic.

How would Infrastructure Event Management help reduce operational risk?

---

### Scenario 3

A company wants proactive architecture reviews and cost optimization guidance.

Which AWS Support feature provides these services?

---

### Scenario 4

Your finance team needs help with AWS billing and Marketplace subscriptions.

Which AWS Support capability should they use?

---

### Scenario 5

Your organization manages hundreds of production AWS accounts.

Which support plan would provide the most comprehensive operational assistance?

---

### Scenario 6

Trusted Advisor reports several high-risk security findings.

How would AWS Support help prioritize and remediate these recommendations?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Basic Support | Free Foundational Support |
| Developer Support | Development & Testing |
| Business Support | Production Workloads |
| Enterprise On-Ramp | Growing Enterprises |
| Enterprise Support | Mission-Critical Environments |
| TAM | Proactive Technical Guidance |
| Concierge | Billing & Account Assistance |
| IEM | Planned Event Readiness |
| AWS Health Dashboard | Service Health Notifications |
| Trusted Advisor | Best Practice Recommendations |

---

# Summary

AWS Support Plans provide organizations with scalable technical support, architectural guidance, operational best practices, and proactive assistance based on workload criticality. From Basic Support for learning environments to Enterprise Support with a dedicated Technical Account Manager, Concierge Support, Infrastructure Event Management, and full Trusted Advisor access, AWS Support Plans help organizations improve reliability, accelerate incident resolution, and operate production workloads with confidence.