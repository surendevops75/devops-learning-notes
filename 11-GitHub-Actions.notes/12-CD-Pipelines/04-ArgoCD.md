# ArgoCD

ArgoCD is a declarative, GitOps-based Continuous Delivery tool for Kubernetes.

It continuously compares the desired state stored in Git with the actual state running inside Kubernetes and synchronizes the cluster when differences are detected.

ArgoCD is commonly used with Kubernetes and Amazon EKS to implement GitOps-based application deployment.

---

# What is ArgoCD?

ArgoCD is a Kubernetes-native Continuous Delivery tool that follows the GitOps methodology.

The basic principle is:

    Git = Desired State

    Kubernetes = Actual State

    ArgoCD = Reconciliation

ArgoCD continuously monitors the Git repository and Kubernetes cluster.

If the desired state and actual state are different, ArgoCD detects the difference.

---

# Why ArgoCD is Used

Traditional deployment:

    Developer
        |
        ↓
    CI Pipeline
        |
        ↓
    kubectl apply
        |
        ↓
    Kubernetes

GitOps deployment:

    Developer
        |
        ↓
    Git
        |
        ↓
    CI Pipeline
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

ArgoCD removes the need for the CI pipeline to directly manage Kubernetes deployments.

---

# GitOps

GitOps is a deployment methodology where Git is used as the source of truth for infrastructure and application configuration.

Core GitOps principles:

- Git as the source of truth
- Declarative configuration
- Version-controlled deployments
- Automated reconciliation
- Auditable changes
- Repeatable deployments
- Easy rollback
- Drift detection

---

# GitOps Mental Model

Remember:

    Git
      |
      ↓
    Desired State
      |
      ↓
    ArgoCD
      |
      ↓
    Kubernetes
      |
      ↓
    Actual State

ArgoCD continuously works toward:

    Desired State = Actual State

---

# Desired State

Desired state represents how the application should look.

Example:

    replicas: 3

    image:
      repository: myapp
      tag: "1.4.7"

This configuration can be stored in Git.

---

# Actual State

Actual state represents what is currently running inside Kubernetes.

Example:

    Desired Replicas: 3

    Running Replicas: 2

ArgoCD detects the difference.

    Desired State
          |
          ↓
        3 Pods

    Actual State
          |
          ↓
        2 Pods

    Difference Detected

---

# Reconciliation

Reconciliation is the process of comparing desired state with actual state and bringing the actual state back to the desired state.

Flow:

    Git
      |
      ↓
    Desired State
      |
      ↓
    ArgoCD
      |
      ↓
    Compare
      |
      +------ Same ------→ Synced
      |
      +------ Different -→ OutOfSync
                              |
                              ↓
                            Sync
                              |
                              ↓
                         Kubernetes

---

# ArgoCD Architecture

Simplified architecture:

    Git Repository
          |
          ↓
       ArgoCD
          |
          +----------------+
          |                |
          ↓                ↓
    Repo Server       API Server
          |
          ↓
    Application Controller
          |
          ↓
    Kubernetes API
          |
          ↓
       EKS Cluster
          |
          ↓
     Applications

---

# ArgoCD Components

Important ArgoCD components include:

    argocd-server
    argocd-repo-server
    argocd-application-controller
    argocd-redis

Each component has a specific responsibility.

---

# ArgoCD API Server

The ArgoCD API Server provides interfaces for:

- Web UI
- CLI
- API
- Authentication
- Application management
- Repository configuration
- Cluster configuration
- Synchronization operations

The API server is the main interface used by administrators and users.

---

# ArgoCD Repository Server

The repository server retrieves application configuration from Git repositories.

It can work with:

- Kubernetes YAML
- Helm
- Kustomize
- Other supported configuration sources

Conceptually:

    Git Repository
         |
         ↓
    Repo Server
         |
         ↓
    Kubernetes Manifests

---

# ArgoCD Application Controller

The Application Controller is responsible for monitoring applications.

It compares:

    Desired State

with:

    Live State

If a difference is detected, the controller can synchronize the resources depending on the application's configuration.

---

# ArgoCD Redis

Redis is used internally by ArgoCD for caching and related application functionality.

It is an internal ArgoCD component and is not the database used by the application being deployed.

---

# ArgoCD Application

An ArgoCD Application is a Kubernetes custom resource that defines how an application should be deployed.

An Application generally defines:

    Source
    Destination
    Project
    Sync Policy

---

# ArgoCD Application Structure

Conceptually:

    Application
        |
        +-- Project
        |
        +-- Source
        |     |
        |     +-- Repository
        |     +-- Path
        |     +-- Revision
        |
        +-- Destination
        |     |
        |     +-- Cluster
        |     +-- Namespace
        |
        +-- Sync Policy

---

# Example ArgoCD Application

Example:

    apiVersion: argoproj.io/v1alpha1
    kind: Application

    metadata:
      name: myapp

    spec:
      project: default

      source:
        repoURL: https://github.com/example/gitops.git
        targetRevision: main
        path: applications/myapp

      destination:
        server: https://kubernetes.default.svc
        namespace: production

      syncPolicy:
        automated: {}

This Application tells ArgoCD:

- Which Git repository to use
- Which Git revision to track
- Which directory contains the manifests
- Which Kubernetes cluster to deploy to
- Which namespace to use
- Whether synchronization should be automated

---

# Source

The `source` section defines where the desired application configuration comes from.

Common fields:

    repoURL
    targetRevision
    path

Example:

    source:
      repoURL: https://github.com/example/gitops.git
      targetRevision: main
      path: applications/myapp

