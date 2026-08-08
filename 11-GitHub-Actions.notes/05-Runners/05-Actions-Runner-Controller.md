# Actions Runner Controller (ARC)

Actions Runner Controller (ARC) is a Kubernetes-based solution for managing self-hosted GitHub Actions runners.

It allows organizations to run GitHub Actions runners inside Kubernetes and manage their lifecycle through Kubernetes resources.

Conceptually:

```text
GitHub Actions
      |
      ↓
Actions Runner Controller
      |
      ↓
Kubernetes
      |
      ├── Runner Pod
      ├── Runner Pod
      ├── Runner Pod
      └── Runner Pod
```

ARC is particularly useful when an organization needs:

- Dynamic runner provisioning
- Kubernetes-based runner infrastructure
- Autoscaling
- Ephemeral runners
- Large runner fleets
- Better runner lifecycle management

---

# Why ARC?

Traditional self-hosted runners can be provisioned as permanent VMs.

Example:

```text
VM Runner 01
VM Runner 02
VM Runner 03
```

But this can create operational challenges:

```text
Capacity Planning
Patching
Scaling
Cleanup
Runner Lifecycle
Idle Infrastructure
```

ARC allows runner infrastructure to be managed through Kubernetes.

---

# Traditional Self-Hosted Runner

```text
GitHub
   |
   ↓
VM
   |
   └── GitHub Actions Runner
```

If workload increases:

```text
More Jobs
   |
   ↓
Runner Capacity Exhausted
   |
   ↓
Provision More VMs
```

This can take time and requires infrastructure automation.

---

# ARC Architecture

With ARC:

```text
GitHub
   |
   ↓
ARC Controller
   |
   ↓
Kubernetes
   |
   ├── Runner Pod
   ├── Runner Pod
   ├── Runner Pod
   └── Runner Pod
```

The controller manages runner resources according to the configured architecture.

---

# Kubernetes-Native Runner Management

ARC brings GitHub Actions runner management into Kubernetes.

Conceptually:

```text
Kubernetes Cluster
        |
        ├── ARC Controller
        |
        ├── RunnerSet
        |
        └── Runner Pods
```

Kubernetes becomes the platform managing the runner workload.

---

# ARC Components

A modern ARC architecture can include components such as:

```text
ARC Controller
     |
     ↓
Runner Scale Set
     |
     ↓
Runner Pods
```

The controller manages the desired runner infrastructure.

---

# Runner Scale Sets

Runner Scale Sets are used to manage a group of ephemeral GitHub Actions runners.

Conceptually:

```text
Runner Scale Set
       |
       ├── Runner 1
       ├── Runner 2
       ├── Runner 3
       └── Runner 4
```

The number of runners can change according to workload and configuration.

---

# Ephemeral Runner Model

A key ARC pattern is ephemeral runners.

Conceptually:

```text
Job Arrives
    |
    ↓
Runner Created
    |
    ↓
Job Executes
    |
    ↓
Runner Removed
```

This helps reduce persistent state between jobs.

---

# Why Ephemeral Runners?

Persistent runners can retain:

```text
Workspace
Docker images
Temporary files
Credentials
Build artifacts
Processes
Caches
```

Ephemeral runners provide a cleaner lifecycle:

```text
Fresh Runner
     |
     ↓
Execute Job
     |
     ↓
Dispose
```

This can improve isolation.

---

# ARC and Kubernetes

ARC runs inside Kubernetes.

Example:

```text
EKS
 |
 ├── ARC Controller
 |
 ├── Runner Scale Set
 |
 ├── Runner Pod
 |
 ├── Runner Pod
 └── Runner Pod
```

This means Kubernetes infrastructure becomes part of the runner-management architecture.

---

# ARC on AWS EKS

A production architecture can look like:

```text
GitHub
   |
   ↓
GitHub Actions
   |
   ↓
ARC
   |
   ↓
AWS EKS
   |
   ├── Runner Pod
   ├── Runner Pod
   └── Runner Pod
```

This can provide dynamic runner capacity inside AWS infrastructure.

---

# Private Network Architecture

One important use case is private infrastructure.

```text
GitHub
   |
   ↓
ARC
   |
   ↓
Private EKS
   |
   ↓
Runner Pods
   |
   ├── Private APIs
   ├── Internal Services
   └── Deployment Targets
```

The exact connectivity model should be designed according to the organization's network and security requirements.

---

# ARC and Runner Labels

ARC-managed runners can be targeted through runner labels and scale-set configuration.

Conceptually:

