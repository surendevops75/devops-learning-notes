# Health Checks

Health checks are mechanisms used to determine whether an application, container, Pod, service, or infrastructure component is healthy and ready to perform its intended function.

In DevOps and Kubernetes, health checks are critical for:

    Availability
    Reliability
    Zero-Downtime Deployments
    Rolling Deployments
    Canary Deployments
    Blue-Green Deployments
    Load Balancing
    Auto-Healing
    Post-Deployment Validation

The three primary Kubernetes health checks are:

    Startup Probe
    Readiness Probe
    Liveness Probe

Other important health checks include:

    Application Health Checks
    Load Balancer Health Checks
    Dependency Health Checks
    Smoke Tests
    Synthetic Checks

---

# Health Check Mental Model

    Application Starts
        |
        ↓
    Startup Probe
        |
        ↓
    Application Initialized
        |
        ↓
    Readiness Probe
        |
        +------ Fail ------→ No Traffic
        |
        +------ Pass
                |
                ↓
              Traffic
                |
                ↓
          Liveness Probe
                |
                +------ Pass ------→ Continue
                |
                +------ Fail ------→ Restart

---

# Why Health Checks Are Important

Without health checks:

    Application Failure
        |
        ↓
    Traffic Continues
        |
        ↓
    User Errors
        |
        ↓
    Manual Investigation

With health checks:

    Application Failure
        |
        ↓
    Health Check Detects Problem
        |
        ↓
    Remove From Traffic / Restart
        |
        ↓
    Recover
        |
        ↓
    Healthy Application

Health checks help Kubernetes and load balancers determine which application instances can safely serve traffic.

---

# Health Checks in DevOps

Health checks are used throughout the deployment lifecycle.

    Build
      |
      ↓
    Test
      |
      ↓
    Security Scan
      |
      ↓
    Deploy
      |
      ↓
    Health Check
      |
      ↓
    Smoke Test
      |
      ↓
    Validation
      |
      ↓
    Production Traffic

A deployment should not be considered successful simply because a container has started.

The application should also be healthy and ready to serve requests.

---

# Kubernetes Probes

Kubernetes provides three main probes:

    1. Startup Probe
    2. Readiness Probe
    3. Liveness Probe

They answer three different questions.

Startup Probe:

    "Has the application started?"

Readiness Probe:

    "Can the application receive traffic?"

Liveness Probe:

    "Is the application still functioning?"

---

# Startup Probe

A Startup Probe determines whether a slow-starting application has completed its initialization.

It is useful for applications such as:

    Large Java Applications
    Applications With Long Initialization
    Applications Performing Startup Migrations
    Applications Loading Large Configuration
    Applications Initializing Caches

Example:

    Container Starts
        |
        ↓
    Startup Probe
        |
        +------ Fail ------→ Keep Checking
        |
        +------ Pass ------→ Startup Complete
                                |
                                ↓
                         Readiness / Liveness

---

# Why Startup Probe Is Needed

Suppose an application requires three minutes to start.

Without a startup probe:

    Container Starts
        |
        ↓
    Liveness Probe
        |
        ↓
    Application Not Ready Yet
        |
        ↓
    Probe Fails
        |
        ↓
    Container Restart
        |
        ↓
    Application Starts Again
        |
        ↓
    Probe Fails Again

This can create:

    CrashLoopBackOff
    Restart Loops
    Deployment Failures

A startup probe allows the application enough time to initialize before liveness and readiness checks take over.

---

# Startup Probe Example

    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 30
      periodSeconds: 10

Conceptually:

    periodSeconds = 10
    failureThreshold = 30

The application can have approximately 300 seconds of startup-checking time before the startup probe is considered failed.

The exact configuration should be based on actual application startup behavior.

---

# Readiness Probe

A Readiness Probe determines whether a Pod is ready to receive traffic.

Question:

    "Can this Pod safely receive requests?"

If readiness succeeds:

    Pod = Ready

If readiness fails:

    Pod = Not Ready

The container can continue running while the Pod is removed from ready service endpoints.

---

# Readiness Mental Model

    Pod Running
        |
        ↓
    Readiness Probe
        |
        +------ Fail ------→ No Traffic
        |
        +------ Pass ------→ Receive Traffic

Important:

    Readiness Failure
        ≠
    Container Restart

Readiness primarily controls whether the Pod should receive traffic.

---

# Readiness Probe Example

    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      periodSeconds: 5
      timeoutSeconds: 2
      failureThreshold: 3

Kubernetes checks the endpoint periodically.

If the endpoint responds successfully:

    Pod = Ready

If the endpoint repeatedly fails:

    Pod = Not Ready

---

# Readiness Probe Use Cases

Readiness is useful when:

    Application Is Starting
    Database Connection Is Not Ready
    Required Dependency Is Unavailable
    Application Is Temporarily Unable To Serve Traffic
    Pod Is Being Drained
    Deployment Is In Progress
    Application Is Recovering

---

# Readiness During Rolling Deployment

Suppose:

    Old Version:
        v1

    New Version:
        v2

The new Pod starts.

    New Pod
        |
        ↓
    Startup
        |
        ↓
    Readiness Probe
        |
        ↓
    Not Ready
        |
        ↓
    No Production Traffic

Once ready:

    New Pod
        |
        ↓
    Ready
        |
        ↓
    Traffic Allowed

Then the old Pod can be removed according to the Deployment strategy.

---

# Readiness and Zero-Downtime

A common deployment flow is:

    Old Pod
        |
        ↓
    Serving Traffic

    New Pod
        |
        ↓
    Starting
        |
        ↓
    Not Ready
        |
        ↓
    No Traffic

    New Pod
        |
        ↓
    Ready
        |
        ↓
    Traffic Allowed

    Old Pod
        |
        ↓
    Terminated

Health checks help prevent traffic from reaching a new application instance before it is ready.

---

# Liveness Probe

A Liveness Probe determines whether a container is still functioning.

Question:

    "Is this application still alive?"

