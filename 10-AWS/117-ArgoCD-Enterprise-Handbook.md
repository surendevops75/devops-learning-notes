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

# Chapter 3 - ArgoCD Applications, Projects & Git Repository Management

ArgoCD manages Kubernetes deployments using **Applications**.

An Application defines

- What to deploy
- Where to deploy
- Which Git repository to use
- Which Kubernetes cluster to deploy into

For enterprise environments,

Applications are organized into **Projects** to provide security, governance, and multi-team isolation.

---

# What is an ArgoCD Application?

An Application

is the fundamental deployment object

in ArgoCD.

It connects

```text
Git Repository

↓

Kubernetes Cluster
```

An Application continuously synchronizes

the desired state

stored in Git

with

the actual cluster state.

---

# Application Architecture

```text
Git Repository

↓

ArgoCD Application

↓

Amazon EKS

↓

Pods
```

The Application

represents

one deployable workload.

---

# Why Applications?

Without Applications

```text
Git Repository

↓

Manual kubectl

↓

Cluster
```

Problems

- Manual Deployments
- No Ownership
- Difficult Tracking

---

With Applications

```text
Git Repository

↓

Application

↓

Automatic Sync

↓

Cluster
```

Everything

is managed

centrally.

---

# Application Components

Every Application contains

```text
Source

↓

Destination

↓

Project

↓

Sync Policy

↓

Health Status
```

These define

how

the application

is managed.

---

# Source

The Source defines

where

ArgoCD retrieves

the application manifests.

Examples

```text
Git Repository

Helm Repository

OCI Repository
```

---

# Destination

The Destination defines

where

the application

is deployed.

Examples

```text
Amazon EKS Cluster

Namespace
```

Applications

can target

different clusters

and namespaces.

---

# Target Revision

ArgoCD

deploys

a specific revision.

Examples

```text
Main Branch

Release Branch

Git Tag

Commit SHA
```

Production

typically deploys

approved branches

or tags.

---

# Application Workflow

```text
Developer

↓

Git Commit

↓

Repository

↓

Application

↓

Amazon EKS
```

Every deployment

starts

with Git.

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

is

the running configuration

inside Kubernetes.

ArgoCD

continuously compares

desired

and actual state.

---

# Health Status

ArgoCD

evaluates

application health.

Common states

```text
Healthy

Progressing

Degraded

Missing

Unknown
```

Health

indicates

application condition.

---

# Sync Status

ArgoCD

tracks

synchronization status.

Common states

```text
Synced

OutOfSync

Unknown
```

---

# Application Lifecycle

```text
Git Commit

↓

Compare

↓

Sync

↓

Health Check

↓

Running
```

Applications

remain under

continuous monitoring.

---

# What is an ArgoCD Project?

A Project

groups

multiple Applications.

Projects provide

- Security
- RBAC
- Namespace Restrictions
- Repository Restrictions
- Cluster Restrictions

---

# Project Architecture

```text
Project

├── Payment API

├── Orders API

├── Inventory API

└── Notification API
```

Related applications

are managed together.

---

# Why Projects?

Without Projects

```text
All Applications

↓

One Configuration
```

Problems

- Poor Security
- Difficult Governance
- No Team Isolation

---

With Projects

```text
Payments Project

↓

Payment Apps

────────────

Retail Project

↓

Retail Apps
```

Each team

manages

its own applications.

---

# Repository Restrictions

Projects

can limit

which repositories

applications

may use.

Example

```text
Payments Project

↓

payments-git-repository
```

Unauthorized repositories

cannot deploy.

---

# Namespace Restrictions

Projects

can restrict

deployment namespaces.

Example

```text
Payments Project

↓

payments namespace
```

Applications

cannot deploy

outside

approved namespaces.

---

# Cluster Restrictions

Projects

can control

which clusters

applications

may access.

Example

```text
Development Cluster

Testing Cluster

Production Cluster
```

---

# Multi-Team Architecture

```text
Payments Project

↓

Payment Applications

────────────

Retail Project

↓

Retail Applications

────────────

Platform Project

↓

Infrastructure Applications
```

Each team

is isolated.

---

# Multi-Cluster Deployment

One ArgoCD instance

can manage

multiple clusters.

Architecture

```text
Git Repository

↓

ArgoCD

├── Dev Cluster

├── Test Cluster

└── Production Cluster
```

Applications

target

specific clusters.

---

# Git Repository Structure

Enterprise Git repositories

are commonly organized as

