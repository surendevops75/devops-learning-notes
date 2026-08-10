# GitHub Actions - Mock Interview

This mock interview is designed to simulate a real DevOps / DevSecOps
interview focused on GitHub Actions.

The questions progress from:

    Basic
        ↓
    Intermediate
        ↓
    Advanced
        ↓
    Scenario-Based
        ↓
    Production
        ↓
    Architecture
        ↓
    DevSecOps

The goal is not only to explain GitHub Actions syntax.

You should demonstrate that you understand:

    CI/CD
        +
    GitHub Actions
        +
    AWS
        +
    Docker
        +
    Kubernetes
        +
    EKS
        +
    Terraform
        +
    Helm
        +
    ArgoCD
        +
    DevSecOps
        +
    Production Operations

---

# PART 1 - BASIC MOCK INTERVIEW

## Question 1

    What is GitHub Actions?

### Expected Answer

GitHub Actions is a CI/CD automation platform integrated with GitHub.

It can automate:

    Build
        +
    Test
        +
    Security Scanning
        +
    Packaging
        +
    Deployment
        +
    Infrastructure Automation

A workflow is defined using YAML files inside:

    .github/workflows/

---

## Question 2

    What is a workflow?

### Expected Answer

A workflow is an automated process defined in a YAML file.

Example flow:

    Push
        |
        ↓
    Workflow
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Deploy

A repository can contain multiple workflows.

---

## Question 3

    What is a job?

### Expected Answer

A job is a group of steps executed together on a runner.

Example:

    Job
        |
        +--- Checkout
        +--- Build
        +--- Test
        +--- Package

Multiple jobs can execute independently or have dependencies.

---

## Question 4

    What is a step?

### Expected Answer

A step is an individual task inside a job.

For example:

    Job
        |
        +--- Checkout Code
        +--- Install Dependencies
        +--- Run Tests
        +--- Build Artifact

A step can run a shell command or use an action.

---

## Question 5

    What is a runner?

### Expected Answer

A runner is the execution environment where GitHub Actions jobs run.

The runner executes:

    Shell Commands
        +
    Actions
        +
    Scripts
        +
    Build Tools

---

## Question 6

    What is the difference between GitHub-hosted and self-hosted
    runners?

### Expected Answer

GitHub-hosted runners are managed by GitHub.

Self-hosted runners are managed by the organization.

GitHub-hosted:

    Less Infrastructure Management
        +
    Easy Setup
        +
    Ephemeral Environment

Self-hosted:

    Custom Environment
        +
    Private Network Access
        +
    Specialized Requirements

---

## Question 7

    Where are GitHub Actions workflows stored?

### Expected Answer

They are stored under:

    .github/workflows/

For example:

    .github/workflows/ci.yml

---

## Question 8

    What triggers a GitHub Actions workflow?

### Expected Answer

Common triggers include:

    push
        +
    pull_request
        +
    workflow_dispatch
        +
    schedule
        +
    workflow_call

The trigger depends on the required automation.

---

## Question 9

    What is workflow_dispatch?

### Expected Answer

It allows a workflow to be manually triggered.

Typical use cases:

    Manual Deployment
        +
    Rollback
        +
    Maintenance
        +
    Operational Tasks

---

## Question 10

    What is workflow_call?

### Expected Answer

workflow_call allows a workflow to be reused by other workflows.

It is useful for:

    Reusable Workflows
        +
    Standardized CI/CD
        +
    Centralized Automation

---

# PART 2 - INTERMEDIATE MOCK INTERVIEW

## Question 11

    What is the difference between jobs and steps?

### Expected Answer

Steps run inside a job.

Jobs can run independently or have dependencies.

Example:

    Job 1
        |
        +--- Build
        +--- Test

    Job 2
        |
        +--- Security Scan

    Job 3
        |
        +--- Deploy

Job 3 can depend on Job 1 and Job 2.

---

## Question 12

    How do you define dependencies between jobs?

### Expected Answer

Use job dependencies.

Conceptually:

    Build
        |
        ↓
    Test
        |
        ↓
    Deploy

The deployment job should only start after the required validation
jobs succeed.

---

## Question 13

    How do you run jobs in parallel?

### Expected Answer

Jobs run in parallel when they do not depend on each other.

Example:

    Build
        |
        +--- Unit Test
        |
        +--- Security Scan
        |
        +--- Lint

This can reduce pipeline duration.

---

## Question 14

    How would you reduce GitHub Actions pipeline execution time?

### Expected Answer

I would analyze the duration first.

