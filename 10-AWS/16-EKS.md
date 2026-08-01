# Amazon Elastic Kubernetes Service (Amazon EKS)

---

# Introduction

Amazon Elastic Kubernetes Service (Amazon EKS) is a fully managed Kubernetes service that enables organizations to run Kubernetes clusters without managing the Kubernetes control plane.

AWS manages the Kubernetes master components while engineers focus on deploying, managing, and scaling containerized applications.

Amazon EKS combines the power of Kubernetes with the reliability, security, and scalability of AWS services.

It integrates seamlessly with:

- Amazon EC2
- AWS Fargate
- Amazon ECR
- IAM
- VPC
- Elastic Load Balancer
- Auto Scaling
- CloudWatch
- Route53
- AWS Load Balancer Controller
- EBS
- EFS
- Secrets Manager

Today, thousands of enterprise organizations run production workloads on Amazon EKS.

---

# What is Amazon EKS?

Amazon EKS is a managed Kubernetes service.

AWS manages:

- Kubernetes Control Plane
- API Server
- etcd
- Scheduler
- Controller Manager
- High Availability
- Control Plane Upgrades
- Security Patches

You manage:

- Applications
- Worker Nodes
- Pods
- Deployments
- Services
- Ingress
- Storage
- Networking Policies

---

# Why Amazon EKS?

Suppose a company wants to deploy 250 microservices.

Managing Kubernetes manually requires:

- Installing Kubernetes
- Managing Masters
- Configuring etcd
- High Availability
- Upgrades
- Backup
- Security
- Monitoring

Amazon EKS eliminates these operational tasks.

Instead of managing Kubernetes infrastructure,

Engineers deploy applications.

---

# Why Kubernetes?

Traditional deployment

```text
Application

↓

EC2
```

Problems

- Resource Waste
- Difficult Scaling
- Manual Recovery
- Downtime

Container Deployment

```text
Application

↓

Docker

↓

Kubernetes

↓

AWS
```

Benefits

- Auto Healing

- Auto Scaling

- Rolling Updates

- Service Discovery

- High Availability

---

# Real World Problem

A company owns

- Frontend

- Authentication

- Product Service

- Cart Service

- Order Service

- Payment Service

- Inventory

- Notification

- Analytics

Over 150 containers run in production.

Requirements

- Zero Downtime

- High Availability

- Automatic Scaling

- Rolling Updates

- Secure Networking

- Disaster Recovery

Amazon EKS fulfills these requirements.

---

# Enterprise Architecture

```text
                     Internet

                         │

                    Route53 DNS

                         │

                 AWS Load Balancer

                         │

                Amazon EKS Cluster

         ┌──────────────┼──────────────┐

         │              │              │

     Worker Node    Worker Node    Worker Node

         │              │              │

     Multiple Pods  Multiple Pods Multiple Pods

         │              │              │

         └──────────────┼──────────────┘

                 Amazon ECR

                       │

                 CloudWatch

                       │

               Amazon VPC
```

---

# What is Kubernetes?

Kubernetes is an open-source container orchestration platform.

Originally developed by Google.

Now maintained by the Cloud Native Computing Foundation (CNCF).

Responsibilities

- Scheduling Containers

- Scaling Applications

- Load Balancing

- Self Healing

- Rolling Updates

- Service Discovery

---

# Kubernetes Architecture

```text
                 Kubernetes Cluster

         ┌────────────────────────────┐

         │      Control Plane         │

         └────────────────────────────┘

                  │

      ┌───────────┼───────────┐

      │           │           │

 Worker-1    Worker-2    Worker-3
```

---

# What is a Cluster?

A Kubernetes Cluster consists of

- Control Plane

- Worker Nodes

Applications run on Worker Nodes.

---

# Control Plane

The Control Plane manages the cluster.

AWS manages these components automatically.

Components

- API Server

- Scheduler

- Controller Manager

- etcd

Production Recommendation

Never attempt to manage the Control Plane yourself in EKS.

AWS handles upgrades, backups, and availability.

---

# Kubernetes API Server

The API Server is the heart of Kubernetes.

Every request passes through it.

Example

```text
kubectl apply

↓

API Server

↓

Kubernetes
```

Responsibilities

- Authentication

