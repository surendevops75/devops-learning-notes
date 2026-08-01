# Amazon Elastic Container Service (Amazon ECS)

---

# Introduction

Amazon Elastic Container Service (Amazon ECS) is a fully managed container orchestration service that allows you to deploy, manage, scale, and monitor Docker containers without managing Kubernetes control planes.

ECS simplifies container deployment by integrating deeply with AWS services such as:

- Amazon ECR
- Application Load Balancer
- Auto Scaling
- IAM
- CloudWatch
- AWS Secrets Manager
- AWS Systems Manager

Organizations use ECS to run production microservices, APIs, background workers, scheduled jobs, and event-driven applications.

---

# What is Amazon ECS?

Amazon ECS is AWS's native container orchestration service.

It manages:

- Container Scheduling
- Cluster Management
- Service Discovery
- Auto Scaling
- Load Balancing
- Rolling Deployments
- Health Monitoring

Instead of manually starting Docker containers on EC2 instances, ECS automates container lifecycle management.

---

# Why Amazon ECS?

Suppose an organization has 80 Dockerized microservices.

Managing them manually requires:

- Choosing servers
- Starting containers
- Restarting failed containers
- Scaling containers
- Updating versions
- Monitoring health

Amazon ECS automates these operational tasks.

---

# Real World Problem

An online shopping platform contains:

- Frontend
- Product Service
- Cart Service
- Payment Service
- User Service
- Notification Service

Requirements

- High Availability
- Automatic Scaling
- Zero Downtime Deployments
- Health Monitoring
- Secure Secrets Management

Amazon ECS provides these capabilities.

---

# Enterprise Architecture

```text
Developer

↓

GitHub

↓

GitHub Actions / Jenkins

↓

Docker Build

↓

Amazon ECR

↓

Amazon ECS Cluster

↓

Amazon ECS Service

↓

Application Load Balancer

↓

Internet Users
```

---

# ECS Core Components

Amazon ECS consists of:

- Cluster
- Task Definition
- Task
- Service
- Container
- Capacity Provider
- Launch Type
- Scheduler
- Service Discovery

---

# ECS Cluster

A Cluster is a logical grouping of compute resources where containers run.

A cluster can contain:

- EC2 Instances
- AWS Fargate
- External Servers (ECS Anywhere)

Example

```text
Production Cluster

├── Frontend Service

├── Backend Service

├── Payment Service

└── Notification Service
```

---

# Task Definition

A Task Definition is the blueprint of an application.

It defines:

- Docker Image
- CPU
- Memory
- Environment Variables
- Secrets
- Port Mapping
- Logging
- IAM Role

Every application deployment starts from a Task Definition.

---

# Task

A Task is a running instance of a Task Definition.

Example

```text
Task Definition

↓

Task

↓

Running Container
```

---

# Service

An ECS Service ensures the required number of Tasks are always running.

Example

Desired Tasks

```
3
```

If one container fails:

```text
Task Fails

↓

Scheduler Detects Failure

↓

Launch New Task
```

---

# Container

A Container is the running application.

Example

```text
Task

↓

Nginx Container

↓

Running Application
```

One Task may contain multiple containers.

---

# Launch Types

Amazon ECS supports two launch types.

- EC2
- AWS Fargate

---

# EC2 Launch Type

Containers run on EC2 instances managed by you.

Advantages

- Lower cost
- Full OS control
- GPU Support
- Custom AMIs

Disadvantages

- Manage EC2
- Patch OS
- Capacity Planning

---

# AWS Fargate

AWS manages the infrastructure.

You only provide:

- Docker Image
- CPU
- Memory

Advantages

- No EC2 Management
- Automatic Infrastructure
- Serverless Containers
- Simpler Operations

Recommended for most new workloads.

---

# EC2 vs Fargate

| EC2 | Fargate |
|------|----------|
| Manage EC2 | Serverless |
| Lower Cost | Simpler Operations |
| Full Control | Fully Managed |
| Patch OS | AWS Handles OS |
| Best for Large Clusters | Best for Microservices |

---

# Capacity Providers

Capacity Providers manage infrastructure capacity automatically.

Supports

- EC2
- Fargate
- Fargate Spot

Benefits

- Better utilization
- Flexible scaling
- Cost optimization

---

# ECS Scheduler

The ECS Scheduler decides where containers should run.

It considers:

- CPU
- Memory
- Availability Zone
- Placement Constraints
- Resource Availability

---

# Task Placement Strategies

Supported strategies include:

- Binpack
- Spread
- Random

Binpack

Uses minimum EC2 instances.

Spread

Distributes tasks evenly.

Random

Random placement.

---

# Service Discovery

Amazon ECS integrates with AWS Cloud Map.

Example

```text
payment.internal

↓

Payment Service
```

Applications communicate using DNS instead of IP addresses.

---

# Networking Modes

Supported networking modes

