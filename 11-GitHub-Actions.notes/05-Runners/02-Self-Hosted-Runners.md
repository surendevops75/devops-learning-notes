# Self-Hosted Runners

A **self-hosted runner** is a machine that you manage and connect to GitHub Actions to execute workflow jobs.

Unlike GitHub-hosted runners:

```text
GitHub-Hosted Runner
    |
    └── GitHub manages the infrastructure

Self-Hosted Runner
    |
    └── Organization manages the infrastructure
```

Self-hosted runners provide greater control over:

- Infrastructure
- Network connectivity
- Installed software
- Hardware
- Security controls
- Internal systems
- Custom tooling

---

# Why Self-Hosted Runners?

GitHub-hosted runners are convenient for standard CI workloads.

However, organizations may need a runner that can access private infrastructure.

Example:

```text
GitHub
   |
   ↓
Self-Hosted Runner
   |
   ↓
Private Network
   |
   ├── EKS
   ├── Internal APIs
   ├── Databases
   └── Internal Tools
```

This is one of the major reasons organizations use self-hosted runners.

---

# Basic Architecture

```text
Developer
    |
    ↓
GitHub Repository
    |
    ↓
GitHub Actions Workflow
    |
    ↓
Self-Hosted Runner
    |
    ├── Checkout
    ├── Build
    ├── Test
    ├── Scan
    └── Deploy
```

The runner executes the job assigned by GitHub Actions.

---

# Self-Hosted Runner Components

A self-hosted runner generally consists of:

```text
Runner Machine
    |
    ├── Operating System
    ├── GitHub Actions Runner Software
    ├── Required Tools
    ├── Network Connectivity
    ├── Security Controls
    └── Monitoring
```

Typical DevOps tools may include:

```text
Git
Docker
kubectl
Helm
Terraform
AWS CLI
Azure CLI
Java
Node.js
Python
Maven
Security Tools
```

Only install tools that are required.

---

# Supported Operating Systems

Self-hosted runners can be configured on supported operating systems such as:

```text
Linux
Windows
macOS
```

For DevOps workloads, Linux is commonly preferred.

Example:

```yaml
runs-on:
  - self-hosted
  - linux
```

---

# Self-Hosted Runner Labels

A self-hosted runner can have labels.

Example:

```text
self-hosted
linux
x64
production
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

The job is routed to an eligible runner matching the requested labels.

Detailed label management is covered in:

```text
03-Runner-Labels.md
```

---

# Runner Groups

Organizations can place runners into groups.

Conceptually:

```text
Organization
     |
     ├── Development Runners
     |
     ├── QA Runners
     |
     ├── UAT Runners
     |
     └── Production Runners
```

Runner groups can help control which repositories or workflows are allowed to use specific runners.

Detailed coverage:

```text
04-Runner-Groups.md
```

---

# Basic Workflow

Example:

```yaml
jobs:

  build:

    runs-on:
      - self-hosted
      - linux

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package
```

Here:

```text
self-hosted
```

indicates that the job requires a self-hosted runner.

---

# Self-Hosted Runner Lifecycle

A conceptual lifecycle:

```text
Provision Runner
       |
       ↓
Install Runner Software
       |
       ↓
Register Runner
       |
       ↓
Apply Labels / Group
       |
       ↓
Runner Online
       |
       ↓
Execute Jobs
       |
       ↓
Monitor / Maintain
       |
       ↓
Update / Replace
```

Production environments should treat runner lifecycle as infrastructure management.

---

# Provisioning a Runner

A runner can be provisioned on infrastructure such as:

```text
Virtual Machine
Bare Metal
Cloud Instance
Private Server
Container-based Infrastructure
Kubernetes-managed Runner
```

The correct model depends on workload and organizational requirements.

---

# VM-Based Runner

Example architecture:

```text
AWS
 |
 └── EC2
      |
      └── Self-Hosted Runner
            |
            ├── Docker
            ├── kubectl
            ├── Helm
            └── Terraform
