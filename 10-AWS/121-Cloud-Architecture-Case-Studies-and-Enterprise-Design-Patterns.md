# Cloud Architecture Case Studies & Enterprise Design Patterns

# Chapter 1 - Enterprise Cloud Architecture Fundamentals

Modern enterprises rarely deploy applications on a single server.

Today's cloud platforms are designed around:

- High Availability
- Scalability
- Fault Tolerance
- Security
- Automation
- Observability
- Disaster Recovery

Cloud architecture is about designing systems that continue to operate even when components fail.

---

# Why Cloud Architecture Matters

Traditional Infrastructure:

```text
Application

↓

Single Server

↓

Single Database

↓

Failure

↓

Application Down
```

Cloud Architecture:

```text
Users

↓

Load Balancer

↓

Application Servers

↓

Database Cluster

↓

Storage

↓

Monitoring

↓

Backup
```

No single component should become a single point of failure.

---

# Characteristics of Enterprise Cloud Architecture

A production-grade architecture should provide:

- High Availability
- Elastic Scalability
- Fault Isolation
- Security
- Cost Optimization
- Automation
- Disaster Recovery
- Monitoring
- Compliance

These principles apply regardless of the cloud provider.

---

# Shared Responsibility Model

Cloud security is a shared responsibility.

```text
Cloud Provider

↓

Physical Infrastructure

Networking

Storage Hardware

↓

Customer

↓

Applications

Operating Systems

IAM

Data

Configurations
```

Customers remain responsible for securing their workloads.

---

# Cloud Service Models

| Model | Customer Manages | Provider Manages |
|--------|------------------|------------------|
| IaaS | OS, Runtime, Apps | Infrastructure |
| PaaS | Applications | Platform + Infrastructure |
| SaaS | Data & Users | Entire Application Stack |

---

# Enterprise Infrastructure Layers

```text
Users

↓

DNS

↓

CDN

↓

Load Balancer

↓

Application Layer

↓

Microservices

↓

Database

↓

Storage

↓

Monitoring
```

Each layer has a specific responsibility.

---

# Availability Zones

Enterprise applications are deployed across multiple Availability Zones.

```text
Region

├── AZ-1

├── AZ-2

└── AZ-3
```

If one Availability Zone fails, traffic is redirected to healthy zones.

---

# Regions

Regions provide geographical redundancy.

Example:

```text
India

↓

Primary Region

↓

Disaster Recovery Region
```

Cross-region architectures improve resilience against regional outages.

---

# High Availability

High Availability (HA) ensures applications remain accessible despite failures.

Typical HA components include:

- Multiple Application Servers
- Load Balancer
- Multi-AZ Databases
- Redundant Networking

HA minimizes downtime.

---

# Scalability

Applications should scale based on demand.

```text
Low Traffic

↓

2 Servers

────────────

High Traffic

↓

10 Servers
```

Cloud platforms enable automatic scaling.

---

# Fault Tolerance

Fault tolerance allows applications to continue operating during failures.

```text
Server Failure

↓

Load Balancer

↓

Healthy Server

↓

Application Available
```

The failure of one component should not impact users.

---

# Cloud Design Principles

Enterprise architectures should follow these principles:

- Design for Failure
- Automate Everything
- Decouple Components
- Minimize Single Points of Failure
- Secure by Default
- Monitor Continuously
- Optimize Costs
- Prefer Managed Services

---

# Reference Enterprise Architecture

```text
Users

↓

Route53

↓

Application Load Balancer

↓

Amazon EKS

↓

Microservices

↓

Amazon RDS

↓

Amazon S3

↓

Prometheus

↓

Grafana

↓

ELK
```

This architecture is commonly used for modern cloud-native applications.

---

# Enterprise Best Practices

- Deploy across multiple Availability Zones.
- Avoid single points of failure.
- Automate infrastructure provisioning.
- Monitor every layer.
- Implement least-privilege access.
- Encrypt data in transit and at rest.
- Regularly test disaster recovery.
- Design for horizontal scaling.

---

# Common Mistakes

- Deploying everything in one Availability Zone.
- Relying on a single application server.
- Ignoring disaster recovery planning.
- Hardcoding infrastructure configurations.
- Not monitoring production systems.
- Manual infrastructure provisioning.
- Weak IAM policies.

---

# Interview Questions

## Basic