Then optimize:

    Parallel Jobs
        +
    Dependency Cache
        +
    Docker Build Cache
        +
    Path Filters
        +
    Test Parallelization
        +
    Reusable Workflows
        +
    Appropriate Runners

---

## Question 15

    What is caching in GitHub Actions?

### Expected Answer

Caching stores reusable data between workflow runs.

Examples:

    Maven Dependencies
        +
    npm Dependencies
        +
    Docker Build Layers

Caching reduces repeated downloads and can improve build speed.

---

## Question 16

    What is an artifact in GitHub Actions?

### Expected Answer

An artifact is a file or collection of files produced by a workflow
and preserved for later use.

Examples:

    Test Reports
        +
    Build Packages
        +
    Logs
        +
    Deployment Metadata

---

## Question 17

    What is the difference between an artifact and a Docker image?

### Expected Answer

A GitHub Actions artifact is typically used to pass or preserve
workflow outputs.

A Docker image is a container artifact designed to run an
application.

For production container deployments:

    Build
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    Kubernetes

---

## Question 18

    What are GitHub Actions secrets?

### Expected Answer

Secrets are protected values used by workflows.

Examples:

    API Keys
        +
    Tokens
        +
    Passwords

However, for AWS authentication I prefer OIDC over long-lived AWS
access keys.

---

## Question 19

    What are environment variables?

### Expected Answer

Environment variables provide configuration values to workflow steps.

They can exist at different scopes such as:

    Workflow
        +
    Job
        +
    Step

They should not be used carelessly for sensitive credentials.

---

## Question 20

    What is the difference between secrets and variables?

### Expected Answer

Variables are intended for normal configuration values.

Secrets are intended for sensitive values and receive additional
protection.

Sensitive information should not be stored as normal variables.

---

# PART 3 - GITHUB ACTIONS SECURITY

## Question 21

    How would you securely authenticate GitHub Actions with AWS?

### Expected Answer

I would use OIDC.

Architecture:

    GitHub Actions
        |
        ↓
    OIDC Token
        |
        ↓
    AWS IAM Trust Policy
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS

This avoids long-lived AWS access keys.

---

## Question 22

    Why is OIDC better than storing AWS access keys?

### Expected Answer

Long-lived access keys can remain valid until manually rotated or
revoked.

OIDC provides:

    Short-Lived Credentials
        +
    Federated Identity
        +
    Restricted Trust
        +
    Better Security

---

## Question 23

    How would you restrict an AWS OIDC role?

### Expected Answer

I would restrict the trust relationship based on appropriate
attributes such as:

    Repository
        +
    Organization
        +
    Branch
        +
    Environment

The production role should only be accessible to the intended
production workflow.

---

## Question 24

    What is least privilege?

### Expected Answer

Least privilege means giving an identity only the permissions it
needs.

For GitHub Actions:

    Minimal GitHub Permissions
        +
    Minimal AWS IAM Permissions
        +
    Minimal Kubernetes RBAC

---

## Question 25

    What is the risk of excessive GITHUB_TOKEN permissions?

### Expected Answer

If a workflow is compromised, excessive permissions increase the
blast radius.

Therefore permissions should be explicitly minimized.

---

## Question 26

    How would you secure third-party GitHub Actions?

### Expected Answer

I would:

    Review the Action
        +
    Use Trusted Actions
        +
    Pin Versions
        +
    Prefer Immutable References
        +
    Limit Permissions
        +
    Monitor Updates

For high-security environments, pinning to a specific commit SHA
provides stronger immutability.

---

## Question 27

    What is the risk of running untrusted pull request code?

### Expected Answer

The code may attempt to:

    Access Secrets
        +
    Modify Files
        +
    Execute Commands
        +
    Abuse Runner Permissions

Therefore untrusted PR workflows should not receive production
credentials.

---

## Question 28

    Should production secrets be available to forked pull requests?

### Expected Answer

No.

Forked pull requests are untrusted execution contexts.

Production credentials should not be exposed to them.

---

# PART 4 - CI/CD DESIGN MOCK INTERVIEW

## Question 29

    Design a CI pipeline for a Java Maven application.

### Expected Answer

I would design:

    Checkout
        |
        ↓
    Setup JDK
        |
        ↓
    Cache Maven Dependencies
        |
        ↓
    Unit Tests
        |
        ↓
    SonarQube
        |
        ↓
    Security Scan
        |
        ↓
    Maven Package
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Push ECR

---

## Question 30

    Design a CI pipeline for a Node.js application.

### Expected Answer

    Checkout
        |
        ↓
    Setup Node.js
        |
        ↓
    npm ci
        |
        ↓
    Lint
        |
        ↓
    Unit Tests
        |
        ↓
    Security Scan
        |
        ↓
    Build
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    ECR