```text
gitops/

├── development/

├── testing/

├── staging/

└── production/
```

Each environment

contains

its own manifests.

---

# Environment Promotion

Applications move

through environments

using Git.

```text
Development

↓

Testing

↓

Staging

↓

Production
```

Promotion

is controlled

through pull requests

and approvals.

---

# Enterprise Deployment Flow

```text
Developer

↓

Application Repository

↓

CI Pipeline

↓

Manifest Repository

↓

ArgoCD Application

↓

Amazon EKS
```

CI

updates Git.

ArgoCD

performs deployment.

---

# Banking Example

```text
Payments Project

↓

Payment API

↓

Git Repository

↓

Amazon EKS

↓

Production
```

Every banking application

belongs

to an approved project.

---

# Enterprise Best Practices

- Create one Application per deployable workload.
- Group related applications into Projects.
- Restrict repositories using Projects.
- Restrict namespaces using Projects.
- Separate environments into different Git directories or repositories.
- Deploy only approved branches or tags.
- Keep Git as the single source of truth.
- Use pull requests for environment promotion.

---

# Common Mistakes

- Placing every application in the default Project.
- Allowing unrestricted Git repositories.
- Deploying to any namespace.
- Mixing development and production manifests.
- Using one Application for unrelated services.
- Bypassing Git for production changes.
- Ignoring OutOfSync applications.

---

# Interview Questions

## Basic

- What is an ArgoCD Application?
- What is an ArgoCD Project?
- What is the difference between Health Status and Sync Status?
- What is the Source field?
- What is the Destination field?

## Intermediate

- How do Projects improve security?
- Why should repositories be restricted?
- Explain namespace restrictions.
- How does ArgoCD manage multiple clusters?
- What is environment promotion in GitOps?

## Advanced

- Design an enterprise GitOps architecture using ArgoCD Applications, Projects, multiple Git repositories, and Amazon EKS clusters for secure multi-team deployments.
- Explain how Applications, Projects, repository restrictions, namespace restrictions, and cluster restrictions work together to provide governance and security in ArgoCD.
- A financial organization manages hundreds of microservices across Development, Testing, Staging, and Production environments. Explain how you would design the Application hierarchy, Project structure, Git repository organization, multi-cluster deployment strategy, RBAC, and promotion workflow to ensure secure, scalable, and auditable GitOps operations.

---

# Chapter 4 - ArgoCD Sync Policies, Sync Options & Reconciliation

The core responsibility of ArgoCD

is to keep

the Kubernetes cluster

synchronized

with

the desired state

stored in Git.

This synchronization process

is called

**Sync**.

ArgoCD continuously compares

Git

with

the Kubernetes cluster

and decides

whether changes are required.

---

# Sync Architecture

```text
Git Repository

↓

Desired State

↓

ArgoCD

↓

Compare

↓

Kubernetes Cluster

↓

Actual State
```

If differences exist,

ArgoCD

initiates synchronization.

---

# What is Synchronization?

Synchronization

is the process

of applying

Git changes

to the Kubernetes cluster.

Workflow

```text
Git Commit

↓

Compare

↓

Sync

↓

Cluster Updated
```

---

# Reconciliation Loop

ArgoCD

continuously runs

a reconciliation loop.

```text
Git

↓

Compare

↓

Cluster

↓

Detect Drift

↓

Synchronize
```

This process

runs continuously.

---

# Why Reconciliation?

Without reconciliation

```text
Git

↓

Deployment

↓

Manual Cluster Change
```

Problems

- Configuration Drift
- Inconsistent Clusters
- Unknown Changes

---

With reconciliation

```text
Git

↓

Compare

↓

Drift Detected

↓

Sync

↓

Cluster Corrected
```

Git

always wins.

---

# Sync Policies

ArgoCD supports

two synchronization modes

```text
Manual Sync

────────────

Automatic Sync
```

Choose

based on

environment requirements.

---

# Manual Sync

Manual Sync

requires

an engineer

to approve

every deployment.

Workflow

```text
Git Commit

↓

OutOfSync

↓

Manual Sync

↓

Deployment
```

Often used

in production.

---

# Automatic Sync

Automatic Sync

deploys changes

immediately

after Git updates.

Workflow

```text
Git Commit

↓

Automatic Sync

↓

Deployment
```

Common

for development

and testing.

---

# Auto Sync Architecture

```text
Developer

↓

Git Commit

↓

ArgoCD

↓

Amazon EKS

↓

Pods Updated
```