1. What is cloud architecture?
2. What is High Availability?
3. Difference between a Region and an Availability Zone.
4. What is the Shared Responsibility Model?
5. Explain IaaS, PaaS, and SaaS.

## Intermediate

1. How do you eliminate single points of failure?
2. Explain horizontal vs vertical scaling.
3. What is fault tolerance?
4. Why are managed services preferred?
5. What are the characteristics of enterprise cloud architecture?

## Advanced

1. Design a highly available cloud architecture for a production microservices application serving millions of users.
2. Explain how High Availability, scalability, fault tolerance, automation, observability, and disaster recovery work together in enterprise cloud architecture.
3. Design a secure, highly available, multi-region architecture for a financial application while minimizing downtime and ensuring compliance.

---

# Chapter 2 - Enterprise Design Principles

Enterprise cloud architecture is not just about choosing AWS services.

It is about following proven design principles that ensure applications remain:

- Highly Available
- Scalable
- Secure
- Reliable
- Observable
- Cost Efficient
- Maintainable

These principles are followed by organizations such as Amazon, Netflix, Google, Microsoft, Uber, and Airbnb.

---

# Design for Failure

The first rule of cloud architecture is:

> **Everything will eventually fail.**

Servers fail.

Networks fail.

Databases fail.

Entire Availability Zones may become unavailable.

Instead of trying to prevent failures completely,

architectures should be designed to continue operating when failures occur.

---

# Design for Failure Example

Poor Design

```text
Users

↓

Single EC2

↓

Database
```

If the EC2 instance fails,

the entire application becomes unavailable.

Enterprise Design

```text
Users

↓

Load Balancer

↓

EC2-1

EC2-2

EC2-3

↓

Database
```

Traffic automatically shifts to healthy instances.

---

# Eliminate Single Points of Failure (SPOF)

A Single Point of Failure is any component whose failure causes the entire system to fail.

Examples:

- Single EC2 Instance
- Single Database
- Single NAT Gateway
- Single Kubernetes Master
- Single Load Balancer

Enterprise systems eliminate SPOFs through redundancy.

---

# Horizontal Scaling

Horizontal scaling means adding more servers.

```text
2 Servers

↓

4 Servers

↓

8 Servers
```

Benefits:

- Higher Availability
- Better Fault Tolerance
- Improved Performance
- Easier Maintenance

Horizontal scaling is preferred for cloud-native applications.

---

# Vertical Scaling

Vertical scaling means increasing server resources.

```text
2 CPU

↓

4 CPU

↓

8 CPU
```

Advantages:

- Simple implementation
- No application changes required

Limitations:

- Hardware limits
- Downtime during resizing
- Does not improve fault tolerance

---

# Horizontal vs Vertical Scaling

| Horizontal Scaling | Vertical Scaling |
|--------------------|------------------|
| Add more servers | Increase server size |
| High availability | Limited availability |
| Better fault tolerance | Single server dependency |
| Cloud-native | Traditional systems |

Modern microservices primarily use horizontal scaling.

---

# Loose Coupling

Applications should minimize dependencies between services.

Poor Design

```text
Service A

↓

Service B

↓

Service C

↓

Service D
```

Failure in one service affects the others.

Better Design

```text
Service A

Service B

Service C

↓

Message Queue

↓

Independent Processing
```

Loose coupling improves resilience.

---

# Stateless Design

Enterprise applications should remain stateless whenever possible.

Poor Design

```text
User

↓

Application

↓

Session Stored in Memory
```

If the server fails,

the session is lost.

Better Design

```text
User

↓

Load Balancer

↓

Application

↓

Redis / Database
```

Session data survives server failures.

---

# Idempotency

An operation is **idempotent** if executing it multiple times produces the same result.

Example:

```text
Terraform Apply

↓

Infrastructure Exists

↓

No Changes
```

Benefits:

- Safe retries
- Reliable automation
- Predictable deployments

Infrastructure automation should always be idempotent.

---

# Immutable Infrastructure

Instead of modifying servers,

replace them.

Traditional

```text
Server

↓

Manual Changes

↓

Configuration Drift
```

Immutable

```text
New Image

↓

New Server

↓

Replace Old Server
```

Immutable infrastructure reduces configuration drift.

---

# Infrastructure as Code (IaC)

Infrastructure should be managed through code.

```text
Terraform

↓

Version Control

↓

Review

↓

Deploy
```