```text
Runner Scale Set
       |
       ├── linux
       ├── kubernetes
       └── production
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

Use the label and routing model supported by the configured ARC version.

---

# ARC and Runner Groups

ARC can be integrated into an organization-wide runner governance model.

Conceptually:

```text
Organization
     |
     ↓
Production Runner Group
     |
     ↓
ARC
     |
     ↓
Runner Scale Set
     |
     ↓
Runner Pods
```

This combines:

```text
Access Control
+
Dynamic Runner Management
```

---

# ARC and Environment Separation

A large organization can separate runner infrastructure:

```text
Development
   |
   └── Dev Runner Scale Set

QA
   |
   └── QA Runner Scale Set

UAT
   |
   └── UAT Runner Scale Set

Production
   |
   └── Production Runner Scale Set
```

Each environment can have different:

```text
Network
IAM
Kubernetes permissions
Runner labels
Security policies
Capacity
```

---

# ARC and GitHub Actions Workflow

Example:

```yaml
jobs:

  build:

    runs-on:
      - self-hosted
      - linux

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package
```

The job is routed to an eligible self-hosted runner managed through the ARC architecture.

---

# ARC Scaling Concept

Suppose:

```text
10 jobs
```

arrive.

ARC can manage the runner pool so that additional runner capacity can be created according to configuration.

Conceptually:

```text
Low Workload
     |
     ↓
2 Runner Pods

High Workload
     |
     ↓
10 Runner Pods
```

After workload decreases:

```text
10 Runner Pods
     |
     ↓
Reduced Runner Capacity
```

The exact scaling behavior depends on the ARC configuration and Kubernetes capacity.

---

# Traditional vs ARC Scaling

### Traditional VM Runners

```text
Job Load
   |
   ↓
Need More Capacity
   |
   ↓
Provision VM
   |
   ↓
Install / Register Runner
```

### ARC

```text
Job Load
   |
   ↓
Runner Scale Set
   |
   ↓
Kubernetes
   |
   ↓
Runner Pods
```

ARC can make dynamic runner lifecycle more Kubernetes-native.

---

# Runner Scale Set Architecture

```text
                   GitHub
                      |
                      ↓
             Runner Scale Set
                      |
             ┌────────┼────────┐
             ↓        ↓        ↓
          Runner    Runner    Runner
            Pod       Pod       Pod
             |        |        |
             └────────┼────────┘
                      ↓
                  Kubernetes
```

---

# ARC Controller

The controller is responsible for managing runner-related Kubernetes resources and coordinating with GitHub according to the configured ARC architecture.

Conceptually:

```text
GitHub
   |
   ↓
ARC Controller
   |
   ↓
Runner Resources
   |
   ↓
Kubernetes
```

---

# Kubernetes Reconciliation

ARC follows a Kubernetes-style control model.

Conceptually:

```text
Desired State
      |
      ↓
ARC Controller
      |
      ↓
Actual Runner State
      |
      ↓
Reconcile
```

If the desired runner capacity changes, ARC can reconcile the runner infrastructure accordingly.

---

# Desired vs Actual State

Example:

```text
Desired:
5 runners

Actual:
3 runners
```

The controller works toward the configured desired state.

Conceptually:

```text
Desired State
     |
     ↓
Controller
     |
     ↓
Create / Manage Runners
     |
     ↓
5 Runners
```

---

# ARC and Kubernetes Pods

Runner workloads can execute as Kubernetes Pods.

Example:

```text
Pod
 |
 ├── GitHub Actions Runner
 ├── Build Tools
 ├── Docker tooling
 ├── kubectl
 └── Helm
```

The actual runner image should be built and maintained according to the organization's requirements.

---

# Runner Image

A custom runner image can contain required tooling.

Example:

```text
Runner Image
 |
 ├── Git
 ├── Docker tooling
 ├── kubectl
 ├── Helm
 ├── Terraform
 ├── AWS CLI
 ├── Java
 ├── Node.js
 └── Security Tools
```

Do not install unnecessary tools.

---

# Custom Runner Images

For production, runner images should be:

```text
Versioned
Tested
Scanned
Documented
Reproducible
```

Example lifecycle:

```text
Dockerfile
    |
    ↓
Build Image
    |
    ↓
Security Scan
    |
    ↓
Test
    |
    ↓
Publish
    |
    ↓
Runner Scale Set
```

---

# Runner Image Security

A compromised runner image can affect every job using it.

Therefore:

```text
Base Image
   |
   ↓
Security Scan
   |
   ↓
Patch
   |
   ↓
