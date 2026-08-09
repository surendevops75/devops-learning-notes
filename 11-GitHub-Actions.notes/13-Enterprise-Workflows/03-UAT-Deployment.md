# QA Deployment

QA Deployment is the process of deploying an application to the Quality Assurance environment so that testers and QA engineers can validate application functionality, integration, security, performance, and overall release quality before the application moves to higher environments.

A typical enterprise deployment flow is:

    Developer
        |
        ↓
    Git
        |
        ↓
    CI Pipeline
        |
        +-- Build
        +-- Unit Tests
        +-- Code Quality
        +-- Security Scan
        |
        ↓
    Artifact / Container Image
        |
        ↓
    QA Environment
        |
        ↓
    QA Validation
        |
        ↓
    Approval
        |
        ↓
    SIT / UAT / Production

---

# Purpose of QA Deployment

The main purpose of QA deployment is to validate whether a new application version works correctly in an environment that is separate from development.

QA deployment helps identify:

    Functional Defects
    Integration Issues
    Configuration Issues
    Database Issues
    API Issues
    Deployment Issues
    Security Issues
    Performance Issues
    Environment-Specific Problems

The goal is:

    Build
      |
      ↓
    Deploy
      |
      ↓
    Test
      |
      ↓
    Fix
      |
      ↓
    Retest
      |
      ↓
    Approve

---

# QA Environment

A QA environment is an environment used by QA engineers and testers to validate application releases.

Typical components include:

    Application Servers
    Kubernetes Cluster
    Databases
    Load Balancers
    Message Queues
    Caches
    External Integrations
    Monitoring
    Logging

Example:

    QA Environment

    ALB
      |
      ↓
    Kubernetes
      |
      +-- User Service
      +-- Product Service
      +-- Cart Service
      +-- Order Service
      +-- Payment Service
      +-- Inventory Service
      +-- Notification Service
      |
      ↓
    Database
      |
      +-- PostgreSQL
      +-- MongoDB
      |
      ↓
    RabbitMQ / Other Dependencies

---

# QA vs Development

Development environment:

    Developer Focus
        |
        ↓
    Code Development
        |
        ↓
    Debugging
        |
        ↓
    Rapid Changes

QA environment:

    Tester Focus
        |
        ↓
    Release Validation
        |
        ↓
    Functional Testing
        |
        ↓
    Integration Testing
        |
        ↓
    Defect Identification

The QA environment should be more stable than the development environment.

---

# QA vs Production

QA:

    Testing
    Validation
    Defect Detection
    Test Data
    Frequent Deployments

Production:

    Real Users
    Real Business Traffic
    High Availability
    Production Data
    Strict Change Controls

Therefore:

    QA
      |
      ↓
    Validate Release
      |
      ↓
    Approval
      |
      ↓
    Production

---

# QA Deployment Lifecycle

    Code Commit
        |
        ↓
    Pull Request
        |
        ↓
    Code Review
        |
        ↓
    CI Pipeline
        |
        +-- Build
        +-- Unit Test
        +-- SonarQube
        +-- Trivy
        |
        ↓
    Artifact
        |
        ↓
    QA Deployment
        |
        ↓
    Health Checks
        |
        ↓
    Smoke Tests
        |
        ↓
    QA Testing
        |
        ↓
    Defect / Pass
        |
        ↓
    Approval

---

# QA Deployment Process

A typical QA deployment process is:

    1. Developer Completes Code

    2. Code Is Committed

    3. Pull Request Is Created

    4. Code Review Is Completed

    5. CI Pipeline Runs

    6. Application Is Built

    7. Tests Are Executed

    8. Security Scans Are Executed

    9. Artifact Is Published

    10. QA Deployment Starts

    11. Application Is Deployed

    12. Health Checks Run

    13. Smoke Tests Run

    14. QA Testing Starts

    15. Defects Are Reported

    16. Fixes Are Developed

    17. Application Is Redeployed

    18. Regression Testing Runs

    19. QA Approval Is Given

---

# Source Code to QA

The basic flow is:

    Developer
        |
        ↓
    Git Repository
        |
        ↓
    CI Pipeline
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
    Artifact
        |
        ↓
    QA Environment

This creates a controlled path from source code to QA.

---

# Build Artifact

The application should be packaged into a deployable artifact.

Examples:

    JAR
    WAR
    Docker Image
    Node.js Package
    Python Package

For containerized applications:

    Source Code
        |
        ↓
    Docker Build
        |
        ↓
    Docker Image
        |
        ↓
    Container Registry
        |
        ↓
    QA Deployment

