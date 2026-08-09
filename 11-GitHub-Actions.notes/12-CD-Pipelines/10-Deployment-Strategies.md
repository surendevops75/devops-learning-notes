# Deployment Strategies

Deployment strategy defines how a new application version is released to an environment and how production traffic is moved from the old version to the new version.

The main deployment strategies are:

    Recreate
    Rolling
    Blue-Green
    Canary

Other advanced strategies include:

    A/B Testing
    Shadow Deployment
    Feature Flags
    Progressive Delivery

The correct strategy depends on:

    Application Architecture
    Availability Requirements
    Risk
    Traffic Management
    Infrastructure Cost
    Rollback Requirements
    Database Compatibility

---

# Deployment Strategy Mental Model

Remember:

    Recreate
        |
        ↓
    Stop Old
        |
        ↓
    Deploy New


    Rolling
        |
        ↓
    Replace Instances Gradually


    Blue-Green
        |
        ↓
    Maintain Two Environments
        |
        ↓
    Switch Traffic


    Canary
        |
        ↓
    Small Traffic
        |
        ↓
    Gradually Increase

---

# Why Deployment Strategies Matter

A deployment is not only about installing a new application version.

A production deployment must answer:

    How will the new version be released?

    How much traffic will it receive?

    How will we validate it?

    What happens if it fails?

    How quickly can we rollback?

    How much infrastructure is required?

Deployment strategies provide structured answers to these questions.

---

# Recreate Deployment

Recreate is the simplest deployment strategy.

The existing application instances are stopped before the new version is deployed.

Flow:

    Old Version
        |
        ↓
    Stop Old Instances
        |
        ↓
    Deploy New Version
        |
        ↓
    Start New Instances
        |
        ↓
    Production

---

# Recreate Example

Initial:

    Pod 1 → v1.4.6
    Pod 2 → v1.4.6
    Pod 3 → v1.4.6

Deployment:

    Stop All Pods
        |
        ↓
    No Application Instances
        |
        ↓
    Deploy v1.4.7
        |
        ↓
    Start New Pods

Final:

    Pod 1 → v1.4.7
    Pod 2 → v1.4.7
    Pod 3 → v1.4.7

---

# Recreate Advantages

- Simple implementation
- Easy to understand
- No mixed application versions
- Useful when old and new versions cannot coexist
- Lower temporary infrastructure requirements

---

# Recreate Disadvantages

- Downtime may occur
- Not suitable for highly available applications
- Rollback may require redeployment
- Users may experience service interruption
- Not ideal for critical production applications

---

# Rolling Deployment

Rolling deployment gradually replaces old instances with new instances.

Example:

    Old:
        6 Pods → v1.4.6

During deployment:

    2 Pods → v1.4.7
    4 Pods → v1.4.6

Then:

    4 Pods → v1.4.7
    2 Pods → v1.4.6

Finally:

    6 Pods → v1.4.7

---

# Rolling Deployment Mental Model

    OLD
    v1.4.6
       |
       ↓
    Replace Some
       |
       ↓
    Validate
       |
       ↓
    Replace More
       |
       ↓
    Validate
       |
       ↓
    Continue
       |
       ↓
    NEW
    v1.4.7

---

# Kubernetes RollingUpdate

Kubernetes Deployments support:

    strategy:
      type: RollingUpdate

Important parameters:

    maxUnavailable
    maxSurge

Example:

    strategy:
      type: RollingUpdate

      rollingUpdate:
        maxUnavailable: 1
        maxSurge: 1

---

# maxUnavailable

`maxUnavailable` controls how many desired Pods can be unavailable during the rollout.

Example:

    replicas: 10

    maxUnavailable: 2

During deployment:

    Up to 2 Pods
    may be unavailable

The value should be selected according to application availability requirements.

---

# maxSurge

`maxSurge` controls how many additional Pods can temporarily exist above the desired replica count.

Example:

    replicas: 10

    maxSurge: 2

During deployment:

    Maximum Pods ≈ 12

This requires additional cluster capacity.

---

# Rolling Deployment Advantages

- Kubernetes native
- Lower cost than Blue-Green
- Gradual replacement
- Reduced downtime
- Simple implementation
- Supports automated rollback
- Works well with multiple replicas

---

# Rolling Deployment Disadvantages

- Old and new versions can coexist
- Database compatibility is important
- Rollout can become slow
- Requires good readiness checks
- Requires sufficient cluster capacity
- Debugging mixed-version behavior can be difficult

---

# Blue-Green Deployment

Blue-Green maintains two environments.

    Blue
        |
        └── Current Production

    Green
        |
        └── New Version

Traffic is initially sent to Blue.

After Green is validated:

    Traffic → Green

If Green fails:

    Traffic → Blue

---

# Blue-Green Mental Model

    BLUE
    v1.4.6
      |
      ↓
    100% Traffic


    GREEN
    v1.4.7
      |
      ↓
    Testing

After validation:

    GREEN
    v1.4.7
      |
      ↓
    100% Traffic

---

# Blue-Green Advantages

- Fast rollback
- Strong production-like validation
- Reduced deployment risk
- Simple traffic switching
- Previous version remains available
- Can provide minimal downtime

