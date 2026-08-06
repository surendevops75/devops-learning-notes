# Kubernetes Enterprise Handbook

# Chapter 1 - Kubernetes Architecture Fundamentals

Modern applications require

- High Availability
- Scalability
- Self-Healing
- Zero Downtime Deployments
- Automated Rollbacks
- Infrastructure Automation

Managing hundreds or thousands of containers manually is impossible.

Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, networking, and lifecycle management of containerized applications.

Today Kubernetes is the de facto standard for container orchestration and is used by organizations ranging from startups to Fortune 500 enterprises.

---

# What is Kubernetes?

Kubernetes is a container orchestration platform that manages

- Containers
- Scheduling
- Networking
- Scaling
- Service Discovery
- Load Balancing
- Self Healing
- Rolling Updates

Instead of managing individual Docker containers,

Kubernetes manages the desired state of an application.

---

# Why Kubernetes?

Without Kubernetes

```text
Developer

↓

Docker Container

↓

Manual Deployment

↓

Manual Scaling

↓

Manual Recovery
```

Problems

- Manual operations
- Difficult scaling
- No self healing
- Downtime
- Complex networking

---

With Kubernetes

```text
Developer

↓

Docker Image

↓

Kubernetes

↓

Scheduling

↓

Scaling

↓

Self Healing

↓

Production
```

Everything becomes automated.

---

# Kubernetes Features

Kubernetes provides

- Container Scheduling
- Auto Healing
- Horizontal Scaling
- Rolling Updates
- Rollbacks
- Service Discovery
- Load Balancing
- Secret Management
- Storage Orchestration

---

# Kubernetes Architecture

A Kubernetes cluster consists of

```text
Cluster

├── Control Plane

└── Worker Nodes
```

The Control Plane manages the cluster.

Worker Nodes run applications.

---

# High-Level Architecture

```text
Users

↓

Ingress / Load Balancer

↓

Services

↓

Pods

↓

Worker Nodes

↓

Control Plane
```

---

# Cluster Components

Major components include

Control Plane

- API Server
- etcd
- Scheduler
- Controller Manager
- Cloud Controller Manager

Worker Node

- kubelet
- kube-proxy
- Container Runtime

---

# Control Plane

The Control Plane makes all cluster decisions.

Responsibilities

- Scheduling
- Cluster State
- API Management
- Scaling Decisions
- Node Monitoring

Think of it as the brain of Kubernetes.

---

# Worker Node

Worker Nodes execute workloads.

Each worker node contains

```text
Worker Node

├── kubelet

├── kube-proxy

├── Container Runtime

└── Pods
```

---

# Kubernetes API Server

The API Server is the entry point for all Kubernetes operations.

Every command

```bash
kubectl apply

kubectl get

kubectl delete
```

communicates with the API Server.

---

# API Server Responsibilities

- Authentication
- Authorization
- Admission Control
- Resource Validation
- REST API
- Cluster Communication

All components communicate through the API Server.

---

# etcd

etcd is Kubernetes' distributed key-value database.

It stores

- Cluster State
- Configuration
- Secrets
- Deployments
- Services
- Nodes

Architecture

```text
API Server

↓

etcd

↓

Cluster State
```

Without etcd,

the cluster loses its state information.

---

# Scheduler

The Scheduler decides

where Pods should run.

Workflow

```text
New Pod

↓

Scheduler

↓

Best Worker Node
```

Scheduling decisions consider

- CPU
- Memory
- Affinity
- Taints
- Node Labels
- Resource Availability

---

# Controller Manager

Controllers continuously compare

Desired State

vs

Current State.

Example

Desired

```text
3 Pods
```

Current

```text
2 Pods
```

Controller creates one additional Pod.

---

# Common Controllers

- Deployment Controller
- ReplicaSet Controller
- Node Controller
- Job Controller
- Endpoint Controller

Each controller manages a specific resource.

---

# Cloud Controller Manager

Used in cloud environments.

Integrates Kubernetes with

- AWS
- Azure
- GCP

Responsibilities include

- Load Balancers
- Volumes
- Node Management
- Routes

Amazon EKS uses the AWS Cloud Controller Manager.

---

# kubelet

Every worker node runs kubelet.

Responsibilities

- Register Node
- Monitor Pods
- Pull Images
- Start Containers
- Report Health

Workflow

```text
API Server

↓

kubelet

↓

Container Runtime

↓

Pods
```

---

# kube-proxy

kube-proxy manages networking.

Responsibilities

- Service Networking
- Load Balancing
- Packet Forwarding

It enables communication between Pods and Services.

---

# Container Runtime

The container runtime executes containers.

Examples

- containerd
- CRI-O

(Docker Engine is no longer the default runtime in modern Kubernetes.)

---

# Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod contains

- One or More Containers
- Shared Network
- Shared Storage

Example

```text
Pod

├── Application Container

└── Sidecar Container
```

---

# Node

A Node is a worker machine.

It can be

- Physical Server
- Virtual Machine
- Cloud Instance

Each node hosts multiple Pods.

---

# Cluster

A Cluster is a collection of nodes managed by a Control Plane.

```text
Cluster

├── Control Plane

├── Worker Node 1

├── Worker Node 2

└── Worker Node 3
```

---

# Kubernetes Workflow

```text
Developer

↓

kubectl apply

↓

API Server

↓

Scheduler

↓

Worker Node

↓

Pod Created
```

---

# Desired State

Kubernetes works using declarative configuration.

Example

You declare

```text
Replicas = 3
```

Kubernetes ensures

three Pods are always running.

---

# Self Healing

Suppose

```text
Pod

↓

Crash
```

Controller detects failure

↓

Creates New Pod

Applications remain available.

---

# Horizontal Scaling

Instead of

```text
1 Pod
```

Kubernetes scales to

```text
5 Pods

↓

10 Pods

↓

20 Pods
```

based on demand.

---

# Rolling Updates

Deployment

```text
Version 1

↓

Version 2

↓

Zero Downtime
```

Pods are replaced gradually.

---

# Rollback

If deployment fails

```text
Version 2

↓

Rollback

↓

Version 1
```

Recovery is automatic.

---

# Enterprise Architecture

```text
Users

↓

Route53

↓

Application Load Balancer

↓

Ingress Controller

↓

Kubernetes Service

↓

Pods

↓

Amazon RDS

↓

Amazon S3
```

This architecture supports highly available enterprise applications.

---

# Banking Example

```text
Customers

↓

ALB

↓

Payment Service Pods

↓

Aurora Database

↓

SNS Notifications
```

Each component scales independently.

---

# Kubernetes vs Docker

| Docker | Kubernetes |
|----------|------------|
| Container Runtime | Container Orchestrator |
| Single Host | Cluster |
| Manual Scaling | Auto Scaling |
| Manual Recovery | Self Healing |
| Manual Networking | Built-in Networking |

---

# Kubernetes vs Docker Swarm

| Kubernetes | Docker Swarm |
|-------------|--------------|
| Enterprise Standard | Simpler |
| Rich Ecosystem | Smaller Ecosystem |
| Highly Scalable | Medium Scale |
| Advanced Scheduling | Basic Scheduling |
| Large Community | Smaller Community |

---

# Benefits

- High Availability
- Automatic Recovery
- Horizontal Scaling
- Rolling Updates
- Infrastructure Automation
- Efficient Resource Utilization
- Cloud Portability
- Declarative Management

---

# Best Practices

- Keep Control Plane highly available.
- Use multiple Worker Nodes across Availability Zones.
- Deploy stateless workloads whenever possible.
- Monitor cluster health continuously.
- Secure the API Server.
- Backup etcd regularly.
- Use namespaces to isolate workloads.
- Follow the principle of least privilege.

---

# Common Mistakes

- Running production on a single node.
- Exposing the API Server publicly without protection.
- Ignoring etcd backups.
- Running everything in the default namespace.
- Overcommitting CPU and memory.
- Ignoring resource requests and limits.
- Treating Pods as virtual machines.

---

# Interview Questions

## Basic

- What is Kubernetes?
- Why do we need Kubernetes?
- Explain Kubernetes architecture.
- What are the Control Plane components?
- What is a Worker Node?
- What is a Pod?

## Intermediate

- Explain the responsibilities of the API Server.
- What is etcd?
- How does the Scheduler select a node?
- kubelet vs kube-proxy.
- How does Kubernetes achieve self healing?

## Advanced

- Design a highly available Kubernetes cluster for an enterprise banking platform running across multiple Availability Zones.
- Explain how the Control Plane components interact during a deployment.
- A production Kubernetes cluster experiences a worker node failure during peak traffic. Explain how Kubernetes maintains application availability, schedules replacement Pods, updates Service endpoints, and restores the desired state while minimizing customer impact.

---

# Chapter 2 - Pods, ReplicaSets & Deployments (Deep Dive)

Containers are the building blocks of applications,

but Kubernetes does **not** deploy containers directly.

Instead, Kubernetes manages

```text
Container

↓

Pod

↓

ReplicaSet

↓

Deployment
```

Understanding this hierarchy is essential for designing highly available and scalable applications.

---

# Kubernetes Object Hierarchy

```text
Deployment

↓

ReplicaSet

↓

Pod

↓

Container
```

Each layer has a specific responsibility.

---

# What is a Pod?

A Pod is the **smallest deployable unit** in Kubernetes.

It represents one or more tightly coupled containers that share

- Network
- Storage
- Lifecycle

Architecture

```text
Pod

├── Application Container

└── Sidecar Container
```

Pods are ephemeral by design.

---

# Why Pods Instead of Containers?

Without Pods

```text
Application Container

↓

Managed Individually
```

Problems

- Difficult networking
- No shared storage
- No lifecycle management

---

With Pods

```text
Pod

↓

Containers

↓

Shared Network

↓

Shared Storage
```

Containers work together as one unit.

---

# Pod Characteristics

Every Pod has

- One IP Address
- Shared Network Namespace
- Shared Storage Volumes
- One Lifecycle
- One Scheduling Unit

Containers inside a Pod communicate using

```text
localhost
```

---

# Single Container Pod

Most production Pods contain

one container.

Example

```text
Pod

↓

Nginx Container
```

Simple and easy to manage.

---

# Multi-Container Pod

Some applications require

multiple containers.

Example

```text
Pod

├── Java Application

├── Log Collector

└── Monitoring Agent
```

All containers share the same Pod resources.

---

# Sidecar Pattern

A common multi-container pattern.

```text
Pod

├── Main Application

└── Sidecar Container
```

Sidecars perform

- Logging
- Monitoring
- Proxying
- Metrics Collection

without modifying the main application.

---

# Pod Lifecycle

```text
Pending

↓

Running

↓

Succeeded

↓

Failed

↓

Unknown
```

Pods transition through these phases automatically.

---

# Pending State

The Pod has been accepted,

but containers are not yet running.

Possible reasons

- Image Pull
- Scheduling Delay
- Volume Attachment

---

# Running State

The Pod has started successfully.

Applications begin serving traffic.

---

# Succeeded State

All containers completed successfully.

Usually used for

- Jobs
- Batch Processing

---

# Failed State

Containers terminated unexpectedly.

Possible reasons

- Application Crash
- OOMKilled
- Configuration Errors

---

# Unknown State

The Control Plane cannot determine the Pod status.

Usually caused by

- Network Issues
- Node Failures

---

# Pod Scheduling

Workflow

```text
Pod Created

↓

Scheduler

↓

Worker Node

↓

kubelet

↓

Container Runtime

↓

Running Pod
```

Pods are scheduled only once.

---

# Static Pods

Normally,

Pods are managed through the API Server.

Static Pods are managed directly by

```text
kubelet
```

Common use

- Control Plane Components

---

# What is a ReplicaSet?

A ReplicaSet ensures

a specified number of identical Pods are always running.

Example

Desired

```text
3 Pods
```

Current

```text
2 Pods
```

ReplicaSet creates

```text
1 New Pod
```

---

# ReplicaSet Architecture

```text
ReplicaSet

↓

Pod A

↓

Pod B

↓

Pod C
```

If a Pod fails,

ReplicaSet automatically replaces it.

---

# ReplicaSet Responsibilities

- Maintain Replica Count
- Replace Failed Pods
- Monitor Pod Health

ReplicaSets do **not** perform rolling updates.

---

# ReplicaSet Self Healing

Suppose

```text
Pod B

↓

Deleted
```

ReplicaSet detects

```text
Desired = 3

Current = 2
```

Creates

```text
New Pod
```

