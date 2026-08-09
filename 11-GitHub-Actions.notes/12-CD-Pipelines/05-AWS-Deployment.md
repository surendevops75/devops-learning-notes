# AWS Deployment

AWS Deployment means deploying and running applications on Amazon Web Services using AWS infrastructure and services.

A typical DevOps deployment can use:

    GitHub
       |
       ↓
    CI Pipeline
       |
       ↓
    Docker Image
       |
       ↓
    Amazon ECR
       |
       ↓
    Amazon EKS
       |
       ↓
    Kubernetes
       |
       ↓
    Application

AWS provides services for:

    Compute
    Networking
    Storage
    Databases
    Containers
    Security
    Load Balancing
    DNS
    Monitoring

---

# AWS Deployment Architecture

A typical production architecture can look like:

    Internet
       |
       ↓
    Route 53
       |
       ↓
    Application Load Balancer
       |
       ↓
    EKS
       |
       +-- User Service
       +-- Product Service
       +-- Cart Service
       +-- Orders Service
       +-- Payment Service
       +-- Inventory Service
       +-- Notification Service
       |
       ↓
    RDS / Other Databases

CI/CD:

    Developer
       |
       ↓
    GitHub
       |
       ↓
    GitHub Actions / Jenkins
       |
       ↓
    Docker Build
       |
       ↓
    Security Scan
       |
       ↓
    Amazon ECR
       |
       ↓
    ArgoCD
       |
       ↓
    Amazon EKS

---

# AWS Deployment Services

Important AWS services used in DevOps deployments include:

    EC2
    VPC
    IAM
    S3
    ECR
    EKS
    ALB
    RDS
    Route 53
    NAT Gateway
    ACM

These services can be combined to build a production-grade application platform.

---

# AWS VPC

VPC stands for Virtual Private Cloud.

A VPC provides an isolated networking environment for AWS resources.

Typical architecture:

    AWS Account
        |
        ↓
       VPC
        |
        +-- Public Subnets
        |
        +-- Private Subnets
        |
        +-- Route Tables
        |
        +-- Internet Gateway
        |
        +-- NAT Gateway
        |
        +-- Security Groups

---

# Public and Private Subnets

A common production architecture separates resources into public and private subnets.

Public subnet:

    Internet Gateway
          |
          ↓
    Public Subnet
          |
          ↓
    ALB

Private subnet:

    Private Subnet
          |
          +-- EKS Nodes
          +-- Application Resources
          +-- Internal Services

The application workloads are generally kept private while the load balancer handles external traffic.

---

# Internet Gateway

An Internet Gateway provides connectivity between a VPC and the internet.

Example:

    Internet
       |
       ↓
    Internet Gateway
       |
       ↓
    Public Subnet

Public resources can use the Internet Gateway when their route tables and security controls allow it.

---

# NAT Gateway

A NAT Gateway allows resources in private subnets to initiate outbound internet connections without being directly accessible from the internet.

Example:

    Private Subnet
        |
        ↓
    NAT Gateway
        |
        ↓
    Internet Gateway
        |
        ↓
    Internet

Typical use case:

    Private EKS Node
        |
        ↓
    Download Package / Pull Dependency
        |
        ↓
    NAT Gateway
        |
        ↓
    Internet

---

# Route Tables

Route tables control how network traffic is routed.

Public subnet:

    0.0.0.0/0
        |
        ↓
    Internet Gateway

Private subnet:

    0.0.0.0/0
        |
        ↓
    NAT Gateway

Correct route-table configuration is important for AWS deployments.

---

# Security Groups

Security Groups act as virtual firewalls for AWS resources.

Example:

    ALB Security Group
        |
        +-- HTTP 80
        +-- HTTPS 443

    EKS Security Group
        |
        +-- Traffic from ALB
        +-- Required cluster traffic

Use the principle of least privilege.

Avoid:

    0.0.0.0/0
    All Ports

unless there is a specific and justified requirement.

---

# IAM

IAM stands for Identity and Access Management.