---

# Blue-Green Disadvantages

- Higher infrastructure cost
- Two environments may be required
- Database compatibility is still important
- Traffic switching architecture is required
- More infrastructure management

---

# Canary Deployment

Canary deployment gradually exposes the new version to production traffic.

Example:

    Stable:
        95%

    Canary:
         5%

Then:

    Stable:
        75%

    Canary:
        25%

Then:

    Stable:
        50%

    Canary:
        50%

Finally:

    Stable:
         0%

    Canary:
       100%

---

# Canary Mental Model

    New Version
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

# Canary Advantages

- Small initial blast radius
- Real production traffic
- Gradual exposure
- Early failure detection
- Progressive rollout
- Fast rollback
- Good for high-risk releases

---

# Canary Disadvantages

- More complex traffic management
- Strong monitoring required
- Multiple application versions run simultaneously
- Database compatibility required
- Progressive rollout takes longer
- Requires automation for mature implementations

---

# A/B Testing

A/B testing routes different users to different application experiences.

Example:

    Users
      |
      ↓
    Traffic Router
      |
      +--------→ Version A
      |
      └--------→ Version B

The goal is usually to compare:

    User Behavior
    Conversion
    Performance
    Business Metrics

A/B testing is more focused on comparing experiences or features than simply replacing an application version.

---

# A/B Testing Example

Suppose:

    Version A:
        Existing Checkout

    Version B:
        New Checkout

Traffic:

    50% → A
    50% → B

Measure:

    Conversion Rate
    Cart Completion
    Response Time
    Error Rate

Then compare results.

---

# Shadow Deployment

Shadow deployment sends a copy of production traffic to a new version without using its responses for real users.

Example:

    User
      |
      ↓
    Production
      |
      +--------→ Stable
      |
      └--------→ Shadow

The Stable version responds to the user.

The Shadow version processes copied traffic but its response is not returned to the user.

---

# Shadow Deployment Mental Model

    User Request
        |
        ↓
    Traffic Router
        |
        +--------→ Stable
        |              |
        |              ↓
        |          User Response
        |
        └--------→ Shadow
                       |
                       ↓
                 Discard Response

---

# Shadow Deployment Advantages

- Real production traffic
- New version can be tested safely
- No direct user impact
- Useful for performance testing
- Useful for large application changes
- Helps identify production-only issues

---

# Shadow Deployment Disadvantages

- Additional infrastructure
- Duplicate processing
- Database side effects must be controlled
- External API calls can become dangerous
- More complex architecture
- Requires careful data handling

---

# Feature Flags

Feature flags separate deployment from feature activation.

Example:

    Application v1.4.7
          |
          ↓
    Feature Flag
          |
          +---- Disabled → Old Behavior
          |
          └---- Enabled → New Behavior

The application can be deployed before the feature is enabled.

---

# Feature Flag Deployment

Flow:

    Deploy Code
        |
        ↓
    Feature Disabled
        |
        ↓
    Validate Application
        |
        ↓
    Enable for Small Group
        |
        ↓
    Monitor
        |
        ↓
    Increase Users
        |
        ↓
    Enable for Everyone

---

# Feature Flags vs Canary

Canary controls:

    Application Version Exposure

Feature flags control:

    Feature Exposure

They can be used together.

Example:

    Application v1.4.7
        |
        ↓
    Canary 5%
        |
        ↓
    Feature Flag
        |
        ↓
    New Feature

---

# Progressive Delivery

Progressive Delivery is a broader approach that gradually introduces changes while continuously evaluating application health.

It can combine:

    Canary
    Blue-Green
    Feature Flags
    Automated Analysis
    Observability
    Rollback

Mental model:

    Release
       |
       ↓
    Small Exposure
       |
       ↓
    Observe
       |
       ↓
    Increase
       |
       ↓
    Observe
       |
       ↓
    Full Release

---

# Progressive Delivery Architecture

    Git
      |
      ↓
    CI/CD
      |
      ↓
    Kubernetes
      |
      ↓
    Progressive Rollout
      |
      ↓
    Metrics
      |
      ↓
    Automated Analysis
      |
      +------ Healthy ------→ Continue
      |
      +------ Unhealthy ----→ Rollback

---

# Deployment Strategy Comparison

| Strategy | Traffic Approach | Downtime | Cost | Rollback | Complexity |
|---|---|---|---|---|---|
| Recreate | Stop then start | Possible | Low | Slower | Low |
| Rolling | Gradual Pod replacement | Low/Minimal | Low/Medium | Good | Medium |
| Blue-Green | Full environment switch | Low | High | Very Fast | Medium |
| Canary | Gradual traffic | Low | Medium | Fast | High |
| Shadow | Duplicate traffic | No user-facing downtime | High | Easy | High |
| A/B | User segmentation | Low | Medium/High | Depends | High |

---

# Deployment Strategy Selection

Use Recreate when:

    Old and New Cannot Coexist
    Downtime Is Acceptable
    Application Is Simple

Use Rolling when:

    Kubernetes Is Used
    Multiple Replicas Exist
    Lower Cost Is Important
    Gradual Instance Replacement Is Enough