If the liveness probe repeatedly fails:

    Kubernetes can restart the container.

---

# Liveness Mental Model

    Container Running
        |
        ↓
    Liveness Probe
        |
        +------ Pass ------→ Continue
        |
        +------ Fail ------→ Restart Container

---

# Liveness Probe Example

    livenessProbe:
      httpGet:
        path: /live
        port: 8080
      periodSeconds: 10
      timeoutSeconds: 2
      failureThreshold: 3

If the application becomes unresponsive and repeatedly fails the liveness probe:

    Liveness Failure
        |
        ↓
    Container Restart
        |
        ↓
    Application Recovery

---

# Readiness vs Liveness

This is one of the most important Kubernetes interview topics.

Readiness asks:

    "Should this Pod receive traffic?"

Liveness asks:

    "Should this container continue running?"

Therefore:

    Readiness Failure
        |
        ↓
    Remove From Traffic

    Liveness Failure
        |
        ↓
    Restart Container

---

# Startup vs Readiness vs Liveness

| Probe | Main Question | Main Purpose |
|---|---|---|
| Startup | Has the application started? | Protect slow startup |
| Readiness | Can the application receive traffic? | Control traffic |
| Liveness | Is the application functioning? | Container recovery |

---

# Probe Failure Comparison

Startup failure:

    Application has not successfully started
        |
        ↓
    Continue checking
        |
        ↓
    Eventually restart if startup cannot succeed

Readiness failure:

    Application should not receive traffic
        |
        ↓
    Remove from ready endpoints

Liveness failure:

    Application is considered unhealthy
        |
        ↓
    Restart container

---

# Probe Types

Kubernetes supports different probe mechanisms.

Common types:

    HTTP
    TCP
    Exec
    gRPC

Choose the mechanism that best represents the application's health.

---

# HTTP Probe

HTTP probes make an HTTP request to a configured endpoint.

Example:

    GET /health

Expected response:

    HTTP 200

Conceptually:

    Kubernetes
        |
        ↓
    HTTP Request
        |
        ↓
    /health
        |
        +------ Success ------→ Healthy
        |
        +------ Failure ------→ Unhealthy

HTTP probes are commonly used for:

    REST APIs
    Web Applications
    Java Applications
    Node.js Applications
    Python Applications
    Microservices

---

# HTTP Probe Parameters

Important parameters include:

    path
    port
    scheme
    host
    httpHeaders
    initialDelaySeconds
    periodSeconds
    timeoutSeconds
    successThreshold
    failureThreshold

---

# initialDelaySeconds

Defines how long Kubernetes waits before starting the probe.

Example:

    initialDelaySeconds: 20

Conceptually:

    Container Starts
        |
        ↓
    Wait 20 Seconds
        |
        ↓
    Start Probe

For applications with unpredictable startup time, startupProbe is usually a better solution than simply setting a very large initial delay.

---

# periodSeconds

Defines how frequently the probe runs.

Example:

    periodSeconds: 10

Conceptually:

    Check
      |
      ↓
    Wait
      |
      ↓
    Check
      |
      ↓
    Wait
      |
      ↓
    Check

---

# timeoutSeconds

Defines how long Kubernetes waits for a probe response.

Example:

    timeoutSeconds: 2

If the application does not respond within the configured timeout, the probe attempt is considered failed.

---

# failureThreshold

Defines how many consecutive failures are required before a probe is considered failed.

Example:

    failureThreshold: 3

Conceptually:

    Failure
        |
        ↓
    Failure
        |
        ↓
    Failure
        |
        ↓
    Probe Considered Failed

---

# successThreshold

Defines how many consecutive successful probes are required before a failed probe is considered successful again.

Example:

    successThreshold: 1

For many HTTP probes, the default value is sufficient.

---

# TCP Probe

TCP probes check whether a TCP connection can be established.

Example:

    readinessProbe:
      tcpSocket:
        port: 8080
      periodSeconds: 10

Conceptually:

    Kubernetes
        |
        ↓
    TCP Connection
        |
        +------ Success ------→ Healthy
        |
        +------ Failure ------→ Unhealthy

TCP probes verify network-level availability.

They do not necessarily prove that the application is functioning correctly.

---

# Exec Probe

Exec probes execute a command inside the container.

Example:

    livenessProbe:
      exec:
        command:
          - /bin/sh
          - -c
          - test -f /tmp/healthy

If the command exits successfully:

    Probe = Success

If the command exits with a failure:

    Probe = Failure

Exec probes can be useful for custom local checks.

---

# Exec Probe Use Cases

Exec probes can be useful when:

    Application Has No HTTP Endpoint
    Custom Local Check Is Required
    Process State Must Be Verified
    Local File or Process State Represents Health

Avoid unnecessarily complicated commands because they can consume resources and make troubleshooting harder.

---

# gRPC Health Check

For gRPC applications, gRPC health checking can be used where supported.

Conceptually:

    Kubernetes
        |
        ↓
    gRPC Health Check
        |
        +------ Healthy ------→ Healthy
        |
        +------ Unhealthy ----→ Unhealthy

This is useful for applications built around gRPC.

---

# HTTP vs TCP vs Exec vs gRPC

| Probe | What It Checks | Typical Use |
|---|---|---|
| HTTP | HTTP Endpoint | REST APIs and Web Applications |
| TCP | TCP Connection | Network Services |
| Exec | Local Command | Custom Local Checks |
| gRPC | gRPC Health Service | gRPC Applications |

---

# Health Endpoint

A simple health endpoint might be:

    GET /health

Response:

    {
      "status": "UP"
    }

The endpoint should be fast and lightweight.

---

# Readiness Endpoint

A readiness endpoint might be:

    GET /ready

Response:

    {
      "status": "READY"
    }

Readiness can represent whether the application is capable of serving production traffic.

---

# Liveness Endpoint

A liveness endpoint might be:

    GET /live

Response:

    {
      "status": "ALIVE"
    }