No manual intervention

is required.

---

# Sync Status

Applications

can have

the following sync states

```text
Synced

OutOfSync

Unknown
```

---

# Synced

```text
Git

=

Cluster
```

Desired state

matches

the cluster.

---

# OutOfSync

```text
Git

≠

Cluster
```

The cluster

differs

from Git.

Synchronization

is required.

---

# Unknown

ArgoCD

cannot determine

application state.

Possible causes

- API Failure
- Repository Issue
- Cluster Connectivity

---

# Self-Healing

Self-Healing

automatically restores

the desired state

if

manual cluster changes

occur.

Workflow

```text
Manual kubectl Edit

↓

Drift

↓

ArgoCD

↓

Restore Git State
```

This is

one of GitOps'

most powerful features.

---

# Pruning

Sometimes

resources are deleted

from Git.

Pruning ensures

they are also removed

from Kubernetes.

Workflow

```text
Git Resource Deleted

↓

Sync

↓

Cluster Resource Deleted
```

Without pruning,

orphaned resources remain.

---

# Sync Options

ArgoCD supports

additional sync behaviors.

Examples

- Prune
- Self Heal
- Create Namespace
- Validate Resources
- Apply Out of Sync Only

These improve

deployment flexibility.

---

# Retry Policy

If synchronization fails,

ArgoCD

can retry automatically.

Workflow

```text
Sync Failed

↓

Retry

↓

Deployment Successful
```

Useful

for temporary failures.

---

# Selective Sync

Instead of

deploying everything,

ArgoCD

can synchronize

specific resources.

Example

```text
Deployment

↓

Update

────────────

Service

↓

No Change
```

---

# Partial Synchronization

Large applications

may contain

hundreds of resources.

Selective synchronization

reduces

deployment time.

---

# Sync Waves

Some resources

must be deployed

before others.

Example

```text
Namespace

↓

ConfigMap

↓

Secret

↓

Deployment

↓

Service

↓

Ingress
```

ArgoCD

supports

ordered synchronization.

---

# Hooks

Hooks execute

tasks

before,

during,

or after

synchronization.

Examples

```text
Database Migration

↓

Deploy

↓

Smoke Test
```

Hooks automate

deployment workflows.

---

# Health Check

After synchronization,

ArgoCD verifies

application health.

Possible states

```text
Healthy

Progressing

Degraded

Missing
```

Deployment

is not complete

until healthy.

---

# Drift Detection

Drift occurs

when

someone modifies

the cluster

directly.

Workflow

```text
Git

↓

Desired State

────────────

kubectl Edit

↓

Cluster Changed

↓

OutOfSync
```

ArgoCD

detects

the difference.

---

# Enterprise Deployment Flow

```text
Developer

↓

GitHub

↓

Merge

↓

ArgoCD

↓

Compare

↓

Sync

↓

Amazon EKS
```

Every deployment

is Git-driven.

---

# Banking Example

```text
Payment API

↓

Git Commit

↓

Automatic Sync

↓

Amazon EKS

↓

Health Check

↓

Production
```

No engineer

logs into

the production cluster

to deploy.

---

# Enterprise Sync Strategy

```text
Development

↓

Automatic Sync

────────────

Testing

↓

Automatic Sync

────────────

Production

↓

Manual Approval

↓

Manual Sync
```

Different environments

can use

different sync policies.

---

# Enterprise Best Practices

- Enable Auto Sync for development environments.
- Use Manual Sync for production if approvals are required.
- Enable Self-Healing to prevent configuration drift.
- Enable Pruning to remove obsolete resources.
- Monitor OutOfSync applications continuously.
- Use Sync Waves for resource dependencies.
- Validate application health after every sync.
- Keep Git as the only deployment source.

---

# Common Mistakes

- Disabling Self-Healing in production.
- Leaving orphaned resources by disabling Prune.
- Using Auto Sync without branch protection.
- Ignoring OutOfSync applications.
- Deploying manually with kubectl.
- Mixing manual and Git-driven changes.
- Not validating application health after synchronization.

---

# Interview Questions

## Basic

- What is synchronization in ArgoCD?
- What is Auto Sync?
- Manual Sync vs Auto Sync.
- What is Self-Healing?
- What is Pruning?

## Intermediate

- Explain the reconciliation loop.
- What is Sync Status?
- What are Sync Waves?
- What are Hooks?
- How does ArgoCD detect configuration drift?

## Advanced

