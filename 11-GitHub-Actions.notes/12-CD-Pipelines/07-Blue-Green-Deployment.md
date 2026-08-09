# Blue-Green Deployment

Blue-Green Deployment is a deployment strategy where two separate application environments are maintained:

    Blue  = Current Production Version

    Green = New Version

Only one environment normally receives production traffic at a time.

The new version is deployed to the inactive environment, tested, and then traffic is switched to it.

If the new version fails, traffic can be switched back to the previous environment.

---

# Blue-Green Deployment Mental Model

Remember:

    Blue
    Version 1.4.6
        |
        ↓
    Production Traffic


    Green
    Version 1.4.7
        |
        ↓
    Testing


After validation:

    Traffic
        |
        ↓
    Green
    Version 1.4.7


If failure:

    Traffic
        |
        ↓
    Blue
    Version 1.4.6

---

# Why Blue-Green Deployment Is Used

Blue-Green Deployment helps reduce deployment risk.

Benefits include:

- Fast rollback
- Reduced downtime
- Easy traffic switching
- Production-like validation
- Safer releases
- Simple rollback process
- Ability to test the new version before switching traffic

---

# Blue-Green Deployment Architecture

Basic architecture:

    Users
      |
      ↓
    Load Balancer
      |
      +--------→ Blue
      |           |
      |           └── Version 1.4.6
      |
      └--------→ Green
                  |
                  └── Version 1.4.7

Initially:

    Traffic → Blue

After successful validation:

    Traffic → Green

---

# Blue Environment

Blue represents the currently active production environment.

Example:

    Blue
      |
      ↓
    Version 1.4.6
      |
      ↓
    Production

Users continue using Blue while Green is being prepared.

---

# Green Environment

Green represents the new application version.

Example:

    Green
      |
      ↓
    Version 1.4.7
      |
      ↓
    Testing

Green does not receive normal production traffic until it has been validated.

---

# Blue-Green Deployment Flow

The basic process is:

    1. Blue is serving production
    2. Deploy new version to Green
    3. Validate Green
    4. Switch traffic to Green
    5. Monitor Green
    6. Keep Blue available for rollback
    7. Decommission or retain Blue according to the strategy

---

# Complete Blue-Green Flow

    Blue
    v1.4.6
      |
      ↓
    Production Traffic


    Green
    v1.4.7
      |
      ↓
    Deploy
      |
      ↓
    Test
      |
      ↓
    Validate
      |
      ↓
    Traffic Switch
      |
      ↓
    Green
    v1.4.7
      |
      ↓
    Production

---

# Initial State

Before deployment:

    Users
      |
      ↓
    Load Balancer
      |
      ↓
    Blue
      |
      ↓
    Version 1.4.6

Green may be:

    Empty

or:

    Previous Version

or:

    Ready Environment

---

# Deploy New Version

The new version is deployed to Green.

Example:

    Blue:
      Version 1.4.6

    Green:
      Version 1.4.7

Traffic remains:

    Users
      |
      ↓
    Blue

Green can now be tested without affecting normal production traffic.

---

# Green Validation

Before switching traffic, validate:

    Application Startup
    Pod Health
    Readiness
    Liveness
    API Health
    Database Connectivity
    Service Connectivity
    Configuration
    Smoke Tests
    Security Checks
    Performance
    Logs
    Metrics

---

# Health Validation Flow

    Green
      |
      ↓
    Pods Ready?
      |
      ↓
    Services Healthy?
      |
      ↓
    Application Healthy?
      |
      ↓
    Smoke Tests
      |
      ↓
    Metrics Normal?
      |
      +------ No ------→ Do Not Switch
      |
      +------ Yes -----→ Switch Traffic

---

# Traffic Switching

Once Green is validated:

    Before:

    Users
      |
      ↓
    Load Balancer
      |
      ↓
    Blue
    v1.4.6


    After:

    Users
      |
      ↓
    Load Balancer
      |
      ↓
    Green
    v1.4.7

The traffic switch is the critical part of Blue-Green deployment.

---

# Traffic Switching Methods

Traffic can be switched using:

    Load Balancer
    Kubernetes Service
    Ingress
    DNS
    Service Mesh
    Target Groups

The implementation depends on the infrastructure.

---

# Blue-Green With Load Balancer

Example:

    Users
      |
      ↓
    ALB
      |
      +--------→ Blue Target Group
      |
      └--------→ Green Target Group

