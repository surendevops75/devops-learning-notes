# Common Mistakes

Common mistakes in DevOps can lead to:

    Deployment Failures
        +
    Security Incidents
        +
    Downtime
        +
    Performance Problems
        +
    Increased Costs
        +
    Difficult Troubleshooting

The goal is not only to know what works, but also to understand:

    What Can Go Wrong
        +
    Why It Goes Wrong
        +
    How To Detect It
        +
    How To Prevent It
        +
    How To Recover

---

# 1. Manual Deployments

A common mistake is relying heavily on manual deployments.

Bad:

    Developer
        |
        ↓
    SSH
        |
        ↓
    Production Server
        |
        ↓
    Manual Commands

Problems:

    Human Error
    +
    No Consistency
    +
    Difficult Rollback
    +
    Poor Auditability

Better:

    Git
        |
        ↓
    CI/CD
        |
        ↓
    Automated Deployment
        |
        ↓
    Production

---

# 2. No Version Control

Managing infrastructure or application configuration without Git creates problems.

Bad:

    Server
        |
        ↓
    Manual Configuration

Better:

    Git
        |
        ↓
    Versioned Configuration
        |
        ↓
    CI/CD
        |
        ↓
    Deployment

Benefits:

    History
    +
    Review
    +
    Rollback
    +
    Auditability

---

# 3. Committing Secrets to Git

Never commit:

    Passwords
    API Keys
    Access Keys
    Tokens
    Private Keys
    Database Credentials

Bad:

    Git
        |
        ↓
    Password
        |
        ↓
    Repository

If a secret is exposed:

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
    Prevent Recurrence

---

# 4. Hardcoding Credentials

Bad:

    application.properties

    username=admin
    password=password123

Better:

    Secret Store
        |
        ↓
    Application
        |
        ↓
    Runtime Credential

---

# 5. Using Administrator Permissions Everywhere

Bad:

    Developer
    CI/CD
    Application
        |
        ↓
    Administrator

Problems:

    Excessive Privilege
    +
    Larger Blast Radius
    +
    Higher Security Risk

Better:

    Identity
        |
        ↓
    Role
        |
        ↓
    Required Permissions Only

---

# 6. Running Everything as Root

Running applications and containers as root increases risk.

Bad:

    Application
        |
        ↓
    root
        |
        ↓
    Container

Better:

    Application
        |
        ↓
    Non-Root User
        |
        ↓
    Container

---

# 7. Using Privileged Containers

Avoid unnecessary:

    --privileged

Bad:

    Container
        |
        ↓
    Privileged Access
        |
        ↓
    Host

Better:

    Container
        |
        ↓
    Restricted Capabilities
        |
        ↓
    Host

---

# 8. Using Huge Docker Images

Large images cause:

    Slow Builds
    +
    Slow Image Pulls
    +
    Larger Storage
    +
    Larger Attack Surface

Bad:

    Application
        |
        ↓
    Huge Image

Better:

    Multi-Stage Build
        |
        ↓
    Minimal Runtime Image

---

# 9. Poor Docker Layer Ordering

Poor Dockerfile ordering can invalidate caching unnecessarily.

Bad:

    COPY Application
        |
        ↓
    Install Dependencies

Every application change may cause dependency installation again.

Better:

    Copy Dependency Files
        |
        ↓
    Install Dependencies
        |
        ↓
    Copy Application
        |
        ↓
    Build

---

# 10. Copying the Entire Repository

Bad:

    COPY .

This can include:

    .git
    Logs
    Temporary Files
    Local Dependencies
    Documentation
    Secrets

Better:

    .dockerignore
        |
        ↓
    Required Files Only

---

# 11. Storing Secrets Inside Docker Images

Bad:

    Dockerfile
        |
        ↓
    COPY secret.txt
        |
        ↓
    Image

Secrets may remain in image layers.

Better:

    Secret Store
        |
        ↓
    Runtime Injection
        |
        ↓
    Container

---

# 12. Using Latest Tags

Using:

    image:latest

can make deployments difficult to reproduce.

Problem:

    latest
        |
        ↓
    Version Changes
        |
        ↓
    Different Deployment

Better:

    image:v1.2.3

or use an immutable digest where appropriate.

---

# 13. Not Scanning Container Images

Bad:

    Docker Build
        |
        ↓
    Registry
        |
        ↓
    Production

Better:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        ↓
    Registry
        |
        ↓
    Production

---

# 14. Ignoring Critical Vulnerabilities

Bad:

    Critical Vulnerability
        |
        ↓
    Ignore
        |
        ↓
    Production

Better:

    Vulnerability
        |
        ↓
    Assess
        |
        ↓
    Remediate
        |
        ↓
    Rescan
        |
        ↓
    Deploy

---

# 15. No Dependency Scanning

Applications depend on:

    Libraries
    +
    Packages
    +
    Frameworks

A vulnerable dependency can introduce security risk.

Use appropriate:

    SCA
    +
    Dependency Updates
    +
    Vulnerability Management

---

# 16. Ignoring Dependency Updates

Old dependencies may contain:

    Security Vulnerabilities
    +
    Bugs
    +
    Performance Issues

Use a controlled process:

    Dependency Update
        |
        ↓
    Test
        |
        ↓
    Scan
        |
        ↓
    Validate
        |
        ↓
    Deploy

---

# 17. No Secret Rotation

Bad:

    Secret
        |
        ↓
    Created Once
        |
        ↓
    Never Rotated

Better:

    Secret
        |
        ↓
    Rotation
        |
        ↓
    Validation

If compromised:

    Revoke
        |
        ↓
    Rotate Immediately

---

# 18. Printing Secrets in Logs

Bad:

    echo $PASSWORD

This can expose credentials in CI/CD logs.

Better:

    Never Print Sensitive Values

Use secret masking where supported.

---

# 19. Logging Sensitive Information

Never unnecessarily log:

    Passwords
    Tokens
    API Keys
    Private Keys
    Sensitive Personal Data

Bad:

    Authorization: Bearer <token>

Better:

    Authorization: [REDACTED]

---

# 20. No Centralized Logging

Bad:

    Server 1 → Logs
    Server 2 → Logs
    Server 3 → Logs

Troubleshooting becomes difficult.

Better:

    Applications
        |
        ↓
    Centralized Logging
        |
        ↓
    ELK
        |
        ↓
    Search / Investigation

---

# 21. No Monitoring

Deploying without monitoring is dangerous.

Bad:

    Deployment
        |
        ↓
    Production
        |
        X
    No Visibility

Better:

    Deployment
        |
        ↓
    Prometheus
        +
    Grafana
        +
    ELK
        |
        ↓
    Monitoring

---

# 22. Monitoring Only CPU and Memory

CPU and memory alone do not explain application health.

Also monitor:

    Request Rate
    Error Rate
    Latency
    P95
    P99
    Pod Restarts
    Queue Depth
    Database Connections

---

# 23. No Alerting

Monitoring without actionable alerts can delay incident detection.

Bad:

    Metrics
        |
        ↓
    Dashboard
        |
        ↓
    Nobody Notices

Better:

    Metrics
        |
        ↓
    Alert
        |
        ↓
    Investigation
        |
        ↓
    Response

---

# 24. Alerting on Everything

