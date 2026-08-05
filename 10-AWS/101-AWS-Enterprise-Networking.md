# 101 - AWS Enterprise Networking

# Chapter 1 - Enterprise Networking Fundamentals

## What is Enterprise Networking?

Enterprise Networking is the architecture, design, implementation, and management of communication between cloud resources distributed across multiple AWS accounts, VPCs, Availability Zones, AWS Regions, on-premises data centers, and third-party services.

Unlike traditional networking, enterprise cloud networking focuses on scalability, security, automation, high availability, and centralized management.

A small AWS environment might contain only one VPC and a few EC2 instances, while an enterprise environment may include:

- Hundreds of AWS Accounts
- Hundreds of VPCs
- Thousands of EC2 Instances
- Multiple Kubernetes Clusters
- Shared Services
- Hybrid Cloud
- Disaster Recovery Regions
- Centralized Security

Enterprise networking provides the foundation that allows all these resources to communicate efficiently while maintaining isolation and security.

---

## Why Enterprise Networking?

As organizations grow, networking becomes increasingly complex.

Challenges include:

- Connecting multiple VPCs
- Connecting multiple AWS accounts
- Hybrid cloud connectivity
- Shared infrastructure
- Centralized monitoring
- Secure communication
- Regulatory compliance
- Disaster recovery
- Scalability
- Cost optimization

Without a proper networking strategy, the environment becomes difficult to operate, expensive to maintain, and challenging to secure.

---

## Enterprise Network Evolution

### Stage 1 — Single Application

```text
               Internet
                   │
          Internet Gateway
                   │
              Public Subnet
                   │
                  ALB
                   │
              Private Subnet
                   │
                  EC2
                   │
                  RDS
```

Suitable for:

- Learning
- Small applications
- Startup MVPs

---

### Stage 2 — Multiple Applications

```text
                Internet

                    │

             Internet Gateway

                    │

          ┌─────────┴─────────┐

        VPC-A             VPC-B

           ↕ VPC Peering ↕

        Shared Communication
```

Suitable for:

- Small companies
- Independent application teams
- Separate environments

---

### Stage 3 — Enterprise Cloud

```text
                 AWS Organization

     ┌─────────────┬──────────────┬─────────────┐
     │             │              │
 Security      Shared Services   Logging
 Account         Account          Account

                 │
          Transit Gateway
                 │
    ┌────────────┼────────────┐
    │            │            │
 Dev VPC      QA VPC      Prod VPC
    │            │            │
    └────────────┼────────────┘
                 │
         Direct Connect
                 │
         On-Premises DC
```

Suitable for:

- Enterprises
- Banks
- Healthcare
- Government
- SaaS Platforms

---

# Enterprise Networking Goals

A good enterprise network should achieve the following objectives.

## Scalability

The architecture should support growth without requiring redesign.

Example

Today

```text
5 VPCs
```

Future

```text
300 VPCs
```

Networking should continue functioning with minimal operational effort.

---

## Security

Only authorized systems should communicate.

Examples

- Security Groups
- Network ACLs
- IAM
- PrivateLink
- AWS Network Firewall
- VPC Endpoints

---

## High Availability

Avoid single points of failure.

Examples

- Multiple Availability Zones
- Redundant VPN tunnels
- Multiple NAT Gateways
- Multi-Region design

---

## Centralized Management

Instead of managing hundreds of independent connections, networking should be centrally managed.

Typical centralized components

- Transit Gateway
- Shared DNS
- Shared Logging
- Shared Monitoring
- Shared CI/CD

---

## Cost Optimization

Enterprise networking should reduce unnecessary cost.

Examples

Good

```text
100 VPCs

↓

Transit Gateway
```

Poor

```text
100 VPCs

↓

4,950 VPC Peering Connections
```

---

# Core AWS Networking Components

