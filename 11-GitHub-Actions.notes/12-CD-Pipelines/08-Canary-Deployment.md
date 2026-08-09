# Canary Deployment

Canary Deployment is a deployment strategy where a new application version is released to a small percentage of users or traffic first.

The new version is monitored before gradually increasing traffic.

Example:

    Version 1.4.6
        |
        ↓
    95% Traffic

    Version 1.4.7
        |
        ↓
     5% Traffic

If the new version is healthy:

    75% → 1.4.6
    25% → 1.4.7

Then:

    50% → 1.4.6
    50% → 1.4.7

Finally:

    100% → 1.4.7

If problems occur:

    100% → 1.4.6

---

# Why Canary Deployment Is Used

Canary deployment reduces the risk of releasing a new version to all users at once.

Instead of:

    Old Version
        |
        ↓
    100% Users

changing directly to:

    New Version
        |
        ↓
    100% Users

Canary uses:

    Old Version
        |
        ↓
    95% Users

    New Version
        |
        ↓
     5% Users

The new version is exposed gradually.

---

# Canary Deployment Mental Model

Remember:

    Deploy New Version
          |
          ↓
    Small Traffic Percentage
          |
          ↓
       Monitor
          |
          +------ Healthy ------→ Increase Traffic
          |
          +------ Unhealthy ----→ Rollback
                                      |
                                      ↓
                                Previous Version

---

# Canary vs Blue-Green

Blue-Green:

    Blue = 100%
    Green = 0%

Then:

    Blue = 0%
    Green = 100%

Canary:

    Blue = 95%
    Green = 5%

Then gradually:

    Blue = 75%
    Green = 25%

Then:

    Blue = 50%
    Green = 50%

Then:

    Blue = 0%
    Green = 100%

The main difference is gradual traffic exposure.

---

# Canary vs Rolling Deployment

Rolling deployment gradually replaces application instances.

Example:

    Old Pods
      |
      ↓
    New Pods
      |
      ↓
    Old Pods Removed

Canary deployment focuses on traffic percentage:

    95% → Old
     5% → New

The two strategies can also be combined depending on the platform.

---

# Canary Deployment Architecture

Basic architecture:

    Users
      |
      ↓
    Load Balancer
      |
      +------------→ Stable
      |               |
      |               └── v1.4.6
      |
      └------------→ Canary
                      |
                      └── v1.4.7

Initially:

    95% → Stable
     5% → Canary

---

# Stable Environment

The Stable environment contains the currently trusted production version.

Example:

    Stable
       |
       ↓
    v1.4.6
       |
       ↓
    95% Traffic

The Stable version remains available during the Canary release.

---

# Canary Environment

The Canary environment contains the new version.

Example:

    Canary
       |
       ↓
    v1.4.7
       |
       ↓
     5% Traffic

The Canary version receives limited traffic while the deployment is evaluated.

---

# Canary Deployment Flow

Typical process:

    1. Deploy new version
    2. Route small percentage of traffic
    3. Monitor Canary
    4. Compare Canary with Stable
    5. Increase traffic
    6. Continue monitoring
    7. Move 100% traffic to Canary
    8. Make Canary the Stable version

If problems occur:

    Stop rollout
        |
        ↓
    Route traffic to Stable
        |
        ↓
    Investigate Canary

---

# Complete Canary Flow

    Stable
    v1.4.6
       |
       ↓
    95% Traffic

    Canary
    v1.4.7
       |
       ↓
     5% Traffic

            |
            ↓
         Monitor

            |
            ↓
       Healthy?

       /       \
     Yes        No
      |          |
      ↓          ↓
 Increase      Rollback
 Traffic         |
      |           ↓
      ↓        Stable
    Monitor
      |
      ↓
    100% Canary

---

# Initial Canary Deployment

Before release:

    Stable:
        v1.4.6

    Canary:
        Not deployed

Deploy:

    Canary:
        v1.4.7

Traffic:

    Stable = 100%
    Canary = 0%

---

# First Canary Stage

Start with a small percentage.

Example:

    Stable:
        95%

    Canary:
         5%

This limits the blast radius if the new release has problems.

---

# Canary Monitoring

Monitor the Canary version for:

    HTTP 5xx
    Error Rate
    Latency
    Request Rate
    CPU
    Memory
    Pod Restarts
    Application Logs
    Database Health
    Dependency Health
    Availability

Compare Canary against Stable.

---

# Canary Metrics

Example:

    Stable:
        Error Rate = 0.5%
        Latency = 120ms

    Canary:
        Error Rate = 0.6%
        Latency = 125ms

Canary appears healthy.

Another example:

    Stable:
        Error Rate = 0.5%

    Canary:
        Error Rate = 8%

This indicates a serious problem.

The rollout should stop and the Canary should be investigated or rolled back.

---

# Canary Success Criteria

Success criteria should be defined before deployment.

Example:

    Error Rate < 1%
    HTTP 5xx < Defined Threshold
    Latency < Defined Threshold
    No CrashLoopBackOff
    No Significant Pod Restarts
    Health Checks Passing
    Smoke Tests Passing