Too many alerts create:

    Alert Fatigue
        |
        ↓
    Ignored Alerts
        |
        ↓
    Missed Incidents

Alerts should be:

    Actionable
    +
    Meaningful
    +
    Prioritized

---

# 25. No Health Checks

Applications without health checks can receive traffic while unhealthy.

Bad:

    Pod
        |
        ↓
    Unhealthy
        |
        ↓
    Traffic

Better:

    Pod
        |
        ↓
    Readiness Probe
        |
        ↓
    Healthy?
        |
        +------ No → No Traffic
        |
        +------ Yes → Traffic

---

# 26. Incorrect Liveness Probes

An aggressive liveness probe can restart healthy but slow-starting applications.

Bad:

    Slow Startup
        |
        ↓
    Liveness Probe
        |
        ↓
    Failure
        |
        ↓
    Restart
        |
        ↓
    Startup Again
        |
        ↓
    Failure

Better:

    Startup Probe
        |
        ↓
    Application Ready
        |
        ↓
    Liveness Probe

---

# 27. Incorrect Readiness Probes

A readiness probe should represent whether the application can safely receive traffic.

Bad configuration can cause:

    Healthy Pod
        |
        ↓
    Readiness Failure
        |
        ↓
    No Traffic

Review:

    Endpoint
    +
    Timeout
    +
    Initial Delay
    +
    Period
    +
    Failure Threshold

---

# 28. No Startup Probe for Slow Applications

Slow-starting applications can be incorrectly restarted.

Use startup probes when appropriate.

Example:

    Container Start
        |
        ↓
    Startup Probe
        |
        ↓
    Application Ready
        |
        ↓
    Readiness / Liveness

---

# 29. Incorrect Resource Requests

Requests that are too high can cause:

    Poor Scheduling
    +
    Wasted Capacity
    +
    Lower Cluster Utilization

Requests that are too low can cause:

    Resource Contention
    +
    Performance Problems

---

# 30. Incorrect Resource Limits

Too low:

    Application
        |
        ↓
    Memory Limit
        |
        ↓
    OOMKilled

Too high:

    Excessive Reserved Capacity
        |
        ↓
    Poor Utilization

Use actual workload measurements.

---

# 31. Ignoring CPU Throttling

A container can experience CPU throttling when its CPU limit is reached.

Possible result:

    CPU Demand
        |
        ↓
    CPU Limit
        |
        ↓
    Throttling
        |
        ↓
    Increased Latency

Investigate workload behavior before changing limits.

---

# 32. Ignoring OOMKilled

If pods repeatedly show:

    OOMKilled

do not simply increase memory indefinitely.

Investigate:

    Memory Usage
    +
    Application Behavior
    +
    Memory Leaks
    +
    Traffic
    +
    Resource Limits

---

# 33. Scaling Without Finding the Bottleneck

Bad:

    Application Slow
        |
        ↓
    Add Pods
        |
        ↓
    More Database Connections
        |
        ↓
    Database Overload

Better:

    Performance Issue
        |
        ↓
    Identify Bottleneck
        |
        ↓
    Optimize Correct Component

---

# 34. Misusing HPA

HPA is not a universal solution.

Example:

    Slow Database
        |
        ↓
    More Pods
        |
        ↓
    More Database Requests
        |
        ↓
    Worse Performance

Always identify the bottleneck first.

---

# 35. No Cluster Autoscaling

If HPA creates pods but the cluster has no capacity:

    HPA
        |
        ↓
    More Pods
        |
        ↓
    Pods Pending
        |
        ↓
    Insufficient Nodes

Where appropriate:

    HPA
        |
        ↓
    Cluster Autoscaler
        |
        ↓
    New Nodes
        |
        ↓
    Pods Scheduled

---

# 36. Ignoring Node Pressure

Node pressure can cause:

    Pod Evictions
    +
    Scheduling Problems
    +
    Performance Degradation

Monitor:

    CPU
    Memory
    Disk
    PID

---

# 37. Running Too Many Pods on One Node

Excessive pod density can create:

    CPU Contention
    +
    Memory Contention
    +
    Network Contention

Use appropriate:

    Resource Requests
    +
    Limits
    +
    Scheduling Policies

---

# 38. Putting All Replicas on One Node

Bad:

    Node 1
        |
        +-- Pod 1
        +-- Pod 2
        +-- Pod 3

Node failure:

    All Pods Lost

Better:

    Node 1 → Pod 1
    Node 2 → Pod 2
    Node 3 → Pod 3

Use appropriate anti-affinity or topology spreading.

---

# 39. No Pod Disruption Budget

During voluntary disruptions, too many replicas can be unavailable at once.

Consider:

    Pod Disruption Budget
        |
        ↓
    Maintain Minimum Availability

Use it appropriately for critical workloads.

---

# 40. No Graceful Shutdown

Bad:

    Deployment
        |
        ↓
    Pod Terminated Immediately
        |
        ↓
    Active Requests Dropped

Better:

    Termination
        |
        ↓
    Stop New Requests
        |
        ↓
    Complete Existing Requests
        |
        ↓
    Close Connections
        |
        ↓
    Exit

---

# 41. No Rollback Strategy

Bad:

    Deployment Failure
        |
        ↓
    Production
        |
        ↓
    Manual Investigation

Better:

    Deployment
        |
        ↓
    Validation
        |
        +------ Fail
        |
        ↓
    Rollback
        |
        ↓
    Previous Version

---

# 42. Deploying Without Validation

Bad:

    Deploy
        |
        ↓
    Assume Success

Better:

    Deploy
        |
        ↓
    Health Checks
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
    Validate

---

# 43. No Post-Deployment Validation

A successful deployment command does not necessarily mean the application is healthy.

Check:

    Pods
    +
    Services
    +
    Endpoints
    +
    Health Checks
    +
    Logs
    +
    Metrics
    +
    Application Response

---

# 44. Treating Deployment Success as Application Success

Bad:

    kubectl apply
        |
        ↓
    Command Successful
        |
        ↓
    Production Healthy

Better:

    Deployment
        |
        ↓
    Application Health
        |
        ↓
    Traffic
        |
        ↓
    Metrics
        |
        ↓
    Logs
        |
        ↓
    Validation

---

# 45. No Deployment Strategy

Using uncontrolled replacement can cause downtime.

Better options:

    Rolling
    +
    Blue-Green
    +
    Canary

Choose based on:

    Risk
    +
    Architecture
    +
    Business Requirements

---

# 46. Incorrect Rolling Deployment

If replicas are too low and update settings are poorly configured:

    Old Pod
        |
        ↓
    Terminated
        |
        ↓
    New Pod
        |
        ↓
    Startup
        |
        ↓
    No Available Capacity

Configure deployment behavior carefully.

---

# 47. No Canary Validation

Canary deployments should be monitored.

Bad:

    New Version
        |
        ↓
    100% Traffic

Better:

    New Version
        |
        ↓
    5% Traffic
        |
        ↓
    Monitor
        |
        ↓
    Increase Gradually

---

# 48. Ignoring P95 and P99

Average latency can hide slow requests.

Example:

    Average = 150 ms
    P95 = 400 ms
    P99 = 2 seconds

The application may still have serious tail latency.

---