Initially:

    ALB
      |
      ↓
    Blue Target Group

After deployment:

    ALB
      |
      ↓
    Green Target Group

---

# Blue-Green With AWS ALB

A common AWS architecture is:

    Route 53
       |
       ↓
    ALB
       |
       ↓
    Listener
       |
       +--------→ Blue Target Group
       |
       └--------→ Green Target Group

The listener configuration determines which target group receives traffic.

---

# Blue-Green With EKS

Example:

    Users
      |
      ↓
    Route 53
      |
      ↓
    ALB
      |
      ↓
    Kubernetes Ingress
      |
      +--------→ Blue
      |            |
      |            └── Pods v1.4.6
      |
      └--------→ Green
                   |
                   └── Pods v1.4.7

Traffic can be directed to the selected environment.

---

# Blue-Green Using Kubernetes Services

One approach is to maintain separate workloads:

    payment-blue
    payment-green

Example:

    payment-blue
        |
        ↓
    Version 1.4.6


    payment-green
        |
        ↓
    Version 1.4.7

A Service or routing layer can direct traffic to the active version.

---

# Kubernetes Selector-Based Switching

Example:

    Service:

    selector:
      app: payment
      version: blue

Blue Pods:

    app: payment
    version: blue

Green Pods:

    app: payment
    version: green

To switch traffic:

    selector:
      app: payment
      version: green

Traffic moves from Blue to Green.

---

# Kubernetes Blue-Green Example

Blue Deployment:

    apiVersion: apps/v1
    kind: Deployment

    metadata:
      name: payment-blue

    spec:
      replicas: 3

      selector:
        matchLabels:
          app: payment
          version: blue

      template:
        metadata:
          labels:
            app: payment
            version: blue

        spec:
          containers:
            - name: payment
              image: payment:1.4.6

---

# Green Deployment

    apiVersion: apps/v1
    kind: Deployment

    metadata:
      name: payment-green

    spec:
      replicas: 3

      selector:
        matchLabels:
          app: payment
          version: green

      template:
        metadata:
          labels:
            app: payment
            version: green

        spec:
          containers:
            - name: payment
              image: payment:1.4.7

---

# Kubernetes Service

Initially:

    apiVersion: v1
    kind: Service

    metadata:
      name: payment

    spec:
      selector:
        app: payment
        version: blue

Traffic:

    Service
       |
       ↓
    Blue Pods
       |
       ↓
    Version 1.4.6

---

# Switch to Green

Change:

    version: blue

to:

    version: green

Now:

    Service
       |
       ↓
    Green Pods
       |
       ↓
    Version 1.4.7

This provides a simple Blue-Green traffic switch.

---

# Blue-Green With Helm

Helm can manage Blue-Green deployments.

Example:

    values-blue.yaml

    version: blue
    image:
      tag: "1.4.6"


    values-green.yaml

    version: green
    image:
      tag: "1.4.7"

Helm can deploy both versions with different configuration.

---

# Helm Blue-Green Structure

Example:

    helm/
      |
      ├── Chart.yaml
      ├── values.yaml
      └── templates/
          |
          ├── deployment.yaml
          ├── service.yaml
          └── ingress.yaml

Environment-specific values can control:

    Version
    Image
    Replicas
    Labels
    Routing

---

# Blue-Green With ArgoCD

ArgoCD can manage Blue-Green deployment configuration stored in Git.

Flow:

    Git
      |
      ↓
    Blue-Green Configuration
      |
      ↓
    ArgoCD
      |
      ↓
    EKS
      |
      +-- Blue
      |
      └-- Green

Traffic switching can then be controlled through the relevant Kubernetes or ingress configuration.

---

# GitOps Blue-Green Deployment

Example:

    GitOps Repository
        |
        ├── blue
        |    |
        |    └── deployment.yaml
        |
        └── green
             |
             └── deployment.yaml

ArgoCD synchronizes the desired state.

---

# GitOps Traffic Switch

Before:

    Service:
      version: blue

Git commit:

    Switch production traffic to green

After:

    Service:
      version: green

ArgoCD:

    Git
      |
      ↓
    Detect Change
      |
      ↓
    Sync
      |
      ↓
    Kubernetes
      |
      ↓
    Green Receives Traffic

---

