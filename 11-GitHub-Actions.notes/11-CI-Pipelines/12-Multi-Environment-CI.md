# Multi-Environment CI in GitHub Actions

Multi-environment CI is a CI/CD approach where an application is validated and promoted through multiple environments such as Development, QA, UAT, and Production.

The typical flow is:

    Developer
        |
        ↓
    GitHub
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
    Quality Gate
        |
        ↓
    Security Gate
        |
        ↓
    Artifact
        |
        ↓
    DEV
        |
        ↓
    QA
        |
        ↓
    UAT
        |
        ↓
    Production

The goal is to validate software progressively before it reaches production.

---

# Why Multiple Environments Are Used

Deploying directly from development to production creates significant risk.

Without multiple environments:

    Developer
        |
        ↓
      Code
        |
        ↓
      Build
        |
        ↓
    Production

With multiple environments:

    Developer
        |
        ↓
      Code
        |
        ↓
    CI Validation
        |
        ↓
      DEV
        |
        ↓
      QA
        |
        ↓
      UAT
        |
        ↓
    Production

Each environment provides another level of validation.

---

# Common Environments

A typical enterprise application may have:

    DEV
      |
      ↓
    QA
      |
      ↓
    SIT
      |
      ↓
    UAT
      |
      ↓
    Production

Not every organization uses all of these environments.

The exact environment structure depends on the organization's application, testing, release, security, and compliance requirements.

---

# DEV Environment

DEV is commonly used for development validation.

Typical activities:

- Developer testing
- Integration testing
- Feature verification
- Early application validation
- Deployment testing

Typical flow:

    Developer
        |
        ↓
    Feature Branch
        |
        ↓
    CI
        |
        ↓
    DEV

---

# QA Environment

QA is used for broader software testing.

Typical activities:

- Functional testing
- Integration testing
- Regression testing
- API testing
- Application validation
- Defect identification

Typical flow:

    DEV
      |
      ↓
     QA
      |
      ↓
    Testing

---

# SIT Environment

SIT means:

    System Integration Testing

SIT validates the interaction between multiple systems, applications, services, and external dependencies.

Example:

    Application
        |
        +-- Database
        +-- Payment Service
        +-- Notification Service
        +-- External API
        |
        ↓
       SIT

---

# UAT Environment

UAT means:

    User Acceptance Testing

UAT validates whether the application satisfies business requirements.

Typical participants may include:

- Business users
- Product owners
- Functional teams
- QA teams

Typical flow:

    QA
     |
     ↓
    UAT
     |
     ↓
    Business Validation

---

# Production Environment

Production is the live environment used by real users.

Production normally requires stronger controls than lower environments.

Example:

    UAT
      |
      ↓
    Approval
      |
      ↓
    Production

---

# Environment Progression

A common environment progression is:

    DEV
     |
     ↓
    QA
     |
     ↓
    UAT
     |
     ↓
    Production

Each stage should have clearly defined entry and exit criteria.

---

# Entry Criteria

Entry criteria define what must be true before software enters an environment.

Example QA entry criteria:

    Build Passed
    Unit Tests Passed
    Quality Gate Passed
    Security Gate Passed
    Artifact Available

Only after these conditions are satisfied should the artifact move to QA.

---

# Exit Criteria

Exit criteria define what must be completed before promotion to the next environment.

Example QA exit criteria:

    Functional Tests Passed
    Regression Tests Passed
    Critical Defects = 0
    Required Validation Completed

Then:

    QA
      |
      ↓
    UAT

---

# Environment Promotion

Promotion means moving a validated artifact from one environment to another.

Example:

    Build Artifact
        |
        ↓
       DEV
        |
        ↓
       QA
        |
        ↓
       UAT
        |
        ↓
    Production

The preferred approach is to promote the same validated artifact rather than rebuilding it for every environment.

---

# Build Once, Promote Many

A strong CI/CD principle is:

    Build Once
        |
        ↓
    Test
        |
        ↓
    Validate
        |
        ↓
    Create Artifact
        |
        ↓
    Promote Same Artifact
        |
        +---- DEV
        |
        +---- QA
        |
        +---- UAT
        |
        +---- PROD

This improves consistency and traceability.

---

# Why Rebuilding for Every Environment Is Risky

Bad approach:

    Build for DEV
        |
        ↓
    DEV

    Build Again for QA
        |
        ↓
    QA

    Build Again for PROD
        |
        ↓
    PROD

The artifacts may differ.

Better approach:

    Build
      |
      ↓
    Artifact
      |
      +---- DEV
      |
      +---- QA
      |
      +---- UAT
      |
      +---- PROD

---

# Artifact Immutability

An immutable artifact should not be modified after creation.

Example:

    myapp:1.4.7

The same artifact can be promoted:

    myapp:1.4.7
         |
         +-- DEV
         |
         +-- QA
         |
         +-- UAT
         |
         +-- PROD

Only environment-specific configuration should change.

---

# Environment-Specific Configuration

Applications often require different configuration values for different environments.

Example:

    DEV:
        database = dev-db

    QA:
        database = qa-db

    UAT:
        database = uat-db

    PROD:
        database = prod-db

The application artifact can remain the same while environment-specific configuration changes.

---

# Never Hardcode Environment Values

Bad approach:

    Application Image
        |
        +-- DEV Database
        +-- QA Database
        +-- PROD Database

Better approach:

    Same Application Image
             |
             ↓
    Environment Configuration
             |
             ↓
       Deployment

Environment-specific configuration should be injected during deployment when appropriate.

---

# Environment Variables

GitHub Actions supports environment variables.

Example concept:

    env:
      APP_ENV: production

Then a step can use:

    echo "$APP_ENV"

Environment variables can be defined at workflow, job, or step level.

---

# Workflow-Level Environment Variables

Example:

    env:
      APP_NAME: myapp

    jobs:

      build:
        runs-on: ubuntu-latest

        steps:

          - run: echo "$APP_NAME"