# 49. No Performance Baseline

Without a baseline, performance regressions are harder to detect.

Better:

    Before Deployment
        |
        ↓
    Baseline
        |
        ↓
    Deployment
        |
        ↓
    Compare
        |
        ↓
    Detect Regression

---

# 50. Optimizing Without Measuring

Bad:

    Problem
        |
        ↓
    Guess
        |
        ↓
    Change

Better:

    Problem
        |
        ↓
    Metrics
        |
        ↓
    Bottleneck
        |
        ↓
    Optimization
        |
        ↓
    Validation

---

# 51. Unlimited Retries

Bad:

    Failure
        |
        ↓
    Retry
        |
        ↓
    Retry
        |
        ↓
    Retry
        |
        ↓
    Retry

This can amplify load.

Better:

    Failure
        |
        ↓
    Limited Retry
        |
        ↓
    Exponential Backoff
        |
        ↓
    Jitter
        |
        ↓
    Stop

---

# 52. No Timeout

Without timeouts:

    Service A
        |
        ↓
    Service B
        |
        ↓
    Service B Slow
        |
        ↓
    Service A Waits
        |
        ↓
    Resource Exhaustion

Use appropriate timeouts.

---

# 53. Retry Storm

If many clients retry simultaneously:

    Service Failure
        |
        +------ Client 1
        +------ Client 2
        +------ Client 3
        +------ Client 4
        |
        ↓
    Retry Storm
        |
        ↓
    More Load
        |
        ↓
    Worse Failure

Use:

    Backoff
    +
    Jitter
    +
    Retry Limits
    +
    Circuit Breakers

---

# 54. Excessive Synchronous Microservice Calls

Bad:

    Service A
        |
        ↓
    B
        |
        ↓
    C
        |
        ↓
    D
        |
        ↓
    E

Every synchronous call adds:

    Latency
    +
    Failure Dependency

Use asynchronous processing where appropriate.

---

# 55. Ignoring Database Connection Limits

Adding more pods can create too many database connections.

Example:

    10 Pods
        |
        ↓
    10 Connection Pools
        |
        ↓
    Database
        |
        ↓
    Connection Exhaustion

Always consider database capacity when scaling.

---

# 56. No Database Backup Strategy

Bad:

    Database
        |
        X
    No Backup

Better:

    Database
        |
        ↓
    Backup
        |
        ↓
    Secure Storage
        |
        ↓
    Restore Testing

---

# 57. Never Testing Backups

A backup is not enough.

Test:

    Backup
        |
        ↓
    Restore
        |
        ↓
    Validate
        |
        ↓
    Document Result

---

# 58. Public Database Access

Bad:

    Internet
        |
        ↓
    Database

Better:

    Application
        |
        ↓
    Private Network
        |
        ↓
    Database

---

# 59. Open Security Groups

Bad:

    0.0.0.0/0
        |
        ↓
    All Ports

Better:

    Required Source
        |
        ↓
    Required Port
        |
        ↓
    Required Service

---

# 60. No Network Segmentation

Bad:

    Internet
        |
        ↓
    Application
        |
        ↓
    Database

with unnecessary public exposure.

Better:

    Internet
        |
        ↓
    Public ALB
        |
        ↓
    Private Application
        |
        ↓
    Private Database

---

# 61. No Network Policies in Kubernetes

Without appropriate restrictions:

    Pod A
        |
        +------ Pod B
        +------ Pod C
        +------ Pod D
        +------ Database

Better:

    Pod A
        |
        ↓
    Allowed Service Only

Use NetworkPolicies where appropriate.

---

# 62. Excessive Kubernetes Permissions

Bad:

    Application
        |
        ↓
    cluster-admin

Better:

    Application
        |
        ↓
    Service Account
        |
        ↓
    Required Permissions

---

# 63. Giving Developers Direct Production Access

Bad:

    Developer
        |
        ↓
    SSH
        |
        ↓
    Production

Better:

    Developer
        |
        ↓
    Git
        |
        ↓
    Review
        |
        ↓
    CI/CD / GitOps
        |
        ↓
    Production

---

# 64. Manual Changes to Kubernetes

Bad:

    Git
        |
        X
    Kubernetes

    Developer
        |
        ↓
    kubectl edit
        |
        ↓
    Cluster

This can create configuration drift.

Better:

    Git
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

---

# 65. Configuration Drift

Configuration drift occurs when the actual environment differs from the desired configuration.

Example:

    Git
        |
        ↓
    Desired State

    Kubernetes
        |
        ↓
    Actual State

If different:

    Drift

Use GitOps and reconciliation where appropriate.

---

# 66. No Infrastructure as Code

Manual infrastructure creates:

    Inconsistency
    +
    Human Error
    +
    Poor Reproducibility

Better:

    Terraform
        |
        ↓
    Versioned Infrastructure
        |
        ↓
    Automated Deployment

---

# 67. Storing Terraform State Locally

Local state creates collaboration and recovery problems.

Better:

    Terraform
        |
        ↓
    Remote Backend
        |
        ↓
    Secure State

Protect state because it may contain sensitive infrastructure information.

---

# 68. No Terraform State Protection

Terraform state should have:

    Access Control
    +
    Encryption
    +
    Versioning
    +
    Appropriate Locking / Concurrency Protection

---

# 69. Running Terraform Without Review

Bad:

    Terraform Plan
        |
        ↓
    terraform apply

Better:

    Terraform Format
        |
        ↓
    Validation
        |
        ↓
    Plan
        |
        ↓
    Review
        |
        ↓
    Apply

---

# 70. Applying Terraform From a Developer Laptop

Manual production Terraform execution can create:

    Credential Risk
    +
    Environment Differences
    +
    Poor Auditability
    +
    Human Error

Prefer controlled execution through an appropriate CI/CD process for production.

---

# 71. Ignoring Terraform Plan

The plan shows intended changes.

Bad:

    Code Change
        |
        ↓
    Apply
        |
        ↓
    Unexpected Change

Better:

    Code Change
        |
        ↓
    terraform plan
        |
        ↓
    Review
        |
        ↓
    Apply

---

# 72. No Terraform Validation

Use appropriate checks such as:

    terraform fmt
    terraform validate
    terraform plan

Add security and policy scanning where required.

---

# 73. Mixing Environments

Bad:

    Development
        |
        ↓
    Same Resources
        |
        ↓
    Production

Better:

    Development
        |
        ↓
    Separate Environment

    QA
        |
        ↓
    Separate Environment

    Production
        |
        ↓
    Separate Environment

---

# 74. Using Production Data in Development

Production data may contain sensitive information.

Avoid copying production data into development without appropriate:

    Sanitization
    +
    Access Control
    +
    Data Protection

---

# 75. No Environment Isolation

Environment isolation should consider:

    Accounts
    +
    Networks
    +
    Credentials
    +
    Resources
    +
    Access

---

# 76. No Approval for Production

Bad:

    Developer
        |
        ↓
    Production

Better:

    Developer
        |
        ↓
    CI/CD
        |
        ↓
    Approval
        |
        ↓
    Production

---

# 77. No Separation of Duties

A single person should not necessarily control:

    Development
        +
    Approval
        +
    Production Deployment
        +
    Audit

Use appropriate organizational controls.

