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

