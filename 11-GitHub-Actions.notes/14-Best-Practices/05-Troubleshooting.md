# Troubleshooting

Troubleshooting is one of the most important DevOps skills.

A DevOps engineer must be able to:

    Detect
        |
        ↓
    Collect Evidence
        |
        ↓
    Analyze
        |
        ↓
    Identify Root Cause
        |
        ↓
    Fix
        |
        ↓
    Validate
        |
        ↓
    Prevent Recurrence

The goal of troubleshooting is not to randomly change things.

The goal is:

    Observe
        +
    Understand
        +
    Isolate
        +
    Fix
        +
    Validate

---

# 1. Troubleshooting Mindset

A good troubleshooting process follows:

    Problem
        |
        ↓
    Evidence
        |
        ↓
    Hypothesis
        |
        ↓
    Test
        |
        ↓
    Root Cause
        |
        ↓
    Fix
        |
        ↓
    Validation

Avoid:

    Problem
        |
        ↓
    Random Change
        |
        ↓
    Another Problem

---

# 2. First Rule: Do Not Guess

Bad:

    Application Slow
        |
        ↓
    Increase CPU
        |
        ↓
    Restart Everything

Better:

    Application Slow
        |
        ↓
    Collect Metrics
        |
        ↓
    Check Logs
        |
        ↓
    Identify Bottleneck
        |
        ↓
    Apply Targeted Fix

---

# 3. Troubleshooting Evidence

Collect evidence from:

    Metrics
    +
    Logs
    +
    Events
    +
    Configuration
    +
    Deployment History
    +
    Infrastructure
    +
    Network
    +
    Application Behavior

---

# 4. Start With Impact

First determine:

    What Is Broken?
        |
        ↓
    Who Is Affected?
        |
        ↓
    How Many Users?
        |
        ↓
    Which Environment?
        |
        ↓
    Since When?
        |
        ↓
    What Changed?

---

# 5. Production Incident Flow

    Alert
        |
        ↓
    Confirm Incident
        |
        ↓
    Assess Impact
        |
        ↓
    Stabilize
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
        |
        ↓
    Document
        |
        ↓
    Prevent Recurrence

---

# 6. Check Recent Changes

Many incidents are related to recent changes.

Check:

    Application Deployment
    +
    Configuration Change
    +
    Infrastructure Change
    +
    Terraform Change
    +
    Kubernetes Change
    +
    Dependency Update
    +
    Security Policy Change

Timeline:

    Last Known Good
        |
        ↓
    Change
        |
        ↓
    Incident

---

# 7. Five Whys

The Five Whys technique helps identify root causes.

Example:

    Why did the application fail?

        ↓

    Database connection failed.

        ↓

    Why?

    Database connections were exhausted.

        ↓

    Why?

    Application pods increased significantly.

        ↓

    Why?

    HPA scaled aggressively.

        ↓

    Why?

    Scaling threshold was incorrectly configured.

Root cause:

    Incorrect Autoscaling Configuration

---

# 8. Symptoms vs Root Cause

Example:

    Symptom:
    Pods are restarting.

Possible root cause:

    OOMKilled
    +
    Failed Probe
    +
    Application Crash
    +
    Configuration Error
    +
    Dependency Failure

Do not treat the symptom as the root cause.

---

# 9. Troubleshooting Layers

Use a layered approach:

    User
        |
        ↓
    DNS
        |
        ↓
    Load Balancer
        |
        ↓
    Network
        |
        ↓
    Kubernetes
        |
        ↓
    Container
        |
        ↓
    Application
        |
        ↓
    Database

Move from outside to inside.

---

# 10. Application Troubleshooting

Check:

    Application Status
    +
    Logs
    +
    CPU
    +
    Memory
    +
    Dependencies
    +
    Configuration
    +
    Recent Changes

---

# 11. Linux Troubleshooting

Important commands:

    ps
    top
    free
    df
    du
    ss
    netstat
    systemctl
    journalctl
    curl
    ping
    dig
    nslookup
    grep
    awk
    tail

---

# 12. Check Processes

Command:

    ps -ef

Purpose:

    List Running Processes

Example:

    ps -ef | grep nginx

---

# 13. Check CPU

Command:

    top

Look for:

    High CPU Processes
    +
    Load Average
    +
    Memory Usage

---

# 14. Check Memory

Command:

    free -h

Check:

    Total
    +
    Used
    +
    Free
    +
    Available
    +
    Swap

---

# 15. Check Disk Space

Command:

    df -h

Example:

    Filesystem
        |
        ↓
    95% Used
        |
        ↓
    Disk Full Risk

---

# 16. Check Directory Size

Command:

    du -sh <directory>

Find large directories:

    du -sh *

---

# 17. Check Network Ports

Command:

    ss -lntp

Purpose:

    Identify Listening Ports
        +
    Identify Processes

---

# 18. Check Service Status

Command:

    systemctl status nginx

Check:

    Active State
    +
    Recent Logs
    +
    Process
    +
    Failure Reason

---

# 19. Check System Logs

Command:

    journalctl

Examples:

    journalctl -u nginx

    journalctl -f

---

# 20. Test Application Locally

Use:

    curl

Example:

    curl http://localhost:8080

If local request succeeds but external request fails:

    Application
        |
        ↓
    Healthy

Possible problem:

    Load Balancer
    +
    Network
    +
    DNS
    +
    Firewall

---

# 21. DNS Troubleshooting

Commands:

    dig example.com

    nslookup example.com

Check:

    DNS Resolution
    +
    Record
    +
    IP Address
    +
    TTL

---

# 22. DNS Troubleshooting Flow

    User
        |
        ↓
    DNS Query
        |
        ↓
    DNS Server
        |
        ↓
    IP Address
        |
        ↓
    Application

If DNS fails:

    Check Record
        |
        ↓
    Check Resolver
        |
        ↓
    Check TTL
        |
        ↓
    Check DNS Configuration

---

# 23. Network Connectivity

Use:

    ping
    curl
    nc
    ss
    traceroute

Example:

    curl -v https://example.com

This can show:

    DNS
    +
    TCP
    +
    TLS
    +
    HTTP

---

# 24. HTTP Status Codes

Important codes:

    200 → Success
    201 → Created
    301 → Redirect
    302 → Temporary Redirect
    400 → Bad Request
    401 → Unauthorized
    403 → Forbidden
    404 → Not Found
    408 → Request Timeout
    409 → Conflict
    429 → Too Many Requests
    500 → Internal Server Error
    502 → Bad Gateway
    503 → Service Unavailable
    504 → Gateway Timeout

---

# 25. Troubleshooting 400

Possible causes:

    Invalid Request
    +
    Missing Parameters
    +
    Invalid JSON
    +
    Invalid Headers

