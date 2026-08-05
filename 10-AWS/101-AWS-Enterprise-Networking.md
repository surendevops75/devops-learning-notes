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