IAM controls:

    Users
    Roles
    Policies
    Permissions

In DevOps, IAM is commonly used for:

    CI/CD Access
    ECR Access
    EKS Access
    S3 Access
    Terraform
    AWS Services

---

# IAM Roles

Prefer IAM roles instead of long-lived access keys whenever possible.

Example:

    GitHub Actions
         |
         ↓
    IAM Role
         |
         ↓
    AWS Resources

This improves security.

---

# IAM Least Privilege

Grant only the permissions required by the workload.

Bad:

    AdministratorAccess

Better:

    Required ECR Permissions
    Required EKS Permissions
    Required S3 Permissions

Least privilege reduces the impact of compromised credentials.

---

# Amazon EC2

Amazon EC2 provides virtual machines in AWS.

Common DevOps uses:

    Jenkins Server
    Bastion Host
    Self-Hosted Runner
    Application Server
    Utility Server

Example:

    VPC
      |
      ↓
    Private Subnet
      |
      ↓
    EC2
      |
      ↓
    Application

---

# EC2 Deployment

Traditional application deployment:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    CI Pipeline
        |
        ↓
    Build Artifact
        |
        ↓
    EC2
        |
        ↓
    Application

Configuration can be automated using:

    Ansible
    Terraform
    User Data
    Shell Scripts
    CI/CD Pipelines

---

# EC2 With Ansible

Example:

    GitHub
       |
       ↓
    CI Pipeline
       |
       ↓
    Ansible
       |
       ↓
    EC2
       |
       ↓
    Application

Ansible can automate:

    Package Installation
    Configuration
    Service Management
    Application Deployment

---

# Amazon ECR

ECR stands for Elastic Container Registry.

ECR stores Docker container images.

Flow:

    Developer
       |
       ↓
    GitHub
       |
       ↓
    CI Pipeline
       |
       ↓
    Docker Build
       |
       ↓
    Amazon ECR
       |
       ↓
    EKS

---

# ECR Repository

Example:

    ECR
      |
      └── payment
           |
           ├── 1.0.0
           ├── 1.0.1
           ├── 1.0.2
           └── 1.0.3

Use immutable versioning where possible.

---

# Docker Build and ECR

Example:

    docker build -t payment:1.0.0 .

Login to ECR:

    aws ecr get-login-password --region <region> \
      | docker login \
      --username AWS \
      --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com

Tag image:

    docker tag payment:1.0.0 \
      <account-id>.dkr.ecr.<region>.amazonaws.com/payment:1.0.0

Push:

    docker push \
      <account-id>.dkr.ecr.<region>.amazonaws.com/payment:1.0.0

---

# Amazon EKS

EKS stands for Elastic Kubernetes Service.

EKS is AWS's managed Kubernetes service.

Architecture:

    AWS
      |
      ↓
    EKS Cluster
      |
      +-- Control Plane
      |
      +-- Worker Nodes
      |
      +-- Kubernetes Workloads

AWS manages the Kubernetes control plane components while customers manage workloads and cluster configuration according to the selected EKS architecture.

---

# EKS Architecture

Simplified:

    Developer
       |
       ↓
    GitOps
       |
       ↓
    ArgoCD
       |
       ↓
    EKS
       |
       +-- Kubernetes API
       |
       +-- Worker Nodes
              |
              +-- Pods
              +-- Services
              +-- Ingress

---

# EKS Worker Nodes

Worker nodes run Kubernetes workloads.

Example:

    EKS
      |
      ├── Node 1
      |     |
      |     ├── Pod
      |     └── Pod
      |
      ├── Node 2
      |     |
      |     ├── Pod
      |     └── Pod
      |
      └── Node 3
            |
            ├── Pod
            └── Pod

Nodes can be managed using:

    Managed Node Groups
    Self-Managed Nodes
    Other supported compute options

---

# EKS Managed Node Groups

Managed Node Groups simplify worker-node lifecycle management.

AWS can help manage:

    Node Provisioning
    Node Updates
    Scaling
    Lifecycle Operations