Use Blue-Green when:

    Fast Rollback Is Critical
    Two Environments Are Affordable
    Strong Validation Is Required

Use Canary when:

    Gradual Traffic Is Important
    Production Risk Must Be Reduced
    Strong Monitoring Exists

Use Shadow when:

    Real Production Traffic Is Needed
    User Responses Must Remain Unchanged
    New Version Requires Deep Validation

Use A/B when:

    Different User Experiences Need Comparison
    Business Metrics Matter

---

# Deployment Strategy Decision Tree

    New Version Ready
          |
          ↓
    Can Old and New Coexist?
          |
          +------ No ------→ Recreate
          |
          +------ Yes
                    |
                    ↓
            Need Gradual Traffic?
                    |
                    +------ No ------→ Rolling
                    |
                    +------ Yes
                              |
                              ↓
                       Need Full Environment?
                              |
                              +------ Yes → Blue-Green
                              |
                              +------ No
                                        |
                                        ↓
                                    Canary

---

# Recreate vs Rolling

Recreate:

    Stop Everything
        |
        ↓
    Deploy New Version

Rolling:

    Keep Existing
        |
        ↓
    Replace Gradually

Rolling is generally better for highly available applications.

---

# Rolling vs Blue-Green

Rolling:

    Old + New Pods
        |
        ↓
    Gradual Replacement

Blue-Green:

    Environment A
        +
    Environment B
        |
        ↓
    Traffic Switch

Rolling normally uses fewer resources.

Blue-Green normally provides a cleaner and faster environment-level rollback.

---

# Rolling vs Canary

Rolling:

    Replace Pods

Canary:

    Control Traffic

Example:

    Rolling:

    8 Old Pods
    2 New Pods

Canary:

    95% Users
     5% Users

The number of Pods does not automatically guarantee the exact traffic percentage.

---

# Blue-Green vs Canary

Blue-Green:

    100% → Blue

Then:

    100% → Green

Canary:

    95% → Stable
     5% → Canary

Then gradually:

    75% → Stable
    25% → Canary

Canary provides more gradual exposure.

---

# Canary vs A/B

Canary focuses on:

    Safe Release

A/B focuses on:

    User or Business Comparison

Canary question:

    Is the new version healthy?

A/B question:

    Which experience performs better?

---

# Blue-Green vs Shadow

Blue-Green:

    New version can eventually receive real production responses.

Shadow:

    New version receives copied traffic but does not normally serve the user response.

Shadow is useful for validating behavior without changing user-facing responses.

---

# Rolling Deployment With Kubernetes

Architecture:

    Users
      |
      ↓
    ALB
      |
      ↓
    Service
      |
      ↓
    Deployment
      |
      +-- Old ReplicaSet
      |
      └-- New ReplicaSet

Kubernetes gradually changes the ReplicaSets.

---

# Blue-Green With Kubernetes

Architecture:

    Users
      |
      ↓
    ALB
      |
      ↓
    Routing
      |
      +-- Blue Deployment
      |
      └-- Green Deployment

Traffic switches between environments.

---

# Canary With Kubernetes

Architecture:

    Users
      |
      ↓
    Traffic Management
      |
      +-- Stable
      |
      └-- Canary

Traffic percentage changes progressively.

---

# Deployment Strategies With EKS

Amazon EKS can support multiple strategies.

    EKS
      |
      +-- Rolling Deployment
      |
      +-- Blue-Green
      |
      +-- Canary
      |
      +-- Shadow
      |
      └-- Progressive Delivery

Kubernetes provides the base platform while ingress, load balancers, service meshes, or progressive delivery tools can provide advanced traffic control.

---

# Deployment Strategies With ALB

Conceptually:

    Route 53
       |
       ↓
      ALB
       |
       +-- Stable
       |
       +-- Canary
       |
       └-- Blue/Green

ALB can participate in traffic management depending on the chosen architecture and routing configuration.

---

# Deployment Strategies With ArgoCD

ArgoCD provides GitOps-based deployment synchronization.

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
    Kubernetes Deployment

For advanced progressive delivery:

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

---

# Deployment Strategies With Helm

Helm can package and configure Kubernetes deployments.

Example:

    Git
      |
      ↓
    Helm Chart
      |
      ↓
    Kubernetes
      |
      ↓
    Deployment Strategy

Helm can define:

    Image
    Replicas
    Strategy
    Probes
    Resources
    Configuration

---

# Deployment Strategies With GitHub Actions

Typical CI/CD flow:

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
    Deployment
        |
        +-- Rolling
        |
        +-- Blue-Green
        |
        +-- Canary
        |
        ↓
    Validation
        |
        ↓
    Monitoring

---

# Deployment Strategies With Jenkins

Jenkins can orchestrate:

    Build
    Test
    Security Scan
    Image Build
    Image Push
    Deployment
    Validation
    Rollback

Example:

    Jenkins
       |
       ↓
    CI
       |
       ↓
    ECR
       |
       ↓
    Kubernetes
       |
       ↓
    Deployment Strategy

---

# DevSecOps Deployment Strategy

Security should happen before production traffic reaches the new version.