# Blue-Green With GitHub Actions

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
       +-- Security Scan
       |
       ↓
    Docker Image
       |
       ↓
    ECR
       |
       ↓
    Deploy Green
       |
       ↓
    Validate Green
       |
       ↓
    Switch Traffic
       |
       ↓
    Monitor

---

# Blue-Green CI/CD Flow

    Code
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
    Green Deployment
      |
      ↓
    Validation
      |
      +------ Fail ------→ Keep Blue
      |
      +------ Pass ------→ Switch Traffic
                              |
                              ↓
                            Green
                              |
                              ↓
                          Monitoring

---

# Blue-Green With Jenkins

Jenkins can automate the process:

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
    Deploy Green
       |
       ↓
    Validate
       |
       ↓
    Switch Traffic
       |
       ↓
    Monitor

---

# Blue-Green and ECR

ECR stores both image versions.

Example:

    ECR
      |
      └── payment
           |
           ├── 1.4.6
           └── 1.4.7

Blue:

    payment:1.4.6

Green:

    payment:1.4.7

Both images can exist simultaneously.

---

# Blue-Green and Route 53

Route 53 can be part of a Blue-Green architecture.

Example:

    app.example.com
          |
          ↓
       Route 53
          |
          ↓
         ALB
          |
          +--------→ Blue
          |
          └--------→ Green

The traffic switch can happen at the load balancer or routing layer.

---

# Blue-Green and DNS Switching

DNS-based switching is possible, but DNS caching and TTL behavior can affect how quickly traffic moves.

Example:

    app.example.com
          |
          ↓
    DNS Record
          |
          ↓
    Blue

Switch:

    DNS Record
          |
          ↓
    Green

DNS-based Blue-Green requires careful consideration of caching and propagation behavior.

---

# Blue-Green With Separate Environments

Blue and Green can be separate:

    Kubernetes Namespaces

or:

    Kubernetes Deployments

or:

    EKS Clusters

or:

    AWS Infrastructure Environments

The choice depends on cost, isolation, complexity, and operational requirements.

---

# Blue-Green Using Namespaces

Example:

    EKS Cluster
       |
       +-- blue namespace
       |      |
       |      └── v1.4.6
       |
       └-- green namespace
              |
              └── v1.4.7

Traffic is directed to the selected namespace through the routing configuration.

---

# Blue-Green Using Separate Clusters

For stronger isolation:

    AWS
      |
      +-- Blue EKS
      |      |
      |      └── v1.4.6
      |
      └-- Green EKS
             |
             └── v1.4.7

Traffic:

    ALB / DNS
        |
        +--------→ Blue EKS
        |
        └--------→ Green EKS

This provides strong isolation but increases infrastructure cost.

---

# Blue-Green Using Separate Infrastructure

Another model:

    Blue Environment
        |
        +-- VPC
        +-- EKS
        +-- ALB
        +-- Application


    Green Environment
        |
        +-- VPC
        +-- EKS
        +-- ALB
        +-- Application

Traffic can switch between environments.

This provides strong isolation but is more expensive.

---

# Blue-Green Cost Consideration

Blue-Green requires two environments during the transition.

Example:

    Blue
    v1.4.6
      +
    Green
    v1.4.7

Therefore resource usage can increase.

Potential additional costs:

    Compute
    Load Balancers
    Storage
    Database Resources
    Networking
    Monitoring

A team should balance rollback speed against infrastructure cost.

---

# Blue-Green Database Strategy

Database design is one of the most important considerations.

Example:

    Blue Application
          |
          ↓
        Database
          ↑
          |
    Green Application

Both versions may access the same database.

Therefore database changes should be backward compatible whenever possible.

---

# Shared Database Risk

Suppose:

    Blue → Schema A

    Green → Schema B

If Green changes the schema incompatibly:

    Blue
      |
      ↓
    Schema B
      |
      ↓
    Blue May Fail

Therefore Blue-Green deployment must consider database compatibility.

---

# Backward-Compatible Database Migration

Safer approach:

    Current Application
         |
         ↓
    Add Compatible Schema
         |
         ↓
    Deploy Green
         |
         ↓
    Validate
         |
         ↓
    Switch Traffic
         |
         ↓
    Remove Old Schema Later

This reduces rollback risk.

---

# Blue-Green Database Example

Suppose version 1.4.7 needs a new column.

