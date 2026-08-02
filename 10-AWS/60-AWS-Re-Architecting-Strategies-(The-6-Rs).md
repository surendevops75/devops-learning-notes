# AWS Re-Architecting Strategies (The 6 Rs)

---

# Introduction

When organizations migrate workloads to AWS, not every application should be migrated using the same approach. AWS recommends evaluating each application and selecting the most appropriate migration strategy based on business goals, technical requirements, cost, complexity, and modernization objectives.

These migration approaches are collectively known as the **6 Rs of Migration**.

The six strategies are

- Rehost
- Replatform
- Refactor
- Repurchase
- Retire
- Retain

These strategies help organizations optimize migration planning and accelerate cloud adoption while minimizing business risk.

---

# What are the 6 Rs?

The 6 Rs define different approaches for migrating applications to AWS.

Workflow

```text
Application Assessment

↓

Business Analysis

↓

Technical Analysis

↓

Choose Strategy

↓

AWS Migration
```

Each application may use a different migration strategy.

---

# Why Use the 6 Rs?

Without a migration strategy

```text
Applications

↓

Same Migration Method

↓

Higher Risk

↓

Higher Cost
```

Problems

- Incorrect Migration Approach
- Increased Downtime
- Higher Cost
- Delayed Projects
- Poor Cloud Adoption

With the 6 Rs

```text
Application

↓

Assessment

↓

Best Strategy

↓

Successful Migration
```

---

# The 6 Rs Overview

```text
Application

↓

Assessment

│

├── Rehost

├── Replatform

├── Refactor

├── Repurchase

├── Retire

└── Retain
```

---

# Rehost (Lift and Shift)

Rehost moves an application to AWS with minimal or no code changes.

Characteristics

- Fastest Migration
- Low Risk
- Minimal Changes
- Infrastructure Modernization

Example

```text
VMware VM

↓

AWS MGN

↓

Amazon EC2
```

Suitable for

- Legacy Applications
- Large Data Centers
- Quick Cloud Adoption

Advantages

- Fast Migration
- Low Complexity
- Minimal Downtime

Disadvantages

- Limited Cloud Optimization
- Existing Technical Debt Remains

---

# Replatform (Lift, Tinker, and Shift)

Replatform makes small optimizations while migrating.

Examples

```text
EC2

↓

Amazon RDS

------------

Self-Managed Redis

↓

Amazon ElastiCache
```

Suitable for

- Applications requiring minor modernization
- Database modernization
- Managed services adoption

Advantages

- Better Performance
- Reduced Operational Overhead
- Moderate Migration Effort

Disadvantages

- Some Application Changes Required

---

# Refactor (Re-Architect)

Refactoring redesigns applications to fully utilize cloud-native services.

Examples

```text
Monolith

↓

Microservices

↓

Amazon EKS

↓

AWS Lambda
```

Cloud Services

- Lambda
- Amazon EKS
- Amazon ECS
- DynamoDB
- API Gateway
- EventBridge

Advantages

- Maximum Scalability
- Cloud Native
- Higher Resilience
- Better Performance

Disadvantages

- Highest Cost
- Longest Migration Time
- Significant Development Effort

---

# Repurchase

Repurchase replaces an existing application with a SaaS solution.

Example

```text
On-Prem CRM

↓

Salesforce
```

Other Examples

- Microsoft 365
- ServiceNow
- Workday

Advantages

- Reduced Maintenance
- Automatic Updates
- Faster Deployment

Disadvantages

- Licensing Costs
- Vendor Dependency

---

# Retire

Applications that are no longer required are removed.

Example

```text
Legacy Reporting Server

↓

Business Review

↓

Retire Application
```

Benefits

- Lower Costs
- Reduced Complexity
- Smaller Migration Scope

---

# Retain

Some applications remain in the current environment.

Reasons

- Regulatory Requirements
- Technical Constraints
- Licensing Restrictions
- Pending Business Decisions

Example

```text
Mainframe

↓

Remain On-Premises
```

Retain may be temporary until future modernization.

---

# Choosing the Right Strategy

| Requirement | Recommended Strategy |
|-------------|----------------------|
| Fast Migration | Rehost |
| Small Improvements | Replatform |
| Cloud-Native Modernization | Refactor |
| Replace with SaaS | Repurchase |
| Remove Application | Retire |
| Keep On-Premises | Retain |

---

# Migration Decision Workflow

```text
Application

↓

Business Value?

↓

Yes

↓

Modernize?

↓

Yes

↓

Refactor

------------

No

↓

Quick Migration?

↓

Yes

↓

Rehost

↓

No

↓

Replatform
```

---

# AWS Services Supporting the 6 Rs

| Strategy | AWS Services |
|-----------|--------------|
| Rehost | AWS MGN |
| Replatform | AWS DMS, Amazon RDS |
| Refactor | EKS, ECS, Lambda, API Gateway |
| Repurchase | SaaS Applications |
| Retire | Application Portfolio Review |
| Retain | Hybrid Architecture |

---

# Enterprise Migration Architecture