---

## Question 31

    How would you design CI for a microservices monorepo?

### Expected Answer

I would use:

    Path Detection
        +
    Changed Service Detection
        +
    Matrix Jobs
        +
    Reusable Workflows

Example:

    Commit
        |
        ↓
    Detect Changes
        |
        +--- User → Build
        +--- Cart → Skip
        +--- Orders → Build
        +--- Payment → Skip

---

## Question 32

    How would you avoid duplicating workflows across repositories?

### Expected Answer

Use reusable workflows.

    Repository A
        |
    Repository B
        |
    Repository C
        |
        ↓
    Reusable Workflow

Common logic can be centralized.

---

## Question 33

    How would you design DEV, QA, and PROD deployments?

### Expected Answer

    CI
        |
        ↓
    Immutable Artifact
        |
        ↓
    DEV
        |
        ↓
    QA
        |
        ↓
    Production Approval
        |
        ↓
    PROD

Each environment should have separate permissions and configuration.

---

## Question 34

    Should you rebuild the application for each environment?

### Expected Answer

No.

I prefer:

    Build Once
        |
        ↓
    Immutable Artifact
        |
        +--- DEV
        +--- QA
        +--- PROD

Only environment-specific configuration should change.

---

## Question 35

    How would you implement production approval?

### Expected Answer

Use a protected production environment.

    CI
        |
        ↓
    QA
        |
        ↓
    Production Environment
        |
        ↓
    Authorized Approval
        |
        ↓
    Production

---

# PART 5 - DOCKER AND ECR MOCK INTERVIEW

## Question 36

    How would Docker fit into your GitHub Actions pipeline?

### Expected Answer

    Source Code
        |
        ↓
    Build
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    ECR
        |
        ↓
    GitOps
        |
        ↓
    EKS

---

## Question 37

    What image tagging strategy would you use?

### Expected Answer

I prefer immutable identifiers such as:

    Commit SHA
        +
    Version
        +
    Image Digest

Production should not depend on a mutable:

    latest

tag.

---

## Question 38

    Why are image digests important?

### Expected Answer

A digest identifies specific image content.

Tags can change.

Digest:

    Image
        |
        ↓
    Specific Content

This improves deployment traceability and immutability.

---

## Question 39

    Trivy finds a critical vulnerability in your image.
    What do you do?

### Expected Answer

If critical vulnerabilities are blocked by policy:

    Trivy
        |
        ↓
    Critical Finding
        |
        X
    Pipeline Stops

Then:

    Identify Package
        |
        ↓
    Update
        |
        ↓
    Rebuild
        |
        ↓
    Rescan

---

## Question 40

    How would you secure Docker images?

### Expected Answer

Use:

    Trusted Base Image
        +
    Minimal Packages
        +
    Non-Root User
        +
    Vulnerability Scanning
        +
    No Secrets
        +
    Immutable Artifacts

---

# PART 6 - TERRAFORM AND INFRASTRUCTURE MOCK INTERVIEW

## Question 41

    How would GitHub Actions manage Terraform?

### Expected Answer

    Pull Request
        |
        ↓
    terraform fmt
        |
        ↓
    terraform validate
        |
        ↓
    Security Scan
        |
        ↓
    terraform plan
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    terraform apply

---

## Question 42

    Why should Terraform plan run during a pull request?

### Expected Answer

It allows reviewers to see:

    Resources Created
        +
    Resources Changed
        +
    Resources Destroyed

before the infrastructure is modified.

---

## Question 43

    Terraform plan shows a destructive production change.
    What do you do?

### Expected Answer

I would stop the deployment and investigate.

    Plan
        |
        ↓
    Destructive Change
        |
        X
    Production Apply

Then verify whether the change is:

    Intended
        +
    Safe
        +
    Approved

---

## Question 44

    How would you secure Terraform production apply?

### Expected Answer

Use:

    Protected Environment
        +
    Approval
        +
    OIDC
        +
    Least Privilege
        +
    State Protection
        +
    Plan Review

---

# PART 7 - KUBERNETES AND EKS MOCK INTERVIEW

## Question 45

    How would GitHub Actions deploy an application to EKS?

### Expected Answer

In a GitOps architecture:

    GitHub Actions
        |
        ↓
    Build Image
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

I would prefer this over giving CI direct production cluster access
when GitOps is the chosen architecture.

---

## Question 46

    Why use ArgoCD?

### Expected Answer

ArgoCD provides:

    GitOps
        +
    Continuous Reconciliation
        +
    Drift Detection
        +
    Declarative Deployment
        +
    Auditability
        +
    Easier Rollback