Instead of immediately removing the old column:

    Step 1:
    Add new column

    Step 2:
    Keep old column

    Step 3:
    Deploy Green

    Step 4:
    Validate Green

    Step 5:
    Switch traffic

    Step 6:
    Remove old column later

This allows the old Blue version to continue working if rollback is required.

---

# Blue-Green Rollback

Rollback is one of the strongest advantages of Blue-Green deployment.

After switch:

    Users
      |
      ↓
    Green
    v1.4.7

Problem:

    High Error Rate

Rollback:

    Users
      |
      ↓
    Blue
    v1.4.6

The old environment remains available.

---

# Fast Rollback

Blue-Green rollback can be very fast because there is no need to rebuild the previous application.

The previous environment is already deployed.

Flow:

    Green
    v1.4.7
       |
       ↓
    Failure
       |
       ↓
    Traffic Switch
       |
       ↓
    Blue
    v1.4.6

---

# Blue-Green Rollback Flow

    Green
    v1.4.7
       |
       ↓
    Production
       |
       ↓
    Problem Detected
       |
       ↓
    Switch Traffic
       |
       ↓
    Blue
    v1.4.6
       |
       ↓
    Validate
       |
       ↓
    Recovery

---

# Blue-Green Validation Before Switch

Before switching traffic, verify:

    Pods Ready
    Deployment Available
    Service Available
    Ingress Healthy
    ALB Target Healthy
    Application Health
    Database Connectivity
    Dependency Connectivity
    Smoke Tests
    Error Rate
    Latency
    Logs

Only switch traffic after the required checks pass.

---

# Blue-Green Validation After Switch

After switching traffic:

    Monitor:

    Error Rate
    HTTP 5xx
    Latency
    CPU
    Memory
    Pod Restarts
    Request Rate
    Application Logs
    Database Health
    ALB Target Health

If metrics remain healthy:

    Green becomes Production

If metrics degrade:

    Rollback to Blue

---

# Blue-Green Monitoring Flow

    Switch Traffic
        |
        ↓
    Green Production
        |
        ↓
    Monitor
        |
        +------ Healthy ------→ Continue
        |
        +------ Unhealthy ----→ Rollback
                                  |
                                  ↓
                                Blue

---

# Blue-Green Deployment and Health Checks

Health checks should operate at multiple levels.

Infrastructure:

    ALB Health Check

Kubernetes:

    Readiness Probe
    Liveness Probe

Application:

    /health
    /ready

Business:

    Critical API
    Smoke Test

---

# Readiness Probe

Readiness determines whether the new Pods should receive traffic.

Example:

    Green Pod
       |
       ↓
    Readiness Probe
       |
       +------ Fail ------→ No Traffic
       |
       +------ Pass ------→ Ready

This prevents traffic from reaching unready Pods.

---

# Liveness Probe

Liveness checks whether the application container remains operational.

Example:

    Green Pod
       |
       ↓
    Liveness Probe
       |
       +------ Healthy ------→ Continue
       |
       └------ Failure ------→ Restart

---

# Blue-Green Smoke Testing

After Green deployment:

    Green
      |
      ↓
    Smoke Test
      |
      +-- Login
      +-- Health Endpoint
      +-- Critical API
      +-- Database Connectivity
      +-- Service Connectivity

If smoke tests fail:

    Do Not Switch Traffic

---

# Blue-Green Security Validation

Before switching Green to production:

    Container Scan
    Dependency Scan
    Configuration Review
    Kubernetes Security
    IAM Review
    Network Policy
    Secret Validation

Security validation should be integrated into the deployment pipeline.

---

# Blue-Green Deployment With DevSecOps

Complete flow:

    Developer
       |
       ↓
    GitHub
       |
       ↓
    CI Pipeline
       |
       +-- Build
       +-- Unit Test
       +-- SonarQube
       +-- Trivy
       +-- Other Security Checks
       |
       ↓
    ECR
       |
       ↓
    Deploy Green
       |
       ↓
    Validation
       |
       ↓
    Traffic Switch
       |
       ↓
    Production
       |
       ↓
    Monitoring

---

# Blue-Green With ArgoCD and EKS

Architecture:

    GitHub
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
       +-- Blue Deployment
       |      |
       |      └── v1.4.6
       |
       └-- Green Deployment
              |
              └── v1.4.7
                     |
                     ↓
                 Validation
                     |
                     ↓
                Traffic Switch

---

# Blue-Green GitOps Repository

