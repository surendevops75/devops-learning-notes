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