---

# 78. No Branch Protection

Without branch protection:

    Developer
        |
        ↓
    Direct Push
        |
        ↓
    Main

Better:

    Developer
        |
        ↓
    Feature Branch
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Main

---

# 79. Merging Without Code Review

Code review can catch:

    Bugs
    +
    Security Problems
    +
    Configuration Errors
    +
    Poor Design

Use pull requests and appropriate review requirements.

---

# 80. No CI Checks on Pull Requests

Bad:

    Pull Request
        |
        ↓
    Merge

Better:

    Pull Request
        |
        ↓
    Build
        +
    Tests
        +
    Security
        +
    Quality
        |
        ↓
    Review
        |
        ↓
    Merge

---

# 81. Running All Tests Sequentially

Bad:

    Unit Tests
        |
        ↓
    Integration Tests
        |
        ↓
    Security
        |
        ↓
    Lint

Better where dependencies allow:

    Build
        |
        +------ Unit Tests
        +------ Security
        +------ Lint
        |
        ↓
    Integration Tests

---

# 82. No Dependency Caching

Repeated downloads make CI slow.

Bad:

    Every Build
        |
        ↓
    Download Dependencies
        |
        ↓
    Build

Better:

    Cache
        |
        ↓
    Dependencies
        |
        ↓
    Build

---

# 83. No Docker Build Cache

Repeatedly rebuilding unchanged layers increases CI time.

Use appropriate:

    Layer Caching
    +
    BuildKit
    +
    Registry Cache

---

# 84. No Artifact Management

Build artifacts should be traceable.

Bad:

    Build
        |
        ↓
    Unknown Artifact
        |
        ↓
    Production

Better:

    Git Commit
        |
        ↓
    Build
        |
        ↓
    Artifact
        |
        ↓
    Version
        |
        ↓
    Registry
        |
        ↓
    Deployment

---

# 85. Rebuilding the Same Artifact for Each Environment

Bad:

    Build → QA
    Build → UAT
    Build → Production

This can produce different artifacts.

Better:

    Build Once
        |
        ↓
    Artifact
        |
        +------ QA
        |
        +------ UAT
        |
        +------ Production

---

# 86. Using Mutable Artifacts

Avoid relying on:

    latest

for production deployment.

Prefer:

    Versioned Artifact
        |
        ↓
    Immutable Reference
        |
        ↓
    Deployment

---

# 87. No Artifact Retention Policy

Unmanaged artifacts can consume storage.

Define:

    Retention
    +
    Cleanup
    +
    Versioning

while preserving artifacts required for audit and rollback.

---

# 88. No Release Versioning

Without versions:

    Production
        |
        ↓
    What Version Is Running?

Better:

    Application
        |
        ↓
    v1.2.3
        |
        ↓
    Deployment

---

# 89. Poor Git Commit Messages

Bad:

    update
    changes
    fix

Better:

    fix: handle payment timeout
    feat: add inventory validation
    chore: update dependencies

Meaningful commit history helps troubleshooting.

---

# 90. Large Unrelated Commits

Bad:

    One Commit
        |
        +-- Application
        +-- Terraform
        +-- Kubernetes
        +-- Documentation
        +-- Unrelated Changes

Better:

    Focused Commits
        |
        +-- Application Change
        +-- Infrastructure Change
        +-- Documentation Change

---

# 91. Long-Lived Feature Branches

Long-lived branches can create:

    Merge Conflicts
    +
    Drift
    +
    Difficult Integration

Prefer shorter-lived branches where the team's workflow supports it.

---

# 92. No Branching Strategy

Teams should define:

    Main Branch
    +
    Feature Branches
    +
    Pull Requests
    +
    Release Strategy

---

# 93. No Tagging Strategy

Releases should be identifiable.

Example:

    v1.0.0
    v1.1.0
    v1.2.0

Tags help connect:

    Code
        |
        ↓
    Build
        |
        ↓
    Artifact
        |
        ↓
    Deployment

---

# 94. No Change Tracking

Production changes should be traceable.

Example:

    Change Request
        |
        ↓
    Git Commit
        |
        ↓
    Pipeline
        |
        ↓
    Deployment
        |
        ↓
    Audit

---

# 95. No Audit Trail

Without an audit trail, it becomes difficult to determine:

    Who
        +
    What
        +
    When
        +
    Why

made a production change.

---

# 96. Ignoring Failed Deployments

Do not repeatedly retry a failed deployment without understanding the cause.

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
    Retry

---

# 97. No Rollback Testing

A rollback strategy that has never been tested may fail during an incident.

Test:

    Deployment
        |
        ↓
    Rollback
        |
        ↓
    Validation

---

# 98. Rollback Instead of Root Cause Analysis

Rollback restores service, but it does not explain the failure.

Use:

    Rollback
        |
        ↓
    Restore Stability
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Fix
        |
        ↓
    Prevent Recurrence

---

# 99. No Incident Runbooks

Without runbooks:

    Incident
        |
        ↓
    Engineer Starts From Zero

Better:

    Incident
        |
        ↓
    Runbook
        |
        ↓
    Investigation
        |
        ↓
    Resolution

---

# 100. No Post-Incident Review

After an incident:

    Resolve
        |
        ↓
    Review
        |
        ↓
    Root Cause
        |
        ↓
    Corrective Actions
        |
        ↓
    Prevent Recurrence

---

# 101. Blaming Individuals

Bad incident culture:

    Incident
        |
        ↓
    Find Person To Blame

Better:

    Incident
        |
        ↓
    Understand System
        |
        ↓
    Root Cause
        |
        ↓
    Improve Process

---

# 102. No Disaster Recovery Testing

Having a DR document is not enough.

Test:

    Backup
        |
        ↓
    Restore
        |
        ↓
    Infrastructure
        |
        ↓
    Application
        |
        ↓
    Validation

---

# 103. No Recovery Objectives

Define appropriate:

    RTO
    +
    RPO

RTO:

    How quickly should the service be restored?

RPO:

    How much data loss is acceptable?

---

# 104. No Capacity Planning

Unexpected traffic can cause:

    Resource Exhaustion
    +
    High Latency
    +
    Errors

Plan for:

    Normal Load
    +
    Peak Load
    +
    Growth
    +
    Failure Scenarios

---

# 105. Over-Provisioning Everything

Bad:

    Every Resource
        |
        ↓
    Maximum Capacity

Problems:

    High Cost
    +
    Poor Utilization

Better:

    Measure
        |
        ↓
    Right-Size
        |
        ↓
    Autoscale

---

# 106. Under-Provisioning Everything

Bad:

    Minimal Resources
        |
        ↓
    Production Traffic
        |
        ↓
    Saturation
        |
        ↓
    Errors

Balance:

    Performance
    +
    Reliability
    +
    Cost

---

# 107. No Cost Monitoring

Cloud resources can grow unexpectedly.

Monitor:

    Compute
    +
    Storage
    +
    Network
    +
    Database
    +
    Load Balancers

---

# 108. Unused Resources

Common examples:

    Unused EC2 Instances
    Unused EBS Volumes
    Unused Load Balancers
    Old Snapshots
    Old Container Images
    Unused IP Addresses

Implement appropriate cleanup policies.

---

# 109. Ignoring Storage Growth