Flow:

    Code
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
    Trivy
      |
      ↓
    Image
      |
      ↓
    ECR
      |
      ↓
    Deployment Strategy
      |
      ↓
    Health Checks
      |
      ↓
    Production

---

# Deployment Strategy and Security Gates

Example:

    Build
      |
      ↓
    SonarQube
      |
      ↓
    Trivy
      |
      ↓
    Security Gate
      |
      +------ Fail ------→ Stop
      |
      +------ Pass
              |
              ↓
          Deployment
              |
              ↓
           Validation

Do not promote an image that fails required security gates.

---

# Deployment Strategy and Quality Gates

Quality checks can include:

    Unit Tests
    Integration Tests
    Code Quality
    Vulnerability Scans
    Smoke Tests
    Health Checks

Flow:

    CI
      |
      ↓
    Quality Gate
      |
      +------ Fail ------→ Stop
      |
      +------ Pass
              |
              ↓
          Deployment

---

# Deployment Strategy and Observability

Every production deployment should be observable.

Monitor:

    Metrics
    Logs
    Health
    Errors
    Latency
    Availability
    Resource Usage

Stack:

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

---

# Deployment Strategy and Monitoring

Before deployment:

    Baseline Metrics

During deployment:

    Compare New vs Old

After deployment:

    Verify Metrics

Example:

    Before:
        Error Rate = 0.5%

    During:
        Error Rate = 0.6%

    After:
        Error Rate = 0.5%

Deployment appears healthy.

---

# Deployment Strategy and Error Budget

A production deployment should respect application reliability targets.

If a deployment causes:

    Error Rate Increase
    Latency Increase
    Availability Reduction

the rollout should be slowed, paused, or rolled back according to operational policies.

---

# Deployment Strategy and Rollback

Every deployment strategy should define:

    Detection
        |
        ↓
    Decision
        |
        ↓
    Rollback
        |
        ↓
    Validation

Rollback mechanism depends on strategy.

Recreate:

    Redeploy Previous Version

Rolling:

    Kubernetes Rollback

Blue-Green:

    Switch Traffic Back

Canary:

    Set Canary Traffic to 0%

---

# Rollback Speed Comparison

Generally:

    Blue-Green
        |
        ↓
    Very Fast

    Canary
        |
        ↓
    Fast

    Rolling
        |
        ↓
    Fast

    Recreate
        |
        ↓
    Slower

Actual rollback speed depends on implementation and infrastructure.

---

# Database Compatibility

Deployment strategy cannot solve an incompatible database migration by itself.

During many deployment strategies:

    Old Application
        +
    New Application
        |
        ↓
    Same Database

Therefore use:

    Backward-Compatible Changes
        |
        ↓
    Expand
        |
        ↓
    Migrate
        |
        ↓
    Contract

---

# Expand and Contract Pattern

Example:

    Step 1
        |
        ↓
    Add New Database Field

    Step 2
        |
        ↓
    Application Supports Old + New

    Step 3
        |
        ↓
    Deploy New Version

    Step 4
        |
        ↓
    Migrate Data

    Step 5
        |
        ↓
    Remove Old Field Later

This reduces rollback risk.

---

# Deployment Strategy and API Compatibility

During deployment:

    v1.4.6
        +
    v1.4.7

may communicate with other services.

Use:

    Backward-Compatible APIs

Example:

    v1.4.6 → API v1
    v1.4.7 → API v1

Avoid breaking API contracts during a mixed-version rollout.

---

# Deployment Strategy and Microservices

For microservices:

    User
    Product
    Cart
    Orders
    Payment
    Inventory
    Notification

Each service can use an appropriate strategy.

Example:

    Payment → Canary

    Product → Rolling

    Orders → Blue-Green

The strategy does not have to be identical for every service.

---

# Deployment Strategy Per Risk

Low-risk service:

    Rolling

Medium-risk service:

    Canary

High-risk release:

    Blue-Green

Special testing requirement:

    Shadow

Business experiment:

    A/B

The strategy should match the release risk.

---

# Deployment Strategy by Application Type

Stateless Web Application:

    Rolling
    Blue-Green
    Canary

Microservices:

    Rolling
    Canary
    Blue-Green

High-Risk Application:

    Canary
    Blue-Green

Legacy Application:

    Recreate
    Blue-Green

Performance Validation:

    Shadow

Feature Experiment:

    A/B

---

# Stateless Application

Stateless applications are easier to deploy progressively.

Example:

    Web Pods
        |
        +-- v1.4.6
        +-- v1.4.6
        +-- v1.4.7
        +-- v1.4.7

Requests can be distributed across available Pods.

---

# Stateful Application

Stateful applications require additional planning.

Consider:

    Database
    Persistent Storage
    Sessions
    Message Queues
    Transactions
    Data Consistency

Deployment strategy alone does not guarantee safe state transitions.

---

# Session Management

If sessions are stored locally:

    User
      |
      ↓
    Pod A
      |
      ↓
    Session

During rollout:

    User
      |
      ↓
    Pod B
      |
      ↓
    Session Missing

Possible solutions include:

    External Session Store
    Sticky Sessions
    Stateless Authentication

