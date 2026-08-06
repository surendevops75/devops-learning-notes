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