| Component | Purpose |
|------------|----------|
| VPC | Isolated Virtual Network |
| Subnet | Network segmentation |
| Route Table | Packet routing |
| Internet Gateway | Internet access |
| NAT Gateway | Outbound internet |
| Security Group | Stateful firewall |
| Network ACL | Stateless firewall |
| Elastic IP | Static public IP |
| ENI | Virtual Network Interface |
| Transit Gateway | Central Router |
| VPC Peering | Point-to-point connection |
| PrivateLink | Private Service Access |
| VPC Endpoint | AWS Service Access |
| Direct Connect | Dedicated Connection |
| Site-to-Site VPN | Encrypted Tunnel |

---

# AWS Enterprise Networking Services

AWS offers multiple networking options.

Choosing the correct service is critical.

| Requirement | Recommended Service |
|--------------|--------------------|
| Connect two VPCs | VPC Peering |
| Connect hundreds of VPCs | Transit Gateway |
| Private S3 Access | Gateway Endpoint |
| Private AWS API Access | Interface Endpoint |
| Third-party SaaS Access | PrivateLink |
| Hybrid Connectivity | Direct Connect |
| Backup Hybrid Connectivity | Site-to-Site VPN |
| Cross-Account Resource Sharing | AWS RAM |

---

# Enterprise Networking Decision Tree

```text
Need Connectivity?

│

├── Same VPC

│      │

│      └── Route Tables

│

├── Two VPCs

│      │

│      └── VPC Peering

│

├── Multiple VPCs

│      │

│      └── Transit Gateway

│

├── On-Premises

│      │

│      ├── VPN

│      └── Direct Connect

│

├── AWS Service

│      │

│      └── VPC Endpoint

│

└── Third Party Service

       │

       └── PrivateLink
```

---

# Enterprise Design Principles

## Design Small, Scale Big

Don't design only for today's environment.

Always assume future expansion.

---

## Avoid Overlapping CIDRs

Bad Example

```text
Dev

10.0.0.0/16

QA

10.0.0.0/16
```

Cannot be peered later.

Good Example

```text
Dev

10.10.0.0/16

QA

10.20.0.0/16

Prod

10.30.0.0/16
```

---

## Keep Applications Private

Only Load Balancers should normally be public.

Application

```text
Public

ALB

↓

Private EC2

↓

Private Database
```

---

## Separate Environments

Never place Dev and Production in the same VPC.

Good

```text
Dev VPC

QA VPC

Production VPC
```

---

## Centralize Shared Services

Instead of installing Jenkins everywhere

Good

```text
Shared Services VPC

├── Jenkins

├── SonarQube

├── Artifactory

├── Grafana

├── Prometheus

└── ELK
```

Every environment consumes these services.

---

# Common Enterprise Networking Patterns

## Hub and Spoke

```text
             Transit Gateway

      ┌────────┼────────┐

     Dev      QA      Prod

           Shared Services
```

Advantages

- Easy Routing
- Central Security
- Easy Expansion

---

## Shared Services Architecture

```text
Shared Services VPC

↓

DNS

↓

CI/CD

↓

Monitoring

↓

Logging

↓

Authentication
```

Every application VPC communicates with Shared Services instead of hosting duplicate infrastructure.

---

## Centralized Internet Egress

Instead of every VPC having its own internet path

```text
Application VPCs

↓

Transit Gateway

↓

Egress VPC

↓

NAT Gateway

↓

Internet
```

Benefits

- Better Security
- Central Monitoring
- Reduced Operational Complexity

---

## Production Example

A multinational retail company operates:

- 120 AWS Accounts
- 250 VPCs
- 35 Amazon EKS Clusters
- Shared Jenkins
- Shared Prometheus
- Shared ELK
- Two On-Premises Data Centers
- Primary Region: Mumbai
- Disaster Recovery Region: Singapore

Instead of creating thousands of VPC peering connections, the company uses:

- AWS Organizations
- Transit Gateway
- Shared Services VPC
- AWS RAM
- Direct Connect
- Site-to-Site VPN as backup
- PrivateLink for internal platform services
- VPC Endpoints for S3 and Secrets Manager

This architecture reduces operational complexity, centralizes security controls, and supports future expansion without redesign.

---

# Chapter 2 - VPC Peering

## What is VPC Peering?

VPC Peering creates a private network connection between two VPCs, allowing resources in each VPC to communicate using private IP addresses.

Traffic never traverses the public internet. Communication stays on the AWS backbone network, providing secure and low-latency connectivity.

VPC Peering is ideal for connecting a small number of VPCs, such as development and testing environments or two related applications.

## Types of VPC Peering

AWS supports multiple types of VPC Peering depending on the location of the VPCs and AWS accounts.

### 1. Same Account - Same Region

Both VPCs belong to the same AWS account and are located in the same AWS Region.

```text
AWS Account

├── VPC-A
│
└────── VPC Peering ──────┐
                           │
                      VPC-B
```

Example

- Application VPC
- Database VPC

Advantages

- Simple configuration
- Lowest latency
- Easy management

Typical Use Cases

- Development & Testing
- Application Separation
- Shared Database

---

### 2. Cross-Account Peering

VPCs belong to different AWS accounts.

```text
Account-A

VPC-A

↓

Peering Request

↓

Account-B

VPC-B
```

Requirements

- Both accounts must approve the request.
- Non-overlapping CIDRs.
- Route table updates.
- Security Group updates.

Typical Use Cases

- Shared Services
- Partner Integrations
- Multi-Account AWS Organizations

---

### 3. Cross-Region Peering

VPCs exist in different AWS Regions.

```text
Mumbai Region

VPC-A

↓

Cross Region Peering

↓

Singapore Region

VPC-B
```

Typical Use Cases

- Disaster Recovery
- Global Applications
- Regional Data Sharing

Advantages

- AWS Backbone Network
- Private Communication
- Lower latency than Internet

---

# How VPC Peering Works

Consider two VPCs.

```text
VPC-A

CIDR

10.10.0.0/16

↓

Peering Connection

↓

VPC-B

CIDR

10.20.0.0/16
```

When an EC2 instance inside VPC-A sends traffic to 10.20.0.10

Flow

```text
EC2

↓

Route Table

↓

VPC Peering

↓

Destination VPC

↓

EC2
```

Traffic never leaves the AWS backbone.

---

# VPC Peering Lifecycle

```text
Create Request

↓

Pending Acceptance

↓

Accepted

↓

Active

↓

Configure Routes

↓

Configure Security Groups

↓

Communication Established
```

---

# Step-by-Step Configuration

## Step 1

Create VPC Peering Request

AWS Console

↓

VPC

↓

Peering Connections

↓

Create Peering Connection

Specify

- Requester VPC
- Accepter VPC

---

## Step 2

Accept Request

The accepter VPC owner accepts the request.

Status changes

```text
Pending

↓

Active
```

---

## Step 3

Update Route Tables

Example

VPC-A Route Table

| Destination | Target |
|-------------|--------|
|10.20.0.0/16|Peering Connection|

VPC-B Route Table

| Destination | Target |
|-------------|--------|
|10.10.0.0/16|Peering Connection|

Without these routes, traffic cannot flow.

---

## Step 4

Update Security Groups

Allow required traffic.

Example

Application Server

```text
Inbound

TCP 8080

Source

10.20.0.0/16
```

Remember

Even if routing is correct, Security Groups can still block communication.

---

# DNS Resolution Across Peering

By default

```text
Private DNS

↓

Does NOT resolve
```

Enable

```text
DNS Resolution

DNS Hostnames
```

This allows instances to resolve private hostnames across peered VPCs.

Example

Instead of

```
10.20.5.10
```

Application can use

```
db.internal.company.local
```

---

# Security in VPC Peering

Traffic remains