Benefits:

- Repeatability
- Automation
- Version History
- Easy Rollback

---

# Automation First

Manual processes do not scale.

Manual

```text
Engineer

↓

Server

↓

Configuration
```

Automated

```text
Terraform

↓

Ansible

↓

Jenkins

↓

Infrastructure
```

Automation improves consistency and speed.

---

# Self-Healing Systems

Cloud platforms should recover automatically from failures.

```text
Server Fails

↓

Health Check

↓

Auto Scaling

↓

New Server
```

Users experience minimal disruption.

---

# Decoupled Architecture

Applications should communicate asynchronously whenever appropriate.

```text
Order Service

↓

Queue

↓

Payment Service

↓

Notification Service
```

Queues isolate failures and smooth traffic spikes.

---

# Observability

Modern systems require comprehensive observability.

```text
Metrics

↓

Logs

↓

Tracing

↓

Dashboards

↓

Alerts
```

Without observability,

production troubleshooting becomes difficult.

---

# Security by Design

Security should be incorporated from the beginning.

Areas include:

- IAM
- Encryption
- Secrets Management
- Network Security
- Logging
- Auditing

Security is an architectural requirement, not an afterthought.

---

# Cost Optimization

Architecture should balance performance and cost.

Examples:

- Auto Scaling
- Reserved Instances
- Spot Instances
- Storage Lifecycle Policies
- Right-Sized Resources

Efficient architectures minimize unnecessary expenses.

---

# Enterprise Example

Microservices architecture.

```text
Users

↓

Load Balancer

↓

Amazon EKS

↓

Payment

Order

Inventory

Notification

↓

Amazon RDS

↓

Amazon S3
```

Each service scales independently.

---

# Enterprise Best Practices

- Design for failure from the beginning.
- Eliminate single points of failure.
- Prefer horizontal scaling.
- Build stateless services.
- Use immutable infrastructure.
- Automate infrastructure with IaC.
- Implement self-healing mechanisms.
- Continuously monitor applications.
- Design secure architectures by default.
- Optimize cloud costs continuously.

---

# Common Mistakes

- Relying on a single server.
- Storing sessions locally.
- Making manual production changes.
- Tight coupling between services.
- Ignoring observability.
- Scaling vertically by default.
- Delaying security implementation.
- Not automating infrastructure.

---

# Interview Questions

## Basic

1. What is a Single Point of Failure (SPOF)?
2. Explain horizontal scaling.
3. Explain vertical scaling.
4. What is stateless architecture?
5. What is Infrastructure as Code?

## Intermediate

1. Explain immutable infrastructure.
2. Why is loose coupling important?
3. What is idempotency?
4. What is self-healing infrastructure?
5. Why should infrastructure be automated?

## Advanced

1. Design a cloud-native architecture for a production e-commerce platform using the principles of high availability, horizontal scaling, immutable infrastructure, and observability.
2. Explain how stateless design, Infrastructure as Code, automation, idempotency, and self-healing systems work together to build resilient cloud platforms.
3. A financial organization is migrating from traditional virtual machines to Kubernetes-based microservices. Explain how enterprise design principles should guide the migration while improving scalability, resilience, security, and operational efficiency.

---

# Chapter 3 - Enterprise Networking Architecture & Design Patterns

Networking is the foundation of every cloud architecture.

Every request made by a user travels through multiple networking layers before reaching an application.

A well-designed enterprise network provides:

- High Availability
- Security
- Scalability
- Isolation
- Low Latency
- Fault Tolerance

Without a proper networking architecture, even the best application cannot operate reliably.

---

# Enterprise Network Architecture

A typical cloud network follows this structure.

```text
Users

↓

DNS

↓

CDN

↓

Web Application Firewall

↓

Load Balancer

↓

Application Tier

↓

Database Tier

↓

Storage
```

Each layer performs a specific role.

---

# Virtual Private Cloud (VPC)

A **Virtual Private Cloud (VPC)** is a logically isolated network within a cloud provider.

It allows organizations to define:

- IP Address Range
- Subnets
- Routing
- Security Rules
- Internet Connectivity

Every production application begins with a properly designed VPC.

---

# VPC Architecture

```text
AWS Region

↓

VPC

↓

Subnets

↓

Resources
```

A VPC provides complete network isolation from other customers.

---

# CIDR Block Planning

Every VPC is assigned a CIDR block.