The endpoint should generally represent application/process health rather than the health of every external dependency.

---

# Health Endpoint Design

Good health endpoints should be:

    Fast
    Lightweight
    Deterministic
    Reliable
    Predictable

Avoid:

    Heavy Database Queries
    Expensive Calculations
    Long External API Calls
    Complex Business Logic
    Full Business Transactions

---

# Liveness Dependency Checks

Be careful when making liveness dependent on external systems.

Bad design:

    Liveness
        |
        ↓
    External API
        |
        ↓
    External API Down
        |
        ↓
    Liveness Fails
        |
        ↓
    Container Restarts
        |
        ↓
    More Load
        |
        ↓
    More Failures

This can create a restart storm.

---

# Better Liveness Design

Liveness should generally check:

    Application Process
    Application Internal Functioning

Readiness can check whether:

    Critical Dependencies
    Required Services
    Database
    Configuration

are available for serving traffic.

---

# Health Check Dependency Model

A useful mental model is:

    Startup

    "Have I started?"

        ↓

    Readiness

    "Can I serve traffic?"

        ↓

    Liveness

    "Am I still functioning?"

---

# Kubernetes Deployment With Probes

Example configuration:

    apiVersion: apps/v1
    kind: Deployment

    metadata:
      name: payment

    spec:
      replicas: 3

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
              image: payment:1.4.7

              ports:
                - containerPort: 8080

              startupProbe:
                httpGet:
                  path: /health
                  port: 8080
                failureThreshold: 30
                periodSeconds: 10

              readinessProbe:
                httpGet:
                  path: /ready
                  port: 8080
                periodSeconds: 5
                timeoutSeconds: 2
                failureThreshold: 3

              livenessProbe:
                httpGet:
                  path: /live
                  port: 8080
                periodSeconds: 10
                timeoutSeconds: 2
                failureThreshold: 3

---

# Probe Execution Flow

When startupProbe is configured:

    Container Starts
        |
        ↓
    Startup Probe
        |
        +------ Fail ------→ Continue Checking
        |
        +------ Pass
                |
                ↓
          Readiness / Liveness
                |
                ↓
            Application
                |
                ↓
             Traffic

Startup protection prevents liveness and readiness from prematurely treating a slow-starting application as unhealthy.

---

# Kubernetes Service and Readiness

A Kubernetes Service normally routes traffic to eligible ready endpoints.

Example:

    Service
       |
       +-- Pod A → Ready
       +-- Pod B → Ready
       +-- Pod C → Not Ready
       +-- Pod D → Ready

Traffic goes to:

    Pod A
    Pod B
    Pod D

Pod C is not treated as a ready endpoint for normal Service traffic.

---

# EndpointSlices

Kubernetes uses EndpointSlices to track service endpoints.

Conceptually:

    Pod
      |
      ↓
    Readiness
      |
      ↓
    EndpointSlice
      |
      ↓
    Service
      |
      ↓
    Traffic

If a Pod becomes Not Ready, it is not normally treated as an eligible ready endpoint for standard Service traffic.

---

# Health Checks and ALB

AWS Application Load Balancer can perform target health checks.

Architecture:

    Users
      |
      ↓
    ALB
      |
      ↓
    Target Group
      |
      +-- Target A → Healthy
      +-- Target B → Healthy
      +-- Target C → Unhealthy

Traffic is directed according to the load balancer's target health and routing configuration.

---

# ALB Health Check

Typical ALB health-check settings include:

    Health Check Protocol
    Health Check Path
    Health Check Port
    Interval
    Timeout
    Healthy Threshold
    Unhealthy Threshold

Example:

    ALB
      |
      ↓
    GET /health
      |
      +------ 200 ------→ Healthy
      |
      +------ Failure ---→ Unhealthy

---

# ALB and Kubernetes Health

A production application can have multiple health layers.

    ALB Health Check
          |
          ↓
    Ingress
          |
          ↓
    Service
          |
          ↓
    Kubernetes Readiness
          |
          ↓
    Pod
          |
          ↓
    Application

All layers should be configured consistently.

---

# Health Check Layers

A production system can have:

    Layer 1
    Infrastructure Health

        ↓

    Layer 2
    Container Health

        ↓

    Layer 3
    Kubernetes Health

        ↓

    Layer 4
    Application Health

        ↓

    Layer 5
    Dependency Health

        ↓

    Layer 6
    Business Validation

---

# Infrastructure Health

Examples:

    EC2 Instance
    EKS Node
    ALB
    Network
    Load Balancer Target

Questions:

    Is the infrastructure reachable?

    Is the node available?

    Is the target healthy?

---

# Container Health

Questions:

    Is the container running?

    Is the process alive?

    Is the container repeatedly restarting?

Useful commands:

    kubectl get pods

    kubectl describe pod <pod>

---

# Application Health

Questions:

    Is the application responding?

    Is the API available?

    Is the application accepting requests?

Example:

    curl http://localhost:8080/health

---

# Dependency Health

Possible dependencies include:

    Database
    Redis
    RabbitMQ
    External APIs
    Internal Services

A readiness check can consider critical dependencies where appropriate.

---

# Business Health

Technical health does not always mean business health.

Example:

    HTTP 200
        |
        ↓
    Application Technically Healthy

But:

    Payment Transactions Failing
        |
        ↓
    Business Unhealthy

Therefore production validation should include business-level checks where appropriate.

---

# Smoke Tests

Smoke tests are lightweight tests that verify critical functionality after deployment.

Example:

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
    Continue

---

# Smoke Test vs Health Check

Health Check:

    "Is the application healthy?"

Smoke Test:

    "Does a critical user flow work?"

Example:

    Health:
        GET /health → 200

    Smoke:
        Login → Success
        Create Order → Success

Both are useful but serve different purposes.

---

# Synthetic Monitoring

Synthetic monitoring executes predefined requests against the application.

Example:

    Periodically
        |
        ↓
    Request /health
        |
        ↓
    Request Critical API
        |
        ↓
    Validate Response
        |
        ↓
    Alert If Failed