---

# Container Image in QA

For Kubernetes-based applications:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    CI
        |
        ↓
    Docker Build
        |
        ↓
    ECR
        |
        ↓
    QA EKS
        |
        ↓
    Kubernetes Deployment

The exact image version should be traceable to the source commit.

---

# Image Tagging

Example:

    payment:1.4.7

or:

    payment:qa-1.4.7

or:

    payment:<commit-sha>

The tag should clearly identify the version being tested.

Immutable image references are preferred for reliable deployments.

---

# Commit SHA Traceability

A good QA deployment should provide traceability:

    Git Commit
        |
        ↓
    CI Build
        |
        ↓
    Image
        |
        ↓
    QA Deployment
        |
        ↓
    Test Results

Example:

    Commit:
    abc123

    Image:
    payment:abc123

    QA:
    payment deployment using abc123

This makes troubleshooting easier.

---

# Environment Configuration

QA requires environment-specific configuration.

Examples:

    Database URL
    API URLs
    Service URLs
    Environment Variables
    ConfigMaps
    Secrets
    Feature Flags

Example:

    Application
        |
        ↓
    QA Configuration
        |
        +-- QA Database
        +-- QA API
        +-- QA RabbitMQ
        +-- QA Secrets

The same application artifact can often be promoted across environments while environment-specific configuration changes.

---

# Configuration Separation

Avoid mixing:

    Development Configuration

with:

    QA Configuration

and:

    Production Configuration

Example:

    Development
        |
        ↓
    dev.example.internal

    QA
        |
        ↓
    qa.example.internal

    Production
        |
        ↓
    api.example.com

Environment-specific configuration should be managed separately.

---

# Secrets in QA

QA applications may require:

    Database Credentials
    API Tokens
    Authentication Secrets
    TLS Certificates
    Service Credentials

Secrets should be stored securely.

Do not:

    Hardcode Secrets
    Commit Secrets to Git
    Print Secrets in CI Logs

Use appropriate secret-management mechanisms.

---

# QA Database

QA may use a dedicated database.

Example:

    QA Application
        |
        ↓
    QA Database

This prevents QA testing from affecting production data.

Typical QA database tasks include:

    Schema Validation
    Migration Testing
    Data Validation
    CRUD Testing
    Integration Testing

---

# Database Migration in QA

If a release contains database changes:

    Application Release
        |
        ↓
    Database Migration
        |
        ↓
    QA Validation
        |
        ↓
    Application Testing

Verify:

    Migration Successful
    Schema Correct
    Application Compatible
    Existing Data Accessible
    New Functionality Works

---

# QA Test Data

QA may require controlled test data.

Examples:

    Test Users
    Test Products
    Test Orders
    Test Payments
    Test Inventory

Test data should be:

    Controlled
    Repeatable
    Appropriate For Testing
    Non-Production Sensitive Data

---

# QA Deployment Strategies

Common strategies include:

    Direct Deployment
    Rolling Deployment
    Blue-Green Deployment
    Canary Deployment

The selected strategy depends on:

    Application
    Environment
    Risk
    Infrastructure
    Testing Requirements

---

# Direct QA Deployment

Simple flow:

    Build
      |
      ↓
    Artifact
      |
      ↓
    QA
      |
      ↓
    Test

This is common for development and QA environments where rapid iteration is required.

---

# Rolling QA Deployment

Flow:

    Old Pods
        |
        ↓
    New Pod
        |
        ↓
    Health Check
        |
        ↓
    Ready
        |
        ↓
    Replace Old Pod
        |
        ↓
    Next Pod

This helps validate the same deployment mechanism used in higher environments.

---

# Blue-Green QA Deployment

Two environments:

    Blue = Current
    Green = New

Flow:

    Green
      |
      ↓
    Deploy
      |
      ↓
    Validate
      |
      ↓
    Switch Traffic

This can be useful for testing deployment behavior before production.

---

# QA Deployment Using Kubernetes

Typical architecture:

    GitHub
      |
      ↓
    CI/CD
      |
      ↓
    ECR
      |
      ↓
    EKS QA
      |
      ↓
    Kubernetes Deployment
      |
      ↓
    Service
      |
      ↓
    ALB
      |
      ↓
    QA Users

---

# Kubernetes Deployment

A Deployment manages the desired application state.

Example:

    apiVersion: apps/v1
    kind: Deployment

    metadata:
      name: payment

    spec:
      replicas: 2

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
              image: payment:qa-1.4.7

---

# Apply Deployment

