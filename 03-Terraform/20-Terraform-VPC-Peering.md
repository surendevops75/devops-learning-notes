# Terraform Project - VPC Peering

## Introduction

In the previous section we created:

```text
VPC

Internet Gateway

Subnets

Route Tables

Elastic IP

NAT Gateway

Private Routes
```

for the Roboshop infrastructure.

Now let's discuss communication between multiple VPCs.

---

# Problem Statement

Suppose we have:

```text
VPC-1

10.0.0.0/16
```

and

```text
VPC-2

172.31.0.0/16
```

Question:

```text
Can EC2 instances in VPC-1 communicate with EC2 instances in VPC-2?
```

Answer:

```text
No
```

By default:

```text
One VPC Cannot Communicate

With Another VPC
```

---

# Why?

Every VPC is:

```text
Logically Isolated
```

AWS intentionally isolates VPCs for:

```text
Security

Network Separation

Traffic Isolation
```

---

# Solution

AWS provides:

```text
VPC Peering
```

---

# What is VPC Peering?

VPC Peering is:

```text
A Private Network Connection

Between Two VPCs
```

allowing resources to communicate using:

```text
Private IP Addresses
```

---

# Real World Analogy

Village 1:

```text
10.0.0.0/16
```

Village 2:

```text
172.31.0.0/16
```

Initially:

```text
No Road Exists
```

between villages.

VPC Peering creates:

```text
Road Between Villages
```

---

# Architecture

Before Peering:

```text
VPC-1
  ×
VPC-2
```

Communication:

```text
Not Possible
```

---

After Peering:

```text
VPC-1
  ↔
VPC-2
```

Communication:

```text
Possible
```

---

# VPC Peering Requirements

## Requirement 1

CIDR Blocks Must Not Overlap.

---

Valid Example

VPC-1

```text
10.0.0.0/16
```

VPC-2

```text
172.31.0.0/16
```

Allowed:

```text
Yes
```

---

Invalid Example

VPC-1

```text
10.0.0.0/16
```

VPC-2

```text
10.0.0.0/16
```

Allowed:

```text
No
```

---

# Why Overlapping CIDRs Are Not Allowed?

Suppose:

```text
10.0.1.10
```

exists in both VPCs.

AWS cannot determine:

```text
Which Destination
```

should receive traffic.

---

# Requirement 2

Routes Must Be Configured.

Creating peering alone is not enough.

---

# Example

VPC-A

```text
10.0.0.0/16
```

VPC-B

```text
172.31.0.0/16
```

---

# Route in VPC-A

Destination:

```text
172.31.0.0/16
```

Target:

```text
Peering Connection
```

---

# Route in VPC-B

Destination:

```text
10.0.0.0/16
```

Target:

```text
Peering Connection
```

---

# Traffic Flow

```text
EC2
  ↓
Route Table
  ↓
VPC Peering
  ↓
Destination VPC
```

---

# Types of VPC Peering

## Same Region Peering

Example:

```text
VPC-A → us-east-1

VPC-B → us-east-1
```

---

Communication:

```text
Supported
```

---

# Cross Region Peering

Example:

```text
VPC-A → us-east-1

VPC-B → us-west-2
```

---

Communication:

```text
Supported
```

---

# Cross Account Peering

Example:

```text
AWS Account A

AWS Account B
```

---

Communication:

```text
Supported
```

after approval.

---

# Types Supported by AWS

```text
Same Region

Cross Region

Cross Account

Cross Region + Cross Account
```

---

# Example Scenario

Roboshop Infrastructure:

```text
VPC-A

10.0.0.0/16
```

Shared Services VPC:

```text
172.31.0.0/16
```

Contains:

```text
Monitoring

Logging

Security Tools
```

Need communication between:

```text
Roboshop

Shared Services
```

Solution:

```text
VPC Peering
```

---

# Example Private IPs

VPC-A

```text
10.0.1.25
```

---

VPC-B

```text
172.31.1.221
```

---

After Peering:

```text
10.0.1.25

Can Reach

172.31.1.221
```

---

# Common Mistakes

## Overlapping CIDRs

Bad:

```text
10.0.0.0/16

10.0.0.0/16
```

---

## Missing Route Entries

Peering exists.

Routes missing.

Result:

```text
Communication Failure
```

---

## Security Groups Blocking Traffic

Routes are correct.

Peering exists.

Still communication fails.

Reason:

```text
Security Group Rules
```

---

# VPC Peering vs Internet

Communication happens using:

```text
Private Network
```

Not through:

```text
Internet Gateway
```

---

# Frequently Asked Interview Questions

## What is VPC Peering?

### Short Answer

VPC Peering is a private connection between two VPCs that enables communication using private IP addresses.

### Detailed Explanation

VPC Peering allows resources in separate VPCs to communicate as if they were part of the same network, provided routing and security rules are configured properly.

### Production Example

Connecting an application VPC to a shared services VPC hosting monitoring and logging systems.

---

## What Are the Requirements for VPC Peering?

### Short Answer

Non-overlapping CIDRs and proper route table entries.

### Detailed Explanation

AWS requires unique network ranges between peered VPCs and explicit routing to direct traffic through the peering connection.

### Production Example

10.0.0.0/16 communicating with 172.31.0.0/16 through a peering connection.

---

## Can Two VPCs with the Same CIDR Be Peered?

### Short Answer

No.

### Detailed Explanation

Overlapping CIDR ranges create routing ambiguity and AWS does not allow such peering configurations.

---

## Does VPC Peering Require Internet Gateway?

### Short Answer

No.

### Detailed Explanation

Peering uses AWS private networking and does not depend on public internet connectivity.

---

# Key Takeaways

```text
VPCs are isolated by default.

VPC Peering enables private communication.

CIDRs must not overlap.

Routes must be configured on both sides.

Peering works across regions and accounts.

Communication uses private IP addresses.

Security Groups and Routes must allow traffic.
```