Workflow-level variables can be available to jobs and steps according to their scope.

---

# Job-Level Environment Variables

Example:

    jobs:

      deploy:
        runs-on: ubuntu-latest

        env:
          ENVIRONMENT: qa

        steps:

          - run: echo "$ENVIRONMENT"

The variable is available to the steps in that job.

---

# Step-Level Environment Variables

Example:

    steps:

      - name: Deploy
        env:
          ENVIRONMENT: qa
        run: |
          ./deploy.sh

The variable is limited to that step.

---

# GitHub Environments

GitHub Actions provides environments that can be used for deployment targets.

Examples:

    development
    qa
    uat
    production

A GitHub Environment can provide:

- Environment-specific secrets
- Environment-specific variables
- Deployment protection rules
- Required reviewers
- Deployment history

---

# Environment Example

Conceptually:

    jobs:

      deploy:

        environment: production

        runs-on: ubuntu-latest

        steps:

          - name: Deploy
            run: ./deploy.sh

The job targets the production environment.

---

# Why GitHub Environments Matter

GitHub Environments provide a control boundary around deployments.

Example:

    Production Environment
        |
        +-- Production Secrets
        +-- Production Variables
        +-- Required Reviewers
        +-- Deployment Controls
        |
        ↓
    Production Job

This is useful for protecting sensitive environments.

---

# Environment Secrets

Different environments may need different secrets.

Example:

    DEV_SECRET
    QA_SECRET
    UAT_SECRET
    PROD_SECRET

A production secret should not be unnecessarily exposed to development jobs.

---

# Secret Separation

Good design:

    DEV
      |
      +-- DEV credentials

    QA
      |
      +-- QA credentials

    UAT
      |
      +-- UAT credentials

    PROD
      |
      +-- PROD credentials

This follows the principle of environment isolation.

---

# Environment Variables vs Secrets

Environment variables are generally used for non-sensitive configuration.

Examples:

    API_URL
    REGION
    LOG_LEVEL

Secrets are used for sensitive values.

Examples:

    Password
    Token
    Private Key
    API Credential

Sensitive credentials should not be stored as ordinary plain-text variables.

---

# Environment Isolation

A mature environment design separates:

    Configuration
    Secrets
    Infrastructure
    Access
    Permissions
    Deployment Controls

Example:

    DEV
      |
      +-- DEV Config
      +-- DEV Secrets
      +-- DEV IAM
      +-- DEV Infrastructure

    PROD
      |
      +-- PROD Config
      +-- PROD Secrets
      +-- PROD IAM
      +-- PROD Infrastructure

---

# AWS Multi-Environment Architecture

Enterprise organizations may isolate environments using different AWS accounts.

Example:

    DEV Account
        |
        ↓
    QA Account
        |
        ↓
    UAT Account
        |
        ↓
    PROD Account

This provides stronger isolation between environments.

---

# Environment-Specific AWS Resources

Example:

    DEV:
        EKS Dev
        RDS Dev
        ECR Dev

    QA:
        EKS QA
        RDS QA
        ECR QA

    UAT:
        EKS UAT
        RDS UAT
        ECR UAT

    PROD:
        EKS Prod
        RDS Prod
        ECR Prod

Each environment can have independent infrastructure.

---

# AWS Authentication

GitHub Actions should avoid long-lived AWS access keys when possible.

A modern approach is:

    GitHub Actions
          |
          ↓
        OIDC
          |
          ↓
    AWS IAM Role
          |
          ↓
    AWS Resources

This uses short-lived credentials.

---

# OIDC Environment Isolation

Different environments can assume different IAM roles.

Example:

    DEV Job
      |
      ↓
    DEV IAM Role

    QA Job
      |
      ↓
    QA IAM Role

    UAT Job
      |
      ↓
    UAT IAM Role

    PROD Job
      |
      ↓
    PROD IAM Role

The production role should have only the permissions required for production operations.

---

# Least Privilege

Each environment should receive only the permissions required for its operations.

Example:

    DEV Role
       |
       +-- DEV Resources

    QA Role
       |
       +-- QA Resources

    PROD Role
       |
       +-- PROD Resources

Avoid giving every CI job unrestricted access to every environment.

---

# Multi-Environment Branching