Example:

    gitops/
      |
      ├── applications/
      |      |
      |      └── payment/
      |
      ├── blue/
      |      |
      |      └── payment/
      |
      ├── green/
      |      |
      |      └── payment/
      |
      └── routing/
             |
             └── payment-service.yaml

The exact structure depends on the organization's GitOps design.

---

# Blue-Green Image Versioning

Example:

    Blue:

    payment:1.4.6


    Green:

    payment:1.4.7

After successful deployment:

    Green becomes Production

Later:

    Blue can be updated to the next release

---

# Reusing Blue After Deployment

After Green becomes production:

    Green
    v1.4.7
       |
       ↓
    Production

Blue:

    v1.4.6

Blue can eventually be:

    Decommissioned

or:

    Updated to the next version

depending on the deployment strategy.

---

# Blue-Green Continuous Deployment

Example release cycle:

    Release 1:
        Blue = v1.4.6
        Green = v1.4.7

    Switch:
        Green = Production

    Next Release:
        Blue = v1.4.8
        Green = v1.4.7

    Switch:
        Blue = Production

The active and inactive roles can alternate.

---

# Blue-Green Role Switching

Initial:

    Blue = Production
    Green = New Version

After switch:

    Green = Production
    Blue = Previous Version

Next deployment:

    Blue = New Version
    Green = Current Production

Then:

    Blue = Production
    Green = Previous Version

This alternating model gives the strategy its name.

---

# Blue-Green vs Rolling Deployment

Blue-Green:

    Two environments
        |
        ↓
    Switch Traffic

Rolling:

    Gradually replace Pods
        |
        ↓
    Old + New Pods
        |
        ↓
    Complete Replacement

Blue-Green provides a simpler full-environment rollback.

Rolling deployments generally use fewer additional resources.

---

# Blue-Green vs Canary Deployment

Blue-Green:

    100% Traffic → Blue

Then:

    100% Traffic → Green

Canary:

    95% → Blue
     5% → Green

Then:

    50% → Blue
    50% → Green

Then:

    0% → Blue
    100% → Green

Canary provides gradual traffic exposure.

---

# Blue-Green vs Recreate

Recreate:

    Stop Old Version
        |
        ↓
    Deploy New Version

This can cause downtime.

Blue-Green:

    Old Version
        |
        ↓
    Keep Running
        |
        ↓
    Deploy New Version
        |
        ↓
    Switch Traffic

Blue-Green can provide near-zero downtime when implemented correctly.

---

# Blue-Green vs Rolling

| Feature | Blue-Green | Rolling |
|---|---|---|
| Two environments | Yes | No |
| Fast rollback | Yes | Yes |
| Additional resources | Higher | Lower |
| Traffic switch | Full switch | Gradual replacement |
| Testing new version before traffic | Yes | Limited |
| Simple rollback | Very easy | Easy |
| Infrastructure cost | Higher | Lower |

---

# Blue-Green Advantages

## Fast Rollback

Previous environment remains available.

## Reduced Downtime

Traffic can switch between ready environments.

## Easy Validation

Green can be tested before production traffic is switched.

## Simple Traffic Control

Traffic can be moved between Blue and Green.

## Clear Version Separation

Each environment represents a specific version.

---

# Blue-Green Disadvantages

## Higher Infrastructure Cost

Two environments may need to run simultaneously.

## Database Complexity

Both versions may need to work with the same database.

## Configuration Complexity

Configuration must be compatible between environments.

## Traffic Switching Complexity

The routing layer must support controlled switching.

## Resource Requirements

Two complete environments may require significant compute and networking resources.

---

# When to Use Blue-Green

Blue-Green is useful when:

    Fast Rollback Is Important
    Downtime Must Be Minimized
    Production Validation Is Required
    Infrastructure Cost Is Acceptable
    Traffic Switching Is Available
    Application Versions Can Coexist

---

# When Not to Use Blue-Green

Blue-Green may not be ideal when:

    Infrastructure Cost Is Highly Restricted
    Database Changes Are Not Backward Compatible
    Two Environments Are Too Expensive
    Application Versions Cannot Coexist
    Traffic Switching Is Difficult

In such cases, other deployment strategies may be more appropriate.

---

# Blue-Green Production Strategy