```

The runner can then access resources allowed by the EC2 network and IAM configuration.

---

# Private Network Access

One major advantage of self-hosted runners is private network connectivity.

Example:

```text
GitHub
   |
   ↓
Self-Hosted Runner
   |
   ↓
VPC
   |
   ├── Private EKS
   ├── Private RDS
   └── Internal Services
```

This can be useful when deployment targets are not publicly accessible.

---

# Important Security Consideration

A self-hosted runner that can access production infrastructure is highly privileged infrastructure.

If malicious code executes on the runner, it may potentially access:

```text
Cloud credentials
Private network
Kubernetes
Internal services
Secrets
Deployment systems
```

Therefore:

```text
Production Runner
        |
        ↓
High Security Requirement
```

---

# Never Treat Production Runners as Generic Build Machines

Avoid:

```text
All Repositories
      |
      ↓
Production Runner
```

Prefer:

```text
Approved Production Workflow
          |
          ↓
Production Runner Group
          |
          ↓
Production Runner
```

Access should be tightly controlled.

---

# Pull Request Security

Be especially careful with workflows triggered by pull requests.

Example:

```text
Untrusted PR
    |
    ↓
Workflow
    |
    ↓
Production Runner
```

This is dangerous if the workflow can execute arbitrary repository-controlled commands.

A safer model is:

```text
Untrusted PR
    |
    ↓
GitHub-Hosted / Isolated Runner
    |
    ↓
Build + Test
```

Production deployment should happen through protected workflows after appropriate validation and approval.

---

# Fork Security

Fork-based pull requests deserve additional caution.

Conceptually:

```text
External Fork
      |
      ↓
Pull Request
      |
      ↓
Workflow
      |
      ↓
Runner
```

Do not automatically expose production secrets or privileged self-hosted runners to untrusted fork code.

---

# Runner Isolation

A production runner should ideally have limited access to only what it needs.

Example:

```text
Production Runner
       |
       ├── Production EKS
       |
       └── Required Internal Services
```

Avoid unrestricted access to:

```text
Entire VPC
All Databases
All Accounts
All Kubernetes Clusters
All Internal Systems
```

Apply least privilege.

---

# IAM Permissions

For AWS-based runners, the runner's AWS identity should have only required permissions.

Bad:

```text
AdministratorAccess
```

for every deployment runner.

Better:

```text
Specific Deployment Permissions
```

For example, a deployment workflow may require access to:

```text
ECR
EKS
Specific AWS resources
```

The exact permissions should be determined by the deployment process.

---

# OIDC vs Static Credentials

Prefer short-lived authentication mechanisms where possible.

Conceptual flow:

```text
GitHub Actions
      |
      ↓
OIDC
      |
      ↓
AWS IAM
      |
      ↓
Temporary Credentials
      |
      ↓
AWS Resources
```

Avoid storing long-lived AWS access keys permanently on the runner.

---

# Runner Service Account

The runner process itself should not run with unnecessary operating-system privileges.

Apply:

```text
Least Privilege
```

where practical.

Avoid giving the runner administrative access unless required.

---

# Persistent Runner

A persistent runner remains available between jobs.

Example:

```text
Runner
 |
 ├── Job 1
 |
 ├── Job 2
 |
 ├── Job 3
 |
 └── Job 4
```

Potential leftover state:

```text
Workspace
Docker Images
Build Cache
Temporary Files
Logs
Credentials
```

This creates security and reliability considerations.

---

# Persistent Runner Risks

A persistent runner can accumulate:

```text
Old files
Old Docker images
Credentials
Temporary artifacts
Modified configuration
Unexpected processes
Build leftovers
```

One job may potentially influence another if isolation and cleanup are inadequate.

---

# Ephemeral Runner

An ephemeral runner is created for a workload and then removed.

Conceptually:

```text
Create Runner
      |
      ↓
Run Job
      |
      ↓