- Design an enterprise GitOps synchronization strategy using Auto Sync, Manual Sync, Self-Healing, Pruning, Sync Waves, Hooks, and health checks for secure deployments to Amazon EKS.
- Explain how ArgoCD's reconciliation loop continuously compares Git with the Kubernetes cluster and automatically restores the desired state.
- A financial organization requires automated deployments for Development and Testing environments but controlled deployments for Production. Explain how you would design sync policies, reconciliation, approval workflows, self-healing, pruning, retry strategies, and health validation to ensure secure, reliable, and auditable GitOps operations.

---

# Chapter 5 - ArgoCD Application Lifecycle, Health Checks & Resource Tracking

ArgoCD continuously manages

the complete lifecycle

of Kubernetes applications.

An application

does not simply deploy once.

Instead,

ArgoCD continuously

- Tracks
- Monitors
- Synchronizes
- Validates
- Recovers

applications

throughout

their lifecycle.

---

# Application Lifecycle

```text
Git Commit

↓

Application Created

↓

Sync

↓

Deployment

↓

Health Check

↓

Running

↓

Continuous Monitoring
```

The application

remains

under continuous management.

---

# Application Creation

Every application

starts

from Git.

Workflow

```text
Git Repository

↓

ArgoCD Application

↓

Amazon EKS
```

The Git repository

defines

the desired state.

---

# Desired State

Desired State

includes

```text
Deployment

Service

Ingress

ConfigMap

Secret

PersistentVolumeClaim
```

Everything

required

to run

the application.

---

# Manifest Generation

ArgoCD

generates manifests

from

- Plain YAML
- Helm Charts
- Kustomize
- Jsonnet

Workflow

```text
Git Repository

↓

Manifest Generation

↓

Kubernetes Resources
```

---

# Resource Creation

After synchronization,

resources

are created

inside Kubernetes.

```text
Deployment

↓

ReplicaSet

↓

Pods
```

Applications

become operational.

---

# Resource Tracking

ArgoCD tracks

every Kubernetes resource

belonging

to an application.

Examples

```text
Deployment

Service

Ingress

ConfigMap

Secret

Job

CronJob
```

Every resource

is monitored.

---

# Resource Ownership

ArgoCD identifies

which resources

belong

to which application.

Architecture

```text
Application

↓

Deployment

↓

Pods

↓

Service

↓

Ingress
```

Ownership

prevents

resource conflicts.

---

# Health Monitoring

ArgoCD continuously checks

resource health.

Health states

```text
Healthy

Progressing

Degraded

Suspended

Missing

Unknown
```

---

# Healthy

```text
Desired State

=

Running State
```

The application

is operating

normally.

---

# Progressing

Resources

are still

being created

or updated.

Examples

```text
Pod Starting

Rolling Update

Replica Creation
```

---

# Degraded

The application

is deployed

but not functioning correctly.

Examples

```text
CrashLoopBackOff

ImagePullBackOff

Failed Pods

Readiness Failure
```

Immediate investigation

is required.

---

# Missing

Expected resources

cannot be found

inside Kubernetes.

Possible causes

- Manual Deletion
- Failed Deployment
- Cluster Issues

---

# Unknown

ArgoCD

cannot determine

resource health.

Possible causes

- API Server Failure
- Cluster Connectivity
- Repository Issues

---

# Health Evaluation

```text
Deployment

↓

ReplicaSet

↓

Pods

↓

Health Status
```

ArgoCD evaluates

multiple resources

before determining

overall application health.

---

# Sync vs Health

| Sync Status | Health Status |
|-------------|---------------|
| Git vs Cluster | Application Condition |
| Synced | Healthy |
| OutOfSync | Degraded |
| Unknown | Unknown |

These represent

different aspects

of application management.

---

# Resource Tree

ArgoCD

displays

application resources

as a hierarchy.

```text
Application

├── Deployment

├── ReplicaSet

├── Pods

├── Service

├── ConfigMap

└── Ingress
```

This simplifies

troubleshooting.

---

# Rolling Updates

Application updates

typically follow

```text
Old Pods

↓

New Pods

↓

Validation

↓

Traffic Shift
```

Health

is monitored

throughout

the rollout.

---

# Failed Deployment

If deployment fails

```text
Deployment

↓

Pods Fail

↓

Health Degraded

↓

Investigation
```

ArgoCD

reports

the failure immediately.

---

# Application Deletion

When

an application

is removed

from Git,

ArgoCD

can remove

associated resources.

Workflow

