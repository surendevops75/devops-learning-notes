# GitHub Organizations And Teams

## Introduction

As companies grow, managing repositories using individual accounts becomes difficult.

---

Example

```text
10 Developers

50 Repositories

Multiple Teams
```

---

Problems

```text
Permission Management

Repository Ownership

Security Controls

Team Collaboration
```

---

Need Centralized Management.

---

GitHub Provides

```text
Organizations

Teams
```

---

# Problem Statement

Suppose Company Has

```text
Backend Team

Frontend Team

DevOps Team

Security Team
```

---

All Teams Need Access To

```text
GitHub Repositories
```

---

Managing Permissions Individually Becomes Difficult.

---

Need

```text
Centralized Access Control
```

---

# What Is A GitHub Organization?

A GitHub Organization Is:

```text
A Shared Account

For Teams And Repositories
```

---

Think Of It As:

```text
Company Workspace
```

---

Example

```text
microgate

netflix

google

microsoft
```

---

Organization Contains

```text
Repositories

Teams

Members

Policies

Security Controls
```

---

# Why Organizations Are Needed?

Without Organization

```text
Developer Owns Repository
```

---

Problems

```text
Developer Leaves Company

Access Lost

Ownership Problems
```

---

With Organization

```text
Company Owns Repository
```

---

Better Governance.

---

# Organization Structure

```text
Organization

├── Repositories
├── Teams
├── Members
├── Security Policies
└── Secrets
```

---

# What Are Teams?

Teams Are Groups Of Users.

---

Example

```text
DevOps Team

Backend Team

Frontend Team

Security Team
```

---

Instead Of Managing Users Individually

```text
Manage Team Permissions
```

---

# Why Teams Are Important?

Suppose

```text
20 Developers
```

need access to

```text
Terraform Repository
```

---

Instead Of

```text
Granting Access 20 Times
```

---

Grant Access To

```text
DevOps Team
```

---

Much Simpler.

---

# Creating Teams

Organization

↓

Teams

↓

Create Team

---

Example

```text
platform-team

devops-team

security-team
```

---

# Team Permissions

GitHub Supports

```text
Read

Triage

Write

Maintain

Admin
```

---

# Read

Can

```text
View Repository
```

---

Cannot

```text
Modify Code
```

---

# Triage

Can

```text
Manage Issues

Manage Discussions
```

---

Cannot

```text
Push Code
```

---

# Write

Can

```text
Push Code

Create Branches

Create Pull Requests
```

---

# Maintain

Can

```text
Manage Repository Settings

Manage Branches

Manage Teams
```

---

Without Full Admin Rights.

---

# Admin

Full Control.

---

Can

```text
Delete Repository

Manage Security

Manage Permissions
```

---

Use Carefully.

---

# Team Based Access Example

Repository

```text
roboshop-infra
```

---

Permissions

```text
DevOps Team → Write

Security Team → Read

Platform Team → Maintain
```

---

Access Managed Centrally.

---

# Nested Teams

GitHub Supports

```text
Parent Team

Child Team
```

---

Example

```text
Engineering

├── Backend

├── Frontend

└── DevOps
```

---

Simplifies Management.

---

# Repository Access Through Teams

Example

```text
Repository

↓

DevOps Team

↓

All Members Get Access
```

---

No Individual Assignment Needed.

---

# Organization Owners

Highest Privilege Users.

---

Can

```text
Manage Organization

Manage Teams

Manage Repositories

Manage Security
```

---

Usually

```text
Engineering Managers

Platform Leads
```

---

# Members

Regular Users Inside Organization.

---

Permissions Determined By

```text
Team Membership
```

---

# External Collaborators

Users Outside Organization.

---

Example

```text
Consultants

Vendors

Contractors
```

---

Can Be Given Access To

```text
Specific Repositories
```

---

Without Joining Organization.

---

# Organization Secrets

Shared Across Repositories.

---

Example

```text
SONAR_TOKEN

DOCKERHUB_TOKEN

SLACK_WEBHOOK
```

