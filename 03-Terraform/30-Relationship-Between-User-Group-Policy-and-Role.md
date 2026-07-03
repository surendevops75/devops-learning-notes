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

### Detailed Explanation

Authentication answers the question "Who are you?" and verifies whether a user belongs to the system. Authorization answers the question "What can you do?" and determines which resources and actions the authenticated user can access.

### Production Example

A DevOps engineer successfully logs into AWS using username and password. Authentication succeeds. When attempting to terminate an EC2 instance, AWS checks permissions. If the required permission is missing, authorization fails and access is denied.

### Follow-Up Questions

- Which happens first, authentication or authorization?
- Can authentication succeed and authorization fail?
- Give a real AWS example of authorization.

---

## What is the Difference Between User and Role?

### Short Answer

Users are permanent identities for humans, whereas roles are temporary identities used by AWS services or assumed users.

### Detailed Explanation

An IAM User represents a person and typically has long-term credentials such as passwords or access keys. An IAM Role is assumed temporarily and provides temporary credentials. Roles are commonly attached to AWS services such as EC2, Lambda, and EKS.

### Production Example

Surendra logs into AWS using an IAM User. A MongoDB EC2 instance accesses SSM Parameter Store using an attached IAM Role.

### Follow-Up Questions

- Can a user assume a role?
- Can EC2 use an IAM User?
- Why are roles preferred for AWS services?

---

## What is the Difference Between Policy and Role?

### Short Answer

A policy defines permissions, whereas a role uses one or more policies to grant access.

### Detailed Explanation

Policies contain allowed or denied actions on AWS resources. Roles do not directly contain permissions; instead, policies are attached to roles. When a role is assumed, the permissions defined in the attached policies become available.

### Production Example

A MongoDB EC2 instance has a role called MongoDBRole. The role contains an SSMReadPolicy that allows reading parameters from SSM Parameter Store.

### Follow-Up Questions

- Can a role have multiple policies?
- Can a policy be attached to multiple roles?
- What is the difference between managed and inline policies?

---

## Why Use IAM Roles Instead of Access Keys?

### Short Answer

IAM Roles provide temporary credentials and eliminate the need to store access keys.

### Detailed Explanation

Access keys are long-lived credentials that can be leaked, rotated incorrectly, or exposed accidentally. IAM Roles generate temporary credentials automatically and are managed by AWS, making them much more secure.

### Production Example

A Catalogue application running on EC2 reads database credentials from SSM Parameter Store using an attached IAM Role rather than storing AWS access keys on the server.

### Follow-Up Questions

- What are the risks of storing access keys?
- How does AWS provide temporary credentials?
- Which AWS services commonly use IAM Roles?

---

# Key Takeaways

```text
Authentication verifies identity.

Authorization verifies permissions.

AWS resources are nouns.

AWS actions are verbs.

Users represent humans.

Groups are collections of users.

Policies define permissions.

Roles use policies to provide access.

Human access typically follows:
User → Group → Policy → Permissions

AWS service access typically follows:
EC2 → Role → Policy → Permissions

IAM Roles are preferred over access keys.

EC2 instances can securely access SSM Parameter Store using IAM Roles.

Policies define what can be done.

Roles define who can use those permissions.

Temporary credentials are more secure than long-term access keys.
```