A production Blue-Green process can be:

    1. Identify current Blue version
    2. Prepare Green environment
    3. Deploy new application version
    4. Configure Green
    5. Run health checks
    6. Run smoke tests
    7. Validate dependencies
    8. Validate database compatibility
    9. Validate monitoring
    10. Switch production traffic
    11. Monitor Green
    12. Keep Blue available
    13. Rollback if required
    14. Decommission Blue after confidence is established

---

# Blue-Green Deployment Runbook

## Step 1: Check Current Production

    Blue
    v1.4.6
       |
       ↓
    Healthy

Verify:

    Pods
    Services
    Ingress
    ALB
    Metrics
    Logs

---

# Step 2: Deploy Green

    Green
    v1.4.7

Deploy:

    Application
    ConfigMap
    Secret References
    Services
    Ingress Configuration

---

# Step 3: Validate Green

Check:

    kubectl get pods

    kubectl get deployments

    kubectl get services

    kubectl get events

Then run:

    Health Checks
    Smoke Tests
    Application Tests

---

# Step 4: Switch Traffic

Before:

    Users
      |
      ↓
    Blue v1.4.6

After:

    Users
      |
      ↓
    Green v1.4.7

---

# Step 5: Monitor

Monitor:

    Error Rate
    Latency
    CPU
    Memory
    Pod Restarts
    Logs
    ALB Health
    Application Health

---

# Step 6: Rollback If Required

If Green fails:

    Green v1.4.7
        |
        ↓
    Failure
        |
        ↓
    Traffic Switch
        |
        ↓
    Blue v1.4.6

Then:

    Validate
    Monitor
    Investigate

---

# Blue-Green Scenario

## Scenario: New Version Has 503 Errors

Current:

    Blue
    v1.4.6
    Healthy

Deploy:

    Green
    v1.4.7

After traffic switch:

    HTTP 503
    Error Rate Increased

Action:

    Stop Further Changes
        |
        ↓
    Switch Traffic to Blue
        |
        ↓
    Validate Blue
        |
        ↓
    Confirm Recovery
        |
        ↓
    Investigate Green

---

# Blue-Green Scenario

## Scenario: Green Pods Are Unhealthy

Green:

    Pods
      |
      ↓
    CrashLoopBackOff

Action:

    Do Not Switch Traffic

Investigate:

    kubectl logs <pod>

    kubectl describe pod <pod>

If Blue is healthy:

    Keep Traffic → Blue

Fix Green before attempting another deployment.

---

# Blue-Green Scenario

## Scenario: Green Health Check Fails

Green:

    ALB Target
        |
        ↓
    Unhealthy

Check:

    Health Check Path
    Port
    Service
    Target Group
    Readiness Probe
    Security Group
    Application

Do not switch production traffic until the problem is resolved.

---

# Blue-Green Scenario

## Scenario: Database Migration Causes Failure

Green requires:

    Schema B

Blue requires:

    Schema A

If the migration is incompatible:

    Blue
      |
      ↓
    Cannot Work With Schema B

Rollback may not be safe.

Solution:

    Use Backward-Compatible Database Migration

Database compatibility must be planned before Blue-Green deployment.

---

# Blue-Green Scenario

## Scenario: Green Has High Latency

After switching traffic:

    Green
      |
      ↓
    High Latency

Check:

    CPU
    Memory
    Database
    External Dependencies
    Application Logs
    Network
    Connection Pools

If Green is confirmed as the cause:

    Switch Traffic → Blue

Then investigate Green.

---

# Blue-Green Scenario

## Scenario: Security Vulnerability in Green

Green:

    v1.4.7

Security issue discovered.

Action:

    Switch Traffic → Blue
        |
        ↓
    v1.4.6

Then:

    Fix Vulnerability
        |
        ↓
    Build v1.4.8
        |
        ↓
    Security Scan
        |
        ↓
    Deploy Green
        |
        ↓
    Validate
        |
        ↓
    Switch Traffic

---

# Blue-Green Best Practices

- Keep Blue healthy until Green is fully validated
- Use immutable image tags
- Validate Green before traffic switching
- Use readiness probes
- Use liveness probes appropriately
- Run smoke tests
- Monitor application health
- Monitor ALB target health
- Keep previous version available for rollback
- Use backward-compatible database migrations
- Use GitOps where appropriate
- Use automated deployment validation
- Secure traffic switching
- Document rollback procedures
- Avoid unnecessary manual changes
- Protect production access
- Monitor after traffic switching
- Test rollback procedures regularly

---