Command:

    kubectl apply -f deployment.yaml

Then check:

    kubectl get deployment

Then:

    kubectl get pods

Then:

    kubectl rollout status deployment/payment

---

# Verify Rollout

Command:

    kubectl rollout status deployment/payment

Expected:

    deployment "payment" successfully rolled out

If rollout fails:

    Stop QA Validation
        |
        ↓
    Investigate
        |
        ↓
    Fix
        |
        ↓
    Redeploy

---

# QA Health Checks

After deployment, verify:

    Startup Probe
    Readiness Probe
    Liveness Probe

Expected:

    Pods Running
    Pods Ready
    Health Checks Passing

Commands:

    kubectl get pods

    kubectl describe pod <pod-name>

---

# QA Smoke Test

Immediately after deployment, run a smoke test.

Example:

    Application URL
        |
        ↓
    GET /health
        |
        ↓
    HTTP 200
        |
        ↓
    Login
        |
        ↓
    Critical API
        |
        ↓
    Smoke Test Pass

If smoke testing fails:

    Stop QA Testing
        |
        ↓
    Investigate Deployment
        |
        ↓
    Fix / Redeploy

---

# QA Functional Testing

Functional testing verifies that application requirements work correctly.

Examples:

    User Registration
    Login
    Product Search
    Add To Cart
    Order Creation
    Payment
    Inventory
    Notification

The exact tests depend on application requirements.

---

# QA Integration Testing

Integration testing verifies communication between components.

Example:

    User Service
        |
        ↓
    Database

    Order Service
        |
        ↓
    Payment Service

    Payment Service
        |
        ↓
    RabbitMQ

    Notification Service
        |
        ↓
    External Provider

---

# QA Regression Testing

Regression testing verifies that existing functionality still works after new changes.

Flow:

    New Feature
        |
        ↓
    Deployment
        |
        ↓
    New Feature Test
        |
        ↓
    Existing Feature Tests
        |
        ↓
    Regression Result

A new change should not unexpectedly break existing functionality.

---

# QA API Testing

API validation can include:

    HTTP Method
    URL
    Request
    Response
    Status Code
    Headers
    Authentication
    Authorization
    Response Time

Example:

    POST /orders

Expected:

    HTTP 201

Then:

    GET /orders/<id>

Expected:

    HTTP 200

---

# QA UI Testing

For applications with a user interface, QA may validate:

    Login
    Navigation
    Forms
    Buttons
    Search
    Validation Messages
    Browser Compatibility
    Responsive Behavior

---

# QA Performance Testing

Depending on the release, QA may validate:

    Response Time
    Throughput
    Concurrent Users
    CPU
    Memory
    Database Performance

Example:

    Request Rate
        |
        ↓
    Application
        |
        ↓
    Measure Latency
        |
        ↓
    Analyze Performance

---

# QA Security Testing

Security validation may include:

    Authentication
    Authorization
    Input Validation
    Dependency Vulnerabilities
    Container Vulnerabilities
    Secret Exposure
    TLS
    Access Control

DevSecOps tools may include:

    SonarQube
    Trivy
    Veracode

---

# SonarQube in QA Pipeline

SonarQube is normally used during CI before deployment.

Flow:

    Code
      |
      ↓
    SonarQube
      |
      ↓
    Quality Gate
      |
      +------ Fail ------→ Stop
      |
      +------ Pass
              |
              ↓
            Build
              |
              ↓
            QA

---

# Trivy in QA Pipeline

Trivy can scan container images.

Flow:

    Docker Build
        |
        ↓
    Trivy Scan
        |
        ↓
    Vulnerability Result
        |
        +------ Fail ------→ Stop
        |
        +------ Pass
                |
                ↓
              ECR
                |
                ↓
              QA

---

# QA Deployment Approval

Some organizations require approval before deploying to QA.

Flow:

    Build
      |
      ↓
    Test
      |
      ↓
    Security
      |
      ↓
    Approval
      |
      ↓
    QA Deployment

Approval requirements depend on organizational processes.

---

# Automated QA Deployment

A CI/CD pipeline can automatically deploy successful builds to QA.

Example:

    Code Push
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
    Publish Image
        |
        ↓
    Deploy QA
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    QA Testing

---

# GitHub Actions QA Deployment

A conceptual workflow:

    name: QA Deployment

    on:
      push:
        branches:
          - develop

    jobs:

      build:
        runs-on: ubuntu-latest

        steps:
          - Checkout Code
          - Build Application
          - Run Tests
          - Build Docker Image
          - Scan Image
          - Push Image

      deploy-qa:
        needs: build
        runs-on: ubuntu-latest

        steps:
          - Deploy To QA
          - Wait For Rollout
          - Run Health Check
          - Run Smoke Test

