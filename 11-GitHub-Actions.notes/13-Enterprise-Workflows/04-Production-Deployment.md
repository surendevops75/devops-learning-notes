# Production Deployment

Production Deployment is the process of releasing an application version into the live production environment where real users, real business transactions, and production workloads are handled.

Production deployment is the most controlled stage in an enterprise CI/CD process because failures can directly affect:

    Real Users
    Real Transactions
    Business Operations
    Revenue
    Data
    Availability
    Customer Experience

A typical enterprise flow is:

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
    Business Sign-Off
        |
        ↓
    Change Approval
        |
        ↓
    Production
        |
        ↓
    Post-Deployment Validation

---

# Purpose of Production Deployment

The main purpose of production deployment is to release an approved and validated application version safely to real users.

Production deployment focuses on:

    Availability
    Reliability
    Security
    Performance
    Scalability
    Zero / Minimal Downtime
    Data Integrity
    Rollback
    Monitoring
    Business Continuity

The key question is:

    "Can we safely release this version to production?"

---

# QA vs SIT vs UAT vs Production

QA:

    Application Testing

SIT:

    System Integration Testing

UAT:

    Business Acceptance

Production:

    Real Users
    Real Traffic
    Real Business Transactions

Typical flow:

    QA
      |
      ↓
    Technical Validation

    SIT
      |
      ↓
    Integration Validation

    UAT
      |
      ↓
    Business Validation

    Production
      |
      ↓
    Live Operation

---

# Production Environment

A production environment may contain:

    Load Balancer
    Kubernetes Cluster
    Application Services
    Databases
    Message Queues
    Cache
    DNS
    TLS
    Monitoring
    Logging
    External Integrations

Example:

    Internet
        |
        ↓
    Route53
        |
        ↓
    ALB
        |
        ↓
    EKS
        |
        +-- User Service
        +-- Product Service
        +-- Cart Service
        +-- Order Service
        +-- Payment Service
        +-- Inventory Service
        +-- Notification Service
        |
        +-- RDS
        +-- RabbitMQ
        +-- Cache
        |
        ↓
    External Systems

---

# Production Deployment Lifecycle

    Code
      |
      ↓
    CI
      |
      ↓
    Build
      |
      ↓
    Unit Tests
      |
      ↓
    Security Checks
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
    Business Sign-Off
      |
      ↓
    Change Approval
      |
      ↓
    Production Deployment
      |
      ↓
    Health Checks
      |
      ↓
    Smoke Tests
      |
      ↓
    Post-Deployment Validation
      |
      ↓
    Monitoring

---

# Production Deployment Process

A typical enterprise production deployment process is:

    1. UAT Approval

    2. Business Sign-Off

    3. Production Artifact Identified

    4. Production Configuration Prepared

    5. Database Migration Plan Prepared

    6. Deployment Plan Prepared

    7. Rollback Plan Prepared

    8. Monitoring Prepared

    9. Change Request Approved

    10. Deployment Window Confirmed

    11. Production Deployment Started

    12. Application Rollout Validated

    13. Health Checks Executed

    14. Smoke Tests Executed

    15. Production Metrics Monitored

    16. Logs Reviewed

    17. Business Validation Performed

    18. Deployment Declared Successful

---

# Production Readiness

Before production deployment, verify:

    QA Passed
    SIT Passed
    UAT Passed
    Business Sign-Off
    Security Validation
    Artifact Verified
    Configuration Verified
    Database Plan Ready
    Rollback Plan Ready
    Monitoring Ready
    Alerts Ready
    Access Ready
    Change Approved
    Deployment Window Confirmed

---

# Production Deployment Approval

Production deployment normally requires controlled approvals.

Possible approvals:

    Product Owner
    QA Lead
    DevOps Lead
    Release Manager
    Change Manager
    Business Owner

The exact process depends on the organization.

Typical flow:

    UAT Approval
        |
        ↓
    Change Request
        |
        ↓
    Approval
        |
        ↓
    Production Deployment

---

# Change Request

Enterprise organizations may create a change request before production deployment.

A change request may contain:

    Application
    Version
    Change Description
    Business Reason
    Deployment Plan
    Risk
    Impact
    Validation Plan
    Rollback Plan
    Deployment Window
    Approvals

Example:

    Application:
    Payment

    Version:
    1.4.7

    Environment:
    Production

    Change:
    Release Payment Service Version 1.4.7

---

# Production Deployment Window

Production deployments may happen during an approved deployment window.

Example:

    Deployment Window
        |
        ↓
    Pre-Checks
        |
        ↓
    Deployment
        |
        ↓
    Validation
        |
        ↓
    Monitoring

The timing depends on:

    Business Impact
    Traffic
    Risk
    Maintenance Policy
    Change Management

---

# Production Pre-Checks

