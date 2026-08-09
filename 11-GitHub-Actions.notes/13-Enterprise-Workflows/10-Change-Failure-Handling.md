# Change Failure Handling

Change Failure Handling is the process of identifying, containing, recovering from, and learning from a failed production change.

A change can fail because of:

    Application Bug
    Configuration Error
    Infrastructure Failure
    Database Problem
    Dependency Failure
    Deployment Error
    Security Issue
    Environment Problem

A typical enterprise flow is:

    Change
        |
        ↓
    Deployment
        |
        ↓
    Failure Detected
        |
        ↓
    Assess Impact
        |
        ↓
    Contain
        |
        ↓
    Rollback / Fix
        |
        ↓
    Validate
        |
        ↓
    Monitor
        |
        ↓
    Incident / Change Update
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Corrective Actions
        |
        ↓
    Change Closure

---

# What Is Change Failure?

A change failure occurs when an implemented change causes an unexpected problem or does not achieve the intended result.

Examples:

    Application Unavailable
    HTTP 5xx Errors
    Increased Latency
    Failed Transactions
    Database Errors
    Pod Failures
    Configuration Problems
    Security Problems
    Integration Failures

Example:

    Release v1.4.7
        |
        ↓
    Production
        |
        ↓
    Error Rate Increased
        |
        ↓
    Change Failure

---

# Purpose of Change Failure Handling

The purpose is to:

    Protect Users
    Restore Service
    Reduce Business Impact
    Identify Root Cause
    Prevent Recurrence
    Maintain Auditability

The key principle is:

    Detect Quickly
        +
    Contain Quickly
        +
    Recover Safely
        +
    Learn From Failure

---

# Change Failure Lifecycle

A mature process is:

    Detect
        |
        ↓
    Assess
        |
        ↓
    Communicate
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
    Monitor
        |
        ↓
    Investigate
        |
        ↓
    RCA
        |
        ↓
    Corrective Action
        |
        ↓
    Close

---

# Change Failure vs Incident

A change failure is related to a change that caused an unexpected result.

An incident is an interruption or degradation of service.

Example:

    Production Change
        |
        ↓
    Payment Service
        |
        ↓
    HTTP 500
        |
        ↓
    Customer Impact
        |
        ↓
    Incident

The change may be the cause of the incident.

---

# Change Failure vs Defect

A defect is a problem in software or configuration.

Example:

    Code Bug
        |
        ↓
    Deployment
        |
        ↓
    Production Failure

The defect caused the change failure.

---

# Change Failure vs Rollback

Change failure:

    Something went wrong

Rollback:

    Recovery action that returns the system
    to a previous known-good state

Example:

    v1.4.7
        |
        ↓
    Failure
        |
        ↓
    Rollback
        |
        ↓
    v1.4.6

Rollback is one possible response to a change failure.

---

# Detecting Change Failure

Failures can be detected through:

    Monitoring
    Alerts
    Health Checks
    Smoke Tests
    E2E Tests
    User Reports
    Application Logs
    Infrastructure Metrics
    Business Metrics

---

# Monitoring-Based Detection

Example:

    Deployment
        |
        ↓
    Prometheus
        |
        ↓
    Error Rate Increased
        |
        ↓
    Alert
        |
        ↓
    Investigation

---

# Grafana-Based Detection

Grafana can show:

    Error Rate
    Latency
    Request Rate
    CPU
    Memory
    Pod Restarts

Example:

    Before Release
        |
        ↓
    Error Rate = Normal

    After Release
        |
        ↓
    Error Rate = Increased

Possible:

    Change Failure

---

# ELK-Based Detection

ELK can help identify:

    Application Errors
    Exceptions
    HTTP 5xx
    Database Errors
    Connection Errors
    Authentication Errors

Flow:

    Deployment
        |
        ↓
    Logs
        |
        ↓
    ELK
        |
        ↓
    Error Pattern
        |
        ↓
    Investigation

---

# Health Check Detection

Example:

    Deployment
        |
        ↓
    Readiness Probe
        |
        X
    Failed
        |
        ↓
    Pod Not Ready

This can indicate a deployment problem.

---

# Smoke Test Detection

Example:

    Deployment
        |
        ↓
    Smoke Test
        |
        X
    Payment API Failed
        |
        ↓
    Change Failure

---

# E2E Detection

Example:

    Production Deployment
        |
        ↓
    Login
        |
        ↓
    Order
        |
        ↓
    Payment
        |
        X
    Failed
        |
        ↓
    Change Failure

Business workflow failures can be more important than infrastructure health alone.

---

# Change Failure Detection Timeline

Example:

    10:00 PM
        |
        ↓
    Deployment Started

    10:10 PM
        |
        ↓
    Error Rate Increased

    10:12 PM
        |
        ↓
    Alert Triggered

    10:15 PM
        |
        ↓
    Investigation Started

    10:20 PM
        |
        ↓
    Rollback Started

    10:30 PM
        |
        ↓
    Service Recovered

Fast detection reduces impact.

---

# First Response

When a change failure is detected:

    Stop
        |
        ↓
    Assess
        |
        ↓
    Contain
        |
        ↓
    Recover

Do not immediately start making unrelated changes.

---

# Step 1: Stop Further Rollout

If a progressive deployment is running:

    v1.4.7
        |
        ↓
    10% Traffic
        |
        ↓
    Failure
        |
        ↓
    STOP

Do not continue:

    10%
        |
        ↓
    25%
        |
        ↓
    50%
        |
        ↓
    100%

if the release is already showing serious problems.

---

# Step 2: Assess Impact

Determine:

    Which Service?
    Which Environment?
    Which Users?
    How Many Users?
    What Functionality?
    What Severity?
    What Dependencies?
    Is Data Affected?

---

# Impact Assessment

Example:

    Payment Service
        |
        ↓
    Error Rate = 20%
        |
        ↓
    Payment Failures
        |
        ↓
    Customer Impact = High

Possible action:

    Immediate Rollback

---

# Impact Categories

Possible impact:

    No User Impact
    Low Impact
    Moderate Impact
    High Impact
    Critical Impact

The exact classification depends on organizational policy.

---

# Change Failure Severity

Example:

    Low
        |
        ↓
    Internal Function Affected

    Medium
        |
        ↓
    Partial User Impact

    High
        |
        ↓
    Major Business Function Affected

    Critical
        |
        ↓
    Major Service Outage

---

# Step 3: Communicate

During a significant production failure:

    Notify Stakeholders
        |
        ↓
    Application Team
        |
        ↓
    DevOps
        |
        ↓
    Support
        |
        ↓
    Business
        |
        ↓
    Incident Management

Communication should be concise and factual.

---

# Change Failure Communication

Include:

    Change ID
    Application
    Version
    Environment
    Failure
    Impact
    Current Action
    Owner