---

# repoURL

`repoURL` specifies the Git repository containing the application configuration.

Example:

    repoURL: https://github.com/company/gitops.git

The repository must be accessible by ArgoCD.

---

# targetRevision

`targetRevision` specifies the Git revision ArgoCD should track.

Examples:

    main
    master
    v1.0.0
    Git Commit SHA

Example:

    targetRevision: main

---

# path

`path` specifies the directory inside the Git repository containing the application configuration.

Example:

    path: applications/myapp

Repository:

    gitops/
      |
      └── applications/
          |
          └── myapp/
              |
              ├── deployment.yaml
              ├── service.yaml
              └── ingress.yaml

---

# Destination

The destination defines where ArgoCD should deploy the application.

Example:

    destination:
      server: https://kubernetes.default.svc
      namespace: production

The destination generally includes:

    Kubernetes Cluster
    Namespace

---

# Kubernetes Namespace

Applications can be deployed into specific namespaces.

Example:

    destination:
      namespace: production

Other environments might use:

    development
    qa
    uat
    production

---

# Sync

Sync means ArgoCD applies the desired configuration from Git to Kubernetes.

Flow:

    Git
      |
      ↓
    Desired State
      |
      ↓
    ArgoCD
      |
      ↓
    Sync
      |
      ↓
    Kubernetes

After successful synchronization:

    Git State = Kubernetes State

---

# Manual Sync

ArgoCD can be configured for manual synchronization.

Flow:

    Git Change
        |
        ↓
    ArgoCD Detects Change
        |
        ↓
    OutOfSync
        |
        ↓
    Engineer Reviews
        |
        ↓
    Manual Sync
        |
        ↓
    Kubernetes

Manual synchronization can be useful for production environments where deployments require explicit approval.

---

# Automated Sync

ArgoCD can automatically synchronize changes.

Example:

    syncPolicy:
      automated: {}

Flow:

    Git Change
        |
        ↓
    ArgoCD Detects Change
        |
        ↓
    Automated Sync
        |
        ↓
    Kubernetes
        |
        ↓
    Application Updated

---

# Self Heal

Self-healing allows ArgoCD to correct certain manual changes made directly to Kubernetes.

Example:

    Git:
      replicas: 3

    Kubernetes:
      replicas: 1

ArgoCD detects the difference.

With self-healing enabled:

    Kubernetes
      |
      ↓
    replicas: 1
      |
      ↓
    Drift Detected
      |
      ↓
    ArgoCD
      |
      ↓
    replicas: 3

---

# Configuration Drift

Configuration drift occurs when the Kubernetes live state differs from the desired state stored in Git.

Example:

    Git:
      image: 1.4.7

    Kubernetes:
      image: 1.4.6

This results in:

    Desired State ≠ Actual State

ArgoCD detects this difference.

---

# Sync Status

ArgoCD applications commonly show statuses such as:

    Synced
    OutOfSync
    Unknown

---

# Synced

`Synced` means the live Kubernetes resources match the desired configuration tracked by ArgoCD.

Conceptually:

    Git
      =
    Kubernetes

---

# OutOfSync

`OutOfSync` means the live state differs from the desired state.

Example:

    Git:
      replicas: 3

    Kubernetes:
      replicas: 2

ArgoCD:

    OutOfSync

---

# Unknown

`Unknown` means ArgoCD cannot determine the current synchronization state.

Possible causes:

- Repository access problem
- Cluster access problem
- Manifest generation problem
- Temporary communication problem

---

# Health Status

ArgoCD also tracks application health.

Common health states include:

    Healthy
    Progressing
    Degraded
    Suspended
    Missing
    Unknown

---

# Healthy

Healthy means the application's resources are operating as expected according to ArgoCD health checks.

Example:

    Deployment
        |
        ↓
    Pods Ready
        |
        ↓
    Service Available
        |
        ↓
    Healthy

---

# Progressing

Progressing means the application is moving toward the desired healthy state.

Example:

    New Deployment
        |
        ↓
    New Pods Starting
        |
        ↓
    Readiness Checks
        |
        ↓
    Healthy

---

# Degraded

Degraded indicates that the application is not in the expected healthy state.

Possible causes:

- CrashLoopBackOff
- ImagePullBackOff
- Failed readiness probe
- Failed liveness probe
- Application errors
- Missing configuration
- Resource limitations

---

# ArgoCD and Helm

ArgoCD can deploy Helm charts.

Example:

    Git Repository
        |
        └── helm/myapp/
            |
            ├── Chart.yaml
            ├── values.yaml
            └── templates/
        |
        ↓
      ArgoCD
        |
        ↓
    Kubernetes

ArgoCD uses Helm to render the manifests and manages the resulting Kubernetes resources.

---

# Helm Values With ArgoCD

Example:

    source:
      repoURL: https://github.com/example/gitops.git
      path: helm/myapp

      helm:
        values: |
          replicaCount: 3

          image:
            tag: "1.4.7"

ArgoCD can use the provided values when generating manifests.

---

# ArgoCD and Kustomize

ArgoCD also supports Kustomize.

Example:

    base/
      |
      ├── deployment.yaml
      └── service.yaml

    overlays/
      |
      ├── dev/
      ├── qa/
      └── prod/

ArgoCD can deploy the appropriate Kustomize overlay.

---

# GitOps Repository Structure

Example:

    gitops/
      |
      ├── applications/
      |
      ├── environments/
      |     |
      |     ├── dev/
      |     ├── qa/
      |     ├── uat/
      |     └── prod/
      |
      └── charts/