The exact thresholds depend on the application.

---

# Canary Failure Criteria

Possible failure conditions:

    High Error Rate
    High Latency
    HTTP 5xx Increase
    Failed Health Checks
    CrashLoopBackOff
    Memory Exhaustion
    CPU Saturation
    Database Errors
    Dependency Failures
    Failed Smoke Tests
    Security Issue

If failure criteria are reached:

    Stop Canary Rollout

---

# Canary Traffic Progression

A common progression can be:

    Stage 1:
       5%

    Stage 2:
      10%

    Stage 3:
      25%

    Stage 4:
      50%

    Stage 5:
      100%

The exact percentages depend on the organization's risk tolerance.

---

# Canary Traffic Strategy

Example:

    100% Stable
        |
        ↓
     5% Canary
        |
        ↓
    Monitor
        |
        ↓
    10% Canary
        |
        ↓
    Monitor
        |
        ↓
    25% Canary
        |
        ↓
    Monitor
        |
        ↓
    50% Canary
        |
        ↓
    Monitor
        |
        ↓
    100% Canary

---

# Canary Rollback

Suppose:

    Stable = v1.4.6
    Canary = v1.4.7

Traffic:

    90% → Stable
    10% → Canary

Canary begins returning errors.

Rollback:

    100% → Stable
     0% → Canary

The old version continues serving production traffic.

---

# Canary Rollback Flow

    Canary
    v1.4.7
       |
       ↓
    High Error Rate
       |
       ↓
    Stop Rollout
       |
       ↓
    Route Traffic to Stable
       |
       ↓
    Stable
    v1.4.6
       |
       ↓
    Monitor
       |
       ↓
    Recovery

---

# Canary With Kubernetes

Kubernetes can run Stable and Canary Deployments simultaneously.

Example:

    Stable Deployment
        |
        └── v1.4.6


    Canary Deployment
        |
        └── v1.4.7

Both versions run at the same time.

Traffic routing determines how many requests reach each version.

---

# Kubernetes Canary Deployments

Stable:

    apiVersion: apps/v1
    kind: Deployment

    metadata:
      name: payment-stable

    spec:
      replicas: 9

      selector:
        matchLabels:
          app: payment
          version: stable

      template:
        metadata:
          labels:
            app: payment
            version: stable

        spec:
          containers:
            - name: payment
              image: payment:1.4.6

Canary:

    apiVersion: apps/v1
    kind: Deployment

    metadata:
      name: payment-canary

    spec:
      replicas: 1

      selector:
        matchLabels:
          app: payment
          version: canary

      template:
        metadata:
          labels:
            app: payment
            version: canary

        spec:
          containers:
            - name: payment
              image: payment:1.4.7

---

# Kubernetes Service With Canary

A Service can select both Stable and Canary Pods.

Example:

    apiVersion: v1
    kind: Service

    metadata:
      name: payment

    spec:
      selector:
        app: payment

Both versions may receive traffic.

However, a standard Kubernetes Service does not provide precise percentage-based traffic splitting by itself.

For controlled percentage-based Canary traffic, additional routing capabilities are commonly used.

---

# Kubernetes Labels

Stable:

    app: payment
    version: stable

Canary:

    app: payment
    version: canary

Labels help identify different application versions.

Example:

    kubectl get pods --show-labels

---

# Kubernetes Canary Scaling

One simple approach is to run different numbers of Stable and Canary Pods.

Example:

    Stable:
        19 Pods

    Canary:
        1 Pod

Conceptually:

    Stable ≈ 95%
    Canary ≈ 5%

However, Pod count is not a guaranteed traffic percentage because request distribution depends on networking and load-balancing behavior.

For precise traffic percentages, use a traffic-management mechanism.

---

# Canary With Ingress

Ingress can be used to route traffic between Stable and Canary.

Architecture:

    Users
       |
       ↓
    Ingress
       |
       +------------→ Stable
       |
       └------------→ Canary

Traffic rules determine the percentage or conditions for routing.

---

# Canary With NGINX Ingress

NGINX Ingress supports Canary routing features.

Conceptually:

    User
      |
      ↓
    NGINX Ingress
      |
      +---- 95% ----→ Stable
      |
      +----- 5% ----→ Canary

Canary routing can be based on supported configuration mechanisms.

---

# Canary By Header

Canary traffic can be targeted to specific requests.

Example:

    Header:

    X-Canary: true

Requests containing the header can be routed to Canary.

Flow:

    User
      |
      ↓
    X-Canary: true
      |
      ↓
    Ingress
      |
      ↓
    Canary

This can be useful for controlled testing.

---

# Canary By Cookie

A cookie can also be used to route selected users to the Canary version.

Example:

    Cookie:
      canary=true

Flow:

    User
      |
      ↓
    Cookie
      |
      ↓
    Ingress
      |
      ↓
    Canary

This can be useful when testing the new version with selected users.

---

# Canary By User Group

Canary can be targeted to specific groups.

Example:

    Internal Employees → Canary

    External Users → Stable

Flow:

    Users
      |
      +-- Internal → Canary
      |
      └-- External → Stable