- Authorization

- Validation

- Resource Management

---

# etcd

etcd is the Kubernetes database.

Stores

- Pods

- Deployments

- Secrets

- ConfigMaps

- Services

- Cluster State

AWS manages etcd automatically.

Users cannot access it directly.

---

# Scheduler

The Scheduler decides where Pods should run.

It considers

- CPU

- Memory

- Node Labels

- Affinity Rules

- Taints

- Resource Availability

Example

```text
Pod

↓

Scheduler

↓

Worker Node-2
```

---

# Controller Manager

The Controller Manager continuously compares

Desired State

vs

Current State

Example

Desired Pods

```
3
```

Current Pods

```
2
```

Controller

↓

Creates

```
1 New Pod
```

This is called

Declarative Infrastructure.

---

# Worker Nodes

Worker Nodes execute application workloads.

Each Worker Node contains

- kubelet

- kube-proxy

- Container Runtime

- Pods

---

# Managed Node Groups

AWS automatically manages Worker Nodes.

Benefits

- Automatic Updates

- Auto Scaling

- Easy Upgrades

- Integrated with ASG

Recommended for production.

---

# Self Managed Nodes

You manage EC2 instances yourself.

Advantages

- Full Control

- Custom AMIs

Disadvantages

- Manual Updates

- Manual Scaling

- Higher Operational Overhead

---

# Fargate Profiles

AWS Fargate allows Pods to run without EC2 instances.

Architecture

```text
Pod

↓

AWS Fargate

↓

AWS Infrastructure
```

Benefits

- No Worker Nodes

- No EC2 Management

- Serverless Kubernetes

---

# Kubernetes Objects

Common Kubernetes objects

- Pod

- Deployment

- ReplicaSet

- Service

- ConfigMap

- Secret

- Namespace

- Ingress

- PersistentVolume

---

# Pod

A Pod is the smallest deployable unit in Kubernetes.

Usually contains

One Container

Example

```text
Pod

↓

Nginx Container
```

---

# Multi Container Pod

One Pod can contain multiple containers.

Example

```text
Pod

├── Main Application

└── Sidecar
```

Sidecars are commonly used for

- Logging

- Monitoring

- Proxy

---

# Deployment

Deployments manage Pods.

Example

Desired

```
5 Pods
```

If one Pod crashes

Deployment

↓

Creates New Pod

---

# ReplicaSet

ReplicaSets maintain the desired number of Pods.

Desired

```
3
```

Current

```
2
```

ReplicaSet

↓

Creates

```
1 Pod
```

---

# Namespace

Namespaces logically separate workloads.

Example

```text
production

staging

development

monitoring
```

---

# Labels

Labels organize Kubernetes resources.

Example

```yaml
app: payment

environment: production

team: devops
```

---

# Selectors

Selectors identify resources using Labels.

Example

```yaml
app=payment
```

---

# Amazon VPC Integration

Amazon EKS integrates directly with Amazon VPC.

Each Worker Node belongs to a VPC.

Pods communicate through the VPC network.

Architecture

```text
Amazon VPC

↓

Worker Node

↓

Pod

↓

Application
```

---

# Amazon VPC CNI

Amazon EKS uses the AWS VPC CNI Plugin.

Unlike overlay networking,

Each Pod receives its own VPC IP Address.

Example

```text
Worker Node

↓

Pod

↓

10.0.10.55
```

Benefits

- Native Networking

- High Performance

- AWS Security Groups

- Direct VPC Communication

---

# Pod Networking

Traffic Flow

```text
Pod

↓

VPC ENI

↓

Worker Node

↓

Amazon VPC

↓

Another Pod
```

Every Pod communicates using private IP addresses.

---

# Elastic Network Interface (ENI)

Worker Nodes attach one or more ENIs.

Pods receive IPs from these ENIs.

Architecture

```text
EC2

↓

ENI

↓

Pod IPs
```

This is unique to the AWS VPC CNI plugin.

---

# Cluster Endpoint

Every EKS cluster exposes an API endpoint.

Options

- Public Endpoint

- Private Endpoint

- Public + Private

Production Recommendation

Use a private endpoint whenever possible and restrict public access using CIDR allow lists if public access is required.

---

# Authentication

