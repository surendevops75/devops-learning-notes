# Terraform Multi-Account Providers and Cross-Account VPC Peering

## Introduction

In real-world organizations, infrastructure is rarely deployed in a single AWS account.

Most companies separate environments into different AWS accounts.

Example:

```text
DEV Account

QA Account

UAT Account

PROD Account
```

---

# Example Accounts

DEV:

```text
752692907119
```

---

PROD:

```text
160885265516
```

---

# Why Multiple Accounts?

Benefits:

```text
Environment Isolation

Security

Cost Management

Compliance

Reduced Risk
```

---

# Example

Development Engineer accidentally runs:

```bash
terraform destroy
```

---

If DEV and PROD are separate accounts:

```text
Only DEV Impacted
```

---

Production remains safe.

---

# How Does Terraform Work With Multiple Accounts?

Terraform communicates with cloud providers using:

```text
Providers
```

---

# Provider Concept

One AWS Account:

```text
One Provider Configuration
```

---

Example:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

# Multiple Accounts

Terraform can use:

```text
Multiple Providers
```

simultaneously.

---

# Example

DEV Provider:

```hcl
provider "aws" {
  alias  = "dev"
  region = "us-east-1"
}
```

---

PROD Provider:

```hcl
provider "aws" {
  alias  = "prod"
  region = "us-east-1"
}
```

---

# Provider Alias

Alias allows Terraform to distinguish:

```text
DEV Account

PROD Account
```

---

# Architecture

```text
Terraform
      ↓
 ┌─────────────┐
 │ Provider DEV│
 └─────────────┘
      ↓
 DEV Account

 ┌──────────────┐
 │Provider PROD │
 └──────────────┘
      ↓
 PROD Account
```

---

# Resource Example

Create VPC in DEV:

```hcl
resource "aws_vpc" "dev" {
  provider = aws.dev
}
```

---

Create VPC in PROD:

```hcl
resource "aws_vpc" "prod" {
  provider = aws.prod
}
```

---

# Multi-Account Use Cases

```text
Shared Services

Cross Account Networking

Centralized Monitoring

Disaster Recovery

Multi Environment Deployments
```

---

# What is VPC Peering?

By default:

```text
One VPC Cannot Communicate
```

with another VPC.

---

Example:

```text
DEV VPC

PROD VPC
```

---

Communication:

```text
Blocked
```

---

# Solution

Create:

```text
VPC Peering Connection
```

---

# VPC Peering Architecture

```text
DEV VPC
     ↓
VPC Peering
     ↓
PROD VPC
```

---

# Components

VPC Peering contains:

```text
Requestor

Acceptor
```

---

# Requestor

Initiates:

```text
Peering Request
```

---

Trainer Example:

```text
DEV VPC
```

acts as:

```text
Requestor
```

---

# Acceptor

Receives and approves:

```text
Peering Request
```

---

Trainer Example:

```text
PROD VPC
```

acts as:

```text
Acceptor
```

---

# Important Interview Point

```text
Peer Owner = Acceptor
```

---

Remember:

```text
Requestor Creates Request

Acceptor Owns Peer
```

---

# Same Region Cross-Account Peering

Example:

```text
DEV Account
       ↓
VPC Peering
       ↓
PROD Account
```

---

Both VPCs:

```text
us-east-1
```

---

Different:

```text
AWS Accounts
```

---

# Terraform Architecture

```text
Terraform
      ↓
DEV Provider
      ↓
DEV VPC

Terraform
      ↓
PROD Provider
      ↓
PROD VPC
```

---

# Cross Account Peering Flow

```text
DEV VPC
      ↓
Peering Request
      ↓
PROD VPC
      ↓
Accept Request
```

---

# Peering Requirements

## Requirement 1

CIDRs Must Not Overlap

Wrong:

```text
DEV  = 10.0.0.0/16

PROD = 10.0.0.0/16
```

---

Correct:

```text
DEV  = 10.0.0.0/16

PROD = 172.31.0.0/16
```

---

# Requirement 2

Routes Must Exist

DEV Route Table:

```text
172.31.0.0/16
      ↓
Peering Connection
```

---

PROD Route Table:

```text
10.0.0.0/16
      ↓
Peering Connection
```

---

# Communication Example

DEV EC2:

```text
10.0.1.10
```

---

Needs access to:

```text
172.31.1.20
```

---

Flow:

```text
DEV Route
      ↓
Peering
      ↓
PROD Route
      ↓
PROD Server
```

---

# Production Use Case

Application:

```text
DEV Account
```

---

Shared Database:

```text
PROD Shared Services Account
```

---

Communication enabled through:

```text
Cross Account VPC Peering
```

---

# Route53 Migration Example

Trainer Note:

```text
R53 -> old infra
```

---

New Infrastructure:

```text
FALB
 ↓
BALB
 ↓
Components
 ↓
Databases
```

---

Migration:

Old Route53 Record

↓

New ALB

↓

New Infrastructure

---

# Complete Architecture

```text
DEV Account
     ↓
DEV VPC
     ↓
VPC Peering
     ↓
PROD VPC
     ↓
PROD Resources
```

---

# Common Interview Questions

## How Can Terraform Manage Multiple AWS Accounts?

### Short Answer

Using multiple provider configurations and aliases.

### Detailed Explanation

Terraform can define multiple AWS providers, each configured with separate credentials and account access.

### Production Example

Managing DEV and PROD accounts from the same Terraform codebase.

### Follow-Up Questions

- What is a provider alias?
- Can one Terraform project manage multiple accounts?

---

## What is VPC Peering?

### Short Answer

VPC Peering enables communication between two VPCs.

### Detailed Explanation

A peering connection allows resources in different VPCs to communicate using private IP addresses.

### Production Example

DEV VPC communicating with PROD Shared Services VPC.

### Follow-Up Questions

- What are the peering requirements?
- Can overlapping CIDRs be peered?

---

## What is the Difference Between Requestor and Acceptor?

### Short Answer

The Requestor initiates the peering request and the Acceptor approves it.

### Detailed Explanation

Cross-account peering requires one VPC to send the request and another VPC to accept it.

### Production Example

DEV VPC as Requestor and PROD VPC as Acceptor.

### Follow-Up Questions

- Who owns the peering connection?
- Can both VPCs be in different accounts?

---

## What are the Requirements for VPC Peering?

### Short Answer

Non-overlapping CIDRs and proper route table configuration.

### Detailed Explanation

Both VPCs must have unique address ranges and routes pointing to the peering connection.

### Production Example

DEV 10.0.0.0/16 peered with PROD 172.31.0.0/16.

### Follow-Up Questions

- What happens if CIDRs overlap?
- Are route tables required?

---

# Key Takeaways

```text
Organizations commonly use multiple AWS accounts.

Terraform supports multiple accounts using provider aliases.

Each AWS account typically has its own provider configuration.

VPC Peering enables communication between VPCs.

Cross-account peering is commonly used in enterprise environments.

Peering involves a Requestor and an Acceptor.

The Acceptor is considered the Peer Owner.

CIDR ranges must not overlap.

Route tables must be updated for peering communication.

Terraform can manage resources across multiple AWS accounts from a single codebase.
```