This allows internal users to validate the release before broader exposure.

---

# Canary By Geography

Traffic can sometimes be routed based on geography.

Example:

    Region A → Canary
    Region B → Stable

After validation:

    Region A → Canary
    Region B → Canary

This can reduce the blast radius of a release.

---

# Canary With AWS ALB

AWS ALB can participate in traffic routing between application versions.

Conceptual architecture:

    Internet
       |
       ↓
    Route 53
       |
       ↓
    ALB
       |
       +------------→ Stable Target Group
       |
       └------------→ Canary Target Group

Traffic routing can be configured according to the capabilities and architecture being used.

---

# ALB Target Groups

Example:

    ALB
      |
      +-- Stable Target Group
      |      |
      |      └── v1.4.6
      |
      └-- Canary Target Group
             |
             └── v1.4.7

The routing layer determines which target group receives requests.

---

# Canary With EKS

Typical architecture:

    Route 53
       |
       ↓
    ALB
       |
       ↓
    EKS
       |
       +-- Stable Pods
       |      |
       |      └── v1.4.6
       |
       └-- Canary Pods
              |
              └── v1.4.7

Traffic is gradually shifted to Canary.

---

# Canary With Argo Rollouts

Argo Rollouts is designed to provide advanced deployment strategies for Kubernetes, including Canary and Blue-Green deployments.

Conceptual flow:

    Git
      |
      ↓
    ArgoCD
      |
      ↓
    Argo Rollouts
      |
      ↓
    EKS
      |
      +-- Stable
      |
      └-- Canary

Argo Rollouts can coordinate progressive delivery and analysis.

---

# Argo Rollouts Canary Flow

Conceptually:

    Deploy Canary
         |
         ↓
       5%
         |
         ↓
      Analysis
         |
         ↓
       10%
         |
         ↓
      Analysis
         |
         ↓
       25%
         |
         ↓
      Analysis
         |
         ↓
       50%
         |
         ↓
      Analysis
         |
         ↓
      100%

If analysis fails:

    Abort
      |
      ↓
    Stable

---

# Argo Rollouts

A conceptual Rollout configuration can define:

    replicas: 10

    strategy:
      canary:
        steps:
          - setWeight: 5
          - pause: {}
          - setWeight: 10
          - pause: {}
          - setWeight: 25
          - pause: {}
          - setWeight: 50
          - pause: {}

The exact configuration depends on the traffic-management setup.

---

# Canary Analysis

Canary analysis compares the new version against defined metrics.

Example:

    Canary
       |
       ↓
    Prometheus
       |
       ↓
    Metrics
       |
       ↓
    Analysis
       |
       +------ Pass ------→ Increase Traffic
       |
       +------ Fail ------→ Abort

---

# Prometheus and Canary

Prometheus can provide metrics such as:

    Request Rate
    Error Rate
    Latency
    CPU
    Memory

Example:

    Canary Error Rate
          |
          ↓
      Prometheus
          |
          ↓
       Analysis
          |
          ↓
    Pass / Fail

---

# Grafana and Canary

Grafana can visualize Canary metrics.

Example:

    Prometheus
        |
        ↓
    Grafana
        |
        +-- Stable Error Rate
        +-- Canary Error Rate
        +-- Stable Latency
        +-- Canary Latency
        +-- Request Rate

This helps engineers compare versions during rollout.

---

# ELK and Canary

ELK can help analyze application logs.

Example:

    Canary Pods
        |
        ↓
    Logs
        |
        ↓
    ELK
        |
        ↓
    Error Analysis

Look for:

    Exceptions
    HTTP Errors
    Database Errors
    Dependency Failures
    Authentication Errors

---

# Canary Observability Stack

A DevOps observability setup can use:

    Prometheus
        |
        ↓
    Metrics

    Grafana
        |
        ↓
    Dashboards

    ELK
        |
        ↓
    Logs

During Canary:

    Stable Metrics
          +
    Canary Metrics
          +
    Stable Logs
          +
    Canary Logs

Compare them before increasing traffic.

---

# Canary Error Rate Comparison

Example:

    Stable:
        Error Rate = 0.4%

    Canary:
        Error Rate = 0.5%

Canary may be acceptable depending on the defined threshold.

Another example:

    Stable:
        Error Rate = 0.4%

    Canary:
        Error Rate = 7.5%

Stop the rollout.

---

# Canary Latency Comparison

Example:

    Stable:
        P95 = 150ms

    Canary:
        P95 = 160ms

Potentially acceptable depending on the defined threshold.

Problem:

    Stable:
        P95 = 150ms

    Canary:
        P95 = 900ms

Stop and investigate.

---

# Canary Analysis Metrics

Useful metrics include:

    HTTP 5xx Rate
    HTTP 4xx Rate
    Request Success Rate
    P50 Latency
    P95 Latency
    P99 Latency
    CPU Usage
    Memory Usage
    Pod Restarts
    Request Rate
    Database Errors

Choose metrics that reflect actual application health.

---

# Canary Health Checks

