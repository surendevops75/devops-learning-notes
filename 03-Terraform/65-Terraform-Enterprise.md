# Terraform Enterprise

## Introduction

As organizations grow, managing infrastructure becomes more complex.

Challenges:

```text
Hundreds Of Engineers

Thousands Of Resources

Multiple AWS Accounts

Compliance Requirements

Audit Requirements

Security Controls
```

---

Terraform Cloud solves many problems.

However, some organizations cannot use SaaS solutions due to:

```text
Security Policies

Compliance Requirements

Regulatory Restrictions

Data Residency Requirements
```

---

For such organizations, HashiCorp provides:

```text
Terraform Enterprise (TFE)
```

---

# What is Terraform Enterprise?

Terraform Enterprise is:

```text
Self-Hosted Terraform Cloud
```

---

It provides most Terraform Cloud features while running inside the organization's infrastructure.

---

# Terraform Enterprise Architecture

```text
Organization Infrastructure
           ↓
Terraform Enterprise
           ↓
AWS / Azure / GCP
```

---

Everything remains inside:

```text
Company Controlled Environment
```

---

# Terraform Cloud vs Terraform Enterprise

Terraform Cloud:

```text
Hosted By HashiCorp
```

---

Terraform Enterprise:

```text
Hosted By Customer
```

---

Terraform Cloud:

```text
SaaS Platform
```

---

Terraform Enterprise:

```text
Self Managed Platform
```

---

# Why Organizations Choose Terraform Enterprise

Reasons:

```text
Data Sovereignty

Compliance

Security

Private Networking

Internal Governance
```

---

# Industries Commonly Using TFE

```text
Banks

Insurance Companies

Government Organizations

Healthcare

Telecom
```

---

These industries often cannot allow infrastructure metadata outside their environment.

---

# Core Features

Terraform Enterprise provides:

```text
Remote State

Remote Execution

Private Module Registry

RBAC

SSO

Audit Logs

Policy Enforcement

Cost Estimation
```

---

# Remote State Management

State files stored centrally.

---

Benefits:

```text
Secure

Versioned

Highly Available
```

---

Architecture:

```text
Terraform Enterprise
         ↓
State Storage
```

---

# Remote Execution

Instead of running Terraform locally:

```text
Engineer
      ↓
Git Push
      ↓
Terraform Enterprise
      ↓
Plan
      ↓
Apply
```

---

# Role-Based Access Control (RBAC)

Large organizations need:

```text
Different Permission Levels
```

---

Examples:

```text
Admin

Platform Team

DevOps Team

Developers

Auditors
```

---

RBAC controls:

```text
Who Can View

Who Can Plan

Who Can Apply

Who Can Destroy
```

---

# Example

Developer:

```text
Can Run Plan
```

---

Platform Lead:

```text
Can Approve Apply
```

---

Administrator:

```text
Full Access
```

---

# Single Sign-On (SSO)

Terraform Enterprise integrates with:

```text
Active Directory

LDAP

Okta

Azure AD
```

---

Users login using:

```text
Corporate Credentials
```

---

Benefits:

```text
Centralized Authentication

Simplified User Management

Security
```

---

# Audit Logs

Every action is recorded.

Examples:

```text
Who Ran Plan

Who Approved Apply

Who Modified Variables

Who Deleted Workspace
```

---

Useful for:

```text
Compliance

Security Reviews

Troubleshooting
```

---

# Private Module Registry

Organizations often create:

```text
VPC Module

EKS Module

Security Group Module

ALB Module
```

---

Instead of storing modules everywhere:

Terraform Enterprise provides:

```text
Private Module Registry
```

---

Architecture:

```text
Terraform Enterprise
        ↓
Private Modules
        ↓
Teams Consume Modules
```

---

# Example

Platform Team creates:

```text
Standard VPC Module
```

---

Developers use:

```hcl
module "vpc" {
  source = "company/vpc/aws"
}
```

---

Benefits:

```text
Consistency

Reusability

Governance
```

---

# Policy Enforcement

Terraform Enterprise integrates with:

```text
Sentinel Policies
```

---