Check:

    Client Request
        |
        ↓
    API Logs
        |
        ↓
    Validation Errors

---

# 26. Troubleshooting 401

Possible causes:

    Missing Authentication
    +
    Invalid Token
    +
    Expired Token
    +
    Incorrect Credentials

Check:

    Authentication
        |
        ↓
    Token
        |
        ↓
    Authorization Header

---

# 27. Troubleshooting 403

Possible causes:

    Permission Denied
    +
    IAM Policy
    +
    RBAC
    +
    Network Policy
    +
    Application Authorization

---

# 28. Troubleshooting 404

Possible causes:

    Incorrect URL
    +
    Incorrect Route
    +
    Service Configuration
    +
    Ingress Rule

---

# 29. Troubleshooting 429

429 means:

    Too Many Requests

Possible causes:

    Rate Limiting
    +
    Traffic Spike
    +
    Client Retry Storm

Check:

    Request Rate
    +
    Rate Limits
    +
    Client Behavior

---

# 30. Troubleshooting 500

500 means:

    Internal Server Error

Check:

    Application Logs
    +
    Stack Trace
    +
    Dependencies
    +
    Recent Deployment
    +
    Database

---

# 31. Troubleshooting 502

502 often means:

    Gateway
        |
        ↓
    Backend
        |
        X
    Invalid / Failed Response

Check:

    Load Balancer
    +
    Ingress
    +
    Backend
    +
    Service
    +
    Application

---

# 32. Troubleshooting 503

503 generally indicates:

    Service Unavailable

Possible causes:

    No Healthy Pods
    +
    Readiness Failure
    +
    Service Misconfiguration
    +
    Backend Unavailable
    +
    Capacity Problem

---

# 33. Troubleshooting 504

504 indicates:

    Gateway Timeout

Possible causes:

    Slow Backend
    +
    Database
    +
    External API
    +
    Network
    +
    Timeout Configuration

---

# 34. Kubernetes Troubleshooting

Start with:

    kubectl get pods

Then:

    kubectl describe pod <pod>

Then:

    kubectl logs <pod>

For previous crash:

    kubectl logs <pod> --previous

---

# 35. Check Pod Status

Command:

    kubectl get pods -n <namespace>

Look for:

    Running
    +
    Pending
    +
    CrashLoopBackOff
    +
    ImagePullBackOff
    +
    Error
    +
    Completed

---

# 36. Check Pod Details

Command:

    kubectl describe pod <pod> -n <namespace>

Check:

    Events
    +
    Container State
    +
    Restart Count
    +
    Probes
    +
    Volumes
    +
    Scheduling

---

# 37. Check Pod Logs

Command:

    kubectl logs <pod> -n <namespace>

For multiple containers:

    kubectl logs <pod> -c <container> -n <namespace>

---

# 38. Previous Container Logs

If the container restarted:

    kubectl logs <pod> --previous -n <namespace>

This is extremely useful for:

    CrashLoopBackOff
    +
    OOMKilled
    +
    Application Crashes

---

# 39. Check Kubernetes Events

Command:

    kubectl get events -n <namespace>

Or:

    kubectl describe pod <pod>

Events can reveal:

    FailedScheduling
    +
    FailedMount
    +
    ImagePullBackOff
    +
    Unhealthy
    +
    BackOff

---

# 40. CrashLoopBackOff

CrashLoopBackOff means the container is repeatedly starting and failing.

Flow:

    Pod Start
        |
        ↓
    Container Starts
        |
        ↓
    Application Fails
        |
        ↓
    Container Restarts
        |
        ↓
    Backoff
        |
        ↓
    Retry

---

# 41. CrashLoopBackOff Troubleshooting

Check:

    kubectl describe pod <pod>

Then:

    kubectl logs <pod> --previous

Then:

    Check Environment Variables
    +
    Secrets
    +
    ConfigMaps
    +
    Probes
    +
    Resource Limits
    +
    Application Errors

---

# 42. ImagePullBackOff

Possible causes:

    Wrong Image Name
    +
    Wrong Tag
    +
    Registry Authentication
    +
    Network Problem
    +
    Image Does Not Exist

Flow:

    Pod
        |
        ↓
    Pull Image
        |
        X
    Failure
        |
        ↓
    ImagePullBackOff

---

# 43. ImagePullBackOff Troubleshooting

Check:

    kubectl describe pod <pod>

Look at:

    Events

Verify:

    Image Name
    +
    Tag
    +
    Registry
    +
    imagePullSecrets
    +
    Node Network Access

---

# 44. Pod Pending

Possible causes:

    Insufficient CPU
    +
    Insufficient Memory
    +
    Taints
    +
    Node Selector
    +
    Affinity
    +
    No Available Nodes

Check:

    kubectl describe pod <pod>

---

# 45. FailedScheduling

Example:

    0/3 nodes available

Possible reasons:

    Insufficient CPU
    +
    Insufficient Memory
    +
    Taints
    +
    Affinity
    +
    Resource Constraints

---

# 46. OOMKilled

Meaning:

    Container Exceeded Memory Limit

Check:

    kubectl describe pod <pod>

Look for:

    Reason: OOMKilled

Investigate:

    Memory Usage
    +
    Application Behavior
    +
    Memory Limit
    +
    Memory Leak

---

# 47. High CPU in Kubernetes

Check:

    kubectl top pod

and:

    kubectl top nodes

Then investigate:

    Traffic
    +
    CPU Limits
    +
    Application
    +
    HPA
    +
    Recent Changes

---

# 48. High Memory in Kubernetes

Check:

    kubectl top pod

Then:

    Check Memory Trend
        |
        ↓
    Check Application
        |
        ↓
    Check Limits
        |
        ↓
    Check Memory Leak
        |
        ↓
    Fix

---

# 49. Pod Restart Count Increasing

Command:

    kubectl get pods

Check:

    RESTARTS

Possible causes:

    Application Crash
    +
    OOMKilled
    +
    Liveness Probe Failure
    +
    Node Problem

---

# 50. Liveness Probe Failure

Possible causes:

    Application Hung
    +
    Wrong Endpoint
    +
    Wrong Port
    +
    Timeout Too Short
    +
    Application Startup Slow

Check:

    kubectl describe pod <pod>

---

# 51. Readiness Probe Failure

A readiness failure means:

    Pod Not Ready
        |
        ↓
    No Traffic

Possible causes:

    Application Not Ready
    +
    Wrong Endpoint
    +
    Dependency Failure
    +
    Wrong Port
    +
    Timeout

---

# 52. Startup Probe Failure

Possible causes:

    Application Takes Too Long
    +
    Wrong Probe
    +
    Wrong Port
    +
    Incorrect Thresholds

