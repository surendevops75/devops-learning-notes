# Terraform Setup, Workflow and Git Integration

## Introduction

Before creating infrastructure using Terraform, we need to:

1. Install Terraform
2. Configure Terraform in System PATH
3. Install AWS CLI
4. Configure AWS Credentials
5. Understand Terraform Workflow
6. Store Terraform Code in Git Repository

These are the first practical steps in every Terraform project.

---

# Terraform Installation

Terraform is distributed as a binary executable.

Download Terraform from:

```text
https://developer.hashicorp.com/terraform/downloads
```

---

# Verify Installation

After installation, verify:

```bash
terraform --version
```

Example Output:

```text
Terraform v1.13.x
```

---

# Configure Environment Variable (PATH)

After installing Terraform, the executable must be available in the system PATH.

Verify:

```bash
terraform --version
```

If Terraform is not recognized:

```text
Add Terraform Installation Directory

to

System PATH
```

---

# Why PATH is Required?

Without PATH:

```bash
terraform init
```

returns:

```text
terraform: command not found
```

---

# AWS CLI Installation

Terraform needs credentials to communicate with AWS.

AWS CLI helps manage AWS authentication.

Install AWS CLI Version 2.

Verify:

```bash
aws --version
```

Example:

```text
aws-cli/2.x.x
```

---

# Configure AWS Credentials

Run:

```bash
aws configure
```

Example:

```text
AWS Access Key ID:
```

```text
AWS Secret Access Key:
```

```text
Default Region:
```

```text
Output Format:
```

---

# Where Are Credentials Stored?

Linux/Mac:

```text
~/.aws/credentials
```

Windows:

```text
C:\Users\<user>\.aws\credentials
```

---

# Verify AWS Authentication

Command:

```bash
aws sts get-caller-identity
```

Example Output:

```json
{
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/devops"
}
```

This confirms AWS authentication is working.

---

# Terraform Workflow

Terraform follows a simple lifecycle.

```text
Write Code
      ↓
terraform init
      ↓
terraform plan
      ↓
terraform apply
      ↓
Infrastructure Created
      ↓
terraform destroy
```

---

# terraform init

Initializes Terraform.

Command:

```bash
terraform init
```

Purpose:

```text
Download Providers

Initialize Working Directory

Prepare Terraform Environment
```

Example:

```text
AWS Provider Downloaded
```

---

# terraform plan

Shows what Terraform is going to do.

Command:

```bash
terraform plan
```

Purpose:

```text
Create Resources?

Modify Resources?

Delete Resources?
```

No changes are made.

Only a preview is shown.

---

# terraform apply

Creates or modifies infrastructure.

Command:

```bash
terraform apply
```

Terraform asks:

```text
Do you want to perform these actions?
```

Answer:

```text
yes
```

---

# Auto Approve

Skip manual confirmation:

```bash
terraform apply -auto-approve
```

---

# terraform destroy

Removes Terraform-managed infrastructure.

Command:

```bash
terraform destroy
```

Example:

```text
Delete EC2

Delete Security Group

Delete Route53 Record
```

---

# Destroy Without Prompt

```bash
terraform destroy -auto-approve
```

---

# Terraform Resource Block

Terraform creates resources using:

```hcl
resource "type_of_resource" "resource_name" {

  key = value

}
```

---

# Resource Syntax

```hcl
resource "type" "name" {

}
```

Where:

```text
type  → AWS Resource Type

name  → Our Custom Name
```

---

# Example

```hcl
resource "aws_instance" "frontend" {

  ami           = "ami-09c813fb71547fc4f"

  instance_type = "t3.micro"

}
```

---

# Understanding the Resource Block

```hcl
resource "aws_instance" "frontend"
```

Meaning:

```text
aws_instance  → Resource Type

frontend      → Terraform Logical Name
```

---

# Logical Name

Terraform name:

```hcl
frontend
```

does not need to match:

```text
EC2 Instance Name
```

Example:

```hcl
resource "aws_instance" "server1"
```

Perfectly valid.

---

# Why Use Git With Terraform?

Terraform code should always be version controlled.

Benefits:

```text
Version History

Team Collaboration

Rollback Capability

Code Review

Auditability
```

---

# Git vs Drive

## Drive

Used for:

```text
Files

Documents

Images

Backups
```

---

## Git Repository

Used for:

```text
Source Code

Terraform Code

Ansible Playbooks

Application Code
```

---

# Git Workflow

Terraform project lifecycle:

```text
Working Directory
        ↓
git add
        ↓
Staging Area
        ↓
git commit
        ↓
Local Repository
        ↓
git push
        ↓
Remote Repository
```

---

# What is Staging Area?

Temporary area between:

```text
Working Directory

and

Git Commit
```

Command:

```bash
git add .
```

---

# What is Local Repository?

After commit:

```bash
git commit -m "Initial Terraform Code"
```

Code is stored locally.

---

# What is Remote Repository?

Examples:

```text
GitHub

GitLab

Bitbucket
```

Push:

```bash
git push
```

---

# Common Git Problem

Situation:

```text
Commit Successful

Push Failed
```

Usually because:

```text
Wrong Repository

Authentication Issue

Repository Already Exists
```

---

# Reinitialize Git Repository

Remove existing git metadata:

```bash
rm -rf .git
```

---

# Create Fresh Repository

Initialize:

```bash
git init
```

---

Rename Branch:

```bash
git branch -M main
```

---

Add Remote:

```bash
git remote add origin <URL>
```

Example:

```bash
git remote add origin https://github.com/user/repo.git
```

---

# Push Code

```bash
git add .

git commit -m "Initial Commit"

git push -u origin main
```

---

# Why Use .gitignore?

Some files should never be stored in Git.

Examples:

```text
Terraform State Files

Credentials

Temporary Files
```

---

# Terraform .gitignore Example

```gitignore
.terraform/

*.tfstate

*.tfstate.*

terraform.tfvars

crash.log
```

---

# Why Ignore State Files?

State files may contain:

```text
Infrastructure Details

Sensitive Data

Passwords

Resource IDs
```

Never commit them to public repositories.

---

# Project Structure

Example:

```text
terraform-project/

├── main.tf

├── variables.tf

├── terraform.tfvars

├── .gitignore

└── README.md
```

---

# Real DevOps Workflow

```text
Developer Writes Terraform
           ↓
git add
           ↓
git commit
           ↓
git push
           ↓
GitHub Repository
           ↓
CI/CD Pipeline
           ↓
terraform init
           ↓
terraform plan
           ↓
terraform apply
```

---

# Common Mistakes

## Not Running terraform init

Error:

```text
Provider Not Found
```

---

## Committing State Files

Bad Practice.

Use:

```gitignore
*.tfstate
```

---

## Hardcoding Credentials

Bad:

```hcl
access_key = "AKIAxxxx"
```

Use:

```text
AWS CLI

IAM Roles

Environment Variables
```

---

## Skipping terraform plan

Always review infrastructure changes before apply.

---

# Frequently Asked Interview Questions

## What Does terraform init Do?

Initializes Terraform and downloads required providers.

---

## What Does terraform plan Do?

Shows the execution plan without making changes.

---

## What Does terraform apply Do?

Creates or updates infrastructure.

---

## What Does terraform destroy Do?

Deletes Terraform-managed infrastructure.

---

## What is a Resource Block?

A block used to define infrastructure resources.

Example:

```hcl
resource "aws_instance" "frontend"
```

---

## Why Use Git for Terraform?

To manage infrastructure code through version control.

---

## Why Should State Files Not Be Committed?

Because they may contain sensitive information and environment-specific data.

---

## What Files Should Be Ignored?

```text
.terraform/

*.tfstate

terraform.tfvars
```

---

# Key Takeaways

- Install Terraform and configure PATH.
- Install AWS CLI and configure credentials.
- Understand the Terraform lifecycle:
  - init
  - plan
  - apply
  - destroy
- Resources are defined using resource blocks.
- Git should always be used to manage Terraform code.
- Use `.gitignore` to avoid committing sensitive files.
- Never hardcode credentials.
- Always review changes using `terraform plan`.