Example:

    Change CHG-10245

    Payment Service v1.4.7 has experienced
    increased error rates after deployment.

    Rollback is in progress.

---

# Step 4: Containment

Containment prevents the problem from getting worse.

Examples:

    Stop Deployment
    Stop Traffic Increase
    Disable Feature
    Isolate Service
    Rollback
    Route Traffic Away
    Disable Problematic Integration

---

# Feature Flag Containment

If a feature flag exists:

    New Feature
        |
        ↓
    Failure
        |
        ↓
    Disable Feature
        |
        ↓
    Existing Functionality

This can reduce impact without rolling back the entire application.

---

# Canary Containment

Example:

    v1.4.7
        |
        ↓
    5% Traffic
        |
        ↓
    Failure
        |
        ↓
    Stop Promotion
        |
        ↓
    Return Traffic
        |
        ↓
    v1.4.6

---

# Blue-Green Containment

Example:

    Blue
    v1.4.6
        |
        ↓
    Stable

    Green
    v1.4.7
        |
        ↓
    Failure

Action:

    Keep Traffic On Blue
        |
        ↓
    Investigate Green

---

# Rolling Deployment Containment

Example:

    Old Pods
        |
        ↓
    New Pods
        |
        ↓
    Failure

Action:

    Stop Rollout
        |
        ↓
    Keep Healthy Pods
        |
        ↓
    Rollback

---

# Step 5: Rollback Decision

Rollback may be appropriate when:

    Failure Is Severe
    Root Cause Is Known Or Recovery Is Faster
    Previous Version Is Known-Good
    Rollback Is Safe
    User Impact Is High

---

# When Not To Rollback Immediately

Rollback may not always be the best choice.

Examples:

    Database Schema Is Not Backward Compatible
    Data Migration Already Completed
    Rollback Could Cause More Damage
    Failure Is External
    Rollback Does Not Address Root Cause

In these cases:

    Assess
        |
        ↓
    Mitigate
        |
        ↓
    Fix Forward If Appropriate

---

# Rollback vs Fix Forward

Rollback:

    Current Version
        |
        ↓
    Failure
        |
        ↓
    Previous Version

Fix Forward:

    Current Version
        |
        ↓
    Failure
        |
        ↓
    Hotfix
        |
        ↓
    New Version
        |
        ↓
    Deploy

---

# Rollback Strategy

A rollback should be:

    Planned
    Tested
    Fast
    Repeatable
    Observable

---

# Kubernetes Rollback

Example:

    Deployment
        |
        ↓
    Failure
        |
        ↓
    kubectl rollout undo

Then:

    kubectl rollout status deployment/payment -n production

Validate:

    kubectl get pods -n production

---

# Kubernetes Rollout History

Check history:

    kubectl rollout history deployment/payment -n production

This helps identify previous revisions.

---

# Kubernetes Rollback Validation

After rollback:

    Pods Healthy
        |
        ↓
    Readiness Passed
        |
        ↓
    Service Healthy
        |
        ↓
    Smoke Test
        |
        ↓
    Monitoring Stable

---

# Helm Rollback

Check:

    helm history payment -n production

Rollback:

    helm rollback payment <revision> -n production

Validate:

    kubectl get pods -n production

    kubectl rollout status deployment/payment -n production

---

# GitOps Rollback

For ArgoCD:

    Production
        |
        ↓
    Failure
        |
        ↓
    Revert Git Commit
        |
        ↓
    ArgoCD
        |
        ↓
    Previous Desired State
        |
        ↓
    Kubernetes
        |
        ↓
    Validation

---

# Docker Image Rollback

Example:

    Current:
    payment:1.4.7

    Previous:
    payment:1.4.6

Rollback:

    payment:1.4.6

Then:

    Validate
        |
        ↓
    Monitor

---

# Database Change Failure

Database failures require special care.

Example:

    Application Release
        |
        ↓
    Database Migration
        |
        X
    Failure

Do not blindly rollback database changes.

Assess:

    Data State
    Migration State
    Compatibility
    Backup
    Recovery
    Application Dependencies

---

# Database Rollback

Possible strategies:

    Reverse Migration
    Restore Backup
    Fix Forward
    Restore Specific Data

The correct method depends on the migration and data state.

---

# Expand-and-Contract Strategy

For safer database changes:

    Expand
        |
        ↓
    Add New Structure
        |
        ↓
    Deploy Compatible Application
        |
        ↓
    Migrate Data
        |
        ↓
    Switch Application
        |
        ↓
    Contract
        |
        ↓
    Remove Old Structure

This can reduce rollback complexity.

---

# Configuration Failure

Example:

    Application
        |
        ↓
    New Configuration
        |
        X
    Invalid Value
        |
        ↓
    Application Failure

Recovery:

    Restore Previous Configuration
        |
        ↓
    Restart / Reload
        |
        ↓
    Validate

---

# Infrastructure Change Failure

Example:

    Terraform
        |
        ↓
    Production Change
        |
        X
    Resource Failure

First:

    Assess Terraform State

Then:

    Review Plan
        |
        ↓
    Identify Partial Changes
        |
        ↓
    Recover Safely

Do not blindly run commands without understanding the current infrastructure state.

---

# Terraform Failure Handling

Example:

    terraform apply
        |
        X
    Failure
        |
        ↓
    Partial Infrastructure
        |
        ↓
    Assess State
        |
        ↓
    terraform plan
        |
        ↓
    Determine Difference
        |
        ↓
    Correct Safely

---

# Terraform State

After a failed apply:

    Check State
        |
        ↓
    Check Real Infrastructure
        |
        ↓
    Run terraform plan
        |
        ↓
    Understand Drift
        |
        ↓
    Recover

The Terraform state should not be manually deleted as a first response.

---

# Partial Deployment

A deployment may partially succeed.

Example:

    Service A
        |
        ✓
    Service B
        |
        ✓
    Service C
        |
        X
    Service D
        |
        X

Do not assume the whole release succeeded or failed.

Assess each component.

---

# Partial Release Recovery

Flow:

    Partial Deployment
        |
        ↓
    Identify Successful Components
        |
        ↓
    Identify Failed Components
        |
        ↓
    Assess Dependencies
        |
        ↓
    Rollback / Continue / Fix
        |
        ↓
    Validate Complete System

---

# Dependency Failure

Example:

    Payment Service
        |
        ↓
    External Gateway
        |
        X
    Unavailable

The application change may be correct.

Possible response:

    Circuit Breaker
        |
        ↓
    Retry
        |
        ↓
    Fallback
        |
        ↓
    Rollback If Appropriate

---

# External Dependency Failure

Before rolling back the application, determine:

    Did The Application Cause The Failure?

If:

    External Gateway
        |
        X
    Down

and:

    Application
        |
        ✓
    Healthy

then rollback may not solve the problem.

---

# Service Dependency Failure