Review:

    initialDelaySeconds
    +
    periodSeconds
    +
    timeoutSeconds
    +
    failureThreshold

---

# 53. Service Not Reachable

Check:

    kubectl get svc

Then:

    kubectl describe svc <service>

Then:

    kubectl get endpoints <service>

---

# 54. Service Has No Endpoints

If:

    Service
        |
        ↓
    No Endpoints

possible cause:

    Selector Does Not Match Pods

Example:

    Service Selector
        |
        ↓
    app: frontend

Pod:

    app: backend

No Match.

---

# 55. Service Selector Problem

Check:

    kubectl get pods --show-labels

Compare:

    Service Selector

with:

    Pod Labels

They must match appropriately.

---

# 56. Ingress Troubleshooting

Check:

    kubectl get ingress

Then:

    kubectl describe ingress <name>

Verify:

    Host
    +
    Path
    +
    Backend
    +
    Service
    +
    Port
    +
    TLS

---

# 57. ALB Troubleshooting

For ALB-based Kubernetes ingress check:

    Ingress
        |
        ↓
    ALB
        |
        ↓
    Target Group
        |
        ↓
    Service
        |
        ↓
    Pod

Check each layer.

---

# 58. ALB Target Unhealthy

Possible causes:

    Readiness Failure
    +
    Wrong Target Port
    +
    Security Group
    +
    Health Check Path
    +
    Application Not Listening

---

# 59. Application Listening on Wrong Port

Example:

    Container Port = 8080

but application listens on:

    3000

Result:

    Connection Failure

Verify:

    Application
        |
        ↓
    Container Port
        |
        ↓
    Service Port
        |
        ↓
    Target Port
        |
        ↓
    Load Balancer

---

# 60. DNS Works but Application Fails

If:

    DNS
        |
        ↓
    Correct IP

but:

    Application
        |
        X
    Fails

check:

    Load Balancer
    +
    Network
    +
    Service
    +
    Application

---

# 61. DNS Does Not Resolve

Check:

    dig
    nslookup

Verify:

    DNS Record
    +
    Hosted Zone
    +
    Nameservers
    +
    TTL
    +
    Domain Configuration

---

# 62. TLS Certificate Problem

Possible causes:

    Expired Certificate
    +
    Wrong Certificate
    +
    Wrong Domain
    +
    Incorrect Listener
    +
    TLS Configuration

Check:

    Client
        |
        ↓
    TLS
        |
        ↓
    Certificate
        |
        ↓
    Domain

---

# 63. Kubernetes ConfigMap Problem

Check:

    kubectl get configmap

Then:

    kubectl describe configmap <name>

Verify:

    Key
    +
    Value
    +
    Namespace
    +
    Pod Reference

---

# 64. Kubernetes Secret Problem

Check:

    kubectl get secret

Do not expose secret values unnecessarily.

Verify:

    Secret Exists
    +
    Correct Namespace
    +
    Correct Key
    +
    Correct Pod Reference

---

# 65. Environment Variable Problem

Application may fail because:

    Required Variable Missing
        |
        ↓
    Application Startup
        |
        X
    Failure

Check:

    Deployment
        +
    ConfigMap
        +
    Secret
        +
    Application Logs

---

# 66. Volume Mount Problem

Possible causes:

    PVC Not Bound
    +
    Wrong Mount Path
    +
    Permission Problem
    +
    Storage Issue

Check:

    kubectl get pvc

and:

    kubectl describe pvc <pvc>

---

# 67. PVC Pending

Possible causes:

    No StorageClass
    +
    Provisioning Failure
    +
    Capacity Problem
    +
    Access Mode Problem

Check:

    kubectl get storageclass

---

# 68. Node NotReady

Possible causes:

    Kubelet Failure
    +
    Network Failure
    +
    Resource Pressure
    +
    Disk Problem
    +
    Node Failure

Check:

    kubectl get nodes

Then:

    kubectl describe node <node>

---

# 69. Node MemoryPressure

Possible causes:

    High Memory Usage
    +
    Too Many Pods
    +
    Memory Leak

Check:

    kubectl describe node <node>

---

# 70. Node DiskPressure

Possible causes:

    Container Logs
    +
    Images
    +
    Temporary Files
    +
    Disk Usage

Check:

    df -h

and node-level container storage.

---

# 71. Node Network Problem

Symptoms:

    Pod Communication Failure
    +
    DNS Problems
    +
    External Connectivity Problems

Check:

    Node
        |
        ↓
    Network
        |
        ↓
    CNI
        |
        ↓
    Pod

---

# 72. Kubernetes DNS Troubleshooting

Test from a pod:

    nslookup kubernetes.default

Check:

    CoreDNS
    +
    Service
    +
    Network
    +
    DNS Configuration

---

# 73. CoreDNS Troubleshooting

Check:

    kubectl get pods -n kube-system

Then:

    kubectl logs -n kube-system <coredns-pod>

Check:

    CoreDNS Pods
    +
    Service
    +
    Configuration
    +
    Node Network

---

# 74. Pod-to-Pod Connectivity

Test:

    Pod A
        |
        ↓
    Pod B

Check:

    NetworkPolicy
    +
    Service
    +
    DNS
    +
    CNI
    +
    Port

---

# 75. NetworkPolicy Problem

If connectivity suddenly fails:

    Check NetworkPolicy

Verify:

    Source
    +
    Destination
    +
    Port
    +
    Namespace Selector
    +
    Pod Selector

---

# 76. Database Connectivity Troubleshooting

Flow:

    Application
        |
        ↓
    DNS
        |
        ↓
    Network
        |
        ↓
    Security Group
        |
        ↓
    Port
        |
        ↓
    Database

Check each layer.

---

# 77. Database Connection Refused

Possible causes:

    Database Down
    +
    Wrong Host
    +
    Wrong Port
    +
    Security Group
    +
    Network ACL
    +
    Application Configuration

---

# 78. Database Authentication Failure

Possible causes:

    Wrong Username
    +
    Wrong Password
    +
    Secret Problem
    +
    User Permissions
    +
    Authentication Configuration

---

# 79. Database Connection Timeout

Possible causes:

    Network
    +
    Security Group
    +
    Wrong Endpoint
    +
    Database Overload
    +
    Route Problem

---

# 80. Database Slow

Check:

    CPU
    +
    Memory
    +
    Connections
    +
    Slow Queries
    +
    Locks
    +
    Storage
    +
    IOPS

---

# 81. Database Connection Exhaustion

Example:

    Application Pods
        |
        ↓
    Connection Pools
        |
        ↓
    Database
        |
        ↓
    Maximum Connections
        |
        ↓
    New Requests Fail