Before increasing traffic:

    Pod Ready?
        |
        ↓
    Service Healthy?
        |
        ↓
    Application Healthy?
        |
        ↓
    Smoke Test Passed?
        |
        ↓
    Metrics Healthy?
        |
        ↓
    Increase Traffic

---

# Canary Smoke Testing

Smoke tests can include:

    GET /health

    Login

    Authentication

    Critical API

    Database Connectivity

    Service-to-Service Communication

Example:

    Canary
      |
      ↓
    Smoke Tests
      |
      +-- Pass → Continue
      |
      └-- Fail → Abort

---

# Canary With GitHub Actions

Typical flow:

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
    GitOps
       |
       ↓
    ArgoCD
       |
       ↓
    Canary Deployment
       |
       ↓
    Validation
       |
       ↓
    Progressive Traffic

---

# Canary CI/CD Pipeline

    Code
      |
      ↓
    Pull Request
      |
      ↓
    CI
      |
      +-- Build
      +-- Unit Tests
      +-- SonarQube
      +-- Trivy
      |
      ↓
    ECR
      |
      ↓
    Deploy Canary
      |
      ↓
     5%
      |
      ↓
    Monitor
      |
      ↓
    10%
      |
      ↓
    Monitor
      |
      ↓
    25%
      |
      ↓
    Monitor
      |
      ↓
    50%
      |
      ↓
    Monitor
      |
      ↓
    100%

---

# Canary With Jenkins

Jenkins can orchestrate Canary deployment stages.

Example:

    Jenkins
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
    Push Image
       |
       ↓
    Deploy Canary
       |
       ↓
    5%
       |
       ↓
    Validate
       |
       ↓
    Increase Traffic
       |
       ↓
    Monitor
       |
       ↓
    100%

---

# Canary With GitOps

GitOps keeps the desired deployment configuration in Git.

Flow:

    Git
      |
      ↓
    Canary Configuration
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      ↓
    Canary
      |
      ↓
    Traffic Management

Changes should go through version-controlled workflows.

---

# GitOps Canary Rollback

Suppose:

    Git:
      v1.4.7

Canary:

    10%

Problem occurs.

Rollback:

    Git Revert
        |
        ↓
    ArgoCD
        |
        ↓
    Stable Version
        |
        ↓
    EKS

Traffic returns to Stable.

---

# Canary and Immutable Images

Use immutable image versions.

Example:

    payment:1.4.6
    payment:1.4.7

Stable:

    payment:1.4.6

Canary:

    payment:1.4.7

Avoid:

    payment:latest

Immutable versions make deployment and rollback predictable.

---

# Canary and ECR

Example:

    ECR
      |
      └── payment
           |
           ├── 1.4.5
           ├── 1.4.6
           └── 1.4.7

Stable:

    1.4.6

Canary:

    1.4.7

After successful rollout:

    1.4.7 becomes Stable

---

# Canary and Database Changes

Database compatibility is critical.

Example:

    Stable
    v1.4.6
       |
       ↓
    Database
       ↑
       |
    Canary
    v1.4.7

Both versions may access the same database during the Canary phase.

Therefore:

    Database Changes
        |
        ↓
    Must Support
        |
        +-- Stable
        |
        └-- Canary

---

# Backward-Compatible Database Migration

Safer approach:

    Add New Column
        |
        ↓
    Keep Existing Column
        |
        ↓
    Deploy Canary
        |
        ↓
    Validate
        |
        ↓
    Increase Traffic
        |
        ↓
    Remove Old Column Later

Avoid destructive schema changes before the old version is no longer needed.

---

# Canary Database Risk

Suppose Canary expects:

    Column B

Stable expects:

    Column A

If Column A is removed immediately:

    Stable
       |
       ↓
    Failure

Therefore use:

    Expand
       |
       ↓
    Deploy
       |
       ↓
    Migrate
       |
       ↓
    Contract

---

# Canary and Configuration

Configuration should be compatible with both versions during the transition.

Check:

    Environment Variables
    ConfigMaps
    Secrets
    Feature Flags
    API Endpoints
    Database Configuration

A configuration change can be just as risky as a code change.

---

# Canary and Feature Flags

Feature flags can complement Canary deployment.

Example:

    Application v1.4.7
          |
          ↓
    Feature Flag
          |
          +-- 5% Users → New Feature
          |
          └-- 95% Users → Old Feature

This separates application deployment from feature activation.

---

# Canary and Feature Flags

A deployment can be:

    Code Deployment
        |
        ↓
    Feature Disabled
        |
        ↓
    Canary
        |
        ↓
    Validate
        |
        ↓
    Enable Feature Gradually

This provides another layer of release control.

---

# Canary User Targeting

Canary can target:

    Internal Users
    Beta Users
    Selected Customers
    Specific Geography
    Specific Headers
    Specific Cookies

Example:

    Internal Users
         |
         ↓
      Canary

    External Users
         |
         ↓
       Stable

---

# Canary Release Strategy

A safe strategy might be:

    Stage 1:
        1% Canary

    Stage 2:
        5% Canary

    Stage 3:
        10% Canary

    Stage 4:
        25% Canary

    Stage 5:
        50% Canary

    Stage 6:
        100% Canary