Validate
   |
   ↓
Production
```

Use a trusted image pipeline.

---

# ARC and Docker Builds

Container builds are common workloads.

Conceptually:

```text
Runner Pod
    |
    ↓
Docker / Build Tool
    |
    ↓
Container Image
    |
    ↓
ECR
```

The container-building architecture should be chosen carefully because running privileged container workloads inside Kubernetes has security implications.

---

# Docker-in-Docker Considerations

Running Docker directly inside Kubernetes runner pods may require privileged configurations depending on the chosen build architecture.

This can increase risk.

Alternatives may include:

```text
BuildKit
Kaniko
Other rootless/container build approaches
External build services
```

Choose the build mechanism based on security and operational requirements.

---

# ARC and Kubernetes Security

ARC itself runs in Kubernetes, so protect:

```text
Kubernetes API
ARC Controller
Runner Pods
Runner Images
Secrets
Service Accounts
Network Policies
```

Do not assume that Kubernetes automatically makes runners secure.

---

# Service Accounts

Runner workloads should use appropriately scoped Kubernetes identities.

Avoid unnecessarily giving:

```text
cluster-admin
```

to runner pods.

Use the minimum Kubernetes permissions required.

---

# Kubernetes RBAC

A conceptual model:

```text
Runner Pod
    |
    ↓
Service Account
    |
    ↓
RBAC
    |
    ↓
Allowed Kubernetes Resources
```

Separate:

```text
CI permissions
```

from:

```text
Production deployment permissions
```

where possible.

---

# Network Policies

Network policies can restrict communication.

Example:

```text
Runner Pod
    |
    ├── GitHub
    ├── ECR
    └── Required Internal Services
```

Avoid unnecessary access to:

```text
Databases
Internal APIs
Other Namespaces
Management Systems
```

---

# Namespace Isolation

Separate runner workloads into appropriate namespaces when required.

Example:

```text
Kubernetes
 |
 ├── arc-system
 |
 ├── ci-runners
 |
 ├── qa-runners
 |
 └── production-runners
```

The exact namespace strategy depends on the organization's security and operational requirements.

---

# Production Runner Scale Set

A production scale set could conceptually be:

```text
production-runner-scale-set
          |
          ├── Runner Pod
          ├── Runner Pod
          └── Runner Pod
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

The exact `runs-on` value should match the runner labels/routing configuration used by the ARC deployment.

---

# ARC + Production Environment

A production workflow can combine:

```text
Runner Scale Set
+
Runner Group
+
Labels
+
GitHub Environment
+
Required Approval
```

Architecture:

```text
Protected Workflow
       |
       ↓
Production Environment
       |
       ↓
Production Runner Group
       |
       ↓
ARC Runner Scale Set
       |
       ↓
Runner Pod
       |
       ↓
Production
```

---

# ARC + GitOps

ARC is useful for the CI side of a GitOps architecture.

Example:

```text
GitHub
   |
   ↓
ARC Runner
   |
   ├── Build
   ├── Test
   ├── SonarQube
   ├── Trivy
   └── Push Image
          |
          ↓
      Git Manifest
          |
          ↓
        ArgoCD
          |
          ↓
         EKS
```

This can keep direct Kubernetes deployment permissions out of the CI runner.

---

# Recommended GitOps Security Model

Instead of:

```text
Runner
  |
  └── cluster-admin
```

prefer:

```text
Runner
  |
  └── Git + ECR
          |
          ↓
        ArgoCD
          |
          ↓
          EKS
```

This can reduce the runner's production blast radius.

---

# ARC + Terraform

ARC runners can execute Terraform workflows.

Example:

```text
Runner Pod
    |
    ↓
Terraform
    |
    ↓
AWS
```

But cloud permissions should still use least privilege.

Prefer short-lived authentication such as OIDC where supported.

---

# Terraform Workflow

```yaml
jobs:

  terraform:

    runs-on:
      - self-hosted
      - linux
      - terraform

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: '1.9'

      - name: Terraform Init
        run: terraform init

      - name: Terraform Validate
        run: terraform validate

      - name: Terraform Plan
        run: terraform plan
```

---

# ARC + DevSecOps

ARC runners can execute security stages.

Example:

```text
ARC Runner
    |
    ├── SonarQube
    ├── Trivy
    ├── Veracode
    ├── Unit Tests
    └── Build
```

Architecture:

```text
Code
 |
 ↓
ARC Runner
 |
 ├── Build
 ├── SAST
 ├── SCA
 ├── Container Scan
 └── Tests
 |
 ↓
Artifact
```