Examples:

```text
Mandatory Tags

Approved Regions

Approved Instance Types

No Public Databases
```

---

Policies execute before:

```text
Apply
```

---

# Cost Estimation

Terraform Enterprise can estimate:

```text
Infrastructure Cost
```

before deployment.

---

Example:

```text
EC2

RDS

EKS

Load Balancers
```

---

Engineers can see:

```text
Expected Monthly Cost
```

before approving changes.

---

# Workspace Management

Terraform Enterprise uses:

```text
Workspaces
```

to organize infrastructure.

---

Example:

```text
dev-vpc

prod-vpc

dev-eks

prod-eks
```

---

Each workspace contains:

```text
State

Variables

Runs

History
```

---

# VCS Integration

Supports:

```text
GitHub

GitLab

Bitbucket

Azure DevOps
```

---

Workflow:

```text
Git Push
      ↓
Terraform Enterprise
      ↓
Plan
      ↓
Approval
      ↓
Apply
```

---

# Private Networking

Terraform Enterprise can run:

```text
Inside Internal Network
```

---

Benefits:

```text
Private APIs

Private Repositories

Internal Services
```

---

No public internet access required.

---

# Production Example

Banking Organization

Requirements:

```text
No SaaS

Internal Authentication

Audit Trails

Strict Compliance
```

---

Solution:

```text
Terraform Enterprise
```

---

Benefits:

```text
Self Hosted

Private Networking

SSO

RBAC

Audit Logs
```

---

# Architecture Example

```text
Developer
     ↓
Git Repository
     ↓
Terraform Enterprise
     ↓
Policy Checks
     ↓
Approval
     ↓
Apply
     ↓
AWS
```

---

# Terraform Enterprise vs Terraform Cloud

Terraform Cloud:

```text
Managed By HashiCorp
```

---

Terraform Enterprise:

```text
Managed By Organization
```

---

Terraform Cloud:

```text
Fast Setup
```

---

Terraform Enterprise:

```text
Maximum Control
```

---

# Common Interview Questions

## What is Terraform Enterprise?

### Short Answer

Terraform Enterprise is the self-hosted version of Terraform Cloud.

### Detailed Explanation

It provides enterprise-grade Terraform capabilities while running inside an organization's infrastructure.

### Production Example

A bank running Terraform Enterprise inside its private data center.

### Follow-Up Questions

- Why use Terraform Enterprise instead of Terraform Cloud?
- Is Terraform Enterprise self-hosted?

---

## What is RBAC in Terraform Enterprise?

### Short Answer

RBAC controls what actions users can perform.

### Detailed Explanation

Role-Based Access Control allows organizations to restrict planning, applying, and administration privileges.

### Production Example

Developers can run plans while platform leads approve deployments.

### Follow-Up Questions

- Why is RBAC important?
- Can permissions be customized?

---

## What is a Private Module Registry?

### Short Answer

A centralized repository for internally approved Terraform modules.

### Detailed Explanation

Organizations publish reusable modules that teams can consume consistently.

### Production Example

A standard VPC module shared across all teams.

### Follow-Up Questions

- Why use private modules?
- How does it improve governance?

---

## Why Do Organizations Choose Terraform Enterprise?

### Short Answer

For compliance, security, governance, and self-hosting requirements.

### Detailed Explanation

Organizations needing full control over Terraform operations often deploy Terraform Enterprise internally.

### Production Example

Government or financial institutions with strict regulatory requirements.

### Follow-Up Questions

- Does Terraform Enterprise support SSO?
- Does it provide audit logs?

---

# Key Takeaways

```text
Terraform Enterprise is the self-hosted version of Terraform Cloud.

It is designed for large organizations with strict security and compliance requirements.

Terraform Enterprise supports remote state and remote execution.

RBAC controls user permissions.

SSO integrates with corporate identity providers.

Audit logs provide complete change tracking.

Private Module Registry enables module standardization.

Sentinel policies provide governance and compliance enforcement.

Terraform Enterprise supports VCS integrations and approval workflows.

It is commonly used by banks, governments, healthcare, and other regulated industries.
```