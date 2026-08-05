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