Application availability remains unchanged.

---

# Scaling ReplicaSets

Example

Before

```text
Replicas = 3
```

After

```text
Replicas = 8
```

ReplicaSet launches

five additional Pods.

---

# What is a Deployment?

Deployments manage

ReplicaSets.

Architecture

```text
Deployment

↓

ReplicaSet

↓

Pods
```

Deployments provide

- Rolling Updates
- Rollbacks
- Scaling
- Version Management

---

# Why Deployments?

Without Deployments

```text
ReplicaSet

↓

Manual Updates
```

Problems

- Downtime
- Difficult Rollbacks

---

With Deployments

```text
Deployment

↓

Rolling Update

↓

Automatic Rollback
```

Updates become automated.

---

# Deployment Workflow

```text
Developer

↓

kubectl apply

↓

Deployment

↓

ReplicaSet

↓

Pods
```

The Deployment controller manages the complete lifecycle.

---

# Rolling Updates

Deployments update Pods gradually.

Example

```text
Version 1

↓

Pod Updated

↓

Pod Updated

↓

Pod Updated

↓

Version 2
```

Users continue accessing the application during updates.

---

# Rollback

Suppose

Version 2 fails.

Deployment performs

```text
Version 2

↓

Rollback

↓

Version 1
```

Previous ReplicaSet becomes active again.

---

# Deployment Strategy

Default strategy

```text
RollingUpdate
```

Alternative

```text
Recreate
```

RollingUpdate minimizes downtime.

---

# Rolling Update Parameters

Important settings

- maxUnavailable
- maxSurge

Example

```text
Replicas = 4

maxUnavailable = 1

maxSurge = 1
```

Only one Pod becomes unavailable during deployment.

---

# Deployment Revisions

Every Deployment maintains

revision history.

Example

```text
Revision 1

↓

Revision 2

↓

Revision 3
```

Rollback can target any previous revision.

---

# Scaling Deployments

Scaling a Deployment

```text
Deployment

↓

ReplicaSet

↓

10 Pods
```

Deployment updates the ReplicaSet automatically.

---

# Pod Labels

Labels organize Kubernetes resources.

Example

```text
app=frontend

env=production

team=devops
```

Labels are simple key-value pairs.

---

# Selectors

ReplicaSets and Services locate Pods using

selectors.

Example

```text
Selector

↓

app=frontend
```

Only matching Pods are selected.

---

# Labels and Selectors

```text
Deployment

↓

Selector

↓

ReplicaSet

↓

Matching Pods
```

Incorrect selectors may result in

- No Pods Managed
- Wrong Pods Selected

---

# Enterprise Deployment Flow

```text
GitHub

↓

GitHub Actions

↓

Docker Build

↓

Amazon ECR

↓

Deployment

↓

ReplicaSet

↓

Pods

↓

Production
```

This is a common CI/CD workflow.

---

# Banking Example

```text
Payment Deployment

↓

ReplicaSet

↓

5 Payment Pods

↓

Service

↓

Customers
```

If one Pod fails,

another is created automatically.

---

# Deployment vs ReplicaSet

| Deployment | ReplicaSet |
|------------|------------|
| Manages ReplicaSets | Manages Pods |
| Rolling Updates | No Rolling Updates |
| Rollback Support | No Rollback |
| Version History | No Version History |
| Recommended | Rarely Used Directly |

---

# Pod vs Deployment

| Pod | Deployment |
|-----|------------|
| Smallest Unit | Higher-Level Controller |
| Short-Lived | Long-Term Management |
| Single Instance | Multiple Replicas |
| Manual | Automated |

---

# Benefits

- Automatic Recovery
- Zero-Downtime Updates
- Horizontal Scaling
- Version Control
- Rollback Support
- Declarative Configuration

---

# Best Practices

- Use Deployments instead of standalone Pods.
- Avoid creating ReplicaSets manually.
- Keep Pods stateless whenever possible.
- Use meaningful labels and selectors.
- Configure readiness and liveness probes.
- Set CPU and memory requests/limits.
- Use rolling updates for production deployments.
- Maintain Deployment revision history.

---

# Common Mistakes

- Deploying standalone Pods in production.
- Managing ReplicaSets manually.
- Using incorrect label selectors.
- Running multiple unrelated applications in one Pod.
- Forgetting resource requests and limits.
- Using the Recreate strategy for production unnecessarily.
- Deleting Pods manually instead of updating Deployments.

---

# Interview Questions

## Basic

- What is a Pod?
- Why does Kubernetes use Pods instead of containers?
- What is a ReplicaSet?
- What is a Deployment?

## Intermediate

- Deployment vs ReplicaSet.
- Single-container vs Multi-container Pods.
- Explain the Sidecar pattern.
- How does a ReplicaSet provide self-healing?
- Explain Rolling Updates and Rollbacks.

## Advanced

- Design a zero-downtime deployment strategy for a payment application using Deployments, ReplicaSets, and rolling updates.
- Explain what happens internally when `kubectl apply` updates a Deployment from version 1 to version 2.
- A production Deployment running 50 Pods experiences multiple Pod failures during peak traffic. Explain how Kubernetes maintains availability, creates replacement Pods, updates ReplicaSets, manages Service endpoints, and completes the deployment without impacting customers.

---

# Chapter 3 - Kubernetes Services & Networking (Deep Dive)

Pods are temporary.

They can

- Crash
- Restart
- Be Rescheduled
- Receive New IP Addresses

If applications communicated directly with Pod IPs,

production systems would constantly fail.

Kubernetes solves this problem using **Services**.

Services provide

- Stable Networking
- Service Discovery
- Load Balancing
- Internal Communication

---

# Kubernetes Networking Principles

Kubernetes networking is based on four principles.

- Every Pod gets its own IP.
- Pods communicate without NAT.
- Nodes communicate with Pods.
- Applications communicate using Services.

---

# Kubernetes Networking Architecture

```text
Internet

↓

Ingress

↓

Service

↓

Pods

↓

Worker Nodes
```

Services provide stable access to Pods.

---

# Why Services?

Without Services

```text
Client

↓

Pod IP

↓

Pod Restart

↓

New Pod IP

↓

Application Failure
```

---

With Services

```text
Client

↓

Service

↓

Healthy Pod

↓

Healthy Pod

↓

Healthy Pod
```

Applications continue working even if Pods change.

---

# What is a Service?

A Service is a stable network endpoint that provides access to one or more Pods.

A Service

- Has a fixed IP
- Uses Labels
- Uses Selectors
- Load Balances Requests

---

# Service Architecture

```text
Client

↓

Service

↓

Pod A

↓

Pod B

↓

Pod C
```

Traffic is distributed automatically.

---

# Labels and Selectors

Services identify Pods using selectors.

Example

```text
Service

↓

Selector

↓

app=frontend

↓

Matching Pods
```

If labels do not match,

the Service cannot find Pods.

---

# Service Discovery

Instead of remembering Pod IPs,

applications use DNS.

Example

```text
payment-service.default.svc.cluster.local
```

Kubernetes DNS resolves the Service automatically.

---

# Kubernetes DNS

Every cluster runs

CoreDNS.

Workflow

```text
Application

↓

DNS Query

↓

CoreDNS

↓

Service IP

↓

Pods
```

Applications communicate using names rather than IP addresses.

---

# Types of Services

Kubernetes provides

- ClusterIP
- NodePort
- LoadBalancer
- ExternalName

Each serves a different purpose.

---

# ClusterIP

Default Service type.

Architecture

```text
Application

↓

ClusterIP Service

↓

Pods
```

Only accessible inside the cluster.

---

# ClusterIP Use Cases

Ideal for

- Internal APIs
- Databases
- Microservices
- Backend Services

---

# NodePort

Exposes applications through

every Worker Node.

Architecture

```text
Internet

↓

Node IP

↓

NodePort

↓

Pods
```

Useful for testing,

but rarely recommended for production.

---

# NodePort Port Range

Default range

```text
30000

↓

32767
```

Clients access

```text
NodeIP:NodePort
```

---

# LoadBalancer Service

Cloud providers automatically provision

an external Load Balancer.

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

Amazon EKS integrates this with AWS.

---

# LoadBalancer Benefits

- External Access
- Automatic Provisioning
- Health Checks
- High Availability

---

# ExternalName

Maps a Kubernetes Service

to an external DNS name.

Example

```text
Application

↓

ExternalName Service

↓

api.example.com
```

Useful when external systems should appear as Kubernetes Services.

---

# Service Comparison

| Service | Internal | External | Production |
|----------|----------|----------|------------|
| ClusterIP | ✅ | ❌ | ✅ |
| NodePort | ✅ | ✅ | Limited |
| LoadBalancer | ✅ | ✅ | ✅ |
| ExternalName | External DNS | External DNS | Special Cases |

---

# Endpoints

A Service routes traffic using

Endpoints.

Architecture

```text
Service

↓

Endpoint List

↓

Pod A

↓

Pod B

↓

Pod C
```

If a Pod fails,

the endpoint list is updated automatically.

---

# EndpointSlice

Large clusters use

EndpointSlices.

Benefits

- Better Scalability
- Faster Updates
- Reduced API Load

EndpointSlices replace large endpoint objects.

---

# kube-proxy

kube-proxy manages

Service networking.

Responsibilities

- Packet Forwarding
- Load Balancing
- Service Rules

---

# kube-proxy Modes

Common modes

- iptables
- IPVS

IPVS performs better

for large production clusters.

---

# Service Load Balancing

Requests are distributed across

healthy Pods.

Example

```text
Request 1

↓

Pod A

────────────

Request 2

↓

Pod B

────────────

Request 3

↓

Pod C
```

Traffic distribution depends on the proxy mode.

---

# Headless Services

Sometimes applications need

direct Pod access.

Headless Service

```text
ClusterIP = None
```

Applications receive

individual Pod IPs.

Common use cases

- StatefulSets
- Databases
- Kafka
- Elasticsearch

---

# Stateful Application Example

```text
Headless Service

↓

Database Pod 1

↓

Database Pod 2

↓

Database Pod 3
```

Each Pod has its own DNS record.

---

# What is Ingress?

Services expose applications,

but managing multiple Load Balancers becomes expensive.

Ingress provides

HTTP/HTTPS routing.

Architecture

```text
Internet

↓

Ingress Controller

↓

Services

↓

Pods
```

---

# Ingress Controller

The Ingress resource itself

does nothing.

An Ingress Controller implements the routing.

Popular controllers

- AWS Load Balancer Controller
- NGINX Ingress Controller
- Traefik

Amazon EKS commonly uses

AWS Load Balancer Controller.

---

# Ingress Routing

Example

```text
example.com

↓

ALB

↓

Frontend Service

────────────

api.example.com

↓

ALB

↓

Backend Service
```

Multiple applications share

one Load Balancer.

---

# Path-Based Routing

Example

```text
example.com/app

↓

Frontend

────────────

example.com/api

↓

Backend
```

One Load Balancer,

multiple applications.

---

# Host-Based Routing

Example

```text
shop.example.com

↓

Shopping Service

────────────

admin.example.com

↓

Admin Service
```

---

# Ingress vs LoadBalancer

| Ingress | LoadBalancer |
|----------|--------------|
| One ALB | One LB per Service |
| HTTP/HTTPS | Any TCP/UDP |
| Cost Efficient | Higher Cost |
| Path Routing | No Path Routing |
| Host Routing | No Host Routing |

---

# Amazon EKS Networking

Typical architecture

```text
Internet

↓

AWS ALB

↓

Ingress

↓

ClusterIP Service

↓

Pods

↓

Amazon RDS
```

This is the recommended enterprise architecture.

---

# Enterprise Microservices

```text
ALB

↓

Ingress

├── User Service

├── Order Service

├── Payment Service

├── Inventory Service

└── Notification Service
```

One ALB exposes multiple microservices.

---

# Banking Example

```text
Customers

↓

AWS ALB

↓

Payment Service

↓

Aurora

↓

SNS
```

The Payment Service communicates internally using ClusterIP.

---

# Service Mesh (Introduction)

Large organizations often introduce

Service Meshes like

- Istio
- Linkerd

Features

- Traffic Control
- mTLS
- Observability
- Retry
- Circuit Breaking

This topic is covered later.

---

# Enterprise Networking Flow

```text
Users

↓

Route53

↓

AWS WAF

↓

Application Load Balancer

↓

Ingress

↓

ClusterIP Services

↓

Pods

↓

Aurora
```

---

