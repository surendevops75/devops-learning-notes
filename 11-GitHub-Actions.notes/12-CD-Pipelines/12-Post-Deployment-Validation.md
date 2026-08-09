# Post-Deployment Validation

Post-Deployment Validation is the process of verifying that an application has been successfully deployed and is working correctly in the target environment.

A deployment is not successful just because Kubernetes reports that Pods are running.

The application should be validated from multiple perspectives:

    Deployment Status
        |
        ↓
    Pod Health
        |
        ↓
    Readiness
        |
        ↓
    Application Health
        |
        ↓
    Service Connectivity
        |
        ↓
    Load Balancer Health
        |
        ↓
    Smoke Tests
        |
        ↓
    Metrics
        |
        ↓
    Logs
        |
        ↓
    Business Validation
        |
        ↓
    Deployment Success

---

# Why Post-Deployment Validation Is Important

A deployment can technically succeed while the application is still broken.

Example:

    Kubernetes Deployment
        |
        ↓
    Rollout Successful
        |
        ↓
    Pods Running
        |
        ↓
    Application Returns 500 Errors

Therefore:

    Deployment Success
        ≠
    Application Success

Post-deployment validation verifies the actual application behavior after release.

---

# Deployment Validation Mental Model

    Code
      |
      ↓
    Build
      |
      ↓
    Test
      |
      ↓
    Deploy
      |
      ↓
    Kubernetes Rollout
      |
      ↓
    Health Checks
      |
      ↓
    Smoke Tests
      |
      ↓
    Functional Validation
      |
      ↓
    Monitoring
      |
      ↓
    Business Validation
      |
      ↓
    Promote / Rollback

---

# Main Objectives

Post-deployment validation should answer:

    Did the deployment complete?

    Are the Pods healthy?

    Are the Pods Ready?

    Is the application responding?

    Is the Service working?

    Is the Load Balancer healthy?

    Are critical APIs working?

    Are there errors in logs?

    Are error rates normal?

    Is latency acceptable?

    Are critical business functions working?

    Should the release be promoted?

    Should the release be rolled back?

---

# Deployment Validation Layers

A production deployment can be validated at multiple layers:

    Layer 1
    Deployment Validation

    Layer 2
    Pod Validation

    Layer 3
    Health Validation

    Layer 4
    Service Validation

    Layer 5
    Load Balancer Validation

    Layer 6
    Application Validation

    Layer 7
    Smoke Testing

    Layer 8
    Monitoring Validation

    Layer 9
    Business Validation

---

# Deployment Status Validation

First verify whether Kubernetes successfully completed the deployment.

Command:

    kubectl rollout status deployment/<deployment-name>

Example:

    kubectl rollout status deployment/payment

Expected result:

    deployment "payment" successfully rolled out

If rollout fails:

    Deployment
        |
        ↓
    Rollout Failure
        |
        ↓
    Stop Promotion
        |
        ↓
    Investigate
        |
        ↓
    Fix / Rollback

---

# Check Deployment

Command:

    kubectl get deployment

Example:

    NAME       READY   UP-TO-DATE   AVAILABLE
    payment    3/3     3            3

Expected:

    READY = Desired Replicas

    UP-TO-DATE = Desired Replicas

    AVAILABLE = Desired Replicas

If these values are not correct, investigate before continuing.

---

# Check ReplicaSet

Command:

    kubectl get rs

ReplicaSets help verify which version of Pods Kubernetes is managing.

Check:

    Desired
    Current
    Ready

Example:

    payment-7f8d9c8d7f
        |
        +-- Desired: 3
        +-- Current: 3
        +-- Ready: 3

---

# Check Pods

Command:

    kubectl get pods

Example:

    NAME                       READY   STATUS    RESTARTS
    payment-7f8d9c8d7f-abc12   1/1     Running   0
    payment-7f8d9c8d7f-def34   1/1     Running   0
    payment-7f8d9c8d7f-ghi56   1/1     Running   0

Healthy state:

    READY = 1/1
    STATUS = Running
    RESTARTS = 0

Unexpected restarts should be investigated.

---

# Check Pod Readiness

A Pod can be:

    Running
    but
    Not Ready

Example:

    NAME                       READY   STATUS
    payment-7f8d9c8d7f-abc12   0/1     Running

This means the container may be running, but the Pod is not currently ready to receive traffic.

Check:

    kubectl describe pod <pod-name>

---

# Check Pod Conditions

Command:

    kubectl describe pod <pod-name>

Important conditions include:

    Initialized
    Ready
    ContainersReady
    PodScheduled

A production Pod should normally have:

    Ready = True

    ContainersReady = True

---

# Check Health Probes

Post-deployment validation should confirm:

    Startup Probe
    Readiness Probe
    Liveness Probe

Expected:

    Startup = Successful

    Readiness = Passing

    Liveness = Passing

If readiness fails:

    Pod
        |
        ↓
    Not Ready
        |
        ↓
    No Production Traffic

If liveness fails repeatedly:

    Pod
        |
        ↓
    Container Restart

---

# Check Pod Events