Example:

```text
10.0.0.0/16
```

Subnets are created within the VPC.

Example:

```text
10.0.1.0/24

10.0.2.0/24

10.0.3.0/24
```

Proper CIDR planning prevents IP exhaustion and simplifies future expansion.

---

# Public and Private Subnets

Enterprise architectures separate workloads into public and private subnets.

Public Subnet

```text
Internet

↓

Internet Gateway

↓

Public Subnet

↓

Load Balancer

Bastion Host
```

Private Subnet

```text
Private Subnet

↓

Application Servers

↓

Databases

↓

Internal Services
```

Only internet-facing resources should reside in public subnets.

---

# Multi-AZ Architecture

High availability is achieved by distributing resources across multiple Availability Zones.

```text
VPC

├── Public AZ-1

├── Private AZ-1

├── Public AZ-2

└── Private AZ-2
```

Applications continue operating even if one Availability Zone becomes unavailable.

---

# Internet Gateway (IGW)

An Internet Gateway enables communication between the VPC and the internet.

```text
Internet

↓

Internet Gateway

↓

Public Subnet
```

Without an Internet Gateway, public resources cannot communicate with the internet.

---

# NAT Gateway

Private resources often require outbound internet access.

Example:

```text
Private EC2

↓

NAT Gateway

↓

Internet
```

Common use cases:

- Package Updates
- Docker Image Downloads
- API Calls
- Software Installation

Private resources remain inaccessible from the internet.

---

# Route Tables

Route tables determine how network traffic is forwarded.

Example:

```text
Destination

↓

Next Hop

↓

Internet Gateway

↓

NAT Gateway

↓

Local Network
```

Every subnet is associated with a route table.

---

# Security Groups

Security Groups act as virtual firewalls.

Characteristics:

- Stateful
- Instance Level
- Allow Rules Only

Example:

```text
Allow

HTTPS 443

SSH 22

Application Port
```

Traffic not explicitly allowed is denied.

---

# Network ACLs

Network ACLs provide subnet-level security.

Characteristics:

- Stateless
- Allow Rules
- Deny Rules

Comparison:

| Security Group | Network ACL |
|---------------|-------------|
| Stateful | Stateless |
| Instance Level | Subnet Level |
| Allow Only | Allow & Deny |

Most production environments use both.

---

# DNS Architecture

Enterprise DNS workflow:

```text
User

↓

DNS

↓

Load Balancer

↓

Application
```

DNS provides location transparency and simplifies infrastructure changes.

---

# Load Balancer

A Load Balancer distributes incoming requests.

```text
Users

↓

Application Load Balancer

↓

Server 1

Server 2

Server 3
```

Benefits:

- High Availability
- Fault Tolerance
- Automatic Health Checks

---

# Reverse Proxy Pattern

Enterprise applications commonly use reverse proxies.

```text
Client

↓

Load Balancer

↓

Reverse Proxy

↓

Applications
```

Advantages:

- SSL Termination
- Routing
- Authentication
- Caching

---

# Bastion Host Pattern

Administrative access should never be exposed directly.

```text
Administrator

↓

SSH

↓

Bastion Host

↓

Private Servers
```

Only the bastion host is internet accessible.

---

# Three-Tier Network Architecture

A common enterprise design.

```text
Presentation Tier

↓

Application Tier

↓

Database Tier
```

Example:

```text
Internet

↓

Load Balancer

↓

Amazon EKS

↓

Amazon RDS
```

Each tier is isolated from the others.

---

# Zero Trust Networking

Modern architectures follow Zero Trust principles.

Key concepts:

- Verify Every Request
- Least Privilege
- Strong Authentication
- Network Segmentation
- Continuous Monitoring

Never trust traffic based solely on network location.

---

# Kubernetes Networking

Enterprise Kubernetes networking:

```text
Internet

↓

Application Load Balancer

↓

Ingress Controller

↓

Services

↓

Pods
```

Traffic flows through multiple networking layers before reaching containers.

---

# Hybrid Cloud Pattern

Some organizations combine on-premises infrastructure with the cloud.

```text
On-Premises

↓

VPN / Direct Connect

↓

AWS Cloud

↓

Applications
```

Hybrid networking enables gradual cloud migration.

---

# Enterprise Example

Production microservices network.

