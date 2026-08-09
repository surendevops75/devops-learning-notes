# Kubernetes Deployment in CI/CD

Kubernetes is a container orchestration platform used to deploy, manage, scale, and maintain containerized applications.

In a CD pipeline, Kubernetes is commonly the target platform where container images are deployed after they pass CI validation.

A typical flow is:

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
        +-- Security Scan
        |
        ↓
    Docker Image
        |
        ↓
    Container Registry
        |
        ↓
    Kubernetes
        |
        ↓
    Application

---

# Why Kubernetes Is Used for CD

Kubernetes provides capabilities such as:

- Container orchestration
- Automated scheduling
- Self-healing
- Scaling
- Service discovery
- Rolling updates
- Rollbacks
- Load balancing
- Declarative configuration
- High availability

Instead of manually deploying containers to individual servers, the CD pipeline can submit the desired Kubernetes configuration.

---

# Kubernetes Deployment Architecture

A typical deployment architecture is:

    GitHub
       |
       ↓
    GitHub Actions
       |
       ↓
    Docker Build
       |
       ↓
    Security Scan
       |
       ↓
    Container Registry
       |
       ↓
    Kubernetes Cluster
       |
       +-- Deployment
       +-- Service
       +-- ConfigMap
       +-- Secret
       +-- Ingress
       |
       ↓
    Application Pods

---

# Kubernetes Cluster

A Kubernetes cluster consists of control plane components and worker nodes.

Conceptually:

    Kubernetes Cluster
          |
          +----------------------+
          |                      |
          ↓                      ↓
    Control Plane           Worker Nodes
                                |
                    +-----------+-----------+
                    |           |           |
                    ↓           ↓           ↓
                   Pod         Pod         Pod

The control plane manages the cluster while worker nodes run application workloads.

---

# Kubernetes Control Plane

The control plane is responsible for managing the desired state of the cluster.

Important components include:

- API Server
- Scheduler
- Controller Manager
- etcd

Conceptually:

    kubectl
       |
       ↓
    API Server
       |
       +-- Scheduler
       +-- Controllers
       +-- etcd

---

# Kubernetes API Server

The API Server is the primary interface to the Kubernetes control plane.

Tools and applications communicate with Kubernetes through the API Server.

Example:

    GitHub Actions
        |
        ↓
    kubectl
        |
        ↓
    Kubernetes API Server
        |
        ↓
    Cluster

---

# Kubernetes Scheduler

The Scheduler determines where newly created Pods should run.

Example:

    New Pod
       |
       ↓
    Scheduler
       |
       +---- Node 1
       |
       +---- Node 2
       |
       +---- Node 3

The Scheduler considers resource availability and scheduling constraints.

---

# Kubernetes Controllers

Controllers continuously compare the desired state with the current state.

Example:

    Desired:
        3 Pods

    Current:
        2 Pods

    Controller
        |
        ↓
    Create 1 Pod

Final state:

    Desired:
        3 Pods

    Current:
        3 Pods

---

# etcd

etcd is the distributed key-value store used by Kubernetes to store cluster state.

Conceptually:

    Kubernetes API Server
            |
            ↓
          etcd
            |
            ↓
      Cluster State

The control plane uses this state to manage the cluster.

---

# Worker Nodes

Worker nodes run application workloads.

A node can contain:

- kubelet
- Container runtime
- kube-proxy
- Pods

Conceptually:

    Worker Node
        |
        +-- kubelet
        +-- Container Runtime
        +-- kube-proxy
        |
        +-- Pod
        +-- Pod
        +-- Pod

---

# Pod

A Pod is the smallest deployable unit in Kubernetes.

A Pod can contain one or more containers.

Typical architecture:

    Pod
      |
      +-- Application Container
      |
      +-- Sidecar Container

For many applications:

    Pod
      |
      +-- One Application Container

---

# Kubernetes Deployment

A Deployment manages replicated application Pods.

Example:

    Deployment
        |
        +-- Pod
        +-- Pod
        +-- Pod

If one Pod fails:

    Deployment
        |
        +-- Pod
        +-- Pod
        +-- FAILED Pod
                 |
                 ↓
             New Pod

The Deployment works with Kubernetes controllers to maintain the desired number of replicas.

---

# Deployment YAML

A typical Kubernetes Deployment contains:

    apiVersion
    kind
    metadata
    spec

Example structure:

    apiVersion: apps/v1

    kind: Deployment

    metadata:
      name: myapp

    spec:
      replicas: 3

      selector:
        matchLabels:
          app: myapp

      template:
        metadata:
          labels:
            app: myapp

        spec:
          containers:
            - name: myapp
              image: myapp:1.4.7

---

# Kubernetes Desired State

Kubernetes follows a declarative model.

You define:

    Desired State

Kubernetes continuously works toward:

    Desired State

Example:

    Desired:
        replicas = 3

    Current:
        replicas = 2

Kubernetes creates another Pod.

---

# Declarative Deployment

Instead of telling Kubernetes every individual action:

    Create Pod 1
    Create Pod 2
    Create Pod 3

You define:

    replicas: 3

Kubernetes determines how to achieve that state.

---

# Kubernetes Service

A Service provides stable networking for Pods.

Example:

    Client
      |
      ↓
    Service
      |
      +-- Pod
      +-- Pod
      +-- Pod

Pods can be recreated and receive different IP addresses, but the Service provides a stable endpoint.