Amazon EKS uses IAM for authentication.

Workflow

```text
IAM User

↓

AWS STS Token

↓

API Server

↓

Authentication
```

Unlike standard Kubernetes, EKS integrates directly with AWS IAM.

---

# Authorization

After authentication,

Kubernetes RBAC determines permissions.

Example

```text
Developer

↓

Read Pods

----------------

Admin

↓

Full Cluster Access
```

IAM authenticates.

RBAC authorizes.

This distinction is a very common interview topic.

---

# Request Flow

```text
kubectl apply

↓

IAM Authentication

↓

API Server

↓

Scheduler

↓

Worker Node

↓

Pod Created

↓

Application Running
```

---

# Summary

Amazon EKS is AWS's fully managed Kubernetes platform that removes the operational burden of managing Kubernetes control planes while providing seamless integration with AWS networking, security, monitoring, and container services. Understanding the Control Plane, Worker Nodes, Pods, Deployments, VPC CNI, IAM authentication, and RBAC authorization forms the foundation for operating production-grade Kubernetes clusters on AWS.

---

# IAM Roles for Service Accounts (IRSA)

---

# What is IRSA?

IAM Roles for Service Accounts (IRSA) allows Kubernetes Pods to securely access AWS services using IAM Roles.

Without IRSA, every Pod running on the same Worker Node inherits the permissions of the EC2 Node IAM Role.

Example

```text
Worker Node

↓

EC2 IAM Role

↓

All Pods
```

This violates the Principle of Least Privilege.

With IRSA

```text
Pod

↓

Service Account

↓

IAM Role

↓

AWS Service
```

Each application receives only the permissions it needs.

---

# Why IRSA?

Suppose three applications run on one Worker Node.

```text
Payment Service

↓

Amazon DynamoDB

----------------------

Notification Service

↓

Amazon SNS

----------------------

Image Service

↓

Amazon S3
```

Without IRSA

All three Pods share the same EC2 IAM Role.

This means every Pod can potentially access every AWS service allowed by that role.

With IRSA

```text
Payment Pod

↓

Payment IAM Role

↓

DynamoDB

--------------------

Image Pod

↓

Image IAM Role

↓

S3

--------------------

Notification Pod

↓

SNS IAM Role

↓

SNS
```

Each Pod receives only the required permissions.

---

# IRSA Architecture

```text
Application Pod

↓

Kubernetes Service Account

↓

OIDC Identity Provider

↓

AWS STS

↓

Temporary Credentials

↓

IAM Role

↓

AWS Services
```

---

# OIDC Provider

Amazon EKS creates an OpenID Connect (OIDC) identity provider.

Purpose

Authenticate Kubernetes Service Accounts with AWS IAM.

Workflow

```text
Service Account

↓

OIDC Token

↓

AWS STS

↓

Temporary IAM Credentials
```

---

# Service Accounts

Every Pod runs using a Service Account.

Default

```text
default
```

Production

Create dedicated Service Accounts.

Example

```yaml
payment-service-account

notification-service-account

image-service-account
```

---

# IRSA Authentication Flow

```text
Pod Starts

↓

Service Account Token

↓

OIDC Provider

↓

AWS STS AssumeRoleWithWebIdentity

↓

Temporary Credentials

↓

AWS API
```

No static AWS credentials are required.

---

# Benefits

- Least Privilege
- No Access Keys
- Temporary Credentials
- Better Security
- Easy Rotation
- Native AWS Integration

---

# Kubernetes Services

A Service provides stable networking for Pods.

Without Service

```text
Pod

↓

Deleted

↓

New IP
```

Applications lose connectivity.

With Service

```text
Application

↓

Service

↓

Pods
```

The Service IP remains stable.

---

# Service Types

Supported Services

- ClusterIP
- NodePort
- LoadBalancer
- ExternalName

---

# ClusterIP

Default Service type.

Accessible only inside the cluster.

Architecture

```text
Pod

↓

ClusterIP

↓

Pod
```

Used for:

- Internal APIs
- Backend Services

---

# NodePort

Exposes applications on every Worker Node.

```text
NodeIP:30080
```

Architecture

```text
Internet

↓

Worker Node

↓

NodePort

↓

Pod
```

Mostly used for testing.

---