Command:

    kubectl describe pod <pod-name>

Look at:

    Events

Common problems:

    FailedScheduling
    FailedMount
    ImagePullBackOff
    ErrImagePull
    Readiness Probe Failed
    Liveness Probe Failed
    BackOff
    OOMKilled

Events are often one of the fastest ways to identify deployment problems.

---

# Check Application Logs

Command:

    kubectl logs <pod-name>

Check for:

    Startup Errors
    Exceptions
    Database Errors
    Connection Failures
    Authentication Errors
    Configuration Errors
    HTTP 5xx
    Timeout Errors

After deployment, compare logs with the previous version if necessary.

---

# Check Previous Container Logs

If the container restarted:

    kubectl logs <pod-name> --previous

This is useful for identifying:

    Crash
    Configuration Failure
    Out Of Memory
    Startup Failure
    Dependency Failure

---

# Check Service

Command:

    kubectl get svc

Verify:

    Service Exists
    Service Port
    Target Port
    ClusterIP
    Selector

Example:

    payment-service
        |
        ↓
    Port 80
        |
        ↓
    Target Port 8080
        |
        ↓
    Payment Pods

---

# Check Endpoints

Command:

    kubectl get endpoints <service-name>

or:

    kubectl get endpointslices

Verify that healthy Pods are represented as service endpoints.

If no endpoints exist:

    Service
        |
        ↓
    No Ready Pods
        |
        ↓
    No Traffic

---

# Check Service Selector

A common deployment problem is a selector mismatch.

Example:

    Service Selector:

    app: payment

Pod labels:

    app: payments

Result:

    Service
        |
        ↓
    Selector Does Not Match
        |
        ↓
    No Endpoints

Always verify:

    Service Selector

matches:

    Pod Labels

---

# Application Health Endpoint

Test the application health endpoint.

Example:

    curl http://<service>:8080/health

Expected:

    HTTP 200

Possible response:

    {
      "status": "UP"
    }

A successful health endpoint indicates that the application is responding according to the health-check implementation.

---

# Readiness Endpoint

Test:

    curl http://<service>:8080/ready

Expected:

    HTTP 200

Possible response:

    {
      "status": "READY"
    }

This validates that the application considers itself ready to serve traffic.

---

# Liveness Endpoint

Test:

    curl http://<service>:8080/live

Expected:

    HTTP 200

Possible response:

    {
      "status": "ALIVE"
    }

This helps validate the application's liveness endpoint.

---

# HTTP Status Validation

After deployment, verify important HTTP status codes.

Expected:

    HTTP 200
    HTTP 201
    HTTP 204

Depending on the API.

Unexpected:

    HTTP 500
    HTTP 502
    HTTP 503
    HTTP 504

A deployment that produces unexpected 5xx errors should not be promoted without investigation.

---

# API Validation

For an API application, validate critical endpoints.

Example:

    GET /health
    GET /users
    GET /products
    POST /orders
    GET /orders/{id}

The exact endpoints depend on the application.

---

# Smoke Testing

Smoke testing verifies that the most important application functions work after deployment.

Typical flow:

    Deployment
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        +-- Login
        +-- Critical API
        +-- Database
        +-- Service Communication
        |
        ↓
    Validation

Smoke tests should be fast and focused on critical functionality.

---

# Smoke Test Characteristics

Good smoke tests are:

    Fast
    Reliable
    Repeatable
    Automated
    Focused
    Environment-Aware

Avoid making smoke tests:

    Extremely Long
    Highly Fragile
    Dependent On Unnecessary External Systems

---

# Functional Validation

Functional validation checks whether important application functionality works correctly.

Example:

    User Login
        |
        ↓
    Product Search
        |
        ↓
    Create Order
        |
        ↓
    Payment
        |
        ↓
    Inventory Update

For a microservices application, validate critical service-to-service communication.

---

# Business Validation

Technical validation is not always enough.

Example:

    Pods Healthy
        |
        ↓
    APIs Return 200
        |
        ↓
    Payment Transaction Fails

Therefore business validation should verify critical business behavior.

Examples:

    Login Works
    Order Creation Works
    Payment Works
    Inventory Updates
    Notifications Trigger

---

# Database Validation

After deployment, verify database connectivity.

Possible checks:

    Application Can Connect
    Database Credentials Are Correct
    Required Tables Exist
    Required Migrations Completed
    Queries Work
    Connection Pool Is Healthy

Avoid destructive database tests in production.

---

# Database Migration Validation

If the release includes a database migration:

    Deployment
        |
        ↓
    Migration
        |
        ↓
    Application Startup
        |
        ↓
    Validation

Verify:

    Migration Completed
    Schema Is Compatible
    Application Can Read Data
    Application Can Write Data

Backward compatibility is important during rolling deployments.

---

# Backward-Compatible Database Changes

For zero-downtime deployments:

    Old Application
        |
        ↓
    Database

and:

    New Application
        |
        ↓
    Same Database

Both versions may temporarily run simultaneously.

Therefore database changes should be designed carefully.