---

# Kubernetes Service Types

Common Service types include:

- ClusterIP
- NodePort
- LoadBalancer
- ExternalName

The appropriate type depends on how the application needs to be exposed.

---

# ClusterIP

ClusterIP provides internal cluster access.

Example:

    Application A
        |
        ↓
    ClusterIP Service
        |
        +-- Pod
        +-- Pod
        +-- Pod

It is commonly used for internal microservice communication.

---

# NodePort

NodePort exposes a Service through a port on each node.

Conceptually:

    External Client
          |
          ↓
       NodePort
          |
          ↓
       Service
          |
          ↓
         Pods

It is generally less preferred than a managed load balancer for production internet-facing applications.

---

# LoadBalancer

LoadBalancer exposes a Service through an external load balancer when supported by the cloud provider.

Example:

    Internet
       |
       ↓
    Load Balancer
       |
       ↓
    Kubernetes Service
       |
       ↓
    Pods

---

# Ingress

Ingress provides HTTP/HTTPS routing into Kubernetes.

Example:

    Internet
       |
       ↓
    ALB / Ingress
       |
       +---- /users
       |       ↓
       |     Users Service
       |
       +---- /products
       |       ↓
       |     Product Service
       |
       +---- /orders
               ↓
            Order Service

In AWS EKS environments, AWS Load Balancer Controller can be used to provision and manage Application Load Balancers from Kubernetes resources.

---

# Kubernetes ConfigMap

ConfigMap stores non-sensitive configuration.

Examples:

    APP_ENV
    LOG_LEVEL
    API_URL

Conceptually:

    ConfigMap
       |
       ↓
    Application Pod
       |
       ↓
    Environment Variables

Sensitive information should not be stored as plain ConfigMap data.

---

# Kubernetes Secret

Secret is used to store sensitive configuration.

Examples:

    Database Password
    API Token
    Credentials

Conceptually:

    Secret
       |
       ↓
    Application Pod

Secret handling should still follow appropriate security practices because Kubernetes Secrets are not automatically equivalent to a fully managed external secret-management system.

---

# Kubernetes Namespace

Namespaces provide logical separation inside a cluster.

Example:

    Cluster
       |
       +-- dev
       +-- qa
       +-- uat
       +-- prod

Applications and resources can be organized by namespace.

---

# Multi-Environment Kubernetes

A common environment structure is:

    Kubernetes
       |
       +-- DEV
       |    |
       |    +-- Application
       |
       +-- QA
       |    |
       |    +-- Application
       |
       +-- UAT
       |    |
       |    +-- Application
       |
       +-- PROD
            |
            +-- Application

Depending on security and isolation requirements, environments may use separate namespaces, clusters, or cloud accounts.

---

# Kubernetes Deployment From GitHub Actions

A common flow is:

    GitHub
       |
       ↓
    GitHub Actions
       |
       ↓
    Build Image
       |
       ↓
    Push Image
       |
       ↓
    Configure Kubernetes Access
       |
       ↓
    kubectl
       |
       ↓
    Kubernetes API Server
       |
       ↓
    Deployment

---

# Kubernetes Authentication From GitHub Actions

The pipeline must authenticate to the Kubernetes cluster.

For AWS EKS, a common flow is:

    GitHub Actions
        |
        ↓
    AWS Authentication
        |
        ↓
    EKS Access
        |
        ↓
    Kubernetes API Server
        |
        ↓
    Deployment

AWS authentication can use GitHub Actions OIDC and an appropriate IAM role.

---

# EKS Authentication Concept

Typical flow:

    GitHub Actions
        |
        ↓
    GitHub OIDC
        |
        ↓
    AWS IAM Role
        |
        ↓
    AWS EKS
        |
        ↓
    Kubernetes API

The IAM role should have only the permissions required by the deployment workflow.

---

# kubectl

`kubectl` is the command-line client used to communicate with Kubernetes.

Common commands:

    kubectl get pods

    kubectl get deployments

    kubectl get services

    kubectl get nodes

    kubectl describe pod <pod-name>

    kubectl logs <pod-name>

    kubectl apply -f deployment.yaml

    kubectl delete -f deployment.yaml

---

# kubectl Apply

A common deployment command is:

    kubectl apply -f deployment.yaml

This submits the desired Kubernetes configuration to the API Server.

Flow:

    deployment.yaml
         |
         ↓
    kubectl apply
         |
         ↓
    Kubernetes API Server
         |
         ↓
    Desired State

---

# kubectl Apply in CI/CD

A CD pipeline can perform:

    Checkout
       |
       ↓
    Kubernetes Manifests
       |
       ↓
    kubectl apply
       |
       ↓
    Kubernetes Cluster

This allows deployments to be automated.

---

# Kubernetes Manifest Repository

A repository may contain:

    k8s/
      |
      ├── deployment.yaml
      ├── service.yaml
      ├── configmap.yaml
      └── ingress.yaml

The CD pipeline can apply these manifests.

---

# Kubernetes Deployment Workflow

Example:

    Developer
        |
        ↓
    Git Push
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
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    Kubernetes Manifest
        |
        ↓
    kubectl apply
        |
        ↓
    EKS
        |
        ↓
    Pods
        |
        ↓
    Service
        |
        ↓
    Users

---

# Container Image Flow