```text
Application Removed

↓

Git Updated

↓

Sync

↓

Resources Deleted
```

---

# Drift Recovery

Manual changes

cause

configuration drift.

Workflow

```text
kubectl Edit

↓

OutOfSync

↓

Self-Heal

↓

Git State Restored
```

Applications

remain consistent.

---

# Continuous Monitoring

ArgoCD

continuously monitors

```text
Git

↓

Resources

↓

Health

↓

Sync

↓

Cluster
```

Applications

are never

left unmanaged.

---

# Enterprise Lifecycle

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

Health Monitoring

↓

Production
```

---

# Banking Example

```text
Payment API

↓

Git Update

↓

ArgoCD

↓

Rolling Deployment

↓

Health Check

↓

Production
```

Only healthy applications

remain

in production.

---

# Enterprise Dashboard

ArgoCD Web UI

shows

```text
Application

↓

Sync Status

↓

Health Status

↓

Revision

↓

History
```

Operations teams

can quickly

identify problems.

---

# Enterprise Best Practices

- Monitor application health continuously.
- Enable Self-Healing.
- Track every Kubernetes resource.
- Investigate degraded applications immediately.
- Keep applications small and focused.
- Use Health Status for operational monitoring.
- Validate rollouts before promotion.
- Remove orphaned resources using Prune.

---

# Common Mistakes

- Ignoring degraded applications.
- Confusing Sync Status with Health Status.
- Allowing manual cluster changes.
- Deploying unrelated services in one application.
- Ignoring missing resources.
- Not monitoring rollout progress.
- Leaving orphaned Kubernetes resources.

---

# Interview Questions

## Basic

- What is an ArgoCD Application?
- What is Health Status?
- What is Sync Status?
- What is Resource Tracking?
- What is a Healthy application?

## Intermediate

- Healthy vs Progressing vs Degraded.
- How does ArgoCD track Kubernetes resources?
- Why is Resource Ownership important?
- How does ArgoCD detect failed deployments?
- Explain the application lifecycle.

## Advanced

- Design an enterprise application lifecycle using ArgoCD, Amazon EKS, rolling deployments, health checks, resource tracking, and continuous monitoring for large-scale microservices.
- Explain how ArgoCD manages applications throughout their lifecycle using synchronization, health evaluation, resource ownership, and continuous reconciliation.
- A financial organization manages over 300 microservices on Amazon EKS using ArgoCD. Explain how you would design application boundaries, health monitoring, resource tracking, rollout validation, drift recovery, and lifecycle management to ensure highly available, secure, and auditable GitOps deployments.

---

# Chapter 6 - ArgoCD Helm Integration, Kustomize & Configuration Management

Modern Kubernetes applications require

- Environment-specific configuration
- Version-controlled deployments
- Reusable templates
- Easy customization

Instead of maintaining

hundreds of YAML files,

ArgoCD integrates with

- Helm
- Kustomize
- Jsonnet
- Plain Kubernetes Manifests

This enables scalable

GitOps-based configuration management.

---

# Configuration Management Architecture

```text
Git Repository

↓

Helm / Kustomize

↓

ArgoCD

↓

Amazon EKS

↓

Applications
```

ArgoCD

generates manifests

before deployment.

---

# Why Configuration Management?

Without templating

```text
Development

↓

100 YAML Files

────────────

Testing

↓

100 YAML Files

────────────

Production

↓

100 YAML Files
```

Problems

- Duplicate YAML
- Difficult Maintenance
- Configuration Drift

---

With Helm

```text
One Template

↓

Environment Values

↓

Generated Manifests
```

Much easier

to maintain.

---

# Helm Integration

Helm

is the package manager

for Kubernetes.

It packages

Kubernetes resources

into

Charts.

ArgoCD

can deploy

Helm Charts

directly from Git.

---

# Helm Architecture

```text
Git Repository

↓

Helm Chart

↓

Values File

↓

ArgoCD

↓

Amazon EKS
```

---

# Helm Components

A Helm Chart

typically contains

```text
Chart.yaml

↓

Templates

↓

Values.yaml

↓

Generated Manifests
```

---

# Chart.yaml

Defines

chart metadata.

Examples

```text
Application Name

Version

Description

Dependencies
```

---

# Templates Directory

Contains

Kubernetes templates.

Examples

```text
Deployment

Service

Ingress

ConfigMap

Secret
```

Templates

use

Helm variables.

---

# Values File

The values file

contains

environment-specific configuration.

Examples

```text
Replica Count