Storage can grow continuously.

Example:

    Logs
        |
        ↓
    Storage
        |
        ↓
    Growth
        |
        ↓
    Higher Cost
        |
        ↓
    Capacity Risk

Use:

    Retention
    +
    Rotation
    +
    Cleanup

---

# 110. Unlimited Log Retention

Keeping every log forever can create:

    High Storage Cost
    +
    Difficult Search
    +
    Compliance Complexity

Define appropriate retention policies.

---

# 111. Logging Everything at Debug Level

Excessive logs can create:

    High Storage
    +
    High Network Usage
    +
    Difficult Troubleshooting

Use appropriate log levels.

---

# 112. Poor Log Structure

Bad:

    Something happened

Better structured logs can contain:

    Timestamp
    +
    Service
    +
    Request ID
    +
    Level
    +
    Message
    +
    Relevant Context

---

# 113. No Correlation ID

In microservices, tracing a request without correlation information can be difficult.

Example:

    Request
        |
        ↓
    Service A
        |
        ↓
    Service B
        |
        ↓
    Service C

A correlation ID helps connect related log entries.

---

# 114. Ignoring Application Logs

Metrics may show:

    High Latency

Logs can help explain:

    Why

Use both:

    Metrics
        +
    Logs

---

# 115. Using Logs as the Only Monitoring Tool

Logs are valuable, but they do not replace metrics.

Use:

    Metrics
        +
    Logs
        +
    Dashboards
        +
    Alerts

---

# 116. No Resource Monitoring

Monitor:

    CPU
    +
    Memory
    +
    Disk
    +
    Network

But also monitor application-level indicators.

---

# 117. No Production Access Controls

Production access should be:

    Restricted
    +
    Audited
    +
    Role-Based
    +
    Time-Bounded Where Appropriate

---

# 118. Shared Accounts

Avoid shared administrative accounts.

Bad:

    admin
        |
        ↓
    Everyone Uses Same Account

Problems:

    No Individual Accountability
    +
    Credential Sharing
    +
    Difficult Auditing

Better:

    Individual Identity
        |
        ↓
    Role
        |
        ↓
    Permissions

---

# 119. No MFA

For sensitive systems, use MFA where appropriate.

Example:

    User
        |
        ↓
    Password
        +
    MFA
        |
        ↓
    Access

---

# 120. No Credential Expiration / Rotation

Long-lived credentials increase risk.

Use:

    Temporary Credentials
    +
    Rotation
    +
    Least Privilege

where appropriate.

---

# 121. Trusting Internal Networks

Do not assume:

    Internal = Trusted

Use:

    Authentication
    +
    Authorization
    +
    Encryption
    +
    Network Controls

---

# 122. No Encryption

Sensitive data should be protected:

    At Rest
        +
    In Transit

---

# 123. Using HTTP for Sensitive Traffic

Bad:

    Client
        |
        ↓
    HTTP
        |
        ↓
    Sensitive Data

Better:

    Client
        |
        ↓
    HTTPS / TLS
        |
        ↓
    Application

---

# 124. No Security Scanning in CI

Bad:

    Code
        |
        ↓
    Build
        |
        ↓
    Deploy

Better:

    Code
        |
        ↓
    SAST
        |
        ↓
    SCA
        |
        ↓
    Container Scan
        |
        ↓
    Security Gate
        |
        ↓
    Deploy

---

# 125. Treating Security as a Final Stage

Bad:

    Development
        |
        ↓
    Build
        |
        ↓
    Deploy
        |
        ↓
    Security Review

Better:

    Development
        |
        ↓
    Security
        |
        ↓
    Build
        |
        ↓
    Security
        |
        ↓
    Deploy
        |
        ↓
    Runtime Security

Security should be continuous.

---

# 126. No Quality Gates

Code should not automatically proceed when quality requirements fail.

Example:

    Build
        |
        ↓
    SonarQube
        |
        ↓
    Quality Gate
        |
        +------ Fail → Stop
        |
        +------ Pass → Continue

---

# 127. Ignoring SonarQube Findings

SonarQube can identify:

    Bugs
    +
    Vulnerabilities
    +
    Code Smells

Use findings as part of the quality process.

---

# 128. Ignoring Trivy Findings

Trivy can identify vulnerabilities in:

    Container Images
    +
    Filesystems
    +
    Configuration

Do not automatically ignore critical findings.

---

# 129. Ignoring Veracode Findings

Security testing findings should be:

    Reviewed
        |
        ↓
    Prioritized
        |
        ↓
    Remediated
        |
        ↓
    Validated

---

# 130. No Security Gates

Bad:

    Security Scan
        |
        ↓
    Finding
        |
        ↓
    Deploy Anyway

Better:

    Security Scan
        |
        ↓
    Security Gate
        |
        +------ Fail → Stop
        |
        +------ Pass → Continue

---

# 131. Using Third-Party Actions Without Review

Do not blindly trust external CI/CD actions.

Review:

    Source
    +
    Maintainer
    +
    Version
    +
    Permissions
    +
    Required Secrets

---

# 132. Excessive GitHub Actions Permissions

Bad:

    permissions:
      write-all

Better:

    Grant Only Required Permissions

Example:

    permissions:
      contents: read

---

# 133. Not Protecting Production Environment

Production deployments should have appropriate:

    Access Control
    +
    Approval
    +
    Environment Protection
    +
    Audit

---

# 134. Insecure Self-Hosted Runners

Self-hosted runners can contain:

    Credentials
    +
    Workspace Files
    +
    Network Access

Protect with:

    Isolation
    +
    Runner Groups
    +
    Access Controls
    +
    Ephemeral Runners Where Appropriate

---

# 135. Reusing Dirty Runners

Persistent runners may retain:

    Files
    +
    Credentials
    +
    Build Artifacts

Clean or replace runners appropriately.

---

# 136. No Runner Isolation

Do not expose sensitive production networks to untrusted workflows unnecessarily.

Use:

    Network Segmentation
    +
    Runner Groups
    +
    Controlled Workflows

---

# 137. No Branch Protection

Protect important branches with:

    Pull Requests
    +
    Reviews
    +
    Required Checks
    +
    Restricted Pushes

---

# 138. No CODEOWNERS / Ownership Controls

Clearly define who reviews sensitive areas where appropriate.

Examples:

    Terraform
    +
    Kubernetes
    +
    Security
    +
    Production Configuration

---

# 139. No Separation Between CI and CD

A clean architecture can separate:

    CI
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Scan
        |
        ↓
    Artifact

from:

    CD
        |
        ↓
    Deploy
        |
        ↓
    Validate
        |
        ↓
    Monitor

---

# 140. Directly Deploying Unverified Artifacts

Bad:

    Build
        |
        ↓
    Deploy

Better:

    Build
        |
        ↓
    Test
        |
        ↓
    Scan
        |
        ↓
    Artifact
        |
        ↓
    Deploy

---

# 141. No Artifact Traceability

Always know:

    Which Commit?
        |
        ↓
    Which Build?
        |
        ↓
    Which Artifact?
        |
        ↓
    Which Environment?
        |
        ↓
    Which Deployment?

---

# 142. No GitOps Reconciliation

If using GitOps:

    Git
        |
        ↓
    Desired State

