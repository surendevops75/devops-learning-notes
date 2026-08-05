# AWS Fargate (Deep Dive)

# Chapter 1 - AWS Fargate Fundamentals

## What is AWS Fargate?

AWS Fargate is a serverless compute engine for containers that allows you to run containers without provisioning, managing, or scaling EC2 instances.

Instead of worrying about infrastructure, you define:

- Container Image
- CPU
- Memory
- Networking
- IAM Permissions

AWS automatically provisions the required compute resources and runs the container.

With Fargate, you manage **containers**, while AWS manages the **servers**.

---

# Why Was AWS Fargate Introduced?

Before Fargate, running containers on AWS required managing EC2 instances.

Example

```text
Application

↓

Docker Container

↓

Amazon ECS

↓

EC2 Instance

↓

VPC

↓

AWS Infrastructure
```

The operations team had to:

- Launch EC2 instances
- Patch operating systems
- Upgrade AMIs
- Replace unhealthy instances
- Scale Auto Scaling Groups
- Monitor EC2 health
- Plan capacity

This increased operational complexity.

AWS introduced Fargate so engineers could focus on applications instead of infrastructure.

---

# Traditional Container Deployment

```text
Application

↓

Docker

↓

Amazon ECS

↓

EC2

↓

Auto Scaling Group

↓

Launch Template

↓

VPC
```

Responsibilities

Customer manages

- EC2
- AMI
- OS Updates
- Capacity
- Scaling
- Security Patches

---

# Fargate Deployment

```text
Application

↓

Docker Image

↓

Amazon ECS

↓

AWS Fargate

↓

AWS Managed Infrastructure
```

Customer manages only

- Application
- Container
- CPU
- Memory
- IAM
- Networking

AWS manages everything else.

---

# Supported Container Platforms

AWS Fargate supports

- Amazon ECS
- Amazon EKS

Architecture

```text
Application

↓

Docker Image

↓

ECS

↓

Fargate
```

or

```text
Application

↓

Docker Image

↓

Kubernetes (EKS)

↓

Fargate
```

---

# Core Characteristics

AWS Fargate is

- Serverless
- Fully Managed
- Container-native
- Elastic
- Secure
- Highly Available

It is **not**

- A container runtime
- A Kubernetes replacement
- A Docker replacement

It is a compute engine.

---

# Why Enterprises Use Fargate

Organizations choose Fargate because it removes infrastructure management.

Benefits include

- Faster deployments
- Reduced operational overhead
- Automatic infrastructure management
- Better security isolation
- Pay only for allocated CPU and memory
- Easier scaling

---

# Typical Enterprise Use Cases

Fargate is commonly used for

- REST APIs
- Microservices
- Background workers
- Event processing
- Scheduled jobs
- CI/CD runners
- Internal applications
- Batch processing

Less suitable for

- GPU workloads
- Long-running high-performance computing
- Applications requiring privileged containers
- Specialized kernel modules

---

# AWS Fargate Architecture

```text
                Amazon ECS / Amazon EKS

                        │

                  Scheduler

                        │

                  AWS Fargate

                        │

             Firecracker MicroVM

                        │

                  Docker Container

                        │

                  Application
```

The application never interacts with the underlying EC2 instance.

---

# Shared Responsibility Model

Customer manages

- Application code
- Container image
- CPU and memory selection
- IAM Roles
- Security Groups
- Secrets
- Logging configuration

AWS manages

- EC2 instances
- Host operating system
- Hypervisor
- Capacity provisioning
- Infrastructure patching
- Hardware failures

---

# How AWS Fargate Works

When a task starts

```text
User

↓

Run Task

↓

ECS Scheduler

↓

Fargate

↓

Allocate Compute

↓

Create MicroVM

↓

Pull Container Image

↓

Attach Network

↓

Start Container

↓

Application Running
```

The entire infrastructure lifecycle is automated.

---

# Fargate vs EC2 Launch Type

| Feature | Fargate | ECS on EC2 |
|----------|----------|------------|
| Manage EC2 | ❌ | ✅ |
| Patch OS | ❌ | ✅ |
| Capacity Planning | ❌ | ✅ |
| Pay Per Task | ✅ | ❌ |
| Server Access | ❌ | ✅ |
| Infrastructure Managed by AWS | ✅ | ❌ |
| Operational Overhead | Very Low | Medium to High |

---

# ECS Fargate vs EKS Fargate

| ECS Fargate | EKS Fargate |
|--------------|-------------|
| Uses Amazon ECS | Uses Kubernetes |
| Simpler | Kubernetes-native |
| Less operational complexity | More Kubernetes flexibility |
| AWS-managed scheduler | Kubernetes scheduler |
| Easier learning curve | Requires Kubernetes knowledge |

---

# Fargate vs AWS Lambda

Many engineers confuse these services.

| AWS Lambda | AWS Fargate |
|-------------|-------------|
| Functions | Containers |
| Event-driven | Long-running services supported |
| Short execution duration | Long-running applications |
| Limited runtime | Any runtime inside container |
| Stateless | Stateless or stateful with external storage |

Use Lambda for event-driven functions.

Use Fargate for containerized applications.

---

# Fargate vs Kubernetes Worker Nodes

Traditional Kubernetes

```text
Pods

↓

Worker Nodes

↓

EC2 Instances
```

Fargate

```text
Pods

↓

Fargate

↓

AWS Managed Infrastructure
```

No worker nodes to maintain.

---

# Benefits

- No server management
- Automatic infrastructure provisioning
- Better workload isolation
- Easy scaling
- Faster deployments
- Integrated with ECS and EKS
- Highly available
- Secure by design

---

# Limitations

- Higher cost than EC2 for steady, predictable workloads
- Limited OS-level customization
- No SSH access
- No privileged containers
- Limited support for specialized workloads
- Less control over underlying infrastructure

---

# When Should You Choose Fargate?

Choose Fargate when

- You don't want to manage servers.
- Applications are containerized.
- Workloads scale dynamically.
- Teams are small.
- Operational simplicity is a priority.
- Infrastructure management should be minimized.

Avoid Fargate when

- You need GPU support.
- You require privileged containers.
- You need kernel customization.
- Workloads run continuously at high utilization where EC2 may be more cost-effective.

---

# Enterprise Example

An e-commerce company operates

- 150 microservices
- 30 development teams
- Multiple AWS accounts

Instead of maintaining hundreds of EC2 instances,

every service runs on AWS Fargate.

Architecture

```text
Amazon ECR

↓

Amazon ECS

↓

AWS Fargate

↓

Application Containers

↓

Application Load Balancer

↓

Users
```

Operations teams no longer manage

- EC2
- Auto Scaling Groups
- AMIs
- Operating system patching

They focus entirely on applications.

---

# Best Practices

- Build small container images.
- Store images in Amazon ECR.
- Use IAM Task Roles.
- Store secrets in AWS Secrets Manager.
- Send logs to CloudWatch.
- Enable Container Insights.
- Keep containers stateless.
- Use Auto Scaling.
- Use Fargate Spot where appropriate.

---

# Common Mistakes

- Treating Fargate like a VM.
- Trying to SSH into containers.
- Running large monolithic applications.
- Hardcoding secrets.
- Ignoring resource limits.
- Using root users inside containers.
- Oversizing CPU and memory.

---

# Interview Questions

## Basic

- What is AWS Fargate?
- Why was Fargate introduced?
- Is Fargate serverless?
- Which services support Fargate?

## Intermediate

- ECS Fargate vs ECS EC2.
- ECS Fargate vs EKS Fargate.
- Fargate vs Lambda.
- Explain the Fargate architecture.

## Advanced

- Design a serverless container platform for 500 microservices.
- When would you choose EC2 instead of Fargate?
- Explain how AWS manages infrastructure in Fargate.

---

# Chapter 2 - AWS Fargate Architecture & Internal Working

## How Does AWS Fargate Actually Work?

One of the biggest misconceptions is that Fargate is a container runtime.

It is **not**.

Docker (or another OCI-compliant runtime managed by AWS) runs the container, while Fargate provides the underlying compute infrastructure.

Internally, AWS provisions isolated compute resources for every task or pod.

High-level workflow

```text
Developer

↓

Build Docker Image

↓

Amazon ECR

↓

Amazon ECS / Amazon EKS

↓

Scheduler

↓

AWS Fargate

↓

Firecracker MicroVM

↓

Container Runtime

↓

Application
```

The developer never interacts with the underlying infrastructure.

---

# Fargate Internal Architecture

When a task starts, multiple AWS services work together.

```text
                ECS Control Plane

                       │

                 Task Scheduler

                       │

               AWS Fargate Service

                       │

         Compute Provisioning Engine

                       │

             Firecracker MicroVM

                       │

             Container Runtime

                       │

                 Docker Image

                       │

                 Application
```

Each layer has a specific responsibility.

---

# Fargate Task Lifecycle

Every task follows a lifecycle.

```text
Task Definition

↓

Task Requested

↓

Scheduler Decision

↓

Compute Provisioned

↓

MicroVM Created

↓

Network Attached

↓

IAM Credentials Attached

↓

Container Image Pulled

↓

Container Started

↓

Health Check

↓

Application Running

↓

Task Stops

↓

Resources Released
```

AWS automatically removes compute resources after the task ends.

No infrastructure cleanup is required.

---

# What Happens When You Run a Task?

Suppose you execute

```text
Run Task
```

Internally AWS performs

### Step 1

Validate Task Definition

Checks

- CPU
- Memory
- Container Image
- IAM Roles
- Network Configuration

---

### Step 2

Scheduler Placement

Scheduler determines

- Cluster
- Capacity
- Availability Zone
- Networking

---

### Step 3

Provision Compute

Unlike ECS EC2,

AWS provisions brand-new compute capacity.

No EC2 instance is reused.

---

### Step 4

Create Firecracker MicroVM

A lightweight virtual machine is created.

```text
Physical Server

↓

Nitro Hypervisor

↓

Firecracker MicroVM

↓

Container
```

Every task receives its own isolated compute environment.

---

### Step 5

Create Network

Fargate creates

- Elastic Network Interface (ENI)
- Private IP
- Security Groups

The container receives its own network identity.

---

### Step 6

Download Image

```text
Amazon ECR

↓

Container Image

↓

MicroVM
```

Layers are downloaded.

Image is verified.

Container runtime prepares execution.

---

### Step 7

Start Container

Application begins execution.

```text
MicroVM

↓

Container

↓

Application
```

---

### Step 8

Monitoring Starts

CloudWatch

↓

Logs

↓

Metrics

↓

Container Insights

AWS continuously monitors task health.

---

# Firecracker MicroVM

One of the biggest innovations behind AWS Fargate.

Instead of launching a full virtual machine,

AWS launches a **MicroVM**.