Before deployment:

    Verify Artifact
    Verify Git Commit
    Verify Image
    Verify Configuration
    Verify Secrets
    Verify Database
    Verify Cluster Health
    Verify Capacity
    Verify Monitoring
    Verify Alerts
    Verify Rollback
    Verify Access

Example:

    kubectl get nodes

    kubectl get pods -A

Check for:

    Node Problems
    Unhealthy Pods
    Resource Pressure
    Existing Incidents

---

# Production Artifact

The production artifact should be the approved artifact.

Example:

    payment:1.4.7

The same artifact should have already passed:

    QA
    SIT
    UAT

Avoid rebuilding the application just before production unless the organization's process explicitly requires it.

---

# Build Once and Promote

Preferred enterprise model:

    Source
      |
      ↓
    Build
      |
      ↓
    Artifact
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

Example:

    payment:1.4.7

The artifact remains the same while configuration changes by environment.

---

# Production Configuration

Production configuration may include:

    Database URL
    API URLs
    Service URLs
    Environment Variables
    Feature Flags
    External Endpoints
    Resource Configuration

Example:

    ENVIRONMENT=production

    DATABASE_HOST=prod-db

    PAYMENT_API=payment.production

Configuration must be verified carefully before deployment.

---

# Production Secrets

Production may require:

    Database Credentials
    API Credentials
    TLS Certificates
    Authentication Secrets
    Service Credentials

Secrets should be managed securely.

Never:

    Commit Secrets
    Hardcode Secrets
    Print Secrets
    Store Secrets In Plain Text

---

# Production Database

Production databases contain real business data.

Therefore:

    Database Changes
        |
        ↓
    High Risk

Before deployment:

    Backup / Recovery Strategy
    Migration Validation
    Compatibility Check
    Rollback Strategy
    Data Integrity Plan

must be considered.

---

# Database Migration

If the release contains schema changes:

    Application Release
        |
        ↓
    Database Migration
        |
        ↓
    Application Validation

Prefer backward-compatible database changes where possible.

Example:

    Add New Column
        |
        ↓
    Deploy Compatible Application
        |
        ↓
    Validate
        |
        ↓
    Gradually Adopt New Column

---

# Production Database Migration Strategy

A safe migration may follow:

    Step 1
    Add New Schema

        |
        ↓

    Step 2
    Deploy Compatible Application

        |
        ↓

    Step 3
    Start Using New Schema

        |
        ↓

    Step 4
    Validate

        |
        ↓

    Step 5
    Remove Old Schema Later

This reduces deployment risk.

---

# Production Deployment Strategies

Common strategies include:

    Rolling Deployment
    Blue-Green Deployment
    Canary Deployment
    Recreate Deployment

The selected strategy depends on:

    Risk
    Application Architecture
    Traffic
    Availability Requirements
    Infrastructure

---

# Rolling Deployment

Rolling deployment gradually replaces old Pods with new Pods.

Example:

    Old Pods
    Old Pods
    Old Pods

        |
        ↓

    New Pod
    Old Pods
    Old Pods

        |
        ↓

    New Pod
    New Pod
    Old Pod

        |
        ↓

    New Pods
    New Pods
    New Pods

This can provide minimal downtime when configured correctly.

---

# Rolling Deployment Configuration

Important settings include:

    replicas
    maxUnavailable
    maxSurge
    readinessProbe

Example:

    replicas: 3

    strategy:
      type: RollingUpdate

      rollingUpdate:
        maxUnavailable: 0
        maxSurge: 1

This can maintain availability during rollout.

---

# Blue-Green Deployment

Two production environments are maintained:

    Blue = Current
    Green = New

Flow:

    Production Traffic
        |
        ↓
    Blue

    Green
        |
        ↓
    Deploy New Version
        |
        ↓
    Validate
        |
        ↓
    Switch Traffic
        |
        ↓
    Green

If problems occur:

    Traffic
        |
        ↓
    Blue

---

# Blue-Green Advantages

Advantages:

    Fast Rollback
    Full New Environment Testing
    Reduced Deployment Risk
    Clear Version Separation

Challenges:

    Additional Infrastructure
    Additional Cost
    Database Compatibility
    Traffic Switching Complexity

---

# Canary Deployment

Canary deployment sends a small percentage of traffic to the new version.

Example:

    95% Traffic
        |
        ↓
    Version A

    5% Traffic
        |
        ↓
    Version B

Monitor:

    Error Rate
    Latency
    CPU
    Memory
    Business Metrics

If successful:

    90% → A
    10% → B

Then:

    50% → A
    50% → B

Finally:

    0% → A
    100% → B

---

# Canary Deployment Benefits

Canary deployment provides:

    Limited Blast Radius
    Real Traffic Validation
    Early Failure Detection
    Controlled Rollout

It is useful for high-risk releases.

---

# Production Health Checks