Preferred pattern:

    Expand
        |
        ↓
    Deploy Compatible Application
        |
        ↓
    Migrate Data
        |
        ↓
    Contract

---

# Load Balancer Validation

For applications behind an ALB, verify target health.

Conceptually:

    ALB
        |
        ↓
    Target Group
        |
        +-- Target A → Healthy
        +-- Target B → Healthy
        +-- Target C → Healthy

All expected production targets should be healthy.

---

# ALB Health Check Validation

Check:

    Health Check Path
    Health Check Port
    Protocol
    Target Health
    Target Registration

Common failures:

    Wrong Path
    Wrong Port
    Application Not Ready
    Security Group Issue
    Target Port Mismatch
    Application Error

---

# Ingress Validation

For Kubernetes Ingress, verify:

    Ingress Exists
    Hostname Correct
    Listener Configuration
    Backend Service Correct
    Target Port Correct
    TLS Configuration
    Routing Rules

Command:

    kubectl get ingress

---

# DNS Validation

If the deployment changes DNS or routing:

    DNS
      |
      ↓
    Load Balancer
      |
      ↓
    Application

Validate:

    DNS Resolves
    Correct Hostname
    Correct Load Balancer
    TLS Certificate
    Expected Application Response

---

# TLS Validation

After deployment, verify:

    HTTPS Works
    Certificate Is Valid
    Certificate Matches Domain
    TLS Configuration Is Correct

Example:

    curl -I https://example.com

Check:

    HTTP Status
    Redirect
    TLS Certificate
    Response Headers

---

# Authentication Validation

If the application requires authentication, validate:

    Login
    Token Generation
    Token Validation
    Authorization
    Protected API Access

Example:

    Login
        |
        ↓
    Token
        |
        ↓
    Protected API
        |
        ↓
    Authorized Response

---

# Authorization Validation

Validate that users have the correct permissions.

Example:

    Normal User
        |
        ↓
    Restricted API
        |
        ↓
    Expected 403

Admin:

    Admin User
        |
        ↓
    Restricted API
        |
        ↓
    Expected Success

---

# Configuration Validation

Post-deployment validation should verify configuration.

Check:

    Environment Variables
    ConfigMaps
    Secrets
    Application Configuration
    Database URL
    Service URLs
    Feature Flags

A deployment can succeed while incorrect configuration causes runtime failures.

---

# Secret Validation

Verify that applications can access required secrets.

Examples:

    Database Credentials
    API Credentials
    TLS Certificates
    Application Secrets

Never print secret values in CI/CD logs.

---

# ConfigMap Validation

Check:

    kubectl get configmap

Verify that expected configuration is available to the application.

Incorrect configuration can cause:

    Startup Failure
    Readiness Failure
    Runtime Errors

---

# Image Validation

Verify that the correct container image was deployed.

Command:

    kubectl describe pod <pod-name>

Check:

    Image
    Image Tag
    Container Version

Example:

    payment:1.4.7

Do not assume the latest image is running.

---

# Image Digest Validation

For stronger verification, use the image digest.

Example:

    sha256:abcdef...

This provides stronger confidence that the expected immutable image was deployed.

---

# Version Validation

The application should expose a version or build identifier when appropriate.

Example:

    GET /version

Response:

    {
      "version": "1.4.7",
      "commit": "abc123"
    }

This can help confirm exactly what version is running.

---

# Git Commit Validation

Deployment validation can connect:

    Git Commit
        |
        ↓
    CI Build
        |
        ↓
    Container Image
        |
        ↓
    Deployment
        |
        ↓
    Running Version

This creates traceability from source code to production.

---

# Artifact Validation

Verify:

    Correct Image
    Correct Artifact
    Correct Version
    Correct Tag
    Correct Digest

Example:

    Git Commit
        |
        ↓
    Build
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    EKS

---

# Security Validation

After deployment, verify that:

    Application Is Accessible Correctly
    Secrets Are Not Exposed
    TLS Works
    Security Controls Are Active
    Unauthorized Requests Are Blocked
    Container Runs With Expected Permissions

Security validation should be included in the deployment process.

---

# Trivy and Image Validation

Trivy can be used during CI to scan container images before deployment.

Flow:

    Build Image
        |
        ↓
    Trivy Scan
        |
        +------ Vulnerability → Stop
        |
        +------ Pass
                |
                ↓
              Deploy
                |
                ↓
          Post-Deployment Validation

---

# SonarQube and Deployment Validation

SonarQube is primarily used before deployment for code-quality analysis.

Flow:

    Code
      |
      ↓
    SonarQube
      |
      ↓
    Quality Gate
      |
      ↓
    Build
      |
      ↓
    Deploy
      |
      ↓
    Post-Deployment Validation

---

# Post-Deployment Metrics

Monitor:

    Request Rate
    Error Rate
    Latency
    CPU
    Memory
    Pod Restarts
    Availability
    Database Connections

A deployment should be compared with the previous stable version.

---

# Golden Signals

Important production signals include:

    Latency
    Traffic
    Errors
    Saturation

Example:

    Deployment
        |
        ↓
    Monitor Golden Signals
        |
        +-- Latency
        +-- Traffic
        +-- Errors
        └-- Saturation