A containerized deployment commonly follows:

    Source Code
        |
        ↓
    Docker Build
        |
        ↓
    Security Scan
        |
        ↓
    ECR
        |
        ↓
    Kubernetes Deployment
        |
        ↓
    Pod
        |
        ↓
    Container

---

# Docker Image Tag

Example:

    myapp:1.4.7

The Kubernetes Deployment references this image.

Example concept:

    containers:
      - name: myapp
        image: myapp:1.4.7

Immutable version tags improve traceability.

---

# Avoid latest

Avoid using:

    image: myapp:latest

for controlled production deployments when a specific immutable version can be used.

Prefer:

    image: myapp:1.4.7

or:

    image: myapp:<commit-sha>

This makes it easier to identify exactly what is running.

---

# Updating the Image

A CD pipeline may update the image version.

Example:

    Current:

    myapp:1.4.6

    New:

    myapp:1.4.7

The deployment is then updated to use the new image.

---

# kubectl Set Image

Conceptually:

    kubectl set image deployment/myapp \
      myapp=myapp:1.4.7

This updates the image used by the Deployment.

The exact image registry path should be used in a real deployment.

---

# Kubernetes Rollout

After a Deployment changes, Kubernetes performs a rollout according to the Deployment strategy.

Flow:

    v1.4.6
       |
       ↓
    Deployment Update
       |
       ↓
    v1.4.7
       |
       ↓
    New Pods
       |
       ↓
    Old Pods Removed

---

# Checking Rollout Status

A common command is:

    kubectl rollout status deployment/myapp

This allows the pipeline to wait for the rollout result.

Possible outcome:

    Deployment Successful

or:

    Deployment Failed / Timed Out

---

# Rollout History

Kubernetes can maintain rollout history for Deployments.

Command:

    kubectl rollout history deployment/myapp

This helps inspect previous revisions.

---

# Kubernetes Rollback

If a deployment fails, a previous revision can be restored.

Command:

    kubectl rollout undo deployment/myapp

Flow:

    v1.4.6
       |
       ↓
    v1.4.7
       |
       ↓
      FAIL
       |
       ↓
    Rollback
       |
       ↓
    v1.4.6

Rollback strategy should be tested and designed as part of the CD process.

---

# Kubernetes Deployment Strategy

A Deployment commonly uses:

    RollingUpdate

Another available strategy for a Deployment is:

    Recreate

More advanced release patterns such as blue-green and canary deployments can be implemented using additional Kubernetes resources and deployment tooling.

---

# Rolling Update

During a rolling update:

    Old Pods
      |
      ↓
    New Pods Created
      |
      ↓
    New Pods Become Ready
      |
      ↓
    Old Pods Terminated

This allows updates without stopping the entire application.

---

# Deployment Configuration

Important Deployment fields include:

    replicas
    selector
    template
    containers
    image
    resources
    readinessProbe
    livenessProbe
    strategy

These fields control how the application is deployed and operated.

---

# Replica Count

Example:

    replicas: 3

Desired state:

    Pod 1
    Pod 2
    Pod 3

If one Pod fails:

    Pod 1
    Pod 2
    Pod 3 → Failed

Kubernetes attempts to restore:

    Pod 1
    Pod 2
    Pod 4

---

# Resource Requests

Resource requests tell Kubernetes how much CPU and memory a container needs for scheduling.

Example concept:

    resources:
      requests:
        cpu: "250m"
        memory: "256Mi"

The Scheduler uses requests when selecting nodes.

---

# Resource Limits

Resource limits define the maximum CPU and memory available to the container.

Example concept:

    resources:
      limits:
        cpu: "500m"
        memory: "512Mi"

Requests and limits should be selected based on application behavior and observed resource usage.

---

# Readiness Probe

Readiness determines whether a Pod is ready to receive traffic.

Example flow:

    Pod Started
       |
       ↓
    Readiness Check
       |
       +-- FAIL → No Traffic
       |
       +-- PASS → Receive Traffic

This is especially important during deployments.

---

# Liveness Probe

Liveness determines whether a container is still functioning according to the defined health check.

Example:

    Container
       |
       ↓
    Liveness Check
       |
       +-- PASS → Continue
       |
       +-- FAIL → Kubernetes May Restart Container

Readiness and liveness serve different purposes.

---

# Startup Probe

A startup probe can be used for applications that need significant time to initialize.

Flow:

    Container Starts
        |
        ↓
    Startup Probe
        |
        +-- PASS → Other Health Checks
        |
        +-- FAIL → Container Restart

This can prevent liveness checks from restarting a slow-starting application too early.

---

# Kubernetes Health Checks in CD

A good deployment pipeline should validate:

    Deployment Created
        |
        ↓
    Pods Running
        |
        ↓
    Pods Ready
        |
        ↓
    Service Available
        |
        ↓
    Application Health
        |
        ↓
    Smoke Tests

---

# Deployment Validation

After applying Kubernetes resources:

    kubectl apply
        |
        ↓
    kubectl rollout status
        |
        ↓
    kubectl get pods
        |
        ↓
    kubectl get services
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test

---

# Kubernetes Events

When a deployment fails, inspect events.

Command:

    kubectl get events

or:

    kubectl describe pod <pod-name>

Events can reveal problems such as:

    ImagePullBackOff
    FailedScheduling
    CrashLoopBackOff
    FailedMount
    Insufficient Resources

---

# Kubernetes Logs

Application logs can be checked with:

    kubectl logs <pod-name>