# LoadBalancer

Creates an AWS Load Balancer automatically.

Architecture

```text
Internet

↓

AWS Load Balancer

↓

Service

↓

Pods
```

Recommended for production.

---

# ExternalName

Maps Kubernetes Services to external DNS names.

Example

```text
payment-db

↓

rds.amazonaws.com
```

---

# Ingress

Ingress manages HTTP and HTTPS routing.

Instead of creating multiple Load Balancers,

One ALB serves multiple applications.

Architecture

```text
Internet

↓

ALB

↓

Ingress

↓

Multiple Services

↓

Pods
```

---

# AWS Load Balancer Controller

The AWS Load Balancer Controller automatically creates AWS ALBs and NLBs.

Workflow

```text
Ingress Created

↓

AWS Load Balancer Controller

↓

Application Load Balancer

↓

Target Groups

↓

Pods
```

Supported Features

- Host Routing
- Path Routing
- SSL
- ACM
- WAF
- HTTP Redirect
- Sticky Sessions

---

# Path-Based Routing

```text
/api

↓

API Pods

----------------

/admin

↓

Admin Pods

----------------

/

↓

Frontend Pods
```

---

# Host-Based Routing

```text
shop.company.com

↓

Frontend

--------------------

api.company.com

↓

Backend

--------------------

admin.company.com

↓

Admin
```

---

# ConfigMaps

ConfigMaps store application configuration.

Example

```yaml
database_host

api_url

environment
```

Applications read configuration dynamically.

---

# Secrets

Secrets store sensitive information.

Examples

- Passwords
- API Keys
- Database Credentials
- Certificates
- Tokens

Production Recommendation

Integrate Kubernetes Secrets with AWS Secrets Manager.

---

# Persistent Storage

Containers are ephemeral.

If a Pod is deleted

Local storage disappears.

Persistent Volumes solve this problem.

---

# Persistent Volume (PV)

Represents actual storage.

Example

```text
Amazon EBS

↓

Persistent Volume

↓

Pod
```

---

# Persistent Volume Claim (PVC)

Applications request storage.

Workflow

```text
PVC

↓

PV

↓

Amazon EBS
```

---

# Storage Classes

Storage Classes dynamically provision storage.

Common Storage Classes

- gp3
- io2
- EFS

---

# Amazon EBS CSI Driver

Provides block storage.

Architecture

```text
Pod

↓

PVC

↓

EBS CSI Driver

↓

Amazon EBS
```

Best for

- Databases
- Stateful Applications

---

# Amazon EFS CSI Driver

Provides shared file storage.

Architecture

```text
Multiple Pods

↓

EFS CSI Driver

↓

Amazon EFS
```

Suitable for

- Shared Content
- Machine Learning
- CMS Applications

---

# Horizontal Pod Autoscaler (HPA)

Automatically scales Pods.

Example

```text
CPU > 70%

↓

HPA

↓

Pods

3

↓

8
```

Metrics

- CPU
- Memory
- Custom Metrics

---

# Cluster Autoscaler

Adds or removes Worker Nodes.

Workflow

```text
Pods Pending

↓

Cluster Autoscaler

↓

Launch EC2

↓

Schedule Pods
```

---

# Karpenter

Karpenter is AWS's next-generation autoscaler.

Instead of scaling fixed node groups,

Karpenter provisions exactly the required instances.

Workflow

```text
Pending Pods

↓

Karpenter

↓

Optimal EC2 Instance

↓

Pods Running
```

Advantages

- Faster Scaling
- Lower Cost
- Better Instance Selection

AWS recommends Karpenter for modern EKS deployments.

---

# Network Policies

Network Policies control Pod-to-Pod communication.

Example

```text
Frontend

↓

Backend

↓

Database

Blocked

↓

Other Pods
```

Implements Zero Trust networking.

---

# Monitoring

Production clusters require monitoring.

Typical Stack

```text
Prometheus

↓

Grafana

↓

Alertmanager

↓

CloudWatch
```

---

# Prometheus

Collects metrics.

Examples

- CPU
- Memory
- Requests
- Errors
- Latency

---

# Grafana

Visualizes metrics.

Common Dashboards

- Cluster Health
- Node Utilization
- Pod CPU
- Memory
- Network