---

# QA Deployment Gates

A deployment gate determines whether the pipeline can continue.

Example:

    Build
      |
      ↓
    Unit Test
      |
      ↓
    Security Scan
      |
      ↓
    Deploy QA
      |
      ↓
    Health Check
      |
      ↓
    Smoke Test
      |
      +------ Fail ------→ Stop
      |
      +------ Pass
              |
              ↓
           QA Testing

---

# Quality Gate

A quality gate can check:

    Test Results
    Code Quality
    Security
    Coverage
    Vulnerabilities

If the required criteria are not met:

    Quality Gate Failed
        |
        ↓
    Stop Pipeline

---

# QA Deployment Gate

After deployment:

    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    QA Validation
        |
        +------ Failed ------→ Reject
        |
        +------ Passed ------→ Approve

This creates controlled progression to higher environments.

---

# QA Defect Lifecycle

When QA identifies a problem:

    Test
      |
      ↓
    Defect Found
      |
      ↓
    Defect Reported
      |
      ↓
    Developer Investigation
      |
      ↓
    Fix
      |
      ↓
    New Build
      |
      ↓
    QA Redeployment
      |
      ↓
    Retest
      |
      ↓
    Regression Testing
      |
      ↓
    Close Defect

---

# Defect Severity

Defects may be classified according to organizational standards.

Typical categories:

    Critical
    High
    Medium
    Low

Example:

    Critical
        |
        ↓
    Application Cannot Be Used

    High
        |
        ↓
    Major Feature Broken

    Medium
        |
        ↓
    Important Functionality Affected

    Low
        |
        ↓
    Minor Issue

Exact severity definitions vary by organization.

---

# QA Rejection

A release may be rejected when:

    Critical Defects Exist
    Smoke Tests Fail
    Major Functionality Fails
    Integration Tests Fail
    Security Requirements Fail
    Performance Is Unacceptable

Flow:

    QA Testing
        |
        ↓
    Failure
        |
        ↓
    Reject Release
        |
        ↓
    Developer Fix
        |
        ↓
    Redeploy QA

---

# QA Approval

A release may be approved when:

    Functional Tests Pass
    Integration Tests Pass
    Regression Tests Pass
    Security Validation Passes
    Performance Is Acceptable
    Critical Defects Are Resolved

Then:

    QA Approval
        |
        ↓
    SIT / UAT
        |
        ↓
    Production

---

# QA Environment Variables

Example:

    ENVIRONMENT=qa

    DATABASE_HOST=qa-db

    API_URL=https://qa-api.example.internal

    LOG_LEVEL=INFO

These values should be managed through appropriate configuration mechanisms.

---

# ConfigMap in QA

Example:

    apiVersion: v1
    kind: ConfigMap

    metadata:
      name: payment-config

    data:
      ENVIRONMENT: "qa"
      LOG_LEVEL: "INFO"

Application:

    Pod
      |
      ↓
    ConfigMap
      |
      ↓
    Environment Variables

---

# Secret in QA

Example concept:

    apiVersion: v1
    kind: Secret

    metadata:
      name: payment-secret

    stringData:
      DATABASE_USERNAME: "qa-user"
      DATABASE_PASSWORD: "<managed-secret>"

Secrets should be protected and should never be exposed in logs or source control.

---

# Namespace-Based QA

A Kubernetes cluster may use a dedicated namespace.

Example:

    kubectl create namespace qa

Deploy:

    kubectl apply -f deployment.yaml -n qa

Check:

    kubectl get pods -n qa

This provides logical separation for QA resources.

---

# Environment Separation

Example:

    EKS Cluster
        |
        +-- namespace: dev
        |
        +-- namespace: qa
        |
        +-- namespace: sit
        |
        +-- namespace: uat
        |
        └-- namespace: prod

Some organizations use separate clusters or accounts instead.

The architecture depends on security and organizational requirements.

---

# QA Deployment Using Helm

Helm can manage Kubernetes application deployments.

Example:

    helm upgrade --install payment ./payment-chart \
      --namespace qa \
      --create-namespace \
      --set image.tag=qa-1.4.7

Then verify:

    kubectl get pods -n qa

and:

    kubectl rollout status deployment/payment -n qa

---

# Helm Values for QA