The exact stages should be based on traffic volume and risk.

---

# Canary Pause

After each traffic increase, pause and evaluate.

Example:

    5%
      |
      ↓
    Pause
      |
      ↓
    Analyze
      |
      ↓
    10%
      |
      ↓
    Pause
      |
      ↓
    Analyze

This prevents a faulty release from reaching all users immediately.

---

# Automated Canary Analysis

Automated analysis can evaluate metrics.

Flow:

    Canary
      |
      ↓
    Prometheus
      |
      ↓
    Analysis
      |
      +-- Success → Increase Traffic
      |
      └-- Failure → Abort

Automation reduces manual effort and improves consistency.

---

# Canary Abort

If analysis fails:

    Canary
      |
      ↓
    Analysis Failure
      |
      ↓
    Abort Rollout
      |
      ↓
    Route Traffic to Stable
      |
      ↓
    Stable
      |
      ↓
    Investigate

The Canary should not continue automatically after a failed analysis.

---

# Canary Promotion

When all stages pass:

    5%
      |
      ↓
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

At 100%:

    Canary
       |
       ↓
    Production Stable

The Canary becomes the new Stable version.

---

# Canary Promotion Flow

    Canary v1.4.7
         |
         ↓
        5%
         |
         ↓
      Analysis
         |
         ↓
        10%
         |
         ↓
      Analysis
         |
         ↓
        25%
         |
         ↓
      Analysis
         |
         ↓
        50%
         |
         ↓
      Analysis
         |
         ↓
       100%
         |
         ↓
    Promotion Complete

---

# Canary Rollback Flow

    Canary v1.4.7
         |
         ↓
        10%
         |
         ↓
    Error Rate High
         |
         ↓
    Abort Rollout
         |
         ↓
     0% Canary
         |
         ↓
    100% Stable
         |
         ↓
    Investigate
         |
         ↓
    Fix
         |
         ↓
    New Release

---

# Canary Deployment With ALB

Conceptual architecture:

    Internet
       |
       ↓
    Route 53
       |
       ↓
    ALB
       |
       +------------------+
       |                  |
       ↓                  ↓
    Stable TG          Canary TG
       |                  |
       ↓                  ↓
    EKS Stable         EKS Canary
       |                  |
       ↓                  ↓
    v1.4.6              v1.4.7

Traffic:

    95% → Stable
     5% → Canary

---

# Canary Deployment With EKS

Example:

    EKS
      |
      +-- Stable Deployment
      |      |
      |      └── 9 Pods
      |
      └-- Canary Deployment
             |
             └── 1 Pod

Traffic management determines the actual traffic percentage.

Do not assume Pod count alone guarantees exact traffic percentages.

---

# Canary Deployment With Argo Rollouts

Architecture:

    GitHub
       |
       ↓
    GitOps
       |
       ↓
    ArgoCD
       |
       ↓
    Argo Rollouts
       |
       ↓
    EKS
       |
       +-- Stable
       |
       └-- Canary
              |
              ↓
           Analysis
              |
              ↓
        Progressive Traffic

---

# Canary Observability Architecture

    Canary
       |
       +----------------→ Prometheus
       |                      |
       |                      ↓
       |                   Metrics
       |                      |
       |                      ↓
       |                   Grafana
       |
       +----------------→ ELK
                              |
                              ↓
                            Logs

Use observability to compare:

    Stable
    vs
    Canary

---

# Canary Metrics Dashboard

A useful dashboard can display:

    Stable Error Rate
    Canary Error Rate

    Stable P95 Latency
    Canary P95 Latency

    Stable Request Rate
    Canary Request Rate

    Stable CPU
    Canary CPU

    Stable Memory
    Canary Memory

This makes progressive rollout decisions easier.

---

# Canary Production Checklist

Before starting:

    [ ] Previous version is healthy
    [ ] New image is available
    [ ] Security scans passed
    [ ] Tests passed
    [ ] Database migration is compatible
    [ ] Rollback procedure is ready
    [ ] Monitoring is available
    [ ] Health checks are configured
    [ ] Traffic routing is configured
    [ ] Canary thresholds are defined

---

# Canary Validation Checklist

At every stage:

    [ ] Pods healthy
    [ ] Readiness passing
    [ ] Liveness healthy
    [ ] Application health passing
    [ ] Error rate acceptable
    [ ] Latency acceptable
    [ ] Logs normal
    [ ] Database healthy
    [ ] Dependencies healthy
    [ ] Smoke tests passing

Only then increase traffic.

---

# Canary Production Runbook

## Step 1

Verify Stable:

    v1.4.6

## Step 2

Deploy Canary:

    v1.4.7

## Step 3

Validate Canary:

    Pods
    Services
    Health Checks

## Step 4

Route:

    5% → Canary

## Step 5

Monitor:

    Metrics
    Logs
    Errors

## Step 6

Increase:

    10%
    25%
    50%
    100%

## Step 7

Promote Canary.

---

# Canary Rollback Runbook

## Step 1

Detect failure.

## Step 2

Stop traffic increase.

## Step 3

Abort Canary.

## Step 4