---

## Question 47

    What happens if someone manually changes a production
    Kubernetes resource?

### Expected Answer

ArgoCD can detect drift.

    Git
        |
        ↓
    Desired State

    Kubernetes
        |
        ↓
    Actual State

If they differ:

    Drift Detected
        |
        ↓
    Reconciliation

---

## Question 48

    How would you design zero-downtime deployment in EKS?

### Expected Answer

Use:

    Multiple Replicas
        +
    Readiness Probes
        +
    Rolling Deployment
        +
    Graceful Shutdown
        +
    Proper maxUnavailable
        +
    Proper maxSurge
        +
    ALB

---

## Question 49

    A deployment succeeds but users receive 503 errors.
    How would you troubleshoot it?

### Expected Answer

I would check:

    Pod Status
        |
        ↓
    Readiness
        |
        ↓
    Service
        |
        ↓
    Endpoints
        |
        ↓
    ALB Target Health
        |
        ↓
    Ingress
        |
        ↓
    Application Logs

Then verify:

    Port
        +
    Target Port
        +
    Health Check
        +
    Security Rules

---

# PART 8 - DEVSECOPS MOCK INTERVIEW

## Question 50

    Where would you place SonarQube?

### Expected Answer

After source/build validation and before artifact promotion.

    Checkout
        |
        ↓
    Build
        |
        ↓
    SonarQube
        |
        ↓
    Quality Gate
        |
        ↓
    Continue

---

## Question 51

    Where would you place Trivy?

### Expected Answer

Trivy can scan:

    Source Dependencies
        +
    Container Images
        +
    Other Supported Artifacts

For container security:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    ECR

---

## Question 52

    Where would Veracode fit?

### Expected Answer

Veracode can be integrated into the security stage of CI/CD.

Conceptually:

    Build
        |
        ↓
    Security
        |
        +--- SonarQube
        +--- Trivy
        +--- Veracode
        |
        ↓
    Security Gate

---

## Question 53

    What happens if a security gate fails?

### Expected Answer

If the policy requires blocking:

    Security Gate
        |
        ↓
    Failed
        |
        X
    Pipeline Stops

The issue should be remediated or go through an approved exception
process.

---

## Question 54

    What if the scanner is unavailable?

### Expected Answer

For a mandatory security control:

    Scanner Unavailable
        |
        X
    Deployment Blocked

If an emergency exception is allowed:

    Risk Assessment
        |
        ↓
    Authorized Approval
        |
        ↓
    Emergency Deployment
        |
        ↓
    Retrospective Scan

---

# PART 9 - SCENARIO-BASED MOCK INTERVIEW

## Question 55

    Your CI/CD pipeline takes 25 minutes. The team wants it under
    5 minutes. What would you do?

### Expected Answer

First measure the stages.

For example:

    Build      → 5 min
    Tests      → 10 min
    Security   → 5 min
    Packaging  → 5 min

Then:

    Parallelize
        +
    Cache
        +
    Test Splitting
        +
    Docker Cache
        +
    Path Filters
        +
    Optimize Slow Steps

I would not remove important security controls just to achieve
faster builds.

---

## Question 56

    A developer accidentally commits an AWS secret.

### Expected Answer

Treat it as compromised.

    Detect
        |
        ↓
    Revoke
        |
        ↓
    Rotate
        |
        ↓
    Investigate
        |
        ↓
    Remove Exposure
        |
        ↓
    Use OIDC

---

## Question 57

    Production deployment fails halfway through.

### Expected Answer

First determine the current state.

    Deployment Failure
        |
        ↓
    Capture Diagnostics
        |
        ↓
    Determine Partial State
        |
        ↓
    Assess Rollback
        |
        ↓
    Recover
        |
        ↓
    Validate

I would avoid blindly running the deployment again without
understanding the state.

---

## Question 58

    A GitHub Actions runner is compromised.

### Expected Answer

    Isolate Runner
        |
        ↓
    Revoke / Rotate Credentials
        |
        ↓
    Investigate Workflow Activity
        |
        ↓
    Destroy Runner
        |
        ↓
    Provision Clean Runner
        |
        ↓
    Validate

---

## Question 59

    A production image is deleted from ECR.

### Expected Answer

First determine whether running workloads are affected.

Then:

    Recover Artifact
        +
    Review Permissions
        +
    Check Lifecycle Policies
        +
    Preserve Known-Good Images

Production should retain sufficient artifacts for rollback and
recovery.

---

## Question 60

    A critical vulnerability is discovered in production.