Investigate:

    Pool Size
    +
    Pod Count
    +
    Database Capacity

---

# 82. RabbitMQ Troubleshooting

Check:

    Queue Depth
    +
    Consumer Count
    +
    Consumer Health
    +
    Message Rate
    +
    Processing Time
    +
    Connections

---

# 83. Queue Backlog

Example:

    Producer
        |
        ↓
    RabbitMQ
        |
        ↓
    Consumer
        |
        ↓
    Slow Processing
        |
        ↓
    Queue Growth

Possible fixes:

    Add Consumers
    +
    Optimize Processing
    +
    Increase Capacity

---

# 84. CI/CD Troubleshooting

Start with:

    Pipeline Failed
        |
        ↓
    Identify Stage
        |
        ↓
    Read Logs
        |
        ↓
    Identify Error
        |
        ↓
    Reproduce
        |
        ↓
    Fix
        |
        ↓
    Rerun
        |
        ↓
    Validate

---

# 85. GitHub Actions Workflow Failure

Check:

    Workflow Run
        |
        ↓
    Failed Job
        |
        ↓
    Failed Step
        |
        ↓
    Logs
        |
        ↓
    Error Message

---

# 86. GitHub Actions Permission Error

Check:

    permissions:

Possible issue:

    contents: read

when workflow requires:

    contents: write

Grant only required permissions.

---

# 87. GitHub Actions Secret Problem

Check:

    Secret Name
    +
    Repository
    +
    Environment
    +
    Workflow Reference

Never print the secret.

---

# 88. GitHub Actions Dependency Failure

Possible causes:

    Network
    +
    Registry
    +
    Package Manager
    +
    Cache
    +
    Version

Check:

    Dependency Installation Logs

---

# 89. GitHub Actions Runner Problem

Check:

    Runner Status
    +
    Labels
    +
    Runner Availability
    +
    Network
    +
    Permissions

---

# 90. Self-Hosted Runner Problem

Check:

    Runner Service
    +
    Network
    +
    Disk
    +
    CPU
    +
    Memory
    +
    Workspace

---

# 91. Jenkins Pipeline Failure

Check:

    Stage
        |
        ↓
    Console Output
        |
        ↓
    Agent
        |
        ↓
    Credentials
        |
        ↓
    Dependencies
        |
        ↓
    Application

---

# 92. Docker Build Failure

Check:

    Dockerfile
    +
    Build Context
    +
    Base Image
    +
    Dependencies
    +
    Network
    +
    Registry

---

# 93. Docker Push Failure

Possible causes:

    Authentication
    +
    Registry URL
    +
    Repository
    +
    Network
    +
    Permissions

Flow:

    Docker Build
        |
        ↓
    Login
        |
        ↓
    Tag
        |
        ↓
    Push
        |
        X
    Failure

---

# 94. ECR Push Failure

Check:

    AWS Credentials
    +
    IAM Permissions
    +
    ECR Repository
    +
    Region
    +
    Registry URL

---

# 95. EKS Deployment Failure

Check:

    Image
        |
        ↓
    ECR
        |
        ↓
    EKS
        |
        ↓
    Pod
        |
        ↓
    Service
        |
        ↓
    ALB

Troubleshoot layer by layer.

---

# 96. ArgoCD Sync Failure

Check:

    Application Status
        |
        ↓
    Sync Status
        |
        ↓
    Health Status
        |
        ↓
    Events
        |
        ↓
    Manifest
        |
        ↓
    Kubernetes Resource

---

# 97. ArgoCD OutOfSync

Possible causes:

    Manual Kubernetes Change
    +
    Git Change
    +
    Incorrect Manifest
    +
    Automated Sync Behavior

Compare:

    Git Desired State

with:

    Cluster Actual State

---

# 98. ArgoCD Application Degraded

Check:

    ArgoCD
        |
        ↓
    Application
        |
        ↓
    Resource
        |
        ↓
    Pod / Service / Deployment
        |
        ↓
    Kubernetes Events

---

# 99. Terraform Troubleshooting

Start with:

    terraform plan

Then inspect:

    Error
    +
    Provider
    +
    Resource
    +
    State
    +
    Dependency

---

# 100. Terraform State Problem

Check:

    terraform state list

Then:

    terraform state show <resource>

Do not manually modify state unless you understand the consequences.

---

# 101. Terraform Resource Already Exists

Possible cause:

    Resource Exists
        |
        ↓
    Terraform State Does Not Know

Possible solution:

    Import Resource

Then:

    terraform plan

Validate the desired state before applying changes.

---

# 102. Terraform Dependency Problem

Terraform normally builds a dependency graph.

If dependency is unclear:

    Resource A
        |
        ?
    Resource B

Use appropriate references or explicit dependency declarations only when necessary.

---

# 103. Terraform Provider Error

Check:

    Provider Version
    +
    Credentials
    +
    Region
    +
    API Availability
    +
    Permissions

---

# 104. AWS IAM Troubleshooting

Check:

    Identity
        |
        ↓
    Policy
        |
        ↓
    Resource
        |
        ↓
    Action
        |
        ↓
    Condition

Verify least-privilege permissions.

---

# 105. AccessDenied

Possible causes:

    Missing IAM Permission
    +
    Resource Policy
    +
    Explicit Deny
    +
    Wrong Role
    +
    Wrong Account

---

# 106. AWS Resource Not Found

Possible causes:

    Wrong Region
    +
    Wrong Account
    +
    Wrong Resource ID
    +
    Resource Deleted

Verify context before changing anything.

---

# 107. EC2 Troubleshooting

Check:

    Instance State
    +
    System Status
    +
    Instance Status
    +
    Security Group
    +
    Route Table
    +
    Network ACL
    +
    Application
    +
    Disk

---

# 108. EC2 Application Not Reachable

Flow:

    Client
        |
        ↓
    DNS
        |
        ↓
    Internet / Network
        |
        ↓
    Security Group
        |
        ↓
    EC2
        |
        ↓
    Port
        |
        ↓
    Application

---

# 109. Security Group Troubleshooting

Check:

    Source
    +
    Destination Port
    +
    Protocol
    +
    Inbound Rules
    +
    Outbound Rules

---

# 110. Route Table Troubleshooting

Check:

    Subnet
        |
        ↓
    Route Table
        |
        ↓
    Target
        |
        ↓
    Internet Gateway / NAT / Other Target

---

# 111. NAT Gateway Troubleshooting

Private workloads requiring outbound internet access may depend on:

    Private Subnet
        |
        ↓
    Route Table
        |
        ↓
    NAT Gateway
        |
        ↓
    Internet Gateway
        |
        ↓
    Internet

Check each layer.