One possible strategy is:

    feature/*
        |
        ↓
    develop
        |
        ↓
    DEV
        |
        ↓
    release/*
        |
        ↓
    QA
        |
        ↓
    UAT
        |
        ↓
    main
        |
        ↓
    PROD

The exact branch strategy depends on the team's development model.

---

# Branch-Based Deployment

Example:

    develop
       |
       ↓
      DEV

    release/*
       |
       ↓
      QA

    main
       |
       ↓
    Production

This is one possible implementation.

---

# Branch-Based Environment Selection

Conceptually:

    if branch == develop:
        environment = dev

    if branch == release:
        environment = qa

    if branch == main:
        environment = production

However, branch-based production deployment should be carefully designed to prevent accidental deployments.

---

# Tag-Based Production Deployment

Another strategy is:

    Feature Branch
        |
        ↓
    develop
        |
        ↓
       QA
        |
        ↓
       UAT
        |
        ↓
    Git Tag
        |
        ↓
    Production

Example:

    v1.4.7

A production release can be triggered from a release tag.

---

# Manual Production Deployment

Production deployment can be triggered manually.

Example:

    GitHub Actions
        |
        ↓
    Manual Trigger
        |
        ↓
    Production Deployment
        |
        ↓
    Approval
        |
        ↓
    Deploy

This provides an explicit human control point.

---

# workflow_dispatch

`workflow_dispatch` allows a GitHub Actions workflow to be manually triggered.

Conceptually:

    on:
      workflow_dispatch:

A manual workflow can also accept inputs.

Example:

    Environment:
      dev
      qa
      uat
      production

---

# Environment Input

Conceptually:

    on:
      workflow_dispatch:
        inputs:
          environment:
            description: Deployment environment
            required: true
            type: choice
            options:
              - dev
              - qa
              - uat
              - production

The workflow can use the selected environment after applying appropriate controls.

---

# Be Careful With Manual Environment Inputs

A dangerous design is:

    User selects:
        production

        |
        ↓

    Immediate Production Deployment

Better:

    Input
      |
      ↓
    Validation
      |
      ↓
    Production Environment
      |
      ↓
    Required Approval
      |
      ↓
    Deploy

---

# GitHub Environment Protection

Production can require reviewers.

Flow:

    Workflow
       |
       ↓
    production environment
       |
       ↓
    Required Reviewer
       |
       ↓
    Approved
       |
       ↓
    Deployment

This provides a human control point before production.

---

# Environment Protection and Security

Production protection can prevent accidental deployment.

Example:

    Developer
       |
       ↓
    Workflow
       |
       ↓
    Production Environment
       |
       ↓
    Approval
       |
       ↓
    Deploy

The production deployment should not bypass these controls.

---

# Multi-Environment CI Pipeline

Typical pipeline:

    Checkout
       |
       ↓
    Dependency Cache
       |
       ↓
    Build
       |
       ↓
    Unit Tests
       |
       ↓
    SonarQube
       |
       ↓
    Quality Gate
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    Security Gate
       |
       ↓
    Publish Artifact
       |
       ↓
      DEV
       |
       ↓
      QA
       |
       ↓
      UAT
       |
       ↓
    Production

---

# CI vs CD

CI focuses on:

    Build
    Test
    Analyze
    Validate

CD focuses on:

    Deploy
    Promote
    Release

Multi-environment workflows often connect CI validation with CD promotion.

---

# Multi-Environment Pipeline Architecture

    GitHub
       |
       ↓
    GitHub Actions
       |
       ↓
    CI
       |
       +-- Build
       +-- Test
       +-- SonarQube
       +-- Security
       |
       ↓
    Artifact
       |
       ↓
    CD
       |
       +-- DEV
       +-- QA
       +-- UAT
       +-- PROD

---

# Separate CI and CD Workflows

A mature setup may separate CI and CD.

CI:

    Code
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
    Artifact

CD:

    Artifact
      |
      ↓
    DEV
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    PROD

This separation improves control and reusability.

---

# CI Workflow

A CI workflow may perform:

    Checkout
      |
      ↓
    Dependency Installation
      |
      ↓
    Build
      |
      ↓
    Unit Tests
      |
      ↓
    Code Analysis
      |
      ↓
    Security Scans
      |
      ↓
    Artifact Publishing

The CI workflow creates a validated artifact.

---

# CD Workflow

A CD workflow may perform:

    Artifact
      |
      ↓
    Deploy DEV
      |
      ↓
    Validate
      |
      ↓
    Deploy QA
      |
      ↓
    Validate
      |
      ↓
    Deploy UAT
      |
      ↓
    Approval
      |
      ↓
    Deploy PROD

---

# Artifact-Based CD

Preferred model:

    CI
      |
      ↓
    Build Artifact
      |
      ↓
    Registry
      |
      ↓
    CD
      |
      +-- DEV
      +-- QA
      +-- UAT
      +-- PROD

The CD pipeline consumes the existing artifact.

---

# Container Artifact

For containerized applications:

    Source
      |
      ↓
    Docker Build
      |
      ↓
    Image
      |
      ↓
    Trivy
      |
      ↓
    Security Gate
      |
      ↓
    ECR
      |
      ↓
    Deploy

Example image:

    myapp:1.4.7

---

# Image Tag Strategy

Possible image identifiers include:

    latest
    commit SHA
    release version

For environment promotion, immutable identifiers are preferred.

Example:

    myapp:1.4.7

or:

    myapp:<commit-sha>

---

# Why Avoid latest

Using:

    myapp:latest

can make it difficult to determine exactly which version is deployed.

Better:

    myapp:1.4.7

or:

    myapp:<commit-sha>

This improves traceability.

---

# Multi-Environment ECR Flow

    GitHub Actions
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        ↓
    ECR
        |
        ↓
      DEV EKS
        |
        ↓
      QA EKS
        |
        ↓
      UAT EKS
        |
        ↓
     PROD EKS

---

# Multi-Environment Kubernetes

Example:

    DEV Cluster
        |
        +-- Namespace: dev

    QA Cluster
        |
        +-- Namespace: qa

    UAT Cluster
        |
        +-- Namespace: uat

    PROD Cluster
        |
        +-- Namespace: prod

Organizations can choose separate clusters or namespaces depending on isolation requirements.

---

# Namespace-Based Environments

One cluster may contain:

    EKS
     |
     +-- namespace-dev
     +-- namespace-qa
     +-- namespace-uat
     +-- namespace-prod

This provides logical separation but is not equivalent to separate AWS accounts or clusters.

---

# Cluster-Based Environments

Another design is:

    DEV EKS
       |
       ↓
    QA EKS
       |
       ↓
    UAT EKS
       |
       ↓
    PROD EKS

This can provide stronger infrastructure isolation.

---

# Environment Configuration in Kubernetes

Example:

    DEV
      |
      +-- ConfigMap
      +-- Secret
      +-- Ingress
      +-- Resource Limits

    PROD
      |
      +-- ConfigMap
      +-- Secret
      +-- Ingress
      +-- Resource Limits

The application image can remain unchanged.

---

# Helm and Multi-Environment Deployment

Helm can use environment-specific values.

Example:

    helm/
      |
      ├── values.yaml
      ├── values-dev.yaml
      ├── values-qa.yaml
      ├── values-uat.yaml
      └── values-prod.yaml

Deployment:

    DEV:
      values-dev.yaml

    QA:
      values-qa.yaml

    UAT:
      values-uat.yaml

    PROD:
      values-prod.yaml

---

# Helm Environment Flow

    Same Image
        |
        ↓
    Helm Chart
        |
        +-- DEV Values
        +-- QA Values
        +-- UAT Values
        +-- PROD Values
        |
        ↓
    Kubernetes

This supports the build-once-promote-many model.

---

# ArgoCD Multi-Environment

A GitOps repository can contain:

    environments/
      |
      ├── dev/
      ├── qa/
      ├── uat/
      └── prod/

Each environment can reference the appropriate application configuration.

ArgoCD synchronizes the desired state into the appropriate Kubernetes environment.

---

# GitOps Environment Flow

    CI
      |
      ↓
    Build Image
      |
      ↓
    Security Gate
      |
      ↓
    Update GitOps
      |
      ↓
    dev/
      |
      ↓
    ArgoCD
      |
      ↓
    DEV

Then after validation:

    qa/
      |
      ↓
    ArgoCD
      |
      ↓
    QA

The same approach can continue through UAT and Production.

---

# Promotion Through GitOps

Example:

    DEV
      |
      ↓
    GitOps Dev Configuration
      |
      ↓
    Validation
      |
      ↓
    Promote Image Version
      |
      ↓
    GitOps QA Configuration
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    Production

---

# GitOps and Environment Isolation

Each environment can have:

    Separate Configuration
    Separate Secrets
    Separate Cluster
    Separate Namespace
    Separate Approval

The exact isolation level depends on organizational requirements.

---

# Terraform and Multi-Environment Infrastructure

Terraform can manage environment-specific infrastructure.

Example:

    terraform/
      |
      ├── dev/
      ├── qa/
      ├── uat/
      └── prod/

Another approach is:

    modules/
      |
      ├── vpc
      ├── eks
      ├── rds
      └── iam

with separate environment configurations.

---

# Terraform Workspace Approach

Terraform workspaces can represent environments.

Example:

    dev
    qa
    uat
    prod

However, workspaces are not automatically the best solution for every environment architecture.

For stronger environment isolation, separate state and clearly separated configuration may be preferable.

---

# Terraform Environment State

Each environment should have controlled state.

Example:

    DEV State
       |
       ↓
    DEV Infrastructure

    QA State
       |
       ↓
    QA Infrastructure

    PROD State
       |
       ↓
    PROD Infrastructure

Avoid accidentally using production state for development operations.

---

# Multi-Environment Infrastructure Pipeline

Example:

    Terraform Code
        |
        ↓
    Validate
        |
        ↓
    Plan DEV
        |
        ↓
    Apply DEV
        |
        ↓
    Test
        |
        ↓
    Plan QA
        |
        ↓
    Apply QA
        |
        ↓
    UAT
        |
        ↓
    Production Approval
        |
        ↓
    Plan PROD
        |
        ↓
    Apply PROD

---

# Terraform Plan as a Gate

Before applying infrastructure changes:

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

For production, plan and apply should be carefully controlled.

---

# Environment-Specific Terraform Variables

Example:

    dev.tfvars
    qa.tfvars
    uat.tfvars
    prod.tfvars

Values may differ for:

    Instance Size
    Replica Count
    Region
    Network
    Database Configuration

Sensitive values should not be committed in plain text.

---

# Environment Promotion Rules

Example:

    DEV:
      Automatic

    QA:
      Automatic after DEV success

    UAT:
      Approval required

    PROD:
      Approval required

The exact policy depends on organizational requirements.

---

# Automatic DEV Deployment

Typical flow:

    Push
      |
      ↓
    CI
      |
      ↓
    Quality Gate
      |
      ↓
    Security Gate
      |
      ↓
    Deploy DEV

Fast feedback is useful in development.

---

# QA Deployment

Example:

    DEV
      |
      ↓
    Automated Tests
      |
      ↓
    Deploy QA

QA can then perform broader testing.

---

# UAT Deployment

Example:

    QA
      |
      ↓
    Regression Passed
      |
      ↓
    UAT
      |
      ↓
    Business Validation

---

# Production Deployment

Example:

    UAT
      |
      ↓
    Approval
      |
      ↓
    Production
      |
      ↓
    Validation

Production should have the strongest controls.

---

# Deployment Windows

Some organizations restrict production deployments to approved time windows.

Example:

    Release Approved
        |
        ↓
    Deployment Window
        |
        ↓
    Production Deployment

This is common in enterprise environments with formal change management.

---

# JIRA Change Request

An enterprise process may require:

    Code
      |
      ↓
    CI
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    JIRA Change Request
      |
      ↓
    Approval
      |
      ↓
    Production

The exact process depends on the organization.

---

# Multi-Environment Approval Flow

    DEV
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    Business Approval
      |
      ↓
    Change Approval
      |
      ↓
    Production

This provides controlled promotion.

---

# Separation of Duties

In regulated environments, the person who develops the change may not be the same person who approves production deployment.

Example:

    Developer
        |
        ↓
    Build
        |
        ↓
    QA
        |
        ↓
    UAT
        |
        ↓
    Approver
        |
        ↓
    Production

This is known as separation of duties.

---

# Multi-Environment Security

Each environment should have:

- Appropriate IAM permissions
- Separate secrets
- Controlled deployments
- Environment-specific access
- Audit logging
- Approval controls
- Least privilege

Production should receive the strongest protections.

---

# Production Credential Isolation

Never use the same production credentials for every environment.

Bad:

    DEV → PROD Credentials
    QA  → PROD Credentials
    UAT → PROD Credentials

Better:

    DEV → DEV Credentials
    QA  → QA Credentials
    UAT → UAT Credentials
    PROD → PROD Credentials

---

# Multi-Environment Secret Flow

    GitHub Actions
        |
        +-- DEV Job
        |     ↓
        |   DEV Secrets
        |
        +-- QA Job
        |     ↓
        |   QA Secrets
        |
        +-- UAT Job
        |     ↓
        |   UAT Secrets
        |
        +-- PROD Job
              ↓
            PROD Secrets

---

# Environment Deployment Status

GitHub Actions environments can provide deployment visibility.

Example:

    Workflow
       |
       ↓
    DEV Deployment
       |
       ↓
    QA Deployment
       |
       ↓
    UAT Deployment
       |
       ↓
    Production Deployment

This makes deployment history easier to track.

---

# Environment Failure

If deployment to QA fails:

    DEV
      |
      ↓
    QA
      |
      X
    FAIL

Do not automatically promote to UAT or production.

Correct approach:

    QA Failure
        |
        ↓
    Stop Promotion
        |
        ↓
    Investigate
        |
        ↓
    Fix
        |
        ↓
    Re-run
        |
        ↓
    Continue

---

# Environment Health Checks

After deployment:

    Deploy
      |
      ↓
    Health Check
      |
      +-- PASS → Continue
      |
      +-- FAIL → Stop / Rollback

Health checks can include:

    HTTP Health Endpoint
    Kubernetes Readiness
    Application Startup
    Database Connectivity
    Smoke Tests

---

# Smoke Testing

Smoke tests validate basic application functionality after deployment.

Example:

    Deploy QA
       |
       ↓
    GET /health
       |
       ↓
    GET /api/status
       |
       ↓
    Basic API Test
       |
       ↓
    PASS / FAIL

---

# Environment Promotion With Validation

Complete flow:

    Build
      |
      ↓
    Quality Gate
      |
      ↓
    Security Gate
      |
      ↓
    DEV
      |
      ↓
    Smoke Test
      |
      ↓
    QA
      |
      ↓
    Regression Test
      |
      ↓
    UAT
      |
      ↓
    Business Approval
      |
      ↓
    Production
      |
      ↓
    Health Check

---

# Rollback in Multi-Environment CI/CD

If production deployment fails:

    Production
        |
        ↓
    Health Check
        |
        ↓
      FAIL
        |
        ↓
    Rollback
        |
        ↓
    Previous Version

Example:

    v1.4.6
        |
        ↓
    Production

    v1.4.7
        |
        ↓
    Deployment
        |
        ↓
      Failure
        |
        ↓
    Rollback
        |
        ↓
    v1.4.6

---

# Rollback and Immutable Artifacts

If artifacts are immutable:

    v1.4.6
    v1.4.7

Rollback simply means:

    Production
       |
       ↓
    v1.4.6

No rebuild is required.

This improves rollback reliability.

---

# Multi-Environment CI With Docker and EKS

For a Kubernetes application:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +-- Maven / npm / pip
        +-- Unit Tests
        +-- SonarQube
        +-- Trivy
        |
        ↓
    ECR
        |
        ↓
    DEV EKS
        |
        ↓
    QA EKS
        |
        ↓
    UAT EKS
        |
        ↓
    PROD EKS

---

# Multi-Environment CI With ArgoCD

GitHub Actions:

    Build
      |
      ↓
    Test
      |
      ↓
    Security
      |
      ↓
    Push Image
      |
      ↓
    Update GitOps Repository

ArgoCD:

    GitOps Repository
          |
          ↓
        DEV
          |
          ↓
        QA
          |
          ↓
        UAT
          |
          ↓
       Production

---

# GitHub Actions + ArgoCD

A common separation of responsibility is:

GitHub Actions:

    Build
    Test
    Scan
    Publish
    Update GitOps

ArgoCD:

    Detect Desired State
    Sync
    Reconcile
    Monitor Drift
    Deploy to Kubernetes

This keeps CI and GitOps deployment responsibilities clear.

---

# Multi-Environment CI With Helm

Example:

    CI
     |
     ↓
    Build Image
     |
     ↓
    ECR
     |
     ↓
    Helm
     |
     +-- values-dev.yaml
     +-- values-qa.yaml
     +-- values-uat.yaml
     +-- values-prod.yaml
     |
     ↓
    Kubernetes

---

# Multi-Environment CI With Terraform

Terraform:

    Infrastructure Code
          |
          ↓
       Modules
          |
          +-- DEV
          +-- QA
          +-- UAT
          +-- PROD
          |
          ↓
    Environment Infrastructure

Application deployment can then happen on top of the provisioned infrastructure.

---

# Multi-Environment Pipeline Failure Handling

Example:

    DEV → PASS
      |
      ↓
    QA → FAIL
      |
      ↓
    Stop
      |
      ↓
    Investigate
      |
      ↓
    Fix
      |
      ↓
    Re-run QA
      |
      ↓
    Continue

Never promote a failed artifact automatically.

---

# Multi-Environment Pipeline Observability

Track:

    Deployment Status
    Deployment Duration
    Failure Rate
    Rollback Rate
    Environment Health
    Test Results

Example:

    DEV → PASS
    QA  → PASS
    UAT → PASS
    PROD → FAIL

This provides visibility into the release lifecycle.

---

# Deployment Audit Trail

A mature pipeline should answer:

    Who triggered deployment?
        |
        ↓
    Which commit?
        |
        ↓
    Which artifact?
        |
        ↓
    Which environment?
        |
        ↓
    Which approval?
        |
        ↓
    When?
        |
        ↓
    Result?

This is important for enterprise operations and compliance.

---

# Environment Promotion Metadata

Example:

    Application:
        myapp

    Version:
        1.4.7

    Commit:
        abc123

    Environment:
        QA

    Deployment:
        Successful

This helps trace exactly what was deployed.

---

# Multi-Environment Naming

Use clear environment names.

Example:

    dev
    qa
    uat
    prod

Avoid ambiguous names.

Consistency is important across:

    GitHub
    AWS
    Kubernetes
    Terraform
    Helm
    ArgoCD
    Monitoring

---

# Environment Naming Example

AWS:

    dev-account
    qa-account
    uat-account
    prod-account

Kubernetes:

    dev
    qa
    uat
    prod

GitHub:

    development
    qa
    uat
    production

Consistency reduces operational mistakes.

---

# Multi-Environment CI Best Practices

- Build once
- Promote the same artifact
- Use immutable artifact versions
- Separate environment configuration
- Separate environment secrets
- Use least privilege
- Protect production
- Use required approvals where needed
- Use branch protection
- Use quality gates
- Use security gates
- Validate deployments
- Perform health checks
- Use rollback strategies
- Keep audit trails
- Avoid production credentials in lower environments
- Prefer short-lived credentials
- Use OIDC for AWS where appropriate
- Keep CI and CD responsibilities clear
- Use GitOps for controlled Kubernetes deployments
- Monitor every environment

---

# Multi-Environment CI Anti-Patterns

## Anti-Pattern 1: Rebuild for Every Environment

Bad:

    Build DEV
      |
      ↓
    Build QA
      |
      ↓
    Build PROD

Better:

    Build Once
       |
       ↓
    Artifact
       |
       +-- DEV
       +-- QA
       +-- UAT
       +-- PROD

---

# Anti-Pattern 2: Shared Production Credentials

Bad:

    DEV → PROD Credentials

Better:

    DEV → DEV Credentials
    QA  → QA Credentials
    PROD → PROD Credentials

---

# Anti-Pattern 3: No Production Approval

Bad:

    main
      |
      ↓
    Automatic Production

For high-risk environments, use appropriate protection and approval mechanisms.

---

# Anti-Pattern 4: Environment Configuration Inside Application Image

Bad:

    Docker Image
       |
       +-- DEV Database
       +-- QA Database
       +-- PROD Database

Better:

    Same Image
       |
       +-- DEV Configuration
       +-- QA Configuration
       +-- PROD Configuration

---

# Anti-Pattern 5: Using latest

Bad:

    myapp:latest

Better:

    myapp:1.4.7

or:

    myapp:<commit-sha>

Immutable versioning improves traceability.

---

# Anti-Pattern 6: Deploying After a Failed Test

Bad:

    QA Test
       |
       ↓
      FAIL
       |
       ↓
    UAT Deployment

Better:

    QA Test
       |
       ↓
      FAIL
       |
       X
    Stop Promotion

---

# Anti-Pattern 7: No Environment Isolation

Bad:

    Every Job
       |
       ↓
    Full Production Access

Better:

    DEV → DEV Permissions
    QA → QA Permissions
    UAT → UAT Permissions
    PROD → PROD Permissions

---

# Anti-Pattern 8: Manual Changes to Production

Bad:

    CI/CD
      |
      ↓
    Production
      |
      ↓
    Engineer Manually Changes Resources

Better:

    Git
      |
      ↓
    CI/CD
      |
      ↓
    Controlled Deployment

Infrastructure and configuration should be managed through controlled processes.

---

# Interview Questions

## Basic

1. What is a multi-environment pipeline?
2. Why do we need multiple environments?
3. What is DEV?
4. What is QA?
5. What is SIT?
6. What is UAT?
7. What is Production?
8. What is environment promotion?
9. What is an artifact?
10. What does build once, promote many mean?

---

# Intermediate Interview Questions

11. How do you design a multi-environment GitHub Actions pipeline?

12. How do you manage environment-specific variables?

13. How do you manage environment-specific secrets?

14. What are GitHub Actions environments?

15. How do you protect a production environment?

16. How do you implement manual production approval?

17. How do you use workflow_dispatch?

18. How do you promote the same Docker image across environments?

19. How do you handle different Kubernetes environments?

20. How do you use Helm for multiple environments?

21. How do you use Terraform for multiple environments?

22. How do you use ArgoCD for multiple environments?

23. How do you separate AWS credentials between environments?

24. How do you use OIDC with multiple AWS environments?

25. How do you handle environment-specific configuration?

---

# Advanced Interview Questions

26. Design an enterprise multi-environment CI/CD pipeline using GitHub Actions.

27. How would you implement DEV → QA → UAT → PROD promotion?

28. How would you prevent accidental production deployment?

29. How would you design environment isolation in AWS?

30. How would you design multi-environment EKS deployments?

31. How would you implement multi-environment GitOps with ArgoCD?

32. How would you manage Helm values across environments?

33. How would you manage Terraform infrastructure across environments?

34. How would you implement artifact promotion?

35. How would you design rollback across multiple environments?

36. How would you implement separation of duties?

37. How would you manage secrets securely across environments?

38. How would you handle a failed QA deployment?

39. How would you ensure the production artifact is exactly the one tested in QA?

40. How would you optimize a multi-environment pipeline?

---

# Scenario Question

## Design a DEV → QA → UAT → PROD Pipeline

A strong architecture would be:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Unit Tests
        +-- SonarQube
        +-- Security Scans
        |
        ↓
    Quality / Security Gates
        |
        ↓
    Immutable Artifact
        |
        ↓
       DEV
        |
        ↓
    Smoke Tests
        |
        ↓
       QA
        |
        ↓
    Functional / Regression Tests
        |
        ↓
       UAT
        |
        ↓
    Business Approval
        |
        ↓
    Production Approval
        |
        ↓
      PROD
        |
        ↓
    Health Checks
        |
        ↓
    Monitoring / Validation

---

# Scenario Question

## How do you guarantee the same artifact reaches production?

I would use an immutable artifact identifier.

For example:

    myapp:1.4.7

The flow would be:

    Build
      |
      ↓
    myapp:1.4.7
      |
      ↓
    DEV
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    PROD

I would not rebuild the application separately for production.

---

# Scenario Question

## How would you prevent developers from deploying directly to production?

I would use multiple controls:

    1. Protected production environment
    2. Required reviewers
    3. Branch protection
    4. Required CI checks
    5. Separate production IAM role
    6. Least-privilege permissions
    7. Controlled deployment workflow
    8. Audit trail

The production deployment should require the appropriate validation and authorization.

---

# Scenario Question

## How would you manage different database URLs for DEV, QA, and PROD?

I would keep the application artifact the same and provide environment-specific configuration.

Example:

    DEV → DEV DB URL
    QA  → QA DB URL
    UAT → UAT DB URL
    PROD → PROD DB URL

Sensitive credentials would be stored in appropriate secret management systems rather than hardcoded into the image.

---

# Scenario Question

## How would you deploy the same Docker image to different EKS environments?

I would build and scan the image once:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        ↓
    ECR
        |
        ↓
    myapp:1.4.7

Then promote the same image:

    myapp:1.4.7
        |
        +-- DEV EKS
        +-- QA EKS
        +-- UAT EKS
        +-- PROD EKS

Environment-specific Helm values or GitOps configuration would control the deployment settings.

---

# Scenario Question

## How would you implement multi-environment deployments with ArgoCD?

I would keep environment-specific desired state in Git.

Example:

    gitops/
      |
      ├── dev/
      ├── qa/
      ├── uat/
      └── prod/

ArgoCD applications would point to the appropriate environment configuration.

GitHub Actions would build and validate the image, then update the appropriate GitOps configuration as part of the promotion process.

---

# Scenario Question

## QA deployment failed. Should UAT continue?

No.

The normal flow should be:

    QA
      |
      ↓
    Deployment
      |
      ↓
      FAIL
      |
      X
    Stop Promotion

The failure should be investigated and fixed before promotion continues.

---

# Scenario Question

## Production deployment failed after the image was promoted successfully. What would you do?

I would:

    1. Check deployment status
    2. Check application health
    3. Check Kubernetes events
    4. Check logs
    5. Determine whether rollback is required
    6. Roll back to the last known-good artifact
    7. Validate production health
    8. Investigate the root cause

If artifacts are immutable:

    Current:
      v1.4.7

    Previous:
      v1.4.6

Rollback can restore:

    v1.4.6

---

# Scenario Question

## How would you secure AWS credentials in GitHub Actions?

I would prefer GitHub Actions OIDC with AWS IAM roles instead of storing long-lived AWS access keys.

Flow:

    GitHub Actions
          |
          ↓
        OIDC
          |
          ↓
    AWS IAM Role
          |
          ↓
    Environment Resources

Different environments can use different IAM roles.

---

# Scenario Question

## How would you isolate production from lower environments?

I would use multiple layers:

    Separate AWS Account
        +
    Separate IAM Role
        +
    Separate EKS Cluster
        +
    Separate Secrets
        +
    Protected GitHub Environment
        +
    Required Approval
        +
    Least Privilege

The exact isolation level depends on the organization's security and compliance requirements.

---

# Scenario Question

## How would you implement environment-specific Helm configuration?

I would keep the common chart reusable and maintain environment-specific values.

Example:

    helm/
      |
      ├── values.yaml
      ├── values-dev.yaml
      ├── values-qa.yaml
      ├── values-uat.yaml
      └── values-prod.yaml

The same image can be deployed using different configuration values.

---

# Scenario Question

## How would you design Terraform for multiple environments?

I would separate environment configuration and state while reusing common Terraform modules.

Example:

    modules/
      |
      +-- vpc
      +-- eks
      +-- iam
      +-- rds

    environments/
      |
      +-- dev
      +-- qa
      +-- uat
      +-- prod

Each environment would have controlled state and appropriate variables.

---

# Scenario Question

## How would you handle a production-only configuration difference?

I would not rebuild the application.

Instead:

    Same Artifact
        |
        ↓
    Production Configuration
        |
        ↓
    Production Deployment

This keeps the application artifact consistent across environments.

---

# Scenario Question

## What is the biggest benefit of build once, promote many?

The main benefit is consistency.

The exact artifact tested in lower environments is promoted to production.

Flow:

    Build
      |
      ↓
    Test
      |
      ↓
    Validate
      |
      ↓
    Artifact
      |
      ↓
    DEV → QA → UAT → PROD

This reduces the risk of testing one artifact and deploying a different one.

---

# Example GitHub Actions Multi-Environment Workflow

    name: Multi-Environment CI/CD

    on:
      push:
        branches:
          - main

      workflow_dispatch:

    permissions:
      contents: read

    jobs:

      build:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build
            run: |
              ./build.sh

          - name: Test
            run: |
              ./test.sh

      quality:

        needs: build
        runs-on: ubuntu-latest

        steps:

          - name: Quality Checks
            run: |
              ./quality-check.sh

      security:

        needs: build
        runs-on: ubuntu-latest

        steps:

          - name: Security Scan
            run: |
              ./security-scan.sh

      publish:

        needs:
          - quality
          - security

        runs-on: ubuntu-latest

        steps:

          - name: Publish Artifact
            run: |
              ./publish.sh

      deploy-dev:

        needs: publish

        environment:
          name: development

        runs-on: ubuntu-latest

        steps:

          - name: Deploy DEV
            run: |
              ./deploy.sh dev

      deploy-qa:

        needs: deploy-dev

        environment:
          name: qa

        runs-on: ubuntu-latest

        steps:

          - name: Deploy QA
            run: |
              ./deploy.sh qa

      deploy-uat:

        needs: deploy-qa

        environment:
          name: uat

        runs-on: ubuntu-latest

        steps:

          - name: Deploy UAT
            run: |
              ./deploy.sh uat

      deploy-production:

        needs: deploy-uat

        environment:
          name: production

        runs-on: ubuntu-latest

        steps:

          - name: Deploy Production
            run: |
              ./deploy.sh production

---

# Important Point About the Example

The example demonstrates the dependency chain:

    build
      |
      ↓
    quality + security
      |
      ↓
    publish
      |
      ↓
    DEV
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    PROD

In a real enterprise implementation, deployment protection, approvals, artifact retrieval, environment-specific configuration, and rollback mechanisms should be added according to the organization's requirements.

---

# Example GitHub Actions Environment Configuration

Conceptually:

    jobs:

      deploy:

        runs-on: ubuntu-latest

        environment:
          name: production

        steps:

          - name: Deploy
            run: |
              ./deploy.sh

The production environment can have its own:

    Secrets
    Variables
    Protection Rules
    Required Reviewers

---

# Example Manual Environment Selection

Conceptually:

    on:

      workflow_dispatch:

        inputs:

          environment:

            description: Deployment environment

            required: true

            type: choice

            options:
              - dev
              - qa
              - uat
              - production

The workflow can then use the selected environment after appropriate validation and protection.

---

# Environment Promotion Architecture

    ┌─────────────────┐
    │   GitHub Repo   │
    └────────┬────────┘
             ↓
    ┌─────────────────┐
    │ GitHub Actions  │
    └────────┬────────┘
             ↓
    ┌─────────────────┐
    │      Build      │
    └────────┬────────┘
             ↓
    ┌─────────────────┐
    │ Tests / Quality │
    └────────┬────────┘
             ↓
    ┌─────────────────┐
    │ Security Gate   │
    └────────┬────────┘
             ↓
    ┌─────────────────┐
    │     Artifact    │
    └────────┬────────┘
             ↓
          ┌──┴──┐
          ↓     ↓
         DEV   Registry
          ↓
         QA
          ↓
         UAT
          ↓
       Approval
          ↓
        PROD

---

# Multi-Environment DevSecOps Architecture

    Developer
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
        +-- Dependency Scan
        +-- Secret Scan
        |
        ↓
    Quality Gate
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        ↓
    ECR
        |
        ↓
      DEV
        |
        ↓
      QA
        |
        ↓
      UAT
        |
        ↓
    Production Approval
        |
        ↓
      PROD
        |
        ↓
    Health Checks
        |
        ↓
    Monitoring

---

# Multi-Environment GitOps Architecture

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
        +-- Quality
        +-- Security
        |
        ↓
    Container Registry
        |
        ↓
    GitOps Repository
        |
        +-- dev
        +-- qa
        +-- uat
        +-- prod
        |
        ↓
      ArgoCD
        |
        ↓
       EKS

---

# Quality Gates Before Promotion

The artifact should pass required quality controls before environment promotion.

Flow:

    Build
      |
      ↓
    Unit Tests
      |
      ↓
    SonarQube
      |
      ↓
    Quality Gate
      |
      ↓
    Trivy
      |
      ↓
    Security Gate
      |
      ↓
    Artifact
      |
      ↓
    DEV

If a mandatory gate fails:

    Quality Gate
        |
        ↓
      FAIL
        |
        X
    No Promotion

---

# Environment Promotion Gate

Each environment can have its own promotion criteria.

Example:

    DEV
      |
      ↓
    Smoke Tests
      |
      ↓
    QA
      |
      ↓
    Regression Tests
      |
      ↓
    UAT
      |
      ↓
    Business Approval
      |
      ↓
    PROD

---

# Production Deployment Gate

Production should generally have stronger controls.

Example:

    UAT Passed
        |
        ↓
    Change Approved
        |
        ↓
    Production Environment
        |
        ↓
    Required Reviewer
        |
        ↓
    Deploy
        |
        ↓
    Health Check

---

# Multi-Environment CI Checklist

Before considering a multi-environment pipeline complete:

    [ ] Build once
    [ ] Immutable artifact
    [ ] Unit tests
    [ ] Integration tests
    [ ] Quality gate
    [ ] Security gate
    [ ] Dependency scanning
    [ ] Container scanning
    [ ] Environment-specific configuration
    [ ] Environment-specific secrets
    [ ] Least-privilege IAM
    [ ] OIDC authentication
    [ ] DEV environment
    [ ] QA environment
    [ ] UAT environment
    [ ] Production environment
    [ ] Production approval
    [ ] Branch protection
    [ ] Deployment protection
    [ ] Health checks
    [ ] Smoke tests
    [ ] Rollback strategy
    [ ] Artifact traceability
    [ ] Deployment audit trail
    [ ] Monitoring
    [ ] GitOps where applicable
    [ ] Same validated artifact promoted across environments

---

# Interview Summary

If asked:

"How would you design a multi-environment GitHub Actions pipeline?"

A strong answer is:

"I would separate CI validation from environment promotion. The CI pipeline would build the application, run unit and integration tests, perform SonarQube quality analysis and security scans such as Trivy, and publish an immutable artifact only after the required quality and security gates pass. I would then promote the same artifact through DEV, QA, UAT, and Production rather than rebuilding it for each environment. Environment-specific configuration and secrets would be managed separately using GitHub Environments and appropriate secret management. For AWS deployments, I would prefer GitHub Actions OIDC with environment-specific IAM roles. Production would have stronger controls such as protected environments, required approvals, least-privilege permissions, health checks, and rollback capability. For Kubernetes, GitHub Actions can handle CI and artifact publishing while ArgoCD manages GitOps-based deployment to EKS."

---

# Final Mental Model

Remember the complete model:

    Code
      |
      ↓
    CI
      |
      +-- Build
      +-- Test
      +-- Quality
      +-- Security
      |
      ↓
    Immutable Artifact
      |
      ↓
    DEV
      |
      ↓
    Validate
      |
      ↓
    QA
      |
      ↓
    Validate
      |
      ↓
    UAT
      |
      ↓
    Approve
      |
      ↓
    PROD
      |
      ↓
    Validate
      |
      ↓
    Monitor

The most important principles are:

    Build Once
        ↓
    Validate
        ↓
    Promote
        ↓
    Same Artifact
        ↓
    Environment-Specific Configuration
        ↓
    Stronger Controls Near Production

---

# Final Concept

Multi-environment CI/CD is not simply about having DEV, QA, UAT, and Production servers.

It is about creating a controlled promotion process:

    Code
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
    Immutable Artifact
      |
      ↓
    DEV
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    Approval
      |
      ↓
    Production
      |
      ↓
    Validation
      |
      ↓
    Monitoring

The goal is to make sure that the software deployed to production is the same validated artifact that successfully passed the required CI/CD checks, while allowing each environment to have its own configuration, infrastructure, secrets, access controls, approvals, and validation requirements.