For a previous crashed container:

    kubectl logs <pod-name> --previous

Logs are important when investigating deployment failures.

---

# Deployment Troubleshooting

If deployment fails:

    Deployment
        |
        ↓
      FAIL
        |
        ↓
    Check Rollout
        |
        ↓
    Check Pods
        |
        ↓
    Check Events
        |
        ↓
    Check Logs
        |
        ↓
    Check Configuration
        |
        ↓
    Fix / Rollback

---

# ImagePullBackOff

Possible causes:

- Incorrect image name
- Incorrect image tag
- Registry authentication failure
- Image does not exist
- Network problems

Troubleshooting:

    kubectl describe pod <pod-name>

Check the Events section.

---

# CrashLoopBackOff

Possible causes:

- Application startup failure
- Incorrect configuration
- Missing environment variable
- Missing secret
- Application bug
- Resource issue

Troubleshooting:

    kubectl logs <pod-name>

and:

    kubectl logs <pod-name> --previous

---

# FailedScheduling

Possible causes:

- Insufficient CPU
- Insufficient memory
- Node constraints
- Taints and tolerations
- Affinity rules
- Node availability

Check:

    kubectl describe pod <pod-name>

---

# Configuration Failure

A deployment can fail because of incorrect configuration.

Check:

    ConfigMap
    Secret
    Environment Variables
    Volume Mounts
    Application Arguments

The CD pipeline should validate required configuration where practical.

---

# Kubernetes Service Validation

After deployment:

    kubectl get svc

Check:

    Service Exists
    Correct Type
    Correct Port
    Correct Target Port
    Endpoints Available

A running Pod does not automatically mean the application is reachable.

---

# Kubernetes Endpoints

A Service needs healthy endpoints.

Conceptually:

    Service
       |
       ↓
    Endpoints
       |
       +-- Pod 1
       +-- Pod 2
       +-- Pod 3

If there are no endpoints, traffic may not reach the application.

---

# Label Selectors

Services use selectors to identify Pods.

Example:

    Service Selector:
        app: myapp

Pod labels:

    app: myapp

If labels do not match:

    Service
       |
       X
    No Matching Pods

This is a common Kubernetes deployment issue.

---

# Kubernetes Ingress Validation

If using Ingress:

    Internet
       |
       ↓
    Load Balancer
       |
       ↓
    Ingress
       |
       ↓
    Service
       |
       ↓
    Pods

Validate:

    Ingress Exists
    Correct Host
    Correct Path
    Correct Service
    Correct Port
    TLS Configuration

---

# Kubernetes Deployment in EKS

A common AWS architecture is:

    GitHub Actions
        |
        ↓
    AWS OIDC
        |
        ↓
    IAM Role
        |
        ↓
    EKS
        |
        ↓
    Kubernetes API
        |
        ↓
    Deployment
        |
        ↓
    Pods
        |
        ↓
    Service / ALB
        |
        ↓
    Users

---

# EKS Deployment Process

Typical flow:

    1. Checkout source
    2. Build application
    3. Run tests
    4. Build Docker image
    5. Scan image
    6. Push image to ECR
    7. Authenticate to AWS
    8. Configure kubectl for EKS
    9. Deploy Kubernetes manifests
    10. Wait for rollout
    11. Run health checks
    12. Run smoke tests
    13. Report deployment result

---

# EKS kubeconfig

A deployment workflow needs Kubernetes cluster configuration.

A common AWS CLI command is:

    aws eks update-kubeconfig \
      --region <region> \
      --name <cluster-name>

This configures kubectl to communicate with the EKS cluster.

The exact authentication and authorization mechanism depends on the EKS setup.

---

# Kubernetes Deployment With GitHub Actions

Conceptual pipeline:

    Checkout
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Push ECR
        |
        ↓
    AWS Authentication
        |
        ↓
    Configure EKS Access
        |
        ↓
    kubectl apply
        |
        ↓
    Rollout Status
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test

---

# GitHub Actions Kubernetes Deployment Example

Conceptual workflow structure:

    name: Deploy to Kubernetes

    jobs:

      deploy:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Configure AWS Credentials
            uses: aws-actions/configure-aws-credentials@v6

          - name: Update kubeconfig
            run: |
              aws eks update-kubeconfig \
                --region <region> \
                --name <cluster>

          - name: Deploy
            run: |
              kubectl apply -f k8s/

          - name: Wait for rollout
            run: |
              kubectl rollout status deployment/myapp

---

# Important Security Point

The example above shows the deployment structure.

In production:

    GitHub Actions
        |
        ↓
    OIDC
        |
        ↓
    IAM Role
        |
        ↓
    EKS

Use least privilege and restrict which repositories, branches, environments, and workflows can assume deployment roles.

---

# Kubernetes Deployment With Helm

Instead of raw manifests, Helm can manage Kubernetes releases.

Flow:

    GitHub Actions
        |
        ↓
    Helm
        |
        ↓
    Kubernetes
        |
        ↓
    Deployment
        |
        ↓
    Pods

Example concept:

    helm upgrade --install myapp ./helm/myapp

Environment-specific values can be provided separately.

---

# Kubernetes Deployment With ArgoCD

With GitOps:

    GitHub Actions
        |
        ↓
    Build Image
        |
        ↓
    Push ECR
        |
        ↓
    Update GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Pods