# Blue-Green Security Best Practices

- Use least-privilege IAM
- Secure ECR
- Scan images
- Use immutable image tags
- Protect Git repositories
- Use RBAC
- Secure Kubernetes Secrets
- Use HTTPS
- Use ACM certificates
- Restrict security groups
- Protect production routing configuration
- Audit deployment changes
- Secure CI/CD credentials
- Use OIDC for GitHub Actions AWS access where appropriate

---

# Blue-Green Anti-Patterns

## Switching Traffic Without Validation

Bad:

    Deploy Green
        |
        ↓
    Immediately Switch Traffic

Better:

    Deploy Green
        |
        ↓
    Health Checks
        |
        ↓
    Smoke Tests
        |
        ↓
    Validate
        |
        ↓
    Switch Traffic

---

# Blue-Green Anti-Pattern

## Removing Blue Immediately

Bad:

    Green Production
        |
        ↓
    Delete Blue Immediately

If Green fails:

    No Fast Rollback

Better:

    Green Production
        |
        ↓
    Monitor
        |
        ↓
    Keep Blue
        |
        ↓
    Confirm Stability
        |
        ↓
    Decommission Blue

---

# Blue-Green Anti-Pattern

## Incompatible Database Changes

Bad:

    Deploy Green
        |
        ↓
    Destructive Database Migration
        |
        ↓
    Green Failure
        |
        ↓
    Blue Rollback
        |
        ↓
    Blue Cannot Work

Better:

    Backward-Compatible Migration
        |
        ↓
    Green
        |
        ↓
    Validate
        |
        ↓
    Switch
        |
        ↓
    Cleanup Later

---

# Blue-Green Anti-Pattern

## Mutable Docker Tags

Bad:

    app:latest

Better:

    app:1.4.7

or:

    app:<commit-sha>

Immutable versions make Blue and Green clearly identifiable.

---

# Blue-Green Anti-Pattern

## No Monitoring After Switch

Bad:

    Switch Traffic
        |
        ↓
    Assume Success

Better:

    Switch Traffic
        |
        ↓
    Monitor
        |
        +-- Healthy → Continue
        |
        └-- Unhealthy → Rollback

---

# Blue-Green Interview Questions

## Basic Questions

1. What is Blue-Green Deployment?
2. Why is it called Blue-Green?
3. What is the Blue environment?
4. What is the Green environment?
5. How does traffic switching work?
6. What are the advantages of Blue-Green deployment?
7. What are the disadvantages?
8. How does Blue-Green support rollback?
9. What is the difference between Blue-Green and Rolling deployment?
10. What is the difference between Blue-Green and Canary deployment?

---

# Blue-Green Interview Questions

## Intermediate Questions

11. How would you implement Blue-Green deployment on Kubernetes?

12. How would you implement Blue-Green deployment on EKS?

13. How would you switch traffic between Blue and Green?

14. How would you use an ALB for Blue-Green deployment?

15. How would you implement Blue-Green using Kubernetes Services?

16. How would you deploy Blue-Green using Helm?

17. How would you implement Blue-Green with ArgoCD?

18. How would you validate Green before switching traffic?

19. How would you rollback after switching to Green?

20. How would you handle database migrations?

---

# Blue-Green Interview Questions

## Advanced Questions

21. Design a production Blue-Green architecture for EKS.

22. How would you implement Blue-Green with GitHub Actions and ArgoCD?

23. How would you implement Blue-Green for microservices?

24. How would you handle shared databases?

25. How would you handle backward-compatible database migrations?

26. How would you automate traffic switching?

27. How would you automate rollback?

28. How would you monitor the Green environment?

29. How would you prevent an unhealthy Green deployment from receiving traffic?

30. How would you reduce the cost of Blue-Green infrastructure?

31. How would you implement Blue-Green across multiple environments?

32. How would you design zero-downtime Blue-Green deployment?

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

Current production:

    Blue
      |
      ├── User v1.4.6
      ├── Product v1.4.6
      ├── Cart v1.4.6
      ├── Orders v1.4.6
      ├── Payment v1.4.6
      ├── Inventory v1.4.6
      └── Notification v1.4.6

New environment:

    Green
      |
      ├── User v1.4.7
      ├── Product v1.4.7
      ├── Cart v1.4.7
      ├── Orders v1.4.7
      ├── Payment v1.4.7
      ├── Inventory v1.4.7
      └── Notification v1.4.7