### Expected Answer

    Detect
        |
        ↓
    Assess
        |
        ↓
    Contain
        |
        ↓
    Patch / Mitigate
        |
        ↓
    Test
        |
        ↓
    Deploy
        |
        ↓
    Monitor
        |
        ↓
    Review

---

# PART 10 - PRODUCTION INCIDENT MOCK INTERVIEW

## Question 61

    A deployment succeeded but application latency increased
    immediately afterward.

### Expected Answer

I would correlate the deployment with runtime metrics.

Check:

    Prometheus
        |
        ↓
    Grafana
        |
        ↓
    Latency
        +
    CPU
        +
    Memory
        +
    Request Rate

Then:

    ELK
        |
        ↓
    Application Logs

I would compare:

    Previous Version
        vs
    New Version

If the release is confirmed as the cause:

    Rollback
        |
        OR
    Fix Forward

---

## Question 62

    Users are receiving 503 errors after deployment.

### Expected Answer

Check:

    Pods
        |
        ↓
    Readiness
        |
        ↓
    Services
        |
        ↓
    Endpoints
        |
        ↓
    ALB
        |
        ↓
    Target Health
        |
        ↓
    Application Logs

Then determine whether the issue is:

    Application
        +
    Kubernetes
        +
    Service
        +
    Ingress / ALB
        +
    Network

---

## Question 63

    The deployment succeeded, but pods are restarting.

### Expected Answer

I would check:

    kubectl get pods
        |
        ↓
    kubectl describe pod
        |
        ↓
    kubectl logs
        |
        ↓
    kubectl logs --previous

Then investigate:

    OOMKilled
        +
    Application Crash
        +
    Configuration
        +
    Secret
        +
    Probe Failure

---

## Question 64

    Your GitHub Actions workflow is failing intermittently.

### Expected Answer

I would determine whether the failure is:

    Deterministic
        OR
    Transient

Check:

    Logs
        +
    External Dependencies
        +
    Network
        +
    Runner
        +
    Cache
        +
    Race Conditions

Then apply retries only to genuinely transient operations.

---

# PART 11 - ADVANCED MOCK INTERVIEW

## Question 65

    How would you design GitHub Actions for 500 repositories?

### Expected Answer

I would avoid 500 completely independent workflow designs.

Use:

    Reusable Workflows
        +
    Standard Templates
        +
    Organization Policies
        +
    Centralized Security
        +
    Runner Scaling
        +
    Workflow Versioning

---

## Question 66

    How would you manage different application types?

### Expected Answer

Create reusable workflow interfaces.

For example:

    Java
        |
        ↓
    Standard Java CI

    Node.js
        |
        ↓
    Standard Node CI

    Python
        |
        ↓
    Standard Python CI

Common controls remain centralized.

---

## Question 67

    How would you prevent a developer from modifying production
    deployment logic?

### Expected Answer

Use:

    Branch Protection
        +
    CODEOWNERS
        +
    Required Reviews
        +
    Protected Environments
        +
    Restricted Permissions

---

## Question 68

    How would you prevent DEV credentials from being used in PROD?

### Expected Answer

Use:

    Environment-Specific Secrets
        +
    Environment-Specific IAM Roles
        +
    OIDC Trust Conditions
        +
    Protected Environments

---

## Question 69

    How would you design GitHub Actions for multiple AWS accounts?

### Expected Answer

    GitHub Actions
        |
        +--- DEV OIDC Role → DEV Account
        |
        +--- QA OIDC Role → QA Account
        |
        +--- PROD OIDC Role → PROD Account

Each account has separate access boundaries.

---

## Question 70

    How would you deploy across multiple EKS clusters?

### Expected Answer

Use GitOps.

    GitOps
        |
        ↓
    ArgoCD
        |
        +--- EKS Cluster A
        +--- EKS Cluster B
        +--- EKS Cluster C

Each cluster has appropriate desired state.

---

# PART 12 - ARCHITECTURE MOCK INTERVIEW

## Question 71

    Design a complete production CI/CD architecture.

### Expected Answer

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    GitHub Actions
        |
        +--- Build
        +--- Test
        +--- SonarQube
        +--- Trivy
        +--- Veracode
        |
        ↓
    Docker Build
        |
        ↓
    ECR
        |
        ↓
    Immutable Image
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
    ALB
        |
        ↓
    Users

Monitoring:

    EKS
        |
        +--- Prometheus
        +--- Grafana
        +--- ELK

Infrastructure:

    Terraform
        |
        ↓
    AWS

Security:

    OIDC
        +
    IAM
        +
    RBAC
        +
    Security Gates
        +
    Protected Environments

---

## Question 72

    Explain the responsibility of each component.