# Benefits

- Stable Networking
- Built-in DNS
- Automatic Load Balancing
- Service Discovery
- High Availability
- Cloud Integration
- Simplified Communication

---

# Best Practices

- Use ClusterIP for internal communication.
- Use Ingress instead of exposing multiple LoadBalancer Services.
- Use meaningful labels and selectors.
- Monitor CoreDNS health.
- Prefer AWS Load Balancer Controller on EKS.
- Use Headless Services only when applications require direct Pod access.
- Keep backend Services private.
- Enable TLS for external traffic.

---

# Common Mistakes

- Accessing Pods directly instead of Services.
- Incorrect label selectors.
- Using NodePort in production unnecessarily.
- Creating one LoadBalancer for every microservice.
- Ignoring DNS resolution issues.
- Exposing databases through external Services.
- Forgetting health checks on Ingress.

---

# Interview Questions

## Basic

- What is a Kubernetes Service?
- Why are Services required?
- What is ClusterIP?
- What is NodePort?
- What is a LoadBalancer Service?

## Intermediate

- ClusterIP vs NodePort vs LoadBalancer.
- Explain how kube-proxy works.
- What is CoreDNS?
- What is a Headless Service?
- Ingress vs LoadBalancer.

## Advanced

- Design a networking architecture for an enterprise microservices platform running on Amazon EKS using AWS Load Balancer Controller, Ingress, ClusterIP Services, and Amazon Route 53.
- Explain the complete network flow from an internet user accessing an application hosted on Amazon EKS through an AWS Application Load Balancer to the target Pods.
- A production Kubernetes cluster hosts 100 microservices. Explain how you would design scalable, secure, and cost-effective networking using Ingress, ClusterIP Services, DNS, health checks, TLS termination, and AWS-native load balancing.

---

# Chapter 4 - ConfigMaps, Secrets & Storage (Deep Dive)

Applications require more than just containers.

They also need

- Configuration
- Credentials
- Certificates
- Persistent Storage
- Shared Volumes

Hardcoding these values inside container images is considered a poor practice.

Kubernetes separates

- Application Code
- Configuration
- Secrets
- Storage

making applications portable, secure, and easier to manage.

---

# Configuration Management

Instead of storing configuration inside the application,

Kubernetes stores configuration externally.

```text
Application

↓

ConfigMap

↓

Runtime Configuration
```

Applications become environment independent.

---

# What is a ConfigMap?

A ConfigMap stores

non-sensitive configuration data.

Examples

- Application Properties
- Environment Variables
- URLs
- Feature Flags
- Port Numbers

---

# ConfigMap Architecture

```text
ConfigMap

↓

Pod

↓

Application
```

Applications read configuration during startup.

---

# Why ConfigMaps?

Without ConfigMaps

```text
Application Code

↓

Database URL

↓

API URL

↓

Port Number
```

Every environment requires rebuilding the image.

---

With ConfigMaps

```text
Application

↓

ConfigMap

↓

Environment-Specific Configuration
```

The same container image works across all environments.

---

# ConfigMap Data

Typical configuration includes

```text
DATABASE_HOST

API_ENDPOINT

LOG_LEVEL

TIMEZONE

PORT
```

Only non-sensitive values should be stored.

---

# ConfigMap Consumption

Applications can consume ConfigMaps as

- Environment Variables
- Mounted Files
- Command-Line Arguments

---

# ConfigMap as Environment Variables

```text
ConfigMap

↓

Environment Variables

↓

Application
```

Most applications use this approach.

---

# ConfigMap as Volume

```text
ConfigMap

↓

Volume

↓

Configuration Files
```

Useful for applications expecting configuration files.

---

# Updating ConfigMaps

When configuration changes

```text
ConfigMap

↓

Pod Restart (often required)

↓

Updated Configuration
```

Some applications reload configuration automatically,

others require a restart.

---

# ConfigMap Best Practices

- Store only non-sensitive data.
- Separate configuration from application code.
- Keep ConfigMaps small and focused.
- Version configuration changes.
- Use different ConfigMaps for different environments.

---

# What are Secrets?

Secrets store

sensitive information.

Examples

- Database Passwords
- API Keys
- OAuth Tokens
- Certificates
- SSH Keys

---

# Secret Architecture

```text
Secret

↓

Pod

↓

Application
```

Sensitive data is injected securely.

---

# Why Secrets?

Without Secrets

```text
Application Code

↓

Database Password

↓

Git Repository
```

Security risk.

---

With Secrets

```text
Application

↓

Secret

↓

Secure Runtime Access
```

Credentials remain outside the application.

---

# Secret Types

Common secret types

- Opaque
- TLS
- Docker Registry
- Service Account Token

Opaque is the most frequently used.

---

# Secrets as Environment Variables

```text
Secret

↓

Environment Variables

↓

Application
```

Applications access credentials during runtime.

---

# Secrets as Mounted Files

```text
Secret

↓

Volume

↓

Certificate Files

↓

Application
```

Useful for

- TLS Certificates
- SSH Keys
- Private Keys

---

# Secret Security

Secrets should

- Never be committed to Git.
- Use RBAC for access control.
- Be encrypted at rest.
- Be rotated periodically.

---

# Kubernetes Secret Encryption

Enterprise clusters typically use

```text
Secret

↓

API Server

↓

Encryption Provider

↓

etcd
```

Without encryption,

Secrets are stored in etcd as plain data.

---

# ConfigMap vs Secret

| ConfigMap | Secret |
|------------|---------|
| Non-Sensitive Data | Sensitive Data |
| Application Config | Passwords, Tokens |
| Plain Configuration | Access Controlled |
| No Credentials | Credentials |

---

# What is a Volume?

Containers are ephemeral.

If a container restarts,

its local filesystem is lost.

Volumes provide persistent storage.

Architecture

```text
Pod

↓

Volume

↓

Storage
```

---

# Why Volumes?

Without Volumes

```text
Application

↓

Container Restart

↓

Data Lost
```

---

With Volumes

```text
Application

↓

Persistent Volume

↓

Data Retained
```

---

# Volume Lifecycle

```text
Pod

↓

Volume

↓

Storage
```

The volume lifecycle depends on the volume type.

---

# Volume Types

Common Kubernetes volumes

- emptyDir
- hostPath
- Persistent Volume
- Persistent Volume Claim
- CSI Volumes

---

# emptyDir

Created when

the Pod starts.

Deleted when

the Pod is removed.

Useful for

- Temporary Files
- Cache
- Intermediate Processing

---

# hostPath

Uses storage

from the worker node.

```text
Node

↓

Filesystem

↓

Pod
```

Mostly used for

development or testing,

not recommended for production.

---

# Persistent Volume (PV)

A Persistent Volume represents

physical storage inside the cluster.

Examples

- Amazon EBS
- Amazon EFS
- NFS
- SAN

---

# Persistent Volume Architecture

```text
Persistent Volume

↓

Persistent Volume Claim

↓

Pod
```

Applications never access PVs directly.

---

# Persistent Volume Claim (PVC)

Applications request storage using

PVCs.

Example

```text
Application

↓

PVC

↓

Persistent Volume
```

This decouples applications from storage implementation.

---

# Dynamic Provisioning

Modern Kubernetes clusters create storage automatically.

```text
PVC

↓

StorageClass

↓

Persistent Volume

↓

Cloud Storage
```

No manual provisioning required.

---

# StorageClass

A StorageClass defines

how storage should be provisioned.

Example

```text
StorageClass

↓

Amazon EBS

↓

gp3
```

Different workloads can use different StorageClasses.

---

# Amazon EBS CSI Driver

For Amazon EKS,

Persistent Volumes are commonly backed by

Amazon EBS.

Architecture

```text
Pod

↓

PVC

↓

StorageClass

↓

EBS CSI Driver

↓

Amazon EBS
```

---

# Amazon EFS CSI Driver

Shared storage

across multiple Pods.

```text
Multiple Pods

↓

PVC

↓

EFS CSI Driver

↓

Amazon EFS
```

Useful for

- Shared Files
- CMS
- Machine Learning
- Analytics

---

# Stateful Applications

Examples

- PostgreSQL
- MySQL
- MongoDB
- Elasticsearch
- Kafka

These applications require persistent storage.

---

# Stateless vs Stateful

| Stateless | Stateful |
|------------|-----------|
| No Persistent Data | Persistent Data |
| Easy to Scale | Requires Storage |
| Web APIs | Databases |
| Frontend Services | Kafka, Elasticsearch |

---

# Enterprise Architecture

```text
Application

↓

ConfigMap

↓

Secrets

↓

Deployment

↓

PVC

↓

Amazon EBS

↓

Database
```

Configuration,

credentials,

and storage

remain independent.

---

# Banking Example

```text
Payment Service

↓

Secret

↓

Database Password

────────────

PVC

↓

Aurora Backup Files

────────────

ConfigMap

↓

Application Configuration
```

---

# Production Storage Architecture

```text
Pods

↓

PVC

↓

StorageClass

↓

CSI Driver

↓

Amazon EBS

↓

AWS Infrastructure
```

---

# Benefits

- Configuration Separation
- Secure Credential Management
- Persistent Storage
- Cloud-Native Storage Provisioning
- Environment Independence
- Better Security
- Simplified Operations

---

# Best Practices

- Store passwords only in Secrets.
- Store application configuration in ConfigMaps.
- Enable encryption for Secrets in etcd.
- Use PVCs instead of hostPath in production.
- Use dynamic provisioning with StorageClasses.
- Regularly rotate credentials.
- Back up persistent volumes.
- Use Amazon EFS for shared storage and Amazon EBS for block storage.

---

# Common Mistakes

- Storing passwords in ConfigMaps.
- Committing Secrets to Git repositories.
- Using hostPath in production.
- Hardcoding database credentials in container images.
- Not enabling Secret encryption.
- Using emptyDir for persistent application data.
- Deleting PVCs without understanding reclaim policies.

---

# Interview Questions

## Basic

- What is a ConfigMap?
- What is a Secret?
- ConfigMap vs Secret.
- What is a Persistent Volume?
- What is a Persistent Volume Claim?

## Intermediate

- Explain dynamic provisioning.
- What is a StorageClass?
- Amazon EBS vs Amazon EFS.
- How are Secrets stored in Kubernetes?
- How do applications consume ConfigMaps?

## Advanced

- Design persistent storage for a highly available PostgreSQL database running on Amazon EKS using Persistent Volumes, Persistent Volume Claims, StorageClasses, and the EBS CSI Driver.
- Explain how ConfigMaps, Secrets, and Persistent Volumes work together in a production microservices application deployed on Kubernetes.
- A financial application running on Amazon EKS requires secure credential management, dynamic storage provisioning, encrypted persistent volumes, and shared storage for reporting services. Design the complete architecture, explaining every Kubernetes resource and AWS integration.

---

# Chapter 5 - Scheduling, Resource Management & Autoscaling (Deep Dive)

Deploying applications to Kubernetes is only the beginning.

The next challenge is deciding

- Which node should run a Pod?
- How much CPU should a Pod receive?
- What happens when a node is full?
- How does Kubernetes scale automatically?
- How are workloads isolated?

Kubernetes answers these questions through

- Scheduling
- Resource Management
- Node Placement
- Autoscaling

These features ensure efficient cluster utilization and application reliability.

---

# Scheduling Overview

Whenever a new Pod is created,

Kubernetes must select

the most suitable Worker Node.

Workflow

```text
Pod Created

↓

API Server

↓

Scheduler

↓

Best Node Selected

↓

kubelet

↓

Pod Running
```

The Scheduler only assigns the Pod.

The kubelet starts the containers.

---

# Scheduler Responsibilities

The Scheduler evaluates

- CPU Availability
- Memory Availability
- Node Labels
- Node Affinity
- Taints & Tolerations
- Resource Requests
- Existing Workloads

The goal is balanced and efficient resource utilization.

---

# Scheduling Process

```text
Unscheduled Pod

↓

Filter Nodes

↓

Score Nodes

↓

Select Best Node

↓

Bind Pod
```

This process happens automatically.

---

# Resource Requests

Requests define

the minimum resources a Pod requires.

Example

```text
CPU

↓

500m

────────────

Memory

↓

512Mi
```

The Scheduler uses requests

when selecting nodes.

---

# Resource Limits

Limits define

the maximum resources a Pod can consume.

Example

```text
CPU

↓

1 Core

────────────

Memory

↓

1Gi
```

Limits prevent one application

from consuming all cluster resources.