Architecture

```text
Physical Server

↓

Nitro Hypervisor

↓

Firecracker

↓

Container
```

Firecracker was developed by AWS specifically for

- Lambda
- Fargate

Advantages

- Very fast startup
- Strong isolation
- Minimal resource overhead
- High density
- Improved security

---

# Why Not Run Containers Directly?

Traditional containers share the Linux kernel.

```text
Host OS

↓

Docker

↓

Container-A

Container-B
```

If the host kernel is compromised,

multiple containers could be affected.

---

With Firecracker

```text
Host

↓

MicroVM-A

↓

Container-A

Host

↓

MicroVM-B

↓

Container-B
```

Each workload has stronger isolation.

---

# Firecracker vs Traditional Virtual Machine

| Traditional VM | Firecracker |
|----------------|-------------|
| Large OS | Minimal VM |
| Slow startup | Milliseconds startup |
| Heavy memory usage | Lightweight |
| General-purpose | Container workloads |
| Higher overhead | Very low overhead |

---

# Fargate Isolation Model

Each task receives

- Dedicated CPU allocation
- Dedicated memory allocation
- Dedicated ENI
- Dedicated MicroVM

Unlike EC2 launch type,

tasks don't share compute resources in the same way.

This improves tenant isolation.

---

# Resource Allocation

When defining a task

Example

```text
CPU

1 vCPU

Memory

2 GB
```

AWS reserves

```text
1 vCPU

2 GB RAM
```

for that task.

Resources are guaranteed.

---

# Fargate CPU & Memory Combinations

AWS supports predefined combinations.

Examples

| CPU | Memory Options |
|------|----------------|
|0.25 vCPU|0.5 GB–2 GB|
|0.5 vCPU|1 GB–4 GB|
|1 vCPU|2 GB–8 GB|
|2 vCPU|4 GB–16 GB|
|4 vCPU|8 GB–30 GB|
|8 vCPU|16 GB–60 GB|
|16 vCPU|32 GB–120 GB|

CPU and memory cannot be selected independently.

Only supported combinations are allowed.

---

# Why Fixed Combinations?

Benefits

- Predictable scheduling
- Better resource utilization
- Simpler capacity management
- Consistent performance

---

# Fargate Networking Model

Fargate uses

```text
awsvpc
```

network mode.

Each task receives

```text
Elastic Network Interface

↓

Private IP

↓

Security Groups

↓

Route Tables
```

Unlike Docker bridge networking,

every task behaves like an independent EC2 instance.

---

# awsvpc Mode

Example

```text
Task-1

↓

ENI

↓

10.0.1.25

Task-2

↓

ENI

↓

10.0.1.26
```

Each task has

- Own IP
- Own Security Group
- Own network isolation

This simplifies networking.

---

# Advantages of awsvpc Mode

- Native VPC networking
- Security Groups per task
- Easy integration with ALB
- Better isolation
- Predictable networking

---

# ENI Allocation

When a task starts

AWS creates

```text
Task

↓

Elastic Network Interface

↓

Subnet

↓

Private IP

↓

Security Group
```

When the task stops,

the ENI is automatically removed.

---

# Subnet Selection

Production deployments usually specify multiple private subnets.

```text
VPC

├── Private Subnet-A

├── Private Subnet-B

└── Private Subnet-C
```

Benefits

- High availability
- Better scheduling
- Fault tolerance

---

# Security Groups

Every task can have its own Security Group.

Example

```text
Frontend Task

↓

Allow

443

Backend Task

↓

Allow

8080

Database Task

↓

Allow

3306
```

Fine-grained security becomes possible.

---

# Internet Connectivity

Private Subnet

```text
Task

↓

NAT Gateway

↓

Internet
```

Public Subnet

```text
Task

↓

Internet Gateway

↓

Internet
```

Production workloads generally use **private subnets**.

---

# Fargate Platform Versions

AWS periodically releases new platform versions.

They provide

- Security improvements
- Kernel updates
- New features
- Bug fixes
- Performance improvements

Always use the latest supported version unless a specific application requires otherwise.

---

# Enterprise Architecture Example

```text
Application Load Balancer

            │

     Amazon ECS Cluster

            │

      AWS Fargate Tasks

            │

   Private Subnet (Multi-AZ)

            │

      Amazon RDS

            │

     AWS Secrets Manager

            │

     CloudWatch Logs
```

Every task

- Receives its own ENI
- Runs inside its own Firecracker MicroVM
- Uses IAM Task Role
- Sends logs to CloudWatch
- Retrieves secrets securely

This architecture provides high availability, strong isolation, and minimal operational overhead.

---

# Best Practices

- Use private subnets for production.
- Deploy tasks across multiple Availability Zones.
- Allocate CPU and memory based on application profiling.
- Keep task definitions immutable.
- Use awsvpc networking.
- Use dedicated Security Groups for different application tiers.
- Monitor ENI utilization in large environments.

---

# Common Mistakes

- Running production workloads in public subnets.
- Assigning excessive CPU and memory.
- Using one Security Group for every task.
- Ignoring ENI limits.
- Assuming tasks share the same network namespace like Docker bridge networking.
- Hardcoding IP addresses instead of using Service Discovery or Load Balancers.

---

# Interview Questions

## Basic

- What happens internally when a Fargate task starts?
- What is awsvpc mode?
- Does every Fargate task receive its own IP address?

## Intermediate

- Explain the Fargate task lifecycle.
- Why does Fargate use Firecracker?
- How is networking different from Docker bridge mode?

## Advanced

- Explain the complete startup process of a Fargate task.
- Why is Fargate considered more secure than traditional shared-host container deployments?
- Design a highly available Fargate deployment across three Availability Zones.

---

# Chapter 3 - Amazon ECS Task Definitions

## What is a Task Definition?

A Task Definition is the blueprint of an application running on Amazon ECS.

It describes **how a container should run**, including:

- Container Image
- CPU
- Memory
- Network
- Ports
- Environment Variables
- Secrets
- Storage
- Logging
- IAM Roles

Think of it as a Kubernetes Pod Manifest or a Docker Compose file, but specifically designed for Amazon ECS.

A Task Definition itself does **not** run containers.

It is simply a template.

---

# Why Do We Need Task Definitions?

Suppose you have a container image.

```
mycompany/payment:v1.0
```

AWS still doesn't know:

- How much CPU should be allocated?
- How much memory?
- Which port should be exposed?
- Which subnet should it run in?
- Which IAM permissions should it have?
- Which logs should be collected?

A Task Definition answers all of these questions.

---

# ECS Deployment Flow

```text
Developer

↓

Docker Image

↓

Amazon ECR

↓

Task Definition

↓

ECS Service

↓

AWS Fargate

↓

Running Task
```

Without a Task Definition,

ECS cannot launch the container.

---

# Components of a Task Definition

A typical Task Definition contains

```text
Task Definition

├── Family

├── Task Role

├── Execution Role

├── CPU

├── Memory

├── Network Mode

├── Container Definitions

├── Environment Variables

├── Secrets

├── Storage

└── Logging
```

Each component plays an important role.

---

# Task Definition Revisions

Every update creates a new revision.

Example

```text
payment-api

Revision 1

↓

Revision 2

↓

Revision 3

↓

Revision 4
```

Older revisions remain available.

This enables easy rollbacks.

---

# Task Definition Family

The Family groups all revisions.

Example

```text
Family

payment-api

↓

Revision 1

Revision 2

Revision 3
```

Production normally uses the latest stable revision.

---

# Container Definition

Inside every Task Definition is one or more Container Definitions.

Example

```text
Task Definition

├── Application

├── Nginx

└── Fluent Bit
```

A single task may run multiple containers.

---

# Single Container Task

```text
Task

↓

Application Container
```

Most microservices use this architecture.

---

# Multi-Container Task

```text
Task

├── Application

├── Nginx Reverse Proxy

└── Log Collector
```

Containers share the task lifecycle.

If the task stops,

every container stops.

---

# Task CPU

Task CPU defines the compute allocated to the task.

Example

```text
1 vCPU
```

AWS reserves

```
1 Virtual CPU
```

for that task.

---

# Task Memory

Memory defines RAM allocated to the task.

Example

```text
2 GB
```

AWS reserves

```
2 GB RAM
```

The application cannot exceed this allocation.

---

# CPU and Memory Relationship

Not every combination is valid.

Example

Valid

```text
1 vCPU

↓

2 GB
```

Valid

```text
2 vCPU

↓

4 GB
```

Invalid

```text
1 vCPU

↓

64 GB
```

AWS validates supported combinations.

---

# Essential Container

Every container can be marked

```text
Essential

True
```

or

```text
Essential

False
```

If an Essential container fails,

the entire task stops.

Example

```text
Application

Essential=True

↓

Crash

↓

Task Stops
```

---

# Non-Essential Container

Example

```text
Application

Essential=True

↓

Log Collector

Essential=False
```

If the log collector fails,

the application may continue running.

Useful for

- Sidecars
- Monitoring agents
- Metrics exporters

---

# Entry Point

Specifies the executable started inside the container.

Example

```text
python app.py
```

or

```text
java -jar application.jar
```

---

# Command

Overrides the container's default command.

Docker Image

```text
CMD

python app.py
```

Task Definition

```text
Command

python test.py
```

AWS uses

```
python test.py
```

---

# Working Directory

Defines the startup directory.

Example

```text
/app
```

Application starts from

```
/app
```

instead of the image default.

---

# Environment Variables

Applications often require configuration.

Example

```text
DB_HOST

DB_PORT

ENVIRONMENT

LOG_LEVEL
```

Instead of hardcoding,

they are passed during runtime.

---

# Environment Variable Example

```text
Application

↓

DB_HOST

↓

database.company.internal

↓

Application connects
```

---

# Why Avoid Hardcoding?

Bad

```text
Database Password

Inside Source Code
```

Good

```text
Secrets Manager

↓

Environment Variable

↓

Application
```

---

# Secrets

Sensitive values include

- Passwords
- API Keys
- Database Credentials
- Tokens
- Certificates

These should never be stored inside images.

Supported integrations

- AWS Secrets Manager
- Systems Manager Parameter Store

---

# Logging Configuration

Every task should define a logging driver.

Common configuration

```text
Application

↓

stdout

↓

awslogs

↓

CloudWatch Logs
```

Benefits

- Central logging
- Easy troubleshooting
- Retention policies
- CloudWatch Insights

---

# Container Dependencies

Multiple containers may depend on each other.

Example

```text
Log Agent

↓

Starts First

↓

Application Starts
```

Useful for

- Sidecars
- Monitoring
- Service Mesh

---

# Health Checks

Task Definitions support container health checks.

Example

```text
HTTP

↓

/health

↓

200 OK
```

If health checks fail repeatedly,

