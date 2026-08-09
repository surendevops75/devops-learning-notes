# Rolling Deployment

Rolling Deployment is a deployment strategy where the existing application instances are gradually replaced with instances running the new application version.

Instead of stopping all existing instances and deploying the new version at once, the deployment happens in controlled steps.

Example:

    Current:
        10 Pods
        Version 1.4.6

    Rolling Deployment:

        Replace 2 Pods
            ↓
        New Version

        Replace 2 More Pods
            ↓
        New Version

        Continue
            ↓
        All Pods Updated

Final:

    10 Pods
    Version 1.4.7

---

# Why Rolling Deployment Is Used

Rolling Deployment helps update applications while keeping part of the existing application available.

Benefits include:

    Reduced Downtime
    Gradual Replacement
    Lower Resource Usage
    Controlled Deployment
    Kubernetes Native Support
    Easy Integration With CI/CD
    No Need for a Complete Duplicate Environment

---

# Rolling Deployment Mental Model

Remember:

    Old Version
        |
        ↓
    Replace Some Instances
        |
        ↓
    New Version
        |
        ↓
    Validate
        |
        ↓
    Replace More Instances
        |
        ↓
    Validate
        |
        ↓
    Continue
        |
        ↓
    100% New Version

---

# Rolling Deployment Architecture

Basic architecture:

    Users
      |
      ↓
    Load Balancer
      |
      ↓
    Application
      |
      +-- Pod 1 → v1.4.6
      +-- Pod 2 → v1.4.6
      +-- Pod 3 → v1.4.6
      +-- Pod 4 → v1.4.7
      +-- Pod 5 → v1.4.7

During the deployment, both versions may temporarily exist.

---

# Initial State

Suppose there are 6 Pods.

    Pod 1 → v1.4.6
    Pod 2 → v1.4.6
    Pod 3 → v1.4.6
    Pod 4 → v1.4.6
    Pod 5 → v1.4.6
    Pod 6 → v1.4.6

Traffic:

    Users
      |
      ↓
    Load Balancer
      |
      ↓
    6 Pods
      |
      ↓
    v1.4.6

---

# Rolling Update Starts

Replace one or more Pods.

Example:

    Pod 1 → v1.4.7
    Pod 2 → v1.4.6
    Pod 3 → v1.4.6
    Pod 4 → v1.4.6
    Pod 5 → v1.4.6
    Pod 6 → v1.4.6

Now:

    1 Pod → New
    5 Pods → Old

---

# Rolling Update Continues

Next:

    Pod 1 → v1.4.7
    Pod 2 → v1.4.7
    Pod 3 → v1.4.6
    Pod 4 → v1.4.6
    Pod 5 → v1.4.6
    Pod 6 → v1.4.6

Now:

    2 Pods → New
    4 Pods → Old

---

# Rolling Update Final Stages

Continue:

    3 New
    3 Old

Then:

    4 New
    2 Old

Then:

    5 New
    1 Old

Finally:

    6 New
    0 Old

Result:

    100% → v1.4.7

---

# Rolling Deployment Flow

    v1.4.6
    6 Pods
       |
       ↓
    Replace Pod
       |
       ↓
    1 New + 5 Old
       |
       ↓
    Validate
       |
       ↓
    Replace Pods
       |
       ↓
    2 New + 4 Old
       |
       ↓
    Validate
       |
       ↓
    Continue
       |
       ↓
    6 New
       |
       ↓
    Deployment Complete

---

# Kubernetes Rolling Deployment

Kubernetes Deployments support RollingUpdate strategy.

Example:

    apiVersion: apps/v1
    kind: Deployment

    metadata:
      name: payment

    spec:
      replicas: 6

      strategy:
        type: RollingUpdate

      selector:
        matchLabels:
          app: payment

      template:
        metadata:
          labels:
            app: payment

        spec:
          containers:
            - name: payment
              image: payment:1.4.7

Kubernetes manages the gradual replacement of Pods.

---

# RollingUpdate Strategy

Kubernetes provides two important parameters:

    maxUnavailable
    maxSurge

These control how many Pods can be unavailable and how many additional Pods can temporarily exist during the rollout.

---

# maxUnavailable

`maxUnavailable` defines the maximum number or percentage of Pods that can be unavailable during the rolling update.

Example:

    replicas: 10

    maxUnavailable: 2

This allows up to 2 Pods to be unavailable during the update.

---

# maxSurge

`maxSurge` defines the maximum number or percentage of additional Pods that can be created above the desired replica count during the rolling update.

Example:

    replicas: 10

    maxSurge: 2

Kubernetes may temporarily run up to:

    12 Pods

during the rollout.

---

# maxUnavailable and maxSurge

Example:

    replicas: 10

    maxUnavailable: 2

    maxSurge: 2

During rollout:

    Desired Pods = 10

    Maximum unavailable = 2

    Maximum total Pods = 12

This gives Kubernetes flexibility to replace Pods while maintaining availability.

---

# RollingUpdate Example

Suppose:

    replicas: 10

    maxUnavailable: 1

    maxSurge: 2

Kubernetes can:

    Create new Pods
        |
        ↓
    Wait for readiness
        |
        ↓
    Remove old Pods
        |
        ↓
    Create more new Pods
        |
        ↓
    Continue

The exact sequence depends on Pod readiness and scheduling.

---

# RollingUpdate With Percentage

Values can also be percentages.

Example:

    maxUnavailable: 25%

    maxSurge: 25%