---

# Requests vs Limits

| Requests | Limits |
|-----------|---------|
| Minimum Guaranteed | Maximum Allowed |
| Used for Scheduling | Used During Runtime |
| Reserved Resources | Resource Cap |

---

# CPU Scheduling

Example

Node Capacity

```text
4 CPU
```

Current Usage

```text
2 CPU
```

New Pod Request

```text
1 CPU
```

Scheduler places the Pod

because sufficient CPU exists.

---

# Memory Scheduling

Unlike CPU,

memory cannot be overcommitted safely.

If memory is exhausted,

Pods may be terminated with

```text
OOMKilled
```

Always define memory requests and limits.

---

# Quality of Service (QoS)

Kubernetes classifies Pods into

```text
Guaranteed

↓

Burstable

↓

BestEffort
```

QoS affects eviction priority during resource pressure.

---

# Guaranteed

Requests equal limits.

Example

```text
CPU Request

=

CPU Limit
```

Highest priority.

Least likely to be evicted.

---

# Burstable

Requests are lower than limits.

Most production workloads use

Burstable QoS.

---

# BestEffort

No requests.

No limits.

Lowest scheduling priority.

First to be evicted.

Never use for production workloads.

---

# Node Labels

Labels classify nodes.

Example

```text
disk=ssd

zone=ap-south-1a

environment=production
```

Scheduler uses labels

to place workloads.

---

# Node Selector

Simplest scheduling mechanism.

Example

```text
Node Selector

↓

environment=production

↓

Matching Nodes
```

Pods run only on matching nodes.

---

# Node Affinity

More flexible than Node Selector.

Supports

- Required Rules
- Preferred Rules

Architecture

```text
Pod

↓

Affinity Rules

↓

Matching Nodes
```

---

# Required Affinity

Pod **must** run

on matching nodes.

If no node matches,

the Pod remains Pending.

---

# Preferred Affinity

Scheduler attempts

to place Pods

on preferred nodes,

but may choose another node if necessary.

---

# Pod Affinity

Pods can be scheduled

near other Pods.

Example

```text
Frontend

↓

Backend

↓

Same Node
```

Useful when reducing latency.

---

# Pod Anti-Affinity

Pods can also be separated.

Example

```text
Replica 1

↓

Node A

────────────

Replica 2

↓

Node B
```

Improves availability.

---

# Taints

Taints repel Pods.

Example

```text
Node

↓

NoSchedule
```

Pods cannot be scheduled

unless they tolerate the taint.

---

# Tolerations

Tolerations allow Pods

to run on tainted nodes.

Workflow

```text
Tainted Node

↓

Matching Toleration

↓

Pod Scheduled
```

---

# Common Taint Effects

- NoSchedule
- PreferNoSchedule
- NoExecute

Each defines

how strictly scheduling is enforced.

---

# Dedicated Nodes

Example

```text
GPU Node

↓

Taint

↓

ML Pods Only
```

Other workloads cannot run there.

---

# Resource Quotas

Namespaces can enforce

resource consumption.

Example

```text
Development Namespace

↓

Maximum CPU

↓

Maximum Memory

↓

Maximum Pods
```

Prevents resource exhaustion.

---

# LimitRanges

LimitRanges enforce

default resource requests and limits.

Applications automatically receive defaults

if not explicitly specified.

---

# Horizontal Pod Autoscaler (HPA)

HPA scales

Pods

based on metrics.

Example

```text
CPU 80%

↓

Scale Out

↓

10 Pods
```

When demand decreases,

Pods scale back in.

---

# HPA Workflow

```text
Metrics Server

↓

HPA

↓

Deployment

↓

ReplicaSet

↓

Pods
```

Metrics drive scaling decisions.

---

# HPA Metrics

Common metrics

- CPU Utilization
- Memory Utilization
- Custom Metrics
- External Metrics

---

# Vertical Pod Autoscaler (VPA)

Instead of scaling Pods,

VPA adjusts

Pod resource requests.

Example

```text
512Mi

↓

1Gi Memory
```

Useful for workloads

with changing resource requirements.

---

# Cluster Autoscaler

Cluster Autoscaler scales

Worker Nodes.

Workflow

```text
Pending Pods

↓

Cluster Autoscaler

↓

Add Worker Node

↓

Scheduler

↓

Pod Running
```

Works alongside HPA.

---

# HPA vs VPA vs Cluster Autoscaler

| Feature | HPA | VPA | Cluster Autoscaler |
|----------|-----|-----|--------------------|
| Scales Pods | ✅ | ❌ | ❌ |
| Adjusts Resources | ❌ | ✅ | ❌ |
| Adds Nodes | ❌ | ❌ | ✅ |

---

# Enterprise Autoscaling

```text
Traffic Increase

↓

HPA

↓

More Pods

↓

Cluster Autoscaler

↓

More Nodes
```

Applications remain responsive

during traffic spikes.

---

# Amazon EKS Example

```text
Users

↓

ALB

↓

Deployment

↓

HPA

↓

Pods

↓

Managed Node Group

↓

Cluster Autoscaler
```

AWS infrastructure scales automatically.

---

# Banking Example

```text
Payment Service

↓

CPU 85%

↓

HPA

↓

20 Pods

↓

Cluster Autoscaler

↓

Additional Worker Node
```

Customer transactions continue

without interruption.

---

# Scheduling Priorities

PriorityClasses determine

which Pods should run first.

Critical workloads

can preempt

lower-priority Pods.

---

# Enterprise Scheduling Strategy

```text
Critical Banking Pods

↓

High Priority

────────────

Reporting Jobs

↓

Medium Priority

────────────

Batch Jobs

↓

Low Priority
```

Business-critical applications

receive resources first.

---

# Benefits

- Efficient Resource Utilization
- Automatic Scaling
- High Availability
- Better Scheduling Decisions
- Workload Isolation
- Cost Optimization
- Improved Performance

---

# Best Practices

- Always define CPU and memory requests.
- Set resource limits for every production Pod.
- Use Node Affinity instead of Node Selector where flexibility is needed.
- Separate critical workloads using taints and tolerations.
- Configure HPA for stateless applications.
- Use Cluster Autoscaler in cloud environments.
- Apply ResourceQuotas to namespaces.
- Monitor resource utilization continuously.

---

# Common Mistakes

- Running Pods without resource requests.
- Using BestEffort QoS in production.
- Ignoring node affinity rules.
- Mixing production and development workloads on the same nodes.
- Overcommitting memory.
- Using HPA without Metrics Server.
- Forgetting to configure Cluster Autoscaler.

---

# Interview Questions

## Basic

- What is the Kubernetes Scheduler?
- What are resource requests and limits?
- What is HPA?

## Intermediate

- Requests vs Limits.
- Node Selector vs Node Affinity.
- Taints vs Tolerations.
- HPA vs VPA.
- Explain QoS classes.

## Advanced

- Design an autoscaling strategy for a high-traffic e-commerce platform running on Amazon EKS using HPA, Cluster Autoscaler, node affinity, and resource quotas.
- Explain how the Kubernetes Scheduler selects a worker node for a Pod, including filtering, scoring, affinity rules, taints, tolerations, and resource availability.
- A production Kubernetes cluster experiences a sudden traffic spike causing CPU utilization to exceed 90%, multiple Pods remain Pending, and one Availability Zone becomes resource constrained. Explain how HPA, Cluster Autoscaler, resource requests, scheduling policies, and workload priorities work together to restore application availability while maintaining efficient resource utilization.

---

# Chapter 6 - Kubernetes Security (RBAC, Service Accounts, Network Policies & Pod Security)

Security is one of the most critical aspects of a Kubernetes cluster.

A production Kubernetes environment must protect

- Applications
- Containers
- Secrets
- APIs
- Worker Nodes
- Network Traffic

Kubernetes provides multiple layers of security.

```text
Authentication

↓

Authorization

↓

Admission Control

↓

Network Security

↓

Pod Security

↓

Runtime Security
```

Each layer contributes to the overall security posture.

---

# Kubernetes Security Architecture

```text
Users

↓

API Server

↓

Authentication

↓

Authorization (RBAC)

↓

Admission Controller

↓

Cluster Resources
```

Every request passes through these security layers.

---

# Security Principles

Production Kubernetes clusters follow

- Least Privilege
- Zero Trust
- Defense in Depth
- Network Segmentation
- Encryption
- Continuous Monitoring

---

# Authentication

Authentication answers

```text
Who are you?
```

Examples

- X.509 Certificates
- Service Accounts
- OpenID Connect (OIDC)
- IAM (Amazon EKS)

---

# Authorization

Authorization answers

```text
What are you allowed to do?
```

Kubernetes commonly uses

RBAC

(Role-Based Access Control)

---

# Admission Controllers

Admission Controllers validate requests

before they reach etcd.

Examples

- Resource Validation
- Security Policies
- Image Verification
- Default Configuration

---

# RBAC Overview

RBAC controls access

to Kubernetes resources.

Architecture

```text
User

↓

Role

↓

RoleBinding

↓

Permissions
```

RBAC follows the principle of least privilege.

---

# RBAC Components

- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding

---

# Role

A Role grants permissions

inside a namespace.

Example

```text
Namespace

↓

Developer

↓

Read Pods

↓

Read Services
```

Access is limited to that namespace.

---

# ClusterRole

ClusterRoles grant permissions

across the entire cluster.

Example

```text
Cluster

↓

Administrator

↓

Manage Nodes

↓

Manage Namespaces
```

---

# RoleBinding

RoleBinding connects

```text
User

↓

Role

↓

Namespace
```

Permissions become effective.

---

# ClusterRoleBinding

Cluster-wide equivalent

of RoleBinding.

```text
User

↓

ClusterRole

↓

Entire Cluster
```

---

# RBAC Workflow

```text
User

↓

API Server

↓

RBAC

↓

Allow

↓

Request Processed
```

If permission is missing,

the request is denied.

---

# Principle of Least Privilege

Instead of

```text
Administrator Access
```

Grant

```text
Read Pods

↓

Deployments

↓

Services
```

Only required permissions.

---

# Service Accounts

Applications inside Pods

should never use human accounts.

Instead,

they use

Service Accounts.

---

# Service Account Architecture

```text
Pod

↓

Service Account

↓

API Server
```

Applications authenticate securely.

---

# Why Service Accounts?

Without Service Accounts

```text
Application

↓

Administrator Credentials
```

Security risk.

---

With Service Accounts

```text
Application

↓

Dedicated Identity

↓

Least Privilege
```

---

# Service Account Example

```text
Payment Service

↓

Payment Service Account

↓

Read Secrets

↓

Access ConfigMaps
```

Each application receives

its own identity.

---

# IAM Roles for Service Accounts (IRSA)

Amazon EKS integrates

Service Accounts

with

AWS IAM.

Architecture

```text
Pod

↓

Service Account

↓

IAM Role

↓

AWS Services
```

Applications access AWS securely

without static credentials.

---

# IRSA Example

```text
Reporting Pod

↓

IAM Role

↓

Amazon S3
```

No access keys stored

inside the Pod.

---

# Network Policies

By default,

Pods can communicate

with every other Pod.

Network Policies restrict traffic.

---

# Network Policy Architecture

```text
Frontend

↓

Allowed

↓

Backend

────────────

Frontend

↓

Blocked

↓

Database
```

Only authorized communication is permitted.

---

# Ingress Rules

Ingress rules define

incoming traffic.

Example

```text
Frontend

↓

Backend
```

Allowed.

---

# Egress Rules

Egress rules define

outgoing traffic.

Example

```text
Backend

↓

Database
```

Allowed.

Everything else

can be denied.

---

# Zero Trust Networking

Every connection

must be explicitly allowed.

```text
Pod

↓

Network Policy

↓

Allowed Communication
```

No implicit trust exists.

---

# Pod Security

Pods themselves

should follow

security best practices.

Examples

- Non-root User
- Read-only Filesystem
- Drop Linux Capabilities
- Restricted Privileges

---

# Security Context

SecurityContext controls

container privileges.

Common settings

- runAsNonRoot
- readOnlyRootFilesystem
- allowPrivilegeEscalation
- fsGroup

---

# Example Security Context

```text
Container

↓

Non-root User

↓

Read-only Filesystem

↓

Restricted Capabilities
```

---

# Pod Security Standards

Kubernetes defines

three security profiles.

```text
Privileged

↓

Baseline

↓

Restricted
```

Production clusters

should aim for

Restricted.

---

# Image Security

