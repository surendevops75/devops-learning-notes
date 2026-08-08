# GitHub Actions Runner Labels

Runner labels are used to identify and route GitHub Actions jobs to appropriate runners.

They are especially important when an organization has multiple self-hosted runners with different:

- Operating systems
- Environments
- Architectures
- Tools
- Network access
- Hardware
- Security requirements

Basic example:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

This tells GitHub Actions to select a runner matching the requested labels.

---

# Why Runner Labels Matter

Consider an organization with:

```text
Runner 1
  ├── Linux
  └── QA

Runner 2
  ├── Linux
  └── UAT

Runner 3
  ├── Linux
  └── Production
```

A production deployment should not accidentally run on the QA runner.

Use:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

This helps route the job to the appropriate runner.

---

# Basic Label Concept

Think of labels as characteristics attached to a runner.

Example:

```text
Runner
 |
 ├── self-hosted
 ├── linux
 ├── x64
 └── production
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
  - production
```

The runner must satisfy the requested labels.

---

# `self-hosted` Label

Self-hosted runners have the `self-hosted` label.

Example:

```yaml
runs-on:
  - self-hosted
```

This tells GitHub Actions that the job should run on a self-hosted runner.

However, if many self-hosted runners exist, this may be too broad.

Better:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

---

# Operating System Labels

You can use labels to distinguish operating systems.

Example:

```text
linux
windows
macos
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
```

Another:

```yaml
runs-on:
  - self-hosted
  - windows
```

---

# Architecture Labels

Labels can distinguish CPU architectures.

Example:

```text
x64
arm64
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - arm64
```

This is useful when a workload requires a specific architecture.

---

# Environment Labels

Environment-specific labels can help separate infrastructure.

Example:

```text
dev
qa
uat
production
```

Production:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

QA:

```yaml
runs-on:
  - self-hosted
  - linux
  - qa
```

---

# Tool-Based Labels

Labels can identify specialized capabilities.

Example:

```text
docker
terraform
kubernetes
gpu
```

A workflow can request:

```yaml
runs-on:
  - self-hosted
  - linux
  - terraform
```

This indicates that the job requires a runner with the expected Terraform capability.

However, labels should represent a real, maintained capability. A label by itself does not install or verify the software.

---

# Network-Based Labels

Labels can help identify runners with specific network connectivity.

Example:

```text
private-network
eks
internal
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - private-network
```

This can route jobs toward runners intended to access private infrastructure.

Do not treat a label as a security boundary by itself. Access controls and network policies must enforce the actual restriction.

---

# Production Runner Labels

A production runner might have:

```text
self-hosted
linux
x64
production
private-network
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
  - production
  - private-network
```

This makes the intended runner characteristics explicit.

---

# Label Matching

Suppose:

```text
Runner A:
self-hosted
linux
qa

Runner B:
self-hosted
linux
production
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

Runner B matches.

Runner A does not.

Conceptually:

```text
Requested:
self-hosted
linux
production

Runner A:
self-hosted ✓
linux ✓
production ✗

Runner B:
self-hosted ✓
linux ✓
production ✓
```

---

# Multiple Labels

A job can request multiple labels.

Example:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
  - production
```

The combination becomes more specific.

Use this when the job genuinely requires all requested characteristics.

---

# Avoid Over-Labeling

Do not add dozens of labels without a clear purpose.

Bad:

```text
self-hosted
linux
ubuntu
ubuntu22
x64
prod
production
aws
aws-prod
eks
k8s
kubernetes
docker
helm
terraform
...
```

This can become difficult to maintain.

Prefer a small, meaningful label strategy.

Example:

```text
self-hosted
linux
production
```

Add specialized labels only when they are actually needed.

---

# Label Naming Convention

Choose a consistent naming convention.

Example:

```text
environment:
qa
uat
production

os:
linux
windows

architecture:
x64
arm64
```

Or use descriptive labels:

```text
qa
production
linux
arm64
private-network
```

Consistency is more important than a specific naming style.

---

# Environment Label Strategy

A simple enterprise model:

```text
Development
    ↓
dev

QA
    ↓
qa

UAT
    ↓
uat

Production
    ↓
production
```

Then:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

---

# Runner Pool Example

Suppose:

```text
Production Runner Pool

prod-runner-01
  self-hosted
  linux
  production

prod-runner-02
  self-hosted
  linux
  production

prod-runner-03
  self-hosted
  linux
  production
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

Any eligible production runner can execute the job.

---

# Runner Pool Architecture

```text
                 Production
                     |
                     ↓
                Runner Label
                 production
                     |
          ┌──────────┼──────────┐
          ↓          ↓          ↓
      Runner 01   Runner 02   Runner 03
```

This provides a pool rather than depending on one specific machine.

---

# Runner Labels and Load Distribution

Labels select eligible runners.

If multiple runners match:

```text
production
    |
    ├── Runner 01
    ├── Runner 02
    └── Runner 03
```

GitHub can assign jobs among available eligible runners according to its scheduling behavior.

Do not assume a particular runner will always be selected.

---

# Do Not Target a Specific Hostname Unless Necessary

Avoid designing workflows around:

```text
prod-runner-01
```

if any production runner can perform the job.

Prefer:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

This improves resilience.

---

# Specialized Runner

Suppose one runner has a GPU.

Labels:

```text
self-hosted
linux
gpu
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - gpu
```

Only runners with the required labels are eligible.

---

# ARM Runner

Example:

```yaml
runs-on:
  - self-hosted
  - linux
  - arm64
```

This is useful for workloads that specifically need ARM architecture.

---

# Windows Runner

```yaml
runs-on:
  - self-hosted
  - windows
```

The runner pool could be:

```text
Windows Runner Group

runner-01
runner-02
runner-03
```

with labels:

```text
self-hosted
windows
```

---

# Private Network Runner

Example:

```yaml
runs-on:
  - self-hosted
  - linux
  - private-network
```

Architecture:

```text
GitHub
   |
   ↓
Workflow
   |
   ↓
private-network runner
   |
   ↓
Private VPC
```

The label identifies the intended runner class, while actual network controls enforce connectivity.

---

# EKS Deployment Runner

A runner intended for EKS deployments could have:

```text
self-hosted
linux
eks
production
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - eks
  - production
```

This is useful when only specific runners have the required private network and deployment tooling.

---

# GitOps Runner

If ArgoCD is used, the runner may not need direct Kubernetes access.

For example:

```text
self-hosted
linux
gitops
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - gitops
```

The runner can:

```text
Build
Scan
Push Image
Update Git Manifest
```

and ArgoCD performs the cluster reconciliation.

---

# Label vs Security

Important:

```text
Label ≠ Security Control
```

A label such as:

```text
production
```

does not by itself prevent unauthorized access.

Security should also use:

```text
Runner Groups
Repository Access
Workflow Permissions
Environment Protection
IAM
Network Controls
Kubernetes RBAC
Branch Protection
```

---

# Label + Runner Group + Environment

A strong production model combines several controls.

```text
Workflow
   |
   ↓
Runner Group
   |
   ↓
Runner Labels
   |
   ↓
Production Runner
   |
   ↓
Production Environment
   |
   ↓
Approval
   |
   ↓
Deployment
```

Each mechanism has a different purpose.

---

# Runner Labels and Runner Groups

Labels answer:

```text
"What capabilities/type of runner do I need?"
```

Runner groups answer:

```text
"Which runners can this repository/workflow access?"
```

Example:

```text
Production Runner Group
       |
       ├── prod-01
       ├── prod-02
       └── prod-03

Labels:
self-hosted
linux
production
```

---

# Environment Protection

Runner labels do not replace environment protection.

Example:

```yaml
jobs:

  deploy:

    runs-on:
      - self-hosted
      - linux
      - production

    environment:
      name: production
```

The environment can provide additional deployment controls such as required reviewers, depending on the organization's GitHub configuration.

---

# Labels and Untrusted PRs

Avoid:

```text
pull_request
     |
     ↓
