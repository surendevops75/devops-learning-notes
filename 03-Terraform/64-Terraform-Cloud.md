# Terraform Cloud

## Introduction

As Terraform adoption grows, teams face challenges managing infrastructure from local machines.

Problems:

```text
State Files On Laptops

No Centralized Execution

Difficult Team Collaboration

No Approval Process

No Governance
```

---

Terraform Cloud solves these problems by providing:

```text
Managed Terraform Platform
```

---

# What is Terraform Cloud?

Terraform Cloud is a SaaS platform provided by HashiCorp for managing Terraform workflows.

---

It provides:

```text
Remote State Storage

Remote Execution

Team Collaboration

VCS Integration

Policy Enforcement

Cost Estimation
```

---

# Traditional Terraform Workflow

```text
Developer Laptop
       ↓
terraform apply
       ↓
AWS Infrastructure
```

---

Problems:

```text
Who Ran The Apply?

Which Version Was Used?

State File Location?

No Audit Trail
```

---

# Terraform Cloud Workflow

```text
Developer
      ↓
Git Push
      ↓
Terraform Cloud
      ↓
Plan
      ↓
Approval
      ↓
Apply
      ↓
Cloud Provider
```

---

# Key Components

```text
Organizations

Projects

Workspaces

Runs

Variables

State Files
```

---

# Organization

Top-level container.

Example:

```text
MyCompany
```

---

Contains:

```text
Projects

Workspaces

Users
```

---

# Projects

Used to organize workspaces.

Example:

```text
Infrastructure

Networking

Kubernetes

Applications
```

---

# Workspaces

A workspace represents:

```text
One Terraform Configuration
```

---

Example:

```text
dev-vpc

prod-vpc

dev-eks

prod-eks
```

---

# Runs

A Run is:

```text
Plan + Apply Execution
```

---

Triggered by:

```text
Git Push

Manual Execution

API Call
```

---

# Remote State Storage

Without Terraform Cloud:

```text
terraform.tfstate
```

stored locally.

---

With Terraform Cloud:

```text
State Stored Securely
```

inside Terraform Cloud.

---

Benefits:

```text
Centralized

Versioned

Secure

Highly Available
```

---

# Remote Execution

Normally:

```text
terraform apply
```

runs on laptop.

---

Terraform Cloud:

```text
Runs Terraform On HashiCorp Infrastructure
```

---

Flow:

```text
Git Push
      ↓
Terraform Cloud
      ↓
Plan
      ↓
Apply
```

---

# Version Control Integration

Terraform Cloud integrates with:

```text
GitHub

GitLab

Bitbucket

Azure DevOps
```

---

# Example

Developer pushes code:

```text
GitHub Repository
```

---

Terraform Cloud automatically:

```text
Runs Plan
```

---

# Workflow

```text
Git Push
     ↓
Terraform Cloud
     ↓
terraform plan
     ↓
Review
     ↓
terraform apply
```

---

# Variables

Terraform Cloud stores:

```text
Terraform Variables

Environment Variables

Secrets
```

---

Example:

```text
AWS_ACCESS_KEY_ID

AWS_SECRET_ACCESS_KEY

Environment

Region
```

---

# Sensitive Variables

Sensitive values can be marked as:

```text
Sensitive
```

---

Hidden from:

```text
Logs

UI

Outputs
```

---

# Team Collaboration

Multiple engineers can work together.

Example:

```text
DevOps Team

Platform Team

Cloud Team
```

---

All use:

```text
Single Workspace
```

---

# Audit Trail

Terraform Cloud records:

```text
Who Ran Changes

When Changes Occurred

What Changed
```

---

Useful for:

```text
Compliance

Auditing

Troubleshooting
```

---

# Cost Estimation

Terraform Cloud can estimate:

```text
Infrastructure Cost
```

before deployment.

---

Example:

```text
EC2

RDS

Load Balancers
```

---

Before Apply:

```text
Estimated Monthly Cost
```

displayed.

---

# Run Approval Workflow

Example:

```text
Developer Creates Change
```

---

Terraform Cloud:

```text
Generates Plan
```

---

Manager Reviews:

```text
Approve
```

---

Terraform Cloud:

```text
Runs Apply
```

---

# Architecture

```text
Developer
     ↓
Git Repository
     ↓
Terraform Cloud
     ↓
Plan
     ↓
Approval
     ↓
Apply
     ↓
AWS
```

---

# Production Example

Organization:

```text
Roboshop
```

---

Workspaces:

```text
dev-vpc

prod-vpc

dev-eks

prod-eks
```

---

Developer pushes code.

---

Terraform Cloud:

```text
Runs Plan

Shows Changes

Requests Approval

Executes Apply
```

---

No local execution required.

---

# Terraform Cloud vs Local Terraform

Local Terraform:

```text
Runs On Laptop

Local State

Manual Process
```

---

Terraform Cloud:

```text
Remote Execution

Centralized State

Collaboration

Governance
```

---

# Benefits

```text
Centralized Management

Secure State Storage

Team Collaboration

Remote Execution

Audit Trails

Approval Workflows

Cost Estimation

VCS Integration
```

---

# Common Interview Questions

## What is Terraform Cloud?

### Short Answer

Terraform Cloud is a managed platform for running and managing Terraform workflows.

### Detailed Explanation

It provides remote execution, state management, collaboration, policy enforcement, and VCS integration.

### Production Example

Managing AWS infrastructure through GitHub-integrated Terraform Cloud workspaces.

### Follow-Up Questions

- Does Terraform Cloud store state?
- Does Terraform Cloud execute Terraform remotely?

---

## What is a Workspace in Terraform Cloud?

### Short Answer

A workspace is a container for Terraform configuration, state, and execution.

### Detailed Explanation

Each workspace represents an infrastructure deployment and maintains its own state.

### Production Example

Separate workspaces for dev-vpc and prod-vpc.

### Follow-Up Questions

- How is it different from CLI workspaces?
- Can workspaces have separate variables?

---

## What is Remote Execution?

### Short Answer

Terraform runs on Terraform Cloud instead of a local machine.

### Detailed Explanation

Developers push code, and Terraform Cloud performs plan and apply operations remotely.

### Production Example

GitHub push automatically triggering Terraform Cloud runs.

### Follow-Up Questions

- Why is remote execution useful?
- Does it require local credentials?

---

## What are the Benefits of Terraform Cloud?

### Short Answer

Centralized state, collaboration, governance, and remote execution.

### Detailed Explanation

Terraform Cloud provides enterprise-grade workflows for managing infrastructure safely and consistently.

### Production Example

Multiple DevOps engineers managing AWS infrastructure through shared workspaces.

### Follow-Up Questions

- Can Terraform Cloud integrate with GitHub?
- Does it support approvals?

---

# Key Takeaways

```text
Terraform Cloud is HashiCorp's managed Terraform platform.

It provides remote execution and remote state storage.

Workspaces organize Terraform deployments.

GitHub, GitLab, and Bitbucket integration are supported.

State files are securely stored and versioned.

Sensitive variables can be protected.

Audit trails track infrastructure changes.

Approval workflows improve governance.

Terraform Cloud is widely used for team-based infrastructure management.

It removes the need to run Terraform from individual laptops.
```
````