Application workloads still run as Kubernetes Pods.

---

# EKS Networking

EKS networking commonly integrates with the AWS VPC.

Example:

    VPC
      |
      ├── Public Subnets
      |
      └── Private Subnets
             |
             └── EKS Nodes
                    |
                    └── Pods

AWS networking and Kubernetes networking work together to provide connectivity.

---

# EKS Private Subnets

A common production architecture places worker nodes in private subnets.

Example:

    Internet
       |
       ↓
    ALB
       |
       ↓
    Private Subnets
       |
       ↓
    EKS Nodes
       |
       ↓
    Pods

This reduces direct exposure of worker nodes.

---

# Application Load Balancer

ALB stands for Application Load Balancer.

ALB operates at Layer 7 and supports HTTP and HTTPS traffic.

Typical flow:

    User
      |
      ↓
    Route 53
      |
      ↓
    ALB
      |
      ↓
    Kubernetes Ingress
      |
      ↓
    Service
      |
      ↓
    Pods

---

# ALB Ingress

In an EKS environment, Kubernetes Ingress can be integrated with AWS Application Load Balancers.

Example:

    Internet
       |
       ↓
    ALB
       |
       ↓
    Ingress
       |
       ↓
    Service
       |
       ↓
    Pods

The AWS Load Balancer Controller is commonly used to provision and manage AWS load balancer resources from Kubernetes configuration.

---

# AWS Load Balancer Controller

The AWS Load Balancer Controller manages AWS load balancers for Kubernetes workloads.

It can create and configure:

    Application Load Balancer
    Network Load Balancer

based on Kubernetes resources and configuration.

Example:

    Kubernetes Ingress
          |
          ↓
    AWS Load Balancer Controller
          |
          ↓
    Application Load Balancer
          |
          ↓
    Kubernetes Service
          |
          ↓
    Pods

---

# Route 53

Route 53 is AWS's DNS service.

Example:

    app.example.com
          |
          ↓
       Route 53
          |
          ↓
          ALB
          |
          ↓
         EKS

Route 53 provides DNS resolution for the application.

---

# DNS Deployment Flow

    User
      |
      ↓
    app.example.com
      |
      ↓
    Route 53
      |
      ↓
    ALB
      |
      ↓
    EKS
      |
      ↓
    Application

---

# ACM

ACM stands for AWS Certificate Manager.

It can be used to manage TLS certificates.

Typical HTTPS architecture:

    User
      |
      ↓
    HTTPS
      |
      ↓
    ALB
      |
      ↓
    ACM Certificate
      |
      ↓
    Kubernetes
      |
      ↓
    Application

TLS can be terminated at the ALB depending on the architecture.

---

# HTTPS Deployment

Typical flow:

    User
      |
      ↓
    HTTPS :443
      |
      ↓
    ALB
      |
      ↓
    TLS Termination
      |
      ↓
    Kubernetes Service
      |
      ↓
    Pods

---

# Amazon RDS

RDS provides managed relational databases.

Examples include supported database engines such as:

    PostgreSQL
    MySQL
    MariaDB
    Oracle
    SQL Server

Typical application architecture:

    EKS
      |
      ↓
    Application
      |
      ↓
    RDS
      |
      ↓
    Database

---

# RDS Private Deployment

For production applications, RDS is commonly placed in private subnets.

Example:

    VPC
      |
      +-- Public Subnets
      |
      +-- Private Application Subnets
      |
      └-- Private Database Subnets
              |
              └── RDS

The application communicates with RDS through controlled network access.

---

# AWS Deployment With Terraform

Terraform can provision AWS infrastructure.

Example:

    Terraform
       |
       +-- VPC
       +-- Subnets
       +-- Security Groups
       +-- IAM
       +-- ECR
       +-- EKS
       +-- ALB
       +-- RDS
       +-- S3
       |
       ↓
    AWS

---

