# ArgoCD Enterprise Handbook

# Chapter 1 - GitOps Fundamentals & ArgoCD Architecture

Modern Kubernetes deployments require applications to be

- Reliable
- Repeatable
- Secure
- Automated
- Auditable

Traditional deployment methods rely on

- Manual kubectl commands
- Shell Scripts
- Direct Cluster Access

These approaches introduce

- Configuration Drift
- Human Errors
- Difficult Rollbacks
- Limited Auditability

GitOps solves these problems by making **Git the single source of truth**.

ArgoCD is one of the most widely used GitOps tools for Kubernetes.

---

# What is GitOps?

GitOps is an operational model where

Git repositories define

the desired state

of Kubernetes infrastructure

and applications.

Instead of manually changing the cluster,

changes are committed to Git.

Git becomes

the source of truth.

---

# Traditional Deployment

Without GitOps

```text
Developer

↓

kubectl apply

↓

Kubernetes Cluster

↓

Manual Changes
```

Problems

- Manual Errors
- No Version Control
- Configuration Drift
- Difficult Rollback

---

# GitOps Deployment

With GitOps

```text
Developer

↓

Git Repository

↓

ArgoCD

↓

Kubernetes Cluster
```

Everything

is deployed

from Git.

---

# What is ArgoCD?

ArgoCD is a

declarative

GitOps Continuous Delivery tool

for Kubernetes.

It continuously compares

the desired state

stored in Git

with

the actual state

inside the Kubernetes cluster.

If differences exist,

ArgoCD synchronizes

the cluster.

---

# Why ArgoCD?

Without ArgoCD

```text
Developer

↓

Jenkins

↓

kubectl apply

↓

Cluster
```

Deployments

depend

on external scripts.

---

With ArgoCD

```text
Developer

↓

Git

↓

ArgoCD

↓

Cluster
```

Deployment

becomes

automatic,

consistent,

and auditable.

---

# GitOps Principle

Git contains

everything required

to deploy.

Examples

- Kubernetes Manifests
- Helm Charts
- Kustomize Files
- Configuration

Cluster changes

must originate

from Git.

---

# Enterprise GitOps Architecture

```text
Developer

↓

GitHub

↓

Pull Request

↓

Merge

↓

ArgoCD

↓

Amazon EKS

↓

Pods

↓

Customers
```

This architecture

eliminates

manual deployments.

---

# Push vs Pull Deployment

Traditional CI/CD

```text
Jenkins

↓

Push

↓

Cluster
```

GitOps

```text
Git

↓

ArgoCD Pull

↓

Cluster
```

ArgoCD

pulls

changes

instead of

CI pushing them.

---

# ArgoCD Components

ArgoCD consists of

- API Server
- Repository Server
- Application Controller
- Redis
- Web UI
- CLI

Each component

has a specific responsibility.

---

# ArgoCD Architecture

```text
Git Repository

↓

Repository Server

↓

Application Controller

↓

API Server

↓

Kubernetes Cluster
```

---

# API Server

The API Server

provides

- REST API
- CLI Access
- Web UI
- Authentication
- RBAC

Users interact

with ArgoCD

through

the API Server.

---

# Repository Server

The Repository Server

connects to

Git repositories.

Responsibilities

- Clone Repository
- Read Manifests
- Render Helm Charts
- Process Kustomize

---

# Application Controller

The Application Controller

continuously compares

Git

with

the Kubernetes cluster.

Workflow

```text
Desired State

↓

Compare

↓

Actual State

↓

Sync
```

---

# Redis

Redis

stores

application state

and improves

performance

through caching.

---

# Web UI

The ArgoCD Web UI

provides

- Application Status
- Sync Status
- Health Status
- Deployment History
- Rollback

---

# CLI

The CLI

allows engineers

to

- Login
- Sync Applications
- Rollback
- View Status
- Manage Applications

Useful

for automation.

---

# Desired State

Desired State

exists

inside Git.

Examples

```text
Deployment

Service

Ingress

ConfigMap

Secret

Helm Values
```

---

# Actual State

Actual State

is the

running configuration

inside Kubernetes.

ArgoCD

continuously compares

desired

and actual state.

---

# Drift Detection

Configuration Drift

occurs when

the cluster differs

from Git.

Architecture

```text
Git

↓

Desired State

────────────

Cluster

↓

Modified Resource

↓

Drift Detected
```

ArgoCD

identifies

and reports

the drift.

---

# Automatic Reconciliation