Only trusted images

should be deployed.

Enterprise pipelines use

```text
GitHub Actions

↓

Trivy

↓

Image Scan

↓

Amazon ECR

↓

Deployment
```

Images are scanned

before production.

---

# Secret Security

Best practices

- Encrypt Secrets
- RBAC Protection
- Rotate Credentials
- Use External Secret Managers

---

# API Server Security

Protect the API Server using

- TLS
- Authentication
- Authorization
- Audit Logging
- Network Restrictions

The API Server should never be publicly exposed without proper controls.

---

# Node Security

Protect Worker Nodes

using

- OS Hardening
- Regular Patching
- Minimal Packages
- Restricted SSH Access

---

# etcd Security

Protect etcd using

- TLS Encryption
- Encryption at Rest
- Regular Backups
- Restricted Access

Compromise of etcd

means compromise

of the cluster.

---

# Runtime Security

Monitor containers

for

- Privilege Escalation
- Suspicious Processes
- File System Changes
- Unexpected Network Activity

---

# Enterprise Security Architecture

```text
Users

↓

OIDC

↓

API Server

↓

RBAC

↓

Network Policies

↓

Pods

↓

IRSA

↓

AWS Services
```

Every layer contributes

to cluster security.

---

# Amazon EKS Security

Typical architecture

```text
IAM

↓

EKS Cluster

↓

Service Accounts

↓

IAM Roles (IRSA)

↓

Amazon S3

↓

Secrets Manager
```

Applications securely access AWS services.

---

# Banking Example

```text
Payment Service

↓

Restricted Network Policy

↓

Dedicated Service Account

↓

IAM Role

↓

Encrypted Secrets

↓

Aurora Database
```

Every component follows

least privilege.

---

# Security Best Practices

- Enable RBAC.
- Follow the principle of least privilege.
- Use Service Accounts for applications.
- Use IRSA instead of static AWS credentials.
- Apply Network Policies.
- Run containers as non-root.
- Scan images before deployment.
- Encrypt Secrets and etcd.
- Restrict API Server access.
- Monitor cluster activity continuously.

---

# Common Mistakes

- Giving cluster-admin access to everyone.
- Running containers as root.
- Using default Service Accounts.
- Storing AWS access keys inside Pods.
- Deploying unscanned images.
- Ignoring Network Policies.
- Exposing the Kubernetes API publicly without restrictions.

---

# Interview Questions

## Basic

- What is RBAC?
- Role vs ClusterRole.
- What is a Service Account?
- Why are Network Policies required?

## Intermediate

- RoleBinding vs ClusterRoleBinding.
- Explain IRSA in Amazon EKS.
- What is a SecurityContext?
- Explain Pod Security Standards.
- How are Secrets protected in Kubernetes?

## Advanced

- Design a secure Amazon EKS platform using IAM, RBAC, IRSA, Network Policies, encrypted Secrets, Pod Security Standards, and least-privilege access.
- Explain the complete request flow when a Pod running in Amazon EKS accesses an Amazon S3 bucket using IAM Roles for Service Accounts.
- A financial institution must deploy sensitive workloads on Kubernetes while meeting PCI-DSS requirements. Design the complete Kubernetes security architecture, including authentication, authorization, network segmentation, workload identity, secret management, image security, runtime security, monitoring, and AWS integrations.

---

# Chapter 7 - Kubernetes Monitoring, Logging & Observability (Deep Dive)

Running applications in Kubernetes is not enough.

Production systems must also answer questions like

- Is the application healthy?
- Why is a Pod restarting?
- Which node is overloaded?
- Why is latency increasing?
- Which deployment caused the issue?

This is where **Observability** becomes essential.

Observability combines

- Metrics
- Logs
- Events
- Alerts

to provide complete visibility into the cluster.

---

# What is Observability?

Observability is the ability to understand the internal state of a system using external signals.

The three pillars are

```text
Metrics

↓

Logs

↓

Traces
```

For your stack, the primary focus is

- Prometheus
- Grafana
- ELK Stack

---

# Kubernetes Monitoring Architecture

```text
Applications

↓

Prometheus

↓

Grafana

↓

Alerts

↓

Operations Team
```

Logs are collected separately.

```text
Applications

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana
```

---

# Why Monitoring?

Without monitoring

```text
Application Failure

↓

Customer Complaints

↓

Investigation Starts
```

Problems are discovered too late.

---

With monitoring

```text
Application Issue

↓

Alert

↓

Engineer

↓

Resolution
```

Problems are detected before customers notice them.

---

# Monitoring Components

A production Kubernetes cluster monitors

- Nodes
- Pods
- Containers
- Applications
- Network
- Storage
- Cluster Components

---

# Metrics

Metrics are numerical values collected over time.

Examples

- CPU Usage
- Memory Usage
- Pod Count
- Request Rate
- Error Rate
- Latency

---

# Logs

Logs record events occurring inside applications.

Examples

- Startup Messages
- Errors
- Warnings
- API Requests
- Authentication Events

---

# Kubernetes Events

Events describe cluster activities.

Examples

```text
Pod Scheduled

↓

Container Started

↓

Image Pulled

↓

OOMKilled
```

Events help during troubleshooting.

---

# Prometheus

Prometheus is the most popular monitoring system for Kubernetes.

Responsibilities

- Metric Collection
- Time-Series Storage
- Alert Evaluation

---

# Prometheus Architecture

```text
Applications

↓

Metrics Endpoint

↓

Prometheus

↓

Time-Series Database

↓

Grafana
```

---

# Metrics Collection

Applications expose

```text
/metrics
```

Prometheus periodically scrapes these endpoints.

---

# Prometheus Pull Model

Unlike many monitoring systems,

Prometheus uses

```text
Prometheus

↓

Pull Metrics

↓

Applications
```

instead of agents pushing metrics.

---

# Common Kubernetes Metrics

Cluster Metrics

- Node CPU
- Node Memory
- Disk Usage
- Network Usage

Pod Metrics

- CPU
- Memory
- Restarts
- Running Pods

Application Metrics

- Request Rate
- Error Rate
- Response Time

---

# kube-state-metrics

Provides Kubernetes object metrics.

Examples

- Deployments
- ReplicaSets
- Pods
- StatefulSets
- Nodes

Prometheus collects these metrics.

---

# Metrics Server

Metrics Server provides

resource utilization

to Kubernetes.

Used by

- HPA
- kubectl top

It is not a replacement for Prometheus.

---

# Grafana

Grafana visualizes metrics.

Architecture

```text
Prometheus

↓

Grafana

↓

Dashboards
```

Engineers monitor applications

through dashboards.

---

# Common Dashboards

- Cluster Health
- Node Utilization
- Pod Health
- Application Performance
- Resource Usage

---

# Alerts

Monitoring without alerts

has little value.

Alert workflow

```text
Metric

↓

Threshold

↓

Alert

↓

Engineer
```

---

# Alertmanager

Prometheus sends alerts to

Alertmanager.

```text
Prometheus

↓

Alertmanager

↓

Email

↓

Slack

↓

PagerDuty
```

Alertmanager handles routing and grouping.

---

# Common Alerts

Examples

```text
CPU > 80%

Memory > 85%

Pod Restart

Node Not Ready

Disk Full

Certificate Expiring
```

Alerts should be actionable.

---

# Logging Architecture

A common Kubernetes logging pipeline

```text
Pods

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana
```

Logs are centralized for analysis.

---

# Why Centralized Logging?

Without centralized logging

```text
Pod Deleted

↓

Logs Lost
```

---

With centralized logging

```text
Pod

↓

Fluent Bit

↓

ELK

↓

Searchable Logs
```

Logs remain available

even after Pods are deleted.

---

# Fluent Bit

Fluent Bit collects

container logs

from worker nodes

and forwards them to Elasticsearch.

It is lightweight

and widely used in Kubernetes.

---

# Elasticsearch

Elasticsearch stores

and indexes logs.

Supports

- Full Text Search
- Filtering
- Aggregation
- Fast Queries

---

# Kibana

Kibana provides

a web interface

to search and analyze logs.

Engineers use Kibana for

- Troubleshooting
- Log Analysis
- Security Investigation

---

# Monitoring Cluster Components

Monitor

- API Server
- Scheduler
- Controller Manager
- etcd
- kubelet
- CoreDNS

These components determine cluster health.

---

# Node Monitoring

Monitor

- CPU
- Memory
- Disk
- Filesystem
- Network
- Pressure Conditions

Node failures impact many Pods.

---

# Pod Monitoring

Monitor

- Restart Count
- CrashLoopBackOff
- OOMKilled
- Pending Pods
- Readiness
- Liveness

---

# Deployment Monitoring

Track

- Replica Count
- Available Pods
- Rollout Status
- Failed Rollouts

---

# Application Monitoring

Monitor

- Request Rate
- Response Time
- Error Rate
- Active Users
- Business Transactions

Infrastructure metrics alone

are not enough.

---

# Enterprise Monitoring Architecture

```text
Applications

↓

Prometheus

↓

Grafana

↓

Alertmanager

↓

Operations Team

────────────

Applications

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana
```

---

# Amazon EKS Monitoring

Typical production architecture

```text
Amazon EKS

↓

Prometheus

↓

Grafana

↓

CloudWatch

↓

Alerts
```

CloudWatch complements

cluster-level monitoring.

---

# Banking Example

```text
Payment Service

↓

Prometheus Metrics

↓

CPU

↓

Latency

↓

Error Rate

↓

Grafana Dashboard

↓

Alertmanager
```

Operations teams detect issues

before customers are impacted.

---

# Golden Signals

Google's SRE model recommends monitoring

```text
Latency

↓

Traffic

↓

Errors

↓

Saturation
```

These four metrics quickly reveal application health.

---

# Enterprise Monitoring Flow

```text
Application

↓

Metrics

↓

Prometheus

↓

Grafana

↓

Alertmanager

↓

Engineer

↓

Resolution
```

---

# Benefits

- Faster Incident Detection
- Better Troubleshooting
- Historical Analysis
- Capacity Planning
- Performance Optimization
- Reduced Downtime
- Improved Reliability

---

# Best Practices

- Monitor infrastructure and applications together.
- Create dashboards for every critical service.
- Alert only on actionable conditions.
- Centralize logs using ELK.
- Retain logs according to compliance requirements.
- Monitor Kubernetes control plane components.
- Review dashboards regularly.
- Test alerts periodically.

---

# Common Mistakes

- Monitoring only CPU and memory.
- Ignoring application metrics.
- Generating too many alerts (alert fatigue).
- Storing logs only inside containers.
- Not monitoring Kubernetes control plane components.
- Missing business-level metrics.
- Ignoring alert testing.

---

# Interview Questions

## Basic

- What is Prometheus?
- What is Grafana?
- Why is centralized logging required?
- What is the Metrics Server?

## Intermediate

- Prometheus vs Metrics Server.
- Explain the ELK Stack.
- How does Prometheus collect metrics?
- Why is Alertmanager required?
- What are Kubernetes Events?

## Advanced

- Design a complete monitoring and logging platform for a production Amazon EKS cluster using Prometheus, Grafana, Alertmanager, Fluent Bit, Elasticsearch, and Kibana.
- Explain the end-to-end monitoring flow from an application exposing metrics to alert generation and engineer notification.
- A production Kubernetes platform experiences increased latency, Pod restarts, and intermittent API failures during peak traffic. Explain how you would use Prometheus, Grafana, Kubernetes Events, ELK Stack, and CloudWatch to identify the root cause, validate the fix, and implement long-term monitoring improvements.

---

# Chapter 8 - Kubernetes Troubleshooting (50+ Production Scenarios)

Kubernetes is designed to be self-healing,

but production issues still occur.

A senior DevOps Engineer is expected to troubleshoot

- Pods
- Nodes
- Networking
- Storage
- Deployments
- DNS
- Ingress
- Autoscaling
- Security
- Performance

Interviewers rarely ask

"What is Kubernetes?"

Instead, they ask

> "Production is down. How will you troubleshoot?"

This chapter provides a structured approach to real-world production incidents.

---

# Enterprise Troubleshooting Framework

Always follow a structured approach.

```text
Alert

↓

Understand Impact

↓

Collect Evidence

↓

Check Cluster Health

↓

Check Node Health

↓

Check Pod Health

↓

Logs

↓

Events

↓

Metrics

↓

Root Cause

↓

Fix

↓

Validation

↓

Postmortem
```

Never jump directly to conclusions.