---

# 112. S3 Access Troubleshooting

Check:

    IAM Policy
    +
    Bucket Policy
    +
    Block Public Access
    +
    Encryption
    +
    Region
    +
    Object Permissions

---

# 113. RDS Connectivity Troubleshooting

Check:

    RDS Status
    +
    Endpoint
    +
    Port
    +
    Security Group
    +
    Subnet
    +
    Route
    +
    Credentials

---

# 114. Performance Troubleshooting

When application is slow:

    Check Request Rate
        |
        ↓
    Check P95 / P99
        |
        ↓
    Check CPU
        |
        ↓
    Check Memory
        |
        ↓
    Check Database
        |
        ↓
    Check Network
        |
        ↓
    Check External Dependencies
        |
        ↓
    Identify Bottleneck

---

# 115. High CPU Troubleshooting

    High CPU
        |
        ↓
    Identify Process
        |
        ↓
    Check Traffic
        |
        ↓
    Check Recent Changes
        |
        ↓
    Check Application
        |
        ↓
    Optimize / Scale
        |
        ↓
    Validate

---

# 116. High Memory Troubleshooting

    High Memory
        |
        ↓
    Identify Process
        |
        ↓
    Check Memory Trend
        |
        ↓
    Check Application
        |
        ↓
    Check Limits
        |
        ↓
    Check Memory Leak
        |
        ↓
    Fix
        |
        ↓
    Monitor

---

# 117. High Disk Usage

Check:

    df -h

Then:

    du -sh *

Look for:

    Logs
    +
    Temporary Files
    +
    Container Images
    +
    Build Artifacts

---

# 118. Disk Full Troubleshooting

Flow:

    Disk 100%
        |
        ↓
    Identify Files
        |
        ↓
    Identify Cause
        |
        ↓
    Clean Safely
        |
        ↓
    Configure Rotation
        |
        ↓
    Monitor

Do not blindly delete files.

---

# 119. High Network Latency

Check:

    DNS
    +
    Network Path
    +
    Load Balancer
    +
    Application
    +
    Database
    +
    External API

---

# 120. Application Timeout

Possible causes:

    Slow Database
    +
    Slow External API
    +
    Network
    +
    Resource Exhaustion
    +
    Incorrect Timeout

---

# 121. Troubleshooting Timeout

Flow:

    Timeout
        |
        ↓
    Identify Request
        |
        ↓
    Check Application Logs
        |
        ↓
    Check Dependency
        |
        ↓
    Check Network
        |
        ↓
    Check Database
        |
        ↓
    Identify Slow Component

---

# 122. Memory Leak Investigation

Symptoms:

    Memory
        |
        ↓
    Gradually Increasing
        |
        ↓
    High Memory
        |
        ↓
    OOMKilled

Check:

    Application Metrics
    +
    Heap
    +
    Objects
    +
    Recent Changes

---

# 123. Deployment Troubleshooting

Check:

    Deployment Status
        |
        ↓
    ReplicaSet
        |
        ↓
    Pods
        |
        ↓
    Events
        |
        ↓
    Probes
        |
        ↓
    Service
        |
        ↓
    Traffic

---

# 124. Deployment Stuck

Possible causes:

    Pod Pending
    +
    Image Pull
    +
    Readiness Failure
    +
    Resource Capacity
    +
    Scheduling
    +
    Application Startup

---

# 125. Rollout Status

Command:

    kubectl rollout status deployment/<deployment>

If stuck:

    kubectl describe deployment <deployment>

Then inspect pods.

---

# 126. Rollout History

Command:

    kubectl rollout history deployment/<deployment>

Useful for:

    Deployment Tracking
    +
    Rollback Investigation

---

# 127. Rollback

Command:

    kubectl rollout undo deployment/<deployment>

After rollback:

    Check Pods
        |
        ↓
    Check Health
        |
        ↓
    Check Metrics
        |
        ↓
    Validate Application

---

# 128. Database Migration Failure

Never assume application rollback automatically reverses database changes.

Before deployment:

    Migration
        |
        ↓
    Compatibility
        |
        ↓
    Backup
        |
        ↓
    Deployment

Use backward-compatible migration strategies where appropriate.

---

# 129. Application Deployment With Database Change

Prefer:

    Expand
        |
        ↓
    Deploy Compatible Application
        |
        ↓
    Migrate
        |
        ↓
    Contract

This can reduce rollback risk.

---

# 130. Security Incident Troubleshooting

If a credential is compromised:

    Detect
        |
        ↓
    Contain
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
    Remediate
        |
        ↓
    Monitor

---

# 131. Vulnerability Found in Production

Flow:

    Vulnerability
        |
        ↓
    Assess Severity
        |
        ↓
    Determine Exposure
        |
        ↓
    Mitigate
        |
        ↓
    Patch
        |
        ↓
    Rescan
        |
        ↓
    Validate

---

# 132. Certificate Expiration

Symptoms:

    HTTPS Failure
        |
        ↓
    Certificate Error

Check:

    Certificate Expiry
    +
    Domain
    +
    Issuer
    +
    Load Balancer
    +
    Application

---

# 133. Time Synchronization Problem

Incorrect system time can affect:

    TLS
    +
    Authentication
    +
    Tokens
    +
    Logs

Check:

    System Time
    +
    NTP
    +
    Timezone

---

# 134. Authentication Troubleshooting

Check:

    Identity
        |
        ↓
    Credentials
        |
        ↓
    Token
        |
        ↓
    Expiration
        |
        ↓
    Permissions
        |
        ↓
    Resource

---

# 135. Authorization Troubleshooting

Authentication:

    Who Are You?

Authorization:

    What Are You Allowed To Do?

Example:

    User Authenticated
        |
        ↓
    Request
        |
        ↓
    Permission Check
        |
        X
    Forbidden

---

# 136. Troubleshooting Logs

Good log investigation:

    Identify Timestamp
        |
        ↓
    Identify Service
        |
        ↓
    Identify Request
        |
        ↓
    Find Error
        |
        ↓
    Follow Related Logs
        |
        ↓
    Identify Root Cause

---

# 137. Log Correlation

For microservices:

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

Use correlation information to connect related logs.

---

# 138. Search Logs Efficiently

Use filters:

    Timestamp
    +
    Service
    +
    Error Level
    +
    Request ID
    +
    Status Code

Avoid searching blindly through all logs.

---

# 139. Metrics Before Logs

A useful approach:

    Metrics
        |
        ↓
    Identify Time Window
        |
        ↓
    Logs
        |
        ↓
    Find Error
        |
        ↓
    Root Cause

---

# 140. Logs Before Metrics

Sometimes the error itself is obvious in logs.