This can detect problems before users report them.

---

# Health Checks and Monitoring

Health checks provide signals.

Monitoring collects and analyzes those signals.

Example:

    Application
        |
        ↓
    Health
        |
        ↓
    Metrics / Events / Logs
        |
        +-- Prometheus
        +-- Grafana
        └-- ELK

---

# Prometheus and Health Metrics

Prometheus can monitor:

    Pod Availability
    Request Rate
    Error Rate
    Latency
    Restart Count
    Resource Usage

Example:

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

---

# Grafana Health Dashboard

A useful dashboard can display:

    Healthy Pods
    Unhealthy Pods
    Ready Pods
    Restart Count
    HTTP 5xx
    HTTP 4xx
    Request Rate
    P95 Latency
    CPU
    Memory

---

# ELK and Health Checks

ELK helps investigate application-level failures.

Flow:

    Health Check Failure
        |
        ↓
    Application Logs
        |
        ↓
    ELK
        |
        ↓
    Find Root Cause

Look for:

    Exceptions
    Connection Failures
    Timeout Errors
    Authentication Errors
    Database Errors
    Dependency Errors

---

# Health Checks and Alerts

A production system should alert on meaningful failures.

Examples:

    High Error Rate
    No Ready Pods
    High Restart Count
    Failed Health Checks
    High Latency
    ALB Unhealthy Targets

Alerts should focus on actionable conditions.

---

# Health Check Failure Flow

    Application
        |
        ↓
    Health Check
        |
        ↓
    Failure
        |
        ↓
    Kubernetes / Load Balancer
        |
        ↓
    Traffic Removed or Container Restarted
        |
        ↓
    Monitoring
        |
        ↓
    Alert
        |
        ↓
    Engineer Investigation

---

# Health Check and Auto-Healing

Kubernetes provides self-healing capabilities.

Example:

    Container
        |
        ↓
    Liveness Failure
        |
        ↓
    Container Restart
        |
        ↓
    Application Recovery

This reduces manual intervention for certain transient failures.

---

# Health Check and Traffic Removal

Readiness failure:

    Pod
      |
      ↓
    Readiness Failed
      |
      ↓
    Not Ready
      |
      ↓
    Removed From Service Traffic
      |
      ↓
    Application Recovers
      |
      ↓
    Readiness Pass
      |
      ↓
    Traffic Returns

---

# Temporary Dependency Failure

Suppose:

    Database
        |
        ↓
    Temporarily Unavailable

Application may become:

    Not Ready

Instead of:

    Liveness Failure
        |
        ↓
    Restart All Containers

This can be safer if the application process itself remains healthy.

---

# Avoid Restart Storms

A poorly designed liveness probe can cause:

    Dependency Failure
        |
        ↓
    Liveness Fails
        |
        ↓
    Restart
        |
        ↓
    Dependency Still Down
        |
        ↓
    Restart
        |
        ↓
    Restart
        |
        ↓
    Restart

This is a restart storm.

---

# Better Dependency Handling

Use:

    Readiness
        |
        ↓
    Stop Traffic

while:

    Liveness
        |
        ↓
    Check Application Process

This allows the application to remain running while it waits for a temporary dependency problem to recover.

---

# Health Checks and Graceful Shutdown

During termination:

    Pod
      |
      ↓
    Termination Signal
      |
      ↓
    Stop New Traffic
      |
      ↓
    Finish Existing Requests
      |
      ↓
    Shutdown

Readiness and graceful shutdown should work together.

---

# Health Checks During Pod Termination

Conceptually:

    Pod Running
       |
       ↓
    Termination Begins
       |
       ↓
    Pod Becomes Unavailable
       |
       ↓
    Traffic Draining
       |
       ↓
    Existing Requests Finish
       |
       ↓
    Container Stops

This helps reduce dropped requests during deployments.

---

# Health Checks and Rolling Deployment

Rolling deployment:

    Old Pods
        |
        ↓
    New Pod Created
        |
        ↓
    Startup Probe
        |
        ↓
    Readiness Probe
        |
        ↓
    Ready
        |
        ↓
    Traffic
        |
        ↓
    Old Pod Removed
        |
        ↓
    Next Pod

Health checks help control when new Pods become available.

---

# Health Checks and Blue-Green Deployment

Blue:

    Current Production

Green:

    New Version

Flow:

    Green
       |
       ↓
    Startup
       |
       ↓
    Readiness
       |
       ↓
    Health Check
       |
       ↓
    Smoke Test
       |
       ↓
    Validation
       |
       ↓
    Traffic Switch

---

# Health Checks and Canary Deployment

Canary deployment:

    Stable = 95%
    Canary = 5%

Flow:

    Canary
       |
       ↓
    Health Checks
       |
       ↓
    Metrics
       |
       ↓
    Analysis
       |
       +------ Healthy ------→ Increase Traffic
       |
       +------ Unhealthy ----→ Rollback

---

# Health Checks and CI/CD

Health checks are part of post-deployment validation.

Pipeline:

    Build
      |
      ↓
    Test
      |
      ↓
    Security Scan
      |
      ↓
    Deploy
      |
      ↓
    Health Check
      |
      ↓
    Smoke Test
      |
      ↓
    Validation
      |
      ↓
    Promote

---

# GitHub Actions Health Validation

Conceptual flow:

    GitHub Actions
        |
        ↓
    Deploy
        |
        ↓
    kubectl rollout status
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Success / Failure

---

# Rollout Status

Useful command:

    kubectl rollout status deployment/payment

This verifies whether the Deployment rollout has completed successfully.

---

# Check Pod Health

Command:

    kubectl get pods

Example:

    NAME                       READY   STATUS    RESTARTS
    payment-7f8d9c8d7f-abc12   1/1     Running   0
    payment-7f8d9c8d7f-def34   1/1     Running   0
    payment-7f8d9c8d7f-ghi56   1/1     Running   0