For:

    replicas: 20

This can allow approximately:

    5 unavailable Pods

and:

    5 additional Pods

during the rollout.

---

# Why maxUnavailable Matters

If:

    maxUnavailable = 0

Kubernetes aims to keep all desired replicas available during the update.

This can improve availability but may require additional capacity.

Example:

    10 desired Pods

During rollout:

    Old Pods remain available
        +
    New Pods start

Then old Pods are removed as new Pods become ready.

---

# Why maxSurge Matters

If:

    maxSurge = 0

Kubernetes cannot temporarily exceed the desired replica count.

This can reduce additional resource usage but may require taking old Pods down before creating replacement capacity.

The correct setting depends on application availability and infrastructure capacity.

---

# Rolling Deployment and Readiness

Readiness probes are extremely important.

Flow:

    New Pod
       |
       ↓
    Start Application
       |
       ↓
    Readiness Probe
       |
       +------ Fail ------→ Do Not Receive Traffic
       |
       +------ Pass ------→ Receive Traffic
                              |
                              ↓
                         Continue Rollout

---

# Readiness Probe Example

Example:

    readinessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5

Kubernetes waits for the Pod to become ready before considering it available for traffic.

---

# Liveness Probe and Rolling Deployment

Liveness probes help detect containers that are no longer functioning correctly.

Example:

    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10

If a container repeatedly fails the liveness probe, Kubernetes may restart it.

---

# Startup Probe

Startup probes can be useful for applications that require significant startup time.

Example:

    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 30
      periodSeconds: 10

The startup probe prevents Kubernetes from treating a slow-starting application as immediately unhealthy.

---

# Rolling Deployment and Health Checks

A safe rollout should include:

    Readiness Probe
    Liveness Probe
    Startup Probe
    Application Health Check
    Smoke Test
    Metrics
    Logs

These checks help determine whether the new Pods are healthy.

---

# Rolling Deployment and Services

A Kubernetes Service provides stable networking for Pods.

Example:

    Users
      |
      ↓
    Service
      |
      +-- Old Pod
      +-- Old Pod
      +-- New Pod
      +-- New Pod

As new Pods become ready, they can receive traffic.

As old Pods are terminated, traffic gradually moves to the new Pods.

---

# Rolling Deployment Traffic Flow

Initial:

    Service
       |
       +-- v1.4.6
       +-- v1.4.6
       +-- v1.4.6
       +-- v1.4.6

During rollout:

    Service
       |
       +-- v1.4.6
       +-- v1.4.6
       +-- v1.4.7
       +-- v1.4.7

Final:

    Service
       |
       +-- v1.4.7
       +-- v1.4.7
       +-- v1.4.7
       +-- v1.4.7

---

# Rolling Deployment With EKS

Amazon EKS uses standard Kubernetes Deployment behavior.

Architecture:

    Users
      |
      ↓
    Route 53
      |
      ↓
    ALB
      |
      ↓
    Kubernetes Service
      |
      ↓
    EKS
      |
      +-- Old Pods
      |
      +-- New Pods

During rollout:

    Old Pods
        ↓
    Gradually Replaced
        ↓
    New Pods

---

# Rolling Deployment With ALB

Typical architecture:

    Internet
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

During rollout, the Service continues routing traffic to available healthy Pods.

---

# Rolling Deployment With Ingress

Example:

    Users
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
      +-- v1.4.6
      +-- v1.4.6
      +-- v1.4.7
      +-- v1.4.7

The routing layer remains stable while Pods are replaced.

---

# Rolling Deployment With Helm

Helm can manage Kubernetes Deployments using RollingUpdate.

Example:

    values.yaml

    replicaCount: 6

    image:
      repository: payment
      tag: "1.4.7"

    strategy:
      type: RollingUpdate

Helm renders the Kubernetes resources.

---

# Helm Rolling Deployment Flow

    Git
      |
      ↓
    Helm Values
      |
      ↓
    Helm
      |
      ↓
    Kubernetes Deployment
      |
      ↓
    RollingUpdate
      |
      ↓
    New Pods
      |
      ↓
    Health Checks

---

# Rolling Deployment With ArgoCD

ArgoCD can deploy Kubernetes manifests containing a RollingUpdate strategy.

Flow:

    GitOps Repository
        |
        ↓
    Deployment Manifest
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Kubernetes RollingUpdate
        |
        ↓
    New Pods

ArgoCD monitors the desired state and synchronization status.

---

# GitOps Rolling Deployment

Example:

    Git

    image:
      tag: "1.4.7"

        |
        ↓

    ArgoCD

        |
        ↓

    EKS

        |
        ↓

    RollingUpdate

        |
        ↓

    v1.4.6 → v1.4.7

---

# GitOps Rollback

If the new version fails:

    Git
      |
      ↓
    Revert Change
      |
      ↓
    ArgoCD
      |
      ↓
    Previous Image
      |
      ↓
    EKS
      |
      ↓
    RollingUpdate
      |
      ↓
    Previous Version

In a GitOps model, the Git repository should remain the source of truth.

---

# Kubernetes Rollout Commands

Check deployment:

    kubectl get deployment payment

Check rollout status:

    kubectl rollout status deployment/payment

Check rollout history:

    kubectl rollout history deployment/payment

Rollback:

    kubectl rollout undo deployment/payment

Check Pods:

    kubectl get pods

---

# Start a Rolling Deployment

