# Terraform Introduction and Infrastructure as Code (IaC)

## Introduction

So far we have learned:

```text
Linux

Shell Scripting

Git

Ansible
```

Ansible is primarily a:

```text
Configuration Management Tool
```

Terraform is primarily an:

```text
Infrastructure Provisioning Tool
```

---

# Configuration Management

Configuration Management (CM) is the process of configuring servers after they are created.

Examples:

```text
Install Packages

Create Users

Create Directories

Configure Applications

Manage Services

Deploy Code
```

---

# Configuration as Code

Configuration can be written as code and stored in version control.

Example:

```text
Install Nginx

Install Java

Configure MongoDB

Deploy Catalogue Service
```

using:

```text
Ansible Playbooks
```

---

# CRUD Operations in Configuration Management

Configuration Management supports:

```text
Create Configuration

Read Configuration

Update Configuration

Delete Configuration
```

---

# Ansible and Configuration Management

Ansible uses:

```text
Collections

Modules

Parameters
```

Example:

```yaml
- name: Install Nginx

  dnf:
    name: nginx
    state: present
```

---

# Can Ansible Create Infrastructure?

Yes.

Ansible can create:

```text
EC2

Route53

Load Balancers

Security Groups
```

using cloud modules.

---

# Why Is Ansible Not Preferred for Infrastructure Provisioning?

Although Ansible can create infrastructure, it was originally designed for:

```text
Configuration Management
```

and not large-scale infrastructure provisioning.

---

# State Management Problem

One of the biggest limitations:

```text
No Native State Management
```

---

# What is State?

State means:

```text
Current Infrastructure Status
```

Example:

```text
EC2 Exists

Security Group Exists

Route53 Record Exists
```

Terraform maintains this information.

Ansible does not maintain infrastructure state in the same way.

---

# Recommended Workflow

Industry Standard:

```text
Terraform
      │
      ▼
Create Infrastructure
      │
      ▼
Ansible
      │
      ▼
Configure Servers
```

---

# Example

Terraform:

```text
Create VPC

Create Security Groups

Create EC2

Create Route53 Records
```

---

Ansible:

```text
Install Java

Install NodeJS

Configure MongoDB

Deploy Applications
```

---

# What is Infrastructure as Code (IaC)?

Infrastructure as Code means:

```text
Managing Infrastructure

Using Code
```

instead of manual cloud console operations.

---

# Traditional Method

```text
Login AWS Console

Create EC2

Create Security Group

Create Route53 Record
```

Problems:

```text
Manual Work

Human Errors

No Audit Trail
```

---

# Infrastructure as Code Benefits

## Version Control

Infrastructure definitions can be stored in Git.

---

## Automatic CRUD

Terraform can:

```text
Create

Read

Update

Delete
```

infrastructure resources.

---

## Consistent Environments

Same code can create:

```text
DEV

QA

UAT

PRE-PROD

PERF

PROD
```

environments.

---

## Inventory Management

Infrastructure code acts as documentation.

You know exactly:

```text
EC2 Instances

Load Balancers

Databases

Networking Components
```

being used.

---

## Cost Optimization

Unused resources can be identified and removed.

---

## Time Saving

Automation reduces manual effort.

---

## Reduced Human Errors

Infrastructure is created consistently.

---

## Dependency Management

Terraform understands dependencies.

Example:

```text
Security Group
      │
      ▼
EC2 Instance
      │
      ▼
Route53 Record
```

Resources are created in the correct order.

---

## Reusable Infrastructure

Infrastructure can be modularized and reused.

---

# Popular Infrastructure as Code Tools

Examples:

```text
Terraform

Pulumi

CloudFormation

OpenTofu
```

---

# Why Terraform?

Terraform is the most widely adopted IaC tool because it:

```text
Cloud Agnostic

Declarative

State Aware

Modular
```

---

# Terraform Installation

Terraform must be installed before use.