Example:

    values-qa.yaml

    environment: qa

    image:
      repository: payment
      tag: qa-1.4.7

    replicas: 2

    resources:
      requests:
        cpu: 250m
        memory: 256Mi

The QA values file can contain environment-specific configuration.

---

# QA Deployment Using ArgoCD

GitOps flow:

    Developer
        |
        ↓
    Git
        |
        ↓
    Manifest / Helm Values
        |
        ↓
    ArgoCD
        |
        ↓
    QA EKS
        |
        ↓
    Application

ArgoCD continuously reconciles the desired state from Git with the QA Kubernetes environment.

---

# ArgoCD QA Flow

    Git Repository
        |
        ↓
    QA Manifest Updated
        |
        ↓
    ArgoCD Detects Change
        |
        ↓
    Sync
        |
        ↓
    QA EKS
        |
        ↓
    Kubernetes Resources
        |
        ↓
    Health Validation

---

# QA GitOps Promotion

A common GitOps approach is:

    Application Repository
        |
        ↓
    CI Builds Image
        |
        ↓
    ECR
        |
        ↓
    Update QA Manifest
        |
        ↓
    ArgoCD
        |
        ↓
    QA

After QA approval:

    Update SIT Manifest
        |
        ↓
    ArgoCD
        |
        ↓
    SIT

---

# QA Manifest Example

Conceptually:

    environments/
    └── qa/
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        └── values.yaml

Environment configuration remains separated from application source code.

---

# QA Deployment Monitoring

After deployment, monitor:

    Pod Status
    Restart Count
    CPU
    Memory
    HTTP Requests
    HTTP Errors
    Latency
    Application Logs

Tools:

    Prometheus
    Grafana
    ELK

---

# Prometheus QA Validation

Prometheus can provide metrics such as:

    Request Rate
    Error Rate
    Latency
    CPU
    Memory
    Pod Availability

Example:

    QA Deployment
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

# Grafana QA Dashboard

Useful dashboard panels:

    Pod Status
    Pod Restarts
    Request Rate
    HTTP 4xx
    HTTP 5xx
    Latency
    CPU
    Memory

This provides quick visibility into the health of the QA release.

---

# ELK QA Validation

ELK can be used to inspect:

    Application Errors
    Exceptions
    API Requests
    Authentication Failures
    Database Errors
    Dependency Failures

Flow:

    QA Application
        |
        ↓
    Logs
        |
        ↓
    ELK
        |
        ↓
    QA Investigation

---

# QA Deployment Rollback

If a QA deployment fails:

    New Version
        |
        ↓
    QA Validation
        |
        ↓
    Failure
        |
        ↓
    Rollback
        |
        ↓
    Previous Version
        |
        ↓
    Validate Again

Kubernetes command:

    kubectl rollout undo deployment/payment -n qa

Then:

    kubectl rollout status deployment/payment -n qa

---

# Helm Rollback

If Helm is used:

    helm history payment -n qa

Then:

    helm rollback payment <revision> -n qa

Validate:

    kubectl get pods -n qa

and:

    kubectl rollout status deployment/payment -n qa

---

# ArgoCD Rollback

In GitOps, rollback is commonly performed by reverting the Git change to the previously known-good configuration.

Flow:

    Bad Change
        |
        ↓
    Git
        |
        ↓
    Revert
        |
        ↓
    ArgoCD
        |
        ↓
    QA
        |
        ↓
    Previous Version

The exact rollback procedure depends on the organization's GitOps process.

---

# QA Deployment and Change Management

Enterprise environments may require:

    Change Request
    Release Ticket
    Approval
    Deployment Record
    Test Evidence
    QA Approval

A typical process:

    Change Request
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Testing
        |
        ↓
    Evidence
        |
        ↓
    Approval

---

# QA Deployment Evidence

Useful evidence includes:

    Build Number
    Git Commit
    Image Tag
    Deployment Time
    Test Results
    Smoke Test Results
    Security Scan Results
    QA Test Results
    Defect Status
    Approval

This helps with auditing and release traceability.

---

# QA Deployment Version Tracking

Track:

    Application Version
    Build Number
    Git Commit
    Docker Image
    Deployment Time
    Environment

Example:

    Application:
    Payment

    Version:
    1.4.7

    Commit:
    abc123

    Environment:
    QA

    Deployment:
    2026-08-09

---

# QA Deployment Notifications

CI/CD systems may notify teams about:

    Deployment Started
    Deployment Successful
    Deployment Failed
    Smoke Test Failed
    QA Approval
    Rollback

Possible communication channels depend on organizational tooling.

---