### Expected Answer

### GitHub

Source control and workflow management.

### GitHub Actions

CI/CD automation.

### SonarQube

Code quality and static analysis.

### Trivy

Vulnerability scanning.

### Veracode

Application security testing.

### Docker

Container packaging.

### ECR

Container image registry.

### Terraform

Infrastructure provisioning.

### Helm

Kubernetes application packaging.

### ArgoCD

GitOps continuous delivery.

### EKS

Managed Kubernetes platform.

### Prometheus

Metrics collection.

### Grafana

Metrics visualization and dashboards.

### ELK

Centralized logging and log analysis.

---

# PART 13 - REAL-WORLD DEVOPS SCENARIOS

## Question 73

    Your team has 20 microservices. Every commit triggers all
    pipelines. How would you optimize it?

### Expected Answer

Use:

    Path-Based Detection
        +
    Independent Service Pipelines
        +
    Reusable Workflows
        +
    Matrix Jobs

Only changed services should build when architecture allows.

---

## Question 74

    A change in a shared library affects 20 services.
    How would you handle it?

### Expected Answer

    Shared Library
        |
        ↓
    Build
        |
        ↓
    Compatibility Tests
        |
        ↓
    Consumer Testing
        |
        ↓
    Version
        |
        ↓
    Controlled Rollout

---

## Question 75

    A production deployment is healthy, but users report a business
    feature is broken.

### Expected Answer

If the application is technically healthy but the feature is broken:

    Feature Flag
        |
        ↓
    Disable Feature

If no feature flag exists:

    Assess Rollback
        OR
    Fix Forward

---

## Question 76

    A release works in QA but fails in production.

### Expected Answer

Investigate environment differences:

    Configuration
        +
    Secrets
        +
    IAM
        +
    Network
        +
    Dependencies
        +
    Resource Limits

The goal is to avoid "works in QA" situations through better
environment parity.

---

# PART 14 - RAPID-FIRE QUESTIONS

## Question 77

    What is CI?

### Answer

Continuous Integration.

Frequently integrating and validating code changes automatically.

---

## Question 78

    What is CD?

### Answer

Continuous Delivery or Continuous Deployment depending on context.

It automates software delivery and potentially production
deployment.

---

## Question 79

    What is GitOps?

### Answer

Using Git as the source of truth for desired infrastructure and
application state.

---

## Question 80

    What is ArgoCD?

### Answer

A GitOps continuous delivery tool for Kubernetes.

---

## Question 81

    What is OIDC?

### Answer

An identity federation mechanism that allows GitHub Actions to
obtain temporary AWS credentials without storing long-lived access
keys.

---

## Question 82

    What is SAST?

### Answer

Static Application Security Testing.

It analyzes source code for security issues.

---

## Question 83

    What is SCA?

### Answer

Software Composition Analysis.

It identifies risks in third-party dependencies.

---

## Question 84

    What is DAST?

### Answer

Dynamic Application Security Testing.

It tests a running application.

---

## Question 85

    What is DevSecOps?

### Answer

Integrating security throughout development, CI/CD, deployment,
and runtime.

---

## Question 86

    What is immutable infrastructure?

### Answer

Infrastructure or artifacts are replaced rather than modified
in place.

---

## Question 87

    What is an immutable Docker image?

### Answer

A specific image identified by immutable content such as its digest.

---

## Question 88

    What is a security gate?

### Answer

A condition that must pass before the pipeline continues.

---

## Question 89

    What is least privilege?

### Answer

Giving only the permissions required to perform a task.

---

## Question 90

    What is separation of duties?

### Answer

Separating responsibilities so one person or identity does not
control the entire sensitive process.

---

# PART 15 - FINAL MOCK INTERVIEW

## Interviewer

    Tell me how you would design a secure CI/CD pipeline using
    GitHub Actions for a microservices application running on EKS.

### Candidate Answer

I would separate CI and CD responsibilities.

For CI, GitHub Actions would handle:

    Checkout
        +
    Build
        +
    Unit Tests
        +
    SonarQube
        +
    Security Scanning
        +
    Veracode
        +
    Docker Build
        +
    Trivy
        +
    ECR Push

I would use immutable image identifiers such as commit SHA and
image digest.

For AWS authentication, I would use GitHub OIDC with separate
IAM roles for DEV, QA, and PROD instead of long-lived access keys.

For CD, I would use GitOps.

    GitHub Actions
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

For production, I would use protected environments and required
approval.

For deployment strategy, I would use rolling deployments for
normal releases and canary or blue-green deployments for higher-risk
changes.