---

# Fluent Bit

Collects logs.

Workflow

```text
Pod Logs

↓

Fluent Bit

↓

CloudWatch Logs
```

---

# CloudWatch Integration

Amazon EKS integrates with CloudWatch Container Insights.

Collects

- Node Metrics
- Pod Metrics
- Cluster Metrics
- Container Logs

Architecture

```text
Pods

↓

Container Insights

↓

CloudWatch

↓

Dashboard

↓

Alarms
```

---

# Security Best Practices

- Enable IRSA
- Use Private Cluster Endpoints
- Enable Control Plane Logging
- Use Security Groups for Pods
- Scan Images in Amazon ECR
- Use Network Policies
- Encrypt Kubernetes Secrets
- Enable Audit Logs
- Use Pod Security Standards
- Keep Kubernetes Version Updated

---

# Summary

Amazon EKS production environments rely on IAM Roles for Service Accounts (IRSA), Services, Ingress, AWS Load Balancer Controller, persistent storage with EBS and EFS CSI drivers, autoscaling through HPA and Karpenter, and comprehensive monitoring using Prometheus, Grafana, Fluent Bit, and CloudWatch. These components provide secure, scalable, and highly available Kubernetes platforms capable of running enterprise microservices in AWS.

---

# Multi-AZ Architecture

Amazon EKS is designed for High Availability.

Production clusters should always span multiple Availability Zones.

Architecture

```text
                    Amazon EKS Cluster

      ┌───────────────────────────────────────┐

      │           Control Plane               │
      │      (Managed by AWS - Multi AZ)      │
      └───────────────────────────────────────┘

              │            │            │

        ┌─────┘            │            └─────┐

        │                  │                  │

   Worker Node       Worker Node       Worker Node

      AZ-A              AZ-B              AZ-C

        │                  │                  │

     Multiple Pods     Multiple Pods     Multiple Pods
```

Benefits

- High Availability
- Fault Tolerance
- Better Scheduling
- Reduced Downtime

---

# Control Plane High Availability

Unlike self-managed Kubernetes,

Amazon EKS automatically deploys the Control Plane across multiple Availability Zones.

Components replicated

- API Server
- etcd
- Controller Manager
- Scheduler

If one Availability Zone fails,

The Control Plane continues operating.

---

# Worker Node High Availability

Worker Nodes should also be distributed across multiple Availability Zones.

Example

```text
AZ-A

2 Worker Nodes

----------------

AZ-B

2 Worker Nodes

----------------

AZ-C

2 Worker Nodes
```

Pods are automatically distributed.

---

# Pod Distribution

Never deploy all Pods on one Worker Node.

Example

```text
Payment Service

↓

Pod-1

AZ-A

--------------

Pod-2

AZ-B

--------------

Pod-3

AZ-C
```

If one AZ fails,

Remaining Pods continue serving traffic.

---

# Pod Anti-Affinity

Pod Anti-Affinity prevents identical Pods from running on the same node.

Example

Without Anti-Affinity

```text
Worker Node

↓

Payment Pod

↓

Payment Pod

↓

Payment Pod
```

One node failure causes complete outage.

With Anti-Affinity

```text
Node-1

↓

Payment Pod

--------------

Node-2

↓

Payment Pod

--------------

Node-3

↓

Payment Pod
```

---

# Node Affinity

Node Affinity schedules Pods onto specific nodes.

Example

```yaml
Frontend

↓

Frontend Nodes

-------------------

Database

↓

Database Nodes
```

Useful for workload isolation.

---

# Taints and Tolerations

Taints prevent Pods from being scheduled on specific nodes.

Example

```text
GPU Node

↓

Only ML Pods
```

Other Pods cannot run unless they have matching tolerations.

---

# Dedicated Node Groups

Large organizations create separate Node Groups.

Example

```text
Production Node Group

↓

Production Apps

------------------------

Monitoring Node Group

↓

Prometheus

Grafana

------------------------

ML Node Group

↓

AI Workloads
```

Benefits

- Isolation
- Security
- Better Resource Allocation

---

# Multi-Cluster Architecture

Many enterprises run multiple EKS clusters.

Example

```text
Development Cluster

↓

Testing Cluster

↓

Staging Cluster

↓

Production Cluster
```