If Auto Sync

is enabled

```text
Git

↓

ArgoCD

↓

Cluster

↓

Correct State Restored
```

Manual changes

are overwritten

to match Git.

---

# Enterprise Deployment Flow

```text
Developer

↓

Git Commit

↓

Pull Request

↓

Merge

↓

ArgoCD

↓

Amazon EKS

↓

Pods
```

Every deployment

originates

from Git.

---

# Banking Example

```text
Developer

↓

Payment Service

↓

GitHub

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

No engineer

runs

manual

kubectl commands

in production.

---

# Benefits

- Git as Source of Truth
- Automatic Synchronization
- Easy Rollback
- Drift Detection
- Declarative Deployments
- Auditability
- Faster Recovery
- Improved Security

---

# Enterprise Best Practices

- Never deploy directly using kubectl.
- Store all manifests in Git.
- Protect production branches.
- Enable Pull Request reviews.
- Separate CI and CD responsibilities.
- Monitor sync status.
- Keep applications declarative.
- Treat Git as the single source of truth.

---

# Common Mistakes

- Editing Kubernetes resources manually.
- Bypassing Git for production changes.
- Mixing CI and GitOps responsibilities.
- Disabling drift detection.
- Ignoring failed syncs.
- Keeping cluster-only configuration.
- Not version-controlling manifests.

---

# Interview Questions

## Basic

- What is GitOps?
- What is ArgoCD?
- Why is Git the source of truth?
- Push vs Pull deployment.
- What is configuration drift?

## Intermediate

- Explain ArgoCD architecture.
- What is the Application Controller?
- What is the Repository Server?
- How does drift detection work?
- What is automatic reconciliation?

## Advanced

- Design an enterprise GitOps platform using GitHub, Jenkins, ArgoCD, Helm, Amazon EKS, Prometheus, and Grafana.
- Explain how ArgoCD continuously synchronizes Kubernetes clusters with Git repositories and prevents configuration drift.
- A financial organization wants to eliminate manual Kubernetes deployments and adopt GitOps. Explain how you would design the Git repository structure, ArgoCD architecture, synchronization strategy, rollback process, security model, and governance controls to ensure secure, auditable, and highly available deployments.

---

# Chapter 2 - Installing ArgoCD & Understanding Core Components

ArgoCD runs inside

a Kubernetes cluster

and continuously monitors

Git repositories

to synchronize

the desired state

with the cluster.

A production installation

should be

- Secure
- Highly Available
- Scalable
- Easy to Maintain

---

# ArgoCD Deployment Architecture

```text
Developer

↓

GitHub

↓

ArgoCD

↓

Amazon EKS

↓

Applications
```

ArgoCD

acts as

the deployment engine

for Kubernetes.

---

# Where Does ArgoCD Run?

ArgoCD

runs

inside Kubernetes

as a set of Pods.

Architecture

```text
Amazon EKS

↓

argocd Namespace

↓

ArgoCD Components
```

The entire platform

is deployed

inside

its own namespace.

---

# ArgoCD Namespace

Best practice

is to install

ArgoCD

inside

a dedicated namespace.

Example

```text
Kubernetes Cluster

├── kube-system

├── monitoring

├── ingress-nginx

└── argocd
```

This provides

better isolation.

---

# High-Level Architecture

```text
Git Repository

↓

Repository Server

↓

Application Controller

↓

API Server

↓

Kubernetes API

↓

Cluster Resources
```

Each component

has

a dedicated responsibility.

---

# ArgoCD Components

The core components are

- API Server
- Repository Server
- Application Controller
- Redis
- Dex (Optional)
- Notifications Controller (Optional)
- ApplicationSet Controller (Optional)

---

# API Server

The API Server

is the entry point

for users.

Responsibilities

- Authentication
- Authorization
- Web UI
- CLI
- REST API

Architecture

```text
User

↓

API Server

↓

ArgoCD
```

---

# Repository Server

The Repository Server

communicates with

Git repositories.

Responsibilities

- Clone Repository
- Fetch Updates
- Render Helm Charts
- Process Kustomize
- Generate Kubernetes Manifests

Workflow

```text
Git Repository

↓

Repository Server

↓

Manifest Generation
```

---

# Application Controller

The Application Controller

is the heart

of ArgoCD.

Responsibilities

- Compare Desired State
- Detect Drift
- Synchronize Resources
- Track Health
- Track Sync Status

Workflow

```text
Git

↓