# QA Deployment Failure Handling

If deployment fails:

    Detect Failure
        |
        ↓
    Stop Pipeline
        |
        ↓
    Collect Logs
        |
        ↓
    Identify Root Cause
        |
        ↓
    Fix
        |
        ↓
    Rebuild
        |
        ↓
    Redeploy
        |
        ↓
    Retest

Do not blindly retry without understanding the failure.

---

# Common QA Deployment Failures

## ImagePullBackOff

Possible causes:

    Wrong Image
    Wrong Tag
    Registry Authentication
    Image Does Not Exist

Check:

    kubectl describe pod <pod-name> -n qa

---

# Common QA Deployment Failures

## CrashLoopBackOff

Possible causes:

    Application Crash
    Configuration Error
    Missing Secret
    Database Failure
    Incorrect Command
    Resource Issue

Check:

    kubectl logs <pod-name> -n qa

and:

    kubectl logs <pod-name> -n qa --previous

---

# Common QA Deployment Failures

## Readiness Probe Failure

Possible causes:

    Wrong Endpoint
    Wrong Port
    Application Not Ready
    Dependency Failure
    Timeout

Check:

    kubectl describe pod <pod-name> -n qa

---

# Common QA Deployment Failures

## No Service Endpoints

Possible causes:

    Wrong Selector
    Pods Not Ready
    Wrong Labels

Check:

    kubectl get svc -n qa

and:

    kubectl get endpoints -n qa

---

# Common QA Deployment Failures

## ALB Unhealthy

Possible causes:

    Wrong Health Path
    Wrong Port
    Application Error
    Security Configuration
    Ingress Configuration

Validate the complete request path:

    ALB
      |
      ↓
    Ingress
      |
      ↓
    Service
      |
      ↓
    Pod

---

# Common QA Deployment Failures

## Database Connection Failure

Possible causes:

    Wrong Host
    Wrong Port
    Wrong Credentials
    Network Issue
    Database Unavailable
    Security Rules

Check:

    Application Logs
    Configuration
    Secrets
    Network Connectivity

---

# Common QA Deployment Failures

## Configuration Error

Example:

    ENVIRONMENT=production

inside:

    QA

or:

    DATABASE_URL points to production

This can create serious risks.

Always verify environment-specific configuration before testing.

---

# QA Deployment Security

Important controls:

    Environment Separation
    Least Privilege
    Secret Management
    Secure CI/CD Credentials
    Network Restrictions
    TLS
    Image Scanning
    Code Scanning
    Access Control

QA should never become a path to production data exposure.

---

# QA Deployment Best Practices

- Use versioned artifacts
- Use immutable image tags or digests
- Keep QA configuration separate
- Never commit secrets
- Automate deployment
- Automate smoke tests
- Validate health checks
- Validate Service endpoints
- Validate ALB health
- Monitor application metrics
- Review application logs
- Maintain deployment traceability
- Keep test data controlled
- Use rollback procedures
- Record test evidence
- Require appropriate approvals
- Keep QA environment stable
- Use the same deployment mechanism where possible as higher environments

---

# QA Deployment Anti-Patterns

## Manual Copying of Artifacts

Bad:

    Developer Laptop
        |
        ↓
    Manually Copy JAR
        |
        ↓
    QA Server

Problems:

    No Traceability
    Human Error
    Inconsistent Deployment

Better:

    Git
        |
        ↓
    CI/CD
        |
        ↓
    Artifact
        |
        ↓
    QA

---

# QA Deployment Anti-Pattern

## Using Latest Tag

Bad:

    image: payment:latest

The tag can change unexpectedly.

Better:

    image: payment:1.4.7

or an immutable image digest.

---

# QA Deployment Anti-Pattern

## Production Data in QA

Avoid using sensitive production data in QA unless there is a formally controlled and approved process.

Prefer:

    Synthetic Data
    Masked Data
    Sanitized Data

---

# QA Deployment Anti-Pattern

## Skipping Smoke Tests

Bad:

    Deploy
      |
      ↓
    Immediately Declare Success

Better:

    Deploy
      |
      ↓
    Health Check
      |
      ↓
    Smoke Test
      |
      ↓
    Validate

---

# QA Deployment Anti-Pattern

## Ignoring Failed Tests

If QA tests fail:

    Do Not Ignore
    Do Not Hide
    Do Not Bypass

Instead:

    Report
      |
      ↓
    Investigate
      |
      ↓
    Fix
      |
      ↓
    Retest

---

# QA Deployment Anti-Pattern

## Different Deployment Process From Production

