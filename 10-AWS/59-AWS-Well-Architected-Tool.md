# AWS Well-Architected Tool

---

# Introduction

AWS Well-Architected Tool is a fully managed service that helps organizations review workloads against the AWS Well-Architected Framework, identify architectural risks, and implement AWS best practices.

Building workloads in the cloud requires continuous evaluation to ensure they remain secure, reliable, high-performing, cost-efficient, operationally excellent, and sustainable. The AWS Well-Architected Tool simplifies this process by providing structured workload assessments and improvement recommendations.

The tool evaluates workloads across the six pillars of the AWS Well-Architected Framework

- Operational Excellence
- Security
- Reliability
- Performance Efficiency
- Cost Optimization
- Sustainability

It integrates with

- AWS Well-Architected Framework
- AWS Trusted Advisor
- AWS Organizations
- AWS Control Tower
- AWS Security Hub
- AWS Config
- AWS CloudTrail
- Amazon CloudWatch

---

# What is AWS Well-Architected Tool?

AWS Well-Architected Tool evaluates AWS workloads against AWS architectural best practices.

It helps organizations

- Assess Workloads
- Identify Architectural Risks
- Improve Reliability
- Optimize Costs
- Strengthen Security
- Track Continuous Improvements

Workflow

```text
AWS Workload

↓

Well-Architected Tool

↓

Assessment

↓

Recommendations

↓

Improved Architecture
```

---

# Why Use the Well-Architected Tool?

Without Well-Architected Reviews

```text
Production Workload

↓

Architecture Drift

↓

Hidden Risks

↓

Operational Issues
```

Problems

- Security Weaknesses
- High Operational Cost
- Poor Performance
- Reliability Risks
- Limited Governance

With Well-Architected Tool

```text
AWS Workload

↓

Architecture Review

↓

Improvement Plan

↓

Continuous Optimization
```

---

# Real World Problem Statement

A global enterprise operates

- 350 AWS Applications
- Multi-Region Infrastructure
- Kubernetes Clusters
- Hundreds of AWS Accounts

Requirements

- Architecture Reviews
- Risk Assessment
- Cost Optimization
- Security Compliance
- Continuous Improvement

AWS Well-Architected Tool provides standardized workload reviews.

---

# Enterprise Architecture

```text
AWS Workloads

EC2

Lambda

EKS

RDS

      │

      ▼

AWS Well-Architected Tool

      │

Assessment

      │

 ┌────────┼─────────────┐

 │        │             │

Reports Risks Improvement Plans
```

---

# Core Components

AWS Well-Architected Tool consists of

- Workloads
- Lenses
- Milestones
- Workload Reviews
- High Risk Issues (HRIs)
- Improvement Plans
- Reports
- Review Sharing
- Continuous Assessments

---

# Workloads

A workload represents an application or system being evaluated.

Examples

- E-Commerce Platform
- Banking Application
- Kubernetes Platform
- Data Analytics Pipeline

Each workload is reviewed independently.

---

# Lenses

Lenses provide specialized review questions.

Available examples

- AWS Well-Architected Lens
- Serverless Lens
- SaaS Lens
- Machine Learning Lens
- Financial Services Lens
- IoT Lens

Organizations can also create custom lenses.

---

# Milestones

Milestones capture workload status at specific points in time.

Example

```text
Version 1

↓

Migration Complete

↓

Version 2

↓

Production Release
```

Supports architecture evolution tracking.

---

# Workload Review

Review process

```text
Create Workload

↓

Select Lens

↓

Answer Questions

↓

Risk Assessment

↓

Improvement Recommendations
```

---

# High Risk Issues (HRIs)

High Risk Issues identify serious architectural concerns.

Examples

- No Backup Strategy
- Public Database
- Missing Encryption
- Single Availability Zone
- No Monitoring

HRIs should be prioritized first.

---

# Medium Risk Issues (MRIs)

Medium Risk Issues identify improvements that should be addressed but are less critical than HRIs.

Examples

- Missing Tags
- Limited Monitoring
- Manual Processes

---

# Improvement Plans

The tool generates recommendations for resolving identified risks.

Typical workflow

```text
Risk Identified

↓

Recommendation

↓

Implementation

↓

Review

↓

Risk Closed
```

---

# Review Sharing

Organizations can share workload reviews with

- AWS Solutions Architects
- Internal Teams
- Consultants
- Auditors

Supports collaborative architecture reviews.

---

# Continuous Improvement

Reviews should be repeated

- After Major Releases
- Infrastructure Changes
- Security Reviews
- Cost Optimization Initiatives

Supports continuous architecture maturity.

---

# AWS CLI

List Workloads

```bash
aws wellarchitected list-workloads
```

List Lenses

```bash
aws wellarchitected list-lenses
```

Get Workload

```bash
aws wellarchitected get-workload \
--workload-id <workload-id>
```