Example:

    Order
        |
        ↓
    Payment
        |
        ↓
    Payment Failure

Check:

    Payment Service
    Network
    Database
    External Gateway

Trace the dependency chain before selecting recovery.

---

# Change Failure Root Cause

Common root causes:

    Incorrect Code
    Incorrect Configuration
    Missing Environment Variable
    Incorrect Secret
    Database Migration
    Resource Limit
    Dependency Failure
    Network Configuration
    Kubernetes Configuration
    Infrastructure Change

---

# Root Cause Analysis

After service recovery:

    Failure
        |
        ↓
    Evidence
        |
        ↓
    Timeline
        |
        ↓
    Investigation
        |
        ↓
    Root Cause
        |
        ↓
    Corrective Action

---

# RCA Questions

Ask:

    What Changed?

    When Did It Change?

    What Failed?

    When Did Failure Start?

    Which Users Were Affected?

    What Was The First Error?

    What Dependencies Were Involved?

    Why Did Existing Tests Not Detect It?

    Why Did Monitoring Not Prevent It?

    How Can We Prevent Recurrence?

---

# Five Whys

Example:

    Why did payment fail?

    Because Payment Service could not connect
    to the database.

    Why could it not connect?

    Database security rules were incorrect.

    Why were the rules incorrect?

    Terraform configuration changed them.

    Why was the change not detected?

    The validation did not cover the required
    connectivity scenario.

    Why was the validation missing?

    The deployment checklist did not include
    that dependency test.

Root cause:

    Missing Infrastructure Connectivity Validation

---

# Blameless RCA

A mature organization focuses on:

    System
    Process
    Technology
    Controls

instead of:

    Blaming Individuals

The goal is:

    Learn
        |
        ↓
    Improve
        |
        ↓
    Prevent Recurrence

---

# Timeline

A good RCA includes a timeline.

Example:

    10:00
    Deployment Started

    10:08
    New Pods Ready

    10:10
    Error Rate Increased

    10:12
    Alert Triggered

    10:15
    Incident Declared

    10:20
    Rollback Started

    10:28
    Previous Version Healthy

    10:35
    Service Validated

---

# Impact Assessment

Document:

    Duration
    Users Affected
    Services Affected
    Requests Failed
    Business Impact
    Data Impact
    Revenue Impact If Applicable

---

# Change Failure Metrics

Track:

    Change Failure Rate
    Rollback Rate
    Failed Deployment Rate
    Incident Count
    Recovery Time
    Mean Time To Recovery
    Repeat Failure Rate

---

# Change Failure Rate

Change Failure Rate measures the percentage of deployments that result in a production failure requiring remediation.

Example:

    Total Deployments:
    100

    Failed Changes:
    5

    Change Failure Rate:
    5%

A lower rate generally indicates more reliable change delivery.

---

# Recovery Time

Example:

    Failure Detected:
    10:12 PM

    Service Restored:
    10:28 PM

    Recovery Time:
    16 Minutes

Track this over time.

---

# Mean Time To Restore

MTTR measures how quickly service is restored after an incident.

Example:

    Incident 1 → 20 minutes
    Incident 2 → 10 minutes
    Incident 3 → 30 minutes

Average recovery time:

    20 minutes

The exact metric definition can vary by organization.

---

# Detection Time

Example:

    Deployment:
    10:00

    Failure:
    10:05

    Alert:
    10:06

Detection time:

    1 minute after failure

Fast detection reduces impact.

---

# Recovery Time Optimization

Improve recovery by:

    Automated Rollback
    Good Monitoring
    Clear Runbooks
    Tested Recovery
    Immutable Artifacts
    Health Checks
    Safe Deployment Strategies

---

# Change Failure Prevention

Prevent failures through:

    Code Review
    Unit Tests
    Integration Tests
    E2E Tests
    SonarQube
    Trivy
    Veracode
    Infrastructure Validation
    Configuration Validation
    Canary Releases
    Automated Health Checks

---

# Shift Left

Find problems earlier:

    Development
        |
        ↓
    Unit Test
        |
        ↓
    Security Scan
        |
        ↓
    Integration
        |
        ↓
    E2E
        |
        ↓
    Production

The earlier a problem is detected, the lower the potential impact.

---

# Change Failure Prevention in CI

Pipeline:

    Commit
        |
        ↓
    Build
        |
        ↓
    Unit Test
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Integration Test
        |
        ↓
    E2E
        |
        ↓
    Release

---

# Change Failure Prevention in CD

Deployment:

    Approved Artifact
        |
        ↓
    Canary
        |
        ↓
    Health Check
        |
        ↓
    Monitor
        |
        ↓
    Increase Traffic
        |
        ↓
    Full Deployment

If failure:

    Stop
        |
        ↓
    Rollback

---

# Progressive Delivery

Progressive delivery reduces blast radius.

Examples:

    Canary
    Blue-Green
    Rolling Deployment
    Feature Flags

---

# Blast Radius

Blast radius means the scope of impact caused by a failure.

Example:

    Deploy To 100%
        |
        ↓
    Failure
        |
        ↓
    Large Impact

Versus:

    Deploy To 5%
        |
        ↓
    Failure
        |
        ↓
    Limited Impact

---

# Canary Change Failure

Example:

    v1.4.7
        |
        ↓
    5% Traffic
        |
        ↓
    Error Rate Increased
        |
        ↓
    Stop
        |
        ↓
    Rollback
        |
        ↓
    95% Traffic Remains On v1.4.6

---

# Blue-Green Change Failure

Example:

    Blue
    v1.4.6
        |
        ↓
    Stable

    Green
    v1.4.7
        |
        ↓
    Failure

Action:

    Keep Traffic On Blue
        |
        ↓
    Investigate Green

---

# Rolling Change Failure

Example:

    Pod 1
        |
        ✓

    Pod 2
        |
        X

Stop:

    Rollout

Then:

    Restore Previous Version

---

# Feature Flag Change Failure

Example:

    Code
        |
        ↓
    Deploy
        |
        ↓
    Feature Enabled
        |
        X
    Failure

Action:

    Disable Feature
        |
        ↓
    Existing Application Continues

---

# Change Failure and Observability

A good system should answer:

    What Failed?

    Where Did It Fail?

    When Did It Fail?

    How Many Users Are Affected?

    Which Version Is Running?

Use:

    Prometheus
    Grafana
    ELK

---

# Version Identification

During a failure, identify:

    Application
    Version
    Git SHA
    Image Tag
    Deployment
    Environment

Example:

    Payment
    v1.4.7
    Git SHA: abc123
    ECR Image: payment:1.4.7
    Environment: Production

---

# Kubernetes Change Failure Investigation

Check:

    kubectl get pods -n production

    kubectl describe pod <pod> -n production

    kubectl logs <pod> -n production

    kubectl rollout status deployment/payment -n production

    kubectl get events -n production