The correct solution depends on the application.

---

# Deployment Strategy and Message Queues

For applications using:

    RabbitMQ

or other messaging systems, mixed application versions may process messages simultaneously.

Consider:

    Message Schema Compatibility
    Consumer Compatibility
    Producer Compatibility
    Retry Behavior
    Dead Letter Queues

---

# Deployment Strategy and External Dependencies

During rollout, the new version may call:

    Payment APIs
    Authentication APIs
    Databases
    Message Queues
    External Services

Validate:

    Credentials
    API Contracts
    Network Access
    Timeouts
    Retry Behavior

---

# Deployment Strategy and Timeouts

New deployments can expose latency problems.

Use appropriate:

    Connection Timeout
    Read Timeout
    Request Timeout
    Retry Policy

Avoid unlimited retries because they can amplify failures.

---

# Deployment Strategy and Error Handling

Deployment systems should distinguish:

    Application Error
    Infrastructure Error
    Configuration Error
    Dependency Error
    Deployment Error

Example:

    New Pod
        |
        ↓
    Database Connection Failure
        |
        ↓
    Readiness Failure
        |
        ↓
    Deployment Stops

---

# Deployment Strategy and Health Checks

Health checks should cover:

    Infrastructure
    Container
    Application
    Dependencies

Example:

    Pod
      |
      ↓
    Readiness
      |
      ↓
    Application Health
      |
      ↓
    Dependency Health

Do not make health endpoints unnecessarily dependent on every external service if that would cause false failures.

---

# Deployment Strategy and Smoke Tests

Smoke tests validate critical application functionality.

Example:

    Health Endpoint
        |
        ↓
    Login
        |
        ↓
    Critical API
        |
        ↓
    Database
        |
        ↓
    Service Communication

Smoke tests should be fast enough to run during deployment.

---

# Deployment Strategy and Automated Rollback

Ideal flow:

    Deploy
      |
      ↓
    Health Check
      |
      ↓
    Metrics
      |
      ↓
    Analysis
      |
      +------ Healthy ------→ Continue
      |
      +------ Unhealthy ----→ Rollback

Automation reduces response time.

---

# Automated Rollback Example

    Canary
      |
      ↓
    Error Rate > Threshold
      |
      ↓
    Analysis Failed
      |
      ↓
    Abort
      |
      ↓
    Stable
      |
      ↓
    Alert
      |
      ↓
    Investigation

---

# Deployment Strategy and Change Management

Enterprise deployments may require:

    JIRA Change Request
    Approval
    Deployment Window
    Release Notes
    Rollback Plan
    Validation Plan
    Audit Trail

The deployment strategy operates inside the organization's change-management process.

---

# Production Deployment Workflow

    Development
        |
        ↓
    Code Review
        |
        ↓
    CI
        |
        ↓
    Security
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
    Approval
        |
        ↓
    Production
        |
        ↓
    Deployment Strategy
        |
        ↓
    Validation
        |
        ↓
    Monitoring

---

# Enterprise Deployment Strategy

Example:

    QA
      |
      ↓
    Rolling

    SIT
      |
      ↓
    Rolling

    UAT
      |
      ↓
    Blue-Green

    Production
      |
      ↓
    Canary

The strategy can vary by environment.

---

# Production Approval

Before production:

    Build
      |
      ↓
    Tests
      |
      ↓
    Security
      |
      ↓
    UAT
      |
      ↓
    Approval
      |
      ↓
    Production

The deployment strategy should not bypass required approval controls.

---

# Deployment Window

Some organizations use deployment windows.

Example:

    Approved Window
        |
        ↓
    Deploy
        |
        ↓
    Validate
        |
        ↓
    Monitor

If deployment fails:

    Rollback
        |
        ↓
    Incident / Change Failure

---

# Deployment Strategy and Audit

Track:

    Who Deployed
    What Version
    When Deployed
    Git Commit
    Docker Image
    Deployment Strategy
    Approval
    Result
    Rollback

This provides traceability.

---

# Deployment Strategy and Separation of Duties

Enterprise environments may separate:

    Developer
        |
        ↓
    Code Review
        |
        ↓
    CI
        |
        ↓
    Approval
        |
        ↓
    Deployment

This reduces the risk of unauthorized production changes.

---

# Deployment Strategy and GitOps

GitOps improves traceability.

Example:

    Git Commit
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Merge
        |
        ↓
    ArgoCD
        |
        ↓
    Deployment

The desired state is stored in Git.

---

# Deployment Strategy and Immutable Infrastructure

Immutable deployment means new versions are deployed as new immutable artifacts rather than modifying running instances manually.

Example:

    v1.4.6
        |
        ↓
    Immutable Image

    v1.4.7
        |
        ↓
    New Immutable Image

Deploy the new image instead of modifying the existing image.

---

# Deployment Strategy and Docker

Use:

    app:1.4.6

and:

    app:1.4.7

Avoid:

    app:latest

This improves:

    Traceability
    Rollback
    Reproducibility
    Debugging

---

# Deployment Strategy and ECR

Example:

    ECR
      |
      └── payment
           |
           ├── 1.4.5
           ├── 1.4.6
           └── 1.4.7