should be reconciled with:

    Kubernetes
        |
        ↓
    Actual State

Tools such as ArgoCD can detect and reconcile drift according to configured policies.

---

# 143. Manual GitOps Changes

Bad:

    Git
        |
        ↓
    ArgoCD

but engineers frequently modify:

    kubectl edit
        |
        ↓
    Cluster

This can create drift.

---

# 144. Ignoring ArgoCD Drift

If desired and actual state differ:

    Git
        |
        ↓
    Desired State

    Kubernetes
        |
        ↓
    Actual State

Investigate the difference and determine whether it is:

    Expected
        +
    Temporary
        +
    Unauthorized
        +
    Configuration Error

---

# 145. No GitOps Rollback Strategy

GitOps should make rollback reproducible.

Example:

    Current Version
        |
        ↓
    Git Commit
        |
        ↓
    Previous Commit
        |
        ↓
    ArgoCD
        |
        ↓
    Previous Deployment

---

# 146. No Environment-Specific Configuration Strategy

Avoid manually editing files for each environment.

Use controlled approaches such as:

    Environment Configuration
        |
        ↓
    Git
        |
        ↓
    CI/CD / GitOps
        |
        ↓
    Environment

---

# 147. Hardcoding Environment Values

Bad:

    Production URL
        |
        ↓
    Hardcoded Everywhere

Better:

    Environment Configuration
        |
        ↓
    Application

---

# 148. No Configuration Validation

Invalid configuration can break deployments.

Validate:

    YAML
    +
    Kubernetes Manifests
    +
    Terraform
    +
    Application Configuration

before deployment.

---

# 149. No Dry Run / Plan

Where supported:

    Validate
        |
        ↓
    Dry Run / Plan
        |
        ↓
    Review
        |
        ↓
    Apply

This reduces unexpected changes.

---

# 150. Changing Too Many Things at Once

Bad:

    One Deployment
        |
        +-- Application Change
        +-- Database Change
        +-- Infrastructure Change
        +-- Network Change
        +-- Security Change

If it fails, root cause becomes difficult to identify.

Better:

    Controlled Changes
        |
        ↓
    Small Scope
        |
        ↓
    Validate
        |
        ↓
    Continue

---

# 151. Large Production Changes

Large changes increase risk.

Prefer:

    Small Change
        |
        ↓
    Validate
        |
        ↓
    Next Change

where practical.

---

# 152. No Change Failure Strategy

Before deployment define:

    Success Criteria
        +
    Failure Criteria
        +
    Rollback Strategy
        +
    Validation

---

# 153. Ignoring Deployment Windows

For sensitive production changes, coordinate:

    Change Request
    +
    Approval
    +
    Deployment Window
    +
    Monitoring
    +
    Rollback Plan

---

# 154. No Communication During Incidents

During incidents:

    Technical Work
        +
    Stakeholder Communication

Both matter.

---

# 155. Poor Incident Communication

Avoid:

    No Updates
        |
        ↓
    Stakeholders Guess

Better:

    Incident
        |
        ↓
    Status
        |
        ↓
    Impact
        |
        ↓
    Action
        |
        ↓
    Next Update

---

# 156. No Incident Severity

Not every incident has the same impact.

Define appropriate severity levels based on:

    Customer Impact
    +
    Availability
    +
    Business Impact
    +
    Security Impact

---

# 157. Ignoring Business Impact

A technical issue should be evaluated in terms of:

    Users
    +
    Revenue
    +
    Business Operations
    +
    Compliance
    +
    Reputation

---

# 158. No Documentation

Undocumented systems create:

    Knowledge Gaps
    +
    Slow Troubleshooting
    +
    Onboarding Problems

Document:

    Architecture
    +
    Deployment
    +
    Troubleshooting
    +
    Runbooks

---

# 159. Documentation That Is Never Updated

Old documentation can be worse than no documentation.

Review documentation after:

    Architecture Changes
    +
    Tool Changes
    +
    Process Changes
    +
    Incident Lessons

---

# 160. No Standardization

Different teams using completely different patterns can increase operational complexity.

Standardize where appropriate:

    CI/CD
    +
    Terraform
    +
    Kubernetes
    +
    Monitoring
    +
    Security

---

# 161. Over-Engineering

Adding unnecessary:

    Tools
    +
    Services
    +
    Abstractions
    +
    Pipelines

can increase complexity.

Choose solutions based on:

    Requirements
        +
    Risk
        +
    Scale
        +
    Team Capability

---

# 162. Tool-Driven Architecture

Bad:

    Tool
        |
        ↓
    Find Problem For Tool

Better:

    Requirement
        |
        ↓
    Problem
        |
        ↓
    Appropriate Tool

---

# 163. Using Too Many Tools

Example:

    CI Tool 1
    +
    CI Tool 2
    +
    CI Tool 3
    +
    CD Tool 1
    +
    CD Tool 2

can create unnecessary complexity.

Use tools that solve actual requirements.

---

# 164. No Ownership

Every production system should have clear ownership.

Define:

    Application Owner
    +
    Infrastructure Owner
    +
    Security Owner
    +
    Support Owner

---

# 165. No On-Call Process

Production systems need an appropriate incident response process.

Example:

    Alert
        |
        ↓
    On-Call Engineer
        |
        ↓
    Investigation
        |
        ↓
    Resolution

---

# 166. No Runbook for Common Incidents

Useful runbooks include:

    CrashLoopBackOff
    +
    ImagePullBackOff
    +
    OOMKilled
    +
    High CPU
    +
    High Memory
    +
    503 Errors
    +
    Deployment Failure
    +
    Database Connectivity

---

# 167. Troubleshooting Without a Process

Bad:

    Problem
        |
        ↓
    Random Commands

Better:

    Observe
        |
        ↓
    Collect Evidence
        |
        ↓
    Form Hypothesis
        |
        ↓
    Test Hypothesis
        |
        ↓
    Fix
        |
        ↓
    Validate

---

# 168. Changing Production During Investigation Without Evidence

Bad:

    Issue
        |
        ↓
    Random Configuration Change
        |
        ↓
    New Problem

Better:

    Evidence
        |
        ↓
    Hypothesis
        |
        ↓
    Controlled Change
        |
        ↓
    Validation

---

# 169. Not Checking Recent Changes

Many incidents correlate with recent:

    Deployments
    +
    Configuration Changes
    +
    Infrastructure Changes
    +
    Dependency Updates

Always check the timeline.

---

# 170. Not Checking Logs Before Changing Configuration

Logs often contain valuable evidence.

Use:

    kubectl logs
    +
    kubectl logs --previous
    +
    kubectl describe
    +
    Application Logs

before making unnecessary changes.

---

# 171. Not Checking Events

For Kubernetes issues:

    kubectl describe pod <pod>

The Events section can reveal:

    Scheduling Problems
    +
    Image Pull Failures
    +
    Probe Failures
    +
    Mount Failures
    +
    Resource Problems

---

# 172. Not Checking Previous Container Logs

For restarted containers:

    kubectl logs <pod> --previous

can reveal information from the previous crashed container instance.

---

# 173. Ignoring Kubernetes Events

Events can help identify:

    FailedScheduling
    +
    FailedMount
    +
    Unhealthy
    +
    BackOff
    +
    ImagePullBackOff