Updating the image:

    kubectl set image deployment/payment \
      payment=payment:1.4.7

Then:

    kubectl rollout status deployment/payment

Kubernetes starts the rolling update.

---

# Check Rollout History

Command:

    kubectl rollout history deployment/payment

Example:

    deployment.apps/payment
    REVISION  CHANGE-CAUSE
    1         Initial deployment
    2         Updated image to 1.4.6
    3         Updated image to 1.4.7

This helps identify deployment revisions.

---

# Pause a Rolling Deployment

A rollout can be paused:

    kubectl rollout pause deployment/payment

This can be useful when investigating or controlling a rollout.

---

# Resume a Rolling Deployment

Resume:

    kubectl rollout resume deployment/payment

After resuming, Kubernetes continues the rollout.

---

# Rollback a Rolling Deployment

If the new version is unhealthy:

    kubectl rollout undo deployment/payment

Then:

    kubectl rollout status deployment/payment

This returns the Deployment to the previous revision.

---

# Rollback to Specific Revision

Command:

    kubectl rollout undo deployment/payment \
      --to-revision=2

Then:

    kubectl rollout status deployment/payment

---

# Rolling Deployment Failure

Suppose:

    v1.4.6
        |
        ↓
    Rolling Update
        |
        ↓
    v1.4.7
        |
        ↓
    New Pods Fail Readiness
        |
        ↓
    Rollout Stops Progressing

Kubernetes does not automatically assume that every new Pod is healthy.

Readiness and rollout configuration help control the update.

---

# Deployment Status

Check:

    kubectl get deployment payment

Example:

    NAME      READY   UP-TO-DATE   AVAILABLE
    payment   5/6     3            5

This indicates the rollout is not yet fully complete.

---

# ReplicaSets During Rolling Update

Kubernetes Deployments use ReplicaSets.

Example:

    Deployment
        |
        +-- Old ReplicaSet
        |      |
        |      └── v1.4.6
        |
        └-- New ReplicaSet
               |
               └── v1.4.7

During the rollout, both ReplicaSets can temporarily exist.

---

# ReplicaSet Transition

Initial:

    Old ReplicaSet
        |
        └── 6 Pods

During rollout:

    Old ReplicaSet
        |
        └── 4 Pods

    New ReplicaSet
        |
        └── 2 Pods

Final:

    Old ReplicaSet
        |
        └── 0 Pods

    New ReplicaSet
        |
        └── 6 Pods

---

# Rolling Deployment and Pod Disruption

Rolling deployment should consider:

    PodDisruptionBudget

A PodDisruptionBudget can help maintain a minimum number of available Pods during voluntary disruptions.

Example:

    minAvailable: 5

This is useful for highly available applications.

---

# PodDisruptionBudget

Conceptual example:

    apiVersion: policy/v1
    kind: PodDisruptionBudget

    metadata:
      name: payment-pdb

    spec:
      minAvailable: 5

      selector:
        matchLabels:
          app: payment

PDBs help protect availability during certain voluntary disruptions.

---

# Rolling Deployment and HPA

Horizontal Pod Autoscaler can scale the application while a rollout is happening.

Architecture:

    HPA
      |
      ↓
    Deployment
      |
      +-- Old Pods
      |
      +-- New Pods

Scaling and rollout behavior should be tested together.

---

# Rolling Deployment and Resource Limits

New Pods require sufficient resources.

Example:

    CPU Request
    Memory Request
    CPU Limit
    Memory Limit

If nodes do not have enough capacity:

    New Pod
       |
       ↓
    Pending
       |
       ↓
    Rollout Delayed

Therefore cluster capacity should be considered before deployment.

---

# Rolling Deployment and Cluster Capacity

Suppose:

    Desired Pods = 10

    maxSurge = 3

During rollout Kubernetes may need:

    13 Pods

If the cluster can only run:

    10 Pods

The rollout may become constrained.

Therefore:

    Cluster Capacity
        +
    maxSurge
        +
    Replica Count

must be considered together.

---

# Rolling Deployment and Node Capacity

Example:

    EKS Node Capacity:
        10 Pods

Deployment:

    replicas = 10

maxSurge:

    2

Potential requirement:

    12 Pods

If there is no additional capacity:

    New Pods may remain Pending.

Possible solutions:

    Add Node Capacity
    Use Cluster Autoscaler
    Adjust maxSurge
    Optimize Resource Requests

---

# Rolling Deployment and Cluster Autoscaling

When additional Pods are required:

    RollingUpdate
        |
        ↓
    New Pod Pending
        |
        ↓
    Cluster Capacity Check
        |
        ↓
    Autoscaling
        |
        ↓
    New Node
        |
        ↓
    New Pod Scheduled
        |
        ↓
    Rollout Continues

---

# Rolling Deployment and PDB

PDB and RollingUpdate solve different problems.

RollingUpdate controls:

    Deployment Replacement

PDB controls:

    Voluntary Disruptions

Both should be considered when designing highly available Kubernetes applications.

---

# Rolling Deployment and Graceful Shutdown

During Pod termination, the application should shut down gracefully.

Typical flow:

    Pod Receives Termination
        |
        ↓
    Stop Receiving New Traffic
        |
        ↓
    Graceful Shutdown
        |
        ↓
    Finish Existing Requests
        |
        ↓
    Process Terminates

This helps prevent interrupted requests.

---

# terminationGracePeriodSeconds

Kubernetes provides:

    terminationGracePeriodSeconds