Deployment:

    Stable → 1.4.6
    New → 1.4.7

The exact image version should be known before deployment.

---

# Deployment Strategy and Terraform

Terraform can provision:

    VPC
    EKS
    IAM
    ECR
    ALB
    Security Groups
    Networking

Deployment tools can then manage:

    Kubernetes
    Helm
    ArgoCD

Keep infrastructure provisioning and application delivery responsibilities clear.

---

# Deployment Strategy and CI/CD

A mature pipeline looks like:

    Source
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
    Artifact
      |
      ↓
    Deploy
      |
      ↓
    Health Check
      |
      ↓
    Progressive Strategy
      |
      ↓
    Monitor
      |
      ↓
    Promote / Rollback

---

# Deployment Strategy and DevSecOps

Security is integrated into the deployment lifecycle.

    Code
      |
      ↓
    CI
      |
      +-- SonarQube
      +-- Trivy
      |
      ↓
    Artifact
      |
      ↓
    Deployment
      |
      ↓
    Runtime Validation
      |
      ↓
    Monitoring

---

# Deployment Strategy and SAST

SAST tools analyze source code.

Example:

    SonarQube
        |
        ↓
    Code Analysis
        |
        ↓
    Quality Gate

If the gate fails:

    Deployment Stops

---

# Deployment Strategy and Container Scanning

Trivy can scan container images.

Flow:

    Docker Image
        |
        ↓
    Trivy
        |
        ↓
    Vulnerability Results
        |
        +------ Fail ------→ Stop
        |
        +------ Pass ------→ Deploy

---

# Deployment Strategy and Quality Gates

Example:

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
    Quality/Security Gate
      |
      +------ Fail ------→ Stop
      |
      +------ Pass
              |
              ↓
          Deployment

---

# Deployment Strategy and Observability

A deployment is not complete when the Pods become Running.

It is complete when:

    Application Is Healthy
    Traffic Is Successful
    Error Rate Is Normal
    Latency Is Normal
    Dependencies Are Healthy
    Logs Are Normal

Therefore:

    Deployment
        +
    Observability
        =
    Reliable Release

---

# Deployment Strategy Production Checklist

Before deployment:

    [ ] Correct version
    [ ] Image exists
    [ ] Tests passed
    [ ] Security passed
    [ ] Quality gate passed
    [ ] Database compatibility verified
    [ ] API compatibility verified
    [ ] Capacity verified
    [ ] Health checks configured
    [ ] Monitoring ready
    [ ] Rollback ready

During deployment:

    [ ] Pods healthy
    [ ] No CrashLoopBackOff
    [ ] No ImagePullBackOff
    [ ] Readiness passing
    [ ] Error rate normal
    [ ] Latency normal
    [ ] Logs normal

After deployment:

    [ ] All replicas updated
    [ ] Smoke tests passed
    [ ] Metrics normal
    [ ] Logs normal
    [ ] Application stable
    [ ] Rollout completed
    [ ] Old version removed only when safe

---

# Deployment Strategy Troubleshooting

## Pods Are Pending

Check:

    kubectl describe pod <pod>

Look for:

    Insufficient CPU
    Insufficient Memory
    Scheduling Constraints
    Node Problems

---

# Deployment Strategy Troubleshooting

## Pods Are CrashLoopBackOff

Check:

    kubectl logs <pod>

    kubectl logs <pod> --previous

    kubectl describe pod <pod>

Check:

    Configuration
    Secrets
    Dependencies
    Application Startup
    Resources

---

# Deployment Strategy Troubleshooting

## ImagePullBackOff

Check:

    kubectl describe pod <pod>

Verify:

    Image Name
    Image Tag
    ECR Image
    IAM Permissions
    Registry Access

---

# Deployment Strategy Troubleshooting

## Readiness Probe Failure

Check:

    kubectl describe pod <pod>

Verify:

    Path
    Port
    Application Startup
    Service
    Network
    Configuration

---

# Deployment Strategy Troubleshooting

## High Error Rate

Check:

    Prometheus
    Grafana
    ELK
    Application Logs

Compare:

    Old Version
        vs
    New Version

Then:

    Stop Rollout
        |
        ↓
    Investigate
        |
        ↓
    Rollback if Necessary

---

# Deployment Strategy Troubleshooting

## High Latency

Check:

    CPU
    Memory
    Database
    Network
    External Dependencies
    Application Logs

If caused by the release:

    Rollback

---

# Deployment Strategy Troubleshooting

## Rollout Is Stuck

Check:

    kubectl rollout status deployment/<name>

Then:

    kubectl get pods

    kubectl describe deployment <name>

    kubectl describe pod <pod>

    kubectl get events

Look for:

    Failed Probes
    Pending Pods
    Image Errors
    Resource Constraints
    Scheduling Issues

---

# Deployment Strategy Troubleshooting

## Rollout Is Too Slow

Possible causes:

    Large Images
    Slow Startup
    Conservative maxSurge
    Conservative maxUnavailable
    Slow Health Checks
    Insufficient Capacity
    Slow Image Pull

Optimize carefully without reducing availability.

---

# Deployment Strategy Troubleshooting