After deployment I would validate:

    Pod Health
        +
    Readiness
        +
    ALB Target Health
        +
    Application Health
        +
    Smoke Tests

For observability I would use:

    Prometheus
        +
    Grafana
        +
    ELK

If the deployment causes a production issue, I would use the
known-good immutable artifact and GitOps rollback process.

Security would be implemented through:

    Branch Protection
        +
    CODEOWNERS
        +
    OIDC
        +
    Least Privilege
        +
    SonarQube
        +
    Trivy
        +
    Veracode
        +
    Protected Environments
        +
    Artifact Traceability

The overall architecture would be:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    GitHub Actions
        |
        +--- Build
        +--- Test
        +--- Security
        |
        ↓
    Docker
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
    Validation
        |
        ↓
    Monitoring
        |
        ↓
    Rollback If Required

---

# PART 16 - MOCK INTERVIEW EVALUATION

After answering each question, evaluate yourself against these areas.

## 1. Technical Knowledge

Can you explain:

    GitHub Actions
        +
    YAML
        +
    Jobs
        +
    Steps
        +
    Runners
        +
    Secrets
        +
    Environments
        +
    Reusable Workflows

---

## 2. CI/CD Knowledge

Can you explain:

    Build
        +
    Test
        +
    Artifact
        +
    Registry
        +
    Deployment
        +
    Promotion
        +
    Rollback

---

## 3. AWS Knowledge

Can you explain:

    IAM
        +
    OIDC
        +
    ECR
        +
    EKS
        +
    ALB
        +
    VPC

---

## 4. Kubernetes Knowledge

Can you explain:

    Pods
        +
    Deployments
        +
    Services
        +
    Ingress
        +
    Readiness
        +
    Liveness
        +
    RBAC
        +
    Rolling Updates

---

## 5. Infrastructure Knowledge

Can you explain:

    Terraform
        +
    State
        +
    Plan
        +
    Apply
        +
    Modules
        +
    Remote Backend

---

## 6. DevSecOps Knowledge

Can you explain:

    SAST
        +
    SCA
        +
    DAST
        +
    Container Scanning
        +
    Secrets
        +
    IAM
        +
    Supply Chain
        +
    Least Privilege

---

## 7. Production Knowledge

Can you explain:

    High Availability
        +
    Zero Downtime
        +
    Canary
        +
    Blue-Green
        +
    Rolling Deployment
        +
    Rollback
        +
    Incident Response
        +
    Observability

---

# PART 17 - HOW TO ANSWER SCENARIO QUESTIONS

When the interviewer gives you a production scenario, do not jump
directly into commands.

Use this structure:

    1. Understand the Problem
            |
            ↓
    2. Assess Impact
            |
            ↓
    3. Check Evidence
            |
            ↓
    4. Identify Root Cause
            |
            ↓
    5. Contain
            |
            ↓
    6. Recover
            |
            ↓
    7. Validate
            |
            ↓
    8. Prevent Recurrence

Example:

    "The deployment succeeded but users receive 503."

Do not immediately say:

    "I will restart the pod."

Instead:

    Check deployment
        |
        ↓
    Check pod readiness
        |
        ↓
    Check service endpoints
        |
        ↓
    Check ALB target health
        |
        ↓
    Check application logs
        |
        ↓
    Identify root cause
        |
        ↓
    Fix / Rollback
        |
        ↓
    Validate

This demonstrates production troubleshooting maturity.

---

# PART 18 - STRONG INTERVIEW PHRASES

Use precise phrases such as:

    "I would first assess the blast radius."

    "I would avoid making changes before collecting evidence."

    "I would prefer immutable artifacts."

    "I would use OIDC instead of long-lived AWS credentials."

    "I would enforce least privilege."

    "I would separate CI from CD responsibilities."

    "I would use GitOps for Kubernetes deployment."

    "I would validate the deployment using health checks."

    "I would correlate the deployment with application metrics and
     logs."

    "I would use a protected production environment."

    "I would not blindly retry a deterministic failure."

    "I would distinguish rollback from fix-forward."

    "I would document and audit security exceptions."

    "I would minimize the blast radius of production changes."

    "I would test the rollback path instead of assuming it works."

---

# PART 19 - COMMON INTERVIEW MISTAKES

## Mistake 1

Only explaining GitHub Actions syntax.

### Better

Connect GitHub Actions to:

    CI/CD
        +
    AWS
        +
    Docker
        +
    Kubernetes
        +
    Security
        +
    Production

---

## Mistake 2

Giving only theoretical answers.

### Better

Explain:

    Architecture
        +
    Flow
        +
    Decision
        +
    Validation

---

## Mistake 3