---

# ARC and Microservices

A matrix or reusable workflow can process multiple services.

Example:

```text
ARC Runner Scale Set
        |
        ↓
Microservice Jobs
        |
   ┌────┼────┬────┐
   ↓    ↓    ↓    ↓
User Cart Order Payment
```

Each service can run independently when appropriate.

---

# ARC + Matrix

Matrix jobs can increase runner demand.

Example:

```yaml
strategy:

  matrix:
    service:
      - user
      - catalogue
      - cart
      - payment
```

Conceptually:

```text
Matrix
 |
 ├── User
 ├── Catalogue
 ├── Cart
 └── Payment
       |
       ↓
ARC Runner Scale Set
```

This is one area where dynamic runner provisioning can be useful.

---

# ARC and Parallel Jobs

Suppose:

```text
20 matrix combinations
```

arrive.

The runner infrastructure can scale according to the configured architecture and available Kubernetes resources.

Conceptually:

```text
20 Jobs
   |
   ↓
Runner Scale Set
   |
   ↓
Multiple Runner Pods
```

Be careful about:

```text
Kubernetes capacity
AWS cost
ECR limits
External API limits
Database load
```

---

# ARC and max-parallel

Matrix workflows can still use:

```yaml
strategy:
  max-parallel: 5
```

This can prevent an unlimited burst of concurrent jobs.

Architecture:

```text
100 Matrix Jobs
      |
      ↓
max-parallel: 5
      |
      ↓
Controlled Runner Demand
```

This can help protect infrastructure.

---

# ARC and Concurrency

ARC scaling does not replace GitHub Actions concurrency.

For production deployments:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

This can prevent overlapping production workflows.

So:

```text
Concurrency
→ Controls workflow execution

ARC
→ Manages runner infrastructure
```

---

# ARC and Autoscaling

The purpose of autoscaling is to match runner capacity to workload.

Conceptually:

```text
Low Jobs
   |
   ↓
Few Runners

High Jobs
   |
   ↓
More Runners
```

This can reduce idle runner capacity.

---

# Autoscaling Considerations

Autoscaling should consider:

```text
Job queue
Runner availability
Kubernetes capacity
CPU
Memory
Node capacity
Cloud limits
Pod startup time
Cost
```

Do not scale runners without ensuring the Kubernetes cluster can actually host them.

---

# Kubernetes Cluster Autoscaling

There are potentially two scaling layers:

```text
GitHub Jobs
     |
     ↓
ARC Runner Scaling
     |
     ↓
Kubernetes Pods
     |
     ↓
Kubernetes Node Scaling
```

If there are not enough Kubernetes nodes:

```text
Runner Pods
      |
      ↓
Pending
      |
      ↓
Node Capacity Required
```

Cluster autoscaling may be required.

---

# ARC Scaling Architecture

```text
GitHub Job Queue
       |
       ↓
ARC
       |
       ↓
Runner Pods
       |
       ↓
Kubernetes Scheduler
       |
       ↓
Nodes
       |
       ↓
AWS Infrastructure
```

This is an important production consideration.

---

# Cold Start

Ephemeral runners can introduce startup time.

Architecture:

```text
Job Arrives
    |
    ↓
Runner Pod Creation
    |
    ↓
Image Pull
    |
    ↓
Pod Startup
    |
    ↓
Runner Registration
    |
    ↓
Job Execution
```

Large runner images can increase startup time.

---

# Optimizing Startup Time

Consider:

```text
Smaller runner images
Efficient image registry
Pre-pulled images where appropriate
Adequate Kubernetes capacity
Efficient initialization
```

Do not sacrifice security simply to make startup faster.

---

# Runner Image Size

Large image:

```text
5 GB
```

can take longer to pull.

Smaller image:

```text
1 GB
```

may start faster.

Use only required tools.

---

# Caching Considerations

Ephemeral runners reduce persistent state.

This means build caches need deliberate design.

Possible mechanisms:

```text
GitHub Actions cache
Docker registry cache
BuildKit cache
Artifact repositories
```

Use caching where it provides real performance benefits.

---

# ARC and ECR

A common AWS architecture:

```text
ARC Runner
    |
    ↓
Docker Build
    |
    ↓
ECR
```

Authentication should use appropriate short-lived credentials.

Example concept:

```text
GitHub OIDC
    |
    ↓
AWS IAM
    |
    ↓
ECR
```

---

# ARC and AWS OIDC

A runner can obtain cloud permissions through a GitHub OIDC trust relationship.

Conceptually:

```text
GitHub Workflow
      |
      ↓
OIDC Token
      |
      ↓
AWS IAM Trust
      |
      ↓
Temporary Role
      |
      ↓
ECR / AWS
```

This avoids storing permanent AWS access keys on runner pods.

---

# ARC and Secrets

Do not bake secrets into runner images.

Bad:

```text
Runner Image
   |
   └── AWS_SECRET_ACCESS_KEY
```

Better:

```text
Workflow
   |
   ↓
OIDC / Secret Management
   |
   ↓
Temporary Access
```

Use GitHub secrets or approved external secret-management solutions where appropriate.

---

# ARC Runner Compromise

If a runner pod is compromised, the blast radius depends on:

```text
Kubernetes permissions
AWS permissions
Network access
Repository access
Secrets
Runner group access
```

Therefore:

```text
Least Privilege
+
Network Isolation
+
Ephemeral Lifecycle
```

are important.

---

# ARC Security Layers

A production ARC architecture should consider:

```text
GitHub Repository Security
        |
        ↓
Workflow Permissions
        |
        ↓
Runner Groups
        |
        ↓
Runner Labels
        |
        ↓
Kubernetes RBAC
        |
        ↓
Network Policies
        |
        ↓
AWS IAM
        |
        ↓
AWS Resources
```

---

# ARC and Untrusted Pull Requests

Avoid giving untrusted pull requests access to privileged ARC runner scale sets.

Bad:

```text
External PR
    |
    ↓
Production ARC Runner
    |
    ↓
AWS Production
```

Better:

```text
External PR
    |
    ↓
Isolated CI Runner
    |
    ↓
Tests
    |
    ↓
Review
    |
    ↓
Protected Production Workflow
```

---

# ARC Namespace Security

Keep ARC components and runner workloads appropriately isolated.

Example:

```text
arc-system
    |
    └── ARC Controller

ci-runners
    |
    └── CI Runner Pods

production-runners
    |
    └── Production Runner Pods
```

Use Kubernetes security policies and RBAC according to the required architecture.

---

# Pod Security

Runner pods should follow Kubernetes security best practices.

Consider:

```text
Non-root execution where supported
Read-only filesystems where practical
Dropped Linux capabilities
Resource limits
Network policies
Restricted service accounts
Trusted images
Security scanning
```

The exact configuration depends on the runner image and workload requirements.

---

# Resource Requests and Limits

Runner pods consume Kubernetes resources.

Example concept:

```yaml
resources:
  requests:
    cpu: "1"
    memory: "2Gi"

  limits:
    cpu: "2"
    memory: "4Gi"
```

Use values appropriate for the workload.

Incorrect resource settings can cause:

```text
Pending Pods
OOMKilled
CPU throttling
Node pressure
```

---

# Runner Pod Scheduling

Kubernetes can use:

```text
Node selectors
Taints
Tolerations
Affinity
Topology rules
```

to control where runner pods execute.

Example concept:

```text
Production Runner
       |
       ↓
Production Node Pool
       |
       ↓
Runner Pod
```

This can provide additional infrastructure isolation.

---

# Dedicated Node Pool

For sensitive workloads:

```text
EKS
 |
 ├── General Nodes
 |
 └── Runner Nodes
       |
       ├── Runner Pod
       ├── Runner Pod
       └── Runner Pod
```

This can isolate runner workloads from unrelated applications.

---

# Taints and Tolerations

A dedicated runner node pool can use Kubernetes taints.

Conceptually:

```text
Runner Node Pool
      |
      └── taint: workload=runner
```

Runner pods use the corresponding toleration.

This prevents unrelated workloads from scheduling onto runner nodes.

---

# ARC and Node Autoscaling

If runner pods scale from:

```text
2 → 20
```

the EKS cluster may need additional nodes.

Architecture:

```text
GitHub Jobs
    |
    ↓
ARC
    |
    ↓
Runner Pods
    |
    ↓
Pending Pods
    |
    ↓
Cluster Autoscaler / Node Autoscaling
    |
    ↓
More Nodes
```

This should be tested before production use.

---

# Capacity Planning

Do not only plan for runner count.

Plan for:

```text
Runner CPU
Runner Memory
Node CPU
Node Memory
Pod density
Image pull bandwidth
ECR throughput
Network
Job duration
Peak workload
```

---

# ARC Cost Considerations

ARC may reduce idle runner infrastructure, but it does not make compute free.

Costs may come from:

```text
EKS
EC2 Nodes
Fargate where applicable
ECR
Network
Storage
Runner workloads
```