Official Documentation:

```text
https://developer.hashicorp.com/terraform
```

---

# AWS CLI

Terraform interacts with AWS using AWS APIs.

AWS CLI is commonly installed alongside Terraform.

Official Documentation:

```text
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
```

---

# Linux Commands and PATH

When a command runs:

```bash
terraform
```

Linux searches locations such as:

```text
/usr/bin

/bin
```

for executable files.

---

# PATH Variable

Example:

```bash
echo $PATH
```

Output:

```text
/usr/local/bin:/usr/bin:/bin
```

Terraform executable must exist in one of these locations.

---

# What is HCL?

Terraform uses:

```text
HCL

HashiCorp Configuration Language
```

---

# Why HCL?

Designed to be:

```text
Readable

Simple

Human Friendly
```

---

# Example

```hcl
name = "frontend"

instance_type = "t3.micro"
```

---

# Key Value Structure

Most Terraform configuration uses:

```hcl
key = value
```

Example:

```hcl
ami = "ami-12345"

instance_type = "t3.micro"
```

---

# Terraform Resource Block

Basic Structure:

```hcl
resource "type_of_resource" "resource_name" {

  key = value

}
```

---

# Example EC2 Instance

```hcl
resource "aws_instance" "frontend" {

  ami           = "ami-123456"

  instance_type = "t3.micro"

}
```

---

# Understanding Resource Block

```hcl
resource
```

Terraform keyword.

---

```hcl
aws_instance
```

Resource type.

---

```hcl
frontend
```

Logical resource name.

---

# Terraform Workflow

Terraform follows a simple lifecycle.

---

## Step 1

```bash
terraform init
```

Purpose:

```text
Initialize Project

Download Providers

Prepare Working Directory
```

---

## Step 2

```bash
terraform plan
```

Purpose:

```text
Preview Changes
```

Shows what Terraform will create, update, or destroy.

---

## Step 3

```bash
terraform apply
```

Purpose:

```text
Create Infrastructure
```

Terraform asks for confirmation.

---

## Auto Approve

```bash
terraform apply -auto-approve
```

Skips manual confirmation.

---

## Destroy Infrastructure

```bash
terraform destroy -auto-approve
```

Removes all managed resources.

---

# Variables

Like shell scripting:

```bash
VAR_NAME="VALUE"
```

Terraform also supports variables.

Example:

```hcl
variable "instance_type" {

  default = "t3.micro"

}
```

Variables help create reusable infrastructure.

---

# Frequently Asked Interview Questions

## What is Infrastructure as Code?

Infrastructure as Code (IaC) is the practice of managing infrastructure through code rather than manual operations.

Benefits include version control, automation, consistency, and repeatability.

---

## Why Is Terraform Preferred for Infrastructure Provisioning?

Terraform provides:

- State Management
- Dependency Handling
- Reusability
- Multi-Cloud Support

making it ideal for infrastructure provisioning.

---

## What Is the Difference Between Terraform and Ansible?

Terraform focuses on infrastructure provisioning.

Ansible focuses on configuration management and application deployment.

They are commonly used together.

---

## What Does terraform init Do?

Initializes the Terraform project and downloads required providers.

---

## What Does terraform plan Do?

Shows a preview of infrastructure changes before applying them.

---

## What Does terraform apply Do?

Creates or updates infrastructure based on the Terraform configuration.

---

## What Is HCL?

HashiCorp Configuration Language (HCL) is the language used by Terraform to define infrastructure.

---

# Key Takeaways

- Terraform is an Infrastructure as Code tool.
- Ansible is a Configuration Management tool.
- Terraform is commonly used before Ansible.
- Infrastructure can be version controlled and automated.
- HCL is Terraform's configuration language.
- Terraform workflow consists of init, plan, apply, and destroy.
- State management is one of Terraform's biggest advantages.
- Terraform is one of the most important DevOps tools in modern cloud environments.