---

# Kubernetes Events

Events can reveal:

    Image Pull Failure
    Scheduling Failure
    Probe Failure
    Resource Problems
    Mount Errors
    Configuration Errors

Example:

    kubectl get events -n production --sort-by=.lastTimestamp

---

# Pod Logs

Check:

    kubectl logs <pod> -n production

For previous crashed container:

    kubectl logs <pod> --previous -n production

Look for:

    Exceptions
    Connection Errors
    Configuration Errors
    Startup Errors

---

# Deployment Status

Check:

    kubectl rollout status deployment/payment -n production

If rollout is stuck:

    kubectl rollout status deployment/payment -n production

Then:

    kubectl describe deployment payment -n production

---

# ALB Failure Investigation

If users receive 503:

    User
        |
        ↓
    ALB
        |
        ↓
    Target
        |
        ↓
    Service
        |
        ↓
    Pod

Check:

    ALB Target Health
    Ingress
    Service
    Endpoints
    Pods
    Readiness

---

# 503 After Change

Example:

    Deployment
        |
        ↓
    503
        |
        ↓
    Check ALB
        |
        ↓
    Check Target Health
        |
        ↓
    Check Service
        |
        ↓
    Check Endpoints
        |
        ↓
    Check Pods
        |
        ↓
    Check Readiness

---

# Application Latency Increase

Example:

    Release
        |
        ↓
    Latency Increased
        |
        ↓
    Prometheus
        |
        ↓
    Grafana
        |
        ↓
    Application Logs
        |
        ↓
    Database / Dependency
        |
        ↓
    Root Cause

Possible causes:

    Slow Database
    Code Regression
    External API
    Resource Constraints
    Network Issue

---

# Memory Problem After Change

Example:

    Release
        |
        ↓
    Memory Increased
        |
        ↓
    Pods OOMKilled
        |
        ↓
    Application Unstable

Check:

    Resource Requests
    Resource Limits
    Application Behavior
    Logs
    Traffic

Possible action:

    Rollback
        |
        ↓
    Restore Stability

Then investigate root cause.

---

# CrashLoopBackOff After Change

Check:

    kubectl describe pod <pod> -n production

Then:

    kubectl logs <pod> --previous -n production

Validate:

    Environment Variables
    Secrets
    ConfigMaps
    Probes
    Image
    Dependencies
    Resource Limits

---

# ImagePullBackOff After Change

Possible causes:

    Wrong Image Tag
    Image Missing
    ECR Permission
    Registry Authentication
    Incorrect Repository

Check:

    kubectl describe pod <pod> -n production

If the artifact is incorrect:

    Stop
        |
        ↓
    Correct Version
        |
        ↓
    Revalidate
        |
        ↓
    Deploy

---

# Configuration Failure After Change

Example:

    New Environment Variable
        |
        X
    Missing
        |
        ↓
    Application Startup Failure

Recovery:

    Restore Correct Configuration
        |
        ↓
    Restart
        |
        ↓
    Health Check

---

# Secret Failure After Change

Example:

    Application
        |
        ↓
    Secret Changed
        |
        X
    Authentication Failure

Check:

    Secret Reference
    Secret Value
    Permissions
    Mount / Environment
    External Dependency

Do not print secret values in logs.

---

# Network Change Failure

Example:

    Terraform
        |
        ↓
    Security Group Change
        |
        ↓
    Application
        X
    Database Connection

Recovery:

    Identify Network Change
        |
        ↓
    Restore Safe Configuration
        |
        ↓
    Validate Connectivity

---

# IAM Change Failure

Example:

    IAM Policy
        |
        ↓
    Deployment
        |
        X
    ECR Access

Check:

    IAM Role
    Policy
    Trust Relationship
    Permissions
    Service Account

---

# Change Failure With No Rollback

If rollback is not possible:

    Failure
        |
        ↓
    Stabilize
        |
        ↓
    Fix Forward
        |
        ↓
    Validate
        |
        ↓
    Monitor

This is common for certain database and infrastructure changes.

---

# Fix Forward

Fix forward means deploying a corrective change rather than reverting.

Example:

    v1.4.7
        |
        ↓
    Failure
        |
        ↓
    v1.4.8 Hotfix
        |
        ↓
    Deploy
        |
        ↓
    Validate

Use when:

    Rollback Is Unsafe
    Data Changed
    Forward Fix Is Faster
    Compatibility Requires It

---

# Change Failure and Hotfix

A hotfix should still follow appropriate controls.

Flow:

    Failure
        |
        ↓
    Hotfix Development
        |
        ↓
    Testing
        |
        ↓
    Security / Quality Checks
        |
        ↓
    Approval
        |
        ↓
    Production
        |
        ↓
    Validation

Emergency procedures may shorten the process while maintaining required controls.

---

# Emergency Change Failure

If an emergency change fails:

    Emergency Change
        |
        ↓
    Failure
        |
        ↓
    Incident Response
        |
        ↓
    Stabilize
        |
        ↓
    Rollback / Fix
        |
        ↓
    Validate
        |
        ↓
    Post-Incident Review

---

# Change Failure Documentation

Record:

    Change ID
    Application
    Version
    Environment
    Start Time
    Failure Time
    Detection Time
    Recovery Time
    Impact
    Root Cause
    Rollback
    Corrective Action
    Approvers
    Owners

---

# Change Failure Timeline

Example:

    Change:
    CHG-10245

    Version:
    payment:1.4.7

    22:00
    Deployment Started

    22:10
    Error Rate Increased

    22:12
    Alert Triggered

    22:15
    Incident Declared

    22:20
    Rollback Started

    22:28
    Previous Version Healthy

    22:35
    Business Validation Passed

---

# Corrective Action

Corrective actions address the immediate problem.

Example:

    Problem:
    Incorrect Configuration

    Corrective Action:
    Restore Correct Configuration

---

# Preventive Action

Preventive actions reduce the chance of recurrence.

Example:

    Problem:
    Configuration Error

    Preventive Action:
    Add Automated Configuration Validation

---

# Corrective vs Preventive

Corrective:

    Fix The Current Problem

Preventive:

    Reduce Future Risk

Example:

    Incorrect Security Group
        |
        ↓
    Corrective:
    Restore Security Rule

    Preventive:
    Add Terraform Policy Validation

---

# Change Failure Action Items

Examples:

    Add Automated Test
    Improve Monitoring
    Add Alert
    Update Runbook
    Improve Rollback
    Add Validation
    Improve Documentation
    Add Security Check
    Improve Deployment Strategy

---

# Change Failure and Runbooks

A runbook should explain:

    Symptoms
    Checks
    Commands
    Recovery
    Validation
    Escalation

Example:

    503 After Deployment
        |
        ↓
    Check ALB
        |
        ↓
    Check Service
        |
        ↓
    Check Endpoints
        |
        ↓
    Check Pods
        |
        ↓
    Rollback If Required