Autoscaling should balance:

```text
Performance
Availability
Security
Cost
```

---

# ARC Failure Scenarios

Possible failures:

```text
ARC controller unavailable
Runner pods cannot start
Kubernetes capacity exhausted
Node failure
Image pull failure
Network failure
GitHub connectivity issue
Runner registration failure
AWS permission failure
```

Production design should account for these.

---

# Troubleshooting ARC

Start with:

```text
1. GitHub workflow
2. Runner availability
3. Runner Scale Set
4. ARC controller
5. Kubernetes Pods
6. Kubernetes Events
7. Node capacity
8. Network
9. Runner image
10. Authentication
```

---

# Runner Pod Pending

If a runner pod is:

```text
Pending
```

check:

```bash
kubectl get pods -n <namespace>
```

Then:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Look for:

```text
Insufficient CPU
Insufficient memory
Node selector mismatch
Taint/toleration issue
Image pull problem
Scheduling constraints
```

---

# Runner Pod CrashLoopBackOff

Check:

```bash
kubectl logs <pod-name> -n <namespace>
```

and:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

Investigate:

```text
Runner image
Environment
Secrets
Service account
Network
Startup configuration
```

---

# ImagePullBackOff

Check:

```text
1. Image name
2. Image tag
3. Registry access
4. ECR authentication
5. ImagePullSecrets if required
6. Network connectivity
```

Example:

```bash
kubectl describe pod <pod-name> -n <namespace>
```

---

# ARC Controller Troubleshooting

Check ARC controller pods:

```bash
kubectl get pods -n arc-system
```

Check logs:

```bash
kubectl logs <controller-pod> -n arc-system
```

Then inspect:

```text
Controller health
Kubernetes permissions
GitHub connectivity
Configuration
Runner scale set resources
```

---

# Kubernetes Events

Events can reveal scheduling or startup problems.

Example:

```bash
kubectl get events \
  -n <namespace> \
  --sort-by=.lastTimestamp
```

Look for:

```text
FailedScheduling
FailedMount
FailedPull
BackOff
Permission errors
```

---

# ARC Monitoring

Monitor:

```text
ARC controller health
Runner count
Runner availability
Pending jobs
Pending pods
Pod startup time
Job duration
Node utilization
CPU
Memory
Disk
Network
```

---

# Production Observability

A production ARC platform should integrate with the organization's observability stack.

For example:

```text
Prometheus
   |
   ↓
Metrics
   |
   ↓
Grafana
```

and:

```text
Logs
   |
   ↓
ELK Stack
```

This can help identify runner failures and capacity issues.

---

# ARC and Prometheus

Possible metrics to monitor include concepts such as:

```text
Runner availability
Runner count
Job queue
Pod status
Controller health
Scaling behavior
```

The exact metrics exposed depend on the ARC version and deployment configuration.

---

# ARC and Grafana

A dashboard can show:

```text
Runner Count
Active Jobs
Queued Jobs
Runner Startup Time
Pod CPU
Pod Memory
Node Utilization
Failures
```

Architecture:

```text
ARC / Kubernetes
       |
       ↓
Prometheus
       |
       ↓
Grafana
```

---

# ARC Logs and ELK

Centralized logs can help investigate:

```text
Runner failures
Controller failures
Pod startup errors
Authentication errors
Deployment failures
```

Architecture:

```text
Runner / ARC / Kubernetes Logs
             |
             ↓
             ELK
             |
             ↓
          Analysis
```

---

# Production Deployment Architecture

A production-grade architecture can look like:

```text
                    GitHub
                       |
                       ↓
              GitHub Actions
                       |
                       ↓
             Production Workflow
                       |
             ┌─────────┴─────────┐
             ↓                   ↓
      Environment Rules      Runner Group
                                   |
                                   ↓
                              ARC Scale Set
                                   |
                                   ↓
                              EKS Runner Pods
                                   |
                     ┌─────────────┼─────────────┐
                     ↓             ↓             ↓
                   Build        Security       Deploy
                     |
                     ↓
                   ECR
                     |
                     ↓
                  ArgoCD
                     |
                     ↓
                    EKS
```

If GitOps is used, the runner may not need direct cluster deployment privileges.

---

# Recommended Enterprise Architecture