Use both:

    Metrics
        +
    Logs

rather than relying exclusively on one.

---

# 141. Troubleshooting With Prometheus

Check:

    CPU
    +
    Memory
    +
    Request Rate
    +
    Error Rate
    +
    Latency
    +
    Pod Restarts

---

# 142. Troubleshooting With Grafana

Use dashboards to identify:

    When
        +
    Where
        +
    How Much
        +
    Which Service

is affected.

---

# 143. Troubleshooting With ELK

Use ELK to search:

    Errors
    +
    Exceptions
    +
    Timeouts
    +
    Request IDs
    +
    Application Events

---

# 144. Alert Investigation

When an alert fires:

    Alert
        |
        ↓
    Confirm
        |
        ↓
    Check Dashboard
        |
        ↓
    Check Logs
        |
        ↓
    Check Recent Changes
        |
        ↓
    Investigate

---

# 145. False Alerts

If alerts repeatedly trigger without real incidents:

    Review Threshold
        |
        ↓
    Review Query
        |
        ↓
    Review Evaluation Period
        |
        ↓
    Tune Alert

---

# 146. Alert Fatigue

Too many alerts:

    Alerts
        |
        ↓
    Engineer
        |
        ↓
    Ignored

Better:

    Actionable Alerts
        |
        ↓
    Correct Severity
        |
        ↓
    Faster Response

---

# 147. Troubleshooting Performance Regression

    New Deployment
        |
        ↓
    Performance Degradation
        |
        ↓
    Compare Baseline
        |
        ↓
    Check Metrics
        |
        ↓
    Check Logs
        |
        ↓
    Check Database
        |
        ↓
    Check Dependencies
        |
        ↓
    Root Cause
        |
        ↓
    Fix / Rollback

---

# 148. Troubleshooting Configuration Drift

    Git
        |
        ↓
    Desired State

    Cluster
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
    Prevent Manual Changes

---

# 149. Troubleshooting GitOps

Check:

    Repository
        |
        ↓
    Manifest
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Pod

Find the first layer where the expected state differs.

---

# 150. Troubleshooting Method

Use this general sequence:

    1. Understand the Problem
        |
        ↓
    2. Assess Impact
        |
        ↓
    3. Collect Evidence
        |
        ↓
    4. Check Recent Changes
        |
        ↓
    5. Form Hypothesis
        |
        ↓
    6. Test Hypothesis
        |
        ↓
    7. Fix
        |
        ↓
    8. Validate
        |
        ↓
    9. Document
        |
        ↓
    10. Prevent Recurrence

---

# 151. Troubleshooting Checklist

## Application

    Check Status
    Check Logs
    Check CPU
    Check Memory
    Check Dependencies
    Check Configuration
    Check Recent Deployment
    Check Error Rate
    Check Latency

## Kubernetes

    kubectl get pods
    kubectl describe pod
    kubectl logs
    kubectl logs --previous
    kubectl get events
    kubectl get svc
    kubectl get endpoints
    kubectl get ingress
    kubectl get nodes
    kubectl top pod
    kubectl top nodes

## Network

    DNS
    Connectivity
    Ports
    Security Groups
    Network Policies
    Routes
    Load Balancer
    TLS

## Database

    Connectivity
    Credentials
    Connections
    CPU
    Memory
    Storage
    Slow Queries
    Locks

## CI/CD

    Failed Stage
    Failed Step
    Logs
    Runner
    Credentials
    Dependencies
    Artifacts
    Permissions

## Terraform

    Error
    State
    Provider
    Plan
    Resource
    Dependencies
    Credentials

## GitOps

    Git
    Manifest
    ArgoCD
    Sync Status
    Health Status
    Kubernetes Events

---

# 152. Production Troubleshooting Checklist

Before making a change:

    What Is The Impact?
    +
    What Is The Evidence?
    +
    What Changed?
    +
    What Is The Hypothesis?
    +
    What Is The Risk Of This Change?
    +
    How Will I Validate It?
    +
    How Will I Roll It Back?

---

# 153. Safe Troubleshooting

Prefer:

    Read-Only Investigation
        |
        ↓
    Evidence
        |
        ↓
    Controlled Change
        |
        ↓
    Validation

Avoid:

    Random Production Changes

---

# 154. Troubleshooting Commands Quick Reference

## Linux

    ps -ef
    top
    free -h
    df -h
    du -sh *
    ss -lntp
    systemctl status <service>
    journalctl -u <service>
    curl -v <url>
    dig <domain>
    nslookup <domain>

## Kubernetes

    kubectl get pods
    kubectl describe pod <pod>
    kubectl logs <pod>
    kubectl logs <pod> --previous
    kubectl get events
    kubectl get svc
    kubectl get endpoints
    kubectl get ingress
    kubectl get nodes
    kubectl describe node <node>
    kubectl top pod
    kubectl top nodes
    kubectl rollout status deployment/<deployment>
    kubectl rollout history deployment/<deployment>
    kubectl rollout undo deployment/<deployment>

## Terraform

    terraform fmt
    terraform validate
    terraform plan
    terraform state list
    terraform state show <resource>

---

# 155. Troubleshooting Anti-Patterns

## Random Restart

Bad:

    Problem
        |
        ↓
    Restart Everything

Better:

    Problem
        |
        ↓
    Investigate
        |
        ↓
    Identify Cause
        |
        ↓
    Restart Only If Appropriate

---

# 156. Random Scaling

Bad:

    Slow
        |
        ↓
    Add More Pods

Better:

    Slow
        |
        ↓
    Identify Bottleneck
        |
        ↓
    Scale or Optimize Correct Layer

---

# 157. Random Configuration Changes

Bad:

    Error
        |
        ↓
    Change Configuration
        |
        ↓
    Unknown Result

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
    Validate

---

# 158. Ignoring Logs

Bad:

    Alert
        |
        ↓
    Guess

Better:

    Alert
        |
        ↓
    Metrics
        |
        ↓
    Logs
        |
        ↓
    Events
        |
        ↓
    Root Cause

---

# 159. Ignoring Recent Changes

Bad:

    Incident
        |
        ↓
    Investigate Everything

Better:

    Incident
        |
        ↓
    Check Recent Changes
        |
        ↓
    Narrow Investigation
        |
        ↓
    Confirm

---

# 160. Fixing Symptoms Only

Bad:

    Pod Restarting
        |
        ↓
    Restart Pod
        |
        ↓
    Same Problem

Better:

    Pod Restarting
        |
        ↓
    Find Cause
        |
        ↓
    Fix Cause
        |
        ↓
    Validate

---

# 161. No Validation After Fix

Bad:

    Fix
        |
        ↓
    Assume Success