After deployment, validate:

    Pod Status
    Readiness
    Liveness
    Startup
    Service
    Ingress
    ALB
    Application Health

Commands:

    kubectl get pods -n production

    kubectl rollout status deployment/payment -n production

---

# Production Smoke Test

Immediately after deployment:

    Application URL
        |
        ↓
    Health Endpoint
        |
        ↓
    Login
        |
        ↓
    Critical API
        |
        ↓
    Basic Business Workflow

Example:

    GET /health

Expected:

    HTTP 200

---

# Production Validation

Validate:

    Application Accessible
    Pods Healthy
    Services Healthy
    ALB Healthy
    APIs Working
    Database Connected
    Message Queue Working
    Critical Business Workflow Working

---

# Production ALB Validation

For AWS ALB:

    Validate Listener
        |
        ↓
    Target Group
        |
        ↓
    Target Health
        |
        ↓
    Health Check
        |
        ↓
    Routing

Expected:

    Targets Healthy

---

# Production DNS Validation

Request path:

    User
      |
      ↓
    Route53
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

Validate:

    DNS
    TLS
    Routing
    Application Response

---

# Production TLS Validation

Verify:

    HTTPS
    Certificate
    Domain
    Certificate Expiration
    TLS Configuration

Example:

    curl -I https://api.example.com

Check:

    HTTP Status
    Certificate
    Redirect
    Headers

---

# Production Kubernetes Deployment

Example:

    kubectl apply -f deployment.yaml -n production

Then:

    kubectl rollout status deployment/payment -n production

Check:

    kubectl get pods -n production

Then:

    Run Smoke Tests

---

# Production Helm Deployment

Example:

    helm upgrade --install payment ./payment-chart \
      --namespace production \
      --create-namespace \
      --set image.tag=1.4.7

Then:

    kubectl get pods -n production

And:

    kubectl rollout status deployment/payment -n production

---

# Production Helm Values

Example:

    values-production.yaml

    environment: production

    image:
      repository: payment
      tag: 1.4.7

    replicas: 3

    resources:
      requests:
        cpu: 500m
        memory: 512Mi

Production values should be protected and reviewed carefully.

---

# Production Deployment Using ArgoCD

GitOps flow:

    Git
      |
      ↓
    Production Manifest
      |
      ↓
    ArgoCD
      |
      ↓
    Production EKS
      |
      ↓
    Kubernetes
      |
      ↓
    Application

ArgoCD continuously reconciles the desired production state from Git.

---

# ArgoCD Production Validation

Check:

    Sync Status
    Application Health
    Deployment
    Pods
    Services
    Ingress

Expected:

    Synced
        +
    Healthy

Do not consider deployment successful only because ArgoCD reports Sync.

Validate application health separately.

---

# Production GitOps Promotion

Typical flow:

    UAT Approval
        |
        ↓
    Production Manifest Update
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
        |
        ↓
    Validation

---

# Production Deployment Approval Gate

A controlled pipeline may contain:

    UAT Approval
        |
        ↓
    Change Approval
        |
        ↓
    Production Deployment
        |
        ↓
    Health Gate
        |
        ↓
    Smoke Test
        |
        ↓
    Monitoring
        |
        ↓
    Success

---

# Production Monitoring

After deployment, monitor:

    CPU
    Memory
    Request Rate
    Error Rate
    Latency
    Pod Restarts
    Availability
    Database
    Message Queue

Tools:

    Prometheus
    Grafana
    ELK

---

# Prometheus Production Monitoring

Prometheus can monitor:

    HTTP Requests
    HTTP Errors
    Latency
    CPU
    Memory
    Pod Health
    Application Metrics

Flow:

    Production
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

# Grafana Production Dashboard

Important panels:

    Application Availability
    Request Rate
    Error Rate
    HTTP 5xx
    Latency
    CPU
    Memory
    Pod Restarts

The dashboard should make production health visible immediately after deployment.

---

# ELK Production Logs

ELK can help investigate:

    Application Errors
    Exceptions
    API Errors
    Database Errors
    Authentication Failures
    Integration Failures

Flow:

    Application
        |
        ↓
    Logs
        |
        ↓
    ELK
        |
        ↓
    Investigation

---

# Production Alerts

Alerts may be configured for:

    High Error Rate
    High Latency
    Pod Failures
    Pod Restarts
    Resource Exhaustion
    Application Unavailability

Example:

    Error Rate
        |
        ↓
    Threshold Exceeded
        |
        ↓
    Alert
        |
        ↓
    Investigation

---

# Production Error Budget

If an organization uses reliability targets, deployment decisions may consider:

    Availability
    Error Rate
    Latency
    Reliability

A release that causes unacceptable reliability degradation may need rollback.

---

# Production Deployment Metrics