# Terraform AWS Deployment Flow

    GitHub
       |
       ↓
    Terraform Code
       |
       ↓
    CI Pipeline
       |
       +-- terraform fmt
       +-- terraform validate
       +-- terraform plan
       +-- terraform apply
       |
       ↓
    AWS Infrastructure

---

# Terraform State

Terraform maintains state to track infrastructure.

A production environment commonly uses remote state.

Example:

    Terraform
       |
       ↓
    S3 Backend
       |
       ↓
    Terraform State

Remote state provides centralized state management for teams.

---

# AWS Deployment With GitOps

Infrastructure and application deployment can be separated.

Infrastructure:

    Terraform
       |
       ↓
    AWS
       |
       +-- VPC
       +-- EKS
       +-- ECR
       +-- ALB
       +-- RDS

Application:

    GitHub
       |
       ↓
    GitOps Repository
       |
       ↓
    ArgoCD
       |
       ↓
    EKS

---

# Complete AWS DevOps Architecture

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Test
        +-- SonarQube
        +-- Trivy
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Kubernetes
        |
        ↓
    ALB
        |
        ↓
    Route 53
        |
        ↓
    Users

Infrastructure:

    Terraform
        |
        ↓
    AWS
        |
        +-- VPC
        +-- Subnets
        +-- IAM
        +-- EKS
        +-- ECR
        +-- ALB
        +-- RDS

---

# AWS Environment Structure

A company may have:

    Development
        |
        ↓
    QA
        |
        ↓
    UAT
        |
        ↓
    Production

Each environment can have separate:

    AWS Resources
    Kubernetes Namespaces
    EKS Clusters
    Databases
    GitOps Configuration

The exact design depends on organizational requirements.

---

# Development Environment

Example:

    Developer
       |
       ↓
    GitHub
       |
       ↓
    CI
       |
       ↓
    ECR
       |
       ↓
    DEV EKS
       |
       ↓
    Application

Development environments are generally optimized for rapid testing.

---

# QA Environment

Example:

    Application
       |
       ↓
    CI
       |
       ↓
    ECR
       |
       ↓
    QA GitOps
       |
       ↓
    ArgoCD
       |
       ↓
    QA EKS
       |
       ↓
    Testing

---

# UAT Environment

UAT stands for User Acceptance Testing.

Typical flow:

    QA
      |
      ↓
    UAT Approval
      |
      ↓
    UAT GitOps
      |
      ↓
    ArgoCD
      |
      ↓
    UAT EKS
      |
      ↓
    User Acceptance Testing

---

# Production Environment

Typical production flow:

    UAT
      |
      ↓
    Approval
      |
      ↓
    Production GitOps
      |
      ↓
    ArgoCD
      |
      ↓
    Production EKS
      |
      ↓
    ALB
      |
      ↓
    Users

Production should have stronger controls and approval processes.

---

# AWS Deployment Strategy

A typical deployment sequence:

    1. Developer commits code
    2. Pull Request created
    3. CI pipeline starts
    4. Application is built
    5. Tests execute
    6. Code quality is checked
    7. Security scanning runs
    8. Docker image is built
    9. Image is pushed to ECR
    10. GitOps configuration is updated
    11. Pull Request is reviewed
    12. GitOps change is merged
    13. ArgoCD detects the change
    14. ArgoCD synchronizes EKS
    15. Kubernetes creates new Pods
    16. Health checks run
    17. Application becomes available

---

# AWS Deployment Using GitHub Actions

Example pipeline:

    name: Application Deployment

    on:
      push:
        branches:
          - main

    jobs:

      build:
        runs-on: ubuntu-latest

        steps:
          - name: Checkout
            uses: actions/checkout@v4

          - name: Build
            run: |
              mvn clean package

          - name: Docker Build
            run: |
              docker build -t myapp:${{ github.sha }} .

          - name: Push to ECR
            run: |
              docker push <ECR_REPOSITORY>:${{ github.sha }}

The exact workflow depends on the application and AWS authentication design.

---

# AWS Authentication From GitHub Actions