Example:

    spec:
      terminationGracePeriodSeconds: 30

This gives the application time to shut down gracefully.

The appropriate value depends on the application.

---

# PreStop Hook

A lifecycle hook can perform actions before container termination.

Example:

    lifecycle:
      preStop:
        exec:
          command:
            - /bin/sh
            - -c
            - "sleep 10"

This should be designed carefully and should not be used as a substitute for proper readiness and graceful shutdown behavior.

---

# Rolling Deployment and Connection Draining

Load balancers may need time to stop sending traffic to terminating instances.

Example:

    Pod
      |
      ↓
    Termination
      |
      ↓
    Connection Draining
      |
      ↓
    Existing Requests Complete
      |
      ↓
    Pod Removed

This helps minimize dropped connections.

---

# Rolling Deployment and ALB Target Health

ALB health checks help determine whether targets are healthy.

Flow:

    New Pod
      |
      ↓
    ALB Health Check
      |
      +------ Unhealthy ------→ No Production Traffic
      |
      +------ Healthy --------→ Traffic Allowed

Correct health check configuration is critical.

---

# Rolling Deployment and Application Startup

Suppose an application takes 60 seconds to start.

Without proper readiness handling:

    New Pod Starts
        |
        ↓
    Traffic Immediately
        |
        ↓
    Application Not Ready
        |
        ↓
    Errors

With readiness:

    New Pod Starts
        |
        ↓
    Readiness Fails
        |
        ↓
    No Traffic
        |
        ↓
    Application Ready
        |
        ↓
    Readiness Pass
        |
        ↓
    Traffic

---

# Rolling Deployment and Startup Time

Slow applications should use appropriate:

    startupProbe
    readinessProbe
    terminationGracePeriodSeconds

The goal is to ensure Kubernetes understands:

    When the application is starting
    When it is ready
    When it is unhealthy
    How much time it needs to shut down

---

# Rolling Deployment and Zero Downtime

Rolling deployment can support zero or near-zero downtime when:

    Multiple Replicas Exist
        +
    Readiness Probes Work
        +
    Graceful Shutdown Works
        +
    Service Routing Works
        +
    Sufficient Capacity Exists
        +
    Application Supports Concurrent Versions

---

# Zero-Downtime Rolling Deployment

Example:

    6 Healthy Pods
        |
        ↓
    Create New Pod
        |
        ↓
    New Pod Ready
        |
        ↓
    Remove Old Pod
        |
        ↓
    Create Next New Pod
        |
        ↓
    Continue

At least the required number of healthy Pods remain available.

---

# Rolling Deployment and API Compatibility

During the rollout:

    Old Version
        +
    New Version

may run simultaneously.

Therefore APIs should ideally remain backward compatible.

Example:

    v1.4.6 → API v1
    v1.4.7 → API v1

This is safer than:

    v1.4.6 → API v1
    v1.4.7 → Completely Incompatible API

---

# Rolling Deployment and Microservices

For microservices:

    User
    Product
    Cart
    Orders
    Payment
    Inventory
    Notification

Each service can be rolled out independently.

Example:

    Payment
      |
      ↓
    Rolling Deployment
      |
      ↓
    v1.4.6 → v1.4.7

Other services may remain unchanged.

---

# Microservices Rolling Deployment

Example:

    User
      |
      └── v1.4.6

    Product
      |
      └── v1.4.6

    Payment
      |
      ├── v1.4.6
      └── v1.4.7

Only Payment is being updated.

This reduces the blast radius compared with updating every service simultaneously.

---

# Rolling Deployment and Service Compatibility

Suppose:

    Payment v1.4.6

and:

    Orders v1.4.7

During deployment, these versions may communicate.

Therefore:

    API Compatibility
    Contract Compatibility
    Database Compatibility

should be considered.

---

# Rolling Deployment and Database Changes

Database changes must support both old and new application versions when they coexist.

Safer approach:

    Expand
       |
       ↓
    Deploy Compatible Schema
       |
       ↓
    Rolling Application Update
       |
       ↓
    Validate
       |
       ↓
    Contract Later

---

# Expand and Contract

Example:

    Step 1
        |
        ↓
    Add New Database Column

    Step 2
        |
        ↓
    Deploy Application That Supports Both

    Step 3
        |
        ↓
    Rolling Deployment

    Step 4
        |
        ↓
    Migrate Data

    Step 5
        |
        ↓
    Remove Old Column Later

This reduces rollback risk.

---

# Rolling Deployment and ConfigMap

Configuration changes can trigger Pod replacement.

Example:

    ConfigMap
        |
        ↓
    Application
        |
        ↓
    New Configuration

Validate configuration before the rollout.

Incorrect configuration can cause:

    CrashLoopBackOff
    Failed Readiness
    Connection Errors
    Application Failure

---

# Rolling Deployment and Secrets

Secret changes should be handled carefully.

Check:

    Secret Name
    Secret Key
    Application Configuration
    Permissions
    Secret Availability

A new application version may expect a different secret.

---

# Rolling Deployment and ImagePullBackOff

If new Pods show:

    ImagePullBackOff

Check:

    kubectl describe pod <pod>

Possible causes:

    Wrong Image Tag
    Missing ECR Image
    IAM Permission
    Registry Authentication
    Network Problem

The rollout may not complete until the issue is resolved.

---

# Rolling Deployment and CrashLoopBackOff

If new Pods show:

    CrashLoopBackOff