Useful metrics:

    Deployment Duration
    Deployment Success Rate
    Deployment Failure Rate
    Rollback Rate
    Change Failure Rate
    Mean Time To Recovery
    Error Rate
    Latency

These metrics help evaluate deployment quality.

---

# Production Deployment and DORA Metrics

Common DORA metrics include:

    Deployment Frequency
    Lead Time For Changes
    Change Failure Rate
    Time To Restore Service

Production deployment processes should aim for:

    Fast Delivery
        +
    High Stability

---

# Production Rollback

If the new version causes a serious issue:

    Detect
      |
      ↓
    Assess
      |
      ↓
    Stop Rollout
      |
      ↓
    Rollback
      |
      ↓
    Validate
      |
      ↓
    Monitor

Kubernetes:

    kubectl rollout undo deployment/payment -n production

Then:

    kubectl rollout status deployment/payment -n production

---

# Helm Rollback

Check:

    helm history payment -n production

Rollback:

    helm rollback payment <revision> -n production

Then:

    kubectl get pods -n production

Validate the application.

---

# GitOps Rollback

With ArgoCD:

    Production Change
        |
        ↓
    Problem
        |
        ↓
    Revert Git Commit
        |
        ↓
    ArgoCD
        |
        ↓
    Previous State
        |
        ↓
    Production
        |
        ↓
    Validate

Git remains the source of truth.

---

# Rollback Decision

Rollback may be required when:

    Error Rate Increases
    Critical API Fails
    Application Unavailable
    Data Corruption Risk
    Severe Performance Degradation
    Critical Business Workflow Fails

Rollback should be based on predefined criteria where possible.

---

# Stop-the-Line Principle

If a critical production issue appears:

    Detect
      |
      ↓
    Stop Further Rollout
      |
      ↓
    Investigate
      |
      ↓
    Rollback / Fix
      |
      ↓
    Validate
      |
      ↓
    Resume

Do not continue deployment simply because the deployment window is still open.

---

# Production Deployment Failure

If deployment fails:

    Deployment
        |
        X
    Failure
        |
        ↓
    Stop
        |
        ↓
    Collect Logs
        |
        ↓
    Investigate
        |
        ↓
    Rollback / Fix
        |
        ↓
    Validate
        |
        ↓
    Incident Management

---

# Production Incident

A production incident may involve:

    Application Failure
    Infrastructure Failure
    Database Failure
    Network Failure
    Security Issue
    Performance Degradation

Incident flow:

    Detection
        |
        ↓
    Triage
        |
        ↓
    Mitigation
        |
        ↓
    Recovery
        |
        ↓
    Validation
        |
        ↓
    Root Cause Analysis

---

# Production Deployment and Incident Management

If a deployment causes an incident:

    Deployment
        |
        ↓
    Incident
        |
        ↓
    Mitigation
        |
        ↓
    Rollback
        |
        ↓
    Service Recovery
        |
        ↓
    Investigation
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Corrective Action

---

# Production Change Freeze

Organizations may use a change freeze during:

    Critical Business Periods
    Peak Traffic
    Major Events
    Financial Period Close
    Holidays

During a freeze:

    Non-Critical Changes
        |
        ↓
    Restricted / Blocked

Emergency changes may follow a separate process.

---

# Emergency Production Deployment

Emergency deployments may be required for:

    Critical Security Issue
    Major Production Failure
    Severe Data Issue
    Critical Business Impact

Typical process:

    Incident
        |
        ↓
    Emergency Change
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Validation
        |
        ↓
    Monitoring

Emergency processes should still maintain appropriate controls.

---

# Production Deployment Security

Important controls:

    Least Privilege
    Secure Credentials
    Secret Management
    Access Control
    Image Scanning
    Code Scanning
    TLS
    Audit Logs
    Approval Gates

Production access should be restricted.

---

# Production Access

Avoid giving every engineer unrestricted production access.

Use:

    Role-Based Access
    Least Privilege
    Temporary Access
    Auditing
    Approval

Example:

    Developer
        |
        ↓
    Limited Production Access

    DevOps
        |
        ↓
    Controlled Deployment Access

---

# Production Separation of Duties

Example:

    Developer
        |
        ↓
    Code

    QA
        |
        ↓
    Validation

    DevOps
        |
        ↓
    Deployment

    Business
        |
        ↓
    Acceptance

    Change Manager
        |
        ↓
    Approval

This reduces risk in enterprise environments.

---

# Production Deployment Auditability

Record:

    Who Deployed
    What Was Deployed
    Version
    Git Commit
    Image
    Environment
    Deployment Time
    Change Request
    Approvals
    Validation Result
    Rollback Result

This supports troubleshooting and compliance.

---

# Production Version Traceability

A strong pipeline provides:

    Git Commit
        |
        ↓
    CI Build
        |
        ↓
    Artifact
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

Example:

    Commit:
    abc123

    Image:
    payment:1.4.7

    Production:
    payment:1.4.7