Route traffic back to Stable.

## Step 5

Validate Stable.

## Step 6

Monitor recovery.

## Step 7

Investigate Canary.

## Step 8

Fix the issue.

## Step 9

Create a new release.

---

# Canary Scenario

## Scenario: Canary Returns 500 Errors

Stable:

    v1.4.6
    Error Rate = 0.4%

Canary:

    v1.4.7
    Error Rate = 9%

Traffic:

    95% Stable
     5% Canary

Action:

    Stop Rollout
        |
        ↓
    Abort Canary
        |
        ↓
    100% Stable
        |
        ↓
    Investigate Logs
        |
        ↓
    Fix
        |
        ↓
    New Release

---

# Canary Scenario

## Scenario: Canary Has High Latency

Stable:

    P95 = 150ms

Canary:

    P95 = 850ms

Action:

    Stop Traffic Increase
        |
        ↓
    Analyze Application
        |
        ↓
    Check:
        CPU
        Memory
        Database
        Dependencies
        Logs
        Network
        |
        ↓
    Rollback if Necessary

---

# Canary Scenario

## Scenario: Canary Pods Crash

Canary:

    CrashLoopBackOff

Commands:

    kubectl get pods

    kubectl describe pod <pod>

    kubectl logs <pod>

    kubectl logs <pod> --previous

If the problem is caused by the new release:

    Abort Canary
        |
        ↓
    Stable Receives Traffic

---

# Canary Scenario

## Scenario: Canary Image Cannot Be Pulled

Canary:

    ImagePullBackOff

Check:

    ECR Repository
    Image Tag
    IAM Permissions
    Image Availability
    Registry Configuration

If the image is invalid:

    Abort Canary

Do not increase traffic until the image problem is resolved.

---

# Canary Scenario

## Scenario: Canary Health Check Fails

Canary:

    Readiness Probe Failed

Possible causes:

    Wrong Port
    Wrong Path
    Application Startup Failure
    Configuration Error
    Dependency Failure

Action:

    Keep Traffic on Stable
        |
        ↓
    Fix Canary
        |
        ↓
    Validate Again

---

# Canary Scenario

## Scenario: Database Error

Canary:

    Database Connection Errors

Check:

    Connection String
    Credentials
    Network
    Security Group
    Database Capacity
    Schema Compatibility

If Canary is causing production impact:

    Abort Canary
        |
        ↓
    Stable

---

# Canary Scenario

## Scenario: Security Vulnerability

Suppose Canary introduces a critical vulnerability.

Action:

    Stop Rollout
        |
        ↓
    Abort Canary
        |
        ↓
    Stable
        |
        ↓
    Fix Vulnerability
        |
        ↓
    Security Scan
        |
        ↓
    New Version
        |
        ↓
    Canary Again

---

# Canary Deployment Advantages

## Reduced Blast Radius

Only a small percentage of users initially receive the new version.

## Early Problem Detection

Issues can be detected before full rollout.

## Gradual Release

Traffic can be increased progressively.

## Fast Rollback

Traffic can return to Stable.

## Real Production Validation

The new version can be tested with real traffic.

## Better Risk Management

Large deployments become smaller controlled steps.

---

# Canary Deployment Disadvantages

## More Complexity

Traffic routing and progressive rollout require additional configuration.

## Monitoring Requirements

Good metrics and logs are essential.

## Database Compatibility

Multiple application versions may run simultaneously.

## Operational Overhead

Teams must manage Stable and Canary versions.

## Longer Deployment Time

A gradual rollout takes longer than an immediate full deployment.

---

# Canary Cost Considerations

Canary usually requires running both versions simultaneously.

Example:

    Stable:
        10 Pods

    Canary:
        1-5 Pods

Additional resources may include:

    CPU
    Memory
    Load Balancing
    Monitoring
    Networking

However, Canary usually requires fewer additional resources than a full duplicate Blue-Green environment.

---

# Canary Best Practices

- Start with a small traffic percentage
- Define success criteria before deployment
- Define rollback criteria
- Monitor Canary continuously
- Compare Stable and Canary metrics
- Use immutable image tags
- Use health checks
- Use readiness probes
- Use smoke tests
- Use automated analysis where appropriate
- Keep Stable available
- Use backward-compatible database migrations
- Use GitOps for controlled changes
- Keep deployment history
- Automate progressive traffic where possible
- Test rollback procedures
- Monitor after full promotion

---

# Canary Security Best Practices

- Scan images before deployment
- Use immutable image tags
- Use least-privilege IAM
- Secure Kubernetes RBAC
- Protect secrets
- Use HTTPS
- Restrict network access
- Secure ECR
- Use OIDC for GitHub Actions AWS authentication where appropriate
- Audit deployment changes
- Validate security checks before promotion

---

# Canary Anti-Patterns

## Sending Too Much Traffic Initially

Bad:

    90% Stable
    10% Canary

without sufficient validation.

Better:

    99% Stable
     1% Canary

or another appropriately small starting percentage based on traffic volume and risk.

---

# Canary Anti-Pattern

## No Monitoring