In this model, GitHub Actions does not necessarily need direct Kubernetes deployment permissions.

---

# Direct Deployment vs GitOps

Direct deployment:

    GitHub Actions
        |
        ↓
    kubectl
        |
        ↓
    Kubernetes

GitOps deployment:

    GitHub Actions
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

GitOps can provide stronger separation between CI and cluster deployment.

---

# Kubernetes Deployment Strategy Selection

Choose the strategy based on:

    Application Type
    Availability Requirements
    Risk
    Traffic Pattern
    Rollback Requirements
    Infrastructure
    Release Frequency

Common strategies include:

    Rolling
    Blue-Green
    Canary
    Recreate

These strategies are covered in more detail in later CD notes.

---

# Rolling Deployment

Basic flow:

    Old Version
        |
        ↓
    New Version Pods
        |
        ↓
    Readiness Checks
        |
        ↓
    Old Pods Removed

This is Kubernetes' common Deployment strategy.

---

# Recreate Deployment

Recreate strategy stops old Pods before starting new Pods.

Flow:

    Old Pods
        |
        ↓
    Stop Old Pods
        |
        ↓
    Start New Pods

This can create downtime and should be used only when appropriate.

---

# Kubernetes Deployment Availability

Availability depends on:

    Replica Count
    Pod Scheduling
    Readiness Probes
    Resource Availability
    Deployment Strategy
    Node Availability
    Application Health

Example:

    replicas: 3

During a rollout, Kubernetes can maintain availability based on configured rollout parameters and application readiness.

---

# maxUnavailable

`maxUnavailable` controls how many Pods can be unavailable during a rolling update.

Conceptually:

    replicas: 4

    maxUnavailable: 1

This limits the number of unavailable Pods during the update.

---

# maxSurge

`maxSurge` controls how many additional Pods can temporarily be created during a rolling update.

Conceptually:

    replicas: 4

    maxSurge: 1

Kubernetes can temporarily run an additional Pod during the rollout.

---

# Rolling Update Example

Conceptually:

    strategy:
      type: RollingUpdate

      rollingUpdate:
        maxUnavailable: 1
        maxSurge: 1

The values should be chosen based on application availability requirements.

---

# Kubernetes Deployment and Readiness

Readiness is critical during rolling deployments.

Flow:

    New Pod
       |
       ↓
    Start
       |
       ↓
    Readiness Probe
       |
       +-- FAIL → Do Not Send Traffic
       |
       +-- PASS → Send Traffic
       |
       ↓
    Continue Rollout

Without appropriate readiness checks, traffic may reach an application before it is ready.

---

# Kubernetes Deployment and Liveness

Liveness protects against certain runtime failures.

Flow:

    Running Container
        |
        ↓
    Liveness Check
        |
        +-- PASS → Continue
        |
        +-- FAIL → Restart Container

Liveness should not simply duplicate readiness logic.

---

# Deployment Timeout

A CD pipeline should not wait forever for a failed deployment.

Example:

    Deploy
      |
      ↓
    Rollout
      |
      ↓
    Timeout
      |
      ↓
    Deployment Failed
      |
      ↓
    Investigate / Rollback

Pipeline timeout settings should reflect realistic application startup and rollout times.

---

# Kubernetes Deployment Validation

After deployment, validate:

    Deployment Status
    Replica Availability
    Pod Status
    Pod Readiness
    Service Status
    Ingress Status
    Application Health
    Logs
    Events

---

# Basic Validation Commands

Useful commands include:

    kubectl get deployment

    kubectl get pods

    kubectl get svc

    kubectl get ingress

    kubectl rollout status deployment/myapp

    kubectl describe deployment myapp

    kubectl describe pod <pod-name>

    kubectl logs <pod-name>

    kubectl get events

---

# Deployment Success Criteria

A deployment should be considered successful only when the required checks pass.

Example:

    Deployment Applied
          |
          ↓
    Rollout Successful
          |
          ↓
    Pods Ready
          |
          ↓
    Service Healthy
          |
          ↓
    Application Health Check
          |
          ↓
    Smoke Test
          |
          ↓
    SUCCESS

---

# Deployment Failure Criteria

Example:

    Deployment Applied
          |
          ↓
    Rollout
          |
          ↓
        FAIL
          |
          ↓
    Deployment Failed

Possible actions:

    Stop Pipeline
    Collect Logs
    Collect Events
    Investigate
    Rollback
    Notify Team

---

# Kubernetes Deployment Notifications

A production pipeline can notify teams when deployment status changes.

Example:

    Deployment
       |
       +-- SUCCESS → Notification
       |
       +-- FAILURE → Alert

Notification channels depend on the organization's tooling.

---

# Kubernetes Deployment Auditability

A good CD pipeline should record:

    Application
    Image Version
    Commit SHA
    Environment
    Cluster
    Namespace
    Deployment Time
    Deployment Result
    Approver
    Rollback Information

This makes troubleshooting and auditing easier.

---

# Immutable Deployment

Prefer:

    myapp:1.4.7

over:

    myapp:latest

Then record:

    Commit SHA
    Image Digest
    Version
    Deployment Environment

This creates a strong relationship between source code and production deployment.

---

# Image Digest

Container images can be identified using a digest.

Example concept:

    myapp@sha256:<digest>

A digest identifies a specific image content.

Using immutable references improves supply-chain traceability.

---

# Kubernetes Deployment Security