---

# Production Database Safety

Before database changes:

    Validate Migration
        |
        ↓
    Confirm Backup
        |
        ↓
    Confirm Recovery Strategy
        |
        ↓
    Confirm Compatibility
        |
        ↓
    Deploy
        |
        ↓
    Validate

Avoid destructive database changes without a safe migration strategy.

---

# Backward-Compatible Deployment

For microservices, application versions may temporarily coexist.

Example:

    Version A
        |
        +
        |
    Version B

Both should work with the database and APIs during the transition.

This helps support rolling deployments.

---

# API Compatibility

During production rollout, verify:

    Request Compatibility
    Response Compatibility
    API Version
    Database Compatibility
    Service Compatibility

Example:

    Service A v1
        |
        ↓
    Service B v2

Compatibility must be considered before rollout.

---

# Production Deployment and Microservices

Microservices deployment requires attention to:

    Service Dependencies
    API Compatibility
    Database Changes
    Message Formats
    Authentication
    Network
    Configuration

Example:

    Order
      |
      ↓
    Payment
      |
      ↓
    Inventory

A change in one service can affect others.

---

# Production Deployment Dependency Validation

Before deploying:

    Identify Dependencies
        |
        ↓
    Check Current Versions
        |
        ↓
    Validate Compatibility
        |
        ↓
    Deploy
        |
        ↓
    Monitor

---

# Production Deployment Blast Radius

Blast radius means the amount of the system affected by a failure.

Example:

    1 Service
        |
        ↓
    Small Blast Radius

Compared with:

    Entire Application
        |
        ↓
    Large Blast Radius

Canary and gradual rollouts reduce blast radius.

---

# Production Canary Validation

During canary:

    Small Traffic
        |
        ↓
    New Version
        |
        ↓
    Monitor
        |
        +-- Errors High → Rollback
        |
        +-- Healthy → Increase Traffic

Validate:

    Error Rate
    Latency
    Resource Usage
    Business Metrics

---

# Production Blue-Green Validation

Flow:

    Blue
    Current Version

    Green
    New Version

        |
        ↓

    Deploy Green
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Business Validation
        |
        ↓
    Switch Traffic

---

# Production Rolling Validation

During rolling deployment:

    New Pod
        |
        ↓
    Readiness Check
        |
        ↓
    Receive Traffic
        |
        ↓
    Replace Old Pod
        |
        ↓
    Repeat

Readiness checks are critical.

---

# Production Health Gate

A production pipeline can use:

    Deployment
        |
        ↓
    Health Check
        |
        ↓
    Error Rate
        |
        ↓
    Latency
        |
        ↓
    Smoke Test
        |
        ↓
    Business Validation
        |
        ↓
    Continue / Rollback

---

# Production Smoke Tests

Typical smoke tests:

    DNS
    HTTPS
    Login
    Health Endpoint
    Critical API
    Database Connectivity
    Basic Business Workflow

Example:

    User Login
        |
        ↓
    Product Search
        |
        ↓
    Basic Order
        |
        ↓
    Confirmation

Only the most critical flows should be used for fast smoke validation.

---

# Production Post-Deployment Monitoring

After deployment:

    Deployment
        |
        ↓
    Monitor
        |
        ↓
    Compare
        |
        ↓
    Baseline

Compare:

    Before Deployment

with:

    After Deployment

Check:

    Error Rate
    Latency
    Traffic
    Resource Usage
    Business Metrics

---

# Production Baseline

Before deployment, know the normal state.

Example:

    HTTP 5xx:
    Normal

    Latency:
    Normal

    CPU:
    Normal

After deployment:

    HTTP 5xx:
    Increased

    Latency:
    Increased

This indicates a possible regression.

---

# Production Deployment Observation Period

After deployment, teams may observe the system for a defined period.

During observation:

    Monitor Metrics
    Review Logs
    Check Alerts
    Validate Business Flow
    Watch User Impact

The exact observation period depends on the organization's process.

---

# Production Success Criteria

Deployment can be considered successful when:

    Rollout Successful
    Pods Healthy
    Health Checks Pass
    Smoke Tests Pass
    Error Rate Normal
    Latency Normal
    No Critical Alerts
    Critical Business Workflow Works
    No Significant User Impact

---

# Production Deployment Failure Criteria

Deployment may be considered failed when:

    Rollout Fails
    Pods Crash
    Health Checks Fail
    HTTP 5xx Increases
    Latency Increases
    Critical API Fails
    Business Workflow Fails
    Data Integrity Is At Risk
    Critical Alerts Trigger

---

# Production Rollback Criteria

Example criteria:

    Critical API Failure
    Severe Error Rate Increase
    Major Performance Degradation
    Data Corruption Risk
    Security Vulnerability
    Major Business Impact