production runner
```

just because the runner has:

```text
production
```

A malicious PR can potentially execute workflow commands.

Use trusted workflow boundaries and appropriate runner access restrictions.

---

# Label Strategy for CI and CD

A useful architecture:

```text
CI Runners
 |
 ├── self-hosted
 ├── linux
 └── ci

Deployment Runners
 |
 ├── self-hosted
 ├── linux
 └── production
```

Workflow:

```yaml
# CI
runs-on:
  - self-hosted
  - linux
  - ci
```

Production:

```yaml
# CD
runs-on:
  - self-hosted
  - linux
  - production
```

---

# Development / QA / UAT / Production

A common strategy:

```text
dev
qa
uat
production
```

Runner labels:

```text
self-hosted
linux
dev
```

```text
self-hosted
linux
qa
```

```text
self-hosted
linux
uat
```

```text
self-hosted
linux
production
```

This makes environment routing explicit.

---

# Multi-Environment Pipeline

```text
Build
  |
  ↓
CI Runner
  |
  ↓
QA Runner
  |
  ↓
UAT Runner
  |
  ↓
Production Runner
```

However, do not rely only on labels for promotion controls.

Use:

```text
needs
environments
approvals
branch protection
change management
```

where required.

---

# Change Request Process

A production workflow may require:

```text
JIRA Ticket
     |
     ↓
Approval
     |
     ↓
Production Workflow
     |
     ↓
Production Runner
```

The runner label can route the deployment:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

The change-control validation should be implemented separately.

---

# Production Workflow Example

```yaml
name: Production Deploy

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:

  deploy:

    runs-on:
      - self-hosted
      - linux
      - production

    environment:
      name: production

    timeout-minutes: 30

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Verify Runner
        run: |
          uname -a
          kubectl version --client
          helm version

      - name: Deploy
        run: |
          helm upgrade --install catalogue ./helm/catalogue \
            --namespace catalogue \
            --create-namespace

      - name: Verify
        run: |
          kubectl rollout status \
            deployment/catalogue \
            -n catalogue \
            --timeout=5m
```

---

# Labels for Tool Versions

Avoid labels such as:

```text
terraform-1.8.5
```

unless there is a strong operational reason to route jobs this way.

A better approach is often to install/setup the required tool version explicitly:

```yaml
- name: Setup Terraform
  uses: hashicorp/setup-terraform@v3
  with:
    terraform_version: '1.8.5'
```

Labels should identify runner capabilities or placement, not become a replacement for dependency management.

---

# Labels for Docker

A runner could have:

```text
self-hosted
linux
docker
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - docker
```

But ensure the Docker capability is actually maintained.

A label is only useful if its meaning is documented and enforced operationally.

---

# Labels for Terraform

Example:

```text
self-hosted
linux
terraform
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - terraform
```

Again, the label does not install Terraform.

---

# Capability Labels

Examples:

```text
docker
terraform
kubernetes
gpu
arm64
private-network
```

Use these when the runner genuinely provides the required capability.

---

# Environment Labels vs Capability Labels

Environment:

```text
qa
uat
production
```

Capability:

```text
docker
terraform
kubernetes
gpu
arm64
```

These answer different questions.

```text
Environment
→ Where should this job run?

Capability
→ What does this runner need to support?
```

---

# Combining Environment and Capability

Example:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
  - kubernetes
```

This means the workflow needs an eligible runner intended for production and Kubernetes-related workloads.

---

# Runner Label Governance

As organizations grow, unmanaged labels can become confusing.

Example:

```text
prod
production
prod-env
prod-runner
production-runner
production-vpc
```

These may all mean similar things.

Define standards:

```text
production
qa
uat
dev
linux
windows
x64
arm64
```

and document custom labels.

---

# Label Inventory

Maintain a simple inventory.

Example:

| Runner | Labels |
|---|---|
| runner-01 | self-hosted, linux, ci |
| runner-02 | self-hosted, linux, qa |
| runner-03 | self-hosted, linux, uat |
| runner-04 | self-hosted, linux, production |
| runner-05 | self-hosted, linux, production, arm64 |

This makes routing easier to understand.