```text
Users

↓

Route53

↓

CloudFront

↓

AWS WAF

↓

Application Load Balancer

↓

Amazon EKS

↓

Microservices

↓

Amazon RDS

↓

Amazon S3
```

Every layer contributes to security and scalability.

---

# Enterprise Best Practices

- Design VPCs with future growth in mind.
- Separate public and private workloads.
- Deploy across multiple Availability Zones.
- Use NAT Gateways for private subnet internet access.
- Restrict inbound traffic using Security Groups.
- Use Network ACLs for additional subnet protection.
- Minimize administrative access.
- Implement Zero Trust networking.
- Document network architecture thoroughly.
- Continuously monitor network traffic.

---

# Common Mistakes

- Placing databases in public subnets.
- Using a single Availability Zone.
- Opening unnecessary ports.
- Poor CIDR planning.
- Allowing unrestricted SSH access.
- Mixing production and development workloads.
- Ignoring network monitoring.
- Creating overly permissive Security Groups.

---

# Interview Questions

## Basic

1. What is a VPC?
2. What is a subnet?
3. Difference between public and private subnets.
4. What is an Internet Gateway?
5. What is a NAT Gateway?

## Intermediate

1. Difference between Security Groups and Network ACLs.
2. Explain Multi-AZ architecture.
3. Why are route tables required?
4. Explain the Bastion Host pattern.
5. What is Zero Trust networking?

## Advanced

1. Design a secure networking architecture for a production Kubernetes platform hosting microservices on AWS, including VPC, subnets, routing, Security Groups, NAT Gateways, and Load Balancers.
2. Explain how networking components such as VPCs, CIDR blocks, public/private subnets, Internet Gateways, NAT Gateways, Security Groups, and Network ACLs work together to provide secure and scalable cloud infrastructure.
3. A financial organization is migrating to AWS and must securely host internet-facing applications, internal APIs, databases, and Kubernetes clusters. Design the complete networking architecture while ensuring high availability, least privilege, fault tolerance, and future scalability.

---

# Chapter 4 - Enterprise Compute Architecture & Application Design Patterns

Compute is the execution layer of every cloud application.

Whether applications run on:

- Virtual Machines
- Containers
- Kubernetes
- Serverless Platforms

they all depend on compute resources.

A well-designed compute architecture provides:

- High Availability
- Auto Scaling
- Fault Tolerance
- Performance
- Cost Optimization
- Security

Enterprise applications are designed so compute resources can scale automatically while remaining highly available.

---

# Enterprise Compute Architecture

A modern compute architecture follows this flow.

```text
Users

↓

DNS

↓

Load Balancer

↓

Compute Layer

↓

Application

↓

Database

↓

Storage
```

The compute layer processes application requests and communicates with backend services.

---

# Compute Options

Cloud providers offer multiple compute models.

| Compute Type | Example |
|--------------|---------|
| Virtual Machines | Amazon EC2 |
| Containers | Docker |
| Container Orchestration | Amazon EKS |
| Serverless | AWS Lambda |
| Platform Services | App Runner, ECS Fargate |

Each option is suitable for different workloads.

---

# Virtual Machine Pattern

Traditional applications often run on virtual machines.

```text
Load Balancer

↓

EC2-1

EC2-2

EC2-3

↓

Database
```

Advantages:

- Full OS control
- Legacy application support
- Flexible configurations

Disadvantages:

- Manual patching
- OS management
- Higher operational overhead

---

# Container Pattern

Containers package applications with their dependencies.

```text
Docker Image

↓

Container

↓

Application
```

Benefits:

- Consistency
- Faster deployments
- Lightweight execution
- Easy portability

Containers are widely used for microservices.

---

# Kubernetes Pattern

Enterprise Kubernetes architecture.

```text
Users

↓

Load Balancer

↓

Ingress

↓

Services

↓

Pods

↓

Worker Nodes
```

Kubernetes provides:

- Scheduling
- Self-Healing
- Auto Scaling
- Rolling Updates

---

# Stateless Compute Pattern

Modern applications should remain stateless.

```text
Application

↓

Redis

↓

Database
```

Application instances can be replaced without losing user sessions.

Benefits:

- Easy scaling
- Simplified deployments
- Improved resilience

---

# Stateful Workloads

Some workloads require persistent data.

Examples:

- Databases
- Message Brokers
- Elasticsearch
- Kafka

Stateful workloads require persistent storage and careful placement.

---