---

# Change Failure and Automation

Automate where possible:

    Failure Detection
    Rollout Pause
    Health Checks
    Rollback
    Notifications
    Evidence Collection

Example:

    Error Rate
        |
        ↓
    Alert
        |
        ↓
    Deployment Paused
        |
        ↓
    Automated Rollback
        |
        ↓
    Validation

Automation must be carefully designed and tested before being trusted in production.

---

# Automatic Rollback

Automatic rollback can be useful when:

    Failure Condition Is Clear
    Rollback Is Safe
    Previous Version Is Known-Good
    Health Signal Is Reliable

Example:

    Error Rate > Threshold
        |
        ↓
    Rollout Stop
        |
        ↓
    Rollback
        |
        ↓
    Validate

---

# Automatic Rollback Risks

Automatic rollback can be dangerous when:

    Monitoring Is Incorrect
    Failure Signal Is Noisy
    Database Is Changed
    Rollback Is Not Safe
    Dependency Is The Real Problem

Therefore:

    Automate Carefully

---

# Change Failure and Approval

After recovery, determine whether:

    Original Change
        |
        ↓
    Should Be Retried

If yes:

    Root Cause Fixed
        |
        ↓
    Testing
        |
        ↓
    Reapproval
        |
        ↓
    New Deployment

Do not simply rerun the failed change without understanding the failure.

---

# Retrying a Failed Change

Bad:

    Failure
        |
        ↓
    Retry
        |
        ↓
    Failure
        |
        ↓
    Retry

Better:

    Failure
        |
        ↓
    Investigate
        |
        ↓
    Root Cause
        |
        ↓
    Fix
        |
        ↓
    Test
        |
        ↓
    Reapprove
        |
        ↓
    Retry

---

# Change Failure and Release Management

Release management should include:

    Failure Detection
    Rollback
    Incident Handling
    Communication
    RCA
    Corrective Action

Relationship:

    Release
        |
        ↓
    Failure
        |
        ↓
    Change Failure Handling
        |
        ↓
    Recovery
        |
        ↓
    Release Closure

---

# Change Failure and Deployment Window

If a failure occurs inside the deployment window:

    Deployment
        |
        ↓
    Failure
        |
        ↓
    Rollback
        |
        ↓
    Validation

If the recovery exceeds the window:

    Assess
        |
        ↓
    Follow Extension / Emergency Procedure

---

# Change Failure and Monitoring Baseline

Before deployment:

    Capture Baseline

After deployment:

    Compare

Example:

    CPU:
    Normal → Normal

    Error Rate:
    Normal → Increased

    Latency:
    Normal → Increased

This provides evidence for impact assessment.

---

# Business Metric Monitoring

Technical metrics are not always enough.

Monitor business indicators such as:

    Successful Orders
    Payment Success
    Login Success
    Transaction Completion
    User Registration

Example:

    CPU = Normal

But:

    Payment Success = Decreased

This may indicate a business-level change failure.

---

# Technical vs Business Validation

Technical:

    Pods Healthy
    CPU Normal
    Memory Normal
    HTTP 200

Business:

    Order Created
    Payment Completed
    Inventory Updated
    Notification Sent

Both can be required.

---

# Change Failure and Data Integrity

A failure may affect data.

Check:

    Missing Data
    Duplicate Data
    Incorrect State
    Partial Transactions
    Database Consistency

If data is affected:

    Stop Further Damage
        |
        ↓
    Assess Data
        |
        ↓
    Recovery Plan
        |
        ↓
    Validate

---

# Data Failure Example

Example:

    Order Created
        |
        ↓
    Payment Failed
        |
        ↓
    Order State Incorrect

Investigate:

    Order State
    Payment State
    Inventory State

Then restore a consistent business state.

---

# Change Failure and Idempotency

Idempotent operations can make retries safer.

Example:

    Request
        |
        ↓
    Retry
        |
        ↓
    Same Final Result

Without idempotency:

    Retry
        |
        ↓
    Duplicate Transaction

Critical workflows should be designed carefully for safe retries.

---

# Change Failure and Retry

Use retries for:

    Temporary Network Errors
    Transient Dependency Failures

Do not use retries to hide:

    Code Bugs
    Invalid Configuration
    Permanent Errors

---

# Change Failure and Circuit Breaker

For external dependencies:

    Application
        |
        ↓
    External Service
        |
        X
    Failure

Circuit breaker:

    Failure Threshold
        |
        ↓
    Open Circuit
        |
        ↓
    Protect Application

This can reduce cascading failures.

---

# Cascading Failure

Example:

    Service A
        |
        ↓
    Service B
        |
        ↓
    Service C
        |
        X
    Database

Failure can propagate:

    C
    ↓
    B
    ↓
    A
    ↓
    Users

Change failure handling should consider dependencies and blast radius.

---

# Change Failure and Dependency Isolation

Use:

    Timeouts
    Retries
    Circuit Breakers
    Rate Limits
    Bulkheads

when appropriate.

These patterns can limit failure propagation.

---

# Change Failure and Kubernetes Probes

Readiness:

    Controls Traffic

Liveness:

    Detects Unhealthy Container

Startup:

    Handles Slow Startup

Correct probe configuration can reduce deployment failures.

---

# Change Failure and Resource Limits

If a new release increases memory:

    Application
        |
        ↓
    Memory Usage
        |
        ↓
    Limit Exceeded
        |
        ↓
    OOMKilled

Investigate:

    Memory Leak
    Traffic
    Configuration
    Resource Limit

---

# Change Failure and HPA

A release may change resource behavior.

Example:

    Traffic Increased
        |
        ↓
    HPA
        |
        ↓
    Scale Out
        |
        ↓
    New Pods
        |
        X
    Startup Failure

Check:

    HPA
    Pods
    Requests
    Limits
    Startup
    Readiness

---

# Change Failure and Security

A deployment may introduce:

    Vulnerability
    Incorrect IAM
    Exposed Endpoint
    Weak Configuration
    Secret Problem

Recovery may require:

    Disable Feature
    Rollback
    Patch
    Rotate Credentials
    Restrict Access

---

# Change Failure and Secrets

If a secret is accidentally exposed:

    Stop Exposure
        |
        ↓
    Rotate Secret
        |
        ↓
    Update Application
        |
        ↓
    Validate
        |
        ↓
    Security Review

Do not only rollback the application and assume the secret is safe.

---

# Change Failure and IAM

If an IAM change causes failure:

    Application
        |
        ↓
    Access Denied
        |
        ↓
    IAM Policy
        |
        ↓
    Investigate
        |
        ↓
    Restore Required Permission
        |
        ↓
    Validate

Follow least privilege when correcting the issue.

---

# Change Failure and Infrastructure