---

# Error Rate Validation

Before deployment:

    Error Rate = 0.5%

After deployment:

    Error Rate = 8%

This is a strong signal that the deployment may have introduced a problem.

Do not rely only on Pod status.

---

# Latency Validation

Example:

    Before:
    P95 = 200 ms

    After:
    P95 = 1.5 sec

The application may be technically healthy but experiencing a performance regression.

Investigate before promoting the release.

---

# Restart Count Validation

After deployment:

    kubectl get pods

Check:

    RESTARTS

Example:

    payment
        |
        ↓
    Restart Count = 0

Unexpected increasing restart counts should be investigated.

---

# Resource Validation

Check:

    CPU
    Memory

Command:

    kubectl top pods

Possible problems:

    CPU Throttling
    Memory Pressure
    OOMKilled
    Resource Limits Too Low
    Resource Requests Too Low

---

# HPA Validation

If Horizontal Pod Autoscaler is configured, verify:

    Current Replicas
    Desired Replicas
    CPU Utilization
    Memory Utilization

Command:

    kubectl get hpa

A deployment can be healthy initially but fail under increased traffic if autoscaling is not working correctly.

---

# Service-to-Service Validation

For microservices:

    User
      |
      ↓
    Product
      |
      ↓
    Cart
      |
      ↓
    Orders
      |
      ↓
    Payment
      |
      ↓
    Inventory

Validate critical service communication after deployment.

---

# Microservices Validation

Check:

    DNS Resolution
    Service Discovery
    Network Connectivity
    Ports
    Authentication
    API Compatibility
    Timeouts
    Retries

A single service may be healthy while communication between services is broken.

---

# RabbitMQ Validation

If the application uses RabbitMQ:

    Producer
        |
        ↓
    RabbitMQ
        |
        ↓
    Consumer

Validate:

    Connection
    Queue
    Message Publishing
    Message Consumption
    Consumer Health

Monitor queue depth where appropriate.

---

# Cache Validation

If Redis or another cache is used:

    Application
        |
        ↓
    Cache
        |
        ↓
    Database

Validate:

    Connection
    Authentication
    Read
    Write
    Timeout

Avoid making cache health cause unnecessary application restarts.

---

# External Dependency Validation

If the application depends on external APIs:

    Application
        |
        ↓
    External API

Validate:

    Connectivity
    Authentication
    Response
    Timeout
    Error Handling

Do not make external dependency failures automatically restart healthy application processes.

---

# Post-Deployment Log Validation

Search logs for:

    ERROR
    Exception
    WARN
    Timeout
    Connection Refused
    HTTP 500
    HTTP 502
    HTTP 503
    HTTP 504

Compare with the previous deployment when possible.

---

# ELK Validation Flow

    Application
        |
        ↓
    Logs
        |
        ↓
    Logstash
        |
        ↓
    Elasticsearch
        |
        ↓
    Kibana
        |
        ↓
    Investigate

Look for abnormal patterns after deployment.

---

# Prometheus Validation Flow

    Application
        |
        ↓
    Metrics
        |
        ↓
    Prometheus
        |
        ↓
    Grafana
        |
        ↓
    Deployment Comparison

Compare:

    Error Rate
    Latency
    Request Rate
    Resource Usage
    Pod Availability

---

# Deployment Comparison

A useful validation technique is:

    Before Deployment
        |
        ↓
    Capture Baseline
        |
        ↓
    Deploy
        |
        ↓
    Validate
        |
        ↓
    Compare With Baseline

Example:

    Error Rate
    Before = 0.5%
    After  = 0.6%

Likely acceptable depending on requirements.

But:

    Before = 0.5%
    After  = 10%

Requires immediate investigation.

---

# Automated Post-Deployment Validation

Automation can perform:

    Rollout Check
    Pod Check
    Health Check
    Smoke Test
    API Validation
    Metrics Validation

Pipeline:

    Deploy
      |
      ↓
    Rollout Status
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
      +------ Pass ------→ Continue
      |
      +------ Fail ------→ Rollback

---

# GitHub Actions Post-Deployment Validation

Conceptual pipeline:

    GitHub Actions
        |
        ↓
    Deploy
        |
        ↓
    kubectl rollout status
        |
        ↓
    kubectl get pods
        |
        ↓
    Health Endpoint
        |
        ↓
    Smoke Test
        |
        ↓
    Success / Failure

---

# Example CI/CD Validation Logic

Conceptually:

    Deploy
      |
      ↓
    Wait For Rollout
      |
      ↓
    Check Pods
      |
      ↓
    Check Health Endpoint
      |
      ↓
    Run Smoke Tests
      |
      ↓
    Check Metrics
      |
      +------ Healthy ------→ Promote
      |
      +------ Unhealthy ----→ Rollback

---

# Rollout Validation

Command:

    kubectl rollout status deployment/payment --timeout=5m

If successful:

    Continue

If timeout:

    Stop Pipeline
        |
        ↓
    Investigate
        |
        ↓
    Rollback If Required

---

# Pod Validation Script Concept