ECS replaces the unhealthy task.

---

# Restart Behavior

Suppose

```text
Application

↓

Crash
```

ECS Service

↓

Launches

↓

New Task

Self-healing is automatic.

---

# Example Production Task

```text
Task

├── Payment API

├── Fluent Bit

├── CloudWatch Agent

└── Envoy Proxy
```

Each container has a specific responsibility.

---

# Enterprise Example

A banking payment service uses

```text
Task Definition

↓

Java Application

↓

Fluent Bit

↓

Secrets Manager

↓

CloudWatch Logs

↓

IAM Task Role

↓

Amazon EFS
```

When a deployment occurs,

only the Task Definition revision changes.

The ECS Service gradually replaces old tasks with new ones using a rolling deployment strategy.

No infrastructure changes are required.

---

# Best Practices

- Keep one application per container.
- Version Task Definitions.
- Store images in Amazon ECR.
- Store secrets in Secrets Manager.
- Enable health checks.
- Configure centralized logging.
- Keep containers stateless.
- Use immutable deployments.

---

# Common Mistakes

- Hardcoding credentials.
- Oversizing CPU and memory.
- Running multiple unrelated applications in one task.
- Ignoring health checks.
- Using latest image tags in production.
- Not versioning Task Definitions.

---

# Interview Questions

## Basic

- What is an ECS Task Definition?
- Why is a Task Definition required?
- What is a Task Definition revision?

## Intermediate

- Task Definition vs Running Task.
- Essential vs Non-Essential container.
- Task CPU vs Container CPU.
- Explain Environment Variables and Secrets.

## Advanced

- Design a production Task Definition for a microservice.
- Explain how ECS replaces failed tasks.
- How would you securely manage database credentials in ECS Fargate?

---

# Chapter 4 - IAM Roles, Execution Role & Task Role

One of the most frequently asked AWS Fargate interview topics is the difference between **Task Role** and **Execution Role**.

Many engineers know both exist but cannot clearly explain when each one is used.

Understanding these roles is essential for designing secure ECS/Fargate workloads.

---

# Why Does Fargate Need IAM Roles?

A container frequently needs to access AWS services.

Examples

- Read objects from Amazon S3
- Read secrets from AWS Secrets Manager
- Publish messages to Amazon SNS
- Read from Amazon SQS
- Access DynamoDB
- Write logs to CloudWatch

AWS recommends **never** storing AWS Access Keys inside containers.

Instead, IAM Roles provide temporary credentials.

---

# Authentication Flow

```text
Application

↓

IAM Role

↓

AWS STS

↓

Temporary Credentials

↓

AWS Service
```

No Access Keys

No Secret Keys

No Credential Rotation

Everything is automatic.

---

# Two IAM Roles in Fargate

AWS Fargate uses two different IAM roles.

| IAM Role | Used By |
|-----------|----------|
| Task Execution Role | ECS Agent / Fargate Platform |
| Task Role | Your Application |

Many interview questions are based on this distinction.

---

# What is an Execution Role?

The **Execution Role** is used by AWS itself while launching your task.

Your application does **not** use this role.

It gives AWS permission to perform startup operations.

---

# Execution Role Responsibilities

The Execution Role is commonly used for

- Pulling images from Amazon ECR
- Sending logs to CloudWatch
- Retrieving secrets during task startup
- Downloading environment files
- Authenticating with AWS services required before the application starts

Architecture

```text
Fargate

↓

Execution Role

↓

Amazon ECR

↓

Download Image
```

---

# Startup Flow

```text
Run Task

↓

Fargate

↓

Execution Role

↓

Amazon ECR

↓

Pull Image

↓

CloudWatch

↓

Create Log Stream

↓

Start Application
```

Notice

The application has not started yet.

---

# Example

Suppose your image is stored in

```text
Amazon ECR
```

Who downloads the image?

❌ Application

✅ Execution Role

---

# What Happens if Execution Role is Missing?

Task starts

↓

Attempts to pull image

↓

Permission denied

↓

Task fails

Common error

```text
CannotPullContainerError
```

This is one of the most common production issues.

---

# What is a Task Role?

The Task Role is completely different.

It is used **after the application starts**.

The application assumes this role automatically.

---

# Task Role Responsibilities

Your application may need to

- Read S3
- Access DynamoDB
- Read Secrets
- Publish SNS
- Consume SQS
- Invoke Lambda
- Access Parameter Store

The Task Role grants these permissions.

---

# Task Role Flow

```text
Application

↓

Task Role

↓

AWS STS

↓

Temporary Credentials

↓

Amazon S3
```

---

# Example

Java Application

↓

Task Role

↓

Secrets Manager

↓

Retrieve Database Password

↓

Connect to Database

The application never stores credentials.

---

# Task Role vs Execution Role

| Task Role | Execution Role |
|------------|----------------|
| Used by Application | Used by ECS/Fargate |
| Runtime Permissions | Startup Permissions |
| Reads S3 | Pulls ECR Images |
| Accesses DynamoDB | Creates CloudWatch Logs |
| Reads Secrets | Downloads Secrets before startup |
| Business Logic | Infrastructure Operations |

This comparison is asked in many interviews.

---

# Visual Comparison

```text
               Run Task

                  │

        Execution Role

                  │

     Pull Image from ECR

                  │

      Create CloudWatch Logs

                  │

         Start Container

                  │

          Application Runs

                  │

            Task Role

                  │

        Access AWS Services
```

---

# IAM Credential Delivery

How does the application receive credentials?

AWS automatically injects temporary credentials into the task.

```text
Application

↓

Metadata Endpoint

↓

Temporary Credentials

↓

AWS API
```

No credentials are stored inside

- Docker Image
- Source Code
- Environment Variables

---

# Temporary Credential Lifecycle

```text
Task Starts

↓

STS Issues Credentials

↓

Application Uses Credentials

↓

Credentials Rotate Automatically

↓

Task Stops

↓

Credentials Destroyed
```

This significantly improves security.

---

# Least Privilege Example

Bad

```text
Task Role

↓

AdministratorAccess
```

Every AWS service becomes accessible.

---

Good

```text
Allow

s3:GetObject

Only

arn:aws:s3:::payment-reports/*
```

Grant only the permissions required by the application.

---

# Cross-Account Access

Suppose

Application runs in

```text
Development Account
```

but reads S3 from

```text
Shared Services Account
```

Architecture

```text
Application

↓

Task Role

↓

AssumeRole

↓

STS

↓

Shared Account

↓

Amazon S3
```

This is common in enterprise environments.

---

# IAM Roles for Service Accounts (IRSA)

For Amazon EKS on Fargate,

IAM authentication is commonly implemented using **IAM Roles for Service Accounts (IRSA).**

Architecture

```text
Kubernetes Pod

↓

Service Account

↓

IAM Role

↓

STS

↓

AWS Services
```

Every Pod can receive its own IAM permissions.

This avoids sharing node-level credentials.

---

# Security Best Practices

- Never use IAM Users inside containers.
- Use IAM Roles.
- Follow least privilege.
- Use temporary credentials.
- Store secrets in AWS Secrets Manager.
- Enable CloudTrail.
- Rotate permissions through IAM policy updates.
- Avoid wildcard (*) permissions.

---

# Common Production Issues

## Image Pull Failure

Cause

Execution Role missing ECR permissions.

Resolution

Grant

- ecr:GetAuthorizationToken
- ecr:BatchGetImage
- ecr:GetDownloadUrlForLayer

---

## CloudWatch Logs Not Created

Cause

Execution Role missing CloudWatch permissions.

Resolution

Grant

- logs:CreateLogStream
- logs:PutLogEvents

---

## Application Cannot Read S3

Cause

Task Role missing permissions.

Resolution

Grant

```text
s3:GetObject
```

to the Task Role.

---

## Access Denied

Cause

Wrong role assigned.

Resolution

Verify

- Execution Role
- Task Role
- IAM Policies
- Resource Policies

---

# Enterprise Example

A payment microservice uses

- Amazon ECR
- Secrets Manager
- Amazon S3
- Amazon SNS
- CloudWatch

Startup

```text
Execution Role

↓

Pull Image

↓

Create Logs

↓

Start Container
```

Runtime

```text
Application

↓

Task Role

↓

Read Secrets

↓

Generate Invoice

↓

Upload PDF to S3

↓

Publish SNS Notification
```

Infrastructure permissions and application permissions remain completely separated.

---

# Best Practices

- Separate Task Role and Execution Role.
- One Task Role per microservice whenever possible.
- Restrict permissions using least privilege.
- Use managed IAM policies where appropriate.
- Monitor AssumeRole events with CloudTrail.
- Audit IAM policies regularly.

---

# Common Mistakes

- Using AdministratorAccess for Task Roles.
- Using the Execution Role for application permissions.
- Hardcoding AWS credentials.
- Sharing one Task Role across unrelated services.
- Forgetting to update IAM policies after adding new AWS integrations.

---

# Interview Questions

## Basic

- What is an ECS Task Role?
- What is an Execution Role?
- Why are IAM Roles preferred over IAM Users?

## Intermediate

- Task Role vs Execution Role.
- How does a Fargate task obtain AWS credentials?
- Explain the startup sequence involving the Execution Role.

## Advanced

- Design IAM permissions for a payment microservice using S3, SNS, and Secrets Manager.
- Explain how temporary credentials improve container security.
- Design cross-account access for a Fargate application using STS and AssumeRole.

---

# Chapter 5 - Amazon ECS Services, Tasks & Scheduling

One of the most important concepts in Amazon ECS is understanding the relationship between:

- Task Definition
- Task
- Service
- Cluster

Many engineers use these terms interchangeably, but they have completely different responsibilities.

Understanding these concepts is essential for designing highly available Fargate applications.

---

# ECS Deployment Hierarchy

```text
Cluster

↓

Service

↓

Task

↓

Container

↓

Application
```

Everything starts with a Task Definition.

---

# What is an ECS Task?

A Task is a **running instance** of a Task Definition.

Think of it like this:

```text
Docker Image

↓

Task Definition

↓

Task

↓

Running Application
```

If a Task Definition is a blueprint,

then a Task is the actual running application.

---

# Example

Task Definition

```text
payment-api:v5
```

Run Task

↓

AWS launches

```text
Task-1
```

Run again

↓

AWS launches

```text
Task-2
```

Both tasks use the same Task Definition.

---

# Task Lifecycle

```text
Provisioning

↓

Pending

↓

Pulling Image

↓

Starting

↓

Running

↓

Stopping

↓

Stopped
```

AWS automatically removes infrastructure after the task finishes.

---

# Task States

| State | Meaning |
|---------|---------|
| PROVISIONING | Resources being allocated |
| PENDING | Waiting to start |
| ACTIVATING | Preparing networking and storage |
| RUNNING | Application executing |
| DEACTIVATING | Cleaning resources |
| STOPPING | Task shutting down |
| STOPPED | Task completed |