Advantages

- Isolation
- Safer Upgrades
- Independent Scaling
- Better Security

---

# Production Deployment Flow

A typical enterprise deployment pipeline:

```text
Developer

↓

GitHub

↓

Pull Request

↓

GitHub Actions

↓

Docker Build

↓

Amazon ECR

↓

Update Kubernetes Manifest

↓

Git Repository

↓

ArgoCD

↓

Amazon EKS

↓

Pods Updated
```

---

# GitOps with ArgoCD

Git becomes the single source of truth.

Workflow

```text
Git Commit

↓

ArgoCD Detects Change

↓

Sync Cluster

↓

Deploy Application
```

Benefits

- Version Control
- Easy Rollback
- Drift Detection
- Automated Deployments

---

# CI/CD Integration

Typical pipeline

```text
GitHub

↓

Build

↓

Unit Tests

↓

SonarQube

↓

Trivy Scan

↓

Docker Build

↓

Push to Amazon ECR

↓

Update GitOps Repository

↓

ArgoCD

↓

Amazon EKS
```

---

# Rolling Deployment

```text
Version 1

↓

Launch Version 2

↓

Health Check

↓

Terminate Version 1
```

No downtime.

---

# Blue/Green Deployment

```text
Blue Environment

↓

Green Environment

↓

Validation

↓

Traffic Switch
```

Advantages

- Instant Rollback
- Safer Releases

---

# Canary Deployment

Traffic is shifted gradually.

```text
Version 1

100%

↓

Version 2

10%

↓

25%

↓

50%

↓

100%
```

Useful for production releases.

---

# Backup Strategy

Applications are stateless,

but Kubernetes resources must be backed up.

Backup

- Deployments
- Services
- Secrets
- ConfigMaps
- Persistent Volumes

---

# Velero

Velero is the standard Kubernetes backup tool.

Architecture

```text
EKS Cluster

↓

Velero

↓

Amazon S3
```

Supports

- Cluster Backup
- Restore
- Disaster Recovery

---

# Disaster Recovery

Production DR Architecture

```text
Primary Region

↓

Amazon EKS

↓

Amazon ECR Replication

↓

Velero Backup

↓

Amazon S3

↓

Secondary Region

↓

Amazon EKS
```

---

# AWS CLI

Create Cluster

```bash
aws eks create-cluster \
--name production
```

Describe Cluster

```bash
aws eks describe-cluster \
--name production
```

List Clusters

```bash
aws eks list-clusters
```

Update kubeconfig

```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name production
```

---

# eksctl

Create Cluster

```bash
eksctl create cluster \
--name production
```

List Node Groups

```bash
eksctl get nodegroup \
--cluster production
```

Delete Cluster

```bash
eksctl delete cluster \
--name production
```

---

# Terraform

```hcl
module "eks" {

  source = "terraform-aws-modules/eks/aws"

  cluster_name = "production"

  cluster_version = "1.31"

}
```

---

# CloudFormation

```yaml
Resources:

  Cluster:

    Type: AWS::EKS::Cluster

    Properties:

      Name: production
```

---

# Python (Boto3)

```python
import boto3

eks = boto3.client("eks")

response = eks.list_clusters()

print(response)
```

---

# Enterprise Production Architecture

```text
                     Internet

                         │

                      Route53

                         │

                        ALB

                         │

            AWS Load Balancer Controller

                         │

                 Amazon EKS Cluster

      ┌──────────────────────────────────────────┐

      │      Multi-AZ Managed Node Groups        │

      └──────────────────────────────────────────┘

         │          │          │          │

      Frontend   Backend   Payment   Monitoring

         │          │          │          │

       Amazon ECR   EBS   EFS   CloudWatch

                         │

                    Prometheus

                         │

                      Grafana

                         │

                      Fluent Bit

                         │

                      ArgoCD

                         │

                      GitHub
```

---

# Best Practices

- Deploy across at least three Availability Zones
- Use Managed Node Groups
- Enable IRSA
- Use private cluster endpoints
- Enable Control Plane Logs
- Store images in Amazon ECR
- Use immutable image tags
- Use GitOps with ArgoCD
- Enable Cluster Autoscaler or Karpenter
- Configure HPA for workloads
- Enable Prometheus and Grafana
- Use Fluent Bit for centralized logging
- Back up clusters with Velero
- Regularly upgrade Kubernetes versions
- Apply least-privilege RBAC

