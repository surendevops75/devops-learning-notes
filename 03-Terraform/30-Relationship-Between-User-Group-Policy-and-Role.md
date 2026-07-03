# Relationship Between User, Group, Policy and Role

## Introduction

AWS IAM is built using four major components:

```text
User

Group

Policy

Role
```

Understanding the relationship between them is one of the most common AWS interview topics.

---

# Authentication and Authorization

AWS consists of:

```text
Resources

Actions
```

Examples:

Resources:

```text
EC2

S3

ALB

Route53

Secrets Manager
```

Actions:

```text
Create

View

Update

Delete
```

---

# Nouns and Verbs

Resources are:

```text
Nouns
```

Examples:

```text
EC2

S3

ALB
```

Actions are:

```text
Verbs
```

Examples:

```text
Create EC2

View EC2

Delete EC2

Start EC2
```

---

# Authentication

Authentication means:

```text
Prove Yourself
```

Question:

```text
Who Are You?
```

Examples:

```text
Username

Password

Access Key

Secret Key
```

---

# Authorization

Authorization means:

```text
Prove You Have Access
```

Question:

```text
What Can You Do?
```

Examples:

```text
Can Create EC2?

Can Delete VPC?

Can Read S3?
```

---

# IAM Components

```text
User

Group

Policy

Role
```

---

# User

Represents:

```text
Human Identity
```

Examples:

```text
Ramesh

Surendra

John
```

User can login using:

```text
Username & Password

Access Key & Secret Key
```

---

# Group

Group is:

```text
Collection Of Users
```

Example:

```text
DevOps Team
```

Contains:

```text
Ramesh

Surendra

John
```

---

# Why Groups?

Without Groups:

```text
Assign Permissions

To Every User
```

---

With Groups:

```text
Assign Once

To Group
```

All members inherit permissions.

---

# Policy

Policy contains:

```text
Permissions
```

A policy defines:

```text
Which Resource

Can Perform Which Action
```

---

# Policy Structure

```text
Resource + Action
```

Example:

```text
Resource = EC2

Action = View
```

AWS Permission:

```text
ec2:DescribeInstances
```

---

# Trainee Policy Example

Permissions:

```text
View EC2

View S3

View CloudWatch
```

Cannot:

```text
Create EC2

Delete EC2
```

---

# Administrator Policy Example

Permissions:

```text
Full Access
```

Can:

```text
Create

Update

Delete
```

almost everything.

---

# Real Example

User:

```text
Ramesh
```

Assigned:

```text
TraineePolicy
```

Result:

```text
View Resources Only
```

---

User:

```text
Ramesh
```

Assigned:

```text
AdministratorAccess
```

Result:

```text
Full Control
```

---

# Role

Role is:

```text
Collection Of Permissions
```

that can be assumed temporarily.

---

# Role Flow

```text
Role
   ↓
Policies
   ↓
Permissions
```

---

# Human Access Flow

```text
User
   ↓
Group
   ↓
Policy
   ↓
Permissions
```

---

# Service Access Flow

```text
EC2
   ↓
Role
   ↓
Policy
   ↓
Permissions
```

---

# Why Roles?

Instead of storing:

```text
Access Key

Secret Key
```

inside servers,

AWS services can assume roles.

---

# EC2 Role Example

Requirement:

```text
EC2 Instance

↓

Read SSM Parameter Store
```

Parameter:

```text
/roboshop/dev/mysql/mysql_root_password
```

---

Wrong Approach

```text
Store Access Keys

Inside Server
```

---

Recommended Approach

```text
Attach IAM Role
```

Permission:

```text
ssm:GetParameter
```

---

# Production Example

MongoDB Server needs:

```text
Database Password
```

stored in:

```text
SSM Parameter Store
```

Flow:

```text
MongoDB EC2
       ↓
IAM Role
       ↓
SSM Read Policy
       ↓
Read Password
```

No credentials required.

---

# User vs Role

## User

```text
Permanent Identity

Human Login
```

Examples:

```text
Surendra

Ramesh
```

---

## Role

```text
Temporary Identity

Assumed Access
```

Examples:

```text
EC2 Role

Lambda Role

EKS Role
```

---

# Policy vs Role

## Policy

Defines:

```text
Permissions
```

Example:

```text
Read SSM

Read S3
```

---

## Role

Uses:

```text
One Or More Policies
```

to provide access.

---

# Complete IAM Flow

```text
User
   ↓
Group
   ↓
Policy
   ↓
Permissions
```

Human Access.

---

```text
EC2
   ↓
Role
   ↓
Policy
   ↓
Permissions
```

AWS Service Access.

---

# Common Interview Questions

## What is the Difference Between Authentication and Authorization?

### Short Answer

Authentication verifies identity. Authorization verifies permissions.

### Production Example

A DevOps engineer successfully logs into AWS but cannot terminate EC2 instances because permissions are missing.

---

## What is the Difference Between User and Role?

### Short Answer

Users are for humans. Roles are for temporary access and AWS services.

### Production Example

Surendra uses an IAM User. MongoDB EC2 uses an IAM Role.

---

## What is the Difference Between Policy and Role?

### Short Answer

Policy defines permissions. Role uses those permissions.

### Production Example

MongoDB Role contains SSM Read Policy.

---

## Why Use IAM Roles Instead of Access Keys?

### Short Answer

Roles provide temporary credentials and eliminate hardcoded secrets.

### Production Example

EC2 reads passwords from SSM Parameter Store using an IAM Role.