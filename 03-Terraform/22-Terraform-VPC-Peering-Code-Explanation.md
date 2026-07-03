# Terraform VPC Peering Code Explanation

## Introduction

By default:

```text
One VPC Cannot Communicate

With Another VPC
```

AWS isolates VPCs for:

```text
Security

Traffic Isolation

Network Separation
```

To enable communication between VPCs, AWS provides:

```text
VPC Peering
```

---

# Scenario

Custom Roboshop VPC:

```text
10.0.0.0/16
```

Default AWS VPC:

```text
172.31.0.0/16
```

Requirement:

```text
Roboshop VPC

↔

Default VPC
```

Communication should be possible.

---

# Architecture

Before Peering:

```text
Roboshop VPC
      X
Default VPC
```

Communication:

```text
Not Possible
```

---

After Peering:

```text
Roboshop VPC
      ↔
Default VPC
```

Communication:

```text
Possible
```

---

# VPC Peering Resource

Terraform Resource:

```hcl
resource "aws_vpc_peering_connection" "default"
```

Purpose:

```text
Creates a Peering Connection

Between Two VPCs
```

---

# Conditional Creation

```hcl
count = var.is_peering_required ? 1 : 0
```

Terraform Concept:

```text
Conditional Expression
```

Meaning:

If:

```text
is_peering_required = true
```

Create:

```text
1 Peering Connection
```

---

If:

```text
is_peering_required = false
```

Create:

```text
No Peering Connection
```

---

# Interview Question

Why use count here?

Answer:

```text
To create the resource conditionally.
```

---

# Acceptor VPC

```hcl
peer_vpc_id = data.aws_vpc.default.id
```

Represents:

```text
Default VPC
```

This VPC receives the peering request.

Also called:

```text
Acceptor
```

---

# Requester VPC

```hcl
vpc_id = aws_vpc.main.id
```

Represents:

```text
Roboshop VPC
```

This VPC initiates the peering request.

Also called:

```text
Requester
```

---

# Visual Representation

```text
Requester

Roboshop VPC
10.0.0.0/16

        ↓

Acceptor

Default VPC
172.31.0.0/16
```

---

# DNS Resolution

Acceptor Block:

```hcl
accepter {

  allow_remote_vpc_dns_resolution = true

}
```

Purpose:

```text
Enable DNS Resolution

Across Peered VPCs
```

---

# Example

Without DNS Resolution:

```text
Application Cannot Resolve

Private DNS Names
```

---

With DNS Resolution:

```text
Private Hostnames

Can Be Resolved
```

---

# Requester Block

```hcl
requester {

  allow_remote_vpc_dns_resolution = true

}
```

Purpose:

```text
Enable DNS Resolution

From Requester Side
```

---

# Auto Accept

```hcl
auto_accept = true
```

Purpose:

```text
Automatically Accept

The Peering Request
```

---

Without Auto Accept:

```text
Manual Approval Required
```

---

# Tags

```hcl
tags = merge(
  var.vpc_tags,
  local.common_tags,
  {
    Name = "${local.common_name_suffix}-default"
  }
)
```

Purpose:

```text
Apply Standard Tags

To Peering Connection
```

---

# Why Peering Alone Is Not Enough?

Many engineers think:

```text
Create Peering

Done
```

Wrong.

You also need:

```text
Routes
```

---

# Route Requirement

Suppose:

Roboshop VPC

```text
10.0.0.0/16
```

Default VPC

```text
172.31.0.0/16
```

Each VPC must know:

```text
How To Reach

The Other Network
```

---

# Public Route Peering

Terraform Resource:

```hcl
resource "aws_route" "public_peering"
```

Purpose:

```text
Allow Public Subnets

To Reach Default VPC
```

---

# Destination

```hcl
destination_cidr_block =
data.aws_vpc.default.cidr_block
```

Value:

```text
172.31.0.0/16
```

Meaning:

```text
Traffic Destined

For Default VPC
```

---

# Target

```hcl
vpc_peering_connection_id =
aws_vpc_peering_connection.default[count.index].id
```

Meaning:

```text
Send Traffic

Through Peering Connection
```

---

# Private Route Peering

Terraform Resource:

```hcl
resource "aws_route" "private_peering"
```

Purpose:

```text
Allow Private Subnets

To Reach Default VPC
```

---

# Why Private Route?

Application servers are usually deployed in:

```text
Private Subnets
```

They also need access to:

```text
Default VPC Resources
```

---

# Route Flow

```text
Private EC2
      ↓
Private Route Table
      ↓
Peering Connection
      ↓
Default VPC
```

---

# Default VPC Route

Terraform Resource:

```hcl
resource "aws_route" "default_peering"
```

Purpose:

```text
Allow Default VPC

To Reach Roboshop VPC
```

---

# Route

```hcl
destination_cidr_block = var.vpc_cidr
```

Example:

```text
10.0.0.0/16
```

---

# Route Table

```hcl
route_table_id =
data.aws_route_table.main.id
```

Represents:

```text
Default VPC Main Route Table
```

---

# Why Add Route On Both Sides?

Wrong:

```text
Roboshop → Default
```

Only one direction.

---

Correct:

```text
Roboshop ↔ Default
```

Both directions.

---

# Communication Flow

```text
10.0.1.10
      ↓
Peering
      ↓
172.31.1.221
```

---

Response:

```text
172.31.1.221
      ↓
Peering
      ↓
10.0.1.10
```

---

# Important Requirement

CIDRs Must Not Overlap.

---

Valid:

```text
10.0.0.0/16

172.31.0.0/16
```

---

Invalid:

```text
10.0.0.0/16

10.0.0.0/16
```

AWS will reject peering.

---

# Terraform Concepts Used

## Conditional Resource Creation

```hcl
count = var.is_peering_required ? 1 : 0
```

---

## Data Sources

```hcl
data.aws_vpc.default
```

Used to fetch:

```text
Existing Default VPC
```

---

## count.index

```hcl
aws_vpc_peering_connection.default[count.index]
```

Used because the resource is created using count.

---

## merge()

```hcl
merge(...)
```

Used for tagging.

---

# Common Interview Questions

## What is VPC Peering?

### Short Answer

VPC Peering enables private communication between two VPCs.

### Detailed Explanation

VPC Peering creates a private network connection between VPCs allowing resources to communicate using private IP addresses.

---

## What Are The Requirements For VPC Peering?

### Short Answer

Non-overlapping CIDRs and route table entries.

### Detailed Explanation

Peering alone is not sufficient. Route tables on both sides must contain routes pointing to the peering connection.

---

## Why Are Routes Required On Both Sides?

### Short Answer

Communication must be bidirectional.

### Detailed Explanation

Both VPCs must know how to reach each other's CIDR ranges.

---

## What Does auto_accept = true Do?

### Short Answer

Automatically accepts the peering request.

### Detailed Explanation

Without auto_accept, the peering request must be manually approved.

---

## Why Enable DNS Resolution?

### Short Answer

To allow hostname resolution across peered VPCs.

### Detailed Explanation

DNS resolution enables services in one VPC to resolve private DNS names of resources in another peered VPC.

---

# Key Takeaways

```text
VPCs are isolated by default.

VPC Peering enables private communication.

CIDRs must not overlap.

Routes must be configured on both sides.

Peering works using private IP addresses.

DNS resolution can be enabled across VPCs.

auto_accept automatically approves requests.

Peering is commonly used for shared services, monitoring, and migration projects.
```