Image Tag

CPU Limits

Memory Limits

Environment Variables
```

The same chart

supports

multiple environments.

---

# Helm Deployment Flow

```text
Git Commit

↓

Helm Chart

↓

Values

↓

Manifest Generation

↓

ArgoCD

↓

Amazon EKS
```

---

# Environment-Specific Values

Example

```text
Development

↓

values-dev

────────────

Testing

↓

values-test

────────────

Production

↓

values-prod
```

Each environment

uses

different configuration.

---

# Helm Versioning

Every chart

should be versioned.

```text
Chart v1

↓

Chart v2

↓

Chart v3
```

This simplifies

rollback

and upgrades.

---

# What is Kustomize?

Kustomize

customizes

plain Kubernetes YAML

without templates.

It applies

overlays

to

base manifests.

---

# Kustomize Architecture

```text
Base Manifests

↓

Overlay

↓

Generated Manifests

↓

ArgoCD
```

---

# Base Directory

Contains

shared Kubernetes resources.

Example

```text
Deployment

Service

Ingress
```

These resources

are reused.

---

# Overlay Directory

Each environment

adds

its own changes.

Example

```text
Development Overlay

↓

Replica = 1

────────────

Production Overlay

↓

Replica = 5
```

---

# Kustomize Workflow

```text
Base

↓

Overlay

↓

Manifest Generation

↓

Amazon EKS
```

No templates

are required.

---

# Helm vs Kustomize

| Helm | Kustomize |
|-------|-----------|
| Template Engine | Overlay System |
| Values Files | Patches |
| Package Manager | Native Kubernetes Tool |
| Supports Charts | Supports Base + Overlay |

---

# Plain YAML

ArgoCD

can also deploy

plain Kubernetes manifests.

Workflow

```text
Git Repository

↓

YAML

↓

ArgoCD

↓

Amazon EKS
```

Suitable

for

small applications.

---

# Jsonnet Support

ArgoCD

supports

Jsonnet

for

advanced manifest generation.

Used

for

highly customized deployments.

---

# Configuration Promotion

Environment promotion

occurs

through Git.

```text
Development

↓

Testing

↓

Staging

↓

Production
```

Each environment

uses

its own configuration.

---

# Enterprise Git Repository

```text
gitops/

├── helm/

├── kustomize/

├── development/

├── testing/

└── production/
```

Configuration

is organized

by environment.

---

# Enterprise Deployment

```text
Developer

↓

GitHub

↓

Helm Chart

↓

Values

↓

ArgoCD

↓

Amazon EKS
```

---

# Banking Example

```text
Payment API

↓

Helm Chart

↓

Production Values

↓

ArgoCD

↓

Amazon EKS
```

One chart

supports

every environment.

---

# Enterprise Configuration Strategy

```text
Application

↓

Helm Chart

↓

Environment Values

↓

ArgoCD

↓

Amazon EKS
```

Configuration

remains

version controlled.

---

# Enterprise Best Practices

- Store Helm Charts in Git.
- Use separate values files for each environment.
- Keep Helm Charts generic.
- Version every chart release.
- Use Kustomize overlays for Kubernetes-native customization.
- Avoid duplicating YAML.
- Promote configuration through pull requests.
- Keep Git as the single source of truth.

---

# Common Mistakes

- Creating separate charts for every environment.
- Hardcoding production values.
- Duplicating Kubernetes manifests.
- Mixing Helm and manual edits.
- Ignoring chart versioning.
- Keeping environment configuration outside Git.
- Editing generated manifests directly.

---

# Interview Questions

## Basic

- What is Helm?
- What is Kustomize?
- Why does ArgoCD support Helm?
- What is a Helm Chart?
- What is a values file?

## Intermediate

- Helm vs Kustomize.
- What is an Overlay?
- How does ArgoCD generate manifests?
- Why separate values files by environment?
- How does GitOps simplify configuration management?

## Advanced

- Design an enterprise GitOps configuration management strategy using Helm Charts, Kustomize overlays, environment-specific values, Git repositories, and Amazon EKS.
- Explain how ArgoCD integrates with Helm and Kustomize to provide scalable, reusable, and version-controlled Kubernetes deployments.
- A financial organization deploys over 400 microservices across Development, Testing, Staging, and Production environments. Explain how you would organize Helm Charts, values files, Kustomize overlays, Git repositories, versioning, promotion workflows, and ArgoCD applications to ensure maintainable, secure, and auditable configuration management.

---