Conceptually:

    kubectl get pods

Check:

    All Expected Pods Running
    All Expected Pods Ready
    No Unexpected Restarts

If validation fails:

    Exit With Failure

This causes the CI/CD pipeline to stop.

---

# Health Endpoint Validation Script Concept

Conceptually:

    curl -f http://application/health

If HTTP request succeeds:

    Continue

If request fails:

    Pipeline Failure

The exact validation command should match the deployment environment.

---

# Smoke Test Validation

Example flow:

    Health
        |
        ↓
    Login
        |
        ↓
    Critical API
        |
        ↓
    Database Operation
        |
        ↓
    Service Communication
        |
        ↓
    Result

If a critical step fails:

    Deployment = Failed

---

# Post-Deployment Validation Timeout

Validation should have a reasonable timeout.

Example:

    Deployment
        |
        ↓
    Wait 5 Minutes
        |
        ↓
    Validate

If validation takes too long:

    Timeout
        |
        ↓
    Stop Promotion
        |
        ↓
    Investigate

Do not allow CI/CD pipelines to wait indefinitely.

---

# Validation Failure

If post-deployment validation fails:

    Deployment
        |
        ↓
    Validation
        |
        ↓
    Failure
        |
        ↓
    Stop Promotion
        |
        ↓
    Investigate
        |
        +------ Fix Forward
        |
        └------ Rollback

---

# Rollback Decision

Rollback should be considered when:

    Critical API Is Broken
    Error Rate Is High
    Application Is Unavailable
    Pods Keep Restarting
    Health Checks Fail
    Database Compatibility Is Broken
    Critical Business Function Fails
    Severe Performance Regression Occurs

---

# Rollback vs Fix Forward

Rollback:

    New Version
        |
        ↓
    Problem
        |
        ↓
    Previous Stable Version

Fix Forward:

    New Version
        |
        ↓
    Problem
        |
        ↓
    Fix
        |
        ↓
    New Build
        |
        ↓
    Redeploy

The choice depends on:

    Severity
    Root Cause
    Rollback Safety
    Database Changes
    Business Impact
    Time To Fix

---

# Rollback Validation

Rollback should also be validated.

Flow:

    Rollback
        |
        ↓
    Rollout Status
        |
        ↓
    Pod Health
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Metrics
        |
        ↓
    Confirm Recovery

A rollback is not complete until the previous version is confirmed healthy.

---

# ArgoCD Post-Deployment Validation

GitOps flow:

    Git
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    Kubernetes Resources
      |
      ↓
    Health Status
      |
      ↓
    Application Status

Validate:

    Sync Status
    Health Status
    Pods
    Services
    Ingress
    Application Behavior

---

# ArgoCD Sync Validation

Expected:

    Sync Status = Synced

If:

    Sync Status = OutOfSync

investigate:

    Git Configuration
    Kubernetes State
    Sync Errors
    Resource Changes

---

# ArgoCD Health Validation

Expected:

    Health = Healthy

Possible states may indicate:

    Progressing
    Degraded
    Missing
    Suspended

A deployment should not automatically be considered successful if the application remains degraded.

---

# Production Validation Checklist

    Deployment rollout successful
        |
        ↓
    Desired replicas available
        |
        ↓
    Pods Running
        |
        ↓
    Pods Ready
        |
        ↓
    Health probes passing
        |
        ↓
    Services healthy
        |
        ↓
    Endpoints available
        |
        ↓
    Load Balancer healthy
        |
        ↓
    Health endpoint successful
        |
        ↓
    Smoke tests successful
        |
        ↓
    Logs normal
        |
        ↓
    Metrics normal
        |
        ↓
    Business validation successful
        |
        ↓
    Release approved

---

# Post-Deployment Validation Checklist

## Infrastructure

    EC2 / EKS Healthy
    Nodes Healthy
    ALB Healthy
    Network Working

## Kubernetes

    Deployment Successful
    ReplicaSet Healthy
    Pods Running
    Pods Ready
    No Unexpected Restarts

## Application

    Health Endpoint Works
    Readiness Works
    Critical APIs Work
    Dependencies Work

## Networking

    Service Works
    Endpoints Available
    Ingress Works
    ALB Works
    DNS Works
    TLS Works

## Monitoring

    Error Rate Normal
    Latency Normal
    CPU Normal
    Memory Normal
    Logs Normal

## Business

    Critical User Flow Works
    Transactions Work
    Data Is Correct
    Notifications Work

---

# Post-Deployment Validation for EKS

A typical EKS flow:

    GitHub
       |
       ↓
    GitHub Actions
       |
       ↓
    ECR
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
       ↓
    Pods
       |
       ↓
    Service
       |
       ↓
    ALB
       |
       ↓
    Users

Validation:

    EKS Nodes
    Pods
    Probes
    Service
    Endpoints
    ALB
    Application
    Metrics
    Logs

---

# Post-Deployment Validation for Microservices

For a microservices platform:

    User
      |
      ↓
    Product
      |
      ↓
    Cart
      |
      ↓
    Orders
      |
      ↓
    Payment
      |
      ↓
    Inventory
      |
      ↓
    Notification