Flow:

    Failure
        |
        ↓
    Stop Rollout
        |
        ↓
    Rollback
        |
        ↓
    Validate
        |
        ↓
    Recover

---

# Production Deployment Communication

Teams may communicate:

    Deployment Start
    Deployment Progress
    Deployment Success
    Deployment Failure
    Rollback
    Incident
    Recovery

The exact communication channel depends on organizational tooling.

---

# Production Deployment Checklist

    UAT Approved
        |
        ↓
    Business Sign-Off
        |
        ↓
    Change Approved
        |
        ↓
    Artifact Verified
        |
        ↓
    Git Commit Verified
        |
        ↓
    Production Configuration Verified
        |
        ↓
    Secrets Verified
        |
        ↓
    Database Plan Ready
        |
        ↓
    Rollback Plan Ready
        |
        ↓
    Monitoring Ready
        |
        ↓
    Alerts Ready
        |
        ↓
    Deployment Window Confirmed
        |
        ↓
    Production Deployment
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
    ALB Healthy
        |
        ↓
    Smoke Tests Passed
        |
        ↓
    Metrics Normal
        |
        ↓
    Logs Reviewed
        |
        ↓
    Business Validation
        |
        ↓
    Deployment Successful

---

# Real-World Production Deployment Example

Suppose:

    Application:
    Payment

    Version:
    1.4.7

The release has passed:

    QA
    SIT
    UAT

Production process:

    UAT Sign-Off
        |
        ↓
    Change Approval
        |
        ↓
    Pre-Checks
        |
        ↓
    Deploy payment:1.4.7
        |
        ↓
    EKS
        |
        ↓
    Rolling Update
        |
        ↓
    Readiness Checks
        |
        ↓
    Smoke Tests
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
    Business Validation
        |
        ↓
    Successful Deployment

---

# Real-World Production Failure

Version:

    payment:1.4.7

Deployment:

    Successful

Pods:

    Running

But:

    HTTP 5xx Increased

Investigation:

    Grafana
        |
        ↓
    Error Rate Increased
        |
        ↓
    ELK
        |
        ↓
    Application Exception
        |
        ↓
    Root Cause Identified

If impact is severe:

    Stop Rollout
        |
        ↓
    Rollback
        |
        ↓
    Validate
        |
        ↓
    Recover

---

# Real-World Production Rollback

Before:

    payment:1.4.6

New:

    payment:1.4.7

After deployment:

    Error Rate ↑
    Latency ↑

Action:

    kubectl rollout undo deployment/payment -n production

Then:

    kubectl rollout status deployment/payment -n production

Validate:

    Error Rate
    Latency
    Health
    Business Workflow

Expected:

    payment:1.4.6
        |
        ↓
    Stable Production

---

# Real-World Canary Deployment

Current:

    payment:1.4.6

New:

    payment:1.4.7

Initial traffic:

    95% → 1.4.6
     5% → 1.4.7

Monitor:

    Error Rate
    Latency
    Business Metrics

If healthy:

    90% → 1.4.6
    10% → 1.4.7

Then:

    50% → 1.4.6
    50% → 1.4.7

Finally:

    100% → 1.4.7

---

# Real-World Blue-Green Deployment

Current:

    Blue
    payment:1.4.6

New:

    Green
    payment:1.4.7

Flow:

    Deploy Green
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Validate
        |
        ↓
    Switch Traffic
        |
        ↓
    Green Live

If failure:

    Switch Traffic
        |
        ↓
    Blue

---

# Real-World Database Deployment

Suppose version 1.4.7 requires a new column.

Safe approach:

    Add New Column
        |
        ↓
    Deploy Compatible Version
        |
        ↓
    Validate
        |
        ↓
    Start Using Column
        |
        ↓
    Monitor

Avoid deploying an application that immediately requires a schema change that older running Pods cannot support during a rolling deployment.

---

# Production Deployment Interview Questions

## Basic

1. What is production deployment?

2. What is the purpose of production deployment?

3. What checks do you perform before production deployment?

4. What is a deployment window?

5. What is a change request?

6. What is a rollback?

7. What is a smoke test?

8. What is a health check?

9. What is production readiness?

10. What happens after production deployment?

---

# Production Deployment Interview Questions

## Intermediate

11. How do you deploy an application to production?

12. How do you deploy a Docker application to EKS?

13. How do you validate a Kubernetes deployment?

14. How do you perform a rolling deployment?

15. How do you rollback a Kubernetes deployment?

16. How do you manage production configuration?

17. How do you manage production secrets?

18. How do you monitor a production deployment?

19. How do you validate an ALB after deployment?

20. How do you troubleshoot a production deployment failure?

---

# Production Deployment Interview Questions

## Advanced

21. How would you design a zero-downtime production deployment?

22. How would you implement blue-green deployment?

23. How would you implement canary deployment?