Prefer short-lived credentials through an appropriate AWS identity federation mechanism rather than storing long-lived AWS access keys.

Typical architecture:

    GitHub Actions
         |
         ↓
    OIDC
         |
         ↓
    AWS IAM Role
         |
         ↓
    AWS Services

This reduces the need for long-lived static credentials.

---

# OIDC

OIDC stands for OpenID Connect.

GitHub Actions can use OIDC to authenticate to AWS.

Flow:

    GitHub Actions
         |
         ↓
    OIDC Token
         |
         ↓
    AWS STS
         |
         ↓
    Assume IAM Role
         |
         ↓
    Temporary Credentials
         |
         ↓
    AWS

---

# AWS STS

STS stands for Security Token Service.

It provides temporary security credentials.

Example:

    GitHub Actions
        |
        ↓
    AWS STS
        |
        ↓
    Temporary Credentials
        |
        ↓
    ECR / EKS / S3

Temporary credentials are preferred over long-lived access keys where supported.

---

# EKS Deployment With Helm

Example:

    GitHub Actions
        |
        ↓
    ECR
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    Helm
        |
        ↓
    EKS
        |
        ↓
    Pods

Helm manages Kubernetes application packaging while ArgoCD manages GitOps-based deployment.

---

# EKS Deployment With Kubernetes Manifests

Example:

    GitOps Repository
        |
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        └── configmap.yaml
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

ArgoCD applies the declarative configuration.

---

# EKS Deployment With ArgoCD

Flow:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    CI Pipeline
        |
        ↓
    ECR
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Application

---

# AWS Deployment Using Terraform and ArgoCD

Infrastructure:

    Terraform
       |
       ↓
    AWS
       |
       +-- VPC
       +-- EKS
       +-- ECR
       +-- ALB
       +-- IAM
       +-- RDS

Application:

    GitHub
       |
       ↓
    CI
       |
       ↓
    ECR
       |
       ↓
    GitOps
       |
       ↓
    ArgoCD
       |
       ↓
    EKS

---

# Terraform vs ArgoCD

Terraform:

    Infrastructure Provisioning

Examples:

    VPC
    Subnets
    EKS
    ECR
    IAM
    RDS
    Security Groups

ArgoCD:

    Kubernetes Application Delivery

Examples:

    Deployment
    Service
    Ingress
    ConfigMap
    HPA
    Application Configuration

---

# AWS Deployment Security

Important security practices:

- Use IAM roles
- Use least privilege
- Use OIDC for CI/CD where appropriate
- Avoid long-lived credentials
- Keep databases private
- Keep worker nodes private where appropriate
- Restrict security groups
- Encrypt sensitive data
- Use HTTPS
- Use ACM certificates
- Scan container images
- Protect Git repositories
- Use Kubernetes RBAC
- Protect production environments
- Store secrets securely

---

# ECR Security

Important practices:

    Image Scanning
    Immutable Tags
    Repository Policies
    Lifecycle Policies
    Least Privilege IAM

Avoid relying on:

    latest

Use versioned image tags.

---

# AWS Network Security

Typical security architecture:

    Internet
       |
       ↓
    ALB
       |
       ↓
    Private EKS
       |
       ↓
    Application
       |
       ↓
    Private RDS

Security Groups should allow only the required traffic.

---

# Database Security

For RDS:

- Use private subnets
- Restrict inbound access
- Use security groups
- Encrypt data
- Use appropriate credentials management
- Enable backups
- Use Multi-AZ where required
- Monitor database health
- Avoid public database access unless explicitly required

---

# AWS Deployment Observability

Application deployment should be monitored after release.

A DevOps observability stack can include:

    Prometheus
        |
        ↓
    Metrics

    Grafana
        |
        ↓
    Dashboards

    ELK
        |
        ↓
    Logs

Deployment flow:

    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Application
        |
        +-- Metrics → Prometheus
        |
        +-- Dashboards → Grafana
        |
        └-- Logs → ELK

---

# AWS Deployment Validation

