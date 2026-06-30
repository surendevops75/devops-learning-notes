# Roboshop Infrastructure Automation with Ansible

## Introduction

So far we have learned:

- Variables
- Conditions
- Loops
- Facts
- Modules
- Error Handling
- Idempotency

Now it is time to use these concepts in a real-world project.

In this example, we will automate the infrastructure setup for the Roboshop application using Ansible.

---

# What is Roboshop?

Roboshop is a microservices-based e-commerce application used for learning DevOps concepts.

Components include:

```text
Frontend
Catalogue
User
Cart
Shipping
Payment
Dispatch
MongoDB
MySQL
Redis
RabbitMQ
```

Each component typically runs on a separate server.

---

# Traditional Approach

Manual process:

```text
Login to AWS

Create EC2 Instance

Copy IP Address

Update Route53

Repeat For Every Component
```

Problems:

```text
Time Consuming

Human Errors

Not Repeatable

Difficult To Scale
```

---

# Automated Approach

Using Ansible:

```text
Run Playbook
       │
       ▼
Create EC2 Instances
       │
       ▼
Collect IP Addresses
       │
       ▼
Update Route53 Records
       │
       ▼
Ready For Deployment
```

---

# Target Architecture

```text
                Route53
                    │
 ┌──────────────────┼──────────────────┐
 │                  │                  │
 ▼                  ▼                  ▼

frontend       catalogue         user

cart           shipping          payment

redis          mongodb           mysql

rabbitmq
```

---

# Infrastructure Requirements

For each component:

```text
EC2 Instance

Security Group

Private IP

Route53 Record
```

---

# Automation Goals

Our playbook should:

```text
1. Create EC2 Instance

2. Get Instance IP

3. Update Route53 Record

4. Repeat For All Components
```

---

# Example Components List

```yaml
components:

  - frontend
  - catalogue
  - user
  - cart
  - shipping
  - payment
  - redis
  - mongodb
  - mysql
  - rabbitmq
```

---

# Dynamic Infrastructure Creation

Instead of:

```yaml
Create Frontend

Create Catalogue

Create User

Create Cart
```

Use:

```yaml
loop
```

---

# Example

```yaml
loop: "{{ components }}"
```

This allows one task to create multiple servers.

---

# AWS Instance Creation

Ansible can communicate with AWS using AWS modules.

Example workflow:

```text
Ansible
     │
     ▼
AWS API
     │
     ▼
EC2 Instance
```

---

# Required Information

Before creating instances:

```text
AMI ID

Instance Type

Security Group

Subnet

AWS Credentials
```

---

# Example Variables

```yaml
ami_id: ami-09c813fb71547fc4f

instance_type: t3.micro

security_group: sg-xxxxxxxx
```

---

# EC2 Creation Workflow

```text
Loop Through Components
          │
          ▼
Create EC2 Instance
          │
          ▼
Capture Instance Details
          │
          ▼
Store Output
```

---

# Register Output

When an EC2 instance is created:

```yaml
register: ec2_info
```

Ansible stores response details.

---

# Example Response

```json
{
  "instance_id": "i-123456",
  "private_ip": "172.31.10.20",
  "public_ip": "54.12.34.56"
}
```

---

# Why Store Output?

We need:

```text
Private IP

Public IP
```

for Route53 updates.

---

# Route53 Automation

After instance creation:

```text
Update DNS Record
```

Example:

```text
frontend.example.com

catalogue.example.com

user.example.com
```

---

# Frontend Special Case

Frontend must be accessible from the Internet.

Therefore:

```text
Use Public IP
```

---

# Example

```text
frontend.example.com

→ Public IP
```

---

# Backend Components

Examples:

```text
catalogue

user

cart

shipping

payment

mongodb

redis

mysql
```

Use:

```text
Private IP
```

because they communicate internally.

---

# Logic

```text
If Component = Frontend

Use Public IP

Else

Use Private IP
```

---