Check:

    kubectl logs <pod>

    kubectl logs <pod> --previous

    kubectl describe pod <pod>

Possible causes:

    Application Error
    Configuration Error
    Missing Secret
    Dependency Failure
    Resource Problem

---

# Rolling Deployment and OOMKilled

If the new Pods are:

    OOMKilled

Check:

    Memory Requests
    Memory Limits
    Application Memory Usage
    Traffic
    Memory Leaks

A rollout should not continue blindly when the new version consistently exceeds memory limits.

---

# Rolling Deployment and Failed Probes

If readiness fails:

    kubectl describe pod <pod>

Check:

    Events
    Probe Path
    Probe Port
    Application Logs
    Service
    Network
    Startup Time

Do not increase rollout speed until the underlying problem is understood.

---

# Rolling Deployment Monitoring

Monitor:

    Deployment Status
    ReplicaSets
    Pods
    Readiness
    Liveness
    Error Rate
    Latency
    CPU
    Memory
    Logs
    ALB Health
    Application Health

---

# Prometheus Monitoring

Prometheus can collect:

    Request Rate
    Error Rate
    Latency
    CPU
    Memory
    Pod Metrics

Example:

    Rolling Deployment
          |
          ↓
      Prometheus
          |
          ↓
       Metrics
          |
          ↓
       Grafana

---

# Grafana Dashboard

A deployment dashboard can show:

    Deployment Status

    Available Replicas

    Ready Replicas

    Error Rate

    P95 Latency

    CPU Usage

    Memory Usage

    Pod Restarts

This helps identify rollout problems quickly.

---

# ELK Monitoring

ELK can be used for application logs.

Example:

    New Pods
       |
       ↓
    Application Logs
       |
       ↓
    ELK
       |
       ↓
    Search Errors

Look for:

    Exceptions
    Connection Errors
    HTTP 5xx
    Authentication Failures
    Database Errors

---

# Rolling Deployment and Observability

Complete observability:

    Kubernetes
        |
        +-- Metrics → Prometheus
        |
        +-- Dashboards → Grafana
        |
        └-- Logs → ELK

During deployment:

    Old Version
        vs
    New Version

Compare health and behavior.

---

# Rolling Deployment With GitHub Actions

Typical pipeline:

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
    Deployment Update
       |
       ↓
    EKS
       |
       ↓
    RollingUpdate
       |
       ↓
    Validation
       |
       ↓
    Production

---

# GitHub Actions Rolling Pipeline

Example flow:

    Code
      |
      ↓
    Pull Request
      |
      ↓
    CI
      |
      +-- Maven Build
      +-- Unit Tests
      +-- SonarQube
      +-- Trivy
      |
      ↓
    Docker Build
      |
      ↓
    Push to ECR
      |
      ↓
    Update GitOps Repository
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    Rolling Deployment
      |
      ↓
    Health Checks
      |
      ↓
    Smoke Tests

---

# Rolling Deployment With Jenkins

Typical flow:

    Jenkins
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
    Docker Build
       |
       ↓
    ECR
       |
       ↓
    Deploy
       |
       ↓
    Kubernetes RollingUpdate
       |
       ↓
    Validation

---

# Rolling Deployment With ArgoCD

Architecture:

    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Deployment
        |
        ↓
    RollingUpdate
        |
        ↓
    New Pods
        |
        ↓
    Health Checks

ArgoCD monitors synchronization and application health.

---

# Rolling Deployment With Terraform

Terraform is normally used to provision infrastructure rather than manage every application rollout.

Terraform can provision:

    VPC
    EKS
    IAM
    ALB
    Security Groups
    ECR
    Networking

Application deployment can then be handled by:

    Kubernetes
    Helm
    ArgoCD
    CI/CD

This separates infrastructure provisioning from application delivery.

---

# Rolling Deployment Architecture

    Terraform
       |
       ↓
    AWS Infrastructure
       |
       +-- VPC
       +-- EKS
       +-- IAM
       +-- ECR
       +-- ALB
       |
       ↓
    GitHub Actions
       |
       ↓
    Docker Image
       |
       ↓
    GitOps
       |
       ↓
    ArgoCD
       |
       ↓
    Kubernetes RollingUpdate

---

# Rolling Deployment and Docker

Each release should use a unique image version.

Example:

    payment:1.4.6

New:

    payment:1.4.7

Kubernetes updates the Deployment:

    payment:1.4.6
        |
        ↓
    payment:1.4.7

Avoid relying on:

    payment:latest

---

# Immutable Docker Image

Good:

    payment:1.4.7

Better for traceability:

    payment:<commit-sha>

The image should correspond to a specific source version.

This makes rollback easier.

---

# Rolling Deployment and Version Tracking

Track:

    Git Commit
    Docker Image
    Deployment Revision
    Helm Release
    ArgoCD Sync
    Application Version

Example:

    Git Commit
        |
        ↓
    abc1234
        |
        ↓
    payment:1.4.7
        |
        ↓
    Kubernetes Revision
        |
        ↓
    Production

---

# Rolling Deployment Rollback

If version 1.4.7 fails:

    v1.4.6
        |
        ↓
    Rolling Update
        |
        ↓
    v1.4.7
        |
        ↓
    Failure
        |
        ↓
    kubectl rollout undo
        |
        ↓
    v1.4.6

In GitOps:

    Revert Git Commit
        |
        ↓
    ArgoCD
        |
        ↓
    v1.4.6