- Private
- Encrypted on AWS Backbone (AWS-managed infrastructure)

Control communication using

- Security Groups
- Network ACLs
- IAM
- Route Tables

Best Practice

Allow only required ports.

Bad

```text
0.0.0.0/0
```

Good

```text
10.20.0.0/16
```

---

# Advantages of VPC Peering

- Low latency
- High bandwidth
- No internet required
- Private communication
- Simple setup
- Cost effective for small environments

---

# Limitations of VPC Peering

This is where many interviewers focus.

## 1. No Transitive Routing

Suppose

```text
VPC-A

↓

Peering

↓

VPC-B

↓

Peering

↓

VPC-C
```

Can A communicate with C?

❌ No

Traffic cannot pass through another VPC.

AWS intentionally blocks transitive routing.

If A needs to communicate with C

Create another peering

```text
A

↓

Peering

↓

C
```

Or use Transit Gateway.

---

## 2. Full Mesh Problem

Imagine

10 VPCs

Each VPC must connect with every other VPC.

Number of peerings

```
45
```

100 VPCs

```
4950
```

Managing this becomes almost impossible.

This is one of the biggest reasons Transit Gateway exists.

---

## 3. CIDR Overlap Not Supported

Example

VPC-A

```
10.0.0.0/16
```

VPC-B

```
10.0.0.0/16
```

Peering cannot be created.

Always plan CIDR ranges before deployment.

---

## 4. Separate Route Tables

Every VPC must maintain its own routes.

Large environments become operationally expensive.

---

## 5. No Centralized Management

Each peering is managed independently.

There is no central routing hub.

---

# Production Use Cases

## Shared Database

```text
Application VPC

↓

Peering

↓

Database VPC
```

---

## Shared Authentication

```text
Application VPC

↓

Peering

↓

Authentication VPC
```

---

## Development Access

```text
Developer Tools

↓

Peering

↓

Testing Environment
```

---

## Temporary Migration

During cloud migration

```text
Old Application

↓

Peering

↓

New Application
```

Both applications communicate until migration is complete.

---

# VPC Peering vs Transit Gateway

| Feature | VPC Peering | Transit Gateway |
|----------|-------------|-----------------|
| Best For | Few VPCs | Hundreds of VPCs |
| Transitive Routing | ❌ | ✅ |
| Central Management | ❌ | ✅ |
| Multi-Account | ✅ | ✅ |
| Hybrid Cloud | Limited | Excellent |
| Operational Complexity | High at Scale | Low |
| Cost | Lower for few VPCs | Better at enterprise scale |

---

# Best Practices

- Plan CIDR blocks before creating VPCs.
- Avoid overlapping address ranges.
- Use descriptive names for peering connections.
- Enable DNS resolution if applications use private hostnames.
- Keep Security Groups restrictive.
- Remove unused peering connections.
- Use Transit Gateway when the number of VPCs grows.

---

# Common Troubleshooting

| Problem | Possible Cause | Resolution |
|----------|---------------|------------|
| Cannot ping another VPC | Missing Route | Update Route Table |
| Connection timeout | Security Group | Allow required port |
| DNS not resolving | DNS disabled | Enable DNS resolution |
| Peering creation failed | Overlapping CIDRs | Redesign CIDRs |
| Traffic still blocked | NACL | Verify Network ACL rules |

---

# Interview Questions

## Basic

- What is VPC Peering?
- Can VPC Peering use the internet?
- How many VPCs can be connected?

## Intermediate

- Explain Cross-Account Peering.
- Explain Cross-Region Peering.
- Why can't overlapping CIDRs be peered?
- Why are route tables required?

## Advanced

- Why is VPC Peering not suitable for enterprise environments?
- Explain transitive routing with an example.
- At what point would you migrate from VPC Peering to Transit Gateway?
- Design networking for 80 VPCs.
- Compare VPC Peering, Transit Gateway, and PrivateLink.