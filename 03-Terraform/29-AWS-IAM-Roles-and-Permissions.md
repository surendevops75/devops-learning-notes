# AWS IAM Roles and Permissions

## Introduction

AWS IAM stands for:

```text
Identity and Access Management
```

IAM is used to control:

```text
Who Can Access AWS

What They Can Access

What Actions They Can Perform
```

---

# Why IAM?

Without IAM:

```text
Everyone Gets Full Access
```

Problems:

```text
Security Risks

Accidental Deletions

Unauthorized Access

Compliance Issues
```

IAM provides:

```text
Authentication

Authorization

Access Control
```

---

# Authentication

Authentication means:

```text
Who Are You?
```

AWS verifies identity.

Examples:

```text
Username

Password

Access Key

Secret Key

SSO Login
```

---

# Real World Example

Employee enters:

```text
Username

Password
```

System verifies:

```text
Valid Employee
```

This is:

```text
Authentication
```

---

# Authorization

Authorization means:

```text
What Can You Do?
```

After login:

```text
Can Create EC2?

Can Delete VPC?

Can Read S3?
```

AWS checks permissions.

---

# Real World Example

Employee successfully logs in.

Authentication:

```text
Success
```

Now AWS checks:

```text
Can Create EC2?
```

If permission exists:

```text
Allowed
```

Otherwise:

```text
Access Denied
```

---

# Authentication vs Authorization

Authentication:

```text
Who Are You?
```

Authorization:

```text
What Can You Do?
```

---

# IAM Components

```text
Users

Groups

Policies

Roles
```

---

# IAM User

Represents:

```text
One Individual Person
```

Examples:

```text
Surendra

Ravi

John
```

---

# IAM User Example

DevOps Engineer:

```text
surendra
```

AWS Login:

```text
Username

Password
```

User authenticated.

---

# IAM Group

A Group is:

```text
Collection Of Users
```

---

# Examples

```text
DevOps Team

Developers

QA Team

Security Team
```

---

# Group Example

```text
DevOps-Team
```

contains:

```text
Surendra

Ravi

John
```

Instead of assigning permissions individually:

```text
Assign To Group
```

---

# Why Groups?

Without Groups:

```text
User-1 Permission

User-2 Permission

User-3 Permission
```

repeated many times.

---

With Groups:

```text
Group

↓

Permissions

↓

Users
```

Simpler management.

---

# IAM Policy

Policy contains:

```text
Permissions
```

Examples:

```text
Start EC2

Stop EC2

Create S3 Bucket

Read Secrets
```

---

# Policy Example

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeInstances"
  ],
  "Resource": "*"
}
```

---

# Policy Flow

```text
Policy
   ↓
Permissions
   ↓
Actions
```

---

# What is a Role?

A Role is:

```text
Collection Of Permissions
```

that can be assumed temporarily.

---

# Organization Analogy

Role Examples:

```text
Trainee

ASE

SE

Consultant

Team Lead

Team Manager

Delivery Manager

CEO
```

---

Each Role has:

```text
Different Permissions
```

---

# Example

Team Lead:

```text
Approve Changes
```

Trainee:

```text
Cannot Approve Changes
```

Permissions depend on role.

---

# AWS Role Flow

```text
Role
   ↓
Permissions
   ↓
Actions
```

---

# User Role Assignment

```text
User
   ↓
Role
   ↓
Permissions
```

---

# Why Roles?

Instead of storing credentials:

```text
Access Key

Secret Key
```

AWS services can assume roles.

---

# AWS Service Roles

Roles can be attached to:

```text
EC2

Lambda

EKS

ECS

CodeBuild
```

---

# EC2 IAM Role Example

Requirement:

```text
Read Secrets

Access S3

Write CloudWatch Logs
```

---

Wrong Approach

Store:

```text
Access Key

Secret Key
```

inside server.

Problems:

```text
Security Risk

Key Rotation

Credential Leakage
```

---

Recommended Approach

Attach:

```text
IAM Role
```

to EC2.

---

Flow

```text
EC2
   ↓
IAM Role
   ↓
Temporary Credentials
   ↓
AWS Service Access
```

---

# Production Example

MongoDB Server needs:

```text
Read Secrets
```

from:

```text
AWS Secrets Manager
```

Attach Role:

```text
mongodb-role
```

Permissions:

```text
secretsmanager:GetSecretValue
```

MongoDB reads secrets securely.

---

# Another Example

Catalogue Application needs:

```text
Read Images
```

from S3.

Attach:

```text
catalogue-role
```

Permissions:

```text
s3:GetObject
```

No access keys required.

---

# Role Assumption

AWS services do not permanently own permissions.

They:

```text
Assume Role
```

temporarily.

AWS generates:

```text
Temporary Credentials
```

---

# Benefits of IAM Roles

```text
No Hardcoded Credentials

Temporary Credentials

Better Security

Easy Permission Management

AWS Recommended Practice
```

---

# Common IAM Design

```text
User
   ↓
Group
   ↓
Policies
```

Human access.

---

```text
EC2
   ↓
Role
   ↓
Policies
```

Service access.

---

# Production Architecture Example

```text
EC2 Instance
       ↓
IAM Role
       ↓
Secrets Manager

S3

CloudWatch
```

No credentials stored inside the server.

---

# Common Interview Questions

## What is IAM?

### Short Answer

IAM is AWS Identity and Access Management service used for authentication and authorization.

### Detailed Explanation

IAM controls who can access AWS resources and what actions they can perform.

### Production Example

DevOps engineers accessing AWS accounts using IAM users and groups.

### Follow-Up Questions

- Why is IAM important?
- Can IAM control service access?

---

## Difference Between Authentication and Authorization?

### Short Answer

Authentication verifies identity, authorization verifies permissions.

### Detailed Explanation

Authentication answers "Who are you?" while authorization answers "What can you do?"

### Production Example

User logs in successfully but cannot delete VPCs because permissions are missing.

### Follow-Up Questions

- Which comes first?
- Can authentication succeed but authorization fail?

---

## What is an IAM Role?

### Short Answer

An IAM Role is a collection of permissions that can be assumed temporarily.

### Detailed Explanation

Roles provide temporary credentials and are commonly attached to AWS services.

### Production Example

EC2 accessing Secrets Manager through an attached IAM role.

### Follow-Up Questions

- Why use roles instead of access keys?
- Can roles be attached to EC2?

---

## Why Use IAM Roles Instead of Access Keys?

### Short Answer

Roles are more secure and provide temporary credentials.

### Detailed Explanation

Access keys can be leaked or misused, whereas IAM roles use temporary credentials managed by AWS.

### Production Example

MongoDB server retrieving database passwords from Secrets Manager using IAM roles.

### Follow-Up Questions

- What are the risks of hardcoded access keys?
- How does AWS provide temporary credentials?

---

## What is the Difference Between User and Role?

### Short Answer

Users represent people, roles represent permissions that can be assumed.

### Detailed Explanation

Users are used for human access while roles are typically used for AWS services and temporary access.

### Production Example

DevOps engineer uses IAM User, EC2 instance uses IAM Role.

### Follow-Up Questions

- Can a user assume a role?
- Can EC2 use a user instead of a role?

---

# Key Takeaways

```text
IAM controls authentication and authorization.

Authentication verifies identity.

Authorization verifies permissions.

Users represent individuals.

Groups simplify permission management.

Policies define permissions.

Roles provide temporary access.

AWS services commonly use IAM Roles.

IAM Roles are preferred over access keys.

EC2 can securely access AWS services using attached roles.
```