Infrastructure changes may affect:

    Network
    Security Groups
    IAM
    EKS
    ALB
    RDS
    VPC

Use:

    Terraform Plan
    State
    Monitoring
    Validation

to understand the change.

---

# Change Failure and GitOps

GitOps provides a clear desired state.

Example:

    Git
        |
        ↓
    Desired State
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Failure

Recovery:

    Revert Git
        |
        ↓
    ArgoCD
        |
        ↓
    Previous State

---

# Change Failure and Drift

If the cluster differs from Git:

    Desired State
        |
        X
    Actual State

ArgoCD can identify drift.

The team should determine:

    Was Drift Expected?

or:

    Was Drift The Failure?

---

# Change Failure and Audit

Record:

    Change ID
    Failure
    Detection
    Actions
    Approvals
    Recovery
    Root Cause
    Corrective Actions

This supports:

    Accountability
    Learning
    Compliance
    Improvement

---

# Post-Incident Review

After recovery:

    Incident
        |
        ↓
    Recovery
        |
        ↓
    Post-Incident Review
        |
        ↓
    RCA
        |
        ↓
    Action Items
        |
        ↓
    Follow-Up

---

# Post-Incident Review Questions

Ask:

    What Happened?

    What Was Changed?

    What Was The Impact?

    How Was It Detected?

    How Long Did Recovery Take?

    Why Did The Change Fail?

    Why Was It Not Detected Earlier?

    What Worked Well?

    What Did Not Work?

    What Will We Change?

---

# Corrective Action Tracking

Every action should have:

    Action
    Owner
    Priority
    Due Date
    Status

Example:

    Action:
    Add E2E Payment Validation

    Owner:
    Application Team

    Priority:
    High

    Status:
    In Progress

---

# Change Failure Prevention Loop

    Failure
        |
        ↓
    RCA
        |
        ↓
    Corrective Action
        |
        ↓
    Automation
        |
        ↓
    Testing
        |
        ↓
    Monitoring
        |
        ↓
    Safer Future Release

---

# Learning From Failure

A failed change should improve:

    Code
    Tests
    Monitoring
    Deployment
    Runbooks
    Architecture
    Security
    Process

The goal is not simply:

    "Fix The Incident"

The goal is:

    "Make The Next Change Safer"

---

# Change Failure Checklist

## Detection

    Alert Triggered
    Health Check Failed
    Smoke Test Failed
    E2E Failed
    User Impact Detected

## Assessment

    Application Identified
    Version Identified
    Impact Identified
    Dependencies Identified
    Severity Identified

## Recovery

    Rollout Stopped
    Traffic Controlled
    Rollback / Fix Selected
    Recovery Executed
    Validation Completed

## Investigation

    Timeline Created
    Logs Reviewed
    Metrics Reviewed
    Configuration Reviewed
    Root Cause Identified

## Prevention

    Corrective Action
    Preventive Action
    Monitoring Improvement
    Test Improvement
    Runbook Improvement

---

# Production Change Failure Checklist

Before declaring recovery:

    Previous / Correct Version Running
    Pods Healthy
    Readiness Passed
    Services Healthy
    ALB Healthy
    APIs Healthy
    Database Healthy
    Critical Workflow Tested
    Error Rate Normal
    Latency Normal
    No Critical Alerts
    Business Validation Complete

---

# Change Failure Decision Tree

    Failure Detected
        |
        ↓
    Is User Impact High?
        |
        +------ No
        |       |
        |       ↓
        |    Investigate
        |
        +------ Yes
                |
                ↓
            Stop Rollout
                |
                ↓
            Can We Rollback Safely?
                |
                +------ Yes
                |       |
                |       ↓
                |    Rollback
                |
                +------ No
                        |
                        ↓
                    Mitigate
                        |
                        ↓
                    Fix Forward
                        |
                        ↓
                    Validate

---

# Change Failure Decision Tree

After recovery:

    Service Stable?
        |
        +------ No
        |       |
        |       ↓
        |    Continue Incident Response
        |
        +------ Yes
                |
                ↓
            Root Cause Known?
                |
                +------ No
                |       |
                |       ↓
                |    Investigate
                |
                +------ Yes
                        |
                        ↓
                    Corrective Action
                        |
                        ↓
                    Preventive Action

---

# Real-World Example

Application:

    Payment Service

Version:

    v1.4.7

Deployment:

    EKS

Problem:

    HTTP 500 increased after release.

Flow:

    v1.4.7
        |
        ↓
    Production
        |
        ↓
    Error Rate Increased
        |
        ↓
    Prometheus Alert
        |
        ↓
    Deployment Paused
        |
        ↓
    ELK Investigation
        |
        ↓
    Database Connection Errors
        |
        ↓
    Root Cause Identified
        |
        ↓
    Rollback
        |
        ↓
    v1.4.6
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Monitoring Stable
        |
        ↓
    Service Recovered

---

# Real-World Example: Configuration Failure

Change:

    Payment v1.4.7

Problem:

    Missing Environment Variable

Result:

    Pod CrashLoopBackOff

Investigation:

    kubectl describe pod
        |
        ↓
    kubectl logs --previous
        |
        ↓
    Missing Configuration
        |
        ↓
    Root Cause

Recovery:

    Restore Configuration
        |
        ↓
    Restart
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test

Prevention:

    Add Configuration Validation

---

# Real-World Example: Image Failure

Change:

    Payment v1.4.7

Problem:

    ImagePullBackOff

Investigation:

    kubectl describe pod
        |
        ↓
    Image Tag
        |
        ↓
    ECR
        |
        ↓
    Image Missing

Recovery:

    Stop Deployment
        |
        ↓
    Use Correct Image
        |
        ↓
    Revalidate
        |
        ↓
    Deploy

Prevention:

    Add Artifact Verification

---

# Real-World Example: ALB 503

Deployment:

    v1.4.7

Problem:

    Users receive 503

Investigation:

    User
        |
        ↓
    ALB
        |
        ↓
    Target Health
        |
        ↓
    Service
        |
        ↓
    Endpoints
        |
        ↓
    Pods
        |
        ↓
    Readiness Probe

Root Cause:

    New Pods Were Not Ready

Recovery:

    Rollback / Fix Readiness
        |
        ↓
    Validate
        |
        ↓
    Monitor

---

# Real-World Example: Terraform Failure

Change:

    Terraform Network Update

Result:

    Application Cannot Reach Database

Flow:

    Terraform
        |
        ↓
    Network Change
        |
        ↓
    Database Connectivity Lost
        |
        ↓
    Incident
        |
        ↓
    Assess State
        |
        ↓
    Restore Safe Network Configuration
        |
        ↓
    Validate Connectivity
        |
        ↓
    Application Recovery

Prevention:

    Add Infrastructure Connectivity Test

---

# Real-World Example: GitOps Failure