Validate:

    Each Service
    Service-to-Service Communication
    Database Connections
    Message Queues
    External Dependencies
    Critical Business Flow

---

# Version Verification

A deployment should confirm:

    Expected Version
        |
        ↓
    Running Version

Example:

    Expected:
    payment:1.4.7

    Running:
    payment:1.4.7

If the running version is different:

    Investigate Deployment Configuration

---

# GitOps Version Verification

Flow:

    Git Commit
        |
        ↓
    Manifest Update
        |
        ↓
    ArgoCD Sync
        |
        ↓
    Kubernetes
        |
        ↓
    Running Image
        |
        ↓
    Verify Version

This provides deployment traceability.

---

# Post-Deployment Validation and Observability

Observability helps determine whether the release is actually healthy.

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

Combined:

    Metrics
        +
    Logs
        +
    Health Checks
        +
    Smoke Tests
        =
    Strong Validation

---

# Deployment Success Criteria

A deployment can be considered successful when:

    Rollout Completed
    Pods Healthy
    Pods Ready
    Health Checks Passing
    Services Available
    ALB Healthy
    Critical APIs Working
    Smoke Tests Passing
    Error Rate Normal
    Latency Normal
    Logs Normal
    Business Validation Successful

---

# Deployment Failure Criteria

Deployment should be considered failed when:

    Rollout Fails
    Pods Are Not Ready
    Containers Keep Restarting
    Health Checks Fail
    Critical APIs Return Errors
    ALB Targets Are Unhealthy
    Smoke Tests Fail
    Error Rate Increases Significantly
    Latency Increases Significantly
    Critical Business Function Fails

---

# Production Validation Flow

    Deployment
        |
        ↓
    Rollout Status
        |
        ↓
    Pod Health
        |
        ↓
    Readiness
        |
        ↓
    Service
        |
        ↓
    ALB
        |
        ↓
    Health Endpoint
        |
        ↓
    Smoke Test
        |
        ↓
    Metrics
        |
        ↓
    Logs
        |
        ↓
    Business Validation
        |
        ↓
    Decision

Decision:

    Healthy
        |
        ↓
    Promote

or:

    Unhealthy
        |
        ↓
    Rollback / Fix

---

# Real-World Example

Suppose version 1.4.7 is deployed.

Expected:

    payment:1.4.7

Validation:

    Deployment
        |
        ↓
    3/3 Pods Ready
        |
        ↓
    Health Endpoint = 200
        |
        ↓
    Readiness = Passing
        |
        ↓
    ALB Targets = Healthy
        |
        ↓
    Smoke Test = Pass
        |
        ↓
    Error Rate = Normal
        |
        ↓
    Latency = Normal
        |
        ↓
    Logs = Normal

Result:

    Deployment Successful

---

# Real-World Failure Example

Version 1.4.7 is deployed.

Validation:

    Deployment = Successful
    Pods = Running
    Pods = Ready

But:

    Smoke Test = Failed
    HTTP 500 = Increased
    Latency = Increased

Result:

    Deployment Should Not Be Promoted

Action:

    Stop Promotion
        |
        ↓
    Investigate
        |
        ↓
    Rollback If Required

---

# Real-World Readiness Failure

Deployment:

    Version 1.4.7

Pods:

    2/3 Ready

One Pod:

    Readiness Probe Failed

Result:

    Pod Should Not Receive Normal Service Traffic

Investigate:

    Application Logs
    Health Endpoint
    Dependencies
    Configuration
    Resource Usage

---

# Real-World ALB Failure

Pods:

    3/3 Ready

ALB:

    1/3 Healthy

Possible causes:

    Wrong Health Path
    Wrong Port
    Target Configuration
    Security Group
    Application Response
    Ingress Configuration

Do not consider the deployment fully healthy until the traffic path is validated.

---

# Real-World Performance Regression

Before deployment:

    P95 Latency = 200 ms

After deployment:

    P95 Latency = 2 seconds

Pods:

    Healthy

This demonstrates:

    Pod Health
        ≠
    Performance Health

Investigate:

    Application Code
    Database Queries
    External Calls
    Resource Usage
    Network
    Connection Pools

---

# Real-World Business Failure

After deployment:

    Pods = Healthy
    Health = 200
    Smoke Test = Pass

But:

    Orders Are Not Being Created

Possible causes:

    Business Logic
    Database
    Message Queue
    Configuration
    Service Communication

This is why business validation is important.

---

# Post-Deployment Validation Best Practices

- Always validate the rollout
- Verify expected replica count
- Verify Pod readiness
- Check restart counts
- Check health probes
- Verify Service endpoints
- Validate Load Balancer health
- Test critical APIs
- Run smoke tests
- Validate application version
- Check logs
- Check Prometheus metrics
- Check Grafana dashboards
- Validate critical business flows
- Compare metrics with baseline
- Set validation timeouts
- Automate repeatable checks
- Stop promotion when critical checks fail
- Have a rollback strategy
- Validate the rollback itself

---

# Common Mistakes