Destroy Runner
```

This reduces persistent state.

---

# Persistent vs Ephemeral

| Persistent Runner | Ephemeral Runner |
|---|---|
| Long-lived | Short-lived |
| Reused across jobs | Usually one workload |
| Easier to maintain initially | Requires provisioning automation |
| Can retain state | Cleaner environment |
| Greater contamination risk | Better isolation potential |
| Easier for stable internal tooling | Good for dynamic workloads |

For sensitive production workloads, ephemeral designs can provide stronger isolation when implemented correctly.

---

# Runner Cleanup

If using persistent runners, implement cleanup procedures.

Potential cleanup areas:

```text
Workspace
Docker Images
Docker Containers
Temporary Files
Build Artifacts
Logs
Caches
```

Do not blindly delete files that may belong to another workload.

Cleanup must be designed around the runner's concurrency model.

---

# Docker Cleanup

Docker-heavy runners can accumulate disk usage.

Check:

```bash
docker system df
```

Possible cleanup commands must be used carefully.

For example:

```bash
docker image prune
```

or other controlled cleanup mechanisms.

Never run destructive cleanup blindly on a shared production runner.

---

# Runner Disk Monitoring

Monitor disk usage.

Example:

```bash
df -h
```

If disk usage reaches a critical level:

```text
Build
  |
  ↓
Docker Build
  |
  ↓
Disk Full
  |
  ↓
Job Failure
```

Automated monitoring can prevent this situation.

---

# Runner Resource Monitoring

Monitor:

```text
CPU
Memory
Disk
Network
Runner status
Job queue
Job duration
```

Example:

```text
CPU → 90%
Memory → 95%
Disk → 92%
```

These can indicate capacity or cleanup problems.

---

# Runner Capacity

If one runner receives too many jobs:

```text
Job 1 ─┐
Job 2 ─┤
Job 3 ─┤
Job 4 ─┤
Job 5 ─┘
        |
        ↓
     Runner
        |
        ↓
     Queue
```

Possible solutions:

```text
Add runners
Use runner groups
Use labels
Use autoscaling
Use ephemeral runners
Optimize workflows
```

---

# Multiple Self-Hosted Runners

Instead of:

```text
1 Runner
 |
 └── All Jobs
```

use:

```text
Runner Pool
 |
 ├── Runner 1
 ├── Runner 2
 ├── Runner 3
 └── Runner 4
```

Jobs can be distributed across eligible runners.

---

# Production Runner Pool

Example:

```text
Production Runner Group
 |
 ├── prod-runner-01
 ├── prod-runner-02
 └── prod-runner-03
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

This gives the organization more capacity and resilience.

---

# Runner Labels Example

Suppose runners are:

```text
runner-01
  self-hosted
  linux
  qa

runner-02
  self-hosted
  linux
  production

runner-03
  self-hosted
  linux
  production
```

A production workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

can target the production-labeled runner pool.

---

# Custom Tooling

Self-hosted runners are useful when a workflow needs custom software.

Example:

```text
Internal CLI
Internal Security Tool
Private Build Tool
Custom Compiler
Specialized Scanner
```

Install and maintain these tools on the runner image.

---

# Tool Version Management

Do not allow random manual installations.

Example:

```text
Runner Image
 |
 ├── Terraform 1.x
 ├── Helm 3.x
 ├── kubectl supported version
 ├── AWS CLI
 └── Docker
```

Keep versions documented and tested.

---

# Immutable Runner Image

A strong enterprise pattern is to build a standard runner image.

Example:

```text
Base OS
   |
   ↓
Security Updates
   |
   ↓
Required Tools
   |
   ↓
Runner Software
   |
   ↓
Validated Runner Image
```

New runners are created from the approved image.

---

# Golden Image

A standardized image can be treated as a **golden image**.

Example:

```text
Company Runner Golden Image
 |
 ├── Ubuntu
 ├── Git
 ├── Docker
 ├── kubectl
 ├── Helm
 ├── Terraform
 ├── AWS CLI
 └── Security Baseline
```

Benefits:

- Consistency
- Faster provisioning
- Easier patching
- Easier troubleshooting
- Reduced configuration drift

---

# Runner Configuration Management

Configuration should ideally be automated.

Possible tools:

```text
Terraform
Ansible
Cloud-init
Packer
Configuration management
```