```text
Application Portfolio

          │

          ▼

Portfolio Assessment

          │

 ┌────────┼─────────────┐

 │        │             │

Business Technical Compliance

          │

          ▼

      6 Rs Strategy

          │

 ┌────────┼────────────┐

 │        │            │

MGN      DMS     Cloud-Native

          │

          ▼

      AWS Cloud
```

---

# AWS CLI

The 6 Rs are migration planning strategies and do not provide AWS CLI commands.

CLI commands are available for the migration services used during implementation, such as AWS MGN and AWS DMS.

---

# Terraform

Terraform is commonly used after migration to provision AWS infrastructure using Infrastructure as Code.

---

# CloudFormation

CloudFormation helps automate infrastructure deployment following migration.

---

# Python (Boto3)

The 6 Rs are planning methodologies and do not expose Boto3 APIs.

Boto3 automates AWS services used after migration.

---

# Best Practices

- Assess every application individually
- Align migration strategy with business objectives
- Avoid using one strategy for every workload
- Prioritize business-critical applications
- Use AWS Migration Hub for tracking
- Modernize applications where business value exists
- Retire unused applications before migration
- Validate dependencies before migration
- Test applications after migration
- Document migration decisions
- Review migration strategies regularly
- Optimize workloads after migration

---

# Common Mistakes

- Using Lift and Shift for every application
- Ignoring application dependencies
- Refactoring without business justification
- Migrating unused applications
- No migration assessment
- Missing rollback plans
- Ignoring licensing implications
- No post-migration optimization
- Delayed modernization planning
- Poor stakeholder communication

---

# Troubleshooting

## Migration Strategy Unclear

Check

- Business Requirements
- Technical Complexity
- Compliance Requirements
- Application Dependencies

---

## Migration Taking Too Long

Verify

- Selected Strategy
- Project Scope
- Resource Availability
- Migration Planning

---

## High Migration Cost

Check

- Refactoring Scope
- Unnecessary Application Migration
- Licensing Costs
- Infrastructure Sizing

---

## Application Performance Issues After Migration

Verify

- Resource Sizing
- Network Latency
- Architecture Changes
- Monitoring Configuration

---

## Legacy Application Cannot Be Migrated

Consider

- Retain Strategy
- Hybrid Architecture
- Future Modernization
- Business Requirements

---

# Interview Questions

## Basic

1. What are the AWS 6 Rs of migration?
2. What is Rehost?
3. What is Replatform?
4. What is Refactor?
5. What is Repurchase?
6. What is Retire?
7. What is Retain?
8. Which strategy is called Lift and Shift?
9. Which strategy uses SaaS?
10. Which strategy keeps applications on-premises?

---

## Intermediate

11. Explain Rehost vs Replatform.
12. Explain Replatform vs Refactor.
13. Explain migration decision criteria.
14. Explain business value assessment.
15. Explain cloud modernization.
16. Explain migration prioritization.
17. Explain application dependency analysis.
18. Explain hybrid migration strategies.
19. Explain migration governance.
20. Explain migration optimization.

---

## Advanced

21. Design a migration strategy for 500 enterprise applications.
22. How would you classify applications using the 6 Rs?
23. Explain Refactor vs Repurchase.
24. Design cloud modernization architecture.
25. Explain business-driven migration planning.
26. Design hybrid cloud migration.
27. Explain migration best practices.
28. Design enterprise migration governance.
29. Explain post-migration optimization.
30. Best practices for large-scale AWS migrations.

---

# Production Scenarios

### Scenario 1

A company needs to migrate 1,200 virtual machines to AWS within six months with minimal application changes.

Which migration strategy would you recommend and why?

---

### Scenario 2

A legacy monolithic application requires better scalability and faster deployments.

Which migration strategy is most appropriate?

---

### Scenario 3

An organization replaces its on-premises CRM with Salesforce.

Which of the 6 Rs does this represent?

---

### Scenario 4

An application is no longer used by the business but still consumes infrastructure resources.

Which migration strategy should be selected?

---

### Scenario 5

A highly regulated application must remain in the organization's data center.

Which migration strategy best fits this requirement?

---

### Scenario 6

A company wants to move from self-managed MySQL on EC2 to Amazon RDS MySQL with minimal application changes.

Would you recommend Rehost, Replatform, or Refactor? Explain your choice.

---

# Cheat Sheet

| Strategy | Purpose |
|-----------|---------|
| Rehost | Lift and Shift |
| Replatform | Minor Cloud Optimization |
| Refactor | Cloud-Native Modernization |
| Repurchase | Replace with SaaS |
| Retire | Remove Unused Applications |
| Retain | Keep Existing Environment |
| AWS MGN | Rehost |
| AWS DMS | Replatform |
| Amazon EKS/Lambda | Refactor |
| Migration Hub | Track Migration Progress |

---

# Summary

The AWS 6 Rs of Migration provide a structured framework for selecting the most appropriate cloud migration strategy for each application. By evaluating business objectives, technical complexity, compliance requirements, and modernization goals, organizations can choose between Rehost, Replatform, Refactor, Repurchase, Retire, or Retain to achieve efficient, low-risk, and business-aligned cloud migrations while maximizing the benefits of AWS.