## Mistake 1: Checking Only Pod Status

    kubectl get pods

does not prove the complete application is healthy.

Also check:

    Service
    ALB
    Application
    Metrics
    Logs
    Business Flow

---

# Common Mistakes

## Mistake 2: Checking Only HTTP 200

HTTP 200 from:

    /health

does not prove every business function works.

Use:

    Health Checks
    Smoke Tests
    Functional Tests
    Business Validation

---

# Common Mistakes

## Mistake 3: Ignoring Logs

A deployment can show:

    Pods = Running

while logs show:

    Database Errors
    Timeout Errors
    Application Exceptions

Always inspect logs after important deployments.

---

# Common Mistakes

## Mistake 4: Ignoring Metrics

A deployment can be technically successful while causing:

    High Latency
    High Error Rate
    High CPU
    High Memory
    Increased Restarts

Monitor production behavior.

---

# Common Mistakes

## Mistake 5: No Rollback Decision

A production deployment should have clear criteria for:

    Continue
    Stop
    Rollback

Do not wait for a major incident before deciding what constitutes deployment failure.

---

# Common Mistakes

## Mistake 6: No Baseline

Without a baseline:

    Before = Unknown
    After = Unknown

It is difficult to determine whether performance or reliability changed.

Capture important metrics before deployment where possible.

---

# Common Mistakes

## Mistake 7: Validation Takes Too Long

Post-deployment validation should be fast enough to support safe releases.

Use:

    Automated Checks
    Focused Smoke Tests
    Health Checks
    Monitoring

Avoid unnecessary long-running validation during every deployment.

---

# Common Mistakes

## Mistake 8: Hardcoding Environment Values

Avoid hardcoding:

    Production URLs
    Credentials
    Secrets
    Environment-Specific Configuration

Use environment-aware configuration.

---

# Common Mistakes

## Mistake 9: Exposing Secrets in Logs

Never print:

    Passwords
    Tokens
    API Keys
    Secret Values

Post-deployment validation must be safe for CI/CD logs.

---

# Post-Deployment Validation Interview Questions

## Basic

1. What is post-deployment validation?

2. Why is post-deployment validation important?

3. What checks do you perform after deployment?

4. How do you verify a Kubernetes deployment succeeded?

5. How do you check whether Pods are healthy?

6. How do you verify application health?

7. What is a smoke test?

8. What is the difference between deployment validation and application validation?

9. Why should you check logs after deployment?

10. Why should you monitor metrics after deployment?

---

# Post-Deployment Validation Interview Questions

## Intermediate

11. How do you validate a Kubernetes deployment?

12. How do you verify that all Pods are ready?

13. How do you verify Service endpoints?

14. How do you validate an ALB after deployment?

15. How do you validate a health endpoint?

16. How do you validate application version?

17. How do you perform smoke testing?

18. How do you validate microservice communication?

19. How do you validate database connectivity?

20. How do you validate deployment in GitHub Actions?

---

# Post-Deployment Validation Interview Questions

## Advanced

21. How would you design an automated post-deployment validation stage?

22. What criteria would cause you to automatically rollback?

23. How would you compare application health before and after deployment?

24. How would you validate a Canary deployment?

25. How would you validate a Blue-Green deployment?

26. How would you validate a zero-downtime deployment?

27. How would you validate an EKS application behind an ALB?

28. How would you use Prometheus and Grafana for post-deployment validation?

29. How would you use ELK to investigate a deployment failure?

30. How would you validate a microservices deployment end to end?

---

# Scenario-Based Interview Question

## Deployment Succeeded but Users Receive 503

Check:

    kubectl get pods

Then:

    kubectl get svc

Then:

    kubectl get endpoints

Then:

    kubectl describe pod <pod>

Check:

    Readiness
    Service Selector
    Target Port
    ALB Health
    Application Logs

Possible root causes:

    Pods Not Ready
    No Service Endpoints
    Wrong Port
    Wrong Target Port
    ALB Health Check Failure
    Application Error

---

# Scenario-Based Interview Question

## Pods Are Healthy but API Returns 500

Check:

    Application Logs
    Database
    Configuration
    Dependencies
    Recent Code Changes

Health probes may still pass because:

    /health = 200

while:

    /api/order = 500

Therefore test critical APIs separately.

---

# Scenario-Based Interview Question

## Deployment Increased Latency

Before:

    P95 = 200 ms

After:

    P95 = 2 seconds

Check:

    CPU
    Memory
    Database
    Network
    Application Logs
    External APIs
    Connection Pools

Compare metrics with the previous release.

---

# Scenario-Based Interview Question

## Canary Looks Healthy but Business Transactions Fail

Technical:

    Pods = Healthy
    Health = 200
    Error Rate = Low

Business:

    Orders = Failing

Action:

    Stop Promotion

Then investigate:

    Business Logic
    Database
    Payment
    Inventory
    Message Queue

Technical health does not replace business validation.

---

# Scenario-Based Interview Question

## Deployment Is Stuck

Run:

    kubectl rollout status deployment/<deployment-name>

Then:

    kubectl get pods