Bad:

    Deploy Canary
        |
        ↓
    Increase Traffic Automatically
        |
        ↓
    100%

Better:

    Deploy
        |
        ↓
    Monitor
        |
        ↓
    Analyze
        |
        ↓
    Increase
        |
        ↓
    Monitor Again

---

# Canary Anti-Pattern

## No Rollback Plan

Bad:

    Canary Failure
        |
        ↓
    No Defined Action

Better:

    Canary Failure
        |
        ↓
    Abort
        |
        ↓
    Stable
        |
        ↓
    Recovery

---

# Canary Anti-Pattern

## Mutable Image Tags

Bad:

    payment:latest

Better:

    payment:1.4.7

or:

    payment:<commit-sha>

---

# Canary Anti-Pattern

## Incompatible Database Changes

Bad:

    Database Migration
        |
        ↓
    Canary
        |
        ↓
    Stable Cannot Work
        |
        ↓
    Rollback Failure

Better:

    Backward-Compatible Migration
        |
        ↓
    Canary
        |
        ↓
    Gradual Traffic
        |
        ↓
    Cleanup Later

---

# Canary Anti-Pattern

## Increasing Traffic Without Validation

Bad:

    5%
      ↓
    25%
      ↓
    50%
      ↓
    100%

without analysis.

Better:

    5%
      |
      ↓
    Analyze
      |
      ↓
    10%
      |
      ↓
    Analyze
      |
      ↓
    25%
      |
      ↓
    Analyze

---

# Canary Interview Questions

## Basic Questions

1. What is Canary Deployment?
2. Why is it called Canary deployment?
3. What is a Stable version?
4. What is a Canary version?
5. What is the difference between Canary and Blue-Green?
6. What is the difference between Canary and Rolling deployment?
7. Why do we start Canary with a small percentage?
8. What metrics should be monitored during Canary deployment?
9. What is Canary rollback?
10. What are the advantages of Canary deployment?

---

# Canary Interview Questions

## Intermediate Questions

11. How would you implement Canary deployment on Kubernetes?

12. How would you implement Canary deployment on EKS?

13. How would you route 5% traffic to Canary?

14. How would you implement Canary using Ingress?

15. How would you use ALB for Canary deployment?

16. How would you use Argo Rollouts?

17. How would you monitor Canary with Prometheus and Grafana?

18. How would you use logs to validate Canary?

19. How would you rollback a failed Canary deployment?

20. How would you handle database changes during Canary deployment?

---

# Canary Interview Questions

## Advanced Questions

21. Design a production Canary deployment architecture for EKS.

22. How would you implement progressive delivery with ArgoCD?

23. How would you automate Canary analysis?

24. How would you decide Canary success and failure thresholds?

25. How would you implement automatic rollback?

26. How would you route Canary traffic by percentage?

27. How would you route selected users to Canary?

28. How would you implement Canary using headers or cookies?

29. How would you handle database compatibility?

30. How would you design Canary for microservices?

31. How would you monitor Canary during a production rollout?

32. How would you reduce Canary deployment risk?

---

# Real-World DevOps Example

Suppose the application contains:

    User
    Product
    Cart
    Orders
    Payment
    Inventory
    Notification

Current:

    Stable
      |
      ├── User v1.4.6
      ├── Product v1.4.6
      ├── Cart v1.4.6
      ├── Orders v1.4.6
      ├── Payment v1.4.6
      ├── Inventory v1.4.6
      └── Notification v1.4.6

New:

    Canary
      |
      ├── User v1.4.7
      ├── Product v1.4.7
      ├── Cart v1.4.7
      ├── Orders v1.4.7
      ├── Payment v1.4.7
      ├── Inventory v1.4.7
      └── Notification v1.4.7

Traffic:

    95% → Stable
     5% → Canary

---

# Real-World Canary Progression

    v1.4.6 Stable
          |
          ↓
    Deploy v1.4.7 Canary
          |
          ↓
        5%
          |
          ↓
      Monitor
          |
          ↓
       Healthy
          |
          ↓
        10%
          |
          ↓
      Monitor
          |
          ↓
        25%
          |
          ↓
      Monitor
          |
          ↓
        50%
          |
          ↓
      Monitor
          |
          ↓
       100%
          |
          ↓
    v1.4.7 Stable

---

# Real-World Canary Rollback

    Stable
    v1.4.6
       |
       ↓
    90% Traffic

    Canary
    v1.4.7
       |
       ↓
    10% Traffic

Problem:

    Error Rate Increased

Action:

    Stop Rollout
        |
        ↓
    Abort Canary
        |
        ↓
    100% Stable
        |
        ↓
    v1.4.6
        |
        ↓
    Monitor
        |
        ↓
    Recovery

---

# Complete Canary DevOps Architecture

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
        +-- SonarQube
        +-- Trivy
        |
        ↓
    Amazon ECR
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
        +-------------------------+
        |                         |
        ↓                         ↓
     Stable                    Canary
     v1.4.6                    v1.4.7
        |                         |
        +------------+------------+
                     |
                     ↓
                   ALB
                     |
                     ↓
                 Route 53
                     |
                     ↓
                   Users

