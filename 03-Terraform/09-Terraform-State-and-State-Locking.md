# Terraform State and State Locking

## Introduction

One of the most important concepts in Terraform is:

```text
State
```

Terraform is a:

```text
Declarative Infrastructure as Code Tool
```

This means we declare:

```text
What Infrastructure Should Exist
```

and Terraform determines:

```text
What Needs To Be Created

What Needs To Be Updated

What Needs To Be Deleted
```

---

# What is Declarative Infrastructure?

In Terraform, we describe the desired infrastructure.

Example:

```hcl
resource "aws_instance" "frontend" {

  ami           = "ami-09c813fb71547fc4f"

  instance_type = "t3.micro"

}
```

We are not telling Terraform:

```text
How To Create
```

Instead we are declaring:

```text
This Resource Should Exist
```

Terraform decides how to achieve it.

---

# Why Does Terraform Need State?

Terraform must compare three things:

```text
Desired Infrastructure

Actual Infrastructure

Terraform State
```

Without state, Terraform would not know:

```text
What It Created

What Changed

What Must Be Updated
```

---

# The Three Components

## 1. Desired Infrastructure

Defined in:

```text
.tf Files
```

Examples:

```text
main.tf

variables.tf

outputs.tf
```

Represents:

```text
Expected Infrastructure
```

---

## 2. Actual Infrastructure

Resources that currently exist in AWS.

Examples:

```text
EC2

VPC

Route53

RDS
```

Represents:

```text
Reality
```

---

## 3. Terraform State

Stored in:

```text
terraform.tfstate
```

Represents:

```text
Terraform Memory
```

Terraform uses state to track what it manages.

---

# Visual Representation

```text
Terraform Code (.tf)
         ↓
Desired Infrastructure
         ↓

Terraform State
         ↓

Actual AWS Infrastructure
```

Terraform continuously compares all three.

---

# What is terraform.tfstate?

Terraform automatically creates:

```text
terraform.tfstate
```

after the first successful apply.

Example:

```bash
terraform apply
```

Creates:

```text
terraform.tfstate
```

---

# What's Inside State File?

State contains:

```text
Resource IDs

ARNs

IP Addresses

Metadata

Dependencies
```

Example:

```text
EC2 Instance ID

Security Group ID

Route53 Record ID
```

---

# First Terraform Run

Scenario:

```text
No State File Exists
```

Terraform sees:

```text
Desired Infrastructure Exists

Actual Infrastructure Does Not Exist

State File Does Not Exist
```

Result:

```text
Create Everything
```

---

# First Run Flow

```text
terraform apply
        ↓
No State File
        ↓
Create Resources
        ↓
Create terraform.tfstate
```

---

# Second Terraform Run

Now:

```text
State File Exists

Infrastructure Exists
```

Terraform executes:

```bash
terraform plan
```

or

```bash
terraform apply
```

---

# Refreshing State

During execution Terraform performs:

```text
Refreshing State
```

Example:

```text
aws_security_group.allow_all:
Refreshing state...
```

Terraform checks:

```text
Does State Match AWS?
```

---

# Scenario: Nothing Changed

Terraform compares:

```text
Desired Infrastructure

Actual Infrastructure

State File
```

All are identical.

Result:

```text
No Changes
```

Example:

```text
No changes.

Infrastructure matches configuration.
```

---

# Scenario: Resource Deleted Manually

Suppose someone deletes:

```text
EC2 Instance
```

directly from AWS Console.

Terraform code still contains:

```hcl
resource "aws_instance" "frontend"
```

State file still contains:

```text
Frontend Instance Information
```

But actual AWS infrastructure:

```text
Instance Missing
```

---

# Comparison

```text
Desired Infrastructure = Exists

State File = Exists

Actual Infrastructure = Missing
```

---

# Result

Terraform detects:

```text
Infrastructure Drift
```

and plans:

```text
Create Resource Again
```

---

# What is Infrastructure Drift?

Drift occurs when:

```text
Someone Changes Infrastructure

Outside Terraform
```

Examples:

```text
AWS Console

AWS CLI

Manual Changes

Another Automation Tool
```

---

# Drift Example

Terraform Created:

```text
Security Group
```

Someone manually changes:

```text
Inbound Rules
```

using AWS Console.

Terraform detects:

```text
Desired State

≠

Actual State
```

---

# Scenario: Terraform Code Changed

Suppose:

Old:

```hcl
instance_type = "t3.micro"
```

New:

```hcl
instance_type = "t3.small"
```

---

# Comparison

```text
Desired Infrastructure = t3.small

State = t3.micro

Actual Infrastructure = t3.micro
```

---

# Result

Terraform plans:

```text
Update Resource
```

---

# Scenario: Resource Removed from Code

Original:

```hcl
resource "aws_instance" "frontend" {

}
```

---

You remove:

```hcl
aws_instance.frontend
```

from Terraform code.

---

# Comparison

```text
Desired Infrastructure = Missing

State = Exists

Actual Infrastructure = Exists
```

---

# Result

Terraform plans:

```text
Destroy Resource
```

---

# CRUD Operations Using State

Terraform State helps perform:

```text
Create

Read

Update

Delete
```

operations.

---

# Why State Is Important

State allows Terraform to:

```text
Track Resources

Detect Changes

Prevent Duplicates

Manage Dependencies
```

Without state:

```text
Terraform Would Create Duplicate Resources
```

every time.

---

# Terraform State Is Memory