---

# Rolling Deployment Rollback Commands

Check:

    kubectl rollout history deployment/payment

Rollback:

    kubectl rollout undo deployment/payment

Monitor:

    kubectl rollout status deployment/payment

Validate:

    kubectl get pods

---

# Rolling Deployment Pause and Investigation

If rollout is progressing incorrectly:

    kubectl rollout pause deployment/payment

Then:

    kubectl get pods

    kubectl describe deployment payment

    kubectl describe pod <pod>

    kubectl logs <pod>

After fixing:

    kubectl rollout resume deployment/payment

---

# Rolling Deployment Failure Handling

Failure flow:

    New Version
        |
        ↓
    Rolling Update
        |
        ↓
    New Pod Failure
        |
        ↓
    Detect
        |
        ↓
    Stop / Pause
        |
        ↓
    Investigate
        |
        +------ Fix → Resume
        |
        └------ Release Bad → Rollback

---

# Rolling Deployment Scenario

## Scenario: New Pods Fail Readiness

Current:

    6 Pods
    v1.4.6

New release:

    v1.4.7

During rollout:

    4 Pods → v1.4.6
    2 Pods → v1.4.7

New Pods:

    Readiness Probe Failed

Action:

    Check:

    kubectl describe pod <pod>

    kubectl logs <pod>

Then:

    Fix Issue
        |
        ↓
    Resume

or:

    Rollback
        |
        ↓
    v1.4.6

---

# Rolling Deployment Scenario

## Scenario: New Image Causes CrashLoopBackOff

Deployment:

    v1.4.7

New Pods:

    CrashLoopBackOff

Check:

    kubectl logs <pod> --previous

Then:

    kubectl describe pod <pod>

If caused by the release:

    kubectl rollout undo deployment/payment

Validate:

    kubectl rollout status deployment/payment

---

# Rolling Deployment Scenario

## Scenario: New Version Causes High Latency

Before:

    P95 = 150ms

After rollout begins:

    P95 = 800ms

Action:

    Stop rollout
        |
        ↓
    Check Metrics
        |
        ↓
    Check Logs
        |
        ↓
    Check Database
        |
        ↓
    Check Dependencies
        |
        ↓
    Rollback if required

---

# Rolling Deployment Scenario

## Scenario: ImagePullBackOff

New Pods:

    ImagePullBackOff

Check:

    kubectl describe pod <pod>

Possible cause:

    payment:1.4.7

does not exist in ECR.

Action:

    Verify ECR
        |
        ↓
    Verify Image Tag
        |
        ↓
    Fix Image Reference
        |
        ↓
    Retry Deployment

---

# Rolling Deployment Scenario

## Scenario: Insufficient Node Capacity

Deployment:

    replicas = 10

    maxSurge = 3

New Pods:

    Pending

Reason:

    Insufficient CPU / Memory

Action:

    Check:

    kubectl describe pod <pod>

Then:

    Add Capacity
    or
    Adjust Resource Requests
    or
    Adjust maxSurge

---

# Rolling Deployment Scenario

## Scenario: Database Migration Problem

New version:

    v1.4.7

Database migration:

    Incompatible Schema

During rollout:

    Old Pods + New Pods

One version may fail.

Solution:

    Use backward-compatible migration

    Expand
       |
       ↓
    Deploy
       |
       ↓
    Migrate
       |
       ↓
    Contract Later

---

# Rolling Deployment Scenario

## Scenario: Secret Changed

New version requires:

    PAYMENT_API_KEY

But the Secret is missing.

New Pods:

    CrashLoopBackOff

Check:

    kubectl get secret

    kubectl describe pod <pod>

Fix the Secret or deployment configuration before continuing.

---

# Rolling Deployment and Production Safety

Before deployment:

    Verify Image
    Verify Configuration
    Verify Secrets
    Verify Database Compatibility
    Verify Capacity
    Verify Probes
    Verify Monitoring
    Verify Rollback

During deployment:

    Monitor Pods
    Monitor Metrics
    Monitor Logs
    Monitor Application

After deployment:

    Smoke Tests
    Health Checks
    Metrics
    Logs

---

# Rolling Deployment Checklist

Before deployment:

    [ ] Image exists
    [ ] Image tag is immutable
    [ ] Tests passed
    [ ] Security scans passed
    [ ] Configuration verified
    [ ] Secrets verified
    [ ] Database compatibility verified
    [ ] Resource capacity available
    [ ] Probes configured
    [ ] Monitoring ready
    [ ] Rollback ready

During deployment:

    [ ] Pods becoming Ready
    [ ] No CrashLoopBackOff
    [ ] No ImagePullBackOff
    [ ] Error rate normal
    [ ] Latency normal
    [ ] Logs normal

After deployment:

    [ ] All replicas updated
    [ ] All replicas available
    [ ] Smoke tests passed
    [ ] Application healthy
    [ ] Metrics normal

---

# Rolling Deployment Best Practices

- Use multiple replicas
- Configure readiness probes
- Configure liveness probes appropriately
- Use startup probes for slow applications
- Use graceful shutdown
- Configure maxUnavailable carefully
- Configure maxSurge carefully
- Maintain sufficient cluster capacity
- Use immutable image tags
- Use backward-compatible APIs
- Use backward-compatible database migrations
- Monitor during rollout
- Test rollback
- Use PodDisruptionBudgets where appropriate
- Use automated health checks
- Use GitOps for controlled deployment
- Keep deployment history
- Avoid manual production changes

