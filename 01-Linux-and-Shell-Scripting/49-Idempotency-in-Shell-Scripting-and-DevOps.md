# Idempotency in Shell Scripting and DevOps

## Introduction

One of the most important concepts in DevOps is:

```text
Idempotency
```

Many DevOps tools and automation platforms are designed around this principle.

Examples:

- Shell Scripts
- Ansible
- Terraform
- Kubernetes
- AWS Automation

A good automation script should be safe to run multiple times.

---

# What is Idempotency?

A process is called idempotent if:

```text
Running it once

OR

Running it 100 times

Produces the same desired result.
```

---

# Simple Definition

```text
Same Input
      │
      ▼
Same Result
```

even when executed repeatedly.

---

# Real World Example

Suppose a room light is already ON.

If you press:

```text
Turn ON
```

again,

Result:

```text
Still ON
```

No additional change.

This is idempotent behavior.

---

# Non-Idempotent Example

Bank Transfer:

```text
Transfer ₹1000
```

Run once:

```text
₹1000 Transferred
```

Run again:

```text
₹2000 Transferred
```

Different result.

This is not idempotent.

---

# Why Idempotency Matters?

In DevOps:

```text
Scripts
Servers
Networks
Applications
```

may be executed repeatedly.

Automation must avoid:

- Duplicate Resources
- Duplicate Users
- Duplicate Configurations
- Unexpected Changes

---

# Goal of Idempotent Scripts

```text
Run Once
Run Twice
Run 100 Times

Same Desired State
```

---

# Example: User Creation

Non-Idempotent:

```bash
useradd roboshop
```

---

First Run:

```text
User Created
```

---

Second Run:

```text
User Already Exists
```

Error generated.

---

# Idempotent Approach

Check first.

```bash
id roboshop
```

---

Logic:

```text
If User Exists

    Skip

Else

    Create User
```

---

Example

```bash
if id roboshop >/dev/null 2>&1
then
    echo "User Already Exists"
else
    useradd roboshop
fi
```

---

# Example: Directory Creation

Non-Idempotent:

```bash
mkdir /app
```

---

Second Run:

```text
File Exists
```

---

Idempotent Version

```bash
mkdir -p /app
```

---

# Why -p?

```text
Creates Directory If Missing

Skips If Already Exists
```

---

# Example: Package Installation

Command:

```bash
dnf install nginx -y
```

---

Behavior:

```text
Already Installed
```

Package managers are generally idempotent.

---

# Example: Service Start

```bash
systemctl start nginx
```

---

If already running:

```text
No Harm
```

Desired state remains:

```text
Running
```

---

# HTTP Methods and Idempotency

In REST APIs, idempotency is very important.

---

# GET

Purpose:

```text
Read Data
```

Example:

```http
GET /users
```

---

Run Once:

```text
Read Users
```

Run Again:

```text
Read Users
```

No change.

---

GET is:

```text
Idempotent
```

---

# PUT / UPDATE

Purpose:

```text
Update Resource
```

Example:

```http
PUT /users/101
```

---

Change:

```text
Name = Surendra
```

---

Run Again:

```text
Name Still Surendra
```

No additional impact.

---

PUT is generally:

```text
Idempotent
```

---

# DELETE

Purpose:

```text
Delete Resource
```

---

Run Once:

```text
Deleted
```

Run Again:

```text
Already Deleted
```

Desired state remains:

```text
Resource Not Present
```

---

DELETE is generally:

```text
Idempotent
```

---

# POST

Purpose:

```text
Create Resource
```

---

Example:

```http
POST /users
```

---

Run Once:

```text
User Created
```

Run Again:

```text
Another User Created
```

Possible duplicates.

---

POST is generally:

```text
Non-Idempotent
```

---

# Summary

| Method | Idempotent |
|----------|------------|
| GET | Yes |
| PUT | Yes |
| DELETE | Yes |
| POST | No |

---

# Database Example

Suppose we create a database.

Non-Idempotent:

```sql
CREATE DATABASE catalogue;
```

---

Second Run:

```text
Database Already Exists
```