Understanding these states helps during troubleshooting.

---

# What is an ECS Service?

An ECS Service ensures that a specified number of Tasks are always running.

Without a Service,

tasks stop permanently after failure.

Example

```text
Run Task

↓

Application

↓

Crash

↓

Nothing Happens
```

Application remains unavailable.

---

With a Service

```text
Task

↓

Crash

↓

Service Detects Failure

↓

Launch New Task

↓

Application Restored
```

Self-healing is automatic.

---

# Why Services Exist

Suppose you need

```
5 Running Containers
```

A Service continuously maintains

```
Desired Count = 5
```

If one task crashes

```text
5

↓

4

↓

Service Detects

↓

Launch New Task

↓

5
```

The desired count is restored automatically.

---

# ECS Service Architecture

```text
Application Load Balancer

           │

      ECS Service

           │

   Desired Tasks = 3

     │      │      │

   Task1  Task2  Task3
```

The Service manages all running tasks.

---

# Desired Count

Desired Count specifies

**How many tasks should always remain running?**

Example

```text
Desired Count

3
```

Running

```text
Task-1

Task-2

Task-3
```

One crashes

↓

Service launches

↓

Task-4

Desired Count remains

```
3
```

---

# Run Task vs Create Service

This is one of the most common interview questions.

| Run Task | ECS Service |
|-----------|-------------|
| One-time execution | Long-running application |
| No self-healing | Self-healing |
| Manual scaling | Auto Scaling supported |
| Batch jobs | APIs & Microservices |
| Stops after completion | Continuously maintained |

---

# When to Use Run Task

Examples

- Database Migration
- Batch Processing
- One-time Scripts
- Report Generation
- Data Import
- Scheduled Jobs

Architecture

```text
Run Task

↓

Execute

↓

Complete

↓

Stopped
```

---

# When to Use ECS Service

Examples

- REST API
- Web Application
- Payment Service
- Authentication Service
- Notification Service

Architecture

```text
Service

↓

Tasks Always Running

↓

Load Balancer

↓

Users
```

---

# ECS Scheduler

The ECS Scheduler determines

- Where tasks run
- Which Availability Zone
- Which subnet
- Which compute capacity
- Resource availability

Flow

```text
Run Service

↓

Scheduler

↓

Capacity Check

↓

Launch Task

↓

Running
```

---

# Service Scheduler vs Task Scheduler

Task Scheduler

- Launches tasks

Service Scheduler

- Maintains desired task count
- Replaces failed tasks
- Performs deployments
- Handles scaling

---

# ECS Cluster

An ECS Cluster is a logical grouping of ECS Services and Tasks.

Architecture

```text
Cluster

├── Payment Service

├── User Service

├── Inventory Service

├── Order Service

└── Notification Service
```

Clusters organize workloads.

---

# Cluster with Fargate

```text
Amazon ECS Cluster

        │

────────────────────────

Payment Service

↓

Fargate Tasks

────────────────────────

Order Service

↓

Fargate Tasks

────────────────────────

Inventory Service

↓

Fargate Tasks
```

AWS provisions compute independently for every task.

---

# Service Discovery

Instead of hardcoding IP addresses,

services communicate using DNS.

Example

Bad

```text
10.0.15.26
```

Good

```text
payment.internal

inventory.internal

orders.internal
```

Benefits

- Easier scaling
- Easier deployments
- No IP dependency

---

# Load Balancer Integration

Most production services sit behind an Application Load Balancer.

```text
Users

↓

Application Load Balancer

↓

ECS Service

↓

Task-1

Task-2

Task-3
```

The ALB distributes traffic.

---

# Health Checks

The ALB continuously checks task health.

```text
ALB

↓

/health

↓

200 OK

↓

Healthy
```

If a task becomes unhealthy

```text
ALB

↓

Unhealthy

↓

ECS Service

↓

Replace Task
```

---

# Rolling Deployment

Suppose

Version

```
v1
```

is running.

Deploy

```
v2
```

Flow

```text
Task-v1

↓

Launch Task-v2

↓

Health Check

↓

Traffic Shift

↓

Terminate Task-v1
```

Users experience little or no downtime.

---

# Deployment Configuration

Important parameters

Minimum Healthy Percent

Maximum Percent

Example

Desired

```
4 Tasks
```

Maximum Percent

```
200%
```

AWS may temporarily run

```
8 Tasks
```

during deployment.

---

# Service Auto Recovery

Scenario

```text
Task

↓

Application Crash

↓

Health Check Fails

↓

Task Stops

↓

Service Launches New Task
```

Recovery is automatic.

---

# Production Architecture

```text
Users

↓

Route53

↓

Application Load Balancer

↓

Amazon ECS Service

↓

AWS Fargate

↓

Task-1

Task-2

Task-3

↓

Amazon RDS

↓

Amazon ElastiCache
```

Characteristics

- Multi-AZ
- Self-healing
- Load balanced
- Auto scalable
- Highly available

---

# Best Practices

- Use Services for long-running workloads.
- Use Run Task for batch jobs.
- Deploy behind an ALB.
- Enable health checks.
- Use rolling deployments.
- Keep desired count greater than one in production.
- Distribute tasks across multiple Availability Zones.

---

# Common Mistakes

- Using Run Task for production APIs.
- Desired Count set to one for critical services.
- Ignoring ALB health checks.
- Running all tasks in one Availability Zone.
- Hardcoding service IP addresses.

---

# Interview Questions

## Basic

- What is an ECS Task?
- What is an ECS Service?
- What is an ECS Cluster?

## Intermediate

- Run Task vs ECS Service.
- Explain Desired Count.
- Explain ECS Scheduler.
- How does ECS replace failed tasks?

## Advanced

- Design a highly available payment service using ECS Fargate.
- Explain the complete deployment flow from Task Definition to Running Service.
- How does ECS maintain application availability during rolling deployments?

---

# Chapter 6 - Service Auto Scaling, Capacity Providers & Fargate Spot

One of the biggest advantages of AWS Fargate is that applications can scale automatically without provisioning additional EC2 instances.

Unlike traditional infrastructure where engineers first scale servers and then applications, Fargate provisions compute automatically whenever new tasks are required.

This makes scaling much simpler and more responsive.

---

# Understanding Scaling

Consider an e-commerce application.

Normal traffic

```text
Users

↓

200 Requests/Minute

↓

2 Tasks
```

During a sale

```text
Users

↓

10,000 Requests/Minute

↓

20 Tasks
```

Instead of manually launching servers,

Amazon ECS automatically launches additional Fargate tasks.

---

# Types of Scaling

ECS supports three major scaling mechanisms.

| Scaling Type | Purpose |
|--------------|---------|
| Service Auto Scaling | Scale the number of tasks |
| Cluster Auto Scaling | Scale EC2 instances (EC2 Launch Type only) |
| Application Auto Scaling | Underlying AWS scaling service used by ECS |

For AWS Fargate,

only **Service Auto Scaling** is required because AWS manages the compute infrastructure.

---

# Service Auto Scaling Architecture

```text
Users

↓

Application Load Balancer

↓

Amazon ECS Service

↓

CloudWatch Metrics

↓

Application Auto Scaling

↓

Increase Tasks

↓

AWS Fargate
```

The scaling decision is automatic.

---

# Service Auto Scaling Workflow

```text
CPU Utilization

↓

CloudWatch Alarm

↓

Application Auto Scaling

↓

ECS Service

↓

Launch New Task

↓

Fargate Provisions Compute

↓

Application Capacity Increased
```

No EC2 instances are launched.

Only new Fargate tasks are created.

---

# Desired Count vs Running Count

Every ECS Service maintains

```text
Desired Count
```

Example

```text
Desired

4

Running

4
```

If Auto Scaling increases capacity

```text
Desired

10

↓

Scheduler

↓

Running

10
```

The Service always tries to match the desired count.

---

# Scaling Metrics

Auto Scaling can use multiple metrics.

Common metrics

- CPU Utilization
- Memory Utilization
- Request Count
- ALB Target Response Time
- SQS Queue Length
- Custom CloudWatch Metrics

Example

```text
CPU

85%

↓

Scale Out
```

---

# Target Tracking Scaling

The most commonly used scaling policy.

Goal

Maintain a target utilization.

Example

```text
Target CPU

50%
```

Scenario

```text
CPU

30%

↓

No Action

CPU

70%

↓

Scale Out

CPU

15%

↓

Scale In
```

AWS continuously adjusts the number of running tasks to maintain the target.

---

# Step Scaling

Scaling occurs in predefined steps.

Example

| CPU | Action |
|------|--------|
|60%|Add 2 Tasks|
|75%|Add 5 Tasks|
|90%|Add 10 Tasks|

Useful for applications with predictable traffic spikes.

---

# Scheduled Scaling

Applications with known traffic patterns can scale based on time.

Example

```text
08:00 AM

↓

10 Tasks

12:00 PM

↓

25 Tasks

09:00 PM

↓

5 Tasks
```

Useful for

- Business applications
- Office portals
- Daily batch processing

---

# Predictive Scaling

AWS analyzes historical traffic patterns to predict future demand.

Although commonly associated with EC2 Auto Scaling, predictive approaches can complement container workloads depending on the overall architecture.

---

# Scale Out Process

Suppose CPU reaches

```text
85%
```

Workflow

```text
CloudWatch

↓

Alarm Triggered

↓

Application Auto Scaling

↓

Increase Desired Count

↓

Scheduler

↓

Launch Fargate Task

↓

Load Balancer Registers Task
```

Traffic is automatically distributed.

---

# Scale In Process

Traffic decreases.

```text
CPU

15%

↓

CloudWatch

↓

Decrease Desired Count

↓

Stop Excess Tasks

↓

Release Compute
```

AWS immediately stops billing for released Fargate resources.

---

# Cooldown Period

Without cooldown,

Auto Scaling may continuously add and remove tasks.

Example

```text
Scale Out

↓

Traffic Drops

↓

Scale In

↓

Traffic Rises

↓

Scale Out Again
```

This causes scaling oscillation.

Cooldown allows the system to stabilize before another scaling decision.

---

# Minimum and Maximum Capacity

Example

```text
Minimum Tasks

2

Maximum Tasks

25
```

Even during low traffic,

at least

```text
2 Tasks
```

remain available.

---

# Scaling Example

Morning

```text
2 Tasks
```

Lunch

```text
8 Tasks
```

Evening Sale

```text
20 Tasks
```

Night

```text
2 Tasks
```

Everything happens automatically.

---

# Capacity Providers

Capacity Providers determine **where ECS runs your tasks**.

Instead of manually selecting compute every time,