---

Used In Multiple Projects.

---

# Real Production Example

Organization

```text
microgate
```

---

Repositories

```text
roboshop-app

roboshop-infra

roboshop-eks

roboshop-argocd
```

---

Teams

```text
DevOps Team

Backend Team

Security Team
```

---

Permissions Managed At Team Level.

---

# Team Mentions

GitHub Supports

```text
@devops-team

@security-team
```

---

Example

```text
Please review this Terraform change

@devops-team
```

---

All Team Members Notified.

---

# Organizations And CODEOWNERS

Example

```text
terraform/ @devops-team

security/ @security-team
```

---

GitHub Automatically Assigns Reviewers.

---

# Organizations In Enterprise DevOps

Common Setup

```text
Organization

├── Application Repositories
├── Infrastructure Repositories
├── Kubernetes Repositories
├── GitHub Actions Repositories
└── Shared Templates
```

---

Provides

```text
Governance

Security

Scalability
```

---

# Useful Organization Features

```text
Teams

Permissions

Organization Secrets

Security Policies

Repository Templates

CODEOWNERS Integration
```

---

# Common Interview Questions

## What Is A GitHub Organization?

### Short Answer

A GitHub Organization is a shared account used to manage repositories, teams, and users centrally.

### Detailed Explanation

Organizations provide centralized ownership, access control, security, and collaboration features for teams and enterprises.

### Production Example

```text
Company

↓

Organization

↓

Repositories

↓

Teams
```

### Follow-Up Questions

```text
Why Use Organizations Instead Of Personal Accounts?

Who Owns Repositories?

How Are Permissions Managed?
```

---

## What Are GitHub Teams?

### Short Answer

Teams are groups of users used to manage permissions collectively.

### Detailed Explanation

Teams simplify repository access management by assigning permissions to groups instead of individual users.

### Production Example

```text
DevOps Team

↓

Repository Access
```

### Follow-Up Questions

```text
Can Teams Have Different Permissions?

What Are Nested Teams?

How Do Teams Work With CODEOWNERS?
```

---

## Difference Between Organization Owners And Members?

### Short Answer

Owners manage the organization, while members have permissions assigned through teams.

### Detailed Explanation

Owners have full administrative control. Members operate within the permissions granted to them.

### Production Example

```text
Owner

↓

Manage Organization
```

```text
Member

↓

Work In Repositories
```

### Follow-Up Questions

```text
How Many Owners Should Exist?

Can Members Create Repositories?

Can Owners Manage Security Policies?
```

---

## What Are Organization Secrets?

### Short Answer

Organization Secrets are encrypted values shared across multiple repositories.

### Detailed Explanation

They reduce duplication and allow centralized management of commonly used credentials.

### Production Example

```text
DockerHub Token

↓

Used By Multiple Repositories
```

### Follow-Up Questions

```text
Difference Between Repository And Organization Secrets?

Can Access Be Restricted?

When Should Organization Secrets Be Used?
```

---

# Production Scenarios

## Scenario 1

Need Central Repository Ownership.

### Solution

```text
GitHub Organization
```

---

## Scenario 2

Need To Manage 100 Developers.

### Solution

```text
GitHub Teams
```

---

## Scenario 3

Need Shared CI/CD Credentials.

### Solution

```text
Organization Secrets
```

---

## Scenario 4

Need DevOps Team Review For Infrastructure Changes.

### Solution

```text
Teams + CODEOWNERS
```

---

## Scenario 5

Need To Give Vendor Access.

### Solution

```text
External Collaborator
```

---

# Key Takeaways

```text
GitHub Organizations provide centralized repository management.

Organizations own repositories instead of individuals.

Teams simplify access control.

Permissions can be assigned at team level.

Organization Secrets can be shared across repositories.

External Collaborators allow controlled third-party access.

Organizations are essential for enterprise GitHub usage.

Teams integrate with CODEOWNERS and Branch Protection rules.
```