The exact repository structure depends on the team's GitOps strategy.

---

# Application Repository vs GitOps Repository

A common architecture separates application source code from deployment configuration.

Application repository:

    Source Code
    Dockerfile
    Tests
    Application Configuration

GitOps repository:

    Kubernetes Manifests
    Helm Values
    Kustomize Configuration
    ArgoCD Applications

This provides a clean separation between application development and deployment configuration.

---

# CI and CD Separation

GitHub Actions or Jenkins can handle CI.

ArgoCD can handle Kubernetes CD.

CI:

    Build
    Test
    Security Scan
    Package
    Push Image

CD:

    GitOps
    Synchronization
    Reconciliation
    Drift Detection
    Deployment

---

# GitHub Actions + ArgoCD

Typical flow:

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
    Application

---

# Jenkins + ArgoCD

ArgoCD can also work with Jenkins.

Flow:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Jenkins
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

---

# ArgoCD and ECR

ArgoCD does not build Docker images.

The responsibilities are separated.

GitHub Actions or Jenkins:

    Build Image
        |
        ↓
    Security Scan
        |
        ↓
    Push Image
        |
        ↓
    ECR

GitOps:

    Update Image Reference
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

---

# Image Tag Update Flow

Example:

    Current:

    image:
      repository: myapp
      tag: "1.4.6"

CI builds:

    myapp:1.4.7

Push:

    ECR
      |
      ↓
    myapp:1.4.7

GitOps update:

    image:
      repository: myapp
      tag: "1.4.7"

ArgoCD detects the Git change.

---

# Image Deployment Flow

    Developer
        |
        ↓
    Application Repository
        |
        ↓
    GitHub Actions
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
    Update GitOps Repository
        |
        ↓
    Git Commit
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Kubernetes Pods

---

# Immutable Image Tags

Use immutable image versions.

Good:

    myapp:1.4.7

or:

    myapp:<commit-sha>

Avoid:

    myapp:latest

Immutable tags make deployments easier to trace and rollback.

---

# Git as Source of Truth

The main GitOps rule is:

    Git
      |
      ↓
    Desired Deployment State

If a configuration change is required:

    Change Git
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Merge
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

---

# Why GitOps Improves Auditability

Every deployment configuration change can be represented by a Git commit.

Example:

    Commit
      |
      +-- Author
      +-- Timestamp
      +-- Changed Files
      +-- Commit Message
      |
      ↓
    Deployment

This provides an audit trail for deployment changes.

---

# GitOps Rollback

GitOps makes rollback straightforward.

Example:

    Version 1.4.6
        |
        ↓
    Version 1.4.7
        |
        ↓
    Production Issue
        |
        ↓
    Git Revert
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Version 1.4.6

---

# Git Revert Rollback

Example Git history:

    Commit A → image 1.4.5
    Commit B → image 1.4.6
    Commit C → image 1.4.7

If version 1.4.7 causes a problem:

    Revert Commit C

Then:

    Git
      |
      ↓
    image 1.4.6
      |
      ↓
    ArgoCD
      |
      ↓
    Kubernetes
      |
      ↓
    1.4.6

---

# ArgoCD Rollback

ArgoCD also provides deployment history and rollback capabilities.

However, in a GitOps model, Git should remain the source of truth.

For a permanent rollback, reverting the Git change is generally preferred because it keeps:

    Git State = Cluster State

---

# ArgoCD and EKS

ArgoCD can manage applications running on Amazon EKS.

Architecture:

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
       |
       ↓
    Kubernetes
       |
       ↓
    Application

---

# ArgoCD on EKS

Example:

    AWS
      |
      └── EKS
           |
           ├── ArgoCD
           |
           ├── Application Namespace
           |
           ├── Ingress
           |
           └── Application Pods

ArgoCD manages Kubernetes resources running inside EKS.

---

# Multi-Cluster ArgoCD

ArgoCD can manage multiple Kubernetes clusters.

Example:

    ArgoCD
       |
       +------→ DEV EKS
       |
       +------→ QA EKS
       |
       +------→ UAT EKS
       |
       └------→ PROD EKS

This allows centralized GitOps deployment.

---

# Multi-Environment GitOps

Example:

    GitOps Repository
        |
        ├── dev
        |    |
        |    └── myapp
        |
        ├── qa
        |    |
        |    └── myapp
        |
        ├── uat
        |    |
        |    └── myapp
        |
        └── prod
             |
             └── myapp

Each environment can have different:

    Image Version
    Replicas
    Configuration
    Resources
    Namespace

---

# ArgoCD Projects

ArgoCD Projects provide grouping and access control for applications.

A Project can restrict:

    Allowed Repositories
    Allowed Clusters
    Allowed Namespaces
    Allowed Resource Types

This is useful in enterprise environments.

---

# Example ArgoCD Project

Conceptually:

    Project: production

    Repository:
        GitOps Repository

    Cluster:
        Production EKS

    Namespace:
        production

This prevents applications from being deployed outside the permitted locations.

---

# ArgoCD RBAC

ArgoCD supports Role-Based Access Control.

Example:

    Developer
        |
        +-- View Applications

    DevOps Engineer
        |
        +-- View
        +-- Sync

    Platform Administrator
        |
        +-- Manage Applications
        +-- Manage Projects
        +-- Manage Clusters

Use least privilege.

---

# ArgoCD Repository Authentication

ArgoCD needs access to Git repositories.

Authentication can use supported methods such as:

    SSH Keys
    HTTPS Credentials
    Access Tokens

Credentials must be securely managed.

---