the Service uses a Capacity Provider strategy.

---

# Capacity Provider Types

| Capacity Provider | Compute |
|-------------------|---------|
| Fargate | Standard Fargate |
| Fargate Spot | Spare AWS capacity |
| EC2 Capacity Provider | ECS EC2 Instances |

---

# Fargate Capacity Provider

Architecture

```text
ECS Service

↓

Fargate Capacity Provider

↓

AWS Fargate
```

AWS provisions on-demand compute.

Highly reliable.

Suitable for production.

---

# Fargate Spot

AWS often has unused compute capacity.

Instead of leaving it idle,

AWS offers it at a significant discount.

Architecture

```text
Unused AWS Capacity

↓

Fargate Spot

↓

Container Workloads
```

---

# Benefits of Fargate Spot

- Lower cost
- No infrastructure management
- Automatic provisioning
- Same developer experience as standard Fargate

Savings can be substantial depending on workload characteristics.

---

# Limitation of Fargate Spot

AWS can reclaim Spot capacity when needed.

Workflow

```text
AWS Needs Capacity

↓

Spot Interruption Notice

↓

Task Stops

↓

Capacity Removed
```

Therefore,

Spot should only be used for interruptible workloads.

---

# Suitable Spot Workloads

Examples

- Batch Jobs
- Data Processing
- Machine Learning Training
- CI/CD Pipelines
- Image Processing
- Background Workers

Avoid Spot for

- Payment APIs
- Authentication Services
- Banking Transactions
- Critical Production Workloads without fallback

---

# Mixing Fargate and Fargate Spot

Production services often combine both.

Example

```text
10 Tasks

↓

2 Standard Fargate

↓

8 Fargate Spot
```

If Spot capacity disappears,

critical tasks continue running on standard Fargate.

---

# Capacity Provider Strategy

Example

```text
Base

2 Tasks

↓

Standard Fargate

Weight

8

↓

Fargate Spot
```

The first two tasks always use standard Fargate.

Additional tasks prefer Spot capacity.

This balances reliability and cost.

---

# Enterprise Architecture

```text
Users

↓

Application Load Balancer

↓

Amazon ECS Service

↓

Capacity Provider Strategy

├── Fargate

└── Fargate Spot

↓

AWS Fargate

↓

Running Tasks
```

---

# Production Example

An online retail platform experiences

- Normal traffic during weekdays
- Massive spikes during festive sales

Configuration

```text
Minimum

4 Standard Tasks

↓

Auto Scaling

↓

Additional Tasks

↓

Fargate Spot
```

Benefits

- High availability
- Reduced infrastructure cost
- Automatic scaling
- Minimal operational effort

---

# Best Practices

- Use Target Tracking for most applications.
- Set sensible minimum and maximum task counts.
- Configure cooldown periods.
- Use CloudWatch metrics for scaling decisions.
- Use Fargate Spot only for interruptible workloads.
- Mix Fargate and Fargate Spot for production.
- Continuously monitor scaling events.

---

# Common Mistakes

- Scaling only on CPU when memory is the bottleneck.
- Minimum task count set to one for production APIs.
- Using only Spot capacity for critical services.
- Aggressive cooldown settings causing oscillation.
- Ignoring CloudWatch alarms.

---

# Interview Questions

## Basic

- What is ECS Service Auto Scaling?
- What metrics can trigger scaling?
- What is Fargate Spot?

## Intermediate

- Target Tracking vs Step Scaling.
- Explain Capacity Providers.
- How does Fargate Spot reduce costs?
- Explain the scale-out workflow.

## Advanced

- Design an auto-scaling architecture for a payment platform.
- Explain how you would combine Fargate and Fargate Spot in production.
- Design a cost-optimized ECS architecture that can handle unpredictable traffic spikes while maintaining high availability.

---

# Chapter 7 - Storage, Persistent Data & Networking

Containers are designed to be **ephemeral**.

This means a container can be started, stopped, replaced, or deleted at any time.

Therefore, understanding storage is critical when designing production workloads on AWS Fargate.

---

# Ephemeral Storage

By default, every Fargate task receives ephemeral storage.

```text
Fargate Task

↓

Ephemeral Storage

↓

Container
```

Characteristics

- Temporary
- Automatically created
- Automatically deleted
- Local to the task

If the task stops,

all data stored in ephemeral storage is lost.

---

# What Can Ephemeral Storage Be Used For?

Typical use cases

- Temporary files
- Log buffering
- Cache files
- Downloaded files
- Image processing
- Intermediate data

Do NOT store

- Customer uploads
- Database files
- Business documents
- Long-term application data

---

# Lifecycle

```text
Task Starts

↓

Storage Created

↓

Application Uses Storage

↓

Task Stops

↓

Storage Deleted
```

This is why containers should remain stateless.

---

# Stateless Applications

A stateless application does not depend on local storage.

Example

```text
User Request

↓

Container

↓

Database

↓

Response
```

If the container is replaced,

nothing is lost.

---

# Stateful Applications

Stateful applications store important information locally.

Example

```text
Database

↓

Container Disk
```

If the task is replaced,

data is lost.

Running stateful databases directly on Fargate is generally not recommended.

---

# Externalizing State

Instead of storing data inside containers,

AWS recommends

```text
Container

↓

Amazon RDS

Amazon DynamoDB

Amazon S3

Amazon EFS

Amazon ElastiCache
```

The container becomes disposable.

---

# Amazon EFS Integration

AWS Fargate supports mounting Amazon Elastic File System (EFS).

Architecture

```text
Fargate Task

↓

Amazon EFS

↓

Shared Files
```

Multiple tasks can access the same file system simultaneously.

---

# Why Use Amazon EFS?

Common use cases

- Shared uploads
- Shared configuration
- ML models
- Static assets
- Reports
- Documents

Unlike ephemeral storage,

EFS persists even after tasks stop.

---

# Multi-Task Example

```text
Task-A

↓

Amazon EFS

↑

Task-B

↑

Task-C
```

All tasks read and write the same files.

---

# EFS Architecture

```text
Application

↓

Fargate Task

↓

Mount Target

↓

Amazon EFS

↓

Persistent Storage
```

EFS automatically scales storage as data grows.

---

# EFS Security

Access is controlled using

- Security Groups
- IAM Policies
- EFS Access Points
- POSIX Permissions

Production workloads should use **EFS Access Points** to isolate applications.

---

# Amazon S3 vs Amazon EFS

| Amazon S3 | Amazon EFS |
|------------|------------|
| Object Storage | File Storage |
| REST API Access | File System Mount |
| Unlimited Objects | Shared File System |
| Ideal for Files | Ideal for Shared Directories |

Use S3 for object storage.

Use EFS when applications require a mounted file system.

---

# Temporary Storage vs EFS

| Ephemeral Storage | Amazon EFS |
|-------------------|------------|
| Temporary | Persistent |
| Deleted after task stops | Data retained |
| Fast local storage | Shared network storage |
| Single Task | Multiple Tasks |

---

# Networking Overview

Every Fargate task receives

- Private IP
- Elastic Network Interface
- Security Group
- Route Table
- DNS Resolution

Unlike Docker bridge networking,

tasks behave like independent EC2 instances.

---

# Network Architecture

```text
VPC

↓

Private Subnet

↓

Elastic Network Interface

↓

Fargate Task

↓

Application
```

Every task has its own network identity.

---

# awsvpc Network Mode

Fargate supports

```text
awsvpc
```

mode only.

Example

```text
Task-1

↓

10.0.1.20

Task-2

↓

10.0.1.21

Task-3

↓

10.0.1.22
```

Each task has its own IP address.

---

# Why awsvpc?

Advantages

- Better isolation
- Native VPC networking
- Security Groups per task
- Direct ALB integration
- Simplified routing

---

# Security Groups

Instead of assigning Security Groups to EC2,

Fargate assigns them directly to tasks.

Example

```text
Frontend Task

↓

Allow

443

↓

Backend Task

↓

Allow

8080

↓

Database

↓

3306
```

Each layer can have its own security policy.

---

# Public vs Private Subnets

Production recommendation

```text
Internet

↓

ALB

↓

Private Subnet

↓

Fargate Tasks
```

Tasks should normally not receive public IPs.

Instead,

the Application Load Balancer receives internet traffic.

---

# Internet Access

Private subnet tasks requiring outbound internet

```text
Task

↓

NAT Gateway

↓

Internet
```

Common examples

- Download packages
- Call third-party APIs
- Send notifications

---

# VPC Endpoints

Instead of using a NAT Gateway,

Fargate tasks can privately access AWS services.

Example

```text
Task

↓

VPC Endpoint

↓

Amazon S3
```

No internet access required.

Supported services include

- Amazon S3
- Amazon ECR
- CloudWatch Logs
- Secrets Manager
- Systems Manager

Benefits

- Improved security
- Reduced NAT Gateway costs
- Private AWS backbone connectivity

---

# Service Discovery

Instead of using IP addresses,

services communicate using DNS.

```text
Payment Service

↓

payment.internal

↓

Inventory Service

↓

inventory.internal
```

IP changes do not affect applications.

---

# Internal Communication

Example

```text
Order Service

↓

payment.internal

↓

Payment Service

↓

Amazon RDS
```

This is typical in microservice architectures.

---

# Network Packet Flow

```text
User

↓

Application Load Balancer

↓

Security Group

↓

Fargate Task

↓

Application

↓

Amazon RDS
```

Every request passes through VPC networking controls.

---

# Enterprise Architecture

```text
Internet

↓

Route53

↓

Application Load Balancer

↓

Private Subnets

↓

Fargate Tasks

↓

Amazon EFS

↓

Amazon RDS

↓

Secrets Manager
```

Characteristics

- Multi-AZ
- Private networking
- Persistent storage
- Stateless containers
- Centralized secrets
- Secure architecture

---

# Best Practices

- Keep containers stateless.
- Use EFS only when shared storage is required.
- Store uploads in Amazon S3.
- Run tasks in private subnets.
- Use Security Groups per application tier.
- Use VPC Endpoints for AWS services.
- Avoid assigning public IPs to production tasks.

---

# Common Mistakes

- Storing business data in ephemeral storage.
- Running databases inside Fargate containers.
- Using public subnets unnecessarily.
- Hardcoding IP addresses.
- Sharing one Security Group across every application.
- Ignoring VPC Endpoint opportunities.

---

# Interview Questions

## Basic

- What is ephemeral storage?
- What is Amazon EFS?
- Why are containers considered stateless?

## Intermediate

- Amazon EFS vs Amazon S3.
- Public vs Private Subnets for Fargate.
- Why does Fargate use awsvpc mode?
- How do Fargate tasks access the internet?

## Advanced

