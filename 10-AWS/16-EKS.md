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