## Rollback Is Not Safe

Possible cause:

    Incompatible Database Migration

Solution:

    Design backward-compatible migrations.

Also check:

    API Compatibility
    Configuration Compatibility
    Message Compatibility
    External Dependency Compatibility

---

# Deployment Strategy Interview Questions

## Basic Questions

1. What is a deployment strategy?
2. What are the common deployment strategies?
3. What is Recreate deployment?
4. What is Rolling deployment?
5. What is Blue-Green deployment?
6. What is Canary deployment?
7. What is A/B testing?
8. What is Shadow deployment?
9. What is Progressive Delivery?
10. Which strategy does Kubernetes support natively?

---

# Deployment Strategy Interview Questions

## Intermediate Questions

11. What is the difference between Rolling and Blue-Green?

12. What is the difference between Rolling and Canary?

13. What is the difference between Blue-Green and Canary?

14. What is maxUnavailable?

15. What is maxSurge?

16. How does Kubernetes RollingUpdate work?

17. How would you rollback a Rolling deployment?

18. How would you implement Blue-Green on EKS?

19. How would you implement Canary on EKS?

20. How would you monitor a Canary deployment?

---

# Deployment Strategy Interview Questions

## Advanced Questions

21. Design a zero-downtime deployment architecture for EKS.

22. Which deployment strategy would you choose for a high-risk production release and why?

23. How would you implement Canary with ArgoCD?

24. How would you implement automated rollback?

25. How would you handle database migrations during Rolling deployment?

26. How would you handle backward compatibility between old and new microservices?

27. How would you combine Canary deployment with feature flags?

28. How would you implement Shadow deployment?

29. How would you monitor and compare old and new application versions?

30. How would you choose between Rolling, Blue-Green, and Canary?

---

# Scenario-Based Interview Question

## Scenario

You have:

    10 Kubernetes Pods

Current version:

    v1.4.6

New version:

    v1.4.7

You need:

    Zero/Minimal Downtime
    Fast Rollback
    Low Infrastructure Cost

Possible choice:

    Rolling Deployment

Configuration:

    replicas: 10

    strategy:
      type: RollingUpdate

      rollingUpdate:
        maxUnavailable: 1
        maxSurge: 1

Use:

    Readiness Probe
    Liveness Probe
    Monitoring
    Smoke Tests
    Rollback

---

# Scenario-Based Interview Question

## Scenario

A payment service has a high-risk release.

Requirements:

    Production Validation
    Small Blast Radius
    Fast Rollback

Possible strategy:

    Canary

Flow:

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

# Scenario-Based Interview Question

## Scenario

A critical application needs:

    Very Fast Rollback
    Complete New Environment Validation
    Minimal Downtime

Possible strategy:

    Blue-Green

Flow:

    Blue
      |
      ↓
    Production

    Green
      |
      ↓
    New Version
      |
      ↓
    Validate
      |
      ↓
    Switch Traffic

If failure:

    Switch Back to Blue

---

# Scenario-Based Interview Question

## Scenario

A legacy application cannot support two application versions simultaneously.

Possible strategy:

    Recreate

Flow:

    Stop Old
      |
      ↓
    Deploy New
      |
      ↓
    Start New

Trade-off:

    Downtime

---

# Scenario-Based Interview Question

## Scenario

You want to test a new application version against real production traffic but cannot expose users to its responses.

Possible strategy:

    Shadow Deployment

Flow:

    User
      |
      ↓
    Stable
      |
      +--------→ Response to User
      |
      └--------→ Shadow
                      |
                      ↓
                 Analyze Result

---

# Scenario-Based Interview Question

## Scenario

Marketing wants to compare two checkout experiences.

Possible strategy:

    A/B Testing

Flow:

    Users
      |
      ↓
    Router
      |
      +-- Group A → Checkout A
      |
      └-- Group B → Checkout B

Measure:

    Conversion
    Revenue
    Completion Rate
    Errors
    Latency

---

# Scenario-Based Interview Question

## Scenario

You want to deploy a feature but activate it only for internal users first.

Possible solution:

    Feature Flag

Flow:

    Deploy Code
        |
        ↓
    Feature Disabled
        |
        ↓
    Enable Internal Users
        |
        ↓
    Monitor
        |
        ↓
    Enable More Users
        |
        ↓
    Enable Everyone

---

# Production Deployment Strategy

A mature production deployment process:

    Code
      |
      ↓
    Review
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
    Deployment
      |
      ↓
    Health Checks
      |
      ↓
    Progressive Release
      |
      ↓
    Monitoring
      |
      +------ Healthy ------→ Promote
      |
      +------ Unhealthy ----→ Rollback
      |
      ↓
    Production

---

# Complete DevOps Deployment Architecture

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Pull Request
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
    Amazon ECR
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    Amazon EKS
        |
        ↓
    Deployment Strategy
        |
        +-- Rolling
        |
        +-- Blue-Green
        |
        +-- Canary
        |
        +-- Shadow
        |
        ↓
    ALB / Ingress
        |
        ↓
    Users
        |
        ↓
    Monitoring
        |
        +-- Prometheus
        +-- Grafana
        └-- ELK