24. How would you design a production rollback strategy?

25. How would you deploy database changes safely?

26. How would you implement production deployment using GitHub Actions?

27. How would you implement production deployment using ArgoCD?

28. How would you implement approval gates?

29. How would you maintain artifact traceability?

30. How would you reduce production deployment risk?

31. How would you handle a deployment that succeeds but application errors increase?

32. How would you determine whether to rollback?

33. How would you design production monitoring?

34. How would you handle a critical production deployment failure?

---

# Scenario-Based Interview Question

## Deployment Succeeds but Users Receive 503

Check:

    Pods
    Readiness
    Service
    Endpoints
    Ingress
    ALB
    Application Logs

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

Find the failing layer.

---

# Scenario-Based Interview Question

## Pods Are Healthy but Error Rate Increased

Healthy Pods do not guarantee healthy applications.

Check:

    Application Logs
    HTTP 5xx
    Database
    External APIs
    Service Dependencies
    Business Workflow

Use:

    Prometheus
    Grafana
    ELK

---

# Scenario-Based Interview Question

## New Version Has High Latency

Compare:

    Before Deployment
        |
        ↓
    Baseline

with:

    After Deployment
        |
        ↓
    Current Metrics

Check:

    Application
    Database
    External API
    CPU
    Memory
    Network

If impact is severe:

    Rollback

---

# Scenario-Based Interview Question

## Database Migration Failed During Production Deployment

Actions:

    Stop Deployment
        |
        ↓
    Determine Database State
        |
        ↓
    Check Application Compatibility
        |
        ↓
    Assess Rollback Safety
        |
        ↓
    Recover / Fix
        |
        ↓
    Validate
        |
        ↓
    Resume Carefully

Do not blindly rollback database changes.

---

# Scenario-Based Interview Question

## Production Deployment Must Have Zero Downtime

Possible approach:

    Multiple Replicas
        |
        ↓
    Readiness Probes
        |
        ↓
    Rolling Update
        |
        ↓
    maxUnavailable: 0
        |
        ↓
    maxSurge
        |
        ↓
    Gradual Replacement

Also verify:

    Database Compatibility
    API Compatibility
    Load Balancer
    Health Checks

---

# Scenario-Based Interview Question

## How Would You Minimize Blast Radius?

Use:

    Canary Deployment
    Gradual Rollout
    Feature Flags
    Small Initial Traffic
    Strong Monitoring
    Automated Rollback

Flow:

    Small Change
        |
        ↓
    Small Traffic
        |
        ↓
    Validate
        |
        ↓
    Increase
        |
        ↓
    Full Deployment

---

# Scenario-Based Interview Question

## Production Deployment Fails Halfway

Example:

    3 Replicas

Deployment:

    Replica 1 → New
    Replica 2 → New
    Replica 3 → Old

Action:

    Check Rollout Status
        |
        ↓
    Check Events
        |
        ↓
    Check Logs
        |
        ↓
    Check Readiness
        |
        ↓
    Determine Root Cause
        |
        ↓
    Rollback If Required

Kubernetes maintains the deployment state according to its rollout strategy.

---

# Scenario-Based Interview Question

## Production Deployment Causes Business Failure

Example:

    Application Healthy
        |
        ↓
    Payment Successful
        |
        ↓
    Order Status Incorrect

This is a business-impacting production issue.

Action:

    Assess Impact
        |
        ↓
    Stop Further Rollout
        |
        ↓
    Mitigate
        |
        ↓
    Rollback / Fix
        |
        ↓
    Validate
        |
        ↓
    Incident Review

---

# Scenario-Based Interview Question

## New Deployment Works but Old Pods Fail

Possible cause:

    Database Schema Change
    API Compatibility
    Configuration
    Shared Dependency

During rolling deployment:

    Old Version
        +
    New Version

may coexist.

Therefore, compatibility must be considered.

---

# Production Deployment Best Practices

- Build once and promote
- Use immutable artifacts
- Verify the exact production image
- Use controlled approvals
- Use change management
- Maintain a rollback plan
- Use readiness probes
- Use health checks
- Use gradual deployment strategies
- Monitor during and after deployment
- Validate critical business workflows
- Protect production secrets
- Use least privilege
- Maintain auditability
- Keep database changes backward compatible where possible
- Minimize blast radius
- Automate repeatable validation
- Keep production configuration separate
- Maintain clear incident procedures
- Record deployment evidence

---

# Production Deployment Anti-Patterns

## Deploying Untested Code

Bad:

    Development
        |
        ↓
    Production

Better:

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

---

# Production Deployment Anti-Pattern

## Manual Production Deployment

Bad:

    Developer Laptop
        |
        ↓
    Manual Commands
        |
        ↓
    Production

Problems:

    Human Error
    No Traceability
    Inconsistent Process
    Difficult Rollback