---

# Scenario 1 - Pod Stuck in Pending State

## Symptoms

```text
Pod

↓

Pending
```

---

## Investigation

Check

```bash
kubectl describe pod <pod-name>
```

Look for

- FailedScheduling
- Insufficient CPU
- Insufficient Memory
- Node Selector Issues
- Taints
- PVC Pending

---

## Possible Causes

- Cluster Out of Resources
- Node Affinity Mismatch
- Missing Storage
- Tainted Nodes
- Unschedulable Nodes

---

## Resolution

- Scale worker nodes.
- Reduce resource requests.
- Verify node selectors.
- Check PVC binding.
- Review taints and tolerations.

---

# Scenario 2 - CrashLoopBackOff

## Symptoms

```text
Pod

↓

Crash

↓

Restart

↓

Crash

↓

CrashLoopBackOff
```

---

## Investigation

```bash
kubectl logs <pod>

kubectl logs <pod> --previous

kubectl describe pod
```

---

## Common Causes

- Application Crash
- Invalid ConfigMap
- Missing Secret
- Database Connection Failure
- Startup Script Failure

---

## Resolution

- Fix application issue.
- Verify environment variables.
- Check ConfigMaps.
- Validate Secrets.
- Test database connectivity.

---

# Scenario 3 - ImagePullBackOff

## Investigation

Check

```bash
kubectl describe pod
```

---

## Common Causes

- Wrong Image Name
- Wrong Tag
- Private Registry Authentication
- Image Deleted

---

## Resolution

- Verify image.
- Check registry credentials.
- Validate imagePullSecrets.
- Test registry access.

---

# Scenario 4 - OOMKilled

Symptoms

```text
Container

↓

Killed

↓

OOMKilled
```

---

## Investigation

Check

```bash
kubectl describe pod
```

Review

- Memory Usage
- Requests
- Limits

---

## Resolution

- Increase memory limit.
- Fix memory leak.
- Optimize application.
- Configure VPA if appropriate.

---

# Scenario 5 - Node Not Ready

## Investigation

```bash
kubectl get nodes

kubectl describe node
```

---

Check

- kubelet
- Disk Pressure
- Memory Pressure
- Network
- Container Runtime

---

## Resolution

- Restart kubelet.
- Free disk space.
- Verify networking.
- Recover node health.

---

# Scenario 6 - Pod Cannot Reach Another Pod

Check

- Service
- DNS
- Network Policy
- Labels
- Selectors

Commands

```bash
kubectl get svc

kubectl get endpoints
```

---

# Scenario 7 - Service Has No Endpoints

Symptoms

```text
Service

↓

No Endpoints
```

---

Possible Causes

- Wrong Labels
- Selector Mismatch
- Pods Not Ready

---

# Scenario 8 - DNS Resolution Failure

Symptoms

```text
nslookup payment-service

↓

Failed
```

---

Investigation

Check

```bash
kubectl get pods -n kube-system
```

Verify

- CoreDNS
- Service
- Network

---

# Scenario 9 - Ingress Returns 404

Check

- Ingress Rules
- Host
- Path
- Backend Service
- ALB Controller

---

# Scenario 10 - 503 Errors After Deployment

Verify

- Readiness Probe
- Service Endpoints
- Deployment Status
- ALB Target Health

---

# Scenario 11 - Readiness Probe Failing

Possible Causes

- Wrong Port
- Slow Startup
- Wrong Path
- Application Failure

---

# Scenario 12 - Liveness Probe Failing

Symptoms

```text
Pod

↓

Restart

↓

Restart

↓

Restart
```

---

Resolution

- Increase initialDelaySeconds.
- Fix health endpoint.
- Verify application startup.

---

# Scenario 13 - Deployment Stuck

Check

```bash
kubectl rollout status deployment
```

Review

- ReplicaSet
- Events
- Pods

---

# Scenario 14 - Rollout Failed

Resolution

```bash
kubectl rollout undo deployment
```

Restore previous revision.

---

# Scenario 15 - PVC Pending

Check

- StorageClass
- PV
- CSI Driver

---

# Scenario 16 - EBS Volume Not Attached

Verify

- EBS CSI Driver
- IAM Permissions
- Availability Zone

---

# Scenario 17 - HPA Not Scaling

Check

- Metrics Server
- CPU Requests
- HPA Events

---

# Scenario 18 - Cluster Autoscaler Not Working

Review

- Pending Pods
- IAM Permissions
- Auto Scaling Group
- Cluster Autoscaler Logs

---

# Scenario 19 - High CPU Usage

Investigate

- Prometheus
- Grafana
- Top Pods
- Application Metrics

---

# Scenario 20 - High Memory Usage

Review

- Heap
- Cache
- Memory Leak
- Resource Limits

---

# Scenario 21 - High API Latency

Check

- Database
- Redis
- Network
- External APIs

---

# Scenario 22 - API Server Slow

Review

- etcd
- API Requests
- Cluster Size
- Controller Logs

---

# Scenario 23 - etcd Full

Symptoms

```text
API Server

↓

Slow

↓

Timeouts
```

Resolution

- Compact etcd.
- Defragment database.
- Monitor disk usage.

---

# Scenario 24 - CoreDNS Crash

Check

- Memory
- ConfigMap
- Upstream DNS
- Network

---

# Scenario 25 - Worker Node Disk Full

Investigate

- Images
- Logs
- EmptyDir Volumes

Cleanup

Unused images

Container logs

Old files

---

# Scenario 26 - Certificate Expired

Review

- API Server
- kubelet
- Client Certificates

Rotate certificates.

---

# Scenario 27 - RBAC Permission Denied

Symptoms

```text
Forbidden
```

Check

- Role
- RoleBinding
- Service Account

---

# Scenario 28 - IRSA Not Working (Amazon EKS)

Review

- OIDC Provider
- IAM Role
- Trust Policy
- Service Account Annotation

---

# Scenario 29 - Pods Cannot Access Amazon S3

Verify

- IAM Role
- IRSA
- Bucket Policy
- VPC Endpoint

---

# Scenario 30 - Node Under Memory Pressure

Scheduler begins evicting

BestEffort Pods first.

Investigate

- Running Workloads
- Requests
- Limits

---

# Scenario 31 - Deployment Rollback Failed

Check

- Revision History
- ReplicaSets
- Image Availability

---

# Scenario 32 - StatefulSet Pod Not Starting

Review

- PVC
- Headless Service
- Storage

---

# Scenario 33 - DaemonSet Missing on Node

Verify

- Node Selector
- Taints
- Node Labels

---

# Scenario 34 - Job Never Completes

Review

- Logs
- Exit Codes
- Restart Policy

---

# Scenario 35 - CronJob Not Running

Verify

- Schedule
- Time Zone
- Controller

---

# Scenario 36 - Secret Not Found

Check

- Namespace
- Secret Name
- Mount Path

---

# Scenario 37 - ConfigMap Changes Not Applied

Many applications require

Pod restart

after ConfigMap updates.

---

# Scenario 38 - Network Policy Blocking Traffic

Review

- Ingress Rules
- Egress Rules
- Namespace Labels

---

# Scenario 39 - Pods Scheduled on Wrong Nodes

Investigate

- Affinity
- Node Selector
- Taints

---

# Scenario 40 - Container Starts Then Immediately Exits

Review

- Entry Point
- Command
- Application Logs

---

# Scenario 41 - ALB Not Created (Amazon EKS)

Check

- AWS Load Balancer Controller
- IAM Permissions
- IngressClass
- Subnet Tags

---

# Scenario 42 - Worker Nodes Cannot Join Cluster

Review

- Bootstrap
- IAM Role
- Security Groups
- Cluster Endpoint

---

# Scenario 43 - Excessive Pod Restarts

Investigate

- Application Stability
- Resource Limits
- Health Probes

---

# Scenario 44 - Metrics Missing

Check

- Prometheus Targets
- ServiceMonitor
- Exporters

---

# Scenario 45 - Logs Missing

Verify

- Fluent Bit
- Elasticsearch
- Log Rotation

---

# Scenario 46 - High Network Latency

Investigate

- CNI Plugin
- Node Placement
- Cross-AZ Traffic

---

# Scenario 47 - Pods Stuck in Terminating

Check

- Finalizers
- Volume Detach
- Grace Period

---

# Scenario 48 - Namespace Stuck in Terminating

Investigate

- Finalizers
- Remaining Resources
- CRDs

---

# Scenario 49 - Production Deployment Caused Outage

Response

```text
Rollback

↓

Validate

↓

Monitor

↓

RCA

↓

Prevent Recurrence
```

---

# Scenario 50 - Complete Cluster Failure

Recovery Plan

```text
Restore Control Plane

↓

Recover Worker Nodes

↓

Restore etcd

↓

Validate Storage

↓

Deploy Applications

↓

Verify Monitoring

↓

Production
```

---

# Enterprise Troubleshooting Checklist

Always verify

✓ Nodes

✓ Pods

✓ Services

✓ Ingress

✓ DNS

✓ Storage

✓ RBAC

✓ Metrics

✓ Logs

✓ Events

✓ Networking

✓ Cloud Resources

---

# Best Practices

- Start with business impact before technical details.
- Collect logs, metrics, and events before making changes.
- Validate fixes in a controlled manner.
- Keep rollback plans ready for deployments.
- Use Prometheus and Grafana for performance analysis.
- Centralize logs with ELK.
- Document root cause and preventive actions.
- Automate repetitive recovery tasks where possible.

---

# Common Mistakes

- Restarting Pods without understanding the root cause.
- Ignoring Kubernetes Events.
- Troubleshooting only the application and not the infrastructure.
- Skipping rollback when production is impacted.
- Making multiple changes simultaneously.
- Ignoring cloud-specific integrations (ALB, EBS, IAM).
- Closing incidents without validation.

---

# Interview Questions

## Basic

- How do you troubleshoot a Pod stuck in Pending?
- What causes CrashLoopBackOff?
- How do you investigate ImagePullBackOff?

## Intermediate

- Explain your troubleshooting process for DNS failures.
- How do you diagnose HPA not scaling?
- What would you check if an Ingress returns 503?

## Advanced

- Design a troubleshooting runbook for a production Amazon EKS cluster covering Pods, Nodes, Services, Storage, Networking, Security, and AWS integrations.
- Explain your end-to-end approach to diagnosing a complete Kubernetes outage affecting multiple microservices, including metrics, logs, events, networking, and recovery planning.
- A production Amazon EKS platform experiences Pending Pods, CrashLoopBackOff containers, DNS failures, HPA not scaling, ALB health check failures, and increasing API latency after a deployment. Walk through your complete investigation, recovery strategy, rollback decision, and long-term preventive improvements.

---

# Chapter 9 - Amazon EKS (Elastic Kubernetes Service) Deep Dive

Managing Kubernetes clusters manually is complex.

Platform teams are responsible for

- Control Plane Availability
- Upgrades
- Security
- Scaling
- Monitoring
- Disaster Recovery

AWS simplifies Kubernetes management through **Amazon Elastic Kubernetes Service (Amazon EKS)**.

Amazon EKS is a fully managed Kubernetes service that operates the Kubernetes control plane while allowing customers to focus on running applications.

---

# What is Amazon EKS?

Amazon EKS is a managed Kubernetes service provided by AWS.

AWS manages

- Kubernetes Control Plane
- API Server
- etcd
- Scheduler
- Controller Manager
- High Availability

Customers manage

- Worker Nodes
- Applications
- Networking
- Security
- CI/CD

---

# Why Amazon EKS?

Without EKS

```text
Install Kubernetes

↓

Configure Control Plane

↓

Manage etcd

↓

Upgrade Cluster

↓

Monitor Cluster

↓

Recover Failures
```

High operational overhead.

---

With EKS

```text
AWS

↓

Managed Control Plane

↓

Customer

↓

Worker Nodes

↓

Applications
```

Cluster management becomes significantly simpler.

---

# Amazon EKS Architecture

```text
Users

↓

Amazon Route53

↓

Application Load Balancer

↓

Amazon EKS

↓

Managed Node Groups

↓

Pods

↓

Amazon RDS
```

---

# EKS Shared Responsibility Model

AWS manages

- Control Plane
- API Server
- etcd
- Scheduler
- Controller Manager
- High Availability

Customer manages

- Worker Nodes
- Pods
- IAM
- Applications
- Storage
- Networking Policies

---

# EKS Control Plane

The EKS control plane runs across

multiple Availability Zones.

Architecture