---

# Label Drift

Suppose a runner has:

```text
production
```

but it no longer has production network access.

The label becomes misleading.

This is **label drift**.

Prevent it by:

```text
Automating runner provisioning
Automating labels
Auditing runner configuration
Removing stale runners
Maintaining documentation
```

---

# Label Verification

A production process should periodically verify:

```text
Runner Label
      |
      ↓
Actual Capability
```

For example:

```text
production
      |
      ├── Correct network
      ├── Correct IAM
      ├── Correct tools
      └── Correct security posture
```

---

# Dynamic Infrastructure

If runners are dynamically created and destroyed, labels should be applied consistently during provisioning.

Conceptually:

```text
Provision Runner
      |
      ↓
Apply Standard Labels
      |
      ↓
Register Runner
      |
      ↓
Run Jobs
```

This becomes especially important with autoscaling and Actions Runner Controller.

---

# Labels and ARC

Actions Runner Controller can create runner resources dynamically.

A conceptual architecture:

```text
GitHub Actions
      |
      ↓
Runner Controller
      |
      ↓
Runner Pods
      |
      ├── labels
      ├── environment
      └── capabilities
```

Detailed ARC coverage is in:

```text
05-Actions-Runner-Controller.md
```

---

# Troubleshooting: No Matching Runner

Problem:

```text
Job waiting for runner
```

Check:

```text
1. Runner is online
2. Label exists
3. Labels match exactly
4. Runner belongs to correct scope
5. Runner group allows the repository
6. Runner is not busy
7. Runner has required architecture
```

---

# Troubleshooting Example

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

Runner:

```text
self-hosted
linux
prod
```

The job may not match because:

```text
production ≠ prod
```

Use a consistent naming convention.

---

# Troubleshooting: Runner Offline

Suppose:

```text
production
   |
   ├── runner-01 → offline
   ├── runner-02 → offline
   └── runner-03 → offline
```

A job requesting:

```yaml
runs-on:
  - self-hosted
  - production
```

may remain queued.

Investigate:

```text
Runner service
VM health
Network
Runner registration
Cloud infrastructure
Capacity
```

---

# Troubleshooting: Wrong Environment

If a QA job accidentally runs on a production runner, check:

```text
runs-on
runner labels
runner groups
workflow conditions
environment configuration
```

Example mistake:

```yaml
runs-on:
  - self-hosted
  - linux
```

This is too broad if both QA and production runners share those labels.

Better:

```yaml
runs-on:
  - self-hosted
  - linux
  - qa
```

---

# Production Label Design

Recommended concept:

```text
self-hosted
linux
production
```

Then add only necessary capability labels:

```text
self-hosted
linux
production
kubernetes
```

Avoid excessive combinations.

---

# Label-Based Routing Example

```text
                    Job
                     |
          ┌──────────┼──────────┐
          ↓          ↓          ↓
          CI         QA        PROD
          |          |          |
       ci label   qa label  production
          |          |          |
          ↓          ↓          ↓
       Runner     Runner     Runner
```

This provides clear routing.

---

# Label Strategy with GitOps

If ArgoCD handles deployment:

```text
CI Runner
 |
 ├── linux
 ├── ci
 └── gitops
```

The workflow:

```text
Build
  |
  ↓
Security
  |
  ↓
Push Image
  |
  ↓
Update Manifest
  |
  ↓
ArgoCD
  |
  ↓
EKS
```

The runner may not require:

```text
production
kubernetes
cluster-admin
```

because it is not directly deploying to Kubernetes.

This can reduce privileges.

---

# Label Strategy with Direct Helm Deployment

If GitHub Actions directly deploys with Helm:

```text
Runner
 |
 ├── self-hosted
 ├── linux
 ├── production
 └── kubernetes
```

Then:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
  - kubernetes
```

The runner requires appropriate Kubernetes connectivity and authorization.

---

# Security Principle

Do not use labels as the only security mechanism.

Use layered controls:

```text
Repository
    |
    ↓
Protected Branch
    |
    ↓
Workflow Permissions
    |
    ↓