# ArgoCD Cluster Access

ArgoCD needs access to Kubernetes clusters in order to deploy resources.

Permissions should be limited to the resources and namespaces that ArgoCD needs to manage.

Avoid unnecessarily granting broad cluster-admin permissions.

---

# ArgoCD Secrets

Do not store plaintext secrets in Git.

Avoid:

    databasePassword: "mypassword"

Use appropriate secret-management mechanisms.

Examples:

    AWS Secrets Manager
    External Secrets
    Sealed Secrets
    Other Secret Management Solutions

---

# ArgoCD and Secrets

Typical architecture:

    Secret Manager
        |
        ↓
    Secret Integration
        |
        ↓
    Kubernetes Secret
        |
        ↓
    Application

The GitOps repository should contain references or configuration required to retrieve secrets rather than exposing sensitive values.

---

# Sync Waves

Sync waves can control deployment order.

Example:

    Wave 0
        |
        ↓
    Namespace
        |
        ↓
    Wave 1
        |
        ↓
    ConfigMap / Secret
        |
        ↓
    Wave 2
        |
        ↓
    Application
        |
        ↓
    Wave 3
        |
        ↓
    Validation

This is useful when resources have dependencies.

---

# ArgoCD Hooks

ArgoCD supports deployment hooks.

Common hook phases include:

    PreSync
    Sync
    PostSync
    SyncFail

Hooks should be used carefully.

Too much deployment logic inside hooks can make the GitOps process difficult to understand.

---

# PreSync Hook

A PreSync hook runs before normal synchronization.

Possible use cases:

    Preparation
    Validation
    Database Migration Preparation

---

# PostSync Hook

A PostSync hook runs after synchronization.

Possible use cases:

    Smoke Tests
    Validation
    Notifications

---

# SyncFail Hook

A SyncFail hook can be used when synchronization fails.

Possible use cases:

    Notification
    Incident Trigger
    Failure Handling

---

# ArgoCD Notifications

ArgoCD can send notifications about application events.

Example:

    Deployment
        |
        ↓
    ArgoCD
        |
        ↓
    Notification
        |
        +-- Slack
        +-- Email
        +-- Other Systems

Possible notifications:

    Sync Success
    Sync Failed
    Application Degraded
    Application Healthy

---

# ArgoCD Web UI

The ArgoCD UI provides visibility into:

- Applications
- Sync Status
- Health Status
- Kubernetes Resources
- Deployment History
- Events
- Differences
- Application Details

It is useful for deployment monitoring and troubleshooting.

---

# ArgoCD CLI

The ArgoCD command-line tool is:

    argocd

Common commands:

    argocd login
    argocd app list
    argocd app get
    argocd app sync
    argocd app diff
    argocd app history
    argocd app rollback
    argocd app delete

---

# argocd app list

Command:

    argocd app list

This displays ArgoCD applications.

It can show:

    Application Name
    Project
    Cluster
    Namespace
    Sync Status
    Health Status

---

# argocd app get

Command:

    argocd app get myapp

This provides detailed information about an application.

Useful for troubleshooting:

    Sync Status
    Health
    Repository
    Revision
    Resources
    Conditions

---

# argocd app sync

Command:

    argocd app sync myapp

This manually triggers synchronization.

Flow:

    Git
      |
      ↓
    Desired State
      |
      ↓
    argocd app sync
      |
      ↓
    Kubernetes

---

# argocd app diff

Command:

    argocd app diff myapp

This shows differences between desired state and live state.

Example:

    Desired:
      replicas: 3

    Live:
      replicas: 2

The difference helps identify configuration drift.

---

# argocd app history

Command:

    argocd app history myapp

This displays application deployment history.

It helps identify previous deployments and revisions.

---

# ArgoCD Pruning

Pruning removes Kubernetes resources that are no longer defined in the desired state.

Example:

    Git:
      deployment.yaml

An old resource:

    old-configmap

is no longer defined in Git.

With pruning enabled, ArgoCD can remove the resource.

Pruning should be used carefully, especially in production.

---

# Automated Pruning

Flow:

    Git
      |
      ↓
    Resource Removed
      |
      ↓
    ArgoCD
      |
      ↓
    Difference Detected
      |
      ↓
    Prune
      |
      ↓
    Kubernetes

---

# ArgoCD Retry

ArgoCD can retry synchronization operations.

Retries can help with temporary problems such as:

    Temporary API Failure
    Resource Timing Issue
    Transient Connectivity Problem

Retries should not be used to hide permanent configuration errors.

---

# ApplicationSet

ApplicationSet can generate multiple ArgoCD Applications from a template.

Useful for:

    Multiple Environments
    Multiple Clusters
    Multiple Applications
    Large Microservices Platforms

Conceptually:

    ApplicationSet
         |
         +-- DEV Application
         +-- QA Application
         +-- UAT Application
         └-- PROD Application

---

# ApplicationSet Mental Model

    Template
       |
       ↓
    Generator
       |
       ↓
    Multiple Applications
       |
       ↓
    ArgoCD
       |
       ↓
    Kubernetes Clusters

---

# App of Apps Pattern

The App of Apps pattern uses one ArgoCD Application to manage other ArgoCD Applications.

Example:

    Root Application
          |
          +-- User Application
          +-- Product Application
          +-- Cart Application
          +-- Orders Application
          +-- Payment Application
          +-- Inventory Application
          +-- Notification Application

This pattern can help organize large application platforms.

---

# ArgoCD and Microservices

