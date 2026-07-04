# Terraform Module Deployment Order and Automation

## Introduction

In production environments, Terraform projects are usually divided into multiple modules.

Examples:

```text
VPC

Security Groups

Bastion Host

Load Balancers

ACM Certificates

Applications
```

Each module depends on other modules.

Therefore, infrastructure must be deployed in a specific order.

---

# Why Deploy Modules Sequentially?

Suppose:

```text
Backend ALB
```

requires:

```text
VPC

Subnets

Security Groups
```

If these resources do not exist:

```text
Terraform Apply Fails
```

because dependencies are missing.

---

# Roboshop Project Structure

```text
00-vpc/

10-sg/

20-bastion/

30-sg-rules/

50-backend-alb/

70-acm/

80-frontend-alb/
```

---

# Module Dependency Flow

```text
00-vpc
    ↓
10-sg
    ↓
20-bastion
    ↓
30-sg-rules
    ↓
50-backend-alb
    ↓
70-acm
    ↓
80-frontend-alb
```

---

# Deployment Automation

Instead of manually entering every module:

```bash
cd 00-vpc
terraform apply

cd ../10-sg
terraform apply

cd ../20-bastion
terraform apply
```

we can automate the process using a Bash loop.

---

# Automation Script

```bash
for i in 00-vpc/ 10-sg/ 20-bastion/ 30-sg-rules/ 50-backend-alb/ 70-acm/ 80-frontend-alb/; do
    cd $i
    terraform apply -auto-approve
    cd ..
done
```

---

# Understanding the Script

## Step 1 - Loop Through Directories

```bash
for i in directories
```

Variable:

```text
i
```

stores one directory at a time.

Example:

```text
00-vpc/

10-sg/

20-bastion/
```

---

## Step 2 - Move Into Module

```bash
cd $i
```

Example:

```bash
cd 00-vpc/
```

Terraform enters the module directory.

---

## Step 3 - Execute Terraform

```bash
terraform apply -auto-approve
```

Creates or updates infrastructure.

---

## Step 4 - Return to Parent Directory

```bash
cd ..
```

Returns to the root project folder.

---

## Step 5 - Repeat

The loop continues until all modules are deployed.

---

# What Does -auto-approve Mean?

Normally Terraform asks:

```text
Do you want to perform these actions?
```

---

Using:

```bash
-auto-approve
```

Terraform skips manual approval and proceeds automatically.

---

# Why Number Prefixes?

Module names:

```text
00-vpc

10-sg

20-bastion

30-sg-rules

50-backend-alb

70-acm

80-frontend-alb
```

represent:

```text
Execution Order
```

and dependency hierarchy.

---

# Module Breakdown

## 00-vpc

Creates:

```text
VPC

Public Subnets

Private Subnets

Route Tables

Internet Gateway

NAT Gateway
```

Everything else depends on the VPC.

---

## 10-sg

Creates:

```text
Security Groups
```

Requires:

```text
VPC ID
```

from the VPC module.

---

## 20-bastion

Creates:

```text
Bastion Host
```

Requires:

```text
Public Subnet

Security Group
```

---

## 30-sg-rules

Creates:

```text
Ingress Rules

Egress Rules

Security Group Relationships
```

Requires:

```text
Existing Security Groups
```

---

## 50-backend-alb

Creates:

```text
Backend Load Balancer

Listeners

Target Groups
```

Requires:

```text
Subnets

Security Groups
```

---

## 70-acm

Creates:

```text
SSL Certificates
```

Requires:

```text
Domain Name

Route53 Hosted Zone
```

---

## 80-frontend-alb

Creates:

```text
Frontend ALB

HTTPS Listener

Routing Rules
```

Requires:

```text
ACM Certificate

Target Groups

Security Groups
```

---

# Production Example

Roboshop Infrastructure Deployment:

```text
00-vpc
    ↓
10-sg
    ↓
20-bastion
    ↓
30-sg-rules
    ↓
50-backend-alb
    ↓
70-acm
    ↓
80-frontend-alb
```

Running the Bash loop deploys the entire infrastructure automatically.

---

# Benefits of This Approach

```text
Automation

Consistency

Dependency Management

Faster Deployments

Reduced Human Errors

Easy Maintenance
```

---

# Common Interview Questions

## Why Do We Split Terraform Into Multiple Modules?

### Short Answer

To improve maintainability, reusability, and separation of responsibilities.

### Detailed Explanation

Large Terraform projects become difficult to manage as a single codebase. Modules allow infrastructure to be organized into logical components.

### Production Example

Separate modules for VPC, Security Groups, ACM, and ALBs in the Roboshop project.

### Follow-Up Questions

- What are Terraform modules?
- How do modules communicate with each other?

---

## Why Is Deployment Order Important?

### Short Answer

Resources often depend on other resources being created first.

### Detailed Explanation

Resources such as ALBs and Security Groups require VPC and subnet information. Terraform modules must be applied in dependency order.

### Production Example

Backend ALB cannot be created before the VPC and subnets exist.

### Follow-Up Questions

- What happens if dependencies are missing?
- How does Terraform manage dependencies?

---

## Why Use a Bash Loop for Terraform?

### Short Answer

To automate deployment of multiple Terraform modules sequentially.

### Detailed Explanation

The loop enters each module directory, executes Terraform Apply, and continues to the next module automatically.

### Production Example

Deploying the complete Roboshop infrastructure using a single command.

### Follow-Up Questions

- Can the same approach be used for destroy?
- How can failures be handled?

---

## Why Use Numbered Module Names?

### Short Answer

To indicate execution order and dependency hierarchy.

### Detailed Explanation

The numbering helps engineers understand which modules should be deployed first.

### Production Example

00-vpc must be deployed before 10-sg because Security Groups require a VPC.

### Follow-Up Questions

- Is numbering mandatory?
- What other methods can enforce ordering?

---

# Key Takeaways

```text
Terraform projects are commonly split into modules.

Modules often depend on other modules.

Infrastructure must be deployed in the correct order.

Numbered directories indicate execution order.

Bash loops automate module deployment.

-auto-approve skips manual confirmation.

Module-based infrastructure is easier to manage and scale.

This deployment approach is commonly used in production Terraform projects.
```