---

# Rolling Deployment Anti-Patterns

## One Replica

Bad:

    replicas: 1

Rolling updates provide limited availability when only one instance exists.

Better:

    replicas: 3

or more according to application requirements.

---

# Rolling Deployment Anti-Pattern

## No Readiness Probe

Bad:

    New Pod Starts
        |
        ↓
    Traffic Immediately
        |
        ↓
    Application Still Starting
        |
        ↓
    Errors

Better:

    New Pod Starts
        |
        ↓
    Readiness Probe
        |
        ↓
    Ready
        |
        ↓
    Traffic

---

# Rolling Deployment Anti-Pattern

## maxSurge Too High

Suppose:

    replicas = 100

    maxSurge = 100

Potentially:

    200 Pods

This can create unnecessary resource consumption.

Choose values based on:

    Capacity
    Application Size
    Deployment Speed
    Availability Requirements

---

# Rolling Deployment Anti-Pattern

## maxUnavailable Too High

Suppose:

    replicas = 10

    maxUnavailable = 8

Only a small number of Pods may remain available during the rollout.

This can create:

    Reduced Capacity
    Increased Latency
    Availability Risk

Use appropriate availability limits.

---

# Rolling Deployment Anti-Pattern

## Mutable Tags

Bad:

    app:latest

Better:

    app:1.4.7

or:

    app:<commit-sha>

---

# Rolling Deployment Anti-Pattern

## Destructive Database Migration

Bad:

    Remove Column
        |
        ↓
    Rolling Deployment
        |
        ↓
    Old Pods Fail

Better:

    Expand
        |
        ↓
    Rolling Deployment
        |
        ↓
    Validate
        |
        ↓
    Contract

---

# Rolling Deployment Anti-Pattern

## No Capacity Planning

Bad:

    replicas = 10
    maxSurge = 5

but:

    Cluster Capacity = 10

New Pods become Pending.

Better:

    Capacity Planning
        +
    Resource Requests
        +
    maxSurge
        +
    Autoscaling

---

# Rolling Deployment Anti-Pattern

## No Monitoring

Bad:

    Start Deployment
        |
        ↓
    Wait
        |
        ↓
    Assume Success

Better:

    Start
        |
        ↓
    Monitor
        |
        ↓
    Validate
        |
        ↓
    Continue

---

# Rolling Deployment vs Blue-Green

| Feature | Rolling | Blue-Green |
|---|---|---|
| Two full environments | No | Yes |
| Gradual replacement | Yes | No |
| Fast rollback | Good | Very Fast |
| Resource Usage | Lower | Higher |
| Traffic Switch | Gradual Pod replacement | Environment switch |
| Production validation before traffic | Limited | Strong |
| Kubernetes Native | Yes | Requires design |
| Complexity | Lower | Higher |

---

# Rolling Deployment vs Canary

| Feature | Rolling | Canary |
|---|---|---|
| Main idea | Replace instances gradually | Gradually expose traffic |
| Traffic percentage control | Limited | Strong |
| Multiple versions | Temporary | Yes |
| Progressive traffic | Not necessarily | Yes |
| Monitoring | Required | Critical |
| Rollback | Supported | Supported |
| Complexity | Lower | Higher |

---

# Rolling Deployment vs Recreate

Recreate:

    Stop Old Version
        |
        ↓
    Deploy New Version

Possible result:

    Downtime

Rolling:

    Old Pods
        |
        ↓
    New Pods
        |
        ↓
    Gradual Replacement

Possible result:

    Minimal Downtime

---

# Rolling Deployment vs Blue-Green vs Canary

    Rolling
        |
        ↓
    Replace Instances Gradually


    Blue-Green
        |
        ↓
    Maintain Two Environments
        |
        ↓
    Switch Traffic


    Canary
        |
        ↓
    Gradually Increase Traffic

Remember:

    Rolling = Instance Replacement

    Blue-Green = Environment Switching

    Canary = Progressive Traffic

---

# Rolling Deployment Strategy Selection

Choose Rolling when:

    You want simple Kubernetes-native deployments
    You want lower infrastructure overhead
    You have multiple replicas
    The application supports mixed versions
    Gradual instance replacement is acceptable

Choose Blue-Green when:

    Fast rollback is critical
    Two environments are affordable
    Strong pre-production validation is required

Choose Canary when:

    Gradual user exposure is important
    Strong observability is available
    Traffic control is available
    Progressive delivery is required

---

# Rolling Deployment Real-World Example

Suppose a microservices platform contains:

    User
    Product
    Cart
    Orders
    Payment
    Inventory
    Notification

Payment currently runs:

    payment:1.4.6

New version:

    payment:1.4.7

Rolling update:

    1. Deploy v1.4.7
    2. Create new Pod
    3. Wait for readiness
    4. Route traffic to new Pod
    5. Remove old Pod
    6. Create next new Pod
    7. Repeat
    8. Complete rollout
    9. Validate
    10. Monitor

Other services remain unchanged.

---

# Real-World EKS Rolling Architecture

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
    Docker
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
    Deployment
        |
        ↓
    RollingUpdate
        |
        +-- Old Pods
        |
        +-- New Pods
        |
        ↓
    Kubernetes Service
        |
        ↓
    ALB
        |
        ↓
    Users

---

# Real-World Rolling Deployment