Saying "I will restart the pod" immediately.

### Better

First investigate.

---

## Mistake 4

Using long-lived AWS access keys.

### Better

Use OIDC.

---

## Mistake 5

Deploying directly from CI without considering GitOps.

### Better

If GitOps is the architecture:

    CI
        |
        ↓
    GitOps
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

---

## Mistake 6

Using `latest` for production.

### Better

Use immutable image identifiers.

---

## Mistake 7

Ignoring rollback.

### Better

Always explain:

    How deployed
        +
    How validated
        +
    How recovered

---

## Mistake 8

Treating security as a final stage.

### Better

Integrate:

    Security
        |
        ↓
    Throughout CI/CD

---

## Mistake 9

Giving cluster-admin permissions.

### Better

Use least-privilege RBAC.

---

## Mistake 10

Ignoring monitoring after deployment.

### Better

Always mention:

    Prometheus
        +
    Grafana
        +
    ELK

---

# PART 20 - FINAL INTERVIEW CHECKLIST

Before a GitHub Actions interview, make sure you can explain:

    [ ] GitHub Actions architecture

    [ ] Workflow

    [ ] Jobs

    [ ] Steps

    [ ] Runners

    [ ] GitHub-hosted runners

    [ ] Self-hosted runners

    [ ] Workflow triggers

    [ ] workflow_dispatch

    [ ] workflow_call

    [ ] Reusable workflows

    [ ] Matrix strategy

    [ ] Caching

    [ ] Artifacts

    [ ] Secrets

    [ ] Variables

    [ ] Environments

    [ ] Environment protection

    [ ] Branch protection

    [ ] CODEOWNERS

    [ ] OIDC

    [ ] AWS IAM

    [ ] Docker

    [ ] ECR

    [ ] Terraform

    [ ] Helm

    [ ] Kubernetes

    [ ] EKS

    [ ] ArgoCD

    [ ] GitOps

    [ ] SonarQube

    [ ] Trivy

    [ ] Veracode

    [ ] DevSecOps

    [ ] SAST

    [ ] SCA

    [ ] DAST

    [ ] Secret management

    [ ] Least privilege

    [ ] Supply chain security

    [ ] Immutable artifacts

    [ ] Rolling deployment

    [ ] Canary deployment

    [ ] Blue-green deployment

    [ ] Zero-downtime deployment

    [ ] Health checks

    [ ] Rollback

    [ ] Prometheus

    [ ] Grafana

    [ ] ELK

    [ ] Production troubleshooting

    [ ] Incident response

    [ ] CI/CD architecture

    [ ] Security architecture

    [ ] Multi-environment deployment

    [ ] Multi-account AWS deployment

    [ ] Multi-cluster deployment

---

# PART 21 - FINAL MOCK INTERVIEW RULE

For every scenario question, remember:

    DO NOT START WITH A COMMAND.

Start with:

    Understand
        ↓
    Assess
        ↓
    Investigate
        ↓
    Identify Root Cause
        ↓
    Act
        ↓
    Validate
        ↓
    Monitor
        ↓
    Prevent Recurrence

For architecture questions:

    Requirements
        ↓
    Design
        ↓
    Security
        ↓
    Automation
        ↓
    Deployment
        ↓
    Observability
        ↓
    Rollback
        ↓
    Scalability

For DevSecOps questions:

    Identify Risk
        ↓
    Shift Left
        ↓
    Security Gate
        ↓
    Immutable Artifact
        ↓
    Secure Deployment
        ↓
    Runtime Security
        ↓
    Monitoring
        ↓
    Incident Response

---

# FINAL GITHUB ACTIONS INTERVIEW MINDSET

A strong GitHub Actions / DevOps interview answer should not sound
like:

    "I know GitHub Actions YAML."

It should sound like:

    "I can design, secure, operate, troubleshoot, and recover a
     production CI/CD platform using GitHub Actions."

The complete mindset is:

    Git
        |
        ↓
    CI
        |
        +--- Build
        +--- Test
        +--- Security
        |
        ↓
    Immutable Artifact
        |
        ↓
    Registry
        |
        ↓
    GitOps
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes / EKS
        |
        ↓
    Health Validation
        |
        ↓
    Prometheus
        +
    Grafana
        +
    ELK
        |
        ↓
    Production
        |
        ↓
    Rollback / Recovery

And across every stage:

    Security
        +
    Least Privilege
        +
    Traceability
        +
    Automation
        +
    Reliability
        +
    Observability

The objective of CI/CD is not simply:

    "Deploy the application."

The objective is:

    "Deliver changes quickly, securely, reliably, observably, and
     with a proven recovery path."