For a microservices platform:

    GitOps Repository
        |
        +-- user
        +-- product
        +-- cart
        +-- orders
        +-- payment
        +-- inventory
        +-- notification
        |
        ↓
      ArgoCD
        |
        ↓
       EKS

Each service can be independently deployed and synchronized.

---

# Microservices Deployment Flow

    Developer
        |
        ↓
    Service Repository
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
    Microservice

---

# ArgoCD and Immutable Deployments

A good GitOps pipeline should use versioned image tags.

Example:

    image:
      repository: payment
      tag: "1.5.2"

Git commit:

    Update payment image to 1.5.2

ArgoCD detects the commit and deploys the new version.

---

# Production Deployment Flow

A controlled production flow can be:

    Developer
        |
        ↓
    Application Repository
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
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Merge
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

# Production Approval

For production deployments:

    GitOps Change
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Merge
        |
        ↓
    ArgoCD
        |
        ↓
    Production

This provides controlled Git-based deployment.

---

# ArgoCD Troubleshooting

When an application is OutOfSync or Degraded, check:

    ArgoCD Application
    Git Repository
    Git Revision
    Repository Path
    Manifest Generation
    Kubernetes Resources
    Pods
    Events
    Logs
    RBAC
    Repository Access
    Cluster Access

---

# Troubleshooting OutOfSync

First:

    argocd app get myapp

Then:

    argocd app diff myapp

Check for differences in:

    Image
    Replicas
    Labels
    Annotations
    ConfigMaps
    Secrets
    Services
    Ingress
    Resource Configuration

Determine whether the difference is expected.

---

# Troubleshooting Degraded Application

Check:

    kubectl get pods

Then:

    kubectl describe pod <pod-name>

Then:

    kubectl logs <pod-name>

Also:

    kubectl get events

Common problems:

    CrashLoopBackOff
    ImagePullBackOff
    OOMKilled
    Readiness Probe Failure
    Liveness Probe Failure
    Missing Secret
    Invalid Configuration
    Resource Limits

---

# Troubleshooting Sync Failure

Possible causes:

    Invalid YAML
    Invalid Helm Template
    Missing Resource
    RBAC Failure
    Invalid Kubernetes Configuration
    Repository Access Problem
    Cluster Access Problem
    Admission Policy Failure

Troubleshooting flow:

    ArgoCD
       |
       ↓
    Application
       |
       ↓
    Sync Details
       |
       ↓
    Failed Resource
       |
       ↓
    Kubernetes Events
       |
       ↓
    Root Cause

---

# Helm Rendering Failure

If ArgoCD uses Helm and manifest generation fails:

Check:

    Chart.yaml
    values.yaml
    Templates
    Helm Values
    Dependencies
    Helm Syntax

Render locally:

    helm template myapp ./chart \
      -f values-prod.yaml

Compare the result with the ArgoCD configuration.

---

# Repository Access Failure

If ArgoCD cannot access Git:

Check:

    Repository URL
    Credentials
    SSH Key
    Token
    Repository Permissions
    Network Connectivity

The repository must be accessible by ArgoCD.

---

# Cluster Access Failure

If ArgoCD cannot deploy to Kubernetes:

Check:

    Cluster Connection
    Authentication
    RBAC
    Service Account
    Network Connectivity

ArgoCD must have appropriate permissions to manage the required resources.

---

# ArgoCD Security Best Practices

- Use least-privilege RBAC
- Secure ArgoCD administration
- Secure Git credentials
- Secure cluster credentials
- Avoid plaintext secrets
- Use Projects
- Restrict repositories
- Restrict destination clusters
- Restrict namespaces
- Protect production applications
- Use immutable image tags
- Review GitOps pull requests
- Protect production branches
- Audit deployment changes
- Use appropriate secret-management solutions

---

# ArgoCD Project Security

Projects should restrict:

    Source Repositories
    Destination Clusters
    Destination Namespaces
    Resource Types

Example:

    Production Project
        |
        +-- GitOps Repository
        |
        +-- Production EKS
        |
        +-- production namespace

This reduces the risk of unauthorized deployments.

---

# ArgoCD Monitoring

Monitor:

    Application Health
    Sync Status
    Sync Failures
    Repository Errors
    Cluster Errors
    Controller Errors
    Deployment Duration
    OutOfSync Applications
    Failed Synchronizations

---

# ArgoCD and Prometheus

ArgoCD exposes metrics that can be collected by Prometheus.

Architecture:

    ArgoCD
       |
       ↓
    Metrics
       |
       ↓
    Prometheus
       |
       ↓
    Grafana

---

# ArgoCD and Grafana

Grafana can be used to visualize ArgoCD-related metrics.

Useful information:

    Application Health
    Sync Status
    Sync Failures
    Deployment Activity
    Reconciliation Activity

---

# ArgoCD and ELK

ArgoCD logs can be collected into an ELK-based logging platform.

Flow:

    ArgoCD
       |
       ↓
    Logs
       |
       ↓
    Elasticsearch
       |
       ↓
    Kibana

This can help with centralized troubleshooting and operational analysis.

---

# Post-Deployment Validation

After ArgoCD synchronization:

    ArgoCD Sync
        |
        ↓
    Application Health
        |
        ↓
    Pods Ready
        |
        ↓
    Service Available
        |
        ↓
    ALB / Ingress
        |
        ↓
    Application Endpoint
        |
        ↓
    Smoke Test

A successful sync does not automatically mean the application is healthy.

---

# ArgoCD Deployment Failure

