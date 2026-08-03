# AWS App Runner

---

# Introduction

AWS App Runner is a fully managed application service that enables developers to deploy containerized applications and web services directly from source code or container images without managing infrastructure.

Traditionally, deploying applications requires provisioning servers, configuring load balancers, setting up auto scaling, managing networking, and handling operating system updates. AWS App Runner abstracts these operational tasks, allowing developers to focus solely on application development.

AWS App Runner integrates with

- Amazon ECR
- GitHub
- AWS CodeBuild
- AWS CodePipeline
- AWS IAM
- Amazon CloudWatch
- AWS X-Ray
- AWS Secrets Manager
- Amazon VPC
- AWS Certificate Manager (ACM)

It provides automatic deployments, built-in load balancing, HTTPS support, and automatic scaling.

---

# What is AWS App Runner?

AWS App Runner is a fully managed service for deploying web applications and APIs.

It helps organizations

- Deploy Web Applications
- Host REST APIs
- Run Containerized Applications
- Eliminate Infrastructure Management
- Scale Automatically

Workflow

```text
Source Code / Docker Image

↓

AWS App Runner

↓

Build

↓

Deploy

↓

HTTPS Endpoint

↓

Users
```

---

# Why AWS App Runner?

Without App Runner

```text
Application

↓

Provision EC2

↓

Configure Load Balancer

↓

Setup Auto Scaling

↓

Deploy Application
```

Problems

- Infrastructure Management
- Complex Networking
- Manual Scaling
- Deployment Overhead

With App Runner

```text
Source Code

↓

App Runner

↓

Automatic Deployment

↓

Scalable Web Service
```

---

# Real World Problem Statement

A startup develops

- Node.js APIs
- Python Applications
- Java Microservices
- Docker Containers

Requirements

- Fast Deployment
- HTTPS Support
- Automatic Scaling
- Minimal Infrastructure Management

AWS App Runner provides a fully managed deployment platform.

---

# Enterprise Architecture

```text
GitHub / Amazon ECR

          │

          ▼

AWS App Runner

          │

──────────┼───────────

│          │          │

Auto Scaling HTTPS Logging

          │

     Internet Users
```

---

# Core Components

AWS App Runner consists of

- Source Repository
- Container Image
- Build Service
- Runtime Service
- Auto Scaling
- Load Balancer
- HTTPS Endpoint
- VPC Connector

---

# Source Code Deployment

Applications can be deployed directly from source code.

Supported Sources

- GitHub

Workflow

```text
GitHub

↓

App Runner

↓

Build

↓

Deploy
```

Automatic deployments occur when new commits are pushed.

---

# Container Deployment

Applications can also be deployed from Amazon ECR.

Workflow

```text
Docker Image

↓

Amazon ECR

↓

App Runner

↓

Production
```

---

# Automatic Build

App Runner automatically builds applications.

Supported Languages

- Node.js
- Python
- Java
- .NET
- Go
- PHP
- Ruby

No build server management required.

---

# Automatic Scaling

App Runner automatically adjusts capacity.

Architecture

```text
Requests Increase

↓

App Runner

↓

Scale Out

↓

More Instances
```

Benefits

- High Availability
- Cost Optimization
- No Manual Scaling

---

# Load Balancing

App Runner includes a built-in load balancer.

Benefits

- Even Traffic Distribution
- High Availability
- Automatic Health Checks

No additional configuration required.

---

# HTTPS Support

Every App Runner service includes

- HTTPS Endpoint
- Managed TLS Certificate
- Secure Communication

AWS Certificate Manager manages certificates automatically.

---

# VPC Connector

VPC Connector enables secure access to private AWS resources.

Examples

- Amazon RDS
- Amazon ElastiCache
- Internal APIs
- Private Services

Workflow

```text
App Runner

↓

VPC Connector

↓

Private Resources
```

---

# Environment Variables

Applications can use environment variables.

Examples

- Database Endpoint
- API URL
- Region
- Feature Flags

Sensitive values should be stored in

- AWS Secrets Manager
- Systems Manager Parameter Store

---

# Logging and Monitoring

App Runner integrates with

- Amazon CloudWatch
- AWS X-Ray

Supports

- Application Logs
- Metrics
- Tracing

---

# Security

Security Features

- IAM Integration
- HTTPS by Default
- AWS WAF Support
- Secrets Manager
- CloudTrail Logging

---

# AWS CLI

Create Service

```bash
aws apprunner create-service
```

List Services

```bash
aws apprunner list-services
```

Describe Service

```bash
aws apprunner describe-service \
--service-arn <service-arn>
```

---

# Terraform

```hcl
resource "aws_apprunner_service" "web" {

  service_name = "my-web-service"

}
```

---

# CloudFormation