If QA uses:

    Manual Deployment

and Production uses:

    Automated CI/CD

then production-specific deployment issues may not be discovered in QA.

Prefer similar deployment mechanisms across environments when practical.

---

# QA Deployment and Enterprise Promotion

Typical flow:

    Development
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

Each environment provides a different validation stage.

---

# QA Promotion Criteria

Before moving from QA:

    Build Successful
    Security Scans Passed
    Deployment Successful
    Pods Healthy
    Smoke Tests Passed
    Functional Tests Passed
    Integration Tests Passed
    Regression Tests Passed
    Critical Defects Resolved
    QA Approval Received

Then:

    QA
      |
      ↓
    SIT

---

# QA Deployment End-to-End Flow

    Developer
        |
        ↓
    Feature Branch
        |
        ↓
    Pull Request
        |
        ↓
    Code Review
        |
        ↓
    Merge
        |
        ↓
    CI Pipeline
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
    ECR
        |
        ↓
    QA Deployment
        |
        ↓
    EKS
        |
        ↓
    Health Checks
        |
        ↓
    Smoke Tests
        |
        ↓
    Functional Testing
        |
        ↓
    Integration Testing
        |
        ↓
    Regression Testing
        |
        ↓
    QA Approval
        |
        ↓
    SIT

---

# Real-World QA Deployment Example

Suppose the application version is:

    payment:1.4.7

Process:

    Developer
        |
        ↓
    Commit
        |
        ↓
    GitHub Actions
        |
        ↓
    Maven Build
        |
        ↓
    Unit Tests
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    QA EKS
        |
        ↓
    Helm / Kubernetes
        |
        ↓
    Pods
        |
        ↓
    Readiness
        |
        ↓
    Smoke Test
        |
        ↓
    QA Functional Testing
        |
        ↓
    QA Approval

---

# Real-World QA Failure Example

Deployment:

    payment:1.4.7

Kubernetes:

    Deployment = Successful
    Pods = Running

But:

    Smoke Test = Failed
    Payment API = HTTP 500

Action:

    Stop Promotion
        |
        ↓
    Check Logs
        |
        ↓
    Check Database
        |
        ↓
    Check Configuration
        |
        ↓
    Identify Root Cause
        |
        ↓
    Fix
        |
        ↓
    Build 1.4.8
        |
        ↓
    Deploy QA
        |
        ↓
    Retest

---

# Real-World QA Approval Example

Release:

    payment:1.4.7

Results:

    Build = Pass
    Unit Tests = Pass
    SonarQube = Pass
    Trivy = Pass
    Deployment = Pass
    Health Checks = Pass
    Smoke Tests = Pass
    Functional Tests = Pass
    Integration Tests = Pass
    Regression = Pass
    Critical Defects = 0

Result:

    QA Approved

Next:

    QA
      |
      ↓
    SIT

---

# QA Deployment Interview Questions

## Basic

1. What is a QA environment?

2. Why do we deploy applications to QA?

3. What is the purpose of QA deployment?

4. What is the difference between Development and QA?

5. What is the difference between QA and Production?

6. What happens after a successful CI build?

7. What checks do you perform after QA deployment?

8. What is a smoke test?

9. What is regression testing?

10. What is QA approval?

---

# QA Deployment Interview Questions

## Intermediate

11. How do you deploy an application to QA?

12. How do you deploy a Dockerized application to QA?

13. How do you deploy an application to EKS QA?

14. How do you verify a Kubernetes deployment?

15. How do you validate Pods after deployment?

16. How do you validate a Service?

17. How do you validate an ALB?

18. How do you troubleshoot a failed QA deployment?

19. How do you rollback a Kubernetes deployment?

20. How do you manage QA-specific configuration?

21. How do you manage QA secrets?

22. How do you validate database migrations in QA?

---

# QA Deployment Interview Questions

## Advanced

23. How would you design an enterprise QA deployment pipeline?

24. How would you implement automated smoke testing?

25. How would you integrate QA deployment with GitHub Actions?

26. How would you implement GitOps-based QA deployment using ArgoCD?

27. How would you promote an image from QA to higher environments?

28. How would you prevent production configuration from being used in QA?

29. How would you implement rollback after failed QA validation?

30. How would you maintain traceability from Git commit to QA deployment?

31. How would you design QA validation for a microservices application?

32. How would you handle database migrations during QA deployment?

33. How would you ensure QA and production deployment processes are consistent?

34. How would you handle a release that passes technical checks but fails business validation?

---

# Scenario-Based Interview Question