Important security practices:

- Use least-privilege IAM
- Use OIDC where appropriate
- Avoid long-lived cloud credentials
- Restrict cluster access
- Protect production environments
- Scan container images
- Scan dependencies
- Protect Kubernetes Secrets
- Use RBAC
- Use network policies where appropriate
- Keep Kubernetes versions supported
- Restrict deployment permissions
- Audit deployment activity

---

# Kubernetes RBAC

RBAC means:

    Role-Based Access Control

It controls which users, groups, and service accounts can perform actions in Kubernetes.

Conceptually:

    User / Service Account
            |
            ↓
           RBAC
            |
            ↓
    Kubernetes Resources

---

# Deployment Service Account

A CI/CD process can use a dedicated identity with only the permissions required for deployment.

Example:

    GitHub Actions
        |
        ↓
    Deployment Identity
        |
        ↓
    Required Kubernetes Permissions
        |
        ↓
    Application Namespace

Avoid granting unrestricted cluster-admin access unless there is a justified requirement.

---

# Namespace-Level Permissions

For example:

    CI Deployment Identity
        |
        ↓
    QA Namespace
        |
        +-- Deployments
        +-- Services
        +-- ConfigMaps
        +-- Secrets

This can reduce the blast radius compared with unrestricted cluster-wide access.

---

# Production Deployment Permissions

Production should have stronger controls.

Example:

    GitHub Actions
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    Production Deployment Role
        |
        ↓
    Production Cluster

---

# Kubernetes Deployment Best Practices

- Use declarative manifests
- Use immutable image versions
- Avoid `latest`
- Define resource requests
- Define resource limits
- Configure readiness probes
- Configure liveness probes where appropriate
- Use startup probes for slow-starting applications when needed
- Use multiple replicas for highly available workloads
- Use rolling updates when appropriate
- Validate deployments
- Monitor rollout status
- Use health checks
- Keep secrets protected
- Use RBAC
- Use least privilege
- Use GitOps where appropriate
- Keep deployment configuration version controlled
- Test rollback procedures
- Keep production access restricted

---

# Kubernetes Deployment Anti-Patterns

## Anti-Pattern 1: Using latest

    image: myapp:latest

Problem:

    Difficult to identify exact version.

Better:

    image: myapp:1.4.7

---

# Anti-Pattern 2: No Readiness Probe

Without readiness checks:

    New Pod
       |
       ↓
    Traffic Immediately

The application may not be ready to handle requests.

Better:

    New Pod
       |
       ↓
    Readiness Check
       |
       ↓
    Ready
       |
       ↓
    Traffic

---

# Anti-Pattern 3: No Resource Requests

Without resource requests:

    Pod
       |
       ↓
    Scheduler
       |
       ↓
    Unpredictable Placement

Define appropriate requests based on workload requirements.

---

# Anti-Pattern 4: Overly Broad Permissions

Bad:

    CI/CD
       |
       ↓
    cluster-admin

Better:

    CI/CD
       |
       ↓
    Deployment Role
       |
       ↓
    Required Resources Only

---

# Anti-Pattern 5: Direct Production Access From Every Workflow

Bad:

    Every Branch
        |
        ↓
    Production Cluster

Better:

    Protected Branch
        |
        ↓
    Required Checks
        |
        ↓
    Protected Environment
        |
        ↓
    Approval
        |
        ↓
    Production

---

# Anti-Pattern 6: No Rollback Plan

Bad:

    Deployment
       |
       ↓
      FAIL
       |
       ↓
    Manual Investigation Only

Better:

    Deployment
       |
       ↓
      FAIL
       |
       ↓
    Rollback / Recovery
       |
       ↓
    Validate

---

# Anti-Pattern 7: Manual Cluster Changes

Bad:

    Engineer
       |
       ↓
    kubectl edit production resource

Better:

    Git
       |
       ↓
    CI/CD or GitOps
       |
       ↓
    Kubernetes

Controlled deployments improve consistency and auditability.

---

# Kubernetes CD Pipeline Example

Complete flow:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    CI
        |
        +-- Build
        +-- Unit Test
        +-- SonarQube
        +-- Trivy
        |
        ↓
    Merge
        |
        ↓
    Docker Build
        |
        ↓
    ECR
        |
        ↓
    CD
        |
        ↓
    Kubernetes
        |
        ↓
    Deployment
        |
        ↓
    Rolling Update
        |
        ↓
    Readiness Checks
        |
        ↓
    Service
        |
        ↓
    Ingress / ALB
        |
        ↓
    Application
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Success

---

# Kubernetes CD With GitHub Actions and EKS

Example architecture:

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
    Amazon ECR
        |
        ↓
    GitHub Actions
        |
        ↓
    AWS OIDC
        |
        ↓
    IAM Role
        |
        ↓
    Amazon EKS
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
    Users

---

# Kubernetes CD With GitOps

Alternative architecture:

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
        +-- Scan
        +-- Push ECR
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
    Kubernetes Deployment
        |
        ↓
    Application

This approach is covered in detail in the ArgoCD notes.

---

# Interview Questions

## Basic

1. What is Kubernetes?
2. Why is Kubernetes used in CD pipelines?
3. What is a Pod?
4. What is a Deployment?
5. What is a Service?
6. What is an Ingress?
7. What is a Namespace?
8. What is a ConfigMap?
9. What is a Secret?
10. What is a Kubernetes cluster?
11. What is kubectl?
12. What is a Kubernetes manifest?

