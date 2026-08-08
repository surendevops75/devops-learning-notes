# GitHub-Hosted Runners

A **runner** is the machine that executes the jobs defined in a GitHub Actions workflow.

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package
```

Here:

```yaml
runs-on: ubuntu-latest
```

tells GitHub Actions to execute the `build` job on a GitHub-hosted Ubuntu runner.

---

# Runner Architecture

The basic execution model is:

```text
GitHub Repository
       |
       ↓
Workflow
       |
       ↓
Job
       |
       ↓
Runner
       |
       ↓
Steps
       |
       ├── Checkout
       ├── Build
       ├── Test
       └── Deploy
```

The runner is responsible for executing the commands and Actions defined by the job.

---

# GitHub-Hosted vs Self-Hosted

GitHub Actions supports two major runner models:

```text
GitHub-Hosted Runner
        |
        └── GitHub manages the machine

Self-Hosted Runner
        |
        └── Organization manages the machine
```

This folder covers both models.

---

# Basic GitHub-Hosted Runner

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Build
        run: mvn clean package
```

The job runs on a GitHub-hosted Ubuntu environment.

---

# Common Runner Labels

Common GitHub-hosted labels include:

```text
ubuntu-latest
ubuntu-24.04
ubuntu-22.04

windows-latest

macos-latest
```

The exact available labels can change over time, so production workflows should use runner labels supported by the organization and current GitHub documentation.

---

# runs-on

The `runs-on` property specifies the runner environment.

Example:

```yaml
jobs:

  test:

    runs-on: ubuntu-latest

    steps:
      - run: echo "Running tests"
```

Another example:

```yaml
jobs:

  test:

    runs-on: windows-latest

    steps:
      - run: echo "Running on Windows"
```

---

# Linux Runner

Linux is commonly used for DevOps and CI/CD workloads.

Example:

```yaml
runs-on: ubuntu-latest
```

Typical tools:

```text
Docker
Terraform
kubectl
Helm
AWS CLI
Azure CLI
Java
Node.js
Python
Maven
Git
```

Linux runners are particularly convenient for cloud-native pipelines.

---

# Windows Runner

Example:

```yaml
runs-on: windows-latest
```

Windows runners are useful when applications or build tooling require Windows.

Examples:

```text
.NET
Windows-specific applications
PowerShell
Windows build tooling
```

---

# macOS Runner

Example:

```yaml
runs-on: macos-latest
```

macOS runners are commonly used for Apple-platform development and testing.

---

# Runner Selection

Choose the runner based on the workload.

```text
Linux application
       |
       ↓
ubuntu-latest
```

```text
Windows-specific application
       |
       ↓
windows-latest
```

```text
Apple platform build
       |
       ↓
macos-latest
```

---

# Runner Environment

A GitHub-hosted runner provides an execution environment with commonly required tools and environment variables.

However, do not assume every required tool/version is permanently available.

For reproducibility, explicitly configure critical versions where appropriate.

Example:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    java-version: '21'
    distribution: temurin
```

Instead of depending on an unspecified Java version already present on the runner.

---

# Runner Lifecycle

A useful conceptual model for GitHub-hosted runners is:

```text
Job Queued
    |
    ↓
Runner Allocated
    |
    ↓
Job Executes
    |
    ↓
Job Completes
    |
    ↓
Runner Environment Removed
```

This means you should not design a workflow assuming that files or state will persist on the runner for future jobs.

---

# Jobs Run on Separate Runners

Consider:

```yaml
jobs:

  build:
    runs-on: ubuntu-latest

  deploy:
    runs-on: ubuntu-latest
```

These are separate jobs.

Conceptually:

```text
Build Job
   |
   └── Runner A

Deploy Job
   |
   └── Runner B
```

Runner A's filesystem should not be treated as available to Runner B.

---

# Passing Files Between Jobs

If one job creates a file:

```text
Build Job
   |
   └── application.jar