Traffic initially:

    Users
      |
      ↓
    ALB
      |
      ↓
    Blue

After validation:

    Users
      |
      ↓
    ALB
      |
      ↓
    Green

---

# Real-World CI/CD Flow

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
        +-- Trivy
        |
        ↓
    Docker Image
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
        |
        +-- Blue
        |    |
        |    └── Current Production
        |
        └-- Green
             |
             └── New Version
                    |
                    ↓
                 Validation
                    |
                    ↓
               Traffic Switch
                    |
                    ↓
                 Production

---

# Real-World AWS Blue-Green Architecture

    Internet
       |
       ↓
    Route 53
       |
       ↓
    ALB
       |
       +-------------------+
       |                   |
       ↓                   ↓
    Blue Target        Green Target
       |                   |
       ↓                   ↓
    EKS Blue            EKS Green
       |                   |
       ↓                   ↓
    v1.4.6               v1.4.7

Infrastructure:

    Terraform
       |
       ↓
    AWS
       |
       +-- VPC
       +-- Subnets
       +-- Security Groups
       +-- EKS
       +-- ECR
       +-- ALB
       +-- IAM

Deployment:

    GitHub Actions
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

---

# Complete Blue-Green DevOps Architecture

    Developer
        |
        ↓
    Application Repository
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
        +---------------------------+
        |                           |
        ↓                           ↓
      BLUE                        GREEN
    v1.4.6                       v1.4.7
        |                           |
        |                       Validation
        |                           |
        +-------------+-------------+
                      |
                      ↓
                   ALB
                      |
                      ↓
                   Route 53
                      |
                      ↓
                    Users

Traffic initially:

    Users → ALB → Blue

After validation:

    Users → ALB → Green

If Green fails:

    Users → ALB → Blue

---

# Blue-Green Deployment Decision Flow

    New Version Ready
          |
          ↓
    Deploy Green
          |
          ↓
    Green Healthy?
          |
          +------ No ------→ Fix Green
          |
          +------ Yes -----→ Run Tests
                                |
                                ↓
                           Tests Pass?
                                |
                                +-- No → Fix Green
                                |
                                +-- Yes
                                      |
                                      ↓
                               Switch Traffic
                                      |
                                      ↓
                                  Monitor
                                      |
                                      ↓
                               Production Healthy?
                                      |
                                      +-- Yes → Keep Green
                                      |
                                      +-- No → Switch to Blue
                                                   |
                                                   ↓
                                                Recover

---

# Final Blue-Green Mental Model

Remember:

    BLUE
    Current Production
    Version 1.4.6
        |
        | Existing Traffic
        ↓
      Users


    GREEN
    New Version
    Version 1.4.7
        |
        ↓
    Deploy
        |
        ↓
    Test
        |
        ↓
    Validate


After validation:

    Users
      |
      ↓
    GREEN
    Version 1.4.7


If failure:

    Users
      |
      ↓
    BLUE
    Version 1.4.6

The key principle is:

    Deploy New Version
          |
          ↓
    Validate Without Production Traffic
          |
          ↓
    Switch Traffic
          |
          ↓
    Monitor
          |
          ↓
    Rollback Quickly If Required

---

# Final Concept

Blue-Green Deployment provides two application environments:

    Blue = Current Production

    Green = New Release

The complete process is:

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
    Docker Image
      |
      ↓
    ECR
      |
      ↓
    Deploy Green
      |
      ↓
    Health Checks
      |
      ↓
    Smoke Tests
      |
      ↓
    Traffic Switch
      |
      ↓
    Green Production
      |
      ↓
    Monitoring
      |
      +------ Healthy ------→ Continue
      |
      +------ Unhealthy ----→ Blue
                               |
                               ↓
                            Rollback

The biggest advantage of Blue-Green deployment is the ability to deploy and validate a new version before exposing it to production traffic while keeping the previous version available for fast rollback.

A production-ready Blue-Green strategy should combine:

    Immutable Images
        +
    Automated CI/CD
        +
    Kubernetes / EKS
        +
    ALB / Ingress
        +
    Health Checks
        +
    Smoke Tests
        +
    GitOps / ArgoCD
        +
    Monitoring
        +
    Backward-Compatible Database Changes
        +
    Fast Rollback

This makes Blue-Green Deployment a powerful strategy for reducing production deployment risk and minimizing downtime.