---

# Intermediate Interview Questions

13. How do you deploy an application to Kubernetes using GitHub Actions?

14. How do you authenticate GitHub Actions to EKS?

15. How do you update a Docker image in Kubernetes?

16. How do you check deployment status?

17. How do you troubleshoot a failed Kubernetes deployment?

18. What is a rolling update?

19. What is `maxUnavailable`?

20. What is `maxSurge`?

21. What is a readiness probe?

22. What is a liveness probe?

23. What is a startup probe?

24. How do you perform a Kubernetes rollback?

25. How do you expose an application outside the cluster?

26. How do you manage configuration across environments?

27. How do you manage secrets in Kubernetes?

28. How do you secure Kubernetes deployment permissions?

---

# Advanced Interview Questions

29. Design a production Kubernetes CD pipeline using GitHub Actions.

30. How would you deploy a Docker image from ECR to EKS?

31. How would you implement zero-downtime Kubernetes deployment?

32. How would you troubleshoot a rollout stuck in progress?

33. How would you design Kubernetes deployment rollback?

34. How would you secure GitHub Actions access to EKS?

35. How would you implement environment-specific Kubernetes deployments?

36. How would you integrate Helm into Kubernetes CD?

37. How would you integrate ArgoCD into Kubernetes CD?

38. How would you prevent unauthorized production deployments?

39. How would you design Kubernetes deployment validation?

40. How would you handle a deployment where Pods are Running but users receive errors?

---

# Scenario Question

## A deployment succeeded but Pods are not Ready. What would you check?

I would:

    1. Check Pod status
    2. Describe the Pod
    3. Check Events
    4. Check container logs
    5. Check readiness probe
    6. Check ConfigMaps
    7. Check Secrets
    8. Check Service selectors
    9. Check application health endpoint
    10. Check resource availability

Commands:

    kubectl get pods

    kubectl describe pod <pod-name>

    kubectl logs <pod-name>

    kubectl get events

---

# Scenario Question

## Deployment is stuck during rollout. What would you do?

I would check:

    kubectl rollout status deployment/myapp

Then:

    kubectl get pods

    kubectl describe deployment myapp

    kubectl describe pod <pod-name>

    kubectl logs <pod-name>

    kubectl get events

I would identify whether the issue is related to:

    Image
    Scheduling
    Resources
    Probes
    Configuration
    Secrets
    Application Startup

If the new version is unhealthy and the situation requires immediate recovery, I would roll back to the previous known-good revision.

---

# Scenario Question

## Pods are Running but users cannot access the application. What would you check?

I would check the complete traffic path:

    User
      |
      ↓
    ALB / Ingress
      |
      ↓
    Service
      |
      ↓
    Endpoints
      |
      ↓
    Pods

I would verify:

    Ingress
    Service
    Service Selector
    Endpoints
    Target Port
    Pod Readiness
    Application Port
    Load Balancer Configuration

---

# Scenario Question

## How would you achieve zero-downtime deployment?

I would use:

    Multiple Replicas
        +
    RollingUpdate
        +
    Readiness Probes
        +
    Proper maxUnavailable
        +
    Proper maxSurge
        +
    Health Validation

Flow:

    Old Pods
        |
        ↓
    New Pod
        |
        ↓
    Readiness Pass
        |
        ↓
    Traffic
        |
        ↓
    Old Pod Removed
        |
        ↓
    Repeat

This minimizes service interruption during deployment.

---

# Scenario Question

## How would you deploy the same Docker image across DEV, QA, UAT, and PROD?

I would build the image once:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    ECR
        |
        ↓
    myapp:1.4.7

Then promote the same image:

    myapp:1.4.7
        |
        +-- DEV
        +-- QA
        +-- UAT
        +-- PROD

Environment-specific configuration would be injected separately.

---

# Scenario Question

## How would you secure production Kubernetes deployment?

I would use:

    Protected GitHub Environment
        +
    Required Approvals
        +
    OIDC
        +
    Least-Privilege IAM
        +
    Kubernetes RBAC
        +
    Protected Branch
        +
    Immutable Image
        +
    Security Scanning
        +
    Audit Trail

This reduces the risk of unauthorized production deployments.

---

# Scenario Question

## How would you implement Kubernetes deployment using GitOps?

I would separate CI from deployment.

CI:

    Build
      |
      ↓
    Test
      |
      ↓
    Scan
      |
      ↓
    Push Image
      |
      ↓
    Update GitOps Repository

CD:

    GitOps Repository
        |
        ↓
      ArgoCD
        |
        ↓
      Kubernetes
        |
        ↓
      EKS

This provides declarative and auditable deployment management.

---

# Scenario Question

## What happens if a new Kubernetes deployment fails?

The pipeline should:

    Detect Failure
        |
        ↓
    Stop Promotion
        |
        ↓
    Collect Logs
        |
        ↓
    Collect Events
        |
        ↓
    Identify Root Cause
        |
        ↓
    Rollback if Required
        |
        ↓
    Validate
        |
        ↓
    Report Result

---

# Scenario Question

## Why should the CD pipeline wait for rollout status?

Applying a manifest successfully does not necessarily mean the application is healthy.

For example:

    kubectl apply
        |
        ↓
      SUCCESS
        |
        ↓
    Pod Starts
        |
        ↓
      FAILS