Example architecture:

```text
Terraform
   |
   ↓
EC2
   |
   ↓
Runner Installation
   |
   ↓
Labels
   |
   ↓
Ready
```

---

# Infrastructure as Code

A production runner fleet can be provisioned using Terraform.

Conceptually:

```text
Terraform
   |
   ├── Network
   ├── Security Group
   ├── IAM
   ├── EC2
   └── Runner Infrastructure
```

This makes infrastructure repeatable.

---

# Network Security

A self-hosted runner should have controlled network access.

Example:

```text
Runner
 |
 ├── HTTPS → GitHub
 ├── HTTPS → AWS APIs
 ├── HTTPS → ECR
 └── Private Network → EKS
```

Avoid unrestricted outbound and inbound connectivity unless justified.

---

# Inbound Connectivity

A runner generally needs to communicate with GitHub and the services required by its workload.

The exact network architecture depends on the runner model and organization.

For private environments, carefully design:

```text
DNS
Proxy
Firewall
Security Groups
Routing
NAT
Private Endpoints
```

---

# Runner Security Groups

For a cloud-hosted runner VM, security groups should follow least privilege.

Example:

```text
Inbound
 |
 └── Only required management access

Outbound
 |
 ├── GitHub
 ├── AWS APIs
 └── Required private services
```

Avoid opening unnecessary inbound ports.

---

# SSH Access

Do not provide broad SSH access to production runners.

If administrative access is required:

```text
Approved Admin
      |
      ↓
Secure Access Mechanism
      |
      ↓
Runner
```

Use centralized access controls, auditing, and short-lived access where possible.

---

# Runner Updates

The GitHub Actions runner software should be kept current according to the organization's update policy.

Also maintain:

```text
OS
Docker
kubectl
Helm
Terraform
Cloud CLI
Security Tools
```

A stale runner can become a security risk.

---

# Runner Registration

A self-hosted runner must be registered with GitHub.

Conceptually:

```text
Organization / Repository
       |
       ↓
Runner Registration
       |
       ↓
Runner Online
```

Registration should use appropriate authentication and scope.

Avoid distributing registration credentials unnecessarily.

---

# Repository-Level Runner

A runner can be configured for a specific repository.

Conceptually:

```text
Repository
    |
    └── Self-Hosted Runner
```

This provides strong isolation compared with making the runner broadly available.

---

# Organization-Level Runner

An organization can manage runners for multiple repositories.

Conceptually:

```text
Organization
 |
 ├── Repository A
 ├── Repository B
 └── Repository C
        |
        ↓
   Runner Pool
```

Runner groups and access policies become important in this model.

---

# Enterprise Runner Architecture

A larger organization may have:

```text
Organization
 |
 ├── Development Runner Group
 |
 ├── QA Runner Group
 |
 ├── UAT Runner Group
 |
 └── Production Runner Group
```

Each group can have different:

```text
Network
Security
Tools
Permissions
Repositories
Capacity
```

---

# Production Runner Isolation

Production runners should ideally be separated from lower environments.

Example:

```text
QA Runner
   |
   └── QA Infrastructure

UAT Runner
   |
   └── UAT Infrastructure

Production Runner
   |
   └── Production Infrastructure
```

This reduces blast radius.

---

# Runner and Kubernetes

A self-hosted runner can deploy to Kubernetes.

Example:

```text
GitHub Actions
      |
      ↓
Self-Hosted Runner
      |
      ↓
kubectl / Helm
      |
      ↓
EKS
```

The runner needs appropriate Kubernetes authentication and authorization.

---

# Kubernetes RBAC

Do not automatically give:

```text
cluster-admin
```

to every deployment runner.

Prefer the minimum required permissions.

Example concept:

```text
Runner
  |
  ↓
Service Identity
  |
  ↓
RBAC
  |
  ↓
Specific Namespace
```

For example, a service may only need access to:

```text
catalogue namespace
```

rather than the entire cluster.

---

# Production Deployment Runner

Example:

```yaml
deploy:

  runs-on:
    - self-hosted
    - linux
    - production

  environment:
    name: production

  steps:

    - name: Checkout
      uses: actions/checkout@v4

    - name: Verify Cluster
      run: kubectl cluster-info

    - name: Deploy
      run: |
        helm upgrade --install catalogue ./helm/catalogue \
          --namespace catalogue \
          --create-namespace

    - name: Verify Rollout
      run: |
        kubectl rollout status \
          deployment/catalogue \
          -n catalogue \
          --timeout=5m
```

---

# Production Deployment Flow

```text
Developer
    |
    ↓
Pull Request
    |
    ↓
GitHub-Hosted Runner
    |
    ├── Build
    ├── Unit Tests
    ├── SonarQube
    └── Trivy
    |
    ↓
Merge
    |
    ↓
UAT
    |
    ↓
Approval
    |
    ↓
Production Runner
    |
    ↓
EKS
```

This separates untrusted CI activity from privileged production deployment.

---

# Self-Hosted Runner + ArgoCD

If GitOps is used, the runner does not necessarily need direct Kubernetes deployment access.

For example:

```text
GitHub Actions
      |
      ↓
Build Image
      |
      ↓
Push Image
      |
      ↓
Update Git Manifest
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

In this model, the runner may only need:

```text
Git
ECR
Repository access
```

instead of direct Kubernetes credentials.

This can reduce runner privileges.

---

# Self-Hosted Runner + GitOps

A production architecture can be:

```text
CI Runner
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

This is a useful way to reduce direct deployment permissions in GitHub Actions.

---

# Runner Blast Radius

Always ask:

```text
If this runner is compromised,
what can it access?
```

For example:

```text
Runner
 |
 ├── AWS Account
 ├── EKS
 ├── ECR
 ├── Internal APIs
 └── Secrets
```

The goal is to minimize the blast radius.

---

# Production Runner Security Checklist

```text
☐ Least-privilege IAM
☐ Restricted network access
☐ Controlled runner group
☐ Protected environment
☐ Limited repository access
☐ No long-lived secrets on disk
☐ Regular OS patching
☐ Runner software updates
☐ Tool version management
☐ Disk cleanup
☐ CPU/memory monitoring
☐ Audit logging
☐ PR isolation
☐ Ephemeral runners where appropriate
☐ Kubernetes RBAC
```

---

# Self-Hosted Runner vs GitHub-Hosted Runner

| Feature | GitHub-Hosted | Self-Hosted |
|---|---|---|
| Infrastructure management | GitHub | Organization |
| Customization | Limited | High |
| Private network access | Requires additional design | Easier |
| Maintenance | Minimal | Required |
| Scaling | Managed | Organization-managed |
| Custom software | Limited | Full control |
| Hardware choice | Limited | Greater control |
| Security responsibility | Shared with platform | More responsibility |
| Persistent state | Generally ephemeral | Depends on design |
| Production access | Requires architecture | Can be placed in private network |

---

# When Not to Use Self-Hosted Runners

Do not choose self-hosted simply because they provide more control.

If the workload can safely run on GitHub-hosted infrastructure:

```text
GitHub-Hosted
    |
    ↓
Simpler
```

Self-hosted runners introduce:

```text
Infrastructure
Maintenance
Patching
Monitoring
Security
Capacity Planning
```

Use them when there is a genuine requirement.

---

# Decision Tree

```text
Does the job require private infrastructure?

        |
       YES
        ↓
  Consider Self-Hosted

        |
       NO
        ↓
Can GitHub-hosted runners satisfy
the workload and security requirements?

        |
       YES
        ↓
GitHub-Hosted

        |
       NO
        ↓
Self-Hosted
```

---

# Best Practices

- Use self-hosted runners only when needed.
- Separate production runners from lower environments.
- Use runner groups to control access.
- Use labels to route workloads correctly.
- Apply least-privilege IAM.
- Avoid long-lived credentials.
- Prefer OIDC and short-lived credentials.
- Keep runners patched.
- Maintain standardized runner images.
- Monitor CPU, memory, disk, and runner health.
- Clean persistent runners safely.
- Prefer ephemeral runners for sensitive workloads where practical.
- Never expose production runners to untrusted PR code.
- Restrict Kubernetes permissions with RBAC.
- Use GitOps when it can reduce direct deployment privileges.
- Automate runner provisioning with IaC.
- Maintain auditability of production deployments.