Healthy example:

    READY = 1/1
    STATUS = Running
    RESTARTS = 0

---

# Describe Pod Health

Command:

    kubectl describe pod <pod-name>

Look at:

    Conditions
    Events
    Readiness
    Liveness
    Startup
    Container State
    Restart Count

The Events section is especially useful for troubleshooting probe failures.

---

# Container Status

Command:

    kubectl get pod <pod-name> -o json

Useful information includes:

    Container State
    Last State
    Restart Count
    Ready
    Image

This can help identify:

    OOMKilled
    CrashLoopBackOff
    Restarted Containers

---

# Health Check Logs

If a probe fails:

    kubectl logs <pod-name>

For the previous container instance:

    kubectl logs <pod-name> --previous

Check for:

    Startup Errors
    Port Errors
    Configuration Errors
    Database Errors
    Dependency Errors

---

# Probe Failure Events

Command:

    kubectl describe pod <pod-name>

Example event:

    Readiness probe failed:
    HTTP probe failed with statuscode: 503

This indicates that the application returned an unsuccessful HTTP status for the readiness check.

---

# Connection Refused

Example:

    Liveness probe failed:
    dial tcp 10.0.0.10:8080:
    connect: connection refused

Possible causes:

    Application Not Started
    Wrong Port
    Container Not Listening
    Startup Too Slow
    Configuration Error

---

# Timeout Failure

Example:

    Readiness probe failed:
    context deadline exceeded

Possible causes:

    Application Too Slow
    Wrong Endpoint
    Network Issue
    Thread Pool Exhaustion
    Database Delay
    Resource Pressure

---

# HTTP 500 Health Failure

Example:

    Readiness probe failed:
    HTTP probe failed with statuscode: 500

Check:

    Application Logs
    Dependency Health
    Configuration
    Database
    Application Code

---

# HTTP 503 Health Failure

Example:

    Readiness probe failed:
    HTTP probe failed with statuscode: 503

This may indicate:

    Application Not Ready
    Dependency Unavailable
    Initialization Incomplete
    Service Overloaded

---

# Health Check Troubleshooting Flow

    Probe Failure
        |
        ↓
    kubectl describe pod
        |
        ↓
    Check Events
        |
        ↓
    kubectl logs
        |
        ↓
    Check Endpoint
        |
        ↓
    Check Port
        |
        ↓
    Check Configuration
        |
        ↓
    Check Dependencies
        |
        ↓
    Check Resources
        |
        ↓
    Fix
        |
        ↓
    Validate Again

---

# Health Check Troubleshooting

## Step 1: Check Pods

    kubectl get pods

Verify:

    STATUS
    READY
    RESTARTS

---

# Step 2: Describe Pod

    kubectl describe pod <pod-name>

Check:

    Events
    Probe Failures
    Container State
    Conditions

---

# Step 3: Check Logs

    kubectl logs <pod-name>

If the container restarted:

    kubectl logs <pod-name> --previous

---

# Step 4: Check Service

    kubectl get svc

Verify:

    Service Port
    Target Port
    Selector

---

# Step 5: Check Endpoints

    kubectl get endpoints

or:

    kubectl get endpointslices

Verify that Ready Pods are represented as eligible endpoints.

---

# Step 6: Test Health Endpoint

    curl http://<service>:8080/health

Then:

    curl http://<service>:8080/ready

Verify expected responses.

---

# Step 7: Check Resources

    kubectl top pods

Check:

    CPU
    Memory

High resource usage can cause application and probe failures.

---

# Step 8: Check Dependencies

Verify:

    Database
    RabbitMQ
    External APIs
    Internal Services
    Network Connectivity

---

# Health Check Scenario

## Application Takes 2 Minutes to Start

Problem:

    Liveness Probe Starts Too Early

Result:

    Container Restart

Solution:

    Use Startup Probe

Example:

    startupProbe:
      httpGet:
        path: /health
        port: 8080
      failureThreshold: 18
      periodSeconds: 10

This provides approximately three minutes of startup-checking time.

---

# Health Check Scenario

## Application Is Running But Database Is Down

Possible behavior:

    Application Process
        |
        ↓
    Still Running

    Database
        |
        ↓
    Unavailable

If readiness depends on the database:

    Readiness → Fail
        |
        ↓
    Remove Pod From Traffic

Liveness should not necessarily fail just because the database is temporarily unavailable.

---

# Health Check Scenario

## Application Is Frozen

Suppose:

    Process Still Exists
        |
        ↓
    Application Not Responding

Liveness probe:

    Fails
        |
        ↓
    Kubernetes Restarts Container

This is a good use case for liveness.

---

# Health Check Scenario

## Application Is Starting

    Container Starts
        |
        ↓
    Application Initialization
        |
        ↓
    Startup Probe
        |
        ↓
    Pass
        |
        ↓
    Readiness
        |
        ↓
    Ready
        |
        ↓
    Traffic

This is a good use case for startupProbe.

---

# Health Check Scenario

## Application Temporarily Cannot Serve Requests

Suppose:

    Application Running
        |
        ↓
    Critical Dependency Unavailable
        |
        ↓
    Readiness Failure

Result:

    Pod Remains Running
        |
        ↓
    Traffic Removed

When dependency recovers:

    Readiness Pass
        |
        ↓
    Traffic Returns

---

# Health Check Scenario

## Wrong Probe Port

Application:

    Listening on 8080

Probe:

    Port 8081

Result:

    Connection Refused

Fix:

    readinessProbe:
      httpGet:
        path: /ready
        port: 8080

Always verify the actual container listening port.

---

# Health Check Scenario

## Wrong Health Endpoint

Application exposes:

    /health

Probe uses:

    /healthz

Result:

    HTTP 404

Fix:

    Use the correct endpoint.

---

# Health Check Scenario

## Health Endpoint Is Too Slow

Health endpoint:

    Takes 10 seconds

Probe:

    timeoutSeconds = 2

Result:

    Probe Timeout

Fix:

    Improve Health Endpoint