A common interview answer:

```text
Terraform State Is Terraform's Memory
```

Because it remembers:

```text
What Was Created

Where It Exists

What Changed
```

---

# State Refresh Process

Whenever Terraform runs:

```bash
terraform plan
```

or

```bash
terraform apply
```

Terraform:

```text
Reads State

Queries AWS

Compares Differences

Generates Execution Plan
```

---

# State Locking

Another critical production concept.

---

# Why Do We Need State Locking?

Suppose:

Developer A runs:

```bash
terraform apply
```

At the same time:

Developer B also runs:

```bash
terraform apply
```

---

# Problem

Both users attempt to modify:

```text
Same Infrastructure

Same State File
```

Result:

```text
Corrupted State

Conflicting Changes

Infrastructure Issues
```

---

# State Locking Solution

Terraform locks the state while operations are running.

```text
Apply Starts
      ↓
State Locked
      ↓
Changes Applied
      ↓
Lock Released
```

---

# Benefits of State Locking

```text
Prevents Concurrent Changes

Protects State File

Avoids Corruption

Improves Reliability
```

---

# Local State Locking

When using local state:

```text
terraform.tfstate
```

Terraform automatically manages locking locally.

---

# Remote State Locking

Production environments usually use:

```text
AWS S3

Azure Storage

GCS
```

for remote state.

Locking is handled through backend mechanisms.

---

# Production Architecture

```text
Terraform
      ↓
Remote State (S3)
      ↓
State Locking
      ↓
AWS Infrastructure
```

---

# Common State Commands

## List Resources

```bash
terraform state list
```

---

## Show Resource Details

```bash
terraform state show aws_instance.frontend
```

---

## Remove Resource from State

```bash
terraform state rm aws_instance.frontend
```

---

## Move State

```bash
terraform state mv
```

---

# Common Mistakes

## Deleting State File

Bad Practice.

Terraform loses resource tracking information.

---

## Editing State Manually

Highly risky.

May corrupt infrastructure tracking.

---

## Storing State in Git

Never commit:

```text
terraform.tfstate
```

to Git repositories.

---

## Ignoring Infrastructure Drift

Manual changes outside Terraform should be avoided.

---

# Best Practices

## Use Remote State

Production environments should not use local state.

---

## Enable State Locking

Mandatory for team environments.

---

## Avoid Manual Infrastructure Changes

Use Terraform whenever possible.

---

## Protect State Files

State may contain:

```text
Resource IDs

IP Addresses

Sensitive Metadata
```

---

## Backup State Files

Critical for disaster recovery.

---

# Frequently Asked Interview Questions

## What is Terraform State?

### Short Answer

Terraform State is a file that stores information about resources managed by Terraform.

### Detailed Explanation

Terraform State acts as Terraform's memory. It keeps track of resources that Terraform created and helps compare desired infrastructure with actual infrastructure. This comparison allows Terraform to determine what needs to be created, updated, or deleted.

### Production Example

When Terraform creates an EC2 instance, details such as instance ID and metadata are stored in the state file. Future Terraform runs use this information to manage the resource.

### Follow-Up Questions

- Where is state stored?
- Why is state important?
- What happens if state is deleted?

---

## What is terraform.tfstate?

### Short Answer

terraform.tfstate is the default file used by Terraform to store state information.

### Detailed Explanation

It contains resource metadata, IDs, dependencies, and infrastructure information that Terraform uses to manage resources over time.

### Production Example

Terraform stores EC2 instance IDs, Route53 record IDs, and Security Group IDs inside terraform.tfstate.

### Follow-Up Questions

- Should state files be committed to Git?
- Does state contain sensitive information?

---

## What is Infrastructure Drift?

### Short Answer

Infrastructure drift occurs when actual infrastructure differs from Terraform's desired configuration.

### Detailed Explanation

Drift typically happens when resources are modified manually using the AWS Console, CLI, or another tool instead of Terraform.

### Production Example

A Security Group rule is changed manually in AWS Console. Terraform detects that the actual configuration no longer matches the desired state.

### Follow-Up Questions

- How does Terraform detect drift?
- How can drift be avoided?

---

## What is State Locking?

### Short Answer

State locking prevents multiple users from modifying the same state simultaneously.

### Detailed Explanation

Terraform locks the state during operations such as apply and destroy. This prevents race conditions, state corruption, and conflicting infrastructure changes.

### Production Example

Two DevOps engineers accidentally run terraform apply at the same time. State locking ensures only one operation proceeds.

### Follow-Up Questions

- Why is locking important?
- What happens without locking?

---

## Why is State Important?

### Short Answer

State enables Terraform to compare desired infrastructure with actual infrastructure.

### Detailed Explanation

Without state, Terraform would not know what resources already exist, which resources it manages, or what changes are required.

### Production Example

Terraform uses state to determine whether an EC2 instance should be created, modified, or destroyed.

### Follow-Up Questions

- Can Terraform work without state?
- What information does state store?

---

# Key Takeaways

- Terraform State is Terraform's memory.
- State is stored in `terraform.tfstate`.
- Terraform compares:
  
  ```text
  Desired Infrastructure
  Actual Infrastructure
  State
  ```

- State enables Create, Read, Update, and Delete operations.
- Infrastructure drift occurs when resources change outside Terraform.
- State locking prevents concurrent modifications.
- State files should be protected and never committed to Git.
- Understanding state is essential for Terraform interviews and production environments.