---

# Common Mistakes

- Using one runner for every environment.
- Allowing arbitrary repositories to use production runners.
- Giving runners administrator permissions.
- Storing permanent cloud credentials on disk.
- Allowing untrusted PRs to run on privileged runners.
- Not patching runner operating systems.
- Not monitoring disk usage.
- Installing tools manually without version control.
- Assuming persistent runners are clean between jobs.
- Giving deployment runners `cluster-admin`.
- Using self-hosted runners when GitHub-hosted runners are sufficient.
- Giving a runner access to an entire cloud account when only a few resources are needed.

---

# Summary

A self-hosted runner is infrastructure managed by the organization that executes GitHub Actions jobs.

Architecture:

```text
GitHub
   |
   ↓
Workflow
   |
   ↓
Self-Hosted Runner
   |
   ↓
Private Infrastructure
```

The main benefits are:

```text
Control
Customization
Private Connectivity
Specialized Infrastructure
```

The main responsibilities are:

```text
Security
Patching
Monitoring
Capacity
Tool Management
Isolation
Lifecycle Management
```

For production:

```text
CI
 |
 ↓
GitHub-Hosted Runner
 |
 ├── Build
 ├── Test
 └── Security
 |
 ↓
Protected Promotion
 |
 ↓
Self-Hosted Production Runner
 |
 ↓
Private Infrastructure
```

Or, with GitOps:

```text
GitHub Actions
      |
      ↓
Build + Security
      |
      ↓
Image + Git Manifest
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

The key principle is:

```text
A self-hosted runner is not just another build server.

It is part of your CI/CD security boundary.
```

---

# Interview Questions

## Basic

1. What is a self-hosted runner?
2. Why would an organization use a self-hosted runner?
3. How do you configure a workflow to use a self-hosted runner?
4. What are runner labels?
5. What are runner groups?
6. What is the difference between GitHub-hosted and self-hosted runners?

## Intermediate

7. How would you install and configure a self-hosted runner?
8. How do you route a job to a specific runner?
9. How do self-hosted runners access private AWS infrastructure?
10. How would you monitor a self-hosted runner?
11. How would you manage tools and versions on self-hosted runners?
12. What are the risks of persistent self-hosted runners?
13. What is an ephemeral runner?
14. Why are artifacts needed when moving files between jobs?
15. How would you troubleshoot a job stuck waiting for a self-hosted runner?

## Advanced / Production

16. Design a self-hosted runner architecture for a company with Development, QA, UAT, and Production environments.
17. How would you prevent an untrusted pull request from executing on a production runner?
18. A production runner has access to EKS and AWS. How would you minimize its blast radius?
19. How would you authenticate a GitHub Actions runner to AWS without storing long-lived access keys?
20. How would you secure a self-hosted runner that deploys to a private EKS cluster?
21. Why should you avoid giving `cluster-admin` to a production deployment runner?
22. How would you design an ephemeral runner architecture for production?
23. A persistent runner has 95% disk usage because of Docker images and build caches. How would you safely resolve the issue?
24. How would you create a standardized golden image for 50 self-hosted runners?
25. How would you automate runner provisioning using Terraform?
26. How would you separate CI runners from production deployment runners?
27. How can GitOps with ArgoCD reduce the privileges required by a GitHub Actions runner?
28. A malicious pull request executes arbitrary commands. Explain why allowing it to run on a privileged self-hosted runner is dangerous.
29. Design a production deployment architecture using GitHub-hosted runners for CI and self-hosted runners for private deployment.
30. Compare persistent and ephemeral runners from security, performance, maintenance, and cost perspectives.
31. What controls would you implement before allowing a self-hosted runner to deploy to production?
32. How would you design runner monitoring, patching, cleanup, and replacement as an automated lifecycle?