```

another job should not assume the file exists:

```text
Deploy Job
   |
   └── application.jar ❌
```

Use an artifact:

```text
Build
  |
  ↓
Upload Artifact
  |
  ↓
Deploy
  |
  ↓
Download Artifact
```

Example:

```yaml
- name: Upload Artifact
  uses: actions/upload-artifact@v4
  with:
    name: application
    path: target/*.jar
```

---

# Runner and Job Isolation

Each job should be treated as an isolated execution unit.

Example:

```text
Job A
  |
  └── Runner A
       └── temporary workspace

Job B
  |
  └── Runner B
       └── temporary workspace
```

Do not rely on:

```text
files
processes
environment variables
installed tools
```

from one job being available in another job.

Use explicit mechanisms to transfer what is required.

---

# GitHub-Hosted Runner Advantages

GitHub-hosted runners provide:

- Managed infrastructure
- No runner server maintenance
- Easy scaling
- Standard environments
- Multiple operating systems
- Integration with GitHub Actions
- Convenient setup for CI workloads

The organization does not need to maintain the underlying runner machine.

---

# GitHub-Hosted Runner Limitations

Depending on the workload and plan, limitations can include:

- Execution limits
- Resource limitations
- Startup time
- Limited customization
- Network restrictions
- No direct access to private infrastructure by default
- Dependency on hosted runner availability

For workloads requiring private network access or deep customization, self-hosted runners may be more appropriate.

---

# Public vs Private Infrastructure

Consider:

```text
Application
    |
    ↓
Private EKS Cluster
    |
    ↓
Private Network
```

A GitHub-hosted runner may not automatically have network connectivity to the private infrastructure.

You must design secure connectivity or use an appropriate self-hosted runner architecture.

---

# GitHub-Hosted Runner and AWS

A CI workflow might:

```text
GitHub
   |
   ↓
GitHub-Hosted Runner
   |
   ↓
AWS Authentication
   |
   ↓
ECR
   |
   ↓
EKS
```

For AWS authentication, prefer short-lived credentials and federated identity mechanisms such as GitHub OIDC where supported rather than long-lived static AWS access keys.

---

# GitHub OIDC Concept

The conceptual flow is:

```text
GitHub Actions
      |
      ↓
OIDC Identity Token
      |
      ↓
AWS IAM Trust Policy
      |
      ↓
Temporary AWS Credentials
      |
      ↓
AWS Resources
```

This reduces the need to store long-lived cloud credentials in GitHub secrets.

---

# Production AWS Example

Conceptually:

```yaml
permissions:
  id-token: write
  contents: read
```

Then use an approved AWS authentication Action to establish temporary credentials.

After authentication:

```yaml
- name: Login to AWS
  ...

- name: Push Image
  run: ...

- name: Deploy to EKS
  run: ...
```

The exact authentication configuration should follow the organization's AWS IAM and GitHub OIDC security model.

---

# Runner Labels

Runner labels identify the capabilities or characteristics of a runner.

GitHub-hosted example:

```yaml
runs-on: ubuntu-latest
```

Self-hosted example:

```yaml
runs-on:
  - self-hosted
  - linux
  - x64
  - production
```

The job can be routed to a runner matching the required labels.

Detailed label management is covered in:

```text
03-Runner-Labels.md
```

---

# Runner Groups

Organizations can group runners and control which repositories or workflows can use them.

Conceptually:

```text
Organization
     |
     ↓
Runner Group
     |
     ├── Runner 1
     ├── Runner 2
     └── Runner 3
```

Runner groups are especially useful for enterprise governance.

Detailed runner group management is covered in:

```text
04-Runner-Groups.md
```

---

# Self-Hosted Runner

A self-hosted runner is a machine managed by the organization.

Conceptually:

```text
Organization
     |
     ↓
Self-Hosted Runner
     |
     ├── OS
     ├── Tools
     ├── Network
     └── Security Controls
```

Advantages include greater control over:

- Software
- Networking
- Hardware
- Custom dependencies
- Internal connectivity
- Security controls

---

# When to Use GitHub-Hosted Runners

GitHub-hosted runners are often a good choice when:

```text
Build
Test
Scan
Package
```

can run using public/managed infrastructure without requiring special private connectivity.

Example:

```text
GitHub
  |
  ↓
Hosted Runner
  |
  ├── Checkout
  ├── Build
  ├── Test
  ├── Security Scan
  └── Artifact
```

---

# When to Consider Self-Hosted Runners

Consider self-hosted runners when the workflow requires:

```text
Private network access
Internal databases
Private Kubernetes clusters
Custom hardware
Special software
GPU workloads
Custom security controls
Internal deployment tools
```

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
   ├── Database
   └── Internal Services
```

---

# GitHub-Hosted Runner for CI

A typical CI pipeline:

```text
Git Push
   |
   ↓
GitHub Actions
   |
   ↓
GitHub-Hosted Runner
   |
   ├── Checkout
   ├── Build
   ├── Unit Test
   ├── SonarQube
   ├── Docker Build
   └── Trivy
```

This is a common use case.

---

# Self-Hosted Runner for Deployment

A production environment may use:

```text
GitHub Actions
      |
      ↓
Self-Hosted Runner
      |
      ↓
Private Network
      |
      ↓
Kubernetes / EKS
```

The runner can have the required private connectivity.

This architecture must be secured carefully.

---

# Production Runner Security

Runners execute workflow code.

Therefore, treat runners as security-sensitive infrastructure.

Important controls include:

- Least privilege
- Network restrictions
- Runner isolation
- Controlled repository access
- Secure credentials
- Regular patching
- Monitoring
- Logging
- Limited administrative access
- Runner lifecycle management

---

# Do Not Store Long-Lived Secrets on Runners

Avoid permanently storing:

```text
AWS access keys
SSH private keys
Cloud credentials
API tokens
Database passwords
```

on runner machines.

Prefer:

```text
OIDC
Short-lived credentials
Secret management systems
Environment protection
```

---

# Self-Hosted Runner Persistence Risk

A persistent self-hosted runner may retain:

```text
Workspace files
Build artifacts
Credentials
Caches
Temporary files
Docker images
Logs
```

after a workflow finishes.

If an untrusted workflow executes on that runner, this can create security risks.

Runner cleanup and isolation are therefore critical.

---

# Ephemeral Runner Concept

An ephemeral runner exists for a limited workload.

Conceptually:

```text
Job
 |
 ↓
Create Runner
 |
 ↓
Execute Job
 |
 ↓
Destroy Runner
```

This reduces long-lived state.

For large-scale Kubernetes-based environments, Actions Runner Controller can help manage runner workloads.

Detailed ARC coverage is in:

```text
05-Actions-Runner-Controller.md
```

---

# Runner Selection Example

Suppose you have:

```text
ubuntu-latest
self-hosted-private
self-hosted-production
```

A workflow can explicitly choose:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

This prevents a production deployment job from accidentally running on an unsuitable runner.

---

# Production Runner Architecture

A strong enterprise architecture can look like:

```text
                    GitHub
                       |
                GitHub Actions
                       |
          ┌────────────┴────────────┐
          ↓                         ↓
     Hosted Runners          Self-Hosted Runners
          |                         |
          ↓                         ↓
       CI / Test              Private Network
                                    |
                          ┌─────────┼─────────┐
                          ↓         ↓         ↓
                         EKS      Internal   Tools
```

This separates general CI workloads from private infrastructure workloads.

---

# Build vs Deployment Separation

A useful strategy is:

```text
GitHub-Hosted Runner
        |
        ↓
Build
        |
        ↓
Security Scan
        |
        ↓
Artifact / Image
        |
        ↓
Self-Hosted Runner
        |
        ↓
Private Deployment
```

This can reduce exposure of the production network to general CI workloads.

---

# Runner Trust Boundaries

Think about runners as trust boundaries:

```text
Public Repository
       |
       ↓
Untrusted Code
       |
       ↓
Runner
```

versus:

```text
Trusted Repository
       |
       ↓
Protected Workflow
       |
       ↓
Production Runner
```

Production runners should not be freely available to untrusted workflows.

---

# Pull Request Security

Be particularly careful with workflows triggered by:

```text
Pull Requests
Forks
Untrusted Contributors
```

A workflow that executes attacker-controlled code should not automatically receive access to production credentials or sensitive self-hosted infrastructure.

---

# Production Environment Separation

A strong architecture is:

```text
PR
 |
 ↓
GitHub-Hosted Runner
 |
 ├── Build
 ├── Test
 └── Security
       |
       ↓
     Merge
       |
       ↓
Protected Branch
       |
       ↓
Production Workflow
       |
       ↓
Protected Environment
       |
       ↓
Production Runner
```

This creates multiple security controls.

---

# Runner Tool Management

Self-hosted runners require management of:

```text
Docker
kubectl
Helm
Terraform
AWS CLI
Azure CLI
Java
Node.js
Python
Security tools
```

Keep versions controlled and documented.

Avoid uncontrolled manual changes.

---

# Immutable Runner Image

For enterprise self-hosted infrastructure, consider maintaining a standard runner image.

Example:

```text
Runner Image
 |
 ├── Linux
 ├── Docker
 ├── kubectl
 ├── Helm
 ├── Terraform
 ├── AWS CLI
 ├── Security tools
 └── Monitoring
```

Then provision runners from this known baseline.

---

# Runner Patching

Self-hosted runners must be patched regularly.

Patch:

```text
Operating System
Runner Software
Docker
kubectl
Helm
Terraform
Cloud CLI
Security Packages
```

A vulnerability in the runner can affect every workflow executed there.

---

# Runner Monitoring

Monitor:

```text
CPU
Memory
Disk
Network
Runner status
Job failures
Software versions
Security events
```

Especially monitor persistent production runners.

---

# Disk Management

Builds involving Docker can consume significant disk space.

For example:

```text
Docker Images
Docker Layers
Build Cache
Artifacts
Logs
Temporary Files
```

Implement cleanup policies.

Example:

```bash
docker system df
```

and appropriate controlled cleanup procedures.

Do not blindly run destructive cleanup commands on shared runners.

---

# Runner Availability

If a self-hosted runner is unavailable:

```text
Job
 |
 ↓
Waiting
 |
 ↓
No matching runner
```

Possible causes:

- Runner offline
- Incorrect label
- Runner group restriction
- Capacity exhausted
- Network issue
- Runner service stopped

---

# Troubleshooting GitHub-Hosted Runner

Check:

```text
1. runs-on label
2. Workflow syntax
3. Runner availability
4. Job logs
5. Tool setup
6. Permissions
7. External network access
```

---

# Troubleshooting Self-Hosted Runner

Check:

```text
1. Runner online status
2. Runner labels
3. Runner group access
4. Runner service
5. Network connectivity
6. Disk space
7. CPU / memory
8. Tool versions
9. Permissions
10. Security restrictions
```

---

# Common Runner Mistake

Incorrect assumption:

```text
Job A installs Terraform
       |
       ↓
Job B can use Terraform
```

Not necessarily.

Because:

```text
Job A → Runner A

Job B → Runner B
```

Instead, configure Terraform in every job that needs it:

```yaml
- name: Setup Terraform
  uses: hashicorp/setup-terraform@v3
```

or use an approved runner image that already contains the required tooling.

---

# Common Runner Mistake

Do not assume:

```text
ubuntu-latest
```

means the exact same environment forever.

Managed runner images evolve.

For critical tools:

```text
Explicitly configure versions
```

Example:

```yaml
- name: Setup Node
  uses: actions/setup-node@v4
  with:
    node-version: '22'
```

---

# Common Runner Mistake

Do not use a production self-hosted runner for arbitrary PR workloads.

Bad:

```text
Any PR
  |
  ↓
Production Runner
  |
  ↓
Production Network
```

Better:

```text
PR
 |
 ↓
Hosted / Isolated Runner
 |
 ↓
Build + Test
```

and:

```text
Protected Production Workflow
 |
 ↓
Production Runner
```

---

# Production Deployment Example

```yaml
name: Production Deployment

on:
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

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

      - name: Verify Rollout
        run: |
          kubectl rollout status \
            deployment/catalogue \
            -n catalogue \
            --timeout=5m
```

This demonstrates:

```text
Manual Trigger
     |
     ↓
Production Runner
     |
     ↓
Protected Environment
     |
     ↓
Deploy
     |
     ↓
Verification
```

---

# Hosted Runner + Self-Hosted Deployment

A more controlled pipeline can separate CI and deployment:

```text
                 GitHub
                    |
                    ↓
          GitHub-Hosted Runner
                    |
          ┌─────────┼─────────┐
          ↓         ↓         ↓
        Build      Test     Security
          |
          ↓
      Image / Artifact
          |
          ↓
     Protected Job
          |
          ↓
    Self-Hosted Runner
          |
          ↓
     Private Network
          |
          ↓
          EKS
```

This is a useful production architecture when private connectivity is required.

---

# Decision Guide

Use GitHub-hosted runners when:

```text
✓ Standard CI
✓ Public/managed dependencies
✓ No special network access
✓ Easy scaling
✓ Minimal infrastructure management
```

Consider self-hosted runners when:

```text
✓ Private infrastructure
✓ Custom tooling
✓ Special hardware
✓ Private network
✓ Internal services
✓ Specialized security requirements
```

---

# Interview Questions

## Basic

1. What is a GitHub Actions runner?
2. What is a GitHub-hosted runner?
3. What is a self-hosted runner?
4. What does `runs-on` do?
5. What are common GitHub-hosted runner labels?
6. Can different jobs run on different runners?

## Intermediate

7. What is the difference between GitHub-hosted and self-hosted runners?
8. Why don't files automatically persist between jobs?
9. How do you pass artifacts between jobs?
10. Why would an organization choose a self-hosted runner?
11. What are runner labels?
12. What are runner groups?
13. How would you troubleshoot a job stuck waiting for a runner?
14. How do you manage tools and versions on self-hosted runners?

## Advanced / Production

15. Design a GitHub Actions architecture where CI runs on GitHub-hosted runners but production deployment runs on self-hosted runners inside a private AWS network.
16. How would you prevent untrusted pull requests from executing on a production self-hosted runner?
17. Explain the security risks of persistent self-hosted runners.
18. How would you design ephemeral runners for production workloads?
19. How would you secure AWS access from a GitHub-hosted runner?
20. Explain how GitHub OIDC can replace long-lived AWS credentials.
21. A production deployment job is stuck waiting for a runner. Walk through your troubleshooting process.
22. A self-hosted runner has accumulated Docker images and disk usage is 95%. How would you safely resolve it?
23. How would you standardize the software installed on 50 self-hosted runners?
24. Design a runner architecture for an enterprise with development, QA, UAT, and production environments.
25. How would you separate trusted production workflows from untrusted pull-request workflows?
26. When would you use GitHub-hosted runners instead of self-hosted runners?
27. What are the security considerations when allowing production runners to access an EKS cluster?
28. Explain why installing a tool in one GitHub Actions job does not guarantee that another job can use it.
29. How would you design runner monitoring, patching, and lifecycle management for production?
30. Compare persistent self-hosted runners with ephemeral runners from a security and operational perspective.