```text
AWS Managed

↓

API Server

↓

Scheduler

↓

Controller Manager

↓

etcd
```

Highly available by default.

---

# Worker Nodes

Applications run on Worker Nodes.

Options include

- Managed Node Groups
- Self-Managed Nodes
- AWS Fargate

---

# Managed Node Groups

AWS automates

- Node Provisioning
- Node Updates
- Health Monitoring
- Auto Scaling Integration

Recommended for most workloads.

---

# Self-Managed Nodes

Customers manage

- EC2 Instances
- AMIs
- Upgrades
- Scaling

Provides greater flexibility,

but higher operational effort.

---

# AWS Fargate

Run Pods

without managing EC2 instances.

Architecture

```text
Pod

↓

AWS Fargate

↓

AWS Infrastructure
```

Ideal for

- Small workloads
- Event-driven applications
- Platform simplification

---

# EKS Networking

Amazon EKS uses

Amazon VPC networking.

Architecture

```text
VPC

├── Public Subnets

│     ALB

│

└── Private Subnets

      Worker Nodes

      Pods
```

Pods receive VPC IP addresses.

---

# Amazon VPC CNI

Amazon EKS commonly uses

Amazon VPC CNI.

Benefits

- Native VPC Networking
- Security Groups
- High Performance

Pods appear as native VPC resources.

---

# Pod Networking

```text
Pod

↓

VPC IP Address

↓

Security Group

↓

AWS Network
```

Applications integrate naturally with AWS networking.

---

# IAM Authentication

Amazon EKS integrates Kubernetes

with AWS IAM.

Workflow

```text
IAM User

↓

kubectl

↓

API Server
```

AWS IAM authenticates users.

---

# IAM Roles for Service Accounts (IRSA)

Applications access AWS services securely.

Architecture

```text
Pod

↓

Service Account

↓

IAM Role

↓

Amazon S3

↓

Secrets Manager

↓

DynamoDB
```

Static AWS credentials are unnecessary.

---

# EKS Storage

Persistent storage options

- Amazon EBS
- Amazon EFS
- Amazon FSx

Provisioned using

CSI Drivers.

---

# EBS Storage Example

```text
Pod

↓

PVC

↓

StorageClass

↓

EBS CSI Driver

↓

Amazon EBS
```

Suitable for block storage workloads.

---

# EFS Storage Example

```text
Multiple Pods

↓

PVC

↓

EFS CSI Driver

↓

Amazon EFS
```

Ideal for shared storage.

---

# AWS Load Balancer Controller

Ingress resources create

Application Load Balancers automatically.

Architecture

```text
Ingress

↓

AWS Load Balancer Controller

↓

Application Load Balancer

↓

Pods
```

---

# Cluster Autoscaler

Cluster Autoscaler

adds or removes Worker Nodes.

Workflow

```text
Pending Pods

↓

Cluster Autoscaler

↓

New EC2 Nodes

↓

Pods Scheduled
```

---

# Horizontal Pod Autoscaler

HPA increases

Pod replicas.

```text
CPU > 80%

↓

HPA

↓

More Pods
```

Works together with

Cluster Autoscaler.

---

# EKS Upgrade Process

Recommended workflow

```text
Control Plane

↓

Managed Node Groups

↓

Add-ons

↓

Applications
```

Always upgrade

the Control Plane first.

---

# EKS Add-ons

Common managed add-ons

- CoreDNS
- kube-proxy
- VPC CNI
- EBS CSI Driver

AWS manages lifecycle updates.

---

# Monitoring Amazon EKS

Typical stack

```text
Applications

↓

Prometheus

↓

Grafana

↓

CloudWatch

↓

Alertmanager
```

---

# Logging

Centralized logging

```text
Pods

↓

Fluent Bit

↓

CloudWatch Logs

↓

Elasticsearch

↓

Kibana
```

---

# Security

Production EKS clusters use

- IAM
- IRSA
- RBAC
- Network Policies
- Security Groups
- KMS Encryption

---

# High Availability

Amazon EKS provides

```text
Multi-AZ

↓

Managed Control Plane

↓

Managed Node Groups

↓

Auto Scaling
```

Applications remain available

during infrastructure failures.

---

# Enterprise CI/CD Pipeline

```text
Developer

↓

GitHub

↓

GitHub Actions

↓

Docker Build

↓

Amazon ECR

↓

Amazon EKS

↓

Deployment

↓

Prometheus

↓

Grafana
```

---

# Banking Example

```text
Customers

↓

Route53

↓

AWS WAF

↓

ALB

↓

Amazon EKS

↓

Payment Pods

↓

Aurora

↓

CloudWatch

↓

Grafana
```

Supports

high availability

and enterprise security.

---

# Amazon EKS Best Practices

- Use Managed Node Groups unless custom requirements exist.
- Deploy Worker Nodes in private subnets.
- Use IRSA instead of AWS access keys.
- Enable Cluster Autoscaler.
- Use HPA for stateless workloads.
- Deploy across multiple Availability Zones.
- Use AWS Load Balancer Controller for Ingress.
- Encrypt Secrets using AWS KMS.
- Monitor with Prometheus and CloudWatch.
- Keep clusters updated.

---

# Common Mistakes

- Deploying Worker Nodes in public subnets.
- Using IAM users inside Pods.
- Ignoring cluster upgrades.
- Running production without Cluster Autoscaler.
- Exposing the Kubernetes API publicly.
- Not monitoring CoreDNS or VPC CNI.
- Storing AWS credentials inside containers.

---

# Amazon EKS vs Self-Managed Kubernetes

| Amazon EKS | Self-Managed Kubernetes |
|-------------|-------------------------|
| Managed Control Plane | Customer Managed |
| Multi-AZ by Default | Manual HA Setup |
| Automatic Upgrades | Manual Upgrades |
| AWS Integration | Manual Integration |
| Lower Operational Overhead | Higher Operational Overhead |

---

# Amazon EKS vs ECS

| Amazon EKS | Amazon ECS |
|-------------|------------|
| Kubernetes Standard | AWS Native |
| More Flexible | Simpler |
| Large Kubernetes Ecosystem | AWS Managed Simplicity |
| Portable | AWS Focused |
| Higher Learning Curve | Easier to Operate |

---

# Benefits

- Managed Kubernetes
- High Availability
- AWS Integration
- Automatic Scaling
- Enterprise Security
- Simplified Operations
- Cloud-Native Platform
- Production Ready

---

# Interview Questions

## Basic

- What is Amazon EKS?
- What does AWS manage in EKS?
- Managed Node Group vs Self-Managed Node Group.
- What is IRSA?

## Intermediate

- Explain Amazon VPC CNI.
- EKS vs ECS.
- How does AWS Load Balancer Controller work?
- Explain the EKS upgrade process.
- How do Pods access AWS services securely?

## Advanced

- Design a highly available Amazon EKS platform for a banking application using Managed Node Groups, IRSA, AWS Load Balancer Controller, Prometheus, Grafana, Amazon EBS, and Aurora PostgreSQL.
- Explain the complete deployment flow from GitHub Actions to Amazon EKS, including image storage in Amazon ECR, rolling deployments, monitoring, autoscaling, and rollback.
- A production Amazon EKS cluster experiences worker node failures, increasing application traffic, and storage provisioning issues during a peak business event. Explain how Managed Node Groups, Cluster Autoscaler, HPA, CSI Drivers, AWS networking, and monitoring tools work together to maintain application availability and recover from failures.

---

# Chapter 10 - Kubernetes Production Best Practices

Building a Kubernetes cluster is relatively straightforward.

Operating it successfully in production is the real challenge.

Enterprise Kubernetes platforms are designed around

- Reliability
- Security
- Scalability
- Performance
- Cost Optimization
- Disaster Recovery
- Operational Excellence

This chapter summarizes the practices followed by organizations running Kubernetes at scale.

---

# Production Architecture

A typical enterprise Kubernetes platform looks like

```text
Users

↓

Route53

↓

AWS WAF

↓

Application Load Balancer

↓

Ingress Controller

↓

Amazon EKS

↓

Managed Node Groups

↓

Pods

↓

Amazon RDS

↓

Amazon EFS

↓

Prometheus

↓

Grafana

↓

ELK Stack
```

Every component is deployed with high availability.

---

# High Availability

Production clusters should never rely on a single point of failure.

Recommended architecture

```text
Availability Zone A

↓

Worker Nodes

────────────

Availability Zone B

↓

Worker Nodes

────────────

Availability Zone C

↓

Worker Nodes
```

Pods are distributed across all Availability Zones.

---

# Node Groups

Separate workloads by using dedicated node groups.

Example

```text
Production Node Group

↓

Critical Applications

────────────

Monitoring Node Group

↓

Prometheus

↓

Grafana

────────────

Batch Node Group

↓

CronJobs

↓

Reporting
```

This improves isolation and scheduling.

---

# Namespace Strategy

Avoid deploying everything into the

```text
default
```

namespace.

Recommended structure

```text
production

development

staging

monitoring

logging

ingress

kube-system
```

Namespaces simplify

- Resource Isolation
- RBAC
- Resource Quotas

---

# Resource Requests & Limits

Every production workload should define

```text
CPU Request

Memory Request

CPU Limit

Memory Limit
```

Benefits

- Better Scheduling
- Stable Performance
- Prevent Resource Starvation

---

# Health Probes

Always configure

- Liveness Probe
- Readiness Probe
- Startup Probe

Workflow

```text
Application

↓

Health Check

↓

Ready

↓

Receive Traffic
```

Pods should never receive production traffic before they are ready.

---

# Rolling Deployments

Preferred deployment strategy

```text
Version 1

↓

Rolling Update

↓

Version 2
```

Avoid downtime during deployments.

---

# Rollback Strategy

Every deployment must support rollback.

Workflow

```text
Deploy

↓

Validate

↓

Issue Detected

↓

Rollback

↓

Recovery
```

Recovery should be automated whenever possible.

---

# Autoscaling Strategy

Combine

- Horizontal Pod Autoscaler
- Cluster Autoscaler

Architecture

```text
Traffic Increase

↓

HPA

↓

More Pods

↓

Cluster Autoscaler

↓

More Nodes
```

Applications scale automatically.

---

# Pod Disruption Budgets

Prevent excessive Pod disruption during

- Node Maintenance
- Cluster Upgrades

Example

```text
10 Pods

↓

Minimum Available

↓

8 Pods
```

The application remains available.

---

# Affinity Rules

Use

Pod Anti-Affinity

for critical workloads.

Example

```text
Replica 1

↓

Node A

────────────

Replica 2

↓

Node B

────────────

Replica 3

↓

Node C
```

This improves fault tolerance.

---

# Secure Workloads

Containers should

- Run as non-root
- Use read-only filesystems
- Drop unnecessary Linux capabilities
- Avoid privileged mode

Follow the principle of least privilege.

---

# Secret Management

Store sensitive data using

- Kubernetes Secrets
- AWS Secrets Manager
- External Secrets Operator (if adopted)

Never

- Commit credentials to Git
- Hardcode passwords
- Store AWS access keys in Pods

---

# Storage Best Practices

Use

Amazon EBS

for

```text
Databases

Stateful Applications
```

Use

Amazon EFS

for

```text
Shared Storage

CMS

ML Workloads
```

Avoid

```text
hostPath
```

in production.

---

# Networking

Use

```text
Internet

↓

AWS WAF

↓

ALB

↓

Ingress

↓

ClusterIP

↓

Pods
```

Avoid exposing backend services directly.

---

# Logging

Centralize all logs.

Architecture

```text
Pods

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana
```

Logs should remain available even if Pods are deleted.

---

# Monitoring

Monitor

- Cluster Health
- Node Health
- Pod Health
- Application Metrics
- Business Metrics

Stack

```text
Prometheus

↓

Grafana

↓

Alertmanager
```

---

# Alerting

Create alerts for

- Pod Restarts
- High CPU
- High Memory
- Node Not Ready
- PVC Full
- Certificate Expiration
- High API Latency

Avoid excessive alerts that create noise.

---

# Backup Strategy

Regularly back up

- etcd (self-managed clusters)
- Persistent Volumes
- Databases
- Configuration
- Secrets

Verify backup restoration procedures periodically.

---

# Disaster Recovery

Maintain

- Multi-AZ deployment
- Backup strategy
- Infrastructure as Code
- Automated recovery procedures

Recovery objectives should be documented.

---

# CI/CD Best Practices

Pipeline example