- Design persistent storage for a document management application running on Fargate.
- Explain how to securely connect Fargate tasks to Amazon RDS without exposing them to the internet.
- Design a highly available storage and networking architecture for an enterprise microservices platform using AWS Fargate.

---

# Chapter 8 - Logging, Monitoring, Observability & Security

Running containers in production is not enough.

Enterprise environments require complete visibility into

- Application Health
- Infrastructure Health
- Performance
- Errors
- Security
- User Traffic

Without proper observability,

troubleshooting production incidents becomes extremely difficult.

---

# Observability Pillars

Modern observability consists of three pillars.

```text
                 Observability

        ┌──────────┼──────────┐

        Logs     Metrics     Traces
```

Although distributed tracing is optional depending on the observability stack, **Logs** and **Metrics** are essential for every Fargate workload.

---

# Logging Architecture

Every application writes logs to

```text
stdout

stderr
```

AWS captures these streams automatically.

```text
Application

↓

stdout

↓

awslogs Driver

↓

CloudWatch Logs

↓

CloudWatch Insights
```

No log files need to be managed inside the container.

---

# Why Log to stdout?

Containers are ephemeral.

If logs remain inside the container,

they disappear when the task stops.

Instead

```text
Container

↓

stdout

↓

CloudWatch Logs
```

Logs remain available even after the task is terminated.

---

# CloudWatch Logs

CloudWatch Logs provides

- Centralized logging
- Search
- Retention
- Export
- Log Insights
- Metric Filters

Architecture

```text
Application

↓

CloudWatch Logs

↓

Log Groups

↓

Log Streams
```

---

# Log Group

A Log Group stores logs for an application.

Example

```text
/payment-service

/order-service

/inventory-service
```

Retention policies are configured at the Log Group level.

---

# Log Stream

Each running task creates its own Log Stream.

Example

```text
Payment Service

↓

Task-1

↓

Log Stream

Task-2

↓

Log Stream

Task-3

↓

Log Stream
```

This makes troubleshooting individual tasks much easier.

---

# CloudWatch Logs Insights

Logs can be queried using CloudWatch Logs Insights.

Example questions

- Which requests failed?
- Which task generated the error?
- How many exceptions occurred?
- Which endpoint is slow?

Instead of manually reading logs,

engineers can search millions of log entries efficiently.

---

# Structured Logging

Avoid

```text
Database Error
```

Prefer

```json
{
  "timestamp":"2026-08-05T09:30:00Z",
  "service":"payment-api",
  "requestId":"12345",
  "userId":"56789",
  "status":"FAILED",
  "error":"Database Connection Timeout"
}
```

Structured logs are easier to search and analyze.

---

# Logging Best Practices

Include

- Timestamp
- Log Level
- Service Name
- Request ID
- Correlation ID
- User ID (where appropriate)
- Error Details

Avoid logging

- Passwords
- API Keys
- Tokens
- Secrets
- Personally identifiable information (PII)

---

# CloudWatch Metrics

Metrics measure application and infrastructure health.

Common metrics

- CPU Utilization
- Memory Utilization
- Task Count
- Network In
- Network Out
- Running Tasks
- Failed Tasks

Architecture

```text
Fargate Task

↓

CloudWatch Metrics

↓

Dashboard

↓

Alarm
```

---

# CloudWatch Alarms

Alarms notify engineers when thresholds are exceeded.

Example

```text
CPU > 80%

↓

CloudWatch Alarm

↓

SNS

↓

Email

↓

DevOps Team
```

Other examples

- Memory > 90%
- Running Tasks < Desired Count
- High Error Rate
- ALB 5XX Errors

---

# Container Insights

CloudWatch Container Insights provides detailed metrics for ECS and Fargate workloads.

It collects

- CPU Usage
- Memory Usage
- Task Count
- Service Metrics
- Network Statistics

Architecture

```text
Amazon ECS

↓

Container Insights

↓

CloudWatch

↓

Dashboard
```

Container Insights provides much deeper visibility than standard CloudWatch metrics.

---

# Monitoring Dashboard

A typical production dashboard displays

```text
CPU

Memory

Running Tasks

Response Time

ALB Requests

HTTP Errors

Network Traffic
```

Operations teams can identify issues quickly.

---

# Health Checks

Two levels of health checks are commonly used.

### Container Health Check

Checks whether the application inside the container is healthy.

```text
Application

↓

/health

↓

Healthy
```

---

### Load Balancer Health Check

The Application Load Balancer determines whether traffic should be sent to a task.

```text
ALB

↓

Task

↓

200 OK

↓

Healthy
```

If unhealthy,

traffic is automatically redirected to healthy tasks.

---

# Security Architecture

Security begins before the container starts.

```text
Developer

↓

Build Image

↓

Amazon ECR

↓

Image Scan

↓

Fargate

↓

IAM Roles

↓

Application
```

Security exists throughout the deployment lifecycle.

---

# Image Security

Container images should

- Be minimal
- Be updated regularly
- Use trusted base images
- Remove unnecessary packages
- Avoid root user

Example

Instead of

```text
ubuntu:latest
```

Prefer a minimal, well-maintained base image appropriate for your application.

Smaller images

- Download faster
- Contain fewer vulnerabilities
- Improve startup time

---

# Image Scanning

Amazon ECR supports vulnerability scanning.

Workflow

```text
Docker Image

↓

Amazon ECR

↓

Image Scan

↓

Security Report
```

Detected issues should be remediated before deployment.

---

# Secrets Management

Never store

- Passwords
- API Keys
- Certificates

inside

- Dockerfile
- Source Code
- Git Repository
- Container Image

Instead

```text
Application

↓

Task Role

↓

Secrets Manager

↓

Database Password
```

---

# Network Security

Production architecture

```text
Internet

↓

Application Load Balancer

↓

Private Subnet

↓

Fargate Tasks

↓

Amazon RDS
```

Security Groups control communication between every layer.

---

# IAM Security

Use

- Task Role
- Execution Role
- Least Privilege

Never

- Use AdministratorAccess unnecessarily
- Embed Access Keys
- Share IAM Roles across unrelated services

---

# Enterprise Security Architecture

```text
Developer

↓

CI/CD

↓

Amazon ECR

↓

Image Scan

↓

Amazon ECS

↓

Fargate

↓

Task Role

↓

Secrets Manager

↓

Application
```

Every stage has security controls.

---

# Production Monitoring Example

A payment platform monitors

- CPU Utilization
- Memory Usage
- ALB Response Time
- HTTP 5XX Errors
- Running Tasks
- Failed Deployments
- CloudWatch Logs
- Container Insights

Alerts are sent to the DevOps team through Amazon SNS whenever thresholds are exceeded.

---

# Best Practices

- Use structured logging.
- Store logs in CloudWatch.
- Enable Container Insights.
- Configure CloudWatch Alarms.
- Scan container images regularly.
- Use Secrets Manager.
- Keep containers in private subnets.
- Follow least privilege IAM.
- Monitor deployment failures.

---

# Common Mistakes

- Logging secrets.
- Using unstructured logs.
- Ignoring log retention policies.
- Running containers as root.
- Disabling image scanning.
- Ignoring CloudWatch alarms.
- No monitoring dashboards.

---

# Interview Questions

## Basic

- Where are Fargate logs stored?
- What is CloudWatch Logs?
- What is Container Insights?

## Intermediate

- CloudWatch Metrics vs CloudWatch Logs.
- Explain Container Insights.
- Why should applications log to stdout?
- How do you securely manage secrets?

## Advanced

- Design an observability solution for 200 Fargate microservices.
- Explain a production monitoring architecture using CloudWatch, Container Insights, and SNS.
- Design a secure Fargate deployment with image scanning, IAM Roles, Secrets Manager, and private networking.

---

# Chapter 8 - Logging, Monitoring, Observability & Security

Running containers in production is not enough.

Enterprise environments require complete visibility into

- Application Health
- Infrastructure Health
- Performance
- Errors
- Security
- User Traffic

Without proper observability,

troubleshooting production incidents becomes extremely difficult.

---

# Observability Pillars

Modern observability consists of three pillars.

```text
                 Observability

        ┌──────────┼──────────┐

        Logs     Metrics     Traces
```

Although distributed tracing is optional depending on the observability stack, **Logs** and **Metrics** are essential for every Fargate workload.

---

# Logging Architecture

Every application writes logs to

```text
stdout

stderr
```

AWS captures these streams automatically.

```text
Application

↓

stdout

↓

awslogs Driver

↓

CloudWatch Logs

↓

CloudWatch Insights
```

No log files need to be managed inside the container.

---

# Why Log to stdout?

Containers are ephemeral.

If logs remain inside the container,

they disappear when the task stops.

Instead

```text
Container

↓

stdout

↓

CloudWatch Logs
```

Logs remain available even after the task is terminated.

---

# CloudWatch Logs

CloudWatch Logs provides

- Centralized logging
- Search
- Retention
- Export
- Log Insights
- Metric Filters

Architecture

```text
Application

↓

CloudWatch Logs

↓

Log Groups

↓

Log Streams
```

---

# Log Group

A Log Group stores logs for an application.

Example

```text
/payment-service

/order-service

/inventory-service
```

Retention policies are configured at the Log Group level.

---

# Log Stream

Each running task creates its own Log Stream.

Example

```text
Payment Service

↓

Task-1

↓

Log Stream

Task-2

↓

Log Stream

Task-3

↓

Log Stream
```

This makes troubleshooting individual tasks much easier.

---

# CloudWatch Logs Insights

Logs can be queried using CloudWatch Logs Insights.

Example questions

- Which requests failed?
- Which task generated the error?
- How many exceptions occurred?
- Which endpoint is slow?

Instead of manually reading logs,

engineers can search millions of log entries efficiently.

---

# Structured Logging

Avoid

```text
Database Error
```

Prefer

```json
{
  "timestamp":"2026-08-05T09:30:00Z",
  "service":"payment-api",
  "requestId":"12345",
  "userId":"56789",
  "status":"FAILED",
  "error":"Database Connection Timeout"
}
```

Structured logs are easier to search and analyze.

---

# Logging Best Practices

Include

- Timestamp
- Log Level
- Service Name
- Request ID
- Correlation ID
- User ID (where appropriate)
- Error Details

Avoid logging

- Passwords
- API Keys
- Tokens
- Secrets
- Personally identifiable information (PII)

---

# CloudWatch Metrics

Metrics measure application and infrastructure health.

Common metrics

- CPU Utilization
- Memory Utilization
- Task Count
- Network In
- Network Out
- Running Tasks
- Failed Tasks

Architecture

```text
Fargate Task

↓

CloudWatch Metrics

↓

Dashboard

↓

Alarm
```

---

# CloudWatch Alarms

Alarms notify engineers when thresholds are exceeded.

Example

