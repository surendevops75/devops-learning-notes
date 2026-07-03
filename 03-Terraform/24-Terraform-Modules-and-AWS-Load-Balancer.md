# Terraform Modules and AWS Load Balancer

## Infrastructure Categories

Generally infrastructure is divided into two categories.

### Project Infrastructure

Created once and rarely changes.

Examples:

```text
VPC

Subnets

Route Tables

Internet Gateway

NAT Gateway

Load Balancers

Security Groups
```

Characteristics:

```text
One Time Setup

Rare Changes

Shared By Multiple Applications
```

---

### Application Infrastructure

Changes frequently.

Examples:

```text
Catalogue

User

Cart

Shipping

Payment
```

Characteristics:

```text
Frequent Changes

New Features

New Deployments

Scaling Activities
```

---

# Production Example

Project Infra:

```text
VPC

ALB

NAT Gateway

Security Groups
```

created once.

---

Application Infra:

```text
Catalogue EC2

Shipping EC2

Payment EC2
```

can be created, modified, or destroyed frequently.

---

# Terraform Module Approaches

Terraform supports three approaches.

---

## 1. Custom Modules

Developed internally.

Example:

```text
terraform-vpc-module

terraform-ec2-module

terraform-alb-module
```

---

### Advantages

```text
Full Control

Custom Standards

Reusable

Governance Compliance
```

---

### Disadvantages

```text
Need Development

Need Maintenance
```

---

### Production Example

Organization creates:

```text
terraform-vpc-module
```

Every team uses the same approved module.

---

## 2. Open Source Modules

Available from:

```text
Terraform Registry
```

Examples:

```text
terraform-aws-modules/vpc/aws

terraform-aws-modules/eks/aws
```

---

### Advantages

```text
Quick Setup

Community Tested

Less Development
```

---

### Disadvantages

```text
Limited Control

External Dependency
```

---

### Production Example

Instead of developing EKS from scratch:

```text
terraform-aws-modules/eks/aws
```

can be directly used.

---

## 3. Direct Resource Creation

Example:

```hcl
resource "aws_instance" "catalogue" {

}
```

---

### Advantages

```text
Simple

Easy To Learn
```

---

### Disadvantages

```text
Code Duplication

No Reusability

Difficult Maintenance
```

---

### Production Example

Creating:

```text
Frontend

Catalogue

User

Cart

Shipping
```

using separate EC2 resources.

Every modification must be repeated multiple times.

---

# Understanding Load Balancer Using Organization Structure

Suppose a client requests:

```text
Enable WhatsApp Banking
```

Request Flow:

```text
Client
   ↓
Business Analyst
   ↓
Delivery Manager
   ↓
Frontend Manager
   ↓
Frontend Team Lead
   ↓
Frontend Team Members
```

---

# AWS Analogy

Organization:

```text
Delivery Manager
```

Equivalent To:

```text
Load Balancer
```

---

Organization:

```text
Frontend Team
```

Equivalent To:

```text
Target Group
```

---

Organization:

```text
Team Member
```

Equivalent To:

```text
EC2 Instance
```

---

# Mapping

```text
Delivery Manager  → Load Balancer

Team              → Target Group

Team Member       → EC2 Instance
```

---

# What is a Load Balancer?

A Load Balancer receives requests and forwards them to the appropriate backend servers.

Responsibilities:

```text
Traffic Distribution

High Availability

Health Monitoring

Routing
```

---

# Load Balancer Components

```text
Load Balancer

Listeners

Rules

Target Groups

Health Checks

Instances
```

---

# Listener

A listener waits for incoming requests on a specific port.

Examples:

```text
80

443

8080
```

---

# Public Load Balancer

Usually used for:

```text
Frontend Applications
```

Listeners:

```text
80

443
```

---

# Private Load Balancer

Usually used for:

```text
Backend Services
```

Listener:

```text
8080
```

---

# Port Examples

HTTP:

```text
http://daws86s.fun
```

Default Port:

```text
80
```

---

HTTPS:

```text
https://daws86s.fun
```

Default Port:

```text
443
```

---

Custom Port:

```text
http://daws86s.fun:90
```

Port:

```text
90
```

---

# Rules

Rules determine where requests should be forwarded.

Examples:

```text
catalogue.daws86s.fun

→ Catalogue Target Group
```

---

```text
shipping.daws86s.fun

→ Shipping Target Group
```

---

# Target Group

A Target Group contains backend servers.

Example:

```text
Catalogue Target Group
```

Members:

```text
Catalogue-1

Catalogue-2

Catalogue-3
```

---

# Target Group Analogy

```text
Frontend Team
```

contains:

```text
Employee-1

Employee-2

Employee-3
```

Similarly:

```text
Target Group
```

contains:

```text
EC2-1

EC2-2

EC2-3
```

---

# Health Check

Health Checks verify whether an application is available.

Example:

```text
http://catalogue.daws86s.fun:8080/health
```

---

# Health Check Process

Every few seconds:

```text
Load Balancer
      ↓
Health Check URL
      ↓
Application Response
```

---

If response received:

```text
Healthy
```

---

If multiple failures occur:

```text
Unhealthy
```

---

# Production Example

Health Check URL:

```text
http://catalogue.daws86s.fun:8080/health
```

Successful Response:

```text
200 OK
```

Target Status:

```text
Healthy
```

---

Three consecutive failures:

```text
Target Status

↓

Unhealthy
```

Load Balancer stops sending traffic.

---

# Traffic Flow

```text
User
   ↓
Load Balancer
   ↓
Rule Match
   ↓
Target Group
   ↓
Healthy EC2 Instance
```

---

# Common Interview Questions

## What is a Load Balancer?

### Short Answer

A Load Balancer distributes incoming traffic across multiple backend servers.

### Detailed Explanation

It improves availability, scalability, and fault tolerance by routing requests only to healthy instances.

### Production Example

Frontend traffic distributed across multiple Catalogue servers.

---

## What is a Target Group?

### Short Answer

A Target Group is a collection of backend instances.

### Detailed Explanation

Load Balancers forward requests to target groups based on listener rules.

### Production Example

Catalogue Target Group containing three Catalogue EC2 instances.

---

## What is a Health Check?

### Short Answer

A Health Check verifies whether an application is available.

### Detailed Explanation

The Load Balancer periodically checks application endpoints and removes unhealthy targets from traffic routing.

### Production Example

```text
http://catalogue.daws86s.fun:8080/health
```

returns:

```text
200 OK
```

---

## What is a Listener?

### Short Answer

A Listener waits for requests on a specific port.

### Detailed Explanation

Listeners receive traffic and evaluate routing rules before forwarding requests.

### Production Example

Frontend ALB listening on:

```text
80

443
```

Backend ALB listening on:

```text
8080
```

---

# Terraform String Functions

Sometimes Terraform requires conversion between strings and lists.

---

# join()

Converts:

```text
List → String
```

---

Example

```hcl
join(",", ["siva","kumar","reddy","m"])
```

Output:

```text
siva,kumar,reddy,m
```

---

# Production Example

Subnet IDs:

```text
subnet-0b74d14411866ada1

subnet-01c3ab188edc1bb04
```

Terraform:

```hcl
join(",", var.subnet_ids)
```

Output:

```text
subnet-0b74d14411866ada1,subnet-01c3ab188edc1bb04
```

---

# split()

Converts:

```text
String → List
```

---

Example

```hcl
split(",", "siva,kumar,reddy,m")
```

Output:

```text
["siva","kumar","reddy","m"]
```

---

# Production Example

Input:

```text
subnet-1,subnet-2
```

Terraform:

```hcl
split(",", var.subnet_string)
```

Output:

```text
["subnet-1","subnet-2"]
```

---

# Key Takeaways

```text
Project Infrastructure changes rarely.

Application Infrastructure changes frequently.

Terraform supports Custom Modules, Open Source Modules, and Direct Resources.

Load Balancer distributes traffic.

Target Groups contain EC2 instances.

Health Checks determine instance availability.

Listeners receive requests on specific ports.

Rules forward traffic to appropriate target groups.

join() converts List to String.

split() converts String to List.
```