# Auto Scaling Pattern

Applications should automatically scale according to demand.

```text
Low Traffic

↓

2 Instances

────────────

High Traffic

↓

10 Instances
```

Scaling policies are commonly based on:

- CPU Utilization
- Memory Usage
- Request Count
- Queue Length

---

# Self-Healing Pattern

Cloud platforms automatically replace failed instances.

```text
Application Failure

↓

Health Check

↓

Replace Instance

↓

Application Restored
```

Users experience minimal disruption.

---

# Blue-Green Deployment

Two identical environments exist.

```text
Blue

(Current)

↓

Green

(New Version)

↓

Switch Traffic
```

Benefits:

- Zero Downtime
- Easy Rollback
- Safer Releases

---

# Canary Deployment

Traffic is shifted gradually.

```text
Version 1

95%

──────────

Version 2

5%

↓

Monitor

↓

Increase Traffic
```

Benefits:

- Lower deployment risk
- Early issue detection
- Controlled rollout

---

# Rolling Deployment

Instances are updated gradually.

```text
Server 1

↓

Updated

↓

Server 2

↓

Updated

↓

Server 3

↓

Updated
```

Applications remain available during upgrades.

---

# Immutable Deployment

Instead of updating servers,

replace them.

```text
Old Server

↓

New Image

↓

New Server

↓

Terminate Old Server
```

Benefits:

- Consistency
- Easy rollback
- Reduced configuration drift

---

# Microservices Pattern

Enterprise applications are divided into independent services.

```text
Gateway

↓

User

Order

Payment

Inventory

Notification
```

Each service:

- Deploys independently
- Scales independently
- Fails independently

---

# Sidecar Pattern

A helper container runs alongside the main application.

```text
Pod

├── Application

└── Sidecar
```

Common sidecars:

- Log Collectors
- Service Mesh Proxies
- Monitoring Agents

---

# Ambassador Pattern

A proxy container manages outbound communication.

```text
Application

↓

Ambassador

↓

External Service
```

Benefits:

- Centralized communication
- Easier configuration
- Improved security

---

# Adapter Pattern

An adapter converts application output into another format.

```text
Application

↓

Adapter

↓

Monitoring System
```

Useful when integrating legacy applications.

---

# Serverless Pattern

Applications execute only when events occur.

```text
Request

↓

Lambda

↓

Response
```

Advantages:

- No server management
- Automatic scaling
- Pay-per-use pricing

Best suited for event-driven workloads.

---

# Enterprise Compute Architecture

```text
Users

↓

Route53

↓

Application Load Balancer

↓

Amazon EKS

↓

Microservices

↓

Amazon RDS

↓

Amazon ElastiCache

↓

Amazon S3
```

Every compute component scales independently.

---

# Enterprise Best Practices

- Design stateless applications whenever possible.
- Use containers for microservices.
- Enable Auto Scaling.
- Use health checks for self-healing.
- Prefer immutable deployments.
- Choose the appropriate deployment strategy.
- Separate compute from storage.
- Monitor application performance continuously.
- Automate deployments through CI/CD.
- Right-size compute resources for cost optimization.

---

# Common Mistakes

- Running stateful workloads without persistent storage.
- Deploying applications without health checks.
- Manual server configuration.
- Ignoring Auto Scaling.
- Using a single application instance.
- Performing in-place production updates.
- Overprovisioning compute resources.
- Tight coupling between application components.

---

# Interview Questions

## Basic

1. What is cloud compute?
2. Difference between virtual machines and containers.
3. What is Kubernetes?
4. What is Auto Scaling?
5. What is a stateless application?

## Intermediate

1. Explain Blue-Green deployment.
2. Explain Canary deployment.
3. Difference between Rolling and Blue-Green deployments.
4. What is immutable infrastructure?
5. Explain the Sidecar pattern.

## Advanced

1. Design a production-ready compute architecture for a Kubernetes-based microservices application using Auto Scaling, self-healing, immutable deployments, and rolling updates.
2. Explain how compute design patterns such as stateless architecture, Blue-Green deployments, Canary releases, Auto Scaling, Sidecars, and Kubernetes contribute to highly available enterprise platforms.
3. A global e-commerce company expects traffic spikes during seasonal sales. Design a compute architecture that automatically scales, minimizes deployment risk, supports zero-downtime releases, and maintains high availability across multiple Availability Zones.

---