Better:

    Fix
        |
        ↓
    Metrics
        +
    Logs
        +
    Health Checks
        +
    Application Test
        |
        ↓
    Confirm Recovery

---

# 162. No Documentation

After resolving an important incident:

    Problem
        |
        ↓
    Root Cause
        |
        ↓
    Fix
        |
        ↓
    Prevention

Document the result.

---

# 163. No Post-Incident Review

After recovery:

    Review:

    What Happened?
    +
    Why?
    +
    What Worked?
    +
    What Failed?
    +
    What Should Change?

---

# 164. Troubleshooting Decision Tree

    Is Application Reachable?
        |
        +------ No
        |        |
        |        ↓
        |     DNS / Network / LB
        |
        +------ Yes
                 |
                 ↓
             Is Response Slow?
                 |
                 +------ Yes
                 |        |
                 |        ↓
                 |     Metrics / DB / Network
                 |
                 +------ No
                          |
                          ↓
                      Is Error Rate High?
                          |
                          +------ Yes
                          |        |
                          |        ↓
                          |     Logs / Deployment
                          |
                          +------ No
                                   |
                                   ↓
                              Check User Issue

---

# 165. Kubernetes Decision Tree

    Pod Running?
        |
        +------ No
        |        |
        |        ↓
        |     Describe Pod
        |        |
        |        ↓
        |     Check Events
        |
        +------ Yes
                 |
                 ↓
            Pod Ready?
                 |
                 +------ No
                 |        |
                 |        ↓
                 |     Check Readiness
                 |
                 +------ Yes
                          |
                          ↓
                     Service Working?
                          |
                          +------ No
                          |        |
                          |        ↓
                          |     Check Selector
                          |     Check Endpoints
                          |
                          +------ Yes
                                   |
                                   ↓
                               Check Ingress / ALB

---

# 166. Database Troubleshooting Decision Tree

    Can Application Resolve DB?
        |
        +------ No
        |        |
        |        ↓
        |     DNS
        |
        +------ Yes
                 |
                 ↓
            Can Connect?
                 |
                 +------ No
                 |        |
                 |        ↓
                 |     Network / SG / Port
                 |
                 +------ Yes
                          |
                          ↓
                      Authenticate?
                          |
                          +------ No
                          |        |
                          |        ↓
                          |     Credentials
                          |
                          +------ Yes
                                   |
                                   ↓
                              Query Slow?
                                   |
                                   +------ Yes
                                   |        |
                                   |        ↓
                                   |     Query / Index / DB
                                   |
                                   +------ No
                                            |
                                            ↓
                                        Application

---

# 167. CI/CD Troubleshooting Decision Tree

    Pipeline Started?
        |
        +------ No
        |        |
        |        ↓
        |     Trigger / Permissions
        |
        +------ Yes
                 |
                 ↓
             Which Stage?
                 |
                 ↓
             Check Logs
                 |
                 ↓
             Dependency?
                 |
                 +------ Yes → Cache / Registry / Network
                 |
                 ↓
             Build?
                 |
                 +------ Yes → Docker / Code / Tool
                 |
                 ↓
             Test?
                 |
                 +------ Yes → Test / Environment
                 |
                 ↓
             Deploy?
                 |
                 +------ Yes → Kubernetes / AWS / Credentials

---

# 168. Incident Stabilization

During a production incident:

    Detect
        |
        ↓
    Assess
        |
        ↓
    Stabilize
        |
        ↓
    Investigate
        |
        ↓
    Recover

Stabilization may include:

    Rollback
    +
    Traffic Reduction
    +
    Scaling
    +
    Disabling Faulty Feature
    +
    Failover

Only use actions appropriate to the incident.

---

# 169. Rollback During Incident

If a recent deployment is strongly correlated with the incident:

    Current Version
        |
        ↓
    Incident
        |
        ↓
    Rollback
        |
        ↓
    Previous Version
        |
        ↓
    Validate

Then investigate the failed version separately.

---

# 170. Troubleshooting Communication

During a major incident communicate:

    Impact
        +
    Current Status
        +
    Actions
        +
    Next Step

Avoid unnecessary speculation.

---

# 171. Root Cause Analysis

A strong RCA contains:

    Incident
    +
    Timeline
    +
    Impact
    +
    Root Cause
    +
    Contributing Factors
    +
    Resolution
    +
    Corrective Actions
    +
    Preventive Actions

---

# 172. RCA Example

Incident:

    Production API returned 503.

Timeline:

    Deployment
        |
        ↓
    Readiness Failure
        |
        ↓
    No Healthy Backends
        |
        ↓
    503

Root Cause:

    Incorrect readiness probe configuration.

Resolution:

    Corrected probe configuration.

Prevention:

    Add manifest validation
        +
    Deployment smoke tests
        +
    Production readiness checks

---

# 173. Corrective vs Preventive Action

Corrective:

    Fix Current Problem

Preventive:

    Stop Problem From Happening Again

Example:

    Current:
    Fix Broken Probe

    Preventive:
    Add Automated Validation

---

# 174. Troubleshooting Automation

Automate repetitive checks.

Example:

    Alert
        |
        ↓
    Automated Diagnostics
        |
        ↓
    Metrics
        +
    Logs
        +
    Events
        |
        ↓
    Engineer

Automation should support engineers, not hide important evidence.

---

# 175. Runbooks

A runbook should contain:

    Symptoms
    +
    Checks
    +
    Commands
    +
    Expected Results
    +
    Remediation
    +
    Rollback
    +
    Validation

---

# 176. Example Runbook Structure

    Incident:
    CrashLoopBackOff

    Step 1:
    kubectl get pods

    Step 2:
    kubectl describe pod <pod>

    Step 3:
    kubectl logs <pod> --previous

    Step 4:
    Check configuration

    Step 5:
    Check resources

    Step 6:
    Fix root cause

    Step 7:
    Validate pod health

---

# 177. Troubleshooting Principles

- Start with impact
- Do not guess
- Collect evidence
- Check recent changes
- Use metrics
- Use logs
- Use events
- Isolate the problem
- Form a hypothesis
- Test the hypothesis
- Make controlled changes
- Validate every fix
- Document important incidents
- Build preventive controls

---

# 178. Production Troubleshooting Golden Rules

    1. Protect Users First
        |
        ↓
    2. Stabilize the System
        |
        ↓
    3. Collect Evidence
        |
        ↓
    4. Avoid Random Changes
        |
        ↓
    5. Check Recent Changes
        |
        ↓
    6. Find Root Cause
        |
        ↓
    7. Validate Recovery
        |
        ↓
    8. Prevent Recurrence

---

# 179. Interview Questions

## Basic

1. What is troubleshooting?

2. What is the difference between a symptom and root cause?