Initial:

    95% → Stable
     5% → Canary

Progressively:

    90% → Stable
    10% → Canary

    75% → Stable
    25% → Canary

    50% → Stable
    50% → Canary

Final:

     0% → Stable
    100% → Canary

---

# Complete Canary Monitoring Architecture

    Stable
       |
       +--------------------+
                            |
                            ↓
                         Metrics
                            |
                            ↓
                       Prometheus
                            |
                            ↓
                         Grafana


    Canary
       |
       +--------------------+
                            |
                            ↓
                         Metrics
                            |
                            ↓
                       Prometheus
                            |
                            ↓
                         Grafana


    Stable + Canary
          |
          ↓
        Logs
          |
          ↓
         ELK

Compare:

    Stable
       vs
    Canary

---

# Complete Canary Deployment Flow

    Developer
        |
        ↓
    Code
        |
        ↓
    Pull Request
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
    Docker Image
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
    Deploy Canary
        |
        ↓
       5%
        |
        ↓
    Health Checks
        |
        ↓
    Metrics Analysis
        |
        +------ Fail ------→ Abort
        |                      |
        |                      ↓
        |                   Stable
        |
        +------ Pass ------→ 10%
                               |
                               ↓
                            Analyze
                               |
                               ↓
                              25%
                               |
                               ↓
                            Analyze
                               |
                               ↓
                              50%
                               |
                               ↓
                            Analyze
                               |
                               ↓
                             100%
                               |
                               ↓
                        Canary Promoted
                               |
                               ↓
                         New Stable Version

---

# Canary Deployment Decision Flow

    New Version Ready
          |
          ↓
    Deploy Canary
          |
          ↓
        5%
          |
          ↓
      Health Check
          |
          ↓
      Metrics Analysis
          |
          +------ Fail ------→ Abort
          |
          +------ Pass
                    |
                    ↓
                  10%
                    |
                    ↓
                Analyze
                    |
                    +------ Fail → Abort
                    |
                    +------ Pass
                              |
                              ↓
                            25%
                              |
                              ↓
                           Analyze
                              |
                              +------ Fail → Abort
                              |
                              +------ Pass
                                        |
                                        ↓
                                      50%
                                        |
                                        ↓
                                     Analyze
                                        |
                                        +------ Fail → Abort
                                        |
                                        +------ Pass
                                                  |
                                                  ↓
                                                100%
                                                  |
                                                  ↓
                                             Promotion

---

# Final Canary Mental Model

Remember:

    Stable
    v1.4.6
       |
       ↓
    95% Traffic


    Canary
    v1.4.7
       |
       ↓
     5% Traffic

Then:

    Monitor
       |
       ↓
    Healthy?
       |
       +------ No ------→ Abort
       |                   |
       |                   ↓
       |                Stable
       |
       +------ Yes
              |
              ↓
            10%
              |
              ↓
            Monitor
              |
              ↓
            25%
              |
              ↓
            Monitor
              |
              ↓
            50%
              |
              ↓
            Monitor
              |
              ↓
            100%
              |
              ↓
        Canary Promoted
              |
              ↓
        New Stable Version

The key principle is:

    Small Traffic
        |
        ↓
    Observe
        |
        ↓
    Analyze
        |
        ↓
    Increase Traffic
        |
        ↓
    Observe Again
        |
        ↓
    Full Promotion

If anything goes wrong:

    Stop
      |
      ↓
    Abort
      |
      ↓
    Route Traffic to Stable
      |
      ↓
    Investigate
      |
      ↓
    Fix
      |
      ↓
    Release Again

---

# Final Concept

Canary Deployment is a progressive delivery strategy that reduces production risk by exposing a new version to a small percentage of traffic before gradually increasing exposure.

The complete DevOps model is:

    Code
      |
      ↓
    CI
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
    Immutable Image
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
    Canary
      |
      ↓
     5%
      |
      ↓
    Monitor
      |
      ↓
     10%
      |
      ↓
    Monitor
      |
      ↓
     25%
      |
      ↓
    Monitor
      |
      ↓
     50%
      |
      ↓
    Monitor
      |
      ↓
    100%
      |
      ↓
    Promotion

Rollback:

    Canary
      |
      ↓
    Failure
      |
      ↓
    Abort
      |
      ↓
    Stable
      |
      ↓
    Recovery

A strong Canary deployment strategy combines:

    Progressive Traffic
        +
    Health Checks
        +
    Automated Testing
        +
    Prometheus Metrics
        +
    Grafana Dashboards
        +
    ELK Logs
        +
    Immutable Images
        +
    GitOps
        +
    ArgoCD
        +
    EKS
        +
    Automated Analysis
        +
    Fast Rollback
        +
    Backward-Compatible Database Changes

The main goal is simple:

    Do not expose a new release to everyone immediately.

Instead:

    Release Small
        |
        ↓
    Observe
        |
        ↓
    Validate
        |
        ↓
    Increase Gradually
        |
        ↓
    Promote Safely

This reduces blast radius, provides real-production feedback, and allows teams to stop or rollback a release before a faulty version affects the entire user base.