Example:

    New Version
        |
        ↓
    Git Commit
        |
        ↓
    ArgoCD
        |
        ↓
    Sync
        |
        ↓
    New Pods
        |
        ↓
    CrashLoopBackOff
        |
        ↓
    Application Degraded
        |
        ↓
    Investigation
        |
        ↓
    Git Revert
        |
        ↓
    ArgoCD Sync
        |
        ↓
    Previous Version

---

# ArgoCD and Kubernetes

ArgoCD manages Kubernetes resources.

Examples:

    Deployment
    Service
    ConfigMap
    Secret
    Ingress
    Namespace
    HPA
    ServiceAccount
    RBAC Resources

The exact resources depend on the application.

---

# ArgoCD vs Jenkins

Jenkins is a general-purpose automation and CI/CD platform.

ArgoCD is specifically focused on Kubernetes GitOps-based Continuous Delivery.

Jenkins:

    Build
    Test
    Scan
    Package
    Automation

ArgoCD:

    GitOps
    Deployment
    Reconciliation
    Drift Detection
    Synchronization

They can work together.

---

# ArgoCD vs GitHub Actions

GitHub Actions can perform:

    Build
    Test
    Security Scan
    Docker Build
    Image Push
    Git Operations

ArgoCD performs:

    Kubernetes CD
    GitOps
    Synchronization
    Reconciliation
    Drift Detection

A common architecture uses both.

---

# Why ArgoCD Instead of kubectl in CI?

Traditional:

    CI
      |
      ↓
    kubectl apply
      |
      ↓
    Kubernetes

GitOps:

    CI
      |
      ↓
    Update Git
      |
      ↓
    ArgoCD
      |
      ↓
    Kubernetes

GitOps provides:

- Git audit trail
- Declarative deployment
- Drift detection
- Continuous reconciliation
- Controlled deployment
- Easier rollback
- Separation between CI and CD

---

# ArgoCD and kubectl

ArgoCD does not eliminate kubectl.

`kubectl` is still useful for:

    Troubleshooting
    Inspection
    Debugging
    Emergency Operations

However, in a GitOps model, normal application deployment should be performed through Git and ArgoCD.

---

# ArgoCD Scenario

## How Does ArgoCD Work?

ArgoCD monitors the desired configuration stored in Git and compares it with the live Kubernetes state.

If they differ:

    Git
      |
      ↓
    Desired State

    Kubernetes
      |
      ↓
    Actual State

ArgoCD detects the difference and can synchronize Kubernetes with Git.

---

# ArgoCD Scenario

## What is GitOps?

GitOps is a methodology where Git stores the declarative desired state.

The flow is:

    Git
      |
      ↓
    Desired State
      |
      ↓
    ArgoCD
      |
      ↓
    Kubernetes

Git becomes the source of truth for deployment configuration.

---

# ArgoCD Scenario

## What is OutOfSync?

OutOfSync means the live Kubernetes state differs from the desired state represented by the ArgoCD Application.

Example:

    Git:
      replicas: 3

    Kubernetes:
      replicas: 2

ArgoCD:

    OutOfSync

---

# ArgoCD Scenario

## What is the Difference Between Synced and Healthy?

Synced means:

    Desired State = Live State

Healthy means:

    Application Resources Are Operating Normally

An application can be:

    Synced
    Degraded

because the deployed configuration matches Git but the application itself is unhealthy.

---

# ArgoCD Scenario

## What is Self-Healing?

Self-healing allows ArgoCD to automatically correct certain changes made directly to Kubernetes.

Example:

    Git:
      replicas: 3

    Kubernetes:
      replicas: 1

ArgoCD detects the drift and can restore:

    replicas: 3

---

# ArgoCD Scenario

## How Would You Roll Back a Deployment?

I would normally revert the Git change that introduced the problematic version.

Flow:

    Bad Deployment
        |
        ↓
    Git Revert
        |
        ↓
    Previous Configuration
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Previous Version

This keeps Git and Kubernetes synchronized.

---

# ArgoCD Scenario

## How Would You Deploy a New Docker Image?

The flow would be:

    Developer
        |
        ↓
    GitHub Actions
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    ECR
        |
        ↓
    Update GitOps Image Tag
        |
        ↓
    Git Commit
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    New Pods

---

# ArgoCD Scenario

## Why Should CI Not Directly Deploy to Kubernetes?

The main reason is separation of responsibilities.

CI:

    Build
    Test
    Scan
    Package

ArgoCD:

    Deploy
    Reconcile
    Detect Drift
    Synchronize

This creates a cleaner GitOps architecture.

---

# ArgoCD Scenario

## How Would You Troubleshoot an OutOfSync Application?

I would:

    1. Check ArgoCD application status
    2. Check synchronization status
    3. Run application diff
    4. Identify changed resources
    5. Compare Git configuration
    6. Compare Kubernetes resources
    7. Determine whether the difference is expected
    8. Sync if appropriate
    9. Investigate manual changes if unexpected

Commands:

    argocd app get myapp

    argocd app diff myapp

---

# ArgoCD Scenario

## How Would You Troubleshoot a Degraded Application?

I would check:

    ArgoCD Health
    Pod Status
    Pod Events
    Container Logs
    Deployment
    Service
    ConfigMap
    Secret
    Readiness Probe
    Liveness Probe
    Resource Limits

Commands:

    kubectl get pods

    kubectl describe pod <pod>

    kubectl logs <pod>

    kubectl get events

---

# ArgoCD Scenario

## How Would You Secure Production ArgoCD?

I would use:

    1. RBAC
    2. Least Privilege
    3. ArgoCD Projects
    4. Restricted Repositories
    5. Restricted Clusters
    6. Restricted Namespaces
    7. Protected Git Branches
    8. Pull Request Reviews
    9. Secure Credentials
    10. Secret Management
    11. Audit Logging
    12. Production Approval