Better:

    Git
      |
      ↓
    CI/CD
      |
      ↓
    Controlled Production Deployment

---

# Production Deployment Anti-Pattern

## Using Latest Image

Avoid:

    image: payment:latest

Prefer:

    image: payment:1.4.7

or an immutable image digest.

---

# Production Deployment Anti-Pattern

## No Rollback Plan

Never start a high-risk production deployment without knowing:

    What To Roll Back
    How To Roll Back
    Who Can Roll Back
    How To Validate Recovery

---

# Production Deployment Anti-Pattern

## Skipping Monitoring

Bad:

    Deploy
      |
      ↓
    Done

Better:

    Deploy
      |
      ↓
    Monitor
      |
      ↓
    Validate
      |
      ↓
    Confirm Success

---

# Production Deployment Anti-Pattern

## Changing Application and Database at the Same Time Without Compatibility

Bad:

    Database Breaking Change
        +
    New Application
        |
        ↓
    Production

If deployment fails, rollback becomes difficult.

Prefer backward-compatible migration strategies.

---

# Production Deployment Anti-Pattern

## Deploying During an Active Incident

If production is already unstable:

    Active Incident
        |
        ↓
    Avoid Unrelated Deployment

Unless the deployment is part of the approved mitigation.

---

# Production Deployment Anti-Pattern

## Ignoring Error Rate

Even if:

    Pods = Healthy

if:

    HTTP 5xx = Increased

then:

    Production Health = Degraded

Application-level metrics matter.

---

# Production Deployment Anti-Pattern

## No Business Validation

Technical checks are not enough.

Validate:

    Critical Business Workflow

Example:

    Login
      |
      ↓
    Order
      |
      ↓
    Payment
      |
      ↓
    Confirmation

---

# Production Deployment Checklist

## Before Deployment

    UAT Approved
    Business Sign-Off
    Change Approved
    Artifact Verified
    Git Commit Verified
    Image Verified
    Configuration Verified
    Secrets Verified
    Database Plan Ready
    Rollback Plan Ready
    Monitoring Ready
    Alerts Ready
    Access Verified
    Deployment Window Confirmed

## During Deployment

    Rollout Status
    Pod Health
    Readiness
    Error Rate
    Latency
    Logs
    Resource Usage

## After Deployment

    Health Checks
    Smoke Tests
    Business Workflow
    Metrics
    Logs
    Alerts
    User Impact
    Deployment Evidence

---

# Complete Enterprise Production Flow

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
    Container Image
        |
        ↓
    ECR
        |
        ↓
    QA
        |
        ↓
    QA Approval
        |
        ↓
    SIT
        |
        ↓
    SIT Approval
        |
        ↓
    UAT
        |
        ↓
    Business Sign-Off
        |
        ↓
    Change Approval
        |
        ↓
    Production
        |
        ↓
    EKS
        |
        +-- Deployment
        +-- Service
        +-- Ingress
        |
        ↓
    ALB
        |
        ↓
    Real Users
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
    Post-Deployment Validation
        |
        ↓
    Successful Release

---

# Final Production Mental Model

Remember:

    QA
      |
      ↓
    Does the application work?

    SIT
      |
      ↓
    Do all systems work together?

    UAT
      |
      ↓
    Does the business accept it?

    Production
      |
      ↓
    Can we safely operate it for real users?

The production flow is:

    APPROVE
        |
        ↓
    PRE-CHECK
        |
        ↓
    DEPLOY
        |
        ↓
    VERIFY
        |
        ↓
    MONITOR
        |
        ↓
    VALIDATE
        |
        ↓
    CONTINUE
        |
        OR
        ↓
    ROLLBACK

---

# Final Concept

Production deployment is not simply:

    kubectl apply

It is a controlled release process involving:

    Tested Artifact
        +
    Configuration
        +
    Change Approval
        +
    Deployment Strategy
        +
    Health Checks
        +
    Monitoring
        +
    Validation
        +
    Rollback
        +
    Incident Management

The key principle is:

    Build Once
        |
        ↓
    Test Thoroughly
        |
        ↓
    Approve
        |
        ↓
    Deploy Safely
        |
        ↓
    Monitor
        |
        ↓
    Validate
        |
        ↓
    Recover Quickly If Needed

A successful production deployment means:

    Approved Release
        +
    Correct Artifact
        +
    Safe Deployment
        +
    Healthy Infrastructure
        +
    Healthy Application
        +
    Normal Metrics
        +
    Successful Smoke Tests
        +
    Successful Business Workflow
        +
    No Critical User Impact

Therefore:

    Successful Production Deployment
        =
    Release Control
        +
    Automation
        +
    Observability
        +
    Reliability
        +
    Security
        +
    Fast Recovery

The next enterprise topic is:

    Production Deployment
        |
        ↓
    JIRA Change Request