---

# Common Mistakes

- Running production in a single AZ
- Using the `default` namespace for everything
- Using the `latest` image tag
- Hardcoding AWS credentials inside Pods
- Not enabling IRSA
- Not monitoring the cluster
- Ignoring Pod resource requests and limits
- Not configuring Pod Disruption Budgets
- Running without backups
- Skipping security scans

---

# Troubleshooting

## Pods Stuck in Pending

Check

- Node Capacity
- Resource Requests
- Taints
- PVC Binding
- Scheduler Events

---

## ImagePullBackOff

Verify

- Amazon ECR Repository
- Image Name
- Image Tag
- IAM Permissions
- Network Connectivity

---

## CrashLoopBackOff

Check

- Container Logs
- Readiness Probe
- Liveness Probe
- Environment Variables
- Secrets
- ConfigMaps

---

## Node Not Ready

Verify

- kubelet
- CNI Plugin
- EC2 Health
- IAM Role
- Network

---

## High API Server Latency

Check

- Control Plane Logs
- API Requests
- Excessive Controllers
- Resource Quotas

---

# Interview Questions

### Basic

1. What is Amazon EKS?
2. Why use EKS?
3. Difference between ECS and EKS?
4. What is a Pod?
5. What is a Deployment?
6. What is a Service?
7. What is Ingress?
8. What is IRSA?
9. What is a Node Group?
10. What is a Fargate Profile?

### Intermediate

11. Explain the EKS Control Plane.
12. Managed vs Self-Managed Node Groups?
13. Explain VPC CNI.
14. What is the AWS Load Balancer Controller?
15. Explain HPA.
16. Explain Cluster Autoscaler.
17. What is Karpenter?
18. Explain EBS CSI Driver.
19. Explain EFS CSI Driver.
20. Explain RBAC.

### Advanced

21. Explain IRSA internally.
22. How does IAM authentication work in EKS?
23. Explain Pod networking.
24. Explain Control Plane HA.
25. Design a production EKS cluster.
26. Explain GitOps using ArgoCD.
27. How would you secure an EKS cluster?
28. Explain Blue/Green deployment.
29. Explain Canary deployment.
30. How would you troubleshoot a production EKS outage?

---

# Production Scenarios

### Scenario 1

Your EKS cluster suddenly cannot schedule new Pods.

How would you investigate?

---

### Scenario 2

Pods remain in `ImagePullBackOff` after a deployment.

Which components would you verify?

---

### Scenario 3

One Availability Zone fails.

How does EKS maintain application availability?

---

### Scenario 4

A developer accidentally deployed an image with a critical vulnerability.

How would your CI/CD and GitOps pipeline prevent it from reaching production?

---

### Scenario 5

The Payment service experiences high CPU utilization during a flash sale.

How would HPA, Cluster Autoscaler, and Karpenter work together to handle the load?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Control Plane | Cluster Management |
| Managed Node Group | Managed Worker Nodes |
| Pod | Smallest Deployable Unit |
| Deployment | Pod Management |
| Service | Stable Networking |
| Ingress | HTTP/HTTPS Routing |
| IRSA | Pod IAM Access |
| VPC CNI | Pod Networking |
| EBS CSI | Block Storage |
| EFS CSI | Shared Storage |
| HPA | Scale Pods |
| Cluster Autoscaler | Scale Nodes |
| Karpenter | Intelligent Node Provisioning |
| ArgoCD | GitOps |
| Velero | Backup & Restore |

---

# Summary

Amazon EKS is AWS's managed Kubernetes platform for running containerized workloads at enterprise scale. By combining Managed Control Planes, Multi-AZ Node Groups, IRSA, AWS Load Balancer Controller, EBS/EFS CSI drivers, autoscaling with HPA and Karpenter, GitOps using ArgoCD, and comprehensive monitoring with Prometheus, Grafana, Fluent Bit, and CloudWatch, organizations can build secure, highly available, and production-ready Kubernetes environments. Proper architecture, automation, and observability are essential for operating EKS successfully in real-world production deployments.