Change:

    Git Commit

    payment image:
    1.4.7

Flow:

    Git
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
        X
    Failure

Recovery:

    Revert Git Commit
        |
        ↓
    ArgoCD
        |
        ↓
    Previous State
        |
        ↓
    Validation

---

# Real-World Example: Database Failure

Release:

    v1.4.7

Migration:

    Add New Column

Problem:

    Migration Failed

Action:

    Stop Application Promotion
        |
        ↓
    Assess Database State
        |
        ↓
    Determine Migration Status
        |
        ↓
    Recovery Plan
        |
        ↓
    Validate Database
        |
        ↓
    Validate Application

Do not blindly execute a destructive rollback.

---

# Real-World Example: Canary Failure

    v1.4.6
        |
        ↓
    95% Traffic

    v1.4.7
        |
        ↓
    5% Traffic
        |
        ↓
    Error Rate Increased
        |
        ↓
    Stop Promotion
        |
        ↓
    Route Traffic To v1.4.6
        |
        ↓
    Investigate v1.4.7

The limited blast radius protects most users.

---

# Real-World Example: Business Failure

Technical:

    Pods Healthy
    HTTP 200
    CPU Normal

Business:

    Payment Success Rate
        |
        ↓
    Decreased

This is still a serious release problem.

Action:

    Investigate Business Workflow
        |
        ↓
    Payment Service
        |
        ↓
    External Gateway
        |
        ↓
    Logs
        |
        ↓
    Recovery

---

# Real-World Example: Failed Release Retried

Bad:

    v1.4.7
        |
        X
    Failure
        |
        ↓
    Retry Same Version
        |
        X
    Failure

Better:

    v1.4.7
        |
        X
    Failure
        |
        ↓
    RCA
        |
        ↓
    Fix
        |
        ↓
    v1.4.8
        |
        ↓
    Test
        |
        ↓
    Approval
        |
        ↓
    Production

---

# Enterprise Change Failure Flow

    Developer
        |
        ↓
    Git
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
    Artifact
        |
        ↓
    QA
        |
        ↓
    SIT
        |
        ↓
    E2E
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
    Monitoring
        |
        X
    Change Failure
        |
        ↓
    Stop Rollout
        |
        ↓
    Assess
        |
        ↓
    Rollback / Fix
        |
        ↓
    Validation
        |
        ↓
    Incident Update
        |
        ↓
    RCA
        |
        ↓
    Corrective Action
        |
        ↓
    Preventive Action
        |
        ↓
    Change Closure

---

# Change Failure and DevSecOps

A mature DevSecOps pipeline reduces failure risk through:

    Code Review
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
    Veracode
        |
        ↓
    Integration Tests
        |
        ↓
    E2E
        |
        ↓
    Approval
        |
        ↓
    Controlled Deployment
        |
        ↓
    Monitoring
        |
        ↓
    Automated Recovery Where Safe

---

# Change Failure and CI/CD

CI/CD should make recovery easier by providing:

    Versioned Artifacts
    Repeatable Deployments
    Deployment History
    Rollback Capability
    Automated Testing
    Automated Validation
    Audit Trail

---

# Change Failure and GitHub Actions

Conceptual flow:

    GitHub Actions
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security
        |
        ↓
    Deploy
        |
        ↓
    Health Check
        |
        X
    Failure
        |
        ↓
    Stop
        |
        ↓
    Rollback
        |
        ↓
    Validate

---

# Change Failure and ArgoCD

GitOps:

    Git
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        X
    Failure
        |
        ↓
    Revert Git
        |
        ↓
    ArgoCD
        |
        ↓
    Previous State
        |
        ↓
    Validation

---

# Change Failure and Observability Stack

Use:

    Prometheus
        |
        ↓
    Metrics

    Grafana
        |
        ↓
    Visualization

    ELK
        |
        ↓
    Logs

Together:

    Metrics
        +
    Dashboards
        +
    Logs
        =
    Faster Troubleshooting

---

# Change Failure Best Practices

- Detect failures quickly
- Stop further rollout when necessary
- Assess user and business impact
- Identify the exact version
- Identify the exact change
- Communicate significant failures
- Use tested rollback procedures
- Do not blindly rollback unsafe database changes
- Fix forward when appropriate
- Validate after recovery
- Monitor after recovery
- Document the timeline
- Perform root cause analysis
- Use blameless investigation
- Track corrective actions
- Improve automated testing
- Improve monitoring
- Improve deployment strategies
- Reduce blast radius
- Maintain immutable artifacts
- Keep reliable runbooks
- Test recovery procedures
- Learn from every significant failure

---

# Change Failure Anti-Patterns

## Ignoring Alerts

Bad:

    Alert
        |
        ↓
    Ignore

Better:

    Alert
        |
        ↓
    Assess
        |
        ↓
    Act

---

# Change Failure Anti-Pattern

## Continuing A Failed Rollout

Bad:

    Failure At 10%
        |
        ↓
    Continue To 50%
        |
        ↓
    Continue To 100%

Better:

    Failure At 10%
        |
        ↓
    Stop
        |
        ↓
    Assess
        |
        ↓
    Rollback / Fix

---

# Change Failure Anti-Pattern

## Blind Rollback

Bad:

    Failure
        |
        ↓
    Rollback Immediately

Without checking:

    Database
    Data
    Dependencies
    Compatibility

Better:

    Failure
        |
        ↓
    Assess Rollback Safety
        |
        ↓
    Rollback / Fix Forward

---

# Change Failure Anti-Pattern

## Blind Retry

Bad:

    Failure
        |
        ↓
    Retry
        |
        ↓
    Failure
        |
        ↓
    Retry

Better:

    Failure
        |
        ↓
    Investigate
        |
        ↓
    Fix
        |
        ↓
    Test
        |
        ↓
    Retry

---

# Change Failure Anti-Pattern

## Blaming Individuals

Bad:

    Engineer
        |
        ↓
    Mistake
        |
        ↓
    Blame

Better:

    Failure
        |
        ↓
    System Analysis
        |
        ↓
    Process Analysis
        |
        ↓
    Technical Analysis
        |
        ↓
    Improvement

---

# Change Failure Anti-Pattern

## No Root Cause Analysis

Bad:

    Rollback
        |
        ↓
    Done

Better:

    Rollback
        |
        ↓
    Recovery
        |
        ↓
    RCA
        |
        ↓
    Corrective Action

---

# Change Failure Anti-Pattern

## No Preventive Action

Bad:

    Failure
        |
        ↓
    Fix
        |
        ↓
    Close

Better:

    Failure
        |
        ↓
    Fix
        |
        ↓
    RCA
        |
        ↓
    Prevention
        |
        ↓
    Improvement

---

# Change Failure Anti-Pattern

## No Validation After Rollback

Bad:

    Rollback
        |
        ↓
    Assume Healthy

