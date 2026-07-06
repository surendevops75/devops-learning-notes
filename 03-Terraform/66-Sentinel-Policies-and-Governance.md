# Sentinel Policies and Governance

## Introduction

As organizations grow, infrastructure management becomes more challenging.

Problems:

```text
Developers Creating Public Resources

Missing Tags

Wrong Instance Types

Deployments In Restricted Regions

Security Violations
```

---

Even though Terraform automates infrastructure:

```text
Terraform Does Not Automatically Enforce Company Standards
```

---

Organizations need:

```text
Governance

Compliance

Policy Enforcement
```

---

Terraform provides:

```text
Sentinel
```

for this purpose.

---

# What is Sentinel?

Sentinel is:

```text
Policy As Code Framework
```

provided by HashiCorp.

---

Sentinel allows organizations to define:

```text
Rules

Security Standards

Compliance Policies

Governance Controls
```

using code.

---

# Why Sentinel?

Without Sentinel:

```text
Engineers Can Deploy Anything
```

---

Examples:

```text
Public S3 Buckets

Large EC2 Instances

Unapproved Regions

Databases Without Encryption
```

---

Result:

```text
Security Risks

Compliance Violations

Increased Costs
```

---

# Solution

Before Terraform Apply:

```text
Evaluate Policies
```

---

If policies pass:

```text
Deployment Allowed
```

---

If policies fail:

```text
Deployment Blocked
```

---

# Sentinel Architecture

```text
Terraform Plan
        ↓
Sentinel Policy Check
        ↓
Pass
        ↓
Apply
```

---

Or

```text
Terraform Plan
        ↓
Sentinel Policy Check
        ↓
Fail
        ↓
Apply Blocked
```

---

# What is Policy as Code?

Policy as Code means:

```text
Governance Rules Written As Code
```

instead of:

```text
Manual Reviews

Spreadsheets

Documents
```

---

Benefits:

```text
Automation

Consistency

Scalability

Compliance
```

---

# Example Organization Policy

Company Standard:

```text
Every Resource Must Have Tags
```

---

Required Tags:

```text
Environment

Owner

Project
```

---

Without Sentinel:

```text
Engineers May Forget Tags
```

---

With Sentinel:

```text
Deployment Blocked
```

until tags are added.

---

# Example Policy

Rule:

```text
Every EC2 Instance Must Have Owner Tag
```

---

Valid:

```text
Owner = Surendra
```

---

Invalid:

```text
No Owner Tag
```

---

Result:

```text
Policy Failure
```

---

# Approved Instance Types

Organization wants:

```text
Only t3.micro

Only t3.small
```

---

Not Allowed:

```text
m7i.8xlarge

c6i.12xlarge
```

---

Reason:

```text
Cost Control
```

---

Policy:

```text
Only Approved Instance Types
```

---

# Approved Regions

Organization operates only in:

```text
us-east-1

us-west-2
```

---

Developer attempts:

```text
ap-south-1
```

---

Sentinel:

```text
Blocks Deployment
```

---

# Public S3 Bucket Prevention

Policy:

```text
No Public S3 Buckets
```

---

Developer:

```text
Creates Public Bucket
```

---

Result:

```text
Policy Failure
```

---

Apply:

```text
Blocked
```

---

# Database Encryption Policy

Organization Standard:

```text
All Databases Must Be Encrypted
```

---

Valid:

```text
Encrypted RDS
```

---

Invalid:

```text
Unencrypted RDS
```

---

Result:

```text
Deployment Denied
```

---

# Security Group Policy

Policy:

```text
No Open SSH Access
```

---

Allowed:

```text
10.0.0.0/16
```

---

Blocked:

```text
0.0.0.0/0
```

on:

```text
Port 22
```

---

# Environment Governance

Policy:

```text
Production Resources
```

must have:

```text
Backups

Monitoring

Encryption
```

---

Sentinel validates:

```text
Infrastructure Standards
```

before deployment.

---

# Soft Mandatory Policy

Result:

```text
Warning
```

---

Apply can continue.

---

Example:

```text
Tag Recommendation
```

---

# Hard Mandatory Policy

Result:

```text
Deployment Blocked
```

---

Example:

```text
Public Database
```

---

# Advisory Policy

Result:

```text
Informational Only
```

---

No enforcement.

---

Example:

```text
Cost Recommendation
```

---

# Sentinel Workflow

```text
Developer Pushes Code
        ↓
Terraform Plan
        ↓
Sentinel Evaluation
        ↓
Pass
        ↓
Apply
```

---

# Failed Workflow

```text
Developer Pushes Code
        ↓
Terraform Plan
        ↓
Sentinel Evaluation
        ↓
Fail
        ↓
Apply Blocked
```

---

# Production Example

Developer creates:

```text
Public RDS Database
```

---

Terraform Plan:

```text
Generated Successfully
```

---

Sentinel Check:

```text
Database Is Public
```

---

Result:

```text
Policy Violation
```

---

Apply:

```text
Blocked
```

---

# Governance Benefits

```text
Consistency

Compliance

Security

Cost Control

Automation

Reduced Human Error
```

---

# Common Policies in Enterprises

```text
Mandatory Tags

Approved Regions

Approved Instance Types

Encryption Required

No Public Databases

No Public Storage

Cost Limits

Backup Requirements
```

---

# Sentinel and Terraform Enterprise

Sentinel is commonly used with:

```text
Terraform Enterprise

Terraform Cloud
```

---

Integration:

```text
Terraform Plan
       ↓
Sentinel
       ↓
Policy Validation
       ↓
Apply
```

---

# Real World Example

Banking Organization Policy:

```text
No Public Resources

Mandatory Encryption

Mandatory Tags

Approved Regions Only
```

---

Sentinel automatically enforces:

```text
Compliance Standards
```

for every deployment.

---

# Common Interview Questions

## What is Sentinel?

### Short Answer

Sentinel is HashiCorp's Policy as Code framework used for governance and compliance.

### Detailed Explanation

Sentinel evaluates Terraform plans against organizational policies before infrastructure changes are applied.

### Production Example

Blocking creation of public S3 buckets.

### Follow-Up Questions

- What is Policy as Code?
- Is Sentinel used before or after apply?

---

## Why is Sentinel Used?

### Short Answer

To enforce security, compliance, and governance policies automatically.

### Detailed Explanation

Sentinel prevents infrastructure deployments that violate organizational standards.

### Production Example

Allowing only approved EC2 instance types.

### Follow-Up Questions

- Can Sentinel block deployments?
- Can Sentinel enforce tagging?

---

## What is Policy as Code?

### Short Answer

Policy as Code means governance rules are defined and enforced through code.

### Detailed Explanation

Policies become automated, version-controlled, and consistently enforced.

### Production Example

Mandatory encryption requirements for all databases.

### Follow-Up Questions

- Why is Policy as Code better than manual reviews?
- Can policies be version controlled?

---

## What Types of Policies Are Commonly Implemented?

### Short Answer

Security, compliance, tagging, cost, and governance policies.

### Detailed Explanation

Organizations use Sentinel to ensure infrastructure follows approved standards.

### Production Example

Preventing public RDS databases and enforcing mandatory tags.

### Follow-Up Questions

- Can Sentinel enforce regions?
- Can Sentinel enforce instance types?

---

# Key Takeaways

```text
Sentinel is HashiCorp's Policy as Code framework.

Sentinel evaluates Terraform plans before apply.

Policies help enforce governance and compliance.

Organizations use Sentinel for security, cost control, and standardization.

Common policies include mandatory tags, encryption, approved regions, and approved instance types.

Sentinel can block deployments that violate policies.

Policy as Code improves consistency and automation.

Sentinel is commonly used with Terraform Cloud and Terraform Enterprise.

Large enterprises rely on Sentinel for infrastructure governance.

Understanding Sentinel is important for senior DevOps and platform engineering roles.
```