## QA Deployment Succeeds but Pods Are Not Ready

Check:

    kubectl get pods -n qa

Then:

    kubectl describe pod <pod-name> -n qa

Check:

    Readiness Probe
    Application Logs
    Configuration
    Dependencies
    Service

Do not proceed with QA testing until the deployment is healthy.

---

# Scenario-Based Interview Question

## QA Deployment Produces 503

Check:

    Pods
    Readiness
    Service
    Endpoints
    Ingress
    ALB
    Health Endpoint

Request path:

    User
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
      ↓
    Pod

Find the exact layer where the request fails.

---

# Scenario-Based Interview Question

## QA Smoke Test Fails

Process:

    Smoke Test Failure
        |
        ↓
    Check Application Logs
        |
        ↓
    Check Configuration
        |
        ↓
    Check Dependencies
        |
        ↓
    Check Database
        |
        ↓
    Identify Root Cause
        |
        ↓
    Fix
        |
        ↓
    Redeploy
        |
        ↓
    Retest

---

# Scenario-Based Interview Question

## QA Deployment Uses Wrong Image

Expected:

    payment:1.4.7

Actual:

    payment:1.4.6

Check:

    CI Build
    ECR
    Deployment Manifest
    Helm Values
    ArgoCD
    Pod Image

Trace:

    Git
        |
        ↓
    CI
        |
        ↓
    ECR
        |
        ↓
    Manifest
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

---

# Scenario-Based Interview Question

## QA Tests Pass but Production Deployment Later Fails

Possible reason:

    Environment Differences

Examples:

    Different Configuration
    Different Secrets
    Different Database
    Different Network
    Different IAM
    Different Infrastructure
    Different Traffic

Solution:

    Keep deployment mechanisms consistent
    Keep environment configuration controlled
    Validate production-specific differences
    Use infrastructure as code
    Use automated deployment

---

# QA Deployment Checklist

    Code Reviewed
        |
        ↓
    CI Passed
        |
        ↓
    Unit Tests Passed
        |
        ↓
    SonarQube Passed
        |
        ↓
    Trivy Passed
        |
        ↓
    Artifact Published
        |
        ↓
    QA Deployment Started
        |
        ↓
    Rollout Successful
        |
        ↓
    Pods Healthy
        |
        ↓
    Health Checks Passed
        |
        ↓
    Service Validated
        |
        ↓
    ALB Validated
        |
        ↓
    Smoke Tests Passed
        |
        ↓
    Functional Tests Passed
        |
        ↓
    Integration Tests Passed
        |
        ↓
    Regression Passed
        |
        ↓
    Critical Defects Resolved
        |
        ↓
    QA Approval
        |
        ↓
    Promote To SIT

---

# QA Deployment Best-Practice Architecture

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Pull Request
        |
        ↓
    Code Review
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
    ECR
        |
        ↓
    ArgoCD
        |
        ↓
    EKS QA
        |
        +-- Deployment
        +-- Service
        +-- Ingress
        |
        ↓
    ALB
        |
        ↓
    Application
        |
        +-- Health Checks
        +-- Smoke Tests
        +-- Functional Tests
        |
        ↓
    Prometheus
        |
        ↓
    Grafana
        |
        ↓
    ELK
        |
        ↓
    QA Approval
        |
        ↓
    SIT

---

# Final Mental Model

Remember the QA deployment flow:

    CODE
      |
      ↓
    BUILD
      |
      ↓
    TEST
      |
      ↓
    SECURITY
      |
      ↓
    ARTIFACT
      |
      ↓
    DEPLOY QA
      |
      ↓
    HEALTH CHECK
      |
      ↓
    SMOKE TEST
      |
      ↓
    FUNCTIONAL TEST
      |
      ↓
    INTEGRATION TEST
      |
      ↓
    REGRESSION TEST
      |
      ↓
    QA APPROVAL
      |
      ↓
    PROMOTE

The key principle is:

    Build Once
        |
        ↓
    Deploy Consistently
        |
        ↓
    Validate Thoroughly
        |
        ↓
    Approve
        |
        ↓
    Promote

A strong QA deployment process provides:

    Faster Feedback
        +
    Controlled Releases
        +
    Better Defect Detection
        +
    Deployment Traceability
        +
    Reliable Environment Promotion
        +
    Reduced Production Risk

The ultimate goal is:

    Code
      |
      ↓
    Quality
      |
      ↓
    Security
      |
      ↓
    QA Validation
      |
      ↓
    Confidence
      |
      ↓
    Safe Promotion