```text
CPU > 80%

↓

CloudWatch Alarm

↓

SNS

↓

Email

↓

DevOps Team
```

Other examples

- Memory > 90%
- Running Tasks < Desired Count
- High Error Rate
- ALB 5XX Errors

---

# Container Insights

CloudWatch Container Insights provides detailed metrics for ECS and Fargate workloads.

It collects

- CPU Usage
- Memory Usage
- Task Count
- Service Metrics
- Network Statistics

Architecture

```text
Amazon ECS

↓

Container Insights

↓

CloudWatch

↓

Dashboard
```

Container Insights provides much deeper visibility than standard CloudWatch metrics.

---

# Monitoring Dashboard

A typical production dashboard displays

```text
CPU

Memory

Running Tasks

Response Time

ALB Requests

HTTP Errors

Network Traffic
```

Operations teams can identify issues quickly.

---

# Health Checks

Two levels of health checks are commonly used.

### Container Health Check

Checks whether the application inside the container is healthy.

```text
Application

↓

/health

↓

Healthy
```

---

### Load Balancer Health Check

The Application Load Balancer determines whether traffic should be sent to a task.

```text
ALB

↓

Task

↓

200 OK

↓

Healthy
```

If unhealthy,

traffic is automatically redirected to healthy tasks.

---

# Security Architecture

Security begins before the container starts.

```text
Developer

↓

Build Image

↓

Amazon ECR

↓

Image Scan

↓

Fargate

↓

IAM Roles

↓

Application
```

Security exists throughout the deployment lifecycle.

---

# Image Security

Container images should

- Be minimal
- Be updated regularly
- Use trusted base images
- Remove unnecessary packages
- Avoid root user

Example

Instead of

```text
ubuntu:latest
```

Prefer a minimal, well-maintained base image appropriate for your application.

Smaller images

- Download faster
- Contain fewer vulnerabilities
- Improve startup time

---

# Image Scanning

Amazon ECR supports vulnerability scanning.

Workflow

```text
Docker Image

↓

Amazon ECR

↓

Image Scan

↓

Security Report
```

Detected issues should be remediated before deployment.

---

# Secrets Management

Never store

- Passwords
- API Keys
- Certificates

inside

- Dockerfile
- Source Code
- Git Repository
- Container Image

Instead

```text
Application

↓

Task Role

↓

Secrets Manager

↓

Database Password
```

---

# Network Security

Production architecture

```text
Internet

↓

Application Load Balancer

↓

Private Subnet

↓

Fargate Tasks

↓

Amazon RDS
```

Security Groups control communication between every layer.

---

# IAM Security

Use

- Task Role
- Execution Role
- Least Privilege

Never

- Use AdministratorAccess unnecessarily
- Embed Access Keys
- Share IAM Roles across unrelated services

---

# Enterprise Security Architecture

```text
Developer

↓

CI/CD

↓

Amazon ECR

↓

Image Scan

↓

Amazon ECS

↓

Fargate

↓

Task Role

↓

Secrets Manager

↓

Application
```

Every stage has security controls.

---

# Production Monitoring Example

A payment platform monitors

- CPU Utilization
- Memory Usage
- ALB Response Time
- HTTP 5XX Errors
- Running Tasks
- Failed Deployments
- CloudWatch Logs
- Container Insights

Alerts are sent to the DevOps team through Amazon SNS whenever thresholds are exceeded.

---

# Best Practices

- Use structured logging.
- Store logs in CloudWatch.
- Enable Container Insights.
- Configure CloudWatch Alarms.
- Scan container images regularly.
- Use Secrets Manager.
- Keep containers in private subnets.
- Follow least privilege IAM.
- Monitor deployment failures.

---

# Common Mistakes

- Logging secrets.
- Using unstructured logs.
- Ignoring log retention policies.
- Running containers as root.
- Disabling image scanning.
- Ignoring CloudWatch alarms.
- No monitoring dashboards.

---

# Interview Questions

## Basic

- Where are Fargate logs stored?
- What is CloudWatch Logs?
- What is Container Insights?

## Intermediate

- CloudWatch Metrics vs CloudWatch Logs.
- Explain Container Insights.
- Why should applications log to stdout?
- How do you securely manage secrets?

## Advanced

- Design an observability solution for 200 Fargate microservices.
- Explain a production monitoring architecture using CloudWatch, Container Insights, and SNS.
- Design a secure Fargate deployment with image scanning, IAM Roles, Secrets Manager, and private networking.

---

# Chapter 10 - CI/CD, Image Management & Production Deployment

AWS Fargate is only one part of a production container platform.

A complete enterprise deployment pipeline includes

- Source Code
- Build
- Testing
- Security Scanning
- Container Registry
- Deployment
- Monitoring
- Rollback

Modern DevOps pipelines automate every stage.

---

# Complete Deployment Architecture

```text
Developer

↓

GitHub

↓

GitHub Actions / Jenkins

↓

Build Docker Image

↓

Unit Testing

↓

SonarQube

↓

Trivy Scan

↓

Amazon ECR

↓

Amazon ECS Service

↓

AWS Fargate

↓

Application Load Balancer

↓

Users
```

This is one of the most common production architectures.

---

# Container Build Workflow

Step 1

Developer pushes code.

```text
Git Push

↓

GitHub
```

---

Step 2

Pipeline starts automatically.

```text
GitHub

↓

GitHub Actions
```

or

```text
GitHub

↓

Jenkins
```

---

Step 3

Docker Image is built.

```text
Dockerfile

↓

Docker Build

↓

Container Image
```

---

Step 4

Image is scanned.

```text
Docker Image

↓

Trivy

↓

Vulnerability Report
```

Critical vulnerabilities should block deployment.

---

Step 5

Push Image

```text
Docker Image

↓

Amazon ECR
```

Every image should use immutable version tags.

---

Step 6

Deploy

```text
Amazon ECS

↓

Update Service

↓

Rolling Deployment

↓

AWS Fargate
```

No manual deployment is required.

---

# Amazon ECR

Amazon Elastic Container Registry stores Docker images.

Architecture

```text
Developer

↓

Docker Image

↓

Amazon ECR

↓

Amazon ECS

↓

AWS Fargate
```

---

# Why Amazon ECR?

Benefits

- Fully managed
- Private registry
- High availability
- Image scanning
- IAM integration
- Lifecycle policies
- Cross-account support

---

# Image Tagging Strategy

Bad

```text
latest
```

Problem

Impossible to know which version is running.

---

Good

```text
payment-api

1.0.0

1.0.1

1.0.2
```

or

```text
payment-api

Git Commit SHA
```

Every deployment becomes traceable.

---

# Immutable Images

Production images should never change.

Example

Bad

```text
latest

↓

Overwrite
```

Good

```text
payment-api:1.4.7
```

Every deployment references a unique image.

---

# ECR Lifecycle Policies

Old images accumulate over time.

Lifecycle policies automatically remove unused images.

Example

```text
Keep

Latest 20 Images

↓

Delete Older Images
```

Benefits

- Lower storage cost
- Cleaner repository
- Easier management

---

# Image Scanning

Amazon ECR supports vulnerability scanning.

Workflow

```text
Push Image

↓

Amazon ECR

↓

Scan

↓

Critical Vulnerability?

↓

Fail Pipeline
```

Security should be integrated into CI/CD.

---

# Multi-Stage Docker Builds

Production Dockerfiles should use multi-stage builds.

Example

```text
Build Stage

↓

Compile Application

↓

Runtime Stage

↓

Small Production Image
```

Benefits

- Smaller image size
- Faster downloads
- Better security
- Reduced attack surface

---

# Image Optimization

Avoid

```text
Ubuntu

↓

2 GB Image
```

Prefer

```text
Minimal Runtime Image

↓

200 MB
```

Smaller images improve deployment speed.

---

# Deployment Workflow

```text
GitHub

↓

CI/CD Pipeline

↓

Amazon ECR

↓

New Image

↓

Task Definition Revision

↓

ECS Service Update

↓

Rolling Deployment

↓

Traffic Shift

↓

Deployment Complete
```

---

# Task Definition Revision Update

Every deployment creates

```text
Task Definition

Revision 18

↓

Revision 19
```

Old revisions remain available.

Rollback becomes simple.

---

# Rolling Deployment Process

```text
Old Tasks

↓

Launch New Tasks

↓

Health Checks

↓

Register with ALB

↓

Traffic Shift

↓

Old Tasks Removed
```

No service interruption.

---

# Rollback

Suppose

Version

```
2.1.0
```

contains a bug.

Rollback

```text
Revision 21

↓

Revision 20

↓

Deployment Complete
```

Rollback usually takes only a few minutes.

---

# Blue-Green Deployment with CodeDeploy

Architecture

```text
Application Load Balancer

↓

Blue Target Group

↓

Version 1

Green Target Group

↓

Version 2
```

Traffic shifts only after validation.

---

# Canary Deployment

Traffic Distribution

```text
95%

↓

Version 1

5%

↓

Version 2
```

If healthy

```text
20%

↓

50%

↓

100%
```

Risk remains low.

---

# Deployment Pipeline Example

```text
GitHub

↓

GitHub Actions

↓

Unit Tests

↓

SonarQube

↓

Trivy

↓

Docker Build

↓

Amazon ECR

↓

ECS Update Service

↓

AWS Fargate

↓

CloudWatch Verification
```

Deployment is completely automated.

---

# CI/CD Best Practices

- Use immutable image tags.
- Scan every image.
- Enable automatic testing.
- Keep Docker images small.
- Use Task Definition revisions.
- Deploy using rolling or Blue-Green strategy.
- Automate rollback.
- Never deploy manually in production.

---

# Enterprise Production Architecture

```text
Developer

↓

GitHub

↓

GitHub Actions

↓

SonarQube

↓

Trivy

↓

Docker Build

↓

Amazon ECR

↓

Amazon ECS

↓

AWS Fargate

↓

Application Load Balancer

↓

CloudWatch

↓

Amazon SNS
```

This architecture provides

- Continuous Integration
- Continuous Delivery
- Security Scanning
- Automated Deployments
- Central Monitoring
- Automatic Rollback

---

# Common Production Problems

## Image Pull Failure

Symptoms

```text
CannotPullContainerError
```

Possible causes

- Wrong image tag
- ECR permissions
- Missing Execution Role
- Deleted image
- Network connectivity issues

---

## Deployment Stuck

Possible causes

- Health check failures
- Insufficient CPU or memory
- Incorrect Task Definition
- Security Group configuration
- Load Balancer target registration issues

---

## Rollback Triggered

Possible causes

- New application crashes
- Health checks fail
- Container exits immediately
- Startup timeout
- Missing environment variables

---

## Slow Deployment

Check

- Image size
- ECR download time
- Health check interval
- ALB deregistration delay
- Container startup time

---

# Production Case Study