Better:

    Rollback
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
    Monitor

---

# Change Failure Anti-Pattern

## Logging Sensitive Information

Bad:

    Password
    Token
    Secret
        |
        ↓
    Application Logs

Better:

    Secure Secret
        |
        ↓
    Application
        |
        ↓
    Sanitized Logs

---

# Change Failure Anti-Pattern

## No Version Traceability

Bad:

    Production:
    Unknown Version

Better:

    Application:
    Payment

    Version:
    v1.4.7

    Git SHA:
    abc123

    Image:
    payment:1.4.7

---

# Change Failure Interview Questions

## Basic

1. What is a change failure?

2. What is change failure handling?

3. What is the difference between a change failure and an incident?

4. What is rollback?

5. What is fix forward?

6. What is root cause analysis?

7. What is change failure rate?

8. What is MTTR?

9. What is blast radius?

10. Why is change failure documentation important?

---

# Change Failure Interview Questions

## Intermediate

11. What do you do when a production deployment fails?

12. How do you decide between rollback and fix forward?

13. How do you troubleshoot a failed Kubernetes deployment?

14. How do you handle a failed Terraform change?

15. How do you handle a database migration failure?

16. How do you identify the root cause of a change failure?

17. How do you handle a partial deployment?

18. How do you handle a failed canary deployment?

19. How do you handle a change that causes increased latency?

20. How do you validate a rollback?

---

# Change Failure Interview Questions

## Advanced

21. How would you design an enterprise change failure process?

22. How would you reduce change failure rate?

23. How would you implement automated rollback?

24. When should you avoid automatic rollback?

25. How would you design change failure handling for EKS?

26. How would you handle a Terraform apply that partially succeeds?

27. How would you handle a database migration that cannot be rolled back?

28. How would you design change failure handling with ArgoCD?

29. How would you integrate observability with change failure detection?

30. How would you reduce the blast radius of production changes?

31. How would you design a blameless RCA process?

32. How would you prevent the same production failure from recurring?

---

# Scenario-Based Interview Question

## Production Deployment Causes 503

Situation:

    Deployment
        |
        ↓
    503 Errors

Response:

    Stop Rollout
        |
        ↓
    Check ALB
        |
        ↓
    Check Service
        |
        ↓
    Check Endpoints
        |
        ↓
    Check Pods
        |
        ↓
    Check Readiness
        |
        ↓
    Rollback If Required
        |
        ↓
    Validate

---

# Scenario-Based Interview Question

## Terraform Apply Failed Halfway

Response:

    Stop
        |
        ↓
    Check Terraform State
        |
        ↓
    Check Real Infrastructure
        |
        ↓
    terraform plan
        |
        ↓
    Understand Partial State
        |
        ↓
    Recover Safely
        |
        ↓
    Validate

Do not immediately delete state or rerun destructive commands.

---

# Scenario-Based Interview Question

## Release Causes High Latency

Response:

    Deployment
        |
        ↓
    Latency Increased
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
    Application
        |
        ↓
    Database
        |
        ↓
    Dependencies

Then:

    Mitigate
        |
        ↓
    Rollback / Fix
        |
        ↓
    Validate

---

# Scenario-Based Interview Question

## Database Migration Failed

Response:

    Stop Promotion
        |
        ↓
    Assess Database State
        |
        ↓
    Determine Migration Status
        |
        ↓
    Backup / Recovery Assessment
        |
        ↓
    Choose Recovery Strategy
        |
        ↓
    Validate
        |
        ↓
    Resume Release Only After Approval

---

# Scenario-Based Interview Question

## Canary Deployment Fails

Response:

    5% Traffic
        |
        ↓
    Error Rate Increased
        |
        ↓
    Stop Promotion
        |
        ↓
    Return Traffic To Stable Version
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

# Scenario-Based Interview Question

## Rollback Is Not Safe

Response:

    Failure
        |
        ↓
    Rollback Unsafe
        |
        ↓
    Contain Impact
        |
        ↓
    Fix Forward
        |
        ↓
    Validate
        |
        ↓
    Monitor

---

# Scenario-Based Interview Question

## Application Is Healthy But Business Transactions Fail

Response:

    Technical Health
        |
        ✓
    Pods Healthy

    Business Health
        |
        X
    Payment Failed

Investigate:

    Business Workflow
    Application Logic
    Database
    External Dependencies
    Logs
    Metrics

Then recover and validate the complete business workflow.

---

# Scenario-Based Interview Question

## Same Failure Happened Multiple Times

Do not simply repeat the same recovery.

Flow:

    Repeated Failure
        |
        ↓
    Deep RCA
        |
        ↓
    Identify Systemic Cause
        |
        ↓
    Corrective Action
        |
        ↓
    Preventive Action
        |
        ↓
    Test Prevention
        |
        ↓
    Deploy Safely

---

# Final Change Failure Mental Model

Remember:

    DETECT
      |
      ↓
    ASSESS
      |
      ↓
    CONTAIN
      |
      ↓
    RECOVER
      |
      ↓
    VALIDATE
      |
      ↓
    MONITOR
      |
      ↓
    INVESTIGATE
      |
      ↓
    RCA
      |
      ↓
    PREVENT
      |
      ↓
    IMPROVE

If rollback is safe:

    FAILURE
        |
        ↓
    STOP
        |
        ↓
    ROLLBACK
        |
        ↓
    VALIDATE
        |
        ↓
    MONITOR

If rollback is unsafe:

    FAILURE
        |
        ↓
    CONTAIN
        |
        ↓
    FIX FORWARD
        |
        ↓
    VALIDATE
        |
        ↓
    MONITOR

---

# Final Concept

Change Failure Handling is not just about rolling back a deployment.

It is a complete reliability process:

    Detect
        +
    Assess
        +
    Contain
        +
    Recover
        +
    Validate
        +
    Communicate
        +
    Investigate
        +
    Prevent
        =
    Reliable Change Management

A mature enterprise DevOps organization treats failures as opportunities to improve:

    Code
        +
    Testing
        +
    Security
        +
    Deployment
        +
    Monitoring
        +
    Architecture
        +
    Process

The most important principle is:

    Recover Quickly
        +
    Understand Completely
        +
    Prevent Recurrence

The complete enterprise flow is:

    Change
        |
        ↓
    CI
        |
        ↓
    Testing
        |
        ↓
    Security
        |
        ↓
    Approval
        |
        ↓
    Production
        |
        ↓
    Monitoring
        |
        X
    Failure
        |
        ↓
    Stop Rollout
        |
        ↓
    Assess Impact
        |
        ↓
    Rollback / Fix Forward
        |
        ↓
    Validate
        |
        ↓
    Restore Service
        |
        ↓
    RCA
        |
        ↓
    Corrective Action
        |
        ↓
    Preventive Action
        |
        ↓
    Change Closure