---

# ArgoCD Scenario

## How Would You Deploy Helm Applications Using ArgoCD?

I would store the Helm chart in Git.

Example:

    gitops/
      |
      └── helm/
          |
          └── myapp/
              |
              ├── Chart.yaml
              ├── values.yaml
              └── templates/

ArgoCD points to the Helm chart.

ArgoCD renders the chart and deploys the generated Kubernetes resources.

---

# ArgoCD Scenario

## How Would You Use ArgoCD With EKS?

Architecture:

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
       |
       ↓
    Kubernetes
       |
       ↓
    Application

ArgoCD manages the Kubernetes workloads running on EKS.

---

# ArgoCD Scenario

## How Would You Manage Multiple EKS Clusters?

I would register the required Kubernetes clusters with ArgoCD and organize applications using Projects and environment-specific GitOps configuration.

Example:

    ArgoCD
       |
       +-- DEV EKS
       +-- QA EKS
       +-- UAT EKS
       └-- PROD EKS

Permissions should be restricted using RBAC and Projects.

---

# ArgoCD Scenario

## How Would You Handle a Failed Production Deployment?

I would:

    1. Check ArgoCD sync status
    2. Check application health
    3. Identify the failed resource
    4. Check Kubernetes events
    5. Check application logs
    6. Determine whether rollback is required
    7. Revert the Git configuration if necessary
    8. Allow ArgoCD to synchronize the known-good version
    9. Validate application health
    10. Perform root-cause analysis

---

# ArgoCD Scenario

## What Happens If ArgoCD Is Unavailable?

Existing applications running in Kubernetes generally continue running because ArgoCD is not responsible for running the application workloads.

However, during ArgoCD downtime:

    New Deployments
    Git Synchronization
    Drift Detection
    Reconciliation

may be delayed.

Once ArgoCD is available again, it can resume reconciliation.

---

# ArgoCD Scenario

## What Happens If Git Is Temporarily Unavailable?

ArgoCD may not be able to retrieve new desired configuration from Git.

Existing workloads can continue running.

After Git connectivity is restored, ArgoCD can resume synchronization and reconciliation.

---

# ArgoCD Scenario

## Does ArgoCD Build Docker Images?

No.

A typical architecture is:

    GitHub Actions
        |
        ↓
    Build Docker Image
        |
        ↓
    Security Scan
        |
        ↓
    ECR
        |
        ↓
    Update GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

---

# ArgoCD Scenario

## Does ArgoCD Replace Kubernetes?

No.

Kubernetes provides:

    Container Orchestration
    Scheduling
    Service Discovery
    Networking
    Scaling
    Workload Management

ArgoCD provides:

    GitOps
    Continuous Delivery
    Reconciliation
    Drift Detection
    Deployment Synchronization

---

# ArgoCD Scenario

## Does ArgoCD Replace GitHub Actions?

No.

They can work together.

GitHub Actions:

    Build
    Test
    Security Scan
    Docker Build
    Push Image

ArgoCD:

    GitOps
    Deployment
    Synchronization
    Reconciliation
    Drift Detection

---

# ArgoCD Scenario

## How Would You Design a Production GitOps Pipeline?

I would use:

    Application Repository
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
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Merge
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Health Validation

---

# ArgoCD Best Practices

- Keep Git as the source of truth
- Use declarative configuration
- Use immutable image tags
- Use Pull Requests for production changes
- Separate application source code and GitOps configuration where appropriate
- Use RBAC
- Use ArgoCD Projects
- Restrict repositories
- Restrict clusters
- Restrict namespaces
- Protect production applications
- Avoid plaintext secrets
- Use appropriate secret management
- Enable self-healing where appropriate
- Use pruning carefully
- Monitor application health
- Monitor synchronization status
- Maintain rollback procedures
- Use Helm or Kustomize appropriately
- Avoid unnecessary manual Kubernetes changes
- Use Git-based deployment history
- Protect production Git branches

---

# ArgoCD Anti-Patterns

## Manual kubectl as the Normal Deployment Process

Bad:

    Developer
        |
        ↓
    kubectl apply
        |
        ↓
    Production

Better:

    Git
      |
      ↓
    ArgoCD
      |
      ↓
    Production

---

# ArgoCD Anti-Pattern

## Using latest Image Tag

Bad:

    image:
      tag: latest

Better:

    image:
      tag: "1.4.7"

or:

    image:
      tag: "<commit-sha>"

Immutable versions improve traceability.

---

# ArgoCD Anti-Pattern

## Secrets in Git

Bad:

    databasePassword: "production-password"

Better:

    Secret Manager
        |
        ↓
    Kubernetes Secret
        |
        ↓
    Application

---

# ArgoCD Anti-Pattern

## Excessive Permissions

Bad:

    ArgoCD
       |
       ↓
    Unlimited Cluster Access

Better:

    ArgoCD
       |
       ↓
    RBAC
       |
       ↓
    Project Restrictions
       |
       ↓
    Namespace Restrictions
       |
       ↓
    Least Privilege

---

# ArgoCD Anti-Pattern

## Direct Production Changes

Bad:

    Developer
       |
       ↓
    Direct Git Change
       |
       ↓
    Production

Better:

    Pull Request
       |
       ↓
    Review
       |
       ↓
    Approval
       |
       ↓
    Merge
       |
       ↓
    ArgoCD
       |
       ↓
    Production

---

# ArgoCD Interview Questions