- bridge
- host
- awsvpc
- none

Production Recommendation

Use

```
awsvpc
```

Each task receives its own Elastic Network Interface (ENI).

---

# IAM Roles

Two IAM roles are commonly used.

Execution Role

Used for:

- Pull Images
- Send Logs
- Access Secrets

Task Role

Used by the application.

Example

```text
Container

↓

Task Role

↓

Amazon S3
```

---

# Secrets Management

Sensitive information should never be stored inside Docker images.

Use:

- AWS Secrets Manager
- Systems Manager Parameter Store

Example

```text
Container

↓

Secrets Manager

↓

Database Password
```

---

# Logging

Containers send logs to CloudWatch.

Architecture

```text
Container

↓

CloudWatch Logs

↓

Logs Insights

↓

Dashboard
```

---

# Health Checks

Amazon ECS supports

- Container Health Checks
- Load Balancer Health Checks

If unhealthy:

```text
Container

↓

Stopped

↓

Scheduler

↓

New Task
```

---

# Auto Scaling

ECS Services support automatic scaling.

Metrics

- CPU
- Memory
- Request Count

Workflow

```text
CPU > 70%

↓

CloudWatch

↓

ECS Auto Scaling

↓

Launch New Task
```

---

# Rolling Deployments

Deployment Flow

```text
Version 1

↓

Launch Version 2

↓

Health Check

↓

Shift Traffic

↓

Terminate Version 1
```

Zero downtime deployment.

---

# Blue/Green Deployment

Amazon ECS integrates with CodeDeploy.

Architecture

```text
Blue Version

↓

Green Version

↓

Traffic Shift

↓

Production
```

Useful for safer deployments.

---

# Integration with ALB

```text
Users

↓

ALB

↓

Target Group

↓

ECS Service

↓

Containers
```

ALB distributes traffic across healthy containers.

---

# Integration with ECR

```text
GitHub

↓

Docker Build

↓

Amazon ECR

↓

Amazon ECS

↓

Running Containers
```

---

# AWS CLI

Create Cluster

```bash
aws ecs create-cluster \
--cluster-name production
```

List Clusters

```bash
aws ecs list-clusters
```

List Services

```bash
aws ecs list-services \
--cluster production
```

---

# Terraform

```hcl
resource "aws_ecs_cluster" "production" {

  name = "production"

}
```

---

# Best Practices

- Use Fargate for serverless workloads
- Store images in Amazon ECR
- Enable CloudWatch Logs
- Use Task IAM Roles
- Store secrets in Secrets Manager
- Enable Auto Scaling
- Use ALB for production
- Deploy across multiple Availability Zones
- Use immutable image tags
- Configure health checks

---

# Common Mistakes

- Hardcoding secrets
- Running production containers without health checks
- Using latest image tag
- Not configuring IAM roles
- Ignoring CloudWatch logs
- Single AZ deployment
- Running everything in one Task Definition

---

# Troubleshooting

## Tasks Not Starting

Check

- Task Definition
- CPU
- Memory
- IAM Role
- Image URI

---

## Cannot Pull Image

Verify

- ECR Permissions
- Image Exists
- Task Execution Role

---

## Tasks Keep Restarting

Check

- Application Logs
- Health Checks
- Memory
- Exit Code

---

## ALB Shows Unhealthy

Verify

- Target Group
- Container Port
- Health Endpoint
- Security Group

---

# Interview Questions

1. What is Amazon ECS?
2. ECS vs Kubernetes?
3. ECS vs EKS?
4. ECS Cluster?
5. Task Definition?
6. Task vs Service?
7. EC2 vs Fargate?
8. What are Capacity Providers?
9. Explain Task Role.
10. Explain Execution Role.
11. Explain awsvpc mode.
12. ECS deployment strategies?
13. How does ECS integrate with ALB?
14. How does ECS scale?
15. Explain Blue/Green deployment.

---

# Scenario Questions

### Scenario 1

Containers continuously restart after deployment.

How would you troubleshoot?

---

### Scenario 2

ECS cannot pull images from ECR.

What would you check?

---

### Scenario 3

Traffic increases 10x during a sale.

How would ECS handle it?

---

### Scenario 4

Developers accidentally hardcoded database passwords inside Docker images.

How would you redesign the solution?

---

### Scenario 5

An ALB reports every ECS task as unhealthy.

Which components would you investigate?

---

# Summary

Amazon ECS is AWS's native container orchestration platform that simplifies the deployment, management, and scaling of Docker containers. By integrating with Amazon ECR, ALB, CloudWatch, IAM, Secrets Manager, and Auto Scaling, ECS enables organizations to run secure, scalable, and highly available containerized applications with minimal operational overhead.

For production environments, use Fargate or well-managed EC2 capacity providers, enable CloudWatch logging, configure health checks, use Task IAM Roles, store secrets securely, and automate deployments with rolling or blue/green strategies.