---

# 174. No Production Readiness Checklist

Before production:

    Application Tested
        +
    Security Scanned
        +
    Resource Requirements Verified
        +
    Health Checks Configured
        +
    Monitoring Configured
        +
    Alerts Configured
        +
    Rollback Tested
        +
    Documentation Ready

---

# 175. No Pre-Deployment Checklist

A deployment checklist can include:

    Version
    +
    Artifact
    +
    Change Request
    +
    Approval
    +
    Database Changes
    +
    Infrastructure Changes
    +
    Rollback
    +
    Monitoring

---

# 176. No Post-Deployment Checklist

After deployment:

    Pods Healthy
        |
        ↓
    Services Healthy
        |
        ↓
    Health Checks Pass
        |
        ↓
    Logs Normal
        |
        ↓
    Metrics Normal
        |
        ↓
    Smoke Tests Pass
        |
        ↓
    Deployment Confirmed

---

# 177. No Smoke Testing

A deployment should be validated with critical application paths.

Example:

    Deployment
        |
        ↓
    Health Check
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
    Validation

---

# 178. Testing Only in Development

A system that works in development may fail in production due to:

    Scale
    +
    Traffic
    +
    Configuration
    +
    Dependencies
    +
    Infrastructure

Use appropriate testing across environments.

---

# 179. Environment Drift

Example:

    Development
        |
        ↓
    Configuration A

    Production
        |
        ↓
    Configuration B

Differences can cause unexpected behavior.

Use Infrastructure as Code and controlled configuration.

---

# 180. Ignoring Production-Like Testing

Where practical, test important behavior under conditions resembling production:

    Traffic
    +
    Dependencies
    +
    Configuration
    +
    Resource Limits

---

# 181. No Load Testing Before Major Releases

For high-impact changes:

    Expected Load
        |
        ↓
    Load Test
        |
        ↓
    Analyze
        |
        ↓
    Optimize
        |
        ↓
    Release

---

# 182. No Failure Testing

Systems should be tested for appropriate failure scenarios.

Examples:

    Pod Failure
    +
    Node Failure
    +
    Dependency Failure
    +
    Database Failure
    +
    Network Failure

---

# 183. No Chaos / Resilience Testing

Where appropriate, controlled failure testing can validate:

    Recovery
    +
    Failover
    +
    Resilience
    +
    Alerting

Always perform such testing within approved boundaries.

---

# 184. No Dependency Failure Handling

Bad:

    External Service
        |
        X
    Application
        |
        ↓
    Entire Application Fails

Better:

    External Service
        |
        X
    Controlled Timeout
        |
        ↓
    Graceful Degradation

---

# 185. No Circuit Breaker

Repeated calls to an unhealthy service can amplify failure.

Use circuit-breaking patterns where appropriate.

---

# 186. No Backpressure

A fast producer can overwhelm a slow consumer.

Bad:

    Producer
        |
        ↓
    Consumer
        |
        ↓
    Overload

Better:

    Producer
        |
        ↓
    Queue
        |
        ↓
    Controlled Processing

---

# 187. Ignoring Queue Backlog

A growing queue can indicate:

    Slow Consumers
    +
    Insufficient Capacity
    +
    Processing Failure

Monitor:

    Queue Depth
    +
    Consumer Rate
    +
    Processing Latency

---

# 188. No Idempotency

Retries can duplicate operations.

Example:

    Request
        |
        ↓
    Payment
        |
        ↓
    Timeout
        |
        ↓
    Retry
        |
        ↓
    Payment Again

Use idempotency mechanisms for operations that require them.

---

# 189. Non-Idempotent Deployment Scripts

A script that behaves differently each time can cause deployment problems.

Bad:

    Run 1 → Create
    Run 2 → Fail
    Run 3 → Different Result

Better:

    Desired State
        |
        ↓
    Repeated Execution
        |
        ↓
    Same Correct Result

---

# 190. No Retry/Error Handling in Automation

Automation should distinguish:

    Retryable Error
        +
    Non-Retryable Error

Do not retry every failure blindly.

---

# 191. No Pipeline Timeout

A stuck pipeline can consume resources indefinitely.

Use appropriate:

    timeout-minutes

where supported.

---

# 192. No Concurrency Control

Multiple deployments can race.

Example:

    Deployment A
        |
        ↓
    Production

    Deployment B
        |
        ↓
    Production

This can create inconsistent state.

Use appropriate concurrency controls.

---

# 193. Deploying Multiple Versions Simultaneously Without Control

Bad:

    Version A
        +
    Version B
        +
    Version C

without knowing which traffic reaches which version.

Use:

    Rolling
    +
    Canary
    +
    Blue-Green

with clear traffic management.

---

# 194. No Release Freeze During Critical Incidents

During major incidents, uncontrolled changes can make troubleshooting harder.

Use appropriate change-control procedures.

---

# 195. No Post-Incident Actions

After an incident:

    Fix
        |
        ↓
    Close Ticket

is not enough.

Better:

    Incident
        |
        ↓
    Root Cause
        |
        ↓
    Corrective Action
        |
        ↓
    Preventive Action
        |
        ↓
    Validation

---

# Common Mistakes Prevention Model

    PLAN
        |
        ↓
    REVIEW
        |
        ↓
    AUTOMATE
        |
        ↓
    VALIDATE
        |
        ↓
    MONITOR
        |
        ↓
    RESPOND
        |
        ↓
    IMPROVE

---

# DevOps Common Mistakes Checklist

## Source Control

    Use Git
    Protect Main Branch
    Use Pull Requests
    Require Reviews
    Use Meaningful Commits
    Use Release Tags
    Scan Secrets

## CI/CD

    Automate Builds
    Run Tests
    Cache Dependencies
    Parallelize Jobs
    Scan Dependencies
    Scan Containers
    Use Security Gates
    Use Quality Gates
    Version Artifacts
    Protect Production

## Docker

    Use Minimal Images
    Use Multi-Stage Builds
    Use .dockerignore
    Avoid Secrets in Images
    Avoid Root
    Avoid Privileged Containers
    Scan Images
    Use Immutable References

## Kubernetes

    Configure Requests
    Configure Limits
    Configure Probes
    Use HPA Appropriately
    Monitor Nodes
    Use RBAC
    Use Network Policies
    Use Non-Privileged Containers
    Spread Replicas
    Test Rollbacks

## AWS

    Use Least Privilege
    Use Private Networking
    Restrict Security Groups
    Encrypt Data
    Protect S3
    Protect ECR
    Protect RDS
    Monitor Resources

## Terraform

    Use Remote State
    Protect State
    Run Validate
    Run Plan
    Review Changes
    Scan Infrastructure
    Avoid Hardcoded Secrets
    Use Controlled Apply

## GitOps

    Git as Source of Truth
    Avoid Manual Drift
    Monitor ArgoCD
    Review Changes
    Use Versioned Manifests
    Test Rollbacks

## Monitoring

    Prometheus
    Grafana
    ELK
    Latency
    Error Rate
    CPU
    Memory
    Request Rate
    Queue Depth
    Alerts

## Operations

    Runbooks
    Incident Response
    Backup
    Restore Testing
    DR Testing
    Capacity Planning
    Cost Monitoring
    Change Management