After deployment:

    ArgoCD
       |
       ↓
    Sync Status
       |
       ↓
    Deployment Status
       |
       ↓
    Pod Status
       |
       ↓
    Service Status
       |
       ↓
    ALB Status
       |
       ↓
    Application Endpoint
       |
       ↓
    Smoke Test

---

# Kubernetes Deployment Validation

Commands:

    kubectl get pods

    kubectl get deployments

    kubectl get services

    kubectl get ingress

    kubectl get events

Check:

    Pods Ready
    Deployment Available
    Service Correct
    Ingress Correct
    Events Normal

---

# AWS Deployment Validation

AWS CLI can be used to inspect AWS resources.

Examples:

    aws eks describe-cluster --name <cluster-name>

    aws ecr describe-repositories

    aws ecr list-images --repository-name <repository>

    aws elbv2 describe-load-balancers

The exact command depends on the resource being inspected.

---

# Deployment Failure Scenario

Suppose a new version is deployed.

    Git
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    New Pods
      |
      ↓
    CrashLoopBackOff

Troubleshooting:

    kubectl get pods

    kubectl describe pod <pod>

    kubectl logs <pod>

    kubectl get events

Then identify the root cause.

Possible causes:

    Bad Configuration
    Missing Secret
    Incorrect Image
    Application Error
    Resource Limit
    Dependency Failure

---

# ImagePullBackOff Scenario

If Pods show:

    ImagePullBackOff

Check:

    kubectl describe pod <pod>

Look for:

    Image Name
    Image Tag
    Registry
    Authentication
    Pull Errors

For ECR, verify:

    ECR Repository
    Image Exists
    Image Tag
    IAM Permissions
    EKS Node / Pod Access

---

# CrashLoopBackOff Scenario

If the application enters:

    CrashLoopBackOff

Check:

    kubectl logs <pod>

For the previous container:

    kubectl logs <pod> --previous

Then:

    kubectl describe pod <pod>

Check:

    Environment Variables
    Secrets
    ConfigMaps
    Application Configuration
    Resource Limits
    Dependencies
    Startup Errors

---

# ALB Troubleshooting

If users cannot access the application:

Check:

    Route 53
        |
        ↓
    ALB
        |
        ↓
    Listener
        |
        ↓
    Target Group
        |
        ↓
    Kubernetes Service
        |
        ↓
    Pods

Possible issues:

    DNS
    Security Group
    Listener
    Target Health
    Ingress
    Service
    Pod Readiness
    Application Port

---

# ALB Target Health

If ALB targets are unhealthy:

Check:

    Target Group
        |
        ↓
    Health Check
        |
        ↓
    Service
        |
        ↓
    Pod

Verify:

    Health Check Path
    Port
    Protocol
    Security Group
    Readiness
    Application Availability

---

# Route 53 Troubleshooting

If DNS is not resolving:

Check:

    Route 53 Record
        |
        ↓
    DNS Name
        |
        ↓
    ALB DNS Name

Verify:

    Record Type
    Record Value
    Hosted Zone
    DNS Configuration

---

# AWS Deployment Rollback

If the deployment is unhealthy:

    New Version
        |
        ↓
    Production
        |
        ↓
    Problem
        |
        ↓
    Rollback
        |
        ↓
    Previous Version
        |
        ↓
    Health Validation

With GitOps:

    Revert Git Change
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Previous Version

---

# AWS Deployment Best Practices

- Use Infrastructure as Code
- Use Terraform for infrastructure
- Use GitOps for Kubernetes applications
- Use immutable Docker image tags
- Use ECR for container images
- Use EKS for managed Kubernetes
- Use private subnets for workloads where appropriate
- Use ALB for HTTP/HTTPS traffic
- Use Route 53 for DNS
- Use ACM for TLS certificates
- Use IAM roles
- Use OIDC for CI/CD authentication where appropriate
- Use least privilege
- Secure secrets
- Enable image scanning
- Use health checks
- Validate deployments
- Monitor applications
- Maintain rollback procedures
- Use separate environments
- Protect production deployments