```text
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

Amazon EKS

↓

Smoke Tests

↓

Production
```

Every deployment should be repeatable and automated.

---

# Cost Optimization

Optimize costs using

- Cluster Autoscaler
- Right-Sized Nodes
- Spot Instances (where appropriate)
- Image Cleanup
- Resource Requests
- Node Consolidation

Monitor unused resources regularly.

---

# Upgrade Strategy

Recommended sequence

```text
Control Plane

↓

Node Groups

↓

Add-ons

↓

Applications
```

Test upgrades in non-production environments first.

---

# Enterprise Security

Enable

- RBAC
- Network Policies
- IRSA
- Secret Encryption
- Image Scanning
- Runtime Monitoring
- Audit Logging

Security should be integrated into every stage of the platform.

---

# Operational Excellence

Successful platform teams focus on

- Automation
- Documentation
- Monitoring
- Incident Management
- Continuous Improvement

Every production incident should result in

- Root Cause Analysis
- Preventive Actions
- Knowledge Sharing

---

# Enterprise Architecture Example

```text
Users

↓

Route53

↓

AWS WAF

↓

ALB

↓

Ingress

↓

Amazon EKS

↓

Deployments

↓

Services

↓

Pods

↓

Amazon Aurora

↓

Prometheus

↓

Grafana

↓

ELK
```

This architecture supports

- High Availability
- Scalability
- Security
- Observability

---

# Kubernetes Production Checklist

Before deploying to production, verify

✓ Resource Requests & Limits

✓ Liveness, Readiness & Startup Probes

✓ RBAC

✓ Network Policies

✓ Secrets

✓ TLS

✓ Monitoring

✓ Logging

✓ Autoscaling

✓ Rollback Plan

✓ Backup Strategy

✓ Disaster Recovery

✓ CI/CD Validation

---

# Golden Rules

1. Never deploy directly to production without testing.
2. Never expose sensitive credentials inside container images.
3. Always define resource requests and limits.
4. Monitor applications, not just infrastructure.
5. Keep clusters updated.
6. Use Infrastructure as Code.
7. Automate repetitive operational tasks.
8. Design for failure from the beginning.

---

# Common Production Mistakes

- Running workloads without health probes.
- Ignoring resource requests.
- Using the default namespace for everything.
- Giving cluster-admin access to all users.
- Deploying without rollback plans.
- Not monitoring business metrics.
- Forgetting backup validation.
- Allowing configuration drift.

---

# Interview Questions

## Basic

- What are Kubernetes production best practices?
- Why are health probes important?
- Why should workloads use resource limits?

## Intermediate

- Explain a production deployment strategy for Kubernetes.
- How do you design a highly available Kubernetes cluster?
- How do you secure a production EKS platform?

## Advanced

- Design a production-ready Kubernetes platform for an enterprise financial application, covering high availability, security, autoscaling, monitoring, disaster recovery, cost optimization, and CI/CD integration.
- Explain your complete production readiness checklist before deploying a new microservice to Amazon EKS.
- A global enterprise is migrating hundreds of applications to Amazon EKS. Describe the operational standards, deployment strategies, monitoring architecture, security controls, backup policies, upgrade process, and disaster recovery practices you would establish to ensure long-term platform reliability.

---

# Chapter 11 - Kubernetes Enterprise Interview Handbook

Senior Kubernetes interviews rarely focus on definitions.

Instead of asking

> "What is a Pod?"

Interviewers ask

> "Your production cluster is down. Walk me through your troubleshooting process."

or

> "Design a Kubernetes platform for 10 million users."

This chapter contains enterprise-level interview questions covering

- Kubernetes Fundamentals
- Amazon EKS
- Architecture
- Security
- Networking
- Storage
- Scheduling
- CI/CD
- Monitoring
- Production Troubleshooting
- System Design
- Leadership

---

# Section 1 - Kubernetes Fundamentals

## Basic Questions

1. What is Kubernetes?
2. Why do organizations use Kubernetes?
3. Explain Kubernetes architecture.
4. What are the components of the Control Plane?
5. What is a Worker Node?
6. What is a Pod?
7. Pod vs Container.
8. Pod vs Deployment.
9. Deployment vs ReplicaSet.
10. What is a Namespace?

---

## Intermediate Questions

11. Explain the Kubernetes Scheduler.
12. What is etcd?
13. Explain kubelet.
14. What is kube-proxy?
15. How does Kubernetes achieve self-healing?
16. Explain Rolling Updates.
17. Explain Rollbacks.
18. How does Kubernetes maintain the desired state?
19. What are Labels and Selectors?
20. What are Finalizers?

---

# Section 2 - Kubernetes Networking

21. Explain Kubernetes networking.
22. What is ClusterIP?
23. What is NodePort?
24. What is LoadBalancer Service?
25. What is an Ingress?
26. Ingress vs LoadBalancer.
27. Explain CoreDNS.
28. How does Service Discovery work?
29. What is a Headless Service?
30. What is kube-proxy's role in networking?

---

## Advanced Networking

31. Explain the complete request flow from a browser to a Pod.
32. Design networking for 100 microservices.
33. Explain AWS Load Balancer Controller.
34. What happens if CoreDNS fails?
35. How do Network Policies affect traffic?

---

# Section 3 - Storage

36. What is a Persistent Volume?
37. What is a Persistent Volume Claim?
38. Explain StorageClass.
39. Dynamic Provisioning.
40. Amazon EBS vs Amazon EFS.

---

## Advanced Storage

41. Design storage for PostgreSQL on EKS.
42. How does the EBS CSI Driver work?
43. Explain StatefulSet storage.
44. What happens if a PVC cannot bind?
45. How would you migrate storage between clusters?

---

# Section 4 - Scheduling & Autoscaling

46. Explain Scheduling.
47. Requests vs Limits.
48. QoS Classes.
49. Node Affinity.
50. Pod Affinity.
51. Pod Anti-Affinity.
52. Taints vs Tolerations.
53. HPA.
54. VPA.
55. Cluster Autoscaler.

---

## Advanced Scheduling

56. How does the Scheduler choose a node?
57. Explain resource overcommitment.
58. Design autoscaling for Black Friday traffic.
59. Explain Pod Disruption Budgets.
60. PriorityClass vs QoS.

---

# Section 5 - Security

61. Explain RBAC.
62. Role vs ClusterRole.
63. RoleBinding vs ClusterRoleBinding.
64. What is a Service Account?
65. Explain IRSA.
66. What are Network Policies?
67. Explain Pod Security Standards.
68. SecurityContext.
69. How are Secrets protected?
70. Explain Kubernetes authentication.

---

## Advanced Security

71. Design a secure EKS cluster.
72. How would you secure the Kubernetes API Server?
73. Explain Zero Trust networking.
74. How do Pods securely access AWS services?
75. How do you protect etcd?

---

# Section 6 - Monitoring

76. Explain Prometheus.
77. Explain Grafana.
78. Metrics Server vs Prometheus.
79. Explain Alertmanager.
80. Explain Fluent Bit.
81. Explain Elasticsearch.
82. Explain Kibana.
83. What are Kubernetes Events?
84. Golden Signals.
85. What metrics should be monitored?

---

## Advanced Monitoring

86. Design monitoring for 500 microservices.
87. Explain centralized logging.
88. How would you troubleshoot latency using Prometheus?
89. Explain alert strategy.
90. Design an observability platform.

---

# Section 7 - Amazon EKS

91. What is Amazon EKS?
92. What does AWS manage?
93. Managed Node Groups.
94. Self-Managed Nodes.
95. AWS Fargate.
96. Amazon VPC CNI.
97. AWS Load Balancer Controller.
98. EKS Upgrade Process.
99. EKS Add-ons.
100. EKS Best Practices.

---

## Advanced EKS

101. EKS vs ECS.
102. Explain IRSA workflow.
103. Explain VPC networking in EKS.
104. Design production EKS.
105. Explain cluster upgrades.

---

# Section 8 - Troubleshooting

106. Pod Pending.
107. CrashLoopBackOff.
108. OOMKilled.
109. ImagePullBackOff.
110. Node Not Ready.
111. Service has no endpoints.
112. DNS failure.
113. Ingress returns 404.
114. HPA not scaling.
115. PVC Pending.

---

## Advanced Troubleshooting

116. Worker Nodes cannot join EKS.
117. ALB not created.
118. API latency increased.
119. Pods stuck terminating.
120. Complete cluster failure recovery.

---

# Section 9 - CI/CD

121. Explain Kubernetes deployment pipeline.
122. GitHub Actions with Kubernetes.
123. Amazon ECR integration.
124. Rolling deployment strategy.
125. Blue-Green deployment.
126. Canary deployment.
127. Helm deployment.
128. GitOps.
129. ArgoCD.
130. Rollback strategy.

---

## Advanced CI/CD

131. Design an enterprise deployment pipeline.
132. Explain GitOps workflow.
133. How do you validate deployments?
134. How do you minimize deployment risk?
135. How would you deploy hundreds of microservices?

---

# Section 10 - Production Architecture

136. Design an enterprise Kubernetes platform.
137. Design a banking platform on Amazon EKS.
138. Design a multi-region Kubernetes platform.
139. Design a SaaS platform.
140. Design a zero-downtime deployment architecture.

---

# Section 11 - Scenario-Based Questions

141. Production deployment caused 503 errors.
142. Pods restart every few minutes.
143. CPU usage suddenly reaches 100%.
144. Memory leaks affect production.
145. Node failure during peak traffic.
146. DNS outage impacts applications.
147. Storage provisioning fails.
148. Security breach inside the cluster.
149. AWS Region becomes unavailable.
150. Monitoring system stops sending alerts.

---

# Section 12 - Managerial Round

151. Explain your Kubernetes architecture.
152. Walk through your production deployment process.
153. Explain your monitoring strategy.
154. Describe a Kubernetes incident you resolved.
155. How do you upgrade production clusters?
156. Explain your disaster recovery plan.
157. How do you secure production workloads?
158. How do you reduce Kubernetes costs?
159. How do you mentor junior engineers on Kubernetes?
160. Why should we hire you?

---

# Enterprise Answer Framework

For every Kubernetes interview answer,

follow this sequence.

```text
Understand Requirements

↓

Architecture

↓

Security

↓

Networking

↓

Storage

↓

Scalability

↓

Monitoring

↓

Disaster Recovery

↓

Cost Optimization

↓

Trade-offs
```

---

# Enterprise Troubleshooting Framework

```text
Alert

↓

Business Impact

↓

Metrics

↓

Logs

↓

Events

↓

Root Cause

↓

Recovery

↓

Validation

↓

Postmortem
```

---

# Production Readiness Checklist

Before any production deployment verify

✓ Multi-AZ Architecture

✓ Resource Requests & Limits

✓ Health Probes

✓ Autoscaling

✓ Monitoring

✓ Logging

✓ RBAC

✓ Network Policies

✓ Backup Strategy

✓ Rollback Plan

✓ Disaster Recovery

✓ CI/CD Validation

---

# Common Interview Mistakes

- Answering without clarifying requirements.
- Focusing only on Kubernetes resources instead of business impact.
- Ignoring security.
- Forgetting monitoring and logging.
- Missing rollback strategies.
- Not discussing disaster recovery.
- Ignoring cost optimization.
- Giving theoretical answers without production examples.

---

# Final Advice

Enterprise interviewers expect engineers who can

- Design scalable platforms.
- Troubleshoot production incidents.
- Secure Kubernetes environments.
- Automate deployments.
- Monitor systems effectively.
- Communicate clearly during outages.
- Balance reliability, performance, security, and cost.

A strong Kubernetes engineer demonstrates not only technical expertise but also operational excellence, structured problem solving, and ownership throughout the software delivery lifecycle.

---

# File Completed

**File Name:** `112-Kubernetes-Enterprise-Handbook.md`

This handbook now includes:

- ✅ Kubernetes Architecture
- ✅ Pods, Deployments & ReplicaSets
- ✅ Services & Networking
- ✅ ConfigMaps, Secrets & Storage
- ✅ Scheduling & Autoscaling
- ✅ Security (RBAC, IRSA, Network Policies)
- ✅ Monitoring & Logging
- ✅ 50+ Production Troubleshooting Scenarios
- ✅ Amazon EKS Deep Dive
- ✅ Production Best Practices
- ✅ 160 Enterprise Interview Questions
- ✅ Architecture Frameworks
- ✅ Production Checklists
- ✅ Enterprise Troubleshooting Methodologies