---

# Terraform

AWS Well-Architected Tool currently has no native Terraform resources.

Terraform is commonly used to implement recommendations generated during reviews.

---

# CloudFormation

AWS CloudFormation does not currently provide native resources for the Well-Architected Tool.

---

# Python (Boto3)

```python
import boto3

wa = boto3.client("wellarchitected")

response = wa.list_workloads()

print(response)
```

---

# Enterprise Production Architecture

```text
          Enterprise Workloads

   EC2 • Lambda • EKS • RDS

                 │

                 ▼

     AWS Well-Architected Tool

                 │

      Reviews • Lenses • HRIs

                 │

  Reports • Improvement Plans

                 │

 Architecture Optimization Team
```

---

# Best Practices

- Review production workloads regularly
- Address High Risk Issues immediately
- Use the appropriate Well-Architected Lens
- Create milestones after major releases
- Perform reviews before production deployments
- Include architecture reviews in change management
- Share reviews with stakeholders
- Document improvement actions
- Reassess workloads periodically
- Integrate reviews into cloud governance
- Track progress using milestones
- Continuously improve architectures

---

# Common Mistakes

- Performing reviews only once
- Ignoring High Risk Issues
- Using incorrect lenses
- Skipping milestone creation
- No follow-up actions
- Not documenting improvements
- Ignoring operational changes
- Treating reviews as audits instead of continuous improvement
- No stakeholder participation
- Delaying remediation

---

# Troubleshooting

## Unable to Create Workload

Check

- IAM Permissions
- AWS Region
- Service Availability

---

## Review Incomplete

Verify

- All Questions Answered
- Correct Lens Selected
- Review Saved

---

## High Risk Issues Persist

Check

- Recommendations Implemented
- Follow-Up Review
- Architecture Changes

---

## Cannot Share Review

Verify

- IAM Permissions
- AWS Account Access
- Sharing Configuration

---

## API Access Failed

Check

- IAM Policy
- AWS CLI Configuration
- Region Settings

---

# Interview Questions

## Basic

1. What is the AWS Well-Architected Tool?
2. What problem does it solve?
3. What is a workload?
4. What is a lens?
5. What is a milestone?
6. What are High Risk Issues (HRIs)?
7. What are Medium Risk Issues (MRIs)?
8. What is a workload review?
9. Why are milestones important?
10. How does the Well-Architected Tool differ from the Well-Architected Framework?

---

## Intermediate

11. Explain workload reviews.
12. Explain lenses.
13. Explain architecture governance.
14. Explain milestone tracking.
15. Explain review sharing.
16. Explain continuous improvement.
17. Explain risk prioritization.
18. Explain custom lenses.
19. Explain architecture maturity.
20. Explain production review strategies.

---

## Advanced

21. Design an enterprise architecture review process.
22. How would you review 500 AWS workloads?
23. Explain Well-Architected Tool vs Trusted Advisor.
24. Design governance using architecture reviews.
25. Explain Well-Architected Tool integration with DevOps.
26. Design enterprise cloud architecture assessments.
27. Explain operational best practices.
28. Design architecture review workflows.
29. Explain continuous architecture optimization.
30. Best practices for AWS Well-Architected Tool adoption.

---

# Production Scenarios

### Scenario 1

Your organization deploys a new production application every month.

How would milestones help track architectural improvements?

---

### Scenario 2

A workload review identifies multiple High Risk Issues.

How would you prioritize remediation activities?

---

### Scenario 3

An enterprise wants specialized review questions for serverless applications.

Which Well-Architected Tool feature supports this?

---

### Scenario 4

A security audit requires evidence of periodic architecture reviews.

How would the Well-Architected Tool support this requirement?

---

### Scenario 5

Leadership wants architecture reviews integrated into the CI/CD release process.

How would you incorporate the Well-Architected Tool into the workflow?

---

### Scenario 6

Your organization manages hundreds of AWS applications.

How would the Well-Architected Tool standardize architecture governance?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Workload | Application Being Reviewed |
| Lens | Specialized Review Questions |
| Milestone | Architecture Snapshot |
| Workload Review | Best Practice Assessment |
| HRI | High Risk Issue |
| MRI | Medium Risk Issue |
| Improvement Plan | Risk Remediation |
| Review Sharing | Collaboration |
| Well-Architected Framework | Best Practice Guidance |
| Continuous Improvement | Ongoing Optimization |

---

# Summary

AWS Well-Architected Tool is a managed assessment service that helps organizations evaluate cloud workloads against the AWS Well-Architected Framework. Through workloads, lenses, milestones, High Risk Issues (HRIs), Medium Risk Issues (MRIs), improvement plans, and continuous assessments, the tool enables enterprises to standardize architecture reviews, strengthen governance, improve security and reliability, optimize costs, and continuously evolve cloud architectures using AWS best practices.