An enterprise manages

- 180 microservices
- 45 development teams
- 12 deployments per day

Pipeline

```text
GitHub

↓

GitHub Actions

↓

Quality Checks

↓

Security Scan

↓

Amazon ECR

↓

Amazon ECS

↓

AWS Fargate

↓

Rolling Deployment

↓

CloudWatch

↓

Production
```

Average deployment time

- Build: 3 minutes
- Security Scan: 2 minutes
- Deployment: 4 minutes

Entire deployment completes in under 10 minutes without downtime.

---

# Best Practices

- Version every image.
- Never use `latest` in production.
- Scan images before deployment.
- Automate deployments.
- Keep Task Definitions immutable.
- Monitor deployments.
- Enable automatic rollback.
- Store Dockerfiles in version control.

---

# Common Mistakes

- Deploying unscanned images.
- Using mutable image tags.
- Manual production deployments.
- Skipping rollback testing.
- Ignoring failed health checks.
- Large Docker images increasing deployment time.

---

# Interview Questions

## Basic

- What is Amazon ECR?
- Why should we avoid the `latest` tag?
- What is a Task Definition revision?

## Intermediate

- Explain a complete CI/CD pipeline for ECS Fargate.
- Rolling Deployment vs Blue-Green Deployment.
- How does ECS deploy a new container version?

## Advanced

- Design an enterprise CI/CD pipeline for 300 microservices running on AWS Fargate.
- Explain how you would integrate SonarQube, Trivy, Amazon ECR, ECS, and CloudWatch into a secure deployment pipeline.
- Design a zero-downtime deployment strategy with automatic rollback for a mission-critical payment application.

---

# Chapter 11 - Performance Optimization, Cost Optimization & Troubleshooting

Running applications successfully is only the beginning.

Enterprise DevOps teams continuously optimize

- Performance
- Cost
- Availability
- Reliability
- Resource Utilization

A well-designed Fargate platform should provide maximum performance with minimum operational cost.

---

# Performance Optimization

Performance depends on multiple factors.

```text
Application

↓

CPU

↓

Memory

↓

Networking

↓

Storage

↓

Container Image

↓

Database
```

Optimizing only one layer rarely solves production issues.

---

# CPU Optimization

Choosing the correct CPU allocation is critical.

Example

```text
Application

↓

0.25 vCPU

↓

CPU reaches 100%

↓

Slow Response
```

Increasing CPU

```text
Application

↓

1 vCPU

↓

CPU 45%

↓

Better Performance
```

Always monitor actual CPU utilization before changing task sizes.

---

# Memory Optimization

Memory shortages are one of the most common causes of container failures.

Example

```text
Application

↓

Memory Limit

↓

Exceeded

↓

OOM Kill

↓

Task Restart
```

Symptoms

- Random container restarts
- Increased latency
- Failed requests
- ECS replacing tasks frequently

---

# CPU vs Memory Bottleneck

CPU Bottleneck

```text
CPU

95%

Memory

40%
```

Increase CPU.

---

Memory Bottleneck

```text
CPU

35%

Memory

98%
```

Increase memory.

Understanding which resource is exhausted prevents unnecessary cost increases.

---

# Container Image Optimization

Large images increase deployment time.

Example

Bad

```text
Docker Image

↓

2.8 GB
```

Problems

- Slow image download
- Longer deployments
- Higher startup time

---

Better

```text
Docker Image

↓

220 MB
```

Benefits

- Faster deployments
- Faster scaling
- Reduced network usage

---

# Reduce Image Size

Best practices

- Use multi-stage builds.
- Remove unnecessary packages.
- Clean package caches.
- Avoid development tools in runtime images.
- Use lightweight base images appropriate for your workload.

---

# Startup Time Optimization

A slow startup delays deployments.

Workflow

```text
Task Launch

↓

Image Download

↓

Application Startup

↓

Health Check

↓

Ready
```

Reduce startup time by

- Smaller images
- Faster application initialization
- Lazy loading where appropriate
- Efficient dependency management

---

# Database Connection Pooling

Bad

```text
Every Request

↓

New Database Connection
```

High overhead.

---

Better

```text
Application

↓

Connection Pool

↓

Database
```

Benefits

- Lower latency
- Better throughput
- Fewer database connections

---

# Caching

Instead of repeatedly querying the database,

cache frequently accessed data.

```text
Application

↓

Amazon ElastiCache

↓

Database
```

Benefits

- Faster response
- Lower database load
- Better scalability

---

# Networking Optimization

Use

- Private Subnets
- VPC Endpoints
- Security Groups
- Internal ALBs where appropriate

Avoid unnecessary internet routing.

---

# Auto Scaling Optimization

Incorrect

```text
Minimum Tasks

10

↓

Low Traffic
```

High cost.

Better

```text
Minimum

2

↓

Scale Automatically
```

---

# Cost Optimization

Fargate pricing is based primarily on

- vCPU
- Memory
- Operating System
- Architecture
- Running Time

Unlike EC2,

you pay only for the resources allocated to running tasks.

---

# Cost Factors

```text
Task

↓

CPU

↓

Memory

↓

Running Time

↓

Monthly Cost
```

Oversized tasks significantly increase cost.

---

# Right Sizing

Bad

```text
Application

Uses

0.3 CPU

↓

Allocated

4 vCPU
```

Large waste.

---

Better

```text
Uses

0.4 CPU

↓

Allocate

0.5 vCPU
```

Monitor before resizing.

---

# Fargate Spot

Background workloads can reduce costs.

Example

```text
Batch Processing

↓

Fargate Spot
```

Production APIs

↓

Standard Fargate

Mixing both provides good cost efficiency.

---

# Schedule Non-Production Workloads

Development environments often run 24×7 unnecessarily.

Instead

```text
Office Hours

↓

Start Tasks

Night

↓

Stop Tasks
```

Reduces monthly cost.

---

# CloudWatch Cost Monitoring

Monitor

- CPU
- Memory
- Running Tasks
- Scaling Events
- Idle Capacity

Optimization should be data-driven.

---

# Common Performance Issues

## High CPU

Symptoms

- Slow responses
- High latency
- Scaling events

Check

- CPU metrics
- Infinite loops
- Application profiling
- Thread utilization

---

## High Memory

Symptoms

- OOMKilled
- Task restarts
- Application crashes

Check

- Memory leaks
- JVM heap
- Cache size
- Large objects

---

## Slow Startup

Possible causes

- Large Docker image
- Heavy initialization
- Database migration during startup
- Dependency downloads

---

## Image Pull Delay

Check

- Image size
- ECR connectivity
- Execution Role
- Network latency

---

## ALB Health Check Failure

Possible causes

- Wrong health endpoint
- Slow startup
- Port mismatch
- Security Groups
- Application crash

---

## Cannot Access AWS Services

Verify

- Task Role
- IAM Policy
- Security Groups
- VPC Endpoints
- Network ACLs

---

## Cannot Pull Secrets

Check

- Execution Role
- Task Role
- Secrets Manager permissions
- Secret ARN
- Region

---

## Task Stops Immediately

Possible causes

- Application crash
- Missing environment variables
- Invalid command
- Dependency failure
- Memory exhaustion

---

## ECS Service Never Becomes Stable

Verify

- Desired Count
- Health Checks
- Load Balancer
- Container Logs
- Task Definition
- CPU and Memory
- Networking

---

# Production Incident Example

A payment API experiences

```text
CPU

95%

↓

Response Time

8 Seconds

↓

Customers Report Slowness
```

Investigation

```text
CloudWatch

↓

CPU High

↓

Application Profile

↓

Database Queries Slow

↓

Added ElastiCache

↓

CPU Reduced

↓

Latency Improved
```

Root cause analysis is more important than simply increasing CPU.

---

# Enterprise Cost Optimization Strategy

```text
Production

↓

Standard Fargate

↓

Auto Scaling

↓

CloudWatch

↓

Performance Monitoring

Development

↓

Scheduled Shutdown

↓

Fargate Spot

↓

Lower Cost
```

This combination balances availability and operational cost.

---

# Production Architecture

```text
Users

↓

Application Load Balancer

↓

Amazon ECS Service

↓

AWS Fargate

↓

CloudWatch

↓

Container Insights

↓

Amazon RDS

↓

Amazon ElastiCache

↓

Amazon EFS

↓

Amazon S3
```

Characteristics

- Multi-AZ
- Auto Scaling
- Self Healing
- Optimized Resources
- Central Monitoring
- Cost Efficient

---

# Best Practices

- Monitor before resizing tasks.
- Keep Docker images small.
- Use Auto Scaling.
- Use Fargate Spot for interruptible workloads.
- Optimize database queries.
- Enable Container Insights.
- Configure CloudWatch Alarms.
- Profile applications regularly.
- Review resource utilization monthly.

---

# Common Mistakes

- Allocating excessive CPU and memory.
- Running development environments continuously.
- Ignoring CloudWatch metrics.
- Using Fargate Spot for critical production APIs.
- Deploying large container images.
- Scaling without identifying the bottleneck.

---

# Interview Questions

## Basic

- How is Fargate priced?
- What causes OOMKilled?
- Why should Docker images be small?

## Intermediate

- How would you optimize Fargate costs?
- CPU bottleneck vs Memory bottleneck.
- Explain Fargate Spot usage.
- How do you troubleshoot slow container startup?

## Advanced

- Design a cost-optimized Fargate platform for 500 microservices.
- Explain your troubleshooting process when an ECS Service cannot reach a stable state.
- A payment API running on Fargate suddenly experiences high latency and frequent task restarts. Explain your step-by-step investigation and resolution process.

---

# Quick Revision Cheat Sheet

| Requirement | AWS Service / Feature |
|-------------|----------------------|
| Serverless Containers | AWS Fargate |
| Container Orchestration | Amazon ECS / Amazon EKS |
| Container Registry | Amazon ECR |
| Application Permissions | IAM Task Role |
| Infrastructure Permissions | Execution Role |
| Persistent Shared Storage | Amazon EFS |
| Object Storage | Amazon S3 |
| Managed Database | Amazon RDS |
| In-Memory Cache | Amazon ElastiCache |
| Logging | CloudWatch Logs |
| Metrics | CloudWatch Metrics |
| Advanced Monitoring | Container Insights |
| Service Discovery | AWS Cloud Map |
| Load Balancing | Application Load Balancer |
| Auto Scaling | ECS Service Auto Scaling |
| Low-Cost Compute | Fargate Spot |
| Secret Management | AWS Secrets Manager |
| Configuration Management | Systems Manager Parameter Store |
| Zero-Downtime Deployment | Rolling / Blue-Green |
| Vulnerability Scanning | Amazon ECR Image Scanning |
| CI/CD | GitHub Actions / Jenkins / CodePipeline |