Then:

    kubectl describe pod <pod-name>

Check:

    Pending
    ImagePullBackOff
    CrashLoopBackOff
    Readiness Probe Failure
    Insufficient Resources

Fix the root cause before proceeding.

---

# Scenario-Based Interview Question

## New Version Is Not Running

Expected:

    payment:1.4.7

Actual:

    payment:1.4.6

Check:

    Deployment Manifest
    ArgoCD Sync
    Image Tag
    Image Digest
    ReplicaSet
    Pod Image

Trace:

    Git
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Pod

---

# Scenario-Based Interview Question

## Smoke Test Failed

Flow:

    Deployment
        |
        ↓
    Health Check = Pass
        |
        ↓
    Smoke Test = Fail

Do not automatically assume Kubernetes is broken.

Check:

    Application
    API
    Database
    Authentication
    Dependencies
    Configuration

Then:

    Stop Promotion
        |
        ↓
    Investigate
        |
        ↓
    Rollback / Fix

---

# Post-Deployment Validation Runbook

## Step 1

Verify deployment:

    kubectl rollout status deployment/<deployment-name>

## Step 2

Verify Pods:

    kubectl get pods

## Step 3

Verify readiness:

    kubectl describe pod <pod-name>

## Step 4

Verify Service:

    kubectl get svc

## Step 5

Verify endpoints:

    kubectl get endpoints

## Step 6

Test health endpoint:

    curl http://<service>:<port>/health

## Step 7

Run smoke tests.

## Step 8

Check logs.

## Step 9

Check Prometheus and Grafana.

## Step 10

Check ELK.

## Step 11

Validate critical business flow.

## Step 12

Promote or rollback.

---

# Complete CI/CD Post-Deployment Stage

    Code
      |
      ↓
    GitHub
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
    ECR
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
      ↓
    Rollout Validation
      |
      ↓
    Pod Validation
      |
      ↓
    Health Validation
      |
      ↓
    Service Validation
      |
      ↓
    ALB Validation
      |
      ↓
    Smoke Tests
      |
      ↓
    Metrics
      |
      ↓
    Logs
      |
      ↓
    Business Validation
      |
      +------ Pass ------→ Release Successful
      |
      +------ Fail ------→ Rollback / Fix

---

# Complete Production Deployment Validation

    Developer
        |
        ↓
    GitHub
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
    Docker Image
        |
        ↓
    ECR
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
    Startup Probe
        |
        ↓
    Readiness Probe
        |
        ↓
    Service
        |
        ↓
    ALB
        |
        ↓
    Application
        |
        +-- Health Check
        +-- Smoke Test
        +-- Functional Test
        |
        ↓
    Monitoring
        |
        +-- Prometheus
        +-- Grafana
        └-- ELK
        |
        ↓
    Business Validation
        |
        ↓
    Release Decision

---

# Final Deployment Decision

After validation:

    All Checks Pass
        |
        ↓
    Application Healthy
        |
        ↓
    Metrics Normal
        |
        ↓
    Logs Normal
        |
        ↓
    Business Flow Working
        |
        ↓
    Release Successful

If critical checks fail:

    Validation Failure
        |
        ↓
    Stop Promotion
        |
        ↓
    Investigate
        |
        +------ Fix Forward
        |
        └------ Rollback

---

# Final Mental Model

Remember:

    Deployment
        |
        ↓
    Did Kubernetes Deploy?
        |
        ↓
    Are Pods Healthy?
        |
        ↓
    Are Pods Ready?
        |
        ↓
    Is the Service Working?
        |
        ↓
    Is the ALB Healthy?
        |
        ↓
    Is the Application Responding?
        |
        ↓
    Do Critical APIs Work?
        |
        ↓
    Do Smoke Tests Pass?
        |
        ↓
    Are Metrics Normal?
        |
        ↓
    Are Logs Normal?
        |
        ↓
    Does the Business Flow Work?
        |
        ↓
    Promote or Rollback

---

# Final Concept

Post-deployment validation is the final safety layer of a CI/CD deployment.

The complete principle is:

    Deploy
        |
        ↓
    Verify
        |
        ↓
    Test
        |
        ↓
    Observe
        |
        ↓
    Validate
        |
        ↓
    Decide

A production deployment should not be considered successful simply because:

    Docker Image = Built

or:

    Kubernetes Pod = Running

or:

    Deployment = Rolled Out

The real success criteria are:

    Deployment Successful
        +
    Pods Healthy
        +
    Pods Ready
        +
    Services Available
        +
    Load Balancer Healthy
        +
    Application Healthy
        +
    Smoke Tests Passing
        +
    Metrics Normal
        +
    Logs Normal
        +
    Business Validation Successful

Therefore:

    Successful Deployment
        =
    Technical Validation
        +
    Application Validation
        +
    Observability Validation
        +
    Business Validation

This provides a reliable foundation for:

    Rolling Deployments
        +
    Blue-Green Deployments
        +
    Canary Deployments
        +
    GitOps
        +
    Zero-Downtime Deployments
        +
    Automated Rollbacks
        +
    Production Reliability