or:

    Adjust Timeout

But do not hide an actual performance problem by simply increasing timeouts without investigation.

---

# Health Check Scenario

## Application Uses HTTPS

If the endpoint requires HTTPS, configure the appropriate scheme.

Example:

    readinessProbe:
      httpGet:
        path: /ready
        port: 8443
        scheme: HTTPS

Configuration must match how the application exposes the health endpoint.

---

# Health Check Scenario

## Container Has No HTTP Endpoint

Use:

    TCP Probe

or:

    Exec Probe

Example:

    readinessProbe:
      tcpSocket:
        port: 8080

Choose the probe that accurately represents readiness.

---

# Health Check Scenario

## Health Endpoint Requires Authentication

If the health endpoint requires credentials, the probe must be designed appropriately.

Often it is better to expose a dedicated internal health endpoint that does not require application-level authentication while still avoiding sensitive information.

---

# Health Check Security

Health endpoints should not expose:

    Passwords
    API Keys
    Tokens
    Secret Values
    Internal Credentials

Avoid:

    {
      "databasePassword": "secret",
      "apiKey": "xxxxx"
    }

Prefer:

    {
      "status": "UP"
    }

---

# Health Check Best Practices

- Keep health endpoints lightweight
- Use readiness to control traffic
- Use liveness to detect application failure
- Use startup probes for slow-starting applications
- Avoid making liveness depend on external services
- Use appropriate timeouts
- Use appropriate failure thresholds
- Validate health endpoints before deployment
- Monitor health failures
- Test probes during development
- Use graceful shutdown
- Keep health responses simple
- Avoid sensitive information
- Test failure scenarios
- Use multiple layers of validation
- Monitor application behavior after deployment

---

# Health Check Anti-Patterns

## Using Liveness for Readiness

Bad:

    Database Down
        |
        ↓
    Liveness Fails
        |
        ↓
    Restart Container

Better:

    Database Down
        |
        ↓
    Readiness Fails
        |
        ↓
    Remove From Traffic

---

# Health Check Anti-Pattern

## No Readiness Probe

Bad:

    Pod Starts
        |
        ↓
    Immediately Receives Traffic
        |
        ↓
    Application Still Starting
        |
        ↓
    User Errors

Better:

    Pod Starts
        |
        ↓
    Readiness
        |
        ↓
    Ready
        |
        ↓
    Traffic

---

# Health Check Anti-Pattern

## Heavy Health Endpoint

Bad:

    /health
        |
        +-- Query Database
        +-- Call RabbitMQ
        +-- Call External APIs
        +-- Perform Complex Logic
        +-- Run Expensive Queries

This can increase load and create cascading failures.

Better:

    Lightweight
    Fast
    Deterministic

---

# Health Check Anti-Pattern

## Same Endpoint for Everything

Using one complex endpoint for:

    Startup
    Readiness
    Liveness

can make the health model unclear.

Better:

    /health
    /ready
    /live

when the application's needs justify separate semantics.

---

# Health Check Anti-Pattern

## Too Aggressive Probes

Example:

    periodSeconds: 1
    timeoutSeconds: 1
    failureThreshold: 1

A temporary delay can immediately cause failure.

Better:

    Configure thresholds based on real application behavior.

---

# Health Check Anti-Pattern

## Too Lenient Probes

Example:

    failureThreshold: 100

This may delay detection of real failures.

Choose thresholds based on:

    Recovery Time
    Failure Type
    Application Requirements

---

# Health Check Anti-Pattern

## Ignoring Startup Time

Bad:

    Slow Application
        |
        ↓
    Liveness Starts Immediately
        |
        ↓
    Restart Loop

Better:

    Startup Probe
        |
        ↓
    Application Initializes
        |
        ↓
    Readiness / Liveness

---

# Health Check Anti-Pattern

## Assuming Running Means Healthy

Bad:

    STATUS = Running

Therefore:

    Application = Healthy

This is not always true.

A container can be:

    Running
    but
    Not Ready

or:

    Running
    but
    Application Frozen

Always evaluate the correct health signals.

---

# Health Checks and Deployment Strategies

Deployment strategy determines how versions are released.

Health checks determine whether instances are healthy enough to participate.

Together:

    Deployment Strategy
        +
    Health Checks
        =
    Safer Deployment

---

# Rolling Deployment

    New Pod
       |
       ↓
    Startup
       |
       ↓
    Readiness
       |
       ↓
    Ready
       |
       ↓
    Traffic
       |
       ↓
    Old Pod Removed

---

# Blue-Green Deployment

    Green
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

---

# Canary Deployment

    Canary
       |
       ↓
    5%
       |
       ↓
    Health Check
       |
       ↓
    Metrics
       |
       ↓
    Increase Traffic

If health or metrics become unhealthy:

    Stop
       |
       ↓
    Abort
       |
       ↓
    Rollback

---

# Health Checks + Prometheus

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

Monitor:

    Availability
    Error Rate
    Latency
    Restart Count
    Pod Health
    Request Rate

---

# Health Checks + ELK

    Application
        |
        ↓
    Logs
        |
        ↓
    ELK
        |
        ↓
    Investigate Health Failures

Example:

    Readiness Failed
        |
        ↓
    Search Logs
        |
        ↓
    Database Connection Error

---

# Health Checks + ArgoCD

GitOps deployment:

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
      ↓
    Kubernetes Health
      |
      ↓
    Application Status

Health information helps determine whether the deployed resources are progressing toward the desired state.

---

# Health Checks and Progressive Delivery

Progressive delivery:

    Canary
      |
      ↓
    Health Metrics
      |
      ↓
    Analysis
      |
      +------ Pass ------→ Promote
      |
      +------ Fail ------→ Abort

Health analysis becomes part of the deployment decision.

---

# Complete DevOps Health Architecture

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
        +-- Startup Probe
        +-- Readiness Probe
        +-- Liveness Probe
        |
        ↓
    Service
        |
        ↓
    ALB
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