---

Idempotent Version

```sql
CREATE DATABASE IF NOT EXISTS catalogue;
```

---

# MongoDB Example

Before creating schema or loading data:

Check whether database already exists.

Example:

```bash
mongosh mongodb.daws86s.fun \
--quiet \
--eval \
"db.getMongo().getDBNames().indexOf('catalogue')"
```

---

# Understanding the Command

Connect to MongoDB.

Retrieve:

```text
Database Names
```

Check whether:

```text
catalogue
```

already exists.

---

# Why Check?

Avoid:

```text
Duplicate Creation
Duplicate Data Loading
Repeated Operations
```

---

# Roboshop Example

Before creating:

```text
roboshop user
```

Check:

```bash
id roboshop
```

---

Before creating:

```text
/app
```

Check:

```bash
mkdir -p /app
```

---

Before loading schema:

```text
Verify Database Exists
```

---

# Infrastructure Example

Suppose script creates:

```text
EC2 Instance
```

every time.

---

Run Once:

```text
1 Instance
```

---

Run Again:

```text
2 Instances
```

---

Run Again:

```text
3 Instances
```

Not idempotent.

---

# Better Approach

Check:

```text
Instance Exists?
```

If yes:

```text
Reuse Existing Instance
```

Otherwise:

```text
Create New Instance
```

---

# Terraform and Idempotency

Terraform is designed around idempotency.

---

Example:

```bash
terraform apply
```

Run Once:

```text
Resources Created
```

---

Run Again:

```text
No Changes
```

---

Desired state already exists.

---

# Ansible and Idempotency

Ansible modules are mostly idempotent.

Example:

```yaml
- name: Install nginx
  dnf:
    name: nginx
    state: present
```

---

Run Once:

```text
Install nginx
```

Run Again:

```text
Already Installed
```

No changes.

---

# Kubernetes and Idempotency

Command:

```bash
kubectl apply -f deployment.yaml
```

---

Run Again:

```bash
kubectl apply -f deployment.yaml
```

Result:

```text
No Unnecessary Changes
```

Desired state maintained.

---

# Common Non-Idempotent Mistakes

## Creating Users Repeatedly

```bash
useradd roboshop
```

---

## Creating Directories Without Validation

```bash
mkdir /app
```

---

## Loading Database Multiple Times

Duplicate data.

---

## Creating Infrastructure Every Run

Duplicate resources.

---

# Best Practices

## Validate Before Create

Example:

```text
User Exists?
Directory Exists?
Database Exists?
```

---

## Prefer Desired State

Instead of:

```text
Create Resource
```

Think:

```text
Ensure Resource Exists
```

---

## Use Idempotent Commands

Examples:

```bash
mkdir -p
```

```sql
CREATE DATABASE IF NOT EXISTS
```

---

## Avoid Duplicate Operations

Prevent repeated resource creation.

---

# Frequently Asked Interview Questions

## What is Idempotency?

Producing the same result regardless of how many times an operation is executed.

---

## Why Is Idempotency Important?

To make automation safe and repeatable.

---

## Is GET Idempotent?

Yes.

---

## Is POST Idempotent?

Generally No.

---

## Is DELETE Idempotent?

Yes.

---

## Is PUT Idempotent?

Yes.

---

## Give a Shell Script Example of Idempotency.

```bash
mkdir -p /app
```

---

## How Does Terraform Achieve Idempotency?

By comparing current state with desired state before making changes.

---

## Why Is Idempotency Important in DevOps?

Because automation scripts are executed repeatedly across environments.

---

# Key Takeaways

- Idempotency means achieving the same desired state regardless of repeated executions.
- Good DevOps automation should always be idempotent.
- GET, PUT, and DELETE are generally idempotent.
- POST is generally non-idempotent.
- Validate resources before creating them.
- User creation, directory creation, and database operations should be idempotent.
- Terraform, Ansible, and Kubernetes heavily rely on idempotent principles.
- Idempotency prevents duplicate resources and unintended side effects.
- Reliable automation always focuses on desired state rather than repeated actions.