Runner Group
    |
    ↓
Runner Labels
    |
    ↓
Environment Protection
    |
    ↓
IAM / RBAC
    |
    ↓
Production
```

---

# Best Practices

- Keep labels simple and meaningful.
- Use environment labels for environment routing.
- Use capability labels for specialized workloads.
- Maintain consistent naming.
- Avoid targeting individual runners unnecessarily.
- Do not treat labels as security controls.
- Combine labels with runner groups and environments.
- Keep production runners isolated.
- Avoid exposing production labels to untrusted workflows.
- Regularly audit labels.
- Prevent label drift.
- Automate label assignment where possible.
- Do not use labels as a substitute for tool version management.
- Use GitOps when it can reduce direct production runner privileges.
- Document custom labels.

---

# Common Mistakes

- Using only `self-hosted` when multiple runner environments exist.
- Using inconsistent labels such as `prod` and `production`.
- Assuming labels provide security.
- Adding too many unnecessary labels.
- Targeting a specific runner when a pool is sufficient.
- Using labels instead of proper environment protection.
- Leaving stale labels on runners.
- Giving a production label to a runner without production connectivity.
- Assuming a capability label guarantees the software is installed.
- Allowing untrusted PR workflows to target production labels.

---

# Summary

Runner labels identify runners and help route jobs.

Basic:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

Labels can represent:

```text
Environment
Operating System
Architecture
Capability
Network Placement
```

Example:

```text
self-hosted
linux
x64
production
kubernetes
```

Remember:

```text
Label
→ Routing

Runner Group
→ Access Control

Environment
→ Deployment Protection

IAM / RBAC
→ Resource Authorization
```

A production-grade runner architecture can use:

```text
GitHub
   |
   ↓
Protected Workflow
   |
   ↓
Runner Group
   |
   ↓
Labels
   |
   ↓
Production Runner
   |
   ↓
Private Network
   |
   ↓
EKS
```

The key principle is:

```text
Use labels to route jobs correctly,
but never rely on labels alone for security.
```

---

# Interview Questions

## Basic

1. What are runner labels in GitHub Actions?
2. Why are labels required for self-hosted runners?
3. What does the `self-hosted` label mean?
4. How do you specify multiple runner labels?
5. How do you route a job to a production runner?
6. What is the difference between environment and capability labels?

## Intermediate

7. How does GitHub select a runner when multiple labels are specified?
8. How would you design labels for Development, QA, UAT, and Production?
9. How would you label Linux and Windows runners?
10. How would you distinguish x64 and ARM64 runners?
11. How would you create labels for specialized runners such as GPU runners?
12. What happens if no runner matches the requested labels?
13. How would you troubleshoot a job stuck waiting for a runner?
14. What is label drift?
15. Why should you avoid targeting a specific runner when a runner pool is sufficient?

## Advanced / Production

16. Design a label strategy for an organization with CI, QA, UAT, and production runner pools.
17. Explain why runner labels should not be treated as security controls.
18. How would you combine runner labels, runner groups, environments, IAM, and Kubernetes RBAC for production security?
19. A production job is running on a QA runner. Explain how you would troubleshoot and prevent this.
20. A workflow uses only `self-hosted` while production and QA runners are available. What could go wrong?
21. Design a runner-label strategy for a private EKS deployment environment.
22. How would you prevent untrusted pull requests from using a `production` runner label?
23. How would you manage labels when runners are dynamically created and destroyed?
24. Explain how Actions Runner Controller can work with runner labels.
25. A runner has the `production` label but no longer has production network access. How would you detect and correct this label drift?
26. How would you design labels for a microservices CI/CD platform where some runners have Docker, Terraform, Kubernetes, and private-network capabilities?
27. Why should labels not be used as a replacement for dependency/tool version management?
28. How can GitOps reduce the number of capabilities and labels required on a production runner?
29. Design a secure production deployment using runner groups, labels, environment protection, and Kubernetes RBAC.
30. A job requires `self-hosted`, `linux`, `production`, and `kubernetes`, but remains queued. Walk through your complete troubleshooting process.