# Complete Health Validation Flow

    Deployment
        |
        ↓
    Container Starts
        |
        ↓
    Startup Probe
        |
        ↓
    Startup Complete
        |
        ↓
    Readiness Probe
        |
        +------ Fail ------→ No Traffic
        |
        +------ Pass
                |
                ↓
              Traffic
                |
                ↓
          Liveness Probe
                |
                +------ Pass → Continue
                |
                +------ Fail → Restart
                |
                ↓
             Monitoring
                |
                ↓
             Smoke Tests
                |
                ↓
             Validation
                |
                ↓
             Promotion

---

# Production Health Check Runbook

## Step 1: Check Pods

    kubectl get pods

Verify:

    STATUS
    READY
    RESTARTS

---

# Step 2: Check Deployment

    kubectl get deployment <deployment-name>

Verify:

    READY
    UP-TO-DATE
    AVAILABLE

---

# Step 3: Check Rollout

    kubectl rollout status deployment/<deployment-name>

Verify that the rollout completes successfully.

---

# Step 4: Describe Pod

    kubectl describe pod <pod-name>

Check:

    Events
    Probe Failures
    Conditions
    Container State

---

# Step 5: Check Logs

    kubectl logs <pod-name>

If restarted:

    kubectl logs <pod-name> --previous

---

# Step 6: Check Service

    kubectl get svc

Verify:

    Service Port
    Target Port
    Selector

---

# Step 7: Check Endpoints

    kubectl get endpoints

or:

    kubectl get endpointslices

Verify healthy Pods are represented as eligible endpoints.

---

# Step 8: Test Health Endpoint

    curl http://<service>:8080/health

Then:

    curl http://<service>:8080/ready

Verify expected responses.

---

# Step 9: Check Metrics

Review:

    Prometheus
    Grafana

Check:

    Error Rate
    Latency
    CPU
    Memory
    Restarts
    Request Rate

---

# Step 10: Check Logs

Review ELK for:

    Exceptions
    HTTP 5xx
    Database Errors
    Dependency Failures
    Authentication Errors

---

# Step 11: Run Smoke Tests

Validate:

    Login
    Critical API
    Database
    Service Communication
    Important Business Flow

---

# Step 12: Confirm Production Health

Final state:

    Pods Healthy
    Pods Ready
    No Unexpected Restarts
    Health Checks Passing
    Smoke Tests Passing
    Error Rate Normal
    Latency Normal
    Logs Normal

---

# Health Check Interview Questions

## Basic

1. What is a health check?

2. Why are health checks important?

3. What are Kubernetes probes?

4. What is a readiness probe?

5. What is a liveness probe?

6. What is a startup probe?

7. What is the difference between readiness and liveness?

8. What happens when readiness fails?

9. What happens when liveness fails?

10. Why do we need startup probes?

---

# Health Check Interview Questions

## Intermediate

11. What are the different types of Kubernetes probes?

12. What is an HTTP probe?

13. What is a TCP probe?

14. What is an Exec probe?

15. What is a gRPC health check?

16. What is initialDelaySeconds?

17. What is periodSeconds?

18. What is timeoutSeconds?

19. What is failureThreshold?

20. What is successThreshold?

21. How do health checks work during Rolling Deployment?

22. How do health checks work with Kubernetes Services?

23. How does readiness affect traffic?

24. How does liveness affect container recovery?

---

# Health Check Interview Questions

## Advanced

25. How would you design health checks for a production microservices application?

26. How would you prevent liveness probes from causing restart storms?

27. How would you design readiness for an application with database dependencies?

28. How would you design health checks for a slow-starting Java application?

29. How would you troubleshoot repeated readiness failures?

30. How would you troubleshoot a container that keeps restarting because of liveness failures?

31. How would you integrate health checks into a Canary deployment?

32. How would you use Prometheus to validate a deployment?

33. How would you design health checks for an EKS application behind an ALB?

34. How would you implement zero-downtime deployment using health checks?

35. What is the difference between application health and business health?

36. How can poorly designed health checks cause cascading failures?

---

# Scenario-Based Interview Question

## Pod Is Running but Users Receive Errors

You run:

    kubectl get pods

The Pod shows:

    STATUS = Running

But users receive errors.

Do not assume the application is healthy.

Check:

    kubectl describe pod <pod>

Then:

    kubectl logs <pod>

Check:

    Readiness
    Service
    Endpoints
    ALB Target Health
    Application Health

The Pod may be Running but Not Ready.

---

# Scenario-Based Interview Question

## Pods Restart Every Few Minutes

Check:

    kubectl get pods

If restart count increases:

    kubectl describe pod <pod>

Then:

    kubectl logs <pod> --previous

Check:

    Liveness Failure
    OOMKilled
    Application Crash
    Configuration Error

If liveness is the cause:

    Review Probe
    Review Application
    Review Startup Behavior

---

# Scenario-Based Interview Question

## Application Takes 3 Minutes to Start

Problem:

    Liveness Probe Starts Too Early

Solution:

    Use Startup Probe

Concept:

    Startup
       |
       ↓
    3 Minute Initialization
       |
       ↓
    Startup Success
       |
       ↓
    Readiness
       |
       ↓
    Traffic

---

# Scenario-Based Interview Question

## Database Is Down

Application:

    Running

Database:

    Down

Preferred behavior:

    Readiness
        |
        ↓
    Fail
        |
        ↓
    Remove From Traffic

Avoid automatically restarting the application through liveness unless the application process itself is genuinely unhealthy.

---

# Scenario-Based Interview Question

## Canary Deployment Has Increased Errors

Initial:

    Stable = 95%
    Canary = 5%

Metrics:

    Stable Error Rate = 0.5%
    Canary Error Rate = 8%

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
    Investigate

Health metrics become part of the deployment decision.

---

# Scenario-Based Interview Question

## New Pod Is Not Receiving Traffic

Check:

    kubectl get pods

Then:

    kubectl describe pod <pod>

Check:

    Readiness Probe