---

# Deployment Strategy Decision Matrix

| Requirement | Recommended Strategy |
|---|---|
| Simple deployment | Recreate |
| Low infrastructure cost | Rolling |
| Kubernetes standard deployment | Rolling |
| Fast rollback | Blue-Green |
| Production validation | Blue-Green |
| Small blast radius | Canary |
| Gradual traffic | Canary |
| Real traffic without user impact | Shadow |
| Compare user experiences | A/B |
| Gradual feature activation | Feature Flags |
| Automated progressive delivery | Canary / Progressive Delivery |

---

# Deployment Strategy Best Practices

- Select strategy based on application risk
- Understand rollback before deployment
- Use immutable image tags
- Maintain multiple replicas for high availability
- Configure readiness probes
- Configure liveness probes appropriately
- Use startup probes for slow applications
- Use graceful shutdown
- Monitor metrics
- Monitor logs
- Run smoke tests
- Use security gates
- Use quality gates
- Use backward-compatible database changes
- Maintain API compatibility
- Plan infrastructure capacity
- Automate rollback where appropriate
- Use GitOps for traceability
- Test deployment strategies before production
- Document production runbooks

---

# Deployment Strategy Anti-Patterns

## Deploying Without Rollback

Bad:

    Deploy
      |
      ↓
    Hope

Better:

    Deploy
      |
      ↓
    Monitor
      |
      ↓
    Rollback If Required

---

# Deployment Strategy Anti-Pattern

## Using latest Tag

Bad:

    app:latest

Better:

    app:1.4.7

or:

    app:<commit-sha>

---

# Deployment Strategy Anti-Pattern

## No Health Checks

Bad:

    Deploy
      |
      ↓
    Immediately Route Traffic

Better:

    Deploy
      |
      ↓
    Health Check
      |
      ↓
    Validate
      |
      ↓
    Route Traffic

---

# Deployment Strategy Anti-Pattern

## Incompatible Database Changes

Bad:

    Destructive Migration
        |
        ↓
    New Version
        |
        ↓
    Rollback
        |
        ↓
    Old Version Cannot Work

Better:

    Expand
        |
        ↓
    Deploy
        |
        ↓
    Validate
        |
        ↓
    Contract

---

# Deployment Strategy Anti-Pattern

## No Monitoring

Bad:

    Deployment Complete
        |
        ↓
    Assume Success

Better:

    Deployment Complete
        |
        ↓
    Monitor
        |
        ↓
    Validate
        |
        ↓
    Confirm Stability

---

# Final Deployment Strategy Mental Model

Remember:

    RECREATE

    Stop
      |
      ↓
    Replace
      |
      ↓
    Start


    ROLLING

    Old
      |
      ↓
    Replace Gradually
      |
      ↓
    New


    BLUE-GREEN

    Blue
      |
      ↓
    Production

    Green
      |
      ↓
    Validate

    Switch Traffic


    CANARY

    Stable
      |
      ↓
    95%
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
    Increase
      |
      ↓
    100%


    SHADOW

    Production Traffic
      |
      +--------→ Stable → User
      |
      └--------→ Shadow → Analyze


    A/B

    Users
      |
      +--------→ Version A
      |
      └--------→ Version B

---

# Final Concept

Deployment strategies define how application changes reach production safely.

The major strategies are:

    Recreate
        |
        ↓
    Stop Old → Start New


    Rolling
        |
        ↓
    Replace Instances Gradually


    Blue-Green
        |
        ↓
    Two Environments → Switch Traffic


    Canary
        |
        ↓
    Small Traffic → Gradually Increase


    Shadow
        |
        ↓
    Copy Traffic → Analyze New Version


    A/B
        |
        ↓
    Compare User Experiences


    Feature Flags
        |
        ↓
    Deploy Code → Activate Feature Gradually


A production-ready DevOps deployment strategy combines:

    CI/CD
        +
    Automated Testing
        +
    Security Scanning
        +
    Immutable Images
        +
    Kubernetes / EKS
        +
    Helm
        +
    ArgoCD
        +
    Health Checks
        +
    Prometheus
        +
    Grafana
        +
    ELK
        +
    Rollback
        +
    Database Compatibility
        +
    API Compatibility
        +
    Monitoring

The most important rule is:

    Do Not Treat Every Deployment the Same.

Choose the strategy based on:

    Risk
    Availability
    Cost
    Traffic
    Rollback Requirements
    Application Architecture
    Database Compatibility
    Monitoring Capability

A simple decision model is:

    Low Complexity
        |
        ↓
    Rolling

    Fast Rollback
        |
        ↓
    Blue-Green

    Small Blast Radius
        |
        ↓
    Canary

    No Version Coexistence
        |
        ↓
    Recreate

    Production Traffic Testing
        |
        ↓
    Shadow

    Business Experiment
        |
        ↓
    A/B

    Gradual Feature Activation
        |
        ↓
    Feature Flags

The ultimate goal of every deployment strategy is:

    Safe Release
        |
        ↓
    Minimal User Impact
        |
        ↓
    Continuous Validation
        |
        ↓
    Fast Recovery
        |
        ↓
    Reliable Production