---

# AWS Deployment Anti-Patterns

## Using Long-Lived Access Keys

Bad:

    GitHub Actions
        |
        ↓
    Static AWS Access Key
        |
        ↓
    AWS

Better:

    GitHub Actions
        |
        ↓
    OIDC
        |
        ↓
    IAM Role
        |
        ↓
    AWS

---

# AWS Deployment Anti-Pattern

## Public Database

Bad:

    Internet
       |
       ↓
    Public RDS
       |
       ↓
    Database

Better:

    Application
       |
       ↓
    Private RDS
       |
       ↓
    Database

---

# AWS Deployment Anti-Pattern

## Exposing Worker Nodes

Bad:

    Internet
       |
       ↓
    EKS Worker Node
       |
       ↓
    Application

Better:

    Internet
       |
       ↓
    ALB
       |
       ↓
    Private EKS
       |
       ↓
    Application

---

# AWS Deployment Anti-Pattern

## Using latest Tag

Bad:

    image:
      tag: latest

Better:

    image:
      tag: "1.4.7"

or:

    image:
      tag: "<commit-sha>"

---

# AWS Deployment Anti-Pattern

## Manual Infrastructure Changes

Bad:

    Engineer
       |
       ↓
    AWS Console
       |
       ↓
    Infrastructure

Better:

    Terraform
       |
       ↓
    Git
       |
       ↓
    CI/CD
       |
       ↓
    AWS

Infrastructure changes become version-controlled and repeatable.

---

# AWS Deployment Interview Questions

## Basic Questions

1. What AWS services have you used for DevOps deployments?
2. What is ECR?
3. What is EKS?
4. What is an ALB?
5. What is Route 53?
6. What is IAM?
7. What is a VPC?
8. What is a NAT Gateway?
9. What is an Internet Gateway?
10. What is a Security Group?
11. What is RDS?
12. What is ACM?
13. What is an EKS node?
14. What is a private subnet?
15. What is a public subnet?

---

# AWS Deployment Interview Questions

## Intermediate Questions

16. How would you deploy a Docker application to EKS?

17. How would you push Docker images to ECR?

18. How would you connect GitHub Actions to AWS?

19. How would you secure GitHub Actions AWS access?

20. What is OIDC and why would you use it?

21. How does ALB work with EKS?

22. How would you configure Route 53 for an application?

23. How would you configure HTTPS for an ALB?

24. How would you deploy RDS securely?

25. How would you troubleshoot an ImagePullBackOff on EKS?

26. How would you troubleshoot an unhealthy ALB target?

27. How would you troubleshoot DNS issues?

28. How would you use Terraform for AWS infrastructure?

29. What is the difference between Terraform and ArgoCD?

30. How would you implement CI/CD for EKS?

---

# AWS Deployment Interview Questions

## Advanced Questions

31. Design a production AWS architecture for a microservices application.

32. How would you design a highly available EKS deployment?

33. How would you design public and private subnets?

34. How would you secure an EKS production environment?

35. How would you implement GitHub Actions with OIDC and AWS?

36. How would you design ECR security?

37. How would you design an EKS deployment using ArgoCD?

38. How would you implement DEV, QA, UAT, and PROD environments?

39. How would you troubleshoot a production deployment failure?

40. How would you design rollback for EKS applications?

41. How would you secure RDS?

42. How would you design AWS networking for EKS?

43. How would you monitor applications deployed on EKS?

44. How would you design an end-to-end AWS DevOps pipeline?

---

# Real-World Scenario

Suppose a microservices application contains:

    User
    Product
    Cart
    Orders
    Payment
    Inventory
    Notification

AWS architecture:

    Internet
       |
       ↓
    Route 53
       |
       ↓
    ALB
       |
       ↓
    EKS
       |
       ├── User
       ├── Product
       ├── Cart
       ├── Orders
       ├── Payment
       ├── Inventory
       └── Notification
       |
       ↓
    RDS