```text
                    GitHub
                       |
                       ↓
              Protected Repository
                       |
                       ↓
                 GitHub Actions
                       |
          ┌────────────┴────────────┐
          ↓                         ↓
     CI Workflow              Production Workflow
          |                         |
          ↓                         ↓
      ARC CI Group           Protected Environment
          |                         |
          ↓                         ↓
   Runner Scale Set        ARC Production Group
          |                         |
          ↓                         ↓
     Runner Pods             Runner Pods
          |                         |
          ↓                         ↓
      Build/Test             GitOps / Deploy
          |                         |
          ↓                         ↓
         ECR                     ArgoCD
                                    |
                                    ↓
                                   EKS
```

---

# ARC vs Traditional Self-Hosted Runners

| Area | Traditional Self-Hosted | ARC |
|---|---|---|
| Infrastructure | VM / Server | Kubernetes |
| Lifecycle | Often manually managed | Kubernetes-managed |
| Scaling | Manual / external automation | Dynamic architecture |
| Ephemeral runners | Possible | Strong fit |
| Kubernetes integration | External | Native platform model |
| Tool management | VM/image | Runner image |
| Capacity | Pre-provisioned | Can scale with workload |
| Operations | VM management | Kubernetes + ARC management |
| Best for | Stable runner fleets | Dynamic Kubernetes-based fleets |

---

# ARC vs GitHub-Hosted Runners

| Area | GitHub-Hosted | ARC |
|---|---|---|
| Infrastructure | GitHub-managed | Organization-managed |
| Private network | Additional design required | Can run inside private Kubernetes |
| Customization | Limited | High |
| Scaling | Managed by GitHub | Organization-controlled |
| Kubernetes integration | External | Native |
| Maintenance | Low | Higher |
| Security responsibility | Lower operational burden | Higher |
| Runner lifecycle | Managed | Kubernetes/ARC managed |
| Specialized tooling | Limited | Custom images possible |

---

# When to Use ARC

ARC is worth considering when:

```text
✓ Large self-hosted runner fleet
✓ Kubernetes is already a platform standard
✓ Dynamic runner capacity is needed
✓ Ephemeral runners are preferred
✓ Private network access is required
✓ Runner workloads need Kubernetes scheduling
✓ Teams need centralized runner management
```

---

# When ARC May Be Overkill

If you only need:

```text
1–2 stable self-hosted runners
```

and they can easily run as VMs, ARC may add unnecessary complexity.

You now have to manage:

```text
Kubernetes
ARC
Runner Images
Runner Pods
Networking
RBAC
Autoscaling
Monitoring
```

Choose the simplest architecture that meets the requirements.

---

# ARC Production Checklist

```text
☐ Dedicated runner groups
☐ Controlled repository access
☐ Protected production workflows
☐ Environment protection
☐ Least-privilege IAM
☐ Kubernetes RBAC
☐ Network policies
☐ Trusted runner images
☐ Image scanning
☐ Resource requests and limits
☐ Node capacity planning
☐ Runner autoscaling strategy
☐ Cluster autoscaling strategy
☐ Monitoring
☐ Logging
☐ Ephemeral runner lifecycle
☐ Secret management
☐ Disaster recovery plan
☐ Cost monitoring
☐ Regular security reviews
```

---

# Common Mistakes

- Deploying ARC without understanding Kubernetes operations.
- Giving runner pods excessive permissions.
- Using `cluster-admin` unnecessarily.
- Putting production and untrusted workloads in the same runner pool.
- Using privileged container builds without understanding the security implications.
- Creating huge runner images.
- Ignoring image pull/startup time.
- Forgetting Kubernetes node capacity.
- Scaling runners without scaling cluster capacity.
- Storing long-lived credentials in runner images.
- Not monitoring runner pods.
- Not monitoring ARC controller health.
- Allowing untrusted PRs to use privileged runner scale sets.
- Using ARC when a simple VM runner would be sufficient.
- Treating runner labels as a security boundary.

---

# Scenario: Runner Demand Suddenly Increases

Problem:

```text
Normal:
5 jobs

Peak:
100 jobs
```

Architecture:

```text
100 Jobs
   |
   ↓
ARC
   |
   ↓
Runner Scale Set
   |
   ↓
Many Runner Pods
```

Then Kubernetes may need additional nodes:

```text
Runner Pods
    |
    ↓
Node Capacity
    |
    ↓
Node Autoscaling
```

Monitor:

```text
Queue
Pods
Nodes
CPU
Memory
Startup Time
```

---

# Scenario: Runner Pods Are Pending

Check:

```bash
kubectl get pods -n <namespace>
```

Then:

```bash
kubectl describe pod <pod> -n <namespace>
```

If you see:

```text
Insufficient CPU
```

or:

```text
Insufficient memory
```

investigate:

```text
Node capacity
Resource requests
Node autoscaling
Runner pod sizing
```

---

# Scenario: ARC Runner Cannot Access ECR

Check:

```text
OIDC configuration
IAM trust policy
IAM permissions
ECR repository
Network
DNS
Runner identity
```

Architecture:

```text
Runner
  |
  ↓
OIDC
  |
  ↓
AWS IAM
  |
  ↓
ECR
```

Avoid solving this by putting permanent AWS access keys into the runner image.

---

# Scenario: Production Runner Compromised

First question:

```text
What can this runner access?
```

Review:

```text
AWS IAM
Kubernetes RBAC
Network
Secrets
Repository permissions
Runner Group
Environment
```

Then:

```text
1. Isolate runner
2. Revoke credentials
3. Destroy ephemeral runner if applicable
4. Investigate logs
5. Rotate affected secrets
6. Review IAM
7. Review Kubernetes access
8. Rebuild trusted runner image
```

The exact incident-response process should follow the organization's security procedures.

---

# Scenario: ARC Controller Failure

If the controller becomes unavailable:

```text
Existing Runner Pods
        |
        ↓
May continue according to their current lifecycle

New Runner Management
        |
        ↓
May be affected
```

Investigate:

```text
Controller pod
Kubernetes events
Controller logs
RBAC
Network
GitHub connectivity
Resource capacity
```

Design high availability according to the ARC and Kubernetes deployment architecture.

---

# Scenario: GitHub Job Waiting

Workflow:

```text
Job
 |
 ↓
Waiting for runner
```

Check:

```text
1. Runner Scale Set
2. Runner availability
3. Labels/routing
4. Runner group access
5. ARC controller
6. Kubernetes pods
7. Node capacity
8. Network
```

---

# Scenario: Slow Runner Startup

Possible causes:

```text
Large runner image
Slow image pull
Insufficient nodes
Node autoscaling delay
Slow initialization
Network problems
```

Optimize:

```text
Smaller images
Efficient registry access
Pre-pulling where appropriate
Adequate baseline capacity
Fast initialization
```

---

# Interview Questions

## Basic

1. What is Actions Runner Controller?
2. Why would you use ARC?
3. How is ARC different from a traditional self-hosted runner?
4. What is a runner scale set?
5. Why are ephemeral runners useful?
6. Where does ARC run?
7. Can ARC manage runners inside Kubernetes?

## Intermediate

8. Explain the architecture of ARC.
9. How does ARC help with dynamic runner capacity?
10. How does ARC work with Kubernetes?
11. What is the difference between a persistent runner and an ephemeral runner?
12. How would you deploy ARC on EKS?
13. How do runner groups and labels fit into an ARC architecture?
14. How would you troubleshoot a runner pod stuck in `Pending`?
15. How would you troubleshoot an ARC controller failure?
16. How would you manage custom tooling on ARC runners?
17. How do you monitor ARC runners?

## Advanced / Production

18. Design a production ARC architecture for GitHub Actions running inside AWS EKS.
19. Explain how ARC runner scaling interacts with Kubernetes node autoscaling.
20. A workload suddenly increases from 10 jobs to 500 jobs. How would you design ARC to handle the load?
21. How would you secure ARC runner pods?
22. Why should you avoid giving ARC runners `cluster-admin`?
23. How would you use Kubernetes RBAC to restrict runner permissions?
24. How would you use network policies to isolate production runner pods?
25. How would you secure AWS access from ARC runners using GitHub OIDC?
26. How would you design runner images for ARC?
27. What are the security concerns with Docker builds inside Kubernetes runner pods?
28. How would you prevent untrusted pull requests from using privileged production ARC runners?
29. Design an ARC architecture with separate CI, QA, UAT, and Production runner scale sets.
30. How would you combine ARC, runner groups, labels, GitHub environments, IAM, and Kubernetes RBAC?
31. How would you troubleshoot a runner pod in `ImagePullBackOff`?
32. How would you troubleshoot a runner pod in `CrashLoopBackOff`?
33. How would you optimize slow ARC runner startup?
34. How would you monitor ARC using Prometheus and Grafana?
35. How would you centralize ARC and runner logs using ELK?
36. How can GitOps with ArgoCD reduce the privileges required by ARC runners?
37. Compare GitHub-hosted runners, VM-based self-hosted runners, and ARC-managed runners.
38. When would ARC be overkill?
39. Design a disaster-recovery strategy for an ARC-based production runner platform.
40. A production ARC runner is compromised. Walk through your incident-response approach.