## Basic Questions

1. What is ArgoCD?
2. What is GitOps?
3. Why is ArgoCD used?
4. What is an ArgoCD Application?
5. What is synchronization?
6. What is desired state?
7. What is actual state?
8. What is OutOfSync?
9. What is Synced?
10. What is application health?
11. What is self-healing?
12. What is pruning?
13. What is an ArgoCD Project?
14. What is ApplicationSet?
15. What is the App of Apps pattern?

---

# Intermediate Interview Questions

16. How does ArgoCD work?

17. What are the main ArgoCD components?

18. How do you deploy Helm charts using ArgoCD?

19. How do you deploy Kustomize applications using ArgoCD?

20. How do you configure automated synchronization?

21. How do you troubleshoot an OutOfSync application?

22. How do you troubleshoot a Degraded application?

23. How do you rollback an ArgoCD deployment?

24. How does ArgoCD detect configuration drift?

25. How do you manage multiple environments?

26. How do you manage multiple Kubernetes clusters?

27. How do you secure ArgoCD?

28. How do you integrate ArgoCD with GitHub Actions?

29. How do you integrate ArgoCD with ECR?

30. What happens if ArgoCD is unavailable?

---

# Advanced Interview Questions

31. Design a production GitOps architecture using ArgoCD.

32. How would you deploy microservices using ArgoCD?

33. How would you implement GitOps for multiple EKS clusters?

34. How would you design ArgoCD Projects for multiple teams?

35. How would you secure ArgoCD in an enterprise environment?

36. How would you handle secrets in GitOps?

37. How would you implement rollback?

38. How would you manage Helm charts with ArgoCD?

39. How would you implement promotion across DEV, QA, UAT, and PROD?

40. How would you troubleshoot an ArgoCD synchronization failure?

41. How would you handle Kubernetes configuration drift?

42. How would you design ArgoCD for high availability?

43. How would you integrate ArgoCD with Prometheus and Grafana?

44. How would you implement GitOps for a large microservices platform?

---

# Real-World DevOps Example

Suppose the application contains:

    User
    Product
    Cart
    Orders
    Payment
    Inventory
    Notification

Architecture:

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
        |
        ├── User
        ├── Product
        ├── Cart
        ├── Orders
        ├── Payment
        ├── Inventory
        └── Notification

---

# Real-World Deployment Example

Suppose the Payment service releases:

    payment:1.4.7

CI pipeline:

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
    Image 1.4.7

GitOps update:

    image:
      repository: payment
      tag: "1.4.7"

Git commit:

    Update payment image to 1.4.7

Then:

    Git
      |
      ↓
    ArgoCD
      |
      ↓
    Detect Change
      |
      ↓
    Sync
      |
      ↓
    EKS
      |
      ↓
    Payment 1.4.7

---

# Complete GitHub Actions + ArgoCD Architecture

    Developer
        |
        ↓
    GitHub Application Repository
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Unit Test
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
    GitOps Pull Request
        |
        ↓
    Review
        |
        ↓
    Merge
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Kubernetes Deployment
        |
        ↓
    Pods
        |
        ↓
    Service
        |
        ↓
    ALB / Ingress
        |
        ↓
    Application

---

# Complete GitOps Promotion Flow

    Developer
        |
        ↓
    Source Code
        |
        ↓
    CI
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    DEV GitOps
        |
        ↓
    ArgoCD DEV
        |
        ↓
    Testing
        |
        ↓
    QA GitOps
        |
        ↓
    ArgoCD QA
        |
        ↓
    UAT GitOps
        |
        ↓
    Approval
        |
        ↓
    PROD GitOps
        |
        ↓
    ArgoCD PROD
        |
        ↓
    Production

---

# ArgoCD Complete Architecture

    Developer
        |
        ↓
    Application Repository
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
    Git Commit
        |
        ↓
    ArgoCD
        |
        +-- API Server
        +-- Repo Server
        +-- Application Controller
        +-- Redis
        |
        ↓
    Kubernetes / EKS
        |
        ↓
    Applications
        |
        ↓
    Prometheus
        |
        ↓
    Grafana
        |
        ↓
    ELK Stack

---

# Final ArgoCD Mental Model

Remember:

    Application Code
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
    Docker Image
          |
          ↓
    ECR
          |
          ↓
    GitOps Repository
          |
          ↓
    Desired State
          |
          ↓
        ArgoCD
          |
          ↓
    Compare Desired vs Actual
          |
          +-- Same → Synced
          |
          +-- Different → OutOfSync
          |
          ↓
       Synchronize
          |
          ↓
    Kubernetes / EKS
          |
          ↓
    Application
          |
          ↓
    Health Validation

---

# Final Concept

ArgoCD provides GitOps-based Continuous Delivery for Kubernetes.

The most important concept is:

    Git = Desired State

    Kubernetes = Actual State

    ArgoCD = Reconciliation

Complete process:

    Code
      |
      ↓
    CI
      |
      ↓
    Docker Image
      |
      ↓
    ECR
      |
      ↓
    GitOps Configuration
      |
      ↓
    ArgoCD
      |
      ↓
    Kubernetes
      |
      ↓
    Application

For a production DevOps environment:

    GitHub Actions
        |
        ↓
    Build / Test / Security
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
    Helm
        |
        ↓
    Kubernetes
        |
        ↓
    Prometheus / Grafana / ELK
        |
        ↓
    Observability

ArgoCD therefore provides declarative, version-controlled, continuously reconciled Kubernetes deployments and fits naturally into a modern DevOps and GitOps architecture.