Therefore:

    kubectl apply
        |
        ↓
    rollout status
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test

provides stronger deployment validation.

---

# Scenario Question

## What is the difference between readiness and liveness?

Readiness answers:

    "Should this Pod receive traffic?"

Liveness answers:

    "Is this container still functioning according to the health check?"

Example:

    Readiness FAIL
        |
        ↓
    Remove Pod from Service Traffic

    Liveness FAIL
        |
        ↓
    Kubernetes May Restart Container

---

# Scenario Question

## Why are immutable image tags important?

Suppose:

    Production → myapp:1.4.7

If the tag is immutable, we know exactly which application version was deployed.

If the same tag is overwritten, the meaning of:

    myapp:1.4.7

can change.

Therefore immutable versions improve:

    Traceability
    Rollback
    Auditing
    Reproducibility
    Security

---

# Scenario Question

## How would you troubleshoot ImagePullBackOff?

I would:

    1. Check Pod status
    2. Describe the Pod
    3. Check Events
    4. Verify image name
    5. Verify image tag
    6. Verify registry availability
    7. Verify authentication
    8. Verify ECR permissions if using AWS
    9. Verify network access

Useful command:

    kubectl describe pod <pod-name>

The Events section usually provides important clues.

---

# Scenario Question

## How would you troubleshoot CrashLoopBackOff?

I would:

    1. Check current logs
    2. Check previous container logs
    3. Describe the Pod
    4. Check Events
    5. Verify environment variables
    6. Verify ConfigMaps
    7. Verify Secrets
    8. Check resource limits
    9. Check application startup
    10. Check probes

Commands:

    kubectl logs <pod-name>

    kubectl logs <pod-name> --previous

    kubectl describe pod <pod-name>

---

# Scenario Question

## How would you troubleshoot a Service that is not forwarding traffic?

I would check:

    Service
      |
      ↓
    Selector
      |
      ↓
    Endpoints
      |
      ↓
    Pods

Commands:

    kubectl get svc

    kubectl describe svc <service-name>

    kubectl get endpoints

    kubectl get pods --show-labels

I would verify that the Service selector matches the Pod labels.

---

# Scenario Question

## How would you deploy to multiple Kubernetes environments?

I would separate environment-specific configuration.

Example:

    Kubernetes
       |
       +-- DEV
       |    |
       |    +-- values-dev
       |
       +-- QA
       |    |
       |    +-- values-qa
       |
       +-- UAT
       |    |
       |    +-- values-uat
       |
       +-- PROD
            |
            +-- values-prod

The same application artifact can be promoted while configuration differs per environment.

---

# Final Kubernetes CD Mental Model

Remember:

    Source Code
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Test
        +-- Quality
        +-- Security
        |
        ↓
    Immutable Container Image
        |
        ↓
    ECR
        |
        ↓
    Kubernetes
        |
        +-- Deployment
        +-- Service
        +-- ConfigMap
        +-- Secret
        +-- Ingress
        |
        ↓
    Pods
        |
        ↓
    Readiness
        |
        ↓
    Service Traffic
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Successful Deployment

---

# Kubernetes CD Key Points

Remember these points for interviews:

    Kubernetes = Container Orchestration

    Pod = Smallest Deployable Unit

    Deployment = Manages Application Pods

    Service = Stable Network Access

    Ingress = HTTP/HTTPS Routing

    ConfigMap = Non-Sensitive Configuration

    Secret = Sensitive Configuration

    Namespace = Logical Resource Isolation

    kubectl = Kubernetes CLI

    Deployment = Desired State

    RollingUpdate = Gradual Application Replacement

    Readiness = Can Receive Traffic

    Liveness = Is Container Healthy

    Startup Probe = Application Startup Protection

    ECR = Container Image Registry in AWS

    EKS = Managed Kubernetes Service in AWS

    ArgoCD = GitOps Continuous Delivery

---

# Final Kubernetes CD Checklist

    [ ] Application builds successfully
    [ ] Unit tests pass
    [ ] Quality gate passes
    [ ] Security scan passes
    [ ] Docker image built
    [ ] Docker image tagged immutably
    [ ] Image pushed to ECR
    [ ] AWS authentication configured securely
    [ ] Kubernetes access configured
    [ ] Deployment manifest validated
    [ ] Deployment applied
    [ ] Rollout monitored
    [ ] Pods become Ready
    [ ] Service is available
    [ ] Ingress / ALB is available
    [ ] Health checks pass
    [ ] Smoke tests pass
    [ ] Deployment result recorded
    [ ] Rollback strategy available
    [ ] Production permissions restricted
    [ ] Deployment audit trail maintained

---

# Final Concept

Kubernetes CD is the process of automatically delivering a validated application artifact into a Kubernetes environment in a controlled, repeatable, and observable way.

The complete model is:

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
    Quality
      |
      ↓
    Security
      |
      ↓
    Container Image
      |
      ↓
    ECR
      |
      ↓
    Kubernetes
      |
      ↓
    Deployment
      |
      ↓
    Rolling Update
      |
      ↓
    Readiness
      |
      ↓
    Service
      |
      ↓
    Ingress / ALB
      |
      ↓
    Health Check
      |
      ↓
    Smoke Test
      |
      ↓
    Production Application

The key objective is to make Kubernetes deployments repeatable, secure, traceable, highly available, and capable of safe recovery when a deployment fails.