3. What is the first thing you check during an incident?

4. What is CrashLoopBackOff?

5. What is ImagePullBackOff?

6. What does OOMKilled mean?

7. What is a readiness probe?

8. What is a liveness probe?

9. What is a startup probe?

10. How do you check Kubernetes pod logs?

---

# 180. Intermediate Interview Questions

11. How would you troubleshoot a pod in CrashLoopBackOff?

12. How would you troubleshoot ImagePullBackOff?

13. How would you troubleshoot a pod stuck in Pending?

14. How would you troubleshoot a service with no endpoints?

15. How would you troubleshoot a 503 error?

16. How would you troubleshoot a 504 error?

17. How would you troubleshoot high CPU?

18. How would you troubleshoot high memory?

19. How would you troubleshoot a database connection failure?

20. How would you troubleshoot DNS problems?

21. How would you troubleshoot an ALB target marked unhealthy?

22. How would you troubleshoot a failed GitHub Actions pipeline?

23. How would you troubleshoot a failed Terraform apply?

24. How would you troubleshoot an ArgoCD sync failure?

25. How would you troubleshoot configuration drift?

---

# 181. Advanced Interview Questions

26. How would you troubleshoot a production outage?

27. How would you identify the root cause of a distributed system failure?

28. How would you troubleshoot intermittent 503 errors?

29. How would you troubleshoot intermittent latency?

30. How would you troubleshoot a Kubernetes cluster under resource pressure?

31. How would you troubleshoot database connection exhaustion?

32. How would you troubleshoot a microservices dependency failure?

33. How would you troubleshoot a retry storm?

34. How would you troubleshoot a production performance regression?

35. How would you troubleshoot a failed zero-downtime deployment?

36. How would you troubleshoot an ArgoCD application stuck in degraded state?

37. How would you troubleshoot an EKS application that works inside the cluster but not through ALB?

38. How would you troubleshoot a Terraform state inconsistency?

39. How would you troubleshoot an AWS AccessDenied error?

40. How would you design a systematic production troubleshooting process?

---

# 182. Scenario: Deployment Succeeded but Users Receive 503

Answer:

    I would first confirm whether the pods are running and ready.

    Then I would check service endpoints and the ingress or ALB
    configuration.

    I would inspect readiness probe failures, target health,
    application logs, and recent deployment changes.

Flow:

    Deployment
        |
        ↓
    Pods
        |
        ↓
    Readiness
        |
        ↓
    Service
        |
        ↓
    Endpoints
        |
        ↓
    ALB / Ingress
        |
        ↓
    Application
        |
        ↓
    Root Cause

---

# 183. Scenario: Pod Is in CrashLoopBackOff

Answer:

    I would start with kubectl describe pod to inspect events.

    Then I would check kubectl logs --previous to see why the
    previous container instance failed.

    I would check environment variables, secrets, ConfigMaps,
    resource limits, health probes, and application startup errors.

---

# 184. Scenario: Pod Is Pending

Answer:

    I would run kubectl describe pod and inspect the Events section.

    I would look for insufficient CPU or memory, node selectors,
    taints, tolerations, affinity rules, and available node
    capacity.

---

# 185. Scenario: Service Has No Endpoints

Answer:

    I would compare the service selector with the labels on the
    pods.

    Then I would check whether the pods are Ready.

Flow:

    Service
        |
        ↓
    Selector
        |
        ↓
    Pod Labels
        |
        ↓
    Ready Pods
        |
        ↓
    Endpoints

---

# 186. Scenario: Application Returns 504

Answer:

    I would determine which component is timing out.

    I would check ALB or ingress timeout settings, application
    latency, database queries, external APIs, network connectivity,
    and application logs.

---

# 187. Scenario: High CPU After Deployment

Answer:

    I would compare CPU usage before and after the deployment.

    Then I would check traffic, application behavior, recent code
    changes, CPU limits, and whether the workload needs scaling.

---

# 188. Scenario: Database Connection Failure

Answer:

    I would troubleshoot from the application outward:

    DNS
        |
        ↓
    Network
        |
        ↓
    Security Group
        |
        ↓
    Port
        |
        ↓
    Database
        |
        ↓
    Authentication

This isolates the failing layer.

---

# 189. Scenario: ArgoCD Shows OutOfSync

Answer:

    I would compare the desired state in Git with the actual state
    in Kubernetes.

    I would determine whether the difference came from a Git change,
    manual Kubernetes change, or another controller.

    Then I would reconcile the environment through the approved
    GitOps process.

---

# 190. Scenario: Terraform Apply Failed

Answer:

    I would inspect the exact Terraform error first.

    Then I would check Terraform state and run terraform plan.

    I would identify which resources were successfully created,
    fix the underlying issue, and apply the corrected configuration.

I would avoid destroying resources unless destruction is actually
required and understood.

---

# 191. Scenario: GitHub Actions Workflow Fails

Answer:

    I would identify the failed job and failed step.

    Then I would inspect the logs and determine whether the issue
    is related to permissions, secrets, dependencies, runner
    configuration, application code, or deployment.

---

# 192. Scenario: Production Becomes Slow

Answer:

    I would first check P95 and P99 latency.

    Then I would investigate:

    CPU
    +
    Memory
    +
    Request Rate
    +
    Database
    +
    Network
    +
    External Dependencies
    +
    Recent Changes

I would identify the bottleneck before making changes.

---

# 193. Scenario: Disk Is Full

Answer:

    I would check df -h first.

    Then I would identify large directories using du.

    I would determine whether logs, container images, temporary
    files, or build artifacts are responsible.

    I would clean data safely and implement appropriate retention
    or rotation policies.

---

# 194. Scenario: Secret Exposed

Answer:

    I would immediately treat the secret as compromised.

    I would revoke or rotate it, investigate possible usage,
    remove the exposure where appropriate, and add preventive
    secret scanning.

---

# 195. Final Troubleshooting Model

Remember:

    OBSERVE
        |
        ↓
    MEASURE
        |
        ↓
    ISOLATE
        |
        ↓
    HYPOTHESIZE
        |
        ↓
    TEST
        |
        ↓
    FIX
        |
        ↓
    VALIDATE
        |
        ↓
    DOCUMENT
        |
        ↓
    PREVENT

---

# Final Concept

Strong DevOps troubleshooting is not about knowing every command.

It is about knowing:

    What To Check
        +
    In What Order
        +
    Why You Are Checking It
        +
    How To Interpret Evidence
        +
    How To Make Safe Changes
        +
    How To Validate Recovery

The best troubleshooting approach is:

    Evidence First
        +
    Root Cause First
        +
    Controlled Changes
        +
    Continuous Validation
        +
    Prevention