Compare

↓

Cluster

↓

Sync
```

---

# Redis

Redis

stores

cached application data.

Benefits

- Faster UI
- Improved Performance
- Reduced Git Requests

---

# Dex (Optional)

Dex provides

Single Sign-On (SSO)

using

- LDAP
- GitHub
- Google
- Microsoft Entra ID
- OIDC Providers

Enterprise environments

commonly integrate

Dex

with corporate identity systems.

---

# Notifications Controller

This optional component

sends notifications

for events such as

- Successful Sync
- Failed Sync
- Health Degradation
- Rollback

Notification targets

include

- Slack
- Microsoft Teams
- Email
- Webhooks

---

# ApplicationSet Controller

ApplicationSet

automatically creates

multiple ArgoCD Applications.

Useful for

- Multiple Clusters
- Multiple Environments
- Hundreds of Microservices

---

# Component Interaction

```text
Developer

↓

GitHub

↓

Repository Server

↓

Application Controller

↓

API Server

↓

Amazon EKS
```

All components

work together

to maintain

cluster consistency.

---

# Installation Flow

```text
Kubernetes Cluster

↓

Create Namespace

↓

Install ArgoCD

↓

Pods Created

↓

API Available

↓

Applications Managed
```

---

# Production Deployment

Enterprise deployment

typically includes

```text
Load Balancer

↓

ArgoCD API Server

↓

Application Controller

↓

Repository Server

↓

Redis
```

High availability

can be achieved

using multiple replicas.

---

# High Availability Architecture

```text
Load Balancer

↓

API Server (Replica 1)

↓

API Server (Replica 2)

────────────

Application Controller

↓

Multiple Replicas

────────────

Repository Server

↓

Multiple Replicas
```

Single component failures

should not

interrupt deployments.

---

# ArgoCD Data Flow

```text
Git Commit

↓

Repository Server

↓

Manifest Generation

↓

Application Controller

↓

Compare

↓

Synchronize

↓

Amazon EKS
```

---

# Authentication Flow

```text
User

↓

SSO

↓

API Server

↓

RBAC

↓

Application Access
```

Authentication

occurs

before

any operation.

---

# Sync Flow

```text
Git Repository

↓

Desired State

↓

Application Controller

↓

Compare

↓

Cluster

↓

Sync
```

If drift exists,

ArgoCD

restores

the desired state.

---

# Enterprise Deployment

```text
Developer

↓

GitHub

↓

Jenkins

↓

Update Git Manifest

↓

ArgoCD

↓

Amazon EKS
```

Notice that

Jenkins

does **not**

deploy directly.

It only updates Git.

ArgoCD

performs

the deployment.

---

# Banking Example

```text
Developer

↓

Payment Service

↓

GitHub

↓

ArgoCD

↓

Amazon EKS

↓

Production
```

Production changes

always originate

from Git.

---

# Enterprise Best Practices

- Install ArgoCD in a dedicated namespace.
- Use High Availability for production.
- Enable SSO with Dex or an enterprise identity provider.
- Protect API Server access using RBAC.
- Scale Repository Server for large Git repositories.
- Monitor Application Controller performance.
- Separate ArgoCD from application namespaces.
- Use ApplicationSet for large environments.

---

# Common Mistakes

- Installing ArgoCD in the default namespace.
- Running a single replica in production.
- Allowing direct cluster changes.
- Ignoring Repository Server performance.
- Exposing the API Server publicly without authentication.
- Disabling RBAC.
- Running all environments from one unmanaged application.

---

# Interview Questions

## Basic

- Where does ArgoCD run?
- What is the API Server?
- What is the Repository Server?
- What is the Application Controller?
- Why does ArgoCD use Redis?

## Intermediate

- Explain ArgoCD component architecture.
- How does the Application Controller detect drift?
- What is Dex used for?
- What is the ApplicationSet Controller?
- How does ArgoCD authenticate users?

## Advanced

- Design a highly available ArgoCD deployment for Amazon EKS supporting hundreds of microservices, multiple development teams, and enterprise SSO.
- Explain how the API Server, Repository Server, Application Controller, Redis, Dex, and ApplicationSet Controller work together to deliver secure and scalable GitOps deployments.
- A financial organization wants to deploy ArgoCD in production across multiple Kubernetes clusters. Explain how you would design the namespace strategy, high availability architecture, authentication, RBAC, scalability, monitoring, and component interactions to ensure secure and reliable GitOps operations.

---