---

# Interview Questions

## Basic

1. What are common DevOps mistakes?

2. Why should secrets not be committed to Git?

3. Why should containers not run as root?

4. Why should Docker images be small?

5. What is the purpose of resource requests and limits?

6. What is the difference between readiness and liveness probes?

7. Why is monitoring important?

8. Why should Terraform state be stored remotely?

9. Why should production deployments be automated?

10. Why should branches be protected?

---

# Intermediate

11. What mistakes can cause Kubernetes pods to restart repeatedly?

12. How can incorrect resource limits affect applications?

13. What happens when HPA scales an application but the database cannot handle the additional load?

14. What are common Docker security mistakes?

15. What are common CI/CD security mistakes?

16. How would you troubleshoot a slow CI pipeline?

17. How would you troubleshoot a failed Kubernetes deployment?

18. How would you prevent configuration drift?

19. How would you design a rollback strategy?

20. What are common Terraform mistakes?

21. How would you handle a secret accidentally committed to Git?

22. How would you secure GitHub Actions?

23. How would you prevent developers from directly changing production?

24. How would you handle a critical vulnerability discovered during CI?

25. Why should post-deployment validation be performed?

---

# Advanced

26. How would you design a DevOps platform that minimizes operational mistakes?

27. How would you implement guardrails across CI/CD?

28. How would you prevent production configuration drift?

29. How would you design a secure multi-environment deployment architecture?

30. How would you reduce human error in production deployments?

31. How would you design failure handling for a microservices platform?

32. How would you prevent retry storms?

33. How would you design a reliable GitOps architecture?

34. How would you prevent supply-chain vulnerabilities?

35. How would you design production change management?

36. How would you implement separation of duties in CI/CD?

37. How would you build a production readiness checklist?

38. How would you design incident response and post-incident improvement?

39. How would you balance automation with approval controls?

40. How would you identify systemic DevOps problems instead of treating individual incidents separately?

---

# Interview Scenario

## Deployment Succeeded but Application Returns 503

Answer:

    I would not assume that the deployment was successful just
    because the deployment command completed.

    I would check pod status, readiness probes, service endpoints,
    ingress or ALB configuration, application logs, and recent
    changes.

Flow:

    Deployment Successful
        |
        ↓
    503 Error
        |
        ↓
    Check Pods
        |
        ↓
    Check Readiness
        |
        ↓
    Check Service
        |
        ↓
    Check Endpoints
        |
        ↓
    Check ALB / Ingress
        |
        ↓
    Check Logs
        |
        ↓
    Identify Root Cause
        |
        ↓
    Fix / Rollback

---

# Interview Scenario

## Terraform Apply Failed Halfway

Answer:

    I would first inspect the Terraform error and state.

    I would not immediately destroy everything.

    I would determine which resources were successfully created,
    verify the Terraform state, fix the underlying issue, and
    rerun the plan.

Flow:

    terraform apply
        |
        ↓
    Failure
        |
        ↓
    Inspect Error
        |
        ↓
    Check State
        |
        ↓
    terraform plan
        |
        ↓
    Fix Root Cause
        |
        ↓
    Apply
        |
        ↓
    Validate

---

# Interview Scenario

## CI Pipeline Takes 25 Minutes

Answer:

    I would measure the duration of every stage and identify the
    bottleneck.

    I would evaluate dependency caching, Docker layer caching,
    parallel execution, test optimization, artifact reuse, and
    unnecessary repeated operations.

Flow:

    25 Minutes
        |
        ↓
    Measure Stages
        |
        ↓
    Find Bottleneck
        |
        ↓
    Cache
        |
        ↓
    Parallelize
        |
        ↓
    Optimize
        |
        ↓
    Measure Again

---

# Interview Scenario

## Production Has Configuration Drift

Answer:

    I would compare the desired configuration in Git with the
    actual production state.

    If using GitOps, I would inspect ArgoCD for drift and determine
    whether the change was intentional or unauthorized.

    I would restore the desired state through the approved process
    and identify why the manual change occurred.

Flow:

    Git
        |
        ↓
    Desired State

    Production
        |
        ↓
    Actual State

    Difference
        |
        ↓
    Investigate
        |
        ↓
    Reconcile
        |
        ↓
    Prevent Recurrence

---

# Interview Scenario

## Secret Accidentally Committed

Answer:

    I would immediately treat the secret as compromised.

    I would revoke or rotate it, investigate possible usage,
    remove the exposure from the repository where appropriate,
    and add preventive controls such as secret scanning.

Flow:

    Secret Exposure
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
    Clean Exposure
        |
        ↓
    Add Secret Scanning

---

# Interview Scenario

## Application Is Slow After Scaling

Answer:

    I would check whether scaling increased load on a downstream
    dependency.

    I would investigate database connections, database CPU,
    network traffic, external services, and application latency.

    Scaling the application does not automatically solve a
    downstream bottleneck.

Flow:

    Scale Application
        |
        ↓
    More Requests
        |
        ↓
    Database
        |
        ↓
    Connection Saturation
        |
        ↓
    Higher Latency

Better:

    Identify Bottleneck
        |
        ↓
    Optimize Correct Layer

---

# Interview Scenario

## Kubernetes Pods Are OOMKilled

Answer:

    I would check pod events and memory usage, then determine
    whether the application has a memory leak, traffic increased,
    or the configured memory limit is too low.

    I would compare actual usage with requests and limits and
    then apply the appropriate fix.

Flow:

    OOMKilled
        |
        ↓
    Check Events
        |
        ↓
    Check Memory
        |
        ↓
    Check Application
        |
        ↓
    Check Limits
        |
        ↓
    Fix
        |
        ↓
    Monitor

---

# Interview Scenario

## Production Deployment Failed

Answer:

    First I would determine whether the service is currently
    impacted.

    If required, I would execute the rollback strategy to restore
    service.

    Then I would investigate logs, events, health checks, recent
    changes, and deployment output to identify the root cause.

Flow:

    Deployment Failure
        |
        ↓
    Check Impact
        |
        +------ High Impact
        |          |
        |          ↓
        |       Rollback
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
    Validate

---

# Final Common Mistakes Mental Model

Remember:

    AUTOMATE
        |
        ↓
    STANDARDIZE
        |
        ↓
    VALIDATE
        |
        ↓
    MONITOR
        |
        ↓
    DETECT
        |
        ↓
    RECOVER
        |
        ↓
    IMPROVE

Good DevOps is not about avoiding every failure.

It is about building systems where:

    Failures Are Detected Quickly
        +
    Failures Have Limited Impact
        +
    Recovery Is Fast
        +
    Changes Are Traceable
        +
    Security Is Built In
        +
    Processes Are Repeatable
        +
    Lessons Become Improvements

---

# Final Concept

The biggest DevOps mistake is treating DevOps as a collection of tools.

DevOps is a combination of:

    People
        +
    Process
        +
    Automation
        +
    Security
        +
    Infrastructure
        +
    Monitoring
        +
    Continuous Improvement

The objective is:

    Fewer Manual Errors
        +
    Faster Feedback
        +
    Safer Deployments
        +
    Better Reliability
        +
    Better Security
        +
    Predictable Operations