# Conditional Example

```yaml
when: item == "frontend"
```

---

# Route53 Update Flow

```text
Create Instance
       │
       ▼
Get IP Address
       │
       ▼
Update Route53
       │
       ▼
Verify DNS
```

---

# Why Automate DNS?

Manual DNS updates can cause:

```text
Wrong IP

Wrong Record

Deployment Delays
```

Automation eliminates these problems.

---

# Example DNS Records

```text
frontend.example.com

catalogue.example.com

user.example.com

cart.example.com

shipping.example.com
```

---

# Using Facts and Variables

Automation often combines:

```text
Variables

Loops

Conditions

Facts
```

Example:

```yaml
loop: "{{ components }}"
```

Combined with:

```yaml
when: item == "frontend"
```

---

# Infrastructure as Code

This approach is an example of:

```text
Infrastructure as Code (IaC)
```

because infrastructure is defined in code rather than manually created.

---

# Benefits

```text
Repeatable

Version Controlled

Auditable

Consistent

Scalable
```

---

# Real DevOps Workflow

Developer Request:

```text
Need New Shipping Server
```

---

Traditional Process:

```text
Login AWS

Create Instance

Copy IP

Update DNS
```

---

Automated Process:

```text
Update Variables

Run Playbook
```

Done.

---

# Common Mistakes

## Hardcoding Values

Bad:

```yaml
instance_type: t2.micro
```

everywhere.

Use variables instead.

---

## Manual DNS Updates

Causes inconsistencies.

Always automate.

---

## Creating Resources Individually

Bad:

```text
Create Frontend

Create User

Create Cart
```

Prefer loops.

---

## Ignoring Output

Always capture instance information using:

```yaml
register
```

---

# Best Practices

## Use Variables

Store:

```text
AMI IDs

Instance Types

Domain Names
```

in variables.

---

## Use Loops

Avoid duplicate tasks.

---

## Use Conditions

Handle frontend and backend differently.

---

## Automate DNS

Keep infrastructure consistent.

---

## Store Code in Git

Version control all playbooks.

---

# Frequently Asked Interview Questions

## Why Use Ansible for Infrastructure Provisioning?

Ansible provides a simple and agentless way to automate infrastructure creation.

It helps eliminate manual work and ensures consistent environments.

---

## What Information Is Required to Create an EC2 Instance?

Typically:

- AMI ID
- Instance Type
- Security Group
- Subnet
- AWS Credentials

These values are supplied to AWS modules.

---

## Why Do We Register EC2 Output?

After instance creation, AWS returns important information such as:

- Instance ID
- Private IP
- Public IP

This information is required for further automation tasks such as Route53 updates.

---

## Why Does Frontend Use Public IP?

Frontend applications must be accessible from the Internet.

Therefore Route53 points the frontend DNS record to the public IP.

Example:

```text
frontend.example.com → 54.x.x.x
```

---

## Why Do Backend Services Use Private IPs?

Backend services communicate internally and should not be exposed publicly.

Using private IPs improves:

- Security
- Performance
- Network Isolation

---

## How Do Loops Help in Infrastructure Automation?

Loops allow the same task to be executed for multiple components.

Instead of writing:

```text
Create Frontend

Create User

Create Cart
```

one task can handle all components dynamically.

---

## Is This Infrastructure as Code?

Yes.

Infrastructure definitions are stored in code and executed automatically.

Benefits include:

- Repeatability
- Version Control
- Consistency
- Faster Deployments

---

# Key Takeaways

- Roboshop is a multi-tier microservices application.
- Infrastructure provisioning can be automated using Ansible.
- EC2 instances can be created dynamically using loops.
- Route53 records can be updated automatically after instance creation.
- Frontend uses public IPs while backend services typically use private IPs.
- Variables, loops, conditions, and register are heavily used in infrastructure automation.
- Automating infrastructure reduces manual effort and human errors.
- This approach follows Infrastructure as Code principles.