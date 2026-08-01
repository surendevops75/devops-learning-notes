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