CI/CD:

    Developer
       |
       ↓
    GitHub
       |
       ↓
    GitHub Actions
       |
       +-- Build
       +-- Test
       +-- SonarQube
       +-- Trivy
       |
       ↓
    ECR
       |
       ↓
    GitOps Repository
       |
       ↓
    ArgoCD
       |
       ↓
    EKS

Infrastructure:

    Terraform
       |
       ↓
    AWS
       |
       +-- VPC
       +-- Subnets
       +-- IAM
       +-- EKS
       +-- ECR
       +-- ALB
       +-- RDS

---

# Real-World AWS Deployment Flow

    Developer
        |
        ↓
    Application Code
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Test
        +-- SonarQube
        +-- Trivy
        |
        ↓
    Docker Image
        |
        ↓
    Amazon ECR
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    Amazon EKS
        |
        ↓
    Kubernetes
        |
        ↓
    ALB
        |
        ↓
    Route 53
        |
        ↓
    End Users

---

# Complete AWS DevOps Architecture

    ┌──────────────────────────────┐
    │          Developer           │
    └──────────────┬───────────────┘
                   |
                   ↓
    ┌──────────────────────────────┐
    │           GitHub             │
    └──────────────┬───────────────┘
                   |
                   ↓
    ┌──────────────────────────────┐
    │       GitHub Actions         │
    │                              │
    │ Build                        │
    │ Test                         │
    │ SonarQube                    │
    │ Trivy                        │
    └──────────────┬───────────────┘
                   |
                   ↓
    ┌──────────────────────────────┐
    │         Amazon ECR           │
    │       Docker Images          │
    └──────────────┬───────────────┘
                   |
                   ↓
    ┌──────────────────────────────┐
    │      GitOps Repository       │
    └──────────────┬───────────────┘
                   |
                   ↓
    ┌──────────────────────────────┐
    │           ArgoCD             │
    └──────────────┬───────────────┘
                   |
                   ↓
    ┌──────────────────────────────┐
    │           Amazon EKS         │
    │                              │
    │ User                         │
    │ Product                      │
    │ Cart                         │
    │ Orders                       │
    │ Payment                      │
    │ Inventory                    │
    │ Notification                 │
    └──────────────┬───────────────┘
                   |
                   ↓
    ┌──────────────────────────────┐
    │             ALB              │
    └──────────────┬───────────────┘
                   |
                   ↓
    ┌──────────────────────────────┐
    │          Route 53            │
    └──────────────┬───────────────┘
                   |
                   ↓
                Users

Infrastructure:

    Terraform
       |
       ↓
    AWS
       |
       ├── VPC
       ├── Subnets
       ├── Security Groups
       ├── IAM
       ├── ECR
       ├── EKS
       ├── ALB
       └── RDS

---

# Final AWS Deployment Mental Model

Remember:

    Terraform
        |
        ↓
    AWS Infrastructure
        |
        +-- VPC
        +-- Subnets
        +-- IAM
        +-- ECR
        +-- EKS
        +-- ALB
        +-- RDS

Application deployment:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Test
        +-- Security Scan
        |
        ↓
    ECR
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Kubernetes
        |
        ↓
    ALB
        |
        ↓
    Route 53
        |
        ↓
    Users

---

# Final Concept

A production AWS DevOps deployment separates infrastructure provisioning from application deployment.

Infrastructure:

    Terraform
        |
        ↓
    AWS
        |
        +-- VPC
        +-- EKS
        +-- ECR
        +-- IAM
        +-- ALB
        +-- RDS

Application:

    GitHub
        |
        ↓
    CI Pipeline
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Kubernetes
        |
        ↓
    Application

The complete AWS DevOps model is:

    Code
      |
      ↓
    CI
      |
      ↓
    Build
      |
      ↓
    Test
      |
      ↓
    Security Scan
      |
      ↓
    ECR
      |
      ↓
    GitOps
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    ALB
      |
      ↓
    Route 53
      |
      ↓
    Application

This provides a repeatable, secure, automated, and scalable deployment architecture for applications running on AWS.