Then:

    kubectl get endpoints

Check:

    Service Selector
    Pod Labels
    Ready Condition

Possible causes:

    Readiness Failure
    Wrong Service Selector
    Wrong Port
    Wrong Target Port

---

# Scenario-Based Interview Question

## ALB Shows Unhealthy Targets

Check:

    ALB Health Check Path
    ALB Health Check Port
    Kubernetes Service
    Target Port
    Pod Port
    Readiness
    Security Groups
    Application Logs

Flow:

    ALB
      |
      ↓
    Target
      |
      ↓
    Health Check
      |
      ↓
    Failure

Find where the request is failing.

---

# Scenario-Based Interview Question

## Rolling Deployment Is Stuck

Run:

    kubectl rollout status deployment/payment

Then:

    kubectl get pods

Then:

    kubectl describe pod <pod>

Look for:

    Readiness Failure
    ImagePullBackOff
    CrashLoopBackOff
    Pending
    Insufficient Resources

Fix the root cause before continuing the rollout.

---

# Scenario-Based Interview Question

## Health Endpoint Returns 200 But Application Is Broken

Possible reason:

    /health

only checks:

    Process Is Running

but:

    Critical Business Function
        |
        ↓
    Broken

Solution:

    Keep liveness lightweight

and use:

    Readiness
    Smoke Tests
    Business Metrics
    Application Monitoring

for broader validation.

---

# Health Check Best Practice Architecture

    /live
       |
       ↓
    Process Health


    /ready
       |
       ↓
    Traffic Readiness


    /health
       |
       ↓
    Application Health


    Smoke Tests
       |
       ↓
    Critical Business Flow


    Monitoring
       |
       ↓
    Production Behavior

Each layer answers a different question.

---

# Health Check Strategy for Java Application

Example:

    Java Application
        |
        +-- /live
        |
        +-- /ready
        |
        └-- /health

Kubernetes:

    Startup Probe
        |
        ↓
    /health

    Readiness Probe
        |
        ↓
    /ready

    Liveness Probe
        |
        ↓
    /live

This provides clear health semantics.

---

# Health Check Strategy for Node.js

Example:

    Node.js Application
        |
        +-- /health
        +-- /ready
        └-- /live

Kubernetes uses these endpoints according to the deployment requirements.

---

# Health Check Strategy for Python

Example:

    Python Application
        |
        +-- /health
        +-- /ready
        └-- /live

The same conceptual model applies:

    Startup
    Readiness
    Liveness

---

# Health Check Strategy for Microservices

Each microservice should expose health information appropriate to its architecture.

Example:

    User
       |
       +-- /live
       └-- /ready

    Product
       |
       +-- /live
       └-- /ready

    Payment
       |
       +-- /live
       └-- /ready

    Inventory
       |
       +-- /live
       └-- /ready

This allows Kubernetes to manage each service independently.

---

# Complete Production Health Architecture

    Users
       |
       ↓
    Route 53
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
    Ready Pods
       |
       ↓
    Application
       |
       +-- Startup Probe
       |
       +-- Readiness Probe
       |
       +-- Liveness Probe
       |
       +-- Application Health
       |
       +-- Dependency Health
       |
       ↓
    Monitoring
       |
       +-- Prometheus
       +-- Grafana
       └-- ELK

---

# Complete Deployment Health Flow

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
    New Pod
        |
        ↓
    Startup Probe
        |
        ↓
    Readiness Probe
        |
        +------ Fail ------→ No Traffic
        |
        +------ Pass
                |
                ↓
              Traffic
                |
                ↓
          Liveness Probe
                |
                +------ Fail → Restart
                |
                +------ Pass
                        |
                        ↓
                    Monitoring
                        |
                        ↓
                    Smoke Tests
                        |
                        ↓
                    Promotion

---

# Final Health Check Mental Model

Remember these three questions:

    STARTUP

    "Has the application started?"

        ↓

    READINESS

    "Can the application receive traffic?"

        ↓

    LIVENESS

    "Is the application still functioning?"

The actions are:

    Startup Failure
        |
        ↓
    Keep Checking / Eventually Restart If Startup Cannot Succeed


    Readiness Failure
        |
        ↓
    Remove From Traffic


    Liveness Failure
        |
        ↓
    Restart Container

---

# Final Concept

Health checks are a critical part of reliable DevOps and Kubernetes deployments.

The complete model is:

    Application Starts
        |
        ↓
    Startup Probe
        |
        ↓
    Application Initialized
        |
        ↓
    Readiness Probe
        |
        +------ Fail ------→ No Traffic
        |
        +------ Pass
                |
                ↓
              Traffic
                |
                ↓
          Liveness Probe
                |
                +------ Healthy ------→ Continue
                |
                +------ Failed ------→ Restart
                |
                ↓
             Monitoring
                |
                +-- Prometheus
                +-- Grafana
                └-- ELK

A production-ready health-check strategy should combine:

    Startup Probes
        +
    Readiness Probes
        +
    Liveness Probes
        +
    ALB Health Checks
        +
    Application Health
        +
    Smoke Tests
        +
    Prometheus
        +
    Grafana
        +
    ELK
        +
    Graceful Shutdown
        +
    Deployment Validation

The most important rules are:

    Readiness controls traffic.

    Liveness controls container recovery.

    Startup protects slow application initialization.

A strong health-check design prevents:

    Traffic to Unready Pods
        +
    Unnecessary Restarts
        +
    Deployment Failures
        +
    Downtime
        +
    Cascading Failures

and supports:

    Rolling Deployments
        +
    Blue-Green Deployments
        +
    Canary Deployments
        +
    Zero-Downtime Releases
        +
    Kubernetes Self-Healing
        +
    Production Reliability

The ultimate goal is:

    Detect
        |
        ↓
    Decide
        |
        ↓
    Remove Unhealthy Traffic
        |
        ↓
    Recover
        |
        ↓
    Validate
        |
        ↓
    Restore Healthy Service