```yaml
Resources:

  AppRunnerService:

    Type: AWS::AppRunner::Service
```

---

# Python (Boto3)

```python
import boto3

apprunner = boto3.client("apprunner")

response = apprunner.list_services()

print(response)
```

---

# Enterprise Production Architecture

```text
          Developers

               │

      GitHub / Amazon ECR

               │

       AWS App Runner

               │

 ┌─────────────┼─────────────┐

 │             │             │

Auto Scaling HTTPS CloudWatch

               │

        Internet Users
```

---

# Best Practices

- Use Amazon ECR for container images
- Store secrets in AWS Secrets Manager
- Enable CloudWatch monitoring
- Use VPC Connectors for private resources
- Enable automatic deployments
- Follow least-privilege IAM policies
- Monitor application health
- Optimize container image size
- Configure health checks
- Enable HTTPS for all applications
- Use environment variables appropriately
- Review scaling settings regularly

---

# Common Mistakes

- Hardcoding secrets
- Large container images
- Ignoring application logs
- Overly permissive IAM roles
- No health checks
- Deploying without testing
- Ignoring auto-scaling configuration
- Using public databases unnecessarily
- No monitoring
- Missing VPC Connector for private resources

---

# Troubleshooting

## Deployment Failed

Check

- Source Repository
- Build Logs
- IAM Permissions
- Application Configuration

---

## Build Failed

Verify

- Runtime Version
- Dependencies
- Build Commands
- Source Code

---

## Cannot Connect to Database

Check

- VPC Connector
- Security Groups
- Route Tables
- Database Endpoint

---

## Application Unhealthy

Verify

- Health Check Endpoint
- Application Logs
- Environment Variables
- CloudWatch Metrics

---

## Slow Response Time

Check

- Auto Scaling Configuration
- Container Resources
- Database Performance
- Network Latency

---

# Interview Questions

## Basic

1. What is AWS App Runner?
2. Why use App Runner?
3. Which deployment sources are supported?
4. Does App Runner support Docker containers?
5. How does App Runner handle scaling?
6. Does App Runner provide HTTPS?
7. What is a VPC Connector?
8. Which monitoring services integrate with App Runner?
9. How are secrets managed?
10. How does App Runner differ from EC2?

---

## Intermediate

11. Explain App Runner architecture.
12. Explain automatic deployments.
13. Explain container deployment.
14. Explain VPC Connectors.
15. Explain App Runner scaling.
16. Explain CloudWatch integration.
17. Explain App Runner security.
18. Explain health checks.
19. Explain runtime environments.
20. Explain enterprise deployment strategies.

---

## Advanced

21. Design a production web application using AWS App Runner.
22. Explain App Runner vs ECS Fargate.
23. Explain App Runner vs Elastic Beanstalk.
24. Design secure App Runner deployments.
25. Explain App Runner networking.
26. Design high-availability web services.
27. Explain CI/CD integration with App Runner.
28. Explain operational best practices.
29. Design scalable API architecture.
30. Best practices for AWS App Runner.

---

# Production Scenarios

### Scenario 1

A startup wants to deploy a Node.js API without managing servers.

Why would AWS App Runner be a good choice?

---

### Scenario 2

Your application stores Docker images in Amazon ECR.

How does App Runner automatically deploy new container versions?

---

### Scenario 3

A web application hosted on App Runner needs to connect to a private Amazon RDS database.

How would you securely configure connectivity?

---

### Scenario 4

An application experiences a sudden increase in traffic during a product launch.

How does App Runner maintain application availability?

---

### Scenario 5

Your security team requires HTTPS, IAM integration, encrypted secrets, and audit logging.

Which App Runner features satisfy these requirements?

---

### Scenario 6

An enterprise wants a simple platform for deploying internal APIs while minimizing infrastructure management.

How does App Runner compare to managing EC2 instances or Kubernetes clusters?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Source Repository | Application Source Code |
| Amazon ECR | Container Images |
| App Runner Service | Managed Application Hosting |
| Auto Scaling | Automatic Capacity Management |
| Built-in Load Balancer | Traffic Distribution |
| HTTPS Endpoint | Secure Access |
| VPC Connector | Private Resource Access |
| CloudWatch | Monitoring |
| AWS X-Ray | Distributed Tracing |
| Secrets Manager | Secure Secret Storage |

---

# Summary

AWS App Runner is a fully managed application service that simplifies deploying web applications and APIs directly from source code or container images. By providing automatic builds, deployments, HTTPS endpoints, built-in load balancing, auto scaling, VPC connectivity, CloudWatch monitoring, and integrations with Amazon ECR, GitHub, Secrets Manager, and AWS X-Ray, App Runner enables developers to focus on building applications without managing underlying infrastructure.