Initial:

    Payment v1.4.6

    Pod 1 → v1.4.6
    Pod 2 → v1.4.6
    Pod 3 → v1.4.6
    Pod 4 → v1.4.6

Deploy:

    Payment v1.4.7

Stage 1:

    Pod 1 → v1.4.7
    Pod 2 → v1.4.6
    Pod 3 → v1.4.6
    Pod 4 → v1.4.6

Stage 2:

    Pod 1 → v1.4.7
    Pod 2 → v1.4.7
    Pod 3 → v1.4.6
    Pod 4 → v1.4.6

Stage 3:

    Pod 1 → v1.4.7
    Pod 2 → v1.4.7
    Pod 3 → v1.4.7
    Pod 4 → v1.4.6

Final:

    Pod 1 → v1.4.7
    Pod 2 → v1.4.7
    Pod 3 → v1.4.7
    Pod 4 → v1.4.7

---

# Real-World Rollback

Suppose:

    v1.4.7
        |
        ↓
    High Error Rate

Rollback:

    kubectl rollout undo deployment/payment

Kubernetes restores the previous revision.

Then:

    kubectl rollout status deployment/payment

Finally:

    Validate
    Monitor
    Confirm Recovery

---

# Complete Rolling DevOps Architecture

    Developer
        |
        ↓
    Application Repository
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Unit Tests
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
    Kubernetes Deployment
        |
        ↓
    RollingUpdate
        |
        +-----------------------+
        |                       |
        ↓                       ↓
    Old ReplicaSet         New ReplicaSet
        |                       |
        ↓                       ↓
    Old Pods                New Pods
        |                       |
        +-----------+-----------+
                    |
                    ↓
               Kubernetes
                 Service
                    |
                    ↓
                   ALB
                    |
                    ↓
                  Users

---

# Complete Rolling Deployment Flow

    Code
      |
      ↓
    Pull Request
      |
      ↓
    CI
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
    GitOps Update
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    RollingUpdate
      |
      ↓
    New Pod
      |
      ↓
    Readiness Check
      |
      +------ Fail ------→ Stop / Investigate
      |
      +------ Pass
              |
              ↓
         Receive Traffic
              |
              ↓
          Remove Old Pod
              |
              ↓
          Next Pod
              |
              ↓
          Repeat
              |
              ↓
        All Pods Updated
              |
              ↓
        Smoke Tests
              |
              ↓
         Monitoring
              |
              ↓
       Deployment Complete

---

# Rolling Deployment Decision Flow

    New Version Ready
          |
          ↓
    Verify Image
          |
          ↓
    Verify Configuration
          |
          ↓
    Verify Database Compatibility
          |
          ↓
    Deploy
          |
          ↓
    New Pod Created
          |
          ↓
    Readiness Check
          |
          +------ Fail ------→ Stop Rollout
          |
          +------ Pass
                    |
                    ↓
               Traffic
                    |
                    ↓
               Remove Old
                    |
                    ↓
               Next Pod
                    |
                    ↓
                  Repeat
                    |
                    ↓
              All Updated
                    |
                    ↓
               Validation
                    |
                    +------ Fail → Rollback
                    |
                    +------ Pass
                              |
                              ↓
                           Success

---

# Final Rolling Deployment Mental Model

Remember:

    OLD VERSION
        |
        ↓
    Create NEW POD
        |
        ↓
    NEW POD READY?
        |
        +------ No ------→ Investigate
        |
        +------ Yes
              |
              ↓
         Receive Traffic
              |
              ↓
         Remove OLD POD
              |
              ↓
         Create NEXT POD
              |
              ↓
            Repeat
              |
              ↓
       100% NEW VERSION

Rollback:

    New Version
        |
        ↓
    Failure
        |
        ↓
    kubectl rollout undo
        |
        ↓
    Previous Revision
        |
        ↓
    Validate
        |
        ↓
    Recovery

---

# Final Concept

Rolling Deployment gradually replaces old application instances with new instances while maintaining application availability.

The complete DevOps model is:

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
    Immutable Image
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
    RollingUpdate
      |
      ↓
    New Pod
      |
      ↓
    Readiness
      |
      ↓
    Traffic
      |
      ↓
    Remove Old Pod
      |
      ↓
    Repeat
      |
      ↓
    All Pods Updated
      |
      ↓
    Health Validation
      |
      ↓
    Monitoring

A production-ready Rolling Deployment strategy should combine:

    Multiple Replicas
        +
    RollingUpdate
        +
    maxUnavailable
        +
    maxSurge
        +
    Readiness Probes
        +
    Liveness Probes
        +
    Startup Probes
        +
    Graceful Shutdown
        +
    Sufficient Capacity
        +
    Immutable Images
        +
    Backward-Compatible APIs
        +
    Backward-Compatible Database Changes
        +
    Prometheus
        +
    Grafana
        +
    ELK
        +
    GitOps
        +
    ArgoCD
        +
    Fast Rollback

The main goal is:

    Replace Gradually
        |
        ↓
    Keep Application Healthy
        |
        ↓
    Validate New Pods
        |
        ↓
    Continue Safely
        |
        ↓
    Complete Deployment

If the new version fails:

    Stop
      |
      ↓
    Investigate
      |
      ↓
    Rollback
      |
      ↓
    Restore Previous Version
      |
      ↓
    Validate
      |
      ↓
    Recover

Rolling Deployment is one of the most commonly used Kubernetes deployment strategies because it provides controlled application updates without requiring a complete duplicate production environment.